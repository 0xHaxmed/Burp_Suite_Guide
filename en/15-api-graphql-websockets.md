# 15. Modern API Auditing: REST, GraphQL & WebSockets

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/15-api-graphql-websockets.md`)](../ar/15-api-graphql-websockets.md)

---

## 🎯 Chapter Objectives

Master offensive testing methodologies for modern API ecosystems and persistent bidirectional transports within Burp Suite. Intercept and manipulate **RESTful APIs**, extract hidden backend architectures using **GraphQL Introspection** and execute **Batching Attacks**, and capture, tamper with, and hijack persistent message streams via **WebSockets** and **Cross-Site WebSocket Hijacking (CSWSH)**.

---

## 🔍 Technical Explanation

### 1. RESTful API Auditing Mechanics:
Modern single-page applications communicate almost exclusively via asynchronous JSON payloads over RESTful routes:
* **Documentation Discovery:** Systematically probe for exposed API schemas (`/swagger.json`, `/openapi.json`, `/v3/api-docs`).
* **HTTP Verb Tampering:** Altering transport verbs (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`) to evaluate whether authorization middleware only guards specific methods while leaving alternatives exposed.
* **Content-Type Negotiation Smuggling:** Flipping `Content-Type: application/json` to legacy types (`application/xml`, `multipart/form-data`) to force backend engines into processing unhardened fallback parsers vulnerable to XXE or parameter injection.

### 2. GraphQL Assessment & Introspection Abuse:
Unlike REST, GraphQL multiplexes all operations (queries and mutations) through a single HTTP endpoint (typically `/graphql`):
* **Introspection Probing:** A built-in diagnostic subsystem that returns the complete, uncompiled schema (`__schema`, `__type`). If enabled in production, an attacker downloads the entire backend object model, disclosing private entities, hidden administrative fields, and debug queries.
* **Batching Attacks (Rate-Limit Bypass):** GraphQL natively supports array batching, packaging multiple mutations inside a single HTTP wrapper:
  ```json
  [
    {"query": "mutation { login(user: \"admin\", pass: \"123\") }"},
    {"query": "mutation { login(user: \"admin\", pass: \"456\") }"}
  ]
  ```
  This effectively bypasses standard perimeter rate-limiting appliances configured to count HTTP request volume.

### 3. Real-Time WebSockets Interception:
WebSockets establish a persistent, low-latency, full-duplex TCP stream initiated via an HTTP handshake (`Upgrade: websocket`):
* **Proxy > WebSockets History:** Logs bidirectional frame transactions labeled by transport direction (`To client` / `To server`).
* **WebSocket Interception & Tampering:** Enables live, on-the-fly mutation of binary and text frames before delivery to real-time chats or trading feeds.
* **Cross-Site WebSocket Hijacking (CSWSH):** Occurs when the WebSocket upgrade handshake relies exclusively on ambient cookies without validating the `Origin` header or requiring CSRF challenge tokens, allowing malicious origins to hijack active user sockets.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Harvesting the Backend Schema via GraphQL Introspection
1. Intercept a GraphQL query in Proxy and route it to **Repeater**.
2. Replace the body with the standard introspection probe query:
   ```json
   {
     "query": "{ __schema { types { name fields { name } } } }"
   }
   ```
3. Click **Send**; receiving a structured JSON document mapping types and queries confirms introspection is exposed.
4. **Pro Tip:** In `Extensions > BApp Store`, install the **InQL** extension. It automatically digests the raw introspection payload, generates ready-to-test queries for every mutation, and surfaces hidden parameters in a dedicated GUI panel.

### Exercise 2: Intercepting & Mutating Live WebSocket Frames
1. Navigate to `Proxy > WebSockets history` and initiate a session on a target web application utilizing a real-time messaging window.
2. In the `Proxy > Intercept` tab, ensure WebSocket message interception is enabled.
3. Send a message in the web application; Burp's intercept window will halt the outbound WebSocket frame.
4. Inject a test script payload into the message payload attribute:
   `{"chat_id": 402, "text": "<script>alert(origin)</script>"}`
5. Click **Forward**; observe the client-side execution in the receiving window, verifying Stored XSS over the WebSocket transport.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: Performing CSRF exploits over WebSocket](https://portswigger.net/web-security/websockets/lab-cross-site-websocket-hijacking)
  * Practice crafting a cross-origin HTML page to hijack a victim's authenticated WebSocket connection and exfiltrate confidential live chat transcripts.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Attackers target GraphQL and WebSockets because enterprise security visibility and web application firewalls frequently lack real-time inspection filters for multiplexed or persistent socket protocols.
* **🛡️ Defender Perspective:**
  * Engineering teams must disable introspection across public-facing GraphQL deployments, enforce query cost-analysis limits, and validate the `Origin` header during initial WebSocket handshake negotiation.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Routing WebSocket frames to standard HTTP Repeater:** Attempting to manipulate WebSocket frames inside standard HTTP Repeater tabs; you must right-click from `WebSocket history` and select **Send to Repeater** to open a dedicated WebSocket connection tab.
> 2. **Neglecting GraphQL Batching Limits:** Failing to test array wrappers on sensitive endpoints, assuming standard perimeter rate limiters apply equally to internal GraphQL query iterations.
> 3. **Treating WebSockets as Immune to Injection:** Assuming that because messages travel over WebSockets, vulnerabilities like SQL Injection, Command Injection, or XSS cannot manifest.

---

## 📚 References

1. [PortSwigger Web Security Academy: GraphQL API Security](https://portswigger.net/web-security/graphql)
2. [PortSwigger Web Security Academy: WebSockets Vulnerabilities](https://portswigger.net/web-security/websockets)
3. [OWASP API Security Top 10 Specification](https://owasp.org/www-project-api-security/)

---

| [← Previous: 14. Access Control & IDOR](./14-access-control.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 16. HTTP/2 & Request Smuggling →](./16-http2-request-smuggling.md) |
