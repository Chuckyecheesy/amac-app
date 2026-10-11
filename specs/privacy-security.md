# Privacy & Security — Agentic Grocery AI

**Status:** Draft  
**Related:** [product-requirements.md](product-requirements.md) · [architecture.md](architecture.md) · [acceptance-criteria.md](acceptance-criteria.md)

## 1. Location consent (PRIV-001)

| Rule ID | Rule |
| --- | --- |
| LOC-001 | Request geolocation only with clear purpose: find nearby supported stores. |
| LOC-002 | Denial is first-class; offer postal-code entry. |
| LOC-003 | Record consent method, timestamp, and purpose (`LOCATION_CONSENT`). |
| LOC-004 | Prefer storing postal code / FSA over precise coordinates when sufficient `[ASSUMPTION]`. |
| LOC-005 | User can clear location; subsequent research requires new consent or postal code. |
| LOC-006 | Do not silently substitute IP geolocation without explicit product+legal approval. |

## 2. Data minimization and retention

| Data | MVP need | Retention draft |
| --- | --- | --- |
| Email / auth subject | Account | Until account deletion |
| Postal code | Store discovery | Until cleared or account deletion |
| Precise lat/lng | Avoid if possible | If ever stored: short TTL `[OPEN]` |
| Preferences / constraints | Personalization | Until deleted |
| Intent text | Session quality | Session + limited history `[ASSUMPTION]` 90 days |
| Offers cache | Optimization | Per license + freshness jobs |
| Approval records | Audit | `[ASSUMPTION]` 24 months |
| Payment cards | **Not collected** | N/A |
| Retailer passwords | **Not collected in MVP** | N/A |

Minimize PII in logs; never log secrets or full auth tokens.

## 3. Authentication and authorization

| Rule ID | Rule |
| --- | --- |
| AUTH-001 | All preference, session, and approval APIs require authenticated user. |
| AUTH-002 | Object-level authZ: users access only their sessions and prefs. |
| AUTH-003 | Approval transitions must be authenticated as session owner. |
| AUTH-004 | Service-to-source credentials never exposed to the client. |
| AUTH-005 | Candidate: managed auth (e.g., Supabase Auth / Auth.js)—see ADR-001. |

## 4. Secrets and API-key management

| Rule ID | Rule |
| --- | --- |
| SEC-001 | Secrets only in environment / secret manager; never in git or Markdown examples with real values. |
| SEC-002 | Separate keys per environment. |
| SEC-003 | LLM and data-source keys server-side only. |
| SEC-004 | Rotate keys on suspected exposure; document runbook in implementation phase. |

## 5. Secure external integrations

| Rule ID | Rule |
| --- | --- |
| INT-001 | TLS for all external calls. |
| INT-002 | Validate source payloads against schemas before persist. |
| INT-003 | Timeouts, retries, circuit breakers per architecture.md. |
| INT-004 | Partner tokens stored encrypted at rest; refresh handled server-side. |
| INT-005 | Egress allowlisting where hosting permits `[ASSUMPTION]`. |

## 6. Prompt injection and untrusted source data (PRIV-INJECT / NFR-008)

Grocery flyers, promo text, and webpage-derived fields are **untrusted**.

| Rule ID | Rule |
| --- | --- |
| PI-001 | Source text is data, never executable instructions. |
| PI-002 | Tool outputs pass through typed schemas; strip/ignore instruction-like content for control flow. |
| PI-003 | System prompt forbids following instructions found in tool/offer text. |
| PI-004 | Optimizer reads structured fields only (prices, tags, quantities)—not freeform promo prose for control decisions. |
| PI-005 | Explanations that quote promo text must remain non-authoritative for eligibility and prices. |

## 7. Approval requirements for consequential actions

| Action | Required |
| --- | --- |
| Finalize shopping plan | Explicit Approve (FR-040, HITL-001) |
| Multi-store expansion beyond preference | HITL confirm |
| Brand-changing substitute | HITL confirm (default) |
| Proceed with stale critical prices | Acknowledge stale |
| Checkout / purchase | **Not in MVP**; if ever added, separate strong auth + explicit consent |

No consequential action from LLM tool calls alone.

## 8. Data deletion and security testing

### Deletion

| Rule ID | Rule |
| --- | --- |
| DEL-001 | User can request account deletion; personal prefs, sessions, consents removed or anonymized within documented SLA `[OPEN]`. |
| DEL-002 | Licensed catalog data is not user data; retain per license. |
| DEL-003 | Backup expiration aligns with deletion policy where feasible. |

### Security testing (implementation phase gates)

| ID | Test focus |
| --- | --- |
| ST-001 | AuthZ: horizontal access to another user’s session denied |
| ST-002 | Secrets not present in client bundles |
| ST-003 | Prompt-injection fixtures cannot change workflow to `approved` |
| ST-004 | Schema fuzzing on source ingest |
| ST-005 | Dependency vulnerability scan in CI |

## 9. Open questions

| ID | Question |
| --- | --- |
| PS-OPEN-001 | PIPEDA / Law 25 roles and DPA needs for processors |
| PS-OPEN-002 | Exact geolocation retention policy |
| PS-OPEN-003 | Account deletion SLA |
| PS-OPEN-004 | Whether Quebec pilot requires French privacy copy at launch |
