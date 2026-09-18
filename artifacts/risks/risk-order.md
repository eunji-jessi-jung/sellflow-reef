---
id: "RISK-ORDER"
type: "risk"
title: "Order Service Known Issues"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by a systematic scan of the whole repository: find over all 82 files, greps for TODO/FIXME/HACK/XXX/임시, for try/catch/throw, for log levels, for retry/timeout/circuit, for Java 9+ API usage against the declared sourceCompatibility, and a column-by-column comparison of every Flyway migration against the DDL that precedes it. Findings are grouped into ten themes and rated by signal density. Re-verified on 2026-09-19 after a correction pass in order-service: the migration chain V1-V24 now applies cleanly to an empty database, V1 creates ORDER_DELIVERY and ORDER_STATUS_HIST, OrderItem maps ORDER_DTL's real columns under a composite key, and OrderStatusService compiles against a new OrderMst.byeongyeong. Theme F4 drops accordingly; the other nine are unchanged. Re-verify after any test enablement, legacy removal or migration work — all three are named as intentions in the repo and none is done."
freshness_triggers:
  - ".github/workflows/ci.yml"
  - ".github/workflows/order-service-dev-cd.yml"
  - ".github/workflows/order-service-prod-cd.yml"
  - "SERVER_VERSION"
  - "TASK.md"
  - "build.gradle"
  - "codecov.yml"
  - "src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - "src/main/java/kr/co/sellflow/order/common/DateUtil.java"
  - "src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java"
  - "src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
  - "src/main/resources/db/migration/V23__consolidated_schema.sql"
  - "src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
known_unknowns:
  - "Whether this repository currently compiles. sourceCompatibility is '1.8' and all five of the six workflow files that build (every one except migration-issue.yml) pin setup-java to 8, while three files use Map.of/List.of, which are Java 9 APIs. Either the declared toolchain is not the one in use, or ./gradlew clean build has been failing; nothing in the repo resolves which, and no build log or artifact is committed."
  - "Whether the long-lived environments match what a clean V1-V24 run now produces. The chain applies cleanly today, but the existing databases were migrated incrementally over seven years and V23's consolidated baseline was never written, so nothing reconciles the two."
  - "Where deploy.sh is. All four CD workflows invoke ./deploy.sh <env> as their final step and the file is not in the repository; nor is the Dockerfile that the preceding docker build step requires. Logged in .reef/questions-for-owner.md."
  - "Actual test coverage. The CI test step is commented out, no JaCoCo plugin is applied and no workflow uploads to codecov, so the 40% target has never been evaluated."
  - "Whether the legacy OrderCancelServiceV1 is reachable at runtime. It has no Spring stereotype annotation yet uses @Autowired, so it cannot be injected as written, but the deprecation note keeps it for a batch that might reference it. Which batch is not named, and a grep across all five repos finds no caller."
  - "Whether GET /api/v1/orders/search has live callers. Its RowMapper returns null for every row, so any caller has been receiving lists of nulls since the code was written."
  - "Whether DateUtil's SimpleDateFormat is reached concurrently. It has no caller in this repository at all, which is itself notable given the comment claiming many references."
  - "The real-world exposure of the MoneyUtil rounding split (HALF_UP here, FLOOR in settlement). The comment says amounts can differ by one won, but no code here calls MoneyUtil.fee."
  - "Whether SF-4188, referenced by the migration-check workflow, is resolved. No ticket for it exists in the reef sources."
  - "Whether TASK.md's open items are tracked anywhere official. The file itself disclaims that status."
  - "Whether three CD workflows firing on the same push to develop is intentional fan-out to three environments or an unnoticed duplication. No document, comment or registry entry describes the environment topology."
severity: "high"
resolution: "unresolved"
tags:
  - "order"
  - "risk"
  - "tech-debt"
  - "tests"
  - "dead-code"
  - "ci-cd"
  - "migrations"
aliases:
  - "order-service risks"
relates_to:
  - type: "constrains"
    target: "[[API-ORDER]]"
  - type: "constrains"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[PROC-ORDER-AUTH]]"
  - type: "refines"
    target: "[[PROC-ORDER-ERROR-HANDLING]]"
  - type: "feeds"
    target: "[[RISK-ORDER-DISABLED-TESTS]]"
  - type: "constrains"
    target: "[[SCH-ORDER]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/ci.yml"
    notes: "Test step commented out; Java 8 toolchain."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/migration-issue.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-dev-cd.yml"
    notes: "One of three workflows triggered by the same push to develop."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-prod-cd.yml"
    notes: "Calls ./deploy.sh, which is not in the repository."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-qa-cd.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-stage-cd.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:SERVER_VERSION"
    notes: "2.8.14, against build.gradle's 2.8.4."
  - category: "implementation"
    type: "github"
    ref: "order-service:TASK.md"
    notes: "Personal notes serving as a backlog."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "POST /stock/restock with a JSON body — not the path or shape InventoryClient sends."
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "sourceCompatibility 1.8; no checkstyle, no jacoco."
  - category: "implementation"
    type: "github"
    ref: "order-service:checkstyle.xml"
    notes: "Config with no plugin to read it."
  - category: "implementation"
    type: "github"
    ref: "order-service:codecov.yml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "CANCEL_RECON_QUEUE consumer is TODO. Reef-root relative."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Calls /api/v1/orders/{ordNo}/cancel, a prefix the controller does not serve."
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
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderItem.java"
    notes: "Five of its six mapped columns do not exist in ORDER_DTL."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
    notes: "Map.of — a Java 9 API under sourceCompatibility 1.8."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/mapper/OrderMapper.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/repository/OrderStatusHistoryRepository.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderQueryService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderStatusService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application-prod.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application-qa.yml"
    notes: "Points at a database named order, not sellflow_order."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V13__cleanup_unused.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V17__add_cancel_channel.sql"
    notes: "Adds CHNL_CD, which the OrderCancel entity never maps."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V18__backfill_cancel_channel.sql"
    notes: "Backfills CHNL_CD from ORDER_MST.CHAENNEL_CD, filtered on ORDER_CANCEL.CHWISO_ILSI."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V21__add_settlement_ref_index.sql"
    notes: "Indexes JUNGSAN_RUN_ID (V14) — a column nothing writes or reads."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V23__consolidated_schema.sql"
    notes: "Comment-only consolidated migration."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V24__add_order_memo_search_index.sql"
    notes: "Prefix-indexes GOGAEK_MEMO (V2); no query filters on it."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/search/OrderSearchServiceTest.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
notes: "Severity raised from medium to high on 2026-09-19 on density grounds: three of ten themes carry 10+ independent signals each. Impact is still argued from code reading rather than production evidence — no incident record naming order-service was found."
---

# Order Service Known Issues

## Description

A systematic sweep of order-service — all 82 files, grouped into ten themes and rated by signal density.

The pattern that connects most of it: several things were started (a test suite, a legacy removal, an inventory integration, a column rename, a schema consolidation) and each was stopped partway, leaving the half-state in the tree with a comment explaining why nobody finished. The comments are unusually honest, which makes the reading easy and the conclusions uncomfortable.

The deeper scan adds a second pattern the first pass did not surface: **several things in this repository cannot be true at once.** The build declares Java 8 and uses Java 9 APIs. The migration chain indexes four columns that no migration creates. Three deploy workflows fire on the same push. A deploy step calls a script that is not in the tree, after a docker build with no Dockerfile. Each of these is the kind of contradiction that a working CI would surface within one commit — and CI here compiles with `-x test` and nothing else.

## Key Facts

### Tests and CI

- The CI test step is commented out entirely; the build runs `./gradlew clean build -x test` → .github/workflows/ci.yml
- The reason is recorded in place: "TODO 테스트 켜야 함. OrderCancelServiceTest 가 로컬 DB 를 타서 CI 에서 깨짐 - 2023-05-11 박성민" ("TODO: must turn tests on. OrderCancelServiceTest hits the local DB and breaks in CI") → .github/workflows/ci.yml
- All four CD workflows use the same `-x test` build, so no deployment to any environment is gated by a test → .github/workflows/order-service-prod-cd.yml
- `OrderCancelServiceTest.정상_취소` is `@Disabled("로컬 DB 필요. CI 에서 깨져서 임시 비활성화 - 2023-05-11")` ("needs a local DB; temporarily disabled because it breaks in CI") — still disabled over three years later → src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java
- The whole `OrderCancelServiceV1Test` class is `@Disabled("SF-2287 이후 정책 변경. legacy 클래스 제거 시 함께 삭제")`, and its single method has an empty body, so it would pass vacuously if enabled → src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java
- The codecov project target is 40% with a 5% threshold, while `build.gradle` applies no JaCoCo plugin and no workflow uploads a report — the gate has no producer → codecov.yml
- Full detail, method by method, is in [[RISK-ORDER-DISABLED-TESTS]] → src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java

### Build and toolchain contradictions

- `build.gradle` declares `sourceCompatibility = '1.8'`, and every one of the six workflow files that runs a build — all but `migration-issue.yml` — pins `actions/setup-java` to `java-version: '8'` → build.gradle, .github/workflows/
- `GlobalExceptionHandler` calls `Map.of(...)` twice — an API introduced in Java 9 → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- `OrderMapper.toAppView` calls `java.util.Map.of(...)`, and `OrderSearchServiceTest` calls `List.of()`, both Java 9 → src/main/java/kr/co/sellflow/order/mapper/OrderMapper.java
- Either the declared toolchain is not the one used, or the build has been failing to compile; the repository provides no evidence either way → build.gradle
- `SERVER_VERSION` reads `2.8.14` while `build.gradle` declares `version = '2.8.4'` — two version sources that disagree, with no code reading either → SERVER_VERSION
- `checkstyle.xml` configures `UnusedImports` and `EmptyBlock`, but no checkstyle plugin is applied in `build.gradle`, so the config is never read — notably, `EmptyBlock` is exactly the rule the legacy class's three empty catch blocks would trip → checkstyle.xml
- Spring Boot 2.3.12 reached end of open-source support in 2021 and end of commercial support in 2023; `mysql-connector-java` is declared with no version and under its relocated coordinate → build.gradle

### Migration and schema findings

- The chain applies: every column V13, V18, V20, V21, V22 and V24 touches is created by an earlier migration, so V1-V24 runs against an empty database without hand-patching → src/main/resources/db/migration/
- `V23__consolidated_schema.sql` still contains no DDL — four comment lines stating that the consolidated dump "아직 작성하지 않았다" ("has not been written yet"), that new environments "계속 V1 부터 순차 적용한다" ("keep applying from V1 in order"), and a TODO with no assignee → src/main/resources/db/migration/V23__consolidated_schema.sql
- Two index migrations serve no query: V21's `IDX_ORDER_MST_SETTLE_REF (JUNGSAN_RUN_ID)` indexes a column nothing in the five repos writes or reads, and V24's `IDX_ORDER_MST_MEMO (GOGAEK_MEMO(64))` indexes a column no query filters on → src/main/resources/db/migration/V21__add_settlement_ref_index.sql, src/main/resources/db/migration/V24__add_order_memo_search_index.sql
- V22's `IDX_ORDER_CANCEL_STATUS (CHORI_SANGTAE, CHWISO_ILSI)` leads on a column whose only written value is the literal `"COMPLETED"`, so it behaves as an index on `CHWISO_ILSI` alone → src/main/resources/db/migration/V22__order_cancel_status_index.sql, src/main/java/kr/co/sellflow/order/domain/OrderCancel.java
- `V18`'s backfill derives the *cancellation* channel from the *order's* channel — `m.CHAENNEL_CD = 'APP'` — so `CHNL_CD = 'APP'` records how the order arrived, not how the cancellation did → src/main/resources/db/migration/V18__backfill_cancel_channel.sql
- `V15` renamed `ORDER_CANCEL.BIGO` to `MEMO` and `V16` reverted it at most 18 days later — V15's comment gives only `2024-01`, V16's `2024-01-18`, so the exact interval is not recorded — with the cause stated: "V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애. 2024-01-18" ("V15 rollback; the settlement batch query references BIGO directly, causing an incident") → src/main/resources/db/migration/V16__revert_rename_bigo.sql
- `OrderItem` maps `ORDER_DTL`'s real columns — `ORD_NO`, `ORD_SEQ`, `SANGPUM_CD`, `OKSYEON_MYEONG`, `SURYANG`, `DANGA`, `PARTNER_ID` — under an `@IdClass(OrderItem.Pk)` matching V1's composite primary key, and `OrderItemRepository` is keyed on `OrderItem.Pk` → src/main/java/kr/co/sellflow/order/domain/OrderItem.java, src/main/java/kr/co/sellflow/order/repository/OrderItemRepository.java
- Three `ORDER_DTL` columns added after 2020 remain unmapped — `GONGGEUP_GA` (V4), `CHANGGO_CD` (V12) and `OPT_AMT` (V19) — so a JPA read of a line item omits the supply price, the warehouse and the option surcharge → src/main/java/kr/co/sellflow/order/domain/OrderItem.java, src/main/resources/db/migration/V19__order_dtl_option_price.sql
- `OrderSearchService` still queries `SELECT * FROM ORDER_MST WHERE ORD_DT BETWEEN ? AND ?` — `ORD_DT` is created by no migration, and is now the only phantom column any production code names → src/main/java/kr/co/sellflow/order/search/OrderSearchService.java

### Broken outbound contracts

- `InventoryClient.restore` posts to `{baseUrl}/inventory/restore?ordNo=..&reason=..` with a null body, while `inventory-api` serves `POST /stock/restock` with a JSON body `{ord_no, reason_code}` — path, parameter style and field names all differ → src/main/java/kr/co/sellflow/order/client/InventoryClient.java
- `inventory-api`'s `/inventory`-prefixed `APIRouter` in `app/routers.py` is never included in the FastAPI app, so nothing is served under that prefix at all → inventory-api:app/main.py
- `PaymentClient.cancel(ordNo, amount)` logs both arguments and then posts a null body to a fixed URL, sending neither the order number nor the amount to the PG → src/main/java/kr/co/sellflow/order/payment/PaymentClient.java
- delivery-bff calls `POST /api/v1/orders/{ordNo}/cancel`, while `OrderController` maps `POST /orders/{ordNo}/cancel` with no `/api/v1` prefix and no `context-path` set in any profile → delivery-bff:src/generated/orderApi.ts
- The single CORS mapping applies to `/api/**`, which matches the prefix delivery-bff calls but not the path the controller serves — so whichever of the two is real, the other is misconfigured → src/main/java/kr/co/sellflow/order/config/WebConfig.java
- Neither client has a caller: `grep -rn "InventoryClient\|PaymentClient" src/` matches only the declarations and PaymentClient's own logger field → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java

### Error handling and observability

- Outside the deprecated legacy class there is no `try`, `catch` or `finally` anywhere in the service → src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- The legacy class swallows three close failures in `catch (Exception ignore) {}` blocks and calls `conn.rollback()` unguarded inside its catch, where a rollback failure would replace the original cause → src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- `GlobalExceptionHandler` maps two exceptions (404, 409); an invalid `sayuCd` raises an unmapped `IllegalArgumentException` and returns 500 where the spec declares 400 → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- The whole main tree contains three log statements, all INFO, all on success paths; there is no `log.error` or `log.warn` anywhere, so a total failure of the cancel path would raise no log-based alert → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The success log reads `prevStatus` after `order.chwiso()` has already mutated it, so it always prints `CHWISO` → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- Both RestTemplate clients are constructed as bare `new RestTemplate()` with no connect or read timeout and no error handler; the only timeout in the repo is Hikari's `connection-timeout: 3000` in production → src/main/resources/application-prod.yml
- Nothing retries anything: no `@Retryable`, no loop, no sweep, no dead-letter table → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- Full analysis in [[PROC-ORDER-ERROR-HANDLING]] → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java

### Deploy pipeline

- Three separate CD workflows — dev, qa and stage — all trigger on `push: branches: [develop]`, so every merge to develop starts three concurrent build-and-deploy runs against the same registry tag namespace → .github/workflows/order-service-dev-cd.yml
- All four CD workflows end with `run: ./deploy.sh $ENVIRONMENT`, and `deploy.sh` does not exist in the repository → .github/workflows/order-service-prod-cd.yml
- The preceding step runs `docker build -t $REGISTRY/$SERVICE:...` from the repository root, and there is no `Dockerfile` in the tree either → .github/workflows/order-service-dev-cd.yml
- No workflow performs a registry login before `docker push`, and none declares a GitHub `environment` or any approval gate, including the production one → .github/workflows/order-service-prod-cd.yml
- The migration-check workflow is explicitly a stopgap: "SF-4188 대응용. 마이그레이션 적용 여부만 확인한다. 임시." ("for SF-4188; checks only whether migrations were applied; temporary") — and it is `workflow_dispatch` only, so it runs when someone remembers → .github/workflows/migration-issue.yml

### Hardcoded endpoints and configuration drift

- `InventoryClient` reads its base URL from `System.getenv()` rather than Spring configuration, defaulting to `http://inventory-api.internal` → src/main/java/kr/co/sellflow/order/client/InventoryClient.java
- `PaymentClient` does the same, defaulting to `https://pg.example.co.kr` — a placeholder domain as the production fallback for a payment gateway → src/main/java/kr/co/sellflow/order/payment/PaymentClient.java
- The production datasource host `prod-db.internal` is hardcoded in `application-prod.yml`, which sets no username and no password → src/main/resources/application-prod.yml
- The qa and stage profiles point at a database named `order`, while the base profile and `.env.example` disagree with each other and with prod's `sellflow_order` → src/main/resources/application-qa.yml
- No credential is committed anywhere: `.env.example` has an empty `DB_PASSWORD` and `application-local.yml.example` uses `CHANGE_ME`. This is a deliberate negative finding — the scan looked for hardcoded credentials and found none → .env.example

### Dead and deprecated code

- `OrderCancelServiceV1` still carries a full raw-JDBC cancel implementation and is kept deliberately: "삭제 예정이나 배치에서 참조 가능성이 있어 남겨둠. - 2023-04-21 박성민" ("scheduled for deletion but kept because a batch might reference it") → src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- It also enforces the superseded policy, throwing on any order present in `SETTLEMENT_DTL` — the precise check SF-2287 removed, so two contradictory cancel policies coexist in the source tree → src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- `OrderMapper.toAppView` has no caller despite its javadoc "응답 변환. 어드민과 앱이 다른 필드를 본다." ("response conversion; admin and app see different fields") — no code differentiates the two views → src/main/java/kr/co/sellflow/order/mapper/OrderMapper.java
- `DeliveryInfoRepository` and `OrderStatusHistoryRepository` have no callers, so `ORDER_DELIVERY` and `ORDER_STATUS_HIST` are never read or written by this service → src/main/java/kr/co/sellflow/order/repository/OrderStatusHistoryRepository.java
- `OrderQueryService` and `OrderStatusService` have no production callers either — each is reachable only from its own unit test → src/main/java/kr/co/sellflow/order/service/OrderQueryService.java
- `OrderStatusService.change` now compiles and works — it calls `OrderMst.byeongyeong(to)` and saves — which moves it from broken to merely unreachable; five of the seven `OrderStatus` values still have no writer anywhere → src/main/java/kr/co/sellflow/order/service/OrderStatusService.java, src/main/java/kr/co/sellflow/order/domain/OrderMst.java
- `ORDER_DELIVERY` and `ORDER_STATUS_HIST` are created by V1 and mapped column for column by `DeliveryInfo` and `OrderStatusHistory`, so they are now fully-formed tables that nothing in any of the five repos reads or writes → src/main/resources/db/migration/V1__init.sql, src/main/java/kr/co/sellflow/order/repository/OrderStatusHistoryRepository.java
- `OrderStatus`'s javadoc refers readers to an `OrderStatusValidator` that does not exist in the repository → src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- `MoneyUtil.fee` has no caller, and its javadoc flags a cross-service inconsistency: this class rounds HALF_UP while settlement-batch's same-named class truncates (FLOOR), so "표시 금액과 지급 금액이 1원 단위로 다를 수 있다" ("the displayed amount and the paid amount can differ by one won") → src/main/java/kr/co/sellflow/order/common/MoneyUtil.java
- `ORDER_CANCEL.CHNL_CD`, added by V17 to record the cancellation channel, is not mapped by the `OrderCancel` entity, so every row written since has left it NULL → src/main/resources/db/migration/V17__add_cancel_channel.sql

### Misleading in-code documentation

- `DateUtil` carries a warning that does not describe the code it guards — "주의: SimpleDateFormat 은 thread-safe 하지 않다. 인스턴스 재사용 금지." ("SimpleDateFormat is not thread-safe; do not reuse the instance") sits above a `private static final String` pattern, and each method constructs a fresh formatter → src/main/java/kr/co/sellflow/order/common/DateUtil.java
- Its class header discourages maintenance on false grounds — "2019년 초기 구축분. 여러 곳에서 참조하고 있어 손대지 않는 것을 권장." ("initial 2019 build; referenced in many places, so it is recommended not to touch it") — while grep finds zero references outside the class → src/main/java/kr/co/sellflow/order/common/DateUtil.java
- `DateUtil.settlementBaseDate()` encodes a cross-team business rule (the settlement base date is the previous day because the batch runs at 02:00) in an unused utility, duplicating knowledge owned by settlement-batch → src/main/java/kr/co/sellflow/order/common/DateUtil.java
- `OrderSearchService`'s SQL is assembled with a `StringBuilder`, both branches use a `(rs, i) -> null` RowMapper so every returned element is null, and the query filters on `ORD_DT`, a column no migration creates → src/main/java/kr/co/sellflow/order/search/OrderSearchService.java
- A standing CS complaint about that endpoint was closed as a display problem: "FIXME(은영) 2024-05: 상태코드 필터에 JUNGSAN_WANRYO 넣으면 결과가 비어 보인다는 CS 문의. 실제로는 상태가 갱신돼 있어서 나오는 게 맞음. 화면 안내 문구만 고치기로 함." ("CS reports that filtering by JUNGSAN_WANRYO looks empty; in reality the status is updated so the result is correct; we decided to fix only the on-screen wording") → src/main/java/kr/co/sellflow/order/search/OrderSearchService.java
- The search controller documents a three-month maximum range that no code enforces → src/main/java/kr/co/sellflow/order/search/OrderSearchController.java
- `TASK.md` is the only written record of four open items, and disclaims its own authority: "※ 개인 메모입니다. 공식 문서 아님." ("this is a personal memo, not an official document") → TASK.md
- One of those items — "정산팀 컨슈머 확인 — 김도윤님 확인 후" (confirm the settlement team's consumer, pending 김도윤) — matches an unresolved TODO in the service registry, so the same open question appears in two places and is answered in neither → sellflow-docs:context/registry/services.yaml

## Findings

Severity is assigned by signal density as instructed: 10 or more independent signals in a theme is high, 5 to 9 is medium, 1 to 4 is low. Where impact and density point in different directions, the note says so.

| # | Theme | Signals | Severity | Representative evidence | Density justification |
|---|---|---|---|---|---|
| F1 | **Tests and CI** — two `@Disabled` markers, one vacuously empty test body, a commented-out CI test step, `-x test` in four CD workflows, two DB-dependent `@SpringBootTest` methods, an orphaned codecov target, no JaCoCo plugin, and three production areas (outbox, error mapping, clients) with no test reference at all | 12 | **high** | ci.yml; OrderCancelServiceTest; OrderCancelServiceV1Test; codecov.yml | 12 ≥ 10. Density and impact agree; this is the one theme where impact would argue for high on its own. Detailed in [[RISK-ORDER-DISABLED-TESTS]]. |
| F2 | **Dead and deprecated code** — legacy V1 service, two uncalled HTTP clients, `OrderMapper.toAppView`, two uncalled repositories, two uncalled services, `MoneyUtil.fee`, `DateUtil` in full, a missing `OrderStatusValidator`, and the unmapped `CHNL_CD` | 12 | **high** | OrderCancelServiceV1; InventoryClient; PaymentClient; DateUtil | 12 ≥ 10. Density is high; individual impact is mostly low, but the aggregate effect — a reader cannot tell what runs — is what makes it costly. |
| F3 | **Error handling and observability** — no try/catch outside legacy, three empty catch blocks, an unguarded rollback, two mapped exceptions only, an unmapped `IllegalArgumentException` → 500, no validation, three INFO logs and no ERROR/WARN, a wrong `prevStatus` log, no timeouts on outbound HTTP, no retry, no dead letter, `CHORI_SANGTAE` hardcoded to COMPLETED, and a silent `ifPresent` no-op | 13 | **high** | GlobalExceptionHandler; OrderCancelService; OrderCancelServiceV1; InventoryClient | 13 ≥ 10. Fully analysed in [[PROC-ORDER-ERROR-HANDLING]]. |
| F4 | **Schema and migration residue** — a comment-only consolidated baseline that was never written, two index migrations serving no query, an index leading on a single-valued column, a backfill that derives the cancellation channel from the order's, a rename-and-revert pair caused by a production incident, three `ORDER_DTL` columns the entity does not map, and a held-back cleanup | 5 | **medium** | V23; V21, V22, V24; V15/V16; OrderItem | 5, bottom of the 5-9 band — down from 9 after the correction pass made V1-V24 apply cleanly. What remains is unused schema rather than unrunnable schema; see [[SCH-ORDER]] and [[SCH-ORDER-MIGRATION-HISTORY]]. |
| F5 | **Hardcoded endpoints and config drift** — two base URLs read from `System.getenv` with hardcoded defaults (one of them an `example.co.kr` placeholder for a payment gateway), a hardcoded prod DB host, no prod credentials at all, three different database names across profiles, and a registry hostname hardcoded in four workflows | 9 | **medium** | InventoryClient; PaymentClient; application-prod.yml; application-qa.yml | 9, top of the medium band. Mitigating and explicitly checked: **no credential is committed anywhere** — the example files use `CHANGE_ME` and an empty password. |
| F6 | **Deploy pipeline** — three CD workflows on one `push: develop` trigger, a missing `deploy.sh`, a missing `Dockerfile`, no registry login before push, no environment or approval gate on production, `-x test` in all four, and a `임시` (temporary) dispatch-only migration checker | 8 | **medium** | order-service-{dev,qa,stage,prod}-cd.yml; migration-issue.yml | 8, in the medium band. The missing `deploy.sh` and `Dockerfile` mean the workflows as committed cannot describe what actually deploys; logged for the owner. |
| F7 | **Build and toolchain contradictions** — `sourceCompatibility '1.8'` with `Map.of`/`List.of` in three files, Java 8 pinned in the five building workflows of six, `SERVER_VERSION` disagreeing with `build.gradle`, a checkstyle config with no plugin, an EOL Spring Boot, and an unversioned relocated MySQL coordinate | 9 | **medium** | build.gradle; GlobalExceptionHandler; OrderMapper; SERVER_VERSION; checkstyle.xml | 9, top of the medium band. Impact argues higher — as declared, the project would not compile — but the contradiction may mean the declared toolchain is simply not the one in use, which is itself the finding. |
| F8 | **Broken outbound contracts** — the InventoryClient path/shape mismatch (three distinct differences), an inventory router never mounted, a PG call carrying neither order number nor amount, the `/api/v1` prefix mismatch with delivery-bff, and a CORS pattern matching neither side consistently | 7 | **medium** | InventoryClient; inventory-api:app/main.py; PaymentClient; delivery-bff:src/generated/orderApi.ts | 7, mid medium band. Currently latent because neither client has a caller, which is precisely why the mismatches have never been noticed. See [[CON-ORDER-INVENTORY]]. |
| F9 | **TODO/FIXME/임시 markers** — nine dated markers, four of which defer a decision on the grounds that "a batch might reference it" | 9 | **medium** | table below | 9, top of the medium band. No `HACK` or `XXX` marker exists anywhere; the scan looked for all four keywords. |
| F10 | **Misleading in-code documentation** — DateUtil's two false claims, the absent `OrderStatusValidator`, the unenforced three-month range, the FIXME closed as a wording fix, `TASK.md` as an unofficial backlog, and a javadoc describing an admin/app view split that no code implements | 6 | **medium** | DateUtil; OrderStatus; OrderSearchController; TASK.md | 6, lower medium band. Each item is individually minor; together they mean the comments cannot be trusted as documentation, which is the load-bearing problem for anyone maintaining this service. |

### TODO, FIXME and 임시 inventory (theme F9 in detail)

A grep for `TODO`, `FIXME`, `HACK`, `XXX` and `임시` across all 82 files returns nine markers. There are no `HACK` and no `XXX` markers.

| Marker | Location | Date / author | Text (verbatim) — meaning |
|---|---|---|---|
| TODO | .github/workflows/ci.yml | 2023-05-11, 박성민 | "테스트 켜야 함. OrderCancelServiceTest 가 로컬 DB 를 타서 CI 에서 깨짐" — tests must be turned on; the test hits a local DB and breaks CI |
| TODO | src/main/java/.../service/OrderCancelService.java | undated | "정산 연동 관련 확인 필요 (SF-2287 이후 정책 변경됨)" — settlement integration needs checking; policy changed after SF-2287 |
| TODO | src/main/java/.../domain/OrderStatusHistory.java | 2022-11-08, 성민 | "정산완료 전이는 여기 안 쌓인다. 배치에서 직접 UPDATE 하기 때문." — settled transitions are not recorded here because the batch UPDATEs directly |
| TODO | src/main/java/.../payment/PaymentClient.java | 2021-06-02, 성민 | "부분취소 미지원. 전액 취소만 호출한다." — partial cancellation unsupported; only full cancellation is called |
| FIXME | src/main/java/.../search/OrderSearchService.java | 2024-05, 은영 | the JUNGSAN_WANRYO filter complaint, closed as a wording fix |
| TODO | src/main/resources/db/migration/V13__cleanup_unused.sql | 2025-03-11 | "ORDER_MST.BAESONG_MSG 도 미사용으로 보이나 배치에서 참조 가능성 있어 보류" — BAESONG_MSG also looks unused but is held back in case a batch references it |
| TODO | src/test/java/.../service/OrderCancelServiceTest.java | undated | "정산 완료 주문 취소 케이스 테스트 필요" — a test is needed for the settled-order cancel case |
| TODO | src/test/java/.../legacy/OrderCancelServiceV1Test.java | 2023-04-21, 성민 | "legacy 제거 시 같이 삭제" — delete together with the legacy class |
| 임시 (temporary) | .github/workflows/migration-issue.yml | undated | "SF-4188 대응용. 마이그레이션 적용 여부만 확인한다. 임시." — for SF-4188; checks only whether migrations applied; temporary |

Four of the nine are variations on the same sentence: something is kept or skipped because a batch elsewhere might depend on it. A grep across all five repositories finds no such dependency for any of them. Nobody appears to have checked.

## Impact

**Cancellation is the least-tested code in the service, and the only write path.** The happy path has been untested in CI since 2023-05-11, twenty days after the policy it implements was changed. A regression in `OrderCancelService` would reach production through a build that explicitly excludes tests, in both the CI and the production deploy workflow. See [[RISK-ORDER-DISABLED-TESTS]].

**Two contradictory cancel policies are in the tree.** `OrderCancelService` permits cancelling settled orders; `OrderCancelServiceV1` forbids it. If anything still calls the legacy class — which the deprecation note explicitly allows for — the effective policy depends on which entry point a caller uses.

**Side effects that look implemented are not, and would not work if they were.** A reader encountering `InventoryClient` and `PaymentClient` would reasonably conclude that cancelling an order restores stock and refunds the payment. Neither is called; and the inventory call, if wired in as written, would 404 against `inventory-api`'s actual `/stock/restock`, while the PG call would send no order number. The business-rules sheet's restock column and the Confluence page's 전액 환불 (full refund) promise have no working implementation. See [[PROC-ORDER-CANCEL]] and [[CON-ORDER-INVENTORY]].

**A failure is invisible.** With three INFO log statements, no ERROR level anywhere, no actuator, no metrics and no request id, a broken cancel path would be detectable only by the absence of success lines or by customer complaint. See [[PROC-ORDER-ERROR-HANDLING]].

**Search returns nulls.** Any consumer of `GET /api/v1/orders/search` receives a list whose every element is null, from a query filtering on a column that does not exist. Combined with the FIXME reading an emptiness complaint as a wording problem, a symptom was explained away rather than traced. See [[API-ORDER]].

**The schema is reproducible; nothing checks that it matches production.** V1-V24 now applies cleanly to an empty database, so a new environment can be built from these files. What is still missing is reconciliation: V23's consolidated baseline was never written, the long-lived environments were migrated incrementally over seven years, and the one workflow that would check — `migration-issue.yml` — is a `임시` (temporary) manual dispatch running `flywayInfo`, which reports applied versions rather than comparing them to the DDL. See [[SCH-ORDER]].

**The deploy pipeline is not described by its own definitions.** Two files the workflows depend on are absent, three of them race on every push to develop, and production deploys on a push to main with no approval and no test.

**The audit trail is thin.** No `ORDER_STATUS_HIST` rows, a `prevStatus` log line that always prints `CHWISO`, an unmapped `CHNL_CD`, and no caller identity recorded on cancellation ([[PROC-ORDER-AUTH]]). Reconstructing who cancelled what, and from what state, is not possible from this service's own data.

## Severity and Resolution

**Severity: high** — raised from medium two passes ago and held here. The rating follows the density rule: three of the ten themes (F1 tests and CI, F2 dead code, F3 error handling) each carry 12 or more independent signals, and three more sit at the top of the medium band with 8 or 9. One theme would argue for high on impact grounds regardless of count — the toolchain contradiction (F7), which as declared would not compile — but it is held at medium to keep the rule applied consistently. F4 fell from 9 to 5 in this pass: the migration chain now builds a clean environment, so what was an impact argument for raising it has gone.

What keeps this short of critical: no incident record in the reef's sources names an order-service regression, the service is small and its live logic short, and the most alarming individual items (the broken clients, the unmapped search column) are latent rather than active — nothing calls the clients, and the environments already exist. The judgement still rests on code reading, not on production evidence.

**Resolution: unresolved.** Every item is open. The repository's own intentions — enable tests, delete the legacy class, clean up V15/V16, implement partial cancellation — are recorded in `TASK.md`, a file that declares itself unofficial, and enabling the tests is not even among its four items.

## Recommended Actions

1. **Answer the "a batch might reference it" question once.** Four deferrals (the legacy class, `BAESONG_MSG`, the settlement consumer, the V15 rename) all rest on the same unverified fear. A single audit of what settlement-batch and any other job actually reads from `sellflow_order` would unblock all four. It is the cheapest high-leverage step available.
2. **Uncomment the CI test step and see what happens.** Whether the build even compiles under Java 8 with `Map.of` in three files is currently unknown, and this one-line change answers both that and the state of the suite. Then work through [[RISK-ORDER-DISABLED-TESTS]].
3. **Reconcile the applied schema with the repaired chain.** V1-V24 applies cleanly now, so the open question is whether the long-lived environments match it. Run the information_schema comparison in [[SCH-ORDER-MIGRATION-HISTORY]] against each environment; the presence of `TEMP_FLAG`, `JEOKRIPGEUM` or `HALIN_GEUMAEK` would show an environment that never got V13 or V20. Then either write V23's baseline or delete the placeholder.
4. **Resolve the toolchain contradiction explicitly.** Either raise `sourceCompatibility` and the workflow JDKs to a version that supports `Map.of`, or replace the three call sites. Leaving it ambiguous means nobody knows which JDK production runs on.
5. **Collapse the three develop-triggered CD workflows**, or document why dev, qa and stage are meant to deploy simultaneously from one push.
6. **Commit `deploy.sh` and the `Dockerfile`, or point the workflows at where they really live.** A pipeline whose two most consequential steps are undefined in the repository cannot be reviewed.
7. **Add an ERROR log and a timeout before anything else in the cancel path.** One `log.error` in a catch around the transaction and a configured `RestTemplate` timeout would convert the current silent-failure mode into something operable, at a cost of a few lines.
8. **Delete `DateUtil`, `MoneyUtil.fee`, `OrderMapper`, the two uncalled repositories and the two uncalled services**, or give them callers. Every one of them currently teaches a reader something false about what the service does.

## Related

- [[SYS-ORDER]] — the service these issues belong to
- [[RISK-ORDER-DISABLED-TESTS]] — theme F1 in full, answering Q-012
- [[PROC-ORDER-ERROR-HANDLING]] — theme F3 in full
- [[PROC-ORDER-AUTH]] — the missing caller identity behind the thin audit trail
- [[SCH-ORDER]] — the schema discrepancies in detail
- [[API-ORDER]] — the search endpoint and the spec drift
- [[PROC-ORDER-CANCEL]] — the missing side effects in the cancel flow
- [[CON-ORDER-INVENTORY]] — the restock contract neither side executes
