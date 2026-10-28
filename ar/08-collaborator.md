# 08. أداة Burp Collaborator وصيد الثغرات غير المرئية (OAST) `🔒 Pro`

> 🌐 **اللغة / Language:** [English Version (`en/08-collaborator.md`)](../en/08-collaborator.md)

---

## 🎯 الهدف من الفصل

فهم المفهوم الثوري لاختبار أمان التطبيقات خارج النطاق (**Out-of-Band Application Security Testing - OAST**)، واحتراف استخدام خادم **Burp Collaborator** في النسخة الاحترافية (`🔒 Pro`) لاكتشاف وتأكيد أخطر الثغرات "العمياء" (Blind Vulnerabilities) التي لا تظهر أي أثر مباشر في استجابات HTTP العادية، مثل **Blind SSRF**، و **Blind XXE**، و **Blind OS Command Injection**، و **Blind XSS**.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. معضلة الثغرات العمياء (The Blind Vulnerabilities Dilemma):
في الثغرات التقليدية، يرسل المهاجم الحمولة في الطلب وتعود النتيجة واضحة في الرد (In-band). ولكن في كثير من الأنظمة الحديثة:
* يعالج السيرفر مدخلاتك في طابور خلفي (Asynchronous Backend Queue).
* أو يرسل السيرفر طلباً داخلياً إلى خادم آخر دون إرجاع أي مخرجات للمستخدم.
* في هذه الحالات، لا يوجد أي خطأ `500` أو نص واضح يؤكد نجاح الحقن، مما يجعل أدوات الفحص التقليدية تفشل تماماً في رصدها.

### 2. معمارية Burp Collaborator ونموذج OAST:
يعمل Burp Collaborator كخادم وسيط خارجي مستقل تديره شركة PortSwigger (أو يمكن للمؤسسات استضافته كخادم خاص Private Collaborator) يمتلك نطاقاً مخصصاً (مثل `oastify.com`) ويدير الخدمات التالية:
* **خادم أسماء نطاقات مخصص (Authoritative DNS Server):** يسجل أي استعلام DNS يصل إليه بدقة.
* **خادم ويب (HTTP/HTTPS Server):** يسجل كل طلب اتصال وارد مع الترويسات ومصدر الـ IP.
* **خادم بريد إلكتروني (SMTP/SMTPS Server):** يرصد رسائل البريد الإلكتروني الناتجة عن حقن الترويسات.

```
[ Burp Suite GUI ] ---------------- (Polls for interactions) -----------------> [ Collaborator Server ]
        |                                                                                ^
  (Sends Payload:                                                                        |
   xyz.oastify.com)                                                                      |
        v                                                                                |
[ Vulnerable App ] -------- (App performs DNS lookup or HTTP request) -------------------+
```

### 3. دورة حياة الهجوم واكتشاف الثغرة:
1. يولد المختبر نطاقاً فرعياً فريداً وعشوائياً عبر Burp (مثلاً `a8f3b...oastify.com`).
2. يقوم بحقن هذا النطاق داخل باراميتر مشبوه (مثل رابط صورة في ملف PDF أو ترويسة `Referer`).
3. يقوم التطبيق المصاب بالاتصال بالنطاق للاستعلام عنه.
4. يسجل خادم Collaborator تفاصيل هذا الاتصال (عنوان IP السيرفر الداخلي، وقت الاتصال، نوع البروتوكول).
5. يقوم Burp Suite بالاستعلام الدوري (Polling) الآمن والمشفر من خادم Collaborator، وبمجرد رصد التفاعل، يظهر تنبيه فوري في لوحة التحكم مؤكداً وجود الثغرة بنسبة ثقة 100% بدون أي إنذار كاذب (Zero False Positives).

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: استخدام Collaborator Client يدوياً
1. في شريط الأدوات العلوي في Burp Pro، اضغط على **Burp > Burp Collaborator client** (أو من تاب `Tools`).
2. اضغط على زر **Copy to clipboard** لنسخ نطاق فرعي عشوائي فريد (سيكون بصيغة تشبه: `burpcollaborator.net` أو `oastify.com`).
3. اذهب إلى أداة **Repeater**، وحدد باراميتر مشبوه بتمرير روابط خارجية (مثل طلب تصدير تقرير أو تحميل صورة بروفايل عبر URL):
   ```http
   POST /api/fetch-avatar HTTP/1.1
   Host: target-app.com
   Content-Type: application/json

   {
     "avatar_url": "http://YOUR_COLLABORATOR_PAYLOAD_HERE"
   }
   ```
4. أرسل الطلب، ولاحظ أن السيرفر قد يرد بـ `200 OK` أو `{"status": "queued"}` بدون أي تفاصيل.
5. ارجع إلى نافذة **Burp Collaborator client** واضغط زر **Poll now**.
6. إذا ظهرت لك استعلامات **DNS** واتصالات **HTTP** قادمة من عنوان IP يتبع البنية التحتية للهدف، فهذا دليل قاطع على وجود ثغرة **Blind SSRF**.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Blind SSRF with out-of-band detection](https://portswigger.net/web-security/ssrf/blind/lab-out-of-band-detection)
  * تطبيق عملي مخصص لاستخدام Burp Collaborator في رصد استعلام DNS صادر من سيرفر تحليلات خلفي وتأكيد ثغرة Blind SSRF.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * Collaborator هو السلاح الأقوى لاكتشاف الثغرات الحرجة في الشركات الكبرى (Enterprise Targets) التي تستخدم معماريات Microservices معقدة، حيث يؤدي التفاعل البسيط إلى كشف عناوين السيرفرات الداخلية وأسماء النطاقات السحابية المحجوبة.
* **🛡️ منظور المدافع:**
  * لمنع هجمات OAST والـ SSRF، يجب تطبيق سياسات خروج صارمة (Egress Firewall Filtering) تمنع خوادم الويب من إجراء أي اتصالات عشوائية بالإنترنت الخارجي، وحظر استعلامات الـ DNS لخوادم غير مصرح بها.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **استخدام Collaborator في النسخة المجانية:** البحث عن Collaborator في Community Edition؛ الأداة غير متوفرة إطلاقاً في النسخة المجانية وبديلها الخارجي المجاني هو `Interactsh`.
> 2. **استخدام خوادم Collaborator العامة في اختبارات حكومية أو بنكية:** إرسال بيانات عملاء بنكيين عبر خوادم PortSwigger العامة قد يخالف اتفاقيات السرية (NDA) ولوائح الخصوصية (GDPR). في هذه الحالات، يجب تثبيت **Private Burp Collaborator Server** على خادم خاص بالشركة.
> 3. **عدم فحص سجلات الـ DNS والتركيز على HTTP فقط:** في كثير من الأحيان، تسمح جدران الحماية للهدف بإجراء استعلام DNS للتحقق من النطاق ولكنها تحظر اتصال الـ HTTP؛ رصد استعلام الـ DNS وحده كافٍ تماماً لإثبات وجود الثغرة.

---

## 📚 مراجع

1. [PortSwigger Research: Out-of-Band Application Security Testing (OAST)](https://portswigger.net/burp/application-security-testing/oast)
2. [PortSwigger Documentation: Burp Collaborator Overview](https://portswigger.net/burp/documentation/desktop/tools/collaborator)
3. [PortSwigger Documentation: Deploying a Private Collaborator Server](https://portswigger.net/burp/documentation/collaborator/deploying)

---

| [← السابق: 07. أدوات Logger و Organizer](./07-logger-organizer.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 09. أداة Burp Scanner →](./09-scanner.md) |
