# 16. ثغرات تهريب الطلبات ومشاكل HTTP/2 (Request Smuggling)

> 🌐 **اللغة / Language:** [English Version (`en/16-http2-request-smuggling.md`)](../en/16-http2-request-smuggling.md)

---

## 🎯 الهدف من الفصل

فهم الميكانيكا الفيزيائية المعقدة لثغرات تهريب طلبات الـ HTTP (**HTTP Request Smuggling**)، وكيف تؤدي الاختلافات في تفسير بروتوكول HTTP/1.1 بين الخوادم الأمامية والخلفية إلى كسر حدود الأمان، واحتراف اكتشاف واستغلال ثغرات **HTTP/2 Downgrading** المتقدمة عبر أداة Repeater وإضافة **HTTP Request Smuggler** لاختطاف جلسات المستخدمين وتسميم طوابير الاستجابات (**Response Queue Poisoning**).

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. معضلة عدم التزامن (The Desynchronization Dilemma):
تعتمد معماريات الويب الحديثة على وجود خادم أمامي (Front-end مثل Reverse Proxy أو CDN أو Load Balancer) يستقبل طلبات المستخدمين ويمررها عبر اتصال TCP واحد مستمر ومُعاد استخدامه (Keep-Alive TCP Stream) إلى خادم خلفي (Back-end Application Server).
* لكي يعرف الخادم الخلفي أين ينتهي الطلب الأول وأين يبدأ الطلب الثاني، يوفر بروتوكول HTTP/1.1 طريقتين لتحديد نهاية جسم الطلب:
  1. ترويسة `Content-Length (CL)`: تحدد الحجم الصريح بالبايت.
  2. ترويسة `Transfer-Encoding: chunked (TE)`: تقسم الطلب إلى كتل مجزأة تنتهي بكتلة حجمها صفر (`0\r\n\r\n`).
* **حدوث الثغرة:** عندما يفسر الخادم الأمامي حدود الطلب باستخدام إحدى الطريقتين بينما يفسر الخادم الخلفي الحدود بالطريقة الأخرى، يحدث انفصال في التزامن (**Desync**)، مما يسمح للطلب الأول بتهريب جزء خبيث يلتصق ببداية طلب المستخدم البريء التالي في طابور المعالجة!

### 2. أنواع الثغرة الكلاسيكية في HTTP/1.1:
* **CL.TE:** الخادم الأمامي يعتمد `Content-Length` بينما الخادم الخلفي يعتمد `Transfer-Encoding`.
* **TE.CL:** الخادم الأمامي يعتمد `Transfer-Encoding` بينما الخادم الخلفي يعتمد `Content-Length`.
* **TE.TE:** كلا الخادمين يدعمان `Transfer-Encoding`، ولكن يمكن تشويش الترويسة (Obfuscation مثل `Transfer-Encoding: xchunked`) لخداع أحدهما للتراجع لاستخدام `Content-Length`.

### 3. ثغرات العصر الحديث: التحويل التراجعي لـ HTTP/2 (H2 Downgrading):
في بروتوكول `HTTP/2`، لا توجد ترويسات `Content-Length` أو `Transfer-Encoding` نصية، بل تُرسل البيانات كإطارات ثنائية صريحة الطول (Binary Frames).
* **المشكلة الخطيرة:** كثير من خوادم الـ CDN تستقبل الترافيك من المستخدم ببروتوكول `HTTP/2` الحديث، ولكنها تعيد توجيهه للخادم الداخلي القديم بعد تحويله تراجعياً إلى `HTTP/1.1` (**HTTP/2 Downgrading**) لتقليل تكلفة التطوير!
* **ثغرات H2.CL و H2.TE:** عندما يحقن المختبر ترويسة `content-length` مضللة داخل إطار الـ HTTP/2، يتجاهلها الخادم الأمامي أثناء استقبال الـ H2 ولكنه يكتبها نصياً عند التحويل لـ HTTP/1.1، مما يسبب عدم تزامن فوري في السيرفر الخلفي.
* **حقن أسطر جديدة (CRLF Injection in H2 Pseudo-headers):** يتيح بروتوكول HTTP/2 إرسال أحرف `\r\n` داخل قيم الترويسات الزائفة (مثل ترويسة `:path` أو الترويسات المخصصة)؛ وعندما يقوم الخادم الأمامي بتحويلها إلى نص HTTP/1.1، تنقسم الترويسة الواحدة إلى عدة ترويسات جديدة تماماً!

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: استخدام إضافة HTTP Request Smuggler للفحص الآلي
1. من تاب `Extensions > BApp Store`، ابحث عن إضافة **HTTP Request Smuggler** وثبتها.
2. حدد طلباً للهدف في `Proxy > HTTP history` أو `Repeater`.
3. انقر بزر الماوس الأيمن على الطلب واختر: **Extensions > HTTP Request Smuggler > Smuggle probe**.
4. ستقوم الإضافة في الخلفية بإرسال سلسلة من الحمولات الدقيقة المجهزة لقياس فروق التوقيت واستجابات الخادم لكشف ثغرات CL.TE و TE.CL دون إسقاط السيرفر.
5. راقب تاب Dashboard أو سجل الإضافة؛ إذا تأكد وجود الثغرة، ستصدر الإضافة تقريراً مفصلاً بنوع عدم التزامن المضبوط.

### التمرين 2: استغلال ثغرة CL.TE في Repeater لاختطاف طلب المستخدم التالي
1. أرسل طلباً مصاباً لـ Repeater وتأكد من تفعيل بروتوكول HTTP/1.1.
2. اذهب إلى قائمة خيارات Repeater وعطل خيار التحديث التلقائي لطول المحتوى:
   * ألغِ تحديد: **Update Content-Length automatically**.
3. جهز الطلب التالي بحيث يكون طول الـ `Content-Length` يشمل الحجم الظاهري فقط، مع ترك طلب مهرب في الأسفل:
   ```http
   POST / HTTP/1.1
   Host: target-app.com
   Content-Length: 6
   Transfer-Encoding: chunked

   0

   G
   ```
4. عندما يعالج الخادم الخلفي الجزء `0\r\n\r\n`، يعتبر الطلب قد انتهى، ويتبقى الحرف `G` عالقاً في مقبس الـ TCP.
5. عندما يرسل المستخدم الضحية طلبه الطبيعي `GET /home HTTP/1.1`، يلتصق الحرف `G` بطلبه فيتحول إلى: `GGET /home HTTP/1.1` ويعود السيرفر للضحية بخطأ `400 Bad Request`، مما يثبت نجاح التسميم.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: HTTP request smuggling, basic CL.TE vulnerability](https://portswigger.net/web-security/request-smuggling/lab-basic-cl-te)
  * تطبيق عملي خطوة بخطوة لاستغلال عدم التزامن بين السيرفر الأمامي والخلفي لتسميم مسار الطلبات.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * ثغرات Request Smuggling هي "الكأس المقدسة" للثغرات الحرجة؛ لأنها تسمح للمهاجم بتجاوز كافة الجدران النارية الأمامية (WAF Bypasses)، وسرقة كوكيز المستخدمين الآخرين، وإعادة توجيه الترافيك بالكامل.
* **🛡️ منظور المدافع:**
  * الحل الجذري هو استخدام بروتوكول **HTTP/2 من البداية للنهاية (End-to-End HTTP/2)** دون أي تحويل تراجعي إلى HTTP/1.1، أو تكوين الخوادم الأمامية والخلفية لاستخدام معيار إغلاق الاتصال التام بعد كل طلب أو رفض أي طلب يحتوي على كلتا الترويستين معاً.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **ترك Burp يحدث Content-Length تلقائياً أثناء هجمات Smuggling:** إذا ترك خيار `Update Content-Length` مفعلاً في Repeater، سيقوم Burp بإعادة حساب الطول وتصحيحه تلقائياً مما يفسد عدم التزامن المطلوب لهجوم الـ Smuggling.
> 2. **نسيان أسطر الفراغ الدقيقة (`\r\n\r\n`):** يعتمد بروتوكول HTTP/1.1 الحرفي على وجود سطرين فارغين بعد كتلة الصفر في Chunked Encoding؛ إهمالها يسبب تعليق السيرفر (Timeout) دون رد.
> 3. **إطلاق هجمات Smuggling عشوائية على مواقع إنتاج:** التلاعب بطوابير الطلبات (Queue Poisoning) على خوادم حية قد يؤدي لتخريب جلسات آلاف الزوار الحقيقيين وحرمانهم من الخدمة؛ اختبر هذه الثغرات بحذر شديد وضمن أوقات الصيانة المحددة في تفويض الاختبار.

---

## 📚 مراجع

1. [James Kettle Research: HTTP Desync Attacks - Request Smuggling Reborn](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn)
2. [James Kettle Research: HTTP/2 - The Sequel is Always Worse](https://portswigger.net/research/http2)
3. [PortSwigger Web Security Academy: HTTP Request Smuggling](https://portswigger.net/web-security/request-smuggling)

---

| [← السابق: 15. فحص واجهات البرمجة و WebSockets](./15-api-graphql-websockets.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 17. منهجية الفحص الشاملة (Pentest Workflow) →](./17-pentest-workflow.md) |
