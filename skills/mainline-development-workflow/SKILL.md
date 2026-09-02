---
name: mainline-development-workflow
description: The developer side of the delivery loop — receive a requirement at `/ready-for-dev`, design it with /mainline-domain-modeling, implement against the scenarios while validating on a locally running stack, and pass `/ready-for-review` into review with /mainline-quality-gate green. Use whenever picking up a Feature, a bug, or a platform change for implementation.
---

# Development workflow

The developer's half of the loop. It starts at **`/ready-for-dev`**, where the work arrives, and ends at **`/ready-for-review`**,
where it goes to review.

## 0. Check what arrived

Before starting, confirm the handoff was complete: a `.feature` file with testable scenarios,
non-functional requirements stated or marked N/A, scope boundaries, and the prototype or screenshots
where there is UI.

If something is missing, **send it back rather than filling the gap yourself**. A developer inventing
the missing requirement is the same failure as Product inventing it, with less information. Note what
was missing — it belongs in the improvement loop.

## 1. Design — `/mainline-domain-modeling`

**Does this change the system's shape?** A new module, a new contract, a new cross-context
interaction → yes, design it. A bug fix, or a change that fits inside existing module contracts →
no. That is ordinary coding: still gated, no design pass. Platform and infrastructure work is
usually the same — straight to step 2.

When you do design:

1. **Read `domain/` first.** Does this extend an existing context or introduce one? Import, never
   redeclare. A duplicated concept under a second name is what this step exists to catch.
2. Decompose ontologically in OntoUML/UFO, written as Tonto under `domain/`.
3. `tonto-cli validate .` — fix every error and warning. Deriving a design from an unvalidated model
   propagates ontological errors straight into the code.
4. Derive the design: aggregate roots, boundaries, entities vs value objects, roles, domain events.
5. Write the module contracts — responsibilities (including what the module explicitly does *not*
   do), public API, dependencies and direction, invariants.
6. Translate the boundaries into the architecture rules `/mainline-quality-gate` dimension 2 will enforce.
7. **Send the entailed scenarios back to Product** (`/mainline-requirement-workflow` step 5) before building.
   The model proves which scenarios are missing; build against the complete set, not the first draft.

## 2. Implement

- Implement to satisfy the scenarios. Respect module public APIs; keep internals internal.
- **Validate against a running system.** Bring the full stack up locally (`/mainline-local-stack`) and check
  each acceptance criterion against it. Your loop closes on validated criteria — not on the code
  looking right. If validation cannot happen locally, it gets deferred to QA, and QA becomes where
  defects are discovered rather than where quality is assured.
- **One feature is one change.** A Feature is one `.feature`; make it one PR, front to back. If the
  repo layout makes that impossible, say so — it is a platform problem, not a personal one.
- **Run `/mainline-quality-gate` continuously**, not once at the end. The gate is a loop you work inside, not
  an exam you sit at the end.

## 3. Keep the work clean

- **Refactors are separate.** Behavior-preserving work — `/mainline-refactoring` for mechanical moves,
  `/mainline-refactor-smells` for structural cleanup — lands as its own gate-green commit, before the feature
  work that depends on the new shape. Never mixed in.
- **No unrelated changes.** A drive-by fix in the same PR is a fix nobody reviewed.
- **File what you find, now** (`/mainline-file-finding`). A bug, a risk, or a missing rule you are not fixing becomes a filed,
  assigned, notified ticket from inside your session, before you move on. You spent effort to learn
  it; harvest it. A finding carried in your head to the end of the task is a finding you threw away.

## 4. Pass `/ready-for-review`

- [ ] `/mainline-quality-gate` green in CI on the branch — all applicable dimensions.
- [ ] Every acceptance scenario passing, with validation evidence on the ticket.
- [ ] Entailed scenarios from step 1 either implemented or sent back to Product.
- [ ] No unrelated changes; refactors in their own commits.
- [ ] Findings filed.
- [ ] **Signed** by you · **assigned** to a reviewer · **notified**.

Green is not a report someone interprets. **Done means green**, and never lower a threshold or
weaken a spec to get there. If a threshold is wrong, that is a conversation and a `Platform` work
item — not an edit on your branch.

**Then stop.** `/ready-for-review` ends this workflow and ends the sitting. Do not review your own
PR, triage the findings, or carry the card into QA — Review is another person's station, and on a
one-person project it is another sitting with a reviewer who is not the author. See "One station,
one sitting" in `playbook/00-overview.md`.

## Dependency order

`/ready-for-dev` → (design, if the shape changes → entailed scenarios back to Product) → implement → `/mainline-quality-gate`
until green → `/ready-for-review`

That chain is one sitting, and both ends of it are walls. It **starts** when `/ready-for-dev`
has already run in an earlier sitting — you do not write the requirement and then pick it up. It
**ends** at `/ready-for-review`.

## Relationships

- **`/mainline-requirement-workflow`** — hands off at `/ready-for-dev`; receives entailed scenarios back.
- **`/mainline-domain-modeling`** — step 1.
- **`/mainline-quality-gate`** — the binding gate; the exit condition for this whole workflow.
- **`/mainline-local-stack`** — what you validate against in step 2.
- **`/mainline-refactoring` / `/mainline-refactor-smells`** — orthogonal, separate commits.
- **`/mainline-e2e-suite`** — you are gated on not breaking it. QA grows it.
- **`/mainline-review-station`** — receives at `/ready-for-review`.
- **`/mainline-file-finding`** — how a finding becomes a tracked item without leaving the session.
