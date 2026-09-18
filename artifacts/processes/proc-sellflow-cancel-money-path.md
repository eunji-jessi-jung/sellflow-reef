---
id: "PROC-SELLFLOW-CANCEL-MONEY-PATH"
type: "process"
title: "Cancellation Money Path End to End"
domain: "sellflow"
status: "active"
last_verified: 2026-09-19
freshness_note: "Deep pass on 2026-09-19: every file on the path read line by line and cited by line number — OrderController, OrderCancelService, OrderEventPublisher, OrderEventOutbox, OrderEventRelayJob, CancelReconciler, QuartzConfig, DailySettlementJobConfig, SettlementItemProcessor, SettlementItemWriter, MarkSettledTasklet, settlement-anomaly's main.py and detector.py, plus all six settlement migrations and V8/V11/V14 on the order side. The four break points are established by grep over all five repos, recorded verbatim below. Any trigger added to QuartzConfig, any CREATE TABLE for SETTLEMENT_ADJUSTMENT, or any change to the relay's settled-check invalidates this artifact."
freshness_triggers:
  - "order-service/src/main/java/kr/co/sellflow/order/controller/OrderController.java"
  - "order-service/src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
  - "order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "settlement-anomaly/app/main.py"
  - "settlement-anomaly/model/detector.py"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - "sources/raw/exports/cancel_recon_queue_monthly_20260901.csv"
known_unknowns:
  - "Whether any money has ever been clawed back by hand. The code path cannot have done it (four independent breaks, proven by grep), and the 2026-09-01 export queried STATUS='PENDING' only, so off-system corrections are invisible to both. Only a settlement analyst can answer it."
  - "What creates the SETTLEMENT_RUN row with SANGTAE='RUNNING' that SettlementItemWriter.currentRunId() depends on at line 43. No INSERT exists in any of the five repos, yet SETTLEMENT_DTL is demonstrably populated, so the creator is outside the repos."
  - "Whether SETTLEMENT_ADJUSTMENT exists as a physical table. No migration creates it; the 2025-03 handover says it cannot be queried. Until that is settled, break point 2 cannot be ranked as merely 'unscheduled'."
  - "Whether settlement-anomaly runs at all in production, and therefore whether break point 4 is live or moot. Its model pickle is absent from the repo and loaded at import scope, and no caller for POST /detect exists in any of the five repos."
  - "The real KRW value of the backlog. The export's amount column is count x 45,760 for all 41 months, so 188,851,520 KRW is a flat per-case estimate, not a sum of the JUNGSAN_AMT values actually paid. The true figure requires joining CANCEL_RECON_QUEUE to SETTLEMENT_DTL."
  - "Whether duplicate CANCEL_RECON_QUEUE rows exist. The relay's INSERT at lines 43-46 and its outbox ack at lines 50-52 are separate autocommitted statements with no unique constraint on ORD_NO, so a crash between them replays the insert."
tags:
  - "cancellation"
  - "settlement"
  - "money-path"
  - "deep-trace"
  - "broken-chain"
  - "sf-2287"
aliases:
  - "취소 정산 차감 경로"
  - "cancel to clawback"
  - "the money path"
relates_to:
  - type: "refines"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[DEC-ORDER-OUTBOX-RELAY]]"
  - type: "depends_on"
    target: "[[DEC-SETTLEMENT-CANCEL-CLAWBACK]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORPHANED-COMPONENTS]]"
  - type: "refines"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "depends_on"
    target: "[[PROC-SETTLEMENT-RUN-LIFECYCLE]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]]"
  - type: "integrates_with"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java"
    notes: "Lines 22-29: the only cancel entry point; lines 31-39: the two-field request body"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java"
    notes: "Lines 23-29: outbox insert inside the caller's transaction"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "Lines 36-38: the blocking set; lines 49-70: the whole transaction"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V14__add_settlement_ref.sql"
    notes: "JUNGSAN_RUN_ID declared a cache; SETTLEMENT_DTL is the record"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "Sheet 취소정책: all four reason codes carry 정산 차감 = O. Sheet 수수료: 12.0 / 9.5 / 6.0 percent tiers"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "CANCEL_RECON_QUEUE consumer TODO; settlement-anomaly 조치 주체 undefined"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "Payout irreversibility and the unowned action item about cancelled orders in settlement"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The 2023-04 bargain that created the path"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "4,127 PENDING rows over 41 months; every amount is count x 45,760"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
    notes: "2026-08-24 finance query: the balance contradicts the stated process"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
    notes: "Line 19: pickle loaded at import; lines 29-47: the detect query and insert"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "Lines 11, 26-36: the CANCELLED_SETTLED rule and its lowercase row keys"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
    notes: "Lines 63-80: the reader that decides who gets paid"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Lines 18-54: two jobs registered, neither of them the reconciler"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
    notes: "Line 19: DEFAULT_FEE_RATE 0.12; lines 22-30: HALF_UP"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
    notes: "Lines 26-39: the irreversible write; lines 41-45: run id resolution per chunk"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
    notes: "Lines 16-17: the TODO; lines 32-52: the deduction that never runs"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "Lines 29-53: poll, settled-check, conditional insert, unconditional ack"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
    notes: "Lines 28-31: cross-team write, unfiltered MAX(RUN_ID)"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "Lines 7-39: SETTLEMENT_RUN, SETTLEMENT_DTL, CANCEL_RECON_QUEUE"
notes: "Archetype: cross-system flow. The component-level artifacts already cover each hop; this one exists to hold the single chain in one place, with the branch inventory, the transaction map, and the four break points ranked in the order a reader hits them. Where a component artifact and this one disagree, the line number wins."
---

# Cancellation Money Path End to End

## Purpose

One customer cancels one order that has already been paid out to a partner. This artifact follows the money from the HTTP request to the last component that could plausibly do something about it, reading every branch on the way, and states at each hop what is written, in what transaction, and what is not.

The finding is not that the chain is slow. It is that the chain has four independent breaks, each sufficient on its own, and that the first three of them have been in place since the path was built in April 2023. The path is nine hops long; money stops moving at hop six and nothing downstream recovers it.

## Key Facts

- The entry point is `POST /orders/{ordNo}/cancel`, which takes a body of exactly two fields — `sayuCd` and `bigo` — and passes them straight through: `orderCancelService.cancel(ordNo, req.getSayuCd(), req.getBigo())` → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java (lines 22-29, 31-39)
- Nothing on the request identifies the caller or the order's owner. The controller has no security annotation, and the service's only CORS rule maps `/api/**` while this controller is mapped `/orders` — so the one origin restriction in the repository does not cover the cancel endpoint at all → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java (line 10), order-service:src/main/java/kr/co/sellflow/order/config/WebConfig.java (lines 11-14)
- The service checks exactly two things: that the order exists (line 52-53) and that its status is not already `CHWISO` or `BANPUM` (lines 55-58, against the set built at lines 36-38). Settlement state is not among them → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java (lines 36-38, 52-58)
- Three writes commit atomically because `cancel` is annotated `@Transactional` at line 49 and `OrderEventPublisher.publishOrderCancelled` (line 23) joins that transaction rather than starting its own: the `ORDER_CANCEL` insert (line 62), the `ORDER_MST` update (lines 63-64) and the outbox insert (line 66, executing at OrderEventPublisher lines 27-28) → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java (lines 49-66), order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java (lines 23-29)
- The event carries no money and no partner. The payload is `String.format("{\"ordNo\":\"%s\",\"sayuCd\":\"%s\"}", ordNo, sayuCd)` — two fields, no amount, no timestamp, no version → order-service:src/main/java/kr/co/sellflow/order/event/OrderEventPublisher.java (lines 25-26)
- The relay decides whether a cancellation is a money event by a single count: `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` at lines 38-40, and only a result greater than zero reaches the insert at lines 43-46. "Already paid" is defined nowhere else on this path → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java (lines 38-48)
- The relay's two statements per event are not in one transaction. The `CANCEL_RECON_QUEUE` insert (lines 43-46) commits before the outbox ack (lines 50-52), on a plain `JdbcTemplate` with no `@Transactional` on the class or on `executeInternal` (lines 27-28) — so a crash between them replays the event and inserts the queue row twice, and `CANCEL_RECON_QUEUE` has no unique constraint on `ORD_NO` to stop it → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java (lines 27-53), settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql (lines 30-39)
- **Break 1 — no trigger.** `QuartzConfig` declares four beans covering two jobs, `dailySettlementQuartzJob` (lines 18-35) and `orderEventRelayJob` (lines 37-54). `CancelReconciler` is a bare `@Component` (line 19-20) with no `JobDetail`, no `Trigger`, and no `@Scheduled` → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java (lines 18-54), settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java (lines 19-20)
- **Break 1, proven by grep.** `grep -rn "CancelReconciler\|reconcileCancellations\|loadPending" ../sellflow/repos` returns five lines, all inside `CancelReconciler.java` itself: the class declaration (20), its logger (22), `loadPending` (26), `reconcileCancellations` (32) and the self-call at 33. No caller, no test, no configuration reference in any of the five repos → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- **Break 2 — no target table.** Even with a trigger, the first statement of the loop would fail: `INSERT INTO SETTLEMENT_ADJUSTMENT (ORD_NO, SAYU_CD, ADJ_TYPE) VALUES (?, ?, 'CANCEL_CLAWBACK')` at lines 40-43 names a table that `grep -rn "SETTLEMENT_ADJUSTMENT"` finds exactly once across all five repos — at that INSERT. No `CREATE TABLE`, in any of the six settlement migrations or anywhere else → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java (lines 40-43)
- The class documents its own break, dated the day the relay shipped: "TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤" ("TODO: Quartz schedule registration needed; to be added to QuartzConfig after checking with 박성민. 2023-04-24, 김도윤") → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java (lines 16-17)
- The payout side never looks at cancellation. The reader's SQL filters `WHERE m.SANGTAE_CD = 'BAESONG_WANRYO' AND DATE(m.UPD_DTM) = ?` (lines 75-76) and joins only `ORDER_MST` to `ORDER_DTL`; `ORDER_CANCEL`, `CANCEL_RECON_QUEUE` and `SETTLEMENT_ADJUSTMENT` appear in no query in the batch. Its javadoc states the intent: "주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다" ("the order's current status or whether it was cancelled is not part of the condition") → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java (lines 56-80)
- **Break 3 — no deduction input.** Because of that, a `CANCEL_RECON_QUEUE` row has no effect on any later run: the next month's settlement reads the same two tables as the last one and cannot see the queue. "차월 차감" ("deduction from the following month") exists only in `CancelReconciler` and in documents → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java (lines 71-77), settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java (lines 40-43)
- The fee is a constant, not a contract: `private static final BigDecimal DEFAULT_FEE_RATE = new BigDecimal("0.12")` at line 19, applied to every row of every partner at lines 24-25 with `RoundingMode.HALF_UP`. `PartnerContractRepository.find` exists and compiles and is called by nothing — `grep -rn "PartnerContractRepository"` returns only its own file → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java (lines 19-29), settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java
- The write is one-way by design, and the payout it represents is not in this repository at all: `SettlementItemWriter.write` does a plain per-row `INSERT INTO SETTLEMENT_DTL` (lines 31-37) under a javadoc that says "지급 요청이 전송되면 되돌릴 수 없다. 은행 이체는 익영업일에 실행된다" ("once the payment request is sent it cannot be reversed; the bank transfer executes on the next business day") — there is no code here that sends that request → settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java (lines 13-39)
- The 2025-07-12 postmortem confirms irreversibility in production rather than in a comment: duplicate payouts to 17 partners, about 42,000,000 KRW, were resolved by "전량 차월 상계 처리" ("offset in full against the following month"), with the cause recorded as "`SettlementItemWriter` 는 지급 요청 전송 후 되돌릴 수 없음. 취소 경로 없음" ("SettlementItemWriter cannot be reversed after the payment request is sent; there is no cancellation path") → sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md
- The same postmortem left the question this artifact answers open and unowned: "취소 건이 정산 대상에서 제외되는지 점검" ("check whether cancelled orders are excluded from settlement"), raised 2025-07-15, unchecked, with the note that it was split into a separate ticket whose number was never confirmed → sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md
- **Break 4 — the detector that would notice cannot report to anyone.** `detector.detect` flags exactly the population this path leaks: `if r["sangtae_cd"] in CANCELLED_STATES` (line 30, against `{"CHWISO", "BANPUM"}` at line 11) emits `CANCELLED_SETTLED` with a hardcoded score of 1.0. Its output lands in `SETTLEMENT_ANOMALY` with `STATUS='DETECTED'` (main.py lines 42-47) and stops there: `grep -rn "SETTLEMENT_ANOMALY"` across the five repos returns only that INSERT, the README and the DDL — no reader, no screen, no job → settlement-anomaly:model/detector.py (lines 11, 26-36), settlement-anomaly:app/main.py (lines 41-47)
- The registry records break 4 as an open question rather than a defect: `settlement-anomaly` carries "notes: 이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" ("performs detection only; the party responsible for acting is not defined"), and `CANCEL_RECON_QUEUE` carries "consumer: TODO   # 확인 필요" ("consumer: TODO, needs checking") → sellflow-docs:context/registry/services.yaml
- What the data proves, as against what the code proves: the export of 2026-09-01 shows 4,127 rows still `PENDING` across 41 consecutive months beginning 2023-04 — the month the relay shipped — with no month at zero. The code proves no row can leave `PENDING`; the data proves none has been observed to → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv
- The money figure attached to that backlog is an estimate with a uniform unit price, not a ledger: all 41 monthly amounts divide by their counts to exactly 45,760 KRW, and 4,127 x 45,760 = 188,851,520. The header calls the column 추정 미정정 금액 ("estimated uncorrected amount") → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv

## Flow

Nine hops. Trigger: a customer or CS agent calls the cancel endpoint. Frequency: on demand; the relay every 10 minutes; the batch daily at 02:00 KST.

| # | Where | What executes | What it writes | Transaction |
|---|---|---|---|---|
| 1 | order-service `OrderController.cancel` | lines 22-29 | nothing | none |
| 2 | order-service `OrderCancelService.cancel` | lines 49-70 | `ORDER_CANCEL` insert (62), `ORDER_MST.SANGTAE_CD='CHWISO'` (63-64) | one `@Transactional` (49) |
| 3 | order-service `OrderEventPublisher` | lines 23-29 | `ORDER_EVENT_OUTBOX` row, `PUBLISHED_YN='N'` | joins hop 2's transaction |
| 4 | settlement-batch `OrderEventRelayJob` poll | lines 29-33 | nothing | none |
| 5 | settlement-batch `OrderEventRelayJob` gate | lines 38-40 | nothing — `COUNT(1)` on `SETTLEMENT_DTL` | none |
| 6 | settlement-batch `OrderEventRelayJob` insert + ack | lines 43-46, 50-52 | `CANCEL_RECON_QUEUE` row `PENDING`; `PUBLISHED_YN='Y'` | two separate autocommits |
| 7 | settlement-batch `CancelReconciler` | lines 32-52 | would write `SETTLEMENT_ADJUSTMENT` + `PROCESSED` | **never executes** |
| 8 | settlement-batch next `dailySettlementJob` | DailySettlementJobConfig lines 63-80 | `SETTLEMENT_DTL` for the new day | **does not read hops 6-7** |
| 9 | settlement-anomaly `POST /detect` | main.py lines 26-50 | `SETTLEMENT_ANOMALY` `DETECTED` | one commit per row (db.py 22-26) |

```
customer / CS
   │  POST /orders/{ordNo}/cancel   {sayuCd, bigo}
   ▼
OrderController.cancel ─────────────────────────────── L22-29
   ▼
OrderCancelService.cancel  @Transactional ──────────── L49-70
   ├─ findById or 404 .................................. L52-53
   ├─ if status in {CHWISO, BANPUM} → 409 .............. L55-58
   ├─ CancelReason.of(sayuCd) or IllegalArgument → 500 . L60
   ├─ INSERT ORDER_CANCEL (CHORI_SANGTAE='COMPLETED') .. L62
   ├─ UPDATE ORDER_MST SANGTAE_CD='CHWISO' ............. L63-64
   └─ OrderEventPublisher.publishOrderCancelled ........ L66 → outbox L27-28
                                                   [ COMMIT — one transaction ]
   ▼   (≤10 min)
OrderEventRelayJob.executeInternal ─────────────────── L28-58
   ├─ SELECT … PUBLISHED_YN='N' … LIMIT 500 ............ L29-33
   ├─ SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO=? L38-40
   ├─ if >0 → INSERT CANCEL_RECON_QUEUE 'PENDING' ...... L43-46   [ autocommit ]
   └─ UPDATE ORDER_EVENT_OUTBOX PUBLISHED_YN='Y' ....... L50-52   [ autocommit, unconditional ]
   ▼
CANCEL_RECON_QUEUE (PENDING)          ◄── 4,127 rows, 2023-04 → 2026-08
   ║
   ║   BREAK 1: no trigger registered   (QuartzConfig L18-54)
   ║   BREAK 2: no SETTLEMENT_ADJUSTMENT table (CancelReconciler L40-43)
   ╳
CancelReconciler.reconcileCancellations ───────────── L32-52   NEVER RUNS
   ╳
   ║   BREAK 3: next month's batch never reads the queue
   ║            (DailySettlementJobConfig L71-77 joins ORDER_MST + ORDER_DTL only)
   ▼
dailySettlementJob 02:00 KST ──────────────────────── reader L63-80
   ├─ processor: fee = gross × 0.12 HALF_UP ........... L19, L24-25
   ├─ writer:    INSERT SETTLEMENT_DTL (irreversible) . L31-37
   └─ tasklet:   UPDATE ORDER_MST → 'JUNGSAN_WANRYO' .. L28-31
   ▼
settlement-anomaly POST /detect ───────────────────── main.py L26-50
   └─ CANCELLED_SETTLED, score 1.0 ................... detector.py L30-35
   ▼
SETTLEMENT_ANOMALY (STATUS='DETECTED')
   ║   BREAK 4: no reader, no owner, no next state
   ╳
```

## Branch Inventory

Every condition on the path, and — the point of the exercise — the conditions that are absent.

**Checked, hop 2 (`OrderCancelService`, lines 52-60).**

| Condition | Line | Outcome |
|---|---|---|
| Order exists | 52-53 | `OrderNotFoundException` → 404 via `GlobalExceptionHandler` lines 14-18 |
| Status not in `{CHWISO, BANPUM}` | 55-58 | `OrderCancelNotAllowedException` → 409 via lines 20-25 |
| Reason code is one of 01-04 | 60 | `IllegalArgumentException` — unhandled, escapes as 500 |

**Not checked, hop 2.** Each of these is an absence with a consequence, not a stylistic gap:

| Not checked | Evidence of absence | Consequence |
|---|---|---|
| Whether the order was settled | `CHWISO_BULGA` holds two values (lines 36-38); no `SETTLEMENT_DTL` query in the class | The entire backlog. This check existed before SF-2287 — the legacy `OrderCancelServiceV1` still performs it at lines 34-42 with `SELECT COUNT(1) FROM SETTLEMENT_DTL` and throws "정산 완료된 주문은 취소할 수 없습니다" |
| Whether the caller owns the order | no parameter for it (controller lines 23-25); no security annotation anywhere in the class | Any caller who reaches the endpoint can cancel any order number |
| Delivery state | `BAESONG_JUNG` and `BAESONG_WANRYO` are absent from the blocking set | An in-flight or delivered order cancels outright |
| Elapsed time since order | no use of `getJumunIlsi()` in the service | A 2023 order is as cancellable as today's |
| Amount | `getChongGeumaek()` is never called on this path | No value-based routing or approval is possible |
| Reason-specific handling | `sayu` is used only at lines 62 and 66 | Codes 01 and 03 behave identically although the business rules assign them different cost bearers |
| Idempotency of the event | no check before `publishOrderCancelled` | Not reachable in practice: a second cancel is blocked at 55-58 |

**Checked, hop 5-6 (`OrderEventRelayJob`).** One condition only: `settled != null && settled > 0` (line 42). Not checked: whether a queue row for this `ORD_NO` already exists; whether `SAYU_CD` parsed to a real code (the fallback at line 63 returns the literal `"00"`, which is not one of 01-04); whether the insert succeeded before the ack at lines 50-52.

**Checked, hop 8 (`DailySettlementJobConfig` reader, lines 71-77).** Two conditions: `SANGTAE_CD = 'BAESONG_WANRYO'` and `DATE(m.UPD_DTM) = :jungsanIlja`. Not checked: cancellation, in any form. The exclusion of a same-day cancelled order is incidental — `OrderMst.chwiso()` (lines 40-43) overwrites `SANGTAE_CD` and `UPD_DTM` together, so the row drops out of the filter because its status changed, not because anybody asked about cancellation.

## Transaction Map

| Boundary | Statements inside | Annotation | What a failure loses |
|---|---|---|---|
| Hop 2-3 | `ORDER_CANCEL` insert, `ORDER_MST` update, outbox insert | `@Transactional` at OrderCancelService line 49; publisher's own `@Transactional` (line 23) joins it | All three, together. The customer sees the failure |
| Hop 6, insert | one `CANCEL_RECON_QUEUE` insert | none | The claim on the money |
| Hop 6, ack | one outbox update | none | Replay: the insert repeats, creating a duplicate row |
| Hop 7 | adjustment insert + queue update | none declared | Not applicable — never runs |
| Hop 8 | per-chunk of 500, Spring Batch | `settlementStep` chunk size 500 (DailySettlementJobConfig lines 31, 49) | The chunk. `RUN_ID` is re-resolved per chunk at SettlementItemWriter lines 41-45, so a run-state change mid-job splits output across run ids |
| Hop 9 | one `INSERT` per anomaly | `app/db.py` opens a connection and commits per `execute` (lines 22-26) | Partial detection sets; `/detect` has no uniqueness guard, so a re-run doubles rows |

The asymmetry is the finding: the order side is atomic and the settlement side is not. The one transaction on this path protects the record that costs nothing to rebuild (the cancellation), while the statements that carry the financial claim run bare.

## Money Consequence per Hop

Separating what the sources prove from what follows by inference. The distinction matters because the finance team is being asked to book a number.

| Hop | Money consequence | Provable from | Status |
|---|---|---|---|
| 2 | Customer's order becomes cancellable regardless of payout state | OrderCancelService lines 36-38, 55-58; SF-2287 change log | **Proven — code** |
| 3-6 | A claim is recorded that partner money should come back | OrderEventRelayJob lines 38-48; V1 lines 30-39 | **Proven — code** |
| 6 | 4,127 such claims accumulated over 41 months, none observed to leave `PENDING` | cancel_recon_queue_monthly_20260901.csv | **Proven — data**, with the export's own caveat that it queried `PENDING` only |
| 6 | 188,851,520 KRW of exposure | same file | **Inferred.** Every month's amount is count x 45,760 exactly; this is a flat per-case estimate, not a sum of paid amounts. The real figure needs `JOIN SETTLEMENT_DTL` |
| 7 | No clawback is written, ever | grep: `CancelReconciler` has no caller; `SETTLEMENT_ADJUSTMENT` has no DDL | **Proven — absence** |
| 8 | No deduction reaches the next payout | DailySettlementJobConfig lines 71-77 — the queue is in no query | **Proven — absence** |
| 8 | Partners on 9.5% or 6.0% contract rates are charged 12.0% | SettlementItemProcessor line 19; PartnerContractRepository unused; business-rules 수수료 sheet | **Proven — code**; the count of affected partners is unknown, since `PARTNER_CONTRACT` contents are not in any source here |
| 9 | The one component that would notice writes a row nobody reads | detector.py lines 30-35; grep on `SETTLEMENT_ANOMALY`; services.yaml `조치 주체는 정의되어 있지 않음` | **Proven — absence** |
| — | Whether humans recovered any of it off-system | nothing in the five repos; the export explicitly cannot see it | **Unknown** — see `known_unknowns` |

The 2026-08-24 finance mail is the independent check on all of this. 문지영 (재무기획팀) states the contradiction without having read the code: "차감이 정상적으로 이루어지고 있다면 대기 잔액이 이 규모로 누적될 수 없습니다" ("if the deduction were happening properly, a waiting balance could not accumulate to this size"), notes that 2023-04 rows are still present, and asks two questions — how many cases have actually been deducted, and whether the deducting party is a batch or a human. Both are answered here: none by code, and no batch.

## Worked Examples

### 1. A single order, followed from tap to dead end

`ORD20260610000042`, delivered 2026-06-10, partner `P-0147`, one line item at 40,000 KRW x 1.

- **02:00 on 2026-06-11.** The reader picks the order up: its status is `BAESONG_WANRYO` and `DATE(UPD_DTM)` is 2026-06-10. Processor: `gross = 40,000`, `fee = 40,000 x 0.12 = 4,800` (HALF_UP, scale 0), `jungsanAmt = 35,200`. Writer inserts one `SETTLEMENT_DTL` row under `MAX(RUN_ID) WHERE SANGTAE='RUNNING'`. Tasklet flips `ORDER_MST.SANGTAE_CD` to `JUNGSAN_WANRYO`. The bank transfer follows on the next business day; nothing in these repos sends it.
- **11:20 on 2026-06-11.** The customer calls CS. `POST /orders/ORD20260610000042/cancel` with `sayuCd=03`. The guard at lines 55-58 passes, because `JUNGSAN_WANRYO` is not `CHWISO` or `BANPUM`. `ORDER_CANCEL` gets a row with `CHORI_SANGTAE='COMPLETED'` — a status asserting the cancellation is fully processed, written before any downstream work is attempted. `ORDER_MST.SANGTAE_CD` becomes `CHWISO`, erasing the visible trace of the settlement; only `SETTLEMENT_DTL` and the `JUNGSAN_RUN_ID` cache column added by V14 still record it.
- **by 11:30.** The relay polls, counts one `SETTLEMENT_DTL` row, inserts `CANCEL_RECON_QUEUE (ORD_NO='ORD20260610000042', SAYU_CD='03', STATUS='PENDING')`, and acks the outbox row.
- **Then nothing.** Hop 7 has no trigger and no target table. Hop 8 on 2026-06-12 reads `ORDER_MST` and `ORDER_DTL` for orders delivered on 06-11; this order is not in that set and the queue is not in that query. 35,200 KRW stays with `P-0147`; the customer's refund, if any, is a PG operation that `PaymentClient` would perform and that nothing on this path calls.
- **03:00 on 2026-06-12, if settlement-anomaly runs.** `/detect` for `jungsan_ilja = 2026-06-11` joins `SETTLEMENT_DTL` to `ORDER_MST`, sees `SANGTAE_CD = 'CHWISO'`, and would write `CANCELLED_SETTLED` with score 1.0 into `SETTLEMENT_ANOMALY`. That row has `REVIEWED_BY` and `REVIEWED_DTM` columns that no code in any repo writes.

The order is then, simultaneously: cancelled (order-service), settled (`SETTLEMENT_DTL`), pending correction (`CANCEL_RECON_QUEUE`) and flagged anomalous (`SETTLEMENT_ANOMALY`) — four tables in one MySQL instance, four different answers, no reconciliation between them.

### 2. The queries that settle the open questions

Run against `sellflow_order`; all four tables live in the same instance.

```sql
-- 1. Has any row ever left PENDING? The 2026-09-01 export could not answer this:
--    its WHERE clause was STATUS='PENDING'.
SELECT STATUS, COUNT(*) AS rows_, MIN(RECV_DTM), MAX(PROCESSED_DTM)
  FROM CANCEL_RECON_QUEUE
 GROUP BY STATUS;

-- 2. The real exposure, instead of count x 45,760.
SELECT COUNT(DISTINCT q.ORD_NO)      AS orders_,
       COUNT(*)                      AS queue_rows,      -- higher ⇒ duplicates exist
       SUM(d.JUNGSAN_AMT)            AS paid_out_krw
  FROM CANCEL_RECON_QUEUE q
  JOIN SETTLEMENT_DTL d ON d.ORD_NO = q.ORD_NO
 WHERE q.STATUS = 'PENDING';

-- 3. Cancelled orders that were settled but never even reached the queue —
--    the relay only inserts when SETTLEMENT_DTL already existed at relay time.
SELECT COUNT(*)
  FROM ORDER_MST m
  JOIN SETTLEMENT_DTL d ON d.ORD_NO = m.ORD_NO
  LEFT JOIN CANCEL_RECON_QUEUE q ON q.ORD_NO = m.ORD_NO
 WHERE m.SANGTAE_CD IN ('CHWISO','BANPUM')
   AND q.SEQ IS NULL;

-- 4. Fee overcharge from the hardcoded 0.12.
SELECT c.PARTNER_ID, c.FEE_RATE, COUNT(*) AS settled_rows,
       SUM(d.SUSURYO) - SUM(ROUND((d.JUNGSAN_AMT + d.SUSURYO) * c.FEE_RATE, 0)) AS overcharged_krw
  FROM SETTLEMENT_DTL d
  JOIN PARTNER_CONTRACT c ON c.PARTNER_ID = d.PARTNER_ID
 WHERE c.FEE_RATE <> 0.12
 GROUP BY c.PARTNER_ID, c.FEE_RATE;
```

Query 3 is the one no document anticipates. The relay's gate is a point-in-time `COUNT(1)`; a cancellation that arrives in the ten-minute window before the nightly batch writes the detail row passes the gate as "not settled", is acked, and leaves no queue row — while the batch, filtering on status rather than on cancellation, may still pick the order up if it was already `BAESONG_WANRYO` with yesterday's `UPD_DTM`. That population is invisible to the queue, to the backlog figure and to the finance mail.

### 3. What fixing break 1 alone would do

A reasonable first response to this artifact is "register the trigger". Traced line by line, that produces:

1. `loadPending()` (lines 26-30) returns all 4,127 rows in one `List<Map<String,Object>>` — no pagination, no limit.
2. The loop's first statement (lines 40-43) inserts into `SETTLEMENT_ADJUSTMENT`, which no migration creates. On MySQL this raises `ER_NO_SUCH_TABLE` on row one.
3. Nothing catches it. `reconcileCancellations` declares no try/catch and the method is not `@Transactional`, so the job fails on the first row, forever, every interval — and, were the table to exist, a mid-run failure would leave the processed prefix marked `PROCESSED` with no rollback.
4. Even on success, no money moves: the adjustment row is written into a table that hop 8 does not read (break 3). The deduction would still be a human reading `SETTLEMENT_ADJUSTMENT` and adjusting a payout by hand.

Breaks 1, 2 and 3 are therefore ordered, not alternative. The trigger is the cheapest and the least useful of the three to fix.

## Related

- [[PROC-ORDER-CANCEL]] — hops 1-3 in full, including the side effects order-service does not perform
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — hop 6 at the level of a single row and its columns
- [[PROC-SETTLEMENT-DAILY-BATCH]] — hop 8 as a job, step by step
- [[PROC-SETTLEMENT-RUN-LIFECYCLE]] — the `SETTLEMENT_RUN` row hops 8 depends on and no repo creates
- [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] — column-level derivation of the amounts computed at hop 8
- [[CON-ORDER-SETTLEMENT]] — the negotiated agreement this path was built to implement
- [[DEC-ORDER-OUTBOX-RELAY]] — why the transport is a table and not a broker
- [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] — the next-month deduction as a decision, and what it assumed
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the exposure these hops accumulate
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — the wider pattern breaks 1 and 4 belong to
- [[SYS-SETTLEMENT-ANOMALY]] — hop 9 and the review workflow that was designed but never staffed
