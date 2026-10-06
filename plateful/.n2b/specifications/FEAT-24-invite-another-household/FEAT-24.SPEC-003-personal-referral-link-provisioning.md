---
document_type: spec
spec_type: automation
spec_id: FEAT-24.SPEC-003
spec_name: Personal Referral Link Provisioning
spec_slug: personal-referral-link-provisioning
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Personal Referral Link Provisioning

## Overview

**Name:** Personal Referral Link Provisioning
**ID:** FEAT-24.SPEC-003
**Type:** Automation
**Purpose:** Creates and persists an adult member's one reusable personal referral link the first time it is needed, and returns the existing one on every later request.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- Checking whether the requesting adult member already owns a personal referral link
- Creating exactly one durable, reusable personal referral link per adult member the first time one is needed
- Returning the existing link on any subsequent request from the same member
- Reporting creation failure back to the triggering screen for its retry offer

**Non-Goals:**
- Displaying the link, the Share and Copy actions, or the joined-families count -- owned by FEAT-24.SPEC-001 (Invite Another Household Screen); this automation only creates and returns the link value
- Recording a Household Referral when the link is used -- owned by FEAT-24.SPEC-004 (Household Referral Recording); provisioning a link and recording a referral are separate moments, per the Feature Breakdown Brief's Side-Effect Inventory
- Issuing a link to a kid profile or to Riley (Operator) -- excluded per the Access Matrix in user-persona.md: the Household Referrals column gives both kid rows and Riley None, so no request from those roles can reach this automation
- Expiring or rotating a member's link -- product-features.md's Validation & Limits states "one reusable personal link per adult member" with no expiry or rotation named; the link is durable for the life of the member's account

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invite Another Household Screen opens with no link yet | FEAT-24.SPEC-001 (Invite Another Household Screen) | Fires when the signed-in adult member (Maya or Sam) has no personal referral link on record | Requesting Member Profile reference, its owning Household |
| Member taps Retry after a provisioning failure | FEAT-24.SPEC-001 (Invite Another Household Screen) | Fires when the member retries after the Error state | Same as above |

## Processing Logic

1. Receive the requesting Member Profile's reference and confirm it is an adult member (Organiser or Other Adult Member) of an active Household -- FEAT-24.SPEC-006 governs this eligibility check.
2. Check whether this Member Profile already owns a personal referral link.
3. If a link already exists, return it unchanged -- no second link is ever created for the same member.
4. If no link exists, generate one new, durable, reusable personal referral link tied to this Member Profile and its Household.
5. Persist the new link so every future request from this member (this screen, this device, or any other) returns the same value.
6. Return the link to the triggering screen for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Existing link returned | The requesting member already owns a link | None | FEAT-24.SPEC-001 displays the link immediately | FEAT-24.SPEC-001 |
| New link created | The requesting member owns no link yet, and creation succeeds | New personal referral link persisted, tied to the requesting member and household | FEAT-24.SPEC-001 displays the newly created link | FEAT-24.SPEC-001 |
| Creation failure | The requesting member owns no link yet, and creation cannot be completed | No link persisted | FEAT-24.SPEC-001 shows its Error state: "We couldn't create your invite link." with Retry | FEAT-24.SPEC-001 |

## Data Model

**Reads:** Member Profile -- member_type (to confirm the requester is an adult member) and status (to confirm Active); Household -- status (to confirm the household is Active, not Closed/Deleted).
**Creates:** Personal Referral Link -- a durable identifier owned by the requesting adult Member Profile, created once per member. This is the identifier that a Household Referral record's referring_member_link field references once the link is used to attribute a new household (FEAT-24.SPEC-004).
**Updates:** None.
**Deletes:** None.

## Business Rules

- One reusable personal link per adult member (product-features.md, Validation & Limits) -- this automation never creates a second link for a member who already has one; the check-then-create step (Processing Logic, Step 2-4) is what enforces this.
- Only Maya and Sam (adult members, per the Household Referrals column of the Access Matrix) can trigger this automation; a kid profile or Riley never reaches FEAT-24.SPEC-001's trigger condition in the first place.
- The link is ready instantly once created -- no queued or delayed provisioning step exists (Feature Breakdown Brief, Non-Functional Notes, Responsiveness).
- Link creation requires connectivity; an already-created link remains usable offline (FEAT-24.SPEC-001's Offline/Degraded state), but this automation itself cannot run without a connection.

## Edge Cases

- **Member has no connectivity when this automation is triggered** -- Creation fails immediately with the same Creation failure outcome; FEAT-24.SPEC-001's Offline/Degraded state (rather than its Error state) is what the member sees in this specific case, since the screen can distinguish "no connection" from "connection present but creation failed."
- **The requesting household is Closed/Deleted at the moment of the request** -- Creation is refused; no link is created for a member of a household that no longer exists. This is a theoretical edge case in current product flow, since a member of a deleted household has no path back to FEAT-24.SPEC-001 in the first place.
- **Concurrent trigger firing (Maya opens the Invite Another Household Screen on her phone and her laptop at effectively the same time, neither having a link yet)** -- Each request runs its own check-then-create independently; the check-then-create step is idempotent, so only one link is ever persisted for Maya regardless of which request's create step lands first. The second request to complete finds the first request's link already persisted and returns it rather than creating a duplicate.
- **Trigger fires while a previous run is in flight (Retry tapped again before the first attempt has returned)** -- FEAT-24.SPEC-001 disables its Retry button while a provisioning attempt is in progress, so a second run for the same member cannot start until the first completes. Runs for different members proceed independently and never queue behind one another.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Triggered by (inbound) | Fires on first screen open with no link, and on Retry after a failure |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Affects (outbound) | Returns the link (or the failure outcome) for display |
| FEAT-24.SPEC-004 (Household Referral Recording) | References (outbound) | The link this automation creates is the identifier a later Household Referral record's referring_member_link field references |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Governs the one-link-per-member limit this automation enforces |

## Analytics and Success Signals

- **referral_link_provisioned** (outcome: existing_returned / newly_created) -- N/A -- no Stage 2 metric measures link provisioning itself; retained as a funnel-diagnostic signal ahead of the metric-bearing household-creation event (FEAT-24.SPEC-004, which supports success-metrics.md: "Household-to-Household Invitation Growth")
- **referral_link_provisioning_failed** (reason: offline / processing_error) -- N/A -- diagnostic signal only; a failure here must never silently block the member from later retrying, so this event measures how often that retry path is exercised

## Acceptance Criteria

**FEAT-24.SPEC-003-AC-01:** Given Maya opens the Invite Another Household Screen for the first time and has no personal link, when this automation fires, then a new link is created and persisted for her, tied to her Member Profile and household.

**FEAT-24.SPEC-003-AC-02:** Given Sam already has a personal link from a prior visit, when he opens the Invite Another Household Screen again, then this automation returns his existing link unchanged and creates no second one.

**FEAT-24.SPEC-003-AC-03:** Given Maya is signed in on both her phone and her laptop with no link yet on either, when she opens the screen on both at effectively the same time, then only one link is ever persisted for her, and both devices end up displaying that same link.

**FEAT-24.SPEC-003-AC-04:** Given Sam has no connectivity when the automation is triggered, when creation is attempted, then it fails and FEAT-24.SPEC-001 shows its Offline/Degraded state rather than its Error state.

**FEAT-24.SPEC-003-AC-05:** Given Maya has connectivity but link creation fails for a processing reason, when the automation completes, then FEAT-24.SPEC-001 shows "We couldn't create your invite link." with a Retry button.

**FEAT-24.SPEC-003-AC-06:** Given Sam taps Retry after a provisioning failure, when this automation fires again and succeeds, then his link is created and displayed, and no earlier failed attempt leaves behind a partial or duplicate link.

**FEAT-24.SPEC-003-AC-07:** Given Sam taps Retry a second time while the first retry attempt is still in flight, when the Retry button is inspected, then it is disabled and the second tap has no effect until the first attempt completes.

**FEAT-24.SPEC-003-AC-08:** Given a request somehow arrives from a kid profile or Riley (Operator) context, when this automation is invoked, then no link is created, since the Household Referrals column of the Access Matrix grants neither role any access to FEAT-24.SPEC-001's trigger in the first place.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (first open with no link, Retry after failure) | 2 |
| Outcome Paths | 3 (existing returned, newly created, creation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
