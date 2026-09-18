---
id: "GLOSSARY-SETTLEMENT"
type: "glossary"
title: "Settlement Domain Glossary"
domain: "settlement"
status: "draft"
last_verified: 2026-09-19
freshness_note: "Deepened on 2026-09-19 by reading all six settlement-batch migrations, the batch job and relay classes, settlement-anomaly's detector, schema and API, both correction procedures (v0.3 and v1.1), the 2025-03 handover, the finance mail thread and the queue export. The 정정/정산 pair and the two RUN_ID types are now stated explicitly. Stale if a migration changes a column, if a third anomaly code is implemented, or if a revised correction procedure supersedes v1.1."
freshness_triggers:
  - "app/schemas.py"
  - "model/detector.py"
  - "sources/context/policy/정산_정정_업무절차_v1.1.md"
  - "sources/raw/exports/cancel_recon_queue_monthly_20260901.csv"
  - "sql/V1__anomaly_schema.sql"
  - "src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - "src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - "src/main/resources/db/migration/V1__settlement_schema.sql"
  - "src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
known_unknowns:
  - "The full value set of SETTLEMENT_RUN.SANGTAE. Only 'RUNNING' appears in the repository, in the writer's MAX(RUN_ID) subquery. No code in settlement-batch inserts into SETTLEMENT_RUN at all, so nothing observed here sets or clears that status."
  - "Whether ADJ_TYPE has values other than CANCEL_CLAWBACK. Only that literal appears, in a single INSERT in CancelReconciler, and the target table is created by no migration."
  - "Whether 지급 요청 (payment request) is a distinct artefact or simply the SETTLEMENT_DTL row. The writer inserts a row and logs; the javadoc and the handover both speak of a transmission that cannot be recalled, and the transmitting code is in no repo in this reef."
  - "Official English terminology for the Korean business terms. The only bilingual source in the estate is procedure v1.1 §6, which names the two forms in both languages; every other English gloss in this artifact is a translation made for the reef."
  - "Which of PROCESSED_DTM and PROCESSED_AT the correction procedure intends. V1 creates PROCESSED_DTM, V5 adds PROCESSED_AT for the same purpose, CancelReconciler writes PROCESSED_DTM, and V5's own comment says no code uses the new columns."
  - "What the 4,127 queue rows are worth. The export README records the query as SUM(EXPECTED_AMT), a column none of the queue's three migrations creates, and every CSV row is exactly 건수 × 45,760 — so 188,851,520 KRW is a derived figure, not a stored one."
  - "Whether SETTLEMENT_ANOMALY.STATUS ever leaves 'DETECTED'. The column defaults to it, the INSERT writes it explicitly, and REVIEWED_BY / REVIEWED_DTM are written by nothing. The registry records that no team owns the response."
  - "What the 재무기획팀 (finance planning team) means by 정정 in the accounting sense, versus the operational sense 정산팀 uses. The quarterly-close mail asks whether the balance is a liability; no document in this reef answers."
tags:
  - settlement
  - glossary
  - korean-terminology
  - correction
aliases:
  - "정산 용어"
  - "settlement terms"
relates_to:
  - type: "refines"
    target: "[[GLOSSARY-SELLFLOW]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-DAILY-BATCH]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT-RECON-BACKLOG]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-ANOMALY]]"
  - type: "refines"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "취소정책 sheet — SAYU_CD values and 정산 차감 rules; 수수료 sheet — the three rate tiers."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "The operational meaning of 차월 차감 and the unqueryable SETTLEMENT_ADJUSTMENT."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v0.3.md"
    notes: "§5 — the 월 10건 미만 volume estimate."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/policy/정산_정정_업무절차_v1.1.md"
    notes: "§2 definitions, §3 responsibilities (미정정 잔액), §6 form register."
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/registry/services.yaml"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/README.md"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:raw/exports/cancel_recon_queue_monthly_20260901.csv"
  - category: "discussion"
    type: "doc"
    ref: "sellflow-docs:raw/mail/RE_정산_미정정_금액_문의.eml"
    notes: "The 미정정 금액 question from 재무기획팀."
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/main.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:app/schemas.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:model/detector.py"
  - category: "implementation"
    type: "github"
    ref: "settlement-anomaly:sql/V1__anomaly_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V1__settlement_schema.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V2__add_settlement_run_log.sql"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/resources/db/migration/V5__add_recon_processed_columns.sql"
notes: "Korean terms are quoted verbatim with an English gloss alongside. The glosses are translations made for this artifact, not official company terminology — except ST-F-001 and ST-F-002, whose English names come from procedure v1.1 §6."
---

# Settlement Domain Glossary

## Overview

The settlement codebase names its columns in romanised Korean — `JUNGSAN_ILJA`, `SUSURYO`, `SAYU_CD` — while the company documents use hangul and neither provides an English equivalent. This registry pairs the three, so a reader can move between a Slack message, a procedure clause and a SQL column without guessing.

One pair deserves its own warning before the table, because a non-Korean reader will collapse it and the whole backlog story turns on the distinction:

> **정산 (jungsan) = settlement.** The nightly run that pays partners.
> **정정 (jeongjeong) = correction.** Undoing money a settlement already paid.

The two words share no character and mean opposite halves of the same problem, but they are near-homophones to a non-speaker, they are always adjacent in the documents (정산 정정 = "settlement correction"), and the romanised code makes the distinction invisible: `JUNGSAN_AMT` is a settlement amount, while the correction that would claw it back is spelled `CANCEL_CLAWBACK` in English. A document heading that reads 정산 정정 업무절차 is a *correction* procedure, not a settlement procedure. Every unresolved amount in [[RISK-SETTLEMENT-RECON-BACKLOG]] is a 정정 that never happened, in a 정산 that did.

A second hazard: `RUN_ID` is two different columns with two different types in the same schema, and `SANGTAE`/`STATUS` is three different vocabularies. Both are listed below.

## Terms

### 정산 / settlement — the run and its money

| Term | Definition | See Also |
|---|---|---|
| **정산** (jungsan, settlement) | Aggregating a partner's delivered orders for a date, deducting commission, and determining the payable amount. `dailySettlementJob`, daily at 02:00 KST via a Quartz cron `"0 0 2 * * ?"` in `Asia/Seoul` → `settlement-batch:README.md`, `src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| **정산 대상** (settlement target) | An order eligible for a run. Defined by one SQL predicate: `WHERE m.SANGTAE_CD = 'BAESONG_WANRYO' AND DATE(m.UPD_DTM) = ?`. No predicate mentions cancellation, and the reader's javadoc claims none is intended — "주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다. 배송이 완료되었다면 파트너는 이행을 마친 것으로 본다." ("the order's current status and whether it was cancelled are not part of the condition; if delivery completed, the partner is considered to have performed") — but `SANGTAE_CD` *is* a current-status filter, so a cancelled order leaves the target set as a side effect of the cancel's own write → `src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| **정산 기준일** (settlement base date) | The business date a run settles for. Passed to the batch as the job parameter `jungsanIlja`, computed in `DailySettlementQuartzJob` as `LocalDate.now().minusDays(1).toString()` — the JVM's default zone, ISO `yyyy-MM-dd`. A `DateUtil.settlementBaseDate()` exists in settlement-batch (KST, `yyyyMMdd`, returns the day *before yesterday* when called before 02:00) and another in order-service (always yesterday), but neither has any caller: quoting them as the batch's date rule is the single most likely mistake to make in this repository → `src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java`, `src/main/java/kr/co/sellflow/settlement/common/DateUtil.java`, `order-service:src/main/java/kr/co/sellflow/order/common/DateUtil.java` | [[PROC-SELLFLOW-RUNTIME]] |
| **JUNGSAN_ILJA** (정산일자, settlement date) | `DATE NOT NULL` on `SETTLEMENT_RUN`. The same value travels to settlement-anomaly as the `/detect` request field `jungsan_ilja` (a Python `date`), which joins on `r.JUNGSAN_ILJA = %(ilja)s` → `src/main/resources/db/migration/V1__settlement_schema.sql`, `settlement-anomaly:app/main.py` | [[API-SETTLEMENT-ANOMALY]] |
| **JUNGSAN_AMT** (정산금액, settlement amount) | `DECIMAL(15,0) NOT NULL` on `SETTLEMENT_DTL`. The payable amount: gross minus commission, where gross is `DANGA × SURYANG` from the order line. Also one of the two features the anomaly model scores → `src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`, `settlement-anomaly:model/detector.py` | [[SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL]] |
| **SUSURYO** (수수료, commission fee) | `DECIMAL(15,0) NOT NULL` on `SETTLEMENT_DTL`; the platform's cut. Computed in production as `gross × 0.12` with `RoundingMode.HALF_UP`, regardless of the partner's contract. The spec spells the same word `suSuRyo` on `OrderDtl`, a column the order DDL never creates → `src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java`, `sources/raw/specs/order-service-openapi.json` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |
| **수수료율** (fee rate) | Three tiers exist on paper: 기본 12.0% (base — "계약 미체결 파트너 전체", all partners without a contract), 프리미엄 9.5% ("월 거래액 1억 이상", monthly volume over 100M KRW), 신규 프로모션 6.0% ("입점 3개월 이내", within three months of onboarding). The sheet's own caution says contract rates are "2021년 이후 미정비. 현재 전 건 기본 수수료율 적용 중." ("not maintained since 2021; the base rate is currently applied to everything") → `sources/context/business-rules.md` | [[RISK-SETTLEMENT]] |
| **PARTNER_CONTRACT** | The `FEE_RATE DECIMAL(5,4)` / `SETTLE_CYCLE VARCHAR(10) DEFAULT 'DAILY'` table added in V3, and the `PartnerContract` class that models it. No running code reads either; `SettlementItemProcessor` hardcodes `new BigDecimal("0.12")` → `src/main/resources/db/migration/V3__add_partner_contract.sql`, `src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java` | [[RISK-SETTLEMENT]] |
| **지급 요청** (payment request) | The transmission that moves money to a partner, and the reason every other term in this glossary exists. "지급 요청이 전송되면 되돌릴 수 없다. 은행 이체는 익영업일에 실행된다." ("once sent it cannot be undone; the bank transfer executes the next business day"). The handover repeats it in bold. No code in settlement-batch transmits anything — the writer INSERTs a `SETTLEMENT_DTL` row and logs → `src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java`, `sources/context/handover/2025-03_정산팀_인수인계.md` (§5) | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| **정산 리포트** (settlement report) | The summary file the partner portal reads. `SettlementReportWriter.write(runId)` logs a line and writes nothing, and carries the TODO "취소 정정분은 이 리포트에 포함되지 않는다. 별도 확인 필요." ("cancellation corrections are not included in this report; needs separate checking") → `src/main/java/kr/co/sellflow/settlement/report/SettlementReportWriter.java` | [[RISK-SETTLEMENT]] |

### 정정 / correction — undoing a settlement

| Term | Definition | See Also |
|---|---|---|
| **정정** (jeongjeong, correction) | Correction. Not settlement. The word for adjusting an amount that has already been paid out → `sources/context/policy/정산_정정_업무절차_v1.1.md` | [[PROC-SETTLEMENT-CORRECTION]] |
| **정산 정정** (settlement correction) | Procedure v1.1 §2.1, verbatim: "지급이 완료된 정산 금액을 차월 정산에서 차감하여 조정하는 것." — "adjusting an already-paid settlement amount by deducting it from the next month's settlement." The single most load-bearing term in this reef, and the one no code path completes → `sources/context/policy/정산_정정_업무절차_v1.1.md` | [[PROC-SETTLEMENT-CORRECTION]] |
| **정정 대기 건** (pending correction case) | Procedure v1.1 §2.2, verbatim: "취소가 정산 이후에 접수되어 차감이 필요한 상태로 적재된 건." — "a case queued in a state needing deduction, because the cancellation was received after settlement." Materialised as a `CANCEL_RECON_QUEUE` row with `STATUS='PENDING'` → `sources/context/policy/정산_정정_업무절차_v1.1.md`, `src/main/resources/db/migration/V1__settlement_schema.sql` | [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] |
| **차월 차감** (chawol chagam, next-month deduction) | Deducting a correction amount from the following month's payout. 차월 = "the following month", 차감 = "deduction". The only remedy available, precisely because a sent 지급 요청 is irreversible; the handover states it as a rule — "지급 요청이 전송된 뒤에는 되돌릴 수 없다. 차월 차감으로만 정정 가능." ("after the payment request is sent it cannot be reversed; correction is possible only by next-month deduction") → `sources/context/handover/2025-03_정산팀_인수인계.md` (§5), `settlement-batch:README.md` | [[PROC-SETTLEMENT-CORRECTION]] |
| **차월 상계** (chawol sanggye, next-month offset) | The variant term used for incident recovery rather than routine correction: "차월 상계로 처리 결정" ("decided to handle by next-month offset"), the remedy chosen for the 2025-07-12 duplicate payout. 상계 is offsetting mutual balances; 차감 is one-sided deduction → `sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md` | [[RISK-SETTLEMENT]] |
| **정산 차감** (settlement deduction) | The 취소정책 sheet's column heading, marked `O` for all four cancel reasons, with the two rules underneath it: "정산 실행 전 취소 → 해당 주문을 정산 대상에서 제외" ("cancelled before the run → exclude the order from the settlement target") and "정산 실행 후 취소 → 정산팀이 차월 정산에서 수기 차감" ("cancelled after the run → the settlement team deducts manually from the next month's settlement"). The first rule is not implemented — the reader's predicate ignores cancellation → `sources/context/business-rules.md`, `src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java` | [[CON-ORDER-SETTLEMENT]] |
| **미정정 잔액** (mijeongjeong janaek, uncorrected balance) | The money owed back to Sellflow that no correction has yet recovered. Procedure v1.1 §3 assigns it to 재무기획팀: "분기 결산 시 미정정 잔액 확인" ("check the uncorrected balance at quarterly close"). 미- is the negating prefix, so the term literally means "not-yet-corrected balance". The 2026-08 mail thread is that clause being exercised: "차감 대기 상태로 남아 있는 금액이 결산 기준으로 1.8억을 넘습니다" ("the amount remaining in a pending-deduction state exceeds 180 million KRW as of the close") → `sources/context/policy/정산_정정_업무절차_v1.1.md`, `sources/raw/mail/RE_정산_미정정_금액_문의.eml` | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| **미정정 금액** (uncorrected amount) | The same quantity under the name the Slack channel and the export use: "재무기획팀에서 미정정 금액 문의가 왔습니다" ("a query about the uncorrected amount came in from the finance planning team"), and the CSV column heading 추정 미정정 금액(원) — "estimated uncorrected amount (KRW)". Note 추정 (estimated): the figure is derived, not stored → `sources/raw/slack/settlement-dev_2023-04_2026-08.json`, `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv` | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| **수기 정정** (manual correction) | Correction done by a person rather than a batch. The org chart gives 정산팀 the process 정산 오류 수기 정정 ("manual correction of settlement errors"), procedure v0.3 §5 planned for it — "예상 처리량 월 10건 미만. 별도 시스템 없이 수기로 처리한다." ("expected volume under 10 cases a month; handled manually with no separate system") — and the handover says what it became: "실제로는 파트너 문의가 들어온 건만 확인해서 처리해 왔음" ("in practice only cases where a partner enquired have been checked and handled") → `sources/context/org-chart.md`, `sources/context/policy/정산_정정_업무절차_v0.3.md`, `sources/context/handover/2025-03_정산팀_인수인계.md` | [[PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11]] |
| **ST-F-001** | 정산 정정 요청서 / Settlement Correction Request. Prescribed by procedure v1.1 §6, approved by the team lead, retained five years. The handover reports no official form is used: "정산 정정 이력을 남기는 공식 양식이 없음. 각자 엑셀로 관리 중." ("there is no official form for recording correction history; each person manages it in their own spreadsheet") → `sources/context/policy/정산_정정_업무절차_v1.1.md`, `sources/context/handover/2025-03_정산팀_인수인계.md` (§6) | [[PROC-SETTLEMENT-CORRECTION]] |
| **ST-F-002** | 월간 정정 현황 확인서 / Monthly Correction Review. Prescribed by v1.1 §6 as the record of the mandatory monthly queue review (§5: "월 1회 이상 대기 건 현황을 확인한다"). No completed instance exists in this reef, and the handover states the review itself does not happen: "전체 대기열을 주기적으로 확인하는 절차는 없음" ("there is no procedure for periodically checking the whole queue") → `sources/context/policy/정산_정정_업무절차_v1.1.md` | [[PROC-SETTLEMENT-CORRECTION]] |

### Tables, columns and their status vocabularies

| Term | Definition | See Also |
|---|---|---|
| **RUN_ID** — two of them | **`SETTLEMENT_RUN.RUN_ID`** is `BIGINT NOT NULL AUTO_INCREMENT`, the PK of a run and part 1 of `SETTLEMENT_DTL`'s composite PK, resolved at write time by `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'`. **`SETTLEMENT_RUN_LOG.RUN_ID`** is `VARCHAR(32) NOT NULL PRIMARY KEY`, added by V2 for a table nothing writes. They share a name, differ in type, and cannot be joined. `SettlementReportWriter.write(String runId)` takes the string form; the writer produces the numeric one → `src/main/resources/db/migration/V1__settlement_schema.sql`, `src/main/resources/db/migration/V2__add_settlement_run_log.sql` | [[PROC-SETTLEMENT-RUN-LOG-LIFECYCLE]] |
| **SANGTAE** (상태, status) on `SETTLEMENT_RUN` | `VARCHAR(20) NOT NULL`. Only one value is observable in the repository — `'RUNNING'`, in the writer's subquery. No code in settlement-batch inserts a `SETTLEMENT_RUN` row, so nothing here sets it and nothing clears it. Note the order domain spells the same word `SANGTAE_CD` and `SETTLEMENT_RUN_LOG` spells it `STATUS` → `src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java` | [[SCH-SETTLEMENT-BATCH]] |
| **STATUS** on `CANCEL_RECON_QUEUE` | `VARCHAR(20) DEFAULT 'PENDING'`. Two values are written in code: `'PENDING'` by `OrderEventRelayJob`'s INSERT, and `'PROCESSED'` by `CancelReconciler`'s UPDATE. The second has never run — the reconciler is not registered in `QuartzConfig`, which registers only `dailySettlementQuartzJob` and `orderEventRelayJob`. The export confirms the consequence: "STATUS 가 PENDING 외의 값을 가진 행은 조회되지 않았다" ("no row with a STATUS other than PENDING was returned") → `src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `src/main/java/kr/co/sellflow/settlement/config/QuartzConfig.java`, `sources/raw/exports/README.md` | [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] |
| **RECV_DTM** (수신일시, received date-time) | `DATETIME DEFAULT CURRENT_TIMESTAMP` on `CANCEL_RECON_QUEUE`. The *relay* clock, not the cancellation clock: it is stamped by the database when the ten-minute poll picks the outbox row up, so it trails the order's `CHWISO_ILSI` (a JVM `LocalDateTime.now()`) by up to one polling interval. Every figure in the backlog export is grouped by this column, `DATE_FORMAT(RECV_DTM,'%Y-%m')` → `src/main/resources/db/migration/V1__settlement_schema.sql`, `sources/raw/exports/README.md` | [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] |
| **PROCESSED_DTM** vs **PROCESSED_AT** | Two columns for one idea, on the same table. `PROCESSED_DTM DATETIME` is created by V1 and is the one `CancelReconciler` would write (`SET STATUS='PROCESSED', PROCESSED_DTM=NOW()`). `PROCESSED_AT DATETIME NULL` is added by V5 (with `PROCESSED_BY VARCHAR(30) NULL`) under the comment "정정 처리 결과를 남기기 위한 컬럼. 아직 쓰는 코드는 없다." ("columns for recording the correction result; no code uses them yet"). Both are empty in practice, for different reasons: the V1 column's writer never runs, and the V5 columns have no writer at all → `src/main/resources/db/migration/V1__settlement_schema.sql`, `src/main/resources/db/migration/V5__add_recon_processed_columns.sql`, `src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java` | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| **PROCESSED_BY** | `VARCHAR(30) NULL`, V5. The only column in the settlement schema that would name a *person* doing a correction. Never written → `src/main/resources/db/migration/V5__add_recon_processed_columns.sql` | [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] |
| **CANCEL_RECON_QUEUE** | The table holding 정정 대기 건, created by V1 with the comment "SF-2287 대응. 정산 후 취소 건을 수기 정정용으로 적재한다. 정산팀이 주기적으로 확인하여 차월 정산에서 차감한다." ("in response to SF-2287; loads post-settlement cancellations for manual correction, which the settlement team checks periodically and deducts from the next month's settlement"). Written by the relay, drained by nothing. 4,127 `PENDING` rows as of 2026-09-01, oldest 2023-04 → `src/main/resources/db/migration/V1__settlement_schema.sql`, `sources/raw/exports/cancel_recon_queue_monthly_20260901.csv` | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| **SEQ** | `BIGINT NOT NULL AUTO_INCREMENT`, the queue PK. `ORD_NO` carries no unique key, so the same order may be queued any number of times — unlike `ORDER_CANCEL`, where `ORD_NO` *is* the PK → `src/main/resources/db/migration/V1__settlement_schema.sql` | [[CON-SELLFLOW-CANCEL-ENTITY-COMPARISON]] |
| **SAYU_CD** (사유코드, reason code) | `VARCHAR(2) NOT NULL` on the queue. Documented values are the order domain's four: 01 파트너 귀책 (partner at fault), 02 시스템 오류 (system error), 03 고객 변심 (customer change of mind), 04 배송 실패 (delivery failure), all marked 정산 차감 = O. But this side does not validate: the relay extracts it by string surgery — `s.indexOf("\"sayuCd\":\"")` then `s.substring(i + 10, i + 12)` — and returns the literal `"00"` when the marker is absent. `"00"` is in no enum, no sheet and no policy page in the estate → `src/main/java/kr/co/sellflow/settlement/relay/OrderEventRelayJob.java`, `sources/context/business-rules.md` | [[GLOSSARY-SELLFLOW]] |
| **SETTLEMENT_ADJUSTMENT** | The correction-record table `CancelReconciler` INSERTs into. No migration in V1-V6 creates it, and the handover reports "`SETTLEMENT_ADJUSTMENT` 테이블이 문서에는 나오는데 실제로 조회가 안 됨. WIP" ("the table appears in the documentation but cannot actually be queried") → `src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java`, `sources/context/handover/2025-03_정산팀_인수인계.md` (§3) | [[SCH-SETTLEMENT-BATCH]] |
| **ADJ_TYPE** / **CANCEL_CLAWBACK** | `ADJ_TYPE` is the adjustment-type column on `SETTLEMENT_ADJUSTMENT`; `'CANCEL_CLAWBACK'` is the single literal hardcoded into the one INSERT that targets it, marking the clawback of a settled-then-cancelled order. It is also the estate's only English name for 차월 차감 — the Korean term in the procedure and the English constant in the code are the same mechanism, and nothing in either file says so. Never written, because the writer never runs → `src/main/java/kr/co/sellflow/settlement/recon/CancelReconciler.java` | [[PROC-SETTLEMENT-CORRECTION]] |
| **SANGTAE_CD BAESONG_WANRYO** (배송완료, delivery complete) | The order state the settlement reader selects on. A settlement-side use of an order-side vocabulary → `src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java` | [[PROC-SETTLEMENT-DAILY-BATCH]] |
| **SANGTAE_CD JUNGSAN_WANRYO** (정산완료, settlement complete) | The order state written back by `markSettledStep`, by direct UPDATE of a table another team owns. The tasklet's javadoc justifies it: "ORDER_MST 는 주문팀 소유 테이블이나, 통합 DB 정책에 따라 정산 배치가 직접 갱신한다. (2019년 협의)" ("`ORDER_MST` is owned by the order team, but under the integrated-DB policy the settlement batch updates it directly; agreed 2019") → `src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java` | [[DEC-SELLFLOW-SHARED-DB]] |
| **MoneyUtil** | A shared-name hazard, flagged in both files. settlement-batch's `MoneyUtil.fee()` truncates (`RoundingMode.FLOOR`); order-service's class of the same name rounds (`HALF_UP`). "2022 협의 결과이며 문서화되어 있지 않다." ("the result of a 2022 agreement, and it is not documented"). Settlement's own production processor uses `HALF_UP` inline and never calls `MoneyUtil` at all, matching neither rule → `src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java`, `src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java` | [[DEC-SELLFLOW-MONEY-ROUNDING]] |

### 이상 탐지 / anomaly detection vocabulary

`SETTLEMENT_ANOMALY` lives in the same `sellflow_order` instance and is owned by 데이터플랫폼본부 데이터팀, not 정산팀.

| Term | Definition | See Also |
|---|---|---|
| **이상 탐지** (anomaly detection) | settlement-anomaly's whole job: score the day's settlement rows and INSERT findings. Triggered after the batch, "매일 03:00" per its README. The registry records that the follow-up is unowned — "이상 탐지만 수행. 조치 주체는 정의되어 있지 않음. TODO" ("it only performs detection; the party responsible for acting is not defined") → `settlement-anomaly:README.md`, `sources/context/registry/services.yaml` | [[SYS-SETTLEMENT-ANOMALY]] |
| **ANOMALY_CD** | `VARCHAR(30) NOT NULL` on `SETTLEMENT_ANOMALY`, indexed as `IX_SETTLEMENT_ANOMALY_02`. Four values are documented in the README table; two are produced by the detector. The column is unconstrained, so the gap is invisible at the schema level → `settlement-anomaly:sql/V1__anomaly_schema.sql` | [[SCH-SETTLEMENT-ANOMALY]] |
| **CANCELLED_SETTLED** (취소 후 정산 잔존) | "a cancelled order still present in a settlement". Implemented as a rule, not a model: if `r["sangtae_cd"]` is in `CANCELLED_STATES = {"CHWISO", "BANPUM"}`, emit the code with a fixed `"score": 1.0`. It flags exactly the population that fills `CANCEL_RECON_QUEUE` — detected nightly by one division, acted on by none → `settlement-anomaly:model/detector.py` | [[RISK-SETTLEMENT-RECON-BACKLOG]] |
| **AMT_OUTLIER** (금액 이상, amount anomaly) | "deviation from a partner's historical distribution". The only model-driven code: an IsolationForest score over the two features `[JUNGSAN_AMT, SUSURYO]`, emitted when `score > 0.85`. The model artefact is `model/artifacts/iforest_v3.pkl` and `model/features.py` notes "2024-11 이후 재학습 이력 없음" ("no retraining since 2024-11") → `settlement-anomaly:model/detector.py`, `settlement-anomaly:model/features.py` | [[SCH-SETTLEMENT-ANOMALY]] |
| **DUP_SETTLE** (중복 정산, duplicate settlement) | "the same order settled more than once". In the README table; **absent from `detector.py`**. It is also the failure mode that actually occurred on 2025-07-12 → `settlement-anomaly:README.md`, `settlement-anomaly:model/detector.py` | [[RISK-SETTLEMENT]] |
| **FEE_MISMATCH** (수수료 불일치, fee mismatch) | "a commission differing from the contract rate". In the README table; **absent from `detector.py`**, and unactionable anyway while the batch hardcodes 0.12 and nothing reads `PARTNER_CONTRACT` → `settlement-anomaly:README.md` | [[RISK-SETTLEMENT]] |
| **DETECTED** | `STATUS VARCHAR(20) DEFAULT 'DETECTED'` on `SETTLEMENT_ANOMALY`, and the literal the INSERT writes explicitly. The schema anticipates a review workflow — `REVIEWED_BY VARCHAR(30)`, `REVIEWED_DTM DATETIME`, and an index on `(STATUS, DETECTED_DTM)` — and no code in the repository writes any of the three, so no anomaly ever leaves `DETECTED` → `settlement-anomaly:sql/V1__anomaly_schema.sql`, `settlement-anomaly:app/main.py` | [[PROC-SETTLEMENT-ANOMALY-LIFECYCLE]] |
| **DETECTED_DTM** | `DATETIME NOT NULL`, written as `NOW()` at detect time — the 03:00 run, an hour after the 02:00 batch. Not the time the underlying cancellation happened → `settlement-anomaly:app/main.py` | [[SCH-SETTLEMENT-ANOMALY]] |
| **SCORE** | `DECIMAL(5,4) NOT NULL`. Carries two incomparable things under one name: an IsolationForest score for `AMT_OUTLIER`, and the constant `1.0` for `CANCELLED_SETTLED`. A consumer ranking by `SCORE` would put every rule hit above every model hit → `settlement-anomaly:model/detector.py` | [[SCH-SETTLEMENT-ANOMALY]] |
| **AnomalyRow** — unused | The Pydantic model in `app/schemas.py` describing a finding as `run_id: str`, `ord_no`, `anomaly_type`, `score`, `status = "DETECTED"`. Nothing imports it; `main.py` writes raw SQL with the column name `ANOMALY_CD`, not `anomaly_type`, and does not carry `run_id` at all. A fourth name for the anomaly code, in a file no code path reaches → `settlement-anomaly:app/schemas.py`, `settlement-anomaly:app/main.py` | [[SCH-SETTLEMENT-ANOMALY]] |
| **cancel_after_settle_flag** | One of the four names in `model/features.py`, and per its own TODO the dominant one: "cancel_after_settle_flag 가 사실상 단독으로 결과를 좌우한다. 가중치 재조정 필요." ("it effectively determines the outcome single-handedly; the weights need rebalancing"). Note that the live detector does not use `FEATURES` at all — it scores `[jungsan_amt, susuryo]` → `settlement-anomaly:model/features.py`, `settlement-anomaly:model/detector.py` | [[SYS-SETTLEMENT-ANOMALY]] |

## Related

- [[SYS-SETTLEMENT]] — the batch service where most of this vocabulary originates
- [[SCH-SETTLEMENT-BATCH]] — the tables and columns named above
- [[SCH-SETTLEMENT-ANOMALY]] — the anomaly table and its codes
- [[PROC-SETTLEMENT-CORRECTION]] — where the correction terms are used in anger
- [[PROC-SETTLEMENT-DAILY-BATCH]] — where the amount terms are computed
- [[PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE]] — the queue row from PENDING to nowhere
- [[RISK-SETTLEMENT-RECON-BACKLOG]] — what 미정정 잔액 currently amounts to
- [[GLOSSARY-SELLFLOW]] — the cross-service glossary, for terms that mean different things elsewhere
