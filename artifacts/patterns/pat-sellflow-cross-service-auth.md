---
id: "PAT-SELLFLOW-CROSS-SERVICE-AUTH"
type: "pattern"
title: "Four Services, Four Postures, No Service-to-Service Authentication"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Every claim restated from the four PROC-*-AUTH artifacts was re-verified against code on 2026-09-19 before being written here. The decisive check is one command: a case-insensitive grep for spring-boot-starter-security|SecurityConfig|WebSecurityConfigurer|@PreAuthorize|@Secured|RolesAllowed|jwt|bearer|oauth|api[_-]?key|Authorization|HTTPBearer|APIKeyHeader|Depends\\(|middleware|passport|helmet|X-Api across all five repository roots, excluding node_modules, which exits 1 with zero matches. This is a finding of absence and decays the moment any dependency is added — re-run it first."
freshness_triggers:
  - "delivery-bff/package.json"
  - "inventory-api/app/main.py"
  - "inventory-api/requirements.txt"
  - "order-service/build.gradle"
  - "order-service/src/main/java/kr/co/sellflow/order/config/WebConfig.java"
  - "settlement-anomaly/app/main.py"
  - "settlement-batch/build.gradle"
  - "settlement-batch/src/main/resources/application.yml"
  - "sources/raw/specs/order-service-openapi.json"
known_unknowns:
  - "Whether a gateway exists at all. The only evidence in the entire estate is one Korean comment in WebConfig; no config file, registry entry, workflow, manifest or document in this reef names a gateway product, host, policy or token issuer. Checked: all five repos file by file, services.yaml, and the four infra/*/runtime.md extractions."
  - "If a gateway exists, whether it validates the bearerAuth JWT the 2022 spec declares — with which issuer, audience, signing key or clock skew. Unverifiable from these repositories."
  - "What network boundary, if any, stands in front of the two FastAPI services. No ingress manifest, service-mesh policy, firewall rule or Kubernetes resource exists in any of the five repos, so reachability is unknown rather than known-open."
  - "What database credential production actually uses. Neither Java service declares a password property, both Python services omit the password key from their DSN, and no workflow references a secret — the credential must live in deploy.sh, which is not in version control."
  - "Whether the absence of authentication was ever decided, accepted or risk-registered. No ADR, security review, minute or ticket in this reef discusses it."
tags:
  - authentication
  - authorization
  - cross-system
  - security
  - shared-database
aliases:
  - "서비스 간 인증"
  - "service-to-service auth"
relates_to:
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-DOC-CODE-DRIFT]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-AUTH]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-AUTH]]"
  - type: "refines"
    target: "[[PROC-ORDER-AUTH]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-RUNTIME]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-AUTH]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "axios and express only — no auth, helmet, cors or session library"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "express.json() is the only middleware; the handler reads req.params only"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "DSN with no password key, database hardcoded to sellflow_order"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "Unauthenticated write endpoint that also reads ORDER_DTL"
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "Five dependencies, no spring-boot-starter-security"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java"
    notes: "The estate's entire declared access-control surface: one CORS mapping"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "Two routes, no dependency or middleware"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "Cross-owner write authorised by a 2019 verbal agreement in a comment"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
    notes: "datasource url and username, no password property; clustered JDBC Quartz store"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "§4 — the only written access model in the estate, and it is a human one"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "No authentication field exists for any service"
  - category: "external"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "Global security: [{bearerAuth: []}]; 401 declared on 29 of the 32 operations — the three /internal/* probes declare none"
notes: "Restates the four PROC-*-AUTH artifacts only where their claims were independently re-verified against code in this pass. Its contribution is the estate-level reading: four postures that look like four policies but are one absence."
---

## Overview

Four services expose something a caller can reach — order-service over HTTP, inventory-api over HTTP, delivery-bff over HTTP, settlement-anomaly over HTTP — and a fifth, settlement-batch, exposes no HTTP at all. Read individually, each looks like a different authentication posture: one service delegates to a gateway, two have simply never had any, one has no surface to protect. Read together, they are not four policies. They are one absence with four different explanations attached to it.

The decisive check is a single command. A case-insensitive grep across all five repository roots for every plausible spelling of authentication — `spring-boot-starter-security`, `SecurityConfig`, `WebSecurityConfigurer`, `@PreAuthorize`, `@Secured`, `RolesAllowed`, `jwt`, `bearer`, `oauth`, `api[_-]?key`, `Authorization`, `HTTPBearer`, `APIKeyHeader`, `Depends(`, `middleware`, `passport`, `helmet`, `X-Api` — returns zero lines and exits 1. There is no code in this estate that authenticates a caller.

What makes the finding worth a pattern artifact rather than a bullet in each service's risk page is the asymmetry between what is *declared* and what is *enforced*. A specification declares global bearer authentication on thirty paths; no code reads a token. A registry that calls itself the single standard has no authentication field at all. A handover document describes an access model, and it is a model for people requesting spreadsheet access, not for services calling services. The estate's real access control is network reachability plus a shared database account, and neither of those is described anywhere.

## Key Facts

- **A grep for every plausible authentication construct across all five repos returns nothing.** `spring-boot-starter-security|SecurityConfig|WebSecurityConfigurer|@PreAuthorize|@Secured|RolesAllowed|jwt|bearer|oauth|api[_-]?key|Authorization|HTTPBearer|APIKeyHeader|Depends\(|middleware|passport|helmet|X-Api`, case-insensitive, excluding `node_modules`: zero matches, exit status 1 → verified by grep, 2026-09-19, over all five repository roots
- **order-service — declared at the edge, enforced nowhere.** `build.gradle` lists exactly five dependencies: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `flyway-core`, `mysql-connector-java` (runtimeOnly), `spring-boot-starter-test`. Without `spring-boot-starter-security` on the classpath there is no filter chain, no default credential and no prompt → order-service:build.gradle
- The service's entire access-control surface is one CORS line: `registry.addMapping("/api/**").allowedOrigins("https://admin.sellflow.co.kr")`. CORS is a browser convention, not an access control; a `curl` or a server-side call ignores it completely → order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java
- The cancel endpoint is not even inside that pattern. `OrderController` is mapped at `/orders`, so `POST /orders/{ordNo}/cancel` carries no CORS mapping at all, and its handler takes a `@PathVariable` and a `@RequestBody` and consults no caller identity → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- **The only claim of authentication anywhere in the code is a comment**: `어드민 도메인만 허용. 앱은 게이트웨이를 통해 들어온다.` — "only the admin domain is allowed; the app comes in through the gateway". No gateway product, host, policy or issuer is named in any repo, in `services.yaml`, or in any of the four `infra/*/runtime.md` extractions → order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java, sellflow-docs:context/registry/services.yaml
- **The 2022 specification declares what no code enforces.** `order-service-openapi.json` carries a top-level `security: [{"bearerAuth": []}]` with scheme `{"type":"http","scheme":"bearer","bearerFormat":"JWT"}`, across 30 paths, each documenting a `401` described as 인증 실패 ("authentication failure"). Only two of those 30 paths have a handler in today's source, and neither checks a token → sellflow-docs:raw/specs/order-service-openapi.json, order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java
- **inventory-api — no authentication, on a write endpoint, that also reads another team's table.** `app/main.py` imports `logging`, `FastAPI`, `HTTPException`, `BaseModel` and two DB helpers; it registers no middleware and no `Depends`. `POST /stock/restock` performs `UPDATE stock_item SET available_qty = available_qty + %(q)s`, and first performs `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s` → inventory-api:app/main.py
- Its three pinned dependencies are `fastapi==0.104.1`, `uvicorn==0.24.0`, `pymysql==1.1.0` — no JWT, OAuth, crypto or session library — and the container runs `uvicorn app.main:app --host 0.0.0.0 --port 8000`, all interfaces, plain HTTP, no reverse proxy in the image → inventory-api:requirements.txt, inventory-api:Dockerfile
- **delivery-bff — no authentication inbound, and no way to send one outbound.** Its only middleware is `express.json()`; the handler's entire input surface is `req.params.ordNo`. `package.json` has two runtime dependencies, `axios` and `express`. The generated order client sends exactly one header, `{'Content-Type': 'application/json'}`, with no parameter, options object or interceptor through which a token could be supplied → delivery-bff:src/index.ts, delivery-bff:package.json, delivery-bff:src/generated/orderApi.ts
- **settlement-batch — authenticates to a database, not to a service.** It declares `starter-batch`, `starter-quartz`, `starter-jdbc`, `flyway-core` and the MySQL connector: no web starter, therefore no HTTP surface to authenticate. Its only trigger surface is Quartz with `job-store-type: jdbc` and `isClustered: true`, so anything that can write the Quartz tables can cause a settlement run → settlement-batch:build.gradle, settlement-batch:src/main/resources/application.yml
- **And its cross-owner write is authorised by a code comment.** `MarkSettledTasklet` issues `UPDATE ORDER_MST SET SANGTAE_CD='JUNGSAN_WANRYO'` against the order team's table with no API call, token or grant check, justified by `ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)` — "ORDER_MST is a table owned by the order team, but under the consolidated-DB policy the settlement batch updates it directly (agreed 2019)" → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java
- **settlement-anomaly — two unauthenticated routes, one of which writes.** `POST /detect` joins three tables and inserts `SETTLEMENT_ANOMALY` rows for any caller that can reach port 8090; `GET /health` returns the loaded model version to the same population. Neither carries a dependency or middleware → settlement-anomaly:app/main.py
- **No credential material exists in any repository, in either direction.** A case-insensitive grep for `password|passwd|secret|credential` across all five repos returns exactly two lines: an empty `DB_PASSWORD=` in `order-service/.env.example` and `password: CHANGE_ME` in `application-local.yml.example` → verified by grep, 2026-09-19, over all five repository roots
- Consistent with that, neither Java service declares a datasource password property and neither Python service includes a `password` key in its pymysql DSN; both Python DSNs also hardcode `database="sellflow_order"`, so both connect to the shared instance as the default user `sellflow` → settlement-batch:src/main/resources/application.yml, inventory-api:app/db.py, settlement-anomaly:app/db.py
- **The registry has no authentication field for any service.** `services.yaml` declares itself 이 파일이 서비스·소유팀·저장소의 단일 기준이다 ("this file is the single standard for services, owning teams and repositories") and records `name`, `repo`, `owner_team`, `runtime`, `db`, `endpoints` and `notes` — nothing about access → sellflow-docs:context/registry/services.yaml
- **The only written access model in the estate is a human one, and it is incomplete.** The 2025 handover's permission table lists 정산 DB (read) via an infrastructure-team request, 정산 어드민 correction rights 팀장 승인 후 ("after team-leader approval"), and 파트너 포털 access marked `TBD` → sellflow-docs:context/handover/2025-03_정산팀_인수인계.md (§4)

## Where It Appears

| Service | Inbound surface | Posture as it appears | What is actually enforced | Sensitivity of what is exposed |
|---|---|---|---|---|
| order-service | HTTP 8081 — `POST /orders/{ordNo}/cancel`, `GET /api/v1/orders/search` | "terminated at the gateway" | one CORS mapping on `/api/**`, which the cancel endpoint is not under; no token check anywhere | cancels any order by number; searches orders over a date range |
| inventory-api | HTTP 8000 — `POST /stock/restock`, `GET /stock/{sku}` | none, never had any | nothing | writes stock quantities; reads `ORDER_DTL` |
| delivery-bff | HTTP 8083 — `GET /delivery/:ordNo` | none, never had any | nothing | proxies carrier tracking by order number |
| settlement-anomaly | HTTP 8090 — `POST /detect`, `GET /health` | none, never had any | nothing | writes anomaly rows; reads settlement amounts and fees |
| settlement-batch | none (no web starter) | "no surface, therefore no problem" | a database connection, plus write access to the clustered Quartz tables | moves the estate's money and overwrites another team's table |

Two things stand out when the five are read as one estate rather than five services.

**The declared surface and the real surface do not overlap.** What is declared lives in two documents — a 2022 OpenAPI spec asserting global bearer auth, and a registry with no auth field — and what is enforced lives in one CORS line that does not cover the endpoint that matters. Everything else that functions as access control is invisible to both: network reachability, which no manifest in these repos describes, and a shared MySQL account named `sellflow`, which four services share.

**The service with no HTTP surface has the widest blast radius.** settlement-batch is the one component that genuinely cannot be called by a stranger over HTTP, and it is also the one that pays partners, overwrites `ORDER_MST`, and mutates the order service's outbox state. Its authentication is a database connection string. The 2025-07-12 incident is what that is worth in practice: an unintended second run produced duplicate payment requests for 17 partners, roughly 42,000,000 KRW, resolved as 중복 지급 요청 취소 불가 확인, 차월 상계로 처리 결정 — "confirmed the duplicate payment requests cannot be cancelled; decided to handle it by next-month offset" → sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

**And the one real client sends nothing and would not arrive anyway.** delivery-bff's `requestCancel` issues a relative `fetch('/api/v1/orders/${ordNo}/cancel')` from a Node process — no base URL, a path prefix order-service does not serve, from a function no code calls. order-service's `InventoryClient` posts to `/inventory/restore`, a path inventory-api does not serve, from a class nothing calls. Both of the estate's service-to-service HTTP clients are unauthenticated *and* non-functional, which is why the absence has never produced a symptom → delivery-bff:src/generated/orderApi.ts, order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java

## Design Intent

**Determinable for one service, not determinable for the rest — and the distinction matters.**

For order-service there is a stated intent, of a kind. The `WebConfig` comment asserts that the app's traffic arrives through a gateway, and the 2022 spec's global `bearerAuth` is consistent with a design in which a gateway validates a JWT and the service trusts what reaches it. That is a recognised architecture. Whether it is *this* architecture cannot be established: no gateway is named anywhere in the five repos, the registry, or the extracted runtime documents, and the service's own public hostname in the registry (`https://api.sellflow.co.kr/orders`) differs from the spec's servers (`https://order.internal.sellflow.co.kr`), which differ again from the path prefix the only generated client uses. Three different pictures of the same edge, none of them verifiable.

For inventory-api, delivery-bff and settlement-anomaly there is **no stated intent at all** — no comment, no README section, no ticket, no minute. Their READMEs discuss vocabulary drift, retry behaviour and detection types. The absence of authentication is not defended, deferred or acknowledged; it is simply not mentioned. Saying more than that would be guessing, and it is recorded in `known_unknowns` instead.

For settlement-batch the intent is recoverable and is explicitly *not* about authentication: the 2019 consolidated-database agreement made direct table access the sanctioned integration mechanism between teams, which is a decision to replace service authorisation with database access rather than to skip it. See [[DEC-SELLFLOW-SHARED-DB]]. That agreement is recorded in a code comment and a migration header — not in an access-control system, a grant, or a reviewable document.

## Trade-offs

**What this buys.** Almost nothing was spent on it, and for an estate whose real integration path is a shared database, spending more on HTTP authentication would have secured the wrong door. The honest version of the trade-off is that Sellflow integrates through tables, not through APIs (see [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]]), and a bearer token on `POST /stock/restock` would not have changed who can `UPDATE ORDER_MST`. Four services sharing one database account is a single, coherent (if very coarse) access model, and it is enforced by something real.

There is also a genuine simplification: no token issuance, rotation, clock skew, or expiry-related outage has ever been possible here, and no credential has ever leaked from these repositories because none exists in them.

**What it costs.**

1. **The coarse model is maximally coarse.** One MySQL account, `sellflow`, shared by four services, reaching every table in `sellflow_order`. There is no separation between the service that reads stock and the service that pays partners.
2. **Any reachable caller is a fully privileged caller.** Because reachability is the only gate, the security of `POST /detect`, `POST /stock/restock` and `POST /orders/{ordNo}/cancel` is entirely a property of a network boundary that no file in any of these repositories describes. That is not "probably fine"; it is unknown, which is worse than either answer.
3. **The specification actively misleads.** A reader of `order-service-openapi.json` — or any tool that generates a client from it — will conclude that a bearer token is required and that a `401` is possible. Neither is true. This is the clearest instance in the estate of [[PAT-SELLFLOW-DOC-CODE-DRIFT]]'s "aspiration recorded as description".
4. **There is no attribution anywhere.** `SETTLEMENT_ANOMALY.REVIEWED_BY` and `CANCEL_RECON_QUEUE.PROCESSED_BY` are columns that would record who did something; neither has a writer, and no service knows who its caller is, so neither could be filled even if someone wanted to. Procedure v1.1 §5 requires correction history to be recorded and retained five years, against a system with no concept of an actor.
5. **The deployment path is outside review.** All three settlement-batch CD workflows are `workflow_dispatch` with no `permissions:` block, no environment gate and no reviewer, building with `-x test`, then invoking `./deploy.sh $ENVIRONMENT` — a script not in the repository. Whatever production credential exists lives there, unreviewable → settlement-batch:.github/workflows/settlement-batch-prod-cd.yml
6. **Nobody has decided this.** The most costly property is not the absence itself but that it appears in no risk register, ADR or review in this reef. An accepted risk can be revisited; an unnoticed one cannot.

## Agent Guidance

- **Never describe any Sellflow service as authenticated.** If asked "how does service X authenticate", the accurate answer is that it does not, and that the only gate is network reachability plus a shared database account.
- **Treat the 2022 OpenAPI spec's `bearerAuth` as documentation of an intention, never as behaviour.** Any client generated from that spec will be built to send a token that nothing reads, against the 29 spec paths that have no handler.
- **Do not repeat the gateway claim as fact.** One comment in `WebConfig` is the sole evidence. Say "a code comment asserts a gateway; no gateway is named, configured or verifiable in any of the five repositories" — and if the question requires an answer, it needs a human: see `.reef/questions-for-owner.md`.
- **CORS is not access control.** `allowedOrigins("https://admin.sellflow.co.kr")` constrains browsers on other origins and nothing else. It also does not cover `POST /orders/{ordNo}/cancel`, which is mapped at `/orders`, outside `/api/**`.
- **When reasoning about who can do something, reason about database access, not about API access.** The question "can service X cancel an order" is less useful here than "can service X write `ORDER_MST`", and for four services the answer to the second is yes.
- **Before writing any change that adds an endpoint, note that it will be unauthenticated by default and say so.** There is no framework-level default to inherit — no security starter, no middleware, no `Depends`.
- **Do not propose a token scheme as a first step without pricing the database problem.** Adding bearer auth to the HTTP surface would leave every real integration path — `MarkSettledTasklet`'s `UPDATE`, inventory-api's `SELECT` of `ORDER_DTL`, settlement-anomaly's three-table join, the outbox relay — completely untouched.
- **The clustered Quartz store is part of the attack surface.** Anything able to write the Quartz tables in `sellflow_order` can schedule a settlement run, and the 2025-07-12 retrospective records what one extra run cost.

## Related

- [[PROC-ORDER-AUTH]] — order-service's posture in full, including the gateway question
- [[PROC-INVENTORY-AUTH]] — the unauthenticated write endpoint and its shared-DB reach
- [[PROC-DELIVERY-AUTH]] — no inbound auth, and no outbound mechanism to carry a credential
- [[PROC-SETTLEMENT-AUTH]] — authentication to a database rather than to a service
- [[DEC-SELLFLOW-SHARED-DB]] — the decision that replaced service authorisation with table access
- [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]] — the real integration surface an HTTP token would not cover
- [[PAT-SELLFLOW-DOC-CODE-DRIFT]] — the spec that declares an authentication scheme no code enforces
- [[PROC-SELLFLOW-RUNTIME]] — ports, processes and deployment paths
