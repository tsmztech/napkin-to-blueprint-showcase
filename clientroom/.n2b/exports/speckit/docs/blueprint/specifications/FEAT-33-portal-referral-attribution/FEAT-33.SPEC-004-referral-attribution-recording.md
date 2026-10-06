---
document_type: spec
spec_type: automation
spec_id: FEAT-33.SPEC-004
spec_name: Referral Attribution Recording
spec_slug: referral-attribution-recording
parent_feature: FEAT-33
parent_feature_name: Portal Referral Attribution
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Referral Attribution Recording

## Overview

**Name:** Referral Attribution Recording
**ID:** FEAT-33.SPEC-004
**Type:** Automation
**Purpose:** Creates the Referral Attribution record once, at sign-up, from the referring portal captured by FEAT-33.SPEC-002 and the self-reported "how did you hear" answer captured by FEAT-20, degrading either half to unknown when it is missing.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution

## Scope and Non-Goals

**In Scope:**
- Creating exactly one Referral Attribution record per new Freelancer Account, at the moment FEAT-20's hand-off (FEAT-20.SPEC-004) delivers the captured values
- Resolving the referring-portal reference at the moment of creation, treating an unresolvable reference (e.g., the referring account was deleted in the meantime) as unknown
- Recording the self-reported source exactly as received (already normalized to "unknown" by FEAT-20.SPEC-004 when skipped or blank)
- Emitting the signals that feed success-metrics.md's "Growth Through Referral" metric
- Guaranteeing the record is never created twice for the same Freelancer Account

**Non-Goals:**
- Asking the "How did you hear" question -- owned by FEAT-20.SPEC-002; this automation only receives the already-captured answer via FEAT-20.SPEC-004's hand-off
- Capturing the referring-portal reference from the mark click -- owned by FEAT-33.SPEC-002; this automation only receives that value, already resolved at the moment of the click, via the same hand-off
- Exposing the created record to any screen or persona -- excluded per product-features.md, Access ("No persona browses referral data inside the product"); FEAT-33.SPEC-005 is the authority for that restriction, and this automation creates the record with no read path of its own
- Updating or correcting the record after creation -- excluded per the dependency map's lifecycle line ("Never updated"); a Referral Attribution record is immutable from the moment it is created

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| FEAT-20's referral attribution hand-off completes | FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Fires exactly once per new Freelancer Account, immediately after Nadia answers or skips the "How did you hear" question during onboarding | The new Freelancer Account reference, the self-reported source value (text or "unknown"), and the referring-portal reference (present or absent, originally captured by FEAT-33.SPEC-002) |

## Processing Logic

1. Receive the new Freelancer Account reference, the self-reported source value, and the referring-portal reference from FEAT-20.SPEC-004's hand-off.
2. Check whether a Referral Attribution record already exists for this Freelancer Account; if one does, take no further action (idempotency guard -- see Business Rules).
3. If a referring-portal reference is present, verify it still resolves to an existing Freelancer Account. If it does not resolve (for example, the referring account was deleted between the mark click and this sign-up completing), treat the reference as absent.
4. Create a Referral Attribution record for the new Freelancer Account with: `referring_portal` set to the resolved reference, or "unknown" if absent or unresolvable; `self_reported_source` set to the received value (already "unknown" if the question was skipped, per FEAT-20.SPEC-004); `recorded_at` set to the current time.
5. Emit `signup_attributed_to_portal` if a referring portal was recorded (not "unknown"), and emit `signup_source_answered` if a self-reported answer was recorded (not "unknown"). Always emit `referral_attribution_recorded` with both outcomes noted, regardless of whether either half is known, so the growth metric's full denominator of sign-ups is observable.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Recorded with both known | A referring-portal reference resolved and a self-reported answer was given | Referral Attribution record created with both fields populated | None -- invisible to Nadia; her onboarding already advanced in FEAT-20.SPEC-002 | FEAT-33.SPEC-005 (governs the new record's visibility) |
| Recorded with only self-reported known | No referring-portal reference resolved, but a self-reported answer was given | Record created with `referring_portal` set to "unknown" | None | FEAT-33.SPEC-005 |
| Recorded with only referring portal known | A referring-portal reference resolved, but the question was skipped (self-reported source is "unknown") | Record created with `self_reported_source` set to "unknown" | None | FEAT-33.SPEC-005 |
| Recorded fully unknown | Neither a referring-portal reference nor a self-reported answer is available | Record still created (`recorded_at` is always required) with both fields "unknown" | None | FEAT-33.SPEC-005 |
| Recording fails | This automation cannot complete (e.g., a transient failure creating the record) | No Referral Attribution record exists for this account | None -- the freelancer's sign-up and onboarding are already complete and fully unaffected; the growth metric simply undercounts this one sign-up | FEAT-20 (unaffected) |

## Data Model

**Reads:** Freelancer Account -- reads the new account's reference to attach the record to it, and reads the referring-portal reference (another Freelancer Account) only to confirm it still resolves to an existing account.
**Creates:** Referral Attribution -- `referring_portal` (reference to the referring Freelancer Account, or "unknown"), `self_reported_source` (free-text answer, or "unknown"), `recorded_at` (required).
**Updates:** None -- a Referral Attribution record is never updated after creation, per the dependency map's lifecycle line.
**Deletes:** None directly -- the record is later deleted only as part of FEAT-24's account-deletion cascade, outside this automation's scope.

## Business Rules

- This automation fires and creates a record at most once per Freelancer Account -- a second hand-off for the same account (should one somehow occur) is a no-op, since the idempotency check in Processing Logic finds an existing record and takes no further action.
- Both fields degrade independently to "unknown" -- the absence of one never affects the recording of the other (referring_portal and self_reported_source are recorded exactly as received, with no cross-field dependency).
- Recording is non-blocking, mirroring FEAT-20.SPEC-004's own rule: a failure here never reopens, delays, or re-surfaces anything to the freelancer's completed onboarding.
- XBR-32: attribution is used only in aggregate to measure the growth loop -- this automation creates the record but never exposes it to any screen; FEAT-33.SPEC-005 governs that restriction.
- The record's visibility, once created, is governed entirely by FEAT-33.SPEC-005 -- this automation does not re-derive or restate who may access it.

## Edge Cases

- **The referring Freelancer Account is deleted (FEAT-24) between the mark click and this sign-up completing** -- The reference fails to resolve at creation time and is recorded as "unknown," per Processing Logic step 3.
- **FEAT-20.SPEC-004's hand-off is somehow delivered twice for the same account (e.g., a retried delivery)** -- The idempotency check finds the existing record and takes no action; no second record is ever created.
- **The self-reported source contains only whitespace** -- Not possible at this automation's boundary: FEAT-20.SPEC-004 already normalizes a whitespace-only answer to "unknown" before handing it off, so this automation only ever receives a defined value.
- **Concurrent trigger firing (two different new freelancers complete sign-up at effectively the same time)** -- Each fires its own independent recording for its own Freelancer Account; there is no shared state between them, and no race exists since each account can have only its own record.
- **Trigger fires while a previous run is in flight for the same account** -- Cannot occur: FEAT-20.SPEC-004 fires this hand-off at most once per account (its own once-only rule), so no second run for the same account can start while the first is in flight.
- **The Referral Attribution record creation itself fails after Freelancer Account creation already succeeded** -- The Freelancer Account remains fully created and usable; only the attribution record is missing, silently undercounting that sign-up in the growth metric, consistent with the non-blocking business rule.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Triggered by (inbound) | Delivers the Freelancer Account reference, self-reported source, and referring-portal reference |
| FEAT-33.SPEC-002 (Referral Link Capture) | References (inbound) | Originates the referring-portal reference this automation resolves and records |
| FEAT-20 (Onboarding / First-Run Setup) | References (inbound) | Originates the self-reported "how did you hear" answer, asked in FEAT-20.SPEC-002 |
| FEAT-33.SPEC-005 (Referral Data Access Restriction) | Affects (outbound) | Governs who may access the record this automation creates -- no one, by product decision |
| FEAT-24 (Data Export & Account Deletion) | References (outbound) | The record created here is deleted with the owning Freelancer Account, with no independent retention window |

## Analytics and Success Signals

- **signup_attributed_to_portal** (referring portal known: yes) -- supports success-metrics.md: "Growth Through Referral"
- **signup_source_answered** (self-reported source known: yes) -- supports success-metrics.md: "Growth Through Referral"
- **referral_attribution_recorded** (referring portal known: yes/no; self-reported source known: yes/no) -- supports success-metrics.md: "Growth Through Referral" (this event always fires, giving the metric its full sign-up denominator alongside the two conditional numerator events above)

## Acceptance Criteria

**FEAT-33.SPEC-004-AC-01:** Given a new freelancer's account was referred by a resolvable portal and she also answered the "How did you hear" question, when this automation fires, then the Referral Attribution record is created with both `referring_portal` and `self_reported_source` populated, and both `signup_attributed_to_portal` and `signup_source_answered` are emitted.

**FEAT-33.SPEC-004-AC-02:** Given a new freelancer arrived with no referring-portal reference but answered the question, when this automation fires, then the record is created with `referring_portal` set to "unknown" and `self_reported_source` populated, and only `signup_source_answered` fires among the two conditional events.

**FEAT-33.SPEC-004-AC-03:** Given a new freelancer arrived via a resolvable referring portal but skipped the question, when this automation fires, then the record is created with `self_reported_source` set to "unknown" and `referring_portal` populated, and only `signup_attributed_to_portal` fires among the two conditional events.

**FEAT-33.SPEC-004-AC-04:** Given a new freelancer arrived with neither a referring portal nor an answer, when this automation fires, then the record is still created with both fields set to "unknown," and `referral_attribution_recorded` fires noting both as unknown, with neither conditional event firing.

**FEAT-33.SPEC-004-AC-05:** Given a referring-portal reference points to a Freelancer Account that was deleted before this sign-up completed, when this automation resolves the reference, then it is recorded as "unknown" rather than causing an error.

**FEAT-33.SPEC-004-AC-06:** Given a Referral Attribution record already exists for a Freelancer Account, when FEAT-20.SPEC-004's hand-off is somehow delivered again for that same account, then this automation takes no action and no second record is created.

**FEAT-33.SPEC-004-AC-07:** Given this automation's recording fails after the Freelancer Account was already created, when the failure occurs, then the Freelancer Account remains fully usable and no error is surfaced to the new freelancer.

**FEAT-33.SPEC-004-AC-08:** Given this automation successfully creates a Referral Attribution record, when the creation completes, then `referral_attribution_recorded` is emitted noting whether each of the two fields was known or unknown.

**FEAT-33.SPEC-004-AC-09:** Given two different new freelancers complete sign-up at effectively the same time, when this automation fires for each, then each creates its own independent record with no shared state or race between them.

**FEAT-33.SPEC-004-AC-10:** Given a Referral Attribution record has been created for a Freelancer Account, when any process attempts to update it later, then no update path exists -- the record remains exactly as recorded at creation.

**FEAT-33.SPEC-004-AC-11:** Given a Freelancer Account whose Referral Attribution record was created, when that account is later deleted (FEAT-24), then the Referral Attribution record is deleted with it, with no independent retention.

**FEAT-33.SPEC-004-AC-12:** Given this automation creates a record with an unknown referring portal, when FEAT-33.SPEC-005 is later consulted for that record's visibility, then no persona -- including Nadia and Dana -- has any path to view the individual record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
