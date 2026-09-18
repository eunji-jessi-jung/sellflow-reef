---
id: "PROC-SETTLEMENT-DAILY-BATCH"
type: "process"
title: "Daily Partner Settlement Batch"
domain: "settlement"
status: "draft"
last_verified: 2026-09-18
freshness_note: "Traced line by line through DailySettlementJobConfig, SettlementItemProcessor, SettlementItemWriter and MarkSettledTasklet on 2026-09-18. Stale if the reader SQL, the fee rate constant or the step order changes."
freshness_triggers:
  - "src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
  - "src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - "src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - "src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - "src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - "src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java"
known_unknowns:
  - "Who creates the SETTLEMENT_RUN row with SANGTAE='RUNNING'. Without it, currentRunId() returns null and every SETTLEMENT_DTL insert would carry a null RUN_ID. No INSERT exists in any repo."
  - "Whether the jungsanIlja job parameter format ever matches the data. DailySettlementQuartzJob passes an ISO yyyy-MM-dd string in the system default zone, while DateUtil formats yyyyMMdd in KST and is never called."
  - "Whether the batch has ever settled a cancelled order in production. The reader's condition permits it, but no query result confirming an actual occurrence was available."
  - "What the payment request actually is. The writer's javadoc describes an irreversible payment request and a next-business-day bank transfer, but the only code is an INSERT into SETTLEMENT_DTL — the transmission mechanism is not in this repo."
  - "Whether TOTAL_AMT, START_DTM and END_DTM on SETTLEMENT_RUN are ever populated, and by what."
  - "How failures are handled. No skip policy, retry policy, fault-tolerant configuration or listener is declared on either step."
tags:
  - settlement
  - spring-batch
  - pipeline
  - payout
aliases:
  - "dailySettlementJob"
  - "일일 정산 배치"
relates_to:
  - type: "depends_on"
    target: "[[API-SETTLEMENT-BATCH]]"
  - type: "refines"
    target: "[[GLOSSARY-SETTLEMENT]]"
  - type: "refines"
    target: "[[PROC-SETTLEMENT-CORRECTION]]"
  - type: "refines"
    target: "[[RISK-SETTLEMENT]]"
  - type: "depends_on"
    target: "[[SCH-SETTLEMENT-BATCH]]"
  - type: "parent"
    target: "[[SYS-SETTLEMENT]]"
  - type: "integrates_with"
    target: "[[SYS-SETTLEMENT-ANOMALY]]"
sources:
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/business-rules.md"
    notes: "수수료 sheet — base 12.0%, premium 9.5%, promo 6.0%, and the note that contract rates are unmaintained since 2021"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/handover/2025-03_정산팀_인수인계.md"
    notes: "§5 — the irreversibility of a sent payment request"
  - category: "documentation"
    type: "doc"
    ref: "sellflow-docs:context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md"
    notes: "P2 duplicate run; the open action item on excluding cancelled orders"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java"
  - category: "implementation"
    type: "github"
    ref: "settlement-batch:src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java"
notes: ""
---

# Daily Partner Settlement Batch

## Purpose

`dailySettlementJob` converts one day of completed deliveries into partner payouts. It reads yesterday's delivered orders, deducts a commission, writes a settlement detail row per order, and marks those orders settled. It is the only automated money-moving process in the settlement domain, and — because the resulting payment request cannot be recalled — everything it gets wrong has to be fixed by the separate, manual path described in [[PROC-SETTLEMENT-CORRECTION]].

## Key Facts

- The job is two steps in order: `settlementStep` then `markSettledStep` → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java
- `settlementStep` is a chunk-oriented step with chunk size 500, composed of `JdbcCursorItemReader` → `SettlementItemProcessor` → `SettlementItemWriter` → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java
- The reader selects on `m.SANGTAE_CD = 'BAESONG_WANRYO' AND DATE(m.UPD_DTM) = ?` and applies no cancellation filter → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java
- The reader's javadoc presents that omission as intended: "주문의 현재 상태나 취소 여부는 조건에 포함하지 않는다. 배송이 완료되었다면 파트너는 이행을 마친 것으로 본다." — "the order's current status and whether it was cancelled are not part of the condition; if delivery completed, the partner is considered to have fulfilled its obligation". The SQL beneath it does not implement that intent: `m.SANGTAE_CD = 'BAESONG_WANRYO'` *is* a current-status predicate, and `OrderMst.chwiso()` overwrites both `SANGTAE_CD` and `UPD_DTM`, so a cancelled order drops out of the target set anyway. The javadoc is a statement of intent, not a description of behaviour → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java, order-service:src/main/java/kr/co/sellflow/order/domain/OrderMst.java
- `SettlementItemProcessor` hardcodes `DEFAULT_FEE_RATE = new BigDecimal("0.12")` and rounds with `RoundingMode.HALF_UP` → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- The processor never reads `PartnerContract.getFeeRate()`; `PartnerContractRepository` has no caller anywhere in the five repos (verified by grep — the only hits are its own declaration and its import of the domain class) → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, src/main/java/kr/co/sellflow/settlement/repository/PartnerContractRepository.java
- The processor's javadoc states the same: "파트너별 계약 수수료율은 PARTNER_CONTRACT 에 있으나, 현재는 기본 수수료율만 적용한다. (2021년 이후 미정비)" — "per-partner contract rates are in PARTNER_CONTRACT, but currently only the base rate is applied (unmaintained since 2021)" → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- The business-rules workbook independently confirms the data side: "계약별 수수료율은 PARTNER_CONTRACT 에 있으나 2021년 이후 미정비. 현재 전 건 기본 수수료율 적용 중." — "per-contract rates are in PARTNER_CONTRACT but unmaintained since 2021; the base rate is currently applied to everything" → sources/context/business-rules.xlsx (수수료 sheet)
- Two contract tiers therefore exist on paper but not in code: 프리미엄 9.5% for partners above 100M KRW monthly volume and 신규 프로모션 6.0% within three months of onboarding → sources/context/business-rules.xlsx (수수료 sheet)
- `MoneyUtil.fee()` rounds with `RoundingMode.FLOOR`, the opposite of the processor's `HALF_UP`, and `MoneyUtil` is never called by the processor or by any other class → src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java
- `MoneyUtil`'s javadoc records an undocumented cross-service agreement: "정산은 절사(FLOOR), 주문은 반올림(HALF_UP). 2022 협의 결과이며 문서화되어 있지 않다." — "settlement truncates (FLOOR), order rounds (HALF_UP); agreed in 2022 and not documented" → src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java
- The repo's only test, `SettlementItemProcessorTest.수수료는_절사한다` ("commission is truncated"), asserts `MoneyUtil.fee(14999, 0.1) == 1499` — it never instantiates `SettlementItemProcessor` and never exercises the code path that actually computes the fee → src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java
- `SettlementItemWriter` resolves the run by `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'`, and no code in the repo ever INSERTs into `SETTLEMENT_RUN` (verified by grep) → src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- The writer runs one `INSERT` per item inside the chunk loop rather than a batch update → src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- Payment requests are irreversible: "지급 요청이 전송되면 되돌릴 수 없다. 은행 이체는 익영업일에 실행된다. 정정이 필요한 경우 차월 정산에서 조정한다." — "once the payment request is sent it cannot be undone; the bank transfer executes on the next business day; if correction is needed it is adjusted in the next month's settlement" → src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java
- The handover repeats the same rule as the first item under 주의사항 (cautions): "지급 요청이 전송된 뒤에는 되돌릴 수 없다. 차월 차감으로만 정정 가능." — "after a payment request is sent it cannot be undone; correction is only possible by next-month deduction" → sources/context/handover/2025-03_정산팀_인수인계.md (§5)
- `markSettledStep` updates `ORDER_MST` for every order in `(SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN)` — note this subquery omits the `SANGTAE='RUNNING'` filter the writer uses, so the two steps can disagree on which run is current → src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

## Phases

### Phase 0 — Trigger

Quartz fires `dailySettlementTrigger` at 02:00 KST. `DailySettlementQuartzJob` builds job parameters and launches the Spring Batch job:

```java
.addString("jungsanIlja", LocalDate.now().minusDays(1).toString())
.addLong("ts", System.currentTimeMillis())
```
→ src/main/java/kr/co/sellflow/settlement/job/DailySettlementQuartzJob.java

The `ts` parameter guarantees a distinct `JobInstance` every run, so Spring Batch's own duplicate-instance protection never engages. Two nodes firing the same trigger would produce two accepted job instances — which is what happened on 2025-07-12 → sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

`DateUtil.settlementBaseDate()` exists and implements a careful KST rule ("배치가 02:00 에 돌기 때문에, 00:00~02:00 사이에 수동 실행되면 전전일이 기준일이 되어야 한다" — "because the batch runs at 02:00, a manual run between 00:00 and 02:00 should use the day before yesterday as the base date"), formats `yyyyMMdd`, and is called by nothing → src/main/java/kr/co/sellflow/settlement/common/DateUtil.java

### Phase 1 — Read

```sql
SELECT m.ORD_NO, d.PARTNER_ID, d.SANGPUM_CD, d.SURYANG, d.DANGA
  FROM ORDER_MST m
  JOIN ORDER_DTL d ON d.ORD_NO = m.ORD_NO
 WHERE m.SANGTAE_CD = 'BAESONG_WANRYO'
   AND DATE(m.UPD_DTM) = ?
```
→ src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java

This is the load-bearing decision of the whole process. `BAESONG_WANRYO` (배송완료, delivery complete) is checked as the current state, but the query does not exclude orders whose state has since moved to a cancelled value, and it does not consult `CANCEL_RECON_QUEUE`. The design intent is written into the javadoc and quoted in the Key Facts above.

Note the interaction with `SANGTAE_CD`: since `markSettledStep` overwrites the state to `JUNGSAN_WANRYO`, and `order-service` may overwrite it to a cancelled value, the reader's own filter behaves differently depending on the order of same-day events. That interaction was not traced against `order-service` code here.

The business-rules workbook states the intended rule, which the reader does not implement: "정산 실행 전 취소 — 해당 주문을 정산 대상에서 제외" ("cancellation before the settlement run — exclude the order from settlement") → sources/context/business-rules.xlsx (취소정책 sheet). The 2025-07-12 postmortem raised the same concern as an action item that was never closed: "취소 건이 정산 대상에서 제외되는지 점검 — 2025-07-15 제기, 이후 논의 없음" ("check whether cancelled orders are excluded from settlement — raised 2025-07-15, no discussion since") → sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

### Phase 2 — Process

```java
BigDecimal fee = gross.multiply(DEFAULT_FEE_RATE).setScale(0, RoundingMode.HALF_UP);
item.setSusuryo(fee);
item.setJungsanAmt(gross.subtract(fee));
```
→ src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java

`gross` is `DANGA × SURYANG`, computed in the domain object → src/main/java/kr/co/sellflow/settlement/domain/SettlementTarget.java

Three facts collide here, and they are worth separating:

1. The rate is a constant. `PARTNER_CONTRACT.FEE_RATE` is never read, so premium and promotional partners are charged 12 percent → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, sources/context/business-rules.xlsx (수수료 sheet)
2. The rounding mode is `HALF_UP`, while the repo's own `MoneyUtil` — written for exactly this calculation and documenting a 2022 cross-team agreement that settlement truncates — uses `FLOOR` → src/main/java/kr/co/sellflow/settlement/common/MoneyUtil.java
3. The only test asserts the `FLOOR` behaviour through `MoneyUtil`, under the name 수수료는_절사한다 ("commission is truncated"), and never touches the processor → src/test/java/kr/co/sellflow/settlement/job/SettlementItemProcessorTest.java

So the test passes, the agreed rule is implemented in a helper, and the production path does the opposite. For a 14,999 KRW gross at 10 percent the helper yields 1,499 and the processor's arithmetic would yield 1,500 — a one-won difference per order, in the platform's favour, that no test would catch.

### Phase 3 — Write

For each item in the chunk:

```sql
INSERT INTO SETTLEMENT_DTL (RUN_ID, ORD_NO, PARTNER_ID, JUNGSAN_AMT, SUSURYO) VALUES (?, ?, ?, ?, ?)
```
→ src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java

`RUN_ID` comes from `SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN WHERE SANGTAE='RUNNING'`, re-executed once per chunk. Nothing in the repo creates that row, which leaves the run's lifecycle owner unidentified — recorded in `known_unknowns` and in [[SCH-SETTLEMENT-BATCH]].

The writer's javadoc is the authority for irreversibility. The code itself only inserts rows; the mechanism that turns a `SETTLEMENT_DTL` row into an actual payment request is not in this repo.

### Phase 4 — Mark settled

```sql
UPDATE ORDER_MST SET SANGTAE_CD='JUNGSAN_WANRYO', UPD_DTM=NOW()
 WHERE ORD_NO IN (SELECT ORD_NO FROM SETTLEMENT_DTL
                   WHERE RUN_ID = (SELECT MAX(RUN_ID) FROM SETTLEMENT_RUN))
```
→ src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

Two observations. First, this writes a table owned by 주문팀, justified by the 2019 consolidated-database agreement quoted in [[API-SETTLEMENT-BATCH]]. Second, it sets `UPD_DTM=NOW()` on every settled order — the same column the next day's reader filters on with `DATE(m.UPD_DTM) = ?`. The state filter (`BAESONG_WANRYO`) is what prevents those rows from being picked up again.

## Worked Examples

**A normal order**

- `ORDER_DTL`: `DANGA = 22,750`, `SURYANG = 2` → gross 45,500
- Processor: fee = 45,500 × 0.12 = 5,460 exactly, so `HALF_UP` and `FLOOR` agree here → `SUSURYO = 5,460`, `JUNGSAN_AMT = 40,040`
- Writer: `INSERT INTO SETTLEMENT_DTL (RUN_ID, 'ORD20260817001', 'P00132', 40040, 5460)`
- Tasklet: `ORDER_MST.SANGTAE_CD` becomes `JUNGSAN_WANRYO`
→ src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, src/main/java/kr/co/sellflow/settlement/job/SettlementItemWriter.java, src/main/java/kr/co/sellflow/settlement/tasklet/MarkSettledTasklet.java

**A premium partner**

The same order under a 9.5 percent contract should yield a 4,322 KRW fee. Because `PARTNER_CONTRACT` is never read, the partner is charged 5,460 — a 1,138 KRW difference on one order, invisible to the batch and undetectable by the `FEE_MISMATCH` rule that [[SYS-SETTLEMENT-ANOMALY]] documents but does not implement → src/main/java/kr/co/sellflow/settlement/job/SettlementItemProcessor.java, sources/context/business-rules.xlsx (수수료 sheet), README.md (settlement-anomaly)

**An order delivered yesterday and cancelled this morning**

The reader's `BAESONG_WANRYO` filter depends on when the cancellation updated `SANGTAE_CD` relative to 02:00. If the state still reads `BAESONG_WANRYO` at 02:00, the order settles, money leaves, and the only recovery is the next-month deduction — the path traced in [[PROC-SETTLEMENT-CORRECTION]]. The reader's javadoc treats this as intended behaviour rather than a defect, since delivery completion is taken as proof of partner fulfilment → src/main/java/kr/co/sellflow/settlement/config/DailySettlementJobConfig.java

**The 2025-07-12 duplicate run**

Quartz cluster configuration was applied to only one of two nodes during a redundancy change, so the trigger fired twice at 02:04. Payment requests were duplicated for 17 partners, roughly 42,000,000 KRW, and because the writer has no recall path the resolution was next-month offset: "중복 지급 요청 취소 불가 확인, 차월 상계로 처리 결정" — "confirmed the duplicate payment requests cannot be cancelled; decided to handle by next-month offset" → sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md

Of the five follow-up items, two remain open: 지급 요청 전 멱등성 키 도입 (introduce an idempotency key before the payment request), still unassigned, and the cancellation-exclusion check quoted above → sources/context/runbooks/장애회고_2025-07-12_정산배치_중복실행.md, see [[RISK-SETTLEMENT]]

## Related

- [[SYS-SETTLEMENT]] — the service that runs this job
- [[SCH-SETTLEMENT-BATCH]] — the tables written here
- [[API-SETTLEMENT-BATCH]] — the trigger that starts it
- [[PROC-SETTLEMENT-CORRECTION]] — the only path for undoing what this job does
- [[RISK-SETTLEMENT]] — the fee-rate, test-coverage and idempotency risks collected
- [[SYS-SETTLEMENT-ANOMALY]] — the detector that inspects this job's output
- [[GLOSSARY-SETTLEMENT]] — `JUNGSAN_AMT`, `SUSURYO`, `RUN_ID` and the rest of the vocabulary
