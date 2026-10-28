# 11. إضافات Burp ومنصة Montoya API (Extensions & BApp Store)

> 🌐 **اللغة / Language:** [English Version (`en/11-extensions.md`)](../en/11-extensions.md)

---

## 🎯 الهدف من الفصل

توسيع قدرات Burp Suite وتحويلها إلى منصة اختبار اختراق فائقة القوة عبر إتقان إدارة الإضافات من متجر **BApp Store**، وفهم الفرق بين واجهة البرمجة القديمة (Legacy Extender) والمنصة الحديثة المعتمدة (**Montoya API**)، وضبط بيئات تشغيل **Jython** وقيودها، والتعرف على الترسانة الأساسية من أهم الإضافات الاحترافية التي يعتمد عليها كبار صائدي الثغرات.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

### 1. بيئة تشغيل الإضافات ومعمارية Montoya API:
يوفر Burp Suite واجهة برمجة تطبيقات (API) تمكّن المطورين من الوصول المباشر إلى نوى البروكسي، واعتراض الطلبات وتعديلها في الذاكرة، وبناء أدوات فحص مخصصة، وإضافة عناصر جديدة لواجهة المستخدم (UI Tabs):
* **Montoya API:** المعيار الحديث والرسمي لإضافات Burp Suite (أُطلق ليحل محل واجهة `IBurpExtender` القديمة). كُتبت بلغة Java الحديثة وKotlin، وتوفر أداءً فائقاً، ودعماً أصيلاً لبروتوكول HTTP/2، وأدوات متطورة للتعامل مع الـ WebSockets.
* **⚠️ قيود بايثون و Jython (توضيح فني بالغ الأهمية):**
  * إضافات لغة Python لا تعمل مباشرة عبر مفسر بايثون المثبت على نظامك (Python 3)، بل تتطلب تثبيت **Jython Standalone JAR** (وهو محرك لتشغيل بايثون داخل بيئة Java).
  * **العيب الحاسم:** Jython يدعم فقط إصدار **Python 2.7** القديم، ولا يدعم مكتبات Python 3 الحديثة إلا إذا كانت الإضافة تستخدم حلولاً خارجية أو تتواصل عبر gRPC/Sockets.
* **إضافات Ruby:** تتطلب تثبيت مفسر **JRuby Standalone JAR**.

### 2. الترسانة الاحترافية من أهم إضافات BApp Store:

| اسم الإضافة | وظيفتها الأساسية في اختبار الاختراق |
| :--- | :--- |
| **Autorize** | أداة الأتمتة الأولى لفحص صلاحيات الوصول (Access Control) وكشف ثغرات IDOR والترقية الرأسية والأفقية في الخلفية أثناء تصفح الموقع. |
| **Logger++** | بديل متقدم لـ Logger الافتراضي، يوفر فلاتر دقيقة بـ Regex، وبحثاً سريعاً في كل الطلبات والردود مع فرز متعدد الأعمدة. |
| **Turbo Intruder** | محرك إرسال طلبات فائق السرعة مبني بلغة C ومكتوب بلغة Python، يرسل عشرات آلاف الطلبات في الثانية وهو الأداة الأساسية لهجمات Race Conditions المتقدمة. |
| **Param Miner** | أداة التخمين الذكية لاكتشاف الباراميترات المخفية والترويسات غير الموثقة (Unlinked Parameters & Secret Headers) المستخدمة في ثغرات Web Cache Poisoning. |
| **JWT Editor** | فك وتعديل وتوقيع توكنات JSON Web Tokens وتزوير التواقيع الرقمية واختبار هجمات None Algorithm و Key Confusion. |
| **Hackvertor** | أداة معالجة متقدمة تتيح استخدام وسوم تحويل مخصصة داخل الطلب (Tags) مثل `<@base64>payload<@/base64>` لتشفير الحمولات وتجاوز الـ WAFs على الطاير. |
| **Burp Bounty** | إضافة لتوسيع ماسح Burp Scanner عبر بناء قواعد ومسارات فحص مخصصة وقوالب حقن ذكية. |

---

## 🛠️ التطبيق العملي (Step-by-Step)

### الخطوة 1: تهيئة بيئة تشغيل Jython (Python) في Burp
1. قم بتحميل أحدث إصدار من ملف **Jython Standalone JAR** من موقعه الرسمي (مثلاً `jython-standalone-2.7.3.jar`).
2. افتح Burp Suite واذهب إلى: `Settings > Extensions > Python environment`.
3. في خانة **Location of Jython standalone JAR file**، اضغط **Select file** وحدد مسار الملف الذي حملته.
4. الآن أصبحت بيئة Burp جاهزة تماماً لتثبيت وتشغيل أي إضافة مكتوبة بلغة Python بنقرة واحدة من متجر BApp.

### الخطوة 2: تثبيت واستخدام إضافة Param Miner لاكتشاف الترويسات المخفية
1. اذهب إلى تاب `Extensions > BApp Store`.
2. ابحث عن إضافة **Param Miner** واضغط **Install**.
3. اذهب إلى `Proxy > HTTP history` أو `Repeater` وحدد طلباً لصفحة رئيسية في الموقع الهدف.
4. انقر بزر الماوس الأيمن على الطلب واختر: **Extensions > Param Miner > Guess params > Guess headers**.
5. اترك الإعدادات الافتراضية واضغط **OK**؛ ستبدأ الإضافة بالعمل في الخلفية.
6. تابع تقدم العمل عبر تاب `Extensions > Installed > Param Miner > Output`؛ إذا اكتشفت الإضافة ترويسة مخفية (مثل `X-Original-URL` أو `X-Host`)، ستصدر تنبيهاً فورياً في تاب Dashboard.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Web cache poisoning with an unkeyed header](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-an-unkeyed-header)
  * استخدم إضافة Param Miner المكتشفة في هذا الفصل لرصد ترويسة مخفية غير مدرجة في مفتاح الكاش (`X-Forwarded-Host`) واستغلالها لتسميم كاش الويب.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * الإضافات المخصصة هي ما يصنع الفارق في برامج مكافآت الثغرات؛ حيث يبني صائدو الثغرات سكريبتات Montoya API خاصة بهم تحقن ترويسات فريدة وتتفاعل مع ثغرات خاصة لم تدرج بعد في الماسحات التجارية.
* **🛡️ منظور المدافع:**
  * يمكن لفرق الدفاع كتابة إضافات Montoya API مخصصة لمراقبة الترافيك الداخلي وفحص التزام التطبيقات بالمعايير الأمنية المؤسسية قبل تسليم المشاريع للإنتاج.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **محاولة ربط مفسر بايثون 3 القياسي:** تحديد مسار `python.exe` في خانة Jython، مما يؤدي لفشل التشغيل وظهور رسالة خطأ؛ يجب استخدام ملف `jython-standalone-xxx.jar` حصراً.
> 2. **تثبيت عشرات الإضافات الثقيلة في وقت واحد:** تثبيت عدد مفرط من الإضافات دون حاجة يستهلك موارد الذاكرة ويؤدي لبطء شديد في استجابة Burp Suite. قم بتعطيل الإضافات التي لا تستخدمها في مهمتك الحالية.
> 3. **تجاهل ترتيب تنفيذ الإضافات:** في تاب `Extensions > Installed`، تتنفذ الإضافات بترتيب تنازلي من الأعلى للأسفل؛ إذا كانت هناك إضافتان تعدلان على نفس الطلب، فالإضافة العلوية تتنفذ أولاً.

---

## 📚 مراجع

1. [PortSwigger Documentation: Montoya API Reference](https://portswigger.net/burp/extender/api/)
2. [Jython Official Project](https://www.jython.org/)
3. [PortSwigger GitHub: Burp Extensions Official Repository](https://github.com/PortSwigger)

---

| [← السابق: 10. المتصفح المدمج و DOM Invader](./10-dom-invader-browser.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 12. تطبيق Burp على أشهر ثغرات الويب →](./12-vulnerabilities-with-burp.md) |
