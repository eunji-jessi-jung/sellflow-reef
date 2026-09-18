
## SYS-INVENTORY — Does a physical `inventory` database still exist, and where did Alembic revision `1c22` go?

**Questions**
1. `inventory-api` runtime code connects to `sellflow_order` (hardcoded in `app/db.py`), yet
   `.env.sample`, `alembic.ini` and the service registry all declare a separate `inventory`
   database. Does that database exist in any environment today, and is anything still pointed
   at it?
2. Alembic revision `3f9a` declares `down_revision = "1c22"`, and no file defines revision
   `1c22`. Was it deleted, never committed, or does it live outside the repo? Related: does
   `RESTORE_LOG` physically exist anywhere, and with what columns? The only revision that
   creates it does so with no columns at all.

**Why it matters**
The answer decides whether the order/inventory boundary is a real service boundary or a
shared-table coupling (the restock path `SELECT`s `ORDER_DTL`, an order-service table, on the
same connection), and whether stock restorations have ever been auditable. Both feed
SYS-INVENTORY, SCH-INVENTORY, CON-ORDER-INVENTORY and PROC-INVENTORY-RESTORE-LOG-LIFECYCLE.

**Already checked**
Every file in the inventory-api repo (16 files) was read. `app/db.py` never imports
`app/config.py`, so `DB_URL` is inert. `requirements.txt` pins only fastapi, uvicorn and pymysql
— alembic and SQLAlchemy are not installed, so the chain likely never ran from this repo. No
deployment manifest, Helm chart or compose file exists in the repo. A grep of the whole repo for
`1c22` finds it only as the `down_revision` string.

**Files a human would need**
- `inventory-api/app/db.py`, `inventory-api/app/config.py`, `inventory-api/.env.sample`,
  `inventory-api/alembic.ini`, `inventory-api/requirements.txt`
- `inventory-api/alembic/versions/20230414_1120-3f9a_add_restore_log.py`
- `inventory-api/alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py`
- `sellflow-reef/sources/context/registry/services.yaml` (entry `inventory-api`, `db: MySQL (inventory)`)

## SYS-ORDER — Which hostname and path prefix actually front order-service?

**Question.** In production, what URL does a caller hit to cancel an order, and is there a gateway that rewrites the path?

**Why it matters.** Three sources disagree, and the disagreement is not cosmetic: it decides whether the one CORS rule in the service protects the cancel endpoint at all.
- `sources/context/registry/services.yaml` lists the endpoint as `https://api.sellflow.co.kr/orders`.
- `delivery-bff:src/generated/orderApi.ts` calls `POST /api/v1/orders/{ordNo}/cancel`.
- `order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java` maps `POST /orders/{ordNo}/cancel` — no `/api/v1` prefix.
- `order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java` applies its single-origin CORS allowlist to `/api/**` only, which does not match the controller's own mapping.

If no gateway rewrites `/api/v1/orders/...` to `/orders/...`, the customer-app cancel path has been 404-ing since delivery-bff's client was generated (2022-11-08). If a gateway does rewrite it, then the CORS rule guards a path pattern the controller never serves.

**Already checked.** `order-service:src/main/resources/application.yml` sets only `server.port: 8081` and no `server.servlet.context-path`; no reverse-proxy, ingress or gateway config exists anywhere in the order-service repo (full `find . -type f`); a grep across all five repos found `delivery-bff:src/orderClient.ts` as the sole caller.

**Files a human would need.** The gateway/ingress configuration (not in any of the five repos), `sources/context/registry/services.yaml`, `order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java`, `order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java`, `delivery-bff:src/generated/orderApi.ts`.

## SYS-ORDER — Where is order-service's deploy.sh, and what does it deploy?

**Question.** All four CD workflows end with `./deploy.sh $ENVIRONMENT`, and all four build a Docker image first. Neither `deploy.sh` nor a `Dockerfile` exists in the repository. Where do they come from, and what runtime do they target?

**Why it matters.** Without them, nothing in the reef can state how order-service is actually run — replica count, whether Flyway migrates on every pod start, how the DB password reaches the process (no committed yml carries one), or what happens when dev, qa and stage CD all fire on the same push to `develop`.

**Already checked.** Full `find . -type f -not -path "./.git/*"` over order-service — 80 files, no Dockerfile, no deploy.sh, no k8s or helm directory. The same absence is recorded for settlement-batch in [[SYS-SETTLEMENT]]'s known_unknowns, so this is likely an organisation-wide convention rather than an order-team omission.

**Files a human would need.** Whatever repository or runner image supplies `deploy.sh`; `order-service:.github/workflows/order-service-prod-cd.yml` and its dev/qa/stage siblings; `order-service:src/main/resources/application-prod.yml`.

## SYS-SETTLEMENT — Who writes SETTLEMENT_RUN, and does SETTLEMENT_ADJUSTMENT exist?

**Question.** Two tables at the centre of the settlement batch have no writer in any of the five
repositories:
1. `SETTLEMENT_RUN` is created in `V1__settlement_schema.sql` and read twice at runtime
   (`SettlementItemWriter.currentRunId()` uses `SELECT MAX(RUN_ID) ... WHERE SANGTAE='RUNNING'`,
   `MarkSettledTasklet` uses a bare `SELECT MAX(RUN_ID)`), but nothing inserts or updates it.
   Is there an out-of-repo script, a DBA cron, or a manual INSERT that opens and closes a run?
2. `SETTLEMENT_ADJUSTMENT` is inserted into by `CancelReconciler` but is created by no migration
   anywhere. Does the table exist in production?

**Why it matters.** If nobody opens a `RUNNING` run, `currentRunId()` returns null and every
`SETTLEMENT_DTL` insert carries a null RUN_ID — or the batch fails outright. And if
`SETTLEMENT_ADJUSTMENT` does not exist, then even scheduling `CancelReconciler` (the fix implied by
its own 2023-04-24 TODO) would fail on the first row, which changes the remediation plan for the
4,127-row / 188,851,520 KRW backlog.

**What was already checked.** `grep -rn 'SETTLEMENT_RUN'` and `grep -rn 'SETTLEMENT_ADJUSTMENT'`
across all five repos on 2026-09-19 — only reads and the one INSERT, no DDL and no writer. The
2025-03 handover reports the same doubt independently: `SETTLEMENT_ADJUSTMENT 테이블이 문서에는
나오는데 실제로 조회가 안 됨. WIP` ("the SETTLEMENT_ADJUSTMENT table appears in the documents but
cannot actually be queried"). The `deploy.sh` invoked by all three CD workflows is absent from the
repo, so an out-of-repo provisioning step cannot be ruled out from source.

**Paths a human would need.**
- `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`
- `sellflow-reef/sources/context/handover/2025-03_정산팀_인수인계.md` (§3)
## SYS-SETTLEMENT-ANOMALY — Does settlement-anomaly actually run in production, and what triggers it?

**Question.** Is `POST /detect` on settlement-anomaly called by anything today, and if so by what? The
README says "정산 배치 종료 후 (매일 03:00) 트리거된다." ("it is triggered after the settlement batch
ends, daily at 03:00"), but no scheduler or caller exists in any repository.

**Why it matters.** If nothing calls it, 셀플로우 has been carrying a service (and a 7-person team's
listed responsibility, "이상 정산 탐지 모델 운영") that produces no rows — and the CANCELLED_SETTLED
rule is the one automated check that would independently surface the 4,127-row / 188,851,520 KRW
cancel-reconciliation backlog. If something does call it, that trigger is unversioned and invisible.

**What I already checked.** settlement-batch `QuartzConfig.java` registers exactly two jobs
(dailySettlementQuartzJob, cron `0 0 2 * * ?` Asia/Seoul; orderEventRelayJob, every 10 minutes) and
neither performs HTTP. grep for `anomal` and for `8090` across all five repos returns hits only inside
settlement-anomaly itself. The repo has no Dockerfile, no CI workflow, no crontab and no deploy script.
`sources/infra/settlement/queues.md` already files the endpoint under "Unscheduled / externally triggered".

**Files a human would need.** `settlement-anomaly/README.md`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`, plus whatever
lives outside the repos: the host crontab, the Airflow/Jenkins instance, or the deployment manifest
for settlement-anomaly.

## SYS-SETTLEMENT-ANOMALY — Who is meant to act on a SETTLEMENT_ANOMALY row? (Q-025)

**Question.** After a row is inserted with `STATUS='DETECTED'`, who reviews it, through what screen or
query, and who writes `REVIEWED_BY` / `REVIEWED_DTM`?

**Why it matters.** The table was designed with a review workflow (a `(STATUS, DETECTED_DTM)` index and
two reviewer columns) that no code implements. Either the detections are being read some other way and
the reef should record it, or every row ever written is still sitting at DETECTED.

**What I already checked.** No SQL in any of the five repos mentions `SETTLEMENT_ANOMALY` outside
settlement-anomaly's own INSERT. `services.yaml` records the gap itself: "이상 탐지만 수행. 조치 주체는
정의되어 있지 않음. TODO" ("performs detection only; the party responsible for action is not defined").
The README puts remediation out of scope. The 2026 settlement-correction automation plan
(PLAN-2026-014), written by this service's own team lead 윤서진, describes an As-Is flow with no
detection step and never mentions the service.

**Files a human would need.** `settlement-anomaly/sql/V1__anomaly_schema.sql`,
`sources/context/registry/services.yaml`, `sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md`,
and any BI/dashboard tool outside the repos that queries the table.

## SYS-SETTLEMENT-ANOMALY — Where does iforest_v3.pkl come from, and when was it last retrained? (Q-026)

**Question.** Who produced `model/artifacts/iforest_v3.pkl`, from what data, what does its `version`
string say, and has the model been retrained since 2024-11?

**Why it matters.** `app/main.py` loads the pickle at import scope, so the artefact is a startup
dependency; it is not in the repo and nothing in the repo produces it. Versioning is entirely inside
the pickle — swapping the file silently changes production behaviour with no audit trail. The declared
feature list in `features.py` (four features) does not match what `_amount_score` actually scores (raw
`jungsan_amt` and `susuryo`), so it is not even clear the deployed model matches the code around it.

**What I already checked.** The whole repo is ten files; there is no training script, notebook, DVC
pointer, Makefile or CI workflow. `.env.template` sets `MODEL_PATH=./model/detector.pkl`, a different
filename that no code reads. `features.py` says "2024-11 이후 재학습 이력 없음." ("no retraining record
since 2024-11") and carries "TODO(지우) 2025-02-10: cancel_after_settle_flag 가 사실상 단독으로 결과를
좌우한다." ("cancel_after_settle_flag effectively determines the outcome on its own").

**Files a human would need.** `settlement-anomaly/model/detector.py`, `model/features.py`,
`app/main.py`, `.env.template`, and the 데이터팀's model-training repository or storage bucket, which is
not among the five repos.

## SYS-DELIVERY — Who actually owns delivery-bff?

**Question.** Is delivery-bff owned by 커머스본부 물류팀 (이지훈), and if so why does the service registry
still say `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` ("transfer to the logistics team
under discussion since 2025-11, not confirmed")? Separately: does 물류운영본부 배송관리팀 (권나래), which
owns 배송사 관리 · 라스트마일 운영 (carrier management and last-mile operations) but no system, have any
operational stake in this service?

**Why it matters.** The service silently converts carrier outages into a fabricated `PREPARING` status with
no alerting ("3회 실패 시 포기한다. 별도 알림은 없다."). Nobody can be asked to fix that until ownership is
settled. It also blocks assigning SF-4901 (the unstarted regeneration of the 2022 order-service client).

**Already checked.** `sources/context/registry/services.yaml` (TODO, `last_reviewed: 2026-03-02`, no updates
since); `sources/context/org-chart.md` — 조직도 sheet assigns delivery-bff to 커머스본부 물류팀, and the
변경이력 sheet contains no row about any delivery-bff transfer; `delivery-bff:README.md` and
`delivery-bff:package.json` both claim "담당: 커머스본부 물류팀".

**Paths a human would need.** sellflow-reef/sources/context/registry/services.yaml,
sellflow-reef/sources/context/org-chart.xlsx, sellflow/repos/delivery-bff/README.md,
sellflow/repos/delivery-bff/package.json

## SYS-DELIVERY — Where is SF-4901, and has requestCancel ever run?

**Question.** `delivery-bff:src/orderClient.ts` says "재생성은 SF-4901 에서 다루기로 함. (미착수)"
(regeneration will be handled in SF-4901 — not started). SF-4901 is not in `sources/context/tickets/`.
Does it exist, and is it still open? Related: has `requestCancel` ever been called in production? It has no
caller in this repo, its path `/api/v1/orders/{ordNo}/cancel` matches neither the 2022 spec nor today's
`OrderController` (`/orders/{ordNo}/cancel`), and its `fetch` uses a relative URL with no base, which cannot
resolve in a Node process.

**Why it matters.** If it has never run, the safe fix is deletion, not regeneration — and the risk rating in
RISK-DELIVERY changes accordingly. If something outside these five repos calls it, that caller is silently
broken and its cancellations are not reaching order-service.

**Already checked.** Full read of delivery-bff (no internal caller); diffed `src/generated/orderApi.ts`
against `sources/raw/specs/order-service-openapi.json` (v2.4.0, 2022-11-04) and
`sources/apis/order/openapi.code-derived.json` (v2.8.14) and against
`order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java`; searched
`sources/context/tickets/` for SF-4901.

**Paths a human would need.** sellflow/repos/delivery-bff/src/orderClient.ts,
sellflow/repos/delivery-bff/src/generated/orderApi.ts,
sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/controller/OrderController.java

## PROC-INVENTORY-STOCK-ITEM-LIFECYCLE — who creates stock_item rows, and was reserved_qty ever live?

**Question.** Two parts. (1) What process inserts a `stock_item` row? No `INSERT INTO stock_item`
exists in any of the five sellflow repos, yet the restock handler assumes rows are present.
(2) Was `reserved_qty` ever written by anything — an earlier version of inventory-api, a WMS, a
DBA job — or has it been inert since the table was created in `sql/V1__stock.sql`?

**Why it matters.** `GET /stock/{sku}` returns `reserved_qty` to callers as though it were a live
hold. If nothing has ever maintained it, every consumer of that field is reading a stale constant.
And if row creation happens outside the workspace, a restock for a SKU that was never loaded
updates zero rows and still answers `{"restocked": true}` — a silent no-op that nobody can detect
from the response, the logs or `updated_at`.

**Already checked.** Recursive grep across `order-service`, `settlement-batch`, `inventory-api`,
`delivery-bff` and `settlement-anomaly` for `reserved_qty`, `reserve`, `INSERT INTO stock` and
`available_qty`: hits only in `inventory-api/sql/V1__stock.sql`, `inventory-api/app/main.py` (one
SELECT, one UPDATE) and `inventory-api/README.md`. No seed file, no fixture, no admin endpoint, no
Alembic operation on `stock_item`.

**Files a human would need.** `inventory-api/sql/V1__stock.sql`, `inventory-api/app/main.py`,
`inventory-api/app/db.py`, `inventory-api/README.md`; plus whatever loads inventory data outside
this workspace. Owner per `sources/context/org-chart.md`: 재고팀 (데이터플랫폼본부), lead 정하늘.

## PROC-INVENTORY-RESTORE-LOG-LIFECYCLE — does RESTORE_LOG exist in any live database?

**Question.** Does a `RESTORE_LOG` table exist in any dev/stg/prod database, and if so, what
columns does it have and who created it? Related: what did Alembic revision `1c22` do, and what
was `8ba1` ("add restore log reason i", 2024-09-02) meant to add before it was committed with
`pass` in both `upgrade()` and `downgrade()`?

**Why it matters.** The source cannot answer it. `3f9a` calls `op.create_table("RESTORE_LOG")`
with no columns, in a module that never imports `op`; `alembic/env.py` is a two-line stub with no
migration runner; and `down_revision = "1c22"` names a revision with no file, so the chain cannot
resolve. If a table nevertheless exists, it was made by hand and its shape is undocumented — which
changes the advice given to anyone auditing a restock from "no audit trail exists" to "an
undocumented one may".

**Already checked.** Both revision files and `alembic/env.py` read in full; `alembic.ini` targets
`mysql://inventory:@localhost:3306/inventory` while `app/db.py` connects to `sellflow_order`.
Workspace-wide grep for `RESTORE_LOG|RestoreLog|restore_log` returns three hits, all definitional
(the unused dataclass plus `3f9a`'s create/drop). Grep for `1c22` returns only `3f9a`'s own two
references. No CI workflow and no `alembic_version` dump exists in the repo.

**Files a human would need.** `inventory-api/alembic/versions/*.py`, `inventory-api/alembic/env.py`,
`inventory-api/alembic.ini`, `inventory-api/app/models.py`; plus a live MySQL connection
(`SHOW CREATE TABLE RESTORE_LOG;` and `SELECT * FROM alembic_version;`) in each environment.

## PROC-SELLFLOW-OWNERSHIP — Did the delivery-bff transfer to 물류팀 ever complete?

**Question.** `sources/context/registry/services.yaml` records `owner_team: TODO   # 물류팀 이관 논의 중
(2025-11~), 확정 전` ("transfer to 물류팀 under discussion since 2025-11, not yet confirmed"), while
`sources/context/org-chart.md`, `delivery-bff/README.md` and `delivery-bff/package.json` all assert
"담당: 커머스본부 물류팀" with no hedge. Which is current — and separately, who on 물류팀 actually
maintains the service today?

**Why it matters.** `delivery-bff` has no CODEOWNERS, so a pull request against it needs no named
reviewer. RISK-DELIVERY's open items (the uncallable `requestCancel`, SF-4901) have no one to route
to. If the transfer never completed, the service is unowned in the record that claims to be the
single standard for ownership.

**Already checked.** Full read of the registry (its `last_reviewed` is 2026-03-02 with the comment
"이후 갱신 없음" — no updates since); both sheets of the org chart, including 변경이력, which contains
no delivery-related row (last entries 2025-09-01 and 2026-06-15); `find` for CODEOWNERS across all
five repos (only order-service has one); a grep for 이지훈 (물류팀 팀장) across every file in
sources/ and repos/ — he appears in `org-chart.md` and nowhere else; `sources/context/tickets/`
(contains only SF-2287); the 2026-S17 sprint records; and the Slack export. No person from 물류팀
appears in any ticket, sprint item, meeting, incident, source comment or chat message.

**Paths a human would need.** sellflow-reef/sources/context/registry/services.yaml,
sellflow-reef/sources/context/org-chart.md, sellflow-reef/sources/context/org-chart.xlsx,
sellflow/repos/delivery-bff/README.md, sellflow/repos/delivery-bff/package.json

## PROC-SELLFLOW-OWNERSHIP — Who is meant to own the automation of the cancel-reconciliation drain?

**Question.** Every segment of the post-cancellation flow has an assigned owner except one: registering
`CancelReconciler` on a schedule. Its code TODO reads `TODO Quartz 스케줄 등록 필요 (박성민님 확인 후
QuartzConfig 에 추가 예정) - 2023-04-24 김도윤` — blocked on confirmation from a person in 주문팀, in a
repository owned by 정산팀. SF-4512, created the same day by 김도윤, has been `To Do`, `Low`, and
**unassigned** ever since. Did 박성민 ever give that confirmation? And whose backlog should SF-4512
sit on — 주문팀's or 정산팀's?

**Why it matters.** This is the single unowned segment in the chain, and it is the one that would have
stopped `CANCEL_RECON_QUEUE` growing to 4,127 rows / 188,851,520 KRW. The 2026-06-18 kickoff recorded
that the two leads understand the boundary differently — the minutes say so verbatim: "(이 부분 서로
인지가 다름. 확인 필요)". The action item raised to settle it (`이벤트 이후 구간 담당 확인 — 박성민 /
김도윤`) is unchecked with no due date. Until someone owns it, neither the 2026 automation plan nor
the finance division's liability question has an addressee.

**Already checked.** SF-2287's full comment thread (the compensation offered by 김도윤 2023-04-07 and
the consumer promised 2023-04-11); `CancelReconciler.java` and `QuartzConfig` (only
`dailySettlementJobDetail` and `orderEventRelayJobDetail` are registered); the full Slack export
2023-04 to 2026-08 — 김도윤 restates the same blocker on 2023-04-24 10:31 and nothing records the
confirmation arriving, though the export is keyword-filtered on 정정/차감 and is not a full archive;
`tickets_2026-S17.csv` (SF-4512 row, Assignee column empty); policy v0.3 §4 and v1.1 §2 (neither
assigns the scheduling to anyone — both describe a manual procedure owned by 정산팀); the 2025-03
handover §3 and §6 (the question was escalated to the dev team in 2025-02 and never answered, then
handed over unresolved); the 2025-07-12 retrospective ("2025-07-15 제기, 이후 논의 없음",
"티켓 번호 미확인"); and the 2026-08-24 finance mail, which asks the same question of 김도윤 alone.

**Paths a human would need.**
sellflow/repos/settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java,
sellflow/repos/settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java,
sellflow-reef/sources/context/sprints/tickets_2026-S17.csv,
sellflow-reef/sources/context/tickets/SF-2287.md,
sellflow-reef/sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md,
sellflow-reef/sources/context/policy/정산_정정_업무절차_v1.1.md

## PROC-SELLFLOW-OWNERSHIP — Two registry gaps: @sellflow/dba and eta-predictor

**Question.** (a) What team is the GitHub group `@sellflow/dba` — a required reviewer on
`order-service`'s migrations — in org-chart terms? No DBA team exists among the sixteen teams listed.
(b) `eta-predictor` is assigned to 데이터팀 by the org chart but has no entry in `services.yaml` and no
repository. Where does it live, and who owns its repo?

**Why it matters.** (a) Migration review is the one control that would catch a schema change in the
shared `sellflow_order` instance, and the corpus cannot say which humans are on the hook for it —
handover §4 routes DB access to 인프라팀 and the export README routes re-extraction to a DBA, which may
or may not be the same group. (b) The registry states it is the single standard for services; a
system the org chart operates but the registry does not list means the registry cannot be used as a
completeness check.

**Already checked.** `order-service/.github/CODEOWNERS` (all three lines); both org-chart sheets;
`services.yaml` in full; a grep for `eta-predictor` across sources/ and all five repos — it appears
only in `org-chart.md`; `sources/context/handover/2025-03_정산팀_인수인계.md` §4;
`sources/raw/exports/README.md`.

**Paths a human would need.** sellflow/repos/order-service/.github/CODEOWNERS,
sellflow-reef/sources/context/org-chart.md, sellflow-reef/sources/context/registry/services.yaml,
sellflow-reef/sources/raw/exports/README.md

## API-ORDER — Is there a gateway, and does it terminate the spec's bearerAuth?

**Question.** The 2022 OpenAPI spec declares a global `bearerAuth` (HTTP bearer / JWT) that applies to all
32 documented operations, and `WebConfig` carries the comment "앱은 게이트웨이를 통해 들어온다."
("the app comes in through the gateway"). Does that gateway exist today, does it validate a JWT, and does
it cover `POST /orders/{ordNo}/cancel` — the one endpoint in this service that changes state? Relatedly,
what role model do the eight `403` responses the spec declares for `/admin/*` correspond to?

**Why it matters.** This is question Q-009 and it cannot be closed from the code. If no gateway enforces
anything, an unauthenticated request can cancel any order by order number and set off the whole
cancel→outbox→CANCEL_RECON_QUEUE chain. The answer changes the severity in RISK-ORDER and the framing of
all four `proc-*-auth` artifacts from "not visible" to either "enforced upstream" or "not enforced at all".

**Already checked.** `order-service:build.gradle` (four dependencies, no spring-boot-starter-security);
every class under `src/main/java/kr/co/sellflow/order/config/` (only CORS and JPA config);
`grep -rniE 'authorization|bearer|jwt|SecurityConfig|spring-security|OncePerRequest|HandlerInterceptor|addFilter'`
across all five repos — zero matches; `delivery-bff:src/generated/orderApi.ts`, whose only request header is
`Content-Type`; `sources/context/registry/services.yaml`, which advertises
`https://api.sellflow.co.kr/orders` but names no gateway service; all five `application*.yml` profiles.

**Paths a human would need.** sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/config/WebConfig.java,
sellflow/repos/order-service/build.gradle, sellflow-reef/sources/raw/specs/order-service-openapi.json,
sellflow-reef/sources/context/registry/services.yaml

## SCH-ORDER — Who is supposed to write ORDER_CANCEL.CHNL_CD?

**Question.** `CHNL_CD` was added in V17 (column comment: `'취소 접수 채널 (APP/ADMIN/CS)'` — cancel
intake channel) and backfilled in V17/V18 in 2024. Since then nothing writes it: a grep for `CHNL_CD` and
`chnlCd` across all five repos matches only those two migration files, and the `OrderCancel` entity does not
map the field, so every row `OrderCancelService` has inserted since the backfill should carry NULL. Was a
writer planned and never shipped, or is the column populated by something outside these repos?

**Why it matters.** The 2026 automation plan and the correction procedure both reason about where a cancel
came from. If `CHNL_CD` is NULL for every recent row, channel cannot be used to triage the
CANCEL_RECON_QUEUE backlog, and any analysis segmented by channel is describing 2023 data only. It also
decides whether the column should be dropped or wired up.

**Already checked.** `order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java` (fields:
ordNo, chwisoIlsi, chwisoSayuCd, choriSangtae, bigo — no chnlCd);
`order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`;
`order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java` (its INSERT omits the
column too); V17 and V18 under `src/main/resources/db/migration/`; a repo-wide grep across order-service,
settlement-batch, inventory-api, delivery-bff and settlement-anomaly.

**Paths a human would need.** sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/domain/OrderCancel.java,
sellflow/repos/order-service/src/main/resources/db/migration/V17__add_cancel_channel.sql,
sellflow/repos/order-service/src/main/resources/db/migration/V18__backfill_cancel_channel.sql

## API-INVENTORY — Is stock restoration on cancellation done by hand?

**Question.** On the evidence in the five repos, nothing restores stock when an order is cancelled.
`POST /stock/restock` exists and works; `InventoryClient.restore` targets a URL
(`/inventory/restore?ordNo=..&reason=..`) that no router defines; and `InventoryClient` itself has no
caller anywhere in order-service — `OrderCancelService.cancel` saves the cancel row, flips the status,
publishes an event and stops. So: does stock in fact come back after a cancellation, and if so, by what
mechanism — a job outside these repos, a CS admin tool, or a manual correction by the 재고팀?

**Why it matters.** It decides whether this is a dormant defect or a live one. If nothing restores stock,
then every cancellation with reason 01 or 02 since the feature was written has left inventory understated,
and the number is cumulative and unmeasured. If a human does it, that work is invisible to every system and
belongs in the ownership picture alongside the CANCEL_RECON_QUEUE backlog. Either answer changes what
[[CON-ORDER-INVENTORY]] and [[RISK-INVENTORY]] should say. It also determines whether wiring the call up is
a fix or a double-count.

**Already checked.** `inventory-api:app/main.py` and `app/routers.py` (the two mounted routes, the two
unmounted ones, and no `include_router` call anywhere); `order-service:.../client/InventoryClient.java`;
`order-service:.../service/OrderCancelService.java`; a grep for `InventoryClient` / `inventoryClient`
across order-service, which matches only the class declaration and the method definition; the cancellation
policy wiki page, which says only "취소 시 재고 복원은 inventory-api 가 처리합니다" ("stock restoration on
cancellation is handled by inventory-api") and "재고팀과 협의가 필요하다" ("coordination with the inventory
team is required"), naming no mechanism; `sources/context/business-rules.md`, which marks 재고 복원 O for
codes 01, 02 and 04. No inventory-team runbook exists in the reef.

**Paths a human would need.** sellflow/repos/inventory-api/app/main.py,
sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java,
sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java,
sellflow-reef/sources/raw/confluence-snapshots/주문-취소-정책_48213.html,
sellflow-reef/sources/context/business-rules.md

## SCH-INVENTORY — Does RESTORE_LOG exist in production, and what defined revision 1c22?

**Question.** The Alembic chain in inventory-api cannot run: revision `3f9a` declares
`down_revision = "1c22"` and no file in `alembic/versions/` defines `1c22`; `3f9a`'s body calls `op`
without importing it (silenced with `# noqa: F821`), which would raise `NameError`; `alembic/env.py` is a
single import line with none of Alembic's template contents; and `alembic.ini` points at
`mysql://inventory:@localhost:3306/inventory` while `app/db.py` opens `sellflow_order`. So does the
`RESTORE_LOG` table physically exist in any environment — and if it does, what are its columns, given that
no file in the repository ever declared any?

**Why it matters.** `RESTORE_LOG` is the only thing in the design that would evidence a restock ever
happened. `stock_item.updated_at` has no `ON UPDATE CURRENT_TIMESTAMP`, so it still holds insert time, and
the restock write has no idempotency guard — which means that without this table there is no way to audit,
reconcile or even detect a double restock after the fact. Whether the table exists also determines whether
the missing `1c22` revision was lost from version control (a repository-integrity problem worth chasing) or
never existed (in which case the chain was broken from the first commit).

**Already checked.** `inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py` and
`20240902_0931-8ba1_add_restore_log_reason_i.py` — the complete contents of `alembic/versions/`, two files;
`alembic/env.py`; `alembic.ini`; `sql/V1__stock.sql`, which creates only `stock_item` and never mentions
`RESTORE_LOG`; `app/models.py`, whose unused `RestoreLog` dataclass (`ord_no`, `sku_cd`, `qty`, `sayu_cd`,
`result`) is the only field list ever proposed; a grep confirming no code reads or writes the table.

**Paths a human would need.** sellflow/repos/inventory-api/alembic/versions/,
sellflow/repos/inventory-api/alembic/env.py, sellflow/repos/inventory-api/alembic.ini,
sellflow/repos/inventory-api/app/models.py — plus the actual MySQL instance, which is the only place the
answer can come from.

## API-DELIVERY — Which carrier host is delivery-bff actually calling, and does that API need a key?

**Question.** `src/deliveryStatus.ts` reads `process.env.CARRIER_API` and falls back to
`https://api.carrier.example` — a placeholder domain. `.env.template` defines a *differently named*
variable, `CARRIER_API_BASE=https://api.carrier.example.co.kr`. If deployment follows the template, the code
reads nothing and every carrier call goes to the placeholder, fails, and returns the fabricated
`PREPARING`/`stale: true` body with HTTP 200 and no alert. So: what is `CARRIER_API` set to in each
environment, and does the real carrier API require an API key or token — because the code attaches no
headers at all?

**Why it matters.** If the variable is unset in production, the delivery lookup endpoint has been answering
every request with a fabricated status, for every order, indefinitely — and by design it would produce no
alert, no error status code and nothing but three stdout WARN lines to show for it. That is either the most
serious defect in this service or a non-issue, and the two cases are indistinguishable from the repository.
The second half of the question matters independently: a carrier API behind a key would fail the same way
even with the right hostname.

**Already checked.** `delivery-bff:src/deliveryStatus.ts` (the `CARRIER_API` read, the 3000 ms timeout, the
three attempts, the header-less `axios.get`); `delivery-bff:.env.template` (all three variables, two of
which — `ORDER_API_BASE` and `RETRY_COUNT` — are read by no code); `delivery-bff:src/index.ts`;
`delivery-bff:package.json` (no auth or secrets dependency); `sources/apis/delivery/openapi.meta.json`,
which records the same mismatch. No deployment manifest, Helm chart, ConfigMap or CI file exists in the
repo, and the reef's `sources/infra/delivery/` was not able to resolve it either.

**Paths a human would need.** sellflow/repos/delivery-bff/src/deliveryStatus.ts,
sellflow/repos/delivery-bff/.env.template, sellflow-reef/sources/infra/delivery/ — plus the deployment
configuration for delivery-bff in each environment, which is not in any repo here.

## SCH-SETTLEMENT-BATCH — Did migration V4 ever apply, and what is production's real CANCEL_RECON_QUEUE shape?

**Question.** `V4__cancel_recon_queue_index.sql` is `CREATE INDEX IDX_CANCEL_RECON_STATUS ON
CANCEL_RECON_QUEUE (STATUS, REG_DT)`, and `CANCEL_RECON_QUEUE` has no `REG_DT` column — V1 names the
timestamp `RECV_DTM`. MySQL rejects an index on a nonexistent column, which would fail V4 and leave V5 and
V6 unapplied behind it. Yet the 2026-09-01 production export queried `SUM(EXPECTED_AMT)` on the same table,
and no migration in any of the five repos defines an `EXPECTED_AMT` column either. So: what does
`SHOW CREATE TABLE CANCEL_RECON_QUEUE` return in production, and what does `flyway_schema_history` say about
V4, V5 and V6?

**Why it matters.** Three separate claims in this reef depend on the answer. (a) Whether `PROCESSED_AT` and
`PROCESSED_BY` — the V5 columns with no writer — exist at all, or were never applied. (b) Whether the
188,851,520 KRW backlog figure is even computable from the declared schema: it comes from `EXPECTED_AMT`,
a column the migrations do not define, so either production has diverged from version control or the export
ran somewhere else. (c) Whether anyone registering the missing `CancelReconciler` trigger would be writing
into the table they think they are. The reef currently records all three as open, and one DBA query closes
all three.

**Already checked.** All six migrations in `settlement-batch/src/main/resources/db/migration/` read line by
line; every JDBC statement in `settlement-batch` that touches the table (`relay/OrderEventRelayJob.java`,
`recon/CancelReconciler.java`); `sources/raw/exports/README.md`, which records the extraction query and the
note that no non-`PENDING` rows were returned; `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`;
`sources/schemas/settlement/batch/schema.md`, whose tier-4 extraction flags the same V4 defect. No
`flyway_schema_history` dump, no production DDL and no `bin/export-queue.sh` exists in any repo — the export
README itself says the script location is "TBD — 현재는 DBA 에게 요청" ("to be determined; for now, ask the DBA").

**Paths a human would need.** sellflow/repos/settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql,
V4__cancel_recon_queue_index.sql, V5__add_recon_processed_columns.sql,
sellflow-reef/sources/raw/exports/README.md — plus `SHOW CREATE TABLE CANCEL_RECON_QUEUE` and
`SELECT * FROM flyway_schema_history` against the settlement production replica, which only a DBA can run.

## API-SETTLEMENT-BATCH — Is there any supported way to settle one date by hand?

**Question.** `settlement-batch` exposes no HTTP surface, `spring.batch.job.enabled` is `false` so a restart
runs nothing, and all three CD workflows are `workflow_dispatch` with no `inputs:` block — so re-running one
redeploys the service and cannot pass a `jungsanIlja`. The only other date-aware code,
`DateUtil.settlementBaseDate()`, is written for exactly this case ("배치가 02:00 에 돌기 때문에,
00:00~02:00 사이에 수동 실행되면 전전일이 기준일이 되어야 한다" — "because the batch runs at 02:00, a manual
run between 00:00 and 02:00 must use the day before yesterday as the base date") and is called by nothing.
So: when a night's settlement is missed or has to be re-run for a specific date, what does the settlement
team actually do?

**Why it matters.** `DateUtil`'s javadoc is direct evidence that manual runs were expected and designed for.
If a procedure exists — a `deploy.sh` flag, a `java -jar` invocation with `--jungsanIlja=`, a DBA inserting a
`SETTLEMENT_RUN` row by hand — then that procedure, not `QuartzConfig`, is the real operator interface and it
is undocumented. It would also likely answer the reef's longest-standing settlement unknown: who writes
`SETTLEMENT_RUN`, since a manual run would have to create that row too. If no procedure exists, then a missed
night cannot be recovered at all, which is a finding in its own right.

**Already checked.** `settlement-batch/src/main/resources/application.yml` (`spring.batch.job.enabled: false`);
all three `.github/workflows/settlement-batch-*-cd.yml` (no `inputs:`, no `schedule:`, `-x test`, then
`./deploy.sh $ENVIRONMENT`); `SettlementBatchApplication.java` (no CLI argument handling, no
`ApplicationRunner`, no `CommandLineRunner`); `common/DateUtil.java`; `config/QuartzConfig.java`;
`job/DailySettlementQuartzJob.java`; `sources/infra/settlement/runtime.md`, which records that `deploy.sh`
and any Dockerfile are absent from the repo; `sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`,
which describes a duplicate run but not a manual one.

**Paths a human would need.** sellflow/repos/settlement-batch/.github/workflows/settlement-batch-prod-cd.yml,
sellflow/repos/settlement-batch/src/main/resources/application.yml,
sellflow/repos/settlement-batch/src/main/java/kr/co/sellflow/settlement/common/DateUtil.java — plus
`deploy.sh` and whatever operational runbook the 정산팀 keeps outside these repos.

## PROC-ORDER-ORDER-CANCEL-LIFECYCLE — which settlement-batch query referenced ORDER_CANCEL.BIGO in January 2024?

**Question.** `V16__revert_rename_bigo.sql` undoes V15's `BIGO` → `MEMO` rename with the reason
"V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애. 2024-01-18" ("rollback of V15. The
settlement batch query references BIGO directly, causing an outage"). No such query exists in
settlement-batch today. Where was it, and does an equivalent still exist somewhere?

**Why it matters.** If the query lived outside version control — an operations script, a report, a
BI tool, a hand-run SQL — then the set of consumers of order-owned columns is larger than any
repository shows, and no rename of any order column can be reviewed safely. This is the only
attested case of the shared-database coupling causing a production incident, so it is the
strongest evidence available for how risky the next schema change is.

**What I already checked.**
- `grep -rn BIGO` over settlement-batch: no hits at all (only order-service matches).
- `grep -rn "BIGO\|MEMO"` over all five repos: order-service migrations, `OrderCancel`,
  `OrderCancelService`, `OrderController`, and delivery-bff's generated client field `bigo`.
- The archived `#settlement-dev` Slack export (2023-04 to 2026-08) contains no message mentioning
  either column name; the export note says it is a keyword search on 정정/차감, so it may simply not
  cover the incident.
- No runbook or postmortem for 2024-01 exists in `sources/context/runbooks/` — the only postmortem
  there is the 2025-07-12 duplicate-execution one.
- V15 carries no day, only "2024-01", so even the exact rename date is unrecoverable from the files.

**Files a human would need.**
- `order-service/src/main/resources/db/migration/V15__rename_bigo_to_memo.sql`
- `order-service/src/main/resources/db/migration/V16__revert_rename_bigo.sql`
- `order-service/TASK.md` (the cleanup item is still open)
- settlement-batch's git history for 2024-01, and any operations/BI query store outside git

## SCH-ORDER-MIGRATION-HISTORY — were V20, V21, V22 and V24 ever applied, and against what schema?

**Question.** Four migrations reference columns that no migration in the repository creates
(`JEOKLIP_AMT`/`JEOKLIP_RATE`, `SETTLE_REF_NO`, `ORDER_CANCEL.REG_DT`, `ORD_MEMO`), and V18 reads
two more (`ORDER_MST.INFLOW_CHNL`, `ORDER_CANCEL.REG_DT`). Does `flyway_schema_history` in
production show them as successful, and if so, where did those columns come from?

**Why it matters.** It decides whether the repository is the schema's source of truth at all. If
they succeeded, DDL has been applied outside Flyway and every schema-derived artifact in the reef
is a partial picture. If they failed or were repaired by hand, the migration chain has gaps that
the next new environment will hit — and V23, the consolidated baseline meant for exactly that
case, contains no SQL.

**What I already checked.**
- All 24 migration files read line by line; the near-miss twin for each missing name is identified
  (`JEOKRIPGEUM`, `JUNGSAN_RUN_ID`, `CHWISO_ILSI`, `GOGAEK_MEMO`, `CHAENNEL_CD`).
- `application.yml`: `flyway.enabled: true`, `jpa.hibernate.ddl-auto: none` — so Hibernate is not
  creating them either.
- `.github/workflows/ci.yml` runs `./gradlew clean build -x test`; `.github/workflows/migration-issue.yml`
  is `workflow_dispatch` only and runs `flywayInfo`. Neither would catch a failing migration.
- No production schema dump exists anywhere under `sources/`.

**Files a human would need.**
- `SELECT * FROM flyway_schema_history ORDER BY installed_rank` on `sellflow_order` (prod)
- `information_schema.COLUMNS` for the six order tables
- `order-service/src/main/resources/db/migration/V18, V20, V21, V22, V23, V24`

## RISK-ORDER — Does order-service actually compile, and on which JDK does production run?

**Question.** `build.gradle` declares `sourceCompatibility = '1.8'` and all five GitHub workflows pin `actions/setup-java` to `java-version: '8'`, yet three files use `Map.of` / `List.of`, which are Java 9 APIs. Either the declared toolchain is not the one in use, or `./gradlew clean build` has been failing. Which is it, and what JDK does the running production container use?

**Why it matters.** It decides whether the CI workflow has been green or red since the `Map.of` call sites were introduced, and therefore whether *any* automated check has been protecting this service at all — the test step is already commented out, so compilation is the only check left. It also determines whether a contributor can build the project locally from the repo as checked in.

**Already checked.** `build.gradle` (five dependencies, no toolchain block, no `targetCompatibility`); `.github/workflows/{ci,order-service-dev-cd,order-service-qa-cd,order-service-stage-cd,order-service-prod-cd}.yml` (all pin Java 8); grep for `Map.of|List.of|var ` across `src/` → `exception/GlobalExceptionHandler.java` (×2), `mapper/OrderMapper.java`, `search/OrderSearchServiceTest.java`. No build log, no CI badge, no `gradle.properties`, no `.tool-versions` and no `Dockerfile` is committed.

**Files a human would need.** `order-service:build.gradle`, `order-service:.github/workflows/ci.yml`, `order-service:src/main/java/kr/co/sellflow/order/exception/GlobalExceptionHandler.java`, `order-service:src/main/java/kr/co/sellflow/order/mapper/OrderMapper.java`, the GitHub Actions run history for the repo, and the production container image definition.

## RISK-ORDER — Are dev, qa and stage meant to deploy on every push to develop?

**Question.** `order-service-dev-cd.yml`, `order-service-qa-cd.yml` and `order-service-stage-cd.yml` are three separate workflows with identical bodies, all triggered by `push: branches: [develop]`. Is the simultaneous three-environment deploy deliberate, or did one file get copied twice without its trigger being changed?

**Why it matters.** If deliberate, qa and stage can never hold a different build from dev, so there is no staging gate before the same artifact reaches production on a merge to main. If accidental, every merge to develop has been redeploying two environments nobody intended to touch. Either way it should be written down; the service registry does not describe the environment topology.

**Already checked.** All four CD workflow files read in full — same steps, same registry (`registry.sellflow.co.kr`), differing only in the `ENVIRONMENT` env var. No `concurrency` block, no `environment` declaration and no approval gate in any of them. `sources/context/registry/services.yaml` lists no environments for order-service.

**Files a human would need.** `order-service:.github/workflows/order-service-dev-cd.yml`, `-qa-cd.yml`, `-stage-cd.yml`, `-prod-cd.yml`, and whatever `deploy.sh` turns out to be (see the SYS-ORDER question above).

## RISK-ORDER-DISABLED-TESTS — Do order-service tests run anywhere outside this repository, and who owns re-enabling them?

**Question.** The CI test step has been commented out since 2023-05-11 with a TODO signed 박성민. Is there any other place order-service tests are executed — a nightly job, a Jenkins pipeline, a pre-push hook — and who owns turning the CI step back on?

**Why it matters.** It is the difference between "the suite runs somewhere and CI is merely redundant" and "no automated test has run against the only write path in the order domain for three years and four months". That write path emits the `order.cancelled` events behind the 4,127-row, 188,851,520 KRW `CANCEL_RECON_QUEUE` backlog, so the answer changes the severity of both RISK-ORDER-DISABLED-TESTS and RISK-SETTLEMENT-RECON-BACKLOG.

**Already checked.** All five files under `order-service/src/test` read in full (8 test methods; 2 disabled by `@Disabled`, the other 6 never executed because every workflow builds with `-x test`). `codecov.yml` sets a 40% target; `build.gradle` applies no JaCoCo plugin and no workflow uploads coverage, so the gate has no producer. No `.github/workflows` file other than `ci.yml` mentions tests at all, and there is no Jenkinsfile, `Makefile`, or git hook directory in the repository. `TASK.md` lists four open items and enabling the tests is not among them.

**Files a human would need.** `order-service:.github/workflows/ci.yml`, `order-service:codecov.yml`, `order-service:build.gradle`, `order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java`, `order-service:TASK.md`, `sellflow-docs:context/org-chart.md` (for 박성민's current team), and any CI system outside GitHub Actions.

## RISK-INVENTORY — Should cancel reason `04` (배송 실패) restock, and who decided it should not?

**Question.** The 취소정책 sheet marks reason code `04` 배송 실패 (주소불명·수취거부) — delivery
failure, address unknown or receipt refused — as 비용 부담 주체: 파트너, 재고 복원: **O**, with the
note 물류팀 확인 후 처리 ("handled after 물류팀 confirmation"). The code restocks only `01` and
`02`. Is the sheet right and the code wrong, or has the policy changed without the sheet being
updated?

**Why it matters.** If the sheet is right, every cancellation for a failed delivery leaves stock
unreturned while the goods are physically back in the warehouse — a permanent, silent drift
between system and physical stock, on a reason code that is partner-borne and therefore also
affects settlement. Nothing in the system would ever surface the discrepancy: the restock endpoint
returns HTTP 200 `{"restocked": false, "reason": "not_restockable"}` for `04`, identical to the
deliberate decline for `03`, and no audit row is written.

**What I already checked.**
- `inventory-api/app/main.py` — `RESTOCKABLE_REASONS = {"01", "02"}`. The code comment justifies
  only `03`: "03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다". `04` is
  excluded silently, with no comment.
- `inventory-api/app/config.py` — a second copy of the same set, also `{"01", "02"}`, imported only
  by the tests.
- `inventory-api/tests/test_restore.py` — asserts `"03" not in RESTOCKABLE_REASONS` and
  `{"01","02"} <= RESTOCKABLE_REASONS`. Neither test mentions `04`.
- `sources/context/business-rules.md`, 취소정책 sheet — `04` marked 재고 복원 O.
- `sources/raw/confluence-snapshots/주문-취소-정책_48213.html` (2021-03-17, 박성민) — lists `04`
  배송 실패 with 물류팀 확인 후 처리 and no no-restock note. Its §5 says only that restock depends
  on the reason code and 재고팀과 협의가 필요합니다 ("needs to be agreed with 재고팀"). A 2024-08-19
  comment on that page from 강태오 already asks whether the page is still valid.
- `order-service/.../CancelReason.java` — declares all four codes including `BAESONG_SILPAE("04")`,
  with a class comment saying cost-bearing is deliberately not encoded in code.
- Grepped `sources/context/tickets`, `minutes`, `decisions` and `_archive` for `04` / 배송 실패 in a
  restock context — nothing records a decision either way.

**Files a human would need.** `sources/context/business-rules.xlsx` (취소정책 sheet, row 04),
`inventory-api/app/main.py`, `inventory-api/tests/test_restore.py`,
`sources/raw/confluence-snapshots/주문-취소-정책_48213.html`. The people named on those documents
are 정하늘 (재고팀장) and 권나래 (배송관리팀장), with 박성민 (주문팀장) as the policy page's author.

## PROC-DELIVERY-ERROR-HANDLING — Was delivery-bff's "마지막 저장 값" cache ever built, anywhere?

**Question.** delivery-bff's README, the inline comment in `src/index.ts` and the name of the
repository's only test all say the service returns 마지막 저장 값 ("the last stored value") when
the carrier lookup fails. No such store exists in the code. Was it built and removed, was it
built elsewhere (a sidecar, a CDN cache, a gateway), or was it only ever intended?

**Why it matters.** Because of what is returned instead: a hardcoded `{status: 'PREPARING',
stale: true}` with HTTP 200. During a carrier outage every order — including delivered and
cancelled ones — reads as 준비중. If the cache was supposed to exist, this is a regression with a
known intended behaviour and a clear fix. If it never existed, three documents have been
describing an imaginary mechanism since at least 2022, and the fix is a product decision about
what a degraded delivery response should say.

**What I already checked.**
- `delivery-bff/package.json` — dependencies are `axios` and `express` only. No DB driver, no
  redis, no cache library.
- `delivery-bff/src/` (all 105 lines) — no module-level state, no `fs`, no `Map`, no global.
  `syncDeliveryStatus` returns the carrier response or `null`.
- `sources/context/registry/services.yaml` — delivery-bff entry records `db: none`.
- `delivery-bff/tests/deliveryStatus.test.ts` — the test named 재시도 후 마지막 저장 값을
  반환한다 has body `expect(true).toBe(true)`.
- No Dockerfile, no CI workflow, no manifest in the repository, so a sidecar or fronting cache
  cannot be ruled out from source.
- Grepped `sources/context` and `sources/raw` for 배송 / 캐시 / delivery-bff — the only documents
  naming this service are `services.yaml` and `org-chart.md`, neither of which mentions a cache.

**Files a human would need.** `delivery-bff/README.md`, `delivery-bff/src/index.ts`,
`delivery-bff/tests/deliveryStatus.test.ts`, `sources/context/registry/services.yaml`, plus
whatever deployment configuration exists outside the repository. Ownership is itself unresolved —
see the existing SYS-DELIVERY question — so the likely people are 이지훈 (물류팀장) and 권나래
(배송관리팀장).

## PROC-SETTLEMENT-AUTH — How does the database password reach production, and what is in deploy.sh?

**Question.** Neither settlement repo contains a password, secret reference or secret-store client (grep for password/passwd/secret/token/credential across both returns zero hits). `application.yml` sets only `spring.datasource.username`, and `settlement-anomaly/app/db.py` builds a pymysql DSN with no `password` key. Do the production processes connect with an empty password, or does a password arrive via an environment mechanism outside version control? Relatedly, all three CD workflows call `./deploy.sh $ENVIRONMENT`, which is not in the repository and is the only place a deployment credential could live — where does that script live, and does it apply any gate beyond the workflow?

**Why it matters.** The database account is this domain's real principal: it can insert `SETTLEMENT_DTL`, update another team's `ORDER_MST`, and write the Quartz trigger tables that cause a payout run. Whether that account is passwordless, shared across four services, or distinct per service changes the blast radius of every other finding in [[RISK-SETTLEMENT]].

**Already checked.** `settlement-batch/src/main/resources/application.yml`, `settlement-batch/.env.dev`, all three files under `settlement-batch/.github/workflows/`, `settlement-anomaly/app/db.py`, `settlement-anomaly/.env.template`, and `sellflow-reef/sources/infra/settlement/runtime.md`. No Dockerfile or Kubernetes manifest exists in either repo.

**Paths a human would need.** `settlement-batch/.github/workflows/settlement-batch-prod-cd.yml`, wherever `deploy.sh` is kept, and the prod environment's variable configuration.

## RISK-SETTLEMENT — Does SETTLEMENT_ADJUSTMENT exist, and who creates the SETTLEMENT_RUN row?

**Question.** Two tables that the code depends on have no creator anywhere in the five repos. (1) `CancelReconciler` inserts into `SETTLEMENT_ADJUSTMENT`; no migration creates it, and the 2025-03 handover records "`SETTLEMENT_ADJUSTMENT` 테이블이 문서에는 나오는데 실제로 조회가 안 됨" ("the table appears in the documentation but cannot actually be queried"). Does it exist in the production schema? (2) `SettlementItemWriter` takes its `NOT NULL` `RUN_ID` from `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'`, and no code in any repo inserts into `SETTLEMENT_RUN`. What creates that row — a DBA script, an operator, another system?

**Why it matters.** (1) blocks the fix for the largest finding in the domain: scheduling `CancelReconciler` (SF-4512) cannot work if its first INSERT targets a nonexistent table, and 4,127 queued corrections worth 188,851,520 KRW are waiting on it. (2) means the nightly batch's behaviour cannot be reasoned about or safely re-run, because the identity of "the current run" is set by something nobody has documented.

**Already checked.** All six migrations under `settlement-batch/src/main/resources/db/migration/`, `settlement-anomaly/sql/V1__anomaly_schema.sql`, a grep for both table names across all five repos, and `sources/context/handover/2025-03_정산팀_인수인계.md` §3.

**Paths a human would need.** A `SHOW CREATE TABLE SETTLEMENT_ADJUSTMENT` against the production schema, and whatever provisioning or operations scripts sit outside these repos.

## RISK-SETTLEMENT-ANOMALY — Is settlement-anomaly actually running, and where does iforest_v3.pkl come from?

**Question.** `app/main.py` loads `model/artifacts/iforest_v3.pkl` at import scope from a hardcoded path; that file is not in the repository, and no training script, notebook, dataset reference, CI job or model registry exists in any of the five repos to produce or fetch it. There is also no Dockerfile, no CI/CD workflow and no deploy script. Is this service running in any environment? If so, where did the model artefact come from, who retrains it, and what is the deployed Python interpreter (README says 3.9, `services.yaml` says 3.11)?

**Why it matters.** The org chart lists 데이터팀 as operating this model and the registry lists the service as live, but no caller, no scheduler and no consumer for its output exists. If it is not running, 셀플로우 believes it has a settlement safety net that it does not have — for a domain that has already lost roughly 42,000,000 KRW to an undetected duplicate run.

**Already checked.** All ten files of `settlement-anomaly`; `settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`; greps for `/detect`, `anomaly`, `settlement-anomaly` and `8090` across all five repos; `sources/context/registry/services.yaml`; `sources/context/org-chart.md`; `sources/infra/settlement/runtime.md`.

**Paths a human would need.** A `GET http://settlement-anomaly:8090/health` in each environment, a `SELECT COUNT(*) FROM SETTLEMENT_ANOMALY`, and wherever the model artefact is stored.

## RISK-SETTLEMENT-ANOMALY — Why does the 2026 automation plan not mention settlement-anomaly?

**Question.** PLAN-2026-014 (2026-08-18) was written by 윤서진, 데이터팀 팀장 — the team the org chart records as operating `settlement-anomaly` — to automate settlement corrections. Its As-Is flow contains no detection step and the service is never named; the 2026-06-18 kickoff, attended by the 데이터팀 analyst who authored the model's own TODO, concluded that 판정 기준 (decision criteria) do not exist, while `model/detector.py` contains two working rules. Is the omission deliberate scoping, or was the service overlooked?

**Why it matters.** The plan enters its design phase in 2026-10 and a pilot in 2027-01, budgeted on a 월 10건 estimate that the queue export contradicts by roughly two orders of magnitude. If the existing detector is usable, the plan is rebuilding it; if it is not, that should be stated so the plan stops being reviewed against an assumed capability.

**Already checked.** `sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, `sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md`, `sources/context/org-chart.md`, `sources/context/sprints/tickets_2026-S17.csv`, and `settlement-anomaly/model/detector.py`.

**Paths a human would need.** 윤서진 (데이터팀 팀장) and 김도윤 (정산팀 팀장); the Notion original behind PLAN-2026-014.

## PROC-SETTLEMENT-ERROR-HANDLING — What is the batch run-history alerting, and does it fire on failure?

**Question.** The 2025-07-12 postmortem closes an item "배치 실행 이력 알림 추가 — 2025-07-18 완료" ("added batch run-history alerting, completed 2025-07-18") and SF-5099 "정산 배치 실행 이력 알림 개선" was resolved 2026-09-02. No alerting code, dependency or configuration exists anywhere in `settlement-batch` — every failure path ends at an SLF4J log line. What is this alerting, where does it live, and does it notify on a `FAILED` job or only report that a run occurred?

**Why it matters.** A `settlementStep` failure leaves the day half-settled with prior chunks committed, `markSettledStep` never run, and no row anywhere recording the failure (nothing writes `SETTLEMENT_RUN` or `SETTLEMENT_RUN_LOG`). If the alerting only reports successful runs, a silently under-settled day reaches partners before anyone notices — which is how the one recorded incident was found, six and a half hours later, via a partner enquiry.

**Already checked.** All 16 production classes in `settlement-batch`, `build.gradle`, `application.yml`, all three workflows, `sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`, `sources/context/sprints/tickets_2026-S17.csv`.

**Paths a human would need.** SF-5099's implementation (assignee 박성민, 주문팀), and whatever monitoring system holds the alert definition.

## PROC-SETTLEMENT-ANOMALY-LIFECYCLE — Does anyone read SETTLEMENT_ANOMALY, and what actually calls POST /detect?

**Questions**
1. Does any person, dashboard, notebook or scheduled report read `SETTLEMENT_ANOMALY` today? The
   registry says the party responsible for acting on the output is undefined
   ("이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO"), and no code in any of the five repos
   reads the table — but a BI tool or an ad-hoc query outside the repositories would not show up in
   a grep. If someone does read it, who, and what do they do with a row?
2. What invokes `POST /detect` in production? The README claims it is triggered daily at 03:00 after
   the settlement batch, and no scheduler, cron entry, Quartz trigger or HTTP client in any of the
   five repositories calls it. Has it ever run? How many rows does `SETTLEMENT_ANOMALY` hold?
3. Were `DUP_SETTLE` and `FEE_MISMATCH` ever implemented and removed, or only ever documented in the
   README? Both describe defects that are known to be real (the 2025-07-12 duplicate execution, and
   the hardcoded 0.12 fee rate that ignores `PARTNER_CONTRACT`).

**Why it matters**
This is the residue of Q-025 ("what happens to a row in SETTLEMENT_ANOMALY after detection — who acts
on it?"). From code the answer is "nothing", but the more consequential question is whether the table
has any rows at all. `SETTLEMENT_ANOMALY`'s `CANCELLED_SETTLED` rule is the only independent signal of
the same exposure recorded in `CANCEL_RECON_QUEUE` (4,127 rows / 188,851,520 KRW as of 2026-09-01). If
detection has never run, the organisation has one blind control rather than two, and any remediation
plan that treats the detector as a compensating control is mistaken. Feeds
PROC-SETTLEMENT-ANOMALY-LIFECYCLE, RISK-SETTLEMENT-RECON-BACKLOG, API-SETTLEMENT-ANOMALY and
SYS-SETTLEMENT-ANOMALY.

**Already checked**
Every file in `settlement-anomaly` was read (app/main.py, app/db.py, app/schemas.py, model/detector.py,
model/features.py, sql/V1__anomaly_schema.sql, tests/test_detector.py, README.md, .env.template).
`grep -rn "REVIEWED_BY\|REVIEWED_DTM"` across all five repos matches only the CREATE TABLE.
`grep -rn "DUP_SETTLE\|FEE_MISMATCH"` matches only the two README lines. No caller for `/detect` and no
scheduler of any kind exists in the repos; `settlement-batch`'s `QuartzConfig` registers two jobs and
neither is an HTTP call. The org chart puts model operation with 데이터팀 (윤서진), and the registry
leaves the action owner as TODO. A single production query — `SELECT ANOMALY_CD, STATUS, COUNT(*),
MIN(DETECTED_DTM), MAX(DETECTED_DTM) FROM SETTLEMENT_ANOMALY GROUP BY 1, 2` — would answer (2).

**Files a human would need**
- `settlement-anomaly/app/main.py`, `settlement-anomaly/model/detector.py`,
  `settlement-anomaly/sql/V1__anomaly_schema.sql`, `settlement-anomaly/README.md`
- `sellflow-reef/sources/context/registry/services.yaml`, `sellflow-reef/sources/context/org-chart.md`
- Whatever schedules jobs for 데이터팀 services (not present in any of the five repositories)

## PROC-SETTLEMENT-RUN-LIFECYCLE — What creates a SETTLEMENT_RUN row, and what closes it?

**Questions**
1. What inserts a `SETTLEMENT_RUN` row with `SANGTAE='RUNNING'`? No `INSERT` exists in any of the five
   repositories, yet `SettlementItemWriter.currentRunId()` depends on such a row existing and
   `SETTLEMENT_DTL` is demonstrably populated. Is it a DBA script, an admin screen, a job outside these
   repos?
2. What values other than `RUNNING` can `SANGTAE` take, and what sets them? Nothing updates the column
   anywhere in code, and the schema declares no vocabulary.
3. Are `START_DTM`, `END_DTM` and `TOTAL_AMT` ever populated, and by what?
4. Did the 2025-07-12 duplicate execution create one `SETTLEMENT_RUN` row or two? This decides whether
   `SETTLEMENT_DTL`'s `(RUN_ID, ORD_NO)` primary key offers any protection against double settlement.
5. How does a multi-line order settle? The reader emits one item per `ORDER_DTL` line while the target
   table's key is per order, and the writer's `INSERT` has no `ON DUPLICATE KEY` clause — so either
   production orders are effectively single-line, or `settlementStep` fails on multi-line orders.

**Why it matters**
The run row is the identity every payout line hangs off, and it has no creator in the codebase. Until
(1) is answered, nobody can say what would happen if the batch ran with no `RUNNING` row (the writer's
`queryForObject` would return null into a `NOT NULL` primary-key column), nor whether
`MarkSettledTasklet`'s unfiltered `MAX(RUN_ID)` can select a different run from the one just written —
which would mark the wrong orders `JUNGSAN_WANRYO`. (5) determines whether some portion of partner
revenue is silently dropped or whether nightly runs are erroring. Feeds PROC-SETTLEMENT-RUN-LIFECYCLE,
PROC-SETTLEMENT-DAILY-BATCH and SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL.

**Already checked**
`grep -rn "SETTLEMENT_RUN"` across all five repos returns exactly four kinds of match: the `CREATE TABLE`
in V1, the V2/V6 `SETTLEMENT_RUN_LOG` migrations, two `SELECT MAX(RUN_ID)` queries
(`SettlementItemWriter` line 43, `MarkSettledTasklet` line 31), and one `JOIN` in
`settlement-anomaly/app/main.py` line 33. No `INSERT`, no `UPDATE`. `DailySettlementJobConfig` declares
no `JobExecutionListener` or `StepExecutionListener`. `SettlementReportWriter`, which takes a
`String runId`, has no caller. The 2025-07-12 postmortem describes the duplicate run but does not say
how many run rows resulted.

**Files a human would need**
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- Whatever operational script or admin tool opens a settlement run (not in any of the five repos)

## PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE — Has any queue row ever been processed, and does SETTLEMENT_ADJUSTMENT exist?

**Questions**
1. Run `SELECT STATUS, COUNT(*), MIN(RECV_DTM), MAX(PROCESSED_DTM) FROM CANCEL_RECON_QUEUE GROUP BY
   STATUS`. Has any row ever left `PENDING`? The 2026-09-01 export filtered on `STATUS='PENDING'`, so
   it cannot answer this, and the code that would set `PROCESSED` has never been scheduled.
2. Does `SETTLEMENT_ADJUSTMENT` exist as a physical table, and with what columns? No migration in any of
   the five repos creates it; `CancelReconciler` inserts three columns into it; the 2025-03 handover
   says it cannot be queried.
3. Does `CANCEL_RECON_QUEUE` have an `EXPECTED_AMT` column in production? The export's recorded query
   sums it and no migration defines it. If it exists, how is it populated — the relay does not write it.
4. Did migration V4 apply cleanly? It creates an index on `(STATUS, REG_DT)` and the table has no
   `REG_DT` column (V1 defines `RECV_DTM`).

**Why it matters**
(1) and (2) together decide whether the 188,851,520 KRW backlog is "never drained" or "partly drained
by hand". (2) in particular decides whether scheduling `CancelReconciler` — the fix tracked as SF-4512,
`To Do` and unassigned since 2023-04-24 — would work at all or would fail on its first statement.
(3) governs whether the backlog amount is a real sum or, as the file's arithmetic suggests (every
monthly amount is exactly count × 45,760), a flat per-case estimate. Feeds
PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE, PROC-SETTLEMENT-CORRECTION and
RISK-SETTLEMENT-RECON-BACKLOG.

**Already checked**
All six `settlement-batch` migrations read in full. `grep -rn "CANCEL_RECON_QUEUE"` across five repos:
one `INSERT` (`OrderEventRelayJob`), one `SELECT` and one `UPDATE` (`CancelReconciler`), plus the
migrations and the README. `grep -rn "CancelReconciler\|reconcileCancellations"` finds no caller.
`grep -rn "SETTLEMENT_ADJUSTMENT"` finds only the `INSERT` in `CancelReconciler`.
`grep -rn "EXPECTED_AMT"` finds nothing. The export README and CSV were read in full, and the
per-month amounts were recomputed (4,127 rows, 188,851,520 KRW, 41 months, all divisible by 45,760).

**Files a human would need**
- `settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`
- `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`, `V4__cancel_recon_queue_index.sql`, `V5__add_recon_processed_columns.sql`
- `sellflow-reef/sources/raw/exports/README.md` and `cancel_recon_queue_monthly_20260901.csv`
- Production `settlement_prod` read replica (the DBA runs the extraction; no self-service script exists)

## CON-ORDER-SETTLEMENT — which settlement-batch query referenced ORDER_CANCEL.BIGO?

**Question.** In January 2024, `order-service` migration `V15__rename_bigo_to_memo.sql` renamed `ORDER_CANCEL.BIGO` to `MEMO`, and `V16__revert_rename_bigo.sql` (2024-01-18) reverted it with the comment "V15 롤백. 정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애." ("rollback of V15. A settlement batch query referenced BIGO directly and caused an incident"). Which query was that, and does it still run?

**Why it matters.** This is the only recorded production incident caused by the undeclared Order↔Settlement database coupling, and it is the precedent for what a future column rename on the Order side would do. If the offending query still exists outside the repo, `ORDER_CANCEL` is still frozen against renames and nobody has written that down. If it was removed, the constraint has lifted and the Order team does not know that either.

**What I already checked.** `grep -rn "BIGO" settlement-batch/src/` returns nothing — the current settlement-batch tree has no reference to the column. `grep -rn "ORDER_CANCEL" settlement-batch/src/` returns nothing either. So the query is not in the settlement-batch repository as it stands today. `sources/context/registry/services.yaml` records no dependency between the two services at all. No incident report for 2024-01-18 exists under `sources/context/runbooks/` (the only runbook there covers the 2025-07-12 duplicate-batch incident).

**Paths a human would need.**
- `sellflow/repos/order-service/src/main/resources/db/migration/V15__rename_bigo_to_memo.sql`
- `sellflow/repos/order-service/src/main/resources/db/migration/V16__revert_rename_bigo.sql`
- `sellflow/repos/order-service/src/main/resources/db/migration/V9__order_cancel_bigo_extend.sql`
- `sellflow/repos/settlement-batch/src/main/java/kr/co/sellflow/settlement/` (whole tree)
- any operational/DBA script store outside the five repos — the likeliest home for the query

## CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH — Does 정산팀 know that settlement-anomaly reads SETTLEMENT_DTL?

**Question.** Is anyone in 재무본부 정산팀 aware that `settlement-anomaly` (데이터팀) issues a production
SELECT against `SETTLEMENT_DTL`, `SETTLEMENT_RUN` and `ORDER_MST` on every `POST /detect`? And, relatedly:
is `POST /detect` triggered by anything at all in any environment — the README claims a daily 03:00 run,
but no scheduler, cron entry or caller exists in version control in any of the five repositories?

**Why it matters.** The read is an undocumented cross-team schema dependency on six columns with no FK,
no version, no owner and no test on either side. A rename or drop of `JUNGSAN_AMT`, `SUSURYO`,
`PARTNER_ID`, `RUN_ID`, `ORD_NO` or `JUNGSAN_ILJA` would break the consumer silently, and nothing in
settlement-batch's repository would reveal the dependency during review. The same failure mode already
caused a production incident on the order side (V15/V16, `ORDER_CANCEL.BIGO`, 2024-01-18). Whether the
endpoint runs also determines whether the `CANCELLED_SETTLED` rule counts as existing detection coverage
for the 188,851,520 KRW cancel-after-settlement exposure — which matters directly to the 2026 correction
automation plan.

**What was already checked.** All ten files of `settlement-anomaly`; `QuartzConfig` (two triggers only,
neither for this service); `settlement-batch` has no web starter, no HTTP port and no HTTP client;
`grep -rni "settlement-anomaly|SETTLEMENT_ANOMALY|8090|/detect"` across all five repos matches only inside
`settlement-anomaly/`; `grep -rni "cron|schedul|apscheduler|celery" settlement-anomaly/` returns nothing;
`requirements.txt` pins no scheduler; no Dockerfile, CI workflow or systemd unit exists; `services.yaml`
models no table-level dependency between the two services and its `queues:` block has no entry for this read.

**Files a human would need.** `settlement-anomaly/app/main.py`, `settlement-anomaly/app/db.py`,
`settlement-anomaly/README.md`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`,
`settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`,
`order-service/src/main/resources/db/migration/V16__revert_rename_bigo.sql`,
`sources/context/registry/services.yaml`. Plus, from an environment nobody in version control can reach:
`GET :8090/health` per environment and `SELECT ANOMALY_CD, COUNT(*), MAX(DETECTED_DTM) FROM SETTLEMENT_ANOMALY GROUP BY ANOMALY_CD`.

## CON-SELLFLOW-CANCEL-ENTITY-COMPARISON — Has SAYU_CD '00' ever been written, and what is the real backlog amount?

**Question.** Two things only a human with production access can settle:
1. How many `CANCEL_RECON_QUEUE` rows have `SAYU_CD = '00'`? `'00'` is the hardcoded fallback in
   `OrderEventRelayJob.sayuCd(...)` and appears in no vocabulary (not `CancelReason`, not the
   취소정책 sheet, not `RESTOCKABLE_REASONS`). A single non-zero count means the relay's payload
   parsing has silently failed at least once and those rows have no recoverable reason.
2. What is the actual money value of the 4,127 PENDING rows? The queue has no amount column in any
   migration, and the published 188,851,520 KRW is exactly `4,127 × 45,760` — a flat per-case
   estimate, not a sum. The extraction README records the query as `SUM(EXPECTED_AMT)`, naming a
   column that does not exist.

**Why it matters.** The 2026 automation plan and the RISK-SETTLEMENT-RECON-BACKLOG severity both
rest on that 188M figure. If the true distribution is skewed, the number is wrong in an unknown
direction. And a clawback cannot be computed for a row whose reason code is `'00'`.

**What I already checked.**
- `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`, `V4__cancel_recon_queue_index.sql`,
  `V5__add_recon_processed_columns.sql` — no amount column is ever added.
- `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv` — the in-file query comment shows
  `COUNT(*)` only, no SUM; `2196480/48 = 45760` and `188851520/4127 = 45760` exactly.
- `sources/raw/exports/README.md` — records `SUM(EXPECTED_AMT)` and notes that non-PENDING rows were
  not returned at all.
- No export in this reef breaks the queue down by `SAYU_CD`.

**Files a human would need.** Production read replica `settlement_prod`, table `CANCEL_RECON_QUEUE`;
a join to `SETTLEMENT_DTL.JUNGSAN_AMT` on `ORD_NO` would give the real exposure.

## CON-SELLFLOW-CANCEL-ENTITY-COMPARISON — Does the CANCELLED_SETTLED rule actually run?

**Question.** `settlement-anomaly/app/main.py` queries with `pymysql.cursors.DictCursor` and selects
`m.SANGTAE_CD` unaliased, which yields an uppercase dict key. `model/detector.py` then reads
`r["sangtae_cd"]`, `r["ord_no"]` and `r["jungsan_amt"]` in lowercase. On the source as written this
should raise `KeyError` on the first row. Has `/detect` ever returned successfully in production, and
are there any `SETTLEMENT_ANOMALY` rows with `ANOMALY_CD = 'CANCELLED_SETTLED'`?

**Why it matters.** `CANCELLED_SETTLED` is the only automated detection of the exact situation
SF-2287 created. If it has never produced a row, the safety net documented in the settlement-anomaly
README does not exist. The repo's only test, `tests/test_detector.py`, asserts a feature name and
never calls `detect()`.

**What I already checked.**
- `settlement-anomaly/app/db.py` (DictCursor, database `sellflow_order`), `app/main.py` (the SELECT),
  `model/detector.py` (the lowercase key access), `tests/test_detector.py`.
- Independently, the rule is near-unreachable by timing: `DailySettlementJobConfig` selects only
  `SANGTAE_CD = 'BAESONG_WANRYO'` orders and `MarkSettledTasklet` immediately sets them to
  `JUNGSAN_WANRYO`, so a row can only be in `{CHWISO, BANPUM}` if the cancel lands between the 02:00
  batch and the 03:00 detection run.

**Files a human would need.** `SETTLEMENT_ANOMALY` in `sellflow_order` (`SELECT ANOMALY_CD,
COUNT(*) ... GROUP BY 1`), and the settlement-anomaly service logs for "이상 탐지 완료".

## CON-ORDER-INVENTORY — Is stock ever restored on cancellation, and by what?

**Question.** When an order is cancelled, does anything put the stock back? If so, what — a warehouse
management system outside these five repos, an operator screen, a scheduled job, a person running SQL?
And was the divergence on reason code 04 (배송 실패 / delivery failure) a deliberate later decision or an
oversight?

**Why it matters.** Within the five sampled repos, nothing restores stock on cancellation for any reason
code. `OrderCancelService.cancel` contains no inventory call at all, and `InventoryClient.restore` has no
caller. If no out-of-repo mechanism exists, every cancelled order has been leaving `stock_item` overstated
since the feature was scaffolded — a silent, compounding inventory discrepancy. If a manual mechanism does
exist, the reef should name it rather than record an absence, and `PROC-INVENTORY-RESTOCK` needs rewriting.
Code 04 matters separately because `business-rules.xlsx` says it should restock and the code excludes it;
if that was deliberate, the spreadsheet is wrong and should be corrected before anyone automates against it.

**What I already checked.** Grepped all five repos (order-service, settlement-batch, inventory-api,
delivery-bff, settlement-anomaly) for `InventoryClient`, `restore(`, `/stock/restock` and `stock_item`.
The only writer of `stock_item` anywhere is inventory-api's own `restock` handler, which nothing calls.
`RESTORE_LOG` exists (alembic revision 3f9a, 2023-04-14) with a matching dataclass in `app/models.py`, and
no INSERT against it exists in any repo, so the audit trail cannot answer this either. The 2021 Confluence
policy page says only "재고팀과 협의가 필요합니다" (coordination with the inventory team is required) and
never records the outcome. No inventory-team runbook is in `sources/`.

**Files a human would need.**
- `sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`
- `sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/client/InventoryClient.java`
- `sellflow/repos/inventory-api/app/main.py`, `app/db.py`, `app/config.py`
- `sellflow/repos/inventory-api/alembic/versions/20230414_1120-3f9a_add_restore_log.py`
- `sellflow-reef/sources/context/business-rules.md` (취소정책 sheet, 재고 복원 column)
- `sellflow-reef/sources/raw/confluence-snapshots/주문-취소-정책_48213.html` (section 5)

## CON-ORDER-DELIVERY — Where did delivery-bff's "generated" order client actually come from?

**Question.** `delivery-bff/src/generated/orderApi.ts` states it was generated by
`openapi-typescript-codegen 0.23.0` from `order-service-openapi.json` on 2022-11-08. Three of its
assumptions cannot be produced from that file. Was it hand-edited after generation, or generated from a
different, earlier spec that is not in this reef? Relatedly: what is the scope of SF-4901, the regeneration
ticket the wrapper names as 미착수 (not started)?

**Why it matters.** SF-4901 is scoped as a regeneration. If the file was hand-edited, regenerating it from
`sources/apis/order/openapi.json` will not reproduce it and will not fix it either — the current spec is a
2022 artefact documenting 30 paths of which only 2 still have handlers, and it declares a cancel response
(`OrderCancel`) that the live controller does not return (it returns an empty body). Whoever picks up
SF-4901 needs to know they are writing a client, not regenerating one. It also determines whether the
`/api/v1` prefix was once real on the cancel endpoint, which would change how the reef describes
order-service's URL history.

**What I already checked.** Parsed `sources/apis/order/openapi.json` (the reef's verbatim copy of
`sources/raw/specs/order-service-openapi.json`, the file the header names): the string `/api/v1` occurs 0
times, the string `JUMUN_WANRYO` occurs 0 times, there is no `OrderStatus` schema at all (`OrderMst.sangtaeCd`
is a bare `{"type":"string"}`), and the cancel 200 response is `OrderCancel {ordNo, chwisoIlsi, chwisoSayuCd,
choriSangtae, bigo}` rather than the client's `CancelResponse {ordNo, sangtaeCd}`. Enum emission was working
in that spec (`gyeolJeCd`, `chaenNelCd`, `taekBaeSaCd`, `changgoCd`, `chulGoSangtae` all carry enums), so the
missing status enum is a property of the spec, not of the generator. The output shape also argues for
hand-authorship: openapi-typescript-codegen emits a `core/` directory with `OpenAPI.BASE` and a `request()`
helper, not a single file with an inline `fetch` and a hardcoded relative path. No SF-4901 ticket exists in
`sources/context/tickets/`; only SF-2287 is there. There is no git history in the fixture repos, so the edit
cannot be traced.

**Files a human would need.**
- `sellflow/repos/delivery-bff/src/generated/orderApi.ts` (the provenance header and the three assumptions)
- `sellflow/repos/delivery-bff/src/orderClient.ts` (the SF-4901 reference)
- `sellflow-reef/sources/apis/order/openapi.json` and `sources/apis/order/openapi.meta.json`
- `sellflow-reef/sources/raw/specs/order-service-openapi.json`
- `sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/controller/OrderController.java`
- `sellflow/repos/order-service/src/main/java/kr/co/sellflow/order/search/OrderSearchController.java`
- Jira: SF-4901

## PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11 — When does the five-year retention clock start?

**Question.** 정산_정정_업무절차_v1.1 §5 says "정정 이력은 정산 어드민에 기록하며 5년간 보존한다" ("correction
history is recorded in the settlement admin and retained for five years") without naming the start
event. Does the five years run from (a) the date the correction is performed, (b) the date of the
original settlement, or (c) the date the cancellation was received? And separately: is
`CANCEL_RECON_QUEUE` itself covered by any retention or archival policy?

**Why it matters.** The queue's oldest PENDING rows are from 2023-04. Under readings (b) and (c)
they reach five years in 2028-04; under (a) the clock has never started, because no correction has
ever been performed through the system. If the DBA archives or purges `CANCEL_RECON_QUEUE` rows on
any schedule, the only surviving evidence that a 2023 case was ever registered for deduction would
disappear — and that evidence is what a partner dispute about a 2023 payout would turn on. Nobody
can plan the 2028 decision without knowing which clock is running.

**What I already checked.** Both policy versions (v0.3 has no retention clause at all); the 2025
handover (no retention practice described; corrections tracked in personal spreadsheets); all six
Flyway migrations in settlement-batch (no retention comment on `CANCEL_RECON_QUEUE`, no archive
table); the export README; the sprint records. No DBA or records-management policy document exists
anywhere in `sources/`.

**Files a human would need.** `sources/context/policy/정산_정정_업무절차_v1.1.md` (§5, §6),
`sources/context/handover/2025-03_정산팀_인수인계.md` (§4 access table, §6),
`settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`,
`sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`. The 재무본부장 who approved v1.1, and
whoever owns the 정산 어드민, are the people who can answer.

## PROC-SETTLEMENT-ANOMALY-VS-BATCH — Are settlement-batch and settlement-anomaly one service group?

**Question.** Is there a decision, anywhere outside the code, that these two applications form one
"Settlement" service group — and if so, who owns the boundary between them? The service registry
lists them as two independent services with two owner teams in two different divisions (재무본부
정산팀, 데이터플랫폼본부 데이터팀) and two databases; nothing in either repository references the other
by name; and both services' output tables carry an unfilled `TODO` where the consumer or
remediation owner should be.

**Why it matters.** `settlement-anomaly` reads `SETTLEMENT_DTL` by reproducing, as a Python string
literal, the exact five-column sequence that `settlement-batch` writes as a Java string literal.
Nothing keeps them aligned — no shared DDL, no generated model, no view, no contract test. A V7
migration that renames or reorders a column in `SETTLEMENT_DTL` would be a normal change for 정산팀
and a silent production break for 데이터팀, and no document says whose review gate catches that.

**What I already checked.** Both repositories in full; `sources/context/registry/services.yaml`
(last_reviewed 2026-03-02, "이후 갱신 없음"); the org chart and its change log; all decision
artifacts in the reef; the minutes, plan, handover and Slack export. No grouping decision, no
schema-ownership agreement and no cross-team review process appears in any of them. The only
recorded cross-team data agreement is the 2019 shared-instance note in V1 and the 2019 note in
`MarkSettledTasklet` about writing to the order team's `ORDER_MST`.

**Files a human would need.** `sources/context/registry/services.yaml`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`,
`settlement-anomaly/app/main.py`, `settlement-batch/src/main/resources/db/migration/`. 김도윤 (정산팀)
and 윤서진 (데이터팀) are the two leads.

## DEC-SETTLEMENT-ANOMALY-STANDALONE — Was the split ever decided, and does a rationale exist outside sources/?

**Question.** Does any record exist — a design doc, an architecture review, a 본부-level decision, a
mail thread, a Jira epic — of the choice to build settlement anomaly detection as a separate
FastAPI service under 데이터플랫폼본부 데이터팀 rather than as a step inside 재무본부 정산팀's
settlement batch? And separately: was the choice to integrate it by reading `SETTLEMENT_DTL`
directly, rather than through a view, an API, an event or a shared model, ever considered as a
choice at all?

**Why it matters.** I have written DEC-SETTLEMENT-ANOMALY-STANDALONE as a reconstructed ADR and
said so plainly in its Context, because the decision's consequences are live and expensive even
though its reasoning is unrecorded. If a rationale document exists somewhere outside `sources/`,
the artifact's Rationale section should be replaced with it rather than left as inference. If none
exists, that is itself the answer, and the open question becomes forward-looking: ratify the split
by giving `SETTLEMENT_DTL` a consumer-facing interface that 정산팀's own build breaks when they
violate it, or reverse it by folding rule-based detection back into the batch and leaving only the
IsolationForest outside. Either way someone has to decide, because today a V7 migration renaming a
`SETTLEMENT_DTL` column is a routine change for 정산팀 and a silent break for 데이터팀, with no
foreign key, no view, no contract test and no registry entry that would surface the dependency.

**What I already checked.** The whole `sources/` tree, grepped for `anomaly`, `settlement-anomaly`,
`이상 탐지` and `이상탐지`: the only hits are `context/registry/services.yaml`, `context/org-chart.md`
and the reef's own tier-4 extraction notes. None of the sixteen documents under `sources/context/`
mentions the service — not the 2024 automation review draft, the 2026 automation plan, the
2026-06-18 kickoff minutes, the 2025-03 handover, either version of the correction procedure, the
2025-07-12 postmortem, or the 2026-S17 sprint records. Both repositories in full: settlement-batch
contains no reference to settlement-anomaly in any migration comment, javadoc, README line or test.

**Files a human would need.** `sources/context/registry/services.yaml` (the settlement-anomaly entry
and its `db:` field), `sources/context/org-chart.md` (변경이력 rows 2024-07-01, 2025-06-09,
2025-09-01), `settlement-anomaly/app/main.py` and `app/db.py`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`,
`settlement-batch/build.gradle`. 윤서진 (데이터팀 팀장) and 김도윤 (정산팀 팀장) are the two leads;
whoever chaired the 2025 architecture review, if one happened, is the person who would hold a record.

## PAT-SELLFLOW-ORPHANED-COMPONENTS — Who owns the "전수 점검" of unregistered batches, and was it ever done?

**Question.** The 2025-07-12 incident review lists `정산 관련 배치 전수 점검 (스케줄 등록 여부 포함)` ("full audit
of settlement-related batches, including whether they are registered on a schedule") as an open item with
no owner and no date. That item names, exactly, the failure mode this artifact documents. Was the audit
ever performed? If it was, what did it find, and why is `CancelReconciler` still unregistered fourteen
months later? If it was not, who was supposed to pick it up when the postmortem closed?

**Why it matters.** This is the one moment in the recorded history where the organisation named the
pattern rather than an instance of it. Nineteen code components and thirteen schema objects in the five
repositories have no caller, no writer or no DDL, and the estate has no mechanism that would surface any
of them. Whether a one-off audit was attempted and failed to stick, or was never started, determines
whether the corrective is a process (a recurring check with an owner) or a first attempt. It also
determines whether anyone outside this reef already holds a list of these orphans.

**What I already checked.** The full postmortem, including its 비고 note that a settlement-team request to
check the cancel deduction was split into a separate ticket whose `티켓 번호 미확인` (number unconfirmed);
both sprint records for 2026-S17, which contain no audit item; the 2026-06 kickoff action list, which has
two blank rows; the 2026 automation plan's 선결 과제 list, which does not mention an audit; `QuartzConfig`,
which still declares two jobs; and `TASK.md`, whose consumer-check item is still unticked. No audit
output, checklist or ticket reference appears anywhere in `sources/` or in any repository.

**Files a human would need.** `sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md` (재발 방지
section), `sources/context/sprints/2026-S17_baseline.md` and `_log.md`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`,
`order-service/TASK.md`. 박성민 wrote the postmortem; 김도윤 raised the request that was split out; whoever
runs the 정산 배치 on-call rotation would know if the audit happened.

## PAT-SELLFLOW-DOC-CODE-DRIFT — Has any document here ever been revised because the code changed?

**Question.** Every document revision recorded in this reef was triggered by an organisational event — a
procedure being formalised (v0.3 → v1.1, 2024-02), a project being proposed (the 2024 draft, the 2026
plan), a team being restructured (the org-chart change log). None was triggered by a deployment. Is that
a complete picture, or do revisions prompted by code changes exist outside the material in `sources/`?
And separately: is any person or role accountable for keeping a document current once it is published?

**Why it matters.** The seven drift shapes in this artifact all reduce to one missing mechanism: a code
change cannot find the documents it invalidates. If no such revision has ever happened, the corrective is
to create the trigger — doc ownership metadata, a pull-request field, a link from migrations to policies —
rather than to rewrite the documents, which the organisation has already done twice without effect. If
revisions prompted by code changes do exist and simply are not in this reef, then the mechanism exists and
the problem is narrower than it appears. The answer changes what should be recommended, and no one inside
the artifacts can supply it.

**What I already checked.** Every document under `sources/context/` and `sources/raw/` for revision
markers and their stated causes: the Confluence page (last modified 2021-03-17, one unanswered staleness
comment from 2024-08-19), procedure v1.1's 개정 이력 table (two rows, both organisational), the archived
2024 draft's `superseded_by` pointer, `services.yaml`'s `last_reviewed: 2026-03-02 # 이후 갱신 없음`, the
org-chart 변경이력 (seven rows, all personnel or system launches). Also every repository for documentation
ownership: `order-service/.github/CODEOWNERS` covers `/src/main/resources/db/` and the `legacy/` package
and no documentation path; the pull-request template asks nothing about documents; no repository contains
a doc-ownership file.

**Files a human would need.** `sources/raw/confluence-snapshots/주문-취소-정책_48213.html` (page metadata and
comment thread), `sources/context/policy/정산_정정_업무절차_v1.1.md` (§7), `sources/context/registry/services.yaml`,
`sources/context/org-chart.md` (변경이력 sheet), `order-service/.github/CODEOWNERS`,
`order-service/.github/pull_request_template.md`. The Confluence space administrator for 커머스개발 can show
the real page history; 박성민 owns the page and 재무본부장 approved the procedure.

## GLOSSARY-SELLFLOW — Are 파트너 and 셀러 the same population, and is PARTNER_ID one identifier space?

**Question.** Does 파트너 (partner) name the same people as 셀러 (seller)? And are the four `PARTNER_ID`
declarations in the estate — `ORDER_DTL.PARTNER_ID`, `SETTLEMENT_DTL.PARTNER_ID`,
`PARTNER_CONTRACT.PARTNER_ID` and the copy settlement-anomaly reads through a join — one identifier
space, or could the same string mean different sellers in different tables?

**Why it matters.** Settlement pays per `PARTNER_ID`. If the order-side and settlement-side values
are not guaranteed to agree, every payout is a join across an unverified key, and the 1.8억 backlog
in `CANCEL_RECON_QUEUE` cannot be attributed to a partner with confidence. The vocabulary question
is the same question from the business side: 파트너지원팀 sits in CS본부 and its described work is
셀러 온보딩 · 셀러 문의 ("seller onboarding, seller enquiries"), so the team that registers these
entities uses a word that appears nowhere else in the company's systems. Whoever answers also
settles whether a partner onboarding date exists — without it the 신규 프로모션 6.0% fee tier
("입점 3개월 이내", within three months of joining) cannot be applied by any system, ever.

**What I already checked.** All four declarations: they are all `VARCHAR(20)` and there is no
foreign key between any pair, because V1 excludes FKs by design ("FK 제약 없음. 성능 이슈로 2019년
설계 당시 제외함."). Grepped all five repositories for a partner or seller table, seed data, lookup
or registry: only `PARTNER_CONTRACT` exists, it holds just `FEE_RATE`, `SETTLE_CYCLE` and `UPD_DT`,
and no running code reads it — `SettlementItemProcessor` hardcodes `new BigDecimal("0.12")`.
Grepped inventory-api and delivery-bff for `partner`/`PARTNER`: no match in either. 셀러 appears
exactly once in the whole reef, in the org chart.

**Files a human would need.** `order-service/src/main/resources/db/migration/V1__init.sql`,
`settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`,
`settlement-batch/src/main/resources/db/migration/V3__add_partner_contract.sql`,
`sources/context/org-chart.md` (the 파트너지원팀 row), `sources/context/business-rules.md` (the 수수료
sheet). 오세라 (파트너지원팀) and 김도윤 (정산팀) are the two people who together know the answer.

## GLOSSARY-SOURCE-INDEX — Is there a romanisation convention anywhere, and which spelling wins?

**Question.** Does a naming or romanisation convention for identifiers exist anywhere outside the
five repositories — a wiki page, a DBA standard, an onboarding document? And where two spellings of
one Korean word are both live in production, which one is the intended survivor?

**Why it matters.** Ten concepts carry two or more spellings across the estate and 상태 carries
nine. The cost is not aesthetic. `JEOKLIP_AMT` was dropped by V20 while `JEOKRIPGEUM` — the same
word, romanised differently — was left standing, so a cleanup migration removed the wrong twin's
sibling and nobody noticed. V15 renamed `BIGO` to `MEMO` and V16 reverted it within days because a
settlement query referenced the old name and production broke. The next rename will cost the same
unless someone can say what the target spelling is. This also decides the `ORDER_DTL` question:
whether the entity's `SKU_CD`/`QTY`/`UNIT_AMT` names are a planned rename awaiting a migration, or
a mapping error that has been wrong since it was written.

**What I already checked.** Every repository for a style guide, lint rule, checkstyle entry, PR
template clause or schema standard: `order-service/checkstyle.xml` governs Java style only, the PR
template says nothing about naming, and no other repo has any such file. Every document under
`sources/context/` and `sources/raw/`: no data dictionary, no naming standard, no DBA policy. The
CODEOWNERS file routes `/src/main/resources/db/` to `@sellflow/order-team @sellflow/dba`, so a DBA
group reviews migrations — but that group has no artifact or document in the reef, and no
migration comment cites a standard.

**Files a human would need.** `order-service/src/main/resources/db/migration/V7__add_point_columns.sql`
and `V20__drop_unused_point_columns.sql` (the 적립금 pair), `V15__rename_bigo_to_memo.sql` and
`V16__revert_rename_bigo.sql` (the rollback), `order-service/src/main/java/kr/co/sellflow/order/domain/OrderItem.java`
(the unmigrated entity names), `order-service/.github/CODEOWNERS`. 박성민 (주문팀) wrote most of the
migrations; whoever staffs `@sellflow/dba` is the other party.

## PROC-SELLFLOW-CANCEL-MONEY-PATH — Has any of the 188M KRW ever been recovered, and who decides the real number?

**Questions**
1. Has a single post-settlement cancellation ever been clawed back — by spreadsheet, by a manual
   payout adjustment, by netting against an invoice, by anything? The code cannot have done it
   (four independent breaks, each proven by grep), and the 2026-09-01 export filtered on
   `STATUS='PENDING'`, so off-system work is invisible to both sources available here.
2. The 188,851,520 KRW figure is not a sum of money paid. Every one of the 41 monthly amounts is
   exactly `count x 45,760`, so the column is a flat per-case estimate. The real exposure is
   `SUM(SETTLEMENT_DTL.JUNGSAN_AMT)` over the queued orders. Who runs that query, and does the
   answer go to 재무기획팀 as a liability?
3. Fixing break 1 (registering the Quartz trigger, SF-4512) is the cheapest and least useful of the
   three ordered breaks — without `SETTLEMENT_ADJUSTMENT` it fails on row one, and even with it the
   daily batch never reads the adjustment. Is the intended end state an automated deduction inside
   `dailySettlementJob`, or a reviewed manual workflow with the queue as its worklist? The answer
   decides whether SF-4512 should be done at all.
4. `SETTLEMENT_ANOMALY.CANCELLED_SETTLED` detects the same population from the other side and writes
   rows with `REVIEWED_BY`/`REVIEWED_DTM` columns nothing populates. If the queue gets an owner,
   should the anomaly output feed the same worklist, or be retired as a duplicate signal?

**Why it matters**
재무기획팀 asked both of these questions in writing on 2026-08-24 and has not been answered:
"차감이 정상적으로 이루어지고 있다면 대기 잔액이 이 규모로 누적될 수 없습니다" ("if the deduction were
happening properly, a waiting balance could not accumulate to this size"). Quarterly close needs to
know whether to book a liability, and that needs a number no artifact can produce from code alone.
The 2025-07-12 postmortem raised the same question as an action item — "취소 건이 정산 대상에서
제외되는지 점검" — and it was moved to a ticket whose number was never recorded.

**Already checked**
Every file on the path read line by line: `OrderController`, `OrderCancelService`,
`OrderEventPublisher`, `OrderEventOutbox`, `OrderEventRelayJob`, `CancelReconciler`, `QuartzConfig`,
`DailySettlementJobConfig`, `SettlementItemProcessor`, `SettlementItemWriter`, `MarkSettledTasklet`,
`settlement-anomaly/app/main.py` and `model/detector.py`, plus all six settlement migrations.
`grep -rn "CancelReconciler\|reconcileCancellations"` over all five repos returns only the class's own
file; `grep -rn "SETTLEMENT_ADJUSTMENT"` returns exactly one line, the INSERT that targets it;
`grep -rn "SETTLEMENT_ANOMALY"` returns only the DDL, the INSERT and the README. The queue appears in
no query in `settlement-batch` outside `CancelReconciler`.

**Files a human would need**
`sellflow-reef/sources/raw/exports/cancel_recon_queue_monthly_20260901.csv` and its README,
`sellflow-reef/sources/raw/mail/RE_정산_미정정_금액_문의.eml`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`,
`settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`,
`sellflow-reef/sources/context/sprints/tickets_2026-S17.csv` (SF-4512). The three people who
together hold the answer are 김도윤 (정산팀), 문지영 (재무기획팀) and whoever now owns SF-4512.


## ORDER-INTAKE — What creates an ORDER_MST row, and what serves the other 29 endpoints the 2022 spec declares?

**Question.** Which system creates orders and moves them through 결제완료 → 상품준비중 → 배송중 → 배송완료, and is it a deployment of order-service built from a source tree other than the one in `sellflow/order-service`?

**Why it matters.** The reef documents an order domain whose creation half is missing. [[PROC-ORDER-ORDER-MST-LIFECYCLE]] has to record the row's birth as an absence rather than describe it, and it cannot say what performs any of the four pre-cancel transitions. [[PROC-ORDER-CANCEL]] cannot say what drives 결제완료 → 배송완료. [[GLOSSARY-ORDER]] cannot say what `CHORI_SANGTAE` means beyond the single literal `'COMPLETED'`. [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]] states that its access matrix covers five repositories only.

The answer can flip a conclusion that several artifacts rest on. This reef's premise, stated in its own CLAUDE.md, is that five repositories are the estate. If a sixth source tree exists, then the components the reef calls orphaned — `OrderStatusService.change` (no caller), `InventoryClient.restore` (no caller), `PaymentClient.cancel` (no caller) — may well have callers there, and [[PAT-SELLFLOW-ORPHANED-COMPONENTS]], [[CON-ORDER-INVENTORY]] and [[PROC-ORDER-CANCEL]] would each need rewriting in the opposite direction. It also bears on [[DEC-DELIVERY-GENERATED-CLIENT]]: that client was generated from this same 2022 spec and calls `/api/v1/orders/{ordNo}/cancel`, so whatever serves the spec's other 29 endpoints is the most likely place its `/api/v1` prefix was ever correct.

**Already checked.** `grep -rn "INSERT INTO" repos/` over all five repositories returns five INSERT statements in total, and none targets `ORDER_MST`: `SETTLEMENT_DTL` (SettlementItemWriter), `CANCEL_RECON_QUEUE` (OrderEventRelayJob), `SETTLEMENT_ADJUSTMENT` (CancelReconciler), `ORDER_CANCEL` (the deprecated OrderCancelServiceV1) and `SETTLEMENT_ANOMALY` (settlement-anomaly). `OrderController` maps exactly one route, `POST /orders/{ordNo}/cancel`, and the repository contains no other `@RestController`. Against that, the 2022 spec `sources/raw/specs/order-service-openapi.json` (version 2.4.0, generated 2022-11-04) declares **30** paths for this same service, including `POST /orders`, `POST /orders/{ordNo}/split`, `PUT /admin/orders/{ordNo}/force-status` and `POST /admin/orders/bulk-cancel` — that is, order creation and administrative status forcing are documented as order-service's own API. `sources/context/registry/services.yaml` lists five services and no order-intake system, and carries `last_reviewed: 2026-03-02   # 이후 갱신 없음` ("no updates since"). The code names two callers that the registry does not contain: `OrderCancelService`'s javadoc says "취소 요청은 CS 어드민과 고객 앱 양쪽에서 들어온다" ("cancel requests arrive from both the CS admin and the customer app"), and the code-derived order API labels one path "CS 어드민 주문 검색". The four ORDER_MST statuses that precede cancellation are written by nothing in any repository — verified by grepping all five for `GYEOLJE_WANRYO`, `SANGPUM_JUNBI`, `BAESONG_JUNG` and `BAESONG_WANRYO`; the only hits are the enum declaration, the settlement reader's `WHERE` clause and the generated TypeScript union.

**Files a human would need.** Whatever repository serves `POST /orders` today; `sources/raw/specs/order-service-openapi.json`; `order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java`; `order-service:SERVER_VERSION` and the four CD workflows under `order-service:.github/workflows/`, which would show which artefact is actually deployed. 박성민 (주문팀), the author of both the cancel path and the 2021 policy page, is the likeliest single answer source.

## SETTLEMENT-VOLUME — Is the correction workload ten cases a month or a hundred, and has anyone ever measured it?

**Question.** What is the current monthly volume of post-settlement cancellations requiring correction, measured rather than estimated — and does the 2026 automation plan's "월 10건 내외" figure come from a measurement or from repeating a 2023 prediction?

**Why it matters.** This is the one answer that could flip a live decision rather than a documented one. [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]] records a programme now in design that is sized on the 2023 figure. The queue export contradicts it by an order of magnitude: 4,127 rows accumulated since 2023-04, with the very first month already at 48. If the true rate is roughly 100 a month, the plan's "담당 인원이 3명으로 병목 발생 가능성" framing, its 80 percent time-saving target and its 2027-01 pilot date are all sized against the wrong number, and [[RISK-SETTLEMENT-RECON-BACKLOG]] is not a backlog to drain once but a daily inflow nobody handles. Answering it also settles whether 정산팀 have been processing ten cases a month, or have been processing the few that partners complained about and calling that the workload.

**Already checked.** The plan (`sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, PLAN-2026-014, status 검토중, exported 2026-09-01) attributes the figure to "정산팀 확인 결과" with no person, date or method. The 2023 origin is in SF-2287's thread: 김도윤 wrote on 2023-04-07 "CS팀 통계 보니 실제 정산 후 취소로 이어지는 건은 월 10건 미만일 것으로 예상됩니다" — a prediction, and the CS statistics behind it are in no source here. The 2026-06-18 kickoff minute has 정산팀 presenting the same number, and logs the action item "현행 월 처리 건수 실측" with 기한 미정 (no due date); no measurement result appears in `sources/context/sprints/2026-S17_log.md` or `tickets_2026-S17.csv`. The contradicting data is `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`: 41 months, 4,127 rows, all still `STATUS='PENDING'`, oldest 2023-04 at 48 rows. The handover states the real practice plainly: "실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음… 전체 대기열을 주기적으로 확인하는 절차는 없음."

**Files a human would need.** 윤서진 (데이터팀) as the plan's author and 김도윤 (정산팀) as the source of both figures; `sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md`; `sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md`; `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`; the CS team's March 2023 enquiry statistics, which exist in no source the reef holds.

## MONEY-OUT — Is a customer ever refunded on cancellation, and by what?

**Question.** When a cancel succeeds, what refunds the customer's payment — and is the PG integration in `PaymentClient` live, dead, or never wired?

**Why it matters.** [[PROC-ORDER-CANCEL]] currently describes a cancellation that changes a status, writes a row and publishes an event, and has to record "whether refunds happen" as unknown while the 2021 policy page promises 전액 환불 for early-stage cancels. That is the difference between a documented gap and a customer-money defect, and no artifact in the reef can state which it is. The same answer settles [[DEC-SELLFLOW-MONEY-ROUNDING]]'s question of which figure is customer-facing, and it tells [[PROC-SELLFLOW-CANCEL-MONEY-PATH]] whether the money path has one broken end or two.

**Already checked.** `PaymentClient.cancel(ordNo, amount)` exists at `order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java`; it POSTs to `${PG_BASE_URL:-https://pg.example.co.kr}/v2/payments/cancel` with a null body and discards the response. `grep -rn "PaymentClient" repos/` over all five repositories returns only the class's own declaration and logger lines — no injection point, no caller, no test. Its own comment is explicit that the two flows are separate: "결제 취소와 정산 차감은 별개다. PG 취소가 성공해도 파트너에게 이미 지급된 정산 금액은 되돌아오지 않는다." The 2021 Confluence page's §6 links three documents that would hold the policy — 결제 및 환불 정책, 반품 프로세스, 정산 배치 운영 가이드 — and none of the three is in `sources/`. No PG name, merchant id, webhook handler or callback route appears anywhere in the five repositories; `PG_BASE_URL` is set in no `.env` template, workflow or yml.

**Files a human would need.** The 결제 및 환불 정책 Confluence page linked from `sources/raw/confluence-snapshots/주문-취소-정책_48213.html` §6; whatever service owns the PG integration (absent from `sources/context/registry/services.yaml`); `order-service:src/main/java/kr/co/sellflow/order/payment/PaymentClient.java`; `order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`.

## OFF-REPO-SYSTEMS — What are 정산 어드민 and 파트너 포털, and do they perform the settlement writes no repository performs?

**Question.** Two systems are named as working tools in the settlement documents but appear in no repository and in no registry entry: 정산 어드민 (settlement admin) and 파트너 포털 (partner portal). Do they exist today, who operates them, and does 정산 어드민 hold the correction history, the `SETTLEMENT_RUN` row and the 지급 요청 transmission that no code in the five repos accounts for?

**Why it matters.** Several artifacts stop at the same wall from different directions: [[PROC-SETTLEMENT-FLOW-CATALOG]] cannot name what opens a `SETTLEMENT_RUN` or what actually transmits a payment request; [[GLOSSARY-SETTLEMENT]] cannot say whether 지급 요청 is a distinct artefact or just a `SETTLEMENT_DTL` row; [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] cannot say whether `SETTLEMENT_RUN_LOG` has ever held a row; [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] cannot say how manual corrections were carried out. One answer — "정산 어드민 is X, operated by Y, and it writes Z" — either resolves all four or confirms that the reef is missing an entire system, which would change what [[SYS-SETTLEMENT]] claims to cover.

**Already checked.** 정산 어드민 appears in exactly four places in `sources/`, none of them technical: procedure v1.1 §5 ("정정 이력은 정산 어드민에 기록하며 5년간 보존한다"), the 2025-03 handover's access table ("정산 어드민 | 정정 등록 | 팀장 승인 후"), the 2026-S17 sprint goal ("정산 어드민 조회 화면 1차", SF-5120 정산 정정 대기열 조회 화면, 이수민, 5SP), and the code-derived order API's "CS 어드민 주문 검색" summary. 파트너 포털 appears once, in the same handover access table, with its application method recorded as TBD. Neither is in `sources/context/registry/services.yaml`, which lists five services. No repository contains an admin UI, a template, a static asset or a session/login route — `grep -rn "어드민\|admin" repos/` returns only the spec-derived `/admin/*` paths that order-service does not implement. The handover also records that the official correction form does not exist and "각자 엑셀로 관리 중" (everyone keeps their own spreadsheet), which suggests 정산 어드민's 정정 등록 may not be in use even where access is granted.

**Files a human would need.** 이수민 (정산팀), who holds both the handover and SF-5120; `sources/context/handover/2025-03_정산팀_인수인계.md` §4; `sources/context/policy/정산_정정_업무절차_v1.1.md` §5; `sources/context/sprints/2026-S17_baseline.md`; and the repository or vendor behind 정산 어드민, which is in none of the five.

## ORDER-SCHEMA-REALITY — Which of these ORDER_MST columns and which delivery table actually exist in production?

**Question.** For each of `INFLOW_CHNL`, `SETTLE_REF_NO`, `ORD_MEMO`, `ORD_DT`, `BAESONG_MSG` and the whole `ORDER_DELIVERY` table: does it exist in the production schema, and if so what created it?

**Why it matters.** Six artifacts hedge on the same missing DDL. [[SCH-DELIVERY]] can describe `ORDER_DELIVERY`'s four column names and no type, length, nullability or index, because no migration creates the table. [[GLOSSARY-ORDER]] lists five `ORDER_MST` columns that migrations index, backfill and query but never create. [[PROC-ORDER-ORDER-MST-LIFECYCLE]] cannot say whether `SETTLE_REF_NO` is `JUNGSAN_RUN_ID` renamed. [[GLOSSARY-DELIVERY]] cannot say whether `TAKBAE_CD` holds the spec's five carrier codes. If these objects exist, some path outside Flyway creates schema in this database, and every migration-derived statement in [[SCH-ORDER]] is a partial view. If they do not exist, then `V18`'s backfill, `V21`'s and `V24`'s index creations, and `OrderSearchService`'s `SELECT * FROM ORDER_MST WHERE ORD_DT BETWEEN ? AND ?` have all been failing, which would be a different and louder finding.

**Already checked.** Each name was grepped across all five repositories on 2026-09-19. `INFLOW_CHNL`: one hit, `V18__backfill_cancel_channel.sql` line 6, a `WHERE m.INFLOW_CHNL = 'APP'` predicate. `SETTLE_REF_NO`: one hit, `V21__add_settlement_ref_index.sql`, a `CREATE INDEX` on it. `ORD_MEMO`: one hit, `V24__add_order_memo_search_index.sql`, `CREATE INDEX ... ON ORDER_MST (ORD_MEMO(64))`. `ORD_DT`: one hit in a runtime query, `OrderSearchService.java:24`. `BAESONG_MSG`: one hit, a TODO comment in `V13__cleanup_unused.sql`. No `ALTER TABLE ... ADD COLUMN` anywhere in V1–V24 creates any of the five; by contrast `UNSONGJANG_BEONHO` (V5), `PARENT_ORD_NO` (V10) and `JUNGSAN_RUN_ID` (V14) each have a visible `ADD COLUMN`, so the absences are specific rather than a gap in the reading. `ORDER_DELIVERY` returns exactly four lines across all repos, all of them JPA annotations in `DeliveryInfo.java`. `V23__consolidated_schema.sql`, which announces itself as the consolidated baseline "내용은 현재 운영 스키마 덤프와 동일" (contents identical to the current production schema dump), contains four comment lines and no statement of any kind.

**Files a human would need.** A `SHOW CREATE TABLE ORDER_MST` and `SHOW CREATE TABLE ORDER_DELIVERY` against production, and the Flyway `schema_version` table; `order-service:src/main/resources/db/migration/` (V13, V18, V21, V23, V24); `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java`. See also the existing entry "SCH-ORDER-MIGRATION-HISTORY — were V20, V21, V22 and V24 ever applied", which asks the narrower version of this question.

## OUTBOX-OPERATIONS — How large is ORDER_EVENT_OUTBOX, has it ever been pruned, and has the relay ever crashed mid-loop?

**Question.** 1. How many rows does `ORDER_EVENT_OUTBOX` hold today, and does anything delete from it? 2. Has the relay ever failed between its `CANCEL_RECON_QUEUE` insert and its `PUBLISHED_YN` update in production?

**Why it matters.** These two share an answer source — the production table and the relay's logs — and together they decide whether [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]]'s duplicate-insert reasoning is a live defect or a theoretical one. The relay's two statements are separate `JdbcTemplate` calls with no `@Transactional`, and neither table has a uniqueness constraint, so a crash between them re-queues the row on the next poll. `CANCEL_RECON_QUEUE` already holds 4,127 unprocessed rows; if some are duplicates, the backlog figure [[RISK-SETTLEMENT-RECON-BACKLOG]] and [[PROC-SELLFLOW-CANCEL-MONEY-PATH]] both quote is overstated, and the answer would change the number the reef reports. The same lookup settles whether the relay has ever hit its 500-row ceiling in one poll, which is the ten-minute cadence's only failure mode, and whether [[PAT-SELLFLOW-DB-AS-QUEUE]] is describing an accumulating table or a stable one.

**Already checked.** `grep -rn "ORDER_EVENT_OUTBOX" repos/` across all five repositories returns the `V8` creation, the `V11` index, the publisher's INSERT, the relay's SELECT and UPDATE, and nothing else: no DELETE, no TTL, no archival job, no retention migration. `V11__outbox_index.sql` records that the relay had slowed down and gives no volume figure, no query plan and no ticket, and its stated date (2024-05-16) cannot be trusted as a chronology because the comment dates in this repository are non-monotonic. The only volume evidence anywhere in the reef is `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`, which counts a strict subset — settled orders only — and no export of the outbox exists. The 2025-07-12 postmortem (`sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`) covers duplicate partner payment requests and never mentions the relay, although the relay shares the Quartz scheduler whose clustering configuration was applied to only one node.

**Files a human would need.** `SELECT PUBLISHED_YN, COUNT(*) FROM ORDER_EVENT_OUTBOX GROUP BY 1` and `SELECT ORD_NO, COUNT(*) FROM CANCEL_RECON_QUEUE GROUP BY 1 HAVING COUNT(*) > 1` against production; `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`; `order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql` and `V11__outbox_index.sql`; whatever log store holds the relay's per-run count lines.

## SF2287-GOVERNANCE — Was removing the settled-order cancel block approved by anyone, and is 배송완료 meant to be cancellable?

**Question.** 1. Did the removal of the settlement check from the cancel API have an approver above the three people in the ticket thread? 2. Post-change, `BANPUM` blocks a cancel and `BAESONG_WANRYO` does not, which is not what the 2021 policy table says — is that the intended policy?

**Why it matters.** [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] records a policy change that created a standing financial liability — every post-settlement cancel now needs a manual clawback — and the reef cannot say whether anyone outside 주문팀 and 정산팀 agreed to it. The second half is sharper: the live guard is `EnumSet.of(CHWISO, BANPUM)`, so a 배송완료 order can be cancelled through the API while the 2021 table says it must go through 반품. If that is not intended, [[PROC-ORDER-CANCEL]]'s description of the current guard is describing a defect, not a rule, and the fix is a one-line change. If it is intended, the 2021 page is wrong on two rows and [[RISK-SELLFLOW-DOC-DRIFT]] gains its clearest instance.

**Already checked.** The whole of `sources/context/tickets/SF-2287.md`: reporter 최은영 (CS팀), assignee 박성민 (주문팀), created 2023-04-03, resolved 2023-04-21, fix_version order-service 2.8.0, status Done. The thread runs CS → 주문팀 → 정산팀 → a deployment confirmation on 2023-04-24. There is no approver field, no sign-off, no reference to a policy-change process and no mention of 재무본부, whose 본부장 approves the correction procedure that this change made necessary. The 214-enquiry figure it opens with has no underlying CS report in the reef. The 2021 page (`sources/raw/confluence-snapshots/주문-취소-정책_48213.html`) carries the banner "이 문서는 주문팀에서 관리합니다. 정책 변경 시 반드시 이 페이지를 갱신해 주세요", was created and last modified by 박성민 on 2021-03-17, and its §3 table gives 배송완료 as "△ 반품 프로세스로 전환" and 정산완료 as "X 취소 불가". Its only challenge is a comment from 강태오 dated 2024-08-19 asking whether the page is still valid; it is unanswered. The deprecated `OrderCancelServiceV1` still contains the old settlement check and has zero callers.

**Files a human would need.** 박성민 (주문팀) for both halves; `sources/context/tickets/SF-2287.md`; `sources/raw/confluence-snapshots/주문-취소-정책_48213.html` §3 and §4; `order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`; `order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java`.

## CLAWBACK-NOTIFICATION — Has a partner ever been notified of a clawback, and how much correction work happens off-system?

**Question.** 1. Procedure v1.1 §4.4 requires 파트너 통지 for every correction — has that notification ever been sent, and by what? 2. How many of the 4,127 `PENDING` queue rows have in fact been settled by a manual correction that left the row untouched?

**Why it matters.** These are the two facts that decide whether [[RISK-SETTLEMENT-RECON-BACKLOG]]'s headline number means anything. If corrections have been done by hand and the queue was never updated, the backlog is smaller than 4,127 and the queue is not a worklist but an unreconciled log — which changes what [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] can claim about the compensation mechanism and what the 2026 automation programme would be automating. And if no partner has ever been notified, then a mandatory step of a procedure approved by 재무본부장 has never been performed by anything, which is a compliance finding [[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]] currently cannot state.

**Already checked.** Procedure v1.1 (`sources/context/policy/정산_정정_업무절차_v1.1.md`, revised 2024-02-19, approved by 재무본부장, status 발행) assigns 파트너 통지 to 정산팀 in §3 and makes it step 4 of §4. No code in any of the five repositories sends mail, a message or a notification of any kind: there is no mail client, no SMTP configuration, no notification service dependency and no template — checked by grepping all five repos for mail, smtp, notify, 통지 and 알림. The partner-facing report writer inserts into `SETTLEMENT_DTL` and logs. The handover (`sources/context/handover/2025-03_정산팀_인수인계.md`) describes the actual practice as reactive and undocumented: "실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음", "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중." The export's README warns that a row may have been corrected manually while staying `PENDING`. The `SETTLEMENT_ADJUSTMENT` table that would record a correction is created by no migration and cannot be queried, per the same handover §3.

**Files a human would need.** 이수민 and 김도윤 (정산팀), plus the individual spreadsheets the handover describes; `sources/context/policy/정산_정정_업무절차_v1.1.md` §3–§5; `sources/context/handover/2025-03_정산팀_인수인계.md`; `sources/raw/exports/README.md`; whatever the 파트너 포털 shows a partner about deductions.

## RELAY-OWNERSHIP — Who owns the outbox relay, and was a message broker ever on the table?

**Question.** 1. Which team owns `OrderEventRelayJob` — the team whose repository it lives in (정산팀) or the team whose publication step it implements (주문팀)? 2. Was a message broker evaluated and rejected in 2023, or was it never considered?

**Why it matters.** [[DEC-ORDER-OUTBOX-RELAY]] has to present the polling design as an outcome with no recorded rationale, and [[PAT-SELLFLOW-DB-AS-QUEUE]] generalises that pattern across the estate without being able to say whether it was ever a decision. Ownership is the more actionable half: the relay is the single component on which the entire cancel-to-settlement path depends, and no artifact can name a team that would be paged if it stopped. Every downstream question in [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] — retention, crash recovery, the 500-row ceiling — is unanswerable in part because there is no owner to ask.

**Already checked.** `sources/context/registry/services.yaml` assigns `settlement-batch` to 정산팀 with the note "아웃박스 릴레이 잡 동거" ("the outbox relay job cohabits") and offers no field for component-level ownership; `ORDER_EVENT_OUTBOX` is listed under `queues` with producer order-service and consumer settlement-batch, while `CANCEL_RECON_QUEUE`'s consumer is `TODO   # 확인 필요`. SF-2287's thread settles the transport in two lines — 박성민: "이벤트 하나 발행하겠습니다. `order.cancelled` 구독하시면 됩니다", 김도윤: "네 컨슈머 붙여놓겠습니다" — with no discussion of a broker before or after. `V8__add_cancel_event_outbox.sql` states "별도 브로커 없음" as a fact rather than as a decision. No broker of any kind is declared by any of the five services: no Kafka, RabbitMQ, SQS, Redis or ActiveMQ dependency in either `build.gradle`, either `requirements.txt` or `package.json`. `order-service:TASK.md` still lists "정산팀 컨슈머 확인" as open.

**Files a human would need.** 박성민 (주문팀) and 김도윤 (정산팀), the two parties to the 2023 exchange; `sources/context/tickets/SF-2287.md`; `sources/context/registry/services.yaml`; `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`; `order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql`.

## CARRIER-CONTRACT — What does the carrier API actually return, and how is a caller meant to treat `stale`?

**Question.** 1. Which carrier is behind `CARRIER_API`, and what is the response body and error catalogue of `GET /tracking/{ordNo}`? 2. Is any consumer expected to act on the `stale: true` flag, given that the fabricated body is served with HTTP 200?

**Why it matters.** delivery-bff returns the carrier's body to its callers verbatim, cast and unvalidated, so the carrier's contract *is* this service's contract. Until it is known, [[SCH-DELIVERY]] cannot say whether `status` and `carrierCd` carry the vocabularies the 2022 spec documents, [[API-DELIVERY]] cannot describe the endpoint's real response, [[GLOSSARY-DELIVERY]] cannot say whether the delivery domain has one vocabulary or two, and [[PROC-DELIVERY-STATUS-SYNC]] cannot say which failures the bare `catch` is retrying. The `stale` half matters on its own: after three failures the service invents `PREPARING` for an order that may have been delivered, and answers 200. If the app ignores the flag — which nothing documents — customers see a delivered order as 준비중 whenever the carrier is down.

**Already checked.** `CARRIER_API` is declared in `delivery-bff/.env.template` with no value; the code default is the placeholder `https://api.carrier.example`, which is the only carrier URL anywhere in the five repositories. No API key, signing routine, mTLS material, vendor contract or integration note for a carrier exists in any repo or anywhere under `sources/`. The response is consumed by `return res.data as DeliveryStatus` in `src/deliveryStatus.ts` — a cast, with no schema check, no field access and no parsing, so nothing in the repository constrains the payload. The retry is three attempts with a 3000 ms timeout and 500/1000 ms backoff inside one `catch (e)` that distinguishes nothing, so a 404, a 401, a DNS failure and a timeout are retried identically. On the order side the vocabulary does exist and was confirmed during this pass: the 2022 spec's `DeliveryInfo.baesongSangtae` enumerates PREPARING, PICKED_UP, IN_TRANSIT, OUT_FOR_DELIVERY, DELIVERED, FAILED, and `OrderMst.taekBaeSaCd` enumerates CJ, HANJIN, LOTTE, POST, LOGEN — but neither enum appears in delivery-bff, and the spec's own `DeliveryInfo.taekBaeSaCd` is an unconstrained string. The `stale` flag is documented nowhere: no README line, no API description, no consumer.

**Files a human would need.** The carrier vendor's integration document and the deployed value of `CARRIER_API` (in no repository — see the existing entry "API-DELIVERY — Which carrier host is delivery-bff actually calling"); `delivery-bff:src/deliveryStatus.ts` and `src/index.ts`; `sources/raw/specs/order-service-openapi.json`; whoever owns the customer app that calls `GET /delivery/:ordNo`.

## SHARED-DB-2019 — Does the 2019 consolidated-database agreement exist as a document, and what did it permit?

**Question.** Is there a written record of the 2019 통합 DB 협의 — and did it enumerate which teams may read and write which other teams' tables?

**Why it matters.** Three code comments invoke this agreement to justify the estate's most consequential structural property: that services integrate through each other's tables rather than through APIs. [[DEC-SELLFLOW-SHARED-DB]] has to present the decision with no rationale and no participants. [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]]'s access matrix records what the code attempts and cannot say which of those accesses was sanctioned. [[PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE]] cannot say whether a canonical cancellation model was considered and rejected. If the agreement enumerated sanctioned accesses, the matrix can be marked against it and the unsanctioned ones become findings; if it was verbal, then the estate's defining constraint has no owner and no scope, which is itself the answer.

**Already checked.** The agreement is cited in exactly three places, all of them source comments: `MarkSettledTasklet` ("ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)"), `OrderStatusService`'s class javadoc ("정산 배치가 ORDER_MST 를 직접 갱신한다 (2019 협의, 통합 DB 정책)"), and the settlement schema header. No document, ADR, minute, ticket or mail in `sources/` mentions 2019 at all. The registry records a different logical database per service (order, settlement, inventory) while all four services' connection settings resolve to `sellflow_order`, and it names no DBA and no instance owner; `sources/context/org-chart.md` has no DBA or 인프라 entry either, although the 2025-03 handover's access table routes 정산 DB read access to 인프라팀.

**Files a human would need.** 인프라팀, as the only team the sources associate with database access; the 2019 minutes or mail thread, in none of the sources here; `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`; `order-service:src/main/java/kr/co/sellflow/order/service/OrderStatusService.java`; `sources/context/registry/services.yaml`.

## MONEY-ROUNDING — Which fee figure does a partner actually see, and has the divergence ever been reconciled?

**Question.** When a partner queries their settlement, which number are they shown — the batch's `HALF_UP` figure, a portal-side recomputation, or something else — and has anyone ever compared it against the order side's figure?

**Why it matters.** [[DEC-SELLFLOW-MONEY-ROUNDING]] can already state what the code does: three rounding behaviours exist where the comments describe two, and the only one that reaches a payout is `SettlementItemProcessor`'s inline `HALF_UP`. What it cannot state is which figure is customer-facing, and therefore whether the documented 2022 agreement has any effect on anyone. That decides whether the artifact's finding is a live one-won discrepancy between what a partner is shown and what they are paid, or a dead comment in an uncalled utility. The handover reports that partner enquiries are "대부분 금액 차이" (mostly amount differences), so the question is not hypothetical.

**Already checked.** Both `MoneyUtil` classes were read in full. `settlement-batch`'s uses `RoundingMode.FLOOR`; `order-service`'s uses `HALF_UP`; each comment points at the other and dates the split to a 2022 agreement it says is undocumented. `grep -rn "MoneyUtil" repos/` across all five repositories returns the two class declarations, their private constructors, and one test — `SettlementItemProcessorTest` asserting `MoneyUtil.fee(14999, 0.1) == 1499`. Neither utility has a production caller. The live payout arithmetic is `gross.multiply(new BigDecimal("0.12")).setScale(0, RoundingMode.HALF_UP)` in `SettlementItemProcessor`, with the rate hard-coded and the `PARTNER_CONTRACT` rates unused since 2021 by the processor's own comment. No partner-facing display code exists in any repository; 파트너 포털 is named only in the handover's access table. `FEE_MISMATCH` is listed in settlement-anomaly's README as an anomaly type and is not implemented, so nothing checks for a discrepancy either.

**Files a human would need.** Whoever owns 파트너 포털; `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java` and `common/MoneyUtil.java`; `order-service:src/main/java/kr/co/sellflow/order/common/MoneyUtil.java`; `sources/context/business-rules.xlsx` (수수료 sheet); and whoever was party to the 2022 rounding agreement.

## REASON-CODE-CONTRACT — Is the 01–04 reason vocabulary a contract between 주문팀 and 재고팀, and which RESTOCKABLE_REASONS is authoritative?

**Question.** 1. Is the cancel reason code an agreed shared vocabulary, or does each service define its own? 2. `RESTOCKABLE_REASONS = {"01","02"}` is declared twice in inventory-api, in `app/config.py` and again in `app/main.py` — which one is meant to be authoritative?

**Why it matters.** [[GLOSSARY-INVENTORY]] cannot say whether the four codes reaching inventory-api are guaranteed to be order-service's `CancelReason` codes; [[CON-ORDER-INVENTORY]] rests on that equivalence; [[DEC-INVENTORY-RESTOCK-BY-REASON]] cannot say which declaration a change should edit. The two services already disagree on what an invalid code means — inventory-api answers an unknown code such as `"99"` with HTTP 200 and `{"restocked": false, "reason": "not_restockable"}`, while order-service's `CancelReason.of` throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)`. If the vocabulary is a contract, that asymmetry is a defect to file; if it is not, then the 취소정책 sheet is not a specification and [[CON-ORDER-INVENTORY]] should say so. The spelling half of this routes to [[PAT-SELLFLOW-ROMANISED-NAMING]], which cannot say whether inventory-api's divergent spellings follow a separate team vocabulary or are simply typos.

**Already checked.** `order-service`'s `CancelReason` enum defines 01 파트너 귀책, 02 시스템 오류, 03 고객 변심, 04 배송 실패, and its javadoc pushes the cost-allocation question elsewhere: "사유별 비용 부담 주체는 코드에 정의하지 않는다. 정산 정책은 재무본부 소관이며 별도 기준표를 따른다." The same four codes appear in the 2021 Confluence page §2 and in the 취소정책 sheet. inventory-api declares no enum and performs no validation; it holds the set twice, at `app/config.py:5` and `app/main.py:18`, neither importing the other, neither marked deprecated, with differently worded comments. The live path uses `main.py`'s copy — `app/main.py:29` tests `req.reason_code not in RESTOCKABLE_REASONS` against the module-level literal — while the repository's only test imports `config.py`'s copy, so the test exercises a constant the handler does not read. No import, shared package, generated client or schema links the two services. The spelling also differs between them: `config.py` glosses the labels as 파트너귀책 and 시스템오류 without spaces, against order-service's and the sheet's 파트너 귀책 and 시스템 오류.

**Files a human would need.** 재고팀 and 주문팀 jointly; `inventory-api:app/config.py`, `app/main.py`, `tests/test_restore.py`; `order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java`; `sources/context/business-rules.xlsx` (취소정책 sheet). See also the existing entry "RISK-INVENTORY — Should cancel reason `04` (배송 실패) restock", which this one does not repeat.

## CANCEL-OPS — When a cancellation fails, is there any way to find it or retry it?

**Question.** When `OrderCancelService.cancel` throws something other than the two exceptions the service maps, what does an operator do — and is there any identifier that ties a customer complaint to the failed request?

**Why it matters.** [[PROC-ORDER-ERROR-HANDLING]] can describe the failure behaviour but cannot say whether anyone can act on it, and [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] cannot say what recovers a cancel whose event was never written. The reef currently states that no retry path exists; that is a claim about the repositories, not about the operation. If an admin tool or a DBA runbook does the recovery, the artifacts are understating the system's real resilience. If nothing does, then a failed cancel is invisible and unrecoverable, which belongs in [[RISK-ORDER]] as a finding rather than as an open unknown.

**Already checked.** `GlobalExceptionHandler` is a `@RestControllerAdvice` and maps exactly two exceptions — `OrderNotFoundException` to 404 and `OrderCancelNotAllowedException` to 409, each with a `{"message": ...}` body. Everything else, including any database or outbox failure, falls through to Spring's auto-configured `BasicErrorController`: `server.error.*` is unset in `application.yml` and in all four environment profiles (dev, qa, stage, prod), and no captured 500 response from this service exists in any source. The cancel itself is atomic — `cancel` and `publishOrderCancelled` are both `@Transactional` and the outbox row is written through the same JPA repository layer — so the failure mode is not a half-finished cancel but a cancel that silently did not happen. No request id is generated, logged or returned anywhere in the repository — no MDC setup, no filter, no correlation header — so a customer report cannot be matched to a log line. The spec declares `POST /admin/events/outbox/{eventId}/replay`, which the application does not implement; there is no admin endpoint, no replay job and no dead-letter table. The repository has no actuator, no metrics exporter and no Sentry or Datadog dependency, and the cancel path logs nothing at ERROR, so a log-based alert would have nothing to match.

**Files a human would need.** Whoever operates 셀플로우's log platform, and the CS team's escalation runbook (not in `sources/`); `order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`; `order-service:src/main/resources/application.yml` and the four profile files; `sources/raw/specs/order-service-openapi.json` (`/admin/events/outbox/{eventId}/replay`).

## DEPLOYMENT-GAP — How are delivery-bff, settlement-anomaly and inventory-api deployed?

**Question.** Three of the five services have no complete deployment path in version control. What runs them, and where does that definition live?

**Why it matters.** [[PROC-SELLFLOW-RUNTIME]] cannot state instance counts, load balancing or environment-variable injection for any service, which in turn leaves [[PROC-DELIVERY-AUTH]], [[PROC-INVENTORY-AUTH]] and [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]] unable to say whether the four unauthenticated services are network-reachable or not — the difference between "no authentication" and "no authentication and open". It also decides a concrete question [[PROC-DELIVERY-STATUS-SYNC]] holds open: whether `CARRIER_API` is injected at deploy time, and therefore whether delivery-bff talks to a real carrier at all. Three more artifacts stop at the same line: [[PROC-INVENTORY-ERROR-HANDLING]] cannot say whether uvicorn's log records reach anywhere or whether anything watches this service's 5xx rate; [[PROC-ORDER-AUTH]] cannot say whether any network policy restricts port 8081 or what serves the spec's `/internal/health`, `/internal/ready` and `/internal/metrics`; and [[SCH-SETTLEMENT-ANOMALY]] cannot say whether `sql/V1__anomaly_schema.sql` was ever applied, since settlement-anomaly has no migration runner, no CI and no Dockerfile to apply it with.

**Already checked.** A `find` across `sellflow/repos` for every yaml, yml, Dockerfile, compose, Chart, ingress and tf file: `delivery-bff` contributes nothing at all — no Dockerfile, no workflow, no manifest; `settlement-anomaly` contributes only `sql/V1__anomaly_schema.sql` and has no Dockerfile and no CD workflow, so nothing in version control could deploy it; `inventory-api` contributes exactly one file, its Dockerfile, whose `CMD` runs uvicorn on `0.0.0.0:8000` over plain HTTP. Only `order-service` and `settlement-batch` have `.github/workflows` at all, and both end in `./deploy.sh $ENVIRONMENT`, a script absent from both repositories. No ingress manifest, service-mesh policy, firewall rule or Kubernetes resource exists in any of the five repos, so reachability is unknown rather than known-open. This is the same wall as the existing entries "SYS-ORDER — Where is order-service's deploy.sh" and "PROC-SETTLEMENT-AUTH — How does the database password reach production", asked for the three services those entries do not cover.

**Files a human would need.** The infrastructure repository or platform configuration that holds `deploy.sh` and the three services' runtime definitions; `inventory-api:Dockerfile`; `delivery-bff:.env.template`; `settlement-anomaly:README.md` and `.env.template`; 인프라팀, which the 2025-03 handover names as the route for database access.

## DB-GRANTS — Is any service's database account scoped, or do all four share one unrestricted user?

**Question.** What grants does the MySQL user `sellflow` hold, and does each service connect as a distinct account?

**Why it matters.** [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]]'s access matrix records what the code attempts, not what the server permits, and says so. That distinction is the whole safety argument for an estate that integrates through shared tables: if grants are scoped, the matrix is bounded by them; if one account holds everything, then inventory-api's `SELECT` on `ORDER_DTL`, settlement-batch's `UPDATE` on `ORDER_MST` and settlement-anomaly's `INSERT` into the settlement schema are unconstrained by anything but code review. [[PROC-INVENTORY-AUTH]] and [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]] both stop at this line.

**Already checked.** No `GRANT` statement, role definition, user creation or DBA script exists in any of the five repositories. All four database-backed services resolve to the same database, `sellflow_order`, and the same user, `sellflow`: order-service and settlement-batch through their `application*.yml`, inventory-api through `app/db.py`'s hardcoded connection (which ignores `DB_URL`, `alembic.ini` and `.env.sample`, all three of which name a separate `inventory` database), and settlement-anomaly through `app/db.py`. Neither Python service passes a password key at all, so pymysql sends an empty password; neither Java service declares one in any committed profile.

**Files a human would need.** `SHOW GRANTS FOR 'sellflow'@'%'` against each environment, from 인프라팀; `inventory-api:app/db.py`; `settlement-anomaly:app/db.py`; `order-service:src/main/resources/application-prod.yml`; `settlement-batch`'s equivalent profile.

## SETTLEMENT-RUN-LOG — Why do two settlement-run tables exist, and has the second ever held a row?

**Question.** `SETTLEMENT_RUN` (V1) and `SETTLEMENT_RUN_LOG` (created by V2, indexed by V6) overlap on four of five columns and use incompatible key types. Was V2 meant to replace V1 or to sit beside it, and has `SETTLEMENT_RUN_LOG` ever held a row in any environment?

**Why it matters.** [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] documents a table created by migration and written by nothing, and cannot say whether that is an abandoned replacement or a table fed from outside the repositories. The answer also bears on [[PROC-SETTLEMENT-FLOW-CATALOG]] and the existing entry "PROC-SETTLEMENT-RUN-LIFECYCLE — What creates a SETTLEMENT_RUN row": if the batch's run history actually lives in `SETTLEMENT_RUN_LOG`, then the writer everyone is looking for may be writing to the other table, and the reef is asking about the wrong one.

**Already checked.** `V2__add_settlement_run_log.sql` and `V6__settlement_run_log_index.sql` carry no comment, no author and no date, unlike `V1` and `V5` which do; Flyway version numbers give an ordering but no calendar, and this repository's comment dates are non-monotonic where they exist. `SETTLEMENT_RUN_LOG.RUN_ID` is `VARCHAR(32)`, which fits a hex UUID and cannot hold `SETTLEMENT_RUN.RUN_ID`'s `BIGINT`. Nothing in any of the five repositories reads or writes `SETTLEMENT_RUN_LOG` — verified by grepping all five. The only String-typed run ids in the codebase are `SettlementReportWriter.write(String runId)` and settlement-anomaly's `AnomalyRow.run_id: str`, and neither references this table, so the connection is a hypothesis and is recorded as one. SF-5099 (정산 배치 실행 이력 알림 개선, 박성민, 2SP) in the 2026-S17 sprint improves a batch execution-history alert whose storage is in none of the repositories and is demonstrably not this table.

**Files a human would need.** 박성민, as SF-5099's assignee, and whoever owns the batch alerting; `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql` and `V6__settlement_run_log_index.sql`; `sources/context/sprints/tickets_2026-S17.csv`; a `SELECT COUNT(*) FROM SETTLEMENT_RUN_LOG` against each environment.
