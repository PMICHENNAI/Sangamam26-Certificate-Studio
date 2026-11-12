# Sangamam26 Certificate Studio – Setup Guide (Volunteers + Delegates)

Files in this folder
- `delegate-certificate-preview.png` – sample of the delegate certificate
- `index.html` – the member-facing web app (single file; certificate artwork from your PDF is embedded)
- `Code.gs` – Google Apps Script backend (member lookup, audit log, executive dashboard)

## Part A – Google Sheet and backend (about 10 minutes)

1. Create a new Google Sheet, e.g. **Sangamam26 Certificate App**.
2. Open **Extensions → Apps Script**. Delete the sample code and paste the full contents of `Code.gs`. Save.
3. In the function dropdown choose **setup** and click **Run**, then run **installBackupTrigger** once (daily Drive backup) and **securityCheck** (confirms your sheet is not shared by link). Approve the permissions when asked (Google shows an "unverified app" warning for your own script: Advanced → Go to project).
   This creates four sheets:
   - **Volunteers** (Full Name | Email ID | PMI ID) – your volunteer list, from row 2. If you used the earlier version, your old **Members** sheet is renamed to Volunteers automatically.
   - **Delegates** (Full Name | Email ID | PMI ID) – your delegate list, same columns.
   - **AuditLog** – one row per event (lookups, certificates, downloads, emails, shares) with a Certificate Type column.
   - **Dashboard** – KPIs for volunteers and delegates, adoption by type, daily trend, activity and share charts.
4. **Deploy → New deployment → type: Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Click Deploy and copy the **Web app URL** (ends with `/exec`).
5. Test: open `YOUR_URL?action=lookup&q=someone@example.com` in a browser. You should see `{"found":true,...}` for an email that exists in Members, and a new row appears in AuditLog.

When you change `Code.gs` later, use **Deploy → Manage deployments → Edit → New version**, otherwise the live URL keeps the old code.

## Part B – Connect the web app

1. After every change to `Code.gs`, redeploy a **new version** (Deploy → Manage deployments → Edit → New version). Then open `index.html` in a text editor and set the URL near the top of the script:
   `var APPS_SCRIPT_URL = "https://script.google.com/macros/s/XXXX/exec";`
2. Leave it empty to run in **demo mode** (test email `demo@example.com`, PMI ID `1234567`, nothing is logged).
3. Edit the `CFG` block just below it to change the title, message, signatory names or share text.

## Part C – Publish the page

Pick one:
- **pmichennai.org / DigitalOcean**: upload `index.html` to a folder such as `/certificate/` and share that link.
- **Netlify Drop** (app.netlify.com/drop) or **GitHub Pages**: drag and drop the file; you get a public link in a minute.
- **Google Drive** alone cannot serve the page; use one of the options above.

## How it works for members

1. Member enters their **email ID or PMI ID**. The script checks both the **Volunteers** and **Delegates** sheets (email is case-insensitive; PMI ID is compared by digits only).
2. **Found in both** → the member is asked which certificate they want (Volunteer or Delegate) and can create the other one afterwards with one click. **Found in one** → that certificate flow starts directly. **Not found** → a friendly message.
3. Name is pre-filled from the sheet. The member confirms or edits **Full name**, **Email ID**, optional **Designation** and **Phone**, and adds a **photo** (required for volunteers, optional for delegates) with zoom and move controls, then ticks the privacy confirmation.
4. The certificate is drawn on screen. Member can **Download**, **Email me**, **Share** (native sheet on mobile, includes WhatsApp, Instagram and LinkedIn), or use the WhatsApp / LinkedIn / Instagram buttons.

**Email me** sends the certificate only to the email address stored in your sheet for that person, never to an address typed on the page, and shows it masked (for example `de***@gmail.com`). Each verified person can send up to 3 emails. If a person has no email in your sheet, the button is hidden. Gmail accounts can send about 100 emails a day through Apps Script, and Google Workspace accounts about 1,500, so for a very large event use a Workspace account or ask delegates to download instead. You can change the subject and sender name in the `MAIL` block of `Code.gs`.

## Certificate templates

- **Volunteer:** Certificate of Appreciation (your original wording), photo required.
- **Delegate:** Certificate of Participation with a cyan "DELEGATE" tag, the wording "has participated as a Delegate in Sangamam 2026 – The New Art of Delivery, organised by PMI Chennai Chapter on 14 November 2026 at Hilton Chennai", a highlighted line "Eligible to claim 7 PDUs (Professional Development Units)", the same signatories, and an optional photo. Change the PDU wording in the `highlight` setting of the delegate type in `index.html`.
Wording, positions and share text for each type are in the `TYPES` block near the top of the script in `index.html`.

## What is logged (AuditLog columns)

Timestamp, Date, Event, Input Type, Input (masked), Matched, Member Name (sheet), Full Name (entered), Designation, Email, PMI ID, Phone, Device, Session ID, Certificate Type.

Events: `LOOKUP_OK`, `LOOKUP_FAIL`, `CERT_GENERATED`, `DOWNLOAD`, `SHARE_NATIVE`, `SHARE_WHATSAPP`, `SHARE_LINKEDIN`, `SHARE_INSTAGRAM`, `EMAIL_CERT`.

## Dashboard metrics

Registered volunteers and delegates · verification attempts and success rate · certificates generated (total, volunteer, delegate) · unique people certified · downloads · shares · certificates emailed · mobile share. Tables and charts: certificates per day by type, activity by event, shares by channel, adoption by type (registered vs certified), top designations, recent failed lookups. Open **Certificate App → Rebuild dashboard** if you ever change or delete it.

## Security features

Built into the app
- **Your lists are never exposed.** The web app answers one lookup at a time and returns only the person's name and a masked email (for the Email me button), never full emails, PMI IDs or phone numbers.
- **Emails go only to your sheet's address**, with a per-person limit, a JPEG size/type check and a daily quota check.
- **Rate limits.** Per visitor session: 20 lookups per 10 minutes, and a 5-minute lock after 6 failed lookups. Globally: 400 lookups and 80 failed lookups per minute, which stops ID-guessing scripts. Tune these in the `LIMITS` block at the top of `Code.gs`.
- **No forged log rows.** Every audit event needs a one-time token issued only after a successful lookup (valid 6 hours, max 40 events). Calls without a valid token are ignored.
- **Formula-injection protection.** Everything written to AuditLog is length-limited, stripped of control characters and neutralised if it starts with `=`, `+`, `-` or `@`, so nobody can plant spreadsheet formulas in your sheet.
- **Input validation** on both the page and the server (valid email, numeric PMI ID, phone characters, maximum lengths) and a hidden honeypot field that catches simple bots.
- **Browser hardening.** The page has a strict Content Security Policy (it can only talk to Google Apps Script), sends no referrer, and is set to `noindex`.
- **Photos never leave the member's device.**

Protect your Google account and sheet (do these yourself)
1. Turn on **2-Step Verification** (or a passkey) for the Google account that owns the sheet.
2. Keep the spreadsheet **Restricted** (not "anyone with the link"). Only the Web app URL is public. Run **securityCheck** to verify.
3. Share the sheet only with named organizers, and only as **Viewer** unless they must edit Members. `setup()` already locks AuditLog and Dashboard so only you can edit them.
4. Run **installBackupTrigger** for a daily backup copy (kept 14 days) in a Drive folder named "Certificate App Backups".
5. Deploy the Web app **Execute as: Me**. Visitors never get access to your account or sheet, only to the two functions in `Code.gs`.
6. Review **Executions** in Apps Script (left menu) after the event, and delete the Web app deployment or set it to inactive once the event is over (Deploy → Manage deployments → Archive).
7. After the event, delete or archive the Members and AuditLog data if you no longer need it.

If you host on your own server (pmichennai.org / DigitalOcean), also send these headers: `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: no-referrer`, and use HTTPS only. GitHub Pages and Netlify already serve over HTTPS.

Honest limits: a public web page cannot hide its Web app URL, and Apps Script cannot see visitors' IP addresses, so rate limits work per session and globally rather than per person. Someone could still make a few guesses at member emails; the limits above make large-scale guessing impractical, and the app reveals only a name.

## Owner and disclaimer sections

The page shows **App owner: Arumugam Mariappan, VP Technology, PMI Chennai Chapter** at the top and bottom, a privacy note (modelled on the Sangamam26 badge app), a consent checkbox before a certificate is created, and a disclaimer in the footer. Edit the wording directly in `index.html` if the chapter wants changes.

## Notes

- **Privacy:** the web app URL is public, so share the link only with attendees. The AuditLog contains emails and phone numbers, so keep that sheet's sharing restricted.
- **Photos** stay in the member's browser and are never uploaded.
- **Fonts:** the certificate uses Inter and Cormorant Garamond as close web equivalents of Aptos and Teodor from your Canva design. Everything else (background, skyline, logo, badge, signatures) is the original artwork from your PDF.
- Apps Script handles typical event traffic comfortably. If you expect hundreds of simultaneous users, the 2-minute member cache in `Code.gs` already reduces sheet reads.
