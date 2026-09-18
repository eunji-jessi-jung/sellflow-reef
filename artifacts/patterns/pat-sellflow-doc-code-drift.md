---
id: "PAT-SELLFLOW-DOC-CODE-DRIFT"
type: "pattern"
title: "How Documents and Code Drift Apart"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Each of the seven shapes below was re-derived on 2026-09-19 by reading the document and then the code it describes, in that order, rather than by re-reading the reef's earlier findings. The shapes are stable properties of how this organisation writes and revises documents, so they will outlive any individual instance; the instance lists will go stale as soon as any document is revised, and the estate has no mechanism that would announce that."
freshness_triggers:
  - "delivery-bff/src/generated/orderApi.ts"
  - "inventory-api/app/main.py"
  - "order-service/.github/pull_request_template.md"
  - "order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "sources/context/business-rules.md"
  - "sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
  - "sources/context/policy/정산_정정_업무절차_v1.1.md"
  - "sources/context/registry/services.yaml"
  - "sources/raw/confluence-snapshots/주문-취소-정책_48213.html"
  - "sources/raw/exports/README.md"
  - "sources/raw/specs/order-service-openapi.json"
known_unknowns:
  - "Whether any document in this estate has ever been revised because the code changed. Every revision recorded here was prompted by an organisational event — a procedure being formalised, a project being proposed — never by a deployment. The absence of a counter-example is not proof that none exists."
  - "Whether newer versions of the Confluence page or the procedure exist outside this reef. The Confluence export is a 2026-08-30 snapshot of a page last modified 2021-03-17; a newer page under a different id would not appear here."
  - "Who is accountable for each document. The procedure names an approving executive but no owner, the Confluence page names an author who has since answered a staleness comment with silence, and the registry names nobody at all."
  - "Whether the two .xlsx originals differ from the Markdown renderings in sources/context/. The renderings were extracted on 2026-09-18 and carry their own regenerate-if-changed note, which is the same shape of instruction that has failed everywhere else in this estate."
  - "Whether 문지영's 2026-08 mail was ever answered. The thread in sources/raw/mail/ ends with her two questions and no reply, so whether the drift was corrected at the point it was finally noticed cannot be established here."
tags:
  - documentation
  - drift
  - governance
  - provenance
  - stale-artefacts
aliases:
  - "문서-코드 드리프트"
  - "documentation decay"
  - "stale spec"
relates_to:
  - type: "refines"
    target: "[[API-ORDER]]"
  - type: "refines"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "depends_on"
    target: "[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]]"
  - type: "refines"
    target: "[[DEC-SELLFLOW-2026-AUTOMATION-SIZING]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-DB-AS-QUEUE]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORPHANED-COMPONENTS]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Generated 2022-11-08; header forbids hand-editing"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:README.md"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "RESTOCKABLE_REASONS = {01, 02}"
  - category: "implementation"
    type: "github"
    ref: "order-service:.github/pull_request_template.md"
    notes: "Three checklist items, none about documentation"
  - category: "implementation"
    type: "github"
    ref: "order-service:README.md"
  - category: "implementation"
    type: "github"
    ref: "order-service:TASK.md"
    notes: "Disclaims itself as unofficial while holding the only record of an open cross-team question"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:README.md"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
    notes: "A convention that exists only as a comment"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/order/openapi.meta.json"
    notes: "The counter-example — an extraction record that carries its own staleness warning"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/sprints/2026-S17_log.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
  - category: "external"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
notes: "Answers Q-047 at the mechanism level. The instance catalogue and severity assessment live in RISK-SELLFLOW-DOC-DRIFT; this artifact is about why the drift keeps happening in the same seven ways, and what an agent should do when it meets one."
---

## Overview

Q-047 asks which documents describe the cancellation policy and which of them the running code contradicts. The document-by-document answer is assembled from [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]], [[CON-ORDER-DELIVERY]] and [[PROC-INVENTORY-RESTOCK]]; [[RISK-SELLFLOW-DOC-DRIFT]] carries the estate-wide instance list rather than that catalogue. This artifact answers the question behind it: the contradictions are not independent accidents. They arrive in seven recognisable shapes, each with its own mechanism, and each shape has produced more than one instance in this estate.

The mechanism common to all seven is easy to state and easy to miss. Nothing in this organisation connects a change in code to the documents that describe it. There is no doc-ownership field, no link from a migration to a policy, no CI step that fails a spec, no review checklist that asks which pages a change invalidates. Documents are therefore written at moments of organisational attention — a ticket being resolved, a procedure being approved, a project being proposed — and are revised at further moments of organisational attention, never at moments of technical change. Between those moments the code moves and the document does not.

This matters for an agent because it makes the documents systematically unreliable in a *predictable direction*. A Sellflow document reliably describes the world as it was when someone last cared about it, not as it is. Knowing which of the seven shapes a document belongs to tells you what kind of wrong it is likely to be, which is more useful than knowing that it might be wrong.

## Key Facts

- **Shape 1, the frozen snapshot: the cancellation policy page has not been edited since 2021-03-17 and still states the rule repealed in 2023.** It asserts `정산이 완료된 주문은 취소할 수 없습니다` — "settled orders cannot be cancelled" — and that the API `SETTLEMENT_DTL에 해당 주문이 포함되어 있는 경우 409 Conflict를 반환합니다` ("returns 409 Conflict when the order is present in SETTLEMENT_DTL"). Today's `OrderCancelService` blocks only `CHWISO` and `BANPUM`; the settled check survives only in the `@Deprecated` `OrderCancelServiceV1` → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html, order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- **The same page carries an instruction to update it and a reader's report that it was not updated.** Header: `이 문서는 주문팀에서 관리합니다. 정책 변경 시 반드시 이 페이지를 갱신해 주세요` ("this document is maintained by the order team; when the policy changes, be sure to update this page"). Comment, 강태오, 2024-08-19: `이 문서 아직 유효한가요? 작년에 취소 정책 바뀐 걸로 아는데 반영이 안 된 것 같습니다` ("is this document still valid? I believe the cancellation policy changed last year and it does not look reflected here"). The page was exported unchanged on 2026-08-30, two years after the comment → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html
- **Shape 2, the generated artefact frozen by the loss of its generator: delivery-bff's order client was generated on 2022-11-08 and cannot be regenerated.** Its header reads `자동 생성 파일입니다. 직접 수정하지 마세요` ("this is an auto-generated file; do not edit it directly"), so the one instruction it carries forbids the only repair available. It still throws on HTTP 409 with `정산이 완료된 주문은 취소할 수 없습니다`, enforcing the 2023-repealed rule on the caller side → delivery-bff:src/generated/orderApi.ts
- **Its `OrderStatus` union encodes a value the server no longer has.** The generated type lists `JUMUN_WANRYO | BAESONG_JUNG | BAESONG_WANRYO | CHWISO | BANPUM`; today's `OrderStatus` enum has seven values beginning `GYEOLJE_WANRYO`, and `JUMUN_WANRYO` is not among them. The repair is tracked and untouched: `재생성은 SF-4901 에서 다루기로 함. (미착수)` — "regeneration is to be handled in SF-4901 (not started)" → delivery-bff:src/generated/orderApi.ts, delivery-bff:src/orderClient.ts, order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- **The second instance of the same shape is the only API specification the estate has.** `order-service-openapi.json` self-reports `springdoc-openapi 자동 생성. 생성일 2022-11-04` against `info.version` 2.4.0, while `SERVER_VERSION` reads 2.8.14. It documents 30 paths; two `@RestController` classes exist. springdoc is not a declared dependency in `build.gradle`, so the artefact is frozen for the same reason as the client — the generator left and the output stayed → sellflow-docs:raw/specs/order-service-openapi.json, sellflow-docs:apis/order/openapi.meta.json, order-service:build.gradle
- **Shape 3, the repealed rule still asserted as procedure: the restock table states a rule inventory-api does not implement.** `business-rules.xlsx` gives 재고 복원 `O` for code 04 (배송 실패 / delivery failure) and `X` for 03; `inventory-api` restocks only `RESTOCKABLE_REASONS = {"01", "02"}`, so code 04 returns `{"restocked": False, "reason": "not_restockable"}`. The spreadsheet's 03 row is right for the wrong reason and its 04 row is simply wrong → sellflow-docs:context/business-rules.md, inventory-api:app/main.py
- **A second instance of the repeal shape sits inside the schema.** V15 renamed `ORDER_CANCEL.BIGO` to `MEMO`; V16 reverted it at most 18 days later — V15's comment records only the month, `2024-01`, while V16's is dated `2024-01-18`, so the interval is bounded but not known — with the reason `정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애` ("the settlement batch query references BIGO directly, causing an incident"). The cross-team dependency that caused the incident is recorded only in the rollback migration's comment; no document names `BIGO` as a shared identifier → order-service:src/main/resources/db/migration/V16__revert_rename_bigo.sql
- **Shape 4, the estimate that becomes a measurement by repetition: one 2023 forecast is now cited as a 2026 finding.** Origin, SF-2287, 김도윤, 2023-04-07: `월 10건 미만일 것으로 예상됩니다` — "it is expected to be fewer than 10 a month", explicitly a prediction. Procedure v0.3, 2023-05-02: `예상 처리량 월 10건 미만` — still marked 예상 (expected). The 2026 plan, §2: `현행 처리량은 정산팀 확인 결과 월 10건 내외로 파악된다` — "current throughput is understood to be around 10 a month per the settlement team's check", now present tense and attributed to a check → sellflow-docs:context/tickets/SF-2287.md, sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md, sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md
- **The measured figure is 10 to 16 times the reused estimate, and the people citing it knew they had not measured.** The 2026-09-01 extraction gives 4,127 PENDING rows over 41 months — 161, 163, 148 and 162 for 2026-05 through 2026-08. At the kickoff three months earlier, 이수민 said `요즘은 좀 더 되는 것 같기는 한데 정확히 세어보진 않았습니다` ("it seems to be a bit more these days, but we have not counted exactly"), and `현행 월 처리 건수 실측` ("measure actual monthly volume") was logged as an action item with `기한 미정` (no due date) → sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv, sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md
- **Shape 5, the self-declared source of truth with an expired review date: the service registry says it is authoritative and records its own neglect in the same file.** Header: `이 파일이 서비스·소유팀·저장소의 단일 기준이다` ("this file is the single standard for services, owning teams and repositories"), followed by `last_reviewed: 2026-03-02   # 이후 갱신 없음` ("no updates since") → sellflow-docs:context/registry/services.yaml
- **Five of the registry's fields are contradicted by the code they describe.** It lists `MySQL (order)`, `MySQL (settlement)` and `MySQL (inventory)` as separate databases where four services resolve to the single `sellflow_order` schema; `Node 16` against delivery-bff's Node 18 README; `Python 3.11` against inventory-api's `FROM python:3.9-slim`; `owner_team: TODO` for delivery-bff; and `consumer: TODO   # 확인 필요` for `CANCEL_RECON_QUEUE` → sellflow-docs:context/registry/services.yaml, sellflow-docs:infra/settlement/runtime.md, inventory-api:Dockerfile, delivery-bff:README.md
- **Shape 6, the TODO as a durable record rather than a task: the estate's open cross-team question lives in a file that disclaims its own authority.** order-service's `TASK.md` lists `- [ ] 정산팀 컨슈머 확인 — 김도윤님 확인 후` ("check the settlement team's consumer — after confirming with 김도윤") and closes with `※ 개인 메모입니다. 공식 문서 아님` ("this is a personal memo, not an official document"). The matching TODO on the other side, in `CancelReconciler`, is dated 2023-04-24 and names the same two people → order-service:TASK.md, settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java
- **The same shape recurs in the incident review and the sprint log.** The 2025-07-12 postmortem leaves three items unticked, one `담당 미지정` (no owner assigned), one marked `2025-07-15 제기, 이후 논의 없음` ("raised 2025-07-15, no discussion since"), and notes that a settlement request was split into a separate ticket whose `티켓 번호 미확인` (number unconfirmed). The 2026-06 minutes' action list ends with two blank checkbox rows → sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md, sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md
- **Shape 7, the verbal agreement that never became an artefact: the estate's central contract was settled in four ticket comments and written down nowhere.** 박성민: `이벤트 하나 발행하겠습니다. order.cancelled 구독하시면 됩니다` ("I will publish an event; you can subscribe to `order.cancelled`"); 김도윤: `네 컨슈머 붙여놓겠습니다` ("yes, I will attach a consumer"). No design document, ADR or interface specification followed → sellflow-docs:context/tickets/SF-2287.md
- **Three years later neither side could state the agreement, and the minute-taker recorded the divergence rather than resolving it.** 김도윤: `그럼 차감은 자동으로 들어가고 있는 거죠?` ("so the deduction is going in automatically, right?"); 박성민: `저희가 이벤트까지는 보내드리고, 그 뒤는 정산 쪽에서 보시는 걸로 알고 있습니다` ("we send the event, and after that I understand settlement handles it"); the minute adds `(이 부분 서로 인지가 다름. 확인 필요)` — "the two sides understand this differently; needs checking" → sellflow-docs:context/minutes/2026-06-18_정산정정_자동화_킥오프.md
- **A second undocumented agreement governs money.** settlement-batch's `MoneyUtil` comment: `정산은 절사(FLOOR), 주문은 반올림(HALF_UP). 2022 협의 결과이며 문서화되어 있지 않다` — "settlement floors, order rounds half-up; this is the result of a 2022 agreement and is not documented". The convention's only written form is that comment, and the class it lives in is never called in production → settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- **Drift runs in both directions, which is why a single heuristic like "trust the stricter document" fails.** The Confluence page is *stricter* than the code (it forbids a cancellation the code permits); procedure v1.1 is *more optimistic* than the code (it assigns a monthly deduction the code never performs). Both are the same mechanism — each preserves the world at its own writing date — but a reader who assumes documents are conservative will be wrong half the time → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html, sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md
- **The one artefact in this estate that resists the pattern does so by recording its own provenance and doubt.** `openapi.meta.json` states the extraction tier, the tiers attempted, a `staleness_warning` naming the version gap, six pieces of `staleness_evidence` and three `uncertainties` — and points the reader at `openapi.code-derived.json` as `the live-truth surface`. It is stale in the same way the spec is, and unlike the spec it says so → sellflow-docs:apis/order/openapi.meta.json

## Where It Appears

| Shape | Mechanism | Instances | What an agent should expect |
|---|---|---|---|
| **1. Frozen snapshot** | Written correct, with no revision trigger. Ages by sitting still. | Confluence 주문 취소 정책 (2021-03-17, cancellation rule); procedure v0.3 (2023-05-02, kept deliberately as `이력 보존용`); order-service `DateUtil`'s javadoc claiming wide usage that no grep finds | The document describes a real past state. Date it by its last-modified stamp and ask what changed after that date. |
| **2. Generated artefact, generator gone** | Output committed; generator not in the build. Cannot decay gracefully, only stop being true. | `delivery-bff/src/generated/orderApi.ts` (2022-11-08, regeneration = unstarted SF-4901); `raw/specs/order-service-openapi.json` (2022-11-04, springdoc not a dependency); the reef's own `business-rules.md` and `org-chart.md` renderings, which carry a regenerate-if-changed note and no trigger | Check whether the generator still exists before trusting or proposing regeneration. Expect the artefact to be internally consistent and externally wrong. |
| **3. Rule repealed in code, still asserted in procedure** | A deliberate code decision with no propagation step. | SF-2287 removing the settled-cancel block vs the Confluence page; `business-rules.xlsx` restocking code 04 vs inventory-api's `{01, 02}`; the `BIGO` rename reverted by V16 for a cross-team reference no document records | The code is the authority and the document states an intent someone still holds. Report both and say which is enforced. |
| **4. Estimate reused as measurement** | Provenance drops off as a number is copied forward; the hedge disappears in transit. | `월 10건 미만` travelling SF-2287 (2023, 예상) → v0.3 (2023, 예상 처리량) → 2026 plan (현행 ... 파악된다, present tense) against a measured ~100/month; the export README's stated query including `SUM(EXPECTED_AMT)`, a column no migration creates, alongside a CSV whose own header records a different query | Trace any number to its first appearance before using it. A figure that has crossed three documents has usually lost a qualifier. |
| **5. Self-declared registry, expired review** | A file claims authority; authority is asserted once and maintenance is not scheduled. | `services.yaml` (`단일 기준`, `last_reviewed: 2026-03-02 # 이후 갱신 없음`, five fields contradicted); `org-chart.md` 변경이력 ending 2026-06-15 while the plan cites a 2024-07 headcount change as current cause | Read the registry as a declaration of intent about ownership, and the code as the statement of fact about runtime. Where they conflict, the code wins. |
| **6. TODO as durable record** | An unfinished handoff is marked in place instead of tracked, then outlives its context and its people. | `CancelReconciler` TODO (2023-04-24, 41 months); `TASK.md`'s unchecked consumer check, self-disclaimed as `공식 문서 아님`; `services.yaml`'s two `TODO`s; the 2025-07 postmortem's three open items (`담당 미지정`, `이후 논의 없음`, `티켓 번호 미확인`); the 2026-06 minutes' blank action rows; `SettlementReportWriter` and `features.py` TODOs from 2024-11 and 2025-02 | A TODO here is evidence of a decision that was deferred and then forgotten, not of work in progress. Date it and state its age. |
| **7. Verbal agreement, no artefact** | Two teams align in conversation; nothing is written; the two mental models then diverge without contradiction. | SF-2287's four-comment publish/subscribe agreement; the 2026 kickoff where both parties stated incompatible versions of it and the minute recorded `서로 인지가 다름`; the 2025-03 handover's `쌓이는 것으로 알고 있음` and `개발팀에 문의했으나 답변 받지 못했음`; `MoneyUtil`'s undocumented 2022 rounding agreement; the 2019 shared-database decision surviving only in two code comments | There is no document to check. The reef's contract and decision artifacts are reconstructions from code and conversation, and should be cited as such. |

**Where the shapes compound.** The expensive failures in this estate are all two or more shapes stacked. The settlement-correction gap is shape 7 (an agreement never written) plus shape 6 (a TODO that recorded the missing half and was never actioned) plus shape 4 (an estimate that made the gap look unimportant for three years). delivery-bff's repealed-rule enforcement is shape 2 (a frozen generated client) plus shape 3 (a rule repealed in code) plus shape 6 (SF-4901, unstarted). No single shape here has caused material harm on its own.

## Design Intent

**Determinable in outline, and it is not neglect.**

The documents in this estate are written carefully. Procedure v1.1 has an approving executive, a responsibilities table, a retention period, numbered forms and a revision history. The Confluence page has an ownership banner. The registry opens by declaring itself authoritative. The export README opens with `재추출 없이 그대로 인용하지 말 것` — "do not cite this as-is without re-extracting". Every one of these is a deliberate governance gesture by someone who understood the risk.

What the gestures have in common is that each one names an obligation and none creates a trigger. `정책 변경 시 반드시 이 페이지를 갱신해 주세요` is an instruction to a person who will not be present when the policy changes. `last_reviewed` is a field, not a schedule. `재추출 없이 인용하지 말 것` is a warning to a reader who has already opened the file. The estate's document practice is built entirely on obligations attached to documents, and not at all on triggers attached to changes.

The technical counterpart is that no change can find its documents. There is no doc-ownership metadata in any repository, no link from a Flyway migration to the policy it affects, no CODEOWNERS entry covering documentation (order-service's covers `/src/main/resources/db/` and the `legacy/` package only), and no spec check in CI. The pull-request template is the sharpest illustration: its 확인 사항 checklist has exactly three items — `마이그레이션 포함 여부`, `정산 연동 영향 여부`, `롤백 방법` ("whether a migration is included", "whether settlement integration is affected", "rollback method"). The team thought carefully enough about cross-cutting impact to ask about the settlement boundary on every change, and did not think to ask which documents the change invalidates. A developer who wanted to update the right document when removing the settled-cancel check in 2023 would have had to already know that a 2021 Confluence page existed and said so.

The intent that is **not** determinable is whether anyone ever proposed such a trigger. No minute, retrospective or plan in this reef discusses documentation maintenance as a practice, which is itself the finding: in an organisation that has run a P2 incident review and a sprint retrospective in the period covered here, documentation drift has never been an agenda item.

## Trade-offs

**What this way of working buys.**

1. **Documents get written at all, and some of them are good.** Procedure v1.1 is a genuinely competent piece of work: responsibilities, retention, forms, revision history. An organisation that only wrote documents it could guarantee to maintain would have written far fewer, and the reconstruction of intent in this reef would have been impossible.
2. **Superseded versions are kept rather than deleted.** v0.3 survives with `v1.1 로 개정됨. 이력 보존용으로 남겨둔다` ("revised into v1.1; kept for historical record"), and the 2024 draft is archived with a `superseded_by` pointer. That discipline is what makes the estimate's journey in shape 4 traceable at all.
3. **Comments in code carry agreements that would otherwise be lost entirely.** The 2019 shared-database decision, the 2022 rounding convention, the `BIGO` cross-team dependency and the no-broker premise exist only as code comments — a poor archive, but a surviving one, and one that travels with the thing it describes.
4. **Personal artefacts fill the gap quickly.** `TASK.md`, the handover, and the staff spreadsheets that v1.1's forms were supposed to replace are all fast, honest and useful. They are where the open questions actually live.

**What it costs.**

1. **Every document is trustworthy about the past and unreliable about the present, with no marking to distinguish them.** A reader cannot tell a current statement from a 2021 statement without checking the code, which is exactly the work the document was meant to save.
2. **Drift is discovered by accident, late, and usually by an outsider.** The queue backlog surfaced when 재무기획팀's 문지영 noticed an accounting inconsistency — `차감이 정상적으로 이루어지고 있다면 대기 잔액이 이 규모로 누적될 수 없습니다` ("if the deduction were working, the pending balance could not accumulate to this size") — three years and 188,851,520 KRW after the fact.
3. **Stale documents are load-bearing for decisions.** The 2026 automation project's scope, schedule and staffing rest on a number that travelled three documents and lost its hedge; its first listed task is to document rules that `CancelReconciler` has encoded since 2023 — see [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]].
4. **Stale generated artefacts are load-bearing for behaviour, not just belief.** delivery-bff's client does not merely describe the 2022 rule, it enforces it on every caller that goes through `requestCancel`, so a document's staleness has become a runtime behaviour — see [[CON-ORDER-DELIVERY]].
5. **Noticing drift accomplishes nothing without a route to act.** 강태오 flagged the Confluence page in 2024 and it was exported unchanged in 2026; the handover's three open items were passed to the successor with `인수자가 개발팀과 다시 확인할 것` ("the successor should re-confirm with the development team") and the successor was still asking at the 2026 kickoff.
6. **Self-disclaiming documents destroy the signal they carry.** `TASK.md` holds the estate's only written record of the missing consumer and states that it is not an official document, so a reader who honours the disclaimer discards the finding.

**The fair summary.** The problem is not that Sellflow's documents are bad or that nobody cares. The documents are better than average and several people noticed the drift in writing. What is missing is any mechanism that converts noticing into revision, and any mechanism that makes a code change aware of the documents it invalidates. Until one exists, the drift will keep arriving in these same seven shapes.

## Agent Guidance

- **Identify the shape before judging the document.** Shape 1 is wrong about the present and right about its own date; shape 4 is wrong about the magnitude while being right about the direction; shape 5 is wrong about facts and right about intent. "This document is stale" is not an answer a reader can use.
- **Never assume drift runs one way.** This estate has documents stricter than the code and documents more permissive than it. Check the code in both directions rather than applying a safety heuristic.
- **Trace every number to its first appearance.** When a figure appears in a plan, search the tickets, procedures and minutes for its earliest form. In this estate the canonical test is `월 10건`: if a document states it without the word 예상, the qualifier has been lost in transit.
- **For any generated file, find the generator before proposing regeneration.** springdoc is not in order-service's `build.gradle` and no codegen config exists in delivery-bff, so "just regenerate it" is not available for either artefact. Say so rather than recommending it.
- **Read a `last_reviewed` or `fetched_at` field as a claim about the file's age, not its correctness.** `services.yaml` was reviewed in 2026-03 and was already wrong about the databases in 2019.
- **Date every TODO you cite and state its age in months.** A 2023 TODO and a 2026 TODO look identical in a diff and mean entirely different things here. The estate's TODOs are archaeological records, not backlogs.
- **When two documents disagree, do not pick one — check the code and report all three positions.** This is what the reef's contract artifacts do, and it is the only form of answer that survives the next revision of either document.
- **Prefer artefacts that carry their own provenance and doubt.** `openapi.meta.json` and `openapi.code-derived.json` are the reef's live-truth surface for order-service; `openapi.json` is the 2022 artefact. Where the reef offers both, cite the derived one and mention the other — see [[API-ORDER]].
- **When you find a new drift instance, record which shape it is.** The shape is what makes the finding actionable: shape 2 needs a generator, shape 5 needs a review schedule, shape 7 needs a written contract, and shape 6 needs a tracker. A flat list of contradictions invites a documentation rewrite, which this estate has already tried twice — see [[RISK-SELLFLOW-DOC-DRIFT]].
- **Do not use drift to dismiss a document's intent.** Procedure v1.1's monthly review has never happened, and it remains the approved statement of what the 재무본부 expects. An answer that says only "the code does not do this" omits the obligation that still stands — see [[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]].

## Related

- [[API-ORDER]] — the 2022 specification and the code-derived surface beside it
- [[CON-ORDER-DELIVERY]] — a frozen generated client that enforces a repealed rule at runtime
- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — the code change that invalidated four documents and updated none
- [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]] — the decision built on the reused estimate
- [[PAT-SELLFLOW-DB-AS-QUEUE]] — broker vocabulary that survived into documents describing a table
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — components the documents describe as working parts
- [[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]] — the two procedure versions read against what runs
- [[RISK-SELLFLOW-DOC-DRIFT]] — the instance catalogue and severity assessment this artifact explains
