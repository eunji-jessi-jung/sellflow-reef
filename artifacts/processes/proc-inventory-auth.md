---
id: "PROC-INVENTORY-AUTH"
type: "process"
title: "Inventory API Authentication and Authorization"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-verified on 2026-09-19, and again the same day after inventory-api added sqlalchemy and alembic to requirements.txt — neither is an authentication package and the absence claim is unchanged. Verified with an exhaustive case-insensitive grep over the whole inventory-api working tree for every authentication, credential and transport-security token (auth|token|jwt|oauth|api[_-]?key|secret|credential|bearer|password|hmac|signature|login|Depends|middleware|security|HTTPBearer|APIKeyHeader) — the grep returned exit status 1, zero matching lines, zero matching files. This is an absence claim, so it is only as current as the repository snapshot; a single dependency added to app/main.py invalidates it."
freshness_triggers:
  - ".env.sample"
  - "Dockerfile"
  - "app/config.py"
  - "app/db.py"
  - "app/main.py"
  - "app/routers.py"
  - "requirements.txt"
known_unknowns:
  - "Whether an API gateway, service mesh, ingress or sidecar authenticates callers in front of port 8000. Re-checked on 2026-09-19 by finding every yaml/yml/Dockerfile/compose/Chart/ingress/tf file under sellflow/repos: inventory-api contributes exactly one, its Dockerfile, and only order-service and settlement-batch have any .github/workflows at all. So no deployment topology for this service exists in any repository, and a gateway can neither be confirmed nor ruled out from the source."
  - "Whether the deployed database user actually has a password. app/db.py builds its DSN with no password key at all, so pymysql sends an empty password; whether the real MySQL account for user 'sellflow' is passwordless, or whether some wrapper injects one outside the code, is not visible."
  - "Whether the deployed database user's grants are scoped. The code needs SELECT on ORDER_DTL and UPDATE on stock_item in the same schema; whether the account is limited to those two is not determinable from the repository."
  - "Who is permitted to restock as a matter of policy rather than mechanism. Re-checked against sellflow-docs:context/business-rules.md (취소정책 sheet) and sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html: both state which reason codes restock, neither states who may ask."
  - "Whether TLS terminates anywhere in front of the service. The container serves plain HTTP on 0.0.0.0:8000 and the one known caller's default base URL is http://inventory-api.internal."
tags:
  - inventory
  - auth
  - security
  - sellflow
aliases:
  - "inventory auth"
  - "재고 API 인증"
relates_to:
  - type: "depends_on"
    target: "[[API-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-ERROR-HANDLING]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "refines"
    target: "[[RISK-INVENTORY]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
  - type: "integrates_with"
    target: "[[PROC-DELIVERY-AUTH]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:.env.sample"
    notes: "Two lines, DB_URL and LOG_LEVEL. No credential field exists to inject one into."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:Dockerfile"
    notes: "uvicorn binds 0.0.0.0:8000 with no proxy, no auth layer, no TLS."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic.ini"
    notes: "sqlalchemy.url points at the inventory database, contradicting app/db.py."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "DB_URL, RESTOCKABLE_REASONS, LOG_LEVEL only. Imported by no application module."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "DSN with no password key and a hardcoded database name of sellflow_order."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "No Depends, no middleware, no security scheme, no header access."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:requirements.txt"
    notes: "Three packages; none is an auth library."
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "No spring-boot-starter-security, so the estate's largest service has no auth either."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "Sends no credential, and calls a path this service does not serve."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Registry entry for inventory-api records runtime and db only; no auth field for any service."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "The 2021 cancellation policy names inventory-api as the restock processor without any access rule."
notes: "Every claim here is a negative established by exhaustive grep; the grep command itself is reproduced in the body so a reader can re-run it."
---

# Inventory API Authentication and Authorization

## Purpose

To record what protects the inventory endpoints. The answer is nothing that exists in this
repository — not a token, not a key, not a header check, not a dependency. This artifact states
that as a verified negative rather than an impression, shows the command that verified it, and
then does the part that matters more than the absence itself: works out what the absence means
given what these particular endpoints actually do.

## Key Facts

- An exhaustive case-insensitive grep over the whole working tree for `auth|token|jwt|oauth|api[_-]?key|secret|credential|bearer|password|hmac|signature|login|Depends|middleware|security|HTTPBearer|APIKeyHeader` returns zero lines and zero files, exit status 1 → inventory-api (whole tree), command reproduced below
- A second grep for `Header|Request|request\.|headers` across `*.py` matches only `class RestockRequest(BaseModel)` and `def restock(req: RestockRequest)` — no handler in the service ever touches a request header → inventory-api:app/main.py
- `app/main.py` registers no middleware and no `Depends`; its entire import list is `logging`, `FastAPI`, `HTTPException`, `BaseModel` and the two database helpers → inventory-api:app/main.py
- `requirements.txt` pins five packages — `fastapi==0.104.1`, `uvicorn==0.24.0`, `pymysql==1.1.0`, `sqlalchemy==2.0.23`, `alembic==1.12.1` — with no JWT, OAuth, crypto or session library among them; the two additions serve the Alembic migration chain, not the request path → inventory-api:requirements.txt
- `.env.sample` has two lines, `DB_URL` and `LOG_LEVEL`, so no credential is expected to be injected at deploy time either → inventory-api:.env.sample
- `app/config.py` is imported by nothing in `app/` — the only import of it anywhere is `from app.config import RESTOCKABLE_REASONS` in the test suite, so even the module that could hold a secret is dead code → inventory-api:app/config.py, inventory-api:tests/test_restore.py
- The container runs the app directly: `CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]` — all interfaces, plain HTTP, no reverse proxy in the image → inventory-api:Dockerfile
- The unauthenticated `POST /stock/restock` is a **write**: it issues `UPDATE stock_item SET available_qty = available_qty + %(q)s WHERE sku = %(sku)s` → inventory-api:app/main.py
- The same unauthenticated request first **reads another service's table**: `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s` → inventory-api:app/main.py
- That is possible because `app/db.py` hardcodes `"database": "sellflow_order"` — inventory-api connects to the order instance, not to an inventory database → inventory-api:app/db.py
- The DSN carries no password key whatsoever: `{"host": ..., "user": ..., "database": "sellflow_order", "charset": "utf8mb4", "cursorclass": ...}` → inventory-api:app/db.py
- The DB user defaults to `"sellflow"`, a generic shared identity rather than a service-scoped one → inventory-api:app/db.py
- The only known caller sends no credential and no headers object: `restTemplate.postForEntity(baseUrl + "/inventory/restore?ordNo=" + ordNo + "&reason=" + sayuCd, null, Void.class)` → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java
- That caller's path, `/inventory/restore`, is served by neither `app/main.py` (`/stock/restock`) nor `app/routers.py` (`/inventory/stock/{sku_cd}`, `/inventory/health`), and `routers.py` is never included in the app — so the write endpoint has **no working legitimate caller at all** → inventory-api:app/main.py, inventory-api:app/routers.py, order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java
- The absence is not local to this service: `order-service/build.gradle` declares no `spring-boot-starter-security`, and a grep of order-service for `SecurityConfig|WebSecurityConfigurer|@PreAuthorize|jwt|bearer|Authorization|filterChain` also returns nothing → order-service:build.gradle
- `services.yaml`, which calls itself 단일 기준 ("the single standard"), records `name`, `repo`, `owner_team`, `runtime`, `db` and `endpoints` for inventory-api and has no authentication field for any service in the file → sellflow-docs:context/registry/services.yaml
- `README.md` names the owner (데이터플랫폼본부 재고팀) and warns only about vocabulary drift under its 주의 ("caution") heading; it makes no access claim, true or false → inventory-api:README.md

## Steps

### The command that establishes the negative

```
$ cd /sellflow/repos
$ grep -rniE 'auth|token|jwt|oauth|api[_-]?key|secret|credential|bearer|password|hmac|signature|login|Depends|middleware|security|HTTPBearer|APIKeyHeader' inventory-api
$ echo $?
1
```

No output, exit status 1. The same pattern run with `-l` lists no files. A companion grep for
header access (`Header|Request|request\.|headers` over `*.py`) matches only the Pydantic model
name `RestockRequest` and the parameter `req: RestockRequest` — that is, the only thing in this
service resembling a "request" is a body schema, never a header.

### The request path as it actually executes

1. A TCP connection reaches the container on port 8000. Nothing in the image inspects its origin,
   and nothing terminates TLS (`Dockerfile`).
2. uvicorn hands the request to the FastAPI application. No middleware is registered, so no
   pre-handler check runs (`app/main.py`).
3. The route's dependency list is empty, so FastAPI invokes the handler directly (`app/main.py`).
4. The handler validates the request *shape* — `ord_no: str`, `reason_code: str` for restock, a
   path parameter for the lookup. Shape validation is not identity validation, and `reason_code`
   carries no pattern or enum constraint (`app/main.py`).
5. `fetch_one` opens a fresh pymysql connection with a fixed user, an empty password and the
   hardcoded database `sellflow_order`, and reads `ORDER_DTL` (`app/db.py`, `app/main.py`).
6. `execute` opens a second connection and commits an `UPDATE` against `stock_item`
   (`app/db.py`, `app/main.py`).

### What the absence means here specifically

Three properties of these endpoints make the missing credential more than a checklist item.

**It mutates money-adjacent state.** `POST /stock/restock` adds quantity to `available_qty`. The
only gate on the operation is a reason code supplied by the caller in the same body the caller
controls. Anyone who can reach the port and guess an order number can inflate stock by submitting
`{"ord_no": "...", "reason_code": "01"}`, and can repeat it — the handler is not idempotent, so
the same request replayed *n* times adds the quantity *n* times (see
[[PROC-INVENTORY-ERROR-HANDLING]]).

**It is a cross-domain read.** An unauthenticated request to the inventory service causes a
`SELECT` against the order domain's `ORDER_DTL`. The connection is to `sellflow_order`, so this
service holds a live handle on the order estate's schema with a generic user and no password in
the DSN. An attacker does not learn order contents directly — the handler returns only
`{"restocked": true}` — but the service's existence turns "reach inventory's port" into "cause
reads and writes inside the order database".

**There is no attribution.** The handler logs `log.info("재고 복원 완료. ord_no=%s", req.ord_no)`
— order number only, never a caller. And because no application logging configuration is ever
installed (`LOG_LEVEL` in `app/config.py` is read by nothing; there is no `basicConfig` or
`dictConfig` anywhere), even that line is unlikely to be emitted at all. The forensic position
after an incident is: no identity, and probably no record.

### The only control that might exist, and its exact weight

order-service's `InventoryClient` defaults to `http://inventory-api.internal`. A `.internal`
hostname is a naming convention that suggests a private network, and it is the single piece of
evidence in the entire estate pointing at a network boundary. It is not a control: no
NetworkPolicy, ingress, security group, mesh policy or firewall rule exists in any repository to
enforce it, and a name resolves for whoever is on the network that resolves it. Stated plainly —
**if a boundary protects this service, that boundary is undocumented, unversioned, and invisible
to every engineer reading the code.**

Two further observations sharpen the point rather than softening it. First, the caller does not
work: `InventoryClient.restore` posts to `/inventory/restore`, a path this service does not serve
under any router. So the network boundary, whatever it is, is currently the *only* thing standing
in front of an endpoint that has no legitimate traffic to blend into. Second, the estate has no
auth anywhere — order-service, the largest service and the one whose OpenAPI spec declares
`security: [{"bearerAuth": []}]` globally, has no Spring Security dependency to enforce it (see
[[PROC-DELIVERY-AUTH]]). The absence in inventory-api is therefore not an oversight in one
service; it is the estate's posture.

### Registry and README, claim versus reality

| Claim | Where | Reality |
|---|---|---|
| `db: MySQL (inventory)` | sellflow-docs:context/registry/services.yaml | `app/db.py` hardcodes `sellflow_order`; the inventory database is never connected to |
| `DB_URL=mysql://inventory:@localhost:3306/inventory` | inventory-api:.env.sample | `app/config.py` defines `DB_URL`, nothing imports `app/config.py`, and `app/db.py` builds its own DSN from `DB_HOST`/`DB_USER` |
| `sqlalchemy.url = mysql://inventory:@localhost:3306/inventory` | inventory-api:alembic.ini | same — three independent files describe a database the runtime does not use |
| `endpoints:` listed for order-service, absent for inventory-api | sellflow-docs:context/registry/services.yaml | the registry never states how any service is reached or protected, so it cannot be used to confirm or deny a gateway |
| 취소 시 재고 복원은 `inventory-api` 가 처리합니다 ("on cancellation, inventory-api handles stock restoration") | sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html | true as an intent; the path the order side calls does not exist, and the policy page states no access rule |

The registry credential story is worth its own line. All three configuration files agree on a DSN
of `mysql://inventory:@localhost:3306/inventory` — note the empty password between `:` and `@`.
The runtime ignores all three and connects as `sellflow`, also with no password. So the documented
credential and the actual credential differ in user and database, and agree only in having no
password.

## Worked Examples

**An unauthenticated read.** `GET /stock/SKU-1001` with no headers returns
`{"sku": ..., "available_qty": ..., "reserved_qty": ...}` if the row exists, or 404 `"SKU 없음"`
("SKU not found") if it does not. Nothing is checked at any stage (`app/main.py`).

**An unauthenticated write.** `POST /stock/restock` with body
`{"ord_no": "2026090100123", "reason_code": "01"}` and no headers increments `available_qty` for
whichever SKU that order's `ORDER_DTL` row names — across every warehouse row for that SKU, since
the `UPDATE` omits `warehouse_cd` although the primary key is `(sku, warehouse_cd)`
(`app/main.py`, `sql/V1__stock.sql`). The caller needs one valid order number and one of two
reason codes.

**What the known caller sends, and why it fails before auth is even reached.**

```http
POST /inventory/restore?ordNo=2026090100123&reason=01 HTTP/1.1
Host: inventory-api.internal
Content-Length: 0
```

No `Authorization`, no `HttpHeaders` object, no `RestTemplate` interceptor. The response is 404,
because `/inventory/restore` is served by neither the mounted app nor the unmounted router
(`order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java`,
`inventory-api:app/main.py`, `inventory-api:app/routers.py`).

## Related

- [[API-INVENTORY]] — the endpoints served without a credential
- [[CON-ORDER-INVENTORY]] — the caller contract, credential-free and path-mismatched
- [[PROC-INVENTORY-ERROR-HANDLING]] — what the same unauthenticated write does when it goes wrong
- [[PROC-INVENTORY-RESTOCK]] — what the unprotected write actually does
- [[RISK-INVENTORY]] — the absence of auth as a tracked finding among others
- [[SCH-INVENTORY]] — the tables reachable through these endpoints
- [[SYS-INVENTORY]] — the owning service and its shared-database posture
- [[SYS-ORDER]] — the one caller observed, and the order database this service writes into
- [[PROC-DELIVERY-AUTH]] — the same absence on the delivery side, showing this is an estate posture
