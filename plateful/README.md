# Plateful: a family meal planner with AI suggestions

A shared household space. Set up the family once: who eats, what each person can't or won't eat, which nights
are rushed, and the weekly budget. Each week the AI proposes a realistic 7-day dinner plan, uses up what's
already in the fridge, and produces one shared grocery list the whole household ticks off live in the store.

**Why it's a showcase.** It's the most-built AI app genre, the "even the simple one has depth" case. One wrong
suggestion here is a walnut salad for a child with a nut allergy. There are kids in the household, shared live
editing, a supermarket with no signal, and AI costs that must stay small per household.

## Read it in 10 minutes

1. [napkin.md](napkin.md): what the founder typed (about 1,000 words)
2. [BRIEF.md](.n2b/BRIEF.md): the validated brief
3. [product-features.md](.n2b/features/product-features.md): 25 features, each ranked and phased
4. A spec: [fail-closed allergy policy](.n2b/specifications/FEAT-02-dietary-rules-allergy-safety-engine/FEAT-02.SPEC-007-ingredient-data-completeness-and-fail-closed-policy.md),
   one of 14 specs in the [allergy safety engine](.n2b/specifications/FEAT-02-dietary-rules-allergy-safety-engine/feature-overview.md)
5. [technical-architecture.md](.n2b/architecture/technical-architecture.md) and [database-schema.md](.n2b/architecture/database-schema.md)
6. **Build it:** [agent-workspace](.n2b/exports/agent-workspace/README.md). It's a repo-shaped bundle for coding
   agents (Claude Code, Codex, Cursor, Devin): `AGENTS.md`, `BUILD-ORDER.md`, `OPERATING-RULES.md`, and a
   `feature_list.json` with all 196 specs to tick off.

## What the blueprint added that the napkin didn't say

- **Allergy safety is an engine, not a filter.** There are 14 specs: the app checks every candidate meal before
  anyone sees it, re-checks the week when a rule changes mid-week, and **fails closed** when ingredient data is
  incomplete. Families can also report a safety concern, and that feeds an operator alert. The AI is never the
  last line of defence.
- **Kids are handled as data, not as accounts.** Parents manage the profiles, and older kids can vote on
  dinners (FEAT-17). That answers the napkin's open question about how to represent kids, and it keeps
  children's data minimal.
- **The grocery list survives the supermarket.** Live shared editing, aisle grouping, and offline handling,
  because the napkin said "bad signal".
- **The founder's wish list was scoped, not dropped.** Online grocery ordering and family calendar sync became
  Later-phase features with full specs.
- **A success metric became a feature.** "Families throw away less and spend less" can't be proven unless the
  app asks, so a weekly waste-and-spend check-in (FEAT-25) ships in the MVP.

## This run

| | |
|---|---|
| Runtime | Claude Code (cloud sessions, one pipeline step each) |
| n2b | 0.4.0 |
| Model profile | balanced (claude-aliases) |
| Dates | 2026-09-26 → 2026-09-29 |
| Blueprint | 25 features · 196 specs · 2,292 acceptance criteria · 34 ADRs · 28 tables |
| Recommended stack | Next.js PWA · Supabase · Drizzle · Inngest · Claude · Stripe · Vercel |
| Exports | [dev-brief](.n2b/exports/dev-brief/00-README.md) (36 files) · [agent-workspace](.n2b/exports/agent-workspace/README.md) (246 files) |
| Fidelity | both exports passed on the first attempt; see each `FIDELITY-REPORT.md` and `EXPORT-RECEIPT.md` |

The full step-by-step record is in [RUN-LOG.md](RUN-LOG.md). The intake interview is in
[session-notes.md](session-notes.md), with the founder answer key in [ANSWERS.md](ANSWERS.md).

## Deviations (as logged, not cleaned up)

- **Stage 3:** the orchestrator spawned the feature analysts directly, because Claude Code subagents can't spawn
  subagents. Fixed upstream in n2b ([#22](https://github.com/tsmztech/napkin-to-blueprint/issues/22)).
- **Stage 4:** 7 technology-landscape cells fell back to the model's own knowledge instead of web sources. Each
  one is marked. One decision (ADR-028, Zod) uses a library that wasn't in the landscape doc.
- **dev-brief:** 3 non-blocking style warnings: a duplicated bridge heading, and the article in "a
  Important-tier" chapter openers.

## Other runs of this napkin

None yet. A re-run on another runtime or model profile will go in a sibling folder named `plateful--<runtime>`.
