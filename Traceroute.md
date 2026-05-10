Traces the route taken by the packets from your system to another host. Main purpose is to figure out the IP addresses between hops

```bash
traceroute domainname.com
```
`traceroute` uses [[TTL]] to determine how many hops it would take to reach the final destination. For every hop a packet does, the [[TTL]] is decremented by one. Once it reaches 0, the last hop will send and [[ICMP]] echo back to the original sender with a [[TTL]] exceeded error.

By initially using a [[TTL]] of 1, `traceroute` can immediately identify the [[IP Address]] of the next hop. `traceroute` will then increment [[TTL]] to 2 on the next request and so on until it reaches the final destination.
