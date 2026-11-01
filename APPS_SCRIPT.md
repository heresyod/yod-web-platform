# YOD auto-reply + subscriber log — Google Apps Script setup

One-time setup. Takes ~5 minutes.

What this does for you:
- Sends a branded HTML auto-reply to every new subscriber from your Gmail
- Logs every subscriber (email, name, source form, timestamp, IP country if available) to a Google Sheet in your Drive
- Skips the auto-reply if the email already exists in the sheet (server-side dedupe)
- Keeps a growing list you can use at launch day for your real announcement blast

## 1. Create the Google Sheet (first)
1. Open https://sheets.google.com → **Blank spreadsheet**
2. Rename it to `YOD subscribers`
3. In row 1, put these column headers in order:
   - A1: `Timestamp`
   - B1: `Email`
   - C1: `Name`
   - D1: `Source`
   - E1: `User Agent`
4. Copy the Sheet's URL — you need the ID from it. The URL looks like
   `https://docs.google.com/spreadsheets/d/`**`1AbCdeFGhiJKLmnOpqrsTUvwxYZ`**`/edit`
   The bold part is the Sheet ID. Save it.

## 2. Create the script
1. Open **https://script.google.com** (signed in as `heresyod@gmail.com`)
2. Click **New project**
3. Delete the default `function myFunction()` code
4. Paste the entire script from `apps-script.gs` below
5. At the top of the script, replace `1PkyvxIDXrmR4k5ui_mZxuJrSxu6QiWNyVVKmMn614d8` with the Sheet ID from step 1
6. Click **Save** (💾), name the project `YOD Auto-Reply`

## 3. Deploy as a Web app
1. Top-right, click **Deploy** → **Manage deployments** (if you already have one) OR **New deployment**
2. If editing an existing deployment: pencil (✏️) → **Version: New version** → **Deploy**
3. If new: Click the ⚙️ gear next to "Select type" → **Web app**
   - **Description:** `YOD auto-reply v2`
   - **Execute as:** `Me (heresyod@gmail.com)`
   - **Who has access:** **Anyone**
   - Click **Deploy**
4. Google will ask for extra permissions this time (because the script now writes to Sheets) — click **Authorize access**, pick your account, click **Advanced** → **Go to YOD Auto-Reply (unsafe)** → **Allow**
5. **Copy the Web app URL** (unchanged if you edited; new if first-time)

## 4. Paste the URL in index.html (only if the URL is new)
Edit `index.html` → replace the string value of `APPS_SCRIPT_URL`.

## 5. Test
Submit a test email on https://www.theyod.com.
- Admin notification → heresyod@gmail.com (via Web3Forms)
- Auto-reply → test inbox (from your Gmail via Apps Script)
- Sheet gets a new row with that submission
- Submit the same email again → no second auto-reply, no duplicate row

Open Apps Script → **Executions** on the left to see run logs if something looks off.

---

## apps-script.gs — paste this entire file

```js
// ============================================================
// YOD backend — handles form POST from theyod.com
//  1. Validates email
//  2. Deduplicates against the YOD subscribers Sheet
//  3. Writes the subscription to the sheet
//  4. Sends a branded HTML auto-reply from your Gmail
// ============================================================

var SHEET_ID = '1PkyvxIDXrmR4k5ui_mZxuJrSxu6QiWNyVVKmMn614d8';    // ← set this once
var SHEET_NAME = 'Sheet1';               // tab name — leave as default
var OWNER_EMAIL = 'heresyod@gmail.com';

function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var toEmail = String(data.email || '').trim().toLowerCase();
    var toName  = String(data.name  || '').trim().slice(0, 60);
    var kind    = String(data.formType || 'notify').toLowerCase();

    // ---- Validate ----
    if (!toEmail || toEmail.length > 120) return json({ success:false, error:'Invalid email length' });
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/.test(toEmail)) return json({ success:false, error:'Invalid email format' });
    var blocked = ['mailinator.com','tempmail.com','10minutemail.com','guerrillamail.com','trash-mail.com','yopmail.com','sharklasers.com'];
    if (blocked.indexOf(toEmail.split('@')[1]) !== -1) return json({ success:false, error:'Blocked email domain' });
    if (toName && !/^[A-Za-z][A-Za-z\s.'\-]{1,59}$/.test(toName)) toName = '';

    // ---- Dedupe against sheet ----
    var sheet = SpreadsheetApp.openById(SHEET_ID).getSheetByName(SHEET_NAME)
                || SpreadsheetApp.openById(SHEET_ID).getSheets()[0];
    var existing = sheet.getRange(2, 2, Math.max(sheet.getLastRow() - 1, 1), 1).getValues(); // column B
    var already = false;
    for (var i = 0; i < existing.length; i++) {
      if (String(existing[i][0]).trim().toLowerCase() === toEmail) { already = true; break; }
    }

    if (already) {
      return json({ success:true, alreadySubscribed:true });
    }

    // ---- Log to sheet ----
    sheet.appendRow([
      new Date(),
      toEmail,
      toName,
      kind,
      ''  // reserved for user-agent if you want to pass it from the client later
    ]);

    // ---- Send auto-reply ----
    var subject = (kind === 'early')
      ? 'YOD — your early access request is in'
      : 'YOD — you\'re on the list';
    var html = buildEmail(kind, toName);

    GmailApp.sendEmail(toEmail, subject, stripHtml(html), {
      htmlBody: html,
      name: 'YOD',
      replyTo: OWNER_EMAIL
    });

    return json({ success:true, alreadySubscribed:false });
  } catch (err) {
    return json({ success:false, error:String(err) });
  }
}

function doGet() {
  return ContentService.createTextOutput('YOD backend is live.');
}

function json(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}

function stripHtml(h) { return h.replace(/<[^>]+>/g, ' ').replace(/\s+/g, ' ').trim(); }

function buildEmail(kind, name) {
  var greeting = name ? ('Hi ' + name + ',') : 'Hi,';
  var intro, para;
  if (kind === 'early') {
    intro = 'Your early access request has been received. Thank you for choosing to define your own world with YOD.';
    para  = 'Priority access to our debut collection is limited. We will be in touch personally before the first drop goes public, with early access to our premium oversized tees.';
  } else {
    intro = 'Thank you for signing up. You are now on our priority list.';
    para  = 'We will be in touch personally before the first drop goes public — with early access to our debut collection of premium oversized tees, built for those who define their own world.';
  }
  return [
    '<div style="background:#000;padding:40px 20px;font-family:Georgia,serif">',
      '<div style="max-width:560px;margin:0 auto;background:#0a0a0a;border:1px solid #222;padding:36px 32px;color:#d2d2d2;line-height:1.75;font-size:15px">',
        '<div style="text-align:center;font-family:Georgia,serif;font-size:38px;letter-spacing:8px;color:#e0e0e0;margin-bottom:6px">YOD</div>',
        '<div style="text-align:center;font-size:10px;letter-spacing:6px;color:#8a7355;text-transform:uppercase;margin-bottom:28px">Wear Your Definition</div>',
        '<p style="margin:0 0 14px">' + greeting + '</p>',
        '<p style="margin:0 0 14px">' + intro + '</p>',
        '<p style="margin:0 0 14px">' + para + '</p>',
        '<p style="margin:0 0 14px">Follow the journey:<br><a href="https://www.instagram.com/yod.definition/" style="color:#c4a57b">@yod.definition on Instagram</a></p>',
        '<p style="margin:24px 0 0;font-style:italic;color:#c4a57b;text-align:center;letter-spacing:4px;text-transform:uppercase;font-size:11px">— Wear Your Definition —</p>',
        '<p style="margin:28px 0 0;font-size:11px;color:#777;text-align:center;letter-spacing:1px">Team YOD · <a href="https://www.theyod.com" style="color:#8a7355">theyod.com</a></p>',
      '</div>',
    '</div>'
  ].join('');
}
```
