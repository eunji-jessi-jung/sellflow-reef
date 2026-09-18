---
id: "PROC-SELLFLOW-OWNERSHIP"
type: "process"
title: "Service Ownership and Its Four Disagreeing Records"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "The service registry has not been reviewed since 2026-03-02 and says so in a comment. The org chart was rendered from the .xlsx on 2026-09-18. CODEOWNERS exists in one repo of five. Any of the three can move without the others noticing, which is the subject of this artifact — re-read all three together, never one alone."
freshness_triggers:
  - "order-service:.github/CODEOWNERS"
  - "sellflow-docs:context/org-chart.md"
  - "sellflow-docs:context/registry/services.yaml"
  - "sellflow-docs:context/sprints/tickets_2026-S17.csv"
  - "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
known_unknowns:
  - "Whether delivery-bff's transfer to 물류팀 ever completed. The registry says the discussion opened 2025-11 and was not settled; the org chart, the README and package.json all assert 물류팀 flatly, with no date. No document records a decision, and no ticket about delivery-bff exists in sources/context/tickets/."
  - "Who actually works on delivery-bff. No person from 물류팀 appears anywhere outside the org chart — not in a ticket, a sprint record, a Slack message, a meeting, or a source comment. The repo has no CODEOWNERS and no commit history was available to read."
  - "What @sellflow/dba is, organisationally. It is a CODEOWNERS reviewer on order-service migrations but no DBA team appears in the org chart; DB access requests go to 인프라팀 and re-extraction requests go to a DBA per the export README. Whether @sellflow/dba maps to 플랫폼팀, 인프라팀 or something else is not established."
  - "Whether 박성민 ever gave the confirmation that CancelReconciler's TODO and SF-4512 wait on. Nothing in the Slack export, the tickets, the minutes or the runbook records it arriving or being withdrawn; the Slack export is keyword-filtered, so its silence is weak evidence."
  - "Who owns eta-predictor's repository. The org chart assigns the system to 데이터팀, but it is absent from the registry, absent from the five repos, and has no entry anywhere else in sources/."
  - "Whether SF-4512 being unassigned is a decision or an oversight. No document discusses it after its creation date; the 2026-S17 retrospective does not mention it, and the 2026 automation plan does not reference it."
  - "Which team is accountable for acting on settlement-anomaly output. The registry states in as many words that this is undefined and marks it TODO; no later document resolves it."
  - "Whether the org chart's 변경이력 sheet is meant to record individual departures. 정민호 left 2025-03-31 and no row records it, but the sheet contains only team-level events, so it may simply be out of scope rather than stale."
tags:
  - ownership
  - governance
  - org-chart
  - registry
  - codeowners
  - accountability
aliases:
  - "서비스 소유팀"
  - "담당 팀"
  - "owner_team"
relates_to:
  - type: "integrates_with"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "depends_on"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SYS-DELIVERY]]"
  - type: "refines"
    target: "[[SYS-INVENTORY]]"
  - type: "refines"
    target: "[[SYS-ORDER]]"
  - type: "refines"
    target: "[[SYS-SETTLEMENT]]"
  - type: "refines"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "담당: 커머스본부 물류팀 — contradicts the registry's TODO"
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "description field also names 물류팀"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:README.md"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/CODEOWNERS"
    notes: "The only CODEOWNERS file in the five repos"
  - category: "implementation"
    type: "github"
    ref: "order-service:README.md"
  - category: "implementation"
    type: "github"
    ref: "order-service:TASK.md"
    notes: "박성민's personal note; explicitly not an official document"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
    notes: "The recorded disagreement over who owns the post-event segment"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "Rendered from org-chart.xlsx on 2026-09-18; two sheets, 조직도 and 변경이력"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "§2 role table — the only approved allocation of responsibilities"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "last_reviewed 2026-03-02; three TODO markers"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/tickets_2026-S17.csv"
    notes: "SF-4512 row — unassigned since 2023-04-24"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The comment thread in which the compensation was offered and accepted"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
    notes: "Keyword-filtered on 정정/차감, not a full archive — absence is weak evidence"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:README.md"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:README.md"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
    notes: "The dated, signed TODO that defers the schedule registration"
notes: "Operational archetype. This artifact reconciles records of ownership; it does not restate the mechanics of the cancel-reconciliation gap, which live in PROC-SETTLEMENT-CORRECTION, or its financial size, which lives in RISK-SETTLEMENT-RECON-BACKLOG."
---

# Service Ownership and Its Four Disagreeing Records

## Purpose

SellFlow keeps four independent records of who owns what: a service registry, an org chart spreadsheet, GitHub CODEOWNERS, and the accumulated fingerprints of the people who actually touch the code and the tickets. Nothing joins them. This artifact reads all four against each other for the five repositories, states a verdict per service, and then follows the one question the four records answer differently — who owns `CANCEL_RECON_QUEUE`.

The short version: for four of five services the records agree or the disagreement is cosmetic. For `delivery-bff` the registry is the lone dissenter and no human evidence exists on either side. And for the reconciliation gap the records do not disagree about a team so much as they each assign a *different part* of the job to a different team, with the connecting part assigned to nobody — which is why it has sat untouched since 2023.

## Key Facts

- The registry declares itself authoritative for ownership: "이 파일이 서비스·소유팀·저장소의 단일 기준이다. 신규 서비스는 여기에 먼저 등록한다." — "this file is the single standard for services, owning teams and repositories; new services are registered here first" → sellflow-docs:context/registry/services.yaml
- It deliberately holds teams only, deferring people to the org chart: "담당자 정보는 org-chart 를 따르며 여기에는 팀만 적는다." — "assignee information follows the org chart; only teams are recorded here" → sellflow-docs:context/registry/services.yaml
- It has not been reviewed in over six months and annotates its own staleness: "# last_reviewed: 2026-03-02   # 이후 갱신 없음" — "no updates since" → sellflow-docs:context/registry/services.yaml
- It carries three unresolved TODO markers: `delivery-bff` `owner_team: TODO`, `CANCEL_RECON_QUEUE` `consumer: TODO   # 확인 필요` ("needs confirming"), and on `settlement-anomaly` "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" — "performs detection only; the party responsible for action is not defined" → sellflow-docs:context/registry/services.yaml
- Exactly one of the five repositories has a CODEOWNERS file. `find` across all five returned only `order-service/.github/CODEOWNERS`; `settlement-batch`, `inventory-api`, `delivery-bff` and `settlement-anomaly` have none, so for four services GitHub enforces no review ownership at all → order-service:.github/CODEOWNERS
- That file names a GitHub team the org chart does not contain: `/src/main/resources/db/ @sellflow/order-team @sellflow/dba`. No DBA team appears among the sixteen teams in the 조직도 sheet → order-service:.github/CODEOWNERS, sellflow-docs:context/org-chart.md
- The org chart lists 팀장 (team leads) and headcount but no individual contributors, so the three people who actually perform settlement correction work — 정민호, 이수민, 한지우 — cannot be found in either of the two authoritative records → sellflow-docs:context/org-chart.md, sellflow-docs:context/sprints/tickets_2026-S17.csv
- The org chart assigns 데이터팀 two systems, "eta-predictor, settlement-anomaly", but `eta-predictor` has no registry entry and no repository, so the record that claims to be the single standard for services is missing one the org chart operates → sellflow-docs:context/org-chart.md, sellflow-docs:context/registry/services.yaml
- `settlement-anomaly` is owned by 데이터팀 while writing to `db: MySQL (settlement)` — the database of a service the registry assigns to 정산팀. Ownership of the service and ownership of its storage are held by different 본부 (divisions) → sellflow-docs:context/registry/services.yaml
- `delivery-bff` is the only service whose four records conflict on the team itself. The registry says `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` — "transfer to 물류팀 under discussion since 2025-11, not yet confirmed" — while the org chart, the README and `package.json` all assert 물류팀 with no hedge → sellflow-docs:context/registry/services.yaml, sellflow-docs:context/org-chart.md, delivery-bff:README.md, delivery-bff:package.json
- Not one person from 물류팀 appears outside the org chart. 이지훈, its 팀장, occurs in exactly one file across every source and every repo — `org-chart.md` itself. The same is true of 정하늘 (재고팀) → sellflow-docs:context/org-chart.md
- `order-service` is the only service where all four records agree, down to the person: registry 주문팀, org chart 주문팀/박성민, CODEOWNERS `* @sellflow/order-team`, and 박성민's signature on `TASK.md` ("작업 메모 (성민)" — "work notes (Seongmin)"), on three dated source TODOs, on SF-2287 as assignee and on SF-5099 as both assignee and reporter → order-service:.github/CODEOWNERS, order-service:TASK.md, sellflow-docs:context/tickets/SF-2287.md, sellflow-docs:context/sprints/tickets_2026-S17.csv
- Source-comment signatures record the *reporter*, not always the owner: `FIXME(은영) 2024-05` sits in `order-service`'s `OrderSearchService`, but 최은영 leads CS팀, which owns no system in the org chart. Name-in-comment is therefore evidence of who raised a thing, not of who owns the file → order-service:src/main/java/kr/co/sellflow/order/search/OrderSearchService.java, sellflow-docs:context/org-chart.md
- The only *approved* allocation of responsibility in the corpus is procedure v1.1 §3 책임 (revised 2024-02-19, 승인 재무본부장 — approved by the head of the Finance division): 정산팀 "정정 대상 확인, 차월 차감 반영, 파트너 통지"; 주문팀 "취소 이벤트 발행"; 재무기획팀 "분기 결산 시 미정정 잔액 확인" → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md
- That approved allocation describes a human procedure. No document anywhere in the corpus assigns the *registration of a schedule for* `CancelReconciler` to any team — the task exists only as an unassigned ticket and a code comment → sellflow-docs:context/sprints/tickets_2026-S17.csv, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- Ownership records also drift on non-ownership fields, which is a useful staleness signal: registry says `delivery-bff` is `Node 16` where the README says Node 18, and says `settlement-anomaly` is `Python 3.11` where its README says Python 3.9 → sellflow-docs:context/registry/services.yaml, delivery-bff:README.md, settlement-anomaly:README.md

## Scope

**In scope.** The five repositories under `sellflow/repos/`, the queues declared in the registry, and the four records that assert ownership over them: `sources/context/registry/services.yaml`, `sources/context/org-chart.md` (rendered from `org-chart.xlsx`), CODEOWNERS files in the repos, and the human traces in tickets, sprints, minutes, handover, runbook, source comments, Slack and mail.

**Out of scope.** Individual performance or workload; the mechanics of the cancel-reconciliation gap (see [[PROC-SETTLEMENT-CORRECTION]]); the money at stake (see [[RISK-SETTLEMENT-RECON-BACKLOG]]); the eleven teams in the org chart that own no system.

**Naming rule applied throughout.** People are named only as the source documents name them, and every name below is attached to the file that names it. Where a record names no one, that is stated rather than inferred.

## Current State

### Ownership reconciliation

| Service | Registry says | Org chart says | CODEOWNERS says | Who actually touches it (evidence) | Verdict |
|---|---|---|---|---|---|
| `order-service` | `owner_team: 주문팀` | 커머스본부 · 주문팀 · 팀장 박성민 · 8명 · 담당 프로세스 "주문 접수 · 주문 취소" | `* @sellflow/order-team`; `/src/main/resources/db/` also `@sellflow/dba`; an explicit line for `.../order/legacy/` that grants nothing new | 박성민 throughout: `TASK.md` "작업 메모 (성민)", `TODO(성민) 2021-06-02`, `TODO(성민) 2022-11-08`, `TODO(성민) 2023-04-21`, SF-2287 assignee, SF-5099 assignee+reporter, runbook author 2025-07-15. Also `FIXME(은영) 2024-05` from 최은영 of CS팀 | **Agreed, four ways.** The only such case. Caveat: `@sellflow/dba` has no org-chart counterpart |
| `settlement-batch` | `owner_team: 정산팀`; note "일 정산. 아웃박스 릴레이 잡 동거." — "daily settlement; the outbox relay job cohabits here" | 재무본부 · 정산팀 · 팀장 김도윤 · 3명 · "파트너 정산 · 정산 오류 수기 정정" | **none** | 김도윤 signs both in-repo TODOs (`TODO(도윤) 2024-11-02`, and the `CancelReconciler` TODO dated 2023-04-24); 이수민 works the queue by hand per the handover; SF-5120 assignee 이수민, reporter 김도윤 | **Agreed on the team**, but unenforced in GitHub and carrying an inherited part (the relay) that exists only because of an order-team ticket |
| `inventory-api` | `owner_team: 재고팀` | 데이터플랫폼본부 · 재고팀 · 팀장 정하늘 · 5명 · 비고 "2022년 이관" | **none** | Nobody. 정하늘 appears in no file but the org chart; no inventory ticket, Slack message or signed comment exists in the corpus | **Agreed on paper, unattested in practice.** Three records concur; zero human evidence either way |
| `delivery-bff` | `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전` | 커머스본부 · 물류팀 · 팀장 이지훈 · 6명 · "배송 추적 · 배송 예외 처리", no 비고 | **none** | Nobody. No 물류팀 member appears outside the org chart; no delivery ticket exists in `sources/context/tickets/`; the repo's only deferred work points at SF-4901, which is not in the corpus | **Registry is the outlier and is probably just stale** — three records say 물류팀 — but "probably" is the honest word, because no human trace corroborates either side. See below |
| `settlement-anomaly` | `owner_team: 데이터팀`; "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" | 데이터플랫폼본부 · 데이터팀 · 팀장 윤서진 · 7명; 변경이력 "2025-06-09 settlement-anomaly 운영 시작 (데이터팀)" | **none** | 한지우 signs `TODO(지우) 2025-02-10` in `model/features.py`, pulled the queue count in Slack 2026-01-08, is SF-5121 assignee and the kickoff's 데이터팀 attendee; 윤서진 authored the 2026 plan and ran the 2026-09-01 export | **Agreed on who runs the detector, undefined on who acts on its output** — and the registry says so itself |

### The `delivery-bff` case

The registry is one record against three, and the three are unqualified: `README.md` and `package.json`'s `description` both read "담당: 커머스본부 물류팀" ("owner: Commerce Division, Logistics Team"), and the org chart lists `delivery-bff` in 물류팀's 담당 시스템 column with an empty 비고 — no transfer note, where the chart does annotate transfers elsewhere (재고팀 carries "2022년 이관").

Two things keep this from being a clean "registry is stale" verdict.

First, the registry's hedge is specific and dated in a way stale entries usually are not: `물류팀 이관 논의 중 (2025-11~), 확정 전` — "transfer to 물류팀 under discussion since 2025-11, not yet confirmed." Someone wrote that deliberately, four months before the last review. If the transfer had completed, the natural edit was to replace the TODO, not to leave the sentence intact. The org chart cannot adjudicate this, because its 변경이력 sheet records no delivery-related row at all — the last entries are 2025-09-01 (데이터팀 headcount) and 2026-06-15 (the AX roadmap).

Second, and more useful than either record: there is no evidence of anyone working on this service. 이지훈 appears in one file in the entire corpus. No sprint item, ticket, Slack message, meeting or incident touches `delivery-bff`. The repo has no CODEOWNERS, so a pull request against it requires no named reviewer. The question "who owns delivery-bff" may have no operationally meaningful answer at present, and the registry's TODO may be the most accurate of the four records precisely because it declines to give one.

This is logged for the owner in `.reef/questions-for-owner.md`.

### The `CANCEL_RECON_QUEUE` knot: four partial owners and one unowned seam

The registry states the gap without resolving it: `consumer: TODO   # 확인 필요`. Every other record assigns a *piece*. Laid out in the order the pieces were agreed:

**1. The order team removed the block and stopped at the event boundary.** SF-2287 (reporter 최은영 of CS팀, assignee 박성민 of 주문팀, resolved 2023-04-21, shipped in order-service 2.8.0) removed the settlement check from the cancel API. 박성민 raised the money question himself on 2023-04-05: "정산이 이미 나간 건에 대해 취소가 들어오면 그 돈은 어떻게 되나요? 파트너한테 이미 지급된 금액인데요." — "if a cancellation comes in for something already settled, what happens to that money? It has already been paid to the partner." He then defined his own boundary on 2023-04-10: "그럼 취소 발생 시 정산팀에 알림이 가야 할 것 같은데요. 이벤트 하나 발행하겠습니다. `order.cancelled` 구독하시면 됩니다." — "then the settlement team needs to be notified when a cancellation happens. I'll publish an event. Subscribe to `order.cancelled`." → sellflow-docs:context/tickets/SF-2287.md

**2. The settlement team accepted the compensation, in writing, twice.** 김도윤 on 2023-04-07: "정산팀에서 수기로 정정 처리하겠습니다. 차월 정산에서 차감하는 방식으로 처리하면 됩니다." — "the settlement team will handle corrections manually; deducting from the next month's settlement works." And on 2023-04-11, four words: "네 컨슈머 붙여놓겠습니다." — "yes, we'll attach a consumer." The same comment carries the estimate the whole arrangement rests on: "월 10건 미만" — "under 10 cases a month" → sellflow-docs:context/tickets/SF-2287.md

**3. The code records the seam, dated and signed, and hands it to someone on the other team.** `CancelReconciler`'s class comment reads: `TODO Quartz 스케줄 등록 필요 (박성민님 확인 후 QuartzConfig 에 추가 예정) - 2023-04-24 김도윤` — "Quartz schedule registration needed; to be added to QuartzConfig after confirmation from 박성민 — 2023-04-24, 김도윤." This is the pivot of the whole knot. It sits in `settlement-batch`, a 정산팀 repository, signed by the 정산팀 lead, blocked on a person in 주문팀 who has no ownership of that repository, no CODEOWNERS entitlement over it, and no ticket assigning him the confirmation → settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java

The Slack export shows the same sentence being spoken the same day: 2023-04-24 10:31, 김도윤 — "릴레이까지는 만들어뒀는데 정정 배치 스케줄 등록이 남았습니다. 박성민님 확인 후 추가할게요." — "I've built the relay, but registering the correction batch's schedule is still outstanding. I'll add it once 박성민 confirms." No message in the export, and no entry in any other document, records that confirmation ever arriving → sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json

**4. The ticket was filed the same day and never given a name.** SF-4512 `CancelReconciler Quartz 스케줄 등록`, Task, Status `To Do`, Priority `Low`, Assignee **blank**, Reporter 김도윤, Created 2023-04-24, Resolved blank. It appears in the 2026-S17 export — three years and five months later — in exactly the state it was created. It is the oldest open item in that export by more than three years; the next oldest was created 2026-08-06 → sellflow-docs:context/sprints/tickets_2026-S17.csv

Note the shape of the failure. The blocking condition (박성민's confirmation) lives in a code comment. The ticket that would carry it lives in Jira with no assignee. The person who could unblock it is on a different team and is not the reporter. Nothing in the system connects the three, so no queue anywhere shows this as anybody's work item.

**Who the organisation treats as accountable, when it has to pick.** Twice, under pressure, the question went to 정산팀:

- 문지영 of 재무기획팀 addressed the quarterly-close query to 김도윤 directly, Cc 재무기획팀, asking "차감을 수행하는 주체 (배치인지 수기인지)" — "the party performing the deduction — a batch or a person." 주문팀 was not on the thread → sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml
- Procedure v1.1 §3 책임 ("responsibilities"), the only division-head-approved role table in the corpus, gives 정산팀 "정정 대상 확인, 차월 차감 반영, 파트너 통지" and gives 주문팀 exactly one line, "취소 이벤트 발행" — publish the cancellation event. v0.3 (2023-05-02, drafted by 정산팀 정민호) said the same thing more plainly: "대기열 확인 및 차감: 정산팀 / 이벤트 발행: 주문팀" → sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md, sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md

So the formal answer is settled and has been since 2023: **정산팀 owns the drain, as a manual procedure.** What is unowned is the automation of it — and, critically, the belief that it is already automated.

**The belief gap, on the record.** At the 2026-06-18 kickoff, three years after the arrangement was agreed, the two leads discovered they understood it differently in the room:

> 김도윤: "그럼 차감은 자동으로 들어가고 있는 거죠?"
> 박성민: "저희가 이벤트까지는 보내드리고, 그 뒤는 정산 쪽에서 보시는 걸로 알고 있습니다."
> (이 부분 서로 인지가 다름. 확인 필요)

— 김도윤: "so the deduction is going in automatically, right?" 박성민: "we send the event, and my understanding is that everything after that is looked after by settlement." The minute-taker's own parenthesis: "the two sides understand this differently. Needs confirming." The corresponding action item, `이벤트 이후 구간 담당 확인 — 박성민 / 김도윤` ("confirm ownership of the post-event segment"), is unchecked and carries no due date; two further action-item checkboxes on that page are entirely blank → sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md

The same gap had already been documented twice before that meeting and cleared neither time:

- **The 2025 handover.** 정민호 (leaving 2025-03-31) → 이수민, document status "작성중 (일부 항목 확인 필요)" — "in progress, some items need confirming." §2 step 1 hedges the mechanism itself: "주문팀에서 취소 건이 발생하면 `CANCEL_RECON_QUEUE` 에 쌓이는 것으로 알고 있음" — "my understanding is that when the order team has a cancellation it accumulates in CANCEL_RECON_QUEUE." §3 records an unanswered escalation: "대기열을 자동으로 처리하는 배치가 도는지 확인 필요 … 개발팀에 문의했으나 답변 받지 못했음. (2025-02 문의)" — "need to confirm whether a batch processes the queue automatically … asked the dev team but received no answer (asked 2025-02)." §6 미인계 사항 (items not handed over) then lists "위 3번 항목 전부" — all of §3 → sellflow-docs:context/handover/2025-03_정산팀_인수인계.md
- **The 2025-07-12 incident retrospective.** 박성민 raised it in Slack on 2025-07-15: "취소 건이 정산 대상에서 빠지는지 점검 필요해 보입니다. 티켓 만들어 둘까요?" — "we should check whether cancelled orders drop out of the settlement scope. Shall I raise a ticket?" 김도윤 replied "이번 장애랑은 분리해서 보시죠. 저희 쪽에서 정리해서 올리겠습니다" — "let's keep it separate from this incident; we'll write it up on our side." The retrospective records the outcome: "취소 건이 정산 대상에서 제외되는지 점검 — 2025-07-15 제기, 이후 논의 없음" ("raised 2025-07-15, no discussion since") and, in the 비고, "티켓 번호 미확인" — "ticket number unconfirmed." SF-4512 had existed for two years and three months at that moment → sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md, sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json

The same retrospective shows the pattern is not confined to this one item: its remaining prevention action, 지급 요청 전 멱등성 키 도입 (introduce an idempotency key before payout requests), is annotated 담당 미지정 — "owner not assigned."

**Verdict on the knot.** No record is wrong; they are each right about a different segment, and the segments do not meet.

| Segment | Assigned to | By what record | State |
|---|---|---|---|
| Publish `order.cancelled` | 주문팀 | SF-2287 comment 2023-04-10; policy v0.3 §4; policy v1.1 §3 | Done, 2023-04-21 |
| Relay outbox → `CANCEL_RECON_QUEUE` | 정산팀 | Slack 2023-04-24 ("릴레이까지는 만들어뒀는데"); registry `consumer: settlement-batch (OrderEventRelayJob)` | Done, running every 10 minutes |
| Drain the queue, by hand | 정산팀 | policy v1.1 §3 (approved); SF-2287 comment 2023-04-07 | Partially performed — reactively, on partner inquiry only |
| Drain the queue, automatically | **nobody** | SF-4512 unassigned since 2023-04-24; `consumer: TODO`; code TODO blocked on a non-owner | Never started |
| Confirm whose job the automation is | 박성민 / 김도윤 jointly | Kickoff action item, 2026-06-18 | Unchecked, no due date |
| Verify the resulting balance quarterly | 재무기획팀 | policy v1.1 §3 | Performed — and it is what surfaced the problem in 2026-08 |

The one segment with no owner is the only one that would have stopped the balance growing.

### Person-level ownership is not recorded anywhere

The registry defers people to the org chart; the org chart lists only 팀장. The result is that the individuals who hold the operational knowledge are invisible to both authoritative records and can only be recovered by reading tickets and chat:

| Person | Named in | Named by the registry or org chart? |
|---|---|---|
| 박성민 (주문팀) | org chart (팀장), SF-2287, SF-5099, `TASK.md`, three source TODOs, runbook author, kickoff, Slack | Yes — as 팀장 |
| 김도윤 (정산팀) | org chart (팀장), SF-2287, SF-4512 reporter, SF-5120/5121 reporter, two source TODOs, kickoff, Slack, finance mail | Yes — as 팀장 |
| 이수민 (정산팀) | handover recipient, SF-5120 assignee, SF-5088 assignee, kickoff 실무, Slack from 2023-06 | **No** |
| 한지우 (데이터팀) | SF-5121 assignee, `TODO(지우) 2025-02-10`, kickoff 분석, Slack 2026-01-08 | **No** |
| 정민호 (정산팀, left 2025-03-31) | author of policy v0.3, outgoing party in the handover | **No** — and no 변경이력 row records the departure |
| 윤서진 (데이터팀) | org chart (팀장), 2026 plan author, 2026-09-01 export operator | Yes — as 팀장 |
| 최은영 (CS팀) | org chart (팀장), SF-2287 reporter, SF-5088 reporter, `FIXME(은영)` in order-service, kickoff | Yes — as 팀장, but of a team owning no system |
| 문지영 (재무기획팀) | finance mail sender, export requester | **No** — 재무기획팀's 팀장 is listed as 배준호 |

The departure of 정민호 is the sharpest illustration. He wrote the procedure that defines the correction process and was its incumbent until 2025-03-31. His successor inherited a document marked 작성중 whose §6 explicitly did not hand over the open questions, and one of those questions — does the drain batch run? — is the same question the finance division was still asking in August 2026.

### How to use this artifact

- **Never cite one record alone.** For any ownership claim, read the registry, the org chart and the repo's README together; where they disagree, the README and `package.json` have been the more current of the three for `delivery-bff`, while the registry has been the more honest for `CANCEL_RECON_QUEUE`.
- **Treat CODEOWNERS as evidence only for `order-service`.** Its absence in the other four repos means nothing about ownership and everything about enforcement.
- **A team named in a record is not a person who will act.** For 재고팀 and 물류팀 the records give a team and the corpus gives no person at all.
- **When an item spans two teams, look for the seam.** In this corpus the seam is where work stops: an event boundary agreed in a ticket comment, a schedule registration blocked on a member of the other team, an unassigned ticket nobody's board shows.

## Related

- [[CON-ORDER-SETTLEMENT]] — the order↔settlement boundary whose ownership seam this artifact traces
- [[PROC-SETTLEMENT-CORRECTION]] — the mechanism that the unowned segment was supposed to automate
- [[RISK-SELLFLOW-DOC-DRIFT]] — the broader pattern of records diverging from code
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what the unowned segment has cost, measured
- [[SYS-DELIVERY]] — the service with contested ownership and no attested maintainer
- [[SYS-INVENTORY]] — ownership agreed on paper, unattested in practice
- [[SYS-ORDER]] — the only service where all four records agree
- [[SYS-SETTLEMENT]] — owner of the drain, by approved policy
- [[SYS-SETTLEMENT-ANOMALY]] — detector with an owner, output with none
