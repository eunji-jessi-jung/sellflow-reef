---
id: "API-ORDER"
type: "api"
title: "Order Service API"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 into a delta layer over the two OpenAPI files rather than a restatement of them: the surface profile (30 paths / 32 operations, method split, path families), the auth posture that answers Q-009, the non-CRUD action-path inventory, the pagination and envelope conventions, the error-code distribution by path family, and the contract limits OpenAPI cannot express (the SF-2287 cancel rule, the reason-code restock fork, the outbox write). Counts were recomputed from the JSON on 2026-09-19 and one of them contradicts openapi.meta.json — see Key Facts. The code-confirmed table was read from the two controller classes and is current; the spec table is a frozen 2022-11-04 artefact that cannot be regenerated, because springdoc-openapi is not a declared dependency. Re-verify whenever a @RestController is added or removed, or when delivery-bff's generated client is regenerated."
freshness_triggers:
  - "build.gradle"
  - "delivery-bff:src/generated/orderApi.ts"
  - "sources/raw/specs/order-service-openapi.json"
  - "src/main/java/kr/co/sellflow/order/config/WebConfig.java"
  - "src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
  - "src/main/java/kr/co/sellflow/order/search/OrderSearchController.java"
  - "src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
known_unknowns:
  - "Whether the 29 spec-only paths ever existed. They may have been deleted, moved to another service, or never implemented; no code, ticket or note in the reef explains their absence."
  - "Where the gateway is. Auth is absent from all five repos and the CORS comment says the app 'comes in through the gateway', but no gateway config, route table or repo is available here, so whether bearerAuth is terminated anywhere at all is unverified. See PROC-ORDER-AUTH and the Q-009 answer below."
  - "Whether openapi.meta.json's '30 paths / 45 operations' figure came from a different file revision or is simply wrong. Counting the operations in sources/raw/specs/order-service-openapi.json on 2026-09-19 gives 32, not 45."
  - "Which of the three advertised base paths a live caller actually uses: services.yaml's https://api.sellflow.co.kr/orders, the spec's https://order.internal.sellflow.co.kr + /orders, or delivery-bff's /api/v1/orders. Only the second matches the controller's own mapping."
  - "The spec's 200 response for cancel returns an OrderCancel body; the controller returns ResponseEntity.ok().build(), an empty body. Which one clients depend on is unknown."
  - "What happens to a bigo longer than 2000 characters. The spec declares no maxLength, the JPA entity says 500, the column is VARCHAR(2000) and ddl-auto is none, so nothing in the request path validates it."
  - "Whether any caller outside these five repos sends an Authorization header. Nothing in the repo would read it, but its presence would tell us whether a gateway is still injecting one."
  - "GET /api/v1/orders/search has no pagination, no result limit and its documented three-month cap is unenforced. Whether an upstream layer enforces it is unknown."
tags:
  - "order"
  - "rest"
  - "openapi"
  - "spec-drift"
  - "auth"
aliases:
  - "order-service API"
  - "주문 API"
relates_to:
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "constrains"
    target: "[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]]"
  - type: "depends_on"
    target: "[[PROC-ORDER-AUTH]]"
  - type: "refines"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "depends_on"
    target: "[[SCH-ORDER]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "Defines ORDER_API_BASE, which the generated client never reads."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "The 2022-11-08 generated client — the /api/v1 prefix, the five-value OrderStatus and the 409 message all live here."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
    notes: "The only wrapper around the generated client; itself has no caller."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "RESTOCKABLE_REASONS = {'01','02'} — the reason-code side effect the order contract cannot express."
  - category: "implementation"
    type: "github"
    ref: "order-service:SERVER_VERSION"
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "Four dependencies, none of them springdoc-openapi or spring-boot-starter-security."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "Calls a restore path that does not exist, and is never called by the cancel path."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java"
    notes: "Declares length = 500 for BIGO against a VARCHAR(2000) column."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The blocked-state set that replaced the settlement check."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/search/OrderSearchServiceTest.java"
    notes: "The only evidence of the from/to date format the search endpoint expects."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.code-derived.json"
    notes: "Tier-4 extraction from the controllers. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.json"
    notes: "Carries the [STALE — DO NOT TRUST AS CURRENT] banner. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.meta.json"
    notes: "Extraction provenance and staleness evidence. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Advertises a third base path, https://api.sellflow.co.kr/orders. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The decision that inverted the cancel precondition the spec still documents. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "The 2022-11-04 springdoc artefact, v2.4.0. All counts in Surface Profile were computed from this file. Reef-root relative."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "What a 200 from the cancel endpoint eventually causes, ten minutes later, in another service."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "Sets ORDER_MST.SANGTAE_CD to JUNGSAN_WANRYO by direct UPDATE — the force-status action the API never exposed."
notes: "This artifact is a delta layer over the two OpenAPI files: it records what they say in aggregate, what they cannot say, and where they disagree with the code. It does not restate their schemas — read the JSON for those. Company-document refs are relative to the reef root; code refs are relative to each repo root."
---

# Order Service API

## Overview

There are two answers to "what is the order API", and they overlap on exactly one path: 29 of the spec's 30 paths have no handler in the repo, and the code's second endpoint is in no spec. This artifact keeps both, labelled, rather than picking one — and then goes past both, because the interesting facts about this API are the ones OpenAPI has no field for: that nothing enforces the security scheme it declares, that a 200 from its one live write endpoint is a promise about a row in a table rather than about the money, and that the one sentence the spec writes about that endpoint has been false since 2023.

## Key Facts

- The 2022 OpenAPI spec documents 30 paths carrying 32 operations, across four tags (주문 / order, 파트너 / partner, 관리자 / admin, 내부 / internal) → sources/raw/specs/order-service-openapi.json
- The method split is GET 18, POST 12, PUT 2 — there is no DELETE and no PATCH anywhere in the spec, so even the documented API never modelled deletion → sources/raw/specs/order-service-openapi.json
- The reef's own extraction record states "30 paths / 45 operations"; recounting the same file on 2026-09-19 gives 32 operations, and the 45 could not be reproduced → sources/apis/order/openapi.meta.json
- The spec declares a global `security: [{"bearerAuth": []}]` with an HTTP bearer/JWT scheme, and **not one of the 32 operations overrides it** — so the document claims 32 of 32 operations are authenticated → sources/raw/specs/order-service-openapi.json
- A case-insensitive grep for `authorization|bearer|jwt|SecurityConfig|spring-security|OncePerRequest|HandlerInterceptor|addFilter` across all five sellflow repos returns **zero matches**, so 0 of the 2 live operations are gated by anything → order-service:build.gradle, order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java
- The only access control that exists in the repo is a CORS mapping on `/api/**` restricted to `https://admin.sellflow.co.kr`, with the comment "어드민 도메인만 허용. 앱은 게이트웨이를 통해 들어온다." ("only the admin domain is allowed; the app comes in through the gateway") — and CORS binds browsers only, so it gates nothing server-side → src/main/java/kr/co/sellflow/order/config/WebConfig.java
- `POST /orders/{ordNo}/cancel`, the one endpoint that writes, carries no CORS mapping at all, because the mapping is on `/api/**` and the cancel controller is mapped at `/orders` → src/main/java/kr/co/sellflow/order/controller/OrderController.java
- The only known client, delivery-bff's generated `cancelOrder`, sends exactly one header — `Content-Type: application/json` — and no `Authorization`, which is consistent with a service that would ignore it → delivery-bff:src/generated/orderApi.ts
- Ten of the 12 documented POST paths are verb-suffixed action paths rather than resource CRUD (`/cancel`, `/hold`, `/release`, `/address`, `/memo`, `/split`, `/merge`, `/suspend`, `/bulk-cancel`, `/replay`), and a PUT action path (`/force-status`) joins them — the documented API was RPC-over-REST, not resource-oriented → sources/raw/specs/order-service-openapi.json
- Exactly one of those eleven action paths survives in code, and it is `/cancel` → src/main/java/kr/co/sellflow/order/controller/OrderController.java
- Pagination is declared two different ways in one document: `GET /orders` takes `page` and `size` as query parameters, while `POST /admin/orders/search` carries `page`, `size` and `sort` inside the `OrderSearchRequest` body. No cursor or token pagination appears anywhere → sources/raw/specs/order-service-openapi.json
- The spec carries 16 wrapper schemas — `ApiResponse{X}` (`{success, data, message, traceId}`) and `PageResponse{X}` (`{content, meta, success, traceId}`) for eight entity types — which is half of its 32 component schemas. No envelope of any kind exists in the live code: `OrderSearchController` returns a bare `List<OrderMst>` → src/main/java/kr/co/sellflow/order/search/OrderSearchController.java
- Every operation outside `/internal/*` declares two optional headers, `X-Request-Id` and `X-Client-Ver`; neither string appears anywhere in the order-service source, so neither is read, logged or propagated → sources/raw/specs/order-service-openapi.json
- Declared response codes across the 32 operations: 200×31, 401×29, 500×29, 404×19, 409×8, 403×8, 400×2, 201×1. The distribution is by path family, not by operation: `/orders/*` declares 404+409, `/partners/*` declares 404 only, `/admin/*` declares 403 and never 404, and the three `/internal/*` probes declare 200 alone → sources/raw/specs/order-service-openapi.json
- The three `/internal/*` probes declare no 401 despite the global `bearerAuth`, which is the document's only hint that probe traffic was meant to be exempt — and it is expressed by omission, not by a `security: []` override → sources/raw/specs/order-service-openapi.json
- `ErrorResponse` declares `{code, message, traceId}`, but `GlobalExceptionHandler` returns `Map.of("message", ...)` only, and no error-code vocabulary (no enum, no constants class, no lookup) exists anywhere in the repository — so `code` has never had values to carry → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- There are three mutually incompatible base paths in circulation: `services.yaml` advertises `https://api.sellflow.co.kr/orders`, the spec's `servers` are `https://order.internal.sellflow.co.kr` (운영 / production) and `https://order-stg.internal.sellflow.co.kr` (스테이징 / staging) against paths beginning `/orders`, and delivery-bff's generated client fetches `/api/v1/orders/{ordNo}/cancel` → sources/context/registry/services.yaml, sources/raw/specs/order-service-openapi.json, delivery-bff:src/generated/orderApi.ts
- `/api/v1` is a real prefix in this service, but it belongs to exactly one endpoint, `GET /api/v1/orders/search`, which has no `{ordNo}/cancel` sub-path — so the generated client's URL matches neither the spec it was generated from nor the controller it targets → src/main/java/kr/co/sellflow/order/search/OrderSearchController.java
- The spec's cancel description still states the pre-SF-2287 policy: "주문을 취소한다. 정산이 완료된 주문은 취소할 수 없습니다." ("cancels an order; a settled order cannot be cancelled") — SF-2287 deleted that check in 2023 and the code no longer behaves this way → sources/apis/order/openapi.json, sources/context/tickets/SF-2287.md
- An unknown `sayuCd` reaches `CancelReason.of`, which throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` ("unknown cancel reason code"); `GlobalExceptionHandler` declares no handler for it, so the extraction records the outcome as a 500 → src/main/java/kr/co/sellflow/order/domain/CancelReason.java, src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- The drift has a downstream victim: the client generated from this spec on 2022-11-08 still types `OrderStatus` with five values beginning `JUMUN_WANRYO`, a name the enum no longer contains, and lacks `GYEOLJE_WANRYO`, `SANGPUM_JUNBI` and `JUNGSAN_WANRYO` → delivery-bff:src/generated/orderApi.ts, sources/apis/order/openapi.meta.json
- That client also declares `Promise<CancelResponse>` and calls `res.json()`, while the controller answers `ResponseEntity.ok().build()` with an empty body — parsing an empty body as JSON throws, so a successful cancel would surface to the caller as a failure → delivery-bff:src/generated/orderApi.ts, src/main/java/kr/co/sellflow/order/controller/OrderController.java

## Source of Truth

**The code is the source of truth. The spec is a 2022 photograph.**

Stated plainly, because the distinction is easy to lose: the reef holds two separate descriptions of this API, and they are not two versions of one document.

1. `sources/apis/order/openapi.json` — the frozen 2022-11-04 springdoc artefact, 30 paths, info.version 2.4.0. It cannot be regenerated, because springdoc-openapi is not a declared dependency in `build.gradle`, so it will stay frozen at 2022 for as long as that remains true → sources/apis/order/openapi.meta.json, build.gradle
2. `sources/apis/order/openapi.code-derived.json` — a tier-4 surface read straight from the Spring controller annotations, 2 endpoints, info.version 2.8.14 from SERVER_VERSION → sources/apis/order/openapi.code-derived.json

The second is authoritative for anything current. The extraction record says so in as many words: the code-derived file "is the live-truth surface. Prefer it over openapi.json for anything current." → sources/apis/order/openapi.meta.json. The first is kept only as evidence of what was once published and of what clients generated from it still believe.

| | Documented spec | Code-confirmed |
|---|---|---|
| Artefact | `sources/raw/specs/order-service-openapi.json`, copied to `sources/apis/order/openapi.json` | `OrderController`, `OrderSearchController` |
| Version | 2.4.0, generated 2022-11-04 | SERVER_VERSION 2.8.14 (build.gradle still says 2.8.4) |
| Paths / operations | 30 / 32 | 2 / 2 |
| Auth | global `bearerAuth` (JWT), no per-operation override | none present |
| Regenerable | no — springdoc-openapi is not a dependency | n/a |

The overlap between the two is a single endpoint, and even that one has drifted: its documented precondition (settled orders cannot be cancelled) was deliberately removed by SF-2287 in 2023. Treat every spec-only row below as a historical claim, not an interface.

The reef's own metadata records the same split mechanically: `openapi.meta.json` logs extraction tier 3 for the spec and tier 4 for the code-derived companion, and notes that tier 2 (scraping a running `/v3/api-docs`) was skipped at pre-flight because no such endpoint can exist without the springdoc dependency → sources/apis/order/openapi.meta.json

## Surface Profile

Aggregate shape of the documented API, recomputed from `sources/raw/specs/order-service-openapi.json` on 2026-09-19. These numbers are the point of this section: individual path definitions are in the JSON and are not restated here.

| Measure | Documented (2022) | Live (code-derived) |
|---|---|---|
| Paths | 30 | 2 |
| Operations | 32 | 2 |
| GET / POST / PUT | 18 / 12 / 2 | 1 / 1 / 0 |
| DELETE / PATCH | 0 / 0 | 0 / 0 |
| Tags | 4 (주문, 파트너, 관리자, 내부) | 2 (Order, Order Search) |
| Component schemas | 32, of which 16 are `ApiResponse*` / `PageResponse*` wrappers | 4 (`CancelRequest`, `OrderMst`, `OrderStatus`, `ErrorResponse`) |
| Operations declaring `security` at operation level | 0 (all inherit the global requirement) | n/a — no security object exists |
| Servers | 2, both `*.internal.sellflow.co.kr` | none declared |

Path families, by first segment:

| Prefix | Paths | Operations | Shape |
|---|---|---|---|
| `/orders` | 13 | 14 | Collection + item + 7 action sub-paths. `/orders` alone carries both GET and POST. |
| `/admin` | 8 | 8 | Operational tooling: search, bulk-cancel, export, force-status, two stats endpoints, outbox list and replay. |
| `/partners` | 6 | 7 | Read-mostly; `/contract` is the only path with two methods (GET + PUT). |
| `/internal` | 3 | 3 | health, ready, metrics. |

Two structural notes that the per-path tables below make hard to see:

- **No spec path uses DELETE.** Cancellation, suspension and status reversal are all modelled as POST or PUT actions on a sub-path. The schema agrees: `ORDER_CANCEL` is an insert, never a delete of `ORDER_MST` → [[SCH-ORDER]]
- **Nothing exposes a partner resource in this schema.** The seven `/partners/*` paths have no backing table in `sellflow_order`; `PARTNER_ID` exists only as a column on `ORDER_DTL` → [[SCH-ORDER]]

## Auth Posture (answers Q-009)

**Question Q-009: how is authentication and authorisation enforced in order-service — bearerAuth in the spec versus code?**

**Answer: it is not enforced anywhere in any of the five repositories. The spec's declaration is aspirational or historical; the running service accepts unauthenticated requests on both of its endpoints.**

The evidence, in the order it was gathered:

| Claim | Where it comes from | Verdict |
|---|---|---|
| Every operation requires a JWT bearer token | Spec `security: [{"bearerAuth": []}]` at document level; `components.securitySchemes.bearerAuth = {type: http, scheme: bearer, bearerFormat: JWT}` | Declared for 32 of 32 operations |
| Some operations are exempt | No operation declares its own `security`, including the three `/internal/*` probes. Their exemption is implied only by declaring no 401 | Not expressible as written |
| The code checks a token | `build.gradle` declares five dependencies: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `flyway-core`, `mysql-connector-java` (runtimeOnly) and `spring-boot-starter-test`. No `spring-boot-starter-security` | Refuted |
| A filter or interceptor checks a token | `grep -rniE 'authorization\|bearer\|jwt\|SecurityConfig\|spring-security\|OncePerRequest\|HandlerInterceptor\|addFilter'` across order-service, settlement-batch, inventory-api, delivery-bff and settlement-anomaly returns nothing | Refuted, across all five repos |
| Method-level authorisation exists | No `@PreAuthorize`, `@Secured` or `@RolesAllowed` in the source tree; the only Spring config classes are `WebConfig` (CORS only) and `JpaConfig` (auditing + transactions) | Refuted |
| Something upstream terminates auth | `WebConfig`'s comment: "앱은 게이트웨이를 통해 들어온다." ("the app comes in through the gateway") | Unverifiable here — no gateway repo, route table or config exists in the workspace |

So the two live operations sit at different levels of exposure, and neither is authenticated:

| Live operation | Network control | Auth control | Effect of a request with no credentials |
|---|---|---|---|
| `POST /orders/{ordNo}/cancel` | none in this repo — no CORS mapping covers `/orders/**` | none | Cancels the order, writes `ORDER_CANCEL`, flips `ORDER_MST.SANGTAE_CD`, inserts an outbox row |
| `GET /api/v1/orders/search` | CORS restricted to `https://admin.sellflow.co.kr`, which constrains browsers only | none | Returns 200 with a list of nulls (the RowMapper is `(rs, i) -> null`) |

The asymmetry is worth stating directly: the only endpoint with any declared origin restriction is the read-only one that cannot return data, and the endpoint that moves money-adjacent state has none. The `403` responses the spec declares for all eight `/admin/*` paths imply a role model (an admin scope distinct from a user scope) that has no representation in code at all — no role enum, no claim name, no authority string.

This section answers Q-009 for the API surface. The organisational and runtime side of the same question — who is supposed to own the gateway, and what the other four services do — belongs to [[PROC-ORDER-AUTH]].

## Non-CRUD Action Paths

The documented API is shaped as RPC over HTTP: a noun path plus a verb segment, always POST (or, once, PUT). This matters more than it looks, because it is the reason none of these operations can be inferred from the schema — a `POST .../cancel` has side effects in three tables and two other services, and the path tells you none of that.

| Action path | Method | Tag | In code? | What OpenAPI conveys about its effect |
|---|---|---|---|---|
| `/orders/{ordNo}/cancel` | POST | 주문 | **yes** | Summary line only, and that line is now false |
| `/orders/{ordNo}/hold` | POST | 주문 | no | Nothing |
| `/orders/{ordNo}/release` | POST | 주문 | no | Nothing |
| `/orders/{ordNo}/address` | POST | 주문 | no | Nothing |
| `/orders/{ordNo}/memo` | POST | 주문 | no | Nothing |
| `/orders/{ordNo}/split` | POST | 주문 | no | Nothing — though `ORDER_MST.PARENT_ORD_NO` (V10) exists for it |
| `/orders/{ordNo}/merge` | POST | 주문 | no | Nothing — same column |
| `/partners/{partnerId}/suspend` | POST | 파트너 | no | Nothing |
| `/admin/orders/bulk-cancel` | POST | 관리자 | no | Nothing; a bulk form of the one operation with the deepest side effects |
| `/admin/events/outbox/{eventId}/replay` | POST | 관리자 | no | Nothing; replay of an outbox row whose consumer is another team's batch |
| `/admin/orders/{ordNo}/force-status` | PUT | 관리자 | no | Nothing; the only documented way to set a status arbitrarily |

Two absences are findings in their own right. `/admin/events/outbox` and `.../replay` would be the operational handles for the outbox — the mechanism at the centre of the cancel-to-settlement path — and neither exists; the outbox has no operator interface at all, in any service. And `/admin/orders/{ordNo}/force-status` is the documented counterpart of what settlement-batch's `MarkSettledTasklet` does today by writing `ORDER_MST.SANGTAE_CD` directly, without an API → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java, [[SCH-ORDER]]

## Pagination and Response Conventions

| Convention | Documented | Live |
|---|---|---|
| Page parameters | `page`, `size` as query params on `GET /orders`; `page`, `size`, `sort[]` inside the `OrderSearchRequest` body on `POST /admin/orders/search` | none |
| Page metadata | `PageMeta {page, size, totalElements, totalPages}` — offset-based, total-count-based | none |
| Cursor / token pagination | not present | not present |
| Collection envelope | `PageResponse{X} {content, meta, success, traceId}` | none — bare `List<OrderMst>` |
| Single-item envelope | `ApiResponse{X} {success, data, message, traceId}` | none — empty body on cancel |
| Correlation id | `traceId` on both envelopes and on `ErrorResponse`; `X-Request-Id` as an optional request header on 29 operations | none — the string `traceId` appears in no source file |
| Result caps | not declared anywhere in the spec | none in code; the search controller's javadoc says "기간 최대 3개월" ("maximum period three months") and nothing enforces it |

The practical consequence for the one live read endpoint: `GET /api/v1/orders/search` will attempt `SELECT * FROM ORDER_MST WHERE ORD_DT BETWEEN ? AND ?` for whatever range is asked for, with no `LIMIT`, and then map every row to `null` → src/main/java/kr/co/sellflow/order/search/OrderSearchService.java. A caller cannot distinguish "no orders" from "ten million orders" from the response.

## Error-Code Patterns

The spec's status codes are assigned by path family rather than by what an operation does — a strong hint that they were authored as a template and applied, not derived.

| Family | 200/201 | 400 | 401 | 403 | 404 | 409 | 500 |
|---|---|---|---|---|---|---|---|
| `/orders/*` mutations (cancel, hold, release, address, memo, split, merge) | 200 | — | yes | — | yes | yes | yes |
| `/orders/*` reads | 200 | only on `GET /orders` | yes | — | yes | — | yes |
| `POST /orders` | 201 | yes | yes | — | — | yes | yes |
| `/partners/*` | 200 | — | yes | — | yes | — | yes |
| `/admin/*` | 200 | — | yes | yes | **never** | — | yes |
| `/internal/*` | 200 | — | **never** | — | — | — | — |

Against that, what the code can actually produce:

| Status | Produced by | Body |
|---|---|---|
| 200 | `OrderController.cancel` (empty), `OrderSearchController.search` (list of nulls) | no envelope |
| 404 | `OrderNotFoundException` via `GlobalExceptionHandler` | `{"message": "주문을 찾을 수 없습니다. ordNo=..."}` |
| 409 | `OrderCancelNotAllowedException` via `GlobalExceptionHandler` | `{"message": "취소할 수 없는 주문 상태입니다. ordNo=..., status=<Korean label>"}` |
| 500 | `IllegalArgumentException` from `CancelReason.of` — unhandled | Spring's default error body, not `ErrorResponse` |
| 401 / 403 | nothing | — |

Three patterns worth carrying forward:

1. **Error messages are Korean prose with interpolated identifiers, and they are the entire contract.** There is no stable machine-readable code, so a client that wants to distinguish "already cancelled" from "returned" must parse `status=취소` versus `status=반품` out of a sentence — the value is `OrderStatus.getLabel()`, the Korean label, not the enum name → src/main/java/kr/co/sellflow/order/service/OrderCancelNotAllowedException.java
2. **The 409 handler asserts precedence over the document.** Its comment reads "409 로 내려간다. API 스펙 문서와 상태코드는 여기 기준." ("it goes out as 409; for the API spec document, the status codes follow this") → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
3. **A validation error is a server error.** An unrecognised `sayuCd` is a client mistake that surfaces as 500, because the only exception handlers registered are for the two domain exceptions.

## Contract Limits

What follows cannot be expressed in the OpenAPI document, is not expressed anywhere else machine-readable, and is the part an agent most needs.

**1. The cancel precondition is a code-level enum set, and it inverted in 2023.** The rule is `CHWISO_BULGA = EnumSet.of(OrderStatus.CHWISO, OrderStatus.BANPUM)` — only an already-cancelled or returned order is refused. Settlement state is not consulted → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java. SF-2287 requested exactly that: "취소 API에서 정산 상태 확인 로직을 제거해 주세요." ("please remove the settlement-status check from the cancel API"), resolved 2023-04-21 in order-service 2.8.0 → sources/context/tickets/SF-2287.md. The spec, the 2021 Confluence page ("정산완료 / X / 취소 불가" — settled: not cancellable) and delivery-bff's 409 error string all still describe the old rule. A schema field cannot carry this; only [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] and [[PROC-ORDER-CANCEL]] can.

**2. `sayuCd` selects a side effect in another service, and the order API neither says so nor triggers it.** `business-rules.md` sheet 취소정책 assigns per-code outcomes: `01` 파트너 귀책 restocks, `02` 시스템 오류 restocks, `03` 고객 변심 does **not** ("배송 시작 후 취소 시 재고 복원 불가" — stock cannot be restored once delivery has started), `04` 배송 실패 restocks → sources/context/business-rules.md. inventory-api enforces a narrower version of the same fork in code: `RESTOCKABLE_REASONS = {"01", "02"}`, so `04` restocks on paper and does not in code → inventory-api:app/main.py. And none of it is reached from a cancel: `OrderCancelService` never calls `InventoryClient`, and `InventoryClient.restore` posts to `{INVENTORY_BASE_URL}/inventory/restore?ordNo=..&reason=..`, a path inventory-api does not serve — its endpoint is `POST /stock/restock` with a JSON body `{ord_no, reason_code}` → src/main/java/kr/co/sellflow/order/client/InventoryClient.java, inventory-api:app/main.py. A 200 from cancel therefore restocks nothing, for any reason code. See [[CON-ORDER-INVENTORY]].

**3. A 200 means a row was written, not that anyone acted on it.** The cancel transaction inserts into `ORDER_EVENT_OUTBOX` with `PUBLISHED_YN='N'` and a hand-built payload, `String.format("{\"ordNo\":\"%s\",\"sayuCd\":\"%s\"}", ...)`, and the publisher's javadoc is explicit: "구독 측 처리 결과는 본 서비스에서 추적하지 않는다." ("this service does not track the subscriber's processing outcome") → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java. Up to ten minutes later, settlement-batch's `OrderEventRelayJob` polls at most 500 unpublished rows, and for any order already present in `SETTLEMENT_DTL` inserts `CANCEL_RECON_QUEUE (ORD_NO, SAYU_CD, STATUS='PENDING')` before flipping `PUBLISHED_YN='Y'` → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java. The HTTP response carries no event id, no outbox id and no correlation id, so a caller holding a 200 has no handle on any of it. See [[CON-ORDER-SETTLEMENT]].

**4. There is no idempotency contract.** `ORDER_CANCEL` is keyed on `ORD_NO` alone, so a second cancel of the same order violates the primary key rather than returning a clean duplicate response — except that the second call is normally refused at 409 first, because the status is already `CHWISO`. No `Idempotency-Key` header is declared or read, and `X-Request-Id` is declared but never read → [[SCH-ORDER]]

**5. Field-length validation exists in three places and agrees in none.** `CancelRequest.bigo` has no `maxLength` in the spec, `length = 500` on the JPA entity, and `VARCHAR(2000)` in the column since V9 — and with `ddl-auto: none` the entity's number governs nothing → src/main/java/kr/co/sellflow/order/domain/OrderCancel.java, [[SCH-ORDER]]

**6. The base path is unsettled, and the one shipped client picked a fourth option.** The controller serves `/orders/{ordNo}/cancel`; the spec documents `/orders/{ordNo}/cancel` under an internal host; `services.yaml` advertises `https://api.sellflow.co.kr/orders`; delivery-bff's generated client requests `/api/v1/orders/{ordNo}/cancel` as a **relative** URL with no base, while `.env.template` defines an `ORDER_API_BASE` the client never reads → delivery-bff:src/generated/orderApi.ts, delivery-bff:.env.template. The `/api/v1` prefix is not invented from nothing — it is the prefix of the search controller — but no cancel endpoint has ever lived under it in either description of this API. See [[DEC-DELIVERY-GENERATED-CLIENT]].

## Resource Map

| Resource | Spec paths | Live paths | Backing tables |
|---|---|---|---|
| Order | 13 (14 operations) | 0 | `ORDER_MST`, `ORDER_DTL` ([[SCH-ORDER]]) |
| Cancellation | 1 (of the 13 above) | 1 | `ORDER_CANCEL`, `ORDER_EVENT_OUTBOX` ([[PROC-ORDER-CANCEL]]) |
| Order search | 1 (`POST /admin/orders/search`) | 1 (`GET /api/v1/orders/search`) | `ORDER_MST` |
| Partner | 6 (7 operations) | 0 | none in this schema — `PARTNER_ID` is only a column on `ORDER_DTL` |
| Admin / stats / outbox ops | 8 | 0 | `ORDER_EVENT_OUTBOX` |
| Internal probes | 3 | 0 | none |

The absence of the three `/internal/*` endpoints is worth noting on its own: nothing in the repo exposes a health or readiness probe, and `spring-boot-starter-actuator` is not a dependency → build.gradle

### Spec endpoints (2022, unconfirmed unless marked)

Tag 주문 (order) — 13 paths, 14 operations:

| Method | Path | Summary | Responses | In code? |
|---|---|---|---|---|
| GET | /orders | 주문 목록 조회 (list orders) | 200, 400, 401, 500 | no |
| POST | /orders | 주문 생성 (create order) | 201, 400, 409, 401, 500 | no |
| GET | /orders/{ordNo} | 주문 단건 조회 (get one order) | 200, 404, 401, 500 | no |
| GET | /orders/{ordNo}/details | 주문 상세 조회 (order details) | 200, 404, 401, 500 | no |
| GET | /orders/{ordNo}/history | 주문 상태 이력 조회 (status history) | 200, 404, 401, 500 | no |
| GET | /orders/{ordNo}/delivery | 주문 배송 정보 조회 (delivery info) | 200, 404, 401, 500 | no |
| GET | /orders/{ordNo}/payment | 주문 결제 정보 조회 (payment info) | 200, 404, 401, 500 | no |
| POST | /orders/{ordNo}/cancel | 주문 취소 (cancel order) | 200, 404, 409, 401, 500 | **yes** |
| POST | /orders/{ordNo}/hold | 주문 보류 (hold order) | 200, 404, 409, 401, 500 | no |
| POST | /orders/{ordNo}/release | 주문 보류 해제 (release hold) | 200, 404, 409, 401, 500 | no |
| POST | /orders/{ordNo}/address | 배송지 변경 (change address) | 200, 404, 409, 401, 500 | no |
| POST | /orders/{ordNo}/memo | 주문 메모 등록 (add memo) | 200, 404, 409, 401, 500 | no |
| POST | /orders/{ordNo}/split | 주문 분할 (split order) | 200, 404, 409, 401, 500 | no |
| POST | /orders/{ordNo}/merge | 주문 병합 (merge orders) | 200, 404, 409, 401, 500 | no |

Tag 파트너 (partner) — 6 paths, 7 operations:

| Method | Path | Summary | Responses | In code? |
|---|---|---|---|---|
| GET | /partners | 파트너 목록 조회 (list partners) | 200, 404, 401, 500 | no |
| GET | /partners/{partnerId} | 파트너 단건 조회 (get partner) | 200, 404, 401, 500 | no |
| GET | /partners/{partnerId}/orders | 파트너 주문 목록 (partner orders) | 200, 404, 401, 500 | no |
| GET | /partners/{partnerId}/settlements | 파트너 정산 내역 (partner settlements) | 200, 404, 401, 500 | no |
| GET | /partners/{partnerId}/contract | 파트너 계약 조회 (get contract) | 200, 404, 401, 500 | no |
| PUT | /partners/{partnerId}/contract | 파트너 계약 수정 (update contract) | 200, 404, 401, 500 | no |
| POST | /partners/{partnerId}/suspend | 파트너 일시중지 (suspend partner) | 200, 404, 401, 500 | no |

Tag 관리자 (admin) — 8 paths:

| Method | Path | Summary | Responses | In code? |
|---|---|---|---|---|
| POST | /admin/orders/search | 주문 통합 검색 (unified order search) | 200, 403, 401, 500 | no |
| POST | /admin/orders/bulk-cancel | 주문 일괄 취소 (bulk cancel) | 200, 403, 401, 500 | no |
| GET | /admin/orders/export | 주문 내보내기 (export orders) | 200, 403, 401, 500 | no |
| PUT | /admin/orders/{ordNo}/force-status | 주문 상태 강제 변경 (force status) | 200, 403, 401, 500 | no |
| GET | /admin/stats/daily | 일별 통계 (daily stats) | 200, 403, 401, 500 | no |
| GET | /admin/stats/partner | 파트너별 통계 (per-partner stats) | 200, 403, 401, 500 | no |
| GET | /admin/events/outbox | 아웃박스 이벤트 조회 (list outbox events) | 200, 403, 401, 500 | no |
| POST | /admin/events/outbox/{eventId}/replay | 아웃박스 이벤트 재발행 (replay event) | 200, 403, 401, 500 | no |

Tag 내부 (internal) — 3 paths:

| Method | Path | Summary | Responses | In code? |
|---|---|---|---|---|
| GET | /internal/health | 헬스체크 (health check) | 200 | no |
| GET | /internal/ready | 레디니스 (readiness) | 200 | no |
| GET | /internal/metrics | 메트릭 (metrics) | 200 | no |

### Code-confirmed endpoints

| Method | Path | Handler | Request | Success | Errors |
|---|---|---|---|---|---|
| POST | /orders/{ordNo}/cancel | `OrderController.cancel` → `OrderCancelService.cancel` | path `ordNo`; body `CancelRequest {sayuCd, bigo}` | 200, empty body | 404 `OrderNotFoundException`, 409 `OrderCancelNotAllowedException`, 500 from the unhandled `IllegalArgumentException` for an unknown `sayuCd` |
| GET | /api/v1/orders/search | `OrderSearchController.search` → `OrderSearchService.search` | query `from` (required), `to` (required), `sangtaeCd` (optional) | 200, `List<OrderMst>` — but the RowMapper is `(rs, i) -> null`, so every element is null | none declared |

The search endpoint's controller javadoc says "CS 어드민 주문 검색. 기간 최대 3개월." ("CS admin order search; maximum period three months"). No code enforces the three-month limit. The only evidence of the expected `from`/`to` format is its unit test, which passes `"20260801"` and `"20260831"` — bare `yyyyMMdd` strings handed straight to `BETWEEN` → src/test/java/kr/co/sellflow/order/search/OrderSearchServiceTest.java. Its service javadoc carries a CS-facing note: "FIXME(은영) 2024-05: 상태코드 필터에 JUNGSAN_WANRYO 넣으면 결과가 비어 보인다는 CS 문의." ("CS reports that filtering by JUNGSAN_WANRYO looks like it returns nothing"). The FIXME concludes the data is correct and only the on-screen wording should change — though the null-returning RowMapper would make every filter look empty. See [[RISK-ORDER]].

### Worked Example

Cancelling order `ORD20230411002` for reason `03` (고객 변심 / customer changed their mind). This is the one endpoint in this API that changes anything, so it is worth following all the way through.

**Request — as the controller actually serves it:**

```http
POST /orders/ORD20230411002/cancel HTTP/1.1
Host: order-service.internal:8081
Content-Type: application/json

{
  "sayuCd": "03",
  "bigo": "고객 요청으로 취소"
}
```

No `Authorization` header is sent, and none would be read. `X-Request-Id` may be sent — the spec declares it — and would be ignored.

**Success as the code answers it** — `ResponseEntity.ok().build()`:

```http
HTTP/1.1 200 OK
Content-Length: 0
```

**Success as the 2022 spec promises it** (`OrderCancel` schema) — retained because clients generated from the spec expect this shape and will fail to parse the real answer:

```json
{
  "ordNo": "ORD20230411002",
  "chwisoIlsi": "2026-09-19T11:04:22",
  "chwisoSayuCd": "03",
  "choriSangtae": "COMPLETED",
  "bigo": "고객 요청으로 취소"
}
```

**What that 200 actually committed,** in one transaction, none of it visible in the response → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java:

1. `ORDER_CANCEL` — one row, `CHORI_SANGTAE` hardcoded to `'COMPLETED'`, `CHWISO_ILSI` set to `LocalDateTime.now()`
2. `ORDER_MST.SANGTAE_CD` — set to `CHWISO`, `UPD_DTM` refreshed
3. `ORDER_EVENT_OUTBOX` — one row, `EVENT_TYPE='order.cancelled'`, `PUBLISHED_YN='N'`, payload `{"ordNo":"ORD20230411002","sayuCd":"03"}`

And what it did **not** do: no `ORDER_STATUS_HIST` row, no inventory restock call, no PG cancellation, no settlement adjustment. Within ten minutes the relay in another service will see the outbox row and, if this order is already in `SETTLEMENT_DTL`, add it to `CANCEL_RECON_QUEUE` as `PENDING` — a queue whose consumer has never run → [[CON-ORDER-SETTLEMENT]], [[RISK-SETTLEMENT-RECON-BACKLOG]].

**Order not found** — `OrderNotFoundException` → 404:

```json
{
  "message": "주문을 찾을 수 없습니다. ordNo=ORD20230411002"
}
```

("the order cannot be found")

**Order already cancelled or returned** — `OrderCancelNotAllowedException` → 409:

```json
{
  "message": "취소할 수 없는 주문 상태입니다. ordNo=ORD20230411002, status=취소"
}
```

("the order is in a state that cannot be cancelled"). The `status=` value is the enum's Korean label (`OrderStatus.getLabel()`), not its name — `취소` for `CHWISO`, `반품` for `BANPUM`. These are the only two values that can appear, because they are the only two members of the blocked set. Critically, a 409 here does **not** mean "already settled"; that meaning was removed in 2023, but delivery-bff's client still translates any 409 into "정산이 완료된 주문은 취소할 수 없습니다." ("a settled order cannot be cancelled") → delivery-bff:src/generated/orderApi.ts.

**Unknown reason code** — `CancelReason.of("99")` throws `IllegalArgumentException` with no registered handler → 500, with Spring's default error body rather than `ErrorResponse`.

The spec's `ErrorResponse` schema declares `{code, message, traceId}`; the handler returns `Map.of("message", ...)` only, so `code` and `traceId` never appear on any response this service produces.

## How Agents Should Use This

**If you are answering "what endpoints does order-service have?"** — two: `POST /orders/{ordNo}/cancel` and `GET /api/v1/orders/search`. Say the number out loud before quoting anything from the spec, because every tool that reads `openapi.json` will tell you thirty. Cite `sources/apis/order/openapi.code-derived.json`, never `openapi.json`, for anything present-tense.

**If you are asked whether a call is authorised** — nothing in this service authorises anything. Do not infer a gateway; the CORS comment mentions one, but no gateway exists in the workspace and no configuration for it was found. The honest answer is "not enforced in any of the five repos; possibly terminated upstream, unverified" → the Auth Posture table above.

**If you are writing or reviewing a client** — target `/orders/{ordNo}/cancel`, expect an empty 200 body, do not call `.json()` on it, and do not treat 409 as "settled". Generating a client from `openapi.json` reproduces the 2022 mistakes wholesale, which is exactly how delivery-bff's client came to have a URL that matches nothing. See [[DEC-DELIVERY-GENERATED-CLIENT]].

**If you are reasoning about the consequences of a cancel** — the HTTP contract is the wrong layer. Read [[PROC-ORDER-CANCEL]] for the in-service transaction, [[CON-ORDER-SETTLEMENT]] for what the outbox row becomes, [[CON-ORDER-INVENTORY]] for the restock that does not happen, and [[SCH-ORDER]] for the rows written. A 200 from this endpoint is a statement about three inserts and an update, nothing more.

**If you are estimating blast radius for a change to the cancel endpoint** — there is one known client (delivery-bff's `requestCancel`) and it has no caller inside these repos, so the observable live traffic is unattributed. Record that as an unknown rather than concluding the endpoint is unused; the queue it feeds has grown by thousands of rows, so something is calling it → [[RISK-SETTLEMENT-RECON-BACKLOG]].

**What to distrust in this artifact** — the spec tables are a 2022 photograph reproduced for evidence, not an interface. Every row marked "no" in the *In code?* column is a historical claim. The counts in Surface Profile were recomputed on 2026-09-19 and disagree with `openapi.meta.json`'s operation count; prefer the recount, and re-run it before quoting either.

## Related

- [[SYS-ORDER]] — the service exposing this surface
- [[SCH-ORDER]] — the tables behind the resources
- [[PROC-ORDER-CANCEL]] — what the cancel endpoint actually does
- [[PROC-ORDER-AUTH]] — the auth boundary question this artifact answers for the API surface
- [[CON-ORDER-SETTLEMENT]] — what a 200 from cancel becomes downstream
- [[CON-ORDER-INVENTORY]] — the reason-code restock the contract cannot express
- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — the decision that inverted the documented cancel precondition
