# 02. Dashboard, Target & Scope Management

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/02-dashboard-target-scope.md`)](../ar/02-dashboard-target-scope.md)

---

## 🎯 Chapter Objectives

Master Burp Suite's central command station (`Dashboard`), map the target application's attack surface using the hierarchical `Site Map` in the `Target` tab, and configure precise `Scope Rules` to eliminate third-party background traffic noise and avoid testing out-of-scope infrastructure.

---

## 🔍 Technical Explanation

### 1. The Dashboard Tab (Operations Control):
The Dashboard provides an aggregated overview across the engagement lifecycle through four dedicated panels:
* **Tasks:** Manages active background routines, including live passive crawl engines and scheduled automated audit tasks (`🔒 Pro`).
* **Issue Activity:** Displays a real-time feed of identified vulnerabilities, categorized by severity (`High`, `Medium`, `Low`, `Information`) and confidence (`Certain`, `Firm`, `Tentative`).
* **Issue Advisory:** Selecting any vulnerability renders comprehensive remediation context, technical explanations, and raw request/response proof-of-concept pairs.
* **Event Log:** Houses diagnostic system logs, transport errors, socket exhaustion alerts, and extension events.

### 2. The Target Tab & Site Map:
The Target tab automatically constructs a structured, tree-like layout (`Site Map`) of every host, URI path, query parameter, and asset requested by the browser. Discovered links discovered in parsed HTML and JavaScript files are rendered as muted gray nodes until actively requested.

### 3. The Noise Dilemma and the Necessity of Scope:
Modern web applications pull assets from dozens of third-party domains (analytics, telemetry, ad servers, CDNs, and internal browser mechanisms like `detectportal.firefox.com`).
* **The Problem:** Floods HTTP history with thousands of irrelevant transactions.
* **The Legal Hazard:** Sending automated payloads against non-contracted third-party domains represents unauthorized testing and may trigger automated security blocks or legal liability.
* **The Solution:** Configuring **Target Scope**—a deterministic filter that restricts Burp Suite to capture, process, and display only URLs belonging to authorized targets.

---

## 🛠️ Step-by-Step Hands-On Guide

### Step 1: Mapping the Attack Surface
1. With your browser connected to Burp, manually navigate key workflows across the target application (log in, load user dashboards, view items, post comments).
2. Switch to `Target > Site map` to view the newly compiled application tree.

### Step 2: Defining Target Scope
1. In the `Site map` tree, right-click the root domain of your primary target (e.g., `https://target-app.com`).
2. Select **Add to scope**.
3. A confirmation dialog will appear: *"Do you want Proxy to stop sending out-of-scope items to HTTP history?"*
   * Click **Yes** to discard background noise before it hits your project storage.

### Step 3: Advanced Scope Rules Configuration
1. Navigate to `Target > Scope settings`.
2. Configure advanced regular expressions for broad subdomains or strict path boundaries:
   * To include all target subdomains: `.*\.target-app\.com$`
   * To prevent accidental automated logouts during scanning or fuzzing, add the logout URI to **Exclude from scope**:
     * `^https://target-app\.com/logout$`

### Step 4: Filtering Views to Show Only In-Scope Targets
1. In either `Target > Site map` or `Proxy > HTTP history`, click the grey filter bar at the top (`Filter: Showing all items`).
2. Enable the primary filter checkbox: **Show only in-scope items**.
3. (Optional) Filter out static noise by unchecking `CSS`, `Images`, and `Flash`.
4. Click **Apply**. The interface will instantly isolate the authorized targets.

---

## 🧪 Recommended Labs

* **Foundational Lab:** [PortSwigger Academy: Target Scope Configuration & Site Map](https://portswigger.net/web-security/learning-paths)
  * Practice establishing a strictly scoped environment, navigating the target application, and filtering logs to uncover unindexed administrative endpoints.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker / Pentester Perspective:**
  * Strict scope enforcement is the professional line between a legitimate penetration test and illegal activity. It ensures scanning tools and intruder payloads never hit third-party servers outside the signed rules of engagement (RoE).
* **🛡️ Defender / Security Engineer Perspective:**
  * Security teams analyze the complete Site Map to identify shadow APIs, forgotten staging endpoints, and legacy microservices exposed to the internet without proper authentication barriers.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Testing without defining Scope:** Leaving all hosts visible in HTTP History, making it nearly impossible to trace authentication tokens or sensitive parameters among thousands of browser tracking requests.
> 2. **Attacking third-party CDNs or APIs:** Accidentally fuzzing endpoints hosted on `google-analytics.com` or `cloudflare.com`, which violates engagement contracts.
> 3. **Failing to exclude logout endpoints:** Leaving `/logout` in scope during automated fuzzing or scanning, causing the application to terminate the active session and invalidating subsequent audit checks.

---

## 📚 References

1. [PortSwigger Documentation: Target Scope](https://portswigger.net/burp/documentation/desktop/tools/target/scope)
2. [PortSwigger Documentation: Target Site Map](https://portswigger.net/burp/documentation/desktop/tools/target/site-map)
3. [OWASP Web Security Testing Guide: Information Gathering](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/)

---

| [← Previous: 01. Setup & Proxy](./01-setup-proxy-https.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 03. Advanced Proxy →](./03-proxy-advanced.md) |
