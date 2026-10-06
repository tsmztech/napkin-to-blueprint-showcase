---
document_type: spec
spec_type: automation
spec_id: FEAT-24.SPEC-004
spec_name: Household Referral Recording
spec_slug: household-referral-recording
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Household Referral Recording

## Overview

**Name:** Household Referral Recording
**ID:** FEAT-24.SPEC-004
**Type:** Automation
**Purpose:** Attributes a new household's completed setup to the referring household's link when eligible, enforcing the single-attribution, no-self-referral, and 30-day counting-window rules.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Evaluating, at the moment a new household is created, whether it arrived via a personal referral link and is still eligible for attribution
- Creating exactly one Household Referral record when eligible
- Enforcing single-attribution (a new household is attributed to at most one referring household, ever) and no-self-referral
- Enforcing the 30-day counting window between the link's most recent open and the new household's creation
- Triggering FEAT-24.SPEC-007 (Referral Joined Notification) when a record is created

**Non-Goals:**
- Creating the new Household or Member Profile records themselves -- owned by FEAT-01.SPEC-003 (Household Naming & Guided Setup Start); this automation fires once that creation succeeds and only writes the Household Referral record
- Displaying the referring member's link, the joined-families count, or the referral welcome message -- owned by FEAT-24.SPEC-001 and FEAT-24.SPEC-002; this automation is invisible to the new household's own setup experience
- Updating the `upgraded` flag once the new household later becomes a paying one -- owned by FEAT-24.SPEC-005 (Referral Upgrade Tracking); this automation only ever sets the initial record with `upgraded` false
- Any reward, credit, or discount tied to a successful referral -- excluded per scope-boundaries.md SC-10: this automation records the referral without paying for it

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New household completes creation | FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Fires every time a Household record is successfully created, regardless of entry source | New Household reference, its first Member Profile (the account that created it), and -- when present -- the referral context carried from FEAT-24.SPEC-002: the personal referral link opened, its owning Member Profile and Household, and the moment the link was most recently opened |

## Processing Logic

1. Receive the newly created Household from FEAT-01.SPEC-003.
2. Determine whether a referral link context was carried into this account's creation (per FEAT-24.SPEC-002). If none was carried, stop -- this is an unattributed household and no further processing occurs.
3. If a referral link context exists, identify the referring Household and the specific personal referral link (and its owning Member Profile) that was opened.
4. Check self-referral: if the referring Household is the same as the new Household, stop and record no referral (FEAT-24.SPEC-006, no-self-referral rule).
5. Check single attribution: confirm the new Household has never before been the `new_household` on any Household Referral record. If it has, stop and record no referral (FEAT-24.SPEC-006, single-attribution rule).
6. Check the 30-day counting window: confirm the new Household's creation date falls within 30 days of the referral link's most recent open moment (FEAT-24.SPEC-002). If the window has elapsed, stop and record no referral.
7. If all checks pass, create one Household Referral record: referring_household set to the referring Household, referring_member_link set to the personal referral link identifier, new_household set to the new Household, created_date set to the current date, upgraded set to false.
8. Trigger FEAT-24.SPEC-007 (Referral Joined Notification) for the Member Profile that owns the personal referral link used.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Referral recorded | No referral context carried is missing, no self-referral, no prior attribution for this new household, and the household was created within 30 days of the link's most recent open | New Household Referral record created (upgraded: false) | None on the new household's own setup screens; the referring member's joined-families count (FEAT-24.SPEC-001) reflects the new record on next view; FEAT-24.SPEC-007 delivers the referring member's in-app note | FEAT-24.SPEC-001, FEAT-24.SPEC-007 |
| No referral context | The new household's creation carried no referral link context | None | None -- this is the ordinary, unattributed household-creation path and is never surfaced as an error | FEAT-01.SPEC-003 |
| Self-referral blocked | Referral context exists, but the referring household equals the new household | None | None to the new household's own setup; this is a silent, no-action outcome per FEAT-24.SPEC-006 | -- |
| Already attributed | Referral context exists, but the new household already carries a Household Referral record from a prior link open | None -- the earlier record stands unchanged | None to the new household's own setup | -- |
| Window elapsed | Referral context exists and passes self-referral and single-attribution checks, but more than 30 days have passed since the link's most recent open | None | None to the new household's own setup | -- |
| Automation failure | An internal processing error prevents the write after all checks pass | No partial write -- no Household Referral record is left in an inconsistent state | The new household's own setup (FEAT-01.SPEC-003) completes normally regardless, since referral recording is a side effect of household creation, never a precondition for it | FEAT-01.SPEC-003 |

## Data Model

**Reads:** Household -- the new household's creation date and identity; Household Referral -- existing records, to check whether the new household already carries one (single-attribution check).
**Creates:** Household Referral -- referring_household, referring_member_link, new_household, created_date, upgraded (set to false).
**Updates:** None -- this automation never modifies an existing Household Referral record; that is FEAT-24.SPEC-005's exclusive role.
**Deletes:** None.

## Business Rules

- XBR-20: a new household set up from an invite link within 30 days is attributed to at most one referring household, never itself; someone who already has a household is told so and nothing is recorded (enforced upstream at FEAT-24.SPEC-002, and re-enforced here as this automation's own self-referral check).
- Attribution is a one-time, system-only write -- no role, including Maya or Sam, can create, edit, or back-date a Household Referral record directly (FEAT-24.SPEC-006, Authorization Rules).
- Household creation never waits on this automation -- FEAT-01.SPEC-003's own success and navigation are unaffected by whether a referral is recorded, per this feature's Non-Goals (recording is a side effect, not a precondition).
- The referring_member_link on the created record identifies whichever adult member's personal link was opened, even if a different adult member of the same household also has a link -- attribution is per-link, but the joined-families count shown on FEAT-24.SPEC-001 aggregates by referring_household, per the Feature Breakdown Brief's Shared Context.

## Edge Cases

- **Visitor opens two different households' referral links in the same session before completing setup (e.g., a link from Sam's household, then later a link from an unrelated household's member)** -- Last-touch attribution: whichever link's context was most recently carried at the moment FEAT-01.SPEC-003's household creation succeeds is the one this automation evaluates; the earlier link's context is discarded once superseded.
- **The referring household is deleted between the link being opened and the new household completing setup** -- The referring Household no longer resolves to a live record at evaluation time, so no referral is recorded; this is treated the same as "no referral context" rather than as a failure. Cascade behavior for any Household Referral records that already reference a since-deleted household is FEAT-18's responsibility, flagged in the Feature Breakdown Brief's Cross-Feature Touchpoints, not resolved by this automation.
- **New household's creation date lands exactly 30 days after the link's most recent open** -- The window is inclusive of day 30; a household created on day 31 or later falls outside the window and no referral is recorded.
- **Concurrent trigger firing (two different visitors complete FEAT-01.SPEC-003 at effectively the same time, both having opened the same household's same personal link)** -- Each new household's creation is evaluated independently against its own referral context; both can be recorded as separate Household Referral records against the same referring household and the same referring_member_link, since single-attribution is scoped to the new_household side of the relationship, not the referring side. Two different families can legitimately be referred by the same link.
- **Trigger fires while a previous run is in flight for the same new household (a duplicate household-creation signal from FEAT-01.SPEC-003, e.g., a retried request after a network hiccup)** -- The single-attribution check (Processing Logic, Step 5) makes a second attempt for the same new household a no-op: the second run finds the first run's record already present and stops without creating a duplicate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Triggered by (inbound) | New household creation success fires this automation |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | References (inbound) | Supplies the referral link context (referring household, member, and open moment) this automation evaluates |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Governs single-attribution, no-self-referral, and the 30-day counting window enforced here |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Affects (outbound) | The referring household's joined-families count reflects a newly recorded referral |
| FEAT-24.SPEC-007 (Referral Joined Notification) | Triggers (outbound) | A recorded referral fires the referring member's in-app note |
| FEAT-24.SPEC-005 (Referral Upgrade Tracking) | References (outbound) | The record this automation creates is later read and updated by upgrade tracking |

## Analytics and Success Signals

- **referred_household_created** (window_check: within_window; attribution: recorded) -- supports success-metrics.md: "Household-to-Household Invitation Growth"
- **referral_recording_skipped** (reason: no_context / self_referral / already_attributed / window_elapsed) -- N/A -- no Stage 2 metric measures skipped attributions directly; retained so the growth funnel's drop-off points remain observable rather than silently absorbed into "no referral"

## Acceptance Criteria

**FEAT-24.SPEC-004-AC-01:** Given a visitor followed Sam's personal referral link 3 days ago and completes FEAT-01.SPEC-003 today, when this automation fires, then a Household Referral record is created with referring_household set to Sam's household, referring_member_link set to Sam's link, new_household set to the new household, and upgraded set to false.

**FEAT-24.SPEC-004-AC-02:** Given a new household's creation carried no referral link context at all, when this automation fires, then no Household Referral record is created and FEAT-01.SPEC-003 completes exactly as it would for any other new household.

**FEAT-24.SPEC-004-AC-03:** Given a visitor opened their own household's own personal link before creating a second household under a different account, when this automation fires, then no referral is recorded, since the referring household and the new household are the same.

**FEAT-24.SPEC-004-AC-04:** Given a new household already carries a Household Referral record from an earlier link open, when a second referral context somehow reaches this automation for the same new household, then no second record is created and the original record is left unchanged.

**FEAT-24.SPEC-004-AC-05:** Given a visitor opened a link 31 days before completing FEAT-01.SPEC-003, when this automation fires, then no referral is recorded, since the 30-day counting window has elapsed.

**FEAT-24.SPEC-004-AC-06:** Given a visitor opened a link exactly 30 days before completing FEAT-01.SPEC-003, when this automation fires, then the referral is recorded, since the window is inclusive of day 30.

**FEAT-24.SPEC-004-AC-07:** Given the referring household was deleted after the link was opened but before the new household completed setup, when this automation fires, then no referral is recorded and the outcome is treated as having no live referral context.

**FEAT-24.SPEC-004-AC-08:** Given two different visitors each complete FEAT-01.SPEC-003 having opened the same personal link, when this automation fires for each, then two separate Household Referral records are created, both attributing to the same referring household and link.

**FEAT-24.SPEC-004-AC-09:** Given a Household Referral record is created, when the write completes, then FEAT-24.SPEC-007 (Referral Joined Notification) fires for the Member Profile that owns the referring_member_link.

**FEAT-24.SPEC-004-AC-10:** Given a duplicate household-creation signal arrives for a new household that already has a Household Referral record, when this automation runs a second time, then it makes no change and creates no duplicate record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 (recorded, no context, self-referral blocked, already attributed, window elapsed, automation failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
