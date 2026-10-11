# Acceptance Criteria — Agentic Grocery AI

**Status:** Draft  
**Related:** [product-requirements.md](product-requirements.md) · [optimization.md](optimization.md) · [privacy-security.md](privacy-security.md)

Each criterion is testable. IDs are stable (`AC-###`).

## 1. Onboarding and preferences

| ID | Criterion | Req |
| --- | --- | --- |
| AC-001 | New user can register/sign-in and cannot access another user’s prefs. | FR-001, AUTH-002 |
| AC-002 | Onboarding blocks completion until `priority_mode` is set to one of budget/quality/balanced. | FR-001 |
| AC-003 | User can grant location permission and see only stores in verified coverage for resolved area. | FR-002 |
| AC-004 | User can skip/deny location, enter valid postal code `A1A 1A1`, and proceed; invalid codes are rejected. | FR-002 |
| AC-005 | Dietary hard constraints and brand prefs persist and reload on next session. | FR-003 |
| AC-006 | Settings allow updating priority, location method, and constraints; subsequent optimize uses new values. | FR-004 |

## 2. Intent and clarification

| ID | Criterion | Req |
| --- | --- | --- |
| AC-010 | User can submit natural-language restock intent and receive a structured session in `draft`/`researching`. | FR-010 |
| AC-011 | If postal code and priority missing, system asks before optimize; does not invent them. | FR-011 |
| AC-012 | With complete prefs, restock path does not re-ask priority/location unless user changes them. | FR-012 |

## 3. Stores, provenance, freshness

| ID | Criterion | Req |
| --- | --- | --- |
| AC-020 | For an in-coverage postal code fixture, `list_stores` returns only seeded covered stores. | FR-020 |
| AC-021 | Integration tests use licensed/fixture sources only; code paths reject `licensing_status=rejected`. | FR-021 |
| AC-022 | Every displayed price includes source id/name, retrieved_at, and conditions/dates when provided by source. | FR-022 |
| AC-023 | Offers older than source SLA (or draft 48h if SLA null) render `stale` badge. | FR-023 |
| AC-024 | When primary source stub fails, fallback stub used and UI shows reduced confidence. | FR-024 |

## 4. Optimization and pricing correctness

| ID | Criterion | Req |
| --- | --- | --- |
| AC-030 | Unit-price fixture OPT-TC-001: cheaper per base unit selected in budget mode. | FR-030 |
| AC-031 | Same item set yields different objective behavior across three modes (see OPT-TC-012 vs 001). | FR-031 |
| AC-032 | Optimizer pure functions: fixed inputs → identical outputs; LLM disabled in unit tests. | FR-032 |
| AC-033 | Savings line absent unless `baseline_definition` and `baseline_total_cents` set; OPT-TC-008 passes. | FR-033 |
| AC-034 | Loyalty-gated offer with `eligibility=unknown` not applied to totals (OPT-TC-003). | FR-034 |
| AC-035 | Hard constraint fixture excludes violating SKU even if cheapest (OPT-TC-005). | FR-035 |
| AC-036 | With `max_stores=1`, optimizer never returns >1 store; multi-store suggestion triggers HITL flag when threshold crossed. | FR-036 |

## 5. Human approval

| ID | Criterion | Req |
| --- | --- | --- |
| AC-040 | Workflow cannot enter `approved` without POST approval from session owner. | FR-040, HITL-001 |
| AC-041 | Draft plan UI shows at least one explicit trade-off when multi-store or quality-vs-price conflict exists in fixture. | FR-041 |
| AC-042 | Approve/reject/revise writes `APPROVAL_RECORD` with timestamp and result id. | FR-042 |
| AC-043 | No API exists in MVP that places retailer orders or charges payment methods. | FR-043 |

## 6. List review and gaps

| ID | Criterion | Req |
| --- | --- | --- |
| AC-050 | Review screen lists item, store, qty, unit/package price, line total, freshness, source. | FR-050 |
| AC-051 | Editing qty and re-optimize updates totals deterministically. | FR-051 |
| AC-052 | Unavailable product creates gap item; substitute proposal blocked if hard constraint violated. | FR-052 |

## 7. Failures and recovery

| ID | Criterion | Req |
| --- | --- | --- |
| AC-060 | Price verification failure marks offer unconfirmed; UI offers retry/exclude; approve blocked if user hasn’t acknowledged empty critical prices when all prices failed. | FR-060 |
| AC-061 | Mid-session source failure leaves prior valid offers intact; workflow not wiped. | FR-061 |

## 8. Automated calculation tests (CI)

| ID | Criterion | Req |
| --- | --- | --- |
| AC-070 | Unit tests cover OPT-TC-001–012 or documented successors. | NFR-004 |
| AC-071 | Basket total equals sum of line totals for fixture baskets. | FR-032 |
| AC-072 | Promotion eligibility gate tests: eligible/ineligible/unknown. | FR-034 |
| AC-073 | Savings computation tests for each allowed baseline code. | FR-033 |

## 9. Provenance and freshness tests

| ID | Criterion | Req |
| --- | --- | --- |
| AC-080 | Inserting offer without `retrieved_at` fails validation. | FR-022 |
| AC-081 | Expired `valid_to` offers excluded from candidates. | FR-023 |
| AC-082 | Approval with stale lines requires `ack_stale_items=true`. | HITL-004 |

## 10. Human approval tests

| ID | Criterion | Req |
| --- | --- | --- |
| AC-090 | Attempted state jump `optimizing` → `approved` via API rejected. | FR-040 |
| AC-091 | Server recomputes totals on approve; tampered client total rejected. | ARCH boundary |
| AC-092 | Metric/invariant: zero silent finalizes in e2e suite. | M-007 |

## 11. Source failure and recovery tests

| ID | Criterion | Req |
| --- | --- | --- |
| AC-100 | Source 500 with retries exhausted → structured error, no fabricated offers. | FR-060 |
| AC-101 | Circuit open → fallback path or degraded coverage message. | FR-024 |
| AC-102 | Job marks sources failing freshness lag for ops metric. | NFR-006 |

## 12. Privacy and security tests

| ID | Criterion | Req |
| --- | --- | --- |
| AC-110 | Location consent recorded on grant/postal entry. | PRIV-001 |
| AC-111 | Prompt-injection fixture in promo text cannot approve session. | NFR-008, PI-003 |
| AC-112 | Client bundle scan fixture ensures no `API_KEY` pattern from env examples. | SEC-001 |

## 13. Pilot performance and quality metrics

Not hard CI gates until baselines exist; track in pilot:

| ID | Metric | Gate idea |
| --- | --- | --- |
| AC-120 | Median time-to-first-plan | Set after PoC (NFR-005) |
| AC-121 | Provenance completeness | 100% on displayed prices (M-004) |
| AC-122 | User-reported price mismatches | Log & review weekly |
| AC-123 | Approval rate | Informational (M-002) |

## 14. Traceability matrix

| Requirements | Acceptance IDs |
| --- | --- |
| FR-001–004 | AC-001–006 |
| FR-010–012 | AC-010–012 |
| FR-020–024 | AC-020–024, AC-080–081, AC-100–102 |
| FR-030–036 | AC-030–036, AC-070–073 |
| FR-040–043 | AC-040–043, AC-090–092 |
| FR-050–052 | AC-050–052 |
| FR-060–061 | AC-060–061 |
| HITL-004 | AC-082 |
| PRIV/SEC | AC-110–112 |
| Pilot metrics | AC-120–123 |
