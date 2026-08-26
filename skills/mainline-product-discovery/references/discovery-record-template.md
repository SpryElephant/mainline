# Discovery record — <effort name>

Status: <go | no-go | pivot>
Timebox: <start> → <end> (fixed before the first session)
Sessions: <n> with <n> participants
Prototype: <path>, <archived | deleted> on <date>

---

## 1. The riskiest assumption

> <One falsifiable sentence. If this is false, the work is pointless.>

Risk class: <value | usability | feasibility | viability>
Falsification threshold, set before session 1: <the number or the observable outcome>
Result: <confirmed | refuted | deferred as a risk>, on this evidence: <what was observed>

## 2. What we built, and with whom

Medium: <fake door | landing page | concierge | Wizard of Oz | paper | clickable | live-data | spike>
Why this medium: <the cheapest one that could falsify the assumption>
Participants: <roles, not names, and how they were recruited>

## 3. Verdict

<Two or three sentences. What we now know that we did not know before, and what follows from it.>

## 4. Gherkin handed to `mainline-requirement-workflow`

Each rule with its examples. Every scenario cites the observation behind it.

```gherkin
Feature: <the story>

  Rule: <blue card, in the participants' words>

    Scenario: <what the person achieved — never what the screen did>   # session 2, EX line 14
      Given ...
      When ...
      Then ...
```

Rules carried with no scenario yet, and why: <or "none">

## 5. Glossary handed to `mainline-domain-modeling`

| Their word | Our old word | What it is | Ontological candidate |
|---|---|---|---|
| | | | kind / role / phase / relator |

Terminology conflicts to resolve in the model: <two words for one thing, one word for two things>

## 6. Decisions taken

| Decision | Rejected alternative | Why |
|---|---|---|

Friction observed that stays a design concern, not a requirement:

## 7. Risks handed to the register

| Assumption accepted without evidence | Impact if wrong | Retire it by |
|---|---|---|

Left unfinished when the timebox expired:
