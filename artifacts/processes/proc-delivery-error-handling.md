---
id: "PROC-DELIVERY-ERROR-HANDLING"
type: "process"
title: "Delivery BFF Error Handling and Degraded Responses"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Traced on 2026-09-19 through all 112 lines of delivery-bff/src and tests, against the live order-service controller, exception handler and OrderStatus enum. The central claim — that the 마지막 저장 값 (last stored value) the README and the test name promise does not exist — was verified by confirming there is no database driver, cache client, file write or module-level variable anywhere in the repository, and that services.yaml records db: none for this service. Stale if any persistence, alerting or error-branch change lands."
freshness_triggers:
  - "README.md"
  - "package.json"
  - "src/deliveryStatus.ts"
  - "src/generated/orderApi.ts"
  - "src/index.ts"
  - "src/logger.ts"
  - "src/orderClient.ts"
  - "tests/deliveryStatus.test.ts"
known_unknowns:
  - "Which exceptions the retry loop should treat as retryable is undecided in the code. The catch is bare, so a 404 for an unknown order, a 500, a DNS failure and a timeout are all retried identically; no policy document states which of these a retry is appropriate for."
  - "Whether any external monitor watches the [WARN] log lines or the rate of stale:true responses. No dashboard, alert rule, SLO or runbook naming delivery-bff exists in sellflow-reef/sources, and the code comment says 별도 알림은 없다 (there is no separate alert)."
  - "Whether the carrier API is idempotent and what its error taxonomy is. The call is a GET, which suggests retry is safe, but no vendor contract or integration document exists to confirm it or to map its status codes."
  - "Whether an app client actually branches on the stale flag. No consumer of GET /delivery/:ordNo exists in any repository in this workspace, so how the degraded response is rendered to a customer cannot be observed."
  - "What the delivery status vocabulary is meant to be. The success path passes the carrier's status string through untouched and the fallback substitutes the literal PREPARING; no enum, mapping table or document defines the value space."
  - "Whether the process has ever run against a real carrier. CARRIER_API is unset in .env.template — the template defines CARRIER_API_BASE, a different name — so the placeholder default https://api.carrier.example applies unless the deployment injects the variable some other way."
tags:
  - "delivery"
  - "error-handling"
  - "operations"
  - "reliability"
  - "degraded-mode"
aliases:
  - "delivery error handling"
  - "배송 BFF 오류 처리"
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
    target: "[[PROC-DELIVERY-STATUS-SYNC]]"
  - type: "integrates_with"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "constrains"
    target: "[[RISK-DELIVERY]]"
  - type: "integrates_with"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "Claims 3회 재시도 후 마지막 저장 값을 반환한다 — retry three times then return the last stored value."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "Dependencies are axios and express only — no database driver, no cache client, no persistence of any kind."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "The retry loop, the 3s timeout, the 500ms×attempt backoff, and the null return."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "The 409 branch and the unguarded res.json() on every other status."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "The hardcoded {status: 'PREPARING', stale: true} fallback returned with HTTP 200."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/logger.ts"
    notes: "Three console wrappers; the only observability the service has."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:tests/deliveryStatus.test.ts"
    notes: "Test named 재시도 후 마지막 저장 값을 반환한다, body expect(true).toBe(true)."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "Returns ResponseEntity.ok().build() — HTTP 200 with an empty body."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
    notes: "The seven live status values, none of which is PREPARING."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
    notes: "Maps OrderCancelNotAllowedException to 409 with a message about order state, not settlement."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "db: none for delivery-bff — the registry confirms there is no store to hold a last value."
notes: "Two documents and one test name describe a cache this service does not have. The fallback they describe as 'the last stored value' is a literal typed into src/index.ts."
---

# Delivery BFF Error Handling and Degraded Responses

## Purpose

To set out what delivery-bff does when a call fails, and what the caller can tell from the
result. The governing fact is that this service cannot return an error to its client. Every
failure of the carrier — timeout, DNS, 500, 404, connection refused — converges on one HTTP 200
response carrying a hardcoded status of `PREPARING`, and the README, the code comment and the
only test in the repository all describe that response as something it is not.

## Key Facts

- Every carrier failure resolves to the same response: `res.json({ ordNo: req.params.ordNo, status: 'PREPARING', stale: true })`, returned with HTTP 200 → delivery-bff:src/index.ts
- `PREPARING` is a hardcoded string literal in `src/index.ts`, not a value read from anywhere → delivery-bff:src/index.ts
- **There is no store.** `package.json`'s dependencies are `axios` and `express`; there is no database driver, cache client, `fs` write or module-level variable holding previous results anywhere in the 112 lines of `src/` and `tests/` → delivery-bff:package.json, delivery-bff:src/deliveryStatus.ts, delivery-bff:src/index.ts
- The registry agrees: delivery-bff's entry records `db: none` → sellflow-docs:context/registry/services.yaml
- The README nonetheless claims "조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다" — "on lookup failure, retry three times and then return the last stored value" → delivery-bff:README.md
- The code comment repeats the claim and contains its own refutation: "조회 실패 시 마지막 저장 값을 반환한다. 없으면 준비중으로 표기." — "on lookup failure return the last stored value; if there is none, mark as preparing". The "if there is none" branch is the only branch, because there is never one → delivery-bff:src/index.ts
- The repository's only test is named for the same non-existent behaviour, "재시도 후 마지막 저장 값을 반환한다" ("returns the last stored value after retrying"), and its body is `expect(true).toBe(true)` → delivery-bff:tests/deliveryStatus.test.ts
- `PREPARING` is not a value in this estate's status vocabulary: the live `OrderStatus` enum declares `GYEOLJE_WANRYO`, `SANGPUM_JUNBI`, `BAESONG_JUNG`, `BAESONG_WANRYO`, `JUNGSAN_WANRYO`, `CHWISO`, `BANPUM` — romanised Korean throughout, with 상품준비중 rendered `SANGPUM_JUNBI` → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- The degraded body does not match the service's own `DeliveryStatus` interface: it omits `carrierCd` and `updatedAt` and adds an undeclared `stale` field → delivery-bff:src/deliveryStatus.ts, delivery-bff:src/index.ts
- The success path never sets `stale: false`, so the absence of an undeclared field is the client's only signal that a response is real → delivery-bff:src/index.ts
- Retries are indiscriminate: the `catch (e)` is bare, so a 404, a 500, a DNS failure and a timeout are retried identically, three times each → delivery-bff:src/deliveryStatus.ts
- Worst-case latency before the fallback is about 10.5 seconds: three attempts at a 3000 ms axios timeout plus backoff of `500 * attempt` after the first two (500 ms, then 1000 ms). There is no overall deadline and no express timeout → delivery-bff:src/deliveryStatus.ts
- There is no circuit breaker, no concurrency cap and no retry budget, so a sustained carrier outage means every request pays the full ~10.5 s and holds a socket while doing so → delivery-bff:src/deliveryStatus.ts
- Failure logging is three `logger.warn` lines per failed request, each `배송 상태 조회 실패 (n/3) ordNo=...` ("delivery status lookup failed"); there is no `logger.error` on final exhaustion and no metric → delivery-bff:src/deliveryStatus.ts, delivery-bff:src/logger.ts
- The absence of alerting is stated in the source itself: "3회 실패 시 포기한다. 별도 알림은 없다." — "gives up after three failures; there is no separate alert" → delivery-bff:src/deliveryStatus.ts
- On the cancel path, the generated client branches only on HTTP 409 and calls `return res.json()` for every other status, so an order-service error body would be parsed as a `CancelResponse` → delivery-bff:src/generated/orderApi.ts
- Even a *successful* cancel would fail there: `OrderController.cancel` returns `ResponseEntity.ok().build()` — HTTP 200 with an empty body — and `res.json()` on an empty body throws → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java, delivery-bff:src/generated/orderApi.ts
- `requestCancel` has no `try`/`catch` of its own, so every one of these failures propagates to its caller — of which there are none in this repository → delivery-bff:src/orderClient.ts

## Scope

Covered: the single route `GET /delivery/:ordNo` in `src/index.ts`, the carrier call and retry
loop in `src/deliveryStatus.ts`, and the error branches of the cancel client in
`src/generated/orderApi.ts` and `src/orderClient.ts`.

Excluded: the retry mechanism's tuning and the dead `.env.template` variables, which belong to
[[PROC-DELIVERY-STATUS-SYNC]]; the staleness of the generated client as a piece of debt, which
belongs to [[RISK-DELIVERY]]; and who may call any of this, which belongs to
[[PROC-DELIVERY-AUTH]].

## Current State

### The carrier failure path, step by step

```
attempt 1 → axios.get(CARRIER_API/tracking/{ordNo}, timeout 3000)
          ↳ throws → logger.warn('배송 상태 조회 실패 (1/3) ordNo=…') → sleep 500ms
attempt 2 → …                                   → sleep 1000ms
attempt 3 → …                                   → return null            (~10.5s elapsed)
index.ts  → if (!status) return res.json({ ordNo, status: 'PREPARING', stale: true })   [HTTP 200]
```

Nothing in this path can produce a non-200 status. `syncDeliveryStatus` catches every exception
class, and the handler's only branch on `null` is the fallback. There is no `next(err)`, no
express error middleware, and no `process.on('unhandledRejection')` or `uncaughtException`
handler anywhere in the repository. The service is, in the narrow sense, unable to report
failure.

### The consequence: PREPARING for an order that is not preparing

The fallback is a constant, so it does not depend on the order at all. During a carrier outage,
every order queried returns 준비중 ("preparing"):

| What the order actually is | What the client is told during a carrier outage |
|---|---|
| Genuinely being prepared | `PREPARING`, `stale: true` — correct by coincidence |
| In transit (배송중) | `PREPARING` |
| **Delivered (배송완료)** | `PREPARING` |
| **Cancelled (취소)** | `PREPARING` |
| Returned (반품) | `PREPARING` |
| Settled (정산완료) | `PREPARING` |
| Not a real order number at all | `PREPARING` — a 404 from the carrier is retried like an outage, then masked |

The two bolded rows are the damaging ones, and they are damaging in opposite directions. A
customer whose parcel is already in their hands is told it has not shipped, which generates a CS
contact for a non-problem. A customer who cancelled is told their order is being prepared, which
suggests the cancellation did not take — and given that this estate's cancellation flow is
already the subject of a long-running reconciliation problem ([[PROC-ORDER-CANCEL]],
[[RISK-SETTLEMENT-RECON-BACKLOG]]), a customer who believes a cancellation was ignored is
precisely the enquiry class the business least wants to manufacture. The last row matters too:
because a carrier 404 is retried and then masked, an unknown order number is indistinguishable
from a healthy order awaiting dispatch.

A further wrinkle is vocabulary. `PREPARING` is an English token in an estate that names its
states in romanised Korean — the live enum's value for 상품준비중 is `SANGPUM_JUNBI`, and the
convention is documented in [[PAT-SELLFLOW-ROMANISED-NAMING]]. So the fallback status is not a
degraded reading of a known state; it is a value from no vocabulary at all. A consumer mapping
delivery status onto `OrderStatus` has no case for it.

### "마지막 저장 값" — the value that does not exist

Three artefacts describe this service as returning a cached last-known status:

- README.md: "조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다" — "on lookup failure, retry three times and then return the last stored value"
- `src/index.ts` comment: "조회 실패 시 마지막 저장 값을 반환한다. 없으면 준비중으로 표기." — "on lookup failure return the last stored value; if there is none, mark as preparing"
- `tests/deliveryStatus.test.ts`: the test is named "재시도 후 마지막 저장 값을 반환한다" — "returns the last stored value after retrying"

No such value is stored. The evidence is exhaustive rather than inferential: `package.json`
declares `axios` and `express` and nothing else, so there is no database driver and no cache
client; `services.yaml` records `db: none` for this service; `syncDeliveryStatus` holds no
module-level state and writes nothing before returning; and no `fs`, `Map`, `redis` or global
appears anywhere in `src/`. The responses this service returns are computed entirely from the
current request and one constant.

The code comment is the tell. Its second sentence — 없으면 준비중으로 표기, "if there is none,
mark as preparing" — reads as a fallback within a fallback, but it is the whole implementation,
because "there is none" is unconditionally true. The author wrote the degraded case and the case
it degrades from was never built.

The test name is the more consequential artefact, because it is the one that would have caught
this. It asserts `expect(true).toBe(true)` — so the repository contains a test that names the
missing behaviour precisely and verifies nothing. Combined with the absence of any CI
([[RISK-DELIVERY]]), nothing has ever had the opportunity to notice.

### Timeouts, retries and what the app client experiences

Per attempt the axios timeout is 3000 ms. Backoff is `500 * attempt`, applied only between
attempts, so the sequence is 3000 + 500 + 3000 + 1000 + 3000 ≈ **10.5 s** in the worst case before
a response is produced. Express applies no timeout of its own and Node's default server timeout
would not fire first, so the client waits the full period. There is no jitter, so simultaneous
requests retry in lockstep — a thundering-herd shape against a carrier that is already failing.

From the app client's side, then:

| | Healthy carrier | Carrier down |
|---|---|---|
| HTTP status | 200 | 200 |
| Latency | one carrier round trip | ~10.5 s |
| Body | `{ordNo, carrierCd, status, updatedAt}` | `{ordNo, status: 'PREPARING', stale: true}` |
| Distinguishable? | — | only by noticing the `stale` field, or the missing `carrierCd`/`updatedAt` |

The `stale: true` flag deserves credit: it is genuinely present, and a client that checks it can
tell. But it is not part of the `DeliveryStatus` interface the service exports, the success path
never emits `stale: false` to pair with it, and no consumer of this endpoint exists in any
repository here to confirm anyone reads it. The realistic assumption is that a UI renders
`status` and shows 준비중.

### The cancel path's error handling

`cancelOrder` has exactly one error branch:

```ts
if (res.status === 409) {
  throw new Error('정산이 완료된 주문은 취소할 수 없습니다.');
}
return res.json();
```

Three things are wrong with it at once. The 409 message encodes the pre-SF-2287 rule — "an order
whose settlement is complete cannot be cancelled" — which order-service stopped enforcing in
April 2023; today a 409 comes from `GlobalExceptionHandler.cancelNotAllowed` and means
"취소할 수 없는 주문 상태입니다" ("the order is in a state that cannot be cancelled"), i.e. already
cancelled or already returned. Every other status falls through to `res.json()`, so a 404 body
`{"message": "주문을 찾을 수 없습니다. ordNo=…"}` would be returned as a `CancelResponse` with
`ordNo` and `sangtaeCd` both `undefined` — a silent, typed-looking nothing, which `"strict": false`
in tsconfig.json guarantees TypeScript will not question. And success fails too: the live
controller returns `ResponseEntity.ok().build()`, an empty 200 body, on which `res.json()` throws
`Unexpected end of JSON input`.

`requestCancel` wraps all of this in no error handling at all — it returns the promise directly.
The whole path is currently unreachable (nothing in the repository calls `requestCancel`, and the
relative `fetch('/api/v1/...')` has no origin to resolve against in Node), so this is a trap laid
for whoever wires it up rather than an active fault. It is catalogued in [[RISK-DELIVERY]].

### Observability

`src/logger.ts` is three `console` wrappers. On a failed lookup the service emits three `[WARN]`
lines and then nothing — there is no `[ERROR]` on exhaustion, so the moment the service decides
to lie about an order's status is the one moment it does not log. Nothing counts stale responses,
nothing measures carrier latency, and the source states plainly that 별도 알림은 없다 ("there is
no separate alert"). A total carrier outage therefore presents as: HTTP 200 throughout, a slow
endpoint, warning lines in container output, and customers told their delivered parcels are being
prepared.

## Related

- [[API-DELIVERY]] — the two response shapes this endpoint can return
- [[CON-ORDER-DELIVERY]] — the cancel boundary whose error branches are described above
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — the naming convention `PREPARING` sits outside
- [[PROC-DELIVERY-AUTH]] — who can trigger these paths, and the unroutable cancel call
- [[PROC-DELIVERY-STATUS-SYNC]] — the retry mechanism and the placeholder carrier host
- [[PROC-ORDER-CANCEL]] — the cancellation flow a PREPARING response contradicts
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measured cost of customers doubting a cancellation took effect
- [[RISK-DELIVERY]] — the stale fallback and the empty test as tracked findings
- [[SYS-DELIVERY]] — the service overview
- [[SYS-ORDER]] — the callee whose real 409 means something else entirely
