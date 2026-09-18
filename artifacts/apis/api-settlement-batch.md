---
id: "API-SETTLEMENT-BATCH"
type: "api"
title: "Settlement Batch Interface Surface"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened 2026-09-19 by re-reading every file in settlement-batch — build.gradle, application.yml, .env.dev, README.md, all three CD workflows, all six migrations and all seventeen Java classes — plus sources/apis/settlement/batch/openapi.meta.json, sources/infra/settlement/{runtime.md,queues.md} and sources/context/registry/services.yaml. This pass added the fourth interface the artifact was missing: the GitHub Actions workflow_dispatch deploy entry point, which is the only surface a human can operate by hand. It also added the Spring Batch job/step contract as a first-class section, the two queue tables as a named queue interface with their producer/consumer rows from the registry, the verification procedure for the no-HTTP-surface claim, and a How Agents Should Use This section. Nothing found on this pass contradicted the previous one. Becomes stale the moment spring-boot-starter-web or a controller is added, a third Quartz trigger is registered, or a CD workflow gains a push/schedule trigger."
freshness_triggers:
  - ".github/workflows/settlement-batch-prod-cd.yml"
  - "build.gradle"
  - "src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java"
  - "src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "src/main/resources/application.yml"
known_unknowns:
  - "Whether an operator has any supported way to trigger a settlement run manually. No admin endpoint, no CLI entry point and no runbook procedure was found; the only manual control in version control is re-running a CD workflow, which redeploys rather than launching a job."
  - "What deploy.sh does and whether it passes job parameters. All three CD workflows invoke ./deploy.sh $ENVIRONMENT, and the script is not in the repo."
  - "Whether the Quartz JDBC job store lives in the same sellflow_order schema. application.yml sets job-store-type jdbc with no separate datasource, which implies yes, but no QRTZ_ DDL exists in the repo to confirm it."
  - "Which datasource production actually uses. application.yml resolves to sellflow_order via DB_HOST, while .env.dev sets DB_URL to jdbc:mysql://settlement-db-dev.internal:3306/settlement — a variable no code reads. Whether a prod profile overrides this was not determinable from the repo."
  - "Whether QUARTZ_ENABLED has any effect. It is set in .env.dev and read by nothing in the repo (verified by grep), so there is no in-repo kill switch for the schedulers."
  - "Whether anything outside these five repos reads SETTLEMENT_DTL or the settlement report. The partner portal is named in a javadoc but no integration code was found."
  - "Whether the SETTLEMENT_RUN row that the writer depends on is created by an external tool, a DBA procedure or a person. No INSERT INTO SETTLEMENT_RUN exists in any of the five repos."
tags:
  - settlement
  - quartz
  - batch
  - no-http-api
aliases:
  - "settlement-batch triggers"
  - "정산 배치 인터페이스"
relates_to:
  - type: "depends_on"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/settlement/batch/openapi.meta.json"
    notes: "Tier-4 extraction record: api null, reason 'settlement-batch exposes no HTTP surface'"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet — the 01-04 reason-code vocabulary the relay's '00' fallback falls outside of"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "queues: block naming the producer and consumer of both queue tables; CANCEL_RECON_QUEUE consumer is TODO"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "The duplicate-run incident that turns on Quartz clustering"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512 'CancelReconciler Quartz 스케줄 등록', still To Do with no assignee"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Tier-4 queue extraction: no broker, both queues are polled tables"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "Runtime extraction: workflow_dispatch-only deploys, -x test, absent deploy.sh and Dockerfile"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "4,127 PENDING rows / 188,851,520 KRW — the size of what a registered third trigger would process"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.env.dev"
    notes: "DB_URL and QUARTZ_ENABLED, neither of which any code reads"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.github/workflows/settlement-batch-prod-cd.yml"
    notes: "workflow_dispatch-only prod deploy, build with -x test, then ./deploy.sh"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:README.md"
    notes: "The job/step summary and the irreversibility statement about payment requests"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:build.gradle"
    notes: "No spring-boot-starter-web dependency"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/SettlementBatchApplication.java"
    notes: "Bare @SpringBootApplication, no servlet configuration, binds no port"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/DateUtil.java"
    notes: "settlementBaseDate, yyyyMMdd and KST — called by nothing"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
    notes: "FLOOR rounding, also called by nothing outside the test"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "Job and step wiring, chunk size 500, and the reader SQL bound to the jungsanIlja job parameter"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
    notes: "Hardcoded 0.12 fee rate with HALF_UP rounding"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
    notes: "The SETTLEMENT_DTL insert and the SELECT MAX(RUN_ID) WHERE SANGTAE='RUNNING' lookup"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
    notes: "Declared for the partner portal, invoked by no step"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java"
    notes: "The PARTNER_CONTRACT lookup with no caller in any of the five repos"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
    notes: "spring.batch.job.enabled false, clustered JDBC Quartz, the sellflow_order datasource"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "The 2019 shared-instance note and the SF-2287 manual-queue comment"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java"
    notes: "The repo's only test — asserts MoneyUtil FLOOR, never touches the processor it is named for"
notes: "Deliberately thin on HTTP because there is none. The honest artifact is the trigger, job, table and deploy contract."
---

# Settlement Batch Interface Surface

## Overview

`settlement-batch` has no HTTP API. This artifact exists so that a reader looking for `API-SETTLEMENT-BATCH` finds the answer rather than an absence, and so that the interface the service actually has gets written down somewhere.

That interface has four parts, and none of them is a request:

1. **Two Quartz triggers** — the only things that start work on their own.
2. **One Spring Batch job of two steps** — the contract between the trigger and the database, parameterised by exactly one value.
3. **Two database tables used as queues** — the only way another service hands work to this one, or to anyone downstream.
4. **Three GitHub Actions workflows, all `workflow_dispatch`** — the only manual control surface anyone has, and it deploys rather than runs.

The distinction matters for anyone integrating with settlement: there is nothing to call. You either wait for a trigger to fire, or you write a row to a table the batch reads.

## Key Facts

- `build.gradle` declares `spring-boot-starter-batch`, `-quartz`, `-jdbc` and `flyway-core`, but not `spring-boot-starter-web` → build.gradle
- No `@RestController`, `@Controller`, `@RequestMapping` or `starter-web` string exists anywhere in the repo (verified by a recursive grep for all four over the whole tree on 2026-09-19; zero hits) → src/main/java/kr/co/sellflow/settlement/
- The source-extraction record reaches the same conclusion independently and writes `"api": null` with the reason "settlement-batch exposes no HTTP surface" → sources/apis/settlement/batch/openapi.meta.json
- `SettlementBatchApplication` is a bare `@SpringBootApplication` with a `SpringApplication.run` main and no servlet configuration, so the process starts as a non-web application and binds no port → src/main/java/kr/co/sellflow/settlement/SettlementBatchApplication.java
- Exactly two Quartz triggers are registered, both in `QuartzConfig` → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- `dailySettlementTrigger` uses cron `0 0 2 * * ?` in the `Asia/Seoul` timezone → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- `orderEventRelayTrigger` uses `SimpleScheduleBuilder.simpleSchedule().withIntervalInMinutes(10).repeatForever()` → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- `spring.batch.job.enabled: false` prevents Spring Batch from auto-running the job at startup, so Quartz is the sole trigger and a restart does not by itself settle anything → src/main/resources/application.yml, sources/infra/settlement/queues.md
- `CancelReconciler` has no `JobDetail`, no `Trigger` and no caller anywhere in the five repos (verified 2026-09-19 by grepping all five repos for `CancelReconciler` and `reconcileCancellations`; the only hits are its own class declaration, its logger and its own method signature) → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- Quartz runs clustered (`org.quartz.jobStore.isClustered: true`), so a trigger fires on exactly one node — the property that was missing on one node during the 2025-07-12 incident → src/main/resources/application.yml, sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md
- The relay is not a Spring Batch job; it is a `QuartzJobBean` issuing JDBC statements directly, with no chunking, no restartability and no job repository record → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- The trigger and the repository's own date helper disagree about what a settlement date looks like. `DailySettlementQuartzJob` passes `LocalDate.now().minusDays(1).toString()` — the system default zone, ISO `yyyy-MM-dd` — while `DateUtil.settlementBaseDate()` works in `Asia/Seoul`, formats `yyyyMMdd`, and subtracts two days when the hour is before 02:00. `DateUtil` is called by nothing → src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java, src/main/java/kr/co/sellflow/settlement/common/DateUtil.java, sources/apis/settlement/batch/openapi.meta.json
- The mismatch is load-bearing rather than cosmetic: the reader binds that parameter into `DATE(m.UPD_DTM) = ?`, which a `yyyy-MM-dd` string satisfies and a `yyyyMMdd` string does not → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java
- `SettlementItemProcessor` hardcodes `DEFAULT_FEE_RATE` `0.12` with `HALF_UP` rounding and never reads `PARTNER_CONTRACT.FEE_RATE`, explaining itself as "현재는 기본 수수료율만 적용한다. (2021년 이후 미정비)" ("for now only the default fee rate is applied; not tidied up since 2021") → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- The repository's own `MoneyUtil.fee` rounds with `FLOOR` and warns "정산은 절사(FLOOR), 주문은 반올림(HALF_UP). 2022 협의 결과이며 문서화되어 있지 않다." ("settlement truncates, order rounds half up; agreed in 2022 and never documented") — yet `MoneyUtil` is called only by the single unit test, never by the processor, so the two roundings disagree inside this one repository → src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java, src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java
- The one test in the repo is misnamed for what it tests: `SettlementItemProcessorTest.수수료는_절사한다` ("the fee is truncated") asserts `MoneyUtil.fee(14999, 0.1) == 1499` and never instantiates `SettlementItemProcessor`, which rounds `HALF_UP` — so the only test asserts the opposite rule from the one the job applies → src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- `SettlementReportWriter` is a second `@Component` that no step ever calls; its `write(runId)` only logs, and its javadoc claims the partner portal reads the file it does not produce. A TODO from 2024-11-02 adds "취소 정정분은 이 리포트에 포함되지 않는다" ("cancellation adjustments are not included in this report") → src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java, sources/apis/settlement/batch/openapi.meta.json
- All three CD workflows — dev, stage and prod — are triggered `on: workflow_dispatch` and nothing else: no `push`, no `schedule`, no `pull_request` → .github/workflows/settlement-batch-dev-cd.yml, .github/workflows/settlement-batch-stage-cd.yml, .github/workflows/settlement-batch-prod-cd.yml
- Every workflow, prod included, builds with `./gradlew clean build -x test` and then runs `./deploy.sh $ENVIRONMENT`; there is no `deploy.sh` and no Dockerfile in the repository, so the deploy step itself is not in version control → .github/workflows/settlement-batch-prod-cd.yml, sources/infra/settlement/runtime.md
- `.env.dev` declares `DB_URL=jdbc:mysql://settlement-db-dev.internal:3306/settlement` and `QUARTZ_ENABLED=true`, but `application.yml` reads only `DB_HOST` and `DB_USER` and hardcodes the `sellflow_order` schema — so neither variable has any effect, and in particular there is no `QUARTZ_ENABLED` kill switch in the code → .env.dev, src/main/resources/application.yml, sources/infra/settlement/runtime.md
- The service registry names `CANCEL_RECON_QUEUE`'s producer as `settlement-batch (OrderEventRelayJob)` and its consumer as `TODO   # 확인 필요` ("TODO — needs checking"), which is the organisational record of the same gap the code shows → sources/context/registry/services.yaml

## Source of Truth

The authoritative description of this service's invocable surface is `QuartzConfig.java` plus the SQL literals embedded in the job classes, with `DailySettlementJobConfig.java` supplying the job/step wiring and `.github/workflows/*.yml` the deploy entry point. There is no OpenAPI document, no IDL, and no schema registry. The extraction under `sources/apis/settlement/batch/` records a tier-4 read with `api: null` rather than a spec, and points instead at `sources/infra/settlement/runtime.md` and `queues.md`.

For the shape of every table named below, see [[SCH-SETTLEMENT-BATCH]].

### How the "no HTTP surface" claim was verified

State it plainly: **this service exposes no HTTP endpoint of any kind.** Four independent checks, all re-run on 2026-09-19, agree:

| Check | Method | Result |
|---|---|---|
| Dependency | Read `build.gradle` in full | `spring-boot-starter-batch`, `-quartz`, `-jdbc`, `flyway-core`, `mysql-connector-java`, `spring-boot-starter-test`. No `-web`, no `-webflux`, no embedded servlet container → build.gradle |
| Annotation | Recursive grep over the repo for `RestController`, `RequestMapping`, `@Controller` and `starter-web` | Zero hits → src/main/java/kr/co/sellflow/settlement/ |
| Configuration | Read `application.yml` in full | No `server:` block, no `server.port`, no management/actuator configuration → src/main/resources/application.yml |
| Independent extraction | Tier-4 source extraction, run separately | `"api": null`, reason "settlement-batch exposes no HTTP surface. build.gradle declares spring-boot-starter-batch, -quartz and -jdbc but NOT spring-boot-starter-web, and there is no @RestController or @Controller in the repo." → sources/apis/settlement/batch/openapi.meta.json |

A fifth, weaker signal: `services.yaml` gives `endpoints:` for `order-service` and for nothing else in the settlement domain → sources/context/registry/services.yaml

## Resource Map

### 1. Trigger interface

| Trigger | Job | Schedule | Job parameters | Source |
|---|---|---|---|---|
| `dailySettlementTrigger` | `dailySettlementQuartzJob` → Spring Batch `dailySettlementJob` | cron `0 0 2 * * ?`, `Asia/Seoul` | `jungsanIlja` (string, `LocalDate.now().minusDays(1).toString()`), `ts` (long, uniqueness) | src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java, src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java |
| `orderEventRelayTrigger` | `orderEventRelayJob` (`QuartzJobBean`) | every 10 minutes, repeat forever | none | src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java, src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |
| *(none)* | `CancelReconciler.reconcileCancellations()` | **not registered** | — | src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java |

The third row is the notable one. `CancelReconciler` is annotated `@Component`, so Spring instantiates it on every startup; it simply is never invoked. Its javadoc names the missing piece: "TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤" — "TODO: Quartz schedule registration needed; to be added to QuartzConfig after confirming with 박성민" → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java. The matching Jira ticket SF-4512, "CancelReconciler Quartz 스케줄 등록", was created on the same date and is still `To Do` with no assignee → sources/context/sprints/tickets_2026-S17.csv

Both registered triggers are declared `storeDurably()` against a clustered JDBC job store, which means the schedule itself lives in MySQL rather than in the process. Changing `QuartzConfig` therefore changes what is written to the store at startup, not what an operator can see through an API — there is no API that lists triggers → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java, src/main/resources/application.yml

### 2. Job and step contract

`dailySettlementJob` is the only Spring Batch job in the repo. Its shape is fixed at configuration time and is the real contract a caller — that is, a trigger — is bound by → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java:

| Element | Name | Type | Contract |
|---|---|---|---|
| Job | `dailySettlementJob` (`JOB_NAME` constant) | `Job` | `start(settlementStep).next(markSettledStep)` — strictly sequential, no conditional flow, no listener |
| Step 1 | `settlementStep` | chunk-oriented, `CHUNK_SIZE = 500` | `JdbcCursorItemReader<SettlementTarget>` → `SettlementItemProcessor` → `SettlementItemWriter` |
| Reader | `settlementTargetReader` | `@StepScope` | Binds `@Value("#{jobParameters['jungsanIlja']}")` into one `?` via `preparedStatementSetter` |
| Processor | `SettlementItemProcessor` | `ItemProcessor<SettlementTarget, SettlementTarget>` | Sets `SUSURYO = gross × 0.12` (HALF_UP) and `JUNGSAN_AMT = gross − SUSURYO` in place |
| Writer | `SettlementItemWriter` | `ItemWriter<SettlementTarget>` | One `INSERT INTO SETTLEMENT_DTL` per item, all tagged with a `RUN_ID` resolved once per chunk |
| Step 2 | `markSettledStep` | tasklet | `MarkSettledTasklet`, returns `RepeatStatus.FINISHED` after one `UPDATE ORDER_MST` |

Four properties of that contract are worth naming because nothing in the code documents them:

- **Only one job parameter is read.** `jungsanIlja` is the whole input surface. `ts` exists purely so Spring Batch treats each launch as a new job instance rather than rejecting a re-run for the same date as a duplicate — which makes re-running easy and makes double-settling easy in the same stroke → src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java
- **The `RUN_ID` is fetched, never created.** `SettlementItemWriter.currentRunId()` runs `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'` once per `write()` call. No code in any of the five repos inserts that row (verified by grep for `INSERT INTO SETTLEMENT_RUN`, zero hits), so the job's correctness depends on an actor outside the repo → src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- **Step 2 resolves a different run than step 1.** `MarkSettledTasklet` uses a bare `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN`, with no `SANGTAE='RUNNING'` filter. If any newer non-RUNNING run row exists, the two steps operate on different `RUN_ID`s → src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- **There is no failure contract.** No `faultTolerant()`, no skip policy, no retry, no `StepExecutionListener` and no rollback compensation. The writer's javadoc explains why that is dangerous rather than harmless: "지급 요청이 전송되면 되돌릴 수 없다. 은행 이체는 익영업일에 실행된다. 정정이 필요한 경우 차월 정산에서 조정한다." — "once the payment request is sent it cannot be undone; the bank transfer executes the next business day; if a correction is needed it is adjusted in the following month's settlement" → src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java

### 3. Queue interface — the two tables treated as queues

There is no message broker in any of the five repos — no Kafka, RabbitMQ, SQS or Celery dependency → sources/infra/settlement/queues.md. Both "queues" in this system are ordinary MySQL tables polled by Quartz. This is the only asynchronous interface `settlement-batch` has, in either direction.

| | `ORDER_EVENT_OUTBOX` (inbound) | `CANCEL_RECON_QUEUE` (outbound) |
|---|---|---|
| Owner | 주문팀 / [[SYS-ORDER]] | settlement |
| Producer | `order-service` `OrderEventPublisher.publishOrderCancelled` | `settlement-batch` `OrderEventRelayJob` |
| Consumer | `settlement-batch` `OrderEventRelayJob` | **undefined** — registry says `consumer: TODO   # 확인 필요` |
| Poll SQL | `SELECT EVENT_ID, ORD_NO, PAYLOAD FROM ORDER_EVENT_OUTBOX WHERE PUBLISHED_YN='N' AND EVENT_TYPE=? ORDER BY REG_DTM LIMIT 500` | `SELECT SEQ, ORD_NO, SAYU_CD, RECV_DTM FROM CANCEL_RECON_QUEUE WHERE STATUS='PENDING' ORDER BY RECV_DTM` — issued only from the unscheduled `CancelReconciler` |
| Ack | `UPDATE ORDER_EVENT_OUTBOX SET PUBLISHED_YN='Y' WHERE EVENT_ID=?` | `UPDATE CANCEL_RECON_QUEUE SET STATUS='PROCESSED', PROCESSED_DTM=NOW() WHERE SEQ=?` — never executed |
| Throughput ceiling | 500 rows per 10 minutes ≈ 3,000 events/hour | n/a |
| Source | src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, sources/context/registry/services.yaml | src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sources/context/registry/services.yaml |

Two semantics to internalise before integrating:

- `PUBLISHED_YN='Y'` means **"the relay looked at this row"**, not "settlement acted on it". The relay marks every polled row published, including rows whose `SETTLEMENT_DTL` count was zero and which therefore produced no queue entry at all → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, sources/infra/settlement/queues.md
- `CANCEL_RECON_QUEUE` was designed as a **manual** queue, not an automated one. V1's comment: "SF-2287 대응. 정산 후 취소 건을 수기 정정용으로 적재한다. 정산팀이 주기적으로 확인하여 차월 정산에서 차감한다." — "added for SF-2287; post-settlement cancellations are queued for manual correction; the settlement team checks periodically and deducts in the next month's settlement" → src/main/resources/db/migration/V1__settlement_schema.sql. `CancelReconciler` would automate the drain if it were scheduled. It is not, and the queue has never been drained by either route.

### 4. Deploy interface — `workflow_dispatch`

The only entry point a human can operate deliberately is a GitHub Actions run. All three workflows are byte-for-byte identical apart from `ENVIRONMENT` → .github/workflows/settlement-batch-dev-cd.yml, .github/workflows/settlement-batch-stage-cd.yml, .github/workflows/settlement-batch-prod-cd.yml:

| Workflow | Trigger | `ENVIRONMENT` | Steps |
|---|---|---|---|
| `settlement-batch-dev-cd` | `workflow_dispatch` | `dev` | `actions/checkout@v3` → `./gradlew clean build -x test` → `./deploy.sh $ENVIRONMENT` |
| `settlement-batch-stage-cd` | `workflow_dispatch` | `stage` | identical |
| `settlement-batch-prod-cd` | `workflow_dispatch` | `prod` | identical |

```yaml
name: settlement-batch-prod-cd
on:
  workflow_dispatch:
env:
  SERVICE: settlement-batch
  ENVIRONMENT: prod
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: ./gradlew clean build -x test
      - run: ./deploy.sh $ENVIRONMENT
```
→ .github/workflows/settlement-batch-prod-cd.yml

Four things follow, and each is a constraint on anyone operating this service:

- **`workflow_dispatch` is not a job trigger.** Running the prod workflow redeploys the application; it does not launch `dailySettlementJob`. With `spring.batch.job.enabled: false`, a freshly deployed instance settles nothing until 02:00 KST → src/main/resources/application.yml
- **No workflow input exists.** There is no `inputs:` block, so an operator cannot pass a `jungsanIlja` — which is the concrete reason the "manual settlement run" question sits in `known_unknowns` rather than being answered here.
- **Production ships untested.** `-x test` skips the suite in every environment. Given that the suite is one test asserting the rounding mode the job does not use, little is lost — but the pipeline would not catch a regression either → .github/workflows/settlement-batch-prod-cd.yml
- **The last step is outside version control.** `deploy.sh` is not in the repo and neither is a Dockerfile, so how the built jar becomes a running, clustered pair of Quartz nodes cannot be read here → sources/infra/settlement/runtime.md

### Worked Example — a trigger firing, not a request

Every other API- artifact in this reef carries a worked HTTP request and response. This one cannot: there is no listener, no port and no route to send a request to, and the extraction records the same thing as `"api": null` with the reason "settlement-batch exposes no HTTP surface" → sources/apis/settlement/batch/openapi.meta.json. The equivalent worked example is a Quartz trigger firing and the job parameters it hands over.

**The trigger definition, verbatim** → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java:

```java
@Bean
public Trigger dailySettlementTrigger() {
    return TriggerBuilder.newTrigger()
            .forJob(dailySettlementJobDetail())
            .withIdentity("dailySettlementTrigger")
            .withSchedule(CronScheduleBuilder
                    .cronSchedule("0 0 2 * * ?")
                    .inTimeZone(java.util.TimeZone.getTimeZone("Asia/Seoul")))
            .build();
}
```

The nightly firing at 02:00 KST on 2026-09-18 invokes `DailySettlementQuartzJob`, which builds its parameters like this → src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java:

```java
JobParameters params = new JobParametersBuilder()
        .addString("jungsanIlja", LocalDate.now().minusDays(1).toString())
        .addLong("ts", System.currentTimeMillis())
        .toJobParameters();
jobLauncher.run(dailySettlementJob, params);
```

which for that firing is:

```
jungsanIlja = "2026-09-17"          // LocalDate.now().minusDays(1).toString()
ts          = <System.currentTimeMillis() at the moment of firing>
```

`jungsanIlja` is the only parameter the job reads. `ts` exists to make each Spring Batch job instance unique, so a re-run for the same date is not rejected as a duplicate instance. Both are strings and longs in a `JobParameters` map, not a request body — there is no JSON on the wire anywhere in this flow.

The parameter is bound into the reader by `@Value("#{jobParameters['jungsanIlja']}")` and lands in one `?` of the step's SQL → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java:

```sql
SELECT m.ORD_NO, d.PARTNER_ID, d.SANGPUM_CD, d.SURYANG, d.DANGA
  FROM ORDER_MST m
  JOIN ORDER_DTL d ON d.ORD_NO = m.ORD_NO
 WHERE m.SANGTAE_CD = 'BAESONG_WANRYO'
   AND DATE(m.UPD_DTM) = '2026-09-17'
```

Two things about that predicate are worth stating. It selects on delivery completion alone — the javadoc is explicit that "주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다. 배송이 완료되었다면 파트너는 이행을 마친 것으로 본다." ("the order's current status and whether it was cancelled are not part of the condition; if delivery completed, the partner is treated as having fulfilled"). And the literal format matters: `'2026-09-17'` matches, `'20260917'` — the shape `DateUtil` produces — does not.

The second step, `markSettledStep`, then runs `MarkSettledTasklet`, which issues the cross-boundary update described below → src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java:

```sql
UPDATE ORDER_MST SET SANGTAE_CD='JUNGSAN_WANRYO', UPD_DTM=NOW()
 WHERE ORD_NO IN (SELECT ORD_NO FROM SETTLEMENT_DTL
                   WHERE RUN_ID = (SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN))
```

### Worked Example — the relay, and the SQL it actually issues

`orderEventRelayTrigger` is simpler: it fires every ten minutes with no job parameters at all, and the job derives everything from the outbox query → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java. One firing at 2026-09-18 14:20 KST performs, in order → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java:

**Step 1 — poll the outbox** (bound parameter is the constant `EVT_ORDER_CANCELLED = "order.cancelled"`):

```sql
SELECT EVENT_ID, ORD_NO, PAYLOAD FROM ORDER_EVENT_OUTBOX
 WHERE PUBLISHED_YN = 'N' AND EVENT_TYPE = 'order.cancelled'
 ORDER BY REG_DTM LIMIT 500
```

**Step 2 — for each row, the "already settled?" test:**

```sql
SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = 'ORD20260817001'
```

**Step 3 — only when that count is > 0, enqueue a correction:**

```sql
INSERT INTO CANCEL_RECON_QUEUE (ORD_NO, SAYU_CD, STATUS)
VALUES ('ORD20260817001', '03', 'PENDING')
```

**Step 4 — unconditionally, whether or not step 3 ran, acknowledge the event:**

```sql
UPDATE ORDER_EVENT_OUTBOX SET PUBLISHED_YN='Y' WHERE EVENT_ID=91824
```

That is the entire relay contract. The `'03'` in step 3 is not read from a column: it is cut out of the JSON payload by fixed offset, defaulting to `"00"` when the fragment is absent:

```java
int i = s.indexOf("\"sayuCd\":\"");
return i < 0 ? "00" : s.substring(i + 10, i + 12);
```
→ src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java

So the payload contract is an implicit one: a JSON object with a two-character `sayuCd`, parsed without a JSON library. The `"00"` fallback is not one of the four documented reason codes — 01 파트너 귀책 (partner fault), 02 시스템 오류 (system error), 03 고객 변심 (customer change of mind), 04 배송 실패 (delivery failure) → sources/context/business-rules.md (취소정책 sheet)

### Table interface — reads

| Table | Owner | Access | Source |
|---|---|---|---|
| `ORDER_MST` | 주문팀 / [[SYS-ORDER]] | `SELECT` joined with `ORDER_DTL`, filtered `SANGTAE_CD='BAESONG_WANRYO' AND DATE(UPD_DTM)=?` | src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java |
| `ORDER_DTL` | 주문팀 | `SELECT PARTNER_ID, SANGPUM_CD, SURYANG, DANGA` | src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java |
| `ORDER_EVENT_OUTBOX` | 주문팀 | `SELECT ... WHERE PUBLISHED_YN='N' AND EVENT_TYPE='order.cancelled' ORDER BY REG_DTM LIMIT 500` | src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |
| `SETTLEMENT_DTL` | settlement | `SELECT COUNT(1) ... WHERE ORD_NO = ?` as the "already settled" test | src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |
| `SETTLEMENT_RUN` | settlement | `SELECT MAX(RUN_ID) ... WHERE SANGTAE='RUNNING'` (writer) and bare `SELECT MAX(RUN_ID)` (tasklet) | src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java |
| `CANCEL_RECON_QUEUE` | settlement | `SELECT ... WHERE STATUS='PENDING' ORDER BY RECV_DTM` — only from the unscheduled reconciler | src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java |
| `PARTNER_CONTRACT` | settlement | `SELECT PARTNER_ID, FEE_RATE, SETTLE_CYCLE ... WHERE PARTNER_ID = ?` — repository exists, never called | src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java |

### Table interface — writes

| Table | Owner | Access | Source |
|---|---|---|---|
| `SETTLEMENT_DTL` | settlement | `INSERT` one row per settled order | src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java |
| `ORDER_MST` | **주문팀, written cross-boundary** | `UPDATE ... SET SANGTAE_CD='JUNGSAN_WANRYO', UPD_DTM=NOW()` | src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java |
| `ORDER_EVENT_OUTBOX` | **주문팀, written cross-boundary** | `UPDATE ... SET PUBLISHED_YN='Y' WHERE EVENT_ID=?` | src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |
| `CANCEL_RECON_QUEUE` | settlement | `INSERT (ORD_NO, SAYU_CD, STATUS) VALUES (?, ?, 'PENDING')` | src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java |
| `SETTLEMENT_ADJUSTMENT` | undefined | `INSERT (ORD_NO, SAYU_CD, ADJ_TYPE)` — only from the unscheduled reconciler, and no migration anywhere creates the table | src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java |

The cross-boundary writes are sanctioned by an old decision recorded in the schema header: "ORDER_MST 는 주문팀 소유이나 정산 배치가 SANGTAE_CD 를 갱신한다." — "ORDER_MST belongs to the order team, but the settlement batch updates SANGTAE_CD" → src/main/resources/db/migration/V1__settlement_schema.sql. The tasklet repeats the same justification: "통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)" — "under the consolidated-database policy the settlement batch updates it directly (agreed 2019)" → src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

## How Agents Should Use This

**Do not look for an endpoint.** If a task says "call settlement" or "trigger a settlement run via the API", the correct answer is that no such call exists, and the verification table above is the evidence. Do not invent a port, a path or a client; `settlement-batch` does not listen.

**To make settlement do something, write a row or change a trigger.** There are exactly three levers, in ascending order of blast radius:

1. Insert into `ORDER_EVENT_OUTBOX` with `EVENT_TYPE='order.cancelled'` and `PUBLISHED_YN='N'` — the relay picks it up within ten minutes. This is the supported inbound path, and it is how `order-service` already talks to settlement.
2. Register a `JobDetail` and `Trigger` in `QuartzConfig` — this is what SF-4512 asks for, and it is the change that would finally drain `CANCEL_RECON_QUEUE`. Treat it as a money-moving change, not a scheduling change: it would generate `SETTLEMENT_ADJUSTMENT` rows for 4,127 queued items totalling 188,851,520 KRW, into a table no migration creates → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv, src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
3. Run a CD workflow — this redeploys and does not run the job.

**When reasoning about timing, use the trigger, not `DateUtil`.** The job's notion of "yesterday" is `LocalDate.now().minusDays(1)` in the JVM's default zone, formatted ISO. `DateUtil.settlementBaseDate()` looks like the authority and is dead code. Quoting `DateUtil` as the batch's date rule is the single most likely mistake to make in this repository.

**When asked what runs in production, separate three claims.** (a) `QuartzConfig` registers two triggers — verified from code. (b) The registry says `schedule: 매일 02:00 KST` — a document, consistent with the cron. (c) What is actually deployed — unverifiable from here, because `deploy.sh` and any prod profile are outside the repo. Keep (c) in `known_unknowns` rather than asserting it.

**Do not describe `CANCEL_RECON_QUEUE` as having a consumer.** Code, the infra extraction and the service registry independently say it does not: `consumer: TODO   # 확인 필요` → sources/context/registry/services.yaml. The full consequence is analysed in [[RISK-SETTLEMENT-RECON-BACKLOG]]; the interface-level statement is simply that this queue has a producer and no drain.

## Related

- [[SYS-SETTLEMENT]] — the service this surface belongs to
- [[SCH-SETTLEMENT-BATCH]] — definitions of every table listed above
- [[PROC-SETTLEMENT-DAILY-BATCH]] — what happens after `dailySettlementTrigger` fires
- [[SYS-ORDER]] — owner of the three tables accessed cross-boundary
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what the unregistered third trigger costs, measured
