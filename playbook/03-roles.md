# Role cards

One card per role. If you are new to a project on Mainline, read the overview, then your card. That
should be enough to do the job.

---

## Product

**You own:** Inbox, Discovery, Requirement. **You sign:** H1 — `/h1`.
**Skills:** `/mainline-requirement-workflow`, `/mainline-product-discovery`, `/mainline-ui-exploration`.

**Your day.** Take the top card in Inbox. Ask the only question that matters at triage: *can somebody
write the Gherkin?*

**If yes** — write the `.feature` file. One per Slice. Scenarios say what a person achieved, not what
the screen did. Use the client's words, from the glossary.

**If no** — do not write it anyway. Open a Spike and run `/mainline-product-discovery`. Name one riskiest
assumption, fix a timebox in days, pick the cheapest prototype that can falsify it, put it in front
of a real participant, and record the seven tags live. Come back with a discovery record. When the
open question is visual direction rather than what to build, the medium is `/mainline-ui-exploration`.

**Before you hand off (H1),** walk the checklist in `02-stations.md`. The line that catches the most
is the NFR line: auth and SSO, **tenancy**, performance, compliance, retention — stated or explicitly
N/A. "Nobody mentioned it" is how enterprise SSO gets missed.

**Things that are your call, not the developer's:** what the client meant, priority, what is out of
scope, whether an ambiguity is worth resolving before building.

**When Design sends scenarios back to you,** that is the system working. The domain model found
obligations the requirement did not state. Add them.

---

## Developer

**You own:** Design, Build, Gate. **You sign:** H2 — `/h2`.
**Skills:** `/mainline-development-workflow`, `/mainline-domain-modeling`, `/mainline-quality-gate`, `/mainline-local-stack`, `/mainline-file-finding`.

**Your day.** A Slice arrives in Design, assigned to you, with the `.feature` file, the prototype and
the screenshots attached. You should not have to ask anyone anything to start.

1. **Design.** Read `domain/` first — does this extend a context or introduce one? Model it in Tonto,
   validate, derive the aggregates and boundaries, write the module contracts. Send the entailed
   scenarios back to Product. Skip this only for a bug fix or a change that fits existing contracts.
2. **Build.** Implement to the scenarios. Run the full stack locally and validate each acceptance
   criterion against a running system. Run the gate continuously — not once at the end.
3. **Gate.** Green, or not done. Never lower a threshold to pass; if the threshold is wrong, that is
   a conversation, not an edit.

**When you find something you are not fixing** — a bug, a risk, a missing rule — file it *now*, from
inside your session: ticket created, fields set, assigned, notified. One command. Do not carry it in
your head to the end of the task; that is where findings go to die.

**One feature is one change.** You are a full-stack developer on Mainline. If the repo layout is
fighting that, say so — it is a step-6 problem, not a personal one.

**Refactors are separate.** Behavior-preserving work — `/mainline-refactoring` for mechanical moves,
`/mainline-refactor-smells` for structural cleanup — lands as its own gate-green commit, before the feature
work that depends on the new shape. Never mixed in.

---

## Reviewer

**You own:** Review. **You sign:** H3 — `/h3`. **Skills:** `/mainline-review-station`, `/mainline-security-gate`.

**You are not reading the diff line by line.** The tools do that better than you. You are the
judgment layer on top of them, and the accountable signature underneath.

**Your day.** A PR arrives assigned to you, gate already green — if it is not green, it should not
have reached you; send it back.

1. Let the automated review and the security pass run.
2. Read the findings. For each: fix it, or waive it **with a written reason**. The reason is the
   valuable artifact — it is what tells the next person why this was acceptable here.
3. Ask the questions no tool asks. Does this match what the requirement actually said? Does it belong
   in this module, per its contract? Will this be obvious in six months? Is there a simpler shape?
4. Sign. Your name goes on it.

**A waiver with no reason is not a waiver, it is a shrug.** If you find yourself waiving the same
finding repeatedly, the rule is miscalibrated — take it to the improvement loop rather than waiving
it a fourth time.

---

## QA

**You own:** QA. **You sign:** H4 — `/h4`. **Skill:** `/mainline-e2e-suite`.

**Your day.** Changes arrive on staging. The suite runs on the cadence; you can expedite on request.

1. Run the suite. Green is the baseline, not the achievement.
2. **Explore what the suite cannot express.** This is the part only you can do — the odd combination,
   the real-world sequence, the thing that is technically correct and practically wrong.
3. **Everything you find that is repeatable goes into the suite.** If you have run the same manual
   steps three times, that was a test case you have not written yet. Your work should compound; a QA
   process that starts from zero every release is a treadmill.

   **The suite is yours and it is binding.** You decide what is in it; `/mainline-quality-gate` dimension 6
   runs it on every developer's PR and fails the build. Developers are gated on not breaking it,
   never on writing it — so you are not waiting on them, and they are not writing the test that
   happens to pass.
4. **File defects against the requirement they violate.** If there is no requirement to cite, you
   have found something more valuable than a bug: a missing requirement. Send it to Product.
5. **Quarantine a flaky test the day it flakes**, with a ticket. Never retry until green; a growing
   quarantine list goes to the improvement loop.
6. Sign, assign to release, notify.

**What you need, and should demand if you do not have it:** reproducible seed data, a staging
environment that resembles production, and the requirements themselves. Without the last one there is
nothing to assure quality against, and you are just clicking.

---

## Platform / DevOps

**You own:** Release, Operate, and the machinery under every other station.
**Skills:** `/mainline-deployment-pipeline`, `/mainline-observability`, `/mainline-local-stack`, `/mainline-security-gate`.

**Your work is tracked like everyone else's.** Line maintenance is `Platform` work: it enters at
Inbox, gets sized and prioritised, and runs Build → Gate → Review → Release. It is not a station on
the board and it should not be invisible either — untracked platform work only happens when somebody
is annoyed enough, which is the failure mode this whole line exists to remove.

**Your standing responsibilities:**

- **The gate command.** One command, same locally and in CI, non-zero on failure. When it drifts
  apart from CI, everything downstream stops being trustworthy.
- **The local full stack.** One command from a clean clone, no cloud credentials needed. This is what
  makes the developer loop close. LocalStack or equivalent for cloud services.
- **The handoff commands.** `/h1` … `/h4` in `.claude/commands/`. Keep `.github/mainline.json` and
  the field-ID cache current, and keep the `notify` command pointing somewhere people read.
- **CI/CD.** The gate on every PR, the E2E suite where you decided it binds, deploy automated and
  reversible, rollback tested for real.
- **DevSecOps.** SAST, dependency and CVE scanning, secrets scanning — in the pipeline, failing the
  build, not a report nobody opens.
- **Observability.** Alerts to a channel someone reads. One command turns an alert into a ticket.

**Your recurring judgment call:** should this check be a tool or a checklist item? Always prefer the
tool. A checklist item depends on attention; a tool does not.

---

## Lead

**You own:** Inbox triage, the improvement loop, and the line itself.
**Skills:** `/mainline-improvement-loop`, `/mainline-requirement-workflow`, `/mainline-pmi-github-project`.

**Your day.** Watch the Flow view. Your job is not to move work — the commands do that. It is to
notice what the board is telling you:

- **A station that is always full** is understaffed or its exit condition is wrong.
- **A handoff that keeps failing the same check** means the upstream station is not doing its job, or
  the check is miscalibrated. Both are fixable; neither fixes itself.
- **Work that arrives at a station and stalls** usually arrived without something it needed. That is
  a handoff defect, not a person defect.

**Own the escape ledger.** When something reaches production that should not have, run
`04-improvement.md` and record it. Nobody else will, and without it the playbook is a document rather
than a system.

**Onboarding a new project** is `01-onboarding.md`, and it is yours to drive.
