---
id: "GLOSSARY-ORDER"
type: "glossary"
title: "Order Domain Glossary"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by reading all 24 order-service migrations line by line, the five JPA entities, the CancelReason and OrderStatus enums, the service and search classes, and the 2022 OpenAPI export. Every romanised column in the repository is now listed. Re-read on 2026-09-19 after a correction pass in order-service: the ORDER_DTL naming split is resolved — OrderItem maps the DDL names — and four of the six phantom columns turned out to be typos that have been corrected to the real ones, leaving ORD_DT and BAESONG_MSG. Goes stale if a migration adds or renames a column, if OrderStatus or CancelReason changes, or if the OrderItem mapping changes again."
freshness_triggers:
  - "sources/raw/specs/order-service-openapi.json"
  - "src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - "src/main/java/kr/co/sellflow/order/domain/OrderItem.java"
  - "src/main/java/kr/co/sellflow/order/domain/OrderMst.java"
  - "src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - "src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "src/main/resources/db/migration/V1__init.sql"
  - "src/main/resources/db/migration/V17__add_cancel_channel.sql"
known_unknowns:
  - "Whether PARTNER_ID here is the same identifier space as settlement-batch's PARTNER_CONTRACT.PARTNER_ID. Both are VARCHAR(20); no shared definition, FK or key registry exists. Checked: grepped all five repos for a partner table or seed — only PARTNER_CONTRACT exists, and no running code reads it."
  - "The other values of CHORI_SANGTAE. Only the literal \"COMPLETED\" is ever written, by the OrderCancel constructor; no enum, lookup table or column comment defines a second value."
  - "Whether the channel vocabularies are meant to be one vocabulary. ORDER_MST.CHAENNEL_CD is backfilled to 'WEB' and the spec's chaenNelCd enumerates WEB/APP_IOS/APP_AND/OPEN_MARKET/API, while ORDER_CANCEL.CHNL_CD's own column comment says APP/ADMIN/CS. Nothing reconciles them."
  - "What ORD_DT and BAESONG_MSG are. ORD_DT is selected on by OrderSearchService and BAESONG_MSG is named in V13's deferral TODO and in the 2022 spec as baesongMsg; no migration in V1-V24 creates either. Checked by grepping every .sql and .java file in the repo for each name."
  - "Whether BANPUM (반품, return) has a system of record. It is an OrderStatus value and a CS process in the org chart (CS팀 · 반품 접수), but no code in this repository ever writes it and no return table exists."
  - "Romanisation is inconsistent with itself across the codebase: TAEKBAESA_CD (V5) versus TAKBAE_CD (DeliveryInfo) versus taekBaeSaCd (spec); CHAENNEL_CD versus chaenNelCd versus CHNL_CD; JEOKRIPGEUM (V7) versus jeokRipGeum (spec); SANGPUM_CD (order) versus sku (inventory) versus SKU (RESTORE_LOG). No romanisation convention document was found in any repo or in sources/."
  - "Whether UNSONGJANG_BEONHO on ORDER_MST (V5) and INVOICE_NO on ORDER_DELIVERY hold the same waybill number. Both exist, nothing joins or reconciles them, and no code writes either."
tags:
  - "order"
  - "glossary"
  - "korean"
  - "romanisation"
aliases:
  - "order terms"
  - "주문 용어"
relates_to:
  - type: "refines"
    target: "[[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[SCH-ORDER]]"
  - type: "refines"
    target: "[[SCH-ORDER-MIGRATION-HISTORY]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "Module docstring states the SANGPUM_CD / sku equivalence explicitly."
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
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderItem.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderMst.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatusHistory.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventOutbox.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
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
    ref: "order-service:src/main/resources/db/migration/V18__backfill_cancel_channel.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "Sheet 취소정책 — reason codes with cost bearer and restock. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The ticket that removed the settled-order cancellation block."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "Reason code table with Korean descriptions. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "2022-11 export. Field-level Korean descriptions and four enums the DDL does not carry."
notes: "Korean labels are the enums' own strings and the migrations' own comments, quoted verbatim with an English rendering alongside. No Korean text here is a back-translation of English."
---

# Order Domain Glossary

## Overview

The order domain names everything in romanised Korean. A reader who knows the English concept will not recognise `SURYANG` as quantity, and — more dangerously — a reader who learns `SANGPUM_CD` in the order domain will meet the same concept as `sku` one service over.

This registry is now exhaustive over the order repository: every column the 24 migrations create, every column the entities map, and every column that is *referenced but never created*. That last group matters, because five of the six phantom names look exactly like the real ones and an agent cannot tell them apart without this table.

Three spellings of the same word is the normal case here, not the exception. 채널 (channel) is `CHAENNEL_CD` in the migration, `chaenNelCd` in the spec and `CHNL_CD` on the cancel table — with a *different value vocabulary* on the third. 택배사 (carrier) is `TAEKBAESA_CD` on `ORDER_MST`, `TAKBAE_CD` on `ORDER_DELIVERY` and `taekBaeSaCd` in the spec. 수량 (quantity) is `SURYANG` in the order schema, `available_qty` in inventory's own table and `SURYANG` again in inventory's `RESTORE_LOG`. See [[PAT-SELLFLOW-ROMANISED-NAMING]] for why.

## Terms

### `ORDER_MST` — order master

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `ORD_NO` | 주문번호 — order number | `VARCHAR(20)`, primary key of `ORDER_MST`, `ORDER_CANCEL` and `ORDER_DELIVERY`, and the join column in every other service. The V1 header states there are no FK constraints — "FK 제약 없음. 성능 이슈로 2019년 설계 당시 제외함." ("no FK constraints; excluded at design time in 2019 over performance concerns") — so it is a convention, not an enforced key → `src/main/resources/db/migration/V1__init.sql` | [[SCH-ORDER]] |
| `GOGAEK_ID` | 고객ID — customer id | `VARCHAR(20) NOT NULL`. Indexed with `JUMUN_ILSI` as `IX_ORDER_MST_01`. No customer table exists in this schema → `src/main/resources/db/migration/V1__init.sql` | [[SCH-ORDER]] |
| `JUMUN_ILSI` | 주문일시 — order placed date-time | `DATETIME NOT NULL`. Distinct from `REG_DTM` (row insert time) and from `ORD_DT`, which `OrderSearchService` queries and no migration creates → `src/main/resources/db/migration/V1__init.sql`, `src/main/java/kr/co/sellflow/order/search/OrderSearchService.java` | [[RISK-ORDER]] |
| `SANGTAE_CD` | 주문상태코드 — order status code | `VARCHAR(20) NOT NULL` holding an `OrderStatus` enum *name* (`@Enumerated(EnumType.STRING)`), not its Korean label. Written by this service and, for one value, by settlement-batch's tasklet → `src/main/java/kr/co/sellflow/order/domain/OrderMst.java` | `OrderStatus`, [[CON-ORDER-SETTLEMENT]] |
| `CHONG_GEUMAEK` | 총금액 — total amount | `DECIMAL(15,0) NOT NULL`, mapped as `BigDecimal`. Whole won; the scale is zero. Not "the value of a cancellation" — no such figure exists anywhere → `src/main/resources/db/migration/V1__init.sql` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| `BAESONG_JUSO` | 배송주소 — delivery address | `VARCHAR(500)`, nullable. Mapped on `OrderMst` with a getter that is never called in this repository; the only personal-data column in the order schema → `src/main/resources/db/migration/V1__init.sql`, `src/main/java/kr/co/sellflow/order/domain/OrderMst.java` | [[SCH-ORDER]] |
| `REG_DTM` | 등록일시 — row insert time | `DATETIME DEFAULT CURRENT_TIMESTAMP`. Database clock, not the business time; `JUMUN_ILSI` is the business time → `src/main/resources/db/migration/V1__init.sql` | [[SCH-ORDER]] |
| `UPD_DTM` | 수정일시 — row update time | `DATETIME`, set by `OrderMst.chwiso()` and by settlement's `MarkSettledTasklet`. Load-bearing: the settlement reader selects on `DATE(m.UPD_DTM) = ?`, so this column decides which orders are settled on a given day → `src/main/java/kr/co/sellflow/order/domain/OrderMst.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| `GOGAEK_MEMO` | 고객 메모 — customer memo | `VARCHAR(500) NULL`, added V2 (2019-09-11) for "주문 메모 컬럼 추가" ("add order memo column"). The spec gives it `maxLength: 500`. Not mapped on any entity → `src/main/resources/db/migration/V2__add_order_memo.sql`, `sources/raw/specs/order-service-openapi.json` | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `CHAENNEL_CD` | 채널코드 — inflow channel code | `VARCHAR(20) NULL`, added V3 (2020-02-20) "유입채널 구분 추가 (앱 출시 대응)" ("add inflow channel distinction, for the app launch") and backfilled to `'WEB'`. The spec's `chaenNelCd` enumerates `WEB`, `APP_IOS`, `APP_AND`, `OPEN_MARKET`, `API`; the column carries no constraint → `src/main/resources/db/migration/V3__add_channel_code.sql`, `sources/raw/specs/order-service-openapi.json` | [[SCH-ORDER]] |
| `UNSONGJANG_BEONHO` | 운송장번호 — waybill number | `VARCHAR(30) NULL`, added V5 (2020-11-03) at 물류팀's request. Indexed as `IX_ORDER_MST_03`. The spec's `unSongJangBeonho` gives the example `"123456789012"`. Nothing in the repository ever writes it → `src/main/resources/db/migration/V5__add_delivery_columns.sql` | [[GLOSSARY-DELIVERY]] |
| `TAEKBAESA_CD` | 택배사코드 — carrier company code | `VARCHAR(10) NULL`, added V5 alongside the waybill number. Constrained to five values in the spec (`CJ`, `HANJIN`, `LOTTE`, `POST`, `LOGEN`) and to nothing in the DDL. A *second* carrier column, `TAKBAE_CD`, exists on `ORDER_DELIVERY` → `src/main/resources/db/migration/V5__add_delivery_columns.sql`, `src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[GLOSSARY-DELIVERY]] |
| `JEOKRIPGEUM` | 적립금 — loyalty points used | `DECIMAL(15,0) DEFAULT 0`, added V7 (2022-08-22) "적립금 사용 대응" and **dropped by V20** (2024-09) together with `HALIN_GEUMAEK`, because "적립금은 별도 시스템으로 이관됨" ("points were migrated to a separate system"). The spec still calls the concept `jeokRipGeum` (적립금 사용액); no column backs it now → `src/main/resources/db/migration/V7__add_point_columns.sql`, `src/main/resources/db/migration/V20__drop_unused_point_columns.sql` | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `HALIN_GEUMAEK` | 할인금액 — discount amount | `DECIMAL(15,0) DEFAULT 0`, added V7 with `JEOKRIPGEUM`. The spec's `halinGeumaek` is a `double` with example `3000`, while the column is a zero-scale decimal — a type mismatch between the published contract and the store → `src/main/resources/db/migration/V7__add_point_columns.sql`, `sources/raw/specs/order-service-openapi.json` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| `PARENT_ORD_NO` | 상위 주문번호 — parent order number | `VARCHAR(20) NULL`, added V10 (comment date 2024-01-29; these comment dates are non-monotonic from V15 on and are not a reliable chronology — see [[SCH-ORDER-MIGRATION-HISTORY]]) "주문 분할/병합 대응" ("for order split/merge"), indexed as `IX_ORDER_MST_04`. The `DeliveryInfo` javadoc says split delivery is not supported — "분할배송은 지원하지 않는다 (V10 에서 컬럼만 추가됨)" ("split delivery is not supported; V10 added the column only"). So the term names a capability the schema has and the code does not → `src/main/resources/db/migration/V10__add_split_merge.sql`, `src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `JUNGSAN_RUN_ID` | 정산 실행 ID — settlement run reference | `BIGINT NULL`, added V14 (comment date 2025-11-20, later than V15's and V23's although both are numbered after it — the comment dates are not a chronology) at 정산팀's request. The migration itself demotes it: "실제 정산 여부는 SETTLEMENT_DTL 이 정본이다. 본 컬럼은 캐시 성격." ("`SETTLEMENT_DTL` is the record of truth for whether settlement happened; this column is cache-like"). The only order-side column named after a settlement concept, and no code in any repo writes it → `src/main/resources/db/migration/V14__add_settlement_ref.sql` | [[CON-ORDER-SETTLEMENT]] |

### `ORDER_DTL` — order line items

The V1 DDL and the `OrderItem` entity disagree on every column name except `ORD_NO`. Both spellings are live in the repository; neither is deprecated.

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `ORD_SEQ` | 주문 순번 — line sequence | `INT NOT NULL`, second half of `PRIMARY KEY (ORD_NO, ORD_SEQ)`. The `OrderItem` entity mirrors it as the second `@Id` field of an `@IdClass(OrderItem.Pk)` composite key → `src/main/resources/db/migration/V1__init.sql`, `src/main/java/kr/co/sellflow/order/domain/OrderItem.java` | [[SCH-ORDER]] |
| `SANGPUM_CD` | 상품코드 — product code | `VARCHAR(30) NOT NULL`, mapped by the `OrderItem` entity under the same name. **Means something different next door**: inventory-api calls the same concept `sku` and says so — "order-service 와 용어가 다르다. 주문 도메인의 SANGPUM_CD 는 여기서 sku 다." ("the vocabulary differs from order-service; the order domain's SANGPUM_CD is sku here") → `src/main/resources/db/migration/V1__init.sql`, `inventory-api:app/main.py` | [[CON-ORDER-INVENTORY]], [[GLOSSARY-INVENTORY]] |
| `SURYANG` | 수량 — quantity | `INT NOT NULL`, mapped by the entity as a Java `int`. inventory-api reads `SURYANG` directly out of `ORDER_DTL` and reuses the name on its own `RESTORE_LOG`, and settlement's reader selects it to compute gross → `src/main/resources/db/migration/V1__init.sql`, `inventory-api:app/main.py` | [[CON-ORDER-INVENTORY]] |
| `DANGA` | 단가 — unit price | `DECIMAL(15,0) NOT NULL`, mapped by the entity as `BigDecimal`. Settlement's gross is `DANGA × SURYANG`, computed in `SettlementTarget.getGross()` → `src/main/resources/db/migration/V1__init.sql`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/SettlementTarget.java` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| `PARTNER_ID` | 파트너ID — partner/seller id | `VARCHAR(20) NOT NULL`, indexed as `IX_ORDER_DTL_01`. The partner is a *line-item* attribute, not an order attribute — one order can span partners, and settlement therefore pays per line, not per order. No partner table exists in this schema → `src/main/resources/db/migration/V1__init.sql` | [[CON-ORDER-SETTLEMENT]] |
| `OKSYEON_MYEONG` | 옵션명 — option name | `VARCHAR(200) NULL`, added V4 (2020-06-08) "옵션 상품 대응" ("for option products"). A romanisation of the English loanword 옵션 (option) — the estate's one instance of Korean-romanising a word that was English to begin with → `src/main/resources/db/migration/V4__order_dtl_option.sql` | [[PAT-SELLFLOW-ROMANISED-NAMING]] |
| `GONGGEUP_GA` | 공급가 — supply price | `DECIMAL(15,0) NULL`, added V4 with the option name. The partner's price, as distinct from `DANGA` (what the customer pays). Not read by settlement, which computes gross from `DANGA` → `src/main/resources/db/migration/V4__order_dtl_option.sql` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| `CHANGGO_CD` | 창고코드 — despatch warehouse code | `VARCHAR(10) DEFAULT 'GIMPO'`, added V12 (2024-09-02) "용인센터 오픈 대응" ("for the Yongin centre opening"). The spec enumerates `GIMPO` and `YONGIN`. The same concept is `warehouse_cd` in inventory-api, where it is half the primary key and the restock `UPDATE` ignores it → `src/main/resources/db/migration/V12__add_warehouse.sql`, `inventory-api:sql/V1__stock.sql` | [[GLOSSARY-INVENTORY]] |
| `OPT_AMT` | 옵션 금액 — option surcharge | `BIGINT NOT NULL DEFAULT 0`, added V19. The only `BIGINT` money column in the order schema; every other amount is `DECIMAL(15,0)` → `src/main/resources/db/migration/V19__order_dtl_option_price.sql` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| `GONGGEUP_GA`, `CHANGGO_CD`, `OPT_AMT` | 공급가 / 창고코드 / 옵션금액 | The three `ORDER_DTL` columns added after 2020 (V4, V12, V19) that the `OrderItem` entity does **not** map, so a JPA read of a line item omits the supply price, the warehouse and the option surcharge → `src/main/java/kr/co/sellflow/order/domain/OrderItem.java`, `src/main/resources/db/migration/V19__order_dtl_option_price.sql` | [[RISK-ORDER]] |

### `ORDER_CANCEL` — cancellation record

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `CHWISO_ILSI` | 취소일시 — cancellation date-time | `DATETIME NOT NULL`, stamped from the JVM by the `OrderCancel` constructor (`LocalDateTime.now()`), not by the database. Settlement's counterpart `RECV_DTM` is stamped by the DB up to ten minutes later → `src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` | [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] |
| `CHWISO_SAYU_CD` | 취소사유코드 — cancel reason code | `VARCHAR(2) NOT NULL`, values `01`-`04`. Validated by `CancelReason.of` — which throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` ("unknown cancel reason code") — and then never read again by any logic in this service → `src/main/java/kr/co/sellflow/order/domain/CancelReason.java` | `CancelReason` |
| `CHORI_SANGTAE` | 처리상태 — processing status | `VARCHAR(20) NOT NULL`, hardcoded to the literal `"COMPLETED"` by the `OrderCancel` constructor. A one-value status column that V22 nonetheless indexes as the leading column of `(CHORI_SANGTAE, CHWISO_ILSI)` → `src/main/java/kr/co/sellflow/order/domain/OrderCancel.java`, `src/main/resources/db/migration/V22__order_cancel_status_index.sql` | [[PROC-ORDER-CANCEL]] |
| `BIGO` | 비고 — remark, free-text note | `VARCHAR(500)` in V1, widened to `VARCHAR(2000)` in V9 "취소 비고 길이 확장 (CS 요청)". Renamed to `MEMO` by V15 and renamed straight back by V16 — "V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애." ("rollback of V15; the settlement batch query referenced BIGO directly and caused an incident"). The API request field is also `bigo`. Note the entity still declares `length = 500` → `src/main/resources/db/migration/V16__revert_rename_bigo.sql`, `src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `CHNL_CD` | 취소 접수 채널 — cancel intake channel | `VARCHAR(10) NULL`, added V17 with the column comment `'취소 접수 채널 (APP/ADMIN/CS)'`. Backfilled to `ADMIN`, then partly re-classified to `APP` by V18 on the strength of `m.CHAENNEL_CD` — the *order's* channel, not the cancellation's. Not mapped in the `OrderCancel` entity, so the current code never sets it → `src/main/resources/db/migration/V17__add_cancel_channel.sql`, `src/main/resources/db/migration/V18__backfill_cancel_channel.sql` | [[RISK-ORDER]] |

### `ORDER_EVENT_OUTBOX` and `ORDER_DELIVERY`

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| 아웃박스 / outbox | — | The `ORDER_EVENT_OUTBOX` table and the pattern it implements: the cancel transaction writes an event row, and a poller in another service reads it. V8 is explicit that there is no broker — "정산 배치의 릴레이 잡이 주기적으로 폴링한다. (별도 브로커 없음)" ("the settlement batch's relay job polls it periodically; no separate broker") → `src/main/resources/db/migration/V8__add_cancel_event_outbox.sql` | [[DEC-ORDER-OUTBOX-RELAY]] |
| 릴레이 / relay | — | The consumer half of the outbox: `OrderEventRelayJob` in settlement-batch, registered in `QuartzConfig` on a ten-minute `SimpleSchedule`. The word belongs to the order domain's vocabulary (V8's comment and the publisher's javadoc both use it) but names a class in a different repository, owned by a different division → `src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` | [[CON-ORDER-SETTLEMENT]] |
| `EVENT_TYPE` / `order.cancelled` | — | `VARCHAR(50) NOT NULL`. Exactly one value exists in the estate: the constant `OrderEventPublisher.EVT_ORDER_CANCELLED = "order.cancelled"`, matched by a string literal of the same value in the relay. The two constants are not shared — each repo declares its own → `src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java` | [[CON-ORDER-SETTLEMENT]] |
| `PAYLOAD` | — | `TEXT NOT NULL`. Built by `String.format("{\"ordNo\":\"%s\",\"sayuCd\":\"%s\"}", ordNo, sayuCd)` — hand-formatted JSON with exactly two fields, no timestamp and no amount → `src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java` | [[CON-ORDER-SETTLEMENT]] |
| `PUBLISHED_YN` | 발행여부 — published flag | `CHAR(1) DEFAULT 'N'`. order-service only ever writes `'N'`; settlement-batch's relay flips it to `'Y'`. "Published" here means "relayed", not "delivered to a consumer that acted on it" → `src/main/java/kr/co/sellflow/order/event/OrderEventOutbox.java` | [[DEC-ORDER-OUTBOX-RELAY]] |
| `TAKBAE_CD` | 택배코드 — carrier code | The `ORDER_DELIVERY` column mapped by `DeliveryInfo.takbaeCd`. A fourth spelling of the carrier concept, and the shortest: 택배 is parcel delivery, 택배사 is the carrier company → `src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[GLOSSARY-DELIVERY]] |
| `INVOICE_NO` | 운송장번호 — waybill number | The `ORDER_DELIVERY` column mapped by `DeliveryInfo.invoiceNo`. A false friend: it is a shipping waybill, not a billing invoice → `src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[GLOSSARY-DELIVERY]] |
| `BAESONG_SANGTAE` | 배송상태 — delivery status | The `ORDER_DELIVERY` column mapped by `DeliveryInfo.baesongSangtae`. Its value vocabulary is defined nowhere; delivery-bff's only status literal is `PREPARING` → `src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[GLOSSARY-DELIVERY]] |

### Columns referenced but never created

Two names appear in this repository and are created by no migration V1-V24. Verified by grepping every `.sql` and `.java` file for each name.

| Name | Where referenced | What it looks like |
|---|---|---|
| `ORD_DT` | `OrderSearchService.search` — `WHERE ORD_DT BETWEEN ? AND ?` | a date-only twin of `JUMUN_ILSI` |
| `BAESONG_MSG` | V13's deferral TODO, and the spec as `baesongMsg` (배송 메시지, example "부재 시 경비실" — "leave with the guard if absent") | a delivery instruction field |

Four further names used to sit in this table — `INFLOW_CHNL`, `REG_DT` on `ORDER_CANCEL`,
`SETTLE_REF_NO` and `ORD_MEMO`. Each was a near-miss romanisation of a column that did exist, and
the migrations that used them have been corrected to `CHAENNEL_CD` (V3), `CHWISO_ILSI` (V1),
`JUNGSAN_RUN_ID` (V14) and `GOGAEK_MEMO` (V2) respectively. `REG_DT` is still a real column name in
this schema — on `ORDER_STATUS_HIST`, where V1 creates it and `OrderStatusHistory.regDt` maps it.

### `OrderStatus` — the full order status vocabulary

`SANGTAE_CD` stores the constant *name*; the Korean string is the display label carried alongside as enum data.

| Constant | Label (verbatim) | English | Written by | Notes |
|---|---|---|---|---|
| `GYEOLJE_WANRYO` | "결제완료" | payment complete | nothing in this repo | the nominal initial state; no creation path exists in the codebase |
| `SANGPUM_JUNBI` | "상품준비중" | preparing the goods | nothing in this repo | absent from delivery-bff's generated union |
| `BAESONG_JUNG` | "배송중" | in delivery | nothing in this repo | present in delivery-bff's union |
| `BAESONG_WANRYO` | "배송완료" | delivery complete | nothing in this repo | the state settlement's reader selects on: `WHERE m.SANGTAE_CD = 'BAESONG_WANRYO'` |
| `JUNGSAN_WANRYO` | "정산완료" | settlement complete | settlement-batch's `MarkSettledTasklet` only | the enum warns "이 상태는 정산팀 배치가 설정하며 주문팀에서 직접 변경하지 않는다" ("this status is set by the settlement team's batch and is not changed directly by the order team"); absent from delivery-bff's union |
| `CHWISO` | "취소" | cancelled | `OrderMst.chwiso()` | also blocks further cancellation |
| `BANPUM` | "반품" | returned | nothing in this repo | blocks cancellation; CS팀 owns 반품 접수 (return intake) per the org chart, with no system behind it |

Two derived sets are worth naming:

- **`CHWISO_BULGA`** — 취소 불가 ("cancellation not possible"). Not a column: the `EnumSet` in `OrderCancelService` naming the statuses that refuse a cancellation. Contains exactly `CHWISO` and `BANPUM`, with the javadoc "이미 취소되었거나 반품 프로세스로 넘어간 주문만 차단한다" ("only orders already cancelled or moved to the return process are blocked"). Before SF-2287 this test also covered settled orders → `src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`, `sources/context/tickets/SF-2287.md`
- **`CANCELLED_STATES`** — settlement-anomaly's `{"CHWISO", "BANPUM"}`. The same two values, in a different repository, merged into one anomaly code. Order treats them as two states; the detector treats them as one → `settlement-anomaly:model/detector.py`

Status transitions are not recorded: `ORDER_STATUS_HIST` exists as an entity with the TODO "정산완료 전이는 여기 안 쌓인다. 배치에서 직접 UPDATE 하기 때문." ("the settlement-complete transition is not accumulated here, because the batch UPDATEs directly"), and `OrderCancelService.cancel` does not write a history row either → `src/main/java/kr/co/sellflow/order/domain/OrderStatusHistory.java`.

### Cancel reason codes

Labels are quoted verbatim from `CancelReason`; the extra columns come from the 취소정책 sheet and the Confluence page, which agree on the reasons and add consequences the code does not carry.

| Code | Enum constant | Label (verbatim) | English | Cost bearer (취소정책 sheet) | 재고 복원 / restock (sheet) | Restocked in code? |
|---|---|---|---|---|---|---|
| `01` | `PARTNER_GWICHAEK` | "파트너 귀책" | partner at fault — sheet adds 재고부족·출고지연 (stock-out, despatch delay) | 파트너 (partner) | O | yes |
| `02` | `SYSTEM_ORYU` | "시스템 오류" | system error | 셀플로우 (Sellflow) | O | yes |
| `03` | `GOGAEK_BYEONSIM` | "고객 변심" | customer changed their mind | 셀플로우 (Sellflow) | X — "배송 시작 후 취소 시 재고 복원 불가" ("stock cannot be restored when cancelled after delivery has started") | no |
| `04` | `BAESONG_SILPAE` | "배송 실패" | delivery failure — sheet adds 주소불명·수취거부 (address unknown, receipt refused), 물류팀 확인 후 처리 ("handled after the logistics team confirms") | 파트너 (partner) | O | **no** — `RESTOCKABLE_REASONS = {"01","02"}` |
| `00` | *(none)* | — | not a reason code. The literal settlement-batch's relay substitutes when it cannot find `"sayuCd"` in the payload. It appears in no enum, no sheet and no policy page → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` | — | — | — |

All four sheet rows mark 정산 차감 (settlement deduction) as `O`. None of the cost-bearer or restock columns is represented in code, and `CancelReason`'s javadoc says so deliberately: "사유별 비용 부담 주체는 코드에 정의하지 않는다. 정산 정책은 재무본부 소관이며 별도 기준표를 따른다." ("the party bearing the cost per reason is not defined in code; settlement policy belongs to the finance division and follows a separate reference table").

### Domain terms

| Term | Definition | See Also |
|---|---|---|
| 취소 / CHWISO | Cancellation. In this service, precisely three things at once: a transition of `ORDER_MST.SANGTAE_CD` to `CHWISO`, an `ORDER_CANCEL` row keyed on `ORD_NO` (so at most one per order, for ever), and an `order.cancelled` outbox row. All three happen in one `@Transactional` method → `src/main/java/kr/co/sellflow/order/service/OrderCancelService.java` | [[PROC-ORDER-CANCEL]] |
| 반품 / BANPUM | Return. **Not a cancellation**, despite the two being used interchangeably in CS language. It is a separate `OrderStatus`, it blocks cancellation, and no code in this repository ever sets it. Before SF-2287 the return process was the fallback route for a settled order the customer wanted cancelled; the ticket records that this route took "7~10일" (7-10 days) and generated repeat complaints → `src/main/java/kr/co/sellflow/order/domain/OrderStatus.java`, `sources/context/tickets/SF-2287.md` | [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] |
| 결제 취소 / payment cancellation | A third thing again, and explicitly not settlement deduction. `PaymentClient`'s javadoc: "결제 취소와 정산 차감은 별개다. PG 취소가 성공해도 파트너에게 이미 지급된 정산 금액은 되돌아오지 않는다." ("payment cancellation and settlement deduction are separate; even if the PG cancellation succeeds, settlement money already paid to the partner does not come back") → `src/main/java/kr/co/sellflow/order/payment/PaymentClient.java` | [[PROC-SETTLEMENT-CORRECTION]] |

### Terms that shift meaning across services

| Term here | Equivalent elsewhere | Where stated |
|---|---|---|
| `SANGPUM_CD` (order) | `sku` (inventory-api, in both its API and its `stock_item` table) | `inventory-api:sql/V1__stock.sql` — "재고 스키마. 주문 도메인과 용어가 다르다 (SANGPUM_CD ↔ sku)." ("stock schema; the vocabulary differs from the order domain") |
| `SANGPUM_CD` (V1 DDL) | `sku` / `SKU` (inventory-api's `stock_item` and `RESTORE_LOG`) | `inventory-api:sql/V1__stock.sql`, `inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py` — the same product code under three spellings across two services |
| `SURYANG` (order) | `available_qty` / `reserved_qty` (inventory stock) | `inventory-api:README.md` maps it to both with a slash and does not disambiguate |
| `CHANGGO_CD` (order) | `warehouse_cd` (inventory `stock_item` PK) | `src/main/resources/db/migration/V12__add_warehouse.sql`, `inventory-api:sql/V1__stock.sql` |
| `SANGTAE_CD` (order status) | `SANGTAE` (settlement run status), `STATUS` (queue and anomaly) | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` — same word, three entities, three vocabularies |
| `CHWISO_SAYU_CD` (order) | `SAYU_CD` (settlement queue), `reason_code` (inventory), `sayuCd` (event and API) | validated in one place only; see [[GLOSSARY-SELLFLOW]] |

## Related

- [[SYS-ORDER]] — the service whose vocabulary this is
- [[SCH-ORDER]] — where each term appears as a column
- [[SCH-ORDER-MIGRATION-HISTORY]] — when each column entered and, for six of them, left
- [[PROC-ORDER-CANCEL]] — the statuses and reason codes in motion
- [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] — what a "cancellation" is in each of the five services
- [[GLOSSARY-SELLFLOW]] — the cross-service disambiguation table
