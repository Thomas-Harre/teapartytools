# Setup Guide — Tea Party Tools Landing Page

## Overview

The landing page is a single HTML file with no build step. Form submissions are sent to a Google Apps Script endpoint that writes rows to a Google Sheet. Hosting is via GitHub Pages (free).

There are three steps:
1. Set up the Google Sheet + Apps Script (the "backend")
2. Update the HTML with your script URL and email
3. Push to GitHub Pages

---

## Step 1: Google Sheet + Apps Script

### 1a. Create the Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new spreadsheet.
2. Name it something like `Tea Party Access Requests`.
3. In Row 1, add these headers (one per column, exactly as written):

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| Timestamp | Product | Name | Email | Firm | Role | Notes |

4. (Optional) Freeze Row 1: Select Row 1 → View → Freeze → 1 row.

### 1b. Add the Apps Script

1. In your spreadsheet, go to **Extensions → Apps Script**.
2. Delete any code in the editor and replace it with this:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);
    
    sheet.appendRow([
      data.submitted || new Date().toISOString(),
      data.product || '',
      data.name || '',
      data.email || '',
      data.firm || '',
      data.role || '',
      data.notes || ''
    ]);
    
    // Optional: send yourself an email notification
    // Uncomment the lines below and replace the email address:
    //
    // MailApp.sendEmail({
    //   to: 'YOUREMAIL@example.com',
    //   subject: 'New access request: ' + (data.product || 'Unknown'),
    //   body: 'Name: ' + data.name + '\n' +
    //         'Email: ' + data.email + '\n' +
    //         'Firm: ' + data.firm + '\n' +
    //         'Role: ' + data.role + '\n' +
    //         'Product: ' + data.product + '\n' +
    //         'Notes: ' + data.notes + '\n' +
    //         'Submitted: ' + data.submitted
    // });
    
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({ result: 'error', error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Click **Save** (Ctrl+S / Cmd+S). Name the project (e.g., `AccessFormHandler`).

### 1c. Deploy as a Web App

1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" → choose **Web app**.
3. Set:
   - **Description**: `Access form handler`
   - **Execute as**: `Me`
   - **Who has access**: `Anyone`
4. Click **Deploy**.
5. It will ask you to authorise. Click **Authorise access**, choose your Google account, and if you see "Google hasn't verified this app", click **Advanced → Go to AccessFormHandler (unsafe)** — this is fine, it's your own script.
6. **Copy the Web app URL** — it will look like:
   `https://script.google.com/macros/s/AKfycbx.../exec`

Keep this URL — you need it for Step 2.

---

## Step 2: Update the HTML

Open `index.html` and make two changes:

### 2a. Add your Apps Script URL

Find this line near the bottom of the file:

```javascript
const GOOGLE_SCRIPT_URL = 'YOUR_GOOGLE_APPS_SCRIPT_URL_HERE';
```

Replace it with your actual URL:

```javascript
const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbx.../exec';
```

### 2b. Add your email address

Search the file for `YOUREMAIL@example.com` (appears twice — once in the error message, once in the footer) and replace both with your actual contact email.

---

## Step 3: Deploy on GitHub Pages

### 3a. Create a repository

1. Go to [github.com/new](https://github.com/new).
2. Name it something like `teapartytools.github.io` (if you want it as a GitHub user page) or just `tea-party-site`.
3. Set it to **Public** (required for free GitHub Pages).
4. Don't initialise with a README.

### 3b. Push the file

```bash
# In a new local folder:
git init tea-party-site
cd tea-party-site
# Copy index.html into this folder, then:
git add index.html
git commit -m "Initial landing page"
git remote add origin https://github.com/YOURUSERNAME/tea-party-site.git
git branch -M main
git push -u origin main
```

### 3c. Enable GitHub Pages

1. In the repo, go to **Settings → Pages**.
2. Under "Source", select **Deploy from a branch**.
3. Choose **main** branch, **/ (root)** folder.
4. Click **Save**.

Your site will be live within a minute or two at:
`https://YOURUSERNAME.github.io/tea-party-site/`

### 3d. (Optional) Custom domain

If you have a domain (e.g., `teapartytools.co.nz`):

1. In Settings → Pages, type your domain under "Custom domain".
2. At your domain registrar, add a CNAME record:
   - Name: `www` (or `@` for apex)
   - Value: `YOURUSERNAME.github.io`
3. Wait for DNS propagation (usually < 1 hour).
4. Enable "Enforce HTTPS" once the certificate is provisioned.

---

## Testing

1. Before configuring the Google Script URL, the form works in "demo mode" — submissions log to the browser console (F12 → Console tab) but don't go anywhere.
2. After configuring, submit a test request and check your Google Sheet. A new row should appear within a few seconds.
3. Note: because we use `mode: 'no-cors'` for the fetch request (required by Apps Script), the browser can't read the response. The form assumes success after sending. If something goes wrong server-side, you'll see the error in the Apps Script execution log (Extensions → Apps Script → Executions).

---

## Optional Enhancements

- **Email notifications**: Uncomment the `MailApp.sendEmail` block in the Apps Script to get an email each time someone submits.
- **Favicon**: Add a favicon by placing a `favicon.ico` in the same folder and adding `<link rel="icon" href="favicon.ico">` to the `<head>`.
- **Analytics**: Add a simple privacy-respecting analytics script (e.g., Plausible, Fathom) if you want to track page views.
- **Custom domain**: See Step 3d above.

---

## Maintenance

- **To update copy**: Edit `index.html`, commit, push. Changes go live in ~1 minute.
- **To view submissions**: Open your Google Sheet.
- **To redeploy the Apps Script** (if you change the code): Deploy → Manage deployments → Edit → Version: New version → Deploy.
