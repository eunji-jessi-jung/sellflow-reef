---
id: "PROC-SETTLEMENT-ERROR-HANDLING"
type: "process"
title: "Settlement Batch Error Handling and Recovery"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Established 2026-09-19 by reading every Java file in settlement-batch and grepping the whole src tree for Transactional, catch, throw, faultTolerant, skip, retry, Listener and every mail/HTTP client name — the repository contains exactly one try/catch. Cross-checked against the 2025-07-12 postmortem and the S17 ticket export. Stale if a fault-tolerant step, a JobExecutionListener, a transaction annotation or an alerting dependency appears."
freshness_triggers:
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - "settlement-batch/src/main/resources/application.yml"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
  - "sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
known_unknowns:
  - "What the '배치 실행 이력 알림' added on 2025-07-18 and improved by SF-5099 (resolved 2026-09-02) actually is. No alerting code, dependency or configuration exists in settlement-batch, so the mechanism lives outside the repository and its coverage — failures only, or every run — could not be determined."
  - "Which Quartz misfire instruction applies. Neither trigger declares one, so Quartz's defaults govern, and the effective behaviour after a node outage was not verified against a deployed configuration."
  - "Whether any dailySettlementJob execution has actually failed in production. BATCH_JOB_EXECUTION is created by spring.batch.initialize-schema: always, but no export of it was available, so failure frequency is unmeasured."
  - "Whether a partially settled day has ever been re-run manually, and by what procedure. No runbook for re-running the batch exists in sources/context."
  - "How many cancellations have been dropped by the relay's ack-before-insert ordering. Detecting them requires comparing ORDER_EVENT_OUTBOX rows marked PUBLISHED_YN='Y' against CANCEL_RECON_QUEUE, and no such export exists."
  - "How many CANCEL_RECON_QUEUE rows carry the fallback SAYU_CD '00'. The monthly export aggregates by month and status, not by reason code."
tags:
  - settlement
  - operations
  - error-handling
  - spring-batch
  - reliability
aliases:
  - "settlement failure handling"
  - "정산 배치 장애 처리"
relates_to:
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-FLOW-CATALOG]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "The only recorded settlement incident; detection channel, timeline and five follow-up items"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-5099 batch-run-history alerting, Done 2026-09-02"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:infra/settlement/queues.md"
    notes: "Ack semantics and the fixed-offset payload parser"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "No fault tolerance, skip policy, retry policy or listener on either step"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java"
    notes: "The repository's only try/catch"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "Ack-before-success ordering and the substring payload parser"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "SETTLEMENT_DTL primary key (RUN_ID, ORD_NO) and the NOT NULL RUN_ID"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql"
    notes: "A run-status table with no writer"
notes: "The recurring shape here is that failure is survivable but invisible: nothing aborts loudly, nothing retries, and the only recorded detection of a settlement failure came from a partner."
---

# Settlement Batch Error Handling and Recovery

## Purpose

Describe what happens when settlement work fails: mid-run in the daily batch, mid-poll in the outbox relay, and in the parsing step that sits between them. The repository contains one `try`/`catch` in total, no skip policy, no retry policy, no step or job listener, no transaction annotation and no alerting dependency, so almost all of this artifact is a description of defaults and of consequences that nobody wrote code for. The most consequential finding is not a crash: it is the relay's ordering, which permanently drops a cancellation that arrives before that day's settlement row exists. That drop costs no money today — only because the cancel's own status write happens to remove the order from the settlement reader's filter — but it is a silent loss of the event either way.

## Key Facts

- The whole of settlement-batch contains exactly one `try`/`catch` and one `throw` — in `DailySettlementQuartzJob` — and no `@Transactional` anywhere (verified by grep over `src/` for `Transactional`, `catch`, `throw`) → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`
- That one handler catches `Exception` and rethrows it wrapped: `throw new IllegalStateException(DailySettlementJobConfig.JOB_NAME + " 실행 실패", e)` — "dailySettlementJob execution failed". It does not retry, does not record the failure anywhere and does not notify → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`
- Neither step declares fault tolerance. `settlementStep` is built as `.chunk(500).reader(...).processor(...).writer(...).build()` with no `.faultTolerant()`, no `.skipLimit`, no `.skip(...)`, no `.retry(...)` and no listener; `markSettledStep` is a bare `.tasklet(tasklet).build()` → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- So one bad row is a fatal row: any exception from the reader, processor or writer propagates, the chunk's transaction rolls back and the step and job end `FAILED` → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- **Restartability is configured neither on nor off, and is defeated in practice by the job parameters.** The job builder calls neither `.preventRestart()` nor `.incrementer(...)`; instead `DailySettlementQuartzJob` adds `.addLong("ts", System.currentTimeMillis())`, which makes every launch a new `JobInstance`. A failed instance is therefore never resumed — the next trigger starts a fresh one from item zero → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- The same `ts` parameter is what disabled Spring Batch's duplicate-instance protection on 2025-07-12, so the one mechanism that would have blocked the incident is the same one that would have enabled a restart → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`, `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- Committed chunks survive a later failure. The writer performs one `INSERT INTO SETTLEMENT_DTL` per item inside the chunk's transaction, so a failure at row 1,700 leaves the first three chunks — 1,500 settled orders — committed, and the job ends `FAILED` with the day half-settled → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- Nothing marks that partial state. `SETTLEMENT_RUN` has `SANGTAE`, `START_DTM`, `END_DTM` and `TOTAL_AMT`; `SETTLEMENT_RUN_LOG` (added in V2) has `STATUS`, `STARTED_AT`, `ENDED_AT` and `ROW_CNT`. No code in the repository writes to either table — grep for `INSERT INTO SETTLEMENT_RUN` and `SETTLEMENT_RUN_LOG` returns only the migrations themselves → `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`, `settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql`
- A failed `settlementStep` also means `markSettledStep` never runs, because the job is wired `.start(settlementStep).next(markSettledStep)` and the default transition requires `COMPLETED`. Orders settled in the committed chunks therefore keep `SANGTAE_CD='BAESONG_WANRYO'` while a `SETTLEMENT_DTL` row exists for them → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`
- Re-running the same `jungsanIlja` after such a failure has two possible outcomes, both bad, and which one occurs depends on a table nothing writes: if `MAX(RUN_ID) WHERE SANGTAE='RUNNING'` still resolves to the same run, the re-run hits the `PRIMARY KEY (RUN_ID, ORD_NO)` on the already-inserted rows and fails again; if it resolves to a new run, the already-settled orders are settled a second time under a new `RUN_ID` with no constraint to stop it → `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`
- There is a failure mode before the first row: `currentRunId()` runs `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'`, nothing in any repo inserts that row, and `SETTLEMENT_DTL.RUN_ID` is `BIGINT NOT NULL` — so with no `RUNNING` run the very first chunk fails on a null column → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- **The relay never fails a row twice, because it acks whether or not it acted.** `OrderEventRelayJob` sets `PUBLISHED_YN='Y'` for every event it looks at, outside any transaction covering the conditional insert, and only inserts into `CANCEL_RECON_QUEUE` when `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO=?` is greater than zero → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- That ordering permanently drops a real class of cancellation: an order cancelled *before* that night's settlement writes its `SETTLEMENT_DTL` row is relayed with `settled == 0`, so no queue row is created, and the event is marked published anyway. Nothing re-scans published events, so the case is gone. `settlementTargetReader` will not settle that same order hours later: it filters on `SANGTAE_CD='BAESONG_WANRYO' AND DATE(UPD_DTM)=?`, and the cancel overwrote both columns. The order is excluded — incidentally, not because any predicate asks about cancellation — so the dropped event has no payout behind it. What is lost is the record, not the money → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`
- The extracted infra notes reach the same conclusion from the other direction: "`PUBLISHED_YN='Y'` means 'the relay looked at it', not 'settlement acted on it'" → `sellflow-docs:infra/settlement/queues.md`
- **The payload parser fails silently.** `sayuCd(payload)` does `s.indexOf("\"sayuCd\":\"")` and, on a miss, returns the literal `"00"`; on a hit it takes `s.substring(i + 10, i + 12)` at a fixed offset with no JSON parser and no validation → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- `"00"` is not a defined reason code. The cancellation policy defines exactly four — 01 파트너 귀책, 02 시스템 오류, 03 고객 변심, 04 배송 실패 — and each carries a different 비용 부담 주체 (cost-bearing party) and 재고 복원 (stock restoration) rule, so a `00` row is a correction whose financial responsibility cannot be determined from the queue → `sellflow-docs:context/business-rules.md` (취소정책 sheet), `settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql`
- The same parser has a crash path as well as a silent path: if the marker is found within the last two characters of the payload, `substring(i + 10, i + 12)` throws `StringIndexOutOfBoundsException`, which — with no try/catch in the relay — aborts that run mid-batch → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- Because no transaction spans the relay loop, an abort mid-run leaves earlier rows acked and the failing row un-acked, so the next 10-minute poll retries from exactly that row. The relay's recovery behaviour is therefore accidental but sound, and it is the only retry that exists anywhere in the service → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`
- **There is no alerting code.** No mail, Slack, webhook or HTTP client dependency appears in `build.gradle`, and no such call appears in `src/` (verified by grep for `Mail`, `Slack`, `Webhook`, `RestTemplate`). Every failure path ends at a log line → `settlement-batch:build.gradle`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`
- Alerting exists somewhere outside the repository: the postmortem closes "배치 실행 이력 알림 추가 — 2025-07-18 완료" ("added batch run-history alerting, completed 2025-07-18"), and SF-5099 "정산 배치 실행 이력 알림 개선" ("improve batch run-history alerting") was resolved 2026-09-02. Neither is visible in code → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`, `sellflow-docs:context/sprints/tickets_2026-S17.csv`
- The one measured detection latency in this domain came from a partner, not a monitor: the 2025-07-12 duplicate run began at 02:04, duplicate payment requests appeared at 02:11, and the failure was recognised at 08:40 "파트너사 문의로 인지" ("noticed through a partner enquiry") — six and a half hours → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`
- The detector that would have caught it automatically, `DUP_SETTLE`, is documented in settlement-anomaly's README and implemented nowhere → `settlement-anomaly:README.md`, `settlement-anomaly:model/detector.py`

## Scope

**In scope:** the failure and recovery behaviour of `dailySettlementJob` (both steps), of `orderEventRelayJob` including its payload parsing, and of the operational detection and alerting around them.

**Out of scope, and covered elsewhere:** the normal-path mechanics of the batch ([[PROC-SETTLEMENT-DAILY-BATCH]]); the manual correction procedure that absorbs everything the batch gets wrong ([[PROC-SETTLEMENT-CORRECTION]]); the queue backlog those corrections have accumulated ([[RISK-SETTLEMENT-RECON-BACKLOG]]); and failure handling in settlement-anomaly, which is a single request-scoped function with its own artifact ([[RISK-SETTLEMENT-ANOMALY]]).

**Not applicable:** there is no dead-letter table, no error table, no quarantine queue and no compensation transaction anywhere in the settlement domain. The compensation mechanism is human and monthly — "정정이 필요한 경우 차월 정산에서 조정한다" ("if correction is needed, it is adjusted in the next month's settlement") → `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`

## Current State

### Failure inside the daily batch

A chunk-oriented step with no fault tolerance has exactly one behaviour: the first exception ends the job. What varies is where the damage lands.

| Where it fails | What is committed | What is not | Visible as |
|---|---|---|---|
| Reader (SQL error, connection loss) | nothing | everything | job `FAILED`, log line only |
| Processor (arithmetic, null `DANGA`) | all prior chunks | the current chunk onward | job `FAILED`, day half-settled |
| Writer (null `RUN_ID`, duplicate key) | all prior chunks | the current chunk onward | job `FAILED`, day half-settled |
| `markSettledStep` | all of `settlementStep` | the `ORDER_MST` update | job `FAILED`, orders settled but not marked |

→ `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`

The half-settled outcomes are the interesting ones, because they are indistinguishable from a successful thin day unless someone reads the logs. Partners in the committed chunks are paid; partners after the failure point are not, and the next day's run will not pick them up either, because its reader filters `DATE(m.UPD_DTM) = ?` on the new date. A missed day is therefore a silent underpayment that surfaces, if at all, as a partner enquiry — which the handover confirms is how correction work actually arrives: "실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음" ("in practice we have only checked and handled the cases where a partner enquired") → `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`, `sellflow-docs:context/handover/2025-03_정산팀_인수인계.md` (§2)

### Restart and re-run

Spring Batch's restart machinery is present — `spring.batch.initialize-schema: always` creates `BATCH_JOB_EXECUTION` and friends — and is never used, because every launch carries a unique `ts` parameter and therefore is a new `JobInstance` rather than a restart of the failed one. `spring.batch.job.enabled: false` further means nothing re-launches at process start; Quartz is the sole trigger, and its next fire is tomorrow at 02:00 → `settlement-batch:src/main/resources/application.yml`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`

There is no documented procedure for re-running a failed day. No runbook in `sources/context` covers it, and the recovery in the one recorded incident was financial (차월 상계, next-month offset), not technical → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`

### Failure inside the relay

The relay is more robust than the batch and entirely by accident. It has no try/catch, no transaction and no ack batching, so each `UPDATE ... PUBLISHED_YN='Y'` commits on its own. An exception anywhere stops the run with earlier rows durably acked and the failing row still `'N'`, and ten minutes later the poll resumes at that row. There is no poison-message protection, so a payload that reliably throws will block the 500-row window indefinitely — and there is nothing to notice that it has, because the run's only log line, `아웃박스 릴레이 처리 {}건`, is written after the loop and would never be reached → `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`

### The parse failure that is not a failure

```java
private String sayuCd(Object payload) {
    String s = String.valueOf(payload);
    int i = s.indexOf("\"sayuCd\":\"");
    return i < 0 ? "00" : s.substring(i + 10, i + 12);
}
```
→ `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`

`String.valueOf(null)` yields the four characters `"null"`, a payload with a space after the colon misses the marker, and a renamed field misses it too. All three produce `"00"` and an otherwise-normal queue row. Downstream, the reason code is the field that decides who bears the cost: 01 and 04 put it on the partner, 02 and 03 put it on 셀플로우 → `sellflow-docs:context/business-rules.md` (취소정책 sheet). A `00` row is a correction with no attributable party, inserted with the same confidence as a correct one, and `CancelReconciler` — the component that would consume it — copies `SAYU_CD` straight through into `SETTLEMENT_ADJUSTMENT` without validating it → `settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`

### The drop

This is the silent-loss defect, and it needs no exception to occur. Its financial consequence is smaller than it first appears — see step 5 — but the loss of the event itself is unconditional.

1. Customer cancels an order at 01:50. order-service writes `order.cancelled` to `ORDER_EVENT_OUTBOX`.
2. The relay's 02:00 poll reads it, runs `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO=?`, and gets 0 — the settlement batch has not written that order's detail row yet, or is writing it right now.
3. No `CANCEL_RECON_QUEUE` row is created. The relay nonetheless runs `UPDATE ORDER_EVENT_OUTBOX SET PUBLISHED_YN='Y'`.
4. At 02:00–02:0x the batch does *not* settle the order — but not because it was asked about cancellation. Its reader checks `SANGTAE_CD='BAESONG_WANRYO'` and `DATE(UPD_DTM)`, and the cancel at step 1 already overwrote both (`OrderMst.chwiso()` sets `SANGTAE_CD='CHWISO'` and `UPD_DTM=now` in one write), so the row fails both predicates. The reader's javadoc says the opposite — "주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다" — but that is a statement of intent, not a description of the SQL below it.
5. The event is therefore lost with no queue row, and no money is at stake in that particular loss: the order is cancelled, unsettled, and unpayable, because nothing ever sets `SANGTAE_CD` back to `BAESONG_WANRYO`. What is lost is the record — nothing re-reads published events, so there is no trace that a correction was ever considered for this order.

→ `settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java`

The window is the relay interval around the batch, but it is not only a race: any cancellation relayed at any time before that order's settlement row exists is dropped the same way. How many cancellations that is, is not measurable from any export available here. The reason the drop does not currently cost money is an accident of two unrelated writes lining up — the cancel happens to overwrite the two columns the settlement reader filters on — and it would stop holding the moment cancellation stopped setting `SANGTAE_CD`, or the reader keyed on a delivery-completion date instead of `UPD_DTM`. Note also that even a *correctly* queued row goes nowhere, because `CancelReconciler` has never been scheduled — that, and not this drop, is where [[RISK-SETTLEMENT-RECON-BACKLOG]]'s 4,127 rows sit.

### Detection and alerting

Inside the repository: nothing. No alerting dependency, no notification call, no metric, no health endpoint (settlement-batch has no web layer at all). Every error path terminates in SLF4J.

Outside it: two tracked improvements, both about run *history* rather than failure, and neither visible in code — the postmortem's completed "배치 실행 이력 알림 추가" (2025-07-18) and SF-5099 "정산 배치 실행 이력 알림 개선", Done 2026-09-02, 2 story points, reported and assigned to 박성민 of 주문팀 rather than to the owning 정산팀 → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`, `sellflow-docs:context/sprints/tickets_2026-S17.csv`

The 2025-07-12 timeline remains the honest measure of the domain's detection capability, and the incident it describes was not even a crash — every component behaved exactly as written, twice:

| Time | Event |
|---|---|
| 02:04 | duplicate scheduled run (two Quartz instances triggered simultaneously) |
| 02:11 | duplicate payment requests created for some partners |
| 08:40 | noticed through a partner enquiry |
| 09:30 | confirmed the duplicates cannot be cancelled; decided on next-month offset |

→ `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`

Three of that postmortem's five follow-ups are still open, and all three are error-handling items: an idempotency key before the payment request (unassigned), a check on whether cancelled orders are excluded from settlement (raised 2025-07-15, no discussion since), and a full audit of settlement batches including whether they are registered on a schedule — the audit that would have found `CancelReconciler` → `sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md`

## Related

- [[PROC-SETTLEMENT-DAILY-BATCH]] — the normal path whose failures are described here
- [[PROC-SETTLEMENT-FLOW-CATALOG]] — every flow in the domain and which of them run
- [[PROC-SETTLEMENT-CORRECTION]] — the manual, monthly compensation mechanism that substitutes for rollback
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what the queued corrections amount to when nothing drains them
- [[RISK-SETTLEMENT]] — the incident, the missing idempotency key and the untested deploy path
- [[SYS-SETTLEMENT]] — the service these jobs run inside
