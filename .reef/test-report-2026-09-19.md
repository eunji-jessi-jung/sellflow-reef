# Test Your Reef — sellflow-reef                       2026-09-19
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Independent test pass. The 52 questions in `.reef/questions.json` were re-graded from
scratch against `artifacts/**` alone — no source repository, no `sources/` tree, no
`questions-for-owner.md`, and the `status` field already on each question was ignored
rather than trusted. Phase B then checked the result against `ANSWER-KEY.md`.

Progress: ██████████████████░░ 47/52 questions fully answered

Grading rubric (from `/reef:test`): **fully answered** = a clear, sourced answer a reader
does not have to leave the reef to complete. **partially answered** = relevant material
exists but a dimension of the question is missing. **not answerable** = the artifacts do
not cover it, or the coverage is too thin to constitute an answer.

One judgement applied consistently throughout, stated up front because it decides five
gradings: where the *sources themselves* cannot settle a question and the reef says so
explicitly, with the search that establishes the absence, that counts as a full answer —
"nothing in any of the five repos calls this, verified by grep" is an answer. Where the
reef supplies only surrounding evidence and a method, and no answer, that is partial.

---

## Phase A — blind evaluation

### ✓ Fully answered (47)

| Q | Question (abbrev.) | Decisive artifacts |
|---|---|---|
| Q-001 | order-service boundaries and external dependencies | SYS-ORDER (Dependencies table: PG, inventory-api, settlement-batch, delivery-bff, admin origin, registry), PROC-SELLFLOW-RUNTIME |
| Q-002 | Core order entities and their relations | SCH-ORDER (six field tables, ER diagram, Relationships table with "enforced by" column) |
| Q-003 | OrderStatus lifecycle; code-enforced vs externally written | PROC-ORDER-ORDER-MST-LIFECYCLE ("four writers, one guard" table + state diagram), PROC-ORDER-CANCEL |
| Q-004 | CancelReason codes; where cost/restock policy lives | SCH-ORDER (enum tables), GLOSSARY-SELLFLOW (사유 코드 section), DEC-INVENTORY-RESTOCK-BY-REASON |
| Q-005 | States blocking cancel after SF-2287; settlement check | PROC-ORDER-CANCEL, DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL (`EnumSet.of(CHWISO, BANPUM)`; check survives only in deprecated V1) |
| Q-006 | How `order.cancelled` is written; who consumes the outbox | PROC-ORDER-EVENT-OUTBOX-LIFECYCLE (field table, states, creation path), DEC-ORDER-OUTBOX-RELAY |
| Q-007 | Is `InventoryClient.restore()` invoked in the cancel path | PROC-ORDER-CANCEL ("Steps that are not in the list" table), CON-ORDER-INVENTORY |
| Q-008 | Real API surface vs the 2022 OpenAPI drift | API-ORDER (Surface Profile: 30 paths / 32 operations vs 2 live endpoints; per-path "In code?" tables) |
| Q-009 | Auth enforcement — `bearerAuth` in spec vs code | PROC-ORDER-AUTH (answers Q-009 by exhaustion, with the re-runnable method), API-ORDER Auth Posture |
| Q-010 | Flyway V1–V24 history incl. the V15/V16 revert | SCH-ORDER-MIGRATION-HISTORY (all 24 in a timeline; "The V15/V16 story, in full") |
| Q-011 | Why `OrderCancelServiceV1` survives; pre-SF-2287 policy | DEC-ORDER-SF2287 (removed check quoted verbatim), PROC-ORDER-ORDER-CANCEL-LIFECYCLE (three-check zero-caller proof), SYS-ORDER |
| Q-012 | Disabled tests and the behaviour unverified in CI | RISK-ORDER-DISABLED-TESTS (one row per test method, plus three "absent" rows) |
| Q-013 | `dailySettlementJob` composition and schedule | PROC-SETTLEMENT-DAILY-BATCH (phases), API-SETTLEMENT-BATCH (Job and step contract table) |
| Q-014 | Settlement target SQL; does it exclude cancelled orders | PROC-SETTLEMENT-DAILY-BATCH, CON-ORDER-SETTLEMENT (exclusion is incidental, not a rule — javadoc is intent, not behaviour) |
| Q-015 | Fee computation; is `PARTNER_CONTRACT.FEE_RATE` used | SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL, PROC-SETTLEMENT-DAILY-BATCH (hardcoded `0.12`, HALF_UP; repository has no caller) |
| Q-016 | What `MarkSettledTasklet` writes; under what agreement | CON-ORDER-SETTLEMENT, API-SETTLEMENT-BATCH (SQL + the 2019 comment quoted) |
| Q-017 | How the relay moves outbox rows into the queue | PROC-ORDER-EVENT-OUTBOX-LIFECYCLE, API-SETTLEMENT-BATCH (four-step worked SQL) |
| Q-018 | Is `CancelReconciler` scheduled; what happens if not | PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE, DEC-SETTLEMENT-CANCEL-CLAWBACK, RISK-SETTLEMENT-RECON-BACKLOG |
| Q-019 | The six settlement entities; which are written by code | SCH-SETTLEMENT-BATCH (per-table column tables with writer/reader status for all six, incl. the no-DDL `SETTLEMENT_ADJUSTMENT`) |
| Q-020 | Which DB settlement-batch uses; sharing implications | SYS-SETTLEMENT, SCH-SETTLEMENT-BATCH, DEC-SELLFLOW-SHARED-DB |
| Q-021 | The two `MoneyUtil` implementations | DEC-SELLFLOW-MONEY-ROUNDING (three behaviours where the comments describe two), SCH-SETTLEMENT-FIELD-LINEAGE (arithmetic differs too) |
| Q-022 | Deployment and scheduling mechanics | API-SETTLEMENT-BATCH §4 (workflow_dispatch, `-x test`, absent `deploy.sh`), SYS-SETTLEMENT, PROC-SETTLEMENT-AUTH |
| Q-023 | Anomaly types, and model vs rule per type | SYS-SETTLEMENT-ANOMALY, SCH-SETTLEMENT-ANOMALY (`ANOMALY_CD` enum table with "emitted by code?"), PROC-SETTLEMENT-ANOMALY-LIFECYCLE |
| Q-024 | What settlement-anomaly reads/writes across boundaries | CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH (states it answers Q-024; six columns, three tables, two owners) |
| Q-025 | What happens to a `SETTLEMENT_ANOMALY` row afterwards | PROC-SETTLEMENT-ANOMALY-LIFECYCLE ("Q-025, answered" table: no API, no screen, no reader, no owner) |
| Q-027 | Is `/detect` authenticated; what triggers it | API-SETTLEMENT-ANOMALY (mechanism-by-mechanism auth table; "The missing caller" — four independent checks) |
| Q-028 | Inventory data model and its vocabulary bridge | SCH-INVENTORY (stock_item + RESTORE_LOG field tables, ERD, README bridge) |
| Q-029 | Which reason codes restock; where the rule is duplicated | PROC-INVENTORY-RESTOCK, DEC-INVENTORY-RESTOCK-BY-REASON (`main.py` enforces, `config.py` duplicates, tests assert the dead copy) |
| Q-030 | Which DB inventory-api connects to | SYS-INVENTORY ("Q-030 resolved": `sellflow_order` hardcoded against three documents saying `inventory`) |
| Q-031 | Does `/stock/restock` match what `InventoryClient` calls | API-INVENTORY ("Q-031" dimension-by-dimension table; no dimension matches) |
| Q-032 | inventory-api authentication | PROC-INVENTORY-AUTH (reproducible grep, exit status recorded) |
| Q-033 | Multi-line orders and warehouse-level stock | PROC-INVENTORY-RESTOCK ("Where the process loses information"), SCH-INVENTORY worked examples |
| Q-034 | What delivery-bff proxies; carrier-failure behaviour | SYS-DELIVERY, API-DELIVERY, PROC-DELIVERY-ERROR-HANDLING (the PREPARING table) |
| Q-035 | What the generated client assumes; still true? | CON-ORDER-DELIVERY ("This is the answer to Q-035" — seven assumptions, three never true) |
| Q-036 | Does delivery-bff hold persistent data | SYS-DELIVERY ("Q-036 resolved"), SCH-DELIVERY (why a classDiagram and not an ERD) |
| Q-037 | delivery-bff auth and ownership | PROC-DELIVERY-AUTH, SYS-DELIVERY "The ownership conflict", PROC-SELLFLOW-OWNERSHIP |
| Q-038 | Runtime and config; registry vs `package.json` | SYS-DELIVERY (Node 16/18 unresolved, no `engines`, every env var inert), PROC-DELIVERY-STATUS-SYNC |
| Q-039 | Full `order.cancelled` path and where it stops | PROC-SELLFLOW-CANCEL-MONEY-PATH (nine hops, four ranked breaks, line numbers, transaction map) |
| Q-040 | Order/Settlement split over `JUNGSAN_WANRYO` | CON-ORDER-SETTLEMENT, SYS-ORDER "Does NOT Own", PROC-ORDER-ORDER-MST-LIFECYCLE |
| Q-041 | Order↔Inventory restock contract, honoured? | CON-ORDER-INVENTORY (four-dimension wire-format table; "no request is ever issued") |
| Q-042 | Order↔Delivery contract; client staleness | CON-ORDER-DELIVERY ("That is the answer to Q-042": 1,411 days, six independent ways) |
| Q-043 | Team ownership; registry vs org chart vs code | PROC-SELLFLOW-OWNERSHIP (five-service reconciliation with verdicts) |
| Q-044 | Procedure v1.1 requirements; which steps unsupported | PROC-SETTLEMENT-CORRECTION, PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11 (clause-by-clause "Runs?" table — 1 of 12) |
| Q-045 | Backlog size and the evidence for it | RISK-SETTLEMENT-RECON-BACKLOG (4,127 / 188,851,520 KRW with five extraction caveats) |
| Q-046 | Terms meaning different things across services | GLOSSARY-SELLFLOW (disambiguation for 취소, 정산, 파트너, 상태, 사유 코드, 채널, SKU/SANGPUM_CD) |
| Q-047 | Documents describing cancel policy; which are contradicted | PAT-SELLFLOW-DOC-CODE-DRIFT (mechanism + shapes), RISK-SELLFLOW-DOC-DRIFT (instances), DEC-ORDER-SF2287, CON-ORDER-DELIVERY, PROC-INVENTORY-RESTOCK |
| Q-048 | Runtime topology of the five services | PROC-SELLFLOW-RUNTIME (processes, ports, schedules, the shared instance) |

### ~ Partially answered (5)

| Q | What is present | What is missing |
|---|---|---|
| Q-026 — model loading, versioning, last retraining | SYS-SETTLEMENT-ANOMALY and API-SETTLEMENT-ANOMALY fully cover loading (pickle at import scope, hardcoded path, `MODEL_PATH` unread) and versioning (no pin, no checksum, `/health` echoes `payload["version"]`) | The retraining date. The only evidence is a comment in `model/features.py` claiming none since 2024-11, which the artifacts correctly flag as "a comment, not a log", written seven months before the service went live. No training run, log or registry exists to check it against. The reef grades this itself as "Q-026, partially unresolved" and that is the right call |
| Q-049 — do long-lived environments match a clean V1–V24 run? | SCH-ORDER-MIGRATION-HISTORY proves the chain now applies to an empty database, gives the three `information_schema` / `flyway_schema_history` queries that would answer it, and names the three columns (`TEMP_FLAG`, `JEOKRIPGEUM`, `HALIN_GEUMAEK`) whose presence would prove a missed migration | The answer. It requires reading a live schema; V23's consolidated baseline was never written and `migration-issue.yml` runs `flywayInfo`, which reports applied versions rather than comparing DDL. The reef supplies the method and not the result |
| Q-050 — was the `RESTORE_LOG` audit trail deferred or overlooked? | PROC-INVENTORY-RESTORE-LOG-LIFECYCLE gives the complete factual half: six columns, two indexes, zero writers (seven grep hits, all definitional), no warehouse column, no affected-row count, and the `result` field the dataclass has and the table does not | The intent. Recorded as a known unknown: "No ticket, minute or policy document in sources/context mentions 복원 이력." Nothing in the corpus distinguishes deliberate deferral from oversight |
| Q-051 — do `ORDER_DELIVERY` / `ORDER_STATUS_HIST` hold rows? | SCH-ORDER and PAT-SELLFLOW-ORPHANED-COMPONENTS establish DDL in V1, entities mapping column for column, repositories declared and never injected, and no SQL naming either table in any of the five repos | Whether rows exist. Stated as a known unknown in both artifacts: with no reader and no writer, no behaviour would reveal the population either way |
| Q-052 — which queries were the V21/V24 indexes added for? | SCH-ORDER-MIGRATION-HISTORY records that V21 indexes `JUNGSAN_RUN_ID` (V14, added at 정산팀's request) and V24 prefix-indexes `GOGAEK_MEMO` (V2), that neither migration carries a comment, and that no query in any of the five repos touches either column | The queries. V21 and V24 are in the artifact's "undocumented half" — the migrations that "carry no comment at all". Whether an out-of-repo query exists cannot be settled from the artifacts |

### ✗ Not answerable (0)

Nothing in the question bank falls through entirely. Every question has an artifact that
engages with it directly.

### What the grading changed

The `status` field on the question bank was set by the process that built the reef and had
drifted badly. Re-derived independently:

| Was | Now | Questions |
|---|---|---|
| `unanswered` | full | Q-009, Q-036, Q-042 |
| `partial` | full | Q-010, Q-012, Q-019, Q-024, Q-025, Q-035, Q-043, Q-047 |
| `partial` | partial | Q-026 |
| `unanswered` | partial | Q-049, Q-050, Q-051, Q-052 |
| `answered` | full | the remaining 36 |

Eleven questions were under-rated by the reef's own bookkeeping and none was over-rated.
The most consequential were Q-009 (`unanswered`, when PROC-ORDER-AUTH is a 200-line answer
that names its own re-verification method) and Q-042 (`unanswered`, when CON-ORDER-DELIVERY
says in the body "That is the answer to Q-042"). The status field is not maintained by the
skills that write the artifacts.

### Gaps to explore

The five partials are the whole of the gap surface, and four of them are questions the reef
raised about itself during `/reef:update`. All four are **owner questions, not research
gaps** — they ask about production state or about a person's intent, and no amount of
further reading of the five repositories would close them.

1. **Q-049, Q-051 — production schema state.** These need one `information_schema` query per
   environment, which SCH-ORDER-MIGRATION-HISTORY already writes out verbatim. The reef
   cannot run it. *Next action:* route to the owner via `/reef:ask`; do not send `/reef:deep`
   at the migration directory, which has already been read line by line twice.
2. **Q-050, Q-052 — intent behind a deferral and behind two indexes.** Both were searched for
   across all five repos and all of `sources/context`; the corpus contains no ticket, comment
   or minute. *Next action:* owner question. Nothing in the artifact layer would improve.
3. **Q-026 — model retraining history.** The one partial that is not purely an owner
   question: if a training script, MLflow entry or artefact registry exists anywhere outside
   `settlement-anomaly`, it has not been looked for. *Next action:* a targeted `/reef:feed`
   or `/reef:source` pass over any data-team repository not currently indexed. If none
   exists, this becomes an owner question too.

No artifact is shallow enough to warrant `/reef:scuba`. For scale: the thinnest artifact in
the reef, SCH-DELIVERY, is deliberately thin because the service has no datastore, and it
argues that case explicitly rather than leaving a stub.

### Defects found in the test skill itself

Recorded because a test pass should test its own instrument.

1. **The rubric has no category for "the sources cannot answer this, and the reef proves
   it."** `/reef:test` offers full / partial / not-answerable, all of which are statements
   about reef coverage. Four of this reef's five partials are questions where the reef has
   done everything possible — exhaustive grep, the diagnostic query written out, the absence
   established — and the answer still lives in a production database or in a person's memory.
   Grading those the same as thin coverage penalises exactly the behaviour `methodology.md`
   asks for ("honest gaps beat confident lies"). A fourth rating — *answered as far as the
   sources permit* — would make the score mean something.
2. **The progress-bar rule and the headline count disagree.** The skill says
   "Progress: ... {N}/{total} questions answered" while the bar weights partials at one half.
   Here that is 47 (answered) against 49.5 (weighted). This report shows the bar from the
   weighted figure and the caption from the answered count, and says so; the skill should
   pick one.
3. **Step 6 tells the tester to rewrite `status` but never says to reconcile the fields the
   reef's other skills add.** Three questions in this bank carry `answered_by` and `answered`
   fields (Q-030, Q-031, Q-032) that no `/reef:test` step maintains, so they will drift the
   same way `status` just did.
4. **"Do not read source code" is the right rule and is under-specified.** It does not say
   whether `sources/` — the reef's own copy of the company documents — is in or out. It is
   the difference between testing the artifacts and testing the corpus, and this pass had to
   be told explicitly by the operator. The skill should state it.

---

## Phase B — accuracy against the grading key

Phase A above was written and closed before `ANSWER-KEY.md` was opened. Nothing in it was
revised afterwards.

### 1. Fragment recall — 26 / 26

Every fragment in the key's map is stated somewhere in `artifacts/**`, with a source.

| # | Fragment | Stated in |
|---|---|---|
| 1 | Cancel API does not check settlement state | PROC-ORDER-CANCEL, DEC-ORDER-SF2287-CANCEL-BLOCK-REMOVAL, API-ORDER, CON-ORDER-SETTLEMENT |
| 2 | `JUNGSAN_WANRYO` exists but is unchecked on the cancel path *(hard)* | DEC-ORDER-SF2287 (Consequences table marks it "Blocked after SF-2287? **No** / per 2021 policy **Yes**"), PROC-ORDER-CANCEL, PROC-ORDER-ORDER-MST-LIFECYCLE |
| 3 | Settlement selects on delivery completion, not cancellation *(hard)* | SCH-SETTLEMENT-FIELD-LINEAGE-SETTLEMENT-DTL, PROC-SETTLEMENT-DAILY-BATCH, CON-ORDER-SETTLEMENT — **with a correction, see §4** |
| 4 | Payout is irreversible | SYS-SETTLEMENT ("Irreversibility"), PROC-SETTLEMENT-DAILY-BATCH, RISK-SETTLEMENT, PROC-SELLFLOW-CANCEL-MONEY-PATH |
| 5 | The cancel event only moves outbox → queue | PROC-ORDER-EVENT-OUTBOX-LIFECYCLE, DEC-ORDER-OUTBOX-RELAY, API-SETTLEMENT-BATCH |
| 6 | **Nothing drains the queue** *(hard, absence)* | PROC-SETTLEMENT-CORRECTION, DEC-SETTLEMENT-CANCEL-CLAWBACK, PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE, PAT-SELLFLOW-ORPHANED-COMPONENTS, PROC-SELLFLOW-CANCEL-MONEY-PATH |
| 7 | Reason 03 does not restock | PROC-INVENTORY-RESTOCK, DEC-INVENTORY-RESTOCK-BY-REASON, SCH-INVENTORY |
| 8 | Reason 03 cost is borne by Sellflow *(non-code)* | SCH-ORDER (CancelReason table), GLOSSARY-SELLFLOW, PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE, CON-ORDER-INVENTORY |
| 9 | Correction owner = 정산팀 *(non-code, sole source)* | PROC-SELLFLOW-OWNERSHIP ("정산팀 owns the drain, as a manual procedure"), SYS-SETTLEMENT, PROC-SETTLEMENT-CORRECTION |
| 10 | Deliberate 2023 decision, forecast under 10/month | DEC-ORDER-SF2287, DEC-SETTLEMENT-CANCEL-CLAWBACK, DEC-SELLFLOW-2026-AUTOMATION-SIZING |
| 11 | The wiki is wrong (2021-03) | DEC-ORDER-SF2287, RISK-SELLFLOW-DOC-DRIFT, PAT-SELLFLOW-DOC-CODE-DRIFT (shape 1) |
| **11b** | **The wiki accurately describes `OrderCancelServiceV1`, which is still present and has no caller** *(the designed trap)* | SYS-ORDER, DEC-ORDER-SF2287, RISK-ORDER, PROC-ORDER-ORDER-CANCEL-LIFECYCLE, PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW — see §2 |
| 11c | A 2024-08 wiki comment asked and got no reply | DEC-ORDER-SF2287 (quotes 강태오, 2024-08-19), RISK-SELLFLOW-DOC-DRIFT, PAT-SELLFLOW-DOC-CODE-DRIFT |
| 12 | The API spec is wrong too (2022, documents the 409) | API-ORDER, CON-ORDER-DELIVERY, DEC-DELIVERY-GENERATED-CLIENT |
| 13 | AI already detecting, zero action taken | PROC-SETTLEMENT-ANOMALY-LIFECYCLE, SCH-SETTLEMENT-ANOMALY, RISK-SETTLEMENT-ANOMALY — **the reef disputes the first half, see §4** |
| 14 | 4,127 rows / 188,851,520 KRW / ~101 a month | RISK-SETTLEMENT-RECON-BACKLOG, PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE (mean 100.7, trailing 12mo 145.2), DEC-SELLFLOW-2026-AUTOMATION-SIZING |
| 15 | The current plan rests on a false premise | DEC-SELLFLOW-2026-AUTOMATION-SIZING (a whole artifact on precisely this) |
| 16 | 2025-03 handover left "check whether the queue drains" unresolved; the owner left | PROC-SETTLEMENT-CORRECTION, PROC-SELLFLOW-OWNERSHIP, DEC-SETTLEMENT-CANCEL-CLAWBACK |
| 17 | 2025-07 retrospective's cancellation-exclusion item still unticked *(absence)* | RISK-SETTLEMENT (Theme 4), PROC-SETTLEMENT-RUN-LIFECYCLE, SCH-SETTLEMENT-FIELD-LINEAGE, PROC-SELLFLOW-OWNERSHIP |
| 18 | "Only the schedule registration is left", 2023-04-24, repeated for 3 years *(hard)* | PROC-SETTLEMENT-CORRECTION ("The evidence chain, in order" — eight dated sources), PROC-SELLFLOW-OWNERSHIP |
| 19 | 재무기획팀 flagged the inconsistency in 2026-08 and got no answer | RISK-SETTLEMENT-RECON-BACKLOG, PROC-SETTLEMENT-CORRECTION, CON-ORDER-SETTLEMENT |
| 20 | Order and Settlement understand the automation differently (2026-06 kickoff) | CON-ORDER-SETTLEMENT, PROC-SETTLEMENT-CORRECTION, PROC-SELLFLOW-OWNERSHIP ("The belief gap, on the record") |
| 21 | Registry records the queue consumer as `TODO` *(absence)* | API-SETTLEMENT-BATCH, PROC-SELLFLOW-OWNERSHIP, SCH-SETTLEMENT-BATCH, CON-ORDER-SETTLEMENT, PAT-SELLFLOW-ORPHANED-COMPONENTS |
| 22 | v1.1 mandates a monthly review with no record of it happening | PROC-SETTLEMENT-CORRECTION (Step 2), PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11 |
| 23 | SF-4512 To Do and unassigned since 2023-04-24 *(hard)* | DEC-SETTLEMENT-CANCEL-CLAWBACK, PROC-SELLFLOW-OWNERSHIP ("the oldest open item in that export by more than three years"), RISK-SETTLEMENT-RECON-BACKLOG |
| 24 | The 2024 review was shelved for "low volume" against the backlog data | DEC-SELLFLOW-2026-AUTOMATION-SIZING, PROC-SETTLEMENT-CORRECTION-PROCEDURE-V03-VS-V11 |

**Misses: none.** Including all four fragments the key marks as provable only by absence
(#6, #11b, #17, #21) and all four it marks `hard` (#2, #3, #6, #11b, #18, #23).

One fragility worth naming even though it counts as a hit. **#11b rests on a single
sentence in a single artifact** — the last line of SYS-ORDER's "Domain Behavior Highlights":
"Its only remaining function is that the 2021 wiki still describes it accurately while
describing current behaviour inaccurately." Four other artifacts supply every component of
the claim (the class exists, it queries `SETTLEMENT_DTL`, it throws, it has zero callers,
the wiki states the old rule), but only that one sentence joins them into the conclusion the
key calls the maximum trap. Delete that line and a reader would have to assemble it
unaided.

### 2. The two designed traps

**(a) Does the reef state that the 2021 wiki accurately describes `OrderCancelServiceV1`, a
class nothing calls — rather than simply "the wiki is wrong"? — YES.**

> "**The deprecated V1 class is load-bearing only as documentation.** `OrderCancelServiceV1`
> is annotated `@Deprecated` with the comment 삭제 예정이나 배치에서 참조 가능성이 있어
> 남겨둠 ("scheduled for deletion but kept because a batch might reference it"), dated
> 2023-04-21 by 박성민. A repo-wide grep shows no batch references it — and no service,
> either: the class carries no `@Service`/`@Component`, so its `@Autowired DataSource` would
> never be injected even if something did call it. **Its only remaining function is that the
> 2021 wiki still describes it accurately while describing current behaviour
> inaccurately.**"
> — `artifacts/systems/sys-order.md`

The supporting halves are independently stated. DEC-ORDER-SF2287 quotes the surviving check
verbatim — "`SELECT COUNT(1) FROM SETTLEMENT_DTL WHERE ORD_NO = ?` followed by `throw new
IllegalStateException("정산 완료된 주문은 취소할 수 없습니다. ordNo=" + ordNo)`". RISK-ORDER
draws the consequence the key warns about: "**Two contradictory cancel policies are in the
tree.** `OrderCancelService` permits cancelling settled orders; `OrderCancelServiceV1`
forbids it." And PROC-ORDER-ORDER-CANCEL-LIFECYCLE proves the zero-caller half three ways,
including the one a grep alone would miss: the class has no Spring stereotype, so it could
not be injected even if something tried.

The reef did not fall into the trap in either direction. It does not conclude "the wiki is
wrong" and stop, and it does not conclude from a code search that a settlement check is live.

**(b) Does it establish that nothing drains the correction queue, by absence rather than by
assertion? — YES, with the search recorded.**

> "**Break 1, proven by grep.** `grep -rn "CancelReconciler\|reconcileCancellations\|
> loadPending" ../sellflow/repos` returns five lines, all inside `CancelReconciler.java`
> itself: the class declaration (20), its logger (22), `loadPending` (26),
> `reconcileCancellations` (32) and the self-call at 33. No caller, no test, no configuration
> reference in any of the five repos"
> — `artifacts/processes/proc-sellflow-cancel-money-path.md`

PAT-SELLFLOW-ORPHANED-COMPONENTS makes the method a policy rather than a one-off — "This
artifact asserts absences. Each row of the table names the grep that produced it so a reader
can re-derive the finding rather than trust it" — and its agent guidance instructs the same:
"Run the caller grep and say what it returned... An assertion of absence without the command
behind it is not usable by the next reader."

The reef then goes past what the key asks for, and the extra is the more useful half. It
establishes **three further independent breaks** on the same path, so that "schedule the job"
is visibly not the fix:

- `SETTLEMENT_ADJUSTMENT`, the reconciler's first write target, has no `CREATE TABLE` in any
  of the five repos — one grep hit, the INSERT itself.
- The next month's settlement reader joins `ORDER_MST` and `ORDER_DTL` only, so a queue row
  cannot affect any later payout even if an adjustment existed.
- `SETTLEMENT_ANOMALY`, the independent second signal, has no reader either.

PROC-SELLFLOW-CANCEL-MONEY-PATH §"What fixing break 1 alone would do" traces the consequence
line by line: the job would load all 4,127 rows unpaginated and fail with `ER_NO_SUCH_TABLE`
on the first one, every interval, forever. That is a materially better answer than "nothing
drains the queue."

### 3. Honest unknowns — 4 / 4 correctly declined, 0 false claims

| The key's undeterminable | Reef's position | Verdict |
|---|---|---|
| Whether any of the 4,127 were corrected manually | Declared as a known unknown in RISK-SETTLEMENT-RECON-BACKLOG, PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE, PROC-SETTLEMENT-CORRECTION and PROC-SELLFLOW-CANCEL-MONEY-PATH. The reef carries the export README's own caveat — only `STATUS='PENDING'` rows were returned — and the handover's statement that manual work leaves no system record, then writes the one query that would settle it (`SELECT STATUS, COUNT(*) ... GROUP BY STATUS`) and notes nobody has run it | **Correctly declines** |
| Whether `SETTLEMENT_ADJUSTMENT` exists | Declared as a known unknown in **six** artifacts (SYS-SETTLEMENT, SCH-SETTLEMENT-BATCH, PROC-SETTLEMENT-CORRECTION, PROC-SETTLEMENT-CANCEL-RECON-QUEUE-LIFECYCLE, PROC-SETTLEMENT-FLOW-CATALOG, PROC-SELLFLOW-CANCEL-MONEY-PATH, PAT-SELLFLOW-ORPHANED-COMPONENTS). RISK-SETTLEMENT makes it recommended action #1: "One `SHOW CREATE TABLE` answers it" | **Correctly declines** |
| Per-partner contract fee rates | RISK-SETTLEMENT: "How many partners hold a premium or promotional contract is unknown — `PARTNER_CONTRACT` was not queried, and no export of it exists". SCH-SETTLEMENT-FIELD-LINEAGE says the same and supplies the overcharge query rather than an overcharge figure | **Correctly declines** |
| Whether the settlement batch actually runs daily | Declined where it counts, with a caveat — see below | **Correctly declines, with one inconsistency** |

**No fragment is claimed with false certainty.** That is the most important result in this
section, and it holds under a targeted search for over-claiming.

The fourth item is worth a precise note rather than a pass or fail. API-SETTLEMENT-BATCH
handles it exactly right, and says so as instruction:

> "**When asked what runs in production, separate three claims.** (a) `QuartzConfig` registers
> two triggers — verified from code. (b) The registry says `schedule: 매일 02:00 KST` — a
> document, consistent with the cron. (c) What is actually deployed — unverifiable from here,
> because `deploy.sh` and any prod profile are outside the repo. Keep (c) in `known_unknowns`
> rather than asserting it."

Three artifacts then assert (c) anyway, mildly:

- CON-SETTLEMENT-ANOMALY-SETTLEMENT-BATCH: "The producer side, by contrast, runs.
  `SETTLEMENT_DTL` is written nightly."
- PAT-SELLFLOW-CROSS-SERVICE-DATA-FLOW: "`SANGTAE_CD` is overwritten nightly by another
  division's batch".
- PROC-SETTLEMENT-FLOW-CATALOG: F1 and F2 marked "Runs? **Yes**", with no caveat in that
  artifact's `known_unknowns` about production execution.

This is not dishonest certainty — the inference has a stated empirical basis elsewhere
(PROC-SELLFLOW-CANCEL-MONEY-PATH: "no INSERT exists in any of the five repos, yet
`SETTLEMENT_DTL` is demonstrably populated", which follows from 41 months of queue rows that
can only exist when a detail row already did). But the reef contains both the over-statement
and its own correction, and a reader who meets the flow catalogue first will carry a firmer
claim than the evidence supports. **Recommended: add the production-execution caveat to
PROC-SETTLEMENT-FLOW-CATALOG's `known_unknowns` and soften the two "nightly" sentences.**
It is the only place in 80 artifacts where the reef's stated standard and its own prose
diverge.

### 4. Where the reef and the key disagree

Three divergences. In all three the reef is better evidenced than the key, which is worth
saying plainly: these are not errors to correct in the reef.

**(a) Fragment #3, and the key's implicit conclusion from it — the reef corrects the key.**

The key says settlement "extracts by delivery-completion date only — cancellation is
irrelevant". Taken at face value that implies cancelled orders get settled. The reef states
the fragment and then demonstrates the opposite:

> "The javadoc states an intent the SQL beneath it does not implement... The SQL selects on
> `SANGTAE_CD = 'BAESONG_WANRYO' AND DATE(UPD_DTM) = ?`, and `OrderCancelService.cancel()`
> calls `OrderMst.chwiso()`, which sets `SANGTAE_CD` to `CHWISO` and `UPD_DTM` to now in the
> same write. Both predicates therefore fail for a cancelled order... **The exclusion is real
> but incidental — a side effect of the status write, not a rule.**"
> — `artifacts/contracts/con-order-settlement.md`

This matters for the keystone answer, because it changes what the 188M figure is. The reef
follows it through: the ack-before-insert defect drops events silently, but the dropped
population belongs to orders that will never be paid, so "two defects cancel, and neither was
designed to". The reef's own `log.md` records this as a deliberate correction pass over 25
artifacts, not an accident.

**(b) Fragment #13 and the key's headline — the reef disputes that detection is running.**

The key's correct answer states "이상 탐지 모델은 1년 넘게 같은 건들을 지적해 왔으며, 아무도
보지 않았다" — the model has been flagging the same cases for over a year and nobody looked.
The reef agrees completely on "nobody looked" and declines to agree on "has been flagging":

> "Reading all ten of its files produces an unusual risk profile: almost nothing here can go
> wrong in the normal sense, because nothing here demonstrably runs. The endpoint has no
> caller. The scheduler it claims does not exist. The model file it loads is not in the
> repository. ... **it has produced no evidence of ever having detected anything.**"
> — `artifacts/risks/risk-settlement-anomaly.md`

Three independent grounds, each sourced: no caller for `POST /detect` in any of the five
repos (verified four ways); `model/artifacts/iforest_v3.pkl` absent from the repo and loaded
at module import scope, so a missing artefact fails startup rather than a request; and
`detect()` reading `r["sangtae_cd"]` from a `DictCursor` row keyed `SANGTAE_CD`, which would
`KeyError` on the first row. The reef is careful about the last one — SCH-SETTLEMENT-ANOMALY
labels it "an observation, not a confirmed defect, because it was not verified at runtime"
and files it in `known_unknowns`.

This is the reef being more rigorous than its own answer key, and it strengthens the keystone
conclusion rather than weakening it: if detection has never run, the organisation has not two
detectors of the cancel-after-settlement problem but one, and still no consumer for it.

**(c) Fragment #14 — the reef refuses to call 188,851,520 KRW an exposure.**

The key lists "1.89억원" as measured backlog. The reef records the number and then shows it
is `4,127 × 45,760` exactly, in all 41 months, remainder zero — a flat per-case estimate, not
a sum of amounts paid, against a column (`EXPECTED_AMT`) no migration in any repo defines. It
carries the export README's instruction — "재추출 없이 그대로 인용하지 말 것" — everywhere the
figure appears, and supplies the `JOIN SETTLEMENT_DTL` that would produce a real number. A
refinement of the key rather than a contradiction, and the right one for a figure a finance
team is being asked to book as a liability.

**Adjacent to the map, one small miss.** The key's Second Finding S5(2) notes that
settlement-batch's `DateUtil` carries a comment saying it was copied from order-service and
then modified. The reef records the behavioural divergence in four places (GLOSSARY-SELLFLOW,
RISK-SETTLEMENT, PROC-SETTLEMENT-DAILY-BATCH, PROC-SETTLEMENT-RUN-LIFECYCLE) and that the
settlement version is dead code, but nowhere quotes the copied-from provenance comment. The
divergence is captured; its origin is not. S5(1) and S5(3) are fully covered — and on S5(1)
the reef is ahead of the key, noting that `RESTORE_LOG` now has an `ORD_NO` column, which
contradicts the README's "보관하지 않음" the key relies on.

---

## Verdict

| Measure | Result |
|---|---|
| Questions fully answered | **47 / 52** |
| Partially answered | 5 — four are owner questions the reef raised itself; one (Q-026) has a genuinely unrecoverable half |
| Not answerable | 0 |
| Fragment recall | **26 / 26**, including all four provable only by absence |
| Trap (a) — wiki accurately describes the uncalled `OrderCancelServiceV1` | **Passed**, explicitly, though on one sentence |
| Trap (b) — nothing drains the queue, proven by absence | **Passed**, with the grep recorded, plus three further breaks the key does not ask for |
| Honest unknowns | **4 / 4 declined.** No false claim of certainty anywhere |
| Contradictions of the key | 3, all in the reef's favour and all better evidenced |

The reef's own `status` bookkeeping understated it on eleven questions and overstated it on
none. The one defect found is a presentational inconsistency about whether the settlement
batch is known to run in production — three sentences assert what the reef's own agent
guidance tells readers not to assert.
