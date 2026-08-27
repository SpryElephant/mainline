# Attribution

Most of the reference material here is vendored from the **Tonto** project by
[NEMO/UFES](https://nemo.inf.ufes.br/), the research group behind OntoUML and the Unified
Foundational Ontology (UFO).

- **Upstream:** https://github.com/nemo-ufes/Tonto
- **License:** MIT
- **Vendored at:** tonto-cli v0.4.12, 2026-08-03 (content verified unchanged at v0.4.13)
- **Pin the tooling at v0.4.13 or later** — see the tooling table below

## There is an official Tonto Claude Skill

`tonto-cli add-skill <dir> --target claude` emits a complete `tonto-ontology` skill —
`SKILL.md` plus `references/{extending,terminology,summarization,documentation}.md`. It is bundled
inside the npm package, not committed to the git tree, so it does not turn up by browsing the repo.
Targets are `codex`, `claude`, `google`, `all`.

**We do not install it, and we vendor from it rather than from the repo's `.mdc` files.** Two
reasons:

1. **It is a different skill.** Its routing covers four ontology-internal tasks — extend, audit
   terminology, summarise, document. There is no design derivation: nothing about aggregates,
   module boundaries, or public API. That bridge is `ufo-to-ddd-derivation.md`, and it is ours.
   Its workflow rule — "plan and present to the user for confirmation before executing any
   significant change" — is right for interactive ontology editing and would stall
   `/mainline-development-workflow`.
2. **Installing it alongside `/mainline-domain-modeling` gives two skills that both fire on `.tonto` work**,
   with different processes. Vendoring its reference material keeps one entry point.

Its content, however, is **better maintained than the repo's `examples/Guidances/*.mdc`** — see
below. So it is the source of record.

## Vendored files

| Here | Source |
|---|---|
| `ufo-stereotype-selection.md` | `add-skill` → `references/extending.md` |
| `terminology-audit.md` | `add-skill` → `references/terminology.md` |
| `tonto-language.md` | repo `examples/Guidances/.cursor/rules/tonto-guidance.mdc` |
| `cardinality.md` | repo `examples/Guidances/.cursor/rules/tonto-cardinality-guidance.mdc` |

The first two come from `add-skill` because **the skill-bundled versions are corrected**. The
`.mdc` originals contain a mediation example with parenthesised relation end names
(`@mediation [1] -- (employeeEnd) -- [1] Employee`) that **does not parse** — verified against
`tonto-cli validate`. The `add-skill` versions have fixed it. Parentheses are only for end
meta-attributes (`({ ordered } formerContracts)`); a plain end name is a bare identifier.

The last two are vendored from the repo because they have no `add-skill` equivalent: the official
SKILL.md carries a condensed syntax section instead of the 1884-line grammar reference, and ships
no cardinality guide at all.

## Changes from upstream

**`tonto-language.md`, `cardinality.md`** — the Cursor `alwaysApply: true` YAML frontmatter is
stripped. It is Cursor rule-loading semantics with no meaning in a skill.

**`tonto-language.md`** — a **Reserved keywords** subsection is inserted into §2.1, marked with
`BEGIN/END spry-forge insertion` comments. The grammar reference alludes to reserved keywords in a
single line but never lists them; the official SKILL.md has the full list and pitfall table. The
content is upstream's, relocated. `value`, `type`, `description`, and `role` are all reserved and
all are names a domain model reaches for constantly.

**`cardinality.md`** — two headings de-contextualised. Upstream was written against the reader's own
in-progress project ("Examples from Your Healthcare Domain", "From Your Main Package"); those
referents do not exist here. The examples are unchanged.

**`ufo-stereotype-selection.md`, `terminology-audit.md`** — verbatim.

## Not vendored

- `examples/Guidances/.cursor/rules/tonto_llm_guidance.mdc` — upstream's task router, superseded by
  `../SKILL.md`.
- `add-skill` → `summarization.md`, `documentation.md` — explaining and documenting an existing
  ontology for a newcomer. Useful, but a different job from designing a change.

## What is ours

- **`ufo-to-ddd-derivation.md`** — the ontology → design bridge. Nothing upstream addresses deriving
  code from an ontology.
- **`example.tonto`** — a validated reference model exercising the constructs the derivation leans
  on, each annotated with the derivation section that consumes it. It passes `tonto-cli validate`
  at both v0.4.12 and v0.4.13. It exists because several upstream examples do not parse; copy syntax
  from here.

## Verified tooling behaviour (Windows, Node 24.19)

Established by running the CLI, not by reading docs. **Pin at 0.4.13 or later** — 0.4.12 has
materially more broken.

| Command | 0.4.12 | 0.4.13 |
|---|---|---|
| `validate <dir>` | Works | Works |
| `add-skill <dir> --target claude` | Works | Works |
| `generate <dir>` | **Fails** — "Neither file or directory provided", even with a valid absolute path | **Works** → `<outFolder>/<projectName>.json` |
| `transform <dir>` | **Fails** — `ENOENT`/`EISDIR` on `<outFolder>/<projectName>` | **Works** → `<outFolder>/<projectName>`, **no extension** (Turtle) |
| `plantuml <dir>` | **Fails** | **Still fails** — "no package declarations found" on a directory `validate` parses cleanly. Tried `.`, absolute, `src`, and `-d`; all fail |
| `generateSingle <file>` | **Fails** on an existing file | Not re-tested |
| `init` | **Interactive only** — cannot run unattended | Interactive only |
| `--version` | **Reports `0.4.0` regardless of installed version.** Useless as a version check — use `npm ls -g tonto-cli` | Reports correctly |

Node requirement is the package's declared `engines: { node: ">= 20" }`. Upstream's `AGENTS.md` says
Node 24+, but that is for building the Tonto monorepo, not for running the CLI.

`npm install -g` emits warnings about blocked `keytar` and `@vscode/vsce-sign` postinstall scripts.
Harmless — VS Code extension dependencies the CLI does not use. The install succeeds.

`../SKILL.md` depends only on `validate`. The rest are recorded as known-broken rather than omitted,
so nobody re-diagnoses them. Re-test on upgrade; these look like Windows path-handling bugs.

## Version note

The vendored reference files are **byte-identical between 0.4.12 and 0.4.13** (`extending.md`,
`terminology.md` both verified). The version pin here tracks the *tooling*, not the content.

## Updating

```bash
npx tonto-cli@latest add-skill /tmp/tonto-refresh --target claude
```

Then diff `/tmp/tonto-refresh/.claude/skills/tonto-ontology/references/` against
`ufo-stereotype-selection.md` and `terminology-audit.md`. For `tonto-language.md` and
`cardinality.md`, fetch the `.mdc` files directly:

```bash
gh api repos/nemo-ufes/Tonto/contents/examples/Guidances/.cursor/rules/<file>.mdc \
  -H "Accept: application/vnd.github.raw"
```

Re-apply the changes listed above, re-run `tonto-cli validate` against `example.tonto`, re-check the
tooling table, and bump the version and date here and in each file's header comment. Check
`ufo-to-ddd-derivation.md` for stereotypes or relation stereotypes added upstream that the derivation
does not yet cover.
