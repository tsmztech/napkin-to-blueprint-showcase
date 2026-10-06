---
document_type: spec
spec_type: notification
spec_id: FEAT-02.SPEC-011
spec_name: Safety Concern Reporter Acknowledgement
spec_slug: safety-concern-reporter-acknowledgement
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Safety Concern Reporter Acknowledgement

## Overview

**Name:** Safety Concern Reporter Acknowledgement
**ID:** FEAT-02.SPEC-011
**Type:** Notification
**Purpose:** Confirms to the reporting adult that their safety concern was received and the meal has been removed.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- The in-app acknowledgement shown to the reporting adult the moment their report is processed
- Its content, delivery, and preference behavior

**Non-Goals:**
- Telling the organiser when she is not the reporter -- owned by FEAT-02.SPEC-012 (Safety Concern Organiser Alert), a distinct recipient and message
- Telling the operator -- owned by FEAT-02.SPEC-013 (Safety Concern Operator Alert)
- Telling the household the resolution outcome once reviewed -- owned by FEAT-02.SPEC-014 (Safety Concern Resolution Notice), a later, distinct communication
- Removing the meal or creating the Support Request -- owned by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), whose outcome this notification confirms

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when the report is processed | The reporter is already inside the product, on the screen where they just submitted the report (FEAT-02.SPEC-001); the acknowledgement completes that same interaction rather than requiring them to check elsewhere |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Safety concern report processed | FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Fires immediately after the reported meal is removed and the Support Request is created (or the duplicate-safe path is taken) | Reporting Member, reported Recipe name, reported meal's night |

## Audience and Preferences

**Recipients:** Maya (Organiser) or Sam (Other Adult Member) -- whichever adult submitted the report, per the Access Matrix's Safety Reports column (Maya: Full, Sam: Own-only). The acknowledgement is delivered only to the reporting member themselves, never to the other adult.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this acknowledgement has no independent preference control | -- | Always on | -- |

**Quiet Hours:** N/A -- this is an in-session, in-app confirmation delivered as the direct result of the reporter's own action; quiet hours govern notifications that interrupt the recipient away from an active task, and this one occurs within the same interaction the reporter just initiated.

## Content Definition

**In-app:**
- **Title:** Report received
- **Body:** {recipe_name} has been removed from your plan. Your household's operator will review it, and we'll let you know the outcome.
- **CTA:** Show me alternatives -- deep-links to FEAT-04.SPEC-001 (Meal Swap, safe alternatives list) for the emptied slot on {night}

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- name is required at recipe creation (FEAT-08, FEAT-10) |
| {night} | Planned Meal -- night | Thursday | Never empty -- night is required on every Planned Meal |

## Delivery Rules

**Batching:** None -- each report produces exactly one acknowledgement to its own reporter; two separate reports (even from the same reporter on the same day) each produce their own acknowledgement, since each report is submitted as a distinct interaction the reporter is actively completing.
**Deduplication:** At most one acknowledgement per report submission. FEAT-02.SPEC-004's duplicate-report handling (an existing open report on the same recipe) does not suppress this acknowledgement for a genuinely new reporter -- each reporting adult receives their own acknowledgement for their own submission, even if the recipe was already reported by someone else.
**Retry on failure:** N/A -- this is rendered directly within the same screen interaction (FEAT-02.SPEC-001) that produced it; there is no separate delivery channel to retry, since it is not a message sent elsewhere but the screen's own state change.
**Expiry:** N/A -- the acknowledgement is shown once, within the dialog, immediately upon successful submission; it does not wait to be delivered and cannot become stale.

## Edge Cases

- **The reporting adult closes the dialog before reading the acknowledgement fully** -- No re-delivery occurs; the meal's removal is already reflected on the calling screen (the emptied slot), so the reporter can confirm the outcome there even if the dialog's acknowledgement was dismissed quickly.
- **The report is a duplicate against an already-open concern raised by the other adult** -- The current reporter still receives their own acknowledgement worded identically, since from their perspective they successfully reported and the meal is (or already was) removed.
- **The submission itself fails (network error)** -- No acknowledgement is shown; FEAT-02.SPEC-001's own error state ("Couldn't submit your report...") applies instead, and this notification only fires on a successful FEAT-02.SPEC-004 outcome.
- **The reporting adult's session expires between submission and the acknowledgement rendering** -- This cannot occur in practice, since the acknowledgement is the direct synchronous result of a submission that itself required an active session; if the session expired, the submission itself would have failed first per FEAT-02.SPEC-001's Access and Visibility table.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggered by (inbound) | A processed report fires this acknowledgement |
| FEAT-02.SPEC-001 (Report a Safety Concern) | References (inbound) | The acknowledgement renders within this screen's dialog |
| FEAT-04.SPEC-001 (Meal Swap alternatives list) | Navigation (outbound) | The CTA deep-links here for the emptied slot |

## Analytics and Success Signals

- **reporter_acknowledgement_shown** (reporter role) -- supports success-metrics.md: "Zero Allergy Incidents"
- **reporter_acknowledgement_cta_tapped** (destination: meal_swap) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-011-AC-01:** Given Maya submits a safety concern report, when FEAT-02.SPEC-004 processes it successfully, then she sees the acknowledgement "Report received" with the body naming the recipe and confirming its removal.

**FEAT-02.SPEC-011-AC-02:** Given Sam submits a safety concern report, when it processes successfully, then he -- and only he, not Maya -- sees the acknowledgement.

**FEAT-02.SPEC-011-AC-03:** Given Maya sees the acknowledgement, when she taps "Show me alternatives", then she is taken to the safe alternatives list (FEAT-04.SPEC-001) for the emptied slot.

**FEAT-02.SPEC-011-AC-04:** Given a recipe already has an open report from Maya, when Sam later submits his own report on the same recipe, then Sam still receives his own acknowledgement worded the same as a first-time report.

**FEAT-02.SPEC-011-AC-05:** Given a submission fails due to a network error, when FEAT-02.SPEC-001 shows its error state, then this acknowledgement does not appear.

**FEAT-02.SPEC-011-AC-06:** Given no preference control exists for this acknowledgement, when any reporting adult submits a report, then the acknowledgement always appears -- there is no way to turn it off.

**FEAT-02.SPEC-011-AC-07:** Given the acknowledgement is an in-app, in-session confirmation, when it is shown, then no email or push notification is sent for it.

**FEAT-02.SPEC-011-AC-08:** Given the reporting adult dismisses the dialog immediately after the acknowledgement appears, when they return to the plan, then the emptied slot itself confirms the removal, independent of whether they read the acknowledgement text.

**FEAT-02.SPEC-011-AC-09:** Given two reports are submitted by the same reporter on two different meals on the same day, when each is processed, then each produces its own acknowledgement -- they are never batched into one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry N/A, expiry N/A) | 4 |
| Edge Cases | 4 | 4 |
