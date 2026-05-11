Command line packet analyser.

It captures and displays network traffic passing through a network interface in real time, 
letting you inspect packets at a low level.

# tcpdump Cheat Sheet

---

## Basic Syntax

```
tcpdump [options] [filter expression]
```

---

## Essential Options

|Flag|Description|
|---|---|
|`-i <iface>`|Listen on interface (e.g. `eth0`, `any`)|
|`-n`|Don't resolve hostnames|
|`-nn`|Don't resolve hostnames or port names|
|`-v` / `-vv` / `-vvv`|Increase verbosity|
|`-c <count>`|Capture N packets then stop|
|`-s <snaplen>`|Bytes to capture per packet (`0` = full packet)|
|`-q`|Quiet / minimal output|
|`-A`|Print packet payload as ASCII|
|`-X`|Print payload as hex + ASCII|
|`-e`|Print link-layer (Ethernet) headers|
|`-t`|Suppress timestamps|
|`-tttt`|Human-readable timestamps|
|`-D`|List available interfaces|

---

## Saving & Reading Files

```bash
# Write to a .pcap file
tcpdump -i eth0 -w capture.pcap

# Read from a .pcap file
tcpdump -r capture.pcap

# Rotate files every 100MB, keep 10 files
tcpdump -i eth0 -w capture_%Y%m%d_%H%M%S.pcap -G 3600 -C 100 -W 10
```

---

## Filter Expressions (BPF)

Filters are the core of tcpdump. Combine primitives with `and`, `or`, `not` (or `&&`, `||`, `!`).

### By Host

```bash
tcpdump host 192.168.1.1          # traffic to or from this host
tcpdump src host 10.0.0.5         # traffic FROM this host
tcpdump dst host 10.0.0.5         # traffic TO this host
tcpdump net 192.168.1.0/24        # entire subnet
```

### By Port

```bash
tcpdump port 80                   # traffic on port 80 (either direction)
tcpdump src port 443              # traffic FROM port 443
tcpdump dst port 22               # traffic TO port 22
tcpdump portrange 8000-9000       # port range
```

### By Protocol

```bash
tcpdump tcp
tcpdump udp
tcpdump icmp
tcpdump arp
tcpdump ip6
```

### Combining Filters

```bash
tcpdump host 10.0.0.1 and port 80
tcpdump src 10.0.0.1 and not port 22
tcpdump '(port 80 or port 443) and host 10.0.0.1'
```

> Wrap complex expressions in single quotes to prevent shell interpretation.

---

## Common Real-World Commands

```bash
# Capture all traffic on any interface, no DNS resolution
tcpdump -i any -nn

# Watch HTTP traffic (show headers/body)
tcpdump -i eth0 -A -s 0 port 80

# Capture DNS queries
tcpdump -i eth0 -nn port 53

# Capture HTTPS traffic (handshake only — payload is encrypted)
tcpdump -i eth0 port 443

# Monitor SSH connections
tcpdump -i eth0 tcp port 22

# Capture ICMP (ping) traffic
tcpdump -i eth0 icmp

# Find traffic between two specific hosts
tcpdump -i eth0 host 10.0.0.1 and host 10.0.0.2

# Capture everything except SSH (avoid locking yourself out)
tcpdump -i eth0 not port 22

# Show only TCP SYN packets (new connections)
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'

# Capture 100 packets and save to file
tcpdump -i eth0 -c 100 -w output.pcap

# Watch traffic from a subnet, skip DNS lookups
tcpdump -i eth0 -nn net 192.168.0.0/16
```

---

## TCP Flag Filters

TCP flags live at byte offset 13 in the TCP header.

|Filter|Meaning|
|---|---|
|`tcp[13] == 0x02`|SYN only|
|`tcp[13] == 0x12`|SYN-ACK|
|`tcp[13] == 0x01`|FIN only|
|`tcp[13] == 0x04`|RST only|
|`tcp[13] & 0x02 != 0`|Any packet with SYN set|
|`tcp[13] & 0x01 != 0`|Any packet with FIN set|

```bash
# Capture all TCP resets
tcpdump 'tcp[13] & 4 != 0'

# Capture new connections (SYN, not SYN-ACK)
tcpdump 'tcp[tcpflags] == tcp-syn'
```

---

## Reading Output

```
14:32:01.123456 IP 10.0.0.1.54321 > 93.184.216.34.80: Flags [S], seq 0, win 65535, length 0
│              │  │                  │                  │           │
│              │  │                  │                  │           └─ Payload size
│              │  │                  │                  └─ TCP flags [S]=SYN [.]=ACK [F]=FIN [R]=RST [P]=PUSH
│              │  │                  └─ Destination IP.port
│              │  └─ Source IP.port
│              └─ Protocol
└─ Timestamp
```

### TCP Flag Legend

|Flag|Meaning|
|---|---|
|`[S]`|SYN — connection initiation|
|`[.]`|ACK — acknowledgement|
|`[S.]`|SYN-ACK — connection accepted|
|`[P.]`|PSH-ACK — data push|
|`[F.]`|FIN-ACK — connection teardown|
|`[R.]`|RST — connection reset|

---

## Useful Tips

- **Always use `-nn`** in production to prevent reverse DNS from slowing capture and altering timing.
- **Use `-s 0`** to capture full packets; the default snaplen may truncate payloads.
- **Pipe to strings**: `tcpdump -A -s 0 port 80 | grep -i 'user-agent'`
- **Open `.pcap` files in Wireshark** for GUI-based deep inspection.
- **Combine with `grep`**: `tcpdump -l | grep 'pattern'` (`-l` enables line-buffered output for piping).
- On Linux, **`any`** as the interface captures on all interfaces simultaneously.

---

## Privilege & Security Notes

- tcpdump requires `root` or the `CAP_NET_RAW` capability.
- Running as root on a production server carries risk — prefer saving to `.pcap` and analysing offline.
- Use `tcpdump -Z <user>` to drop privileges after opening the interface.

---

_tcpdump uses libpcap under the hood — the same library used by Wireshark, nmap, and many other tools._