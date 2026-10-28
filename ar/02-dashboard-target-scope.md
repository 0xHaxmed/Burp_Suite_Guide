# 02. لوحة التحكم ونطاق الاستهداف (Dashboard, Target & Scope)

> 🌐 **اللغة / Language:** [English Version (`en/02-dashboard-target-scope.md`)](../en/02-dashboard-target-scope.md)

---

## 🎯 الهدف من الفصل

إتقان استخدام شاشة القيادة المركزية (`Dashboard`)، ورسم خريطة سطح الهجوم للتطبيق الهدف (`Site Map`) داخل تاب `Target`، واحتراف تعريف نطاق الاختبار (`Scope Rules`) للتخلص نهائياً من الضوضاء الناتجة عن ترافيك المتصفح والنطاقات الخارجية، وتفادي ارتكاب أخطاء اختبار نطاقات غير مصرح بها.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. تاب Dashboard (غرفة العمليات المركزية):
يحتوي Dashboard على أربعة أقسام أساسية تمنحك رؤية فورية لسير عملية الفحص:
* **Tasks:** إدارة المهام التي تعمل في الخلفية، مثل مهام الزحف التلقائي السلبي (Passive Crawl) أو مهام الفحص النشط المجدولة (`🔒 Pro`).
* **Issue Activity:** قائمة بالثغرات ونقاط الضعف المكتشفة، مصنفة حسب الخطورة (`High`, `Medium`, `Low`, `Information`) ودرجة التأكيد (`Certain`, `Firm`, `Tentative`).
* **Issue Advisory:** شرح موسع ومفصل عند الضغط على أي ثغرة، يوضح آلية عملها، الأدلة (Request/Response)، وتوصيات المعالجة البرمجية (Remediation).
* **Event Log:** السجل الزمني التشغيلي لأداة Burp (أخطاء الشبكة، توقف المنافذ، مشاكل الذاكرة).

### 2. تاب Target و Site Map:
يقوم تاب Target ببناء شجرة هرمية ديناميكية للموقع (`Site Map`) لكل رابط، ومجلد، ومسار، وباراميتر يتم اكتشافه أثناء تصفح التطبيق. يتم تمثيل الروابط غير المزارة بلون رمادي باهت بناءً على الروابط الموجودة داخل ملفات HTML/JS التي تم جلبها.

### 3. معضلة الضوضاء (The Noise Dilemma) وأهمية الـ Scope:
عند تصفح أي موقع ويب حديث (مثلاً `https://target-app.com`)، يُجري المتصفح تلقائياً عشرات الطلبات الخلفية لنطاقات خارجية تابعة لأطراف ثالثة (مثل تحليلات Google، وخطوط Typekit، وشبكات إعلانية، ومخدمات CDN، وتحديثات المتصفح مثل `detectportal.firefox.com`).
* **المشكلة:** ملء سجلات Burp بآلاف الطلبات التي لا علاقة لها بالهدف.
* **الخطر الأمني والقانوني:** قد يؤدي إرسال هجمات أو حمولات خبيثة (Payloads) ضد هذه النطاقات الخارجية إلى اختراق غير مصرح به أو حظر عنوان IP الخاص بك.
* **الحل:** تفعيل **Target Scope**؛ وهو مرشح صارم يحدد لـ Burp Suite النطاقات والمسارات المصرح لك قانونياً بفحصها فقط.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### الخطوة 1: استكشاف التطبيق وتوليد الـ Site Map
1. افتح المتصفح المرتبط بـ Burp وتصفح الموقع الهدف بشكل طبيعي (سجل دخول، تصفح المنتجات، أرسل تعليقاً).
2. انتقل إلى تاب `Target > Site map` لمشاهدة الشجرة الهيكلية لكافة الملفات والمسارات التي تمت زيارتها.

### الخطوة 2: إضافة الهدف إلى الـ Scope
1. في شجرة `Site map`، انقر بزر الماوس الأيمن على النطاق المستهدف (مثلاً `https://target-app.com`).
2. اختر **Add to scope**.
3. ستظهر لك رسالة تأكيدية تسألك: *"Do you want Proxy to stop sending out-of-scope items to HTTP history?"*
   * اختر **Yes** لمنع تراكم الطلبات غير المهمة في السجل.

### الخطوة 3: الضبط المتقدم للـ Scope Rules
1. اذهب إلى `Target > Scope settings`.
2. يمكنك استخدام مطابقة الـ Prefix أو التعبيرات النمطية (Advanced Regex Scope):
   * لإدراج كافة النطاقات الفرعية: `.*\.target-app\.com$`
   * لاستبعاد مسارات معينة تتسبب في تسجيل الخروج أثناء الفحص: أضف المسار التالي في جدول **Exclude from scope**:
     * `^https://target-app\.com/logout$`

### الخطوة 4: فلترة الواجهة لإخفاء الضوضاء
1. في تاب `Target > Site map` أو تاب `Proxy > HTTP history`، انقر على شريط الفلترة الرمادي أعلى الجدول (`Filter: Showing all items`).
2. فعّل الخيار الحاسم: **Show only in-scope items**.
3. (اختياري) يمكنك إخفاء ملفات الـ Static Assets عبر إلغاء تحديد: `CSS`, `Images`, `Flash`.
4. اضغط **Apply**. ستتحول الشاشة إلى واجهة نقية تعرض أهدافك فقط.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Target Scope Configuration & Site Map](https://portswigger.net/web-security/learning-paths)
  * تدرب على استكشاف معمل أكاديمي كامل، إضافة النطاق المخصص لك إلى الـ Scope، وتصفية السجلات لرصد الـ Endpoints غير الموثقة.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم (Pentester):**
  * ضبط الـ Scope هو أول ما يفعله المحترف قبل إطلاق أي هجوم، لضمان عدم توجيه حمولات استغلال (Exploit Payloads) لنطاقات خارج تفويض العقد (Out of Scope)، وهو ما يحميه من المسؤولية الجنائية.
* **🛡️ منظور المدافع (Security Engineer):**
  * يعتمد المدافع على الـ Site Map الكاملة للتطبيق لرصد "المسارات اليتيمة" (Orphaned / Hidden Endpoints) والواجهات البرمجية القديمة (Shadow APIs) المتروكة في بيئة الإنتاج دون حماية كافية.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **العمل بدون تحديد Scope:** بدء الفحص مع ترك خيار إظهار كافة النطاقات مفعلاً، مما يسبب تشتت الذهن وصعوبة استخراج مسارات الهجوم من بين آلاف الطلبات التابعة للمتصفح.
> 2. **اختبار نطاقات CDN أو خدمات التحليلات الخارجية:** إرسال طلبات فحص ثغرات لـ `google-analytics.com` أو `cdnjs.cloudflare.com` ظناً أنها جزء من الموقع، وهو خرق صريح لقواعد الاشتباك (Rules of Engagement).
> 3. **نسيان استبعاد رابط الـ Logout من النطاق:** عندما يقوم الـ Scanner الآلي بالدخول على رابط `logout`، يتم إنهاء الجلسة فوراً وتفشل كافة اختبارات الفحص اللاحقة.

---

## 📚 مراجع

1. [PortSwigger Documentation: Target Scope](https://portswigger.net/burp/documentation/desktop/tools/target/scope)
2. [PortSwigger Documentation: Site Map](https://portswigger.net/burp/documentation/desktop/tools/target/site-map)
3. [OWASP Testing Guide: Information Gathering](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/01-Information_Gathering/)

---

| [← السابق: 01. التثبيت والبروكسي](./01-setup-proxy-https.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 03. البروكسي المتقدم وقواعد المعالجة →](./03-proxy-advanced.md) |
