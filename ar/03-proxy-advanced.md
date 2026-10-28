# 03. البروكسي المتقدم وقواعد المعالجة (Advanced Proxy & Rules)

> 🌐 **اللغة / Language:** [English Version (`en/03-proxy-advanced.md`)](../en/03-proxy-advanced.md)

---

## 🎯 الهدف من الفصل

الارتقاء بمهارات استخدام تاب البروكسي إلى مستوى متقدم عبر إتقان قواعد الاعتراض الذكي (`Intercept Rules`)، واستخدام قواعد الاستبدال التلقائي (`Match & Replace`) للتلاعب بالترويسات والردود في الخلفية دون تدخل يدوي، وتوجيه حركة مرور Burp عبر بروكسيات وسيطة (`Upstream Proxies`) وشبكات **Tor** و **SOCKS Proxies** للعمل بأمان وتجاوز العزل الجغرافي.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. آليات الاعتراض المشروط (Conditional Interception):
الضغط المستمر على زر `Forward` لكل صورة أو سكريبت يمثل إهداراً كبيراً للوقت. يتيح Burp وضع قواعد فلترة تعترض فقط الطلبات الحساسة:
* اعتراض الطلبات التي تحتوي على باراميترات محددة (`POST requests`, `JSON bodies`).
* اعتراض طلبات الـ Scope المحدد فقط.
* **اعتراض الردود (Intercept Responses):** إمكانية إيقاف رد السيرفر قبل وصوله للمتصفح، وهي ميزة حيوية لتجاوز التحقق من جهة العميل (Bypassing Client-side validation) وتعديل نتائج المصادقة المرجعة (`{"isAdmin": false}` إلى `{"isAdmin": true}`).

### 2. محرك الاستبدال التلقائي (Match & Replace Engine):
تسمح ميزة **Match & Replace** بتطبيق تعبيرات نمطية (Regex) أو استبدال نصي مباشر على كل طلب أو رد يمر عبر البروكسي آلياً:
* إضافة ترويسات أمنية مخصصة في كل طلب (مثل `X-Forwarded-For: 127.0.0.1` لاختبار عيوب الـ IP Spoofing).
* استبدال ملفات الـ JavaScript أثناء تحميلها لتعطيل حمايات الـ Debugger أو إزالة فحص كلمات السر.
* إزالة ترويسات الأمان من الردود (مثل إزالة `Content-Security-Policy` أو `X-Frame-Options` لاختبار ثغرات الـ Clickjacking والـ XSS).

### 3. البروكسيات العلوية والاتصال عبر شبكة Tor (Upstream & SOCKS Proxies):
في بيئات العمل الواقعية، قد تحتاج لعزل هوية الفاحص أو العمل من خلال شبكة داخلية لعميل:
* **Upstream HTTP Proxy:** توجيه اتصالات Burp لخادم وسيط للوصول إلى شبكة الشركة الداخلية.
* **SOCKS Proxy (مع Tor):** توجيه كافة الترافيك عبر بروتوكول `SOCKS5` لمنفذ Tor المحلي `127.0.0.1:9050` أو `9150`، مع تفعيل خاصية توجيه طلبات الـ DNS عبر البروكسي (`Do DNS lookups over SOCKS proxy`) لمنع تسرب الـ DNS (DNS Leaks).

---

## 🛠️ التطبيق العملي (Step-by-Step)

### الخطوة 1: ضبط قواعد الـ Intercept الذكية
1. اذهب إلى `Settings > Tools > Proxy`.
2. تحت قسم **Request interception rules**:
   * فعّل خيار: `And URL Is in-scope` لضمان عدم إيقاف أي طلب خارج الهدف.
   * تأكد من وجود قاعدة: `And File extension Does not match: png|jpg|gif|css|js|woff2` لمنع إيقاف الملفات الثابتة.
3. تحت قسم **Response interception rules**:
   * ضع علامة صح على: `Intercept responses based on the following rules`.
   * أضف شرطاً لاعتراض ردود الـ Scope: `And URL Is in-scope`.

### الخطوة 2: تجاوز قيود الـ Front-end عبر Match & Replace
1. اذهب إلى `Settings > Tools > Proxy > Match and replace rules`.
2. اضغط **Add** وأنشئ القواعد التالية:
   * **تجاوز قيود الطول والحقول المعطلة:**
     * *Type:* `Response body`
     * *Match:* `disabled=""`
     * *Replace:* (اتركها فارغة لتفعيل الأزرار المعطلة تلقائياً).
   * **انتحال عنوان الآي بي الداخلي (IP Spoofing):**
     * *Type:* `Request header`
     * *Match:* (اتركها فارغة لإضافة ترويسة جديدة)
     * *Replace:* `X-Forwarded-For: 127.0.0.1`

### الخطوة 3: توجيه ترافيك Burp عبر شبكة Tor (SOCKS5)
1. شغل متصفح Tor Browser على جهازك (يعمل افتراضياً على البورت `9150`) أو خدمة tor daemon (بورت `9050`).
2. في Burp Suite، اذهب إلى `Settings > Network > Connections`.
3. انزل إلى قسم **SOCKS proxy**:
   * **SOCKS proxy host:** `127.0.0.1`
   * **SOCKS proxy port:** `9050` (أو `9150`)
   * **تنبيه حاسم:** ضع علامة صح على **Do DNS lookups over SOCKS proxy** لمنع مزود الخدمة من رؤية أسماء النطاقات التي تفحصها.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: 2FA broken logic](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)
  * استخدم اعتراض الردود (Intercept Response) وفحص الترويسات لتجاوز آلية التحقق بخطوتين.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يوفر Match & Replace للمهاجم ميزة "التخفي والسرعة"؛ فبدلاً من تعديل الترويسات أو تفعيل الصلاحيات يدوياً في كل طلب، تتم العملية بصمت في الخلفية مع كل نقرة في المتصفح.
* **🛡️ منظور المدافع:**
  * يدرك مهندسو الأمان أن خوادم الويب التي تثق بترويسات مثل `X-Forwarded-For` أو `X-Original-URL` بدون تدقيق تكون عرضة للتضليل وتجاوز قواعد الـ Reverse Proxies والـ WAFs.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **ترك Intercept مفعلاً دون الانتباه:** التساؤل عن سبب تجمد المتصفح وعدم تحميل الصفحة، ليتضح أن هناك طلباً عالقاً في تاب Proxy ينتظر ضغط `Forward`.
> 2. **تسريب الـ DNS أثناء استخدام SOCKS Proxy:** تفعيل بروكسي Tor دون تحديد خيار حل الـ DNS عبر البروكسي، مما يؤدي لإرسال استعلامات الـ DNS مباشرة عبر اتصالك الحقيقي وكشف هويتك.
> 3. **تعارض قواعد Match & Replace:** كتابة تعبير نمطي عام جداً يستبدل كلمات شائعة داخل أكواد JavaScript مما يؤدي لانهيار الموقع وتوقف وظائفه التفاعلية.

---

## 📚 مراجع

1. [PortSwigger Documentation: Proxy Intercept Rules](https://portswigger.net/burp/documentation/desktop/tools/proxy/intercept)
2. [PortSwigger Documentation: Match and Replace Rules](https://portswigger.net/burp/documentation/desktop/tools/proxy/http-matching-replacing)
3. [The Tor Project: SOCKS Proxy Configuration](https://tb-manual.torproject.org/)

---

| [← السابق: 02. لوحة التحكم و Scope](./02-dashboard-target-scope.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 04. أداة Repeater وهجمات التوازي →](./04-repeater.md) |
