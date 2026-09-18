---
id: "DEC-SELLFLOW-2026-AUTOMATION-SIZING"
type: "decision"
title: "Sizing the 2026 Correction-Automation Programme on a 2023 Forecast"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Written 2026-09-19 against a plan that was 검토중 (under review) at its 2026-09-01 export. The contradiction recorded here is arithmetic on the 2026-09-01 queue export and holds regardless of the plan's status, but the plan itself may since have been revised — check the Notion source before citing its figures as current."
freshness_triggers:
  - "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
  - "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
  - "sellflow-docs:context/sprints/tickets_2026-S17.csv"
  - "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
known_unknowns:
  - "Whether the 2023 forecast was ever right. It was a prediction made in April 2023 from CS statistics that are not present in the reef; the first month of measured data, 2023-04, already shows 48 rows. Whether the CS statistics were misread, covered a different population, or were simply superseded within the month cannot be determined."
  - "Who told the plan's author that the current volume is 월 10건 내외. The plan attributes it to 정산팀 확인 결과 ('as confirmed by 정산팀') but names no person, date or method, and the kickoff minute has 정산팀 presenting the figure as a 2023 judgement rather than a current measurement."
  - "Whether the action item 현행 월 처리 건수 실측 ('measure the actual monthly volume') was ever done. It was logged with 기한 미정 (no due date) on 2026-06-18; no measurement result appears in the sprint records, and the 2026-09-01 export was produced for 재무기획팀, not for this action item."
  - "Whether the plan's reviewers — 김도윤, 박성민, 배준호 — challenged the figure. The plan's status is 검토중 and no review comments are present in the export."
  - "What the plan's 80% time-saving target was computed against. No baseline handling time per case appears in the plan, the minutes or the sprint records."
  - "Whether the AX programme's selection of this process as a 1차 대상 depended on the volume figure. The plan lists 처리 건수가 꾸준함 ('the case count is steady') as a selection criterion but does not say what count was used."
tags:
  - "sellflow"
  - "settlement"
  - "automation"
  - "planning"
  - "estimation"
  - "adr-reconstructed"
aliases:
  - "월 10건 미만"
  - "월 10건 내외"
  - "정산 정정 자동화 사이징"
relates_to:
  - type: "depends_on"
    target: "[[DEC-SETTLEMENT-CANCEL-CLAWBACK]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "feeds"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "feeds"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/_archive/2024_정산정정_자동화_검토안_초안.md"
    notes: "The 2024 draft shelved on low volume, admitting the volume was unknown."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
    notes: "The kickoff where the 2023 figure is restated and doubted in the same breath."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "정산팀 4→3 in 2024-07; AX roadmap 2026-06-15."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
    notes: "PLAN-2026-014, the plan under review."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
    notes: "§5 — the forecast written into procedure."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/2026-S17_log.md"
    notes: "Sprint outcome for the plan's prerequisite tickets."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The 2023-04-07 origin of the figure."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
    notes: "41 monthly rows — the measurement the plan lacks."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
    notes: "2024-02-13 and 2026-01-08 — volume asserted, then doubted."
notes: "The decision under record here is a sizing assumption, not a build decision. It is filed as a decision because the programme's scope, priority and schedule all rest on it."
---

# Sizing the 2026 Correction-Automation Programme on a 2023 Forecast

## Context

In 2026 셀플로우 began an AX programme to automate repetitive back-office work with AI agents. The settlement team's manual correction work — 정산 오류 수기 정정, which the org chart lists as one of 정산팀's two named processes — was chosen as the first target, and PLAN-2026-014 was written to scope it → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md, sellflow-docs:context/org-chart.md

Any automation plan is sized on a volume. This one is sized on a number first spoken in April 2023, as a forecast, by the person who was proposing to handle the work by hand — and never measured since. Three months before the plan was written, the queue was exported and the number was available. The decision under record here is the choice to size the programme on the 2023 figure, and to restate it as a present-tense fact.

The mechanism that generates the volume is [[DEC-SETTLEMENT-CANCEL-CLAWBACK]]; the exposure it has produced is [[RISK-SETTLEMENT-RECON-BACKLOG]].

## Decision

**The 2026 programme was scoped, prioritised and scheduled on the assumption that post-settlement correction runs at roughly ten cases a month.** The figure is stated as current fact in the plan, presented as the team's own understanding in the kickoff minute, and traceable in an unbroken line back to a single 2023 comment. No measurement was performed at any point in that line.

The chain, dated:

| Date | Document | The figure, verbatim | Standing |
|---|---|---|---|
| 2023-04-07 | SF-2287 comment, 김도윤 | "실제 정산 후 취소로 이어지는 건은 **월 10건 미만**일 것으로 예상됩니다" | Explicit forecast — 예상됩니다, "I expect" |
| 2023-05-02 | 정산 정정 업무절차 v0.3 §5 | "예상 처리량 월 10건 미만. 별도 시스템 없이 수기로 처리한다." | Forecast written into procedure — 예상 처리량, "expected volume" |
| 2024-03-11 | 2024 자동화 검토안 초안 | "처리량이 많지 않아 당장의 필요성이 낮다고 판단" | Used as grounds to shelve automation |
| 2026-06-18 | 자동화 킥오프 minute | "월 10건 내외로 보고 있어 수기로 감당 가능한 수준이라는 것이 2023년 판단이었다" | Correctly labelled a 2023 judgement — then not revisited |
| 2026-08-18 | PLAN-2026-014 §2 | "현행 처리량은 정산팀 확인 결과 **월 10건 내외**로 파악된다" | Present tense — 현행, "current", 파악된다, "is understood to be" |

→ sellflow-docs:context/tickets/SF-2287.md, sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md, sellflow-docs:context/_archive/2024_정산정정_자동화_검토안_초안.md, sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md, sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md

The transformation across those five rows is the finding. A hedged prediction about a future workload becomes an expected volume in a procedure, then a reason not to build anything, then a remembered judgement, then a measured-sounding statement of the present. Nothing was added to it at any step except confidence.

## Key Facts

- The figure originates as a forecast, not an observation: 김도윤, 2023-04-07, "CS팀 통계 보니 실제 정산 후 취소로 이어지는 건은 월 10건 미만일 것으로 예상됩니다" — "looking at CS's statistics I expect cases that actually lead to a post-settlement cancel to be fewer than ten a month" → sellflow-docs:context/tickets/SF-2287.md
- It was made by the lead of the team that would absorb the work, in the same comment in which he undertook to do it by hand — so it is also the number that justified not building anything → sellflow-docs:context/tickets/SF-2287.md
- Procedure v0.3 §5 fixed it in writing four weeks later, together with the automation gate it supports: 예상 처리량 월 10건 미만. 별도 시스템 없이 수기로 처리한다. / 건수가 늘어나면 자동화를 검토한다. (기준 TBD) — "expected volume fewer than ten a month; handled manually with no separate system. If the count grows, automation will be reviewed. (criterion TBD)" → sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md
- That gate has never had a threshold. 기준 TBD in 2023 is still 기준 TBD; v1.1, the approved 2024 revision, drops the automation clause entirely rather than setting a number → sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md, sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md
- The 2024 draft used the unmeasured volume as its reason to stop: 처리량이 많지 않아 당장의 필요성이 낮다고 판단 — "judged that the immediate need is low because the volume is not large" → sellflow-docs:context/_archive/2024_정산정정_자동화_검토안_초안.md
- The same 2024 draft admits, in its last line, that nobody knew the volume: 대기열에 쌓인 건이 실제로 얼마나 되는지 확인되지 않음 — "it has not been confirmed how many cases have actually accumulated in the queue" → sellflow-docs:context/_archive/2024_정산정정_자동화_검토안_초안.md
- The 2026 plan restates the figure in the present tense and marks it as a finding: 📌 현행 처리량은 정산팀 확인 결과 월 10건 내외로 파악된다 — "the current volume is understood, per 정산팀's confirmation, to be around ten a month" → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§2)
- Its cited basis is the 2023 agreement itself: 근거: SF-2287 (2023-04) 협의 내용, 조직도상 정산팀 담당 프로세스 — "basis: the SF-2287 (2023-04) agreement, and the process assigned to 정산팀 in the org chart" → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§2)
- The practitioner doubted it out loud at the kickoff, and the doubt was not resolved: 이수민, "요즘은 좀 더 되는 것 같기는 한데 정확히 세어보진 않았습니다" — "it feels like a bit more these days, but I haven't counted exactly" → sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md (§1.1)
- The kickoff logged an action item to measure it — 현행 월 처리 건수 실측 — 정산팀 (기한 미정), "measure the actual monthly volume — 정산팀 (no due date)" — which remains unticked in the minute and produces no result in any later document → sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md
- The measured figure was available before the plan was written. The queue was exported on 2026-09-01 at the request of 재무기획팀, and the 2026-01-08 slack thread shows the data team had pulled it eight months earlier: 한지우, "뽑아드렸습니다. 생각보다 많습니다" — "I've pulled it for you. It's more than expected" → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv, sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json
- The export shows 4,127 PENDING rows across 41 months: a mean of **100.7 per month** over the whole period, and **145.2 per month** over the trailing twelve (2025-09 through 2026-08) → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv
- Even the very first month in the series, 2023-04, records 48 rows — nearly five times the forecast, in the month the forecast was made → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv
- The trend is monotonic in the annual aggregate: 485 rows across nine months of 2023, 987 in 2024, 1,450 in 2025, 1,205 across eight months of 2026 → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv
- The volume was asserted again in 2024 as a reason to defer: 김도윤, 2024-02-13, "우선순위에서 밀렸습니다. 건수가 많지 않아서 당장은 수기로 커버 가능합니다" — "it got pushed down the priority list. The count isn't large, so manual handling can cover it for now." The 2024 mean was 82.3 rows a month → sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json, sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv
- The plan's own prerequisites were not met in the sprint that followed it: SF-5121 정정 판정 기준 정의 was 할 일 / 착수 못함 ("To Do / could not be started") and SF-5120, the screen that would let 정산팀 see the volume at all, carried over → sellflow-docs:context/sprints/2026-S17_log.md, sellflow-docs:context/sprints/tickets_2026-S17.csv

## Rationale

For the original 2023 forecast, the stated basis is CS's statistics — "CS팀 통계 보니" ("looking at CS's statistics"). Those statistics are not in the reef, so whether the inference was sound cannot be assessed here; that is recorded in `known_unknowns`. What can be said is that the same ticket cites a different CS number, 214 inquiries in March 2023, for post-settlement cancel *requests*, and that the forecast concerns the subset of those that complete. The relationship between the two figures is not worked out anywhere in the thread.

For the 2026 plan's reuse of the figure, the document gives a source line — 근거: SF-2287 (2023-04) 협의 내용 — but no reasoning about currency. It does not say the figure was re-checked, and it does not say it was not; the phrase 정산팀 확인 결과 ("per 정산팀's confirmation") implies a check whose author, date and method are absent. **No motive for restating a three-year-old forecast as a current measurement is determinable from any document read for this artifact**, and none is invented here. The kickoff minute, five weeks earlier, shows the team itself framing the number as 2023년 판단 ("the 2023 judgement") and the practitioner saying she has not counted — which is the most that can be said about what the plan's author had been told.

One contextual factor the plan does record, without connecting it to the volume figure: 정산팀 인원 조정(2024-07, 4명→3명) 이후 업무 부담이 증가했다는 의견이 있어 자동화 우선순위를 상향 조정하였다 — "there were views that the workload increased after the 정산팀 headcount adjustment (2024-07, 4→3), so the automation priority was raised." The org chart confirms the headcount change. So the plan's priority was raised on a perceived workload increase while its scope was held at a volume that implies no increase at all → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§6), sellflow-docs:context/org-chart.md

## Consequences

**The measured contradiction.** The assumption and the data, side by side:

| | Assumed | Measured | Ratio |
|---|---|---|---|
| Monthly volume (41-month mean) | ~10 | 100.7 | **10.1x** |
| Monthly volume (trailing 12 months) | ~10 | 145.2 | **14.5x** |
| Total cases in scope | ~360 over 3 years, none backlogged | 4,127 PENDING, all backlogged | — |
| Amount at stake | not stated | 188,851,520 KRW | — |

→ sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv, sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md

**What an order-of-magnitude sizing error implies for the plan, from the plan's own text:**

1. **Selection.** 처리 건수가 꾸준함 ("the case count is steady") is one of three stated reasons this process was picked as the 1차 대상. The count is not steady; it has roughly tripled since the baseline it is described against. Whether the process still ranks first against other AX candidates is a question the plan cannot answer on the figure it used → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§1)
2. **Target.** 목표: 수기 처리 시간 80% 절감 ("target: 80% reduction in manual handling time") is a percentage of a baseline that is 10x understated. As a business case the saving is an order of magnitude larger than presented; as a design constraint, a system built to clear ten cases a month has a different shape from one clearing 145 → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§3)
3. **Exception handling.** 담당자는 예외 건만 검토 ("the operator reviews only the exceptions") is the load-bearing design assumption. At ten cases a month an exception rate of any size is absorbable by three people; at 145 it is a staffing question. The exception criteria do not exist yet — SF-5121 was not started → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§3), sellflow-docs:context/sprints/2026-S17_log.md
4. **Scope of the As-Is.** The plan's As-Is diagram ends 처리 완료 ("handling complete"), describing a process that runs to completion. Per [[DEC-SETTLEMENT-CANCEL-CLAWBACK]], the automatic drain has never run and the manual drain covers only partner-reported cases. The plan therefore proposes to automate a process whose current-state description is not the current state → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§2)
5. **Backlog.** 4,127 already-pending rows do not appear in the plan in any form. An agent that handles the monthly inflow leaves the accumulated 188.8M KRW untouched, and nothing in the plan's five-stage schedule (현행 분석 → 설계 → 개발 → 시범 운영, 2026-09 to 2027-01) allocates time to it → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§4, §5)
6. **Schedule.** 외부 컨설팅 착수 (AX 파트너) is listed as a 선결 과제 (prerequisite). External work scoped on the wrong order of magnitude is the most expensive place for this error to land → sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md (§4)

**The queue was never measured before the plan was written, and the record says so plainly.** This is the sharpest point, because it is admitted rather than inferred. The 2024 draft closes with its 남은 확인 사항 (remaining items to confirm): 대기열에 쌓인 건이 실제로 얼마나 되는지 확인되지 않음 — "it has not been confirmed how many cases have actually accumulated in the queue." That draft was nonetheless shelved on the grounds that 처리량이 많지 않아 — "the volume is not large." A document that states in one section that the volume is unknown, and in another that it is small, is the clearest available evidence that the figure was never checked → sellflow-docs:context/_archive/2024_정산정정_자동화_검토안_초안.md

Two years later the same absence persists in a different form: the 2026-06-18 kickoff opens an action item to measure the volume with no due date, and the 2026 plan is written two months afterwards without it. The measurement that finally happened, on 2026-09-01, was commissioned by 재무기획팀 for a quarter-close liability judgement — not by either automation effort → sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md, sellflow-docs:raw/exports/README.md

**A structural cause, visible in the sprint record.** 정산팀 has no way to observe its own queue: SF-5120, the view screen, is still unfinished, and the S17 retrospective records the workaround — 조회 화면 없이 대기열 건수를 매번 데이터팀에 요청하는 상태가 반복되고 있다 ("we keep having to ask the data team for the queue count because there is no view screen"). A team that must file a request with another team to learn its own workload will tend to keep quoting the last number it remembers. That is what the five-row chain above looks like from the inside → sellflow-docs:context/sprints/2026-S17_log.md

**One correction that costs nothing.** The plan's §2 figure can be replaced with the export's, and its As-Is diagram corrected to show that the queue has no drain, before the design stage begins. Both facts are already in the reef.

## Related

- [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] — the decision the figure was used to justify, and the mechanism that generates the volume
- [[PROC-SETTLEMENT-CORRECTION]] — the actual current-state process the plan describes differently
- [[RISK-SELLFLOW-DOC-DRIFT]] — the wider pattern of documents outliving the facts they assert
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — the measurement, with its extraction caveats
- [[SYS-SETTLEMENT]] — the team and system in scope
