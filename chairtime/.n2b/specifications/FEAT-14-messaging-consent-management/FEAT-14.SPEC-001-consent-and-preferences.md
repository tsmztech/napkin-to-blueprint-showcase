---
document_type: spec
spec_type: screen
spec_id: FEAT-14.SPEC-001
spec_name: Consent & Preferences
spec_slug: consent-and-preferences
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Consent & Preferences

## Overview

**Name:** Consent & Preferences
**ID:** FEAT-14.SPEC-001
**Type:** Screen
**Purpose:** The consent section of the client's Preferences screen: the client's current texting consent status for this Pro, with an action to opt back in to texting when it is currently off. This section is hosted inside FEAT-06.SPEC-005 (Consent & Email Preferences), the single client-facing Preferences screen; it is not a separately reachable screen.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

**Hosting note:** FEAT-06.SPEC-005 (Consent & Email Preferences) is the single client-facing Preferences screen; FEAT-06.SPEC-003 and FEAT-06.SPEC-004 navigate to it. This spec defines the consent section that FEAT-06.SPEC-005 hosts and keeps the consent-control behavior (status display and re-grant) governed by FEAT-14's rules (XBR-15). FEAT-06.SPEC-005 owns the surrounding screen, its header, and the email field.

## Scope and Non-Goals

**In Scope:**
- Displaying the client's current texting consent status for this Pro (on/off), derived from FEAT-14.SPEC-007's textability determination
- A plain-language explanation that messages arrive by email when texting is off
- An action to re-grant (opt back in to) texting when it is currently off

**Non-Goals:**
- Hosting the screen itself (header, back navigation, page-level loading and error states, entry points) or editing the client's own email address -- owned by FEAT-06.SPEC-005 (Consent & Email Preferences), which holds the Client entity's email field write path; this feature never writes Client contact fields (Feature Dependency Map: Client is a read-only referenced entity for FEAT-14).
- Revoking texting consent from this screen -- excluded per this feature's own Key Capabilities: the product's only revoke paths are the opt-out link (FEAT-14.SPEC-002) and a STOP reply (FEAT-14.SPEC-004); offering a second, in-app revoke control here would create a third precedence case for FEAT-14.SPEC-006 to resolve with no product benefit.
- Showing consent history or past state transitions -- excluded per scope-boundaries.md SC-22: only the current status is client-facing; the timestamp and exact wording shown are retained solely as compliance evidence, never surfaced as a browsable log.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Client opens the Preferences screen (reached from "Preferences" on FEAT-06.SPEC-003 and FEAT-06.SPEC-004); this consent section renders inside it | The Pro relationship already scoped by FEAT-06.SPEC-005's viewing session; current consent status loads fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Their own texting consent status for this one Pro | Re-grant texting consent (only when currently off) | -- |
| The Pro (Talia) | No | No | The Pro sees textability for planning through FEAT-12's dashboard only (View, never override, per the Access Matrix's Messaging & Consent row); this client-facing screen has no Pro entry point |
| Platform Operator (Support) | No | No | Support's View-only access to Messaging Consent (Access Matrix) is delivered through its own troubleshooting view, never this client-facing screen (scope-boundaries SC-05) |
| Unauthenticated | No | No | Reachable only as the consent section of FEAT-06.SPEC-005, inside an already-authenticated viewing session; a direct, unauthenticated attempt to reach the host screen is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The viewing session lasts only for the current page; on return after the underlying access link has become Used or Expired, the client is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Section heading "Texting" inside FEAT-06.SPEC-005's Preferences screen; the screen title and back arrow belong to the host screen (FEAT-06.SPEC-005).

**Body:** A single status section:
- A status line: "Texting: on" or "Texting: off", reflecting FEAT-14.SPEC-007's current textability determination
- When off, a plain-language line directly below: "Your confirmations and reminders go to your email instead."
- When off, a "Turn texting back on" button below that line
- When on, no button is shown -- the status line is the entire content

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Status section stacks vertically, full width.
- **Medium size class and above:** Same vertical order, content column capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (host screen, FEAT-06.SPEC-005) | Tap | Handled by FEAT-06.SPEC-005, which returns to FEAT-06.SPEC-003 or FEAT-06.SPEC-004 | Host screen closes | Standard navigation transition |
| "Turn texting back on" button (shown only when status is off) | Tap | Triggers FEAT-14.SPEC-005 (Consent Re-Grant Action) | Button shows a brief loading state | Success: status line updates to "Texting: on", button disappears, confirmation "Texting turned back on". Failure: inline message "Couldn't update your preference. Try again." with the button re-enabled |
| "Turn texting back on" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Within this section: status line -> "Turn texting back on" button (when shown); the host screen's back arrow precedes the section.
- **Announcements:** The confirmation "Texting turned back on" and the failure message are announced to assistive technology as they appear; the status line's change from "off" to "on" is announced alongside the confirmation.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the status line will appear | Screen first opens | Data finishes loading |
| Populated (Texting On) | "Texting: on" shown, no button | Data loads with active consent (Granted or Re-granted) | Client navigates away, or the client's consent is revoked elsewhere and this screen is reloaded |
| Populated (Texting Off) | "Texting: off" with the email-fallback line and "Turn texting back on" button | Data loads with Revoked consent | Client's re-grant action succeeds |
| Saving | The button shows a loading state | Client taps "Turn texting back on" | The write completes or fails |
| Error | Error banner "We couldn't load your preferences. Try again." in place of the status section, with a retry action | The initial status load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline. Reconnect to update your texting preference." at the top; the status line remains visible but the "Turn texting back on" button is disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and the button re-enables |

## Validation Rules

**Option B -- Inline:**

This screen accepts no free-text input. The single action (re-grant) has no field-level validation; its only precondition is that the current status is off, which the screen itself enforces by hiding the button when status is on.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (host screen's control) | FEAT-06.SPEC-005 returns the client to the screen they arrived from (FEAT-06.SPEC-003 or FEAT-06.SPEC-004) | FEAT-06 |

## Data Model

**Creates:** None.
**Reads:** Messaging Consent -- channel, state, timestamp for this Client and Pro, resolved through FEAT-14.SPEC-007's textability determination.
**Updates:** None directly -- the re-grant write is performed by the triggered automation, FEAT-14.SPEC-005.
**Deletes:** None.

## Business Rules

- The status shown here is exactly the value FEAT-14.SPEC-007 computes -- this screen never derives its own consent logic or caches a stale value across sessions.
- The re-grant action is the only consent change this screen can cause; it is executed entirely by FEAT-14.SPEC-005, and any concurrent-write precedence (e.g., a STOP reply arriving close together) is governed by FEAT-14.SPEC-006.
- XBR-15 governs the underlying behavior this status reflects: no text is sent without active texting consent; a revoke is honored on the very next message; a re-grant takes effect for the client's next message.

## Edge Cases

- **Status becomes stale between load and any action on this screen** -- This screen is a load-time snapshot, not live-updating. If the client's consent changes elsewhere (e.g., a STOP reply arrives) while this screen is open showing "on", the screen does not update in place; the next time the client opens this screen, it reflects the reconciled state per FEAT-14.SPEC-006. No action is available while showing "on" that could conflict with an external change.
- **Client taps "Turn texting back on" at effectively the same moment a STOP reply arrives for the same phone number** -- Per FEAT-14.SPEC-006, the most recent explicit client action by timestamp wins; if the STOP reply's timestamp is later, the write still succeeds as a re-grant attempt, but the resulting state is Revoked, and the client sees "Texting: off" on their next view of this screen rather than the "on" state the tap requested.
- **Client double-taps "Turn texting back on"** -- The second tap is ignored while the first write is in flight (button in loading state).
- **Network failure during the re-grant write** -- Inline message "Couldn't update your preference. Try again." appears; the status line remains unchanged (off) and the button re-enables for another attempt.
- **Client navigates away while the re-grant write is in flight** -- The write completes in the background regardless of navigation; the status is correct on the client's next visit to this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Hosted by (inbound) | The single client-facing Preferences screen renders this consent section; FEAT-06.SPEC-003 and FEAT-06.SPEC-004 reach this section only through it |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Supplies the current status this screen displays |
| FEAT-14.SPEC-005 (Consent Re-Grant Action) | Triggers (outbound) | The "Turn texting back on" button fires this automation |
| FEAT-14.SPEC-006 (Concurrent Consent Update Resolution) | References (inbound) | Governs precedence if a re-grant races a revoke |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| consent_preferences_viewed | current_status (on / off) | Screen finishes loading | N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references texting-consent behavior; retained as an operational signal so this screen's usage is observable |
| consent_regrant_attempted | outcome (succeeded / failed / superseded_by_revoke) | Client taps "Turn texting back on" and the write resolves | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-14.SPEC-001-AC-01:** Given Riley's texting consent for Talia is currently Revoked, when she opens this screen, then it shows "Texting: off", the email-fallback line, and a "Turn texting back on" button.

**FEAT-14.SPEC-001-AC-02:** Given Riley's texting consent is currently Granted, when she opens this screen, then it shows "Texting: on" with no button.

**FEAT-14.SPEC-001-AC-03:** Given Riley taps "Turn texting back on", when the write succeeds, then the status line updates to "Texting: on", the button disappears, and she sees "Texting turned back on".

**FEAT-14.SPEC-001-AC-04:** Given Riley taps "Turn texting back on" and the write fails, when the failure is reported, then she sees "Couldn't update your preference. Try again." and the button remains available.

**FEAT-14.SPEC-001-AC-05:** Given Riley taps "Turn texting back on" at the same time a STOP reply for her phone number is processed with a later timestamp, when precedence resolves per FEAT-14.SPEC-006, then her status shows "Texting: off" on her next view of this screen.

**FEAT-14.SPEC-001-AC-06:** Given the initial load of Riley's status fails, when the failure occurs, then an error banner "We couldn't load your preferences. Try again." appears with a retry action.

**FEAT-14.SPEC-001-AC-07:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then a banner explains she is offline and the "Turn texting back on" button becomes disabled.

**FEAT-14.SPEC-001-AC-08:** Given Riley taps "Turn texting back on" twice rapidly, when the second tap registers, then it is ignored while the first write is still in progress.

**FEAT-14.SPEC-001-AC-09:** Given Riley looks for a way to edit her email address inside this consent section, when she reviews the layout, then no such control exists in the section -- the email field lives elsewhere on the host screen, FEAT-06.SPEC-005.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 4 (loading, error, saving, offline) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
