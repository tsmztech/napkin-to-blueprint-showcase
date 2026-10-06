---
document_type: spec
spec_type: automation
spec_id: FEAT-01.SPEC-006
spec_name: Archive Impact Check
spec_slug: archive-impact-check
parent_feature: FEAT-01
parent_feature_name: Service & Pricing Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 10
---

# Automation Spec: Archive Impact Check

## Overview

**Name:** Archive Impact Check
**ID:** FEAT-01.SPEC-006
**Type:** Automation
**Purpose:** On an archive request, checks for upcoming bookings referencing the service and surfaces an impact warning before the Pro confirms the archive.
**Parent Feature:** FEAT-01 -- Service & Pricing Management

## Scope and Non-Goals

**In Scope:**
- Counting upcoming Bookings (Pending Payment, Confirmed, or Awaiting Outcome, with a future start_time) that reference the service being archived
- Deciding whether to show an impact warning (count of 1 or more) or proceed directly (count of zero)
- Handing the Pro's confirmation or cancellation back to FEAT-01.SPEC-003 to complete or abandon the archive

**Non-Goals:**
- Setting the service's status to Archived -- that write is performed by FEAT-01.SPEC-003 once this automation's outcome and the Pro's confirmation both allow it; this automation only checks and warns
- Resolving the conflict an archived service's upcoming bookings create -- that conflict is flagged on the Pro's booking management surface (FEAT-30) per XBR-11, and resolved there by an explicit Pro decision; this automation never cancels, reschedules, or otherwise touches those bookings
- Guaranteeing the locked price, duration, and deposit of the counted bookings remain unchanged -- that guarantee is FEAT-01.SPEC-005's, not this automation's; this automation only counts and warns
- Cross-pro or cross-client visibility of the count -- excluded per scope-boundaries.md SC-03: the count is shown only to the requesting Pro, about their own account's bookings

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro taps "Archive this service" | FEAT-01.SPEC-003 (Edit Service) | Fires immediately on tap, before any confirmation dialog is shown | The service's identifier, the Pro Account, and the current date/time in the Pro's timezone |

## Processing Logic

1. Receive the archive request for the specified service from FEAT-01.SPEC-003, including the service's identifier and the Pro Account it belongs to.
2. Read all Bookings that reference this service and belong to this Pro Account.
3. Filter to Bookings in an upcoming state -- Pending Payment, Confirmed, or Awaiting Outcome -- whose start_time is in the future relative to the current time in the Pro's timezone.
4. Count the filtered Bookings.
5. If the count is zero, signal FEAT-01.SPEC-003 to proceed directly to archiving, with no warning shown to the Pro.
6. If the count is one or more, return that count to FEAT-01.SPEC-003 for display in an impact warning modal, and hold the archive pending the Pro's explicit confirmation.
7. On the Pro's confirmation, signal FEAT-01.SPEC-003 to set the service's status to Archived, per FEAT-01.SPEC-005's lock guarantee -- none of the counted bookings are altered.
8. On the Pro's cancellation of the warning, take no action -- the service remains Active and the archive does not proceed.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No upcoming bookings | The filtered count is zero | None from this automation -- the archive write happens in FEAT-01.SPEC-003 | No warning modal shown; the archive confirmation flow proceeds directly to the success toast | FEAT-01.SPEC-003 |
| Upcoming bookings found | The filtered count is one or more | None yet -- pending the Pro's decision | Impact warning modal: "{count} upcoming booking(s) reference this service. They will be honored as booked; the service will no longer appear for new bookings." with Confirm and Cancel options | FEAT-01.SPEC-003 |
| Pro confirms archive | The Pro taps Confirm on the impact warning modal | Service.status set to Archived (written by FEAT-01.SPEC-003); the counted bookings are never altered, per FEAT-01.SPEC-005 | Success toast: "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected." Navigate to FEAT-01.SPEC-001 (Archived filter) | FEAT-01.SPEC-001, FEAT-01.SPEC-003, FEAT-01.SPEC-005 |
| Pro cancels the warning | The Pro taps Cancel on the impact warning modal | None | Modal closes; Edit Service screen remains open; service remains Active | FEAT-01.SPEC-003 |
| Automation failure (count could not be determined) | The Booking read fails or times out | None | Blocking error banner on FEAT-01.SPEC-003: "Couldn't check upcoming bookings for this service. Try again." with a Retry action -- the archive is never permitted to proceed without a successful count, since guessing "zero" could hide a real conflict | FEAT-01.SPEC-003 |

## Data Model

**Reads:** Booking -- service reference, state, and start_time fields, scoped to the requesting Pro Account. Service -- the service's identifier and current status (to short-circuit if it is somehow already Archived).
**Creates:** None.
**Updates:** None -- Service.status is written by FEAT-01.SPEC-003, not by this automation.
**Deletes:** None.

## Business Rules

- "Upcoming" is defined as: Booking state is Pending Payment, Confirmed, or Awaiting Outcome, and start_time is in the future relative to the Pro's current time. A Booking already Completed, No-Show, Cancelled, or Expired never counts toward the impact warning, regardless of how recently it ended.
- XBR-11: this automation surfaces the count but never resolves the resulting conflict itself -- the conflict created by archiving with upcoming bookings is flagged on the Pro's booking management surface (FEAT-30) for an explicit Pro decision, independent of this automation's own outcome.
- The impact warning is informational, not a block -- the Pro may still archive a service with any number of upcoming bookings; this automation never prevents the archive outright, it only ensures the Pro sees the count first.
- A zero count is only ever reported after a successful read -- an automation failure never resolves to "assume zero," since that could silently hide a real conflict from the Pro.

## Edge Cases

- **A new booking for this service is completed by a client in the moments between the warning being shown and the Pro confirming** -- The archive proceeds based on the Pro's confirmation of the count already shown; the newly created booking is honored the same as any other upcoming booking, per FEAT-01.SPEC-005, but is simply not reflected in the count the Pro already saw, since that count is a snapshot, not a live guarantee.
- **The service is already Archived when the archive is attempted again (e.g., a stale screen)** -- The automation short-circuits with the message "This service is already archived," and FEAT-01.SPEC-003 routes the Pro back to FEAT-01.SPEC-001 without re-running the count.
- **Concurrent trigger firing (Pro taps Archive for the same service from two signed-in devices around the same time)** -- Each device runs its own independent check and, if applicable, its own warning modal. Whichever device's confirmation completes first sets the service to Archived; the second device's confirmation (if it proceeds) is a redundant no-op, since the service is already Archived -- its screen refreshes to reflect the current state, per the dependency map's Service Contention note.
- **Trigger fires while a previous run is in flight for the same service** -- FEAT-01.SPEC-003 disables the "Archive this service" action while a check is in progress, so a second run for the same service cannot start from the same screen session. Runs for different services proceed independently and never queue behind each other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Edit Service) | Triggered by (inbound) | Fires when the Pro taps "Archive this service" |
| FEAT-01.SPEC-003 (Edit Service) | Affects (outbound) | Returns the count (or the direct-proceed signal) for the Edit Service screen to display or act on |
| FEAT-01.SPEC-001 (Service List) | Affects (outbound) | Pro is returned here, Archived filter, after a confirmed archive |
| FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation) | Enforced rule (inbound) | This automation is an enforcing spec of that rule: it applies the rule's authorization check (Pro only) to the underlying Archive action it gates; it applies no field validation |
| FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time) | References (outbound) | The counted bookings' locked price, duration, and deposit are guaranteed unaffected by that spec, once the archive completes |
| FEAT-30 (Pro Booking Management) | References (outbound) | The conflict created by archiving a service with upcoming bookings is surfaced on that feature's booking management surface, per XBR-11; this automation originates that flag but does not resolve it |

## Analytics and Success Signals

- **archive_impact_checked** (result: no_upcoming / upcoming_found; upcoming_count) -- N/A -- success-metrics.md's only metric connected to this feature, "Service Setup Confidence," measures the add/edit attempt experience; no metric connected to Service & Pricing Management tracks archive-impact outcomes.
- **archive_confirmed_with_upcoming_bookings** (upcoming_count at time of confirmation) -- N/A -- same reason as above; this is a diagnostic signal for the Pro's own operational history (visible via FEAT-16), not a signal any success metric in this feature's slice tracks.
- **archive_impact_check_failed** (reason: read_failure / timeout) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-01.SPEC-006-AC-01:** Given Talia taps "Archive this service" on a service with zero upcoming bookings, when the check completes, then the archive proceeds directly with no warning modal shown.

**FEAT-01.SPEC-006-AC-02:** Given Talia taps "Archive this service" on a service with two upcoming Confirmed bookings, when the check completes, then an impact warning modal reads "2 upcoming booking(s) reference this service. They will be honored as booked; the service will no longer appear for new bookings." with Confirm and Cancel options.

**FEAT-01.SPEC-006-AC-03:** Given Talia sees the impact warning modal showing two upcoming bookings, when she taps Confirm, then the service is set to Archived, both bookings remain unaltered, and she sees the toast "{Service name} archived -- it's no longer bookable, and existing appointments are unaffected."

**FEAT-01.SPEC-006-AC-04:** Given Talia sees the impact warning modal, when she taps Cancel, then the modal closes, the service remains Active, and no data changes.

**FEAT-01.SPEC-006-AC-05:** Given Talia's booking read fails while checking for upcoming bookings, when the failure occurs, then a blocking error banner reads "Couldn't check upcoming bookings for this service. Try again." and the archive does not proceed until a successful check completes.

**FEAT-01.SPEC-006-AC-06:** Given a service has one upcoming booking in Pending Payment and one in Completed, when Talia archives it, then the impact warning counts only the Pending Payment booking, since Completed bookings never count as upcoming.

**FEAT-01.SPEC-006-AC-07:** Given a client completes a new booking for a service in the few seconds between Talia's impact warning being shown and her confirming, when she confirms, then the archive proceeds based on the count she saw, and the new booking is honored exactly as any other upcoming booking per FEAT-01.SPEC-005.

**FEAT-01.SPEC-006-AC-08:** Given Talia attempts to archive a service that a stale screen still shows as Active but that is already Archived, when the automation runs, then it reports "This service is already archived" and returns her to FEAT-01.SPEC-001 without re-running the count.

**FEAT-01.SPEC-006-AC-09:** Given Talia taps Archive for the same service from her phone and her tablet within moments of each other, when both checks and confirmations proceed, then whichever device's confirmation completes first archives the service, and the second device's confirmation is a no-op that simply refreshes to the current Archived state.

**FEAT-01.SPEC-006-AC-10:** Given Talia archives a service with upcoming bookings, when the archive completes, then those bookings' conflicts are made available on her booking management surface (FEAT-30) per XBR-11, for her to resolve there explicitly.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (Pro taps Archive on FEAT-01.SPEC-003) | 1 |
| Outcome Paths | 5 (no upcoming, upcoming found, confirmed, cancelled, failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
