# Lyniro — post-update setup guide

This file explains the changes that were made to your site folder and how to finish wiring up the waitlist. Once you complete the 5-minute setup below, every signup will land in a Google Sheet you own, with no third-party service in the loop.

---

## 1. Wire the waitlist up to Google Sheets

Estimated time: 5 minutes. You'll need a Google account.

### Step 1 — Create the sheet

1. Go to https://sheets.new (this creates a fresh sheet)
2. Rename it to something like `Lyniro Waitlist`
3. In row 1, add these column headers (left to right):
   ```
   Timestamp | Email | Current Tool | Source Page | Referrer | User Agent
   ```

### Step 2 — Open Apps Script

1. In your sheet, click `Extensions` → `Apps Script`
2. Delete the default `function myFunction() {}` placeholder
3. Paste the code below in full:

```javascript
// Lyniro waitlist receiver
// Writes every submission as a new row in the active spreadsheet.
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    sheet.appendRow([
      new Date(),                          // Timestamp (server)
      data.email        || '',             // Email
      data.currentTool  || '',             // Current Tool
      data.source       || '',             // Source Page
      data.referrer     || '',             // Referrer
      data.userAgent    || ''              // User Agent
    ]);

    // Optional: send yourself an email each time someone signs up.
    // Uncomment the two lines below and set NOTIFY_EMAIL to your address.
    // var NOTIFY_EMAIL = 'you@lyniro.com';
    // MailApp.sendEmail(NOTIFY_EMAIL, 'New Lyniro waitlist signup: ' + data.email, JSON.stringify(data, null, 2));

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Allows you to test the deployment in a browser by visiting the URL.
function doGet() {
  return ContentService
    .createTextOutput('Lyniro waitlist webhook is alive.')
    .setMimeType(ContentService.MimeType.TEXT);
}
```

4. Click the floppy-disk save icon (or `Ctrl/Cmd + S`). Name the project `Lyniro Waitlist`.

### Step 3 — Deploy as a web app

1. Top right of the Apps Script editor, click `Deploy` → `New deployment`
2. Click the gear icon next to "Select type" and choose `Web app`
3. Fill in:
   - Description: `Lyniro waitlist v1`
   - Execute as: `Me (your-google-account)`
   - Who has access: `Anyone`
4. Click `Deploy`
5. Google will ask you to authorise the script — accept the permissions (it only needs Sheets + Mail if you uncomment notifications)
6. After deployment, copy the `Web app URL`. It looks like:
   ```
   https://script.google.com/macros/s/AKfycby.../exec
   ```

### Step 4 — Paste the URL into the site

1. Open `index.html` in your site folder
2. Find this line near the very bottom (around line 968):
   ```javascript
   const WAITLIST_WEBHOOK_URL = ''; // <-- paste your Apps Script web app URL here
   ```
3. Paste your URL inside the quotes:
   ```javascript
   const WAITLIST_WEBHOOK_URL = 'https://script.google.com/macros/s/AKfycby.../exec';
   ```
4. Save and re-deploy your site (push to whatever host serves lyniro.com)

### Step 5 — Test it

1. Open https://lyniro.com in an incognito window
2. Scroll to the founding-design-partner form
3. Submit a test email
4. Check your Google Sheet — a new row should appear within a few seconds

If it doesn't work, open your browser's DevTools console and look for errors. The most common cause is forgetting to set "Who has access: Anyone" during deployment.

### What happens before you finish setup

Until you fill in `WAITLIST_WEBHOOK_URL`, the form falls back gracefully — clicking submit opens the visitor's email app pre-filled with their email, the current tool they selected, and the page they came from, addressed to `hello@lyniro.com`. So no signups are lost during the gap between deploy and Apps Script setup.

### Why Apps Script and not Formspree / Netlify Forms?

- You own the data — it lives in your Google Drive, no third-party retention
- Free, no quota until you hit hundreds of thousands of submissions
- One Google Sheet doubles as your CRM until you outgrow it
- Easy to add column logic (welcome email automation, dedupe, scoring) later

---

## 2. SEO changes applied

Below is a summary of what was edited in your codebase. Specific commit-style diffs follow.

### What changed

- Expanded `application/ld+json` schema on the homepage from just `Organization + WebSite` to also include `SoftwareApplication` and `FAQPage` (mirroring the FAQ block on `/customer-onboarding-software/`)
- Added `BreadcrumbList` schema on `/customer-onboarding-software/`, `/saas-onboarding-software/`, `/customer-success-software/`, `/client-onboarding-software/`, and `/vs/*` pages
- Added `FAQPage` schema on `/customer-onboarding-software/` and the `/vs/gainsight/`, `/vs/churnzero/`, `/vs/dock/`, `/vs/rocketlane/` pages (these have FAQ blocks in the visible content)
- Added `Article` schema on the top-traffic blog posts (the ones in your hero blog cluster and footer links)
- Tightened the homepage `<title>` from 64 chars to 58 chars: `Customer Onboarding Software for CS Leaders | Lyniro`
- Fixed the self-referencing internal link in `/blog/why-saas-customers-churn-during-onboarding/` (the anchor that linked back to itself now points to a relevant deeper cluster page)
- Added/improved descriptive `alt` text on hero, dashboard mockup, and feature illustration `<img>` tags where they were missing or generic
- No changes to `_redirects` — it is already correct (force HTTPS, force non-www, that matches your `<link rel=canonical>` declarations)
- Left `robots.txt` and `sitemap.xml` untouched — they were already in good shape

### What you should also do (not code changes)

These are operational items I can't do for you:

- Submit `https://lyniro.com/sitemap.xml` in **Google Search Console** and **Bing Webmaster Tools** (10 min)
- Run **PageSpeed Insights** on top 5 pages to baseline Core Web Vitals — https://pagespeed.web.dev
- Connect a rank tracker (Ahrefs or Semrush) — even the trial — to start measuring the keywords from the audit
- Delete the nested `lyniro_v4/lyniro_v4/` duplicate folder from your deployment source — it's an older copy of your site and shouldn't ship to production
- After your first 3 founding beta partners go live, ship case studies using the structure outlined in the audit doc

### What's still ahead

Three commercial-intent posts are still missing and would be valuable next:

- `/blog/guidecx-alternatives/` and `/vs/guidecx/` — your `/alternatives/guidecx/` and `/vs/guidecx/` folders exist but appear to be older drafts; confirm they're live and indexed
- `/blog/onramp-alternatives/` and `/vs/onramp/` — fully missing, capture the brand search before they grow
- `/blog/arrows-alternatives/` and `/vs/arrows/` — same as above

The pillar guide on "VP of Customer Success" (role, responsibilities, KPIs) is the highest-leverage net-new content piece for capturing ICP awareness traffic.

Run the SEO audit again in 90 days and we'll have real ranking data to grade against.
