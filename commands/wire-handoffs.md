---
description: Onboarding step 4. Find this repository's board, cache its field IDs, look at who works here, and write .github/mainline.json so the four /ready-for-… commands can run.
argument-hint: [notes, or --check to verify an existing configuration]
allowed-tools: Bash(gh:*), Bash(git:*), Read, Write, Grep, Glob, AskUserQuestion
---

Wire the handoffs for this repository: produce `.github/mainline.json` and
`.github/project-fields.json`, verified, so that `/ready-for-dev`, `/ready-for-review`,
`/ready-for-qa` and `/ready-for-release` can run. This is onboarding step 4 in
`playbook/01-onboarding.md`; the contract for the two files is in `commands/README.md`.

Notes from the person, if any: $ARGUMENTS

**Read the repository and GitHub before asking anything.** Ask once, at the end of step 3, with
what you inferred as the recommended answer. Never write a file with a blank that a command will
later trip over.

## 1. The repository and its board

- `gh repo view --json nameWithOwner,owner,isInOrganization,defaultBranchRef` gives `repo` and
  `owner`.
- Find the project: `gh api graphql` on `repository(owner, name) { projectsV2(first: 20) { nodes
  { id number title closed } } }`. Keep open projects.
  - **One:** use it.
  - **Several:** include the choice in the question in step 3, recommending the one whose title
    matches the repository.
  - **None:** stop. The board is onboarding step 3: say so and point at
    `/mainline-pmi-github-project`.
- Confirm the board is a Mainline board: `gh project field-list "$PN" --owner "$OWNER" --format
  json` must contain `Phase` with `Inbox, Requirement, Design, Build, Verify, Review, QA, Release,
  Done` and `Work Type` with `Epic, Feature, Risk, Refactor, Spike, Bug, Chore, Platform`. If an
  option is missing, stop and name it; that is step 3 work, not yours.
- Write the field list to `.github/project-fields.json`. Refresh it even if it exists; a stale ID
  fails with `Could not resolve to a node`.

## 2. The people

- Collaborators with write access or above: `gh api "repos/$REPO/collaborators?affiliation=all"
  --paginate --jq '.[] | select(.permissions.push) | .login'`.
- Who actually works here: `gh api "repos/$REPO/contributors" --jq '.[:10][] | [.login,
  .contributions] | @tsv'` and `gh pr list --state merged --limit 30 --json author,reviews` for
  who has authored and who has reviewed recently.
- Whoever is running this command: `gh api user --jq .login`.

From that, propose a default owner per station:

| Station key | Who | Inference |
|---|---|---|
| `design` | the developer who receives at `/ready-for-dev` | the most frequent recent PR author |
| `review` | the reviewer who receives at `/ready-for-review` | the most frequent recent PR reviewer, and **never the same login as `design`** |
| `qa` | who receives at `/ready-for-qa` | a collaborator who has not been authoring PRs, if there is one; otherwise ask |
| `release` | the release approver at `/ready-for-release` | a repository admin; otherwise the person running this |

A default is a fallback for when a handoff is run without an explicit assignee. It must be a real
login with write access. If a station has no plausible candidate, leave that as a question; do not
invent one.

## 3. The notify command, then one question

Look for an existing channel before proposing one:

- A Slack or Teams webhook already used by CI (`grep -ri "hooks.slack.com\|webhook.office.com"
  .github/`), or a repository secret named like `SLACK_WEBHOOK_URL` (`gh secret list`).
- Anything in `CLAUDE.md` or the README that says where the team talks.

Offer, in order of preference:

1. A webhook already in use: `curl -sS -X POST -H 'Content-type: application/json' --data "{\"text\":\"$MESSAGE $URL\"}" "$SLACK_WEBHOOK_URL"`, with the secret read from the environment, never written into the file.
2. A new webhook the person supplies.
3. Unset. The commands then fall back to GitHub's assignment notification and say so on every
   handoff. Acceptable to start; write it down as a `Platform` item.

Now ask **one** `AskUserQuestion` covering: the project (only if there were several), the four
default assignees, and the notify command. Lead each with the inferred answer marked as
recommended, so confirming is one keystroke.

## 4. Write, verify, commit

- Write `.github/mainline.json` exactly in the shape given in `commands/README.md`: `owner`,
  `repo`, `projectNumber`, `projectId`, `notify`, `assignees` with the four station keys. No
  secrets in the file.
- Verify each piece the way the handoff commands will use it:
  - `gh project item-list "$PN" --owner "$OWNER" --limit 1 --format json` succeeds.
  - Every assignee resolves: `gh api "users/$LOGIN" --jq .login`, and is in the collaborator list.
  - Every field ID in `project-fields.json` for `Phase` and `Work Type` is present.
  - If `notify` is set, send one message: `Mainline handoffs wired for <repo>` with the repository
    URL, and confirm the exit status was zero. Say in the report that this message went out.
- Commit both files with the message `chore: wire the Mainline handoffs` on the current branch if it
  is not the default branch. On the default branch, leave them staged and say so; opening the PR is
  the person's call.

## 5. Report

State, in this order: the board (number, title), the four defaults, the notify transport, what was
verified and how, what was committed, and anything left open. If anything was left open, say which
handoff command will refuse to run until it is filled in.

**`--check`:** skip the questions. Read the existing files and run only the verification in step 4,
reporting each check as pass or fail. This is what to run when a handoff command says the
configuration is wrong.
