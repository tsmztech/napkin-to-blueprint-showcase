---
document_type: feature-overview
feature_number: FEAT-17
feature_name: Older-Kid Dinner Voting
feature_slug: older-kid-dinner-voting
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 6
screen_count: 3
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Older-Kid Dinner Voting

## Summary

**Feature:** Older-Kid Dinner Voting
**ID:** FEAT-17
**Description:** Older kids can vote on which dinner they'd prefer among the week's options, giving them a voice in the plan without needing a full account.
**Priority:** Important
**Phase:** Later
**Type:** User-Facing
**Rationale:** The brief names this directly — "older kids want to vote on dinners" (BRIEF.md, Target Users & Roles) — but also leaves kids' representation as an explicit open question, with the founder leaning toward "possibly a limited login for older kids later" (BRIEF.md, Open Questions). Phased to Later because it depends on resolving that open question about how older kids are represented at all.

**Key Capabilities:**
- Vote on an option — Older kid picks a preferred dinner among a small set of safe alternatives for a given night
- See the outcome — Older kid sees which option was chosen once the organiser finalizes or the vote naturally resolves

This feature is Later-phase and gated on the Later-phase older-kid limited login existing at all (Access field; scope-boundaries.md SC-02). It depends on AI Weekly Dinner Plan Generation (FEAT-03) for the options it offers and on Dietary Rules & Allergy Safety Engine (FEAT-02) to guarantee every option is already safe (Interactions field). The Access field also gives Maya (Organiser) Full access to Dinner Voting — she opens voting rounds and makes the final call when votes conflict — so this Brief elaborates that organiser-side capability alongside the two Key Capabilities Stage 2 names for the older kid.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-17.SPEC-001 | Dinner Vote Casting | Screen | Jordan (older kid, limited login) | Older kid views the 2-3 safety-checked options for an open voting round on a night and casts one preference |
| FEAT-17.SPEC-002 | Voting Round Setup | Screen | Maya | Maya opens a voting round for a given night by selecting the 2-3 already safety-checked options to offer |
| FEAT-17.SPEC-003 | Vote Outcome & Resolution | Screen | Maya, Sam, Jordan (older kid, limited login) | Shows the current vote split or resolved outcome for a night; lets Maya make the final call when votes conflict |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | Automation | Maya | When Maya opens a round, validates the chosen options against the 2-3 count and FEAT-02's safety check, creates the round, and fires dinner_vote_opened |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | Automation | Maya, Sam, Jordan (older kid, limited login) | Recomputes the tally on each cast vote, auto-resolves a unanimous round, defaults to the AI's original suggestion when no vote was cast, writes the outcome onto the Weekly Plan, and fires dinner_vote_resolved |
| FEAT-17.SPEC-006 | Dinner Voting Rules — Access, Validation & Conflict Resolution | Logic/Rule | All | Governs who may open rounds, vote, view outcomes, or make the final call; enforces the option-count/safety-check and one-vote-per-profile-per-round limits; and rejects with refresh any vote cast after the round is already resolved |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Vote on an option | FEAT-17.SPEC-001 | Older kid selects one of the round's 2-3 safety-checked options; the write itself is validated by SPEC-006 | Phase 2 (Explicit) |
| See the outcome | FEAT-17.SPEC-003, FEAT-17.SPEC-005 | Outcome screen displays the tally or resolution that SPEC-005 computes or that Maya sets via her final call | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-17.SPEC-002 | Voting Round Setup | Phase 2 (Explicit surface, Access-field elaboration) | The Access field gives Maya Full access to Dinner Voting and states she "opens voting rounds" — a capability Stage 2's Key Capabilities list does not name directly but the Access field requires be elaborated into a spec, per this Brief's Stage 2 Depth Fields rule |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | Phase 4 (Trigger-Response, cross-feature effect) | Opening a round is more than a direct data write: it must validate the option count and re-confirm each option already passed FEAT-02's safety check (Interactions field, XBR-01) before the round can exist |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | Phase 4 (Trigger-Response) + Phase 6 (failure/no-consensus handling) | The Primary Flows & Alternates field names three distinct resolution paths (happy-path outcome, no-vote fallback to the AI's suggestion, and tie/no-consensus routed to Maya) that no single screen interaction covers; this automation is the shared trigger-response logic behind all three, and it is what actually updates the Weekly Plan (dependency map: Weekly Plan "Updated by FEAT-17, vote outcome") |
| FEAT-17.SPEC-006 | Dinner Voting Rules — Access, Validation & Conflict Resolution | Phase 5 (Rule-Constraint Discovery — authorization + conditional logic) | Role-differentiated authorization (Own-only vote vs. Full open/resolve vs. View outcome vs. None for the young-kid row, Riley, and unauthorized visitors), the option-count/safety-check and one-vote-per-profile limits (Validation & Limits field), and the post-resolution vote-rejection rule (Dinner Vote's Contention note) form conditional logic shared across every screen and automation in this feature — past the standalone-spec threshold |

**No Integration or Notification specs:** The Dependencies slice of assumptions-constraints.md states plainly "None — this feature relies on no external capability," and the Communications field states "N/A — voting is a self-initiated, in-app action; no separate notification is sent for this Later-phase feature." Both category-level checks (Phase 7, Check 4) resolve to zero without a gap: integration_count and notification_count are legitimately 0.

## Entity-Lifecycle Coverage Matrix

**Entity: Dinner Vote**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-17.SPEC-001 (vote), FEAT-17.SPEC-004 (round) | An older kid casts a vote (choice) into a round that SPEC-004 creates when Maya opens it, validated by SPEC-006 | One round per night; one vote per older-kid profile per round |
| Read (single) | FEAT-17.SPEC-003 | Vote Outcome screen reads one night's round and its votes | Also read cross-feature by FEAT-03 during plan review (dependency map) |
| Read (list) | FEAT-17.SPEC-003 | Displays every cast vote in the round to show the split | -- |
| Update | FEAT-17.SPEC-005 | Resolution field set by tally (auto-resolve) or by Maya's final call (SPEC-003 UI, SPEC-005 automation applies it) | A vote itself is never edited after casting -- only the round's resolution is updated |
| Delete/Archive | N/A | No delete or archive spec exists for Dinner Vote — an explicit non-goal. Votes are retained for the life of the household account alongside the Weekly Plan history they belong to (scope-boundaries.md SC-18), with no separate purge policy | -- |
| State Transition | FEAT-17.SPEC-004 (Open), FEAT-17.SPEC-005 (Resolved) | A round moves Open -> Resolved, either by tally (unanimous or fallback) or by Maya's override | -- |

**Entity: Weekly Plan** (this feature only updates one field on an existing plan; creation, full read, and deletion belong to other features per the dependency map)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-03 (AI generation) and FEAT-23 (manual planning) — not this feature's responsibility | Cross-feature ownership, not a gap |
| Read (single) | FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-17.SPEC-003 | Each screen reads the relevant night's plan context to show what is being voted on | -- |
| Read (list) | N/A | This feature never lists whole plans — that is FEAT-03's/FEAT-23's Weekly Plan screen responsibility | Cross-feature ownership |
| Update | FEAT-17.SPEC-005 | Writes the resolved option onto the night's slot once the round resolves | Dependency map: Weekly Plan "Updated by FEAT-17 (Later, vote outcome)" |
| Delete/Archive | N/A | Owned by FEAT-18 (household deletion cascade) / plan archival at week end — not this feature | Cross-feature ownership |
| State Transition | N/A | Weekly Plan's own status lifecycle (Generated/Reviewed/Approved/Active/Archived) belongs to FEAT-03/FEAT-23; this feature only ever writes the chosen-option value on one night's slot | Cross-feature ownership |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Member Profile | FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-17.SPEC-006 | Identifies the voting older-kid profile and its `member_type`, so the one-vote-per-profile rule and the Access Matrix's role checks can be enforced |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya opens a voting round for a night, selecting 2-3 options | Validate the option count and that every option already passed FEAT-02's safety check, create the round, fire dinner_vote_opened | Standalone Automation | FEAT-17.SPEC-004 |
| Older kid casts a vote | Record the vote, enforcing one vote per older-kid profile per round | Inline in triggering screen, validated by SPEC-006 | FEAT-17.SPEC-001 |
| A vote is cast (dinner_vote_cast) | Recompute the round's tally | Standalone Automation | FEAT-17.SPEC-005 |
| Tally recompute finds unanimous agreement | Auto-resolve the round to that option, write it onto the Weekly Plan, fire dinner_vote_resolved | Standalone Automation | FEAT-17.SPEC-005 |
| No vote is cast for a night by the plan's resolution point | Round resolves to the AI's original suggestion — voting never blocks the plan | Standalone Automation | FEAT-17.SPEC-005 |
| Votes split across more than one option with no consensus | Round is flagged as split; Maya sees the split and makes the final call rather than the app deciding | Standalone Automation (flag) feeding a Screen action | FEAT-17.SPEC-005 / FEAT-17.SPEC-003 |
| Maya makes her final call on a split round | Round resolves to Maya's choice, writes it onto the Weekly Plan, fires dinner_vote_resolved | Inline in triggering screen, applies SPEC-005's resolution write | FEAT-17.SPEC-003 |
| A vote is cast after the round has already resolved | Rejected with refresh, showing the resolved outcome instead | Standalone Logic/Rule | FEAT-17.SPEC-006 |
| Vote submission fails | Retries automatically | Inline in triggering screen (Error state) | FEAT-17.SPEC-001 |
| Vote cast while offline | Held locally, synced once connectivity returns | Inline in triggering screen (Offline-degraded state) | FEAT-17.SPEC-001 |
| A safety-related change removes one of a round's options mid-week | Option removal is FEAT-02's responsibility; the round's remaining valid options carry forward | Cross-feature — owned by Dietary Rules & Allergy Safety Engine | FEAT-02 responsibility, referenced by FEAT-17.SPEC-004 |

## Shared Context

**Shared Entities:**
- Dinner Vote — round context and options created by SPEC-004 (opening), individual votes created by SPEC-001 (casting), resolution updated by SPEC-005, displayed by SPEC-003, all validated against SPEC-006. Fields: round (night + 2-3 safety-checked options), voter (older-kid Member Profile), choice, resolution.
- Weekly Plan — the relevant night's slot is read for context by SPEC-001/002/003 and updated with the resolved choice by SPEC-005. No other Weekly Plan field is touched by this feature.
- Member Profile — read by SPEC-001, SPEC-002, and SPEC-006 to identify the voting older-kid profile and enforce role-based access; never created, updated, or deleted by this feature.

**Shared UI Patterns:**
- Safety-checked option card — the same option-display pattern (recipe name, the "checked against allergies" badge, and the "always check labels" disclaimer per XBR-01) appears in SPEC-001 (voting), SPEC-002 (Maya's option selection), and SPEC-003 (outcome display). Spec Writers for all three screens should describe this card consistently rather than re-deriving its content.

**Shared Validation:**
- FEAT-17.SPEC-006 defines the role/access rules, the option-count and safety-check limits, the one-vote-per-profile rule, and the post-resolution rejection rule. SPEC-001 through SPEC-005 all reference SPEC-006 rather than restating any of these rules.

## Internal Dependency Map

```
FEAT-17.SPEC-002 (Voting Round Setup) -> [Maya selects options and opens the round] -> FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation)
FEAT-17.SPEC-002 (Voting Round Setup) -> [validates against] -> FEAT-17.SPEC-006 (Dinner Voting Rules)
FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation) -> [round now open] -> FEAT-17.SPEC-001 (Dinner Vote Casting)
FEAT-17.SPEC-001 (Dinner Vote Casting) -> [validates against] -> FEAT-17.SPEC-006 (Dinner Voting Rules)
FEAT-17.SPEC-001 (Dinner Vote Casting) -> [vote cast] -> FEAT-17.SPEC-005 (Vote Round Resolution & Fallback)
FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) -> [split detected, no consensus] -> FEAT-17.SPEC-003 (Vote Outcome & Resolution)
FEAT-17.SPEC-003 (Vote Outcome & Resolution) -> [Maya's final call] -> FEAT-17.SPEC-005 (Vote Round Resolution & Fallback)
FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) -> [round resolved] -> FEAT-17.SPEC-003 (Vote Outcome & Resolution)
FEAT-17.SPEC-006 (Dinner Voting Rules) -> [rejects a late vote, points to] -> FEAT-17.SPEC-003 (Vote Outcome & Resolution)
```

**Default Entry:** FEAT-17.SPEC-002 (Voting Round Setup) for Maya, navigated from her Sunday plan review (FEAT-03); FEAT-17.SPEC-001 (Dinner Vote Casting) for the older kid, navigated from their view of the week's plan once a round is open. Both roles reach FEAT-17.SPEC-003 (Vote Outcome & Resolution) to see a round's split or resolved outcome.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-17.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Maya opens a voting round for a night from within her Sunday plan review | Sunday Plan Review & First Swap journey, step 3 |
| FEAT-17.SPEC-004 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Only options that have already passed the safety check may ever be offered in a round (XBR-01) | Round creation |
| FEAT-17.SPEC-004 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The candidate options a round offers come from the AI's proposed alternatives for that night | Round creation |
| FEAT-17.SPEC-005 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Writes the resolved option onto the night's slot in the Weekly Plan | Round resolves, by tally or Maya's final call |
| FEAT-17.SPEC-005 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | XBR-01: the resolution write, like every other path onto the plan, carries the same safety check and badge before anyone sees it | Round resolution writes to the plan |
| FEAT-17.SPEC-003 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The vote split or outcome is surfaced as part of the Sunday plan review, for households with voting enabled | Maya or Sam views the week's plan |

## Non-Functional Notes

**Data volumes / growth:** At most one open round per night, offering 2-3 options, with at most one vote per older-kid profile in the household (2-6 members per household, ASMP-24). Volume is trivial and scales only with household count, not with any growing dataset of its own.

**Responsiveness:** "Voting is a simple, instant tap" (States field) — casting completes with no perceptible wait; a failed submission retries automatically rather than surfacing a blocking error. No dedicated non-functional entry in assumptions-constraints.md names this feature directly, so it falls under the product's general one-handed, low-latency interaction bar (ASMP-29, Accessibility and one-handed use).

**Data sensitivity / privacy:** A Dinner Vote is children's data — a child's dinner preference — and is minimal, parent-controlled, and used only for the household's own plan (dependency map Data Sensitivity note; ASMP-26). It carries no name beyond the older-kid profile's own limited identity and no additional personal detail.

**Compliance flags:** Because the older-kid limited login and its votes are children's data, children's-privacy-class protections apply — minimal collection, parental control, no behavioral advertising to minors (ASMP-27). No medical or diet-advice regime applies, since this feature makes no dietary recommendation of its own (scope-boundaries.md SC-06).

## Non-Goals

- **Independent kid login as the v1 default** — Excluded per scope-boundaries.md (SC-02): the older-kid limited login this feature depends on is a Later-phase possibility, not a v1 baseline; the young-kid (no-login) profile row has `None` on Dinner Voting and can never vote, matching the Access Matrix.
- **The app arbitrarily deciding a tie** — Excluded per this feature's own Primary Flows & Alternates field, which states directly that "the organiser sees the split and makes the final call rather than the app arbitrarily deciding." No automatic tie-break algorithm is in scope; every genuine split routes to Maya.
- **A vote-triggered notification** — Excluded per this feature's own Communications field ("N/A ... no separate notification is sent for this Later-phase feature"); voting and its outcome are surfaced only within the in-app plan review, never as a push, email, or SMS message.
- **Automatic purge of vote history** — An intentional lifecycle decision surfaced by the CRUD matrix: Dinner Vote records are retained for the life of the household account alongside the Weekly Plan history they belong to (scope-boundaries.md SC-18), with no separate retention window or purge policy.
- **Kid-initiated round creation** — Excluded per the Access Matrix: the older-kid row's Dinner Voting access is `Own-only` (casting a vote), never `Full`; only Maya (`Full`) can open a voting round or make the final call on a split.
