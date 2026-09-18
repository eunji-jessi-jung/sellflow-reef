---
id: "SYS-SETTLEMENT-ANOMALY"
type: "system"
title: "Settlement Anomaly Detection Service"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Re-read line-by-line against the whole settlement-anomaly repo (10 files) on 2026-09-19, plus settlement-batch's QuartzConfig and OrderEventRelayJob, the extracted openapi.json / schema.md / runtime.md, services.yaml and the org chart. Goes stale the moment model/detector.py gains the two missing rule types, a scheduler or caller for POST /detect appears anywhere, the pickle under model/artifacts/ is replaced, or anyone writes REVIEWED_BY."
freshness_triggers:
  - ".env.template"
  - "README.md"
  - "app/db.py"
  - "app/main.py"
  - "model/detector.py"
  - "model/features.py"
  - "requirements.txt"
  - "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "sql/V1__anomaly_schema.sql"
known_unknowns:
  - "Who or what calls POST /detect (Q-027, and the residue of Q-024). The README claims a 03:00 daily trigger after the settlement batch; settlement-batch's QuartzConfig registers only dailySettlementQuartzJob (02:00 KST) and orderEventRelayJob (every 10 min), and a grep for 'anomal' and for '8090' across all five repos returns hits only inside settlement-anomaly itself. If the service runs at all, the trigger lives outside every repo the reef can see — a crontab, an Airflow DAG or a person."
  - "Whether model/artifacts/iforest_v3.pkl exists in the deployed environment. app/main.py loads it at module import scope, so its absence would make the process fail to start; the file is not in the repo, is not produced by any script in the repo, and there is no Dockerfile or CI workflow that would fetch it."
  - "What the '_v3' in iforest_v3.pkl and the string returned by GET /health actually denote. The version is whatever payload['version'] the pickle carries — an opaque value written by a training run that exists nowhere in this repo (Q-026, partially unresolved)."
  - "Whether the model has been retrained since 2024-11. model/features.py says 'no retraining record since 2024-11', but that is a comment, not a log; nothing in the repo or in sources/context records training runs."
  - "Whether detector.detect() can run at all. It reads lowercase row keys (r['sangtae_cd']) while app/db.py uses pymysql DictCursor, which returns keys with the column's case — uppercase here. Static reading says every call raises KeyError; not verified against a live database. See SCH-SETTLEMENT-ANOMALY."
  - "Who acts on a SETTLEMENT_ANOMALY row (Q-025, unresolved). The registry records the gap as a TODO, the README declares remediation out of scope, and no repo, runbook, policy or ticket in sources/context names a consumer, a screen or a query over the table."
  - "Whether the deployed interpreter is Python 3.9 (README) or 3.11 (services.yaml). requirements.txt pins libraries but not an interpreter, and there is no Dockerfile or .python-version."
  - "Whether DUP_SETTLE and FEE_MISMATCH were implemented and later removed, or only ever documented. No git history was consulted; only the working tree was read."
tags:
  - settlement
  - anomaly-detection
  - fastapi
  - python
  - machine-learning
aliases:
  - "settlement-anomaly"
  - "정산 이상 탐지"
relates_to:
  - type: "refines"
    target: "[[API-SETTLEMENT-ANOMALY]]"
  - type: "integrates_with"
    target: "[[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]]"
  - type: "refines"
    target: "[[DEC-SETTLEMENT-ANOMALY-STANDALONE]]"
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]]"
  - type: "integrates_with"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "integrates_with"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:.env.template"
    notes: "DB_URL and MODEL_PATH — both dead; neither is read by any code."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:README.md"
    notes: "Stack, ownership, the four declared detection types, the claimed 03:00 trigger, the out-of-scope statement."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
    notes: "pymysql DictCursor, database hardcoded to sellflow_order."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "The cross-boundary SELECT, the INSERT, model load at import scope."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/schemas.py"
    notes: "AnomalyRow — defined, unused, and not matching the table."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "The two implemented rules and the pickle loader."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/features.py"
    notes: "Four declared features, the 2024-11 retraining note and the 2025-02-10 TODO."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:requirements.txt"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
    notes: "The only DDL; REVIEWED_BY / REVIEWED_DTM."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:tests/test_detector.py"
    notes: "The repo's only test — one assertion over a list of strings."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "The reader filters on BAESONG_WANRYO only — cancellation is deliberately not a condition."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Registers two jobs, neither of which calls this service; no 03:00 trigger exists."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "Writes CANCEL_RECON_QUEUE for exactly the population CANCELLED_SETTLED re-derives."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/settlement/anomaly/openapi.json"
    notes: "Tier-4 extracted spec for the two endpoints, incl. the x-anomaly-codes gap table."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
    notes: "Names 한지우 (데이터팀) as the analyst — the 지우 of the 2025-02-10 TODO in features.py."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "데이터플랫폼본부 데이터팀 (윤서진, 7 people) owns it; 변경이력 dates the 2025-06-09 start."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
    notes: "Authored by the same team lead; never mentions this service."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Python 3.11, db MySQL (settlement), and the TODO that no remediation owner is defined."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "The duplicate-run incident that DUP_SETTLE would have caught, had it been implemented."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Lists POST /detect under 'Unscheduled / externally triggered'."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:schemas/settlement/anomaly/schema.md"
    notes: "Tier-4 ERD and the cross-service read table."
notes: "Code refs are relative to each repo root and prefixed with the repo name. Company-document and extracted-source refs are relative to the reef root."
---

# Settlement Anomaly Detection Service

## Overview

`settlement-anomaly` is a small FastAPI service that reads one settlement date's results, scores them, and writes suspicious rows into `SETTLEMENT_ANOMALY`. It is a detector and nothing more — the README closes with "후속 조치 프로세스는 본 서비스 범위 밖이다." ("the follow-up remediation process is outside this service's scope") (`README.md`).

Three things make it worth a careful read. First, it has no boundary of its own: `app/db.py` connects straight into `sellflow_order` and joins two of settlement-batch's tables to one of order-service's in a single SELECT (`app/main.py`). Second, its strongest rule, `CANCELLED_SETTLED`, re-derives by query exactly the population that `OrderEventRelayJob` already parks in `CANCEL_RECON_QUEUE` — so 셀플로우 has two independent detectors of the cancel-after-settlement problem and a consumer for neither (see [[RISK-SETTLEMENT-RECON-BACKLOG]]). Third, nothing in any of the five repos calls it, and nothing in any repo or document reads its output.

See [[API-SETTLEMENT-ANOMALY]] for the endpoint surface, [[SCH-SETTLEMENT-ANOMALY]] for the table and its mismatches, [[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]] for what a row does after insertion (the answer is: nothing), [[DEC-SETTLEMENT-ANOMALY-STANDALONE]] for why it is a separate Python service at all, and [[RISK-SETTLEMENT-ANOMALY]] for the consequences.

## Key Facts

- Stack is FastAPI 0.104.1 on uvicorn 0.24.0 with scikit-learn 1.3.2, pymysql 1.1.0 and pydantic 2.5.2; no auth library, no HTTP client, no scheduler → settlement-anomaly:requirements.txt
- The FastAPI app declares `title="settlement-anomaly", version="1.4.0"` and exposes exactly two routes, `POST /detect` and `GET /health` → settlement-anomaly:app/main.py
- The README states the interpreter as Python 3.9 while services.yaml records `runtime: FastAPI / Python 3.11`; requirements.txt pins no interpreter and there is no Dockerfile or `.python-version` to settle it → settlement-anomaly:README.md, sellflow-docs:context/registry/services.yaml
- Owned by 데이터플랫폼본부 데이터팀 (Data Platform Division, Data Team), led by 윤서진, 7 people, whose listed process includes "이상 정산 탐지 모델 운영" ("operating the anomalous-settlement detection model") → sellflow-docs:context/org-chart.md (조직도)
- Operation started 2025-06, dated precisely by the org-chart change log: "2025-06-09 settlement-anomaly 운영 시작 (데이터팀)" ("settlement-anomaly entered operation (Data Team)") → sellflow-docs:context/org-chart.md (변경이력)
- The README declares four detection types — `AMT_OUTLIER`, `DUP_SETTLE`, `CANCELLED_SETTLED`, `FEE_MISMATCH` — in a table; `model/detector.py` implements two of them, `CANCELLED_SETTLED` and `AMT_OUTLIER` → settlement-anomaly:README.md, settlement-anomaly:model/detector.py
- Detection is hybrid by design — "금액 이상은 IsolationForest, 나머지는 규칙 기반으로 판정한다." ("amount anomalies use IsolationForest, the rest are judged by rules") → settlement-anomaly:model/detector.py
- `POST /detect` runs one SELECT joining `SETTLEMENT_DTL` → `SETTLEMENT_RUN` (on `RUN_ID`, filtered by `JUNGSAN_ILJA`) → `ORDER_MST` (on `ORD_NO`), i.e. it reads two settlement-batch tables and one order-service table in a single statement across two service boundaries (answers Q-024) → settlement-anomaly:app/main.py
- The only table it writes is `SETTLEMENT_ANOMALY`, always with `STATUS='DETECTED'` and `DETECTED_DTM=NOW()`, one INSERT per hit with no batching and no transaction spanning the loop → settlement-anomaly:app/main.py
- `app/db.py` hardcodes `database="sellflow_order"` — the shared instance created by the 2019 consolidation — and only reads `DB_HOST` and `DB_USER` from the environment; the `DB_URL` in `.env.template` names a `settlement` database and is never read by any code → settlement-anomaly:app/db.py, settlement-anomaly:.env.template
- No caller and no scheduler exists for `POST /detect` anywhere: settlement-batch's `QuartzConfig` registers only `dailySettlementQuartzJob` (cron `0 0 2 * * ?` Asia/Seoul) and `orderEventRelayJob` (every 10 minutes), and grepping `anomal` and `8090` across all five repos returns hits only inside settlement-anomaly itself → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java, sellflow-docs:infra/settlement/queues.md
- `REVIEWED_BY` and `REVIEWED_DTM` exist on the table and are written by nothing — not by this repo, not by any other of the five, and no screen, query or runbook over `SETTLEMENT_ANOMALY` appears in sources/context (Q-025 is unresolved because the consumer genuinely does not exist) → settlement-anomaly:sql/V1__anomaly_schema.sql, sellflow-docs:context/registry/services.yaml
- The registry states the gap in one line: "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" ("performs detection only; the party responsible for action is not defined") → sellflow-docs:context/registry/services.yaml
- The model is a pickle loaded at module import scope — `detector = AnomalyDetector.load("model/artifacts/iforest_v3.pkl")` at line 19 of `app/main.py`, outside any route and before the first request — so a missing or unreadable artefact fails the whole process at startup, not the request → settlement-anomaly:app/main.py
- `model/artifacts/iforest_v3.pkl` is not in the repository, and no training script, notebook, DVC pointer, Makefile or CI workflow that would produce or fetch it exists there either; the repo contains exactly ten files → settlement-anomaly:model/detector.py, sellflow-docs:infra/settlement/runtime.md
- Versioning is whatever the pickle says: `load()` reads `payload["model"]` and `payload["version"]`, and `GET /health` returns that string verbatim as `{"status": "ok", "model": detector.version}` — there is no version pin, no checksum and no registry (this is the whole of Q-026 that code can answer) → settlement-anomaly:model/detector.py, settlement-anomaly:app/main.py
- `model/features.py` opens with "이상 탐지 피처. 2024-11 이후 재학습 이력 없음." ("anomaly detection features. No retraining record since 2024-11") — a comment written before the service entered operation in 2025-06 → settlement-anomaly:model/features.py
- `features.py` declares four features (`amount_zscore`, `partner_daily_count`, `cancel_after_settle_flag`, `fee_rate_delta`) but `_amount_score` feeds the model a two-element vector of raw `jungsan_amt` and `susuryo`, so the declared feature list and the scored vector have nothing in common → settlement-anomaly:model/features.py, settlement-anomaly:model/detector.py
- A dated in-code TODO says the cancel flag dominates: "TODO(지우) 2025-02-10: cancel_after_settle_flag 가 사실상 단독으로 결과를 좌우한다. 가중치 재조정 필요." ("cancel_after_settle_flag effectively determines the outcome on its own; weights need rebalancing"); 한지우 is the 데이터팀 analyst listed on the 2026 automation kickoff → settlement-anomaly:model/features.py, sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md
- The repo's only test asserts that the string `cancel_after_settle_flag` is present in the `FEATURES` list; it never constructs a detector and never touches the rules → settlement-anomaly:tests/test_detector.py
- The 2026 automation plan for exactly this problem domain, authored by this service's own team lead 윤서진, never mentions settlement-anomaly — its As-Is flow is drawn as 주문 취소 → 대기열 적재 → 정산팀 확인, with no detection step → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md

## Responsibilities

**What the code actually does**

| Responsibility | Entry point | Notes |
|---|---|---|
| Read one settlement date's joined rows | `detect()` in `app/main.py` | One SELECT across `SETTLEMENT_DTL` + `SETTLEMENT_RUN` + `ORDER_MST`. |
| Flag cancelled-yet-settled orders | `AnomalyDetector.detect` | Rule-based, constant score 1.0. |
| Score amounts for outliers | `AnomalyDetector._amount_score` | IsolationForest, flagged above 0.85. |
| Persist hits | `execute(...)` in `app/main.py` | One INSERT per hit, `STATUS='DETECTED'`. |
| Report the loaded model version | `GET /health` | Echoes the pickle's own `version` field. |
| Own the `SETTLEMENT_ANOMALY` DDL | `sql/V1__anomaly_schema.sql` | The repo's only DDL; applied by hand — there is no Flyway or Alembic here. |

## Does NOT Own

- **Remediation of anything it detects.** Stated twice, in the README ("후속 조치 프로세스는 본 서비스 범위 밖이다.") and in the registry TODO. No code path updates `STATUS`, `REVIEWED_BY` or `REVIEWED_DTM` → settlement-anomaly:README.md, sellflow-docs:context/registry/services.yaml
- **`SETTLEMENT_DTL`, `SETTLEMENT_RUN`, `ORDER_MST`.** It reads all three and writes none of them; they belong to settlement-batch and order-service → settlement-anomaly:app/main.py
- **`CANCEL_RECON_QUEUE`.** The queue holding the same population is written by settlement-batch's relay job; this service neither reads nor writes it, and never cross-checks its own findings against it → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **Its own database.** No schema of its own; it lives inside `sellflow_order` alongside order and settlement tables → settlement-anomaly:app/db.py
- **Its own trigger.** No Quartz, APScheduler, cron file or CI workflow exists in the repo; the README's schedule is a claim about something outside it → settlement-anomaly:README.md
- **Its own model training.** No training code, dataset reference or experiment record is in the repo; the artefact arrives from somewhere unnamed → settlement-anomaly:model/detector.py
- **Authentication.** No dependency, middleware or header check on either route; nothing in requirements.txt could provide one → settlement-anomaly:app/main.py, settlement-anomaly:requirements.txt
- **`AnomalyRow`.** `app/schemas.py` defines it with `run_id` / `anomaly_type` fields that match neither the table nor the insert; no route uses it → settlement-anomaly:app/schemas.py

## Core Concepts

**Anomaly code (`ANOMALY_CD`)** — the classification string written to `SETTLEMENT_ANOMALY`. Four values are documented, two are producible. The vocabulary is registered in [[GLOSSARY-SETTLEMENT]].

**Score** — a single `DECIMAL(5,4)` column carrying two incomparable quantities: a constant 1.0 for `CANCELLED_SETTLED` (a certainty, not a probability) and the negated IsolationForest `score_samples` output for `AMT_OUTLIER` (unbounded). Ranking a mixed result set by `SCORE` is therefore meaningless → settlement-anomaly:model/detector.py, settlement-anomaly:sql/V1__anomaly_schema.sql

**Detection-only boundary** — the service writes rows and stops. The table's review columns describe a workflow that was designed and never built.

**A join instead of an integration** — there is no HTTP client and no queue consumer anywhere in the repo. Every "integration" this service has is a table name inside a SQL string, which is why [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]] documents a read contract nobody signed.

## Dependencies

| System | Integration Type | Purpose | Auth Method |
|---|---|---|---|
| settlement-batch (`SETTLEMENT_DTL`) | Direct SQL read on the shared `sellflow_order` instance | Source rows: `RUN_ID`, `ORD_NO`, `PARTNER_ID`, `JUNGSAN_AMT`, `SUSURYO` | MySQL user from `DB_USER` (default `sellflow`), no password in `DSN` → settlement-anomaly:app/db.py |
| settlement-batch (`SETTLEMENT_RUN`) | Direct SQL read, joined on `RUN_ID` | Restricts the scan to one `JUNGSAN_ILJA` | Same shared MySQL credential |
| order-service (`ORDER_MST`) | Direct SQL read, joined on `ORD_NO` | Supplies `SANGTAE_CD`, the input to the `CANCELLED_SETTLED` rule | Same shared MySQL credential |
| `SETTLEMENT_ANOMALY` (own table, same instance) | Direct SQL write | One INSERT per detected hit | Same shared MySQL credential |
| `model/artifacts/iforest_v3.pkl` | Local filesystem, `pickle.load` at import | Supplies the IsolationForest and the version string | None — an unsigned pickle deserialised at startup |
| Unknown caller | Inbound HTTP `POST /detect` | Triggers a detection run | **None.** No auth on either route |

Every database dependency crosses a service boundary with no API, no contract and no read-only credential. The 2019 shared-instance decision is what makes this possible — see `DEC-SELLFLOW-SHARED-DB` and [[SYS-SETTLEMENT]].

## Domain Behavior Highlights

**The `CANCELLED_SETTLED` rule scores 1.0 by construction.** `detect()` checks `r["sangtae_cd"] in {"CHWISO", "BANPUM"}` (취소 / cancelled and 반품 / returned) and, when it matches, appends `{"code": "CANCELLED_SETTLED", "score": 1.0}` and `continue`s — the amount model never sees that row. So the constant is not a confidence estimate; it is the rule asserting certainty, and it also means a cancelled row can never be reported as an amount outlier → settlement-anomaly:model/detector.py

**That rule is a cross-service consistency check on a condition the batch never asks about.** `DailySettlementJobConfig.settlementTargetReader` selects `WHERE m.SANGTAE_CD = 'BAESONG_WANRYO'` under a Javadoc that says "주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다. 배송이 완료되었다면 파트너는 이행을 마친 것으로 본다." ("the order's current status and whether it was cancelled are not included in the condition; if delivery completed, the partner is considered to have fulfilled") — a statement of intent that the SQL below it does not implement, since `SANGTAE_CD` *is* a current-status predicate and a cancel overwrites it. The detector joins `ORDER_MST`'s status as it stands at detection time, not as it stood at settlement time, so a `CANCELLED_SETTLED` hit means a cancellation landed *after* settlement — precisely the SF-2287 population that `OrderEventRelayJob` inserts into `CANCEL_RECON_QUEUE` with `STATUS='PENDING'`. Two independent detectors, no consumer for either → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java

**The "model" is one pickle file and a string (Q-026).** `AnomalyDetector.load(path)` opens the file, `pickle.load`s it and takes `payload["model"]` and `payload["version"]`. The path is the hardcoded literal `"model/artifacts/iforest_v3.pkl"` in `app/main.py` — the `MODEL_PATH=./model/detector.pkl` in `.env.template` points at a different filename and is never read. Versioning therefore has no mechanism at all: whatever string the pickle carries is what `/health` reports, and swapping the file silently changes the model with no audit trail. Retraining likewise has no mechanism; the only statement about it is the comment in `features.py` saying there has been none since 2024-11, seven months before the service went live → settlement-anomaly:app/main.py, settlement-anomaly:model/detector.py, settlement-anomaly:.env.template, settlement-anomaly:model/features.py

**Nothing reads a detected row (Q-025).** `SETTLEMENT_ANOMALY` has an index on `(STATUS, DETECTED_DTM)` — the shape of a "show me open items" query — and columns `REVIEWED_BY` / `REVIEWED_DTM` for whoever would close them. Checked: all five repos (no other SQL mentions the table), services.yaml (records the owner as undefined, TODO), the runbooks, the correction policy documents and the 2026 automation plan. The plan is the sharpest evidence: written in 2026-08 by 윤서진, this service's own team lead, to automate settlement corrections, its As-Is diagram runs 주문 취소 → 정산 정정 대기열 적재 → 정산팀 담당자 확인 → 차월 정산 차감 with no detection step and no mention of the service her team has operated since 2025-06 → settlement-anomaly:sql/V1__anomaly_schema.sql, sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md, sellflow-docs:context/registry/services.yaml

**The 03:00 daily trigger is claimed, not implemented.** The README says "정산 배치 종료 후 (매일 03:00) 트리거된다." ("it is triggered after the settlement batch ends, daily at 03:00"). The batch itself does fire at 02:00 KST via Quartz, so the hour is plausible — but the scheduler that would call `/detect` an hour later is in no repo. `QuartzConfig` has two `@Bean` triggers and neither does HTTP; settlement-batch has no web client; the extracted infra notes file this endpoint under "Unscheduled / externally triggered". Absence of evidence is the finding here: either an unversioned crontab outside the repos calls it, or the service has not run since it was written → settlement-anomaly:README.md, settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java, sellflow-docs:infra/settlement/queues.md

**`DUP_SETTLE` is documented and absent — and it is the failure that actually happened.** The 2025-07-12 incident was a duplicate settlement batch run. The detector for it exists only as a README table row → settlement-anomaly:README.md, sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

**`FEE_MISMATCH` compares against a contract rate nobody stores.** It would check the applied commission against the contract rate — a comparison the settlement batch cannot make either, because it hardcodes the rate (see [[PROC-SETTLEMENT-DAILY-BATCH]]) → settlement-anomaly:README.md

## Runtime Components

| Component | Tech Stack | Purpose | Entry Point |
|---|---|---|---|
| HTTP app | FastAPI 0.104.1 on uvicorn 0.24.0, port 8090 | Serves `POST /detect` and `GET /health` | `uvicorn app.main:app --port 8090` (README) → `app/main.py` |
| Detection run | Plain synchronous function, no background task | Query → score → insert, in one request | `detect(req: DetectRequest)` in `app/main.py` |
| Detector | Python + scikit-learn 1.3.2 IsolationForest | Rule for `CANCELLED_SETTLED`, model for `AMT_OUTLIER` | `AnomalyDetector.detect` / `._amount_score` in `model/detector.py` |
| Model artefact | Pickle, loaded once at import | Supplies the estimator and the version string | `AnomalyDetector.load("model/artifacts/iforest_v3.pkl")`, `app/main.py` line 19 |
| DB access | pymysql 1.1.0, `DictCursor`, connection per call | `fetch_all` / `execute` against `sellflow_order` | `app/db.py` |
| Schema | Hand-applied MySQL DDL, no migration tool | Creates `SETTLEMENT_ANOMALY` | `sql/V1__anomaly_schema.sql` |
| Tests | pytest-style module, one assertion | Asserts a string is in `FEATURES` | `tests/test_detector.py` |
| Deployment | — | **None in the repo**: no Dockerfile, no CI workflow, no deploy script | — |

## Related

- [[API-SETTLEMENT-ANOMALY]] — the two endpoints, their shapes and the missing caller
- [[SCH-SETTLEMENT-ANOMALY]] — the `SETTLEMENT_ANOMALY` table, the case mismatch and the unused `AnomalyRow`
- [[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]] — what happens to a detected row: `DETECTED` and no further state
- [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]] — the unwritten read contract over `SETTLEMENT_DTL` / `SETTLEMENT_RUN`
- [[DEC-SETTLEMENT-ANOMALY-STANDALONE]] — why detection became a separate Python service on someone else's tables
- [[RISK-SETTLEMENT-ANOMALY]] — unowned output, untriggered endpoint, unversioned model
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the backlog this service's `CANCELLED_SETTLED` rule independently re-derives
- [[PROC-SETTLEMENT-DAILY-BATCH]] — the batch whose output this service reads, and the hardcoded commission rate
- [[SYS-SETTLEMENT]] — the settlement system and the shared-database decision underneath it
- [[GLOSSARY-SETTLEMENT]] — `ANOMALY_CD`, `CHWISO`, `BANPUM`, `JUNGSAN_ILJA` and the rest of the vocabulary
