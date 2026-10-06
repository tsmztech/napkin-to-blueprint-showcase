---
document_type: spec
spec_type: integration
spec_id: FEAT-07.SPEC-006
spec_name: Plan-Ready Email Fallback Integration
spec_slug: plan-ready-email-fallback-integration
parent_feature: FEAT-07
parent_feature_name: Weekly Plan Ready Notification
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Integration Spec: Plan-Ready Email Fallback Integration

## Overview

**Name:** Plan-Ready Email Fallback Integration
**ID:** FEAT-07.SPEC-006
**Type:** Integration
**Purpose:** The product delivers the plan-ready message by transactional email to a member whose device notifications are unavailable or disabled, so the weekly rhythm still reaches them.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- Sending the plan-ready message's Email variant to a member resolved to the Email channel
- Retry behavior for a transient send failure
- User-facing behavior when this capability is slow, unavailable, or a send is permanently rejected (bounced or invalid address)
- Disclosure of what data is shared with this capability for this route

**Non-Goals:**
- Choosing the transactional email vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate.
- Deciding which members resolve to the Email channel -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec only carries out delivery once a member has already been resolved to Email.
- The exact email subject and body content -- owned by FEAT-07.SPEC-002 (Plan-Ready Notification Message); this spec transports that content, it does not author it.
- Account sign-up, sign-in recovery, billing, data-export, deletion, or safety-report email -- those routes on the same transactional email capability are owned by FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-14.SPEC-012, and FEAT-18.SPEC-012 respectively; this spec owns only the plan-ready fallback route and does not duplicate any of those other routes' behavior, per the Feature Dependency Map's Cross-Feature Touchpoints entry distinguishing this spec from FEAT-01.SPEC-017.

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional email capability -- Required for account sign-up and sign-in recovery, the plan-ready email fallback, billing and grace-period notices, data-export and deletion confirmations, safety-concern reports reaching the operator, and support acknowledgements; without it, several account, billing, and safety messages would have no reliable route." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this feature's own route: FEAT-07.SPEC-006, plan-ready email fallback)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A member without device notifications available or enabled still receives "next week's plan is ready" by email | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |
| Every eligible member in a household still receives the plan-ready message the week the device-notification capability itself is unavailable | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient email address | Member Profile -- sign_in (email) | Each dispatch to a member resolved to the Email channel | The capability needs an address to deliver to |
| Recipient display name | Member Profile -- display_name | Each such dispatch | Personalizes the greeting per FEAT-07.SPEC-002's content template |
| Fixed message content (subject and body text) | FEAT-07.SPEC-002's content templates -- no other entity fields | Each such dispatch | The capability needs the exact text to send |

Household plan contents, meal names, budget figures, dietary rules, and every Member Profile field beyond email and display name never leave the product through this route.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the result of a send attempt | No dependency-map entity is updated; the outcome feeds only this spec's own retry logic, since a household's Weekly Plan visibility never depends on it (XBR-12) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Send succeeded | The capability confirms the email was delivered | None | None beyond the email itself now being in the member's inbox | FEAT-07.SPEC-002 |
| Send failed (transient) | The capability reports a temporary failure | None -- retry scheduled per FEAT-07.SPEC-002's Delivery Rules (up to 3 retries over 6 hours) | No user feedback during retry; the plan is already visible in-app | FEAT-07.SPEC-002 |
| Send failed (permanent -- invalid or hard-bounced address) | The capability reports the address cannot receive mail at all | None -- no further retry is attempted for this week's dispatch | No user-facing error that week; the plan remains visible in-app regardless (XBR-12), and the member sees it the next time they open the app | FEAT-07.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | The send is queued and retried per FEAT-07.SPEC-002's Delivery Rules; the household's plan remains fully visible in-app throughout, with no in-app waiting indicator for this background path. | No fallback email is sent to the affected member(s) that week; the plan remains fully visible in-app immediately (XBR-12), and no error is shown to the household -- the affected member simply sees the plan the next time they open the app. | A hard bounce or invalid-address rejection ends that week's dispatch attempt with no further retry; the plan remains visible in-app, and there is no separate user-facing error, since a member without a working address for this week's send already reaches the plan by opening the app directly. |

## Consent and Disclosure

- **Email established as the account's contact channel** -- Each adult's sign-in email is provided during account creation (FEAT-01), at which point it is established as the account's contact channel for the product's transactional messages. This spec adds no separate disclosure moment beyond that: sending "next week's plan is ready" to an address the member already provided for their own account, when their plan-ready preference is on, stays within that address's existing purpose.
- **What is never shared** -- Household plan contents, meal names, budget figures, and dietary information are never included; only the fixed subject and body naming that the plan is ready, addressed with the member's own display name, ever leaves the product through this route.

## Edge Cases

- **A duplicate send event fires for the same household-week** -- FEAT-07.SPEC-002's deduplication guarantee (at most one message per member per household-week) prevents a second email from reaching the member even if the underlying send call is retried by the capability.
- **A send event arrives for a member removed from the household before delivery completes** -- The event is discarded on arrival; a former member's address is never used to deliver a household's plan-ready message.
- **The capability goes down mid-send while dispatching to multiple email-resolved members in the same household** -- Each recipient's send is independent; a failure for one member does not affect delivery to another.
- **Events arrive out of order (a failure notice arrives after a success notice for the same dispatch)** -- The most recent event by event time governs; a stale failure notice arriving after a confirmed success changes nothing, and no duplicate or corrective email is sent.
- **A member's email address is changed on their Member Profile between dispatch and delivery** -- The dispatch already in flight uses the address read at send time; a mid-flight address change takes effect starting with the following week's dispatch, not retroactively for the one already sent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Triggered by (inbound) | Hands off Email-channel dispatches to this integration |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Affects (outbound) | Send, retry, and failure outcomes govern what that spec's Delivery Rules describe |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | References (inbound) | Reads this integration's role as the fallback channel when resolving each member's channel |
| FEAT-01.SPEC-017 (Account Sign-Up & Sign-In Recovery Email) | Cross-reference | Shares the same transactional email capability for a distinct route (account/recovery); this spec owns only the plan-ready fallback route |

## Analytics and Success Signals

- **plan_ready_email_sent** (generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_email_delivery_failed_permanent** (reason: bounced / invalid_address) -- N/A -- no Stage 2 metric tracks permanent email-delivery failure specifically for this route; retained so a member who silently stops receiving this message by email is observable rather than invisible.

## Acceptance Criteria

**FEAT-07.SPEC-006-AC-01:** Given Sam is resolved to the Email channel for this week's message, when the send completes successfully, then he receives the email with the subject "Next week's plan is ready."

**FEAT-07.SPEC-006-AC-02:** Given a send to Sam fails transiently, when it is retried within the 6-hour window and the retry succeeds, then he receives exactly one email.

**FEAT-07.SPEC-006-AC-03:** Given a send to Sam fails on every retry within the 6-hour window, when the final retry fails, then no further attempt is made that week, and Sam sees the plan the next time he opens the app with no error shown.

**FEAT-07.SPEC-006-AC-04:** Given the capability reports Sam's address as permanently invalid, when that report is received, then no further retry occurs and no user-facing error appears anywhere in the product.

**FEAT-07.SPEC-006-AC-05:** Given the capability is slow to respond, when a send is attempted, then it is queued and retried per FEAT-07.SPEC-002's Delivery Rules while the household's plan remains fully visible in-app.

**FEAT-07.SPEC-006-AC-06:** Given the capability is down for the entire dispatch window, when email-resolved members are due to be sent this week's message, then no email is sent to them that week and no error is shown to the household.

**FEAT-07.SPEC-006-AC-07:** Given Sam's sign-in email was already provided during his own account setup, when this route sends him the plan-ready email, then no separate disclosure prompt is required beyond that original account setup.

**FEAT-07.SPEC-006-AC-08:** Given a duplicate send event fires for a household-week that has already delivered its message, when the duplicate is processed, then no second email reaches the member.

**FEAT-07.SPEC-006-AC-09:** Given Sam is removed from the household after his dispatch was queued but before a send event for it arrives, when that event arrives, then it is discarded.

**FEAT-07.SPEC-006-AC-10:** Given the capability goes down partway through sending to a household with two email-resolved members, when one member's send already succeeded, then that success stands and only the remaining member's send is affected.

**FEAT-07.SPEC-006-AC-11:** Given only the plan-ready message's fixed text, recipient email, and display name are ever sent to this capability, when any dispatch occurs, then no meal, budget, or dietary data is included in what leaves the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 3 (slow, down, rejects) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
