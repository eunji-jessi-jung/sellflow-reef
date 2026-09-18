---
id: "API-DELIVERY"
type: "api"
title: "Delivery BFF API Surface"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Derived by reading src/index.ts, src/deliveryStatus.ts, src/orderClient.ts and src/generated/orderApi.ts on 2026-09-18. The service publishes no OpenAPI document of its own, so this artifact and the tier-4 extraction record in sources/apis/delivery/ are the only endpoint inventory; it goes stale on any route change. Reconciled on 2026-09-18 against openapi.meta.json: the generated client's stale OrderStatus union, the SF-4901 regeneration ticket and the two unread .env.template variables were folded in. The extraction contradicted nothing here; it upgraded the cancel-path mismatch from an open question to a confirmed finding. Deepened on 2026-09-19 with a surface profile, an explicitly-verified authentication posture, the contract limits the framework does and does not enforce, a fuller worked example of the stale fallback including its wall-clock cost, a point-by-point write-up of the outbound order-service contract the generated client encodes, and agent guidance."
freshness_triggers:
  - "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "sources/raw/specs/order-service-openapi.json"
  - "src/deliveryStatus.ts"
  - "src/generated/orderApi.ts"
  - "src/index.ts"
  - "src/orderClient.ts"
known_unknowns:
  - "delivery-bff publishes no OpenAPI or schema document of its own; the endpoint list here is read from route registrations in src/index.ts and nothing validates it."
  - "The carrier API's contract is unknown — no spec, no vendor documentation, no error catalogue. Only the request shape GET {CARRIER_API}/tracking/{ordNo} is visible."
  - "The success response's HTTP status is whatever express defaults res.json to (200); no explicit status codes are set anywhere in src/index.ts."
  - "No error path other than the stale fallback exists: a malformed ordNo, an unknown order, and a carrier outage are indistinguishable to a caller."
  - "The base URL the generated client resolves against is unknown. cancelOrder fetches the relative path /api/v1/orders/{ordNo}/cancel with no origin, so in Node 18 it would need a global base that the repo never configures."
  - "Whether a gateway rewrites the /api/v1 prefix the generated client sends onto the /orders/{ordNo}/cancel path OrderController actually maps. The mismatch itself is confirmed; a rewriting component is not documented anywhere."
  - "No versioning, deprecation, rate limit, pagination, or CORS policy is expressed for GET /delivery/:ordNo."
tags:
  - "delivery"
  - "api"
  - "express"
  - "bff"
aliases:
  - "delivery-bff endpoints"
relates_to:
  - type: "integrates_with"
    target: "[[API-ORDER]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "depends_on"
    target: "[[PROC-DELIVERY-AUTH]]"
  - type: "depends_on"
    target: "[[PROC-DELIVERY-STATUS-SYNC]]"
  - type: "constrains"
    target: "[[RISK-DELIVERY]]"
  - type: "refines"
    target: "[[SCH-DELIVERY]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "Three variables; two are read by nothing and the third is spelled differently in code."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "Claims Node 18 and repeats the last-stored-value promise the code does not keep."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "Two runtime dependencies, axios and express; no auth, validation or test-runner dependency."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "Outbound carrier call."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Generated cancelOrder client, 2022-11-08."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "The single exposed route and the degraded response."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tests/deliveryStatus.test.ts"
    notes: "One assertion, expect(true).toBe(true), with an uninstalled runner."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tsconfig.json"
    notes: "strict: false, which is why the degraded literal is never checked against DeliveryStatus."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "The live cancel endpoint the generated client is supposed to be calling."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
    notes: "The live seven-value status enum the generated union is measured against."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "Shows which states actually produce a 409 today: CHWISO and BANPUM only."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/delivery/openapi.json"
    notes: "Reef-relative. Tier-4 reconstruction of the one route, plus the x-outbound-calls block."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/delivery/openapi.meta.json"
    notes: "Reef-relative. Tier-4 extraction record: 1 endpoint, plus the stale-client and dead-env-var findings."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Reef-relative. delivery-bff's owner_team: TODO and the Node 16 runtime claim."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "Reef-relative. Dates the policy change the generated client's 409 message predates."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "order-service API 2.4.0, generated 2022-11-04; declares global bearerAuth and documents /orders/{ordNo}/cancel with no /api/v1 prefix."
notes: "The sources list is sorted by ref; delivery-bff refs are relative to the delivery-bff repo root, order-service refs to the order-service repo root."
---

## Overview

The API surface of delivery-bff is one endpoint wide. `GET /delivery/:ordNo` is the whole exposed contract; everything else in the repository is outbound — one call to a carrier tracking API and one generated client for order cancellation.

The part worth dwelling on is the failure response. When the carrier lookup gives up, the endpoint does not return 502 or 504. It returns HTTP 200 with a body the code describes as the last stored value, even though this service stores nothing. That gap between the comment and the code is the most load-bearing thing in this artifact.

## Surface Profile

| Property | Value | Evidence |
|---|---|---|
| Style | Read-only JSON proxy; one GET, no collections, no writes | src/index.ts |
| Framework | Express 4.18 on Node, TypeScript 5.3 compiled to CommonJS, ES2019 target | package.json, tsconfig.json |
| Runtime | README says Node 18; the service registry says Node 16. Unresolved | README.md, sources/context/registry/services.yaml |
| App identity | `"name": "delivery-bff", "version": "1.3.0"` | package.json |
| Base URL | `http://delivery-bff.internal:8083` — the extraction's inference from `app.listen(8083)`; the repo sets no hostname | src/index.ts, sources/apis/delivery/openapi.json |
| Port | 8083, a hardcoded literal in `app.listen` with no env override | src/index.ts |
| Exposed endpoints | 1 | src/index.ts |
| Outbound calls | 2 — one carrier, one order-service (the latter unreachable from any route) | src/deliveryStatus.ts, src/generated/orderApi.ts |
| Authentication | None inbound, none outbound — see below | src/index.ts, src/deliveryStatus.ts, src/generated/orderApi.ts |
| Media type | `application/json` out; no request body is ever read | src/index.ts |
| Field casing | camelCase throughout (`ordNo`, `carrierCd`, `updatedAt`) | src/deliveryStatus.ts |
| Versioning | None. The path carries no version and no header is negotiated | src/index.ts |
| Health / readiness / metrics | None. Contrast order-service's `/internal/health`, `/internal/ready`, `/internal/metrics` | src/index.ts, sources/raw/specs/order-service-openapi.json |
| Published spec | None | sources/apis/delivery/openapi.meta.json |
| Tests | One file, one assertion: `expect(true).toBe(true)`. It imports from `vitest`, which appears in neither `dependencies` nor `devDependencies`, and `package.json` declares no `test` script — so the suite cannot even be run | tests/deliveryStatus.test.ts, package.json |

### Authentication and Authorization Posture

**There is no authentication anywhere in this service — not on the inbound route, not on the
carrier call, not on the order-service call.** Stated in full because the third of those is the
interesting one: the service it calls *does* demand a credential on paper.

How it was verified:

1. **Inbound.** The route is `app.get('/delivery/:ordNo', async (req, res) => {...})`. No
   middleware precedes it except `express.json()`; there is no `passport`, no session, no
   `req.headers.authorization` read, and no guard function anywhere in `src/` (src/index.ts).
2. **Dependency surface.** `package.json` lists two runtime dependencies in total, `axios` and
   `express`, and two dev dependencies, `typescript` and `@types/express`. No JWT, session,
   OAuth, API-key or validation library is installed (package.json).
3. **Configuration.** `.env.template` defines three variables — `ORDER_API_BASE`,
   `CARRIER_API_BASE`, `RETRY_COUNT` — none of which is a secret, a key or a token. There is no
   other configuration file (.env.template).
4. **Outbound to the carrier.** `axios.get(url, { timeout: 3000 })` — a two-key options object
   with no `headers` and no `auth` (src/deliveryStatus.ts). A vendor tracking API that required a
   key would reject every call, which is one of the ways the fallback path could be permanently
   live without anyone noticing.
5. **Outbound to order-service.** `fetch(url, { method: 'POST', headers: { 'Content-Type':
   'application/json' }, body: ... })` — exactly one header, and it is not `Authorization`
   (src/generated/orderApi.ts). The order-service spec declares a document-level
   `security: [{ "bearerAuth": [] }]` with `type: http`, `scheme: bearer`, `bearerFormat: JWT`
   (sources/raw/specs/order-service-openapi.json). The generated client would therefore be
   rejected on authentication before its path mismatch even mattered — two independent reasons
   the same call cannot succeed.

There is no authorization either: `ordNo` is taken from the path and used immediately, with no
check that the caller is entitled to see that order. Any party that can reach port 8083 can read
the delivery status of any order number it can guess, and order numbers are sequential-looking
(`20260918000123`). The reasoning about network placement belongs to [[PROC-DELIVERY-AUTH]].

### Contract Limits

| Limit | Enforced? | Detail |
|---|---|---|
| `ordNo` validation | **No** | Taken straight from `req.params.ordNo` and interpolated into the carrier URL with no trim, no length check, no character allowlist and no `encodeURIComponent` (src/index.ts, src/deliveryStatus.ts) |
| Response validation | **No** | `return res.data as DeliveryStatus` is a compile-time cast with no runtime check; whatever the carrier sends is forwarded (src/deliveryStatus.ts) |
| Response shape stability | **No** | Two different shapes come out of one endpoint, and the fallback drops `carrierCd` and `updatedAt` (src/index.ts) |
| Error status codes | **None used** | No `res.status(...)` call exists anywhere; both outcomes are express's default 200 (src/index.ts) |
| Upstream timeout | **Yes** | 3000 ms per attempt (src/deliveryStatus.ts) |
| Retries | **Yes, fixed** | `MAX_RETRY = 3`, a module constant; `.env.template`'s `RETRY_COUNT` is read by nothing (src/deliveryStatus.ts, .env.template) |
| Backoff | **Yes, linear** | `setTimeout(r, 500 * attempt)` — 500 ms then 1000 ms, no jitter (src/deliveryStatus.ts) |
| Worst-case latency | **Not bounded by the route** | ~10.5 s before the fallback returns; no overall request deadline and no circuit breaker (src/deliveryStatus.ts) |
| Caching | **No** | Every request hits the carrier; no cache, no `Cache-Control`, no ETag (src/index.ts) |
| Rate limiting, CORS, request size, compression | **No** | `express.json()` is the only middleware, and no route reads a body (src/index.ts) |
| Pagination | N/A | No collection endpoint |
| Type safety | **Weakened** | `"strict": false` in tsconfig.json, so the fallback literal is never checked against `DeliveryStatus` (tsconfig.json) |
| Alerting on degradation | **No** | Three `logger.warn` lines to stdout; the code says so itself — "별도 알림은 없다" ("there is no separate notification") (src/deliveryStatus.ts, src/logger.ts) |

The two that most change a caller's behaviour: there is no status code to key off, and there is no
bound on how long the endpoint may take before it answers with a fabricated value.

## Key Facts

- The service exposes exactly one route: `app.get('/delivery/:ordNo', ...)` → src/index.ts
- The server listens on port 8083 → src/index.ts
- `express.json()` is registered although the only route takes no request body → src/index.ts
- On success the route returns the carrier payload unmodified via `res.json(status)` → src/index.ts
- On failure the route returns HTTP 200 with `{ ordNo, status: 'PREPARING', stale: true }` → src/index.ts
- The fallback `'PREPARING'` is a hardcoded literal in the route handler, not a value read from any store → src/index.ts
- The route comment claims "조회 실패 시 마지막 저장 값을 반환한다. 없으면 준비중으로 표기" (on lookup failure return the last stored value; if absent, mark as preparing) although the service has no store → src/index.ts
- The consumed carrier endpoint is `GET ${CARRIER_API}/tracking/{ordNo}` with a 3000 ms timeout → src/deliveryStatus.ts
- The consumed order endpoint is `POST /api/v1/orders/{ordNo}/cancel` issued by the generated client → src/generated/orderApi.ts
- The generated client throws "정산이 완료된 주문은 취소할 수 없습니다" (an order whose settlement is complete cannot be cancelled) on HTTP 409 → src/generated/orderApi.ts
- The order-service OpenAPI spec declares a global `security: [{bearerAuth: []}]` with JWT bearer format, and the generated client sends only a Content-Type header → sources/raw/specs/order-service-openapi.json
- No route handler in src/index.ts calls `requestCancel`, so the cancellation path is unreachable through the HTTP surface → src/index.ts
- The generated client's path `/api/v1/orders/{ordNo}/cancel` does not match today's server: `OrderController` is `@RequestMapping("/orders")` plus `POST /{ordNo}/cancel`, which is `/orders/{ordNo}/cancel` with no `/api/v1` prefix → sources/apis/delivery/openapi.meta.json, see [[API-ORDER]]
- The same client's `OrderStatus` union is 2022-era — `'JUMUN_WANRYO' | 'BAESONG_JUNG' | 'BAESONG_WANRYO' | 'CHWISO' | 'BANPUM'` — missing `GYEOLJE_WANRYO`, `SANGPUM_JUNBI` and `JUNGSAN_WANRYO`, and carrying `JUMUN_WANRYO`, which the server enum no longer contains → src/generated/orderApi.ts, sources/apis/delivery/openapi.meta.json
- Regenerating the client is a known, unstarted task: the wrapper's javadoc says "재생성은 SF-4901 에서 다루기로 함. (미착수)" ("regeneration is to be handled in SF-4901; not started") → src/orderClient.ts
- `.env.template` defines three variables and the code reads none of them as written: `CARRIER_API_BASE` versus the `CARRIER_API` that `deliveryStatus.ts` actually reads, `ORDER_API_BASE` which nothing reads, and `RETRY_COUNT` which nothing reads because `MAX_RETRY` is the hardcoded literal `3` → .env.template, src/deliveryStatus.ts, sources/apis/delivery/openapi.meta.json
- No `res.status(...)` call exists anywhere in the service, so every response — success or degraded — carries express's default 200 → src/index.ts
- No authentication library is installed: `package.json` declares exactly two runtime dependencies, `axios` and `express`, and two dev dependencies, `typescript` and `@types/express` → package.json
- The carrier call attaches no headers at all — `axios.get(url, { timeout: 3000 })` has no `headers` and no `auth` key → src/deliveryStatus.ts
- `ordNo` reaches the carrier URL unvalidated and unencoded: `req.params.ordNo` is passed to `syncDeliveryStatus`, which interpolates it into a template literal with no `encodeURIComponent` → src/index.ts, src/deliveryStatus.ts
- The worst case before the fallback is roughly 10.5 seconds — three 3000 ms timeouts plus 500 ms and 1000 ms of linear backoff — and no overall request deadline or circuit breaker caps it → src/deliveryStatus.ts
- Retry behaviour is a module constant, not configuration: `MAX_RETRY = 3`, while `.env.template`'s `RETRY_COUNT=3` is read by nothing → src/deliveryStatus.ts, .env.template
- The live cancel endpoint the generated client targets is `@RequestMapping("/orders")` plus `@PostMapping("/{ordNo}/cancel")`, returning `ResponseEntity<Void>` — an empty body, where the client does `return res.json()` → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java, src/generated/orderApi.ts
- The 409 the generated client hardcodes no longer means what its message says: `OrderCancelService` blocks only `CHWISO` and `BANPUM` — `private static final Set<OrderStatus> CHWISO_BULGA = EnumSet.of(OrderStatus.CHWISO, OrderStatus.BANPUM)`, documented as "이미 취소되었거나 반품 프로세스로 넘어간 주문만 차단한다" ("only orders already cancelled or moved into the return process are blocked") — with `JUNGSAN_WANRYO` (정산완료, settlement complete) absent from the set → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The live `OrderStatus` enum has seven constants — GYEOLJE_WANRYO, SANGPUM_JUNBI, BAESONG_JUNG, BAESONG_WANRYO, JUNGSAN_WANRYO, CHWISO, BANPUM — against the generated client's five → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java, src/generated/orderApi.ts
- The one test file cannot run: it imports `describe, it, expect` from `vitest`, which appears in neither `dependencies` nor `devDependencies`, and `package.json` declares only `build` and `start` scripts → tests/deliveryStatus.test.ts, package.json
- The extraction could not reach a runtime tier: there is no lockfile and no `node_modules`, so express, axios and typescript cannot be resolved, and Express offers no OpenAPI reflection in any case → sources/apis/delivery/openapi.meta.json

## Source of Truth

There is no specification for this API. delivery-bff ships no OpenAPI document, no JSON Schema, and no route test — `tests/deliveryStatus.test.ts` contains only `expect(true).toBe(true)`. The route registration in `src/index.ts` is the source of truth, and this artifact is transcribed from it. The reef's own `sources/apis/delivery/openapi.json` is a tier-4 description written by reading those same route registrations, not a document the service publishes; its record `openapi.meta.json` counts one endpoint and lists the findings folded into this artifact → sources/apis/delivery/openapi.meta.json

For the consumed order API there is a spec — `sources/raw/specs/order-service-openapi.json`, version 2.4.0, generated 2022-11-04 by springdoc-openapi — but it is the same 2022 vintage that produced the stale client, and it too documents a cancellation policy the server has since dropped (see [[RISK-DELIVERY]]). For the carrier API there is nothing at all.

## Resource Map

### Exposed

| Method | Path | Handler | Auth | Success | Failure |
|--------|------|---------|------|---------|---------|
| GET | `/delivery/:ordNo` | src/index.ts | none ([[PROC-DELIVERY-AUTH]]) | 200, DeliveryStatus body | 200, degraded body with `stale: true` |

Path parameter `ordNo` — 주문번호 (order number). Passed straight into the carrier URL with no validation, trimming, or encoding.

There is no health, readiness, or metrics endpoint, which contrasts with order-service's `/internal/health`, `/internal/ready`, and `/internal/metrics` (sources/raw/specs/order-service-openapi.json).

### Consumed

| Target | Method | Path | Client | Timeout | Notes |
|--------|--------|------|--------|---------|-------|
| Carrier tracking API | GET | `${CARRIER_API}/tracking/{ordNo}` | axios, src/deliveryStatus.ts | 3000 ms | Retried up to 3 times; see [[PROC-DELIVERY-STATUS-SYNC]] |
| order-service | POST | `/api/v1/orders/{ordNo}/cancel` | generated fetch, src/generated/orderApi.ts | none set | Relative path, no origin, no Authorization header |

`CARRIER_API` defaults to `https://api.carrier.example` when unset — a placeholder host, and the environment variable the template actually defines is a different name (`CARRIER_API_BASE`), which is why the default is likely what runs. That mismatch is documented in [[PROC-DELIVERY-STATUS-SYNC]].

### The degraded response

This deserves to be stated plainly. `src/index.ts` reads:

```ts
const status = await syncDeliveryStatus(req.params.ordNo);
if (!status) {
  // 조회 실패 시 마지막 저장 값을 반환한다. 없으면 준비중으로 표기.
  return res.json({ ordNo: req.params.ordNo, status: 'PREPARING', stale: true });
}
```

The comment — "on lookup failure return the last stored value; if absent, mark as preparing" — describes a cache-or-last-known-value strategy. The README repeats the promise: "조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다" (on lookup failure, retry three times and then return the last stored value). Neither is what the code does. There is no store to read a last value from; `'PREPARING'` is a literal typed into the handler. Every failed lookup, for every order, in every delivery state, returns the same word.

The consequences a caller should know about:

- The HTTP status is 200, so status-code-based error handling never fires. `stale: true` is the only signal, and a client that ignores that field cannot tell a real answer from a fabricated one.
- The reported state is optimistic in one direction and wrong in the other: an order already 배송완료 (delivered) reads back as PREPARING during a carrier outage.
- The body omits `carrierCd` and `updatedAt`, so a client typed strictly against `DeliveryStatus` gets an object that does not satisfy the interface. TypeScript would normally catch this on the server side, but `"strict": false` in tsconfig.json means the inline literal is never checked against the interface at all.

### Worked Example

**Success.** The carrier answers on the first attempt:

```http
GET /delivery/20260918000123 HTTP/1.1
Host: delivery-bff.internal:8083
```

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```
```json
{
  "ordNo": "20260918000123",
  "carrierCd": "CJ",
  "status": "BAESONG_JUNG",
  "updatedAt": "2026-09-18T09:12:00+09:00"
}
```

The body is the carrier's body. `syncDeliveryStatus` does `return res.data as DeliveryStatus` with no field mapping or validation, so anything extra the carrier sends is forwarded to the client too.

**Stale fallback.** The same request while the carrier is unreachable — three attempts, roughly 3 s + 0.5 s + 3 s + 1 s + 3 s of wall clock before the answer:

```http
GET /delivery/20260918000123 HTTP/1.1
Host: delivery-bff.internal:8083
```

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```
```json
{
  "ordNo": "20260918000123",
  "status": "PREPARING",
  "stale": true
}
```

Identical for any `ordNo`. The three `[WARN] 배송 상태 조회 실패 (n/3) ordNo=...` lines that `src/logger.ts` writes to stdout are the only record that this happened — the code comment in src/deliveryStatus.ts confirms "별도 알림은 없다" (there is no separate notification).

**Consumed cancellation.** `requestCancel('20260918000123', '02', '오배송')` produces:

```http
POST /api/v1/orders/20260918000123/cancel HTTP/1.1
Content-Type: application/json
```
```json
{ "sayuCd": "02", "bigo": "오배송" }
```

No `Authorization` header is attached despite the spec's global bearerAuth, and no `X-Request-Id` despite the spec offering it on 29 of its 32 operations. If order-service answers 409 the client throws `Error('정산이 완료된 주문은 취소할 수 없습니다.')` — a rule that SF-2287 removed from the server in April 2023, so the client would be reporting a policy that no longer exists. See [[RISK-DELIVERY]] and [[CON-ORDER-DELIVERY]].

## The Outbound Contract to order-service

`src/generated/orderApi.ts` is not documentation of order-service — it is a *frozen belief* about
order-service, dated `2022-11-08T04:12:33Z` and generated by `openapi-typescript-codegen 0.23.0`
from `order-service-openapi.json`. Everything it encodes was true of some 2022 version of that
service. Here is each assumption, and where it now diverges.

| # | What the client encodes | What order-service does today | Divergence |
|---|---|---|---|
| 1 | Path `/api/v1/orders/{ordNo}/cancel` | `@RequestMapping("/orders")` + `@PostMapping("/{ordNo}/cancel")` = `/orders/{ordNo}/cancel` | **Diverged.** The `/api/v1` prefix exists on neither the controller nor the 2022 spec's own paths, which are already `/orders/{ordNo}/cancel` — so the prefix was added by the generator's base-path configuration, not by the server |
| 2 | Method POST | POST | Agrees |
| 3 | Request body `CancelRequest { sayuCd: string; bigo?: string }` | `OrderController.CancelRequest` with `getSayuCd()`/`getBigo()` | **Agrees** — field names and optionality both match. The one genuinely correct part of the contract |
| 4 | `sayuCd` unconstrained `string` | `CancelReason.of(code)` throws `IllegalArgumentException("알 수 없는 취소 사유 코드: ...")` for anything outside `01`–`04`; the controller javadoc says "사유 코드(01~04)" | **Under-specified.** The client cannot prevent an invalid code; the server converts it into a 500-class failure, not a 400 |
| 5 | Response body `CancelResponse { ordNo; sangtaeCd }`, obtained by `return res.json()` | `ResponseEntity<Void>` — `ResponseEntity.ok().build()`, an empty body | **Diverged.** `res.json()` on an empty body rejects; the declared return type has no counterpart on the server at all |
| 6 | `sangtaeCd` typed as the five-value `OrderStatus` union | A seven-value enum, and this field is not returned | **Doubly diverged.** See the enum table in [[SCH-DELIVERY]] |
| 7 | `if (res.status === 409) throw new Error('정산이 완료된 주문은 취소할 수 없습니다.')` — "an order whose settlement is complete cannot be cancelled" | 409 arises from `OrderCancelNotAllowedException`, thrown only when the status is `CHWISO` or `BANPUM`. `JUNGSAN_WANRYO` is deliberately not in `CHWISO_BULGA` | **Inverted.** The one condition the client names as the cause of 409 is the one condition that no longer causes it. A settled order now cancels successfully; an already-cancelled or returned order is what 409 means |
| 8 | No `Authorization` header | Spec declares document-level `bearerAuth` (http/bearer/JWT) | **Diverged** — or the spec overstates the server; nothing in `OrderController` or `WebConfig` reads a token either, so which side is wrong is not resolvable here |
| 9 | No `X-Request-Id` | The 2022 spec offers `X-Request-Id` on every path | Omitted; correlation is impossible across the hop |
| 10 | Relative URL with no origin — `fetch('/api/v1/orders/...')` | — | **Structurally broken.** In Node there is no document base, so the fetch has nothing to resolve against. `.env.template`'s `ORDER_API_BASE=http://order-service.internal` exists for exactly this purpose and is read by no code |

Item 7 is the one to carry away, because it is the reason this file matters beyond tidiness. The
client's error message is the **pre-SF-2287 policy**. SF-2287 (created 2023-04-03, resolved
2023-04-21, fix version order-service 2.8.0) removed the block on cancelling settled orders; the
client still tells its caller that the block exists. A reader who takes `orderApi.ts` as a
description of the system gets the single most consequential rule in the settlement domain exactly
backwards — and it is the rule the whole `CANCEL_RECON_QUEUE` backlog hangs off. See
[[CON-ORDER-DELIVERY]] and [[RISK-DELIVERY]].

Two mitigating facts, and they are only mitigating. First, `requestCancel` is exported from
`src/orderClient.ts` and called by nothing — no route in `src/index.ts` reaches it — so none of
the ten divergences is exercised at runtime. Second, the wrapper's own javadoc admits the problem:
"생성 클라이언트가 2022 스펙 기준이라 상태코드 목록이 현재와 다르다. 재생성은 SF-4901 에서
다루기로 함. (미착수)" — "the generated client is based on the 2022 spec, so its status-code list
differs from the present one; regeneration is to be handled in SF-4901 (not started)". The note
identifies only the status-code drift. It does not mention the path prefix, the inverted 409
semantics, the empty response body or the missing origin — four defects that the acknowledged one
sits on top of.

## How Agents Should Use This

**Answering questions.**

- *"What does the delivery API return when the carrier is down?"* — HTTP 200 with
  `{ordNo, status: "PREPARING", stale: true}`. Say "fabricated", not "cached": the README and the
  code comment both promise a last-stored value, and there is no store. Point at `stale: true` as
  the only signal a caller has.
- *"Can delivery-bff cancel an order?"* — No. The capability exists as code and is unreachable:
  `requestCancel` has no caller, and the client it wraps is broken in at least four independent
  ways. Do not describe it as an integration that works.
- *"Is the delivery API authenticated?"* — No, in any direction. Reproduce the five-step check
  above rather than asserting it.
- *"What are the order statuses?"* — Never answer from `src/generated/orderApi.ts`. The
  authoritative list is `order-service:.../domain/OrderStatus.java`; see [[API-ORDER]].

**Before changing anything.**

- Turning the fallback into a real 502/504 is a breaking change for any caller currently reading
  200 + `stale`. There is no consumer inventory in the reef, so that blast radius is unknown —
  record it as a question before acting.
- The env-var mismatch (`CARRIER_API_BASE` in the template versus `CARRIER_API` in the code) means
  the service is most likely running against the placeholder default
  `https://api.carrier.example`. Fixing the variable name is not cosmetic; it may be the change
  that first makes real carrier calls happen, with all the behaviour that implies.
- Regenerating the client (SF-4901) fixes items 3, 5 and 6 of the table above only if it is
  regenerated against the *current* server, not against `sources/raw/specs/order-service-openapi.json`,
  which is the same 2022 document that produced the problem.

**Traps to avoid.**

- Do not treat `sources/apis/delivery/openapi.json` as a spec the service publishes. It is a
  tier-4 reconstruction written by reading route registrations
  (sources/apis/delivery/openapi.meta.json).
- Do not cite `tests/deliveryStatus.test.ts` as evidence that the retry behaviour is verified. Its
  body is `expect(true).toBe(true)` and its runner is not installed.
- Do not report the owner team from `package.json` ("커머스본부 물류팀") as settled. The service
  registry records `owner_team: TODO` with the note "물류팀 이관 논의 중 (2025-11~), 확정 전"
  ("transfer to the logistics team under discussion since 2025-11; not confirmed")
  (sources/context/registry/services.yaml).

## Related

- [[API-ORDER]] — the live cancel endpoint and status enum the generated client drifted from
- [[CON-ORDER-DELIVERY]] — the cross-system boundary this API sits on
- [[PROC-DELIVERY-AUTH]] — why none of these calls carry credentials
- [[PROC-DELIVERY-STATUS-SYNC]] — the retry behaviour behind the two responses
- [[RISK-DELIVERY]] — the stale client and the silent fallback as defects
- [[SCH-DELIVERY]] — the shapes of the bodies above
- [[SYS-DELIVERY]] — the service overview
