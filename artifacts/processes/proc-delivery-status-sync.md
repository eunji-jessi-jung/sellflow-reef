---
id: "PROC-DELIVERY-STATUS-SYNC"
type: "process"
title: "Delivery Status Sync and Retry"
domain: "delivery"
status: "draft"
last_verified: 2026-09-18
freshness_note: "Traced line by line through src/deliveryStatus.ts and src/index.ts on 2026-09-18, cross-read against README.md and .env.template. Stale if MAX_RETRY, the axios timeout, the backoff, or the env var names change."
freshness_triggers:
  - ".env.template"
  - "README.md"
  - "src/deliveryStatus.ts"
  - "src/index.ts"
  - "src/logger.ts"
known_unknowns:
  - "Which exceptions the retry loop treats as retryable is unknown — the catch is bare, so a 404 for an unknown order, a 401, a DNS failure, and a timeout are all retried identically."
  - "Whether the process actually runs against a real carrier is unverifiable from the repo: CARRIER_API is unset in .env.template, so the placeholder default https://api.carrier.example applies unless the deployment injects the variable another way."
  - "No deployment manifest was found, so how environment variables reach the running process is unknown and the mismatch below cannot be confirmed as live behaviour."
  - "Whether the carrier API is idempotent under retry is undocumented; the call is a GET, which suggests it is safe, but no vendor contract confirms it."
  - "The absence of alerting is stated in a code comment; whether an external monitor watches the [WARN] log lines or the stale response rate is unknown — no dashboard, alert rule, or runbook is present in the repo."
  - "No circuit breaker, budget, or concurrency limit exists, so behaviour under a sustained carrier outage (every request paying the full ~10.5 s before answering) is untested."
tags:
  - "delivery"
  - "process"
  - "retry"
  - "resilience"
  - "configuration"
aliases:
  - "syncDeliveryStatus"
  - "배송 상태 동기화"
relates_to:
  - type: "refines"
    target: "[[API-DELIVERY]]"
  - type: "constrains"
    target: "[[RISK-DELIVERY]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "Defines ORDER_API_BASE, CARRIER_API_BASE, RETRY_COUNT — none of which the code reads."
  - category: "documentation"
    type: "doc"
    ref: "order-service:README.md"
    notes: "Claims retry-then-last-stored-value behaviour."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "The retry loop itself."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "Consumes null and substitutes the hardcoded fallback."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/logger.ts"
notes: ""
---

## Purpose

`syncDeliveryStatus(ordNo)` is the one piece of real logic in delivery-bff: it fetches a delivery status from the carrier and, because the carrier is unreliable, retries. The function's own comment states the reason — "배송사 API가 불안정하여 재시도를 넣어두었다" (the carrier API is unstable, so retries were added).

Reading it closely produces two findings that are easy to miss. The retry policy is entirely hardcoded even though `.env.template` looks like it configures it, and exhaustion is silent by design.

## Key Facts

- `MAX_RETRY` is a module-level constant set to 3 → src/deliveryStatus.ts
- Each attempt uses an axios GET with `{ timeout: 3000 }` — 3 seconds → src/deliveryStatus.ts
- Backoff is linear: `setTimeout(r, 500 * attempt)`, so 500 ms after attempt 1 and 1000 ms after attempt 2 → src/deliveryStatus.ts
- No sleep occurs after the final attempt; the loop returns null at `attempt === MAX_RETRY` before reaching the backoff → src/deliveryStatus.ts
- The catch block is bare — every thrown error is retried with no classification → src/deliveryStatus.ts
- Each failure logs a warning `배송 상태 조회 실패 (${attempt}/${MAX_RETRY}) ordNo=${ordNo}` (delivery status lookup failed) → src/deliveryStatus.ts
- The function comment states "3회 실패 시 포기한다. 별도 알림은 없다" (it gives up after 3 failures; there is no separate notification) → src/deliveryStatus.ts
- README.md promises "조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다" (on lookup failure, retry three times then return the last stored value), a value the service never stores → README.md
- The code reads `process.env.CARRIER_API` while .env.template defines `CARRIER_API_BASE`, so the template variable has no effect → src/deliveryStatus.ts
- `.env.template` defines `RETRY_COUNT=3` but nothing in the repo reads it; retries come from the hardcoded MAX_RETRY → .env.template
- `.env.template` defines `ORDER_API_BASE=http://order-service.internal` but nothing reads it either; the generated client fetches a relative path → src/generated/orderApi.ts
- The unset-CARRIER_API default is the placeholder `https://api.carrier.example` → src/deliveryStatus.ts

## Steps

1. **Enter the loop.** `for (let attempt = 1; attempt <= MAX_RETRY; attempt++)`, with MAX_RETRY fixed at 3.
2. **Call the carrier.** `axios.get(`${CARRIER_API}/tracking/${ordNo}`, { timeout: 3000 })`. `CARRIER_API` is resolved once at module load from `process.env.CARRIER_API ?? 'https://api.carrier.example'`, so it is fixed for the life of the process.
3. **On success, return immediately.** `return res.data as DeliveryStatus` — the raw body, cast without validation. No further attempts.
4. **On any throw, log a warning** at level WARN through `src/logger.ts`, which is a `console.log`/`console.warn`/`console.error` wrapper. The message carries the attempt number and the ordNo but not the error itself, so the cause of the failure is not recorded.
5. **If this was attempt 3, return null** and stop. No alert, no metric, no event.
6. **Otherwise sleep `500 * attempt` ms** — 500 ms then 1000 ms — and loop.
7. **The caller substitutes a literal.** `src/index.ts` checks `if (!status)` and responds with `{ ordNo, status: 'PREPARING', stale: true }`. Null becomes a 200 with a fabricated state; see [[API-DELIVERY]].

Worst-case latency before a caller gets any answer: three 3-second timeouts plus 1.5 seconds of backoff, about 10.5 seconds. Nothing caps this, and the route has no timeout of its own.

### The configuration that does nothing

`.env.template` reads in full:

```
ORDER_API_BASE=http://order-service.internal
CARRIER_API_BASE=https://api.carrier.example.co.kr
RETRY_COUNT=3
```

All three lines are inert, and the failure mode differs per line:

| Template variable | What the code actually does | Effect of setting it |
|---|---|---|
| `CARRIER_API_BASE` | `src/deliveryStatus.ts` reads `process.env.CARRIER_API` | None. The real carrier host `https://api.carrier.example.co.kr` in the template is never used; the placeholder default `https://api.carrier.example` applies instead. |
| `RETRY_COUNT` | `MAX_RETRY = 3` is a hardcoded constant | None. Retry count cannot be tuned without a code change. |
| `ORDER_API_BASE` | `src/generated/orderApi.ts` calls `fetch('/api/v1/orders/...')` with no origin | None. The generated client has no base-URL hook at all. |

The `CARRIER_API_BASE` / `CARRIER_API` name mismatch is the sharpest of the three: an operator following the template would set a correct production hostname and see the service keep calling a placeholder domain, with no startup warning, because the `??` default silently absorbs the missing variable. The `ORDER_API_BASE` case is worse in kind — a relative `fetch` path has no host to resolve against outside a browser, so the cancel client cannot reach order-service however the environment is configured.

### Silence on exhaustion

Three things combine here. The code gives up after three attempts; the comment says "별도 알림은 없다" (there is no separate notification); and the failure surfaces to the caller as HTTP 200. A carrier outage is therefore invisible in the three places an operator would normally look — status codes, alerts, and error rates. The only evidence is WARN lines on stdout from a console-based logger, and `.eslintrc.json` sets `no-console` to `warn`, which suggests console logging is regarded as a lint smell in this repo rather than the intended logging strategy. This is carried into [[RISK-DELIVERY]].

## Worked Examples

**Carrier healthy.** Request for ordNo 20260918000123. Attempt 1 returns 200 in 120 ms. `syncDeliveryStatus` returns the parsed body; the route serializes it. No log line is written — the success path is entirely unlogged.

**Carrier flaky, recovers on the second attempt.** Attempt 1 times out at 3000 ms and logs `[WARN] 배송 상태 조회 실패 (1/3) ordNo=20260918000123`. The loop sleeps 500 ms. Attempt 2 succeeds. Total latency about 3.6 s; the client sees a normal 200 with no indication that anything went wrong, since `stale` is only set on the give-up path.

**Carrier down.** Attempts 1, 2, and 3 each time out at 3000 ms, logging `(1/3)`, `(2/3)`, `(3/3)`. Sleeps of 500 ms and 1000 ms sit between them; no sleep follows attempt 3. After roughly 10.5 seconds the function returns null and `src/index.ts` answers:

```json
{ "ordNo": "20260918000123", "status": "PREPARING", "stale": true }
```

If the order was in fact 배송완료 (delivered), the customer is now told it is being prepared. Nothing anywhere records the discrepancy.

**Unknown order number.** Suppose the carrier answers 404 for an ordNo it does not know. The bare catch treats this exactly like a timeout: three attempts, three warnings, ~1 s of pointless backoff, then the same PREPARING body. A permanently wrong input and a transient outage are indistinguishable both to the retry loop and to the caller.

## Related

- [[API-DELIVERY]] — the responses this process produces
- [[RISK-DELIVERY]] — dead configuration and silent failure as risks
- [[SYS-DELIVERY]] — the service overview
