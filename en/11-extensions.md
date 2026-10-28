# 11. Extensions Ecosystem & The Montoya API

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/11-extensions.md`)](../ar/11-extensions.md)

---

## 🎯 Chapter Objectives

Expand Burp Suite into a tailored, industrial-grade offensive testing platform by navigating the **BApp Store**, contrasting the legacy Extender API with the modern **Montoya API** framework, properly provisioning runtime environments for **Jython** while recognizing runtime constraints, and integrating the essential arsenal of battle-tested community extensions utilized by top-tier bug bounty hunters.

---

## 🔍 Technical Explanation

### 1. Extension Runtime Architecture & The Montoya API:
Burp Suite exposes an extensible programmatic API that enables developers to hook the core proxy engine, inspect and rewrite HTTP/2 and WebSocket frames in flight, register automated scan heuristics, and add bespoke UI tabs to the main workbench:
* **The Montoya API:** Introduced as the comprehensive, modern replacement for the legacy `IBurpExtender` interface. Authored natively in Java and Kotlin, it delivers superior throughput, type safety, native HTTP/2 frame manipulation, and fine-grained control over collaborator events.
* **⚠️ Python & Jython Limitations (Critical Technical Clarification):**
  * Burp Suite cannot execute Python extensions directly via your local operating system's standard Python 3 interpreter. It relies on the **Jython Standalone JAR** (a Java-based compiler executing Python within the JVM).
  * **The Core Constraint:** Jython strictly targets **Python 2.7**. Modern Python 3 language constructs, asynchronous coroutines, and third-party C-extension libraries (such as `numpy` or `cryptography`) are fundamentally incompatible unless the extension bridges out of the JVM via gRPC or local sockets.
* **Ruby Integration:** Requires configuring the **JRuby Standalone JAR**.

### 2. The Professional BApp Arsenal:

| Extension Name | Core Operational Value in Web Penetration Testing |
| :--- | :--- |
| **Autorize** | The preeminent automated authorization engine. Concurrently replays authenticated requests with low-privileged and unauthenticated cookies to detect IDOR and privilege escalation on the fly. |
| **Logger++** | Advanced diagnostic logger featuring multithreaded regex search, custom columns, and full protocol payload indexing. |
| **Turbo Intruder** | Ultra-high-speed asynchronous HTTP client combining a C-based HTTP engine with a custom Python interface, achieving tens of thousands of requests per second for extreme race conditions. |
| **Param Miner** | Automated heuristic fuzzer designed to uncover unlinked request parameters and secret headers critical for Web Cache Poisoning exploits. |
| **JWT Editor** | Interactive suite for decoding, signing, and manipulating JSON Web Tokens, automating signature stripping (`none` algorithm) and key confusion attacks. |
| **Hackvertor** | Tag-based text converter (`<@base64>payload<@/base64>`) that dynamically parses, nests, and obfuscates payloads directly inside Repeater and Intruder. |
| **Burp Bounty** | Profile builder that transforms Burp Scanner into an active heuristic engine for custom CVE patterns and detection rules. |

---

## 🛠️ Step-by-Step Hands-On Guide

### Step 1: Configuring the Jython (Python) Runtime
1. Download the latest **Jython Standalone JAR** package from the official Jython repository (e.g., `jython-standalone-2.7.3.jar`).
2. Inside Burp Suite, navigate to `Settings > Extensions > Python environment`.
3. In the field **Location of Jython standalone JAR file**, click **Select file** and point to your downloaded JAR.
4. Burp can now seamlessly install and execute Python-authored BApp extensions from the store with zero compilation errors.

### Step 2: Uncovering Hidden Headers using Param Miner
1. Go to `Extensions > BApp Store`.
2. Locate **Param Miner** and click **Install**.
3. In `Proxy > HTTP history` or `Repeater`, identify an HTTP GET transaction for the application's root index page.
4. Right-click the transaction and select: **Extensions > Param Miner > Guess params > Guess headers**.
5. Keep defaults and click **OK** to launch background analysis.
6. Monitor analysis via `Extensions > Installed > Param Miner > Output`; when unkeyed headers (such as `X-Original-URL` or `X-Forwarded-Host`) are discovered, an alert is automatically published to the Dashboard.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: Web cache poisoning with an unkeyed header](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-an-unkeyed-header)
  * Utilize Param Miner to detect an unkeyed reverse proxy header and leverage it to poison the public frontend cache with a malicious XSS payload.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Custom extensions represent the competitive edge in bug bounty hunting. Top researchers write lightweight Montoya API plugins to inject bespoke headers across every manual transaction.
* **🛡️ Defender Perspective:**
  * Application security teams can author lightweight internal Montoya plugins to intercept traffic in pre-production staging environments and enforce strict compliance rules (e.g., verifying custom authorization tokens or mandatory security headers).

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Attempting to bind standard Python 3 executables:** Pointing the Jython path to `python.exe` or `python3` binary paths, which immediately crashes the extension loader. You must point strictly to a standalone `.jar` archive.
> 2. **Extension Overload:** Installing dozens of resource-heavy extensions simultaneously, saturating JVM heap allocations and severely degrading UI responsiveness.
> 3. **Ignoring Extension Execution Order:** Extensions in `Extensions > Installed` execute sequentially from top to bottom. If two extensions mutate the same request header, the topmost extension runs first.

---

## 📚 References

1. [PortSwigger Documentation: Montoya API Reference](https://portswigger.net/burp/extender/api/)
2. [Jython Official Project Site](https://www.jython.org/)
3. [PortSwigger GitHub: Official BApp Store Repositories](https://github.com/PortSwigger)

---

| [← Previous: 10. Embedded Browser & DOM Invader](./10-dom-invader-browser.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 12. Vulnerability Testing with Burp →](./12-vulnerabilities-with-burp.md) |
