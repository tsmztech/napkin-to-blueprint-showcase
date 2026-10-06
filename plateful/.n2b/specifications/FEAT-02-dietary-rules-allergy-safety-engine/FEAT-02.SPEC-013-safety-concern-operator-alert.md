---
document_type: spec
spec_type: notification
spec_id: FEAT-02.SPEC-013
spec_name: Safety Concern Operator Alert
spec_slug: safety-concern-operator-alert
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Safety Concern Operator Alert

## Overview

**Name:** Safety Concern Operator Alert
**ID:** FEAT-02.SPEC-013
**Type:** Notification
**Purpose:** Sends the safety-concern report to the operator by transactional email for review.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- The email alert sent to Riley (Operator) whenever a safety-concern Support Request is created
- Its content, delivery, and retry behavior over the transactional email capability

**Non-Goals:**
- Acknowledging the reporter -- owned by FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement)
- Telling the organiser -- owned by FEAT-02.SPEC-012 (Safety Concern Organiser Alert)
- Telling the household the resolution -- owned by FEAT-02.SPEC-014 (Safety Concern Resolution Notice)
- The email transport mechanics (send retries, capability degradation) -- owned by FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports); this spec defines the content and trigger, that spec defines the delivery contract
- Any in-app surface for the operator -- Riley has no in-app notification surface of his own in this feature; his review happens entirely through FEAT-22's Support View, which this alert's CTA points to

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always | Riley (Operator) is not a routine, signed-in user of the product's day-to-day screens (BRIEF.md, Target Users & Roles: "not a product role"); email is the reliable way to reach him for an on-demand review task, per the External Touchpoints table's transactional-email entry for FEAT-02 |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Safety-concern Support Request created | FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Fires once per newly created Support Request (not for a duplicate report against an already-open concern, since no new Support Request is created in that case) | Recipe name and ingredient list, reporter's role and first name, household name, reporting member's optional note, kid's age band and allergy detail if the concern involves a kid profile |

## Audience and Preferences

**Recipients:** Riley (Operator, support -- from v1) only, per the Access Matrix's Support View column (Full for Riley). No other role receives this alert.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a fixed operational alert with no recipient-controlled preference | -- | Always on | -- |

**Quiet Hours:** N/A -- the operator role has no quiet-hours concept in the product definition; a safety concern is treated as review-worthy whenever it is raised, consistent with the trust-critical nature of the feature.

## Content Definition

**Email:**
- **Subject:** Safety concern reported -- {household_name}
- **Body:**
  A household has reported a safety concern.

  Household: {household_name}
  Reported by: {reporter_name} ({reporter_role})
  Recipe: {recipe_name}
  Ingredients: {ingredient_list}
  Note from reporter: {reporter_note}

  Review this report in the Support View to record your findings.
- **CTA (button):** Review report -- deep-links to FEAT-22.SPEC-001 (Support Request Queue) with this household's open Support Request highlighted

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {household_name} | Household -- household_name | The Nguyen Family | Never empty -- required at household creation |
| {reporter_name} | Member Profile -- display_name | Maya | Never empty -- required on every Member Profile |
| {reporter_role} | Member Profile -- member_type | Organiser | Never empty -- required on every Member Profile |
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- required at recipe creation |
| {ingredient_list} | Recipe -- ingredients (names only, for review context) | Mushrooms, arborio rice, vegetable stock, parmesan | Never empty -- a recipe with zero ingredients cannot exist per FEAT-08/FEAT-10's "at least one ingredient to save" rule |
| {reporter_note} | Support Request -- note | "This has peanuts listed on the box but not in the app's ingredient list." | Renders as "No note provided." when the reporter left the note field empty |

## Delivery Rules

**Batching:** None -- each safety-concern Support Request is its own review case and is sent as its own email; batching two households' concerns into one email would slow the operator's per-household review and blur the audit trail.
**Deduplication:** At most one operator alert per Support Request. A duplicate report against an already-open concern (per FEAT-02.SPEC-004) does not create a new Support Request and therefore does not trigger a second alert.
**Retry on failure:** Governed by FEAT-02.SPEC-010's transactional email delivery contract -- transient failures retry per that spec's rules; regardless of email outcome, the Support Request remains visible and reviewable through FEAT-22's Support View.
**Expiry:** This alert does not expire in the sense of becoming unsendable -- there is no time window after which the report becomes not-worth-alerting-on, since a household's safety concern remains open and reviewable until resolved. If delivery is never confirmed, the underlying Support Request itself is the surviving signal Riley can find through the Support View.

## Edge Cases

- **Riley never opens the email** -- The Support Request remains Raised and visible in the Support View indefinitely until reviewed; this notification has no re-send or nagging behavior, since operator support access is on-demand per the feature's own scope, not a service-level-tracked queue.
- **Two households report safety concerns at effectively the same time** -- Each produces its own independent email; there is no cross-household batching.
- **A safety concern involves a kid profile's allergy** -- The email includes only that kid's age band and the specific allergy detail relevant to this report, never the kid's other profile data, per the Access Matrix notes on Riley's Kid Profile Data access.
- **The recipe's ingredient list is very long** -- The full ingredient list is included regardless of length, since the operator's review depends on seeing the complete list, not a truncated summary.
- **A second safety concern is reported for a different recipe by the same household while the first is still open** -- Each Support Request produces its own alert; the two are never merged, since they concern different recipes and may resolve independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggered by (inbound) | Report creation fires this alert |
| FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports) | References (outbound) | Delivery mechanics and degradation behavior for this email are defined there |
| FEAT-22 (Operator Read-Only Support Access) | Navigation (outbound) | The CTA deep-links to the Support View for this household's open request |

## Analytics and Success Signals

- **operator_alert_sent** (household reference) -- supports success-metrics.md: "Zero Allergy Incidents"
- **operator_alert_cta_tapped** -- N/A -- email link engagement by the operator is not tracked as a Stage 2 metric; retained only if the transactional email capability reports open/click data, which is not guaranteed, so this event is emitted only when available.

## Acceptance Criteria

**FEAT-02.SPEC-013-AC-01:** Given a safety-concern Support Request is created, when this notification fires, then Riley receives an email with the subject "Safety concern reported -- {household_name}" and the recipe, ingredients, reporter identity, and note in the body.

**FEAT-02.SPEC-013-AC-02:** Given the reporter left the note field empty, when the email is composed, then the note line renders as "No note provided."

**FEAT-02.SPEC-013-AC-03:** Given Riley taps "Review report" in the email, when the link opens, then he lands on the Support View (FEAT-22) for that household's open Support Request.

**FEAT-02.SPEC-013-AC-04:** Given a duplicate report is submitted against an already-open concern on the same recipe, when FEAT-02.SPEC-004 processes it, then no second operator alert is sent, since no new Support Request was created.

**FEAT-02.SPEC-013-AC-05:** Given the reported concern involves a kid profile's allergy, when the email is composed, then it includes only the kid's age band and the specific allergy detail, never other Kid Profile Data.

**FEAT-02.SPEC-013-AC-06:** Given two households report safety concerns at effectively the same time, when both are processed, then each produces its own independent email to Riley.

**FEAT-02.SPEC-013-AC-07:** Given Riley never opens the alert email, when he later checks the Support View directly, then the open Support Request is still fully visible and reviewable there.

**FEAT-02.SPEC-013-AC-08:** Given no preference control exists for this alert, when any safety concern is reported, then the alert is always sent -- there is no way for it to be turned off.

**FEAT-02.SPEC-013-AC-09:** Given the recipe has a long ingredient list, when the email is composed, then the full list is included without truncation.

**FEAT-02.SPEC-013-AC-10:** Given the transactional email capability is temporarily unavailable, when the send is attempted, then delivery is retried per FEAT-02.SPEC-010's rules, and the Support Request remains reviewable regardless of the email's delivery status.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
