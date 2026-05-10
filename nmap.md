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