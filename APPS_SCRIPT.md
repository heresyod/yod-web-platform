# YOD auto-reply — Google Apps Script setup

One-time setup. Takes ~3 minutes.

## 1. Create the script
1. Open **https://script.google.com** (signed in as `heresyod@gmail.com`)
2. Click **New project**
3. Delete the default `function myFunction()` code
4. Paste the entire script from `apps-script.gs` (below)
5. Click **Save** (💾), name the project `YOD Auto-Reply`

## 2. Deploy as a Web app
1. Top-right, click **Deploy** → **New deployment**
2. Click the ⚙️ gear next to "Select type" → choose **Web app**
3. Settings:
   - **Description:** `YOD auto-reply v1`
   - **Execute as:** `Me (heresyod@gmail.com)`
   - **Who has access:** **Anyone**
4. Click **Deploy**
5. Google will ask for permissions — click **Authorize access**, pick your account, click **Advanced** → **Go to YOD Auto-Reply (unsafe)** → **Allow**
6. **Copy the Web app URL** — it looks like `https://script.google.com/macros/s/AKfy.../exec`

## 3. Paste the URL in index.html
1. Open `index.html`
2. Find `APPS_SCRIPT_URL = 'PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE'`
3. Replace the placeholder with the Web app URL you copied
4. Commit and push

## 4. Test
Fill out either form on https://www.theyod.com with a test email you can check.
- Admin notification → arrives at heresyod@gmail.com (Web3Forms)
- Auto-reply → arrives at the test email (from your Gmail via Apps Script)

If the auto-reply doesn't arrive, open the Apps Script project, click **Executions** on the left — any errors will show there.

---

## apps-script.gs — paste this entire file into the Apps Script editor

```js
// YOD auto-reply — handles POST from theyod.com forms
// Sends a branded HTML confirmation to the subscriber from your Gmail.

function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var toEmail = String(data.email || '').trim().toLowerCase();
    var toName  = String(data.name  || '').trim().slice(0, 60);
    var kind    = String(data.formType || 'notify').toLowerCase();

    // Validation
    if (!toEmail || toEmail.length > 120) return json({ success: false, error: 'Invalid email length' });
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/.test(toEmail)) return json({ success: false, error: 'Invalid email format' });
    var blocked = ['mailinator.com','tempmail.com','10minutemail.com','guerrillamail.com','trash-mail.com','yopmail.com','sharklasers.com'];
    var domain = toEmail.split('@')[1];
    if (blocked.indexOf(domain) !== -1) return json({ success: false, error: 'Blocked email domain' });
    if (toName && !/^[A-Za-z][A-Za-z\s.'\-]{1,59}$/.test(toName)) toName = ''; // clean bad names

    var subject = (kind === 'early')
      ? 'YOD — your early access request is in'
      : 'YOD — you\'re on the list';
    var html = buildEmail(kind, toName);

    GmailApp.sendEmail(toEmail, subject, stripHtml(html), {
      htmlBody: html,
      name: 'YOD',
      replyTo: 'heresyod@gmail.com'
    });

    return json({ success: true });
  } catch (err) {
    return json({ success: false, error: String(err) });
  }
}

function doGet() {
  return ContentService.createTextOutput('YOD auto-reply endpoint is live.');
}

function json(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
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
