---
name: e2e-suite
description: Grow and maintain the end-to-end test suite that runs against the running stack — the QA station's compounding asset and dimension 6 of quality-gate. Covers what deserves an E2E test and what does not, writing tests that trace to requirements rather than to screens, seed data and isolation, and flake discipline. Use at the QA station whenever verifying a change on staging, and whenever a defect is found that should never recur.
---

# End-to-end suite

The suite is **QA's compounding asset**. Every release either grows it or wastes the work that went
into that release's testing.

Two owners, deliberately split:

- **QA owns the content.** What is in the suite, what it asserts, what gets added after each round.
- **`quality-gate` owns the enforcement.** Dimension 6 runs the suite and fails the build. Developers
  are gated on **not breaking it**, never on authoring it.

That split is what keeps both jobs honest. A developer required to write the E2E test for their own
feature writes the one that passes. A QA team whose tests do not block a merge is writing
documentation.

## What deserves an E2E test

E2E is the most expensive test you can write and the slowest to run. Spend it where nothing cheaper
works:

| Write an E2E test | Do not |
|---|---|
| A journey that crosses modules, services, or the network | Anything one module can prove alone — that is dimension 1 |
| A defect that reached staging or production | A defect a unit test would have caught — write that instead, and ask why it did not exist |
| The path that costs the most if it breaks — sign-in, checkout, the thing the client demos | Every branch of a form. Combinatorics belong in unit tests |
| Behavior that only appears when real persistence, real auth, or a real queue is involved | Styling, copy, layout |

**The rule of thumb:** if you can prove it without starting the whole system, you should.

## Writing a test

1. **Start from the requirement, not the screen.** Open the `.feature` file for the Slice. The E2E
   test asserts what the scenario says a person achieved. If there is no scenario to trace to, stop —
   you have found a missing requirement, which is worth more than the test. Send it to Product.
2. **Use the domain's words.** The same glossary the Gherkin uses. A test that reads
   `refundGuia(order)` survives a redesign; `clickButton('#btn-2')` does not.
3. **Assert outcomes, not steps.** "The refund appears on the customer's statement," not "the confirm
   dialog closed." Steps are how you got there and they change constantly; outcomes are the
   requirement.
4. **Put the selectors behind a page object** (or screenplay tasks). One redesign should break one
   file, not forty. A suite that is expensive to update is a suite that gets deleted.
5. **Make it independent.** Every test creates the state it needs and cleans up, or runs against a
   reset database. Tests that depend on execution order fail in parallel CI and get marked flaky when
   they are simply wrong.
6. **Make it deterministic.** No sleeps — wait for the condition, not the clock. No dependence on
   today's date, the current user's locale, or a record that happens to exist.

## Seed data

The suite is only as reproducible as its data.

- Seed data lives in the repo and is applied by a command, not by a person restoring a dump.
- Every scenario's fixtures are named for the situation they create — `customer-with-two-open-guias`,
  not `test_data_3`.
- Seeding is fast enough to run per test class. If it is not, that is a `local-stack` problem worth
  fixing — slow seeding silently pushes people toward shared mutable state.
- **Never seed from a production copy.** Beyond the obvious data-protection problem, it makes the
  suite depend on facts nobody wrote down.

## Flake discipline

**A flaky test is worse than no test.** It teaches the team to re-run the gate instead of reading it,
and that one habit destroys the authority of every other dimension.

1. A test that flakes is **quarantined out of the gate the same day**, with a ticket. Not next sprint.
2. Fix the cause, which is almost always a race the test exposed and the application owns. Treat a
   flaky test as a bug report about the system before you treat it as a bug in the test.
3. **Never add a retry to make it green.** A retry is a decision to stop knowing whether the feature
   works.
4. Track the quarantine list. If it is growing, the suite is dying, and that belongs in the
   improvement loop.

## Runtime budget

The suite must stay inside the window a developer will actually wait for.

- Start with the whole suite on every PR. It is fine until it isn't.
- When it outgrows the window, split explicitly: a **smoke set** — the highest-cost-if-broken
  journeys — on every PR, and the **full set** on the release path. Write down which tests are in
  which and why.
- Parallelize before you cut. Most suites are slow because they are serial, not because they are big.
- **Never let it silently stop running.** A suite that quietly dropped off the PR path is the most
  expensive kind of green there is.

## The QA loop

At the QA station, per round:

1. Run the suite against staging.
2. Explore what the suite cannot express — the odd sequence, the real-world combination, the thing
   that is technically correct and practically wrong.
3. **Everything repeatable you found becomes a test, this round.** If you have run the same manual
   steps three times, that was a test case you have not written yet.
4. File defects against the requirement they violate.
5. The new tests merge with the release, not after it.

## Failure modes

- **The suite tests the UI instead of the system.** Symptom: a redesign breaks fifty tests and none of
  them found a bug. Cure: assert outcomes, hide selectors behind page objects.
- **Retry-until-green.** The single fastest way to make a gate worthless.
- **QA writing tests nobody runs.** If the suite does not block a merge, it is documentation. Wire it
  into dimension 6.
- **Developers writing E2E tests to unblock themselves.** They write the test that passes. Keep
  authorship with QA and gating with the gate.
- **The suite only runs in CI.** Then it cannot be debugged, and every failure becomes an archaeology
  project. It must run against `local-stack`.
- **Coverage theatre.** Two hundred E2E tests covering form validation, and none covering checkout.
  Spend the expensive tests on the expensive failures.

## Relationships

- **`quality-gate`** — dimension 6 runs this suite. QA owns what is in it.
- **`local-stack`** — what the suite runs against, locally and in CI.
- **`requirement-workflow`** — the `.feature` files every test traces back to.
- **`observability`** — a defect that reached production should usually produce both an E2E test and
  an alert. The test stops the recurrence; the alert catches the next thing you did not think of.
