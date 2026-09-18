---
id: "PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW"
type: "pattern"
title: "The Real Integration Surface Is the Table, Not the API"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "The matrix was built by reading every SQL statement in all five repositories — JdbcTemplate strings, Spring Batch reader SQL, pymysql calls, JPA @Table annotations and every Flyway/alembic migration — and attributing each table to the repo whose migration creates it. It will go stale the moment any service adds a query; the cheapest re-check is to grep each repo for SELECT|INSERT|UPDATE|DELETE and re-attribute the table names found."
freshness_triggers:
  - "inventory-api/app/main.py"
  - "order-service/src/main/resources/db/migration/V1__init.sql"
  - "order-service/src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
  - "settlement-anomaly/app/main.py"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
  - "sources/context/registry/services.yaml"
known_unknowns:
  - "Whether database-level grants restrict any of these accesses in production. No GRANT statement, role definition or DBA script exists in any of the five repos; all four services default to the same user 'sellflow', so the matrix records what the code attempts, not what the server permits."
  - "Whether ORDER_MST.UPD_DTM is used by anything other than the settlement reader's DATE(m.UPD_DTM) = ? predicate. Both order-service and settlement-batch write it, and nothing else in the five repos reads it, but a report or admin tool outside these repos could."
  - "Whether any cross-service access exists that this pass could not see — a stored procedure, a view, a BI tool, a DBA script or the unversioned deploy.sh. The matrix covers the five repositories only."
  - "Whether the 2019 consolidated-database agreement enumerated which cross-owner accesses were sanctioned. Only two accesses cite it (MarkSettledTasklet and the settlement schema header); the agreement text is not in this reef."
tags:
  - coupling
  - cross-system
  - data-flow
  - integration
  - shared-database
aliases:
  - "테이블 직접 접근"
  - "integration surface"
  - "table access matrix"
relates_to:
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-CROSS-SERVICE-AUTH]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-DB-AS-QUEUE]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORPHANED-COMPONENTS]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-OWNERSHIP]]"
  - type: "refines"
    target: "[[SCH-ORDER]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-BATCH]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
    notes: "database hardcoded to sellflow_order"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL — a cross-owner read"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V1__init.sql"
    notes: "ORDER_MST, ORDER_DTL, ORDER_CANCEL — the order team's tables"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "Three-table join across two owners"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "The settlement reader's ORDER_MST JOIN ORDER_DTL"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "Reads and writes the order team's outbox"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "UPDATE ORDER_MST by a non-owner"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "The declared integration surface — two queue entries and three fictional databases"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/runtime.md"
    notes: "Shared-database topology table"
notes: "The matrix is the artifact. Everything else exists to explain why the declared surface in services.yaml describes a small fraction of it."
---

## Overview

Sellflow's service registry declares its integration surface in seven lines: a `queues:` block naming `CANCEL_RECON_QUEUE` and `ORDER_EVENT_OUTBOX`, each with a producer and a consumer. That is the whole of the estate's documented coupling.

The real coupling is a good deal larger, and it is almost entirely invisible from the outside, because it does not travel over a network interface at all. Four of the five services connect to the same MySQL schema, `sellflow_order`, and reach across ownership boundaries with plain SQL. settlement-batch `UPDATE`s the order team's `ORDER_MST`. inventory-api `SELECT`s the order team's `ORDER_DTL`. settlement-anomaly joins across two owners' tables in a single statement. The settlement batch's own item reader joins `ORDER_MST` to `ORDER_DTL` before it writes a single settlement row.

Not one of these appears in `services.yaml`. Three of the five services declare a `db:` value there that does not exist — `MySQL (settlement)`, `MySQL (inventory)`, `MySQL (order)` — where the code shows one shared instance. The declared surface is not merely incomplete; it describes a topology the estate does not have.

This pattern matters because every conclusion an agent draws about blast radius, deployment ordering, schema change safety and failure isolation depends on which surface it is reasoning about.

## Key Facts

- **Four services resolve to one schema.** order-service and settlement-batch both configure `jdbc:mysql://${DB_HOST}:3306/sellflow_order`; inventory-api and settlement-anomaly both hardcode `"database": "sellflow_order"` in their pymysql DSN → order-service:src/main/resources/application.yml, settlement-batch:src/main/resources/application.yml, inventory-api:app/db.py, settlement-anomaly:app/db.py
- **The registry declares three databases that do not exist.** `services.yaml` records `db: MySQL (order)`, `db: MySQL (settlement)` and `db: MySQL (inventory)` as if they were separate, and its `queues:` block is the only integration it names at all → sellflow-docs:context/registry/services.yaml
- **settlement-batch writes the order team's primary table.** `MarkSettledTasklet` runs `UPDATE ORDER_MST SET SANGTAE_CD='JUNGSAN_WANRYO', UPD_DTM=NOW() WHERE ORD_NO IN (SELECT ORD_NO FROM SETTLEMENT_DTL WHERE RUN_ID = (SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN))` — with no status predicate, so it overwrites `CHWISO` and `BANPUM` alike → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java
- That write is authorised by a comment, not a grant: `ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)` — "ORDER_MST is a table owned by the order team, but under the consolidated-DB policy the settlement batch updates it directly (agreed 2019)" → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java
- **inventory-api reads the order team's detail table on every restock.** `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s`, inside an endpoint with no authentication, from a service whose README says of `ORD_NO`: (보관하지 않음) — "not stored" → inventory-api:app/main.py, inventory-api:README.md
- Note the mismatch that read conceals: the query uses `fetch_one`, so an order with several lines yields exactly one row, and the subsequent `UPDATE stock_item` restores the quantity of that single line only → inventory-api:app/main.py
- **settlement-anomaly joins across two owners in one statement:** `FROM SETTLEMENT_DTL d JOIN SETTLEMENT_RUN r ON r.RUN_ID = d.RUN_ID JOIN ORDER_MST m ON m.ORD_NO = d.ORD_NO WHERE r.JUNGSAN_ILJA = %(ilja)s` — two settlement tables and one order table, read by a third team's service → settlement-anomaly:app/main.py
- **The settlement job's own reader is a cross-owner join before any settlement exists:** `FROM ORDER_MST m JOIN ORDER_DTL d ON d.ORD_NO = m.ORD_NO WHERE m.SANGTAE_CD = 'BAESONG_WANRYO' AND DATE(m.UPD_DTM) = ?` → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java
- **settlement-batch also mutates the order team's outbox state.** `OrderEventRelayJob` selects from `ORDER_EVENT_OUTBOX` and then runs `UPDATE ORDER_EVENT_OUTBOX SET PUBLISHED_YN='Y' WHERE EVENT_ID=?` — the consumer writes the producer's table → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **And it reads `SETTLEMENT_DTL` to decide, per event, whether an obligation exists:** `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **One deprecated class reaches the other way.** `OrderCancelServiceV1` — order-service reading settlement's table — ran `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` to block cancellation. It is `@Deprecated` with zero callers, so the order→settlement read direction exists in the source and not in the running system → order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- **HTTP integration, by contrast, is two clients and neither works.** order-service's `InventoryClient` posts to `/inventory/restore`, a path inventory-api does not serve and which no order-service code calls; delivery-bff's `requestCancel` fetches a relative `/api/v1/orders/{ordNo}/cancel`, a prefix order-service does not serve, from a function nothing calls → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java, delivery-bff:src/generated/orderApi.ts
- **So the estate's only functioning cross-service integration is SQL.** Every live cross-boundary data movement in the five repos is a statement against `sellflow_order`; the two HTTP clients that would have moved data are both non-functional → all five repos, verified 2026-09-19
- **The scheduler shares the same instance,** so settlement's trigger state is data in the same schema as the orders: `spring.quartz.job-store-type: jdbc`, `org.quartz.jobStore.isClustered: true` → settlement-batch:src/main/resources/application.yml
- **Ownership and access are declared in different places and never reconciled.** Ownership lives in `services.yaml` (`owner_team`, one of which is `TODO`); access lives in SQL strings scattered across three languages; no file in the estate lists both → sellflow-docs:context/registry/services.yaml

## Where It Appears

### The table access matrix

All tables live in the single `sellflow_order` schema. "Owning service" is the service whose migration creates the table.

| Table | Owning service | Other services that **read** it | Other services that **write** it | Mechanism |
|---|---|---|---|---|
| `ORDER_MST` | order-service (`V1__init.sql`) | settlement-batch (`DailySettlementJobConfig` reader join); settlement-anomaly (`/detect` join) | **settlement-batch** (`MarkSettledTasklet` `UPDATE ... SANGTAE_CD='JUNGSAN_WANRYO'`) | direct SQL, shared instance |
| `ORDER_DTL` | order-service (`V1__init.sql`) | settlement-batch (reader join); **inventory-api** (`SELECT SANGPUM_CD, SURYANG`) | — | direct SQL, shared instance |
| `ORDER_CANCEL` | order-service (`V1__init.sql`) | none found | none found | JPA, owner only |
| `ORDER_EVENT_OUTBOX` | order-service (`V8`, 2023-04-18) | settlement-batch (`OrderEventRelayJob` `SELECT ... PUBLISHED_YN='N'`) | **settlement-batch** (`UPDATE ... PUBLISHED_YN='Y'`) | polled table — see [[PAT-SELLFLOW-DB-AS-QUEUE]] |
| `SETTLEMENT_RUN` | settlement-batch (`V1`) | settlement-anomaly (`/detect` join) | — | direct SQL, shared instance |
| `SETTLEMENT_DTL` | settlement-batch (`V1`) | settlement-anomaly (`/detect`); order-service `OrderCancelServiceV1` (**deprecated, zero callers**) | — | direct SQL, shared instance |
| `CANCEL_RECON_QUEUE` | settlement-batch (`V1`, `V4`, `V5`) | `CancelReconciler` — **unscheduled, never runs**; ad-hoc DBA extracts | — | direct SQL, owner only |
| `SETTLEMENT_ANOMALY` | settlement-anomaly (`V1__anomaly_schema.sql`) | none found | none found | direct SQL, owner only |
| `stock_item` | inventory-api (`sql/V1__stock.sql`) | none found | none found | direct SQL, owner only |
| `PARTNER_CONTRACT` | settlement-batch (`V3`) | `PartnerContractRepository` — **never called**; `0.12` is hardcoded instead | — | would be direct SQL |
| `SETTLEMENT_RUN_LOG` | settlement-batch (`V2`, `V6`) | **none** | **none** | — |
| `RESTORE_LOG` | inventory-api (alembic `3f9a`) | **none** | **none** | — |
| `SETTLEMENT_ADJUSTMENT` | **no migration anywhere** | — | `CancelReconciler` — unscheduled | would be direct SQL |
| Quartz + Spring Batch metadata | settlement-batch (`initialize-schema: always`, JDBC job store) | — | — | framework-managed, same instance |

Six cross-owner accesses are live: two writes (`ORDER_MST`, `ORDER_EVENT_OUTBOX`, both by settlement-batch) and four reads (settlement-batch of `ORDER_MST`+`ORDER_DTL`, inventory-api of `ORDER_DTL`, settlement-anomaly of `ORDER_MST`). None of the six appears in `services.yaml`.

### Declared surface vs real surface

| | Declared in `services.yaml` | Found in code |
|---|---|---|
| Databases | 3 separate (`order`, `settlement`, `inventory`) + `none` | 1 shared instance, `sellflow_order` |
| Integration points | 2 queue entries, one with `consumer: TODO` | 2 polled tables **plus** 6 cross-owner direct accesses |
| Cross-owner writes | 0 | 2 (`ORDER_MST`, `ORDER_EVENT_OUTBOX`) |
| HTTP service-to-service calls | 0 named | 2 clients, both non-functional |
| Authentication on any of it | no field exists | none — see [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]] |

### The flow, end to end

```
order-service            settlement-batch              settlement-anomaly      inventory-api
─────────────            ────────────────              ──────────────────      ─────────────
ORDER_MST ─────read────► reader join                                  ◄─read── (none)
ORDER_DTL ─────read────► reader join                                  ◄─read── SELECT SURYANG
ORDER_MST ◄────WRITE──── MarkSettledTasklet
ORDER_EVENT_OUTBOX ─read/WRITE─► OrderEventRelayJob
                         SETTLEMENT_DTL ──────read────► /detect join
                         SETTLEMENT_RUN ──────read────► /detect join
                         CANCEL_RECON_QUEUE ─► (no consumer)
                                                       SETTLEMENT_ANOMALY (no reader)
```

## Design Intent

**Determinable, and unusually well recorded for this estate — though only in code comments.**

The 2019 consolidation is cited twice, in the two places a reviewer would most need it. The settlement schema header states the topology: `주의: sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)` — "caution: it uses the same instance as sellflow_order (consolidation decision of 2019)". `MarkSettledTasklet` states the licence that topology granted: the order team owns `ORDER_MST`, and under 통합 DB 정책 ("the consolidated-DB policy") settlement updates it directly, 2019년 협의 ("agreed in 2019").

So direct cross-owner table access is not an accident or a shortcut taken behind anyone's back. It is the sanctioned integration mechanism, and the services were built on that assumption — see [[DEC-SELLFLOW-SHARED-DB]].

**Not determinable:** whether the 2019 agreement enumerated *which* accesses were sanctioned, or granted a general licence. Only two accesses cite it; the four others cite nothing at all. inventory-api's read of `ORDER_DTL` in particular carries no justification anywhere, and is in some tension with that service's own README, which lists `ORD_NO` as (보관하지 않음) — "not stored" — as if the order domain were outside its reach. The agreement text is not in this reef, so whether that read was ever sanctioned is unknown and recorded in `known_unknowns` rather than inferred.

**Also not determinable:** whether database grants narrow any of this in production. No `GRANT`, role definition or DBA script exists in any repo, and all four services default to the same `sellflow` user. The matrix therefore records what the code attempts, not necessarily what the server permits.

## Trade-offs

**What it buys.** A great deal, and it should be said plainly. Cross-service reads are transactionally consistent for free, because there is one transaction manager. The outbox is atomic with the cancellation it describes. A join across order and settlement data — exactly what an anomaly detector needs — is one statement with no data pipeline, no replication lag and no schema registry. There is one backup, one restore, one point-in-time recovery. For an organisation of this size, the alternative (five databases, five APIs, an integration layer and eventual consistency everywhere) would have cost more than it returned, and much of what this estate gets wrong would have gone wrong in that world too.

**What it costs.**

1. **Every table is a public API without being declared one.** A column rename is a cross-team breaking change with no compile step to catch it. `V15__rename_bigo_to_memo.sql` followed by `V16__revert_rename_bigo.sql` is what that costs in practice, in one repository, for one column.
2. **Ownership is advisory.** The order team owns `ORDER_MST` in the registry and does not control its contents: `SANGTAE_CD` is overwritten nightly by another division's batch, with no predicate protecting `CHWISO`.
3. **A service's blast radius cannot be read from its interface.** inventory-api looks like a stock service and can read every order line in the estate. settlement-anomaly looks like a read-only detector and holds a connection that can write anything.
4. **The declared surface actively misleads.** An engineer planning a change from `services.yaml` would believe three separate databases exist and that two queues are the only coupling. Both beliefs are false, and both would produce an incorrect impact analysis.
5. **There is no isolation of failure or of load.** A long-running analytical join in settlement-anomaly contends with order writes on the same instance. There is no read replica in any configuration in these repos — the one that exists (`settlement_prod` read replica) appears only in an export README, used by a DBA.
6. **Authorisation cannot be expressed.** Since the integration path is SQL under one shared account, there is nowhere to state "settlement may set `SANGTAE_CD` but not `CHONG_GEUMAEK`". See [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]].
7. **The coupling grows silently.** Adding a cross-owner read requires no dependency, no config, no review and no registry entry — just a table name in a string. That is why six of them exist and none is documented.

## Agent Guidance

- **Start from the matrix, not from the registry.** When asked "what depends on X", `services.yaml` will give a two-line answer that is wrong. Grep all five repos for the table name instead.
- **A table name in a SQL string is a dependency edge.** Treat `ORDER_MST` appearing in settlement-batch exactly as you would treat an HTTP call: it is settlement-batch depending on the order schema, with no version, no contract and no test.
- **Before proposing any schema change, grep all five repos for the table and column name** — including the Python services, whose SQL is in ordinary string literals and will not be found by any Java-aware tool.
- **Do not assume a service only touches "its own" tables.** Three of the four data-touching services read a table they do not own, and one writes two.
- **When answering "who can change this value", answer with writers, not owners.** `ORDER_MST.SANGTAE_CD` has two writers in two repositories owned by two divisions.
- **Treat an unreferenced table as unwritten, not as empty-by-coincidence.** `SETTLEMENT_RUN_LOG`, `RESTORE_LOG` and `SETTLEMENT_ADJUSTMENT` each appear in the matrix with no live accessor; see [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] for how that was verified.
- **Do not model HTTP as the integration layer here.** Both cross-service HTTP clients are non-functional. Any sequence diagram showing order-service calling inventory-api over HTTP describes code that exists and behaviour that does not.
- **When reasoning about consistency, remember there is only one database.** Nothing in this estate is eventually consistent because of replication; the only asynchrony is the 10-minute relay poll.

## Related

- [[DEC-SELLFLOW-SHARED-DB]] — the 2019 decision that made the table the integration surface
- [[PROC-SELLFLOW-OWNERSHIP]] — who owns what, and why ownership is advisory here
- [[CON-ORDER-SETTLEMENT]] — the order/settlement boundary as a contract
- [[CON-ORDER-INVENTORY]] — the order/inventory boundary, including the read of `ORDER_DTL`
- [[CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH]] — the detector's three-table join
- [[PAT-SELLFLOW-DB-AS-QUEUE]] — the two tables that also carry messages
- [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]] — why none of this coupling is authorised
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — the tables in the matrix with no accessor at all
- [[SCH-ORDER]] — the order schema most of this reaches into
- [[SCH-SETTLEMENT-BATCH]] — the settlement schema in the same instance
