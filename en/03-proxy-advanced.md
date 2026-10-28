# 03. Advanced Proxy Features & Traffic Manipulation Rules

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/03-proxy-advanced.md`)](../ar/03-proxy-advanced.md)

---

## 🎯 Chapter Objectives

Elevate proxy usage beyond basic request inspection by mastering conditional interception rules (`Intercept Rules`), configuring automated header and body rewriting with `Match & Replace`, and chaining Burp traffic through upstream proxies, enterprise bastions, and the **Tor Network** via `SOCKS5` proxies for anonymity and network pivoting.

---

## 🔍 Technical Explanation

### 1. Conditional Interception Mechanics:
Manually forwarding every image, font, and script is inefficient and degrades testing workflow. Burp's interception engine can evaluate predicates on every transaction:
* Intercepting specific verbs and content types (`POST`, `PUT`, `application/json`).
* Restricting interception strictly to in-scope items.
* **Response Interception:** Halting upstream server responses before they reach the client browser. This is essential for bypassing client-side access controls, injecting script hooks, and rewriting response flags (e.g., flipping `{"isAdmin": false}` to `{"isAdmin": true}`).

### 2. The Match & Replace Engine:
Operating as an automated regex pipeline, **Match & Replace** automatically alters incoming requests and outgoing responses on the wire without human intervention:
* Injecting custom headers across every outbound request (e.g., simulating IP spoofing via `X-Forwarded-For: 127.0.0.1`).
* Stripping client-side restrictions from downloaded scripts (removing debugger traps, anti-tampering logic, or input length restrictions).
* Neutralizing defensive security headers in responses (e.g., stripping `Content-Security-Policy` or `X-Frame-Options` to facilitate proof-of-concept Clickjacking or DOM XSS audits).

### 3. Upstream Routing and SOCKS Integration (Tor Pivoting):
In enterprise assessments and red teaming, testers often need to pivot through jump boxes or anonymize egress IP addresses:
* **Upstream HTTP Proxy:** Relays Burp traffic through corporate gateways requiring NTLM/Kerberos authentication.
* **SOCKS5 Proxy (Tor Network):** Routes TCP connections through a local Tor daemon listening on `127.0.0.1:9050` or `9150`.
* **DNS Resolution over SOCKS:** Prevents **DNS Leaks** by forcing the remote proxy endpoint to resolve hostnames rather than utilizing local host resolvers.

---

## 🛠️ Step-by-Step Hands-On Guide

### Step 1: Configuring Intelligent Intercept Predicates
1. Navigate to `Settings > Tools > Proxy`.
2. Under **Request interception rules**:
   * Enable: `And URL Is in-scope` to ensure out-of-scope browsing is never halted.
   * Verify the file exclusion rule: `And File extension Does not match: png|jpg|gif|css|js|woff2`.
3. Under **Response interception rules**:
   * Check: `Intercept responses based on the following rules`.
   * Add a condition: `And URL Is in-scope`.

### Step 2: Client-Side Control Bypass via Match & Replace
1. Go to `Settings > Tools > Proxy > Match and replace rules`.
2. Click **Add** and configure the following rules:
   * **Un-disabling UI Elements:**
     * *Type:* `Response body`
     * *Match:* `disabled=""`
     * *Replace:* (Leave empty to enable all disabled submit buttons)
   * **Simulating Internal Origin (IP Spoofing):**
     * *Type:* `Request header`
     * *Match:* (Leave empty to append a new header)
     * *Replace:* `X-Forwarded-For: 127.0.0.1`

### Step 3: Chaining Burp Through Tor via SOCKS5
1. Launch the local Tor service or Tor Browser (`127.0.0.1:9150` for Browser, `9050` for service).
2. Inside Burp, navigate to `Settings > Network > Connections`.
3. Scroll down to the **SOCKS proxy** panel:
   * **SOCKS proxy host:** `127.0.0.1`
   * **SOCKS proxy port:** `9050` (or `9150`)
   * **Mandatory Security Checkbox:** Enable **Do DNS lookups over SOCKS proxy**.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: 2FA broken logic](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)
  * Intercept and manipulate authentication responses and cookies to bypass multi-factor security barriers.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Match & Replace offers automated stealth. Testers can maintain consistent header injection, user-agent spoofing, or session token overrides across complex multi-step navigation flows without manual edits.
* **🛡️ Defender Perspective:**
  * Confirms that reverse proxy trust must be configured rigorously. Web servers that blindly evaluate client-controlled forwarding headers (`X-Forwarded-Host`, `X-Original-URL`) expose internal administrative endpoints to trivial bypasses.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Unmonitored Interception State:** Forgetting that `Intercept is ON`, resulting in a frozen browser experience while Burp silently holds an unforwarded request.
> 2. **DNS Leakage during SOCKS Pivoting:** Routing HTTP payloads over SOCKS5 while leaving local DNS queries unproxied, immediately disclosing assessment targets to local ISPs.
> 3. **Overly Broad Match & Replace Regex:** Crafting overly generic replacement rules that corrupt application JavaScript syntax, rendering complex SPAs completely unresponsive.

---

## 📚 References

1. [PortSwigger Documentation: Proxy Intercept Options](https://portswigger.net/burp/documentation/desktop/tools/proxy/intercept)
2. [PortSwigger Documentation: Match and Replace Rules](https://portswigger.net/burp/documentation/desktop/tools/proxy/http-matching-replacing)
3. [The Tor Project: SOCKS Proxy Specification](https://tb-manual.torproject.org/)

---

| [← Previous: 02. Dashboard & Scope](./02-dashboard-target-scope.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 04. Repeater & Parallel Attacks →](./04-repeater.md) |
