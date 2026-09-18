---
id: "GLOSSARY-DELIVERY"
type: "glossary"
title: "Delivery Domain Glossary"
domain: "delivery"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by sweeping every file in delivery-bff (all 11: src/{index,deliveryStatus,orderClient,logger}.ts, src/generated/orderApi.ts, tests/deliveryStatus.test.ts, package.json, tsconfig.json, .eslintrc.json, .env.template, README.md) for every identifier, literal and Korean string, then cross-reading order-service's DeliveryInfo entity, DeliveryInfoRepository, OrderStatus, V5__add_delivery_columns.sql and the 2022 OpenAPI export, plus the org chart and the service registry. The pass resolved the draft's central gap: the 2022 spec does define a six-value baesongSangtae vocabulary (PREPARING, PICKED_UP, IN_TRANSIT, OUT_FOR_DELIVERY, DELIVERED, FAILED) that the earlier draft recorded as non-existent. Goes stale if delivery-bff gains a status vocabulary of its own, if the carrier-code enum changes, if ORDER_DELIVERY acquires a migration, or if the generated client is regenerated under SF-4901."
freshness_triggers:
  - "delivery-bff:.env.template"
  - "delivery-bff:README.md"
  - "delivery-bff:package.json"
  - "delivery-bff:src/deliveryStatus.ts"
  - "delivery-bff:src/generated/orderApi.ts"
  - "delivery-bff:src/index.ts"
  - "delivery-bff:src/logger.ts"
  - "delivery-bff:src/orderClient.ts"
  - "order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java"
  - "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - "order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql"
  - "sellflow-docs:context/org-chart.md"
  - "sellflow-docs:raw/specs/order-service-openapi.json"
known_unknowns:
  - "Whether delivery-bff's runtime status values are the spec's six baesongSangtae values. The vocabulary now has a documented definition — PREPARING, PICKED_UP, IN_TRANSIT, OUT_FOR_DELIVERY, DELIVERED, FAILED in the 2022 spec's DeliveryInfo schema — and PREPARING, the one literal in the BFF, is its first member. But DeliveryStatus.status in the BFF is typed as a bare string, the carrier response is returned verbatim without validation, and no file links the two. The correspondence is now plausible rather than established."
  - "Whether the BFF's carrierCd uses the same five codes as the spec's taekBaeSaCd enum. Nothing constrains it: the field is a plain string on an interface that is cast onto an unvalidated carrier response body."
  - "The Korean company names behind CJ, HANJIN, LOTTE, POST and LOGEN. The spec enumerates the codes and gives no labels, and no other file in any repo or in sources/ expands them. The expansions in the table below are general knowledge, not a citation."
  - "Whether TAKBAE_CD (the ORDER_DELIVERY column) and taekBaeSaCd (the spec field) are guaranteed to hold the same value set. The column carries no constraint and, unlike the spec field, is created by no migration."
  - "Where ORDER_DELIVERY is defined. No migration V1-V24 creates the table or any of its three columns (INVOICE_NO, TAKBAE_CD, BAESONG_SANGTAE); the DeliveryInfo entity and DeliveryInfoRepository map a table that the repository's own DDL history never creates. Verified by grepping every .sql file in order-service."
  - "What 마지막 저장 값 ('the last stored value') refers to. Both the README and the fallback comment in src/index.ts promise it, and delivery-bff has no store of any kind — the registry records db: none, and the fallback returns a hardcoded PREPARING. Whether a store was planned, removed, or never existed is not recorded anywhere."
  - "How consumers are meant to treat the stale flag. It is the only signal that the body is fabricated, the HTTP status stays 200, and no client documentation, README line or API description explains it."
  - "Whether 배송 예외 처리 (delivery exception handling), the process the org chart assigns to 물류팀 alongside delivery-bff, has any system behind it. Nothing in delivery-bff implements anything by that name, and the three-retry-then-give-up path carries the comment 별도 알림은 없다 ('there is no separate alert')."
  - "Who owns the vocabulary. The service registry records delivery-bff's owner_team as TODO with 물류팀 이관 논의 중 (2025-11~), 확정 전 ('transfer to the logistics team under discussion since 2025-11, not yet confirmed'), while the org chart and package.json both name 커머스본부 물류팀."
tags:
  - "delivery"
  - "glossary"
  - "korean"
  - "romanisation"
aliases:
  - "delivery terms"
  - "배송 용어"
relates_to:
  - type: "refines"
    target: "[[API-DELIVERY]]"
  - type: "refines"
    target: "[[CON-ORDER-DELIVERY]]"
  - type: "refines"
    target: "[[DEC-DELIVERY-GENERATED-CLIENT]]"
  - type: "refines"
    target: "[[GLOSSARY-ORDER]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-DELIVERY-STATUS-SYNC]]"
  - type: "refines"
    target: "[[RISK-DELIVERY]]"
  - type: "refines"
    target: "[[SCH-DELIVERY]]"
  - type: "parent"
    target: "[[SYS-DELIVERY]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:.env.template"
    notes: "ORDER_API_BASE, CARRIER_API_BASE, RETRY_COUNT — none of the three is read by any source file."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:README.md"
    notes: "배송사 API를 프록시한다; the 마지막 저장 값 promise."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:package.json"
    notes: "Service description and claimed owning team."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/deliveryStatus.ts"
    notes: "DeliveryStatus interface, CARRIER_API, MAX_RETRY, the retry javadoc."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/generated/orderApi.ts"
    notes: "OrderStatus union, CancelRequest/CancelResponse, the 409 sentence."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/index.ts"
    notes: "The one route, the PREPARING/stale fallback, the listen port."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/logger.ts"
    notes: "The three log levels that constitute the service's whole observability."
  - category: "implementation"
    type: "github"
    ref: "delivery-bff:src/orderClient.ts"
    notes: "requestCancel, sayuCd/bigo, the SF-4901 deferral."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java"
    notes: "ORDER_DELIVERY: INVOICE_NO, TAKBAE_CD, BAESONG_SANGTAE; no getter for takbaeCd."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
    notes: "BAESONG_JUNG and BAESONG_WANRYO with their Korean labels."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/repository/DeliveryInfoRepository.java"
    notes: "Empty JpaRepository; the only access path to ORDER_DELIVERY."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql"
    notes: "UNSONGJANG_BEONHO and TAEKBAESA_CD on ORDER_MST, 2020-11-03, 물류팀 요청."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:apis/delivery/openapi.json"
    notes: "Code-derived delivery surface; names the StaleDeliveryStatus shape."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "물류팀 (이지훈, 6) against delivery-bff; 배송관리팀 (권나래, 15) against no system."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "delivery-bff: owner_team TODO, runtime Node 16, db none."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
    notes: "The only document that defines a delivery status vocabulary, a carrier-code enum and a despatch-status enum."
notes: "Korean terms are given in Hangul with romanisation and English meaning, because the codebase mixes all three. Korean strings quoted here are the source files' own; none is a back-translation from English. Terms the code references but never defines are marked as such in place rather than omitted."
---

# Delivery Domain Glossary

## Overview

Delivery vocabulary in 셀플로우 arrives in three layers at once: English architecture words (BFF, stale), romanised Korean identifiers in the code (`carrierCd`, `TAKBAE_CD`, `BAESONG_SANGTAE`), and the Hangul those identifiers were romanised from. The same concept appears under three or four spellings in three or four files, so this table records each spelling where it actually occurs rather than nominating a preferred form. See [[PAT-SELLFLOW-ROMANISED-NAMING]] for why the estate is like this.

Two structural facts shape the whole registry, and both are worth holding in mind while reading.

**The domain's vocabulary is almost entirely borrowed.** delivery-bff itself defines four field names (`ordNo`, `carrierCd`, `status`, `updatedAt`), one status literal (`PREPARING`), one flag (`stale`) and two functions. Everything else a reader will meet under the word "delivery" — the carrier codes, the waybill number, the despatch states, the six-value status vocabulary — is defined in order-service's 2022 OpenAPI export or in its `ORDER_DELIVERY` entity, neither of which delivery-bff reads → delivery-bff:src/deliveryStatus.ts, sellflow-docs:raw/specs/order-service-openapi.json

**A documented status vocabulary does exist, and nothing points at it.** The 2022 spec's `DeliveryInfo.baesongSangtae` carries a six-value enum — `PREPARING`, `PICKED_UP`, `IN_TRANSIT`, `OUT_FOR_DELIVERY`, `DELIVERED`, `FAILED`. The single status literal in delivery-bff is `PREPARING`, the first of those six. That is strong circumstantial evidence and not a link: the BFF types `status` as a bare `string`, casts the carrier's response body onto its interface without validation, and never references the spec. The order-side column `BAESONG_SANGTAE` that the spec documents is likewise unconstrained → sellflow-docs:raw/specs/order-service-openapi.json, delivery-bff:src/deliveryStatus.ts, order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java

Three disagreements recur below. The carrier code is `carrierCd` in the BFF, `TAKBAE_CD` on `ORDER_DELIVERY`, `TAEKBAESA_CD` on `ORDER_MST` and `taekBaeSaCd` in the spec — four names for one field. The waybill number is `unSongJangBeonho` in the spec, `UNSONGJANG_BEONHO` on `ORDER_MST` and `INVOICE_NO` on `ORDER_DELIVERY`. And `PREPARING` is the only status value in the estate that is not romanised Korean at all, unlike every value of the order-status enum next to it.

## Terms

### delivery-bff's own vocabulary

Everything defined inside the repo. This is the complete list; there is nothing else.

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| BFF | — | Backend-for-frontend: an aggregation layer built for one client rather than as a general service. delivery-bff is one literally — `package.json` describes it as 셀플로우 배송 조회 BFF ("sellflow delivery-lookup BFF"), the README says 배송사 API를 프록시한다 ("it proxies the carrier API"), it exposes one route and the registry records `db: none` → `delivery-bff:package.json`, `delivery-bff:README.md` | [[SYS-DELIVERY]] |
| `DeliveryStatus` | — | The BFF's only declared data shape, four fields: `ordNo`, `carrierCd`, `status`, `updatedAt`, all `string`. It is not a parse result — the carrier body is cast onto it with `res.data as DeliveryStatus`, so the interface describes an expectation, not a validated fact → `delivery-bff:src/deliveryStatus.ts` | [[SCH-DELIVERY]] |
| `syncDeliveryStatus` | — | The one outbound function that runs. Its name says "synchronise", and it synchronises nothing: it performs a read-through `GET` and returns `DeliveryStatus | null` without writing anywhere → `delivery-bff:src/deliveryStatus.ts` | [[PROC-DELIVERY-STATUS-SYNC]] |
| `carrierCd` | 택배사 코드 — carrier code | Field on `DeliveryStatus`. A plain `string` with no enum constraint, populated from whatever the carrier returns. The fifth spelling of the carrier-code concept in the estate and the only English-rooted one → `delivery-bff:src/deliveryStatus.ts` | [[GLOSSARY-SELLFLOW]] |
| `status` | 배송상태 — delivery status | Field on `DeliveryStatus`, typed `string`. The BFF neither constrains it nor maps it; the only value it ever produces itself is the fallback `PREPARING` → `delivery-bff:src/deliveryStatus.ts`, `delivery-bff:src/index.ts` | [[API-DELIVERY]] |
| `updatedAt` | — | Field on `DeliveryStatus`, typed `string`, carrier-supplied. Note that the fabricated fallback body omits it entirely, so a degraded response has no timestamp of any kind → `delivery-bff:src/deliveryStatus.ts`, `delivery-bff:src/index.ts` | [[PROC-DELIVERY-STATUS-SYNC]] |
| `PREPARING` | glossed in the route comment as 준비중 (in preparation) | The hardcoded status returned when the carrier lookup fails three times. A literal in `src/index.ts`, not a value read from any store. It is also the first member of the 2022 spec's `baesongSangtae` enum, which is the closest thing to a definition it has → `delivery-bff:src/index.ts`, `sellflow-docs:raw/specs/order-service-openapi.json` | [[API-DELIVERY]], [[RISK-DELIVERY]] |
| `stale` | — | The boolean `stale: true` added to the degraded body. The sole indication that the status is fabricated rather than fetched, since the HTTP status stays 200. The reef's code-derived spec names the shape `StaleDeliveryStatus` — that name exists only there, not in the code → `delivery-bff:src/index.ts`, `sellflow-docs:apis/delivery/openapi.json` | [[API-DELIVERY]] |
| 마지막 저장 값 | "the last stored value" | What both the README (조회 실패 시 3회 재시도 후 마지막 저장 값을 반환한다 — "on lookup failure, retry three times then return the last stored value") and the fallback comment (조회 실패 시 마지막 저장 값을 반환한다. 없으면 준비중으로 표기 — "on lookup failure return the last stored value; if there is none, mark it as 준비중") promise. **There is no store.** The service has no database, no cache and no file; the fallback is a literal → `delivery-bff:README.md`, `delivery-bff:src/index.ts` | [[RISK-DELIVERY]] |
| `CARRIER_API` | — | The environment variable the code actually reads, defaulting to the placeholder `https://api.carrier.example`. `.env.template` declares a *different* name, `CARRIER_API_BASE`, with a different placeholder host → `delivery-bff:src/deliveryStatus.ts`, `delivery-bff:.env.template` | [[SYS-DELIVERY]] |
| `MAX_RETRY` | — | Hardcoded to `3`. The retry javadoc states the policy and its consequence: 배송사 API가 불안정하여 재시도를 넣어두었다. 3회 실패 시 포기한다. 별도 알림은 없다 ("the carrier API is unstable so retries were added; after three failures it gives up; there is no separate alert") → `delivery-bff:src/deliveryStatus.ts` | [[PROC-DELIVERY-STATUS-SYNC]] |
| `RETRY_COUNT` | — | **Referenced but never read.** Declared in `.env.template` as `3`; no file under `src/` reads it, because the retry limit is the hardcoded `MAX_RETRY` → `delivery-bff:.env.template` | [[RISK-DELIVERY]] |
| `ORDER_API_BASE` | — | **Referenced but never read.** Declared in `.env.template` as `http://order-service.internal`; the generated client issues a relative `fetch` instead, so the base URL has no consumer → `delivery-bff:.env.template`, `delivery-bff:src/generated/orderApi.ts` | [[CON-ORDER-DELIVERY]] |
| `logger` | — | The whole of the service's observability: three functions writing `[INFO]`, `[WARN]` and `[ERROR]` prefixes to the console. `.eslintrc.json` sets `no-console` to `warn`, which the logger itself triggers → `delivery-bff:src/logger.ts`, `delivery-bff:.eslintrc.json` | [[PROC-DELIVERY-STATUS-SYNC]] |
| 배송 상태 조회 실패 | "delivery status lookup failed" | The one Korean log line the service emits, at `warn` level, formatted with the attempt number and `ordNo`. Because a total failure still returns 200, this log is the only trace a degraded response leaves → `delivery-bff:src/deliveryStatus.ts` | [[RISK-DELIVERY]] |
| `ordNo` | 주문번호 — order number | The single identifier the delivery domain joins on: the path parameter of `/delivery/:ordNo`, the carrier lookup key, the `@Id` of `ORDER_DELIVERY`, and the primary key of the order itself. Note what the carrier call implies — `GET ${CARRIER_API}/tracking/${ordNo}` keys tracking on the *order* number, not the waybill number, so the carrier is expected to resolve orders rather than parcels → `delivery-bff:src/index.ts`, `delivery-bff:src/deliveryStatus.ts` | [[CON-ORDER-DELIVERY]] |

### Terms inherited from the checked-in order client

Defined inside delivery-bff, in `src/generated/orderApi.ts` and its wrapper, and true of nothing. Listed because an agent reading the repo will meet them and must not mistake them for order-service's vocabulary. The full contract analysis is in [[CON-ORDER-DELIVERY]]; the decision that put the file there is [[DEC-DELIVERY-GENERATED-CLIENT]].

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `JUMUN_WANRYO` | 주문완료 — order complete | First member of the client's `OrderStatus` union. **Exists nowhere else in the estate**: not in order-service's enum, and zero occurrences in the 2022 spec the client names as its source → `delivery-bff:src/generated/orderApi.ts`, `sellflow-docs:raw/specs/order-service-openapi.json` | [[GLOSSARY-ORDER]] |
| `OrderStatus` (client) | — | A five-value union: `JUMUN_WANRYO`, `BAESONG_JUNG`, `BAESONG_WANRYO`, `CHWISO`, `BANPUM`. The live enum has seven; `GYEOLJE_WANRYO`, `SANGPUM_JUNBI` and `JUNGSAN_WANRYO` are missing and `JUMUN_WANRYO` is invented → `delivery-bff:src/generated/orderApi.ts`, `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java` | [[GLOSSARY-ORDER]] |
| `sayuCd` | 사유코드 — reason code | Field of `CancelRequest`, typed `string` and unconstrained here. Live it must be `01`–`04`; anything else throws inside order-service. The order-side spelling is `CHWISO_SAYU_CD` → `delivery-bff:src/generated/orderApi.ts`, `delivery-bff:src/orderClient.ts` | [[GLOSSARY-ORDER]] |
| `bigo` | 비고 — remark, free-text note | Optional field of `CancelRequest`. The same word as order-service's `BIGO` column, which V15 renamed to `MEMO` and V16 renamed back → `delivery-bff:src/generated/orderApi.ts` | [[GLOSSARY-ORDER]] |
| `sangtaeCd` | 상태코드 — status code | The one field of `CancelResponse` besides `ordNo`. The live controller returns an empty body, so this name describes nothing that is ever received → `delivery-bff:src/generated/orderApi.ts` | [[CON-ORDER-DELIVERY]] |
| 정산이 완료된 주문은 취소할 수 없습니다 | "an order whose settlement is complete cannot be cancelled" | The exact sentence the client throws on any HTTP 409. It states a policy order-service abolished on 2023-04-21; today a 409 means `CHWISO` or `BANPUM` → `delivery-bff:src/generated/orderApi.ts` | [[DEC-DELIVERY-GENERATED-CLIENT]] |
| `SF-4901` | marked 미착수 — "not started" | The ticket named in `orderClient.ts` as the place regeneration will be handled. Not present in `sources/context/tickets/`; the string occurs in the whole estate only in that file → `delivery-bff:src/orderClient.ts` | [[DEC-DELIVERY-GENERATED-CLIENT]] |

### Carrier codes

The five-value `taekBaeSaCd` enum in the 2022 spec is the only enumerated carrier vocabulary anywhere in the estate. Neither the `TAKBAE_CD` column nor the BFF's `carrierCd` is constrained to it.

| Code | Carrier | Source |
|---|---|---|
| `CJ` | CJ대한통운 — the largest Korean parcel carrier | `sellflow-docs:raw/specs/order-service-openapi.json` |
| `HANJIN` | 한진택배 | `sellflow-docs:raw/specs/order-service-openapi.json` |
| `LOTTE` | 롯데택배 | `sellflow-docs:raw/specs/order-service-openapi.json` |
| `POST` | 우체국택배 — the Korea Post parcel service | `sellflow-docs:raw/specs/order-service-openapi.json` |
| `LOGEN` | 로젠택배 | `sellflow-docs:raw/specs/order-service-openapi.json` |

The spec gives the codes and no labels. The company names in the middle column are general knowledge, not a citation — recorded in `known_unknowns` accordingly.

### Status vocabularies

Three separate value sets are in play, and no code maps between any two of them.

**`baesongSangtae` — delivery status, 2022 spec, six values.** The only documented delivery-status vocabulary in the estate → `sellflow-docs:raw/specs/order-service-openapi.json`

| Value | Meaning | Note |
|---|---|---|
| `PREPARING` | preparing for despatch | the only one that also exists as a literal in delivery-bff |
| `PICKED_UP` | collected by the carrier | |
| `IN_TRANSIT` | in the carrier network | |
| `OUT_FOR_DELIVERY` | out with the courier | |
| `DELIVERED` | delivered | |
| `FAILED` | delivery failed | the delivery-side counterpart of cancel reason `04` `BAESONG_SILPAE` (배송 실패) |

**`chulGoSangtae` — 출고상태, despatch status, five values.** A line-item concept in the spec (`OrderDtl`), not an order-level one, and absent from every DDL and every entity: `WAIT`, `PICKING`, `PACKED`, `SHIPPED`, `DONE`. It is the warehouse's vocabulary — 김포센터운영팀 and 용인센터운영팀 own 입고 · 피킹 · 패킹 · 출고 (inbound, picking, packing, despatch) per the org chart — and nothing in any repository reads or writes it → `sellflow-docs:raw/specs/order-service-openapi.json`, `sellflow-docs:context/org-chart.md`

**`OrderStatus` — the order's own two delivery-phase values.** Romanised Korean, unlike both sets above → `order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java`

| Constant | Label (verbatim) | English | Note |
|---|---|---|---|
| `BAESONG_JUNG` | "배송중" | in delivery | present in the checked-in client's union |
| `BAESONG_WANRYO` | "배송완료" | delivery complete | present in the client's union; also the state settlement's reader selects on, so "delivered" is what triggers paying the partner |

The practical consequence of three unmapped vocabularies: a delivered order (`BAESONG_WANRYO` in the order schema, `DELIVERED` in the spec's) reads back from `GET /delivery/:ordNo` as `PREPARING` during a carrier outage, and nothing reconciles the two → [[RISK-DELIVERY]]

### `ORDER_DELIVERY` — the order-side delivery record

Three columns, one entity, one empty repository, and no DDL. Verified by grepping every `.sql` file in order-service for the table name and for each column: no migration V1–V24 creates any of them.

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `ORDER_DELIVERY` | — | The table `DeliveryInfo` maps, one row per order. The entity javadoc states the limitation directly: 배송 정보. 주문당 1건. 분할배송은 지원하지 않는다 (V10 에서 컬럼만 추가됨) — "delivery information; one per order; split delivery is not supported, V10 added the column only" → `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[SCH-DELIVERY]] |
| `INVOICE_NO` | 운송장번호 — waybill number | Column mapped as `DeliveryInfo.invoiceNo`. A false friend: it is a shipping waybill number, not a billing invoice. The same concept is `UNSONGJANG_BEONHO` on `ORDER_MST` and `unSongJangBeonho` in the spec; nothing joins or reconciles the two columns → `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[GLOSSARY-ORDER]] |
| `TAKBAE_CD` | 택배코드 — carrier code | Column mapped as `DeliveryInfo.takbaeCd`. The shortest of the four carrier spellings: 택배 is parcel delivery, 택배사 is the carrier company, so this one names the service where the others name the firm. **The field has no getter** — `DeliveryInfo` exposes `getOrdNo`, `getInvoiceNo` and `getBaesongSangtae` and nothing for `takbaeCd`, so no Java code in the estate can read it → `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[GLOSSARY-ORDER]] |
| `BAESONG_SANGTAE` | 배송상태 — delivery status | Column mapped as `DeliveryInfo.baesongSangtae`, typed `String`, no constraint. The persisted counterpart of the BFF's transient `status`. The 2022 spec's six-value enum is the nearest thing it has to a definition, and the column does not enforce it → `order-service:src/main/java/kr/co/sellflow/order/domain/DeliveryInfo.java` | [[SCH-DELIVERY]] |
| `DeliveryInfoRepository` | — | `interface DeliveryInfoRepository extends JpaRepository<DeliveryInfo, String>` with an empty body — no query methods, and no service or controller in order-service injects it. The only access path to the table, and it is unused → `order-service:src/main/java/kr/co/sellflow/order/repository/DeliveryInfoRepository.java` | [[SCH-DELIVERY]] |

### Delivery columns on `ORDER_MST`

Added by V5 on 2020-11-03 under the header 배송 정보 컬럼 / 2020-11-03 물류팀 요청 / 주문팀 반영 ("delivery information columns; requested by the logistics team, applied by the order team") — the one dated record of the two teams' boundary in the schema.

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `UNSONGJANG_BEONHO` | 운송장번호 — waybill number | `VARCHAR(30) NULL` on `ORDER_MST`, indexed as `IX_ORDER_MST_03`. The spec's `unSongJangBeonho` gives the example `"123456789012"`. Nothing in any repository ever writes it, and delivery-bff never exposes it — `DeliveryStatus` has no waybill field at all → `order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql` | [[GLOSSARY-ORDER]] |
| `TAEKBAESA_CD` | 택배사코드 — carrier company code | `VARCHAR(10) NULL` on `ORDER_MST`, added in the same migration. Constrained to the five codes in the spec and to nothing in the DDL. The third spelling of the carrier concept, alongside `TAKBAE_CD` and `carrierCd` → `order-service:src/main/resources/db/migration/V5__add_delivery_columns.sql` | [[GLOSSARY-ORDER]] |

### Delivery fields the 2022 spec defines and no code implements

Each appears in the spec's `DeliveryInfo` or `OrderMst` schema. None is created by a migration, mapped by an entity or read by delivery-bff → `sellflow-docs:raw/specs/order-service-openapi.json`

| Field | Korean | English | Note |
|---|---|---|---|
| `chulGoIlsi` | 출고일시 | despatch date-time | `format: date-time` |
| `baesongWanRyoIlsi` | 배송완료일시 | delivery-complete date-time | the timestamp behind the `BAESONG_WANRYO` state that settlement selects on |
| `suChwiIn` | 수취인 | recipient | personal data, in the spec only |
| `yeonRakCheo` | 연락처 | contact number | personal data, in the spec only |
| `jaeSiDoHoisu` | 재시도 횟수 | retry count | a carrier-side retry counter, unrelated to the BFF's own `MAX_RETRY` |
| `baesongMsg` | 배송 메시지 | delivery message | `maxLength: 200`, example 부재 시 경비실 ("leave with the guard if absent"); also referenced by a TODO in V13 and created by no migration |
| `baesongBi` | 배송비 | delivery fee | `format: double`, example `3000`; the domain's only money field, and it exists in no table |
| `baesongJuso` | 배송주소 | delivery address | the one delivery field that *does* exist as a column, `ORDER_MST.BAESONG_JUSO`, mapped on `OrderMst` with a getter nobody calls |

### Organisational terms

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| 물류팀 | logistics team | 커머스본부 물류팀 (Commerce Division logistics team), lead 이지훈, 6 people, listed against `delivery-bff` in the org chart and named in both `package.json` and the README. The service registry contradicts it with `owner_team: TODO` → `sellflow-docs:context/org-chart.md`, `sellflow-docs:context/registry/services.yaml` | [[SYS-DELIVERY]] |
| 배송 추적 | delivery tracking | One of the two processes the org chart assigns to 물류팀. It is what `syncDeliveryStatus` does → `sellflow-docs:context/org-chart.md` | [[PROC-DELIVERY-STATUS-SYNC]] |
| 배송 예외 처리 | delivery exception handling | The other process assigned to 물류팀. **Nothing in delivery-bff implements anything by that name**, and the exception path the code does have ends in 별도 알림은 없다 ("there is no separate alert") → `sellflow-docs:context/org-chart.md`, `delivery-bff:src/deliveryStatus.ts` | [[RISK-DELIVERY]] |
| 배송관리팀 | delivery management team | 물류운영본부 배송관리팀, lead 권나래, 15 people, owning 배송사 관리 · 라스트마일 운영 ("carrier management, last-mile operations"). The team that deals with the carriers, listed against no system — the `-` in the systems column → `sellflow-docs:context/org-chart.md` | [[SYS-DELIVERY]] |
| 배송사 | carrier company | The word the README uses for the upstream — 배송사 API를 프록시한다. The carrier behind `CARRIER_API` is never identified: the code default and `.env.template` both give placeholder hosts, and no carrier contract or SLA document exists in `sources/` → `delivery-bff:README.md`, `delivery-bff:.env.template` | [[SYS-DELIVERY]] |

### The 배송 (baesong) root

배송 is delivery, and it is the root of most Korean identifiers in this domain. Collected for a reader who meets an unfamiliar one: 배송상태 (delivery status), 배송주소 (delivery address), 배송비 (delivery fee), 배송중 (in delivery), 배송완료 (delivery complete), 배송 메시지 (delivery message), 배송 실패 (delivery failure), 배송사 (carrier company), 배송 추적 (delivery tracking), 배송 예외 처리 (delivery exception handling). Distinguish it from 택배 (taekbae), parcel delivery as a service, and 택배사 (taekbaesa), the company providing it → `sellflow-docs:raw/specs/order-service-openapi.json`, `sellflow-docs:context/org-chart.md`

## Related

- [[SYS-DELIVERY]] — the service these terms describe
- [[API-DELIVERY]] — where these terms appear on the wire
- [[SCH-DELIVERY]] — the field-level shapes behind these names
- [[CON-ORDER-DELIVERY]] — the client vocabulary and what each of its terms is worth
- [[DEC-DELIVERY-GENERATED-CLIENT]] — why the client's dead vocabulary is in the repo at all
- [[PROC-DELIVERY-STATUS-SYNC]] — the retry, the fallback and the log line in motion
- [[RISK-DELIVERY]] — the fabricated status, the unread env vars and the unreconciled vocabularies
- [[GLOSSARY-ORDER]] — the order-side spellings of the carrier code, the waybill number and the cancel fields
- [[GLOSSARY-SELLFLOW]] — the cross-service disambiguation table
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — why one concept has four spellings
