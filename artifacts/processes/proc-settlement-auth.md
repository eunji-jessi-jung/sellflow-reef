---
id: "PROC-SETTLEMENT-AUTH"
type: "process"
title: "Settlement Authentication and Access Control"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened 2026-09-19 by a full read of both settlement repos, their Flyway migrations and all three CD workflows, plus a grep for the strings password/passwd/secret/token/credential across both repositories, which returned nothing. The findings here are absences, and an absence is only as reliable as the search that established it — re-check if a security starter, a gateway manifest, a Dockerfile or a secret-store reference appears."
freshness_triggers:
  - "settlement-anomaly/.env.template"
  - "settlement-anomaly/app/db.py"
  - "settlement-anomaly/app/main.py"
  - "settlement-batch/.env.dev"
  - "settlement-batch/.github/workflows/settlement-batch-prod-cd.yml"
  - "settlement-batch/build.gradle"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - "settlement-batch/src/main/resources/application.yml"
known_unknowns:
  - "Whether settlement-anomaly's port 8090 is reachable from outside the cluster; no ingress, gateway or network policy exists in any repo"
  - "Who or what calls POST /detect, and with what credential — the README claims a 03:00 trigger and no caller was found in any of the five repos"
  - "Which MySQL account the production processes actually use. settlement-batch reads DB_USER (default sellflow) and settlement-anomaly reads DB_USER (default sellflow); only .env.dev, a dev file, names a distinct account (settle_dev). No prod env file exists in either repo."
  - "Where the database password comes from. Neither repo contains a password property, a secret reference or a secret-store client, and both connection paths would otherwise authenticate with an empty password."
  - "Whether the settlement-batch GitHub repository has environment protection rules, required reviewers or deployment branch restrictions. None are declared in the workflow files, and repository settings are not in version control."
  - "What deploy.sh does with credentials. It is invoked by all three workflows and is not present in the repository."
  - "What authorisation governs the 정산 어드민 (settlement admin) the procedure document and the handover refer to; no such application exists in any of the five repos"
tags:
  - auth
  - security
  - settlement
  - supply-chain
aliases:
  - "settlement access control"
relates_to:
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[PROC-ORDER-AUTH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-ANOMALY]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "the only description of human access control found anywhere"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "§3 responsibilities and §5 record retention — the human authorisation model"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "what write access to the Quartz tables is worth in KRW"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "extracted runtime facts: env vars, deploy path, shared schema"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:.env.template"
    notes: "DSN with an empty password; MODEL_PATH never read"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.env.dev"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.github/workflows/settlement-batch-dev-cd.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:.github/workflows/settlement-batch-prod-cd.yml"
    notes: "workflow_dispatch, no permissions block, no secrets, calls an absent deploy.sh"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:build.gradle"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "writes another team's table with no service-level authorisation"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "the 2019 consolidation note that makes the cross-team write legitimate"
notes: "An honest artifact about an absence. The only access control in this domain that is written down anywhere is a human process, and it governs people rather than services. Updated 2026-09-19 to resolve four of the original five known_unknowns."
---

## Purpose

Record how the two settlement services authenticate callers and authorise actions. The short answer is that one of them has no callers to authenticate and the other authenticates nobody. The longer answer is about three surfaces that do exist and are not usually thought of as auth surfaces: the database account, the Quartz scheduler tables, and the deployment workflow. Each of those is, in this system, a path to moving partner money.

## Key Facts

- settlement-batch has no web layer at all: `build.gradle` declares `spring-boot-starter-batch`, `-quartz`, `-jdbc` and `flyway-core`, and no web or security starter, so there is no HTTP surface to authenticate → `settlement-batch:build.gradle`
- Its only trigger surface is Quartz, whose state lives in the shared database (`job-store-type: jdbc`, `org.quartz.jobStore.isClustered: true`), so anything able to write the Quartz tables can cause a settlement run → `settlement-batch:src/main/resources/application.yml`
- What that is worth is on record: on 2025-07-12 an unintended second run produced duplicate payment requests for 17 partners, roughly 42,000,000 KRW, and the resolution was "중복 지급 요청 취소 불가 확인, 차월 상계로 처리 결정" — "confirmed the duplicate payment requests cannot be cancelled; decided to handle it by next-month offset" → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- **Resolved (was a known_unknown): there is no credential material in either repository at all.** Grepping `password`, `passwd`, `secret`, `token` and `credential` case-insensitively across every file of settlement-batch and settlement-anomaly returns zero hits → verified by grep, 2026-09-19, over `settlement-batch:` and `settlement-anomaly:` repository roots
- `application.yml` configures `spring.datasource.url` and `spring.datasource.username` and declares no `password` property, so as committed the JDBC connection authenticates with no password unless one arrives through an environment override that no file in the repo documents → `settlement-batch:src/main/resources/application.yml`
- settlement-anomaly's DSN dictionary is `host`, `user`, `database`, `charset`, `cursorclass` — with no `password` key, which pymysql sends as an empty password → `settlement-anomaly:app/db.py`
- The committed template agrees: `DB_URL=mysql://anomaly:@localhost:3306/settlement` has an empty password between the colon and the `@`. It is also never read — `app/db.py` reads `DB_HOST` and `DB_USER` and hardcodes `database="sellflow_order"` → `settlement-anomaly:.env.template`, `settlement-anomaly:app/db.py`
- **Partly resolved: the dev account is distinct, the prod account is not evidenced.** `.env.dev` sets `DB_USER=settle_dev`, and unlike `DB_URL` that variable *is* consumed, because `application.yml` interpolates `${DB_USER:sellflow}`. Both services otherwise fall back to the same shared `sellflow` user, and no prod env file exists in either repo → `settlement-batch:.env.dev`, `settlement-batch:src/main/resources/application.yml`, `settlement-anomaly:app/db.py`
- A development database hostname and username are committed to version control: `DB_URL=jdbc:mysql://settlement-db-dev.internal:3306/settlement`, `DB_USER=settle_dev` → `settlement-batch:.env.dev`
- **The batch reads and writes another service's tables with no service-level authorisation of any kind.** `MarkSettledTasklet` issues `UPDATE ORDER_MST SET SANGTAE_CD='JUNGSAN_WANRYO'` directly against 주문팀's table; there is no API call, no token, no grant check — only a database connection → `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`
- That is by agreement rather than by accident, and the agreement is recorded in a code comment and a migration header, not in an access-control system: "ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)" — "ORDER_MST is a table owned by the order team, but under the consolidated-DB policy the settlement batch updates it directly (agreed in 2019)" → `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- The reverse direction is equally ungated: `OrderEventRelayJob` reads `ORDER_EVENT_OUTBOX` and marks rows `PUBLISHED_YN='Y'`, i.e. the settlement service mutates the order service's outbox state, again by direct SQL → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- settlement-anomaly's two routes carry no dependency, middleware or header check; `POST /detect` will scan a settlement date and write `SETTLEMENT_ANOMALY` rows for any caller that can reach port 8090, and `GET /health` discloses the loaded model version string to the same population → `settlement-anomaly:app/main.py`
- The gate on `/detect` is therefore network reachability alone, and no ingress, service mesh policy, firewall rule or Kubernetes manifest exists in any of the five repos to describe what that reachability is → `sellflow-docs:infra/settlement/runtime.md`, `settlement-anomaly:app/main.py`
- **Resolved: the deployment workflows carry no permission or secret declarations.** All three CD workflows are `on: workflow_dispatch` with no `permissions:` block, no `environment:` gate, no reviewer requirement and no `secrets.*` reference anywhere — so they run with the repository's default `GITHUB_TOKEN` permissions and any account with write access can dispatch the production job → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`, `settlement-batch:.github/workflows/settlement-batch-dev-cd.yml`, `settlement-batch:.github/workflows/settlement-batch-stage-cd.yml`
- Because the workflows reference no secrets, whatever credential reaches production must be held inside `./deploy.sh $ENVIRONMENT` — a script that is invoked by all three workflows and is not in the repository, so the production deployment's credential handling is not in version control and cannot be reviewed → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`, `sellflow-docs:infra/settlement/runtime.md`
- The prod workflow also builds with `./gradlew clean build -x test`, so the only automated gate between a dispatch and a money-moving deployment is compilation → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`
- settlement-anomaly has no CI or CD workflow, no Dockerfile and no deploy script at all, so how the running process is started and what identity it runs as is not described anywhere in version control → `sellflow-docs:infra/settlement/runtime.md`
- The only access model written down anywhere is a human one, and it is incomplete: the handover's permission table lists 정산 DB (read) via an infrastructure-team request, 정산 어드민 correction rights "팀장 승인 후" ("after team-leader approval"), and 파트너 포털 access marked `TBD` → `sellflow-docs:context/handover/2025-03_정산팀_인수인계.md` (§4)
- The published procedure assigns responsibility rather than permission — 정산팀 for confirming and deducting, 주문팀 for publishing cancel events, 재무기획팀 for checking the uncorrected balance at quarter close — and requires correction history to be kept in the 정산 어드민 for five years, in an application that exists in no repo → `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md` (§3, §5)

## Steps

There is no authentication step to describe, so what follows is the trust chain as it actually stands, ordered from the weakest link outwards.

1. **The database account is the real principal.** Four services resolve to the same `sellflow_order` schema, and settlement's two are among them. Neither repo carries a password, so the account is either passwordless or configured outside version control. A single account that can `INSERT INTO SETTLEMENT_DTL`, `UPDATE ORDER_MST` and write the Quartz tables is, functionally, the settlement system's administrator → `settlement-batch:src/main/resources/application.yml`, `settlement-anomaly:app/db.py`
2. **Scheduling is authorisation.** Quartz stores its triggers in that same database in clustered JDBC mode. Write access to those tables is the ability to cause a payout run, and the 2025-07-12 incident priced an unintended run at roughly 42 million KRW with no recall path → `settlement-batch:src/main/resources/application.yml`, `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
3. **Cross-service writes are unmediated.** The batch updates `ORDER_MST` and `ORDER_EVENT_OUTBOX` — both owned by 주문팀 — with raw SQL under a 2019 verbal agreement. There is no grant, no API, no audit of who changed an order's state, and nothing technically prevents the same connection from writing any other table in the schema → `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
4. **The deployment workflow is an unreviewed path to production.** `workflow_dispatch` with no environment protection declared, no required reviewer, no test step and no secrets in the file. The credential lives in an absent script. Whoever can press the button can ship code into the daily payout → `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`
5. **Network reachability is the only control on `/detect`.** Neither anomaly route checks who is calling. Whatever isolation exists is provided by infrastructure that no manifest in any repo describes → `settlement-anomaly:app/main.py`
6. **Human authorisation sits entirely outside the code.** Correction rights are granted by team-leader approval and exercised through an admin application that is in none of the five repos. Whether that application enforces the approval, and whether it writes the five-year record the procedure requires, is not verifiable from here → `sellflow-docs:context/handover/2025-03_정산팀_인수인계.md`, `sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md`

## Worked Example

**Calling the detector.** A caller wanting an anomaly pass needs only network access:

```
POST http://settlement-anomaly:8090/detect
Content-Type: application/json

{"jungsan_ilja": "2026-09-17"}
```

No `Authorization` header is required or inspected. The response reports how many anomalies were written:

```json
{"detected": 3, "jungsan_ilja": "2026-09-17"}
```

The same call made twice writes the same anomalies twice: the INSERT has no uniqueness guard and `SETTLEMENT_ANOMALY` has no unique constraint beyond its auto-increment primary key → `settlement-anomaly:app/main.py`, `settlement-anomaly:sql/V1__anomaly_schema.sql`

**Reading the model version without credentials.** `GET /health` returns `{"status": "ok", "model": detector.version}`, where the version string comes from inside the pickle file. An unauthenticated caller therefore learns which model artefact is deployed → `settlement-anomaly:app/main.py`, `settlement-anomaly:model/detector.py`

**Deploying to production.** The entire gate, as committed:

```yaml
on:
  workflow_dispatch:
env:
  ENVIRONMENT: prod
jobs:
  deploy:
    steps:
      - uses: actions/checkout@v3
      - run: ./gradlew clean build -x test
      - run: ./deploy.sh $ENVIRONMENT
```
→ `settlement-batch:.github/workflows/settlement-batch-prod-cd.yml`

No `permissions:`, no `environment:`, no reviewer, no tests, and no secret — meaning the deployment credential is inside `deploy.sh`, which is not in the repository.

## Related

- [[SYS-SETTLEMENT]] -- the batch service with no web surface
- [[SYS-SETTLEMENT-ANOMALY]] -- the unauthenticated HTTP service
- [[PROC-SETTLEMENT-DAILY-BATCH]] -- what a scheduler trigger actually causes
- [[RISK-SETTLEMENT]] -- committed configuration, the skipped tests and the duplicate-run incident
- [[RISK-SETTLEMENT-ANOMALY]] -- the ungated endpoint and the unsigned pickle
- [[PROC-ORDER-AUTH]] -- the same absence on the order side, where a specification at least claims bearer auth
- [[DEC-SELLFLOW-SHARED-DB]] -- why the database account is the real trust boundary
