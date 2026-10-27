# Project plan

## Goal and scope

Build an accessible shopping website that helps visitors discover suitable products through conventional browsing and an intelligent product-guidance agent. Phase 1 proves the complete journey using synthetic data and simulated checkout, not a production commerce launch.

Approved planning baseline: Next.js + TypeScript storefront/server, PostgreSQL, a guest cart, and a provider-neutral agent grounded in catalog data. Specific package versions, model provider, deployment host, and schedule remain implementation decisions. This plan provisions no cloud resources and makes no hosting or model-spend commitments.

## Roadmap

| Phase | Outcome | Scope | Exit gate |
|---|---|---|---|
| 1 - Demo MVP | A complete demonstration shopping journey | Seeded catalog, browsing, search, guest cart, simulated checkout, catalog-grounded product guidance | All Phase 1 issues and release acceptance criteria complete |
| 2 - Commerce readiness | A controlled transactional pilot | Identity, payment-provider sandbox then separately approved live payments, inventory reservation, fulfillment integration, administration, privacy and threat reviews | Reconciliation, payment and order integrity, operational and compliance readiness approved |
| 3 - Production and optimization | Operable, scalable commerce | Approved production deployment, monitoring/SLOs, load testing, accessibility audit, recovery drills, consent-based personalization and agent evaluation improvements | Release approvals, tested recovery, agreed performance/cost budgets and incident ownership |

Phase 2 and 3 are roadmap direction, not implementation commitments or Phase 1 issues. Estimate dates after the backlog is sized and owners/capacity are known.

## Phase 1 experience

1. A visitor browses sample products and opens product details with prices, currency, availability, and descriptions.
2. They search/filter/sort the catalog and see explicit loading, empty, and error states.
3. They add products to a guest cart, adjust quantities, and retain the cart across reloads.
4. They review authoritative server-calculated totals and confirm a simulated order; no payment details are requested and no real order is fulfilled.
5. They ask the agent to find or compare products. Its recommendations cite actual catalog products and disclose unknown information.

### Explicit exclusions

No live payments, shipping/tax calculation integrations, fulfillment, inventory reservation, merchant administration, account authentication, persistent chat history, personal-data collection, autonomous purchases, or production-readiness claims. Demo totals consist of product subtotals only; tax and shipping are explicitly not calculated.

## Architecture and data flow

| Component | Responsibility and boundary |
|---|---|
| Next.js storefront | Responsive catalog, product details, search/filter UI, guest cart, checkout and agent chat |
| TypeScript server layer | Validates input and exposes catalog, simulated-order, and agent operations; secrets stay server-side |
| PostgreSQL | Seeded products and demo orders, with order-line price/name snapshots and integer minor-unit monetary values |
| Guest cart | Versioned browser storage of product IDs and bounded quantities; never authoritative prices |
| Agent adapter | Provider-neutral model interface with deterministic test double; provider selected and documented during implementation |
| Catalog tools | Validated, read-only search and product-detail operations returning product IDs and canonical relative links |
| Runtime controls | Request and tool limits, timeouts, redacted structured logs, configuration validation and documented single-instance demo rate limiting |

The browser requests catalog data from the server, which reads PostgreSQL. At checkout, the browser submits product IDs/quantities and a scoped idempotency key. The server re-reads current products, calculates totals and atomically persists an order and line snapshots. A price/availability mismatch requires customer review rather than silently accepting changed totals. Repeated equivalent submissions reuse the same result; conflicting reuse of the key is rejected.

Chat requests go through a server-side model adapter. The agent can only call validated catalog tools, not arbitrary URLs, database queries, order mutations, or payment tools. Tool results are untrusted data, not instructions. Product facts in answers must map to returned catalog records, and unsupported claims must be declined or expressed as unknown. Recommendations link to real product detail pages; adding to cart remains a user action.

### Data and error contracts

- Products have stable IDs, name, description, category, price in minor units, one demo currency, demo availability, and safe local/approved image references.
- Orders are explicitly `simulated`, use an unguessable confirmation token, contain no payment/address/contact data, and expose no enumerable public order listing.
- Define pagination, maximum quantities, query/text limits, error codes, and money-display rules before dependent UI work.
- Reject malformed inputs with useful errors. Preserve cart contents when checkout fails; distinguish unavailable products, changed prices, invalid quantities, model timeout, rate limit, and database failure.
- Agent failures do not block browsing, cart, or checkout. Provide a clear retry/browse fallback, not invented answers.
- Local cart/chat data stays in the browser; chat is ephemeral and not persisted by the application. Document provider-side handling, demo-order retention, and any unavoidable infrastructure logs.

## Phase 1 work breakdown

IDs below are stable planning IDs, also included in GitHub issue titles. The tracking issue contains live links.

| ID | Deliverable | Prerequisites |
|---|---|---|
| P1-01 | Scaffold the app and document architecture/configuration contracts | None |
| P1-02 | Product/order schema, migrations, and deterministic synthetic seed | P1-01 |
| P1-03 | Validated, paginated catalog service/API | P1-02 |
| P1-04 | Accessible catalog and product-detail screens | P1-03 |
| P1-05 | Catalog search, category filters, sort and pagination | P1-03, P1-04 |
| P1-06 | Persistent guest cart and quantity handling | P1-03, P1-04 |
| P1-07 | Authoritative, idempotent simulated-checkout service | P1-02, P1-03, P1-06 |
| P1-08 | Demo checkout and confirmation experience | P1-06, P1-07 |
| P1-09 | Read-only catalog tools and provider-neutral agent contract | P1-01, P1-03 |
| P1-10 | Grounded product-guidance orchestration and model adapter | P1-09 |
| P1-11 | Accessible chat with verified product links | P1-04, P1-10 |
| P1-12 | Abuse controls, privacy-safe diagnostics, and agent guardrails | P1-07, P1-10 |
| P1-13 | CI checks and deterministic test harness | P1-01 |
| P1-14 | End-to-end journey and agent regression release gates | P1-05, P1-08, P1-11, P1-12, P1-13 |
| P1-15 | Developer runbook and demo release checklist | P1-12, P1-14 |

This dependency graph describes implementation sequencing, not tasks already completed. All issues start unassigned and open. No coding agent is assigned.

## Verification and release acceptance

Tests belong with their implementation issues; P1-13 supplies automation and P1-14 covers integrated behavior.

- A fresh checkout can be configured, migrated, seeded, and started from documented steps without live customer data.
- Browsing, product details, search/filter/sort, quantity changes, reload persistence, simulated checkout and confirmation work on mobile and desktop.
- Seed at least 24 clearly fictional products across four categories, with deterministic boundary cases.
- Prices and totals are integer-safe and server-authoritative. Malformed/tampered prices, invalid quantities, stale/unavailable products, conflicting idempotency keys and concurrent duplicate checkout are tested.
- Money/currency formatting is consistent. Checkout explicitly states that no payment, shipment, tax calculation or reservation occurs.
- Agent factual product claims cite valid catalog IDs/links. A versioned suite has at least 20 fixtures spanning discovery, comparison, no-match, ambiguity, malicious input and failure handling; all deterministic safety/grounding checks must pass. Real-model smoke evaluation records provider/model/date and human-reviewed failures separately; it is not represented as a deterministic guarantee.
- Prompt injection in user text and catalog records, unsupported product claims, tool misuse and attempts to order/pay are covered by explicit tests.
- All key journeys are keyboard-operable, with labeled inputs, useful focus behavior, screen-reader status announcements and no serious/critical automated accessibility violations on the tested screens.
- CI runs type checks, lint, unit/integration and end-to-end gates with deterministic model doubles and isolated test data. Pull requests do not require paid model credentials; real-model checks are separately opt-in.
- No secrets, raw chat prompts, personal information or payment data in source/logs. Limits, error paths, provider settings and operational caveats are documented.
- No release is marked complete while milestone issues, mandatory checks or accepted release-blocking defects remain open.

## Risks and decisions

| Risk or open decision | Mitigation / owner issue |
|---|---|
| Model cost, provider availability and data handling | P1-10 selects/documents a provider, server-only configuration and bounded calls; no resources purchased by this plan |
| Hallucination and catalog prompt injection | P1-09/P1-10/P1-12 restrict tools, validate citations, treat catalog text as data and exercise refusal/fallback cases |
| Price tampering, duplicate submissions and stale carts | P1-07 server recomputation, review-required mismatches, transactional persistence and idempotency |
| Public demo abuse | P1-12 explicit request/input/token/tool limits, timeouts and documented demo-only deployment limitations |
| Scope creep into production commerce | Keep simulation labels and exclusions visible; production features require Phase 2 approval |
| Hosting/provider/versions not chosen | P1-01 records dependency/runtime decisions; P1-10 records model decisions; P1-15 documents the selected demo execution path |
| Unestimated effort and capacity | Size and assign issues before committing dates; none are implied here |
