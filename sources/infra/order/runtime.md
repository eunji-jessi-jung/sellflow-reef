# Runtime — Order (order-service)

> From `build.gradle`, `SERVER_VERSION`, `src/main/resources/application*.yml`, `.env.example` and
> the five GitHub Actions workflows. Tier 4 (code reading).

| | |
|---|---|
| Stack | Java 8, Spring Boot 2.3.12, spring-boot-starter-web, spring-data-jpa (Hibernate, MySQL5InnoDBDialect), Flyway, MySQL connector |
| Version | `SERVER_VERSION` = 2.8.14; `build.gradle` version = 2.8.4 — **the two disagree** |
| Port | 8081 (`server.port`), no context path |
| DB | `jdbc:mysql://${DB_HOST}:3306/sellflow_order` (`serverTimezone=Asia/Seoul`); prod: `prod-db.internal`, Hikari pool 40, connection-timeout 3000ms |
| Schema management | Flyway `classpath:db/migration` (V1–V24); `jpa.hibernate.ddl-auto: none` |
| CORS | `WebConfig` — `/api/**` restricted to `https://admin.sellflow.co.kr`. `/orders/**` (including the cancel endpoint) has no CORS mapping; comment: `앱은 게이트웨이를 통해 들어온다.` |
| Auth | **none in this repo** — no Spring Security dependency, no filter, no method security. The 2022 spec declares a global `bearerAuth`. Presumed terminated at a gateway that is not in these five repos. Unverified. |
| Profiles | local (example only), dev, qa, stage, prod |

**Env vars** (`.env.example`): `SPRING_PROFILES_ACTIVE`, `DB_URL`, `DB_USER`, `DB_PASSWORD`,
`PG_BASE_URL`, `INVENTORY_BASE_URL`. Note `.env.example`'s `DB_URL` names a database called
`order`, which no `application*.yml` uses — they all target `sellflow_order`. `INVENTORY_BASE_URL`
is read directly via `System.getenv()` in `InventoryClient` (default
`http://inventory-api.internal`); `.env.example` sets it to port 8082 locally, while inventory-api's
own Dockerfile serves on 8000.

## Deployment

`.github/workflows/`: `ci.yml`, `order-service-{dev,qa,stage,prod}-cd.yml`, plus
`migration-issue.yml`. Prod deploys on push to `main` or manual dispatch; builds with
`./gradlew clean build -x test` (tests skipped), builds a Docker image tagged
`registry.sellflow.co.kr/order-service:prod-<sha>`, pushes it, then runs `./deploy.sh prod`.

Neither `Dockerfile` nor `deploy.sh` is present in the repo, so the image contents and the deploy
mechanism are not in version control here.

## Outbound dependencies

| Target | Call | Note |
|---|---|---|
| inventory-api | `POST {INVENTORY_BASE_URL}/inventory/restore?ordNo=..&reason=..` (`InventoryClient`) | Path does not exist in inventory-api — see `sources/apis/inventory/openapi.meta.json`. `InventoryClient` is also not called by `OrderCancelService`. |
| PG | `PaymentClient` / `PG_BASE_URL` | present in `.env.example`; not traced in this pass |
| settlement-batch | indirect, via the `ORDER_EVENT_OUTBOX` table | see `sources/infra/settlement/queues.md` |
