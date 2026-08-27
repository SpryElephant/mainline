---
name: mainline-review-station
description: The Review station — run the automated review and the security pass on a pull request, apply human judgment to the findings, fix or waive each one with a written reason, record the named sign-off, and pass the H3 handoff into QA. Use when a pull request arrives for review, and whenever a waiver is being decided.
---

# Review station

Every change is reviewed by another **person**. Keep the person — drop the assumption that they must
read every line. A human reading a large diff carefully is slower and less thorough than a tool.
Your judgment is applied to the *findings*, and your name is the accountable signature underneath.

## 0. Check what arrived

`/mainline-quality-gate` green in CI on the branch, acceptance scenarios passing with validation evidence, no
unrelated changes. If the gate is not green it should not have reached you — send it back.

## 1. Run the automated review

An automated review at high effort on the PR diff. It reads the code; you read its output.

## 2. Run the security pass

`/mainline-security-gate`, PR-bound dimensions: SAST, secrets, dependencies, IaC. Images and runtime posture
bind on the release path, not here.

## 3. Decide each finding

Fix it, or waive it **with a written reason**. "Looks fine" is not a reason — the reason is the
artefact, because it is what tells the next person why this was acceptable here.

Severity follows `/mainline-security-gate`'s triage policy: Critical stops the line, High is fixed before the
next release, Medium becomes a dated `Platform` item, Low is batched, a false positive is suppressed
inline with a reason and never globally.

**A repeated waiver is a miscalibrated rule.** Waiving the same finding a third time is an
`/mainline-improvement-loop` entry filed with `/mainline-file-finding`, not a fourth waiver.

## 4. Ask what no tool asks

- Does this do what the requirement actually said?
- Does it belong in this module, per its contract?
- Will it be obvious in six months? Is there a simpler shape?

## 5. Pass H3

- [ ] Automated review run; findings resolved or waived with a written reason.
- [ ] Security pass clean.
- [ ] `/mainline-quality-gate` green on the merge commit.
- [ ] **Named human sign-off recorded** where an auditor can find it.
- [ ] **Signed** by you · **assigned** to QA · **notified**.

## Failure modes

- **Rubber-stamping the tool.** The tools produce findings, not decisions. Ungated approval of a
  findings list is not a review.
- **Reading the diff instead of the findings.** Slower, less thorough, and it crowds out step 4 —
  the only part of the job a tool cannot do.
- **Waivers with no reason.** A waiver with no reason is a shrug, and it teaches the next reviewer
  that the rule is optional.
- **Reviewing a red gate.** Every minute spent here is spent on a change that may not survive.

## Relationships

- **`/mainline-security-gate`** — step 2; the dimensions, tools and triage policy.
- **`/mainline-development-workflow`** — hands off at H2. Send incomplete handoffs back.
- **`/mainline-e2e-suite`** — receives at H3.
- **`/mainline-file-finding`** — how a finding you are not fixing becomes a tracked item.
- **`/mainline-improvement-loop`** — where repeated waivers and escaped defects go.
