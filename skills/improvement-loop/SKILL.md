---
name: improvement-loop
description: Turn an escape — a defect, gap or incident that reached a station it should never have reached — into an amended check, by finding the earliest handoff that could have caught it and changing that one. Covers the escape ledger, the four flow metrics, and when to escalate from amending a check to redesigning a station. Use whenever something escaped, after an incident review, and at the monthly metrics read.
---

# Improvement loop

Something reached a station it should never have reached. **That is a defect in the process, not
only in the code.** Fixing the code and moving on guarantees the same class of miss recurs, because
nothing about the line changed.

**Owner: the lead.** Run it whenever something escapes.

## The loop

1. **Name the escape.** What was wrong, and where it surfaced. One sentence.
2. **Find the earliest handoff that could have caught it.** Not the closest — the earliest. A missing
   requirement discovered in QA escaped H1, not H3. Amending H3 to catch missing requirements is how
   checklists become long and useless.
3. **Amend that check**, in this order of preference:
   - **A tool** that fails automatically. Always this if it is possible.
   - **A checklist line** on the handoff, if a tool cannot express it.
   - **Nothing.** Some things are genuinely one-offs. Record the decision not to change anything —
     that stops the same debate recurring.
4. **File the amendment as a `Platform` work item** (`file-finding`) — owner and date — unless it is a one-line edit
   you are making right now. An agreed change with nobody assigned is a change that did not happen,
   and this is the step most often skipped.
5. **Record it in the ledger.**
6. **Watch for repetition.** If the same check is amended three times, the check is not the problem —
   the station is. Escalate to the station's design rather than adding a fourth line.

## Rules

- **Never blame the person.** A person who missed something was working a check that did not require
  them to see it. Change the check.
- **Prefer subtraction.** A handoff with fifteen checklist lines gets skimmed, which means it has
  zero. If you add a line, look hard for one to remove or automate.
- **The amendment ships with the retro.** Edit the playbook that day.

## The escape ledger

`docs/escape-ledger.md`, one row per escape:

| Date | What escaped | Surfaced at | Should have been caught at | Change made |
|---|---|---|---|---|

The ledger is the evidence that the line is improving, and the first thing to show someone who asks
what they are paying for. **Keep it anonymous enough to share** — describe the miss, never the
account.

## What feeds it

| Source | Entry looks like |
|---|---|
| A defect QA found that Review should have caught | Escape at H3 or earlier |
| A requirement gap found in Build | Escape at H1 |
| An incident (`observability`) | The missing signal, plus the earliest check that could have caught the defect |
| A repeated waiver (`review-station`) | A miscalibrated rule, not a fourth waiver |
| A growing E2E quarantine list (`e2e-suite`) | The suite is dying; that is a station problem |
| A defect traced to emulator drift (`local-stack`) | A missing contract test against the real service |

## What to measure

Four numbers, reviewed monthly. Each tells you where to look next.

| Metric | What it tells you | Read it from |
|---|---|---|
| **Lead time per station** | Where work waits. A station always full is understaffed or its exit condition is wrong. | `Phase` transition timestamps on the board |
| **Handoff pass rate** | Which handoff bounces most. A low rate means the upstream station is not finishing its job. | Handoff command runs: moved vs. blocked |
| **Escape rate by handoff** | Which check is not biting. This is the one that drives amendments. | The ledger |
| **Rework rate** | How often work moves backwards. The honest measure of whether the line works. | Backward `Phase` transitions |

Do not add a fifth without removing one. Metrics nobody reads are worse than no metrics — they
create the impression of measurement.

## Failure modes

- **Fixing the closest check instead of the earliest.** Every checklist grows, none of them bite.
- **The retro with no amendment.** If a retro produces no changed check, it was not honest.
- **Amendments with no owner or date.** The review happened; nothing changed.
- **A ledger nobody writes in.** Then the line is a document, not a system.

## Relationships

- **`review-station`**, **`e2e-suite`**, **`observability`**, **`local-stack`** — the four commonest
  sources of entries.
- **`quality-gate`** / **`security-gate`** — where a tool-shaped amendment lands.
- **`file-finding`** — how an amendment becomes a tracked item.
- **`pmi-github-project`** — amendments are `Platform` work items on the board.
