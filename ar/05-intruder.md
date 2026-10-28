# 05. أداة Intruder والأتمتة الذكية (Intruder & Automation)

> 🌐 **اللغة / Language:** [English Version (`en/05-intruder.md`)](../en/05-intruder.md)

---

## 🎯 الهدف من الفصل

إتقان استخدام أداة **Intruder** لأتمتة إرسال طلبات الـ HTTP المخصصة، وفهم الفروق التقنية الدقيقة بين أنواع الهجمات الأربعة، ومعالجة الـ Payloads، وتوظيف تقنيات الفرز والتحليل الآلي المتقدم (`Grep Match` و `Grep Extract`)، وتخطي حمايات الـ CSRF والـ Tokens المتغيرة باستخدام **Macros** و **Session Handling Rules**.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

تأخذ أداة `Intruder` قالباً لطلب HTTP أساسي، وتحدد فيه نقاط حقن محاطة بعلامات الفقرة (`§payload_here§`)، لتقوم باستبدال تلك المواضع بقيم من قوائم مدخلات (Wordlists) وإرسالها بمعدل سرعة قابل للضبط.

### 1. أنواع الهجوم الأربعة (Attack Types):

| نوع الهجوم | عدد قوائم الـ Payloads | طريقة الحقن | أفضل استخدام تقني |
| :--- | :---: | :--- | :--- |
| **Sniper** | قائمة واحدة (1 Set) | يستهدف كل موضع على حدة بالترتيب؛ بينما تظل باقي المواضع ثابتة كما هي في الطلب الأصلي. | فحص ثغرات XSS / SQLi، أو عمل Fuzzing لباراميتر واحد. |
| **Battering Ram** | قائمة واحدة (1 Set) | يضع نفس القيمة في **كافة** المواضع في نفس الطلب في ذات الوقت. | الحالات التي تتطلب تطابق قيمتين مثل `username=admin&password=admin`، أو ملء حقلين بنفس القيمة. **(ليس لأخطاء Race Conditions)**. |
| **Pitchfork** | قوائم متعددة (حتى 20 قائمة) | يحقن القوائم بالتوازي بناءً على الفهرس (Index-based)؛ القيمة الأولى من القائمة 1 مع القيمة الأولى من القائمة 2. | هجمات **Credential Stuffing** بقوائم مسربة مرتبة مسبقاً (يوزر مع باسورده المقابل). |
| **Cluster Bomb** | قوائم متعددة (حتى 20 قائمة) | يجرب كل التباديل والتوافيق الرياضية الممكنة (Cartesian Product)؛ كل كلمة من قائمة 1 مع كل كلمات قائمة 2. | هجمات **Brute Force** الشاملة لتخمين اليوزر والباسورد دون معرفة الارتباط بينهما. |

### 2. معضلة خنق السرعة في النسخة المجانية (Throttling):
في **Burp Suite Community Edition**، يتم تقييد سرعة Intruder عمداً (Throttling)؛ حيث تبدأ الأداة بسرعة عادية لأول 20-30 طلباً ثم تفرض تأخيراً إجبارياً متصاعداً بين كل طلب.
* **البديل السريع في الـ Brute Force الكبير:** استخدام أدوات سريعة من سطر الأوامر مثل `ffuf`، أو استخدام إضافة **Turbo Intruder** (المتاحة في متجر BApp والتي تتجاوز محرك Java التقليدي).

### 3. أتمتة قراءة النتائج (Grep Match & Grep Extract):
مع آلاف الطلبات، يستحيل فتح كل رد يدوياً:
* **Grep - Match:** يضيف عموداً في جدول النتائج يوضح بعلامة صح وجود أو غياب كلمة مفتاحية (مثل `Invalid password` أو `Welcome back`).
* **Grep - Extract:** يستخرج نصوصاً محددة من الرد (مثل أسماء المستخدمين، رسائل الأخطاء، أو توكنات الأمان) ويعرضها في عمود مستقل.
* **Recursive Grep:** يستخرج قيمة من الرد الحالي ويحقنها كـ Payload في الطلب الذي يليه مباشرة (مفيد جداً في تجديد التوكنات المتسلسلة).

---

## 🛠️ التطبيق العملي (Step-by-Step)

### التمرين 1: كسر صفحة تسجيل الدخول (Credential Stuffing عبر Pitchfork)
1. التقط طلب تسجيل دخول في Proxy وأرسله لـ Intruder (`Ctrl + I`).
2. في تاب **Positions**، اضغط زر **Clear §** لمسح التحديد التلقائي.
3. حدد قيمة اسم المستخدم واضغط **Add §**، ثم حدد قيمة كلمة السر واضغط **Add §**:
   ```http
   POST /login HTTP/1.1
   Host: target-app.com
   Content-Type: application/x-www-form-urlencoded

   username=§test§&password=§pass123§
   ```
4. غيّر نوع الهجوم في القائمة المنسدلة إلى **Pitchfork**.
5. انتقل لتاب **Payloads**:
   * في Payload set 1: الصق قائمة بأسماء المستخدمين.
   * في Payload set 2: الصق قائمة بكلمات السر المقابلة لها.
6. انتقل لتاب **Settings**، وانزل إلى قسم **Grep - Match**:
   * امسح القائمة الافتراضية، وأضف العبارة: `Invalid username or password`.
7. اضغط **Start attack**؛ الطلب الناجح ستكون خانة Grep Match عنده غير محددة وسيكون طول الرد (`Length`) مختلفاً.

### التمرين 2: تخطي فحص CSRF المتغير باستخدام Macros
عندما يتطلب فورم اللوجن توكن CSRF جديد مع كل طلب:
1. اذهب إلى `Settings > Sessions > Macros` واضغط **Add**.
2. اختر من السجل طلب `GET /login` ثم اضغط **OK**؛ سيقوم Burp تلقائياً برصد حقل التوكن واستخراجه.
3. تحت قسم **Session handling rules**، اضغط **Add**:
   * في تاب **Scope**: حدد أداة `Intruder` فقط وضمن نطاق الهدف.
   * في تاب **Details > Rule Actions**: أضف إجراء `Run a Macro` واختر الماكرو الذي أنشأته، مع تحديد تحديث باراميتر `csrf` فقط.
4. الآن، قبل كل طلب يرسله Intruder، سيقوم بطلب صفحة اللوجن أولاً لجلب توكن جديد وحقنه آلياً.

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: Username enumeration via response timing](https://portswigger.net/web-security/authentication/password-reset/lab-username-enumeration-via-response-timing)
  * تطبيق عملي لاستخدام هجمات Intruder لتحليل توقيت استجابة السيرفر واكتشاف أسماء المستخدمين الصالحة.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم:**
  * يجمع المهاجم بين قواميس ذكية وقواعد معالجة الحمولات (Payload Processing) لتوليد تركيبات مخصصة تماثل سلوك المستخدم الحقيقي وتتجاوز الحظر.
* **🛡️ منظور المدافع:**
  * لمنع هجمات Intruder، يعتمد المدافعون على تطبيق **Rate Limiting** قوي على مستوى الـ IP وحساب المستخدم، واستخدام آليات إقفال الحساب المؤقت (Account Lockout)، ودمج حمايات CAPTCHA بعد 3 محاولات فاشلة.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **استخدام Battering Ram لكسر اليوزر والباسورد العادي:** في Battering Ram يتم إدخال نفس القيمة في الخانتين، وبالتالي لن تنجح إلا إذا كان اليوزر مساوياً للباسورد.
> 2. **اختلاف أطوال القوائم في هجوم Pitchfork:** إذا كانت قائمة المستخدمين تحتوي على 100 اسم وقائمة الباسوردات 50 فقط، سيتوقف الهجوم بعد المحاولة رقم 50 فوراً ولن يكمل باقي المستخدمين.
> 3. **تجاهل خنق السرعة في النسخة المجانية:** الانتظار لأيام لإنهاء هجوم brute-force في Community Edition بدلاً من استخدام أدوات بديلة متخصصة ومجانية مثل `ffuf`.

---

## 📚 مراجع

1. [PortSwigger Documentation: Burp Intruder Attack Types](https://portswigger.net/burp/documentation/desktop/tools/intruder/attack-types)
2. [PortSwigger Documentation: Handling CSRF tokens in Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder/configure-attack/settings#extract-grep)
3. [PortSwigger Documentation: Session Handling Rules and Macros](https://portswigger.net/burp/documentation/desktop/settings/sessions/rules)

---

| [← السابق: 04. أداة Repeater وهجمات التوازي](./04-repeater.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 06. أدوات Decoder و Comparer و Sequencer →](./06-decoder-comparer-sequencer.md) |
