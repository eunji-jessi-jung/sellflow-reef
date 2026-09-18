---
id: "API-INVENTORY"
type: "api"
title: "Inventory API Surface"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Compiled on 2026-09-18 from the FastAPI decorators in app/main.py and app/routers.py, cross-checked against the tier-4 extracted spec in sources/apis/inventory/openapi.json and against order-service's InventoryClient. The mounted-versus-unmounted split was verified by grepping the whole repo for include_router (zero hits). Reconciled on 2026-09-18 against openapi.meta.json: the database-name contradiction, the unused app/models.py dataclasses and the per-warehouse behaviour of stock_item were folded in; the extraction contradicted nothing already written here. Deepened on 2026-09-19 to answer Q-031 field by field, to state the authentication posture explicitly and say how it was verified, to record the contract limits the framework does and does not enforce, and to add agent guidance. The deepening pass found one thing that changes the shape of the whole question: InventoryClient has no caller anywhere in order-service, so the mismatched request is never issued at all. Goes stale the moment app/main.py registers the router, the client acquires a caller, or the client changes its URL."
freshness_triggers:
  - "app/config.py"
  - "app/main.py"
  - "app/routers.py"
  - "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "tests/test_health.py"
  - "tests/test_restore.py"
known_unknowns:
  - "Whether a gateway, ingress rewrite or reverse proxy in front of the service maps /inventory/restore onto /stock/restock and translates query parameters into a JSON body. Nothing in this repository shows such a component, so the mismatch is reported as observed, not diagnosed. Note that the question is now largely academic: no code calls InventoryClient.restore, so nothing reaches such a proxy in the first place."
  - "Why app/routers.py was written and never mounted. Its docstring says restock relocation has been pending since 2023, but no ticket or commit rationale is in the repo."
  - "Whether any caller outside the five sampled repos — a batch job, a CS admin tool, a manual curl runbook — posts to /stock/restock. Verified absent within order-service, settlement-batch, inventory-api, delivery-bff and settlement-anomaly only."
  - "Whether stock restoration is instead performed by hand. The confluence page says 재고팀과 협의가 필요하다 (coordination with the inventory team is required) without naming a mechanism, and the reef holds no inventory-team runbook."
  - "Whether any liveness or readiness probe targets GET /inventory/health. If so it would be failing, since the route is not served."
  - "The service's actual base URL in each environment. Only order-service's default, http://inventory-api.internal, is visible."
  - "Whether OPTIONS/CORS, rate limiting or request size limits are applied anywhere."
tags:
  - inventory
  - fastapi
  - api
  - sellflow
aliases:
  - "inventory-api endpoints"
  - "재고 API"
relates_to:
  - type: "constrains"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[GLOSSARY-INVENTORY]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-AUTH]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "refines"
    target: "[[RISK-INVENTORY]]"
  - type: "depends_on"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:.env.sample"
    notes: "DB_URL points at an inventory database that the code never opens."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "Reads DB_URL; nothing consumes it."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "Hardcodes database sellflow_order."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "The two mounted routes."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
    notes: "Dataclasses that match no query and no route."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
    notes: "APIRouter defined but never included."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:requirements.txt"
    notes: "Three pins, none of them an auth or middleware library."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
    notes: "stock_item primary key is (sku, warehouse_cd)."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_health.py"
    notes: "Calls the health function directly, not over HTTP."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
    notes: "Asserts the app/config.py copy of RESTOCKABLE_REASONS, not the one app/main.py enforces."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "Relative to the order-service repo, not inventory-api. The would-be caller side; itself uncalled."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The cancellation path that would have invoked InventoryClient and does not."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/inventory/openapi.json"
    notes: "Reef-relative. Tier-4 extraction, including the x-unmounted block."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/inventory/openapi.meta.json"
    notes: "Reef-relative. Extraction record: 2 mounted endpoints, 2 unmounted, plus the uncertainty list."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "Reef-relative. §5 재고 복원."
notes: "The sources list is sorted by ref; inventory-api refs are relative to the inventory-api repo root, order-service refs to the order-service repo root."
---

# Inventory API Surface

## Overview

Two routers exist in this repository and only one of them is served. `app/main.py` attaches its
routes to the `FastAPI` instance with decorators, so `POST /stock/restock` and
`GET /stock/{sku}` are live. `app/routers.py` builds an `APIRouter(prefix="/inventory")` with
two more routes — but nothing ever calls `app.include_router`, so those paths return 404 in the
running service.

That alone would be a tidy piece of dead code. What makes the surface worth documenting is a
third path in the picture. order-service's `InventoryClient` posts to
`{base}/inventory/restore?ordNo=..&reason=..`. That path appears in neither router. And even if
the `/inventory` router were mounted, it would not match: the router defines only
`/inventory/stock/{sku_cd}` and `/inventory/health`, both `GET`. Meanwhile the endpoint that
does implement restocking, `POST /stock/restock`, expects a JSON body, not query parameters.
Three descriptions of one operation, and no two of them agree.

The 2026-09-19 deepening pass added a fourth observation that reframes the other three. The Java
client that sends the mismatched request has no caller. `InventoryClient.restore` is invoked by
nothing in order-service — `OrderCancelService.cancel`, the only cancellation path, saves an
`OrderCancel` row, flips the order status and publishes an event, and never touches inventory.
So the contract mismatch documented below is real, but it has never been exercised: the request
that would fail is not sent.

## Surface Profile

| Property | Value | Evidence |
|---|---|---|
| Style | REST-ish JSON over HTTP; no HATEOAS, no envelope | app/main.py |
| Framework | FastAPI 0.104.1 on uvicorn 0.24.0, Python 3.9 per README | requirements.txt, README.md |
| App identity | `FastAPI(title="inventory-api", version="3.2.1")` | app/main.py |
| Base URL | `http://inventory-api.internal` (the only one visible anywhere, and it is the caller's default, not a value this repo sets) | order-service `.../client/InventoryClient.java` |
| Port | 8000, fixed by the container command `uvicorn app.main:app --host 0.0.0.0 --port 8000` | inventory-api/Dockerfile |
| Mounted endpoints | 2 | app/main.py |
| Defined-but-unmounted endpoints | 2 | app/routers.py |
| Endpoints a caller expects and nobody serves | 1 | order-service `.../client/InventoryClient.java` |
| Authentication | None, on every endpoint — see below | app/main.py, app/routers.py, requirements.txt |
| Media type | `application/json` only, both directions | app/main.py |
| Field casing | snake_case on the live endpoints, camelCase on the unmounted stub | app/main.py, app/routers.py |
| Versioning | None in the path; the version lives only in the `FastAPI(...)` constructor | app/main.py |
| Published spec | None in the repo; FastAPI's auto-generated `/openapi.json` and `/docs` would be served at runtime and would describe the two mounted routes only | app/main.py |

### Authentication and Authorization Posture

**There is no authentication and no authorization on any endpoint of this service.** Not weak
auth, not auth deferred to a gateway that the repo names — none, and nothing in the repository
even alludes to a credential.

How that was verified, so the claim can be re-checked rather than taken on trust:

1. **Route decorators.** Both mounted routes are bare: `@app.post("/stock/restock")` and
   `@app.get("/stock/{sku}")`. Neither carries a `dependencies=[...]` argument, and neither
   handler signature contains a `Depends(...)` parameter — `restock(req: RestockRequest)` and
   `get_stock(sku: str)` are the complete signatures (app/main.py). The two unmounted routes in
   app/routers.py are equally bare, and `APIRouter(prefix="/inventory", tags=["inventory"])`
   is constructed without a `dependencies` argument (app/routers.py).
2. **Application construction.** `app = FastAPI(title="inventory-api", version="3.2.1")` takes no
   `dependencies` and no `middleware` argument, and no `app.add_middleware(...)`,
   `@app.middleware`, or exception-handler registration appears anywhere in the module
   (app/main.py).
3. **Dependencies.** `requirements.txt` pins exactly three packages — `fastapi==0.104.1`,
   `uvicorn==0.24.0`, `pymysql==1.1.0`. No `python-jose`, `passlib`, `authlib`, `pyjwt`, or
   `fastapi-users`; there is no library present that could verify a token even if code wanted to
   (requirements.txt).
4. **Configuration.** `app/config.py` defines three names — `DB_URL`, `RESTOCKABLE_REASONS`,
   `LOG_LEVEL` — and `.env.sample` defines two, `DB_URL` and `LOG_LEVEL`. No API key, no shared
   secret, no issuer, no audience, no allowlist (app/config.py, .env.sample).
5. **Caller side.** The would-be caller confirms the same posture from the other direction.
   `InventoryClient` constructs a plain `new RestTemplate()` with no interceptor and calls
   `postForEntity(url, null, Void.class)` — a `null` request entity, so not even a header map is
   attached, let alone an `Authorization` one (order-service `.../client/InventoryClient.java`).

The consequence is worth stating plainly for anyone reasoning about blast radius: `POST
/stock/restock` mutates `available_qty` on the shared `sellflow_order` database, unauthenticated,
for any caller that can reach the pod's network. The reasoning about network-level mitigations
belongs to [[PROC-INVENTORY-AUTH]]; this artifact records only what the API itself enforces,
which is nothing.

### Contract Limits

What the framework enforces, and what it silently does not:

| Limit | Enforced? | Detail |
|---|---|---|
| Request body schema | Yes | `RestockRequest(ord_no: str, reason_code: str)` — both required; a missing or non-string field yields FastAPI's standard 422 with a `detail` array (app/main.py) |
| `reason_code` value range | No | Typed `str`, not an enum. `"99"`, `""` and `"ABC"` all validate and simply fall through to `{"restocked": false, "reason": "not_restockable"}` (app/main.py) |
| `ord_no` format or length | No | Plain `str`. The order-side column is `VARCHAR(20)` (order-service `.../db/migration/V1__init.sql`), but nothing here checks it |
| `sku` path-parameter length | No | Typed `str`. The extracted spec annotates `maxLength: 30` to mirror the `VARCHAR(30)` column, but that constraint is the extractor's inference and is not in the code (sources/apis/inventory/openapi.json, sql/V1__stock.sql) |
| Extra body fields | Ignored | Pydantic v1/v2 default behaviour; no `extra="forbid"` is configured (app/main.py) |
| Idempotency | No | No key, no dedupe, no guard. Posting the same `ord_no` twice adds the quantity twice — see the worked example below |
| Rate limiting / request size / CORS | No | No middleware of any kind is registered (app/main.py) |
| Pagination | N/A | No collection endpoint exists |
| Deprecation signalling | No | No `deprecated=True`, no `Sunset` header, no version negotiation |
| Error body shape | Partly | `HTTPException(404, "...")` produces FastAPI's `{"detail": "주문 상세 없음"}`; the refusal case is *not* an error and returns 200 (app/main.py) |

Two of these matter more than the rest. First, refusal and success share HTTP 200, so status-code
handling cannot distinguish them. Second, there is no idempotency control on a write that adds to
a quantity, which makes retries unsafe by construction.

## Key Facts

- `app/main.py` mounts exactly two routes via decorators: `@app.post("/stock/restock")` and `@app.get("/stock/{sku}")` → app/main.py
- `app/routers.py` declares `router = APIRouter(prefix="/inventory", tags=["inventory"])` with `GET /inventory/stock/{sku_cd}` and `GET /inventory/health` → app/routers.py
- A repository-wide grep for `include_router` returns zero hits, so the `/inventory` router is never registered on the application → app/main.py, app/routers.py
- The extracted spec records the same split, listing the two `/inventory` paths under `x-unmounted` with the note that calling them returns 404 → sources/apis/inventory/openapi.json
- `GET /inventory/stock/{sku_cd}` is a stub regardless: it returns `{"skuCd": sku_cd, "qty": 0}` without touching the database → app/routers.py
- order-service calls `POST {baseUrl}/inventory/restore?ordNo=<ordNo>&reason=<sayuCd>` with a `null` body → order-service `src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- `/inventory/restore` is defined by neither `app/main.py` nor `app/routers.py` → app/main.py, app/routers.py
- The live restock endpoint takes a Pydantic body model `RestockRequest(ord_no: str, reason_code: str)`, not query parameters → app/main.py
- The caller's base URL defaults to `http://inventory-api.internal`, from `INVENTORY_BASE_URL` → order-service `src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- The caller discards the response: `restore` returns `void` and uses `Void.class` as the response type, so a 404 or 422 is never observed by order-service → order-service `src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- `POST /stock/restock` returns HTTP 200 in both outcomes, distinguishing them only by the body (`{"restocked": true}` versus `{"restocked": false, "reason": "not_restockable"}`) → app/main.py
- `POST /stock/restock` raises `HTTPException(404, "주문 상세 없음")` ("no order detail") when no `ORDER_DTL` row matches → app/main.py
- `GET /stock/{sku}` raises `HTTPException(404, "SKU 없음")` ("no such SKU") when no `stock_item` row matches → app/main.py
- No endpoint declares a security dependency, no `Depends(...)` appears in any handler signature, and `FastAPI(...)` is constructed with neither a `dependencies` nor a `middleware` argument; see [[PROC-INVENTORY-AUTH]] → app/main.py, app/routers.py
- `requirements.txt` pins only `fastapi==0.104.1`, `uvicorn==0.24.0` and `pymysql==1.1.0` — no JWT, OAuth, session or password library is installed, so no endpoint could verify a credential even if it tried → requirements.txt
- `InventoryClient.restore` has no caller: a repository-wide grep of order-service for `InventoryClient` and `inventoryClient` returns only the class declaration and the method definition in the client file itself → order-service `src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- The cancellation path that would be the natural caller does not use it: `OrderCancelService.cancel` loads `OrderMst`, saves an `OrderCancel`, calls `order.chwiso()`, publishes `publishOrderCancelled` and logs — with no inventory call and no inventory import in the file → order-service `src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`
- `RESTOCKABLE_REASONS` is defined twice with the same value in two modules, and the enforcing copy is the one in `app/main.py`; `app/config.py`'s copy, annotated "복원 대상 사유코드. 01 파트너귀책, 02 시스템오류." ("restock-eligible reason codes; 01 partner fault, 02 system error"), is imported by nothing → app/main.py, app/config.py
- The only restock test asserts the unenforced copy: `tests/test_restore.py` imports `from app.config import RESTOCKABLE_REASONS`, so the suite would stay green if `app/main.py`'s set changed → tests/test_restore.py, app/main.py
- The restock write has no idempotency control of any kind — no key, no dedupe, no state check — so two identical `POST /stock/restock` calls for one order add the quantity twice → app/main.py
- `reason_code` is typed `str` rather than an enum, so any value validates and unknown codes are answered with the same 200 refusal body as the known non-restockable ones → app/main.py
- Both mounted endpoints run against `sellflow_order`, the order service's database, with user `sellflow`: `app/db.py` hardcodes `"database": "sellflow_order"` and never reads `DB_URL`, while `app/config.py` and `.env.sample` define `DB_URL=mysql://inventory:@localhost:3306/inventory` → app/db.py, app/config.py, .env.sample, sources/apis/inventory/openapi.meta.json
- `stock_item`'s primary key is `(sku, warehouse_cd)`, but neither mounted endpoint mentions a warehouse: `GET /stock/{sku}` selects without one, so it answers with an arbitrary single warehouse row rather than a total, and the restock `UPDATE ... WHERE sku = %(sku)s` touches every warehouse row for that SKU → sql/V1__stock.sql, app/main.py, sources/apis/inventory/openapi.json
- `POST /stock/restock` restocks only for `RESTOCKABLE_REASONS = {"01", "02"}`, with the comment "03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다." ("03, customer change of mind, is often cancelled after delivery has started, so it is not a restock case") — and reason `04` is silently outside the set too → app/main.py
- `app/models.py` declares `Stock(sku_cd, qty, updated_at)` and `RestoreLog`, neither of which matches the live SQL columns (`sku`, `available_qty`, `reserved_qty`) and neither of which is imported by any route; its header claims "창고별 재고는 별도 시스템" ("per-warehouse stock is a separate system") although `stock_item` is keyed by warehouse → app/models.py, sources/apis/inventory/openapi.meta.json
- The extraction could not reach tier 2: `app.main` imports `app.db`, which imports `pymysql` at module scope, so importing the app to dump `app.openapi()` fails before any route is registered → sources/apis/inventory/openapi.meta.json
- The only health test imports the handler function directly (`from app.routers import health`) rather than issuing an HTTP request, so it passes even though the route is unreachable → tests/test_health.py
- The app is declared as `FastAPI(title="inventory-api", version="3.2.1")` → app/main.py

## Source of Truth

The authoritative description of this surface is the code, in this order of precedence:

1. `app/main.py` — the decorators on the `app` object. These, and only these, are served.
2. `app/routers.py` — a defined but unregistered `APIRouter`. Aspirational.
3. `sources/apis/inventory/openapi.json` — a tier-4 extraction (structured reading of the
   route decorators, no runtime import), which reproduces the split faithfully and marks the
   unmounted paths under `x-unmounted`. Its companion `openapi.meta.json` records the tiers
   attempted, the endpoint counts (2 mounted, 2 unmounted) and the open uncertainties, and it
   reaches the same three-way-mismatch conclusion as this artifact, flagged "as uncertain, not
   asserted" → sources/apis/inventory/openapi.meta.json

There is no committed OpenAPI document inside the repository, no `.http` collection, and no
contract test. FastAPI would serve a generated `/openapi.json` and `/docs` at runtime, but
those would describe the two mounted routes only.

The caller's expectation is a fourth, independent source: the hardcoded URL string in
order-service's `InventoryClient`. It is not derived from anything in this repository.

## Resource Map

### Mounted — served by the running app

| Method | Path | Handler | Request | Success | Errors |
|---|---|---|---|---|---|
| POST | `/stock/restock` | `restock` (app/main.py) | JSON body `RestockRequest{ord_no, reason_code}` | 200 `{"restocked": true}` or 200 `{"restocked": false, "reason": "not_restockable"}` | 404 `"주문 상세 없음"` (no order detail); 422 on body validation failure |
| GET | `/stock/{sku}` | `get_stock` (app/main.py) | Path param `sku` | 200 `{sku, available_qty, reserved_qty}` | 404 `"SKU 없음"` (no such SKU) |

### Defined but not mounted — 404 in the running app

| Method | Path | Handler | Behaviour if it were mounted |
|---|---|---|---|
| GET | `/inventory/stock/{sku_cd}` | `get_stock` (app/routers.py) | Returns the hardcoded stub `{"skuCd": sku_cd, "qty": 0}`; no database access |
| GET | `/inventory/health` | `health` (app/routers.py) | Returns `{"status": "ok"}` |

`app/routers.py` explains itself in a docstring: "재고 조회 라우터. 복원은 main.py 에 있다
(이관 예정, 2023부터)." — "Stock lookup router. Restore lives in main.py (migration planned,
since 2023)." The planned migration has not happened, and the router that was to receive it was
never wired up.

### Called by order-service — defined nowhere

| Method | Path | Caller | Status |
|---|---|---|---|
| POST | `/inventory/restore?ordNo=..&reason=..` | `InventoryClient.restore` (order-service) | Not defined in `app/main.py` or `app/routers.py` |

### The three-way mismatch

Lining the three descriptions up makes the divergence concrete:

| Aspect | What the caller sends | What the unmounted router offers | What the live endpoint expects |
|---|---|---|---|
| Path | `/inventory/restore` | `/inventory/stock/{sku_cd}`, `/inventory/health` | `/stock/restock` |
| Method | POST | GET | POST |
| Parameters | Query string: `ordNo`, `reason` | Path param `sku_cd` | JSON body: `ord_no`, `reason_code` |
| Naming style | camelCase (`ordNo`) | camelCase (`skuCd`) | snake_case (`ord_no`) |
| Mounted | — | No | Yes |

Three things fail to line up at once: the path exists in no router, the parameter style differs
from the live endpoint's body contract, and the router the path superficially resembles is not
even registered. On top of that, `InventoryClient.restore` returns `void` and passes
`Void.class` to `postForEntity`, so order-service never inspects the outcome — a 404 from the
inventory side would leave no trace on the caller. The cancellation policy page, for its part,
simply states that "취소 시 재고 복원은 inventory-api 가 처리합니다" ("stock restoration on
cancellation is handled by inventory-api") and refers the reader to the 재고팀 for the details
(sources/raw/confluence-snapshots/주문-취소-정책_48213.html). The cross-system consequences
belong to [[CON-ORDER-INVENTORY]] and [[SYS-ORDER]].

Whether some proxy rewrites the path and the parameters is not answerable from this repository;
it is recorded in `known_unknowns` rather than asserted either way.

### Q-031 — does the restock endpoint match what InventoryClient calls?

**Verdict: no. They do not match on a single dimension — not the path, not the HTTP method's
target, not the parameter location, not the parameter names, not the field-name casing. And the
mismatch is moot in practice, because `InventoryClient.restore` has no caller, so the request is
never issued.**

The comparison, field by field. The caller's single statement is:

```java
restTemplate.postForEntity(baseUrl + "/inventory/restore?ordNo=" + ordNo
        + "&reason=" + sayuCd, null, Void.class);
```

and the live endpoint is `@app.post("/stock/restock")` taking `RestockRequest(ord_no, reason_code)`.

| Dimension | `InventoryClient.restore` sends | `POST /stock/restock` expects | Match? |
|---|---|---|---|
| Path | `/inventory/restore` | `/stock/restock` | **No.** Different prefix *and* different resource name. `/inventory/restore` is defined by neither app/main.py nor app/routers.py, so it is a 404 in the running app |
| Method | POST | POST | Yes — the only agreement |
| Parameter transport | Query string, appended by string concatenation | JSON request body, parsed by Pydantic | **No** |
| Order identifier — name | `ordNo` | `ord_no` | **No.** camelCase versus snake_case |
| Order identifier — value | The order number, unencoded (no `URLEncoder`, no `UriComponentsBuilder`) | `str`, required | Value compatible; transport and name are not |
| Reason — name | `reason` | `reason_code` | **No.** Different word, not merely different casing |
| Reason — value | `sayuCd`, i.e. a `CancelReason` code `01`–`04` | `reason_code`, of which only `01` and `02` restock | Value domain compatible; name is not |
| Request body | Literal `null` — `postForEntity(url, null, Void.class)` | Required; a missing body is a 422 | **No.** Even if the path matched, FastAPI would reject the call |
| Content-Type | None set; `null` body means Spring sends no entity | `application/json` required for body parsing | **No** |
| Response type expected | `Void.class`, discarded | `{"restocked": bool, ...}` | **No**, and the caller could not observe the difference |
| Auth | None attached | None required | Yes, trivially |

Run the two side by side and every substantive dimension diverges. Had the request been issued
against the live service it would have failed at the very first hop with a 404 on
`/inventory/restore`; had the path been corrected to `/stock/restock` it would then have failed
with a 422 on the empty body; had the body been supplied as query parameters it would have failed
again, because `ord_no`/`reason_code` are read from the body and `ordNo`/`reason` are not read at
all. Three independent defects stacked on one call.

**Why it has never been noticed.** Two silences compound. `restore` returns `void` and passes
`Void.class` to `postForEntity`, so no status code, body or exception path is inspected on the
caller side — a 404 would leave no trace. And, as of this pass, there is no caller to leave a
trace for: grepping order-service for `InventoryClient` and `inventoryClient` returns only the
class declaration and the method definition. `OrderCancelService.cancel` — the one cancellation
path, and the obvious place for a restock trigger — imports `OrderCancelRepository`,
`OrderMstRepository` and `OrderEventPublisher` and no client at all. The restock feature exists
as three fragments that have never been connected: an endpoint with no caller, a client with no
caller, and a URL that names neither of them.

**What this does not tell us.** It does not tell us that stock is never restored. The business
rules mark 재고 복원 O (stock restoration: yes) for codes 01, 02 and 04
(sources/context/business-rules.md), and the cancellation policy page says only that
"취소 시 재고 복원은 inventory-api 가 처리합니다" ("stock restoration on cancellation is handled
by inventory-api") and that "재고팀과 협의가 필요하다" ("coordination with the inventory team is
required") — no mechanism (sources/raw/confluence-snapshots/주문-취소-정책_48213.html). Whether
some out-of-repo job or a manual process posts to `/stock/restock` is recorded in
`known_unknowns`; the search was performed across the five repos in `repos/` and found nothing.
The cross-system framing is [[CON-ORDER-INVENTORY]], and the defect framing is
[[RISK-INVENTORY]].

### Worked Example — the live restock call

Restocking order `2026090100123`, cancelled with reason `01` (파트너 귀책 / partner fault):

```http
POST /stock/restock HTTP/1.1
Host: inventory-api.internal
Content-Type: application/json

{"ord_no": "2026090100123", "reason_code": "01"}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"restocked": true}
```

The same request with `reason_code` `03` (고객 변심 / customer change of mind) or `04`
(배송 실패 / delivery failure) is also a 200, with a different body and no database write:

```json
{"restocked": false, "reason": "not_restockable"}
```

For reason `04` that outcome contradicts the business-rule spreadsheet, which marks 재고 복원 O
for that code. The conflict is documented in [[PROC-INVENTORY-RESTOCK]].

Note that both outcomes share status 200, so a caller cannot tell success from refusal without
parsing the body — and order-service, discarding the response entirely, does not.

**Unknown order number.** The `SELECT ... FROM ORDER_DTL` finds nothing and the handler raises:

```http
POST /stock/restock HTTP/1.1
Host: inventory-api.internal
Content-Type: application/json

{"ord_no": "9999999999999", "reason_code": "01"}
```

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{"detail": "주문 상세 없음"}
```

`주문 상세 없음` means "no order detail". Note that this 404 is reachable only for restockable
reason codes: the `RESTOCKABLE_REASONS` check runs *before* the lookup, so a nonexistent order
cancelled with reason `03` returns 200 `{"restocked": false, "reason": "not_restockable"}` and
never reveals that the order does not exist (app/main.py).

**Malformed body.** Pydantic rejects it before the handler runs, with FastAPI's standard shape:

```http
POST /stock/restock HTTP/1.1
Content-Type: application/json

{"ord_no": "2026090100123"}
```

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{"detail": [{"loc": ["body", "reason_code"], "msg": "field required", "type": "value_error.missing"}]}
```

**The same call twice.** There is no idempotency key, no dedupe table and no state check, so a
retried restock is applied again:

```json
{"restocked": true}
```
```json
{"restocked": true}
```

Both are 200, both ran `UPDATE stock_item SET available_qty = available_qty + 2 ...`, and
`available_qty` has now moved by 4 for a 2-unit cancellation. Combined with the missing
`warehouse_cd` predicate described in [[SCH-INVENTORY]], a single retry against a two-warehouse
SKU overstates inventory by 4× the cancelled quantity.

**The stock read.** `GET /stock/SKU-1001`:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"sku": "SKU-1001", "available_qty": 42, "reserved_qty": 3}
```

No `warehouse_cd` field appears in the response even though it is half the primary key, so the
answer cannot be attributed to a warehouse (app/main.py, sql/V1__stock.sql).

## How Agents Should Use This

**Answering questions.**

- *"How does a cancelled order get its stock back?"* — On the evidence in these five repos, it
  does not, automatically. Say so, and cite the three fragments: the endpoint
  (`POST /stock/restock`, app/main.py), the client that targets a different URL
  (`InventoryClient.restore`), and the absence of any caller for that client
  (`OrderCancelService.cancel`). Do not present the confluence page's "inventory-api 가
  처리합니다" as a description of running behaviour; it is a 2021 page, last modified
  2021-03-17, and a 2024 comment on it already asks "이 문서 아직 유효한가요?" ("is this document
  still valid?").
- *"Which reason codes restock?"* — `01` and `02`, from `RESTOCKABLE_REASONS` in **app/main.py**.
  Read that copy, not `app/config.py`'s, and not `tests/test_restore.py`'s subject. Note the
  conflict with the business-rules sheet, which marks 재고 복원 O for `04` as well
  (sources/context/business-rules.md) — that conflict is owned by [[PROC-INVENTORY-RESTOCK]].
- *"Is this API secured?"* — No. Reproduce the five-step verification above rather than asserting
  it; the value of the claim is in how it was checked.

**Before changing anything.**

- Treat `app/routers.py` as aspirational. Editing it changes nothing at runtime until someone
  calls `app.include_router(router)`, which no file does.
- Treat `app/models.py` as documentation of intent, not of the schema. Its field names
  (`sku_cd`, `qty`) do not exist as columns.
- Wiring up the restock path is not a one-line fix. At minimum it needs a caller, an agreed path,
  a body rather than query parameters, a `warehouse_cd` predicate on the UPDATE, a loop over
  `ORDER_DTL` lines instead of `fetch_one`, and an idempotency guard — and the reason-code set
  has to be reconciled with the business-rules sheet first, which is a policy question, not a
  code question.

**Traps to avoid.**

- Do not infer success from HTTP 200. The refusal path is also 200.
- Do not quote the extracted spec's `maxLength: 30` on `sku` as an enforced constraint; it is the
  extractor's inference from the column type (sources/apis/inventory/openapi.json).
- Do not describe this service as owning an inventory database. It connects to `sellflow_order`
  (app/db.py), and `DB_URL` — which names an `inventory` database — is read by `app/config.py`
  and used by nothing.
- Do not assume the two `/inventory/...` routes are reachable. They 404.

## Related

- [[CON-ORDER-INVENTORY]] — the cross-system boundary this mismatch sits on
- [[GLOSSARY-INVENTORY]] — `sku`, `ord_no`, `reason_code` and their order-domain names
- [[PROC-INVENTORY-AUTH]] — why none of these endpoints requires a credential
- [[PROC-INVENTORY-RESTOCK]] — the logic behind `POST /stock/restock`
- [[RISK-INVENTORY]] — the unmounted router and the silent caller as defects
- [[SCH-INVENTORY]] — the tables these endpoints read and write
- [[SYS-INVENTORY]] — the owning service
- [[SYS-ORDER]] — the calling system
