---
id: "SYS-ORDER"
type: "system"
title: "Order Service"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-read line-by-line against the order-service repo on 2026-09-19: build.gradle, SERVER_VERSION, the base application.yml plus the four environment profiles and the local template, every config class, both controllers, all 24 Flyway migrations, the full test tree and all six .github/workflows files. Goes stale if build.gradle, application*.yml, WebConfig, the controller set, db/migration or the CD workflows change. Re-verified on 2026-09-19 after a correction pass in order-service: V1-V24 now applies cleanly to an empty database and V1 creates all five of its tables, so the migration caveat is withdrawn — see [[SCH-ORDER-MIGRATION-HISTORY]]. The repo still disagrees with itself on version (SERVER_VERSION 2.8.14 vs build.gradle 2.8.4)."
freshness_triggers:
  - ".github/workflows/ci.yml"
  - ".github/workflows/order-service-prod-cd.yml"
  - "SERVER_VERSION"
  - "build.gradle"
  - "src/main/java/kr/co/sellflow/order/config/WebConfig.java"
  - "src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "src/main/java/kr/co/sellflow/order/search/OrderSearchController.java"
  - "src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "src/main/resources/application.yml"
  - "src/main/resources/db/migration/*.sql"
known_unknowns:
  - "No order-creation code exists in the repo. `grep -rn 'INSERT INTO ORDER_MST'` across all five repos returns only V1__init.sql's CREATE TABLE and the legacy cancel class's UPDATE — nothing inserts an ORDER_MST row anywhere in the sellflow source tree. Where orders are actually created is unresolved."
  - "The deployment topology beyond the CD workflows is unknown. All four CD workflows run `docker build -t $REGISTRY/$SERVICE:...` and `./deploy.sh $ENVIRONMENT`, but neither a Dockerfile nor deploy.sh is present anywhere in the repo (confirmed by a full `find . -type f`)."
  - "services.yaml lists the public endpoint as https://api.sellflow.co.kr/orders, delivery-bff's generated client calls the path /api/v1/orders/{ordNo}/cancel, and application.yml only sets port 8081 with no context path. Which hostname and path prefix front the service in production is not resolvable from these files. Raised in questions-for-owner.md."
  - "박성민 is named as the assignee on SF-2287 and as the author of most repo comments (TASK.md, the legacy class, the CI comment, the PaymentClient TODO), but no file in the repo or in services.yaml states current individual ownership; services.yaml deliberately records team only — 담당자 정보는 org-chart 를 따르며 여기에는 팀만 적는다 (\"individual-owner information follows the org chart; only the team is recorded here\")."
  - "services.yaml carries `last_reviewed: 2026-03-02` with the inline note 이후 갱신 없음 (\"no update since\"), so its description of the service may lag the code."
  - "The stage and qa profiles point at a database named `order`, not `sellflow_order`, and .env.example likewise sets DB_URL to .../order. Whether this is a real environment difference or a copy-paste error is not stated anywhere."
  - "Whether the deployed artifact actually compiles under Java 8. GlobalExceptionHandler and OrderMapper both use `Map.of(...)`, a Java 9 API, while build.gradle sets sourceCompatibility 1.8 and every workflow provisions java-version 8. The repo contains no build output to check against."
  - "What is meant to call OrderStatusService. It compiles against OrderMst.byeongyeong(OrderStatus) and saves the entity, and no caller exists outside its unit test, so the five OrderStatus values nothing writes stay unwritten."
tags:
  - "order"
  - "spring-boot"
  - "java8"
  - "mysql"
  - "sellflow"
aliases:
  - "order-service"
  - "주문 서비스"
relates_to:
  - type: "refines"
    target: "[[API-ORDER]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[DEC-ORDER-OUTBOX-RELAY]]"
  - type: "refines"
    target: "[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]]"
  - type: "refines"
    target: "[[GLOSSARY-ORDER]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-ORDER-AUTH]]"
  - type: "refines"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[PROC-ORDER-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-ORDER-ORDER-MST-LIFECYCLE]]"
  - type: "refines"
    target: "[[RISK-ORDER]]"
  - type: "refines"
    target: "[[RISK-ORDER-DISABLED-TESTS]]"
  - type: "refines"
    target: "[[SCH-ORDER]]"
  - type: "refines"
    target: "[[SCH-ORDER-MIGRATION-HISTORY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "The only external caller of the cancel endpoint found in any repo; generated 2022-11-08 and still describes the pre-SF-2287 409 behaviour."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "States the vocabulary mismatch with the order domain; exposes /stock/restock, not the /inventory/restore path InventoryClient builds."
  - category: "implementation"
    type: "github"
    ref: "order-service:.env.example"
    notes: "PG_BASE_URL and INVENTORY_BASE_URL — the only record that the two HTTP clients are configurable."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/CODEOWNERS"
    notes: "Migrations need 주문팀 plus the DBA group; legacy/ is pinned to 주문팀."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/ci.yml"
    notes: "Builds with -x test; the Test step is commented out."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-prod-cd.yml"
    notes: "Prod deploy on push to main; dev, qa and stage CD all trigger on push to develop."
  - category: "implementation"
    type: "github"
    ref: "order-service:README.md"
    notes: "States the stack, the database and the owning team, and that settlement-batch owns JUNGSAN_WANRYO."
  - category: "implementation"
    type: "github"
    ref: "order-service:SERVER_VERSION"
    notes: "2.8.14 — disagrees with build.gradle's 2.8.4."
  - category: "implementation"
    type: "github"
    ref: "order-service:TASK.md"
    notes: "박성민's personal checklist; records the outbox consumer confirmation and the V15/V16 cleanup as still open."
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "Spring Boot 2.3.12.RELEASE, sourceCompatibility 1.8, Flyway, MySQL connector; five dependencies total."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/config/JpaConfig.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java"
    notes: "@Deprecated, no stereotype annotation, zero callers repo-wide."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application-prod.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "The outbox consumer — polls ORDER_EVENT_OUTBOX and fills CANCEL_RECON_QUEUE."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "Writes SANGTAE_CD='JUNGSAN_WANRYO' into ORDER_MST directly."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "Records that settlement shares the order database instance."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Service registry; owner team 주문팀, last_reviewed 2026-03-02."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "Names 박성민 (주문팀) as assignee; carries the 월 10건 미만 volume estimate and the manual-clawback agreement."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "2021-03-17 wiki page, never updated after SF-2287; describes the removed block as current policy."
notes: "Company-document refs are relative to the reef root; code refs are prefixed with the owning repo and relative to that repo's root."
---

# Order Service

## Overview

order-service is 셀플로우's order system. The README describes its scope as 주문 접수, 조회, 취소를 담당한다 ("it is responsible for order intake, lookup and cancellation") — though only lookup and cancellation are actually implemented in this repository (`order-service:README.md`, `order-service:src/main/java/kr/co/sellflow/order/`).

It is a Spring Boot 2.3 application on Java 8, persisting to a MySQL schema named `sellflow_order` and migrating it with Flyway. What makes it interesting is less the code than its boundaries: its database instance is shared with the settlement system, one of its order states is written by somebody else's batch job, and its most recent architectural addition — an outbox table — exists precisely because those boundaries were never tightened.

This artifact is the root of the order knowledge graph. Go from here to [[SCH-ORDER]] and [[SCH-ORDER-MIGRATION-HISTORY]] for the data model and how it drifted, [[API-ORDER]] for the endpoint surface (and the large gap between the documented and the real one), [[PROC-ORDER-ORDER-MST-LIFECYCLE]] / [[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]] / [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] for the three entity lifecycles, [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] for the decision that reshaped the domain, and [[RISK-ORDER]] / [[RISK-ORDER-DISABLED-TESTS]] for what is broken or abandoned.

## Key Facts

- Runs Spring Boot 2.3.12.RELEASE on Java 8 (`sourceCompatibility = '1.8'`), and every CI and CD workflow provisions `java-version: '8'` → order-service:build.gradle, order-service:.github/workflows/ci.yml
- Declared dependencies are limited to spring-boot-starter-web, spring-boot-starter-data-jpa, flyway-core, mysql-connector-java and spring-boot-starter-test — no Spring Security, no springdoc, no messaging client → order-service:build.gradle
- Listens on port 8081 with no context path; the only external caller found in any repo calls `/api/v1/orders/{ordNo}/cancel`, while `OrderController` maps `/orders/{ordNo}/cancel` → order-service:src/main/resources/application.yml, delivery-bff:src/generated/orderApi.ts
- Persists to MySQL schema `sellflow_order` with `jpa.hibernate.ddl-auto: none`, so the schema is owned entirely by Flyway, which reads `classpath:db/migration` (V1 through V24) → order-service:src/main/resources/application.yml
- The settlement system shares the same MySQL instance by a 2019 decision — 주의: sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정) ("note: it uses the same instance as sellflow_order (2019 consolidation decision)") → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
- CORS is restricted to a single origin, `https://admin.sellflow.co.kr`, and only for the `/api/**` path pattern — which does not cover `OrderController`'s `/orders/**` mapping at all → order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java
- Only two `@RestController` classes exist in the whole repository: `OrderController` (one POST) and `OrderSearchController` (one GET) → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- Four Spring profiles are configured (dev, qa, stage, prod), each with its own datasource; prod sets a Hikari pool of 40 and a 3000 ms connection timeout → order-service:src/main/resources/application-prod.yml
- Tests do not run anywhere. CI builds with `./gradlew clean build -x test` and the Test step is commented out with the note 테스트 켜야 함. OrderCancelServiceTest 가 로컬 DB 를 타서 CI 에서 깨짐 ("need to turn tests on; OrderCancelServiceTest hits a local DB and breaks in CI"), dated 2023-05-11; all four CD workflows also pass `-x test`, while codecov.yml still demands a 40% project target → order-service:.github/workflows/ci.yml, order-service:codecov.yml
- Three of the four CD workflows — dev, qa and stage — all trigger on `push: branches: [ develop ]`, so a single merge to develop deploys the same SHA to three environments at once; prod deploys on push to main → order-service:.github/workflows/order-service-dev-cd.yml, order-service:.github/workflows/order-service-prod-cd.yml
- The declared version disagrees with itself: SERVER_VERSION says 2.8.14, build.gradle says 2.8.4 → order-service:SERVER_VERSION
- The cancel block for settled orders was removed by SF-2287: `OrderCancelService` now blocks only `CHWISO` and `BANPUM`, with the Javadoc 이미 취소되었거나 반품 프로세스로 넘어간 주문만 차단한다 ("only orders already cancelled or moved into the return process are blocked") → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- `OrderCancelServiceV1`, the pre-2.8.0 cancel implementation that still contains the settlement check, is `@Deprecated`, carries no Spring stereotype annotation, and has zero callers — a repo-wide grep finds the class name only in its own declaration and in its `@Disabled` test → order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- JPA auditing and transaction management are enabled globally through a bare `@Configuration` class, yet no entity carries `@CreatedDate` or `@LastModifiedDate` — the auditing has nothing to audit → order-service:src/main/java/kr/co/sellflow/order/config/JpaConfig.java
- Database migrations require review by both 주문팀 and the DBA group, and `legacy/` is separately pinned to 주문팀 → order-service:.github/CODEOWNERS
- `OrderSearchService`'s RowMapper is `(rs, i) -> null`, so every admin search returns a list of nulls regardless of the query, and the class's `FIXME(은영) 2024-05` note records a CS complaint about exactly that screen without connecting it to the mapper → order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java

## Responsibilities

**What the code actually does**

| Responsibility | Entry point | Notes |
|---|---|---|
| Cancel an order | `OrderController.cancel` → `OrderCancelService.cancel` | The one write path in the service. Loads `ORDER_MST`, rejects `CHWISO`/`BANPUM`, inserts `ORDER_CANCEL`, flips status, publishes to the outbox — all in one `@Transactional`. See [[PROC-ORDER-CANCEL]]. |
| Search orders for CS admin | `OrderSearchController.search` → `OrderSearchService.search` | Javadoc claims 기간 최대 3개월 ("maximum period three months") but no code enforces it; the RowMapper returns null for every row. |
| Look up one order and its items | `OrderQueryService` | No controller calls it — it is reachable only from `OrderQueryServiceTest`. |
| Own the `sellflow_order` schema | `src/main/resources/db/migration/` | 24 Flyway migrations, two of which cancel each other out (V15/V16). See [[SCH-ORDER-MIGRATION-HISTORY]]. |
| Publish `order.cancelled` | `OrderEventPublisher.publishOrderCancelled` | Writes one `ORDER_EVENT_OUTBOX` row with a hand-formatted JSON payload. See [[DEC-ORDER-OUTBOX-RELAY]]. |
| Translate domain errors to HTTP | `GlobalExceptionHandler` | Two handlers only: `OrderNotFoundException` → 404, `OrderCancelNotAllowedException` → 409. See [[PROC-ORDER-ERROR-HANDLING]]. |

## Does NOT Own

- **Order creation.** No code path inserts into `ORDER_MST`. A grep for `INSERT INTO ORDER_MST` across all five sellflow repos returns nothing; the only writes are the cancel path's UPDATE and settlement-batch's `MarkSettledTasklet` UPDATE. Whatever creates orders is outside the sellflow source tree entirely.
- **The `JUNGSAN_WANRYO` state.** settlement-batch updates `ORDER_MST` directly — 정산에 포함된 주문의 상태를 정산완료로 갱신한다 ("updates the status of orders included in settlement to settlement-complete") — and the README confirms 정산 관련 상태(`JUNGSAN_WANRYO`)는 settlement-batch 가 갱신한다. See [[CON-ORDER-SETTLEMENT]].
- **Inventory restoration.** `InventoryClient.restore` exists and has no caller anywhere; it would also POST to `/inventory/restore`, a path inventory-api does not expose (it exposes `/stock/restock` and `/stock/{sku}`). See [[CON-ORDER-INVENTORY]].
- **Payment reversal.** `PaymentClient.cancel` exists and has no caller. Its own Javadoc draws the line that matters: 결제 취소와 정산 차감은 별개다. PG 취소가 성공해도 파트너에게 이미 지급된 정산 금액은 되돌아오지 않는다 ("payment cancellation and settlement deduction are separate; even if the PG cancellation succeeds, settlement money already paid to the partner does not come back").
- **The settlement clawback.** The compensating correction agreed in SF-2287 is Settlement's, not this service's. order-service's entire obligation is one outbox row.
- **Authentication.** There is no Spring Security dependency and no filter. The only statement on the subject is a WebConfig comment. See [[PROC-ORDER-AUTH]].
- **Cancel-reason cost attribution.** `CancelReason`'s Javadoc is explicit: 사유별 비용 부담 주체는 코드에 정의하지 않는다. 정산 정책은 재무본부 소관이며 별도 기준표를 따른다 ("which party bears the cost per reason is not defined in code; settlement policy belongs to the Finance division and follows a separate reference table").
- **Status history for settlement transitions.** `OrderStatusHistory` carries the note 정산완료 전이는 여기 안 쌓인다. 배치에서 직접 UPDATE 하기 때문 ("the settlement-complete transition is not recorded here, because the batch UPDATEs directly").

## Core Concepts

**Romanised Korean naming.** Columns and fields are romanised Korean (`ORD_NO`, `SANGTAE_CD`, `CHWISO_SAYU_CD`, `CHONG_GEUMAEK`), not English. The same concept is spelled differently in neighbouring services; inventory-api states this outright — order-service 와 용어가 다르다. 주문 도메인의 SANGPUM_CD 는 여기서 sku 다. ("the vocabulary differs from order-service; the order domain's SANGPUM_CD is sku here"). See [[PAT-SELLFLOW-ROMANISED-NAMING]] and [[GLOSSARY-ORDER]].

**A shared database, not a service boundary.** The 2019 consolidation decision means settlement reads and writes order tables directly. The consequences are visible throughout: V15 renamed `ORDER_CANCEL.BIGO` to `MEMO` and V16 reverted it at most 18 days later (V15's comment gives only `2024-01`; V16's is dated `2024-01-18`) with the note 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애 ("the settlement batch query references BIGO directly, causing an incident"); a status value exists that this service never writes.

**Outbox instead of a broker.** The only asynchronous integration is a database table polled by another service's Quartz job, at ten-minute intervals, in batches of 500. V8's header says it plainly: 정산 배치의 릴레이 잡이 주기적으로 폴링한다. (별도 브로커 없음) ("the settlement batch's relay job polls it periodically (no separate broker)"). See [[DEC-ORDER-OUTBOX-RELAY]] and [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]].

**Fire and forget.** `OrderEventPublisher`'s Javadoc states 구독 측 처리 결과는 본 서비스에서 추적하지 않는다 ("this service does not track the subscriber's processing result"). The outbox row is marked `PUBLISHED_YN='Y'` by the relay whether or not a reconciliation ever follows. This is the seam through which 188M KRW of unreconciled cancellations became invisible to the order side.

**Access control lives elsewhere.** The WebConfig comment — 어드민 도메인만 허용. 앱은 게이트웨이를 통해 들어온다. ("only the admin domain is allowed; the app comes in through the gateway") — is the only statement about authentication anywhere in the code. See [[PROC-ORDER-AUTH]].

## Dependencies

| System | Integration Type | Purpose | Auth Method |
|---|---|---|---|
| MySQL `sellflow_order` | JDBC (HikariCP via Spring Data JPA) | Primary and only datastore; schema owned by Flyway (`ddl-auto: none`). prod pool max 40. | Username from `${DB_USER:sellflow}`; no password in any committed yml — supplied by the environment (`order-service:src/main/resources/application.yml`, `order-service:.env.example`). |
| settlement-batch | Shared database — it writes `ORDER_MST` directly, and polls `ORDER_EVENT_OUTBOX` | Sets `SANGTAE_CD='JUNGSAN_WANRYO'`; relays cancel events into `CANCEL_RECON_QUEUE` | None — same MySQL instance, by the 2019 consolidation decision. No network boundary exists to authenticate. See [[CON-ORDER-SETTLEMENT]]. |
| inventory-api | Outbound HTTP via `RestTemplate` (`InventoryClient`) | Intended stock restoration on cancel | None. No header, token or credential is set; the call is a bare `postForEntity`. **Dead**: no caller, and the path `/inventory/restore` does not exist on inventory-api. See [[CON-ORDER-INVENTORY]]. |
| PG (payment gateway) | Outbound HTTP via `RestTemplate` (`PaymentClient`) | Full-amount payment cancellation | None in code; base URL from `PG_BASE_URL`, defaulting to `https://pg.example.co.kr`. **Dead**: no caller. Partial cancellation unsupported — TODO(성민) 2021-06-02: 부분취소 미지원. 전액 취소만 호출한다. |
| delivery-bff | Inbound HTTP — it calls the cancel endpoint | Customer-app cancel path | None observed. The generated client sends only `Content-Type: application/json`; it was generated 2022-11-08 and still documents the pre-SF-2287 409. See [[CON-ORDER-DELIVERY]]. |
| admin.sellflow.co.kr (CS admin UI) | Inbound HTTP, browser origin | Order search and admin-initiated cancel | CORS allowlist of exactly one origin, on `/api/**` only. No authentication of any kind. See [[PROC-ORDER-AUTH]]. |
| registry.sellflow.co.kr | Container registry, CI only | Image push target for all four environments | GitHub Actions runner credentials; no login step appears in the workflow, so it is implicit in the runner image. |

## Domain Behavior Highlights

- **The outbox is the entire settlement notification.** `OrderCancelService.cancel` ends with `eventPublisher.publishOrderCancelled(...)`, which saves one `ORDER_EVENT_OUTBOX` row inside the same transaction. There is no broker, no retry, no acknowledgement and no dead-letter path. settlement-batch's `OrderEventRelayJob` polls `PUBLISHED_YN='N'` rows every ten minutes, and marks each `'Y'` unconditionally — including rows for which it inserted a `CANCEL_RECON_QUEUE` entry that nothing ever consumes. A new engineer should read the outbox as a hand-off, not a guarantee.
- **SF-2287 removed the cancel block, and nothing replaced it in this repo.** Before 2.8.0 the cancel path counted `SETTLEMENT_DTL` rows and threw 정산 완료된 주문은 취소할 수 없습니다 ("a settled order cannot be cancelled"); that logic now survives only in the deprecated `OrderCancelServiceV1`. The current `CHWISO_BULGA` set is `EnumSet.of(CHWISO, BANPUM)`. The agreed compensation was manual — 정산팀에서 수기로 정정 처리하겠습니다. 차월 정산에서 차감하는 방식 ("Settlement will handle the correction manually, deducting it in the following month's settlement") — sized on the estimate 월 10건 미만 ("under 10 cases a month"). See [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]].
- **The deprecated V1 class is load-bearing only as documentation.** `OrderCancelServiceV1` is annotated `@Deprecated` with the comment 삭제 예정이나 배치에서 참조 가능성이 있어 남겨둠 ("scheduled for deletion but kept because a batch might reference it"), dated 2023-04-21 by 박성민. A repo-wide grep shows no batch references it — and no service, either: the class carries no `@Service`/`@Component`, so its `@Autowired DataSource` would never be injected even if something did call it. Its only remaining function is that the 2021 wiki still describes it accurately while describing current behaviour inaccurately.
- **Romanised column names are a compatibility surface, not a style choice.** Because settlement queries the order tables directly, a column name is part of a cross-team contract. V15 → V16 is the proof: a four-day-old rename caused an incident and was reverted, and TASK.md still lists V15 롤백 정리 (V16 으로 되돌림, 나중에 정리) ("clean up the V15 rollback (reverted by V16, tidy later)") as an open item years on. Never rename an order column without checking settlement-batch.
- **Three status-writing paths exist and only one is validated.** `OrderMst.chwiso()` is the domain method (used by the cancel path); `OrderStatusService.change` is a generic mutator calling `OrderMst.byeongyeong`, with no transition validation, whose Javadoc warns 정산완료(JUNGSAN_WANRYO) 전이는 이 클래스를 거치지 않는다 ("the settlement-complete transition does not pass through this class"); and settlement-batch's SQL UPDATE is the third. `OrderStatus`'s Javadoc points at an `OrderStatusValidator` that does not exist in the repository. See [[PROC-ORDER-ORDER-MST-LIFECYCLE]].

## Runtime Components

| Component | Tech Stack | Purpose | Entry Point |
|---|---|---|---|
| Application bootstrap | Spring Boot 2.3.12 / Java 8, packaged into a container image built by the CD workflows | Single process, port 8081, profile selected by `SPRING_PROFILES_ACTIVE` | `order-service:src/main/java/kr/co/sellflow/order/OrderServiceApplication.java` |
| Cancel API | Spring MVC `@RestController` | `POST /orders/{ordNo}/cancel` with body `{sayuCd, bigo}`; returns 200 empty | `order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java` |
| Admin search API | Spring MVC `@RestController` + `JdbcTemplate` | `GET /api/v1/orders/search?from&to&sangtaeCd` — the only endpoint under the CORS-covered `/api/**` prefix | `order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchController.java` |
| Error translation | `@RestControllerAdvice` | Maps `OrderNotFoundException` → 404 and `OrderCancelNotAllowedException` → 409, each as `{"message": ...}` | `order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java` |
| Persistence layer | Spring Data JPA, Hibernate `MySQL5InnoDBDialect`, six `JpaRepository` interfaces (five under `repository/`, plus `OrderEventOutboxRepository` under `event/`) | Entity access for `ORDER_MST`, `ORDER_DTL`, `ORDER_CANCEL`, `ORDER_DELIVERY`, `ORDER_STATUS_HIST`, `ORDER_EVENT_OUTBOX` | `order-service:src/main/java/kr/co/sellflow/order/repository/`, `order-service:src/main/java/kr/co/sellflow/order/event/OrderEventOutboxRepository.java` |
| Schema migrator | Flyway (`flyway-core`, `locations: classpath:db/migration`) | Runs V1–V24 at startup; the sole owner of the schema since `ddl-auto: none` | `order-service:src/main/resources/db/migration/` |
| Event outbox writer | Spring `@Component` + JPA, `javax.transaction.Transactional` | Writes one `order.cancelled` row per cancel, payload built by `String.format` | `order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java` |
| Outbound HTTP clients | `RestTemplate` instantiated per component, base URL from environment variables | Inventory restock and PG cancellation — both present, both uncalled | `order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java`, `order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java` |
| Shared utilities | Plain Java statics | `DateUtil` (settlement base date = yesterday; `SimpleDateFormat`, explicitly noted as not thread-safe), `MoneyUtil` (HALF_UP, against settlement's FLOOR) | `order-service:src/main/java/kr/co/sellflow/order/common/` |
| Legacy cancel path | Raw JDBC, manual transaction handling | Pre-2.8.0 cancel including the `SETTLEMENT_DTL` check; unregistered as a bean, uncalled | `order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java` |
| CI / CD | GitHub Actions on ubuntu-latest, temurin 8 | `ci.yml` builds without tests; four CD workflows build, push to `registry.sellflow.co.kr` and run `./deploy.sh` (absent from the repo); `migration-issue.yml` is a manual `flywayInfo` check labelled 임시 ("temporary") | `order-service:.github/workflows/` |

## Related

- [[API-ORDER]] — the endpoint surface, documented versus real
- [[SCH-ORDER]] — tables, entities and their mismatches
- [[SCH-ORDER-MIGRATION-HISTORY]] — all 24 Flyway migrations and where they drifted from the schema
- [[PROC-ORDER-ORDER-MST-LIFECYCLE]] — the order master lifecycle and its three status writers
- [[PROC-ORDER-ORDER-CANCEL-LIFECYCLE]] — the cancel record entity
- [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] — the outbox row from write to relay
- [[PROC-ORDER-CANCEL]] — the cancel flow and its missing side effects
- [[PROC-ORDER-ERROR-HANDLING]] — the two handled exceptions and everything that falls through
- [[PROC-ORDER-AUTH]] — what authentication exists (very little)
- [[GLOSSARY-ORDER]] — romanised Korean domain vocabulary
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — why the naming is a cross-team contract
- [[RISK-ORDER]] — dead code, thread-unsafe utilities, broken search
- [[RISK-ORDER-DISABLED-TESTS]] — the disabled tests and the CI step that never runs
- [[DEC-ORDER-OUTBOX-RELAY]] — why the outbox exists and why there is no broker
- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — removing the settled-order cancel block
- [[CON-ORDER-SETTLEMENT]] — the shared-database boundary with settlement-batch
- [[CON-ORDER-INVENTORY]] — the restock boundary that is not wired up
- [[CON-ORDER-DELIVERY]] — the delivery lookup boundary
