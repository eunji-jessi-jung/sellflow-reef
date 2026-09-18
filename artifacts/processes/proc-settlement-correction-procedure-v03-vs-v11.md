---
id: "PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11"
type: "process"
title: "정산 정정 업무절차 — v0.3 vs v1.1, a Documentary Diff"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "A line-by-line diff of the two issued versions of the correction procedure, performed 2026-09-19, with each clause tested against the code in settlement-batch and against the 2025 handover, the 2026 minutes, the 2026 plan and the 2026-09-01 queue export. The diff itself is stable — both documents are finished artefacts and v0.3 is explicitly retained for history. What will go stale is the implementation column: it becomes wrong the moment SF-4512 registers CancelReconciler, or a 정산 어드민 correction log appears. The retention arithmetic is date-dependent and is stated with its reference date."
freshness_triggers:
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
  - "sources/context/policy/정산_정정_업무절차_v1.1.md"
  - "sources/context/sprints/tickets_2026-S17.csv"
  - "sources/raw/exports/cancel_recon_queue_monthly_20260901.csv"
known_unknowns:
  - "Whether a v1.0 ever existed. The revision history in v1.1 §7 lists only v0.3 (2023-05-02) and v1.1 (2024-02-19); the jump from 0.3 to 1.1 is unexplained and no intermediate draft is present in sources/context/policy/ or sources/context/_archive/."
  - "Who at 재무본부 approved v1.1. The header records the approver only by title — 승인: 재무본부장 — and the org chart names 본부 but not 본부장, so the approver cannot be resolved to a person from the available material."
  - "Why the 예상 처리량 월 10건 미만 clause and the automation-review trigger were deleted in v1.1. The revision-history row mentions neither deletion, and no minute, ticket or mail in the available material discusses the change."
  - "Whether the ST-F-001 and ST-F-002 forms mandated by v1.1 §6 exist as templates anywhere. Neither is present in sources/, and the 2025 handover says no official record format exists in practice."
  - "Where the 정산 어드민 that v1.1 §5 names as the system of record for correction history lives. No such application exists in any of the five repositories; the handover lists it as an access target requiring 팀장 approval, so it exists as something, but not as code the reef can see."
  - "Whether the five-year retention period in v1.1 §5 runs from the correction date, the original settlement date or the cancellation date. The clause says only 정정 이력은 ... 5년간 보존한다 without naming the start event, and the difference decides whether any 2023 case has expired."
  - "Whether CANCEL_RECON_QUEUE rows are themselves covered by any retention or archival policy. Neither version of the procedure mentions the queue table as a record; V1 creates it with no retention comment, and no DBA policy document is present."
  - "How many corrections have ever been recorded on either form, or anywhere. The handover says corrections are tracked in individual spreadsheets; none are present in the available material."
  - "Whether the quarterly check assigned to 재무기획팀 in v1.1 §3 was performed before 2026-08-24. The mail of that date reads as a first discovery, but its absence from earlier material is not proof it never happened."
tags:
  - settlement
  - policy
  - correction
  - document-diff
  - documented-vs-actual
  - retention
aliases:
  - "정산 정정 업무절차 개정 비교"
  - "correction procedure v0.3 v1.1 diff"
relates_to:
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
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
    ref: "sellflow-docs:context/_archive/2024_정산정정_자동화_검토안_초안.md"
    notes: "2024-03-11 automation review draft — shelved three weeks after v1.1 was issued"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "The 01-04 cancellation-reason vocabulary the queue's SAYU_CD fallback falls outside of"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "The 2025 handover — the only first-hand account of how the procedure is actually performed"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
    notes: "Where the deleted volume figure resurfaced in 2026"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "정산팀 and 재무기획팀 placement, and the 2024-07 headcount reduction"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
    notes: "The 2026 plan restating 월 10건 내외 as present-tense current volume"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
    notes: "The 2023-05-02 draft — left side of the diff"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "The 2024-02-19 issued version — right side of the diff"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "CANCEL_RECON_QUEUE consumer TODO"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512, SF-5120, SF-5121 status as of sprint 2026-S17"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "김도윤's 2023-04-07 comment — the origin of the 월 10건 figure"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
    notes: "Extraction caveats that bound the measured contradiction"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "41 monthly rows — the measurement the volume clause is tested against"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
    notes: "2026-08-24 — the first evidenced execution of the v1.1 §3 quarterly clause"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
    notes: "2023-06-02 and 2024-02-13 exchanges bracketing the v1.1 revision"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
    notes: "Two registered triggers; no reconciler trigger"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
    notes: "The code for v1.1 §4 step 3, never scheduled"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
    notes: "The code for v1.1 §4 step 1 — the only implemented clause"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
    notes: "v1.1 §4 step 4 파트너 통지 — never called, and its TODO says corrections are excluded"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
    notes: "CANCEL_RECON_QUEUE columns available for substantiating an old case"
notes: "A documentary diff. The mechanism and the money are traced in PROC-SETTLEMENT-CORRECTION and RISK-SETTLEMENT-RECON-BACKLOG; this artifact is about what the two policy documents say, what changed between them, and which of their clauses the code performs. It corrects one claim in RISK-SETTLEMENT-RECON-BACKLOG — see 'The five-year retention window, measured'."
---

# 정산 정정 업무절차 — v0.3 vs v1.1, a Documentary Diff

## Purpose

Two versions of the settlement correction procedure exist in `sources/context/policy/`. v0.3 (2023-05-02) is a draft written by one person weeks after the code change that created the need for it. v1.1 (2024-02-19) is an issued, division-head-approved procedure that superseded it. v0.3 is retained deliberately: "v1.1 로 개정됨. 이력 보존용으로 남겨둔다." — "revised into v1.1. Retained for the historical record."

An agent asked "what is the settlement correction procedure" will find both and must know which is authoritative, what the revision actually changed (the revision-history row does not say), and — the part no document states — which clauses of either version the running code performs. That last question is the point of this artifact: the answer is one clause, of one version, out of twelve numbered obligations across the two.

## Key Facts

- v0.3 is a draft: 제정 2023-05-02, 작성 정산팀 정민호, 상태 초안 ("enacted 2023-05-02, written by 정민호 of 정산팀, status: draft"). v1.1 is issued and approved: 개정 2024-02-19, 작성 정산팀, 승인 재무본부장, 상태 발행 ("revised 2024-02-19, written by 정산팀, approved by the Finance division head, status: issued") → sources/context/policy/정산_정정_업무절차_v0.3.md, sources/context/policy/정산_정정_업무절차_v1.1.md
- v0.3 was written eleven days after the code it governs was confirmed deployed — 최은영 of CS confirmed the SF-2287 release on 2023-04-24, and the procedure is dated 2023-05-02. The behaviour existed before the procedure describing it → sources/context/tickets/SF-2287.md, sources/context/policy/정산_정정_업무절차_v0.3.md
- The revision grew the document from five sections to seven and changed its character: v1.1 added 용어 (terms), 기록 및 보존 (records and retention), 서식 (forms) and 개정 이력 (revision history), and deleted 적용 범위 (scope) and 비고 (remarks) → both policy files
- v1.1's own revision-history row describes the change as "책임 구분 명확화, 보존기간·서식 추가" — "clarification of responsibility allocation; retention period and forms added". It does not mention that two clauses were deleted, one of which was the expected-volume figure that every later document went on to cite → sources/context/policy/정산_정정_업무절차_v1.1.md
- The deleted volume clause read: "예상 처리량 월 10건 미만. 별도 시스템 없이 수기로 처리한다." — "expected throughput under 10 cases a month. Handled manually, with no separate system." → sources/context/policy/정산_정정_업무절차_v0.3.md
- That figure originates in a single Jira comment, not a measurement. 김도윤 (정산팀), SF-2287, 2023-04-07: "CS팀 통계 보니 실제 정산 후 취소로 이어지는 건은 **월 10건 미만**일 것으로 예상됩니다. 당분간은 수기로 충분합니다." — "looking at the CS team's statistics, I expect cases that actually lead to a post-settlement cancellation to be **under 10 a month**. Manual handling will be enough for now." The verb is 예상됩니다 (I expect / it is anticipated) — a forecast made two weeks before the feature shipped → sources/context/tickets/SF-2287.md
- Although v1.1 deleted the figure from policy, it survived outside policy and drifted upward in confidence: the 2026-06-18 kickoff minutes record "월 10건 내외로 보고 있어 수기로 감당 가능한 수준이라는 것이 2023년 판단이었다" ("we regard it as around 10 a month, and that manual handling is manageable, was the 2023 judgement"), and the 2026-08-18 automation plan states it in the present tense as a verified current figure: "📌 현행 처리량은 정산팀 확인 결과 **월 10건 내외**로 파악된다." — "the current throughput is understood, per confirmation with the settlement team, to be **around 10 cases a month**." → sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md, sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md
- The same minutes record the practitioner's own caveat, which the plan did not carry forward. 이수민: "요즘은 좀 더 되는 것 같기는 한데 정확히 세어보진 않았습니다." — "it does seem to be a bit more these days, but I haven't counted exactly." An action item "현행 월 처리 건수 실측 — 정산팀 (기한 미정)" ("measure actual monthly case volume — settlement team, no deadline set") was opened and left undated → sources/context/minutes/2026-06-18_정산정정_자동화_킥오프.md
- The 2026-09-01 export measures the queue at 139 to 163 rows a month across 2026 — roughly fourteen to sixteen times the figure, and the very first month on record (2023-04, 48 rows) already exceeded it nearly fivefold, i.e. the forecast was wrong from the month the feature shipped, not merely outgrown → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv
- v1.1 introduced a third party with no counterpart in v0.3. Its §3 responsibility table assigns 재무기획팀 exactly one duty: "분기 결산 시 미정정 잔액 확인" — "verify the uncorrected balance at the quarterly close". v0.3 §4 named only two parties, 정산팀 and 주문팀 → both policy files
- The first evidence of that quarterly check being performed is the mail thread of 2026-08-24, two and a half years after the duty was assigned, and it reads as a discovery rather than a routine: 문지영 of 재무기획팀 writes "차감이 정상적으로 이루어지고 있다면 대기 잔액이 이 규모로 누적될 수 없습니다" — "if deductions were happening properly, a pending balance could not accumulate to this size" → sources/raw/mail/RE_정산_미정정_금액_문의.eml
- v1.1 introduced retention, also with no counterpart in v0.3: "정정 이력은 정산 어드민에 기록하며 5년간 보존한다." — "correction history is recorded in the settlement admin and retained for five years." Both forms in §6 carry 보존 5년 in their own column → sources/context/policy/정산_정정_업무절차_v1.1.md
- Of the twelve numbered obligations across the two versions, the code performs one: v0.3 §3 step 1 / v1.1 §4 step 1, queue registration, implemented by `OrderEventRelayJob`. Every other clause — the review cadence, the deduction, the partner notification, the history record, the forms, the quarterly balance check — has no implementing code in any of the five repositories → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java
- The 2024 automation review draft is dated 2024-03-11 — twenty-one days *after* v1.1 was issued, not between the versions — and it contradicts itself in two lines: it was shelved because "처리량이 많지 않아 당장의 필요성이 낮다고 판단" ("judged low priority as throughput is not large"), while its own open-items section says "대기열에 쌓인 건이 실제로 얼마나 되는지 확인되지 않음" ("it has not been confirmed how many cases have actually accumulated in the queue") → sources/context/_archive/2024_정산정정_자동화_검토안_초안.md
- As of 2026-09-19 no queue row has yet passed a five-year mark: the oldest `PENDING` month is 2023-04, which reaches five years in 2028-04. The retention clause is nonetheless inoperative for these rows for a different reason — it protects 정정 이력 (correction history) in the 정산 어드민, and no correction history exists for any of the 4,127 rows, because no correction was performed → sources/raw/exports/cancel_recon_queue_monthly_20260901.csv, sources/context/policy/정산_정정_업무절차_v1.1.md, sources/context/handover/2025-03_정산팀_인수인계.md

## Scope

In scope: a section-by-section diff of `정산_정정_업무절차_v0.3.md` and `정산_정정_업무절차_v1.1.md`; the provenance of the clauses that changed; the 2024 archived review draft as evidence of what happened around the revision; and a clause-by-clause test of both versions against the code in `settlement-batch` and the operational record.

Out of scope: the causal chain by which corrections fail to happen ([[PROC-SETTLEMENT-CORRECTION]]) and the size of the resulting exposure ([[RISK-SETTLEMENT-RECON-BACKLOG]]). Both are cited here where a clause depends on them, and neither is restated.

## Current State

### Section-by-section change table

| v0.3 | v1.1 | Change | Material content |
|---|---|---|---|
| Header — 제정 2023-05-02, 작성 정산팀 정민호, 상태 **초안** | Header — 개정 2024-02-19, 작성 정산팀, 승인 **재무본부장**, 상태 **발행** | **Promoted** draft → issued | Authorship moved from a named individual to the team; an approver appears for the first time. 정민호 subsequently left the company on 2025-03-31, so the only named author of either version is gone (handover header) |
| §1 목적 — "정산 완료 이후 발생한 주문 취소 건의 금액 정정 **절차를** 정한다" | §1 목적 — "... 금액 정정 **절차와 책임을** 정한다" | **Reworded** | Four characters added: 와 책임 ("and responsibility"). The document's purpose expands from defining a procedure to assigning accountability |
| §2 적용 범위 — "정산 배치가 지급 요청을 생성한 이후 취소가 접수된 건" ("cases where a cancellation was received after the settlement batch generated a payment request") | — | **Removed** | The trigger condition is deleted as a section. v1.1 §2.2 partly absorbs it as a definition ("취소가 정산 이후에 접수되어 차감이 필요한 상태로 적재된 건" — "a case loaded in a state requiring deduction because the cancellation was received after settlement"), but the precise boundary — *payment request generated* — is lost |
| — | §2 용어: 2.1 정산 정정, 2.2 정정 대기 건 | **Added** | 2.1: "지급이 완료된 정산 금액을 차월 정산에서 차감하여 조정하는 것" — "adjusting an already-paid settlement amount by deducting it from the following month's settlement". The first formal definition of the domain term in any Sellflow document |
| §4 담당 — two bullets: 대기열 확인 및 차감 → 정산팀; 이벤트 발행 → 주문팀 | §3 책임 — three-row table | **Promoted and expanded** | 정산팀 gains 파트너 통지 (partner notification): "정정 대상 확인, 차월 차감 반영, 파트너 통지". 주문팀 unchanged ("취소 이벤트 발행"). **재무기획팀 is new**: "분기 결산 시 미정정 잔액 확인" — "verify the uncorrected balance at the quarterly close" |
| §3 절차 — 3 steps | §4 절차 — 4 steps | **Expanded** | Step 1 gains the tag (시스템) marking it as system-performed. Step 2 gains a cadence: "정산팀 확인 (**월 1회 이상**)" — "settlement team review (**at least once a month**)", where v0.3 said only "정산팀이 대기열을 확인한다" with no frequency. Step 4 is new: "파트너 통지 및 이력 기록" — "partner notification and history recording" |
| §5 비고 — "예상 처리량 **월 10건 미만**. 별도 시스템 없이 수기로 처리한다." | — | **Removed** | The expected-volume figure and the explicit statement that handling is manual with no system both disappear from policy. See "The volume clause" below |
| §5 비고 — "건수가 늘어나면 자동화를 검토한다. (기준 TBD)" ("if the case count grows, automation will be considered; threshold TBD") | — | **Removed** | The only self-correcting mechanism in v0.3 — a trigger to revisit the manual approach — is deleted without replacement. Nothing in v1.1 obliges anyone to reconsider the design as volume changes |
| — | §5 기록 및 보존 | **Added** | "정정 이력은 정산 어드민에 기록하며 **5년간 보존한다**" — "correction history is recorded in the settlement admin and **retained for five years**"; plus "월 1회 이상 대기 건 현황을 확인한다" — "the status of pending cases is reviewed at least once a month", restating §4 step 2 as a records obligation |
| — | §6 서식 — ST-F-001 정산 정정 요청서 (Settlement Correction Request), ST-F-002 월간 정정 현황 확인서 (Monthly Correction Review) | **Added** | Both 승인: 팀장 (team-lead approved), both 보존: 5년. The first controlled forms in the correction process |
| — | §7 개정 이력 | **Added** | Two rows: v0.3 2023-05-02 제정(초안); v1.1 2024-02-19 "책임 구분 명확화, 보존기간·서식 추가". No v1.0 appears |

Net: v1.1 is a compliance document. It adds definitions, named responsibilities, a cadence, a retention period, controlled forms and an approver; it removes the two clauses that described the design honestly as provisional (a volume estimate and a trigger to revisit it). What was gained is auditability. What was lost is the document's own record that its approach had an expiry condition.

### The volume clause and where it came from

This is the single clause with the longest downstream consequence, so its chain is set out in full.

| When | Where | What it says | Status of the claim |
|---|---|---|---|
| 2023-04-07 | SF-2287, 김도윤 comment | "CS팀 통계 보니 실제 정산 후 취소로 이어지는 건은 **월 10건 미만**일 것으로 예상됩니다" — "per the CS team's statistics, I expect post-settlement cancellations to be **under 10 a month**" | **Forecast.** Made 14 days before the feature shipped; nothing had been measured because nothing had run |
| 2023-05-02 | v0.3 §5 비고 | "예상 처리량 월 10건 미만" — "expected throughput under 10 a month" | Forecast, correctly labelled 예상 (expected) |
| 2024-02-19 | v1.1 | *(clause deleted)* | Absent from policy from this date onward |
| 2024-03-11 | archive review draft, 보류 사유 | "처리량이 많지 않아 당장의 필요성이 낮다고 판단" — "judged low priority as throughput is not large" | **Asserted as fact**, in the same document that admits the queue size is unknown |
| 2026-06-18 | kickoff minutes §1.1 | "월 10건 내외로 보고 있어 수기로 감당 가능한 수준이라는 것이 2023년 판단이었다" — "around 10 a month ... was the 2023 judgement" | Attributed to 2023, correctly. Immediately qualified by 이수민: "요즘은 좀 더 되는 것 같기는 한데 정확히 세어보진 않았습니다" |
| 2026-08-18 | automation plan §2 | "현행 처리량은 정산팀 확인 결과 **월 10건 내외**로 파악된다" — "**current** throughput is understood, per confirmation with the settlement team, to be around 10 a month" | **Present tense, sourced to a confirmation.** The hedge and the attribution to 2023 are both gone |
| 2026-09-01 | queue export | 2026 monthly rows: 161, 163, 148, 162 | **Measurement.** 14–16× the planning figure |

Two things are worth separating. The forecast was reasonable in 2023 — it was made by the team lead who would perform the work, based on CS statistics, and labelled as an expectation. The failure is not the estimate. It is that the estimate lost its hedge with each restatement while never acquiring a measurement, and that the one document that had formal standing (v1.1) deleted the figure without replacing it with a measured one, leaving every later document free to cite the 2023 draft instead. The 2026 automation project is currently sized against it → sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md, sources/raw/exports/cancel_recon_queue_monthly_20260901.csv

Note also what the export's own caveats do and do not permit. The README warns that only `STATUS='PENDING'` rows were returned and that manual corrections handled outside the system are invisible, so the queue count is not the same population as "cases handled per month". But the direction is unambiguous: 4,127 rows accumulated where fewer than 10 a month were forecast would have implied roughly 490 over the same 41 months → sources/raw/exports/README.md

### The quarterly verification clause

v1.1 §3, row three, in full:

> | 재무기획팀 | 분기 결산 시 미정정 잔액 확인 |

"Financial Planning team | verify the uncorrected balance at the quarterly close."

This clause has no counterpart in v0.3 and is the only obligation in either version assigned outside 정산팀 and 주문팀. Three observations follow from the material.

1. **It was the clause that eventually worked.** The 2026-08-24 mail from 문지영 (재무기획팀) is exactly this duty being discharged — a quarterly-close question about whether to book the balance as a liability: "분기 결산에 부채로 인식해야 하는지 판단이 필요합니다" — "a judgement is needed on whether to recognise this as a liability at the quarterly close." The problem surfaced through the procedure's own control, not through the settlement team or the code → sources/raw/mail/RE_정산_미정정_금액_문의.eml
2. **It took two and a half years to discharge.** v1.1 was issued 2024-02-19; the first evidence of the check is 2026-08-24, i.e. at least nine quarterly closes later. No earlier instance appears in the mail, minutes, sprint records or Slack. Absence of evidence is not proof, and this is recorded as a known unknown above.
3. **The clause assigns verification without granting a means.** 재무기획팀 has no read access route defined in the procedure, and the handover records that no screen exists for the queue at all — "대기열 건수를 조회하는 화면이 없음. DB 직접 조회만 가능." ("there is no screen to query the queue count; only direct DB access"). In practice the check required a favour: the 2026-09-01 export was run by 데이터팀 on request, and the ticket to build the screen, SF-5120, was still `In Progress` at the end of sprint 2026-S17 → sources/context/handover/2025-03_정산팀_인수인계.md, sources/raw/exports/README.md, sources/context/sprints/tickets_2026-S17.csv

### The retention clause

v1.1 §5, first bullet: "정정 이력은 정산 어드민에 기록하며 5년간 보존한다." — "correction history is recorded in the settlement admin and retained for five years." §6 repeats 5년 as the 보존 column for both ST-F-001 and ST-F-002.

The clause presumes three things that the available material does not support:

- **A system of record.** The 정산 어드민 (settlement admin) exists in no repository among the five. The handover lists it as an access target — "정산 어드민 | 정정 등록 | 팀장 승인 후" ("settlement admin | correction registration | after team-lead approval") — so it is something, but not something the reef can inspect → sources/context/handover/2025-03_정산팀_인수인계.md
- **Records to retain.** The handover's §6 미인계 사항 (items not handed over) says the opposite: "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중." — "there is no official form for recording correction history; each person manages it in their own Excel." Written thirteen months after v1.1 mandated the two forms → sources/context/handover/2025-03_정산팀_인수인계.md
- **A start event for the five-year clock.** The clause does not say whether the period runs from the correction, the original settlement or the cancellation. For a queue that has never been drained the distinction is decisive, and it is recorded as a known unknown.

### The five-year retention window, measured

Reference date **2026-09-19**. Oldest `PENDING` month in the queue: **2023-04** (48 rows, 2,196,480 KRW).

| Start event assumed | Oldest rows' expiry | Expired as of 2026-09-19? |
|---|---|---|
| Cancellation / queue insertion (`RECV_DTM`, 2023-04) | 2028-04 | **No** — 3 years 5 months elapsed of 5 |
| Original settlement (same month, by construction: a row exists only because the order was already settled) | 2028-04 | **No** |
| The correction itself | never starts | **No — and it never will**, because no correction was performed |

**No queue row has yet passed the five-year window.** The earliest possible expiry under any reading is 2028-04, nineteen months away. This corrects a claim in [[RISK-SETTLEMENT-RECON-BACKLOG]], which states that "the oldest rows are now beyond the five-year retention that procedure v1.1 §5 sets for correction history"; on the 2023-04 / 2026-09 arithmetic, they are not.

The substantiation problem for a 2023 case is real, but it is not a retention problem. It is an absence problem, and it is worse than expiry would be:

- **The record the policy protects does not exist.** v1.1 §5 retains 정정 이력 — correction *history*. A correction history row is created by performing a correction. None of the 4,127 rows has been corrected through the system, so for each of them the five-year clock has no start event and there is nothing in the 정산 어드민 to produce.
- **What does exist is a queue row, which no policy covers.** `CANCEL_RECON_QUEUE` gives `SEQ`, `ORD_NO`, `SAYU_CD`, `RECV_DTM`, `STATUS`, `PROCESSED_DTM` — enough to show that a case was registered on a date and never processed, but nothing about amount, partner or decision. Neither version of the procedure names the queue table as a record or assigns it a retention period, so its survival to 2028 and beyond is a matter of DBA practice, not policy → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql
- **One of its fields is unreliable.** `OrderEventRelayJob` extracts `SAYU_CD` by `substring(i + 10, i + 12)` on the raw payload string and returns the literal `"00"` when the fragment is absent — a value outside the documented 01–04 cancellation-reason vocabulary. A 2023 row carrying `00` cannot state why the order was cancelled, which is precisely what a partner dispute turns on → settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, sources/context/business-rules.md
- **The amount is not in the queue at all.** The 2026-09-01 export sums `EXPECTED_AMT`, a column no migration in `settlement-batch` defines. To put a figure on a 2023 case one must reconstruct it from `SETTLEMENT_DTL` — which has its own retention question nobody has asked → sources/raw/exports/README.md

So the honest statement to a partner about a 2023-04 case, as of 2026-09, is: the system can show that a cancellation was registered for deduction on a date in April 2023 and that the deduction was never applied; it cannot produce a correction record, because the policy's record only comes into existence when the correction does. **The date that matters for planning is 2028-04** — after which, if a correction is finally performed, the five-year retention on its history would be starting from a transaction more than five years old, and whoever performs it should decide first which reading of §5 they are working under.

### Which clauses the running code implements

Numbering follows each document's own sections. "Implemented" means code exists *and* runs.

| Clause | v0.3 | v1.1 | Implementing code | Runs? |
|---|---|---|---|---|
| Queue registration on cancellation | §3.1 "취소 접수 시 `CANCEL_RECON_QUEUE` 에 정정 대상으로 적재된다" | §4.1 "취소 접수 → 정정 대기 등록 (시스템)" | `OrderEventRelayJob.executeInternal` — polls `ORDER_EVENT_OUTBOX`, inserts `STATUS='PENDING'` when `COUNT(1) FROM SETTLEMENT_DTL > 0` | **Yes** — `orderEventRelayTrigger`, every 10 minutes, registered in `QuartzConfig` |
| Review of the pending queue | §3.2 "정산팀이 대기열을 확인한다" (no cadence) | §4.2 "정산팀 확인 (월 1회 이상)"; §5 "월 1회 이상 대기 건 현황을 확인한다" | none — no screen, no report, no query job | **No.** Handover: "전체 대기열을 주기적으로 확인하는 절차는 없음. 문의가 오면 그 건만 본다." — "there is no procedure for periodically reviewing the whole queue; when an inquiry comes in, only that case is looked at." SF-5120 (a screen) still `In Progress` |
| Deduction from the following month | §3.3 "차월 정산 시 해당 금액을 차감한다" | §4.3 "차월 정산 반영" | `CancelReconciler.reconcileCancellations()` — drains PENDING into `SETTLEMENT_ADJUSTMENT` with `ADJ_TYPE='CANCEL_CLAWBACK'` | **No.** Not registered in `QuartzConfig`, called by nothing. Its own TODO: "TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤". And `SETTLEMENT_ADJUSTMENT` has no DDL in any repository, and no code applies an adjustment to a payout |
| Partner notification | — | §4.4 "파트너 통지 및 이력 기록" | `SettlementReportWriter.write(runId)` — the only partner-facing output; called by no step, and its body only logs | **No.** Its own TODO: "TODO(도윤) 2024-11-02: 취소 정정분은 이 리포트에 포함되지 않는다. 별도 확인 필요." — "cancel corrections are not included in this report; needs separate checking" |
| Correction history record | — | §4.4, §5 "정정 이력은 정산 어드민에 기록하며 5년간 보존한다" | none in any repository | **No.** Handover §6: "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중." |
| Forms ST-F-001 / ST-F-002 | — | §6 | n/a — paper/admin controls, not code | **No evidence.** Neither form appears in `sources/`; the handover reports no official format in use |
| Quarterly balance verification | — | §3, 재무기획팀 row | n/a — a finance control, not code | **Once, evidenced** — 2026-08-24, and only by asking 데이터팀 to run a manual extraction |
| Cancel event publication | §4 "이벤트 발행: 주문팀" | §3, 주문팀 row "취소 이벤트 발행" | `order-service` writes `order.cancelled` to `ORDER_EVENT_OUTBOX` | **Yes** — the one obligation of the other team, and it is met |
| Automation reconsideration trigger | §5 "건수가 늘어나면 자동화를 검토한다" | *(deleted)* | n/a | **Deleted before it could bind.** The 2024-03-11 review draft was shelved; the 2026 plan restarted the question from scratch |

**One clause of the correction procedure runs.** Queue registration — the step performed by a machine, written by the order team's counterpart in the settlement repo in 2023, and the only one that requires no human to remember anything. Every clause that depends on a person or on a second scheduled job is unperformed.

The asymmetry has a shape worth naming: the implemented clause is the one that *creates* obligations, and none of the clauses that *discharge* them run. That is the mechanism by which the backlog is not merely unprocessed but self-accelerating, and it was already visible in the Slack channel eleven months before v1.1 was written — 이수민, 2023-06-02: "정정 배치 도는 건가요? 이번 달 정산에 차감 반영된 게 안 보여서요" ("is the correction batch running? I don't see any deduction reflected in this month's settlement"); 김도윤, same day: "아직입니다. 일단 문의 들어온 건만 수기로 보고 있습니다" ("not yet. For now I'm only handling by hand the cases that come in as inquiries") → sources/raw/slack/settlement-dev_2023-04_2026-08.json

v1.1 was drafted and approved with that exchange already in the channel, and it added a monthly review cadence and a retention period to a process whose central step was known not to be running.

### What happened around the revision

The archived review draft is usually assumed to sit between the two versions. It does not — it is dated **2024-03-11, twenty-one days after v1.1 was issued** on 2024-02-19. Read in that order it is the immediate sequel to the revision, and it is the most revealing document of the three.

- Its 배경 states the motivation in one sentence: "정정 업무가 수기로 처리되고 있어 담당자 부재 시 누락 위험이 있다." — "correction work is handled manually, so there is a risk of omission when the responsible person is absent." Twelve months later that risk materialised precisely as described: 정민호 left on 2025-03-31 and the handover records item 3 in full as 미인계 (not handed over) → sources/context/_archive/2024_정산정정_자동화_검토안_초안.md, sources/context/handover/2025-03_정산팀_인수인계.md
- Its 검토 내용 proposes exactly the component that already existed: "대기열을 읽어 차감을 생성하는 배치를 추가한다" — "add a batch that reads the queue and generates deductions." `CancelReconciler` had been written eleven months earlier; what was missing was four lines in `QuartzConfig`, and SF-4512 to add them had been open since 2023-04-24 → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java, sources/context/sprints/tickets_2026-S17.csv
- Its 보류 사유 and its 남은 확인 사항 contradict each other across four lines: shelved because throughput is low, while recording that the accumulated queue size has never been confirmed. The same self-contradiction — asserting the volume that justifies the decision while admitting it is unmeasured — is the one the 2026 plan repeats → sources/context/_archive/2024_정산정정_자동화_검토안_초안.md
- The Slack channel brackets the revision on both sides with the same exchange. 이수민, 2024-02-13 (six days before v1.1): "작년에 얘기 나왔던 정정 자동화는 어떻게 됐을까요" — "what happened to the correction automation we discussed last year?" 김도윤: "우선순위에서 밀렸습니다. 건수가 많지 않아서 당장은 수기로 커버 가능합니다" — "it slipped in priority. The case count isn't large, so manual handling covers it for now." By 2024-02 the queue had accumulated roughly 900 rows on the export's monthly figures → sources/raw/slack/settlement-dev_2023-04_2026-08.json, sources/raw/exports/cancel_recon_queue_monthly_20260901.csv

### Which version to cite

**v1.1 is authoritative.** v0.3 says so itself and is retained only for history. But three pieces of content exist *only* in v0.3 and are still being relied on in 2026:

| Content | Only in | Still cited in |
|---|---|---|
| 예상 처리량 월 10건 미만 | v0.3 §5 | 2026-06-18 minutes, 2026-08-18 automation plan (as 월 10건 내외, present tense) |
| 별도 시스템 없이 수기로 처리한다 | v0.3 §5 | the 2026-08-24 mail thread, where 김도윤 states "건수가 많지 않아 수기로 보고 있습니다" |
| 자동화 검토 트리거 (건수가 늘어나면 자동화를 검토한다) | v0.3 §5 | nothing — deleted and never replaced |

Anyone quoting "10 cases a month" is quoting a superseded draft's restatement of a 2023 forecast, at three removes from any measurement. The correct current statement of volume is the 2026-09-01 export, with its caveats.

### Agent guidance

- **Cite v1.1 for obligations, v0.3 only for history.** If a claim about expected volume is needed, cite the export, not either policy.
- **Do not infer implementation from either document.** Eleven of twelve obligations across the two versions have no running implementation. The implementation column above, not the procedure, answers "does this happen".
- **The five-year retention has not yet expired for any queue row** (earliest 2028-04 as of 2026-09-19). If asked whether a 2023 case is still substantiable, the blocker is that no correction record was ever created, not that one was destroyed.
- **Treat the 재무기획팀 quarterly clause as the live control.** It is the only clause in either version that has demonstrably surfaced the problem, and the 2026 Q3 close question it raised has no recorded answer.

## Related

- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — the table the procedure's step 1 writes to, field by field
- [[PROC-SETTLEMENT-CORRECTION]] — the mechanism and the evidence chain this diff does not restate
- [[RISK-SELLFLOW-DOC-DRIFT]] — the wider pattern of authoritative documents describing behaviour the code lacks
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measured exposure, and the retention claim this artifact corrects
- [[SCH-SETTLEMENT-BATCH]] — `CANCEL_RECON_QUEUE` and the missing `SETTLEMENT_ADJUSTMENT`
- [[SYS-ORDER]] — the publisher of the cancel event the procedure's step 1 depends on
- [[SYS-SETTLEMENT]] — the owning system
