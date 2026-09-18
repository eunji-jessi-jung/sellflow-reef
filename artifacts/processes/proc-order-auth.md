---
id: "PROC-ORDER-AUTH"
type: "process"
title: "Order Service Authentication and Access Control"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 to answer Q-009 definitively: every file in order-service was re-read (build.gradle, all 5 application*.yml, .env.example, all 29 main classes, all 5 test classes, all 5 workflows) and the 2022 OpenAPI spec was parsed path by path for its security and response declarations. The finding is an absence, so it is only as durable as the next dependency addition — re-check build.gradle and WebConfig first. The gateway question is unresolvable from inside these repositories and has been logged for the owner."
freshness_triggers:
  - ".env.example"
  - "build.gradle"
  - "src/main/java/kr/co/sellflow/order/config/WebConfig.java"
  - "src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
  - "src/main/java/kr/co/sellflow/order/search/OrderSearchController.java"
  - "src/main/resources/application-prod.yml"
  - "src/main/resources/application.yml"
known_unknowns:
  - "Whether a gateway exists, what product it is, and who operates it. The only evidence remains one comment in WebConfig; no config, no registry entry, no workflow and no document in the reef names a gateway product, host or policy. Checked: every file in order-service (find . -type f), the service registry, and all five repos for ingress/proxy config."
  - "Whether that gateway validates the bearerAuth JWT the 2022 spec declares, with which issuer, audience, signing key or clock skew. Unverified and unverifiable from these repositories."
  - "Whether POST /orders/{ordNo}/cancel is reachable from outside the cluster. It is not under /api/**, so it carries no CORS mapping, but CORS is not an access control and says nothing about network reachability."
  - "Which environment variable actually supplies the production datasource password. application-prod.yml sets no username or password at all, so the value must arrive through Spring relaxed binding from the environment (SPRING_DATASOURCE_PASSWORD or similar) or a secret manager. Nothing in the repository documents which."
  - "Whether the /internal/health, /internal/ready and /internal/metrics paths the spec declares are served by the gateway or by a sidecar. They are served by nothing in this application."
  - "Whether any deployment-time network policy restricts who can reach port 8081. No manifest, chart or compose file exists in the repo, and deploy.sh — invoked by all four CD workflows — is absent from the tree."
tags:
  - "order"
  - "auth"
  - "cors"
  - "gateway"
  - "gap"
  - "q-009"
aliases:
  - "order auth"
  - "주문 인증"
  - "Q-009"
relates_to:
  - type: "constrains"
    target: "[[API-ORDER]]"
  - type: "refines"
    target: "[[PROC-ORDER-ERROR-HANDLING]]"
  - type: "constrains"
    target: "[[RISK-ORDER]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:.env.example"
    notes: "DB_PASSWORD present but empty; no token, key or secret variable of any kind."
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-prod-cd.yml"
    notes: "No secret injection step; deploy.sh is not in the tree."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.meta.json"
    notes: "Independently records the same unresolved auth question. Reef-root relative."
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "Five dependencies, none security-related and none actuator."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Public endpoint hostname, different from the app's own port."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "Declares global bearerAuth with bearerFormat JWT, 401 on 29 of 32 operations, 403 on all 8 /admin/* paths, and three /internal/* paths. Reef-root relative."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Calls the cancel endpoint with only a Content-Type header — no Authorization."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/config/JpaConfig.java"
    notes: "The only other @Configuration class; auditing and transactions only."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java"
    notes: "The only access-control code in the service."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java"
    notes: "No actor, channel or user column is mapped."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java"
    notes: "Two handlers, 404 and 409; no 401/403 path exists."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchController.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application-prod.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V17__add_cancel_channel.sql"
    notes: "Adds CHNL_CD for cancel channel; the entity never maps it."
notes: "Archetype: workflow. This artifact answers Q-009. Most of it is necessarily a negative finding — the enforcement point, if one exists, is outside the repository and undocumented in the reef's sources."
---

# Order Service Authentication and Access Control

## Purpose

To answer Q-009 — how authentication and authorisation are enforced in order-service, and what the spec's `bearerAuth` declaration corresponds to in running code — as precisely as the sources allow.

**The answer, stated definitively: nothing in order-service enforces `bearerAuth`.** No credential is validated, no claim is extracted, no permission is checked, and no request attribute carrying an identity is read anywhere in the service. The spec's global `security: [{bearerAuth: []}]` corresponds to zero lines of running code in this repository. The only access-control construct that exists at all is a single CORS mapping, which is a browser convention rather than an access control, and which does not cover the one write endpoint the service exposes.

This is a negative finding reached by exhaustion, not by inference. The method is recorded under "How this was checked" below so that a future reader can repeat it.

## Key Facts

- No security dependency is declared: `build.gradle` lists exactly five — spring-boot-starter-web, spring-boot-starter-data-jpa, flyway-core, mysql-connector-java (runtimeOnly) and spring-boot-starter-test → build.gradle
- Spring Boot 2.3.12 with no `spring-boot-starter-security` means Spring Security's auto-configuration never enters the classpath, so there is no default filter chain, no default `user` account, and no HTTP Basic prompt → build.gradle
- No servlet `Filter`, `HandlerInterceptor`, `HandlerMethodArgumentResolver`, `@PreAuthorize`, `@Secured`, `@RolesAllowed` or `WebSecurityConfigurerAdapter` exists anywhere under `src/main/java`; a grep across all 29 classes returns nothing → src/main/java/kr/co/sellflow/order/
- The service has exactly two `@Configuration` classes. `JpaConfig` enables auditing and transaction management. `WebConfig` overrides `addCorsMappings` and nothing else — notably not `addInterceptors` → src/main/java/kr/co/sellflow/order/config/JpaConfig.java
- The whole of the service's access-control surface is one line: `registry.addMapping("/api/**").allowedOrigins("https://admin.sellflow.co.kr")` → src/main/java/kr/co/sellflow/order/config/WebConfig.java
- Because no `allowedMethods` is given, Spring's `CorsRegistration` defaults apply — GET, HEAD and POST only — and because `allowCredentials` is never set, the browser may not send cookies or Authorization headers on those cross-origin calls at all → src/main/java/kr/co/sellflow/order/config/WebConfig.java
- The comment above that mapping is the only statement about authentication in the entire codebase: "어드민 도메인만 허용. 앱은 게이트웨이를 통해 들어온다." ("only the admin domain is allowed; the app comes in through the gateway") → src/main/java/kr/co/sellflow/order/config/WebConfig.java
- `POST /orders/{ordNo}/cancel` — the service's only write endpoint — is mapped under `/orders`, not `/api/**`, so the CORS mapping does not apply to it at all → src/main/java/kr/co/sellflow/order/controller/OrderController.java
- Neither controller method accepts a `@RequestHeader`, `HttpServletRequest`, `Principal` or `Authentication` parameter. The cancel handler's entire input is a path variable and a two-field body (`sayuCd`, `bigo`) → src/main/java/kr/co/sellflow/order/controller/OrderController.java
- The 2022 spec declares `X-Request-Id` and `X-Client-Ver` header parameters on the cancel operation; the controller reads neither, so even the non-security headers the spec promises are discarded → sellflow-docs:raw/specs/order-service-openapi.json
- The spec declares a global security requirement `[{bearerAuth: []}]` with `type: http`, `scheme: bearer`, `bearerFormat: JWT`, and no per-operation override anywhere — all 32 operations inherit it → sellflow-docs:raw/specs/order-service-openapi.json
- The spec declares `401` on 29 of its 32 operations and `403` on all eight `/admin/*` paths; `GlobalExceptionHandler` maps exactly two exceptions, to 404 and 409, so no code path in the service can emit either status → src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java
- No JWT or JOSE library is present anywhere — no jjwt, no nimbus-jose-jwt, no oauth2-resource-server, no java-jwt — so the service could not parse a bearer token even if it received one → build.gradle
- No claim is extracted and no permission is evaluated: the only authorisation-shaped decision in the service is `CHWISO_BULGA.contains(order.getSangtaeCd())`, which is a state check on the order, not a check on the caller → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The actual caller of the cancel endpoint sends no credential: delivery-bff's generated client calls `fetch('/api/v1/orders/${ordNo}/cancel', { method: 'POST', headers: { 'Content-Type': 'application/json' } })` — the header map has one entry and it is not `Authorization` → delivery-bff:src/generated/orderApi.ts
- `spring-boot-starter-actuator` is not a dependency and no `management.*` key appears in the base `application.yml` or in any of the four environment profiles (dev, qa, stage, prod; `application-local.yml.example` is a template, not an active profile), so **no** actuator endpoint is exposed — neither secured nor unsecured. The spec's `GET /internal/health`, `/internal/ready` and `/internal/metrics` (the only three operations with no `401` declared) are served by nothing in this application → build.gradle
- No TLS, no `server.servlet.context-path` and no `server.ssl.*` key appears in any profile; the app binds plain HTTP on port 8081 → src/main/resources/application.yml
- The caller's identity is never recorded. `V17__add_cancel_channel.sql` added `ORDER_CANCEL.CHNL_CD VARCHAR(10) NULL COMMENT '취소 접수 채널 (APP/ADMIN/CS)'` ("cancellation intake channel"), but the `OrderCancel` entity maps only ORD_NO, CHWISO_ILSI, CHWISO_SAYU_CD, CHORI_SANGTAE and BIGO — so every row written since is left NULL → src/main/java/kr/co/sellflow/order/domain/OrderCancel.java
- The registry lists the public endpoint as `https://api.sellflow.co.kr/orders`, a different host and scheme from the app's own plain-HTTP 8081 — consistent with a fronting layer, but it names none → sellflow-docs:context/registry/services.yaml
- No secret of any kind is configured in the repo: `.env.example` declares `DB_PASSWORD=` (empty), `PG_BASE_URL` and `INVENTORY_BASE_URL`, and no token, key or client-secret variable; `application-prod.yml` sets neither username nor password, so production credentials must arrive via environment binding or a secret manager the repo does not describe → .env.example

## Input/Output

**Input to the auth step:** nothing. There is no auth step. A request arrives as an HTTP method, a path, optional headers that are never read, and a JSON body.

**Output:** no `401`, no `403`, no `WWW-Authenticate` header, and no identity attached to the request or to any row the request writes.

## Steps

What can actually be traced for an inbound request:

1. **The request arrives on port 8081**, plain HTTP. Spring Boot's default filter chain is whatever `spring-boot-starter-web` contributes — character encoding, form content, request context. No security filter is in it, because no security starter is on the classpath.
2. **CORS is evaluated — for browser requests to `/api/**` only.** `OrderSearchController` (`/api/v1/orders/search`) is the only mapped controller inside that pattern. A browser on any origin other than `https://admin.sellflow.co.kr` is refused the response. A server-side call, a `curl`, or any non-browser client ignores CORS entirely.
3. **No interceptor runs.** `WebConfig` does not override `addInterceptors`, and no `@Component` implements `HandlerInterceptor`.
4. **The handler runs, with no identity.** `OrderController.cancel` cancels whichever order number it is handed. `OrderSearchController.search` returns whatever the date range matches. Neither consults a caller.
5. **Exceptions map to 404 or 409 only.** `GlobalExceptionHandler` handles `OrderNotFoundException` and `OrderCancelNotAllowedException`. Everything else falls through to Spring's default error handling as a 500. See [[PROC-ORDER-ERROR-HANDLING]].
6. **Presumed, not verified: a gateway sits in front.** The WebConfig comment asserts the app's traffic "comes in through the gateway"; the registry's public hostname differs from the app's own port; delivery-bff calls a `/api/v1` prefix the controller does not serve. If a JWT is validated anywhere, that is where it happens. Nothing in the reef's current sources names the gateway, its policy, or its token issuer.

## Error Handling

There is no error handling for authentication, because there is no authentication. Specifically:

- **No 401 is producible.** A client generated from the 2022 spec expects `401` with an `ErrorResponse` body when its token is missing or expired, and will use it to trigger a refresh. It will never see one from this service.
- **No 403 is producible.** The eight `/admin/*` paths that declare `403` are not implemented in this service at all; the only admin-adjacent endpoint that exists is `/api/v1/orders/search`, which the spec does not describe.
- **The error body shape also diverges.** The spec's `ErrorResponse` is `{code, message, traceId}`; `GlobalExceptionHandler` returns `Map.of("message", e.getMessage())` — one field, no code, no trace id. A client parsing `code` or `traceId` gets null even on the two statuses the service does implement.
- **Failures leak Korean operator text to whoever asked.** `OrderNotFoundException` renders "주문을 찾을 수 없습니다. ordNo=" + ordNo ("the order could not be found"), and `OrderCancelNotAllowedException` renders "취소할 수 없는 주문 상태입니다. ordNo=..., status=..." ("the order is in a state that cannot be cancelled"). Since no caller is authenticated, these messages confirm the existence and state of any order number to any requester who can reach the port — an order-number enumeration oracle.

## Worked Examples

**The one enforcement that is real.** A CS operator's browser at `https://admin.sellflow.co.kr` calls `GET /api/v1/orders/search?from=20260801&to=20260831`. The origin matches the single allowed value, so the browser permits the response. The same call from `https://evil.example.com` is blocked — by the browser, and by nothing else. `curl` from anywhere with network reach gets the data either way. Note also that because `allowCredentials` is never set, the admin UI cannot attach a cookie or `Authorization` header to this cross-origin call even if it had one.

**The one that is not.**

```
curl -X POST http://order-service.internal:8081/orders/ORD20230411002/cancel \
     -H 'Content-Type: application/json' \
     -d '{"sayuCd":"03","bigo":"test"}'
```

No credential is carried and none is required. Within this repository there is nothing to reject it: no filter, no principal, no ownership check tying the order to `GOGAEK_ID`, and no record of who asked. The order is cancelled, `ORDER_MST.SANGTAE_CD` becomes `CHWISO`, an `ORDER_CANCEL` row is written with `CHNL_CD` left NULL, and an `order.cancelled` row is queued in `ORDER_EVENT_OUTBOX` for settlement to pick up. Whether such a request can reach port 8081 in production is exactly the question the sources cannot answer.

**What the spec promises versus what runs.**

| Spec declaration (2022-11-04, v2.4.0) | Counterpart in running code |
|---|---|
| `security: [{bearerAuth: []}]`, global | none — no security dependency, no filter, no token library |
| `bearerFormat: JWT` | no JWT parser on the classpath |
| `401` on 29 operations | unreachable; handler maps only 404 and 409 |
| `403` on all 8 `/admin/*` paths | those paths do not exist in the service |
| `ErrorResponse {code, message, traceId}` | `{"message": "..."}` only |
| `X-Request-Id`, `X-Client-Ver` headers on cancel | read by nothing |
| `GET /internal/health`, `/internal/ready`, `/internal/metrics` | no actuator starter, no `management.*` config — not served |
| 32 operations across `/orders`, `/partners`, `/admin`, `/internal` | 2 endpoints exist: `POST /orders/{ordNo}/cancel`, `GET /api/v1/orders/search` |

## How this was checked

So that the negative can be re-verified rather than re-argued:

1. `find . -type f` over the whole repo (82 files, excluding `.git`) — every one was opened.
2. `build.gradle` read in full for any security, actuator, JWT or filter dependency. Five dependencies total.
3. All five `application*.yml` files read for `management.*`, `server.ssl.*`, `context-path`, and credentials.
4. Grep across `src/` for `Filter`, `Interceptor`, `Secured`, `PreAuthorize`, `Principal`, `Authentication`, `RequestHeader`, `actuator`, `management`, `/internal` — no production match.
5. The 2022 OpenAPI JSON parsed operation by operation for `security`, response codes, and parameters.
6. The one caller found across all five repos (delivery-bff's generated client) read for an `Authorization` header — absent.

## Related

- [[SYS-ORDER]] — the service this applies to
- [[API-ORDER]] — the spec whose bearerAuth declaration has no implementation here
- [[PROC-ORDER-ERROR-HANDLING]] — what the client sees instead of 401, and why 500 is the default outcome
- [[RISK-ORDER]] — the missing-credential finding in its wider risk context
