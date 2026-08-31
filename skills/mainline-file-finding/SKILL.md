---
name: mainline-file-finding
description: File a finding as a tracked work item without leaving the session — classify it, write it against the right issue form, create it, set its board fields, assign it and notify the owner. Use the moment a bug, risk, missing requirement, change request or piece of platform debt turns up in work you are not stopping to fix.
---

# File a finding

**A finding you did not file is a finding you paid for and threw away.** You spent the effort to
learn it; the cost of capturing it is thirty seconds and the cost of losing it is discovering it
again, later, more expensively.

File it **now, from inside the session** — not at the end of the task, not in a note to yourself.
This is the harvesting rule that `/mainline-development-workflow`, `/mainline-product-discovery`, `/mainline-e2e-suite`,
`/mainline-review-station` and `/mainline-observability` all defer to.

## 0. Cache the board IDs — once per repo

Setting project fields needs the project's field IDs and per-option IDs. Fetching them on every call
is slow and trips GitHub's secondary rate limit. Resolve them once and commit the result to
`.github/project-fields.json`:

```bash
gh project field-list "$PN" --owner "$OWNER" --format json > .github/project-fields.json
```

Read `$PN`, `$PID`, `$OWNER` and `$REPO` from `.github/mainline.json` (see `commands/README.md`). Refresh the cache when a field or option is added — a stale ID fails loudly with
`Could not resolve to a node`, which is the good failure mode.

## 1. Classify it

The classification picks the `Work Type`, the label, and the form. Get this right and everything
downstream is automatic.

| The finding | `Work Type` | Label | Form |
|---|---|---|---|
| Something is broken against a stated requirement | `Bug` | `type:bug` | `bug.yml` |
| A threat to scope, schedule or quality that has not happened yet | `Risk` | `type:risk` | `risk.yml` |
| A rule nobody wrote down — the system does something no scenario covers | `Feature` | `type:feature` | `feature.yml` |
| A material change to the scope baseline, sequence, or a committed decision | `Change request` | `type:change-request` | `change-request.yml` |
| A question that must be answered before scope can be committed | `Spike` | `type:spike` | `spike.yml` |
| Structure worth improving, no behavior change | `Refactor` | `type:refactor` | blank |
| Work on the line itself — gate, stack, CI, alerting, a baseline burn-down | `Platform` | `type:platform` | blank |
| Housekeeping with no user-visible effect | `Chore` | `type:chore` | blank |

**A defect that cites no requirement is not a bug.** It is either a missing requirement — file it as
a Feature and assign it to Product — or a preference, which is not a finding at all.

**When two fit, file the cheaper one.** A `Spike` that turns out to be a `Change request` costs an
hour. A `Change request` that was only ever a question costs a governance cycle.

## 2. Write it

- **Title states the finding, not the symptom.** "Refunds over R$500 bypass the manager PIN", not
  "refund bug".
- **Body carries what the next person needs to not repeat your work** — where you found it, what you
  observed, the reproduction if you have one, and the requirement or scenario it violates.
- **Cite the source.** The `.feature` file and scenario, the session log line, the alert, the CI run,
  the PR. A finding with no provenance gets re-litigated instead of fixed.
- **Say what you did not do.** If you worked around it, say so — the workaround is the next person's
  first surprise.

Use the issue form where one exists; it enforces the fields the board needs. `risk.yml` in
particular requires the statement (*if cause, then event, leading to consequence*), the trigger, and
the response.

## 3. File it

Three calls: create the issue, add it to the project, set its fields.

```bash
url=$(gh issue create --repo "$REPO" --title "$TITLE" --body-file "$BODY" \
      --label "$LABEL" --assignee "$OWNER_LOGIN" | grep -oE 'https://github.com[^ ]+')

iid=$(gh project item-add "$PN" --owner "$OWNER" --url "$url" --format json \
      | python -c "import sys,json;print(json.load(sys.stdin)['id'])")

set_opt() { gh project item-edit --id "$iid" --project-id "$PID" \
            --field-id "$1" --single-select-option-id "$2" >/dev/null; }
```

`item-edit` takes `--text`, `--number` and `--date` for the non-select fields, and `--clear` to
unset one.

## 4. Set the fields

| Field | Value |
|---|---|
| `Phase` | **`Inbox`** — findings enter the line like any other work, and the lead triages them |
| `Work Type` | From step 1 |
| `Area` | The module it belongs to, if you know it. Leave unset rather than guess |
| `Size` · `Priority` | Your honest ROM and MoSCoW. The lead may overrule both at triage |
| `Probability` · `Impact` · `Exposure` | **Risks only** — `Exposure = P × I`, the register's sort key. Risks carry these *instead of* Size and Priority, and sit on the risk register rather than the `Phase` pipeline |
| `Target` | Only if something real fixes the date |

**Never set `Phase` past `Inbox`.** A finding that files itself straight into Build has skipped
triage, which is the one decision that is not yours to make from inside a session.

## 5. Assign and notify

- **Assign it.** An unassigned finding is a finding nobody owns. Default to the lead for triage; name
  a specific owner only when it is unambiguous — the module's owner, or Product for a missing
  requirement.
- **Notify** in the project channel, with the issue link. Where the transport is not wired yet,
  GitHub's own assignment notification is the fallback — but say so rather than assuming it landed.
- **Then go back to what you were doing.** Filing a finding is not an invitation to fix it.

## What not to file

- **Something you are fixing in this change.** Fix it; the commit is the record.
- **A preference with no requirement behind it.** Take it to the requirement, or drop it.
- **A duplicate.** Search first — `gh issue list --repo "$REPO" --search "<terms>"`. Comment on the
  existing one instead; a second ticket splits the evidence.
- **A finding you cannot state in one sentence.** If you cannot, you have not understood it yet, and
  the ticket will be noise. Spend two more minutes, then file.

## Failure modes

- **Filing at the end of the task.** By then the specifics are gone and the ticket says "investigate
  the refund thing." The value was in the detail you had at the moment you saw it.
- **The unassigned backlog.** Findings filed and never triaged read as diligence and function as a
  landfill. Assignment is what makes filing real.
- **Re-fetching the field IDs every time.** Slow, and it eventually trips the secondary rate limit
  mid-loop, leaving a half-configured item on the board.
- **Guessing `Area` or `Priority`.** A wrong field is worse than an empty one, because the lead
  trusts it.
- **Filing the workaround instead of the defect.** The ticket then closes when the workaround ships.

## Relationships

- **`/mainline-pmi-github-project`** — the board, the fields, the labels and the issue forms this skill writes
  into. Stand it up first.
- **`/mainline-development-workflow`** — "file what you find, now" at build time.
- **`/mainline-product-discovery`** — `ASSUM` observations become risks through this skill.
- **`/mainline-e2e-suite`** — a quarantined flaky test files a ticket the same day.
- **`/mainline-review-station`** — a repeated waiver files a `Platform` item against the rule.
- **`/mainline-observability`** — an alert or error group becomes a work item the same way.
- **`/mainline-improvement-loop`** — amendments to a check are `Platform` findings with an owner and a date.
