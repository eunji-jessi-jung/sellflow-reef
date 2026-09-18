---
id: "DEC-INVENTORY-RESTOCK-BY-REASON"
type: "decision"
title: "Restock Conditional on the Cancel Reason Code"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Written 2026-09-19 from the inventory-api source, the order-service reason enum and client, and the business-rules spreadsheet rendering. No design document for this rule exists in the reef; the only recorded justification is a two-line source comment, so this ADR is reconstructed from code and the rule table it disagrees with."
freshness_triggers:
  - "inventory-api:app/config.py"
  - "inventory-api:app/main.py"
  - "inventory-api:tests/test_restore.py"
  - "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - "sellflow-docs:context/business-rules.md"
known_unknowns:
  - "Why reason 04 (배송 실패) is excluded from RESTOCKABLE_REASONS. The business-rules sheet requires restoration for it, and no comment, commit note, ticket or document read for this artifact gives any reason for its absence. The handler comment justifies only 03."
  - "Whether 04 was ever considered. The constant was written as {'01','02'} in both places with no history available — the repos are not git checkouts in this workspace, so no commit trail could be inspected."
  - "Which of the two RESTOCKABLE_REASONS declarations is intended to be canonical. Neither imports the other, neither is marked deprecated, and their comments differ in wording."
  - "Who approved the business-rules 취소정책 sheet and when. The rendering carries no approval, owner or version metadata; the .xlsx is described as the original but its provenance is not recorded."
  - "Whether restock is performed by some other path — a manual operation, an ops script, or a warehouse system. inventory-api's restock endpoint has no caller in the five repos, but the reef contains no inventory operations runbook that would rule out an out-of-band process."
  - "Why 03 is treated as a proxy for 'shipping already started' rather than the order status being checked. The handler receives only ord_no and reason_code, and does not read ORDER_MST."
  - "What RESTORE_LOG contains. The migration that creates it passes no column definitions, and the RestoreLog dataclass in app/models.py is never instantiated."
tags:
  - "inventory"
  - "cancel"
  - "business-rules"
  - "restock"
  - "adr-reconstructed"
aliases:
  - "재고 복원 사유코드"
  - "RESTOCKABLE_REASONS"
relates_to:
  - type: "constrains"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "depends_on"
    target: "[[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "feeds"
    target: "[[RISK-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet — the reason-by-reason rule table including 재고 복원."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "재고팀 owns inventory-api and the 재고 복원 process; 주문팀 owns 주문 취소."
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "§2 reason codes, §5 — restock deferred to inventory-api and 재고팀."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:README.md"
    notes: "Points the reader at app/main.py for the reason-dependent rule."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
    notes: "First declaration of RESTOCKABLE_REASONS."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
    notes: "Second declaration, the handler, and the comment about 03."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
    notes: "Both tests import from app.config."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "The client with no caller, and a different path and parameter style."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
    notes: "The four reason codes as the order domain defines them."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
    notes: "The cancel path — no inventory call in it."
notes: "The cross-service contract is CON-ORDER-INVENTORY; the flow is PROC-INVENTORY-RESTOCK. This artifact records only the reason-code rule and its divergence from the approved table."
---

# Restock Conditional on the Cancel Reason Code

## Context

When an order is cancelled, the stock it consumed may or may not go back on the shelf. 셀플로우 decided early that this depends on *why* the order was cancelled, and that the decision belongs to 재고팀 rather than 주문팀.

Both halves of that split are documented. The 2021 order-cancel policy defines the four reason codes and then hands the restock question away: 취소 시 재고 복원은 `inventory-api` 가 처리합니다. 취소 사유 코드에 따라 복원 여부가 달라지므로 재고팀과 협의가 필요합니다 — "restock on cancellation is handled by inventory-api. Whether stock is restored depends on the cancel reason code, so agreement with 재고팀 is required" → sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html (§5). The org chart confirms the ownership: 재고팀 sits in 데이터플랫폼본부, owns inventory-api, and holds 재고 복원 as a named process, while 주문 취소 belongs to 주문팀 in 커머스본부 → sellflow-docs:context/org-chart.md

The order domain's own reason enum takes the same line, declining to encode any policy: 사유별 비용 부담 주체는 코드에 정의하지 않는다. 정산 정책은 재무본부 소관이며 별도 기준표를 따른다 — "the party bearing the cost per reason is not defined in code; settlement policy is 재무본부's remit and follows a separate rule table" → order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java

The separate rule table exists. It is the 취소정책 sheet of `business-rules.xlsx`, and it disagrees with the code.

## Decision

**Restock is gated on the cancel reason code, and the gate is an allow-list of two values held in inventory-api.**

```python
# 취소 시 재고를 되돌리는 사유 코드.
# 03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다.
RESTOCKABLE_REASONS = {"01", "02"}
```

— "reason codes for which stock is returned on cancellation. 03 (customer change of mind) is not a restock target because such cancellations often come after shipping has started" → inventory-api:app/main.py

The handler applies it as the first thing it does, before it looks the order up at all:

```python
if req.reason_code not in RESTOCKABLE_REASONS:
    log.info("복원 대상 아님. ord_no=%s, reason=%s", req.ord_no, req.reason_code)
    return {"restocked": False, "reason": "not_restockable"}
```

The same constant is declared a second time in `app/config.py`, with a different comment — 복원 대상 사유코드. 01 파트너귀책, 02 시스템오류 ("restock-target reason codes: 01 partner fault, 02 system error") — and that second copy is the one the tests import. Neither file imports the other.

Decided by: not determinable. No ticket, minute, design note or approval for this rule appears in the reef; the two source comments are the entire written record.

## Key Facts

- The rule is an allow-list of exactly two codes, `{"01", "02"}`, applied to `reason_code` before any order lookup → inventory-api:app/main.py
- The order domain defines four codes, so the allow-list silently excludes half of them: `PARTNER_GWICHAEK("01")`, `SYSTEM_ORYU("02")`, `GOGAEK_BYEONSIM("03")`, `BAESONG_SILPAE("04")` → order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java
- The approved rule table requires restoration for three of the four. Its 재고 복원 column reads O for 01, O for 02, **X for 03**, and **O for 04** → sellflow-docs:context/business-rules.md (Sheet: 취소정책)
- The reason-04 row, verbatim: `| 04 | 배송 실패 (주소불명·수취거부) | 파트너 | O | O | 물류팀 확인 후 처리 |` — "04 | delivery failure (address unknown, receipt refused) | partner [bears the cost] | O [restore stock] | O [deduct from settlement] | handled after 물류팀 confirmation" → sellflow-docs:context/business-rules.md
- So the code and the rule table agree on 01, 02 and 03, and disagree on 04 — and the disagreement is in the direction of not returning goods to stock that the table says should be returned → inventory-api:app/main.py, sellflow-docs:context/business-rules.md
- The exclusion of 03 is the only one the code explains: 03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다 — "03 (customer change of mind) is not a restock target because such cancellations often come after shipping has started" → inventory-api:app/main.py
- That reasoning matches the rule table's note on the same row — 배송 시작 후 취소 시 재고 복원 불가, "stock cannot be restored when cancelled after shipping has started" — and the 2021 policy's 배송 시작 전만 가능 ("possible only before shipping starts") → sellflow-docs:context/business-rules.md, sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html (§2)
- Nothing anywhere justifies excluding 04. I looked in both inventory-api declarations and their comments, the README, the tests, the alembic revisions, the order-service enum and client, the 2021 policy page and the business-rules sheet. The code says nothing about 04 at all → inventory-api:app/main.py, inventory-api:app/config.py, inventory-api:README.md
- The constant is duplicated: `app/config.py:5` and `app/main.py:18` both define `RESTOCKABLE_REASONS = {"01", "02"}`, with no import between them and no deprecation marker on either → inventory-api:app/config.py, inventory-api:app/main.py
- The tests assert against the copy the handler does not use. `tests/test_restore.py` opens with `from app.config import RESTOCKABLE_REASONS`, so both assertions — `"03" not in RESTOCKABLE_REASONS` and `{"01","02"} <= RESTOCKABLE_REASONS` — would still pass if the `main.py` copy were changed to anything at all → inventory-api:tests/test_restore.py
- The tests also assert only a subset relation (`<=`), so adding 04 to the config copy would not fail them either; and no test exercises the handler → inventory-api:tests/test_restore.py
- The whole path is unreachable from order-service. `InventoryClient.restore()` is the only code that would call it, and grepping all of `order-service/src` for `InventoryClient` returns exactly one hit — the class's own declaration. `OrderCancelService.cancel()` saves the cancel, flips the order status, publishes the event, and never touches inventory → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java, order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java
- Even if it were called, it would not reach the handler as written: the client posts to `/inventory/restore?ordNo=…&reason=…` with query parameters and a null body, while inventory-api exposes `POST /stock/restock` taking a JSON body of `ord_no` / `reason_code`. The `/inventory` prefix belongs to `routers.py`, which declares only a stock lookup and a health check → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java, inventory-api:app/main.py, inventory-api:app/routers.py
- The client's javadoc hands the policy away, consistent with the ownership split: 재고 API 연동. 취소 시 복원 요청을 보낸다. 복원 여부 판단은 재고팀 쪽 로직 — "inventory API integration. Sends a restore request on cancellation. The decision on whether to restore is 재고팀's logic" → order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java
- The restock handler reads `ORDER_DTL`, a table of the order domain, directly from inventory-api's own DB connection — so the rule's enforcement point also crosses a schema boundary → inventory-api:app/main.py
- Restock is single-row by construction: the handler uses `fetch_one` on `ORDER_DTL` and updates one `sku`, so a cancelled multi-line order would restore at most one line even on the working path → inventory-api:app/main.py

## Rationale

**The only in-code justification covers 03**, and is quoted in full above: cancellations for 고객 변심 (customer change of mind) frequently arrive after the goods have shipped, so the stock is not on the shelf to return. That reasoning is coherent, is corroborated by two independent policy sources, and is the kind of rule an inventory team would own.

It is also, on the handler's own terms, a proxy rather than a test. The endpoint receives `ord_no` and `reason_code` and nothing else; it never reads the order's status, so "often comes after shipping has started" is applied as an unconditional rule to every 03 case including those cancelled before dispatch. Whether that trade was deliberate is not recorded.

**Nothing justifies the exclusion of 04.** This needs stating plainly, because the two exclusions look alike in the code and are not alike in the record. 03 is excluded with a stated reason that the approved rule table agrees with. 04 is excluded silently, against a rule table that says O, and against a business reading that points the other way: 배송 실패 means the parcel came back — address unknown or receipt refused — which is the case where the goods most certainly *are* available to restock. The sheet also assigns the cost to 파트너 and requires 물류팀 확인 후 처리, neither of which is a reason to leave stock written off.

No document read for this artifact offers a motive for the exclusion, and none is invented here; it sits in `known_unknowns`, along with the question of whether 04 was considered at all. The repos in this workspace are not git checkouts, so no commit message could be consulted — that avenue is closed rather than unexplored.

## Consequences

**The code excludes a reason the approved sheet requires.**

| Reason | Code rule | 취소정책 sheet | Agree? |
|---|---|---|---|
| 01 파트너 귀책 | restock | O | yes |
| 02 시스템 오류 | restock | O | yes |
| 03 고객 변심 | no restock | X | yes |
| 04 배송 실패 | **no restock** | **O** | **no** |

→ inventory-api:app/main.py, sellflow-docs:context/business-rules.md

A 04 cancellation returns `{"restocked": False, "reason": "not_restockable"}` and logs 복원 대상 아님 ("not a restock target"). Nothing raises, nothing alerts, and the response is a 200 — so a caller could not distinguish "policy says no" from "policy says yes but the constant disagrees" without reading the source.

**The duplicated constant makes the divergence hard to fix correctly.** There are two declarations and two audiences: the handler reads `app/main.py`, the tests read `app/config.py`. Anyone correcting the rule by editing `config.py` — the file named `config`, the one the tests import, the one whose comment reads like documentation — would change nothing about the service's behaviour, and every test would still pass. The reverse edit would change behaviour while the tests kept asserting the old rule. Neither file signals which is authoritative.

**The tests assert the copy the handler does not use, and assert it weakly.** `test_reason_03_is_not_restockable` and `test_reason_01_and_02_are_restockable` both operate on the config copy; the second uses a subset check, so it tolerates any superset. There is no test for 04 in either direction, and no test that calls `restock()`. The suite's coverage of this decision is therefore: one true statement about a constant the production path ignores → inventory-api:tests/test_restore.py

**None of it runs.** The rule is unreachable in practice, and the reachability question has two independent failures. First, no caller: `InventoryClient` is instantiated by nothing, and `OrderCancelService.cancel()` — the single cancel path since [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — performs four persistence steps — save `ORDER_CANCEL`, flip `ORDER_MST` to `CHWISO`, save it, insert the outbox row — none of which is an inventory call (the full statement-by-statement listing, seven steps including the reason lookup and the log, is in [[PROC-ORDER-CANCEL]]). Second, no matching route: the client's URL and parameter style do not correspond to any endpoint inventory-api declares. So a contract described in a 2021 policy page, encoded in a service, and covered by tests, has no live path behind it. How I checked: grep for `InventoryClient` across `order-service/src` (one hit, the declaration itself), read of `OrderCancelService.cancel()` in full, and enumeration of every route in `app/main.py` and `app/routers.py`.

**What this means for anyone acting on it.** Correcting the 04 exclusion is a two-line change with no observable effect today, and a material effect the moment the cancel path is reconnected — which is the condition under which nobody would be looking at it. The safer sequence is to reconcile the constant with the sheet, collapse the duplicate, point the tests at the surviving copy and add a 04 case, *before* any work that restores the order-to-inventory call. The surrounding flow and the contract's other gaps are covered in [[PROC-INVENTORY-RESTOCK]] and [[CON-ORDER-INVENTORY]].

**A note on authority.** This artifact treats the 취소정책 sheet as the rule of record because both the order enum and the 2021 policy defer to a separate rule table for exactly this, and it is the only such table in the reef. Its own approval metadata is absent, which is recorded in `known_unknowns` — if a later source shows 재고팀 formally amended the rule for 04, this divergence becomes a documentation defect instead of a code defect. Either way the two disagree today.

## Related

- [[CON-ORDER-INVENTORY]] — the cross-service contract this rule sits inside
- [[DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL]] — the cancel path that would trigger a restock
- [[PROC-INVENTORY-RESTOCK]] — the restock flow end to end
- [[RISK-INVENTORY]] — the inventory risk themes, including the dead integration
- [[SYS-INVENTORY]] — the owning service
