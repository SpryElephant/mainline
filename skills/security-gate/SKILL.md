---
name: security-gate
description: The DevSecOps gate — static application security testing, dependency and CVE scanning, secrets detection, infrastructure-as-code and container image scanning, defined as dimensions with tools calibrated per stack and wired to fail the build rather than fill a dashboard. Use when onboarding a project onto the line, at the Review station, and whenever a vulnerability class needs to stop recurring.
---

# Security gate

Same shape as `quality-gate`, aimed at a different question: not *is this correct* but *what does
this let an attacker do*. **Dimensions are invariant; tools are calibrated per stack.**

The distinction from `quality-gate` dimension 3 is worth being precise about. Static analysis asks
whether the code is bad practice. This asks whether it is exploitable. They overlap and neither
subsumes the other — a perfectly idiomatic function can still concatenate user input into SQL.

**Failures fail the build.** A security tool that writes to a dashboard is a security tool nobody
reads. The whole reason `quality-gate` works is that it is binding, and the same applies here.

## Dimensions

### 1. SAST — the code

Taint-style analysis for injection, deserialization, path traversal, SSRF, weak crypto, authz gaps.

| Reference | Python | C# / .NET | TypeScript / JS | Go | Polyglot |
|---|---|---|---|---|---|
| Semgrep | `bandit`, Semgrep | Roslyn security analyzers | ESLint security plugins, Semgrep | `gosec` | CodeQL, Semgrep |

Where `quality-gate` dimension 5 (CPG) is already in use, some of this belongs there instead — taint
from an HTTP boundary to a sink is exactly what a code-property graph is for. Do not run two tools to
prove the same fact.

### 2. Dependencies — SCA

Known-vulnerable packages, transitively. This is where most real risk actually lives, and it is the
cheapest dimension to turn on.

| Ecosystem | Tool |
|---|---|
| Any | Trivy, Grype, Snyk, GitHub Dependabot alerts |
| JVM | OWASP Dependency-Check, `gradle dependencyCheckAnalyze` |
| Node | `npm audit`, `osv-scanner` |
| Python | `pip-audit`, `safety` |
| .NET | `dotnet list package --vulnerable` |
| Go | `govulncheck` — reachability-aware, so far fewer false alarms |

Pin a lockfile. A scan without one describes a build that no longer exists.

### 3. Secrets

Credentials in the repo, in history, in CI logs, in container layers.

Tools: `gitleaks`, `trufflehog`, GitHub secret scanning + push protection.

- Run on the diff **and** on history — a rotated key in an old commit is still a live key if it was
  never actually rotated.
- **A detected secret is an incident, not a lint failure.** Rotate it first. Removing the commit
  without rotating leaves the credential valid and the team believing it is not.
- Push protection on, so the answer to "how did that get in" stops being "through the front door."

### 4. Infrastructure as code

Public buckets, permissive security groups, unencrypted volumes, over-broad IAM, missing logging.

Tools: `checkov`, `tfsec`/Trivy, `kube-linter`, `cfn-nag`.

Cheap and high-yield: most cloud incidents are misconfiguration, not exploitation.

### 5. Container images

Base-image CVEs, root users, embedded secrets. Tools: Trivy, Grype, Docker Scout.

Rebuild on a schedule, not only on change. A base image that was clean in March is not clean now, and
an unchanged service is exactly the one nobody is looking at.

### 6. Runtime posture *(only where it applies)*

Auth flows, session handling, security headers, TLS, rate limiting. DAST tools (OWASP ZAP) against
the running stack. Reach for it when the application is internet-facing and the E2E suite already
gives you a running system to point at.

## Where each dimension binds

Match the check to the window it fits in:

| Dimension | Binds on | Why |
|---|---|---|
| SAST (diff-scoped) | PR | Fast. Catches it while the author still has context. |
| Secrets | PR + push protection | Cheapest possible place. |
| Dependencies | PR + nightly | PR catches what you added; nightly catches what the world discovered overnight. |
| IaC | PR | It is code. |
| Images | Build + scheduled rebuild | Drift is time-based, not change-based. |
| DAST | Release path | Needs a running system, too slow for a PR. |

Anything that does not fit the PR window runs on a schedule and **fails loudly to an owner** — never
into a report.

## Handling the backlog

Turning six scanners on an existing codebase produces hundreds of findings, and the usual outcome is
that someone disables the scanners. Do not let that be the outcome.

1. **Baseline explicitly.** Snapshot today's findings into an ignore file **with dates and owners**.
   The gate blocks anything *new* from the first day.
2. **Burn the baseline down** as `Platform` work items, worst exposure first. A baseline with no
   burn-down plan is a permanent exemption wearing a temporary name.
3. **Never add to the baseline to get green.** That is the security equivalent of lowering a coverage
   threshold, and it is the single failure that makes the whole gate decorative.

## Triage policy

Agree it once, write it in the project charter, and hold to it:

| Severity | Action |
|---|---|
| Critical | Stop the line. Fix or roll back today. |
| High | Fixed before the next release. |
| Medium | A `Platform` work item with a date. |
| Low | Batched into scheduled maintenance. |
| False positive | Suppressed **inline, with a written reason**. Never globally. |

An unwritten policy means every finding is renegotiated from scratch, which reliably converges on
ignoring them.

## Failure modes

- **The dashboard.** Findings go to a tool nobody opens. If it does not fail a build, it does not
  exist.
- **Alert fatigue.** Every finding at maximum severity, so none of them mean anything. Tune the rules
  or the tool loses its authority permanently.
- **Baseline as a graveyard.** Suppressions with no owner and no date.
- **Deleting the commit instead of rotating the key.** The most common secrets mistake, and it leaves
  you exploitable while feeling resolved.
- **Scanning only what changed, forever.** New CVEs land against code you did not touch. Something
  must run on a clock.

## Relationships

- **`quality-gate`** — same discipline, same one-command wiring. Keep them separate commands so a
  security failure is legible as a security failure.
- **`deployment-pipeline`** — dimensions that bind on the release path live there.
- **`observability`** — detection at runtime for what the gate cannot see statically.
- **`review-station`** — the human reads these findings and either fixes or waives them **with a
  written reason**.
