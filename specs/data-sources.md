# Data Sources — Agentic Grocery AI

**Status:** Draft — feasibility not validated  
**Related:** [docs/research/data-source-matrix.md](../docs/research/data-source-matrix.md) · [adr/002-grocery-data-strategy.md](../adr/002-grocery-data-strategy.md) · [product-requirements.md](product-requirements.md)

## 1. Principles

1. **Do not fabricate** retailer coverage, API capabilities, or licensing permissions.
2. **Do not assume** public web pages may be scraped or commercially reused.
3. Label every source row with validation status: `unverified` | `researching` | `licensed` | `rejected`.
4. Prefer authorized feeds, retailer partner APIs, and licensed aggregators over ad-hoc collection.

**[FACT]** As of this specification phase, **no external grocery data source has been contractually validated or integration-tested** in this project.

## 2. Candidate Canadian grocery data sources

Candidates only. Details and status live in the [data-source matrix](../docs/research/data-source-matrix.md).

| Category | Examples (candidates) | Notes |
| --- | --- | --- |
| National / large banners | Loblaw companies banners, Sobeys/Empire banners, Metro, Walmart Canada, Costco Canada | Access method and licensing **unverified** |
| Regional banners | Save-On-Foods, Federated Co-op, independent banners | Coverage varies by province |
| Flyer / deal aggregators | Flipp and similar consumer flyer platforms | May be consumer-only; commercial API/licensing **unverified** |
| Open / government | None identified as a complete grocery price feed for MVP | Do not invent open datasets |
| Loyalty / coupons | PC Optimum, Scene+, store coupons | Eligibility often account-specific; MVP must not claim prices without eligibility |
| Manual curated pilot | Licensed flyer PDFs or partner CSV for one city | Possible fallback for vertical slice |

## 3. Retailer and regional coverage

**[ASSUMPTION]** Pilot starts with **one metro area** and **1–3 banners** once a licensed/authorized path exists.

| Dimension | MVP rule |
| --- | --- |
| Geographic | Only postal codes in verified coverage marked available (NFR-007) |
| Retailer | Store discovery returns only onboarded banners |
| Phased rollout | Expand when source evaluation passes freshness + licensing gates |

Coverage claims in product UI must match the coverage table in the database—not marketing aspiration.

## 4. Public, licensed, and authenticated distinctions

| Class | Definition | MVP use |
| --- | --- | --- |
| Public webpage | HTML/PDF visible in a browser without login | **Not sufficient** for reuse without legal review |
| Public API | Documented API with terms allowing our use case | Allowed if terms verified and recorded |
| Licensed feed | Contract/TOU grants ingest + display rights | Preferred |
| Authenticated user-delegated | User OAuth/connect to loyalty | Only if partner supports; store tokens securely |
| User-pasted data | User uploads a flyer or list | Allowed with clear “user-provided” provenance |

## 5. Source freshness and reliability

| Field | Purpose |
| --- | --- |
| `retrieved_at` | When we fetched the record |
| `valid_from` / `valid_to` | Offer window if known |
| `freshness_sla_hours` | Per-source expected refresh `[OPEN]` |
| `reliability_tier` | `A` stable feed / `B` best-effort / `C` experimental |

**Classifier (normative for app logic):**

| State | Rule (draft) |
| --- | --- |
| `fresh` | `now <= valid_to` (if set) AND age ≤ source SLA |
| `stale` | Within offer window but age > SLA, or SLA unknown and age > `[ASSUMPTION]` 48h |
| `expired` | `now > valid_to` |
| `unknown` | Missing timestamps or conflicting sources |

Exact SLA values are **open** until sources are validated (OPEN-002).

## 6. Usage rights and licensing checks

Before any integration:

1. Identify rights holder and terms URL / contract.
2. Confirm permitted: store, transform, display in our app, cache duration.
3. Confirm prohibited: redistribution, training, resale.
4. Record outcome in data-source matrix (`licensing_status`).
5. Legal/product sign-off gate in [roadmap.md](roadmap.md).

**[OPEN]** No license has been signed for this project yet.

## 7. Loyalty and coupon limitations

| Limitation | Product rule |
| --- | --- |
| Account-specific pricing | Do not show as user’s price unless eligibility established (FR-034) |
| Stacking rules opaque | If stacking unknown → treat benefit as `unknown`, not applied |
| Expiry and region gates | Must be in provenance conditions |
| Credential access | MVP avoids storing retailer passwords (PRIV-004) |
| “Up to” marketing savings | Not a baseline for savings math |

## 8. Source evaluation methodology

For each candidate (see ADR-002):

1. **Access:** Is there a documented API, licensed file drop, or partner program?
2. **Fields:** Products, GTINs/SKUs, store IDs, prices, promo text, dates, unit size?
3. **Coverage:** Provinces, banners, store-level vs banner-level?
4. **Freshness:** Update cadence; historical accuracy `[ESTIMATE]` via sample audit.
5. **License:** Written permission for our use case?
6. **Cost:** Fee model; fit lean MVP.
7. **Operability:** Auth, rate limits, support channel.
8. **Decision:** `pilot` / `hold` / `reject` with rationale.

Validation status becomes `licensed` only after evidence is attached (contract ID or written TOU excerpt reference stored privately—not secrets in git).

## 9. Fallback strategies

| Scenario | Fallback |
| --- | --- |
| Primary feed down | Secondary authorized source for same banner if any; else mark store degraded |
| No licensed national API at PoC | Narrow pilot: one region + partner CSV/flyer license |
| Loyalty ineligible/unknown | Show shelf/flyer price only; label loyalty as unverified |
| Unit size missing | Block unit-price compare; show package price + “unit price unavailable” |
| Out of coverage | Honest empty state; do not scrape ad hoc |

## 10. Open questions

| ID | Question |
| --- | --- |
| DS-OPEN-001 | Which aggregator or retailer programs allow commercial assistant use in Canada? |
| DS-OPEN-002 | Store-level vs banner-level pricing availability? |
| DS-OPEN-003 | GTIN coverage sufficient for equivalence matching? |
| DS-OPEN-004 | Acceptable cache duration per licensor? |
| DS-OPEN-005 | Quebec bilingual flyer obligations for pilot? |
