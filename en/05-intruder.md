# 05. Intruder & Intelligent Automation

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/05-intruder.md`)](../ar/05-intruder.md)

---

## 🎯 Chapter Objectives

Master **Intruder** for automated, highly customizable HTTP payload dispatch, dissect the mathematical and structural differences between the four attack types, configure payload transformation rules, utilize automated response parsing (`Grep Match` and `Grep Extract`), and bypass dynamic anti-automation mechanisms (such as volatile anti-CSRF tokens) using **Session Handling Rules** and **Macros**.

---

## 🔍 Technical Explanation

`Intruder` accepts an HTTP baseline request, wraps target input vectors within delimiter characters (`§injection_point§`), and systematically replaces those markers with values sourced from predefined payload generators according to strict dispatch logic.

### 1. The Four Attack Types:

| Attack Type | Payload Sets | Injection Dispatch Logic | Optimal Security Use Case |
| :--- | :---: | :--- | :--- |
| **Sniper** | 1 Set | Targets each marker sequentially; all other markers retain their baseline request values. | Single parameter fuzzing, XSS/SQLi injection, or directory discovery. |
| **Battering Ram** | 1 Set | Injects the exact **same** payload into **all** defined markers simultaneously on every request. | Scenarios requiring identical parameters (e.g., `user=admin&pass=admin`). **(Not for Race Conditions)**. |
| **Pitchfork** | Multiple (Up to 20 Sets) | Iterates across payload sets in parallel using synchronized list indices (Set 1 Item 1 paired with Set 2 Item 1). | **Credential Stuffing** using known username:password breach dumps. |
| **Cluster Bomb** | Multiple (Up to 20 Sets) | Computes the complete Cartesian product across all lists (every permutation of Set 1 against all entries in Set 2). | Comprehensive **Brute Force** cracking where pairings between usernames and passwords are unknown. |

### 2. Community Edition Throttling Mechanics:
In **Burp Suite Community Edition**, Intruder is artificially throttled. After an initial threshold of ~20 to 30 requests, an artificial exponential delay is introduced between outbound sockets.
* **Production Workarounds:** For high-throughput brute forcing, security researchers export raw requests to high-performance Go-based fuzzers like `ffuf`, or leverage the **Turbo Intruder** extension (available in the BApp store, bypassing the standard UI engine with an asynchronous C/Python HTTP stack).

### 3. Response Analysis Automation (Grep Match & Grep Extract):
When evaluating thousands of responses, manual inspection is impossible:
* **Grep - Match:** Flags matching regular expressions or literal strings (e.g., `Invalid credentials` or `Login successful`) as visual checkboxes in the result table.
* **Grep - Extract:** Extracts variable strings from HTML/JSON responses (e.g., user profiles, error traces, or verification tokens) directly into dedicated table columns.
* **Recursive Grep:** Pipes an extracted string from response $N$ into the payload parameter of request $N+1$ (crucial for rolling nonce tokens).

---

## 🛠️ Step-by-Step Hands-On Guide

### Exercise 1: Credential Stuffing via Pitchfork Attack
1. Capture an authentication request in Proxy and send it to Intruder (`Ctrl + I`).
2. In the **Positions** tab, click **Clear §** to remove automated guesses.
3. Select the username value and click **Add §**; select the password value and click **Add §**:
   ```http
   POST /login HTTP/1.1
   Host: target-app.com
   Content-Type: application/x-www-form-urlencoded

   username=§test§&password=§pass123§
   ```
4. Set the attack type to **Pitchfork**.
5. Switch to the **Payloads** tab:
   * Payload set 1: Paste your target username list.
   * Payload set 2: Paste the corresponding password dump.
6. Under **Settings > Grep - Match**:
   * Clear defaults and add: `Invalid username or password`.
7. Click **Start attack**; successful authentication attempts will display an unchecked Grep Match box alongside an anomalous response `Length` or `302 Redirect` status code.

### Exercise 2: Circumventing Anti-CSRF Tokens with Session Handling Macros
When an authentication form enforces a fresh anti-CSRF token per request:
1. Navigate to `Settings > Sessions > Macros` and click **Add**.
2. Select the prior `GET /login` transaction from your history and click **OK**; Burp's heuristic parser will detect the hidden CSRF input field.
3. Under **Session handling rules**, click **Add**:
   * Under the **Scope** tab: restrict the rule strictly to `Intruder` and your target scope.
   * Under **Details > Rule Actions**: add `Run a Macro`, select the newly created macro, and configure it to update only the `csrf` parameter.
4. Launch the Intruder attack. Intruder will automatically execute the macro to fetch a fresh token before dispatching every payload iteration.

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: Username enumeration via response timing](https://portswigger.net/web-security/authentication/password-reset/lab-username-enumeration-via-response-timing)
  * Utilize Intruder to identify valid usernames through measurable differences in server response processing time.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Attackers leverage payload processing rules (e.g., URL-encoding, base64 chaining, hash salting) to generate real-time variations that evade basic heuristic detection.
* **🛡️ Defender Perspective:**
  * Defeating Intruder automation requires adaptive **Rate Limiting** based on IP reputation and user session keys, progressive account lockout triggers, and multi-factor challenge prompts after consecutive failures.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Misapplying Battering Ram to Standard Brute Force:** Battering Ram injects the exact same string into both username and password fields; it will only succeed if the account uses credentials where username equals password.
> 2. **Mismatched List Lengths in Pitchfork Attacks:** If your username list contains 100 entries but the password list contains only 50, Pitchfork will abort at request 50, leaving the remaining 50 accounts unchecked.
> 3. **Relying on Community Intruder for Large-Scale Dictionaries:** Wasting hours waiting for an intentionally throttled tool when free CLI alternatives like `ffuf` execute thousands of requests per second.

---

## 📚 References

1. [PortSwigger Documentation: Intruder Attack Types](https://portswigger.net/burp/documentation/desktop/tools/intruder/attack-types)
2. [PortSwigger Documentation: Handling CSRF in Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/settings#extract-grep)
3. [PortSwigger Documentation: Session Handling Rules](https://portswigger.net/burp/documentation/desktop/settings/sessions/rules)

---

| [← Previous: 04. Repeater & Parallel Attacks](./04-repeater.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 06. Decoder, Comparer & Sequencer →](./06-decoder-comparer-sequencer.md) |
