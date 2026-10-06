---
document_type: technical-feasibility
produced_by: feasibility-planner
status: final
stage: 4
feature_count: 25
created: 2026-09-28
---

# Technical Feasibility Assessment

## 1. Feasibility Summary

| Feature | Verdict | Driving Factors |
|---------|---------|-----------------|
| FEAT-01 (Household Setup & Member Profiles) | Standard-with-integration | Transactional email for sign-up/recovery is an external contract (FEAT-01.SPEC-017 ## Degradation Behavior); offline draft queuing (FEAT-01.SPEC-013 ## Business Rules) and role rules (FEAT-01.SPEC-016) are established patterns |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Hard | Fail-closed, exhaustive allergen matching on every path onto the plan, including compound-term expansion, inside the swap/generation latency budgets (FEAT-02.SPEC-002 ## Processing Logic, ## Edge Cases; FEAT-02.SPEC-007 ## Field Validation Rules; feature-overview.md ## Non-Functional Notes) |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Research-spike recommended | Whether an LLM selecting from the household's candidate pool reliably returns a full 7-night, safety-passing, in-budget week in well under a minute at small per-household cost is unproven (FEAT-03.SPEC-010 ## Data Exchanged, ## Edge Cases; FEAT-03.SPEC-006; ASMP-23, SC-16) |
| FEAT-04 (One-Tap Meal Swap) | Standard-with-integration | AI alternatives are an external call with a specified degradation contract (FEAT-04.SPEC-007 ## Degradation Behavior); the per-slot lock (FEAT-04.SPEC-009 ## Business Rules) is a conventional row-level pattern |
| FEAT-05 (Pantry-Aware Suggestions) | Straightforward | Short household list with exact-name matching and merge-on-duplicate (FEAT-05.SPEC-003, FEAT-05.SPEC-006 ## Business Rules); offline adds reuse the shared offline-queue machinery |
| FEAT-06 (Shared Grocery List) | Hard | Live multi-device sync within 2 seconds plus offline queue reconciliation with idempotent ticks, last-write-wins quantities, deletion precedence and concurrent recalculation merge (FEAT-06.SPEC-005 ## Degradation Behavior; FEAT-06.SPEC-008 ## Cross-Field Rules, ## Edge Cases) |
| FEAT-07 (Weekly Plan Ready Notification) | Standard-with-integration | Push plus email fallback via external delivery services (FEAT-07.SPEC-005, FEAT-07.SPEC-006 ## Degradation Behavior) against an 80%-within-one-minute target (feature-overview.md ## Non-Functional Notes) |
| FEAT-08 (Recipe Library (Starter Recipes)) | Standard-with-integration | Starter corpus depends on an external recipe/food-content source meeting the ingredient-completeness bar (FEAT-08.SPEC-004 ## Data Exchanged, ## Edge Cases); search itself is simple filter (FEAT-08.SPEC-001) |
| FEAT-09 (Household Invitations & Membership) | Straightforward | Link-based invitations, 14-day scheduled expiry and first-write-wins acceptance races are conventional (FEAT-09.SPEC-006; FEAT-09.SPEC-007 ## Edge Cases); notifications are in-app only (FEAT-09.SPEC-012..014 ## Channels) |
| FEAT-10 (Recipe Import from Web Link) | Standard-with-integration | Web-page extraction is an external/self-hosted capability with a defined failure path to manual entry (FEAT-10.SPEC-005 ## Degradation Behavior); legal acceptability is an open product question (ASMP-36) |
| FEAT-11 (Leftover Rollover to Lunches) | Straightforward | Deterministic post-generation slot computation with a two-day ceiling and withdrawal on source change (FEAT-11.SPEC-002 ## Processing Logic; FEAT-11.SPEC-004) |
| FEAT-12 (Meal Rating & Preference Learning) | Straightforward | Ratings are simple records; "learning" is a counted threshold and aggregated weighting input handed to FEAT-03 (FEAT-12.SPEC-004 ## Business Rules; FEAT-12.SPEC-005 ## Processing Logic) |
| FEAT-13 (Tonight's Dinner Reminder) | Standard-with-integration | Daily per-household-local-time push dispatch through the shared device-notification integration, plus a same-day correction (FEAT-13.SPEC-001 ## Trigger Definition; FEAT-13.SPEC-002, FEAT-13.SPEC-004 ## Channels) |
| FEAT-14 (Subscription & Billing Management) | Standard-with-integration | Payment processing with inbound renewal/retry events and a 7-day grace window (FEAT-14.SPEC-009 ## Inbound Events, ## Degradation Behavior; FEAT-14.SPEC-007) |
| FEAT-15 (Member Onboarding) | Straightforward | One-time routing after acceptance, reusing existing plan/list surfaces (FEAT-15.SPEC-002 ## Edge Cases; FEAT-15.SPEC-003) |
| FEAT-16 (Units, Currency & Locale Configuration) | Straightforward | Fixed conversion factors applied at display time, no exchange-rate service (FEAT-16.SPEC-004 ## Cross-Field Rules, ## Scope and Non-Goals) |
| FEAT-17 (Older-Kid Dinner Voting) | Straightforward | Small voting rounds with deterministic resolution (FEAT-17.SPEC-005 ## Processing Logic); depends on an older-kid limited-login role (FEAT-17.SPEC-006 ## Field Validation Rules) |
| FEAT-18 (Account & Data Management) | Standard-with-integration | Export file generation and storage, transactional email, and a 30-day cascade deletion that must reach external services (FEAT-18.SPEC-006, FEAT-18.SPEC-008 ## Processing Logic; FEAT-18.SPEC-012 ## Degradation Behavior) |
| FEAT-19 (Weekly Plan History) | Straightforward | Paginated reads over retained history with safety re-check on reuse (FEAT-19.SPEC-003 ## Processing Logic) |
| FEAT-20 (Online Grocery Ordering Handoff) | Research-spike recommended | No UK self-service grocer API was found and Instacart covers the US only (FEAT-20.SPEC-002 ## Data Exchanged; technology-landscape.md Online Grocery Ordering Integration) |
| FEAT-21 (Family Calendar Sync) | Standard-with-integration | OAuth calendar connection and per-night entry sync with silent retries (FEAT-21.SPEC-002 ## Degradation Behavior; FEAT-21.SPEC-004 ## Business Rules) |
| FEAT-22 (Operator Read-Only Support Access) | Straightforward | Request-scoped, read-only access gating with an append-only access record (FEAT-22.SPEC-004 ## Processing Logic; FEAT-22.SPEC-006, FEAT-22.SPEC-007) |
| FEAT-23 (Manual Weekly Planning) | Straightforward | Per-night picks with reject-with-refresh writes, riding on the shared safety engine and list recalculation (FEAT-23.SPEC-006 ## Edge Cases; FEAT-23.SPEC-004) |
| FEAT-24 (Invite Another Household) | Straightforward | Durable per-member referral link and a write-once attribution record (FEAT-24.SPEC-003 ## Processing Logic; FEAT-24.SPEC-006) |
| FEAT-25 (Weekly Waste & Spend Check-In) | Straightforward | One weekly record per household on a week-boundary schedule plus a simple trend calculation (FEAT-25.SPEC-003 ## Trigger Definition; FEAT-25.SPEC-004) |

## 2. Per-Feature Assessments

### FEAT-01 — Household Setup & Member Profiles

**Verdict:** Standard-with-integration — setup screens, validation and role rules are well-trodden (FEAT-01.SPEC-003..010, FEAT-01.SPEC-014, FEAT-01.SPEC-016), but account confirmation and password recovery depend on an external transactional-email capability with a queued-send degradation contract (FEAT-01.SPEC-017 ## Degradation Behavior).

**Required Capabilities:**
- Account sign-up, sign-in and password recovery with non-disclosing responses (FEAT-01.SPEC-001, FEAT-01.SPEC-002; FEAT-01.SPEC-017 ## Degradation Behavior — "timing differences never disclose whether an account exists")
- Role-based authorization across Organiser, Other Adult Member and kid profiles (FEAT-01.SPEC-016 ## Authorization Rules)
- Parental consent capture before a kid profile exists, recorded as the organiser's affirmative confirmation (FEAT-01.SPEC-007 ## Scope and Non-Goals); kid-profile data minimality enforced at validation (FEAT-01.SPEC-014)
- Dietary-rule capture against a standard allergen list plus free-text extra ingredients up to 80 characters (FEAT-01.SPEC-015 ## Field Validation Rules), with change history retained (feature-overview.md ## Non-Functional Notes)
- Background trigger on hard-rule changes that starts FEAT-02's mid-week re-check and an in-app notice (FEAT-01.SPEC-012; FEAT-01.SPEC-018 ## Channels); free-tier subscription provisioning at setup (FEAT-01.SPEC-011)
- Concurrency: Maya editing from laptop and phone at once resolves last-write-wins per setting, and a removal racing an own-account edit resolves reject-with-refresh (FEAT-01.SPEC-005, FEAT-01.SPEC-006, FEAT-01.SPEC-008 ## Edge Cases; feature-dependency-map.md, Household and Member Profile **Contention:**)
- Offline/degraded: every setup screen keeps a local draft and queues saves offline for automatic submission on reconnect (FEAT-01.SPEC-013 ## Business Rules); allergy removal is blocked offline because it needs a live check (FEAT-01.SPEC-015 ## Edge Cases); email outages never block account creation (FEAT-01.SPEC-017 ## Degradation Behavior)
- Scale: up to 12 member profiles per household across several thousand households, history kept for the life of the account (feature-overview.md ## Non-Functional Notes; ASMP-24); setup screens are exempt from the ASMP-23 heavier targets

**Candidate Approaches:** Identity can come from a managed provider (Clerk, Supabase Auth or Auth0 in the Authentication & Identity area), where Clerk's prebuilt components shorten the setup flow and Supabase Auth ties role checks to Postgres row-level security for kid-profile restrictions; or from a self-hosted library (Better Auth, NextAuth.js), which gives full control over the parental-consent gating and kid-profile no-login model at the cost of owning session hardening. Recovery and confirmation email fits any of Resend, Postmark or SendGrid (Email & Messaging Delivery); Postmark's transactional-only positioning suits time-sensitive recovery links, Resend's free tier suits early volume. Offline drafts map to the Service-worker / IndexedDB client-side caching option (Caching & Performance), with client state held in TanStack Query + Zustand, Redux Toolkit + RTK Query or Jotai (State Management); Jotai's atoms fit per-field draft state. Queued email sends need a durable job runner (Inngest, Trigger.dev, Upstash QStash or BullMQ in Background Jobs & Scheduling).

**Risks & Unknowns:** ASMP-27 states "verifiable parental consent" but FEAT-01.SPEC-007 ## Scope and Non-Goals relies on the organiser's affirmative confirmation with no verification service. Whether that satisfies children's-privacy-class obligations in the US and UK is a legal question, and the landscape has no consent-verification option; if stronger verification is needed, a new capability has to be researched (see Section 5). Offline-queued saves replayed after a server-side validation or authorization change (for example, the organiser role was handed over while a draft was queued) have to surface as a retry state and never be dropped silently (FEAT-01.SPEC-013 ## Business Rules). Dietary Rule data is the most sensitive class in the product, so any managed auth or BaaS vendor that stores profile data falls inside the children's-data processing boundary (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-02 — Dietary Rules & Allergy Safety Engine

**Verdict:** Hard — the check has to be exhaustive and fail closed on every path onto the plan (XBR-01). It compares full ingredient text against each member's rules, expands compound terms such as "mixed nuts" to every allergen they could contain, excludes any recipe with incomplete quantity or unit data, and must still fit the seconds-level swap and sub-minute generation budgets as the recipe pool grows, with "no full re-scan shortcuts" (FEAT-02.SPEC-002 ## Processing Logic, ## Edge Cases; FEAT-02.SPEC-007 ## Field Validation Rules; feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Deterministic, stateless per-candidate safety evaluation against every member's hard rules, with a vegetarian-variant exception and plain-language ineligibility reasons (FEAT-02.SPEC-002 ## Processing Logic steps 1–11; FEAT-02.SPEC-006; FEAT-02.SPEC-008)
- An ingredient-to-allergen classification that maps free-text ingredients from starter and imported recipes onto the standard allergen list and named extra ingredients, including compound-term expansion (FEAT-02.SPEC-002 ## Edge Cases; FEAT-01.SPEC-015 ## Field Validation Rules)
- Fail-closed completeness gate: every ingredient needs a name, quantity and unit (FEAT-02.SPEC-007 ## Field Validation Rules)
- Mid-week re-check on rule change that flags and removes failing dinners, offers alternatives and updates the list (FEAT-02.SPEC-003; XBR-02)
- Safety-concern intake that removes the meal, excludes the recipe while the report is open, raises a Support Request and emails the operator (FEAT-02.SPEC-004, FEAT-02.SPEC-005, FEAT-02.SPEC-009; FEAT-02.SPEC-010 ## Degradation Behavior; FEAT-02.SPEC-013 ## Channels — email)
- Concurrency: concurrent checks are read-only and independent (FEAT-02.SPEC-002 ## Edge Cases); a safety removal always wins over any concurrent plan change (feature-dependency-map.md, Weekly Plan and Planned Meal **Contention:**)
- Offline/degraded: the check never runs client-side unchecked, so nothing is shown unchecked when offline. The report screen needs connectivity (FEAT-02.SPEC-001 ## States). Operator email outages leave the Support Request visible in the Support View (FEAT-02.SPEC-010 ## Degradation Behavior)
- Scale: one check per candidate recipe per household on every generation, swap, manual pick, vote and history reuse across several thousand households, and it must stay equally fast as recipe pools grow (feature-overview.md ## Non-Functional Notes; ASMP-22, ASMP-23, ASMP-24)

**Candidate Approaches:** The engine is application logic in whichever Backend / API Layer option is selected (NestJS, Fastify, Express.js, Next.js Route Handlers / Server Actions or Django). Rule and ingredient data sit in a relational store (Neon, Supabase, Amazon RDS for PostgreSQL, PlanetScale for Postgres or self-hosted PostgreSQL in the Database area). Precomputing per-recipe allergen sets at ingest and caching per-household verdicts (Upstash Redis, Redis Cloud or Vercel KV in Caching & Performance) are ways to meet the latency budget without skipping checks. The ingredient-to-allergen taxonomy can come from a content vendor's ingredient data (Spoonacular Food API or Edamam Recipe Search API in Recipe & Food Content Data; Edamam is positioned for nutrition and allergen analysis) or from a curated in-house mapping maintained alongside the Curated/licensed starter set + manual editorial seeding option. Operator alerts go through any Email & Messaging Delivery option (Resend, Postmark, SendGrid).

**Risks & Unknowns:** The engine is only as safe as its ingredient-to-allergen mapping. Imported recipes (FEAT-10) carry arbitrary free text such as "stock", "pesto" and "Worcestershire sauce" whose allergen content is implicit. The landscape has no dedicated allergen-taxonomy option, only content vendors' own taxonomies, and vendor lock-in to an ingredient taxonomy is flagged for Spoonacular and Edamam. Under-matching breaks the "zero allergy incidents, ever" metric (success-metrics.md, cited in FEAT-23 ## Non-Functional Notes). Over-matching and strict quantity/unit completeness could exclude so many recipes that households with several allergies get repetitive or partial plans (FEAT-03.SPEC-010 ## Edge Cases — very small pools). Caching verdicts creates invalidation risk: any rule change, recipe edit or open safety report has to invalidate cached verdicts, or a stale "pass" could leak through.

**Spike Recommendation:** Build an ingredient-to-allergen classifier prototype and run it over a sample of starter-source recipes and pasted-link recipes (for example, 200 recipes across target cuisines). Measure (a) recall against a hand-labeled allergen truth set, (b) the share of recipes excluded by the FEAT-02.SPEC-007 completeness bar, and (c) per-candidate check latency. The result tells the build team whether a vendor taxonomy is enough, a curated mapping is needed, or both are layered, and whether the completeness rule needs a product-level carve-out for "to taste" style ingredients.

### FEAT-03 — AI Weekly Dinner Plan Generation

**Verdict:** Research-spike recommended — the resolvable unknown is whether a managed LLM, given the household's constraints and verified candidate pool (FEAT-03.SPEC-010 ## Data Exchanged), reliably returns a full seven-night selection that passes FEAT-02, fits the budget rule (FEAT-03.SPEC-006) and schedule, in well under a minute (ASMP-23), at a per-household token cost inside the pre-revenue budget (SC-16, as cited in FEAT-04 ## Non-Functional Notes). The specs also define partial-result and failure outcomes whose frequency is unknown (FEAT-03.SPEC-010 ## Edge Cases).

**Required Capabilities:**
- AI plan-generation integration that sends constraints, aggregated ratings, pantry items and the candidate pool, and receives candidate selections plus a coverage signal (FEAT-03.SPEC-010 ## Data Exchanged)
- Scheduled per-household generation at the organiser's chosen arrival day and time, plus immediate first-plan generation on upgrade (FEAT-03.SPEC-003 ## Trigger Definition; FEAT-03.SPEC-004)
- Week-start auto-adoption and once-per-week organiser approval (FEAT-03.SPEC-005; FEAT-03.SPEC-008; XBR-07)
- Budget-fit and household-scaled quantity computations (FEAT-03.SPEC-006, FEAT-03.SPEC-007); tier gating (FEAT-03.SPEC-009)
- Live plan propagation across household devices (FEAT-03.SPEC-011 ## Degradation Behavior)
- Concurrency: only one generation run in flight per household and week (FEAT-03.SPEC-010 ## Edge Cases); approval, auto-adoption, swaps and safety removals all write the plan with reject-with-refresh per slot (feature-dependency-map.md, Weekly Plan **Contention:**); late responses after a failure are discarded (FEAT-03.SPEC-010 ## Edge Cases)
- Offline/degraded: an already-generated plan stays viewable offline and approving needs a reconnect (FEAT-03.SPEC-011 ## Degradation Behavior; ASMP-25). An AI outage keeps the prior week visible with a Retry banner, and slow responses show "This is taking longer than usual" (FEAT-03.SPEC-010 ## Degradation Behavior)
- Scale: one generation per paid household per week, several thousand households, one archived week per household per week kept indefinitely; generation well under a minute; sync in the near-instant window (feature-overview.md ## Non-Functional Notes; ASMP-22, ASMP-23, ASMP-24)

**Candidate Approaches:** The generation call can target Anthropic Claude API, OpenAI GPT API or Google Gemini API (AI & Intelligent Behavior). All three offer structured output. Claude's prompt caching and OpenAI's Batch API (50% cheaper, for the non-interactive scheduled run) are the cost levers that differ, and Gemini Flash is the lowest listed per-token price. A Self-hosted open-weight model is in the landscape but is characterized there as disproportionate to "one weekly plan plus a few swaps". A hybrid is also possible within the same options: deterministic pre-filtering (safety, schedule, budget) shrinks the pool before the LLM arranges it, reducing tokens and failure modes. Scheduling options are Inngest, Trigger.dev (durable, step-based retries), Upstash QStash (HTTP delivery at a set time) or BullMQ (Background Jobs & Scheduling). Live propagation options are Supabase Realtime, Ably, Pusher (Channels) or Socket.io (self-hosted) (Real-time & Collaboration); Ably's ordering guarantees and Pusher's lighter channel model are the differentiators for a 2–6-member household channel.

**Risks & Unknowns:** Constraint-satisfaction quality: LLMs can propose recipes outside the supplied pool, skip nights or ignore budget. The spec covers this by discarding out-of-pool candidates and treating short results as partial (FEAT-03.SPEC-010 ## Edge Cases), but how often that happens drives the failure rate households see. The token budget grows with pool size, because the full candidate pool with ingredient lists is sent on every attempt (FEAT-03.SPEC-010 ## Data Exchanged), so imported recipes (FEAT-10) raise cost per generation. Scheduled bursts: default arrival slots (Sunday evening, platform-parameters.md `plan-arrival-time-slots`) cluster thousands of generations into a few time slots, which exposes provider rate limits. Dietary constraints leave the product (per member, without identifying detail), so the provider's data-retention terms fall under children's-privacy-class handling (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** Run a bounded generation harness with 20–30 synthetic households spanning multiple allergies, vegetarian members, tight budgets and small pools, against two or three of the landscape's LLM options. Measure the full-week success rate after FEAT-02 filtering, p95 latency, tokens per generation and the resulting monthly cost at several thousand paid households, plus behavior under Sunday-evening burst concurrency. That tells the Architect which provider and prompt structure (full-pool vs pre-filtered) meets ASMP-23 and SC-16, and whether a deterministic fallback selector is needed.

### FEAT-04 — One-Tap Meal Swap

**Verdict:** Standard-with-integration — swap alternatives come from the external AI capability with a fully specified degradation contract (FEAT-04.SPEC-007 ## Degradation Behavior). The per-slot lock with first-to-acquire-wins and guaranteed release (FEAT-04.SPEC-009 ## Business Rules) and the suggestion lifecycle (FEAT-04.SPEC-010) are conventional transactional patterns.

**Required Capabilities:**
- AI swap-alternatives generation, filtered through FEAT-02 and ranked with scarcity explanations (FEAT-04.SPEC-007; FEAT-04.SPEC-008)
- Apply a swap atomically: lock, safety re-check, write, grocery-list recalculation and live propagation (FEAT-04.SPEC-004; XBR-03)
- Other adults suggest and the organiser accepts or declines; suggestions lapse when the night passes (FEAT-04.SPEC-002, FEAT-04.SPEC-003, FEAT-04.SPEC-005; XBR-06); in-app suggestion notices (FEAT-04.SPEC-006 ## Channels)
- Concurrency: one active swap per slot, and a second attempt is refused outright, not queued (FEAT-04.SPEC-009 ## Business Rules). Swap Suggestion resolves first-decision-wins, and a direct swap supersedes an open suggestion (feature-dependency-map.md, Swap Suggestion **Contention:**)
- Offline/degraded: an AI outage shows a Retry with the original meal unchanged, and slow responses show a note after 8 seconds (FEAT-04.SPEC-007 ## Degradation Behavior); no half-created suggestion or meal is ever written
- Scale: a few swaps per household per week; tap-to-updated-plan-and-list under 10 seconds; alternatives within a couple of seconds (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Alternatives can come from the same AI & Intelligent Behavior option as FEAT-03 (Anthropic Claude API, OpenAI GPT API, Google Gemini API), where lower-latency model tiers such as Gemini Flash matter more here than in weekly generation. Alternatively, a deterministic ranking over the already-verified candidate pool can serve alternatives without an LLM call, keeping the AI path for explanation or ranking only. The per-slot lock can be a database row lock or conditional write in any Database option, or a short-lived key in Upstash Redis or Redis Cloud (Caching & Performance). Lapse scheduling fits Inngest, Trigger.dev, Upstash QStash or BullMQ (Background Jobs & Scheduling). Propagation reuses the Real-time & Collaboration choice (Supabase Realtime, Ably, Pusher (Channels), Socket.io (self-hosted)).

**Risks & Unknowns:** The under-10-second budget chains an external LLM call, a safety check, a write, list recalculation and cross-device propagation. LLM tail latency alone can consume most of it (FEAT-04.SPEC-007 ## Degradation Behavior already anticipates more than 8 seconds). A lock held in a cache store can outlive a crashed request unless it has a TTL, which conflicts with "never left held" (FEAT-04.SPEC-009 ## Business Rules). Lapse timing depends on the household's local time zone and daylight-saving transitions (FEAT-04.SPEC-005 ## Edge Cases).

**Spike Recommendation:** None — the FEAT-03 spike's latency measurements can include a swap-alternatives prompt to settle whether the couple-of-seconds target needs the deterministic path.

### FEAT-05 — Pantry-Aware Suggestions

**Verdict:** Straightforward — a short, per-household list with exact-name duplicate merge (FEAT-05.SPEC-003) and exact-name recipe matching for callouts and weighting (FEAT-05.SPEC-006 ## Business Rules). Weighting is only an input to FEAT-03's generation (FEAT-05.SPEC-005), and no new external service is involved.

**Required Capabilities:**
- Pantry item entry, clear and "used it up?" prompts (FEAT-05.SPEC-001; FEAT-05.SPEC-002)
- Exact-name matching between pantry items and recipe ingredients for plan callouts and paid-tier weighting (FEAT-05.SPEC-006; FEAT-05.SPEC-005)
- Pantry items excluded from the grocery list on both tiers (FEAT-05.SPEC-007; XBR-04)
- Concurrency: Maya and Sam add and clear items concurrently, including offline; the same item added twice merges and clearing is idempotent (FEAT-05.SPEC-003 ## Edge Cases; feature-dependency-map.md, Pantry Item **Contention:**)
- Offline/degraded: adds are queued locally and synced on reconnect (feature-overview.md ## Non-Functional Notes; FEAT-05.SPEC-001 ## States; ASMP-25)
- Scale: short lists with no fixed cap, several thousand households; adding confirms instantly (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Offline queuing can share the Service-worker / IndexedDB client-side caching implementation built for FEAT-06 (Caching & Performance). Sync can ride the same Real-time & Collaboration channel (Supabase Realtime, Ably, Pusher (Channels), Socket.io (self-hosted)) or plain request/response with TanStack Query + Zustand or Redux Toolkit + RTK Query refetch (State Management), since pantry edits have no 2-second propagation target. Matching is a relational query in any Database option.

**Risks & Unknowns:** Exact-name matching ("spinach" does not match "baby spinach", FEAT-05.SPEC-006 ## Business Rules) is simple to build but may under-deliver the "uses the spinach and feta you already have" promise. That is a product-quality trade-off, not a feasibility blocker.

**Spike Recommendation:** None

### FEAT-06 — Shared Grocery List

**Verdict:** Hard — the list must show a tick or add on another device within 2 seconds (feature-overview.md ## Non-Functional Notes) and stay fully editable offline. Queued changes then reconcile with idempotent ticks, last-write-wins quantity edits by original event time, deletion precedence and duplicate-line merge, all while plan-driven recalculation rewrites plan-derived lines concurrently (FEAT-06.SPEC-005 ## Degradation Behavior; FEAT-06.SPEC-008 ## Cross-Field Rules, ## Edge Cases; feature-dependency-map.md, Grocery List Item **Contention:** "High").

**Required Capabilities:**
- Live list sync across household devices with optimistic local updates (FEAT-06.SPEC-005 ## Degradation Behavior)
- List generation and recalculation from the current plan, with ingredient consolidation and household-scaled quantities, preserving manual items, ticks and "already have it" marks (FEAT-06.SPEC-002; FEAT-06.SPEC-006; XBR-03)
- "Already have it" marks that also add to the pantry (FEAT-06.SPEC-003; XBR-04); week rollover and carry-over (FEAT-06.SPEC-004); manual-item validation and merge (FEAT-06.SPEC-007); access rules (FEAT-06.SPEC-009)
- Aisle grouping and units per household locale (XBR-11, owned by FEAT-16)
- Concurrency: multiple members plus recalculation edit simultaneously. Ticks are idempotent, quantity edits last-write-wins by event time with arrival-order tiebreak, deletion wins over queued edits, and a removed member's queued changes are discarded (FEAT-06.SPEC-008 ## Cross-Field Rules, ## Business Rules, ## Edge Cases)
- Offline/degraded: a persistent offline banner, with every read and write remaining usable and queued for sync (FEAT-06.SPEC-005 ## Degradation Behavior; ASMP-25)
- Scale: several thousand households, one active list each plus retained history; sub-2-second propagation; recalculated items within a couple of seconds (feature-overview.md ## Non-Functional Notes; ASMP-22, ASMP-24)

**Candidate Approaches:** Transport can be Supabase Realtime (Postgres change-data-capture, no separate service if Supabase is the database), Ably (guaranteed ordering, exactly-once delivery and connection recovery, closest to the event-ordering rules), Pusher (Channels) (lighter household-scoped channels; the merge logic is entirely the team's) or Socket.io (self-hosted) (full control over the reconciliation protocol at the cost of owning scaling and reconnection), all in Real-time & Collaboration. The offline queue and local replica map to Service-worker / IndexedDB client-side caching (Caching & Performance), with server-state caching in TanStack Query + Zustand or Redux Toolkit + RTK Query (State Management). Server-side merge rules live in the Backend / API Layer option against any Database option. Hot-path reads can be cached in Upstash Redis or Redis Cloud. The landscape has no dedicated CRDT or local-first sync engine, so these merge semantics are custom application logic whichever transport is selected.

**Risks & Unknowns:** Last-write-wins "by original event time" (FEAT-06.SPEC-008 ## Cross-Field Rules) depends on client clocks, and skewed device clocks can make an older edit win. The spec's arrival-order tiebreak only covers exact ties. Recalculation racing queued offline ticks on lines whose identity changes (FEAT-06.SPEC-008 ## Edge Cases) needs stable line identity across recalculation. Supabase Realtime's documented household-scale connection figures (~200 concurrent on Pro) and Pusher's free-tier limits need checking against in-store usage peaks such as weekend shopping. Service-worker behavior differs across mobile browsers, especially iOS Safari background sync, which affects how reliably "changes sync later" works (ASMP-25).

**Spike Recommendation:** Prototype the offline queue plus the FEAT-06.SPEC-008 merge rules on one or two candidate transports, for example Ably vs Supabase Realtime. Script two devices through offline tick/edit/delete sequences, clock skew, and a concurrent recalculation. The spike answers whether the rules converge with no duplicates on the target mobile browsers and whether server-assigned sequencing needs to replace client event times. That decides the Real-time & Collaboration selection and the sync protocol shape.

### FEAT-07 — Weekly Plan Ready Notification

**Verdict:** Standard-with-integration — delivery depends on two external capabilities, device push (FEAT-07.SPEC-005) and email fallback (FEAT-07.SPEC-006), each with a queued/no-error degradation contract (## Degradation Behavior in both). The at-most-once-per-household-week rule is a conventional idempotency key (FEAT-07.SPEC-002 ## Delivery Rules).

**Required Capabilities:**
- Event-driven dispatch on generation completion (FEAT-07.SPEC-001 ## Trigger Definition) with per-member channel resolution and eligibility (FEAT-07.SPEC-003) and organiser-set arrival day/time (FEAT-07.SPEC-004)
- Web push to a responsive web app, with no native apps (FEAT-07.SPEC-005 ## Scope and Non-Goals, SC-05); email fallback (FEAT-07.SPEC-006)
- Deduplication and expiry: one message per member per household-week, and offline-device messages are discarded when the next week's message is due (FEAT-07.SPEC-002 ## Delivery Rules)
- Concurrency: duplicate or retried completion signals must never double-send (FEAT-07.SPEC-002 ## Delivery Rules); preference edits racing dispatch resolve on the current preference (FEAT-07.SPEC-003)
- Offline/degraded: offline devices get queued push on reconnect. A push outage drops to the email fallback, and an email outage sends nothing and shows no error, because the plan's in-app availability never depends on delivery (FEAT-07.SPEC-005, FEAT-07.SPEC-006 ## Degradation Behavior; XBR-12)
- Scale: at most one message per household per week to 2–6 members across several thousand households; at least 80% delivered within one minute of completion (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Push can go through OneSignal (fastest self-serve Web Push setup) or Courier (one API over push and email with routing, adding an abstraction layer), and email through Resend, Postmark or SendGrid, all in Email & Messaging Delivery. A Courier-style unified layer handles channel fallback centrally, whereas OneSignal + Resend/Postmark keeps fallback logic in the app. Dispatch jobs fit Inngest, Trigger.dev, Upstash QStash or BullMQ (Background Jobs & Scheduling).

**Risks & Unknowns:** Web Push reach on iPhones depends on the user installing the web app to the home screen, and without that, iOS members fall to email. That could put the 80%-within-a-minute target and the "at least half opened same day" metric at risk (feature-overview.md ## Non-Functional Notes). Clustered Sunday-evening arrival slots concentrate dispatch bursts (platform-parameters.md `plan-arrival-time-slots`), so provider throughput and rate limits matter.

**Spike Recommendation:** None

### FEAT-08 — Recipe Library (Starter Recipes)

**Verdict:** Standard-with-integration — browse and search is a simple filter (FEAT-08.SPEC-001; profile Section 3 Search, simple filter), but the library's existence depends on an external recipe/food-content source delivering ingredient-complete content in batches, with incomplete batches held out (FEAT-08.SPEC-004 ## Data Exchanged, ## Edge Cases).

**Required Capabilities:**
- Search by name or ingredient and filter by dietary badge, with ineligible-recipe disclosure (FEAT-08.SPEC-001; FEAT-08.SPEC-003)
- Recipe detail with locale-converted quantities and safety badge (FEAT-08.SPEC-002; XBR-11; FEAT-02.SPEC-008)
- Seeding and maintenance pipeline: new content, corrections, retirements, and a completeness gate before content becomes Active (FEAT-08.SPEC-004 ## Data Exchanged, ## Edge Cases)
- Concurrency: N/A for households, since starter recipes are read-only (feature-dependency-map.md, Recipe **Contention:**); duplicate corrections resolve most-recent-wins (FEAT-08.SPEC-004 ## Edge Cases)
- Offline/degraded: a content-source outage has no household-visible impact (FEAT-08.SPEC-004 ## Degradation Behavior); failed detail loads offer retry without losing search context (feature-overview.md ## Non-Functional Notes)
- Scale: one shared corpus sized to the Recipe Library Coverage at Launch metric; concurrent search from several thousand households with results in about a second (feature-overview.md ## Non-Functional Notes; ASMP-23)

**Candidate Approaches:** Content can come from Spoonacular Food API (~365,000 recipes with ingredient data), Edamam Recipe Search API (nutrition and allergen specialization), a Curated/licensed starter set + manual editorial seeding (highest control over the FEAT-02.SPEC-007 completeness bar), or Tasty/Yummly-class licensed content partnerships (negotiated terms), all in Recipe & Food Content Data. Search at this complexity is satisfiable by PostgreSQL full-text search. Meilisearch, Typesense or Algolia (Search) add typo tolerance at the cost of an index-sync pipeline and, for Algolia, higher cost.

**Risks & Unknowns:** Vendor licensing terms for storing and redisplaying recipe content (as opposed to live API lookups) are not captured in the landscape, and some API tiers restrict caching. The API tier costs listed ($149–$999/month) sit against the founder's sub-$100/month pre-revenue budget (SC-16, cited in FEAT-04 ## Non-Functional Notes). The share of vendor recipes that pass the quantity+unit completeness bar is unknown (see the FEAT-02 spike).

**Spike Recommendation:** None — covered by the FEAT-02 spike's completeness measurement on a starter-source sample; licensing terms are a Section 5 question.

### FEAT-09 — Household Invitations & Membership

**Verdict:** Straightforward — invitation create/resend/revoke, 14-day scheduled expiry, first-write-wins acceptance, organiser hand-over requiring acceptance and member departure are conventional state transitions (FEAT-09.SPEC-006, FEAT-09.SPEC-007 ## Edge Cases, FEAT-09.SPEC-009, FEAT-09.SPEC-010). All notifications are in-app (FEAT-09.SPEC-012..014 ## Channels).

**Required Capabilities:**
- Shareable invitation link, acceptance flow and Member Profile creation that triggers onboarding once (FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-09.SPEC-007; XBR-18)
- Organiser hand-over with acceptance so exactly one organiser exists (FEAT-09.SPEC-003, FEAT-09.SPEC-004, FEAT-09.SPEC-009; XBR-15); leave-household processing with rating anonymization (FEAT-09.SPEC-008; XBR-16)
- Scheduled 14-day invitation expiry (FEAT-09.SPEC-006)
- Live member-list update on acceptance (profile Section 3 Real-time, FEAT-01.SPEC-004)
- Concurrency: acceptance vs revocation vs expiry resolves reject-with-refresh, first recorded wins, and two simultaneous accepts yield one member (FEAT-09.SPEC-007 ## Edge Cases; feature-dependency-map.md, Invitation **Contention:**)
- Offline/degraded: a drafted invitation is held locally until it can be sent (feature-overview.md ## Non-Functional Notes; FEAT-09.SPEC-001 ## States)
- Scale: 1–12 members and multiple outstanding invitations per household; send confirms within a couple of seconds (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Invitation tokens and state live in any Database option, with transactional conditional updates for race resolution. Invitee sign-up flows through the Authentication & Identity option (Clerk, Supabase Auth, Auth0, Better Auth, NextAuth.js), where managed providers differ in how custom invitation tokens attach to sign-up. Expiry fits any Background Jobs & Scheduling option (Inngest, Trigger.dev, Upstash QStash, BullMQ) or a query-time expiry check with no job at all. The live member list reuses the Real-time & Collaboration option.

**Risks & Unknowns:** Hand-over interacts with account deletion and billing ownership. The Subscription is organiser-controlled (FEAT-14), so payment-method ownership across a hand-over has to be traced into the payment integration, and FEAT-09 specs do not cover it.

**Spike Recommendation:** None

### FEAT-10 — Recipe Import from Web Link

**Verdict:** Standard-with-integration — extraction is an external or self-hosted capability with a specified progress/failure contract that falls back to manual entry (FEAT-10.SPEC-005 ## Degradation Behavior). Review-before-save, duplicate detection, the 30-per-week rate limit and the safety re-check on save or edit are conventional (FEAT-10.SPEC-006, FEAT-10.SPEC-007, FEAT-10.SPEC-008).

**Required Capabilities:**
- Fetch and parse a pasted recipe URL into ingredients, steps and cook time for review (FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-005)
- Manual entry fallback and edit (FEAT-10.SPEC-003, FEAT-10.SPEC-004)
- Duplicate-link detection surfacing the existing recipe (FEAT-10.SPEC-006; XBR-19)
- Per-household rate limit of 30 imports per week with no lifetime cap (FEAT-10.SPEC-007)
- Safety re-check on every save or edit before plan eligibility (FEAT-10.SPEC-008; XBR-19)
- Concurrency: Maya and Sam editing the same imported recipe resolves reject-with-refresh on the second save (FEAT-10.SPEC-004; feature-dependency-map.md, Recipe **Contention:**)
- Offline/degraded: importing requires connectivity. An unreachable extractor shows a failure path with manual entry, and slow extraction keeps the progress indicator until timeout (FEAT-10.SPEC-001 ## States; FEAT-10.SPEC-005 ## Degradation Behavior)
- Scale: up to 30 imports per household per week; extraction in a few seconds; saved recipes searchable within about a second (feature-overview.md ## Non-Functional Notes; ASMP-23)

**Candidate Approaches:** Parsing options are recipe-scrapers (an open-source library that parses schema.org JSON-LD, Microdata and OpenGraph, Python-based) or a Custom JSON-LD/schema.org parser (self-built, in the backend language). A Managed recipe-extraction API (e.g., Apify recipe-scraper actors) moves site-specific maintenance to a marketplace vendor, and AI-fallback extraction (JSON-LD/Microdata/heuristic parsing + LLM fallback) covers pages without structured markup using the AI & Intelligent Behavior provider. All four are in Web Page Recipe Extraction. recipe-scrapers is Python while most Backend / API Layer candidates are Node, so that pairing may need a separate worker. Extraction jobs and timeouts fit any Background Jobs & Scheduling option. Rate-limit counters fit Upstash Redis or Redis Cloud, or a database counter.

**Risks & Unknowns:** Legality of importing other sites' content is an unresolved product and legal question that gates v1 (ASMP-36; BRIEF.md Open Questions). Server-side fetching of arbitrary user-supplied URLs is a server-side request forgery surface that needs egress restrictions. Many sites block scrapers or render recipes client-side. Extracted ingredient lines often lack a quantity or unit ("salt to taste"), which the FEAT-02.SPEC-007 fail-closed completeness bar excludes, so a successfully imported recipe may never be plan-eligible without manual edits. The AI-fallback option sends third-party page content to an LLM provider, which adds per-import cost variance.

**Spike Recommendation:** Run a bounded extraction test on about 100 links from popular US and UK recipe sites with one self-built/library parser and one managed option. Measure the extraction success rate, the share that pass FEAT-02.SPEC-007 without edits, and median latency against the "few seconds" target. The result tells the team whether an AI-fallback tier is needed and how often the review screen must prompt for missing quantities.

### FEAT-11 — Leftover Rollover to Lunches

**Verdict:** Straightforward — leftover-lunch suggestions are a deterministic pass over the newly generated plan: one-day default, two-day fallback, no suggestion past the XBR-10 ceiling (FEAT-11.SPEC-002 ## Processing Logic; FEAT-11.SPEC-003), with withdrawal when the source dinner changes (FEAT-11.SPEC-004).

**Required Capabilities:**
- Post-generation computation linking each leftover lunch to exactly one source dinner (FEAT-11.SPEC-002; FEAT-11.SPEC-003; XBR-10)
- Update or withdrawal on source swap or removal (FEAT-11.SPEC-004); eaten/skipped marking on the leftover card (FEAT-11.SPEC-001)
- Concurrency: Maya and Sam marking a leftover lunch eaten or skipped resolves last-write-wins, and a source swap racing a mark resolves via withdrawal (FEAT-11.SPEC-001 ## Edge Cases; feature-dependency-map.md, Planned Meal **Contention:**)
- Offline/degraded: an already-generated suggestion stays viewable offline, and a computation failure omits the suggestion silently (feature-overview.md ## Non-Functional Notes)
- Scale: at most one leftover lunch per eligible dinner per week per household; no separate loading step (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Runs as a step inside the generation workflow in any Background Jobs & Scheduling option (Inngest and Trigger.dev model it as a durable step; BullMQ or Upstash QStash as a chained job), or synchronously in the Backend / API Layer option after generation completes. Offline viewing reuses the Service-worker / IndexedDB client-side caching layer.

**Risks & Unknowns:** None identified — no external dependency or scale demand; correctness depends on FEAT-04 and FEAT-23 reliably emitting source-change events.

**Spike Recommendation:** None

### FEAT-12 — Meal Rating & Preference Learning

**Verdict:** Straightforward — rating capture is simple CRUD with proxy rules (FEAT-12.SPEC-002). "Learning" is a counted-threshold update to a soft dislike (FEAT-12.SPEC-005 ## Processing Logic) plus per-recipe aggregated weighting passed into FEAT-03's generation (FEAT-12.SPEC-004 ## Business Rules), so no model training or new external service is specified.

**Required Capabilities:**
- Post-dinner thumbs up/down prompt, with adults recording young kids' ratings by proxy (FEAT-12.SPEC-001; FEAT-12.SPEC-002)
- Repeated-dislike threshold (`repeated-dislike-rating-count`) that creates a learned soft dislike, tier-gated (FEAT-12.SPEC-005 ## Processing Logic; XBR-17)
- Aggregated-per-recipe weighting input for paid-tier generation that never overrides safety (FEAT-12.SPEC-004 ## Business Rules; FEAT-03.SPEC-010 ## Data Exchanged)
- Authorization so individual ratings are never shown broken out (FEAT-12.SPEC-003)
- Concurrency: two adults recording the same kid's rating resolves last-write-wins, and learned dislikes merge with explicit rules without overwriting them (feature-dependency-map.md, Rating and Dietary Rule **Contention:**)
- Offline/degraded: a failed submission retries automatically in the background (feature-overview.md ## Non-Functional Notes; FEAT-12.SPEC-001 ## States)
- Scale: one rating per member per planned meal across several thousand households, kept for life (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Ratings and threshold counts live in any Database option. The learned-update trigger runs inline in the Backend / API Layer option or as an event in Inngest, Trigger.dev or BullMQ (Background Jobs & Scheduling). The weighting's effect on plans depends on the AI & Intelligent Behavior option selected for FEAT-03 (Anthropic Claude API, OpenAI GPT API, Google Gemini API). Background retry of submissions maps to the Service-worker / IndexedDB client-side caching queue.

**Risks & Unknowns:** Whether an LLM actually honors "weight toward highly rated" inputs measurably is part of the FEAT-03 spike's unknown. Rating deletion vs anonymization on member removal or departure (XBR-16) needs careful data modeling. Young-kid ratings are children's data (feature-overview.md ## Non-Functional Notes), and aggregated ratings leave the product in generation requests.

**Spike Recommendation:** None

### FEAT-13 — Tonight's Dinner Reminder

**Verdict:** Standard-with-integration — the nudge is delivered by push through the shared device-notification integration (FEAT-13.SPEC-002 ## Channels, via FEAT-07.SPEC-005), triggered daily per household at 4:00 pm local time (FEAT-13.SPEC-001 ## Trigger Definition; platform-parameters.md `nightly-nudge-send-time`), with at most one same-day correction (FEAT-13.SPEC-003, FEAT-13.SPEC-004; XBR-09).

**Required Capabilities:**
- Daily scheduled trigger evaluated in each household's local time zone (FEAT-13.SPEC-001 ## Trigger Definition, ## Edge Cases)
- Eligibility and channel resolution per member; kids and the operator are never recipients (FEAT-13.SPEC-005; XBR-13)
- Prep-reminder derivation from recipe prep requirements (FEAT-13.SPEC-006)
- Same-day correction after a swap, sent at most once (FEAT-13.SPEC-003; XBR-09)
- Concurrency: a swap landing while the nudge is dispatching needs a deterministic "sent before or after" cut so the correction fires at most once (FEAT-13.SPEC-003 ## Edge Cases)
- Offline/degraded: push to offline devices queues per the FEAT-07.SPEC-005 contract (FEAT-13.SPEC-002 ## Delivery Rules)
- Scale: at most one nudge and one correction per household per day across several thousand households; near-instant delivery at the chosen time (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Per-household local-time firing can be a timezone-bucketed cron fan-out or per-household scheduled events in Inngest, Trigger.dev, Upstash QStash or BullMQ (Background Jobs & Scheduling). QStash and BullMQ differ mainly in per-message pricing vs flat Redis cost at several thousand daily messages. Delivery goes through the Email & Messaging Delivery push option shared with FEAT-07 (OneSignal or Courier).

**Risks & Unknowns:** The same iOS Web Push install dependency as FEAT-07 applies. Daily volume (several thousand households × 365 days) can exceed free tiers of per-run-priced job platforms (Inngest 25,000 runs/month, QStash 500 messages/day), which affects cost. Daylight-saving transitions must not double-fire or skip the nudge.

**Spike Recommendation:** None

### FEAT-14 — Subscription & Billing Management

**Verdict:** Standard-with-integration — the paid subscription depends on an external payment processor that emits upgrade, renewal and retry outcome events (FEAT-14.SPEC-009 ## Inbound Events) under a slow/down/rejects contract (## Degradation Behavior). Grace-period and refund logic (FEAT-14.SPEC-005, FEAT-14.SPEC-007) is established subscription-billing work.

**Required Capabilities:**
- Tier overview, upgrade (monthly/yearly), billing and payment management, downgrade or cancel at period end (FEAT-14.SPEC-001..004)
- Inbound processor events that drive billing_state, including a 7-day grace window after a failed renewal (FEAT-14.SPEC-009 ## Inbound Events; FEAT-14.SPEC-007; FEAT-14.SPEC-005)
- Applying subscription changes that switch tier gating for FEAT-03, FEAT-05 and FEAT-12 (FEAT-14.SPEC-008; XBR-05)
- In-app plus email billing confirmations and grace notices (FEAT-14.SPEC-010, FEAT-14.SPEC-011; FEAT-14.SPEC-012 ## Degradation Behavior)
- Billing amounts in the household currency, at least USD and GBP (feature-overview.md ## Non-Functional Notes; ASMP-28)
- Concurrency: organiser changes racing processor events resolve reject-with-refresh against stale billing state (feature-dependency-map.md, Subscription **Contention:**; FEAT-14.SPEC-005 and FEAT-14.SPEC-007 ## Edge Cases)
- Offline/degraded: viewing the tier works offline and changes need connectivity. A processor outage disables Subscribe with a message and queues grace retries without shortening the window (FEAT-14.SPEC-009 ## Degradation Behavior)
- Scale: one subscription per household, several thousand households; upgrade confirms within a few seconds (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Stripe Billing provides payment and subscription primitives, with the team building the grace-period and refund logic. Paddle acts as merchant of record and absorbs US sales-tax and UK VAT handling at a higher per-transaction rate. Chargebee or Recurly on top of Stripe express dunning and grace rules as configuration. All four are in Payments & Billing. Webhook ingestion runs in the Backend / API Layer option, with durable processing in Inngest, Trigger.dev or BullMQ (Background Jobs & Scheduling). Email delivery maps to Resend, Postmark or SendGrid.

**Risks & Unknowns:** Webhook delivery is at-least-once and can arrive out of order, so billing-state transitions have to be idempotent and ordering-tolerant (FEAT-14.SPEC-009 ## Inbound Events). The spec's 7-day grace window has to be reconciled with the processor's own retry schedule, since the two can diverge. Selling in two tax jurisdictions affects the merchant-of-record vs processor choice. The founder's three-month revenue timeline (ASMP-33) favors lower-integration options.

**Spike Recommendation:** None

### FEAT-15 — Member Onboarding

**Verdict:** Straightforward — a once-per-acceptance routing automation (FEAT-15.SPEC-002, FEAT-15.SPEC-003) to a landing view that reuses existing plan and list surfaces (FEAT-15.SPEC-001), with no stored records of its own (feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Route a newly accepted member to the plan and list together, with an explained empty state (FEAT-15.SPEC-001; FEAT-15.SPEC-002 ## Edge Cases)
- Once-only rule per acceptance; a re-invited former member onboards again (FEAT-15.SPEC-003; XBR-18)
- Signal emission (member_onboarding_started/completed/shown_empty_household; feature-overview.md ## Non-Functional Notes)
- Concurrency: simultaneous acceptances by different invitees run independently, and reloads do not re-trigger (FEAT-15.SPEC-002 ## Edge Cases)
- Offline/degraded: the landing inherits the plan and list offline behavior (FEAT-15.SPEC-001 ## States; ASMP-25)
- Scale: N/A — single per-member event with no growing dataset (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Routing lives in the Frontend Framework option (Next.js, Nuxt 3, SvelteKit, Remix, Astro). Signals go to PostHog, Mixpanel or Amplitude (Analytics & Product Telemetry), where PostHog's self-host option matters if household-linked events must stay in-house.

**Risks & Unknowns:** None identified — no external dependency; correctness depends on FEAT-09.SPEC-007's single "acceptance succeeded" signal.

**Spike Recommendation:** None

### FEAT-16 — Units, Currency & Locale Configuration

**Verdict:** Straightforward — conversion uses fixed factors (1 cup → 240 ml, 1 oz → 28 g, and so on) at display time and never rewrites stored values. Currency is a display label with no exchange-rate conversion (FEAT-16.SPEC-004 ## Cross-Field Rules, ## Scope and Non-Goals).

**Required Capabilities:**
- Household unit system, currency and aisle-name settings with validation and defaults (FEAT-16.SPEC-001, FEAT-16.SPEC-002, FEAT-16.SPEC-003)
- Display-time conversion applied consistently across plan, recipes, list, budget and check-in (FEAT-16.SPEC-004; XBR-11)
- Concurrency: Maya on two devices resolves last-write-wins per setting, and a failed save keeps the prior value (feature-dependency-map.md, Household **Contention:**)
- Offline/degraded: settings screens follow the product's standard offline messaging (FEAT-16.SPEC-001 ## States); conversion is local and needs no connectivity
- Scale: negligible — one locale record per household (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Formatting options are next-intl (Next.js-native), react-i18next, FormatJS (react-intl) (ICU currency/number formatting) or LinguiJS (Internationalization). The fixed conversion table is plain application code shared by client and server.

**Risks & Unknowns:** Fixed volume-to-volume and weight-to-weight factors are specified, but the specs define no volume↔weight conversion (cups to grams for solids). Recipes authored in cups will show millilitres, not grams, under the metric setting. This is a product expectation point, not a build risk.

**Spike Recommendation:** None

### FEAT-17 — Older-Kid Dinner Voting

**Verdict:** Straightforward — rounds of 2–3 safety-validated options with a deterministic tally, unanimous or split resolution, and a fallback at the resolution point (FEAT-17.SPEC-004, FEAT-17.SPEC-005 ## Processing Logic). The only new platform demand is an older-kid limited-login role (FEAT-17.SPEC-006 ## Field Validation Rules).

**Required Capabilities:**
- Voting round setup with options checked by FEAT-02 (FEAT-17.SPEC-002; FEAT-17.SPEC-004; XBR-01)
- Vote casting by older-kid limited logins, tally and resolution (FEAT-17.SPEC-001, FEAT-17.SPEC-003, FEAT-17.SPEC-005)
- A limited-login identity for minors holding children's-privacy-class data (feature-overview.md ## Non-Functional Notes; ASMP-27)
- Concurrency: a vote cast after Maya resolves the round is rejected with refresh (feature-dependency-map.md, Dinner Vote **Contention:**; FEAT-17.SPEC-003 ## Edge Cases)
- Offline/degraded: failed vote submissions retry automatically (feature-overview.md ## Non-Functional Notes; FEAT-17.SPEC-001 ## States)
- Scale: trivial — one open round per night, one vote per older kid (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** The limited-login role can be modeled in any Authentication & Identity option. Clerk and Auth0 support custom roles and restricted sessions as managed features. Supabase Auth pairs roles with row-level security. Better Auth and NextAuth.js leave the kid-session model entirely to the team. Tally and resolution are Backend / API Layer logic; the resolution-point fallback fits any Background Jobs & Scheduling option.

**Risks & Unknowns:** Accounts for minors raise the children's-privacy bar (ASMP-27). Whether a managed identity provider may hold minors' credentials under its terms, and what consent the Later-phase login requires, are open (BRIEF.md Open Questions on kids' representation).

**Spike Recommendation:** None

### FEAT-18 — Account & Data Management

**Verdict:** Standard-with-integration — export generation produces a downloadable file that needs object storage and an email or in-app ready signal (FEAT-18.SPEC-006 ## Processing Logic; FEAT-18.SPEC-012 ## Degradation Behavior). Household deletion is a durable cascade that must finish within 30 days and reach external services such as the calendar disconnect (FEAT-18.SPEC-008 ## Processing Logic step 5).

**Required Capabilities:**
- Export compiling every household record into one readable file, with automatic retry, a download link and a ready notification (FEAT-18.SPEC-001, FEAT-18.SPEC-006, FEAT-18.SPEC-013)
- Member removal, household deletion and own-account deletion cascades (FEAT-18.SPEC-007, FEAT-18.SPEC-008, FEAT-18.SPEC-009; XBR-15, XBR-16), including immediate sign-out of every member (FEAT-18.SPEC-008 step 3)
- Contact support with an email acknowledgement (FEAT-18.SPEC-005; FEAT-18.SPEC-015 ## Channels — email)
- Transactional email for export-ready, deletion-completed and support acknowledgement (FEAT-18.SPEC-012)
- Concurrency: removal racing a member's own edits or offline queue resolves in favor of removal (FEAT-18.SPEC-007 ## Edge Cases; FEAT-06.SPEC-008 ## Edge Cases), and deletion supersedes in-flight edits (feature-dependency-map.md, Household **Contention:**)
- Offline/degraded: email outages leave in-app states unaffected and queue sends (FEAT-18.SPEC-012 ## Degradation Behavior); export and deletion show progress, never an indefinite wait (feature-overview.md ## Non-Functional Notes)
- Scale: exports compile years of history per household and stay reasonably fast; deletion completes within 30 days (feature-overview.md ## Non-Functional Notes; ASMP-24, ASMP-27)

**Candidate Approaches:** Export files can go in Cloudflare R2 (zero egress), Amazon S3 (lifecycle policies for auto-expiry), Supabase Storage (bundled if Supabase is the database) or Backblaze B2 (File & Object Storage). Multi-step export and cascade processing fits durable-step platforms (Inngest, Trigger.dev) or BullMQ workers (Background Jobs & Scheduling). Trigger.dev's self-host option bears on children's-data residency. Emails go through Resend, Postmark or SendGrid.

**Risks & Unknowns:** "Deleted within 30 days" has to reach copies held outside the primary database: processor-held payment data (FEAT-14), push subscriptions, email-provider logs, analytics events, cached verdicts, database backups and previously generated export files. None of these is enumerated in FEAT-18.SPEC-008. The export file concentrates the most sensitive data, including kids' allergy data (feature-overview.md ## Non-Functional Notes), so download links need expiry and access control. Session revocation within a short window depends on the Authentication & Identity option's revocation support.

**Spike Recommendation:** None

### FEAT-19 — Weekly Plan History

**Verdict:** Straightforward — history browse and detail are paginated reads over retained Weekly Plan and Grocery List records (FEAT-19.SPEC-001, FEAT-19.SPEC-002). Reuse re-runs the shared safety check per archived meal against today's rules (FEAT-19.SPEC-003 ## Processing Logic).

**Required Capabilities:**
- Browse past weeks without a depth cap; past-week detail with plan and list (FEAT-19.SPEC-001, FEAT-19.SPEC-002)
- Reuse a past week into a future week with a per-meal safety re-check (FEAT-19.SPEC-003; XBR-01); access and reuse authorization (FEAT-19.SPEC-004)
- Concurrency: reuse writes into a future week follow the Weekly Plan reject-with-refresh rule (feature-dependency-map.md, Weekly Plan **Contention:**; FEAT-19.SPEC-001 ## Edge Cases)
- Offline/degraded: history screens show standard offline messaging; reuse needs connectivity (FEAT-19.SPEC-001 ## States)
- Scale: years of weekly history per household, with past weeks loading within a couple of seconds as history grows (feature-overview.md ## Non-Functional Notes; ASMP-23, ASMP-24)

**Candidate Approaches:** Indexed, household-scoped pagination in any Database option (Neon, Supabase, Amazon RDS for PostgreSQL, PlanetScale for Postgres, self-hosted PostgreSQL) through the ORM / Data Access option (Prisma ORM, Drizzle ORM, TypeORM). Client-side caching of browsed weeks via TanStack Query + Zustand or Redux Toolkit + RTK Query.

**Risks & Unknowns:** None identified — volumes are per-household and small (about 52 weeks a year).

**Spike Recommendation:** None

### FEAT-20 — Online Grocery Ordering Handoff

**Verdict:** Research-spike recommended — the resolvable unknown is which grocery-ordering partners can actually receive a list payload and return acceptance and item-availability for both launch markets. Instacart Connect / IDP covers the US only, and no self-service Tesco or UK grocer API was found, so the landscape flags UK coverage as a research gap (FEAT-20.SPEC-002 ## Data Exchanged; technology-landscape.md Online Grocery Ordering Integration).

**Required Capabilities:**
- Snapshot of unticked items sent to an external ordering capability, with acceptance/rejection and availability responses handled (FEAT-20.SPEC-002 ## Data Exchanged, ## Edge Cases)
- Eligibility and authorization rules for handoff (FEAT-20.SPEC-003)
- Signals grocery_handoff_initiated, grocery_handoff_succeeded and grocery_handoff_failed (feature-overview.md ## Non-Functional Notes)
- Concurrency: duplicate or out-of-order response events for one attempt are ignored after the first, and responses for abandoned attempts are discarded (FEAT-20.SPEC-002 ## Edge Cases)
- Offline/degraded: when the capability is down, nothing is sent and the list stays unchanged with Retry; slow responses show a "Still working" note (FEAT-20.SPEC-002 ## Degradation Behavior; FEAT-20.SPEC-001 ## States)
- Scale: N/A — no entity of its own; payload bounded by one grocery list (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** US coverage can come from Instacart Connect / Instacart Developer Platform (IDP) with self-service keys. UK coverage can come from Grocery aggregator APIs (e.g., listed on API marketplaces), each to be evaluated, or from a Direct retailer partnership/custom integration (e.g., a UK grocer such as Tesco) through business development (Online Grocery Ordering Integration). The options differ in onboarding path (self-serve vs negotiated) and market reach.

**Risks & Unknowns:** No UK partner may be reachable on reasonable terms, which would make the feature US-only. Mapping free-text list lines to retailer catalog SKUs (units, pack sizes) is not specified upstream. The Grocery List leaves the product boundary here (feature-overview.md ## Non-Functional Notes), which needs a data-sharing disclosure. Later phase, so no MVP impact (ASMP-37).

**Spike Recommendation:** Run a time-boxed partner discovery before the Later phase. Confirm Instacart IDP's commercial terms and its list-to-cart item-matching behavior. Contact at least two UK grocers or aggregators about API availability. Test whether list lines like "2 onions" resolve to catalog items at an acceptable match rate. The answer settles the launch market set and the integration shape for FEAT-20.SPEC-002.

### FEAT-21 — Family Calendar Sync

**Verdict:** Standard-with-integration — an external calendar capability with an OAuth-style connect/disconnect (FEAT-21.SPEC-001; FEAT-21.SPEC-002 ## Degradation Behavior) and background per-night create/update with silent per-night retries, additive-only and non-blocking (FEAT-21.SPEC-003; FEAT-21.SPEC-004 ## Business Rules).

**Required Capabilities:**
- Connect and disconnect one calendar per household (FEAT-21.SPEC-001; FEAT-21.SPEC-004)
- Background per-night entry create/update from the current plan, content limited to dish and night (FEAT-21.SPEC-003; FEAT-21.SPEC-005)
- Inbound sync-outcome and disconnect events (FEAT-21.SPEC-002)
- Concurrency: plan changes during sync, and connect vs disconnect, resolve under the one-connection rule with per-night retry counters (FEAT-21.SPEC-004 ## Business Rules)
- Offline/degraded: sync failures are silent and retried; a failed connect returns to Not Connected with the plan unaffected (FEAT-21.SPEC-002 ## Degradation Behavior)
- Scale: at most one entry per planned meal per connected household, one week ahead (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Google Calendar API (direct) for a single provider. Cronofy or Nylas as unified APIs across Google, Microsoft and Apple (Nylas bundles more scope than needed, and the landscape flags a forced-migration lock-in signal). Direct multi-provider integration (Google Calendar API + Microsoft Graph API, self-built). All four are in Family Calendar Integration. Retries fit any Background Jobs & Scheduling option.

**Risks & Unknowns:** Google's OAuth verification for calendar-write scopes can add review lead time. Apple/iCloud calendar has no first-party REST API outside the unified vendors. Later phase (ASMP-37). The household-deletion cascade must disconnect the calendar (FEAT-18.SPEC-008 step 5).

**Spike Recommendation:** None

### FEAT-22 — Operator Read-Only Support Access

**Verdict:** Straightforward — access is gated to one household with an open Support Request (FEAT-22.SPEC-006). The view is read-only with kid-profile and payment fields masked (FEAT-22.SPEC-007), and every open and close appends to an access record visible to the organiser (FEAT-22.SPEC-004 ## Processing Logic; FEAT-22.SPEC-009).

**Required Capabilities:**
- Support request queue and read-only household view for the operator (FEAT-22.SPEC-001, FEAT-22.SPEC-002)
- Scope gating tied to request status, with field-level visibility rules (FEAT-22.SPEC-006, FEAT-22.SPEC-007; XBR-14)
- Append-only access-session logging and status transitions (FEAT-22.SPEC-004, FEAT-22.SPEC-005, FEAT-22.SPEC-008); in-app household notices (FEAT-22.SPEC-009, FEAT-22.SPEC-010 ## Channels)
- Concurrency: only one household open at a time for Riley, and request resolution closes access (FEAT-22.SPEC-002 ## Edge Cases; feature-dependency-map.md, Support Request **Contention:** None)
- Offline/degraded: standard offline messaging on operator screens (FEAT-22.SPEC-002 ## States)
- Scale: rare, on-demand, one household at a time (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** An operator role in the Authentication & Identity option (Clerk, Auth0 or Supabase Auth roles; Better Auth or NextAuth.js custom roles). Enforcement at the data layer (Supabase row-level security policies, or query-scoped repositories via Prisma ORM or Drizzle ORM) vs the API layer (NestJS guards and similar in the Backend / API Layer area). Audit records in any Database option.

**Risks & Unknowns:** Read-only must be enforced server-side, not just by hiding UI. Observability tools (Sentry session replay, Datadog RUM in Observability & Operations) can capture masked kid data from the operator's screens unless scrubbed.

**Spike Recommendation:** None

### FEAT-23 — Manual Weekly Planning

**Verdict:** Straightforward — per-night pick, change and clear with reject-with-refresh on stale night state (FEAT-23.SPEC-006 ## Edge Cases) and safe-choice filtering through the shared safety engine (FEAT-23.SPEC-004). List updates and live propagation reuse FEAT-06 and FEAT-03's sync, and no AI is involved (feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Week builder, recipe picker and search within about a second (FEAT-23.SPEC-001, FEAT-23.SPEC-002; ASMP-23)
- Placement blocked for ineligible recipes, with reasons in words (FEAT-23.SPEC-004; XBR-01)
- Pick suggestions from other adults (FEAT-23.SPEC-003; XBR-06); validation limits (FEAT-23.SPEC-005)
- Apply a pick and trigger list recalculation (FEAT-23.SPEC-006; XBR-03)
- Concurrency: stale night state is rejected ("This night changed while you were choosing"), a safety removal wins, and picks on different nights proceed independently (FEAT-23.SPEC-006 ## Edge Cases)
- Offline/degraded: failed saves stay visible with retry (FEAT-23.SPEC-006 ## Edge Cases); the plan stays viewable offline (ASMP-25)
- Scale: zero AI cost; weekly plan growth equal to FEAT-03's; picks instant once reflected on the list (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Picker search via PostgreSQL full-text search or Meilisearch, Typesense or Algolia (Search). Conditional writes per night slot in any Database option. Propagation via the shared Real-time & Collaboration option (Supabase Realtime, Ably, Pusher (Channels), Socket.io (self-hosted)).

**Risks & Unknowns:** Inherits FEAT-02's latency risk, because every picker listing needs per-recipe verdicts for the household. Without precomputed verdicts, filtering the whole library on each search could miss the one-second target (ASMP-23).

**Spike Recommendation:** None

### FEAT-24 — Invite Another Household

**Verdict:** Straightforward — one durable, reusable referral link per adult member (FEAT-24.SPEC-003 ## Processing Logic), a write-once attribution record created at setup completion within 30 days (FEAT-24.SPEC-004; FEAT-24.SPEC-006; XBR-20), and an upgrade flag derived from the new household's subscription (FEAT-24.SPEC-005).

**Required Capabilities:**
- Personal referral link provisioning and a share screen (FEAT-24.SPEC-001, FEAT-24.SPEC-003)
- Referral welcome screen that shows only the inviter's first name to visitors (FEAT-24.SPEC-002)
- Attribution captured through sign-up and setup, and upgrade tracking (FEAT-24.SPEC-004, FEAT-24.SPEC-005); in-app referral-joined notice (FEAT-24.SPEC-007 ## Channels)
- Concurrency: link provisioning is idempotent, returning the same link from any device (FEAT-24.SPEC-003 ## Processing Logic); the referral record is never edited (feature-dependency-map.md, Household Referral **Contention:** None)
- Offline/degraded: link creation needs connectivity, and an existing link is shown from local state (FEAT-24.SPEC-001 ## States; FEAT-24.SPEC-003 ## Edge Cases)
- Scale: roughly one referral record per new household, several thousand in year one (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Attribution can travel through the sign-up flow of the Authentication & Identity option (custom metadata in Clerk, Auth0 or Supabase Auth; a custom parameter in Better Auth or NextAuth.js). Growth measurement can go to PostHog, Mixpanel or Amplitude (Analytics & Product Telemetry) alongside the in-database record.

**Risks & Unknowns:** Attribution can break if the 30-day window crosses devices (link opened on one device, setup completed on another). That is a measurement-accuracy risk, not a feasibility one.

**Spike Recommendation:** None

### FEAT-25 — Weekly Waste & Spend Check-In

**Verdict:** Straightforward — one optional record per household per week created on the plan's week boundary (FEAT-25.SPEC-003 ## Trigger Definition), with a simple trend against a starting-point record (FEAT-25.SPEC-004) and validation and access rules (FEAT-25.SPEC-005).

**Required Capabilities:**
- Weekly check-in card and trend view (FEAT-25.SPEC-001, FEAT-25.SPEC-002)
- Week-boundary cycle trigger per household (FEAT-25.SPEC-003)
- Spend shown in household currency (XBR-11)
- Concurrency: two adults answering the same week's check-in resolves under FEAT-25.SPEC-005's single-record rule, and cycle firing is once per household per week (FEAT-25.SPEC-003 ## Edge Cases)
- Offline/degraded: the card follows the product's standard offline messaging (FEAT-25.SPEC-001 ## States); ASMP-25's live-list resilience explicitly does not apply (feature-overview.md ## Non-Functional Notes)
- Scale: about 52 records per household per year (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** The cycle can be a lazily created record when the card first renders in a new week (no job needed) or a scheduled job in any Background Jobs & Scheduling option (Inngest, Trigger.dev, Upstash QStash, BullMQ). Cross-household success aggregation can go to PostHog, Mixpanel or Amplitude.

**Risks & Unknowns:** None identified — trivial volume, no external dependency.

**Spike Recommendation:** None

## 3. Cross-Feature Technical Themes

| Theme / Shared Subsystem | Features Involved | Evidence That Makes It Shared |
|--------------------------|-------------------|-------------------------------|
| App-enforced safety check on every path onto the plan | FEAT-02, FEAT-03, FEAT-04, FEAT-08, FEAT-10, FEAT-17, FEAT-19, FEAT-23 | XBR-01; generation (FEAT-03.SPEC-010 ## Data Exchanged "only after passing FEAT-02's safety check"), swaps (FEAT-04.SPEC-008), badges on browse (FEAT-08.SPEC-003), import save/edit re-check (FEAT-10.SPEC-008), vote options (FEAT-17.SPEC-004), history reuse (FEAT-19.SPEC-003 ## Processing Logic), manual picks (FEAT-23.SPEC-004) all invoke FEAT-02.SPEC-002 |
| AI generation integration and cost envelope | FEAT-03, FEAT-04, FEAT-05, FEAT-12, FEAT-10 | Plan generation (FEAT-03.SPEC-010) and swap alternatives (FEAT-04.SPEC-007) share the ASMP-30 capability; pantry (FEAT-05.SPEC-005) and rating (FEAT-12.SPEC-004) weighting are inputs to the same call; the landscape's AI-fallback extraction option for FEAT-10 depends on the same provider (technology-landscape.md Section 3) |
| Real-time propagation across household devices | FEAT-03, FEAT-04, FEAT-06, FEAT-09, FEAT-23 | FEAT-03.SPEC-011 and FEAT-06.SPEC-005 Integration specs; FEAT-04 and FEAT-23 rely on them (feature-dependency-map.md External Touchpoints, ASMP-35); live member list on acceptance (profile Section 3 Real-time, FEAT-01.SPEC-004) |
| Offline local persistence and queued sync | FEAT-01, FEAT-03, FEAT-05, FEAT-06, FEAT-09, FEAT-11, FEAT-12, FEAT-17 | Setup drafts (FEAT-01.SPEC-013), offline plan viewing (FEAT-03.SPEC-011 ## Degradation Behavior), pantry adds (FEAT-05 ## Non-Functional Notes), list queue (FEAT-06.SPEC-008), held invitation drafts (FEAT-09 ## Non-Functional Notes), leftover viewing (FEAT-11 ## Non-Functional Notes), rating and vote background retries (FEAT-12.SPEC-001, FEAT-17.SPEC-001 ## States) |
| Shared-record contention (reject-with-refresh, last-write-wins, merge) | FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-06, FEAT-09, FEAT-10, FEAT-11, FEAT-12, FEAT-14, FEAT-17, FEAT-23 | 16 of 17 entities carry **Contention:** rules in feature-dependency-map.md ## Shared Data Entities; Weekly Plan/Planned Meal writes from FEAT-02/03/04/11/23 share the per-slot reject-with-refresh and safety-removal-wins rules |
| Plan → grocery list derivation and recalculation | FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-23 | XBR-03 and XBR-04; every plan write (FEAT-02.SPEC-004, FEAT-03.SPEC-005, FEAT-04.SPEC-004, FEAT-23.SPEC-006) triggers FEAT-06.SPEC-002, which excludes pantry items (FEAT-05.SPEC-007) |
| Scheduled per-household background jobs (local-time aware) | FEAT-03, FEAT-04, FEAT-06, FEAT-07, FEAT-09, FEAT-13, FEAT-18, FEAT-21, FEAT-25 | Weekly generation (FEAT-03.SPEC-003), suggestion lapse (FEAT-04.SPEC-005), week rollover (FEAT-06.SPEC-004), plan-ready dispatch (FEAT-07.SPEC-001), invitation expiry (FEAT-09.SPEC-006), daily nudge (FEAT-13.SPEC-001), export and deletion cascades (FEAT-18.SPEC-006, FEAT-18.SPEC-008), calendar retries (FEAT-21.SPEC-004), check-in cycle (FEAT-25.SPEC-003) |
| Device-notification (push) delivery | FEAT-04, FEAT-07, FEAT-13, FEAT-23 | FEAT-07.SPEC-005 is the shared boundary used by FEAT-13.SPEC-002/004 and by FEAT-04.SPEC-006 and FEAT-23 pick suggestions (feature-dependency-map.md External Touchpoints, ASMP-31) |
| Transactional email delivery with queued retry | FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18 | Five Integration specs share ASMP-32 and the same "no user-visible effect, queued until recovery" degradation pattern (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012, FEAT-18.SPEC-012 ## Degradation Behavior) |
| In-app notification inbox | FEAT-01, FEAT-02, FEAT-04, FEAT-09, FEAT-14, FEAT-18, FEAT-22, FEAT-24 | In-app channel in FEAT-01.SPEC-018, FEAT-02.SPEC-011/012/014, FEAT-04.SPEC-006, FEAT-09.SPEC-012..014, FEAT-14.SPEC-010/011, FEAT-18.SPEC-013, FEAT-22.SPEC-009/010, FEAT-24.SPEC-007 ## Channels |
| Tier gating from subscription state | FEAT-03, FEAT-05, FEAT-12, FEAT-14, FEAT-23 | XBR-05; FEAT-03.SPEC-009, FEAT-05.SPEC-005, FEAT-12.SPEC-004 all read the Subscription tier that FEAT-14.SPEC-008 writes |
| Role-based authorization including kid and operator roles | FEAT-01, FEAT-06, FEAT-09, FEAT-12, FEAT-14, FEAT-17, FEAT-18, FEAT-22 | FEAT-01.SPEC-016, FEAT-06.SPEC-009, FEAT-09.SPEC-011, FEAT-12.SPEC-003, FEAT-14.SPEC-006, FEAT-17.SPEC-006, FEAT-18.SPEC-011, FEAT-22.SPEC-006 Authorization Rules |
| Children's-privacy-class data handling across vendors | FEAT-01, FEAT-02, FEAT-03, FEAT-12, FEAT-17, FEAT-18, FEAT-22 | Data Sensitivity lines for Member Profile, Dietary Rule, Rating, Dinner Vote and Support Request (feature-dependency-map.md); dietary constraints leave the product in generation requests (FEAT-03.SPEC-010 ## Data Exchanged); export concentrates kid data (FEAT-18 ## Non-Functional Notes) |
| Locale-aware display (units, currency, aisles) | FEAT-03, FEAT-06, FEAT-08, FEAT-14, FEAT-16, FEAT-23, FEAT-25 | XBR-11; FEAT-16.SPEC-004 conversion consumed by plan cost, recipe quantities, list aisles, billing amounts (FEAT-14 ## Non-Functional Notes) and check-in spend |
| Recipe corpus and ingredient data quality | FEAT-02, FEAT-08, FEAT-10 | FEAT-08.SPEC-004 and FEAT-10.SPEC-005 both feed Recipe ingredients that FEAT-02.SPEC-007's completeness policy and FEAT-02.SPEC-002's allergen matching consume (XBR-19) |

## 4. Key Technical Risks

| Risk | Features Affected | Driving Evidence | Possible Mitigation Directions |
|------|-------------------|------------------|--------------------------------|
| Ingredient-to-allergen matching misses an allergen in free-text or compound ingredients, breaking the zero-incident promise | FEAT-02, FEAT-03, FEAT-04, FEAT-10, FEAT-17, FEAT-19, FEAT-23 | FEAT-02.SPEC-002 ## Edge Cases (compound terms), ## Processing Logic step 6; imported free text (FEAT-10.SPEC-005); no allergen-taxonomy option in technology-landscape.md | Run the FEAT-02 spike; layer a content-vendor taxonomy under a curated override list; conservative expansion of ambiguous terms; a regression suite of labeled recipes gating every rule or taxonomy change |
| The fail-closed completeness bar excludes a large share of recipes, leaving plans partial or repetitive for allergy-heavy households | FEAT-02, FEAT-03, FEAT-08, FEAT-10 | FEAT-02.SPEC-007 ## Field Validation Rules (quantity+unit on every ingredient); FEAT-03.SPEC-010 ## Edge Cases (small pools) | Measure the exclusion rate in the FEAT-02/FEAT-10 spikes; prompt users on the review screen to fill missing quantities; a product decision on "to taste" ingredients |
| LLM generation fails to fill a safe, in-budget week often enough, or exceeds the under-a-minute or cost envelope | FEAT-03, FEAT-04, FEAT-12, FEAT-05 | FEAT-03.SPEC-010 ## Edge Cases, ## Degradation Behavior; ASMP-23; SC-16 | Run the FEAT-03 spike across the AI & Intelligent Behavior options; deterministic pre-filtering to shrink prompts; a deterministic fallback selector; batch or cached-prompt pricing for the scheduled run |
| Offline grocery-list reconciliation diverges across devices (client clock skew, recalculation races, mobile service-worker limits) | FEAT-06, FEAT-05, FEAT-04, FEAT-23 | FEAT-06.SPEC-008 ## Cross-Field Rules (event-time LWW), ## Edge Cases; ASMP-25 | Run the FEAT-06 spike; server-assigned sequencing or hybrid logical clocks; stable line identity across recalculation; the Real-time & Collaboration options differ in ordering guarantees, which is a selection criterion |
| The swap chain (AI call + safety + write + recalculation + propagation) exceeds the under-10-seconds target | FEAT-04, FEAT-06, FEAT-13 | FEAT-04 ## Non-Functional Notes; FEAT-04.SPEC-007 ## Degradation Behavior (8-second slow note) | Precomputed alternatives from the verified pool; lower-latency model tiers; optimistic UI with server confirmation |
| Web Push reach on iOS depends on home-screen install, so plan-ready and nudge metrics underperform | FEAT-07, FEAT-13, FEAT-04 | FEAT-07.SPEC-005 ## Scope and Non-Goals (web app, no native, SC-05); FEAT-07 ## Non-Functional Notes (80% within one minute) | An install prompt as part of onboarding; the email fallback already specified (FEAT-07.SPEC-006); track reach by platform via Analytics & Product Telemetry |
| Children's data flows to third-party processors (LLM, auth, realtime, analytics, observability, email) beyond the parent-controlled boundary | FEAT-01, FEAT-02, FEAT-03, FEAT-12, FEAT-17, FEAT-18, FEAT-22 | ASMP-26, ASMP-27; FEAT-03.SPEC-010 ## Data Exchanged; Data Sensitivity lines in feature-dependency-map.md | Vendor data-processing terms and zero-retention settings as a selection criterion; minimize fields sent; scrub kid data from telemetry and session replay; self-host options (Trigger.dev, PostHog, Better Auth) where they reduce exposure |
| Deletion within 30 days misses copies outside the primary database | FEAT-18, FEAT-14, FEAT-21, FEAT-03 | FEAT-18.SPEC-008 ## Processing Logic (entity list omits processor, push, email, analytics, backup and export-file copies); ASMP-27 | A deletion inventory per vendor; object-storage lifecycle expiry for export files; backup retention at or under 30 days |
| Scheduled bursts at default time slots hit provider rate limits or job-platform quotas | FEAT-03, FEAT-07, FEAT-13 | platform-parameters.md `plan-arrival-time-slots`, `nightly-nudge-send-time`; FEAT-03.SPEC-003 ## Trigger Definition | Staggered dispatch within a slot; provider rate-limit discovery; job-platform pricing at the projected monthly run count |
| Payment webhooks arrive duplicated or out of order, corrupting billing_state or grace timing | FEAT-14, FEAT-03, FEAT-05, FEAT-12 | FEAT-14.SPEC-009 ## Inbound Events; FEAT-14.SPEC-007; feature-dependency-map.md, Subscription **Contention:** | Idempotent event handling keyed on processor event IDs; state-machine guards; a billing-management layer (Chargebee, Recurly) or merchant of record (Paddle) as alternatives to custom dunning |
| Legal status of web recipe import blocks FEAT-10 at v1 | FEAT-10 | ASMP-36; BRIEF.md Open Questions | A product/legal decision before v1; manual entry (FEAT-10.SPEC-003) remains a functional fallback |
| UK grocery ordering partner unavailable | FEAT-20 | technology-landscape.md Online Grocery Ordering Integration (no Tesco API found) | The FEAT-20 spike; a US-first scope; aggregator evaluation |

## 5. Open Questions for the Build Team

| # | Question | Why It Matters | What Would Resolve It |
|---|----------|----------------|------------------------|
| 1 | Does the organiser's affirmative confirmation (FEAT-01.SPEC-007) meet "verifiable parental consent" for US and UK children's-privacy obligations (ASMP-27)? | If not, FEAT-01 needs a consent-verification capability the landscape does not cover, which changes its integration scope | Legal review of COPPA/UK Age Appropriate Design Code obligations; if verification is required, a landscape addition for consent-verification services |
| 2 | Which ingredient-to-allergen taxonomy source backs the safety engine, vendor-provided or curated? | Drives FEAT-02's Hard verdict and the Recipe & Food Content Data selection | The FEAT-02 spike's recall and exclusion-rate results |
| 3 | Which LLM provider and prompt structure meet the under-a-minute, cost-per-household and full-week success targets? | Determines FEAT-03's feasibility path and FEAT-04's alternatives path | The FEAT-03 spike (success rate, p95 latency, tokens per run, monthly cost at projected households) |
| 4 | Should ingredients without a quantity or unit ("to taste", "a pinch") be exempt from the fail-closed completeness bar? | Affects how many starter and imported recipes are ever plan-eligible (FEAT-02.SPEC-007, FEAT-08, FEAT-10) | A product decision informed by the FEAT-02/FEAT-10 spike exclusion rates |
| 5 | Do the recipe-content vendors' licenses permit storing and redisplaying recipes in the household library, and at what tier cost versus the sub-$100/month pre-revenue budget? | Determines which Recipe & Food Content Data options are viable for FEAT-08 | Vendor terms review for Spoonacular, Edamam and licensed-content partners |
| 6 | Is importing recipes from other websites legally acceptable, and in what form (full steps vs ingredients plus link)? | Gates FEAT-10 at v1 (ASMP-36) | A founder/legal decision |
| 7 | What reach does Web Push achieve for the household base on iOS, and is a home-screen-install prompt acceptable in onboarding? | Affects FEAT-07's 80%-within-one-minute metric and FEAT-13's value | Early telemetry by platform; a product decision on install prompting |
| 8 | Do offline list edits reconcile on client event time or server-assigned order? | Separates a manageable FEAT-06 build from a divergence-prone one | The FEAT-06 spike under clock skew |
| 9 | Merchant of record (tax handled) or processor plus self-built dunning, given US and UK sales and a three-month revenue target? | Shapes FEAT-14's integration effort and compliance surface | Founder decision on tax-handling ownership vs per-transaction cost |
| 10 | Which grocery partners, and which markets, are in scope for the Later-phase handoff? | FEAT-20 verdict depends on UK availability | The FEAT-20 partner-discovery spike |
| 11 | Which external copies (processor, push subscriptions, email logs, analytics, backups, export files) fall inside the 30-day deletion promise? | FEAT-18 compliance with ASMP-27 | A vendor-by-vendor deletion inventory once the stack is selected |
| 12 | What identity and consent model applies to the Later-phase older-kid limited login? | FEAT-17 depends on it; managed identity providers' terms for minors' accounts vary | Resolution of BRIEF.md's open question on kids' representation, plus provider terms review |
| 13 | Should volume-to-weight conversion (cups to grams for solids) be supported? | FEAT-16 specifies only fixed same-dimension factors; UK households may expect grams | A product decision; if yes, per-ingredient density data from the Recipe & Food Content Data source |
