Server Message Block Protocol

Used for sharing access to files, printers, serial ports and other resources on a network

![[Pasted image 20260511093041.png]]

Microsoft Windows operating systems since Windows 95 have included client and server SMB protocol support. Samba, an open source server that supports the SMB protocol, was released for Unix systems.

## Enumerating SMB

First step is to do [[port]] scanning using tools like [[nmap]]

`Enum4Linux` is a tool used to enumerate SMB shares on both Windows and Linux systems

The syntax of Enum4Linux is nice and simple: **"enum4linux [options] ip"**

**TAG**            **FUNCTION**

-U             get userlist  
-M             get machine list  
-N             get namelist dump (different from -U and-M)  
-S             get sharelist  
-P             get password policy information  
-G             get group and member list

-a             all of the above (full basic enumeration)

## SMBClient

`Syntax: smbclient //[IP]/[SHARE] -U [USERNAME] -p [PORT]`
`Example: smbclient //10.10.10.10/secrets -U Anonymous -p 445`

Once inside the share, you can view the available commands by typing "help". The most useful of which are:

- **ls** or **dir**: List files and directories
- **[DIR]**: Move to a different directory
- **get [FILE]**: Download the file


