---
id: "RISK-INVENTORY"
type: "risk"
title: "Inventory API Known Issues"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Systematic re-scan on 2026-09-19 of all 16 files in inventory-api, plus the caller side in order-service and the three documents that describe this service: services.yaml, business-rules.md and the 2021 Confluence cancellation-policy page. Re-verified the same day after a correction pass in inventory-api: the three Alembic defects (missing base revision, unimported op, column-less and empty migrations) are fixed and are struck from the findings table, and requirements.txt now declares sqlalchemy and alembic. The findings that remain are the behavioural ones — no auth, discarded row counts, the duplicated reason set, the missing warehouse predicate, and an audit table with a schema and no writer. Every finding is a present-tense reading of the code, not a runtime observation."
freshness_triggers:
  - ".env.sample"
  - "alembic.ini"
  - "alembic/env.py"
  - "alembic/versions/20230414_1120-3f9a_add_restore_log.py"
  - "alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
  - "app/config.py"
  - "app/db.py"
  - "app/main.py"
  - "app/models.py"
  - "app/routers.py"
  - "sql/V1__stock.sql"
  - "tests/test_restore.py"
severity: "high"
resolution: "open"
known_unknowns:
  - "Whether stock is corrected on cancellation by some other system. Nothing in this reef shows one, and the order side's only inventory client has no caller, but the absence of a caller does not prove the absence of a mechanism."
  - "Whether reserved_qty is written by anything at all. No code path in this repository touches it; the column exists only in sql/V1__stock.sql and in the GET response."
  - "Whether the restock endpoint has ever been exercised in production. Its only written client posts to a different path, so any traffic it has received came from somewhere not visible in these repositories."
  - "Whether a separate inventory database exists as .env.sample, alembic.ini and services.yaml all claim, and what is in it if so."
  - "Whether RESTORE_LOG has ever been created in a deployed database. The revision that creates it is now runnable and has a base, but alembic/env.py is still a stub with no run_migrations_online, so nothing in the repo provides a runner."
  - "Whether the deployed MySQL account for user 'sellflow' has a password. The DSN in app/db.py has no password key, so pymysql sends an empty one."
  - "Who decided that reason 04 does not restock, and when. The code has a comment justifying only 03; no ticket, minute or decision record in sellflow-reef/sources mentions 04."
tags:
  - dead-code
  - inventory
  - risk
  - security
  - tech-debt
aliases:
  - "inventory-api risks"
  - "재고 API 리스크"
relates_to:
  - type: "constrains"
    target: "[[API-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-AUTH]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-ERROR-HANDLING]]"
  - type: "constrains"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "integrates_with"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:.env.sample"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic.ini"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/env.py"
    notes: "Two lines; no run_migrations_online, no target_metadata."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet — reason 04 배송 실패 is marked 재고 복원 O."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Records db: MySQL (inventory) for a service that connects to sellflow_order."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "2021 policy page naming inventory-api as the restock processor; reason 04 listed without a no-restock note."
notes: "Eighteen findings in 174 lines of code. The recurring shape is duplication in which the inert copy is the discoverable one — two restock rules, two data models, two routers, two databases, two migration systems; in each pair only one side is live, and the tests, the config file and the registry all point at the dead side."
---

## Description

inventory-api is 16 files and 174 lines. That smallness is what makes this artifact unusual: the
findings below are not a sample, they are the complete set, and they outnumber the service's
modules. The recurring shape is duplication in which the *inert* copy is the one a reader finds
first — the restock rule is declared twice and the tests assert the dead copy; the database is
named in three configuration files and the code ignores all three; a router is written and never
mounted; a dataclass module describes tables with different field names than the SQL. Around that
sits a second, sharper problem: the one endpoint that mutates stock has no authentication, no
audit table, no row-count check and no idempotency, and its only written client calls a path that
does not exist.

## Key Facts

- Every authentication, credential and transport-security token is absent: a case-insensitive grep over the whole tree for `auth|token|jwt|oauth|api[_-]?key|secret|credential|bearer|password|hmac|signature|login|Depends|middleware|security` returns zero lines, exit status 1 → inventory-api (whole tree)
- The alembic chain is runnable and rooted: `3f9a` declares `down_revision = None` and imports both `op` and `sqlalchemy`, and `8ba1` chains from it → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py
- `alembic/env.py` is still a two-line stub — one import annotated `# noqa: F401` and one comment — with no `run_migrations_online`, so the repository supplies no migration runner for those revisions → inventory-api:alembic/env.py
- `RESTORE_LOG` is created with six columns (`LOG_SEQ`, `ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD`, `REG_DTM`) and two indexes, and **still has no writer**: no INSERT exists in any of the five repos → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:app/main.py
- `app/models.py`'s `RestoreLog` dataclass does not match the table it is named after: it declares `sku_cd` and `qty` where the DDL has `SKU` and `SURYANG`, adds a `result` field with no column, and omits `LOG_SEQ` and `REG_DTM` → inventory-api:app/models.py, inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py
- `RESTOCKABLE_REASONS = {"01", "02"}` is declared twice, in `app/config.py` and again in `app/main.py`; only the `main.py` copy governs the endpoint → inventory-api:app/main.py, inventory-api:app/config.py
- The test suite asserts against the inert copy — `from app.config import RESTOCKABLE_REASONS` — so changing the live rule leaves both tests green → inventory-api:tests/test_restore.py
- `app/config.py` has no other importer anywhere in the service, which makes `DB_URL` and `LOG_LEVEL` dead alongside it → inventory-api:app/config.py, inventory-api:app/main.py
- The live rule diverges from the 취소정책 sheet for reason `04` (배송 실패 / delivery failure), which the sheet marks 재고 복원 `O` (restock: yes) with the note 물류팀 확인 후 처리 ("handled after 물류팀 confirmation"), while the code's set contains only `01` and `02` → sellflow-docs:context/business-rules.md, inventory-api:app/main.py
- The code comment justifies only `03`: "03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다" ("03, customer change of mind, is often cancelled after delivery has started, so it is not a restock target") — `04`'s exclusion is undocumented in code, ticket or minute → inventory-api:app/main.py
- `app/db.py` hardcodes `"database": "sellflow_order"` and builds its DSN from `DB_HOST` and `DB_USER`, ignoring the `DB_URL` that `app/config.py`, `.env.sample` and `alembic.ini` all set to `mysql://inventory:@localhost:3306/inventory` → inventory-api:app/db.py, inventory-api:.env.sample, inventory-api:alembic.ini
- `services.yaml`, which opens by declaring itself 단일 기준 ("the single standard"), records `db: MySQL (inventory)` for this service → sellflow-docs:context/registry/services.yaml
- The DSN has no `password` key at all, so pymysql authenticates as user `sellflow` with an empty password → inventory-api:app/db.py
- The restock handler reads the order domain's table directly: `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s` → inventory-api:app/main.py
- `execute` returns the affected row count and the handler discards it, so zero rows updated is reported as `{"restocked": true}` → inventory-api:app/db.py, inventory-api:app/main.py
- `fetch_one` returns a single row, so a multi-line order restocks one SKU and silently ignores the rest → inventory-api:app/main.py
- The `UPDATE` omits `warehouse_cd` although the primary key is `(sku, warehouse_cd)`, so one restock credits every warehouse row for that SKU → inventory-api:sql/V1__stock.sql, inventory-api:app/main.py
- `app/routers.py` defines `/inventory/stock/{sku_cd}` and `/inventory/health` and is never included in the application, so neither is served — including the service's only health endpoint → inventory-api:app/routers.py, inventory-api:app/main.py
- The dataclasses in `app/models.py` use field names the SQL does not have (`sku_cd` vs `sku`, `qty` vs `available_qty`) and are imported by nothing → inventory-api:app/models.py, inventory-api:sql/V1__stock.sql
- The only client written against this service posts to `/inventory/restore?ordNo=..&reason=..`, a path no mounted or unmounted router serves — and `grep -rn "InventoryClient" src` in order-service matches only the class's own declaration, so nothing calls even that → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java

## Findings

| # | Finding | Evidence | Class | Severity | Detectable today? |
|---|---|---|---|---|---|
| 1 | No authentication or authorization of any kind; `POST /stock/restock` mutates stock for any caller who can reach port 8000 | whole-tree grep returns zero hits; `app/main.py` has no `Depends`, no middleware | security | **high** | No — nothing logs a caller identity |
| 2 | The unauthenticated endpoint reads another service's table, `ORDER_DTL`, over a connection to `sellflow_order` | `app/main.py`, `app/db.py` | security / coupling | **high** | No |
| 3 | Row count from `execute` is discarded, so an unknown SKU returns `{"restocked": true}` after changing nothing | `app/db.py` returns `n`; `app/main.py` assigns it to nothing | correctness | **high** | No — success and no-op are byte-identical |
| 4 | Duplicated `RESTOCKABLE_REASONS`, with the test suite asserting the dead copy in `app/config.py` | `app/main.py`, `app/config.py`, `tests/test_restore.py` | correctness / testing | **high** | No — the tests stay green when the rule changes |
| 5 | Reason `04` (배송 실패) does not restock in code; the 취소정책 sheet marks it 재고 복원 `O` | `sellflow-docs:context/business-rules.md`, `app/main.py` | business-rule drift | **high** | Only by manual reconciliation |
| 6 | Hardcoded DSN points at `sellflow_order` while `app/config.py`, `.env.sample`, `alembic.ini` and `services.yaml` all say `inventory` | `app/db.py` vs four documents | configuration | **high** | No — the service works, against the wrong database |
| 7 | The DSN carries no password key; pymysql authenticates with an empty password as the generic user `sellflow` | `app/db.py` | security | **high** | No |
| 8 | `alembic/env.py` is a stub with no `run_migrations_online` and no `target_metadata`, so the repository provides no runner for its own revisions | `alembic/env.py` | schema management | **medium** | Yes — `alembic upgrade` has nothing to execute under |
| 9 | `alembic.ini` points at the `inventory` database while `app/db.py` opens `sellflow_order`, so a migration and the application would not target the same schema | `alembic.ini`, `app/db.py` | schema management | **medium** | No |
| 10 | `app/models.py`'s `RestoreLog` dataclass contradicts the `RESTORE_LOG` DDL field for field | `app/models.py`, `alembic/versions/…3f9a…py` | dead code / drift | **low** | No |
| 11 | Nothing writes `RESTORE_LOG`, so no audit trail of restock decisions exists | `app/main.py` (no INSERT anywhere) | audit | **high** | No — the absence is the point |
| 12 | `UPDATE` omits `warehouse_cd` although the PK is `(sku, warehouse_cd)`; one restock credits every warehouse | `sql/V1__stock.sql`, `app/main.py` | correctness | **high** | No |
| 13 | `fetch_one` returns one row, so multi-line orders restock one SKU silently | `app/main.py` | correctness | **high** | No |
| 14 | `app/routers.py` never mounted — the service's only health endpoint is unreachable | `app/routers.py`, `app/main.py` | operability | **medium** | Yes, on first probe attempt |
| 15 | `app/models.py` field names contradict the SQL and the module is imported by nothing | `app/models.py`, `sql/V1__stock.sql` | dead code | **low** | No |
| 16 | The only written client calls `/inventory/restore`, which is not served — and has no caller itself | `order-service:…/InventoryClient.java` | integration | **high** | Yes — a 404 per call, if it were ever called |
| 17 | No try/except, no exception handler, no rollback anywhere; every non-404 failure is a bare `500` | `app/main.py`, `app/db.py` | operability | **medium** | Partially — uvicorn logs the traceback |
| 18 | The application logger is never configured and `LOG_LEVEL` is read by nothing, so both `log.info` lines are dropped under uvicorn's default config | `app/main.py`, `app/config.py`, `Dockerfile` | observability | **medium** | No |

## Impact

**Silent stock errors, in both directions.** Findings 3, 12 and 13 are all failures that report
success. A multi-line order returns one SKU to stock and drops the others (under-restock); the
one it does return is credited to every warehouse row for that SKU (over-restock); and if the SKU
is not in `stock_item` at all, nothing happens and the response is still `{"restocked": true}`.
No log line, no metric and no table distinguishes any of these from a correct restock. Physical
stock and system stock diverge with no signal.

**A rule nobody can test, already out of step.** The live constant and the tested constant are
different objects (finding 4), so the suite provides zero protection for the one business rule
this service exists to enforce. That rule is already divergent: the 취소정책 sheet marks reason
`04` 배송 실패 (delivery failure, partner-borne, 물류팀 확인 후 처리) as 재고 복원 `O`, and the
code does not restock it. Every cancellation for a failed delivery therefore leaves stock
unreturned, with the goods physically back at the warehouse. Whether the code or the sheet is
wrong is a business question — see `.reef/questions-for-owner.md` — but one of them is, and no
mechanism exists that would ever surface the disagreement.

**No audit, and now for only one reason.** `RESTORE_LOG` has the columns the question needs —
`ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD`, `REG_DTM`, indexed on order number and on reason — and
nothing writes a row (finding 11). The README compounds it by saying the inventory domain does
not keep `ORD_NO` at all — 주문 도메인의 `ORD_NO` ↔ 재고 도메인 (보관하지 않음), "not stored" —
which the table itself now contradicts. So the question "was this cancellation's stock returned?"
still cannot be answered from inventory data, but the remedy is a single INSERT rather than a
schema design. That is precisely the question a finance reviewer or a partner
raises, and it is the same class of question the settlement side cannot answer about its own
backlog (see [[RISK-SETTLEMENT-RECON-BACKLOG]]).

**An unauthenticated write into the order database.** Findings 1, 2 and 7 compound rather than
add. The endpoint is reachable without a credential, it mutates stock, it reads the order
domain's `ORDER_DTL`, and the connection it does all this on is a passwordless generic account on
the shared `sellflow_order` instance. Neither a caller identity nor a row count is recorded, so
abuse and malfunction look identical, and both look like success.

**Configuration that documents a system that does not exist.** Three files in the repository and
the company's self-declared 단일 기준 registry all describe an `inventory` database on a
`mysql://inventory:@...` DSN. The code connects to `sellflow_order` as `sellflow`. Any engineer
sizing a migration, granting privileges, or reasoning about blast radius from the documents will
be reasoning about the wrong database — which also means any privilege-scoping exercise performed
against the documented account would have no effect on the real one.

**Boundary invisibility.** Reading `ORDER_DTL` directly (finding 2) means schema changes in
order-service land here without notice and without a contract. This estate has already had one
production break of exactly that kind in a different consumer — see [[RISK-SELLFLOW-DOC-DRIFT]]
and [[CON-ORDER-INVENTORY]].

## Severity and Resolution

**Severity: high.** Raised from medium at this depth, and the justification is density rather
than any single defect.

Eighteen findings sit in 174 lines, of which the two live modules — `app/main.py` at 55 lines and
`app/db.py` at 22 — carry eleven of them. That is roughly one defect per seven lines of running
code. Density matters here for a specific reason, not as a metric for its own sake: when a
service is this small, the defects are not independent. The same 30-line handler is the one that
lacks authentication, discards the row count, reads another domain's table, ignores the composite
key, restocks one line of many, enforces a rule contradicted by the policy sheet, and writes no
audit row. Every one of those failure modes returns the same HTTP 200 `{"restocked": true}`, so
they cannot be distinguished from each other or from success — and the service's only durable
record, `RESTORE_LOG`, is uncolumned, unmigratable and unwritten.

The severity claim is therefore: *this service cannot be observed to be wrong*. Six of the
eighteen findings are silent-failure findings, five are audit or observability findings, and the
two tests that exist assert against a constant the runtime does not read. There is no
configuration of production events under which a stakeholder would learn that restock is broken,
short of a physical stock count. Combined with an unauthenticated write into the shared order
database on a passwordless account, that clears the bar for high.

Two things argue the other way and are recorded honestly. The blast radius may currently be zero:
the endpoint has no working caller — `InventoryClient` posts to a path that is not served, and
nothing in order-service calls `InventoryClient` — so it is possible that no restock has ever
happened through this API. And the assessment is code-reading only: no traffic data, error rate,
or stock-discrepancy report was available. But "possibly never invoked" is not mitigation for a
service whose stated purpose is stock restoration; it is the most serious finding of the set,
because it means the compensating action the cancellation policy promises may not exist at all.

**Resolution: open, and untracked.** No ticket in `sellflow-reef/sources` references any of these
items — not the auth gap, not the reason-04 divergence, not the migration chain. That is a
different position from the settlement backlog, which is at least measured
([[RISK-SETTLEMENT-RECON-BACKLOG]]). Here nothing has been measured.

## Recommended Actions

Ordered so that each step makes the next one possible.

1. **Determine whether restock happens at all today, by any route.** Query `stock_item` for
   `updated_at` values and compare against cancellation volume. Until this is answered every other
   item is speculative in its impact. This is the cheapest question and the one that reframes
   everything else.
2. **Stop the silent successes.** Have the restock handler read `execute`'s return value and
   return `{"restocked": false, "reason": "no_rows_matched"}` when it is zero. Two lines, and it
   converts three invisible failure modes into visible ones.
3. **Fix the write itself:** `fetchall` instead of `fetch_one` for multi-line orders, and either
   include `warehouse_cd` in the `WHERE` clause or decide explicitly which warehouse a restock
   credits.
4. **Delete one of the two `RESTOCKABLE_REASONS` and repoint the tests at the survivor.** The
   duplication is the reason the rule is untestable; removing it is a prerequisite for item 5
   meaning anything.
5. **Settle reason `04` with the 취소정책 owner.** The sheet says restock, the code says not, and
   nothing records a decision. Whichever way it goes, record it as a decision artifact.
6. **Reconcile the DSN with the four documents that disagree with it** — and while doing so,
   establish whether the `sellflow` account has a password.
7. **Finish the alembic setup.** The revisions are sound now — a rooted chain, real DDL, both
   indexes — but `env.py` is still a stub with no runner and `alembic.ini` still names a different
   database from the one the application opens. One of those two has to move before `alembic
   upgrade head` means anything.
8. **Write the audit row.** `RESTORE_LOG` has exactly the columns the restock handler already has
   in hand — order number, SKU, quantity, reason code — and an index on each of the two fields
   anyone would query by. The only missing piece is the INSERT.
9. **Mount `app/routers.py` or delete it** — the health endpoint in particular, since its absence
   means no liveness probe target exists.
10. **Configure logging**, so that the two `log.info` lines this service already writes actually
    appear, and add the SKU, the quantity and the row count to them.

## Related

- [[API-INVENTORY]] — the mounted and unmounted routers, and the response shapes
- [[CON-ORDER-INVENTORY]] — the caller contract that does not connect
- [[PROC-INVENTORY-AUTH]] — the verified absence of authentication
- [[PROC-INVENTORY-ERROR-HANDLING]] — the full failure matrix behind findings 3, 17 and 18
- [[PROC-INVENTORY-RESTOCK]] — the rule and its two declarations
- [[RISK-SELLFLOW-DOC-DRIFT]] — the spreadsheet-versus-code divergence in estate context
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measured backlog this unmeasured one sits beside
- [[SCH-INVENTORY]] — `stock_item`'s composite key and the uncolumned `RESTORE_LOG`
- [[SYS-INVENTORY]] — the service these issues belong to
