# Mainline commands

The four handoff commands. Copy this folder into a project's `.claude/commands/`, alongside the
skills in `.claude/skills/`.

**Work moves between people by command, not by memory.** A handoff check that depends on someone
remembering to walk it is not a check. These are what make Mainline a line rather than a set of
habits.

| Command | Handoff | From → to |
|---|---|---|
| `/ready-for-dev` | Requirement → Design | Product → Developer |
| `/ready-for-review` | Verify → Review | Developer → Reviewer |
| `/ready-for-qa` | Review → QA | Reviewer → QA |
| `/ready-for-release` | QA → Release | QA → Release approver |

Two more commands live here and are not handoffs: `/wire-handoffs` writes the configuration below
(onboarding step 4), and `/mainline-help` (a skill, in `skills/`) tells a person where their work is
and which command to run.

Filing a finding is not a handoff — that is the `/mainline-file-finding` skill, usable from inside any session.

## The contract

Every one of them does the same five things, in this order, **stopping at the first failure**:

1. **Runs the handoff checks** for that station. The checklist lives in the station's skill; the
   command does not restate it, so there is one place to change a check.
2. **Blocks** if any check fails, naming the failing check and what would satisfy it. Nothing moves.
3. **Moves `Phase`** on the board.
4. **Assigns** the next owner.
5. **Notifies** them with everything attached, and **records the result on the ticket** as a comment.

A check that passes but leaves nobody holding the work has not finished. A handoff that moves work
without recording the result leaves the lead nothing to read.

**Never move a card past a failing check.** If a check is wrong, that is an `/mainline-improvement-loop` entry
and a `Platform` item — not a reason to pass the handoff anyway.

## Configuration

Two files in the target repo. `/wire-handoffs` writes both; `/wire-handoffs --check` verifies them.

- **`.github/mainline.json`** — the project's own parameters:

  ```json
  {
    "owner": "<login>",
    "repo": "<owner>/<repo>",
    "projectNumber": 0,
    "projectId": "PVT_...",
    "notify": "<shell command taking \"$MESSAGE\" and \"$URL\">",
    "assignees": { "design": "", "review": "", "qa": "", "release": "" }
  }
  ```

- **`.github/project-fields.json`** — the cached field and option IDs, the same cache `/mainline-file-finding`
  uses. Refresh it with `gh project field-list "$PN" --owner "$OWNER" --format json` when a field or
  option is added.

`notify` is deliberately a shell command rather than a named service. Slack, Teams, email or a
webhook all work; the line does not care which, and swapping it is one line. Where it is unset the
commands fall back to GitHub's own assignment notification and **say so** rather than pretending the
message landed.

`assignees` holds the default owner per station. A command uses it when the next owner is not passed
explicitly, and refuses to move the card when neither is available — unassigned work is how a station
silently fills up.
