# 18. كتابة التقرير الأمني وتوثيق الثغرات (Reporting & Documentation)

> 🌐 **اللغة / Language:** [English Version (`en/18-reporting.md`)](../en/18-reporting.md)

---

## 🎯 الهدف من الفصل

تحويل مخرجات الفحص والأدلة المسجلة داخل أدوات Burp Suite إلى **تقارير أمنية احترافية عالية الجودة** مقبولة لدى كبرى منصات صيد الثغرات (HackerOne / Bugcrowd) وعملاء اختبار الاختراق من الشركات، وفهم كيفية تقييم خطورة الثغرة طبقاً لمعيار **CVSS v3.1 / v4.0**، وصياغة خطوات إعادة إنتاج دقيقة وقاطعة (PoC)، وتقديم توصيات برمجية واضحة للمطورين لمعالجة الثغرة بشكل جذري.

---

## 🔍 كيف تعمل الأداة (Technical Explanation)

التقرير الأمني هو المنتج النهائي والوحيد الذي يراه العميل ومسؤولو الأمان؛ مهما كانت براعتك في استغلال الثغرة في Burp، إذا لم يُكتب التقرير بوضوح، فسيتم رفضه أو تخفيض مكافأته أو تجاهله.

### 1. الهيكل القياسي لتقرير الثغرة الاحترافي:
يتكون التقرير الفعال من 6 أركان أساسية:
1. **العنوان والربط المعياري (Title & Classification):** عنوان واضح يحدد نوع الثغرة والمسار المصاب (مثلاً: `IDOR on /api/v1/user/profile leads to full PII leak`) وربطها برقم `CWE` ومعايير `OWASP`.
2. **تقييم الخطورة (Severity & CVSS Score):** حساب درجة الخطورة بنظام **Common Vulnerability Scoring System (CVSS)** لتحديد درجة دقيقة من `0.0` إلى `10.0` بالاعتماد على: ناقل الهجوم (Attack Vector)، تعقيد الهجوم (Attack Complexity)، والصلاحيات المطلوبة (Privileges Required).
3. **الملخص التنفيذي والأثر (Business Impact):** شرح موجه للإدارة غير التقنية يوضح كيف يمكن لهذه الثغرة أن تضر بالشركة مالياً أو قانونياً أو تضر بسمعة العلامة التجارية.
4. **خطوات إعادة الإنتاج (Step-by-Step PoC):** خطوات مرقمة ومحددة بدقة تمكّن فريق الفرز (Triage Team) من تكرار الثغرة وتأكيدها في دقيقتين.
5. **الدليل الخام من Burp Suite (Raw HTTP Evidence):** استخراج الطلب والاستجابة الخام المحفوظين في أداة `Organizer` لتوفير الإثبات القاطع غير القابل للشك.
6. **الحل والتوصيات البرمجية (Remediation):** تقديم كود مصحح أو توجيهات أمنية واضحة للمطورين، بدلاً من مجرد كتابة نصائح عامة مثل "قم بتنظيف المدخلات".

---

## 🛠️ التطبيق العملي (Step-by-Step)

### قالب تقرير أمني جاهز للاستخدام (Markdown Template):

```markdown
# [HIGH] Insecure Direct Object Reference (IDOR) on User Profile Endpoint

## 📌 Metadata
* **Target:** `https://api.target-app.com/v1/users/{id}`
* **Vulnerability Type:** CWE-639: Authorization Bypass Through User-Controlled Key
* **CVSS v3.1 Score:** 7.5 (High) - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N`
* **Date:** 2026-10-28

---

## 📝 Description
An Insecure Direct Object Reference (IDOR) vulnerability was identified in the user profile API endpoint. The application fails to validate whether the requesting authenticated session possesses the authorization rights to access the specified customer record.

---

## 💥 Business Impact
An unauthenticated or low-privileged attacker can enumerate sequential integer IDs to scrape the complete personally identifiable information (PII) of all registered customers—including full names, email addresses, phone numbers, and physical billing addresses—leading to severe regulatory non-compliance (GDPR/CCPA violations) and identity theft.

---

## 🔬 Steps to Reproduce
1. Log into your account as a regular user and note your account ID (`1001`).
2. Navigate to `Profile Settings` and intercept the request in **Burp Suite**.
3. Send the profile retrieval request to **Repeater**:
   `GET /api/v1/users/1001 HTTP/1.1`
4. Change the ID parameter in the URI from `1001` to `1002`.
5. Dispatch the request; observe an HTTP `200 OK` exposing the private data of user `1002`.

---

## 📜 Raw Proof of Concept (from Burp Organizer)

### Request:
```http
GET /api/v1/users/1002 HTTP/1.1
Host: api.target-app.com
Authorization: Bearer <VICTIM_LOW_PRIV_TOKEN>
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: application/json
```

### Response:
```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 214

{
  "id": 1002,
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "phone": "+1-555-0199",
  "ssn": "XXX-XX-1234",
  "address": "123 Elm Street, Dallas, TX"
}
```

---

## 🛡️ Remediation
Enforce server-side authorization checks verifying that the requesting user's identity stored in the session token matches the requested resource ID:

```javascript
// Secure Implementation Example (Node.js/Express)
app.get('/api/v1/users/:id', authenticateToken, async (req, res) => {
  const requestedUserId = req.params.id;
  const authenticatedUserId = req.user.id;

  if (requestedUserId !== authenticatedUserId && !req.user.isAdmin) {
    return res.status(403).json({ error: "Access Denied: Unauthorized access to resource." });
  }

  const profile = await getUserProfile(requestedUserId);
  return res.json(profile);
});
```
```

---

## 🧪 Lab مقترح

* **المعمل التطبيقي:** [PortSwigger Academy: User ID controlled by request parameter with data leakage in redirect](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)
  * حل المعمل، استخرج الـ HTTP Request/Response الخام عبر Organizer، واكتب تقريراً أمنياً كاملاً بناءً على القالب المرفق أعلاه.

---

## ⚔️ منظور المهاجم (Attacker) vs. 🛡️ منظور المدافع (Defender)

* **⚔️ منظور المهاجم / صائد الثغرات:**
  * كتابة تقرير احترافي وواضح يضمن قبول الثغرة من أول مراجعة، ويسرع صرف المكافأة (Bounty)، ويمنع حدوث خلافات على تقييم الخطورة (Severity Disputes).
* **🛡️ منظور المدافع / مهندس الأمان:**
  * التقرير المتكامل يوفر للمهندسين الوقت المستغرق في محاولة فهم الثغرة، ويتيح للفريق البرمجي سد الثغرة مباشرة دون إدخال ترقيعات سطحية قابلة للتجاوز.

---

## ❌ أخطاء شائعة للمبتدئين

> [!WARNING]
> 1. **المبالغة غير المبررة في تقييم الخطورة (Severity Inflation):** تصنيف ثغرة Self-XSS أو غياب ترويسة أمان بسيطة على أنها ثغرة حرجة (Critical)، مما يؤدي لفقدان المصداقية وخفض نقاط السمعة (Reputation Points).
> 2. **تقديم تقرير يحتوي على سكرين شوت فقط:** إرفاق صورة بدون كتابة خطوات واضحة وبدون توفير الطلب والرد بصيغة نصية خام، مما يجبر فريق الفرز على طلب مزيد من المعلومات وتأخير المعالجة.
> 3. **عدم إثبات الأثر الحقيقي (Lack of Impact):** الاكتفاء بإثبات تنفيذ أمر `alert(1)` في XSS دون شرح كيف يمكن استغلاله لسرقة حسابات المستخدمين أو تجاوز صلاحيات الموقع.

---

## 📚 مراجع

1. [FIRST CVSS v3.1 Specification Guide](https://www.first.org/cvss/v3.1/specification-document)
2. [HackerOne: How to Write a Quality Vulnerability Report](https://www.hackerone.com/blog/how-write-good-vulnerability-report)
3. [Bugcrowd Vulnerability Rating Taxonomy (VRT)](https://bugcrowd.com/vulnerability-rating-taxonomy)

---

| [← السابق: 17. منهجية الفحص الشاملة](./17-pentest-workflow.md) | [الفهرس](../README.md#🗂️-فهرس-الفصول-النسخة-العربية---ar) | [التالي: 19. أشهر أخطاء المبتدئين →](./19-beginner-mistakes.md) |
