---
description: Hand a Slice from Gate to Review — run the `/ready-for-review` checks, move Phase, assign the reviewer, notify and record.
argument-hint: <issue-number> [assignee]
allowed-tools: Bash(gh:*), Read, Grep, Glob
---

Run the **`/ready-for-review` handoff — Gate → Review (Developer → Reviewer)** for issue $1.

Read `.github/mainline.json` and `.github/project-fields.json` for the project parameters and the
field/option IDs. If either is missing, stop and say which.

## 1. Run the checks

The checklist is **`/mainline-development-workflow` step 4**. Read it there and verify each line.

What each line means in practice:

- **`/mainline-quality-gate` green in CI on the branch** — check the actual run, not the last local result
  (`gh pr checks`). A gate that has not run is not a gate that passed.
- Every acceptance scenario passing, with **validation evidence on the ticket** — validated against a
  running stack, not "the code looks right."
- Entailed scenarios from the design step either implemented or sent back to Product.
- No unrelated changes; behavior-preserving refactors in their own commits.
- Findings filed (`/mainline-file-finding`), not carried in someone's head.

## 2. Block on failure

If any check fails: report which one, quote the evidence you found or could not find, and **stop**.
Do not move `Phase`, do not assign, do not notify.

A red gate is the commonest failure here, and it must never be waved through. If a threshold is
wrong, that is a `Platform` item and an `/mainline-improvement-loop` entry, not a reason to hand off.

## 3. Move, assign, notify, record

Only when every check passed:

- Set `Phase` to **Review**.
- Assign `$2`, or `assignees.review` from the config. Never the author — Review is by another person.
  If neither resolves, or the only candidate is the author, stop.
- Notify with the PR link, the gate output and the validation evidence attached.
- Comment on the issue recording the result: the handoff, who signed, who it went to, and the time.

Report what moved and what was attached.
