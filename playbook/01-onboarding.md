# Onboarding a project onto Mainline

Eight steps. Work them in order. Each ends with an **acceptance test** — an observable fact, not an
opinion. If you cannot demonstrate it, the step is not done.

When step 8 passes, the project is on Mainline.

> **Report progress by step number.** "We are stuck on step 4" is something we can act on. "It isn't
> working" is not.

---

## Step 1 — Assess

**Goal:** know what you are onboarding before you change anything.
**Who:** lead + one engineer from the project.
**Time:** half a day.

Greenfield projects skip to step 2. Everything else gets an interview.

- [ ] **What runs.** Can you start the system on a laptop today? What is missing when you try?
- [ ] **What is tested.** Is there a test suite? Does it pass? Does anyone trust it?
- [ ] **What is written down.** Are there requirements anywhere — tickets, docs, a spec, a head?
- [ ] **What hurts.** Ask the team for the three things that most slow them down. Write their words.
- [ ] **Repository shape.** One repo or several? Can one feature be one PR?
- [ ] **Who signs.** Who currently approves a change, and is there a compliance reason for it?
- [ ] Write `docs/mainline-assessment.md`: the answers, plus the **top three constraints**.

**The finding to expect:** requirements were never written down. That is almost always the real
reason a codebase "can't be fixed" — not the code. If so, step 5 is the long pole, not step 2.

> **Acceptance test:** `docs/mainline-assessment.md` exists and names the top three constraints in
> the team's own words.

---

## Step 2 — Install the skills and commands

**Goal:** the engineering standard is in the repo and the gate runs.
**Who:** one engineer.

- [ ] Copy the skill folders from `skills/` into the project's `.claude/skills/` — including
      `domain-modeling/references/` — and `commands/` into `.claude/commands/`.
- [ ] **Calibrate `quality-gate`.** Pick one tool per dimension for the language and set the
      thresholds. Use the per-language tables inside the skill.

  | # | Dimension | Pick a tool | Threshold |
  |---|---|---|---|
  | 1 | Behavior (BDD) | | every requirement has a running scenario |
  | 2 | Architecture | | boundaries from the design, as tests |
  | 3 | Static analysis | | zero findings; failures fail the build |
  | 4 | Test adequacy | | complexity ceiling, coverage floor, CRAP ceiling |
  | 5 | Flow / CPG *(optional)* | | only for flow / reachability / taint facts |
  | 6 | End-to-end | | the existing suite passes against a running stack |
  | 7 | Mutation *(optional)* | | mutation score floor on the scoped set |

- [ ] **Wire them behind one command** that exits non-zero on any failure — `./gradlew check`,
      `make check`, `npm run check`. One command, no exceptions. Local must equal CI.
- [ ] **Calibrate `security-gate`** the same way — SAST, dependencies, secrets, IaC, images,
      runtime posture — behind its own single command. Baseline today's findings *with dates and
      owners*; the gate blocks anything new from day one, and the baseline is burned down as
      `Platform` items, worst exposure first.
- [ ] Point `refactoring` at the language's rewrite engine (OpenRewrite / Roslyn / ts-morph / …).
- [ ] Install the design toolchain: `npm install -g tonto-cli@0.4.13` (Node ≥ 20 — pin it; verify
      with `npm ls -g tonto-cli`, **not** `--version`). Create `domain/` with a `tonto.json` manifest,
      and a **separate CI job** — outside the gate command — asserting the `.tonto` files still validate.
- [ ] Add the prototype quarantine: a `prototypes/` directory excluded from the build, from CI and
      from coverage, plus **one architecture rule forbidding any production module to import it**.

**Calibrate, don't cargo-cult.** The dimension is fixed; the tool is yours. A Python project uses
`ruff` and `pytest-bdd`; a Go project uses `golangci-lint` and `godog`. Keep the dimension.

> **Acceptance test:** the single gate command runs clean locally, and a deliberately bad change —
> an unused variable, an uncovered branch in a complex method, a forbidden import — fails it.

---

## Step 3 — Stand up the board

**Goal:** the board *is* the line. Where a card sits is which station the work is at.
**Who:** one engineer with `gh` authenticated (`project`, `workflow`, `repo`, `read:org` scopes).

- [ ] Run `pmi-github-project`: project, custom fields, milestones, labels, WBS seed, risk register,
      charter, issue forms.
- [ ] Confirm `Phase` carries `Inbox, Requirement, Design, Build, Gate, Review, QA, Release, Done`.
- [ ] Confirm `Work Type` carries `Epic, Slice, Risk, Refactor, Spike, Bug, Chore, Platform`. An
      Epic breaks into Slices, **one Slice per `.feature` file**. Discovery is a Spike, off the
      pipeline. **`Platform` is work on the line itself** — the gate command, the local stack, the
      handoff commands, CI, alerting. It is not a station; it is a work type, and it enters at Inbox
      like anything else, skipping Requirement and Design the way a bug fix does. Tracking it is what
      stops line maintenance from being invisible unpaid work.
- [ ] Create the saved views (UI only — `gh` cannot): **Flow** (board, grouped by Phase — the daily
      driver), Roadmap, Backlog, Risk register, By milestone.
- [ ] Trunk-based branching: squash-only, delete on merge, branch protection requiring the `gate`
      status check on the default branch.
- [ ] Confirm `required_approving_review_count` is **1**. Review is a station.
- [ ] Write `security-gate`'s triage policy — Critical / High / Medium / Low / false positive — into
      the project charter.

**Known traps:** `Type` is reserved by GitHub — the field is `Work Type`, and its JSON key is
`"work Type"`. `field-create` fails silently under `>/dev/null` when it trips the secondary rate
limit — verify with `field-list` and retry. Pushing `.github/workflows/*` needs the `workflow` scope.

> **Acceptance test:** a real Slice sits on the Flow view with Phase, Work Type, Area, Size and
> Priority set, and its parent Epic shows sub-issue progress.

---

## Step 4 — Wire the handoffs

**Goal:** work moves between people by command, not by memory. **This is the step that makes it a
line instead of a set of habits.**
**Who:** lead + one engineer.

`/h1` … `/h4` ship in `commands/`. Each runs its station's checks, blocks and names the failure,
moves `Phase`, assigns the next owner, notifies them with everything attached, and records the result
on the ticket. The contract is in `commands/README.md`; what is project-specific is only the config.

- [ ] Write `.github/mainline.json` — owner, repo, project number and ID, the `notify` command, and
      the default assignee per station. See `commands/README.md`.
- [ ] Cache the board's field and option IDs to `.github/project-fields.json`.
- [ ] Walk `/h1` through `/h4` once each on a real card.
- [ ] Nobody copy-pastes between tools. If a step requires a human to move text from one window to
      another, it is not done.

> **Acceptance test:** take a real Slice from Requirement to Design with one command. Then break one
> check deliberately and confirm the command refuses to move it and names the failing check. The
> assignee is notified. Nothing was copy-pasted.

---

## Step 5 — Requirements baseline

**Goal:** there is something to assure quality *against*.
**Who:** Product, with the client or the system's current owner.

Without requirements there is no QA, because there is nothing to test against — and no rewrite is
possible, because there is nothing to rewrite *to*.

- [ ] Every Slice in flight has a `.feature` file with Given/When/Then scenarios.
- [ ] A domain glossary exists, in the client's words, seeding the Tonto model.
- [ ] **Legacy projects: reconstruct.** Run `product-discovery` with the medium set to a read-only
      spike against the live system, and the current system's owner as participant. Record `RULE`,
      `EX` and `TERM` from what the system actually does. Timebox it. The output is a discovery
      record, not a full specification — start with the highest-traffic flows.
- [ ] Anything nobody can state → a Spike, not a guess. Writing scenarios you cannot source is how
      invented requirements enter the system, and the line will then build them with full rigor.

> **Acceptance test:** pick a shipped feature at random. Its scenarios exist, are testable, and use
> glossary terms rather than invented synonyms.

---

## Step 6 — Make the full stack run locally

**Goal:** a developer or an agent can validate acceptance criteria without deploying anything.
**Who:** platform engineer. **Skill:** `local-stack`.

This is what closes the build loop. If validation cannot happen locally, it gets deferred to QA — and
QA becomes the place defects are discovered rather than the place quality is assured.

- [ ] One command starts the whole system from a clean clone: front end, API, database, queues,
      third-party stubs.
- [ ] Cloud services run locally. For AWS — Lambda, S3, SQS, DynamoDB, API Gateway — use
      **LocalStack**, an API-equivalent of the AWS services on your own machine.
- [ ] Seed data exists and is reproducible. A stack you cannot populate is a stack you cannot test.
- [ ] Secrets for local runs are stubbed, never real.
- [ ] Emulator parity gaps — IAM semantics, consistency, throttling, limits — written down in the
      stack README, and the emulator version pinned.
- [ ] **One feature is one change.** If the front end and back end are separate repos and a Slice
      cannot be one PR, fix that here — combine the repos, or define the paired-PR process explicitly.
      Half a feature reviewed in isolation is half a review.

> **Acceptance test:** on a machine that has never run the project and has no cloud credentials, one
> command brings the stack up and an agent validates an acceptance scenario end to end.

---

## Step 7 — Review, QA, Release, Operate

**Goal:** the four stations after merge exist and are staffed.
**Who:** lead, reviewer, QA, platform.
**Skills:** `review-station`, `security-gate`, `e2e-suite`, `deployment-pipeline`, `observability`.

- [ ] **Review** (`review-station`) — an automated review runs at high effort on every PR, plus the
      PR-bound security dimensions. A named human reads the findings and signs. Findings are
      resolved or waived *with a written reason*. The signature is recorded where an auditor can find
      it.
- [ ] **QA** — the suite runs against staging on a fixed cadence, with an expedite path. What QA
      finds is added to the permanent suite. Defects are filed **against the requirement they
      violate**, not as free-floating bug reports.
- [ ] **E2E suite** (`e2e-suite`) — browser tests against a running stack, growing with every Slice.
      **QA owns the content; `quality-gate` dimension 6 enforces it.** Developers are gated on not
      breaking the suite, never on authoring it. Start with the whole suite on every PR; split into a
      smoke set (PR) and full set (release) only when it outgrows the window, and write down which is
      which.
- [ ] **Release** — deploy is automated and reversible. Migrations are backward-compatible or gated.
      Rollback has been performed at least once, on purpose. The differences between staging and
      production — scale, data volume, third-party sandbox vs live, flag state — are enumerated in
      the README.
- [ ] **Operate** — alerts and a dashboard exist for the new behavior *before* it ships. An alert
      fires to a channel a person actually reads, and turning one into a ticket is one command.

> **Acceptance test:** one change goes from merge to production with each handoff recorded, and a
> rollback has actually been performed — not just documented.

---

## Step 8 — Run one real feature the whole way

**Goal:** prove the line works with the real team, using only this playbook.
**Who:** everyone.

- [ ] Pick a real Slice. Not a toy, not a spike.
- [ ] Carry it Inbox → Done using only the commands and checklists here. When you have to ask a
      person how something works, write down the question — that is a gap in the playbook.
- [ ] Every handoff leaves a recorded result on the ticket.
- [ ] Hold a retro with `improvement-loop` (`04-improvement.md`). It must produce **at least one
      amended check**. If it produces none, the retro was not honest.

> **Acceptance test:** it shipped; every handoff has a recorded result; and one check has been
> changed as a result.

---

## Onboarding summary

| Step | Done? |
|---|---|
| 1. Assess | ☐ |
| 2. Install the skills and commands | ☐ |
| 3. Stand up the board | ☐ |
| 4. Wire the handoffs | ☐ |
| 5. Requirements baseline | ☐ |
| 6. Full stack runs locally | ☐ |
| 7. Review, QA, Release, Operate | ☐ |
| 8. One real feature end to end | ☐ |

Steps 2, 3, 4 and 5 are mostly copying, calibrating and configuring. Steps 6 and 7 are where the
real work is.
