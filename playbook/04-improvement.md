# The improvement loop

Something reached a station it should never have reached. A defect surfaced in QA that Review should
have caught. A requirement gap surfaced in Build that H1 should have caught. Something reached
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
| *2026-08* | A feature shipped without enterprise SSO. Enterprise tenancy was assumed by everyone and written down by no one, so nothing downstream could check it — downstream can only check what upstream wrote. | Production | **H1** | Added the NFR line to H1: auth and SSO, **tenancy**, performance, compliance, retention — each stated or explicitly marked N/A. |

The row above is a worked example, kept because it is the clearest one we have. A project starts its
own ledger empty.

Add a row every time. The ledger is the evidence that the line is improving, and it is the first
thing to show someone who asks what they are paying for.

**Keep it anonymous enough to share.** Describe the miss, not the account. "A feature shipped without
enterprise SSO" teaches everything the entry needs to teach; naming the client teaches nothing and
makes the ledger unshareable.
