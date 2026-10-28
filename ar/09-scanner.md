# 09. أداة Burp Scanner والفحص الآلي الذكي (Burp Scanner) `🔒 Pro`

> 🌐 **اللغة / Language:** [English Version (`en/09-scanner.md`)](../en/09-scanner.md)

---

## 🎯 الهدف من الفصل

إتقان استخدام محرك الفحص الآلي **Burp Scanner** المتوفر حصرياً في النسخة الاحترافية (`🔒 Pro`)، وفهم الفارق الهندسي بين محرك الزحف (**Crawl**) ومحرك التدقيق الأمني (**Audit**)، وضبط إعدادات الفحص المخصصة لاكتشاف الثغرات العميقة مع تجنب التسبب في انهيار خوادم الإنتاج (Denial of Service) أو ملء قواعد البيانات بالبيانات العشوائية.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

يعمل Burp Scanner كمنظومة فحص هجينة متطورة تنقسم إلى مرحلتين رئيسيتين:

### 1. محرك الزحف (The Crawl Engine):
على عكس أدوات الزحف البدائية القديمة التي تبحث فقط عن روابط `<a href>`, يمتلك Burp Scanner متصفح Chromium حقيقي يعمل بدون واجهة (Headless Browser) مدمج في قلب الأداة:
* يقوم بتنفيذ أكواد JavaScript المعقدة وتطبيقات الصفحة الواحدة (**Single Page Applications - SPAs**) مثل Angular و React و Vue.js.
* يتتبع تغيرات الـ DOM والأحداث التفاعلية (`onclick`, `onmouseover`, form submissions).
* يبني خريطة دقيقة لكل الحالات الانتقالية لحالة التطبيق (State Machine).

### 2. محرك التدقيق (The Audit Engine):
بمجرد اكتشاف نقاط الإدخال والـ Endpoints، يبدأ محرك التدقيق بإجراء نوعين من الفحص:
* **الفحص السلبي (Passive Audit):** يحلل ترويسات الاستجابات وملفات الكود بدون إرسال أي طلبات جديدة (لكشف ترويسات الأمان المفقودة، تسريب معلومات الإصدارات، كوكيز بدون حماية `Secure` أو `HttpOnly`).
* **الفحص النشط (Active Audit):** يحقن حمولات أمنية ذكية في كل باراميتر وترويسة، ويراقب التغيرات في الاستجابة أو يتكامل مع **Burp Collaborator** لرصد الثغرات غير المباشرة.

### 3. الفحص المصادق عليه (Authenticated Scanning):
يتيح للمحرك فحص ما وراء شاشات تسجيل الدخول عبر طريقتين:
* **Recorded Login Sequences:** تسجيل حركة الدخول بالفيديو والمتصفح المدمج لحل نماذج تسجيل الدخول المعقدة.
* **Username/Password Form Auto-fill:** تعبئة النماذج تلقائياً واستخدام الـ Session Handling Rules لتجديد الجلسة في حال انتهاء صلاحيتها.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### الخطوة 1: إطلاق فحص مخصص ومحدد (Targeted Scan)
1. في شجرة `Target > Site map`، انقر بزر الماوس الأيمن على المسار الذي تريد فحصه فقط (مثلاً `/api/v2/user`).
2. اختر **Scan** لفتح نافذة إعدادات الفحص المتقدمة.
3. اختر نوع الفحص:
   * **Crawl and Audit:** لفحص المسار واستكشاف صفحاته الفرعية.
   * **Audit selected items:** لفحص الطلبات المحددة فقط دون زحف إضافي (أسرع وأقل خطورة).

### الخطوة 2: تخصيص إعدادات التدقيق الأمني (Scan Configurations)
1. في نافذة الفحص، انتقل إلى قسم **Scan configuration** واضغط **New**:
2. تحت قسم **Issues reported**:
   * اختر نوع الثغرات المستهدفة بدقة (مثلاً: حدد ثغرات SQLi و SSRF فقط وعطل باقي الثغرات لتقليل زمن الفحص).
3. تحت قسم **Handling application errors**:
   * حدد الحد الأقصى للمحاولات الفاشلة قبل إيقاف الفحص لمنع إغراق السيرفر بالطلبات المتكررة عند حدوث بطء في الاستجابة.

### الخطوة 3: حماية بيئات الإنتاج من أضرار الفحص
1. اذهب إلى إعدادات الفحص تحت **Audit optimization**:
   * اختر سرعة الفحص: `Fast` أو `Normal`.
   * اختر دقة التحقق من الثغرات: `Minimize false negatives` (لعدم تفويت أي ثغرة) أو `Minimize false positives`.
2. **قاعدة الأمان الذهبية:** تحت خيارات حقن البيانات، تجنب تفعيل فحص ثغرات تعديل البيانات الصامتة أو نماذج الحذف (`DELETE requests`) على خوادم حية.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Discovering vulnerabilities using Burp Scanner](https://portswigger.net/web-security/learning-paths)
  * أطلق فحصاً نشطاً عبر Burp Pro على معمل مصاب لاستخراج تقرير ثغرات كامل وتحليل مسار الاستغلال التلقائي.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يوفر الماسح مئات الساعات في اكتشاف الثغرات الشائعة (Low-hanging fruit)، مما يفرغ وقت المهاجم للتركيز على ثغرات المنطق التجاري (Business Logic) التي لا تفهمها الخوارزميات الآلية.
* **🛡️ منظور المدافع:**
  * الفحص الآلي المستمر هو خط الدفاع الأول لاكتشاف أخطاء التكوين البرمجي والمكتبات القديمة الضعيفة قبل أن يستغلها المهاجمون.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **إطلاق Crawl كامل على موقع إنتاج حي ضخم:** الزحف الأعمى قد يؤدي لضغط هائل على قاعدة البيانات، أو النقر التلقائي على أزرار "حذف الحساب" أو إرسال آلاف رسائل البريد الإلكتروني للمستخدمين الحقيقيين عبر استمارات الاتصال.
> 2. **الاعتماد الكامل على الفحص الآلي (Point & Click):** الثقة العمياء في تقرير Scanner؛ فالماسح الآلي يعجز تماماً عن فهم ثغرات الصلاحيات (Access Control) والمنطق المالي للتطبيقات.
> 3. **تجاهل الثغرات ذات التصنيف Tentative:** إهمال النتائج التي يضع عليها Burp وسم "محتملة"، في حين أنها غالباً ما تكون ثغرة حقيقية تحتاج فقط إلى تعديل يدوي بسيط في الـ Payload في أداة Repeater لإثباتها.

---

## 📚 مراجع

1. [PortSwigger Documentation: Burp Scanner Overview](https://portswigger.net/burp/documentation/desktop/tools/scanner)
2. [PortSwigger Documentation: Custom Scan Configurations](https://portswigger.net/burp/documentation/desktop/tools/scanner/scan-configurations)
3. [PortSwigger Documentation: Authenticated Scanning and Login Recorder](https://portswigger.net/burp/documentation/desktop/tools/scanner/authenticated-scanning)

---

| [← السابق: 08. أداة Burp Collaborator](./08-collaborator.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 10. المتصفح المدمج و DOM Invader →](./10-dom-invader-browser.md) |
