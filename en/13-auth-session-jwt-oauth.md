# 13. Authentication, Session Management, JWT & OAuth Testing

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/13-auth-session-jwt-oauth.md`)](../ar/13-auth-session-jwt-oauth.md)

---

## 🎯 Chapter Objectives

Deconstruct and audit modern web authentication schemes and session state mechanisms with Burp Suite, manipulate and exploit **JSON Web Tokens (JWT)** utilizing the `JWT Editor` extension, bypass **Two-Factor Authentication (2FA/MFA)** controls, and exploit security flaws across **OAuth 2.0** authorization flows.

---

## 🔍 Technical Explanation

### 1. Anatomy of JWT Vulnerabilities & Signature Forgery:
A standard JSON Web Token consists of three base64url-encoded components separated by dots (`Header.Payload.Signature`):
* **The `none` Algorithm Flaw:** The JWT standard defines an unsecured token algorithm named `none`. Vulnerable libraries accept tokens where the algorithm header is explicitly changed to `{"alg": "none"}` and the cryptographic signature segment is stripped. If accepted, an attacker can mutate claim data (e.g., flipping `"role": "user"` to `"role": "admin"`), obtaining instant privilege escalation.
* **Algorithm Confusion (Asymmetric to Symmetric / RS256 vs HS256):** Occurs when a server expects an asymmetric key pair (`RS256`), but the library blindly accepts symmetric HMAC (`HS256`). An attacker re-signs the tampered token using the server's public RSA key (which is publicly accessible) as the HMAC secret key. When the server validates the token using its public key, the mathematical validation succeeds.
* **Header Parameter Injections (JWK / JKU):** Attackers embed their own generated public key directly inside the `jwk` header parameter or point the `jku` (JSON Web Key Set URL) parameter to an attacker-controlled endpoint (or Burp Collaborator) to force the server to verify the token against untrusted keys.

### 2. Two-Factor Authentication (2FA) Bypass Mechanics:
* **Broken Verification Logic:** Exploiting unthrottled numeric verification code endpoints via high-speed `Intruder` attacks, or manipulating session cookies in Proxy to uncouple the active user identity from the 2FA state machine.
* **Forced Browsing to Protected Origins:** Direct navigation to post-authentication endpoints (`/account/overview`) immediately after submitting valid primary credentials, bypassing the 2FA submission screen if the server sets authenticated session cookies prematurely.

### 3. OAuth 2.0 Authorization Flow Vulnerabilities:
* **Authorization Code Redirection Hijacking:** Tampering with the `redirect_uri` parameter during the authorization handshake to force identity providers to leak authorization codes or bearer tokens to an attacker-controlled endpoint.
* **Omission of the State Parameter (OAuth CSRF):** Failing to implement a dynamic, cryptographically bound `state` parameter allows attackers to bind their own social identities to victim sessions via CSRF.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Exploiting the JWT `none` Algorithm
1. Intercept an authenticated API transaction in **Repeater** bearing a Bearer JWT:
   ```http
   GET /api/v1/admin/users HTTP/1.1
   Host: target-app.com
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidmljdGltIiwiaXNBZG1pbiI6ZmFsc2V9.signature
   ```
2. Paste the Header into **Decoder**, decode as Base64, and alter the algorithm:
   ```json
   {
     "alg": "none",
     "typ": "JWT"
   }
   ```
3. Decode and modify the Payload claims:
   ```json
   {
     "user": "administrator",
     "isAdmin": true
   }
   ```
4. Re-encode both JSON objects via **Base64-URL**, join them with a dot, and remove the signature while retaining the trailing dot:
   `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW5pc3RyYXRvciIsImlzQWRtaW4iOnRydWV9.`
5. Dispatch the request in Repeater; an HTTP `200 OK` exposing administrative user lists confirms unverified signature execution.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: JWT authentication bypass via unverified signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)
  * Practice crafting unsigned JWT tokens in Repeater to bypass authorization and access privileged admin panels.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Flaws in authentication logic provide maximum impact with minimal detection risk. Forging a JWT token grants persistent, legitimate-looking administrative access that bypasses IDS/IPS heuristic alarms.
* **🛡️ Defender Perspective:**
  * Authentication libraries must reject the `none` algorithm unconditionally, enforce strict key whitelisting, verify signature timestamps (`exp`, `nbf`), and require exact URI matching for OAuth callback endpoints.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Omitting the trailing period in unsigned JWTs:** A JWT structurally requires two delimiter periods (`header.payload.`); omitting the second period renders the token structurally malformed.
> 2. **Using standard Base64 instead of Base64URL:** Standard Base64 uses characters like `+`, `/`, and padding `=` which break HTTP header parsing. JWT mandates Base64URL encoding (`-`, `_`, no padding).
> 3. **Assuming 2FA endpoints are rate-limited:** Neglecting to check for rate limiting in 2FA code verification, assuming 6-digit codes cannot be brute-forced without testing with Intruder or Turbo Intruder.

---

## 📚 References

1. [PortSwigger Web Security Academy: JWT Attacks](https://portswigger.net/web-security/jwt)
2. [PortSwigger Web Security Academy: OAuth 2.0 Flaws](https://portswigger.net/web-security/oauth)
3. [RFC 7519: JSON Web Token Standard Specification](https://datatracker.ietf.org/doc/html/rfc7519)

---

| [← Previous: 12. Vulnerability Testing](./12-vulnerabilities-with-burp.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 14. Access Control & IDOR →](./14-access-control.md) |
