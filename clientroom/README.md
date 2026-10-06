# Clientroom: a client portal and invoicing for solo freelancers

One branded link per client project. The client sees the agreed proposal, the milestones, every deliverable
with its version history, an "approve" or "request changes" on each, and every invoice with a way to pay. The
freelancer sees all clients, projects and money in one place, and reminders and the paper trail happen
automatically.

**Why it's a showcase.** Plenty of freelancers have tried to build this in Notion. It looks like a project
board plus a payment link. Underneath are two sides with different permissions, proposals that work as
evidence, approvals that trigger invoices, tax and currency per country, files over a gigabyte, and money that
must never pass through the platform.

## Read it in 10 minutes

1. [napkin.md](napkin.md): what the founder typed (about 1,100 words)
2. [BRIEF.md](.n2b/BRIEF.md): the validated brief
3. [product-features.md](.n2b/features/product-features.md): 33 features, each ranked and phased
4. A spec: [approvals that can't be edited afterwards](.n2b/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-004-entry-immutability-content-attribution-rules.md),
   from the [immutable activity trail](.n2b/specifications/FEAT-13-immutable-activity-audit-trail/feature-overview.md)
5. [technical-architecture.md](.n2b/architecture/technical-architecture.md) and [database-schema.md](.n2b/architecture/database-schema.md)
6. **Build it:** [speckit](.n2b/exports/speckit/README.md). This is [GitHub Spec Kit](https://github.com/github/spec-kit)
   format: a `.specify/` constitution and 33 numbered spec folders in build order, ready for Spec Kit's plan and tasks steps.

## What the blueprint added that the napkin didn't say

- **The paper trail is a feature with rules.** Accepted proposals, approvals and sent invoices go into an
  activity trail that is never silently edited. It has a printable record copy and a retention rule that still
  works when an account is deleted.
- **Client-side roles are spelled out** (FEAT-18): who signs, who reviews, who sees invoices, who can invite
  colleagues. That was the napkin's first open question.
- **The open questions were scoped, not guessed.** Legally binding e-signatures (FEAT-26) and a custom domain per
  freelancer (FEAT-27) became post-MVP features (v1 and Later), each with full specs. The feasibility review flags e-signature standards as a
  research spike.
- **Hard problems are named.** Invoice generation, payment processing, and data export plus deletion are rated
  Hard. The architecture flags storage cost for big files against the $100/month budget.

## This run

| | |
|---|---|
| Runtime | Claude Code (cloud sessions, one pipeline step each) |
| n2b | 0.4.0 |
| Model profile | balanced (claude-aliases) |
| Dates | 2026-09-26 → 2026-10-02 |
| Blueprint | 33 features · 220 specs · 3,102 acceptance criteria · 31 ADRs · 49 tables |
| Recommended stack | Next.js 16 · Neon Postgres · Drizzle · Cloudflare R2 · Resend · Stripe Connect / Billing · Trigger.dev + outbox · Better Auth + magic links · Documenso · Vercel |
| Exports | [dev-brief](.n2b/exports/dev-brief/00-README.md) (44 files) · [speckit](.n2b/exports/speckit/README.md) (338 files) · jira (halted by the gate, see below) |
| Fidelity | dev-brief and speckit passed on the first attempt; see each `FIDELITY-REPORT.md` and `EXPORT-RECEIPT.md` |

The full step-by-step record is in [RUN-LOG.md](RUN-LOG.md). The intake interview is in
[session-notes.md](session-notes.md), with the founder answer key in [ANSWERS.md](ANSWERS.md).

## The gate that said no

The first second export was **Jira**. A Jira backlog needs every acceptance criterion in a "when…, then…" shape so
it can become a story. 3,101 of 3,102 were. One (`FEAT-20.SPEC-002-AC-25`) had no "then" clause. n2b's backlog
builder refused to produce a backlog with a hole in it, failed twice the same way, and halted.

Nothing under `.n2b/` may be hand-edited, so the run switched its second export to Spec Kit, which passed. The
halted attempt's [FIDELITY-REPORT.md](.n2b/exports/jira/FIDELITY-REPORT.md) is kept as the record. The root cause
was that Stage 3 never checked acceptance-criterion shape. It's fixed upstream in n2b
([#23](https://github.com/tsmztech/napkin-to-blueprint/issues/23)).

## Other deviations (as logged, not cleaned up)

- **Stage 3:** the orchestrator spawned the feature analysts directly, because Claude Code subagents can't spawn
  subagents. Fixed upstream in n2b ([#22](https://github.com/tsmztech/napkin-to-blueprint/issues/22)).
- **dev-brief:** 1 non-blocking style warning (the article in "a Important-tier" chapter openers).

## Other runs of this napkin

None yet. A re-run on another runtime or model profile will go in a sibling folder named `clientroom--<runtime>`.
