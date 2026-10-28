# 10. المتصفح المدمج وأداة DOM Invader (Embedded Browser & DOM Invader)

> 🌐 **اللغة / Language:** [English Version (`en/10-dom-invader-browser.md`)](../en/10-dom-invader-browser.md)

---

## 🎯 الهدف من الفصل

التعرف على قوة **المتصفح المدمج (Burp's Embedded Browser)** وكيف يوفر عليك عناء ضبط الشهادات والبروكسيات، واحتراف استخدام أداة **DOM Invader** المدمجة داخله لاكتشاف وتتبع أخطر ثغرات الجافاسكريبت من جهة العميل تلقائياً، مثل **DOM-based XSS**، والتلاعب برسائل الويب (**Web Messages - postMessage**)، وثغرات تلويث النماذج الأولية (**Client-Side Prototype Pollution**).

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. المتصفح المدمج (Burp Embedded Chromium Browser):
يأتي Burp Suite مزوداً بنسخة مخصصة ومعزولة من متصفح Chromium:
* **Zero-Configuration:** يعمل فورياً بدون الحاجة لتثبيت أي إضافات (مثل FoxyProxy) وبدون الحاجة لتثبيت شهادة `PortSwigger CA`، حيث تم تجهيز المتصفح مسبقاً للوثوق بشهادات Burp المحلية.
* **Sandbox Isolation:** يعمل في بيئة معزولة تماماً تمنع تداخل الكوكيز أو تاريخ التصفح مع متصفحك اليومي الشخصي.

### 2. أداة DOM Invader (صائد ثغرات الـ DOM):
في ثغرات **DOM-based XSS**، تنتقل البيانات الخبيثة من مصدر دخل (**Source** مثل `location.search` أو `document.cookie`) إلى نقطة تنفيذ تنفيذية خطرة داخل كود الجافاسكريبت (**Sink** مثل `innerHTML` أو `eval`) بالكامل داخل المتصفح دون أن تمر بالسيرفر!
* **المشكلة:** أدوات الفحص التقليدية لا يمكنها رؤية هذا التدفق لأنه يحدث داخل الذاكرة التنفيذية للمتصفح.
* **الحل الثوري (DOM Invader):** إضافة متطورة مدمجة في أدوات المطورين في المتصفح المدمج تقوم بحقن كود استكشافي فريد (**Canary String**)، وتتتبع مساره في كل سطر جافاسكريبت:
  * تراقب ما إذا كان الـ Canary قد وصل إلى أي Sink خطير.
  * تحدد ما إذا كانت الحماية المطبقة قد قامت بترميز الحروف الخاصة أم تركتها قابلة للاستغلال.

### 3. فحص Web Messages و Prototype Pollution:
* **Web Messages:** تراقب أحداث `window.addEventListener('message', ...)` وتعترض رسائل `postMessage` للكشف عما إذا كان التطبيق يتحقق من مصدر الرسالة (`event.origin`) أم يقبل أوامر خبيثة من نوافذ الـ iframe الخارجية.
* **Client-side Prototype Pollution:** تختبر ما إذا كان من الممكن حقن خصائص مثل `__proto__` أو `constructor.prototype` للتلاعب بسلوك كائنات الجافاسكريبت الأساسية في الصفحة وتصعيدها إلى XSS.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### الخطوة 1: تشغيل المتصفح المدمج وتفعيل DOM Invader
1. في تاب `Proxy > Intercept`، اضغط على زر **Open browser**.
2. داخل نافذة المتصفح المدمج، اضغط على زر قائمة المتصفح > ثم افتح **Developer Tools** (أو اضغط `F12`).
3. ستجد تاباً جديداً باسم **DOM Invader**؛ اضغط عليه.
4. فعّل المفتاح الرئيسي: **DOM Invader is on**.
5. تحت إعدادات الأداة:
   * فعّل خيار **Postmessage interception** لرصد الرسائل التفاعلية.
   * فعّل خيار **Prototype pollution** للكشف عن ثغرات تلويث الكائنات.
   * انسخ نص الـ Canary الافتراضي (سلسلة نصية عشوائية قصيرة).

### الخطوة 2: اختبار DOM XSS خطوة بخطوة
1. تصفح صفحة بحث في موقع يعتمد على معالجة الـ DOM (مثلاً صفحة بحث تبحث بدون إعادة تحميل).
2. في خانة البحث، الصق نص الـ Canary متبوعاً بحروف خاصة مثل: `CANARY"'>`
3. ارجع إلى تاب **DOM Invader** في أدوات المطورين:
   * سيعرض لك جدولاً فورياً يوضح: الـ **Source** المستغل (مثلاً `location.search`)، والـ **Sink** الذي استقبل القيمة (مثلاً `element.innerHTML`).
4. اضغط على زر **Exploit** المجاور للنتيجة:
   * سيقوم DOM Invader باختبار وتوليد حمولة XSS جاهزة ومخصصة لتجاوز الفلتر تلقائياً وإطلاق صندوق التنبيه `alert(1)`.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: DOM XSS using web messages](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-dom-xss-using-web-messages)
  * تدرب على استخدام DOM Invader لرصد استماع غير آمن لرسائل `postMessage` وتوليد كود الـ iframe المستغل للثغرة.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يتيح DOM Invader للمهاجمين تفكيك آلاف أسطر أكواد الجافاسكريبت المعقدة والمضغوطة (Minified / Obfuscated JS) في ثوانٍ معدودة، مما يجعل العثور على ثغرات الـ DOM أسرع بمئات المرات من الفحص اليدوي.
* **🛡️ منظور المدافع:**
  * لمنع ثغرات الـ DOM، يجب على المطورين تجنب إدخال مدخلات المستخدم في Sinks خطرة مثل `innerHTML` واستبدالها بـ `textContent`، والتحقق الصارم دائماً من `event.origin` في كل مستمع لرسائل `postMessage`.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **البحث عن DOM Invader في متصفح خارجي (Firefox):** DOM Invader هي ميزة حصرية مدمجة في المتصفح المدمج داخل Burp (Chromium) ولا تتوفر تلقائياً في المتصفحات الخارجية العادية.
> 2. **نسيان فتح أدوات المطورين (DevTools):** عدم فتح نافذة الـ DevTools يمنعك من رؤية مخرجات وتنبيهات DOM Invader أثناء التصفح.
> 3. **تجاهل فحص Prototype Pollution:** الاعتقاد بأن تلويث النماذج الأولية في العميل ثغرة نظرية فقط؛ في الواقع يمكن في كثير من الأحيان دمجها مع كود موجود في الصفحة (Gadget) وتصعيدها إلى ثغرة XSS كاملة وسرقة الجلسات.

---

## 📚 مراجع

1. [PortSwigger Documentation: Burp Suite Embedded Browser](https://portswigger.net/burp/documentation/desktop/tools/proxy/browser)
2. [PortSwigger Documentation: DOM Invader Full Guide](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
3. [OWASP Client-Side Security Testing: Testing for DOM-Based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)

---

| [← السابق: 09. أداة Burp Scanner](./09-scanner.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 11. إضافات Burp ومنصة Montoya API →](./11-extensions.md) |
