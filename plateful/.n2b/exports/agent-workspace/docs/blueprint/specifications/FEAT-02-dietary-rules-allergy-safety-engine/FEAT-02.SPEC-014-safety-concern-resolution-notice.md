---
document_type: spec
spec_type: notification
spec_id: FEAT-02.SPEC-014
spec_name: Safety Concern Resolution Notice
spec_slug: safety-concern-resolution-notice
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Safety Concern Resolution Notice

## Overview

**Name:** Safety Concern Resolution Notice
**ID:** FEAT-02.SPEC-014
**Type:** Notification
**Purpose:** Tells the household the outcome once the operator has resolved the report.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Notifying the household when a safety-concern Support Request is resolved
- Covering both resolution outcomes: recipe confirmed safe (released) and recipe confirmed unsafe (kept excluded)
- Content, channels, and delivery behavior across in-app and email

**Non-Goals:**
- Acknowledging the original report -- owned by FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement), a distinct, earlier message
- Alerting the organiser at removal time -- owned by FEAT-02.SPEC-012 (Safety Concern Organiser Alert), a distinct, earlier message
- Performing the review or writing the resolution outcome -- owned by FEAT-22 (Operator Read-Only Support Access) and FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), which triggers this notification
- The email transport mechanics -- owned by FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports); this spec defines content and trigger for its email channel, that spec defines the delivery contract

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always | The household's normal rhythm is checking the plan in-app; the resolution is a plan-relevant fact that belongs where the plan itself lives |
| Email | Always, in addition to in-app | A safety-concern resolution can arrive days after the original report, when the household may not be actively in the product; per the transactional-email External Touchpoint (ASMP-32), a safety-report resolution is one of the emails this capability delivers, ensuring the outcome reaches the household even if they have not opened the app since the report |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Safety-concern Support Request resolved | FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Fires once per resolution, for either outcome (released or kept excluded) | Recipe name, resolution outcome, reporting Member's name, household reference |

## Audience and Preferences

**Recipients:** Maya (Organiser) and Sam (Other Adult Member) -- the whole household's adults, per the Access Matrix's Safety Reports column (Maya: Full, Sam: Own-only), since the outcome affects the shared plan and candidate pool both adults act within, regardless of who originally reported.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a safety-outcome notice with no independent on/off control | -- | Always on | -- |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for household members, and a safety resolution is treated as timely enough to deliver whenever the review completes rather than being held for a preferred hour.

## Content Definition

**In-app:**
- **Title (released):** {recipe_name} is safe again
- **Body (released):** Our operator reviewed the safety concern about {recipe_name} and confirmed it's safe. It may appear in your plan again going forward.
- **Title (kept excluded):** {recipe_name} will stay off your plan
- **Body (kept excluded):** Our operator reviewed the safety concern about {recipe_name} and confirmed the concern was valid. This recipe will not be suggested to your household again.
- **CTA:** View recipe -- deep-links to FEAT-08.SPEC-002 (Recipe Detail View) for {recipe_name}

**Email:**
- **Subject (released):** Update on your safety report: {recipe_name} is safe
- **Subject (kept excluded):** Update on your safety report: {recipe_name} stays off your plan
- **Body:**
  Hi,

  You reported a safety concern about {recipe_name} on {report_date}. Our operator has reviewed it.

  {resolution_detail}

  You can review this and any other reports in your household settings.
- **CTA (button):** Open Plateful -- deep-links to the household's plan (FEAT-03)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- required at recipe creation |
| {report_date} | Support Request -- derived from its creation timestamp | September 20 | Never empty -- every Support Request has a creation time |
| {resolution_detail} | Derived from Support Request's resolution outcome | "Good news -- the recipe is confirmed safe and may appear in your plan again." or "The concern was valid, and this recipe will not be suggested to your household again." | Never empty -- the notice only sends once an outcome is recorded |

## Delivery Rules

**Batching:** None -- each resolved Support Request produces its own notice; two households' resolutions are always separate, and even two resolutions for the same household on different recipes are sent as separate notices, since each concerns a distinct recipe and reporter context.
**Deduplication:** At most one resolution notice per resolved Support Request. A Support Request cannot be resolved twice (FEAT-22 transitions status Under review -> Resolved exactly once), so no duplicate-resolution scenario can arise from the source data itself; if a delivery retry occurs, the notice is not re-sent once delivery is confirmed.
**Retry on failure:** In-app delivery has no retry -- it is shown the next time any household adult opens the product. Email delivery failure is retried per FEAT-02.SPEC-010's transactional email delivery contract; after retries are exhausted, the in-app notice stands as the delivery of record.
**Expiry:** The in-app notice does not expire -- it remains visible until an adult views it or navigates to the affected recipe; the resolution fact itself (the recipe's current eligibility) is also reflected the next time the recipe is considered as a candidate (FEAT-02.SPEC-002), so the outcome is never lost even if the notice itself goes unread.

## Edge Cases

- **The household is deleted before the notice can be delivered** -- No notice is delivered, consistent with FEAT-02.SPEC-005's edge case for a deleted household; there is no recipient left to notify.
- **The recipe was already edited or re-imported between the report and the resolution (FEAT-10)** -- The notice still names the original recipe; if released, the notice's "may appear in your plan again" wording holds true only once the edited recipe also passes the ordinary safety check again (per XBR-19), which this notice does not itself guarantee -- it reports the report's resolution, not a renewed pass/fail determination.
- **Both Maya and Sam are entitled recipients and both are actively using the app when the resolution completes** -- Each receives their own copy of the in-app and email notice; the notice is not deduplicated across recipients, only across separate resolution events for the same recipient.
- **The original reporter is no longer a household member (has left) by the time the resolution completes** -- The notice still goes to Maya and Sam as the current household adults, since the recipe's eligibility affects the household's ongoing plan regardless of who originally reported it; the departed member's own historical reporting is preserved per SC-18 but they receive no further notice as a non-member.
- **The recipe is kept excluded, and the household later tries to search for it directly in the Recipe Library** -- The recipe shows the ineligibility reason (FEAT-02.SPEC-008) consistent with a permanent exclusion, which is the lasting, always-current signal beyond this one-time notice.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Triggered by (inbound) | The resolution fires this notice |
| FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports) | References (outbound) | Delivery mechanics for the email channel |
| FEAT-08 (Recipe Library recipe detail) | Navigation (outbound) | The in-app CTA deep-links to the recipe's detail view |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (outbound) | The email CTA deep-links to the household's current plan |
| FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule) | References (outbound) | The lasting eligibility signal beyond this one-time notice |

## Analytics and Success Signals

- **resolution_notice_delivered** (channel: in_app / email; outcome: released / kept_excluded) -- supports success-metrics.md: "Zero Allergy Incidents"
- **resolution_notice_cta_tapped** (channel, destination: recipe_detail / plan_view) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-014-AC-01:** Given a safety-concern Support Request is resolved with a "confirmed safe" outcome, when this notice fires, then both Maya and Sam receive an in-app and email notice titled "{recipe_name} is safe again."

**FEAT-02.SPEC-014-AC-02:** Given a safety-concern Support Request is resolved with a "confirmed unsafe" outcome, when this notice fires, then both Maya and Sam receive an in-app and email notice titled "{recipe_name} will stay off your plan."

**FEAT-02.SPEC-014-AC-03:** Given Sam taps the in-app notice's "View recipe" CTA, when the tap registers, then he lands on the recipe's detail view.

**FEAT-02.SPEC-014-AC-04:** Given the household is deleted before the resolution completes, when FEAT-02.SPEC-005 processes the resolution, then no notice is delivered to anyone.

**FEAT-02.SPEC-014-AC-05:** Given a resolution's email delivery fails and retries are exhausted, when the household later opens the app, then the in-app notice still stands as the delivery of record.

**FEAT-02.SPEC-014-AC-06:** Given no preference control exists for this notice, when any resolution completes, then the notice is always delivered to Maya and Sam -- there is no way to turn it off.

**FEAT-02.SPEC-014-AC-07:** Given the original reporter has since left the household, when the resolution completes, then the notice is still delivered to the current household adults (Maya and Sam), not to the departed member.

**FEAT-02.SPEC-014-AC-08:** Given two households have concerns resolved at effectively the same time, when both notices are sent, then each household receives only its own notice, with no batching across households.

**FEAT-02.SPEC-014-AC-09:** Given a recipe is kept excluded and a household later searches for it in the Recipe Library, when they view it, then it shows the ineligibility reason consistent with a permanent exclusion (FEAT-02.SPEC-008).

**FEAT-02.SPEC-014-AC-10:** Given Maya and Sam are both actively using the app when a resolution completes, when the notice is delivered, then each receives their own independent copy of both the in-app and email notice.

**FEAT-02.SPEC-014-AC-11:** Given a Support Request can only be resolved once, when a resolution completes, then exactly one resolution notice is sent per Support Request -- never more.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
