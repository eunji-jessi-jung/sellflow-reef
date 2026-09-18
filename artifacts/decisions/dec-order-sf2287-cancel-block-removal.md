---
id: "DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL"
type: "decision"
title: "SF-2287 — Removing the Settled-Order Cancel Block"
domain: "order"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Reconstructed on 2026-09-19 from the SF-2287 Jira export, the 2021 Confluence policy snapshot it repealed, the current OrderCancelService, the deprecated V1 service that still contains the removed check, and the #settlement-dev slack export. No architecture decision record was written at the time — the ticket thread is the whole record. If a real ADR surfaces, it supersedes this."
freshness_triggers:
  - "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
  - "sellflow-docs:context/tickets/SF-2287.md"
  - "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
known_unknowns:
  - "Whether the decision was formally approved by anyone above the two engineers. The ticket thread runs 최은영 (CS) → 박성민 (주문팀) → 김도윤 (정산팀) and ends with a deployment confirmation. No approver, no sign-off field and no reference to a policy-change process appears anywhere in it."
  - "Why the 2021 Confluence policy page was never updated. Its own warning banner says 주문팀 owns it and must update it on policy change; 박성민, the block's author and the page's author, made the change and did not. Nothing in the material records a reason."
  - "Whether the 배송완료 / 반품 row of the 2021 policy table was meant to survive. Post-change, BANPUM blocks a cancel but BAESONG_WANRYO does not, which is not what the 2021 table describes and is not discussed in the ticket."
  - "Whether any other caller still enforces a settlement check. OrderCancelServiceV1 still contains it but is @Deprecated with zero callers in order-service; the delivery-bff generated client still throws on 409 but has no caller either."
  - "The origin of the 214 figure. The ticket states 'CS팀 기준 3월 한 달간 관련 문의 214건' but no underlying CS report is present in the reef."
  - "Why no test was ever added for the case the ticket created. The TODO in OrderCancelServiceTest is unsigned and undated."
tags:
  - "order"
  - "cancel"
  - "sf-2287"
  - "policy-change"
  - "adr-reconstructed"
aliases:
  - "정산 완료 주문 취소 차단 해제"
  - "SF-2287"
relates_to:
  - type: "constrains"
    target: "[[DEC-DELIVERY-GENERATED-CLIENT]]"
  - type: "integrates_with"
    target: "[[DEC-ORDER-OUTBOX-RELAY]]"
  - type: "feeds"
    target: "[[DEC-SELLFLOW-2026-AUTOMATION-SIZING]]"
  - type: "feeds"
    target: "[[DEC-SETTLEMENT-CANCEL-CLAWBACK]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "refines"
    target: "[[RISK-SELLFLOW-DOC-DRIFT]]"
  - type: "parent"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:TASK.md"
    notes: "박성민's personal checklist; the first item is this change."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
    notes: "JUNGSAN_WANRYO is still a declared state."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java"
    notes: "Still contains the removed SETTLEMENT_DTL check; @Deprecated, no callers."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The surviving guard, EnumSet.of(CHWISO, BANPUM)."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java"
    notes: "The untouched TODO for the case this decision created."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The decision record in practice — description, five comments, change log."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "'2.8.0 에서 취소 로직 교체됨'"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "The 2021 policy this decision repealed, never updated."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json"
    notes: "Deployment announcement 2023-04-21 17:02."
notes: "The compensating mechanism agreed in the same thread is recorded separately in DEC-SETTLEMENT-CANCEL-CLAWBACK and deliberately not restated here."
---

# SF-2287 — Removing the Settled-Order Cancel Block

## Context

Until April 2023, `order-service` refused to cancel an order once the settlement batch had paid the partner for it. The rule was deliberate and documented. The 2021 Confluence page 주문 취소 정책, authored by 박성민 of 주문팀, states it in bold: **정산이 완료된 주문은 취소할 수 없습니다.** — "an order whose settlement is complete cannot be cancelled" — and describes the mechanism precisely: the cancel API checks the order's settlement state and returns `409 Conflict` when the order appears in `SETTLEMENT_DTL` → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html (§4)

The business problem was timing. The settlement batch runs at 02:00 KST daily, so an order placed on one day is already settled — and therefore uncancellable — by the following morning. The 2021 page presents this same fact reassuringly ("정산 배치는 매일 새벽 2시에 실행됩니다. 따라서 주문 당일 자정까지는 취소가 자유롭게 가능합니다" — "the settlement batch runs at 2am daily, so cancellation is free until midnight on the day of order"). SF-2287 presents it as the defect: **"주문 다음날 오전이면 이미 차단"** — "by the morning after the order it is already blocked."

CS carried the cost. The ticket reports 214 related inquiries in March 2023 alone, most of them customers cancelling for reasons the company itself considered legitimate — 오배송, 상품 하자, 주문 착오 (mis-delivery, defective goods, ordering mistake). The fallback the 2021 policy prescribed, routing them into the return process, added 7 to 10 days and produced a second complaint → sellflow-docs:context/tickets/SF-2287.md

## Decision

Remove the settlement-state check from the cancel path, and publish an event so 정산팀 can compensate afterwards.

- **Requested** 2023-04-03 by 최은영 (CS팀), in one sentence: 취소 API에서 정산 상태 확인 로직을 제거해 주세요 — "please remove the settlement-state check from the cancel API."
- **Agreed** across five comments, 2023-04-05 to 2023-04-11, between 박성민 (주문팀) and 김도윤 (정산팀). No other approver appears.
- **Resolved** 2023-04-21, shipped as `order-service 2.8.0`, announced in `#settlement-dev` at 17:02 the same day.
- **Confirmed** 2023-04-24 by the reporter: "이번 주 관련 문의 3건으로 줄었습니다" — "related inquiries this week are down to 3."

What changed in code, per the ticket's own 변경 내역 (change log): `OrderCancelService.java` — 정산 상태 확인 로직 제거 (settlement-state check removed), and 이벤트 발행 추가 (`order.cancelled` publication added). The current service carries no reference to settlement at all; its only guard is a two-value set → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java

```java
private static final Set<OrderStatus> CHWISO_BULGA = EnumSet.of(
        OrderStatus.CHWISO,
        OrderStatus.BANPUM);
```

The field comment states the new rule exactly: 이미 취소되었거나 반품 프로세스로 넘어간 주문만 차단한다 — "block only orders already cancelled or moved into the return process."

The compensating half of the bargain — a next-month clawback performed by Settlement — was agreed in the same thread and is recorded in [[DEC-SETTLEMENT-CANCEL-CLAWBACK]]. It is not restated here; what matters for this decision is that the block was removed on the strength of it.

## Key Facts

- The pre-decision rule was a documented policy, not an accident: 정산완료 / X / 취소 불가 in the 2021 policy table, with 409 Conflict named as the mechanism → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html (§3, §4)
- The removed implementation is still readable in the deprecated V1 service: `SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` followed by `throw new IllegalStateException("정산 완료된 주문은 취소할 수 없습니다. ordNo=" + ordNo)` — "an order whose settlement is complete cannot be cancelled" → order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- That V1 class is `@Deprecated` as of 2.8.0 with the note 삭제 예정이나 배치에서 참조 가능성이 있어 남겨둠 ("scheduled for deletion but kept in case a batch references it"), dated 2023-04-21 by 박성민 — the same day the new service shipped → order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java
- The business trigger was volume plus latency: 214 CS inquiries in March 2023, against a settlement batch at 02:00 KST that made orders uncancellable within hours → sellflow-docs:context/tickets/SF-2287.md, sellflow-docs:context/business-rules.md (정산 배치 실행 시각, 매일 02:00 KST)
- The current guard blocks exactly two states, `CHWISO` and `BANPUM`; `JUNGSAN_WANRYO` is not among them although it is still a declared enum constant → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java, order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- `OrderStatus.JUNGSAN_WANRYO` carries the warning 이 상태는 정산팀 배치가 설정하며 주문팀에서 직접 변경하지 않는다 — "this state is set by the settlement team's batch; 주문팀 does not change it directly" — so after this decision an order can be cancelled out of a state 주문팀 does not control → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java
- The decision was taken entirely inside a Jira thread between two team leads; no ADR, no policy amendment and no approval record exists for it in the reef → sellflow-docs:context/tickets/SF-2287.md
- The 2021 policy page was never amended. It still reads as current, and its own banner instructs 주문팀 to update it on policy change: 이 문서는 주문팀에서 관리합니다. 정책 변경 시 반드시 이 페이지를 갱신해 주세요 — "this document is managed by 주문팀; please be sure to update this page when the policy changes" → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html
- A reader noticed the drift 16 months later and was not answered: 강태오, 2024-08-19 — "이 문서 아직 유효한가요? 작년에 취소 정책 바뀐 걸로 아는데 반영이 안 된 것 같습니다" ("is this document still valid? I understand the cancel policy changed last year and it doesn't look reflected"). No reply follows in the snapshot → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html
- The service registry records the outcome without the policy: 2.8.0 에서 취소 로직 교체됨 — "the cancel logic was replaced in 2.8.0" → sellflow-docs:context/registry/services.yaml
- The service javadoc still flags the change as unfinished business: TODO 정산 연동 관련 확인 필요 (SF-2287 이후 정책 변경됨) — "TODO: settlement integration needs checking (the policy changed after SF-2287)" → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- The author's own task list shows the same shape: 취소 API 정산 상태 확인 제거 (SF-2287) checked off, 정산팀 컨슈머 확인 ("confirm the settlement team's consumer") still open → order-service:TASK.md
- No test covers the case the decision created. `OrderCancelServiceTest` ends with an unaddressed `// TODO 정산 완료 주문 취소 케이스 테스트 필요` — "a test is needed for the cancel-a-settled-order case" → order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java
- As of 2026-09-19 the repealed 409 policy has been dead for 1,247 days, counting from the 2023-04-21 deployment → sellflow-docs:raw/slack/settlement-dev_2023-04_2026-08.json

## Rationale

The ticket argues from customer experience and CS load, and states its reasoning in three places. Quoted verbatim, with meaning:

> "CS팀 기준 3월 한 달간 관련 문의 **214건**이 접수되었습니다. 대부분은 고객이 정당한 사유(오배송, 상품 하자, 주문 착오)로 취소를 요청했으나 정산 배치가 이미 실행되어 차단된 케이스입니다."

"By CS's count, 214 related inquiries were received in the single month of March. Most are cases where the customer requested cancellation for a legitimate reason — mis-delivery, defective goods, an ordering mistake — but was blocked because the settlement batch had already run."

> "고객 입장에서는 '어제 주문했는데 왜 취소가 안 되냐'는 것이고, 실제로 정산 배치는 새벽 2시에 돌기 때문에 **주문 다음날 오전이면 이미 차단**됩니다."

"From the customer's point of view it is 'I ordered yesterday, why can't I cancel?' — and in fact, because the settlement batch runs at 2am, by the morning after the order it is already blocked."

> "반품 프로세스로 안내하면 처리 기간이 7~10일로 늘어나 컴플레인이 재차 발생합니다."

"If we route them to the return process, handling stretches to 7–10 days and the complaint recurs."

The engineering objection was raised once, by the implementer, and is worth quoting because it names precisely the risk this decision accepted:

> **박성민, 2023-04-05** — "정산 상태 체크를 빼는 건 어렵지 않습니다. 다만 정산이 이미 나간 건에 대해 취소가 들어오면 그 돈은 어떻게 되나요? 파트너한테 이미 지급된 금액인데요."

"Removing the settlement-state check is not hard. But if a cancel comes in for something already settled, what happens to that money? It's an amount already paid out to the partner."

It was answered by 김도윤 on 2023-04-07 with an offer of manual correction and a volume forecast — the content of [[DEC-SETTLEMENT-CANCEL-CLAWBACK]], and the sizing assumption examined in [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]]. On the strength of that answer, the block came out.

What is *not* determinable from any source read for this artifact: whether anyone weighed the money at risk against the 214 inquiries, whether an approval was sought outside the two teams, and why the policy page was left standing. The thread contains no such discussion. These are recorded in `known_unknowns` rather than guessed at.

## Consequences

**Which states still block a cancel.** Exactly two, both terminal-ish and both about the cancel/return path itself:

| Order state | Blocked after SF-2287? | Blocked per 2021 policy? |
|---|---|---|
| `GYEOLJE_WANRYO` (결제완료) | No | No — 즉시 취소, 전액 환불 |
| `SANGPUM_JUNBI` (상품준비중) | No | No — 즉시 취소, 전액 환불 |
| `BAESONG_JUNG` (배송중) | No | Conditional — 물류팀 확인 필요 |
| `BAESONG_WANRYO` (배송완료) | No | Conditional — 반품 프로세스로 전환 |
| `JUNGSAN_WANRYO` (정산완료) | **No** | **Yes — 취소 불가** |
| `CHWISO` (취소) | Yes | — |
| `BANPUM` (반품) | Yes | — |

→ order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java, sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html (§3)

Two things follow that the ticket does not mention. First, the conditional rows collapsed along with the settled row: the 2021 policy made 배송중 and 배송완료 subject to a 물류팀 check or a conversion to the return process, and the new guard implements neither — a delivered order can be cancelled outright. Second, `BANPUM` blocking a cancel is the only remnant of the return-process routing the old policy relied on.

**Nothing maps a settlement state to 409 any more.** The status code itself still has a producer: `GlobalExceptionHandler` maps `OrderCancelNotAllowedException` to `HttpStatus.CONFLICT`, and `OrderCancelService` throws it when the order is already `CHWISO` or `BANPUM`. What SF-2287 removed is the *settlement* precondition behind it, so a 409 today means "already cancelled or returned", never "already settled" → order-service:src/main/java/kr/co/sellflow/order/controller/OrderController.java. Two artefacts still describe the repealed behaviour as current: the 2021 Confluence page, and the checked-in generated client in delivery-bff, which is the subject of [[DEC-DELIVERY-GENERATED-CLIENT]].

**No test covers the case the decision created.** The test class has three tests — order-not-found, already-cancelled, and a `@Disabled` happy path ("로컬 DB 필요. CI 에서 깨져서 임시 비활성화 - 2023-05-11", "needs a local DB; temporarily disabled because it broke CI") — and a TODO for exactly the case SF-2287 introduced:

```java
// TODO 정산 완료 주문 취소 케이스 테스트 필요
```

"a test is needed for the cancel-a-settled-order case." It is still there. So the behaviour this decision deliberately enabled, and which is the entry point for every row in `CANCEL_RECON_QUEUE`, has never been asserted in CI — and the one test that would exercise the cancel path end to end has been off since 2023-05-11 → order-service:src/test/java/kr/co/sellflow/order/service/OrderCancelServiceTest.java

**The documentation debt is measurable.** 1,247 days after the change, the policy page describing the old behaviour is still the only page in 주문 도메인 on the subject, and it now contradicts both the code and the business-rules spreadsheet, which records the post-decision reality (정산 실행 후 취소 → 정산팀이 차월 정산에서 수기 차감, "cancel after settlement runs → 정산팀 deducts manually in the following month's settlement") → sellflow-docs:context/business-rules.md, see [[RISK-SELLFLOW-DOC-DRIFT]]

**The compensating half.** SF-2287 made the cancel possible; it did not make the money come back. What was agreed in exchange, what was built for it, and what happened to it is recorded in [[DEC-SETTLEMENT-CANCEL-CLAWBACK]].

## Related

- [[DEC-ORDER-OUTBOX-RELAY]] — the transport chosen for the `order.cancelled` event this decision added
- [[DEC-SETTLEMENT-CANCEL-CLAWBACK]] — the compensating mechanism agreed in the same thread
- [[DEC-SELLFLOW-2026-AUTOMATION-SIZING]] — the 월 10건 미만 forecast made in this thread, and what the queue actually shows
- [[DEC-DELIVERY-GENERATED-CLIENT]] — a checked-in client that still encodes the repealed 409
- [[PROC-ORDER-CANCEL]] — the cancel flow as it works today
- [[RISK-SELLFLOW-DOC-DRIFT]] — the un-updated policy page in context
- [[SYS-ORDER]] — the owning service
