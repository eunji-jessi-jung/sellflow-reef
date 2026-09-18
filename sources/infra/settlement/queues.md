# Queues & scheduled work — Settlement

> Derived from Quartz configuration, job classes and SQL in `settlement-batch`, cross-checked
> against `sources/context/registry/services.yaml`. Tier 4 (code reading).

## There is no message broker

Both "queues" in this system are database tables polled by Quartz jobs. `order-service`'s
`V8__add_cancel_event_outbox.sql`: `정산 배치의 릴레이 잡이 주기적으로 폴링한다. (별도 브로커 없음)`.
`settlement-batch/README.md`: `별도 메시지 브로커 없이 ORDER_EVENT_OUTBOX 를 폴링하여 정산 후 취소 건을 CANCEL_RECON_QUEUE 에 적재한다. (SF-2287)`
No Kafka, RabbitMQ, SQS or Celery dependency appears in any of the five repos.

## ORDER_EVENT_OUTBOX

| | |
|---|---|
| Producer | order-service `OrderEventPublisher.publishOrderCancelled` (event type `order.cancelled`, the only type written) |
| Consumer | settlement-batch `OrderEventRelayJob` |
| Transport | polling, `SELECT ... WHERE PUBLISHED_YN='N' AND EVENT_TYPE=? ORDER BY REG_DTM LIMIT 500` |
| Interval | every 10 minutes (`SimpleScheduleBuilder.withIntervalInMinutes(10).repeatForever()`) |
| Ack | `UPDATE ORDER_EVENT_OUTBOX SET PUBLISHED_YN='Y'` after processing each row |
| Payload | `{"ordNo":"...","sayuCd":".."}`, built with `String.format`, parsed back out by `substring(i+10, i+12)` — not a JSON parser |

Matches the registry entry (`producer: order-service`, `consumer: settlement-batch (OrderEventRelayJob)`).

**Observations.** The 500-row cap with a 10-minute interval gives a ceiling of ~3,000 cancel
events/hour. The relay marks a row published even when the insert branch is skipped, so
`PUBLISHED_YN='Y'` means "the relay looked at it", not "settlement acted on it". The payload
parser assumes a fixed 2-character `sayuCd` at a fixed offset and silently yields `"00"` on any
shape change.

## CANCEL_RECON_QUEUE

| | |
|---|---|
| Producer | settlement-batch `OrderEventRelayJob` — inserts only when `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO=?` is > 0, i.e. only for orders already settled |
| Consumer | **undefined** |
| Intended consumer | `CancelReconciler.reconcileCancellations()` — drains PENDING rows into `SETTLEMENT_ADJUSTMENT` with `ADJ_TYPE='CANCEL_CLAWBACK'` |
| Status | `CancelReconciler` is a `@Component` that **nothing calls**. `QuartzConfig` registers only `dailySettlementJobDetail` and `orderEventRelayJobDetail`. Its own TODO says the trigger was never added: `TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤` |
| Designed handling | manual. V1: `정산팀이 주기적으로 확인하여 차월 정산에서 차감한다.` |

The registry records the same gap: `consumer: TODO   # 확인 필요`.

## Quartz schedules (settlement-batch)

| Trigger | Job | Schedule |
|---|---|---|
| `dailySettlementTrigger` | `dailySettlementQuartzJob` → Spring Batch `dailySettlementJob` | cron `0 0 2 * * ?`, TimeZone `Asia/Seoul` |
| `orderEventRelayTrigger` | `orderEventRelayJob` | simple, every 10 minutes, forever |

Quartz runs in clustered JDBC mode (`spring.quartz.job-store-type: jdbc`,
`org.quartz.jobStore.isClustered: true`), so the scheduler state lives in the same MySQL instance.
`spring.batch.job.enabled: false` prevents Spring Batch from auto-running the job at startup —
Quartz is the sole trigger.

## Unscheduled / externally triggered

- `settlement-anomaly` `POST /detect` — its README claims a daily 03:00 trigger after the batch,
  but no scheduler exists in that repo and no caller was found in any of the five. The trigger is
  external and unidentified.
- `SettlementReportWriter.write(runId)` — a `@Component` called by no step. The partner portal is
  said to read its output file, but the method only logs.
