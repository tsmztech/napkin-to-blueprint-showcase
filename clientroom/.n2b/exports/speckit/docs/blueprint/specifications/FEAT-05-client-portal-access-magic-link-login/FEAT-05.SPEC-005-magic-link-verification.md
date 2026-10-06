---
document_type: spec
spec_type: automation
spec_id: FEAT-05.SPEC-005
spec_name: Magic Link Verification
spec_slug: magic-link-verification
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Magic Link Verification

## Overview

**Name:** Magic Link Verification
**ID:** FEAT-05.SPEC-005
**Type:** Automation
**Purpose:** Validates a clicked link, creates the scoped session, and records the sign-in.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Validating the clicked token against FEAT-05.SPEC-006's single-use and time-limit rules
- Checking the token resolves to a contact and scope the requesting browser is entitled to see (FEAT-05.SPEC-007)
- Marking the token used and creating the scoped portal session on success
- Updating the Client Contact's `last_sign_in` timestamp on success
- Triggering the Activity Log Entry for the sign-in (FEAT-13)

**Non-Goals:**
- Generating the token in the first place, or invalidating a prior one on re-request -- owned by FEAT-05.SPEC-004 (Magic Link Issuance); this automation only consumes a token FEAT-05.SPEC-004 already produced.
- Defining single-use, time-limit, and recognized-contact rules -- owned by FEAT-05.SPEC-006, enforced here rather than re-derived.
- Defining client isolation and role-scoped display -- owned by FEAT-05.SPEC-007; this automation applies that spec's scoping when creating the session, but does not define the scoping logic itself.
- Rendering the verification progress, success transition, or error explanation -- owned by FEAT-05.SPEC-002 (Link Verification Landing), the sole screen that triggers and displays the outcome of this automation.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Contact opens the emailed link | FEAT-05.SPEC-002 (Link Verification Landing) | Fires automatically when the screen loads with a token | The token embedded in the clicked link |

## Processing Logic

1. Receive the token from the triggering screen.
2. Look up the token among issued Sign-in link/tokens. If no such token exists, proceed to the Invalid outcome.
3. If the token exists, check its current state per FEAT-05.SPEC-006: already used, expired (past platform parameter: `magic-link-expiry-window` from issuance), or invalidated by a later re-request. If any of these apply, proceed to the corresponding outcome (each rendered identically by FEAT-05.SPEC-002, per that spec's Business Rules).
4. If the token is valid and unused, resolve the Client Contact it is bound to, and confirm that contact's `status` is still Active (a contact could be removed between issuance and click). If not Active, proceed to the Invalid outcome.
5. Apply FEAT-05.SPEC-007's isolation rules to determine the scope (the specific freelancer and client company) the resulting session is bound to.
6. Mark the token used (this is the single consuming read -- no other verification attempt against the same token can succeed afterward).
7. Create the scoped portal session, bound to the resolved Client Contact, freelancer, and client company.
8. Update the Client Contact's `last_sign_in` field to the current timestamp.
9. Trigger an Activity Log Entry for the sign-in, handed to FEAT-13 (XBR-05): event_type "client portal sign-in," actor the Client Contact, occurred_at now, affected_record the Client Contact, project none (a sign-in is account-level, not project-level).
10. Return the Success outcome, carrying the new session, to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Success | Token exists, is unused, unexpired, not invalidated, and its Client Contact is Active | Token marked used; scoped session created; Client Contact `last_sign_in` updated; Activity Log Entry written | Screen transitions from Verifying directly to Portal Home | FEAT-05.SPEC-002 (transition); FEAT-05.SPEC-003 (destination); FEAT-13 (trail entry) |
| Invalid -- unknown token | No matching token found | None | FEAT-05.SPEC-002 shows the plain "not valid anymore" explanation | FEAT-05.SPEC-002 |
| Invalid -- already used | Token exists but was already marked used by an earlier verification | None | Same plain explanation as unknown token | FEAT-05.SPEC-002 |
| Invalid -- expired | Token exists, unused, but past its expiry | None | Same plain explanation | FEAT-05.SPEC-002 |
| Invalid -- invalidated by re-request | Token exists, unused, unexpired, but superseded by a later issuance for the same contact (FEAT-05.SPEC-006) | None | Same plain explanation | FEAT-05.SPEC-002 |
| Invalid -- contact no longer Active | Token is otherwise valid, but the bound Client Contact's `status` is no longer Active | None | Same plain explanation | FEAT-05.SPEC-002 |
| Failure (processing error) | An internal error occurs during lookup, session creation, or the `last_sign_in` write | No partial state -- if the session cannot be created, the token is not marked used, so the contact retains the ability to retry the same click | FEAT-05.SPEC-002 shows its Offline/Degraded state when the failure is connectivity-related, or otherwise the same "not valid anymore" explanation as any invalid token, with the link remaining usable to retry since it was never consumed | FEAT-05.SPEC-002 |

## Data Model

**Reads:** Sign-in link/token -- state (issued, used, expired, invalidated), bound Client Contact reference; Client Contact -- `status`, freelancer and client company it belongs to.
**Creates:** Portal session (feature-internal) -- scoped to the resolved Client Contact, freelancer, and client company; Activity Log Entry -- via FEAT-13, on successful sign-in.
**Updates:** Sign-in link/token -- marked used; Client Contact -- `last_sign_in` set to the current timestamp.
**Deletes:** None.

## Business Rules

- XBR-05: every successful sign-in writes an append-only Activity Log Entry with actor and timestamp; this is the only record this automation contributes to the trail.
- FEAT-05.SPEC-006 governs every condition that makes a token invalid (unknown, used, expired, invalidated); this automation checks those conditions but does not redefine them.
- FEAT-05.SPEC-007 governs the scope the resulting session is bound to; a session is never created with a broader scope than the one Client Contact record it is bound to, even when that person holds contact records for other freelancers.
- Marking a token used (step 6) happens only once verification has otherwise fully succeeded up to that point; a token is never marked used and then subsequently fail this automation for a different reason, which would strand the contact with no usable link and no way to know why.

## Edge Cases

- **Token is opened twice in rapid succession (e.g., an email client pre-fetches the link, then the contact taps it)** -- Whichever verification reaches step 6 first marks the token used; the second verification finds the token already used and returns the Invalid -- already used outcome. To avoid stranding a legitimate contact behind an email pre-fetch, FEAT-05.SPEC-008's link is constructed so that automated pre-fetching by mail clients does not itself consume the token (the link requires the contact's own tap-through action, not merely being fetched as a preview resource).
- **Concurrent trigger firing (the same token opened on two devices at once)** -- Only one verification can mark the token used; per the business rule above, the losing attempt sees Invalid -- already used, never a partial or inconsistent session.
- **Trigger fires while a previous run for the same token is still in flight** -- The token's used-state check and used-state write are treated as a single atomic step (6); a second verification arriving before the first completes waits for that step to resolve and then evaluates the token's now-current state, guaranteeing at most one Success outcome per token.
- **Client Contact is removed (FEAT-18) after the link was issued but before it is clicked** -- Verification proceeds through steps 1-4 and returns Invalid -- contact no longer Active at step 4, before any session is created.
- **The token's bound client company is archived between issuance and click** -- The session is still created if the contact and token otherwise verify (archiving a client does not revoke an already-issued, unused, unexpired link); FEAT-05.SPEC-003 (Portal Home) then reflects that project's archived state normally rather than this automation blocking sign-in.
- **Verification succeeds but the Activity Log Entry hand-off to FEAT-13 fails** -- The session is still created and the contact still reaches Portal Home; the trail write is retried by FEAT-13's own delivery guarantee, since blocking a successful sign-in on a downstream logging failure would contradict this automation's own Success outcome already returned to the contact.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-002 (Link Verification Landing) | Triggered by (inbound) | Screen load with a token fires this automation; the outcome drives the screen's state |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Defines every condition checked in steps 2-4 |
| FEAT-05.SPEC-007 (Portal Access & Isolation Rules) | References (inbound) | Defines the scope applied when creating the session in step 5 |
| FEAT-05.SPEC-003 (Portal Home) | Affects (outbound) | Destination the contact reaches on Success |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Receives the sign-in trail entry (XBR-05) |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Supplies the Client Contact's current `status` checked in step 4 |

## Analytics and Success Signals

- **magic_link_used** (outcome: success) -- supports success-metrics.md: "Client Portal Login Success"
- **magic_link_expired** (reason: expired / already_used / invalidated_by_reissue / contact_inactive / unknown_token) -- supports success-metrics.md: "Client Portal Login Success" (a failed-or-expired attempt recoverable through one additional request is exactly what this metric's target measures)

## Acceptance Criteria

**FEAT-05.SPEC-005-AC-01:** Given Owen clicks a valid, unused, unexpired link addressed to his Active Client Contact record, when verification runs, then the token is marked used, his `last_sign_in` is updated, a sign-in Activity Log Entry is written, and he is delivered to FEAT-05.SPEC-003 (Portal Home).

**FEAT-05.SPEC-005-AC-02:** Given Priya clicks a link that has already been used, when verification runs, then the outcome is Invalid -- already used, and FEAT-05.SPEC-002 shows the plain explanation.

**FEAT-05.SPEC-005-AC-03:** Given Owen clicks a link after platform parameter: `magic-link-expiry-window` has elapsed since issuance, when verification runs, then the outcome is Invalid -- expired.

**FEAT-05.SPEC-005-AC-04:** Given Owen requests a new link while an old one is still unused, then clicks the old link, when verification runs, then the outcome is Invalid -- invalidated by re-request.

**FEAT-05.SPEC-005-AC-05:** Given Priya's Client Contact record was removed by FEAT-18 after her link was issued, when she clicks the link, then the outcome is Invalid -- contact no longer Active, and no session is created.

**FEAT-05.SPEC-005-AC-06:** Given a fabricated or guessed token that was never issued, when verification runs, then the outcome is Invalid -- unknown token, shown identically to any other invalid outcome.

**FEAT-05.SPEC-005-AC-07:** Given a valid token is opened on two devices at effectively the same moment, when both verifications run, then exactly one succeeds and the other returns Invalid -- already used.

**FEAT-05.SPEC-005-AC-08:** Given an email client pre-fetches Owen's sign-in link as a preview without his own tap, when the pre-fetch occurs, then the token is not consumed, and Owen's own subsequent tap still verifies successfully.

**FEAT-05.SPEC-005-AC-09:** Given verification succeeds but the Activity Log Entry hand-off to FEAT-13 fails, when this occurs, then Owen still reaches Portal Home normally, and the trail entry is retried by FEAT-13 rather than blocking his sign-in.

**FEAT-05.SPEC-005-AC-10:** Given a processing error occurs before the token is marked used, when the error occurs, then the token remains valid and unused, and the contact can retry the same link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
