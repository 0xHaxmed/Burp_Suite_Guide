# 04. Repeater & Parallel Attacks (Race Conditions)

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/04-repeater.md`)](../ar/04-repeater.md)

---

## 🎯 Chapter Objectives

Master **Repeater** as a surgical workspace for manual HTTP transaction inspection and modification, explore Inspector and view layouts, and leverage Tab Groups to exploit sub-millisecond **Race Conditions** via the revolutionary **Single-Packet Attack** mechanism rather than erroneously misapplying fuzzing tools.

---

## 🔍 Technical Explanation

### 1. The Repeater Philosophy (Methodical Manual Analysis):
While `Intruder` automates high-volume blind fuzzing, `Repeater` isolates a single HTTP request for precise parameter manipulation. Testers can alter individual characters, recalculate signatures, and observe fine-grained differentials in responses:
* **Interface Architecture:** Raw editable request pane on the left, inspectable response pane on the right, selectable across multiple views (`Pretty`, `Raw`, `Hex`, and browser-rendered `Render`).
* **The Inspector Panel:** Parses requests into structural widgets (Request Attributes, Query Parameters, Body Parameters, Cookies, Headers), enabling inline URL/Base64 transformations without breaking protocol syntax.

### 2. Race Conditions and the Single-Packet Attack:
A **Race Condition** occurs when a multithreaded web application processes multiple concurrent requests against shared state without synchronization barriers (e.g., applying a promotional gift card multiple times, double-withdrawing funds, or bypassing rate limits):
* **The Technical Fallacy:** Attempting to trigger Race Conditions using Intruder's Battering Ram. Battering Ram transmits requests sequentially over standard sockets and cannot synchronize packet arrival times at the remote network interface.
* **The Repeater Solution (Single-Packet Attack):**
  * Collate requests into a designated **Tab Group**.
  * Select **Send group in parallel**.
  * Under HTTP/2, Burp utilizes multiplexed stream frames packed into a single TCP packet. Under HTTP/1.1, it synchronizes via *Last-Byte Sync*—holding back the terminating byte of each request across concurrent connections and blasting them simultaneously. This eliminates packet jitter and forces the application backend to process all transactions within the exact same CPU clock cycle.

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Step-by-Step SQL Injection Exploitation
1. Locate an interesting product query in `Proxy > HTTP history` (e.g., `GET /products?category=Gifts`).
2. Press `Ctrl + R` (or right-click > **Send to Repeater**).
3. Inject a single quote `'` delimiter to break query parsing:
   ```http
   GET /products?category=Gifts' HTTP/1.1
   Host: target-app.com
   ```
4. Click **Send**; analyze the response for database syntax errors or status `500 Internal Server Error`.
5. Incrementally balance the query and determine column count via `UNION SELECT`:
   ```http
   GET /products?category=Gifts' UNION SELECT NULL,NULL-- HTTP/1.1
   Host: target-app.com
   ```

### Exercise 2: Exploiting Race Conditions via Parallel Tab Groups
1. Intercept a coupon redemption request and forward it to Repeater:
   ```http
   POST /cart/apply-coupon HTTP/1.1
   Host: target-app.com
   Content-Type: application/x-www-form-urlencoded

   csrf=token123&coupon=PROMO20
   ```
2. Right-click the tab and select **Duplicate** to generate 15 identical requests.
3. Group these tabs by right-clicking them and selecting **Add tabs to group > Create group** (named `Coupon-Race`).
4. Click the drop-down arrow beside the **Send** button and select: **Send group in parallel**.
5. Inspect the responses: Multiple `200 OK` responses containing applied discounts confirm that the state machine was overwhelmed before database balance updates could be committed.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: Limit overrun race conditions](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun)
  * Hands-on lab using Repeater's parallel group dispatch to apply a single-use discount coupon multiple times simultaneously.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Repeater gives complete autonomy over timing and syntax. Subtle business logic flaws, authorization boundaries, and concurrency errors that automated scanners miss are discovered through careful manual probing.
* **🛡️ Defender Perspective:**
  * Mitigating concurrency vulnerabilities requires atomic database transactions (`SELECT FOR UPDATE`), distributed mutex locks, or token buckets applied at the database transaction layer rather than relying on standard application-level conditional checks.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Using Battering Ram for Race Conditions:** Battering Ram does not synchronize socket transmission; requests arrive staggered across different network frames, missing race windows.
> 2. **Neglecting Content-Length Header Synchronization:** Manually altering body payloads in raw views without letting Burp recompute the `Content-Length` header, which causes servers to hang or truncate requests.
> 3. **Unorganized Tab Sprawl:** Opening dozens of unnamed tabs without structuring them into Tab Groups, leading to confusion and lost audit trails.

---

## 📚 References

1. [PortSwigger Research: Smashing the State Machine](https://portswigger.net/research/smashing-the-state-machine)
2. [PortSwigger Documentation: Burp Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater)
3. [James Kettle: Single-Packet Attack Whitepaper](https://portswigger.net/research/single-packet-attack-the-true-impact-of-parallel-requests)

---

| [← Previous: 03. Advanced Proxy](./03-proxy-advanced.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 05. Intruder & Automation →](./05-intruder.md) |
