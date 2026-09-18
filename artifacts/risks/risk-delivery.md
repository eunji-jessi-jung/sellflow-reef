---
id: "RISK-DELIVERY"
type: "risk"
title: "Delivery BFF Known Risks"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Systematic re-scan on 2026-09-19 of all 11 files in delivery-bff (112 lines across src/ and tests/), against the live order-service controller, exception handler, OrderStatus enum and build.gradle, the 2022-11-04 OpenAPI spec, ticket SF-2287, services.yaml and org-chart.md. The stale-client theme closes only when SF-4901 is done; the ownership theme closes only when the registry's TODO is resolved."
freshness_triggers:
  - "README.md"
  - "package.json"
  - "sources/context/org-chart.md"
  - "sources/context/registry/services.yaml"
  - "sources/context/tickets/SF-2287.md"
  - "src/deliveryStatus.ts"
  - "src/generated/orderApi.ts"
  - "src/index.ts"
  - "src/logger.ts"
  - "src/orderClient.ts"
  - "tests/deliveryStatus.test.ts"
  - "tsconfig.json"
severity: "high"
resolution: "open"
known_unknowns:
  - "SF-4901 is referenced only in a code comment in src/orderClient.ts. No ticket file for it exists in sellflow-reef/sources, so its scope, owner, priority and age are unknown beyond the comment's 미착수 (not started)."
  - "Whether the stale fallback has ever caused a production incident is unknown; no postmortem or incident record naming delivery-bff was found, and the only runbook in sources covers the 2025-07 settlement duplicate-run."
  - "Whether CI runs elsewhere — an org-level workflow, a Jenkins job, a shared pipeline — is unknown. The absence recorded here is the absence of any CI configuration inside the repository; note that order-service and settlement-batch both have .github/workflows and this repository has none."
  - "Whether requestCancel is called by anything outside this repository (imported as a library) is unknown; within the repository it has no caller."
  - "Whether any client of GET /delivery/:ordNo reads the stale flag. No consumer of this endpoint exists in any repository in this workspace."
  - "Which team actually operates this service today. services.yaml says TODO, README.md and package.json say 커머스본부 물류팀, org-chart.md assigns delivery-bff to 물류팀 while giving 배송사 관리 (carrier management) to 물류운영본부 배송관리팀 — three documents, three positions."
  - "Severity is assessed from code reading alone, with no traffic volume, carrier error rate or CS enquiry data to calibrate it."
tags:
  - "delivery"
  - "risk"
  - "technical-debt"
  - "generated-code"
  - "testing"
  - "ownership"
aliases:
  - "delivery-bff risks"
  - "배송 BFF 리스크"
relates_to:
  - type: "refines"
    target: "[[API-DELIVERY]]"
  - type: "refines"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-AUTH]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-STATUS-SYNC]]"
  - type: "integrates_with"
    target: "[[RISK-INVENTORY]]"
  - type: "integrates_with"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "Claims a 마지막 저장 값 fallback and names 커머스본부 물류팀 as owner."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "axios + express only; vitest absent from devDependencies; no test script; no CI."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Generated 2022-11-08; relative URL, stale OrderStatus union, pre-SF-2287 409 message, sayuCd: string."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "The hardcoded PREPARING fallback."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/logger.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
    notes: "Comment naming SF-4901 as 미착수."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tests/deliveryStatus.test.ts"
    notes: "The repository's only test asserts expect(true).toBe(true)."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tsconfig.json"
    notes: "strict false; include covers src only, so tests are outside the type-check."
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "No spring-boot-starter-security — the declared bearerAuth is unenforced."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "Route is /orders/{ordNo}/cancel and the 200 body is empty."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
    notes: "The four valid reason codes and the IllegalArgumentException on anything else."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
    notes: "409 means 취소할 수 없는 주문 상태, not settlement."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "CHWISO_BULGA is only CHWISO and BANPUM."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "물류팀 owns delivery-bff; 배송관리팀 owns 배송사 관리 · 라스트마일 운영."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "owner_team: TODO, last_reviewed 2026-03-02, runtime recorded as Node 16."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "Removed the settlement-complete cancel block; fix_version order-service 2.8.0, resolved 2023-04-21."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "The 2022-11-04 spec the client was generated from; servers have no /api/v1 prefix."
notes: "Sixteen findings in 112 lines. The service's defining property is that it cannot report failure: every error path converges on one HTTP 200 whose body is a literal, and the documents describing that literal all describe a cache the service does not have."
---

## Description

delivery-bff is 112 lines across `src/` and `tests/`, which makes its risk profile unusually
legible: the findings below are the complete set, not a sample. Two of them dominate.

The first is a degraded response that lies. Every carrier failure resolves to one hardcoded HTTP
200 body, `{status: 'PREPARING', stale: true}`, which the README, the code comment and the
repository's only test all describe as "마지막 저장 값" — the last stored value. There is no store.

The second is a piece of frozen code: a client generated in November 2022 that still encodes an
order-status vocabulary and a cancellation policy the server abandoned in April 2023, and which
— for four independent reasons — could not successfully call order-service even if it were wired
up. Around these sit the ordinary markers of a service nobody currently owns: one placeholder
test, strict mode off, console logging, no CI, no authentication, three environment variables
that do nothing, and an ownership question three documents answer three ways.

## Key Facts

- Every carrier failure returns HTTP 200 with the hardcoded body `{ ordNo, status: 'PREPARING', stale: true }`; no code path in the service can produce a non-200 status → delivery-bff:src/index.ts, delivery-bff:src/deliveryStatus.ts
- There is no store behind the word "stale": `package.json` declares only `axios` and `express`, nothing in `src/` holds module-level state or writes to disk, and `services.yaml` records `db: none` → delivery-bff:package.json, sellflow-docs:context/registry/services.yaml
- README.md, the `src/index.ts` comment and the test name all assert the non-existent behaviour, e.g. "조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다" ("on lookup failure, retry three times and return the last stored value") → delivery-bff:README.md, delivery-bff:src/index.ts, delivery-bff:tests/deliveryStatus.test.ts
- `PREPARING` belongs to no vocabulary in this estate: the live enum spells 상품준비중 as `SANGPUM_JUNBI` and declares seven values, none of them English → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- The generated client states its own provenance: `generator: openapi-typescript-codegen 0.23.0`, `source: order-service-openapi.json`, `generated: 2022-11-08T04:12:33Z`, from spec version 2.4.0 dated 2022-11-04 → delivery-bff:src/generated/orderApi.ts, sellflow-docs:raw/specs/order-service-openapi.json
- Its path is wrong twice over: it fetches `/api/v1/orders/${ordNo}/cancel`, while the live controller is `@RequestMapping("/orders")` + `@PostMapping("/{ordNo}/cancel")` and the spec's servers carry no `/api/v1` prefix either → delivery-bff:src/generated/orderApi.ts, order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java, sellflow-docs:raw/specs/order-service-openapi.json
- The URL is **relative**, so in Node there is no origin to resolve it against and the call fails before any network attempt → delivery-bff:src/generated/orderApi.ts
- Its response shape never matches: `CancelResponse` expects `{ordNo, sangtaeCd}`, and the live controller returns `ResponseEntity.ok().build()` — an empty 200 body, on which `res.json()` throws → delivery-bff:src/generated/orderApi.ts, order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- The `OrderStatus` union carries `JUMUN_WANRYO`, which the live enum does not declare, and omits `GYEOLJE_WANRYO`, `SANGPUM_JUNBI` and `JUNGSAN_WANRYO`, which it does → delivery-bff:src/generated/orderApi.ts, order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- `cancelOrder` throws `Error('정산이 완료된 주문은 취소할 수 없습니다.')` on HTTP 409, and its doc comment repeats the rule: "취소 요청. 정산 완료 주문은 409 를 반환합니다" → delivery-bff:src/generated/orderApi.ts
- SF-2287 repealed that rule, resolved 2023-04-21 in order-service 2.8.0, its 변경 내역 recording "`OrderCancelService.java` — 정산 상태 확인 로직 제거" ("settlement status check logic removed") → sellflow-docs:context/tickets/SF-2287.md
- Today a 409 means something else: `GlobalExceptionHandler` maps `OrderCancelNotAllowedException` to 409 with the message "취소할 수 없는 주문 상태입니다" ("the order is in a state that cannot be cancelled"), and `CHWISO_BULGA = EnumSet.of(CHWISO, BANPUM)` → order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java, order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- `CancelRequest.sayuCd` is typed as a bare `string`, not a union, although exactly four codes exist and the server throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` on anything else → delivery-bff:src/generated/orderApi.ts, order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java
- Regeneration is tracked only in a code comment: "재생성은 SF-4901 에서 다루기로 함. (미착수)" ("regeneration is to be handled in SF-4901 — not started") → delivery-bff:src/orderClient.ts
- The repository's only test asserts `expect(true).toBe(true)`, imports `vitest` which is absent from `devDependencies`, has no `test` script to run it, and lives outside `tsconfig.json`'s `include: ["src"]` → delivery-bff:tests/deliveryStatus.test.ts, delivery-bff:package.json, delivery-bff:tsconfig.json
- `"strict": false` disables null checks and implicit-any errors across the codebase → delivery-bff:tsconfig.json
- The repository contains no `.github/` directory, no workflow, no Dockerfile and no pipeline configuration of any kind, while order-service and settlement-batch each have several workflows → delivery-bff (whole tree)
- No authentication exists in either direction, and the bearerAuth the order spec declares is not enforced by order-service either — `build.gradle` has no `spring-boot-starter-security` → delivery-bff:src/index.ts, order-service:build.gradle
- Ownership is recorded three different ways: `services.yaml` says `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` ("transfer to 물류팀 under discussion since 2025-11, not confirmed"), README.md and package.json say 커머스본부 물류팀, and org-chart.md assigns delivery-bff to 물류팀 while giving 배송사 관리 · 라스트마일 운영 to 물류운영본부 배송관리팀 → sellflow-docs:context/registry/services.yaml, delivery-bff:README.md, sellflow-docs:context/org-chart.md
- `ORDER_API_BASE`, `CARRIER_API_BASE` and `RETRY_COUNT` are all defined in `.env.template` and read by nothing; the code reads `CARRIER_API`, a name that appears in no template → delivery-bff:.env.template, delivery-bff:src/deliveryStatus.ts
- The repository contains no `TODO` or `FIXME` marker anywhere — a grep finds none; the only deferred-work marker in the codebase is the Korean 미착수 in `src/orderClient.ts`, which no tooling would ever flag → delivery-bff (whole tree)

## Findings

| # | Finding | Evidence | Class | Severity | Detectable today? |
|---|---|---|---|---|---|
| 1 | Carrier failure returns HTTP 200 with a hardcoded `PREPARING`, so delivered and cancelled orders read as "preparing" during an outage | `src/index.ts`, `src/deliveryStatus.ts` | correctness / customer-facing | **high** | No — 200, no error log on exhaustion, no metric |
| 2 | The "마지막 저장 값" the README, the code comment and the test name promise does not exist; there is no store | `README.md`, `src/index.ts`, `tests/deliveryStatus.test.ts`, `package.json`, `services.yaml` | documentation drift | **high** | No — three documents agree with each other and not with the code |
| 3 | `PREPARING` is outside the estate's status vocabulary, which is romanised Korean (`SANGPUM_JUNBI`) | `src/index.ts`, `order-service:…/OrderStatus.java` | integration | **medium** | Only by a consumer's unmapped-value branch |
| 4 | Generated client calls `/api/v1/orders/…` — a prefix neither the live controller nor the spec's servers have | `src/generated/orderApi.ts`, `…/OrderController.java` | integration | **high** | Yes, on first call — 404 |
| 5 | The fetch URL is relative, so it cannot resolve in Node at all | `src/generated/orderApi.ts` | correctness | **high** | Yes, on first call — throws before the network |
| 6 | `CancelResponse` shape is never produced: the live controller returns an empty 200 body, so `res.json()` throws | `src/generated/orderApi.ts`, `…/OrderController.java` | integration | **high** | Yes, on first successful call |
| 7 | `OrderStatus` union has a value the server dropped and lacks three it has, with `strict: false` suppressing complaints | `src/generated/orderApi.ts`, `…/OrderStatus.java`, `tsconfig.json` | type safety | **medium** | No |
| 8 | The 409 branch encodes the pre-SF-2287 settlement policy; today 409 means the order is already cancelled or returned | `src/generated/orderApi.ts`, `SF-2287.md`, `GlobalExceptionHandler.java` | business-rule drift | **high** | No — a wrong message, not an error |
| 9 | `sayuCd` typed as free `string` although exactly four codes exist; an invalid one reaches the server and raises `IllegalArgumentException` | `src/generated/orderApi.ts`, `CancelReason.java` | type safety | **medium** | Only server-side, as a 500 |
| 10 | The only test asserts nothing, names behaviour that does not exist, and cannot be run — vitest is not a devDependency and there is no test script | `tests/deliveryStatus.test.ts`, `package.json` | testing | **high** | No |
| 11 | No CI, no workflow, no Dockerfile — nothing builds, lints or tests this service on change | whole tree; contrast order-service `.github/workflows` | process | **medium** | No |
| 12 | No authentication inbound or outbound, and order-service does not enforce the bearerAuth its spec declares | `src/index.ts`, `order-service:build.gradle` | security | **high** | No |
| 13 | Ownership unsettled across three documents: registry `TODO`, README 물류팀, org-chart splitting delivery-bff from 배송사 관리 | `services.yaml`, `README.md`, `org-chart.md` | governance | **high** | Yes, on reading — and unresolved since 2025-11 |
| 14 | `.env.template`'s three variables are all inert; the code reads `CARRIER_API`, a name that appears in no template | `.env.template`, `src/deliveryStatus.ts` | configuration | **medium** | No — an operator can believe the carrier host is configured |
| 15 | `strict: false`, `include: ["src"]` excluding tests, and a console-based logger that the project's own `no-console: warn` rule flags | `tsconfig.json`, `src/logger.ts`, `.eslintrc.json` | hygiene | **low** | Yes, on lint — if lint were run |
| 16 | No `TODO`/`FIXME` anywhere; deferred work is recorded in Korean prose (미착수) that no tool can find | whole tree, `src/orderClient.ts` | process | **low** | No — and that is the finding |

## Impact

**Customer-facing, and silent.** Finding 1 is the one that reaches people. A carrier outage does
not produce an error, an alert or an error-level log line — it produces correct-looking HTTP 200
responses telling every customer their order is being prepared. A customer holding a delivered
parcel contacts CS about a shipment that has not moved; a customer who cancelled sees their order
apparently still in fulfilment. SF-2287 measured what this class of confusion costs: 214 CS
enquiries in March 2023 from a cancellation message customers could not act on, falling to about
3 a week after the fix. The delivery fallback is the same shape of problem — a system telling
customers something untrue about their own order — with the difference that nothing here logs the
moment it happens.

**The documentation actively misleads.** Finding 2 is worse than a stale README, because three
independent artefacts agree with each other and not with the code: the README, the inline comment
and the test name all describe a cache. An engineer triaging "why did this order show PREPARING?"
will look for the store, not find it, and — reading the comment's 없으면 준비중으로 표기 ("if
there is none, mark as preparing") — may well conclude the cache missed rather than that it never
existed. This is the estate's documented pattern, catalogued in [[RISK-SELLFLOW-DOC-DRIFT]]:
the authoritative description and the running code diverged and only the description was
maintained.

**The cancel client is inert and, if revived, wrong four ways.** Findings 4, 5, 6 and 8 compound:
the relative URL prevents the call from being made, the `/api/v1` prefix would 404 if it were, the
empty 200 body would throw if it reached the right route, and the 409 message would misreport the
one error it does handle. `requestCancel` has no caller in this repository, so the present impact
is latent. But the latency is the danger: the next engineer to wire up cancellation inherits four
failures at once, with `strict: false` guaranteeing the compiler stays quiet about the shape
mismatches.

**Nothing would catch a regression.** With one empty test that cannot be run (finding 10) and no
CI (11), every behaviour described above is protected by nothing. The test that would have caught
finding 2 exists, is named for exactly that behaviour, and asserts `true`.

**No named owner is obliged to act.** Finding 13 is why the rest persist. `services.yaml` — the
file that opens by declaring itself 단일 기준, the single standard — has said `owner_team: TODO`
since at least its last review on 2026-03-02, with the note that transfer to 물류팀 has been under
discussion since 2025-11. The README and package.json assert 물류팀 anyway. And the org chart
adds a third position: 물류팀 (커머스본부, 6 people, 이지훈) is listed against delivery-bff, while
배송사 관리 · 라스트마일 운영 — carrier management and last-mile operations, which is what this
service actually proxies — belongs to 물류운영본부 배송관리팀 (15 people, 권나래). So the team
that owns the code and the team that owns the carrier relationship are in different 본부
(divisions), which is also why finding 14's missing carrier credential has no obvious home.

**Security.** With no authentication anywhere ([[PROC-DELIVERY-AUTH]]), anyone who can reach port
8083 can enumerate delivery status by order number, unthrottled and unlogged. The order side
offers no compensating control: its spec declares JWT bearer auth and its build declares no
security dependency.

## Severity and Resolution

**Severity: high.** Raised from medium at this depth. The justification is density and
convergence, not any single finding.

Sixteen findings sit in 112 lines of source. `src/index.ts` is 17 lines and carries three of them;
`src/generated/orderApi.ts` is 38 lines and carries six. That is roughly one finding per seven
lines. As with [[RISK-INVENTORY]], density matters here because the defects are not independent —
they converge on a single property: **this service cannot report that anything is wrong.** The
route has no error branch; the retry loop swallows every exception class; the final give-up logs
at WARN and not ERROR; the degraded response is HTTP 200; nothing counts stale responses; there is
no alert, and the source says so in as many words (별도 알림은 없다); there is no CI; and the one
test asserts nothing. Under a total carrier outage, the observable signals available to the
company are: a slower endpoint, and some warning lines in container output.

What tips it past medium is that the failure is customer-facing and the misinformation is
specific. This is not a service degrading to "unknown" — it degrades to a confident, wrong,
positive claim about an individual customer's order, including for orders that were delivered or
cancelled. Add an unresolved owner (nobody is obliged to fix it), no authentication (anyone can
query it), and documentation that would misdirect the person who did investigate, and the
combination clears the bar.

Three things argue the other way, recorded honestly. The cancellation path — the largest cluster
of findings — has no caller in this repository, so those findings are latent rather than active.
No incident record naming delivery-bff exists. And this is code reading only: without carrier
error rates or CS enquiry data, the frequency of the degraded path is unknown, and frequency is
exactly what separates a high from a critical here. If carrier availability is good, the lie is
rare; nothing in the repository indicates either way, and the retry loop's existence — 배송사
API가 불안정하여 재시도를 넣어두었다, "the carrier API is unstable so retries were added" — is the
only evidence, and it points the wrong way.

**Resolution: open, unassigned.**

- Regenerating the client is tracked as **SF-4901**, stated as 미착수 (not started) in a code
  comment. No ticket document for it exists in `sellflow-reef/sources`. Regeneration alone is
  insufficient: the only available spec (`sellflow-docs:raw/specs/order-service-openapi.json`,
  2022-11-04) still describes the pre-SF-2287 policy and still lacks the current status values, so
  regenerating from it reproduces findings 7 and 8 exactly. The spec must be refreshed from
  springdoc first.
- The ownership `TODO` in `services.yaml` carries no ticket reference at all — only the inline
  note that the transfer has been under discussion since 2025-11.
- Findings 1, 2, 10, 11, 12, 14, 15 and 16 carry no tracking reference anywhere in the repository
  or in the reef's sources.

## Recommended Actions

1. **Stop the fallback from asserting a status.** Either return HTTP 503 with an explicit
   `unavailable` body, or keep 200 but return `status: null` with `stale: true`. A one-line change
   converts a confident falsehood into an honest unknown, and it is the highest-value item here.
2. **Log at ERROR on exhaustion and count stale responses.** Currently the only moment worth
   alerting on is the only moment that is not logged.
3. **Correct the README and the code comment**, or build the store they describe. Leaving three
   documents describing a cache that does not exist is the finding most likely to waste an
   engineer's incident.
4. **Rename the test or delete it.** A test named for behaviour that does not exist, asserting
   `true`, is worse than no test — it reads as coverage.
5. **Settle ownership.** Every other item needs an owner, and the registry's `TODO` has outlasted
   two reviews. Note the org chart's split: whoever takes the code may not be who holds the
   carrier relationship.
6. **Refresh the spec, then regenerate** — in that order — and repoint the client at an absolute
   base URL read from `ORDER_API_BASE`, which already exists in `.env.template` and is read by
   nothing.
7. **Turn on `strict`, add `tests` to the `include`, add a `test` script and `vitest` to
   devDependencies, and add a CI workflow** — the four cheapest items, and jointly the ones that
   would have surfaced findings 6, 7 and 10 automatically.
8. **Decide what a delivery status value space is.** Both the carrier's strings and the fallback
   literal currently pass through undefined; see [[PAT-SELLFLOW-ROMANISED-NAMING]] for the
   convention the rest of the estate follows.

## Related

- [[API-DELIVERY]] — the degraded response and the consumed cancel call
- [[CON-ORDER-DELIVERY]] — the boundary the stale client sits on
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — the naming convention `PREPARING` falls outside
- [[PROC-DELIVERY-AUTH]] — the missing credential, and the unenforced requirement on the other side
- [[PROC-DELIVERY-ERROR-HANDLING]] — the full mechanics behind findings 1, 2, 4, 5, 6 and 8
- [[PROC-DELIVERY-STATUS-SYNC]] — silent exhaustion and dead configuration
- [[RISK-INVENTORY]] — the same density argument in the estate's other small service
- [[RISK-SELLFLOW-DOC-DRIFT]] — the documented-versus-actual pattern finding 2 belongs to
- [[SYS-DELIVERY]] — the service and its ownership conflict
- [[SYS-ORDER]] — the callee whose spec, routes and responses the client no longer matches
