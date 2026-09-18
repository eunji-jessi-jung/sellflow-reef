---
id: "CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH"
type: "contract"
title: "Settlement-Anomaly ↔ Settlement-Batch Table-Read Contract"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Answers Q-024. Built 2026-09-19 from the whole of settlement-anomaly (ten files), settlement-batch's six Flyway migrations and its reader/mapper/writer/relay classes, order-service's OrderStatus enum and its V14–V16 migrations, plus the extracted schema and runtime notes. The contract described here is a single SQL string in app/main.py with no counterpart in settlement-batch's repository, so it has only one freshness anchor on each side: re-read whenever that SELECT changes, whenever SETTLEMENT_DTL/SETTLEMENT_RUN gain or lose a column, or whenever OrderStatus gains a cancellation-like value."
freshness_triggers:
  - "order-service/src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "order-service/src/main/resources/db/migration/V16__revert_rename_bigo.sql"
  - "settlement-anomaly/app/db.py"
  - "settlement-anomaly/app/main.py"
  - "settlement-anomaly/model/detector.py"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
known_unknowns:
  - "Whether the SELECT has ever run against production. There is no caller for POST /detect, no scheduler in settlement-anomaly, and no row count from SETTLEMENT_ANOMALY in any source, so the contract's exercise cannot be demonstrated — only its text."
  - "Who, if anyone, at 정산팀 knows that settlement-anomaly reads their tables. No document in sources/context names the dependency: services.yaml lists the two services separately with no relation, and neither the 2026 automation plan nor the 2026-06-18 kickoff mentions the read."
  - "Which settlement-batch query referenced ORDER_CANCEL.BIGO in January 2024 (the V16 precedent). The migration comment is the only attestation and grep -rn BIGO settlement-batch/ returns nothing today, so the precedent's mechanism is documented but its code is gone."
  - "Whether SETTLEMENT_RUN ever contains rows at all. No code in settlement-batch or settlement-anomaly inserts into it; both read it. If it is empty the join yields nothing and /detect returns 0 regardless of the data."
  - "What the deployed SETTLEMENT_ANOMALY / SETTLEMENT_DTL grants are. app/db.py connects as DB_USER (default 'sellflow') with no password in configuration; no grant script exists in any repo, so whether the read is even permitted at the database level is unverified."
tags:
  - contract
  - cross-system
  - shared-database
  - settlement
  - anomaly-detection
  - implicit-coupling
aliases:
  - "settlement-anomaly table read"
  - "이상 탐지 ↔ 정산 배치 연동"
  - "CANCELLED_SETTLED contract"
relates_to:
  - type: "depends_on"
    target: "[[API-SETTLEMENT-ANOMALY]]"
  - type: "integrates_with"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-ORDER-MIGRATION-HISTORY]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
  - type: "integrates_with"
    target: "[[SYS-SETTLEMENT]]"
  - type: "integrates_with"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
    notes: "The seven SANGTAE_CD values; CHWISO, BANPUM, BAESONG_WANRYO, JUNGSAN_WANRYO"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java"
    notes: "The 2024-05 FIXME about JUNGSAN_WANRYO confusing a consumer of the same column"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "Post-SF-2287 cancel path: only CHWISO/BANPUM block a cancellation"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V15__rename_bigo_to_memo.sql"
    notes: "The rename that broke a settlement query"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql"
    notes: "'정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애' — the precedent for this contract's failure mode"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Lists both services with no relation between them; CANCEL_RECON_QUEUE consumer 'TODO # 확인 필요'"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Quartz trigger inventory; files POST /detect under 'Unscheduled / externally triggered'"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "Shared-database topology table — all five services resolve to sellflow_order"
  - category: "external"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "4,127 PENDING rows / 188,851,520 KRW — the population CANCELLED_SETTLED re-derives"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:schemas/settlement/anomaly/schema.md"
    notes: "'Tables read from other services' — the three-table read, extracted"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:schemas/settlement/batch/schema.md"
    notes: "SETTLEMENT_DTL/SETTLEMENT_RUN column definitions and migration churn"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:.env.template"
    notes: "DB_URL naming a dedicated anomaly user and settlement database; read by no code"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:README.md"
    notes: "The claimed '정산 배치 종료 후 (매일 03:00) 트리거된다'"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
    notes: "Hardcoded database='sellflow_order', DictCursor, no password"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "The contract itself — one SELECT literal spanning three tables and two other services"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "CANCELLED_STATES = {'CHWISO', 'BANPUM'}; the CANCELLED_SETTLED rule"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:requirements.txt"
    notes: "Five pins, none of them a scheduler"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
    notes: "SETTLEMENT_ANOMALY, the sink; REVIEWED_BY/REVIEWED_DTM never written"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:tests/test_detector.py"
    notes: "The repo's only test; asserts a string is in a list, touches no SQL"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "The reader: WHERE m.SANGTAE_CD = 'BAESONG_WANRYO', with the javadoc that claims cancellation-blindness the SQL does not implement"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Two triggers; neither calls settlement-anomaly"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/SettlementTarget.java"
    notes: "jungsanAmt/susuryo set after mapping — the derived money columns the detector consumes"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/SettlementTargetRowMapper.java"
    notes: "Maps ORD_NO, PARTNER_ID, SANGPUM_CD, SURYANG, DANGA — never JUNGSAN_AMT or SUSURYO"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
    notes: "The producer of every column settlement-anomaly reads, and its currentRunId() SETTLEMENT_RUN read"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "The other detector of the same exposure — COUNT(1) FROM SETTLEMENT_DTL, then CANCEL_RECON_QUEUE"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "Settlement writes ORDER_MST.SANGTAE_CD='JUNGSAN_WANRYO' — the value that makes the join's population finite"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "SETTLEMENT_DTL and SETTLEMENT_RUN DDL; the 2019 shared-instance note"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V4__cancel_recon_queue_index.sql"
    notes: "An index on a column that does not exist — evidence that schema changes here are unchecked"
notes: "The unusual thing about this contract is that only one party has a copy. Everything settlement-batch is obliged to do here is written down exclusively in the other team's Python file."
---

## Parties

- **settlement-anomaly (데이터팀)** — the *consumer*, and the only party that holds a copy of the contract. `POST /detect` issues one SQL statement that reads two settlement-batch tables and one order-service table, joins them, and scores the result. It writes nothing back into either producer's tables; its only write is `INSERT INTO SETTLEMENT_ANOMALY`.
- **settlement-batch (정산팀)** — the *producer* of `SETTLEMENT_DTL` and `SETTLEMENT_RUN`. It does not know it is a producer. Nothing in its repository — no migration comment, no javadoc, no README line, no test — mentions settlement-anomaly or the columns it exposes.
- **order-service (주문팀)** — a *third, incidental producer*. The join reaches `ORDER_MST.SANGTAE_CD`, whose values are defined by an enum in order-service and whose rows are written by both order-service and settlement-batch. Order-service is equally unaware.

There is no API between these parties, no queue, no event, no view, no foreign key, and no written agreement. The integration medium is the shared `sellflow_order` MySQL schema that all five services resolve to, established by the 2019 consolidation decision recorded in the settlement schema's own header → `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`, `sellflow-docs:infra/settlement/runtime.md`

## Key Facts

- **The entire contract is one SQL string literal in one file.** `app/main.py` lines 28–36 build `SELECT d.RUN_ID, d.ORD_NO, d.PARTNER_ID, d.JUNGSAN_AMT, d.SUSURYO, m.SANGTAE_CD FROM SETTLEMENT_DTL d JOIN SETTLEMENT_RUN r ON r.RUN_ID = d.RUN_ID JOIN ORDER_MST m ON m.ORD_NO = d.ORD_NO WHERE r.JUNGSAN_ILJA = %(ilja)s`. It is the only cross-service read in the repository → `settlement-anomaly:app/main.py`
- **Six columns cross the boundary, across three tables and two owning teams.** From settlement-batch: `SETTLEMENT_DTL.{RUN_ID, ORD_NO, PARTNER_ID, JUNGSAN_AMT, SUSURYO}` and `SETTLEMENT_RUN.{RUN_ID, JUNGSAN_ILJA}`. From order-service: `ORDER_MST.{ORD_NO, SANGTAE_CD}`. Two join keys, both natural and neither declared: `SETTLEMENT_RUN.RUN_ID = SETTLEMENT_DTL.RUN_ID` and `ORDER_MST.ORD_NO = SETTLEMENT_DTL.ORD_NO` → `settlement-anomaly:app/main.py`, `sellflow-docs:schemas/settlement/anomaly/schema.md`
- **No foreign key backs either join.** `V1__settlement_schema.sql` declares `SETTLEMENT_DTL` with `PRIMARY KEY (RUN_ID, ORD_NO)` and two secondary indexes and no `REFERENCES` clause anywhere; the extracted batch schema records "No foreign keys" for the whole repository. The database will therefore not refuse a change that breaks the join → `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`, `sellflow-docs:schemas/settlement/batch/schema.md`
- **Every column read is produced by exactly one statement on the other side.** `SettlementItemWriter.write()` does `INSERT INTO SETTLEMENT_DTL (RUN_ID, ORD_NO, PARTNER_ID, JUNGSAN_AMT, SUSURYO) VALUES (?,?,?,?,?)` — a five-column insert matching the five columns the detector selects, column for column → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, `settlement-anomaly:app/main.py`
- The two amount columns arrive at `SETTLEMENT_DTL` already derived, not raw: `SettlementTargetRowMapper` maps only `ORD_NO`, `PARTNER_ID`, `SANGPUM_CD`, `SURYANG`, `DANGA` out of the order tables, and `jungsanAmt`/`susuryo` are set later on the same object by the processor. So the detector's `AMT_OUTLIER` feature vector `[[jungsan_amt, susuryo]]` consumes a computed figure whose derivation lives entirely in settlement-batch and is invisible to the consumer → `settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/SettlementTargetRowMapper.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/domain/SettlementTarget.java`, `settlement-anomaly:model/detector.py`
- **`SETTLEMENT_RUN` contributes exactly one thing — the date filter — and it is the table with no writer.** The detector joins it solely to reach `JUNGSAN_ILJA`; meanwhile the extracted batch schema records that `SETTLEMENT_RUN` is read by `SettlementItemWriter.currentRunId()`, read by `MarkSettledTasklet`, read by settlement-anomaly, and **inserted by nothing in either repo** → `settlement-anomaly:app/main.py`, `sellflow-docs:schemas/settlement/batch/schema.md`
- **The contract has no owner on either side.** `services.yaml` lists `settlement-batch` (owner `정산팀`) and `settlement-anomaly` (owner `데이터팀`) as unrelated entries; its `queues:` section, the only place in the registry where an inter-service relation is expressed, names `CANCEL_RECON_QUEUE` and `ORDER_EVENT_OUTBOX` and nothing about this read. The file's own header claims to be authoritative — "이 파일이 서비스·소유팀·저장소의 단일 기준이다" ("this file is the single standard for services, owning teams and repositories") — and was last reviewed 2026-03-02 → `sellflow-docs:context/registry/services.yaml`
- **It has no version.** There is no schema version, no API version, no `Accept` header, no migration number the consumer pins to. `settlement-anomaly` carries `FastAPI(title="settlement-anomaly", version="1.4.0")`, which versions its own HTTP surface, not the tables it reads; settlement-batch's Flyway versions (V1–V6) are known only inside settlement-batch → `settlement-anomaly:app/main.py`, `sellflow-docs:infra/settlement/runtime.md`
- **It has no test on either side.** settlement-anomaly's only test is `assert "cancel_after_settle_flag" in FEATURES` — it constructs no connection and executes no SQL. settlement-batch's only test is `SettlementItemProcessorTest`, and its CI builds every environment with `./gradlew clean build -x test`, so even that one does not run on deploy → `settlement-anomaly:tests/test_detector.py`, `sellflow-docs:infra/settlement/runtime.md`
- **A column rename in settlement-batch therefore breaks settlement-anomaly silently, and nothing in settlement-batch's repository would show it.** `grep -rn "settlement-anomaly\|SETTLEMENT_ANOMALY\|8090\|/detect"` across all five repos returns hits only inside `settlement-anomaly/` itself — no reference from the producers back to the consumer exists to be noticed during a schema change → `settlement-anomaly:app/main.py`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- **The precedent is not hypothetical: it has already happened on the order side.** `V15__rename_bigo_to_memo.sql` (2024-01) renamed `ORDER_CANCEL.BIGO` to `MEMO`; `V16__revert_rename_bigo.sql` (2024-01-18) undid it with the reason in the file — "V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애." ("rollback of V15. The settlement batch query references BIGO directly, causing an outage") → `order-service:src/main/resources/db/migration/V15__rename_bigo_to_memo.sql`, `order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql`
- The V16 episode also shows how such a break is resolved here: the column was renamed *back* rather than the consuming query fixed, because the query lived in another team's repository and release train. This contract has the same shape with one column fewer of warning — V16's break announced itself as an outage in a running batch; a break here would announce itself as zero detections → `order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql`, `sellflow-docs:schemas/settlement/batch/schema.md`
- **The batch reader asks nothing about cancellation — though not with the effect its javadoc claims.** `settlementTargetReader` selects `WHERE m.SANGTAE_CD = 'BAESONG_WANRYO' AND DATE(m.UPD_DTM) = ?`; no join or predicate mentions cancellation, but the first predicate is a current-status check, and a cancel overwrites `SANGTAE_CD` and `UPD_DTM` together, so cancelled orders fall out of the target set as a side effect. The javadoc states an intent the SQL does not implement: "배송완료 일자 기준이며 주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다. 배송이 완료되었다면 파트너는 이행을 마친 것으로 본다." ("it is based on the delivery-completion date; the order's current status and whether it was cancelled are not part of the condition. If delivery completed, the partner is considered to have fulfilled its obligation.") → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- **The anomaly detector re-derives by query exactly what that blindness leaves behind.** `detector.py` defines `CANCELLED_STATES = {"CHWISO", "BANPUM"}` and flags any joined row whose `SANGTAE_CD` is in that set as `CANCELLED_SETTLED`, score `1.0`, with the rule comment "규칙: 취소 상태인데 정산에 포함되어 있으면 이상" ("rule: if it is in a cancelled state but included in settlement, it is an anomaly") → `settlement-anomaly:model/detector.py`
- **`OrderEventRelayJob` computes the same population from the other direction, and parks it in `CANCEL_RECON_QUEUE`.** For each unpublished `order.cancelled` event it runs `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` and inserts a `PENDING` row when the count is positive — "already settled AND now cancelled", which is the join predicate `SETTLEMENT_DTL ⋈ ORDER_MST WHERE SANGTAE_CD ∈ {CHWISO, BANPUM}` expressed event-at-a-time instead of day-at-a-time → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `settlement-anomaly:model/detector.py`
- **Neither of the two detectors has a consumer.** `CANCEL_RECON_QUEUE`'s intended drain, `CancelReconciler`, is registered on no Quartz trigger (`QuartzConfig` declares exactly `dailySettlementJobDetail`, cron `0 0 2 * * ?`, and `orderEventRelayJobDetail`, every 10 minutes); the registry records `consumer: TODO   # 확인 필요` ("TODO — needs checking"). `SETTLEMENT_ANOMALY`'s rows are read by nothing, and its `REVIEWED_BY`/`REVIEWED_DTM` columns have no writer → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`, `sellflow-docs:context/registry/services.yaml`, `settlement-anomaly:sql/V1__anomaly_schema.sql`
- The measured size of the population both detectors see: 4,127 `PENDING` rows totalling 188,851,520 KRW, oldest month 2023-04 → `sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv`
- **Nothing triggers the detection.** The README asserts "정산 배치 종료 후 (매일 03:00) 트리거된다." ("it is triggered after the settlement batch finishes, daily at 03:00"), but `QuartzConfig` registers no third trigger, settlement-batch has no HTTP client, `settlement-anomaly` contains no cron file, systemd unit, CI workflow or scheduling library (`requirements.txt` is fastapi, uvicorn, scikit-learn, pymysql, pydantic), `grep -rni "cron\|schedul\|apscheduler\|celery" settlement-anomaly/` returns nothing, and the extracted infra notes file `POST /detect` under "Unscheduled / externally triggered" → `settlement-anomaly:README.md`, `settlement-anomaly:requirements.txt`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`, `sellflow-docs:infra/settlement/queues.md`
- The claimed 03:00 would in any case be one hour after the batch's `0 0 2 * * ?` cron, which is consistent with the README's story — the trigger was designed, dated and documented, and then not built → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`, `settlement-anomaly:README.md`
- **Even if it were triggered, the read would not survive first contact.** `app/db.py` uses `pymysql.cursors.DictCursor` and the SELECT aliases no column, so rows arrive keyed `RUN_ID`, `ORD_NO`, `PARTNER_ID`, `JUNGSAN_AMT`, `SUSURYO`, `SANGTAE_CD`, while `detector.py` reads `r["sangtae_cd"]`, `r["ord_no"]`, `row["jungsan_amt"]`, `row["susuryo"]` — a `KeyError` on the first row → `settlement-anomaly:app/db.py`, `settlement-anomaly:model/detector.py`
- **The consumer connects with more privilege than the contract needs.** `app/db.py` hardcodes `database="sellflow_order"` and reads only `DB_HOST` and `DB_USER` (default `sellflow`), with no password key in the DSN — so the read crosses the boundary as a general application user on the shared instance, not as a scoped reader. `.env.template`'s `DB_URL=mysql://anomaly:@localhost:3306/settlement` describes a dedicated `anomaly` user on a dedicated `settlement` database and is read by no code → `settlement-anomaly:app/db.py`, `settlement-anomaly:.env.template`
- The `SANGTAE_CD` vocabulary this contract depends on is an enum in a third repository: `OrderStatus` declares `GYEOLJE_WANRYO`, `SANGPUM_JUNBI`, `BAESONG_JUNG`, `BAESONG_WANRYO`, `JUNGSAN_WANRYO`, `CHWISO`, `BANPUM`. Two of these strings are duplicated as a Python set literal in `detector.py`, and a third (`BAESONG_WANRYO`) as a Java string literal in the batch reader. No shared constant, code table or `CHECK` constraint links the three copies → `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java`, `settlement-anomaly:model/detector.py`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- `ORDER_MST.SANGTAE_CD` is additionally written by settlement-batch itself — `MarkSettledTasklet` sets `SANGTAE_CD='JUNGSAN_WANRYO'` for the last run's orders, under the same 2019 agreement ("ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)" — "ORDER_MST is the order team's table, but under the integrated-DB policy the settlement batch updates it directly"). So the column this contract reads has two writers in two repositories and an enum owner in a third → `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`

## Agreement

**There is no agreement.** That is the finding, and it should be read literally rather than as a criticism of documentation hygiene.

What exists in place of one is a SELECT statement in `settlement-anomaly/app/main.py`, written by 데이터팀 around 2025-06, that names five columns of `SETTLEMENT_DTL`, two of `SETTLEMENT_RUN` and two of `ORDER_MST` — tables owned by 정산팀 and 주문팀 respectively — and joins them on `RUN_ID` and `ORD_NO`. What the producers owe the consumer, and for how long, is nowhere recorded; what the consumer is entitled to assume is recorded only in the shape of the query itself.

The obligations that the statement silently imposes on settlement-batch are:

| Implicit obligation | What the consumer assumes | Where it is written down |
|---|---|---|
| `SETTLEMENT_DTL` keeps the names `RUN_ID`, `ORD_NO`, `PARTNER_ID`, `JUNGSAN_AMT`, `SUSURYO` | column names, exactly, uppercase | only in `settlement-anomaly/app/main.py` |
| `SETTLEMENT_RUN` keeps `RUN_ID` and `JUNGSAN_ILJA`, and `JUNGSAN_ILJA` stays a DATE comparable to a Python `date` | type and semantics of the date filter | only in `settlement-anomaly/app/main.py` |
| `SETTLEMENT_DTL.RUN_ID` remains joinable to `SETTLEMENT_RUN.RUN_ID` | referential shape, with no FK to enforce it | nowhere |
| `SETTLEMENT_DTL.ORD_NO` remains joinable to `ORDER_MST.ORD_NO` — an inner join, so an order deleted or archived out of `ORDER_MST` silently removes its settlement row from detection | referential shape across a *third* team's table | nowhere |
| `JUNGSAN_AMT` and `SUSURYO` remain numeric and remain the settled amount and the fee | the meaning of two derived money columns | nowhere; the derivation is in `SettlementItemProcessor`, which the consumer never reads |
| `SANGTAE_CD` continues to carry the literal strings `CHWISO` and `BANPUM` for cancellation and return | order-service's enum, copied as a Python set literal | `OrderStatus.java`, in a repository neither party to this read owns |

None of these appear in settlement-batch's migrations, javadoc, README or tests, and none appear in `services.yaml`. The registry's only vocabulary for inter-service coupling is its `queues:` block, which has no entry for a table read. The consequence is stated plainly by the V16 precedent recorded in [[SCH-ORDER-MIGRATION-HISTORY]]: in this system a column name on a shared table is a cross-team interface, and there is no mechanism — no FK, no view, no contract test, no CI check, no consumer registry — that would tell the owning team who depends on it.

One further asymmetry is worth naming. This contract is *read-only* in both directions that matter: settlement-anomaly never writes `SETTLEMENT_DTL`, `SETTLEMENT_RUN` or `ORDER_MST`, and it never calls either producer over HTTP. That is the single property keeping the coupling benign — nothing this consumer does can corrupt a producer's data. It is also why the coupling has survived unnoticed: a read leaves no trace in the producer's logs, its code, or its incident history until it stops working, and a read that stops working produces silence.

```mermaid
sequenceDiagram
    autonumber
    participant OS as order-service<br/>(주문팀)
    participant SB as settlement-batch<br/>(정산팀)
    participant DB as sellflow_order<br/>(shared MySQL)
    participant SA as settlement-anomaly<br/>(데이터팀)
    participant X as trigger for POST /detect

    Note over OS,SA: 02:00 KST — Quartz dailySettlementTrigger (cron 0 0 2 * * ?)
    SB->>DB: SELECT ORD_NO, PARTNER_ID, SANGPUM_CD, SURYANG, DANGA<br/>FROM ORDER_MST JOIN ORDER_DTL<br/>WHERE SANGTAE_CD='BAESONG_WANRYO'
    Note right of SB: no cancellation predicate; javadoc claims<br/>"주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다"<br/>but SANGTAE_CD is a current-status filter
    SB->>DB: INSERT INTO SETTLEMENT_DTL<br/>(RUN_ID, ORD_NO, PARTNER_ID, JUNGSAN_AMT, SUSURYO)
    SB->>DB: UPDATE ORDER_MST SET SANGTAE_CD='JUNGSAN_WANRYO'

    Note over OS,DB: any time — a customer cancels an already-settled order (SF-2287, 2023-04)
    OS->>DB: UPDATE ORDER_MST SET SANGTAE_CD='CHWISO'<br/>+ INSERT ORDER_EVENT_OUTBOX

    rect rgb(238,238,238)
    Note over SB,DB: every 10 min — OrderEventRelayJob: detector #1
    SB->>DB: SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO=?
    SB->>DB: INSERT INTO CANCEL_RECON_QUEUE (STATUS='PENDING')
    Note right of DB: 4,127 rows / 188,851,520 KRW<br/>CancelReconciler has no Quartz trigger — no consumer
    end

    Note over X,SA: 03:00 KST — README claims a daily trigger here
    X--xSA: POST /detect  — NO CALLER FOUND<br/>(no scheduler in repo, no client in any of the 5 repos)

    rect rgb(238,238,238)
    Note over SA,DB: if it ran — detector #2, same population
    SA->>DB: SELECT d.RUN_ID, d.ORD_NO, d.PARTNER_ID, d.JUNGSAN_AMT,<br/>d.SUSURYO, m.SANGTAE_CD<br/>FROM SETTLEMENT_DTL d<br/>JOIN SETTLEMENT_RUN r ON r.RUN_ID = d.RUN_ID<br/>JOIN ORDER_MST m ON m.ORD_NO = d.ORD_NO<br/>WHERE r.JUNGSAN_ILJA = %(ilja)s
    DB-->>SA: rows keyed UPPERCASE (DictCursor)
    Note right of SA: detector reads r["sangtae_cd"] → KeyError
    SA->>DB: INSERT INTO SETTLEMENT_ANOMALY<br/>(ANOMALY_CD='CANCELLED_SETTLED', SCORE=1.0, STATUS='DETECTED')
    Note right of DB: no reader; REVIEWED_BY/REVIEWED_DTM never written
    end
```

## Current State

**The read is defined and, on the evidence available, has never been exercised.**

The verification for the trigger claim is worth setting out step by step, because "no caller" is the load-bearing fact and absence has to be demonstrated rather than asserted:

1. `settlement-anomaly/README.md` claims one: "정산 배치 종료 후 (매일 03:00) 트리거된다."
2. The only scheduler in the settlement domain is `settlement-batch`'s `QuartzConfig`, which declares four beans forming two trigger pairs — `dailySettlementJobDetail`/`dailySettlementTrigger` (cron `0 0 2 * * ?`, Asia/Seoul) and `orderEventRelayJobDetail`/`orderEventRelayTrigger` (every 10 minutes, forever). There is no third.
3. settlement-batch could not call an HTTP endpoint even if a trigger existed: the extracted runtime notes record `SettlementBatchApplication` as having no web starter and no HTTP port, and no HTTP client dependency.
4. `settlement-anomaly` contains no scheduling mechanism of its own. Its whole file list is ten files; `requirements.txt` pins fastapi, uvicorn, scikit-learn, pymysql and pydantic; `ls -la` shows no `.github/`, no crontab, no Dockerfile, no systemd unit; and a case-insensitive grep for `cron`, `schedul`, `apscheduler` and `celery` across the repository returns nothing.
5. `grep -rni "settlement-anomaly\|SETTLEMENT_ANOMALY\|8090\|/detect"` across all five repositories returns matches only inside `settlement-anomaly/` itself — four in its README, one in `.env.template`, four in `app/main.py` and `sql/V1__anomaly_schema.sql`. No external caller of any kind exists in version control.
6. The independently extracted infra notes reach the same conclusion from the other side, filing `POST /detect` under "Unscheduled / externally triggered" with "the trigger is external and unidentified".

So the contract's current state is: a well-formed, correctly-named, correctly-joined SELECT against live production tables, with no scheduled moment at which it runs, and — from the row-key case mismatch in step (6) of the diagram — a consumer that would fail on the first returned row if it did.

The producer side, by contrast, runs. `SETTLEMENT_DTL` is written nightly. The columns the contract depends on exist today exactly as the query names them; the contract is *currently satisfied*, and satisfied by coincidence rather than by any commitment.

## Impact Analysis

### If settlement-batch changes `SETTLEMENT_DTL` or `SETTLEMENT_RUN`

A rename or drop of any of `RUN_ID`, `ORD_NO`, `PARTNER_ID`, `JUNGSAN_AMT`, `SUSURYO` or `JUNGSAN_ILJA` produces a MySQL error inside settlement-anomaly's `fetch_all`, surfacing as a 500 from `POST /detect` — **in the other team's service, in the other team's logs, under the other team's alerting**, which is to say nowhere that 정산팀 will look. Nothing in settlement-batch's own build, test or deploy would fail: CI runs `-x test`, there is no FK, and no reference to the consumer exists in the repository to grep for.

The failure mode is worse than an error, though, because of what the consumer does with it. There is no caller for `/detect`, so a break would not even produce an error — it would produce continued silence, indistinguishable from the current silence. The only observable difference between "this contract works" and "this contract has been broken for eighteen months" is the row count of a table nobody queries.

The V15/V16 precedent establishes that this is a real failure mode in this organisation and not a theoretical one, and it establishes the resolution pattern too: in January 2024 주문팀 renamed `ORDER_CANCEL.BIGO` for internal consistency, an unnamed settlement-batch query broke in production, and the fix, at most 18 days later (V15's comment gives only `2024-01`; V16's is dated `2024-01-18`), was to rename the column back — "V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애." The difference is instructive: that break announced itself as an outage in a running batch. This one has no running consumer to break loudly.

Adjacent evidence that schema changes here are not carefully consumer-checked: V4 indexes a `CANCEL_RECON_QUEUE.REG_DT` column that does not exist, V5 adds `PROCESSED_AT`/`PROCESSED_BY` while the code writes V1's `PROCESSED_DTM`, and V2/V6 create and index a `SETTLEMENT_RUN_LOG` that no code reads or writes. A team that ships indexes on non-existent columns in its own schema will not catch a dependency it has never been told about.

### If order-service changes `SANGTAE_CD` values

This is the sharper exposure, because it degrades *quietly and partially* rather than erroring.

`detector.py` hardcodes `CANCELLED_STATES = {"CHWISO", "BANPUM"}`. If 주문팀 adds a new terminal cancellation-like status — a partial cancellation, a payment-failure void, a fraud reversal — the join still succeeds, the query still returns rows, `/detect` still returns HTTP 200, and every order in the new state is scored as *normal*. No error, no log line, no count discrepancy. The batch reader has the mirror-image exposure on the same column: it keys on the literal `'BAESONG_WANRYO'`, so a rename or a new pre-settlement status changes what gets settled at all.

The vocabulary lives in `OrderStatus.java` with seven values and no persistence-level constraint: `ORDER_MST.SANGTAE_CD` is `VARCHAR(20) NOT NULL` with an index and no `CHECK`. Three independent copies of these strings exist — the enum in order-service, a Python set in settlement-anomaly, a Java literal in settlement-batch — and the column has two writers across two repositories (order-service's cancel path, and settlement-batch's `MarkSettledTasklet` writing `JUNGSAN_WANRYO`). Nothing keeps the three copies in step, and order-service has no reason to know that two exist.

A concrete near-miss already in the record: `OrderSearchService` carries "FIXME(은영) 2024-05: 상태코드 필터에 JUNGSAN_WANRYO 넣으면 결과가 비어 보인다는 CS 문의" — a status-code value crossing a boundary and confusing a consumer, inside order-service's own repository.

### What a remediation plan must not assume

This section exists because the 2026 settlement-correction automation is being designed around this problem domain, and this artifact's most consequential claim is about what is *not* there.

1. **Do not assume this is a working control.** `CANCELLED_SETTLED` is a correct rule that has, on the evidence, never fired. It has no trigger (verified six ways above), it would `KeyError` on its first row if it did fire, and its output table has no reader. Counting it as existing detection coverage would overstate the organisation's position on exactly the exposure the plan is meant to close.
2. **Do not assume the two detectors are independent safeguards.** `OrderEventRelayJob` and `CANCELLED_SETTLED` compute the *same* population — settled orders subsequently cancelled — by two different routes. They are not defence in depth; they are one control implemented twice, and drained zero times. Remediating one does not cover the other, and remediating the queue does not remove the need to decide what `SETTLEMENT_ANOMALY` is for.
3. **Do not assume that a fixed detector would produce new information.** If `/detect` were triggered and its row keys fixed tomorrow, its `CANCELLED_SETTLED` output would be a re-derivation of the 4,127 rows already sitting `PENDING` in `CANCEL_RECON_QUEUE` since 2023-04. The organisation's problem is not that the exposure is undetected; it is detected twice and acted on never. See [[RISK-SETTLEMENT-RECON-BACKLOG]].
4. **Do not assume the plan's volume figure.** The 2026 automation plan is sized on "월 10건 내외" ("around 10 cases a month"), a 2023 estimate; the 2026-09-01 queue export shows 4,127 rows over 41 months, 100–160 a month for the last two years. A detector re-derived from `SETTLEMENT_DTL` would surface a caseload an order of magnitude above the design assumption.
5. **Do not assume a schema change is safe because the tests pass.** There are no tests on either side of this boundary, and settlement-batch deploys with `-x test` in every environment. The only thing that would reveal a break is someone reading `settlement-anomaly/app/main.py` — a file in a repository owned by a different 본부.
6. **Do not assume the coupling is discoverable from the producer.** Any remediation that touches `SETTLEMENT_DTL`, `SETTLEMENT_RUN` or `ORDER_MST` needs a consumer inventory built by grepping the other four repositories, because `services.yaml` — which declares itself the single standard — does not model table-level dependencies at all. This artifact and [[SCH-SETTLEMENT-ANOMALY]] are currently the only written record that the dependency exists.

## Related

- [[SYS-SETTLEMENT-ANOMALY]] — the consuming service, its ownership and its components
- [[SYS-SETTLEMENT]] — the producing service, the daily batch and its schedule
- [[SYS-ORDER]] — owner of `ORDER_MST` and of the `SANGTAE_CD` vocabulary this contract depends on
- [[SCH-SETTLEMENT-BATCH]] — column definitions for `SETTLEMENT_DTL` and `SETTLEMENT_RUN`, and the migration churn around them
- [[SCH-SETTLEMENT-ANOMALY]] — the "tables read from other services" table and the row-key case analysis
- [[SCH-ORDER-MIGRATION-HISTORY]] — the V15/V16 BIGO episode in full, the precedent for this contract's failure mode
- [[API-SETTLEMENT-ANOMALY]] — `POST /detect`, the endpoint whose absent caller means this contract never executes
- [[CON-ORDER-SETTLEMENT]] — the other cancellation contract, whose `CANCEL_RECON_QUEUE` this one independently re-derives
- [[DEC-SELLFLOW-SHARED-DB]] — the 2019 decision that makes a cross-service table read possible at all
- [[RISK-SETTLEMENT-ANOMALY]] — the consumer-side risk register, including the missing model artefact and the absent trigger
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the 188,851,520 KRW exposure both detectors see and neither resolves
