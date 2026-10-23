# MKS Fashion — frontend prototype

Clickable, responsive frontend prototype for **mksfashion.com**: custom T-shirt & DTF printing, print-only
orders (DTF transfers / print stickers by the piece, printing by the linear metre), banners and award shields —
plus a separate **admin panel** prototype.

> **Prototype only.** No backend, database, real authentication, payments, emails or file uploads.
> All prices, roll widths, limits and operational details are **placeholders** the MKS team can edit in the admin.
> Demo data lives in the browser's `localStorage` and can be reset at any time.

---

## 1. Open / run it

No build step and no install. Any of these works:

| Option | How |
|---|---|
| **Laragon** (recommended) | The folder is already in `C:\laragon\www\msk_fashion`. Start Laragon and open `http://msk_fashion.test/` (auto virtual host) or `http://localhost/msk_fashion/`. |
| **Double-click** | Open `index.html` directly in Chrome / Edge / Firefox (`file://` works). |
| **Any static server** | `python -m http.server 8080` in this folder → `http://localhost:8080/` |

- **Admin:** `/admin/` → you'll see the prototype sign-in screen; any email + password opens it (not secure — by design).
- **Customer account:** `/account.html` is signed in as the demo customer *Sana Malik*.
- **Reset demo data:** footer → *Reset demo data*, or Admin → Settings → *Reset demo data*.
- If something looks stale after an update, hard-refresh (**Ctrl+F5**).

Fonts load from Google Fonts (system fonts are used offline). Everything else is local.

---

## 2. What's included

### Customer website
| Page | File |
|---|---|
| Home (hero slider, services, ordering paths incl. *prints only*, featured, how it works, quality, bulk, sample testimonials, FAQ) | `index.html` |
| Shop / catalogue (search, sort, filters, chips, load more) | `shop.html` |
| Category page | `category.html?slug=…` |
| Product detail (gallery, variants, sizes, bulk table, specs) | `product.html?slug=…` |
| **Custom-order builder — path chooser + 3 paths** | `customize.html` |
| ↳ Printed garments & products | `customize.html?path=garment` |
| ↳ Prints only — DTF transfers / print stickers by quantity | `customize.html?path=transfers` |
| ↳ Printing by the linear metre (gang sheet or individual designs → layout review) | `customize.html?path=meter` |
| Bulk order / quote request | `bulk-order.html` |
| Cart (mixed carts, quote / layout-review group, coupons) | `cart.html` |
| Checkout prototype (contact → shipping → delivery → review; payment UI is visual only) | `checkout.html` |
| Customer account (orders, order detail, proofs, quotes, accept quote → cart, saved designs, profile) | `account.html` |
| About, Services, FAQ, Contact, policy templates, 404 | `about.html`, `services.html`, `faq.html`, `contact.html`, `policy.html?doc=…`, `404.html` |

### Admin panel (`/admin/`)
Dashboard · Orders · Order detail · **Production jobs** · Quote requests · Quote detail (draft quote, layout review,
send, approve/reject, convert) · Artwork & proofs · Customers · Products · Product editor · Categories · Inventory ·
Pricing rules · **Print services** · Discounts · Homepage content · Settings & staff roles · Sign-in (visual only).

### Print-only ordering (no garment)
- **By quantity (pieces):** upload artwork per design, design name + optional reference, print width × height
  (unit shown — cm by default), quantity per design, material (DTF heat transfer, or self-adhesive UV DTF print
  sticker *only because that material is configured as adhesive*), size-band unit price, per-design quantity
  discount, subtotal; several designs per order, each becoming its own cart line.
- **By linear metre:** material, roll width (configured in admin and shown in cm + inches), length in **linear
  metres** with configurable minimum / increment / decimals, price per linear metre, length tiers, prepared gang
  sheet *or* individual designs (size + copies), optional arrangement service with configurable charge.
  The builder explains that a linear metre is 1 m of roll **length** at the chosen width — not a square metre.
- We **never** infer fit or length from an uploaded image. Individual designs (and gang sheets that ask for
  arrangement) are flagged **“Layout review required”** and go through a quote: staff confirm the length, send a
  quote, the customer accepts it in their account and adds the *quoted job* to the cart to check out.
- Previews are labelled **“Illustrative — not a production file”**.

### Three independent statuses
Every order has **production status** (order review → confirmed → in production → ready → dispatched → delivered),
**payment status** and **artwork approval** (per file, summarised per item/order). None changes another.
Print-only lines never reduce garment stock; production summaries keep **garments**, **transfer pieces** and
**printing length** apart.

---

## 3. Folder structure

```
msk_fashion/
├── index.html … 404.html          Customer pages (one file per screen)
├── admin/                         Admin pages (thin HTML shells + page scripts)
├── assets/
│   ├── css/app.css                Design tokens + all storefront components
│   ├── css/admin.css              Admin layout/components (extends app.css)
│   ├── img/brand/                 Official logo exports (light/dark), favicon
│   ├── img/photos/                Studio photos cropped from MKS social posts
│   └── js/
│       ├── config.js              Namespace, root URL, data version
│       ├── data/                  MOCK DATA — catalog.js, pricing.js, statuses.js, sample-records.js
│       ├── core/                  BUSINESS RULES (no DOM) — pricing-engine.js, print-pricing.js, validation.js, format.js
│       ├── services/              DATA ACCESS — store.js (localStorage), api.js (mock API = future Laravel endpoints)
│       ├── ui/                    Reusable UI — icons, mockups, components, layout, catalog-view, print-builder
│       ├── pages/                 One script per customer page
│       └── admin/                 Admin layout + one script per admin page
├── resources/brand-source/        Full-resolution original logo files (not loaded by pages)
├── docs/DEMO-TESTS.md             Demo interactions to try
└── docs/LARAVEL-MAPPING.md        Screens & data → Laravel routes, controllers, tables, APIs
```

### Layering (Laravel-ready)
1. **`data/`** — mock data only (replace with DB seeders).
2. **`core/`** — pure pricing/validation rules. `pricing-engine.js` + `print-pricing.js` are written so Laravel can
   port them 1:1 (`PricingService`, `PrintPricingService`) and **recalculate prices on the server**.
3. **`services/api.js`** — the only data access used by pages. Each method is commented with the REST route it
   stands in for (e.g. `POST /api/cart/items`). Swapping to Laravel = replace method bodies with `fetch()`.
   The mock API already behaves "server-side": it re-prices cart lines from product + config, never trusts page prices.
4. **`ui/` + `pages/` + `admin/`** — DOM only.

### Tech choices
Plain HTML, CSS and vanilla JavaScript (classic scripts on a `window.MKS` namespace, so it also runs from
`file://`). **No Tailwind, jQuery or framework**: the component CSS maps directly onto Blade components, and
there's nothing to build. Product images are SVG mockups drawn in code, so colours and uploaded artwork preview live.

---

## 4. Demo interactions
See **[docs/DEMO-TESTS.md](docs/DEMO-TESTS.md)** — includes the five print-only journeys.

## 5. Next phase (Laravel)
See **[docs/LARAVEL-MAPPING.md](docs/LARAVEL-MAPPING.md)**.

## 6. Placeholders the MKS team must supply
Real prices, size bands, roll widths, minimums and increments; materials offered; delivery fees and timeframes;
payment methods; phone / WhatsApp / email / address / hours (Admin → Settings); policy texts; real testimonials;
product photography.
