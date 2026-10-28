# 🛡️ The Ultimate Burp Suite Guide | الدليل الشامل لاحتراف Burp Suite

[![Language: English](https://img.shields.io/badge/Language-English-blue.svg)](#)
[![Version: 2026 Edition](https://img.shields.io/badge/Burp%20Suite-2026%20Edition-orange.svg)](#)
[![License: Educational](https://img.shields.io/badge/License-Educational%20Only-green.svg)](#)
[![Arabic Version](https://img.shields.io/badge/README-%D8%A7%D9%84%D8%B9%D8%B1%D8%A8%D9%8A%D8%A9-green.svg)](./README.md)

> 🌐 **Language Switcher:** [اقرأ هذا الدليل باللغة العربية (README.md)](./README.md) | [Browse Arabic Chapters (`ar/`)](./ar/)

An open-source, hands-on, and practical reference guide tailored for Web Application Penetration Testers, CTF players, and Security Engineers. Covering the complete **Burp Suite** ecosystem from core fundamentals to cutting-edge web exploitation techniques up to 2026.

---

> [!WARNING]
> **Legal & Ethical Disclaimer:**
> This content is strictly intended for educational purposes, authorized security assessments, and legal capture-the-flag competitions (Ethical Hacking / Authorized Penetration Testing / CTF). Using these tools or techniques against systems without explicit, prior written authorization is illegal. The author assumes no liability for any unauthorized use of this material.

---

## 🎯 Guide Philosophy

Most available documentation explains Burp Suite tools as disjointed tabs, leaving security practitioners struggling to connect the dots on actual penetration tests. This guide is built around three core pillars:

1. **Methodological Workflow:** How tools hand off data from one to another across a real-world assessment lifecycle.
2. **Deep Technical Insights:** Demystifying what happens under the hood at the network, transport, and HTTP/HTTPS layer without unnecessary fluff.
3. **Hands-on Labs:** Every chapter directly links to verified, real-world labs from **PortSwigger Web Security Academy** and **TryHackMe** for immediate validation.

---

## 🗂️ Table of Contents (English Version - `en/`)

| # | Chapter | Description & Topics Covered | Arabic Version |
|---|---------|------------------------------|----------------|
| **00** | [Burp Suite Editions Comparison](./en/00-editions-comparison.md) | Detailed comparison: Community vs. Professional vs. Enterprise, and core feature parity. | [العربية](./ar/00-editions-comparison.md) |
| **01** | [Setup, Proxy & CA Certificate](./en/01-setup-proxy-https.md) | MitM architecture, FoxyProxy configuration, Root CA installation, and SSL troubleshooting. | [العربية](./ar/01-setup-proxy-https.md) |
| **02** | [Dashboard, Target & Scope](./en/02-dashboard-target-scope.md) | Dashboard architecture, building Site Maps, and eliminating traffic noise with Scope rules. | [العربية](./ar/02-dashboard-target-scope.md) |
| **03** | [Advanced Proxy & Traffic Manipulation](./en/03-proxy-advanced.md) | Intercept mastery, HTTP History, Match & Replace rules, and Upstream Proxies / Tor routing. | [العربية](./ar/03-proxy-advanced.md) |
| **04** | [Repeater & Parallel Attacks](./en/04-repeater.md) | Targeted manual analysis, and exploiting Race Conditions via parallel Single-Packet Attacks. | [العربية](./ar/04-repeater.md) |
| **05** | [Intruder & Intelligent Automation](./en/05-intruder.md) | The 4 attack types, payload configurations, Grep Extract parsing, and CSRF token handling with Macros. | [العربية](./ar/05-intruder.md) |
| **06** | [Decoder, Comparer & Sequencer](./en/06-decoder-comparer-sequencer.md) | Data decoding & transformation, response diffing, and session token entropy analysis. | [العربية](./ar/06-decoder-comparer-sequencer.md) |
| **07** | [Logger & Organizer](./en/07-logger-organizer.md) | Full-fidelity traffic logging, analyzing WAF blocks, and archiving verified proof-of-concept evidence. | [العربية](./ar/07-logger-organizer.md) |
| **08** | [Burp Collaborator](./en/08-collaborator.md) `🔒 Pro` | Out-of-Band Application Security Testing (OAST) for Blind SSRF, Blind XSS, and OOB SQLi. | [العربية](./ar/08-collaborator.md) |
| **09** | [Burp Scanner](./en/09-scanner.md) `🔒 Pro` | Automated auditing: Crawl & Audit engine tuning, false-positive reduction, and avoiding denial-of-service. | [العربية](./ar/09-scanner.md) |
| **10** | [Embedded Browser & DOM Invader](./en/10-dom-invader-browser.md) | Utilizing pre-configured Chromium, DOM-based XSS debugging, Web Message testing, and Prototype Pollution. | [العربية](./ar/10-dom-invader-browser.md) |
| **11** | [Extensions & Montoya API](./en/11-extensions.md) | BApp Store curation, custom extension development, and essential tools (Autorize, Logger++, Turbo Intruder). | [العربية](./ar/11-extensions.md) |
| **12** | [Vulnerability Testing with Burp](./en/12-vulnerabilities-with-burp.md) | Hands-on walkthroughs: XSS, SQLi, SSRF, XXE, File Upload bypasses, and Insecure Deserialization. | [العربية](./ar/12-vulnerabilities-with-burp.md) |
| **13** | [Authentication, Session, JWT & OAuth](./en/13-auth-session-jwt-oauth.md) | Cookie security, JWT signature attacks, 2FA bypass logic, and OAuth 2.0 grant vulnerabilities. | [العربية](./ar/13-auth-session-jwt-oauth.md) |
| **14** | [Access Control & Authorization](./en/14-access-control.md) | Vertical & horizontal privilege escalation, IDOR exploitation, and automated matrix auditing with Autorize. | [العربية](./ar/14-access-control.md) |
| **15** | [API, GraphQL & WebSockets Testing](./en/15-api-graphql-websockets.md) | Intercepting RESTful endpoints, GraphQL introspection abuse, and real-time WebSocket tampering. | [العربية](./ar/15-api-graphql-websockets.md) |
| **16** | [HTTP/2 & Request Smuggling](./en/16-http2-request-smuggling.md) | Front-end vs back-end desynchronization, HTTP/2 downgrading anomalies, and HTTP Request Smuggler. | [العربية](./ar/16-http2-request-smuggling.md) |
| **17** | [Comprehensive Pentest Workflow](./en/17-pentest-workflow.md) | Visual Mermaid flowchart illustrating the complete end-to-end testing lifecycle from recon to final report. | [العربية](./ar/17-pentest-workflow.md) |
| **18** | [Security Reporting & Documentation](./en/18-reporting.md) | Professional vulnerability reporting, CVSS v3.1/v4.0 scoring, actionable PoC drafting, and remediation advice. | [العربية](./ar/18-reporting.md) |
| **19** | [Common Beginner Mistakes](./en/19-beginner-mistakes.md) | 15 critical pitfalls and anti-patterns encountered by junior testers and how to avoid them. | [العربية](./ar/19-beginner-mistakes.md) |
| **⌨️** | [Shortcuts Cheat Sheet](./en/shortcuts-cheatsheet.md) | High-speed keyboard shortcut reference for Windows, Linux, and macOS. | [العربية](./ar/shortcuts-cheatsheet.md) |
| **📚** | [Resources & Reference Material](./en/resources.md) | Authoritative books, recommended certifications, cheat sheets, and essential practice platforms. | [العربية](./ar/resources.md) |

---

## 🏷️ Badges & Callout Conventions

Throughout this repository, specific tags highlight critical operational information:

* `🔒 Pro`: Indicates features or capabilities exclusive to Burp Suite Professional or Enterprise editions.
* `> [!NOTE]`: Essential contextual background, mechanics, or architectural details.
* `> [!TIP]`: Productivity optimizations, keyboard shortcuts, or workflow accelerations.
* `> [!WARNING]`: Actions that may trigger account lockouts, WAF IP bans, or unintended server instability.
* `> [!IMPORTANT]`: Mandatory rules, operational safety guidelines, or industry testing standards.

---

## 💻 Prerequisites

To gain the highest value from this repository, familiarity with the following is recommended:
1. Fundamentals of the HTTP protocol (methods, headers, status codes, stateless nature).
2. Basic understanding of core web technologies: HTML, JavaScript, JSON, and client-server models.
3. A modern desktop browser (Firefox / Chromium) or Burp's pre-packaged Embedded Browser.
4. A free registered account on [PortSwigger Web Security Academy](https://portswigger.net/web-security) for interactive lab practice.

---

## 🤝 Contributing

Contributions, technical corrections, and enhancements are warmly welcome:
1. Fork the repository.
2. Create a descriptive feature branch (`git checkout -b feature/refine-chapter-x`).
3. Ensure technical terminology remains standardized and accurate across both Arabic (`ar/`) and English (`en/`) versions.
4. Open a well-documented Pull Request referencing relevant lab or documentation sources.

---

<p align="center">
<b>Happy (Ethical) Hacking & Secure Pentesting 🛡️</b>
</p>
