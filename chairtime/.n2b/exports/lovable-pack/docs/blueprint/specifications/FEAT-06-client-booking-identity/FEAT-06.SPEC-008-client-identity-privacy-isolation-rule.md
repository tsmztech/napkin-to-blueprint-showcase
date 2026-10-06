---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-008
spec_name: Client Identity & Privacy Isolation Rule
spec_slug: client-identity-privacy-isolation-rule
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 14
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Client Identity & Privacy Isolation Rule

## Overview

**Name:** Client Identity & Privacy Isolation Rule
**ID:** FEAT-06.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs phone-to-Client matching scoped to one Pro, and the hard boundary that no client can ever see another phone number's or Pro's bookings.
**Parent Feature:** FEAT-06 -- Client Booking Identity
**Governed Entity:** Client (partial -- this feature reads phone and email, and updates email; it never creates or deletes Client records)

## Scope and Non-Goals

**In Scope:**
- The exact matching logic that resolves a phone number to a Client record, scoped to one Pro
- The privacy isolation boundary that prevents any cross-client or cross-Pro visibility
- Authorization over which role may perform phone-to-Client matching and what happens when matching fails or succeeds
- The "no hint" behavior when a phone number does not match

**Non-Goals:**
- Creating or deleting Client records -- owned by FEAT-05 (first booking) and FEAT-30 (Pro books a client in) for creation, and FEAT-13 for deletion; this spec governs only the read-time matching this feature performs
- Editing the client's name, phone number, or private notes -- phone is a read-only identity key to this feature, and name/notes remain Pro-editable only via FEAT-13
- Access Link expiry, single-use, and rate-limit rules -- owned by FEAT-06.SPEC-007; this spec governs identity matching, not link lifecycle
- Any cross-pro client profile or shared identity across Pros -- excluded per scope-boundaries SC-04: a phone number's Client record is scoped to exactly one Pro; booking with a second Pro always creates a separate, unconnected record

## Governed Entity

**Entity:** Client (partial)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Required, 1-100 characters; read-only to this feature |
| phone | text | Required, valid reachable format; identity key within one Pro; read-only to this feature -- this is the field this spec's matching logic operates on |
| email | text | Required when texts are declined, otherwise optional; the one field this feature writes (via FEAT-06.SPEC-005) |
| private_note | text | Pro-only, up to 1,000 characters; never read or written by this feature |
| booking_notes | text | The client's optional note per booking; never read or written by this feature |
| booking_history | derived | Derived list of this client's Bookings with this Pro only; read (not written) by FEAT-06.SPEC-003 and FEAT-06.SPEC-004, scoped by this spec's matching rule |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Access Link Request | Phone-to-Client matching at the moment a link is requested |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Scope resolution against the matched Client/Booking at the moment a link is redeemed |
| FEAT-06.SPEC-005 | Consent & Email Preferences | Matching used to identify which Client record's email and consent are being read and written |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| phone | Must exactly match an existing Client record's phone field, scoped to the one Pro in context | Always, whenever this feature performs a match | On link request (FEAT-06.SPEC-001) and on redemption (FEAT-06.SPEC-002) | No error is ever shown for "not found" -- see Business Rules for the no-hint pattern | Yes (a non-match blocks any further access) |
| name, private_note, booking_notes | No validation beyond data type -- these fields are never read or written by this feature | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Match is Pro-scoped | phone, Pro Account (context) | The same phone number may match different Client records under different Pros; a match is valid only within the one Pro the request or link is already scoped to | N/A -- a match under a different Pro is treated identically to no match at all |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Match a phone number to a Client record | The Client (Riley) | The entered or link-carried phone number exactly matches a Client record for the one Pro in context | No match: FEAT-06.SPEC-001 shows the plain "no bookings found" message with no hint about whether the number exists elsewhere; FEAT-06.SPEC-002 routes to the shared "request a new link" prompt |
| Match a phone number to a Client record | The Pro (Talia) | Never through this feature -- the Pro's own client lookup is FEAT-13 (Client Record Management), not this mechanism | Not applicable here -- this action is not exposed to the Pro through this feature at all |
| Match a phone number to a Client record | Platform Operator (Support) | Never | Support access never uses or bypasses client identity (scope-boundaries SC-05); no route to this action exists for Support |
| View another client's bookings or another Pro's data through this feature | No role, ever | Never -- this is a hard boundary, not a conditional one | There is no control, message, or path that exposes this; a client can never view another phone number's bookings even if they guess or mistype one (product-features.md, Validation & Limits) |
| Read the matched Client's email and consent state | The Client (Riley) | Only for the Client record their own phone number matched | Not applicable to any other role -- see the "view another client's bookings" row above |
| Update the matched Client's email | The Client (Riley) | Only for the Client record their own phone number matched (via FEAT-06.SPEC-005) | Not applicable to any other role through this feature |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Matched Client identity (session-scoped) | Derived once, at the moment a phone number is matched (FEAT-06.SPEC-001) or an Access Link is redeemed (FEAT-06.SPEC-002); held for the remainder of that viewing session | On match / on redemption | No -- the client cannot switch which Client identity a session resolves to mid-session; a different phone number requires a fresh request |

## Business Rules

- SC-03: cross-pro or cross-client visibility does not exist in this product; there is no shared or comparative view across accounts of any kind.
- SC-04: a phone number's Client record is scoped to exactly one Pro; a person booking with a second Pro has two unconnected records, never a merged or shared identity.
- The "no hint" pattern: a phone number with no matching Client record for this Pro produces the identical experience as a phone number that matches a different Pro entirely, or a mistyped number -- the client is never told which case applies (shared with FEAT-06.SPEC-002's identical redemption-failure pattern).
- XBR-18: access links open only that client's bookings with that Pro -- the matching performed here is what makes that boundary enforceable at every read this feature performs.
- Client deletion (owned by FEAT-13, XBR-19) removes the matched record entirely; a later booking under the same phone number creates a new, unconnected Client record rather than resurrecting the old one, so a match performed after such a deletion and rebooking always resolves to the new record only.

## Edge Cases

- **Client enters a phone number with different formatting than what is on file (e.g., extra spaces or a leading country code)** -- The match is performed on the normalized, reachable-format value already stored on the Client record; formatting differences that resolve to the same underlying number still match. A number that resolves to a genuinely different value does not match.
- **Two Client records under different Pros share the same phone number** -- Each match is scoped strictly to the one Pro already in context (from the booking page or link the client is using); the other Pro's record is never considered, checked, or referenced in any way, and its existence is never revealed.
- **Client's phone number was changed by the Pro (FEAT-13) after a link was issued but before it is redeemed** -- Per XBR-18, the phone change invalidates the client's existing access links; a redemption attempt against the old scope no longer resolves to a valid match and is treated as a scope resolution failure (FEAT-06.SPEC-002).
- **Client's Client record was deleted (FEAT-13, XBR-19) after a link was issued but before it is redeemed** -- The match can no longer resolve; redemption fails as a scope resolution failure, with the same no-hint experience as any other failed match.
- **A malicious actor tries many phone numbers in sequence to discover which ones exist for a Pro** -- Every non-match returns the identical "no bookings found" message with no timing, wording, or behavioral difference from a match that then finds zero bookings would show if such a state were possible; the per-phone-number rate limit (FEAT-06.SPEC-007) also bounds how many attempts a single number can drive in a given window.
- **Client rebooks with the same Pro under the same phone number after their prior Client record was deleted** -- FEAT-05 or FEAT-30 creates a new Client record (per FEAT-13's XBR-19); this spec's matching then resolves the phone number to that new record only, with no continuity to the deleted one's history.

## Acceptance Criteria

**FEAT-06.SPEC-008-AC-01:** Given Riley enters the phone number on file for her Client record with Talia, when FEAT-06.SPEC-001 matches it, then it resolves to her Client record for Talia and access proceeds.

**FEAT-06.SPEC-008-AC-02:** Given Riley enters a phone number with no matching Client record for Talia, when the match is attempted, then FEAT-06.SPEC-001 shows the plain "no bookings found" message with no hint about whether the number exists elsewhere.

**FEAT-06.SPEC-008-AC-03:** Given a phone number matches a Client record under a different Pro but not under Talia, when Riley enters it on Talia's request screen, then it is treated identically to no match at all.

**FEAT-06.SPEC-008-AC-04:** Given Riley's Client record with Talia was deleted and she later rebooks with the same phone number, when she next requests an access link, then it matches only her new Client record, with no visibility into the deleted record's history.

**FEAT-06.SPEC-008-AC-05:** Given Talia changed Riley's phone number in her client record (FEAT-13) after issuing Riley a link, when Riley taps that old link, then it fails to resolve and is treated as a scope resolution failure.

**FEAT-06.SPEC-008-AC-06:** Given Talia looks for a way to match a phone number to a client record through this feature's own screens, when she looks, then no such control exists -- her own client lookup is FEAT-13, not this mechanism.

**FEAT-06.SPEC-008-AC-07:** Given Platform Operator (Support) is assisting with a help request, when Support attempts to match a client's phone number through this feature, then no operational path exists for doing so.

**FEAT-06.SPEC-008-AC-08:** Given Riley's phone number has matched her Client record with Talia, when FEAT-06.SPEC-003 loads her bookings, then only bookings belonging to that matched Client record with Talia are shown -- never another client's or another Pro's bookings.

**FEAT-06.SPEC-008-AC-09:** Given a malicious actor submits many different phone numbers in sequence, when each is checked, then every non-match returns the identical message with no distinguishing detail, and the per-phone-number rate limit (FEAT-06.SPEC-007) bounds the attempts a single number can drive.

**FEAT-06.SPEC-008-AC-10:** Given Riley's phone number is entered with extra spaces compared to what is stored, when the match normalizes both values to the same reachable format, then it still resolves to her Client record.

**FEAT-06.SPEC-008-AC-11:** Given Riley's matched Client identity is established for a viewing session (FEAT-06.SPEC-001 or FEAT-06.SPEC-002), when she views FEAT-06.SPEC-005, then only her own email and consent state for Talia are shown and editable.

**FEAT-06.SPEC-008-AC-12:** Given two different phone numbers each have a Client record with Talia, when one client's phone number is entered, when a match is performed, then it resolves only to that one client's own record, never exposing the other's.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
