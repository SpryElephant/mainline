---
name: mainline-pmi-github-project
description: Reproducible setup for a PMI-style project-management system on GitHub Projects — a kanban board whose Phase column encodes the delivery workflow, a WBS of epics/features, milestones as release gates, a scored risk register, a charter + issue forms + change control, and trunk-based branching with a CI-enforced quality gate. Use to stand this structure up in a new repo, rebuild it, or extend it.
---

# PMI-style project management on GitHub Projects

A runbook to reproduce the full setup: **full-PMI-artifact rigor, kanban flow** (pull-based, no
sprints). It is opinionated and tailored for a **small/solo technical project** — it deliberately
skips the PMI areas that add no value at that scale.

Reference assets ship next to this file in `references/` — copy them into the target repo:
`references/gate.yml`, `references/charter-template.md`, `references/issue-templates/*`.

Day-to-day filing onto this board is `/mainline-file-finding`; this skill stands the board up.

## When to use
Setting up project management in a new repo, rebuilding this structure, or extending it (new field,
milestone, risk, epic). Not for day-to-day feature delivery — that is `/mainline-requirement-workflow` and `/mainline-development-workflow`.

## Philosophy — tailor, don't cargo-cult
Keep a thin slice of five knowledge areas: **Scope, Schedule, Quality, Risk, Integration/Change
control.** Skip **Cost, Procurement, Resource-leveling, EVM, critical-path Gantt, formal
Communications plans** — overhead with no payoff solo. The board is the visible control layer over an
existing quality discipline; it surfaces what exists (roadmap = scope baseline, ADR/design docs =
config management, the test gate = quality plan) and adds the four controls usually missing: a visible
WBS, a roadmap, a risk register, and progress tracking.

**The key idea:** the board's `Phase` field *is* your delivery process, so the board visualises and
enforces the methodology instead of a generic To-Do/Doing/Done.

**What is never tailored away:** the four handoffs and the station boundaries between them. What
scales down on a small team is PMI's ceremony, not the line's checks. A one-person project runs all
four `/ready-for-…` commands, and each one ends the sitting — see `playbook/00-overview.md`, "One
station, one sitting".

## Prerequisites
```bash
# gh authenticated as the right account, with these scopes:
#   project        — create/manage Projects v2
#   workflow       — push .github/workflows/* (else the push is rejected)
#   repo, admin:public_key (optional), read:org
gh auth status
gh auth refresh -h github.com -s project,workflow   # if missing
```

## Parameters (set once)
```bash
OWNER=<login>            # user or org that owns the Project
REPO=<owner>/<repo>      # the repository
PN=<project-number>      # filled in after step 1
PID=<project-node-id>    # filled in after step 1 (PVT_...)
DEFAULT_BRANCH=$(gh api "repos/$REPO" --jq .default_branch)
```

## Step 1 — Project + link
```bash
gh project create --owner "$OWNER" --title "<Name> — Development & Delivery"
# (or repurpose an empty auto-created one: gh project list --owner "$OWNER"; gh project edit N ...)
gh project link "$PN" --owner "$OWNER" --repo "$REPO"
PID=$(gh project view "$PN" --owner "$OWNER" --format json --jq .id)
```

## Step 2 — Custom fields (the PMI data model)
```bash
mk() { gh project field-create "$PN" --owner "$OWNER" --name "$1" --data-type "$2" \
       ${3:+--single-select-options "$3"} >/dev/null && echo "  + $1"; }
mk Phase       SINGLE_SELECT "Inbox,Requirement,Design,Build,Verify,Review,QA,Release,Done"  # = the line
mk "Work Type" SINGLE_SELECT "Epic,Feature,Risk,Refactor,Spike,Bug,Chore,Platform"  # NOT "Type" — reserved
mk Area        SINGLE_SELECT "<your modules, comma-separated>"
mk Size        SINGLE_SELECT "XS,S,M,L,XL"                                # ROM estimate
mk Priority    SINGLE_SELECT "Must,Should,Could,Wont"                     # MoSCoW
mk Probability SINGLE_SELECT "Low,Medium,High"                            # risk items
mk Impact      SINGLE_SELECT "Low,Medium,High"                            # risk items
mk Exposure    SINGLE_SELECT "Low,Medium,High,Critical"                   # = P x I, the sort key
mk Target      DATE ""                                                    # roadmap dates
mk "Design Doc" TEXT ""                                                   # link to the ADR/design doc
```
The built-in **Status** field stays for coarse automation; **Phase** is the real board. Native
**Parent issue** / **Sub-issues progress** fields give the WBS hierarchy for free.

> **`Type` is a reserved field name.** GitHub took it for the built-in issue-types feature, and
> `field-create` now fails with *"Name cannot have a reserved value"*. Use **`Work Type`**. Note
> that `gh project item-list --format json` lowercases only the first letter and keeps the space,
> so the key is `"work Type"` — not `workType`.

## Step 3 — Milestones (release gates) + labels
```bash
ms() { gh api "repos/$REPO/milestones" -f title="$1" -f description="$2" --jq '"  + "+.title'; }
ms "M1 — <gate>" "<contents>"      # e.g. cheap wins
ms "M2 — <gate>" "<contents>"
ms "Backlog — Demand-driven" "Built on real demand."

lb() { gh label create "$1" --repo "$REPO" --color "$2" --description "$3" --force >/dev/null; }
lb type:epic 6f42c1 "WBS level 1"; lb type:feature 8a63d2 "increment (a .feature)"
lb type:risk b60205 "risk-register item"; lb type:spike fbca04 "time-boxed investigation"
lb type:refactor 0e8a16 "behaviour-preserving"; lb type:change-request d93f0b "scope/baseline change"
lb type:bug d73a4a "defect against a requirement"; lb type:chore cfd3d7 "housekeeping"
lb type:platform 5319e7 "work on the line itself"
# area:* labels in one colour, one per module
for a in <module1> <module2> <module3>; do lb "area:$a" 1d76db ""; done
```

## Step 4 — Seed the WBS (epics) and risk register
Setting project fields needs the field IDs and per-option IDs. Fetch them once:
```bash
gh project field-list "$PN" --owner "$OWNER" --format json > fields.json
python - <<'PY'   # prints "Field = <id>" then "  <option> -> <id>"
import json
for f in json.load(open("fields.json"))["fields"]:
    if f.get("options"):
        print(f["name"],"=",f["id"])
        for o in f["options"]: print("   ",o["name"],"->",o["id"])
PY
```
Then create each item, add it to the project, and set its fields. Helper:
```bash
# fill in FIELD/OPTION IDs from the dump above
seed_epic() { # title | milestone | Size-opt | Prio-opt | Area-opt | body
  local url iid
  url=$(gh issue create --repo "$REPO" --title "Epic: $1" --label type:epic \
        --milestone "$2" --body "$6" | grep -oE 'https://github.com[^ ]+')
  iid=$(gh project item-add "$PN" --owner "$OWNER" --url "$url" --format json \
        | python -c "import sys,json;print(json.load(sys.stdin)['id'])")
  ed(){ gh project item-edit --id "$iid" --project-id "$PID" --field-id "$1" \
        --single-select-option-id "$2" >/dev/null; }
  ed $TYPE_FIELD $EPIC_OPT; ed $PHASE_FIELD $INBOX_OPT
  ed $SIZE_FIELD "$3"; ed $PRIO_FIELD "$4"; ed $AREA_FIELD "$5"
}
```
Risks are the same shape with `--label type:risk` and Probability/Impact/Exposure set instead of
Size/Priority. Score each `Exposure = P x I`; write **statement / trigger / response** in the body
(the `risk` issue form does this). Seed the real risks the project faces, not placeholders.

> **Throttle:** creating many issues fast can hit GitHub's secondary rate limit and fail one
> silently — verify the count afterwards (`gh issue list --repo "$REPO" --json number --jq length`)
> and retry any that dropped.

## Step 5 — Charter + issue forms
```bash
cp references/charter-template.md            "$REPO_DIR/docs/PROJECT_CHARTER.md"   # then fill it in
cp -r references/issue-templates/.           "$REPO_DIR/.github/ISSUE_TEMPLATE/"    # epic/feature/risk/bug/change-request/spike/config
```
Edit the charter placeholders and the `config.yml` links. **Issue forms only activate once on the
default branch** — they must be committed and pushed.

## Step 6 — Views (UI only — the API cannot create these)
On the board, add these saved views:
1. **Flow** — Board layout, group by **Phase**. The daily driver.
2. **Roadmap** — Roadmap layout, dates = **Target**, markers by **Milestone**.
3. **Backlog** — Table, filter `Type = Epic`, sort **Priority** then **Size**.
4. **Risk register** — Table (or Board grouped by **Exposure**), filter `Type = Risk`, sort Exposure ↓.
5. **By milestone** — group by **Milestone**.

## Step 7 — Automation (UI only: Project ••• → Workflows)
Enable and configure: **Auto-add to project** (filter `is:issue is:open` — the default `label:bug`
is wrong), **Auto-add sub-issues to project**, **Item closed → Status: Done**, **Pull request merged
→ Status: Done**, optional **Item reopened → Status: Todo**. Built-in workflows drive the native
**Status** field only, not **Phase**.

> **On Mainline, `Phase` is not pulled by hand.** Each of the four handoffs is a command that runs
> that station's checks, refuses to move the card if one fails, sets the next assignee, and notifies
> them. **This holds at every team size, including one person.** Hand-pulling `Phase` skips the
> checks, and skipped checks are what the line exists to prevent. With four people it is also where
> work goes to die: nobody is sure whose turn it is, and the checks are whatever each person
> remembered.

## Step 8 — Versioning & branching (trunk-based, CI-enforced)
```bash
# a) CI runs the gate on every PR (adapt references/gate.yml to your stack; needs the `workflow` scope to push)
cp references/gate.yml "$REPO_DIR/.github/workflows/gate.yml"

# b) repo merge policy: squash-only + auto-delete branches
gh api -X PATCH "repos/$REPO" -F allow_squash_merge=true -F allow_merge_commit=false \
  -F allow_rebase_merge=false -F delete_branch_on_merge=true -F allow_auto_merge=true

# c) branch protection: require a PR + the gate check (context = the CI job name, here "gate")
gh api -X PUT "repos/$REPO/branches/$DEFAULT_BRANCH/protection" --input - <<'JSON'
{ "required_status_checks": { "strict": true, "contexts": ["gate"] },
  "enforce_admins": false,
  "required_pull_request_reviews": { "required_approving_review_count": 1 },
  "restrictions": null, "required_linear_history": true,
  "allow_force_pushes": false, "allow_deletions": false }
JSON
```
- **Flow:** short-lived branch `<type>/<issue#>-<slug>` → PR (`Closes #N`) → squash-merge → delete.
- **SemVer** via annotated tags + GitHub Releases, cut at milestone completion. Define what "breaking"
  means (your public contracts). Stay **pre-1.0 (`0.x`)** until those stabilise.
  `git tag -a vX.Y.Z -m "…"; git push origin vX.Y.Z; gh release create vX.Y.Z --title … --notes …`
- **Conventional Commits** (`feat`/`fix`/`docs`/`chore`/`refactor`/`ci`; `feat!`/`BREAKING CHANGE`)
  pair with SemVer (feat→minor, fix→patch, breaking→major) and enable later changelog automation.

## Gotchas (learned the hard way)
- **`gh` cannot create views or configure built-in automation workflows** — those are UI-only (steps 6–7).
- **`gh project item-list --format json` keys are lowercased** field names (`phase`, not `Phase`).
  Only the first letter is lowercased and spaces survive, so `Work Type` becomes `"work Type"`.
  Checking the wrong key reports every item as unset while the board is in fact correct.
- **`field-create` fails silently under `>/dev/null 2>&1`.** Creating ten fields in a loop trips
  GitHub's secondary rate limit part-way through. Verify with `field-list` and retry, rather than
  trusting the loop finished.
- **Pushing any `.github/workflows/*` file needs the token's `workflow` scope** or the push is rejected.
- **Deleting a remote branch may be blocked by the Claude Code safety classifier** — ask the user to run it.
- **First CI run flushes out local↔clean-env drift:** the exec bit on `gradlew`/wrapper scripts (Windows
  ignores it → `Permission denied` on Linux; fix with `git update-index --chmod=+x <script>`), a Node/JDK
  version the CI image lacks (pin via `.nvmrc` / `setup-*` and a single source of truth), and lint/coverage
  rules that only bite on a clean `npm ci`. Expect to iterate the gate green on the very first PR — that is
  the gate doing its job, and branch protection correctly refuses to merge until it is green.

## Field & option model (reference)
| Field | Type | Options | PMI area |
|---|---|---|---|
| Phase | select | Inbox, Requirement, Design, Build, Verify, Review, QA, Release, Done | Integration/process |
| Work Type | select | Epic, Feature, Risk, Refactor, Spike, Bug, Chore, Platform | Scope |
| Area | select | your modules | Scope |
| Size | select | XS, S, M, L, XL | Schedule (estimate) |
| Priority | select | Must, Should, Could, Wont | Scope |
| Probability / Impact | select | Low, Medium, High | Risk |
| Exposure | select | Low, Medium, High, Critical | Risk (P×I) |
| Target | date | — | Schedule |
| Design Doc | text | — | Config management |
