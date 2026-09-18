# Runtime — Settlement

> From `build.gradle`, `application.yml`, `.env.dev`, the Quartz/Batch configuration and the
> GitHub Actions workflows in `settlement-batch`; and from `requirements.txt`, `.env.template` and
> `README.md` in `settlement-anomaly`. Tier 4 (code reading).

## settlement-batch

| | |
|---|---|
| Stack | Java 8 (`sourceCompatibility 1.8`), Spring Boot 2.3.12, Spring Batch 4, Quartz, Flyway, MySQL connector |
| Version | `1.6.2` (build.gradle) |
| Entry point | `SettlementBatchApplication` — no web starter, no HTTP port |
| Scheduler | Quartz, JDBC job store, clustered |
| Batch | `spring.batch.initialize-schema: always`, `spring.batch.job.enabled: false` |
| Flyway | enabled, `classpath:db/migration` (V1–V6) |
| DB | `jdbc:mysql://${DB_HOST:localhost}:3306/**sellflow_order**` — the shared order instance |
| Deploy | `.github/workflows/settlement-batch-{dev,stage,prod}-cd.yml`, all `workflow_dispatch` only, `./gradlew clean build -x test` then `./deploy.sh $ENVIRONMENT` |

**Env vars:** `DB_HOST`, `DB_USER` (application.yml). `.env.dev` adds `DB_URL`, `DB_USER`,
`QUARTZ_ENABLED`.

**Contradictions to flag:**
- `.env.dev` points at `jdbc:mysql://settlement-db-dev.internal:3306/**settlement**` — a separate
  settlement database — while `application.yml` hardcodes the `sellflow_order` schema and reads
  only `DB_HOST`, not `DB_URL`. `.env.dev`'s `DB_URL` therefore has no effect.
- `QUARTZ_ENABLED` is read by nothing in the repo.
- Production deploys are manual-only (`workflow_dispatch`); there is no Dockerfile in the repo,
  yet `deploy.sh` (also absent) is invoked. The prod deploy path is not fully in version control.
- No test step: every environment builds with `-x test`.
- Prod Quartz is clustered, so two instances share the trigger state; nothing else guards against
  a double run beyond Spring Batch's `ts` job parameter making each launch unique — which, if
  anything, permits re-runs rather than preventing them.

## settlement-anomaly

| | |
|---|---|
| Stack | FastAPI 0.104.1, uvicorn 0.24.0, scikit-learn 1.3.2, pymysql 1.1.0, pydantic 2.5.2 |
| Version | `1.4.0` (`FastAPI(version=...)`) |
| Python | README says 3.9, service registry says 3.11 — unresolved, no Dockerfile or `.python-version` in the repo |
| Run | `uvicorn app.main:app --port 8090` (README) |
| DB | `app/db.py` hardcodes host/user from env and `database="sellflow_order"` |
| Model artefact | `model/artifacts/iforest_v3.pkl`, loaded at import time — **not present in the repo** |
| Deploy | no CI workflow, no Dockerfile in the repo |

**Env vars read by code:** `DB_HOST`, `DB_USER`. Declared in `.env.template` but unused:
`DB_URL` (points at a `settlement` database), `MODEL_PATH` (the path is hardcoded in `main.py` and
differs from the template's `./model/detector.pkl`).

## Shared-database topology

All three settlement-touching services plus order-service and inventory-api resolve to the same
MySQL schema, `sellflow_order`:

| Service | Config | Schema reached |
|---|---|---|
| order-service | `application.yml` / `application-prod.yml` (`prod-db.internal`) | sellflow_order |
| settlement-batch | `application.yml` | sellflow_order |
| settlement-anomaly | `app/db.py` (hardcoded) | sellflow_order |
| inventory-api | `app/db.py` (hardcoded) | sellflow_order |

The service registry instead lists four separate databases (`MySQL (order)`, `MySQL (settlement)`,
`MySQL (inventory)`). The code contradicts the registry. V1 of the settlement schema states the
reason: `sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)`.
