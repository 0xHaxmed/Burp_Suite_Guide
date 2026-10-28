# 04. أداة Repeater وهجمات التوازي (Repeater & Parallel Attacks)

> 🌐 **اللغة / Language:** [English Version (`en/04-repeater.md`)](../en/04-repeater.md)

---

## 🎯 الهدف من الفصل

إتقان استخدام أداة **Repeater** كبيئة عمل مركزة للتعديل والتحليل اليدوي الدقيق لطلبات HTTP/S الفردية، وفهم كيفية استخدام ميزة مجموعات التاب (`Tab Groups`) لإرسال الطلبات المتزامنة واختبار ثغرات التزامن (**Race Conditions**) باستخدام تقنية هجوم الحزمة الواحدة (**Single-Packet Attack**)، بدلاً من الاعتماد الخاطئ على أدوات غير مخصصة لذلك.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. فلسفة Repeater (التحليل اليدوي المتأني):
بينما تركز أداة `Intruder` على إرسال آلاف الطلبات آلياً وبشكل أعمى، تمنحك `Repeater` القدرة على عزل طلب واحد، وتعديل حرف واحد في الترويسات أو جسم الطلب، وإعادة إرساله مئات المرات مع مراقبة وتحليل الفروق الدقيقة في الرد:
* **واجهة العرض:** تعديل الطلب على اليسار، ومشاهدة الاستجابة على اليمين مع خيارات عرض متعددة (`Pretty`, `Raw`, `Hex`, `Render`).
* **لوحة Inspector:** تفكيك الطلب إلى عناصر جدولية سهلة القراءة (Request Attributes, Query Parameters, Cookies, Headers) تتيح التعديل والتشفير المباشر دون الحاجة لكتابة التنسيقات يدوياً.

### 2. هجمات سباق البيانات (Race Conditions) وتقنية الحزمة الواحدة (Single-Packet Attack):
تحدث ثغرة **Race Condition** عندما يتلقى السيرفر عدة طلبات متزامنة في نافذة زمنية بالغة الصغر (بضعة أجزاء من الألف من الثانية) لمعالجة نفس المورد المشترك (مثل استرداد رمز خصم مالي، أو تحويل رصيد، أو التصويت مرتين):
* **الخطأ الشائع:** الاعتماد على أداة Battering Ram في Intruder لفحص Race Conditions (وهو خطأ تقني لأن Battering Ram ترسل الطلبات تباعاً ولا تضمن وصولها المتزامن).
* **الحل الاحترافي في Repeater:**
  * تنظيم الطلبات المتشابهة في مجموعة تابات (`Tab Group`).
  * تفعيل خيار **Send group in parallel**.
  * في بروتوكول HTTP/2، يستخدم Burp تقنية **Single-Packet Attack**: حيث يتم دمج جميع طلبات المجموعة داخل حزمة TCP واحدة (أو كتم البايت الأخير عبر HTTP/1.1 Last-Byte Sync وإرسال كل البايتات الأخيرة دفعة واحدة)، مما يضمن وصول الطلبات ومعالجتها داخل أنوية المعالج في الخادم في نفس اللحظة بالمللي ثانية، مما يفجر نافذة السباق (Race Window).

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: التحليل اليدوي وكشف ثغرة SQL Injection
1. من سجل `Proxy > HTTP history`، حدد الطلب الذي يحمل باراميتر البحث (مثلاً `GET /products?category=Gifts`).
2. اضغط اختصار `Ctrl + R` (أو كليك يمين > **Send to Repeater**).
3. أضف علامة اقتباس أحادية `'` في نهاية الباراميتر:
   ```http
   GET /products?category=Gifts' HTTP/1.1
   Host: target-app.com
   ```
4. اضغط **Send**؛ إذا عاد الخادم برمز خطأ `500 Internal Server Error` أو كشف رسالة خطأ لقاعدة البيانات، فهذا مؤشر قوي على وجود ثغرة.
5. ابدأ بناء استعلام الـ `UNION SELECT` تدريجياً لتحديد عدد الأعمدة:
   ```http
   GET /products?category=Gifts' UNION SELECT NULL,NULL-- HTTP/1.1
   Host: target-app.com
   ```

### التمرين 2: استغلال Race Condition عبر إرسال الطلبات بالتوازي (Parallel Group)
1. في متجر إلكتروني يمنح كوبون خصم بنسبة 20%، التقط طلب تطبيق الكوبون في Proxy وأرسله إلى Repeater:
   ```http
   POST /cart/apply-coupon HTTP/1.1
   Host: target-app.com
   Content-Type: application/x-www-form-urlencoded

   csrf=token123&coupon=DISCOUNT20
   ```
2. اضغط كليك يمين على تاب الطلب في Repeater واختر **Duplicate** وكرر العملية لإنشاء 10 إلى 20 نسخة من الطلب.
3. اضغط على أيقونة المجلد أو كليك يمين على التابات واختر **Create tab group** وسمها `Coupon Race`.
4. اضغط على السهم الصغير المجاور لزر **Send** في المجموعة، واختر: **Send group in parallel**.
5. راجع الردود: إذا ظهرت استجابات `200 OK` متعددة وتم تطبيق الخصم 5 مرات بدل مرة واحدة، فهذا يثبت نجاح الاستغلال عبر الـ Race Condition.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Limit overrun race conditions](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun)
  * تطبيق عملي مخصص لاستخدام ميزة إرسال الطلبات بالتوازي في Repeater لشراء منتج باستخدام كوبون تخفيض لمرات متعددة متزامنة.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * تمنح أداة Repeater المهاجم السيطرة المطلقة على كل بايت؛ حيث يمكنه فحص المنطق التجاري (Business Logic Flaws) والتلاعب بالحقول بدون تشويش أو ضغط غير محسوب على السيرفر.
* **🛡️ منظور المدافع:**
  * لمواجهة ثغرات الـ Race Conditions، لا يكفي التحقق على مستوى الكود العادي؛ بل يجب استخدام آليات إقفال قواعد البيانات المتزامنة (**Database Mutex Locks**) أو العمليات الذرية (**Atomic Transactions**) لمنع معالجة طلبين في آن واحد على نفس السجل.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **الاعتماد على Battering Ram لاختبار Race Conditions:** هجمات Battering Ram ليست متزامنة شبكياً ولا تضمن التزامن في وصول الحزم، الأداة الصحيحة هي إرسال التوازي في Repeater أو إضافة `Turbo Intruder`.
> 2. **تعديل نص الطلب مع إهمال ترويسة Content-Length:** عند تغيير طول الجسم يدوياً في Raw View دون تحديث الترويسة، قد يرفض السيرفر الطلب (لحسن الحظ، يحدّث Burp قيمة Content-Length افتراضياً، ولكن يجب الحذر عند تعطيل التحديث التلقائي).
> 3. **تراكم مئات التابات غير المنظمة:** فتح عشرات التابات في Repeater بدون تسمية أو تنظيم في مجموعات (Tab Groups) يؤدي لضياع نقاط الضعف وصعوبة العثور على الأدلة.

---

## 📚 مراجع

1. [PortSwigger Research: Smashing the state machine - The true potential of web race conditions](https://portswigger.net/research/smashing-the-state-machine)
2. [PortSwigger Documentation: Burp Repeater](https://portswigger.net/burp/documentation/desktop/tools/repeater)
3. [James Kettle: Single-Packet Attack Methodology](https://portswigger.net/research/single-packet-attack-the-true-impact-of-parallel-requests)

---

| [← السابق: 03. البروكسي المتقدم وقواعد المعالجة](./03-proxy-advanced.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 05. أداة Intruder والأتمتة الذكية →](./05-intruder.md) |
