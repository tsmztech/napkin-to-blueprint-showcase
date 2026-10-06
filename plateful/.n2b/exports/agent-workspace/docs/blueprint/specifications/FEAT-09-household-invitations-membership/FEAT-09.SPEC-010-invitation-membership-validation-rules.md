---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-010
spec_name: Invitation & Membership Validation Rules
spec_slug: invitation-membership-validation-rules
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 14
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Invitation & Membership Validation Rules

## Overview

**Name:** Invitation & Membership Validation Rules
**ID:** FEAT-09.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs the contact-detail requirement, re-invite blocking, the 14-day expiry window, hand-over-recipient eligibility, and the accept/revoke/expiry race resolution.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership
**Governed Entity:** Invitation (plus the hand-over eligibility conditions on Member Profile and Household.organiser)

## Scope and Non-Goals

**In Scope:**
- Field validation for the Invitation entity's contact_detail
- Re-invite blocking against existing active/invited members and outstanding invitations
- The 14-day expiry window's definition (its enforcement is FEAT-09.SPEC-006's)
- Hand-over-recipient eligibility (an Active adult member, never the current organiser)
- The race resolution between acceptance, revocation, and expiry on the same Invitation
- Default values for Invitation fields at creation

**Non-Goals:**
- Who may send, revoke, or hand over -- role-based authorization is governed by FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules), not this spec
- Sign-in email and password format for the invitee's new account -- governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules), a general account-field concern this feature references rather than duplicates
- Actually performing the expiry transition, the acceptance processing, or the hand-over transfer -- owned respectively by FEAT-09.SPEC-006, FEAT-09.SPEC-007, and FEAT-09.SPEC-009, which enforce the rules defined here

## Governed Entity

**Entity:** Invitation
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| contact_detail | text | The invitee's contact detail, entered by the organiser as a label for the invitation and for re-invite blocking |
| status | enum | Sent, Accepted, Revoked, or Expired |
| sent_by | reference | The organiser who created the invitation |
| sent_date | date | When the invitation was created (used to compute the 14-day expiry window) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Household Invitations Manager | On contact detail field blur and on Send Invitation submit (creation); on Revoke and Resend actions |
| FEAT-09.SPEC-002 | Invitation Acceptance | On screen load (invitation validity) and on Accept & Join submit (re-check) |
| FEAT-09.SPEC-006 | Invitation Expiry | On each scheduled run (expiry window and precedence against a concurrent accept/revoke) |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | On acceptance processing (race resolution) |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | On recipient selection and on Send Request submit (hand-over-recipient eligibility) |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | On Accept submit (whether the request is still outstanding) |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | On processing (re-check that the request is still outstanding) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| contact_detail | Required, non-empty | Always | On blur, on submit | "Enter an email address or phone number" | Yes |
| contact_detail | Must be a valid email address format OR a valid phone number format | Always | On blur | "Enter a valid email address or phone number" | Yes |
| contact_detail | Max 254 characters | Always | On blur | "That's too long -- enter a shorter email address or phone number" | Yes |
| status | No validation beyond data type | Always -- system-managed, never directly entered by any user | -- | -- | -- |
| sent_by | No validation beyond data type | Always -- system-assigned from the current session | -- | -- | -- |
| sent_date | No validation beyond data type | Always -- system-assigned at creation | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Re-invite blocking against active members | contact_detail | If the entered contact_detail matches the sign_in email of an existing Active Member Profile in this household, the invitation is blocked | "This person is already a member of your household." |
| Duplicate outstanding invitation | contact_detail, status | If the entered contact_detail exactly matches the contact_detail of another invitation with status Sent for this household, the new invitation is blocked | "There's already an outstanding invitation for this contact -- resend it instead of sending a new one." |

## Authorization Rules

N/A -- role-based authorization for every action this feature defines (send, revoke, resend, accept, hand over, leave) is governed entirely by FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules), which is this feature's single authoritative home for the role-action matrix. This spec governs only the entity-level and race-condition rules above.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Sent | On create only | No |
| sent_by | The current organiser's Member Profile reference | On create only | No |
| sent_date | Current date | On create only | No |
| expiry_date | Derived: sent_date + 14 days | Always, recalculated whenever sent_date is set (including on a fresh Sent invitation created by Resend) | No |

## Business Rules

- Invitations expire after 14 days from their sent date (feature-overview.md, Validation & Limits) -- this is a fixed, feature-level product decision, stated as a concrete number because it is specific to this feature's own definition, not a platform-wide policy value delegated to build time.
- A household may hold multiple outstanding (Sent) invitations at once (feature-overview.md, Validation & Limits) -- the duplicate-outstanding-invitation rule above blocks only an exact contact-detail repeat, not multiple invitations to different people.
- **Accept/revoke/expiry race resolution:** the Invitation entity's status transition follows first-committed-wins. Whichever of an acceptance (FEAT-09.SPEC-007), a revocation (FEAT-09.SPEC-001), or an automatic expiry (FEAT-09.SPEC-006) is durably recorded first against a given invitation determines its outcome; every other concurrent attempt against the same invitation is rejected and shown the invitation's now-current status (reject-with-refresh), per the dependency map's Contention note for Invitation.
- **Hand-over-recipient eligibility:** a hand-over recipient must be an Active adult member with member_type Other Adult Member; the current organiser can never be selected as her own recipient, and a member with status Invited, Left, or Removed is never eligible.
- **Hand-over request standing:** at most one outstanding hand-over request exists per household at a time (XBR-15); a request becomes non-outstanding when the organiser cancels it, the recipient accepts or declines it, or the recipient's eligibility is lost (e.g., they leave) while it is pending.
- An Expired invitation resend creates a new Invitation record (new sent_date, new expiry_date, status Sent) rather than reviving the expired one in place; the expired record is retained as-is.

## Edge Cases

- **Entered contact detail matches an Active member's sign_in email exactly but differs in letter case** -- The comparison is case-insensitive for email addresses; the invitation is still blocked as an already-active member.
- **Entered contact detail is a phone number with no matching Member Profile field to compare against** -- Re-invite blocking against active members applies only when the entered contact detail is in email format (the only contact channel stored on Member Profile.sign_in); a phone-number contact detail is checked only against other outstanding invitations' contact_detail for the duplicate-outstanding rule, never against member sign-in emails.
- **Invitation reaches exactly 14 days and 0 hours since its sent_date at the same moment a scheduled expiry run and an acceptance both arrive** -- First-committed-wins applies exactly as with any other race: whichever transition is recorded first stands, with no special-cased tie-break beyond commit order.
- **Organiser resends an invitation whose original contact_detail no longer matches any real recipient concern (e.g., a typo)** -- Resend uses the original invitation's contact_detail as entered; correcting a typo requires revoking the Expired invitation's record is unnecessary since it is already terminal, and sending a fresh invitation with the corrected contact detail is a new Send action on FEAT-09.SPEC-001, not a Resend.
- **Hand-over recipient list has exactly one eligible member and that member is mid-way through their own Leave Household confirmation (FEAT-09.SPEC-005) when selected** -- If the member's departure (FEAT-09.SPEC-008) is recorded before the hand-over request is sent, they no longer appear as eligible and the organiser's stale selection is rejected with refresh, showing the now-empty eligible-recipient state.
- **Duplicate-outstanding-invitation check runs against a contact detail that matches an invitation revoked seconds earlier** -- A Revoked invitation does not block a new Send; the duplicate check applies only to invitations currently in Sent status.

## Acceptance Criteria

**FEAT-09.SPEC-010-AC-01:** Given Maya leaves the contact detail field empty, when she attempts to send an invitation, then she sees "Enter an email address or phone number" and the invitation is not created.

**FEAT-09.SPEC-010-AC-02:** Given Maya enters text that is neither a valid email nor a valid phone number, when she blurs the field, then she sees "Enter a valid email address or phone number."

**FEAT-09.SPEC-010-AC-03:** Given Maya enters a contact detail matching Sam's sign_in email exactly, when she attempts to send the invitation, then it is blocked with "This person is already a member of your household."

**FEAT-09.SPEC-010-AC-04:** Given Maya enters a contact detail matching Sam's sign_in email but in different letter case, when she attempts to send the invitation, then it is still blocked, since the email comparison is case-insensitive.

**FEAT-09.SPEC-010-AC-05:** Given a Sent invitation already exists for "jane@example.com", when Maya attempts to send a second invitation to the same contact detail, then it is blocked with "There's already an outstanding invitation for this contact -- resend it instead of sending a new one."

**FEAT-09.SPEC-010-AC-06:** Given an invitation was sent exactly 14 days ago and remains Sent, when FEAT-09.SPEC-006 evaluates it, then it is eligible for expiry.

**FEAT-09.SPEC-010-AC-07:** Given an invitation was sent 13 days ago and remains Sent, when FEAT-09.SPEC-006 evaluates it, then it is not yet eligible for expiry.

**FEAT-09.SPEC-010-AC-08:** Given an invitation is revoked at the same moment it becomes eligible for expiry, and the revoke is recorded first, when both are processed, then the invitation's final status is Revoked, not Expired.

**FEAT-09.SPEC-010-AC-09:** Given an invitation is accepted at the same moment Maya revokes it, and the acceptance is recorded first, when both are processed, then the invitation's final status is Accepted and Maya's revoke is rejected with the current (Accepted) status shown.

**FEAT-09.SPEC-010-AC-10:** Given Maya selects herself as a hand-over recipient (defensive case -- she is excluded from the eligible list), when the recipient list is built, then Maya never appears as a selectable option.

**FEAT-09.SPEC-010-AC-11:** Given Maya selects a member whose status is Left, when the eligibility check runs, then the selection is rejected as ineligible.

**FEAT-09.SPEC-010-AC-12:** Given a household already has one outstanding hand-over request, when the organiser attempts to view FEAT-09.SPEC-003's recipient list, then no new request can be sent until the existing one resolves or is cancelled (XBR-15).

**FEAT-09.SPEC-010-AC-13:** Given an Expired invitation is resent, when the resend is processed, then a new Invitation record is created with a fresh sent_date, a recalculated expiry_date 14 days later, and status Sent, leaving the original Expired record unchanged.

**FEAT-09.SPEC-010-AC-14:** Given Maya enters a contact detail that is a phone number matching no Member Profile field, when she attempts to send the invitation, then the active-member re-invite check does not block it (no email field exists to compare), while the duplicate-outstanding-invitation check still applies.

**FEAT-09.SPEC-010-AC-15:** Given a hand-over request's sole eligible recipient completes their own departure (FEAT-09.SPEC-008) before the request is sent, when Maya's stale selection is submitted, then it is rejected with refresh and the recipient list reflects no eligible members.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 0 (N/A -- covered by FEAT-09.SPEC-011) | 0 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
