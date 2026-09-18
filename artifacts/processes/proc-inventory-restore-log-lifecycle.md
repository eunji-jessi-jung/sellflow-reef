---
id: "PROC-INVENTORY-RESTORE-LOG-LIFECYCLE"
type: "process"
title: "RESTORE_LOG Row Lifecycle (the table that has none)"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-traced on 2026-09-19 after a correction pass in inventory-api, by reading both Alembic revision files in full, alembic/env.py, alembic.ini, requirements.txt and app/models.py, then grepping all five sellflow repos for RESTORE_LOG, RestoreLog and restore_log — seven hits, all definitional, none a write. The table now has six columns and two indexes and a chain that resolves from a base; what it still has is no rows and no writer, which is the whole of the remaining finding. Goes stale the moment env.py gains a real runner or any INSERT INTO RESTORE_LOG is written."
freshness_triggers:
  - "alembic.ini"
  - "alembic/env.py"
  - "alembic/versions/20230414_1120-3f9a_add_restore_log.py"
  - "alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
  - "app/main.py"
  - "app/models.py"
known_unknowns:
  - "Whether 3f9a was ever applied in any environment. There is no alembic_version dump, no deployment log and no CI workflow in the repo to check against, and alembic/env.py defines no runner to apply it with."
  - "Whether a RESTORE_LOG table exists in any live database. The DDL is definite now, but nothing reads or writes the table, so no behaviour would reveal its presence or absence."
  - "Why app/models.py's RestoreLog dataclass was never reconciled with the DDL. It names sku_cd and qty where the table has SKU and SURYANG, adds a result field with no column, and omits LOG_SEQ and REG_DTM."
  - "What a restore record should hold beyond the six columns. The DDL has no warehouse code and no affected-row count, both of which the restock handler's known failure modes turn on."
  - "Who, if anyone, was asked for an audit trail of restocks. No ticket, minute or policy document in sources/context mentions 복원 이력."
tags:
  - inventory
  - restore-log
  - audit
  - migrations
  - absence-of-evidence
  - sellflow
aliases:
  - "복원 이력"
  - "RESTORE_LOG"
relates_to:
  - type: "depends_on"
    target: "[[API-INVENTORY]]"
  - type: "feeds"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "integrates_with"
    target: "[[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]]"
  - type: "feeds"
    target: "[[RISK-INVENTORY]]"
  - type: "feeds"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic.ini"
    notes: "sqlalchemy.url points at the inventory DB, not the one app/db.py uses."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/env.py"
    notes: "Two lines; no config, no target_metadata, no run_migrations_online."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
    notes: "Base revision (down_revision None); creates RESTORE_LOG with six columns and IX_RESTORE_LOG_01 on ORD_NO."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
    notes: "Creates and drops IX_RESTORE_LOG_02 on SAYU_CD."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "The only DB access layer; connects to sellflow_order."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "The restock handler, whose only trace is a log.info line."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
    notes: "The unused RestoreLog dataclass, whose fields no longer match the table's DDL."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
    notes: "The other, unrelated migration system; stock_item.updated_at has no ON UPDATE."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "Reef-relative. 재고팀 owns 재고 복원."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Reef-relative. Declares inventory-api's db as MySQL (inventory)."
notes: ""
---

# RESTORE_LOG Row Lifecycle (the table that has none)

## Purpose

`RESTORE_LOG` is the only audit surface `inventory-api` ever attempted. Its name says what it was
for: a durable record of which order restored which quantity of which SKU, for what reason, with
what result — exactly the questions anyone reconciling a cancellation would ask.

This artifact documents its lifecycle, and the lifecycle is empty. The table is now fully defined
— six columns, a surrogate key, an index on the order number and another on the reason code — and
not one row can exist, because no code in any of the five repositories inserts into it. That is
not a gap in the research; it is the finding, and it is sharper than it used to be. When the
schema was shapeless, the missing audit trail could be read as a design that was never finished.
It is now a design that was finished and never wired up.

## Key Facts

- The table is introduced by Alembic revision `3f9a` (`20230414_1120-3f9a_add_restore_log.py`, Create Date 2023-04-14 11:20:11) with six columns: `LOG_SEQ` (`sa.BigInteger`, `primary_key=True`, `autoincrement=True`), `ORD_NO` (`sa.String(20)`, `nullable=False`), `SKU` (`sa.String(30)`, `nullable=False`), `SURYANG` (`sa.Integer`, `nullable=False`), `SAYU_CD` (`sa.String(2)`, `nullable=False`) and `REG_DTM` (`sa.DateTime`, `server_default=sa.func.now()`) → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py
- The same revision creates `IX_RESTORE_LOG_01` on `ORD_NO`, and its `downgrade()` drops the table → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py
- `3f9a` is the base of the chain (`down_revision = None`), and its docstring says why: "stock_item 은 alembic 도입 이전에 sql/V1__stock.sql 로 만들었다. alembic 은 RESTORE_LOG 부터 적용한다." ("stock_item was made with sql/V1__stock.sql before Alembic was introduced; Alembic applies from RESTORE_LOG onward") → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py
- The revision imports what it uses — `import sqlalchemy as sa` and `from alembic import op` — and `requirements.txt` pins both `sqlalchemy==2.0.23` and `alembic==1.12.1` → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:requirements.txt
- The follow-up revision `8ba1` (`20240902_0931-8ba1_add_restore_log_reason_i.py`, Create Date 2024-09-02 09:31:44, `down_revision = "3f9a"`) creates `IX_RESTORE_LOG_02` on `SAYU_CD` and drops it on downgrade — the reason index its truncated filename names → inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py
- Both indexed columns are the two an auditor would filter on: `ORD_NO` to answer "was this order's stock returned", `SAYU_CD` to answer "how much did reason 01 restore last month". Neither query has any rows to run against → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py
- The column vocabulary is mixed: `ORD_NO`, `SURYANG` and `SAYU_CD` are the order domain's romanised names, while `SKU` is the inventory domain's, so one table spans both naming systems → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:sql/V1__stock.sql
- The table has **no warehouse column and no affected-row count**, so even a populated `RESTORE_LOG` could not distinguish the over-restock and no-op failure modes that [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]] documents → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:sql/V1__stock.sql
- `alembic/env.py` is two lines — a `from alembic import context  # noqa: F401` and a Korean comment, "표준 alembic env. 프로젝트 설정은 alembic.ini 참조." ("standard alembic env; see alembic.ini for project configuration") — with no `target_metadata`, no `run_migrations_online()` and no `run_migrations_offline()`, so the migration runner has no entry point → inventory-api:alembic/env.py
- `alembic.ini` sets `sqlalchemy.url = mysql://inventory:@localhost:3306/inventory`, while `app/db.py` hardcodes `database="sellflow_order"` with user `sellflow` — the migration and the application target different databases, so even a successful `3f9a` would create the table where the service never looks → inventory-api:alembic.ini, inventory-api:app/db.py
- **No code writes `RESTORE_LOG`.** A grep across all five repos for `RESTORE_LOG`, `RestoreLog` and `restore_log` returns seven hits, every one of them definitional: the `class RestoreLog` dataclass in `app/models.py`, the docstring line, the `create_table` / `create_index` / `drop_table` calls in `3f9a`, and the `create_index` / `drop_index` calls in `8ba1`. There is no `INSERT`, no ORM session, no repository and no writer of any kind → inventory-api:app/models.py, inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py
- **No code reads it either.** `app/db.py` exposes only `fetch_one` and `execute`, and the only two SQL literals in the service name `ORDER_DTL` and `stock_item` → inventory-api:app/db.py, inventory-api:app/main.py
- The unused dataclass `RestoreLog(ord_no: str, sku_cd: str, qty: int, sayu_cd: str, result: str)` no longer describes the table: it matches on `sayu_cd`/`SAYU_CD` and `ord_no`/`ORD_NO` (case aside), renames `SKU` to `sku_cd` and `SURYANG` to `qty`, declares a `result` field with no column, and omits `LOG_SEQ` and `REG_DTM`. It is imported by nothing → inventory-api:app/models.py, inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py
- `result` is the one field the dataclass has and the table does not — the outcome of the attempt, which is exactly what the discarded row count from `execute` would have supplied → inventory-api:app/models.py, inventory-api:app/db.py
- The only trace a restock leaves is a Python log line, `log.info("재고 복원 완료. ord_no=%s", req.ord_no)` ("stock restoration complete"), and its non-restocking counterpart `log.info("복원 대상 아님. ord_no=%s, reason=%s", ...)` ("not subject to restoration") → inventory-api:app/main.py
- Neither log line records the SKU, the quantity or the number of rows affected, so even the application log cannot reconstruct what moved → inventory-api:app/main.py
- The table name is upper-case and underscore-separated, matching the legacy order-domain convention rather than `inventory-api`'s own lower-case English style (`stock_item`, `available_qty`) — a naming fossil from before the 2022 transfer to 데이터플랫폼본부 → inventory-api:sql/V1__stock.sql, sellflow-docs:context/org-chart.md
- The 재고팀 (Inventory Team) is listed as owning the 재고 복원 (stock restoration) process, so the missing audit trail belongs to a team that has an explicitly assigned process → sellflow-docs:context/org-chart.md

## How the absence was established

Absence claims are only as good as the search behind them, so the method is recorded here.

1. **Name sweep across every repo.** From the source root, a recursive grep for
   `RESTORE_LOG\|RestoreLog\|restore_log` over all of `repos/order-service`,
   `repos/settlement-batch`, `repos/inventory-api`, `repos/delivery-bff` and
   `repos/settlement-anomaly` returned seven lines, all inside `inventory-api`:
   `app/models.py:14` (the dataclass declaration), one docstring line and three DDL calls in the
   `3f9a` revision, and two index calls in `8ba1`. Nothing outside that one service mentions the
   table.

2. **Writer sweep.** The service's only data-access surface is `app/db.py`, which exposes exactly
   two functions, `fetch_one` and `execute`. Every call site of either lives in `app/main.py`, and
   the SQL passed to them names `ORDER_DTL` (SELECT) and `stock_item` (SELECT, UPDATE). There is
   no third call site and no ORM, so there is no path by which a row could be written.

3. **Chain check.** `3f9a` declares `down_revision = None` and `8ba1` declares
   `down_revision = "3f9a"`, so the two files form a complete chain from base to head. A grep for
   `1c22` — the phantom base this artifact previously recorded — now returns nothing anywhere in
   the workspace.

4. **Runner check.** `alembic/env.py` was read in full: two lines, one of them a comment. A real
   Alembic `env.py` defines `run_migrations_online()` and/or `run_migrations_offline()` and
   invokes one of them at module scope; this one defines neither.

What this cannot establish is the state of any live database. Source code can prove that nothing
in these repos writes the table; it cannot prove that a table does not exist in production, having
been created by hand. That question is in `known_unknowns`.

## Fields

Six, all defined by revision `3f9a`:

| Column | Type | Constraints | Index | Purpose |
|---|---|---|---|---|
| `LOG_SEQ` | `BigInteger` | primary key, autoincrement | (PK) | Surrogate key; the only autoincrement key in this service |
| `ORD_NO` | `String(20)` | `NOT NULL` | `IX_RESTORE_LOG_01` | The cancelled order number. Same width as `ORDER_DTL.ORD_NO` |
| `SKU` | `String(30)` | `NOT NULL` | — | The product. Same width as `stock_item.sku`; the order domain's `SANGPUM_CD` |
| `SURYANG` | `Integer` | `NOT NULL` | — | Quantity restored, under the order domain's name rather than `qty` |
| `SAYU_CD` | `String(2)` | `NOT NULL` | `IX_RESTORE_LOG_02` (revision `8ba1`) | The cancel reason code, `01`–`04` |
| `REG_DTM` | `DateTime` | `server_default now()` | — | When the row was written — the timestamp `stock_item.updated_at` fails to provide |

Read as an audit record this is most of what is needed: it answers "which order restored how much
of what, why, and when". Two things it does not carry are worth naming, because both correspond to
documented failure modes of the write it would record. There is no `warehouse_cd`, although
`stock_item`'s key is `(sku, warehouse_cd)` and the restock `UPDATE` credits every warehouse row
for a SKU; and there is no affected-row count, although `execute` returns one and the handler
discards it, so a restock against an unknown SKU is indistinguishable from a successful one. The
unused `RestoreLog` dataclass has a `result` field that would have covered the second; the DDL does
not (inventory-api:app/models.py, inventory-api:app/db.py, inventory-api:sql/V1__stock.sql).

## Relationships

| Related entity | Link | State |
|---|---|---|
| `stock_item` | the mutation `RESTORE_LOG` would record | Live and mutated; see [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]]. `RESTORE_LOG.SKU` and `stock_item.sku` are the same concept at the same width, with no FK between them. |
| `ORDER_DTL` / `ORDER_CANCEL` (order-service) | `ord_no`, `sayu_cd` would key back to them | Live, read cross-database by the restock handler. |
| `sql/V1__stock.sql` | the other migration system | Independent by design, and `3f9a`'s docstring says so: `stock_item` predates Alembic and came from this file, so the Alembic chain starts at `RESTORE_LOG` and never mentions it. |

## Creation Path

The migrations themselves are sound: `3f9a` is a valid base revision that imports what it uses and
emits real DDL, `8ba1` chains from it, and `requirements.txt` declares both `alembic` and
`sqlalchemy`. Two obstacles sit between those files and a table:

- **The runner does not exist.** `alembic/env.py` is two lines — an import and a comment — with no
  `run_migrations_online()`, no `run_migrations_offline()` and no `target_metadata`, so there is no
  entry point under which the revisions would execute.
- **The target is the wrong database.** `alembic.ini` sets
  `sqlalchemy.url = mysql://inventory:@localhost:3306/inventory`, while `app/db.py` hardcodes
  `database="sellflow_order"`. A successful migration would create the table where the service
  never looks. The registry's claim that inventory-api's db is "MySQL (inventory)" matches the
  migration config and not the code (sellflow-docs:context/registry/services.yaml,
  inventory-api:app/db.py).

Neither obstacle is about `RESTORE_LOG` specifically; both are about the repository never having
had a working migration step at all.

## States and Transitions

A row of `RESTORE_LOG` has no states because no row can exist. The *table definition*, however,
has a history worth stating plainly:

| Date | Event | Net effect on the table |
|---|---|---|
| 2023-04-14 | revision `3f9a`, "add restore log" — the base of the chain | Defines `RESTORE_LOG` with six columns and `IX_RESTORE_LOG_01` on `ORD_NO`. |
| 2024-09-02 | revision `8ba1`, "add restore log reason index" | Adds `IX_RESTORE_LOG_02` on `SAYU_CD`. |
| 2026-09-19 | this reading | Six columns, two indexes, zero writers, zero readers. |

The 2024 revision is the more telling of the two. Seventeen months after the table was defined,
somebody came back and indexed it by reason code — a change that only makes sense if someone
expected to ask "how much did reason *n* restore". Between the two visits, and in the two years
since, nobody wrote the `INSERT` that would have produced a row to index.

## Worked Examples

### Auditing a restock today

Suppose a partner disputes a September cancellation: they claim two units of `SKU-1001` were never
returned to sellable stock. The natural query is:

```sql
-- the query an auditor would want to run, written against the real columns
SELECT LOG_SEQ, ORD_NO, SKU, SURYANG, SAYU_CD, REG_DTM
  FROM RESTORE_LOG
 WHERE ORD_NO = '2026090100123';   -- served by IX_RESTORE_LOG_01
-- Empty set. The columns are right, the index is right, and no row has ever
-- been written: nothing in any of the five repos issues an INSERT.
```

What is actually available instead:

```sql
-- the current state of the row, with no history attached
SELECT sku, warehouse_cd, available_qty, reserved_qty, updated_at
  FROM stock_item
 WHERE sku = 'SKU-1001';
```

This returns a number, not an event. It cannot say whether the number includes a restock, when it
changed, or which order caused it — and `updated_at` will not help, because `stock_item.updated_at`
has `DEFAULT CURRENT_TIMESTAMP` with no `ON UPDATE` clause, so it does not move when
`available_qty` does (inventory-api:sql/V1__stock.sql).

The remaining evidence is the application log:

```
INFO app.main: 재고 복원 완료. ord_no=2026090100123
```

That line proves a request for that order passed the reason-code gate and reached the end of the
handler. It does not record the SKU, the quantity, the warehouse, or how many rows the `UPDATE`
actually changed — which, per
[[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]], may have been zero. And log retention is not configured
anywhere in the repo; `app/config.py` sets only `LOG_LEVEL`.

### Reconstructing the answer, and where it fails

An auditor's fallback chain, in order of decreasing reliability:

1. `ORDER_CANCEL.CHWISO_SAYU_CD` for the order — establishes whether the reason code was one of
   `01`/`02` and therefore whether a restock *should* have been attempted.
2. The application log — establishes whether a request arrived and which branch it took, for as
   long as the log is retained.
3. `stock_item.available_qty` — a current total that cannot be decomposed into the events that
   produced it.

Step 2 is the only link between intent and effect, and it is a text line in an unretained log. The
chain has no step that records what the database actually did. That is precisely the gap
`RESTORE_LOG` was named to fill.

## Agent Guidance

- **Never suggest querying `RESTORE_LOG` for history.** The schema is correct and the table has no
  writer, so any query returns an empty set. If asked for restock history, say that none is
  recorded and name the three fallbacks above.
- **Do not treat the Alembic chain as evidence the table was created.** The revisions are valid,
  but `alembic/env.py` provides no runner and `alembic.ini` names a database the service never
  opens. `sql/V1__stock.sql` is still the file that describes what the service actually uses.
- **Distinguish "not written by this code" from "does not exist".** Everything here is established
  from source. Whether `RESTORE_LOG` sits in some database is unanswerable from the repo and is
  listed in `known_unknowns`.
- **When asked how to add an audit trail, the answer is one INSERT, not a schema.** The restock
  handler already holds every value the table needs — `ord_no` and `reason_code` from the request,
  `SANGPUM_CD` and `SURYANG` from the `ORDER_DTL` read. Note that the table has no warehouse column
  and no row-count column, so the over-restock and no-op failure modes would still go unrecorded;
  the unused dataclass's `result` field is the closest anyone came to covering the second.

## Related

- [[API-INVENTORY]] — the endpoints whose effects would have been logged here
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — `RESTORE_LOG`, `ord_no`, `sayu_cd`: the naming fossil
- [[PROC-INVENTORY-RESTOCK]] — the process that produces no audit record
- [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]] — the row whose mutations go unrecorded
- [[RISK-INVENTORY]] — the missing audit trail as a standing risk
- [[RISK-SELLFLOW-DOC-DRIFT]] — an audit table whose documentation and dataclass disagree with its DDL
- [[SCH-INVENTORY]] — the two-migration-system split and the ER diagram
- [[SYS-INVENTORY]] — the owning service and team
