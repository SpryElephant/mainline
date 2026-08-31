# <Project> — Project Charter

> A living, one-page charter: *why the project exists*, *what "success" means*, *how work is governed*.
> Keep it short — detail lives in the design/decision log, the quality gate, and the Project board.

- **Project board:** <link to the GitHub Project>
- **Repository:** `<owner>/<repo>`
- **Sponsor / Product Owner / Lead:** <name>
- **Management style:** Full-artifact PMI rigor, **kanban flow** (pull-based, no fixed sprints).

## 1. Vision
<One paragraph: the outcome the project exists to produce.>

## 2. Objectives
1. <objective>
2. <objective>

## 3. Success criteria (measurable)
- <criterion — e.g. coverage/quality/delivery-discipline targets>

## 4. Scope
**In scope**
- <deliverable groups>

**Out of scope (for now)**
- <explicitly excluded items — name them so scope stays honest>

## 5. Milestones (release gates — dates are targets, not commitments)
| Gate | Contents |
|---|---|
| **M1 — <name>** | <contents> |
| **M2 — <name>** | <contents> |
| **Backlog** | demand-driven |

## 6. Constraints & assumptions
- **Constraint:** <e.g. single maintainer; WIP-limited to one epic at a time>
- **Constraint:** <e.g. the quality gate is binding — it cannot be waived to hit a date>
- **Assumption:** <key assumption, validated per item in the design phase>

## 7. Governance — how work flows
Every requirement flows **Requirement → Design → Build → Verify → Done**, mirrored by the board's `Phase`:
1. **Requirement** — the executable spec (e.g. acceptance tests / scenarios).
2. **Design** — decompose + define module API; record it as a numbered design doc (ADR).
3. **Build** — implement, gated continuously by the quality gate.
4. **Verify** — run the gate, the binding pass/fail checks (tests, architecture, static analysis, coverage).

The step-by-step for working an item, and the rule that **design docs are the durable record while
issues are transient trackers that link to them**, lives in the design-doc README.

## 8. Change control
The scope baseline is <roadmap doc> + the milestones. A material change to scope, sequence, or a
committed decision is a **`type:change-request`** issue → recorded as a new, higher-numbered design
doc that supersedes the old one (append, never rewrite).

## 9. Risk
Risks live on the board as **`type:risk`** items, scored Probability × Impact → Exposure, reviewed
regularly. Current top exposure: <risk> (<Exposure>).

## 10. Versioning & branching
Trunk-based: `master` always releasable; short-lived branch per item → PR (`Closes #N`) →
squash-merge → delete. The quality gate runs in CI and is required by branch protection. Releases
follow **SemVer** via tags + GitHub Releases at milestone completion, staying **pre-1.0 (`0.x`)** until
the public contracts stabilise.
