---
id: "PAT-SELLFLOW-ORDER-CANCEL-DIVERGENCE"
type: "pattern"
title: "One Concept, Five Models — How \"An Order Cancellation\" Fragmented"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Pattern-level reading built on a scuba-depth re-read of all five representations in code. The field-by-field evidence lives in CON-SELLFLOW-CANCEL-ENTITY-COMPARISON and is deliberately not restated here; this artifact will go stale when a new service starts writing a cancel record, or when any service gains the amount column none of them has."
freshness_triggers:
  - "delivery-bff/src/generated/orderApi.ts"
  - "inventory-api/app/main.py"
  - "order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "settlement-anomaly/model/detector.py"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
  - "sources/context/tickets/SF-2287.md"
known_unknowns:
  - "Whether anyone ever decided that a cancellation should be modelled five ways, or whether each model was added locally without reference to the others. No design document, ADR or architecture review naming more than one representation exists in this reef; SF-2287 is the closest thing and it discusses only the event."
  - "Which team, if any, owns the *concept* of a cancellation as opposed to one of its representations. services.yaml assigns owners per service and has no concept-level ownership field at all."
  - "Whether the divergence was ever costed. No incident, ticket or retrospective in this reef attributes a failure to the divergence itself — the visible failures are attributed to the missing reconciler instead."
  - "Whether a canonical cancellation model was considered and rejected for a reason. The 2019 shared-database agreement is cited in code comments but the agreement text is not in this reef."
tags:
  - cancellation
  - cross-system
  - modelling
  - shared-database
aliases:
  - "취소 개념 분화"
  - "cancel concept fragmentation"
relates_to:
  - type: "refines"
    target: "[[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]]"
  - type: "depends_on"
    target: "[[DEC-SELLFLOW-SHARED-DB]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ORPHANED-COMPONENTS]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "constrains"
    target: "[[PROC-ORDER-CANCEL]]"
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "Cancellation as a request/response pair, frozen at 2022"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "Cancellation as a transient decision, nothing persisted"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
    notes: "Cancellation as a derived query over order status"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet — the only cross-service statement of what a cancellation implies"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/tickets/SF-2287.md"
    notes: "The 2023 conversation in which a sixth consequence was agreed and never modelled"
notes: "Complements CON-SELLFLOW-CANCEL-ENTITY-COMPARISON. That artifact holds the field-level table; this one holds the pattern-level reading — why a concept fragments this way, what it costs, and how to reason about it."
---

## Overview

Ask any of the five services what a cancellation is and you get five different kinds of answer, not five variants of one answer. order-service says it is **a row**, keyed by order number, written inside a transaction. settlement-batch says it is **a queue entry**, derived from an event, relayed up to ten minutes later. inventory-api says it is **a decision** — a function call that returns `{"restocked": true}` and persists no record of itself. settlement-anomaly says it is **a query result**, recomputed nightly from the order's current status. delivery-bff says it is **a request/response pair** in a TypeScript type, frozen in 2022.

These are different *categories of thing*, not different schemas for one thing. That distinction is what makes the divergence hard to fix and easy to misread. Two schemas for one entity can be reconciled by mapping fields; a row, an event, a decision and a query cannot be reconciled at all without first deciding which one is the cancellation and which are merely consequences of it.

The field-level evidence is in [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]]. This artifact asks the question that table raises but does not answer: why does a concept fragment this way in a shared-database estate, what does the fragmentation cost, and how should an agent reason about it?

## Key Facts

- The five representations are five different modelling categories — row, queue entry, transient decision, derived query, wire type — not five schemas for one entity → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, inventory-api:app/main.py, settlement-anomaly:model/detector.py, delivery-bff:src/generated/orderApi.ts
- Only one representation is written inside the cancelling transaction. `OrderCancelService.cancel` saves `ORDER_CANCEL`, mutates `ORDER_MST`, and appends the outbox row under one `@Transactional`; everything else in the estate is downstream of that commit and therefore of a later, different, possibly absent moment → order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- Not one of the five carries the money. `ORDER_CANCEL` has no amount column, `CANCEL_RECON_QUEUE` has none in V1, V4 or V5, `SETTLEMENT_ANOMALY` stores only a `SCORE`, inventory moves quantities, and the delivery client's `CancelResponse` carries an order number and a status — so the one quantity the 2023 agreement was about is modelled nowhere → order-service:src/main/resources/db/migration/V1__init.sql, settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql, settlement-anomaly:sql/V1__anomaly_schema.sql, delivery-bff:src/generated/orderApi.ts
- The reason code is the only field all five agree exists, and each applies a different vocabulary to it: order-service validates `01`–`04` and throws otherwise, the relay can write `00` from a parse fallback, inventory-api tests membership of `{"01","02"}`, settlement-anomaly ignores reasons entirely → order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, inventory-api:app/main.py, settlement-anomaly:model/detector.py
- Each representation has its own clock. `CHWISO_ILSI` is the JVM's `LocalDateTime.now()` at cancel time; `RECV_DTM` is the database clock up to ten minutes later; `DETECTED_DTM` is `NOW()` during the 03:00 detection run; inventory-api records no time at all — so "when was this cancelled" has three answers and one silence → order-service:src/main/java/kr/co/sellflow/order/domain/OrderCancel.java, settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java, settlement-anomaly:app/main.py, inventory-api:app/main.py
- Two representations disagree on whether cancellation and return are the same event. order-service keeps `CHWISO` and `BANPUM` as distinct `OrderStatus` values and refuses to cancel an order in either; settlement-anomaly merges them: `CANCELLED_STATES = {"CHWISO", "BANPUM"}` → order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java, settlement-anomaly:model/detector.py
- One representation can be destroyed by an unrelated job. `MarkSettledTasklet` overwrites `ORDER_MST.SANGTAE_CD` with `JUNGSAN_WANRYO` for every order in the latest settlement run, with no status predicate — so an order cancelled and then settled stops reading as cancelled in the field that three consumers treat as the answer → settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java
- The divergence has a datable origin. Before SF-2287 (2023-04) a settled order simply could not be cancelled — `OrderCancelServiceV1` raised `정산 완료된 주문은 취소할 수 없습니다` ("a settled order cannot be cancelled") — so "cancelled" and "settled" were mutually exclusive and one representation sufficed. Removing the block created a state the model had never had to express → order-service:src/main/java/kr/co/sellflow/order/legacy/OrderCancelServiceV1.java, sellflow-docs:context/tickets/SF-2287.md
- The compensation agreed in SF-2287 was never given a model at all. 김도윤 wrote "정산팀에서 수기로 정정 처리하겠습니다. 차월 정산에서 차감하는 방식으로 처리하면 됩니다" ("the settlement team will handle the correction manually; deducting it from the following month's settlement is enough"), and a manual monthly deduction is precisely the kind of obligation that has no row anywhere → sellflow-docs:context/tickets/SF-2287.md
- The only artefact that states what a cancellation *means* across services is a spreadsheet, not a schema: the 취소정책 sheet gives per-reason 재고 복원 (restock) and 정산 차감 (settlement deduction) columns — and it disagrees with inventory-api on reason `04` → sellflow-docs:context/business-rules.md, inventory-api:app/main.py
- No registry field names an owner for the concept. `services.yaml` calls itself 단일 기준 ("the single standard") and records `owner_team` per service, per repo and per queue — never per domain concept → sellflow-docs:context/registry/services.yaml
- The fragmentation is invisible to the teams because each representation is locally correct. Every one of the five is a reasonable model of the slice of the problem its owner sees; none is a bug in isolation → all five repos, read side by side

## Where It Appears

The divergence is not confined to cancellation — it is the shape this estate produces whenever a concept crosses a team boundary. Cancellation is simply the most expensive instance.

| Concept | Representations | Categories involved | Reconciled anywhere? |
|---|---|---|---|
| **An order cancellation** | 5 (`ORDER_CANCEL`, `ORDER_EVENT_OUTBOX`+`CANCEL_RECON_QUEUE`, inventory restock call, `SETTLEMENT_ANOMALY` row, `CancelRequest`/`CancelResponse`) | row, event→queue, transient decision, derived query, wire type | no |
| **A settlement run** | 2 (`SETTLEMENT_RUN`, `SETTLEMENT_RUN_LOG`) | two tables, one of which has neither reader nor writer | no — see [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] |
| **A partner fee rate** | 3 (`PARTNER_CONTRACT.FEE_RATE`, `DEFAULT_FEE_RATE = 0.12` in code, the 수수료 sheet's three tiers) | table, constant, spreadsheet | no — only the constant executes |
| **A quantity of stock** | 2 (`ORDER_DTL.SURYANG`, `stock_item.available_qty`) | two schemas, two vocabularies | partly — inventory-api's README carries a translation table |
| **"Cancelled" as a state** | 4 readings (`ORDER_CANCEL` exists / `SANGTAE_CD='CHWISO'` / `CHWISO` or `BANPUM` / the 2022 client's 5-value union) | row existence, enum value, merged set, stale union | no |

The pattern's tell is in the third column. Where two representations are in the same category — two tables, two schemas — a mapping exists or could be written. Where they are in different categories, nobody even attempts one, because there is no obvious place to put it.

## Design Intent

**Partly determinable, and the determinable part is small.**

What the sources do support: the shared database made divergence cheap on purpose. The settlement schema header records the choice — "sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)" ("it uses the same instance as sellflow_order; consolidated decision of 2019") — and `MarkSettledTasklet` records what the choice licensed: "ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)" ("ORDER_MST is a table owned by the order team, but under the consolidated-DB policy the settlement batch updates it directly; agreed 2019") → settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql, settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

What the sources also support: SF-2287 is a decision about an *event*, taken by two people in a ticket comment thread over eighteen days, with no modelling step. 박성민 asked the right question — "정산이 이미 나간 건에 대해 취소가 들어오면 그 돈은 어떻게 되나요?" ("if a cancellation comes in for an order already paid out, what happens to that money?") — and the answer was a process commitment, not a schema. The event was modelled because an event was what the order team could ship; the obligation was not modelled because no team owned it → sellflow-docs:context/tickets/SF-2287.md

**What is not determinable, and must not be guessed:** whether anyone ever intended five representations. There is no design document, no ADR, no architecture review in this reef that names more than one of them in the same breath. The five did not arrive by a decision; they arrived by five decisions taken separately, each reasonable in its own frame. That reading is an inference from the absence of a cross-cutting document, and it is recorded in `known_unknowns` rather than asserted as intent.

## Trade-offs

**What the fragmentation genuinely buys.**

Each team gets to model the cancellation in the terms its own job is expressed in, and ships without negotiating. inventory-api genuinely does not need a cancellation record — its job is a quantity adjustment, and a row describing *why* would be a second copy of a fact order-service already owns. settlement-anomaly genuinely benefits from deriving cancellation from current state rather than from an event, because a derived query catches cases the event pipeline missed, which is exactly what a detector should do. delivery-bff's wire type is the right shape for a BFF. Four of the five models are defensible on their own terms, and the estate shipped SF-2287 in eighteen days because nobody had to agree on a shared model first.

The shared database also makes fragmentation survivable in a way it would not be across a network. Any service can join any other's table, so a missing representation can be reconstructed by a query rather than by a migration and a backfill — settlement-anomaly does precisely this, reading `SETTLEMENT_DTL`, `SETTLEMENT_RUN` and `ORDER_MST` in one statement.

**What it costs.**

1. **No representation is authoritative for the question anyone actually asks.** "Was this order cancelled, and does anyone owe money for it?" needs `ORDER_CANCEL` for the fact, `CANCEL_RECON_QUEUE` for the obligation, and `SETTLEMENT_DTL` for the amount — three tables, two owners, no join key that expresses the relationship.
2. **The most-read representation is the least reliable.** `ORDER_MST.SANGTAE_CD` is what most consumers check, and it is the one field another team's batch overwrites unconditionally.
3. **Divergence is silent.** Nothing compares representations. If the relay stopped writing tomorrow, `ORDER_CANCEL` and `CANCEL_RECON_QUEUE` would diverge without any check failing, because no check exists.
4. **A concept with five models and no owner cannot be changed.** Adding an amount to a cancellation — the single change that would most help — requires a migration in one repo, a parse change in another, a policy decision in Finance, and agreement between three teams, one of which (`delivery-bff`) has `owner_team: TODO` in the registry.
5. **The cost lands on a function of the estate that nobody's local model covers: money.** The 188,851,520 KRW measured in [[RISK-SETTLEMENT-RECON-BACKLOG]] is not caused by the divergence, but the divergence is why it went unnoticed for 41 months — no representation was shaped to make an unmet obligation visible.

**The honest counterfactual.** A single canonical cancellation entity would not have saved this estate. The reconciler was never scheduled; a canonical model with no consumer drains no queue. What a canonical model would have bought is narrower and still valuable: one place to look, one clock, one vocabulary, and a natural home for the amount — which is to say, it would have made the failure legible years earlier, not prevented it.

## Agent Guidance

- **Ask which representation before answering any cancellation question.** "Is order X cancelled?" is ambiguous in this estate. Name the source in the answer: `ORDER_CANCEL` says yes as of the cancel transaction; `ORDER_MST.SANGTAE_CD` says whatever the last writer said, which may be settlement; `SETTLEMENT_ANOMALY` says whatever the detector concluded, if it ran at all.
- **Treat `ORDER_CANCEL` as the fact and everything else as a consequence.** It is the only record written synchronously, with a validated reason, under a unique key. Every other representation is downstream, later, lossier, or optional.
- **Never infer an obligation from a state, or a state from an obligation.** A `CANCEL_RECON_QUEUE` row means "the relay saw a cancel event for an order that had a settlement row" — it does not mean the order is currently cancelled, and it emphatically does not mean money moved. Absence of a row means the relay did not see it, not that nothing is owed.
- **Do not assume a representation is populated because its schema exists.** Four of the five have columns or whole tables with no writer; check [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] before treating a field as data.
- **When asked to "add a field to the cancellation", ask which of the five.** Adding it to one is a one-line migration and a silent new divergence. Adding it to all five requires three teams and a vocabulary decision.
- **Expect the categories to defeat naive mapping.** An agent asked to "reconcile the cancellation models" will try to align field names; that will produce a mapping table that looks complete and answers nothing, because the models differ in what kind of object they are, not in what they call their columns. Start from "what question does each representation answer", not "what columns does each have".
- **Watch for the merge.** settlement-anomaly treats `BANPUM` (return) as cancellation; order-service does not. Any count produced by the detector is a count of cancellations *and* returns, and will not match any count produced from `ORDER_CANCEL`.

## Related

- [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] — the field-level comparison this artifact reads at pattern level
- [[PROC-ORDER-CANCEL]] — the cancellation flow as order-service executes it
- [[PROC-SETTLEMENT-CORRECTION]] — the obligation that no representation models
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what the unmodelled obligation has cost so far
- [[DEC-SELLFLOW-SHARED-DB]] — the 2019 decision that made divergence cheap
- [[PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW]] — the direct table access that lets each model be built independently
- [[PAT-SELLFLOW-ORPHANED-COMPONENTS]] — why a representation's existence does not imply its use
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — the vocabulary split that runs along the same boundaries
- [[GLOSSARY-SELLFLOW]] — the term registry this divergence makes necessary
