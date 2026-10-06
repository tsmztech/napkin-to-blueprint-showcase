---
document_type: spec
spec_type: automation
spec_id: FEAT-20.SPEC-004
spec_name: Referral Attribution Capture Hand-off
spec_slug: referral-attribution-capture-hand-off
parent_feature: FEAT-20
parent_feature_name: Onboarding / First-Run Setup
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Referral Attribution Capture Hand-off

## Overview

**Name:** Referral Attribution Capture Hand-off
**ID:** FEAT-20.SPEC-004
**Type:** Automation
**Purpose:** Captures Nadia's optional "How did you hear about us?" answer and the referring-portal reference, then hands both to Portal Referral Attribution (FEAT-33) for recording.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- Capturing the self-reported answer (or "unknown" when skipped) at the moment Nadia answers or skips the question in FEAT-20.SPEC-002
- Reading the referring-portal reference that FEAT-20.SPEC-001 persisted on the Freelancer Account (`onboarding_referring_portal_ref`) at sign-up, or recording it as absent when the visitor arrived without one
- Handing both values off to FEAT-33 exactly once per Freelancer Account
- Tolerating the hand-off's own failure without blocking or reversing onboarding progress

**Non-Goals:**
- Creating or owning the Referral Attribution record itself -- per the dependency map, "Created by FEAT-33 at sign-up (answer captured in FEAT-20)"; this automation only captures and hands off the two values, FEAT-33 persists them
- Displaying the "How did you hear" question -- owned by FEAT-20.SPEC-002; this automation begins where that screen's answer/skip interaction ends
- Measuring or reporting the referral growth loop -- owned by FEAT-33's own aggregate measurement (success-metrics.md, "Growth Through Referral"); this automation only supplies the raw values
- Re-asking or re-capturing the answer after it is resolved -- the question is resolved exactly once, per FEAT-20.SPEC-002's Business Rules and FEAT-20.SPEC-005 R-06; this automation never fires a second time for the same account

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia answers the "How did you hear" question | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires once, when Nadia taps Continue after entering an answer | Freelancer Account reference, the entered answer text; the referring-portal reference (if any) is read from the account's onboarding_referring_portal_ref |
| Nadia skips the "How did you hear" question | FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Fires once, when Nadia taps Skip without entering an answer | Freelancer Account reference, no answer text (treated as "unknown"); the referring-portal reference (if any) is read from the account's onboarding_referring_portal_ref |
| Onboarding completes with the question unresolved | FEAT-20.SPEC-003 (Onboarding Completion Detection) | Fires once, when completion is recorded while onboarding_how_did_you_hear_resolved is false (FEAT-20.SPEC-005 R-09) | Freelancer Account reference, no answer text (treated as "unknown"); the referring-portal reference (if any) read from the account |

## Processing Logic

1. Receive the Freelancer Account reference and the answer (text or absent) from the trigger, then read onboarding_referring_portal_ref (present or absent) from the account; if onboarding_how_did_you_hear_resolved is already true, stop -- the hand-off has already fired.
2. If no answer was entered (the Skip path), set the self-reported source value to "unknown."
3. If a referring-portal reference is absent (the visitor arrived without following a FEAT-33 mark), record it as absent rather than substituting any inferred value.
4. Hand off the self-reported source value and the referring-portal reference (or its absence) to Portal Referral Attribution (FEAT-33) for it to create the Referral Attribution record, identifying the hand-off by the Freelancer Account reference so FEAT-33 creates at most one Referral Attribution record per account.
5. Whether the hand-off in step 4 succeeded or failed, set onboarding_how_did_you_hear_resolved to true on the account with a compare-and-set (change false to true only if it is still false). This automation is the only writer of onboarding_how_did_you_hear_resolved after account creation: FEAT-20.SPEC-002 and FEAT-20.SPEC-003 only trigger this automation and never write the field. If the compare-and-set finds the field already true (another run resolved it first), do nothing further.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Hand-off succeeds with an answer | Nadia entered an answer and the hand-off to FEAT-33 completes | FEAT-33 creates its Referral Attribution record with the self-reported source and referring-portal reference; this automation then sets onboarding_how_did_you_hear_resolved to true | None -- Nadia already saw her onboarding step advance in FEAT-20.SPEC-002; this hand-off is invisible to her | FEAT-33 (Referral Attribution record) |
| Hand-off succeeds with "unknown" | Nadia skipped the question and the hand-off to FEAT-33 completes | FEAT-33 creates its Referral Attribution record with self-reported source "unknown"; this automation then sets onboarding_how_did_you_hear_resolved to true | None | FEAT-33 (Referral Attribution record) |
| Hand-off fails | The hand-off to FEAT-33 cannot complete (e.g., a transient failure) | No Referral Attribution record is created at this time; onboarding_how_did_you_hear_resolved is still set to true (compare-and-set), so the question is resolved and never re-asked and the hand-off is not retried | None -- onboarding progression in FEAT-20.SPEC-002 is never blocked or delayed by this failure | FEAT-20.SPEC-002 (unaffected) |

## Data Model

**Reads:** Freelancer Account -- onboarding_referring_portal_ref (persisted at sign-up by FEAT-20.SPEC-001) and onboarding_how_did_you_hear_resolved, plus the answer text supplied by the trigger.
**Creates:** None directly -- the Referral Attribution record is created by FEAT-33, not by this automation.
**Updates:** Freelancer Account -- onboarding_how_did_you_hear_resolved set true by compare-and-set after the hand-off attempt (this automation is its only writer after account creation, whether the hand-off succeeded or failed); no other field.
**Deletes:** None.

## Business Rules

- This automation fires at most once per Freelancer Account, in lockstep with the question being resolved exactly once (answered, skipped, or auto-resolved at onboarding completion). Because the question is resolved only on answer or skip -- not on render -- a Nadia who saw it and left is asked again on return and the hand-off still fires when she finally resolves it.
- This automation is the single writer of onboarding_how_did_you_hear_resolved (after its default of false at account creation). It performs the hand-off first and then sets the field true by compare-and-set; the value after a failed hand-off is also true, because the question has been answered, skipped, or auto-resolved and is never re-asked. FEAT-20.SPEC-002 and FEAT-20.SPEC-003 only trigger this automation.
- A skipped question is captured as "unknown," never as an empty or missing value -- FEAT-33's Referral Attribution record always receives a defined self_reported_source value.
- The referring-portal reference, when present, is passed through unchanged from the value persisted on the account at sign-up (FEAT-20.SPEC-001) -- this automation never re-derives or re-validates it.
- This hand-off is non-blocking: onboarding's own progression (advancing to the next guided step) never waits on or is reversed by the hand-off's outcome (XBR-32: the referral mark's attribution is used only in aggregate, never a gate on the freelancer's own experience).

## Edge Cases

- **The hand-off to FEAT-33 fails** -- No retry is attempted from this automation; onboarding continues normally, and the self-reported source is simply never recorded for this account, and onboarding_how_did_you_hear_resolved is still set to true (accepted per this feature's non-blocking business rule -- a failed attribution never re-surfaces to Nadia, since she is never asked twice per FEAT-20.SPEC-002).
- **Nadia enters free text that is empty after trimming whitespace** -- Treated identically to an explicit Skip: the self-reported source is recorded as "unknown."
- **A referring-portal reference is present but that referring Freelancer Account is later deleted (FEAT-24)** -- The hand-off already completed at sign-up time with the reference as it stood then; FEAT-33 owns how a later deletion affects an already-recorded attribution, which is outside this automation's scope.
- **Nadia sees the question, closes the browser without answering, and returns** -- Nothing has fired and onboarding_how_did_you_hear_resolved is false; the persisted reference is intact; the hand-off fires when she answers or skips on her return.
- **Onboarding completes before Nadia ever answers or skips** -- The third trigger fires once with "unknown" and the persisted reference, so the reference is not lost; the question is not shown afterward.
- **Concurrent trigger firing (e.g., Nadia answers in one tab while a second tab skips, or completion lands at the same moment)** -- Each run first checks onboarding_how_did_you_hear_resolved and stops if it is already true. If two runs both pass that check, both hand off, but FEAT-33 treats the Freelancer Account reference as the identity of the hand-off and creates only one Referral Attribution record; only the run whose compare-and-set changes the field from false to true completes, the other does nothing further. The hand-off therefore results in exactly one record.
- **Trigger fires while a previous run is in flight** -- A second run for the same account re-reads onboarding_how_did_you_hear_resolved before handing off; once the earlier run has set it, the second run stops without a duplicate hand-off, and if it read the field before the earlier run set it, the same account-identified hand-off and compare-and-set rules above prevent a second record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Triggered by (inbound) | Answering or skipping the "How did you hear" question fires this hand-off; SPEC-002 only triggers it and never writes onboarding_how_did_you_hear_resolved |
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | References (inbound) | Persists the referring-portal reference carried forward from a FEAT-33 entry on the account at creation, where this automation reads it |
| FEAT-20.SPEC-003 (Onboarding Completion Detection) | Triggered by (inbound) | Completion with the question unresolved fires the hand-off once with "unknown"; SPEC-003 only triggers it and never writes onboarding_how_did_you_hear_resolved |
| FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules) | References (inbound) | Defines onboarding_how_did_you_hear_resolved and onboarding_referring_portal_ref, R-06 and R-09 |
| FEAT-33 (Portal Referral Attribution) | Affects (outbound) | Hands off the self-reported source and referring-portal reference for Referral Attribution creation |

## Analytics and Success Signals

- **how_did_you_hear_answered** (answered vs. skipped) -- supports success-metrics.md: "Growth Through Referral" (this event is also emitted by FEAT-20.SPEC-002 at the interaction level; this automation's own emission confirms the value actually reached the hand-off, distinguishing an answered-but-failed-hand-off case from a fully recorded one)
- **referral_attribution_handoff_failed** (had a referring-portal reference: yes/no) -- N/A -- no Stage 2 metric measures hand-off failures directly; retained so a silently dropped attribution is observable rather than invisible

## Acceptance Criteria

**FEAT-20.SPEC-004-AC-01:** Given Nadia types "A friend recommended it" and taps Continue on the "How did you hear" question, when this automation fires, then it hands off that answer to FEAT-33 for the Referral Attribution record.

**FEAT-20.SPEC-004-AC-02:** Given Nadia taps Skip without entering an answer, when this automation fires, then it hands off "unknown" as the self-reported source to FEAT-33.

**FEAT-20.SPEC-004-AC-03:** Given Nadia arrived by following a "Made with Clientroom" mark, when this automation fires (even in a later session than sign-up), then the referring-portal reference read from her account's onboarding_referring_portal_ref is included unchanged in the hand-off to FEAT-33.

**FEAT-20.SPEC-004-AC-04:** Given Nadia arrived without following any referral mark, when this automation fires, then the hand-off records the referring-portal reference as absent rather than substituting any inferred value.

**FEAT-20.SPEC-004-AC-05:** Given the hand-off to FEAT-33 fails, when the failure occurs, then Nadia's onboarding progression in FEAT-20.SPEC-002 continues unaffected and she is never re-asked the question.

**FEAT-20.SPEC-004-AC-06:** Given Nadia types only whitespace into the answer field before tapping Continue, when this automation fires, then the self-reported source is recorded as "unknown," identical to an explicit Skip.

**FEAT-20.SPEC-004-AC-07:** Given onboarding has already fired this hand-off once for Nadia's account, when any later action occurs, then this automation never fires a second time for that account.

**FEAT-20.SPEC-004-AC-08:** Given the hand-off succeeds, when it completes, then Nadia sees no confirmation of her own -- the hand-off is invisible, since her onboarding step already advanced in FEAT-20.SPEC-002.

**FEAT-20.SPEC-004-AC-09:** Given the hand-off fails for an account with a referring-portal reference present, when the failure occurs, then `referral_attribution_handoff_failed` is emitted noting a referring-portal reference was present, so the dropped attribution is observable.

**FEAT-20.SPEC-004-AC-10:** Given Nadia saw the "How did you hear" question, closed the browser without answering, and signed in again the next day, when she then answers, then this automation fires once and includes the referring-portal reference persisted at sign-up.

**FEAT-20.SPEC-004-AC-11:** Given onboarding completes while the question was never answered or skipped, when completion is recorded, then this automation fires once with "unknown" and the persisted reference.

**FEAT-20.SPEC-004-AC-12:** Given Nadia answers in one tab and skips in another at effectively the same moment, when both run, then FEAT-33 ends with exactly one Referral Attribution record for her account, only one run's compare-and-set changes onboarding_how_did_you_hear_resolved from false to true, and the other run does nothing further.

**FEAT-20.SPEC-004-AC-13:** Given Nadia answers or skips the question, when FEAT-20.SPEC-002 triggers this automation, then onboarding_how_did_you_hear_resolved is still false when the trigger arrives and becomes true only after this automation's hand-off attempt finishes, written by this automation and not by FEAT-20.SPEC-002.

**FEAT-20.SPEC-004-AC-14:** Given the hand-off to FEAT-33 fails, when this automation finishes, then onboarding_how_did_you_hear_resolved is true, the question is never shown again, and no retry of the hand-off occurs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
