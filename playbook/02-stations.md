# The stations

Each station: what it is for, who owns it, what it produces, and — where work changes hands — the
handoff check that must pass before it moves.

Every handoff check ends the same way: **signed, assigned, notified.** A check that passes but leaves
nobody holding the work has not finished. Each one is a `/ready-for-…` command that runs the
checks, refuses to move the card if one fails, and records the result on the ticket.

**A handoff is where a sitting ends.** Run the command and stop; the next station is the next
sitting, even when you own it too. Two places on the line are deliberately not handoffs and are
marked as such below — those you work straight through. See "One station, one sitting" in
`00-overview.md`.

---

## Inbox

**Owner:** lead. **Skill:** `/mainline-requirement-workflow` §1

Triage. Set Work Type, Area, Size, Priority, Target. Then one decision:

> **Can somebody write the Gherkin?**
> Yes → Requirement. No → Discovery (a Spike, timeboxed, off the pipeline).

**Not everything is a Feature.** A bug fix, and `Platform` work on the line itself — the gate command,
the local stack, the handoff commands, CI, alerting — enter here too and go straight to Build,
skipping Requirement and Design. They are still gated, reviewed and released like anything else.
Platform work is tracked as work precisely so that maintaining the line does not become invisible
unpaid effort that only happens when somebody is annoyed enough.

Getting the discovery question wrong is expensive in one direction only. Sending known work through Discovery wastes a
few days; sending unknown work straight to Requirement means inventing the spec and then building it
with full rigor — the most expensive possible way to be wrong.

---

## Discovery *(off-pipeline)*

**Owner:** Product. **Skill:** `/mainline-product-discovery`. **Work Type:** Spike.

Run when nobody can state the acceptance criteria — a client described an outcome, the rules live in
an operator's head, the value is disputed, or you are replacing a system nobody documented.

1. **Name one riskiest assumption** — the sentence that, if false, makes the work pointless. Make it
   falsifiable with a threshold. Classify it: value, usability, feasibility, or viability. **One risk
   per prototype.** Fix the timebox now, in days.
2. **Choose the medium and the participant** for that risk class. Cheapest thing that can falsify the
   assumption wins: fake door, landing page, concierge, Wizard of Oz, paper, clickable, live-data,
   spike. For a legacy system: a read-only spike, with the current system's owner. When the open
   question is visual direction, the medium is `/mainline-ui-exploration` — several worlds compared, not one
   prototype tested.
3. **Build it quarantined** — `prototypes/<name>/`, outside the build, invisible to CI and coverage,
   and no production module may import it. Real domain content, never placeholder. The whole flow
   works end to end; a dead primary button is a lie about the design.
4. **Run sessions and record live** — seven tags, one line each: `RULE`, `EX`, `TERM`, `FRIC`,
   `ASSUM`, `Q`, `CHOICE`. Give tasks, never instructions. The moment you explain the screen, the
   usability finding is gone.
5. **Synthesize by Example Mapping** — `RULE` → blue cards → `Rule:` blocks. `EX` → green cards →
   scenarios. `Q` → red cards, blocking. `TERM` → the glossary. `ASSUM` → the risk register.
6. **Pass the discovery gate** — every rule has ≥1 example; every example maps to exactly one rule;
   zero open red cards on anything handed over; the riskiest assumption resolved.
7. **Write the record, delete the prototype.** `docs/discovery/<name>.md` is the only thing that
   survives. A prototype kept alive gets imported eventually.

**Out:** Gherkin first draft, glossary, decisions, accepted risks.

There is no code quality bar here and that is deliberate. A prototype exists to be wrong quickly;
holding it to coverage and complexity thresholds makes it cautious, slow, and — worst of all — good
enough to keep. Discovery ends on the timebox date, never on a definition of done. Whatever is
unfinished becomes a risk.

---

## Requirement

**Owner:** Product. **Skill:** `/mainline-requirement-workflow`.

One `.feature` file per Feature, in Gherkin. **This is the spec.** Not the ticket description, not the
Figma, not the conversation.

Scenarios state what a person achieved, never what the screen did. *"Ana refunds a R$120 cash order
with no manager present"* survives a redesign; *"Ana clicks Refund, then Confirm"* does not. Use the
participant's own words from the glossary.

Expect to come back here after Design: the domain model entails scenarios the requirement did not
state.

### `/ready-for-dev` — Requirement → Design *(Product → Developer)*

- [ ] A `.feature` file exists. Every scenario is Given/When/Then and is testable.
- [ ] **Every scenario traces to something observed or to a stated business rule.** No invented
      requirements.
- [ ] Terms come from the glossary or the existing Tonto model — not new synonyms for existing
      concepts.
- [ ] **Non-functional requirements stated, or explicitly marked N/A:** authentication and SSO,
      **tenancy**, performance, compliance, data retention.
- [ ] Out of scope stated explicitly.
- [ ] Prototype or screenshots attached wherever there is UI.
- [ ] **Signed** by Product · **assigned** to a developer · **notified** with everything attached.

> **Why the NFR line exists.** A feature once shipped without enterprise SSO because enterprise
> tenancy was assumed by everyone and written down by no one. Nothing downstream could catch it,
> because downstream only checks what upstream wrote. That is the whole reason this check is here —
> see `04-improvement.md`.

---

## Design

**Owner:** the developer who will build it. **Skill:** `/mainline-domain-modeling`.

**This station is system design.** It decides how the software is structured: the domain model, the
module boundaries and their contracts, the data model, and the architecture rules the gate will
enforce. It is not UI/UX design. What the screens look like and how the flow feels is decided by
Product before `/ready-for-dev`, through `/mainline-ui-exploration` inside Discovery, and arrives here attached to
the Feature as a prototype or screenshots. The developer builds to that design. If it cannot be built
as drawn, that is a conversation with Product, not a redesign in Build.

| | UI/UX design | System design |
|---|---|---|
| Question it answers | What does the person see and do, and how does it feel? | How is the software structured so the scenarios hold? |
| Owner | Product | The developer who will build it |
| When | Before `/ready-for-dev`, during Discovery | After `/ready-for-dev`, at this station |
| Skill | `/mainline-ui-exploration` | `/mainline-domain-modeling` |
| Output | A chosen direction, a prototype or screenshots, the design decisions | A validated `.tonto` model, module contracts, architecture rules |
| Where it lives | Attached to the Feature | `domain/` in the repo |

Not a document — the artifact the design is *derived* from, and it lives in the repo.

1. **Read `domain/` first.** Does this extend an existing context or introduce one? Import, never
   redeclare. A duplicated concept under a second name is what this step exists to catch.
2. **Decompose ontologically** in OntoUML/UFO and write it as Tonto in `domain/`. Two checks catch
   most errors: hunt for **relators** (if a relationship carries attributes, a lifecycle, or rules,
   it is a first-class thing — name it), and challenge every noun that looks like a **role**
   (`Customer`, `Owner`, `Supplier` are almost never kinds).
3. **Validate** — `tonto-cli validate .`. Fix every error and warning. Deriving a design from an
   unvalidated model propagates ontological errors straight into the code.
4. **Derive the design** — aggregate roots from the ultimate sortals (relators are usually roots);
   boundaries from the connectors; entities vs value objects from identity; roles resolved to an
   interface, a projection, or a cross-context entity — never a duplicate identity; events and
   policies from `event` and `situation`.
5. **Write the module contracts** — responsibilities (including what it explicitly does *not* do),
   public API, dependencies and direction, invariants. Everything not in the public API is internal.
6. **Take the entailed Gherkin back to Requirement.** Phase transitions, mediation bounds, lifecycle
   events, non-empty collections. **This is how we check the system requirements are faithful to the
   product requirements** — and it runs in the stronger direction: the model proves which scenarios
   are *missing*, rather than a human reading two documents side by side and feeling satisfied.

**Out:** a validated `.tonto` model, module contracts, and the architecture rules dimension 2 will
enforce.

**No handoff.** The same developer continues into Build.

**When to skip:** a bug fix, or a change that fits inside existing module contracts, is ordinary
coding — still gated, no design pass. Re-enter Design only when the change alters the system's
*shape*: a new module, a new contract, a new cross-context interaction.

---

## Build

**Owner:** developer. **Skills:** `/mainline-development-workflow`, `/mainline-local-stack`.

Implement to the scenarios. Respect module public APIs. Run the gate continuously.

- **Full stack, locally.** Start the whole system and validate each acceptance criterion against it
  before handing off. An agent's loop closes on validated criteria — not on "the code looks right."
- **Full stack, one change.** A Feature is one `.feature`; make it one PR. Front-end and back-end are
  not two jobs.
- **File what you find, now** (`/mainline-file-finding`). A bug, a risk, or a missing rule you are not fixing becomes a filed,
  assigned, notified ticket from inside the session. You spent effort to learn it; harvest it. A
  finding you meant to mention tomorrow is a finding you threw away.

---

## Verify

**Owner:** developer. **Skill:** `/mainline-quality-gate`.

The developer proves the work is done by running the gate until it is green. The station is
called Verify so that "gate" means one thing only: the command.

Seven dimensions, one command, exits non-zero on any failure. CI runs the same command. Branch
protection requires it. **Local equals CI.**

| # | Dimension | Passes when |
|---|---|---|
| 1 | Behavior | every requirement has a passing scenario. No passing scenario = not implemented |
| 2 | Architecture | boundary rules green. *A boundary that isn't tested isn't a boundary* |
| 3 | Static analysis | report empty. Findings fail the build; they are never advisory |
| 4 | Test adequacy | complexity, coverage and CRAP within the project's thresholds |
| 5 | Flow / CPG *(optional)* | data-flow, reachability and taint gates green |
| 6 | End-to-end | the existing suite passes against a running stack. **QA owns the suite; you are gated on not breaking it** |
| 7 | Mutation *(optional)* | surviving mutants under the floor, on the scoped set |

**Done means green.** Never weaken a spec or lower a threshold to pass. The gate is the proof; it is
not a report someone interprets.

Not a handoff — this is Build's exit condition, and the precondition for `/ready-for-review`.

---

## Review

**Owner:** a reviewer who is not the author. **Skills:** `/mainline-review-station`, `/mainline-security-gate`.

Every change is reviewed by another **person**. Keep the person — drop the assumption that they must
read every line. A human reading a large diff carefully is slower and less thorough than a tool, and
we have tools.

1. The automated review runs at high effort on the PR.
2. The security pass runs on the PR-bound dimensions: SAST, secrets, dependencies, IaC. Images and
   runtime posture bind on the release path.
3. **A named person reads the findings** and decides. Their judgment is applied to the findings, not
   to the diff line by line.
4. Findings are fixed, or waived **with a written reason**. "Looks fine" is not a reason.

### `/ready-for-review` — Verify → Review *(Developer → Reviewer)*

- [ ] Gate green in CI on the branch.
- [ ] Every acceptance scenario passing, with validation evidence on the ticket.
- [ ] No unrelated changes. Behavior-preserving refactors are separate commits, and separate PRs.
- [ ] **Signed** by the developer · **assigned** to a reviewer · **notified**.

### `/ready-for-qa` — Review → QA *(Reviewer → QA)*

- [ ] Automated review run; findings resolved or waived with a reason.
- [ ] Security pass clean.
- [ ] Gate green on the merge commit.
- [ ] **Named human sign-off recorded** where an auditor can find it.
- [ ] **Signed** by the reviewer · **assigned** to QA · **notified**.

---

## QA

**Owner:** QA. **Skill:** `/mainline-e2e-suite`.

QA assures quality **against the requirements**. That is why the requirement is the spine: without a
written spec there is no such thing as QA, only people clicking around hoping to notice something.

1. **Run the suite against staging** — on the cadence, or on an expedite request.
2. **Explore what the suite cannot express.** Judgment, not repetition. If you are executing the same
   manual steps a third time, that is a test case, not a QA activity.
3. **Add what you find to the permanent suite.** QA's work compounds or it is wasted. The suite is
   yours: you decide what is in it, and `/mainline-quality-gate` dimension 6 makes it binding on every
   developer. That split is deliberate — a developer writing the E2E test for their own feature
   writes the one that passes, and a suite that does not block a merge is documentation.
4. **File defects against the requirement they violate.** A defect that cites no requirement is
   either a missing requirement — send it back to Product — or a preference.
5. **Quarantine a flaky test out of the gate the day it flakes**, with a ticket. Never retry until
   green. A growing quarantine list is an `04-improvement.md` entry: the suite is dying.

**Cadence:** a scheduled run at a fixed time, plus an expedite path when someone asks. Both are
commands; neither is a person remembering.

### `/ready-for-release` — QA → Release

- [ ] Suite green on staging.
- [ ] New coverage merged into the permanent suite, shipping *with* the release rather than after.
- [ ] Open defects either fixed, or accepted with a named person and a date.
- [ ] Nothing newly quarantined without a ticket.
- [ ] **Signed** by QA · **assigned** to the release approver · **notified**.

---

## Release

**Owner:** release approver. **Skill:** `/mainline-deployment-pipeline`.

- Deploy is automated and **reversible**. Rollback is a command that has been run for real, not a
  paragraph in a runbook.
- Migrations are backward-compatible, or gated behind a flag.
- Security gates run in the pipeline. DevSecOps, not DevOps.
- **Alerts and dashboards for the new behavior exist before it goes live.** Adding them after the
  first incident means the first incident was found by a customer.
- Tag and release notes at milestone completion.

---

## Operate

**Owner:** on call. **Skill:** `/mainline-observability`. Standing station — no card sits here.

Monitoring, alerting, logs, traces, error tracking. Alerts fire to a channel a person actually reads,
and turning an alert into a ticket is one command.

We are moving faster than before, so more can fall through. Observability is what makes that safe
rather than frightening — it is the reason you can move fast without dreading it.

**Feeds two places:** new work goes to Inbox. Anything that should have been caught upstream goes to
`04-improvement.md`.
