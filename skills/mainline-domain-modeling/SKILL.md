---
name: mainline-domain-modeling
description: Decompose a requirement ontologically in OntoUML/UFO, written as a validated Tonto (.tonto) model, then derive the design from it — aggregates, entities, value objects, domain events, module responsibilities and public API — plus the Gherkin obligations the model entails. Use in the design phase, before implementation, whenever adding or changing a module or feature.
---

# Domain modeling (OntoUML/UFO → DDD)

Turn a requirement into a validated ontological model, and derive the design from it: what the
aggregates are, where their boundaries fall, which module owns what, and what the public API is —
before any code is written.

The ontology is not documentation. It is the artefact the design is *derived* from, it lives in the
repo next to the code, and it is machine-validated.

**Language-agnostic.** OntoUML says what exists; the derivation in `references/ufo-to-ddd-derivation.md`
says what that means for a design. "Module" is whatever unit of encapsulation the target language
uses — package, namespace, assembly, crate — and "public API" is whatever it exposes as a boundary,
with everything else internal.

## Why ontology first

DDD's tactical patterns are implementation vocabulary; they do not tell you whether `Customer` is a
thing or a role, whether a status is a subtype or a state, or where an aggregate boundary belongs.
UFO does, because it makes **identity**, **rigidity**, and **existential dependence** explicit — and
those three settle most of the design. The derivation is therefore largely mechanical, and the
errors it prevents (duplicate party records, attributed join tables, identity-changing state
transitions, polymorphic foreign keys) are ones DDD's vocabulary cannot even name.

Use the **full UFO taxonomy**. Every stereotype earns its place in the derivation — `historicalRole`
gives you non-revocable status, `situation` + `@triggers` gives you policies, `type` vs `powertype`
tells you whether a classification is data or code. Scope discipline belongs on *which concepts a
requirement pulls in*, never on which stereotypes are allowed.

## Prerequisites

Requires **Node ≥ 20** (the package's declared engine).

```bash
npm install -g tonto-cli@0.4.13     # pin: 0.4.12 and earlier have broken commands
npm ls -g tonto-cli                 # verify — see note below
```

**Do not verify with `tonto-cli --version`.** Up to and including 0.4.12 it reports a hardcoded
`0.4.0` regardless of what is installed, so it cannot tell you which version you have. Fixed in
0.4.13. Use `npm ls -g tonto-cli`.

No-install alternative, which is how the tooling table below was produced:

```bash
npx -y tonto-cli@0.4.13 validate .
```

Two `npm install -g` warnings about blocked `keytar` / `@vscode/vsce-sign` postinstall scripts are
expected and harmless — they are VS Code extension dependencies the CLI does not use.

Recommended: the [Tonto VS Code extension](https://marketplace.visualstudio.com/items?itemName=Lenke.tonto)
(`Lenke.tonto`) for live validation and diagram views. Diagrams come from the extension, not the
CLI — see the tooling note below.

If `tonto-cli` is unavailable, still write the `.tonto` model — the ontological discipline is most
of the value — but say plainly in the output that it is unvalidated.

**Do not run `tonto-cli add-skill`.** It installs upstream's own `tonto-ontology` skill, which
covers ontology-internal tasks only, has no design derivation, and pauses for user confirmation
mid-flow. Its reference material is already vendored here; `ATTRIBUTION.md` explains the split.

## Reference files

Consult these rather than working from memory. The first four are vendored from the Tonto project
(MIT, NEMO/UFES); see `references/ATTRIBUTION.md`.

| File | Use for |
|---|---|
| `references/ufo-stereotype-selection.md` | **Choosing a stereotype.** The UFO decision tree: substantial / moment / perdurant, with Tonto snippets per branch |
| `references/tonto-language.md` | **Syntax and ontological semantics.** Full grammar, every stereotype and relation stereotype, gensets, multi-level, worked examples, common errors |
| `references/cardinality.md` | **Reading and choosing cardinalities.** Reading direction is easy to get backwards |
| `references/terminology-audit.md` | **Auditing names against stereotypes** — "is `Child` a subkind or a phase?" |
| `references/ufo-to-ddd-derivation.md` | **The bridge to design.** Ours, not upstream. Aggregates, boundaries, events, invariants, Gherkin obligations, anti-patterns |
| `references/example.tonto` | **A validated model to copy syntax from.** Ours. Passes `tonto-cli validate`; some upstream examples do not parse |

## Steps

### 1. Read the existing model

Inspect `domain/` before adding anything. Which concepts already exist? Does this requirement extend
a context or introduce one? Import rather than redeclare — a duplicated concept under a second name
is the failure this step exists to catch.

### 2. Ontological decomposition

Name what *exists* in the domain for this requirement, then classify each concept with the full UFO
taxonomy using `references/ufo-stereotype-selection.md`. Work the decision tree explicitly:

- **Substantial** — supplies its own identity → `kind`; a collection of like things → `collective`;
  an amount of matter → `quantity`
- **Moment** (existentially dependent) — truth-maker of a material relation → `relator`; measurable
  characteristic → `quality`; intrinsic non-measurable → `mode`
- **Perdurant** — bounded occurrence → `event`; state of affairs → `situation`; ongoing → `process`
- **Specialisation** — rigid → `subkind`; anti-rigid intrinsic → `phase`; anti-rigid relational →
  `role`; event-grounded → `historicalRole`
- **Across kinds** — rigid → `category`; semi-rigid → `mixin`; anti-rigid intrinsic → `phaseMixin`;
  anti-rigid relational → `roleMixin`
- **Second-order** — open classification → `type`; closed → `powertype`

Two checks that catch most errors early:

- **Hunt for relators.** Every material relation has a truth-maker. If a relationship carries
  attributes, a lifecycle, or rules, it is a relator — name it and make it first-class.
- **Challenge every noun that looks like a role.** `Customer`, `Employee`, `Supplier`, `Owner` are
  almost never kinds.

Write it as Tonto in `domain/`, using `references/tonto-language.md` for syntax and
`references/example.tonto` as a known-good template. Set cardinalities deliberately
(`references/cardinality.md`) — they become invariants in step 5.

Two syntax traps worth knowing before the first parse error: relation end names are **bare
identifiers** (parentheses are only for meta-attributes), and `value`, `type`, `description`,
`role`, `label`, and `class` are **reserved** and cannot be attribute names — see the reserved
keywords table in `references/tonto-language.md` §2.1.

### 3. Validate

```bash
tonto-cli validate .
```

**Fix every error and warning before proceeding.** Deriving a design from an unvalidated model
propagates ontological errors straight into the code, which defeats the point.

Then run the terminology audit (`references/terminology-audit.md`) — validation checks structure,
not whether the names match the ontology. `class` with no stereotype is a warning to resolve, not
to suppress.

**Tooling reality (verified on Windows, Node 24.19).** On **0.4.13**, `validate`, `generate`, and
`transform` all work — this is the main reason to pin at or above it, since on 0.4.12 the latter two
fail on path resolution even with valid absolute paths.

Still broken on 0.4.13: **`plantuml`** fails with "no package declarations found" on a directory
`validate` parses cleanly, for every argument form tried. Get diagrams from the VS Code extension
instead. **`init`** is interactive and cannot run unattended — scaffold `tonto.json` by hand (see
Artefacts).

Output locations are unobvious: `generate` writes `<outFolder>/<projectName>.json`, and `transform`
writes `<outFolder>/<projectName>` with **no extension** (it is Turtle). Full table and re-test steps
in `references/ATTRIBUTION.md`; do not re-diagnose these.

### 4. Derive the design

Apply `references/ufo-to-ddd-derivation.md`. In order:

1. **Aggregate roots** — from the ultimate sortals. Relators are usually roots (§3); this is the
   highest-value rule in the derivation.
2. **Boundaries** — from the connectors. Composition `<o>--` is inside the aggregate, aggregation
   `<>--` and association `--` are by-ID references across aggregates (§8).
3. **Entities vs. value objects** — from identity. `quantity`, `quality`, and most `mode`s are value
   objects (§2).
4. **Type structure** — `subkind` may be a subclass, `phase` must not be (§4). Gensets give sealed
   hierarchies and state machines (§8).
5. **Roles** — resolve each to nothing, an interface, a projection, or a cross-context entity (§4).
   Never a duplicate identity.
6. **Events and policies** — `event` → domain event, `situation` + `@triggers` → policy,
   `process` → saga (§6).

Then check the design against the anti-pattern table (§11). A hit there means the ontology or the
derivation is wrong — go back, do not work around it.

### 5. Define module contracts

For each affected module:

- **Responsibilities** — what it owns and is accountable for, and explicitly what it does *not* do
- **Public API** — types and operations other modules may call; everything else internal
- **Dependencies** — which modules it may talk to, and the direction
- **Invariants** — the rules it guarantees, derived from cardinalities and mediation constraints (§8)

Map contexts onto the ontology as projections, not copies: one core reference ontology, one
projection per bounded context (§9). This is what makes `Person`-in-core / `Customer`-in-Sales
legitimate rather than duplication.

### 6. Trace back to Gherkin

The model *entails* scenario obligations — phase transitions, mediation bounds, lifecycle events,
non-empty collections, historical-role survival. Read them off the traceback table (§10) and hand
them to `/mainline-requirement-workflow` as required scenarios. A missing scenario here is a coverage gap the
model can prove exists.

## Artefacts

```
domain/
├── tonto.json              # manifest
├── core.<concept>.tonto    # reference ontology — where identity and dependence are settled
├── <context>.tonto         # one projection per bounded context, importing from core
└── generated/              # tonto-cli output — not version controlled
```

A package is declared by a file's `package` statement, **not** by its directory — folders are
organisation only.

## Output

1. A validated `.tonto` model committed under `domain/`
2. A module spec per affected module — responsibilities, public API, dependencies, invariants
3. The list of Gherkin obligations the model entails

(1) and (2) feed the architecture rules in `/mainline-quality-gate` (dimension 2); (2) and (3) feed the build
steps in `/mainline-development-workflow`.

## Scope discipline

Model the concepts this requirement touches, and the ones needed to place them correctly. Do not
model the whole domain speculatively. Reach for `mixin`, `phaseMixin`, and multi-level constructs
when the domain actually presents that structure — not to demonstrate that you can.
