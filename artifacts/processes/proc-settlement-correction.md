---
id: "PROC-SETTLEMENT-CORRECTION"
type: "process"
title: "Settlement Correction Workflow"
domain: "settlement"
status: "draft"
last_verified: 2026-09-18
freshness_note: "Traced 2026-09-18 across the issued procedure v1.1, the code in settlement-batch, and eight company documents spanning 2023-04 to 2026-09. The central finding — that no scheduled component drains CANCEL_RECON_QUEUE — was verified by grep across all five repos on that date and will stop being true the moment SF-4512 is done."
freshness_triggers:
  - "src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "src/main/resources/db/migration/V1__settlement_schema.sql"
known_unknowns:
  - "How many corrections have actually been performed manually. The handover says only partner-reported cases are handled, and no record of them exists in the system — the team manages them in individual spreadsheets."
  - "Whether any deduction ever reaches a partner's next-month payout. No code applies a CANCEL_RECON_QUEUE row or a SETTLEMENT_ADJUSTMENT row to a settlement run."
  - "Whether SETTLEMENT_ADJUSTMENT exists as a table at all. No migration creates it and the handover reports it cannot be queried."
  - "Whether the ST-F-001 and ST-F-002 forms mandated by procedure v1.1 §6 have ever been filled in. The handover states no official record format exists in practice."
  - "Whether the monthly review required by v1.1 §4 step 2 has ever taken place. The handover says the full queue is never reviewed; no review record was found."
  - "What the 재무기획팀 quarterly close ultimately did with the pending balance. The mail thread of 2026-08-24 ends with the question open."
  - "Who agreed to own the post-event segment. The 2026-06-18 minutes record the disagreement and an action item with no due date."
tags:
  - settlement
  - correction
  - workflow
  - cancel-reconciliation
  - documented-vs-actual
aliases:
  - "정산 정정"
  - "settlement correction"
  - "cancel reconciliation"
relates_to:
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "2025-03 handover — nobody reviews the full queue"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
    notes: "Kickoff minutes — the ownership disagreement between 박성민 and 김도윤"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
    notes: "Automation plan still assuming ~10 cases per month"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
    notes: "2023 draft procedure"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "Issued procedure, approved by the head of Finance 2024-02-19"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512 still To Do since 2023-04-24"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The 2023 decision that created the queue"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "41 months of PENDING counts and amounts"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
    notes: "2026-08-24 finance query — pending balance over 180M KRW"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
    notes: "#settlement-dev, keyword-filtered 2023-04 to 2026-08"
notes: "The pivotal artifact of this reef. Everything else in the settlement domain is context for the gap documented here."
---

# Settlement Correction Workflow

## Purpose

When an order is cancelled after it has already been settled, the money is gone — the payment request cannot be recalled. The agreed remedy is to deduct the amount from the partner's next monthly settlement. This artifact traces that remedy from the issued procedure through to the code, and records where the two part company.

The short version: steps 1 of the procedure is automated and works. Steps 2 through 4 have no implementation that runs. The component written to perform them exists, is complete, and has never been scheduled. Consequently `CANCEL_RECON_QUEUE` is, in practice, append-only.

## Key Facts

- Procedure v1.1 was approved by the head of 재무본부 on 2024-02-19 and carries status 발행 (issued) → sources/context/policy/정산_정정_업무절차_v1.1.md
- v1.1 §4 defines four steps: "1. 취소 접수 → 정정 대기 등록 (시스템) / 2. 정산팀 확인 (월 1회 이상) / 3. 차월 정산 반영 / 4. 파트너 통지 및 이력 기록" — "1. cancellation received → registered as pending correction (system) / 2. settlement team review (at least monthly) / 3. reflected in the next month's settlement / 4. partner notification and history recording" → sources/context/policy/정산_정정_업무절차_v1.1.md
- v1.1 §3 assigns to 정산팀: "정정 대상 확인, 차월 차감 반영, 파트너 통지" — "identifying correction targets, applying next-month deduction, notifying the partner" — and to 주문팀 only "취소 이벤트 발행" (publishing the cancellation event) → sources/context/policy/정산_정정_업무절차_v1.1.md
- v1.1 §5 requires a monthly review: "월 1회 이상 대기 건 현황을 확인한다." — "the pending-case status is reviewed at least once a month" → sources/context/policy/정산_정정_업무절차_v1.1.md
- The 2023 draft v0.3 already assumed a small volume: "예상 처리량 월 10건 미만. 별도 시스템 없이 수기로 처리한다." — "expected volume under 10 cases per month; handled manually without a separate system" → sources/context/policy/정산_정정_업무절차_v0.3.md
- Step 1 is implemented: `OrderEventRelayJob` polls `ORDER_EVENT_OUTBOX` every 10 minutes for `order.cancelled`, checks `SETTLEMENT_DTL` for the order, and inserts a `PENDING` row into `CANCEL_RECON_QUEUE` → src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java
- Steps 2 through 4 have no running implementation. `CancelReconciler.reconcileCancellations()` would insert `SETTLEMENT_ADJUSTMENT` rows and flip queue rows to `PROCESSED`, but it has no Quartz registration and no caller anywhere in the five repos — grep for `CancelReconciler` and `reconcileCancellations` across all repos returns only its own declaration → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- The class documents its own omission: "TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤" — "TODO: needs Quartz schedule registration, to be added to QuartzConfig after confirming with 박성민" → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- Ticket SF-4512 "CancelReconciler Quartz 스케줄 등록" was created 2023-04-24 and is still `To Do`, priority `Low`, with no assignee, as of the 2026-S17 sprint export → sources/context/sprints/tickets_2026-S17.csv
- Even if it ran, the reconciler would insert into `SETTLEMENT_ADJUSTMENT`, a table no migration in the repo creates → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, see [[SCH-SETTLEMENT-BATCH]]
- And even then, nothing applies an adjustment to a payout: no code reads `SETTLEMENT_ADJUSTMENT` or `CANCEL_RECON_QUEUE` when computing a settlement run → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- The 2026-09-01 export counted 4,127 rows still `PENDING`, totalling 188,851,520 KRW, spread over 41 months beginning 2023-04 → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- 2023-04 is the month SF-2287 shipped — the deployment confirmation is dated 2023-04-24 → sources/context/tickets/SF-2287.md, sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- Monthly volume in 2026 runs 139 to 163 cases, against the ~10 per month that both the procedure and the 2026 automation plan still assume → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv, sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md
- Partner-facing reporting omits corrections entirely: "TODO(도윤) 2024-11-02: 취소 정정분은 이 리포트에 포함되지 않는다. 별도 확인 필요." — "cancellation corrections are not included in this report; needs separate checking" → src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java

## Steps

### Step 1 — 취소 접수 → 정정 대기 등록 (cancellation received → pending correction registered)

**Documented:** automated, marked "(시스템)" in v1.1 §4 → sources/context/policy/정산_정정_업무절차_v1.1.md

**Implemented:** yes. Every 10 minutes `OrderEventRelayJob` runs:

1. `SELECT EVENT_ID, ORD_NO, PAYLOAD FROM ORDER_EVENT_OUTBOX WHERE PUBLISHED_YN='N' AND EVENT_TYPE='order.cancelled' ORDER BY REG_DTM LIMIT 500`
2. For each event, `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` — the test for "was this already settled"
3. If settled, `INSERT INTO CANCEL_RECON_QUEUE (ORD_NO, SAYU_CD, STATUS) VALUES (?, ?, 'PENDING')` and log "정산 정정 대기 등록 완료" ("pending correction registration complete")
4. Mark the outbox row `PUBLISHED_YN='Y'` regardless

→ src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java

This step works and has worked since 2023-04. That is precisely why the evidence of the broken downstream is so complete: the queue is a faithful, four-year record of every correction that should have happened.

### Step 2 — 정산팀 확인, 월 1회 이상 (settlement team review, at least monthly)

**Documented:** mandatory monthly review with a record form, ST-F-002 월간 정정 현황 확인서 (Monthly Correction Review), approved by the team lead and retained five years → sources/context/policy/정산_정정_업무절차_v1.1.md (§5, §6)

**Actual:** the 2025-03 handover states plainly that the review does not happen: "전체 대기열을 주기적으로 확인하는 절차는 없음. 문의가 오면 그 건만 본다." — "there is no procedure for periodically reviewing the whole queue; when an inquiry comes in, only that case is looked at" → sources/context/handover/2025-03_정산팀_인수인계.md (§2)

The same document explains why review is impractical: "대기열 건수를 조회하는 화면이 없음. DB 직접 조회만 가능." — "there is no screen for querying the queue count; only direct DB access" → sources/context/handover/2025-03_정산팀_인수인계.md (§3). Three years later this is still true — SF-5120 정산 정정 대기열 조회 화면 (correction queue view screen) was committed to sprint 2026-S17 and carried over as 진행중 (in progress) → sources/context/sprints/2026-S17_log.md

The sprint retrospective describes the workaround: "조회 화면 없이 대기열 건수를 매번 데이터팀에 요청하는 상태가 반복되고 있다." — "without a view screen, the pattern of asking the data team for the queue count every time keeps repeating" → sources/context/sprints/2026-S17_log.md

### Step 3 — 차월 정산 반영 (reflect in next month's settlement)

**Documented:** 정산팀 applies the deduction → sources/context/policy/정산_정정_업무절차_v1.1.md (§3)

**Implemented:** `CancelReconciler` was written for exactly this and is never invoked. What it would do:

```java
INSERT INTO SETTLEMENT_ADJUSTMENT (ORD_NO, SAYU_CD, ADJ_TYPE) VALUES (?, ?, 'CANCEL_CLAWBACK')
UPDATE CANCEL_RECON_QUEUE SET STATUS='PROCESSED', PROCESSED_DTM=NOW() WHERE SEQ=?
```
→ src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java

Three separate blockers sit between this code and a deduction reaching a partner, and each is independently sufficient:

1. No trigger. `QuartzConfig` registers `dailySettlementTrigger` and `orderEventRelayTrigger` and nothing else → src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
2. No target table. No migration creates `SETTLEMENT_ADJUSTMENT` → src/main/resources/db/migration/
3. No consumer. Even a populated `SETTLEMENT_ADJUSTMENT` would not change a payout, because [[PROC-SETTLEMENT-DAILY-BATCH]] computes gross minus a flat fee and never subtracts an adjustment → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java

**Actual practice:** manual, reactive and unrecorded. "실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음." — "in practice only cases where a partner inquired have been checked and handled" → sources/context/handover/2025-03_정산팀_인수인계.md (§2)

### Step 4 — 파트너 통지 및 이력 기록 (partner notification and history recording)

**Documented:** corrections are recorded in the settlement admin and retained five years; ST-F-001 정산 정정 요청서 (Settlement Correction Request) is the prescribed form → sources/context/policy/정산_정정_업무절차_v1.1.md (§5, §6)

**Actual:** "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중." — "there is no official form for recording correction history; each person manages it in their own Excel file" → sources/context/handover/2025-03_정산팀_인수인계.md (§6)

And the automated partner report excludes corrections by the author's own admission, quoted in the Key Facts → src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java

## Worked Examples

### The evidence chain, in order

This is the part worth reading end to end. Eight independent sources, spanning 2023-04 to 2026-09, each of which observes one face of the same gap without anyone assembling them.

**2023-04-07 — the decision.** In SF-2287, CS asks for the post-settlement cancellation block to be removed. 김도윤 accepts the resulting workload: "정산팀에서 수기로 정정 처리하겠습니다. 차월 정산에서 차감하는 방식으로 처리하면 됩니다." — "the settlement team will handle corrections manually; deducting from the next month's settlement works." He sizes it: "실제 정산 후 취소로 이어지는 건은 월 10건 미만일 것으로 예상됩니다." — "cases actually resulting in post-settlement cancellation are expected to be under 10 per month" → sources/context/tickets/SF-2287.md

**2023-04-11 — the commitment.** 박성민 offers the event; 김도윤 replies "네 컨슈머 붙여놓겠습니다." — "yes, I will attach a consumer" → sources/context/tickets/SF-2287.md

**2023-04-21 — deployment.** 박성민 in #settlement-dev: "SF-2287 배포 나갔습니다. 취소 시 order.cancelled 이벤트 발행됩니다." — "SF-2287 is deployed; order.cancelled is published on cancellation" → sources/raw/slack/settlement-dev_2023-04_2026-08.json

**2023-04-24 — the gap opens.** 김도윤: "릴레이까지는 만들어뒀는데 정정 배치 스케줄 등록이 남았습니다. 박성민님 확인 후 추가할게요." — "I have built the relay, but registering the correction batch's schedule is still outstanding; I will add it after confirming with 박성민" → sources/raw/slack/settlement-dev_2023-04_2026-08.json. The same sentence is in the code as a TODO dated 2023-04-24, and the same sentence became ticket SF-4512 on 2023-04-24 → src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sources/context/sprints/tickets_2026-S17.csv

**2023-06-02 — the first miss.** 이수민: "정정 배치 도는 건가요? 이번 달 정산에 차감 반영된 게 안 보여서요" — "is the correction batch running? I do not see any deduction reflected in this month's settlement." 김도윤: "아직입니다. 일단 문의 들어온 건만 수기로 보고 있습니다" — "not yet; for now I am handling manually only the cases that come in as inquiries" → sources/raw/slack/settlement-dev_2023-04_2026-08.json

The gap was known, correctly diagnosed and correctly reported within six weeks of the queue's first row.

**2024-02-13 — deprioritised.** 이수민 asks about the automation; 김도윤: "우선순위에서 밀렸습니다. 건수가 많지 않아서 당장은 수기로 커버 가능합니다" — "it slipped in priority; the volume is not large so manual coverage is fine for now." The export shows 67 new pending rows that month → sources/raw/slack/settlement-dev_2023-04_2026-08.json, sources/raw/exports/cancel_recon_queue_monthly_20260901.csv

**2024-02-19 — the procedure is issued anyway.** Six days after that exchange, v1.1 is approved by the head of Finance, mandating a monthly review that nobody performs → sources/context/policy/정산_정정_업무절차_v1.1.md

**2025-03-21 — the handover.** 정민호 leaves on 2025-03-31 and hands 이수민 a document whose §3 lists, under items unresolved at handover: "대기열을 자동으로 처리하는 배치가 도는지 확인 필요. 2023년에 주문팀에서 이벤트 연동을 해주셨는데, 그 뒤 실제로 차감이 자동으로 되는지는 확인해 본 적 없음. 개발팀에 문의했으나 답변 받지 못했음. (2025-02 문의)" — "need to check whether a batch automatically processes the queue. The order team connected the event in 2023, but whether the deduction actually happens automatically since then has never been checked. Asked the dev team but received no answer (asked 2025-02)" → sources/context/handover/2025-03_정산팀_인수인계.md. §6 lists all of §3 as 미인계 (not handed over).

**2025-07-15 — raised and deflected.** During the duplicate-run postmortem 박성민 writes: "어제 회고에서 나온 건인데, 취소 건이 정산 대상에서 빠지는지 점검 필요해 보입니다. 티켓 만들어 둘까요?" — "from yesterday's retrospective: it seems we need to check whether cancelled orders are excluded from settlement. Shall I make a ticket?" 김도윤: "이번 장애랑은 분리해서 보시죠. 저희 쪽에서 정리해서 올리겠습니다" — "let us treat it separately from this incident; we will write it up on our side." The postmortem's own note closes the loop: "티켓 번호 미확인" — "ticket number unconfirmed" → sources/raw/slack/settlement-dev_2023-04_2026-08.json, sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

**2026-01-08 — the first quantification.** 이수민 asks for a queue count because there is still no screen. 한지우 replies: "뽑아드렸습니다. 생각보다 많습니다. 확인해보시는 게 좋을 것 같아요" — "I pulled it for you. It is more than expected. It would be good to look into it" → sources/raw/slack/settlement-dev_2023-04_2026-08.json

**2026-06-18 — the ownership disagreement.** At the automation kickoff:

- 김도윤: "그럼 차감은 자동으로 들어가고 있는 거죠?" — "so the deduction is going in automatically, right?"
- 박성민: "저희가 이벤트까지는 보내드리고, 그 뒤는 정산 쪽에서 보시는 걸로 알고 있습니다." — "we send the event, and my understanding is that settlement handles it from there"
- The minute-taker adds: "(이 부분 서로 인지가 다름. 확인 필요)" — "the two sides understand this differently; needs confirmation"

→ sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md

Procedure v1.1 §3 in fact answers the question unambiguously — deduction is 정산팀's responsibility — and the settlement team lead is the one asking whether it is automatic. The action item "이벤트 이후 구간 담당 확인" (confirm ownership of the post-event segment) was recorded with no due date, in a list that also contains two blank rows.

The same meeting records the volume assumption surviving intact: "월 10건 내외로 보고 있어 수기로 감당 가능한 수준이라는 것이 2023년 판단이었다." — "the 2023 judgement was that at around 10 cases a month it was manageable manually" — and 이수민's hesitation: "요즘은 좀 더 되는 것 같기는 한데 정확히 세어보진 않았습니다." — "it feels like a bit more these days, but I have not counted precisely" → sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md

**2026-08-18 — the plan repeats the number.** The automation plan, written by 데이터팀 and reviewed by the leads of all three teams, states as its As-Is: "현행 처리량은 정산팀 확인 결과 월 10건 내외로 파악된다." — "current volume is understood, per the settlement team, to be around 10 cases per month." Its As-Is flow diagram ends with "→ 처리 완료" (processing complete), a state 4,127 rows have never reached → sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md

The plan also justifies the priority with a related fact: "정산팀 인원 조정(2024-07, 4명→3명) 이후 업무 부담이 증가했다" — "the workload increased after the settlement team was reduced from 4 to 3 in 2024-07" → sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md, sources/context/org-chart.xlsx (변경이력 sheet)

**2026-08-24 — finance notices.** 문지영 of 재무기획팀 writes to 김도윤, having received his answer that corrections are handled manually in the next month's settlement:

> "그런데 차감 대기 상태로 남아 있는 금액이 결산 기준으로 1.8억을 넘습니다. 2번이 1번과 맞지 않아 보여서 다시 여쭙습니다. 차감이 정상적으로 이루어지고 있다면 대기 잔액이 이 규모로 누적될 수 없습니다. 데이터팀에서 뽑아준 자료로는 2023년 4월 건도 아직 남아 있는 것으로 나옵니다."

"But the amount left pending deduction exceeds 180 million KRW as of the close. Point 2 does not seem consistent with point 1, so I am asking again. If deductions were happening properly, a pending balance could not accumulate to this size. According to the data the data team pulled, even cases from April 2023 are still outstanding."

She asks two questions: how many cases have actually been deducted, and whether the deduction is performed by a batch or by hand. She needs to know whether to book it as a liability at quarter close → sources/raw/mail/RE_정산_미정정_금액_문의.eml

Those two questions are answered by this artifact: none through the system, and by hand, reactively, only when a partner asks.

**2026-09-01 — the measurement.** 윤서진 of 데이터팀 extracts 41 months of monthly `PENDING` counts from the production read replica at the request of 재무기획팀 문지영: 4,127 rows, 188,851,520 KRW, oldest month 2023-04 → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv, sources/raw/exports/README.md

### Volume: the assumption against the data

| Period | Assumed | Measured |
|---|---|---|
| 2023 (v0.3, SF-2287) | under 10/month | 34–69/month from 2023-04 → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| 2026-06 (kickoff minutes) | 월 10건 내외 | 163 in 2026-06 |
| 2026-08 (automation plan) | 월 10건 내외 | 162 in 2026-08 |

Over 41 months the monthly figure grew from 48 to a 2026 range of 139–163, roughly a 14- to 16-fold gap against the planning assumption that has been restated unchanged in every document since 2023 → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv

The figures carry the extraction caveats recorded in the export README; they are handled in [[RISK-SETTLEMENT-RECON-BACKLOG]], which is the artifact to cite for the financial exposure itself.

## Related

- [[SYS-SETTLEMENT]] — the service that owns the queue and the unscheduled reconciler
- [[PROC-SETTLEMENT-DAILY-BATCH]] — the batch whose irreversibility makes this correction path necessary
- [[SCH-SETTLEMENT-BATCH]] — `CANCEL_RECON_QUEUE` and the missing `SETTLEMENT_ADJUSTMENT`
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measured financial exposure
- [[GLOSSARY-SETTLEMENT]] — 정산 정정, 정정 대기 건, 차월 차감, `ADJ_TYPE` `CANCEL_CLAWBACK`
- [[SYS-ORDER]] — the publisher of `order.cancelled`, and the other side of the ownership disagreement
