# Glossary

Plain definitions of the words this playbook uses. Each term is defined once, in one to three
sentences, with a pointer to where it is used in full. If a word in the playbook is not here and you
had to ask someone what it meant, that is a gap: add it.

Roles (Product, Developer, Reviewer, QA, Platform, Lead) are defined in `03-roles.md` and are not
repeated here.

---

## The line

**Line.** The whole path a piece of work takes from Inbox to production, drawn as a sequence of
stations. A project "on Mainline" runs every change along this path. See `00-overview.md`.

**Station.** One named stop on the line, with one owner, one job, and one skill that says how the job
is done. The stations are Inbox, Discovery, Requirement, Design, Build, Verify, Review, QA, Release and
Operate. See `02-stations.md`.

**Handoff.** A place on the line where work changes hands from one role to another. There are exactly
four: `/ready-for-dev` (Product to Developer), `/ready-for-review` (Developer to Reviewer),
`/ready-for-qa` (Reviewer to QA), `/ready-for-release` (QA to Release). Every handoff runs a check, then is signed, assigned and notified. Everything else on the
line is one person working their own loop.

**`/ready-for-dev`, `/ready-for-review`, `/ready-for-qa`, `/ready-for-release`.** The four handoffs, and the four commands that run them. Each
command runs its station's checks, refuses to move the work if a check fails and names the failing
check, moves the card, assigns the next owner, notifies them, and records the result on the ticket.
See `commands/README.md`.

**Signed, assigned, notified.** The three things every handoff ends with. Signed: a named person put
their name on it. Assigned: the next owner is set on the card. Notified: that person was told, with
everything they need attached. A handoff missing any of the three has not finished.

**Board.** The GitHub Project that tracks the work. The board is the line: which column a card sits
in is which station the work is at. Stood up once by `/mainline-pmi-github-project`.

**Phase.** The board field that records which station a card is at. Its values are the station
names. Only the handoff commands should change it.

**Flow view.** The board view grouped by Phase. The daily driver for the Lead, and the place to see
where every piece of work is.

**Inbox.** Where all work enters, including bugs and Platform work. The Lead triages it and asks one
question: can somebody write the Gherkin? Yes goes to Requirement; no goes to Discovery.

**Discovery.** Off-pipeline work, run as a Spike, to find out what the requirement should be when
nobody can state it yet. A timeboxed throwaway prototype, real sessions with a participant, and a
written discovery record. Owned by Product.

**Requirement.** The station where the `.feature` file is written. The `.feature` file is the
specification; nothing else counts as a requirement.

**Design.** The station where the developer does the system design: the domain model, the module
boundaries and their contracts. Not UI/UX design, which is Product's work before `/ready-for-dev`. See the two
side by side in `02-stations.md`.

**Build.** The station where the developer implements to the scenarios, runs the full stack locally,
validates each acceptance criterion against a running system, and runs the gate continuously.

**Verify.** The station where the developer proves the work is done by running the gate until it
is green. Not a handoff: it is Build's exit condition and the precondition for `/ready-for-review`.
The station is called Verify so that "gate" means only the command (see Checks and quality below).

**Review.** The station where a second person, never the author, reads the automated review and
security findings, fixes or waives each one with a written reason, and signs.

**QA.** The station where the end-to-end suite runs against staging, a person explores what the
suite cannot express, and what they find is added to the suite. Defects are filed against the
requirement they violate.

**Release.** The station where the change is deployed to production. Deploy is automated and
reversible; rollback has been performed for real.

**Operate.** The standing station after release: monitoring, alerting, logs, traces. New work found
here goes to Inbox; anything that should have been caught upstream goes to the improvement loop.

---

## Work items

**Work Type.** The board field that says what kind of work a card is: Epic, Feature, Risk, Refactor,
Spike, Bug, Chore or Platform. (The field is named `Work Type` because `Type` is reserved by
GitHub.)

**Epic.** A body of work large enough to break into several Features. Sub-issue progress rolls up to
it.

**Feature.** The unit of delivery, and the one kind of work that brings a new requirement. One
Feature is one `.feature` file, one PR, and one trip along the line, full stack: front end and back
end together. Capitalised, it is the `Work Type`; in lower case, "feature" keeps its everyday
meaning.

**Spike.** A timeboxed investigation with a question to answer rather than a feature to ship.
Discovery is a Spike. A Spike ends on its timebox date, not on a definition of done.

**Bug.** A defect against a requirement. Enters at Inbox and goes straight to Build, skipping
Requirement and Design, but is still gated, reviewed and released.

**Chore.** Housekeeping with no behavior change and no requirement.

**Refactor.** Behavior-preserving change to the shape of the code. Lands as its own gate-green
commit and its own PR, never mixed into feature work.

**Risk.** An entry in the risk register: something that might go wrong, scored by probability times
impact, with a trigger and a planned response.

**Platform.** Work on the line itself: the gate command, the local stack, the handoff commands, CI,
alerting. It is a Work Type, not a station. Tracking it is what stops line maintenance from being
invisible unpaid work.

**Area, Size, Priority, Target, Milestone.** The other board fields set at triage. Area is the
module the work touches. Size is a rough estimate (XS to XL). Priority is Must, Should, Could or
Won't. Target is a roadmap date. A Milestone is a release, and completing one cuts a tag and
release notes.

**Finding.** Anything a session turns up that is not being fixed right now: a bug, a risk, a missing
rule. A finding is filed as a ticket, with fields set, assigned and notified, before the session
ends. That is `/mainline-file-finding`. A finding not filed is a finding thrown away.

---

## Artifacts

**`.feature` file.** A text file in Gherkin holding the scenarios for one Feature. It is the
requirement. It lives in the repository and runs as a test.

**Gherkin.** The Given / When / Then language the scenarios are written in. Given the starting
state, when a person does something, then an outcome is observable. Scenarios say what a person
achieved, never what the screen did.

**Scenario.** One Given / When / Then example in a `.feature` file. Every scenario traces to
something observed or to a stated business rule. A scenario with neither behind it is an invented
requirement.

**Acceptance criterion.** What must be true for a scenario to pass. Validated against a running
system, locally, before handoff.

**Non-functional requirement (NFR).** A requirement about how the system behaves rather than what it
does: authentication and single sign-on, tenancy, performance, compliance, data retention. Each is
stated or explicitly marked not applicable at `/ready-for-dev`.

**Glossary (project).** The list of domain terms in the client's own words. Scenarios and the domain
model use these terms and never invent synonyms. Distinct from this document, which defines the
playbook's own words.

**Domain model.** The formal description of the concepts in the problem domain and how they relate,
written in Tonto and kept in `domain/` in the repository. The system design is derived from it.

**Tonto.** The textual language the domain model is written in, based on OntoUML. `tonto-cli
validate` checks the model, and a CI job asserts it still validates. See `/mainline-domain-modeling`.

**Module contract.** For one module: what it is responsible for, what it explicitly does not do, its
public API, its dependencies and their direction, and its invariants. Everything not in the public
API is internal. The architecture rules the gate enforces come from here.

**Prototype.** Throwaway software built during Discovery or UI exploration to answer one question. It
lives in `prototypes/`, is excluded from the build, CI and coverage, and no production module may
import it. It is deleted when the discovery record is written.

**Discovery record.** The only thing that survives Discovery: `docs/discovery/<name>.md`, holding the
first-draft Gherkin, the glossary terms, the decisions and the accepted risks.

**Assessment.** `docs/mainline-assessment.md`, written in onboarding step 1: what runs, what is
tested, what is written down, what hurts, and the top three constraints in the team's own words.

**Escape ledger.** `docs/escape-ledger.md`, one row per escape: what escaped, where it surfaced,
where it should have been caught, and what check was changed. See `04-improvement.md`.

---

## Checks and quality

**Gate.** The single command that decides whether work is done. It runs every quality dimension the
project has calibrated and exits non-zero if any fails. Green means done; anything else means not
done. The same command runs locally and in CI, and its result is never overridden by a person
looking at the code and deciding it is probably fine. Never lower a threshold or weaken a test to
get it green. See `/mainline-quality-gate`.

**Gate dimension.** One of the seven kinds of check the gate runs: behavior (the scenarios pass),
architecture (the module boundaries hold), static analysis (zero findings), test adequacy
(complexity, coverage and CRAP within thresholds), flow / CPG (optional), end-to-end (the suite
passes against a running stack), and mutation (optional). The dimension is fixed; the tool that
implements it is chosen per project.

**Calibrate.** Pick the tool and set the thresholds for each gate dimension to fit the project's
language and stack. The dimension never changes; the tool and the numbers do.

**Coverage.** The share of code exercised by tests. One input to test adequacy, never the whole of
it.

**Cyclomatic complexity.** A count of the independent paths through a method. High complexity needs
either good coverage or simplification.

**CRAP score.** Change Risk Anti-Patterns. A number derived from a method's complexity and its
coverage that is high when complex code is poorly tested. The gate sets a ceiling on it.

**Static analysis.** Tools that read the code without running it and report bad practices,
likely bugs and style violations. Findings fail the build; they are never advisory.

**End-to-end (E2E) suite.** Browser (or API) tests that run against the whole system running
together. QA owns what is in it; gate dimension 6 runs it on every developer's PR. Developers are
gated on not breaking it, never on writing it.

**Mutation testing.** Deliberately altering the code in small ways to check that the tests notice.
A surviving mutant is a change the tests did not catch. Optional dimension 7.

**Flow / CPG.** Checks over a code property graph for facts about data flow, reachability and
taint, for example that untrusted input cannot reach a database call unchecked. Optional dimension 5.

**Security gate.** The DevSecOps checks, behind their own single command: SAST, dependency and CVE
scanning, secrets scanning, infrastructure as code, container images and runtime posture. Binding,
not a dashboard. Existing findings are baselined with dates and owners; anything new blocks.

**Automated review.** A tool-driven review of a PR that produces findings for a person to judge. At
the Review station the person reads the findings, not the diff line by line.

**Waiver.** A reviewer's decision not to act on a finding, recorded with a written reason. A waiver
with no reason is not a waiver. The same finding waived repeatedly means the rule is miscalibrated
and goes to the improvement loop.

**Signature.** A named person recording that they take responsibility for the work moving past a
handoff. Recorded on the ticket where an auditor can find it.

**Local stack.** The whole system, front end, API, database, queues and stubbed third parties,
started on one machine with one command from a clean clone and no cloud credentials. Cloud
services run locally through an emulator such as LocalStack. This is what lets a developer or an
agent validate acceptance criteria without deploying anything. See `/mainline-local-stack`.

**Staging.** An environment that resembles production closely enough for QA to run the suite
against it. The differences from production (scale, data volume, sandbox versus live third
parties, flag state) are written down in the README.

**Rollback.** Returning production to the previous release with one command. It counts only once it
has been performed for real, on purpose.

**Flaky test.** A test that passes and fails without the code changing. It is quarantined out of the
gate the day it flakes, with a ticket. Never retry until green.

**Quarantine.** Two uses. A quarantined test is removed from the gate with a ticket to fix it. The
prototype quarantine is the `prototypes/` directory that the build, CI and coverage ignore and that
production code may not import.

**Observability.** Monitoring, alerting, logs, traces and error tracking for the running system.
Alerts and a dashboard exist for new behavior before it ships. Alerts fire to a channel a person
reads, and turning one into a ticket is one command.

**Escape.** Anything that reached a station it should never have reached: a defect QA found that
Review should have caught, a requirement gap found in Build, an incident in production. An escape
is a defect in the process, not only in the code.

**Improvement loop.** What happens after an escape. Name it, find the earliest handoff that could
have caught it, amend that check (a tool first, a checklist line second), file the amendment as a
Platform item with an owner and a date, and add a row to the escape ledger. Owned by the Lead.
See `04-improvement.md` and `/mainline-improvement-loop`.

---

## Design

**UI/UX design.** What the screens look like and how the flow feels. Product's work, done before `/ready-for-dev`
through `/mainline-ui-exploration` inside Discovery, and attached to the Feature as a prototype or
screenshots.

**System design.** How the software is structured so the scenarios hold: the domain model, the
module boundaries and their contracts, the data model. The Developer's work, done at the Design
station through `/mainline-domain-modeling`.

**UI exploration.** Building several genuinely different clickable prototypes of the same flow so a
visual direction is chosen by comparing real artifacts rather than adjectives. Throwaway; only the
decisions cross into production.

---

## Tools, agents and people

**Tool.** A deterministic check: the gate command, the validator, CI. If a check can be a tool, it
is a tool. Never ask a person or an agent to eyeball what a tool can prove.

**Agent.** An AI assistant doing the work at a station: drafting requirements, deriving the design,
implementing, reviewing. Agents run the skills.

**Skill.** A written procedure for one station, installed into the project's `.claude/skills/` and
run by an agent. Every station has one. See `skills/README.md`.

**Command.** A slash command installed into the project's `.claude/commands/`. The four handoff
commands are `/ready-for-dev`, `/ready-for-review`, `/ready-for-qa` and `/ready-for-release`;
`/wire-handoffs` configures them. Skills are invoked the same way, as `/mainline-<name>`.

**`/mainline-help`.** The skill to run when you do not know what to do next. It reads the board,
says where your work is and which command moves it, and answers questions from the playbook.

**Person.** Decides and signs. Priority, ambiguity, what the client actually meant, and
accountability for what ships are always a person's.
