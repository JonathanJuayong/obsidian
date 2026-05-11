File Transfer Protocol

Allows remote transfer of files over a network using a client-server model.

## Active vs Passive

- In an Active connection, the client opens a port and listens. The server is required to actively connect to it. 
- In a Passive connection, the server opens a port and listens (passively) and the client connects to it.

# FTP Cheat Sheet

## Connecting

|Command|Description|
|---|---|
|`ftp <host>`|Connect to a host|
|`ftp <host> <port>`|Connect on a specific port|
|`ftp -p <host>`|Connect in passive mode|
|`open <host>`|Open connection from within FTP shell|
|`bye` / `quit` / `exit`|Close connection and exit|
|`close` / `disconnect`|Close connection, stay in shell|

---

## Authentication

|Command|Description|
|---|---|
|`user <username>`|Send username|
|`pass <password>`|Send password|
|`anonymous`|Log in as anonymous user|

---

## Navigation

|Command|Description|
|---|---|
|`pwd`|Print working directory (remote)|
|`lpwd`|Print working directory (local)|
|`cd <dir>`|Change remote directory|
|`lcd <dir>`|Change local directory|
|`ls` / `dir`|List remote directory contents|
|`ls -la`|List with hidden files and details|
|`!ls`|List local directory contents|

---

## Transferring Files

|Command|Description|
|---|---|
|`get <file>`|Download a file|
|`get <file> <localname>`|Download and rename locally|
|`put <file>`|Upload a file|
|`put <file> <remotename>`|Upload and rename remotely|
|`mget <pattern>`|Download multiple files (e.g. `mget *.txt`)|
|`mput <pattern>`|Upload multiple files|
|`recv <file>`|Alias for `get`|
|`send <file>`|Alias for `put`|

---

## Transfer Modes

|Command|Description|
|---|---|
|`ascii`|Switch to ASCII mode (text files)|
|`binary` / `bin`|Switch to binary mode (images, archives, etc.)|
|`type`|Show current transfer mode|
|`passive`|Toggle passive mode on/off|

> **Tip:** Always use `binary` mode for non-text files to avoid corruption.

---

## File & Directory Operations

|Command|Description|
|---|---|
|`mkdir <dir>`|Create remote directory|
|`rmdir <dir>`|Remove remote directory|
|`delete <file>`|Delete a remote file|
|`mdelete <pattern>`|Delete multiple remote files|
|`rename <old> <new>`|Rename a remote file|

---

## Prompting & Automation

|Command|Description|
|---|---|
|`prompt`|Toggle interactive prompting for mget/mput|
|`glob`|Toggle filename globbing (wildcards)|
|`hash`|Toggle hash mark (`#`) progress display|
|`tick`|Toggle transfer progress counter|
|`verbose`|Toggle verbose output|

---

## Useful Options (CLI flags)

|Flag|Description|
|---|---|
|`-v`|Verbose mode|
|`-d`|Debug mode|
|`-n`|No auto-login|
|`-i`|Disable interactive prompting|
|`-p`|Passive mode|

---

## .netrc File (Auto-login)

Store credentials in `~/.netrc` to avoid typing them each time:

```
machine ftp.example.com
login myuser
password mypassword
```

```bash
chmod 600 ~/.netrc   # Secure the file
```

---

## Common Workflows

### Download all files from a directory

```ftp
binary
prompt off
mget *
```

### Upload all local `.csv` files

```ftp
binary
prompt off
mput *.csv
```

### Scripted / non-interactive FTP

```bash
ftp -n <host> <<EOF
user myuser mypassword
binary
cd /uploads
put report.csv
bye
EOF
```

---

## SFTP vs FTPS vs FTP

|Protocol|Port|Encryption|Notes|
|---|---|---|---|
|FTP|21|None|Plain text — avoid on public networks|
|FTPS|990 / 21|SSL/TLS|FTP with TLS wrapper|
|SFTP|22|SSH|Different protocol; uses SSH|

> **Security tip:** Prefer **SFTP** or **FTPS** over plain FTP whenever possible.

---

## Quick Reference Card

```
Connect:    ftp <host>
Login:      user / pass
List:       ls
Get:        get <file>
Put:        put <file>
Binary:     binary
ASCII:      ascii
Quit:       bye
```