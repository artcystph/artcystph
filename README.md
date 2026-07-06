# artcyst.ph — Custom Leather Folio Storefront

A made-to-order e-commerce site for **artcyst.ph**, a handcrafted leather folio and journal brand based in Ilocos Norte, Philippines. The site lets customers fully customize a folio (size, leather color, cord, charms, engraving, add-ons), submit an order, upload proof of payment, and receive an automated PDF invoice — all backed by Google Apps Script and Google Sheets instead of a traditional server/database.

## Contents

| File | Description |
|---|---|
| `index.html` | The main storefront — product showcase, live 3D/visual customizer, pricing calculator, checkout flow, and waitlist signup. Single-page app (HTML/CSS/JS, Tailwind + Swiper.js + Three.js). |
| `Code.gs` | Google Apps Script backend deployed as a Web App. Handles order intake, stock management, pricing validation, PDF invoice generation, email notifications, and Google Sheets record-keeping. |
| `paymentportal.html` | A standalone payment portal page where customers upload proof of payment for an existing order. |

## How it works

1. **Frontend (`index.html`)** — Customers configure their folio and see live pricing. On checkout, the page sends the order as JSON to the Apps Script Web App endpoint (`doPost`).
2. **Backend (`Code.gs`)** — Deployed as a Google Apps Script Web App (Execute as: **Me**, Access: **Anyone**). It:
   - Validates and recalculates pricing server-side (never trusts client totals)
   - Checks and decrements stock levels
   - Prevents duplicate order submissions
   - Writes a 17-column order record to a Google Sheet
   - Generates a PDF invoice and emails it to the customer
   - Sends an owner notification email
   - Saves proof-of-payment uploads to Google Drive
   - Manages a waitlist and batch capacity when stock runs out
3. **Payment Portal (`paymentportal.html`)** — A lighter page customers use after checkout to upload their proof-of-payment image/file for a specific order ID, which is sent to the same Apps Script backend.

## Setup / Deployment

### 1. Backend (Google Apps Script)
1. Create a new Google Sheet (or use an existing one) to store orders.
2. Open **Extensions → Apps Script** in that Sheet and paste in `Code.gs`.
3. Update the `CONFIG` object at the top of `Code.gs` with your own:
   - `SPREADSHEET_ID`
   - `NOTIFY_EMAIL`
   - `INVOICE_FOLDER_ID` (Google Drive folder for generated invoices)
   - `POP_FOLDER_ID` (Google Drive folder for proof-of-payment uploads)
4. Run `setupSheetHeaders()` once to initialize the sheet structure.
5. Deploy as a **Web App**:
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Copy the deployment URL (it will look like `https://script.google.com/macros/s/XXXXX/exec`).

### 2. Frontend
1. In `index.html` and `paymentportal.html`, replace the `GAS_URL` constant with your own Web App deployment URL.
2. Host the HTML files on any static hosting provider (GitHub Pages, Netlify, Vercel, etc.).
3. Make sure the `Content-Security-Policy` `connect-src` directive in each HTML file includes your Apps Script domains (already set to `script.google.com` / `script.googleusercontent.com`).

### 3. Optional triggers
Run these once from the Apps Script editor to enable scheduled/automated behavior:
- `setupExpiryTrigger()` — auto-expires pending, unpaid orders
- `setupOnEditTrigger()` — restores stock when an order is manually cancelled in the Sheet

## Notes

- Pricing logic (base prices, shipping by region, add-on costs, engraving surcharge, etc.) is duplicated between the frontend (for live display) and backend (`SERVER_PRICES` in `Code.gs`, for validation) — keep both in sync if prices change.
- This project has no traditional backend server or database; Google Sheets acts as the datastore and Google Apps Script as the API layer.
- Replace placeholder brand assets (logos, OG images, favicon paths) before going live.

## Tech Stack

- HTML / CSS / vanilla JavaScript
- [Tailwind CSS](https://tailwindcss.com/) (via CDN)
- [Swiper.js](https://swiperjs.com/) for carousels
- [Three.js](https://threejs.org/) for 3D product visualization
- Google Apps Script (backend/API)
- Google Sheets (database)
- Google Drive (file storage for invoices & proof of payment)
