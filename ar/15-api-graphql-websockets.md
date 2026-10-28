# 15. فحص واجهات البرمجة الحديثة: API و GraphQL و WebSockets

> 🌐 **اللغة / Language:** [English Version (`en/15-api-graphql-websockets.md`)](../en/15-api-graphql-websockets.md)

---

## 🎯 الهدف من الفصل

إتقان منهجيات اختبار واجهات البرمجة والتطبيقات اللحظية الحديثة عبر Burp Suite، واحتراف اعتراض وفحص واجهات **RESTful APIs**، واستغلال استعلامات الاستبطان (**Introspection**) وهجمات التجميع (**Batching Attacks**) في **GraphQL**، واعتراض والتلاعب بإطارات الاتصال الدائم عبر قنوات **WebSockets** وكشف ثغرات **Cross-Site WebSocket Hijacking (CSWSH)**.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. اختبار واجهات RESTful APIs:
تعمل واجهات الـ REST عبر بروتوكول HTTP القياسي وتعتمد بشكل رئيسي على تبادل البيانات بصيغة JSON:
* **استكشاف التوثيق المخفي:** البحث عن ملفات توثيق Swagger/OpenAPI (مثل `/swagger.json`, `/v2/api-docs`, `/openapi.yaml`).
* **تغيير طرق الـ HTTP (Verb Tampering):** اختبار ما إذا كان تغيير الفعل من `GET` إلى `POST` أو `PUT` أو `PATCH` يتجاوز قيود التحقق من الصلاحيات.
* **التلاعب بترويسة Content-Type:** تبديل `application/json` إلى `application/x-www-form-urlencoded` أو `application/xml` لإجبار السيرفر على استخدام معالجات قديمة معرضة للحقن.

### 2. فحص واجهات GraphQL:
تختلف واجهات GraphQL عن REST بأنها ترسل كافة الطلبات (استعلامات `queries` وتعديلات `mutations`) إلى مسار موحد واحد (عادةً `/graphql`) باستخدام طلبات `POST`:
* **استعلامات الاستبطان (Introspection Queries):** ميزة مدمجة تتيح لأي مستخدم الاستعلام عن المخطط الكامل لقاعدة البيانات (`__schema`). إذا لم يقم المطور بتعطيلها في بيئة الإنتاج، يستطيع الفاحص تحميل خريطة كاملة لكافة الجداول والحقول والعمليات السرية وغير الموثقة.
* **هجمات التجميع (Batching / Rate-Limit Bypass):** تتيح GraphQL إرسال مصفوفة من عشرات الاستعلامات في طلب HTTP واحد:
  ```json
  [
    {"query": "mutation { login(user: \"admin\", pass: \"123\") }"},
    {"query": "mutation { login(user: \"admin\", pass: \"456\") }"}
  ]
  ```
  هذا الأسلوب يتجاوز تماماً أنظمة الـ Rate Limiting التقليدية التي تعد طلبات الـ HTTP، حيث يتم تجربة 100 كلمة سر في طلب واحد فقط!

### 3. اعتراض واختبار قنوات WebSockets:
يبدأ اتصال WebSocket بطلب HTTP عادي يطلب ترقية الاتصال (`Upgrade: websocket`). بمجرد قبول السيرفر، يتحول الاتصال إلى قناة ثنائية الاتجاه دائمة (Full-Duplex):
* **تاب Proxy > WebSockets history:** يسجل كل رسالة يتم تبادلها بين الطرفين مع تحديد الاتجاه (`To client` أو `To server`).
* **اعتراض وتعديل الرسائل اللحظية (WebSocket Intercept):** إيقاف الرسائل وتعديلها على الطاير قبل وصولها للشات المباشر أو خادم الألعاب.
* **ثغرات CSWSH:** عندما لا يتحقق الخادم من ترويسة `Origin` أثناء مصافحة الترقية (Handshake)، يمكن لموقع خبيث سرقة بيانات الجلسة اللحظية للمستخدم.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: استخراج مخطط GraphQL الكامل عبر Introspection
1. التقط أي طلب موجه لمسار `/graphql` وأرسله إلى **Repeater**.
2. استبدل محتوى الاستعلام باستعلام الاستبطان القياسي لفحص المخطط:
   ```json
   {
     "query": "{ __schema { types { name fields { name } } } }"
   }
   ```
3. اضغط **Send**؛ إذا عاد السيرفر بـ `200 OK` وقائمة بأنواع البيانات، فهذا يعني أن الاستبطان مفعّل.
4. (نصيحة للمحترفين): استخدم إضافة **InQL** من متجر BApp Store؛ حيث تقوم تلقائياً بتحليل المخطط وتوليد كافة الاستعلامات والـ Mutations الجاهزة للاختبار بنقرة واحدة.

### التمرين 2: اعتراض وتعديل رسائل WebSocket في المحادثات الحية
1. اذهب إلى `Proxy > WebSockets history` وتصفح صفحة محادثة دعم فني حية (Live Chat) في الموقع الهدف.
2. انتقل لتاب `Proxy > Intercept`، واضغط على مفتاح خيارات اعتراض الويب سوكت لتمكين الاعتراض.
3. اكتب رسالة في المتصفح واضغط إرسال:
   * سيوقف Burp إطار الـ WebSocket الصادر.
4. عدل محتوى الرسالة في Burp لحقن حمولة XSS:
   `{"message": "<img src=x onerror=alert(document.domain)>"}`
5. اضغط **Forward**؛ إذا ظهر التنبيه في شاشة موظف الدعم، فقد أثبتت وجود ثغرة Stored XSS عبر قنوات الـ WebSockets.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Performing CSRF exploits over WebSocket](https://portswigger.net/web-security/websockets/lab-cross-site-websocket-hijacking)
  * تدرب على استغلال غياب التحقق من ترويسة Origin لسرقة محادثات الويب سوكت الحساسة عبر هجوم Cross-Site WebSocket Hijacking.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يفضل المهاجمون استهداف واجهات GraphQL لسرعة جمع المعلومات عن الهيكل الداخلي لقواعد البيانات، واستغلال قنوات WebSockets لكونها غالباً ما تُهمل من قِبل جدران حماية الويب (WAFs).
* **🛡️ منظور المدافع:**
  * الدفاع يتطلب: تعطيل الـ Introspection في بيئات الإنتاج، وفرض قيود على عمق وتعقيد الاستعلامات (Query Depth & Cost Limiting)، والتحقق الصارم من ترويسة `Origin` واستخدام توكنات CSRF فريدة أثناء مصافحة الـ WebSockets.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **التعامل مع رسائل WebSockets كطلبات HTTP عادية:** محاولة إرسال إطار WebSocket إلى أداة Repeater المخصصة لـ HTTP، في حين يجب استخدام **Repeater المخصص للويب سوكت** (عبر كليك يمين > Send to Repeater من تاب WebSocket history).
> 2. **تجاهل تجربة تغيير HTTP Verbs في واجهات REST:** التوقف عن الفحص بمجرد رفض مسار `POST /api/user`، بدلاً من تجربة `PUT` أو `PATCH` لاكتشاف عيوب التحقق غير المكتمل.
> 3. **إهمال فحص GraphQL Alias Overloading:** عدم استغلال ميزة الأسماء المستعارة (Aliases) لإرسال مئات الاستعلامات المعقدة وتفجير موارد المعالج في الخادم (DoS).

---

## 📚 مراجع

1. [PortSwigger Web Security Academy: GraphQL API Vulnerabilities](https://portswigger.net/web-security/graphql)
2. [PortSwigger Web Security Academy: Testing WebSockets](https://portswigger.net/web-security/websockets)
3. [OWASP API Security Top 10 Project](https://owasp.org/www-project-api-security/)

---

| [← السابق: 14. اختبار صلاحيات الوصول و IDOR](./14-access-control.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 16. ثغرات HTTP/2 و Request Smuggling →](./16-http2-request-smuggling.md) |
