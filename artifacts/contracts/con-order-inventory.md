---
id: "CON-ORDER-INVENTORY"
type: "contract"
title: "Order ↔ Inventory Restock Contract"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened 2026-09-19 by reading both sides end to end rather than in summary: order-service's InventoryClient.java, OrderCancelService.java, CancelReason.java and .env.example, and inventory-api's app/main.py, app/routers.py, app/config.py, app/db.py, app/models.py, sql/V1__stock.sql, both alembic revisions and both test files, cross-checked against sources/apis/inventory/openapi.json, sources/context/business-rules.md and the 2021 Confluence cancellation policy. This pass added a field-by-field mismatch table (path, transport, parameter names, body), resolved the warehouse known-unknown from the stock_item primary key, and added three couplings the previous version did not record: the router that is never mounted, the RESTORE_LOG table with no writer, and the fact that inventory-api's only live link to Order is a direct SELECT on ORDER_DTL in the sellflow_order database. Goes stale the moment InventoryClient acquires a caller, app/main.py calls include_router, the URL or payload shape on either side changes, or inventory-api stops reading ORDER_DTL."
freshness_triggers:
  - "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
  - "inventory-api:app/config.py"
  - "inventory-api:app/db.py"
  - "inventory-api:app/main.py"
  - "inventory-api:app/routers.py"
  - "inventory-api:sql/V1__stock.sql"
  - "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "sellflow-docs:context/business-rules.md"
known_unknowns:
  - "Whether stock is restored on cancellation by some route outside these five repos — a warehouse system, an operator screen, a scheduled job, a manual UPDATE. Checked by grepping all five sampled repos for InventoryClient, for restore(, and for the literal strings /stock/restock and stock_item; the only writer of stock_item found anywhere is inventory-api's own restock handler. Absence within the sample is not absence in the estate."
  - "Whether the reason-04 divergence (spreadsheet says restock, code does not) was a deliberate later decision or an oversight. The 2021 Confluence page does not carry a restock column at all, so the spreadsheet is the only document asserting it, and no ticket, comment or migration records a change."
  - "Why app/routers.py was written with an /inventory prefix and never mounted. Its docstring says the move has been pending since 2023 ('이관 예정, 2023부터'), but no ticket is referenced."
  - "Whether a gateway or ingress rewrite in front of inventory-api maps /inventory/restore onto /stock/restock and converts query parameters into a JSON body. Nothing in either repo shows such a component. The question is largely academic while InventoryClient has no caller."
  - "What RESTORE_LOG was meant to record and who was meant to write it. The table is created by alembic revision 3f9a (2023-04-14) and a matching RestoreLog dataclass exists in app/models.py, but no INSERT against it exists in any of the five repos."
  - "Whether stock_item rows are reconciled after the fact when a cancellation does not restock. No such job, query or runbook is present in this reef."
tags:
  - cancellation
  - cross-system
  - inventory
  - orphaned-component
  - restock
  - shared-database
aliases:
  - "재고 복원 연동"
  - "restock contract"
relates_to:
  - type: "constrains"
    target: "[[API-INVENTORY]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "constrains"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "constrains"
    target: "[[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[RISK-INVENTORY]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "integrates_with"
    target: "[[SYS-INVENTORY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "inventory-api:README.md"
    notes: "Vocabulary table SANGPUM_CD ↔ sku; places the restock decision on the inventory side."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
    notes: "Creates RESTORE_LOG on 2023-04-14. Nothing writes it."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "Second, unused copy of RESTOCKABLE_REASONS; DB_URL that db.py ignores."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "Hardcodes database=sellflow_order — the order-service schema."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "POST /stock/restock: the live endpoint, its JSON body and its reason-code rule."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
    notes: "APIRouter(prefix=/inventory) that app/main.py never registers."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
    notes: "stock_item PRIMARY KEY (sku, warehouse_cd)."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
    notes: "Asserts the reason-code rule against app/config, not against the handler."
  - category: "implementation"
    type: "github"
    ref: "order-service:.env.example"
    notes: "INVENTORY_BASE_URL=http://localhost:8082, differing from the code default."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "The caller-less client and the four-way payload mismatch."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
    notes: "01-04 reason codes as Order defines them."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The cancel transaction, which never touches inventory."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet: 재고 복원 column per reason code."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "2021 policy page; section 5 assigns restock to inventory-api and defers the per-code rule."
notes: "A contract that both sides describe in prose, that neither side executes, and whose only working link between the two services is an undeclared table read."
---

## Parties

- **Order (order-service)** — owner of the cancellation transaction and of the `ORD_NO` / `SANGPUM_CD` / `SURYANG` vocabulary. Holds `InventoryClient`, whose class comment states its job: "재고 API 연동. 취소 시 복원 요청을 보낸다. 복원 여부 판단은 재고팀 쪽 로직." (inventory API integration; sends a restore request on cancellation; the decision whether to restore is the inventory team's logic) → `order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- **Inventory (inventory-api)** — owner of `stock_item`, run by 데이터플랫폼본부 재고팀 (Data Platform Division, inventory team) since the team was created in 2022. Its README places the decision on its own side: "취소 시 재고 복원은 사유 코드에 따라 달라진다" (on cancellation, stock restoration depends on the reason code) → `inventory-api/README.md`
- **A third party neither side declares**: the `sellflow_order` MySQL database, which inventory-api opens directly in order to read `ORDER_DTL` → `inventory-api/app/db.py`

## Key Facts

- `InventoryClient.restore(ordNo, sayuCd)` exists in Order and has no caller anywhere in the five sampled repos; grepping `order-service`, `settlement-batch`, `inventory-api`, `delivery-bff` and `settlement-anomaly` for `InventoryClient` returns exactly one line, the class declaration itself → `order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- The cancel transaction never mentions inventory: `OrderCancelService.cancel` loads `ORDER_MST`, rejects `CHWISO`/`BANPUM`, resolves the reason code, saves an `ORDER_CANCEL` row, flips the status, publishes `order.cancelled` to the outbox, and logs. There is no inventory call, no injected `InventoryClient` field, and no `@Autowired` for one → `order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`
- The client targets `POST {base}/inventory/restore?ordNo=..&reason=..`. That path exists in neither of inventory-api's two route definitions, and the `/inventory` prefix that would have to host it is itself never mounted → `order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`, `inventory-api/app/routers.py`
- `app/routers.py` declares `APIRouter(prefix="/inventory", tags=["inventory"])` with `/inventory/stock/{sku_cd}` and `/inventory/health`, but `app/main.py` never calls `app.include_router`; grepping the whole repository for `include_router` returns zero hits, so every `/inventory/*` path 404s in the running service → `inventory-api/app/routers.py`, `inventory-api/app/main.py`, `sources/apis/inventory/openapi.json`
- The live endpoint is `POST /stock/restock`, which takes a Pydantic `RestockRequest` model — a required JSON body of `{ord_no, reason_code}` — and not query parameters → `inventory-api/app/main.py`
- Java sends a literal `null` request body: `restTemplate.postForEntity(url, null, Void.class)`. Even if the path matched, FastAPI would reject the request with 422 for a missing body before any inventory logic ran → `order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`, `inventory-api/app/main.py`
- The client discards the answer. Its return type is `void` and its response type is `Void.class`, so `{"restocked": false, "reason": "not_restockable"}` — the response the service returns for reasons 03 and 04 — would be thrown away without Order ever learning that stock did not come back → `order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`, `inventory-api/app/main.py`
- Inventory restocks only reason codes `01` and `02`, from `RESTOCKABLE_REASONS = {"01", "02"}` in `app/main.py`. The code comment explains the exclusion of 03: "03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다" (03, customer change of mind, is often cancelled after delivery has started, so it is not a restock target). Code 04 is excluded without comment → `inventory-api/app/main.py`
- The rule is written twice in the same repo and read once. `app/config.py` defines its own `RESTOCKABLE_REASONS = {"01", "02"}` with the gloss "복원 대상 사유코드. 01 파트너귀책, 02 시스템오류." (restock-eligible reason codes: 01 partner fault, 02 system error), and `app/main.py` imports nothing from `config` — it defines the set again locally. The tests assert the constant in `config`, which is the copy the endpoint does not use → `inventory-api/app/config.py`, `inventory-api/app/main.py`, `inventory-api/tests/test_restore.py`
- Order's own enum defines all four codes — `PARTNER_GWICHAEK("01")`, `SYSTEM_ORYU("02")`, `GOGAEK_BYEONSIM("03")`, `BAESONG_SILPAE("04")` — and explicitly declines to model the consequence: "사유별 비용 부담 주체는 코드에 정의하지 않는다. 정산 정책은 재무본부 소관이며 별도 기준표를 따른다." (which party bears the cost per reason is not defined in code; settlement policy belongs to the finance division and follows a separate reference table) → `order-service/src/main/java/kr/co/sellflow/order/domain/CancelReason.java`
- The business-rules spreadsheet says reason `04` (배송 실패 / delivery failure, 주소불명·수취거부 — unknown address or refused receipt) carries 재고 복원 = O, which the code does not do. Its note for 04 is "물류팀 확인 후 처리" (handle after logistics-team confirmation), which may or may not describe a manual step → `sources/context/business-rules.md`
- The 2021 Confluence page never settled the per-code rule at all. Section 5 재고 복원 says only: "취소 시 재고 복원은 inventory-api가 처리합니다. 취소 사유 코드에 따라 복원 여부가 달라지므로 재고팀과 협의가 필요합니다." (on cancellation, inventory-api handles restock; because whether stock is restored depends on the cancel reason code, coordination with the inventory team is required). The coordination is recorded as required, never as done → `sources/raw/confluence-snapshots/주문-취소-정책_48213.html`
- `inventory-api` does not need Order's API to read Order's data. `app/db.py` hardcodes `"database": "sellflow_order"`, and `restock` runs `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s` — an order-service table, read cross-domain, from the same schema that holds `stock_item` → `inventory-api/app/db.py`, `inventory-api/app/main.py`
- `app/config.py` declares `DB_URL` defaulting to `mysql://inventory:@localhost:3306/inventory`, naming a separate `inventory` database. Nothing reads `DB_URL`; `db.py` builds its own DSN and points at `sellflow_order`. The configuration file therefore documents a database separation that does not exist → `inventory-api/app/config.py`, `inventory-api/app/db.py`
- The restock UPDATE has no warehouse predicate. `stock_item`'s primary key is `(sku, warehouse_cd)`, and the handler runs `UPDATE stock_item SET available_qty = available_qty + %(q)s WHERE sku = %(sku)s`, so a single restock would add the full cancelled quantity to *every* warehouse row holding that SKU → `inventory-api/sql/V1__stock.sql`, `inventory-api/app/main.py`
- The restock reads one order line. `fetch_one` returns a single row, so a multi-line order would restock only whichever `ORDER_DTL` row MySQL returned first and silently ignore the rest → `inventory-api/app/db.py`, `inventory-api/app/main.py`
- `RESTORE_LOG` was created by alembic revision `3f9a` on 2023-04-14, a week before SF-2287 shipped, and a matching `RestoreLog` dataclass (`ord_no, sku_cd, qty, sayu_cd, result`) sits in `app/models.py`. No INSERT against `RESTORE_LOG` exists in any of the five repos, and `restock` writes nothing but a log line. The follow-up revision `8ba1` (2024-09-02, "add restore log reason i") has an `upgrade()` body of `pass` → `inventory-api/alembic/versions/20230414_1120-3f9a_add_restore_log.py`, `inventory-api/alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py`, `inventory-api/app/models.py`
- The two sides do not even agree on where inventory-api lives. `InventoryClient` defaults `INVENTORY_BASE_URL` to `http://inventory-api.internal`, while `order-service/.env.example` sets it to `http://localhost:8082` → `order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`, `order-service/.env.example`

## Agreement

### What the documents say the contract is

Order cancels an order; Order tells Inventory; Inventory decides from the reason code whether the quantity returns to `available_qty`. Both READMEs, the Confluence page and the business-rules spreadsheet describe some version of this. Nothing anywhere states the wire format, the auth posture, the retry policy, the idempotency key, the ordering against the cancel transaction, or what Order should do with a `restocked: false`.

The per-code expectation is written down in three places and no two of them fully agree:

| Reason code | Label | business-rules.xlsx 재고 복원 | 2021 Confluence | inventory-api behaviour |
|---|---|---|---|---|
| 01 | 파트너 귀책 (partner fault) | O | not stated | restock |
| 02 | 시스템 오류 (system error) | O | not stated | restock |
| 03 | 고객 변심 (customer change of mind) | X — "배송 시작 후 취소 시 재고 복원 불가" | not stated | no restock |
| 04 | 배송 실패 (delivery failure) | O — "물류팀 확인 후 처리" | not stated | no restock |

The spreadsheet's reasoning for 03 (stock cannot be restored when a cancellation follows the start of delivery) matches the code comment almost word for word, which is good evidence that 01–03 were once genuinely agreed. Code 04 is where documentation and code part company, and no source in this reef explains why.

### What the wire format actually is, field by field

This is not one mismatch with knock-on effects. It is four independent mismatches, any one of which alone would break the call:

| Dimension | order-service sends | inventory-api expects | Verdict |
|---|---|---|---|
| Path | `POST /inventory/restore` | `POST /stock/restock` | No overlap. And `/inventory/*` is not mounted at all, so even the prefix is fictional. |
| Transport of parameters | Query string, string-concatenated onto the URL, unencoded | JSON request body, parsed into a Pydantic model | Query parameters would be ignored; the body would be missing. |
| Parameter names | `ordNo`, `reason` (camelCase) | `ord_no`, `reason_code` (snake_case) | Both names differ, so even a body-shaped request would fail validation. |
| Request body | Literal `null` (`postForEntity(url, null, Void.class)`) | Required body — `RestockRequest` has no defaults | FastAPI answers 422 before the handler runs. |
| Response handling | `Void.class`, method returns `void` | `{"restocked": bool}`, optionally `{"reason": "not_restockable"}` | Order cannot distinguish "restocked" from "declined" from "404 no order detail". |
| Authentication | None sent | None required | The only dimension on which the two sides agree. |

The naming split is the `PAT-SELLFLOW-ROMANISED-NAMING` boundary in miniature: Order romanises Korean field names in camelCase (`ordNo`, `sayuCd`), Inventory snake-cases them (`ord_no`, `reason_code`), and the same concept — 취소 사유 코드 — is called `sayuCd`, `reason` and `reason_code` within a single intended round trip.

### The second, undeclared coupling

The HTTP contract above is the one both sides document. The coupling that actually exists in running code is a database read.

```
inventory-api (app/db.py)  ──pymysql──▶  sellflow_order.ORDER_DTL      (order-service's table)
inventory-api (app/db.py)  ──pymysql──▶  sellflow_order.stock_item     (inventory's own table)
```

`db.py` hardcodes `"database": "sellflow_order"` for every connection it opens, so both tables live in the order-service schema. `restock` reads `SANGPUM_CD` and `SURYANG` straight out of `ORDER_DTL` — no API call, no event, no contract document — and then updates `stock_item`. This is where the vocabulary translation the Inventory README documents (`SANGPUM_CD` → `sku`, `SURYANG` → `available_qty`) is actually performed: in a SQL result-set field access, with nothing on the Order side aware that the mapping exists or that a column rename would break it. See `DEC-SELLFLOW-SHARED-DB` for why this is possible at all.

```mermaid
sequenceDiagram
    autonumber
    actor CS as CS admin / customer app
    participant OC as OrderController
    participant OCS as OrderCancelService
    participant ODB as sellflow_order DB
    participant IC as InventoryClient
    participant INV as inventory-api

    CS->>OC: POST /orders/{ordNo}/cancel {sayuCd, bigo}
    OC->>OCS: cancel(ordNo, sayuCd, bigo)
    OCS->>ODB: INSERT ORDER_CANCEL, UPDATE ORDER_MST -> CHWISO
    OCS->>ODB: INSERT ORDER_EVENT_OUTBOX (order.cancelled)
    OCS-->>OC: void
    OC-->>CS: 200 (empty body)

    rect rgb(250, 235, 235)
    note over OCS,IC: THE DOCUMENTED CONTRACT - never executed.<br/>No caller for InventoryClient exists in any of the five repos.
    OCS--xIC: restore(ordNo, sayuCd)
    IC--xINV: POST /inventory/restore?ordNo=..&reason=.. (body: null)
    note over IC,INV: Would fail four ways: path unmounted,<br/>query vs JSON body, camelCase vs snake_case,<br/>null body vs required RestockRequest.
    INV--xIC: 404 (path not served)
    end

    rect rgb(235, 245, 235)
    note over INV,ODB: THE UNDECLARED COUPLING - the only live link.<br/>Reachable only if something calls POST /stock/restock.
    INV->>ODB: SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO=?
    INV->>ODB: UPDATE stock_item SET available_qty = available_qty + ? WHERE sku=?
    note over INV,ODB: No warehouse_cd predicate; stock_item PK is (sku, warehouse_cd).
    end
```

## Current State

**Is stock restored when an order is cancelled today? On the evidence in these five repos, no — for any reason code.**

The chain breaks at the first link, not the last. `OrderCancelService.cancel` is the only cancellation path in order-service, and it contains no inventory call of any kind. `InventoryClient` is a `@Component` that Spring will instantiate and inject into nothing. So the question of whether the wire format matches is downstream of a simpler fact: no request is ever issued.

That makes this three absences stacked on top of each other, and it is worth naming them separately because each would need a different fix:

1. **An endpoint with no caller.** `POST /stock/restock` is implemented, tested at the constant level, and reachable — and nothing in the sample calls it.
2. **A client with no caller.** `InventoryClient.restore` compiles, has a base URL, has an environment variable, and is invoked by nothing.
3. **A URL naming neither.** `/inventory/restore` is not the client's own service's route and not the server's route. It matches the unmounted router's prefix but not its paths. It is a third address that exists only in a string literal.

Add to that a table with no writer (`RESTORE_LOG`, created 2023-04-14), a config constant with no reader (`config.RESTOCKABLE_REASONS`), a `DB_URL` with no consumer, and a router with no registration, and the picture is of a feature that was designed, partially scaffolded on both sides, and never joined in the middle.

### How far that conclusion can be pushed

It is bounded by the sample. The grep covered `order-service`, `settlement-batch`, `inventory-api`, `delivery-bff` and `settlement-anomaly` — the five repos this reef holds. A warehouse management system, an operator screen, a cron job in another repository, or a human running `curl` would all be invisible here. The spreadsheet's note on code 04, "물류팀 확인 후 처리" (handle after logistics-team confirmation), and the Confluence page's "재고팀과 협의가 필요합니다" (coordination with the inventory team is required) both gesture at manual handling without naming a mechanism. No inventory-team runbook is in this reef. That is recorded in `known_unknowns`, not resolved.

What can be said without qualification is that **order-service does not restore stock**, because order-service's only cancel path is fully readable and does not try to.

### The warehouse question, now answered

The previous version of this artifact listed as unknown "which warehouse a restock should target; the order line carries `CHANGGO_CD` since V12 but no caller passes it". The Inventory side answers it, and the answer is worse than "unspecified": `stock_item`'s primary key is `(sku, warehouse_cd)`, so a SKU has one row per warehouse, and the restock UPDATE filters on `sku` alone. A restock of 3 units for a SKU stocked in four warehouses would add 3 units to each of the four — 12 units created out of one cancellation. The contract has no field for a warehouse because neither side's payload has one, and the handler compensates by updating all of them.

## Impact Analysis

### What a consumer of this reef must NOT assume

- **Do not assume a cancellation returns stock.** If you are reconciling inventory against orders, cancelled orders are the obvious suspect for a positive discrepancy (stock shown as sold that was never shipped), and nothing in this code corrects it.
- **Do not assume the reason-code table in `business-rules.xlsx` describes system behaviour.** It describes intent. For 04 it is contradicted by the code, and for every code it is moot while no call is made. Cite it as policy, never as behaviour.
- **Do not assume `/inventory/restore` is an endpoint.** It appears in a string literal in Java and nowhere else. Searching for it in inventory-api will find the router prefix and mislead you; that router is not mounted.
- **Do not assume `app/config.py` describes the running configuration.** Its `RESTOCKABLE_REASONS` is not the set the handler uses (though the values currently coincide), and its `DB_URL` names a database the service never opens. A change to `config.py` alone would change the tests' verdict and not the service's behaviour — a trap worth stating explicitly.
- **Do not assume the two services are decoupled because the HTTP call is dead.** They share a database. A rename of `ORDER_DTL.SANGPUM_CD` or `ORDER_DTL.SURYANG` breaks inventory-api with no compile-time, contract-test or code-review signal on the order-service side.
- **Do not assume wiring `InventoryClient` into `OrderCancelService` would fix anything.** It would produce a 404 per cancellation, swallowed — `postForEntity` on a 404 raises `HttpClientErrorException`, which `cancel` does not catch, and which would therefore roll back the `@Transactional` cancel. Adding the missing call without also fixing the path, the transport, the field names and the body would convert a silent gap into cancellation failures.
- **Do not assume `RESTORE_LOG` is an audit trail.** It has existed since 2023-04-14 and has never been written to by any code in this sample. Querying it to establish what was restored will return an empty set that means "nothing was recorded", not "nothing was restocked".
- **Do not assume a restock, once wired, would be idempotent or warehouse-correct.** `available_qty = available_qty + q` with no guard means a retry double-counts, and the missing `warehouse_cd` predicate means a multi-warehouse SKU is over-credited by a factor equal to its warehouse count.

### Blast radius if the contract were completed as written

A fix that only corrects the URL and payload would still inherit: no idempotency, no warehouse targeting, single-line reads on multi-line orders, no audit row, and a silent `restocked: false` for codes 03 and 04 that Order's `void` signature cannot observe. The contract needs designing, not repairing.

## Related

- [[SYS-ORDER]] -- the caller that never calls, and the owner of `ORDER_DTL`
- [[SYS-INVENTORY]] -- the callee, and the service that reads `ORDER_DTL` behind the contract's back
- [[API-INVENTORY]] -- the mounted-versus-unmounted route split in full
- [[SCH-INVENTORY]] -- `stock_item`'s `(sku, warehouse_cd)` key, which the restock UPDATE ignores
- [[PROC-ORDER-CANCEL]] -- the transaction in which the restock call is absent
- [[PROC-INVENTORY-RESTOCK]] -- the endpoint's own logic and its reason-code rule
- [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] -- the table created for this contract that nothing writes
- [[RISK-INVENTORY]] -- the same findings framed as exposure
- [[PAT-SELLFLOW-ROMANISED-NAMING]] -- the camelCase/snake_case split that makes `ordNo` and `ord_no` two different fields
- [[GLOSSARY-SELLFLOW]] -- the `SANGPUM_CD` / `sku` and `sayuCd` / `reason_code` vocabulary splits this boundary crosses
- [[DEC-SELLFLOW-SHARED-DB]] -- why inventory-api can read order tables at all
