---
id: "PAT-SELLFLOW-DB-AS-QUEUE"
type: "pattern"
title: "The Database Is the Queue — Polled Tables Instead of a Broker"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "The absence of a broker was re-verified on 2026-09-19 by a case-insensitive grep for kafka|rabbit|amqp|sqs|pubsub|celery|redis|nats|activemq|jms across all five repository roots, which returned zero lines; the two build.gradle files, requirements.txt in both Python services and delivery-bff's package.json were also read dependency by dependency. An absence is only as durable as the next dependency addition — re-run that grep before relying on this."
freshness_triggers:
  - "delivery-bff/package.json"
  - "inventory-api/requirements.txt"
  - "order-service/build.gradle"
  - "order-service/src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
  - "settlement-anomaly/requirements.txt"
  - "settlement-batch/build.gradle"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "sources/infra/settlement/queues.md"
known_unknowns:
  - "Whether a broker was ever evaluated and rejected. No ADR, meeting minute or Slack message in this reef discusses the choice; V8's comment states 별도 브로커 없음 as a fact, not as a decision with a rationale."
  - "Whether the 500-row / 10-minute ceiling was chosen deliberately or copied. Neither number is explained in any comment, commit message or document available here."
  - "Whether ORDER_EVENT_OUTBOX is ever pruned. No DELETE against it exists in any repo and no retention job is scheduled, but a DBA-side job outside version control cannot be ruled out from these repositories."
  - "What happens on a relay crash mid-loop in production. The behaviour is inferable from the code (no transaction wrapper is declared) but has not been observed, and no incident record of it exists."
tags:
  - messaging
  - outbox
  - polling
  - quartz
  - shared-database
aliases:
  - "DB 폴링 큐"
  - "table-as-queue"
  - "no broker"
relates_to:
  - type: "depends_on"
    target: "[[DEC-ORDER-OUTBOX-RELAY]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORPHANED-COMPONENTS]]"
  - type: "constrains"
    target: "[[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SELLFLOW-RUNTIME]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:build.gradle"
    notes: "Five dependencies, none of them a messaging client"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
    notes: "The 2023-04-18 migration and its Korean comment stating 별도 브로커 없음"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:README.md"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:build.gradle"
    notes: "batch, quartz, jdbc, flyway, mysql — no broker client"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "The queues block, with consumer: TODO"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Tier-4 extraction of both queues and the Quartz schedules"
notes: "Verification of the broker's absence is recorded in freshness_note as a reproducible command, because the finding is an absence and absences decay."
---

## Overview

Sellflow has two things it calls queues. Neither is a queue in the messaging sense: both are MySQL tables in the shared `sellflow_order` schema, written by one service and polled by a Quartz job in another. There is no broker anywhere in the estate — no Kafka, no RabbitMQ, no SQS, no Redis, no Celery, nothing.

The design is stated plainly in the migration that introduced it. `V8__add_cancel_event_outbox.sql`, written by 박성민 on 2023-04-18, carries the comment `정산 배치의 릴레이 잡이 주기적으로 폴링한다. (별도 브로커 없음)` — "the settlement batch's relay job polls it periodically (no separate broker)". The relay class repeats it: `별도 메시지 브로커 없이 ORDER_EVENT_OUTBOX 를 10분 간격으로 폴링하여 정산 정정 대기열에 적재한다` — "without a separate message broker, it polls ORDER_EVENT_OUTBOX every 10 minutes and loads settlement-correction entries into the queue".

The pattern is worth documenting not because it is unusual — the transactional outbox is a respected design — but because the properties this implementation has are not the properties its users assume it has. The people who built it, and the documents that describe it, talk about publishing and subscribing. What the code does is different in four specific ways, and each of them matters.

## Key Facts

- **No broker exists in any of the five repositories.** A case-insensitive grep for `kafka|rabbit|amqp|sqs|pubsub|celery|redis|nats|activemq|jms` across `order-service`, `settlement-batch`, `inventory-api`, `delivery-bff` and `settlement-anomaly` returns zero lines → verified by grep, 2026-09-19, over all five repository roots
- The dependency lists corroborate it individually: order-service declares five dependencies (`starter-web`, `starter-data-jpa`, `flyway-core`, `mysql-connector-java`, `starter-test`); settlement-batch declares five (`starter-batch`, `starter-quartz`, `starter-jdbc`, `flyway-core`, `mysql-connector-java`); inventory-api pins `fastapi`, `uvicorn`, `pymysql`; settlement-anomaly adds `scikit-learn` and `pydantic`; delivery-bff has `axios` and `express` → order-service:build.gradle, settlement-batch:build.gradle, inventory-api:requirements.txt, settlement-anomaly:requirements.txt, delivery-bff:package.json
- **Queue one — `ORDER_EVENT_OUTBOX`.** Producer: order-service `OrderEventPublisher.publishOrderCancelled`, inside the cancel transaction. Consumer: settlement-batch `OrderEventRelayJob`. Only one event type is ever written, `order.cancelled` → order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **Queue two — `CANCEL_RECON_QUEUE`.** Producer: the same relay job, inserting only when `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` is greater than zero. Consumer: none registered. Its intended consumer, `CancelReconciler`, is a `@Component` that nothing calls → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- **Polling interval and batch ceiling.** The relay trigger is `SimpleScheduleBuilder.simpleSchedule().withIntervalInMinutes(10).repeatForever()` and the query is capped at `LIMIT 500`, giving a hard ceiling of about 3,000 cancel events an hour. Neither number is explained anywhere → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **Acknowledgement means "looked at", not "handled".** The relay runs `UPDATE ORDER_EVENT_OUTBOX SET PUBLISHED_YN='Y' WHERE EVENT_ID=?` unconditionally, outside the `if (settled > 0)` branch that does the actual work. A cancel event for an unsettled order is marked published having caused nothing → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **Delivery is at-most-once, where the design assumed at-least-once.** `executeInternal` declares no `@Transactional` and no try/catch. If the `INSERT` into `CANCEL_RECON_QUEUE` succeeds and the process then dies before the `UPDATE`, the row is re-inserted on the next pass (duplicate); if the process dies between the insert's transaction and the rest of the loop, the remaining events simply wait. But the more consequential direction is the opposite: because the ack is written whether or not anything happened, any failure *inside* the settled branch — an insert error, a `sayuCd` parse producing a value the queue rejects — still leaves a `PUBLISHED_YN='Y'` row that no retry will ever revisit → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **There is no dead-letter anywhere.** No table, column, status value or log path in any of the five repos is a dead-letter destination. A `CANCEL_RECON_QUEUE` row has exactly two states in the code, `'PENDING'` written by the relay and `'PROCESSED'` written only by the unscheduled reconciler; there is no failure state → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- **The payload is written by `String.format` and read back by a fixed-offset substring.** The publisher builds `{"ordNo":"%s","sayuCd":"%s"}`; the relay's `sayuCd(Object payload)` finds `"sayuCd":"` and takes `substring(i + 10, i + 12)`, returning `"00"` if the marker is absent. No JSON parser is involved on either side, so any change to the payload shape silently degrades to a reason code that order-service's own `CancelReason.of` would reject → order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- **The scheduler itself lives in the same database.** `spring.quartz.job-store-type: jdbc` with `org.quartz.jobStore.isClustered: true` puts trigger state in MySQL, so the queue, the consumer's schedule and the business data are all one failure domain and one backup → settlement-batch:src/main/resources/application.yml
- **The producer explicitly disclaims knowledge of the outcome.** `OrderEventPublisher`'s javadoc: `구독 측 처리 결과는 본 서비스에서 추적하지 않는다` — "this service does not track the subscriber's processing result". That is a correct description of an outbox, and it is also why the order team could not have noticed the reconciler was missing → order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- **The vocabulary of brokers survived into the documents even though the broker never existed.** SF-2287's agreement is phrased as publish/subscribe — 박성민: "이벤트 하나 발행하겠습니다. `order.cancelled` 구독하시면 됩니다" ("I'll publish an event; you can subscribe to `order.cancelled`"); 김도윤: "네 컨슈머 붙여놓겠습니다" ("yes, I'll attach a consumer") — and the registry's `queues:` block lists `producer` and `consumer` fields for both tables → sellflow-docs:context/tickets/SF-2287.md, sellflow-docs:context/registry/services.yaml
- That vocabulary is exactly where the estate's central misunderstanding lives. In the 2026 kickoff 김도윤 asked "그럼 차감은 자동으로 들어가고 있는 거죠?" ("so the deduction is going in automatically, right?") and 박성민 answered "저희가 이벤트까지는 보내드리고, 그 뒤는 정산 쪽에서 보시는 걸로 알고 있습니다" ("we send the event, and after that I understand settlement handles it"), with the minute-taker adding 이 부분 서로 인지가 다름. 확인 필요 ("the two sides understand this differently; needs checking") → sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md

## Where It Appears

| | `ORDER_EVENT_OUTBOX` | `CANCEL_RECON_QUEUE` |
|---|---|---|
| Introduced | V8, 2023-04-18, order-service | V1 settlement schema, settlement-batch |
| Producer | `OrderEventPublisher` (in the cancel transaction) | `OrderEventRelayJob` (conditional on a settlement row existing) |
| Consumer | `OrderEventRelayJob` | **none registered** — `CancelReconciler` is unscheduled |
| Transport | `SELECT ... WHERE PUBLISHED_YN='N' AND EVENT_TYPE=? ORDER BY REG_DTM LIMIT 500` | `SELECT ... WHERE STATUS='PENDING' ORDER BY RECV_DTM` (in the unscheduled class) |
| Trigger | Quartz simple trigger, every 10 minutes | none |
| Ack | `PUBLISHED_YN='Y'`, written unconditionally | `STATUS='PROCESSED'`, `PROCESSED_DTM=NOW()` — never written |
| Ordering | by `REG_DTM`, single consumer, no partitioning | n/a |
| Backpressure | 500 rows per pass; a burst above ~3,000/hour accumulates | unbounded — 4,127 rows at last count |
| Retention | no `DELETE` in any repo | no `DELETE` in any repo |
| Dead letter | none | none |
| Idempotency | none — `ORD_NO` has no unique key in either table | none |
| Observability | `log.info("아웃박스 릴레이 처리 {}건", ...)` only when non-empty | `log.info("정정 대상 없음")` / `정정 처리 완료 {}건` — from a class that never runs |

Both instances share the same five properties, which is what makes this a pattern rather than two coincidences: a table plays the role of the topic, a Quartz job plays the role of the consumer group, a status column plays the role of the offset, the ack is a status update rather than a commit, and there is no failure path at all.

## Design Intent

**Determinable in part, and stated rather than argued.**

The intent that *is* recoverable: the outbox was added specifically to let order-service tell settlement about a cancellation without a synchronous call, as part of SF-2287, and the absence of a broker was a stated premise rather than a conclusion. V8's comment and the relay's javadoc both assert 별도 브로커 없음 / 별도 메시지 브로커 없이 as a given. The shared database made it nearly free: both services already had a connection to `sellflow_order`, so an outbox table needed no new infrastructure, no new credential, and no new operational surface — see [[DEC-SELLFLOW-SHARED-DB]].

The intent that is **not determinable**: why. No ADR, meeting minute, Slack message or ticket in this reef discusses a broker being considered, costed or rejected. The 2023-04 Slack channel goes straight from deployment to consumer work without any architecture discussion. Whether this was a considered choice, an organisational constraint, or simply the path of least resistance for a team already inside one database cannot be established from the material available, and is recorded in `known_unknowns`.

Two specific numbers are likewise unexplained. Neither `LIMIT 500` nor `withIntervalInMinutes(10)` is justified in a comment, a document or a commit message. They may be deliberate sizing or they may be defaults that were never revisited.

## Trade-offs

**What this design genuinely buys — and in a single-instance MySQL estate it is more than usual.**

1. **Real transactional atomicity, for free.** Because order-service writes `ORDER_CANCEL`, `ORDER_MST` and the outbox row inside one `@Transactional` against one database, the event cannot exist without the cancellation and the cancellation cannot exist without the event. A broker would have reintroduced the dual-write problem that outboxes exist to solve, and would have needed this same table anyway.
2. **The consumer is inside the same transaction boundary as its side effect.** `OrderEventRelayJob` reads the outbox and writes `CANCEL_RECON_QUEUE` in one database. With a broker, delivery and the downstream insert would be two systems and would need their own idempotency scheme.
3. **No new operational surface.** No cluster to run, patch, monitor, secure or fail over. Given that the estate has no service-to-service authentication at all (see [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]]), adding a broker would have added a second unauthenticated hop rather than removing one.
4. **The queue is queryable with the tools everyone already has.** The 2026-09-01 finance extraction that finally quantified the backlog was one `GROUP BY` against a table. That measurement would have been materially harder against a broker's retained log.
5. **Backups and point-in-time recovery cover the messages.** Whatever restores `sellflow_order` restores the in-flight events too.

**What it costs.**

1. **"Delivered" and "handled" are the same bit, and it means the weaker of the two.** `PUBLISHED_YN='Y'` is written whether or not the settled branch ran. There is no state in the system that distinguishes *relayed and acted on* from *relayed and ignored*, which is precisely the distinction the SF-2287 agreement depended on.
2. **At-most-once semantics under a design that assumed at-least-once.** The participants' own words — 발행 (publish), 구독 (subscribe), 컨슈머 (consumer) — carry an expectation of redelivery on failure. There is no redelivery. An event whose handling fails after the ack is gone.
3. **No dead-letter, so no failure is ever surfaced.** A broker's dead-letter queue would have made the missing reconciler visible within days, because rows would have piled up in a place designed to be alarming. Here they piled up in a place designed to look normal — 4,127 of them, over 41 months, in a `PENDING` state that is also the correct state for a row that arrived five minutes ago.
4. **No consumer registry, so a missing consumer is undetectable.** A broker knows whether a subscription exists. A table does not. `services.yaml` records the gap as `consumer: TODO   # 확인 필요` ("needs checking") and that TODO has outlived the people who wrote it.
5. **Throughput is capped and the cap is silent.** 500 rows per 10 minutes is ample today (roughly 160 cancel-after-settle rows a *month*), but nothing alarms if the outbox backlog grows, because nothing measures the outbox backlog.
6. **Unbounded growth with no retention.** No `DELETE` exists against either table in any repo. `ORDER_EVENT_OUTBOX` accumulates every cancellation ever made, and `CANCEL_RECON_QUEUE` accumulates every unmet obligation, forever.
7. **One failure domain.** The Quartz job store, the queues and the business data share an instance. A database incident does not degrade messaging; it stops it, silently, with no buffer.

**The fair summary.** The outbox half of this design is sound and would survive review. The consumer half is where the properties silently weakened: an unconditional ack, no retry, no dead letter, no consumer registration, and a schedule that one of the two jobs never got. It is worth being precise about which half failed, because "we should have used Kafka" is the wrong lesson — a broker would not have scheduled `CancelReconciler` either.

## Agent Guidance

- **Do not describe this estate as event-driven.** It has two polled tables. Using broker vocabulary in an answer imports guarantees — redelivery, dead-lettering, consumer-group liveness — that do not exist here, and that misunderstanding is already documented as costing this organisation money.
- **`PUBLISHED_YN='Y'` proves the relay ran, and nothing else.** Never read it as evidence that settlement acted. To know whether an event produced an obligation you must check for a `CANCEL_RECON_QUEUE` row; to know whether the obligation was met you must check `STATUS`, and today the answer is always `PENDING`.
- **A `PENDING` row is not a recent row.** The status has no age semantics. Join on `RECV_DTM` before drawing any conclusion about freshness.
- **Assume no retry.** When reasoning about a failure scenario in the relay, the default is that the event is lost, not that it comes back. Ask "what marked it published" before asking "when will it be retried".
- **Before claiming any table is a queue with a consumer, find the `@Bean` that registers the trigger.** In this estate, `QuartzConfig` is the only place a job becomes real. A `@Component` with a plausible name and a working method is not a scheduled job — see [[PAT-SELLFLOW-ORPHANED-COMPONENTS]].
- **Check `services.yaml`'s `queues:` block against `QuartzConfig`, not instead of it.** The registry is the declared surface; the Quartz beans are the real one. Where they disagree, the beans win.
- **When estimating latency, the floor is the polling interval.** Cancel-to-queue latency is 0–10 minutes plus relay duration, not milliseconds. Any timeline that shows the queue row appearing "immediately" is wrong.
- **If asked to add a queue here, follow the pattern rather than introducing a broker** — unless the request is explicitly about infrastructure. A sixth service with a Kafka client would be the only broker in the estate and would need the authentication story that [[PAT-SELLFLOW-CROSS-SERVICE-AUTH]] shows does not exist.

## Related

- [[DEC-ORDER-OUTBOX-RELAY]] — the outbox-and-relay decision recorded at decision level
- [[DEC-SELLFLOW-SHARED-DB]] — the 2019 consolidation that made a table cheaper than a broker
- [[PROC-ORDER-EVENT-OUTBOX-LIFECYCLE]] — the outbox row's own lifecycle
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — the queue row's lifecycle, and why it has no terminal state
- [[PROC-SELLFLOW-RUNTIME]] — the Quartz schedules and how the processes run
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what the missing consumer has accumulated
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — the unscheduled reconciler among its peers
- [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]] — the wider pattern of coupling through tables rather than interfaces
