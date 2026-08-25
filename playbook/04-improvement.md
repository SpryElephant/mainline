# The improvement loop

Something reached a station it should never have reached. A defect surfaced in QA that Review should
have caught. A requirement gap surfaced in Build that H1 should have caught. Something reached
production that nobody caught at all.

**That is a defect in the process, not only in the code.** Fixing the code and moving on guarantees
the same class of miss recurs, because nothing about the line changed.

## The loop

**Owner: the lead.** Run it whenever something escapes.

1. **Name the escape.** What was wrong, and where it surfaced. One sentence.
2. **Find the earliest handoff that could have caught it.** Not the closest — the earliest. A missing
   requirement discovered in QA escaped H1, not H3. Fixing H3 to catch missing requirements is how
   checklists become long and useless.
3. **Amend that check**, in this order of preference:
   - **A tool** that fails automatically. Always this if it is possible.
   - **A checklist line** on the handoff, if a tool cannot express it.
   - **Nothing.** Some things are genuinely one-offs. Record the decision not to change anything —
     that is a legitimate outcome and writing it down stops the same debate recurring.
4. **File the amendment as a `Platform` work item** — owner and date — unless it is a one-line edit
   you are making right now. An agreed change with nobody assigned is a change that did not happen,
   and this is the step that most often gets skipped.
5. **Record it in the ledger** below.
6. **Watch for repetition.** If the same check is amended three times, the check is not the problem —
   the station is. Escalate to the process map rather than adding a fourth line.

## Rules

- **Never blame the person.** A person who missed something was working a check that did not require
  them to see it. Change the check.
- **Prefer subtraction.** A handoff with fifteen checklist lines gets skimmed, which means it has
  zero. If you add a line, look hard for one to remove or automate.
- **The amendment ships with the retro.** A change agreed and not made is a change that did not
  happen. Edit the playbook that day.

## Escape ledger

| Date | What escaped | Surfaced at | Should have been caught at | Change made |
|---|---|---|---|---|
| *2026-08* | A feature shipped without enterprise SSO. Enterprise tenancy was assumed by everyone and written down by no one, so nothing downstream could check it — downstream can only check what upstream wrote. | Production | **H1** | Added the NFR line to H1: auth and SSO, **tenancy**, performance, compliance, retention — each stated or explicitly marked N/A. |

The row above is a worked example, kept because it is the clearest one we have. A project starts its
own ledger empty.

Add a row every time. The ledger is the evidence that the line is improving, and it is the first
thing to show someone who asks what they are paying for.

**Keep it anonymous enough to share.** Describe the miss, not the account. "A feature shipped without
enterprise SSO" teaches everything the entry needs to teach; naming the client teaches nothing and
makes the ledger unshareable.

## What to measure

Four numbers, reviewed monthly. Each one tells you where to look next.

| Metric | What it tells you |
|---|---|
| **Lead time per station** | Where work waits. A station that is always full is understaffed or its exit condition is wrong. |
| **Handoff pass rate** | Which handoff bounces most. A low rate means the upstream station is not finishing its job. |
| **Escape rate by handoff** | Which check is not biting. This is the one that drives amendments. |
| **Rework rate** | How often work moves backwards. The honest measure of whether the line is actually working. |

Do not add a fifth without removing one. Metrics nobody reads are worse than no metrics, because they
create the impression of measurement.
