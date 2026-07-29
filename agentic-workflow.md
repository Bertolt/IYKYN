# How I work with AI — the wider system

The transcripts in this folder are single sessions. This file is the bigger picture: across an
8-repo AI platform I work on, agentic tooling isn't just *used to write code* — it's a **standing
part of the engineering system**. Custom subagents handle review and cross-repo safety, a library of
skills runs the day-to-day, and a set of scheduled routines quietly keep the codebase healthy while
opening work for humans to approve.

It's deliberately high-level — enough to show the shape, not the internals. The tooling is
concentrated in the services doing the heavy lifting.

---

## Custom subagents (~18)

Purpose-built agents I reach for so the main thread stays focused. Grouped by what they do:

- **Review & architecture** — specialists that review code, backend/frontend architecture, container
  stacks and deploy scripts for quality, security and convention.
- **Cross-repo & contract safety** — agents that check a change against sibling repos and validate
  API implementations against their specs, so a change in one place doesn't silently break another.
- **Read-only explorers** — context-gatherers that map a neighbouring service, the database, or a
  trace without touching anything, so decisions are made on evidence.
- **Domain specialists** — agents that know a specific subsystem (the AI service, the data pipelines)
  well enough to work in it safely.
- **Quality & evaluation** — an agent that runs end-to-end tests and judges output quality
  (LLM-as-judge) to catch regressions.
- **Ops** — guarded agents that watch for changes, triage issues from logs, and carry out server
  operations under strict safety rails.

## Skills (~20)

On-demand commands that turn repeatable work into one step:

- **Dev-environment control** — start/stop/verify the local services and dashboards consistently.
- **PR review & repo hygiene** — opinionated PR review (including the mandatory cross-repo checks),
  submodule re-pinning, and branch/worktree cleanup.
- **Evaluation harnesses** — launch the end-to-end quality evals and trace-inspection flows.
- **Pipeline control** — configure, launch and track data/ETL pipeline runs.
- **Workflow glue** — file well-structured tickets into the board that the autonomous routines pick
  up (see below).

## Scheduled routines — autonomous, always human-reviewed

A set of routines run on a schedule and do real maintenance on their own — but **every one of them
ends at a human**: it opens a *draft* PR or files a ticket, never merges. Autonomy with a person in
the loop.

- **Issue → draft-PR coding bot** — pulls small, well-scoped tickets from a project board and opens
  draft PRs to resolve them.
- **Weekly PR review** — a read-only pass over open PRs that leaves review comments.
- **Docs-drift sync** — re-aligns documentation with what actually merged that week.
- **Test-coverage** — adds the highest-value missing tests as a draft PR.
- **Tech-debt scouting** — finds and files atomic tech-debt tickets for the coding bot to pick up.
- **Nightly quality eval** — runs the evaluation harness on a schedule and records scores, so quality
  regressions surface on their own.

## Agentic release — fully agentic, with a human in the loop

The release pipeline for the platform is the clearest example of the whole philosophy. **The release
runbook is codified as an agent orchestration rather than a hand-run checklist** — a release skill
sequences the upgrade end-to-end and delegates every host-mutating step to a single guarded ops
agent, bracketed by automated pre-flight (go/no-go) and post-deploy verification gates.

And it is **fully agentic with a human firmly in the loop**. The deliberate human gates:

- **Staging before production** — a release must deploy to staging, be verified green, and carry a
  committed sign-off before the same flow is allowed to run against production.
- **Human-run migrations** — database migrations are never executed by the agent; the exact step is
  surfaced for a person to run.
- **Review as normal** — config and migration changes ship as reviewed PRs.
- **Guardrails as a backstop** — destructive operations (data, containers, force-push) are blocked at
  the tool layer regardless of what any agent attempts.

Conceptually: **cut → preflight → deploy → verify → sign-off**, run once for staging and again for
production, with a human-gated sign-off between them.

---

**The throughline:** the judgment stays human — what to build, what to trust, when to ship. The
agents carry it out, and everything they produce comes back to a person before it counts.
