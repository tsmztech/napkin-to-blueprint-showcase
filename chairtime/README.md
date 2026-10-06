# Chairtime: deposit-first booking for solo beauty & wellness pros

A mobile-first booking link for one independent pro: a barber, nail tech, lash artist, massage therapist or
tattoo artist. Clients pick a service and a time, pay a card deposit and get reminders. The pro stops losing
money to no-shows and stops running their diary from Instagram DMs.

**Why it's a showcase.** Booking apps are among the most-built ideas for vibe coders. It looks like a calendar
plus a form. Underneath are two roles with different views, a real money flow (deposit → balance → no-show
forfeit → refund window), two-way calendar sync, opt-in for text messages, a card-data boundary, time zones,
and making sure two clients can never get the same slot.

## Read it in 10 minutes

1. [napkin.md](napkin.md): what the founder typed (about 1,100 words)
2. [BRIEF.md](.n2b/BRIEF.md): the validated brief, with the founder's open questions kept as questions, not guessed
3. [product-features.md](.n2b/features/product-features.md): 30 features, each ranked and phased
4. A spec: [no-show marking + deposit forfeit](.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-002-no-show-marking-deposit-forfeiture.md)
5. [technical-architecture.md](.n2b/architecture/technical-architecture.md) and [database-schema.md](.n2b/architecture/database-schema.md)
6. **Build it:** [lovable-pack/PROMPTS.md](.n2b/exports/lovable-pack/PROMPTS.md). Prompt 0 sets up the project, then
   there is one prompt per feature, in build order. Paste [KNOWLEDGE.md](.n2b/exports/lovable-pack/KNOWLEDGE.md) into
   Lovable's project knowledge first.

## What the blueprint added that the napkin didn't say

- **Open questions became scoped features, not guesses.** The napkin asked about a waitlist, recurring
  appointments, paying the balance in the app, and tipping. Each became a Nice-to-Have feature with its own specs
  (FEAT-20 to FEAT-23), so a builder can see the cost of saying yes.
- **A completeness audit found 4 missing features:** booking-page settings, payout account connection, pro
  sign-in and account lifecycle, and the pro's own booking management. The draft had 26 features. Look for
  `[AUDIT-ADDED]` markers.
- **Market research changed the product.** Competitors' layered fees are the top trust complaint, so the
  subscription is one flat price, with a 30-day notice for any price change. Look for `[RESEARCH-INFORMED]` markers.
- **Hard problems are flagged.** The feasibility review rates real-time slot availability (FEAT-03) as Hard and
  two-way calendar sync (FEAT-04) as needing a research spike before anyone commits to a date.

## This run

| | |
|---|---|
| Runtime | Claude Code (cloud sessions, one pipeline step each) |
| n2b | 0.4.0; lovable-pack exported on 0.4.1 (see deviations) |
| Model profile | balanced (claude-aliases) |
| Dates | 2026-09-26 → 2026-10-02 |
| Blueprint | 30 features · 219 specs · 2,996 acceptance criteria · 30 ADRs · 41 tables |
| Recommended stack | Next.js 15 · Supabase Postgres · Drizzle · Better Auth · Stripe Connect · Twilio / Postmark · Inngest · Nylas · Vercel |
| Exports | [dev-brief](.n2b/exports/dev-brief/00-README.md) (41 files) · [lovable-pack](.n2b/exports/lovable-pack/README.md) (269 files) |
| Fidelity | both exports passed; see each `FIDELITY-REPORT.md` and `EXPORT-RECEIPT.md` |

The full step-by-step record is in [RUN-LOG.md](RUN-LOG.md). The intake interview is in
[session-notes.md](session-notes.md), with the founder answer key in [ANSWERS.md](ANSWERS.md).

## Deviations (as logged, not cleaned up)

- **Stage 2 market research** ran with web page fetching blocked in the cloud sandbox, so it used web search
  results only. This is logged in `.n2b/tracking/stages/`.
- **Stage 3:** the orchestrator spawned the feature analysts directly, because Claude Code subagents can't spawn
  subagents. Fixed upstream in n2b ([#22](https://github.com/tsmztech/napkin-to-blueprint/issues/22)).
- **The lovable-pack export failed its first fidelity gate.** Nine spec summaries quoted other features' IDs,
  and the verbatim rule clashed with the "no foreign IDs in a prompt" rule. n2b 0.4.1 fixed the rule
  ([#19](https://github.com/tsmztech/napkin-to-blueprint/issues/19)), and the re-run passed with all 219 summaries
  byte-for-byte canonical. Both attempts are in `RUN-LOG.md` (rows 32–33).
- One architecture decision (ADR-024, Zod) uses a library that wasn't in the technology landscape doc. It's logged
  as a deviation in Stage 4.

## Other runs of this napkin

None yet. A re-run on another runtime or model profile will go in a sibling folder named `chairtime--<runtime>`.
