# sellflow-reef — Knowledge Reef

Structured knowledge base for 셀플로우 (sellflow), a Korean e-commerce fulfilment company: five services covering ordering and cancellation, daily partner settlement, settlement anomaly detection, inventory, and delivery lookup.

## What this is

A Reef — interlinked markdown artifacts extracted from source code and from the company's own documents (wiki export, tickets, procedures, meeting minutes, a handover, an incident postmortem, Slack and mail threads, sprint records, a data export and two spreadsheets). Every claim cites a source file. Artifacts record what is known, what is partly known, and what is explicitly unknown.

Artifacts are written in English. Korean source text is quoted verbatim with the English meaning alongside — never silently translated.

## Services

- **Order** (Order & Cancellation) — customer ordering and cancellation, and the outbox that tells settlement a cancel happened. Source: `order-service` (Java 8 / Spring Boot 2.3)
- **Settlement** (Partner Settlement) — daily partner payout batch, the cancel-reconciliation queue, and anomaly detection over settlement results. Sources: `settlement-batch` (Spring Batch / Quartz), `settlement-anomaly` (FastAPI / scikit-learn)
- **Inventory** — stock levels and restock-on-cancel, which depends on the cancel reason code. Source: `inventory-api` (FastAPI)
- **Delivery** (Delivery Lookup) — carrier status proxy for the app; ownership unsettled in the registry. Source: `delivery-bff` (Express / TypeScript)

## How to use this reef

**Before making cross-service changes**, read the contract artifacts:

- `artifacts/contracts/con-order-settlement.md` — the cancellation event path from Order to a settlement correction, and where it stops. The highest-value artifact in this reef.
- `artifacts/contracts/con-order-inventory.md` — the restock contract both sides describe and neither executes.
- `artifacts/contracts/con-order-delivery.md` — a generated client that froze a business rule repealed in 2023.

Three service pairs have no contract artifact — Inventory×Settlement, Delivery×Settlement, Delivery×Inventory. That is deliberate: a grep across all five repos found no HTTP call, shared schema, event or table reference between them.

**Before modifying a service**, read its system artifact: `artifacts/systems/sys-order.md`, `sys-settlement.md`, `sys-settlement-anomaly.md`, `sys-inventory.md`, `sys-delivery.md`.

**Before changing data models**, read `artifacts/schemas/` — five SCH- artifacts with field tables and Mermaid ER diagrams. Note that four of the five services share one MySQL database (`sellflow_order`); see `artifacts/decisions/dec-sellflow-shared-db.md`.

**Before changing business logic**, read `artifacts/processes/` (cancellation flow, settlement batch, settlement correction, restock, delivery sync, auth per service, runtime topology) and `artifacts/decisions/` (outbox-over-broker, shared database, money rounding).

**Before trusting any document**, read `artifacts/patterns/pat-sellflow-doc-code-drift.md` (how documents and code drift apart here) and `artifacts/patterns/pat-sellflow-orphaned-components.md` (what exists but is never invoked).

**Before reading any schema**, read `artifacts/patterns/pat-sellflow-romanised-naming.md`. Identifiers are Korean written in Latin letters (`SANGTAE_CD`, `CHWISO_SAYU_CD`), not English. `artifacts/glossary/glossary-sellflow.md` resolves the cross-service terms and `artifacts/glossary/glossary-source-index.md` maps each term to the file that defines it.

## Key risks and known unknowns

- `artifacts/risks/risk-settlement-recon-backlog.md` — 4,127 correction rows never processed, 188,851,520 KRW, oldest from 2023-04. The measured consequence of the Order↔Settlement contract stopping one step short.
- `artifacts/risks/risk-settlement.md` — unscheduled reconciler, hardcoded fee rate, the 2025-07 duplicate-run incident with two action items still open.
- `artifacts/risks/risk-sellflow-doc-drift.md` — every authoritative document describes behaviour the code no longer has; the 2026 automation plan is sized against a 2023 estimate the data contradicts by 14-16x.
- `artifacts/risks/risk-order.md`, `risk-inventory.md`, `risk-delivery.md` — dead code, disabled tests, broken caller contracts per service.
- No service authenticates anything in code. The order OpenAPI spec declares bearer JWT; no enforcement exists in any repo. See the four `proc-*-auth` artifacts.
- 533 known_unknowns are recorded across the 80 artifacts. The recurring themes: what runs in production versus what is in the repos, whether manual work happens outside the systems, and who owns the step after an event is published.

## Artifact counts

| Type | Count |
|---|---|
| SYS- | 5 |
| SCH- | 7 |
| API- | 5 |
| PROC- | 28 |
| DEC- | 9 |
| CON- | 5 |
| RISK- | 8 |
| GLOSSARY- | 6 |
| PAT- | 7 |
| **Total** | **80** |

All at `status: draft` (snorkel + scuba Phase 1 depth). Lint: 0 errors, 8 warnings (all `title_case`, on titles that carry literal identifiers such as `SETTLEMENT_DTL`, `stock_item` and `settlement-batch`, or Korean text — left as written).

## Sources

Five code repos under `repos/` in the sellflow source root, plus this reef's own `sources/` tree, which is indexed as `sellflow-docs` so document citations resolve. Citation forms: `<repo>:<path>` for code, `sellflow-docs:<path>` for documents.

The two spreadsheets are also kept as extracted Markdown (`sources/context/business-rules.md`, `sources/context/org-chart.md`) because the indexer skips binary files; the `.xlsx` originals remain authoritative.

## Freshness

Last updated 2026-09-19 by `/reef:update`, which re-indexed all five repos and refreshed 23 artifacts
against a correction pass in order-service and inventory-api. The headline changes: order-service's
Flyway chain V1-V24 now applies cleanly to an empty database (six migrations previously named columns
no migration created), `ORDER_DELIVERY` and `ORDER_STATUS_HIST` have DDL in V1, `OrderItem` maps
`ORDER_DTL`'s real columns, and inventory-api's Alembic chain is rooted and creates `RESTORE_LOG` with
real columns. None of those objects gained a reader or a writer, so the orphan findings stand — see
`artifacts/patterns/pat-sellflow-orphaned-components.md`. Four questions were raised (Q-049 to Q-052).

Before this, 2026-09-19 by `/reef:scuba` (Phase 1 automated deepening; the manifest's 67 planned items
are all complete, 0 skipped, 0 dropped). Run `/reef:update` to sync with code changes, `/reef:health`
for a status check, or `/reef:scuba` to deepen the drafts with domain knowledge the code cannot supply.
