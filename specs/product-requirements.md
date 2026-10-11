# Product Requirements — Agentic Grocery AI

**Status:** Draft for specification review  
**Related:** [user-journeys.md](user-journeys.md) · [acceptance-criteria.md](acceptance-criteria.md) · [architecture.md](architecture.md) · [privacy-security.md](privacy-security.md)

## 1. Problem statement

Household grocery shopping in Canada is fragmented across retailers, flyers, loyalty programs, and unit-price conventions. Shoppers waste time comparing promotions that may be location-specific, time-bound, or eligibility-gated—and often miss cheaper equivalent products or overpay when quality preferences are unclear.

Existing tools typically either (a) show flyers without basket optimization, or (b) optimize without transparent provenance and human approval. Users need an assistant that researches real deals, explains trade-offs, and asks for confirmation at consequential steps.

## 2. Product vision

Build a **human-in-the-loop, agentic AI grocery assistant** that helps users restock groceries, find real grocery deals from authorized sources, and optimize shopping baskets.

Example intent:

> "I'm running out of groceries. Help me restock."

The system should understand needs, research relevant stores and promotions, compare suitable products, and produce an optimized shopping list. Research and calculations run autonomously; important decisions require user input and explicit approval before a plan is finalized.

**[FACT]** Initial purchase execution does **not** include autonomous checkout.

## 3. Target users and personas

### 3.1 Target users

| Segment | Description |
| --- | --- |
| Primary | Individuals and families buying household groceries in Canada |
| Secondary (post-MVP) | Shared households with multiple shoppers (same account preferences) |

### 3.2 Personas

#### P-001 — Busy parent (budget-conscious)

- Shops weekly for a family of 4; cares about total spend and avoiding stockouts.
- Willing to visit 1–2 stores if savings justify travel.
- Uses one loyalty program when convenient; will not share passwords lightly.

#### P-002 — Quality-focused cook

- Prefers specific brands, organic produce, or dietary standards.
- Accepts higher prices when quality difference is clear.
- Needs substitutions that never violate hard dietary rules.

#### P-003 — Balanced restocker

- Wants a “good enough” basket quickly: mix of savings and preferred brands.
- Returns often; expects remembered preferences and postal code.
- Approves plans on mobile web before shopping in person.

## 4. User stories

| ID | As a… | I want… | So that… | Priority |
| --- | --- | --- | --- | --- |
| US-001 | new user | to set location via permission or postal code | the app shows nearby relevant stores | MVP |
| US-002 | new user | to choose shopping priority (budget / quality / balance) | recommendations match my goals | MVP |
| US-003 | shopper | to say I’m running low and get a restock plan | I spend less time researching | MVP |
| US-004 | shopper | to answer clarifying questions only when needed | I am not interrupted for trivial details | MVP |
| US-005 | shopper | to see store options and deal comparisons with sources | I trust the plan | MVP |
| US-006 | shopper | to review unit prices and basket totals | I can verify the math | MVP |
| US-007 | shopper | to approve or reject the final shopping plan | nothing consequential happens without me | MVP |
| US-008 | shopper | to see when data is stale or uncertain | I don’t act on bad prices | MVP |
| US-009 | shopper | to set dietary and hard constraints | substitutions never violate them | MVP |
| US-010 | shopper | to have preferences remembered | returning restocks are faster | MVP |
| US-011 | shopper | to handle missing products with safe alternatives | I can still complete my list | MVP |
| US-012 | shopper | **not** to have the app purchase for me | I retain control of checkout | MVP |

## 5. Functional requirements

Requirements use stable IDs. Acceptance criteria live in [acceptance-criteria.md](acceptance-criteria.md).

### 5.1 Onboarding and profile

| ID | Requirement | Links |
| --- | --- | --- |
| FR-001 | The system SHALL allow first-time users to create an account and complete a minimal onboarding profile (household size optional, shopping priority required). | AC-001, AC-002 |
| FR-002 | The system SHALL ask for location via browser permission **or** manual Canadian postal-code entry; location use is optional. | AC-003, AC-004 · PRIV-001 |
| FR-003 | The system SHALL persist household preferences (priority mode, dietary hard constraints, preferred/excluded brands, default store set when known) with user consent. | AC-005 · DM-USER |
| FR-004 | The system SHALL allow users to view and update preferences at any time. | AC-006 |

### 5.2 Intent understanding and clarification

| ID | Requirement | Links |
| --- | --- | --- |
| FR-010 | The system SHALL accept natural-language restock and shopping intents in the web UI. | AC-010 |
| FR-011 | The orchestrator SHALL ask clarifying questions when required inputs are missing (priority, location, hard constraints, budget cap if budget mode and cap unknown). | AC-011 · UJ-CLARIFY |
| FR-012 | The system SHALL avoid asking for information already stored in preferences unless the user indicates a change. | AC-012 |

### 5.3 Store discovery and data

| ID | Requirement | Links |
| --- | --- | --- |
| FR-020 | Given a location or postal code, the system SHALL identify candidate nearby grocery stores from the supported coverage set. | AC-020 |
| FR-021 | The system SHALL only ingest grocery prices/promotions from sources classified as authorized for the intended use (see [data-sources.md](data-sources.md)). | AC-021 · DS-* |
| FR-022 | Every price, promotion, and coupon claim SHALL store provenance: source, applicable dates/conditions, and retrieval timestamp. | AC-022 · DM-PRICE |
| FR-023 | The system SHALL flag uncertain or stale data in the UI before plan approval. | AC-023 |
| FR-024 | When a primary source fails, the system SHALL apply documented fallbacks and disclose reduced confidence. | AC-024 |

### 5.4 Product comparison and optimization

| ID | Requirement | Links |
| --- | --- | --- |
| FR-030 | The system SHALL normalize comparable products to unit price where package size and unit of measure are known. | AC-030 · OPT-UNIT |
| FR-031 | The optimizer SHALL support three modes: budget-first, quality-first, and balanced. | AC-031 · OPT-MODE |
| FR-032 | Optimization calculations for prices, totals, constraints, and savings SHALL be deterministic application code (not LLM arithmetic). | AC-032 |
| FR-033 | Savings SHALL only be shown with a documented baseline definition. | AC-033 · OPT-SAVE |
| FR-034 | The system SHALL never present coupon/loyalty prices as available unless eligibility is established for that user/session. | AC-034 |
| FR-035 | Substitutions SHALL never violate explicit dietary or other hard constraints. | AC-035 |
| FR-036 | Store selection MAY consider travel burden as a soft factor; travel time/distance are advisory unless user sets a hard max-stores or max-distance preference. | AC-036 |

### 5.5 Human-in-the-loop and approval

| ID | Requirement | Links |
| --- | --- | --- |
| FR-040 | The AI MAY research and propose autonomously; it SHALL require explicit user approval before finalizing the shopping plan. | AC-040 · HITL-001 |
| FR-041 | The system SHALL present trade-offs (price vs quality, multi-store vs single-store) at decision points before finalization. | AC-041 |
| FR-042 | The system SHALL record approval decisions (approve / reject / revise) with timestamps. | AC-042 · DM-APPROVAL |
| FR-043 | The MVP SHALL NOT execute autonomous checkout or purchasing. | AC-043 |

### 5.6 Shopping list and review

| ID | Requirement | Links |
| --- | --- | --- |
| FR-050 | The system SHALL produce a structured shopping list with items, stores, quantities, unit prices, line totals, and source flags. | AC-050 |
| FR-051 | Users SHALL be able to edit quantities, remove items, lock preferred products, and re-run optimization. | AC-051 |
| FR-052 | Missing or unavailable products SHALL trigger a clear gap state with optional constrained substitutions. | AC-052 |

### 5.7 Error and recovery

| ID | Requirement | Links |
| --- | --- | --- |
| FR-060 | Price verification failures SHALL block presenting that price as confirmed; user is notified with recovery options. | AC-060 |
| FR-061 | Partial source outages SHALL not wipe the session; completed research steps remain available where valid. | AC-061 |

## 6. Non-functional requirements

| ID | Requirement | Links |
| --- | --- | --- |
| NFR-001 | Initial platform is a responsive web application. | ARCH |
| NFR-002 | Architecture SHALL minimize infrastructure and API cost for MVP (lean stack). | ADR-001 |
| NFR-003 | Personal data SHALL be minimized; credentials never stored in client-visible config or docs. | PRIV-* |
| NFR-004 | Key optimizer paths (unit price, totals, eligibility gates, savings) SHALL have automated tests. | AC-070+ |
| NFR-005 | End-to-end restock proposal for a typical 15–25 item list SHOULD complete within `[ESTIMATE]` 60 seconds of interactive wait when sources are healthy; exact SLO TBD after PoC. | OPEN-PERF |
| NFR-006 | System SHALL emit structured logs/traces for workflow states, source calls, and optimizer runs (no secrets in logs). | ARCH-OBS |
| NFR-007 | Phased Canada-wide rollout: only regions/retailers with verified coverage are marked available. | DS-COV |
| NFR-008 | Untrusted source text SHALL be treated as data, not instructions (prompt-injection resistance). | PRIV-INJECT |

## 7. MVP scope and exclusions

### 7.1 In scope (MVP)

- Web app onboarding, preferences, location/postal code.
- Single orchestrator with modular tools/capabilities.
- Authorized deal/price ingestion for a **pilot coverage set** (retailers/regions TBD after feasibility).
- Deterministic basket optimization (three modes).
- Provenance, staleness flags, human approval.
- Shopping-list review UI; export/print-friendly view `[ASSUMPTION]` acceptable as MVP share mechanism.

### 7.2 Out of scope (MVP)

| Exclusion | Rationale |
| --- | --- |
| Autonomous checkout / cart push to retailer | Product decision; liability and auth complexity |
| Native iOS/Android apps | Web-first |
| Multi-agent autonomous teams | Unjustified for MVP |
| Unlicensed scraping | Legal/data integrity risk |
| Full Canada coverage day one | Phased by verified coverage |
| Recipe planning / meal kits | Scope control |
| In-app payments | Not needed without checkout |

## 8. Human-in-the-loop requirements

| ID | Rule |
| --- | --- |
| HITL-001 | Finalize shopping plan only after explicit Approve action. |
| HITL-002 | Ask before expanding to additional stores when multi-store savings exceed a soft threshold **or** when user preference is single-store `[ASSUMPTION]` threshold configurable; default ask. |
| HITL-003 | Ask before any substitution that changes brand family or dietary-relevant attributes even if not a hard violation. |
| HITL-004 | Surface uncertain/stale items and require acknowledgment if any critical-path price is stale. |
| HITL-005 | Never treat LLM suggestions as approved prices or eligibility. |

## 9. Privacy and safety requirements

Summary; normative detail in [privacy-security.md](privacy-security.md).

| ID | Requirement |
| --- | --- |
| PRIV-001 | Location optional; postal code alternative; consent recorded. |
| PRIV-002 | Minimize PII; no payment card storage in MVP. |
| PRIV-003 | Household preferences protected under authenticated access. |
| PRIV-004 | Loyalty credentials: MVP avoids storing retailer passwords; eligibility integrations only via approved OAuth/partner flows if/when available `[OPEN]`. |
| PRIV-005 | Dangerous or illegal request content is refused; dietary hard constraints are never overridden for savings. |

## 10. Success metrics

| ID | Metric | Target | Type |
| --- | --- | --- | --- |
| M-001 | Pilot users who complete onboarding | `[ESTIMATE]` ≥ 70% of invited | Product |
| M-002 | Sessions reaching plan approval | `[ESTIMATE]` ≥ 50% of restock sessions with healthy data | Product |
| M-003 | Optimizer unit-test pass rate in CI | 100% on main | Quality |
| M-004 | Provenance completeness on displayed prices | 100% | Integrity |
| M-005 | User-reported “price wrong in store” rate | Track; threshold set after pilot baseline | Quality |
| M-006 | Median time-to-first-plan | Measure in PoC; set SLO later | Performance |
| M-007 | Explicit approval rate vs silent finalize | 0 silent finalizes (must be zero) | Safety |

## 11. Assumptions

| ID | Assumption |
| --- | --- |
| A-001 | Users can shop in person or via retailer apps using an exported list; we do not need checkout for MVP value. |
| A-002 | At least one authorized Canadian price/promotion feed or partnership will be obtainable for pilot geography. |
| A-003 | Postal code is sufficient for store discovery granularity in MVP. |
| A-004 | English UI first; French localization post-MVP unless pilot requires it. |

## 12. Open questions

| ID | Question | Blocks |
| --- | --- | --- |
| OPEN-001 | Which retailers/regions have verifiable authorized data access for pilot? | Data feasibility gate |
| OPEN-002 | What freshness SLA is achievable per source? | Staleness UX thresholds |
| OPEN-003 | Can any loyalty price be verified without storing credentials? | FR-034 implementation |
| OPEN-004 | Exact performance SLO for NFR-005 | Pilot ops |
| OPEN-005 | French-language requirement for Quebec pilot? | Localization |

## 13. Traceability map (summary)

| Area | Requirements | Acceptance |
| --- | --- | --- |
| Onboarding / location | FR-001–004 | AC-001–006 |
| Intent / clarify | FR-010–012 | AC-010–012 |
| Stores / data | FR-020–024 | AC-020–024 |
| Optimize | FR-030–036 | AC-030–036 |
| HITL | FR-040–043 | AC-040–043 |
| List / gaps | FR-050–052 | AC-050–052 |
| Errors | FR-060–061 | AC-060–061 |
| NFR / privacy | NFR-*, PRIV-* | AC-070+, privacy ACs |
