---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-008
spec_name: Phone Number Change Consent Invalidation Rule
spec_slug: phone-number-change-consent-invalidation-rule
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Phone Number Change Consent Invalidation Rule

## Overview

**Name:** Phone Number Change Consent Invalidation Rule
**ID:** FEAT-14.SPEC-008
**Type:** Logic/Rule
**Purpose:** Invalidates a client's existing texting consent whenever their phone number changes, so fresh consent is required before the new number is ever texted.
**Parent Feature:** FEAT-14 -- Messaging Consent Management
**Governed Entity:** Messaging Consent

## Scope and Non-Goals

**In Scope:**
- Detecting a Client phone-number change and invalidating the affected Messaging Consent record
- Defining what "invalidated" means for the record's fields and for downstream textability
- The relationship between this rule and the fresh-consent capture that follows

**Non-Goals:**
- Changing the Client's phone number itself -- owned by FEAT-13.SPEC-002 (Client Contact Edit); this spec only reacts to a change that has already been saved there.
- Invalidating Access Links on a phone-number change -- owned by FEAT-06 (Client Booking Identity), per the dependency map's Client Contention note; this spec covers Messaging Consent only, a distinct entity with a distinct rule.
- Capturing the fresh consent required after invalidation -- owned by FEAT-14.SPEC-003, which this rule's invalidated record simply makes eligible for a first-time-style capture again.

## Governed Entity

**Entity:** Messaging Consent
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| channel | enum | The consent's channel (text; WhatsApp reserved for a later phase) |
| state | enum | Granted \| Revoked \| Re-granted |
| timestamp | date | When the current state was set |
| consent_wording | text | The exact wording shown to the client when consent was given, kept as evidence |
| phone_number | text | The phone number the consent applies to |

**Referenced (read-only):** Client -- phone, the field whose change triggers this rule.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-002 | Client Contact Edit | Saving a changed phone number is the event this rule reacts to |
| FEAT-14.SPEC-003 | Consent Capture at Booking | Reads the invalidated state as its starting point when the client's next booking captures fresh consent |
| FEAT-14.SPEC-007 | Textability Determination Rule | Reflects the invalidation immediately, since its phone-number-match condition already fails once the Client's phone no longer matches the consent record |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| channel | No validation beyond data type -- unaffected by invalidation | Always | -- | -- | -- |
| state | Not directly changed by this rule -- invalidation is expressed through the phone_number mismatch (see Cross-Field Rules), not by forcing state to Revoked | Always | -- | -- | -- |
| timestamp | Not updated by this rule -- the original grant/revoke timestamp remains historically accurate; invalidation is a separate, derived condition layered on top | Always | -- | -- | -- |
| consent_wording | No validation beyond data type -- untouched by a phone-number change; it remains evidence of what was shown for the old number | Always | -- | -- | -- |
| phone_number | Left unchanged on the Messaging Consent record itself when the Client's phone changes -- the record's phone_number continues to reflect the number the original consent applied to, which is now stale relative to Client.phone | On every Client phone-number change | On Client phone-number save (FEAT-13.SPEC-002) | N/A -- no user-facing error; the mismatch this creates is exactly the mechanism that expresses invalidation | No (this is the intended effect, not a rejected value) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Invalidation-by-mismatch | Messaging Consent.phone_number, Client.phone | The moment Client.phone changes and no longer equals the Messaging Consent record's phone_number, the record is treated as not applicable to the client's current number -- FEAT-14.SPEC-007's textability determination resolves false for it, exactly as it would for a Revoked record, without this rule needing to write a new state value | N/A -- the mismatch condition itself is the invalidation; no separate write is required |
| Fresh-consent eligibility | Messaging Consent.phone_number, Client.phone | Once invalidated by mismatch, the relationship becomes eligible for FEAT-14.SPEC-003 to capture fresh consent at the client's next booking, which updates phone_number to the current number alongside the new state | N/A -- this is the designed recovery path, not an error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change a client's phone number (the event this rule reacts to) | The Pro (Talia) | Full, on her own clients only (Access Matrix: Client Records = Full for the Pro), via FEAT-13.SPEC-002 | -- |
| Change a client's phone number | The Client (Riley) | Never -- phone is the client's identity key and is not directly editable by the client themselves (product-features.md, Client entity); only the Pro corrects it | Phone number field is not offered as editable anywhere in the client-facing product |
| Trigger or waive this invalidation rule directly | Any role | Never -- invalidation is automatic and unconditional whenever a phone-number change is saved; no role can opt a client out of needing fresh consent after a number change | No control exists for any role to bypass the fresh-consent requirement after a phone-number change, since it is a US SMS-consent requirement (ASMP-24), not a product preference |
| Read whether a relationship's consent is currently invalidated by a phone-number mismatch | The Pro (Talia) | View-only, via FEAT-14.SPEC-007's textability output on FEAT-12 | -- |
| Read whether a relationship's consent is currently invalidated by a phone-number mismatch | The Client (Riley) | Own-only, via FEAT-14.SPEC-001, where an invalidated relationship simply shows "Texting: off" like any other revoked state | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Effective textability of an invalidated relationship | Derived to false via FEAT-14.SPEC-007's phone_number-match condition, without any direct write to this rule's governed record | Continuously, from the moment Client.phone changes until fresh consent is captured | No |

## Business Rules

- XBR-15: a changed phone number requires fresh consent -- this rule is that requirement's entire mechanism, expressed as a mismatch condition rather than a forced state change, so the historical record of the original consent (its timestamp and wording) is preserved unaltered as evidence of what was once agreed for the old number.
- Invalidation is silent to both the Pro and the Client at the moment it happens -- no notification fires from this rule itself; the Client simply sees "Texting: off" on their next view of FEAT-14.SPEC-001, and the Pro sees the client as not currently textable on FEAT-12, both through the ordinary textability read path.
- This rule never deletes or overwrites the prior consent evidence -- consent_wording, the original timestamp, and the old phone_number all remain on the record exactly as captured, satisfying the retention expectation in scope-boundaries.md SC-22 alongside the fresh-consent requirement.
- The client re-gains texting only through the ordinary capture paths that already exist -- a subsequent booking (FEAT-14.SPEC-003) or, once the Pro's phone-number edit has propagated, an in-app re-grant (FEAT-14.SPEC-005) -- this rule creates no new consent-capture surface of its own.

## Edge Cases

- **The Pro corrects a typo in the phone number that does not actually represent a different real-world number (e.g., fixing a transposed digit for the same client)** -- The rule cannot distinguish a typo fix from a genuine number change; it applies invalidation uniformly to any saved change in the Client.phone value, per XBR-15's plain requirement, even though the practical risk is the same in both cases -- fresh consent is required either way.
- **The client re-grants texting via FEAT-14.SPEC-005 for a relationship whose consent is currently invalidated by a phone-number mismatch** -- The write succeeds and sets state to Re-granted, but phone_number remains at its old, stale value (per FEAT-14.SPEC-005's own scope), so the mismatch persists and FEAT-14.SPEC-007 still resolves Textable = false until a booking (FEAT-14.SPEC-003) updates phone_number to the current number.
- **The Pro changes the phone number and then changes it back to the original number within the same session** -- Each save is evaluated independently; after the second save, Client.phone once again equals the Messaging Consent record's phone_number, so the mismatch condition no longer holds and textability reflects the record's state field again as if no interruption occurred -- there is no "invalidation history" that persists once the numbers realign.
- **A phone-number change happens while a message is already queued to send to the old number** -- FEAT-14.SPEC-007's fresh-at-read-time evaluation (not cached from queue time) means the send, when it actually goes out, correctly finds the mismatch and routes to email, consistent with FEAT-08.SPEC-011's send-time channel decision.
- **The client has never had an existing Messaging Consent record at all when their phone number changes (a client who has never texted-opted-in at any booking)** -- There is nothing to invalidate; this rule has no effect, and the client's status continues to reflect whatever their prior state already was (Revoked from their original booking-time choice).

## Acceptance Criteria

**FEAT-14.SPEC-008-AC-01:** Given Talia corrects Riley's phone number on FEAT-13.SPEC-002, when the change saves, then Riley's existing Messaging Consent record's phone_number no longer matches her Client.phone.

**FEAT-14.SPEC-008-AC-02:** Given Riley's phone number was just changed, when FEAT-14.SPEC-007 evaluates her textability, then it returns Textable = false, even if her consent state field still reads Granted.

**FEAT-14.SPEC-008-AC-03:** Given Riley's phone number changed and her consent is now invalidated, when she books again with Talia and opts in, then FEAT-14.SPEC-003 captures fresh consent, updating phone_number to the current number.

**FEAT-14.SPEC-008-AC-04:** Given Riley's phone number changed, when the invalidation occurs, then her original consent_wording and timestamp remain unchanged on the record.

**FEAT-14.SPEC-008-AC-05:** Given Riley (the Client) looks for a way to change her own phone number in the product, when she reviews her available settings, then no such control exists -- only Talia can change it via FEAT-13.SPEC-002.

**FEAT-14.SPEC-008-AC-06:** Given Talia fixes a typo in Riley's phone number that represents the same real number, when the change saves, then the rule still invalidates Riley's consent, since the rule cannot distinguish a typo fix from a genuine change.

**FEAT-14.SPEC-008-AC-07:** Given Riley's consent is invalidated by a phone-number mismatch, when she taps "Turn texting back on" on FEAT-14.SPEC-001, then the write sets state to Re-granted but the mismatch persists until her next booking updates phone_number.

**FEAT-14.SPEC-008-AC-08:** Given Talia changes Riley's phone number and then reverts it to the original value in the same session, when the second save completes, then the mismatch no longer exists and Riley's textability reflects her state field as before.

**FEAT-14.SPEC-008-AC-09:** Given Riley has never had a Messaging Consent record at all, when her phone number changes, then this rule has no effect, since there is no record to invalidate.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
