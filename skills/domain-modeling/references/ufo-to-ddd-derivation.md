# UFO → DDD derivation

How an OntoUML/UFO model becomes a design: aggregates, entities, value objects, events, module
boundaries, and invariants.

This file is **ours**, not upstream. The Tonto guidance covers building a correct ontology; it says
nothing about deriving code from one. See `ATTRIBUTION.md`.

Read this *after* the ontology is written and `tonto-cli validate` is clean. Deriving from an
unvalidated model propagates ontological errors into the design, which is the whole thing this
skill exists to prevent.

---

## 1. Why the derivation is mostly mechanical

DDD's tactical patterns are an *implementation* vocabulary — entity, value object, aggregate,
repository. UFO is an *ontological* one — what a thing is, whether it carries identity, whether it
can stop being what it is. The second determines the first far more tightly than DDD folklore
admits:

- **Identity** is the aggregate-root question, and `kind` (plus the other ultimate sortals) is
  exactly the construct that supplies identity.
- **Rigidity** is the inheritance question. Rigid types can be subclasses; anti-rigid types cannot,
  because an instance would have to change class.
- **Existential dependence** is the aggregate-boundary question, and composition vs. aggregation
  encodes it directly.

So most of what follows is a lookup, not a judgement call. Where genuine judgement remains, it is
called out as a **test** with a concrete discriminator.

---

## 2. Ultimate sortals → aggregate roots

Ultimate sortals supply a principle of identity. Every object instantiates exactly one of them, so
**each ultimate sortal is at most one aggregate root**, and its identity criterion is that
aggregate's ID.

| Stereotype | Derives to | Notes |
|---|---|---|
| `kind` | **Entity**, aggregate-root candidate | The identity attribute is the aggregate ID type |
| `collective` | **Entity** (root) or collection **VO** | See test below |
| `quantity` | **Value object** — a measure | Never an entity. No identity across split/merge |
| `quality` | **Value object** with an ordered value space | The quality space gives you comparison + range invariant |
| `mode` | **VO**, or entity *inside* the bearer's aggregate | See test below |
| `relator` | **Entity — usually the aggregate root** | The highest-value rule in this file. See §3 |
| `type` / `powertype` | **Type Object** — reference-data aggregate | See §7 |

**`collective` test.** Does the collective have invariants over its membership (a `Committee` needs
≥ 3 members) or its own identity independent of the members? Then it is an aggregate root and the
invariant lives on it. If it is merely "some `X`s" with no rules and no identity, it is a
collection value object. The `@memberOf` cardinality settles ownership: `[1]` on the whole side
means members are exclusively owned and live *inside* the aggregate; `[*]` means they are shared
and must be separate aggregates referenced by ID.

**`mode` test.** Does it have a lifecycle of its own — does it change state over time
independently of its bearer? A `MedicalCondition` is diagnosed, treated, resolved: entity inside
the patient's aggregate. A `Skill` at a level: value object. `extrinsicMode` depends on an external
individual, which means it is relator-adjacent — model it inside the relator's aggregate, not the
bearer's.

**`quantity` vs `quality`.** Both are value objects but they behave differently. A quantity
supports split and merge and those operations must conserve the total (`@subQuantityOf` is the
signal). A quality supports comparison and range-checking against its value space (`@value` gives
you the magnitude-plus-unit representation). Money is a quantity; temperature is a quality.

---

## 3. Relators are where aggregates actually live

A `relator` is the truth-maker of a material relation — the thing that makes it true that these
individuals are related. `Employment`, `Enrollment`, `Tenancy`, `Subscription`, `Marriage`.

**Default derivation: the relator is an Entity and it is the aggregate root.**

This resolves three chronic DDD problems at once:

1. **The homeless-behaviour problem.** Logic that "belongs to no entity" and gets dumped into a
   domain service almost always belongs to the relator. Before reaching for a domain service, check
   whether you failed to identify a relator.
2. **The attributed join table.** A many-to-many relation with attributes nobody owns
   (`start_date`, `terms`, `status` on a join row) is an unrecognised relator.
3. **Aggregate boundary discovery.** The relator's `@mediation` ends *are* the aggregate's
   by-ID references. You do not have to guess what is inside the boundary.

```tonto
relator Employment {
  startDate: date [1]
  endDate: date [0..1]
  @mediation [1..*] -- employee -- [1] Person
  @mediation [1..*] -- employer -- [1] Organization
}
```

(End names are bare identifiers. Parentheses are only for end *meta-attributes* —
`({ ordered } formerContracts)`. The parenthesised form appears in some upstream examples and does
not parse; `references/example.tonto` is a validated model to copy from.)

derives to: `Employment` aggregate root, holding `PersonId` and `OrganizationId` by reference (not
by containment — mediation is never composition), owning the invariant that both participants are
bound for the employment's lifetime, and owning every rule about terms, termination, and overlap.

**When a relator is a VO instead.** If it is immutable, has no lifecycle, and is never referenced
independently — a historical record of a completed relation — it collapses to a value object inside
whichever aggregate owns the history.

**Mediation cardinality reads as an invariant.** `[1]` on the relator side means the participation
is mandatory and immutable for the relator's lifetime: enforce at construction, forbid a setter.
`Marriage` with `@mediation [2..2] -- spouses -- [1] Person` is an exactly-two invariant checked in
the factory.

---

## 4. Sortals → inheritance, state, and projections

Sortals inherit identity from an ultimate sortal. What differs is **rigidity**, and rigidity decides
whether a type-level distinction is safe.

| Stereotype | Rigidity | Derives to |
|---|---|---|
| `subkind` | Rigid | **Subclass / sum-type variant.** Safe: an instance can never migrate |
| `phase` | Anti-rigid, intrinsic | **State field + transition invariants.** Never a subclass |
| `role` | Anti-rigid, relational | **Not a class.** Interface, projection, or nothing. See below |
| `historicalRole` | Anti-rigid, event-grounded | **Derived predicate over event history** |

### `phase` — the subclassing trap

A phase is anti-rigid: instances move between phases. Modelling `Active` / `Suspended` /
`Closed` as subclasses would require an object to change class, i.e. change identity. It derives
instead to a state field on the entity, plus the guards on each legal transition.

A `disjoint complete` genset over phases is a **state machine with an exhaustive state set** — that
is the enum, and exhaustiveness is checkable at compile time in most languages.

```tonto
disjoint complete genset AccountStatus
  where Active, Suspended, Closed specializes Account
```

→ `enum AccountStatus { Active, Suspended, Closed }` on the `Account` root, with transitions as
methods that enforce the guards, and **one Gherkin scenario per legal and illegal transition**
(§8).

### `role` — the duplicate-identity trap

A role is a `Person` *in a context*, and that context is a relator. `Customer` is not a thing; it is
a `Person` seen through a `Subscription`. Modelling it as its own persisted entity is the most
common domain-model defect there is, and it produces duplicate person records that then need
merging forever.

Roles derive to one of four things, in order of preference:

1. **Nothing.** The kind entity, reached through the relator. Usually correct.
2. **An interface / port** the entity satisfies in that context, when behaviour differs by role.
3. **A read model / projection type** — `CustomerView` — for query-side use. Not an aggregate, no
   repository, no writes.
4. **A local entity in a *different bounded context*.** This is the legitimate case: when
   `Customer` is the Sales context's own aggregate with its own identity, mapped to `Person` in
   the Identity context. That is context mapping, not duplication — the distinction is that the
   mapping is explicit and one direction is authoritative.

If you find yourself giving a role a repository, you have chosen (4) implicitly. Make it explicit or
back out.

### `historicalRole` — the revocation trap

`Alumnus`, `WarVeteran`, `FormerCustomer`. Grounded in a past event, so it **survives termination of
the relator**. It cannot be computed from current relations — that is the bug. It derives to a query
over the event history, or a flag set by the terminating event handler and never cleared.

The tell: if terminating an `Enrollment` also makes someone stop being an `Alumnus`, the derivation
is wrong.

---

## 5. Non-sortals → interfaces, never tables

Non-sortals classify across kinds and **do not supply identity**. Consequence: a non-sortal never
gets a repository and never gets a table of its own. If you are tempted, you have mistaken it for a
kind.

| Stereotype | Rigidity | Derives to |
|---|---|---|
| `category` | Rigid | **Interface / trait.** Shared behaviour, no shared storage |
| `mixin` | Semi-rigid | **Interface + a flag** on the kinds where it is accidental |
| `phaseMixin` | Anti-rigid, intrinsic | **Interface + state predicate** on each implementer |
| `roleMixin` | Anti-rigid, relational | **Polymorphic port.** See below |
| `historicalRoleMixin` | Anti-rigid, historical | `roleMixin` + `historicalRole` rules combined |

### `roleMixin` — the polymorphic-association rule

A `roleMixin` is a role played by instances of *different kinds*: `Customer` may be a `Person` or an
`Organization`; `Supplier` likewise. This is the most practically valuable non-sortal, because it
tells you something a plain foreign key cannot express.

When a relator mediates a `roleMixin`, **the aggregate reference must carry a kind discriminator**:
a `(kind, id)` pair, not a bare FK. Getting this from the ontology rather than from a production
incident is most of the value of doing this at all.

`mixin` being semi-rigid — essential for some instances, accidental for others — is exactly why it
cannot be pure inheritance: some implementers can stop satisfying it.

---

## 6. Perdurants → events, read models, and sagas

DDD has domain events and nothing else in this space. UFO distinguishes three things, and the
distinction maps cleanly onto patterns DDD teaches separately without connecting them.

| Stereotype | Derives to |
|---|---|
| `event` | **Domain event** — immutable, past tense, named for what happened |
| `situation` | **Read model / projection**, or a policy precondition. Never an aggregate |
| `process` | **Long-running process / saga**, or a scheduled projection |

**`event`.** `@participation` ends give the payload's aggregate ID references. `@creation` means the
event is the aggregate's lifecycle start — the factory result. `@termination` means it closes the
lifecycle. `@bringsAbout` and `@triggers` are choreography: they are the arrows in a saga, and each
one is a policy that must be somewhere in the design.

**`situation`.** A state of affairs holding at a moment — `SystemOverload`, `CreditLimitExceeded`.
This is the construct people most often wrongly promote to an entity. It is a *projection* over
current state. Its real value is with `@triggers`: `situation` triggers `event` is precisely a
**policy / process manager** — "when this holds, do that". Every such pair in the ontology is a
policy you owe an implementation.

**`process`.** If it has clean temporal boundaries it is an `event`, not a process. A genuine
process — ongoing, no natural end — is usually infrastructure or a saga, not an aggregate.

---

## 7. Multi-level → Type Object

`type` and `powertype` are second-order: their instances are themselves types.

- **`powertype`** contains *all* possible specialisations of its base. Closed. Derives to a sealed
  enumeration or a fixed registry.
- **`type`** is open — new instances can appear at runtime. Derives to a genuine **reference-data
  aggregate** with its own repository and its own lifecycle (`ProductType`, `JobPosition`,
  `AnimalSpecies`).
- **`instanceOf`** derives to a by-ID reference from the first-order entity to the type entity.
  `@instantiation` marks the link explicitly.

The practical payoff: `type` tells you the classification is *data*, so business users can add
categories without a deployment. `powertype` tells you it is *code*, so they cannot. Getting this
backwards is a recurring and expensive mistake.

---

## 8. Structure → aggregate boundaries

The connector encodes existential dependence, which is exactly the aggregate-boundary question.
This is the most mechanical rule in the file and it replaces a decision DDD normally leaves to
taste.

| Connector | Meaning | Boundary derivation |
|---|---|---|
| `<o>--` composition | Exclusive, existentially dependent | Part lives **inside** the aggregate. No separate repository. Deleted with the root |
| `<>--` aggregation | Shareable, independent | Part is a **separate aggregate**, referenced by ID |
| `--` association | Plain reference | Reference **by ID** across aggregates |

Relation stereotypes refine it:

| Stereotype | Derivation |
|---|---|
| `@mediation` | Relator ↔ participant. **By-ID reference**, never containment |
| `@characterization`, `@inherence` | Bearer owns the mode/quality — **inside** the aggregate |
| `@componentOf` | Functional part; inside with `<o>--`, referenced with `<>--` |
| `@memberOf`, `@subCollectionOf` | Collection membership; ownership per the `collective` test (§2) |
| `@subQuantityOf` | VO split/merge; the operation must conserve the total |
| `@constitution` | **Two aggregates, same matter, different identity** (clay / statue). A trap — do not collapse them |
| `@material` | **Derived. Do not store.** It is entailed by the relator — see below |
| `@derivation` | Marks the relator → material relation link. Confirms the above |
| `@formal`, `@comparative` | Pure function. No storage |
| `@externalDependence` | Cannot exist without the other → creation-time invariant |
| `@historicalDependence` | Event-history query; supports `historicalRole` |
| `@manifestation` | A disposition realised in an event — the `mode` is a behaviour trigger |
| `@instantiation` | Type Object link (§7) |
| `@value` | The quality's magnitude-and-unit representation |

**The `@material` rule deserves emphasis.** A material relation is *made true by* its relator. If
you both store the relator and store the material relation, you have two sources of truth for the
same fact, and they will drift. `Person worksFor Organization` is not a table; it is a query over
`Employment`.

### Cardinality → invariants

| Cardinality | Invariant |
|---|---|
| `[1]` | Mandatory. Non-null, established at construction |
| `[1..*]` | Non-empty collection, enforced by the root |
| `[0..1]` | Optional type |
| `[n..m]` | Explicit range check in the factory |

**Mandatory on both ends** (`[1]`–`[1]`, `[1..*]`–`[1]`) means neither side can exist without the
other, so they must be created in one transaction — a strong signal they belong to the **same
aggregate**. See `cardinality.md` for reading direction, which is easy to get backwards.

### Genset → type structure

| Genset | Derives to |
|---|---|
| `disjoint complete` over `subkind`s | **Sealed hierarchy / sum type**, exhaustive matching |
| `disjoint incomplete` over `subkind`s | Open hierarchy, or enum with a fallback case |
| `disjoint complete` over `phase`s | **State machine** with an exhaustive state set |
| `overlapping` | **Not a hierarchy.** Independent flags or interfaces |

---

## 9. Bounded contexts and the single-ontology tension

DDD deliberately permits two contexts to hold contradictory models of the same word. UFO wants one
consistent account of what exists. These pull against each other and the conflict is real, not
terminological.

The resolution — which is roughly how NEMO frames core vs. application ontologies — is:

- **One core `.tonto` domain ontology** is the shared reference. It is where identity, rigidity, and
  existential dependence are settled. It is not implemented directly.
- **Each bounded context is a projection of it**: a subset, renamed in that context's ubiquitous
  language, with its own local aggregates.
- **A role in the core ontology is frequently an entity in a context projection** (§4, case 4). That
  is the legitimate mechanism by which `Person` in the core becomes `Customer` in Sales.

Practically: `domain/core.*.tonto` for the reference ontology, `domain/<context>.tonto` per context
projection importing from core. When two contexts disagree, they disagree in their projections, and
the core stays consistent.

Do not try to make one ontology be the implementation model everywhere. That is how ontology-driven
design gets a bad name.

---

## 10. Traceback to Gherkin

The ontology is not just a design input — it *generates spec obligations*. Each of these is a
scenario you owe `requirement-workflow`, and their absence is a gap in coverage that the model can
prove:

| Ontological construct | Scenario obligation |
|---|---|
| `disjoint complete` genset over `phase`s | One scenario per legal transition, **and one per illegal transition being rejected** |
| `relator` with `[1]` mediation | Cannot construct the relator without both participants |
| `relator` with `[n..m]` mediation | Boundary scenarios at `n-1`, `n`, `m`, `m+1` |
| `event` with `@creation` | Aggregate does not exist before, does after |
| `event` with `@termination` | Post-termination operations are rejected |
| `historicalRole` | Role **survives** termination of the relator that granted it |
| `[1..*]` on any end | Empty collection is rejected |
| `@externalDependence` | Creation fails when the depended-upon entity is absent |
| `situation` + `@triggers` | When the situation holds, the event fires; when it does not, it does not |
| `roleMixin` in a mediation | The relation works for **each** participating kind |

---

## 11. Anti-patterns the ontology catches

What a validated model tells you that you got wrong. Each of these is a defect the DDD vocabulary
alone cannot name:

| Symptom in the design | Ontological diagnosis |
|---|---|
| Duplicate person/party records needing perpetual merging | A `role` was modelled as an entity (§4) |
| Join table with attributes nobody owns | An unrecognised `relator` (§3) |
| Identity changes when an object transitions state | A `phase` was modelled as a `subkind` (§4) |
| A repository for an abstract type with no instances of its own | A `category` was given storage (§5) |
| Two sources of truth that drift apart | A `@material` relation was stored alongside its relator (§8) |
| Foreign key that points at two different tables | A `roleMixin` was modelled as a plain FK (§5) |
| Domain service holding all the interesting logic | A missing `relator` (§3) |
| Business users need a deploy to add a category | A `type` was modelled as a `powertype` (§7) |
| Historical status wrongly revoked | A `historicalRole` computed from current relations (§4) |
| Deleting one aggregate silently destroys shared data | Aggregation modelled as composition (§8) |

`terminology-audit.md` covers the complementary check — whether the *names* match the stereotypes.
