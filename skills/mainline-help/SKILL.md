---
name: mainline-help
description: The front desk for a project on Mainline. Tells a person where their work is, what to do next and which command or skill does it, and answers "how do I" questions from the playbook. Use when someone asks what to do now, what is next, where a card is, how a handoff works, which command to run, how to file a bug or a risk, what a term means, or anything else about working the line. Also the first thing to run on a project you have not worked on before.
allowed-tools: Bash(gh:*), Read, Grep, Glob
---

# Mainline help

The person asking has a question, not a request for the playbook. Answer the question, name the
one thing to run or read next, and stop. Everything you say comes from the playbook, the skills
and the board. Never invent a rule, and never move a card yourself: point at the command that does.

## 0. Is this project on Mainline?

Check for `.github/mainline.json` and `.claude/commands/ready-for-dev.md`.

- **Both present:** carry on.
- **Neither present:** say the project is not on Mainline yet, point at `playbook/01-onboarding.md`
  (eight steps, each with an acceptance test), and offer to start with step 1. Stop.
- **Commands present, config missing:** onboarding stopped before step 4. Say so and point at
  `/wire-handoffs`. Stop.

## 1. No argument: "what do I do now?"

1. Find out who is asking: `gh api user --jq .login`.
2. Read `.github/mainline.json` for the owner and project number. List the board:
   `gh project item-list "$PN" --owner "$OWNER" --format json --limit 200`. Keep the items whose
   `assignees` include the login and whose `phase` is not `Done`.
3. For each card, in one short block: the issue number and title; the station it is at (`phase`);
   what that station's job is, in one line from the table below; the skill to work it with; and the
   command that moves it on, or who moves it if there is no command at that station.
4. If they hold nothing: say so, point at the Flow view on the board, and, if they are the Lead,
   remind them the Lead's job is to watch the flow (`playbook/03-roles.md`, Lead).

Lead with the card that has been sitting longest.

## 2. An issue number: "where is #123 and what does it need?"

`gh issue view "$N" --json number,title,assignees,labels,url` plus the board item for it. Report:
the station, who holds it, and what must be true before it moves. The checks are in the station's
skill (table below); read the checklist there and say which lines are already satisfied and which
are not, with evidence from the issue, the PR and CI. Then name the command.

## 3. A question: "how do I ...?"

Find the answer in the playbook or the skill, quote the sentence or the checklist line, and name
the file it came from. Then say the one thing to run or read next.

If the playbook does not answer the question, say that plainly. Then file the gap with
`/mainline-file-finding` as a `Platform` item titled "Playbook gap: <the question>". A question the
playbook could not answer is a defect in the playbook.

For a word, use `playbook/05-glossary.md`. For a role, `playbook/03-roles.md`.

## The line at a glance

| Station | Owner | Work it with | What moves it on |
|---|---|---|---|
| Inbox | Lead | `/mainline-requirement-workflow` §1 (triage) | The Lead sets the fields and moves it to Requirement or Discovery |
| Discovery | Product | `/mainline-product-discovery`, `/mainline-ui-exploration` | The discovery record; then Requirement |
| Requirement | Product | `/mainline-requirement-workflow` | `/ready-for-dev` |
| Design | Developer | `/mainline-domain-modeling` | The same developer continues into Build |
| Build | Developer | `/mainline-development-workflow`, `/mainline-local-stack` | Gate green |
| Verify | Developer | `/mainline-quality-gate` (run the gate until green) | `/ready-for-review` |
| Review | Reviewer | `/mainline-review-station`, `/mainline-security-gate` | `/ready-for-qa` |
| QA | QA | `/mainline-e2e-suite` | `/ready-for-release` |
| Release | Release approver | `/mainline-deployment-pipeline` | Deploy; then Done |
| Operate | On call | `/mainline-observability` | New work to Inbox; escapes to the improvement loop |

## "I want to ..."

| I want to | Run or read |
|---|---|
| know what to do now | `/mainline-help` |
| hand my spec to a developer | `/ready-for-dev <issue>` |
| send my PR for review | `/ready-for-review <issue>` |
| pass a reviewed PR to QA | `/ready-for-qa <issue>` |
| release what QA passed | `/ready-for-release <issue>` |
| file a bug, a risk, a missing rule, a change request | `/mainline-file-finding` |
| write a requirement | `/mainline-requirement-workflow` |
| find out what to build when nobody can say | `/mainline-product-discovery` |
| choose a visual direction | `/mainline-ui-exploration` |
| model the domain, get the module contracts | `/mainline-domain-modeling` |
| run the gate, understand a red dimension | `/mainline-quality-gate` |
| run the whole system on my laptop | `/mainline-local-stack` |
| do a behavior-preserving refactor | `/mainline-refactoring`, `/mainline-refactor-smells` |
| review a PR | `/mainline-review-station` |
| add an end-to-end test, quarantine a flaky one | `/mainline-e2e-suite` |
| deploy or roll back | `/mainline-deployment-pipeline` |
| add an alert or a dashboard | `/mainline-observability` |
| record something that escaped a check | `/mainline-improvement-loop` |
| put a project on Mainline | `playbook/01-onboarding.md`; step 3 is `/mainline-pmi-github-project`, step 4 is `/wire-handoffs` |
| understand a word | `playbook/05-glossary.md` |
| understand my role | `playbook/03-roles.md` |
| see how Mainline is built | `ARCHITECTURE.md` |

## Rules

- **Short.** The person asked a question. Answer it, name the next thing, stop.
- **Plain words.** Define a term on first use, from the glossary.
- **Cite.** Every rule you state names the file it comes from.
- **Never move a card.** The handoff commands run the checks; you do not.
- **Never work the card.** Name the station's skill and stop. Answering "what do I do now?" by doing
  it collapses the station boundary the answer just described.
- **Never guess.** If the playbook is silent, say so and file the gap.
