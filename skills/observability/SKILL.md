---
name: observability
description: Know what production is doing and turn what it tells you back into work — instrumentation before ship, alerts on user-visible symptoms with an owner and an action, one-command alert-to-ticket, and incident review that feeds the improvement loop. Use when onboarding a project onto the line, at the Operate station, and whenever an incident was found by a customer rather than by a signal.
---

# Observability

The Operate station's machinery, and the thing that makes moving fast survivable.

The reason it matters more now than it used to: the line ships more, in smaller pieces, more often.
That is strictly better *if* you find out quickly when something is wrong, and strictly worse if you
do not. Observability is what converts speed from a risk into a capability — it is why the team can
move fast without dreading it.

## The rule

**Instrumentation ships with the feature, not after the first incident.** A dashboard added after an
outage is a dashboard that was missing during the outage. The Release station's checklist says alerts
and dashboards for the new behavior exist *before* it goes live, and this is the skill that makes
that checkable rather than aspirational.

## What to instrument

| Signal | What it answers | Watch for |
|---|---|---|
| **Errors** | What is broken right now | Grouped by cause, not by stack frame. With release version and user impact attached. |
| **Latency** | Is it usable | p50, p95, p99 — never the mean. The mean hides exactly the users who are suffering. |
| **Traffic** | Is anyone there | A sudden drop is as much a signal as a spike, and is more often missed. |
| **Saturation** | What runs out next | Connections, memory, queue depth, disk, rate-limit headroom. |
| **Business outcomes** | Is it doing its job | Orders placed, refunds processed, sign-ups completed. **The most valuable and most often skipped.** |

Business metrics catch the failures technical metrics cannot see: everything returns 200 and nobody
can complete a purchase. If you instrument one thing beyond errors, instrument the outcome the
feature exists to produce.

Structured logs, correlation IDs across services, and traces on paths that cross a boundary. A log
line you cannot correlate to a request is an anecdote.

## Alerts

**An alert is a claim that a human must act, now.** Everything else is a dashboard.

Every alert has three things, or it does not get created:

1. **An owner** — a person or rotation, not a channel nobody owns.
2. **An action** — the first thing to do, linked. Even one line beats none.
3. **A user-visible symptom behind it** — alert on what people feel (error rate, latency, orders
   failing), not on causes (CPU at 80%). Cause-based alerts fire constantly during normal operation
   and stay silent during the novel failure.

Rules:

- **Every alert fires to a channel someone actually reads.** An alert routed to a channel with no
  readers is a monitoring system that exists to make you feel monitored.
- **Delete alerts nobody acts on.** An alert acknowledged-and-ignored three times is training the
  team to ignore alerts, and that training generalizes to the alert that mattered.
- **Page only for what warrants waking someone.** Everything else is a ticket. Getting this wrong in
  either direction is expensive: too many pages burns out the rotation, too few means customers are
  your monitoring.
- **Tune noisy alerts the week they get noisy**, or they train the exact reflex they exist to prevent.

## Alert → ticket in one command

The Operate station's harvesting rule, and the same principle as filing findings from inside a
session: **if turning a signal into tracked work is friction, signals get dropped.**

One command takes an alert or an error group and produces a work item with the alert, the trace, the
affected release, and the impact already attached — assigned and notified. Nobody retypes anything.

## Dashboards

One per service, answering in ten seconds: *is it up, is it fast, is it erroring, is it doing its
job?* Plus a release marker overlay, because "what changed" is the first question in every incident
and deploy time answers it more often than anything else.

Resist the wall of forty graphs. A dashboard nobody can read under pressure is decoration.

## After an incident

1. **Restore service first.** Understanding is a separate activity from stopping the bleeding.
2. **Write it up while it is fresh** — timeline, impact, what was actually wrong, how it was found.
3. **How was it found?** If the answer is "a customer told us," the gap is a missing signal, and that
   is its own work item.
4. **Blameless.** A person who made a mistake was working in a system that allowed it. Change the
   system.
5. **Feed the improvement loop.** An incident that reached production escaped a check upstream — find
   the earliest one that could have caught it and amend it. Actions become `Platform` work items with
   owners and dates, or they do not happen.

Usually two outputs: an **E2E test** so that exact defect cannot recur, and an **alert** so the next
thing you did not think of is caught faster.

## SLOs *(when the service is important enough)*

Pick the few user journeys that matter, set an objective, measure it, and alert on burn rate rather
than instantaneous thresholds. The error budget converts "are we moving too fast?" from an argument
into a number.

Do not do this for everything. Three meaningful SLOs beat thirty nobody looks at.

## Failure modes

- **Everything logged, nothing observable.** Terabytes of unstructured logs and no correlation ID.
- **Alerts on causes.** CPU alerts fire during normal load and miss the novel failure entirely.
- **The unread channel.** Alerts route somewhere nobody has opened in months.
- **Instrumentation added after the incident**, every time — so every first occurrence is invisible.
- **No business metrics.** All systems green, no orders since Tuesday.
- **Incident write-ups with no owner or date on the actions.** The review happened; nothing changed.

## Relationships

- **`deployment-pipeline`** — release markers on dashboards; automated rollback triggers read these
  signals.
- **`e2e-suite`** — an incident usually produces a test as well as an alert.
- **`security-gate`** — runtime detection for what static analysis cannot see.
- **`requirement-workflow`** — new work discovered in production enters at Inbox like anything else.
- **The improvement loop** — where escapes go, and where alerts that keep firing get re-examined.
