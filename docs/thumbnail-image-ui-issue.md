# Thumbnail Image UI Issue — PDP (`pdp.html`)

**Page:** `pdp.html` (Twist Textured High Shine Band Ring 18 Karat)
**Status:** Fixed
**Date:** 22 Oct 2026

---

## 1. Issue

On the PDP, the product gallery did not match the reference (Kalyan Jewellers) UI:

1. The 4 thumbnails were **rectangles**, not squares — on desktop they stretched to fill the height of the main image.
2. After a first fix, the **main image dropped below the thumbnails** (the fix had been pasted over the base CSS, so `.pdp-gallery` lost `display: grid` and `.gallery-main` lost its `aspect-ratio`).
3. Thumbnails appeared stuck together with **no visible gap** (a white `background` on the thumbnail strip hid the gap).
4. When the **4th thumbnail was selected, it was half-clipped** at the bottom of the thumbnail column and the strip did not scroll to reveal it.

## 2. Root Cause

| Symptom | Cause |
|---|---|
| Rectangular thumbnails | Desktop CSS used `.gallery-thumbs button { flex: 1; aspect-ratio: auto; }` — buttons stretched to share the main image's height |
| Main image below thumbnails | Square-thumbnail CSS pasted into the base section instead of the `@media (min-width: 900px)` block, destroying `display: grid` / main-image sizing |
| No gap between thumbnails | `.gallery-thumbs { background: #fff; }` — the white strip filled the gap, making 4 images look like one block |
| Last thumbnail clipped, no scroll | Strip was `overflow: hidden` (can never scroll), its content was 520px tall (36 + 8 + 4×96 + 3×16 + 8 + 36) against `max-height: 500px`, and no JS scrolled the strip to the selected thumbnail |

## 3. Fix Applied

Gallery rebuilt to match the reference UI: vertical square thumbnails on the left with up/down scroll buttons, and a sliding main image (`object-fit: contain` on `#fdfaf5`) on the right.

### Files changed (3)

| File | Change |
|---|---|
| `pdp.html` | New gallery markup: `.gallery-thumbs-wrap` (up/down `.gallery-scroll-btn` + `.gallery-thumbs` with `data-index`), `.gallery-main` containing hover `.gallery-arrow` buttons, `.gallery-viewport` → `.gallery-track[data-gallery-track]` → 4 × `.gallery-slide`, and `.gallery-dots` (mobile) |
| `css/styles.css` | Base: `.gallery-main` cream (`#fdfaf5`) square stage, slider track (`display: flex`, `transition: transform .5s`), slides `object-fit: contain`, dots (active `#cf254a`); thumbnails hidden on mobile. Desktop (`@media (min-width: 900px)`): thumbnails `96px × 96px` squares, white cards, active border `#cf254a`, `1rem` gap with transparent strip background, hover arrows (15% width, full height), dots hidden |
| `js/main.js` | Slider logic replaces the old single-image swap: `goToGallery(index)` translates the track by `-index × 100%`, syncs active thumbnail + dot, disables up/down buttons at the ends, and smooth-scrolls the thumbnail strip |

### Follow-up fixes within this issue

- **Gap:** `.gallery-thumbs` gap `1rem`, `background: transparent`, buttons given their own white background + shadow so the page colour shows between cards.
- **Thumbnail auto-scroll:** strip capped at `max-height: 320px` (3 thumbnails visible, like the reference), `overflow-y: auto`, hidden scrollbar; `goToGallery()` uses `scrollBy()` on the strip only (not `scrollIntoView()`, which would move the page) to reveal the selected thumbnail fully:

```js
const thumbsBox = document.querySelector(".gallery-thumbs");
const activeThumb = thumbs[galleryIndex];
if (thumbsBox && activeThumb) {
  const boxRect = thumbsBox.getBoundingClientRect();
  const thumbRect = activeThumb.getBoundingClientRect();
  if (thumbRect.top < boxRect.top) {
    thumbsBox.scrollBy({ top: thumbRect.top - boxRect.top - 2, behavior: "smooth" });
  } else if (thumbRect.bottom > boxRect.bottom) {
    thumbsBox.scrollBy({ top: thumbRect.bottom - boxRect.bottom + 2, behavior: "smooth" });
  }
}
```

## 4. Verification

- `node --check js/main.js` passes.
- All 4 product images exist (`ring.jpg`, `p5.jpg`, `p3.jpg`, `p1.jpg`, all 1100×1100 — shown uncropped with `object-fit: contain`).
- No stale `data-gallery-main` references remain.
- Manual/headless check: clicking thumbnail 4 slides the main image, the strip scrolls up so thumbnail 4 (with its red border) is fully visible; returning to thumbnail 1 scrolls it back. Hover arrows appear on desktop; dots show on mobile.

## 5. How to test

1. Hard-refresh `pdp.html` (`Ctrl + Shift + R`).
2. Click each thumbnail — main image slides, red border follows.
3. Click thumbnail 4 — strip scrolls up; thumbnail 4 fully visible.
4. Hover the main image — prev/next arrows appear.
5. Resize below 900px — thumbnails hide, dots appear under the image.
