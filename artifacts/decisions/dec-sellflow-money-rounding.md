---
id: "DEC-SELLFLOW-MONEY-ROUNDING"
type: "decision"
title: "Divergent Money Rounding Between Order and Settlement"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-18
freshness_note: "snorkel-depth scan; both MoneyUtil classes and the settlement processor were read in full, and the comments themselves say the agreement is undocumented"
freshness_triggers:
  - "order-service/src/main/java/kr/co/sellflow/order/common/MoneyUtil.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
  - "settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
known_unknowns:
  - "Rationale not available from code alone — the comment says the rule was agreed in 2022 and is not documented anywhere, and no such document was found"
  - "Which figure a partner actually sees in the portal, and therefore which rounding is customer-facing"
  - "Whether the 1-KRW divergence has ever been measured or reconciled"
  - "Why the settlement processor rounds inline with HALF_UP instead of calling its own FLOOR utility"
tags:
  - money
  - rounding
  - settlement
aliases:
  - "수수료 절사 vs 반올림"
relates_to:
  - type: "constrains"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "constrains"
    target: "[[RISK-SETTLEMENT]]"
  - type: "constrains"
    target: "[[SYS-ORDER]]"
  - type: "constrains"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/common/MoneyUtil.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "수수료 sheet: 기본 12.0%"
notes: "Three rounding behaviours, two of them in the same repo."
---

## Context

Two services compute a fee on the same amount. Both have a class called `MoneyUtil` with a method called `fee(long amount, double rate)`. The two implementations return different numbers, and each one's comment points at the other.

## Decision

The decision on record is that Order rounds fees half-up for display while Settlement truncates them when paying out — deliberate and, by the comments' own admission, unwritten. The code does not carry it out. The only rounding that reaches a payout is `SettlementItemProcessor`'s inline `setScale(0, RoundingMode.HALF_UP)`; `FLOOR` exists solely in `MoneyUtil.fee()`, which no production path calls. So both services round half-up, and the 2022 agreement survives only as a comment in a dead utility and a test that exercises it.

## Key Facts

- Order's utility uses `RoundingMode.HALF_UP` and warns that the paid figure may differ: "표시 금액과 지급 금액이 1원 단위로 다를 수 있다" (the displayed amount and the paid amount can differ by a single won) → `order-service/src/main/java/kr/co/sellflow/order/common/MoneyUtil.java`
- Settlement's utility uses `RoundingMode.FLOOR` and states the agreement is undocumented: "정산은 절사(FLOOR), 주문은 반올림(HALF_UP). 2022 협의 결과이며 문서화되어 있지 않다" (settlement truncates, order rounds half-up; this was agreed in 2022 and is not documented) → `settlement-batch/src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java`
- The batch's actual payout calculation uses neither utility: `SettlementItemProcessor` multiplies inline and calls `setScale(0, RoundingMode.HALF_UP)` → `settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`
- The only test in the settlement repo asserts truncation through `MoneyUtil.fee(14999, 0.1) == 1499`, so it passes while the production path rounds the other way → `settlement-batch/src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java`
- The fee rate in the processor is hard-coded at `0.12`, matching the spreadsheet's 기본 12.0% tier and ignoring the premium and promotional tiers recorded beside it → `sources/context/business-rules.xlsx`

## Rationale

Not available from code alone. The settlement comment dates the agreement to 2022 and says explicitly that it was never written down. The plausible reading for what was *agreed* — truncation favours the platform on every payout, half-up gives the customer-facing figure the expected arithmetic — is an inference and is recorded here as such, not as a finding. Why the processor rounds inline with `HALF_UP` rather than calling the `FLOOR` utility beside it is not recoverable from any source; nothing states whether that was a reversal, an oversight, or a refactor that never returned.

## Consequences

### Positive

- Each side's intent is at least stated in a comment, which is how the divergence is discoverable at all → `settlement-batch/src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java`

### Negative

- Three behaviours exist where the comments describe two: order half-up, settlement-utility floor, and settlement-processor half-up. The class that actually moves money is the one nobody's comment describes → `settlement-batch/src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`
- The repo's single test gives false assurance: it exercises the utility the payout path does not use → `settlement-batch/src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java`
- `FEE_MISMATCH` is one of the anomaly types the detection service documents but has not implemented, so the one component that could have caught a rate or rounding discrepancy does not look for it → `settlement-anomaly/README.md`

### Neutral

- At 12 percent the difference is at most one won per line, which is why it has plausibly gone unexamined for four years; the spreadsheet's other tiers (9.5%, 6.0%) are not applied by any code path, so the question of their rounding has never arisen → `sources/context/business-rules.xlsx`

## Related

- [[SYS-ORDER]] -- half-up side
- [[SYS-SETTLEMENT]] -- the side that was supposed to truncate; its live processor rounds half-up and its FLOOR utility is uncalled
- [[PROC-SETTLEMENT-DAILY-BATCH]] -- where the payout figure is actually computed
- [[RISK-SETTLEMENT]] -- the hard-coded rate and the misleading test
