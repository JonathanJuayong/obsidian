Domain Name System

Its main purpose is to map [[IP Address]] to [[Domain Names]]

When a user makes a request using a domain name such as [tryhackme.com](http://tryhackme.com), DNS 'translates' this to its IP address then ultimately supplies the requester with the correct IP address. It's also worth noting that in some scenarios the IP address returned by DNS won't always be the origin server's IP address If [[DNS records]] are being proxied by a service such as Cloudflare's [[DDoS]] protection.

![[Pasted image 20260510171946.png]]

### The DNS Hierarchy

DNS is a distributed, hierarchical system with four levels:

```
Root (.)
  └── TLD (.com, .org, .au)
        └── Authoritative (google.com)
              └── Subdomain (mail.google.com)
```

**Root Servers** — There are 13 sets of root servers worldwide (labeled A–M). They don't know where `google.com` is, but they know who manages `.com`.

**TLD (Top-Level Domain) Servers** — Managed by registries (e.g., Verisign handles `.com`). They know which nameserver is authoritative for `google.com`.

**Authoritative Nameservers** — The final authority for a domain. They hold the actual DNS records (e.g., "google.com → 142.250.80.46").

**Recursive Resolver** — Usually run by your ISP or a public service like Google (8.8.8.8) or Cloudflare (1.1.1.1). It does the legwork of querying the hierarchy on your behalf.

---

### How a DNS Lookup Works (Step by Step)

When you type `www.example.com` in your browser:

1. **Browser cache** — Checks if it already knows the IP from a recent visit.
2. **OS cache** — Checks the local system cache and `/etc/hosts` file.
3. **Recursive Resolver** — Your device asks your configured DNS resolver (e.g., 1.1.1.1).
4. **Root Server** — The resolver asks a root server: _"Who handles .com?"_ → gets TLD server address.
5. **TLD Server** — The resolver asks the `.com` TLD server: _"Who handles example.com?"_ → gets authoritative nameserver address.
6. **Authoritative Server** — The resolver asks example.com's nameserver: _"What's the IP for [www.example.com](http://www.example.com)?"_ → gets the answer.
7. **Response returned** — The resolver caches the result and returns the IP to your device.
8. **Browser connects** — Your browser opens a TCP connection to that IP.

This entire process typically takes **20–120ms**, often much less due to caching.