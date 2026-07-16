# Pallet Tracker — LWH Bay History PWA

A zero-build, static PWA for looking up a pallet by **LWH_ID** or **Customer_ID** and
seeing its complete bay move history, pulled live from your published Google Sheets CSV.

## What it does

- Search by **LWH_ID** (exact) or **Customer_ID** (exact, or partial if nothing exact matches).
  If a Customer_ID matches more than one pallet, you get a pick list.
- Shows a summary card: Customer, Item, ItemDesc, Lot #, Qty, current bay, days in current bay,
  last moved by.
- Shows a full **bay history timeline** (every PreviousBay → NewBay move, oldest to newest,
  with date/time, mover, and qty), current bay flagged "Now."
- **Barcode/QR scanner** (camera, via ZXing) — scan a pallet tag and it auto-searches.
- **OCR scan** (camera, via Tesseract.js) — for tags that aren't a clean barcode/QR. Captures
  a photo, reads the text, and lets the person confirm/edit before searching (OCR isn't
  perfect, so it's a review step, not auto-submit).
- **Dark / light theme toggle** (top right), remembers the choice per device.
- Works offline for the app shell; the CSV always tries live network first so bay data is
  never stale, but falls back to last-loaded data if the connection drops.
- Recent lookups saved locally for one-tap re-search.
- "Copy summary" button — copies a plain-text summary + full move history to the clipboard,
  handy for pasting into an email or chat when helping someone troubleshoot.

## Files

```
index.html               — the whole app (HTML/CSS/JS, no build step)
manifest.json             — PWA install manifest
sw.js                      — service worker (offline app shell, always-live CSV)
icon-192.png / icon-512.png / icon-512-maskable.png
```

## Deploy to GitHub Pages

1. Create (or reuse) a GitHub repo, e.g. `pallet-tracker`.
2. Drop all files in this folder into the repo root (or a `/docs` folder if you prefer).
3. Repo Settings → Pages → set the source branch/folder → Save.
4. Give it a minute, then visit the published URL. On a phone, use "Add to Home Screen" /
   "Install app" to get the PWA icon and full-screen behavior.

## If you need to point it at a different sheet

Open `index.html`, find this line near the top of the `<script>` block:

```js
const CSV_URL = "https://docs.google.com/spreadsheets/d/e/.../pub?output=csv";
```

Replace with your published CSV link (File → Share → Publish to web → CSV, in Google Sheets).
The app expects these exact header names:

```
Location, LWH_ID, Customer_ID, Customer, Item, ItemDesc, LotNum, Units,
CurrentQty, MoveWarehouse, PreviousBay, NewBay, MoveQty, MovedDateTime, MovedBy
```

If your headers ever change, search `index.html` for the field names above (they're used
directly, e.g. `r.NewBay`, `r.MovedDateTime`) and update accordingly.

## Notes on the data

Your published sheet is already a **moves log** — every time a pallet gets scanned to a new
bay, a new row is appended with the same `LWH_ID` but a new `PreviousBay` → `NewBay` pair and
timestamp. The app just filters all rows for a given `LWH_ID` and sorts by `MovedDateTime`,
so no extra processing is needed on the Sheets side. The "current bay" shown is simply the
`NewBay` from the most recent row.

## Camera permissions

Both the barcode scanner and OCR scanner request camera access on first use. On shared
warehouse devices, the browser may ask every session unless the site is added to the home
screen as an installed PWA (installed apps tend to retain permission more reliably than a
browser tab).

## Browser support

Works in any modern mobile browser (Chrome/Edge/Safari). Camera features require HTTPS,
which GitHub Pages provides by default.
