---
id: "CON-SELLFLOW-CANCEL-ENTITY-COMPARISON"
type: "contract"
title: "\"A Cancellation\" Across Five Services — Entity Comparison"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "scuba-depth read of all five representations plus the two policy documents; every field was read from DDL or code, nothing observed at runtime"
freshness_triggers:
  - "delivery-bff/src/generated/orderApi.ts"
  - "inventory-api/app/config.py"
  - "inventory-api/app/main.py"
  - "order-service/src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - "order-service/src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "order-service/src/main/resources/db/migration/V1__init.sql"
  - "settlement-anomaly/model/detector.py"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
  - "sources/context/business-rules.xlsx"
known_unknowns:
  - "Whether a SAYU_CD of '00' has ever actually been written to CANCEL_RECON_QUEUE; the queue export groups by month and status only, and no export in this reef breaks the queue down by reason code"
  - "Whether the reason-04 divergence (spreadsheet says restore stock, inventory-api does not) was a decision or an oversight; no document in this reef records a change"
  - "Whether `delivery-bff/src/generated/orderApi.ts` was really generated from any spec — its `OrderStatus` union and `CancelResponse` type appear in no spec in this reef, and its header claims generation from `order-service-openapi.json`"
  - "Which system, if any, holds the money value of a cancellation; no cancel representation in any of the five services carries an amount column"
  - "Whether the manual settlement corrections the 2023 agreement promised were recorded anywhere outside these databases; the queue export README says non-PENDING rows were not returned at all"
tags:
  - cancellation
  - cross-system
  - entity-comparison
  - vocabulary
aliases:
  - "취소 엔티티 비교"
  - "cancellation representations"
relates_to:
  - type: "constrains"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "depends_on"
    target: "[[DEC-ORDER-OUTBOX-RELAY]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "CancelRequest / CancelResponse / OrderStatus union, header dated 2022-11-08"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
    notes: "maps every 409 to the settlement message; defers regeneration to SF-4901"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderMst.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
    notes: "the only 409 the service can now raise"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V1__init.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V17__add_cancel_channel.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V9__order_cancel_bigo_extend.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
    notes: "DictCursor over sellflow_order"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet: per-reason 재고 복원 and 정산 차감 columns"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
    notes: "records the extraction query as SUM(EXPECTED_AMT)"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
notes: "Not a contract between two named parties but a contract nobody wrote: five services each hold something they call a cancellation, and no two of them agree on what its fields are."
---

## Parties

Five services each keep their own idea of "a cancellation". None of them shares a schema, a
vocabulary or a clock with any other, and only one pair (Order → Settlement, via the outbox) passes
a cancellation between them at all.

- **order-service** — the originator. Owns `ORDER_CANCEL` (one row per cancelled order) and the
  `CancelReason` / `OrderStatus` enums. It is the only party that validates a reason code.
- **settlement-batch** — the relay and the backlog. Turns an outbox row into a `CANCEL_RECON_QUEUE`
  row, but only for orders that already appear in `SETTLEMENT_DTL`.
- **inventory-api** — a decision, not a record. `POST /stock/restock` inspects a reason code, maybe
  adds quantity back to `stock_item`, and persists no cancellation of its own.
- **settlement-anomaly** — a derived query. It never receives a cancellation; it infers one nightly
  from `ORDER_MST.SANGTAE_CD` and emits a `CANCELLED_SETTLED` row.
- **delivery-bff** — the caller-facing shape. A generated TypeScript client with a `CancelRequest`,
  a `CancelResponse` and an `OrderStatus` union that no longer matches the service.

## Key Facts

- Order's cancellation is one row keyed on the order: `ORDER_CANCEL` has `PRIMARY KEY (ORD_NO)`, so an order can be cancelled exactly once, for ever → `order-service/src/main/resources/db/migration/V1__init.sql`
- Settlement's cancellation is an append-only queue row keyed on `SEQ BIGINT AUTO_INCREMENT` with `ORD_NO` unconstrained, so the same order can appear any number of times → `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- Both sides declare the reason code as `VARCHAR(2)` (`CHWISO_SAYU_CD` and `SAYU_CD`), which makes them look interchangeable at the schema level → `order-service/src/main/resources/db/migration/V1__init.sql`, `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- They are not interchangeable: Order gets the code from `CancelReason.of(code)`, which throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` ("unknown cancel reason code") on anything outside `01`–`04` → `order-service/src/main/java/kr/co/sellflow/order/domain/CancelReason.java`
- Settlement gets the same code by string surgery on the outbox payload — `s.indexOf("\"sayuCd\":\"")` then `s.substring(i + 10, i + 12)` — and returns the literal `"00"` when the marker is not found → `settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- `"00"` appears in no vocabulary anywhere: not in `CancelReason`, not in the 취소정책 sheet, not in the Confluence policy page, not in `RESTOCKABLE_REASONS` → `order-service/src/main/java/kr/co/sellflow/order/domain/CancelReason.java`, `sources/context/business-rules.md`, `inventory-api/app/config.py`
- The event that carries the code has exactly two fields — `String.format("{\"ordNo\":\"%s\",\"sayuCd\":\"%s\"}", ordNo, sayuCd)` — so no timestamp and no amount ever crosses the Order→Settlement boundary → `order-service/src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java`
- Order stamps `CHWISO_ILSI` from the application JVM (`LocalDateTime.now()` in the `OrderCancel` constructor) while Settlement stamps `RECV_DTM` from the database (`DEFAULT CURRENT_TIMESTAMP`) at relay time, up to one polling interval of 10 minutes later → `order-service/src/main/java/kr/co/sellflow/order/domain/OrderCancel.java`, `settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- Order's own per-cancellation state field is a constant: the `OrderCancel` constructor hardcodes `this.choriSangtae = "COMPLETED"`, so `CHORI_SANGTAE VARCHAR(20) NOT NULL` has exactly one value in practice → `order-service/src/main/java/kr/co/sellflow/order/domain/OrderCancel.java`
- The cancel transaction writes no status-history row: `OrderCancelService.cancel` injects only `OrderMstRepository`, `OrderCancelRepository` and `OrderEventPublisher`, and the pre-cancel status survives only in a log line that reads `order.getSangtaeCd()` *after* `order.chwiso()` has already overwritten it → `order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`
- settlement-anomaly has no cancellation entity at all: it selects `m.SANGTAE_CD` from `ORDER_MST` and calls any row whose status is in `CANCELLED_STATES = {"CHWISO", "BANPUM"}` a `CANCELLED_SETTLED` anomaly with a fixed `"score": 1.0` → `settlement-anomaly/model/detector.py`, `settlement-anomaly/app/main.py`
- Order treats `CHWISO` (cancel) and `BANPUM` (return) as two different things and blocks cancellation of both via `CHWISO_BULGA = EnumSet.of(OrderStatus.CHWISO, OrderStatus.BANPUM)`; settlement-anomaly collapses them into one set and reports them under a single code → `order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`, `settlement-anomaly/model/detector.py`
- inventory-api neither stores nor keys a cancellation: `restock()` takes `RestockRequest{ord_no, reason_code}`, updates `stock_item` and returns `{"restocked": True}` or `{"restocked": False, "reason": "not_restockable"}`; no table in `inventory-api/sql/V1__stock.sql` records that a cancellation happened → `inventory-api/app/main.py`, `inventory-api/sql/V1__stock.sql`
- The 취소정책 sheet marks 재고 복원 (stock restoration) `O` for reason `04` 배송 실패 (delivery failure), but `RESTOCKABLE_REASONS = {"01", "02"}` excludes it → `sources/context/business-rules.md`, `inventory-api/app/config.py`
- No cancellation representation in any of the five services carries a money amount. `ORDER_CANCEL`, `CANCEL_RECON_QUEUE` (through V1, V4 and V5) and the outbox payload all lack an amount column; `ORDER_MST.CHONG_GEUMAEK` is the order total and `SETTLEMENT_DTL.JUNGSAN_AMT` is what was paid out, neither of which is "the value of this cancellation" → `order-service/src/main/resources/db/migration/V1__init.sql`, `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- The one published money figure for the backlog is derived, not stored: the export README records the query as `SUM(EXPECTED_AMT)`, a column that exists in none of the queue's three migrations, and every row of the CSV is exactly `건수 × 45,760` (`2196480 / 48`, `188851520 / 4127`) → `sources/raw/exports/README.md`, `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`
- delivery-bff's `OrderStatus` union — `'JUMUN_WANRYO' | 'BAESONG_JUNG' | 'BAESONG_WANRYO' | 'CHWISO' | 'BANPUM'` — contains a value (`JUMUN_WANRYO`) that is in no server enum, and omits `GYEOLJE_WANRYO`, `SANGPUM_JUNBI` and, critically, `JUNGSAN_WANRYO` → `delivery-bff/src/generated/orderApi.ts`, `order-service/src/main/java/kr/co/sellflow/order/domain/OrderStatus.java`
- The BFF's shapes cannot be reproduced from the spec it names: `order-service-openapi.json` defines the 200 response of `POST /orders/{ordNo}/cancel` as `OrderCancel` (`ordNo`, `chwisoIlsi`, `chwisoSayuCd`, `choriSangtae`, `bigo`) and has no `CancelResponse` and no status enum, yet the file header says `source: order-service-openapi.json` and `직접 수정하지 마세요` ("do not edit directly") → `sources/raw/specs/order-service-openapi.json`, `delivery-bff/src/generated/orderApi.ts`
- The live controller returns `ResponseEntity<Void>` from `@PostMapping("/{ordNo}/cancel")` under `@RequestMapping("/orders")`, while the BFF fetches `/api/v1/orders/${ordNo}/cancel` and calls `res.json()` on the result → `order-service/src/main/java/kr/co/sellflow/order/controller/OrderController.java`, `delivery-bff/src/generated/orderApi.ts`

## Agreement

### Field-level comparison

| Concept | order-service | settlement-batch | inventory-api | settlement-anomaly | delivery-bff |
|---|---|---|---|---|---|
| **What a cancellation *is*** | a **row** — `ORDER_CANCEL`, one per order | a **queue entry** — `CANCEL_RECON_QUEUE`, relayed from an outbox **event** | a **decision** — an HTTP call with no persistence | a **derived query** — inferred nightly from order status | a **request/response pair** in a TypeScript client |
| **Identifier** | `ORD_NO VARCHAR(20)`, `PRIMARY KEY` → at most one cancellation per order | `SEQ BIGINT AUTO_INCREMENT` PK; `ORD_NO VARCHAR(20)` with no unique key → duplicates permitted | `ord_no: str` in the request body; nothing keyed, nothing stored | `ANOMALY_ID BIGINT AUTO_INCREMENT` PK; `ORD_NO VARCHAR(20)` non-unique | `ordNo: string` in the URL path and in `CancelResponse.ordNo` |
| **Reason code — name** | `CHWISO_SAYU_CD` (col) / `sayuCd` (API+event) | `SAYU_CD` | `reason_code` | *absent* | `sayuCd` |
| **Reason code — type/width** | `VARCHAR(2) NOT NULL`; Java `String` via `CancelReason` enum | `VARCHAR(2) NOT NULL` | Python `str`, unconstrained | — | `string`, unconstrained |
| **Reason code — vocabulary** | `01` 파트너 귀책 / `02` 시스템 오류 / `03` 고객 변심 / `04` 배송 실패, enforced — anything else throws | `01`–`04` **plus `00`**, enforced by nothing; value is a substring of a JSON string | membership test against `{"01","02"}`; everything else falls through to `not_restockable` | — | none; any string is sent |
| **Timestamp** | `CHWISO_ILSI DATETIME NOT NULL`, JVM `LocalDateTime.now()` at cancel time | `RECV_DTM DATETIME DEFAULT CURRENT_TIMESTAMP` — DB clock, at relay time (≤10 min later); `PROCESSED_DTM` (V1) and `PROCESSED_AT` (V5) both unwritten | `stock_item.updated_at DATETIME DEFAULT CURRENT_TIMESTAMP` with no `ON UPDATE`, and the restock `UPDATE` does not set it | `DETECTED_DTM` = `NOW()` at detect time — the 03:00 run *after* the 02:00 batch | none — the response carries no time at all |
| **State field** | two: `ORDER_CANCEL.CHORI_SANGTAE VARCHAR(20)` and `ORDER_MST.SANGTAE_CD VARCHAR(20)` | `STATUS VARCHAR(20) DEFAULT 'PENDING'`; `PROCESSED_BY VARCHAR(30)` (V5) unwritten | none | `STATUS VARCHAR(20) DEFAULT 'DETECTED'`; `REVIEWED_BY` / `REVIEWED_DTM` | `sangtaeCd: OrderStatus` on `CancelResponse` |
| **State vocabulary** | `CHORI_SANGTAE` is always the literal `"COMPLETED"`; `SANGTAE_CD` is one of seven `OrderStatus` values, of which `CHWISO` and `BANPUM` are distinct | `'PENDING'` written by the relay; `'PROCESSED'` written only by `CancelReconciler`, which is not registered in `QuartzConfig` | — | `'DETECTED'` written; no code in the repo writes any other value. Input vocabulary is `CANCELLED_STATES = {"CHWISO","BANPUM"}` — **merged** | a 5-value union containing a phantom (`JUMUN_WANRYO`) and missing `JUNGSAN_WANRYO` |
| **Amount** | **none.** `ORDER_CANCEL` has no money column; `ORDER_MST.CHONG_GEUMAEK DECIMAL(15,0)` is the order total | **none.** No amount column in V1, V4 or V5. `SETTLEMENT_DTL.JUNGSAN_AMT DECIMAL(15,0)` is what was already paid | quantity, not money: `available_qty = available_qty + SURYANG` | reads `JUNGSAN_AMT` and `SUSURYO` as model features; stores only `SCORE DECIMAL(5,4)` | **none** |
| **Free text** | `BIGO VARCHAR(500)` → `VARCHAR(2000)` in V9; `CHNL_CD VARCHAR(10)` added in V17 | none | none | none | `bigo?: string`, optional |

### How one cancellation propagates

```mermaid
sequenceDiagram
    autonumber
    actor CS as CS admin / app
    participant BFF as delivery-bff<br/>(request/response)
    participant ORD as order-service<br/>(a ROW)
    participant OBX as ORDER_EVENT_OUTBOX<br/>(an EVENT)
    participant STL as settlement-batch<br/>(a QUEUE ROW)
    participant INV as inventory-api<br/>(a DECISION)
    participant ANM as settlement-anomaly<br/>(a DERIVED QUERY)

    CS->>BFF: requestCancel(ordNo, sayuCd, bigo)
    BFF->>ORD: POST /api/v1/orders/{ordNo}/cancel
    Note over BFF,ORD: BFF expects CancelResponse{ordNo, sangtaeCd};<br/>controller returns ResponseEntity<Void>
    ORD->>ORD: CancelReason.of(sayuCd) — throws on anything but 01-04
    ORD->>ORD: INSERT ORDER_CANCEL (ORD_NO PK, CHWISO_ILSI=JVM now,<br/>CHORI_SANGTAE='COMPLETED')
    ORD->>ORD: ORDER_MST.SANGTAE_CD := CHWISO
    ORD->>OBX: INSERT order.cancelled {"ordNo","sayuCd"} — no time, no amount
    ORD--xINV: no call is ever made (InventoryClient has zero callers)
    loop every 10 min
        STL->>OBX: SELECT ... WHERE PUBLISHED_YN='N'
        STL->>STL: sayuCd(payload) — substring, else "00"
        alt order already in SETTLEMENT_DTL
            STL->>STL: INSERT CANCEL_RECON_QUEUE (SAYU_CD, RECV_DTM=DB now, STATUS='PENDING')
        end
        STL->>OBX: UPDATE PUBLISHED_YN='Y'
    end
    Note over STL: CancelReconciler would flip PENDING→PROCESSED,<br/>but it is not registered in QuartzConfig — row stays PENDING for ever
    Note over ANM: nothing is sent to anomaly detection
    ANM->>ANM: 03:00 — JOIN ORDER_MST, sangtae_cd in {CHWISO, BANPUM}
    ANM->>ANM: INSERT SETTLEMENT_ANOMALY (ANOMALY_CD='CANCELLED_SETTLED', SCORE=1.0)
```

## Current State

### 1. `SAYU_CD` — identical declaration, different provenance

The two columns are declared the same way, which is exactly what makes the divergence dangerous: a
join or a `UNION` between `ORDER_CANCEL.CHWISO_SAYU_CD` and `CANCEL_RECON_QUEUE.SAYU_CD` type-checks
and looks correct. But Order's value went through an enum lookup that rejects unknown codes, while
Settlement's value is `s.substring(i + 10, i + 12)` over `String.valueOf(payload)` with a
`return i < 0 ? "00"` fallback. `"00"` is not a reason; it is the parser's way of saying it could not
find one. Because the column is `NOT NULL` and two characters wide, that failure is
indistinguishable in the data from a real code.

The fallback is reachable for any payload that does not literally contain `"sayuCd":"`. The publisher
today always produces one, so the "00" path depends on payload format staying byte-identical — a
coupling that nothing in either repo tests or documents. It is also silent: the method logs nothing
and the insert proceeds.

### 2. The reason vocabulary, service by service

| Code | Label (verbatim / meaning) | order-service | 취소정책 sheet: 재고 복원 | inventory-api | 취소정책 sheet: 정산 차감 | settlement-batch |
|---|---|---|---|---|---|---|
| `01` | 파트너 귀책 — partner fault | enum `PARTNER_GWICHAEK` | O | restock | O | stored, never acted on |
| `02` | 시스템 오류 — system error | enum `SYSTEM_ORYU` | O | restock | O | stored, never acted on |
| `03` | 고객 변심 — customer change of mind | enum `GOGAEK_BYEONSIM` | X | no restock | O | stored, never acted on |
| `04` | 배송 실패 — delivery failure | enum `BAESONG_SILPAE` | **O** | **no restock** | O | stored, never acted on |
| `00` | — (no label anywhere) | rejected by `CancelReason.of` | not listed | `not_restockable` | not listed | **writable** by the relay fallback |

Two divergences, not one. The visible one is code `04`: the spreadsheet requires stock restoration
and `RESTOCKABLE_REASONS = {"01", "02"}` does not perform it. The quieter one is that the entire
정산 차감 (settlement deduction) column reads `O` for all four codes — every cancellation is supposed
to reduce settlement — while `SAYU_CD` in `CANCEL_RECON_QUEUE` is read by nothing that runs. The
reason code is stored in Settlement purely so that a reconciler that has never executed could one day
branch on it.

### 3. "Cancelled" as a state — four incompatible readings

- **order-service** keeps `CHWISO` (취소, cancel) and `BANPUM` (반품, return) as separate
  `OrderStatus` values and blocks new cancellations for both, because the business paths differ: the
  Confluence page routes a post-delivery request to the return process, "반품 프로세스로 전환"
  (switch to the return process).
- **settlement-anomaly** merges them: `CANCELLED_STATES = {"CHWISO", "BANPUM"}`, one code
  `CANCELLED_SETTLED`, one fixed score of `1.0`. A return and a cancellation are the same finding.
- **inventory-api** has no notion of a cancelled state at all. It has a *reason* filter, so the
  question it answers is not "is this cancelled" but "does this reason class return stock" — and for
  half the vocabulary the answer is no.
- **delivery-bff** carries `CHWISO` and `BANPUM` in its union but not `JUNGSAN_WANRYO`. Since
  SF-2287 the most interesting order a client can encounter is a settled one, and a settled order's
  status cannot be represented in the client's own type. It also carries `JUMUN_WANRYO`, which no
  server enum defines. Its own wrapper admits the drift — "생성 클라이언트가 2022 스펙 기준이라
  상태코드 목록이 현재와 다르다" (the generated client is based on the 2022 spec so its status code
  list differs from the current one) — and defers the fix to SF-4901, marked 미착수 (not started).

The same file compounds this at the error level. It maps *any* 409 to
`'정산이 완료된 주문은 취소할 수 없습니다.'` (a settled order cannot be cancelled). Since SF-2287
removed the settlement check, the only 409 the server can now raise is
`OrderCancelNotAllowedException`, thrown when the order is already `CHWISO` or `BANPUM`. The client
therefore reports the one cause that is no longer possible, and hides the one that is.

### 4. Event, row, status, or query — it is all four

A single cancellation exists simultaneously as:

- a **row** (`ORDER_CANCEL`) — unique per order, the only record that keeps the free-text `BIGO` and
  the channel `CHNL_CD`;
- an **event** (`ORDER_EVENT_OUTBOX`) — two fields, consumed once, then flipped to `PUBLISHED_YN='Y'`
  whether or not a queue row was written;
- a **status** (`ORDER_MST.SANGTAE_CD = 'CHWISO'`) — mutable, and written by more than one owner: the
  settlement batch's `MarkSettledTasklet` also writes this column;
- a **queue row** (`CANCEL_RECON_QUEUE`) — created only when `SELECT COUNT(1) FROM SETTLEMENT_DTL
  WHERE ORD_NO = ?` is non-zero, so it exists for settled orders only;
- a **derived query** (`SETTLEMENT_ANOMALY`) — recomputed nightly from the status, with no link back
  to any of the above.

None of these five carries a foreign key to any other; `V1__init.sql` says so outright: "FK 제약 없음"
(no FK constraints). The only join key is `ORD_NO`, and it is `VARCHAR(20)` everywhere, which is the
one thing the five services do agree on.

## Impact Analysis

**Counting.** The four representations count different populations and none of them counts
"cancellations". `ORDER_CANCEL` counts cancelled orders (all of them, settled or not).
`CANCEL_RECON_QUEUE` counts cancellations of *already-settled* orders, and only those that the relay
processed — and it can double-count, because `ORD_NO` has no unique constraint and the relay's
insert-then-mark sequence is not one transaction. `SETTLEMENT_ANOMALY` counts *detections*, which
recur every night for as long as the status persists and the order stays in the examined run, so its
row count is closer to a day-count than an order-count. `stock_item` counts nothing at all.

**Money.** The 188,851,520 KRW figure that frames the whole SF-2287 backlog is not summed from any
cancellation record, because no cancellation record has an amount. It is `4,127 × 45,760`, a flat
per-case estimate applied uniformly across 41 months — the CSV's own header calls it
추정 미정정 금액 (estimated uncorrected amount), and the export README's recorded query names a
column, `EXPECTED_AMT`, that no migration creates. The count is real; the money is an assumption
about the average, and the per-month figures are that assumption multiplied out.

**Timing.** Three clocks and three latencies stack: the JVM writes `CHWISO_ILSI`, the DB writes
`RECV_DTM` up to ten minutes later, and `DETECTED_DTM` lands at the 03:00 detection run. Any
reconciliation of a cancellation against a settlement day has to choose one of these, and the three
can fall on different calendar dates for a cancellation made near midnight.

**Detection.** The `CANCELLED_SETTLED` rule is narrower than it reads. Its input rows come from
`SETTLEMENT_DTL` joined on the *current* run's `JUNGSAN_ILJA`, and `MarkSettledTasklet` sets every
one of those orders to `JUNGSAN_WANRYO` in the same 02:00 job. For the rule to fire, an order must be
cancelled in the window between the batch writing `JUNGSAN_WANRYO` and the 03:00 detection run — a
cancellation on any earlier settlement day is never re-examined. The detector also indexes rows with
lowercase keys (`r["sangtae_cd"]`, `r["ord_no"]`, `r["jungsan_amt"]`) while `settlement-anomaly/app/db.py`
uses `pymysql.cursors.DictCursor` over a query that selects `m.SANGTAE_CD` unaliased, which yields
uppercase keys. Whether this raises `KeyError` in production is not verifiable from source alone and
is recorded as an open question rather than a conclusion.

**Quantity.** When restock does run, `UPDATE stock_item SET available_qty = available_qty + %(q)s
WHERE sku = %(sku)s` filters on `sku` alone, while the table's primary key is
`(sku, warehouse_cd)`. One cancellation therefore credits the quantity to every warehouse row for
that SKU. `fetch_one` also returns a single `ORDER_DTL` row, so a multi-line order restocks only one
line.

## Canonical Writing Rules

If an agent must write or interpret a cancellation, use this table and nothing else.

| Question | Authoritative representation | Why |
|---|---|---|
| Was this order cancelled, and when, and why? | `ORDER_CANCEL` (`ORD_NO`, `CHWISO_ILSI`, `CHWISO_SAYU_CD`, `BIGO`, `CHNL_CD`) | Only record written inside the cancel transaction, with a validated reason and a unique key |
| What reason code is legitimate? | `CancelReason` in order-service | The only enforced vocabulary; `01`–`04` and nothing else |
| Is this order currently cancelled? | `ORDER_MST.SANGTAE_CD` | But note it is overwritten by `MarkSettledTasklet`, so a cancelled-then-settled order may read `JUNGSAN_WANRYO` |
| Was it a cancellation or a return? | `ORDER_MST.SANGTAE_CD` — `CHWISO` vs `BANPUM` | Only order-service distinguishes them |
| Which cancellations still owe Settlement a clawback? | `CANCEL_RECON_QUEUE` rows with `STATUS='PENDING'` | The only record of the SF-2287 obligation — treat as a *backlog*, not a ledger |
| Did stock go back? | `stock_item.available_qty` history, if any exists | inventory-api writes no cancellation record; the endpoint has no caller, so the honest answer today is "no, for any reason code" |
| What should have happened per policy? | 취소정책 sheet in `context/business-rules.md`, cross-checked against the code | The sheet and the code disagree on reason `04` |

**Never treat as a count of anything:**

- `SETTLEMENT_ANOMALY` rows — detections, re-emitted per run, not cancellations.
- `CANCEL_RECON_QUEUE` row counts as "cancellations" — they are settled-order cancellations only, and
  `ORD_NO` is not unique.
- `ORDER_EVENT_OUTBOX` rows with `PUBLISHED_YN='Y'` as "delivered" — the relay marks published
  unconditionally, including for orders where no queue row was written.
- The 추정 미정정 금액 column of the monthly export as an amount — it is a count times a constant.

**Never write:** a `SAYU_CD` of `"00"` into anything; a `CHORI_SANGTAE` other than `"COMPLETED"`
without changing the constructor that hardcodes it; a `CANCEL_RECON_QUEUE` row from outside the relay
(nothing dedupes on `ORD_NO`); or a `JUNGSAN_WANRYO` status into any structure typed by the BFF's
`OrderStatus` union, which cannot represent it.

## Related

- [[PROC-ORDER-CANCEL]] -- the originating flow this artifact compares against
- [[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]] -- the `ORDER_CANCEL` row in full
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] -- the queue row in full
- [[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]] -- the derived-query representation
- [[PROC-INVENTORY-RESTOCK]] -- the decision-only representation and its reason filter
- [[CON-ORDER-SETTLEMENT]] -- the outbox boundary that carries two fields
- [[CON-ORDER-INVENTORY]] -- the restock contract neither side executes
- [[DEC-ORDER-OUTBOX-RELAY]] -- why the event exists and why it is polled
- [[DEC-SELLFLOW-SHARED-DB]] -- why four of these five representations live in one database
- [[GLOSSARY-SELLFLOW]] -- the cross-service vocabulary this artifact stress-tests
