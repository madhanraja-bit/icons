# Chandra / Silver — storefront walkthrough specimen

**Ticket:** [KJEI-T0089](https://prana.praniontech.tools/projects/tasks/KJEI-T0089/edit) (origin [KJEI-T0088](https://prana.praniontech.tools/projects/tasks/KJEI-T0088/edit))  
**Absorbed:** 22 Oct 2026 from `vk_own_understanding/Oct 22/chandra/`  
**Plays:** **approved** 22 Oct 2026 (Engineering Owner Vijayakumar K. on KJEI-T0089; Customer Owner co-sign welcome)  
**Scenarios:** [S-CHN-01](../../scenarios/browse/S-CHN-01-open-chandra-home.md) · [S-CHN-02](../../scenarios/browse/S-CHN-02-browse-silver-products.md) · [S-CHN-03](../../scenarios/browse/S-CHN-03-division-driven-product-look.md)  
**Journey:** [09](../../choreography/09-chandra-silver-experience.md)  
**Architecture fence:** [ADR-0013](../../architecture/ADR-0013-chandra-silver-dual-look.md) **Accepted**

## Open locally

```bash
# from this folder
python3 -m http.server 8765
# then http://localhost:8765/
```

| File | Screen | FR / UC |
|------|--------|---------|
| [`index.html`](index.html) | Chandra home (LP) | `FR-CHN-01` / UC-CHN-001 |
| [`plp.html`](plp.html) | Silver listing (PLP) | `FR-CHN-02` / UC-CHN-002 |
| [`pdp.html`](pdp.html) | Chandra product detail (PDP look) | `FR-CHN-03` / UC-CHN-003 |

## Role

- **Look-and-feel SoR** for Chandra home / listing / Silver product look until a live UAT Chandra surface exists.
- Demo / agreement surface — **not** a substitute for UCD acceptance or scenario Then tables.
- Placeholders and unfinished creatives allowed (`OI-31`). Named collection tiles in the specimen are **not** required destinations until `OI-30` closes. Exact main-nav labels remain `OI-29`.
- Shared Kalyan header remains required on the live site (`FR-CHN-01`…`03`); specimen chrome is illustrative.
- Gold PLP auto-load / sticky Filter are **not** assumed for Chandra listings in this FR set.

Ref mockup PNGs from the working folder were not copied (not linked by the HTML).
