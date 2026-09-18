# sellflow-reef

A knowledge layer over [sellflow](https://github.com/eunji-jessi-jung/sellflow),
built with [reef](https://github.com/eunji-jessi-jung/reef).

Eighty interlinked artifacts. Every claim cites the file it came from. Every gap
the sources could not close is written down rather than smoothed over.

```
SYS-  5   what each service does
SCH-  7   what the data means
API-  5   what you can call
PROC- 28  what happens when
DEC-  9   why it was built that way
CON-  5   what two systems agreed on
RISK- 8   what could go wrong
PAT-  7   what keeps happening
GLOSSARY- 6
```

## Start here

1. **[CLAUDE.md](CLAUDE.md)** — the map. Written for an agent; readable by a person.
2. **[artifacts/contracts/con-order-settlement.md](artifacts/contracts/con-order-settlement.md)**
   — the cancellation path from Order to Settlement, and where it stops. The
   highest-value artifact here.
3. **[artifacts/processes/proc-sellflow-cancel-money-path.md](artifacts/processes/proc-sellflow-cancel-money-path.md)**
   — the same path traced line by line, nine hops, four independent breaks.
4. **[artifacts/patterns/pat-sellflow-doc-code-drift.md](artifacts/patterns/pat-sellflow-doc-code-drift.md)**
   — why the documents in that company are wrong, in seven recognisable shapes.
   The one artifact here that is not really about sellflow.

The whole thing is wikilinked. Open the directory as an Obsidian vault to see the
graph.

## What the reef found

The brief was to work out how sellflow's settlement-correction process runs and
what to automate. The answer is that there is nothing to automate: the process
described by the company's own approved procedure has never executed. A queue has
accumulated for 41 months, an anomaly detector has been flagging the same
population into a table nobody reads, and the one component that would move the
money has no scheduler and no caller.

Reaching that required the reef to do something a search cannot: **prove absences.**
`grep` returning only a definition and no call site is the evidence, and the
artifacts state it in that form.

It also required not stopping at the first agreement. The company wiki describes
a settlement check that the code really does contain — in a `@Deprecated` class
nothing invokes. Code search and document search independently produce the same
wrong answer, and they corroborate each other.

## What it does not claim

The headline number, 188,851,520 KRW, is marked **inferred**, not measured, in
every artifact that uses it: the export's amount column is a flat per-case
estimate, not a sum of amounts actually paid.
[RISK-SETTLEMENT-RECON-BACKLOG](artifacts/risks/risk-settlement-recon-backlog.md)
carries five caveats on that figure and tells you to re-extract before citing it.

532 `known_unknowns` are recorded across the 80 artifacts.
**[.reef/questions-for-owner.md](.reef/questions-for-owner.md)** turns the ones
only a human can settle into 67 ranked questions, each stating what it would
unblock and what was already checked — so the person who knows can answer without
repeating the search.

Every artifact is `status: draft`. That is not incompleteness, it is the contract:
`active` means a domain expert confirmed it, and none has. The reef was built with
nobody to ask.

## Verifying it

**[ANSWER-KEY.md](ANSWER-KEY.md)** is the grading key for the fixture — 26 fragments
of the correct answer and the file each one lives in, plus the four questions that
are genuinely undeterminable from the sources. It is a spoiler; skip it if you want
to work the brief yourself.

## Reproducing it

```bash
git clone https://github.com/eunji-jessi-jung/sellflow.git
git clone https://github.com/eunji-jessi-jung/sellflow-reef.git
cd sellflow-reef
python3 <plugin>/scripts/reef.py health --reef .
```

The two repositories must sit side by side — `.reef/project.json` points at
`../sellflow/repos/*`. State is committed, so `diff`, `health` and `update` work
on a fresh clone: change something under `sellflow/` and the reef will tell you
which artifacts stopped being true.

It was built with `/reef:init` → `/reef:snorkel` + `/reef:source` → `/reef:scuba`
→ `/reef:deep` → `/reef:test`, then `/reef:update` after a correction pass on the
sources. [log.md](log.md) has the sequence with timestamps.

## Everything here is fictional

sellflow is an invented company. See [DISCLAIMER.md](DISCLAIMER.md).

## Licence

Apache-2.0. See [LICENSE](LICENSE).
