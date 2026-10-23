# Filter Issue — PLP Filters Not Working (`plp.html`)

**Page:** `plp.html` (Shop All Silver — Product Listing)
**Status:** Fixed
**Date:** 22 Oct 2026

---

## 1. Issue

The filter sidebar on the PLP looked complete (Category, Collection, Price Range, Material, Craft Technique), but **filtering did nothing**: ticking any checkbox, moving the price slider, or clicking **Clear All** never changed the product grid, and the "Showing 12 of 186 results" count never updated.

## 2. Root Cause

The filters were purely decorative — no part of the filtering chain existed:

| Missing piece | Detail |
|---|---|
| No filter values | Checkboxes had no `value` attribute and groups had no identifier, so JS could not tell what was selected |
| No product data | Product cards had no category/collection/price/material/craft data to filter against |
| No filtering logic | `js/main.js` only opened/closed the mobile filter drawer; no change handlers existed |
| Dead controls | **Clear All** had no hook; the price slider's label was static text |
| Trap in defaults | *Necklaces* and *Alokha* were pre-checked — with real filtering switched on as-is, 10 of 12 products would have been hidden on page load |

## 3. Fix Applied

### Files changed (3)

| File | Change |
|---|---|
| `plp.html` | Each group tagged `data-filter-group` (`category`, `collection`, `price`, `material`, `craft`); every checkbox given a slug `value` (e.g. `necklaces`, `gold-plated`, `filigree`); pre-checked defaults removed; price slider tagged `data-price-range` (default reset to max ₹25,000) with `data-price-output` label; all 12 product cards tagged (see §4); **Clear All** buttons tagged `data-clear-filters`; empty-state message added (`data-no-results`) |
| `js/main.js` | New filtering block: reads checked values per group, applies them on `change`/`input`, updates result count and price label, handles **Clear All** (uncheck all + reset price), initialises on load |
| `css/styles.css` | `.no-results` styling only (spans the grid, centred, with a Clear All button). The sidebar scroll fix was not touched — see `filter-scroll-issue.md` |

### Filtering rules

- **Within a group: OR** — e.g. ticking Rings + Earrings shows both.
- **Across groups: AND** — e.g. Necklaces + Sterling Silver shows only sterling necklaces.
- **No checkbox ticked in a group = that group does not restrict results.**
- **Price slider = maximum price**; its label updates live (`₹500 — ₹X`).

```js
const matches =
  (selected.category.length === 0 || selected.category.includes(card.dataset.category)) &&
  (selected.collection.length === 0 || selected.collection.includes(card.dataset.collection)) &&
  (selected.material.length === 0 || selected.material.includes(card.dataset.material)) &&
  (selected.craft.length === 0 || selected.craft.includes(card.dataset.craft)) &&
  Number(card.dataset.price) <= maxPrice;
```

## 4. Product data assigned

Collections were taken from the PDP/title where known (Twist Ring = Alokha, Charm Bracelet = Eka, Men's Pendant = Rith); the rest were assigned sensibly so every filter value is testable.

| Product | Price | Category | Collection | Material | Craft |
|---|---|---|---|---|---|
| Sterling Silver Cuban Chain | 4,275 | necklaces | aarya | sterling | thappa |
| Silver Bar Pendant Necklace | 3,600 | necklaces | alokha | gold-plated | kundan |
| Hammered Cuff Bangle | 5,120 | bangles | aakaara | sterling | thappa |
| Rope Twist Bracelet | 2,890 | bracelets | vibra | sterling | thappa |
| Twist Textured Band Ring | 3,040 | rings | alokha | gold-plated | thappa |
| Filigree Pendant | 4,750 | necklaces | kripa | sterling | filigree |
| Hoop Earrings Silver | 3,980 | earrings | aarya | sterling | thappa |
| Layered Chain Set | 6,200 | necklaces | kraya | gold-plated | kundan |
| Minimal Stud Earrings | 2,450 | earrings | vibra | sterling | kundan |
| Oxidized Statement Ring | 5,640 | rings | aakaara | oxidized | meenakari |
| Charm Bracelet Eka | 3,150 | bracelets | eka | sterling | kundan |
| Men's Pendant Rith | 7,100 | necklaces | rith | oxidized | meenakari |

Note: *Anklets* and *Nose Pins* have no products yet — selecting them correctly shows the "No products match the selected filters" state.

## 5. Verification (headless Chrome)

| Test | Result |
|---|---|
| Initial load | 12 shown, "Showing 12 of 12 results" |
| Category = Rings | 2 (Twist Ring, Oxidized Statement Ring) |
| Category = Necklaces | 5 |
| Necklaces + Material = Sterling | 2 |
| Price ≤ ₹3,000 | 2, label updates to `₹500 — ₹3,000` |
| Clear All | Back to 12 |
| Category = Anklets (no products) | 0 shown, empty-state message visible |

`node --check js/main.js` passes.

## 6. How to test

1. Hard-refresh `plp.html` (`Ctrl + Shift + R`).
2. Tick **Rings** → 2 products; add **Earrings** → 4.
3. Add **Gold-Plated Silver** → only matching rings/earrings remain.
4. Drag the price slider down → grid and label update.
5. Click **Clear All** → all 12 return.
6. Tick **Anklets** → empty-state message appears.
