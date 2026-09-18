---
id: "DEC-SELLFLOW-SHARED-DB"
type: "decision"
title: "Shared MySQL Instance Across Services"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Snorkel-depth scan; the decision itself is dated 2019 in code comments and no primary record of it was found, so this artifact reconstructs it from its traces. Re-checked on 2026-09-19 after a correction pass in order-service: OrderStatusService changed, but its class javadoc recording the settlement carve-out — \"정산완료(JUNGSAN_WANRYO) 전이는 이 클래스를 거치지 않는다\" — is unchanged, and MarkSettledTasklet still writes ORDER_MST directly, so every claim here stands."
freshness_triggers:
  - "inventory-api/app/db.py"
  - "order-service/src/main/resources/application.yml"
  - "settlement-anomaly/app/db.py"
  - "settlement-batch/src/main/resources/application.yml"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
known_unknowns:
  - "Rationale not available from code alone — the 2019 agreement is referenced in three comments but no meeting record, ADR or ticket describing it was found"
  - "Whether the shared instance is also shared at the schema level in production, or whether the deployed topology differs from the checked-in configuration"
  - "Which team owns the instance operationally; the registry lists a different database per service and names no DBA owner"
  - "Whether a separation was ever attempted and abandoned"
tags:
  - architecture
  - coupling
  - database
relates_to:
  - type: "constrains"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "constrains"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "constrains"
    target: "[[SYS-INVENTORY]]"
  - type: "constrains"
    target: "[[SYS-ORDER]]"
  - type: "constrains"
    target: "[[SYS-SETTLEMENT]]"
  - type: "constrains"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderStatusService.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/application.yml"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/db.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
notes: "Reconstructed decision. The comments agree on the year and the substance; no primary source was located."
---

## Context

Four of the five services need order data. Order owns it. In 2019, when the order schema was first laid down, a choice was made about how the others would get at it. The migration header for the settlement schema states the outcome plainly: "주의: sellflow_order 와 동일 인스턴스를 사용한다. (2019년 통합 결정)" (note: this uses the same instance as `sellflow_order`; a 2019 consolidation decision).

The same 2019 agreement is invoked twice more in code, both times to justify one service writing another's table.

## Decision

Services share one MySQL instance and read — and in one case write — each other's tables directly, rather than exchanging data through APIs.

## Key Facts

- Order, Settlement, Inventory and Settlement-Anomaly all point at the database `sellflow_order` → `order-service/src/main/resources/application.yml`, `settlement-batch/src/main/resources/application.yml`, `inventory-api/app/db.py`, `settlement-anomaly/app/db.py`
- The settlement migration names the decision and its year → `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- Settlement's tasklet updates Order's `ORDER_MST` under the same justification: "ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)" (ORDER_MST is a table owned by the order team, but under the integrated-DB policy the settlement batch updates it directly) → `settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`
- Order's own status service documents the carve-out from the other side: "정산완료(JUNGSAN_WANRYO) 전이는 이 클래스를 거치지 않는다" (the transition to JUNGSAN_WANRYO does not pass through this class) → `order-service/src/main/java/kr/co/sellflow/order/service/OrderStatusService.java`
- Inventory reads `ORDER_DTL`, an order-domain table, to resolve a restock → `inventory-api/app/main.py`
- Settlement-Anomaly joins `SETTLEMENT_DTL` to `ORDER_MST` in a single query across both domains → `settlement-anomaly/app/main.py`
- The service registry contradicts all of this, listing a separate database per service — `MySQL (order)`, `MySQL (settlement)`, `MySQL (inventory)` → `sources/context/registry/services.yaml`

## Rationale

Not available from code alone. Three comments assert a 2019 decision without recording who made it or why, and no meeting note, ADR or ticket in the reef describes it. The adjacent comment in the order schema — "주의: FK 제약 없음. 성능 이슈로 2019년 설계 당시 제외함" (note: no FK constraints; excluded at design time in 2019 for performance reasons) — suggests the same period favoured operational simplicity and throughput over structural separation, but that is an inference, not a citation.

## Consequences

### Positive

- Settlement can mark thousands of orders settled in one `UPDATE` rather than through per-order API calls → `settlement-batch/src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java`
- Anomaly detection can join settlement rows to order status in a single query with no synchronisation concern → `settlement-anomaly/app/main.py`
- The outbox relay can check "was this order settled?" with a direct `SELECT COUNT(1) FROM SETTLEMENT_DTL`, which is why the relay needs no API from Settlement's producer → `settlement-batch/src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`

### Negative

- A column rename in Order broke Settlement in production: V15 renamed `ORDER_CANCEL.BIGO` to `MEMO` and V16 reverted it at most 18 days later — V15's comment gives only the month `2024-01` against V16's `2024-01-18`, so the gap is bounded, not recorded — with the reason "정산 배치 쿼리가 BIGO 를 직접 참조하고 있어 장애" (the settlement batch query references BIGO directly, causing an incident) → `order-service/src/main/resources/db/migration/V16__revert_rename_bigo.sql`
- Order's status history is incomplete by construction, because the settlement write bypasses the service layer that records it → `order-service/src/main/java/kr/co/sellflow/order/domain/OrderStatusHistory.java`
- Ownership boundaries in the registry and CODEOWNERS no longer describe who can change what; the DBA group guards Order's migration directory, but Settlement's SQL reaches the same tables from another repo → `order-service/.github/CODEOWNERS`

### Neutral

- The vocabulary boundary between domains survives inside one database: Inventory's own README documents the translation it performs when reading order tables → `inventory-api/README.md`
- Cross-service integration in this estate is mostly table-shaped rather than API-shaped, which is why the reef's contract artifacts describe tables and polling jobs rather than endpoints

## Related

- [[SYS-ORDER]] -- owner of the shared order tables
- [[SYS-SETTLEMENT]] -- writes into Order's tables under this decision
- [[SYS-INVENTORY]] -- reads Order's tables under this decision
- [[SYS-SETTLEMENT-ANOMALY]] -- joins across both domains under this decision
- [[CON-ORDER-SETTLEMENT]] -- the boundary this decision shapes most
- [[CON-ORDER-INVENTORY]] -- the boundary where the table read replaced the API call
