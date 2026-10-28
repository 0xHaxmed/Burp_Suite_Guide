# 06. أدوات المعالجة والمقارنة وفحص العشوائية (Decoder, Comparer & Sequencer)

> 🌐 **اللغة / Language:** [English Version (`en/06-decoder-comparer-sequencer.md`)](../en/06-decoder-comparer-sequencer.md)

---

## 🎯 الهدف من الفصل

إتقان استخدام أدوات الدعم التحليلي الثلاث داخل Burp Suite: تحويل وتشفير البيانات عبر **Decoder** وفهم حدود استخدامها، والمقارنة البصرية الدقيقة بين الاستجابات عبر **Comparer**، وإجراء الفحص الإحصائي لعشوائية التوكنات ومعرفات الجلسة (**Entropy Testing**) عبر أداة **Sequencer** لكشف ثغرات التنبؤ بالجلسات و **Session Fixation**.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. أداة Decoder (محول البيانات السريع):
تعمل أداة `Decoder` كلوحة تحويل بيانات فورية تدعم الترميز (Encode)، وفك الترميز (Decode)، والتجزئة (Hash):
* **الصيغ المدعومة:** `URL`, `HTML`, `Base64`, `ASCII Hex`, `Hex`, `Octal`, `Binary`, `Gzip`.
* **ميزة التراكم (Stacking):** تطبيق عدة طبقات ترميز متتالية فوق بعضها (مثلاً: النص الأصلي ← Base64 ← URL Encode).
* **Smart Decode:** ميزة استكشاف تلقائي تحاول التعرف الذكي على نوع الترميز وفكه دون تدخل يدوي.
* **⚠️ توضيح فني هام بخصوص الـ WAF:** Decoder هي مجرد أداة مساعدة للترميز، وتجاوز جدران الحماية (WAF Bypasses) عملية أوسع بكثير تتطلب تقنيات متقدمة مثل التشويش (Obfuscation)، تغيير حالة الأحرف (Case variation)، الترميز المزدوج (Double encoding)، وتلويث الباراميترات (HTTP Parameter Pollution).

### 2. أداة Comparer (المقارنة البصرية للاستجابات):
تقوم `Comparer` بعمل مقارنة تفاضلية (Diffing) بين قطعتين من البيانات (طلبين أو ردين):
* **المقارنة بالكلمات (Words):** لكشف التغيرات في نصوص الـ HTML أو استجابات الـ JSON.
* **المقارنة بالبايتات (Bytes):** لكشف الفروق في الملفات الثنائية (Binary payloads/images).
* **⚠️ دوره الحقيقي في Blind SQLi:** Comparer ليس الأداة الأساسية لتأكيد ثغرات الـ Blind SQLi؛ تأكيد الثغرة يتم برمجياً عبر فروق الـ Boolean (طول الاستجابة `Response Length` في Repeater/Intruder مع `Grep`) أو الفروق الزمنية (Time-based delays). دور Comparer ينحصر فقط في كونه أداة مساعدة لإظهار الحقول المتغيرة الدقيقة بين الاستجابتين.

### 3. أداة Sequencer (محلل عشوائية التوكنات والإنتروبيا):
تختبر `Sequencer` جودة التوليد العشوائي للقيم الحساسة (مثل `Session IDs`, `CSRF tokens`, `Password Reset tokens`):
* **مفهوم الإنتروبيا (Entropy):** كلما كانت قيمة التوكن عشوائية وغير قابلة للتنبؤ، ارتفع معدل الإنتروبيا المقاس بالبت (Effective Entropy Bits).
* **الاختبارات الإحصائية:** تطبق الأداة اختبارات قياسية مستندة إلى معايير **FIPS 140-2** لفحص توزيع البتات، والتكرار، والأنماط الدورية.
* **أهميته في كشف Session Fixation:** إذا كان السيرفر يصدر معرفات جلسة ذات إنتروبيا ضعيفة أو ذات نمط تزايدي، يمكن للمهاجم التنبؤ بمعرف جلسة المستخدم القادم وسرقة حسابه دون الحاجة لكلمة السر.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: استخدام Decoder لتحليل التوكنات المتراكبة
1. التقط طلباً يحتوي على Cookie مشفرة تبدو مثل: `V1ZSS2ExbFdXa2hUYms1dlpHMHhOVmxYY0hWaU1rNXpZV3hzTkdNeFdXZFhhM1JvVUZFOVBRPT0=`
2. أرسل القيمة إلى Decoder (`Ctrl + C` ثم لصق في Decoder).
3. اضغط على زر **Decode as... > Base64**، ستلاحظ فك الطبقة الأولى.
4. كرر العملية عدة مرات عبر قائمة **Decode as...** لاكتشاف محتوى التوكن الحقيقي أسفل طبقات الترميز المتعددة.

### التمرين 2: فحص عشوائية Session Cookie عبر Sequencer
1. في متصفحك، حدد صفحة تسجيل الدخول التي تقوم بتعيين كوكي الجلسة (`Set-Cookie: session=...`).
2. في سجل `Proxy > HTTP history`، كليك يمين على الطلب واختر **Send to Sequencer**.
3. في تاب **Select live capture token**:
   * سيحدد Burp تلقائياً أسماء الكوكيز المكتشفة في الترويسة، اختر `session`.
4. اضغط زر **Start live capture**:
   * سيبدأ Burp بإرسال الطلب آلاف المرات وتجميع عينات التوكنات في الخلفية.
   * اجمع 1,000 عينة كحد أدنى (الوضع المثالي 10,000 عينة).
5. اضغط **Pause** ثم **Analyze now**.
6. راجع تقرير النتائج:
   * قيمة **Effective Entropy**: إذا كانت أقل من 64 bits، فالتوكن يعتبر ضعيفاً ومقلقاً أمنياً؛ إذا تجاوزت 112 bits، فالتوليد العشوائي قوي ومطابق للمعايير الكريبتوغرافية.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Blind SQL injection with conditional responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)
  * استغل ثغرة Blind SQL Injection عبر تحليل التغير في الاستجابات بالعين وبمساعدة Comparer لملاحظة النص المشروط `Welcome back`.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يستخدم Sequencer لاستهداف الرموز الرقمية المكونة من 4 إلى 6 أرقام المستخدمة في استعادة الحسابات عبر الـ SMS، للتأكد من ما إذا كانت خوارزمية توليد الأرقام العشوائية (PRNG) قابلة للتنبؤ رياضياً.
* **🛡️ منظور المدافع:**
  * يجب على المطورين استخدام مولدات أرقام عشوائية آمنة كريبتوغرافياً (CSPRNG مثل `java.security.SecureRandom` أو `crypto.randomBytes`) وتجنب الدوال البدائية مثل `Math.random()` في إنشاء توكنات الجلسات.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **الاعتقاد بأن Decoder يتجاوز الـ WAF بمفرده:** استخدام Base64 أو URL encoding على حمولة XSS وإرسالها لخادم لا يفك هذا الترميز سيؤدي فقط إلى عدم تنفيذ الحمولة، لأن السيرفر الهدف يجب أن يكون مبرمجاً لفك الترميز أولاً.
> 2. **تحليل Sequencer بعدد عينات ضئيل:** أخذ 20 أو 30 عينة فقط والضغط على Analyze يعطي نتائج إحصائية غير دقيقة ومضللة تماماً؛ الحد الأدنى للثقة الإحصائية هو 100 عينة والموصى به 1,000 فأكثر.
> 3. **الاعتماد على Comparer كأداة حاسمة في ثغرات Blind SQLi:** تضييع وقت طويل في إرسال طلبات الـ Blind SQLi واحداً تلو الآخر ومقارنتها في Comparer، بدلاً من أتمتتها في Intruder بالاعتماد على عمود طول الاستجابة (Length).

---

## 📚 مراجع

1. [PortSwigger Documentation: Burp Decoder](https://portswigger.net/burp/documentation/desktop/tools/decoder)
2. [PortSwigger Documentation: Burp Comparer](https://portswigger.net/burp/documentation/desktop/tools/comparer)
3. [NIST FIPS 140-2 Cryptographic Standards](https://csrc.nist.gov/publications/detail/fips/140/2/final)

---

| [← السابق: 05. أداة Intruder والأتمتة](./05-intruder.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 07. أدوات Logger و Organizer →](./07-logger-organizer.md) |
