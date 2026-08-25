---
name: requirement-workflow
description: The Product side of the delivery loop — decide whether the requirement can be written at all, capture it as Gherkin scenarios in a .feature file, and pass the H1 handoff into development. Also covers receiving the scenarios the domain model entails back from the design phase. Use when a work item enters Inbox, when writing or revising a requirement, and before handing work to a developer.
---

# Requirement workflow

Product's half of the loop. It ends at **H1**, where the work changes hands.

The requirement is the spine of everything downstream. Without it there is no QA — there is nothing
to assure quality *against* — and no rewrite is possible, because there is nothing to rewrite *to*.
A codebase that "cannot be fixed" is almost always a codebase whose requirements were never written.

## 1. Triage — can somebody write the Gherkin?

The only question at Inbox. It has one expensive wrong answer.

- **Yes** → step 2.
- **No** → open a **Spike** and run `product-discovery`. Nobody can state the acceptance criteria
  because nobody yet knows whether the thing is wanted, usable, or buildable.

**Do not write the scenarios anyway.** Guessing here means the line delivers an invented spec with
full rigor — the most expensive possible way to be wrong. Sending known work through discovery costs
a few days; sending unknown work straight to a requirement costs a release.

"No" looks like: a client described an outcome rather than a system; the rules live in an operator's
head; the value of the feature is genuinely disputed; you are replacing a system nobody documented.

## 2. Write the requirement

One `.feature` file per Slice, in Gherkin. **This is the spec** — not the ticket description, not
the Figma, not the conversation it came from.

- **State what a person achieved, never what the screen did.** *"Ana refunds a R$120 cash order with
  no manager present"* survives a redesign. *"Ana clicks Refund, then Confirm"* does not, and it
  smuggles a design decision into a requirement.
- **Use the domain's words** — from the discovery glossary or the existing Tonto model. If they say
  *guia*, the scenario says guia. Inventing a synonym for a concept that already exists is how two
  models of the same thing end up in one system.
- **Every scenario traces** to an observed example or a stated business rule. A scenario with neither
  behind it is a guess wearing a spec's clothes.
- **Rules become `Rule:` blocks**, not repeated setup. Declarative, not a click script.
- Table-shaped examples of *one* rule collapse into a `Scenario Outline`. Never collapse examples of
  different rules — that hides the rule boundary, which is the thing worth seeing.

## 3. State the non-functional requirements

The scenarios describe what the system does. This is where you say what it must *be*. Each one
stated, or explicitly marked N/A:

- **Authentication and SSO** — including enterprise identity providers
- **Tenancy** — single-tenant, multi-tenant, enterprise accounts, data isolation
- **Performance** — the number, and where it is measured
- **Compliance** — regulatory regime, audit requirements, data residency
- **Data retention** — how long, and who can delete

**Why this is a numbered step and not a checkbox.** A feature once shipped without enterprise SSO
because enterprise tenancy was assumed by everyone and written down by no one. Nothing downstream
could catch it: downstream can only check what upstream wrote. "Nobody mentioned it" is not a
defence — the whole job of this step is to mention it.

"N/A" is a real answer and a fast one. An unanswered line is not.

## 4. Pass H1

The handoff into development. All of it, or the work does not move:

- [ ] A `.feature` file exists; every scenario is Given/When/Then and testable.
- [ ] Every scenario traces to an observation or a stated rule.
- [ ] Terms come from the glossary or the existing model — no new synonyms.
- [ ] Non-functional requirements stated or explicitly N/A (step 3).
- [ ] Out of scope stated explicitly.
- [ ] Prototype or screenshots attached wherever there is UI.
- [ ] **Signed** by Product · **assigned** to a developer · **notified** with everything attached.

The developer should be able to start without asking you anything. If they have to ask, the handoff
was incomplete — note what they asked, because that question is the next line on this checklist.

## 5. Receive the entailed scenarios

Expect work to come back. The design phase decomposes the domain ontologically, and the model
**entails** scenarios the requirement did not state — phase transitions, mediation bounds, lifecycle
events, non-empty collections.

This is the system working, not a rejection. The model can *prove* a scenario is missing, which is a
stronger check than any human comparing two documents and feeling satisfied. Add them, and re-pass
H1 for anything that changes scope.

## Scope changes after H1

A discovered requirement mid-build comes back here — it does not get absorbed into the branch. New
scenario, re-signed, re-handed off. Scope that enters through the back door is scope nobody
estimated, tested, or agreed to.

## Relationships

- **`product-discovery`** — run when triage says nobody can write the Gherkin. Returns a discovery
  record whose Gherkin is the first draft of step 2, plus the glossary.
- **`development-workflow`** — receives at H1.
- **`domain-modeling`** — consumes the glossary; sends entailed scenarios back at step 5.
- **`ui-exploration`** — when the open question is visual direction rather than what to build.
- **`pmi-github-project`** — the board this work item sits on. One Slice per `.feature` file.
