---
document_type: spec
spec_type: screen
spec_id: FEAT-08.SPEC-003
spec_name: Reminder Reply Acknowledgment
spec_slug: reminder-reply-acknowledgment
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Reminder Reply Acknowledgment

## Overview

**Name:** Reminder Reply Acknowledgment
**ID:** FEAT-08.SPEC-003
**Type:** Screen
**Purpose:** Confirms to the client, after they tap "I'll be there" in a reminder, that their attendance was recorded -- and handles the cases where the tapped link no longer works.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The landing page a client reaches after FEAT-08.SPEC-008 records an "I'll be there" acknowledgment
- The Error/Permission-Denied states for a booking-specific link that is expired, already used, or belongs to a passed appointment

**Non-Goals:**
- Processing the tap itself (recording the acknowledgment on the Booking) -- owned by FEAT-08.SPEC-008 (Reminder Reply Routing); this screen only renders after that processing completes.
- The "I need to reschedule" path -- that tap routes directly into FEAT-10 (Client-Initiated Cancel/Reschedule) and never reaches this screen, per the Brief's Internal Dependency Map.
- Any booking management action (viewing full booking details, cancelling, rescheduling) -- this screen is a one-line confirmation, not a booking dashboard; those actions live in FEAT-06 (Client Booking Identity) and FEAT-10, reached only if the client explicitly navigates onward.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-002 (Appointment Reminder Message) via FEAT-08.SPEC-008 (Reminder Reply Routing) | Client taps "I'll be there" in a text or email reminder | Booking reference (from the tapped booking-specific manage link); the acknowledgment has already been recorded on the Booking by the time this screen renders |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, scoped to the one Booking the tapped link names | Navigate onward to manage the booking (via a link to FEAT-06) | -- |
| The Pro (Talia) | Not applicable -- this screen is never navigated to by the Pro; the Pro instead sees the "I'll be there" status reflected on the dashboard (FEAT-12) | No | If a Pro somehow opens the link, it is treated as any other visitor without that specific client's session context: the same booking-specific link content is shown (the link carries no signed-in-role distinction), since the link's scope, not a login, is what gates access |
| Platform Operator (Support) | Not applicable -- Support has no client-link access; Support views delivery status via FEAT-16, never by using or bypassing a client's access link (XBR-24) | No | Support is never issued or expected to open a booking-specific manage link; there is no dedicated denial experience because this path does not exist for Support |
| Unauthenticated | Yes -- this screen requires no sign-in; the booking-specific link itself is the credential (XBR-18) | Yes, for the single acknowledgment already recorded before arrival | N/A -- there is no signed-in-only version of this screen; a person with no valid link sees the Expired/Invalid state below instead of the acknowledgment |
| Expired session | N/A -- this screen has no session concept; each visit is scoped entirely to the tapped link | N/A | A visit with an expired or already-used link shows the Expired/Invalid Link state (see States), which reads "This link is no longer active. Request a new link from your confirmation or reminder message, or contact {pro_display_name} directly." |

## Layout and Content

**Header:** No back arrow (this screen is a landing destination, not a step in a flow the client navigated forward through) and no page chrome beyond the Pro's display name, so the client immediately recognizes whose appointment this concerns.

**Body:** A single centered content block:
- A confirmation icon/mark (non-interactive, decorative)
- Heading: "You're all set, {client_first_name}"
- A one-line summary restating the appointment: "{service_name} with {pro_display_name} on {appointment_date} at {appointment_time}"
- A secondary line: "We've let {pro_display_name} know you're coming."
- A single link/button: "Manage my booking" (navigates to FEAT-06's booking view via the same booking-specific link's scope)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width content block as described, vertically centered on the visible viewport.
- **Medium size class and above:** Same single-column content, capped at a consistent platform-wide narrow-content width and horizontally centered; no structural change beyond width capping, since this screen has no dense content that benefits from a wider layout.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Manage my booking" link | Tap | Navigate to FEAT-06 (Client Booking Identity), scoped to the same Booking | Screen transitions to the booking view | Standard navigation transition |
| Confirmation icon and summary text | -- | Display-only, non-interactive | None | None |

### Accessibility Notes

- **Focus order:** Heading is announced first on screen load (as the page's primary landmark), followed by the summary line, then the "Manage my booking" link.
- **Load announcement:** On successful load, the heading "You're all set, {client_first_name}" is announced to assistive technology as the page's content, since there is no separate loading transition for the client to perceive (the acknowledgment was already recorded before this screen renders).
- **Keyboard alternatives:** The single interactive element ("Manage my booking") is a standard link, fully reachable and activatable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Acknowledged (default) | The confirmation content described in Layout and Content | The tapped link is valid, unexpired, and its acknowledgment was successfully recorded by FEAT-08.SPEC-008 | Client navigates onward via "Manage my booking" or closes the page |
| Expired/Invalid Link | Heading "This link is no longer active." Body: "Request a new link from your confirmation or reminder message, or contact {pro_display_name} directly." No further action available on this screen. | The tapped link has expired (the appointment has passed, XBR-18) or was already used from a prior visit that already recorded the acknowledgment | Client leaves the page; there is no in-page recovery path, since a booking-specific link is never reissued from this screen |
| Loading | A brief, minimal loading indicator while the link's validity and the Booking reference are resolved | Immediately on tapping the reminder link, before resolution completes | Resolution completes, transitioning to Acknowledged or Expired/Invalid Link |
| Error | Message: "Something went wrong loading your confirmation. Try the link again from your reminder message." with no retry button on this screen (the client re-opens the original message's link) | The Booking reference cannot be resolved for a reason other than expiry (a transient failure) | Client re-opens the link from their original reminder message |
| Offline/Degraded | N/A -- this screen requires connectivity to resolve the link and display the current acknowledgment; without connectivity the link simply fails to load, which the client's device reports as a standard page-load failure, not a screen-level offline state this spec defines | Connectivity lost while attempting to open the link | Connectivity restored and the link is reopened |

## Validation Rules

Not applicable -- this screen accepts no user input; the acknowledgment was already recorded by FEAT-08.SPEC-008 before this screen renders.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Manage my booking" tap | FEAT-06 (Client Booking Identity), booking view | FEAT-06 |
| Expired/Invalid Link -- no in-page action | -- (client leaves or re-opens the original message) | -- |

## Data Model

**Creates:** None -- the acknowledgment itself (Booking.attendance_reply) is written by FEAT-08.SPEC-008 before this screen loads; this screen only reads the result.
**Reads:** Booking -- service, start_time, attendance_reply (to confirm it reflects "I'll be there"); Client -- name (for the greeting); Pro Account -- display_name.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The acknowledgment this screen confirms is written exactly once per booking by FEAT-08.SPEC-008; this screen never re-records or overwrites it, even on a repeat visit to the same valid link.
- Access to this screen is governed entirely by the booking-specific Access Link's own scope and expiry rules (XBR-18), owned by FEAT-06/FEAT-08.SPEC-010 -- no separate sign-in or session state exists for this screen.
- A single-use "already used" state is deliberately not shown as an error: revisiting a still-unexpired link after its acknowledgment was recorded simply re-renders the same Acknowledged content, since re-confirming a true fact is harmless (unlike a single-use on-demand access link, this booking-specific link is reusable until the appointment passes, per the Access Link entity's lifecycle in feature-dependency-map.md).

## Edge Cases

- **Client opens the link twice from the same device** -- The second visit re-renders the same Acknowledged state; no error and no re-processing, since the underlying Booking.attendance_reply is unchanged.
- **Client opens the link on a second device after already acknowledging on the first** -- Same as above: this booking-specific link is not single-use, so both devices show the Acknowledged state consistently.
- **Client forwards the reminder message to someone else, who taps the link** -- The recipient sees the same Acknowledged (or Expired) content scoped to that one booking; they cannot navigate to any other booking, client, or Pro data through this screen (XBR-18).
- **Appointment passes between the reminder being sent and the client tapping the link** -- The link has expired per XBR-18; the client sees the Expired/Invalid Link state rather than a stale acknowledgment.
- **The Pro reschedules the booking after the reminder was sent but before the client taps "I'll be there"** -- FEAT-08.SPEC-010 issues a fresh manage link on a Pro-initiated reschedule; the old reminder's link no longer resolves to the current appointment and shows the Expired/Invalid Link state, directing the client to their latest message instead.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Navigation (inbound) | The "I'll be there" tap in the reminder leads here |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | References (inbound) | Records the acknowledgment this screen confirms |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Owns the link's scope, validity, and expiry that gate this screen |
| FEAT-06 (Client Booking Identity) | Navigation (outbound) | "Manage my booking" opens the full booking view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| reminder_acknowledgment_viewed | link_state (valid / expired) | Screen loads | supports success-metrics.md: "Reminder Response Rate" |
| reminder_acknowledgment_manage_tapped | -- | Client taps "Manage my booking" | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-08.SPEC-003-AC-01:** Given Riley taps "I'll be there" in her reminder text, when FEAT-08.SPEC-008 records the acknowledgment, then she lands on this screen showing "You're all set, Riley" with her appointment summary.

**FEAT-08.SPEC-003-AC-02:** Given Riley is on the Acknowledged state, when she taps "Manage my booking", then she navigates to FEAT-06's booking view for the same appointment.

**FEAT-08.SPEC-003-AC-03:** Given Riley's appointment has already passed, when she taps the "I'll be there" link from an old reminder, then she sees "This link is no longer active." with no acknowledgment content shown.

**FEAT-08.SPEC-003-AC-04:** Given Riley opens the same valid link twice from two different devices, when each visit resolves, then both show the identical Acknowledged content, with no error on the second visit.

**FEAT-08.SPEC-003-AC-05:** Given Riley forwards her reminder to a friend and the friend taps the link, when the link resolves, then the friend sees only Riley's one booking's acknowledgment content and cannot reach any other booking or client data.

**FEAT-08.SPEC-003-AC-06:** Given the Pro reschedules Riley's booking after sending the reminder, when Riley later taps the original "I'll be there" link, then she sees the Expired/Invalid Link state, since a fresh link was issued for the rescheduled booking.

**FEAT-08.SPEC-003-AC-07:** Given the link resolves but the Booking reference cannot be loaded due to a transient failure, when the page attempts to render, then Riley sees "Something went wrong loading your confirmation. Try the link again from your reminder message."

**FEAT-08.SPEC-003-AC-08:** Given Riley's device loses connectivity while opening the link, when the page attempts to load, then the link fails to load as a standard page-load failure, and no partial or stale acknowledgment content is shown.

**FEAT-08.SPEC-003-AC-09:** Given Riley is on the Acknowledged screen, when it renders, then no interactive elements beyond "Manage my booking" are present, and the heading and summary are read-only content.

**FEAT-08.SPEC-003-AC-10:** Given the tapped link is scoped to a Booking with attendance_reply already set to "I'll be there" from a prior visit, when this screen loads again, then it re-renders the same Acknowledged content without re-processing the acknowledgment.

**FEAT-08.SPEC-003-AC-11:** Given a screen reader user reaches this page, when it finishes loading, then the heading "You're all set, {client_first_name}" is announced as the primary landmark, followed by the appointment summary and the "Manage my booking" link in that order.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 | 2 |
| States | 4 (acknowledged, expired/invalid, loading, error) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
