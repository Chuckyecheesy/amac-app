# ADR-001 — Initial Technical Stack

**Status:** Proposed (pending validation)  
**Date:** 2026-10-11  
**Related:** [specs/architecture.md](../specs/architecture.md) · [specs/roadmap.md](../specs/roadmap.md)

## Context

We need a lean MVP web application: authenticated users, workflow-based restock sessions, one LLM orchestrator with tools, deterministic optimization in application code, PostgreSQL-shaped persistence, and low ops cost. No implementation in the documentation phase.

## Decision drivers

- Minimize infrastructure and API cost (NFR-002).
- Strong TypeScript ergonomics for shared types (API, tools, optimizer).
- Managed auth and DB acceptable for speed.
- Deterministic optimizer must be easy to unit test.
- Canada-friendly hosting options preferred but not mandatory if unavailable `[ASSUMPTION]`.

## Proposed stack (candidate → recommendation)

| Layer | Candidate | Recommendation | Rationale |
| --- | --- | --- | --- |
| Frontend | Next.js + TypeScript | **Adopt (App Router or Pages—choose at scaffold)** | Web-first MVP; one deployable with API routes |
| UI styling | Lightweight CSS modules / Tailwind | Defer exact choice to implementation | Not blocking architecture |
| Backend | Modular server code in Next.js route handlers + domain modules | **Adopt for MVP** | Fewest moving parts |
| Database | PostgreSQL via Supabase | **Adopt candidate** | Managed Postgres, auth, RLS fit multi-tenant prefs |
| Auth | Supabase Auth or Auth.js | **Prefer Supabase Auth if using Supabase** | Fewer vendors |
| AI | One managed LLM with tool calling | **Adopt** | Matches single-orchestrator design |
| Optimizer | Pure TypeScript (or SQL functions for totals) | **Pure TS modules + tests** | Determinism and CI simplicity |
| Jobs | Hosting cron / Supabase scheduled functions | **Lightweight cron** | Feed refresh without always-on workers |
| Hosting | Vercel / Fly / Render + managed DB | **Pick at PoC based on cost/region** | Avoid lock-in in docs |
| Observability | Structured logs + OpenTelemetry-capable exporter | **Minimal structured logs first** | Cost control |

### Recommendation summary

**Yes — proceed with Next.js/TypeScript + PostgreSQL (Supabase candidate) + single tool-calling LLM + in-process deterministic optimizer + managed hosting**, unless PoC shows hard blockers (e.g., residency, cold-start, or DB policy limits).

## Alternatives considered

| Alternative | Pros | Cons | Verdict |
| --- | --- | --- | --- |
| Separate FastAPI/Go API + React SPA | Clear isolation | More deployables, higher ops | Reject for MVP |
| MongoDB | Flexible docs for offers | Weaker relational constraints for money/approvals | Reject |
| Multi-agent framework | Parallel research agents | Complexity, cost, harder HITL control | Reject until justified |
| LLM does basket math | Faster to prototype | Non-deterministic money errors | Reject |
| Fully self-hosted GPU + DB | Control | Cost/ops | Reject for MVP |
| Firebase | Fast auth | Relational/optimizer fit weaker | Hold |

## Costs and trade-offs

| Trade-off | Detail |
| --- | --- |
| Co-located Next.js API | Simple, but long-running source pulls may need early extraction to jobs |
| Supabase | Speed vs vendor coupling; mitigate via standard Postgres SQL |
| Single LLM vendor | Quality depends on one provider; abstract behind interface |
| Cron freshness | Not real-time; honesty via stale flags |
| TypeScript optimizer | Enough for MVP; may rewrite hotspots later if needed |

`[ESTIMATE]` Cost drivers: LLM tokens per session, managed DB, cron pull volume, hosting bandwidth. Exact numbers require vendor quotes after traffic assumptions.

## Decisions that still need validation

| ID | Decision | Validation |
| --- | --- | --- |
| STACK-V-001 | Supabase vs other Postgres host | Residency, RLS, cost, auth fit |
| STACK-V-002 | LLM vendor and model | Tool calling reliability on grocery fixtures; Canadian data processing terms |
| STACK-V-003 | Hosting region | Latency to users; data processing agreements |
| STACK-V-004 | App Router vs Pages | Team familiarity; existing repo uses Pages (legacy school app)—greenfield grocery app may still choose App Router when implementation starts |
| STACK-V-005 | Job runner | Cron limits vs queue |

## Consequences

- Implementation should not introduce Kafka, service meshes, or agent swarms in MVP.
- Existing `amac-app` code in this repository is **out of scope** for the grocery MVP; either migrate to a clean app directory later or split repositories `[OPEN]`.
- All money paths covered by automated tests before pilot (NFR-004).

## Open questions

- Should grocery MVP live in a new package/app within this monorepo or a new repository?
- Is French UI required before Quebec users are invited?
