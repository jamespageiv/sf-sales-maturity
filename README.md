# Salesforce B2B Sales Maturity Diagnostic — v3

A professional diagnostic tool for assessing B2B sales maturity across 7 RevOps dimensions, with a consultant admin dashboard, company-level analytics, and a live question editor.

---

## 📁 Repository Files

| File | Audience | Purpose |
|---|---|---|
| `index.html` | Respondents | Assessment + personalized results report |
| `admin.html` | You (password protected) | Analytics dashboard — Overview, All Responses, Companies, Trends, Best Practices Library |
| `questions.html` | You (password protected) | Live question editor — add, edit, reorder, activate/deactivate |
| `README.md` | You | This setup guide |

---

## 🚀 Step 1 — Host on GitHub Pages (5 minutes, free)

1. Go to **github.com** → Sign in or create a free account
2. Click **New repository** → Name it `sf-sales-maturity` → Set to **Public** → **Create repository**
3. Click **uploading an existing file** → drag all 3 HTML files + README into the box → **Commit changes**
4. Go to **Settings → Pages** → Source: **Deploy from a branch → main → / (root)** → **Save**

Your live URLs (replace `YOUR-USERNAME`):
- **Assessment:** `https://YOUR-USERNAME.github.io/sf-sales-maturity/`
- **Admin dashboard:** `https://YOUR-USERNAME.github.io/sf-sales-maturity/admin.html`
- **Question editor:** `https://YOUR-USERNAME.github.io/sf-sales-maturity/questions.html`

> Pages can take 1–3 minutes to go live the first time. Subsequent updates are ~60 seconds.

---

## 📊 Step 2 — Connect Google Sheets (15 minutes)

### 2a — Create the spreadsheet
- Go to **sheets.google.com** → Create a blank sheet → Name it **"Sales Maturity Responses"**

### 2b — Open Apps Script
- In the sheet: **Extensions → Apps Script** → Delete all existing code → Paste the full script below

### 2c — Full Apps Script (paste this entire block)

```javascript
// HANDLES ALL THREE FILES: index.html (submit), admin.html (read), questions.html (save/read)

function doPost(e) {
  var data = JSON.parse(e.postData.contents);

  // Save questions from questions.html
  if (data.action === 'saveQuestions') {
    var qSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Questions')
      || SpreadsheetApp.getActiveSpreadsheet().insertSheet('Questions');
    qSheet.getRange(1, 1).setValue(JSON.stringify(data.questions));
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  // Save assessment response from index.html
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(Object.keys(data));
  }
  sheet.appendRow(Object.values(data));
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}

function doGet(e) {
  // Return all responses for admin.html
  if (e.parameter.action === 'getAll') {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var rows = sheet.getDataRange().getValues();
    if (rows.length < 2) return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok', data: [] }))
      .setMimeType(ContentService.MimeType.JSON);
    var headers = rows[0];
    var data = rows.slice(1).map(function(row) {
      var obj = {};
      headers.forEach(function(h, i) { obj[h] = row[i]; });
      return obj;
    });
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok', data: data }))
      .setMimeType(ContentService.MimeType.JSON);
  }

  // Return questions for index.html and questions.html
  if (e.parameter.action === 'getQuestions') {
    var qSheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Questions');
    if (!qSheet) return ContentService
      .createTextOutput(JSON.stringify({ questions: [] }))
      .setMimeType(ContentService.MimeType.JSON);
    var val = qSheet.getRange(1, 1).getValue();
    var questions = val ? JSON.parse(val) : [];
    return ContentService
      .createTextOutput(JSON.stringify({ questions: questions }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### 2d — Deploy as Web App
1. Click **Deploy → New Deployment**
2. Click the ⚙️ gear icon → Select **Web app**
3. Set **Execute as:** Me
4. Set **Who has access:** Anyone
5. Click **Deploy** → **Copy the Web App URL**

### 2e — Paste the URL into all 3 files
Open each file in a text editor and find the `GOOGLE_SHEETS_API` / `GOOGLE_SHEETS_WEBHOOK` constant:

**index.html** — find:
```javascript
const GOOGLE_SHEETS_WEBHOOK = "YOUR_APPS_SCRIPT_URL_HERE";
const QUESTIONS_SHEET_URL   = "YOUR_APPS_SCRIPT_URL_HERE";
```

**admin.html** — find:
```javascript
const GOOGLE_SHEETS_API = "YOUR_APPS_SCRIPT_URL_HERE";
```

**questions.html** — find:
```javascript
const GOOGLE_SHEETS_API = "YOUR_APPS_SCRIPT_URL_HERE";
```

Replace all four placeholders with your Web App URL. Save all files and re-upload to GitHub.

> ⚠️ **Important:** Every time you edit the Apps Script code, you must create a **New Deployment** (not update existing). Copy the new URL and update all 3 files.

---

## 🔐 Step 3 — Change Your Passwords

Open `admin.html` and `questions.html` in a text editor. Find:
```javascript
const ADMIN_PASSWORD = "salesforce2024";
```
Replace with a secure password in both files. Save and re-upload to GitHub.

> Keep `admin.html` and `questions.html` URLs private — share only `index.html` with respondents.

---

## 🧪 Tester Domain Management

Anyone completing the assessment with a `@salesforce.com` email is automatically grouped as **🧪 Testers** in your admin dashboard and excluded from all customer analytics.

To add more tester domains, open `index.html` and `admin.html` and find:
```javascript
const TESTER_DOMAINS = ["salesforce.com"];
```
Add more domains: `["salesforce.com", "partner.com", "youragency.com"]`

---

## ✏️ Editing Questions (questions.html)

1. Go to `https://YOUR-USERNAME.github.io/sf-sales-maturity/questions.html`
2. Sign in with your admin password
3. **Edit** any question's text or answer options inline
4. **Reorder** questions using the ↑ ↓ arrows
5. **Deactivate** questions to hide them from respondents without deleting history
6. **Add** new questions with the "+ Add Question" button
7. Changes save automatically to Google Sheets and go live in the assessment immediately

> Deactivated questions remain in the database so historical responses are preserved and still score correctly.

---

## 🏢 Company Analytics (admin.html → Companies tab)

- Every respondent is grouped by their **email domain** (e.g., `acme.com`)
- The Companies tab shows each domain with: aggregate score, dimension heatmap, individual respondents, and consultant talking points based on their lowest 3 dimensions
- `@salesforce.com` (and other tester domains) appear in a separate **Testers** group at the top
- Multiple respondents from the same company are blended into an aggregate company view

---

## 📚 Best Practices Library (admin.html → Best Practices tab)

A consultant-only reference drawn from the Agent-Ready Briefing document. Organized by dimension × tier, showing:
- What organizations at each tier typically look like
- Salesforce signals that indicate maturity level
- Recommended interventions (never visible to respondents)

Use this when building your prescription after reviewing the diagnostic results.

---

## 🔄 Updating Files After Initial Setup

1. Make your edits locally in a text editor
2. Go to your GitHub repository
3. Click the filename → click the pencil ✏️ icon to edit inline, OR drag a new version into the repository
4. Click **Commit changes**
5. Live site updates in ~60 seconds

---

## 📋 Repository Structure

```
sf-sales-maturity/
├── index.html       ← Send this URL to respondents
├── admin.html       ← Your private analytics dashboard  
├── questions.html   ← Your private question editor
└── README.md        ← This file
```

---

*Built for Salesforce Consulting — v3, aligned to the Agent-Ready Briefing for B2B Sales Maturity Diagnostic*
