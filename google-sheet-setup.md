# Connect the contact form to a live Google Sheet

Takes about 5 minutes. No server, no paid service.

## 1. Create the Sheet

1. Go to <https://sheets.new> and name it e.g. **ExioCraftz Submissions**.
2. In row 1, add these headers, one per column (A → K), spelled exactly:

```
Timestamp | Name | Email | Phone | Company | What They Want | Service | Budget | Details | Source
```

## 2. Add the script

In the Sheet: **Extensions → Apps Script**. Delete whatever is there and paste this:

```javascript
function doPost(e) {
  var lock = LockService.getScriptLock();
  lock.waitLock(20000);
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    var d = JSON.parse(e.postData.contents);
    sheet.appendRow([
      d.ts || new Date().toISOString(),
      d.name || '',
      d.email || '',
      d.phone || '',
      d.company || '',
      d.what || '',
      d.service || '',
      d.budget || '',
      d.details || '',
      d.source || 'Website'
    ]);
    return ContentService.createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}
```

Save (disk icon).

## 3. Publish it

1. **Deploy → New deployment**.
2. Gear icon next to "Select type" → **Web app**.
3. Execute as: **Me**.
4. Who has access: **Anyone** — this is required; without it the website cannot post. The URL is unguessable and the script only ever appends rows.
5. **Deploy**, then **Authorize access** and approve the Google warning screen (it appears because the script is yours and unverified).
6. Copy the **Web app URL**. It ends in `/exec`.

## 4. Connect the site

Open the site, go to the footer **Admin** link (or `#admin`, or Alt+Shift+A), paste the URL into the
**Google Sheet endpoint** field, and press **Connect**.

Submit a test message from the contact page — a new row should appear in the Sheet within a second or two.

## Notes

- Every submission is also kept in the browser as a backup, so the **Export to Excel** button still works.
- The **Sheet** column in the admin table shows `In Sheet`, `Local only`, or `Failed` per submission.
- **Retry Unsent** re-posts anything that didn't make it (e.g. submitted while offline).
- The URL can also be set permanently in the design's Tweaks panel under "Form submissions" — do that so it applies for every visitor, not just your browser.
- If you change the Apps Script later, use **Deploy → Manage deployments → Edit → New version**, otherwise the old code keeps running.
