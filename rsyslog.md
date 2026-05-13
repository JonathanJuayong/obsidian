# Log Collection with rsyslog

## Table of Contents

1. [What is rsyslog?](#what-is-rsyslog)
2. [Installation](#installation)
3. [Configuration Basics](#configuration-basics)
4. [Collecting Local Logs](#collecting-local-logs)
5. [Collecting Remote Logs](#collecting-remote-logs)
6. [Forwarding Logs to a Central Server](#forwarding-logs-to-a-central-server)
7. [Filtering and Routing Logs](#filtering-and-routing-logs)
8. [Log Rotation](#log-rotation)
9. [Testing Your Configuration](#testing-your-configuration)
10. [Troubleshooting](#troubleshooting)

---

## What is rsyslog?

**rsyslog** (Rocket-fast System for Log Processing) is an open-source log processing daemon used on most Linux distributions. It extends the traditional syslog protocol with features like:

- High-performance log ingestion and forwarding
- TCP and UDP transport (with TLS support)
- Structured logging (JSON, CEF)
- Buffering and queuing for reliability
- Modular input/output plugins

rsyslog uses a **facility.severity** system to categorise messages, and routes them via rules defined in its configuration files.

---

## Installation

rsyslog is pre-installed on most distributions. If not:

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install rsyslog -y

# RHEL / CentOS / Fedora
sudo dnf install rsyslog -y

# Enable and start the service
sudo systemctl enable rsyslog
sudo systemctl start rsyslog
```

Verify the version:

```bash
rsyslogd -v
```

---

## Configuration Basics

The main configuration file is `/etc/rsyslog.conf`. Additional drop-in files live in:

```
/etc/rsyslog.d/*.conf
```

The configuration is split into three sections:

|Section|Purpose|
|---|---|
|**Modules**|Load input/output plugins (`module(...)`)|
|**Global directives**|Set defaults (queue size, file permissions, etc.)|
|**Rules**|Define what to collect and where to send it|

A rule follows this pattern:

```
FILTER    ACTION
```

For example:

```
*.info    /var/log/messages
```

### Facility and Severity Reference

|Facility|Description|
|---|---|
|`auth` / `authpriv`|Authentication and security|
|`cron`|Scheduled tasks|
|`daemon`|System daemons|
|`kern`|Kernel messages|
|`mail`|Mail subsystem|
|`syslog`|rsyslog internal messages|
|`user`|Generic user-level messages|
|`local0–local7`|Custom application use|

|Severity|Level|
|---|---|
|`emerg`|0 — System unusable|
|`alert`|1 — Immediate action required|
|`crit`|2 — Critical conditions|
|`err`|3 — Error conditions|
|`warning`|4 — Warning conditions|
|`notice`|5 — Normal but significant|
|`info`|6 — Informational|
|`debug`|7 — Debug-level messages|

---

## Collecting Local Logs

### Enable Standard Input Modules

At the top of `/etc/rsyslog.conf`, ensure the IMUXSOCK and IMJOURNAL modules are loaded to collect local system logs:

```bash
# Provides support for local system logging
module(load="imuxsock")

# Reads from systemd journal (on systemd-based systems)
module(load="imjournal" StateFile="imjournal.state")
```

### Basic Local Log Rules

```bash
# Auth logs — captures login attempts, sudo usage, SSH
auth,authpriv.*             /var/log/auth.log

# All emergency messages to all logged-in users
*.emerg                     :omusrmsg:*

# Kernel messages
kern.*                      /var/log/kern.log

# Cron logs
cron.*                      /var/log/cron.log

# Everything at info and above, except mail, auth, and cron
*.info;mail.none;authpriv.none;cron.none    /var/log/messages

# Catch-all debug log
*.debug                     /var/log/debug.log
```

### Collecting Logs from a Specific Application

Applications can write to `local0`–`local7` facilities. For example, to collect logs from a custom app writing to `local3`:

```bash
local3.*    /var/log/myapp.log
```

In your application, configure it to log to `local3` (most logging libraries support syslog facilities).

---

## Collecting Remote Logs

To receive logs from other machines, enable a network input module.

### UDP (simpler, lower overhead)

```bash
# /etc/rsyslog.conf

module(load="imudp")
input(type="imudp" port="514")
```

### TCP (reliable delivery, recommended)

```bash
module(load="imtcp")
input(type="imtcp" port="514")
```

### TLS-Encrypted TCP (recommended for production)

```bash
module(load="imtcp")
module(load="gtls" streamDriver.mode="1" streamDriver.authMode="anon")

input(
  type="imtcp"
  port="6514"
  streamDriver.mode="1"
  streamDriver.authMode="anon"
)
```

> **Firewall note:** Open the relevant port on the log server:
> 
> ```bash
> sudo firewall-cmd --permanent --add-port=514/tcp
> sudo firewall-cmd --reload
> # or with ufw:
> sudo ufw allow 514/tcp
> ```

### Store Remote Logs by Host

Use a template to organise incoming logs into per-host directories:

```bash
# Define a template: /var/log/remote/<hostname>/<date>.log
template(name="RemoteHostLog" type="string"
  string="/var/log/remote/%HOSTNAME%/%$YEAR%-%$MONTH%-%$DAY%.log")

# Apply the template to all remotely received messages
*.* ?RemoteHostLog
```

---

## Forwarding Logs to a Central Server

On the **client** (log sender), configure rsyslog to forward logs upstream.

### Forward All Logs via UDP

```bash
*.* @192.168.1.100:514
```

> Single `@` = UDP

### Forward All Logs via TCP

```bash
*.* @@192.168.1.100:514
```

> Double `@@` = TCP

### Forward Specific Logs Only

```bash
# Forward only auth logs
auth,authpriv.*  @@logserver.example.com:514

# Forward critical and above from all facilities
*.crit           @@logserver.example.com:514
```

### Forwarding with a Queue (Resilient)

If the central server is temporarily unreachable, a disk-assisted queue prevents log loss:

```bash
action(
  type="omfwd"
  target="logserver.example.com"
  port="514"
  protocol="tcp"
  queue.type="LinkedList"
  queue.filename="fwd_queue"
  queue.saveOnShutdown="on"
  action.resumeRetryCount="-1"
)
```

---

## Filtering and Routing Logs

### Filter by Facility and Severity

```bash
# Only errors and above from the mail facility
mail.err    /var/log/mail-errors.log

# Everything except debug from all facilities
*.!debug    /var/log/general.log
```

### Filter by Message Content (Property-Based)

```bash
# Drop noisy messages containing a specific string
:msg, contains, "health check"    stop

# Route messages matching a pattern to a file
:msg, regex, ".*failed password.*"    /var/log/auth-failures.log
```

### Filter by Program Name

```bash
# Collect logs from sshd only
:programname, isequal, "sshd"    /var/log/sshd.log
& stop
```

> `& stop` prevents the message from being processed by subsequent rules.

---

## Log Rotation

rsyslog itself doesn't rotate logs — that's handled by [[**logrotate**]]. Create a config file in `/etc/logrotate.d/`:

```bash
# /etc/logrotate.d/rsyslog
/var/log/messages
/var/log/auth.log
/var/log/kern.log
/var/log/cron.log
{
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    sharedscripts
    postrotate
        /usr/bin/systemctl kill -s HUP rsyslog.service >/dev/null 2>&1 || true
    endscript
}
```

---

## Testing Your Configuration

### Check Config Syntax

```bash
sudo rsyslogd -N1
```

A clean output ends with `rsyslogd: End of config validation run.`

### Send a Test Message

```bash
logger -p auth.info "Test auth log message"
logger -p local3.debug "Test app log message"
```

Then verify it landed in the expected file:

```bash
tail -f /var/log/auth.log
tail -f /var/log/messages
```

### Reload After Changes

```bash
sudo systemctl restart rsyslog
# or for a soft reload (no log gap):
sudo systemctl kill -HUP rsyslog
```

---

## Troubleshooting

### rsyslog Won't Start

```bash
# Check for config errors
sudo rsyslogd -N1

# Check service status and recent errors
sudo systemctl status rsyslog
sudo journalctl -u rsyslog -n 50
```

### Logs Not Appearing

- Confirm the correct facility/severity in the rule matches what the application sends
- Use `logger` to send a test message with a known facility
- Check file permissions: rsyslog typically runs as `root` or `syslog`
- Ensure no earlier rule with `& stop` is swallowing the message

### Remote Logs Not Arriving

- Verify the input module is loaded (`imtcp` or `imudp`)
- Check firewall rules on both sender and receiver
- Test connectivity: `nc -zv logserver.example.com 514`
- On the sender, check for queue backlog in `/var/spool/rsyslog/`

### Check What rsyslog Is Actually Doing

Enable internal debug output temporarily:

```bash
sudo rsyslogd -dn 2>&1 | head -100
```

---

## Quick Reference

```
/etc/rsyslog.conf          Main config file
/etc/rsyslog.d/*.conf      Drop-in config directory
/var/log/                  Default log output directory
/var/spool/rsyslog/        Queue spool directory

systemctl restart rsyslog  Apply config changes
rsyslogd -N1               Validate config syntax
logger -p facility.sev     Send a test log message
```

# Example rsyslog.conf

A well-commented example covering a typical server setup.

---

```bash
# /etc/rsyslog.conf
# =============================================================
# SECTION 1: MODULES
# =============================================================

# Local system logging (e.g. from logger command)
module(load="imuxsock")

# Read from systemd journal
module(load="imjournal" StateFile="imjournal.state")

# Receive logs over UDP (comment out if not a log server)
module(load="imudp")
input(type="imudp" port="514")

# Receive logs over TCP (comment out if not a log server)
module(load="imtcp")
input(type="imtcp" port="514")


# =============================================================
# SECTION 2: GLOBAL DIRECTIVES
# =============================================================

# Where to store the journal state file
global(workDirectory="/var/spool/rsyslog")

# Default permissions for log files and directories
$FileOwner root
$FileGroup adm
$FileCreateMode 0640
$DirCreateMode 0755
$Umask 0022

# Suppress duplicate messages
$RepeatedMsgReduction on


# =============================================================
# SECTION 3: TEMPLATES
# =============================================================

# Standard log line format
template(name="StandardFormat" type="string"
  string="%TIMESTAMP% %HOSTNAME% %syslogtag%%msg%\n")

# JSON format (useful for log shippers like Filebeat/Logstash)
template(name="JSONFormat" type="string"
  string="{\"timestamp\":\"%TIMESTAMP:::date-rfc3339%\",\"host\":\"%HOSTNAME%\",\"severity\":\"%syslogseverity-text%\",\"facility\":\"%syslogfacility-text%\",\"program\":\"%programname%\",\"message\":\"%msg:::json%\"}\n")

# Per-host remote log storage
template(name="RemoteHostLog" type="string"
  string="/var/log/remote/%HOSTNAME%/%$YEAR%-%$MONTH%-%$DAY%.log")


# =============================================================
# SECTION 4: RULES
# =============================================================

# --- Noise reduction ---
# Drop noisy/useless messages before they hit any rules below
:msg, contains, "Connection closed by authenticating user"  stop
:msg, contains, "CRON["                                     stop


# --- Auth / Security ---
auth,authpriv.*                         /var/log/auth.log


# --- Kernel ---
kern.*                                  /var/log/kern.log


# --- Mail ---
mail.info                               /var/log/mail.info
mail.warning                            /var/log/mail.warn
mail.err                                /var/log/mail.err


# --- Cron ---
cron.*                                  /var/log/cron.log


# --- Emergency: broadcast to all logged-in users ---
*.emerg                                 :omusrmsg:*


# --- Custom application on local3 ---
local3.*                                /var/log/myapp.log
& stop


# --- General catch-all ---
# Everything at info and above, excluding noisy facilities
*.info;mail.none;authpriv.none;cron.none    /var/log/messages


# --- Store incoming remote logs by hostname ---
*.* ?RemoteHostLog


# --- Forward everything to a central log server ---
# Uses a disk-assisted queue so logs are not lost if server is unreachable
action(
  type="omfwd"
  target="logserver.example.com"
  port="514"
  protocol="tcp"
  template="StandardFormat"
  queue.type="LinkedList"
  queue.filename="fwd_queue"
  queue.maxDiskSpace="1g"
  queue.saveOnShutdown="on"
  action.resumeRetryCount="-1"
)
```

---

## Notes

### Order matters

Rules are evaluated top to bottom. The `& stop` after `local3.*` prevents those messages from also landing in `/var/log/messages`. Always put more specific rules before general catch-alls.

### Noise reduction up top

Dropping known-noisy messages early prevents them from matching any subsequent rule, keeping logs clean and reducing disk I/O.

### Remote storage and forwarding can coexist

The `?RemoteHostLog` template writes incoming logs to disk locally, and the `omfwd` action forwards them upstream simultaneously. Both rules see the same message.

### Disable what you don't need

The `imudp` and `imtcp` input modules should be commented out if this machine is a **client only** and not acting as a central log server. Unnecessary open ports are unnecessary attack surface.

### Queue behaviour

The `omfwd` action uses a disk-assisted queue (`queue.type="LinkedList"`, `queue.saveOnShutdown="on"`). If the central log server is unreachable, messages are spooled to `/var/spool/rsyslog/fwd_queue` and replayed automatically when connectivity is restored. `action.resumeRetryCount="-1"` means it retries forever.

---

## Validate and apply

```bash
# Check syntax before restarting
sudo rsyslogd -N1

# Apply changes
sudo systemctl restart rsyslog

# Confirm it's running cleanly
sudo systemctl status rsyslog
```