# UAT vs Local HTML — Comparison Report

**UAT:** https://uat-ecom.kalyanjewellers.net/
**Local:** `index.html` (LP) · `plp.html` (PLP) · `pdp.html` (PDP) in this folder
**Google Doc:** https://docs.google.com/document/d/1R4FgYnnzuUX9Si8Hio8we_2Q6jVrQTy3wEwozZ29vQ0/edit?tab=t.i2uygtlnfop1
**Date:** 22 Oct 2026

---

## ⚠️ Access note — read first

The Google Doc **could not be opened** from here: it returns **403 Forbidden** (it is not shared as "Anyone with the link can view"). Therefore the issue list *inside that document* could not be cross-checked line-by-line.

This report therefore compares:

1. The **live UAT site** (fetched directly: home, `/collections/all`, and a product page), against
2. The **local HTML**, plus the issues fixed in this working session (see `thumbnail-image-ui-issue.md`, `filter-issue.md`, `filter-scroll-issue.md`).

**Action needed:** share the Google Doc with "Anyone with the link → Viewer" (or paste its contents here), and the "Pending" table in §3 can be re-mapped 1:1 against the document's own issue numbers.

---

## 1. Summary

| | Count |
|---|---|
| Issues fixed in local HTML (§2) | **7** |
| Gaps vs UAT — pending (§3) | **13** (grouped P1/P2/P3) |
| Items to confirm interactively on UAT (§4) | **6** |

Overall: the local site is a **look-and-feel specimen** (per `README.md`: demo/agreement surface, placeholders allowed, not a live store). The gallery and PLP filtering issues raised during the walkthrough are fixed; everything that needs **real catalogue data, accounts, cart, search, or store systems** is still pending, because the local pages are static.

---

## 2. ✅ Fixed in local HTML

All of these were broken in the local HTML and are now fixed and verified (headless Chrome / measurement).

| # | Issue | Page | Fixed by | Evidence |
|---|---|---|---|---|
| F1 | Thumbnails were rectangles, not square like UAT | PDP | 96×96px square thumbnails, active border `#cf254a` | `thumbnail-image-ui-issue.md` |
| F2 | Gallery did not match UAT reference UI (vertical thumbs + up/down buttons, cream stage, contain-fit, sliding main image, hover arrows, mobile dots) | PDP | Gallery rebuilt to the UAT reference pattern | `thumbnail-image-ui-issue.md` §3 |
| F3 | Main image dropped below thumbnails after an edit (grid lost) | PDP | Base CSS restored; desktop overrides kept inside `@media (min-width: 900px)` only | `thumbnail-image-ui-issue.md` §2–3 |
| F4 | No visible gap between thumbnails | PDP | `gap: 1rem`, strip background transparent, per-card white background + shadow | `thumbnail-image-ui-issue.md` §3 |
| F5 | Selecting the 4th thumbnail left it half-clipped; thumbnail strip did not scroll | PDP | Strip capped at 320px (3 visible), `overflow-y: auto`, hidden scrollbar; JS `scrollBy()` reveals the selected thumbnail without moving the page | `thumbnail-image-ui-issue.md` §3 |
| F6 | Filters did nothing (no values, no product data, no logic; Clear All dead; price slider static) | PLP | Full filtering: group/value tagging, 12 products tagged, OR-within-group / AND-across-groups, live price + result count, Clear All, empty state | `filter-issue.md` |
| F7 | Filter sidebar could not be scrolled to the bottom without scrolling the page; afterwards the last option (Meenakari) still sat off-screen | PLP | Internal scroll (`overflow-y: auto`, `overscroll-behavior: contain`), sticky Filters header, height corrected to `calc(100dvh - 13rem)` so the box bottom stays in the viewport | `filter-scroll-issue.md` |

---

## 3. ⏳ Pending — present/working on UAT, missing or static locally

### P1 — Catalogue & commerce (core store behaviour)

| # | Gap | UAT | Local HTML |
|---|---|---|---|
| P1.1 | Real catalogue | 429 items on `/collections/all`, real titles/prices | 12 hard-coded cards with dummy prices/titles |
| P1.2 | Unique product pages | Every product has its own URL/SKU, e.g. `/products/kumud-22k-gold-ring-kalyan-jewellers` (SKU PBREV0807) | All 12 cards link to the **same** `pdp.html`; the PDP always shows the ₹3,040 silver ring |
| P1.3 | Wishlist per product | "Add to wishlist" on every PLP card; wishlist heart on PDP | No per-card wishlist; header Wishlist links are dead (`href="#"` — 85 across the 3 pages) |
| P1.4 | Cart / buy flow | Live Shopify/Hydrogen store (sold-out state, Request Callback, Find in Store on the Kumud PDP) | No cart, quantity, or buy action anywhere; Request Callback form only shows a demo `alert()` |
| P1.5 | Search | Live store search in header | Header search input exists on all pages but does nothing |

### P2 — Page content & data accuracy

| # | Gap | UAT | Local HTML |
|---|---|---|---|
| P2.1 | PDP gallery images | All views belong to the same product | Gallery mixes different products (ring → necklace → pendant) as "views" of one ring |
| P2.2 | PDP specifications | Design number, Section, Type, Division, Gender, Metal Details, gold-rate note on the Kumud PDP | Generic silver spec tables; no design/section/division data |
| P2.3 | PDP price breakup | "View Product Details & Price Breakup" on UAT | Present, but only a 3-row MRP/Discount/Offer accordion — **partial** |
| P2.4 | Reviews | "Write a review" on UAT PDP (no review list on the fetched page) | Fabricated "43,123 Reviews", 4.8 score and 3 invented review cards — content does not match UAT; decide which behaviour is wanted |
| P2.5 | PLP filter taxonomy | Real store facets over 429 products | Local filters work, but only over 12 tagged dummy products; Anklets/Nose Pins return the empty state; local price range (₹500–₹25,000) vs UAT silver items seen at ₹15,450 |
| P2.6 | Result count | "All (429 items)" | Static HTML says "Showing 12 of 186 results" — JS corrects it to "of 12" on load, but the source number 186 is stale |
| P2.7 | Hero/landing content | Full Kalyan storefront (gold/diamond campaigns) | Hero still says **"Banner Comes Here"** (desktop + mobile) — placeholder creative |
| P2.8 | Main navigation | UAT nav includes a **CHANDRA** entry (per reference screenshots) | Local nav (Jewelry…More) has no Chandra link |

### P3 — Secondary actions (dead links locally)

| # | Gap | Local state |
|---|---|---|
| P3.1 | PDP "Live Video Call" / "Store Availability" buttons | Dead (`href="#"`) |
| P3.2 | Store / Profile / Wishlist header utilities | Dead on all 3 pages |
| P3.3 | Footer links, Book Appointment / Get Direction (Keep-in-Touch banner) | Dead (`href="#"`) |
| P3.4 | Sold-out / availability states | Local PDP always shows the same purchasable state; no sold-out handling like UAT's Kumud PDP |

**Partially matching (not counted above):** breadcrumb, Request Callback + Find in Store buttons, trust badges (BIS, returns, delivery), "You May Also Like", product-details accordions, and the mobile filter drawer all exist locally and visually match UAT — they are only missing real data/actions behind them.

---

## 4. ❓ To confirm interactively on UAT (could not be verified by fetching)

UAT is a JavaScript (Hydrogen) storefront; fetching returns limited content, and this machine has no direct browser access to it. These need a human pass on UAT before they can be marked fixed/pending:

1. PLP **Sort** options (price, featured, newest…) — local PLP has **no sort at all** (likely pending).
2. PLP pagination / load-more / infinite scroll behaviour.
3. Add-to-cart flow for an **in-stock** PDP (the fetched Kumud PDP is sold out).
4. PDP ring **size selector**, quantity, pin-code/delivery check, if present.
5. Exact UAT filter list on the silver collection, to align the local filter taxonomy (P2.5).
6. Header search suggestions/results behaviour.

---

## 5. Recommended order

1. **Share the Google Doc** (view access) → re-map §3 to the document's issue numbers; some §3 items may already be tracked there, and the document may list issues not covered here.
2. Decide scope: this specimen is **look-and-feel only** per `README.md` — P1 items (catalogue, cart, wishlist, search) normally belong to the live Hydrogen build, not this static HTML. If the expectation is only visual parity, prioritise instead: P2.1 (same-product gallery images), P2.7 (hero creative), P2.8 (CHANDRA nav), P2.6 (result count), then P3 dead links.
3. Re-check §4 on UAT in a browser and update this document.

---

## Appendix — method

- UAT fetched 22 Oct 2026: `/` (limited — SPA shell), `/collections/all` (429 items, per-card wishlist), `/products/kumud-22k-gold-ring-kalyan-jewellers` (SKU, ₹54,846.88, sold out, Product Summary, spec tables).
- Google Doc: `403 Forbidden` on both the edit and export URLs (not link-shared).
- Local audit: `index.html`, `plp.html`, `pdp.html`, `css/styles.css`, `js/main.js`; fixed items cross-checked against `docs/thumbnail-image-ui-issue.md`, `docs/filter-issue.md`, `docs/filter-scroll-issue.md` and re-verified after the fixes.
- Screenshots of UAT PDP/PLP supplied earlier in this session (Kalyan header with CHANDRA nav, Kumud PDP, thumbnail reference markup) were used for the PDP gallery comparison.
