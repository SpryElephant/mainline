---
description: Hand a Feature from Review to QA — run the `/ready-for-qa` checks, move Phase, assign QA, notify and record.
argument-hint: <issue-number> [assignee]
allowed-tools: Bash(gh:*), Read, Grep, Glob
---

Run the **`/ready-for-qa` handoff — Review → QA (Reviewer → QA)** for issue $1.

Read `.github/mainline.json` and `.github/project-fields.json` for the project parameters and the
field/option IDs. If either is missing, stop and say which.

## 1. Run the checks

The checklist is **`/mainline-review-station` step 5**. Read it there and verify each line.

What each line means in practice:

- The automated review ran, and **every finding is resolved or waived with a written reason**. A
  waiver with no reason fails the check — "looks fine" is not a reason.
- The security pass is clean, or its findings are waived with reasons, per the triage policy in the
  project charter. A Critical finding stops the line rather than being waived.
- **`/mainline-quality-gate` green on the merge commit**, not just on the branch.
- **A named human sign-off is recorded** where an auditor can find it. A review approved by nobody in
  particular is not a signature.

## 2. Block on failure

If any check fails: report which one, quote the finding or the missing reason, and **stop**. Do not
move `Phase`, do not assign, do not notify.

If the same finding is being waived repeatedly, say so — the rule is miscalibrated, and that is an
`/mainline-improvement-loop` entry rather than a fourth waiver.

## 3. Move, assign, notify, record

Only when every check passed:

- Set `Phase` to **QA**.
- Assign `$2`, or `assignees.qa` from the config. If neither resolves, stop.
- Notify with the review findings and their dispositions, the security pass result, and the sign-off
  attached.
- Comment on the issue recording the result: the handoff, who signed, who it went to, and the time.

Report what moved and what was attached.

## 4. Stop

**This command is the end of the sitting.** Report and stop.

Do not begin QA. Do not run the suite against staging, and do not file QA defects. That is the next
station, and it starts in a new session — including when the next owner is the person who just ran
this command. A handoff to yourself is still a handoff: the value is that the next station's checks
meet the work from outside, not that a different name appears on the card.

If you are the next owner, say so, name the command or skill that station is worked with, and end
the session there.
