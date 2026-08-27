---
description: Hand a Slice from QA to Release — run the `/ready-for-release` checks, move Phase, assign the release approver, notify and record.
argument-hint: <issue-number> [assignee]
allowed-tools: Bash(gh:*), Read, Grep, Glob
---

Run the **`/ready-for-release` handoff — QA → Release** for issue $1.

Read `.github/mainline.json` and `.github/project-fields.json` for the project parameters and the
field/option IDs. If either is missing, stop and say which.

## 1. Run the checks

The checklist is **`/mainline-e2e-suite`, "Pass `/ready-for-release`"**. Read it there and verify each line.

What each line means in practice:

- The suite is **green on staging** — the actual run, with its link.
- New coverage is **merged into the permanent suite**, shipping *with* this release rather than
  after it. A test that lands next release is a test that did not gate this one.
- Open defects are either fixed, or **accepted with a named person and a date**. An unowned accepted
  defect fails the check.
- Nothing newly quarantined without a ticket. If the quarantine list grew, say by how much — a
  growing list is an `/mainline-improvement-loop` entry.

Defects found this round should already be filed against the requirement they violate; anything
citing no requirement goes to Product as a missing requirement, not to Release.

## 2. Block on failure

If any check fails: report which one, quote the evidence, and **stop**. Do not move `Phase`, do not
assign, do not notify.

## 3. Move, assign, notify, record

Only when every check passed:

- Set `Phase` to **Release**.
- Assign `$2`, or `assignees.release` from the config. If neither resolves, stop.
- Notify with the suite result, the tests added this round, and the list of accepted defects with
  their owners and dates.
- Comment on the issue recording the result: the handoff, who signed, who it went to, and the time.

Report what moved and what was attached.
