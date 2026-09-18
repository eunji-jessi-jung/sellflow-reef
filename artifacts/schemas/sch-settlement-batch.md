---
id: "SCH-SETTLEMENT-BATCH"
type: "schema"
title: "Settlement Batch Data Model"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened 2026-09-19 by re-reading all six Flyway migrations line by line, every JDBC statement in settlement-batch, sources/schemas/settlement/batch/schema.md and the 2026-09-01 queue export. This pass added enum tables for SANGTAE, CANCEL_RECON_QUEUE.STATUS and ADJ_TYPE with their provenance; drew SETTLEMENT_RUN_LOG into the ER diagram and marked SETTLEMENT_ADJUSTMENT as having no DDL anywhere in the five repos; and added three worked join queries — the settled-then-cancelled reconstruction, the backlog aging query the 2026-09-01 export actually ran, and the orphan-detection query that finds SETTLEMENT_DTL rows for cancelled orders. The V4/REG_DT, PROCESSED_AT/PROCESSED_DTM and SETTLEMENT_RUN/SETTLEMENT_RUN_LOG defects were each re-verified against the migration text. Stale as soon as a V7 migration lands or SETTLEMENT_ADJUSTMENT is created somewhere."
freshness_triggers:
  - "src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - "src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - "src/main/resources/db/migration/*.sql"
known_unknowns:
  - "The real definition of SETTLEMENT_ADJUSTMENT. No migration in any of the five repos creates it; only an INSERT in CancelReconciler references it, giving three columns (ORD_NO, SAYU_CD, ADJ_TYPE) and nothing else — no PK, no types, no amount column."
  - "Whether V4 ever executed successfully in any environment. It indexes CANCEL_RECON_QUEUE (STATUS, REG_DT) but V1 defines RECV_DTM, not REG_DT. On MySQL that statement raises ER_KEY_COLUMN_DOES_NOT_EXITS, which would fail the migration and block V5 and V6 behind it. No evidence either way: the 2026-09-01 export filters WHERE STATUS='PENDING' and returned no non-PENDING rows and no V5 columns, so it cannot show whether V4 applied. Unresolved."
  - "Whether SETTLEMENT_RUN_LOG is populated at all. No code in any of the five repos reads or writes it, and its RUN_ID is VARCHAR(32) while SETTLEMENT_RUN.RUN_ID is BIGINT."
  - "The full valid value set for SETTLEMENT_RUN.SANGTAE. Only 'RUNNING' appears in code; there is no CHECK constraint, no enum type and no lookup table, so the other states are inferrable only from the column's existence."
  - "The full valid value set for CANCEL_RECON_QUEUE.STATUS. 'PENDING' is written by the relay and 'PROCESSED' by the unscheduled reconciler; the export README notes that no rows with any other status were returned."
  - "EXPECTED_AMT on CANCEL_RECON_QUEUE. The 2026-09-01 export query sums it, but no migration in this repo defines that column — so either a migration outside the repo added it or the export ran against a different shape."
  - "The full definitions of ORDER_MST, ORDER_DTL and ORDER_EVENT_OUTBOX, beyond the columns touched here. They are read and written by settlement-batch but owned by the order team, and only the order-side migrations declare the rest."
  - "How often CANCEL_RECON_QUEUE.SAYU_CD holds the fallback value '00'. OrderEventRelayJob returns '00' whenever the outbox payload does not contain a sayuCd fragment, and '00' is outside the 01-04 vocabulary, but no export counts those rows."
  - "Whether SETTLEMENT_RUN.TOTAL_AMT is ever populated. No code in any of the five repos writes it, so the run header's own money column has no writer."
tags:
  - settlement
  - mysql
  - flyway
  - schema
aliases:
  - "정산 스키마"
  - "settlement-batch schema"
relates_to:
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet — the 01-04 SAYU_CD vocabulary; 수수료 sheet — the rate tiers PARTNER_CONTRACT should hold"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "§3 reports SETTLEMENT_ADJUSTMENT cannot be queried"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Records this service's db as MySQL (settlement), which the datasource contradicts; and CANCEL_RECON_QUEUE consumer as TODO"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "The duplicate run that the composite PK does not prevent across RUN_IDs"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The 2023-04 change that created CANCEL_RECON_QUEUE and the oldest backlog rows"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
    notes: "The 2026-09-01 extraction query, which sums an EXPECTED_AMT column no migration defines"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "41 months of PENDING rows; 합계 4,127 / 188,851,520"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:schemas/settlement/batch/schema.md"
    notes: "Tier-4 extraction of the settlement ERD from the migrations and the embedded JDBC."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "Third reader of SETTLEMENT_RUN and SETTLEMENT_DTL, joining on RUN_ID."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "CANCELLED_STATES = {CHWISO, BANPUM} — the same population CANCEL_RECON_QUEUE collects."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "settlementTargetReader, the cross-boundary read of ORDER_MST and ORDER_DTL."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/PartnerContract.java"
    notes: "Javadoc admitting the rate table has not been maintained since 2021."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
    notes: "The 0.12 HALF_UP calculation that produces JUNGSAN_AMT and SUSURYO."
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
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java"
    notes: "The repository with no caller in any of the five repos."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
    notes: "The JDBC URL naming sellflow_order and spring.batch.initialize-schema."
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
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V6__settlement_run_log_index.sql"
notes: ""
---

# Settlement Batch Data Model

## Overview

Six Flyway migrations define the settlement side of the shared `sellflow_order` MySQL instance. Column naming is romanised Korean throughout — `JUNGSAN_ILJA` for settlement date, `SUSURYO` for commission, `SAYU_CD` for reason code — and [[GLOSSARY-SETTLEMENT]] carries the translations.

Three things about this schema do not line up, and all three are visible without running anything: a table the code writes to but no migration creates, an index on a column name that does not exist, and a pair of columns added in V5 that duplicate an existing column and that nothing writes. A fourth, quieter one is a whole duplicate run-header table, `SETTLEMENT_RUN_LOG`, that no code touches at all.

## Key Facts

- Six Flyway migrations exist, V1 through V6, under `src/main/resources/db/migration/` → src/main/resources/db/migration/
- `SETTLEMENT_RUN` is the run header: `RUN_ID` auto-increment PK, `JUNGSAN_ILJA`, `SANGTAE`, start/end timestamps and `TOTAL_AMT` → src/main/resources/db/migration/V1__settlement_schema.sql
- `SETTLEMENT_DTL` carries one row per settled order with composite PK `(RUN_ID, ORD_NO)` and secondary indexes on `ORD_NO` and `PARTNER_ID` → src/main/resources/db/migration/V1__settlement_schema.sql
- `CANCEL_RECON_QUEUE` was created for SF-2287 as a manual-correction queue: "SF-2287 대응. 정산 후 취소 건을 수기 정정용으로 적재한다. 정산팀이 주기적으로 확인하여 차월 정산에서 차감한다." — "added for SF-2287; post-settlement cancellations are queued for manual correction, the settlement team checks periodically and deducts in the next month's settlement" → src/main/resources/db/migration/V1__settlement_schema.sql
- `SETTLEMENT_ADJUSTMENT` is INSERTed into by `CancelReconciler` with columns `(ORD_NO, SAYU_CD, ADJ_TYPE)`, yet no migration anywhere creates it — verified 2026-09-19 by grepping all five repos for `SETTLEMENT_ADJUSTMENT`, which returns exactly one hit, the INSERT itself → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, src/main/resources/db/migration/
- The settlement team independently reported the same gap: "SETTLEMENT_ADJUSTMENT 테이블이 문서에는 나오는데 실제로 조회가 안 됨. WIP" — "the SETTLEMENT_ADJUSTMENT table appears in the documentation but cannot actually be queried" → sources/context/handover/2025-03_정산팀_인수인계.md (§3)
- V4 creates `IDX_CANCEL_RECON_STATUS ON CANCEL_RECON_QUEUE (STATUS, REG_DT)` while V1 defines the timestamp column as `RECV_DTM`; there is no `REG_DT` column on that table, and MySQL rejects an index on a nonexistent column, so V4 cannot succeed against the V1 shape → src/main/resources/db/migration/V4__cancel_recon_queue_index.sql, src/main/resources/db/migration/V1__settlement_schema.sql
- V1 already created `KEY IX_CANCEL_RECON_QUEUE_01 (STATUS, RECV_DTM)`, so even under the correct column name V4 would duplicate an index that already exists — the migration is both wrong and unnecessary → src/main/resources/db/migration/V1__settlement_schema.sql
- `REG_DT` is not an invented name: it echoes the order side's `REG_DTM`, the column `OrderEventRelayJob` orders the outbox by, which is the likeliest source of the slip → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- V5 adds `PROCESSED_AT` and `PROCESSED_BY`, duplicating the existing `PROCESSED_DTM`, and its own comment admits nothing uses them: "정정 처리 결과를 남기기 위한 컬럼. 아직 쓰는 코드는 없다." — "columns for recording correction results. There is no code using them yet" → src/main/resources/db/migration/V5__add_recon_processed_columns.sql
- The only code that writes a processed timestamp writes `PROCESSED_DTM`, not `PROCESSED_AT` — so `CANCEL_RECON_QUEUE` now has two "processed at" columns, and the newer one has no writer while the older one has a writer that never runs → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, src/main/resources/db/migration/V5__add_recon_processed_columns.sql
- `PROCESSED_BY` is the only place in the schema that anticipates an *actor* for a correction, and it is `VARCHAR(30)` with no writer — the schema records that someone expected a named human or job to own each drained row, and nobody ever did → src/main/resources/db/migration/V5__add_recon_processed_columns.sql
- `PARTNER_CONTRACT` (V3) holds `FEE_RATE DECIMAL(5,4)` per partner, but no settlement code reads it — `PartnerContractRepository` has no caller anywhere in the five repos (verified by grep, re-run 2026-09-19) → src/main/resources/db/migration/V3__add_partner_contract.sql, src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java
- `SETTLEMENT_RUN_LOG` (V2, indexed in V6) duplicates `SETTLEMENT_RUN` column for column in intent — a run id, a start time, an end time and a status — while agreeing with it on none of the names or types: `RUN_ID VARCHAR(32)` against `BIGINT`, `STARTED_AT/ENDED_AT` against `START_DTM/END_DTM`, `STATUS` against `SANGTAE`, and it adds `ROW_CNT` which `SETTLEMENT_RUN` lacks while omitting `TOTAL_AMT` and `JUNGSAN_ILJA` which it has → src/main/resources/db/migration/V2__add_settlement_run_log.sql, src/main/resources/db/migration/V1__settlement_schema.sql
- The naming convention itself flips at V2: V1 is romanised-Korean and `_DTM`-suffixed (`RECV_DTM`, `START_DTM`, `SANGTAE`), V2 onward is English and `_AT`/`_DT`-suffixed (`STARTED_AT`, `STATUS`, `UPD_DT`, `PROCESSED_AT`). The two halves of this schema were written to different conventions → src/main/resources/db/migration/V1__settlement_schema.sql, src/main/resources/db/migration/V2__add_settlement_run_log.sql, src/main/resources/db/migration/V5__add_recon_processed_columns.sql
- No code in any of the five repos reads or writes `SETTLEMENT_RUN_LOG` (verified by grep, 2026-09-19 — the only two hits are its own CREATE TABLE and its CREATE INDEX) → src/main/resources/db/migration/V2__add_settlement_run_log.sql, src/main/resources/db/migration/V6__settlement_run_log_index.sql
- The 2026-09-01 data export sums `EXPECTED_AMT` from `CANCEL_RECON_QUEUE`, a column no migration in this repo defines → sources/raw/exports/README.md
- The shared instance is visible in the datasource itself, not only in the V1 comment: `jdbc:mysql://${DB_HOST:localhost}:3306/sellflow_order` — the same schema order-service uses, while the service registry records this service's database as `MySQL (settlement)` → src/main/resources/application.yml, sources/context/registry/services.yaml
- V1 states the arrangement in its header: "주의: sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)" — "note: this uses the same instance as sellflow_order (integration decision of 2019)" → src/main/resources/db/migration/V1__settlement_schema.sql
- There is not a single `FOREIGN KEY` clause in any of the six migrations; every relationship drawn below is maintained by application code alone → src/main/resources/db/migration/
- Spring Batch's own metadata tables are not in these migrations at all; `spring.batch.initialize-schema: always` creates them, so the schema holds tables no Flyway file describes → src/main/resources/application.yml
- `CANCEL_RECON_QUEUE.SAYU_CD` is not read from a column but cut out of the outbox JSON by offset — `int i = s.indexOf("\"sayuCd\":\"")` then `s.substring(i + 10, i + 12)` — and returns the literal `"00"` when the fragment is absent, a value outside the 01–04 vocabulary → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- `MarkSettledTasklet` resolves the run with a bare `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN`, without the `SANGTAE='RUNNING'` filter that `SettlementItemWriter` applies, so the two steps can disagree about which run is current → src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- The code says the same thing the workbook says about rates: "파트너 계약. 수수료율은 PARTNER_CONTRACT 에 있으나 2021 이후 정비되지 않았다." — "partner contract; the commission rate is in PARTNER_CONTRACT but has not been maintained since 2021" → src/main/java/kr/co/sellflow/settlement/domain/PartnerContract.java
- Three order-team tables are accessed cross-boundary: `ORDER_MST` and `ORDER_DTL` are read by the settlement reader, `ORDER_MST` is updated by the tasklet, and `ORDER_EVENT_OUTBOX` is polled and updated by the relay → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java, src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java, src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java

## Entities

```mermaid
erDiagram
    SETTLEMENT_RUN ||--o{ SETTLEMENT_DTL : "RUN_ID, app-enforced (no FK)"
    ORDER_MST ||--o{ SETTLEMENT_DTL : "ORD_NO, cross-service (no FK)"
    ORDER_MST ||--o{ ORDER_EVENT_OUTBOX : "ORD_NO, emits order.cancelled"
    ORDER_EVENT_OUTBOX ||--o{ CANCEL_RECON_QUEUE : "ORD_NO, via OrderEventRelayJob"
    CANCEL_RECON_QUEUE ||--o{ SETTLEMENT_ADJUSTMENT : "ORD_NO, would produce (NO DDL ANYWHERE)"
    PARTNER_CONTRACT ||--o{ SETTLEMENT_DTL : "PARTNER_ID, declared but never read"
    SETTLEMENT_RUN ||..|| SETTLEMENT_RUN_LOG : "duplicate run header, no code, no shared key type"

    SETTLEMENT_RUN {
        BIGINT RUN_ID PK "AUTO_INCREMENT, no INSERT anywhere"
        DATE JUNGSAN_ILJA
        VARCHAR SANGTAE "only RUNNING seen in code"
        DATETIME START_DTM
        DATETIME END_DTM
        DECIMAL TOTAL_AMT "no writer"
    }
    SETTLEMENT_DTL {
        BIGINT RUN_ID PK_FK "PK part 1"
        VARCHAR ORD_NO PK "PK part 2, IX_01"
        VARCHAR PARTNER_ID "IX_02"
        DECIMAL JUNGSAN_AMT "gross minus SUSURYO"
        DECIMAL SUSURYO "gross times 0.12 HALF_UP"
    }
    CANCEL_RECON_QUEUE {
        BIGINT SEQ PK
        VARCHAR ORD_NO
        VARCHAR SAYU_CD "01-04, plus 00 fallback"
        DATETIME RECV_DTM "V1, the real timestamp"
        VARCHAR STATUS "PENDING default, PROCESSED never written"
        DATETIME PROCESSED_DTM "V1, writer never runs"
        DATETIME PROCESSED_AT "V5, duplicate, no writer"
        VARCHAR PROCESSED_BY "V5, no writer"
    }
    SETTLEMENT_ADJUSTMENT {
        VARCHAR ORD_NO "inferred from INSERT only"
        VARCHAR SAYU_CD "inferred from INSERT only"
        VARCHAR ADJ_TYPE "always CANCEL_CLAWBACK"
    }
    PARTNER_CONTRACT {
        VARCHAR PARTNER_ID PK
        DECIMAL FEE_RATE "V3, ignored, 0.12 hardcoded"
        VARCHAR SETTLE_CYCLE "default DAILY"
        DATETIME UPD_DT
    }
    SETTLEMENT_RUN_LOG {
        VARCHAR RUN_ID PK "VARCHAR32, not BIGINT"
        DATETIME STARTED_AT "V6 index"
        DATETIME ENDED_AT
        VARCHAR STATUS
        INT ROW_CNT
    }
    ORDER_MST {
        VARCHAR ORD_NO PK "owned by 주문팀"
        VARCHAR SANGTAE_CD "set to JUNGSAN_WANRYO by the tasklet"
        DATETIME UPD_DTM "the reader filters DATE(UPD_DTM)"
    }
    ORDER_EVENT_OUTBOX {
        BIGINT EVENT_ID PK "owned by 주문팀"
        VARCHAR ORD_NO
        VARCHAR EVENT_TYPE "only order.cancelled is relayed"
        VARCHAR PAYLOAD "JSON, sayuCd cut by substring"
        CHAR PUBLISHED_YN "set Y by the relay"
        DATETIME REG_DTM
    }
```

Two annotations on that diagram carry the artifact's main findings:

- **`SETTLEMENT_ADJUSTMENT` has no DDL anywhere.** It is drawn because `CancelReconciler` writes to it, not because it is defined. A grep for the table name across all five repos on 2026-09-19 returns one hit — the INSERT — and none of the six migrations, nor order-service's, inventory-api's or settlement-anomaly's DDL, creates it. Its three columns are inferred from that INSERT's column list; whether it has a primary key, an amount column, a status, or even exists in production is unknown → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- **`SETTLEMENT_RUN` and `SETTLEMENT_RUN_LOG` are drawn with a non-identifying, dotted link** because there is no key relationship between them at all. They are two designs for the same concept — one used by three readers and written by nobody, the other written and read by nobody → src/main/resources/db/migration/V1__settlement_schema.sql, src/main/resources/db/migration/V2__add_settlement_run_log.sql

### SETTLEMENT_RUN

One row per settlement execution. `SANGTAE` is the run state; the only value observed in code is `'RUNNING'`, used by the writer to find the current run.

| Column | Type | Notes |
|---|---|---|
| RUN_ID | BIGINT AUTO_INCREMENT PK | Referenced by `SETTLEMENT_DTL.RUN_ID`; no FK |
| JUNGSAN_ILJA | DATE NOT NULL | Settlement date; the anomaly service filters on it |
| SANGTAE | VARCHAR(20) NOT NULL | State; `'RUNNING'` is the only value in code |
| START_DTM | DATETIME | Nullable; never written by any code in the five repos |
| END_DTM | DATETIME | Nullable; never written by any code in the five repos |
| TOTAL_AMT | DECIMAL(18,0) | Never written by any code in the five repos |

No `INSERT INTO SETTLEMENT_RUN` exists anywhere in the five repos (verified by grep, 2026-09-19), yet `SettlementItemWriter.currentRunId()` depends on a `'RUNNING'` row being present → src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java. If no such row exists, `queryForObject` returns `null` for `MAX(RUN_ID)` and every `SETTLEMENT_DTL` insert carries a null `RUN_ID` into a `NOT NULL` PK column — so a missing run header does not degrade the batch, it fails it.

Three readers exist and they do not agree on how to pick the run:

| Reader | Query | Consequence |
|---|---|---|
| `SettlementItemWriter` | `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'` | Writes detail rows under the newest running run |
| `MarkSettledTasklet` | `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN` | Marks orders from the newest run *of any state* |
| settlement-anomaly `/detect` | `JOIN SETTLEMENT_RUN r ON r.RUN_ID = d.RUN_ID WHERE r.JUNGSAN_ILJA = %(ilja)s` | Sees every run for that date, including duplicates |

→ src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java, settlement-anomaly:app/main.py

### SETTLEMENT_DTL

| Column | Type | Notes |
|---|---|---|
| RUN_ID | BIGINT NOT NULL | PK part 1 |
| ORD_NO | VARCHAR(20) NOT NULL | PK part 2, indexed separately as `IX_SETTLEMENT_DTL_01` |
| PARTNER_ID | VARCHAR(20) NOT NULL | Indexed as `IX_SETTLEMENT_DTL_02` |
| JUNGSAN_AMT | DECIMAL(15,0) NOT NULL | Gross minus commission |
| SUSURYO | DECIMAL(15,0) NOT NULL | Commission, `gross × 0.12` HALF_UP |

Both money columns are `DECIMAL(15,0)` — scale zero, i.e. whole KRW, which is why the rounding mode in the processor is a data decision and not a presentation one → src/main/resources/db/migration/V1__settlement_schema.sql, src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java

This is the system of record for "has this order been settled": order-service's own V14 migration says so — "실제 정산 여부는 SETTLEMENT_DTL 이 정본이다" ("SETTLEMENT_DTL is the authoritative record of whether settlement actually happened") → sources/schemas/settlement/batch/schema.md

The composite PK means the same order cannot be written twice within one run, but says nothing across runs — which is how the 2025-07-12 duplicate execution produced two payment requests for the same partners → sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

`OrderEventRelayJob` uses `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` as its "has this order already been settled" test — the `IX_SETTLEMENT_DTL_01 (ORD_NO)` index exists precisely for that lookup → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, src/main/resources/db/migration/V1__settlement_schema.sql

### CANCEL_RECON_QUEUE

| Column | Type | Origin | Notes |
|---|---|---|---|
| SEQ | BIGINT AUTO_INCREMENT PK | V1 | |
| ORD_NO | VARCHAR(20) NOT NULL | V1 | No FK to `ORDER_MST` or `SETTLEMENT_DTL` |
| SAYU_CD | VARCHAR(2) NOT NULL | V1 | Cancellation reason code; values 01–04 in the business-rules workbook, plus the relay's `"00"` fallback when the payload carries no `sayuCd` |
| RECV_DTM | DATETIME DEFAULT CURRENT_TIMESTAMP | V1 | The column V4's index should have referenced |
| STATUS | VARCHAR(20) DEFAULT 'PENDING' | V1 | `'PENDING'` written by the relay, `'PROCESSED'` by the unscheduled reconciler |
| PROCESSED_DTM | DATETIME | V1 | Written only by `CancelReconciler`, which never runs |
| PROCESSED_AT | DATETIME | V5 | Duplicate of the above; no writer anywhere |
| PROCESSED_BY | VARCHAR(30) | V5 | No writer anywhere |

Indexes: `IX_CANCEL_RECON_QUEUE_01 (STATUS, RECV_DTM)` from V1, plus V4's `IDX_CANCEL_RECON_STATUS (STATUS, REG_DT)` against a column that does not exist → src/main/resources/db/migration/V1__settlement_schema.sql, src/main/resources/db/migration/V4__cancel_recon_queue_index.sql

Note what the table does **not** have: no amount column in any migration, no partner id, no link to the `RUN_ID` the order was settled under, and no unique constraint on `ORD_NO`. A row records that an order was cancelled after settlement and why — not how much money is at stake, nor which payout to claw it back from. The 2026-09-01 export nevertheless sums an `EXPECTED_AMT`, which is the sharpest evidence that the production shape and the repo's migrations have diverged → src/main/resources/db/migration/V1__settlement_schema.sql, sources/raw/exports/README.md

### SETTLEMENT_ADJUSTMENT

Not created by any migration, in this repo or any of the other four. Its existence and shape are inferred entirely from one INSERT statement:

```sql
INSERT INTO SETTLEMENT_ADJUSTMENT (ORD_NO, SAYU_CD, ADJ_TYPE) VALUES (?, ?, 'CANCEL_CLAWBACK')
```
→ src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java

Because the only writer is never invoked (see [[PROC-SETTLEMENT-CORRECTION]]), the missing migration has never been exercised as a failure — the `INSERT` has never run, so the absent table has never thrown. That is the ordering to keep straight: registering the missing Quartz trigger would not merely start the drain, it would immediately hit a table that may not exist.

The handover reached the same dead end from the other direction, reporting that the table appears in documentation but cannot be queried → sources/context/handover/2025-03_정산팀_인수인계.md (§3)

### PARTNER_CONTRACT and SETTLEMENT_RUN_LOG

| Column | Type | Notes |
|---|---|---|
| PARTNER_ID | VARCHAR(20) PK | V3 |
| FEE_RATE | DECIMAL(5,4) NOT NULL | The rate the processor never reads |
| SETTLE_CYCLE | VARCHAR(10) NOT NULL DEFAULT 'DAILY' | Read by nothing; the batch is daily by Quartz cron, not by this column |
| UPD_DT | DATETIME | V3's only `_DT` column |

`DECIMAL(5,4)` holds rates up to 9.9999, which comfortably covers the workbook's three tiers — 기본 12.0% (base, for partners with no contract), 프리미엄 9.5% (premium, monthly volume over 100M KRW), 신규 프로모션 6.0% (new-joiner promotion, within 3 months of onboarding) → src/main/resources/db/migration/V3__add_partner_contract.sql, sources/context/business-rules.md (수수료 sheet). The workbook then states the operative reality: "계약별 수수료율은 PARTNER_CONTRACT 에 있으나 2021년 이후 미정비. 현재 전 건 기본 수수료율 적용 중." — "per-contract commission rates are in PARTNER_CONTRACT but have not been maintained since 2021; the base rate is currently applied to everything". The schema can express three tiers; the code applies one.

| Column | Type | Notes |
|---|---|---|
| RUN_ID | VARCHAR(32) PK | V2 — not the `BIGINT` of `SETTLEMENT_RUN` |
| STARTED_AT | DATETIME NOT NULL | Indexed by V6 as `IDX_SETTLEMENT_RUN_LOG_DT` |
| ENDED_AT | DATETIME NULL | |
| STATUS | VARCHAR(20) NOT NULL | Not `SANGTAE` |
| ROW_CNT | INT NOT NULL DEFAULT 0 | Has no counterpart on `SETTLEMENT_RUN` |

`SETTLEMENT_RUN_LOG` is an orphan: created in V2, indexed in V6, typed inconsistently with `SETTLEMENT_RUN`, and referenced by no code. The pairing is the most complete duplication in the schema — two run-header tables where one has three readers and no writer, and the other has neither. Anyone adding run bookkeeping must first decide which of the two is the target, because the migrations answer for both and the code answers for neither → src/main/resources/db/migration/V2__add_settlement_run_log.sql, src/main/resources/db/migration/V6__settlement_run_log_index.sql

## Worked Examples

### Enum vocabularies

None of these is a database enum. There is no `ENUM` type, no `CHECK` constraint and no lookup table anywhere in the six migrations — every set below is the set of literals that appear in code or in a company document, which is why each row carries its own provenance → src/main/resources/db/migration/

**`SETTLEMENT_RUN.SANGTAE` — `VARCHAR(20) NOT NULL`**

| Value | Meaning | Written by | Read by | Evidence |
|---|---|---|---|---|
| `RUNNING` | run in progress | **nothing in the five repos** | `SettlementItemWriter.currentRunId()` | src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java |
| *(any other)* | unknown | — | `MarkSettledTasklet` reads across all states with a bare `MAX(RUN_ID)` | src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java |

A completed or failed state must exist for the column to be meaningful — `END_DTM` implies it — but no literal for one appears anywhere. Recorded in `known_unknowns`.

**`CANCEL_RECON_QUEUE.STATUS` — `VARCHAR(20) DEFAULT 'PENDING'`**

| Value | Meaning | Written by | Observed in production? | Evidence |
|---|---|---|---|---|
| `PENDING` | queued, awaiting correction | `OrderEventRelayJob` INSERT, and the column default | Yes — all 4,127 rows in the 2026-09-01 export | src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| `PROCESSED` | drained into an adjustment | `CancelReconciler` UPDATE — which is registered on no schedule | **No** — the export README states "STATUS 가 PENDING 외의 값을 가진 행은 조회되지 않았다." ("no rows with a STATUS other than PENDING were returned") | src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sources/raw/exports/README.md |

This is a two-value vocabulary of which one value has never been written in 41 months of data. The export README adds the caveat that keeps it honest: "수기 정정분이 시스템 밖에서 처리되었다면 이 수치에 반영되지 않는다." — "if manual corrections were handled outside the system, they are not reflected in these figures" → sources/raw/exports/README.md

**`SETTLEMENT_ADJUSTMENT.ADJ_TYPE` — type unknown, no DDL**

| Value | Meaning | Written by | Evidence |
|---|---|---|---|
| `CANCEL_CLAWBACK` | next-month clawback for a post-settlement cancellation | `CancelReconciler` INSERT, hardcoded as a SQL literal | src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java |

One literal, hardcoded, in a statement that has never executed, into a table with no DDL. The column's width and type are unknown because nothing declares them. That a *type* column exists at all implies other adjustment kinds were envisaged — a manual adjustment, a fee correction — but none appears in code.

**`CANCEL_RECON_QUEUE.SAYU_CD` — `VARCHAR(2) NOT NULL`**

| Value | Korean | English | Cost borne by | 재고 복원 (restock) | Evidence |
|---|---|---|---|---|---|
| `01` | 파트너 귀책 (재고부족·출고지연) | partner fault — out of stock, shipping delay | 파트너 (partner) | O | sources/context/business-rules.md (취소정책 sheet) |
| `02` | 시스템 오류 | system error | 셀플로우 (Sellflow) | O | sources/context/business-rules.md |
| `03` | 고객 변심 | customer change of mind | 셀플로우 | X — "배송 시작 후 취소 시 재고 복원 불가" (no restock once shipping has begun) | sources/context/business-rules.md |
| `04` | 배송 실패 (주소불명·수취거부) | delivery failure — bad address, refused | 파트너 | O | sources/context/business-rules.md |
| `00` | — | **not a documented code** | — | — | `OrderEventRelayJob` fallback when the payload has no `sayuCd` fragment → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |

All four documented codes are marked `정산 차감: O` — every cancellation reason requires a settlement deduction. The workbook also fixes the rule the whole schema turns on: "정산 실행 전 취소 → 해당 주문을 정산 대상에서 제외 / 정산 실행 후 취소 → 정산팀이 차월 정산에서 수기 차감" ("cancelled before the settlement run: exclude the order from settlement; cancelled after the run: the settlement team deducts it manually in the following month's settlement") → sources/context/business-rules.md (취소정책 sheet)

### Join query 1 — reconstruct one order's settlement-then-cancellation

The path from a settled order to its stranded correction row spans three tables and two owners. There is no view for it and no FK to guide it; this is the join a human runs by hand:

```sql
SELECT  q.SEQ,
        q.ORD_NO,
        q.SAYU_CD,
        q.RECV_DTM        AS cancelled_relayed_at,
        q.STATUS,
        q.PROCESSED_DTM,                    -- V1 column, written by the reconciler
        q.PROCESSED_AT,                     -- V5 column, always NULL
        d.RUN_ID,
        d.PARTNER_ID,
        d.JUNGSAN_AMT     AS paid_out,
        d.SUSURYO         AS fee_taken,
        r.JUNGSAN_ILJA    AS settled_for_date,
        r.SANGTAE         AS run_state,
        m.SANGTAE_CD      AS order_state_now
  FROM  CANCEL_RECON_QUEUE q
  JOIN  SETTLEMENT_DTL     d ON d.ORD_NO = q.ORD_NO
  JOIN  SETTLEMENT_RUN     r ON r.RUN_ID = d.RUN_ID
  JOIN  ORDER_MST          m ON m.ORD_NO = q.ORD_NO
 WHERE  q.ORD_NO = 'ORD20260817001';
```

Join keys, all application-enforced with no FK: `CANCEL_RECON_QUEUE.ORD_NO = SETTLEMENT_DTL.ORD_NO`, `SETTLEMENT_DTL.RUN_ID = SETTLEMENT_RUN.RUN_ID`, `ORDER_MST.ORD_NO = SETTLEMENT_DTL.ORD_NO` → src/main/resources/db/migration/V1__settlement_schema.sql

Two cautions the schema forces on this query. The `SETTLEMENT_DTL` join can return **more than one row per queue entry**, because the PK is `(RUN_ID, ORD_NO)` and an order settled in two runs has two detail rows — exactly the 2025-07-12 case → sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md. And `d.JUNGSAN_AMT` is the amount paid out, not the amount to claw back; no column anywhere holds the clawback figure, which is why the export had to invent `EXPECTED_AMT`.

The lifecycle that query reconstructs, step by step:

1. `dailySettlementJob` writes `SETTLEMENT_DTL(RUN_ID=8801, ORD_NO='ORD20260817001', PARTNER_ID='P00132', JUNGSAN_AMT=40040, SUSURYO=5460)` for a 45,500 KRW gross at the hardcoded 12 percent rate → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
2. `markSettledStep` sets `ORDER_MST.SANGTAE_CD='JUNGSAN_WANRYO'` for that order → src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java
3. The customer cancels. `order-service` writes an `order.cancelled` row to `ORDER_EVENT_OUTBOX` → sources/context/tickets/SF-2287.md
4. Within 10 minutes `OrderEventRelayJob` finds a `SETTLEMENT_DTL` row for that order and inserts `CANCEL_RECON_QUEUE(ORD_NO='ORD20260817001', SAYU_CD='03', STATUS='PENDING')` → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
5. The row stops there. Step 5 would be `CancelReconciler` writing `SETTLEMENT_ADJUSTMENT` and flipping `STATUS='PROCESSED'`, but it has no trigger → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java

The schema-level consequence: `CANCEL_RECON_QUEUE` is append-only in practice.

### Join query 2 — age the backlog

This is, verbatim in intent, the query the 2026-09-01 extraction ran → sources/raw/exports/README.md:

```sql
SELECT DATE_FORMAT(RECV_DTM,'%Y-%m') AS ym, COUNT(*), SUM(EXPECTED_AMT)
  FROM CANCEL_RECON_QUEUE
 WHERE STATUS='PENDING'
 GROUP BY 1
 ORDER BY 1;
```

It touches one table and no join, which is the point: nothing in this schema links a queued correction to the money it concerns, so an aging report can only count rows and sum a column the migrations do not define. The result was 41 months, 합계 (total) 4,127 rows and 188,851,520 KRW, the oldest bucket dating to 2023-04 — the month SF-2287 shipped → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv, sources/context/tickets/SF-2287.md

The join-based alternative, which the schema does support and which would not need `EXPECTED_AMT`, is:

```sql
SELECT DATE_FORMAT(q.RECV_DTM,'%Y-%m') AS ym,
       COUNT(DISTINCT q.SEQ)            AS queued_rows,
       SUM(d.JUNGSAN_AMT)               AS amount_paid_out_on_cancelled_orders
  FROM CANCEL_RECON_QUEUE q
  JOIN SETTLEMENT_DTL     d ON d.ORD_NO = q.ORD_NO
 WHERE q.STATUS = 'PENDING'
 GROUP BY 1
 ORDER BY 1;
```

Read the second column carefully: it sums what was *paid*, and double-counts any order settled in two runs. It is the closest the declared schema gets to the backlog's value, and it is not the same number. See [[RISK-SETTLEMENT-RECON-BACKLOG]].

### Join query 3 — find settled-but-cancelled orders the queue missed

The relay only enqueues orders whose cancellation event reached the outbox *after* they were settled. An order cancelled before the relay's first run in 2023-04, or whose event was marked `PUBLISHED_YN='Y'` without an insert, leaves no queue row. The cross-check runs directly against the two settlement tables and `ORDER_MST`:

```sql
SELECT d.RUN_ID, d.ORD_NO, d.PARTNER_ID, d.JUNGSAN_AMT, m.SANGTAE_CD, r.JUNGSAN_ILJA
  FROM SETTLEMENT_DTL d
  JOIN SETTLEMENT_RUN r ON r.RUN_ID = d.RUN_ID
  JOIN ORDER_MST      m ON m.ORD_NO = d.ORD_NO
  LEFT JOIN CANCEL_RECON_QUEUE q ON q.ORD_NO = d.ORD_NO
 WHERE m.SANGTAE_CD IN ('CHWISO','BANPUM')
   AND q.SEQ IS NULL;
```

The first three lines of that query are byte-for-byte the join `settlement-anomaly` already runs for its `CANCELLED_SETTLED` rule, and `{'CHWISO','BANPUM'}` is exactly its `CANCELLED_STATES` constant — so this schema already contains a second, independent detector of the same population, in a different service, writing to a different table → settlement-anomaly:app/main.py, settlement-anomaly:model/detector.py, see [[SCH-SETTLEMENT-ANOMALY]]

### Join query 4 — reconcile the two run-header tables

Included because it is the query anyone will reach for on first encountering both tables, and it cannot be written honestly:

```sql
-- Does NOT work as intended: SETTLEMENT_RUN.RUN_ID is BIGINT,
-- SETTLEMENT_RUN_LOG.RUN_ID is VARCHAR(32), and the log table is empty.
SELECT r.RUN_ID, r.JUNGSAN_ILJA, r.SANGTAE, l.STATUS, l.ROW_CNT
  FROM SETTLEMENT_RUN r
  LEFT JOIN SETTLEMENT_RUN_LOG l ON l.RUN_ID = CAST(r.RUN_ID AS CHAR);
```

MySQL will run it after the implicit cast, and every `l.` column will be NULL, because nothing has ever written a `SETTLEMENT_RUN_LOG` row → src/main/resources/db/migration/V2__add_settlement_run_log.sql. Recording the query here, with its result, is cheaper than letting the next reader rediscover it.

## Related

- [[SYS-SETTLEMENT]] — the service that owns these tables
- [[PROC-SETTLEMENT-DAILY-BATCH]] — how `SETTLEMENT_RUN` and `SETTLEMENT_DTL` get written
- [[PROC-SETTLEMENT-CORRECTION]] — why `CANCEL_RECON_QUEUE` only grows
- [[GLOSSARY-SETTLEMENT]] — the romanised Korean column vocabulary
- [[SCH-SETTLEMENT-ANOMALY]] — the anomaly table that reads `SETTLEMENT_DTL`
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the 4,127 PENDING rows these tables hold
