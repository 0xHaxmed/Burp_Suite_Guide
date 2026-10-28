# 🛡️ الدليل الشامل لاحتراف Burp Suite | The Ultimate Burp Suite Guide

[![Language: Arabic](https://img.shields.io/badge/Language-Arabic%20%2F%20%D8%B9%D8%B1%D8%A8%D9%8A-blue.svg)](#)
[![Version: 2026 Edition](https://img.shields.io/badge/Burp%20Suite-2026%20Edition-orange.svg)](#)
[![License: Educational](https://img.shields.io/badge/License-Educational%20Only-green.svg)](#)
[![English Version](https://img.shields.io/badge/README-English-red.svg)](./README.en.md)

> 🌐 **Language Switcher:** [Read this guide in English (README.en.md)](./README.en.md) | [تصفح النسخة الإنجليزية للفصول (`en/`)](./en/)

دليل مرجعي تطبيقي وعملي مفتوح المصدر مصمم لمختبري اختراق تطبيقات الويب (Web Penetration Testers)، ولاعبي مسابقات الـ CTF، والمهندسين الأمنيين، يغطي منظومة أداة **Burp Suite** كاملة من المفاهيم التأسيسية حتى أحدث تقنيات الفحص المتقدمة لعام 2026.

---

> [!WARNING]
> **إخلاء مسؤولية أخلاقي وقانوني (Legal & Ethical Disclaimer):**
> هذا المحتوى مخصص للأغراض التعليمية واختبارات الاختراق المصرح بها قانونياً فقط (Ethical Hacking / Authorized Penetration Testing / CTF). استخدام هذه التقنيات أو الأدوات ضد أي أهداف بدون إذن خطي مسبق وصريح يُعد جريمة يعاقب عليها القانون. الكاتب غير مسؤول عن أي استخدام غير مشروع لمحتويات هذا الدليل.

---

## 🎯 فلسفة هذا الدليل

معظم الشروحات المتوفرة تشرح أدوات Burp Suite كأقسام منعزلة، مما يترك الدارس عاجزاً عن تشكيل رؤية شاملة في بيئات العمل الحقيقية. يرتكز هذا الدليل على ثلاثة محاور:

1. **الترابط المنهجي (Methodological Workflow):** كيف تسلّم كل أداة مخرجاتها للأداة التالية في سيناريو اختراق واقعي.
2. **العمق التقني بدون تعقيد:** شرح ما يحدث تحت الغطاء (Under the Hood) على مستوى الشبكة والـ HTTP/S، مع تثبيت المصطلحات التقنية الإنجليزية بموقعها.
3. **التطبيق العملي المباشر:** كل فصل مربوط بمعامل حقيقية من **PortSwigger Web Security Academy** و **TryHackMe** لتطبيق المفاهيم فوراً.

---

## 🗂️ فهرس الفصول (النسخة العربية - `ar/`)

| # | الفصل | الوصف والمحتوى | النسخة الإنجليزية |
|---|-------|----------------|-------------------|
| **00** | [مقارنة نسخ Burp Suite](./ar/00-editions-comparison.md) | مقارنة تفصيلية بين Community و Professional و Enterprise، والفرق الجوهري بينها. | [English](./en/00-editions-comparison.md) |
| **01** | [التثبيت وضبط البروكسي وشهادة CA](./ar/01-setup-proxy-https.md) | إعداد Burp كـ MitM، ضبط FoxyProxy، تثبيت شهادة Burp CA، وتجاوز قيود SSL. | [English](./en/01-setup-proxy-https.md) |
| **02** | [لوحة التحكم ونطاق الاستهداف Scope](./ar/02-dashboard-target-scope.md) | فهم Dashboard، بناء خريطة الموقع Site Map، واستبعاد الضوضاء (Noise Filtering). | [English](./en/02-dashboard-target-scope.md) |
| **03** | [البروكسي المتقدم وقواعد المعالجة](./ar/03-proxy-advanced.md) | ضبط Intercept، سجل الـ HTTP History، قواعد Match & Replace، واستخدام Upstream Proxies / Tor. | [English](./en/03-proxy-advanced.md) |
| **04** | [أداة Repeater وهجمات التوازي](./ar/04-repeater.md) | التحليل اليدوي المركز، وفحص Race Conditions باستخدام Parallel Single-Packet Attack. | [English](./en/04-repeater.md) |
| **05** | [أداة Intruder والأتمتة الذكية](./ar/05-intruder.md) | أنواع الهجمات الأربعة، أنواع الـ Payloads، استخراج القيم عبر Grep Extract، وتخطي CSRF بالـ Macros. | [English](./en/05-intruder.md) |
| **06** | [أدوات Decoder و Comparer و Sequencer](./ar/06-decoder-comparer-sequencer.md) | فك الترميز وتجاوز الدفاعات، مقارنة الاستجابات بدقة، وتحليل عشوائية الجلسات (Entropy). | [English](./en/06-decoder-comparer-sequencer.md) |
| **07** | [أدوات Logger و Organizer](./ar/07-logger-organizer.md) | تتبع الطلبات وتحليل تعامل الجدران النارية (WAF)، وأرشفة أدلة الاختراق (Evidence). | [English](./en/07-logger-organizer.md) |
| **08** | [أداة Burp Collaborator](./ar/08-collaborator.md) `🔒 Pro` | صيد ثغرات Out-of-Band (OAST) مثل Blind SSRF، و Blind XSS، و Out-of-Band SQLi. | [English](./en/08-collaborator.md) |
| **09** | [أداة Burp Scanner](./ar/09-scanner.md) `🔒 Pro` | أسرار الفحص الآلي: ضبط محرك الـ Crawl و الـ Audit، تقليل الإنذارات الكاذبة وحماية التطبيقات من الانهيار. | [English](./en/09-scanner.md) |
| **10** | [المتصفح المدمج و DOM Invader](./ar/10-dom-invader-browser.md) | استخدام المتصفح المعزول، واكتشاف ثغرات DOM-based XSS و Web Message و Prototype Pollution. | [English](./en/10-dom-invader-browser.md) |
| **11** | [إضافات Burp ومنصة Montoya API](./ar/11-extensions.md) | متجر BApp، كتابة الإضافات، وأهم الإضافات الاحترافية (Autorize, Logger++, Turbo Intruder, Hackvertor). | [English](./en/11-extensions.md) |
| **12** | [تطبيق Burp على أشهر ثغرات الويب](./ar/12-vulnerabilities-with-burp.md) | ورش عملية لاكتشاف واستغلال: XSS, SQLi, SSRF, XXE, File Upload, و Deserialization. | [English](./en/12-vulnerabilities-with-burp.md) |
| **13** | [اختبار المصادقة والجلسات و JWT و OAuth](./ar/13-auth-session-jwt-oauth.md) | فحص الـ Cookies، فك واختراق توكنات JWT، وتجاوز آليات التحقق بخطوتين (2FA) وعيوب OAuth. | [English](./en/13-auth-session-jwt-oauth.md) |
| **14** | [اختبار صلاحيات الوصول و IDOR](./ar/14-access-control.md) | كشف ثغرات الترقية الرأسية والأفقية، وأتمتة الكشف بالكامل عبر أداة `Autorize`. | [English](./en/14-access-control.md) |
| **15** | [فحص واجهات البرمجة API و GraphQL و WebSockets](./ar/15-api-graphql-websockets.md) | اعتراض وتحليل REST APIs، استغلال Introspection في GraphQL، وفحص قنوات الـ WebSockets. | [English](./en/15-api-graphql-websockets.md) |
| **16** | [ثغرات HTTP/2 و Request Smuggling](./ar/16-http2-request-smuggling.md) | فهم انهيار الحدود بين السيرفرات (Front-end/Back-end)، واستغلال HTTP Request Smuggler. | [English](./en/16-http2-request-smuggling.md) |
| **17** | [منهجية الفحص الشاملة (Pentest Workflow)](./ar/17-pentest-workflow.md) | خريطة تدفق بصرية (Mermaid Flowchart) توضح تسلسل خطوات الاختبار من الصفر حتى التقرير. | [English](./en/17-pentest-workflow.md) |
| **18** | [كتابة التقرير الأمني وتوثيق الثغرات](./ar/18-reporting.md) | هيكل التقرير القياسي، تصنيف خطورة الثغرة بـ CVSS v3.1/v4.0، صياغة الـ PoC، وتوصيات الترقيع. | [English](./en/18-reporting.md) |
| **19** | [أشهر أخطاء المبتدئين](./ar/19-beginner-mistakes.md) | 15 خطأ يقع فيها مختبرو الاختراق الجدد وكيف تتفاداها كالمحترفين. | [English](./en/19-beginner-mistakes.md) |
| **⌨️** | [ورقة غش الاختصارات](./ar/shortcuts-cheatsheet.md) | دليل الاختصارات اليومية السريعة للوحة المفاتيح (Windows / Linux / macOS). | [English](./en/shortcuts-cheatsheet.md) |
| **📚** | [المصادر والمراجع المعتمدة](./ar/resources.md) | الكتب المرجعية، الدورات التدريبية الموصى بها، والـ Cheat Sheets الرسمية. | [English](./en/resources.md) |

---

## 🏷️ دليل العلامات والتنبيهات

* `🔒 Pro`: ميزة أو أداة حصرية للنسخ المدفوعة (Burp Suite Professional / Enterprise).
* `> [!NOTE]`: معلومة توضيحية معمقة أو سياق تقني إضافي.
* `> [!TIP]`: نصيحة لتسريع عملك أثناء الفحص أو اختصار في لوحة المفاتيح.
* `> [!WARNING]`: تنبيه من إجراء قد يتسبب في حظر حسابك أثناء الفحص أو انهيار الخادم الهدف.
* `> [!IMPORTANT]`: قاعدة ذهبية لا يجب تجاهلها في منهجيات الفحص الأمني.

---

## 💻 المتطلبات المسبقة (Prerequisites)

1. فهم أساسيات بروتوكول HTTP (الطلبات، الاستجابات، Status Codes، الـ Headers الشائعة).
2. إلمام أولي بلغات وتقنيات الويب الأساسية: HTML, JavaScript, JSON.
3. متصفح ويب حديث (Mozilla Firefox أو Google Chrome) أو استخدام المتصفح المدمج داخل Burp.
4. حساب مجاني على [PortSwigger Web Security Academy](https://portswigger.net/web-security) للوصول للمعامل التطبيقية المذكورة في نهاية كل فصل.

---

## 🤝 المساهمة (Contributing)

نرحب بأي تصحيحات أو إضافات تسهم في إثراء هذا المرجع:
1. قم بعمل `Fork` للمستودع.
2. أنشئ فرعاً جديداً لتعديلك (`git checkout -b feature/update-chapter-x`).
3. أرسل `Pull Request` يوضح سبب التعديل ومصدر المعلومة إن أمكن.

---

<p align="center">
<b>Happy (Ethical) Hacking & Secure Pentesting 🛡️</b>
</p>
