---
id: "PROC-ORDER-ERROR-HANDLING"
type: "process"
title: "Order Service Error Handling and Failure Behaviour"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Established on 2026-09-19 by reading every class under order-service/src/main/java, grepping the whole tree for try/catch, throw, log, retry, timeout and circuit, and tracing the cancel transaction boundary through OrderCancelService, OrderEventPublisher and the two unused HTTP clients. The partial-failure conclusions are counterfactual by necessity — the clients have no caller today — and must be re-derived the moment either client is wired in. Re-checked on 2026-09-19 after a correction pass in order-service: OrderStatusService.change now compiles and saves, and its silent `findById(...).ifPresent(...)` no-op on a missing order is unchanged, so every error-handling claim below stands."
freshness_triggers:
  - "src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - "src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - "src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
  - "src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java"
  - "src/main/java/kr/co/sellflow/order/payment/PaymentClient.java"
  - "src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "src/main/resources/application-prod.yml"
known_unknowns:
  - "What Spring's default /error response body actually looks like in production. BasicErrorController is auto-configured by spring-boot-starter-web and server.error.* is unset in the base application.yml and in all four environment profiles (dev, qa, stage, prod), so the defaults apply, but no captured 500 response from this service exists in any source to confirm the shape."
  - "Whether anything outside the repository alerts on failures. There is no actuator, no metrics exporter, no Sentry/Datadog dependency and no log-based alert rule in the repo; whether the platform tails stdout and alerts on ERROR lines is undocumented — and moot, since the service never logs at ERROR."
  - "Whether the relay or the settlement side ever fails to process an outbox row, and what happens then. The publisher's javadoc states plainly that this service does not track it."
  - "Whether the PG cancel call was ever wired in and later removed, or never wired in at all. Git history was not consulted; the class has carried a 2021 TODO since before the SF-2287 change."
  - "Whether an operator has any way to retry a failed cancel. No admin endpoint, no replay job and no dead-letter table exists in this service; the spec's POST /admin/events/outbox/{eventId}/replay is not implemented."
  - "Whether the 500s a failing cancel would produce are visible to anyone. No request id is generated, logged or returned, so a customer report cannot be matched to a log line."
tags:
  - "order"
  - "error-handling"
  - "exceptions"
  - "transactions"
  - "outbox"
  - "observability"
aliases:
  - "order error handling"
  - "주문 예외 처리"
relates_to:
  - type: "refines"
    target: "[[API-ORDER]]"
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[DEC-ORDER-OUTBOX-RELAY]]"
  - type: "refines"
    target: "[[PROC-ORDER-AUTH]]"
  - type: "refines"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "constrains"
    target: "[[RISK-ORDER]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "The restock endpoint the order-side client does not match."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "Declares ErrorResponse {code, message, traceId} and 500 on 29 operations."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "The only caller; branches on 409 and calls res.json() on everything else."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "RestTemplate with no timeout, no error handler, no caller."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
    notes: "Throws IllegalArgumentException, which nothing handles."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java"
    notes: "CHORI_SANGTAE is hardcoded to COMPLETED at construction."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
    notes: "@Transactional with default propagation; joins the cancel transaction."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
    notes: "The service's entire error-mapping surface: two handlers."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java"
    notes: "The only try/catch in the service; three empty catch blocks."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java"
    notes: "Posts a null body to the PG with neither ordNo nor amount in the request."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The one write path; @Transactional from javax.transaction."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderStatusService.java"
    notes: "ifPresent silently no-ops when the order is missing."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application-prod.yml"
    notes: "The only timeout anywhere in the service is Hikari's connection-timeout: 3000."
notes: "Archetype: operational. The headline is that error handling in this service is almost entirely Spring's defaults: two mapped exceptions, no try/catch outside the deprecated legacy class, no retry, no timeout on outbound HTTP, and no log statement above INFO."
---

# Order Service Error Handling and Failure Behaviour

## Purpose

To record what order-service does when something goes wrong: which exceptions are caught and translated, which propagate, what a client actually receives, what is logged, and whether a partial failure can leave `ORDER_MST`, `ORDER_CANCEL` and `ORDER_EVENT_OUTBOX` out of step with each other or with the outside world.

The summary is that the service has almost no error handling of its own. Two exception types are translated to HTTP statuses; everything else is Spring Boot's default. There is no retry anywhere, no timeout on any outbound HTTP call, no circuit breaker, no dead-letter path, and not one log statement above INFO in the entire main source tree. The one place that does contain hand-written error handling is the `@Deprecated` legacy class, and what it contains is three empty catch blocks.

## Key Facts

- `GlobalExceptionHandler` is the only `@RestControllerAdvice` in the service and declares exactly two handlers: `OrderNotFoundException` → 404 and `OrderCancelNotAllowedException` → 409 → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- Its own comment confirms the 409 is deliberate and treated as the contract: "409 로 내려간다. API 스펙 문서와 상태코드는 여기 기준." ("it goes down as 409; the API spec document and status codes follow this") → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- Both handlers return `Map.of("message", e.getMessage())` — a single-field body — while the spec's `ErrorResponse` schema declares `{code, message, traceId}`, so `code` and `traceId` are null even on the two statuses the service does implement → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- `CancelReason.of(code)` throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` ("unknown cancellation reason code") for anything outside 01–04, including null. No handler exists for it, so an invalid `sayuCd` produces a **500**, not the 400 the spec declares → src/main/java/kr/co/sellflow/order/domain/CancelReason.java
- There is no `@Valid`, no bean-validation annotation and no `spring-boot-starter-validation` dependency anywhere; a grep for `Valid`/`NotNull` across `src/main` matches only a javadoc mention of a non-existent `OrderStatusValidator` → src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- Outside the deprecated legacy class there is **no** `try`, `catch` or `finally` in the entire service — a grep of `src/` returns matches only in `OrderCancelServiceV1` → src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- That class swallows three exception types outright — `if (rs != null) try { rs.close(); } catch (Exception ignore) {}` and the same for `PreparedStatement` and `Connection` — and its `catch (Exception e) { if (conn != null) conn.rollback(); throw e; }` can itself throw from `rollback()`, replacing the original cause → src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- The whole main source tree contains three logging statements, all at INFO, all on success paths: the PG call's "pg cancel ordNo={} amount={}", the cancel service's "주문 취소 완료" ("order cancellation complete") and the status service's "status change". There is no `log.error`, `log.warn` or `log.debug` anywhere → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The success log is also wrong about state: it prints `prevStatus={}` from `order.getSangtaeCd()` **after** `order.chwiso()` has already set the status, so every line reports `CHWISO` as the previous status → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- No retry, backoff, circuit breaker or resilience library appears anywhere; a case-insensitive grep of `src/` for `retry`, `timeout` and `circuit` matches a single line — Hikari's `connection-timeout: 3000` in the production profile → src/main/resources/application-prod.yml
- Both HTTP clients build `new RestTemplate()` as a field initialiser, so they use `SimpleClientHttpRequestFactory` with **no connect or read timeout**: a hung `inventory-api` or PG would hold the calling thread, and its database connection, indefinitely → src/main/java/kr/co/sellflow/order/client/InventoryClient.java
- Neither client sets a `ResponseErrorHandler`, so RestTemplate's default applies: any 4xx/5xx becomes an unchecked `HttpStatusCodeException`, and any connection failure an unchecked `ResourceAccessException` → src/main/java/kr/co/sellflow/order/payment/PaymentClient.java
- `PaymentClient.cancel(ordNo, amount)` logs both arguments and then sends neither: `restTemplate.postForEntity(baseUrl + "/v2/payments/cancel", null, Void.class)` posts a null body to a fixed URL with no order number and no amount, and discards the response → src/main/java/kr/co/sellflow/order/payment/PaymentClient.java
- `InventoryClient.restore` posts to `{baseUrl}/inventory/restore?ordNo=..&reason=..` with a null body, while `inventory-api` exposes `POST /stock/restock` taking a JSON body `{ord_no, reason_code}` — the path, the parameter style and the field names all differ, so the call would 404 if it were ever made → inventory-api:app/main.py
- Neither client is called by anything. `grep -rn "InventoryClient\|PaymentClient" src/` matches only the class declarations and `PaymentClient`'s own logger field, so **no external call happens during a cancel today** and there is no mid-cancel partial-failure window involving them → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- `OrderEventPublisher.publishOrderCancelled` is annotated `@Transactional` (javax) with default propagation, so it joins the caller's transaction rather than starting its own: the outbox row and the `ORDER_MST`/`ORDER_CANCEL` writes commit or roll back together → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- `OrderCancel` is constructed with `this.choriSangtae = "COMPLETED"` unconditionally — the row records the cancellation as fully processed at the moment it is created, before any downstream refund, restock or settlement correction has been attempted → src/main/java/kr/co/sellflow/order/domain/OrderCancel.java
- `OrderStatusService.change` wraps its work in `findById(...).ifPresent(...)`, so a status change for a non-existent order succeeds silently — no exception, no log, no return value → src/main/java/kr/co/sellflow/order/service/OrderStatusService.java
- `OrderSearchService` has no error handling at all; a `DataAccessException` from `jdbcTemplate.query` (the SQL filters on `ORD_DT`, a column no migration creates) propagates to Spring's default handler as a 500 → src/main/java/kr/co/sellflow/order/search/OrderSearchService.java
- The only caller in the estate branches on exactly one failure status: delivery-bff throws `Error('정산이 완료된 주문은 취소할 수 없습니다.')` ("an order whose settlement is complete cannot be cancelled") on 409 and calls `res.json()` on everything else, so a 404 or 500 is parsed as if it were a `CancelResponse` → delivery-bff:src/generated/orderApi.ts

## Scope

**In scope.** Everything reachable from the two live HTTP endpoints — `POST /orders/{ordNo}/cancel` and `GET /api/v1/orders/search` — plus the outbox write inside the cancel transaction, the two unused HTTP clients, and the deprecated legacy cancel path that remains compilable in the tree.

**Out of scope, and deliberately so.** What happens after an outbox row is written is another system's problem by explicit design: the publisher's javadoc states "구독 측 처리 결과는 본 서비스에서 추적하지 않는다." ("this service does not track the subscriber's processing result"). The relay job, the `CANCEL_RECON_QUEUE` and the never-scheduled reconciler are covered by [[DEC-ORDER-OUTBOX-RELAY]] and [[CON-ORDER-SETTLEMENT]].

**Not covered because it does not exist.** Authentication failures ([[PROC-ORDER-AUTH]]: no 401 or 403 is producible), rate limiting, request tracing, dead-letter storage, and operator-facing replay.

## Current State

### What is caught and translated

| Thrown | Where from | Caught by | Client sees |
|---|---|---|---|
| `OrderNotFoundException` | `OrderCancelService.cancel`, `OrderQueryService.get` | `GlobalExceptionHandler.notFound` | 404 + `{"message":"주문을 찾을 수 없습니다. ordNo=..."}` |
| `OrderCancelNotAllowedException` | `OrderCancelService.cancel` when status is CHWISO or BANPUM | `GlobalExceptionHandler.cancelNotAllowed` | 409 + `{"message":"취소할 수 없는 주문 상태입니다. ordNo=..., status=..."}` |

Both messages are Korean operator text rendered straight into the HTTP body, including the order number. Because no caller is authenticated ([[PROC-ORDER-AUTH]]), these responses double as an existence-and-state oracle for arbitrary order numbers.

### What propagates uncaught

| Thrown | Trigger | Client sees |
|---|---|---|
| `IllegalArgumentException` | `sayuCd` outside 01–04, or null/absent in the body | 500 via Spring's default `/error` — the spec says 400 |
| `HttpMessageNotReadableException` | malformed or missing JSON body | Spring's default 400, body unmapped |
| `DataAccessException` | any DB failure, including the search query's non-existent `ORD_DT` column | 500 |
| `HttpStatusCodeException` / `ResourceAccessException` | a failing or hung downstream HTTP call — *if either client were wired in* | 500, after an unbounded wait |
| anything else | — | 500 |

`server.error.*` is unset in the base `application.yml` and in all four environment profiles — dev, qa, stage, prod (a fifth file, `application-local.yml.example`, is a template, not an active profile) — so the 500 body is Spring's `BasicErrorController` default (`timestamp`, `status`, `error`, `path`), which matches neither the handler's `{message}` shape nor the spec's `ErrorResponse`. A client therefore encounters three distinct error body shapes from one service.

### Retry, timeout, alerting

There is none of the first, one of the second, and none of the third.

- **Retry:** nothing retries anything. There is no `@Retryable`, no loop, no scheduled sweep, no dead-letter table. A cancel that fails fails once and is gone; the customer's only recourse is to press the button again, and `OrderCancel`'s primary key is `ORD_NO`, so a second attempt on an order that already has a cancel row would be a JPA merge rather than an insert.
- **Timeout:** the sole timeout in the repository is `connection-timeout: 3000` on the Hikari pool in production (pool size 40). Outbound HTTP has none.
- **Alerting:** no actuator, no metrics, no health endpoint, no error-tracking dependency, and — decisively — no log statement above INFO. A total failure of the cancel path would produce no ERROR line for anyone to alert on. It would be visible only as the *absence* of "주문 취소 완료" lines.

### The mid-cancel failure the code cannot currently have

The brief's question — what happens if `InventoryClient` or the PG call fails mid-cancel — has a firm negative answer today: **it cannot happen, because neither is called.** `OrderCancelService.cancel` performs exactly five steps that touch state — load the order, check the blocked-status set, persist an `OrderCancel`, flip `ORDER_MST` to `CHWISO`, and write the outbox row. ([[PROC-ORDER-CANCEL]] numbers the same method as seven, counting the reason-code resolution and the closing log line as steps of their own.) There is no HTTP call in the path. That was verified by grepping `src/` for both class names; the only matches are the declarations themselves. The restock the business rules and the Confluence page promise, and the refund the PG client implies, are not attempted at all — see [[CON-ORDER-INVENTORY]] and [[PROC-ORDER-CANCEL]].

Stating the counterfactual precisely, because it is what a future change would inherit:

1. `@Transactional` on `cancel` comes from `javax.transaction.Transactional`, whose default `rollbackOn` covers `RuntimeException` and `Error`. RestTemplate's failures are all unchecked.
2. So an exception from a client call placed *inside* `cancel` would roll back the `ORDER_CANCEL` insert, the `ORDER_MST` update **and** the outbox row together.
3. The order and the outbox therefore stay consistent with each other in every failure ordering — this is the one thing the current design gets right, and it is a consequence of the publisher joining the caller's transaction rather than opening its own.
4. The inconsistency would be with the **outside world**: a PG refund that succeeded before the exception is not reversed, and an inventory restock that succeeded is not undone. Rollback covers the database and nothing else, and there is no compensating action anywhere in the service.
5. Ordering would decide the failure mode. Calling the PG before the commit risks a refunded but still-active order; calling it after the commit — outside the transaction — risks a cancelled order that was never refunded. Neither is handled, because neither call exists.

### The legacy path, which does have error handling

`OrderCancelServiceV1` is the only place with hand-written failure logic, and it is instructive about the house style of the era: raw JDBC, `setAutoCommit(false)`, an explicit `conn.rollback()` in a `catch (Exception e)` that rethrows, and a `finally` block that closes the `ResultSet`, `PreparedStatement` and `Connection` inside three separate `catch (Exception ignore) {}` blocks. A failure to close — a broken socket, a pool that has already reclaimed the connection — is discarded without a log line. The `rollback()` call is itself unguarded, so a rollback failure replaces the original exception with a `SQLException` about the rollback. The class is `@Deprecated`, has no Spring stereotype annotation (so it cannot be injected as written, despite its `@Autowired` field), and is retained on the stated grounds that "배치에서 참조 가능성이 있어 남겨둠" ("kept because a batch might reference it").

### What the client actually experiences

delivery-bff's generated client handles one status:

```ts
if (res.status === 409) {
  throw new Error('정산이 완료된 주문은 취소할 수 없습니다.');
}
return res.json();
```

Three consequences follow. A 409 is reported to the user as a settlement problem, when since SF-2287 it can only mean the order was already cancelled or returned. A 404 or a 500 falls through to `res.json()`, which parses an error body and returns it typed as `CancelResponse` — with `"strict": false` in `tsconfig.json`, TypeScript does not object. And a hung request has no client-side timeout either, so an unbounded server-side wait becomes an unbounded client-side one.

## Related

- [[SYS-ORDER]] — the service whose failure behaviour this describes
- [[PROC-ORDER-CANCEL]] — the flow these failure modes sit on
- [[PROC-ORDER-AUTH]] — why 401 and 403 never appear among these statuses
- [[API-ORDER]] — the spec's `ErrorResponse` shape and the statuses it promises
- [[DEC-ORDER-OUTBOX-RELAY]] — why the outbox write is transactional and where tracking stops
- [[CON-ORDER-INVENTORY]] — the restock call that is never made, and would 404 if it were
- [[CON-ORDER-SETTLEMENT]] — where responsibility passes out of this service, and stops
- [[RISK-ORDER]] — the same findings in risk form, with severity
