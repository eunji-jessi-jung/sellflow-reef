---
id: "RISK-SELLFLOW-DOC-DRIFT"
type: "risk"
title: "Documentation Drift Across Sellflow"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-18
freshness_note: "snorkel-depth scan; every contradiction listed was checked against both the document and the code on the same day"
freshness_triggers:
  - "order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - "sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
  - "sources/context/policy/정산_정정_업무절차_v1.1.md"
  - "sources/context/registry/services.yaml"
  - "sources/raw/confluence-snapshots/주문-취소-정책_48213.html"
  - "sources/raw/specs/order-service-openapi.json"
known_unknowns:
  - "Which of these documents anyone actually consults today; only the Confluence page carries a reader comment challenging its currency"
  - "Whether a newer version of the cancellation policy exists outside this reef"
  - "Who owns each document; the procedure names an approving executive, the Confluence page names an author, the registry names nobody"
  - "Whether the registry is generated from anything or maintained by hand"
severity: "medium"
resolution: "open"
tags:
  - documentation
  - drift
  - governance
aliases:
  - "문서 드리프트"
relates_to:
  - type: "refines"
    target: "[[API-ORDER]]"
  - type: "refines"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "refines"
    target: "[[CON-ORDER-SETTLEMENT]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/service/OrderCancelService.java"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/plans/2026_정산정정_AI에이전트_자동화_기획.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
  - category: "external"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
notes: "Cross-cutting risk. Each individual drift is small; the pattern is that no document in this estate is invalidated by the change that invalidates it."
---

## Description

Every authoritative document in this reef describes a system that no longer behaves the way it says. That is unremarkable on its own — documents age. What makes it a risk here is the shape of the ageing: in each case the code changed and the document did not, nobody noticed at the time, and a later decision was then made on the strength of the stale document.

The 2026 automation plan is the clearest instance. It cites SF-2287's 2023 volume estimate as current fact, and its scope and schedule follow from that number.

## Key Facts

- The Confluence cancellation policy still states that settled orders cannot be cancelled and that order-service returns 409 for them, a rule removed in April 2023 → `sources/raw/confluence-snapshots/주문-취소-정책_48213.html`, `order-service/src/main/java/kr/co/sellflow/order/service/OrderCancelService.java`
- A reader flagged that page as stale two years ago and it was never revised: "이 문서 아직 유효한가요? 작년에 취소 정책 바뀐 걸로 아는데 반영이 안 된 것 같습니다" (is this document still valid? I believe the cancellation policy changed last year and it does not look reflected here), 강태오, 2024-08-19 → `sources/raw/confluence-snapshots/주문-취소-정책_48213.html`
- The page's own header instructs the opposite of what happened: "정책 변경 시 반드시 이 페이지를 갱신해 주세요" (when the policy changes, be sure to update this page) → `sources/raw/confluence-snapshots/주문-취소-정책_48213.html`
- The 2026 automation plan repeats the 2023 estimate as a present-tense finding, "현행 처리량은 정산팀 확인 결과 월 10건 내외로 파악된다" (current throughput is understood to be around 10 cases a month per the settlement team's check), against a measured 100-160 per month → `sources/context/plans/2026_정산정정_AI에이전트_자동화_기획.md`, `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv`
- The issued procedure v1.1 assigns Settlement a monthly queue review that the 2025 handover says has never existed as a practice → `sources/context/policy/정산_정정_업무절차_v1.1.md`, `sources/context/handover/2025-03_정산팀_인수인계.md`
- The procedure mandates two forms, ST-F-001 and ST-F-002, retained five years; the handover reports there is no official record format and staff keep personal spreadsheets: "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중" → `sources/context/policy/정산_정정_업무절차_v1.1.md`, `sources/context/handover/2025-03_정산팀_인수인계.md`
- The service registry declares itself the single source of truth — "이 파일이 서비스·소유팀·저장소의 단일 기준이다" (this file is the single standard for services, owning teams and repositories) — and has not been reviewed since 2026-03-02 → `sources/context/registry/services.yaml`
- The registry lists three separate databases where the code shows one shared instance, records Node 16 against a Node 18 README, Python 3.11 against a Python 3.9 Dockerfile, and carries `owner_team: TODO` for delivery-bff → `sources/context/registry/services.yaml`
- The 2022 OpenAPI document remains the only API specification, describing 30 paths of which 2 survive in today's controllers → `sources/raw/specs/order-service-openapi.json`
- The business-rules spreadsheet's restock column disagrees with inventory-api for reason code 04 → `sources/context/business-rules.xlsx`, `inventory-api/app/main.py`

## Impact

The direct impact is misplaced confidence. Three decisions in the record were made on stale documents:

1. The 2026 automation project is scoped, staffed and scheduled against a volume estimate that is an order of magnitude below the measured number, and its 선결 과제 list still includes documenting the current rules — meaning the project intends to discover, in 2026-09, what the CSV export already shows.
2. delivery-bff's generated client enforces a repealed rule on the caller side because the specification it was generated from was never reissued.
3. CS guidance, insofar as it follows the Confluence page, would still route settled-order cancellations into the returns process that SF-2287 existed to avoid.

The indirect impact is that the reef cannot treat any single document as authoritative. Where a document and the code disagree, this reef records both and says which is which.

## Severity and Resolution

**Severity:** medium at snorkel depth. No drift found here is individually dangerous; collectively they are the mechanism by which the settlement-correction gap stayed invisible for three years.

**Resolution:** open. The cheapest corrective is not a documentation rewrite but a single number: re-running the queue export and putting the result into the automation plan would invalidate the estimate that everything else is built on. The registry's own staleness marker (`last_reviewed: 2026-03-02`) and the Confluence comment thread show the drift is already visible to the people involved; what is missing is a trigger that forces a document to be revisited when the code it describes changes.

## Related

- [[CON-ORDER-SETTLEMENT]] -- the boundary where document and code diverge most expensively
- [[CON-ORDER-DELIVERY]] -- a repealed rule preserved in generated code
- [[RISK-SETTLEMENT-RECON-BACKLOG]] -- the measured consequence
- [[API-ORDER]] -- the stale specification in detail
- [[PROC-SETTLEMENT-CORRECTION]] -- procedure v1.1 against what runs
- [[GLOSSARY-SELLFLOW]] -- where the same term is defined differently by different documents
