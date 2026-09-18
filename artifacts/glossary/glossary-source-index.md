---
id: "GLOSSARY-SOURCE-INDEX"
type: "glossary"
title: "Sellflow Term to Source Index"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Built on 2026-09-19 by enumerating every file in the five repositories and reading all of them, then grepping each candidate identifier back across the whole tree to find its creating statement. Re-verified on 2026-09-19 after correction passes in order-service and inventory-api. It now covers the 14 tables that a DDL statement creates, the 1 that is written by running code and created by nothing (SETTLEMENT_ADJUSTMENT), every enum and status literal, and the business rules that exist only in documents. ORDER_DELIVERY and ORDER_STATUS_HIST moved into the created column, RESTORE_LOG gained real columns, and the four ORDER_DTL naming conflicts were resolved in the DDL's favour. Goes stale the moment a migration is added in either Java repo, an Alembic revision lands in inventory-api, the delivery-bff client is regenerated (SF-4901), or a new document arrives under sources/context."
freshness_triggers:
  - "alembic/versions/"
  - "app/main.py"
  - "app/models.py"
  - "sources/context/business-rules.md"
  - "sources/context/policy/"
  - "sources/context/registry/services.yaml"
  - "sources/raw/exports/README.md"
  - "sources/raw/specs/order-service-openapi.json"
  - "sql/V1__anomaly_schema.sql"
  - "sql/V1__stock.sql"
  - "src/generated/orderApi.ts"
  - "src/main/java/kr/co/sellflow/order/domain/"
  - "src/main/resources/db/migration/"
known_unknowns:
  - "Whether the production schemas match the migrations. order-service V23 reserves a consolidated baseline and states it was never written, so nothing in the repository reconciles the migrations against a live database. Every 'no defining location' row below is therefore a statement about the repositories, not about the database."
  - "Whether inventory-api's sql/V1__stock.sql was ever applied. The repo carries two migration systems side by side — a Flyway-style .sql file and an Alembic chain that now declares the split deliberately — but alembic/env.py provides no runner, so nothing in the repo says what actually ran."
  - "What columns RESTORE_LOG has. The Alembic revision that creates it calls op.create_table('RESTORE_LOG') with no column arguments, so the table is defined by a statement that defines nothing."
  - "Which of the four PARTNER_ID declarations is canonical. All are VARCHAR(20); PARTNER_CONTRACT.PARTNER_ID is the only one that is a primary key, and no code reads that table."
  - "Whether the 2022 OpenAPI export still describes the running API. It was generated 2022-11-04 from a 2.4.0 build; SERVER_VERSION now reads 2.8.14. Fields it defines that no DDL creates may be real, stale or never-built, and this index cannot tell which."
  - "Whether any term in this index has a definition outside the reef — in a Confluence page not snapshotted, a DBA script, or the 정산 어드민. Absence here means absence from the five repos and the sources/ tree, nothing more."
tags:
  - "cross-system"
  - "glossary"
  - "index"
  - "lookup"
  - "provenance"
aliases:
  - "term source index"
  - "where is this defined"
relates_to:
  - type: "refines"
    target: "[[GLOSSARY-DELIVERY]]"
  - type: "refines"
    target: "[[GLOSSARY-INVENTORY]]"
  - type: "refines"
    target: "[[GLOSSARY-ORDER]]"
  - type: "parent"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-DOC-CODE-DRIFT]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORPHANED-COMPONENTS]]"
  - type: "refines"
    target: "[[SCH-ORDER-MIGRATION-HISTORY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
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
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatusHistory.java"
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
    ref: "order-service:src/main/resources/db/migration/V13__cleanup_unused.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V18__backfill_cancel_channel.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V1__init.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V21__add_settlement_ref_index.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V22__order_cancel_status_index.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V23__consolidated_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V24__add_order_memo_search_index.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "The only definition of cost bearer, restock policy and fee tiers anywhere."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
    notes: "The only appearance of EXPECTED_AMT in the reef."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "2022-11-04 export; defines four enums and eleven fields that no DDL creates."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/features.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
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
    ref: "settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
notes: "A lookup table, not prose. Citation form is <repo>:<path> for code and sellflow-docs:<path> for documents. 'Defines' means the statement that brings the name into existence — the CREATE/ALTER that makes a column, the enum constant, the class field, or the document clause that states a rule. Interpretation of each term belongs to the linked artifact."
---

# Sellflow Term to Source Index

## Overview

This artifact answers one question: **where is this name actually defined?** It exists because in 셀플로우 the answer is frequently "nowhere", and frequently "in two places that disagree" — and neither of those answers is visible from the file you happen to be reading.

How to use it. Find the term in the Terms tables below. The **Defines it** column gives the exact file and the kind of statement that creates the name. The **Service** column says which repository owns that statement, which is not always the repository you found the term in. The **Interpreted by** column links to the artifact that explains what the term means; this index deliberately carries no interpretation of its own. Two markers matter more than the rest:

- **CONFLICT** — the name has more than one candidate definition and they differ. Both rows are listed. Do not pick one without reading both.
- **The final table, "No defining location"** — names that are referenced, indexed, queried, summed or relied on, and created by nothing in the five repositories or the `sources/` tree. These rows are the most valuable content here. A reader who assumes a referenced name exists will write a query that returns an error, or worse, a migration that silently never applied.

How it was built. Every file in `/repos/{order-service,settlement-batch,settlement-anomaly,inventory-api,delivery-bff}` was enumerated and read — 24 Flyway migrations in order-service, 6 in settlement-batch, 2 standalone DDL files, 2 Alembic revisions, every Java, Python and TypeScript source, and every README, and every document under `sources/context/` and `sources/raw/`. Each candidate identifier was then grepped back across the whole repository tree to locate its creating statement: table names (`ORDER_DELIVERY`, `ORDER_STATUS_HIST`, `SETTLEMENT_ADJUSTMENT`), suspected phantom columns (`ORD_DT`, `REG_DT`, `BAESONG_MSG`, `EXPECTED_AMT`), and cross-cutting concepts (`PARTNER`, `sayu`, `sku`). A name is recorded as undefined only when that grep returned no creating statement anywhere. On the 2026-09-19 re-run, eight names that had been recorded as undefined resolved to creating statements: `ORDER_DELIVERY` and `ORDER_STATUS_HIST` are created by order-service V1, `TEMP_FLAG` by V1 and dropped by V13, and `INFLOW_CHNL`, `ORDER_CANCEL.REG_DT`, `SETTLE_REF_NO`, `ORD_MEMO` and the `JEOKLIP_*` pair were near-miss spellings that the migrations now write correctly.

One limit, stated up front. order-service's `V23__consolidated_schema.sql` is a baseline whose body is entirely comments and which states that the consolidated dump was never written — "아직 작성하지 않았다" — so new environments keep applying V1 onward. This index therefore describes what the repositories define, not what the production database contains, and the two may differ.

## Key Facts

- Fourteen tables are created by a DDL statement somewhere in the five repositories: `ORDER_MST`, `ORDER_DTL`, `ORDER_DELIVERY`, `ORDER_STATUS_HIST`, `ORDER_CANCEL`, `ORDER_EVENT_OUTBOX`, `SETTLEMENT_RUN`, `SETTLEMENT_DTL`, `CANCEL_RECON_QUEUE`, `SETTLEMENT_RUN_LOG`, `PARTNER_CONTRACT`, `stock_item`, `RESTORE_LOG`, `SETTLEMENT_ANOMALY` → `order-service:src/main/resources/db/migration/V1__init.sql`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`, `inventory-api:sql/V1__stock.sql`, `settlement-anomaly:sql/V1__anomaly_schema.sql`
- Exactly one table is written by running code and created by nothing: `SETTLEMENT_ADJUSTMENT`, the target of `CancelReconciler`'s INSERT → `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`
- Three of the fourteen created tables have no reader and no writer anywhere in the five repos: `ORDER_DELIVERY`, `ORDER_STATUS_HIST` and `RESTORE_LOG` → `order-service:src/main/java/kr/co/sellflow/order/repository/DeliveryInfoRepository.java`, `order-service:src/main/java/kr/co/sellflow/order/repository/OrderStatusHistoryRepository.java`, `inventory-api:app/main.py`
- delivery-bff defines no table, no column and no schema of any kind; its only type declarations sit in a file generated on 2022-11-08 that carries the header "자동 생성 파일입니다. 직접 수정하지 마세요." ("this is an auto-generated file; do not edit it directly") → `delivery-bff:src/generated/orderApi.ts`
- The `ORDER_DTL` entity and the `ORDER_DTL` DDL agree on every column the entity maps, under a composite `@IdClass` key matching V1's `PRIMARY KEY (ORD_NO, ORD_SEQ)`; three later columns (`GONGGEUP_GA`, `CHANGGO_CD`, `OPT_AMT`) are simply unmapped → `order-service:src/main/java/kr/co/sellflow/order/domain/OrderItem.java`, `order-service:src/main/resources/db/migration/V1__init.sql`
- `REG_DT` is a real column in exactly one place — `ORDER_STATUS_HIST`, created by order-service V1 — and a phantom in one other: settlement-batch V4 indexes `CANCEL_RECON_QUEUE (STATUS, REG_DT)` on a table whose V1 column is `RECV_DTM` → `order-service:src/main/resources/db/migration/V1__init.sql`, `settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql`
- Two order-side names are referenced and created by nothing: `ORD_DT`, selected on by `OrderSearchService`, and `BAESONG_MSG`, named in V13's deferral TODO and in the 2022 spec → `order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java`, `order-service:src/main/resources/db/migration/V13__cleanup_unused.sql`
- Twenty rows in the final table below name something with no defining location anywhere in the reef; the largest single group is the cancel-consequence vocabulary (cost bearer, deduction, restock for `04`), which exists in one spreadsheet and in no repository → `sellflow-docs:context/business-rules.md`
- No constant is shared across repository boundaries. `order.cancelled` is declared independently as `OrderEventPublisher.EVT_ORDER_CANCELLED` and as a private field of the same name in `OrderEventRelayJob`; `RESTOCKABLE_REASONS` is declared twice inside inventory-api alone, and `main.py`'s copy is the one that governs → `order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `inventory-api:app/config.py`
- Four enums that constrain values exist only in the 2022 OpenAPI export and in no DDL: `chaenNelCd`, `taekBaeSaCd`, `changgoCd`, `chulGoSangtae`. The columns they correspond to, where a column exists at all, carry no constraint → `sellflow-docs:raw/specs/order-service-openapi.json`

## Terms

### Tables

| Term | Service | Defines it | Kind | Interpreted by |
|---|---|---|---|---|
| `ORDER_MST` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `CREATE TABLE`, 2019-06 | [[SCH-ORDER]] |
| `ORDER_DTL` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `CREATE TABLE` | [[SCH-ORDER]] |
| `ORDER_CANCEL` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `CREATE TABLE` | [[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]] |
| `ORDER_DELIVERY` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `CREATE TABLE`; no reader, no writer | [[SCH-ORDER]] |
| `ORDER_STATUS_HIST` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `CREATE TABLE`; no reader, no writer | [[SCH-ORDER]] |
| `ORDER_EVENT_OUTBOX` | order-service | `order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql` | `CREATE TABLE`, SF-2287 | [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] |
| `SETTLEMENT_RUN` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `CREATE TABLE` | [[PROC-SETTLEMENT-RUN-LIFECYCLE]] |
| `SETTLEMENT_DTL` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `CREATE TABLE` | [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] |
| `CANCEL_RECON_QUEUE` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `CREATE TABLE` | [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] |
| `SETTLEMENT_RUN_LOG` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql` | `CREATE TABLE`; no writer exists | [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] |
| `PARTNER_CONTRACT` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql` | `CREATE TABLE`; no running reader | [[SCH-SETTLEMENT-BATCH]] |
| `stock_item` | inventory-api | `inventory-api:sql/V1__stock.sql` | `CREATE TABLE`, PK `(sku, warehouse_cd)` | [[SCH-INVENTORY]] |
| `RESTORE_LOG` | inventory-api | `inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py` | `op.create_table`, 6 columns, 2 indexes; no writer | [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] |
| `SETTLEMENT_ANOMALY` | settlement-anomaly | `settlement-anomaly:sql/V1__anomaly_schema.sql` | `CREATE TABLE`, 2025-06 | [[SCH-SETTLEMENT-ANOMALY]] |

### Order columns — created by a migration

| Term | Service | Defines it | Kind | Interpreted by |
|---|---|---|---|---|
| `ORD_NO` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(20)`, PK of three tables, no FK anywhere | [[GLOSSARY-ORDER]] |
| `GOGAEK_ID` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(20) NOT NULL` | [[SCH-ORDER]] |
| `JUMUN_ILSI` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `DATETIME NOT NULL` | [[SCH-ORDER]] |
| `SANGTAE_CD` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(20) NOT NULL`; values from `OrderStatus` | [[GLOSSARY-ORDER]] |
| `CHONG_GEUMAEK` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `DECIMAL(15,0) NOT NULL` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| `BAESONG_JUSO` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(500)`; the schema's only personal-data column | [[SCH-ORDER]] |
| `REG_DTM` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `DATETIME DEFAULT CURRENT_TIMESTAMP` | [[SCH-ORDER]] |
| `UPD_DTM` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `DATETIME`; the settlement reader's date predicate | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| `ORD_SEQ` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `INT NOT NULL`, second half of the `ORDER_DTL` PK | [[SCH-ORDER]] |
| `SANGPUM_CD` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(30) NOT NULL` — CONFLICT, see below | [[CON-ORDER-INVENTORY]] |
| `SURYANG` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `INT NOT NULL` — CONFLICT with the entity's `QTY` | [[CON-ORDER-INVENTORY]] |
| `DANGA` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `DECIMAL(15,0) NOT NULL`; the gross multiplicand | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| `PARTNER_ID` (order) | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(20) NOT NULL` on the *line item* | [[CON-ORDER-SETTLEMENT]] |
| `CHWISO_ILSI` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `DATETIME NOT NULL`, stamped from the JVM | [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] |
| `CHWISO_SAYU_CD` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(2) NOT NULL`; the only validated reason field | [[GLOSSARY-ORDER]] |
| `CHORI_SANGTAE` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql` | `VARCHAR(20) NOT NULL`; one value ever | [[PROC-ORDER-CANCEL]] |
| `BIGO` | order-service | `order-service:src/main/resources/db/migration/V1__init.sql`, widened V9, renamed V15, restored V16 | `VARCHAR(500)` → `VARCHAR(2000)` | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `GOGAEK_MEMO` | order-service | `order-service:src/main/resources/db/migration/V2__add_order_memo.sql` | `ALTER TABLE ADD COLUMN`, 2019-09-11 | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `CHAENNEL_CD` | order-service | `order-service:src/main/resources/db/migration/V3__add_channel_code.sql` | `VARCHAR(20) NULL`, backfilled `'WEB'` | [[GLOSSARY-SELLFLOW]] |
| `OKSYEON_MYEONG` | order-service | `order-service:src/main/resources/db/migration/V4__order_dtl_option.sql` | `VARCHAR(200) NULL`, 2020-06-08 | [[PAT-SELLFLOW-ROMANISED-NAMING]] |
| `GONGGEUP_GA` | order-service | `order-service:src/main/resources/db/migration/V4__order_dtl_option.sql` | `DECIMAL(15,0) NULL` | [[SCH-ORDER]] |
| `UNSONGJANG_BEONHO` | order-service | `order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql` | `VARCHAR(30) NULL` — CONFLICT with `INVOICE_NO` | [[GLOSSARY-DELIVERY]] |
| `TAEKBAESA_CD` | order-service | `order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql` | `VARCHAR(10) NULL` — CONFLICT with `TAKBAE_CD` | [[GLOSSARY-DELIVERY]] |
| `JEOKRIPGEUM` | order-service | `order-service:src/main/resources/db/migration/V7__add_point_columns.sql` | `DECIMAL(15,0) DEFAULT 0`, 2022-08-22 | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `HALIN_GEUMAEK` | order-service | `order-service:src/main/resources/db/migration/V7__add_point_columns.sql` | `DECIMAL(15,0) DEFAULT 0` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| `EVENT_TYPE`, `PAYLOAD`, `PUBLISHED_YN`, `EVENT_ID` | order-service | `order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql` | `CREATE TABLE` columns | [[DEC-ORDER-OUTBOX-RELAY]] |
| `PARENT_ORD_NO` | order-service | `order-service:src/main/resources/db/migration/V10__add_split_merge.sql` | `VARCHAR(20) NULL`, 2024-01-29; the feature is unimplemented | [[SCH-ORDER-MIGRATION-HISTORY]] |
| `CHANGGO_CD` | order-service | `order-service:src/main/resources/db/migration/V12__add_warehouse.sql` | `VARCHAR(10) DEFAULT 'GIMPO'`, 2024-09-02 | [[GLOSSARY-INVENTORY]] |
| `JUNGSAN_RUN_ID` | order-service | `order-service:src/main/resources/db/migration/V14__add_settlement_ref.sql` | `BIGINT NULL`; the migration calls it 캐시 성격 (cache-like) | [[CON-ORDER-SETTLEMENT]] |
| `CHNL_CD` | order-service | `order-service:src/main/resources/db/migration/V17__add_cancel_channel.sql` | `VARCHAR(10) NULL` with an inline `COMMENT` naming its value set | [[GLOSSARY-SELLFLOW]] |
| `OPT_AMT` | order-service | `order-service:src/main/resources/db/migration/V19__order_dtl_option_price.sql` | `BIGINT NOT NULL DEFAULT 0`; the only `BIGINT` money column | [[DEC-SELLFLOW-MONEY-ROUNDING]] |

### Settlement, inventory and anomaly columns

| Term | Service | Defines it | Kind | Interpreted by |
|---|---|---|---|---|
| `RUN_ID` (numeric) | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `BIGINT AUTO_INCREMENT` — CONFLICT with the log table's `VARCHAR(32)` | [[GLOSSARY-SETTLEMENT]] |
| `JUNGSAN_ILJA` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `DATE NOT NULL` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| `SANGTAE` (run) | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `VARCHAR(20) NOT NULL`; only `'RUNNING'` observed, in a subquery | [[GLOSSARY-SETTLEMENT]] |
| `TOTAL_AMT` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `DECIMAL(18,0)`; never written | [[SCH-SETTLEMENT-BATCH]] |
| `JUNGSAN_AMT` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `DECIMAL(15,0) NOT NULL` | [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] |
| `SUSURYO` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `DECIMAL(15,0) NOT NULL` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| `PARTNER_ID` (settlement) | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `VARCHAR(20) NOT NULL` — CONFLICT: four declarations, no FK | [[CON-ORDER-SETTLEMENT]] |
| `SEQ`, `SAYU_CD`, `RECV_DTM`, `STATUS`, `PROCESSED_DTM` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `CANCEL_RECON_QUEUE` columns | [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] |
| `PROCESSED_AT`, `PROCESSED_BY` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql` | `ALTER TABLE`; the migration says "아직 쓰는 코드는 없다" ("no code uses them yet") | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| `FEE_RATE`, `SETTLE_CYCLE`, `UPD_DT` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql` | `CREATE TABLE` columns; no running reader | [[SCH-SETTLEMENT-BATCH]] |
| `STARTED_AT`, `ENDED_AT`, `ROW_CNT` | settlement-batch | `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql` | `CREATE TABLE` columns; no writer | [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] |
| `sku`, `warehouse_cd`, `available_qty`, `reserved_qty`, `updated_at` | inventory-api | `inventory-api:sql/V1__stock.sql` | `CREATE TABLE` columns; `updated_at` has no `ON UPDATE` | [[SCH-INVENTORY]] |
| `ANOMALY_ID`, `ANOMALY_CD`, `SCORE`, `DETECTED_DTM`, `STATUS`, `REVIEWED_BY`, `REVIEWED_DTM` | settlement-anomaly | `settlement-anomaly:sql/V1__anomaly_schema.sql` | `CREATE TABLE` columns; the last two are never written | [[SCH-SETTLEMENT-ANOMALY]] |

### Enums, constants and literals

| Term | Service | Defines it | Kind | Interpreted by |
|---|---|---|---|---|
| `OrderStatus` (7 constants with Korean labels) | order-service | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java` | Java enum | [[GLOSSARY-ORDER]] |
| `CancelReason` (`01`-`04`) | order-service | `order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java` | Java enum with `of()` validation | [[PROC-ORDER-CANCEL]] |
| `CHWISO_BULGA` | order-service | `order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java` | `EnumSet.of(CHWISO, BANPUM)` | [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] |
| `order.cancelled` | order-service | `order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java` | `public static final String` — CONFLICT: re-declared privately in `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` | [[CON-ORDER-SETTLEMENT]] |
| `"COMPLETED"` | order-service | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` | constructor literal | [[PROC-ORDER-CANCEL]] |
| `CANCELLED_STATES` | settlement-anomaly | `settlement-anomaly:model/detector.py` | `{"CHWISO", "BANPUM"}` | [[PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE]] |
| `CANCELLED_SETTLED`, `AMT_OUTLIER` | settlement-anomaly | `settlement-anomaly:model/detector.py` | emitted code literals | [[SCH-SETTLEMENT-ANOMALY]] |
| `DUP_SETTLE`, `FEE_MISMATCH` | settlement-anomaly | `settlement-anomaly:README.md` | README table only; absent from the detector | [[RISK-SETTLEMENT-ANOMALY]] |
| `FEATURES` | settlement-anomaly | `settlement-anomaly:model/features.py` | list of four names the live detector does not use | [[SYS-SETTLEMENT-ANOMALY]] |
| `'DETECTED'` | settlement-anomaly | `settlement-anomaly:sql/V1__anomaly_schema.sql` | column `DEFAULT`, repeated as an INSERT literal | [[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]] |
| `RESTOCKABLE_REASONS` | inventory-api | `inventory-api:app/main.py` | `{"01", "02"}` — CONFLICT: a second copy in `inventory-api:app/config.py`, imported only by tests | [[DEC-INVENTORY-RESTOCK-BY-REASON]] |
| `'PENDING'` | settlement-batch | `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` | INSERT literal, matching the column default | [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] |
| `'PROCESSED'`, `'CANCEL_CLAWBACK'` | settlement-batch | `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java` | literals in a component no scheduler calls | [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] |
| `DEFAULT_FEE_RATE = 0.12` | settlement-batch | `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java` | `BigDecimal` constant | [[RISK-SETTLEMENT]] |
| `'RUNNING'` | settlement-batch | `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java` | subquery literal; nothing inserts a run row | [[PROC-SETTLEMENT-RUN-LIFECYCLE]] |
| `'PREPARING'`, `stale` | delivery-bff | `delivery-bff:src/index.ts` | response literals on the degraded path | [[PROC-DELIVERY-ERROR-HANDLING]] |
| `MAX_RETRY = 3` | delivery-bff | `delivery-bff:src/deliveryStatus.ts` | module constant; `.env.template`'s `RETRY_COUNT` is never read | [[PROC-DELIVERY-ERROR-HANDLING]] |
| `OrderStatus` union (5 members) | delivery-bff | `delivery-bff:src/generated/orderApi.ts` | generated TypeScript union — CONFLICT: contains `JUMUN_WANRYO`, which is in no Java enum, and omits three that are | [[DEC-DELIVERY-GENERATED-CLIENT]] |
| `chaenNelCd`, `taekBaeSaCd`, `changgoCd`, `chulGoSangtae`, `gyeolJeCd` | — (spec) | `sellflow-docs:raw/specs/order-service-openapi.json` | OpenAPI `enum` declarations; no DDL constrains any of them | [[API-ORDER]] |

### Rules defined only in documents

| Term | Owner | Defines it | Kind | Interpreted by |
|---|---|---|---|---|
| 비용 부담 주체 per reason code | 재무본부 | `sellflow-docs:context/business-rules.md` | 취소정책 sheet column | [[PROC-ORDER-CANCEL]] |
| 재고 복원 per reason code | 재고팀 / 재무본부 | `sellflow-docs:context/business-rules.md` | 취소정책 sheet column — CONFLICT with `RESTOCKABLE_REASONS` on `04` | [[DEC-INVENTORY-RESTOCK-BY-REASON]] |
| 정산 차감 before / after the run | 정산팀 | `sellflow-docs:context/business-rules.md` | two rules beneath the reason table; the first is unimplemented | [[CON-ORDER-SETTLEMENT]] |
| 수수료율 tiers (12.0 / 9.5 / 6.0) | 재무본부 | `sellflow-docs:context/business-rules.md` | 수수료 sheet | [[RISK-SETTLEMENT]] |
| 정산 정정 / 정정 대기 건 | 정산팀 | `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md` | §2.1 and §2.2 definitions | [[PROC-SETTLEMENT-CORRECTION]] |
| ST-F-001 / ST-F-002 | 정산팀 | `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md` | §6 form register | [[PROC-SETTLEMENT-CORRECTION]] |
| 미정정 잔액 ownership | 재무기획팀 | `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md` | §3 responsibility table | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| Service ownership and queue producers/consumers | — | `sellflow-docs:context/registry/services.yaml` | the registry's own claim to be 단일 기준 (the single standard); `CANCEL_RECON_QUEUE`'s consumer is `TODO` | [[PROC-SELLFLOW-OWNERSHIP]] |
| Team → system → process mapping | 인사팀 | `sellflow-docs:context/org-chart.md` | 조직도 sheet | [[PROC-SELLFLOW-OWNERSHIP]] |
| 취소 가능 시점 by order status | 주문팀 | `sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html` | §3 table; superseded by SF-2287 and never updated | [[RISK-SELLFLOW-DOC-DRIFT]] |

### Conflicts — two candidate definitions that differ

| Term | Candidate A | Candidate B | Nature of the conflict |
|---|---|---|---|
| product code | `SANGPUM_CD VARCHAR(30)` → `order-service:src/main/resources/db/migration/V1__init.sql` | `sku` / `SKU` → `inventory-api:sql/V1__stock.sql`, `inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py` | cross-service: one product code under three spellings. inventory-api's README documents the mapping; nothing enforces it |
| quantity | `SURYANG INT` → `order-service:src/main/resources/db/migration/V1__init.sql` | `available_qty` / `reserved_qty` → `inventory-api:sql/V1__stock.sql` | cross-service. inventory-api's own `RESTORE_LOG` then uses `SURYANG`, so both conventions live in one repo |
| restore record shape | `RESTORE_LOG(LOG_SEQ, ORD_NO, SKU, SURYANG, SAYU_CD, REG_DTM)` → `inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py` | `RestoreLog(ord_no, sku_cd, qty, sayu_cd, result)` → `inventory-api:app/models.py` | DDL versus dataclass, same repo. The dataclass adds a `result` field with no column and omits two that exist; neither is exercised |
| carrier code | `TAEKBAESA_CD VARCHAR(10)` on `ORDER_MST` → `order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql` | `TAKBAE_CD` on `ORDER_DELIVERY` → `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | two columns on two tables for one concept; a third spelling, `taekBaeSaCd`, carries the only enum |
| waybill number | `UNSONGJANG_BEONHO VARCHAR(30)` → `order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql` | `INVOICE_NO` → `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | as above; nothing joins or reconciles them, and no code writes either |
| channel vocabulary | `chaenNelCd` enum `WEB/APP_IOS/APP_AND/OPEN_MARKET/API` → `sellflow-docs:raw/specs/order-service-openapi.json` | `CHNL_CD` comment `'취소 접수 채널 (APP/ADMIN/CS)'` → `order-service:src/main/resources/db/migration/V17__add_cancel_channel.sql` | two value sets for one word, neither constrained at the schema level |
| loyalty points | `JEOKRIPGEUM` (V7, dropped by V20) → `order-service:src/main/resources/db/migration/V7__add_point_columns.sql` | `jeokRipGeum` (spec field) → `sellflow-docs:raw/specs/order-service-openapi.json` | the column had a two-year life; the spec field outlived it and is now backed by nothing |
| `RUN_ID` | `BIGINT AUTO_INCREMENT` → `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql` | `VARCHAR(32) PRIMARY KEY` → `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql` | same name, two types, one schema; no join is possible |
| restock for reason `04` | 재고 복원 `O` → `sellflow-docs:context/business-rules.md` | `RESTOCKABLE_REASONS = {"01","02"}` → `inventory-api:app/main.py` | the policy says restore, the code does not |
| `RESTOCKABLE_REASONS` | `inventory-api:app/main.py` (governs behaviour) | `inventory-api:app/config.py` (imported only by tests) | two copies of the same set in one repo; they currently agree, and nothing keeps them agreeing |
| order status set | seven constants → `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java` | five members including `JUMUN_WANRYO` → `delivery-bff:src/generated/orderApi.ts` | a 2022 snapshot of a vocabulary that has since changed; the client has never been regenerated |
| `MoneyUtil.fee` rounding | `HALF_UP` → `order-service:src/main/java/kr/co/sellflow/order/common/MoneyUtil.java` | `FLOOR` → `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java` | identical class name, opposite rule; the live processor calls neither |
| `settlementBaseDate` | yesterday → `order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java` | day-before-yesterday before 02:00 → `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/DateUtil.java` | identical method name, two answers |

### No defining location

Referenced by running code, a migration, an index or a document; created by nothing in the five repositories or in `sources/`. Twenty rows.

| Term | Referenced at | What it would be | Interpreted by |
|---|---|---|---|
| `SETTLEMENT_ADJUSTMENT` | `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java` | the correction-record table. The handover confirms the absence: "테이블이 문서에는 나오는데 실제로 조회가 안 됨" ("the table appears in the documentation but cannot actually be queried") | [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] |
| `ADJ_TYPE` | same INSERT | the adjustment-type column of that table | [[PROC-SETTLEMENT-CORRECTION]] |
| `ORD_DT` | `order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java` | a date-only twin of `JUMUN_ILSI`; every order search runs through it | [[RISK-ORDER]] |
| `REG_DT` (settlement) | `settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql` | a twin of `RECV_DTM`. The name is real on order-service's `ORDER_STATUS_HIST`, which is what makes it easy to miss here | [[SCH-SETTLEMENT-BATCH]] |
| `BAESONG_MSG` | `order-service:src/main/resources/db/migration/V13__cleanup_unused.sql` (a TODO), and `sellflow-docs:raw/specs/order-service-openapi.json` as `baesongMsg` | a delivery instruction field, example 부재 시 경비실 ("leave with the guard if absent") | [[GLOSSARY-DELIVERY]] |
| `EXPECTED_AMT` | `sellflow-docs:raw/exports/README.md` | the amount column the backlog figures are summed from; `CANCEL_RECON_QUEUE` has no amount column at all | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| `"00"` (reason code) | `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` | a fifth reason code, in no enum, sheet or policy page | [[GLOSSARY-SELLFLOW]] |
| the second value of `CHORI_SANGTAE` | `order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` | the rest of a status vocabulary that has one member | [[PROC-ORDER-CANCEL]] |
| 정산 어드민 | `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md` §5, handover §4 | the system of record for corrections, with a five-year retention duty | [[PROC-SETTLEMENT-CORRECTION]] |
| 파트너 포털 | `settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java` | the consumer of a report file the writer does not write | [[RISK-SETTLEMENT]] |
| eta-predictor | `sellflow-docs:context/org-chart.md` | a 데이터팀 system since 2024-03, absent from the service registry and from this reef | [[PROC-SELLFLOW-OWNERSHIP]] |
| 판정 기준 | `sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md`, `sellflow-docs:context/sprints/2026-S17_log.md` | the criteria an automated correction would judge by; recorded as TBD in 2026-06 and as 착수 못함 ("could not be started") in 2026-09 | [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]] |
| SF-4901 | `delivery-bff:src/orderClient.ts` | the ticket to regenerate the 2022 client; no ticket file exists under `sources/` | [[DEC-DELIVERY-GENERATED-CLIENT]] |

## Related

- [[GLOSSARY-SELLFLOW]] — the cross-service glossary this index is the lookup layer for
- [[GLOSSARY-ORDER]] — interpretation of the order-side rows
- [[GLOSSARY-SETTLEMENT]] — interpretation of the settlement and anomaly rows
- [[GLOSSARY-INVENTORY]] — interpretation of the stock rows
- [[GLOSSARY-DELIVERY]] — interpretation of the carrier and waybill rows
- [[SCH-ORDER-MIGRATION-HISTORY]] — when each order column entered, and which six never did
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — the components behind the "defined, never called" rows
- [[PAT-SELLFLOW-DOC-CODE-DRIFT]] — the pattern behind the "documents only" and CONFLICT rows
