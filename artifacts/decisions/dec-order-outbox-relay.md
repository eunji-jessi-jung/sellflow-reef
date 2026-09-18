---
id: "DEC-ORDER-OUTBOX-RELAY"
type: "decision"
title: "Database Outbox for Settlement Notification"
domain: "order"
status: "draft"
last_verified: 2026-09-18
freshness_note: "Reconstructed on 2026-09-18 from SF-2287's comment thread, the V8 migration comment, the publisher javadoc and settlement-batch's relay job. This is a reconstructed ADR — no architecture decision record exists in the reef for it — so it is only as good as those four artefacts; a real ADR, if one surfaces, supersedes this."
freshness_triggers:
  - "src/main/java/kr/co/sellflow/order/event/OrderEventOutbox.java"
  - "src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - "src/main/resources/db/migration/V11__outbox_index.sql"
  - "src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
known_unknowns:
  - "The rationale for choosing polling over a message broker is not recorded anywhere I could find. SF-2287's thread jumps from 박성민 offering an event to 김도윤 accepting a consumer, with no discussion of transport; V8 states the outcome ('별도 브로커 없음') without a reason. Whether a broker existed and was rejected, or was never on the table, is unknown."
  - "Whether any message broker exists anywhere in the 셀플로우 stack. None of the five registered services declares one."
  - "Who owns the relay. It lives in settlement-batch (정산팀) but implements order-service's publication step; the registry notes only '아웃박스 릴레이 잡 동거' ('the outbox relay job cohabits')."
  - "Why V11's index was needed. The migration says the relay had slowed down but records no volume figures or query plan."
  - "What happens to an outbox row whose relay processing fails midway. The relay flips PUBLISHED_YN to 'Y' outside any visible transaction boundary with the CANCEL_RECON_QUEUE insert."
  - "Whether outbox rows are ever purged. No retention policy, archival job or cleanup migration exists."
  - "Whether the volume assumption behind the manual compensation ('월 10건 미만' / fewer than ten a month) has held since 2023. No follow-up measurement exists in the reef."
  - "Whether the decision was ever reviewed. TASK.md still lists '정산팀 컨슈머 확인' ('confirm the settlement team's consumer') as open, and the registry lists CANCEL_RECON_QUEUE's consumer as TODO."
tags:
  - "order"
  - "outbox"
  - "integration"
  - "sf-2287"
  - "adr-reconstructed"
aliases:
  - "order.cancelled outbox"
  - "아웃박스 릴레이"
relates_to:
  - type: "integrates_with"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[SCH-ORDER]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:TASK.md"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "The polling consumer, owned by 정산팀."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Queue registry: ORDER_EVENT_OUTBOX producer/consumer. Reef-root relative."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The conversation the decision came out of. Reef-root relative."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventOutbox.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V11__outbox_index.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V8__add_cancel_event_outbox.sql"
notes: "No ADR document exists for this decision; it is reconstructed from a ticket thread and code comments. The Rationale section says what was recorded and marks the rest as unrecorded."
---

# Database Outbox Polled by the Settlement Batch

## Context

In March 2023 the CS team logged 214 enquiries about cancellations that order-service refused. The cause was a policy working exactly as designed: settled orders could not be cancelled, and since the settlement batch runs at 02:00 KST, "주문 다음날 오전이면 이미 차단" ("by the morning after the order, it is already blocked") (SF-2287). The alternative offered to customers was the return process, which SF-2287 records as taking 7–10 days.

SF-2287 asked for the settlement check to be removed. 박성민 (주문팀) accepted the code change immediately but raised the consequence on 2023-04-05: "정산이 이미 나간 건에 대해 취소가 들어오면 그 돈은 어떻게 되나요? 파트너한테 이미 지급된 금액인데요." ("if a cancellation comes in for something already settled, what happens to that money? It has already been paid to the partner.")

김도윤 (정산팀) answered on 2023-04-07 that the settlement team would correct such cases by hand, deducting them from the following month, and that they already did so for a few cases monthly. He estimated the future volume as "월 10건 미만" ("fewer than ten cases a month").

That answer created a requirement that did not exist before: the settlement team now needed to know when a cancellation happened. On 2023-04-10, 박성민 proposed the mechanism — "그럼 취소 발생 시 정산팀에 알림이 가야 할 것 같은데요. 이벤트 하나 발행하겠습니다. `order.cancelled` 구독하시면 됩니다." ("then a notification should go to the settlement team when a cancellation occurs; I will publish an event, and you can subscribe to `order.cancelled`"). 김도윤 replied the next day: "네 컨슈머 붙여놓겠습니다." ("yes, I will attach a consumer.")

The thread ends there. Between "I will publish an event" and the table that shipped a week later, no discussion of transport is recorded.

## Decision

order-service writes cancellation events to a database table, `ORDER_EVENT_OUTBOX`, inside the same transaction as the cancellation itself. There is no message broker. A Quartz job inside settlement-batch polls the table every ten minutes, and the two services communicate through the shared MySQL instance they already share for everything else.

The migration that created the table states the shape of the decision plainly: "정산 배치의 릴레이 잡이 주기적으로 폴링한다. (별도 브로커 없음)" ("the settlement batch's relay job polls periodically — no separate broker"), authored by 주문팀 박성민 on 2023-04-18.

## Key Facts

- The decision is recorded only in a migration comment and a javadoc; no ADR or design document for it exists in the reef → src/main/resources/db/migration/V8__add_cancel_event_outbox.sql
- `V8__add_cancel_event_outbox.sql` states the absence of a broker explicitly: "별도 브로커 없음" ("no separate broker") → src/main/resources/db/migration/V8__add_cancel_event_outbox.sql
- The migration ties itself to the ticket in its first line: "SF-2287: 정산 완료 주문 취소 허용" ("SF-2287: allow cancellation of settled orders") → src/main/resources/db/migration/V8__add_cancel_event_outbox.sql
- Only one event type is ever published — `EVT_ORDER_CANCELLED = "order.cancelled"` → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- The publisher's javadoc records the attribution and the subscriber: "SF-2287 로 추가됨. 정산팀이 order.cancelled 를 구독한다." ("added by SF-2287; the settlement team subscribes to order.cancelled") → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- The same javadoc states the tracking limitation: "구독 측 처리 결과는 본 서비스에서 추적하지 않는다." ("this service does not track the subscriber's processing result") → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- It also describes a relay that forwards to a broker — "별도 릴레이가 메시지 브로커로 전송한다" ("a separate relay sends it to a message broker") — which contradicts both the migration comment and the actual relay, since no broker is involved → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- The payload is built by `String.format` rather than a JSON serialiser: `{"ordNo":"%s","sayuCd":"%s"}` → src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java
- Publication is part of the cancel transaction, so an event cannot exist without its cancellation → src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The consumer lives in settlement-batch and polls at ten-minute intervals: "ORDER_EVENT_OUTBOX 를 10분 간격으로 폴링하여 정산 정정 대기열에 적재한다" ("polls ORDER_EVENT_OUTBOX every ten minutes and loads the settlement-correction queue") → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- The relay reads at most 500 unpublished rows per run, ordered by `REG_DTM` → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- The relay, not the publisher, flips `PUBLISHED_YN` to `'Y'` → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- It parses the payload by string index arithmetic — `s.indexOf("\"sayuCd\":\"")` then a fixed two-character substring — rather than by JSON parsing → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- The relay only forwards events whose order appears in `SETTLEMENT_DTL`; everything else is marked published and dropped → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- A second index, `(EVENT_TYPE, PUBLISHED_YN)`, was added in May 2024 at the settlement team's request because "아웃박스 릴레이 느려짐" ("the outbox relay has slowed down") → src/main/resources/db/migration/V11__outbox_index.sql
- The service registry records the pairing and an unanswered question: `ORDER_EVENT_OUTBOX` has producer order-service and consumer settlement-batch, while the downstream `CANCEL_RECON_QUEUE` lists "consumer: TODO # 확인 필요" ("needs checking") → sources/context/registry/services.yaml

## Rationale

**What is recorded.** Two things, and both are outcomes rather than reasons:

- The need for a signal at all, which SF-2287 documents well: the manual correction process cannot start until the settlement team knows a cancellation happened.
- The choice of an in-transaction database write, which follows naturally from the two services already sharing a MySQL instance by the 2019 consolidation decision. An event row in the same database commits atomically with the cancellation, so an order can never be cancelled without its event, and no distributed transaction is needed.

**What is not recorded.** The choice of polling over a broker. This is the central gap in the decision and it is worth being precise about how thoroughly it is absent: SF-2287's thread contains no mention of Kafka, RabbitMQ, SQS or any transport; `V8`'s comment states "별도 브로커 없음" as a fact without a reason; `OrderEventPublisher`'s javadoc describes a broker that does not exist, suggesting the author's mental model and the shipped design differed; and no ADR, meeting note or design document in the reef's sources discusses it.

Plausible reasons exist — no broker was operated anywhere in the stack, the shared database made it free, the estimated volume was under ten events a month — but each is inference on my part, not evidence. The reef records the decision, not a justification for it.

## Consequences

### Positive

- **Atomicity without coordination.** The event row and the cancellation commit together in one local transaction. No two-phase commit, no dual-write inconsistency, no outbox-without-transaction anti-pattern.
- **Nothing new to operate.** No broker to provision, monitor or secure. The mechanism uses a table, an index and a Quartz job in a batch process that already ran nightly.
- **Replayable by construction.** `PUBLISHED_YN` makes the backlog queryable with a `SELECT`, and a mis-processed batch can be reset by an UPDATE. The 2022 spec even documented an admin replay endpoint for this, though it was never implemented.
- **Proportionate to the volume it was designed for.** At fewer than ten events a month, a ten-minute poll is ample.

### Negative

- **No delivery tracking on the publisher's side.** `OrderEventPublisher`'s javadoc states it outright: the subscriber's processing result is not tracked. order-service writes the row and learns nothing more — not whether it was read, not whether the correction happened, not whether it failed. [[PROC-ORDER-CANCEL]] has no compensating path because there is nothing to compensate against.
- **The producer cannot observe its own consumer.** `PUBLISHED_YN` is flipped by the other service, so the one health signal order-service could read is written by the party it would be monitoring.
- **Latency up to ten minutes, plus the batch window.** A cancellation at 01:55 may not be relayed before the 02:00 settlement run begins.
- **The coupling is a table, not a contract.** settlement-batch depends on the column names, the payload's exact byte layout (it substrings by index) and the table's presence. The V15/V16 rename incident shows this class of coupling has already broken production once for a different column. See [[SCH-ORDER]].
- **Silent filtering.** The relay drops any event whose order is not in `SETTLEMENT_DTL`, marking it published. Any future consumer wanting all cancellations would find the rows already consumed.
- **The compensation is human.** The whole chain terminates in a `PENDING` row in `CANCEL_RECON_QUEUE` and a person deducting an amount next month. The registry cannot say who consumes that queue.

### Neutral

- **The pattern is well known, its shape here unusual.** This is the transactional outbox pattern with the relay owned by the consumer rather than the producer, and no broker at the end of it. The registry acknowledges the arrangement as cohabitation rather than architecture: "아웃박스 릴레이 잡 동거" ("the outbox relay job cohabits").
- **It is still working, as far as anyone has written down.** No incident involving the outbox appears in the reef sources, and the only recorded adjustment is one index in 2024.
- **The decision remains unratified.** `TASK.md` lists "정산팀 컨슈머 확인 — 김도윤님 확인 후" ("confirm the settlement team's consumer — pending 김도윤") as an open item, three years after the consumer shipped. Whether anybody ever verified the other end is not recorded.

## Related

- [[SYS-ORDER]] — the publishing service
- [[PROC-ORDER-CANCEL]] — the flow that emits the event
- [[SCH-ORDER]] — the `ORDER_EVENT_OUTBOX` table
- [[CON-ORDER-SETTLEMENT]] — the boundary this decision spans
