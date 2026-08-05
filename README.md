# نظام الاستعلام الإلكتروني الموحد v4.4
## مديرية غاز جنوب المنطقة الوسطى

نسخة ثابتة (Static) تعمل على GitHub Pages أو أي استضافة HTML، بدلاً من Google Apps Script Frontend.

---

## بنية المشروع

```
/
├── index.html          ← الصفحة الكاملة (HTML + CSS + JS)
├── README.md
```

---

## التحويلات المُنجزة

| من | إلى |
|---|---|
| `google.script.run.fn(args)` | `apiRun('fn', [args], onSuccess, onFailure)` |
| `<? for sheetList ?>` (server template) | `initSheetList()` — يجلب الشهور من الـ API |
| كل الوظائف (checkAdminAuth, getResult, ...) | `fetch(API_URL, { method: 'POST', body: JSON.stringify({action, args}) })` |

---

## متطلبات الـ API (Google Apps Script)

يجب أن يدعم الـ Apps Script استقبال طلبات **POST** عبر دالة `doPost(e)` بالتنسيق:

```json
{ "action": "functionName", "args": [...] }
```

### دالة `getSheetList` المطلوبة إضافتها للـ Code.gs:

```javascript
function getSheetList() {
  var ss = SpreadsheetApp.openById(SPREADSHEET_ID);
  var disabled = getDisabledSheetsList_(ss);
  var names = ss.getSheets()
    .map(function(s) { return s.getName(); })
    .filter(function(n) { return n.includes("-202") && !disabled.includes(n); });
  names.sort(function(a, b) {
    var pA = a.split('-'), pB = b.split('-');
    return (parseInt(pB[1] + pB[0])) - (parseInt(pA[1] + pA[0]));
  });
  return { sheets: names };
}
```

### دالة `doPost` المطلوبة إضافتها للـ Code.gs:

```javascript
function doPost(e) {
  try {
    var body = JSON.parse(e.postData.contents);
    var action = body.action;
    var args = body.args || [];

    var fnMap = {
      'checkAdminAuth':    function() { return checkAdminAuth(args[0]); },
      'getResult':         function() { return getResult(args[0], args[1]); },
      'getAdminStats':     function() { return getAdminStats(args[0]); },
      'getChartData':      function() { return getChartData(args[0], args[1]); },
      'getLogEntries':     function() { return getLogEntries(args[0], args[1], args[2], args[3], args[4]); },
      'getTopUsers':       function() { return getTopUsers(args[0], args[1]); },
      'getAllSheets':       function() { return getAllSheets(args[0]); },
      'deleteSheetByName': function() { return deleteSheetByName(args[0], args[1]); },
      'toggleSheetStatus': function() { return toggleSheetStatus(args[0], args[1]); },
      'getAdminActivity':  function() { return getAdminActivity(args[0]); },
      'getErrorLogEntries':function() { return getErrorLogEntries(args[0]); },
      'clearQueryLog':     function() { return clearQueryLog(args[0]); },
      'exportLogData':     function() { return exportLogData(args[0]); },
      'getSystemInfo':     function() { return getSystemInfo(args[0]); },
      'searchByName':      function() { return searchByName(args[0], args[1], args[2]); },
      'processUpload':     function() { return processUpload(args[0], args[1], args[2]); },
      'getSheetList':      function() { return getSheetList(); }
    };

    var result = fnMap[action] ? fnMap[action]() : { error: 'action غير معروف: ' + action };

    return ContentService
      .createTextOutput(JSON.stringify(result))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({ error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

> **ملاحظة CORS:** يجب نشر الـ Apps Script بإعدادات "Anyone" ليتمكن المتصفح من الوصول إليه.
> إذا ظهرت أخطاء CORS، يمكن إضافة headers مناسبة في دالة `doPost`.

---

## تشغيل المشروع

```bash
# للمعاينة المحلية (بعد clone)
npx serve .

# أو افتح index.html مباشرة في المتصفح
```

## نشر على GitHub Pages

1. ارفع الملفات على GitHub repository
2. فعّل GitHub Pages من Settings → Pages
3. اختر `main` branch و `/root`
4. الموقع سيكون متاحاً على: `https://<username>.github.io/<repo>/`

---

## التقنيات المستخدمة

- **HTML/CSS/JS** — Vanilla (بدون frameworks)
- **Tailwind CSS** — عبر CDN
- **Lucide Icons** — عبر CDN
- **Chart.js** — للرسوم البيانية
- **SheetJS (XLSX)** — لقراءة ملفات Excel
- **Google Apps Script** — كـ REST API backend
