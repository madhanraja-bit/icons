# Filter Scroll Issue — PLP Sidebar (`plp.html`)

**Page:** `plp.html` (Shop All Silver — Product Listing)
**Status:** Fixed (two stages)
**Date:** 22 Oct 2026

---

## 1. Issue

Two related scrolling problems with the filter sidebar:

1. **Stage 1:** All filter options could not be accessed. The sidebar (Category 7 + Collection 8 + Price + Material 3 + Craft 4) is taller than the viewport, and the only way to see the lower groups (Price, Material, Craft Technique) was to scroll the **entire page**.
2. **Stage 2 (after Stage 1 fix):** The sidebar scrolled internally and filtering worked, but the **last option (Craft Technique → Meenakari) was still not visible** — the sidebar's bottom was cut off below the screen, even when the filter was scrolled to its maximum.

## 2. Root Cause

### Stage 1

Desktop CSS (`@media (min-width: 900px)`) was:

```css
.filters {
  display: block;
  position: sticky;
  top: calc(var(--header-h) + 1rem);
}
```

`position: sticky` with **no `max-height` and no `overflow`** — a sticky element taller than the viewport simply extends past the bottom of the screen; its lower content can only be revealed by scrolling the page.

(Mobile was already fine: `.filters.is-open` already had `max-height` + `overflow-y: auto`.)

### Stage 2

The Stage 1 height cap only subtracted the header:

```css
max-height: calc(100dvh - var(--header-h) - 2rem);
```

But the sidebar's **natural top is ~190px** from the viewport top (sticky header 96px + breadcrumb + page padding), not immediately below the header. Measured with headless Chrome at 1280×800:

- Viewport height: `713px`
- Sidebar box bottom: `775px` → **62px of the box below the viewport**

So even at maximum internal scroll, the last option sat inside the box but **off-screen** — matching the user screenshot (Filigree / Thappa / Kundan visible, Meenakari cut).

## 3. Fix Applied

### File changed (1)

**`css/styles.css`** only — inside `@media (min-width: 900px)`, the `.filters` rule used by `plp.html`. No HTML or JS changes; the filtering logic (see `filter-issue.md`) is untouched.

### Stage 1 changes

```css
.filters {
  display: block;
  position: sticky;
  top: calc(var(--header-h) + 1rem);
  max-height: calc(100dvh - var(--header-h) - 2rem);
  overflow-y: auto;
  overflow-x: hidden;
  overscroll-behavior: contain;   /* wheel over sidebar never scrolls the page */
  -webkit-overflow-scrolling: touch;
  scrollbar-width: thin;
  scrollbar-color: rgba(255, 255, 255, 0.45) transparent;
}

.filters::-webkit-scrollbar { width: 6px; }
.filters::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.45);
  border-radius: 999px;
}
```

Plus a sticky sidebar header so **Filters / Clear All** stays visible while options scroll under it (the aside's top padding was moved onto the head so it sticks flush):

```css
.page-plp .filters { padding-top: 0; }

.page-plp .filters-head {
  position: sticky;
  top: 0;
  z-index: 1;
  background: #4a335f;
  margin: 0 -1.15rem 1rem;
  padding: 1.25rem 1.15rem 0.5rem;
}
```

### Stage 2 changes

Reserve the sidebar's real offset from the viewport top (~190px ≈ 13rem with margin) instead of just the header, and give the last group breathing room (scroll containers can clip the container's own bottom padding):

```css
max-height: calc(100dvh - 13rem);
scroll-padding-bottom: 1.5rem;
```

```css
.page-plp .filter-group:last-child {
  margin-bottom: 0;
  padding-bottom: 1.75rem;
}
```

With `top: calc(var(--header-h) + 1rem)` still in place, the box bottom now stays inside the viewport **both** at page load and after the sidebar sticks during page scroll.

## 4. Verification (headless Chrome)

Sidebar scrolled internally to its maximum (`scrollTop = scrollHeight`) at three viewport sizes — the page itself did not move, the Filters header stayed pinned, and the last option was checked against the **viewport**, not just the sidebar box:

| Viewport | Sidebar bottom vs viewport | Last option (Meenakari) |
|---|---|---|
| 1280 × 713 | 695 ≤ 713 ✅ | Fully visible |
| 1024 × 428 | 410 ≤ 428 ✅ | Fully visible |
| 1440 × 813 | 795 ≤ 813 ✅ | Fully visible |

Before the Stage 2 fix the same check read `775 > 713` ❌ (last option off-screen).

## 5. How to test

1. Hard-refresh `plp.html` (`Ctrl + Shift + R`).
2. Hover the mouse over the filter sidebar and scroll — only the sidebar moves; the product grid stays put.
3. Scroll the sidebar to the bottom — **Meenakari** (last Craft Technique option) is fully visible, and **Filters / Clear All** remains pinned at the top of the sidebar.
4. All filter options are reachable without scrolling the page at any window height.
