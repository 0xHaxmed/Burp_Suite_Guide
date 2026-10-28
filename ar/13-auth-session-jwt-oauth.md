# 13. اختبار المصادقة والجلسات وتوكنات JWT و OAuth

> 🌐 **اللغة / Language:** [English Version (`en/13-auth-session-jwt-oauth.md`)](../en/13-auth-session-jwt-oauth.md)

---

## 🎯 الهدف من الفصل

إتقان فحص وتفكيك آليات المصادقة وإدارة الجلسات الحديثة باستخدام أداة Burp Suite، واحتراف اختبار توكنات **JSON Web Tokens (JWT)** واختراقها عبر إضافة `JWT Editor`، وتجاوز آليات التحقق بخطوتين (**2FA Bypasses**)، ورصد واستغلال عيوب تدفقات تفويض **OAuth 2.0**.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. تشريح توكنات JWT وهجمات التزوير:
يتكون الـ JWT من ثلاثة أجزاء مشفرة بـ Base64URL تفصل بينها نقاط (`Header.Payload.Signature`):
* **هجوم خوارزمية None (None Algorithm Attack):** يدعم معيار JWT خوارزمية تسمى `none` للمواقف التي لا تتطلب توقيعاً. في الأنظمة المصابة، يقوم المختبر بتعديل الـ Header إلى `{"alg": "none"}` وحذف جزء التوقيع بالكامل؛ فإذا قبل السيرفر التوكن، يمكن للمهاجم تعديل أي قيمة داخل الـ Payload (مثل تغيير `"user": "guest"` إلى `"user": "admin"`) والحصول على صلاحيات الأدمن فوراً.
* **هجوم خلط الخوارزميات (Algorithm Confusion: RS256 vs HS256):** يحدث عندما يغير المهاجم خوارزمية التشفير من غير المتناظرة `RS256` (التي تستخدم مفتاحاً عاماً ومفتاحاً خاصاً) إلى الخوارزمية المتناظرة `HS256`. يقوم المهاجم بتوقيع التوكن المزور باستخدام المفتاح العام للسيرفر (Public Key) كمفتاح HMAC سري؛ وإذا كان السيرفر غير محمي، سيتحقق من التوقيع بنجاح باستخدام نفس المفتاح العام المخزن لديه!
* **حقن المفاتيح وتزييف الروابط (JWK / JKU Injection):** تضمين مفتاح التوقيع العام للمهاجم مباشرة في ترويسة التوكن أو توجيه السيرفر لجلب المفاتيح من خادم يسيطر عليه المهاجم عبر ترويسة `jku`.

### 2. تجاوزات التحقق بخطوتين (2FA / MFA Bypasses):
* **عيوب المنطق البرمجي (Broken Logic):** تخمين كود الـ 2FA المكون من 4 أو 6 أرقام عبر أداة `Intruder` لغياب حماية الـ Rate Limiting، أو التلاعب بالكوكيز في الـ Proxy لربط الجلسة بحساب الضحية وتخطي خطوة طلب الكود.
* **التصفح القسري (Forced Browsing):** محاولة فتح الروابط الداخلية (مثل `/my-account` أو `/dashboard`) مباشرة بعد إدخال اليوزر والباسورد دون إكمال خطوة إدخال رمز التحقق الثاني.

### 3. ثغرات OAuth 2.0:
* **تسريب كود التفويض (Redirect URI Hijacking):** التلاعب بباراميتر `redirect_uri` في طلب المصادقة لتوجيه السيرفر لإرسال كود التفويض (Authorization Code) أو التوكن إلى خادم يسيطر عليه المهاجم (أو خادم Burp Collaborator).
* **غياب باراميتر State (OAuth CSRF):** عندما لا يستخدم التطبيق باراميتر `state` عشوائياً، يمكن للمهاجم إجبار حساب الضحية على الارتباط بحساب المهاجم الاجتماعي.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: استغلال ثغرة JWT None Algorithm
1. التقط طلباً يحتوي على ترويسة مصادقة Bearer JWT في أداة **Repeater**:
   ```http
   GET /admin/dashboard HTTP/1.1
   Host: target-app.com
   Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiY2FybG9zIiwiaXNBZG1pbiI6ZmFsc2V9.signature
   ```
2. انسخ جزء الـ Header وافتحه في **Decoder**: فك تشفيره بـ Base64، وعدل الخوارزمية لتصبح:
   ```json
   {
     "alg": "none",
     "typ": "JWT"
   }
   ```
3. انسخ جزء الـ Payload، وفك تشفيره وعدل الصلاحيات:
   ```json
   {
     "user": "administrator",
     "isAdmin": true
   }
   ```
4. أعد تشفير الجزأين بصيغة **Base64-URL** وضع نقطة بينهما، واحذف جزء التوقيع تماماً مع الإبقاء على النقطة الأخيرة:
   `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiYWRtaW5pc3RyYXRvciIsImlzQWRtaW4iOnRydWV9.`
5. أرسل الطلب في Repeater؛ إذا ظهرت لك لوحة تحكم الإدارة، فقد تم قبول التوكن المزور بنجاح.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: JWT authentication bypass via unverified signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)
  * تطبيق عملي مخصص لتعديل توكن JWT وتجاوز فحص المصادقة والحصول على صلاحيات الإدارة بدون الحاجة لتوقيع صالح.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يستهدف المهاجمون طبقات المصادقة المعقدة (JWT و OAuth) لأن أي خطأ بسيط في التحقق من صحة التواقيع أو الروابط المرجعية يمنحهم وصولاً كاملاً إلى كافة بيانات المستخدمين دون إثارة الشبهات.
* **🛡️ منظور المدافع:**
  * يجب على المطورين رفض أي توكن يستخدم خوارزمية `none` بشكل قاطع، والتحقق الصارم من التوقيع الرقمي باستخدام مفتاح سري قوي، وفرض مطابقة تامة (Exact Whitelist Matching) لروابط `redirect_uri` في تدفقات OAuth.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **نسيان النقطة الأخيرة عند حذف توقيع JWT:** في هجوم `none algorithm`، يتطلب هيكل التوكن وجود نقطتين (`header.payload.`); حذف النقطة الثانية يؤدي لفشل السيرفر في التعرف على صيغة التوكن.
> 2. **استخدام ترميز Base64 القياسي بدلاً من Base64URL:** التوكنات لا تقبل الرموز مثل `+` و `/` وتستبدلها بـ `-` و `_` مع حذف علامات التساوي `=`. استخدام Base64 عادي يشوه التوكن.
> 3. **إهمال فحص وجود قيود الـ Rate Limiting في أكواد 2FA:** محاولة إدخال أكواد تجريبية في المتصفح يدوياً بدلاً من فحص عدد المحاولات المسموح بها في Intruder لتحديد إمكانية كسر الكود بالكامل.

---

## 📚 مراجع

1. [PortSwigger Web Security Academy: JWT Attacks](https://portswigger.net/web-security/jwt)
2. [PortSwigger Web Security Academy: OAuth 2.0 Vulnerabilities](https://portswigger.net/web-security/oauth)
3. [RFC 7519: JSON Web Token (JWT) Standard](https://datatracker.ietf.org/doc/html/rfc7519)

---

| [← السابق: 12. تطبيق Burp على ثغرات الويب](./12-vulnerabilities-with-burp.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 14. اختبار صلاحيات الوصول و IDOR →](./14-access-control.md) |
