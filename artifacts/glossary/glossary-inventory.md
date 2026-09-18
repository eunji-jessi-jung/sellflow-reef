---
id: "GLOSSARY-INVENTORY"
type: "glossary"
title: "Inventory Domain Glossary"
domain: "inventory"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by reading every one of the 16 files in inventory-api line by line — README.md, requirements.txt, .env.sample, Dockerfile, alembic.ini, alembic/env.py, both revisions under alembic/versions/, app/config.py, app/db.py, app/main.py, app/models.py, app/routers.py, sql/V1__stock.sql and both test modules — and cross-checking every shared term against order-service's CancelReason enum, InventoryClient and V1 DDL, the 취소정책 sheet of business-rules, the 2021 Confluence cancellation policy, the org chart and the service registry. Every identifier, column, status literal, reason code, environment variable and Korean phrase in the repository is now listed, including the ones that are referenced but never defined. Re-read on 2026-09-19 after a correction pass in inventory-api: RESTORE_LOG now has six named columns, so five identifiers that were previously undefined (LOG_SEQ, ORD_NO, SKU, SURYANG, SAYU_CD, REG_DTM as used on that table) are now real terms, and revisions 3f9a and 8ba1 have definite effects. Goes stale if sql/V1__stock.sql changes, if either declaration of RESTOCKABLE_REASONS moves or changes, if a third alembic revision lands, if app/routers.py is mounted, or if the README's terminology table is edited."
freshness_triggers:
  - ".env.sample"
  - "README.md"
  - "alembic.ini"
  - "alembic/env.py"
  - "alembic/versions/20230414_1120-3f9a_add_restore_log.py"
  - "alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
  - "app/config.py"
  - "app/db.py"
  - "app/main.py"
  - "app/models.py"
  - "app/routers.py"
  - "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
  - "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
  - "sellflow-docs:context/business-rules.md"
  - "sql/V1__stock.sql"
  - "tests/test_restore.py"
known_unknowns:
  - "The authoritative Korean business term for each English column name. inventory-api is bilingual by accident — Korean comments and log messages, English columns — and unlike order-service it carries no Korean column names at all, so there is nothing to romanise back. No data dictionary exists in sources/."
  - "What reserved_qty reserves, and for whom. It is declared NOT NULL DEFAULT 0 in sql/V1__stock.sql and returned by GET /stock/{sku}, and no code in any of the five repos writes it. The reservation lifecycle — who reserves, when, and what releases a reservation — is defined nowhere. Checked by grepping all five repos for reserved_qty: the DDL, the SELECT literal in app/main.py and the extracted schema are the only hits."
  - "The domain meaning and legal values of warehouse_cd. No lookup table, enum, seed file or constraint exists on either side of the boundary. The only warehouse literal anywhere in the estate is 'GIMPO', the DEFAULT on order-service's ORDER_DTL.CHANGGO_CD, and the 용인센터 (Yongin centre) named in that migration's comment has no code of its own."
  - "Whether the four reason codes reaching inventory-api are guaranteed to be the CancelReason codes. inventory-api declares no enum, performs no validation and has no import or shared artifact linking it to order-service; the equivalence rests on two source comments and the 취소정책 sheet. An unknown code such as '99' is answered with HTTP 200 and {\"restocked\": false, \"reason\": \"not_restockable\"}, whereas order-service's CancelReason.of throws IllegalArgumentException(\"알 수 없는 취소 사유 코드: \" + code). The two services therefore treat an invalid reason code as two different classes of event."
  - "Whether sayu_cd (the unused RestoreLog dataclass), reason_code (the live RestockRequest model) and reason (the query parameter InventoryClient sends) are meant to be one concept under three names, or whether one of them was intended to carry something else. Nothing in either repo states the equivalence."
  - "What the RestoreLog.result field was meant to hold. No value vocabulary for it exists — no enum, no constant, no comment, and no code ever constructs a RestoreLog."
  - "Whether 'restock' and 재고 복원 are interchangeable in business usage, or whether 재고 복원 is narrower (cancellation-driven only). The code uses the English word in the route path and response key and the Korean phrase in the docstring, the log lines, the README, the org chart and §5 of the 2021 policy — never both in the same sentence."
  - "Whether the order domain's SURYANG maps onto available_qty, reserved_qty or their sum. The README's bridge row lists both with a slash and does not disambiguate; the restock handler adds it to available_qty only, which is behaviour rather than a definition."
  - "Why config.py's gloss spells the reason labels without spaces — 파트너귀책, 시스템오류 — while order-service's CancelReason spells them 파트너 귀책, 시스템 오류 and the 취소정책 sheet agrees with order-service. Nothing records whether this is a typo or a separate inventory-side vocabulary."
  - "Whether DB_URL, LOG_LEVEL and the alembic.ini sqlalchemy.url name a database that still exists. All three point at a database called inventory; app/db.py ignores all three and hardcodes database=\"sellflow_order\". No code in the repo ever opens the inventory database."
tags:
  - "inventory"
  - "glossary"
  - "korean"
  - "romanisation"
  - "sellflow"
aliases:
  - "inventory terms"
  - "재고 용어"
relates_to:
  - type: "refines"
    target: "[[API-INVENTORY]]"
  - type: "refines"
    target: "[[CON-ORDER-INVENTORY]]"
  - type: "refines"
    target: "[[DEC-INVENTORY-RESTOCK-BY-REASON]]"
  - type: "integrates_with"
    target: "[[GLOSSARY-ORDER]]"
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PAT-SELLFLOW-ROMANISED-NAMING]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTOCK]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]]"
  - type: "feeds"
    target: "[[RISK-INVENTORY]]"
  - type: "refines"
    target: "[[SCH-INVENTORY]]"
  - type: "parent"
    target: "[[SYS-INVENTORY]]"
  - type: "integrates_with"
    target: "[[SYS-ORDER]]"
sources:
  - category: "implementation"
    type: "github"
    ref: "inventory-api:.env.sample"
    notes: "DB_URL and LOG_LEVEL, neither of which app/db.py reads."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:Dockerfile"
    notes: "Python 3.9 base image and the uvicorn entrypoint naming app.main:app."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:README.md"
    notes: "The 주의 section and the three-row vocabulary bridge table, quoted verbatim below."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic.ini"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/env.py"
    notes: "Two lines; imports context and nothing else."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20230414_1120-3f9a_add_restore_log.py"
    notes: "Base revision; creates RESTORE_LOG with six columns and IX_RESTORE_LOG_01 on ORD_NO."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py"
    notes: "Creates IX_RESTORE_LOG_02 on SAYU_CD."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/config.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/db.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/models.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:app/routers.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:requirements.txt"
    notes: "Five pins: fastapi, uvicorn, pymysql, sqlalchemy, alembic."
  - category: "implementation"
    type: "github"
    ref: "inventory-api:sql/V1__stock.sql"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_health.py"
  - category: "implementation"
    type: "github"
    ref: "inventory-api:tests/test_restore.py"
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java"
    notes: "Source of the ordNo / sayuCd / reason spellings."
  - category: "implementation"
    type: "github"
    ref: "order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java"
    notes: "The four reason codes and their verbatim Korean labels."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "Sheet 취소정책 — the 재고 복원 column per reason code. Reef-root relative."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/org-chart.md"
    notes: "재고팀 owns inventory-api and 재고 복원 as a named process."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
    notes: "Records inventory-api's db as MySQL (inventory), which no code opens."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html"
    notes: "§5 재고 복원 — the policy hands the restock decision to inventory-api and 재고팀."
notes: "Korean text is quoted verbatim from the repository's own comments, docstrings and log strings, or from the source documents, with an English rendering alongside. No Korean here is a back-translation of English."
---

# Inventory Domain Glossary

## Overview

The inventory domain speaks a different language from the order domain, and unusually, the
repository says so out loud — three times, in three different files. `README.md` carries a
section headed 주의 ("caution") whose first line is 주문 도메인과 용어 체계가 다르다
("the terminology system differs from the order domain"), followed by a three-row mapping table.
The DDL repeats it — 재고 스키마. 주문 도메인과 용어가 다르다 (SANGPUM_CD ↔ sku)
("inventory schema; the vocabulary differs from the order domain") — and so does the module
docstring of `app/main.py`: order-service 와 용어가 다르다. 주문 도메인의 SANGPUM_CD 는 여기서
sku 다 ("the vocabulary differs from order-service; the order domain's SANGPUM_CD is sku here").

That warning is the single most useful sentence in the repository, and it also understates the
problem. The order domain names everything in romanised Korean, so its glossary is a
transliteration exercise. Inventory does the opposite: its **columns are plain English**
(`sku`, `available_qty`, `reserved_qty`, `warehouse_cd`, `updated_at`), while its **comments,
docstrings and log messages are Korean**. There is therefore almost nothing here to romanise —
the one exception is `sayu_cd` (사유 코드, reason code), which survives on an unused dataclass.
See [[PAT-SELLFLOW-ROMANISED-NAMING]] for the estate-wide convention this repository opts out of.

Quoting the README's bridge table verbatim, with the English meanings added:

| 주문 도메인 (order domain) | 재고 도메인 (inventory domain) |
|---|---|
| `SANGPUM_CD` | `sku` |
| `SURYANG` | `available_qty` / `reserved_qty` |
| `ORD_NO` | (보관하지 않음) |

보관하지 않음 means "not stored" — the inventory domain has no counterpart for the order number
at all. That third row is the most consequential of the three. `ord_no` does travel into the
service, as a field of the restock request and as the bind parameter of the `ORDER_DTL` lookup,
but it is never persisted: it appears in two log lines and nowhere else, because `RESTORE_LOG` —
the one table that would hold it, and which has an `ORD_NO` column and an index on it — is
written by nothing
(`app/main.py`, `alembic/versions/20230414_1120-3f9a_add_restore_log.py`). The vocabulary gap and
the missing audit trail are the same fact seen twice; see
[[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]].

A second layer of divergence sits *inside* this repository, between the live SQL literals and the
dataclasses in `app/models.py`. `sku` versus `sku_cd`, `available_qty` versus `qty`,
`reason_code` versus `sayu_cd`: each concept has a shipped name and a shadow name, and the shadow
names are the ones an agent will meet first if it opens `models.py` expecting an ORM. Terms
marked **unused** below are the shadow names. Nothing in the repository is deprecated, imported
across, or otherwise marked as the loser.

## Terms

### `stock_item` — the one live table

Created by `sql/V1__stock.sql`, a Flyway-style file that is not under Alembic control; no Alembic
revision creates it and no Alembic revision knows it exists.

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `stock_item` | 재고 — stock | The only table any running code touches. `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4`, `PRIMARY KEY (sku, warehouse_cd)`. Lower-case and English, against an estate where every other table name is upper-case romanised Korean → `sql/V1__stock.sql` | [[SCH-INVENTORY]] |
| `sku` | — (no Korean original; English initialism) | Stock-keeping unit code, `VARCHAR(30) NOT NULL`, first component of the primary key. The inventory domain's name for what the order domain calls `SANGPUM_CD`, stated as such in three files. **Collides across the boundary:** the same physical value carries a different identifier name on each side → `sql/V1__stock.sql`, `README.md`, `app/main.py` | [[GLOSSARY-ORDER]], [[SYS-ORDER]] |
| `warehouse_cd` | 창고 코드 — warehouse code | `VARCHAR(10) NOT NULL`, second component of the primary key. A term the schema takes seriously and the code does not: the restock `UPDATE` omits it from its `WHERE` clause, so a restock touches every warehouse row for that SKU, and `GET /stock/{sku}` returns an arbitrary single row via `fetch_one`. `app/models.py` has no warehouse field at all, its docstring claiming 창고별 재고는 별도 시스템 ("per-warehouse stock is a separate system") → `sql/V1__stock.sql`, `app/main.py`, `app/models.py` | [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]] |
| `available_qty` | — | Sellable quantity, `INT NOT NULL DEFAULT 0`. The only stock column any code path writes, and it is only ever incremented: `SET available_qty = available_qty + %(q)s`. No decrement exists in any of the five repos, so the restock increment currently has no matching consumption → `sql/V1__stock.sql`, `app/main.py` | [[PROC-INVENTORY-RESTOCK]] |
| `reserved_qty` | — | Held-back quantity, `INT NOT NULL DEFAULT 0`. Returned by `GET /stock/{sku}` and written by nothing, anywhere. The word "reserved" is the entire specification; no code, comment or document says what a reservation is, who creates one, or what releases it → `sql/V1__stock.sql`, `app/main.py` | [[API-INVENTORY]], [[RISK-INVENTORY]] |
| `updated_at` | — | `DATETIME DEFAULT CURRENT_TIMESTAMP` with **no `ON UPDATE` clause**, so despite its name it records row creation time and stays frozen while `available_qty` moves. A false friend of the same-named columns elsewhere → `sql/V1__stock.sql` | [[SCH-INVENTORY]] |

### Restock — the endpoint, the rule, and the vocabulary around it

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| restock / 재고 복원 | 재고 복원 — stock restoration | Returning cancelled units to sellable stock. English in the route (`POST /stock/restock`) and the response key (`{"restocked": ...}`); Korean everywhere a human reads — the handler docstring 주문 취소에 따른 재고 복원 ("stock restoration following order cancellation"), the success log 재고 복원 완료 ("stock restoration complete"), the README, the org chart's process column for 재고팀, and §5 of the 2021 policy page → `app/main.py`, `sellflow-docs:context/org-chart.md`, `sellflow-docs:raw/confluence-snapshots/주문-취소-정책_48213.html` | [[PROC-INVENTORY-RESTOCK]] |
| `RESTOCKABLE_REASONS` | 복원 대상 사유코드 — restock-target reason codes | The allow-list `{"01", "02"}`. **Declared twice, in two files, with two different comments, and never imported across.** `app/main.py:18` governs behaviour; `app/config.py:5` is what the tests import → `app/main.py`, `app/config.py`, `tests/test_restore.py` | [[DEC-INVENTORY-RESTOCK-BY-REASON]] |
| `RestockRequest` | — | The live Pydantic request model: exactly two `str` fields, `ord_no` and `reason_code`. No quantity, no SKU, no warehouse, no line number — everything else is looked up from `ORDER_DTL` → `app/main.py` | [[API-INVENTORY]] |
| `reason_code` | 사유 코드 — reason code | The cancellation reason on `RestockRequest`. A bare `str`: no enum, no length, no pattern, no validation. Unknown values are not errors; they simply fail the allow-list test → `app/main.py` | [[CON-ORDER-INVENTORY]] |
| `restocked` | — | The boolean response key, present on both branches. `true` means the `UPDATE` was issued, not that a row was matched — `execute` returns the affected row count and the handler discards it → `app/main.py` | [[API-INVENTORY]] |
| `not_restockable` | — | The only string literal the service ever puts in a `reason` response field, returned with HTTP **200** alongside `{"restocked": false}`. There is no second value and no error code; a policy refusal and an unrecognised code are the same response → `app/main.py` | [[API-INVENTORY]] |
| 복원 대상 아님 | "not a restock target" | The log line emitted on the refusal branch, at `INFO`, carrying `ord_no` and `reason`. The only record that a refusal happened — nothing is persisted → `app/main.py` | [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] |
| 주문 상세 없음 | "no order detail" | The HTTP 404 detail raised when the `ORDER_DTL` lookup returns nothing → `app/main.py` | [[API-INVENTORY]] |
| SKU 없음 | "no such SKU" | The HTTP 404 detail raised by `GET /stock/{sku}` when no `stock_item` row matches → `app/main.py` | [[API-INVENTORY]] |

### Cancel reason codes as inventory sees them

inventory-api holds no enum. The codes exist here as two bare strings in a set and a one-line
comment; the constants, labels and validation live one repository away. Labels are quoted
verbatim from order-service's `CancelReason`; the restock column is from the 취소정책 sheet.

| Code | order-service constant | Label (verbatim) | English | config.py gloss (verbatim) | 취소정책 sheet 재고 복원 | Restocked here? |
|---|---|---|---|---|---|---|
| `01` | `PARTNER_GWICHAEK` | "파트너 귀책" | partner at fault | 파트너귀책 *(no space)* | O | yes |
| `02` | `SYSTEM_ORYU` | "시스템 오류" | system error | 시스템오류 *(no space)* | O | yes |
| `03` | `GOGAEK_BYEONSIM` | "고객 변심" | customer changed their mind | — | X | no |
| `04` | `BAESONG_SILPAE` | "배송 실패" | delivery failure | — | **O** | **no** |

Two facts about this table are worth stating separately, because they are the reason the table is
here at all.

**The exclusion of 03 is explained; the exclusion of 04 is not.** The handler's comment argues
only about 03 — 03(고객 변심)은 배송이 시작된 뒤 취소되는 경우가 많아 복원 대상이 아니다
("03, customer change of mind, is not a restock target because such cancellations often arrive
after shipping has started") — and the 취소정책 sheet agrees with it, noting 배송 시작 후 취소 시
재고 복원 불가 ("stock cannot be restored when cancelled after shipping has started"). For 04 the
sheet says O and the code says no, and nothing in either repository, the policy page or the sheet
gives a motive. The full treatment is [[DEC-INVENTORY-RESTOCK-BY-REASON]]
→ `app/main.py`, `sellflow-docs:context/business-rules.md`

**The two services disagree on what an invalid code is.** order-service's `CancelReason.of`
throws `IllegalArgumentException("알 수 없는 취소 사유 코드: " + code)` ("unknown cancel reason
code"). inventory-api performs no membership check against the four legal codes at all — only
against the two restockable ones — so `"99"`, `""` and `"4"` are all answered with HTTP 200 and
`not_restockable`, indistinguishable from a correctly-refused 03
→ `order-service:src/main/java/kr/co/sellflow/order/domain/CancelReason.java`, `app/main.py`

### Order-domain terms that appear inside inventory code

These are not inventory terms. They are here because the restock handler reads the order schema
directly, over inventory-api's own connection — see [[CON-ORDER-INVENTORY]] and the shared-database
arrangement behind it.

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `ORDER_DTL` | 주문 상세 — order detail | An order-service table, `SELECT SANGPUM_CD, SURYANG FROM ORDER_DTL WHERE ORD_NO = %(o)s`. Read with `fetch_one`, so a multi-line cancelled order restocks at most one line → `app/main.py` | [[SYS-ORDER]] |
| `SANGPUM_CD` | 상품 코드 — product code | The order domain's product code, read out of `ORDER_DTL` and used as the value bound to `stock_item.sku`. The rename happens here, in a dict lookup, with no mapping layer → `app/main.py` | [[GLOSSARY-ORDER]] |
| `SURYANG` | 수량 — quantity | The order domain's line quantity, added to `available_qty`. The README maps it to `available_qty` / `reserved_qty` without saying which; the handler settles the question in practice and not in writing → `app/main.py`, `README.md` | [[GLOSSARY-ORDER]] |
| `ORD_NO` / `ord_no` | 주문번호 — order number | The order key. `ORD_NO` in the SQL, `ord_no` on the request model and the unused dataclass, `ordNo` in the caller's query string. Not stored by inventory in any form — the README's third bridge row says 보관하지 않음, and no inventory table has such a column → `app/main.py`, `README.md`, `sql/V1__stock.sql` | [[GLOSSARY-SELLFLOW]] |
| `sellflow_order` | — | The database name hardcoded in `app/db.py`'s DSN. The inventory service's live connection is to the order team's database; the `inventory` database named in `.env.sample`, `alembic.ini` and the service registry is opened by nothing → `app/db.py`, `.env.sample`, `alembic.ini`, `sellflow-docs:context/registry/services.yaml` | [[SYS-INVENTORY]] |

### `RESTORE_LOG` — the audit trail that is a name and nothing else

| Term | Korean / meaning | Definition | See Also |
|---|---|---|---|
| `RESTORE_LOG` | 복원 이력 — restore history | The restore audit table. Created by Alembic revision `3f9a` (2023-04-14) with six columns — `LOG_SEQ`, `ORD_NO`, `SKU`, `SURYANG`, `SAYU_CD`, `REG_DTM` — and indexed on `ORD_NO`; **never read or written by any code in any of the five repos**. The one upper-case table name in the repository, matching the estate convention that `stock_item` breaks → `alembic/versions/20230414_1120-3f9a_add_restore_log.py` | [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] |
| `LOG_SEQ` | — | `RESTORE_LOG`'s surrogate key: `BigInteger`, autoincrement. The only autoincrement key in this service; `stock_item` uses a natural composite key instead → `alembic/versions/20230414_1120-3f9a_add_restore_log.py` | [[SCH-INVENTORY]] |
| `SKU` | — | `RESTORE_LOG`'s product column, upper-cased. The same concept as `stock_item.sku` and the order domain's `SANGPUM_CD`; this is the only place the inventory spelling appears in upper case → `alembic/versions/20230414_1120-3f9a_add_restore_log.py` | [[SCH-INVENTORY]] |
| `SURYANG` | 수량 — quantity | `RESTORE_LOG`'s quantity column. Borrowed from the order domain rather than from this service's own `available_qty`/`qty` vocabulary — one table, two naming systems → `alembic/versions/20230414_1120-3f9a_add_restore_log.py`, `order-service:src/main/resources/db/migration/V1__init.sql` | [[GLOSSARY-ORDER]] |
| `SAYU_CD` | 사유 코드 — reason code | `RESTORE_LOG`'s cancel-reason column, `String(2)`, holding the same `01`–`04` vocabulary as `CHWISO_SAYU_CD` and `RestockRequest.reason_code`. Indexed by revision `8ba1` → `alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py` | [[PROC-INVENTORY-RESTOCK]] |
| `RestoreLog` — **unused** | — | The dataclass sketching what a restore-log row was meant to hold: `ord_no`, `sku_cd`, `qty`, `sayu_cd`, `result`. Never instantiated, never imported. The only evidence anywhere of the table's intended shape → `app/models.py` | [[SCH-INVENTORY]] |
| `sayu_cd` — **unused** | 사유 코드 — reason code | The romanised-Korean field name `RestoreLog` uses for what the live model calls `reason_code`. The single romanised-Korean identifier in the entire inventory codebase. order-service's client names the same value `sayuCd` in Java and sends it as `reason` on the wire — so one concept carries four names across the boundary → `app/models.py`, `order-service:src/main/java/kr/co/sellflow/order/client/InventoryClient.java` | [[PAT-SELLFLOW-ROMANISED-NAMING]] |
| `result` — **unused** | — | A `RestoreLog` field with no defined values. No enum, constant, comment or code anywhere suggests what would be written into it → `app/models.py` | [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] |
| `3f9a` | — | The base revision (`down_revision = None`) that creates `RESTORE_LOG` and `IX_RESTORE_LOG_01`. Its docstring records why the chain starts here: "stock_item 은 alembic 도입 이전에 sql/V1__stock.sql 로 만들었다" ("stock_item was made with sql/V1__stock.sql before Alembic was introduced") → `alembic/versions/20230414_1120-3f9a_add_restore_log.py` | [[SCH-INVENTORY]] |
| `8ba1` | — | Revision "add restore log reason index" (2024-09-02, title truncated by the filename). Creates `IX_RESTORE_LOG_02` on `SAYU_CD` and drops it on downgrade → `alembic/versions/20240902_0931-8ba1_add_restore_log_reason_i.py` | [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] |

### Shadow names — identifiers that exist in code and match no live column or route

| Term | Definition | See Also |
|---|---|---|
| `Stock` — **unused** | The dataclass `Stock(sku_cd, qty, updated_at)`. Only `updated_at` matches a real column; `sku_cd` and `qty` match nothing. Not imported anywhere → `app/models.py`, `sql/V1__stock.sql` | [[SCH-INVENTORY]] |
| `sku_cd` — **unused** | The shadow name for `sku`, used by both dataclasses, by the unmounted router's path parameter, and by that router's `skuCd` response key. No live column, request field or response field uses it → `app/models.py`, `app/routers.py` | [[API-INVENTORY]] |
| `qty` — **unused** | The shadow name for quantity, in both dataclasses and in the unmounted router's hardcoded `{"skuCd": ..., "qty": 0}`. The live schema has no `qty` column — only `available_qty` and `reserved_qty` — and no live response returns one → `app/models.py`, `app/routers.py`, `sql/V1__stock.sql` | [[SCH-INVENTORY]] |
| `router` / `/inventory` prefix — **unmounted** | The `APIRouter(prefix="/inventory", tags=["inventory"])` in `app/routers.py`, declaring `/inventory/stock/{sku_cd}` (a stub returning `qty: 0`) and `/inventory/health`. `app/main.py` never calls `include_router`, so both return 404 in the running service — and `/inventory/restore`, the path order-service's client posts to, is not declared even here → `app/routers.py`, `app/main.py` | [[CON-ORDER-INVENTORY]] |
| `status` / `"ok"` | The health stub's only response, `{"status": "ok"}`, and the only assertion in `tests/test_health.py`. The test imports the function directly rather than exercising a route, so it passes while the endpoint is unreachable → `app/routers.py`, `tests/test_health.py` | [[RISK-INVENTORY]] |

### Infrastructure and configuration vocabulary

| Term | Definition | See Also |
|---|---|---|
| `DSN` | The connection dict in `app/db.py`: `DB_HOST` (default `localhost`), `DB_USER` (default `sellflow`), `database` hardcoded to `"sellflow_order"`, `charset` `utf8mb4`, `cursorclass` `DictCursor`. No password key at all → `app/db.py` | [[SYS-INVENTORY]] |
| `DB_HOST`, `DB_USER` | The only environment variables the running code reads. Neither appears in `.env.sample` → `app/db.py`, `.env.sample` | [[SYS-INVENTORY]] |
| `DB_URL` | `mysql://inventory:@localhost:3306/inventory` in both `.env.sample` and `app/config.py`, and the same string as `sqlalchemy.url` in `alembic.ini`. Read into a module constant and used by nothing → `.env.sample`, `app/config.py`, `alembic.ini` | [[SYS-INVENTORY]] |
| `LOG_LEVEL` | Declared in `.env.sample` and read into `app/config.py` with default `INFO`. Never passed to `logging` — `app/main.py` calls `logging.getLogger(__name__)` and no `basicConfig` or `dictConfig` exists → `app/config.py`, `app/main.py` | [[RISK-INVENTORY]] |
| `fetch_one` / `execute` | The two data-access helpers in `app/db.py`. Both open a fresh `pymysql.connect` per call — there is no pool — and both interpolate via `%(name)s` bind parameters. `execute` commits and returns the affected row count, which no caller reads → `app/db.py` | [[SCH-INVENTORY]] |
| `3.2.1` | The version string on `FastAPI(title="inventory-api", version="3.2.1")`, and the only version marker in the repository. Nothing else — no tag file, no changelog, no packaging metadata — corroborates it → `app/main.py` | [[SYS-INVENTORY]] |
| 재고팀 / 데이터플랫폼본부 | "inventory team" / "data platform division". The owning team and its division. The README names them with the transfer year — 담당: 데이터플랫폼본부 재고팀 (2022년 신설 시 이관), "owner: data platform division, inventory team (transferred on its founding in 2022)" — and the org chart records the move on 2022-03-01 (커머스본부 → 데이터플랫폼본부) and gives 재고팀 both 재고 관리 and 재고 복원 as named processes → `README.md`, `sellflow-docs:context/org-chart.md` | [[SYS-INVENTORY]] |

### Terms that collide with the order domain

Five entries above are genuine cross-boundary hazards. A reader moving between the two
repositories will meet all five, and only the first is signposted in either codebase.

- **`sku` ↔ `SANGPUM_CD`** — the same product identifier, renamed at the boundary. Stated
  explicitly in three inventory files. Note that inventory-api is not consistent with itself
  either: `stock_item` spells it `sku` and `RESTORE_LOG` spells it `SKU`.
- **`SURYANG` → `available_qty` / `reserved_qty`** — one order-side quantity term facing two
  inventory-side columns, with the README declining to say which.
- **`warehouse_cd` ↔ `CHANGGO_CD`** — 창고 코드 under two spellings. Half the inventory primary
  key; a defaulted column (`'GIMPO'`) on the order side; constrained by nothing on either.
- **`ord_no` / `ORD_NO` / `ordNo`** — three spellings across the request body, the order table and
  the caller's query string, for a value the inventory domain then does not keep.
- **`reason_code` / `sayu_cd` / `sayuCd` / `reason`** — the cancellation reason under four names
  across the live request model, the unused dataclass, the Java client's parameter and the query
  string that client actually sends. Validated on the order side only.

Cross-service disambiguation belongs to [[GLOSSARY-SELLFLOW]]; this artifact records the
inventory side of each pair, and [[GLOSSARY-ORDER]] records the other.

## Related

- [[API-INVENTORY]] — where `reason_code`, `ord_no`, `restocked` and `not_restockable` appear in request and response shapes
- [[CON-ORDER-INVENTORY]] — the boundary at which `SANGPUM_CD` becomes `sku`, and the client that never calls it
- [[DEC-INVENTORY-RESTOCK-BY-REASON]] — `RESTOCKABLE_REASONS` as a decision, and its divergence from the 취소정책 sheet on code 04
- [[GLOSSARY-ORDER]] — the vocabulary on the other side of the bridge table
- [[GLOSSARY-SELLFLOW]] — the cross-service disambiguation registry
- [[PAT-SELLFLOW-ROMANISED-NAMING]] — the estate convention this repository is the exception to
- [[PROC-INVENTORY-RESTOCK]] — 재고 복원 in motion
- [[PROC-INVENTORY-RESTORE-LOG-LIFECYCLE]] — `RESTORE_LOG`, revisions `3f9a` and `8ba1`, and the write that was never written
- [[PROC-INVENTORY-STOCK-ITEM-LIFECYCLE]] — `available_qty`, `reserved_qty` and `warehouse_cd` over a row's life
- [[RISK-INVENTORY]] — the shadow names, the unmounted router and the unread configuration as risks
- [[SCH-INVENTORY]] — the columns behind these terms
- [[SYS-INVENTORY]] — the owning service and team
- [[SYS-ORDER]] — the domain on the other side of the vocabulary bridge
