---
name: mainline-local-stack
description: Make the entire system runnable on one machine with one command — services, database, queues, cloud-service emulation (LocalStack and equivalents), third-party stubs, and reproducible seed data — so a developer or an agent can validate acceptance criteria without deploying. Use when onboarding a project onto the line, and whenever validation is being deferred to a deployed environment.
---

# Local stack

**The single highest-leverage piece of infrastructure on the line.** It is what closes the
developer's loop: implement → run the system → check the acceptance criterion → fix → repeat, until
the scenarios actually pass.

Without it, validation is deferred to whatever environment does exist — and QA stops being where
quality is assured and becomes where defects are discovered. Everything upstream then degrades to
"it looks right," which is not a standard anybody can gate on.

It matters more with agents than it did without them. An agent's loop terminates on a *verified*
condition. Give it a system it can run and it will iterate until the criteria pass; give it none and
it will iterate until the code looks plausible, which is a different and much worse stopping rule.

## The contract

One command. Clean clone. No cloud credentials. Everything up.

```
git clone … && cd … && <the up command>
```

- **One command**, wired the way `/mainline-quality-gate` is wired — `make up`, `npm run dev`, `docker compose
  up`. Not a six-step README.
- **No cloud credentials required**, and no access to any shared environment. A new laptop, an
  offline flight, and a CI runner all behave the same.
- **Cold start under the time a person will actually wait.** Ten minutes is tolerable once; ten
  minutes on every branch switch means people stop using it, and then it rots.
- **The same stack CI uses** for dimension 6. A suite that only runs in CI cannot be debugged.
- **Idempotent.** Running it twice does not corrupt state. There is a reset command, and people know
  it.

## What has to be in it

| Component | What "done" means |
|---|---|
| **Application services** | Every service the feature path touches, including the front end. Not just the one you are working on. |
| **Data stores** | Real engine, same major version as production. Not SQLite standing in for Postgres — the differences surface exactly where the bugs are. |
| **Async infrastructure** | Queues, topics, schedulers. Async is where the interesting defects live; stubbing it out hides them until staging. |
| **Cloud services** | Emulated locally (see below). |
| **Third-party APIs** | Stubbed, with recorded real responses. Never live calls to a vendor sandbox — that makes your test suite depend on their uptime. |
| **Auth** | A real flow with local users, including the roles the scenarios need. |
| **Seed data** | Reproducible, in the repo, applied by command. |
| **Secrets** | Local dummies, committed. Never a real credential, never a shared vault for local dev. |

## Cloud service emulation

| Platform | Use |
|---|---|
| **AWS** (reference) | **LocalStack** — API-equivalent implementations of Lambda, S3, SQS, SNS, DynamoDB, API Gateway, Step Functions and more, running in Docker. The de facto option for cloud-native local development. |
| GCP | Official emulators — Pub/Sub, Firestore, Datastore, Bigtable, Spanner |
| Azure | Azurite (storage), Cosmos DB emulator, Service Bus emulator |
| Anything else | Testcontainers, or the vendor's own container image |

**A Lambda-based API is the hard case and it is worth doing.** The instinct is to say "you cannot run
serverless locally, so just deploy to a dev environment." That instinct is what leaves the
developer's loop open. LocalStack plus the framework's local invoke path closes it.

### Parity discipline

An emulator is not the service. Treat the gaps as known facts, not surprises:

- **Write the gaps down** in the project's stack README — IAM semantics, eventual consistency,
  throttling, service limits, features the emulator does not implement.
- **When a defect is traced to emulator drift**, that is a `/mainline-improvement-loop` entry, not a shrug: it
  means something needs a contract test against the real service.
- **Pin the emulator version** the same way you pin a language version. A silent emulator upgrade
  that changes behavior is indistinguishable from your own regression.

## Seed data

- Lives in the repo, applied by a command, named for the situations it creates
  (`customer-with-two-open-guias`, not `dump_final_v3`).
- Covers the states the scenarios need — not one happy-path user.
- Fast enough to reset per test run. If it is not, tests start sharing mutable state and the suite
  becomes order-dependent.
- **Never a production copy.** Data protection is the obvious reason; the better one is that it makes
  the environment depend on facts nobody wrote down.

## Setting it up

1. **Inventory the feature path.** Everything a request touches end to end. Anything you cannot run
   locally is the actual scope of this work.
2. **Containerize what you own**; use official images for what you do not.
3. **Emulate the cloud services** and pin the versions.
4. **Stub the third parties** with recorded responses.
5. **Write the up / down / reset / seed commands** and put them in the README.
6. **Prove it on a machine that has never run the project.** A colleague's laptop, or a fresh CI
   runner. This is the only test that counts — you cannot detect missing setup on the machine that
   already has it.

> **Acceptance test:** on a clean machine with no cloud credentials, one command brings the stack up
> and an agent validates an acceptance scenario end to end.

## Failure modes

- **"Just point it at the dev environment."** Now two developers share mutable state, the loop is
  slow enough that people batch changes, and nothing is reproducible. This is the default failure and
  it always looks reasonable at the time.
- **Drift.** The stack worked in March and nobody ran it since. Cure: CI runs the same stack for
  dimension 6, so drift breaks the build instead of ambushing the next person.
- **Partial stack.** The API runs locally but the front end points at staging. The integration bugs —
  the ones worth catching — are exactly the ones this hides.
- **SQLite standing in for the real database.** The differences appear precisely where the defects
  are: transactions, concurrency, types, migrations.
- **Undocumented manual steps.** "You also need to run this script and set that variable" is the
  same as not having a local stack; it just fails later, for someone else.

## Relationships

- **`/mainline-development-workflow`** — step 2 validates against this.
- **`/mainline-quality-gate`** — dimension 6 runs against this, locally and in CI.
- **`/mainline-e2e-suite`** — the suite targets it.
- **`/mainline-deployment-pipeline`** — the same definitions should generate the deployed topology wherever
  possible. Two independent descriptions of one system drift apart by default.
