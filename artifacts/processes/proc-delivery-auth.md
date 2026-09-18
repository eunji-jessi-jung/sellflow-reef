---
id: "PROC-DELIVERY-AUTH"
type: "process"
title: "Delivery BFF Authentication and Authorization"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-verified on 2026-09-19 by grepping the whole delivery-bff tree (node_modules and .git excluded) case-insensitively for auth|token|jwt|oauth|api[_-]?key|apikey|secret|credential|bearer|password|hmac|signature|session|login|passport|helmet|cors|middleware|use\\( — the only matching line in the repository is app.use(express.json()) in src/index.ts. A second grep for headers|Authorization|req\\.header|process\\.env across src/ and .env.template matches exactly two lines, neither a credential. Absence claim; any new middleware, header or dependency invalidates it."
freshness_triggers:
  - ".env.template"
  - "order-service:build.gradle"
  - "package.json"
  - "sources/raw/specs/order-service-openapi.json"
  - "src/deliveryStatus.ts"
  - "src/generated/orderApi.ts"
  - "src/index.ts"
  - "src/orderClient.ts"
known_unknowns:
  - "Whether an upstream gateway, service mesh or ingress authenticates callers before they reach port 8083. delivery-bff has no Dockerfile, no CI workflow and no manifest of any kind — a find across sellflow/repos for yaml/yml/Dockerfile/compose/Chart/ingress/tf returns nothing from this repository at all — so a fronting control can neither be confirmed nor ruled out."
  - "What the carrier API requires of its clients. No API key, signing routine, mTLS material, vendor contract or integration document for the carrier exists anywhere in the repository or in sellflow-reef/sources; the only carrier URL in the code is the placeholder default https://api.carrier.example."
  - "Whether the absence of auth is accepted risk or oversight. No ADR, security review, ticket or minute covering delivery-bff was found in sellflow-reef/sources."
  - "Who is entitled to see a given order's delivery status, as a matter of policy. Nothing in business-rules.md or the 2021 cancellation-policy page states an access rule for delivery information."
tags:
  - "delivery"
  - "process"
  - "auth"
  - "security-gap"
aliases:
  - "delivery-bff auth"
  - "배송 BFF 인증"
relates_to:
  - type: "constrains"
    target: "[[API-DELIVERY]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-ERROR-HANDLING]]"
  - type: "depends_on"
    target: "[[PROC-DELIVERY-STATUS-SYNC]]"
  - type: "integrates_with"
    target: "[[PROC-INVENTORY-AUTH]]"
  - type: "constrains"
    target: "[[RISK-DELIVERY]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "Three lines, all base URLs and a retry count. No credential field exists to populate."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "Runtime dependencies are axios and express only; no auth library, no helmet, no cors."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "Carrier call: axios.get(url, { timeout: 3000 }) — no headers argument."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Sends only Content-Type; relative URL; no Authorization header and no place to add one."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "Only express.json(); the handler reads req.params.ordNo and nothing else."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "Five dependencies; spring-boot-starter-security is not among them."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "@PostMapping(\"/{ordNo}/cancel\") with no security annotation and no auth parameter."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "Global security bearerAuth (http/bearer/JWT); a 401 인증 실패 response on 29 of the 32 operations — the three /internal/* probes declare none."
notes: "The inbound gap and the outbound gap have different shapes, and the outbound one resolves in an unexpected direction: the declared requirement the client fails to meet is not enforced by the server either."
---

## Purpose

Every service artifact in this reef carries an auth process document. This one records an
absence: delivery-bff performs no authentication and no authorization, in either direction. It
does not identify its callers, and it does not identify itself to either system it calls.

Recording that as a finding rather than skipping the artifact matters because the two directions
have different shapes, and because the outbound gap resolves in a direction the previous pass
left open. Inbound, nothing is checked because nothing is there. Outbound, there is a documented
expectation the client does not meet — and, as of this pass, evidence that the server does not
enforce it either.

## Key Facts

- A case-insensitive grep over the whole repository (excluding `node_modules` and `.git`) for `auth|token|jwt|oauth|api[_-]?key|apikey|secret|credential|bearer|password|hmac|signature|session|login|passport|helmet|cors|middleware|use\(` returns exactly one line: `app.use(express.json());` → delivery-bff:src/index.ts
- A grep for `headers|Authorization|req\.header|process\.env` across `src/` and `.env.template` returns exactly two lines: the `CARRIER_API` env read and the generated client's `Content-Type` object — no credential and no inbound header access anywhere → delivery-bff:src/deliveryStatus.ts, delivery-bff:src/generated/orderApi.ts
- The only express middleware registered is `express.json()`, which parses a body the single route does not use → delivery-bff:src/index.ts
- The route handler's entire input surface is `req.params.ordNo`; it never touches `req.headers`, `req.cookies` or a client certificate → delivery-bff:src/index.ts
- `package.json` lists two runtime dependencies, `axios ^1.6.2` and `express ^4.18.2` — no auth library, no `helmet`, no `cors`, no session store → delivery-bff:package.json
- The carrier request carries no credential: `axios.get(\`${CARRIER_API}/tracking/${ordNo}\`, { timeout: 3000 })` passes a timeout and nothing else → delivery-bff:src/deliveryStatus.ts
- `.env.template` has three lines — `ORDER_API_BASE`, `CARRIER_API_BASE`, `RETRY_COUNT` — and no credential, key or secret field; there is not even a placeholder an operator could fill in → delivery-bff:.env.template
- All three of those variables are inert: the code reads `process.env.CARRIER_API` (a different name) and hardcodes `MAX_RETRY = 3`, while the order client uses no base URL at all → delivery-bff:src/deliveryStatus.ts, delivery-bff:src/generated/orderApi.ts
- The generated order client sends one header, `{ 'Content-Type': 'application/json' }`, and has no parameter, field, options object or interceptor through which a token could be supplied → delivery-bff:src/generated/orderApi.ts
- The order-service OpenAPI spec declares a global requirement, `security: [{ "bearerAuth": [] }]`, with scheme `{"type": "http", "scheme": "bearer", "bearerFormat": "JWT"}` → sellflow-docs:raw/specs/order-service-openapi.json
- That spec documents a `401` response described as 인증 실패 ("authentication failure") on every one of its 30 paths, including `/orders/{ordNo}/cancel` → sellflow-docs:raw/specs/order-service-openapi.json
- **order-service does not enforce it.** `build.gradle` declares five dependencies — `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `flyway-core`, `mysql-connector-java`, `spring-boot-starter-test` — and `spring-boot-starter-security` is not among them → order-service:build.gradle
- A grep of order-service for `spring-boot-starter-security|SecurityConfig|WebSecurityConfigurer|@PreAuthorize|jwt|bearer|Authorization|authentication|filterChain|interceptor` across `*.java`, `*.xml`, `*.yml` and `*.gradle` returns nothing → order-service (whole tree)
- `OrderController.cancel` is annotated only `@PostMapping("/{ordNo}/cancel")` and takes `@PathVariable` and `@RequestBody` — no principal, no security annotation → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- The client would not reach order-service regardless of credentials: it issues a **relative** `fetch('/api/v1/orders/${ordNo}/cancel')`, while the spec's servers are `https://order.internal.sellflow.co.kr` and `https://order-stg.internal.sellflow.co.kr` and the live controller is mapped at `/orders`, with no `/api/v1` prefix anywhere → delivery-bff:src/generated/orderApi.ts, sellflow-docs:raw/specs/order-service-openapi.json, order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java

## Steps

### The commands that establish the negatives

```
$ cd /sellflow/repos
$ grep -rniE 'auth|token|jwt|oauth|api[_-]?key|apikey|secret|credential|bearer|password|hmac|\
signature|session|login|passport|helmet|cors|middleware|use\(' delivery-bff \
    --exclude-dir=node_modules --exclude-dir=.git
delivery-bff/src/index.ts:6:app.use(express.json());

$ grep -rnE 'headers|Authorization|req\.header|process\.env' delivery-bff/src delivery-bff/.env.template
delivery-bff/src/deliveryStatus.ts:4:const CARRIER_API = process.env.CARRIER_API ?? 'https://api.carrier.example';
delivery-bff/src/generated/orderApi.ts:31:    headers: { 'Content-Type': 'application/json' },

$ grep -rniE 'spring-boot-starter-security|SecurityConfig|WebSecurityConfigurer|@PreAuthorize|\
jwt|bearer|Authorization|authentication|filterChain|interceptor' order-service \
    --include='*.java' --include='*.xml' --include='*.yml' --include='*.gradle'
(no output)
```

Three greps, three negatives. The first says delivery-bff has no auth construct; the sole hit is
a JSON body parser matched by the deliberately over-broad `use\(` alternative. The second says no
credential is sent and no inbound header is ever read. The third is the new one, and it settles a
question the previous pass left open.

### Inbound — nothing gates the endpoint

1. A request arrives at `GET /delivery/:ordNo` on port 8083 (`src/index.ts`).
2. `express.json()` parses a body that this route does not use. It is the only middleware in the
   application, so no pre-handler check occurs.
3. The handler runs unconditionally. There is no caller identity, so there is nothing to
   authorize against and no check that the requester may view this order number.
4. It calls `syncDeliveryStatus(req.params.ordNo)` and returns whatever comes back.

The consequence is stated plainly: order numbers in this estate are sequential-looking date-prefixed
strings (`20260918000123` in the reef's own worked examples), so anyone who can reach port 8083
can enumerate delivery status — carrier code, status and timestamp — across the customer base, with
no rate limit, no credential and no log line naming them. `src/logger.ts` is three `console`
wrappers and the handler logs nothing on the success path at all.

### Outbound to the carrier — no credential, and nowhere to put one

`syncDeliveryStatus` issues `axios.get(url, { timeout: 3000 })`. There is no `headers` key in the
options object, no axios instance with defaults, no interceptor, and no signing step. Whatever
the carrier requires of its clients, this code supplies none of it.

`.env.template` confirms this is structural rather than an omission at deploy time. Its three
lines are `ORDER_API_BASE`, `CARRIER_API_BASE` and `RETRY_COUNT` — base URLs and a tuning knob.
There is no `CARRIER_API_KEY`, no `CARRIER_SECRET`, no token field, so an operator who obtained a
carrier key from 배송관리팀 would have no documented place to put it and no code path that would
read it. And the template is doubly misleading here: the variable the code actually reads is
`CARRIER_API`, not `CARRIER_API_BASE`, so even the hostname in the template has no effect and the
service falls back to the placeholder `https://api.carrier.example` (see
[[PROC-DELIVERY-STATUS-SYNC]]).

That combination — a placeholder host and no credential — is self-consistent in an uncomfortable
way. A service calling a real carrier without a key would fail loudly and get fixed. A service
calling a non-existent host fails quietly into the stale fallback described in
[[PROC-DELIVERY-ERROR-HANDLING]], and the missing credential is never reached as a question.

### Outbound to order-service — the declared-versus-sent gap, and who is out of step

| | order-service OpenAPI 2.4.0 (2022-11-04) | delivery-bff generated client (2022-11-08) | order-service as it runs today |
|---|---|---|---|
| Credential | JWT bearer, required globally | none sent, none possible | **none required** — no Spring Security on the classpath |
| 401 | documented as 인증 실패 on 29 of the spec's 32 operations (not on the three `/internal/*` probes) | not handled; would fall through to `res.json()` | cannot be produced |
| Path | servers `https://order.internal.sellflow.co.kr`, path `/orders/{ordNo}/cancel` | relative `/api/v1/orders/{ordNo}/cancel` | `@RequestMapping("/orders")` + `/{ordNo}/cancel` |
| `X-Request-Id` | offered on 29 of the 32 operations | not sent | not read |

The previous pass recorded "whether order-service actually enforces the bearerAuth its OpenAPI
spec declares" as a known unknown. It does not. `build.gradle` has no security starter, no
`SecurityConfig` class exists, no `@PreAuthorize` appears, and the cancel controller takes no
principal. The spec's `security` block was emitted by springdoc from configuration that is not in
the repository, or was added by hand — the spec's own description says 일부 설명은 수기
보정되었습니다 ("some descriptions have been manually corrected"). Either way, the declared
requirement is aspirational.

This resolves the gap in the *opposite* direction from the one that would be reassuring. The BFF
is not failing to authenticate against a server that would reject it; both sides are open. The
practical readings are:

- The client's missing `Authorization` header is not why the cancel path does not work. The
  relative URL is — `fetch('/api/v1/...')` has no origin to resolve against outside a browser
  (see [[PROC-DELIVERY-ERROR-HANDLING]]) — and the path prefix would be wrong even if it did.
- Nobody can rely on the spec's `401` to reason about access control, and no consumer's code
  should branch on it.
- The absence is estate-wide, not service-local. inventory-api has the same total absence
  ([[PROC-INVENTORY-AUTH]]), and order-service — the system holding customer and partner order
  data — has it too. Treating delivery-bff's gap as a delivery problem would mis-scope the
  remediation.

### What this artifact does not claim

It does not claim the service is exposed to the public internet, nor that an operator has
overlooked the problem. A gateway or mesh could terminate authentication in front of port 8083
and the repository would look exactly like this either way — although delivery-bff is the thinnest
case in the estate for that hypothesis, since it has no Dockerfile, no workflow and no manifest of
any kind, so nothing in version control describes how it is deployed at all. That question stays
open. The security consequences are carried as a theme in [[RISK-DELIVERY]].

## Worked Examples

**Inbound, unauthenticated.** `GET /delivery/20260918000123` with no headers is served
identically to one carrying any headers; the handler never looks. Two different customers, an
internal tool, and an anonymous caller all receive the same response for the same `ordNo`, and
nothing distinguishes them in the logs.

**Outbound to the carrier, uncredentialed.**

```http
GET /tracking/20260918000123 HTTP/1.1
Host: api.carrier.example
```

No `Authorization`, no API key, no signature — and a placeholder host, because `CARRIER_API_BASE`
in `.env.template` is not the variable the code reads.

**Outbound to order-service, uncredentialed and unroutable.** `requestCancel('20260918000123', '02')`
attempts:

```http
POST /api/v1/orders/20260918000123/cancel HTTP/1.1
Content-Type: application/json
```
```json
{ "sayuCd": "02" }
```

Against the spec's declared contract this should be rejected with `401 인증 실패`. Against
order-service as it actually runs, no credential is required — but the request never arrives:
the URL is relative, and the live route is `/orders/{ordNo}/cancel` with no `/api/v1` prefix.

## Related

- [[API-DELIVERY]] — the endpoints that run without a credential check
- [[CON-ORDER-DELIVERY]] — the boundary where the declared bearerAuth expectation lives
- [[PROC-DELIVERY-ERROR-HANDLING]] — what happens to an unauthenticated call that fails
- [[PROC-DELIVERY-STATUS-SYNC]] — the placeholder carrier host that hides the missing key
- [[PROC-INVENTORY-AUTH]] — the same total absence in another service, establishing the pattern
- [[RISK-DELIVERY]] — the absence recorded as a tracked risk theme
- [[SYS-DELIVERY]] — the service overview
- [[SYS-ORDER]] — the callee whose spec declares an auth requirement its code does not enforce
