---
id: "RISK-SETTLEMENT-RECON-BACKLOG"
type: "risk"
title: "Cancel Reconciliation Queue Backlog"
domain: "settlement"
status: "draft"
last_verified: 2026-09-18
freshness_note: "Figures are a point-in-time extraction from 2026-09-01, 17 days before this verification, against a queue that grows by roughly 145 rows a month. The extraction caveats in the export README are load-bearing and are restated below. Re-extract before citing the amount anywhere it matters."
freshness_triggers:
  - "sources/context/sprints/tickets_2026-S17.csv"
  - "sources/raw/exports/cancel_recon_queue_monthly_20260901.csv"
  - "src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
known_unknowns:
  - "The true total. The export returned only STATUS='PENDING' rows; rows in any other state were not returned, so both the count and the amount are lower bounds on what was queued and upper bounds on nothing."
  - "How much of the balance was settled manually outside the system. The export README warns that manual corrections handled outside the system are not reflected, and the handover says manual handling leaves no system record."
  - "The provenance of EXPECTED_AMT. The export sums it, but no migration in settlement-batch defines that column — so how the per-row amount is computed and by what is unknown."
  - "Whether any of the 4,127 rows is a duplicate. The relay inserts without a uniqueness check, so an order cancelled twice, or an outbox row relayed twice, would produce two rows."
  - "Whether 재무기획팀 booked the balance as a liability at the 2026 Q3 close. The mail thread ends with the question open."
  - "How many SETTLEMENT_ANOMALY rows with ANOMALY_CD='CANCELLED_SETTLED' exist. No export of that table was available, and no caller for POST /detect was found."
  - "Whether the population in CANCEL_RECON_QUEUE and the population the CANCELLED_SETTLED rule would flag are the same set. They are defined differently — one by an outbox event, the other by the order's current state at detection time."
severity: "medium"
resolution: "open"
tags:
  - settlement
  - financial-exposure
  - backlog
  - cancel-reconciliation
aliases:
  - "정정 대기 잔액"
  - "미정정 금액"
relates_to:
  - type: "refines"
    target: "[[API-SETTLEMENT-ANOMALY]]"
  - type: "depends_on"
    target: "[[PROC-SELLFLOW-CANCEL-MONEY-PATH]]"
  - type: "depends_on"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
  - type: "refines"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "The note that no remediation owner is defined for anomaly output"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512 To Do, SF-5120 In Progress, SF-5121 To Do"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "settlement-anomaly — the CANCELLED_SETTLED rule"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
    notes: "Extraction provenance and caveats"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "41 monthly rows plus a total line"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
notes: "This is the financial-exposure artifact. The mechanism behind it is documented separately in PROC-SETTLEMENT-CORRECTION."
---

# Cancel Reconciliation Queue Backlog

## Description

`CANCEL_RECON_QUEUE` accumulates one row for every order that was cancelled after it had already been settled — money already sent to a partner for goods the customer no longer has. Each row represents an amount that should be deducted from that partner's next monthly payout.

Nothing drains the queue. The component written to do so, `CancelReconciler`, has never been registered on a schedule; the mechanism is traced in [[PROC-SETTLEMENT-CORRECTION]]. The result is 41 consecutive months of accumulation, first measured on 2026-09-01 at the request of 재무기획팀, who needed to know whether to book the balance as a liability.

This artifact holds the measurement and its caveats. It deliberately does not restate the causal chain.

## Key Facts

- The queue holds 4,127 rows in `STATUS='PENDING'`, totalling 188,851,520 KRW, across 41 months from 2023-04 to 2026-08 → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- The oldest month is 2023-04, the month SF-2287 shipped and the relay began writing rows → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv, sources/context/tickets/SF-2287.md
- Not a single month in 41 has a zero balance, so no drain event of any kind is visible in the data → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- Monthly volume rose from 48 in 2023-04 to a 2026 range of 139 to 163 → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- The extraction was run on 2026-09-01 at 14:22 against `settlement_prod` (read replica) by 윤서진 of 데이터팀, requested by 문지영 of 재무기획팀 → sources/raw/exports/README.md, sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- Finance independently reached the same order of magnitude before the export, and stated the contradiction plainly: "차감이 정상적으로 이루어지고 있다면 대기 잔액이 이 규모로 누적될 수 없습니다." — "if deductions were happening properly, a pending balance could not accumulate to this size" → sources/raw/mail/RE_정산_미정정_금액_문의.eml
- The queue count is not observable to its own owners. There is no screen; SF-5120 정산 정정 대기열 조회 화면 was still `In Progress` at the end of sprint 2026-S17 → sources/context/sprints/tickets_2026-S17.csv, sources/context/handover/2025-03_정산팀_인수인계.md (§3)
- The fix is a registered Quartz trigger. SF-4512 has been `To Do`, priority `Low`, unassigned, since 2023-04-24 → sources/context/sprints/tickets_2026-S17.csv
- `settlement-anomaly`'s `CANCELLED_SETTLED` rule would flag this same population independently: any settled order whose `ORDER_MST.SANGTAE_CD` is `CHWISO` or `BANPUM` → model/detector.py
- That second signal has no owner either: "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" — "performs detection only; the party responsible for action is not defined" → sources/context/registry/services.yaml
- And it may not run at all: no caller for `POST /detect` exists in any of the five repos → see [[API-SETTLEMENT-ANOMALY]]

## Impact

### The measured figures

| Measure | Value | Source |
|---|---|---|
| Rows in `STATUS='PENDING'` | 4,127 | sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| Estimated uncorrected amount | 188,851,520 KRW | sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| Months covered | 41 (2023-04 through 2026-08) | sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| Oldest outstanding month | 2023-04, 48 rows, 2,196,480 KRW | sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| Most recent full month | 2026-08, 162 rows, 7,413,120 KRW | sources/raw/exports/cancel_recon_queue_monthly_20260901.csv |
| Extraction date | 2026-09-01 14:22, `settlement_prod` read replica | sources/raw/exports/README.md |

Growth by year, from the monthly rows: 2023 (9 months from April) 485 rows; 2024 987; 2025 1,450; 2026 (8 months) 1,205 — a 41-month mean of 100.7 rows a month, with the trailing twelve months at 145.2 and still rising → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv

### The extraction caveats

These are not decoration. The export README states them, and citing the figure without them would overstate its precision in both directions.

1. **Only `PENDING` rows were returned.** "STATUS 가 PENDING 외의 값을 가진 행은 조회되지 않았다." — "rows with a STATUS other than PENDING were not returned." The query's `WHERE STATUS='PENDING'` means the export cannot say whether any row has ever been marked `PROCESSED`, and therefore cannot by itself prove that nothing drains the queue. The code is what proves that → sources/raw/exports/README.md, src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
2. **Manual corrections outside the system are invisible.** "수기 정정분이 시스템 밖에서 처리되었다면 이 수치에 반영되지 않는다." — "if manual corrections were handled outside the system, they are not reflected in these figures." The handover confirms this is exactly how corrections have been done — reactively, for partner-reported cases only, tracked in individual spreadsheets. So some portion of the 188.8M has been settled in reality while remaining `PENDING` in the table → sources/raw/exports/README.md, sources/context/handover/2025-03_정산팀_인수인계.md (§2, §6)
3. **The amount column's provenance is unverified.** The query sums `EXPECTED_AMT`, a column no migration in `settlement-batch` defines. The header calls the result 추정 미정정 금액 (estimated uncorrected amount), which is the right hedge → sources/raw/exports/README.md, src/main/resources/db/migration/
4. **The export is not reproducible on demand.** "재추출 방법 ... (스크립트 위치 TBD — 현재는 DBA 에게 요청)" — "re-extraction method ... script location TBD; currently request it from the DBA" → sources/raw/exports/README.md
5. **It is a point in time.** Seventeen days have passed since the extraction (2026-09-01 → 2026-09-18); at roughly 145 rows a month that is on the order of 80 further rows, not a material change to the order of magnitude.

Read together: 188,851,520 KRW is the amount recorded as awaiting deduction, not the amount actually owed. The unrecoverable part of the real figure is bounded below by how much was quietly fixed by hand — a quantity that, by the team's own account, was never written down anywhere the system can see.

### Financial consequence

문지영 of 재무기획팀 asked the two questions this artifact answers, for a specific reason — whether to recognise the balance as a liability at quarter close:

> "분기 결산에 부채로 인식해야 하는지 판단이 필요합니다. 아래 두 가지만 확인 부탁드립니다. - 현재까지 실제로 차감 처리된 건수 - 차감을 수행하는 주체 (배치인지 수기인지)"

"A judgement is needed on whether to recognise this as a liability at the quarterly close. Please confirm just these two things: the number of cases actually deducted to date, and who performs the deduction — a batch or a person."

→ sources/raw/mail/RE_정산_미정정_금액_문의.eml

From the code: the number deducted through the system is zero, because the only component that writes a deduction has never run. The party performing deductions is a person, reactively, only when a partner asks. Note that procedure v1.1 §3 already assigns quarterly verification of exactly this balance to 재무기획팀 — "분기 결산 시 미정정 잔액 확인" — so the review that surfaced the problem in 2026 was mandated in 2024 → sources/context/policy/정산_정정_업무절차_v1.1.md

### Partner consequence

Each row is an amount a partner was paid and should not have kept. Two properties make it worse for the partner relationship than for the books: the partner-facing report omits corrections entirely, and corrections surface only through partner inquiry. So a deduction, when one does eventually happen, arrives without a paper trail the partner can reconcile against → src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java, sources/context/handover/2025-03_정산팀_인수인계.md (§2)

Procedure v1.1 §5 says "정정 이력은 정산 어드민에 기록하며 5년간 보존한다." — "correction history is recorded in the settlement admin and retained for five years." The oldest queue rows date from 2023-04, so on the arithmetic they are not yet past that window; five years from 2023-04 falls in 2028-04. The sharper problem is that the clock has no start event: the retention rule covers *correction history*, which comes into existence only once a correction is performed, and no correction has been performed for any of the 4,127 rows. There is therefore no retained record to expire, and whether a 2023-04 case can still be substantiated to a partner depends on evidence the procedure never required anyone to create → sources/context/policy/정산_정정_업무절차_v1.1.md, see [[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]]

### The second, independent signal

`settlement-anomaly` would surface the same problem by a different route. Its `CANCELLED_SETTLED` rule flags any settled order whose current state is `CHWISO` or `BANPUM`, writing a `SETTLEMENT_ANOMALY` row with score 1.0 → model/detector.py

Two organisations would therefore be holding evidence of the same exposure: 정산팀 in `CANCEL_RECON_QUEUE` and 데이터팀 in `SETTLEMENT_ANOMALY`. Neither table has a defined consumer. The registry says so of the anomaly output in as many words, and `SETTLEMENT_ANOMALY`'s `REVIEWED_BY` and `REVIEWED_DTM` columns have no writer in any repo → sources/context/registry/services.yaml, sql/V1__anomaly_schema.sql

Whether the second signal exists at all depends on whether anything calls `POST /detect`, which nothing in the five repos does. That makes the detector a latent control rather than an operating one — and worth noting for anyone tempted to treat it as a compensating control during a remediation plan.

## Severity and Resolution

**Severity:** medium, at snorkel depth, which is the floor rather than a considered rating. The measured amount, the 41-month duration, the named stakeholder awaiting an accounting judgement, and the fact that the balance has grown every year since 2023 would all argue for higher after a proper assessment. It is recorded as medium here because this pass measured the exposure rather than evaluating it.

**Resolution:** open.

- SF-4512 (schedule the reconciler) — `To Do`, unassigned, `Low` priority, since 2023-04-24 → sources/context/sprints/tickets_2026-S17.csv
- SF-5120 (a screen to see the queue) — `In Progress`, carried into the next sprint → sources/context/sprints/2026-S17_log.md
- SF-5121 (define the correction decision criteria) — `To Do`, 착수 못함 ("could not be started") → sources/context/sprints/2026-S17_log.md
- The finance mail of 2026-08-24 — no reply is present in the material available
- Even with SF-4512 done, two blockers remain: no migration creates `SETTLEMENT_ADJUSTMENT`, and no code applies an adjustment to a payout → src/main/resources/db/migration/, see [[SCH-SETTLEMENT-BATCH]]

**Before citing these figures anywhere consequential, re-extract.** The export README says so first: "이 폴더의 파일은 특정 시점에 운영 DB 에서 뽑은 것이다. 재추출 없이 그대로 인용하지 말 것." — "the files in this folder were pulled from the production DB at a particular moment. Do not cite them as-is without re-extracting" → sources/raw/exports/README.md

## Related

- [[PROC-SETTLEMENT-CORRECTION]] — why the queue never drains, with the full evidence chain
- [[RISK-SETTLEMENT]] — the other risk themes in this domain
- [[SCH-SETTLEMENT-BATCH]] — `CANCEL_RECON_QUEUE` and the missing `SETTLEMENT_ADJUSTMENT`
- [[SYS-SETTLEMENT]] — the owning service
- [[SYS-SETTLEMENT-ANOMALY]] — the second, unowned signal
- [[API-SETTLEMENT-ANOMALY]] — why that second signal may not be running
- [[PROC-SELLFLOW-CANCEL-MONEY-PATH]] — where in the chain the money stops, and which figures are provable versus estimated
