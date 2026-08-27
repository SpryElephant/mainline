---
name: mainline-product-discovery
description: Validate a product or feature by prototyping it before any requirement is committed, and turn what the prototype taught you into high-quality Gherkin. Runs outside the quality gate on quarantined throwaway code, records typed observations during real sessions, and hands /mainline-requirement-workflow a discovery record with scenarios, domain vocabulary, and accepted risks. Use when nobody can yet say what should be built, or when the riskiest thing about a feature is whether anyone wants it.
---

# Product discovery

Some work arrives without a requirement. Nobody can write the Gherkin because nobody yet knows
whether the thing is wanted, usable, or buildable. Writing the scenarios anyway invents them, and
the line then delivers an invented spec with full rigor — the most expensive possible way
to be wrong.

This skill sits **before** `/mainline-requirement-workflow`. It buys the missing knowledge with a throwaway
prototype and returns a **discovery record**: validated scenarios, the participants' own vocabulary,
and the risks nobody retired.

> **No gate here.** Discovery code is quarantined, untested, and deleted at the end. What crosses
> into production is what the prototype *taught*, never what it *is*. The rigor moves from the code
> to the learning — the four checks in step 6 replace `/mainline-quality-gate`.

## When to use

| Use it | Do not use it |
|---|---|
| Nobody can state the acceptance criteria | The requirement is known — go to `/mainline-requirement-workflow` |
| A client describes an outcome, not a system | A bug fix, or a change inside existing contracts |
| The rules live in an operator's head, unwritten | The rules are already written down somewhere |
| Replacing a legacy system nobody documented | The legacy system's spec exists and is trusted |
| A feature whose value is genuinely disputed | A feature whose value is agreed and whose shape is not — that is design |

## The gap this closes

Every standard prototyping process — Design Sprint, Design Thinking, Lean Startup, continuous
discovery — ends in a *decision* plus tacit knowledge in the heads of whoever watched the sessions.
None of them emits a requirement. Six weeks later the tacit knowledge is gone, and the team writes
the scenarios from memory, which means from imagination.

The fix is one cheap artefact recorded live, and one synthesis step:

```
prototype session  →  observation log  →  Example Mapping  →  discovery record
   (live, raw)         (typed lines)       (synthesis)         (Gherkin + terms + risks)
                                                                      ↓
                                                 /mainline-requirement-workflow
```

## Process

### 1. Name one riskiest assumption

Write the single sentence that, if false, makes the work pointless. Then classify it — the class
picks the prototype.

| Risk | The question | What the prototype must do |
|---|---|---|
| **Value** | Will anyone use it? | Offer the thing for real and count who takes it |
| **Usability** | Can they work it out unaided? | Put a working flow in front of the actual user |
| **Feasibility** | Can we build it? | A spike against the real system or the real data |
| **Viability** | Does it work for the business — legal, support, margin? | Not a prototype. A stakeholder walkthrough |

**One risk per prototype.** A prototype attacking two answers neither: a usability test on a feature
nobody wants produces confident findings about nothing.

State the assumption as falsifiable, with a threshold. "Ops staff will refund without calling a
manager" is testable. "Ops staff will like the new screen" is not.

Fix the **timebox** here, in days, and write it down next to the assumption.

### 2. Choose the medium and the participant

The observation vocabulary in step 4 never changes. The medium and the participant do — calibrate
them to the project, the way `/mainline-quality-gate` calibrates tools to a language.

| Project nature | Riskiest thing, usually | Prototype medium | Participant |
|---|---|---|---|
| Consumer / UI-led | Value, then usability | Clickable prototype — use `/mainline-ui-exploration` | 5 target users per round |
| Internal or ops tool | Rules living only in people's heads | Wizard of Oz over the real process | The operator doing the job today |
| API / platform / SDK | Usability of the contract | Write the client code first; fake the server | A developer who is not you |
| Data / ML | Feasibility, and what the data really contains | Notebook over a real extract | Whoever must act on the output |
| Legacy replacement | Feasibility, undocumented behavior | Read-only spike against the live system | The current system's owner |
| Regulated (fintech, health) | Viability — legal and audit | Paper walkthrough of the rules | The compliance owner |
| Workflow / marketplace | Value, on both sides | Concierge: deliver it by hand, no software | One participant from each side |

The cheapest medium that can falsify the assumption wins. Techniques in rough cost order: **fake
door**, **landing-page test**, **concierge** (manual delivery), **Wizard of Oz** (a human behind the
curtain), **paper prototype**, **clickable prototype**, **live-data prototype** (real reads, fake
writes), **spike** (code written to answer one question, then deleted).

### 3. Build the prototype — quarantined

These rules are what make dropping the engineering gate safe rather than reckless:

- It lives in `prototypes/<name>/`, outside the build.
- CI does not run it. Coverage ignores it. `/mainline-quality-gate` never sees it.
- **No production module may import from it.** Enforce that with one architecture rule in the real
  project — the only place discovery touches the gate.
- Real domain content, never lorem or placeholder greys. Fake content hides the problems the
  prototype exists to find.
- The whole flow, end to end, and every advancing control works. A dead primary button is a lie
  about the design.
- It is archived or deleted in step 7.

Build only what the assumption needs. The settings screen never decided anything.

### 4. Run the sessions and record observations

Record **seven types**, one line each, in one file per session. Each type earns its place by being
**unrecoverable once the session ends**.

| Tag | What it is | Why it must be captured live |
|---|---|---|
| `RULE` | A constraint someone asserted in passing | "We'd never do that for a corporate account" — said once, written nowhere |
| `EX` | A concrete run: real data in, real outcome out | The specific values *are* the scenario |
| `TERM` | The participant's word for a thing, and ours | The cheapest source of ubiquitous language there is |
| `FRIC` | Where a person stalled, backtracked, or misread | Behavioral. A transcript does not contain it |
| `ASSUM` | Something the prototype bakes in with no evidence | The person who baked it forgets within a week |
| `Q` | An open question | Cheap to note, expensive to rediscover |
| `CHOICE` | Direction taken, direction rejected, and why | Stops the team re-litigating it in month three |

**One line per observation. No forms.** Anything with fields to fill in does not get filled in
during a live session.

```
RULE   Refund over R$500 needs a manager PIN. (Ana, ops lead)
TERM   They say "guia". We said "invoice". A guia has a barcode and a due date.
EX     Ana refunded order 4471, R$120, cash, no PIN prompted, 40s.
FRIC   Ana clicked "Close" expecting a save. Backtracked twice.
Q      Partial refund against a split payment — what happens?
ASSUM  We assumed one till per store. Ana mentioned two on Saturdays.
CHOICE Single screen over a wizard. Ana never reads step 2.
```

**Division of labor.** Record the session. The live note-taker writes only `FRIC` and `EX` — the two
a transcript cannot reconstruct. The other five come from the transcript afterwards, first pass by
Claude, corrected by someone who was in the room.

**Session discipline:** think-aloud protocol, five participants per round (the sixth finds almost
nothing new), and fix between participants when a defect blocks the flow. Give tasks, never
instructions — "refund this customer", then silence. The moment you explain the screen, the
usability finding is gone.

Template: `references/observation-log-template.md`.

### 5. Synthesize by Example Mapping

Turn the logs into cards — the standard four colors (Wynne), which map onto the tags exactly:

| Card | From | Becomes |
|---|---|---|
| Yellow — the story | The assumption from step 1 | The `Feature:` |
| Blue — a rule | `RULE` | A `Rule:` block |
| Green — an example | `EX` | A `Scenario`, or one row of a `Scenario Outline` |
| Red — a question | `Q` | Blocks its rule until answered |

The other three tags do not become cards. They still have destinations:

| Tag | Goes to | Becomes |
|---|---|---|
| `TERM` | `/mainline-domain-modeling` | A Tonto class, role, or relation name — and the words used in the Gherkin |
| `FRIC` | The discovery record | A design decision. Usually **not** a requirement; occasionally it exposes a missing `RULE` |
| `ASSUM` | The risk register, via `/mainline-file-finding` | A test to run, or a risk explicitly accepted |
| `CHOICE` | The discovery record | A short decision entry, not a requirement |

### 6. Pass the discovery gate

There is no code quality bar here. Discovery still needs an exit condition, or it becomes an
unbilled swamp. Four checks, all free:

1. **Every rule has at least one example.** A rule with no example is somebody's opinion.
2. **Every example maps to exactly one rule.** An example under no rule means a rule is missing. An
   example that fits three means the rule is too coarse — split it.
3. **Zero open red cards on anything you hand over.** An unanswered question is where invented
   requirements come from. Answer it, or drop its rule from this round.
4. **The riskiest assumption is resolved** — confirmed, refuted, or explicitly deferred as a risk.

Plus the **timebox** fixed in step 1. Discovery ends on the date, not on a definition of done.
Whatever is unfinished becomes a risk in the record, never an extension.

### 7. Write the discovery record and destroy the prototype

One record per discovery effort — `docs/discovery/<name>.md`, the only artefact that survives.
Template: `references/discovery-record-template.md`. It carries:

1. The riskiest assumption, its risk class, and the timebox.
2. The medium built, and who participated.
3. The verdict — **go**, **no-go**, or **pivot** — with the evidence.
4. The Gherkin handed to `/mainline-requirement-workflow` step 2.
5. The glossary from `TERM`, handed to `/mainline-domain-modeling`.
6. The decisions from `CHOICE`.
7. Assumptions accepted without evidence, handed to the risk register.

Then delete or archive `prototypes/<name>/`. A prototype kept alive gets imported eventually.

## Writing the Gherkin

Prototype-derived scenarios fail in one specific way: they describe the prototype. Such a scenario
dies with the throwaway code it narrates.

- **State what the person achieved, never what the screen did.** "Ana refunds a R$120 cash order
  with no manager present" survives. "Ana clicks Refund, then Confirm" does not.
- **Use the participant's words**, from `TERM`. If they say *guia*, the scenario says guia.
- **Every scenario traces to a real `EX`.** Cite the session and the line. A scenario with no
  observation behind it was invented, and preventing that is what this skill exists for.
- **Rules become `Rule:` blocks, not repeated setup.** Declarative, not a click script.
- Table-shaped examples of one rule collapse into a `Scenario Outline`. Never collapse examples of
  *different* rules — that hides the rule boundary.

These scenarios are the *first draft* of the spec, not the last. `/mainline-development-workflow` step 1 entails
more of them from the domain model.

## Handing off to `/mainline-domain-modeling`

`TERM` seeds the ontology, so the Tonto model does not start from a blank page:

- Nouns the participant treats as independent things → candidate **kinds**.
- "While they are an *X* they can *Y*" → **roles** and **phases**.
- "The agreement / contract / booking between A and B" → a **relator**.
- Lifecycle verbs in sequence — *issued, settled, archived* → a **phase** partition, and the
  transitions between them become entailed scenarios.

Two words for one thing in the log is a terminology conflict. Resolve it in the model, then use the
one word everywhere. Discovery finds these conflicts for free; production finds them expensively.

## Failure modes

- **The prototype becomes the product.** The most expensive failure in the field. Cured by the
  quarantine in step 3, not by intending to rewrite it later.
- **Two risks, one prototype.** Usability findings about a feature nobody wants. Split the round.
- **A demo instead of a test.** The team explains the screen, the participant is polite, everyone
  learns nothing. Give a task, then stay quiet.
- **Scenarios written from memory a month later.** The log is the whole point. No log, no discovery
  record, no handoff.
- **Discovery with no timebox.** Unbounded exploration on a client's budget. Fix the date in step 1.
- **Confirming the assumption instead of testing it.** Write the falsification threshold down
  *before* the first session, and hold to it.
- **Handing over red cards.** An open question becomes an invented answer downstream. Resolve it or
  drop it.

## Relationship to the other skills

```
/mainline-product-discovery  ──►  /mainline-requirement-workflow  ──►  /mainline-development-workflow  ──►  /mainline-quality-gate
  (no gate,                         (Gherkin)                         (/mainline-domain-modeling →       (binding)
   throwaway code)                                                     implementation)
```

- **`/mainline-ui-exploration`** is one medium inside step 3 — use it when the risk is usability or visual
  direction and the answer needs several worlds compared, not one prototype tested.
- **`/mainline-requirement-workflow`** consumes the discovery record at step 2. Discovery never produces code that
  ships.
- **`/mainline-domain-modeling`** consumes the glossary.
- **`/mainline-pmi-github-project`** holds a discovery effort as its own work item — a **Spike**, timeboxed,
  outside the `Phase` pipeline — and receives the accepted assumptions into the risk register.
