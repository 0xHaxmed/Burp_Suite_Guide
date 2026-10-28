# 08. Burp Collaborator & Out-of-Band Testing (OAST) `🔒 Pro`

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/08-collaborator.md`)](../ar/08-collaborator.md)

---

## 🎯 Chapter Objectives

Understand the fundamental mechanics of **Out-of-Band Application Security Testing (OAST)**, master the deployment and operation of **Burp Collaborator** in Burp Suite Professional (`🔒 Pro`), and reliably detect and prove high-severity blind vulnerabilities that yield zero reflection in direct HTTP responses, including **Blind SSRF**, **Blind XXE**, **Blind OS Command Injection**, and **Blind XSS**.

---

## 🔍 Technical Explanation

### 1. The Blind Vulnerability Conundrum:
In standard in-band vulnerabilities, injected payloads produce immediate feedback within the corresponding HTTP response body or status code. Modern enterprise applications, however, frequently decouple input ingestion from backend processing:
* Payloads are placed into asynchronous message brokers (e.g., Kafka, RabbitMQ) and executed by detached worker services.
* Server-side backend daemons fetch internal resources without echoing data back to the user interface.
* In such environments, traditional scanners and manual inspection fail entirely because the application responds with a static `200 OK` or `{"status": "processing"}`.

### 2. Burp Collaborator Architecture & The OAST Model:
Burp Collaborator is an authoritative network appliance maintained by PortSwigger (or deployed as an on-premises enterprise instance) bound to a dedicated wildcard domain (e.g., `oastify.com`). It runs customized implementations of core network daemons:
* **Authoritative DNS Daemon:** Records every incoming `A`, `AAAA`, `CNAME`, or `TXT` query down to the sub-millisecond, capturing origin client IPs.
* **HTTP/S Listener:** Captures inbound web hooks, inspecting custom headers, TLS parameters, and POST request bodies.
* **SMTP/SMTPS Gateway:** Intercepts out-of-band email notifications triggered by header injection or mail-relay flaws.

```
[ Burp Suite GUI ] ---------------- (Polls for interactions) -----------------> [ Collaborator Server ]
        |                                                                                ^
  (Sends Payload:                                                                        |
   xyz.oastify.com)                                                                      |
        v                                                                                |
[ Vulnerable App ] -------- (App performs DNS lookup or HTTP request) -------------------+
```

### 3. The End-to-End Exploit Lifecycle:
1. The tester generates a cryptographically unique subdomain via Burp Collaborator (e.g., `a8f3b...oastify.com`).
2. The payload is injected into suspect input vectors (e.g., webhook endpoints, XML entities, or avatar image URLs).
3. The vulnerable target executes the payload and attempts network resolution against the unique domain.
4. The Collaborator server logs the asynchronous interaction.
5. Burp Suite periodically polls Collaborator via secure, end-to-end encrypted HTTPS channels, producing an instant, high-confidence alert inside the Dashboard without false positives.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Manual Verification of Blind SSRF
1. In Burp Suite Professional, open the client via **Burp > Burp Collaborator client** (or from the `Tools` menu).
2. Click **Copy to clipboard** to harvest an ephemeral subdomain (e.g., `random_id.oastify.com`).
3. Switch to **Repeater** and identify an endpoint accepting external resource links (such as an analytics ping or PDF generation request):
   ```http
   POST /analytics/track HTTP/1.1
   Host: target-app.com
   Content-Type: application/json

   {
     "referrer_url": "http://YOUR_COLLABORATOR_PAYLOAD_HERE"
   }
   ```
4. Dispatch the request; observe a generic `200 OK` response with no visible execution trace.
5. Return to the **Burp Collaborator client** window and click **Poll now**.
6. The arrival of asynchronous **DNS lookups** or **HTTP GET** requests originating from target infrastructure IP blocks delivers undeniable proof of **Blind SSRF**.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: Blind SSRF with out-of-band detection](https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)
  * Practice identifying an un-echoed vulnerability by configuring a Collaborator payload inside an HTTP analytics tracker to capture a DNS resolution.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Collaborator enables attackers to harvest internal server hostnames and establish covert command-and-control communication channels out of hardened corporate DMZs where standard inbound access is barred.
* **🛡️ Defender Perspective:**
  * Mitigation requires strict **Egress Firewall Rules**. Servers processing user-supplied input must be completely blocked from making unauthenticated outbound DNS lookups or HTTP requests to the public internet.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Searching for Collaborator in Community Edition:** Burp Collaborator is strictly an enterprise-grade `🔒 Pro` feature. Free alternative tooling includes CLI utilities like `Interactsh`.
> 2. **Using Public Collaborator in Highly Regulated Assessments:** Sending proprietary banking or healthcare payloads through PortSwigger's public cloud violates strict NDAs and compliance frameworks (e.g., PCI-DSS, HIPAA). Deploy a **Private Burp Collaborator Server** under your organization's control.
> 3. **Dismissing DNS-only interactions:** Often, enterprise egress firewalls allow port 53 (DNS) while blocking port 80/443 (HTTP). A captured DNS interaction is full technical proof that the code reached the network socket and parsed your payload.

---

## 📚 References

1. [PortSwigger Research: OAST Application Security Testing](https://portswigger.net/burp/application-security-testing/oast)
2. [PortSwigger Documentation: Burp Collaborator Architecture](https://portswigger.net/burp/documentation/desktop/tools/collaborator)
3. [PortSwigger Guide: Deploying a Private Collaborator Instance](https://portswigger.net/burp/documentation/collaborator/deploying)

---

| [← Previous: 07. Logger & Organizer](./07-logger-organizer.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 09. Burp Scanner →](./09-scanner.md) |
