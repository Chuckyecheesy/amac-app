# Data Source Matrix — Canadian Grocery (Research)

**Status:** Research backlog — **no source validated or integration-tested**  
**Last updated:** 2026-10-11  
**Related:** [specs/data-sources.md](../../specs/data-sources.md) · [adr/002-grocery-data-strategy.md](../../adr/002-grocery-data-strategy.md)

## How to read this matrix

| Column | Meaning |
| --- | --- |
| Validation status | `unverified` (default) / `researching` / `licensed` / `rejected` |
| Licensing status | Distinct from technical access; may be unknown |
| Estimated cost | `[ESTIMATE]` only until quotes exist |
| Available data fields | Expected/desired fields—not confirmed unless noted |

**Labels:** `[FACT]` project fact · `[ESTIMATE]` · `[ASSUMPTION]` · `[OPEN]`

**[FACT]** Rows below are **candidates for research**. Presence here is not evidence of API availability, partnership, or permission to use.

---

## Matrix

### DS-R-001 — Loblaw companies banners (candidate)

| Field | Value |
| --- | --- |
| Candidate source | Loblaw banners (e.g., Loblaws, Real Canadian Superstore, No Frills, Independent—exact banner list TBD) |
| Retailer / geo coverage | Broad Canada presence historically; **per-store feed coverage unverified** |
| Available data fields | Unknown until partner docs reviewed; desired: store id, SKU/GTIN, price, promo dates, unit size |
| Access method | `[OPEN]` Partner API / flyer program / none—research required |
| Licensing status | `unverified` |
| Price / promotion freshness | Unknown |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | Loyalty (PC Optimum) prices likely account-specific; do not claim without eligibility |

### DS-R-002 — Empire / Sobeys banners (candidate)

| Field | Value |
| --- | --- |
| Candidate source | Sobeys, Safeway, FreshCo, Foodland, IGA (regional)—list TBD |
| Retailer / geo coverage | Strong in multiple provinces; exact digital catalog access unverified |
| Available data fields | Unknown |
| Access method | `[OPEN]` |
| Licensing status | `unverified` |
| Price / promotion freshness | Unknown |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | Banner assortment differs by region |

### DS-R-003 — Metro Inc. banners (candidate)

| Field | Value |
| --- | --- |
| Candidate source | Metro, Food Basics, Super C (as applicable) |
| Retailer / geo coverage | Primarily Ontario/Quebec—confirm before pilot targeting |
| Available data fields | Unknown |
| Access method | `[OPEN]` |
| Licensing status | `unverified` |
| Price / promotion freshness | Unknown |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | Quebec language/flyer rules may apply for UI copy `[OPEN]` |

### DS-R-004 — Walmart Canada (candidate)

| Field | Value |
| --- | --- |
| Candidate source | Walmart Canada grocery |
| Retailer / geo coverage | Wide national store footprint; e-comm vs in-store price parity **unverified** |
| Available data fields | Unknown |
| Access method | `[OPEN]` Affiliate/partner/API programs—do not assume scrape rights |
| Licensing status | `unverified` |
| Price / promotion freshness | Unknown |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | Marketplace vs store grocery mix must be filtered carefully if ever integrated |

### DS-R-005 — Costco Canada (candidate)

| Field | Value |
| --- | --- |
| Candidate source | Costco Canada |
| Retailer / geo coverage | Membership warehouses; fewer locations per metro |
| Available data fields | Unknown; membership-gated pricing likely |
| Access method | `[OPEN]` |
| Licensing status | `unverified` |
| Price / promotion freshness | Unknown |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | Membership eligibility; pack sizes differ from typical grocery equivalence |

### DS-R-006 — Flyer aggregator platforms (candidate class)

| Field | Value |
| --- | --- |
| Candidate source | Consumer flyer aggregators (e.g., Flipp and peers)—**names listed as research targets only** |
| Retailer / geo coverage | Often multi-banner Canada flyers in consumer apps; **commercial API licensing unverified** |
| Available data fields | Flyer product images/text, valid dates, store circulars—field stability unknown |
| Access method | `[OPEN]` Official API / partnership only; scraping not assumed lawful |
| Licensing status | `unverified` |
| Price / promotion freshness | Circular-week cadence typical for flyers `[ASSUMPTION]` |
| Estimated cost | Unknown (could be commercial license) |
| Validation status | `unverified` |
| Known limitations | Flyer price ≠ checkout price; may lack GTIN; loyalty stacking unclear |

### DS-R-007 — Regional western banners (candidate)

| Field | Value |
| --- | --- |
| Candidate source | Save-On-Foods and other western regional banners |
| Retailer / geo coverage | Western Canada emphasis |
| Available data fields | Unknown |
| Access method | `[OPEN]` |
| Licensing status | `unverified` |
| Price / promotion freshness | Unknown |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | Not sufficient alone for Canada-wide claim |

### DS-R-008 — Partner file drop / pilot CSV (fallback candidate)

| Field | Value |
| --- | --- |
| Candidate source | Negotiated pilot feed (CSV/JSON) for specific stores |
| Retailer / geo coverage | Intentionally narrow (one metro) |
| Available data fields | Definable in contract: SKU, name, size, price, valid window, store id |
| Access method | SFTP/S3/email drop with schema |
| Licensing status | `unverified` until agreement exists |
| Price / promotion freshness | Contractual SLA |
| Estimated cost | `[ESTIMATE]` negotiation-dependent; possibly favorable for pilot |
| Validation status | `unverified` |
| Known limitations | Manual ops; not scalable nationally without automation |

### DS-R-009 — User-provided flyer / receipt data (degraded mode)

| Field | Value |
| --- | --- |
| Candidate source | User paste/upload |
| Retailer / geo coverage | Whatever user provides |
| Available data fields | Unstructured → needs parsing; high error risk |
| Access method | In-app upload |
| Licensing status | User grants use for their session; not a substitute for retailer license for shared catalog |
| Price / promotion freshness | User-dependent |
| Estimated cost | Engineering time; low data $ |
| Validation status | `unverified` as primary strategy; acceptable degraded fallback per ADR-002 |
| Known limitations | Does not meet “research deals autonomously” alone |

### DS-R-010 — Loyalty / coupon platforms (candidate class)

| Field | Value |
| --- | --- |
| Candidate source | Retailer loyalty (PC Optimum, Scene+, store apps) & digital coupons |
| Retailer / geo coverage | Program-specific |
| Available data fields | Offers often personalized |
| Access method | `[OPEN]` OAuth/partner only; **no password collection in MVP** |
| Licensing status | `unverified` |
| Price / promotion freshness | Highly time-bound |
| Estimated cost | Unknown |
| Validation status | `unverified` |
| Known limitations | FR-034: never show as available without eligibility=`eligible` |

---

## Cross-cutting research checklist

For each candidate moved to `researching`:

1. [ ] Find official developer/partner portal or business contact.
2. [ ] Obtain terms that mention commercial display/caching.
3. [ ] Request sample payload (fields audit).
4. [ ] Measure or request freshness cadence.
5. [ ] Document cost model.
6. [ ] Legal sign-off before `licensed`.
7. [ ] Technical sandbox test before claiming “integration tested.”

## Summary counts (as of last update)

| Validation status | Count |
| --- | --- |
| unverified | 10 |
| researching | 0 |
| licensed | 0 |
| rejected | 0 |

## Open research questions

| ID | Question |
| --- | --- |
| MX-OPEN-001 | Is there any Canada grocery catalog API with self-serve commercial access in 2026? |
| MX-OPEN-002 | Minimum viable licensed flyer dataset for one metro PoC? |
| MX-OPEN-003 | Evidence requirements legal will accept for TOU-only (no MSA)? |
