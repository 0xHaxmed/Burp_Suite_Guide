# 10. Embedded Browser & DOM Invader

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/10-dom-invader-browser.md`)](../ar/10-dom-invader-browser.md)

---

## 🎯 Chapter Objectives

Harness the efficiency of **Burp's Embedded Browser** to eliminate external proxy and certificate configuration overhead, and master **DOM Invader**—the advanced in-browser instrumentation engine—to dynamically identify, trace, and exploit complex client-side vulnerabilities including **DOM-based XSS**, **Web Message (`postMessage`) security flaws**, and **Client-Side Prototype Pollution**.

---

## 🔍 Technical Explanation

### 1. Burp's Embedded Chromium Browser:
Burp Suite ships with a fully isolated, custom-built instance of Chromium:
* **Zero-Configuration:** Requires no extension installation (FoxyProxy) or root CA certificate exports. The embedded browser is pre-seeded with trust for Burp's local CA certificate.
* **Sandbox & Session Isolation:** Completely separated from host browser profiles, ensuring your personal sessions, passwords, and extensions never leak into testing databases.

### 2. DOM Invader (Client-Side Instrumentation Engine):
In traditional reflected/stored XSS, payloads bounce off the remote web server. In **DOM-based XSS**, untrusted data flows entirely on the client side: from an execution **Source** (e.g., `location.hash`, `window.name`, `document.referrer`) directly into a dangerous execution **Sink** (e.g., `innerHTML`, `document.write`, `eval`) within the browser's JavaScript runtime without ever touching the web server.
* **The Blind Spot:** Server-side proxy logs cannot observe this data flow.
* **The DOM Invader Solution:** Integrated directly into Chromium DevTools, DOM Invader hooks JavaScript execution contexts, injecting a unique **Canary String** and tracking data taint propagation:
  * Analyzes whether user-controlled input reaches a dangerous Sink.
  * Evaluates sanitization routines to confirm whether contextual HTML/JS escaping was missed.

### 3. Auditing Web Messages & Prototype Pollution:
* **Cross-Document Messaging (`postMessage`):** Intercepts event handlers (`window.addEventListener('message', ...)`) and audits whether receiving windows validate the `event.origin` property before processing payloads, identifying unauthorized cross-origin execution risks.
* **Client-Side Prototype Pollution:** Probes for unchecked object merges or URL-based object modification (`__proto__`, `constructor.prototype`), highlighting gadget chains that escalate prototype modifications into full DOM XSS.

---

## 🛠️ Step-by-Step Hands-On Guide

### Step 1: Launching the Embedded Browser & Initializing DOM Invader
1. Under `Proxy > Intercept`, click **Open browser**.
2. Within the launched Chromium window, press `F12` (or right-click > **Inspect**) to open Developer Tools.
3. Select the **DOM Invader** tab on the far right of DevTools.
4. Toggle the master switch: **DOM Invader is on**.
5. Under configuration settings:
   * Enable **Postmessage interception**.
   * Enable **Prototype pollution**.
   * Copy the generated **Canary string**.

### Step 2: Exploiting a DOM XSS Sink
1. Navigate to a target web application page that handles query parameters via client-side scripts.
2. In the target input field or URL hash, append your Canary alongside delimiter characters: `CANARY"'>`
3. Review the **DOM Invader** DevTools panel:
   * The tree view will reveal the exact **Source** (e.g., `location.search`) and the receiving **Sink** (e.g., `element.innerHTML`).
4. Click the **Exploit** button adjacent to the identified sink:
   * DOM Invader will automatically compute the appropriate escaping sequence and fire an interactive `alert(1)` proof of concept.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: DOM XSS using web messages](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-using-web-messages)
  * Practice using DOM Invader to inspect unvalidated `postMessage` listeners and construct an iframe-based exploit to trigger cross-origin DOM XSS.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * DOM Invader instantly cuts through hundreds of thousands of lines of obfuscated, bundled frontend JavaScript code (Webpack, Vite), revealing exploitable client-side injection sinks in seconds.
* **🛡️ Defender Perspective:**
  * Preventing DOM XSS requires developers to deprecate dangerous DOM sinks (`innerHTML`, `outerHTML`) in favor of safe setters (`textContent`, `innerText`), while enforcing strict `event.origin` whitelisting across all `postMessage` event listeners.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Expecting DOM Invader in external browsers:** DOM Invader is an internal feature of Burp's pre-packaged Chromium browser; it does not exist in standalone Firefox or Chrome installations.
> 2. **Testing without opening DevTools:** DOM Invader only executes its instrumentation and displays findings when the browser's Developer Tools pane is actively opened.
> 3. **Overlooking Prototype Pollution findings:** Dismissing prototype pollution warnings as harmless client quirks without investigating available JavaScript gadget libraries in the DOM that can escalate into remote code execution or DOM XSS.

---

## 📚 References

1. [PortSwigger Documentation: Burp Embedded Browser](https://portswigger.net/burp/documentation/desktop/tools/proxy/browser)
2. [PortSwigger Documentation: DOM Invader Master Guide](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
3. [OWASP Client-Side Testing Guide: DOM-Based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)

---

| [← Previous: 09. Burp Scanner](./09-scanner.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 11. Extensions & Montoya API →](./11-extensions.md) |
