---
id: "GLOSSARY-SELLFLOW"
type: "glossary"
title: "Sellflow Unified Glossary"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by reading all five repositories end to end — every migration in order-service (V1-V24) and settlement-batch (V1-V6), the inventory and anomaly DDL, both Alembic revisions, every Java, Python and TypeScript source file — and every document under sources/context and sources/raw. The disambiguation table now carries a defining file for each service's version of a shared term, and the two absence sections list terms that exist on only one side. Re-verified on 2026-09-19 after correction passes in order-service and inventory-api: the spelling variants that were phantoms (INFLOW_CHNL, SETTLE_REF_NO, ORD_MEMO, ORDER_CANCEL.REG_DT, JEOKLIP_AMT) are gone from the migrations, the OrderItem entity now uses the DDL names, and inventory-api's RESTORE_LOG introduces the order domain's SURYANG and SAYU_CD into the inventory repo. Goes stale if any DDL renames a column, if OrderStatus or CancelReason changes, if the delivery-bff generated client is regenerated (SF-4901), or if a romanisation convention is written down anywhere."
freshness_triggers:
  - "app/config.py"
  - "app/main.py"
  - "model/detector.py"
  - "sources/context/business-rules.md"
  - "sources/context/org-chart.md"
  - "sources/context/policy/정산_정정_업무절차_v1.1.md"
  - "sources/context/registry/services.yaml"
  - "sources/raw/specs/order-service-openapi.json"
  - "sql/V1__stock.sql"
  - "src/generated/orderApi.ts"
  - "src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - "src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "src/main/resources/db/migration/V1__init.sql"
  - "src/main/resources/db/migration/V1__settlement_schema.sql"
known_unknowns:
  - "Whether 파트너 (partner) and 셀러 (seller) name the same population. PARTNER_ID is a column in two schemas; 셀러 appears only in the org chart, as the work of 파트너지원팀 — 셀러 온보딩 · 셀러 문의. No document or code defines either, no partner or seller registry exists, and PARTNER_CONTRACT holds only a fee rate and a cycle."
  - "Whether the four PARTNER_ID columns are one identifier space. ORDER_DTL.PARTNER_ID, SETTLEMENT_DTL.PARTNER_ID and PARTNER_CONTRACT.PARTNER_ID are all VARCHAR(20) with no FK between them; settlement-anomaly reads a fourth copy through a join. Checked every migration in all five repos: no seed data, no lookup table, no key registry."
  - "The intended romanisation scheme. Ten concepts carry two or more spellings across the estate and one carries five. No convention document, style guide, linter rule or PR template clause exists in any repo or in sources/."
  - "Whether 취소 (cancellation) and 반품 (return) are one concept or two, company-wide. order-service treats them as two OrderStatus values; settlement-anomaly merges them into CANCELLED_STATES; the Confluence policy routes one into the other; the 2026-06-18 kickoff minutes exclude 환불/반품 from the automation scope without defining either."
  - "What 환불 (refund) denotes as a system behaviour. The Confluence policy promises 전액 환불 (full refund) for two order states, PaymentClient cancels at the PG with a TODO saying 부분취소 미지원 (partial cancellation unsupported), and no repository has a refund record, table or amount field."
  - "Whether the four cancel reason codes carry the same meaning in every service. The labels agree between CancelReason and the 취소정책 sheet, but only order-service validates the code; settlement-batch parses it by byte offset and substitutes \"00\" on failure, and inventory-api keys restock behaviour on it while disagreeing with the sheet on 04."
  - "Which of the several 상태/status vocabularies a reader of a cross-service report is looking at. Nine distinct status fields exist across the five repos with nine distinct value sets, and no document names more than one of them."
  - "Whether 정산 어드민 (settlement admin) and 파트너 포털 (partner portal) are systems, screens of another system, or spreadsheets. Both are named as places where work happens; neither appears in the service registry or as a repository in this reef."
tags:
  - "cross-system"
  - "glossary"
  - "korean"
  - "romanisation"
  - "terminology"
aliases:
  - "unified glossary"
  - "셀플로우 용어"
relates_to:
  - type: "refines"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]]"
  - type: "refines"
    target: "[[GLOSSARY-DELIVERY]]"
  - type: "refines"
    target: "[[GLOSSARY-INVENTORY]]"
  - type: "refines"
    target: "[[GLOSSARY-ORDER]]"
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[GLOSSARY-SOURCE-INDEX]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE]]"
  - type: "depends_on"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-OWNERSHIP]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
sources:
  - category: "documentation"
    type: "github"
    ref: "delivery-bff:README.md"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
  - category: "documentation"
    type: "github"
    ref: "inventory-api:README.md"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/common/MoneyUtil.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderItem.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java"
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
    ref: "order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "Sheets 취소정책 and 수수료 — the only place cost bearer, restock and fee tiers are stated."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "Division, team and process vocabulary; the 셀러/파트너 split."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:README.md"
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
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/DateUtil.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql"
notes: "Cross-service registry. The per-service glossaries hold the full local vocabulary; this one resolves the collisions and records the terms that exist on only one side of the document/code line. Korean is quoted verbatim from the source with an English rendering alongside; no Korean text here is a back-translation of English."
---

# Sellflow Unified Glossary

## Overview

셀플로우 is five services written by four divisions over seven years, and it has no shared vocabulary. That is not a stylistic observation — it is the operating condition an agent has to work in. The same Korean word is romanised three ways in three repositories; the same English word names three different things; and the terms that carry the most money are the ones defined in a spreadsheet and nowhere in code.

This glossary holds three kinds of entry, and the order matters:

1. **Disambiguations.** A concept that appears in two or more services with a different definition, lifecycle or field shape. Reading one service's version and assuming the others match is the single most productive source of wrong conclusions in this estate. Each row names the defining file per service.
2. **Company vocabulary.** The Korean business terms used in the procedures, minutes, handover, org chart and tickets, quoted verbatim with their meaning and with a note on whether any code implements them.
3. **Absences.** Terms that exist only in documents, and terms that exist only in code. Both are findings. A term with no code is work a person does by hand; a term with no business definition is behaviour nobody agreed to.

For the per-service vocabulary go to [[GLOSSARY-ORDER]], [[GLOSSARY-SETTLEMENT]], [[GLOSSARY-INVENTORY]] and [[GLOSSARY-DELIVERY]]. For the question "where is this term defined?" go to [[GLOSSARY-SOURCE-INDEX]], which is the lookup table behind this prose. For why the identifiers look the way they do, [[PAT-SELLFLOW-ROMANISED-NAMING]].

One warning before the tables. Two near-homophone words carry the whole estate's money story, and romanisation hides the difference: **정산** (jungsan) is *settlement*, the nightly run that pays partners; **정정** (jeongjeong) is *correction*, undoing money a settlement already paid. They share no character. A document titled 정산 정정 업무절차 is a correction procedure.

## Terms

### 1. Disambiguation — one concept, several services

#### 취소 / cancel

The word does not survive a service boundary intact. Five services, five objects.

| Service | What "cancel" is there | Cardinality | Defining file |
|---|---|---|---|
| order-service | A transaction doing three things at once: `ORDER_MST.SANGTAE_CD` → `CHWISO`, an `ORDER_CANCEL` row, an `order.cancelled` outbox row. Blocked only when the current status is `CHWISO` or `BANPUM` | at most one per order, for ever — `ORD_NO` is the `ORDER_CANCEL` primary key | `order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`, `order-service:src/main/resources/db/migration/V1__init.sql` |
| settlement-batch | Money to be clawed back. A `CANCEL_RECON_QUEUE` row, inserted only when `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` is greater than zero — so a cancellation of an unsettled order is not a cancellation here at all | unbounded — the PK is `SEQ`, and `ORD_NO` has no unique key | `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` |
| settlement-anomaly | A state, not an event: `CANCELLED_STATES = {"CHWISO", "BANPUM"}`. Cancellation and return are one thing here, and finding either in a settlement emits `CANCELLED_SETTLED` with a fixed `"score": 1.0` | one anomaly row per settled-and-cancelled order per run | `settlement-anomaly:model/detector.py` |
| inventory-api | A reason code arriving on `POST /stock/restock`. If it is `01` or `02`, `available_qty` goes up; otherwise the response is `{"restocked": False, "reason": "not_restockable"}`. No record of the cancellation is kept — `RESTORE_LOG` is never written | no record at all | `inventory-api:app/main.py` |
| delivery-bff | An outbound call through a client generated in 2022, whose own comment promises "정산이 완료된 주문은 취소할 수 없습니다." ("an order whose settlement is complete cannot be cancelled") and throws on HTTP 409. That rule was removed in 2023 by SF-2287. `requestCancel` is exported and called from nowhere | none | `delivery-bff:src/generated/orderApi.ts`, `delivery-bff:src/orderClient.ts` |

Documents add two more senses. The Confluence policy page (last modified 2021-03-17) states 정산완료 · 취소 불가 ("settlement complete — cancellation not possible") and routes such customers to 반품, which SF-2287 removed; a 2024-08-19 comment on that page asks whether it is still valid and is unanswered. The 2026-06-18 kickoff minutes put 환불/반품 (refund/return) outside the automation scope without saying what separates them from 취소. See [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] and [[PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE]].

#### 정산 / settlement

| Service | What "settlement" denotes | Defining file |
|---|---|---|
| settlement-batch | The nightly run and its output: a `SETTLEMENT_RUN` row and one `SETTLEMENT_DTL` row per order, `JUNGSAN_AMT` = gross − `SUSURYO`. Daily at 02:00 KST by Quartz cron `"0 0 2 * * ?"` | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java` |
| order-service | A *status* on someone else's order — `JUNGSAN_WANRYO` — plus `ORDER_MST.JUNGSAN_RUN_ID`, which V14 itself demotes: "실제 정산 여부는 SETTLEMENT_DTL 이 정본이다. 본 컬럼은 캐시 성격." ("`SETTLEMENT_DTL` is the record of truth for whether settlement happened; this column is cache-like"). Neither is written by order-service code | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java`, `order-service:src/main/resources/db/migration/V14__add_settlement_ref.sql` |
| settlement-anomaly | The set of `SETTLEMENT_DTL` rows joined to `SETTLEMENT_RUN` for one `JUNGSAN_ILJA` — a day's worth of settlement as input data, never as an action | `settlement-anomaly:app/main.py` |
| business documents | A *consequence*: 정산 차감 (settlement deduction), a column in the 취소정책 sheet marked `O` for all four cancel reasons, with the rule 정산 실행 전 취소 → 해당 주문을 정산 대상에서 제외 ("cancelled before the run → exclude the order from the settlement target"). Not implemented — the reader's predicate ignores cancellation entirely | `sellflow-docs:context/business-rules.md`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java` |

#### 파트너 / partner

| Where | What it is | Defining file |
|---|---|---|
| order-service | `ORDER_DTL.PARTNER_ID VARCHAR(20) NOT NULL`, indexed as `IX_ORDER_DTL_01`. A **line-item** attribute, so one order can span partners — which is why settlement pays per line, not per order. No partner table | `order-service:src/main/resources/db/migration/V1__init.sql` |
| settlement-batch | `SETTLEMENT_DTL.PARTNER_ID VARCHAR(20) NOT NULL`, the payee; and `PARTNER_CONTRACT (PARTNER_ID PK, FEE_RATE DECIMAL(5,4), SETTLE_CYCLE VARCHAR(10) DEFAULT 'DAILY')`, which the running processor does not read — it hardcodes `new BigDecimal("0.12")` | `settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java` |
| settlement-anomaly | A grouping key selected through a join (`d.PARTNER_ID`), plus the unused feature name `partner_daily_count`. The live detector scores only `[jungsan_amt, susuryo]`, so partner identity never reaches the model | `settlement-anomaly:app/main.py`, `settlement-anomaly:model/features.py` |
| inventory-api, delivery-bff | **Absent.** Grepping both repositories for `partner` or `PARTNER` returns nothing | — |
| org chart | A population supported by 파트너지원팀 (partner support team, CS본부), whose described work is 셀러 온보딩 · 셀러 문의 ("seller onboarding, seller enquiries") — the only appearance of 셀러 (seller) in the estate. Nothing states whether 셀러 and 파트너 are the same people | `sellflow-docs:context/org-chart.md` |
| documents | The recipient of 파트너 통지 (partner notification, procedure v1.1 §4 step 4) and the reader of the 파트너 포털 (partner portal), which reads a report file `SettlementReportWriter` does not actually write | `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java` |

#### 상태 / status

Nine status fields, nine value sets, no shared vocabulary and no lookup table anywhere.

| Field | Owner | Observed values | Defining file |
|---|---|---|---|
| `ORDER_MST.SANGTAE_CD` | order | the seven `OrderStatus` constants, stored as names via `@Enumerated(EnumType.STRING)` | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java` |
| `ORDER_CANCEL.CHORI_SANGTAE` | order | `"COMPLETED"` only, hardcoded in the constructor | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` |
| `ORDER_DELIVERY.BAESONG_SANGTAE` | order (entity only) | none — nothing reads or writes it, and no migration creates the table | `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` |
| `SETTLEMENT_RUN.SANGTAE` | settlement | `'RUNNING'` only, and only as a subquery predicate; no code inserts a run row | `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java` |
| `SETTLEMENT_RUN_LOG.STATUS` | settlement | none — the table has no writer | `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql` |
| `CANCEL_RECON_QUEUE.STATUS` | settlement | `'PENDING'` written by the relay; `'PROCESSED'` only in `CancelReconciler`, which is registered in no Quartz trigger. The 2026-09-01 export confirms: "STATUS 가 PENDING 외의 값을 가진 행은 조회되지 않았다" ("no row with a STATUS other than PENDING was returned") | `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `sellflow-docs:raw/exports/README.md` |
| `SETTLEMENT_ANOMALY.STATUS` | anomaly | `'DETECTED'`, both as column default and as an explicit literal. `REVIEWED_BY` and `REVIEWED_DTM` exist and are never written | `settlement-anomaly:sql/V1__anomaly_schema.sql` |
| `DeliveryStatus.status` | delivery | `'PREPARING'`, the fabricated value returned after three failed carrier calls, alongside `stale: true` and HTTP 200 | `delivery-bff:src/index.ts` |
| `RestoreLog.result` | inventory | the same idea under an English name that is not "status". Never populated — the dataclass is imported nowhere | `inventory-api:app/models.py` |

A tenth vocabulary exists on paper only: the 2022 spec's `chulGoSangtae` (출고상태, despatch status) enumerates `WAIT`, `PICKING`, `PACKED`, `SHIPPED`, `DONE`. No column, entity or query in any repository carries it → `sellflow-docs:raw/specs/order-service-openapi.json`.

#### 사유 코드 / reason code

One two-character value, six names and exactly one validator.

| Name | Where | Validated? | Defining file |
|---|---|---|---|
| `CHWISO_SAYU_CD` | `ORDER_CANCEL` column, `VARCHAR(2) NOT NULL` | yes — `CancelReason.of` throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` ("unknown cancel reason code") | `order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java` |
| `sayuCd` | the outbox payload, the cancel API request body, and delivery-bff's `CancelRequest` | no | `order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java`, `delivery-bff:src/generated/orderApi.ts` |
| `SAYU_CD` | `CANCEL_RECON_QUEUE` column | no — extracted by `s.indexOf("\"sayuCd\":\"")` then `s.substring(i + 10, i + 12)`, returning the literal `"00"` when the marker is absent | `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` |
| `reason` | the query parameter order-service sends to inventory | no | `order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java` |
| `reason_code` | inventory's live `RestockRequest` model | membership test only, against `RESTOCKABLE_REASONS = {"01", "02"}` | `inventory-api:app/main.py` |
| `sayu_cd` | inventory's unused `RestoreLog` dataclass | no | `inventory-api:app/models.py` |

`"00"` deserves its own line: it is the value settlement-batch writes when it cannot find `sayuCd` in a payload. It appears in no enum, no policy page and no sheet in the estate, and it is not distinguishable afterwards from a real code.

The **consequences** of a reason code live only in the 취소정책 sheet, and `CancelReason`'s javadoc says that is deliberate: "사유별 비용 부담 주체는 코드에 정의하지 않는다. 정산 정책은 재무본부 소관이며 별도 기준표를 따른다." ("the party bearing the cost per reason is not defined in code; settlement policy belongs to the finance division and follows a separate reference table"). The sheet and the code disagree on one row:

| Code | Label (verbatim) | English | 비용 부담 주체 / cost bearer | 재고 복원 / restock per sheet | restocked in code? |
|---|---|---|---|---|---|
| `01` | "파트너 귀책" | partner at fault (재고부족·출고지연 — stock-out, despatch delay) | 파트너 | O | yes |
| `02` | "시스템 오류" | system error | 셀플로우 | O | yes |
| `03` | "고객 변심" | customer changed their mind | 셀플로우 | X — "배송 시작 후 취소 시 재고 복원 불가" | no |
| `04` | "배송 실패" | delivery failure (주소불명·수취거부 — address unknown, receipt refused) | 파트너 | O | **no** — `RESTOCKABLE_REASONS = {"01","02"}` |

→ `sellflow-docs:context/business-rules.md`, `inventory-api:app/main.py`. See [[DEC-INVENTORY-RESTOCK-BY-REASON]] and [[CON-ORDER-INVENTORY]].

#### 채널 / channel

| Name | Value vocabulary | Where stated |
|---|---|---|
| `ORDER_MST.CHAENNEL_CD` | `VARCHAR(20) NULL`, backfilled to `'WEB'`, no constraint | `order-service:src/main/resources/db/migration/V3__add_channel_code.sql` |
| `chaenNelCd` (spec) | `WEB`, `APP_IOS`, `APP_AND`, `OPEN_MARKET`, `API` — described as 유입채널코드 (inflow channel code) | `sellflow-docs:raw/specs/order-service-openapi.json` |
| `ORDER_CANCEL.CHNL_CD` | a *different* set, per its own column comment `'취소 접수 채널 (APP/ADMIN/CS)'` ("cancel intake channel"). Backfilled to `ADMIN`, then partly re-classified to `APP`. Not mapped on the `OrderCancel` entity, so live code never sets it | `order-service:src/main/resources/db/migration/V17__add_cancel_channel.sql` |

Nothing reconciles the two vocabularies. V18's backfill bridges them in one direction only — it copies `ORDER_MST.CHAENNEL_CD = 'APP'` into `ORDER_CANCEL.CHNL_CD`, writing an *order* channel into a *cancellation* channel column whose declared value set is different.

#### SKU 대 SANGPUM_CD — the product identifier

Five spellings; the only estate-wide mapping statement is a README table in the *receiving* service.

| Spelling | Where | Defining file |
|---|---|---|
| `SANGPUM_CD` | `ORDER_DTL` column, `VARCHAR(30) NOT NULL`; also `SettlementTarget.sangpumCd`, mapped straight out of the order table | `order-service:src/main/resources/db/migration/V1__init.sql`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/SettlementTargetRowMapper.java` |
| `sangpumCd` | spec field on `OrderDtl`, described 상품코드 | `sellflow-docs:raw/specs/order-service-openapi.json` |
| `SKU` | `RESTORE_LOG` column in inventory-api, upper-cased — the inventory spelling under the order domain's casing convention | `inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py` |
| `sku` | `stock_item` column and first half of its PK. inventory-api states the equivalence three times, in its README, its DDL and its module docstring: "주문 도메인의 SANGPUM_CD 는 여기서 sku 다." ("the order domain's SANGPUM_CD is sku here") | `inventory-api:sql/V1__stock.sql`, `inventory-api:app/main.py` |
| `sku_cd` | the unused `Stock` / `RestoreLog` dataclasses and the unmounted router's `skuCd` response key | `inventory-api:app/models.py`, `inventory-api:app/routers.py` |

The mapping is stated only in prose, by one of the two parties, and never in a machine-readable form. The same holds for quantity: `SURYANG` (order DDL, and the `OrderItem` entity) → `available_qty` / `reserved_qty` (inventory's `stock_item`) → `SURYANG` again (inventory's own `RESTORE_LOG`), where the README's bridge row writes both `stock_item` columns with a slash and declines to say which.

#### Utilities with identical names and different behaviour

| Name | order-service | settlement-batch | Consequence |
|---|---|---|---|
| `MoneyUtil.fee` | `RoundingMode.HALF_UP` | `RoundingMode.FLOOR` | Both files flag it; settlement's javadoc adds "2022 협의 결과이며 문서화되어 있지 않다." ("the result of a 2022 agreement, and it is not documented"). The live `SettlementItemProcessor` calls neither — it rounds `HALF_UP` inline → `order-service:src/main/java/kr/co/sellflow/order/common/MoneyUtil.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java` |
| `DateUtil.settlementBaseDate` | always yesterday | the day *before* yesterday when the hour is under 2 | Two answers to 정산 기준일 (settlement base date) depending on which jar you are in → `order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/DateUtil.java` |
| `RUN_ID` | `ORDER_MST.JUNGSAN_RUN_ID BIGINT` | `SETTLEMENT_RUN.RUN_ID BIGINT` versus `SETTLEMENT_RUN_LOG.RUN_ID VARCHAR(32)` | Three columns, two types, no join between any pair. settlement-anomaly selects `d.RUN_ID` but its own `SETTLEMENT_ANOMALY` table has no such column → `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql`, `settlement-anomaly:sql/V1__anomaly_schema.sql` |

See [[DEC-SELLFLOW-MONEY-ROUNDING]] and [[PROC-SELLFLOW-RUNTIME]].

### 2. Company-wide Korean business vocabulary

Terms taken verbatim from the documents under `sources/context/` and `sources/raw/`. The last column says whether any of the five repositories implements the term — an empty answer is the finding.

| Term (verbatim) | Meaning | Where stated | In code? |
|---|---|---|---|
| 셀플로우 | Sellflow, the company and the platform. Also the cost bearer for reasons 02 and 03 in the 취소정책 sheet ("셀플로우 부담" — borne by Sellflow) | `sellflow-docs:context/business-rules.md` | only as the package root `kr.co.sellflow` and the schema name `sellflow_order` |
| 본부 / 팀 | Division / team. The org chart's two-level structure; six 본부 and sixteen 팀, 248 people | `sellflow-docs:context/org-chart.md` | as `owner_team` values in the registry only |
| 담당 시스템 / 담당 프로세스 | "System owned" / "process owned" — the org chart's two ownership columns. Five of sixteen teams own a system; eleven own only processes | `sellflow-docs:context/org-chart.md` | no |
| 이관 | Transfer of ownership between teams. 재고팀 moved to 데이터플랫폼본부 in 2022 ("2022년 이관"); delivery-bff's transfer is still open — `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` ("transfer to the logistics team under discussion, not yet confirmed") | `sellflow-docs:context/org-chart.md`, `sellflow-docs:context/registry/services.yaml` | echoed in `inventory-api:README.md` and `inventory-api:app/routers.py` ("이관 예정, 2023부터") |
| 협의 | An inter-team agreement. Load-bearing three times: the 2019 통합 DB 정책 agreement that lets settlement UPDATE `ORDER_MST`, the 2019 no-FK design decision, and the undocumented 2022 rounding agreement | `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`, `order-service:src/main/resources/db/migration/V1__init.sql` | as comments only; no artefact records any 협의 |
| 귀책 | Attribution of fault. 파트너 귀책 = the partner is at fault | `sellflow-docs:context/business-rules.md` | yes — `CancelReason.PARTNER_GWICHAEK` |
| 비용 부담 주체 | The party bearing the cost of a cancellation | `sellflow-docs:context/business-rules.md` | no, deliberately — see `CancelReason`'s javadoc |
| 재고 복원 | Restoring cancelled units to sellable stock | `sellflow-docs:context/business-rules.md`, Confluence §5 | yes — `inventory-api:app/main.py`, English in the endpoint (`/stock/restock`), Korean in the log lines |
| 출고 / 입고 / 피킹 / 패킹 | Despatch / inbound / picking / packing. The four processes of both 물류운영본부 centres (165 people, the largest headcount in the company) | `sellflow-docs:context/org-chart.md` | only as the spec's unimplemented `chulGoSangtae` enum and the 출고창고코드 gloss on `changgoCd` |
| 라스트마일 | Last mile. 배송관리팀's process, alongside 배송사 관리 (carrier management) | `sellflow-docs:context/org-chart.md` | no |
| 지급 요청 | Payment request — the irreversible transmission. "지급 요청이 전송되면 되돌릴 수 없다. 은행 이체는 익영업일에 실행된다." ("once sent it cannot be undone; the bank transfer executes the next business day") | `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, handover §5 | no transmitting code exists in any repo; the writer INSERTs a row and logs |
| 익영업일 | The next business day. The window in which a payment becomes unrecoverable | `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java` | no calendar, holiday table or business-day utility exists |
| 차월 차감 / 차월 상계 | Next-month deduction / next-month offset. 차감 is one-sided; 상계 is mutual offsetting, the word the 2025-07-12 incident used for recovering a duplicate payout | handover §5, `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md` | one English literal, `ADJ_TYPE='CANCEL_CLAWBACK'`, in code that never runs |
| 수기 | By hand. 수기 정정 (manual correction), 수기 차감 (manual deduction) — the org chart gives 정산팀 the process 정산 오류 수기 정정 | `sellflow-docs:context/org-chart.md`, `sellflow-docs:context/business-rules.md` | by definition not; and the handover records what it became: "문의가 오면 그 건만 본다" ("when an enquiry comes in, only that case is looked at") |
| 미정정 잔액 / 미정정 금액 | Uncorrected balance / amount. 미- negates, so "not-yet-corrected". Procedure v1.1 §3 assigns the quarterly check to 재무기획팀; the 2026-08 mail exercises that clause: "차감 대기 상태로 남아 있는 금액이 결산 기준으로 1.8억을 넘습니다" ("the amount remaining pending deduction exceeds 180 million KRW as of the close") | v1.1 §3, `sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml` | no — the figure is derived from the queue, never stored |
| 결산 | Closing of accounts. 분기 결산 (quarterly close) is where the uncorrected balance surfaces; 회계팀 owns 결산 · 세무 | v1.1 §3, `sellflow-docs:context/org-chart.md` | no |
| 부채로 인식 | To recognise as a liability. The decision 재무기획팀 is trying to make about the backlog | `sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml` | no |
| 입점 / 월 거래액 | Onboarding onto the marketplace / monthly transaction volume. The two criteria for the non-default fee tiers — 신규 프로모션 6.0% "입점 3개월 이내", 프리미엄 9.5% "월 거래액 1억 이상" | `sellflow-docs:context/business-rules.md` | no — neither date nor volume is stored anywhere, and the processor applies 12% to everything |
| 대기열 | Queue. The Korean word the documents use for what the schema calls `CANCEL_RECON_QUEUE`; every document-side sentence about the backlog says 대기열, and no document names the table | v0.3 §3, handover §2, minutes §2, sprint records | yes, as the table name — in English |
| 정본 | The authoritative record. Used once, to demote a column: "실제 정산 여부는 SETTLEMENT_DTL 이 정본이다" | `order-service:src/main/resources/db/migration/V14__add_settlement_ref.sql` | the concept has no mechanism — nothing enforces the precedence |
| 백필 | Backfill. V18's re-classification of cancel channels, and the only migration in the estate that reads a nonexistent column | `order-service:src/main/resources/db/migration/V18__backfill_cancel_channel.sql` | yes |
| 발행 | Issued / published. The status of procedure v1.1 ("상태: 발행"), and the concept behind `PUBLISHED_YN` — which in practice means "the relay looked at this row", not "a consumer acted on it" | v1.1 header, `order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql` | yes, with a shifted meaning |
| 서식 / 승인 / 보존 | Form / approval / retention. Procedure v1.1 §6 registers two forms, 팀장 approved, 5년 retention | v1.1 §6 | no — the handover says "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중." ("no official form for recording corrections; each person manages it in their own spreadsheet") |
| 제정 / 개정 이력 | Enactment / revision history. v0.3 was 제정 2023-05-02, v1.1 개정 2024-02-19 | v1.1 §7 | no |
| 인수인계 / 인계자 / 인수자 | Handover / outgoing person / incoming person. The 2025-03 document is marked 작성중 (in progress) and carries a §6 미인계 사항 (items not handed over) | `sellflow-docs:context/handover/2025-03_정산팀_인수인계.md` | no |
| 장애 / 장애 회고 / 재발 방지 | Incident / incident retrospective / recurrence prevention. The 2025-07-12 P2, and the three of its five prevention items still open | `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md` | no incident record exists in any schema |
| 멱등성 키 | Idempotency key. Prevention item from the 2025-07 duplicate-payout incident, 담당 미지정 ("owner unassigned") | same runbook | no |
| 컨슈머 | Consumer, in the messaging sense. "네 컨슈머 붙여놓겠습니다" ("yes, I will attach a consumer"), 2023-04-11 on SF-2287; the registry's own consumer field for `CANCEL_RECON_QUEUE` reads `TODO   # 확인 필요` ("to be confirmed") | `sellflow-docs:context/tickets/SF-2287.md`, `sellflow-docs:context/registry/services.yaml` | the outbox has one; the correction queue has none |
| 어드민 | Admin. 정산 어드민 is where corrections are supposedly recorded and retained; CS 어드민 is one of the two sources of cancel requests | v1.1 §5, `order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java` | neither exists as a repository; the order CORS rule does allow `https://admin.sellflow.co.kr` on `/api/**` |
| 판정 기준 | Decision criteria — what an automated correction would judge a case by. Recorded as absent twice: "기준 문서가 현재 없음. → TBD" in the minutes, and SF-5121 정정 판정 기준 정의 closing a sprint as 할 일 · 착수 못함 ("to do; could not be started") | `sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md`, `sellflow-docs:context/sprints/2026-S17_log.md` | no |
| 전사 AX 로드맵 | Company-wide AX (AI transformation) roadmap, 2026-06-15, owned by 재무기획팀. The reason the correction work became a project | `sellflow-docs:context/org-chart.md`, `sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md` | no |

### 3. Romanisation that changes between services

Each row is one Korean word written more than one way in the codebase. No convention document exists in any repository or in `sources/`; see [[PAT-SELLFLOW-ROMANISED-NAMING]].

| Korean | Spellings in the estate | Repos involved |
|---|---|---|
| 택배(사) — parcel carrier | `TAEKBAESA_CD` (V5 column) · `TAKBAE_CD` (`DeliveryInfo`) · `taekBaeSaCd` (spec, with a five-value enum) · `carrierCd` (BFF interface, unconstrained) | order-service, delivery-bff, spec |
| 채널 — channel | `CHAENNEL_CD` · `chaenNelCd` · `CHNL_CD` (abbreviated, different value set) | order-service, spec |
| 사유 — reason | `CHWISO_SAYU_CD` · `SAYU_CD` · `sayuCd` · `sayu_cd` · `reason_code` · `reason` | all four repos that handle a cancellation |
| 상품 — product | `SANGPUM_CD` · `sangpumCd` · `sku` · `sku_cd` · `SKU` | order-service, inventory-api, settlement-batch, spec |
| 수량 — quantity | `SURYANG` · `suryang` · `qty` · `available_qty` · `reserved_qty` | order-service, inventory-api |
| 수수료 — commission | `SUSURYO` (column) · `susuryo` (detector row key) · `suSuRyo` (spec field on a column that does not exist) · `FEE_RATE` (contract) · `fee` (method) | order-service, settlement-batch, settlement-anomaly, spec |
| 적립금 — loyalty points | `JEOKRIPGEUM` (V7, dropped by V20) · `jeokRipGeum` (spec, now backed by no column) | order-service, spec |
| 상태 — status | `SANGTAE_CD` · `SANGTAE` · `sangtaeCd` · `sangtae_cd` · `STATUS` · `status` · `CHORI_SANGTAE` · `BAESONG_SANGTAE` · `result` | all five repos |
| 옵션 — option (itself an English loanword) | `OKSYEON_MYEONG` (V4, mapped by the entity) · `okSyeonMyeong` (spec) · `OPT_AMT` (V19, unmapped) | order-service, spec |
| 일시 / 일자 — timestamp / date | `_ILSI` (`CHWISO_ILSI`, `JUMUN_ILSI`) · `_DTM` (`REG_DTM`, `UPD_DTM`, `RECV_DTM`, `PROCESSED_DTM`, `DETECTED_DTM`) · `_DT` (`REG_DT` on `ORDER_STATUS_HIST`, real; `ORD_DT` and settlement's `REG_DT`, neither of which exists where it is referenced) · `_AT` (`PROCESSED_AT`, `STARTED_AT`, `ENDED_AT`, `updated_at`) · `_ILJA` (`JUNGSAN_ILJA`) | all five repos |

The `_DT` family is the expensive one, and it is expensive precisely because one member is real. `REG_DT` is a genuine column on order-service's `ORDER_STATUS_HIST` (V1), which makes the settlement-side reference easy to overlook: V4 indexes `CANCEL_RECON_QUEUE (STATUS, REG_DT)` on a table whose V1 column is `RECV_DTM`. `ORD_DT`, selected on by `OrderSearchService`, is created by nothing.

### 4. Terms that exist only in documents

Each has a name, an owner or a consequence in prose, and no implementation anywhere in the five repositories. Verified by grepping every repo for each term and its plausible romanisations.

| Term | What the document says it is | Source |
|---|---|---|
| 정산 어드민 | Where correction history is recorded and retained five years; access requires 팀장 승인 (team lead approval) | v1.1 §5, handover §4 |
| 파트너 포털 | The system a partner reads settlement results in; the handover's access row for it says `TBD` | `settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java`, handover §4 |
| ST-F-001 / ST-F-002 | 정산 정정 요청서 (Settlement Correction Request) and 월간 정정 현황 확인서 (Monthly Correction Review), both 팀장-approved, both retained five years | v1.1 §6 |
| 프리미엄 9.5% / 신규 프로모션 6.0% | Two fee tiers below the 12% base | `sellflow-docs:context/business-rules.md` |
| 반품 프로세스 | The return process. Linked as a sibling document from the Confluence policy; SF-2287 records that it took 7~10일 and generated repeat complaints. `BANPUM` is an `OrderStatus` value no code ever writes | Confluence §4 and §6, SF-2287 |
| 환불 / 전액 환불 | Refund / full refund, promised for two order states | Confluence §3 |
| eta-predictor | A system owned by 데이터팀 since 2024-03, listed beside settlement-anomaly | `sellflow-docs:context/org-chart.md` |
| 상품 등록 · 카테고리 관리 | 상품팀's processes. No product service exists among the five repos | `sellflow-docs:context/org-chart.md` |
| 물류팀 확인 SLA | The confirmation time for cancelling an in-transit order. Asked on the policy page 2021-06-02, answered "확인해서 추가하겠습니다" ("I will check and add it"), still absent | Confluence comments |
| SF-4901 | The ticket to regenerate delivery-bff's 2022 client. Referenced in code as 미착수 (not started); no ticket file exists in `sources/` | `delivery-bff:src/orderClient.ts` |
| `EXPECTED_AMT` | A column summed by the 2026-09-01 export query (`SUM(EXPECTED_AMT)`). No migration in any repo creates it, and `CANCEL_RECON_QUEUE` as defined has no amount column at all | `sellflow-docs:raw/exports/README.md` |
| bin/export-queue.sh | The re-extraction script for the backlog figures, whose own README says "(스크립트 위치 TBD — 현재는 DBA 에게 요청)" ("script location to be determined; currently requested from the DBA") | `sellflow-docs:raw/exports/README.md` |

### 5. Terms that exist only in code

The mirror image: a name that governs behaviour, with no business definition in any document.

| Term | What it does | Where | Nothing defines |
|---|---|---|---|
| `"00"` | The reason code written when the payload parser misses | `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` | its meaning, or how to tell it from a real code afterwards |
| `CHORI_SANGTAE = "COMPLETED"` | The only value the processing-status column has ever held | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` | what a second value would be |
| `ADJ_TYPE = 'CANCEL_CLAWBACK'` | The estate's only English name for 차월 차감, in the one INSERT that targets `SETTLEMENT_ADJUSTMENT` | `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java` | the mapping to the Korean term; and no migration creates the table |
| `PREPARING` + `stale: true` | The fabricated delivery status returned with HTTP 200 after three carrier failures, with no alert — "3회 실패 시 포기한다. 별도 알림은 없다." | `delivery-bff:src/index.ts`, `delivery-bff:src/deliveryStatus.ts` | whether it corresponds to `SANGPUM_JUNBI` (상품준비중); how a client should treat `stale` |
| `reserved_qty` | Half of the inventory quantity pair, returned by the stock endpoint | `inventory-api:sql/V1__stock.sql` | what a reservation is, who makes one, or how it is released — no code writes it |
| `SETTLE_CYCLE DEFAULT 'DAILY'` | The contract's settlement cycle | `settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql` | the other cycles; nothing reads the column |
| `score > 0.85` | The anomaly threshold that decides whether a settlement row is reported | `settlement-anomaly:model/detector.py` | why 0.85; the model has not been retrained since 2024-11 |
| `cancel_after_settle_flag` | A feature name whose own TODO says it "사실상 단독으로 결과를 좌우한다" ("effectively determines the outcome single-handedly") — and which the live detector does not use at all | `settlement-anomaly:model/features.py` | the feature set actually in production |
| `DEFAULT_FEE_RATE = 0.12` | The commission applied to every partner regardless of contract | `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java` | the sheet does name 12.0% as 기본, but scopes it to "계약 미체결 파트너 전체" ("all partners without a contract") — a scope the code ignores |
| `X-Request-Id` / `X-Client-Ver` | Headers on every operation of the 2022 spec | `sellflow-docs:raw/specs/order-service-openapi.json` | any tracing or versioning policy; no repo reads either header |

## Related

- [[GLOSSARY-SOURCE-INDEX]] — the term-to-defining-file lookup behind these tables
- [[GLOSSARY-ORDER]] — the full order vocabulary, including every romanised column
- [[GLOSSARY-SETTLEMENT]] — the full settlement vocabulary, 정산 and 정정 separated
- [[GLOSSARY-INVENTORY]] — the inventory side of the product and quantity bridge
- [[GLOSSARY-DELIVERY]] — the carrier and waybill vocabulary
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — why one word has five spellings
- [[PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE]] — the same disambiguation seen as a cross-service pattern
- [[CON-ORDER-SETTLEMENT]] — the boundary 정산 정정 is supposed to cross
- [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] — what a cancellation is in each of the five services
- [[PROC-SELLFLOW-OWNERSHIP]] — which division owns each term's defining file
- [[RISK-SELLFLOW-DOC-DRIFT]] — where a document defines a term the code contradicts
