# 14. Access Control, IDOR & Authorization Auditing (Autorize)

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/14-access-control.md`)](../ar/14-access-control.md)

---

## 🎯 Chapter Objectives

Establish an audit framework for Broken Object Level Authorization (**BOLA / IDOR**) and functional access control mechanisms, understand horizontal and vertical privilege escalation vectors, bypass URL filters using specialized request headers, and automate access control matrix validation at scale using the **Autorize** extension.

---

## 🔍 Technical Explanation

### 1. Taxonomy of Access Control Vulnerabilities:
* **Vertical Privilege Escalation:** An unprivileged user accesses administrative actions (e.g., executing `/admin/delete-user` or altering billing tables) due to missing role-based checks.
* **Horizontal Privilege Escalation & IDOR:** An authenticated subject (User A) accesses or modifies discrete resources owned by an equivalent peer (User B) simply by altering identifier parameters (`id=105` to `id=106`) in URLs, query strings, or JSON bodies.
* **Header-Based Authorization Bypasses:** Target frameworks (such as Spring or custom API gateways) evaluate routing headers like `X-Original-URL: /admin` or `X-Rewrite-URL`, bypassing reverse proxy access control layers while forwarding internal routes to backends.

### 2. The Autorize Dual-Replay Engine Architecture:
In extensive enterprise architectures spanning hundreds of endpoints, manually swapping session tokens in Repeater is error-prone. The **Autorize** extension automates this matrix auditing:
* The tester logs in as a low-privileged subject, extracts the session cookies, and injects them into Autorize's configuration listener.
* The tester navigates the target application using high-privileged administrative credentials in their standard browser.
* **Automated Dual-Replay Cycle:** For every HTTP transaction initiated by the administrator, Autorize instantly clones and dispatches two parallel background requests:
  1. Request $A$: Executed with the low-privileged session token.
  2. Request $B$: Executed unauthenticated (cookies stripped).
* **Autorize Heuristic Status System:**
  * 🔴 **Bypassed! (Red):** Definitive authorization failure! The low-privileged subject was granted access or modification rights.
  * 🟢 **Enforced (Green):** Access securely blocked with HTTP `403 Forbidden` or `401 Unauthorized`.
  * 🟠 **Is Enforced??? (Orange):** Ambiguous state requiring manual triage due to identical response lengths or generic redirection statuses.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Configuring & Running an Autorize Matrix Scan
1. In `Extensions > BApp Store`, install **Autorize**.
2. Open two separate browser profiles:
   * Profile 1: Log in as standard test user (`carlos`).
   * Profile 2: Log in as administrative user (`admin`).
3. Copy the low-privileged user's cookie header (`Cookie: session=xyz...`).
4. Switch to the **Autorize** tab in Burp:
   * Paste the cookie string into **Configuration > Low Privileged User Cookies**.
   * Toggle the top button: **Autorize is on**.
5. With Profile 2, systematically exercise administrative workflows (user creation, profile modification, settings updates).
6. Inspect the Autorize grid: Rows highlighted in bright red (`Bypassed!`) immediately pinpoint severe horizontal or vertical privilege escalation flaws.

### Exercise 2: Bypassing Front-End Route Filters via `X-Original-URL`
1. Attempt direct access to an administrative path in Repeater:
   ```http
   GET /admin HTTP/1.1
   Host: target-app.com
   ```
   * The reverse proxy responds with `403 Access Denied`.
2. Rewrite the path to an authorized public root and inject `X-Original-URL`:
   ```http
   GET / HTTP/1.1
   Host: target-app.com
   X-Original-URL: /admin/delete?username=carlos
   ```
3. Dispatch the transaction; an HTTP `200 OK` confirming user deletion indicates that front-end proxy enforcement was successfully bypassed.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: URL-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)
  * Practice bypassing reverse proxy access control lists using the `X-Original-URL` header to access an administrative dashboard.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Broken Object Level Authorization (BOLA) vulnerabilities constitute the most common and lucrative findings in bug bounty programs due to microservice architectures decoupling UI visibility from backend data retrieval.
* **🛡️ Defender Perspective:**
  * Authorization must be verified as a deterministic, centralized gatekeeper at the data access object (DAO) and service layer for every single request, validating that `session.user_id === record.owner_id`.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Equating UI Element Obfuscation with Security:** Assuming that hiding a "Delete User" button in the frontend HTML prevents an attacker from issuing raw `POST /api/users/delete` requests directly in Repeater.
> 2. **Unfiltered Autorize Logging:** Omitting static file exclusions in Autorize configuration, causing images, scripts, and stylesheets to flood the results table with misleading false positives.
> 3. **Testing IDOR Exclusively on GET Verbs:** Overlooking state-mutating requests (`PUT`, `PATCH`, `DELETE`) where unauthorized data modifications and record deletions frequently occur unchecked.

---

## 📚 References

1. [PortSwigger Web Security Academy: Access Control Flaws](https://portswigger.net/web-security/access-control)
2. [OWASP Web Security Testing Guide: Testing for IDOR](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
3. [Autorize Official GitHub Repository](https://github.com/Quitten/Autorize)

---

| [← Previous: 13. Authentication & JWT](./13-auth-session-jwt-oauth.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 15. API, GraphQL & WebSockets →](./15-api-graphql-websockets.md) |
