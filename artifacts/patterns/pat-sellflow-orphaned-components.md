---
id: "PAT-SELLFLOW-ORPHANED-COMPONENTS"
type: "pattern"
title: "Orphaned Components — Code and Schema With No Caller, Writer or Scheduler"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Every orphan below was re-verified on 2026-09-19 by grepping the identifier across all five repository roots and counting the hits that are not its own declaration. The method is reproducible and the commands are recorded in the Where It Appears table. An absence finding is only valid until the next commit adds a caller, so re-run the greps before relying on any single row; the pattern as a whole is durable because the oldest instance has survived 41 months. Re-verified on 2026-09-19 after correction passes in order-service and inventory-api: the rows that recorded *missing DDL* — ORDER_DELIVERY, ORDER_STATUS_HIST, RESTORE_LOG and the seven phantom order columns — are gone, because the DDL now exists. Every one of those objects is still an orphan, just a better-formed one: it has a schema and no caller."
freshness_triggers:
  - "delivery-bff/src/index.ts"
  - "inventory-api/alembic/versions/*.py"
  - "inventory-api/app/main.py"
  - "order-service/.github/workflows/ci.yml"
  - "order-service/src/main/resources/db/migration/*.sql"
  - "settlement-anomaly/app/main.py"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "settlement-batch/src/main/resources/db/migration/*.sql"
  - "sources/infra/settlement/queues.md"
known_unknowns:
  - "Whether any of these components is invoked from outside version control — a DBA script, a cron entry on the batch host, an internal tool, or a manual curl. Nothing in the five repositories calls them, which is a statement about the repositories and not about the production hosts."
  - "Who creates SETTLEMENT_RUN rows. SettlementItemWriter and settlement-anomaly both read the table and no repository ever INSERTs into it, so either an unversioned script writes it or the daily batch has never produced a row through this code path."
  - "Whether SETTLEMENT_ADJUSTMENT exists in the production database at all. No migration in any repo creates it; the 2025-03 handover reports it could not be queried; nothing since then has revisited the question."
  - "Whether any of the fully-formed orphans hold rows. RESTORE_LOG, ORDER_DELIVERY and ORDER_STATUS_HIST all have DDL and no writer in any of the five repos; whether something outside these repos populates them cannot be established from source."
  - "Whether order-service and settlement-batch have a checked-in Gradle wrapper. Every CI step invokes ./gradlew and no such file is present in either repository tree; this may be a fixture omission rather than a real gap."
  - "Whether model/artifacts/iforest_v3.pkl exists on the settlement-anomaly host. It is loaded at import time and is absent from the repository, so either the deployment supplies it or the service does not start."
tags:
  - absence-of-evidence
  - dead-code
  - orphan
  - scheduling
  - schema-drift
aliases:
  - "고아 컴포넌트"
  - "dead code"
  - "unscheduled jobs"
  - "tables with no writer"
relates_to:
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-DB-AS-QUEUE]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-RUNTIME]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "refines"
    target: "[[RISK-ORDER-DISABLED-TESTS]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-ORDER-MIGRATION-HISTORY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "The only mounted route; nothing imports orderClient"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "No include_router call anywhere in the app"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/ci.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/migration-issue.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatusHistory.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The cancel path, which calls neither client"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V21__add_settlement_ref_index.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V24__add_order_memo_search_index.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Two JobDetail beans, two Trigger beans, and no third"
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
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "The one place a human recorded an orphan finding at the time"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
notes: "This artifact asserts absences. Each row of the table names the grep that produced it so a reader can re-derive the finding rather than trust it."
---

## Overview

Sellflow's five repositories contain a large amount of code and schema that exists, compiles, would work if invoked, and is never invoked. The pattern is not isolated dead code — every codebase has some — but a specific and repeated failure mode: something is built, its registration step is left for a second pass, the second pass never happens, and nothing in the estate notices. There is no runtime that complains about an unregistered `@Component`, no broker that reports a topic without a subscriber, no CI that fails on an unreferenced column, and no review gate that asks whether the thing just written is reachable.

The consequence is that absence has to be established by search rather than observed. A missing consumer looks exactly like a working one from every vantage point a person in this organisation has: the class is there, its method is correct, its log lines are written, the registry lists the queue, and the table fills up. Only a grep for callers distinguishes the two.

That is why this artifact is structured around commands rather than assertions. Thirty-odd orphans are recorded below; each names the search that produced it. The most expensive of them — `CancelReconciler` — has been unreachable since 2023-04-24 and is the direct mechanism behind the 188,851,520 KRW recorded in [[RISK-SETTLEMENT-RECON-BACKLOG]].

## Key Facts

- **The oldest orphan is 41 months old and documented itself.** `CancelReconciler` is a `@Component` whose own javadoc reads `TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤` — "TODO: Quartz schedule registration needed (to be added to QuartzConfig after checking with 박성민)". `grep -rn CancelReconciler` across all five repos returns two lines, both inside that file: the class declaration and its own logger → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- **`QuartzConfig` is the estate's only registration point, and it declares exactly two jobs.** Four `@Bean` methods — `dailySettlementJobDetail`, `dailySettlementTrigger`, `orderEventRelayJobDetail`, `orderEventRelayTrigger`. Anything in settlement-batch that is not one of these two jobs, or a step inside `dailySettlementJob`, has no path to execution → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- **A second unscheduled component sits beside the first.** `SettlementReportWriter.write(runId)` is a `@Component` no step calls; `grep -rn SettlementReportWriter` returns two lines, both its own. Its body only logs, though its class comment claims `파트너 포털이 이 파일을 읽어간다` — "the partner portal reads this file" — and it carries `TODO(도윤) 2024-11-02: 취소 정정분은 이 리포트에 포함되지 않는다` ("cancel corrections are not included in this report") → settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java
- **A whole table exists with no reader and no writer.** `SETTLEMENT_RUN_LOG` is created by `V2__add_settlement_run_log.sql` with five columns and indexed by `V6__settlement_run_log_index.sql`. `grep -rn SETTLEMENT_RUN_LOG` returns exactly those two migration lines and nothing in any Java, Python or TypeScript file. It was given an index before it was given a caller → settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql, settlement-batch:src/main/resources/db/migration/V6__settlement_run_log_index.sql
- **A table that is read but never written.** `SETTLEMENT_RUN` is created in V1 and read twice — `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'` in `SettlementItemWriter.currentRunId()` and a `JOIN SETTLEMENT_RUN r` in settlement-anomaly. `grep -rn "INSERT INTO SETTLEMENT_RUN"` across all five repos returns zero lines. Every settlement detail row is written against a `RUN_ID` whose origin is not in version control → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, settlement-anomaly:app/main.py
- **A table that is written but never created.** `SETTLEMENT_ADJUSTMENT` appears once in the entire estate, as the `INSERT` target inside the unscheduled `CancelReconciler`. No migration in any repository creates it. The 2025-03 handover recorded the same finding from the other direction: `SETTLEMENT_ADJUSTMENT 테이블이 문서에는 나오는데 실제로 조회가 안 됨. WIP` — "the SETTLEMENT_ADJUSTMENT table appears in the documents but cannot actually be queried" → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sellflow-docs:context/handover/2025-03_정산팀_인수인계.md
- **Order-service's cancellation path calls neither of its two outbound clients.** `OrderCancelService.cancel` writes `ORDER_CANCEL`, updates `ORDER_MST` and publishes the outbox row. `grep -rn InventoryClient` and `grep -rn PaymentClient` each return only their own declarations. The restock call and the PG cancellation call both exist as code and neither is ever made → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java, order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java, order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java
- **`InventoryClient` is doubly orphaned: it has no caller, and its target endpoint does not exist either.** It posts to `/inventory/restore`. inventory-api's mounted surface is `POST /stock/restock` and `GET /stock/{sku}`; the `/inventory` prefix lives in `app/routers.py`, whose `APIRouter` is never mounted — `grep -rn include_router` across inventory-api returns zero lines → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java, inventory-api:app/main.py, inventory-api:app/routers.py
- **An entire FastAPI router is unmounted, and only its test knows it exists.** `inventory-api/app/routers.py` declares `APIRouter(prefix="/inventory")` with `/stock/{sku_cd}` and `/health`. `main.py` never imports it. The only reference in the repository is `from app.routers import health` inside `tests/test_health.py`, so the health check that is tested is not the health check that is served → inventory-api:app/routers.py, inventory-api:tests/test_health.py
- **Two utility classes in order-service have zero callers, and one of them says the opposite.** `grep -rn "DateUtil\."` and `grep -rn "MoneyUtil\."` across all five repos return one line between them (a settlement-batch test). Order-service's `DateUtil` javadoc reads `2019년 초기 구축분. 여러 곳에서 참조하고 있어 손대지 않는 것을 권장` — "built in 2019; referenced in many places, so it is recommended not to touch it" — against zero references → order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java, order-service:src/main/java/kr/co/sellflow/order/common/MoneyUtil.java
- **The rounding rule everyone cites lives in an unused method.** settlement-batch's `MoneyUtil.fee` uses `RoundingMode.FLOOR` and its comment records the 2022 agreement that settlement floors while order rounds. Its only caller is `SettlementItemProcessorTest`. The production processor, `SettlementItemProcessor`, inlines `new BigDecimal("0.12")` with `RoundingMode.HALF_UP` and never touches `MoneyUtil`. The documented convention is therefore enforced only in a test → settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- **`PARTNER_CONTRACT` has a table, a domain class, a repository and no reader.** V3 creates the table; `PartnerContract` and `PartnerContractRepository.find` exist; `grep -rn PartnerContractRepository` returns only its own two lines. `SettlementItemProcessor`'s comment states the consequence plainly: `파트너별 계약 수수료율은 PARTNER_CONTRACT 에 있으나, 현재는 기본 수수료율만 적용한다. (2021년 이후 미정비)` — "per-partner contract fee rates are in PARTNER_CONTRACT, but currently only the default rate is applied (not maintained since 2021)" → settlement-batch:src/main/resources/db/migration/V3__add_partner_contract.sql, settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- **Columns are added for a workflow that is never built, and the migration says so at the time.** `V5__add_recon_processed_columns.sql` adds `PROCESSED_AT` and `PROCESSED_BY` to `CANCEL_RECON_QUEUE` under the comment `정정 처리 결과를 남기기 위한 컬럼. 아직 쓰는 코드는 없다` — "columns for recording the correction result; no code uses them yet". Both greps return only that migration. They also duplicate `PROCESSED_DTM`, which V1 already created and which only the unscheduled reconciler writes → settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql, settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
- **The anomaly review workflow exists only as two columns.** `SETTLEMENT_ANOMALY.REVIEWED_BY` and `REVIEWED_DTM` appear once each, in the CREATE TABLE. The service's only `INSERT` writes `ORD_NO, ANOMALY_CD, SCORE, DETECTED_DTM, STATUS` with `STATUS` hardcoded to `'DETECTED'`; there is no `UPDATE` against the table anywhere. The README states the reason without calling it a gap: `후속 조치 프로세스는 본 서비스 범위 밖이다` — "the follow-up process is outside this service's scope" → settlement-anomaly:sql/V1__anomaly_schema.sql, settlement-anomaly:app/main.py, settlement-anomaly:README.md
- **The detection endpoint has no identified caller.** `POST /detect` is the service's only business route; its README claims `정산 배치 종료 후 (매일 03:00) 트리거된다` — "triggered after the settlement batch finishes, daily at 03:00". settlement-batch contains no HTTP client, no reference to port 8090 and no reference to the service by name; no repository calls it → settlement-anomaly:app/main.py, settlement-anomaly:README.md, settlement-batch:build.gradle
- **`RESTORE_LOG` is complete and untouched.** The 2023-04-14 revision creates it with `LOG_SEQ`, `ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD` and `REG_DTM` plus an index on `ORD_NO`, and the 2024-09-02 follow-up indexes `SAYU_CD`. `grep -rn RESTORE_LOG` across the five repos returns only those two revisions and an unused dataclass — no INSERT, no SELECT → inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py, inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py, inventory-api:app/main.py
- **Order-service's migrations now index real columns that nothing queries.** V21 indexes `ORDER_MST(JUNGSAN_RUN_ID)`, a column no code in any of the five repos writes or reads; V24 prefix-indexes `ORDER_MST(GOGAEK_MEMO(64))`, on which no query filters; V22 leads on `CHORI_SANGTAE`, whose only written value is the literal `"COMPLETED"`. Three indexes, no queries → order-service:src/main/resources/db/migration/V21__add_settlement_ref_index.sql, order-service:src/main/resources/db/migration/V24__add_order_memo_search_index.sql, order-service:src/main/resources/db/migration/V22__order_cancel_status_index.sql
- **Two order-side names are still referenced and created by nothing**, and one is in production code: `ORD_DT` in `OrderSearchService`'s query, and `BAESONG_MSG` in V13's deferral TODO. settlement-batch repeats the shape: V4 indexes `CANCEL_RECON_QUEUE (STATUS, REG_DT)` where the column is `RECV_DTM` → order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java, order-service:src/main/resources/db/migration/V13__cleanup_unused.sql, settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql
- **Two JPA entities map tables that no migration creates.** `DeliveryInfo` maps `ORDER_DELIVERY` and `OrderStatusHistory` maps `ORDER_STATUS_HIST`; neither name appears in any `.sql` file in any repo. Both repositories — `DeliveryInfoRepository`, `OrderStatusHistoryRepository` — also have zero callers, so the mismatch has never been exercised → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatusHistory.java, order-service:src/main/resources/db/migration/V1__init.sql
- **The whole order-service test suite has no runner.** `ci.yml` builds with `./gradlew clean build -x test` and the test step is commented out with `TODO 테스트 켜야 함. OrderCancelServiceTest 가 로컬 DB 를 타서 CI 에서 깨짐 - 2023-05-11 박성민` ("TODO: need to turn tests on; OrderCancelServiceTest hits a local DB and breaks CI"). All three settlement-batch deploy workflows also pass `-x test`. Five test classes across the estate are therefore unreachable in the same sense as the unscheduled components → order-service:.github/workflows/ci.yml, settlement-batch:.github/workflows/settlement-batch-prod-cd.yml
- **A CI workflow invokes a Gradle task that is not declared.** `migration-issue.yml` runs `./gradlew flywayInfo`; order-service's `build.gradle` declares `org.flywaydb:flyway-core` as a runtime library and does not apply the Flyway Gradle plugin, which is what would define that task → order-service:.github/workflows/migration-issue.yml, order-service:build.gradle
- **The generated order client in delivery-bff has no importer.** `src/generated/orderApi.ts` exports `cancelOrder`; `src/orderClient.ts` wraps it as `requestCancel`; `src/index.ts` mounts only `GET /delivery/:ordNo` and imports only `syncDeliveryStatus` and `logger`. `grep -rn "orderClient"` and `grep -rn requestCancel` return one line — the export itself → delivery-bff:src/index.ts, delivery-bff:src/orderClient.ts, delivery-bff:src/generated/orderApi.ts
- **Configuration values are declared and read by nothing.** `settlement-batch/.env.dev` sets `QUARTZ_ENABLED=true`, which appears nowhere else in any repo; it also sets `DB_URL` pointing at a separate `settlement` database while `application.yml` hardcodes `sellflow_order` and reads only `DB_HOST` and `DB_USER`. inventory-api's `app/config.py` defines `RESTOCKABLE_REASONS` that only the tests import, because `main.py` redefines the same set inline → settlement-batch:.env.dev, settlement-batch:src/main/resources/application.yml, inventory-api:app/config.py, inventory-api:app/main.py
- **The service registry records two of these gaps as open questions and has not been revisited.** Its `queues:` block carries `consumer: TODO   # 확인 필요` ("needs checking") for `CANCEL_RECON_QUEUE`, and settlement-anomaly's entry reads `이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO` — "performs detection only; the remediation owner is not defined". The file's own header says `last_reviewed: 2026-03-02   # 이후 갱신 없음` ("no updates since") → sellflow-docs:context/registry/services.yaml

## Where It Appears

Each row names the search that establishes the absence. "Own lines only" means every hit was inside the component's own declaration.

### Components with no caller or scheduler

| Component | Repo / path | What is missing | Evidence of the absence |
|---|---|---|---|
| `CancelReconciler` | settlement-batch `recon/CancelReconciler.java` | Quartz trigger; any caller | `grep -rn CancelReconciler` → own lines only. `QuartzConfig` declares 2 jobs, neither is this. Unreachable since 2023-04-24 (41 months) |
| `SettlementReportWriter` | settlement-batch `report/SettlementReportWriter.java` | Any step or job calling `write()` | `grep -rn SettlementReportWriter` → own lines only. Not a step in `DailySettlementJobConfig` |
| `PartnerContractRepository` / `PartnerContract` | settlement-batch `repository/`, `domain/` | Any caller; the fee rate is hardcoded instead | `grep -rn PartnerContractRepository` → own lines only. Unmaintained since 2021 per its own comment |
| `MoneyUtil` (settlement) | settlement-batch `common/MoneyUtil.java` | A production caller | `grep -rn "MoneyUtil\."` → one hit, `SettlementItemProcessorTest` |
| `MoneyUtil` (order) | order-service `common/MoneyUtil.java` | Any caller at all | `grep -rn "MoneyUtil\."` → zero hits in order-service |
| `DateUtil` (order) | order-service `common/DateUtil.java` | Any caller; javadoc claims many | `grep -rn "DateUtil\."` → zero hits. Comment dates it to 2019 |
| `DateUtil.settlementBaseDate` (settlement) | settlement-batch `common/DateUtil.java` | A caller; the job inlines its own date | `grep -rn "DateUtil\."` → zero hits. `DailySettlementQuartzJob` uses `LocalDate.now().minusDays(1)` |
| `InventoryClient` | order-service `client/InventoryClient.java` | A caller, and a matching endpoint | `grep -rn InventoryClient` → own lines only. Posts to `/inventory/restore`, which is not mounted |
| `PaymentClient` | order-service `payment/PaymentClient.java` | A caller | `grep -rn PaymentClient` → own lines only. Cancellation never calls PG |
| `OrderMapper` | order-service `mapper/OrderMapper.java` | A caller | `grep -rn "OrderMapper\."` → zero hits |
| `OrderCancelServiceV1` | order-service `legacy/` | Any caller; only a `@Disabled` test references it | `grep -rn OrderCancelServiceV1` → the class and its disabled test. Kept because "배치에서 참조 가능성" (the batch might reference it); no repo does |
| `DeliveryInfoRepository`, `OrderStatusHistoryRepository` | order-service `repository/` | Any caller | `grep -rn` each → own declarations only |
| `app/routers.py` router | inventory-api | `app.include_router(router)` | `grep -rn include_router` → zero hits in the repo. Only `tests/test_health.py` imports from it |
| `Stock`, `RestoreLog` dataclasses | inventory-api `app/models.py` | Any importer | `grep -rn "from app.models"` → zero hits |
| `AnomalyRow` | settlement-anomaly `app/schemas.py` | Any importer; `main.py` writes raw SQL | `grep -rn AnomalyRow` → own declaration only |
| `requestCancel` / `generated/orderApi.ts` | delivery-bff `src/` | An importer; `index.ts` mounts one route | `grep -rn requestCancel`, `grep -rn orderClient` → one line each |
| `POST /detect` | settlement-anomaly `app/main.py` | An identified caller or scheduler | No HTTP client, port 8090 reference or service name in any other repo |
| Whole test suite | order-service `src/test/`, settlement-batch `src/test/` | A CI step that runs it | `ci.yml` uses `-x test` with the test job commented out since 2023-05-11; all three settlement CD workflows use `-x test` |
| `flywayInfo` task | order-service `.github/workflows/migration-issue.yml` | The Flyway Gradle plugin | `build.gradle` plugins block has `spring-boot`, `dependency-management`, `java` only |

### Tables and columns with no writer, no reader, or no DDL

| Object | Repo / path | What is missing | Evidence of the absence |
|---|---|---|---|
| `SETTLEMENT_RUN_LOG` | settlement-batch V2, indexed by V6 | Both a reader and a writer | `grep -rn SETTLEMENT_RUN_LOG` → the two migration lines and nothing else |
| `SETTLEMENT_RUN` | settlement-batch V1 | A writer | `grep -rn "INSERT INTO SETTLEMENT_RUN"` → zero lines across all five repos; two readers exist |
| `SETTLEMENT_ADJUSTMENT` | referenced by `CancelReconciler` | Its DDL | `grep -rn SETTLEMENT_ADJUSTMENT` → one line, the INSERT. Handover 2025-03 reports it is not queryable |
| `RESTORE_LOG` | inventory-api alembic `3f9a`, `8ba1` | A reader and a writer | `grep -rn RESTORE_LOG` → the two revision files and the unused dataclass. `3f9a` creates six columns and an index on `ORD_NO`; `8ba1` indexes `SAYU_CD`; nothing inserts |
| `SETTLEMENT_ANOMALY.REVIEWED_BY`, `.REVIEWED_DTM` | settlement-anomaly `sql/V1__anomaly_schema.sql` | Any writer | `grep -rn REVIEWED_BY` → the DDL line only. No UPDATE exists against the table |
| `CANCEL_RECON_QUEUE.PROCESSED_AT`, `.PROCESSED_BY` | settlement-batch V5 | Any writer; V1 already had `PROCESSED_DTM` | `grep -rn PROCESSED_AT` → the migration line only. Migration comment admits it |
| `ORDER_DELIVERY`, `ORDER_STATUS_HIST` | order-service V1, mapped by `DeliveryInfo` and `OrderStatusHistory` | A reader and a writer | `grep -rn ORDER_DELIVERY` → the `CREATE TABLE` and one `@Table` annotation. Both repositories are declared and never injected |
| `ORD_DT` | order-service `OrderSearchService` | The migration that would create it | `grep -rn ORD_DT` → the query only; `JUMUN_ILSI` is the order-date column that exists |
| `BAESONG_MSG` | order-service V13's TODO, and the 2022 spec as `baesongMsg` | The column, in any migration | `grep -rn BAESONG_MSG` → the comment and the spec |
| `CANCEL_RECON_QUEUE.REG_DT` | settlement-batch V4 (index) | The column; it is `RECV_DTM` | `grep -rn REG_DT` in settlement-batch → V4 and an unrelated `REG_DTM` |
| `EXPECTED_AMT` | `sources/raw/exports/README.md` query | The column, in any migration | `grep -rn EXPECTED_AMT` across repos and sources → the README line only |
| `PARENT_ORD_NO`, `CHANGGO_CD`, `JUNGSAN_RUN_ID`, `OPT_AMT`, `CHNL_CD`, `GOGAEK_MEMO`, `CHAENNEL_CD`, `OKSYEON_MYEONG`, `UNSONGJANG_BEONHO` | order-service V2–V19 | Any code that reads or writes them | Each grep returns only its own migration. `DeliveryInfo`'s comment says as much for V10: `분할배송은 지원하지 않는다 (V10 에서 컬럼만 추가됨)` — "split delivery is not supported (V10 added only the column)" |
| `QUARTZ_ENABLED` | settlement-batch `.env.dev` | A reader | `grep -rn QUARTZ_ENABLED` → the `.env.dev` line only |

The counts: nineteen code components with no caller, twelve schema objects with no writer, no reader or no DDL. Three of the orphans — `CancelReconciler`, `SETTLEMENT_ADJUSTMENT` and the `CANCEL_RECON_QUEUE` processing columns — are links in a single chain, which is why the correction workflow has never run end to end.

Two of the rows above changed character rather than disappearing. `RESTORE_LOG`, `ORDER_DELIVERY` and `ORDER_STATUS_HIST` used to appear here as objects *without DDL*; each now has a complete definition and an entity or index that matches it, and each is still read and written by nothing. That is the purer form of the pattern: the half-built change was finished on the schema side and never collected on the code side.

## Design Intent

**Not a design at all, in most cases — a deferral that was never collected.**

The recoverable intent is consistent and benign. Each orphan was written as one half of a two-step change, and the second step was left to someone else or to later:

- `CancelReconciler` was written by 김도윤 on 2023-04-24 with the registration explicitly deferred pending 박성민's confirmation. The ticket conversation shows why it felt safe: 김도윤 estimated `월 10건 미만` ("fewer than 10 a month") and said `당분간은 수기로 충분합니다` ("manual handling is enough for now"). The reconciler was the automation that was not yet needed.
- `PARTNER_CONTRACT` and its repository were built for per-partner fee rates and then overtaken by the data not being maintained — `2021년 이후 미정비`. The code is correct; the table's contents are not trusted.
- `PROCESSED_AT`/`PROCESSED_BY` and `REVIEWED_BY`/`REVIEWED_DTM` are schema laid down ahead of a workflow, a normal and reasonable habit. Both migrations are honest that no code uses them yet.
- The schema-side orphans have a different and more interesting cause: `V23__consolidated_schema.sql` reserves a consolidated baseline for new environments and states it was never written — `아직 작성하지 않았다` — so nothing in the repository reconciles the migrations against any live database. An index or a column can therefore sit unused for years with no mechanism that would notice.

The intent that is **not** recoverable is why none of these was ever collected. No document, ticket or minute in this reef contains a review step that asks whether a new component is reachable, and no repository has a check that would fail. The 2025-07 incident review came closest — its open item `정산 관련 배치 전수 점검 (스케줄 등록 여부 포함)` ("full audit of settlement-related batches, including whether they are registered on a schedule") names exactly this pattern — and it remains unticked with no owner.

## Trade-offs

**What deferring registration buys.**

1. **A half-finished feature ships without risk to what runs.** An unregistered `@Component` cannot corrupt data, exhaust a connection pool or slow a batch. Leaving `CancelReconciler` in the tree was strictly safer than scheduling it untested against live payouts.
2. **The intent survives in reviewable form.** Three years later, the reconciler is still the clearest available statement of what the correction workflow was meant to do — better than the procedure documents, which describe the manual process rather than the automated one.
3. **Schema ahead of code avoids a second migration.** Adding `REVIEWED_BY` with the table costs nothing and saves an `ALTER` on a large table later.
4. **Deprecated code kept in place protects unknown callers.** `OrderCancelServiceV1` was retained because `배치에서 참조 가능성이 있어` ("the batch might reference it"). In an estate with four services inside one database and no dependency graph, that caution is rational even when it turns out to be unnecessary.

**What it costs.**

1. **An orphan is indistinguishable from a working component from every human vantage point.** The class exists, the registry lists the queue, the table fills. Three separate people asked whether the deduction was automatic — the 2025-03 handover (`실제로 차감이 자동으로 되는지는 확인해 본 적 없음`, "I have never checked whether the deduction actually happens automatically"), the 2026-06 kickoff, and 문지영's 2026-08 mail — and none of them could answer it, because answering it requires a grep for callers.
2. **The cost compounds silently and is only measurable in arrears.** The unscheduled reconciler produced no error, no alert and no log line for 41 months; the first quantification was a finance extraction in 2026-09 showing 4,127 rows and 188,851,520 KRW.
3. **Orphans mislead the next reader in the direction of confidence.** Order-service's `DateUtil` warns that it is widely referenced when it is referenced nowhere. `SettlementReportWriter` states that the partner portal reads its output when it writes no file. A reader who trusts the comments concludes the opposite of the truth.
4. **Orphaned schema makes the database its own separate source of truth.** Because the columns and tables that exist in production cannot be derived from the migrations in the repository, any question about the real schema has to be answered by asking a DBA — which is exactly the loop the sprint retrospective complains about (`조회 화면 없이 대기열 건수를 매번 데이터팀에 요청하는 상태가 반복되고 있다`, "we keep having to ask the data team for the queue count because there is no screen").
5. **Tests that never run are orphans with a false signal attached.** A `@Test` method reads as coverage. `-x test` in every workflow means the estate's five test classes assert nothing about any deployment, including the two tests that correctly encode the restock reason-code rule.
6. **The estate cannot distinguish "not built" from "built and unplugged".** The 2026 automation plan's first task is `현행 처리 규칙 문서화` ("document the current processing rules"), and its To-Be diagram describes what `CancelReconciler` already implements. The project is scoped to build something that exists.

**The fair summary.** Every individual deferral here was a defensible engineering choice at the moment it was made. What is absent is not judgement but a closing mechanism: nothing in this estate ever asks, after the fact, whether a thing that was written is reachable. The pattern is the missing question, not the missing code.

## Agent Guidance

- **Never infer that a component runs from the fact that it exists.** In this estate the base rate of unreachable-but-plausible code is high enough that existence is weak evidence. Before describing any behaviour as happening, find the caller.
- **Know each runtime's single registration point and check it directly.** settlement-batch: `QuartzConfig` beans and the steps in `DailySettlementJobConfig`. order-service: `@RestController` classes, of which there are two. inventory-api and settlement-anomaly: route decorators on the `app` object in `main.py`, not routers in other modules. delivery-bff: the `app.get` calls in `src/index.ts`. Anything outside these is unreachable unless something else calls it.
- **Run the caller grep and say what it returned.** `grep -rn <Identifier>` across all five repository roots, then subtract the hits inside the component's own file. Report the number. An assertion of absence without the command behind it is not usable by the next reader.
- **For a table, ask the write question and the read question separately, and ask about the DDL third.** This estate has an instance of each failure: `SETTLEMENT_RUN_LOG` (neither), `SETTLEMENT_RUN` (read, never written), `SETTLEMENT_ADJUSTMENT` (written, never created), `RESTORE_LOG` and `ORDER_STATUS_HIST` (created and indexed, never written).
- **Treat a code comment about usage as a claim to verify, not a fact.** Three comments in this estate assert usage that does not exist. Where a comment and a grep disagree, the grep wins and the disagreement is worth recording.
- **Do not reconcile the migration history with the schema by assuming the migrations are complete.** `V23__consolidated_schema.sql` puts the authoritative DDL outside version control. When a migration references a column no migration creates, the correct answer is that the repository cannot tell you, not that the column is missing in production — see [[SCH-ORDER-MIGRATION-HISTORY]].
- **When asked to fix an orphan, check whether the missing piece is a registration or a decision.** Scheduling `CancelReconciler` is four lines in `QuartzConfig` and would immediately post 4,127 clawback adjustments into a table that may not exist, for partners who were paid up to 41 months ago. The four lines are trivial; the consequence is not. Surface it rather than making it.
- **A `@Disabled` test and a commented-out CI step are the same finding as an unscheduled job.** When assessing whether behaviour is protected, check that the suite runs at all — see [[RISK-ORDER-DISABLED-TESTS]].
- **Record orphans in `known_unknowns` when the absence could be explained from outside the repositories.** Production cron entries, DBA scripts and manual invocations are invisible here. "No caller in the five repositories" is the honest claim; "never runs" usually is not.

## Related

- [[CON-ORDER-INVENTORY]] — the restock contract whose client, endpoint and caller are all orphaned
- [[PAT-SELLFLOW-DB-AS-QUEUE]] — why a table-based queue cannot report a missing consumer
- [[PROC-SELLFLOW-RUNTIME]] — what is actually scheduled, against what exists
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — the lifecycle whose terminal state only an unscheduled class can write
- [[RISK-ORDER-DISABLED-TESTS]] — the test suite as an orphan of the same kind
- [[RISK-SELLFLOW-DOC-DRIFT]] — documents that describe the orphans as working parts
- [[RISK-SETTLEMENT]] — the unscheduled reconciler and hardcoded fee rate at risk level
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measured cost of the oldest orphan
- [[SCH-ORDER-MIGRATION-HISTORY]] — the migration chain that references columns it never creates
