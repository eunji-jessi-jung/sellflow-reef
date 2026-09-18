---
id: "SYS-INVENTORY"
type: "system"
title: "Inventory API Service"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by re-reading the entire inventory-api repo file by file (all 16 files, including both Alembic revisions, alembic/env.py, sql/V1__stock.sql and both test modules), then reconciling against the tier-4 extractions at sources/apis/inventory/openapi.json and sources/schemas/inventory/schema.md, the 취소정책 sheet of business-rules, the service registry, the org chart and the 주문 취소 정책 Confluence snapshot. This pass resolved Q-030 (which database the service connects to) and Q-032 (authentication) from code. Re-verified on 2026-09-19 after a correction pass in inventory-api: the Alembic chain is now rooted and runnable, RESTORE_LOG is created with real columns and two indexes, and requirements.txt declares sqlalchemy and alembic — so Q-030 and Q-032 stand unchanged while the schema-management findings do not. Goes stale if app/db.py, app/main.py, requirements.txt or the registry entry for inventory-api changes, or if any deployment manifest is added to the repo."
freshness_triggers:
  - ".env.sample"
  - "Dockerfile"
  - "README.md"
  - "alembic.ini"
  - "alembic/versions/*.py"
  - "app/config.py"
  - "app/db.py"
  - "app/main.py"
  - "app/routers.py"
  - "requirements.txt"
  - "sql/V1__stock.sql"
known_unknowns:
  - "Whether the running deployment overrides DB_HOST/DB_USER so that the hardcoded database name 'sellflow_order' resolves to a different physical instance — no deployment manifest, Helm chart or compose file exists in the repo (verified by listing every file in it)."
  - "Whether a physical 'inventory' database still exists anywhere. .env.sample, alembic.ini and services.yaml all name one, but no code in the repo ever opens it."
  - "Whether the reserved_qty column is ever written. It is declared in sql/V1__stock.sql and returned by GET /stock/{sku}, but no code path in the repository writes it, and no other repo references it (checked by grepping inventory-api end to end)."
  - "How the service is exposed on the network (ingress, gateway, service mesh). Only the container port 8000 and the caller's default base URL http://inventory-api.internal are visible; neither is an enforced control."
  - "Whether the Alembic revisions have ever been run. The chain is rooted at 3f9a and both revisions are valid, but alembic/env.py is a two-line stub with no run_migrations_online and no target_metadata, so the repository contains no runner for them."
  - "Whether RESTORE_LOG exists in any physical database. Its DDL is definite now, but no code in any of the five repos writes a row, so the table's state cannot be inferred from behaviour."
  - "Whether the database user 'sellflow' used by app/db.py has been granted SELECT on ORDER_DTL deliberately or incidentally — no grant script, migration or contract in either repo records the decision."
  - "Whether any caller other than order-service reaches this service. No service in the workspace other than order-service references it, but nothing enumerates callers authoritatively."
  - "Why ownership moved to 데이터플랫폼본부 in 2022 — the org chart records the move (2022-03-01, '데이터플랫폼본부 신설, 재고팀 이관') but not the rationale."
  - "Who owns the 'exception' the routers.py docstring promises: '복원은 main.py 에 있다 (이관 예정, 2023부터)' ('restock lives in main.py; migration planned, from 2023'). No ticket, plan or commit in the available sources tracks that migration."
tags:
  - inventory
  - fastapi
  - python
  - sellflow
aliases:
  - "inventory-api"
  - "재고 API"
relates_to:
  - type: "refines"
    target: "[[API-INVENTORY]]"
  - type: "constrains"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[DEC-INVENTORY-RESTOCK-BY-REASON]]"
  - type: "refines"
    target: "[[GLOSSARY-INVENTORY]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-AUTH]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]]"
  - type: "refines"
    target: "[[RISK-INVENTORY]]"
  - type: "refines"
    target: "[[SCH-INVENTORY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:.env.sample"
    notes: "Declares DB_URL against a separate 'inventory' database; declares no credential of any kind."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:Dockerfile"
    notes: "python:3.9-slim, uvicorn on 0.0.0.0:8000, no proxy or auth layer in the image."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:README.md"
    notes: "Ownership, Python 3.9 / FastAPI / MySQL, the vocabulary bridge table, and the pointer to app/main.py for the restock rule."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic.ini"
    notes: "sqlalchemy.url repeats the 'inventory' database that no code opens."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/env.py"
    notes: "Two lines; imports context and defers to alembic.ini. No target_metadata."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
    notes: "Base revision (down_revision None); creates RESTORE_LOG with six columns and IX_RESTORE_LOG_01."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
    notes: "Creates and drops IX_RESTORE_LOG_02 on SAYU_CD."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "Second declaration of RESTOCKABLE_REASONS; DB_URL that app/db.py never reads."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "Hardcoded database 'sellflow_order'; DSN carries no password."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "Both mounted endpoints, the restock rule, and the cross-domain ORDER_DTL read."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
    notes: "Stock and RestoreLog dataclasses; neither is imported anywhere."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
    notes: "APIRouter with prefix /inventory, never included on the app."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:requirements.txt"
    notes: "Five pins: fastapi, uvicorn, pymysql, sqlalchemy, alembic. No auth library, no test runner."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
    notes: "The only DDL for stock_item; PK (sku, warehouse_cd)."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_health.py"
    notes: "Tests the unmounted router's stub."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
    notes: "Asserts the reason set from app/config.py, not the one app/main.py actually uses."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "Relative to the order-service repo. The caller side: POST /inventory/restore with query params and a null body."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/inventory/openapi.json"
    notes: "Reef-relative tier-4 extraction of the FastAPI surface, including x-unmounted."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "Reef-relative. 취소정책 sheet: the 재고 복원 column per reason code."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "Reef-relative. 조직도 and 변경이력 sheets."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Reef-relative. Registry entry claims db: MySQL (inventory)."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "Reef-relative. §5 재고 복원."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:schemas/inventory/schema.md"
    notes: "Reef-relative tier-4 ERD extraction; predates the Alembic correction and still describes the columnless RESTORE_LOG."
notes: "Company-document refs are relative to the reef root; inventory-api code refs are relative to that repo's root and unprefixed in the body."
---

# Inventory API Service

## Overview

`inventory-api` is 셀플로우's stock service: a small FastAPI application that answers stock
lookups for a SKU and restores stock when an order is cancelled. The repository is tiny —
sixteen files, roughly two hundred lines — which makes it unusually easy to read end to end,
and that full read is what makes the interesting parts visible.

The README states the service's purpose and its ownership in one line:
"셀플로우 재고 관리. Python 3.9 / FastAPI / MySQL. 담당: 데이터플랫폼본부 재고팀 (2022년 신설 시 이관)"
— "Sellflow inventory management. Python 3.9 / FastAPI / MySQL. Owner: Data Platform Division,
Inventory Team (transferred when the division was created in 2022)" (README.md). The org chart
confirms both the team and the move: 재고팀 (Inventory Team), lead 정하늘, five people, system
`inventory-api`, processes "재고 관리 · 재고 복원" ("inventory management · stock restoration"),
note "2022년 이관" ("transferred in 2022"), with the 변경이력 (change history) sheet dating it
2022-03-01 (sources/context/org-chart.xlsx).

The finding worth carrying out of this artifact is a boundary that is not where the paperwork
says it is. Every document describes a service with its own database: `.env.sample` sets
`DB_URL=mysql://inventory:@localhost:3306/inventory`, `alembic.ini` repeats the same URL, and
the service registry records `db: MySQL (inventory)` (sources/context/registry/services.yaml).
The code does something else. `app/db.py` builds its connection dictionary with
`"database": "sellflow_order"` hardcoded — the same shared MySQL instance order-service and
settlement-batch use — and `DB_URL` from `app/config.py` is never read by `app/db.py` at all.
Consequently the restock path reads `ORDER_DTL`, an order-domain table, with a direct SQL
`SELECT` rather than asking order-service for the order's lines (app/main.py). The declared
service boundary and the actual data boundary do not coincide.

The second thing this pass establishes is negative and definitive: there is no authentication
anywhere in the service. A case-insensitive grep of the entire repository for `auth`, `token`,
`depends`, `middleware`, `security`, `api key`, `jwt` and `cors` returns zero matches in any
file. The detail is in [[PROC-INVENTORY-AUTH]]; the fact belongs here because it changes how
every other boundary in this artifact should be read.

## Key Facts

- Runtime image is `python:3.9-slim` and the container entrypoint is `uvicorn app.main:app --host 0.0.0.0 --port 8000` → Dockerfile
- Dependencies are five pins — `fastapi==0.104.1`, `uvicorn==0.24.0`, `pymysql==1.1.0`, `sqlalchemy==2.0.23`, `alembic==1.12.1`. The migration tooling the Alembic directory needs is declared; there is still no authentication package and no test runner despite two test modules, and no ORM is used at runtime — `app/main.py` issues raw SQL through pymysql → requirements.txt, app/main.py
- The FastAPI application declares `title="inventory-api", version="3.2.1"`, which the extracted spec carries through as the API version → app/main.py, sources/apis/inventory/openapi.json
- Owner is 데이터플랫폼본부 재고팀 (Data Platform Division, Inventory Team), lead 정하늘, 5 people → sources/context/org-chart.xlsx
- The team was transferred into the newly created 데이터플랫폼본부 on 2022-03-01: "데이터플랫폼본부 신설, 재고팀 이관 (커머스본부 → 데이터플랫폼본부)" ("Data Platform Division established, Inventory Team transferred, Commerce Division → Data Platform Division") → sources/context/org-chart.xlsx
- **Q-030 resolved.** The service connects to `sellflow_order`, not to any inventory database. `app/db.py` hardcodes `"database": "sellflow_order"` in its DSN and assembles the rest from `DB_HOST` (default `localhost`) and `DB_USER` (default `sellflow`). `DB_URL` is defined in `app/config.py` but `app/db.py` neither imports `app.config` nor reads the variable, so the `inventory` URL in `.env.sample`, in `alembic.ini` and in the registry is inert configuration → app/db.py, app/config.py, .env.sample, alembic.ini, sources/context/registry/services.yaml
- The DSN carries no `password` key at all, so the connection is made as user `sellflow` without one → app/db.py
- **Q-032 resolved.** There is no authentication and no authorization. A case-insensitive grep of every file in the repository for `auth|token|depends|middleware|security|api.key|jwt|cors` returns zero matches; no route carries a `dependencies=` argument, no middleware is registered, and `app/config.py` and `.env.sample` declare no secret, key or allowlist. Anything that can reach port 8000 can read any SKU and can increment `available_qty` → app/main.py, app/routers.py, app/config.py, .env.sample, requirements.txt
- The one write endpoint reads the order-domain table `ORDER_DTL` directly with `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s` instead of calling order-service — inventory-api stores no order number of its own, as the README's bridge table admits with "`ORD_NO` | (보관하지 않음)" ("not retained") → app/main.py, README.md, sources/schemas/inventory/schema.md
- Restock is decided purely by the caller-supplied cancel reason code against `RESTOCKABLE_REASONS = {"01", "02"}`, and that set is declared twice — in `app/main.py`, which the handler uses, and again in `app/config.py`, which nothing imports → app/main.py, app/config.py
- The code comment gives the reasoning for excluding 03: "03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다" ("03, customer change of mind, is often cancelled after shipping has begun, so it is not subject to restoration"); `app/config.py` glosses the included two as "01 파트너귀책, 02 시스템오류" ("01 partner fault, 02 system error") → app/main.py, app/config.py
- The 취소정책 (cancellation policy) sheet marks 재고 복원 (stock restoration) as O for 01, O for 02, X for 03 and **O for 04 (배송 실패 — delivery failure)**, so the code silently withholds restock for reason 04 → sources/context/business-rules.md, app/main.py
- The Confluence cancellation policy assigns stock restoration to this service — "취소 시 재고 복원은 inventory-api 가 처리합니다" ("stock restoration on cancellation is handled by inventory-api") — and then defers the per-reason rule to this team rather than stating it: "취소 사유 코드에 따라 복원 여부가 달라지므로 재고팀과 협의가 필요합니다" ("whether stock is restored depends on the cancellation reason code, so coordination with the Inventory Team is required") → sources/raw/confluence-snapshots/주문-취소-정책_48213.html
- The Alembic chain resolves from a base: `3f9a` declares `down_revision = None` and imports `op` and `sqlalchemy`, `8ba1` chains from it. What is still missing is the runner — `alembic/env.py` is two lines with no `run_migrations_online` and no `target_metadata` → alembic/versions/20230414_1120-3f9a_add_restore_log.py, alembic/env.py
- `RESTORE_LOG` is created with `LOG_SEQ`, `ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD` and `REG_DTM`, indexed on `ORD_NO` (revision `3f9a`) and on `SAYU_CD` (revision `8ba1`). **No code reads or writes the table** → alembic/versions/20230414_1120-3f9a_add_restore_log.py, alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py, app/main.py
- Two migration systems coexist by design, and `3f9a`'s docstring records the split: `sql/V1__stock.sql` created `stock_item` before Alembic was introduced and is not under Alembic control, while the Alembic chain covers `RESTORE_LOG` only → sql/V1__stock.sql, alembic/versions/20230414_1120-3f9a_add_restore_log.py
- `stock_item`'s primary key is `(sku, warehouse_cd)`, but both SQL statements in `app/main.py` filter on `sku` alone: the lookup returns an arbitrary single warehouse row via `fetch_one`, and the restock `UPDATE` adds the quantity to every warehouse row for that SKU → sql/V1__stock.sql, app/main.py
- The two test modules test neither mounted endpoint: `test_health.py` calls the stub on the unmounted router, and `test_restore.py` asserts against `app.config.RESTOCKABLE_REASONS` — the copy the handler does not use — so the live rule could change without failing a test → tests/test_health.py, tests/test_restore.py, app/main.py
- Two endpoints are served and two more are defined but never mounted; the extracted spec records the split under `x-unmounted` with the note that calling them returns 404 → app/main.py, app/routers.py, sources/apis/inventory/openapi.json
- The only known caller posts to a path that does not exist on either surface: `restTemplate.postForEntity(baseUrl + "/inventory/restore?ordNo=" + ordNo + "&reason=" + sayuCd, null, Void.class)` → order-service `src/main/java/kr/co/sellflow/order/client/InventoryClient.java`

## Responsibilities

| Responsibility | Entry point | Notes |
|---|---|---|
| Serve stock for a SKU | `GET /stock/{sku}` in `app/main.py` | Selects `sku, available_qty, reserved_qty` from `stock_item`; 404 `"SKU 없음"` ("no such SKU") when absent. Not aggregated across warehouses. |
| Restore stock on cancellation | `POST /stock/restock` in `app/main.py` | Gate is the reason code; see [[DEC-INVENTORY-RESTOCK-BY-REASON]] and [[PROC-INVENTORY-RESTOCK]]. |
| Hold the restock-by-reason rule | `RESTOCKABLE_REASONS` in `app/main.py` | The Confluence policy explicitly defers this rule to this team, so the repository is its only home. |
| Own the `stock_item` table | `sql/V1__stock.sql` | The only DDL in the repo. Lifecycle in [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]]. |

## Does NOT Own

- **It does not own `ORDER_DTL`, which it reads.** The table belongs to order-service; this service reaches into it over a shared database connection (app/main.py, app/db.py). See [[CON-ORDER-INVENTORY]] and [[SYS-ORDER]].
- **It does not own an `inventory` database.** Configuration names one in three places; no code opens it (app/db.py, .env.sample, alembic.ini, sources/context/registry/services.yaml).
- **It does not own an audit trail.** `RESTORE_LOG` is created columnless and never written; the `RestoreLog` dataclass in `app/models.py` is imported by nothing. See [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]].
- **It does not own authentication.** Nothing in the repo performs it, and no deployment artifact that might delegate it exists here. See [[PROC-INVENTORY-AUTH]].
- **It does not own warehouse-level stock.** `app/models.py` says so outright — "재고 모델. SKU 단위로만 관리한다. 창고별 재고는 별도 시스템." ("Inventory model. Managed at SKU granularity only. Per-warehouse stock is a separate system.") — even though `stock_item` carries `warehouse_cd` in its primary key (app/models.py, sql/V1__stock.sql).
- **It does not own reservation.** `reserved_qty` is declared and returned but never written by any code path in the repository (sql/V1__stock.sql, app/main.py).
- **It does not decide whether an order may be cancelled.** It is told, after the fact, by a reason code it does not validate against any order state (app/main.py).

## Core Concepts

**Two databases on paper, one in practice.** The service's configuration describes an isolated
`inventory` database while its runtime code opens `sellflow_order`. Everything downstream
follows: the service can issue cross-domain SQL, so it does, and the order/inventory boundary
becomes a shared-table coupling rather than an API contract. The cross-domain read is catalogued
in [[SCH-INVENTORY]].

**A vocabulary boundary the README is explicit about.** The repository openly acknowledges that
it speaks a different language from the order domain and prints the bridge table itself
(README.md). `SANGPUM_CD` in the order domain is `sku` here; `SURYANG` maps onto
`available_qty` / `reserved_qty`; `ORD_NO` is "보관하지 않음" ("not retained"). The terms and
their collisions are in [[GLOSSARY-INVENTORY]].

**Ownership sits outside the commerce organisation.** Stock restoration is triggered by a
commerce process — order cancellation, owned by 주문팀 under 커머스본부 — but implemented by a
team under 데이터플랫폼본부 (sources/context/org-chart.xlsx). The Confluence policy page,
written by 주문팀's 박성민 in 2021, does not state the restoration rule; it points at the other
team instead.

**Its one known caller does not match its one live endpoint.** order-service's `InventoryClient`
posts to `/inventory/restore` with query parameters, a path served by neither the mounted app
nor the unmounted router. The three-way mismatch is laid out in [[API-INVENTORY]] and belongs,
as a cross-system concern, to [[CON-ORDER-INVENTORY]].

## Domain Behavior Highlights

**Restock is a lookup on a code, not a judgement about stock.** The handler's first action is
`if req.reason_code not in RESTOCKABLE_REASONS`. Nothing else is consulted — not the order's
status, not whether the shipment left, not whether a previous restock already ran for the same
order number. The code comment explains 03's exclusion as a proxy for shipment state ("배송이
시작된 뒤 취소되는 경우가 많아" — "because it is often cancelled after shipping has begun"), which
is a statistical argument standing in for a check the service cannot make from its own data
(app/main.py). The decision and its reasoning are recorded in
[[DEC-INVENTORY-RESTOCK-BY-REASON]].

**The rule disagrees with the business-rule sheet, in one direction.** 취소정책 marks 재고 복원
as O for code 04, 배송 실패 (주소불명·수취거부) — "delivery failure (unknown address, refusal of
receipt)" — with the note "물류팀 확인 후 처리" ("processed after logistics team confirmation").
`RESTOCKABLE_REASONS` contains only 01 and 02, so returned-but-undelivered goods are never
restocked by this service (sources/context/business-rules.md, app/main.py). Code 03 matches;
codes 01 and 02 match. Only 04 diverges, and it diverges towards under-counting stock.

**The vocabulary mismatch is load-bearing, not cosmetic.** Because the restock handler must
join the two domains itself, one SQL statement contains both vocabularies at once: it selects
`SANGPUM_CD, SURYANG` and feeds them into `sku` and a quantity increment (app/main.py). The
README's bridge table is therefore not documentation about a past migration — it is the mapping
a reader needs to follow the live write path. The `ORD_NO` row of that table ("보관하지 않음")
also explains the absent audit trail: the service has no column anywhere in which to record
which order caused a stock movement, which is exactly what `RESTORE_LOG` was meant to fix.

**The audit trail is built and unused.** Revision `3f9a` (2023-04-14) creates `RESTORE_LOG` with
`LOG_SEQ`, `ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD` and `REG_DTM`, and indexes it on `ORD_NO`;
revision `8ba1` (2024-09-02) adds the reason index its filename promises, on `SAYU_CD`. Those two
columns — order number and reason code — are exactly the pair the README says this service does
not keep and the pair every audit question turns on. The dataclass in `app/models.py` sketches a
different shape (`ord_no`, `sku_cd`, `qty`, `sayu_cd`, `result`) and is imported by nothing, and
no code anywhere inserts a row. The table is ready and the write was never written
(alembic/versions/*, app/models.py, app/main.py). See [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]].

**The migration chain resolves; the runner does not exist.** `3f9a` is the base
(`down_revision = None`) and its docstring records why — `stock_item` predates Alembic and came
from `sql/V1__stock.sql` — and `8ba1` chains from it. `requirements.txt` now declares
`sqlalchemy==2.0.23` and `alembic==1.12.1`, so the imports resolve. What remains missing is
`alembic/env.py`, still a two-line file with no `run_migrations_online` and no `target_metadata`,
and `alembic.ini`'s `sqlalchemy.url`, which names the `inventory` database that `app/db.py` never
opens. `alembic upgrade head` would therefore either find no runner or target the wrong schema
(alembic/versions/*, alembic/env.py, alembic.ini, requirements.txt).

**Errors are mostly not handled.** The only explicit failure paths are two `HTTPException(404)`
raises — `"주문 상세 없음"` ("no order detail") and `"SKU 없음"` ("no such SKU") — plus the
non-error `{"restocked": False, "reason": "not_restockable"}`. A restock against an unknown SKU
updates zero rows and still returns `{"restocked": true}`, because `execute` returns a row count
that the handler discards (app/main.py, app/db.py). See [[PROC-INVENTORY-ERROR-HANDLING]].

## Dependencies

| System | Integration Type | Purpose | Auth Method |
|---|---|---|---|
| MySQL `sellflow_order` (shared instance with order-service and settlement-batch) | Direct connection via pymysql, DSN assembled in `app/db.py` | Read and write `stock_item`; read `ORDER_DTL` | User from `DB_USER` (default `sellflow`), **no password in the DSN** (app/db.py) |
| `ORDER_DTL` table — owned by order-service, not by this service | Cross-domain SQL `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s`, on the same connection | Resolve an order number to the SKU and quantity to restock, because this service retains no order number | Same database credential; no API call, no contract, no grant recorded in either repo (app/main.py, README.md) |
| order-service (inbound caller) | HTTP POST from `InventoryClient` to `/inventory/restore?ordNo=…&reason=…`, default base `http://inventory-api.internal` | Request a restock on cancellation | **None.** The client sends a `null` body and no headers object; the server checks nothing (order-service `.../client/InventoryClient.java`, app/main.py) |
| Alembic (declared, non-functional) | `alembic.ini` + `alembic/env.py` + two revisions | Intended schema migration for `RESTORE_LOG` | `sqlalchemy.url = mysql://inventory:@localhost:3306/inventory` — a user with no password against a database no runtime code opens (alembic.ini) |

No message broker, cache, object store, secrets manager or observability agent appears anywhere
in the repository; `requirements.txt`'s five pins are the complete third-party surface, and two of
them (`sqlalchemy`, `alembic`) exist for the migration chain rather than for the running service.

## Runtime Components

| Component | Tech Stack | Purpose | Entry Point |
|---|---|---|---|
| HTTP application | FastAPI 0.104.1 on uvicorn 0.24.0, Python 3.9 | Serves the two mounted endpoints | `app.main:app` — `CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]` (Dockerfile) |
| Restock handler | FastAPI route + Pydantic `RestockRequest` | The service's only write path | `POST /stock/restock` → `restock()` in app/main.py |
| Stock lookup handler | FastAPI route | SKU stock read | `GET /stock/{sku}` → `get_stock()` in app/main.py |
| Database helpers | pymysql 1.1.0, `DictCursor` | `fetch_one` and `execute`; each opens and closes its own connection — there is no pool | `app/db.py` |
| Unmounted inventory router | FastAPI `APIRouter(prefix="/inventory")` | Two stubs — a hardcoded `{"skuCd": …, "qty": 0}` and a health check — that `app/main.py` never includes, so both 404 in the running service | `app/routers.py` (no `include_router` anywhere in the repo) |
| Unused domain models | Python dataclasses | `Stock` and `RestoreLog`; imported by no module and matching no table exactly | `app/models.py` |
| Migration tooling | Alembic config without the Alembic dependency | Nominally migrates `RESTORE_LOG`; chain is unrunnable | `alembic.ini`, `alembic/env.py`, `alembic/versions/` |
| Tests | Two pytest-style modules, no runner pinned | Assert the unmounted health stub and the unused copy of the reason set | `tests/test_health.py`, `tests/test_restore.py` |

## Related

- [[API-INVENTORY]] — the served and unserved endpoint surface, and the caller mismatch
- [[CON-ORDER-INVENTORY]] — the order ↔ inventory boundary as a contract
- [[DEC-INVENTORY-RESTOCK-BY-REASON]] — why the reason code alone decides restock
- [[GLOSSARY-INVENTORY]] — inventory vocabulary and its collisions with the order domain
- [[PROC-INVENTORY-AUTH]] — the (absent) authentication and authorization story
- [[PROC-INVENTORY-ERROR-HANDLING]] — the two 404s, and the failures that return success
- [[PROC-INVENTORY-RESTOCK]] — the restock workflow and its conflict with the business rules
- [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] — the audit table that was never given columns
- [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]] — how `stock_item` rows come into being and change
- [[RISK-INVENTORY]] — collected defects and gaps
- [[SCH-INVENTORY]] — `stock_item`, the unused dataclasses, and the cross-domain read
- [[SYS-ORDER]] — the calling system and owner of `ORDER_DTL`
