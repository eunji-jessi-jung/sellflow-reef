---
id: "API-SETTLEMENT-ANOMALY"
type: "api"
title: "Settlement Anomaly Detection API"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened 2026-09-19 by re-reading every file in settlement-anomaly (app/main.py, app/db.py, app/schemas.py, model/detector.py, model/features.py, requirements.txt, sql/V1__anomaly_schema.sql, .env.template, README.md, tests/test_detector.py) against sources/apis/settlement/anomaly/openapi.json, its openapi.meta.json and sources/infra/settlement/runtime.md. This pass added an explicit surface profile and an auth-posture statement that names every mechanism checked for and not found; a contract-limits section covering idempotency, the DECIMAL(5,4) score ceiling, the 422-only error vocabulary and the absence of pagination or a batch form; the MODEL_PATH environment variable that main.py ignores; and a How Agents Should Use This section. The no-caller finding was re-verified by grep across all five repos and still holds. Stale if a route is added, if authentication appears, or if a caller for POST /detect shows up in any repo."
freshness_triggers:
  - ".env.template"
  - "app/db.py"
  - "app/main.py"
  - "app/schemas.py"
  - "model/detector.py"
  - "requirements.txt"
known_unknowns:
  - "Who calls POST /detect in production. A grep for '/detect', 'settlement-anomaly', '8090' and 'anomaly' across all five repos, re-run 2026-09-19, returns hits only inside settlement-anomaly itself."
  - "Whether an external scheduler (cron, Airflow, Kubernetes CronJob) exists outside the repos. No infrastructure manifests, no Dockerfile and no CI workflow exist for this service."
  - "Whether a network policy or gateway fronts the service and supplies the authentication the application does not implement."
  - "The real base URL. The extracted spec records http://settlement-anomaly.internal:8090 derived from the README's uvicorn command, not from a deployment manifest."
  - "What /detect returns when the model file is missing. AnomalyDetector.load runs at import scope, so the process would fail to start rather than return an error."
  - "Whether anything outside this repository reconciles the row-key case mismatch — a cursor class other than DictCursor, or SQL column aliases applied elsewhere. Read as committed, detect() raises KeyError on the first row, which is why the extraction flags this rather than asserting it."
  - "Whether POST /detect has ever run to completion in production. The key mismatch and the absent model artefact both point the other way, but no log, run record or incident in the reef either confirms or refutes it."
  - "Which Python version runs it. README says 3.9, the service registry says 3.11, and requirements.txt pins no interpreter."
  - "Whether MODEL_PATH is honoured by a deployed variant of main.py. The committed main.py hardcodes model/artifacts/iforest_v3.pkl and never reads the variable, while .env.template sets it to a different path, ./model/detector.pkl."
tags:
  - settlement
  - fastapi
  - anomaly-detection
  - api
aliases:
  - "POST /detect"
  - "이상 탐지 API"
relates_to:
  - type: "refines"
    target: "[[API-SETTLEMENT-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/settlement/anomaly/openapi.json"
    notes: "Tier-4 extraction of the two routes, plus the x-anomaly-codes block"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/settlement/anomaly/openapi.meta.json"
    notes: "Extraction record: 2 endpoints, plus the row-key, feature-set and schema uncertainties"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Owner 데이터팀, runtime Python 3.11, and '조치 주체는 정의되어 있지 않음. TODO'"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Files POST /detect under 'Unscheduled / externally triggered'"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "No CI workflow, no Dockerfile, and the env vars the code does and does not read"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:.env.template"
    notes: "DB_URL points at a settlement database and MODEL_PATH at a path main.py never reads"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:README.md"
    notes: "Claims a daily 03:00 trigger after the settlement batch, and four anomaly codes"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
    notes: "DictCursor and the hardcoded sellflow_order database"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "Both routes, the DetectRequest model, the three-table join and the INSERT"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/schemas.py"
    notes: "AnomalyRow, unused and mismatched against SETTLEMENT_ANOMALY"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/features.py"
    notes: "Four declared features against the two the detector scores"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:requirements.txt"
    notes: "Five pins, none of them a scheduler or an HTTP client"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
    notes: "SCORE DECIMAL(5,4) and the absence of any uniqueness constraint"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:tests/test_detector.py"
    notes: "The repo's only test — asserts a string is in a list constant, exercises no route"
notes: ""
---

# Settlement Anomaly Detection API

## Overview

Two endpoints, both defined in `app/main.py`: `POST /detect` runs detection for one settlement date and persists the hits, and `GET /health` reports liveness plus the loaded model version. Neither carries any authentication.

The more interesting property of this API is who uses it. The README states the service is triggered daily at 03:00 after the settlement batch, but no scheduler and no caller exists in any of the five repos.

## Surface Profile

| Property | Value | Source |
|---|---|---|
| Framework | FastAPI 0.104.1 on uvicorn 0.24.0, Pydantic 2.5.2 | requirements.txt |
| App title / version | `FastAPI(title="settlement-anomaly", version="1.4.0")` | app/main.py |
| Routes | 2 — `POST /detect`, `GET /health` | app/main.py, sources/apis/settlement/anomaly/openapi.json |
| Base URL | `http://settlement-anomaly.internal:8090` (inferred from the README's uvicorn command, not from a manifest) | README.md, sources/apis/settlement/anomaly/openapi.json |
| Media type | `application/json` in and out, on both routes | app/main.py |
| Auth | **none of any kind** — see the posture section below | app/main.py |
| Versioning | none — no path prefix, no `Accept` version, no header. The `1.4.0` appears only in the OpenAPI `info` block | app/main.py |
| Declared error responses | `422` only, and that one supplied by FastAPI rather than by the handler | sources/apis/settlement/anomaly/openapi.json, app/main.py |
| Side effects | `POST /detect` writes rows to `SETTLEMENT_ANOMALY` in the production `sellflow_order` schema | app/main.py, app/db.py |
| Deploy | no CI workflow, no Dockerfile in the repo | sources/infra/settlement/runtime.md |

## Key Facts

- The FastAPI app is created as `FastAPI(title="settlement-anomaly", version="1.4.0")` → app/main.py
- Exactly two routes are declared: `@app.post("/detect")` and `@app.get("/health")` → app/main.py
- `DetectRequest` has a single field, `jungsan_ilja: date` → app/main.py
- The response of `/detect` is `{"detected": <int>, "jungsan_ilja": <date>}` → app/main.py
- `/health` returns `{"status": "ok", "model": detector.version}`, where the version comes from the pickled model payload → app/main.py, model/detector.py
- Neither endpoint declares a dependency, security scheme, API key check or auth middleware; there is no `Depends`, no `HTTPBearer`, no `OAuth2`, no `APIKeyHeader`, no `add_middleware` call and no `dependencies=` argument to `FastAPI(...)` in the repo (verified by reading app/main.py in full on 2026-09-19) → app/main.py
- No caller for `/detect` exists in any of the five repos — a grep for `/detect`, `settlement-anomaly`, `anomaly` and `8090` across all repos, re-run 2026-09-19, returns five hits and all five are inside settlement-anomaly itself: its README title, its README uvicorn line, its `.env.template` MODEL_PATH line, its `FastAPI(title=...)` and its own `@app.post("/detect")` decorator → verified by grep, 2026-09-19
- The README nonetheless asserts an operational trigger: "정산 배치 종료 후 (매일 03:00) 트리거된다." — "it is triggered after the settlement batch finishes (daily at 03:00)" → README.md
- `settlement-batch` cannot be the caller: it has no HTTP client dependency and its `build.gradle` does not include a web or REST client starter → build.gradle (settlement-batch), see [[API-SETTLEMENT-BATCH]]
- The model is loaded at module import scope — `detector = AnomalyDetector.load("model/artifacts/iforest_v3.pkl")` sits at module level, above the route definitions — so a missing artefact prevents the process from starting rather than failing a request. The extraction confirms the artefact is absent from the repo, which is why tier-2 runtime extraction was skipped → app/main.py, sources/apis/settlement/anomaly/openapi.meta.json
- `.env.template` declares `MODEL_PATH=./model/detector.pkl`, which is both unread by the code and a different path from the hardcoded one → .env.template, app/main.py, sources/infra/settlement/runtime.md
- The service reaches the database directly rather than through any settlement API, joining three tables in one query → app/main.py, app/db.py
- The row keys do not line up: `app/db.py` builds its connection with `pymysql.cursors.DictCursor`, so every row comes back keyed by the SQL column labels in upper case (`RUN_ID`, `ORD_NO`, `SANGTAE_CD`, `JUNGSAN_AMT`, `SUSURYO`), while `model/detector.py` reads `r["sangtae_cd"]`, `r["ord_no"]`, `row["jungsan_amt"]` and `row["susuryo"]` in lower case. As committed, `detect()` raises `KeyError` on the first row → app/db.py, model/detector.py, sources/apis/settlement/anomaly/openapi.meta.json
- The README documents four anomaly codes — `AMT_OUTLIER`, `DUP_SETTLE`, `CANCELLED_SETTLED`, `FEE_MISMATCH` — but `model/detector.py` emits only two, `CANCELLED_SETTLED` and `AMT_OUTLIER`; `DUP_SETTLE` (중복 정산 / duplicate settlement) and `FEE_MISMATCH` (수수료 불일치 / fee rate mismatch) are implemented nowhere → README.md, model/detector.py, sources/apis/settlement/anomaly/openapi.json
- The declared feature set and the scoring call disagree: `model/features.py` lists `amount_zscore`, `partner_daily_count`, `cancel_after_settle_flag` and `fee_rate_delta`, while `_amount_score` feeds the model only `[jungsan_amt, susuryo]`. The same file records "2024-11 이후 재학습 이력 없음" ("no retraining since 2024-11") and a TODO from 2025-02-10: "cancel_after_settle_flag 가 사실상 단독으로 결과를 좌우한다. 가중치 재조정 필요." ("cancel_after_settle_flag effectively decides the outcome on its own; the weights need rebalancing") → model/features.py, model/detector.py
- The repo's only test asserts nothing about the API or the detector's behaviour — `test_cancel_flag_is_a_feature` checks that a string is present in a list constant → tests/test_detector.py
- `app/schemas.py` declares an `AnomalyRow` (`run_id`, `ord_no`, `anomaly_type`, `score`, `status`) that no route imports and that does not match the table: `SETTLEMENT_ANOMALY` has `ANOMALY_CD`, not `anomaly_type`, and no `RUN_ID` column at all → app/schemas.py, sources/apis/settlement/anomaly/openapi.meta.json
- `.env.template` points `DB_URL` at a settlement database, but `app/db.py` hardcodes `database="sellflow_order"` and never reads `DB_URL`, so detection runs against the shared order instance → .env.template, app/db.py
- `/detect` is not idempotent: calling it twice for the same `jungsan_ilja` inserts a second set of `SETTLEMENT_ANOMALY` rows, since the INSERT has no uniqueness guard and the table has no unique constraint → app/main.py, sql/V1__anomaly_schema.sql
- Every write is its own connection and its own transaction: `app/db.py`'s `execute` opens a `pymysql.connect`, executes and commits per call, and `/detect` calls it once per detected anomaly — so a failure midway through leaves a partial detection set committed, with no run marker to identify it → app/db.py, app/main.py

## Source of Truth

`app/main.py` is authoritative. The extracted spec at `sources/apis/settlement/anomaly/openapi.json` was produced by reading the FastAPI decorators rather than by importing the app — the extraction record notes that importing would fail with `FileNotFoundError` because `model/artifacts/iforest_v3.pkl` is missing from the repo → sources/apis/settlement/anomaly/openapi.meta.json. That record, `openapi.meta.json`, is the companion to read alongside the spec: it counts the two endpoints and lists the uncertainties the spec itself cannot express.

Base URL in the extracted spec is `http://settlement-anomaly.internal:8090`, inferred from the README's `uvicorn app.main:app --port 8090`. That inference has not been confirmed against a deployment manifest; there is no Dockerfile, no CI workflow and no Kubernetes manifest for this service anywhere in the material available → sources/infra/settlement/runtime.md

## Resource Map

### anomaly

| Method | Path | Purpose | Auth | Request | Response |
|---|---|---|---|---|---|
| POST | `/detect` | Detect anomalies for one settlement date and insert them into `SETTLEMENT_ANOMALY` | none | `DetectRequest { jungsan_ilja: date }` | `{ detected: int, jungsan_ilja: date }` |
| GET | `/health` | Liveness plus loaded model version | none | — | `{ status: "ok", model: string }` |

FastAPI returns `422` for a request body that fails validation — for example a `jungsan_ilja` that is not a parseable date. No other error responses are declared, and the handler has no try/except, so a database error or the detector's key mismatch would surface as an unhandled `500` → app/main.py

### Worked Example

Request:

```http
POST /detect HTTP/1.1
Host: settlement-anomaly.internal:8090
Content-Type: application/json

{"jungsan_ilja": "2026-08-17"}
```

What the handler does, in order → app/main.py:

1. Runs the query

```sql
SELECT d.RUN_ID, d.ORD_NO, d.PARTNER_ID, d.JUNGSAN_AMT, d.SUSURYO, m.SANGTAE_CD
  FROM SETTLEMENT_DTL d
  JOIN SETTLEMENT_RUN r ON r.RUN_ID = d.RUN_ID
  JOIN ORDER_MST m ON m.ORD_NO = d.ORD_NO
 WHERE r.JUNGSAN_ILJA = '2026-08-17'
```

A `DictCursor` returns rows shaped like this — note the upper-case keys, which is where the defect below begins → app/db.py:

```python
{"RUN_ID": 8801, "ORD_NO": "ORD20260817001", "PARTNER_ID": "P00132",
 "JUNGSAN_AMT": Decimal("40040"), "SUSURYO": Decimal("5460"), "SANGTAE_CD": "CHWISO"}
```

2. Passes the rows to `AnomalyDetector.detect`, which is written to flag cancelled-state orders as `CANCELLED_SETTLED` (score 1.0) and otherwise to score the amount, flagging above 0.85 as `AMT_OUTLIER` → model/detector.py. Read literally, the step does not get that far: the rows arrive keyed `SANGTAE_CD`, and the detector asks for `r["sangtae_cd"]`, so the first row raises `KeyError` → app/db.py, model/detector.py
3. Inserts one `SETTLEMENT_ANOMALY` row per hit with `STATUS='DETECTED'`
4. Logs "이상 탐지 완료. 기준일=%s, 탐지 %d건" — "detection complete. base date=%s, %d detected"

Response, as the handler is written to answer:

```json
{"detected": 7, "jungsan_ilja": "2026-08-17"}
```

Whether any caller has ever seen such a body is a separate question. With the row-key mismatch in place and no `try`/`except` in the handler, a request against a non-empty result set ends as an unhandled `KeyError` — a 500 whose body is FastAPI's generic `{"detail": "Internal Server Error"}`, with the traceback only in the server log → app/main.py, app/db.py, model/detector.py

Note also what the success body does **not** carry: no anomaly ids, no order numbers, no breakdown by code, and no run identifier. A caller that wanted to know *which* orders were flagged would have to query `SETTLEMENT_ANOMALY` itself — and no endpoint exists to read that table → app/main.py, see [[SCH-SETTLEMENT-ANOMALY]]

Health check:

```http
GET /health HTTP/1.1
```
```json
{"status": "ok", "model": "iforest_v3"}
```

The `model` value is whatever string sits under `payload["version"]` in the pickle; `iforest_v3` here is inferred from the artefact filename, not observed. And because the model is loaded at import, `/health` can only ever answer when the artefact loaded successfully — it cannot report a model problem, only fail to exist → app/main.py, model/detector.py

### Authentication and authorisation posture

**There is none, on either route.** This is worth stating explicitly rather than by omission, because `POST /detect` is an unauthenticated write against the production `sellflow_order` schema.

Every mechanism checked for, on 2026-09-19, by reading `app/main.py` in full:

| Mechanism | Present? |
|---|---|
| `Depends(...)` on either route | no |
| `dependencies=[...]` on `FastAPI(...)` | no |
| `HTTPBearer`, `HTTPBasic`, `OAuth2PasswordBearer`, `APIKeyHeader` | no — `fastapi.security` is never imported |
| `app.add_middleware(...)` of any kind | no |
| `@app.middleware("http")` | no |
| A token, key or secret read from the environment | no — the only env vars any code reads are `DB_HOST` and `DB_USER` |
| CORS configuration | no |

→ app/main.py, app/db.py, sources/infra/settlement/runtime.md

So the gate on `POST /detect` is, in the code, nothing at all. Anything that can reach port 8090 can cause detection rows to be written. Whether the port is reachable beyond the cluster depends on a network policy or gateway that is not in any repo, and that question stays in `known_unknowns` — the artifact records the application-level posture, which is open, and declines to guess at the infrastructure-level one. The reef's four `proc-*-auth` artifacts record the same pattern across all five services: no service authenticates anything in code.

### Contract limits

What a caller can and cannot rely on, all read off the code rather than a spec:

| Limit | Value | Consequence | Source |
|---|---|---|---|
| Request surface | one field, `jungsan_ilja: date` | No partner filter, no run filter, no dry-run flag, no limit. Detection is all-or-nothing for a date | app/main.py |
| Date semantics | matched against `SETTLEMENT_RUN.JUNGSAN_ILJA`, not against `DETECTED_DTM` | A date with several runs (as on 2025-07-12) processes all of them together | app/main.py, see [[SCH-SETTLEMENT-BATCH]] |
| Idempotency | none | Re-posting the same date duplicates every anomaly row; no unique key exists to stop it | app/main.py, sql/V1__anomaly_schema.sql |
| Transactionality | one commit per inserted row | A mid-run failure leaves a partial, unmarked detection set | app/db.py |
| Score range | `SCORE DECIMAL(5,4)`, i.e. ±9.9999 | `CANCELLED_SETTLED` writes exactly 1.0 and is safe; `AMT_OUTLIER` writes an unbounded negated IsolationForest score and may be truncated silently | sql/V1__anomaly_schema.sql, model/detector.py |
| Anomaly vocabulary | 2 producible codes of the 4 documented | A caller relying on `DUP_SETTLE` or `FEE_MISMATCH` will wait forever | model/detector.py, README.md |
| Error vocabulary | `422` from FastAPI validation; everything else is an unhandled `500` | No structured error body, no error code, nothing distinguishing "no rows for that date" (a `200` with `detected: 0`) from a real failure | app/main.py |
| Response payload | counts only | The flagged orders are not in the response and no read endpoint exists | app/main.py |
| Pagination / batching | none | No way to detect a date range in one call, and no bound on how many rows the join returns | app/main.py |
| Timeout / retry guidance | none declared | The join and the per-row inserts are synchronous inside the request; a large date would hold the request open for their duration | app/main.py, app/db.py |

## The missing caller

The README's claim of a 03:00 trigger is worth stating precisely, because four separate checks failed to find it:

- No scheduler library in `requirements.txt` — fastapi, uvicorn, scikit-learn, pymysql, pydantic only; and no HTTP client either, so this service calls nothing and nothing in it schedules anything → requirements.txt
- No scheduling code in the repo; the only Quartz configuration in the workspace is in settlement-batch, and it registers two triggers, neither of which makes an HTTP call → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java (settlement-batch)
- No mention of the service, its port or its path in any of the other four repos (grep, re-run 2026-09-19 — zero hits outside settlement-anomaly)
- No deployment artefact that could carry a `CronJob` or a sidecar: this service has no CI workflow and no Dockerfile at all, unlike settlement-batch which at least has three `workflow_dispatch` workflows → sources/infra/settlement/runtime.md

The infra extraction reaches the same conclusion independently and files `POST /detect` under "Unscheduled / externally triggered" → sources/infra/settlement/queues.md.

So either an external scheduler outside the repos calls it, or detection does not run at all. Nothing in the available material distinguishes the two cases. This matters for [[RISK-SETTLEMENT-RECON-BACKLOG]]: the `CANCELLED_SETTLED` rule would independently surface the backlog population, but only if something invokes it — and the registry records that even if it did, "조치 주체는 정의되어 있지 않음. TODO" ("the party responsible for acting is not defined — TODO") → sources/context/registry/services.yaml

## How Agents Should Use This

**Treat `POST /detect` as a write, not a query.** Its name suggests analysis; its effect is inserting rows into a shared production schema. There is nothing to authenticate against it and nothing to undo it, so never suggest "just call `/detect` to check" as a diagnostic step. Reading the results is a `SELECT` against `SETTLEMENT_ANOMALY` — there is no endpoint for it.

**Do not describe this API as scheduled.** The correct phrasing is: the README claims a daily 03:00 trigger, and no caller or scheduler exists in any of the five repos. State the claim and the absence together; asserting either half alone misleads.

**Before citing an anomaly code, check it is producible.** Only `CANCELLED_SETTLED` and `AMT_OUTLIER` can ever be written. `DUP_SETTLE` and `FEE_MISMATCH` exist in the README and in the extracted spec's `x-anomaly-codes` block, and nowhere in `model/detector.py`. This is the most likely thing to get wrong when answering "what anomalies does settlement detect?"

**When asked whether detection works, distinguish three layers.** (a) The routes exist and are shaped as documented — verified from code. (b) The process would fail to start without `model/artifacts/iforest_v3.pkl`, which is absent from the repo — verified, and the reason tier-2 extraction was skipped. (c) Even if it started, `detect()` reads lower-case keys from an upper-case `DictCursor` row and would raise `KeyError` — observed in code, not verified at runtime, and recorded as an observation rather than an assertion because a deployed `db.py` or a wrapper outside the repo could normalise the keys.

**Do not use `app/schemas.py` as the response contract.** `AnomalyRow` is imported by nothing and matches neither the table nor the INSERT. The real contract is the two inline dict literals in `app/main.py`.

**If asked to add authentication, note what else it touches.** This service shares a database connection style, an unauthenticated posture and a hardcoded `sellflow_order` with `inventory-api`; a change here is a pattern decision across the estate, not a local one → app/db.py, sources/infra/settlement/runtime.md

## Related

- [[SYS-SETTLEMENT-ANOMALY]] — the service behind these endpoints
- [[SCH-SETTLEMENT-ANOMALY]] — the table `/detect` writes
- [[SCH-SETTLEMENT-BATCH]] — the tables `/detect` reads
- [[API-SETTLEMENT-BATCH]] — the batch service's non-HTTP surface, which cannot be the caller
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what this detector would see if it ran
