---
id: "DEC-SETTLEMENT-ANOMALY-STANDALONE"
type: "decision"
title: "Anomaly Detection as a Standalone Service on Another Team's Tables"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Reconstructed on 2026-09-19 from the whole of settlement-anomaly (ten files), settlement-batch's build.gradle, README, SettlementItemWriter and its six Flyway migrations, the service registry, the org chart and its change log, the extracted runtime/queue/openapi notes, and a grep of the entire sources/ tree for any written rationale. No decision record for this split exists; this ADR is reconstructed from artefacts, and every inference is labelled as one. Stale the moment a Dockerfile, a CI workflow, a shared DDL or a contract test appears in either repository, or SETTLEMENT_DTL gains a V7 migration."
freshness_triggers:
  - "sellflow-docs:context/org-chart.md"
  - "sellflow-docs:context/registry/services.yaml"
  - "sellflow-docs:infra/settlement/runtime.md"
  - "settlement-anomaly:README.md"
  - "settlement-anomaly:app/db.py"
  - "settlement-anomaly:app/main.py"
  - "settlement-anomaly:requirements.txt"
  - "settlement-anomaly:sql/V1__anomaly_schema.sql"
  - "settlement-batch:build.gradle"
  - "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - "settlement-batch:src/main/resources/db/migration/"
known_unknowns:
  - "The rationale itself. No decision record, minute, ticket, wiki page, mail thread or Slack message anywhere in sources/ discusses why detection was built as a separate service rather than inside settlement-batch. I grepped the whole sources/ tree for 'anomaly', 'settlement-anomaly', '이상 탐지' and '이상탐지': the only hits are the service registry, the org chart, and the reef's own tier-4 extraction notes (runtime.md, queues.md, the two schema.md files, the openapi pair). The sixteen documents under sources/context — including the 2024 draft review, the 2026 automation plan, the kickoff minutes, the handover, both policy versions, the incident postmortem and the sprint records — mention it nowhere. The decision below is the shape of the artefacts, not a record of anyone's reasoning."
  - "Whether the split was ever decided at all, as opposed to defaulting. A Python/scikit-learn service in a Java 8 shop is also simply what the team that owns models would build; no source distinguishes a considered architectural choice from the path of least resistance."
  - "Who reviewed or approved it. services.yaml declares itself the single standard for services and owning teams — '이 파일이 서비스·소유팀·저장소의 단일 기준이다' — and its settlement-anomaly entry carries no approver, no date and no reference to settlement-batch; the file has not been reviewed since 2026-03-02."
  - "Who was told about the table read. Nothing in settlement-batch's repository — no migration comment, no javadoc, no README line, no test — names settlement-anomaly or the five SETTLEMENT_DTL columns it depends on. Whether anyone at 정산팀 knows is unrecorded (see .reef/questions-for-owner.md)."
  - "What the intended consumer of SETTLEMENT_ANOMALY was. The table has REVIEWED_BY and REVIEWED_DTM and an index on (STATUS, DETECTED_DTM), so a review workflow was designed; the registry records the owner as 'TODO' and no source names a screen, a query or a person."
  - "Whether a shared-library, view or API alternative was ever considered and rejected. No trade-off discussion exists in any source, so the Alternatives section below reconstructs what was available in this codebase, not what anyone weighed."
  - "Whether the split has ever been operationally exercised. There is no caller for POST /detect in any of the five repositories, no Dockerfile, no CI workflow and no deploy script, so 'a separately deployed service' is a claim about a deployment nobody can point at from version control."
tags:
  - "settlement"
  - "anomaly-detection"
  - "service-boundary"
  - "ownership"
  - "shared-database"
  - "adr-reconstructed"
aliases:
  - "why settlement-anomaly is a separate service"
  - "이상 탐지 별도 서비스 분리"
  - "anomaly standalone split"
relates_to:
  - type: "feeds"
    target: "[[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-OWNERSHIP]]"
  - type: "feeds"
    target: "[[PROC-SETTLEMENT-ANOMALY-VS-BATCH]]"
  - type: "feeds"
    target: "[[RISK-SETTLEMENT-ANOMALY]]"
  - type: "depends_on"
    target: "[[SCH-ORDER-MIGRATION-HISTORY]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "constrains"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
  - type: "refines"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V15__rename_bigo_to_memo.sql"
    notes: "The 2024-01 rename of ORDER_CANCEL.BIGO — this decision's failure mode, already realised once."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql"
    notes: "'V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애.' — the break, and the rename-it-back resolution."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/settlement/anomaly/openapi.json"
    notes: "The two-endpoint surface the split produced, and the x-anomaly-codes gap between documented and emitted codes."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "The two owner teams in two divisions; the 2025-06-09 operation-start row and the 2025-09-01 headcount row."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Two independent service entries, two owner teams, two declared databases, last_reviewed 2026-03-02; the remediation-owner TODO."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "POST /detect filed under 'Unscheduled / externally triggered'; the Quartz trigger inventory that contains no third trigger."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "Side-by-side stacks, the deploy rows, and the shared-database topology table showing both services resolving to sellflow_order."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:.env.template"
    notes: "DB_URL naming a dedicated anomaly user and settlement database — the separation the code does not implement."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:README.md"
    notes: "Stack, owning team, 2025-06 start, the four declared detection types, and the out-of-scope statement."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
    notes: "pymysql, hardcoded database='sellflow_order' — the boundary that is not a boundary."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "The five-column SELECT that reproduces settlement-batch's insert order as a string literal."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "CANCELLED_STATES duplicated from order-service's enum; the hybrid rule/model design that motivates a Python runtime."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:requirements.txt"
    notes: "scikit-learn 1.3.2 — the dependency Java 8 / Spring Batch could not host."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
    notes: "The service's own DDL, Flyway-named with no Flyway; REVIEWED_BY / REVIEWED_DTM."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:README.md"
    notes: "재무본부 정산팀 ownership, Java 8 / Spring Boot 2.3 / Spring Batch 4, the 02:00 job."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:build.gradle"
    notes: "sourceCompatibility 1.8, Spring Batch/Quartz/Flyway/JDBC — no ML runtime and no HTTP client."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
    notes: "The producer-side string literal whose column order the consumer reproduces."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "SETTLEMENT_DTL DDL, the 2019 shared-instance note, and the cross-team ORDER_MST precedent."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql"
    notes: "An index on a column that does not exist — the schema-change review standard the read depends on."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
    notes: "Columns added for code that does not exist; the same repository's habit of shipping schema ahead of consumers."
notes: "The comparison of the two services is in PROC-SETTLEMENT-ANOMALY-VS-BATCH and the read itself in CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH; neither is repeated here. This artifact records only the decision, what it bought, what it cost, and the fact that nobody wrote it down."
---

# Anomaly Detection as a Standalone Service on Another Team's Tables

## Context

At some point before 2025-06, 셀플로우 decided that settlement anomaly detection would be a separate application rather than a step, a job or a module inside the settlement batch that produces the data it examines.

**No written rationale for that decision exists.** This needs stating first, because everything below is reconstruction. I searched the whole of the reef's `sources/` tree for `anomaly`, `settlement-anomaly`, `이상 탐지` and `이상탐지`. The only files that match are the service registry, the org chart, and the reef's own tier-4 extraction notes — `infra/settlement/runtime.md`, `infra/settlement/queues.md`, `schemas/settlement/anomaly/schema.md`, `schemas/settlement/batch/schema.md` and the `apis/settlement/anomaly/` pair. None of the sixteen company documents under `sources/context/` mentions the service at all: not the 2024 automation review draft, not the 2026 automation plan, not the 2026-06-18 kickoff minutes, not the 2025-03 handover, neither version of the correction procedure, not the 2025-07-12 incident postmortem, and not the 2026-S17 sprint records → sellflow-docs:context/registry/services.yaml, sellflow-docs:context/org-chart.md

What does exist is a repository, a table, two registry rows and two org-chart lines. An ADR reconstructed from artefacts is still worth writing, because the consequences are live and the reasoning is recoverable from what was built even though the reasoning was never recorded. Where this artifact infers, it says so; where it cannot, the gap is in `known_unknowns`.

The side-by-side comparison of the two applications belongs to [[PROC-SETTLEMENT-ANOMALY-VS-BATCH]] and the column-level read to [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]]. Neither is repeated here.

## Decision

**Settlement anomaly detection was built as `settlement-anomaly`: a separate FastAPI / scikit-learn service, in a separate repository, owned by a different team in a different division, with its own table and its own DDL — and it obtains its input by reading settlement-batch's and order-service's tables directly, reproducing the producer's column order as a Python string literal.**

The split, as the artefacts define it:

| Dimension | settlement-batch | settlement-anomaly |
|---|---|---|
| Owner team | 정산팀 (3 people, 팀장 김도윤) | 데이터팀 (7 people, 팀장 윤서진) |
| Division (본부) | 재무본부 | 데이터플랫폼본부 |
| Repository | `sellflow/settlement-batch` | `sellflow/settlement-anomaly` |
| Runtime | Java 8, Spring Boot 2.3.12, Spring Batch 4, Quartz | FastAPI 0.104.1, uvicorn, scikit-learn 1.3.2 |
| Schema ownership | `V1`–`V6` Flyway migrations | one hand-applied `V1__anomaly_schema.sql`, no migration tool |
| Trigger | Quartz cron `0 0 2 * * ?`, Asia/Seoul | none in version control; README claims 03:00 |
| Interface offered | none — no web starter, no HTTP port | `POST /detect`, `GET /health` on port 8090 |
| Coupling medium | — | a SQL string literal over the shared `sellflow_order` schema |

→ sellflow-docs:context/org-chart.md, sellflow-docs:context/registry/services.yaml, settlement-batch:build.gradle, settlement-anomaly:requirements.txt, sellflow-docs:infra/settlement/runtime.md

What the decision did **not** include, and this is the part that matters: no interface was created along with the boundary. There is no shared DDL, no generated model class, no database view, no API, no event, no foreign key and no contract test between the two. The boundary exists in ownership, repository, language and deployment — and not in data.

## Key Facts

- The two applications are owned by teams in different divisions: `settlement-batch` by 재무본부 정산팀 (3 people, 팀장 김도윤), `settlement-anomaly` by 데이터플랫폼본부 데이터팀 (7 people, 팀장 윤서진), whose listed responsibility includes "이상 정산 탐지 모델 운영" ("operating the anomalous-settlement detection model") → sellflow-docs:context/org-chart.md (조직도)
- The split is dated by the org chart's change log to 2025-06-09 — "settlement-anomaly 운영 시작 (데이터팀)" ("settlement-anomaly entered operation (Data Team)") — and the README agrees with "운영 시작: 2025-06" → sellflow-docs:context/org-chart.md (변경이력), settlement-anomaly:README.md
- The change log shows the same team was reinforced three months later, 2025-09-01, "데이터팀 증원 4명 → 7명 (AI 과제 확대)" ("Data Team headcount 4 → 7, expansion of AI initiatives"), while 정산팀 had been cut 4 → 3 on 2024-07-01. The team that took the new service was growing; the team that owns the data it reads was shrinking → sellflow-docs:context/org-chart.md (변경이력)
- The service registry lists the two as unrelated entries with no relation field between them, and its `queues:` block — the only place the registry expresses inter-service coupling — names `CANCEL_RECON_QUEUE` and `ORDER_EVENT_OUTBOX` and nothing about a table read. The file calls itself authoritative ("이 파일이 서비스·소유팀·저장소의 단일 기준이다" — "this file is the single standard for services, owning teams and repositories") and carries `# last_reviewed: 2026-03-02   # 이후 갱신 없음` ("no updates since") → sellflow-docs:context/registry/services.yaml
- The runtime split is real and was forced by the model: `settlement-anomaly` pins `scikit-learn==1.3.2`, while `settlement-batch/build.gradle` declares `sourceCompatibility = '1.8'` and five Spring/MySQL dependencies with no ML runtime and no HTTP client. An IsolationForest could not have been hosted in the batch as it stands → settlement-anomaly:requirements.txt, settlement-batch:build.gradle
- Detection is explicitly hybrid, which is why only part of it needed Python at all — "금액 이상은 IsolationForest, 나머지는 규칙 기반으로 판정한다." ("amount anomalies use IsolationForest, the rest are judged by rules"). Of the two rules actually implemented, `CANCELLED_SETTLED` is pure SQL-expressible logic with no model in it → settlement-anomaly:model/detector.py
- The service owns exactly one table and its own DDL — `sql/V1__anomaly_schema.sql`, headed "담당: 데이터플랫폼본부 데이터팀 (2025-06)" — named in Flyway's convention while the repository declares no Flyway, no Alembic and no migration library in `requirements.txt` → settlement-anomaly:sql/V1__anomaly_schema.sql, settlement-anomaly:requirements.txt
- **The schema separation is nominal.** `app/db.py` hardcodes `database="sellflow_order"`, the shared instance, so the "own schema" is one table sitting inside the order/settlement database rather than a boundary of any kind → settlement-anomaly:app/db.py, sellflow-docs:infra/settlement/runtime.md
- The separation the split implies *was* written down — in a file nothing reads. `.env.template` sets `DB_URL=mysql://anomaly:@localhost:3306/settlement`: a dedicated `anomaly` user on a dedicated `settlement` database. No code reads `DB_URL`; `app/db.py` reads only `DB_HOST` and `DB_USER` (default `sellflow`) → settlement-anomaly:.env.template, settlement-anomaly:app/db.py
- **The coupling is a reproduced column list.** `SettlementItemWriter.write()` inserts `(RUN_ID, ORD_NO, PARTNER_ID, JUNGSAN_AMT, SUSURYO)` as a Java string literal; `app/main.py` selects `d.RUN_ID, d.ORD_NO, d.PARTNER_ID, d.JUNGSAN_AMT, d.SUSURYO` as a Python string literal — the same five columns in the same order, written twice, in two languages, in two repositories, by two teams, with nothing linking them → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, settlement-anomaly:app/main.py
- The same SELECT reaches a third team's table: it joins `ORDER_MST` on `ORD_NO` for `SANGTAE_CD`, whose vocabulary is an enum in order-service, and duplicates two of its values as `CANCELLED_STATES = {"CHWISO", "BANPUM"}`. So the split placed one consumer across three teams' data with three independent copies of the status vocabulary → settlement-anomaly:app/main.py, settlement-anomaly:model/detector.py
- Nothing in settlement-batch's repository records that it has a consumer. No migration comment, javadoc, README line or test names `settlement-anomaly`, `SETTLEMENT_ANOMALY`, port 8090 or `/detect`; the reference exists only in the consumer's repository → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql, settlement-anomaly:app/main.py
- The deployment half of "separately deployed" is not in version control. `settlement-anomaly` has no Dockerfile, no CI workflow, no deploy script and no manifest — the only operational instruction anywhere is the README's `uvicorn app.main:app --port 8090` → settlement-anomaly:README.md, sellflow-docs:infra/settlement/runtime.md
- The service exposes an HTTP interface and settlement-batch cannot use it: `SettlementBatchApplication` has no web starter, no HTTP port and no HTTP client dependency, so the producer could not call `POST /detect` even if a trigger were wanted. The extracted infra notes file that endpoint under "Unscheduled / externally triggered" → sellflow-docs:infra/settlement/runtime.md, sellflow-docs:infra/settlement/queues.md, settlement-batch:build.gradle
- **The output side of the split was left undefined and is still marked so.** The registry's note for the service reads "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" ("performs detection only; the party responsible for acting is not defined"), and the README states the same from inside: "후속 조치 프로세스는 본 서비스 범위 밖이다." ("the follow-up remediation process is outside this service's scope") → sellflow-docs:context/registry/services.yaml, settlement-anomaly:README.md
- The table was nevertheless built for a handover that has no receiver: `REVIEWED_BY VARCHAR(30)`, `REVIEWED_DTM DATETIME` and `KEY IX_SETTLEMENT_ANOMALY_01 (STATUS, DETECTED_DTM)` — the columns and the index of a review queue, with no writer and no reader in any of the five repositories → settlement-anomaly:sql/V1__anomaly_schema.sql
- The registry's `db:` fields record the split as the designers evidently meant it — `settlement-batch: MySQL (settlement)`, `settlement-anomaly: MySQL (settlement)` — while both applications' code resolves to `sellflow_order`. The registry describes an architecture the code does not have → sellflow-docs:context/registry/services.yaml, sellflow-docs:infra/settlement/runtime.md
- The producer's own migration history shows the review standard the unwritten read depends on: `V4` creates an index on `CANCEL_RECON_QUEUE (STATUS, REG_DT)` where the `V1` column is `RECV_DTM`, and `V5` adds `PROCESSED_AT`/`PROCESSED_BY` beside code that writes `PROCESSED_DTM`, under the comment "아직 쓰는 코드는 없다" ("no code writes them yet") → settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql, settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql

## Rationale

**No rationale is recorded.** What follows is reconstruction from the artefacts, and each item is labelled by how strongly the evidence supports it. The honest version of this section is short.

**Well supported — the model needed a Python runtime.** `settlement-batch` is Java 8 with `sourceCompatibility = '1.8'` and a dependency list of Spring Batch, Quartz, Spring JDBC, Flyway and the MySQL connector. `AMT_OUTLIER` is an IsolationForest from scikit-learn 1.3.2. Hosting that inside the batch would have meant either a JVM reimplementation or an out-of-process call the batch has no client for. This is the one motive the code states clearly enough to rely on → settlement-batch:build.gradle, settlement-anomaly:requirements.txt, settlement-anomaly:model/detector.py

**Well supported — the capability belonged to a different team.** The org chart assigns 데이터팀 "리포팅 · 지표 산출 · ETA 예측 모델 운영 · 이상 정산 탐지 모델 운영" and lists two systems under it, `eta-predictor` and `settlement-anomaly`. The change log shows `eta-predictor` starting 2024-03-02 under the same team, fifteen months before this service. A model-operating team building its second model service in its own stack is the least surprising reading of the artefacts → sellflow-docs:context/org-chart.md

**Partly supported — capacity.** 정산팀 went 4 → 3 in 2024-07 and 데이터팀 went 4 → 7 in 2025-09 "(AI 과제 확대)". A three-person team owning a payout batch was in no position to take on model operation. The dates bracket the decision rather than explain it, and no source connects them → sellflow-docs:context/org-chart.md (변경이력)

**Not supported by any source — the choice of a direct table read as the integration.** This is the consequential half of the decision and there is nothing behind it: no design note, no ticket, no minute. What can be said is that the direct read was the locally available option. The 2019 consolidation put every service in one MySQL schema — `V1__settlement_schema.sql` states it as settled policy, "sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)" ("it uses the same instance as sellflow_order; 2019 consolidation decision") — and the same file normalises cross-team writes: "ORDER_MST 는 주문팀 소유이나 정산 배치가 SANGTAE_CD 를 갱신한다." ("ORDER_MST is the order team's table, but the settlement batch updates SANGTAE_CD"). Against that precedent, reading another team's table without asking is not a deviation; it is the house style, recorded in [[DEC-SELLFLOW-SHARED-DB]] → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql

**Contradicted by the code — a data boundary was intended.** The `.env.template` describing an `anomaly` user on a `settlement` database, and the registry's two separate `db:` entries, both describe a service with its own storage. `app/db.py` hardcodes `sellflow_order` and reads neither. Whoever wrote the template and the registry row believed in a boundary the implementation never created; whether that gap was noticed is unrecorded → settlement-anomaly:.env.template, settlement-anomaly:app/db.py, sellflow-docs:context/registry/services.yaml

### Alternatives

No source shows any of these being weighed. They are reconstructed from what this codebase made available in 2025, and are listed because the consequences below follow from their absence, not because anyone rejected them.

| Alternative | What it would have cost | What it would have prevented |
|---|---|---|
| A step inside `dailySettlementJob` | A JVM outlier implementation or dropping `AMT_OUTLIER`; 정산팀 owning model code with three people | The whole coupling. Detection would run in the same transaction boundary as production, on the objects the writer already holds |
| A database view owned by 정산팀 (`V_SETTLEMENT_DTL_FOR_ANOMALY`) | One migration in settlement-batch, and 정산팀 accepting it as an interface | A rename would break the view in the producer's own Flyway run, at deploy time, in the owner's repository |
| A read API on settlement-batch | A web starter on a batch application that deliberately has none | Column names would stop being the interface entirely |
| An event or an outbox row at settlement completion | Reuse of the `ORDER_EVENT_OUTBOX` pattern already running in this repository every 10 minutes | The pull-based read, its missing trigger, and the consumer's dependence on the producer's schedule |
| A contract test in either repository's CI | A test, and CI that runs tests | Silent breakage. Neither side has a usable gate: settlement-anomaly has no CI at all, and settlement-batch builds every environment with `./gradlew clean build -x test` |
| A shared model or DDL artefact | A cross-repository publishing mechanism the organisation does not have | The five-column literal being written twice in two languages |

→ settlement-batch:build.gradle, settlement-batch:src/main/resources/db/migration/, sellflow-docs:infra/settlement/runtime.md, sellflow-docs:infra/settlement/queues.md

## Consequences

**A column rename in `SETTLEMENT_DTL` is a routine change for one team and a silent break for the other.** This is the decision's central cost and it is not hypothetical. Renaming `JUNGSAN_AMT`, or dropping `PARTNER_ID`, or reordering the insert in `SettlementItemWriter`, requires nothing of 정산팀 beyond a V7 migration. Their build passes — CI runs `-x test`. No foreign key objects; `SETTLEMENT_DTL` has none. No reference to the consumer exists in their repository to grep for. The registry, which declares itself the single standard for services and repositories, has no vocabulary for a table-level dependency and has not been reviewed since 2026-03-02. The break surfaces as a MySQL error inside a Python process in another 본부, in another team's logs, under another team's alerting → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql, settlement-anomaly:app/main.py, sellflow-docs:context/registry/services.yaml, sellflow-docs:infra/settlement/runtime.md

The organisation has already run this experiment on the order side. `V15__rename_bigo_to_memo.sql` renamed `ORDER_CANCEL.BIGO` in January 2024; `V16__revert_rename_bigo.sql` undid it at most 18 days later — V15's comment records only `2024-01`, V16's `2024-01-18` — with the reason in the file — "V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애." ("rollback of V15. The settlement batch query references BIGO directly, causing an outage"). The resolution is as instructive as the incident: the column was renamed *back*, because the consuming query lived in another team's repository and release train. The full episode is in [[SCH-ORDER-MIGRATION-HISTORY]] → order-service:src/main/resources/db/migration/V15__rename_bigo_to_memo.sql, order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql

**The V16 break was loud; this one would be silent.** That precedent announced itself as an outage in a batch that runs nightly. `POST /detect` has no caller anywhere in version control, and its output table has no reader, so a break here changes nothing observable. The only difference between "this works" and "this has been broken for a year" is the row count of a table nobody queries. The producer's own migrations show the standard this relies on: `V4` indexes a `CANCEL_RECON_QUEUE` column that does not exist, and `V5` adds columns for code that does not exist → settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql, settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql, sellflow-docs:infra/settlement/queues.md

**The output table's consumer is an unfilled TODO, and the split is why.** Detection was scoped out of the batch, and remediation was scoped out of detection: the README says "후속 조치 프로세스는 본 서비스 범위 밖이다." and the registry says "조치 주체는 정의되어 있지 않음. TODO". Both statements are correct about their own service and neither team owns the gap between them. `SETTLEMENT_ANOMALY` has the columns of a review workflow — `REVIEWED_BY`, `REVIEWED_DTM`, an index on `(STATUS, DETECTED_DTM)` — and every row is written `STATUS='DETECTED'` by the only code that touches it, so the column has exactly one reachable value. The split produced a handoff with a sender and no receiver → settlement-anomaly:README.md, settlement-anomaly:sql/V1__anomaly_schema.sql, sellflow-docs:context/registry/services.yaml

**The TODO shape repeats across this domain, which suggests it is structural rather than an oversight here.** `CANCEL_RECON_QUEUE` carries `consumer: TODO   # 확인 필요` in the same registry file; `delivery-bff` carries `owner_team: TODO`. In each case a component was built, an owner was named for building it, and the party responsible for the step *after* it was left blank → sellflow-docs:context/registry/services.yaml

**Two teams now detect the same thing and neither acts on it.** 정산팀's `OrderEventRelayJob` parks settled-then-cancelled orders in `CANCEL_RECON_QUEUE`; 데이터팀's `CANCELLED_SETTLED` rule re-derives the same population by query. Neither knows about the other — the two repositories do not reference each other — and neither queue has a drain. Duplicated detection with zero remediation is a direct consequence of splitting the capability without anyone owning the join between the halves. The measured exposure is in [[RISK-SETTLEMENT-ANOMALY]] and the detection overlap in [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]] → sellflow-docs:infra/settlement/queues.md

**The producer cannot trigger the consumer, so the pipeline has a gap where the split is.** The README claims "정산 배치 종료 후 (매일 03:00) 트리거된다.", one hour after the batch's `0 0 2 * * ?` cron — a coherent design. But `SettlementBatchApplication` has no web starter and no HTTP client, `QuartzConfig` registers no third trigger, and `settlement-anomaly` has no scheduler of its own. The two halves of what was conceptually one nightly pipeline were placed in runtimes that cannot call each other, and nothing was built to bridge them → settlement-anomaly:README.md, sellflow-docs:infra/settlement/queues.md, sellflow-docs:infra/settlement/runtime.md

**Divergence in engineering practice followed the org boundary.** settlement-batch has Flyway with six versioned migrations and a CD workflow per environment; settlement-anomaly has one hand-applied DDL file named in Flyway's convention with no Flyway, no lockfile, no Dockerfile, no CI and one test asserting that a string is in a list. Nothing required the new service to inherit the old one's floor, and nothing checked that it had one → settlement-anomaly:sql/V1__anomaly_schema.sql, sellflow-docs:infra/settlement/runtime.md

**What honouring the decision would now require**, taken from the artefacts rather than from a plan: give `SETTLEMENT_DTL` a consumer-facing interface that 정산팀 owns and their own build breaks when they violate — a view is the cheapest, since both services already share the schema; register the dependency somewhere 정산팀 reads, which today means `services.yaml` gaining a concept it does not have; name an owner for `SETTLEMENT_ANOMALY` rows, without which detection is inert regardless of how well it works; and decide who triggers `/detect`, since neither the producer's runtime nor the consumer's repository can. Reversing the split — folding rule-based detection back into the batch and leaving only the model outside — is the other coherent option, and no source shows it having been considered either.

## Related

- [[SYS-SETTLEMENT-ANOMALY]] — the service this decision produced, file by file
- [[SYS-SETTLEMENT]] — the settlement domain and the batch on whose output it depends
- [[PROC-SETTLEMENT-ANOMALY-VS-BATCH]] — the full side-by-side comparison of the two applications
- [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]] — the column-level read contract that only one party holds a copy of
- [[SCH-SETTLEMENT-BATCH]] — `SETTLEMENT_DTL` and `SETTLEMENT_RUN`, the tables this decision made into an interface
- [[SCH-SETTLEMENT-ANOMALY]] — `SETTLEMENT_ANOMALY`, the table with a review workflow and no reviewer
- [[SCH-ORDER-MIGRATION-HISTORY]] — the V15/V16 `BIGO` rename, this decision's failure mode already realised
- [[DEC-SELLFLOW-SHARED-DB]] — the 2019 consolidation that made a cross-team table read the path of least resistance
- [[PROC-SELLFLOW-OWNERSHIP]] — how ownership is recorded across 셀플로우, and where it is recorded as TODO
- [[RISK-SETTLEMENT-ANOMALY]] — the consequences as a risk register: no caller, no consumer, no model artefact
