---
document_type: spec
spec_type: automation
spec_id: FEAT-06.SPEC-004
spec_name: Linked Asset Reachability Check
spec_slug: linked-asset-reachability-check
parent_feature: FEAT-06
parent_feature_name: Deliverable Upload & Sharing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Linked Asset Reachability Check

## Overview

**Name:** Linked Asset Reachability Check
**ID:** FEAT-06.SPEC-004
**Type:** Automation
**Purpose:** Validates that a pasted external link resolves before the deliverable is marked ready, flagging an unreachable link to the freelancer.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- Checking that a pasted Figma, Google Drive, or Dropbox link resolves and is reachable
- Setting the Deliverable's link_status to reachable or flagged based on the outcome
- Flagging an unreachable link to Nadia before the deliverable is ever shown to the client as broken

**Non-Goals:**
- Validating the link's format (well-formed URL, recognized domain) -- owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules), which runs before this automation is invoked
- Copying, mirroring, or previewing the linked asset's content -- excluded per scope-boundaries.md (SC-08): linked assets are referenced by URL only, never copied into the product
- Re-checking a link's reachability on an ongoing basis after the deliverable is marked ready -- this automation runs once per submitted link at attach time; no periodic re-verification exists in the product definition
- Processing uploaded files -- handled entirely by FEAT-06.SPEC-003 (Resumable Upload Handling); this automation processes pasted links only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Link submitted for attach | FEAT-06.SPEC-001 (Deliverable Upload) | Fires when Nadia taps Attach Deliverable with "Paste a link" selected and the link has already passed FEAT-06.SPEC-005's format check | The submitted URL, the milestone reference |
| Link re-submitted after being flagged | FEAT-06.SPEC-001 (Deliverable Upload) | Fires when Nadia edits a flagged link and re-submits | The revised URL, the milestone reference, the prior flagged attempt |

## Processing Logic

1. Receive the format-valid URL and milestone reference from FEAT-06.SPEC-001.
2. Create the Deliverable record if one does not already exist for this submission: kind = linked external asset, milestone = the carried reference, uploaded_at = current time, link_status = pending.
3. Attempt to resolve the link: request the target address and evaluate whether it responds successfully and is accessible (not requiring credentials the product does not hold, not returning a not-found or access-denied response).
4. If the link resolves successfully, set link_status to reachable, the Deliverable's status to Active, and the owning Milestone's status to "Deliverable Uploaded" (dependency map: Milestone updated by FEAT-06), mirroring FEAT-06.SPEC-003's completion step -- a link-delivered deliverable reaches this milestone state exactly as a file-delivered one does.
5. If the link does not resolve, set link_status to flagged and leave the Deliverable's status at its pending state -- the deliverable is never shown to the client while flagged.
6. Signal FEAT-06.SPEC-006 (Deliverable Ready Notification) only when link_status becomes reachable -- never while flagged (XBR-12's "never for a partial file" applies equally to an unresolved link, since it is not yet a usable deliverable).
7. Return the outcome (reachable or flagged) to FEAT-06.SPEC-001 for immediate display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Link reachable | The target resolves successfully and is accessible | Deliverable.link_status = reachable; Deliverable.status = Active; Milestone.status = "Deliverable Uploaded" | FEAT-06.SPEC-001 shows success and returns Nadia to FEAT-06.SPEC-002; FEAT-06.SPEC-002's card shows reachable | FEAT-06.SPEC-001, FEAT-06.SPEC-002, FEAT-06.SPEC-006 (notification fires) |
| Link flagged | The target fails to resolve, or resolution reports access is denied | Deliverable.link_status = flagged; Deliverable.status remains pending | FEAT-06.SPEC-001 shows the inline message "This link couldn't be reached. Check that it's shared and try again."; the deliverable is not shown to the client | FEAT-06.SPEC-001 |
| Reachability service unavailable | The check itself cannot be performed (the capability behind the check is unreachable, not the target link) | Deliverable.link_status remains pending; no Active transition | FEAT-06.SPEC-001 shows "Couldn't verify this link right now. Try again in a moment." with a Retry option that re-runs the check on the same URL | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Milestone -- reference and current status, to attach the Deliverable.
**Creates:** Deliverable record (kind, milestone, uploaded_at, link_status) when a link is first submitted.
**Updates:** Deliverable.link_status (pending -> reachable or flagged), Deliverable.status (-> Active only when reachable). Milestone.status (-> "Deliverable Uploaded") only when reachable.
**Deletes:** None.

## Business Rules

- XBR-12: the client is never notified, and the deliverable is never shown to the client, while link_status is flagged or pending -- only a reachable link produces the deliverable-ready signal.
- The reachability check runs once per submission; a flagged link requires Nadia to edit and re-submit before another check runs (no automatic periodic re-check).
- This automation defers all link-format validation to FEAT-06.SPEC-005 -- it only ever receives links that have already passed the format check, so it never itself judges whether a URL is well-formed or from a recognized domain.

## Edge Cases

- **Link target requires sign-in the product cannot provide** -- Treated as flagged: the check cannot confirm reachability without credentials it does not hold, so it reports the same "couldn't be reached" outcome rather than a distinct "requires sign-in" message, since the freelancer's remedy (share the link publicly or with link-access) is the same either way.
- **Link resolves slowly (large file behind the link, or a slow third-party service)** -- The check waits for a definitive response rather than timing out prematurely; if no response arrives within a bounded wait, the outcome is "Reachability service unavailable" with a Retry option, not "flagged" -- a slow response is not evidence the link is broken.
- **Nadia edits a flagged link to point at a completely different asset** -- Treated as a fresh submission: the check re-runs against the new URL with no memory of the prior flagged attempt influencing the new result.
- **Concurrent trigger firing (two link submissions for the same milestone with no existing deliverable, from two sessions)** -- Each check runs independently against its own pending Deliverable record; whichever reaches reachable first becomes the milestone's round-1 deliverable, and the second is rejected-with-refresh per FEAT-06.SPEC-001's concurrent-attach edge case.
- **Trigger fires while a previous run is in flight (Nadia taps Attach Deliverable twice on the same link before the first check returns)** -- The second tap is ignored while the first check is in progress, per FEAT-06.SPEC-001's debounced-button interaction; only one check runs per submission.
- **Linked asset is later moved or its sharing permissions are revoked, after the deliverable was already marked reachable** -- Out of scope for this automation, which checks reachability once at attach time only; a subsequently broken link surfaces to the client as an unreachable open attempt, which is a client-facing behavior owned by FEAT-07, not re-checked here.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Deliverable Upload) | Triggered by (inbound) | Link submission and re-submission both start this automation |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | References (inbound) | Link format validation runs before this automation is invoked |
| FEAT-06.SPEC-006 (Deliverable Ready Notification) | Affects (outbound) | A reachable outcome is the trigger this notification waits for |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Affects (outbound) | Displays this automation's link_status badge |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | A successful link attach writes an append-only trail entry (XBR-05) |

## Analytics and Success Signals

- **deliverable_linked** (link domain: figma / drive / dropbox, milestone reference) -- N/A -- no Stage 2 metric measures linked-asset attachment specifically; Deliverable Upload Reliability is defined in terms of file transfers, so this event is retained for feature-level visibility only.
- **upload_failed** (failure reason: link_unreachable, link domain) -- N/A -- FEAT-06.SPEC-001's Analytics section establishes that "Deliverable Upload Reliability" is defined in terms of file transfers, so this link-path event, like SPEC-001's `deliverable_linked`, is N/A for that metric; retained for feature-level visibility only.

## Acceptance Criteria

**FEAT-06.SPEC-004-AC-01:** Given Nadia submits a valid, publicly shared Figma link, when the reachability check runs, then link_status is set to reachable, the Deliverable becomes Active, the owning Milestone's status becomes "Deliverable Uploaded", and FEAT-06.SPEC-006 is triggered.

**FEAT-06.SPEC-004-AC-02:** Given Nadia submits a Google Drive link that fails to resolve, when the reachability check runs, then link_status is set to flagged, the Deliverable's status remains pending, and FEAT-06.SPEC-001 shows "This link couldn't be reached. Check that it's shared and try again."

**FEAT-06.SPEC-004-AC-03:** Given a deliverable's link is flagged, when Nadia checks the client-facing view before fixing it, then the deliverable is never shown to the client as broken -- it simply does not appear as ready.

**FEAT-06.SPEC-004-AC-04:** Given Nadia edits a flagged link and re-submits it, when the check re-runs, then it evaluates only the new URL, independent of the prior flagged result.

**FEAT-06.SPEC-004-AC-05:** Given the reachability check itself cannot run because the underlying service is unavailable, when this occurs, then Nadia sees "Couldn't verify this link right now. Try again in a moment." with a Retry option, distinct from a flagged link.

**FEAT-06.SPEC-004-AC-06:** Given a linked asset requires sign-in the product cannot provide, when the check runs, then the outcome is flagged with the same "couldn't be reached" message as any other unreachable link.

**FEAT-06.SPEC-004-AC-07:** Given a linked asset resolves slowly but eventually responds within the bounded wait, when the response arrives, then the check completes normally rather than timing out prematurely.

**FEAT-06.SPEC-004-AC-08:** Given two of Nadia's sessions submit different links for the same milestone with no existing deliverable, when both checks return reachable, then the first to complete becomes the round-1 Active deliverable and the second is rejected-with-refresh toward the Replace flow.

**FEAT-06.SPEC-004-AC-09:** Given Nadia taps Attach Deliverable twice rapidly on the same link, when the second tap occurs, then it is ignored while the first check is in progress and only one check runs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (initial submission, re-submission after flag) | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
