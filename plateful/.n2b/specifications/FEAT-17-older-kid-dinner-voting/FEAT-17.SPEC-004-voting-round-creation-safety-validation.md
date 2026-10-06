---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-004
spec_name: Voting Round Creation & Safety Validation
spec_slug: voting-round-creation-safety-validation
parent_feature: FEAT-17
parent_feature_name: Older-Kid Dinner Voting
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Voting Round Creation & Safety Validation

## Overview

**Name:** Voting Round Creation & Safety Validation
**ID:** FEAT-17.SPEC-004
**Type:** Automation
**Purpose:** When Maya opens a voting round, validates the chosen options against the 2-3 count and FEAT-02's safety check, creates the round, and fires dinner_vote_opened.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting

## Scope and Non-Goals

**In Scope:**
- Validating the option count (2-3) for a round Maya is opening
- Re-confirming each selected option's current safety-check status against the household's Dietary Rules before the round can exist
- Preventing more than one open round per night
- Creating the Dinner Vote round record and firing dinner_vote_opened

**Non-Goals:**
- Computing the safety determination itself -- owned by FEAT-02 (Dietary Rules & Allergy Safety Engine); this automation only re-confirms FEAT-02's existing determination at round-creation time, per XBR-01's authority assignment.
- Generating the candidate options -- owned by FEAT-03 (AI Weekly Dinner Plan Generation); this automation validates only the options Maya has already selected from FEAT-03's proposed alternatives.
- Recomputing the tally once votes are cast, or resolving the round -- handled by FEAT-17.SPEC-005 (Vote Round Resolution & Fallback).

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Maya taps "Open Vote" | FEAT-17.SPEC-002 (Voting Round Setup) | Maya has selected a night and 2-3 candidate options and submits the action | Selected night, the list of selected options (2-3 recipe references), each option's most recently computed safety-check status, Maya's Member Profile reference |

## Processing Logic

1. Receive the selected night and the selected option list (2-3 recipe references) from the triggering screen.
2. Check whether an open or resolved Dinner Vote round already exists for that night; if so, stop and return the "round already exists" outcome.
3. Check that the selected option count is exactly 2 or 3; if not, stop and return the "invalid option count" outcome.
4. Re-confirm each selected option's current safety-check status against the household's Dietary Rules via FEAT-02; a mid-week rule change could have altered a status since FEAT-03 last proposed the option.
5. If every selected option currently passes the safety check, create the Dinner Vote round: night, the validated 2-3 options, status Open.
6. Fire dinner_vote_opened.
7. If one or more selected options no longer pass the safety check, stop without creating the round and return the "option fails safety check" outcome, naming each failing option.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Round opened | All checks pass (round-uniqueness, option count, safety re-check) | Dinner Vote round created with status Open | Confirmation "Vote opened for {night}" | FEAT-17.SPEC-002, FEAT-17.SPEC-001, FEAT-17.SPEC-003 |
| Rejected -- round already exists | A round (open or resolved) already exists for this night | None | "A vote is already open for this night." (open) or "This night's vote is already decided." (resolved), each with a link to FEAT-17.SPEC-003 | FEAT-17.SPEC-002 |
| Rejected -- invalid option count | Fewer than 2 or more than 3 options selected | None | "Choose 2 to 3 options." | FEAT-17.SPEC-002 |
| Rejected -- option fails safety check | One or more selected options no longer pass FEAT-02's check | None | "{Option name} no longer passes the allergy check and can't be offered." naming each failing option | FEAT-17.SPEC-002 |
| Automation failure | Processing error (e.g., the safety-check re-confirmation cannot complete) | None | "Could not open the vote. Try again." -- blocking, since a round is never created on an unconfirmed safety check (XBR-01's fail-closed rule) | FEAT-17.SPEC-002 |

## Data Model

**Reads:** Dietary Rule (via FEAT-02) -- each candidate option's current pass/fail safety status for the household; existing Dinner Vote rounds for the night (to detect an already-open or already-resolved round).
**Creates:** Dinner Vote -- round (night, the validated 2-3 options), status Open.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-01: every path onto the plan -- including voting options -- passes the same app-enforced allergy check before anyone sees it; the check fails closed, so a round is never created offering an option that does not currently pass.
- At most one round (open or resolved) exists per night (Non-Functional Notes: data volumes).
- A round must offer exactly 2 or 3 options (FEAT-17.SPEC-006).
- This automation runs synchronously -- FEAT-17.SPEC-002 waits for its result before confirming the round is open.

## Edge Cases

- **One selected option fails the re-check but the other 1-2 still pass, leaving fewer than 2 valid options** -- The round is rejected in full, not partially opened; Maya must re-select from the remaining valid candidates in FEAT-17.SPEC-002.
- **A household's dietary rule changes at the exact moment this automation runs** -- This automation's own safety re-check at round-creation time is authoritative; an option affected by the change is treated as failing and the round is rejected, consistent with the fail-closed rule.
- **Concurrent trigger firing (Maya opens rounds for two different nights at effectively the same time)** -- Each run evaluates its own night independently with no shared state, so both can succeed.
- **Trigger fires while a previous run for the same night is in flight (a double-tap on Open Vote, or two devices submitting for the same night close together)** -- The Open Vote action is disabled during submission on the triggering screen (FEAT-17.SPEC-002), and this automation's own round-uniqueness check (step 2) treats the check-then-create sequence for a given night as atomic, so a second run for the same night is rejected with "A vote is already open for this night." even if it started before the first run's round became visible to it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-002 (Voting Round Setup) | Triggered by (inbound); Affects (outbound) | Fires on Open Vote tap; returns success or a specific rejection reason to the screen |
| FEAT-17.SPEC-001 (Dinner Vote Casting) | Affects (outbound) | A successfully opened round becomes votable on this screen |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | References (inbound) | Option-count and safety-check rules this automation enforces |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (inbound) | Re-confirms each candidate option's safety status |
| FEAT-03 (AI Weekly Dinner Plan Generation) | References (inbound) | Candidate options originate from FEAT-03's proposed alternatives for the night |

## Analytics and Success Signals

- **dinner_vote_opened** (night, option count, options offered by recipe reference) -- N/A -- no metric in success-metrics.md carries Older-Kid Dinner Voting (FEAT-17) as its Connected Feature; this Later-phase feature has no metric of its own in the current success-metrics set.
- **dinner_vote_open_rejected** (reason: round_already_exists / invalid_option_count / safety_check_failed / processing_error) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-17.SPEC-004-AC-01:** Given Maya has selected 2 safety-checked options for a night with no existing round, when she submits Open Vote, then a Dinner Vote round is created with status Open, dinner_vote_opened fires, and she sees "Vote opened for {night}."

**FEAT-17.SPEC-004-AC-02:** Given Maya has selected 3 safety-checked options for a night with no existing round, when she submits Open Vote, then a Dinner Vote round is created offering all 3 options.

**FEAT-17.SPEC-004-AC-03:** Given a night already has an open round, when Maya attempts to open another round for the same night, then no new round is created and she sees "A vote is already open for this night."

**FEAT-17.SPEC-004-AC-04:** Given a night already has a resolved round, when Maya attempts to open a new round for the same night, then no new round is created and she sees "This night's vote is already decided."

**FEAT-17.SPEC-004-AC-05:** Given Maya has selected only 1 option, when she submits Open Vote, then no round is created and she sees "Choose 2 to 3 options."

**FEAT-17.SPEC-004-AC-06:** Given Maya has selected 2 options and one of them no longer passes FEAT-02's safety check at the moment of submission, when this automation runs, then no round is created and Maya sees "{Option name} no longer passes the allergy check and can't be offered." naming that option.

**FEAT-17.SPEC-004-AC-07:** Given the safety-check re-confirmation cannot complete due to a processing error, when Maya submits Open Vote, then no round is created and she sees "Could not open the vote. Try again."

**FEAT-17.SPEC-004-AC-08:** Given Maya double-taps Open Vote for the same night in quick succession, when both submissions reach this automation, then only the first creates a round and the second is rejected with "A vote is already open for this night."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
