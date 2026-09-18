---
id: "PROC-INVENTORY-RESTOCK"
type: "process"
title: "Stock Restoration on Order Cancellation"
domain: "inventory"
status: "draft"
last_verified: 2026-09-18
freshness_note: "Traced on 2026-09-18 step by step through app/main.py's restock handler and app/db.py's helpers, then compared against the 취소정책 sheet of business-rules.xlsx and §5 of the 주문-취소-정책 Confluence snapshot. Goes stale if RESTOCKABLE_REASONS changes in either of the two places it is declared, or if the cancellation policy spreadsheet is revised."
freshness_triggers:
  - "app/config.py"
  - "app/db.py"
  - "app/main.py"
  - "sql/V1__stock.sql"
  - "tests/test_restore.py"
known_unknowns:
  - "Whether the exclusion of reason 04 is deliberate or an oversight. The code comment justifies excluding 03 only and says nothing about 04, and no ticket or commit message in the repo explains it."
  - "Which of the two rules is currently authoritative in the business. The spreadsheet says 04 restocks; the code says it does not; no document reconciles them."
  - "How multi-line orders are handled in practice, if at all. fetch_one returns one row, so at most one SKU per cancellation is restored, but nothing states whether multi-line orders reach this endpoint."
  - "Whether the request ever arrives at all, given that order-service posts to /inventory/restore rather than /stock/restock. See API-INVENTORY."
  - "Whether restock is retried, deduplicated or made idempotent anywhere. Calling the endpoint twice would increment available_qty twice; no guard exists."
  - "Which warehouse a restored unit logically belongs to. The cancellation carries no warehouse information, and the UPDATE does not filter on warehouse_cd."
  - "Who monitors the outcome. The only trace of a restock is a Python log line; RESTORE_LOG is never written."
  - "The meaning of the spreadsheet note '물류팀 확인 후 처리' for reason 04 — whether a manual logistics step is expected to substitute for the missing automatic restock."
tags:
  - inventory
  - restock
  - cancellation
  - workflow
  - sellflow
aliases:
  - "재고 복원"
  - "restock-on-cancel"
relates_to:
  - type: "depends_on"
    target: "[[API-INVENTORY]]"
  - type: "constrains"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[GLOSSARY-INVENTORY]]"
  - type: "refines"
    target: "[[RISK-INVENTORY]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "The second, unused declaration of RESTOCKABLE_REASONS."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "fetch_one returns a single row."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "The restock handler and the authoritative RESTOCKABLE_REASONS."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "Reef-relative. Sheet 취소정책, column 재고 복원."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "Reef-relative. §2 취소 사유 코드 and §5 재고 복원."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
    notes: "Composite PK (sku, warehouse_cd)."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "Relative to the order-service repo. The trigger."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
    notes: "Asserts against the config.py copy, not the one the app uses."
notes: ""
---

# Stock Restoration on Order Cancellation

## Purpose

When an order is cancelled, the units it held should return to sellable stock — unless the
cancellation reason says otherwise. This process documents what `POST /stock/restock` actually
does, which is narrower and blunter than either the business rules or the code's own comments
suggest.

Four things are worth knowing before reading the steps: the restock rule is declared twice in
the codebase, the rule the code enforces disagrees with the business-rule spreadsheet on one
reason code, the handler restores at most one SKU per cancellation, and the UPDATE it issues is
not qualified by warehouse even though the table's key is.

## Key Facts

- The set of restockable reasons is declared in `app/main.py` as `RESTOCKABLE_REASONS = {"01", "02"}` and the handler in the same module checks against it → app/main.py
- The identical set is declared a second time in `app/config.py` as `RESTOCKABLE_REASONS = {"01", "02"}` → app/config.py
- `app/main.py` never imports from `app/config.py`; its only import from the package is `from app.db import fetch_one, execute`, so the config copy has no effect on behaviour → app/main.py
- The only importer of the config copy is the test suite: `from app.config import RESTOCKABLE_REASONS` in `tests/test_restore.py` → tests/test_restore.py
- The tests therefore assert against a constant the application does not use, and would keep passing if the `app/main.py` copy were changed → app/config.py, app/main.py, tests/test_restore.py
- The business-rule spreadsheet marks 재고 복원 (stock restoration) as O for reason 01, O for 02, X for 03 and **O for 04** → sources/context/business-rules.xlsx, sheet 취소정책
- The code excludes 04, so reason 04 (배송 실패 / delivery failure) never restores stock despite the spreadsheet requiring it → app/main.py, sources/context/business-rules.xlsx
- The spreadsheet's only restriction for 03 is the note "배송 시작 후 취소 시 재고 복원 불가" ("stock cannot be restored when cancellation occurs after shipping has started") → sources/context/business-rules.xlsx
- The code comment justifies excluding 03 and says nothing at all about 04: "03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다." — "03 (customer change of mind) is often cancelled after shipping has started, so it is not subject to restoration." → app/main.py
- The spreadsheet's row for 04 reads 사유 "배송 실패 (주소불명·수취거부)" ("delivery failure — unknown address, refusal of receipt"), 비용 부담 주체 파트너 (partner bears the cost), 비고 "물류팀 확인 후 처리" ("processed after logistics team confirmation") → sources/context/business-rules.xlsx
- A non-restockable reason produces HTTP 200 with `{"restocked": False, "reason": "not_restockable"}`, not an error → app/main.py
- Order lines are fetched with `fetch_one`, which calls `cur.fetchone()` and returns a single row, so a multi-line order restores only its first line's SKU → app/db.py, app/main.py
- The restoring statement is `UPDATE stock_item SET available_qty = available_qty + %(q)s WHERE sku = %(sku)s`, with no `warehouse_cd` predicate → app/main.py
- Because `stock_item`'s primary key is `(sku, warehouse_cd)`, that UPDATE touches every warehouse row for the SKU, multiplying the restored quantity by the number of warehouses → app/main.py, sql/V1__stock.sql
- Nothing is written to `RESTORE_LOG` during the process, although the table is created by a migration and a `RestoreLog` dataclass exists → app/main.py, app/models.py, alembic/versions/20230414_1120-3f9a_add_restore_log.py
- The only record of a restock is a log line, `log.info("재고 복원 완료. ord_no=%s", req.ord_no)` ("stock restoration complete") → app/main.py
- The Confluence policy page does not state the per-reason rule; it delegates: "취소 사유 코드에 따라 복원 여부가 달라지므로 재고팀과 협의가 필요합니다" ("whether stock is restored depends on the cancellation reason code, so coordination with the Inventory Team is required") → sources/raw/confluence-snapshots/주문-취소-정책_48213.html
- The process has **no live trigger**. order-service's `InventoryClient.restore(ordNo, sayuCd)` is the only code that would start it, and it has no caller anywhere in the estate — `OrderCancelService` never calls inventory → order-service `src/main/java/kr/co/sellflow/order/client/InventoryClient.java`, `src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`. See [[CON-ORDER-INVENTORY]]

## Steps

1. **Trigger (does not fire today).** The designed entry point is order-service calling `InventoryClient.restore(ordNo, sayuCd)`,
   which posts to `{baseUrl}/inventory/restore?ordNo=..&reason=..`. The comment on that class
   places the decision on this side: "복원 여부 판단은 재고팀 쪽 로직" ("the judgment on
   whether to restore is the Inventory Team's logic"). Whether the request reaches
   `POST /stock/restock` at all is an open question — see [[API-INVENTORY]].

2. **Validate the body.** FastAPI parses `RestockRequest{ord_no: str, reason_code: str}`. A
   missing or mistyped field is a 422 before the handler runs (app/main.py).

3. **Filter on the reason code.** `if req.reason_code not in RESTOCKABLE_REASONS:` — the
   `app/main.py` copy, `{"01", "02"}`. On a miss the handler logs
   "복원 대상 아님" ("not subject to restoration") and returns HTTP 200 with
   `{"restocked": False, "reason": "not_restockable"}` (app/main.py). Nothing is written; the
   caller receives a success status.

4. **Read the order line — across the domain boundary.**
   `fetch_one("SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s", {"o": req.ord_no})`.
   This is an order-domain table, read directly rather than through order-service; it is
   reachable because `app/db.py` connects to the shared `sellflow_order` database
   (app/main.py, app/db.py). See [[SCH-INVENTORY]].

5. **Handle an empty result.** If no row comes back, the handler raises
   `HTTPException(404, "주문 상세 없음")` ("no order detail") (app/main.py).

6. **Increment available stock.**
   `UPDATE stock_item SET available_qty = available_qty + %(q)s WHERE sku = %(sku)s`, with
   `q` taken from `SURYANG` and `sku` from `SANGPUM_CD` (app/main.py). `execute` opens its own
   connection, runs the statement, commits and returns the affected row count — which the
   handler ignores (app/db.py, app/main.py).

7. **Log and return.** `log.info("재고 복원 완료. ord_no=%s", req.ord_no)` and
   `{"restocked": True}` (app/main.py). No `RESTORE_LOG` row, no event, no acknowledgement of
   how many rows were actually changed.

### Where the process loses information

Three losses are structural rather than incidental:

**One row per order.** `fetch_one` wraps `cur.fetchone()` (app/db.py). A cancellation of a
three-line order restores one line. The remaining lines are not logged as skipped — the handler
has no idea they exist, because the query never asked for them.

**Every warehouse, not one.** The UPDATE names only `sku` in its `WHERE` clause, while the
table's primary key is `(sku, warehouse_cd)` (app/main.py, sql/V1__stock.sql). A SKU stocked in
two warehouses therefore gains the restored quantity twice. The nature of the request makes
this hard to fix in place: the cancellation payload carries no warehouse at all, so the handler
has no basis on which to pick one.

**No durable trace.** `RESTORE_LOG` exists as a migration and `RestoreLog` exists as a
dataclass, and neither is ever used (app/models.py,
alembic/versions/20230414_1120-3f9a_add_restore_log.py). Reconstructing what was restored, for
which order and why, is possible only from application logs.

### The rule conflict

The spreadsheet and the code state different rules, and they diverge on exactly one code:

| Reason code | 사유 (reason) | Spreadsheet 재고 복원 | Code (`RESTOCKABLE_REASONS`) | Agreement |
|---|---|---|---|---|
| 01 | 파트너 귀책 (재고부족·출고지연) — partner fault, out of stock or late dispatch | O | restocks | agree |
| 02 | 시스템 오류 — system error | O | restocks | agree |
| 03 | 고객 변심 — customer change of mind | X | does not restock | agree |
| 04 | 배송 실패 (주소불명·수취거부) — delivery failure, unknown address or refusal of receipt | **O** | **does not restock** | **conflict** |

Two details make the conflict more interesting than a simple omission. First, the spreadsheet's
X for 03 is conditional — its note is "배송 시작 후 취소 시 재고 복원 불가" ("stock cannot be
restored when cancellation occurs after shipping has started") — while the code applies it
unconditionally, and its comment reasons probabilistically rather than factually:
"03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아" ("03 is often cancelled after shipping
has started"). The handler never checks whether shipping actually started; it has no order
status to check against.

Second, the code comment argues only about 03. There is no sentence anywhere in the repository
that justifies leaving 04 out. Goods that failed delivery are, physically, goods that come back
— and the spreadsheet says the partner bears the cost and that logistics confirms the case
("물류팀 확인 후 처리"). Whether the omission is deliberate, whether a manual logistics step
substitutes for it, or whether it is simply a gap, cannot be answered from the sources at hand;
it sits in `known_unknowns`.

The Confluence policy page could have settled it but does not. §5 assigns the work to
`inventory-api` and refers the reader to the 재고팀, leaving the rule itself undocumented
outside the code (sources/raw/confluence-snapshots/주문-취소-정책_48213.html).

### The duplicated constant

`RESTOCKABLE_REASONS = {"01", "02"}` appears twice with identical values:

- `app/config.py`, with the comment "복원 대상 사유코드. 01 파트너귀책, 02 시스템오류."
  ("reason codes subject to restoration: 01 partner fault, 02 system error")
- `app/main.py`, with the comment about 03 quoted above

`app/main.py` imports nothing from `app/config.py` — its only intra-package import is
`from app.db import fetch_one, execute` — so the main copy is the one that governs behaviour.
The config copy is imported exclusively by `tests/test_restore.py`, whose two assertions are
`"03" not in RESTOCKABLE_REASONS` and `{"01", "02"} <= RESTOCKABLE_REASONS`. The tests therefore
verify a constant that the application never consults. Editing the live rule in `app/main.py`
would leave the suite green, which is the sort of quiet failure worth flagging: see
[[RISK-INVENTORY]].

## Worked Examples

**Reason 01, single-warehouse SKU — the happy path.** `POST /stock/restock` with
`{"ord_no": "2026090100123", "reason_code": "01"}`. `01` is in `RESTOCKABLE_REASONS`, so the
handler reads `ORDER_DTL` and gets `SANGPUM_CD='SKU-1001'`, `SURYANG=2`. The UPDATE adds 2 to
`available_qty` for `SKU-1001`. Response: `{"restocked": true}`. One warehouse row exists, so
the result is correct (app/main.py).

**Reason 04 — the rule conflict in practice.** Same order, `reason_code` `04`. `04` is not in
`{"01", "02"}`, so the handler logs "복원 대상 아님" and returns HTTP 200 with
`{"restocked": false, "reason": "not_restockable"}`. No stock moves. The spreadsheet says it
should have: 재고 복원 O for code 04 (app/main.py, sources/context/business-rules.xlsx). Because
order-service discards the response body entirely, nothing on the calling side records that
the restock was declined (order-service `InventoryClient.java`).

**Reason 02, two-warehouse SKU — quantity inflation.** `{"ord_no": "...", "reason_code": "02"}`
for a SKU stocked in `WH-SEOUL` and `WH-BUSAN`, with `SURYANG=2`. The statement
`UPDATE stock_item SET available_qty = available_qty + 2 WHERE sku = 'SKU-1001'` matches both
rows. Four units enter inventory for a two-unit cancellation. The response is still
`{"restocked": true}`, and since the handler ignores `execute`'s return value, the fact that
two rows changed rather than one is not observed (app/main.py, app/db.py, sql/V1__stock.sql).

**A three-line order — silent partial restock.** An order containing three different SKUs is
cancelled with reason `01`. `fetch_one` returns the first row only, so one SKU is restored and
the other two are not. The response is `{"restocked": true}` — indistinguishable from a complete
restock (app/db.py, app/main.py).

## Related

- [[API-INVENTORY]] — the endpoint contract and whether the trigger actually lands on it
- [[CON-ORDER-INVENTORY]] — the order ↔ inventory boundary this process crosses
- [[GLOSSARY-INVENTORY]] — reason codes, 재고 복원, `RESTOCKABLE_REASONS`
- [[RISK-INVENTORY]] — the rule conflict, the duplicated constant and the missing audit trail
- [[SCH-INVENTORY]] — `stock_item`, `ORDER_DTL` and the unwritten `RESTORE_LOG`
- [[SYS-INVENTORY]] — the owning service
- [[SYS-ORDER]] — the cancelling system that triggers this process
