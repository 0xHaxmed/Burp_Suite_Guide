# 07. Audit Trails & Evidence Management: Logger & Organizer

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/07-logger-organizer.md`)](../ar/07-logger-organizer.md)

---

## 🎯 Chapter Objectives

Master **Logger** as a comprehensive black-box telemetry recorder across all internal Burp modules to diagnose dropped connections and WAF blocks, and leverage **Organizer** as a structured evidence repository for snapshotting, annotating, and preserving critical Proof-of-Concept (PoC) HTTP transactions during active engagements.

---

## 🔍 Technical Explanation

### 1. The Logger Tab (Universal Traffic Telemetry):
Unlike `Proxy > HTTP history` which records solely the traffic passing through browser proxy listeners, **Logger** records every byte dispatched or received by Burp Suite across **all** internal subsystems:
* Manual manipulations from `Repeater`.
* High-volume automated attacks from `Intruder`.
* Automated audit traffic from the `Scanner` engine (`🔒 Pro`).
* Background API calls triggered by third-party `Extensions`.
* **Deep Diagnostic Telemetry:** Exposes critical operational metrics unavailable elsewhere: initiating tool source (`Tool Source`), exact server latency down to the millisecond (`Duration`), remote destination IP and port, and payload byte sizes.
* **WAF Block Diagnostics:** When exploit attempts fail silently or time out, inspecting Logger reveals whether upstream reverse proxies issued immediate `403 Forbidden` responses or severed TCP sockets via hard resets (`RST`).

### 2. The Organizer Tab (Structured Engagement Evidence):
During multi-day penetration tests, testers issue thousands of payloads. Identifying a critical vulnerability only to lose the reproducible state amidst rolling logs is a common frustration.
* **The Read-Only Snapshot Model:** Sending an HTTP transaction to `Organizer` freezes it as an immutable, read-only snapshot. It permanently preserves response bodies, headers, status codes, and execution timestamps against accidental modification.
* **Annotation & Workflow Triaging:** Provides dedicated workflows for labeling findings, attaching severity ratings (`Critical`, `High`, `Medium`, `Low`), assigning color flags, and drafting reproduction steps directly inside the workspace.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Diagnosing WAF Interference with Logger
1. When injecting custom SQLi or command injection strings in Repeater, the application suddenly drops responses or returns generic blank pages.
2. Switch immediately to the **Logger** tab.
3. Click the filter bar at the top and isolate Repeater traffic:
   * `Filter by tool: Repeater`.
4. Inspect transactions with abnormally short duration times returning status `403 Forbidden`, `406 Not Acceptable`, or `502 Bad Gateway`.
5. Examine the raw response body in Logger to determine whether Cloudflare, AWS WAF, or ModSecurity dropped the request, enabling you to refine your payload evasion strategy.

### Exercise 2: Archiving an IDOR Proof-of-Concept in Organizer
1. You identify an IDOR flaw in Repeater by tampering with a user ID in an API request (`/api/users/102`), returning unredacted sensitive PII.
2. In Repeater, right-click the HTTP transaction and choose **Send to Organizer** (or press `Ctrl + O`).
3. Switch to the **Organizer** tab:
   * The transaction is preserved as a permanent snapshot.
4. In the metadata panel on the right, populate engagement notes:
   * **Title:** `Insecure Direct Object Reference on User Profile Endpoint`
   * **Severity:** `High`
   * **Notes:** `Tampering with customer_id from 101 to 102 exposes full passport details. No authorization validation enforced.`
5. During report generation, open Organizer and copy the pristine, verified PoC directly into the deliverable.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: User ID controlled by request parameter](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)
  * Capture the authorization bypass transaction and store the validated finding inside Organizer before completing the lab.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Real-time evidence retention via Organizer ensures bug bounty researchers never lose reproducibility if an endpoint is patched mid-test, while Logger provides instant feedback on defensive filtering rules.
* **🛡️ Defender Perspective:**
  * Defenders require comprehensive, uncorrupted HTTP evidence to reproduce vulnerabilities in staging environments and formulate regression tests.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Relying solely on visual desktop screenshots:** Capturing screenshots without preserving the underlying raw HTTP request/response in Organizer makes validating or retesting findings significantly more difficult.
> 2. **Searching for extension traffic in HTTP History:** Wondering why automated extension actions do not show up in Proxy history, unaware that non-proxy traffic is isolated within Logger.
> 3. **Uncapped Logger memory consumption:** Leaving Logger active with unfiltered logging during massive automated scans, leading to high JVM memory consumption. Use filters or configure memory limits in settings.

---

## 📚 References

1. [PortSwigger Documentation: Burp Logger](https://portswigger.net/burp/documentation/desktop/tools/logger)
2. [PortSwigger Documentation: Burp Organizer](https://portswigger.net/burp/documentation/desktop/tools/organizer)
3. [NIST SP 800-115: Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/publications/detail/sp/800-115/final)

---

| [← Previous: 06. Utility Tools](./06-decoder-comparer-sequencer.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 08. Burp Collaborator →](./08-collaborator.md) |
