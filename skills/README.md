# Mainline skills

The skill library. Copy these into a project's `.claude/skills/`, calibrate the gates to the
project's stack, and work the line.

Every skill is **language- and platform-agnostic by construction**: the *dimensions* and the process
are invariant, the *tools* are calibrated per project. Each skill that names a reference toolchain
carries a substitution table. Keep the dimension; swap the tool.

## By station

| Station | Skill | Owner |
|---|---|---|
| Inbox → Discovery | `mainline-product-discovery` | Product |
| Discovery (UI/UX design) | `mainline-ui-exploration` | Product |
| Requirement | `mainline-requirement-workflow` | Product |
| Design (system design) | `mainline-domain-modeling` | Developer |
| Build | `mainline-development-workflow` | Developer |
| Build | `mainline-local-stack` | Platform (built), everyone (used) |
| Gate | `mainline-quality-gate` | Developer |
| Review | `mainline-review-station`, `mainline-security-gate` | Reviewer |
| QA | `mainline-e2e-suite` | QA |
| Release | `mainline-deployment-pipeline` | Platform |
| Operate | `mainline-observability` | Platform |
| Any | `mainline-refactoring`, `mainline-refactor-smells` | Developer |
| Any | `mainline-file-finding` | Everyone |
| Any | `mainline-improvement-loop` | Lead |
| Setup | `mainline-pmi-github-project` | Lead |

## The skills

**Delivery**

- `mainline-requirement-workflow` — Product's half of the loop. Triage (can anyone write the Gherkin?), the
  `.feature` file, the non-functional requirements, the H1 handoff.
- `mainline-development-workflow` — the developer's half. Receive at H1, design, implement against a running
  stack, gate green, hand off at H2.
- `mainline-domain-modeling` — OntoUML/UFO decomposition as a validated Tonto model, derived into DDD tactical
  design plus module contracts and the Gherkin the model entails.
- `mainline-product-discovery` — when nobody can write the spec yet. Throwaway quarantined prototype, live
  observation log, Example Mapping, discovery record.
- `mainline-ui-exploration` — several genuinely different UI directions for one flow, so a direction is chosen
  by comparing artefacts rather than adjectives.

**Quality**

- `mainline-quality-gate` — the binding gate. Seven dimensions behind one command: behavior, architecture,
  static analysis, test adequacy, the end-to-end suite, and — optional — flow/CPG and mutation.
- `mainline-e2e-suite` — QA's compounding asset. What deserves an E2E test, how to write one that survives a
  redesign, seed data, flake discipline. Enforced as `mainline-quality-gate` dimension 6.
- `mainline-review-station` — the Review station. Automated review plus the security pass, human judgment on
  the findings, waivers with written reasons, the recorded sign-off, the H3 handoff.
- `mainline-refactoring` — large mechanical behavior-preserving rewrites via a rewrite engine.
- `mainline-refactor-smells` — Fowler smell-driven structural cleanup, ranked by complexity and CRAP.

**Platform**

- `mainline-local-stack` — the whole system on one machine with one command. What closes the developer's loop.
- `mainline-security-gate` — DevSecOps. SAST, dependencies, secrets, IaC, images, runtime posture. Binding,
  not a dashboard.
- `mainline-deployment-pipeline` — merge to production, automated and reversible. Expand/contract migrations,
  flags, rehearsed rollback.
- `mainline-observability` — instrument before ship, alert on symptoms with an owner and an action, one
  command from alert to ticket.

**Setup and improvement**

- `mainline-pmi-github-project` — the board, where `Phase` *is* the line. WBS, milestones, risk register,
  charter, issue forms, trunk-based branching with the CI gate.
- `mainline-file-finding` — a bug, risk or missing rule becomes a tracked, assigned, notified item without
  leaving the session. Day-to-day filing onto that board.
- `mainline-improvement-loop` — what the line learns when something escapes. Amend the earliest check, the
  escape ledger, the four flow metrics.

## Installing

See `playbook/01-onboarding.md`, steps 2 and 3. In short: copy the folders, calibrate `mainline-quality-gate`
and `mainline-security-gate` behind one command each, point `mainline-refactoring` at the language's rewrite engine,
install `tonto-cli`, add the `prototypes/` quarantine, run `mainline-pmi-github-project`, then write
`.github/mainline.json` and cache the board's field IDs — used by `mainline-file-finding` and the `/h1` … `/h4`
commands in `commands/`.

## Provenance

These skills began as [`SpryElephant/spry-forge`](https://github.com/SpryElephant/spry-forge) and are
maintained here. Changes made for Mainline:

- `feature-workflow` split into `mainline-requirement-workflow` and `mainline-development-workflow`. One skill spanning
  the H1 handoff cannot enforce it, and it made each role read two-thirds irrelevant material.
- `mainline-quality-gate` gained dimension 6 — the end-to-end suite against a running stack. QA authors it,
  developers are gated on not breaking it.
- New: `mainline-e2e-suite`, `mainline-local-stack`, `mainline-review-station`, `mainline-security-gate`, `mainline-deployment-pipeline`,
  `mainline-observability`, `mainline-improvement-loop`, `mainline-file-finding` — the stations after merge and the harvesting
  rule, which Forge did not cover.
- `mainline-pmi-github-project`: `Phase` extended with Review, QA and Release; `Work Type` gained `Platform`;
  `bug` issue form and the `type:bug` / `type:chore` / `type:platform` labels added;
  review count raised from 0 to 1; `Phase` transitions are commanded and gated rather than pulled by
  hand.

`mainline-domain-modeling/references/` vendors material from the Tonto project (MIT, NEMO/UFES) — see
`mainline-domain-modeling/references/ATTRIBUTION.md`.
