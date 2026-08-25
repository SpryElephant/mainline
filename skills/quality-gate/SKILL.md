---
name: quality-gate
description: The project's quality gate — every feature and module must have tests. Behavior via executable specs (BDD/Gherkin), architecture via boundary tests, bad practices caught by a static analyzer, test adequacy judged by cyclomatic complexity and CRAP score plus coverage, optional flow/reachability gates on a code-property graph, and an end-to-end suite run against the running stack. Use to define or verify quality for any feature/module, and as the binding pass/fail gate before work is considered done.
---

# Quality gate

Nothing is "done" until it passes this gate. Apply it per feature and per module.

The gate is defined by its **dimensions**, not by a particular toolchain. The dimensions and the
pass/fail discipline are invariant across languages; the concrete tools and thresholds are
**calibrated per project**. Pick one tool per dimension from the reference table, wire them behind a
single command, and make failures fail the build — never advisory warnings.

> **Reference toolchain.** A concrete JVM/Gradle stack — Cucumber-JVM, ArchUnit, PMD, JaCoCo (+ a CRAP
> check), and Joern, run behind a single `./gradlew check` — is used as the *reference row* in the
> tables below. It is illustrative only. In a Python, C#, TypeScript, Go, … project, swap in the
> equivalent tool for each dimension and keep the dimension.

## The single command

Every project exposes **one command that runs the whole gate** and exits non-zero on any failure
(`./gradlew check`, `make check`, `npm run check`, `dotnet test` + analyzers, `nox -s gate`, …). CI
runs exactly that command on every PR; branch protection requires it. Local == CI. Everything below
is a step wired into that one command.

## Dimensions

### 1. Behavior — executable specs (BDD)
- Every requirement is a spec file (Given/When/Then) that runs as a test. A requirement without a
  passing scenario is **not implemented**.
- Keep specs in the language's conventional BDD location and bind them to real code.

| Reference (JVM) | Python | C# / .NET | TypeScript / JS | Go |
|---|---|---|---|---|
| Cucumber-JVM + JUnit 5 | `behave`, `pytest-bdd` | Reqnroll / SpecFlow | `@cucumber/cucumber`, `jest-cucumber` | `godog` |

### 2. Architecture — boundaries as tests
- Encode the boundaries from `domain-modeling` as **architecture tests**: module dependency rules,
  public-vs-internal access, naming, layering.
- Architecture is enforced by tests, not by review convention. **A boundary that isn't tested isn't a
  boundary.**
- Where the design was derived from a Tonto model, the aggregate boundaries are already explicit —
  composition is containment, aggregation and association are by-ID references — so they translate
  directly into dependency rules. A part reachable outside its aggregate root is an architecture
  violation, not a style preference.

| Reference (JVM) | Python | C# / .NET | TypeScript / JS | Go |
|---|---|---|---|---|
| ArchUnit | `import-linter`, `pytest-archon` | NetArchTest, ArchUnitNET | `dependency-cruiser`, `ts-arch` | `go-arch-lint`, `depguard` |

### 3. Static analysis — bad practices fail the build
- A static analyzer runs as part of the gate. Flagged bad practices **fail the build** — they are not
  advisory warnings. Keep the report empty; the gate stays green only when it is.

| Reference (JVM) | Python | C# / .NET | TypeScript / JS | Go |
|---|---|---|---|---|
| PMD (+ SpotBugs) | `ruff`, `pylint`, `flake8` | Roslyn analyzers, `dotnet format` | ESLint, Biome | `golangci-lint`, `staticcheck` |

### 4. Test adequacy — complexity, coverage & CRAP
- **Cyclomatic complexity** per method is bounded; outliers are flagged and refactored (see
  `refactor-smells`).
- **Coverage** thresholds are enforced by the gate.
- **CRAP score** (complexity × coverage — high complexity *must* be well covered or simplified) per
  method must stay under threshold. This is the metric that couples the two: it forbids complex code
  that is also thinly tested.
- **Calibrate the thresholds per project/language** (complexity ceiling, coverage floor, CRAP ceiling).
  The reference uses coverage ≥ 0.90 line / ≥ 0.75 branch as the "well-covered" bar and CRAP ≤ 30.

| Metric | Reference (JVM) | Python | C# / .NET | TypeScript / JS | Go |
|---|---|---|---|---|---|
| Coverage | JaCoCo | `coverage.py`, `pytest-cov` | coverlet | `c8`, `nyc` | `go test -cover` |
| Complexity | JaCoCo `COMPLEXITY` | `radon`, `cognitive-complexity` | Roslyn metrics | ESLint `complexity`, `ts-complexity` | `gocyclo` |
| CRAP | JaCoCo-derived check | derive from coverage + complexity | derive | derive | derive |

CRAP is rarely built in — derive it wherever coverage and per-method complexity are both available
(`CRAP = c²·(1−cov)³ + c`).

### 5. Flow-level gates — code-property graph (optional, high-value)
Architecture and static-analysis tools stop at the **declaration boundary**: who references whom, and
bad shapes inside one method. They cannot follow a *value* across methods, reason about *reachability*,
or check *what happens on every path*. A code-property graph (CPG) adds that axis, so flow and
call-presence invariants become gates.

| Reference | Polyglot alternative |
|---|---|
| Joern (C/C++, Java, Python, JS/TS, Go, …) | CodeQL |

- **When to reach for it.** Only for flow / reachability / call-presence facts. If an architecture test
  can express it cheaply, it belongs there — not here. Good fits: defensive-copy / immutability
  (data-flow), error-handling discipline (control-flow), "X is only reached through Y" (call graph),
  domain-leak-across-the-wire (taint).
- **Authoring loop — never skip.** Probe how the construct actually appears in the CPG (a throwaway
  query dumping `code` / `methodFullName` / the calls in a method) *before* writing the assertion.
  Guessing node shapes produces brittle gates.
- **Precision rule.** Key on **resolved fullNames** (`methodFullName`, `typeFullName`), not class-name
  string regexes. A name regex is a false-positive magnet, and a gate that false-positives once is dead
  — nobody trusts it again.
- **Scope to zero-false-positive sets.** Enforce on populations that are unambiguous (e.g. value-object
  types for immutability rules) and document carve-outs explicitly. Widen only when a marker makes the
  wider set unambiguous too.
- **Each gate is a small function** returning violation strings, registered in a `gates` list. The
  runner loads the CPG once and exits non-zero on any violation. Prove every new gate *bites*: inject
  the violation, confirm FAIL, revert.
- **Mechanics.** Provision the CLI out-of-repo (download once, never vendor). Build the CPG from
  sources, run the gate script, wire both into the single gate command, and allow a local skip flag.

### 6. End-to-end behavior — the running system

Dimension 1 binds specs to **code**. This one binds to a **running system**: a browser or a real
client against the full stack, real navigation, real persistence, no test doubles at the boundary.
It is the only dimension that can catch a defect living *between* correctly-implemented modules.

**The developer is gated on regression, not on authorship.** This dimension asserts that the
existing suite still passes. Growing the suite for new behavior belongs to QA (`e2e-suite`), at the
QA station. The split is deliberate: gating a developer on tests that QA owns and must author would
deadlock the merge, and letting developers write throwaway E2E tests to unblock themselves produces
a suite nobody trusts.

- **Runs against the local full stack** (`local-stack`), so it behaves identically on a laptop and in
  CI. An E2E suite that only runs in CI cannot be debugged.
- **Flake is failure.** A flaky test is worse than no test — it teaches the team to re-run the gate
  instead of reading it, and one re-run habit kills every dimension's authority. Quarantine a flaky
  test out of the gate the day it flakes, with a ticket. Never retry-until-green.
- **Runtime budget.** If the suite outgrows the PR window, split it explicitly: a **smoke set** on
  every PR, the **full set** on the release path. Write down which is which. A suite that silently
  stopped running on PRs is the most expensive kind of green.

| Reference | Python | C# / .NET | TypeScript / JS | Go |
|---|---|---|---|---|
| Playwright | `playwright-python` | Playwright .NET | Playwright, Cypress | `chromedp`, `rod` |

For a headless system with no UI, the same dimension applies at the API boundary against the running
stack (REST-assured, `supertest`, `httpx`) — the point is the *deployed shape*, not the browser.

## The gate — all applicable dimensions must pass
1. Behavior specs green
2. Architecture rules green
3. Static analysis clean
4. Complexity within bounds
5. CRAP / coverage within thresholds
6. Flow / CPG gates green *(if the project uses dimension 5)*
7. End-to-end suite green against the running stack

Run the project's single gate command. Not done until it is green — never weaken a spec or lower a
threshold to pass.

## Not a gate dimension: domain-model validation

`tonto-cli validate` belongs to the **design phase** (`domain-modeling`), not to this gate. The gate
binds only on tools whose failure always means the *code* is wrong, and it is the proof — coupling it
to a 0.4.x single-maintainer tool would put its authority at the mercy of an upstream regression.

Guarding against model rot is still worth doing, as a **separate CI job** that asserts the `.tonto`
files still parse and validate. Keep it out of the single gate command, and let it fail independently.
