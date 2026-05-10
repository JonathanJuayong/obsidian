Find the [[DNS Records]] of a domain
``` bash
nslookup OPTIONS DOMAIN_NAME SERVER
```

## Basic Queries`

| Command                                | Description                           | Record Type |     |
| -------------------------------------- | ------------------------------------- | ----------- | --- |
| `nslookup example.com`                 | Default lookup (IPv4)                 | A           | ``  |
| `nslookup -type=AAAA example.com`      | IPv6 address lookup                   | AAAA        | ``  |
| `nslookup -type=MX example.com`        | Mail server records                   | MX          | ``  |
| `nslookup -type=NS example.com`        | Authoritative nameservers             | NS          | ``  |
| `nslookup -type=TXT example.com`       | TXT records (SPF, DKIM, verification) | TXT         | ``  |
| `nslookup -type=CNAME sub.example.com` | Canonical name alias                  | CNAME       | ``  |
| `nslookup -type=SOA example.com`       | Start of authority (zone info)        | SOA         | ``  |
|`nslookup -type=ANY example.com`       | All record types (may be filtered)    | ANY         | ``  |

## Reverse & Server Options

| Command                            | Description                          |
| ---------------------------------- | ------------------------------------ |
| `nslookup 93.184.216.34`           | Reverse lookup — IP → hostname       |
| `nslookup example.com 8.8.8.8`     | Query a specific DNS server (Google) |
| `nslookup example.com 1.1.1.1`     | Query Cloudflare's resolver          |
| `nslookup -debug example.com`      | Show full response packets           |
| `nslookup -timeout=10 example.com` | Set query timeout in seconds         |


## Interactive Mode

Launch with `nslookup` (no arguments), then use these commands:

| Command          | Description                   |
| ---------------- | ----------------------------- |
| `server 8.8.8.8` | Switch DNS server mid-session |
| `set type=MX`    | Change query type             |
| `set debug`      | Enable debug output           |
| `set all`        | Show current session settings |
| `exit`           | Quit interactive mode         |



> **Tip:** On Linux/macOS, prefer `dig` for scripting — it gives more control and parseable output. `nslookup` is universal and great for quick checks anywhere.