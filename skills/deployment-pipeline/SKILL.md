---
name: deployment-pipeline
description: Get a change from merge to production automatically and reversibly — environments and parity, expand/contract database migrations, feature flags, the deploy and rollback commands, and release approval and tagging. Use when onboarding a project onto the line, at the Release station, and whenever a deploy required someone to remember a step.
---

# Deployment pipeline

The Release station's machinery. Two properties matter more than everything else combined:

1. **Automated** — the same steps every time, run by a machine, with no tribal knowledge.
2. **Reversible** — you can put the previous version back, quickly, and you know that because you
   have done it on purpose.

Everything else here serves those two.

## Environments

| Environment | Purpose | Parity requirement |
|---|---|---|
| **Local** (`local-stack`) | The developer loop | Same engines and major versions. Emulated cloud services. |
| **Staging** | The QA station | Same topology and configuration shape as production. Realistic data volume, never a production data copy. |
| **Production** | — | — |

**Differences between staging and production are where releases die.** Enumerate them deliberately —
scale, data volume, third-party sandbox vs live, feature flag state — and write them in the project
README. A difference you know about is a risk; one you do not is an incident.

One config mechanism across all three. If local reads a `.env`, staging reads Parameter Store, and
production reads something a person edits in a console, you have three systems.

## The deploy

- **One command**, or a merge that triggers it. If a deploy needs a person to remember an order of
  operations, it will eventually be done wrong, and probably at the worst moment.
- **Infrastructure as code.** No console changes. A console change is a change nobody reviewed, and
  it silently invalidates every environment description you have.
- **Immutable artifacts** — build once, promote the same artifact through environments. Rebuilding
  per environment means production runs something no one tested.
- **Idempotent and re-runnable.** A half-finished deploy must be safe to run again.
- **Health-checked.** The pipeline verifies the new version is serving before it declares success.
  "The deploy job went green" is not the same fact.

## Rollback

**Rollback is a command that has been run for real.** A rollback paragraph in a runbook is a
hypothesis, and incidents are a bad time to test hypotheses.

- Rehearse it on purpose — on staging at minimum, on production ideally, on a normal afternoon.
- Know the window: how long from "we should roll back" to "we have." Write the number down.
- Know what rollback does **not** undo. This is almost always the database, which is why migrations
  get their own section.
- Prefer a fast, boring rollback over a clever forward-fix under pressure.

> A project is not on Mainline until a rollback has actually been performed. Not documented —
> performed.

## Database migrations

The part rollback cannot save you from, and therefore the part that needs discipline.

**Expand / contract**, always:

1. **Expand** — add the new column/table. Nothing reads it. Deploy.
2. **Migrate** — backfill. Both shapes work. Deploy.
3. **Contract** — remove the old shape, once nothing references it and you are past the window where
   you would roll back. Deploy.

Three deploys, not one. It feels slow exactly until the first time a rollback would otherwise have
lost data.

- **Every migration is backward-compatible with the previous application version**, or it is gated
  behind a flag. During any rolling deploy both versions are live simultaneously — that is not an
  edge case, it is every deploy.
- **Never destructive in the same release that stops using the data.** Separate them by a release.
- **Test migrations against realistic volume.** A migration that locks a table for eight minutes is
  an outage, and it is fine on a laptop with a thousand rows.
- Migrations run automatically as part of deploy, forward-only, versioned in the repo.

## Feature flags

Flags separate *deploying* from *releasing*, which is what makes trunk-based development work with
real users.

- Ship dark, enable deliberately. A flag flip is a faster and safer rollback than a redeploy.
- **Every flag has an owner and a removal date.** Flags are debt: each one doubles a code path, and
  the combinations are what nobody tests.
- Removing a stale flag is a `Platform` work item, not something that happens spontaneously.

## Release approval and tagging

- The Release station's sign-off is recorded — who approved, and against which QA result.
- Tag with SemVer at milestone completion; generate release notes from Conventional Commits.
- The tag identifies the exact artifact that was deployed. Reconstructing what was live at 14:00 last
  Tuesday should take seconds.

## Progressive delivery *(when the blast radius justifies it)*

Canary or blue/green, with automated rollback on error-rate or latency regression. Worth the cost
when a bad release is expensive; over-engineering for a low-traffic internal tool.

Whatever the strategy, **the rollback trigger must be automatic**. A human watching a dashboard is
not a safety mechanism at three in the morning.

## Failure modes

- **The undocumented step.** "You also have to bump the version in that other place." It will be
  missed, under pressure, by whoever knows least.
- **Rollback that has never been run.** Discovered to be broken exactly when needed.
- **Destructive migration shipped with the code that stops using the column.** Now rollback loses
  data, so rollback is off the table, so you forward-fix in an incident.
- **Console changes.** Now the IaC is fiction, and the next `apply` reverts a fix nobody recorded.
- **Staging that is nothing like production.** Everything passes; production still breaks; people
  conclude testing is pointless.
- **Flags that never get removed.** Two hundred code paths, none of them tested in combination.

## Relationships

- **`local-stack`** — same service definitions where possible; two descriptions of one system drift.
- **`security-gate`** — dimensions that bind on the release path run here.
- **`observability`** — dashboards and alerts for the new behavior exist *before* the deploy, not
  after the first incident.
- **`e2e-suite`** — the full suite runs on the release path when it has outgrown the PR window.
- **`pmi-github-project`** — milestones are the release gates; tags are cut at their completion.
