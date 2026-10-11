# Basket Optimization — Agentic Grocery AI

**Status:** Draft  
**Related:** [product-requirements.md](product-requirements.md) · [data-model.md](data-model.md) · [acceptance-criteria.md](acceptance-criteria.md)

**Invariant:** All monetary math runs in deterministic application code (FR-032). The LLM may explain results but must not invent or alter numbers.

## 1. Modes

### OPT-MODE-BUDGET — Budget-first

**Objective:** Minimize `basket_total_cents` subject to hard constraints; among ties, prefer fewer stores, then preferred brands.

**Secondary:** Maximize `savings_cents` only when baseline is defined.

### OPT-MODE-QUALITY — Quality-first

**Objective:** Maximize preference match score; minimize total only among equal match scores.

Match score (draft deterministic):

\[
score = w_b \cdot brand\_match + w_t \cdot dietary\_tag\_match + w_q \cdot quality\_tier\_match
\]

Default weights `[ASSUMPTION]`: \(w_b=0.5\), \(w_t=0.3\), \(w_q=0.2\). Stored in `algorithm_version`.

### OPT-MODE-BALANCED

**Objective:** Minimize hybrid cost:

\[
hybrid = basket\_total\_cents - \lambda \cdot preference\_score\_scaled
\]

`λ` default `[ASSUMPTION]` chosen so a strong brand preference can outweigh small unit-price gaps; tuned in PoC; versioned.

## 2. Product equivalence and unit-price normalization (OPT-UNIT)

### 2.1 Equivalence

Two variants are comparable if they share `equivalence_key` **or** (same category AND compatible UOM AND not excluded by brand lock rules).

Hard constraints always filter before scoring.

### 2.2 Unit price

When `net_quantity > 0` and `uom` in `{g,kg,ml,l}`:

1. Convert to base unit (`g` or `ml`).
2. `unit_price = price_cents / base_quantity`.
3. Display per 100g / 100ml / each as appropriate.

If quantity/UOM missing → unit price `unavailable`; package price may still be used for non-comparable lines.

**Currency:** CAD cents as integers; avoid floating cash totals.

## 3. Budget and dietary constraints

| Type | Examples | Enforcement |
| --- | --- | --- |
| Hard dietary | peanut_free, vegan, halal, allergen bans | Filter; never substitute into violation (FR-035) |
| Hard budget | `budget_cap_cents` when user marks hard | Feasible set only; if empty → gap report |
| Soft budget | target spend | Penalty in score; allow exceed with HITL warning |
| Brand lock | user locked variant | Optimizer must keep or ask |
| Store count | `max_stores` | Hard if set |

## 4. Store selection and travel

1. Candidate stores = coverage ∩ distance/postal filter.
2. Default: solve single-store baskets per store; pick best by mode.
3. Multi-store search: optional if `max_stores > 1` or unset; if savings vs best single-store exceed soft threshold → **HITL** ask (HITL-002).
4. Travel distance is advisory unless `max_distance_km` set.

Soft threshold `[ASSUMPTION]`: multi-store suggested when extra savings ≥ 5% of basket **and** ≥ $5 CAD—validate in pilot.

## 5. Substitution rules

| Rule ID | Rule |
| --- | --- |
| SUB-001 | Never violate hard dietary constraints. |
| SUB-002 | Prefer same `equivalence_key`. |
| SUB-003 | Brand family change → propose, require HITL unless user enabled auto-substitute. |
| SUB-004 | Size change allowed if unit price computed and qty adjusted to meet need. |
| SUB-005 | If no safe substitute → `gap_reason` on basket item. |

## 6. Coupon and loyalty eligibility

```text
if offer.price_type in {loyalty_gated, member} or coupon applied:
  assessment = eligibility(offer, user)
  if assessment != eligible:
    do not use that price in basket_total
    may show as informational "if eligible" with status=unknown/ineligible
```

Stacking: only apply when `stackable=yes` or explicit rule engine entry; else ignore stack benefit.

## 7. Savings calculation and baseline (OPT-SAVE)

Savings MUST NOT be shown without `baseline_definition`.

### Allowed baselines (pick one per result; record on OPTIMIZATION_RESULT)

| Code | Definition |
| --- | --- |
| `BASE-SINGLE-SHELF` | Sum of non-promo shelf prices for the **same** chosen variants at the same stores |
| `BASE-CHEAPEST-EQUIV-QUALITY` | For budget mode explanation against quality picks—only if explicitly labeled |
| `BASE-PRIOR-APPROVED` | Prior approved basket for same intent (if exists) |
| `BASE-NONE` | No savings line displayed |

\[
savings\_cents = max(0,\ baseline\_total\_cents - basket\_total\_cents)
\]

Forbidden: inventing “regular price” not present in authorized data; using “up to” marketing claims.

## 8. Uncertainty handling

| Flag | Effect on optimize | Effect on approval |
| --- | --- | --- |
| `fresh` | Normal | None |
| `stale` | Allowed with flag | Require ack (HITL-004) if any line used |
| `expired` | Excluded from candidates | N/A |
| `unknown` | Excluded from auto-select unless only option → HITL | Ack required |
| `eligibility_unknown` | Loyalty price not applied | Info only |

## 9. Worked examples

### Example A — Unit price (budget)

Given authorized offers (illustrative **fixture numbers**, not live market data):

| Variant | Size | Price |
| --- | --- | --- |
| Oats A | 1 kg | $4.00 |
| Oats B | 500 g | $2.50 |

Base unit g: A = 400¢/1000g = 0.4¢/g; B = 250¢/500g = 0.5¢/g.  
Budget mode prefers A if equivalence holds and constraints pass.

### Example B — Loyalty not eligible

Shelf $6.00; loyalty $4.00 but `eligibility=unknown`.  
Basket must use $6.00 (or exclude). UI may say loyalty $4.00 if eligible—not as current price.

### Example C — Hard constraint blocks substitute

User: `peanut_free`. Cheapest sauce contains peanut tag → filtered. Next safe candidate selected or gap.

### Example D — Savings with baseline

Chosen promo total $20.00; `BASE-SINGLE-SHELF` for same variants $26.00.  
`savings_cents = 600`; definition stored. Without shelf baseline in data → `BASE-NONE`.

## 10. Deterministic test cases

| ID | Setup | Expect |
| --- | --- | --- |
| OPT-TC-001 | Two equiv variants; known sizes; budget mode | Lower unit price wins |
| OPT-TC-002 | Missing net_quantity on one | No unit compare; documented fallback rule |
| OPT-TC-003 | Loyalty price + ineligible | Loyalty not in total |
| OPT-TC-004 | Loyalty price + eligible | May apply per rules |
| OPT-TC-005 | Hard allergen vs cheaper item | Cheaper excluded |
| OPT-TC-006 | Brand lock | Locked variant kept |
| OPT-TC-007 | Budget hard cap infeasible | Feasible=false; gaps listed |
| OPT-TC-008 | Savings without baseline fields | `savings_cents` null; no claim |
| OPT-TC-009 | Stale offer only candidate | Included only with uncertainty flag |
| OPT-TC-010 | Same inputs twice | Identical totals (`algorithm_version` fixed) |
| OPT-TC-011 | Multi-store vs single | Suggestion flagged for HITL when threshold crossed |
| OPT-TC-012 | Quality mode | Preferred brand wins over cheaper equiv |

Map to AC-030–036 and AC-070+.

## 11. Open questions

| ID | Question |
| --- | --- |
| OPT-OPEN-001 | Final λ and multi-store thresholds after user testing |
| OPT-OPEN-002 | Need modeling (quantity to buy) from “running out” utterances |
| OPT-OPEN-003 | Tax/deposit handling by province |
