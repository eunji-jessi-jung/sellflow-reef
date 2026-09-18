---
id: "SYS-SETTLEMENT"
type: "system"
title: "Partner Settlement Batch Service"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-read line-by-line against settlement-batch 1.6.2 on 2026-09-19 — every Java class, all six Flyway migrations, application.yml, .env.dev and the three CD workflows — alongside the settlement team's own documents. Goes stale if QuartzConfig gains or loses a trigger, if a V7 migration lands, if DEFAULT_FEE_RATE in SettlementItemProcessor changes, or if the service registry is re-reviewed (it still carries last_reviewed 2026-03-02 with a note that it has not been updated since)."
freshness_triggers:
  - ".env.dev"
  - ".github/workflows/settlement-batch-prod-cd.yml"
  - "build.gradle"
  - "src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - "src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "src/main/resources/application.yml"
  - "src/main/resources/db/migration/*.sql"
known_unknowns:
  - "Who or what creates the SETTLEMENT_RUN row with SANGTAE='RUNNING' that SettlementItemWriter.currentRunId() depends on. Re-verified on 2026-09-19 by grepping all five repos for 'SETTLEMENT_RUN' and for 'INSERT INTO SETTLEMENT_RUN': the table is created in V1 and read by SettlementItemWriter and MarkSettledTasklet, and nothing anywhere writes it. Either an out-of-repo script or a human inserts it, or currentRunId() returns null on a virgin database."
  - "Whether SETTLEMENT_ADJUSTMENT exists in the production database at all. No migration in any of the five repos creates it — it appears only in the INSERT inside CancelReconciler — and the 2025 handover records the same doubt: 'SETTLEMENT_ADJUSTMENT 테이블이 문서에는 나오는데 실제로 조회가 안 됨. WIP' ('the SETTLEMENT_ADJUSTMENT table appears in the documents but cannot actually be queried')."
  - "What deploy.sh does and where it lives. All three CD workflows invoke ./deploy.sh $ENVIRONMENT and the script is not in the repository; there is no Dockerfile or deployment manifest either, so the prod deploy path is not fully in version control."
  - "Whether the partner portal actually reads a settlement report file. SettlementReportWriter's Javadoc asserts '파트너 포털이 이 파일을 읽어간다' ('the partner portal reads this file'), but write() only logs a line, no file writer exists in the repo, and no component calls the method."
  - "Whether SettlementBatchApplication runs as one long-lived process or per-invocation. Clustered JDBC Quartz plus a 10-minute repeat-forever relay trigger implies long-lived, but no deployment manifest was available to confirm it."
  - "Which database the deployed service really reaches. application.yml hardcodes the sellflow_order schema, .env.dev names a separate settlement database, and services.yaml says 'MySQL (settlement)'. The .env.dev value is provably dead in code (see Key Facts) but the deployed environment's DB_HOST was not observed."
  - "Why the commission rate is 0.12. No document in sources/context states the figure; SettlementItemProcessor calls it DEFAULT_FEE_RATE and dates the neglect ('2021년 이후 미정비') without saying where 12% came from or who approved it."
  - "Whether settlement-anomaly's POST /detect is ever invoked against this service's output. Its README claims a daily 03:00 run after the batch, but no scheduler exists in that repo and no caller was found in the five repos."
tags:
  - settlement
  - spring-batch
  - quartz
  - mysql
  - java
  - db-as-queue
aliases:
  - "settlement-batch"
  - "정산 배치"
relates_to:
  - type: "refines"
    target: "[[API-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[API-SETTLEMENT-BATCH]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[DEC-SELLFLOW-MONEY-ROUNDING]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[DEC-SETTLEMENT-CANCEL-CLAWBACK]]"
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-RUNTIME]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-AUTH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-FLOW-CATALOG]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-RUN-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
  - type: "integrates_with"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "2025-03 handover — what the team actually does, and the three items handed over unresolved"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "조직도 + 변경이력 sheets — team size and the 2024-07 headcount cut"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
    notes: "2023 draft procedure — the 'fewer than 10 a month' sizing assumption"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "2024 issued procedure — responsibility table, forms, retention"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Service registry entry for settlement-batch; CANCEL_RECON_QUEUE consumer recorded as TODO"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "P2 postmortem — duplicate Quartz trigger, ~42M KRW double payout, open action items"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Extracted queue/schedule inventory for settlement"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "Extracted runtime facts and the shared-database topology table"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "41 monthly rows plus the 합계 total line (4,127 / 188,851,520)"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.env.dev"
    notes: "DB_URL, DB_USER, QUARTZ_ENABLED — two of the three are read by nothing"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.github/workflows/settlement-batch-prod-cd.yml"
    notes: "workflow_dispatch only, builds with -x test, calls an absent ./deploy.sh"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:README.md"
    notes: "Stack, job list, relay description"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:build.gradle"
    notes: "Spring Boot 2.3.12, Java 8, version 1.6.2, no spring-boot-starter-web"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
    notes: "FLOOR rounding and the undocumented 2022 divergence from order-service"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "Two-step job, chunk 500, the reader SQL and its cancellation-blind comment"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "The only two registered triggers"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
    notes: "Hardcoded DEFAULT_FEE_RATE 0.12"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
    notes: "Row-by-row INSERT, MAX(RUN_ID) lookup, irreversibility Javadoc"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
    notes: "The unscheduled clawback component and its 2023-04-24 TODO"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "Outbox polling, settled-only insert, substring payload parsing"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
    notes: "Log-only report writer with an uncalled method"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "Cross-team UPDATE of ORDER_MST"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
    notes: "Datasource, Quartz cluster mode, batch auto-start disabled, Flyway locations"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "SETTLEMENT_RUN, SETTLEMENT_DTL, CANCEL_RECON_QUEUE and the shared-instance header"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql"
    notes: "Index on a column CANCEL_RECON_QUEUE does not have"
notes: "This artifact is the root of the settlement knowledge graph: every settlement-domain artifact hangs off it. Company-document refs are relative to the reef root; code refs are relative to the settlement-batch source root unless another repo is named."
---

# Partner Settlement Batch Service

## Overview

`settlement-batch` is 셀플로우's daily partner payout engine. Once a day it reads the previous day's delivered orders out of the shared order database, deducts a commission, records a settlement detail row per order, and marks the order as settled. It has no HTTP surface at all — everything it does is started by a Quartz trigger and expressed as writes to shared tables, which is why its interface contract is documented in [[API-SETTLEMENT-BATCH]] rather than as an OpenAPI spec.

Two jobs live in the same process: the settlement job itself and an outbox relay that pulls `order.cancelled` events from the order service's outbox table and parks post-settlement cancellations in a correction queue. A third component, `CancelReconciler`, was written to drain that queue but was never registered in `QuartzConfig` and has never run. The queue it feeds now holds 4,127 rows worth 188,851,520 KRW. That is the central knot of this system: the compensating control agreed in 2023 exists as a class and as a written procedure, and as nothing else.

Three further components are present-but-inert — `SettlementReportWriter`, `PartnerContractRepository` and `MoneyUtil` — so a reader of this repository should assume nothing runs merely because it compiles. The rule for this system is: if it is not in `QuartzConfig` or reachable from `DailySettlementJobConfig`, it does not execute.

## Key Facts

- Stack is Java 8 (`sourceCompatibility = '1.8'`) / Spring Boot 2.3.12.RELEASE / Spring Batch 4 / Quartz, at version 1.6.2 → settlement-batch:build.gradle
- `spring-boot-starter-web` is absent from the dependency list, so the service has no HTTP surface and no port → settlement-batch:build.gradle
- Quartz runs in cluster mode (`org.quartz.jobStore.isClustered: true`) over a JDBC job store, so scheduler state lives in the same MySQL instance as the business data → settlement-batch:src/main/resources/application.yml
- `QuartzConfig` registers exactly two triggers and no others: `dailySettlementTrigger` on cron `0 0 2 * * ?` in `Asia/Seoul`, and `orderEventRelayTrigger` on a simple schedule of every 10 minutes repeating forever → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- Spring Batch's own auto-start is disabled (`spring.batch.job.enabled: false`) while `initialize-schema: always` still provisions the batch metadata tables, so Quartz is the sole launcher → settlement-batch:src/main/resources/application.yml
- `dailySettlementJob` is two steps — `settlementStep` (chunk size 500, reader/processor/writer) followed by `markSettledStep` (a tasklet) → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java
- The reader's javadoc says it ignores cancellation — "전일 배송완료된 주문을 대상으로 한다. ... 주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다. 배송이 완료되었다면 파트너는 이행을 마친 것으로 본다." ("it targets orders delivered the previous day ... the order's current status and whether it was cancelled are not part of the condition; if delivery completed, the partner is considered to have fulfilled") — but the SQL's first predicate, `m.SANGTAE_CD = 'BAESONG_WANRYO'`, is a current-status filter, and a cancel overwrites `SANGTAE_CD` and `UPD_DTM` in one write. Cancelled orders are excluded in practice, as a side effect rather than by rule; the javadoc is intent, not behaviour → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java, order-service:src/main/java/kr/co/sellflow/order/domain/OrderMst.java
- The commission rate is a hardcoded `BigDecimal("0.12")` named `DEFAULT_FEE_RATE`, applied to every partner, while the per-partner rate sits unused in `PARTNER_CONTRACT`: "파트너별 계약 수수료율은 PARTNER_CONTRACT 에 있으나, 현재는 기본 수수료율만 적용한다. (2021년 이후 미정비)" — "per-partner contractual rates are in PARTNER_CONTRACT, but only the default rate is applied at present (not maintained since 2021)" → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- `PartnerContractRepository.find()` is the only code that would read `PARTNER_CONTRACT`, and a grep of all five repos finds no caller for it — the V3 migration created a table that nothing queries at runtime → settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java, settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql
- The processor rounds fees with `RoundingMode.HALF_UP` while the shared `MoneyUtil.fee()` rounds with `FLOOR`, and the only unit test in the repository asserts the `FLOOR` behaviour that production never uses (`수수료는_절사한다` — "the fee is truncated") → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java, settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java
- `MoneyUtil` records the divergence and its undocumented status outright: "정산은 절사(FLOOR), 주문은 반올림(HALF_UP). 2022 협의 결과이며 문서화되어 있지 않다." — "settlement truncates, order rounds half-up; this was agreed in 2022 and is not documented" → settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java
- `CancelReconciler` is a `@Component` that nothing calls, carrying its own unclosed registration TODO: "TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤" — "TODO: Quartz schedule registration needed (to be added to QuartzConfig after 박성민 confirms)" → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- The consequence is quantified in the 2026-09-01 production export: 4,127 `PENDING` rows totalling 188,851,520 KRW across 41 months, against a 2023 procedure that sized the workload at "예상 처리량 월 10건 미만" ("expected volume under 10 cases a month") → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv, sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md
- The schema migration header states the shared-instance decision and the cross-team write in the same breath: "주의: sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)" and "ORDER_MST 는 주문팀 소유이나 정산 배치가 SANGTAE_CD 를 갱신한다." — "note: this uses the same instance as sellflow_order (2019 consolidation decision)" / "ORDER_MST belongs to the order team, but the settlement batch updates SANGTAE_CD" → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
- Six Flyway files exist, V1 through V6. How far the chain actually gets is **unresolved**: V4 creates an index on a column that does not exist — `CREATE INDEX IDX_CANCEL_RECON_STATUS ON CANCEL_RECON_QUEUE (STATUS, REG_DT)` where V1 defined the timestamp column as `RECV_DTM`. MySQL rejects an index on a nonexistent column, so on a clean database V4 fails and blocks V5 and V6 behind it; no source shows whether it ever applied in any environment. See [[SCH-SETTLEMENT-BATCH]] and [[RISK-SETTLEMENT]] → settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql, settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
- V5 adds `PROCESSED_AT` and `PROCESSED_BY` beside the existing `PROCESSED_DTM`, and says so: "정정 처리 결과를 남기기 위한 컬럼. 아직 쓰는 코드는 없다." — "columns for recording correction results; no code writes them yet". `CancelReconciler` still updates `PROCESSED_DTM`, so both new columns remain unwritten → settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- `.env.dev` sets `DB_URL=jdbc:mysql://settlement-db-dev.internal:3306/settlement` and `QUARTZ_ENABLED=true`, but `application.yml` interpolates only `${DB_HOST}` and `${DB_USER}` — neither `DB_URL` nor `QUARTZ_ENABLED` is read anywhere in the repo, so the "separate settlement database" they imply is inert configuration → settlement-batch:.env.dev, settlement-batch:src/main/resources/application.yml
- All three CD workflows are `workflow_dispatch` only, build with `./gradlew clean build -x test`, and then invoke a `./deploy.sh` that is not in the repository — production deploys are manual and partly outside version control → settlement-batch:.github/workflows/settlement-batch-prod-cd.yml
- Ownership is 재무본부 정산팀 (Finance Division, Settlement Team), led by 김도윤, at 3 people after the 2024-07-01 cut recorded as "정산팀 인원 조정 4명 → 3명" ("settlement team headcount adjusted 4 → 3") → sellflow-docs:context/org-chart.md
- The registry disagrees with the code about the database (`db: MySQL (settlement)` versus a hardcoded `sellflow_order` schema) and records the queue consumer as unresolved — "consumer: TODO   # 확인 필요" ("needs checking") — under a header saying it has not been revisited since 2026-03-02 → sellflow-docs:context/registry/services.yaml, settlement-batch:src/main/resources/application.yml

## Responsibilities

**Owned and operating**

- Computing the daily payable amount per order (gross minus a 12% commission) and writing one `SETTLEMENT_DTL` row per order — see [[PROC-SETTLEMENT-DAILY-BATCH]] and [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- Flipping settled orders to `SANGTAE_CD='JUNGSAN_WANRYO'` in `ORDER_MST`, a table owned by the order team → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java
- Relaying `order.cancelled` events out of `ORDER_EVENT_OUTBOX` into `CANCEL_RECON_QUEUE`, but only for orders that already have a `SETTLEMENT_DTL` row → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- Owning the Flyway schema described in [[SCH-SETTLEMENT-BATCH]].

**Written but not operating** (each verified by grepping all five repos for callers)

| Component | What it would do | Why it does not |
|---|---|---|
| `CancelReconciler.reconcileCancellations()` | Drain `PENDING` queue rows into `SETTLEMENT_ADJUSTMENT` with `ADJ_TYPE='CANCEL_CLAWBACK'` | Never registered in `QuartzConfig`; no caller. See [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] and [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]. |
| `SettlementReportWriter.write(runId)` | Produce the partner-facing settlement report file | Method body is a single log line; no step or job calls it |
| `PartnerContractRepository.find()` | Load the per-partner `FEE_RATE` | No caller; the processor uses a hardcoded constant instead |
| `MoneyUtil.fee()` | Shared truncating fee arithmetic | Referenced only from the unit test |
| `DateUtil.settlementBaseDate()` | Pick the right business date when run manually before 02:00 | No caller; `DailySettlementQuartzJob` computes `LocalDate.now().minusDays(1)` inline instead |

## Does NOT Own

- **The cancellation decision and the `order.cancelled` event.** Both belong to 주문팀 / [[SYS-ORDER]]; the v1.1 procedure assigns "이벤트 발행" ("event publication") to 주문팀 and only "정정 대상 확인, 차월 차감 반영, 파트너 통지" ("identifying corrections, applying the next-month deduction, notifying partners") to 정산팀 → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md
- **`ORDER_MST` and `ORDER_DTL`.** It reads both and writes one column of one of them, by the 2019 shared-instance arrangement rather than by ownership — see [[CON-ORDER-SETTLEMENT]] and [[DEC-SELLFLOW-SHARED-DB]] → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
- **`ORDER_EVENT_OUTBOX`.** The table is created and written by order-service; settlement-batch only polls it and sets `PUBLISHED_YN='Y'` → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **Anomaly detection over settlement output.** That is [[SYS-SETTLEMENT-ANOMALY]], owned by 데이터팀; the registry notes no remediation owner is defined — "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" ("performs detection only; the party responsible for acting is not defined") → sellflow-docs:context/registry/services.yaml
- **The partner portal and the settlement admin screen.** Neither is in this repository; the handover lists partner-portal access as "TBD" and says queue volume can only be seen by querying the database directly → sellflow-docs:context/handover/2025-03_정산팀_인수인계.md
- **Quarterly recognition of the uncorrected balance.** v1.1 assigns that to 재무기획팀: "분기 결산 시 미정정 잔액 확인" ("check the uncorrected balance at quarterly close") → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md
- **Payment execution.** The batch records a payout amount; the bank transfer happens elsewhere on the next business day, and this service has no cancel path for it → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java

## Core Concepts

**정산일자 (JUNGSAN_ILJA, settlement date)** — the business date a run settles, passed as the `jungsanIlja` job parameter. `DailySettlementQuartzJob` derives it as `LocalDate.now().minusDays(1)` with a `ts` millisecond parameter appended to make each launch unique to Spring Batch → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java

**Run** — one execution, identified by `RUN_ID`. Every detail row carries the run it belongs to, and the writer resolves the current run by `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'`. `MarkSettledTasklet` then resolves it differently, as a bare `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN` with no status filter — the two lookups can diverge. See [[PROC-SETTLEMENT-RUN-LIFECYCLE]] → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

**Irreversibility** — "지급 요청이 전송되면 되돌릴 수 없다. 은행 이체는 익영업일에 실행된다." — "once the payment request is sent it cannot be undone; the bank transfer executes on the next business day". The same rule is the first warning in the team handover and the reason the 2025-07-12 duplicate run had to be netted off rather than reversed → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, sellflow-docs:context/handover/2025-03_정산팀_인수인계.md

**차월 정산 조정 (next-month adjustment)** — the only correction path, precisely because payouts are irreversible. Defined in v1.1 as "지급이 완료된 정산 금액을 차월 정산에서 차감하여 조정하는 것" ("adjusting an already-paid settlement amount by deducting it from the following month's settlement"). Procedure in [[PROC-SETTLEMENT-CORRECTION]], vocabulary in [[GLOSSARY-SETTLEMENT]], the decision itself in [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md

**Shared-instance coupling** — because settlement and order share one MySQL instance, integration is by table rather than by API. The relay reads `ORDER_EVENT_OUTBOX` directly and the tasklet writes `ORDER_MST` directly. There is no HTTP client of any kind in this repository → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

**Romanised Korean naming** — every column and field is romanised Korean (`SANGTAE_CD`, `SUSURYO`, `JUNGSAN_AMT`, `SAYU_CD`), the same convention as order-service. See [[PAT-SELLFLOW-ROMANISED-NAMING]] → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql

## Dependencies

| System | Integration Type | Purpose | Auth Method |
|---|---|---|---|
| MySQL `sellflow_order` (shared instance) | JDBC, direct SQL | Everything: reads `ORDER_MST`/`ORDER_DTL`, writes `SETTLEMENT_DTL`, updates `ORDER_MST.SANGTAE_CD` | Username/password from `${DB_USER:sellflow}` and the datasource URL; no password property is set in `application.yml` at all → settlement-batch:src/main/resources/application.yml |
| [[SYS-ORDER]] | Shared table (`ORDER_EVENT_OUTBOX`), polled every 10 minutes | Receive `order.cancelled`; acknowledge by setting `PUBLISHED_YN='Y'` | None — same database credentials, no service-to-service authentication → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |
| [[SYS-ORDER]] | Shared table (`ORDER_MST`), direct UPDATE | Set `JUNGSAN_WANRYO` on settled orders | None; sanctioned by the 2019 arrangement recorded in V1 → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java |
| Quartz JDBC job store (same MySQL instance) | JDBC, clustered | Trigger persistence and cluster coordination for both jobs | Same datasource credentials → settlement-batch:src/main/resources/application.yml |
| Flyway | JDBC, startup migration | Owns V1–V6 of the settlement tables; whether V4–V6 ever applied is unresolved | Same datasource credentials → settlement-batch:src/main/resources/application.yml |
| [[SYS-SETTLEMENT-ANOMALY]] | Downstream reader of the same tables | Detects anomalies over settlement output; this service neither calls it nor is called by it | Not applicable — no call exists in either direction → sellflow-docs:infra/settlement/queues.md |
| Bank / payment execution | Out of band, not in this repository | Executes the transfer on the next business day | Unknown — no client, credential or endpoint for it exists in the repo → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java |

Authentication posture across the service is covered in [[PROC-SETTLEMENT-AUTH]]; the runtime topology it sits in is [[PROC-SELLFLOW-RUNTIME]].

## Domain Behavior Highlights

**The database is the message bus.** Both "queues" in this system are tables polled by Quartz. The relay runs `SELECT EVENT_ID, ORD_NO, PAYLOAD FROM ORDER_EVENT_OUTBOX WHERE PUBLISHED_YN='N' AND EVENT_TYPE=? ORDER BY REG_DTM LIMIT 500` every 10 minutes, giving a hard ceiling of about 3,000 cancel events per hour. No Kafka, RabbitMQ, SQS or Celery dependency appears in any of the five repos. The payload is not parsed as JSON either — `sayuCd()` does `s.substring(i+10, i+12)` after an `indexOf("\"sayuCd\":\"")` and silently returns `"00"` when the marker is missing, so any change to the payload shape degrades to a wrong reason code rather than an error → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, sellflow-docs:infra/settlement/queues.md

**Acknowledgement means "looked at", not "acted on".** The relay sets `PUBLISHED_YN='Y'` for every row it reads, including rows where the `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO=?` guard was false and nothing was enqueued. Once acknowledged, an event is unrecoverable from the outbox, so a mis-timed relay run (cancellation relayed before that day's settlement wrote its detail row) drops the correction permanently and silently → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java

**One flat fee rate for every partner.** `SettlementItemProcessor` multiplies gross by a hardcoded `0.12` and `setScale(0, RoundingMode.HALF_UP)`. `PARTNER_CONTRACT` (V3) holds `FEE_RATE DECIMAL(5,4)` and `SETTLE_CYCLE`, and `PartnerContractRepository` can read it, but nothing in the running path does. A partner whose contract says anything other than 12% is settled at 12% — and `SETTLE_CYCLE VARCHAR(10) NOT NULL DEFAULT 'DAILY'` is likewise never consulted, so every partner is settled daily regardless → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql

**Settlement is blind to cancellation, by design.** The reader filters on `SANGTAE_CD='BAESONG_WANRYO' AND DATE(m.UPD_DTM) = ?` and the Javadoc states the intent explicitly (see Key Facts). That design is coherent only if the clawback that compensates for it actually runs. It does not — which is why the 2025-07-12 postmortem's open item "취소 건이 정산 대상에서 제외되는지 점검" ("check whether cancelled orders are excluded from settlement"), raised 2025-07-15, still reads "이후 논의 없음" ("no discussion since") → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java, sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

**The clawback exists on paper and in a class, and nowhere else.** `CancelReconciler` would insert `SETTLEMENT_ADJUSTMENT` rows with `ADJ_TYPE='CANCEL_CLAWBACK'` and flip queue rows to `PROCESSED`. It is unscheduled; `SETTLEMENT_ADJUSTMENT` has no `CREATE TABLE` in any migration in any of the five repos; and the 2025 handover independently reports the table cannot be queried. Three documents describe the procedure as operating (v0.3 §3, v1.1 §4, the registry), and the queue export shows a monotonically growing `PENDING` population that contradicts all three → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sellflow-docs:context/handover/2025-03_정산팀_인수인계.md, sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv

**The manual fallback is reactive, not periodic.** v1.1 §4 requires "정산팀 확인 (월 1회 이상)" ("settlement team review, at least monthly") and §5 requires a recorded monthly review form (ST-F-002). The handover says what really happens: "실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음" ("in practice only the cases partners asked about have been checked and handled"), with the explicit warning "전체 대기열을 주기적으로 확인하는 절차는 없음" ("there is no procedure for periodically reviewing the whole queue"). Corrections done this way leave no system record, which is why the export's totals are a lower bound → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md, sellflow-docs:context/handover/2025-03_정산팀_인수인계.md

**The report the partner portal supposedly reads is a log line.** `SettlementReportWriter`'s class comment says "정산 결과 요약을 파일로 떨군다. 파트너 포털이 이 파일을 읽어간다." ("it drops a settlement summary to a file; the partner portal reads that file"), while `write()` contains only `log.info(...)`. Its own TODO from 2024-11-02 flags a second gap: "취소 정정분은 이 리포트에 포함되지 않는다. 별도 확인 필요." ("cancellation corrections are not included in this report; needs separate checking") → settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java

**Duplicate runs are possible and have happened.** Clustered Quartz is the only guard, and it failed once: on 2025-07-12 two instances triggered at 02:04 because "배치 서버 이중화 작업 중 Quartz 클러스터 설정이 한쪽에만 반영됨" ("during batch-server duplication work the Quartz cluster setting was applied to only one side"), producing duplicate payment requests for 17 partners worth about 4,200만원 (~42M KRW), all netted off the following month. Spring Batch offers no protection here either: the `ts` job parameter makes every launch unique, which permits re-runs rather than preventing them. The postmortem's idempotency-key action item remains "담당 미지정" ("no owner assigned") → sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md, settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java

**Writes are row-at-a-time inside a chunk.** `SettlementItemWriter.write()` loops a chunk of up to 500 items issuing one `jdbcTemplate.update()` per order rather than a batch update, and resolves `runId` once per chunk with a separate query → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java

Error handling across all of this — what throws, what is swallowed, and what is merely logged — is collected in [[PROC-SETTLEMENT-ERROR-HANDLING]]; the full inventory of runnable and unrunnable flows is [[PROC-SETTLEMENT-FLOW-CATALOG]]; the run-log table that records none of it is [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]].

## Runtime Components

| Component | Tech Stack | Purpose | Entry Point |
|---|---|---|---|
| `SettlementBatchApplication` | Spring Boot 2.3.12, Java 8 | The single process; no web starter, no HTTP port | `src/main/java/kr/co/sellflow/settlement/SettlementBatchApplication.java` |
| `dailySettlementTrigger` → `DailySettlementQuartzJob` | Quartz cron `0 0 2 * * ?` (Asia/Seoul), clustered JDBC store | Launches the Spring Batch job with `jungsanIlja` and `ts` parameters | `src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java` |
| `dailySettlementJob` / `settlementStep` | Spring Batch 4, `JdbcCursorItemReader`, chunk 500 | Read delivered orders, compute fee and payable, insert `SETTLEMENT_DTL` | `src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java` |
| `markSettledStep` / `MarkSettledTasklet` | Spring Batch tasklet, `JdbcTemplate` | Single UPDATE flipping `ORDER_MST.SANGTAE_CD` to `JUNGSAN_WANRYO` | `src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java` |
| `orderEventRelayTrigger` → `OrderEventRelayJob` | Quartz simple schedule, every 10 min forever, `JdbcTemplate` | Poll `ORDER_EVENT_OUTBOX`, enqueue post-settlement cancellations, acknowledge rows | `src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java` |
| `CancelReconciler` | `@Component`, `JdbcTemplate` — **unscheduled** | Would drain `CANCEL_RECON_QUEUE` into `SETTLEMENT_ADJUSTMENT` | No entry point; absent from `QuartzConfig` |
| `SettlementReportWriter` | `@Component`, SLF4J — **uncalled** | Would emit the partner settlement report | No entry point; `write()` only logs |
| `PartnerContractRepository` | `@Repository`, `JdbcTemplate` — **uncalled** | Would load per-partner `FEE_RATE` and `SETTLE_CYCLE` | No entry point |
| Flyway migrator | flyway-core, `classpath:db/migration` | Attempts V1–V6 at startup; V4 cannot succeed against the V1 shape, so how far it gets is unresolved | `src/main/resources/application.yml` |
| Quartz scheduler | spring-boot-starter-quartz, `job-store-type: jdbc`, `isClustered: true` | Trigger persistence and cluster coordination | `src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java` |
| CD workflows (dev/stage/prod) | GitHub Actions, `workflow_dispatch` | Build with `-x test`, then call an absent `./deploy.sh` | `.github/workflows/settlement-batch-{dev,stage,prod}-cd.yml` |

## Related

- [[SCH-SETTLEMENT-BATCH]] — the tables this service owns and the migrations that do not line up with them
- [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] — where every `SETTLEMENT_DTL` column's value comes from
- [[API-SETTLEMENT-BATCH]] — the trigger-and-table interface that stands in for an HTTP API
- [[PROC-SETTLEMENT-DAILY-BATCH]] — step-by-step behaviour of `dailySettlementJob`
- [[PROC-SETTLEMENT-FLOW-CATALOG]] — every flow in the service, running and dormant
- [[PROC-SETTLEMENT-RUN-LIFECYCLE]] — the `SETTLEMENT_RUN` row nobody writes
- [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] — the `SETTLEMENT_RUN_LOG` table nobody writes either
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — the queue's states and the transitions that never fire
- [[PROC-SETTLEMENT-CORRECTION]] — the documented correction procedure against what the code does
- [[PROC-SETTLEMENT-ERROR-HANDLING]] — what fails loudly, what fails silently
- [[PROC-SETTLEMENT-AUTH]] — authentication posture, such as it is
- [[PROC-SELLFLOW-RUNTIME]] — where this service sits in the runtime topology
- [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] — the 2023 decision to compensate SF-2287 with a next-month clawback
- [[DEC-SELLFLOW-SHARED-DB]] — the 2019 consolidation that makes table-level integration possible
- [[DEC-SELLFLOW-MONEY-ROUNDING]] — why settlement truncates and order rounds
- [[RISK-SETTLEMENT]] — collected risk themes for this service
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the 4,127-row, 188,851,520 KRW exposure
- [[RISK-SELLFLOW-DOC-DRIFT]] — the pattern of documents describing behaviour the code does not have
- [[GLOSSARY-SETTLEMENT]] — settlement domain vocabulary
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — the naming convention shared with order-service
- [[CON-ORDER-SETTLEMENT]] — the shared-database boundary with order-service
- [[SYS-ORDER]] — the upstream producer of orders and cancellation events
- [[SYS-SETTLEMENT-ANOMALY]] — the detection service reading this service's output
- [[API-SETTLEMENT-ANOMALY]] — the detection service's HTTP surface, including the CANCELLED_SETTLED rule
- [[SCH-SETTLEMENT-ANOMALY]] — the tables that detection writes
