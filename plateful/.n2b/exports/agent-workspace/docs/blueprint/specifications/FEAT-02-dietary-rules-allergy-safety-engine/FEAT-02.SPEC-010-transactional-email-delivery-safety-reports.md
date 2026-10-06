---
document_type: spec
spec_type: integration
spec_id: FEAT-02.SPEC-010
spec_name: Transactional Email Delivery (Safety Reports)
spec_slug: transactional-email-delivery-safety-reports
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Transactional Email Delivery (Safety Reports)

## Overview

**Name:** Transactional Email Delivery (Safety Reports)
**ID:** FEAT-02.SPEC-010
**Type:** Integration
**Purpose:** Delivers the operator-facing safety-concern alert and the household's resolution notice through the product's transactional email capability.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Sending the safety-concern report email to the operator (Riley) for review
- Sending the household's resolution notice by email where email is that notification's delivery channel
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send for either of these two safety-report emails
- Disclosure of what data these two emails carry to the email capability

**Non-Goals:**
- Choosing the email vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate.
- Any other transactional email the product sends (account & recovery, plan-ready fallback, billing confirmations, export/deletion/support-acknowledgement) -- those are owned by FEAT-01.SPEC-017, FEAT-07.SPEC-006, FEAT-14.SPEC-012, and FEAT-18.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table; this spec covers only the two safety-report emails FEAT-02 originates.
- Composing the exact subject/body content of the two emails -- owned by FEAT-02.SPEC-013 (Safety Concern Operator Alert) and FEAT-02.SPEC-014 (Safety Concern Resolution Notice); this spec defines only the delivery contract those notifications rely on.
- In-app or push delivery of these notices -- FEAT-02.SPEC-011 through FEAT-02.SPEC-014 define their own channel mixes; this spec covers the email channel specifically.

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Requires a transactional email capability" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec is the FEAT-02 entry: "FEAT-02.SPEC-010 (safety reports)")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley receives the safety-concern report by email so it can be reviewed without being signed into the product's own screens | Report a safety concern | FEAT-02.SPEC-013 (Safety Concern Operator Alert) |
| Maya and Sam receive the household's resolution outcome by email if they are not reachable in-app at the moment it resolves | Report a safety concern (resolution notice) | FEAT-02.SPEC-014 (Safety Concern Resolution Notice) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Reported recipe name and ingredient list | Recipe -- name, ingredients | A safety-concern Support Request is created | The operator needs the recipe's content to review the reported concern |
| Reporter's note (if provided) | Support Request -- note | A safety-concern Support Request is created | The operator needs the reporter's own description of the concern |
| Reporting member's role and first name | Member Profile -- display_name, member_type | A safety-concern Support Request is created | The operator needs to know who raised the concern and in what capacity (organiser or other adult) |
| Household reference and reported member's age band (kid profile only, when the reported concern involves a kid's allergy) | Household -- household_name; Member Profile -- age_band | A safety-concern Support Request is created | The operator needs enough context to locate the household and understand which member's allergy is at issue; per the Access Matrix notes, this is the one context in which the operator sees a kid's allergy detail |
| Resolution outcome (recipe confirmed safe / kept excluded) | Support Request -- status, resolution outcome | A safety-concern Support Request is resolved | The household needs to know the outcome of its report |
| Recipient's first name and email | Member Profile -- display_name, sign_in (email) | Either email is sent | The email capability must know where and to whom to address the message |

No payment details, no Weekly Plan or Grocery List content beyond the single reported recipe, and no other household member's Dietary Rule data ever leaves the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The email capability reports the result of a send attempt | No entity field is updated by this outcome alone -- it drives only this spec's own retry logic (see Delivery Rules in FEAT-02.SPEC-013 and FEAT-02.SPEC-014); Support Request status is never derived from email delivery outcome |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Send succeeded | The email capability confirms the operator alert or resolution notice was accepted for delivery | None | No user-facing confirmation is shown for a successful transactional send -- the household and operator experience is simply "the email arrives" | FEAT-02.SPEC-013, FEAT-02.SPEC-014 |
| Send failed (transient) | The email capability reports a temporary failure (e.g., a momentary outage) | None -- retry is scheduled per FEAT-02.SPEC-013's and FEAT-02.SPEC-014's own Delivery Rules | No user feedback during retry -- the in-app safety-concern flow (FEAT-02.SPEC-001, FEAT-02.SPEC-004) already confirmed the meal's removal independent of email delivery | FEAT-02.SPEC-013, FEAT-02.SPEC-014 |
| Send failed (permanent, e.g., invalid recipient address) | The email capability reports the address cannot receive mail | None to Support Request; the failure is recorded for operational visibility only | No household- or operator-facing feedback -- since the operator alert has no fallback recipient, a permanent failure here is treated as an operational incident (see Edge Cases), not a user-facing message | FEAT-02.SPEC-013, FEAT-02.SPEC-014 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-02.SPEC-001 (Report a Safety Concern) | N/A -- this screen's own submission (meal removal, Support Request creation) does not wait on email delivery; the confirmation shown to the reporter is independent of this integration's state. | N/A -- same as above; the report is recorded and the meal removed regardless of the email capability's availability. | N/A -- the screen sends no request to this capability directly; FEAT-02.SPEC-004 triggers the email asynchronously after the screen's own confirmation. |
| FEAT-22.SPEC-001 (Support Request Queue, FEAT-22) | The operator alert email may arrive later than usual; the underlying Support Request is visible to Riley in the Support View immediately regardless of email delivery timing. | The operator alert email does not arrive; Riley can still discover and review the open Support Request directly through the Support View, which does not depend on email. | N/A -- a rejection from the capability (e.g., malformed address, which cannot occur for a fixed operator address) is treated as a down-equivalent operational condition; the Support View remains the reliable path. |

## Consent and Disclosure

- **Safety-report data shared with the email capability** -- The household is not shown a separate consent prompt before this data is shared, since sending the operator alert is an inseparable part of submitting a safety concern (FEAT-02.SPEC-001's own confirmation states "your household's operator will review the ingredients," which discloses that the report -- including the recipe and any note -- reaches the operator). No further opt-out exists for this specific email, since it is core to the safety-review process the report itself initiates.
- **Resolution notice recipient disclosure** -- The household is told, as part of FEAT-02.SPEC-001's confirmation, that it will be told the outcome once reviewed; the resolution notice's email channel is one of its stated delivery channels (FEAT-02.SPEC-014), not a separately disclosed sharing event.
- **What is never shared** -- Payment details, the full Weekly Plan, the Grocery List, and any other household member's Dietary Rule data beyond the one member whose allergy is at issue in the specific report never leave the product through this integration.

## Edge Cases

- **The operator alert email fails permanently (e.g., the operator's configured address is temporarily unreachable)** -- Since Riley discovers open Support Requests directly through the Support View (FEAT-22) rather than solely through email, a permanent failure here does not block the review process; it is logged as an operational condition for the founder to notice, not surfaced to the household.
- **The same resolution event is delivered to the email capability twice due to a retry** -- FEAT-02.SPEC-014's deduplication rule (at most one resolution notice per resolved Support Request) prevents a duplicate email from reaching the household even if this integration's send call is retried.
- **An email event arrives for a Support Request that has since been superseded (e.g., resolved twice due to a data anomaly)** -- The email reflects the Support Request's status at the moment the send was triggered; this integration does not re-fetch status at delivery time, so a stale send is possible only if the underlying automation (FEAT-02.SPEC-005) itself fired twice, which its own concurrency handling prevents.
- **Capability goes down mid-send for the resolution notice** -- The household still has the resolution outcome visible in-app (FEAT-02.SPEC-014's other channel, if any) and via the Support Request's status; the email is retried per FEAT-02.SPEC-014's Delivery Rules once the capability recovers.
- **Operator alert and resolution notice for the same Support Request are queued for delivery at effectively the same time (a report resolved unusually quickly)** -- Each is a distinct message to a distinct recipient (operator vs. household) and is sent independently; no batching or merging occurs between the two, since they serve different audiences.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-013 (Safety Concern Operator Alert) | Triggered by (inbound) | The operator alert's email channel is delivered through this integration |
| FEAT-02.SPEC-014 (Safety Concern Resolution Notice) | Triggered by (inbound) | The resolution notice's email channel is delivered through this integration |
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggers (outbound, indirect) | Report creation is the originating event that leads to the operator alert send |
| FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Triggers (outbound, indirect) | Resolution is the originating event that leads to the resolution notice send |
| FEAT-22 (Operator Read-Only Support Access) | Affects (outbound) | The reliable fallback path when the operator alert email is degraded |

## Analytics and Success Signals

- **safety_report_email_sent** (recipient: operator / household; email: operator_alert / resolution_notice) -- N/A -- no Stage 2 metric measures safety-report email delivery specifically; retained so the reliability of this delivery path is observable rather than invisible, given the trust-critical nature of the feature it supports.
- **safety_report_email_failed** (recipient, failure type: transient / permanent) -- supports success-metrics.md: "Zero Allergy Incidents" (a household or operator who never receives a safety-report communication is a gap in the zero-incident trust promise, so failures here are tracked against that metric).

## Acceptance Criteria

**FEAT-02.SPEC-010-AC-01:** Given a safety-concern Support Request is created, when this integration sends the operator alert, then the recipe name, ingredients, reporter's role and first name, and any note are included in the email sent to the operator.

**FEAT-02.SPEC-010-AC-02:** Given a safety-concern Support Request is resolved, when this integration sends the resolution notice, then the household's recipient receives an email stating the outcome.

**FEAT-02.SPEC-010-AC-03:** Given the transactional email capability is temporarily unavailable when a safety concern is reported, when FEAT-02.SPEC-001's submission completes, then the meal is still removed and the reporter still sees the confirmation, independent of the email capability's state.

**FEAT-02.SPEC-010-AC-04:** Given the operator alert email fails permanently, when Riley checks for open Support Requests, then the Support View (FEAT-22) still shows the report, since it does not depend on email delivery.

**FEAT-02.SPEC-010-AC-05:** Given a resolution-notice send is retried after a transient failure, when the retry succeeds, then only one resolution notice reaches the household, per FEAT-02.SPEC-014's deduplication rule.

**FEAT-02.SPEC-010-AC-06:** Given a household submits a safety concern, when they read FEAT-02.SPEC-001's confirmation, then it discloses that the operator will review the ingredients, which is the disclosure covering this integration's operator-alert data share.

**FEAT-02.SPEC-010-AC-07:** Given a reported concern involves a kid profile's allergy, when the operator alert is composed, then it includes only that kid's age band and the allergy detail relevant to the report -- never the kid's other profile data.

**FEAT-02.SPEC-010-AC-08:** Given no payment or full Weekly Plan data is ever part of a safety-report email, when either email is composed, then it contains only the recipe, note, reporter/recipient identity, and resolution outcome as defined in Data Exchanged.

**FEAT-02.SPEC-010-AC-09:** Given an operator alert and a resolution notice for the same Support Request become due at effectively the same time, when both are sent, then each is delivered independently to its own recipient with no merging.

**FEAT-02.SPEC-010-AC-10:** Given the email capability reports a permanent failure for a resolution-notice send, when the failure is recorded, then no household-facing error message appears, since the outcome remains visible through the product's own Support Request status.

**FEAT-02.SPEC-010-AC-11:** Given the email capability is down when a safety concern is reported, when it later recovers, then the queued operator alert is delivered per FEAT-02.SPEC-013's retry rules without being lost.

**FEAT-02.SPEC-010-AC-12:** Given the same resolution event triggers a duplicate send attempt due to a retry, when the second attempt is processed, then it changes nothing the household sees and no second email is delivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 screens x 3 conditions, 3 N/A cells justified and excluded from the count of active paths but listed) | 6 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |
