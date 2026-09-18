---
id: "RISK-SETTLEMENT"
type: "risk"
title: "Settlement Service Risk Themes"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-scanned 2026-09-19 at deep depth: every file of settlement-batch read in full (16 production classes, 588 lines, 6 migrations, 3 workflows, 1 test), plus a reachability grep for each class name across src/, a TODO/FIXME/XXX/HACK sweep, and a column-by-column reconciliation of the Flyway migrations against the SQL the code issues. Severity was raised from medium to high on the basis of that density. Stale if the postmortem's three open items close, if a test suite appears, or if CancelReconciler is scheduled."
freshness_triggers:
  - "settlement-anomaly/model/features.py"
  - "settlement-batch/.env.dev"
  - "settlement-batch/.github/workflows/settlement-batch-prod-cd.yml"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
  - "settlement-batch/src/main/resources/db/migration/V4__cancel_recon_queue_index.sql"
  - "settlement-batch/src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java"
  - "sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
known_unknowns:
  - "Whether the .env.dev credentials are still valid. The file commits a hostname and username but no password; whether that account exists was not checked."
  - "The financial size of the fee-rate exposure. How many partners hold a premium or promotional contract is unknown — PARTNER_CONTRACT was not queried, and no export of it exists in sources/raw."
  - "Whether any duplicate settlement has occurred since 2025-07-12. Without DUP_SETTLE implemented and without any code writing SETTLEMENT_RUN, there is no detector and no run history to inspect."
  - "Whether V4's CREATE INDEX ever executed. It names a column, REG_DT, that CANCEL_RECON_QUEUE does not have; if Flyway ran it against MySQL it would have failed and blocked V5 and V6, so either the deployed table differs from V1 or the migrations have not all been applied. No schema dump was available to decide."
  - "Where SETTLEMENT_ADJUSTMENT is defined, if anywhere. CancelReconciler inserts into it, no migration in any of the five repos creates it, and the 2025 handover records that it cannot be queried."
  - "Whether -x test in the CD workflows was a deliberate policy or an expedient. No decision record was found."
  - "What deploy.sh does. It is invoked by all three workflows, is not in the repository, and is the only place a deployment credential could live."
  - "Whether the anomaly model still produces meaningful AMT_OUTLIER scores after 22 months without retraining. No score distribution was available."
severity: "high"
resolution: "open"
tags:
  - settlement
  - risk
  - technical-debt
  - operations
  - dead-code
aliases:
  - "정산 리스크"
relates_to:
  - type: "refines"
    target: "[[API-SETTLEMENT-ANOMALY]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-MONEY-ROUNDING]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-AUTH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-FLOW-CATALOG]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "수수료 sheet — the unapplied rate tiers"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "§3 — SETTLEMENT_ADJUSTMENT cannot be queried"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "P2 incident and its five follow-up items, three still open"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512 open since 2023-04-24, unassigned"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/features.py"
    notes: "no retraining since 2024-11"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.env.dev"
    notes: "Committed dev connection settings"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.github/workflows/settlement-batch-prod-cd.yml"
    notes: "workflow_dispatch only, builds with -x test, invokes an absent deploy.sh"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/DateUtil.java"
    notes: "Dead: no caller anywhere in src/"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
    notes: "Dead in production; referenced only by the single test"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Registers two of the three schedulable components"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
    notes: "Hardcoded 0.12, HALF_UP"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
    notes: "Dead: the clawback that SF-2287 was compensated with"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
    notes: "Dead: logs instead of writing the partner-portal file"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java"
    notes: "Dead: the only reader of the contracted fee rate"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql"
    notes: "Indexes REG_DT, a column CANCEL_RECON_QUEUE does not have"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
    notes: "Adds PROCESSED_AT/PROCESSED_BY; the code writes PROCESSED_DTM"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java"
    notes: "The repository's only test; does not touch the class it names"
notes: "Upgraded from snorkel to deep depth on 2026-09-19. The 2026-09-18 pass rated this medium and explicitly called that a floor; the systematic scan supports high."
---

# Settlement Service Risk Themes

## Description

This is a systematic scan of `settlement-batch`, with the findings from `settlement-anomaly` that bear directly on settlement correctness. Every production class was read and then grepped for callers; every migration was reconciled against the SQL the code issues; every workflow was read in full.

The repository is small — 16 production classes, 588 lines, 6 migrations, 3 workflows and 1 test — which makes the density measurable rather than impressionistic. **Six of the sixteen production classes have no caller.** The three that matter most are the three that would have closed this domain's three largest gaps: `CancelReconciler` (the clawback that SF-2287 was compensated with), `PartnerContractRepository` (the only reader of the contracted fee rate), and `SettlementReportWriter` (the partner-facing report). Each is fully written, each is annotated as a Spring bean, and each is unreachable.

The largest single exposure — the correction backlog — has its own artifact, [[RISK-SETTLEMENT-RECON-BACKLOG]], because it carries a measured financial figure and a named stakeholder waiting on an answer. Failure and recovery behaviour has its own artifact, [[PROC-SETTLEMENT-ERROR-HANDLING]]. Everything specific to the anomaly service is in [[RISK-SETTLEMENT-ANOMALY]].

## Key Facts

### Theme 1 — Dead components (6 of 16 production classes)

- `CancelReconciler` is a `@Component` with no Quartz registration and no caller in any of the five repos; `QuartzConfig` declares exactly two `JobDetail` beans and neither is it → `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`
- Its own javadoc says so, dated: "TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤" — "TODO: needs Quartz schedule registration (to be added to QuartzConfig after checking with 박성민)" → `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`
- SF-4512 "CancelReconciler Quartz 스케줄 등록" has been `To Do`, priority Low, with no assignee and no story points since 2023-04-24 → `sellflow-docs:context/sprints/tickets_2026-S17.csv`
- `CANCEL_RECON_QUEUE` held 4,127 `PENDING` rows totalling 188,851,520 KRW as of 2026-09-01 → `sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv`
- `SettlementReportWriter.write()` emits one log line and writes no file, despite its javadoc "정산 결과 요약을 파일로 떨군다. 파트너 포털이 이 파일을 읽어간다." — "drops a settlement result summary to a file; the partner portal reads this file". It is called by no step (grep: the only occurrences of the class name are in its own file) → `settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java`
- `PartnerContractRepository` is a `@Repository` holding the one query that reads `PARTNER_CONTRACT.FEE_RATE`, and nothing calls it; `PartnerContract`, the class it returns, is referenced only by that repository → `settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/PartnerContract.java`
- `MoneyUtil` is referenced only by the single test file; the production fee calculation does not use it → `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java`, `settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java`
- `DateUtil` has no reference anywhere outside its own file, including `DateUtil.settlementBaseDate()`, which implements the one piece of correct date reasoning in the repository: "배치가 02:00 에 돌기 때문에, 00:00~02:00 사이에 수동 실행되면 전전일이 기준일이 되어야 한다" — "because the batch runs at 02:00, a manual run between 00:00 and 02:00 should use the day before yesterday as the base date" → `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/DateUtil.java`
- What runs instead is `DailySettlementQuartzJob`'s inline `LocalDate.now().minusDays(1).toString()` — no timezone (system default, while the whole domain is Asia/Seoul), ISO format `yyyy-MM-dd` where `DateUtil` uses `yyyyMMdd`, and no 00:00–02:00 correction → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`

### Theme 2 — Hardcoded commission rate

- `SettlementItemProcessor` declares `private static final BigDecimal DEFAULT_FEE_RATE = new BigDecimal("0.12")` and never reads `PARTNER_CONTRACT.FEE_RATE` → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`
- It documents its own gap: "파트너별 계약 수수료율은 PARTNER_CONTRACT 에 있으나, 현재는 기본 수수료율만 적용한다. (2021년 이후 미정비)" — "per-partner contract rates are in PARTNER_CONTRACT, but currently only the base rate is applied (unmaintained since 2021)" → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`
- Two contracted tiers exist on paper and are not applied: 프리미엄 9.5% for partners above 100M KRW monthly volume and 신규 프로모션 6.0% within three months of onboarding → `sellflow-docs:context/business-rules.md` (수수료 sheet)
- The workbook confirms the data side independently: "계약별 수수료율은 PARTNER_CONTRACT 에 있으나 2021년 이후 미정비. 현재 전 건 기본 수수료율 적용 중." — "per-contract rates are in PARTNER_CONTRACT but unmaintained since 2021; the base rate is applied to everything" → `sellflow-docs:context/business-rules.md` (수수료 sheet)
- Every affected partner is overcharged on every order, silently: the batch produces no comparison, and the `FEE_MISMATCH` detector that would catch it exists only as a README table row → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`, `settlement-anomaly:model/detector.py`
- The rounding mode compounds it. The processor uses `RoundingMode.HALF_UP`; `MoneyUtil` — written for this calculation and unused — uses `FLOOR` under the note "정산은 절사(FLOOR), 주문은 반올림(HALF_UP). 2022 협의 결과이며 문서화되어 있지 않다." — "settlement truncates, order rounds; agreed in 2022 and not documented" → `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java`

### Theme 3 — Migrations that do not match the code or the schema

- V4 creates an index on a column that does not exist: `CREATE INDEX IDX_CANCEL_RECON_STATUS ON CANCEL_RECON_QUEUE (STATUS, REG_DT)`, while V1 defines the timestamp column as `RECV_DTM` and there is no `REG_DT` → `settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- V1 already created the equivalent index, `KEY IX_CANCEL_RECON_QUEUE_01 (STATUS, RECV_DTM)`, so V4 is both wrong and redundant → `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- V5 adds `PROCESSED_AT` and `PROCESSED_BY`, while `CancelReconciler` writes `PROCESSED_DTM` — the column V1 already defined. The new columns have no writer and the migration says so: "정정 처리 결과를 남기기 위한 컬럼. 아직 쓰는 코드는 없다." — "columns for recording correction results; no code writes them yet" → `settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`
- **`SETTLEMENT_ADJUSTMENT` has no DDL anywhere.** `CancelReconciler` inserts into it with `ADJ_TYPE='CANCEL_CLAWBACK'`; no migration in settlement-batch creates it, and a grep across all five repos finds the name only in that one class → `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- The 정산팀 hit the same wall from the operations side in 2025: "`SETTLEMENT_ADJUSTMENT` 테이블이 문서에는 나오는데 실제로 조회가 안 됨. WIP" — "the SETTLEMENT_ADJUSTMENT table appears in the documentation but cannot actually be queried" → `sellflow-docs:context/handover/2025-03_정산팀_인수인계.md` (§3)
- So even if SF-4512 were closed tomorrow and `CancelReconciler` were scheduled, its first statement would be an INSERT into a table with no evidence of existing — the clawback would fail on its first row → `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`
- V2 creates `SETTLEMENT_RUN_LOG` (`RUN_ID VARCHAR(32)`, `STATUS`, `STARTED_AT`, `ENDED_AT`, `ROW_CNT`) and V6 indexes it; nothing reads or writes it, and its `RUN_ID` is a `VARCHAR(32)` while `SETTLEMENT_RUN.RUN_ID` is a `BIGINT AUTO_INCREMENT` → `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql`, `settlement-batch:src/main/resources/db/migration/V6__settlement_run_log_index.sql`
- Nothing writes `SETTLEMENT_RUN` either, though `SettlementItemWriter` depends on a row in it with `SANGTAE='RUNNING'` to supply the `NOT NULL` `RUN_ID` of every settlement detail row → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`

### Theme 4 — The 2025-07-12 duplicate run and its open items

- A P2 incident on 2025-07-12 duplicated payment requests for 17 partners, roughly 42,000,000 KRW, when Quartz cluster settings were applied to only one of two nodes during a redundancy change → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- The root-cause note names the structural weakness: "`SettlementItemWriter` 는 지급 요청 전송 후 되돌릴 수 없음. 취소 경로 없음." — "SettlementItemWriter cannot be undone after the payment request is sent; there is no cancellation path" → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- Three of the five follow-up items remain unchecked: the idempotency key before the payment request (담당 미지정, no owner assigned); the check on whether cancelled orders are excluded from settlement (raised 2025-07-15, no discussion since); and "정산 관련 배치 전수 점검 (스케줄 등록 여부 포함)" — "a full audit of settlement batches, including whether they are registered on a schedule" → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- That third item is the one that would have found `CancelReconciler`. It was written down in July 2025 and never done; the queue has grown since → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`, `sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv`
- Nothing prevents a recurrence at the application level: `DailySettlementQuartzJob` adds a `ts` job parameter that makes every launch a distinct `JobInstance`, disabling Spring Batch's own duplicate protection → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`
- The detection type that would catch it, `DUP_SETTLE`, is documented in the anomaly README and not implemented → `settlement-anomaly:README.md`, `settlement-anomaly:model/detector.py`

### Theme 5 — Error handling and bare catches

- The repository contains exactly one `try`/`catch`, and it catches `Exception` wholesale, wraps it in an `IllegalStateException` and rethrows: there is no logging, no classification, and no notification in the handler → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`
- Neither batch step declares `.faultTolerant()`, a skip policy, a retry policy or a listener, so any single bad row fails the whole job with prior chunks already committed → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- The relay's payload parser silently substitutes `"00"` for the cancellation reason whenever the marker is absent, and `"00"` is not one of the four defined reason codes → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `sellflow-docs:context/business-rules.md` (취소정책 sheet)
- The relay acks every event it inspects, so a cancellation relayed before that day's `SETTLEMENT_DTL` row exists is dropped permanently — traced step by step in [[PROC-SETTLEMENT-ERROR-HANDLING]] → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`

### Theme 6 — Test coverage

- `settlement-batch` contains exactly one test file with one test method, against 16 production classes → `settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java`
- That test is named `SettlementItemProcessorTest` but never instantiates `SettlementItemProcessor`; it asserts `MoneyUtil.fee(14999, 0.1) == 1499` → `settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java`
- It therefore asserts `FLOOR` behaviour while the processor it is named after uses `HALF_UP` — the test would pass unchanged if the processor's arithmetic were deleted → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java`
- `settlement-anomaly`'s single test asserts that a string appears in a list of feature names and never calls the detector → `settlement-anomaly:tests/test_detector.py`

### Theme 7 — Delivery pipeline

- All three CD workflows — dev, stage and prod — are `on: workflow_dispatch` only, so there is no build on push, no CI check on a pull request, and every production deploy is a manual button press → `settlement-batch:.github/workflows/settlement-batch-dev-cd.yml`, `settlement-batch:.github/workflows/settlement-batch-stage-cd.yml`, `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`
- All three run `./gradlew clean build -x test`, so the one test that exists does not run on the way to production either → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`
- All three then invoke `./deploy.sh $ENVIRONMENT`, a script that is not in the repository; there is no Dockerfile either, so the production deployment path is not in version control → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`, `sellflow-docs:infra/settlement/runtime.md`
- None of the three declares a `permissions:` block, an `environment:` gate, a reviewer or any `secrets.*` reference — see [[PROC-SETTLEMENT-AUTH]] → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`
- `.env.dev` is committed with a database hostname and username (`DB_URL=jdbc:mysql://settlement-db-dev.internal:3306/settlement`, `DB_USER=settle_dev`) and disagrees with `application.yml` about both host and schema → `settlement-batch:.env.dev`, `settlement-batch:src/main/resources/application.yml`
- `settlement-anomaly` has no CI or CD workflow at all → `sellflow-docs:infra/settlement/runtime.md`

## Findings

Ordered by severity. "Reach" is how much of the settlement domain the finding touches.

| # | Theme | Finding | Evidence | Reach | Severity |
|---|---|---|---|---|---|
| 1 | Dead components | `CancelReconciler` never scheduled; the SF-2287 clawback has never run | `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`, `QuartzConfig.java`, SF-4512 | 4,127 rows / 188,851,520 KRW | **high** |
| 2 | Migrations | `SETTLEMENT_ADJUSTMENT` has no DDL in any repo; the handover confirms it cannot be queried | `CancelReconciler.java`, `sellflow-docs:context/handover/2025-03_정산팀_인수인계.md` §3 | Blocks any fix to #1 | **high** |
| 3 | Hardcoded value | `DEFAULT_FEE_RATE = 0.12` applied to every partner; `PartnerContractRepository` dead | `SettlementItemProcessor.java`, `PartnerContractRepository.java`, business-rules 수수료 | Every order, every day | **high** |
| 4 | Incident | No idempotency guard before an irreversible payment request; `ts` parameter defeats Spring Batch's duplicate check | `DailySettlementQuartzJob.java`, postmortem | One recurrence ≈ 42M KRW | **high** |
| 5 | Error handling | Relay acks before acting, permanently dropping pre-settlement cancellations | `OrderEventRelayJob.java`, `sellflow-docs:infra/settlement/queues.md` | Unmeasured, continuous | **high** |
| 6 | Migrations | Nothing writes `SETTLEMENT_RUN`, yet `SETTLEMENT_DTL.RUN_ID` is `NOT NULL` and sourced from it | `SettlementItemWriter.java`, `V1__settlement_schema.sql` | Whole batch | **high** |
| 7 | Test coverage | One test for 16 classes, asserting an unused helper under the name of the class it does not test | `SettlementItemProcessorTest.java` | Whole repo | **medium-high** |
| 8 | Pipeline | Prod deploys are `workflow_dispatch` only, build with `-x test`, and call an absent `deploy.sh` | the three workflow files | Whole repo | **medium-high** |
| 9 | Dead components | `SettlementReportWriter` logs instead of writing the file the partner portal is said to read; its TODO notes corrections are excluded | `SettlementReportWriter.java` | Partner-facing reporting | **medium** |
| 10 | Migrations | V4 indexes `REG_DT`, a column `CANCEL_RECON_QUEUE` does not have, duplicating an index V1 already created | `V4__cancel_recon_queue_index.sql`, `V1__settlement_schema.sql` | Migration chain integrity | **medium** |
| 11 | Migrations | V5's `PROCESSED_AT`/`PROCESSED_BY` have no writer; the code writes V1's `PROCESSED_DTM` | `V5__add_recon_processed_columns.sql`, `CancelReconciler.java` | Correction audit trail | **medium** |
| 12 | Migrations | `SETTLEMENT_RUN_LOG` (V2, indexed by V6) has no reader or writer and a `RUN_ID` type that disagrees with `SETTLEMENT_RUN` | `V2__add_settlement_run_log.sql`, `V6__settlement_run_log_index.sql` | Run observability | **medium** |
| 13 | Error handling | Relay's `sayuCd` parser silently yields `"00"`, a code outside the four defined values, and can throw on a short payload | `OrderEventRelayJob.java`, business-rules 취소정책 | Every queued correction | **medium** |
| 14 | Dead components | `DateUtil.settlementBaseDate()` implements the correct KST/02:00 rule and is called by nothing; the inline replacement has no timezone | `DateUtil.java`, `DailySettlementQuartzJob.java` | Base-date correctness | **medium** |
| 15 | Rounding | Production path rounds `HALF_UP` against a documented 2022 agreement that settlement truncates | `SettlementItemProcessor.java`, `MoneyUtil.java` | Sub-won per order, in the platform's favour | **medium** |
| 16 | Config | `.env.dev` committed; contradicts `application.yml` on host and schema | `.env.dev`, `application.yml` | Dev only | **low-medium** |
| 17 | TODO/FIXME | Only two TODO markers exist in the whole repo, and both mark load-bearing gaps (#1 and #9). No FIXME, XXX or HACK markers anywhere | grep over `settlement-batch:src/` | — | informational |

The last row is worth stating explicitly, because it inverts the usual reading of a TODO sweep: a low TODO count here does not indicate low debt. It indicates that most of the debt in this repository was never marked.

## Impact

**Financial.** The commission-rate gap misprices every order for any partner holding a non-base contract, continuously and undetectably; a single 45,500 KRW order costs a 프리미엄 partner 1,138 KRW more than contracted. The rounding inconsistency adds a sub-won-per-order drift in the platform's favour. Neither has been sized, because `PARTNER_CONTRACT` was not queried. The measured figures in this domain are the 2025-07-12 incident (≈42,000,000 KRW across 17 partners, resolved by next-month offset rather than recovery) and the unreconciled queue (188,851,520 KRW across 4,127 rows) → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`, `sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv`

**Operational.** A team of three, reduced from four in 2024-07, runs an irreversible daily payout with one test that does not test the code it names, deployed by a manually dispatched workflow that skips that test and calls a script outside version control. There is no application-level guard against the exact failure that occurred in 2025, and no runbook for re-running a failed day → `sellflow-docs:context/org-chart.md` (변경이력), `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`

**Structural.** The fix for the largest finding is blocked by the second largest: scheduling `CancelReconciler` cannot work until `SETTLEMENT_ADJUSTMENT` exists, and no one has recorded where that table was supposed to come from. The 2026 automation plan that intends to replace this work by agent is sized on a 2023 estimate of 월 10건 내외 (around ten cases a month) that the queue export contradicts by about an order of magnitude — the export runs at roughly 100-160 rows a month against an estimate of fewer than 10, i.e. around 15x → `sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, `sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv`

**Detection.** The service intended as the safety net covers half its documented surface, has not been retrained in 22 months, has no confirmed caller and produces rows nobody owns. Its two missing rules are precisely the two failure modes this domain has actually experienced: duplicate settlement and fee mismatch → `settlement-anomaly:model/detector.py`, `settlement-anomaly:README.md`, see [[RISK-SETTLEMENT-ANOMALY]]

## Severity and Resolution

**Severity: high.** Raised from the 2026-09-18 assessment of medium, which that pass explicitly recorded as "a floor, not a judgement" pending a systematic scan. This is that scan, and the rating is justified on density rather than on any single finding:

- **37.5% of production classes are unreachable** (6 of 16), and the dead set is not incidental scaffolding — it is the clawback, the contract-rate reader, the partner report and the correct date rule. The system's designed behaviour and its actual behaviour differ by exactly those four capabilities.
- **Two thirds of the migrations are defective or orphaned** (V2, V4, V5 and V6 of six — 67%: one indexes a nonexistent column, one adds columns nothing writes, two build a run-log table nothing touches), and the single most important table in the correction path has no migration at all.
- **One test covers 588 lines across 16 classes, and it tests a class that production does not call**, under a name that claims to test the one that it does. Coverage is not merely thin; it is misleading, and it does not run in CI because CI is `-x test` and there is no CI on push.
- **Every finding sits on an irreversible path.** `SettlementItemWriter`'s javadoc and the 정산팀 handover agree that a sent payment request cannot be recalled, so defects here are not corrected — they are absorbed by a manual next-month deduction performed by a three-person team that, per the handover, only looks at cases a partner asks about.
- **Two findings carry measured KRW figures** (42,000,000 and 188,851,520), and both were produced by mechanisms still in place today.

Individually, several of these findings are ordinary technical debt. Together, in a 588-line service that moves partner money every night with no test gate, no idempotency guard, no run record and no owner for its follow-ups since 2023, they describe a system whose correctness is unverified rather than merely untested.

**Resolution: open.** Nothing here is mitigated. The three open items from the 2025-07-12 postmortem and SF-4512 are the only tracked remediation; SF-4512 has been unassigned since 2023-04-24 and the postmortem items have sat untouched since 2025-07 → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`, `sellflow-docs:context/sprints/tickets_2026-S17.csv`

## Recommended Actions

Ordered so that each step is executable given the state of the one before it. None of these is a decision this artifact can make; each needs the owning team.

1. **Establish whether `SETTLEMENT_ADJUSTMENT` exists in production.** One `SHOW CREATE TABLE` answers it. Every plan involving the clawback depends on the answer, and it has been an open question since at least 2025-03 (finding #2).
2. **Determine who creates the `SETTLEMENT_RUN` row.** The daily batch cannot be reasoned about — or safely re-run — until the run lifecycle has a named owner (finding #6).
3. **Do the audit the postmortem already ordered:** "정산 관련 배치 전수 점검 (스케줄 등록 여부 포함)". It is written down, it is unassigned, and it is the item that surfaces findings #1 and #14 without any new analysis.
4. **Add the idempotency key** the postmortem's second open item names, before the payment request. It is the only control that would have prevented the one incident this domain has recorded (finding #4).
5. **Reorder the relay** so the outbox row is acked only after the queue insert, within one transaction — and decide what should happen to a cancellation whose order is not yet settled, rather than dropping it (finding #5).
6. **Quantify the fee-rate exposure** by counting `PARTNER_CONTRACT` rows whose `FEE_RATE` differs from 0.12 and joining against a month of `SETTLEMENT_DTL`. This converts finding #3 from a described risk into a number, which is what the 재무기획팀 enquiry of 2026-08-25 is already asking for in the adjacent case.
7. **Make the one existing test run in CI, then make it test the production path.** Removing `-x test` is a one-line change; pointing the test at `SettlementItemProcessor` exposes the `HALF_UP`/`FLOOR` divergence immediately (findings #7, #8, #15).
8. **Reconcile the migration chain** — V4's `REG_DT`, V5's unwritten columns, V2/V6's unused table — against the deployed schema before adding any further migration (findings #10, #11, #12).

## Related

- [[RISK-SETTLEMENT-RECON-BACKLOG]] — finding #1, quantified
- [[RISK-SETTLEMENT-ANOMALY]] — the detector that would have caught findings #3 and #4, and why it does not
- [[PROC-SETTLEMENT-ERROR-HANDLING]] — findings #5, #13 and the recovery behaviour behind them
- [[PROC-SETTLEMENT-FLOW-CATALOG]] — which of this domain's flows actually run
- [[PROC-SETTLEMENT-DAILY-BATCH]] — where findings #3, #6, #7 and #15 originate
- [[PROC-SETTLEMENT-CORRECTION]] — the full trace behind finding #1
- [[PROC-SETTLEMENT-AUTH]] — the deployment and database-access side of finding #8
- [[API-SETTLEMENT-ANOMALY]] — the missing caller noted in the detection impact
- [[DEC-SELLFLOW-MONEY-ROUNDING]] — the rounding divergence behind finding #15
- [[SYS-SETTLEMENT]] — the service being scanned
