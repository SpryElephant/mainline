---
name: refactor-smells
description: Smell-driven, behavior-preserving refactoring in the Fowler sense — rank existing code by cyclomatic complexity, name the actual code smell, and apply a catalog refactoring (Replace Conditional with Polymorphism, Decompose Conditional, Extract Class, Replace Magic Literal, etc.) that lowers complexity while the quality gate proves behavior is unchanged. Use to improve the structure of existing code with no new requirement. NOT for mechanical package/type moves (that is `refactoring`) and NOT for new behavior (that is `development-workflow`).
---

# Refactor smells (Fowler)

Improving the *structure* of existing code without changing what it does. Driven by metrics (where to look) and the smell catalog (what to fix), proven safe by `quality-gate`.

This is **not** the `refactoring` skill. That one is mechanical reference rewriting (package/type moves) via an automated rewrite engine. This one is judgment-driven structural improvement — replacing conditionals with polymorphism, decomposing fat methods, killing magic literals and type codes, extracting classes. It shares the words "behavior-preserving refactoring" with `refactoring`, but it is a different craft.

It is also **not** `development-workflow`: no new requirement, no new behavior, no Gherkin scenario. The existing scenarios *are* the spec — they must stay green before and after, unchanged.

## Why this is safe here

Fowler's method has one precondition: **self-testing code**. This method is only licensed where the `quality-gate` exists — behavior specs, architecture tests, coverage, complexity/CRAP, and any flow gates. That safety net is the entire license to refactor: every move is validated by the gate command, not by inspection. On a codebase without the gate, this skill would be reckless. With it, it is disciplined.

## What "better" means — and does not

The goal is **the smell removed and complexity/CRAP down, with the gate still green.** That is the success metric.

**"Less code" is a side effect, never the goal.** Several of the most valuable refactorings *add* lines and types — Replace Conditional with Polymorphism trades one fat `if/else` for several small, named classes. Optimizing for terseness produces clever, dense code, which is the opposite of the intent. Optimize for *clarity and lower complexity*, and let line count fall where it may.

## Process

One target at a time. The loop:

1. **Target — metrics say *where*.** Read the ranked complexity table (see "Selecting targets" below for what it is and why). Pick the top **READY** row — highest cyclomatic complexity that is already well-covered. **CHARACTERIZE-FIRST** rows are the next step's backlog, not refactor targets yet.

2. **Triage — is it safe to touch yet?** CRAP is `complexity × (1 − coverage)`, so a high score can mean *badly structured* **or** merely *under-tested*.
   - High complexity **and** good coverage → safe to refactor now.
   - High complexity **and** thin coverage → **characterization tests first.** Pin the current behavior (including the ugly edge cases) with tests *before* changing structure. The coverage gap is exactly where a "simplification" silently changes behavior the gate can't catch. This is Feathers' discipline — get the code under test, then refactor.

3. **Diagnose — name the smell.** This is the judgment step. Identify the actual smell(s) from the catalog below; don't refactor a number, refactor a named problem. The static analyzer already flags some smells — those flags can *also* feed the target list.

4. **Apply one named refactoring.** Pick the catalog move that addresses the smell. One refactoring, small and reversible.

5. **Prove it — the gate is the proof.** Run the gate command. Green → the behavior is unchanged and the structure is better. Red → the refactoring changed behavior or broke an invariant; revert and reassess. Never weaken a scenario or threshold to pass.

6. **Commit, then loop.** Land it as its own standalone, behavior-preserving commit (respect the no-commit-without-permission rule — propose it, don't auto-commit). Then pick the next target.

## Smell → refactoring catalog (starting set)

- **Long / complex method** (high cyclomatic) → Extract Method, Decompose Conditional, Replace Nested Conditional with Guard Clauses, Replace Loop with Pipeline.
- **Switch/`if-else` on a type code or `kind`** → Replace Conditional with Polymorphism / Replace Type Code with State or Strategy. (If a type/`kind` tag is *intentionally* an uninterpreted carrier, refactor the consumers that branch on it, not the carrier.)
- **Magic literals / strings** → Replace Magic Literal with named constant; or push the value into a value object / enum.
- **Large class, low cohesion** → Extract Class, Move Method, group by responsibility.
- **Primitive obsession** → Introduce Value Object (fits the DDD model — prefer an immutable value type).
- **Feature envy / shotgun surgery** → Move Method/Field to the data it acts on.
- **Repeated conditional shape across the codebase** → unify behind a polymorphic type or strategy.

The catalog is open — add the move that fits. The constraints are fixed: one smell at a time, gate-green between each.

## Selecting targets

Target selection is a **reporting-only** read of the metrics the gate already produces — it never fails the build. The single machine-readable source is the gate's **per-method coverage + complexity report** (the JVM reference parses JaCoCo's `jacocoTestReport.xml`; use the equivalent coverage report in your stack). The static analyzer emits only threshold *violations* (the gate keeps that empty), and any flow/CPG gates are boolean — so the coverage report is the one source with per-method numbers.

Per method the report gives cyclomatic complexity `c`, line coverage `cov`, and branch coverage. Rank the methods in a table sorted by **`c` descending**, with columns: `c`, a **triage** flag, `cov` / `branch`, CRAP, and the method id.

**Rank by raw complexity, not CRAP — this is the locked decision.** CRAP is `c²·(1−cov)³ + c`; it only grows when *coverage* is low, so it flags *undertested* code, not *badly-structured* code. The gate already fails any method above the CRAP ceiling, so by construction every method is under it and ranking by CRAP would surface nothing actionable. Complexity is what identifies a refactoring target; this skill ranks comparatively (the worst offenders relative to the codebase), since there are no threshold breaches to chase.

**Coverage drives triage, not the rank.** The triage flag reuses the gate's *own* coverage floors — no new constants:
- `cov ≥ <line floor>` **and** `branch ≥ <branch floor>` → **READY** — safe to refactor now.
- otherwise → **CHARACTERIZE-FIRST** — pin behavior with tests before touching structure (step 2 of the loop).

Show the CRAP column only as *distance to the gate's ceiling*, so you can see which targets also sit near a gate failure. Tie-break among equal `c` is CRAP descending.

**Exclusions** (kept off the list as noise): generated sources (already excluded from coverage), `equals`/`hashCode`/`toString`/accessors and their language equivalents, and anything below a complexity floor (e.g. `c < 4`) — the list is only the genuinely branchy methods. A per-class complexity-sum view also surfaces God-class candidates.

The task is reporting-only and orthogonal to the gate: it informs target selection, it does not gate.

## Invariants

- **Zero behavior change.** Scenarios are unchanged and green before *and* after. If a scenario must change, this is not a refactor — it is `development-workflow`.
- **The gate is the proof, never inspection.** Not done until the gate is green. Never lower a threshold or soften a test to make a refactor "pass."
- **Tests first when coverage is thin.** Don't refactor code the gate can't see. Characterize, then refactor.
- **One smell, one refactoring, one commit.** Keep diffs bisectable; never batch unrelated smell-fixes.
- **Clarity over brevity.** Lower complexity and a removed smell, not fewer lines, is the win.
