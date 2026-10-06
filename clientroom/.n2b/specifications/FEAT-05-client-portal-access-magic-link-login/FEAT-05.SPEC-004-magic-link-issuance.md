---
document_type: spec
spec_type: automation
spec_id: FEAT-05.SPEC-004
spec_name: Magic Link Issuance
spec_slug: magic-link-issuance
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Magic Link Issuance

## Overview

**Name:** Magic Link Issuance
**ID:** FEAT-05.SPEC-004
**Type:** Automation
**Purpose:** Generates a single-use, time-limited sign-in link for a recognized contact and invalidates any prior unused link for that contact.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Looking up the submitted email against recognized Client Contacts (FEAT-18)
- Generating a single-use, time-limited token when the email matches a recognized, active contact
- Invalidating any prior unused token for that contact before issuing the new one
- Handing the issued link to FEAT-05.SPEC-008 (Magic Link Sign-In Email) for delivery
- Returning the same neutral outcome to the triggering screen regardless of whether the email matched

**Non-Goals:**
- Verifying a clicked link, creating the session, or recording the sign-in -- owned by FEAT-05.SPEC-005 (Magic Link Verification), a distinct automation with its own trigger and outcomes.
- Defining what "single-use," "time-limited," and "invalidate prior unused links" mean in detail -- owned by FEAT-05.SPEC-006 (Link Validity & Recognition Rules), the single source of truth this automation enforces rather than re-derives.
- Sending the email itself -- owned by FEAT-05.SPEC-008 (Magic Link Sign-In Email); this automation only produces the link and triggers that notification.
- Creating, inviting, or removing Client Contacts -- owned entirely by FEAT-18 per the Entity-Lifecycle Coverage Matrix; this automation only reads the existing contact roster to check recognition.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Contact submits a sign-in request | FEAT-05.SPEC-001 (Request Sign-In Link) | Fires on every validly formatted email submission, regardless of recognition outcome | Submitted email |
| Contact requests a fresh link after an expired/invalid one | FEAT-05.SPEC-002 (Link Verification Landing) | Fires when the contact taps "Send me a new link" and an email is known from the failed link's context | The email known from the expired link's context |

## Processing Logic

1. Receive the submitted email from the triggering screen.
2. Look up an active Client Contact record (status: Active) whose `email` field matches the submitted email, scoped to any freelancer's client roster (a person may be a contact for more than one freelancer, each under a separate Client Contact record per the dependency map's Relationships note).
3. If no active, recognized contact matches: take no further action beyond returning the neutral outcome to the triggering screen (Outcome: No Match). No token is generated and no email is sent.
4. If one or more active, recognized contact records match (the same email held by more than one freelancer's roster): issue a separate token per matching Client Contact record, one per freelancer, each following the remaining steps independently.
5. For each matching Client Contact: check whether an existing unused, unexpired token already exists for that contact (per FEAT-05.SPEC-006). If one exists, mark it invalidated before proceeding.
6. Generate a new single-use token bound to that Client Contact record, with an expiry set to the current time plus platform parameter: `magic-link-expiry-window` (FEAT-05.SPEC-006).
7. Hand the issued token and its associated Client Contact and Branding Profile (for the owning freelancer) to FEAT-05.SPEC-008 (Magic Link Sign-In Email) for delivery.
8. Return the neutral "request received" outcome to the triggering screen, identical to the No Match outcome in step 3.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Link issued (one match) | Submitted email matches exactly one active Client Contact | A new token is created; any prior unused token for that contact is invalidated | Same neutral confirmation as No Match -- no distinguishable feedback | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 (triggering screen); FEAT-05.SPEC-008 (email delivery) |
| Link issued (multiple matches, one contact is a client of several freelancers) | Submitted email matches active Client Contact records under more than one freelancer | A new token is created and any prior unused token invalidated for each matching contact record independently | Same neutral confirmation; each freelancer's email arrives as its own message (FEAT-05.SPEC-008), each usable only for that freelancer's portal | FEAT-05.SPEC-001 or FEAT-05.SPEC-002; FEAT-05.SPEC-008 (once per matching contact) |
| No match | Submitted email matches no active Client Contact | None | Same neutral confirmation as a successful issuance -- the contact cannot tell recognition failed | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 |
| Match found but contact is Removed | Submitted email matches a Client Contact whose `status` is Removed | None -- treated identically to No Match | Same neutral confirmation | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 |
| Automation failure | Processing error while looking up recognition or generating the token | None persisted -- the operation is atomic per contact match | Same neutral confirmation is still shown (the triggering screen never surfaces an issuance failure, since doing so would itself leak whether the email matched); the request is treated as failed to deliver and no email attempt is made | FEAT-05.SPEC-001 or FEAT-05.SPEC-002 |

## Data Model

**Reads:** Client Contact -- `email`, `status`, the freelancer and client company it belongs to; Sign-in link/token (feature-internal) -- any existing unused, unexpired token for the matched contact.
**Creates:** Sign-in link/token (feature-internal, not a Domain Entity Inventory record) -- one per matched, active Client Contact, per FEAT-05.SPEC-006.
**Updates:** Sign-in link/token -- marks any prior unused token for the matched contact as invalidated.
**Deletes:** None.

## Business Rules

- XBR-28: only Client Contacts added through FEAT-18 and currently Active are ever issued a link; a Removed contact's email is treated identically to an unrecognized one.
- FEAT-05.SPEC-006 governs single-use, time-limit, and invalidate-on-re-request behavior end to end; this automation enforces those rules rather than defining its own token lifetime or reuse logic.
- The neutral outcome (step 8) is returned identically whether or not a match was found, whether one or several freelancers matched, and even on an internal processing failure -- this automation never produces a triggering-screen-visible signal that would let a submitted email be tested for recognition.
- A person who is a Client Contact for several freelancers receives one issued link per freelancer, each scoped and invalidated independently -- issuing or invalidating one freelancer's link never affects another's (FEAT-05.SPEC-007).

## Edge Cases

- **Submitted email has mixed case or surrounding whitespace** -- Comparison against Client Contact `email` is case-insensitive and whitespace-trimmed, consistent with `email` being unique within a client company per the dependency map.
- **Contact is re-added (Removed, then re-invited) between two requests** -- The current `status` at the moment of this request governs; a request made while Removed yields No Match, and a later request made after re-activation issues a link normally.
- **Contact requests a new link while an unused token from the previous request is still valid** -- The prior token is invalidated as part of step 5 before the new one is generated, per FEAT-05.SPEC-006; the previous email's link, if since opened, shows the "not valid anymore" explanation (FEAT-05.SPEC-002).
- **Concurrent trigger firing (the same contact submits two requests within moments, e.g., a double-click across two browser tabs)** -- Each request runs independently; whichever completes last wins the invalidation race, since it invalidates whatever token existed when it read state, and its own newly generated token remains the valid one. At most one token per contact survives the pair.
- **Trigger fires while a previous run for the same contact is still in flight** -- The dependency-invalidate step reads the token state at the moment it runs, so a second request that starts before the first finishes generating its token will (once it reaches step 5) invalidate whichever token exists at that instant, including one generated moments earlier by the still-finishing first run; the end state always has at most one valid token per contact, consistent with the single-valid-link business rule above.
- **Delivery capability (FEAT-14) is unavailable when handing off to FEAT-05.SPEC-008** -- The token is still created; delivery failure and retry behavior belong to FEAT-05.SPEC-008 and the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001), not to this automation, which completes once the hand-off is made.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-001 (Request Sign-In Link) | Triggered by (inbound) | Every submitted request fires this automation |
| FEAT-05.SPEC-002 (Link Verification Landing) | Triggered by (inbound) | The one-tap re-request (when the email is known from context) fires this automation |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Defines token lifetime, single-use, and invalidation rules enforced here |
| FEAT-05.SPEC-008 (Magic Link Sign-In Email) | Affects (outbound) | The issued token and contact/branding context are handed off for delivery |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Recognition is checked against the Client Contact roster FEAT-18 maintains |

## Analytics and Success Signals

- **magic_link_issued** (matched_freelancer_count: 1 or more) -- supports success-metrics.md: "Client Portal Login Success"
- **magic_link_issuance_no_match** () -- N/A -- no Stage 2 metric measures unrecognized-email attempts directly, and this event is never surfaced to the requester (the neutral outcome rule above); retained internally so recognition failures are distinguishable from delivery failures when diagnosing login success shortfalls.

## Acceptance Criteria

**FEAT-05.SPEC-004-AC-01:** Given Owen submits his own recognized, Active email on FEAT-05.SPEC-001, when the automation runs, then a new single-use token is generated and handed to FEAT-05.SPEC-008 for delivery.

**FEAT-05.SPEC-004-AC-02:** Given Priya submits an email that matches no Client Contact, when the automation runs, then no token is created and the triggering screen shows the identical confirmation as a successful issuance.

**FEAT-05.SPEC-004-AC-03:** Given Owen has an unused, unexpired token from a prior request, when he submits a new request, then the prior token is invalidated before the new token is generated.

**FEAT-05.SPEC-004-AC-04:** Given a person's email matches Active Client Contact records under two different freelancers, when the automation runs, then a separate token is issued for each freelancer, each independently scoped and each triggering its own FEAT-05.SPEC-008 email.

**FEAT-05.SPEC-004-AC-05:** Given a Client Contact whose `status` is Removed submits their email, when the automation runs, then the outcome is identical to No Match -- no token is issued.

**FEAT-05.SPEC-004-AC-06:** Given Priya submits her email with extra whitespace and different letter casing than stored, when the automation runs, then it still matches her Client Contact record and issues a token.

**FEAT-05.SPEC-004-AC-07:** Given Owen submits two requests within moments of each other from two open tabs, when both complete, then exactly one valid token remains for Owen's contact record.

**FEAT-05.SPEC-004-AC-08:** Given the automation encounters an internal processing error while checking recognition, when it fails, then the triggering screen still shows the same neutral confirmation as a successful request, and no email is sent.

**FEAT-05.SPEC-004-AC-09:** Given Priya taps "Send me a new link" on FEAT-05.SPEC-002 with her email known from context, when the automation runs, then it processes identically to a fresh submission from FEAT-05.SPEC-001, including invalidating her prior token.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
