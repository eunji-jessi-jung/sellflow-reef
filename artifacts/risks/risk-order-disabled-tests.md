---
id: "RISK-ORDER-DISABLED-TESTS"
type: "risk"
title: "Order Service Disabled and Non-Executing Tests"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Compiled on 2026-09-19 by reading all five files under order-service/src/test in full, codecov.yml, build.gradle and all six workflow files, then grepping the test tree for every production type to establish what is never touched. This answers Q-012. It becomes stale the moment the commented-out Test step in ci.yml is uncommented — that single line is the hinge for almost every finding here. Re-checked on 2026-09-19 after a correction pass in order-service: OrderStatusService now compiles against OrderMst.byeongyeong, so finding 8 is no longer about an unresolvable call — it is about a test that asserts the lookup and not the transition."
freshness_triggers:
  - ".github/workflows/ci.yml"
  - ".github/workflows/order-service-prod-cd.yml"
  - "build.gradle"
  - "codecov.yml"
  - "src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java"
  - "src/test/java/kr/co/sellflow/order/search/OrderSearchServiceTest.java"
  - "src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
  - "src/test/java/kr/co/sellflow/order/service/OrderQueryServiceTest.java"
  - "src/test/java/kr/co/sellflow/order/service/OrderStatusServiceTest.java"
known_unknowns:
  - "Whether the suite passes at all when run locally. The two non-disabled tests in OrderCancelServiceTest assert against hardcoded order numbers (NOT_EXIST, ORD20230411001) that must already be in a reachable database, and no fixture, SQL seed, Testcontainers config or @Sql annotation exists in the repo."
  - "Whether tests run anywhere outside these workflows — a nightly job, a Jenkins pipeline, a developer's pre-push hook. The absence recorded here is the absence of any test execution in the repository's own CI and CD definitions."
  - "Why 정상_취소 was disabled on 2023-05-11 specifically, three weeks after SF-2287 shipped on 2023-04-21. The comment says the test broke CI; whether the SF-2287 change is what broke it is not recorded anywhere, and no PR reference is given."
  - "Whether codecov is wired up at the organisation level. codecov.yml sets a 40% target, but no workflow uploads a report and build.gradle applies no coverage plugin, so nothing in this repository can produce the input it grades."
  - "What 박성민's current relationship to this repository is. The CI TODO, the legacy deprecation note and the V1 test TODO are all signed by him between 2021 and 2023; the org chart was not consulted for this artifact."
severity: "high"
resolution: "unresolved"
tags:
  - "order"
  - "risk"
  - "tests"
  - "ci"
  - "coverage"
  - "q-012"
aliases:
  - "order disabled tests"
  - "Q-012"
relates_to:
  - type: "refines"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[PROC-ORDER-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[RISK-ORDER]]"
  - type: "feeds"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/ci.yml"
    notes: "The Test step is commented out; build runs -x test."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-prod-cd.yml"
    notes: "Production deploy also builds with -x test."
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "No jacoco plugin; spring-boot-starter-test is the only test dependency."
  - category: "implementation"
    type: "github"
    ref: "order-service:codecov.yml"
    notes: "40% project target, 5% threshold, no producer of coverage data."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The policy change whose behaviour the TODO says still needs a test."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
    notes: "Never referenced by any test."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The behaviour under (non-)test."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java"
    notes: "Class-level @Disabled; the single test body is comments only."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/search/OrderSearchServiceTest.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
    notes: "Method-level @Disabled on the only happy-path cancel test."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/service/OrderQueryServiceTest.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/service/OrderStatusServiceTest.java"
notes: "Answers Q-012. Severity is high rather than medium — unlike the rest of RISK-ORDER, this one is not a latent smell: the write path that moves money-bearing state has demonstrably not been exercised by automation since 2023-05-11, and the production deploy workflow is the same build that skips it."
---

# Order Service Disabled and Non-Executing Tests

## Description

Q-012 asks which order-service tests are disabled and what behaviour is therefore unverified in CI. The literal answer is two: one method and one class carry `@Disabled`. The useful answer is all of them.

The repository contains five test classes holding eight test methods. Two of those methods are switched off by annotation. The remaining six are never executed either, because the only step that would run them — `- name: Test / run: ./gradlew test` in `.github/workflows/ci.yml` — is commented out, and every build in the repository, including the four CD workflows that deploy to dev, qa, stage and production, runs `./gradlew clean build -x test`. Nothing in this repository has run a test since the day that step was commented out, 2023-05-11.

Two further layers sit under that. The class that *is* disabled by annotation would pass vacuously if enabled, because its only test method has an empty body. And the coverage gate that would have caught the gap — `codecov.yml`, target 40% — has no producer: `build.gradle` applies no JaCoCo plugin and no workflow uploads a report.

The date matters. SF-2287 shipped on 2023-04-21, removing the block on cancelling already-settled orders. Twenty days later the happy-path cancel test was disabled and CI's test step was commented out. The behaviour SF-2287 introduced has never had a test at all — the class's last line is a TODO saying so — and the behaviour it modified stopped being checked three weeks after the change.

## Key Facts

- The CI workflow's build step is `./gradlew clean build -x test`, and the only test step in the file is commented out → .github/workflows/ci.yml
- The reason is recorded verbatim in place: "TODO 테스트 켜야 함. OrderCancelServiceTest 가 로컬 DB 를 타서 CI 에서 깨짐 - 2023-05-11 박성민" ("TODO: the tests must be turned on. OrderCancelServiceTest hits the local DB and breaks in CI") → .github/workflows/ci.yml
- All four CD workflows — dev, qa, stage and prod — use the identical `-x test` build, so no deployment to any environment is gated by a test → .github/workflows/order-service-prod-cd.yml
- `OrderCancelServiceTest.정상_취소` ("normal cancellation"), the only happy-path cancel test, is `@Disabled("로컬 DB 필요. CI 에서 깨져서 임시 비활성화 - 2023-05-11")` ("a local DB is required; temporarily disabled because it breaks in CI") — the word 임시 means *temporary*, and it has now stood for three years and four months → src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java
- The same class's final line is an unactioned note: "// TODO 정산 완료 주문 취소 케이스 테스트 필요" ("a test is needed for the settled-order cancellation case") — precisely the behaviour SF-2287 introduced → src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java
- The whole `OrderCancelServiceV1Test` class carries `@Disabled("SF-2287 이후 정책 변경. legacy 클래스 제거 시 함께 삭제")` ("policy changed after SF-2287; delete together with the legacy class") → src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java
- Its javadoc restates the reasoning — "SF-2287 이후 정산 완료 주문도 취소가 허용되면서 이 테스트는 현재 정책과 맞지 않는다" ("since SF-2287 allowed settled orders to be cancelled too, this test no longer matches the current policy") — and carries "TODO(성민) 2023-04-21: legacy 제거 시 같이 삭제" ("delete along with the legacy removal") → src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java
- That class's single method `정산완료_주문은_취소할_수_없다` ("a settled order cannot be cancelled") has an empty body — three given/when/then comments and no statement — so enabling it would produce a passing test that verifies nothing → src/test/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1Test.java
- No JUnit 4 `@Ignore`, no `assumeTrue` guard and no `@Tag`-based exclusion exists anywhere in the test tree; the suite is JUnit 5 throughout and the only skip mechanism used is `@Disabled` → src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java
- The two non-disabled tests in `OrderCancelServiceTest` are `@SpringBootTest` integration tests asserting against hardcoded order numbers `NOT_EXIST` and `ORD20230411001`, with no fixture, `@Sql`, embedded database or Testcontainers configuration anywhere in the repo — so they share the disabled test's dependency on database contents → src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java
- No test in the repository references `OrderEventPublisher`, `OrderEventOutbox`, `InventoryClient`, `PaymentClient`, `GlobalExceptionHandler`, `CancelReason`, `OrderMapper`, `DateUtil` or `MoneyUtil`; a grep of `src/test` for each returns nothing → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- `codecov.yml` sets a project target of 40% with a 5% threshold, while `build.gradle` applies no JaCoCo plugin and no workflow contains a codecov upload step — the gate has no input and has never been evaluated → codecov.yml
- `OrderSearchServiceTest`'s single test asserts only that `jdbcTemplate.query` was invoked, never inspecting the return value, so the `(rs, i) -> null` RowMapper that makes every search result null is invisible to it → src/test/java/kr/co/sellflow/order/search/OrderSearchServiceTest.java
- `OrderStatusServiceTest.상태를_변경한다` ("changes the status") verifies `repo.findById("ORD1")` and nothing about the resulting status, so it would pass whatever `OrderMst.byeongyeong` did — including nothing at all → src/test/java/kr/co/sellflow/order/service/OrderStatusServiceTest.java

## Findings

One row per test method in the repository. "Runs in CI" is `no` for every row, because no CI or CD workflow executes tests at all.

| # | Test | Disable mechanism | Reason, verbatim (meaning) | Runs in CI | Behaviour left unverified |
|---|---|---|---|---|---|
| 1 | `OrderCancelServiceTest.정상_취소` ("normal cancellation") | method `@Disabled` | "로컬 DB 필요. CI 에서 깨져서 임시 비활성화 - 2023-05-11" ("a local DB is required; temporarily disabled because it breaks in CI") | no | **The entire cancel happy path.** That `ORDER_CANCEL` is inserted with `CHWISO_SAYU_CD` and `CHORI_SANGTAE='COMPLETED'`, that `ORDER_MST.SANGTAE_CD` becomes `CHWISO` with `UPD_DTM` set, and that an `order.cancelled` row lands in `ORDER_EVENT_OUTBOX`. This is the service's only write path. |
| 2 | `OrderCancelServiceV1Test.정산완료_주문은_취소할_수_없다` ("a settled order cannot be cancelled") | class `@Disabled` | "SF-2287 이후 정책 변경. legacy 클래스 제거 시 함께 삭제" ("policy changed after SF-2287; delete together with the legacy class") | no | Nothing — and it would verify nothing if enabled: the method body is three comments (`// given: SETTLEMENT_DTL 에 해당 주문이 존재`, `// when: cancel 호출`, `// then: IllegalStateException`) with no arrange, act or assert. The legacy class's contradictory policy is therefore neither tested nor removed. |
| 3 | `OrderCancelServiceTest.존재하지_않는_주문은_예외` ("a non-existent order throws") | none — blocked by CI only | CI runs `-x test` | no | That an unknown order number yields `OrderNotFoundException` → HTTP 404. Also depends on `NOT_EXIST` being genuinely absent from whatever database the `@SpringBootTest` context connects to. |
| 4 | `OrderCancelServiceTest.이미_취소된_주문은_예외` ("an already-cancelled order throws") | none — blocked by CI only | CI runs `-x test` | no | The `CHWISO_BULGA` guard, i.e. that CHWISO and BANPUM are rejected with 409. It asserts against the hardcoded `ORD20230411001` and only covers CHWISO; **BANPUM is covered by no test at all.** |
| 5 | `OrderSearchServiceTest.상태코드가_없으면_파라미터_두개로_조회한다` ("queries with two parameters when no status code is given") | none — blocked by CI only | CI runs `-x test` | no | Anything about results. It stubs and then verifies the same `jdbcTemplate.query` call, so it cannot see the null RowMapper, the `ORD_DT` column that no migration creates, or the unenforced three-month range limit. |
| 6 | `OrderQueryServiceTest.주문이_없으면_예외` ("throws when the order is absent") | none — blocked by CI only | CI runs `-x test` | no | Mock-only; exercises a service with no production caller. |
| 7 | `OrderQueryServiceTest.주문을_반환한다` ("returns the order") | none — blocked by CI only | CI runs `-x test` | no | Mock-only; asserts non-null on a `new OrderMst()`. |
| 8 | `OrderStatusServiceTest.상태를_변경한다` ("changes the status") | none — blocked by CI only | CI runs `-x test` | no | **All status transitions.** It verifies the repository lookup, not the transition: neither the value `byeongyeong` assigns, nor the `save`, nor the silent `ifPresent` no-op on a missing order is asserted. |
| — | *(absent)* settled-order cancellation | never written | "TODO 정산 완료 주문 취소 케이스 테스트 필요" ("a test is needed for the settled-order cancellation case") | n/a | The behaviour SF-2287 shipped in 2023 — that an order in `JUNGSAN_WANRYO` **can** now be cancelled, and that the resulting outbox event is what settlement will later have to claw back. Never tested in any form. |
| — | *(absent)* outbox write | never written | no comment | n/a | No test references `OrderEventPublisher` or `OrderEventOutbox`. The payload's hand-formatted JSON, the `PUBLISHED_YN='N'` default and the transactional coupling to the cancel are unverified end to end. |
| — | *(absent)* error mapping | never written | no comment | n/a | No test references `GlobalExceptionHandler` or `CancelReason`. That an invalid `sayuCd` produces a 500 rather than the spec's 400 ([[PROC-ORDER-ERROR-HANDLING]]) would have been caught by one. |

## Impact

**The cancel path is simultaneously the riskiest and the least-verified code in the estate.** It is the only write path in order-service; it is the origin of the `order.cancelled` event that feeds the settlement correction chain; and since SF-2287 it can act on orders whose money has already been paid out to a partner. Row 1 above is the test that would exercise it, and it has been off since 2023-05-11.

**The disabling reason has outlived its own framing.** "임시 비활성화" — *temporarily* disabled — is the language of a one-sprint workaround. The stated cause is environmental (the test needs a local database), and the fix is well understood in 2026: an embedded database, Testcontainers, or a slice test with a mocked repository. Nothing in the repo suggests the problem was ever re-examined.

**The safety net is doubly absent.** Even a team that re-enabled the two non-disabled `@SpringBootTest` methods would gain little: they assert against order numbers that must pre-exist in a database. And even a passing suite would not be measured, because `codecov.yml`'s 40% target has no producer. The repository presents three artefacts that each imply testing discipline — a test directory, a coverage configuration, and a CI workflow — while executing none.

**The gap aligns exactly with the estate's known money problem.** `CANCEL_RECON_QUEUE` holds 4,127 unprocessed rows worth 188,851,520 KRW ([[RISK-SETTLEMENT-RECON-BACKLOG]]), every one of them originating in a cancel event this service emits. The correctness of the emission — one event, right order number, right reason code, exactly once per cancel — is verified by no automated test anywhere in order-service.

**A regression would reach production unimpeded.** The path from a merge to `main` to a running production container is `./gradlew clean build -x test` → `docker build` → `./deploy.sh prod`. There is no test gate, no coverage gate, and no manual approval step in the workflow.

## Severity and Resolution

**Severity: high.** The brief's density rule puts this over the line on count alone — twelve distinct signals were found (two `@Disabled` markers, one commented-out CI step in ci.yml plus the same `-x test` build in all five building workflows of six, one vacuous test body, two hardcoded-order-number dependencies, one orphaned coverage config, one missing JaCoCo plugin, one never-written settled-cancel test, and three whole production areas with no test reference at all). But the rating does not rest on density alone: unlike most of [[RISK-ORDER]], this is not a latent code smell. It is a verified, dated, three-year absence of automated verification over the one code path that moves money-bearing state, on a deploy pipeline that runs the same test-skipping build straight to production. The mitigating facts — the service is small, the cancel logic is short, and no incident in the sources is attributed to an order-service regression — are why this is high and not critical.

**Resolution: unresolved.** Every item is open, and the repository's own record of the intention is `TASK.md`, a file whose footer reads "※ 개인 메모입니다. 공식 문서 아님." ("this is a personal memo, not an official document") — and which does not list enabling the tests among its four open items at all. The CI TODO is the only place the intention is written down.

## Recommended Actions

1. **Uncomment the test step in `ci.yml` and see what happens.** Six of the eight methods have no `@Disabled` on them; running them is a single-line change and would establish, for the first time since 2023, whether the suite is green, red, or uncompilable. Do this before anything else, because every other recommendation depends on the answer.
2. **Break `OrderCancelServiceTest`'s database dependency.** Replace the `@SpringBootTest` + hardcoded-order-number arrangement with mocked `OrderMstRepository`/`OrderCancelRepository`/`OrderEventPublisher` collaborators, which removes the stated cause of the 2023 disable and lets `정상_취소` be re-enabled in the same change.
3. **Write the settled-order cancel test the TODO asks for.** It is the behaviour SF-2287 shipped, the origin of every row in `CANCEL_RECON_QUEUE`, and the one case where a bug costs money rather than a customer complaint.
4. **Assert the outbox row.** A cancel that updates `ORDER_MST` without writing `ORDER_EVENT_OUTBOX` is silently invisible to settlement; nothing today would notice.
5. **Delete `OrderCancelServiceV1` and its empty test together**, as both files' own TODOs instruct, or state in writing why the legacy class is still needed. This closes a disabled test and a contradictory cancel policy in one commit.
6. **Either wire up JaCoCo and a codecov upload, or delete `codecov.yml`.** A 40% target that nothing measures is worse than no target, because it reads as a control that is in force.
7. **Put a test gate on the CD workflows**, at minimum on prod. Deploying with `-x test` is a separate decision from CI skipping tests, and nothing in the repo records that it was ever deliberately made.

## Related

- [[SYS-ORDER]] — the service whose suite this is
- [[RISK-ORDER]] — the wider risk inventory, of which this is the highest-severity strand
- [[PROC-ORDER-CANCEL]] — the flow that rows 1, 3 and 4 would have covered
- [[PROC-ORDER-ERROR-HANDLING]] — the error mapping no test touches
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the downstream money consequence of the untested emission
