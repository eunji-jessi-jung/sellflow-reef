---
id: "RISK-SETTLEMENT-ANOMALY"
type: "risk"
title: "Settlement Anomaly Service Risks"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Collected 2026-09-19 from a complete read of settlement-anomaly — all ten files — plus the org chart, the service registry, the 2026 automation plan and the kickoff minutes. The central findings are absences (no caller, no scheduler, no consumer, no model artefact, no training code), each established by reading the whole repository and grepping the other four. Stale if a scheduler, a caller, a Dockerfile, a training script or the pickle itself appears."
freshness_triggers:
  - "settlement-anomaly/.env.template"
  - "settlement-anomaly/app/db.py"
  - "settlement-anomaly/app/main.py"
  - "settlement-anomaly/model/detector.py"
  - "settlement-anomaly/model/features.py"
  - "settlement-anomaly/sql/V1__anomaly_schema.sql"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
  - "sources/context/registry/services.yaml"
known_unknowns:
  - "Whether the service is running in production at all. There is no Dockerfile, no CI workflow, no deploy script and no health-check record; the only evidence of operation is the org-chart line '2025-06-09 settlement-anomaly 운영 시작 (데이터팀)'."
  - "Where model/artifacts/iforest_v3.pkl comes from. It is not in the repository and no training script, notebook, DVC pointer, Makefile or CI job that would produce or fetch it exists in any of the five repos."
  - "Whether detector.detect() has ever executed successfully, given the uppercase/lowercase row-key mismatch. No runtime trace, log sample or production row was available to test the reading."
  - "Whether SETTLEMENT_ANOMALY contains any rows. No export of the table exists in sources/raw, and no query over it appears in any repo or runbook."
  - "Who calls POST /detect. The README asserts a daily 03:00 trigger; no scheduler exists in the repo and grepping /detect, anomaly, settlement-anomaly and 8090 across all five repos returns hits only inside this one."
  - "Whether the omission of this service from the 2026 automation plan is deliberate scoping or an oversight. Only its author, 윤서진, can say."
  - "What the deployed Python interpreter is. The README says 3.9, services.yaml says 3.11, and requirements.txt pins none."
severity: "medium"
resolution: "open"
tags:
  - settlement
  - anomaly-detection
  - risk
  - machine-learning
  - dead-service
aliases:
  - "settlement-anomaly risks"
  - "이상 탐지 리스크"
relates_to:
  - type: "refines"
    target: "[[API-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-AUTH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-FLOW-CATALOG]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
    notes: "The kickoff attended by this service's own analyst; the service is not mentioned"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "데이터팀, 팀장 윤서진, '이상 정산 탐지 모델 운영'; 2025-06-09 operation start"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
    notes: "Authored by the owning team's lead; never mentions the service"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "'조치 주체는 정의되어 있지 않음. TODO'"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Files POST /detect under 'Unscheduled / externally triggered'"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:.env.template"
    notes: "MODEL_PATH and DB_URL, neither read by any code"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
    notes: "Hardcoded sellflow_order, no password, DictCursor"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "Import-scope pickle load, unauthenticated routes, non-idempotent insert loop"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/schemas.py"
    notes: "AnomalyRow — unused and mismatched against the table"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "Two of four documented rules; lowercase row keys"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/features.py"
    notes: "Four declared features; no retraining since 2024-11"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
    notes: "SETTLEMENT_ANOMALY, its review columns and its DECIMAL(5,4) score"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:tests/test_detector.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "The only scheduler in the domain; registers nothing that calls this service"
notes: "Ten files, one of which is a test asserting that a string is in a list. The risk here is not what this service does wrong; it is that an organisation believes it has a settlement safety net."
---

# Settlement Anomaly Service Risks

## Description

`settlement-anomaly` is the only component in 셀플로우 whose purpose is to notice when settlement goes wrong. It was stood up in 2025-06, four weeks before the domain's one recorded incident, and it is listed in the org chart as an operated system under 데이터팀's 이상 정산 탐지 모델 운영 ("operating the anomalous-settlement detection model").

Reading all ten of its files produces an unusual risk profile: almost nothing here can go wrong in the normal sense, because nothing here demonstrably runs. The endpoint has no caller. The scheduler it claims does not exist. The model file it loads is not in the repository. The rows it would write have no reader, no owner and no state after `DETECTED`. And as committed, the detection function would raise `KeyError` on its first row.

The risk, then, is not damage. It is **assurance**: the settlement domain carries a 188,851,520 KRW unreconciled queue and a 42,000,000 KRW duplicate-payout incident, and it has a detection service in its org chart, its service registry and its architecture story that has produced no evidence of ever having detected anything. The sharpest single piece of evidence is that when this service's own team lead wrote the 2026 plan to automate settlement corrections, she did not mention it.

## Key Facts

- The whole repository is ten files: `README.md`, `.env.template`, `requirements.txt`, `app/{main,db,schemas}.py`, `model/{detector,features}.py`, `sql/V1__anomaly_schema.sql`, `tests/test_detector.py` → `settlement-anomaly:README.md`
- **The output table has no consumer.** `SETTLEMENT_ANOMALY` is written by `POST /detect` and read by nothing: no other SQL in any of the five repos names it, and no screen, query, runbook or report over it appears anywhere in `sources/context` → `settlement-anomaly:sql/V1__anomaly_schema.sql`, `settlement-anomaly:app/main.py`
- The table was designed for a review workflow that was never built: `REVIEWED_BY VARCHAR(30)` and `REVIEWED_DTM DATETIME` have no writer anywhere, and the index `IX_SETTLEMENT_ANOMALY_01 (STATUS, DETECTED_DTM)` is the shape of a "show me open items" query that no code issues → `settlement-anomaly:sql/V1__anomaly_schema.sql`
- Every row is inserted with `STATUS='DETECTED'` and no code path ever changes it, so the column has exactly one reachable value → `settlement-anomaly:app/main.py`
- The registry states the gap in one line and marks it unresolved: "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" — "performs detection only; the party responsible for acting is not defined" → `sellflow-docs:context/registry/services.yaml`
- The README says the same from the other side: "후속 조치 프로세스는 본 서비스 범위 밖이다." — "the follow-up remediation process is outside this service's scope" → `settlement-anomaly:README.md`
- **The endpoint has no caller and no scheduler.** `POST /detect` is the only way to produce a row, and nothing invokes it: `settlement-batch`'s `QuartzConfig` registers exactly two triggers (`dailySettlementQuartzJob`, cron `0 0 2 * * ?`, and `orderEventRelayJob`, every 10 minutes), settlement-batch has no HTTP client in `build.gradle`, and this repo contains no cron file, APScheduler usage, CI workflow or systemd unit → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`, `settlement-anomaly:requirements.txt`
- The README nonetheless asserts an operational trigger: "정산 배치 종료 후 (매일 03:00) 트리거된다." — "it is triggered after the settlement batch finishes, daily at 03:00". The extracted infra notes file the same endpoint under "Unscheduled / externally triggered" → `settlement-anomaly:README.md`, `sellflow-docs:infra/settlement/queues.md`
- **The model is a pickle loaded at import scope from a hardcoded path that is not in the repository.** Line 19 of `app/main.py` is `detector = AnomalyDetector.load("model/artifacts/iforest_v3.pkl")`, outside any route, so a missing artefact fails process startup rather than a request — and `model/artifacts/` does not exist in the repo → `settlement-anomaly:app/main.py`, `settlement-anomaly:model/detector.py`
- `load()` does `pickle.load(f)` on that file and takes `payload["model"]` and `payload["version"]` — an unsigned, unchecksummed deserialisation of a file whose provenance is undocumented, executed at startup → `settlement-anomaly:model/detector.py`
- **`MODEL_PATH` is declared and never read.** `.env.template` sets `MODEL_PATH=./model/detector.pkl`; no code reads that variable, and the filename differs from the hardcoded one, so the template documents a model file that the service would not load even if the variable were honoured → `settlement-anomaly:.env.template`, `settlement-anomaly:app/main.py`
- `DB_URL` in the same template is equally inert: it points at `mysql://anomaly:@localhost:3306/settlement` while `app/db.py` reads only `DB_HOST` and `DB_USER` and hardcodes `database="sellflow_order"`. So the template describes a dedicated `settlement` database and a dedicated `anomaly` user, and the code connects to the shared order instance as `sellflow` → `settlement-anomaly:.env.template`, `settlement-anomaly:app/db.py`
- **The declared features do not match the scored vector.** `features.py` lists four — `amount_zscore`, `partner_daily_count`, `cancel_after_settle_flag`, `fee_rate_delta` — while `_amount_score` builds `[[float(row["jungsan_amt"]), float(row["susuryo"])]]`, two raw amounts. The two sets have no member in common → `settlement-anomaly:model/features.py`, `settlement-anomaly:model/detector.py`
- `features.py` opens with "이상 탐지 피처. 2024-11 이후 재학습 이력 없음." — "anomaly detection features. No retraining record since 2024-11" — a note written about a period that begins seven months *before* the service entered operation on 2025-06-09 → `settlement-anomaly:model/features.py`, `sellflow-docs:context/org-chart.md` (변경이력)
- A dated in-code TODO says the model is effectively one rule: "TODO(지우) 2025-02-10: cancel_after_settle_flag 가 사실상 단독으로 결과를 좌우한다. 가중치 재조정 필요." — "cancel_after_settle_flag effectively determines the outcome on its own; weights need rebalancing" → `settlement-anomaly:model/features.py`
- **There is no training code.** No script, notebook, dataset reference, experiment record, `Makefile`, DVC pointer or CI job that would produce `iforest_v3.pkl` exists in this repo or any of the other four → `settlement-anomaly:model/detector.py`
- There is also no version pin, no checksum and no model registry: `GET /health` returns `{"status": "ok", "model": detector.version}`, echoing whatever string the pickle carries, so swapping the file silently changes the model with no audit trail → `settlement-anomaly:app/main.py`, `settlement-anomaly:model/detector.py`
- **Neither route has any authentication.** No `Depends`, no security scheme, no middleware, and nothing in `requirements.txt` that could provide one — so any caller that can reach port 8090 can write `SETTLEMENT_ANOMALY` rows and read the deployed model version → `settlement-anomaly:app/main.py`, `settlement-anomaly:requirements.txt`
- `/detect` is not idempotent: it has no uniqueness guard and the table has no unique constraint beyond its auto-increment key, so calling it twice for the same `jungsan_ilja` doubles the rows → `settlement-anomaly:app/main.py`, `settlement-anomaly:sql/V1__anomaly_schema.sql`
- As committed, `detect()` cannot process a row: `app/db.py` uses `pymysql.cursors.DictCursor` and the SELECT aliases nothing, so rows arrive keyed `RUN_ID`, `ORD_NO`, `PARTNER_ID`, `JUNGSAN_AMT`, `SUSURYO`, `SANGTAE_CD`, while `detector.py` reads `r["sangtae_cd"]`, `r["ord_no"]`, `row["jungsan_amt"]`, `row["susuryo"]` — a `KeyError` on the first row (analysed in [[SCH-SETTLEMENT-ANOMALY]]) → `settlement-anomaly:app/db.py`, `settlement-anomaly:model/detector.py`
- Half the documented detection surface does not exist: the README declares `AMT_OUTLIER`, `DUP_SETTLE`, `CANCELLED_SETTLED` and `FEE_MISMATCH`; `detector.py` emits two. The missing two are the two failures this domain has actually had — a duplicate settlement run (2025-07-12) and a commission rate that ignores contracts → `settlement-anomaly:README.md`, `settlement-anomaly:model/detector.py`, `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- The `SCORE` column is `DECIMAL(5,4)`, maximum 9.9999, and carries two incomparable quantities: a constant `1.0` asserted by the cancellation rule and an unbounded `-score_samples(...)` from IsolationForest. A sufficiently extreme outlier would overflow the column and fail its INSERT → `settlement-anomaly:sql/V1__anomaly_schema.sql`, `settlement-anomaly:model/detector.py`
- `app/schemas.py` defines an `AnomalyRow` (`run_id`, `ord_no`, `anomaly_type`, `score`, `status`) that no route imports and that matches neither the insert nor the table — `SETTLEMENT_ANOMALY` has `ANOMALY_CD`, not `anomaly_type`, and no `RUN_ID` column at all → `settlement-anomaly:app/schemas.py`, `settlement-anomaly:sql/V1__anomaly_schema.sql`
- The only test asserts that the string `cancel_after_settle_flag` appears in the `FEATURES` list; it never constructs a detector, never loads a model and never touches a rule → `settlement-anomaly:tests/test_detector.py`
- There is no deployment path in version control: no Dockerfile, no CI or CD workflow, no deploy script, and the schema file is plain DDL with no migration tool to apply it → `settlement-anomaly:sql/V1__anomaly_schema.sql`, `sellflow-docs:infra/settlement/runtime.md`
- **The owning team's own 2026 automation plan never mentions this service.** PLAN-2026-014, authored 2026-08-18 by 윤서진 — 데이터팀 팀장, the team the org chart lists as operating `settlement-anomaly` — draws the As-Is flow as 주문 취소 → 정산 정정 대기열 적재 → 정산팀 담당자 확인 → 차월 정산 차감 반영 → 처리 완료, with no detection step anywhere → `sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, `sellflow-docs:context/org-chart.md` (조직도)
- The kickoff that preceded that plan was attended by 한지우, the 데이터팀 analyst whose name is on the `features.py` TODO, and the minutes record the same omission: the meeting concludes that 판정 기준 (decision criteria) must be defined from scratch — "기준 문서가 현재 없음. → TBD" — without reference to the rules already written in `detector.py` → `sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md`, `settlement-anomaly:model/features.py`
- The plan's premise is also contradicted by the data this service was built to watch: it states "현행 처리량은 정산팀 확인 결과 월 10건 내외로 파악된다" — "current volume is understood to be around 10 cases a month" — against a queue export showing 4,127 pending rows → `sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, `sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv`
- The README, the registry and the org chart disagree about the runtime: README says Python 3.9, `services.yaml` says 3.11, and `requirements.txt` pins no interpreter → `settlement-anomaly:README.md`, `sellflow-docs:context/registry/services.yaml`

## Findings

| # | Finding | Evidence | Consequence if the service is running | Consequence if it is not | Severity |
|---|---|---|---|---|---|
| 1 | No consumer for `SETTLEMENT_ANOMALY`; `REVIEWED_BY`/`REVIEWED_DTM` have no writer and `STATUS` never leaves `DETECTED` | `sql/V1__anomaly_schema.sql`, `app/main.py`, `services.yaml` | Detections accumulate unread | Nothing detected in the first place | **high** |
| 2 | No caller and no scheduler for `POST /detect`, against a README claiming a daily 03:00 trigger | `QuartzConfig.java`, `app/main.py`, `README.md`, `infra/settlement/queues.md` | — | The safety net has never been armed | **high** |
| 3 | The owning team lead's 2026 automation plan for this exact problem never mentions the service | `plans/2026_정산정정_AI에이전트_자동화_기획.md`, `org-chart.md` | The org does not know its own coverage | Same | **high** |
| 4 | `model/artifacts/iforest_v3.pkl` loaded at import from a hardcoded path and absent from the repo | `app/main.py`, `model/detector.py` | Startup fails; service does not come up | Consistent with #2 | **high** |
| 5 | No training code, no dataset reference, no experiment record, no model registry or checksum | whole repo | The model cannot be reproduced or audited | Same | **medium-high** |
| 6 | Row keys mismatch: `DictCursor` returns uppercase, `detector.py` reads lowercase | `app/db.py`, `model/detector.py` | `KeyError` on the first row of every run | Latent | **medium-high** |
| 7 | Declared features (4) share nothing with the scored vector (2 raw amounts) | `model/features.py`, `model/detector.py` | `AMT_OUTLIER` scores are uninterpretable against the documented design | Latent | **medium-high** |
| 8 | Two of four documented detection types unimplemented — and they are the two failures this domain has had | `README.md`, `model/detector.py`, the 2025-07-12 postmortem | No duplicate or fee-mismatch detection | Same | **medium-high** |
| 9 | No authentication on either route; `/detect` writes rows and is not idempotent | `app/main.py`, `sql/V1__anomaly_schema.sql` | Any caller on port 8090 can inflate the table | Latent | **medium** |
| 10 | No retraining since 2024-11, seven months before launch; one feature dominates by the team's own note | `model/features.py` | Scores drift unmeasured | Latent | **medium** |
| 11 | `MODEL_PATH` and `DB_URL` declared in `.env.template` and read by nothing; both name different targets than the code uses | `.env.template`, `app/main.py`, `app/db.py` | An operator can believe the model and database are configurable when they are not | Same | **medium** |
| 12 | `SCORE DECIMAL(5,4)` holds both a constant 1.0 and an unbounded model score; values above 9.9999 fail the insert | `sql/V1__anomaly_schema.sql`, `model/detector.py` | Extreme outliers — the ones that matter — are the ones that fail to persist | Latent | **medium** |
| 13 | One test, asserting a string is in a list; no CI, no Dockerfile, no deploy path in version control | `tests/test_detector.py`, `infra/settlement/runtime.md` | Nothing verifies any of the above | Same | **medium** |
| 14 | `AnomalyRow` in `schemas.py` unused and mismatched against the table | `app/schemas.py`, `sql/V1__anomaly_schema.sql` | Misleading to the next reader | Same | **low** |
| 15 | Runtime version disagreement: README 3.9 vs registry 3.11, nothing pinned | `README.md`, `services.yaml` | Undefined deployment target | Same | **low** |

## Impact

**Assurance.** This is the dominant impact and it does not depend on whether the service runs. 셀플로우 has a named detection service, in the service registry, in the org chart as a team responsibility, and in the architecture story of the settlement domain. Findings #1 through #4 together mean no evidence exists that it has ever detected anything, and no one would learn if it stopped. A safety net believed to be in place is worse than a known absence, because it suppresses the question.

**Coverage of the failures that actually occurred.** The domain has had exactly two categories of expensive failure: a duplicate settlement run costing roughly 42,000,000 KRW across 17 partners, and a commission rate that ignores contracted tiers. `DUP_SETTLE` and `FEE_MISMATCH` are the two documented rules that do not exist (finding #8). The implemented rule, `CANCELLED_SETTLED`, re-derives by query the same population that `OrderEventRelayJob` already parks in `CANCEL_RECON_QUEUE` — so the domain has two independent detectors of the cancel-after-settlement problem and a consumer for neither → `settlement-anomaly:model/detector.py`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`

**Organisational.** Finding #3 is the one with consequences beyond this repo. The 2026 plan is currently 검토중 (under review) with a 2027-01 pilot, budgeted around a 월 10건 estimate. It is being designed as though no detection capability exists — and, functionally, it is right, but for reasons nobody has written down. The analyst who authored the `features.py` TODO sat in the kickoff that concluded no decision criteria exist, while `detector.py` contains two → `sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, `sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md`

**Security.** Bounded but real: an unauthenticated write endpoint (finding #9) on a shared database connection with no password in configuration, plus an unsigned pickle deserialised at startup (finding #4). Anyone who can place a file at `model/artifacts/iforest_v3.pkl` executes code in this process. See [[PROC-SETTLEMENT-AUTH]].

**Data.** If the service ever does run, findings #6, #7 and #12 mean the rows it writes would be either absent (KeyError), uninterpretable (feature mismatch), or truncated at the extremes (score overflow). There is no scenario evidenced here in which `SETTLEMENT_ANOMALY` currently contains trustworthy data.

## Severity and Resolution

**Severity: medium** — and the reasoning for *not* rating it high is as important as the rating.

Arguments for high: the finding density is extreme even by this codebase's standards. Ten files yield fifteen findings, four of them structural absences (no consumer, no caller, no model artefact, no training code). The service is the designated safety net for a domain that has lost 42,000,000 KRW to one incident and carries 188,851,520 KRW unreconciled. Its own team's plan does not know it exists.

Arguments for medium, which is what this artifact adopts: **an inert service cannot move money.** Every path in this repository terminates in an INSERT into a table nobody reads. There is no evidence of a wrong payout, a wrong order state, or a wrong decision traceable to this service, and none is structurally possible from the code as written — it reads three tables and writes one that no consumer joins. The security exposure is real but bounded by the same inertness and by network reachability that no manifest describes.

The assurance risk — the belief that detection exists — is the highest-value component here, and it is already counted where it bites: in [[RISK-SETTLEMENT]]'s detection impact and in [[RISK-SETTLEMENT-RECON-BACKLOG]]'s account of a backlog that two independent detectors both see and neither resolves. Rating this artifact high as well would double-count it. Medium reflects what this service can do, not what the organisation believes it does.

**Resolution: open, and untracked.** Not one of the fifteen findings has a ticket. The S17 export contains no settlement-anomaly items; the only tracked work touching this problem domain is SF-5120 (a queue viewing screen, In Progress) and SF-5121 (defining correction criteria, To Do, assigned to the same 한지우 who wrote the model TODO) — neither of which references the service → `sellflow-docs:context/sprints/tickets_2026-S17.csv`

## Recommended Actions

1. **Answer the operational question first: is it running?** One `GET /health` against port 8090 in each environment settles findings #2 and #4 at once — a response proves a process exists and names the loaded model version; a connection failure proves it does not. Every other action depends on this answer, and it is a question only someone with environment access can answer (recorded in `.reef/questions-for-owner.md`).
2. **Count the rows.** `SELECT ANOMALY_CD, COUNT(*), MIN(DETECTED_DTM), MAX(DETECTED_DTM) FROM SETTLEMENT_ANOMALY GROUP BY ANOMALY_CD` converts findings #1 and #6 from inference into fact, including whether `detect()` has ever completed a run.
3. **Decide the service's status explicitly** — operated, or retired. It currently occupies the ambiguous third state: listed as operated, evidenced as inert. If it is to be retired, the registry, the org chart and the README all need correcting; if it is to be operated, it needs a trigger, an owner for its output, and the model artefact in source control or a registry.
4. **Raise the omission with 윤서진 before the plan's 2026-10 design phase.** The plan and the service address the same problem for the same team and do not reference each other. `CANCELLED_SETTLED` is a working definition of exactly the 판정 기준 the plan lists as a 선결 과제 (prerequisite task) and the kickoff recorded as nonexistent.
5. **Fix the row-key mismatch or prove it is not real** (finding #6): alias the SELECT columns to lowercase, or read uppercase keys. This is a one-line change that decides whether the service has ever produced a single detection.
6. **Put the model under version control or a registry**, with a checksum, and read `MODEL_PATH` instead of the hardcoded literal (findings #4, #5, #11). Until then the deployed model is unidentifiable and unreproducible.
7. **Implement `DUP_SETTLE` before `FEE_MISMATCH`** if either is implemented (finding #8). `DUP_SETTLE` is a `GROUP BY ORD_NO HAVING COUNT(DISTINCT RUN_ID) > 1` over the same query the service already runs, and it detects the one incident class this domain has demonstrably experienced.
8. **Gate the endpoint** (finding #9) if the service is to be operated: it writes to a shared production database from an unauthenticated route on an undocumented network.

## Related

- [[SYS-SETTLEMENT-ANOMALY]] — the service, its components and its ownership
- [[API-SETTLEMENT-ANOMALY]] — the two endpoints and the missing caller
- [[SCH-SETTLEMENT-ANOMALY]] — the table, the review columns and the row-key case analysis
- [[PROC-SETTLEMENT-FLOW-CATALOG]] — where `POST /detect` sits among the domain's flows, and which of them run
- [[PROC-SETTLEMENT-AUTH]] — the ungated endpoint and the unsigned pickle
- [[RISK-SETTLEMENT]] — the batch-side risks this service was meant to detect
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the backlog that `CANCELLED_SETTLED` independently re-derives
