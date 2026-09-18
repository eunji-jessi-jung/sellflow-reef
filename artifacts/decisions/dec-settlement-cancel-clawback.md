---
id: "DEC-SETTLEMENT-CANCEL-CLAWBACK"
type: "decision"
title: "Next-Month Clawback as Compensation for Post-Settlement Cancels"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Reconstructed on 2026-09-19 from the SF-2287 thread, CancelReconciler, QuartzConfig, the V1 migration comment, procedure v0.3 and v1.1, the SF-4512 ticket row and the #settlement-dev export. This is an ADR for a decision that was made and then not executed; the evidence for the second half is negative evidence, and how it was checked is stated inline."
freshness_triggers:
  - "sellflow-docs:context/sprints/tickets_2026-S17.csv"
  - "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
  - "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "settlement-batch:src/main/resources/db/migration/"
known_unknowns:
  - "Whether 박성민 was ever asked. The TODO says the schedule would be added '박성민님 확인 후' ('after 박성민 confirms'). No such exchange appears in the slack export, the ticket, or either procedure version."
  - "Why SF-4512 was filed at priority Low. It is the only ticket standing between the decision and its execution, and it was filed by the same person who wrote the TODO, three days after the deployment. No justification is recorded anywhere."
  - "Whether SETTLEMENT_ADJUSTMENT was ever designed. The handover says it appears in documentation but cannot be queried; no such document is present in the reef and no migration in settlement-batch creates the table."
  - "How EXPECTED_AMT is computed. The 2026 export sums that column, but neither V1 nor any later migration declares it on CANCEL_RECON_QUEUE, so the clawback amount per row has no defined provenance."
  - "How the manual process that 김도윤 said already existed ('이미 유사 케이스를 월 몇 건씩 수동으로 처리하고 있어서') was carried out before 2023-04, and whether any record of it survives. The handover says corrections are tracked in individual spreadsheets."
  - "Whether any of the 4,127 rows has been settled in reality by a manual correction that left the row PENDING. The export README warns this is possible and the handover says it is the normal way of working."
  - "Whether the partner notification that v1.1 §4 requires has ever been sent for a clawback. No code sends one and the partner-facing report omits corrections."
tags:
  - "settlement"
  - "cancel-reconciliation"
  - "sf-2287"
  - "sf-4512"
  - "adr-reconstructed"
  - "decided-not-executed"
aliases:
  - "차월 정산 차감"
  - "정산 정정"
  - "cancel clawback"
relates_to:
  - type: "depends_on"
    target: "[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]]"
  - type: "feeds"
    target: "[[DEC-SELLFLOW-2026-AUTOMATION-SIZING]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "feeds"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "§2, §3 — how corrections are actually done, and the missing table."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
    notes: "The procedure written to carry the decision, 2023-05-02."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "The approved revision, 2024-02-19."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/2026-S17_log.md"
    notes: "Sprint outcome for the queue-visibility work."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512, To Do, Low, unassigned, created 2023-04-24."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "Where the compensation was offered and accepted."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "The measured result: 4,127 rows / 188,851,520 KRW."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
    notes: "2023-04-24 and 2023-06-02 — the schedule left open, then confirmed not running."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Two jobs registered; the reconciler is not one of them."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
    notes: "The class that would perform the clawback, with the 2023-04-24 TODO."
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "CANCEL_RECON_QUEUE and its SF-2287 comment."
notes: "The measured financial exposure lives in RISK-SETTLEMENT-RECON-BACKLOG; the mechanism trace lives in PROC-SETTLEMENT-CORRECTION. This artifact records the decision itself and its execution status."
---

# Next-Month Clawback as Compensation for Post-Settlement Cancels

## Context

[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] made it possible to cancel an order after the partner had already been paid for it. That created a question the implementer asked out loud before agreeing to build it:

> **박성민 (주문팀), 2023-04-05** — "정산 상태 체크를 빼는 건 어렵지 않습니다. 다만 정산이 이미 나간 건에 대해 취소가 들어오면 그 돈은 어떻게 되나요? 파트너한테 이미 지급된 금액인데요."

"Removing the settlement-state check is not hard. But if a cancel comes in for something already settled, what happens to that money? It's an amount already paid out to the partner."

This artifact records the answer, who gave it, and what became of it. The financial exposure it produced is measured in [[RISK-SETTLEMENT-RECON-BACKLOG]]; the end-to-end mechanism is traced in [[PROC-SETTLEMENT-CORRECTION]].

## Decision

**Settlement will recover the money by deducting it from the partner's next monthly payout.** Agreed 2023-04-07 by 김도윤, team lead of 정산팀, in the SF-2287 thread, accepted implicitly by 박성민, who proceeded to build the event feed on 2023-04-10.

Dated record of what was decided and by whom:

| Date | Who | What |
|---|---|---|
| 2023-04-05 | 박성민 (주문팀) | Raises the money question |
| 2023-04-07 | 김도윤 (정산팀) | Offers manual next-month deduction; forecasts 월 10건 미만 |
| 2023-04-10 | 박성민 (주문팀) | Will publish `order.cancelled` for 정산팀 to consume |
| 2023-04-11 | 김도윤 (정산팀) | "네 컨슈머 붙여놓겠습니다" — "yes, I'll attach a consumer" |
| 2023-04-21 | 박성민 | SF-2287 deployed (order-service 2.8.0) |
| 2023-04-24 | 김도윤 | Relay built; schedule registration left open. Files SF-4512 |
| 2023-05-02 | 정민호 (정산팀) | Writes procedure v0.3 codifying the manual process |
| 2024-02-19 | 정산팀, approved by 재무본부장 | Procedure v1.1 assigns 차월 차감 반영 to 정산팀 as a standing responsibility |

→ sellflow-docs:context/tickets/SF-2287.md, sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json, sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md, sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md, sellflow-docs:context/sprints/tickets_2026-S17.csv

The decision was implemented in three parts. Two of them exist and work:

1. **A queue.** `CANCEL_RECON_QUEUE`, created in `V1__settlement_schema.sql` under a comment naming the ticket: `-- SF-2287 대응. 정산 후 취소 건을 수기 정정용으로 적재한다. / -- 정산팀이 주기적으로 확인하여 차월 정산에서 차감한다.` — "in response to SF-2287. Loads post-settlement cancel cases for manual correction. 정산팀 checks it periodically and deducts in the following month's settlement" → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
2. **A feed.** `OrderEventRelayJob`, registered in Quartz on a ten-minute interval, which is what fills the queue → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
3. **A drain.** `CancelReconciler`, written to read `PENDING` rows, insert a `CANCEL_CLAWBACK` row into `SETTLEMENT_ADJUSTMENT`, and mark the queue row `PROCESSED`. This is the part that was never executed.

## Key Facts

- The compensation was offered by the lead of the team that would perform it, in one comment, and was accepted without a written design → sellflow-docs:context/tickets/SF-2287.md
- The decision was codified as company procedure within a month: v0.3 §3 states 취소 접수 시 CANCEL_RECON_QUEUE 에 정정 대상으로 적재된다 → 정산팀이 대기열을 확인한다 → 차월 정산 시 해당 금액을 차감한다 ("on cancel receipt it is loaded into CANCEL_RECON_QUEUE as a correction target; 정산팀 checks the queue; the amount is deducted at the following month's settlement") → sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md
- v1.1, approved by 재무본부장 on 2024-02-19, makes the same three-way split binding: 정산팀 for 정정 대상 확인, 차월 차감 반영, 파트너 통지; 주문팀 for 취소 이벤트 발행; 재무기획팀 for 분기 결산 시 미정정 잔액 확인 → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md (§3)
- `CancelReconciler` implements exactly the agreed mechanism — its class javadoc reads CANCEL_RECON_QUEUE 의 PENDING 건을 읽어 차월 정산에서 차감 처리한다 ("reads PENDING rows from CANCEL_RECON_QUEUE and processes the deduction in the following month's settlement") → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- It has never been scheduled. `QuartzConfig` declares two `JobDetail`/`Trigger` pairs — `dailySettlementQuartzJob` (`0 0 2 * * ?`, Asia/Seoul) and `orderEventRelayJob` (every 10 minutes) — and nothing else. I checked by reading the whole file and by grepping the repo: `CancelReconciler` is referenced by no other class → settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- Because `@Component` only registers the bean, `reconcileCancellations()` has no invoker of any kind — no scheduler, no controller, no CLI entry point in settlement-batch → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- The gap was known on the day it opened and written into the code as a TODO with an owner, a dependency and a date (quoted in full below) → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- SF-4512 CancelReconciler Quartz 스케줄 등록 was created 2023-04-24 by 김도윤, and is still `To Do`, priority `Low`, `Assignee` blank, `Resolved` blank, as of the 2026-S17 ticket export → sellflow-docs:context/sprints/tickets_2026-S17.csv
- Six weeks after the decision, the team knew the deduction was not happening. 이수민, 2023-06-02: "정정 배치 도는 건가요? 이번 달 정산에 차감 반영된 게 안 보여서요" ("is the correction batch running? I don't see any deductions reflected in this month's settlement"). 김도윤: "아직입니다. 일단 문의 들어온 건만 수기로 보고 있습니다" ("not yet; for now we only handle the ones that come in as inquiries") → sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json
- `SETTLEMENT_ADJUSTMENT`, the table the reconciler inserts into, is created by no migration. I grepped all six files under `src/main/resources/db/migration/` and the whole of settlement-batch: the only occurrence of the name anywhere in the repo is the `INSERT` statement itself → settlement-batch:src/main/resources/db/migration/, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- The 2025 handover reached the same conclusion from the operations side: `SETTLEMENT_ADJUSTMENT` 테이블이 문서에는 나오는데 실제로 조회가 안 됨. WIP — "the SETTLEMENT_ADJUSTMENT table appears in documentation but cannot actually be queried. WIP" → sellflow-docs:context/handover/2025-03_정산팀_인수인계.md (§3)
- The manual half was also never performed as designed. The handover records the real practice: 실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음 ("in practice we've only checked and handled the cases that came in as partner inquiries"), with the explicit warning 전체 대기열을 주기적으로 확인하는 절차는 없음 ("there is no procedure for periodically checking the whole queue") — against v1.1 §4's 정산팀 확인 (월 1회 이상), "checked by 정산팀 at least once a month" → sellflow-docs:context/handover/2025-03_정산팀_인수인계.md (§2), sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md
- The measured outcome after 41 months: 4,127 rows in `STATUS='PENDING'` totalling 188,851,520 KRW, extracted 2026-09-01, with no month in the series showing a zero balance → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv
- A later migration added result columns for the correction that never runs, and says so: 정정 처리 결과를 남기기 위한 컬럼. 아직 쓰는 코드는 없다 — "columns for recording the correction result. No code writes them yet" → settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql
- The service registry has never been able to name the consumer: `CANCEL_RECON_QUEUE` is listed with `producer: settlement-batch (OrderEventRelayJob)` and `consumer: TODO   # 확인 필요` ("needs checking") → sellflow-docs:context/registry/services.yaml

## Rationale

The justification given at the time, verbatim:

> **김도윤 (정산팀), 2023-04-07** — "정산팀에서 수기로 정정 처리하겠습니다. 차월 정산에서 차감하는 방식으로 처리하면 됩니다. 이미 유사 케이스를 월 몇 건씩 수동으로 처리하고 있어서 프로세스 자체는 있습니다.
>
> 물량이 많아지면 자동화가 필요하겠지만, CS팀 통계 보니 실제 정산 후 취소로 이어지는 건은 **월 10건 미만**일 것으로 예상됩니다. 당분간은 수기로 충분합니다."

"정산팀 will handle the corrections manually. Deducting from the following month's settlement is sufficient. We already handle a few similar cases a month by hand, so the process itself exists.

If the volume grows, automation will be needed, but looking at CS's statistics I expect cases that actually lead to a post-settlement cancel to be **fewer than ten a month**. Manual handling is enough for the time being."

Three claims carry the decision: that a manual process already existed, that the volume would be small, and that automation was a later problem gated on volume growing. The first is asserted but not evidenced anywhere in the reef — no pre-2023 correction record, form or spreadsheet is present, and the 2025 handover says corrections are tracked 각자 엑셀로 ("each person in their own Excel"). The second is the sizing assumption examined in [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]]; the export contradicts it by an order of magnitude. The third was written into v0.3 §5 as a gate with no threshold: 건수가 늘어나면 자동화를 검토한다. (기준 TBD) — "if the case count grows, automation will be reviewed. (criterion TBD)" → sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md

No rationale is recorded anywhere for the second decision inside this one — the choice to write `CancelReconciler` and then not schedule it. The TODO names a dependency but not a reason for it, and nothing in the slack export, the ticket, either procedure or the sprint records explains why SF-4512 was filed at `Low`. That is recorded in `known_unknowns` rather than inferred.

## Consequences

**This is an ADR for a decision that was made and then not executed.** Stating it in those terms is the point of the artifact: the agreement was real, documented, procedurally ratified and technically implemented, and the one step that would have made it operate was never taken. Everything below follows from that.

**It was never scheduled, and the code says so.** The TODO in `CancelReconciler`, verbatim:

```java
 * <p>TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정)
 *      - 2023-04-24 김도윤
```

"TODO: Quartz schedule registration needed (to be added to QuartzConfig after 박성민 confirms) — 2023-04-24 김도윤"

Three days after the order-side change deployed, the compensating mechanism was complete except for one line of configuration, blocked on a confirmation from another team. As of 2026-09-19 that comment is 1,244 days old and `QuartzConfig` still registers two jobs, neither of them this one → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java

**Scheduling it alone would fail.** This matters for anyone about to close SF-4512 and consider the problem solved. The first thing the job does per row is:

```java
"INSERT INTO SETTLEMENT_ADJUSTMENT (ORD_NO, SAYU_CD, ADJ_TYPE) VALUES (?, ?, 'CANCEL_CLAWBACK')"
```

No migration in `settlement-batch` creates `SETTLEMENT_ADJUSTMENT`. The six files under `src/main/resources/db/migration/` cover the settlement schema, a run log, partner contracts, two `CANCEL_RECON_QUEUE` index/column changes and a run-log index — and none of them mentions the table. The 2025 handover independently reports that it cannot be queried. So a registered trigger would run, load 4,127 pending rows, and fail on the first insert; and because each row's `INSERT` and `UPDATE` are issued as separate `JdbcTemplate` calls with no surrounding `@Transactional`, the failure mode on a partially-working schema would be worth checking before anyone runs it against production → settlement-batch:src/main/resources/db/migration/, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sellflow-docs:context/handover/2025-03_정산팀_인수인계.md (§3)

**The measured result.** 4,127 rows and 188,851,520 KRW accumulated across 41 consecutive months, 2023-04 through 2026-08, with no zero month anywhere in the series — the shape you would expect from a queue with a producer and no consumer. The oldest month, 2023-04, is the month SF-2287 shipped. The extraction caveats that bound this figure in both directions are set out in [[RISK-SETTLEMENT-RECON-BACKLOG]] and are not repeated here → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv

**The procedural consequence.** Because v1.1 is an approved procedure, the un-executed half of this decision is not merely a backlog item — it is a standing responsibility of 정산팀 (차월 차감 반영, 파트너 통지) that has not been discharged for any of the 4,127 rows, and a standing responsibility of 재무기획팀 (분기 결산 시 미정정 잔액 확인) that surfaced the problem in 2026 exactly as written → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md

**The team cannot see its own queue.** SF-5120, a screen to view the correction queue, was still `In Progress` at the end of sprint 2026-S17, and the retrospective names the consequence: 조회 화면 없이 대기열 건수를 매번 데이터팀에 요청하는 상태가 반복되고 있다 — "we keep having to ask the data team for the queue count because there is no view screen" → sellflow-docs:context/sprints/2026-S17_log.md

**What it would take to execute the decision as agreed**, from the code rather than from a plan: create `SETTLEMENT_ADJUSTMENT`; establish where the per-row amount comes from, since the column the 2026 export sums (`EXPECTED_AMT`) is not declared by any migration; write the code that applies an adjustment to a payout, which does not exist; add the partner notification v1.1 §4 requires; and only then register the trigger. SF-4512 is the last of those five steps, not the only one.

## Related

- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — the decision this one was the compensation for
- [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]] — the 월 10건 미만 forecast used to justify manual handling
- [[PROC-SETTLEMENT-CORRECTION]] — the full mechanism trace
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measured exposure and its caveats
- [[SCH-SETTLEMENT-BATCH]] — `CANCEL_RECON_QUEUE` and the missing `SETTLEMENT_ADJUSTMENT`
- [[SYS-SETTLEMENT]] — the owning service
