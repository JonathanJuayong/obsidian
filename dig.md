Domain Information Groper

# `dig` Cheat Sheet

> **dig** (Domain Information Groper) — a flexible DNS lookup utility.

---

## Basic Syntax

```
dig [@server] [name] [type] [options]
```

---

## Common Lookups

|Command|Description|
|---|---|
|`dig example.com`|Default lookup (A record)|
|`dig example.com A`|IPv4 address|
|`dig example.com AAAA`|IPv6 address|
|`dig example.com MX`|Mail exchange records|
|`dig example.com NS`|Name servers|
|`dig example.com TXT`|TXT records (SPF, DKIM, etc.)|
|`dig example.com CNAME`|Canonical name record|
|`dig example.com SOA`|Start of Authority|
|`dig example.com PTR`|Pointer record|
|`dig example.com SRV`|Service locator|
|`dig example.com CAA`|Certification Authority Authorization|
|`dig example.com ANY`|All available records|

---

## Reverse DNS Lookup

```bash
dig -x 8.8.8.8                    # Reverse lookup for an IP
dig -x 2001:4860:4860::8888       # Reverse lookup for IPv6
```

---

## Specifying a DNS Server

```bash
dig @8.8.8.8 example.com          # Query Google's DNS
dig @1.1.1.1 example.com          # Query Cloudflare's DNS
dig @ns1.example.com example.com  # Query a specific nameserver
```

---

## Output Control

|Flag|Description|
|---|---|
|`+short`|Minimal output (answer only)|
|`+noall +answer`|Show only the answer section|
|`+nocmd`|Suppress the initial comment|
|`+nocomments`|Suppress comment lines|
|`+noauthority`|Hide authority section|
|`+noadditional`|Hide additional section|
|`+nostats`|Hide stats footer|
|`+multiline`|Print records in multiline format|
|`+all`|Show everything|

```bash
dig example.com +short            # e.g. → 93.184.216.34
dig example.com MX +noall +answer # Clean MX answer only
```

---

## Tracing & Debugging

```bash
dig example.com +trace            # Trace the full DNS resolution path
dig example.com +dnssec           # Request DNSSEC records
dig example.com +cd               # Disable DNSSEC validation
dig example.com +cdflag           # Set checking disabled flag
dig example.com +time=5           # Set query timeout (seconds)
dig example.com +tries=2          # Set number of retries
dig example.com +tcp              # Force TCP instead of UDP
```

---

## Batch / Multiple Queries

```bash
# Query multiple types at once
dig example.com A example.com MX

# Read queries from a file
dig -f queries.txt

# queries.txt format:
# example.com A
# example.com MX
```

---

## Port & Protocol

```bash
dig example.com -p 5353           # Query on a custom port (e.g. mDNS)
dig example.com +vc               # Use TCP (virtual circuit)
dig example.com +tls @1.1.1.1    # DNS over TLS (DoT)
```

---

## TTL & Caching

```bash
dig example.com +ttlid            # Show TTL as a number
dig example.com +ttlunits         # Show TTL with units (e.g. 5m)
dig example.com +expire           # Show zone expiry in OPT record
```

---

## EDNS Options

```bash
dig example.com +edns=0           # Set EDNS version
dig example.com +noedns           # Disable EDNS
dig example.com +bufsize=1232     # Set EDNS buffer size
dig example.com +subnet=192.0.2.0/24  # Send client subnet
```

---

## Useful One-Liners

```bash
# Check SPF record
dig example.com TXT +short | grep spf

# Find all nameservers for a domain
dig example.com NS +short

# Check if a domain has DMARC
dig _dmarc.example.com TXT +short

# Get DKIM public key
dig selector._domainkey.example.com TXT +short

# Lookup with timing info
dig example.com | grep "Query time"

# Who is authoritative for a domain?
dig example.com NS +noall +answer +authority
```

---

## Output Sections Explained

```
;; QUESTION SECTION   → What was asked
;; ANSWER SECTION     → Direct answers
;; AUTHORITY SECTION  → Authoritative nameservers
;; ADDITIONAL SECTION → Extra helpful records
;; Query time         → Round-trip time in ms
;; SERVER             → DNS server that responded
```

---

## Record Types Reference

|Type|Full Name|Use|
|---|---|---|
|`A`|Address|IPv4 address|
|`AAAA`|Quad-A|IPv6 address|
|`CNAME`|Canonical Name|Alias to another name|
|`MX`|Mail Exchange|Email routing|
|`NS`|Name Server|Authoritative DNS servers|
|`PTR`|Pointer|Reverse DNS|
|`SOA`|Start of Authority|Zone metadata|
|`TXT`|Text|Arbitrary text (SPF, DKIM…)|
|`SRV`|Service|Service location|
|`CAA`|CA Authorization|SSL cert issuance control|
|`DS`|Delegation Signer|DNSSEC chain of trust|
|`DNSKEY`|DNS Key|DNSSEC public key|

---

_`man dig` or `dig -h` for the full reference._