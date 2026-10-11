# Agentic Grocery AI

Human-in-the-loop agentic grocery assistant for Canadian households. The product helps users restock groceries, discover real deals from authorized sources, and optimize shopping baskets—while keeping consequential decisions under explicit user approval.

> **Development stage:** Specification only. No application implementation has started in this documentation phase.
>
> **Repository note:** This workspace also contains a pre-existing Next.js school application (`amac-app`). That application code is untouched. The previous root README is preserved at [docs/legacy/amac-app-readme.md](docs/legacy/amac-app-readme.md).

## Goals

- Help individuals and families restock household groceries with clear trade-offs (budget, quality, or balance).
- Research nearby stores and promotions from **authorized** data sources only.
- Optimize baskets with **deterministic, testable** price and savings calculations.
- Keep a human in the loop: clarify preferences, surface uncertainty, and require approval before finalizing a plan.
- Launch as a lean Canada-wide web MVP with phased retailer/geographic coverage.

## Non-goals (MVP)

- Autonomous checkout or purchasing.
- Multi-agent swarms or unconstrained agent autonomy.
- Scraping or commercial reuse of public web pages without verified rights.
- Claiming coupon/loyalty prices without established eligibility.
- Fabricating prices, coverage, or licensing status.

## Current stage

| Stage | Status |
| --- | --- |
| Product & technical specification | **In progress (this PR)** |
| Data-source feasibility validation | Pending (see open questions) |
| Technical proof of concept | Not started |
| Application implementation | Not started |

## Specification index

| Document | Purpose |
| --- | --- |
| [specs/product-requirements.md](specs/product-requirements.md) | Problem, vision, personas, requirements, MVP scope, metrics |
| [specs/user-journeys.md](specs/user-journeys.md) | End-to-end flows and decision points |
| [specs/architecture.md](specs/architecture.md) | System design, orchestration, deployment |
| [specs/data-sources.md](specs/data-sources.md) | Canadian grocery sources, licensing, freshness |
| [specs/data-model.md](specs/data-model.md) | Entities, relationships, indexes |
| [specs/optimization.md](specs/optimization.md) | Basket modes, normalization, savings, test cases |
| [specs/privacy-security.md](specs/privacy-security.md) | Consent, minimization, auth, prompt injection |
| [specs/acceptance-criteria.md](specs/acceptance-criteria.md) | Testable MVP criteria linked to requirements |
| [specs/roadmap.md](specs/roadmap.md) | Phases, gates, dependencies |
| [adr/001-initial-stack.md](adr/001-initial-stack.md) | Proposed stack and trade-offs |
| [adr/002-grocery-data-strategy.md](adr/002-grocery-data-strategy.md) | Data strategy ADR |
| [docs/research/data-source-matrix.md](docs/research/data-source-matrix.md) | Candidate source research matrix |

## Critical principles

1. Do not fabricate grocery prices, retailer coverage, API capabilities, or licensing permissions.
2. Distinguish **verified facts**, **estimates**, **assumptions**, and **open questions**.
3. Never present a coupon or loyalty price as available unless eligibility is established.
4. Never calculate savings without a documented comparison baseline.
5. Never substitute products that violate hard dietary or household constraints.
6. Require explicit user approval before finalizing a shopping plan.
7. Keep secrets out of source code and documentation.

## Label legend

Used across specifications:

| Label | Meaning |
| --- | --- |
| `[FACT]` | Verified or product-decision fact for this project |
| `[ESTIMATE]` | Approximate; needs measurement |
| `[ASSUMPTION]` | Working assumption until validated |
| `[OPEN]` | Unresolved; blocks or risks delivery if ignored |

## License / rights

Grocery data usage rights are **not claimed** in this repository. See [specs/data-sources.md](specs/data-sources.md) and [adr/002-grocery-data-strategy.md](adr/002-grocery-data-strategy.md).
