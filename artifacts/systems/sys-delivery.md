---
id: "SYS-DELIVERY"
type: "system"
title: "Delivery Lookup BFF"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Full re-read of the delivery-bff repo (every file: src/{index,deliveryStatus,orderClient,logger}.ts, src/generated/orderApi.ts, tests/deliveryStatus.test.ts, package.json, tsconfig.json, .eslintrc.json, .env.template, README.md) on 2026-09-19, cross-read against the extracted delivery OpenAPI and runtime notes, the service registry, the org chart, and order-service's live OrderController / OrderCancelService / OrderStatus plus the 2022 raw spec. Goes stale if the repo gains persistence, auth, a CI workflow or a regenerated client, or if the registry line for delivery-bff stops saying TODO."
freshness_triggers:
  - ".env.template"
  - ".eslintrc.json"
  - "README.md"
  - "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "package.json"
  - "sources/context/org-chart.xlsx"
  - "sources/context/registry/services.yaml"
  - "src/deliveryStatus.ts"
  - "src/generated/orderApi.ts"
  - "src/index.ts"
  - "src/orderClient.ts"
  - "tests/deliveryStatus.test.ts"
  - "tsconfig.json"
known_unknowns:
  - "Ownership is genuinely unresolved: the registry records owner_team as TODO while the org chart assigns delivery-bff to 커머스본부 물류팀. No document in the reef settles which is authoritative."
  - "Who operates the service day to day is unknown — 물류운영본부 배송관리팀 (권나래, 15 people) owns 배송사 관리 · 라스트마일 운영 (carrier management and last-mile operations) but is listed against no system, so its relationship to delivery-bff is undocumented."
  - "No deployment manifest, Dockerfile, Helm chart, or CI workflow exists in the repo, so how the service is built, released, or scaled is unknown."
  - "The runtime version is contradictory: the registry says Node 16, the README says Node 18. package.json declares no engines field and there is no .nvmrc to break the tie."
  - "Which callers consume GET /delivery/:ordNo is undocumented — no client list, no gateway config, no CORS or host allowlist in the repo."
  - "The carrier behind CARRIER_API is unidentified. The code default is the placeholder host https://api.carrier.example, .env.template names a different placeholder https://api.carrier.example.co.kr, and no carrier contract or SLA document is present."
  - "Whether requestCancel has ever been invoked in production is unknowable from this repo — it is exported but has no caller here, and no other repo in sellflow/repos imports delivery-bff."
  - "Why the generated client's path carries an /api/v1 prefix that neither the 2022 spec nor today's controller uses. SF-4901, the ticket named in src/orderClient.ts for regeneration, is not present in sources/context/tickets/."
  - "No observability beyond console logging — no metrics, tracing, or health endpoint is present, so carrier outages are invisible to monitoring."
tags:
  - "delivery"
  - "bff"
  - "express"
  - "typescript"
  - "node"
aliases:
  - "delivery-bff"
  - "배송 조회 BFF"
relates_to:
  - type: "refines"
    target: "[[API-DELIVERY]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "constrains"
    target: "[[DEC-DELIVERY-GENERATED-CLIENT]]"
  - type: "refines"
    target: "[[GLOSSARY-DELIVERY]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-AUTH]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-STATUS-SYNC]]"
  - type: "refines"
    target: "[[RISK-DELIVERY]]"
  - type: "refines"
    target: "[[SCH-DELIVERY]]"
  - type: "depends_on"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "Declares ORDER_API_BASE, CARRIER_API_BASE, RETRY_COUNT — none of which any source file reads."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.eslintrc.json"
    notes: "eslint:recommended with no-console: warn; eslint is not a devDependency and there is no lint script."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "Korean four-line description; states Node 18 and 커머스본부 물류팀 ownership."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "version 1.3.0; dependencies axios and express only; scripts build and start; no test or lint script."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "Carrier proxy with 3 retries, 3s timeout, 500ms*attempt backoff, no alerting."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Generated 2022-11-08 by openapi-typescript-codegen 0.23.0; OrderStatus union, CancelRequest/CancelResponse, relative fetch."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "Single Express route GET /delivery/:ordNo; app.listen(8083); degraded 200 response."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/logger.ts"
    notes: "Three console wrappers; the entire observability surface."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
    notes: "requestCancel wrapper; comment names SF-4901 as the unstarted regeneration ticket."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tests/deliveryStatus.test.ts"
    notes: "Single vitest case asserting expect(true).toBe(true); vitest is not a dependency."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tsconfig.json"
    notes: "target ES2019, commonjs, strict false, include src only."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "@RequestMapping(\"/orders\") + @PostMapping(\"/{ordNo}/cancel\"), returns ResponseEntity<Void>."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
    notes: "Live seven-value enum used to diff the generated client's five-value union."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "CHWISO_BULGA blocks only CHWISO and BANPUM — the post-SF-2287 409 semantics."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/delivery/openapi.json"
    notes: "Tier-4 extracted spec for the one delivery-bff route, including x-outbound-calls."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.code-derived.json"
    notes: "Current order-service surface (v2.8.14) used for the generated-client diff."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "조직도 sheet — 커머스본부 물류팀 (이지훈, 6 people) listed against delivery-bff."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "owner_team is literally TODO; runtime Node 16; db none; last_reviewed 2026-03-02."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "2023-04 removal of the settled-order cancellation block, shipped in order-service 2.8.0."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/delivery/runtime.md"
    notes: "Extracted runtime sheet: stack, port, build, absent deploy assets, inert env vars."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "order-service API v2.4.0, springdoc-generated 2022-11-04 — the named source of the checked-in client."
notes: ""
---

## Overview

delivery-bff is a small Node/TypeScript service that fronts a carrier tracking API for 셀플로우's delivery lookup. It is a backend-for-frontend in the literal sense: it holds no data of its own, exposes one read endpoint, and forwards the shape the carrier returns. The README describes it in one line — "배송사 API를 프록시한다" (it proxies the carrier API) — and adds "조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다" (on lookup failure it retries three times and then returns the last stored value).

The whole repository is about 145 lines across ten files. Its interest is not its size but what it reveals. Three of its claims do not survive reading the code: it says it owns nothing yet the registry will not say who owns it; it says Node 18 while the registry says Node 16; and it says it returns a "마지막 저장 값" (last stored value) when there is nothing anywhere in the service that stores a value. The first two are contradictions between company documents; the third is a contradiction inside the code, and it is the reason [[SCH-DELIVERY]] has no entity tables to draw.

### The ownership conflict

Three documents speak, and they do not agree. This artifact records each verbatim rather than picking a winner.

- `sources/context/registry/services.yaml` — the file's own header says "이 파일이 서비스·소유팀·저장소의 단일 기준이다. 신규 서비스는 여기에 먼저 등록한다." (this file is the single standard for services, owning teams, and repositories; new services are registered here first). For delivery-bff it records `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` — transfer to the logistics team has been under discussion since November 2025 and is not confirmed. The same file carries `# last_reviewed: 2026-03-02   # 이후 갱신 없음` (no updates since).
- `sources/context/org-chart.md` (rendered from org-chart.xlsx) — the 조직도 sheet lists 본부 커머스본부, 팀 물류팀, 팀장 이지훈, 인원 6, 담당 시스템 `delivery-bff`, 담당 프로세스 "배송 추적 · 배송 예외 처리" (delivery tracking and delivery exception handling). Its 변경이력 (change-history) sheet has no row about a delivery-bff transfer at all, so the org chart does not corroborate the registry's 2025-11 discussion.
- The repository — `package.json` describes itself as "셀플로우 배송 조회 BFF. 담당: 커머스본부 물류팀" and `README.md` repeats "담당: 커머스본부 물류팀" (owner: Commerce Division Logistics Team).

So code and org chart assert an ownership the designated system of record has not confirmed. A fourth party complicates it: the org chart's 물류운영본부 배송관리팀 (권나래, 15 people) owns 배송사 관리 · 라스트마일 운영 — carrier management and last-mile operations — which is exactly the relationship this service automates, yet its 담당 시스템 column is `-`. Nothing in the reef says whether the team that manages the carriers has any say over the service that calls them. Treat delivery-bff as effectively unowned in the registry until that TODO line changes.

## Key Facts

- The service is a Node/TypeScript Express application listening on port 8083, hardcoded as `app.listen(8083)` with no env override → delivery-bff:src/index.ts
- Package version is 1.3.0 and the only runtime dependencies are axios ^1.6.2 and express ^4.18.2 → delivery-bff:package.json
- **Q-036 resolved — delivery-bff holds no persistent data of its own.** package.json declares no database driver, ORM, cache or queue client; no source file imports `fs`, and no module-level mutable variable caches a response; there is no migration directory, schema file, Dockerfile or volume; and the registry records `db: none`. The only state in the process is the `for` loop counter in `syncDeliveryStatus` → delivery-bff:package.json
- Because nothing is stored, the README's "마지막 저장 값을 반환한다" and the matching in-code comment "조회 실패 시 마지막 저장 값을 반환한다. 없으면 준비중으로 표기." (on lookup failure return the last stored value; if there is none, mark it as preparing) describe a store that does not exist — the fallback is the literal `{ ordNo, status: 'PREPARING', stale: true }` → delivery-bff:src/index.ts
- The registry records `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` while the org chart assigns delivery-bff to 커머스본부 물류팀 (이지훈, 6 people) — the two disagree and the registry is the self-declared source of truth → sellflow-docs:context/registry/services.yaml
- The registry records the runtime as `Node 16 / TypeScript` while README.md says "Node 18 / TypeScript / Express"; package.json declares no `engines` field, so nothing in the repo breaks the tie → sellflow-docs:infra/delivery/runtime.md
- Every declared configuration knob is inert: `.env.template` declares `ORDER_API_BASE`, `CARRIER_API_BASE` and `RETRY_COUNT`, but the code reads `process.env.CARRIER_API` (a different name), hardcodes `MAX_RETRY = 3`, and issues the order-service call against a relative path with no base URL at all → delivery-bff:.env.template
- **Q-035/Q-042 contributed — the generated client's path assumption was never true.** `src/generated/orderApi.ts` calls `POST /api/v1/orders/{ordNo}/cancel`, but the live `OrderController` is `@RequestMapping("/orders")` + `@PostMapping("/{ordNo}/cancel")` = `/orders/{ordNo}/cancel`, and the 2022 spec it names as its source also documents `/orders/{ordNo}/cancel`. The `/api/v1` prefix matches nothing in order-service except the unrelated `OrderSearchController` → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- The client's `OrderStatus` union — `JUMUN_WANRYO | BAESONG_JUNG | BAESONG_WANRYO | CHWISO | BANPUM` — matches neither side: the live enum has seven values (`GYEOLJE_WANRYO`, `SANGPUM_JUNBI`, `BAESONG_JUNG`, `BAESONG_WANRYO`, `JUNGSAN_WANRYO`, `CHWISO`, `BANPUM`) and `JUMUN_WANRYO` is not among them, while the 2022 spec named as the generator's source declares no `OrderStatus` schema whatsoever → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- The client's 409 handling encodes the pre-SF-2287 policy — it throws `'정산이 완료된 주문은 취소할 수 없습니다.'` (a settled order cannot be cancelled) — but since SF-2287 shipped in order-service 2.8.0 (2023-04-21) `OrderCancelService.CHWISO_BULGA` blocks only `CHWISO` and `BANPUM`, so a 409 today means "already cancelled or returned", not "settled" → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The client's `CancelResponse { ordNo, sangtaeCd }` cannot be produced by either era of the API: today `cancel` returns `ResponseEntity<Void>` (empty body, so `res.json()` would throw), and the 2022 spec returned an `OrderCancel` object with `chwisoIlsi`, `chwisoSayuCd`, `choriSangtae`, `bigo` — different field names again → sellflow-docs:raw/specs/order-service-openapi.json
- `requestCancel` in src/orderClient.ts is exported but referenced by no route in src/index.ts, so the cancel path is a library surface rather than a capability this service exposes; the file's own comment defers the fix — "재생성은 SF-4901 에서 다루기로 함. (미착수)" (regeneration will be handled in SF-4901 — not started) → delivery-bff:src/orderClient.ts
- TypeScript strict mode is disabled (`"strict": false`), the target is ES2019/commonjs, and `include` is `["src"]` only — so `tests/` is never type-checked by the build → delivery-bff:tsconfig.json
- The single test is inert in three independent ways: it imports `vitest`, which is in neither `dependencies` nor `devDependencies`; package.json has no `test` script; and its body is `expect(true).toBe(true)` under a Korean name claiming it verifies "재시도 후 마지막 저장 값을 반환한다" (returns the last stored value after retrying) → delivery-bff:tests/deliveryStatus.test.ts
- `.eslintrc.json` sets `no-console: warn`, yet eslint is not installed and no `lint` script exists — and the only console usage is `src/logger.ts`, the service's entire observability surface → delivery-bff:.eslintrc.json
- Carrier failures are silent by design: the doc comment states "3회 실패 시 포기한다. 별도 알림은 없다." (after three failures it gives up; there is no separate notification), and the route still answers HTTP 200, so an outage reaches users as "준비중" and reaches monitoring not at all → delivery-bff:src/deliveryStatus.ts

## Responsibilities

1. **Serving delivery status by order number.** `GET /delivery/:ordNo` calls `syncDeliveryStatus(req.params.ordNo)` and returns the carrier payload unmodified and unvalidated (src/index.ts, sellflow-docs:apis/delivery/openapi.json).
2. **Proxying and retrying against the carrier.** `GET ${CARRIER_API}/tracking/{ordNo}` with a 3-second axios timeout, up to three attempts, `500 * attempt` ms backoff between them — detailed in [[PROC-DELIVERY-STATUS-SYNC]] (src/deliveryStatus.ts).
3. **Degrading rather than failing.** On exhaustion the route answers HTTP 200 with the hardcoded `{ status: 'PREPARING', stale: true }` body. The `stale` boolean is the only failure signal a caller receives — see [[PROC-DELIVERY-ERROR-HANDLING]] (src/index.ts).
4. **Holding a wrapper for order cancellation.** `requestCancel(ordNo, sayuCd, bigo)` delegates to the generated `cancelOrder`. Nothing calls it (src/orderClient.ts).

## Does NOT Own

Recorded because each absence is a finding, and because [[CON-ORDER-DELIVERY]] and [[RISK-DELIVERY]] depend on them being explicit.

- **No database, cache or file state.** See the Q-036 fact above. The registry agrees (`db: none`), and nothing in the repo contradicts it.
- **No delivery entity.** `ORDER_DELIVERY`, 운송장번호 (`unSongJangBeonho`) and 택배사코드 (`taekBaeSaCd`) live in order-service's schema, not here; this service never reads or writes them and never contacts order-service to fetch them ([[SCH-DELIVERY]], sellflow-docs:raw/specs/order-service-openapi.json).
- **No order state.** It cannot set `SANGTAE_CD`; the only order mutation it can even express is the unreachable cancel wrapper, and `JUNGSAN_WANRYO` is written solely by settlement-batch.
- **No carrier relationship.** The carrier's identity, contract and SLA are outside the repo; the org chart puts 배송사 관리 with 물류운영본부 배송관리팀, a team with no listed system.
- **No authentication, authorization or rate limiting.** There is no middleware beyond `express.json()`; anyone who can reach port 8083 can query any 주문번호 ([[PROC-DELIVERY-AUTH]]).
- **No alerting, metrics, tracing or health endpoint.** `src/logger.ts` wraps `console` and that is all.
- **No build, release or deployment definition.** No Dockerfile, Helm chart, compose file or CI workflow exists in the repo (sellflow-docs:infra/delivery/runtime.md).

## Core Concepts

- **BFF.** One aggregation layer in front of the carrier for a client application. Defined with the rest of the domain vocabulary in [[GLOSSARY-DELIVERY]].
- **ordNo as the join key.** Everything keys on 주문번호 (order number), the same identifier order-service uses for `OrderMst` — see [[CON-ORDER-DELIVERY]].
- **Transient contract types.** `DeliveryStatus { ordNo, carrierCd, status, updatedAt }` and the generated order types are in-flight shapes, not stored records; [[SCH-DELIVERY]] explains why no ER diagram is warranted.
- **The stale flag.** The single boolean that distinguishes a real carrier answer from a fabricated one, on a response whose HTTP status is 200 either way ([[API-DELIVERY]]).
- **A generated client frozen in 2022.** `src/generated/orderApi.ts` carries the header "자동 생성 파일입니다. 직접 수정하지 마세요." (this is an automatically generated file; do not edit it directly), `generator: openapi-typescript-codegen 0.23.0`, `generated: 2022-11-08T04:12:33Z`. The don't-edit instruction plus an unstarted regeneration ticket is why nobody has fixed it — the subject of [[DEC-DELIVERY-GENERATED-CLIENT]].

## Dependencies

| System | Integration Type | Purpose | Auth Method |
|---|---|---|---|
| Carrier tracking API (unidentified) | Outbound HTTP (axios GET `${CARRIER_API}/tracking/{ordNo}`, 3s timeout, 3 attempts) | The sole data source for delivery status; its response body is returned to the caller verbatim | **None.** No API key, header, or signature is set anywhere in src/deliveryStatus.ts |
| order-service ([[SYS-ORDER]]) | Outbound HTTP via the checked-in generated client (`POST /api/v1/orders/{ordNo}/cancel`) | Request order cancellation — nominally; no route reaches `requestCancel`, and the path does not match the live controller | **None.** The generated `fetch` sends only `Content-Type: application/json`; the 2022 spec documented optional `X-Request-Id` / `X-Client-Ver` headers and a 401 response, none of which the client implements |
| Callers of `GET /delivery/:ordNo` | Inbound HTTP | Unknown — no client list, gateway config, CORS policy or host allowlist exists in the repo | **None.** No middleware beyond `express.json()`; see [[PROC-DELIVERY-AUTH]] |

The asymmetry matters: delivery-bff depends on order-service's *identifier* (ordNo) continuously and on order-service's *API* never — the one coded integration is unreachable. [[CON-ORDER-DELIVERY]] carries the boundary analysis.

## Domain Behavior Highlights

**It is a proxy with no store, and it says otherwise.** The service's most load-bearing sentence appears three times — in README.md, in the comment above the fallback in src/index.ts, and in the name of the only test — and in all three places it claims a "마지막 저장 값" (last stored value). There is no store (Q-036 above). What actually ships is a constant: any order whose carrier lookup fails three times is reported as `PREPARING`, whatever its real state. An order that is 배송완료 (delivered) and an order that was never shipped produce identical responses during a carrier outage, and only the `stale: true` flag — which no documented consumer is known to read — distinguishes either from a genuine answer. Callers that trust `status` will regress a delivery backwards.

**A generated client compiled against a spec that has moved — and that never quite matched.** The interesting finding is not simply that the 2022 client is stale; it is that regenerating it would not have produced this file. Diffing `src/generated/orderApi.ts` against `sources/raw/specs/order-service-openapi.json` (the file its header names as `source:`, springdoc-generated 2022-11-04, order-service v2.4.0) and against today's `sources/apis/order/openapi.code-derived.json` (v2.8.14):

| Assumption in the generated client | 2022 spec it names as its source | order-service today |
|---|---|---|
| Path `POST /api/v1/orders/{ordNo}/cancel` | `/orders/{ordNo}/cancel`; servers are bare hosts with no `/api/v1` base | `/orders/{ordNo}/cancel` — `/api/v1` exists only for `OrderSearchController` (`/api/v1/orders/search`) |
| Response `CancelResponse { ordNo, sangtaeCd }` parsed with `res.json()` | `OrderCancel { ordNo, chwisoIlsi, chwisoSayuCd, choriSangtae, bigo }` | `ResponseEntity<Void>` — empty body |
| `OrderStatus = JUMUN_WANRYO \| BAESONG_JUNG \| BAESONG_WANRYO \| CHWISO \| BANPUM` | No `OrderStatus` schema exists in the spec at all | Seven values; no `JUMUN_WANRYO`; adds `GYEOLJE_WANRYO`, `SANGPUM_JUNBI`, `JUNGSAN_WANRYO` |
| 409 means "정산이 완료된 주문은 취소할 수 없습니다" | 409 = "취소 불가 상태 (배송완료·정산완료 등)" — consistent with the client | 409 = already `CHWISO` or `BANPUM` only; settled orders are cancellable since SF-2287 |
| `sayuCd: string`, free-form | `sayuCd` required, no enum | Enum `01`–`04` (파트너 귀책 / 시스템 오류 / 고객 변심 / 배송 실패); an unknown code raises `IllegalArgumentException` in `CancelReason.of` and surfaces as 500 |
| No auth or tracing headers sent | Documents optional `X-Request-Id`, `X-Client-Ver` and a 401 response | Not re-verified in this pass |

So two failures compound. The client drifted from the API (rows 3–5, the SF-2287 consequence), and it was wrong on the day it was written (rows 1–2, which no version of the spec supports). On top of that, `fetch` is called with a relative path and no base URL, which cannot resolve in a Node process under either candidate runtime. The net effect is that `requestCancel` would fail before it ever reached order-service — which is consistent with the fact that nothing calls it and nobody has noticed. Recorded as a decision in [[DEC-DELIVERY-GENERATED-CLIENT]] and as risk in [[RISK-DELIVERY]].

**The carrier integration is the whole service, and it is unnamed and unauthenticated.** `CARRIER_API` defaults to `https://api.carrier.example` — a reserved example domain — while `.env.template` offers a *different* placeholder, `https://api.carrier.example.co.kr`, under a *different* variable name (`CARRIER_API_BASE`) that the code never reads. No credential is attached to the outbound request. Retry is fixed at three attempts despite `RETRY_COUNT=3` being declared in the template, so operators believe they have a knob they do not have. The failure handling is deliberate and documented — "별도 알림은 없다" — meaning the service converts a dependency outage into a plausible-looking wrong answer rather than an error anyone can page on. [[PROC-DELIVERY-ERROR-HANDLING]] traces each branch.

## Runtime Components

| Component | Tech Stack | Purpose | Entry Point |
|---|---|---|---|
| HTTP server | Express 4.18 on Node (16 or 18 — unresolved), TypeScript 5.3 compiled to commonjs/ES2019 | The single process; registers one route and `express.json()`, listens on hardcoded port 8083 | `src/index.ts` → `node dist/index.js` (`npm start`, after `npm run build` = `tsc`) |
| Carrier status client | axios 1.6, 3s timeout, `MAX_RETRY = 3`, `500 * attempt` ms backoff | Fetches `${CARRIER_API}/tracking/{ordNo}`; returns `DeliveryStatus` or `null` | `syncDeliveryStatus()` in `src/deliveryStatus.ts` |
| Order cancel client (unreachable) | Global `fetch`, no base URL, hand-rolled 409 branch | Would POST a cancellation to order-service; unreferenced by any route | `requestCancel()` in `src/orderClient.ts` → `cancelOrder()` in `src/generated/orderApi.ts` |
| Logger | Three arrow functions wrapping `console.log` / `warn` / `error` with `[INFO]` / `[WARN]` / `[ERROR]` prefixes | The service's entire observability surface; `error` is exported but never called | `src/logger.ts` |
| Test suite (inert) | vitest — not installed, no `test` script, excluded from `tsconfig.include` | Nominally covers the retry fallback; asserts `expect(true).toBe(true)` | `tests/deliveryStatus.test.ts` |
| Lint config (inert) | `eslint:recommended`, `no-console: warn` | Would flag `src/logger.ts`; eslint is not a dependency and no script invokes it | `.eslintrc.json` |

No Dockerfile, Helm chart, compose file, CI workflow, `.nvmrc` or `engines` field exists, so nothing in the repo describes how this process is built into an artifact or placed on a host (sellflow-docs:infra/delivery/runtime.md).

## Related

- [[API-DELIVERY]] — the exposed and consumed API surface, including the degraded response
- [[CON-ORDER-DELIVERY]] — the boundary with the order side
- [[DEC-DELIVERY-GENERATED-CLIENT]] — the decision to check in a generated client and defer regeneration to SF-4901
- [[GLOSSARY-DELIVERY]] — delivery vocabulary, Korean and romanized
- [[PROC-DELIVERY-AUTH]] — authentication, of which there is none
- [[PROC-DELIVERY-ERROR-HANDLING]] — what is caught, retried, swallowed and never alerted
- [[PROC-DELIVERY-STATUS-SYNC]] — the retry and degradation workflow
- [[RISK-DELIVERY]] — stale generated client, dead config, absent tests and CI
- [[SCH-DELIVERY]] — the transient data shapes and why there is no ERD
- [[SYS-ORDER]] — the upstream order system this service keys on and mirrors
