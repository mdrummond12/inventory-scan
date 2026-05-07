# Inventory Scanner — Physical Count Tool

A single-file web app that turns a phone's camera into a barcode scanner for physical inventory counts. Scan an item, enter a quantity, repeat. Export the session as CSV or JSON, or POST each entry to an inventory API.

**File:** `inventory-scanner.html` (zero build step, no dependencies to install — everything loads from a CDN)

---

## What it does

1. **Scan** a barcode with the rear camera (QR, UPC, EAN, Code 128, Code 39, Data Matrix, PDF417, ITF, Codabar, Aztec — anything ZXing supports)
2. **Enter** the physical quantity counted (defaults to 1, with +/− buttons and direct keyboard entry)
3. **Add** the line to the session
4. **Export** the whole session as CSV or JSON when done — or wire up the API hook to push entries live

The scanner auto-debounces repeat reads of the same code (1.5s window) so a barcode lingering in frame does not spam the list.

---

## Running it

The browser will only grant camera access on `https://` or `localhost` — opening the file with `file://` will fail with a permission error.

**Easy options:**
- Drop the HTML file into [Netlify Drop](https://app.netlify.com/drop), Vercel, Cloudflare Pages, or GitHub Pages — instant HTTPS URL
- Serve locally with `python3 -m http.server 8000` plus an HTTPS tunnel like `ngrok http 8000`

Then open the resulting URL on your phone, tap **Start Scanner**, and grant camera permission.

---

## Features

- **Manual entry fallback** — tap the barcode field to type a SKU when a label is damaged or missing
- **Merge duplicates toggle** (on by default) — re-scanning the same item adds to its quantity instead of creating a new row. Turn off if you need separate line items per scan (e.g., counting by location).
- **Tap any session row** to reload that barcode for another count
- **Trash icon** removes a line
- **Auto-save** to `localStorage` — closing the tab does not lose work
- **Session ID** is stamped on every export so distinct counts stay separable
- **Battery-aware** — scanning pauses when the tab is backgrounded
- **Haptic feedback** on successful scan (mobile only)

---

## Data formats

### CSV export
Columns:
```
session_id,barcode,format,quantity,timestamp_iso
S1A2B3C,012345678905,EAN_13,12,2026-05-07T14:32:11.000Z
```
Imports cleanly into Excel, Google Sheets, and most ERP/inventory systems.

### JSON export
```json
{
  "sessionId": "S1A2B3C",
  "exportedAt": "2026-05-07T14:35:00.000Z",
  "counts": [
    {
      "barcode": "012345678905",
      "format": "EAN_13",
      "quantity": 12,
      "timestamp": "2026-05-07T14:32:11.000Z"
    }
  ]
}
```

---

## API integration hook

The HTML contains a marked block where each `Add to Count` action can also POST to an inventory API in real time. Find this in the `<script>` section:

```javascript
/* OPTIONAL: POST to your inventory API here */
// submitToApi({ sessionId: SESSION_ID, barcode: ..., qty, ts: ... });
```

And the example function below it:

```javascript
async function submitToApi(entry) {
  const res = await fetch('https://YOUR-INVENTORY-API/physical-counts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(entry)
  });
  if (!res.ok) throw new Error('HTTP ' + res.status);
}
```

Uncomment the call site, fill in your endpoint URL, add any auth headers your API requires, and entries will push live as the user adds them. The local session list still saves to `localStorage` so a network blip does not lose data.

**Common integration targets:**
- Google Sheets — use a Google Apps Script Web App as the endpoint, or the Sheets API
- Airtable — POST to `https://api.airtable.com/v0/{baseId}/{tableName}` with a Bearer token
- Custom REST API — straight `fetch` POST as shown
- Generic webhook (Zapier, Make, n8n) — paste the webhook URL in place of `YOUR-INVENTORY-API`

---

## Tech stack

- Plain HTML/CSS/JS — no framework, no build step
- [@zxing/browser](https://github.com/zxing-js/browser) v0.1.5 (loaded from unpkg CDN) for barcode decoding
- Google Fonts: Fraunces (display) + JetBrains Mono (body)
- `MediaDevices.getUserMedia` for camera access (rear camera preferred)
- `localStorage` for session persistence

---

## Browser support

- iOS Safari 14.5+
- Android Chrome 88+
- Desktop Chrome / Edge / Firefox / Safari (modern versions) — useful for testing, less useful for actual counting

Camera access requires HTTPS in all cases.

---

## Customization ideas

Things to ask a future Claude chat in this project to add:

- A **location/bin field** alongside quantity (for warehouses with multiple bin locations)
- A **lookup step** that fetches product name/expected count from your inventory API after a scan, then displays a variance vs. counted qty
- **Multi-user sessions** with a server-side store instead of `localStorage`
- A **torch/flash toggle** for dark stockrooms
- **Sound feedback** in addition to haptic
- Switching the export format to match a specific ERP's import template (NetSuite, QuickBooks, Cin7, Fishbowl, etc.)
"# inventory-scan" 
