# 16. HTTP/2 Anomalies & HTTP Request Smuggling

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/16-http2-request-smuggling.md`)](../ar/16-http2-request-smuggling.md)

---

## 🎯 Chapter Objectives

Deconstruct the low-level physical mechanics of **HTTP Request Smuggling**, analyze how parsing ambiguities in the HTTP/1.1 framing layer break boundary isolation between reverse proxies and backend servers, and master modern **HTTP/2 Downgrading (H2.CL / H2.TE)** exploitation vectors utilizing Burp Repeater and the specialized **HTTP Request Smuggler** extension to execute **Response Queue Poisoning** and user request hijacking.

---

## 🔍 Technical Explanation

### 1. The Desynchronization Dilemma:
Modern multi-tiered web architectures employ edge reverse proxies, CDNs, or load balancers that multiplex thousands of external client requests across a persistent, pipelined TCP connection (`Keep-Alive`) routed to internal application backend servers.
* For the backend server to discern where request $A$ terminates and request $B$ initiates, HTTP/1.1 provides two distinct framing mechanisms:
  1. The `Content-Length (CL)` header: Explicit octet length count.
  2. The `Transfer-Encoding: chunked (TE)` header: Delimited data chunks terminating in a null-length chunk (`0\r\n\r\n`).
* **The Vulnerability Trigger:** If the front-end and back-end disagree on which header takes precedence, **Desynchronization** occurs. The front-end forwards a request segment that the backend interprets as the preamble of the subsequent user's inbound request!

### 2. Classical HTTP/1.1 Smuggling Variants:
* **CL.TE:** Front-end prioritizes `Content-Length`; back-end prioritizes `Transfer-Encoding`.
* **TE.CL:** Front-end prioritizes `Transfer-Encoding`; back-end prioritizes `Content-Length`.
* **TE.TE:** Both servers support chunked framing, but obfuscating the header (e.g., `Transfer-Encoding: xchunked` or `Transfer-Encoding[\x0b]: chunked`) induces one proxy to ignore it and fall back to `Content-Length`.

### 3. Modern HTTP/2 Downgrading Anomalies (H2.CL / H2.TE):
HTTP/2 natively eliminates text-based framing ambiguities by enforcing binary framing with explicit frame payload lengths (`DATA` frames).
* **The Dangerous Trade-off (H2 Downgrading):** While edge proxies speak HTTP/2 with client browsers, many enterprise infrastructures convert inbound HTTP/2 traffic back down into cleartext `HTTP/1.1` before routing it to legacy internal clusters (**HTTP/2 Downgrading**).
* **H2.CL and H2.TE Vectors:** If an attacker injects an explicit `content-length` or `transfer-encoding` header inside an HTTP/2 frame, the front-end ignores it during H2 frame decoding, but blindly serializes it into the downgraded HTTP/1.1 stream, inducing instant backend desync.
* **CRLF Injection in Pseudo-Headers:** Attackers inject literal carriage-return line-feeds (`\r\n`) within HTTP/2 pseudo-headers (e.g., `:path` or custom headers). Upon downgrading, the front-end writes raw newlines, splitting the single HTTP/2 header into multiple distinct HTTP/1.1 headers.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Automated Detection via HTTP Request Smuggler
1. In `Extensions > BApp Store`, install the **HTTP Request Smuggler** extension.
2. Identify an in-scope target transaction in `Proxy > HTTP history`.
3. Right-click the transaction and select: **Extensions > HTTP Request Smuggler > Smuggle probe**.
4. The extension dispatches differential timing probes measuring delay discrepancies between CL.TE and TE.CL payloads without causing service denial.
5. Review results in the Dashboard issue activity log; confirmed desync vulnerabilities are flagged alongside exact timing metrics.

### Exercise 2: Manual CL.TE Desynchronization in Repeater
1. Send a target request to **Repeater** and ensure HTTP/1.1 is selected.
2. In the Repeater menu, disable automatic content length calculation:
   * Uncheck **Update Content-Length automatically**.
3. Construct a standard CL.TE probe payload where `Content-Length` encompasses only the immediate chunk header, leaving the trailing prefix dangling on the wire:
   ```http
   POST / HTTP/1.1
   Host: target-app.com
   Content-Length: 6
   Transfer-Encoding: chunked

   0

   G
   ```
4. When the backend parses `0\r\n\r\n`, it finalizes transaction $A$. The unread character `G` remains cached in the backend TCP socket buffer.
5. When the next client request arrives (`GET /home HTTP/1.1`), it is prefixed by the buffered character, morphing into `GGET /home HTTP/1.1`, returning an immediate HTTP `400 Bad Request` or unhandled routing error.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: HTTP request smuggling, basic CL.TE vulnerability](https://portswigger.net/web-security/request-smuggling/lab-basic-cl-te)
  * Practice configuring raw HTTP/1.1 headers in Repeater to desynchronize front-end and back-end proxies.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Request Smuggling is a high-severity critical primitive that bypasses perimeter WAF rules, enabling attackers to hijack other users' authenticated session cookies, tamper with cache contents, and inject arbitrary JavaScript.
* **🛡️ Defender Perspective:**
  * Complete mitigation requires implementing **End-to-End HTTP/2** without protocol downgrading, or configuring front-end proxies to normalize ambiguous headers and reject requests containing both `Content-Length` and `Transfer-Encoding`.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Leaving "Update Content-Length" enabled in Repeater:** Allowing Burp to automatically recalculate `Content-Length` during a smuggling exploit overrides your custom length byte count, instantly neutralizing the desync condition.
> 2. **Missing terminating CRLF sequences (`\r\n\r\n`):** Omitting the trailing double newlines after the terminating zero chunk causes the backend to hang indefinitely while waiting for missing bytes.
> 3. **Indiscriminate Testing on High-Traffic Production Environments:** Firing uncontrolled request smuggling payloads on live production environments can poison the request queues of real users, leading to service disruption and session hijacking of innocent parties.

---

## 📚 References

1. [James Kettle Research: HTTP Desync Attacks - Request Smuggling Reborn](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)
2. [James Kettle Research: HTTP/2 - The Sequel is Always Worse](https://portswigger.net/research/http2)
3. [PortSwigger Web Security Academy: HTTP Request Smuggling](https://portswigger.net/web-security/request-smuggling)

---

| [← Previous: 15. Modern API Auditing](./15-api-graphql-websockets.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 17. Pentest Workflow →](./17-pentest-workflow.md) |
