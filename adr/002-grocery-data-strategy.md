# ADR-002 — Grocery Data Strategy

**Status:** Proposed (feasibility unresolved)  
**Date:** 2026-10-11  
**Related:** [specs/data-sources.md](../specs/data-sources.md) · [docs/research/data-source-matrix.md](../docs/research/data-source-matrix.md) · [specs/roadmap.md](../specs/roadmap.md)

## Context

The MVP requires **real** grocery deal data for a Canada pilot, with provenance, freshness, and lawful use. Scraping public pages is **not** assumed to be permitted. No source has been licensed or integration-tested yet.

## Decision drivers

- FR-021 authorized sources only.
- Lean cost.
- Store-level accuracy sufficient for user trust.
- Eligibility honesty for loyalty/coupons (FR-034).
- Phased geographic rollout (NFR-007).

## Source selection criteria

A source may enter **pilot** only if it meets all of:

1. **License fit:** Written terms or contract allow ingest, store, transform, and display in our assistant for the pilot audience.
2. **Field sufficiency:** Price, identity (GTIN/SKU/native id), store or banner scope, and effective dates (or documented lack thereof).
3. **Freshness:** Update cadence compatible with stale-flag UX; SLA recorded.
4. **Operability:** Stable auth, rate limits understood, failure mode documented.
5. **Cost:** Compatible with lean MVP.
6. **Coverage honesty:** Geographic/retailer scope explicit.

## Licensing requirements

| Requirement | Rule |
| --- | --- |
| Evidence | Matrix `licensing_status` moves to `licensed` only with evidence reference (private contract ID / TOU attestation)—not pasted secrets |
| Scope | Display + cache duration + territory documented |
| Prohibitions | Respect no-resale / no-training / no-redistribution clauses |
| User-provided data | Allowed with `user_provided` class and clear UI labeling |
| Scraping | Default **forbidden** until counsel says otherwise for a specific site |

## Data freshness and coverage thresholds

**Draft thresholds** `[ASSUMPTION]`—finalize after vendor discovery:

| Metric | Pilot threshold |
| --- | --- |
| Refresh lag P50 | ≤ 24h for weekly flyers; ≤ 6h for “live” price APIs if offered |
| Stale UX | Flag when age > source SLA |
| Coverage | ≥1 metro FSA set with ≥2 banners **or** 1 banner with deep assortment if only one license |
| Identity match rate | `[ESTIMATE]` ≥ 70% of pilot basket items resolvable to an offer—measure in PoC |
| Provenance completeness | 100% of displayed prices |

Failure to meet thresholds → delay pilot or shrink scope; do not silently widen scraping.

## Fallback options

| Priority | Fallback |
| --- | --- |
| 1 | Second licensed feed for same banner |
| 2 | Partner CSV/file drop for pilot stores |
| 3 | User-pasted flyer items with `user_provided` provenance (degraded mode) |
| 4 | Waitlist / coverage messaging without fake prices |

## Decision

**Adopt a license-first, phased-coverage strategy:** build ingestion abstractions now (spec + future interfaces); bind the first concrete source only after G0 (roadmap). Optimizer and UI must treat source failures and unknown eligibility as first-class.

Until G0 passes, all retailer capabilities in docs remain **candidates**.

## Alternatives considered

| Approach | Verdict |
| --- | --- |
| Scrape retailer sites | Reject by default (rights + brittleness) |
| User-only manual prices | Insufficient for “real deals” vision; emergency fallback only |
| Delay product until national API | Too slow; prefer narrow licensed pilot |
| Synthetic data only forever | Blocks pilot claims; OK for early engineering tests only |

## Unresolved feasibility questions

| ID | Question |
| --- | --- |
| DATA-V-001 | Which Canadian banners offer partner APIs for assistants? |
| DATA-V-002 | Do flyer aggregators license commercial reuse / APIs in Canada? |
| DATA-V-003 | Store-level vs zone-level pricing in available feeds? |
| DATA-V-004 | Can loyalty prices be verified without storing retailer passwords? |
| DATA-V-005 | What cache TTLs do licensors allow? |
| DATA-V-006 | GTIN completeness for equivalence matching? |

## Consequences

- Roadmap gate G0 is mandatory before public “real deals” marketing.
- Engineering can proceed on fixtures that **mimic** authorized schemas without claiming live coverage.
- Matrix in `docs/research/` is the source of truth for validation status.
