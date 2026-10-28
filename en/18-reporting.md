# 18. Security Reporting & Vulnerability Documentation

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/18-reporting.md`)](../ar/18-reporting.md)

---

## 🎯 Chapter Objectives

Transform technical vulnerability findings and raw artifacts preserved in Burp Suite into **executive-ready, high-fidelity security reports** suitable for bug bounty programs (HackerOne / Bugcrowd) and corporate penetration testing deliverables, accurately calculate vulnerability severity utilizing **CVSS v3.1 / v4.0** metrics, construct clear deterministic reproduction steps, and formulate actionable engineering remediation guidance.

---

## 🔍 Technical Explanation

A vulnerability report represents the sole tangible deliverable seen by clients and engineering leadership. Irrespective of technical sophistication within Burp Suite, poorly communicated findings lead to rejected triage submissions, lowered bounty payouts, or ignored tickets.

### 1. Structural Anatomy of an Industry-Standard Vulnerability Report:
1. **Title & Categorization:** Descriptive summary naming the exact flaw and vulnerable URI (e.g., `IDOR on /api/v1/user/profile leads to full PII leak`), mapped directly to standard `CWE` and `OWASP Top 10` taxonomies.
2. **Severity & CVSS Scoring:** Mathematical risk rating established via the **Common Vulnerability Scoring System (CVSS)** between `0.0` and `10.0`, objectively scored against Attack Vector (AV), Attack Complexity (AC), Privileges Required (PR), User Interaction (UI), Scope (S), and Confidentiality/Integrity/Availability (CIA) impacts.
3. **Executive Summary & Business Impact:** Business-focused narrative articulating concrete financial, legal, or reputational risks (e.g., GDPR data breach liabilities, unauthorized fund transfers).
4. **Deterministic Steps to Reproduce:** Ordered, unambiguous reproduction walkthrough that enables security triage teams to replicate the issue within two minutes.
5. **Raw HTTP Evidence (Proof of Concept):** Unredacted raw HTTP request and response pairs retrieved directly from **Burp Organizer** or Repeater.
6. **Engineering Remediation Guidance:** Concrete defensive source code implementations and architectural patterns rather than generic guidance like "sanitize all user input."

---

## 🛠️ Step-by-Step Hands-On Guide

### Production-Ready Markdown Vulnerability Report Template:

```markdown
# [HIGH] Insecure Direct Object Reference (IDOR) on User Profile Endpoint

## 📌 Metadata
* **Target:** `https://api.target-app.com/v1/users/{id}`
* **Vulnerability Type:** CWE-639: Authorization Bypass Through User-Controlled Key
* **CVSS v3.1 Score:** 7.5 (High) - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`
* **Date:** 2026-10-28

---

## 📝 Description
An Insecure Direct Object Reference (IDOR) vulnerability was identified in the user profile API endpoint. The application fails to validate whether the requesting authenticated session possesses the authorization rights to access the specified customer record.

---

## 💥 Business Impact
An unauthenticated or low-privileged attacker can enumerate sequential integer IDs to scrape the complete personally identifiable information (PII) of all registered customers—including full names, email addresses, phone numbers, and physical billing addresses—leading to severe regulatory non-compliance (GDPR/CCPA violations) and identity theft.

---

## 🔬 Steps to Reproduce
1. Log into your account as a regular user and note your account ID (`1001`).
2. Navigate to `Profile Settings` and intercept the request in **Burp Suite**.
3. Send the profile retrieval request to **Repeater**:
   `GET /api/v1/users/1001 HTTP/1.1`
4. Change the ID parameter in the URI from `1001` to `1002`.
5. Dispatch the request; observe an HTTP `200 OK` exposing the private data of user `1002`.

---

## 📜 Raw Proof of Concept (from Burp Organizer)

### Request:
```http
GET /api/v1/users/1002 HTTP/1.1
Host: api.target-app.com
Authorization: Bearer <VICTIM_LOW_PRIV_TOKEN>
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
```

### Response:
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 214

{
  "id": 1002,
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "phone": "+1-555-0199",
  "ssn": "XXX-XX-1234",
  "address": "123 Elm Street, Dallas, TX"
}
```

---

## 🛡️ Remediation
Enforce server-side authorization checks verifying that the requesting user's identity stored in the session token matches the requested resource ID:

```javascript
// Secure Implementation Example (Node.js/Express)
app.get('/api/v1/users/:id', authenticateToken, async (req, res) => {
  const requestedUserId = req.params.id;
  const authenticatedUserId = req.user.id;

  if (requestedUserId !== authenticatedUserId && !req.user.isAdmin) {
    return res.status(403).json({ error: "Access Denied: Unauthorized access to resource." });
  }

  const profile = await getUserProfile(requestedUserId);
  return res.json(profile);
});
```
```

---

## 🧪 Recommended Labs

* **Interactive Lab:** [PortSwigger Academy: User ID controlled by request parameter with data leakage in redirect](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
  * Complete the lab, extract raw HTTP transactions from Organizer, and author a comprehensive submission report following the provided template.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * Flawless documentation accelerates triage velocity, builds professional credibility with platform program managers, and prevents down-voting of issue severity.
* **🛡️ Defender Perspective:**
  * High-quality reports provide software engineers with the exact technical reproduction vector, eliminating back-and-forth communication delays and facilitating rapid patch deployment.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Severity Inflation:** Overstating benign findings (e.g., classifying Self-XSS or missing SSL ciphers as Critical), damaging your professional reputation with triage teams.
> 2. **Screenshot-Only Submissions:** Providing visual desktop screenshots without raw HTTP requests and responses, forcing triage engineers to spend unnecessary time requesting raw logs.
> 3. **Demonstrating Zero Practical Impact:** Presenting abstract proof (e.g., executing a harmless `alert(1)`) without explaining the real-world attack chain (e.g., hijacking session tokens or executing unauthorized API transactions).

---

## 📚 References

1. [FIRST CVSS v3.1 Specification Standard](https://www.first.org/cvss/v3.1/specification-document)
2. [HackerOne: How to Write an Impactful Vulnerability Report](https://www.hackerone.com/blog/how-write-good-vulnerability-report)
3. [Bugcrowd Vulnerability Rating Taxonomy (VRT)](https://bugcrowd.com/vulnerability-rating-taxonomy)

---

| [← Previous: 17. Pentest Workflow](./17-pentest-workflow.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 19. Common Beginner Mistakes →](./19-beginner-mistakes.md) |
