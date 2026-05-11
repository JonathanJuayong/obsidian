- Network Mapper
- industry standard for scanning and mapping networks
- nmap use the following to map out the network:
	- [[ARP Scan]]
	- [[ICMP Scan]]
	- [[TCP/UDP Scan]]
```bash
$: nmap -sL TARGETS
```

This command provides a detailed list of all the hosts nmap will scan without actually scanning them

```bash
$: nmap -iL list_of_hosts.txt
```
This command allows nmap to scan a list of hosts saved in a txt file

![[Pasted image 20260504182947.png]]
![[Pasted image 20260504183110.png]]

# nmap Cheat Sheet

## Basic Syntax

```
nmap [OPTIONS] TARGET [TARGET...]
```

Targets can be hostnames, IPs, CIDR ranges, or ranges like `192.168.1.1-50`.

---

## Target Specification

```bash
nmap 192.168.1.1                  # Single IP
nmap 192.168.1.1-254              # IP range
nmap 192.168.1.0/24               # CIDR subnet
nmap 10.0.0.1 10.0.0.2           # Multiple hosts
nmap scanme.nmap.org              # Hostname
nmap -iL targets.txt              # Read targets from file
nmap --exclude 192.168.1.5        # Exclude a host
nmap --excludefile skip.txt       # Exclude hosts from file
```

---

## Scan Types

| Option            | Scan Type         | Description                                           |
| ----------------- | ----------------- | ----------------------------------------------------- |
| `-sS`             | TCP SYN (stealth) | Half-open scan; fast and stealthy (default with root) |
| `-sT`             | TCP Connect       | Full 3-way handshake; no root needed                  |
| `-sU`             | UDP               | Scan UDP ports; slow but important                    |
| `-sA`             | TCP ACK           | Map firewall rules; doesn't determine open/closed     |
| `-sN`             | TCP Null          | No flags set; evades some firewalls                   |
| `-sF`             | TCP FIN           | FIN flag only                                         |
| `-sX`             | Xmas              | FIN+PSH+URG flags                                     |
| `-sW`             | TCP Window        | Like ACK but uses window size                         |
| `-sM`             | Maimon            | FIN+ACK probe                                         |
| `-sI zombie:port` | Idle (zombie)     | Truly blind scan via zombie host                      |
| `-sO`             | IP Protocol       | Scan for supported IP protocols                       |
| `-sn`             | Ping scan         | Host discovery only, no port scan                     |
| `-sV`             | Version detection | Probe open ports for service/version                  |
| `-sC`             | Default scripts   | Run default NSE scripts                               |
| `-sL`             | List scan         | List targets only, no scanning                        |

---

## Port Specification

```bash
nmap -p 22                        # Single port
nmap -p 22,80,443                 # Multiple ports
nmap -p 1-1024                    # Port range
nmap -p-                          # All 65535 ports
nmap -p U:53,T:80                 # UDP 53 and TCP 80
nmap --top-ports 100              # Scan top 100 most common ports
nmap --top-ports 1000             # Scan top 1000 most common ports
nmap -F                           # Fast scan (top 100 ports)
nmap -r                           # Scan ports in sequential order
```

---

## Host Discovery

```bash
nmap -sn 192.168.1.0/24          # Ping sweep (no port scan)
nmap -Pn                          # Skip host discovery (treat all as up)
nmap -PS22,80,443                 # TCP SYN ping on specific ports
nmap -PA80,443                    # TCP ACK ping
nmap -PU53                        # UDP ping
nmap -PE                          # ICMP echo ping
nmap -PP                          # ICMP timestamp ping
nmap -PM                          # ICMP netmask ping
nmap -PR                          # ARP ping (LAN only)
nmap -n                           # Never do DNS resolution
nmap -R                           # Always do DNS resolution
nmap --dns-servers 8.8.8.8        # Use specific DNS server
```

---

## Service & Version Detection

```bash
nmap -sV target                   # Detect service versions
nmap -sV --version-intensity 5    # Intensity 0–9 (default 7)
nmap -sV --version-light          # Fewer probes (faster, less accurate)
nmap -sV --version-all            # Max probes (slower, more accurate)
nmap -A                           # Aggressive: -sV -sC -O --traceroute
```

---

## OS Detection

```bash
nmap -O target                    # Enable OS detection
nmap -O --osscan-guess            # Guess OS more aggressively
nmap -O --osscan-limit            # Limit OS detection to likely hosts
nmap -A target                    # OS + version + scripts + traceroute
```

---

## Timing & Performance

|Option|Name|Description|
|---|---|---|
|`-T0`|Paranoid|Very slow; IDS evasion|
|`-T1`|Sneaky|Slow; IDS evasion|
|`-T2`|Polite|Slower; less bandwidth|
|`-T3`|Normal|Default|
|`-T4`|Aggressive|Faster; assumes reliable network|
|`-T5`|Insane|Very fast; may miss results|

```bash
nmap -T4 --min-rate 1000         # At least 1000 packets/sec
nmap --max-retries 1             # Reduce retries for speed
nmap --host-timeout 30s          # Abandon hosts after 30 seconds
nmap --scan-delay 1s             # Delay between probes
nmap --max-parallelism 10        # Limit parallel probes
```

---

## NSE Scripts

```bash
nmap -sC target                   # Run default scripts
nmap --script=banner target       # Run a specific script
nmap --script=http-title target   # Get HTTP page titles
nmap --script=vuln target         # Run all vuln scripts
nmap --script=safe target         # Run all safe scripts
nmap --script=exploit target      # Run exploit scripts
nmap --script=auth target         # Test authentication
nmap --script=brute target        # Run brute-force scripts
nmap --script=discovery target    # Run discovery scripts

# Script with arguments
nmap --script=http-brute --script-args userdb=users.txt,passdb=pass.txt target

# Update script database
nmap --script-updatedb
```

### Useful Scripts

```bash
nmap --script=http-title -p 80,443,8080 target
nmap --script=ssl-cert -p 443 target
nmap --script=ssl-enum-ciphers -p 443 target
nmap --script=ftp-anon -p 21 target
nmap --script=smb-vuln-ms17-010 -p 445 target   # EternalBlue
nmap --script=smb-enum-shares -p 445 target
nmap --script=dns-zone-transfer --script-args dns-zone-transfer.domain=example.com -p 53 target
nmap --script=http-robots.txt -p 80 target
nmap --script=mysql-empty-password -p 3306 target
nmap --script=ssh-hostkey -p 22 target
```

---

## Firewall / IDS Evasion

```bash
nmap -f target                    # Fragment packets (8-byte frags)
nmap -ff target                   # 16-byte fragments
nmap --mtu 24 target              # Custom MTU (must be multiple of 8)
nmap -D RND:10 target             # Decoy scan with 10 random decoys
nmap -D 1.2.3.4,5.6.7.8,ME target # Named decoys (ME = real IP)
nmap -S 1.2.3.4 target            # Spoof source IP
nmap -e eth0 target               # Use specific network interface
nmap --source-port 53 target      # Spoof source port
nmap -g 53 target                 # Same as --source-port
nmap --data-length 25 target      # Append random data to packets
nmap --randomize-hosts            # Randomize scan order
nmap --proxies socks4://127.0.0.1:9050 target  # Scan via proxy
nmap --badsum target              # Send packets with bad checksums (test IDS)
nmap -sI zombie:port target       # Idle scan using zombie host
```

---

## Output Formats

```bash
nmap -oN output.txt target        # Normal text output
nmap -oX output.xml target        # XML output
nmap -oG output.gnmap target      # Grepable output
nmap -oA output target            # All three formats (output.{nmap,xml,gnmap})
nmap -oS output.txt target        # Script kiddie output (leet speak)
nmap -v target                    # Verbose output
nmap -vv target                   # More verbose
nmap --open target                # Show only open ports
nmap --reason target              # Show reason for port state
nmap --packet-trace target        # Show all packets sent/received
nmap --iflist                     # List interfaces and routes
```

---

## IPv6

```bash
nmap -6 ::1                       # Scan IPv6 target
nmap -6 -sV fe80::1%eth0          # Version scan IPv6
```

---

## Common Combinations

```bash
# Quick scan of common ports
nmap -T4 -F target

# Full port scan with version detection
nmap -p- -sV -T4 target

# Comprehensive scan
nmap -A -T4 target

# Stealth SYN scan, all ports, version, scripts
nmap -sS -sV -sC -p- -T4 target

# UDP + TCP scan
nmap -sS -sU -T4 target

# Ping sweep a subnet
nmap -sn 192.168.1.0/24

# Scan with OS detection, no ping, all ports
nmap -O -Pn -p- target

# Vulnerability scan
nmap -sV --script=vuln target

# SMB vulnerability check
nmap -p 445 --script=smb-vuln-* target

# Web server fingerprinting
nmap -p 80,443,8080,8443 -sV --script=http-headers,http-title target

# Save all output formats
nmap -A -T4 -oA myscan target
```

---

## Port States

|State|Meaning|
|---|---|
|`open`|Port is accepting connections|
|`closed`|Port is accessible but no service listening|
|`filtered`|Firewall is blocking the probe; state unknown|
|`unfiltered`|Port is accessible but open/closed unknown (ACK scan)|
|`open\|filtered`|Can't tell if open or filtered (UDP, Null, FIN, Xmas)|
|`closed\|filtered`|Can't tell if closed or filtered (Idle scan)|

---

## Tips

- Use `-Pn` when hosts block ICMP but have open ports
- Combine `-sS -sU` to scan both TCP and UDP in one run
- `-oA` is good practice — saves all formats for later analysis
- Use `--open` to cut noise and focus on actionable results
- `nmap -sV --version-intensity 0` is faster and often sufficient
- Run as root for SYN scans (`-sS`); otherwise falls back to `-sT`
- `nmap --script-help <script>` shows docs for any NSE script
- `locate *.nse` or `ls /usr/share/nmap/scripts/` to browse all scripts