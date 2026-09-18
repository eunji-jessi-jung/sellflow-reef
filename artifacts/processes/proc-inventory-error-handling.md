---
id: "PROC-INVENTORY-ERROR-HANDLING"
type: "process"
title: "Inventory API Error Handling and Failure Modes"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Traced on 2026-09-19 line by line through app/main.py and app/db.py, with a grep for 'try|except|raise|rollback|commit|exception_handler|status_code' across the repository's Python — which returns exactly four lines: two HTTPException raises and one conn.commit(), plus no except, no try, no rollback and no exception handler. Re-verified on 2026-09-19 after a correction pass in inventory-api: the Alembic chain is now rooted and RESTORE_LOG has real columns and two indexes, so the audit-trail finding narrows from three defects to one — nothing writes the table. No error-handling code changed. Stale if any try/except, exception handler, or connection-management change lands."
freshness_triggers:
  - "alembic/env.py"
  - "app/config.py"
  - "app/db.py"
  - "app/main.py"
  - "requirements.txt"
  - "sql/V1__stock.sql"
known_unknowns:
  - "Whether uvicorn's logging configuration is overridden at deploy time. No deployment manifest or entrypoint script exists in the repo beyond the Dockerfile's bare CMD, so whether the application logger's records reach anywhere cannot be settled from the source."
  - "Whether MySQL's own error log or slow-query log is collected. Nothing in the repository configures or references log shipping."
  - "The actual pymysql failure mode under connection exhaustion. Each call opens a new connection with no pool and no max, so behaviour past max_connections depends on server configuration that is not in the repo."
  - "Whether any external monitor watches the 5xx rate of this service. No alert rule, dashboard, runbook or SLO document naming inventory-api was found anywhere in sellflow-reef/sources."
  - "Whether RESTORE_LOG has columns in the deployed database. The migration that creates it declares none, and the follow-up revision's upgrade() is a bare pass, so the deployed shape cannot be inferred from the repository."
  - "Whether the alembic chain has ever been applied. It is rooted and valid now, but alembic/env.py is a two-line stub with no run_migrations_online, so the repository supplies no runner for it."
tags:
  - inventory
  - error-handling
  - operations
  - reliability
  - sellflow
aliases:
  - "inventory error handling"
  - "재고 API 오류 처리"
relates_to:
  - type: "refines"
    target: "[[API-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-AUTH]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "constrains"
    target: "[[RISK-INVENTORY]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:Dockerfile"
    notes: "Bare uvicorn CMD; no entrypoint script, so uvicorn's default logging config applies."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
    notes: "Base revision; creates RESTORE_LOG with six columns and an index on ORD_NO."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
    notes: "upgrade() and downgrade() are both bare pass."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "Defines LOG_LEVEL; imported by no application module."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "Two helpers, one connection each, one explicit commit, no rollback and no except."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "The two handlers and their only two failure branches."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:requirements.txt"
    notes: "fastapi 0.104.1, uvicorn 0.24.0, pymysql 1.1.0 — the versions whose default behaviour applies."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
    notes: "PRIMARY KEY (sku, warehouse_cd) — the key the UPDATE ignores."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "The caller that discards the response entity — and that nothing in order-service calls."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet; the four reason codes a caller can legitimately send."
notes: "The governing observation is that this service has exactly two error branches, both 404, and every other failure is either a 500 with no body or a success response that is not one."
---

# Inventory API Error Handling and Failure Modes

## Purpose

To answer, for an on-call engineer or an agent reasoning about a restock that did not happen:
what does inventory-api do when something goes wrong, and can the caller tell? The short answer
is that the service has two error branches, both `404`, and that everything else resolves to one
of two outcomes — an unstructured `500`, or a `200` that says `{"restocked": true}` whether or
not anything was restocked.

## Key Facts

- The whole repository contains no `try`, no `except` and no `rollback`. A grep for `try|except|raise|rollback|commit|exception_handler|status_code` across `*.py` returns four lines in total: `conn.commit()` in `app/db.py`, and two `raise HTTPException(404, ...)` in `app/main.py` → inventory-api:app/db.py, inventory-api:app/main.py
- No `@app.exception_handler` is registered, so only FastAPI's built-in handlers for `HTTPException` and `RequestValidationError` are active; any other exception propagates to Starlette's `ServerErrorMiddleware` and becomes a bare `500 Internal Server Error` with a plain-text body → inventory-api:app/main.py
- On an unknown order number, `fetch_one` returns `None` and the handler raises `HTTPException(404, "주문 상세 없음")` ("order detail not found") — this is the one failure the caller can identify unambiguously → inventory-api:app/main.py
- On an **unknown or absent SKU**, nothing fails: `execute` returns `cur.execute`'s row count, the handler assigns it to nothing, and the response is `{"restocked": True}` regardless → inventory-api:app/main.py, inventory-api:app/db.py
- `execute` is the only function in the service that knows how many rows changed, and it is the only information the handler throws away → inventory-api:app/db.py
- On a **malformed or unrecognised reason code** — `"99"`, `""`, `"1"`, `"abc"` — the handler takes the same branch as the deliberate policy decline for `"03"` (고객 변심, customer change of mind) and returns HTTP 200 `{"restocked": False, "reason": "not_restockable"}`; the reason code has no `pattern`, `Literal` or enum constraint on the Pydantic model → inventory-api:app/main.py
- A *structurally* invalid body (missing `ord_no`, `reason_code` as a number) is the one input error handled well: Pydantic rejects it and FastAPI returns `422` with a field-level detail list → inventory-api:app/main.py
- The read and the write are **two separate transactions on two separate connections**: `fetch_one` and `execute` each call `pymysql.connect(**DSN)` independently, so nothing holds `ORDER_DTL` and `stock_item` consistent across the operation → inventory-api:app/db.py
- `execute` commits explicitly with `conn.commit()`; this is load-bearing, because pymysql 1.1.0's `Connection.__exit__` closes the connection rather than committing, and autocommit is off by default → inventory-api:app/db.py, inventory-api:requirements.txt
- If `conn.commit()` itself raises, there is no `rollback()` and no handler: the `with` block closes the connection, the server discards the transaction, and the caller receives a `500` → inventory-api:app/db.py
- The operation is not idempotent and carries no dedupe key: the same `{ord_no, reason_code}` replayed *n* times adds `SURYANG` to `available_qty` *n* times, because the `UPDATE` is a relative increment with no guard → inventory-api:app/main.py
- `fetch_one` returns a single row, so a multi-line order restocks one SKU and silently ignores the rest — no error, no warning, same `{"restocked": true}` → inventory-api:app/main.py
- The `UPDATE` omits `warehouse_cd` although the primary key is `(sku, warehouse_cd)`, so one restock credits the quantity to **every** warehouse row for that SKU — an over-restock that also returns `{"restocked": true}` → inventory-api:app/main.py, inventory-api:sql/V1__stock.sql
- The logger is obtained with `logging.getLogger(__name__)` and never configured: there is no `basicConfig`, no `dictConfig`, and `LOG_LEVEL` in `app/config.py` is read by nothing, since no application module imports `app/config.py` at all → inventory-api:app/main.py, inventory-api:app/config.py
- Under the Dockerfile's bare `uvicorn app.main:app` command, uvicorn's default logging configuration sets up the `uvicorn`, `uvicorn.error` and `uvicorn.access` loggers only and leaves the root logger without handlers, so records from the `app.main` logger fall through to Python's last-resort handler, which emits `WARNING` and above — both of this service's log calls are `log.info` → inventory-api:Dockerfile, inventory-api:app/main.py
- No durable record of any restock decision exists, for one reason only: `RESTORE_LOG` is created with `LOG_SEQ`, `ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD` and `REG_DTM` and indexed on `ORD_NO` and `SAYU_CD`, and **no code in the service ever writes the table** → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py, inventory-api:app/main.py
- The one client written against this service discards the outcome entirely — `restTemplate.postForEntity(url, null, Void.class)` with the returned `ResponseEntity` unassigned and unchecked — and that client is itself called from nowhere in order-service (`grep -rn "InventoryClient" src` matches only its own declaration) → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java

## Scope

This artifact covers the two mounted HTTP handlers in `app/main.py` — `POST /stock/restock` and
`GET /stock/{sku}` — and the database layer they run on, `app/db.py`. The unmounted router in
`app/routers.py` is excluded because it serves no traffic; its `/inventory/health` endpoint is
the only "health" construct in the service and it is not reachable, which is itself recorded in
[[RISK-INVENTORY]].

Out of scope: what the caller *should* do on failure (see [[CON-ORDER-INVENTORY]]), and the
question of who may call at all (see [[PROC-INVENTORY-AUTH]]).

## Current State

### The complete failure matrix

| Condition | Where it is detected | HTTP status | Body | Can the caller tell? |
|---|---|---|---|---|
| Body missing a field, or wrong type | Pydantic, before the handler | 422 | field-level `detail` list | **Yes** — the one well-handled case |
| `reason_code` `"03"` (policy: no restock) | `RESTOCKABLE_REASONS` check | 200 | `{"restocked": false, "reason": "not_restockable"}` | Only as "not restockable" |
| `reason_code` `"04"` (배송 실패 / delivery failure) | same branch | 200 | identical to `"03"` | **No** — and the sheet says 04 *should* restock |
| `reason_code` garbage (`"99"`, `""`, `"abc"`) | same branch | 200 | identical to `"03"` | **No** — a typo looks like a policy decision |
| Order number unknown | `if not items` | 404 | `{"detail": "주문 상세 없음"}` | **Yes** |
| Order has several lines | not detected | 200 | `{"restocked": true}` | **No** — lines 2..n silently skipped |
| SKU absent from `stock_item` | not detected (0 rows updated) | 200 | `{"restocked": true}` | **No** |
| SKU present in several warehouses | not detected | 200 | `{"restocked": true}` | **No** — every warehouse credited |
| Request replayed / retried | not detected | 200 | `{"restocked": true}` | **No** — stock inflated again |
| DB unreachable, auth refused, table missing | nowhere | 500 | `Internal Server Error` (plain text) | Only that *something* broke |
| Read succeeds, write fails | nowhere | 500 | `Internal Server Error` | **No** — cannot distinguish from read failure |
| Commit fails | nowhere | 500 | `Internal Server Error` | **No** |
| SKU lookup miss on `GET /stock/{sku}` | `if not row` | 404 | `{"detail": "SKU 없음"}` | **Yes** |

Read down the "Can the caller tell?" column: four cases out of thirteen. The service's two most
consequential silent failures — zero rows updated, and the same row updated twice — are both
reported as success.

### On a database failure

There is no code path that contemplates one. `pymysql.connect(**DSN)` is called inside
`fetch_one` and again inside `execute`, both bare. An `OperationalError` — host unreachable,
credentials refused (the DSN carries no password at all), `max_connections` exceeded, table
missing — propagates out of the helper, out of the handler, and into Starlette's
`ServerErrorMiddleware`, which re-raises after producing a plain `500 Internal Server Error`.
uvicorn logs the traceback on its own `uvicorn.error` logger, so the stack trace *is* visible in
container output even though the application's own `log.info` lines are not.

Three consequences follow. The caller gets no machine-readable error code, no `detail` and no
correlation id, so it cannot retry selectively. There is no connection pool and no timeout on
`connect`, so a slow or saturated database converts into request pile-up rather than fast
failure. And because the read and the write are separate connections, a failure between them
leaves the operation half-done with no compensating action — though in this specific handler the
read is harmless and the write is the only mutation, so "half-done" means "not done", which is
the better of the two possible directions.

### On transactions and commits

`execute` is the only writer:

```python
def execute(sql, params=None):
    with pymysql.connect(**DSN) as conn:
        with conn.cursor() as cur:
            n = cur.execute(sql, params or {})
        conn.commit()
        return n
```

The commit is explicit and correct for pymysql 1.1.0, whose connection context manager closes
rather than commits. But the function is a single-statement transaction by construction: there
is no way to group the `ORDER_DTL` read with the `stock_item` write, and no caller of `execute`
could batch two statements atomically even if it wanted to. `n` — the row count, the single
piece of evidence that would let the handler distinguish "restocked" from "matched nothing" — is
returned faithfully by this function and dropped by its only caller.

### On logging

Two `log.info` calls exist, one per branch of the restock handler:

- `log.info("복원 대상 아님. ord_no=%s, reason=%s", req.ord_no, req.reason_code)` — "not eligible for restoration"
- `log.info("재고 복원 완료. ord_no=%s", req.ord_no)` — "stock restoration complete"

Neither carries a caller identity, a SKU, a quantity, a row count or a trace id — so even if both
were emitted, they would not answer "was this order's stock returned, and by how much?". And in
the image as built they are probably not emitted: the Dockerfile's `CMD` invokes uvicorn with no
`--log-config`, uvicorn's default `LOGGING_CONFIG` configures only the `uvicorn*` loggers, and
the root logger is left with no handler, so records from the `app.main` logger below `WARNING`
are dropped by Python's last-resort handler. The `LOG_LEVEL` variable that `.env.sample`
advertises is defined in `app/config.py` and read by nothing.

The intended durable record was `RESTORE_LOG`, and it is now fully specified. Its 2023-04-14
migration is the base of the chain and creates the table with `LOG_SEQ`, `ORD_NO`, `SKU`,
`SURYANG`, `SAYU_CD` and `REG_DTM` plus an index on `ORD_NO`; the 2024-09-02 follow-up adds the
reason index on `SAYU_CD`. Every field the failure modes above would need to be diagnosed — which
order, which SKU, how many units, under what reason, at what time — has a column waiting for it.
Nothing inserts a row. The audit gap is no longer a schema problem; it is a missing `INSERT` in
the restock handler, which currently logs and returns.

### What the caller does with all this

Nothing. `InventoryClient.restore` in order-service issues the POST and discards the
`ResponseEntity<Void>` without inspecting its status. A 404, a 422 and a 200 with
`{"restocked": false}` are indistinguishable to it — and since `RestTemplate` raises
`HttpClientErrorException` on 4xx by default, a 404 would surface as an unhandled exception
rather than as a business outcome. The question is academic today: `grep -rn "InventoryClient"
src` in order-service matches only the class's own declaration, so nothing calls it, and the path
it would call (`/inventory/restore`) is not served by this API in any case.

## Related

- [[API-INVENTORY]] — the response shapes described above, as an interface contract
- [[CON-ORDER-INVENTORY]] — the caller that discards every one of these outcomes
- [[PROC-INVENTORY-AUTH]] — who can trigger these failure paths (anyone who can reach the port)
- [[PROC-INVENTORY-RESTOCK]] — the happy path this artifact is the shadow of
- [[RISK-INVENTORY]] — the same defects ranked and scored
- [[SCH-INVENTORY]] — `stock_item`'s composite key and the uncolumned `RESTORE_LOG`
- [[SYS-INVENTORY]] — the service these handlers belong to
