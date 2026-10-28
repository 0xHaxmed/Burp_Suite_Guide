# 01. التثبيت وضبط البروكسي وشهادة CA (Setup, Proxy & CA Certificate)

> 🌐 **اللغة / Language:** [English Version (`en/01-setup-proxy-https.md`)](../en/01-setup-proxy-https.md)

---

## 🎯 الهدف من الفصل

تجهيز بيئة عمل متكاملة لأداة **Burp Suite**، وفهم كيفية عمل خادم البروكسي كـ *Man-in-the-Middle*، وربط المتصفحات الخارجية (Firefox / Chromium) به عبر ملحق `FoxyProxy`، وتثبيت شهادة المرجع المصدق (`PortSwigger CA Certificate`) لفك تشفير اتصالات الـ HTTPS دون ظهور أي تحذيرات أمان أو انقطاع في الفحص.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

يعمل تاب **Proxy** داخل Burp Suite كخادم وسيط محلي (Local HTTP/S Intercepting Proxy) يستقر بين متصفح الويب (Client) وخادم الويب (Server) على العنوان الافتراضي `127.0.0.1` والمنفذ `8080`.

```
[ Browser (Client) ] <---> [ Burp Suite Proxy (127.0.0.1:8080) ] <---> [ Web Server ]
```

### معضلة تشفير الـ HTTPS وفك التشفير (SSL/TLS Termination):
في الاتصال العادي عبر بروتوكول `HTTPS`، يُنشئ المتصفح نفقاً مشفراً باستخدام بروتوكول TLS مع الخادم الهدف، وتكون البيانات غير قابلة للقراءة أو التعديل لأي طرف ثالث.
لكي يتمكن Burp Suite من فحص محتويات الطلبات والردود وتعديلها، يتبع آلية **Man-in-the-Middle (MitM)** عبر الخطوات التالية:

1. **اعتراض طلب الاتصال (CONNECT Request):** عندما يطلب المتصفح فتح موقع مشفر (مثلاً `https://example.com`)، يرسل أمر `CONNECT example.com:443` إلى بروكسي Burp.
2. **إنشاء شهادة ديناميكية على الطاير (On-the-Fly Certificate):** يقوم Burp بتوليد شهادة SSL خاصة باسم النطاق المطلوب (`example.com`) ويوقعها باستخدام شهادة الجذر الخاصة به (`PortSwigger CA`).
3. **التحقق من الشهادة:** إذا لم تكن شهادة `PortSwigger CA` مثبتة ضمن قائمة مراجع الشهادات الموثوقة (Trusted Root Certification Authorities) داخل المتصفح أو نظام التشغيل، سيرفض المتصفح الاتصال فوراً ويظهر تحذير أمني صارم (`SEC_ERROR_UNKNOWN_ISSUER`).
4. **التشفير المزدوج:** بمجرد تثبيت الشهادة والوثوق بها، ينهي Burp اتصال TLS مع المتصفح، ويفك تشفير الترافيك، ثم يفتح اتصال TLS منفصل ومستقل تماماً مع الخادم الأصلي، مما يتيح لك رؤية النصوص البرمجية والبيانات الخام في واجهة Burp.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### الخطوة 1: تنزيل وتشغيل Burp Suite
1. قم بتحميل النسخة المتوافقة مع نظامك من موقع [PortSwigger الرسمي](https://portswigger.net/burp/releases).
2. عند فتح البرنامج لأول مرة:
   * اختر **Temporary project in memory** (في النسخة المجانية) أو **New project on disk** (في النسخة الاحترافية).
   * اختر **Use Burp defaults** ثم اضغط **Start Burp**.

### الخطوة 2: تهيئة البروكسي في متصفح Firefox عبر FoxyProxy
1. افتح متصفح Firefox وثبت إضافة **FoxyProxy Standard** من متجر الإضافات الرسمي.
2. انقر على أيقونة FoxyProxy > اختر **Options** > ثم اضغط **Add**.
3. أدخل البيانات التالية:
   * **Title / Description:** `Burp Suite`
   * **Proxy Type:** `HTTP`
   * **Proxy IP address or DNS name:** `127.0.0.1`
   * **Port:** `8080`
4. اضغط **Save**، ثم انقر على أيقونة الإضافة واختر تفعيل بروفايل `Burp Suite`.

### الخطوة 3: استخراج وتثبيت شهادة Burp CA Certificate
1. تأكد من تشغيل Burp وتفعيل بروكسي FoxyProxy في المتصفح.
2. افتح المتصفح واكتب في شريط العنوان: `http://burp` (أو `http://127.0.0.1:8080`).
3. اضغط على رابط **CA Certificate** في الزاوية العلوية اليمنى لتحميل ملف الشهادة باسم `cacert.der`.
4. في متصفح Firefox:
   * ادخل على **Settings** > اكتب في خانة البحث `Certificates` > اضغط على **View Certificates**.
   * اختر تاب **Authorities** > اضغط على زر **Import**.
   * اختر ملف `cacert.der` الذي قمت بتحميله.
   * **هام جداً:** ضع علامة صح أمام خيار: **Trust this CA to identify websites** ثم اضغط **OK**.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [TryHackMe: Burp Suite - The Basics (Task 5 & 6)](https://tryhackme.com/room/burpsuitebasics)
  * تطبيق عملي خطوة بخطوة لربط المتصفح، تنزيل الشهادة، والتقاط أول طلب HTTP بنجاح.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * البروكسي هو نقطة الانطلاق الأساسية لرؤية الحقيقة غير المفلترة؛ فكل قيود المتصفح والتحققات التي يراها المستخدم العادي تُكشف وتُلغى بمجرد وصول الطلب للبروكسي.
* **🛡️ منظور المدافع:**
  * إدراك آلية عمل البروكسي يبرهن للمطورين وفِرق الدفاع أن أي حماية يتم تطبيقها على مستوى المتصفح (Client-side) مثل JavaScript validation أو التشفير المحلي للأشكال يمكن تجاوزها بنسبة 100%، وأن صمام الأمان الوحيد يجب أن يكون مبنياً بالكامل داخل السيرفر (Server-side validation).

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **نسيان اختيار خيار الثقة بالشهادة:** استيراد شهادة `cacert.der` دون تفعيل خيار `Trust this CA to identify websites` سيستمر في إظهار رسائل الخطأ التحذيرية في كل موقع HTTPS تفتحه.
> 2. **تعارض المنافذ (Port 8080 Conflict):** إذا كان هناك برنامج آخر (مثل خادم محلي XAMPP أو Apache أو Tomcat) يستخدم المنفذ 8080، سيفشل Burp في تشغيل الـ Proxy Listener. الحل: اذهب إلى `Proxy > Proxy settings > Proxy listeners` وغيّر المنفذ إلى `8081` أو `8888` وقم بتحديث FoxyProxy بنفس الرقم.
> 3. **ترك البروكسي مفعل أثناء التصفح الشخصي:** يؤدي ذلك إلى توقف الإنترنت إذا كان Burp مغلقاً، أو تسجيل بيانات تصفحك الخاصة وكلمات سر حساباتك الشخصية داخل سجل Burp دون قصد.

---

## 📚 مراجع

1. [PortSwigger: Installing Burp's CA certificate in Firefox](https://portswigger.net/burp/documentation/desktop/external-browser-config/certificate/firefox)
2. [Mozilla Developer Network: How SSL/TLS Certificates Work](https://developer.mozilla.org/en-US/docs/Glossary/Certificate)
3. [FoxyProxy Official Documentation](https://getfoxyproxy.org/)

---

| [← السابق: 00. مقارنة النسخ](./00-editions-comparison.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 02. لوحة التحكم و Scope →](./02-dashboard-target-scope.md) |
