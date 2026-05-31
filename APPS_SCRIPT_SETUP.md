# 🔌 تفعيل تسجيل نشاط الدخول (Google Apps Script)

موقع النتائج يعمل بالكامل بدون هذا الإعداد، لكن قسم **"نشاط الدخول"** في صفحة المدرس لن يعرض بيانات حقيقية حتى تُكمل هذه الخطوات.

## نظرة عامة

نستخدم **Google Apps Script** + **Google Sheet** كقاعدة بيانات مجانية بدون سيرفر:
- كل دخول طالب (ناجح أو فاشل) يُرسَل إلى Apps Script
- Apps Script يخزّن السجل في Google Sheet
- صفحة المدرس تقرأ السجل وتعرض إحصائيات (عبر JSONP لتجاوز CORS)

## الخطوات

### 1. أنشئ Google Sheet جديد

اذهب إلى [sheets.google.com](https://sheets.google.com) → New → Blank

سمّ الورقة الأولى `Activity` وأضف هذه الرؤوس في الصف الأول:

| timestamp | uid | name | status | device | browser | userAgent | attemptedUid |
|---|---|---|---|---|---|---|---|

### 2. أنشئ Apps Script

في نفس الـSheet: **Extensions → Apps Script**

احذف الكود الموجود والصق هذا:

```javascript
const PASSWORD = "AI4203"; // يجب أن يطابق كلمة سر admin.html
const SHEET_NAME = "Activity";

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActive().getSheetByName(SHEET_NAME);
    sheet.appendRow([
      new Date().toISOString(),
      data.uid || '',
      data.name || '',
      data.status || '',
      data.device || '',
      data.browser || '',
      data.userAgent || '',
      data.attemptedUid || ''
    ]);
    return ContentService.createTextOutput(JSON.stringify({ok: true}))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ok: false, error: String(err)}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  const callback = e.parameter.callback;
  const password = e.parameter.password;

  if (password !== PASSWORD) {
    const body = JSON.stringify({ok: false, error: "Unauthorized"});
    return callback
      ? ContentService.createTextOutput(callback + "(" + body + ")").setMimeType(ContentService.MimeType.JAVASCRIPT)
      : ContentService.createTextOutput(body).setMimeType(ContentService.MimeType.JSON);
  }

  try {
    const sheet = SpreadsheetApp.getActive().getSheetByName(SHEET_NAME);
    const values = sheet.getDataRange().getValues();
    const headers = values[0];
    const rows = values.slice(1).map(r => {
      const obj = {};
      headers.forEach((h, i) => { obj[h] = r[i] instanceof Date ? r[i].toISOString() : r[i]; });
      return obj;
    });
    const body = JSON.stringify({ok: true, rows: rows});
    return callback
      ? ContentService.createTextOutput(callback + "(" + body + ")").setMimeType(ContentService.MimeType.JAVASCRIPT)
      : ContentService.createTextOutput(body).setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    const body = JSON.stringify({ok: false, error: String(err)});
    return callback
      ? ContentService.createTextOutput(callback + "(" + body + ")").setMimeType(ContentService.MimeType.JAVASCRIPT)
      : ContentService.createTextOutput(body).setMimeType(ContentService.MimeType.JSON);
  }
}
```

### 3. انشر السكربت كـ Web App

1. اضغط **Deploy → New deployment** (أعلى يمين الـ Apps Script editor)
2. اختر النوع: **Web app**
3. الإعدادات:
   - **Execute as**: Me
   - **Who has access**: Anyone (مهم — بدونها لن يستطيع الموقع الوصول)
4. اضغط **Deploy**
5. وافق على الصلاحيات إذا طُلبت
6. **انسخ الـ Web app URL** الذي يظهر (شكله: `https://script.google.com/macros/s/AKfyc.../exec`)

### 4. الصق الـ URL في الموقع

افتح هذين الملفين:

- `site/index.html`
- `site/admin.html`

في كل منهما ابحث عن:

```javascript
const SCRIPT_URL = ""; // <-- e.g. "https://script.google.com/macros/s/AKfyc.../exec"
```

والصق الـ URL بين علامتي التنصيص:

```javascript
const SCRIPT_URL = "https://script.google.com/macros/s/AKfycXXXXXX.../exec";
```

### 5. أعد رفع الموقع (لو منشور)

```bash
git add site/index.html site/admin.html
git commit -m "enable login activity tracking"
git push
```

## ✅ كيف تتحقق من التفعيل

1. افتح `index.html` وأدخل رقم طالب صحيح → سجل الدخول
2. افتح `admin.html` بكلمة السر `AI4203`
3. انتقل إلى قسم **"نشاط الدخول"** في الأسفل
4. ستظهر إحصائيات: إجمالي التسجيلات، طلبة دخلوا، إلخ
5. افتح الـ Google Sheet للتحقق من ظهور صف جديد

## 🔒 ملاحظات أمنية

- كلمة السر `PASSWORD` في Apps Script يجب أن **تطابق** كلمة سر admin (افتراضياً `AI4203`)
- البيانات في الـSheet تبقى خاصة بحسابك على Google (الطلاب لا يستطيعون رؤيتها)
- لو غيّرت `PASSWORD` في Apps Script، يجب تحديثها في `admin.html` كذلك

## 🐛 استكشاف الأخطاء

| المشكلة | الحل |
|---|---|
| `خطأ في الاتصال بـ Google Apps Script` | تأكد أن النشر مفعّل و"Who has access" = Anyone |
| `Unauthorized` | كلمة السر في Apps Script لا تطابق كلمة سر admin |
| `انتهت المهلة بدون استجابة` | الـ URL خاطئ، أو السكربت لم يُحفظ بعد التعديل |
| لا تظهر تسجيلات جديدة | تأكد أن `SCRIPT_URL` معبّأ في `index.html` أيضاً |
