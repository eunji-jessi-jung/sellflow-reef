---
id: "PAT-SELLFLOW-ROMANISED-NAMING"
type: "pattern"
title: "Romanised Korean Naming in Schemas and Code"
domain: "sellflow"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Snorkel-depth scan; the pattern is visible in every repo and its exceptions were enumerated by reading the schemas side by side. Re-checked on 2026-09-19 against order-service V1 and inventory-api's Alembic base: the two 택배사 spellings are now both in DDL (TAEKBAESA_CD on ORDER_MST, TAKBAE_CD on ORDER_DELIVERY), and inventory-api's RESTORE_LOG mixes the two conventions inside one table. The JEOKRIPGEUM/JEOKLIP_AMT example no longer holds — V20 was corrected to drop the columns V7 actually added — and has been replaced. Re-check if a migration lands in either Java repo or in inventory-api."
freshness_triggers:
  - "inventory-api/sql/V1__stock.sql"
  - "order-service/src/main/resources/db/migration/V1__init.sql"
  - "settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql"
  - "sources/raw/specs/order-service-openapi.json"
known_unknowns:
  - "Whether a naming standard was ever written; no such document exists in this reef"
  - "Why inventory-api broke the convention in 2022 — whether it was a team preference or a stated policy"
  - "Which romanisation scheme, if any, was intended; the spellings are inconsistent (CHAENNEL_CD, JEOKRIPGEUM, OKSYEON_MYEONG)"
tags:
  - conventions
  - naming
  - schema
aliases:
  - "Korean romanisation convention"
relates_to:
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[SCH-INVENTORY]]"
  - type: "refines"
    target: "[[SCH-ORDER]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-BATCH]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:README.md"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/OrderStatus.java"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V1__init.sql"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/resources/db/migration/V3__add_channel_code.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "external"
    type: "doc"
    ref: "sellflow-docs:raw/specs/order-service-openapi.json"
notes: "Not a defect. Documented because an agent reading these schemas without it will mistranslate the domain."
---

## Overview

Almost every identifier in the Java and SQL half of this estate is Korean written in Latin letters: `SANGTAE_CD` is 상태코드 (status code), `CHWISO_SAYU_CD` is 취소사유코드 (cancellation reason code), `JUNGSAN_WANRYO` is 정산완료 (settlement complete). The words are Korean; only the alphabet is not. Nothing is translated, so an English reader gets no help from the identifier, and a machine translator gets nothing to work with either.

The pattern runs consistently through order-service and settlement-batch, which were built in 2019 by teams working in Korean, and stops at the two services built later by other divisions. Where it stops is the interesting part: the boundary between romanised and English naming is also an ownership boundary, and crossing it is where the estate's vocabulary problems live.

## Key Facts

- The 2019 order schema is romanised throughout: `ORD_NO`, `GOGAEK_ID`, `JUMUN_ILSI`, `SANGTAE_CD`, `CHONG_GEUMAEK`, `BAESONG_JUSO` → `order-service/src/main/resources/db/migration/V1__init.sql`
- Enum constants follow the same rule, with the Korean label carried alongside as data: `JUNGSAN_WANRYO("정산완료")` → `order-service/src/main/java/kr/co/sellflow/order/domain/OrderStatus.java`
- Settlement's schema, written by a different division, keeps the convention: `JUNGSAN_ILJA`, `JUNGSAN_AMT`, `SUSURYO`, `SAYU_CD` → `settlement-batch/src/main/resources/db/migration/V1__settlement_schema.sql`
- inventory-api, transferred to the data-platform division in 2022, uses English instead: `sku`, `available_qty`, `reserved_qty`, `warehouse_cd`, lower-case → `inventory-api/sql/V1__stock.sql`
- inventory-api documents the crossing explicitly, with a translation table in its README mapping `SANGPUM_CD` to `sku` and `SURYANG` to `available_qty`/`reserved_qty` → `inventory-api/README.md`
- The romanisation is not consistent with itself, and both variants can be real: 택배사 (carrier) is `TAEKBAESA_CD` on `ORDER_MST` and `TAKBAE_CD` on `ORDER_DELIVERY`, two columns created by the same V1 file, and `taekBaeSaCd` again in the API spec → `order-service/src/main/resources/db/migration/V1__init.sql`, `order-service/src/main/resources/db/migration/V5__add_delivery_columns.sql`, `sellflow-docs:raw/specs/order-service-openapi.json`
- 채널 (channel) takes a third form per surface: `CHAENNEL_CD` in the migration, `chaenNelCd` in the API spec, and `CHNL_CD` — an abbreviation, with a different value set — on `ORDER_CANCEL` → `order-service/src/main/resources/db/migration/V3__add_channel_code.sql`, `order-service/src/main/resources/db/migration/V17__add_cancel_channel.sql`
- The boundary can run *inside a single table*: inventory-api's `RESTORE_LOG` carries `ORD_NO`, `SURYANG` and `SAYU_CD` from the order domain's romanised vocabulary alongside `SKU` from its own English one → `inventory-api/alembic/versions/20230414_1120-3f9a_add_restore_log.py`
- The API specification propagates the romanised names outward to every client, so the convention is not merely internal → `sellflow-docs:raw/specs/order-service-openapi.json`

## Where It Appears

| Repo | Convention | Example | Built / transferred |
|---|---|---|---|
| order-service | romanised Korean, UPPER_SNAKE | `CHWISO_SAYU_CD` | 2019, 주문팀 |
| settlement-batch | romanised Korean, UPPER_SNAKE | `JUNGSAN_ILJA` | 2019-, 정산팀 |
| settlement-anomaly | English table and column names, Korean business codes | `SETTLEMENT_ANOMALY`, `ANOMALY_CD` | 2025, 데이터팀 |
| inventory-api | English, lower_snake | `available_qty` | 2022 transfer, 재고팀 |
| delivery-bff | English types, romanised fields from the generated client | `DeliveryStatus.carrierCd` | 물류팀 |

The mixed cases are the ones to watch. settlement-anomaly names its own table in English but reads romanised columns from the shared database, so a single query in `app/main.py` spans both conventions. delivery-bff defines an English interface whose field names are romanised because they were generated from the order spec.

## Design Intent

Not recoverable from the sources. No naming standard document exists in this reef. The most that can be said is that the convention tracks the era and the division: the 2019 commerce and finance teams romanised, the data-platform teams from 2022 onward did not, and nobody appears to have reconciled the two — inventory-api's README documents the mapping rather than proposing a migration, which suggests coexistence was accepted rather than decided.

## Trade-offs

What it buys: a direct correspondence between the word a business user says in a meeting and the identifier in the schema. `정산 정정` in the procedure document is `JUNGSAN` and `RECON` in the code, and 김도윤 in a Slack thread and `CANCEL_RECON_QUEUE` in a migration are recognisably the same subject. For a team that works entirely in Korean, that is real value.

What it costs: the identifiers are opaque to anyone who does not read Korean, including tooling. Romanisation without a scheme produces variants of the same word (`CHAENNEL`/`chaenNel`/`CHNL`, `TAEKBAESA`/`TAKBAE`/`taekBaeSa`), so identifiers cannot be reliably matched by string similarity — and because both variants of a pair can be real columns on real tables, a near-miss is not evidence of a mistake. And because the convention stops at a service boundary, the same quantity carries two unrelated names on either side of a call — which is precisely the boundary where [[CON-ORDER-INVENTORY]] is broken.

## Agent Guidance

- Never translate a romanised identifier into English and use the translation as a name. `SURYANG` is not `quantity` anywhere in the code; searching for `quantity` in order-service finds nothing.
- Never assume two similar identifiers are the same field. `SANGPUM_CD` (order) and `sku` (inventory) hold the same value; `SANGPUM_CD` (order) and `sku_cd` (the unused inventory dataclass) do not appear together in any live path.
- When reading a query that touches both conventions, check which database the table belongs to before trusting a join. Four services share one instance, so a romanised and an English table name in the same SQL statement is normal here, not a mistake.
- Enum constants carry their Korean label as a separate field. `OrderStatus.JUNGSAN_WANRYO.getLabel()` returns 정산완료; the constant name is the persisted value and the label is for display. Do not persist the label.
- When adding a column to order-service or settlement-batch, follow the local convention rather than importing English naming. Mixed naming inside one table would be worse than either convention alone, and the V15/V16 rename-and-revert shows how expensive a rename is under a shared database.

## Related

- [[GLOSSARY-SELLFLOW]] -- the term registry this pattern makes necessary
- [[SCH-ORDER]] -- the canonical romanised schema
- [[SCH-SETTLEMENT-BATCH]] -- the same convention in the finance domain
- [[SCH-INVENTORY]] -- where the convention stops
- [[CON-ORDER-INVENTORY]] -- the boundary the vocabulary split runs along
