# The improvement loop

Something reached a station it should never have reached. A defect surfaced in QA that Review should
have caught. A requirement gap surfaced in Build that `/ready-for-dev` should have caught. Something reached
production that nobody caught at all.

**That is a defect in the process, not only in the code.** Fixing the code and moving on guarantees
the same class of miss recurs, because nothing about the line changed.

**Owner: the lead. Skill:** `/mainline-improvement-loop` — the loop itself, the rules, the ledger format and
the four metrics live there. Run it whenever something escapes.

In short: name the escape, find the **earliest** handoff that could have caught it, amend that check
— a tool first, a checklist line second, an explicit decision to change nothing third — file the
amendment as a `Platform` item with an owner and a date, and record a row in the ledger.

## Escape ledger — `docs/escape-ledger.md`

| Date | What escaped | Surfaced at | Should have been caught at | Change made |
|---|---|---|---|---|
| *2026-08* | A feature shipped without enterprise SSO. Enterprise tenancy was assumed by everyone and written down by no one, so nothing downstream could check it — downstream can only check what upstream wrote. | Production | **`/ready-for-dev`** | Added the NFR line to `/ready-for-dev`: auth and SSO, **tenancy**, performance, compliance, retention — each stated or explicitly marked N/A. |
| *2026-08* | On the first step-8 run, one session carried a Feature from Requirement to Release and ran no handoff command. Every station's work was done by the actor who had just done the previous station's, so no check ever met the work from outside. | Onboarding step 8 | **Onboarding step 4** — the handoffs were wired but nothing said a session must stop at one | Added the fourth rule, **one station, one sitting**, to `00-overview.md`; a stop step to each `/ready-for-…` command; removed the solo-project exemption from `/mainline-pmi-github-project`; widened the roles rule in `03-roles.md`; and tightened step 8's acceptance test to require the four commands, timestamped in station order across separate sittings. |

The rows above are worked examples, kept because they are the clearest we have. A project starts its
own ledger empty.

The second one is worth reading twice. Nothing was broken — every station's work was done, and done
well. What was missing was the boundary between them, and the escape was invisible on the board,
because the board showed a card that had moved through every phase. That is what a process defect
looks like when the code is fine.

Add a row every time. The ledger is the evidence that the line is improving, and it is the first
thing to show someone who asks what they are paying for.

**Keep it anonymous enough to share.** Describe the miss, not the account. "A feature shipped without
enterprise SSO" teaches everything the entry needs to teach; naming the client teaches nothing and
makes the ledger unshareable.
