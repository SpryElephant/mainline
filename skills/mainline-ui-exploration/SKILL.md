---
name: mainline-ui-exploration
description: Produce several genuinely different interface directions for a product and build a clickable prototype of each, so a direction is chosen by comparing real artefacts rather than by arguing about adjectives. Use when starting a product's visual design, reworking an existing one, or whenever "what should this look like" needs more than one answer before committing. Not for building the real UI — that is /mainline-development-workflow.
---

# UI exploration

Design is a **divergence problem before it is a craft problem.** Picking a direction by discussing
adjectives produces the direction everybody already had in mind. Picking it by walking through
several working prototypes of the same flow produces an informed choice, and the runners-up donate
their best ideas to the winner.

This skill produces **N clickable prototypes of one identical flow**, each a different design world.

> **Not the real UI.** Prototypes are throwaway artefacts for deciding. They carry no
> requirement, no test, and no gate. What crosses into production is the design *decisions*,
> re-implemented through `/mainline-development-workflow` — never the prototype code.

This skill answers *which direction*. It does not answer *what should be built* — that is
`/mainline-product-discovery`, which treats a UI exploration as one prototype medium among several and turns
what the sessions taught into Gherkin. Run discovery around this skill when the requirement itself
is still open.

## Why output goes generic, and the one structural fix

A model asked to pick the taste, explore the options, and write the code in a single pass returns
the weighted average of every interface it has seen. That average is recognisable: Inter, a
purple-to-blue gradient, a centred hero, three icon cards, everything `rounded-lg`.

**Generic is a workflow problem, not a model problem.** The fix is to separate the stages, and to
constrain each one. Moderate constraints beat open briefs — creativity follows a U-shaped curve
against constraint, and "make it beautiful" sits at the flat, useless end.

## Process

### 1. Establish the brand foundation and the flow — before any visuals

Two artefacts, in words:

- **What it should and should not feel like.** Take it from the product's own documents if they
  exist. The *should nots* do more work than the shoulds: they are what stops every direction
  drifting to the same safe middle.
- **The flow.** The exact screen sequence every prototype will implement, start to finish. Same
  flow in all of them — **this is what makes them comparable.** Different flows produce a
  beauty contest instead of a decision.

### 2. Write the ban list

Name the specific defaults that are forbidden in *this* project. Start here and add the ones you
personally reach for:

Inter / Roboto / Space Grotesk as the default face · purple-to-blue gradients · centred hero with
one call to action · three-card icon rows · uniform `rounded-lg` · 0.1-opacity shadows on
everything · glassmorphism as the whole idea · emoji as section markers · warm cream `#F4F1EA`
with a serif and a terracotta accent · near-black with a single acid-green pop · gradient text on
numbers · bounce/elastic easing.

A direction that lands on one of these has not been designed, it has been defaulted to. If a
direction genuinely calls for a banned element, it must earn it in the thesis.

### 3. Plan every direction in text first

**No code in this stage.** For each direction write a short, specific block:

| Field | What it must contain |
|---|---|
| **Thesis** | One sentence: what this direction believes about the product |
| **Borrowed world** | A real thing outside software it draws from — an instrument, a printed object, a place. This is where distinctiveness comes from; "modern and clean" is not a world |
| **Palette** | 4–6 named hex values, with the neutral's hue bias stated |
| **Type** | Two or three roles with actual families and why they belong to this world |
| **Navigation model** | How a person moves through it — the structural choice, not the styling |
| **Motion** | What moves, on what trigger, with what easing character |
| **Risk** | The one thing that could make this direction fail |

Then **review the plan against the ban list and against the other directions.** Rewrite anything
that reads like the default you would produce for any product in this category. Note what changed.

### 4. Force structural divergence

Colour is not a direction. Two prototypes that differ only in palette are one prototype.

Every direction must differ from every other on **at least two** of these axes:

- **Navigation model** — tab bar / single canvas / card stack / one question at a time / map-first / spatial
- **Information density** — one thing per screen versus a dense board
- **Primary input** — tap, swipe, drag, dial, draw, type
- **Metaphor** — what real object or place it behaves like
- **Type personality** — display-led, mono-led, text-led, image-led
- **Motion character** — still, mechanical, fluid, physical

Write the divergence check down. If two directions collide, replace one — do not nudge its hue.

### 5. Extract a design system, then build from it

Per direction, in this order:

1. Write the tokens — colour, type scale, spacing, radius, motion — as CSS custom properties.
2. Build every screen by referencing those tokens only. Inventing a value mid-build is the moment
   a direction stops being a system and becomes a pile of screens.

Give specs, not adjectives. "Increase contrast, 20px type, more padding" beats "make it nicer."

### 6. Make it genuinely clickable

A prototype nobody can walk through is a mood board. Each one must:

- Run the **whole flow**, first screen to last, in a device frame.
- Have every advancing control actually work. A dead primary button is a lie about the design.
- Show real content from the product's own domain. Lorem, "Card title", and placeholder greys
  hide exactly the problems a prototype exists to find.
- Be **one self-contained file** with no external requests, so it opens anywhere and still works
  in a year.
- State its own name and thesis somewhere in the frame, so a person comparing seven of them knows
  which world they are standing in.

### 7. Compare, then decide

Build an index that opens all of them from one place, and walk the same flow in each. Record for
each direction: what it does better than the rest, what it costs to build for real, and which of
its ideas should be transplanted into whichever wins.

**The runners-up are not waste.** They are where the winner's best details come from.

## Delivering

Prototypes live under `prototypes/<name>/` — the same quarantine `/mainline-product-discovery` uses: outside
the build, invisible to CI and coverage, and no production module may import them. They are
exploration, not product, and holding them to production thresholds would only make them cautious.

Publish them somewhere clickable and hand over the links with the comparison, not just the files.

## Failure modes

- **Seven skins, one idea.** The commonest failure. Cured by step 4, not by trying harder.
- **The flow drifts between directions.** Now nothing is comparable. Fix the flow in step 1 and do
  not touch it.
- **Prototype code leaks into production.** It was written to be thrown away and has no tests.
  Port the decisions; rewrite the code.
- **Designing the tenth screen.** Prototype the flow that decides the direction. Settings screens
  never decided anything.
- **Falling for the most polished one.** Polish is cheap to add later; a wrong navigation model is
  not. Judge the structure.
