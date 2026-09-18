---
id: "PROC-SELLFLOW-RUNTIME"
type: "process"
title: "Sellflow Runtime Topology"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-18
freshness_note: "snorkel-depth scan of configuration, Dockerfiles and CD workflows; nothing here was observed against a running environment"
freshness_triggers:
  - "delivery-bff/package.json"
  - "inventory-api/Dockerfile"
  - "order-service/.github/workflows/order-service-prod-cd.yml"
  - "order-service/src/main/resources/application.yml"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch/src/main/resources/application.yml"
  - "sources/context/registry/services.yaml"
known_unknowns:
  - "How many instances of each service run, and behind what gateway or load balancer; no deployment manifest, Helm chart or docker-compose file exists in any repo"
  - "What deploy.sh does; both Java CD workflows call it and neither repo contains it"
  - "Whether settlement-anomaly is deployed at all — it has no Dockerfile and no CD workflow"
  - "What triggers settlement-anomaly's /detect at 03:00 as its README claims; no scheduler in any repo calls it"
  - "Whether the production Quartz cluster is correctly configured on every node after the 2025-07 incident; the fix is recorded as done but not verified here"
tags:
  - deployment
  - runtime
  - scheduling
aliases:
  - "runtime architecture"
relates_to:
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:Dockerfile"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/workflows/order-service-prod-cd.yml"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application-prod.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
notes: "Assembled from configuration only. No environment was inspected."
---

## Purpose

Describe what actually runs, where, and on what schedule — the processes, their ports, their triggers and the one database they all converge on. This is the orientation artifact for anyone trying to reason about timing, since almost every interesting behaviour in this estate is a consequence of when something runs rather than what it calls.

## Key Facts

- Five deployable units: three HTTP services, one batch process and one HTTP service with no deployment pipeline → `sources/context/registry/services.yaml`
- Ports are fixed in configuration: order-service 8081, inventory-api 8000, delivery-bff 8083, settlement-anomaly 8090 → `order-service/src/main/resources/application.yml`, `inventory-api/Dockerfile`, `delivery-bff/src/index.ts`, `settlement-anomaly/README.md`
- Four of the five processes connect to the same MySQL database, `sellflow_order` → `DEC-SELLFLOW-SHARED-DB`
- settlement-batch runs no web tier: `spring.batch.job.enabled: false` with Quartz as the only entry point → `settlement-batch/src/main/resources/application.yml`
- Two Quartz triggers exist: `dailySettlementTrigger` at cron `0 0 2 * * ?` in `Asia/Seoul`, and `orderEventRelayTrigger` every 10 minutes, repeating forever → `settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`
- Quartz runs in JDBC clustered mode, `org.quartz.jobStore.isClustered: true` → `settlement-batch/src/main/resources/application.yml`
- Both Java CD pipelines build with `-x test` and then invoke a `deploy.sh` that is not in either repo → `order-service/.github/workflows/order-service-prod-cd.yml`
- Production order-service pins a Hikari pool of 40 connections with a 3-second connection timeout against `prod-db.internal` → `order-service/src/main/resources/application-prod.yml`
- Only inventory-api has a Dockerfile; order-service's CD workflow runs `docker build` against a Dockerfile that is not present in the repo → `inventory-api/Dockerfile`, `order-service/.github/workflows/order-service-prod-cd.yml`
- delivery-bff has no CI or CD workflow at all → `delivery-bff/package.json`

## Phases

### Continuous — request-serving processes

| Process | Runtime | Port | Inbound | Outbound |
|---|---|---|---|---|
| order-service | Spring Boot 2.3 / Java 8 | 8081 | admin console (CORS-restricted), gateway | MySQL `sellflow_order` |
| inventory-api | FastAPI / Python 3.9 | 8000 | none observed in code | MySQL `sellflow_order` |
| delivery-bff | Express / Node | 8083 | mobile app | carrier tracking API |
| settlement-anomaly | FastAPI / Python | 8090 | none observed in code | MySQL `sellflow_order` |

### Every 10 minutes — outbox relay

`orderEventRelayJob` polls up to 500 unpublished `order.cancelled` rows, checks each against `SETTLEMENT_DTL`, inserts `PENDING` rows into `CANCEL_RECON_QUEUE`, and marks the outbox rows published. This is the only continuously running integration between two services in the estate.

### Daily 02:00 KST — settlement

`dailySettlementQuartzJob` launches `dailySettlementJob`: `settlementStep` reads yesterday's delivered orders in chunks of 500, computes the fee, writes `SETTLEMENT_DTL`; `markSettledStep` then updates `ORDER_MST` to `JUNGSAN_WANRYO` for the latest run.

The 02:00 hour is load-bearing in ways spread across the estate. Order's `DateUtil.settlementBaseDate()` simply returns yesterday, while Settlement's version subtracts two days when invoked before 02:00 — two utilities with the same name and the same method name, returning different dates. SF-2287's whole justification rests on this schedule: because the batch runs at 2am, an order placed yesterday was already blocked from cancellation by the following morning.

### Daily 03:00 KST — anomaly detection, as documented

The settlement-anomaly README states it is triggered after the settlement batch finishes. No trigger for it exists in any of the five repos. The README's claim and the absence of a caller are both recorded; which is true of the deployed environment is unknown.

### Never scheduled — the correction job

`CancelReconciler` is a Spring `@Component` with no trigger, no endpoint and no caller. It is the only part of the settlement flow that would move money back. See [[PROC-SETTLEMENT-CORRECTION]].

## Worked Example

A cancellation arriving at 09:00 on a weekday, traced through the clock:

1. 09:00 — order-service handles `POST /orders/{ordNo}/cancel`, writes `ORDER_CANCEL`, sets `SANGTAE_CD='CHWISO'`, inserts one `ORDER_EVENT_OUTBOX` row.
2. Within 10 minutes — `orderEventRelayJob` picks the row up. If the order appears in `SETTLEMENT_DTL` (which it will if it was delivered and the 02:00 batch has run), a `PENDING` row lands in `CANCEL_RECON_QUEUE`. The outbox row is marked published either way.
3. Next 02:00 — `dailySettlementJob` runs again. Its reader asks nothing about cancellation, but it selects `SANGTAE_CD='BAESONG_WANRYO' AND DATE(UPD_DTM)=?`, and step 1 overwrote both columns — so this order is not picked up. The exclusion is a side effect of the status write, not a rule the batch applies.
4. Next 03:00 — if the detector is in fact triggered, it joins `SETTLEMENT_DTL` to `ORDER_MST`'s *current* status, so this order matches the `CANCELLED_SETTLED` rule only if it had already been settled on an earlier run (the step-2 case). A `SETTLEMENT_ANOMALY` row is then written, with no defined downstream owner.
5. Never — nothing runs `CancelReconciler`, so the `PENDING` row stays `PENDING`.

The 2025-07-12 incident is the counter-example worth keeping beside this: when the Quartz cluster setting was applied to only one of two nodes during a redundancy change, step 3 ran twice at 02:04, producing duplicate payment requests to 17 partners worth roughly 42 million KRW, discovered by a partner enquiry at 08:40 and settled by offsetting the next month. Two of that incident's follow-up actions are still open, including "지급 요청 전 멱등성 키 도입" (introduce an idempotency key before sending a payment request), unassigned.

## Related

- [[SYS-ORDER]] -- request-serving process
- [[SYS-SETTLEMENT]] -- the scheduled half of the estate
- [[SYS-SETTLEMENT-ANOMALY]] -- documented schedule with no observed trigger
- [[SYS-INVENTORY]] -- request-serving process
- [[SYS-DELIVERY]] -- request-serving process
- [[DEC-SELLFLOW-SHARED-DB]] -- why four processes converge on one database
- [[PROC-SETTLEMENT-DAILY-BATCH]] -- the 02:00 job in detail
- [[PROC-SETTLEMENT-CORRECTION]] -- the job that never runs
