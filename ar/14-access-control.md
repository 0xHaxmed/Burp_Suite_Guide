# 14. اختبار صلاحيات الوصول وثغرات IDOR (Access Control & Autorize)

> 🌐 **اللغة / Language:** [English Version (`en/14-access-control.md`)](../en/14-access-control.md)

---

## 🎯 الهدف من الفصل

إتقان منهجية فحص صلاحيات الوصول والتحكم في التفويض (**Access Control & Authorization**)، واكتشاف ثغرات الإسناد المباشر للكائنات غير الآمنة (**IDOR**)، وثغرات الترقية الرأسية والأفقية، وأتمتة مصفوفة الصلاحيات بالكامل باستخدام إضافة **Autorize** الرائدة لتوفير عشرات الساعات من الاختبار اليدوي.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. تصنيفات عيوب صلاحيات الوصول:
* **الترقية الرأسية للصلحيات (Vertical Privilege Escalation):** قدرة مستخدم عادي (User) على تنفيذ وظائف إدارية مخصصة لمدير النظام (Administrator)، مثل فتح مسار `/admin/delete-user` أو تعديل إعدادات الموقع.
* **الترقية الأفقية للصلاحيات والـ IDOR (Horizontal Privilege Escalation):** قدرة مستخدم (User A) على الوصول إلى موارد وبيانات مستخدم آخر مساوٍ له في الرتبة (User B) بمجرد تغيير رقم المعرف (`id=105` إلى `id=106`) في رابط الـ URL أو جسم طلب الـ JSON.
* **تجاوز الصلاحيات عبر ترويسات إعادة التوجيه:** استغلال أطر عمل برمجية (مثل Spring) تثق بترويسات مثل `X-Original-URL: /admin` أو `X-Rewrite-URL` لتخطي قواعد الجدار الناري التي تحظر مسار الإدارة ظاهرياً فقط.

### 2. إضافة Autorize وآلية الأتمتة المزدوجة (The Dual-Replay Engine):
في تطبيقات الويب الضخمة التي تحتوي على مئات الـ Endpoints، يصبح تبديل الكوكيز يدوياً في Repeater لكل طلب أمراً مستحيلاً وعرضة للخطأ البشري. هنا يأتي دور إضافة **Autorize**:
* تقوم بتسجيل الدخول بحساب الضحية العادي ونسخ ترويسة الكوكي الخاصة به ووضعها داخل لوحة تحكم Autorize.
* يتصفح المختبر الموقع بحساب الأدمن (أو العكس) بالمتصفح العادي.
* **ما تفعله الإضافة في أجزاء من الثانية:** تأخذ كل طلب يرسله الأدمن وتكرره مرتين فوراً في الخلفية:
  1. المرة الأولى: باستخدام كوكي المستخدم العادي.
  2. المرة الثانية: بدون أي كوكي إطلاقاً (فحص المستخدم غير المسجل - Unauthenticated).
* **نظام الإشارات الملونة في Autorize:**
  * 🔴 **Bypassed (أحمر):** ثغرة حتمية! استطاع المستخدم العادي قراءة البيانات أو تنفيذ العملية بنجاح.
  * 🟢 **Enforced (أخضر):** محمي! رفض السيرفر الطلب برمز `403 Forbidden` أو `401 Unauthorized`.
  * 🟠 **Is Enforced??? (برتقالي):** استجابة تحتاج لمراجعة يدوية لتطابق أطوال الردود أو تشابه أكواد الحالة.

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: إعداد واستخدام إضافة Autorize لأتمتة فحص الصلاحيات
1. من تاب `Extensions > BApp Store`، ابحث عن إضافة **Autorize** وثبتها.
2. افتح متصفحين (أو بروفايلين مختلفين):
   * المتصفح 1: سجل الدخول بحساب المستخدم العادي (`carlos`).
   * المتصفح 2: سجل الدخول بحساب المدير (`admin`).
3. انسخ ترويسة الكوكي الكاملة الخاصة بالمستخدم العادي (`Cookie: session=xyz...`).
4. انتقل إلى تاب **Autorize** في Burp:
   * الصق الكوكي في خانة **Configuration / Low Privileged User Cookies**.
   * اضغط على زر **Autorize is off** ليتحول إلى **Autorize is on**.
5. تصفح لوحة التحكم ومسارات الحذف والإضافة في المتصفح بحساب الـ `admin`.
6. راقب جدول النتائج في Autorize: أي صف يظهر بلون أحمر عريض (`Bypassed!`) هو ثغرة ترقية صلاحيات مؤكدة يمكنك النقر عليها لمعاينة الطلب واستخراجه.

### التمرين 2: تجاوز قيود الـ Admin عبر ترويسة `X-Original-URL`
1. أرسل طلب الوصول لصفحة الإدارة الممنوعة في Repeater:
   ```http
   GET /admin HTTP/1.1
   Host: target-app.com
   ```
   * يعود السيرفر بـ `403 Access Denied`.
2. غيّر المسار إلى مسار عام مسموح به، وأضف ترويسة `X-Original-URL` تشير لمسار الإدارة:
   ```http
   GET / HTTP/1.1
   Host: target-app.com
   X-Original-URL: /admin/delete?username=carlos
   ```
3. اضغط **Send**؛ إذا حذف السيرفر المستخدم وعاد بـ `200 OK`، فقد تم تجاوز طبقة التحقق من الصلاحيات بالكامل.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: URL-based access control can be circumvented](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)
  * تدرب على استغلال ثغرة في التحقق من الصلاحيات القائم على الروابط وتجاوزها باستخدام ترويسة `X-Original-URL`.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * ثغرات الصلاحيات هي "المنجم الذهبي" في مسابقات Bug Bounty؛ لأنها شائعة جداً في واجهات الـ REST API وتعطي أقصى تقييم خطورة (High/Critical) لسهولة استغلالها وقوة تأثيرها.
* **🛡️ منظور المدافع:**
  * يجب بناء التحقق من الصلاحيات كطبقة برمجية وسيطة مركزية وموحدة (Centralized Authorization Middleware) على مستوى الكود المصدري في كل دالة وكل استعلام قاعدة بيانات، وليس مجرد الاعتماد على فلاتر الـ URL في الـ Reverse Proxy.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **الاعتماد على إخفاء الأزرار في الواجهة كحماية أمنية:** يظن بعض المطورين أن عدم إظهار زر "Delete" للمستخدم العادي يحميه، متناسين أن الفاحص يستطيع إرسال طلب `POST /user/delete` مباشرة عبر Repeater.
> 2. **عدم ضبط فلاتر الاستبعاد في Autorize:** ترك Autorize تفحص ملفات الصور والـ CSS والنصوص الثابتة يملأ الجدول بإنذارات خاطئة مضللة؛ اضبط الفلتر لاستبعاد امتدادات `jpg|png|css|js`.
> 3. **تجاهل فحص الترقية الأفقية (IDOR) في طلبات الـ PUT و DELETE:** التركيز فقط على طلبات القراءة (`GET`) وإهمال اختبار ما إذا كان المستخدم يستطيع تعديل بيانات الآخرين أو حذف ملفاتهم عبر طلبات التعديل.

---

## 📚 مراجع

1. [PortSwigger Web Security Academy: Access Control Vulnerabilities](https://portswigger.net/web-security/access-control)
2. [OWASP Testing Guide: Testing for Insecure Direct Object References (IDOR)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References)
3. [Autorize Official GitHub Repository](https://github.com/Quitten/Autorize)

---

| [← السابق: 13. اختبار المصادقة و JWT و OAuth](./13-auth-session-jwt-oauth.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 15. فحص واجهات البرمجة API و GraphQL و WebSockets →](./15-api-graphql-websockets.md) |
