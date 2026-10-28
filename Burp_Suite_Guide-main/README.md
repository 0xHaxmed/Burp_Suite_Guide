# 🛡 The Complete Burp Suite Guide (in Arabic)

A structured, cumulative walkthrough of **Burp Suite** — from installation to every core tool — written in Arabic for Arabic-speaking learners, with real hands-on examples (SQL Injection, IDOR, Credential Stuffing...) and standard English technical terminology kept inline, as is common in the cybersecurity field.

> ⚠️ **Disclaimer:** This content is for educational and ethical purposes only (Ethical Hacking / CTF / authorized labs). Only use what you learn here on systems you have explicit permission to test.

---

## 📖 Guide Contents

| # | Section | Covers |
|---|---------|--------|
| 1 | Installing Burp Suite & connecting it to Firefox | Download, FoxyProxy setup, CA certificate for HTTPS |
| 2 | Dashboard | Tasks, auto-detected issues, Event log |
| 3 | The Proxy Concept | How Burp sits as a man-in-the-middle between browser and server |
| 4 | Target | Defining and filtering Scope |
| 5 | Advanced Proxy | Intercept, HTTP History, WebSockets, Proxy Settings |
| 6 | Repeater | Manual request editing/resending + a full Union SQL Injection walkthrough |
| 7 | Intruder | Automation & fuzzing — all attack types (Sniper/Battering Ram/Pitchfork/Cluster Bomb), payload types, Grep Match/Extract |
| 8 | Decoder | Encoding, decoding, and hashing |
| 9 | Comparer | Comparing responses |
| 10 | Sequencer | Token entropy analysis & Session Fixation |
| 11 | Logger | Detailed logging of all traffic |
| 12 | Organizer | Archiving requests and evidence |
| 13 | Extensions | Extending Burp via the BApp Store |
| 14 | Full comparison table | Quick side-by-side comparison of every tool |
| 15 | Workflow | How all the tools tie together in a real pentest scenario |

---

## 📂 Files

- [`Burp_Suite_Guide_Arabic.pdf`](./Burp_Suite_Guide_Arabic.pdf) — the complete guide in PDF format, ready to read or print.

---

## 🎯 Who is this for?

- Beginners who want to understand Burp Suite as one connected workflow instead of scattered facts.
- Anyone studying Web Application Penetration Testing or preparing for certifications like OSCP / eWPT, or working through TryHackMe's Burp Suite rooms.
- Arabic speakers who'd rather have a single written reference than piece things together from scattered videos.

## 🧠 How to use it

Best read in order front-to-back, since it's built cumulatively — each tool builds on the understanding from the one before it. If you already have the basics down, jump straight to the section you need using the table of contents at the start of the PDF.

## 🙏 Sources

This content draws on:
- The official [TryHackMe](https://tryhackme.com) Burp Suite rooms (Repeater, Intruder, Other Modules, Extensions).
- Hands-on walkthrough videos covering Burp setup, browser integration, and tab-by-tab exploration.
- Official [PortSwigger](https://portswigger.net/burp/documentation) documentation.

## 📜 License

This content is free to use and share for personal, educational purposes. If you reuse or adapt it, please credit the source.

---

<p align="center">Happy (Ethical) Hacking 🕵️‍♂️</p>
