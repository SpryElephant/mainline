---
name: mainline-refactoring
description: Large, mechanical, behavior-preserving refactors driven by an automated rewrite engine — package/namespace moves and renames, type/API migrations, structural reshaping across many files. Use whenever a change is broad and rote enough that hand-editing every reference would be error-prone, instead of for a new requirement (that goes through /mainline-development-workflow).
---

# Refactoring (automated rewrite engine)

For **structural, behavior-preserving** change at scale: moving/renaming packages or namespaces,
migrating a type or API, reshaping module boundaries. An automated rewrite engine rewrites references
deterministically and relocates files, so the diff is reviewable and the run is repeatable.

This is **not** `/mainline-development-workflow`. A refactor introduces no new requirement and no behavior change — it
has no Gherkin scenario. It lands as its own commit, green through `/mainline-quality-gate`, *before* any feature
work that depends on the new shape.

> **Reference tool.** The canonical engine is **OpenRewrite** (JVM), used via the Gradle/Maven rewrite
> plugin and declarative `rewrite.yml` recipes. Pick the equivalent for the target language:
>
> | Reference (JVM) | Python | C# / .NET | TypeScript / JS | Polyglot |
> |---|---|---|---|---|
> | OpenRewrite | `rope`, `libcst`, Bowler | Roslyn analyzers/codefixes, `dotnet format` | `ts-morph`, `jscodeshift` | `comby`, `ast-grep` |
>
> Whatever the engine, the two properties that make this skill work are the same: it rewrites
> *references* (not just text) and the run is *deterministic and repeatable*.

## What the engine does — and does not — touch

Reference-rewriting recipes (rename package, change type, change method) rewrite **language-level type
references** (imports, fully-qualified names) and move source files to the matching directory. They do
**not** touch:

- **String literals** that name packages/types — most importantly architecture rules
  (`resideInAPackage("…")` and equivalents) and any string math on names (`substring`, `split`,
  `startsWith`).
- **Build config** — code-gen package names, generated-source globs, source-set / project paths.
- **Templates & resources** — templating engines referencing FQNs, `.properties`, YAML, descriptors.
- **Docs & memory** — design docs, decision records, memory entries.

Every refactor has this **manual tail**. Finding it is the real work; the automated rewrite is the easy
part.

## Process

1. **Scope & map.** Enumerate every package/type that moves or renames as an explicit `old → new`
   table. Confirm structure-preserving renames keep existing distinctions intact.
2. **Find the manual tail.** `grep` the old names across **non-code** surfaces — architecture/test
   infra, build config, templates, resources, docs, memory. List each hit; these are the hand-edits the
   engine will skip.
3. **Author the recipe.** Add the rewrite engine to the build temporarily and write the transformation
   declaratively. Reference (OpenRewrite) example:
   ```yaml
   type: specs.openrewrite.org/v1beta/recipe
   name: com.example.<RefactorName>
   recipeList:
     - org.openrewrite.java.ChangePackage:
         oldPackageName: com.example.old
         newPackageName: com.example.new
         recursive: true
   ```
   Common moves across engines: rename a package/namespace tree (recursively), change one type's FQN,
   rename a method.
4. **Run it.** Execute the rewrite. Review the diff — files relocate, imports/FQNs update.
5. **Fix the manual tail** from step 2 by hand.
6. **Verify with `/mainline-quality-gate`.** Run the project's gate command. The architecture tests are the
   **safety net**: a missed package string fails a rule and names the exact stale reference. Loop until
   green.
7. **Clean up.** Remove the temporary engine config. Commit as a standalone, behavior-preserving
   refactor. Update affected design docs and memory.

## Invariants

- **Zero behavior change.** No scenario changes; the gate must be green before *and* after.
- **The gate is the proof.** Don't declare done until the gate passes — the architecture tests catch
  the string-literal references the rewrite can't.
- **Separate commit.** A refactor is its own commit/PR, never mixed with feature work.
