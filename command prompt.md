# Command Prompt Cheat Sheet

### Windows CMD vs Linux Comparison

---

## Navigation

|Description|Windows CMD|Linux|
|---|---|---|
|Change directory|`cd Documents`|`cd Documents`|
|Go up one level|`cd ..`|`cd ..`|
|Go to root|`cd \`|`cd /`|
|Go to home directory|`cd %USERPROFILE%`|`cd ~`|
|List files and folders|`dir`|`ls`|
|List including hidden files|`dir /a`|`ls -a`|
|List with details|`dir /q`|`ls -l`|
|List files in subdirectories|`dir /s`|`ls -R`|
|Display folder tree|`tree C:\Projects`|`tree ~/Projects`|
|Print current directory|`cd`|`pwd`|

---

## File & Folder Management

|Description|Windows CMD|Linux|
|---|---|---|
|Create a new folder|`mkdir NewFolder`|`mkdir NewFolder`|
|Remove an empty folder|`rmdir OldFolder`|`rmdir OldFolder`|
|Remove folder and all contents|`rmdir /s /q OldFolder`|`rm -rf OldFolder`|
|Copy a file|`copy file.txt D:\Backup`|`cp file.txt ~/Backup`|
|Copy folder recursively|`xcopy C:\Src D:\Dest /e`|`cp -r ~/Src ~/Dest`|
|Robust/advanced file copy|`robocopy C:\Src D:\Dest /mir`|`rsync -av ~/Src ~/Dest`|
|Move or rename a file|`move file.txt D:\Docs`|`mv file.txt ~/Docs`|
|Delete a file|`del file.txt`|`rm file.txt`|
|Force delete without prompt|`del /f /q file.txt`|`rm -f file.txt`|
|Rename a file or folder|`ren old.txt new.txt`|`mv old.txt new.txt`|
|Display file contents|`type notes.txt`|`cat notes.txt`|
|Display file page by page|`more notes.txt`|`less notes.txt`|
|Write text to a file|`echo Hello > hello.txt`|`echo Hello > hello.txt`|
|Append text to a file|`echo World >> hello.txt`|`echo World >> hello.txt`|
|Search text in a file|`find "word" file.txt`|`grep "word" file.txt`|
|Search recursively|`findstr /s "word" *.txt`|`grep -r "word" .`|
|Create empty file|`type nul > file.txt`|`touch file.txt`|
|Show file/folder size|`dir file.txt`|`du -sh file.txt`|

---

## System Information

|Description|Windows CMD|Linux|
|---|---|---|
|OS and hardware info|`systeminfo`|`uname -a` / `hostnamectl`|
|Display computer name|`hostname`|`hostname`|
|Display current user|`whoami`|`whoami`|
|Display OS version|`ver`|`cat /etc/os-release`|
|CPU info|`wmic cpu get name`|`lscpu`|
|RAM info|`wmic memorychip get capacity`|`free -h`|
|Disk usage|`wmic logicaldisk get size,freespace`|`df -h`|
|Uptime|`systeminfo \| find "Boot Time"`|`uptime`|
|Hardware details|`msinfo32` (GUI)|`lshw`|

---

## Process & Task Management

|Description|Windows CMD|Linux|
|---|---|---|
|List running processes|`tasklist`|`ps aux`|
|Filter by process name|`tasklist /fi "imagename eq chrome.exe"`|`ps aux \| grep chrome`|
|Kill process by name|`taskkill /im notepad.exe /f`|`pkill notepad`|
|Kill process by PID|`taskkill /pid 1234 /f`|`kill -9 1234`|
|Interactive process viewer|_(Task Manager GUI)_|`top` / `htop`|
|Launch a program|`start notepad.exe`|`./program`|
|Run in background|`start /b command`|`command &`|
|List open files by process|_(use Sysinternals)_|`lsof -p <pid>`|

---

## Networking

|Description|Windows CMD|Linux|
|---|---|---|
|Show IP configuration|`ipconfig`|`ip addr` / `ifconfig`|
|Show full network details|`ipconfig /all`|`ip addr show`|
|Flush DNS cache|`ipconfig /flushdns`|`sudo systemd-resolve --flush-caches`|
|Release DHCP IP|`ipconfig /release`|`sudo dhclient -r`|
|Renew DHCP IP|`ipconfig /renew`|`sudo dhclient`|
|Test connectivity|`ping google.com`|`ping google.com`|
|Ping continuously|`ping -t 8.8.8.8`|`ping 8.8.8.8` _(continuous by default)_|
|Trace route to host|`tracert google.com`|`traceroute google.com`|
|DNS lookup|`nslookup google.com`|`nslookup google.com` / `dig google.com`|
|Show active connections|`netstat -an`|`netstat -an` / `ss -an`|
|Show connections with process|`netstat -b`|`netstat -tulnp`|
|Show ARP table|`arp -a`|`arp -a`|
|Download a file|_(use PowerShell's Invoke-WebRequest)_|`wget <url>` / `curl -O <url>`|

---

## Disk Management

|Description|Windows CMD|Linux|
|---|---|---|
|Check and fix disk errors|`chkdsk C: /f`|`fsck /dev/sda1`|
|Show disk partitions|`diskpart` (interactive)|`fdisk -l` / `lsblk`|
|Show disk usage|`wmic logicaldisk get freespace`|`df -h`|
|Show folder size|`dir /s folder`|`du -sh folder`|
|Mount a drive|_(auto via Windows)_|`mount /dev/sdb1 /mnt`|
|Unmount a drive|_(use Disk Management GUI)_|`umount /mnt`|
|Format a drive|`format D: /fs:NTFS`|`mkfs.ext4 /dev/sdb1`|
|Scan and repair system files|`sfc /scannow`|`sudo apt --fix-broken install`|

---

## User & Permissions

|Description|Windows CMD|Linux|
|---|---|---|
|List user accounts|`net user`|`getent passwd`|
|Create a user|`net user john pass123 /add`|`sudo useradd -m john`|
|Delete a user|`net user john /delete`|`sudo userdel -r john`|
|Add user to Admins/sudo|`net localgroup administrators john /add`|`sudo usermod -aG sudo john`|
|Change user password|`net user john newpass`|`sudo passwd john`|
|Show file permissions|`icacls C:\folder`|`ls -l`|
|Change file permissions|`icacls file /grant User:F`|`chmod 755 file`|
|Change file owner|`takeown /f file.txt`|`chown user:group file.txt`|
|Run as another user|`runas /user:admin cmd`|`sudo -u admin command`|
|Run as root/admin|_(Run as Administrator)_|`sudo command`|
|Switch user|_(open new session)_|`su - username`|

---

## Environment & Variables

|Description|Windows CMD|Linux|
|---|---|---|
|List all environment variables|`set`|`env` / `printenv`|
|Set a variable (session only)|`set MYVAR=hello`|`export MYVAR=hello`|
|Print a variable's value|`echo %PATH%`|`echo $PATH`|
|Set a persistent variable|`setx MYVAR hello`|Add to `~/.bashrc` or `~/.profile`|
|Display the PATH variable|`path`|`echo $PATH`|
|Unset a variable|_(not supported in CMD)_|`unset MYVAR`|

---

## Output & Redirection

| Operator        | Description                 | Windows Example      | Linux Example           |
| --------------- | --------------------------- | -------------------- | ----------------------- |
| `>`             | Redirect output (overwrite) | `dir > list.txt`     | `ls > list.txt`         |
| `>>`            | Redirect output (append)    | `dir >> list.txt`    | `ls >> list.txt`        |
| `\|`            | Pipe to another command     | `dir \| find "txt"`  | `ls \| grep txt`        |
| `<`             | Use file as input           | `sort < names.txt`   | `sort < names.txt`      |
| `2>`            | Redirect error output       | `dir 2> errors.txt`  | `ls 2> errors.txt`      |
| `2>&1`          | Redirect errors to stdout   | `dir > out.txt 2>&1` | `ls > out.txt 2>&1`     |
| Suppress output | Discard all output          | `ping host > nul`    | `ping host > /dev/null` |
|                 |                             |                      |                         |

---

## Useful Shortcuts & Tips

|Description|Windows CMD|Linux|
|---|---|---|
|Clear the screen|`cls`|`clear`|
|Close the terminal|`exit`|`exit`|
|Show help for a command|`<command> /?`|`man <command>` / `<command> --help`|
|View command history|`F7` / `doskey /history`|`history`|
|Scroll through history|`↑ / ↓`|`↑ / ↓`|
|Auto-complete|`Tab`|`Tab`|
|Cancel running command|`Ctrl + C`|`Ctrl + C`|
|Pause output|`Ctrl + S`|`Ctrl + S`|
|Resume output|`Ctrl + Q`|`Ctrl + Q`|
|Run commands sequentially|`cmd1 & cmd2`|`cmd1; cmd2`|
|Run next only if success|`cmd1 && cmd2`|`cmd1 && cmd2`|
|Run next only if failure|_(not standard in CMD)_|`cmd1 \| cmd2`|

---

## Scripting Basics Comparison

**Windows Batch (`.bat`)**

```bat
@echo off              :: Suppress command echoing
REM This is a comment
set NAME=World
echo Hello %NAME%!
pause                  :: Wait for keypress
if "%1"=="" goto end   :: Conditional
for %%f in (*.txt) do echo %%f   :: Loop over files
:end
```

**Linux Bash (`.sh`)**

```bash
#!/bin/bash            # Shebang line
# This is a comment
NAME="World"
echo "Hello $NAME!"
read -p "Press enter to continue"
if [ -z "$1" ]; then   # Conditional
  exit 0
fi
for f in *.txt; do echo "$f"; done  # Loop over files
```

---

> **Windows tip:** Run Command Prompt as Administrator for privileged commands (`sfc`, `chkdsk`, `net user`, etc.).

> **Linux tip:** Prefix commands with `sudo` for root-level access. Use `man <command>` for full documentation on any command.