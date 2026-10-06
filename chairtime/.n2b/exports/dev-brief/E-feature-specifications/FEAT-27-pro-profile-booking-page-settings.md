# FEAT-27 — Pro Profile & Booking Page Settings

This chapter covers Pro Profile & Booking Page Settings (FEAT-27), a Important-tier feature. It carries 13 specifications carrying 145 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-27.SPEC-001 | Profile & Booking Page Settings | screen | 15 |
| FEAT-27.SPEC-002 | Booking Link Rename | screen | 12 |
| FEAT-27.SPEC-003 | Timezone & Currency Settings | screen | 12 |
| FEAT-27.SPEC-004 | Pause Bookings | screen | 13 |
| FEAT-27.SPEC-005 | Notification Preferences | screen | 11 |
| FEAT-27.SPEC-006 | Help Request | screen | 10 |
| FEAT-27.SPEC-007 | Booking Link Name Validation & Uniqueness Rule | logic-rule | 12 |
| FEAT-27.SPEC-008 | Currency Lock Rule | logic-rule | 9 |
| FEAT-27.SPEC-009 | Pause State Precedence Rule | logic-rule | 11 |
| FEAT-27.SPEC-010 | Booking Link Forwarding & Reservation Expiry | automation | 10 |
| FEAT-27.SPEC-011 | Automatic Pause Resume | automation | 9 |
| FEAT-27.SPEC-012 | Profile Photo Storage Capability | integration | 11 |
| FEAT-27.SPEC-013 | Help Request Acknowledgment | notification | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Pro Profile & Booking Page Settings

## Summary

**Feature:** Pro Profile & Booking Page Settings
**ID:** FEAT-27
**Description:** The Pro controls how they appear and how their booking page behaves — display name, photo, short intro, studio location, booking link name, timezone and currency, pausing new bookings for a holiday, and which notifications they get — and can ask for help from the same place.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Experience has the client see "the pro's name" on the booking page, its Scale section requires that "timezone and currency must not be hard-coded," and a client of a rented chair or home studio cannot find the appointment without a location — yet no draft feature let the Pro edit any of these after onboarding. Important rather than Core because the booking loop still works on the values set once during setup; MVP because a Pro who moves studio or goes on holiday must be able to change them from day one. The studio's full address appears only in a booked client's confirmation, protecting home-studio pros from publishing their home address to anyone who taps the link. [AUDIT-ADDED: 3 -- the Pro Account entity had no feature for editing its profile, link, timezone, currency, pause state, or notification preferences] [RESEARCH-INFORMED: the visual quality and branding of the booking page is consistently praised by users of the closest competitor, from Capterra/GetApp reviews (HIGH confidence); a photo and short intro serve that without adding design tooling]

**Key Capabilities:**
- Edit the public profile -- display name, photo, short intro, and the general area shown publicly
- Set the full studio address -- shown only in booked clients' confirmations and reminders
- Choose or change the booking link name -- the previous link keeps forwarding so an Instagram bio link never breaks
- Set timezone and currency for the account
- Pause new bookings with an optional message (for example, "on holiday until 3 June") and resume them -- existing bookings are unaffected
- Choose which Pro notifications to receive and how (in-app, text, or email)
- Preview the booking page as a client sees it
- Send a help request to support from settings

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-27.SPEC-001 | Profile & Booking Page Settings | Screen | The Pro, Platform Operator (Support) | The Pro edits display name, photo, intro, general area and studio address, and reaches every other settings sub-flow (link, timezone/currency, pause, notifications, help) and the booking-page preview from here |
| FEAT-27.SPEC-002 | Booking Link Rename | Screen | The Pro, Platform Operator (Support) | The Pro views and changes their booking link name, sees the forwarding guarantee on the old name, and gets suggestions when a name is already taken |
| FEAT-27.SPEC-003 | Timezone & Currency Settings | Screen | The Pro, Platform Operator (Support) | The Pro sets account timezone and currency, sees a warning before a timezone change is saved, and sees currency locked once it has been |
| FEAT-27.SPEC-004 | Pause Bookings | Screen | The Pro, Platform Operator (Support) | The Pro pauses new bookings with an optional message and end date, or resumes them, seeing whether a system-imposed pause is also in effect |
| FEAT-27.SPEC-005 | Notification Preferences | Screen | The Pro, Platform Operator (Support) | The Pro chooses which Pro notifications they receive and on which channel(s) -- in-app, text, or email |
| FEAT-27.SPEC-006 | Help Request | Screen | The Pro | The Pro describes a problem and sends a help request to support from settings |
| FEAT-27.SPEC-007 | Booking Link Name Validation & Uniqueness Rule | Logic/Rule | The Pro | Enforces the 3-40 character format, cross-pro uniqueness, and the reject-with-refresh behavior when another pro claims a name first |
| FEAT-27.SPEC-008 | Currency Lock Rule | Logic/Rule | The Pro | Determines whether currency is still editable, locking it permanently the moment the account's first deposit is taken |
| FEAT-27.SPEC-009 | Pause State Precedence Rule | Logic/Rule | The Pro | Governs how a Pro-chosen pause and a system-imposed (subscription-lapse) pause coexist, including that the Pro's resume toggle cannot clear a system-imposed pause, and that a pause end date cannot be in the past |
| FEAT-27.SPEC-010 | Booking Link Forwarding & Reservation Expiry | Automation | The Pro, The Client | On rename, keeps the old link name forwarding to the new one for at least 12 months, reserving it from reuse by any other pro until that window lapses |
| FEAT-27.SPEC-011 | Automatic Pause Resume | Automation | The Pro, The Client | Resumes bookings automatically when a Pro-chosen pause reaches its end date, without disturbing a still-active system-imposed pause |
| FEAT-27.SPEC-012 | Profile Photo Storage Capability | Integration | The Pro, The Client | External capability that stores, replaces and serves the Pro's profile photo within the size/format limits, so the booking page still works without one |
| FEAT-27.SPEC-013 | Help Request Acknowledgment | Notification | The Pro, Platform Operator (Support) | Confirms to the Pro that their help request was received, carrying the reference support's read-only lookup (FEAT-19) points to |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Edit the public profile -- display name, photo, short intro, general area | FEAT-27.SPEC-001, FEAT-27.SPEC-012 | SPEC-001 is the editing screen; SPEC-012 stores and serves the photo | Phase 2 (Explicit) |
| Set the full studio address | FEAT-27.SPEC-001 | Same screen as the public profile; the field is flagged confirmation-only, never shown on the public page | Phase 2 (Explicit) |
| Choose or change the booking link name | FEAT-27.SPEC-002, FEAT-27.SPEC-007, FEAT-27.SPEC-010 | SPEC-002 is the rename screen; SPEC-007 validates the new name; SPEC-010 keeps the old one forwarding | Phase 2 (Explicit) |
| Set timezone and currency for the account | FEAT-27.SPEC-003, FEAT-27.SPEC-008 | SPEC-003 is the settings screen with the pre-save timezone warning; SPEC-008 governs when currency can still change | Phase 2 (Explicit) |
| Pause new bookings with an optional message and resume them | FEAT-27.SPEC-004, FEAT-27.SPEC-009, FEAT-27.SPEC-011 | SPEC-004 is the pause/resume screen; SPEC-009 governs precedence against a system-imposed pause; SPEC-011 resumes automatically at the chosen end date | Phase 2 (Explicit) |
| Choose which Pro notifications to receive and how | FEAT-27.SPEC-005 | Sole owner of the notification_preferences setting; delivery itself stays with FEAT-08 | Phase 2 (Explicit) |
| Preview the booking page as a client sees it | FEAT-27.SPEC-001 | Inline entry action on the main settings screen that navigates to FEAT-05's existing preview mode and emits booking_page_previewed | Phase 2 (Explicit) |
| Send a help request to support from settings | FEAT-27.SPEC-006, FEAT-27.SPEC-013 | SPEC-006 is the request form; SPEC-013 sends the acknowledgment | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-27.SPEC-007 | Booking Link Name Validation & Uniqueness Rule | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field states five distinct, interacting conditions on one field (format, length, cross-pro uniqueness, ≥12-month forward window, non-claimable reservation), and the Pro Account entity's Contention line names a specific conflict-resolution behavior (reject-with-refresh) -- crosses the standalone-Logic/Rule threshold |
| FEAT-27.SPEC-008 | Currency Lock Rule | Phase 5 (Rule-Constraint Discovery) | XBR-25 (authority FEAT-27) and the Validation & Limits field both state a state-dependent rule ("cannot be changed after the first deposit is taken") that FEAT-07.SPEC-003 already depends on as a charge precondition -- a derivation/authorization rule shared across features, not a simple field validation |
| FEAT-27.SPEC-009 | Pause State Precedence Rule | Phase 5 (Rule-Constraint Discovery) | Coordination note 2 and the Pro Account Contention line describe conditional logic between two independent pause sources (Pro-chosen and FEAT-18's system-imposed) whose interaction the validated FEAT-18 Brief explicitly assigns to this feature -- exactly the "rules that depend on other field values or entity state" trigger for a standalone rule |
| FEAT-27.SPEC-010 | Booking Link Forwarding & Reservation Expiry | Phase 4 (Trigger-Response) + time-based triggers | XBR-27 (authority FEAT-27) describes a multi-step, cross-feature consequence of a rename (old link forwards, name reserved, then released after ≥12 months) with a genuine expiry condition -- processing logic, not a bare inline interaction |
| FEAT-27.SPEC-011 | Automatic Pause Resume | Phase 4 (time-based triggers) | The Primary Flows field states bookings "resume automatically on the chosen date," a scheduled state transition with no user action -- a time-based automation, not an inline screen consequence |
| FEAT-27.SPEC-012 | Profile Photo Storage Capability | Phase 4 (External Dependencies lens) | ASMP-35 names this feature's own file-storage dependency, and the External Touchpoints row "File storage -- Pro profile photos" lists this analysis batch as where it is specified |
| FEAT-27.SPEC-013 | Help Request Acknowledgment | Phase 4 (Notification surfacing) | The Communications field names an acknowledgment with real delivery behavior (it must carry a reference FEAT-19's support view can look up) -- crosses the inline-vs-standalone threshold for Notification specs |

## Entity-Lifecycle Coverage Matrix

**Entity: Pro Account** *(this feature's fields only: profile, booking_link_name, timezone, currency, pause state/message, notification_preferences)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-15.SPEC-004 (Setup Progress Tracking & Resume), created when sign-in completes, per the dependency map's Pro Account Lifecycle line | Not a gap -- this feature only updates the record |
| Read (single) | FEAT-27.SPEC-001, SPEC-002, SPEC-003, SPEC-004, SPEC-005 | Each screen loads the current values of its scoped fields on open, including the onboarding-set values on a first visit | -- |
| Read (list) | N/A | One Pro Account per signed-in Pro; no list surface exists or is needed | -- |
| Update | FEAT-27.SPEC-001 (profile, general area, studio address), SPEC-002 (booking_link_name, via SPEC-007/SPEC-010), SPEC-003 (timezone, currency, via SPEC-008), SPEC-004 (pause state/message/end date, via SPEC-009) SPEC-005 (notification_preferences) | Every writable field this feature owns has an editing screen and, where the field carries conditional logic, a paired Logic/Rule spec | Last-write-wins for ordinary fields, reject-with-refresh for link-name contention and locked currency, per the Pro Account Contention line |
| Delete/Archive | N/A | Owned by FEAT-29, after a 30-day cooling-off period following account closure, per the dependency map's Pro Account Lifecycle line | Not a gap -- explicit ownership elsewhere |
| State Transition | FEAT-27.SPEC-004, SPEC-009, SPEC-011 | This feature owns the Active <-> Paused (Pro-chosen) transition and its precedence against FEAT-18's Active <-> Paused (subscription-lapse) transition; FEAT-29 separately owns Closing/Closed | SPEC-009 is explicit that the Pro's "resume bookings" toggle cannot clear a system-imposed pause |

**Entity: Help Request** *(captured by this feature per Data Notes; not listed as a Shared Data Entity in the dependency map slice -- see Shared Context for this discrepancy)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-27.SPEC-006 | Written when the Pro submits the help request form | -- |
| Read (single) | N/A -- owned by FEAT-19 | Support's read-only lookup of a help request, opened after the Pro sends one, is FEAT-19's screen per coordination note 8 and nav row "FEAT-27 help request -> FEAT-19" | Not a gap: this feature writes, FEAT-19 reads |
| Read (list) | N/A -- owned by FEAT-19 | Same reasoning; no help-request list or inbox exists inside this feature (coordination note 8: "do not invent an external capability for it") | Not a gap |
| Update | N/A | Key Capabilities name only "send a help request," with no edit or withdraw capability stated anywhere in Stage 2 for this feature | -- |
| Delete/Archive | N/A -- indefinite retention, explicit non-goal | No retention or purge policy is stated in Stage 2 for help requests; they are treated as part of the Pro's account activity trail that XBR-24 keeps visible to the Pro, so they are retained indefinitely rather than silently omitted | See Non-Goals |
| State Transition | N/A -- owned by FEAT-19 | Any request-status lifecycle (e.g., open/resolved) belongs to FEAT-19's support-side handling, which is out of scope for this feature per its Out of Scope boundary ("Producing outputs for other features") | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deposit Transaction | FEAT-27.SPEC-008 | Checked to determine whether the account's first deposit has already been taken, which permanently locks currency |
| Booking | FEAT-27.SPEC-003 | Read to compose the pre-save timezone-change warning (for example, how many upcoming bookings will now display converted) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro saves changed display name, photo, intro, general area, or studio address | Write to Pro Account; public booking page reflects the change immediately | Inline in triggering screen | FEAT-27.SPEC-001 |
| Pro uploads or replaces a profile photo | Store the file within size/format limits and make it servable to the public booking page | Standalone Integration | FEAT-27.SPEC-012 |
| Pro submits a new booking link name | Validate format, length, and cross-pro uniqueness before allowing save | Standalone Logic/Rule | FEAT-27.SPEC-007 |
| Booking link name save succeeds | Old name starts forwarding to the new one and is reserved from reuse; reservation is released after at least 12 months | Standalone Automation | FEAT-27.SPEC-010 |
| Booking link name is claimed by another pro between the Pro loading the rename screen and saving | Reject the save and refresh with current availability, per the Pro Account Contention line | Standalone Logic/Rule | FEAT-27.SPEC-007 |
| Pro changes timezone | Show a clear warning that existing bookings keep their real moment in time and will display converted, before the change is saved | Inline in triggering screen | FEAT-27.SPEC-003 |
| Pro attempts to change currency after the first deposit has been taken | Block the change; explain that currency is locked | Standalone Logic/Rule | FEAT-27.SPEC-008 |
| Pro turns on a pause with a message and/or end date | Booking page shows the pause message instead of available times; existing bookings and their reminders are unaffected (XBR-11, XBR-14) | Inline in triggering screen, enforcement is cross-feature | FEAT-27.SPEC-004 (enforced by FEAT-05.SPEC-008) |
| Pause end date is reached | Automatically resume bookings unless a system-imposed pause is still active | Standalone Automation | FEAT-27.SPEC-011 |
| Pro taps "resume bookings" while a subscription-lapse pause is active | Toggle is accepted for the Pro-chosen pause only; the account stays paused until billing is restored (FEAT-18) | Standalone Logic/Rule | FEAT-27.SPEC-009 |
| Pro changes a notification preference | Write to notification_preferences; no message is sent by this feature -- FEAT-08 reads the new preference on its next send | Inline in triggering screen | FEAT-27.SPEC-005 |
| Pro taps "preview my booking page" | Navigate into FEAT-05's existing preview mode and emit booking_page_previewed | Inline in triggering screen, cross-feature destination | FEAT-27.SPEC-001 |
| Pro submits a help request | Create the request (something FEAT-19's support lookup can reference) and acknowledge receipt to the Pro | Inline create + Standalone Notification | FEAT-27.SPEC-006 / FEAT-27.SPEC-013 |
| A save of any setting fails | Keep the entered values on screen with a retry action; nothing is silently lost | Inline in triggering screen | FEAT-27.SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 / SPEC-005 / SPEC-006 |
| Pro opens any settings screen while offline | Show current settings read-only; block changes until connectivity returns (ASMP-27) | Inline in triggering screen | FEAT-27.SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 / SPEC-005 |

## Shared Context

**Discrepancy flagged, not resolved:** The dependency map slice's Shared Data Entities section lists only "Pro Account" for this feature, but this feature's own Data Notes field names "help requests" among the data it captures. This Brief models Help Request as a second, lightweight entity this feature creates (Entity-Lifecycle Coverage Matrix above) because Stage 2 explicitly assigns the capture to this feature and FEAT-19 needs something to reference; the Requirements Architect should confirm whether Help Request belongs in the Feature Dependency Map's Shared Data Entities table.

**Shared Entities:**
- Pro Account -- read by every screen in this feature (SPEC-001 through SPEC-005), updated by the same five screens on their respective fields, and read externally by FEAT-05 (profile, pause state), FEAT-08 (studio address, notification preferences), FEAT-12, FEAT-01/02/03/07 (timezone/currency), and FEAT-28 (country/currency match). Fields touched here: display_name, photo, intro, general_area, studio_address, booking_link_name, timezone, currency, status (pause sub-state), notification_preferences.
- Help Request -- created by SPEC-006, acknowledged by SPEC-013, read externally by FEAT-19's support view.

**Shared UI Patterns:**
- Settings hub navigation -- SPEC-001 is the entry point that lists and links to SPEC-002 (link), SPEC-003 (timezone/currency), SPEC-004 (pause), SPEC-005 (notifications) and SPEC-006 (help), and carries the preview action; every sub-screen returns to this hub, so Spec Writers should describe entry/exit consistently.
- Save-failure recovery -- every editing screen in this feature (SPEC-001 through SPEC-006) keeps the Pro's entered values on screen and offers retry rather than clearing the form, per the States field's Error expectation; Spec Writers should describe this identically across screens rather than re-deriving it per screen.
- Offline read-only posture -- every screen in this feature shows the Pro's last-loaded settings read-only when offline and disables save controls until connectivity returns (ASMP-27, States field's Offline-degraded expectation).

**Shared Validation:**
- FEAT-27.SPEC-007 owns booking link name validity; SPEC-002 defers to it rather than duplicating the format/uniqueness rules.
- FEAT-27.SPEC-008 owns whether currency is still editable; SPEC-003 defers to it before allowing a currency change.
- FEAT-27.SPEC-009 owns pause precedence; SPEC-004 defers to it to decide what the resume toggle can and cannot do at any given moment.

## Internal Dependency Map

```
SPEC-001 (Profile & Booking Page Settings) -> [Pro taps "change my link"] -> SPEC-002 (Booking Link Rename)
SPEC-001 (Profile & Booking Page Settings) -> [Pro taps "timezone & currency"] -> SPEC-003 (Timezone & Currency Settings)
SPEC-001 (Profile & Booking Page Settings) -> [Pro taps "pause bookings"] -> SPEC-004 (Pause Bookings)
SPEC-001 (Profile & Booking Page Settings) -> [Pro taps "notifications"] -> SPEC-005 (Notification Preferences)
SPEC-001 (Profile & Booking Page Settings) -> [Pro taps "get help"] -> SPEC-006 (Help Request)
SPEC-001 (Profile & Booking Page Settings) -> [Pro taps "preview my page"] -> FEAT-05 (Public Booking Page & Booking Flow, preview mode)
SPEC-001 (Profile & Booking Page Settings) -> [Pro saves a photo] -> SPEC-012 (Profile Photo Storage Capability)
SPEC-002 (Booking Link Rename) -> [Pro saves a new name] -> SPEC-007 (Booking Link Name Validation & Uniqueness Rule) -> [accepted] -> SPEC-010 (Booking Link Forwarding & Reservation Expiry)
SPEC-007 (Booking Link Name Validation & Uniqueness Rule) -> [name already taken] -> SPEC-002 (shows suggestions)
SPEC-003 (Timezone & Currency Settings) -> [Pro attempts a currency change] -> SPEC-008 (Currency Lock Rule)
SPEC-004 (Pause Bookings) -> [Pro sets or clears a pause] -> SPEC-009 (Pause State Precedence Rule)
SPEC-009 (Pause State Precedence Rule) -> [end date reached] -> SPEC-011 (Automatic Pause Resume) -> SPEC-004 (reflects resumed state)
SPEC-006 (Help Request) -> [Pro submits] -> SPEC-013 (Help Request Acknowledgment)
SPEC-006 (Help Request) -> [request created] -> FEAT-19 (Platform Support Read-Only Access)
```

**Default Entry:** SPEC-001 (Profile & Booking Page Settings) -- the screen shown when the Pro opens settings (nav row "FEAT-12 navigation -> FEAT-27") and the screen shown inside the FEAT-15 wizard's profile step.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-27.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Renders inside the wizard's profile step and shows values already captured during sign-in; also re-evaluates go-live readiness (XBR-26) when display name and studio location are completed | Pro completes the wizard's profile step |
| FEAT-27.SPEC-001 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Settings is reached from the app's navigation | Pro opens settings |
| FEAT-27.SPEC-001 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Public profile fields and pause state feed what the booking page displays; the preview action opens FEAT-05's existing preview mode | Every profile save; Pro taps "preview" |
| FEAT-27.SPEC-001 | Outbound | FEAT-08 (Automated Booking Messaging) | Studio address feeds confirmation/reminder content | Every studio address save |
| FEAT-27.SPEC-012 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | The stored photo is served on the public booking page | Public booking page load |
| FEAT-27.SPEC-002 / SPEC-010 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | A closed, paused-to-closure, or mistyped link renders FEAT-05.SPEC-008's "this booking page isn't available" message; a renamed link's old name still resolves through FEAT-05 during its forwarding window | Client visits a booking link |
| FEAT-27.SPEC-008 | Inbound | FEAT-07 (Deposit Payment at Booking) | Reads whether the account's first Deposit Transaction has occurred to decide if currency is still editable; FEAT-07.SPEC-003 separately checks the resulting lock as a charge precondition | Pro attempts to change currency; every deposit charge |
| FEAT-27.SPEC-003 | Outbound | FEAT-02 (Availability & Working Hours Setup), FEAT-03 (Real-Time Slot Availability Engine), FEAT-08 (Automated Booking Messaging), FEAT-12 (Pro Daily Schedule Dashboard), FEAT-01 (Service & Pricing Management) | Timezone and currency, once saved, are read by every feature that computes, labels, or prices in the Pro's account settings (XBR-25) | Timezone or currency save |
| FEAT-27.SPEC-009 / SPEC-011 | Inbound | FEAT-18 (Pro Subscription & Billing Account Management) | FEAT-18.SPEC-004 sets and lifts a system-imposed pause on subscription lapse/recovery; this feature's precedence rule decides how that interacts with a Pro-chosen pause | Subscription lapses or is restored |
| FEAT-27.SPEC-004 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Pause state and message are enforced by FEAT-05.SPEC-008 (Booking Page Availability Gate) | Pro turns a pause on or off, or it auto-resumes |
| FEAT-27.SPEC-005 | Outbound | FEAT-08 (Automated Booking Messaging) | FEAT-08.SPEC-005 and SPEC-006 read notification_preferences to route each Pro notification | Every Pro-facing notification FEAT-08 sends |
| FEAT-27.SPEC-013 | Outbound | FEAT-08 (Automated Booking Messaging) | If the acknowledgment is delivered by text or email rather than shown in-app, it travels over FEAT-08.SPEC-012 / SPEC-013's capability | Help request submitted |
| FEAT-27.SPEC-006 / SPEC-013 | Outbound | FEAT-19 (Platform Support Read-Only Access) | The created help request is what support opens the read-only account view against; FEAT-19's support_view_opened signal carries this request's reference | Pro submits a help request |
| FEAT-27 (account-wide) | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Anyone not signed in as the Pro is sent to sign-in before reaching any screen in this feature (XBR-29) | Any attempt to open settings |
| FEAT-27.SPEC-003 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | FEAT-28.SPEC-004 checks that the payout account's country and currency match this feature's saved currency (SC-20) | Timezone or currency save; payout connection |

## Non-Functional Notes

**Data volumes / growth:** One Pro Account record per pro, updated in place -- no growth pattern of its own. Help Request records grow slowly and only on demand (one per help-seeking moment, an Edge/Recovery-coverage journey step), never at booking volume.

**Responsiveness:** This is "a small settings screen that renders at once" (States field) -- no loading state applies to any screen in this feature; every screen shows already-cached account data immediately on open. A save completes without a perceptible wait, and the change is visible on the public booking page immediately (Primary Flows: "the public booking page shows the change immediately").

**Data sensitivity / privacy:** The Pro's display name, photo, intro and general area are intentionally public; the full studio address is personal data that may reveal a home address and is disclosed only inside a booked client's own confirmation and reminder, never on the public page or to anyone else (Pro Account entity's Data Sensitivity note; ASMP-23). Support's access to this feature's data is view-only and status-level -- never sign-in codes, never private client notes (ASMP-20, XBR-24) -- and every support view of a Pro's account is logged in that Pro's visible account activity. Every screen in this feature must remain fully usable at phone width, with a screen reader, and without relying on color alone (ASMP-28).

**Compliance flags:** N/A -- no named compliance regime (health, financial, or otherwise) applies to profile, link, timezone, currency, pause, notification-preference, or help-request data; the applicable privacy posture is covered above under Data sensitivity / privacy (ASMP-23).

## Non-Goals

- **Support acting on the Pro's behalf** -- Excluded per scope-boundaries.md (SC-05): support's help-request access, surfaced through FEAT-19, is read-only troubleshooting; support can never edit any setting in this feature, sign in as the Pro, or change sign-in details on the Pro's behalf.
- **Any additional role editing these settings** -- Excluded per scope-boundaries.md (SC-02): the Access field and Access Matrix give Full access to the Pro alone; no manager, staff, or admin role exists to share or delegate settings access.
- **Notifications for ordinary settings changes** -- Explicit per this feature's Communications field ("changes to settings send no client messages"); only a help request produces an acknowledgment (FEAT-27.SPEC-013). A profile, link, timezone, currency, pause, or notification-preference change is silent to the client and to the Pro beyond the in-screen save confirmation.
- **Retention or purge policy for help requests** -- Intentional lifecycle decision surfaced by the CRUD matrix: Stage 2 states no retention window for help requests, so they are kept indefinitely as part of the Pro's visible account activity (XBR-24), consistent with how this product treats other durable account history; there is no delete/archive path to design.
- **Support editing, deleting, or setting a status on a help request** -- Excluded per scope-boundaries.md (SC-05) and coordination note 8: the support-side lookup, any ticket status, and the read-only timeline view belong entirely to FEAT-19 (same batch); this feature's ownership stops at creating the request and acknowledging it.



# Screen Spec: Profile & Booking Page Settings

## Overview

**Name:** Profile & Booking Page Settings
**ID:** FEAT-27.SPEC-001
**Type:** Screen
**Purpose:** Talia edits her public profile (display name, photo, intro, general area) and her confirmation-only studio address, and reaches every other settings sub-flow (booking link, timezone/currency, pause, notifications, help) and the booking-page preview from one hub.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Displaying and editing display_name, photo, intro, general_area, and studio_address on the Pro Account
- The settings hub's navigation into every sub-flow: booking link rename (FEAT-27.SPEC-002), timezone/currency (FEAT-27.SPEC-003), pause bookings (FEAT-27.SPEC-004), notification preferences (FEAT-27.SPEC-005), and help request (FEAT-27.SPEC-006)
- The "preview my booking page" action into FEAT-05's preview mode
- Rendering inside the FEAT-15 onboarding wizard's profile step, using the same fields and save behavior as the standalone settings entry

**Non-Goals:**
- Booking link renaming, timezone/currency, pause, notification preferences, and help request -- each owned by its own sibling spec (FEAT-27.SPEC-002 through FEAT-27.SPEC-006); this screen only links to them
- Storing or serving the uploaded photo file -- owned by FEAT-27.SPEC-012 (Profile Photo Storage Capability); this screen only triggers the upload and displays the result
- The client-facing rendering of these fields on the public booking page -- owned by FEAT-05 (Public Booking Page & Booking Flow); this screen only writes the values FEAT-05 reads
- Any additional role editing these settings -- excluded per scope-boundaries.md SC-02: the Access Matrix gives Full access to the Pro alone; no manager, staff, or admin role exists to share or delegate settings access

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard, navigation) | Talia opens settings from the app's navigation | None -- screen loads the current Pro Account values |
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Talia completes the sign-in step and the wizard hands off to the profile step | Wizard-embedded mode flag; on save, control returns to the wizard shell rather than closing to FEAT-12 |
| FEAT-27 sub-screens (SPEC-002 through SPEC-006) | Talia taps back from any sub-screen | None -- returns to this hub with current values |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Edit and save display_name, photo, intro, general_area, studio_address; navigate to every sub-flow; preview the booking page | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen; they see the resulting public profile fields on the booking page (FEAT-05) and the studio address only inside their own booking confirmation, never this settings screen |
| Platform Operator (Support) | Full screen, read-only (all fields visible, including studio_address, per ASMP-20 status-level troubleshooting access) | No actions -- every edit control is rendered disabled | Every edit control (photo upload, text fields, sub-flow navigation buttons that would change data) is shown disabled with the label "View-only in support mode"; the "preview my booking page" action remains available since it changes nothing on the Pro Account |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered but unsaved field edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Profile & Booking Page" with a back arrow (returns to FEAT-12, or hands control back to the FEAT-15 wizard shell when embedded) and a "Save" action button (right-aligned, enabled only while there are unsaved changes).

**Body, top section -- Public profile:**
- Photo (tap to upload or replace; shows a gentle prompt "Add a photo" when empty, per the Empty state)
- Display Name (text input, required, 1-60 characters)
- Intro (multi-line text input, optional, up to 300 characters, with a live remaining-character count)
- General Area (text input, the public area shown on the booking page -- for example, a neighborhood or city, never the full address)

**Body, middle section -- Studio location (confirmation-only):**
- Studio Address (text input, required before go-live per XBR-26) with a supporting line: "Shown only in a booked client's own confirmation and reminder -- never on your public booking page."

**Body, lower section -- More settings (navigation list):**
- "Booking link" row -- shows the current booking_link_name, navigates to FEAT-27.SPEC-002
- "Timezone & currency" row -- shows the current timezone and currency, navigates to FEAT-27.SPEC-003
- "Pause bookings" row -- shows current pause state ("Taking bookings" or "Paused"), navigates to FEAT-27.SPEC-004
- "Notifications" row -- navigates to FEAT-27.SPEC-005
- "Preview my booking page" row -- navigates to FEAT-05's preview mode
- "Get help" row -- navigates to FEAT-27.SPEC-006

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width, sections stacked in the order above.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; the navigation list rows remain full-width within that cap.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (standalone entry) or hand control back to FEAT-15.SPEC-001 (wizard-embedded entry) | Screen closes | Standard transition |
| Photo | Tap | Opens the device's photo picker, then triggers FEAT-27.SPEC-012 (Profile Photo Storage Capability) upload | Photo shows an uploading state, then the new photo | Success: photo updates in place. Failure: photo reverts to its previous state with an inline message from FEAT-27.SPEC-012 |
| Display Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Display Name input | Blur | Validates required, 1-60 characters | Error state if invalid | "Display name is required" or "Display name must be 60 characters or fewer" |
| Intro input | Type | Captures text input, updates remaining-character count | Field and counter update | Counter shows characters remaining |
| General Area input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Studio Address input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Studio Address input | Blur | Validates required (before go-live) | Error state if empty and account is not yet live | "A studio location is required before your booking link can go live" |
| "Booking link" row | Tap | Navigate to FEAT-27.SPEC-002 (Booking Link Rename) | Screen closes | Standard transition |
| "Timezone & currency" row | Tap | Navigate to FEAT-27.SPEC-003 (Timezone & Currency Settings) | Screen closes | Standard transition |
| "Pause bookings" row | Tap | Navigate to FEAT-27.SPEC-004 (Pause Bookings) | Screen closes | Standard transition |
| "Notifications" row | Tap | Navigate to FEAT-27.SPEC-005 (Notification Preferences) | Screen closes | Standard transition |
| "Preview my booking page" row | Tap | Navigate to FEAT-05's preview mode; emits booking_page_previewed | Screen closes | FEAT-05's preview screen opens showing the identical client-facing screen with a preview banner |
| "Get help" row | Tap | Navigate to FEAT-27.SPEC-006 (Help Request) | Screen closes | Standard transition |
| Save button | Tap | Validates display_name and studio_address; if valid, writes display_name, photo, intro, general_area, studio_address to the Pro Account | Button shows loading state during save | Success: toast "Settings saved" and public booking page reflects the change immediately. Failure: inline error banner with retry, entered values preserved |

### Accessibility Notes

- **Focus order:** Back arrow -> Photo -> Display Name -> Intro -> General Area -> Studio Address -> Booking link row -> Timezone & currency row -> Pause bookings row -> Notifications row -> Preview row -> Get help row -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** "Settings saved" is announced on success; on validation failure, focus moves to the first field in error.
- **Photo upload announcements:** The uploading state and its outcome (success or failure message) are announced to assistive technology.
- **Keyboard alternatives:** Every action, including the photo picker trigger, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first visit) | Fields show the values captured during onboarding (FEAT-15); photo shows a gentle prompt "Add a photo" if none was set | First time the Pro opens this screen after sign-in | Talia edits any field |
| Filled | Fields show current saved values | Screen loads with existing data | Talia begins editing |
| Editing | Fields show in-progress input; Save enabled | Talia types in any field or uploads a photo | Talia taps Save or navigates away |
| Saving | Save button shows loading state, fields remain visible but read-only | Talia taps Save with valid data | Save completes or fails |
| Error | Inline error banner at the top of the form with a retry action; entered values preserved | Save fails | Talia taps Retry or corrects the field causing failure and retries |
| Support view (read-only) | Full screen with every edit control disabled and labeled "View-only in support mode" | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- showing your last saved settings." at the top; fields remain viewable but disabled for editing | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- fields re-enable |

## Validation Rules

**Option B -- Inline (simple validations not warranting a standalone Logic/Rule spec for this screen's own fields):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| display_name | Required, 1-60 characters | On blur and on submit | "Display name is required" / "Display name must be 60 characters or fewer" |
| intro | Optional, up to 300 characters | On change (counter) and on submit | "Intro must be 300 characters or fewer" |
| general_area | Optional, no format restriction beyond data type | On submit | -- |
| studio_address | Required before the booking link can go live (XBR-26); otherwise optional at this stage | On submit | "A studio location is required before your booking link can go live" |

Booking link name, timezone, currency, pause, and notification-preference validation are each governed by their own sibling specs (FEAT-27.SPEC-007, FEAT-27.SPEC-008, FEAT-27.SPEC-009) and are not duplicated here.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (standalone entry) | Pro Daily Schedule Dashboard | FEAT-12 |
| Back arrow tap (wizard-embedded entry) | Setup Wizard Shell | FEAT-15 (FEAT-15.SPEC-001) |
| "Booking link" row tap | FEAT-27.SPEC-002 (Booking Link Rename) | -- |
| "Timezone & currency" row tap | FEAT-27.SPEC-003 (Timezone & Currency Settings) | -- |
| "Pause bookings" row tap | FEAT-27.SPEC-004 (Pause Bookings) | -- |
| "Notifications" row tap | FEAT-27.SPEC-005 (Notification Preferences) | -- |
| "Preview my booking page" row tap | Booking page preview mode | FEAT-05 (FEAT-05.SPEC-001) |
| "Get help" row tap | FEAT-27.SPEC-006 (Help Request) | -- |
| Successful save (wizard-embedded) | Setup Wizard Shell, advances to the next step | FEAT-15 (FEAT-15.SPEC-001), and re-evaluates go-live readiness (XBR-26) via FEAT-15.SPEC-007 |

## Data Model

**Creates:** None -- the Pro Account record already exists, created by FEAT-15.SPEC-004 on first sign-in.
**Reads:** Pro Account -- display_name, photo, intro, general_area, studio_address (this screen's fields); booking_link_name, timezone, currency, status (pause sub-state) (summary values shown in the navigation-list rows only, not editable here).
**Updates:** Pro Account -- display_name, photo, intro, general_area, studio_address.
**Deletes:** None.

## Business Rules

- Public profile fields (display_name, photo, intro, general_area) are shown on the public booking page (FEAT-05) immediately on save, per the Feature Breakdown Brief's Non-Functional Notes.
- studio_address is disclosed only inside a booked client's own confirmation and reminder (FEAT-08), never on the public page or to anyone else, per ASMP-23 and the Pro Account entity's Data Sensitivity note.
- XBR-26: display_name and studio_address being set feeds the go-live readiness evaluation (FEAT-15.SPEC-007); this screen re-evaluates readiness on every successful save while the account has not yet gone live.
- Save-failure recovery is identical across every editing screen in this feature (FEAT-27.SPEC-001 through SPEC-006): entered values are always kept on screen with a retry action, per the Feature Breakdown Brief's Shared Context.
- Offline read-only posture is identical across every screen in this feature (ASMP-27): current settings remain viewable, changes require connectivity.

## Edge Cases

- **Talia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save your settings. Check your connection and try again." with a Retry button. Form data preserved.
- **Talia uploads a photo that fails size or format limits** -- FEAT-27.SPEC-012 returns the exact rejection reason inline below the photo; the previous photo (or empty state) remains unaffected.
- **Talia's Pro Account was updated on another signed-in device while this screen was open (for example, she also renamed her booking link from her other phone in the same moment)** -- Save is not rejected: per the dependency map's Contention note for the Pro Account, ordinary profile fields (display_name, photo, intro, general_area, studio_address) resolve last-write-wins, so this screen's save simply overwrites those fields with Talia's latest edits here; it never touches booking_link_name, timezone, currency, or pause state, which are owned by their own sibling screens and cannot be stale-overwritten from this screen.
- **Talia opens this screen for the first time with no photo set** -- The Empty state's gentle "Add a photo" prompt is shown; saving with no photo is allowed (photo is optional, per Validation & Limits).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Booking Link Rename) | Navigation (outbound) | "Booking link" row navigates here |
| FEAT-27.SPEC-003 (Timezone & Currency Settings) | Navigation (outbound) | "Timezone & currency" row navigates here |
| FEAT-27.SPEC-004 (Pause Bookings) | Navigation (outbound) | "Pause bookings" row navigates here |
| FEAT-27.SPEC-005 (Notification Preferences) | Navigation (outbound) | "Notifications" row navigates here |
| FEAT-27.SPEC-006 (Help Request) | Navigation (outbound) | "Get help" row navigates here |
| FEAT-27.SPEC-012 (Profile Photo Storage Capability) | Triggers (outbound) | Photo upload/replace goes through this integration |
| FEAT-05.SPEC-001 (Public Booking Page & Booking Flow, landing/service list) | Navigation (outbound) | "Preview my booking page" opens FEAT-05's preview mode; FEAT-05.SPEC-001 lists this spec as its preview entry point |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound and outbound) | Entry from app navigation; back arrow returns there |
| FEAT-15.SPEC-001 (Setup Wizard Shell & Step Navigation) | Navigation (inbound and outbound) | Renders inside the wizard's profile step; successful save hands control back to the shell |
| FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) | References (outbound) | display_name and studio_address completion feeds this rule's readiness evaluation |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view during a help request |
| FEAT-08 (Automated Booking Messaging) | References (outbound) | studio_address is read by FEAT-08 to compose confirmation and reminder content |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| profile_updated | fields changed (display_name / photo / intro / general_area / studio_address), viewer role (always pro -- support cannot save) | A save on this screen completes successfully | N/A -- success-metrics.md's 20 metrics cover the 14 Core and 2 Important features it names by Connected Feature, and none is connected to Pro Profile & Booking Page Settings; retained so profile edit activity is observable |
| booking_page_previewed | entry source (settings hub) | Talia taps "preview my booking page" | N/A -- no success-metrics.md metric is connected to this feature; retained so preview usage is observable |

## Acceptance Criteria

**FEAT-27.SPEC-001-AC-01:** Given Talia is on the Profile & Booking Page Settings screen for the first time after sign-in, when the screen loads, then it shows the values captured during onboarding and a gentle "Add a photo" prompt if no photo was set.

**FEAT-27.SPEC-001-AC-02:** Given Talia enters "Talia Reyes Lashes" as her display name and taps Save, when the save completes, then she sees "Settings saved" and the public booking page (FEAT-05) reflects the new name immediately.

**FEAT-27.SPEC-001-AC-03:** Given Talia clears her display name and taps Save, when validation runs, then the display name field shows "Display name is required" and the save does not proceed.

**FEAT-27.SPEC-001-AC-04:** Given Talia uploads a new photo, when FEAT-27.SPEC-012 confirms storage, then her photo updates in place on this screen and on the public booking page.

**FEAT-27.SPEC-001-AC-05:** Given Talia uploads a photo that exceeds the size limit, when FEAT-27.SPEC-012 rejects it, then she sees the exact rejection reason inline and her previous photo is unaffected.

**FEAT-27.SPEC-001-AC-06:** Given Talia taps the "Booking link" row, when the tap registers, then she is navigated to FEAT-27.SPEC-002 (Booking Link Rename).

**FEAT-27.SPEC-001-AC-07:** Given Talia taps "Preview my booking page", when the tap registers, then FEAT-05's preview mode opens showing the identical client-facing screen with a preview banner, and booking_page_previewed is emitted.

**FEAT-27.SPEC-001-AC-08:** Given Talia's Pro Account has never had a studio address set, when she leaves studio_address empty and taps Save, then she sees "A studio location is required before your booking link can go live" and go-live readiness (FEAT-15.SPEC-007) is not satisfied.

**FEAT-27.SPEC-001-AC-09:** Given Talia has unsaved changes, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-27.SPEC-001-AC-10:** Given a support operator opens this screen via FEAT-19, when they view it, then every field and edit control is visible but disabled and labeled "View-only in support mode", and the "preview my booking page" action remains available.

**FEAT-27.SPEC-001-AC-11:** Given Talia opens this screen with no connectivity, when the screen loads, then she sees her last saved settings read-only with the offline banner and cannot edit until connectivity returns.

**FEAT-27.SPEC-001-AC-12:** Given Talia is completing the FEAT-15 onboarding wizard's profile step, when she saves this screen's fields, then control returns to the Setup Wizard Shell (FEAT-15.SPEC-001) and it advances to the next step.

**FEAT-27.SPEC-001-AC-13:** Given Talia's save fails due to a network error, when the failure is returned, then she sees "Could not save your settings. Check your connection and try again." with a Retry button, and her entered values remain on screen.

**FEAT-27.SPEC-001-AC-14:** Given Talia renamed her booking link from a second signed-in device moments before saving this screen, when this screen's save completes, then only display_name, photo, intro, general_area, and studio_address are written -- booking_link_name is untouched and reflects the rename from the other device.

**FEAT-27.SPEC-001-AC-15:** Given Talia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 7 (empty, filled, editing, saving, error, support view, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Booking Link Rename

## Overview

**Name:** Booking Link Rename
**ID:** FEAT-27.SPEC-002
**Type:** Screen
**Purpose:** Talia views and changes her booking link name, sees the forwarding guarantee on the old name before she confirms, and gets suggestions when a name is already taken.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Displaying the current booking_link_name as a full shareable link
- Entering a new booking_link_name and previewing it before saving
- Surfacing format, length, and uniqueness feedback from FEAT-27.SPEC-007 inline
- Explaining the forwarding guarantee (old name keeps working) before the Pro confirms a rename

**Non-Goals:**
- The format, length, uniqueness, and reject-with-refresh rules themselves -- owned by FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule); this screen only surfaces its outcomes
- Setting up the forwarding and reservation-release mechanism -- owned by FEAT-27.SPEC-010 (Booking Link Forwarding & Reservation Expiry); this screen only informs the Pro that it will happen
- Choosing the booking link name for the first time during onboarding -- that happens on this same screen when reached from FEAT-15's profile step (Entry Points), and is not a distinct flow requiring its own spec

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Booking link" row | Current booking_link_name |
| FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule) | A submitted name is rejected because another pro claimed it in the meantime | Refreshed availability state and suggested alternatives |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter and save a new booking_link_name | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen; a client who visits a renamed link is forwarded transparently by FEAT-05, per XBR-27 |
| Platform Operator (Support) | Full screen, read-only (current and previous names, forwarding window remaining) | No actions -- the input field and Save are shown disabled | Input field and Save button are disabled with the label "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any entered but unsaved name is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Booking Link" with a back arrow (returns to FEAT-27.SPEC-001).

**Body, top section -- Current link:** The full shareable link displayed as read-only text (for example, "chairtime.app/talia-lashes") with a "Copy link" action.

**Body, middle section -- Rename:**
- New link name input (text input, prefixed with the fixed domain portion so only the link-name segment is editable), with helper text "3-40 letters, numbers, or hyphens"
- Live availability indicator below the input (checking / available / taken)

**Body, lower section -- Forwarding guarantee:** A fixed, always-visible explanatory line: "If you rename your link, your old one keeps working and forwards here for at least {forwarding_window} -- so an old Instagram bio link never breaks." ({forwarding_window} renders the duration held by platform parameter: `booking-link-forward-window-months`, expressed in months.)

**Footer:** "Save new name" action button, enabled only when the entered name passes format/length checks and availability shows available.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| "Copy link" action | Tap | Copies the current full link to the clipboard | None | Confirmation toast "Link copied" |
| New link name input | Type | Captures text input; triggers FEAT-27.SPEC-007's format/length check on change, and an availability check on pause in typing | Availability indicator updates to "Checking..." then "Available" or "Taken" | Live indicator below the field |
| New link name input | Blur | Runs FEAT-27.SPEC-007's format/length validation | Error state if invalid | Exact error message from FEAT-27.SPEC-007 |
| "Save new name" button | Tap | Submits the new name to FEAT-27.SPEC-007 for final validation; on acceptance, writes booking_link_name and triggers FEAT-27.SPEC-010 (forwarding setup) | Button shows loading state during save | Success: dialog "Your link is now {new-name}. Your old link keeps forwarding for at least {forwarding_window}." (per platform parameter: `booking-link-forward-window-months`) with a "Done" action returning to FEAT-27.SPEC-001. Failure (taken by another pro since the check): the field shows "Taken" with suggested alternatives, per FEAT-27.SPEC-007 |
| Suggested alternative chip | Tap | Fills the new link name input with the suggested value and re-runs the availability check | Input and indicator update | Indicator shows "Available" for the suggestion (suggestions are generated as already-available) |

### Accessibility Notes

- **Focus order:** Back arrow -> Current link -> Copy link action -> New link name input -> Suggested alternative chips (when shown) -> Save new name.
- **Availability announcements:** The availability indicator's state change ("Checking...", "Available", "Taken") is announced to assistive technology as it updates.
- **Save feedback:** The success dialog's full text is announced on save; on rejection, focus moves to the input field and its "Taken" state and suggestions are announced.
- **Keyboard alternatives:** Every action, including selecting a suggested alternative, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Viewing (default) | Current link shown, input empty, Save disabled | Screen opens | Talia begins typing a new name |
| Checking | Availability indicator shows "Checking..." | Talia pauses typing after entering a candidate name | The availability check returns |
| Available | Indicator shows "Available"; Save enabled | The candidate name passes format, length, and uniqueness checks | Talia edits the field further (returns to Checking) or taps Save |
| Taken | Indicator shows "Taken" with up to 3 suggested alternatives; Save disabled | The candidate name fails the uniqueness check | Talia edits the field, selects a suggestion, or navigates away |
| Saving | Save button shows loading state | Talia taps Save on an Available name | Save completes or is rejected |
| Rejected (claimed since check) | Field reverts to Taken state with fresh suggestions, per FEAT-27.SPEC-007's reject-with-refresh behavior | Another pro claims the name between Talia's check and her save | Talia selects a new suggestion or types another name |
| Support view (read-only) | Current link and history shown; input and Save disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- link renaming needs a connection." above the current-link section; the current link remains viewable, the rename input is disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- input re-enables |

## Validation Rules

Validation governed by FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule). See that spec for the format, length, uniqueness, and reject-with-refresh rules. This screen applies FEAT-27.SPEC-007's checks on field change (format/length) and on submit (uniqueness, final acceptance).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful save, "Done" tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- booking_link_name (current value, displayed as the full link).
**Updates:** Pro Account -- booking_link_name (on successful save, via FEAT-27.SPEC-007's acceptance and FEAT-27.SPEC-010's forwarding setup).
**Deletes:** None.

## Business Rules

- Every rename is validated by FEAT-27.SPEC-007 before it is accepted -- Talia cannot save a name that fails format, length, or uniqueness checks.
- A successful rename always triggers FEAT-27.SPEC-010, which begins forwarding the old name and reserves it from reuse for at least platform parameter: `booking-link-forward-window-months` -- this screen never saves a rename without that follow-on automation firing.
- XBR-27: the old name keeps forwarding transparently and never resolves to another pro's page during its forwarding window.
- This screen never exposes a way to reuse a name still inside another pro's reservation window -- FEAT-27.SPEC-007's uniqueness check covers reserved names, not just currently active ones.

## Edge Cases

- **Talia's candidate name is claimed by another pro between her availability check and her Save tap** -- The save is rejected and the screen refreshes to show "Taken" with fresh suggestions, per FEAT-27.SPEC-007's reject-with-refresh resolution for the Pro Account's booking_link_name contention.
- **Talia enters a name identical to her current one** -- Save is disabled with a note "This is already your current link name" -- no rename action is offered for a no-op change.
- **Talia leaves this screen mid-check without saving** -- The candidate name is discarded; her current booking_link_name is unaffected.
- **Network failure during save** -- Error banner: "Could not save your new link. Check your connection and try again." with a Retry button; her current link name remains unchanged and the candidate name is preserved in the field.
- **Talia enters a name that was previously hers (renamed away from, now inside her own forwarding window)** -- FEAT-27.SPEC-007 treats it as available to reclaim, since it is reserved against other pros only, not against the pro who owns the forwarding record; the availability indicator shows "Available."
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound and outbound) | Entry from the "Booking link" row; back arrow and success both return there |
| FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule) | References (inbound) | Format, length, uniqueness, and reject-with-refresh rules applied to the input |
| FEAT-27.SPEC-010 (Booking Link Forwarding & Reservation Expiry) | Triggers (outbound) | A successful save fires this automation |
| FEAT-05.SPEC-001 / FEAT-05.SPEC-008 (Public Booking Page & Booking Flow) | References (outbound) | The renamed link and its forwarding old name are resolved by these specs when a client visits either |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| booking_link_renamed | had_prior_forward (yes/no -- whether the old name was itself a forwarded name) | A rename save completes successfully | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so rename activity is observable |
| booking_link_rename_rejected | reason (format / length / taken) | A rename attempt is rejected by FEAT-27.SPEC-007 | N/A -- no connected success-metrics.md metric; retained so rejection frequency (a proxy for naming friction) is observable |

## Acceptance Criteria

**FEAT-27.SPEC-002-AC-01:** Given Talia is on the Booking Link Rename screen, when it loads, then she sees her current full link and a "Copy link" action.

**FEAT-27.SPEC-002-AC-02:** Given Talia types a candidate name that passes format and length checks and is unclaimed, when the availability check completes, then the indicator shows "Available" and Save becomes enabled.

**FEAT-27.SPEC-002-AC-03:** Given Talia types a candidate name already used by another pro, when the availability check completes, then the indicator shows "Taken" with up to 3 suggested alternatives and Save stays disabled.

**FEAT-27.SPEC-002-AC-04:** Given Talia taps a suggested alternative chip, when it is selected, then the input fills with that name and the indicator shows "Available."

**FEAT-27.SPEC-002-AC-05:** Given Talia has an available candidate name, when she taps "Save new name", then her booking_link_name updates and she sees "Your link is now {new-name}. Your old link keeps forwarding for at least {forwarding_window}." with {forwarding_window} rendering platform parameter: `booking-link-forward-window-months`.

**FEAT-27.SPEC-002-AC-06:** Given Talia's candidate name is claimed by another pro between her check and her Save tap, when the save is submitted, then it is rejected, the screen refreshes to "Taken" with fresh suggestions, and her prior booking_link_name is unchanged.

**FEAT-27.SPEC-002-AC-07:** Given Talia enters a name identical to her current booking_link_name, when she views the Save control, then it is disabled with the note "This is already your current link name."

**FEAT-27.SPEC-002-AC-08:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your new link. Check your connection and try again." and her candidate name remains in the field.

**FEAT-27.SPEC-002-AC-09:** Given a support operator opens this screen via FEAT-19, when they view it, then the input and Save are disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-002-AC-10:** Given Talia opens this screen with no connectivity, when the screen loads, then her current link is viewable and the rename input is disabled with "You're offline -- link renaming needs a connection."

**FEAT-27.SPEC-002-AC-11:** Given Talia previously renamed away from "talia-lashes" and it is still inside her own forwarding window, when she types "talia-lashes" as a new candidate, then the availability indicator shows "Available" (reclaimable by its own former owner).

**FEAT-27.SPEC-002-AC-12:** Given Talia taps "Save new name" twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (viewing, checking, available, taken, saving, rejected, support view, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Timezone & Currency Settings

## Overview

**Name:** Timezone & Currency Settings
**ID:** FEAT-27.SPEC-003
**Type:** Screen
**Purpose:** Talia sets her account timezone and currency, sees a warning before a timezone change is saved, and sees currency locked once it has been.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Displaying and changing the Pro Account's timezone
- Displaying the Pro Account's currency, editable only while FEAT-27.SPEC-008 says it is still unlocked
- Showing the pre-save warning about how existing bookings display after a timezone change
- Showing the exact locked explanation once currency can no longer change

**Non-Goals:**
- The currency-lock determination itself (whether the first deposit has been taken) -- owned by FEAT-27.SPEC-008 (Currency Lock Rule); this screen only reflects and defers to it
- Recomputing or re-displaying existing bookings in the new timezone -- owned by FEAT-12 (Pro Daily Schedule Dashboard) and every feature that reads timezone (FEAT-01, FEAT-02, FEAT-03, FEAT-08); this screen only warns before the change
- Choosing timezone or currency for the first time during onboarding -- captured by FEAT-15's own setup flow at account creation; this screen is reached afterward, for changes

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Timezone & currency" row | Current timezone and currency values |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Change timezone always; change currency only while unlocked (FEAT-27.SPEC-008) | If currency is locked, the currency control is shown disabled with the exact locked explanation (see Interactions) -- not a bare "not allowed" |
| The Client (Riley) | No | No | Clients have no entry point to this screen; they experience the resulting timezone-labeled times and currency-formatted prices on the booking page (FEAT-05) |
| Platform Operator (Support) | Full screen, read-only | No actions -- both controls shown disabled | Both controls disabled and labeled "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress timezone change (not yet confirmed past the warning) is discarded; a currency change already confirmed completes atomically before the session check applies |

## Layout and Content

**Header:** Screen title "Timezone & Currency" with a back arrow (returns to FEAT-27.SPEC-001).

**Body, top section -- Timezone:** A selection input showing the current timezone (for example, "America/Los_Angeles"), with helper text "Every booking time is shown in this timezone, labeled, so it's never ambiguous (XBR-25)."

**Body, lower section -- Currency:** A selection input showing the current currency (for example, "USD"). While unlocked, the control is editable with helper text "You can change this until your first deposit is taken." Once locked (per FEAT-27.SPEC-008), the control is disabled and the helper text is replaced with the exact locked explanation.

**Footer:** "Save" action button, enabled while there is an unsaved timezone or (while unlocked) currency change.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| Timezone selector | Select a new timezone | Captures the selection, does not save yet | Save button enables | Selector shows the newly chosen timezone |
| Currency selector (unlocked) | Select a new currency | Captures the selection, does not save yet | Save button enables | Selector shows the newly chosen currency |
| Currency selector (locked) | Tap | No action -- control is disabled | None | The locked explanation is already visible as static text; no further interaction occurs |
| Save button (timezone changed) | Tap | Opens a confirmation dialog stating the exact warning before committing | Dialog appears | Dialog text: "Changing your timezone won't move your existing bookings in time -- they'll keep their real moment and now display converted to the new timezone. Continue?" with "Continue" and "Keep current timezone" |
| Confirmation dialog -- "Continue" | Tap | Writes the new timezone (and any unlocked currency change) to the Pro Account | Dialog closes, screen shows saved values | Toast "Settings saved" |
| Confirmation dialog -- "Keep current timezone" | Tap | Discards the timezone change only; any unlocked currency change already selected is preserved for a subsequent save | Dialog closes, timezone selector reverts | Timezone selector shows the prior value |
| Save button (currency changed only, timezone unchanged) | Tap | Writes the new currency directly (no warning dialog -- the warning is specific to timezone's effect on existing bookings) | Button shows loading state | Success: toast "Settings saved". Failure: FEAT-27.SPEC-008's exact rejection if currency has since locked |

### Accessibility Notes

- **Focus order:** Back arrow -> Timezone selector -> Currency selector -> Save.
- **Warning dialog announcement:** The full warning text is announced to assistive technology when the dialog opens.
- **Locked-currency announcement:** The locked explanation is announced when the screen loads with currency already locked, and again if a save attempt is rejected because currency locked between load and save.
- **Save feedback:** "Settings saved" is announced on success; failures move focus to the relevant control with its exact message.
- **Keyboard alternatives:** Every action, including the confirmation dialog's two choices, is reachable and dismissible by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Viewing (default) | Current timezone and currency shown; Save disabled | Screen opens | Talia changes either selector |
| Editing | Selector(s) show new, unsaved value(s); Save enabled | Talia changes timezone and/or currency (while unlocked) | Talia taps Save or navigates away |
| Timezone-change warning | Confirmation dialog shown with the exact warning text | Talia taps Save with a changed timezone | Talia chooses "Continue" or "Keep current timezone" |
| Saving | Save button shows loading state | Talia confirms "Continue," or taps Save with only a currency change | Save completes or fails |
| Currency locked | Currency selector disabled; exact locked explanation shown in place of the "editable until first deposit" helper text | FEAT-27.SPEC-008 reports the account's first deposit has been taken | Never exits -- currency lock is permanent, per FEAT-27.SPEC-008 |
| Error | Inline error banner with retry; entered values preserved | Save fails (network failure, or currency locked since load) | Talia taps Retry, or acknowledges the lock and proceeds with the timezone-only portion of her change |
| Support view (read-only) | Both controls disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- these settings need a connection to change." above the form; values remain viewable, controls disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- controls re-enable |

## Validation Rules

Currency editability is governed by FEAT-27.SPEC-008 (Currency Lock Rule). See that spec for the exact locking condition and rejection message. Timezone has no restriction beyond selecting a valid supported timezone value; no additional field-level validation exists on this screen.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful save | Stays on this screen showing the newly saved values | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- timezone, currency (current values); Booking -- read by FEAT-27.SPEC-008 and by this screen's warning composition to describe how many upcoming bookings display converted after a timezone change.
**Updates:** Pro Account -- timezone (on any confirmed change); currency (on a change, only while FEAT-27.SPEC-008 reports it unlocked).
**Deletes:** None.

## Business Rules

- XBR-25: timezone and currency are per-account settings; every slot and appointment time is computed and shown in the Pro's timezone, labeled; currency is fixed once the first deposit is taken.
- A timezone change never moves an existing Booking's real moment in time -- it only changes the timezone used to display it, per the warning dialog's exact text and XBR-11 ("setup changes never silently cancel a confirmed booking").
- Currency editability is governed entirely by FEAT-27.SPEC-008 -- this screen never independently determines or overrides the lock.
- FEAT-28.SPEC-004 checks that the payout account's country and currency match this screen's saved currency (SC-20); a currency change here does not retroactively validate that match -- FEAT-28 surfaces any resulting mismatch on its own screen.

## Edge Cases

- **Talia changes both timezone and currency in the same visit, and currency locks between her edit and her Save tap** -- The timezone-change warning dialog still appears and, on "Continue," the timezone change saves; the currency change is rejected with FEAT-27.SPEC-008's exact locked message, and the currency selector reverts to its prior (locked) value while the timezone save proceeds independently.
- **Talia attempts to change currency after her first deposit was taken moments earlier (from a booking completed while this screen was open)** -- The currency selector is not live-updating; her save attempt is rejected with FEAT-27.SPEC-008's exact message and the screen refreshes to show currency now locked.
- **Talia selects the same timezone she already has** -- Save is not enabled for a no-op timezone selection; no warning dialog appears.
- **Network failure during save** -- Error banner: "Could not save your settings. Check your connection and try again." with a Retry button; entered values preserved.
- **Talia has many upcoming bookings and changes timezone** -- The warning dialog's text is fixed regardless of booking count; per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix, this screen reads Booking only to compose the warning, and no booking data itself is altered by the save.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound) | Entry from the "Timezone & currency" row; back arrow returns there |
| FEAT-27.SPEC-008 (Currency Lock Rule) | References (inbound) | Determines whether the currency selector is editable and supplies the exact locked message |
| FEAT-01, FEAT-02, FEAT-03, FEAT-08, FEAT-12 | References (outbound) | These features read the saved timezone and currency for computation, labeling, and pricing (XBR-25) |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (outbound) | Checks the saved currency against the payout account's country/currency match |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| timezone_updated | -- | A timezone change is confirmed and saved | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so timezone-change activity is observable |
| currency_updated | -- | A currency change is saved while unlocked | N/A -- no connected success-metrics.md metric; retained so currency-change activity (which stops entirely once locked) is observable |
| currency_change_blocked | -- | A currency change is rejected because the account is locked | N/A -- no connected success-metrics.md metric; retained so lock-related friction is observable |

## Acceptance Criteria

**FEAT-27.SPEC-003-AC-01:** Given Talia is on the Timezone & Currency Settings screen, when it loads, then it shows her current timezone and currency.

**FEAT-27.SPEC-003-AC-02:** Given Talia selects a new timezone and taps Save, when the confirmation dialog appears, then it reads "Changing your timezone won't move your existing bookings in time -- they'll keep their real moment and now display converted to the new timezone. Continue?"

**FEAT-27.SPEC-003-AC-03:** Given Talia sees the timezone-change warning dialog, when she taps "Continue", then her timezone updates and she sees "Settings saved."

**FEAT-27.SPEC-003-AC-04:** Given Talia sees the timezone-change warning dialog, when she taps "Keep current timezone", then the dialog closes and her timezone selector reverts to its prior value.

**FEAT-27.SPEC-003-AC-05:** Given Talia's account has not yet taken a first deposit, when she views the currency selector, then it is editable with the helper text "You can change this until your first deposit is taken."

**FEAT-27.SPEC-003-AC-06:** Given Talia's account has taken its first deposit, when she views the currency selector, then it is disabled and shows FEAT-27.SPEC-008's exact locked explanation.

**FEAT-27.SPEC-003-AC-07:** Given Talia's currency locks between her edit and her Save tap, when she attempts to save the currency change, then it is rejected with FEAT-27.SPEC-008's exact message and the selector reverts to the now-locked value; any accompanying timezone change still saves.

**FEAT-27.SPEC-003-AC-08:** Given Talia selects her currently-set timezone (no actual change), when she views the Save control, then no warning dialog is offered for that field since nothing changed.

**FEAT-27.SPEC-003-AC-09:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your settings. Check your connection and try again." and her selections remain on screen.

**FEAT-27.SPEC-003-AC-10:** Given a support operator opens this screen via FEAT-19, when they view it, then both selectors are disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-003-AC-11:** Given Talia opens this screen with no connectivity, when the screen loads, then her current values are viewable and both selectors are disabled with "You're offline -- these settings need a connection to change."

**FEAT-27.SPEC-003-AC-12:** Given Talia taps Save twice in rapid succession after confirming the timezone warning, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (viewing, editing, warning, saving, locked, error, support view, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Pause Bookings

## Overview

**Name:** Pause Bookings
**ID:** FEAT-27.SPEC-004
**Type:** Screen
**Purpose:** Talia pauses new bookings with an optional message and end date, or resumes them, and sees whether a system-imposed pause is also in effect.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Turning a Pro-chosen pause on (with an optional message and optional end date) and off
- Showing whether a system-imposed (subscription-lapse) pause is also active, and what that means for the resume toggle
- Deferring to FEAT-27.SPEC-009 for precedence between the two pause sources

**Non-Goals:**
- The precedence logic itself (what the resume toggle can and cannot do when both pause sources are present) -- owned by FEAT-27.SPEC-009 (Pause State Precedence Rule); this screen only reflects its outcome
- Automatically resuming bookings when a chosen end date arrives -- owned by FEAT-27.SPEC-011 (Automatic Pause Resume); this screen only sets the end date and reflects the resulting state
- Setting or lifting the system-imposed pause -- owned by FEAT-18 (Pro Subscription Billing & Account Management); this screen only displays that a system-imposed pause exists when FEAT-18.SPEC-004 has set one

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Pause bookings" row | Current pause state (Active / Paused, and which source(s)) |
| FEAT-27.SPEC-011 (Automatic Pause Resume) | The chosen end date is reached and bookings resume automatically | Refreshed state showing "Taking bookings" |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Turn a Pro-chosen pause on/off, edit its message and end date; the resume toggle's exact effect is governed by FEAT-27.SPEC-009 | If a system-imposed pause is also active, the "resume bookings" toggle is shown enabled but its exact behavior (per FEAT-27.SPEC-009) is stated inline: turning it on only clears the Pro-chosen pause, the account stays paused until billing is restored |
| The Client (Riley) | No | No | Clients have no entry point to this screen; they see the pause message (or nothing, if not paused) directly on the booking page (FEAT-05.SPEC-008), never this settings screen |
| Platform Operator (Support) | Full screen, read-only | No actions -- the pause toggle, message, and end-date fields shown disabled | Controls disabled and labeled "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress, unsaved pause message or end date is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Pause Bookings" with a back arrow (returns to FEAT-27.SPEC-001).

**Body, top section -- Status banner:** One of: "Taking bookings" (Active, no pause of either kind); "Paused by you" (Pro-chosen pause only); "Paused -- billing needs attention" (system-imposed pause present, per FEAT-18.SPEC-004); "Paused by you and by billing" (both present).

**Body, middle section -- Pro-chosen pause controls:**
- "Pause new bookings" toggle
- Message input (multi-line text, optional, shown when the toggle is on), helper text "For example, 'On holiday until 3 June'"
- End date input (optional date picker, shown when the toggle is on), helper text "Bookings resume automatically on this date. Leave blank to resume manually."

**Body, lower section -- System-imposed pause note (shown only when present):** Fixed text: "Your account is also paused because of a billing issue. Resolve it in Billing to fully resume bookings." with a link to FEAT-18's billing screen.

**Footer:** "Save" action button, enabled while there is an unsaved change to the toggle, message, or end date.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| "Pause new bookings" toggle (turning on) | Tap | Reveals the message and end-date inputs | Toggle shows on; inputs appear | Inputs animate into view |
| "Pause new bookings" toggle (turning off / "resume bookings") | Tap | Submits the resume action to FEAT-27.SPEC-009 for precedence evaluation | Toggle shows off if only a Pro-chosen pause was active; stays reflecting a remaining system-imposed pause otherwise | If only Pro-chosen: status banner updates to "Taking bookings." If a system-imposed pause remains: status banner updates to "Paused -- billing needs attention" and an inline note states "Your bookings stay paused until your billing issue is resolved." |
| Message input | Type | Captures text input | Field shows entered text | Standard input focus state |
| End date input | Select a date | Captures the date | Field shows the chosen date | Standard input display |
| Save button | Tap | Validates the end date (cannot be in the past, per FEAT-27.SPEC-009), then writes the pause toggle, message, and end date to the Pro Account | Button shows loading state during save | Success: toast "Settings saved" and the public booking page reflects the pause message immediately (enforced by FEAT-05.SPEC-008). Failure: inline error with retry, entered values preserved |
| "Resolve it in Billing" link | Tap | Navigate to FEAT-18's billing screen | Screen closes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Status banner (announced) -> Pause toggle -> Message input (when visible) -> End date input (when visible) -> System-imposed pause note and link (when visible) -> Save.
- **Status announcements:** The status banner's content is announced to assistive technology whenever it changes.
- **Resume-toggle clarification:** When a system-imposed pause is present, the exact wording "Your bookings stay paused until your billing issue is resolved" is announced immediately after the toggle is turned off, so the outcome is never ambiguous to a screen-reader user.
- **Keyboard alternatives:** Every action, including the date picker, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Taking bookings | Status banner "Taking bookings"; toggle off; message and end-date inputs hidden | No pause of either kind is active | Talia turns the toggle on, or FEAT-18.SPEC-004 sets a system-imposed pause |
| Paused by you | Status banner "Paused by you"; toggle on; message/end-date fields shown with saved values | Pro-chosen pause is active, no system-imposed pause | Talia turns the toggle off, the chosen end date is reached (FEAT-27.SPEC-011), or a system-imposed pause is added |
| Paused -- billing needs attention | Status banner "Paused -- billing needs attention"; toggle off; system-imposed note and billing link shown | Only a system-imposed pause is active | FEAT-18 restores billing and lifts the system-imposed pause |
| Paused by you and by billing | Status banner "Paused by you and by billing"; toggle on with fields shown; system-imposed note and billing link also shown | Both pause sources are active simultaneously | Either source clears (per FEAT-27.SPEC-009's precedence) |
| Saving | Save button shows loading state | Talia taps Save with valid data | Save completes or fails |
| Error | Inline error banner with retry; entered values preserved | Save fails, or the chosen end date is rejected as being in the past | Talia taps Retry or corrects the end date |
| Support view (read-only) | Status banner and all fields visible, all controls disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- pause changes need a connection." above the status banner; last-known state remains viewable, controls disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- controls re-enable |

## Validation Rules

**Option B -- Inline (validation not warranting duplication here, since the field belongs to this screen's own definition):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| end_date | Cannot be in the past | On selection and on submit | "The resume date can't be in the past. Choose today or a later date." |

Precedence between a Pro-chosen pause and a system-imposed pause (what the resume toggle can and cannot clear) is governed by FEAT-27.SPEC-009 (Pause State Precedence Rule). See that spec for the full rule.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| "Resolve it in Billing" link tap | Billing & Subscription Management Screen | FEAT-18 (FEAT-18.SPEC-002) |
| Successful save | Stays on this screen showing the newly saved pause state | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- status (pause sub-state: Active / Paused, and which source(s)), pause message, pause end date.
**Updates:** Pro Account -- status (Pro-chosen pause on/off, via FEAT-27.SPEC-009's precedence evaluation), pause message, pause end date.
**Deletes:** None.

## Business Rules

- FEAT-27.SPEC-009 governs precedence: the Pro's "resume bookings" toggle can only clear the Pro-chosen pause, never a system-imposed pause -- this screen states that outcome explicitly rather than implying a full resume.
- XBR-14: a paused account (either source) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged.
- The booking page's pause message display is enforced by FEAT-05.SPEC-008 (Booking Page Availability Gate), not by this screen directly -- this screen only writes the message that FEAT-05.SPEC-008 reads.
- A chosen end date triggers FEAT-27.SPEC-011 to resume bookings automatically on that date, unless a system-imposed pause is still active at that time (per FEAT-27.SPEC-009).

## Edge Cases

- **Talia sets an end date, then a system-imposed pause is added by FEAT-18 before that date arrives** -- FEAT-27.SPEC-011 still resumes the Pro-chosen pause on the chosen date, but the account stays paused overall because the system-imposed pause remains, per FEAT-27.SPEC-009; this screen's status banner updates to "Paused -- billing needs attention" on that date.
- **Talia tries to turn off the pause toggle while a system-imposed pause is also active** -- The toggle is accepted for the Pro-chosen pause only; the status banner immediately updates to "Paused -- billing needs attention" and the inline note confirms bookings stay paused until billing is resolved.
- **Talia selects an end date in the past** -- Inline error "The resume date can't be in the past. Choose today or a later date." shown on selection and blocking Save.
- **Network failure during save** -- Error banner: "Could not save your pause settings. Check your connection and try again." with a Retry button; entered values preserved.
- **Talia's chosen end date is reached while she has this screen open** -- The screen is not live-updating; if she then attempts a save, FEAT-27.SPEC-011's already-applied resume is reflected on reload before her save proceeds, avoiding a stale overwrite.
- **Talia turns the pause toggle on and off repeatedly without saving** -- No pause takes effect until Save is tapped; the booking page is unaffected until a save completes.
- **Talia sets different pause messages from two signed-in devices at effectively the same time** -- Resolution: last-write-wins, per the dependency map's Contention note for the Pro Account entity's ordinary fields; whichever save commits last is the message shown on the booking page, and neither device is shown an error (the system-imposed pause source, by contrast, is never subject to this last-write-wins path -- it is set only by FEAT-18, per FEAT-27.SPEC-009).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound) | Entry from the "Pause bookings" row; back arrow returns there |
| FEAT-27.SPEC-009 (Pause State Precedence Rule) | References (inbound) | Governs what the resume toggle can and cannot clear |
| FEAT-27.SPEC-011 (Automatic Pause Resume) | Triggers (outbound) / Triggered by (inbound) | A saved end date arms this automation; its firing refreshes this screen's state |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (outbound) | Enforces the pause state and message on the public booking page |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | References (inbound) | Sets the system-imposed pause this screen displays |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Navigation (outbound) | "Resolve it in Billing" link destination |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| bookings_paused | has_message (yes/no), has_end_date (yes/no) | Talia's save turns the Pro-chosen pause on | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so pause adoption is observable |
| bookings_resumed | trigger (manual / automatic) | The Pro-chosen pause clears, either by Talia's toggle or by FEAT-27.SPEC-011 | N/A -- no connected success-metrics.md metric; retained so resume activity is observable |

## Acceptance Criteria

**FEAT-27.SPEC-004-AC-01:** Given Talia's account is fully Active, when she opens this screen, then she sees "Taking bookings" and the pause toggle off.

**FEAT-27.SPEC-004-AC-02:** Given Talia turns the pause toggle on, enters "On holiday until 3 June" as her message and sets an end date, when she taps Save, then her booking page shows the pause message instead of available times immediately (FEAT-05.SPEC-008), and the status banner shows "Paused by you."

**FEAT-27.SPEC-004-AC-03:** Given Talia's account has only a Pro-chosen pause active, when she turns the toggle off and saves, then the status banner updates to "Taking bookings" and bookings resume.

**FEAT-27.SPEC-004-AC-04:** Given Talia's account has a system-imposed pause from a lapsed subscription and no Pro-chosen pause, when she opens this screen, then she sees "Paused -- billing needs attention" with a "Resolve it in Billing" link, and the pause toggle is off.

**FEAT-27.SPEC-004-AC-05:** Given Talia's account has both a Pro-chosen pause and a system-imposed pause, when she turns the toggle off and saves, then the Pro-chosen pause clears but the status banner still shows "Paused -- billing needs attention," per FEAT-27.SPEC-009.

**FEAT-27.SPEC-004-AC-06:** Given Talia selects an end date that is in the past, when she attempts to save, then she sees "The resume date can't be in the past. Choose today or a later date." and the save does not proceed.

**FEAT-27.SPEC-004-AC-07:** Given Talia has a Pro-chosen pause with an end date of today, when that date is reached, then FEAT-27.SPEC-011 resumes bookings automatically and this screen shows "Taking bookings" on next open.

**FEAT-27.SPEC-004-AC-08:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your pause settings. Check your connection and try again." and her entered values remain on screen.

**FEAT-27.SPEC-004-AC-09:** Given a support operator opens this screen via FEAT-19, when they view it, then the toggle, message, and end-date fields are visible but disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-004-AC-10:** Given Talia opens this screen with no connectivity, when the screen loads, then her last-known pause state is viewable and all controls are disabled with "You're offline -- pause changes need a connection."

**FEAT-27.SPEC-004-AC-11:** Given Talia's end date is reached while a system-imposed pause remains active, when the automatic resume fires (FEAT-27.SPEC-011), then the status banner updates to "Paused -- billing needs attention" rather than "Taking bookings."

**FEAT-27.SPEC-004-AC-12:** Given Talia has toggled the pause on and off several times without saving, when she navigates away without tapping Save, then her account's actual pause state is unchanged and the booking page is unaffected.

**FEAT-27.SPEC-004-AC-13:** Given Talia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (taking bookings, paused by you, paused billing, paused both, saving, error, support view, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Screen Spec: Notification Preferences

## Overview

**Name:** Notification Preferences
**ID:** FEAT-27.SPEC-005
**Type:** Screen
**Purpose:** Talia chooses which Pro notifications she receives and on which channel(s) -- in-app, text, or email.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Setting the channel combination for booking-activity notifications and attention alerts (FEAT-08.SPEC-005, FEAT-08.SPEC-006)
- Writing the notification_preferences field on the Pro Account

**Non-Goals:**
- Sending any notification -- every Pro notification's delivery, timing, retry, and content is owned by FEAT-08 (Automated Booking Messaging); this screen only sets the preference FEAT-08 reads on its next send
- Client-facing messaging consent -- owned by FEAT-14 (Messaging Consent Management); this is a distinct preference for the Pro's own account notifications, never subject to client SMS-consent rules
- A quiet-hours window for Pro notifications -- excluded per product-features.md FEAT-27 (Validation & Limits names only channel choice, not timing); no quiet-hours control is defined for the Pro's own notifications anywhere in Stage 2

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Notifications" row | Current notification_preferences value |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Change the channel selection | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen; their own texting-consent preference is a separate control owned by FEAT-14 |
| Platform Operator (Support) | Full screen, read-only | No actions -- selector shown disabled | Selector disabled and labeled "View-only in support mode" |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unsaved selection is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Notifications" with a back arrow (returns to FEAT-27.SPEC-001).

**Body:** A single selection control offering the four channel combinations used across the product's Pro notifications (FEAT-08.SPEC-005, FEAT-08.SPEC-006): "In-app only", "In-app + text", "In-app + email", "In-app + text + email". Helper text: "This applies to new booking activity and anything needing your attention -- delivery failures, calendar reconnection, refund issues, and disputes."

**Footer:** "Save" action button, enabled while there is an unsaved change.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| Channel selection control | Select an option | Captures the choice, does not save yet | Save button enables | Selected option shown highlighted |
| Save button | Tap | Writes notification_preferences to the Pro Account | Button shows loading state | Success: toast "Settings saved" -- no client message is sent (this is a Pro-only preference). Failure: inline error with retry, selection preserved |

### Accessibility Notes

- **Focus order:** Back arrow -> Channel selection control -> Save.
- **Save feedback:** "Settings saved" is announced on success; failure moves focus to the selection control with its exact message.
- **Keyboard alternatives:** The selection control and Save are both reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Viewing (default) | Current preference shown selected; Save disabled | Screen opens | Talia selects a different option |
| Editing | New selection shown; Save enabled | Talia changes the selection | Talia taps Save or navigates away |
| Saving | Save button shows loading state | Talia taps Save | Save completes or fails |
| Error | Inline error banner with retry; selection preserved | Save fails | Talia taps Retry |
| Support view (read-only) | Current preference shown, selector disabled | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- these settings need a connection to change." above the form; current preference remains viewable, selector disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- selector re-enables |

## Validation Rules

No field-level validation beyond selecting one of the four defined channel combinations -- the control offers only these four values, so an invalid selection cannot be entered.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful save | Stays on this screen showing the newly saved preference | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- notification_preferences (current value).
**Updates:** Pro Account -- notification_preferences.
**Deletes:** None.

## Business Rules

- This screen writes notification_preferences only -- it never sends a notification itself; FEAT-08 reads the new preference on its next send, per the Feature Breakdown Brief's Side-Effect Inventory.
- Preferences are evaluated at delivery time by FEAT-08, not at the moment they are changed here -- a change made after a notification's trigger but before its delivery governs that delivery, consistent with the product-wide rule stated in FEAT-08's own notification specs.
- Changing this preference sends no message to the Pro or to any client, per this feature's Non-Goals ("changes to settings send no client messages").

## Edge Cases

- **A notification triggers between Talia's edit and her Save tap** -- The preference in effect at delivery time governs (FEAT-08's own rule); an unsaved in-progress edit here has no effect until Save completes.
- **Network failure during save** -- Error banner: "Could not save your settings. Check your connection and try again." with a Retry button; Talia's selection remains shown.
- **Talia selects the option already saved (no actual change)** -- Save stays disabled; nothing is written.
- **Talia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Talia changes this preference from two signed-in devices at effectively the same time** -- Resolution: last-write-wins, per the dependency map's Contention note for the Pro Account entity's ordinary preference fields; whichever save commits last is the value FEAT-08 reads on its next send, and neither device is shown an error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound) | Entry from the "Notifications" row; back arrow returns there |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | References (outbound) | Reads this screen's saved preference to select delivery channels |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Reads this screen's saved preference to select delivery channels |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for this screen's read-only view |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| notification_preferences_updated | new_channel_combination (in_app_only / in_app_text / in_app_email / in_app_text_email) | A preference save completes successfully | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so preference-change activity is observable |

## Acceptance Criteria

**FEAT-27.SPEC-005-AC-01:** Given Talia is on the Notification Preferences screen, when it loads, then her currently saved channel combination is shown selected.

**FEAT-27.SPEC-005-AC-02:** Given Talia selects "In-app + text + email" and taps Save, when the save completes, then she sees "Settings saved" and no client receives any message from this change.

**FEAT-27.SPEC-005-AC-03:** Given Talia has saved "In-app only", when FEAT-08 next sends her a booking-activity notification, then it is delivered in-app only, per FEAT-08.SPEC-005.

**FEAT-27.SPEC-005-AC-04:** Given Talia changes her preference after a notification has already triggered but before FEAT-08 delivers it, when delivery occurs, then the newly saved preference governs that delivery.

**FEAT-27.SPEC-005-AC-05:** Given Talia's save fails from a network error, when the failure returns, then she sees "Could not save your settings. Check your connection and try again." and her selection remains shown.

**FEAT-27.SPEC-005-AC-06:** Given Talia selects the option that is already her current saved preference, when she views the Save control, then it remains disabled.

**FEAT-27.SPEC-005-AC-07:** Given a support operator opens this screen via FEAT-19, when they view it, then the selection control is disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-005-AC-08:** Given Talia opens this screen with no connectivity, when the screen loads, then her current preference is viewable and the selector is disabled with "You're offline -- these settings need a connection to change."

**FEAT-27.SPEC-005-AC-09:** Given Talia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-27.SPEC-005-AC-10:** Given Talia has never changed this setting, when she completes onboarding, then her preference defaults to "In-app + text" (FEAT-15.SPEC-008), which this screen shows selected on first visit.

**FEAT-27.SPEC-005-AC-11:** Given Talia's session expires while she has an unsaved selection, when she signs in again, then her unsaved selection is restored and Save remains enabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 6 (viewing, editing, saving, error, support view, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Help Request

## Overview

**Name:** Help Request
**ID:** FEAT-27.SPEC-006
**Type:** Screen
**Purpose:** Talia describes a problem and sends a help request to support from settings.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Capturing a free-text description of Talia's problem and creating a Help Request record on submit
- Triggering the acknowledgment notification

**Non-Goals:**
- Support's read-only lookup of the request, or any ticket status -- owned entirely by FEAT-19 (Platform Support Read-Only Access), per scope-boundaries.md SC-05 and coordination note 8
- The acknowledgment's content and delivery -- owned by FEAT-27.SPEC-013 (Help Request Acknowledgment); this screen only triggers it on submit
- Editing or withdrawing a previously sent help request -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: no edit capability is stated anywhere in Stage 2 for this feature

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Get help" row | None -- form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Submit a help request | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen and no equivalent capability in this feature |
| Platform Operator (Support) | No -- support does not use this submission form; support's own view of a submitted request is FEAT-19's read-only screen, not this one | No | Not applicable -- this screen has no support-facing mode; support's access is entirely through FEAT-19 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unsubmitted description is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Get Help" with a back arrow (returns to FEAT-27.SPEC-001).

**Body:** A single-column form:
- Description (multi-line text input, required, up to 1,000 characters, with a live remaining-character count), helper text "Tell us what's going on -- we'll get back to you."

**Footer:** "Send" action button, enabled only when Description is non-empty.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; Send remains in the footer.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| Description input | Type | Captures text input, updates remaining-character count | Field and counter update | Counter shows characters remaining |
| Send button | Tap | 1. Validates Description is non-empty and within the limit. 2. Creates the Help Request record. 3. Triggers FEAT-27.SPEC-013 (acknowledgment) | Button shows loading state during submit | Success: toast "Your request was sent -- we'll be in touch." and navigation returns to FEAT-27.SPEC-001. Failure: inline error with retry, entered text preserved |

### Accessibility Notes

- **Focus order:** Back arrow -> Description input -> Send.
- **Validation announcements:** The "Description is required" error is announced to assistive technology and associated with the field.
- **Submit feedback:** The success toast is announced; on failure, focus moves to the Description field and the error banner is announced.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Description field empty, Send disabled | Screen first opens | Talia begins typing |
| Filling | Description contains text, Send enabled once non-empty | Talia types | Talia taps Send or navigates away |
| Sending | Send button shows loading state, field disabled | Talia taps Send with valid input | Submit completes or fails |
| Error | Inline error banner with retry; entered text preserved | Submit fails | Talia taps Retry |
| Offline/Degraded | Banner "You're offline -- sending a help request needs a connection." above the form; Description remains editable for drafting but Send is disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- Send re-enables |

## Validation Rules

**Option B -- Inline:**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| description | Required, non-empty, max 1,000 characters | On submit | "Please describe your problem before sending" / "Your description must be 1,000 characters or fewer" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful send | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |

## Data Model

**Creates:** Help Request record -- description set from form input, created_at timestamp, and the requesting Pro Account reference (the reference FEAT-19's support lookup opens against).
**Reads:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- A submitted Help Request always triggers the acknowledgment notification (FEAT-27.SPEC-013) -- Talia cannot submit without it firing.
- Help Request records are retained indefinitely with no edit, withdraw, or delete path, per the Feature Breakdown Brief's Non-Goals ("Retention or purge policy for help requests" -- kept as part of the Pro's visible account activity, XBR-24).
- Support's subsequent access to this request is read-only and logged in Talia's visible account activity (XBR-24) -- this screen itself has no visibility into that access.

## Edge Cases

- **Talia submits with only whitespace in Description** -- Treated as empty; "Please describe your problem before sending" is shown and the submit does not proceed.
- **Network failure during submit** -- Error banner: "Could not send your request. Check your connection and try again." with a Retry button; her description text is preserved.
- **Talia navigates away with an unsubmitted description** -- Confirmation dialog: "You have an unsent message. Discard?" with "Discard" and "Keep Writing" options.
- **Talia submits two help requests in the same session** -- Each submission creates its own independent Help Request record and triggers its own acknowledgment; no deduplication is applied, since each is a distinct, deliberate request.
- **Talia taps Send twice rapidly** -- Second tap is ignored while the first submit is in progress (button in loading state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound and outbound) | Entry from the "Get help" row; back arrow and successful send both return there |
| FEAT-27.SPEC-013 (Help Request Acknowledgment) | Triggers (outbound) | A successful submit fires this notification |
| FEAT-19 (Platform Support Read-Only Access) | Triggers (outbound) | The created request is what support's read-only lookup opens against |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_request_sent | description_length | A help request submission completes successfully | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so help-seeking frequency is observable |

## Acceptance Criteria

**FEAT-27.SPEC-006-AC-01:** Given Talia is on the Help Request screen, when it loads, then the Description field is empty and Send is disabled.

**FEAT-27.SPEC-006-AC-02:** Given Talia types "My deposit link isn't showing the right price" and taps Send, when the submit completes, then a Help Request record is created, she sees "Your request was sent -- we'll be in touch.", and she is returned to FEAT-27.SPEC-001.

**FEAT-27.SPEC-006-AC-03:** Given Talia taps Send with an empty Description, when validation runs, then she sees "Please describe your problem before sending" and the submit does not proceed.

**FEAT-27.SPEC-006-AC-04:** Given Talia's submit fails from a network error, when the failure returns, then she sees "Could not send your request. Check your connection and try again." and her description text remains in the field.

**FEAT-27.SPEC-006-AC-05:** Given Talia has an unsent description and taps the back arrow, when the tap registers, then a confirmation dialog appears asking "You have an unsent message. Discard?" with "Discard" and "Keep Writing" options.

**FEAT-27.SPEC-006-AC-06:** Given Talia successfully submits a help request, when the submission completes, then the acknowledgment notification (FEAT-27.SPEC-013) fires.

**FEAT-27.SPEC-006-AC-07:** Given Talia opens this screen with no connectivity, when the screen loads, then she can still draft her description but Send is disabled with "You're offline -- sending a help request needs a connection."

**FEAT-27.SPEC-006-AC-08:** Given Talia types a description exceeding 1,000 characters and attempts to send, when validation runs, then she sees "Your description must be 1,000 characters or fewer" and the submit does not proceed.

**FEAT-27.SPEC-006-AC-09:** Given Talia submits a second, unrelated help request later in the same session, when it completes, then a second, independent Help Request record is created and its own acknowledgment fires.

**FEAT-27.SPEC-006-AC-10:** Given Talia taps Send twice in rapid succession, when the first submit is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (empty, filling, sending, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Booking Link Name Validation & Uniqueness Rule

## Overview

**Name:** Booking Link Name Validation & Uniqueness Rule
**ID:** FEAT-27.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines the format, length, and cross-pro uniqueness rules for booking_link_name, and the reject-with-refresh behavior when another pro claims a candidate name first.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings
**Governed Entity:** Pro Account -- the booking_link_name field and its forwarding-reservation state

## Scope and Non-Goals

**In Scope:**
- Format and length rules for booking_link_name
- Cross-pro uniqueness, including names reserved by another pro's still-active forwarding window
- The reject-with-refresh behavior when a candidate name is claimed by another pro between availability check and save
- Authorization for who may rename a booking link

**Non-Goals:**
- The forwarding and reservation-release mechanism itself (what happens after an accepted rename) -- owned by FEAT-27.SPEC-010 (Booking Link Forwarding & Reservation Expiry); this spec governs acceptance, not the consequence
- The rename screen's layout and suggestion-chip presentation -- owned by FEAT-27.SPEC-002 (Booking Link Rename); this spec only supplies the rules it enforces
- Resolving a renamed or forwarded link on the public booking page -- owned by FEAT-05.SPEC-001 and FEAT-05.SPEC-008; this spec governs the Pro Account field itself, not the client-facing lookup

## Governed Entity

**Entity:** Pro Account (booking_link_name and its forwarding-reservation state)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| booking_link_name | text | The public link segment identifying the Pro's booking page; unique across all pros |
| previous_link_names | derived (list) | Names this Pro Account has renamed away from, each with the timestamp its forwarding window began |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-27.SPEC-002 | Booking Link Rename | On field change (format/length), on pause in typing (availability), and on submit (final acceptance) |
| FEAT-27.SPEC-010 | Booking Link Forwarding & Reservation Expiry | Consumes this spec's acceptance outcome to begin forwarding and reservation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| booking_link_name | Required, non-empty | Always | On blur and on submit | "Enter a link name" | Yes |
| booking_link_name | 3-40 characters | Always | On blur and on submit | "Your link name must be 3-40 characters" | Yes |
| booking_link_name | Letters, numbers, or hyphens only | Always | On blur and on submit | "Only letters, numbers, and hyphens are allowed" | Yes |
| booking_link_name | Must not already be a different pro's active booking_link_name | Always | On pause in typing (availability check) and on submit | "That link name is taken" | Yes |
| booking_link_name | Must not be inside another pro's active forwarding-reservation window (platform parameter: `booking-link-forward-window-months`) | Always | On pause in typing (availability check) and on submit | "That link name is taken" (reservation and active-use are shown identically to the Pro renaming; the distinction is internal only) | Yes |
| previous_link_names | No validation beyond data type -- system-maintained, never directly editable | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Own-name reclaim exception | booking_link_name, previous_link_names | A candidate name that matches one of the requesting Pro's own previous_link_names entries is available to that same Pro, even while its forwarding-reservation window is still active -- the reservation exists to keep the name from other pros, not from its own former owner | N/A -- this is a permissive exception, not an error condition |
| No-op rename guard | booking_link_name (current) vs. candidate | If the candidate equals the Pro's current booking_link_name, the rename action is not offered as a change | "This is already your current link name" |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Rename booking link | The Pro (Talia) | Always, on her own Pro Account only | -- |
| Rename booking link | The Client (Riley) | Never | No control of any kind is exposed to clients; a client's own experience is limited to being forwarded transparently (XBR-27) |
| Rename booking link | Platform Operator (Support) | Never | The rename input and Save control are shown disabled and labeled "View-only in support mode" on FEAT-27.SPEC-002; support has no path to submit a rename |
| View current and previous link names | The Pro (Talia) | Always, her own account | -- |
| View current and previous link names | Platform Operator (Support) | Always, view-only, for troubleshooting (ASMP-20) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| booking_link_name (initial value) | Suggested from the Pro's display_name at account creation (FEAT-15), rendered available-by-construction | On Pro Account creation, before this spec's rules first apply | Yes -- Talia may change it immediately or at any later time through FEAT-27.SPEC-002 |
| previous_link_names | Appended with the outgoing name and a forwarding-start timestamp each time a rename is accepted | On every accepted rename (via FEAT-27.SPEC-010) | No -- system-maintained history, never user-edited |

## Business Rules

- XBR-27: a renamed booking link keeps forwarding from the old name for at least platform parameter: `booking-link-forward-window-months`; a closed, paused-to-closure, or mistyped link shows the plain "this booking page isn't available" message, never another pro's page.
- Uniqueness is checked against both currently active booking_link_name values across all pros and every name still inside another pro's forwarding-reservation window -- a name is available only when neither condition holds.
- Acceptance of a rename here is what authorizes FEAT-27.SPEC-010 to begin forwarding and reservation for the outgoing name; this spec never itself starts that mechanism.
- The reject-with-refresh resolution named by the dependency map's Pro Account Contention line applies specifically to this field: if a candidate name is claimed by another pro between the Pro's availability check and her save, the save is rejected and the screen (FEAT-27.SPEC-002) refreshes with current availability and fresh suggestions -- the first committed rename wins.

## Edge Cases

- **Candidate name at exactly 3 characters** -- Passes validation. 2 characters shows the length error.
- **Candidate name at exactly 40 characters** -- Passes validation. 41 characters shows the length error.
- **Candidate name with an underscore or space** -- Fails the character-set rule; "Only letters, numbers, and hyphens are allowed."
- **Two pros submit the same never-before-used candidate name at effectively the same moment** -- The first submission committed wins; the second is rejected with "That link name is taken" and its screen refreshes with fresh suggestions, per reject-with-refresh.
- **A pro renames back to a name she used longer ago than platform parameter: `booking-link-forward-window-months`, now outside anyone's reservation** -- Available to any pro, including a different pro than its original owner, since the reservation window has fully lapsed.
- **A pro attempts to reuse a name still inside her own forwarding-reservation window** -- Available to reclaim (Cross-Field Rules: Own-name reclaim exception), since the reservation protects against other pros, not the name's own former owner.
- **Support attempts to submit a rename through a direct request while viewing FEAT-27.SPEC-002** -- Not possible: the rename input and Save control do not exist in an actionable state in Support's view; this spec's Authorization Rules confirm Support is never an allowed actor for this action regardless of any client-side state.

## Acceptance Criteria

**FEAT-27.SPEC-007-AC-01:** Given Talia enters a candidate name of "ab" (2 characters), when validation runs, then she sees "Your link name must be 3-40 characters."

**FEAT-27.SPEC-007-AC-02:** Given Talia enters a candidate name of exactly 40 characters using only letters, numbers, and hyphens, when validation runs, then it passes the format and length checks.

**FEAT-27.SPEC-007-AC-03:** Given Talia enters a candidate name containing an underscore, when validation runs, then she sees "Only letters, numbers, and hyphens are allowed."

**FEAT-27.SPEC-007-AC-04:** Given Talia enters a candidate name currently active as another pro's booking_link_name, when the availability check runs, then she sees "That link name is taken."

**FEAT-27.SPEC-007-AC-05:** Given Talia enters a candidate name still inside another pro's forwarding-reservation window, when the availability check runs, then she sees "That link name is taken", identically to an actively-used name.

**FEAT-27.SPEC-007-AC-06:** Given Talia enters a candidate name that is one of her own previous_link_names still inside its own forwarding window, when the availability check runs, then it shows as available to her.

**FEAT-27.SPEC-007-AC-07:** Given Talia's candidate name is claimed by another pro between her availability check and her Save tap, when she submits, then her save is rejected with "That link name is taken" and FEAT-27.SPEC-002 refreshes with current availability, per reject-with-refresh.

**FEAT-27.SPEC-007-AC-08:** Given Talia enters her current booking_link_name unchanged, when she views the Save control on FEAT-27.SPEC-002, then it is disabled with "This is already your current link name."

**FEAT-27.SPEC-007-AC-09:** Given Talia (the Pro) attempts to rename her link, when she submits a valid, available name, then the action is allowed and FEAT-27.SPEC-010 is authorized to begin forwarding for the outgoing name.

**FEAT-27.SPEC-007-AC-10:** Given a support operator views FEAT-27.SPEC-002, when they look for a way to submit a rename, then no actionable input or Save control is available to them, consistent with this spec's Authorization Rules denying the action to Support entirely.

**FEAT-27.SPEC-007-AC-11:** Given two pros submit the same previously-unused candidate name within moments of each other, when the first submission commits, then the second is rejected with "That link name is taken" and refreshed suggestions.

**FEAT-27.SPEC-007-AC-12:** Given a name's forwarding-reservation window has fully lapsed (more than platform parameter: `booking-link-forward-window-months` since the rename), when any pro (including one other than its original owner) checks its availability, then it shows as available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Currency Lock Rule

## Overview

**Name:** Currency Lock Rule
**ID:** FEAT-27.SPEC-008
**Type:** Logic/Rule
**Purpose:** Determines whether the Pro Account's currency is still editable, locking it permanently the moment the account's first deposit is taken.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings
**Governed Entity:** Pro Account -- the currency field, evaluated against Deposit Transaction (read-only)

## Scope and Non-Goals

**In Scope:**
- The lock condition itself: whether currency is still editable
- The exact message shown when a currency change is blocked
- Authorization for who may attempt a currency change

**Non-Goals:**
- The timezone/currency screen's layout and save flow -- owned by FEAT-27.SPEC-003 (Timezone & Currency Settings); this spec only supplies the lock determination it defers to
- Computing deposit amounts or determining what counts as a "deposit taken" for pricing purposes -- owned by FEAT-07 (Deposit Payment at Booking); this spec only reads whether any Deposit Transaction exists for the account
- Unlocking currency after account reopening or any other lifecycle event -- product-features.md and BRIEF.md describe this as a permanent, one-way lock with no reset path; excluded as a deliberate design choice protecting every prior booking's price integrity

## Governed Entity

**Entity:** Pro Account (currency field)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| currency | text (enum of supported currencies) | The currency all of the Pro's prices, deposits, and payouts are expressed in; required per account |

**Referenced Entity (read-only):** Deposit Transaction -- checked for existence to determine whether the account's first deposit has occurred (FEAT-07's entity; fields not modified by this spec).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-27.SPEC-003 | Timezone & Currency Settings | On screen load (to show the selector as editable or locked) and on submit (final rejection if locked) |
| FEAT-07.SPEC-003 | Deposit Amount & Eligibility Rules | As a charge precondition -- currency must already equal the Pro Account's locked value before any deposit charge is authorized |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| currency | Editable only while no Deposit Transaction exists for the account | Always (this is the lock condition itself) | On screen load and on submit | "Your currency is locked because you've already taken a deposit. To keep every past booking's price accurate, currency can't change once you've started collecting payments." | Yes |
| currency | Must be one of the product's supported currency values | Always, while still unlocked | On selection and on submit | "Choose a supported currency" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Lock derives from Deposit Transaction existence, not from currency's own state | currency, Deposit Transaction (existence) | The lock is a derived boolean: locked = (at least one Deposit Transaction exists for this Pro Account). currency's own value never determines its own lock. | N/A -- this is a derivation rule, not a user-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change currency | The Pro (Talia) | Only while unlocked (no Deposit Transaction exists yet for the account) | Once locked: the currency selector on FEAT-27.SPEC-003 is disabled and shows the exact locked explanation above; a direct submit attempt (for example, from a stale screen state) is rejected with the same message |
| Change currency | The Client (Riley) | Never | No control of any kind is exposed to clients; currency is an account-wide Pro setting |
| Change currency | Platform Operator (Support) | Never | The currency selector is shown disabled and labeled "View-only in support mode" on FEAT-27.SPEC-003, regardless of lock state |
| View currency lock status | The Pro (Talia) | Always | -- |
| View currency lock status | Platform Operator (Support) | Always, view-only (ASMP-20) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| currency (initial value) | Set during onboarding (FEAT-15) based on the Pro's stated country/market | On Pro Account creation | Yes -- freely, until the lock condition below becomes true |
| is_locked (derived, not a stored field) | True if and only if at least one Deposit Transaction exists for this Pro Account; false otherwise | Evaluated live on every screen load and every submit attempt | No -- this is a system-derived state, never directly set by any role |

## Business Rules

- XBR-25: currency is fixed once the first deposit is taken; this is the authoritative business rule this spec implements for the Pro Account's currency field.
- FEAT-07.SPEC-003 separately checks the resulting lock as a charge precondition (currency must match the locked value before any deposit charge is authorized) -- this spec owns the lock's determination; FEAT-07.SPEC-003 consumes it, never redefines it.
- The lock is permanent and one-directional: once locked, no lifecycle event (account pause, reopening, or any other state change) unlocks it again.
- FEAT-28.SPEC-004 checks that the payout account's country and currency match the Pro Account's currency (SC-20); this spec's lock does not itself validate that match -- FEAT-28 surfaces any resulting mismatch independently.

## Edge Cases

- **Talia's very first deposit transaction is created for her account while she has the Timezone & Currency Settings screen open** -- The screen is not live-updating; her lock state is re-evaluated fresh on her next submit attempt, and if a Deposit Transaction now exists, the change is rejected with the exact locked message even though the selector appeared editable when the screen loaded.
- **Talia's account has a Deposit Transaction that was later fully refunded** -- The lock remains permanent: the rule is existence of a Deposit Transaction record, not its current status, so a refunded deposit still counts as "already taken" for locking purposes.
- **A brand-new Pro Account with zero bookings and zero deposits** -- currency is fully editable; the lock condition (at least one Deposit Transaction exists) is false.
- **Support views a locked account's currency setting** -- Shown disabled per the Authorization Rules, identically to how it would appear if Support could otherwise act (which it never can, locked or not).
- **Talia attempts to change currency at the exact same moment her first-ever booking's deposit charge succeeds** -- Whichever completes first is authoritative: if the deposit charge commits first, her currency-change submission is rejected with the locked message; if her currency-change submission somehow committed first (not possible under normal use since the lock check is evaluated at submit time against the current Deposit Transaction existence), no ambiguity arises because the check is a live existence read, not a cached flag.

## Acceptance Criteria

**FEAT-27.SPEC-008-AC-01:** Given Talia's Pro Account has never had a Deposit Transaction, when she views the Timezone & Currency Settings screen, then the currency selector is editable.

**FEAT-27.SPEC-008-AC-02:** Given Talia's Pro Account has at least one Deposit Transaction, when she views the currency selector, then it is disabled and shows "Your currency is locked because you've already taken a deposit. To keep every past booking's price accurate, currency can't change once you've started collecting payments."

**FEAT-27.SPEC-008-AC-03:** Given Talia's account locks between her screen load and her submit attempt, when she submits a currency change, then it is rejected with the exact locked message even though the selector appeared editable on load.

**FEAT-27.SPEC-008-AC-04:** Given Talia's account has a Deposit Transaction that was later fully refunded, when she views the currency selector, then it remains locked -- the refund does not unlock it.

**FEAT-27.SPEC-008-AC-05:** Given FEAT-07.SPEC-003 evaluates a deposit charge's eligibility, when it checks currency, then it relies on this spec's lock determination rather than re-deriving its own.

**FEAT-27.SPEC-008-AC-06:** Given Talia (the Pro) attempts to change currency while unlocked, when she selects a supported currency and saves, then the change is allowed.

**FEAT-27.SPEC-008-AC-07:** Given Talia attempts to select an unsupported currency value, when she submits, then she sees "Choose a supported currency."

**FEAT-27.SPEC-008-AC-08:** Given a support operator views a locked account's currency setting, when they look for an edit control, then none is offered -- the selector is disabled and labeled "View-only in support mode," regardless of the lock state.

**FEAT-27.SPEC-008-AC-09:** Given Talia's account has zero bookings and zero deposits, when she views the currency selector, then it is fully editable and shows no lock message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Pause State Precedence Rule

## Overview

**Name:** Pause State Precedence Rule
**ID:** FEAT-27.SPEC-009
**Type:** Logic/Rule
**Purpose:** Governs how a Pro-chosen pause and a system-imposed (subscription-lapse) pause coexist, including that the Pro's resume toggle cannot clear a system-imposed pause, and that a pause end date cannot be in the past.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings
**Governed Entity:** Pro Account -- the status field (pause sub-state), pause message, and pause end date

## Scope and Non-Goals

**In Scope:**
- The precedence logic between a Pro-chosen pause and a system-imposed pause
- Validation of the pause message and end date fields
- Authorization for who may set or clear each pause source

**Non-Goals:**
- Setting or lifting the system-imposed pause itself -- owned entirely by FEAT-18 (Pro Subscription Billing & Account Management, FEAT-18.SPEC-004); this spec only defines how that source interacts with the Pro-chosen one
- The Pause Bookings screen's layout and controls -- owned by FEAT-27.SPEC-004 (Pause Bookings); this spec only supplies the rules it enforces
- Automatically resuming the Pro-chosen pause when its end date arrives -- owned by FEAT-27.SPEC-011 (Automatic Pause Resume); this spec only defines what "resumed" means when a system-imposed pause may still be present

## Governed Entity

**Entity:** Pro Account (status field and its pause-related sub-fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum (Active, Paused) with independent pause-source flags | Whether the account is currently taking new bookings; Paused may result from the Pro's own choice, a system-imposed subscription lapse, or both simultaneously |
| pause_message | text | Optional message shown on the booking page in place of available times, set only for the Pro-chosen pause source |
| pause_end_date | date | Optional date on which the Pro-chosen pause automatically resumes; the system-imposed pause carries no end date of its own -- it lifts only when FEAT-18 restores billing |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-27.SPEC-004 | Pause Bookings | On screen load (to render the correct status banner) and on submit (toggle, message, end-date changes) |
| FEAT-27.SPEC-011 | Automatic Pause Resume | On the chosen end date, to determine whether resuming the Pro-chosen pause fully resumes bookings or leaves the account paused |
| FEAT-05.SPEC-008 | Booking Page Availability Gate | Reads the combined pause state to decide what a client sees |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pause_message | Optional, no length restriction beyond data type -- "no validation beyond data type" | Always | -- | -- | -- |
| pause_end_date | Cannot be in the past | Always, when provided | On selection and on submit | "The resume date can't be in the past. Choose today or a later date." | Yes |
| pause_end_date | Optional -- a pause with no end date requires a manual resume | Always | On submit | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Message and end date only apply while the Pro-chosen pause is on | pause toggle, pause_message, pause_end_date | If the Pro-chosen pause toggle is off, pause_message and pause_end_date are cleared and hold no effect (they are re-enterable the next time the toggle is turned on) | N/A -- not an error, a state-dependent clearing rule |
| Precedence combination rule | Pro-chosen pause state, system-imposed pause state | The account's overall status is Paused if either source is active; it is Active only when both are inactive. The two sources are tracked and can be cleared independently. | N/A -- this is the core precedence derivation, stated in Business Rules below |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Turn on the Pro-chosen pause (with optional message and end date) | The Pro (Talia) | Always, on her own account | -- |
| Turn off the Pro-chosen pause ("resume bookings") | The Pro (Talia) | Always accepted for the Pro-chosen pause specifically; never clears a system-imposed pause | If a system-imposed pause remains active after the toggle clears the Pro-chosen one, the screen states plainly: "Your bookings stay paused until your billing issue is resolved." -- never implying a full resume occurred |
| Set or lift the system-imposed pause | FEAT-18 (Pro Subscription Billing & Account Management), acting automatically on the Pro's behalf | Set on subscription lapse past the grace period; lifted on billing restoration | Not applicable to the Pro directly -- the Pro's only lever over this source is resolving billing through FEAT-18, which this spec's screen (FEAT-27.SPEC-004) links to |
| Turn the pause on/off | The Client (Riley) | Never | No control of any kind is exposed to clients |
| Turn the pause on/off | Platform Operator (Support) | Never | The pause toggle and related fields are shown disabled and labeled "View-only in support mode" on FEAT-27.SPEC-004 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status (overall pause state, derived) | Paused if either the Pro-chosen pause flag or the system-imposed pause flag is true; Active only when both are false | Evaluated live on every read (screen load, booking-page gate check) | No -- this is a system derivation from the two independent source flags |
| pause_message, pause_end_date | Empty/absent by default | On Pro Account creation | Yes -- set only when the Pro turns on the Pro-chosen pause |

## Business Rules

- XBR-14: a paused account (from either source) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged.
- The Pro's "resume bookings" toggle can only ever clear the Pro-chosen pause source -- it has no effect on a system-imposed pause, which lifts only when FEAT-18 confirms billing is restored (FEAT-18.SPEC-004).
- Reaching a Pro-chosen pause's end date (via FEAT-27.SPEC-011) clears the Pro-chosen source the same way the manual toggle does: if a system-imposed pause remains, the account stays overall Paused.
- A pause_end_date cannot be in the past at the moment it is set or re-saved -- this protects against a pause that would appear to have already silently expired.

## Edge Cases

- **Talia turns off the Pro-chosen pause while a system-imposed pause is also active** -- The Pro-chosen source clears immediately; the overall status stays Paused because the system-imposed source remains, and the screen states this outcome explicitly rather than implying a full resume.
- **A system-imposed pause is added by FEAT-18 while a Pro-chosen pause with an end date is already counting down** -- Both sources are now active; when the Pro-chosen pause's end date is reached, FEAT-27.SPEC-011 clears it, but the overall status stays Paused until FEAT-18 separately lifts the system-imposed source.
- **Talia sets a pause_end_date of today** -- Passes validation (today is not "in the past"); FEAT-27.SPEC-011 evaluates the resume at the end of that same day.
- **Talia sets a pause with no end date and no message** -- Valid; the pause continues until she manually turns it off (subject to the precedence rule above if a system-imposed pause coexists).
- **The system-imposed pause is lifted by FEAT-18 while the Pro-chosen pause remains active** -- The overall status stays Paused, now solely from the Pro-chosen source, and the screen's status banner updates from "Paused by you and by billing" to "Paused by you."
- **Both pause sources clear at effectively the same moment (Talia turns off her pause the instant FEAT-18 restores billing)** -- Each source's clearing is independent and idempotent; the overall status becomes Active only once both underlying flags are confirmed false, with no race condition since each source only ever clears its own flag.

## Acceptance Criteria

**FEAT-27.SPEC-009-AC-01:** Given Talia's account has only a Pro-chosen pause active, when she turns the toggle off, then the overall status becomes Active ("Taking bookings").

**FEAT-27.SPEC-009-AC-02:** Given Talia's account has both a Pro-chosen pause and a system-imposed pause active, when she turns the toggle off, then the Pro-chosen source clears but the overall status stays Paused, and she sees "Your bookings stay paused until your billing issue is resolved."

**FEAT-27.SPEC-009-AC-03:** Given Talia's account has only a system-imposed pause active, when she views the pause screen, then the Pro-chosen toggle is off and no resume action is offered for the system-imposed source -- only a link to resolve billing.

**FEAT-27.SPEC-009-AC-04:** Given Talia sets a pause_end_date in the past, when she attempts to save, then she sees "The resume date can't be in the past. Choose today or a later date." and the save does not proceed.

**FEAT-27.SPEC-009-AC-05:** Given Talia sets a pause_end_date of today, when validation runs, then it passes.

**FEAT-27.SPEC-009-AC-06:** Given a Pro-chosen pause's end date is reached while a system-imposed pause remains active, when FEAT-27.SPEC-011 fires, then the Pro-chosen source clears but the overall status stays Paused.

**FEAT-27.SPEC-009-AC-07:** Given FEAT-18 lifts a system-imposed pause while a Pro-chosen pause remains active, when the lift is confirmed, then the overall status stays Paused, now solely from the Pro-chosen source.

**FEAT-27.SPEC-009-AC-08:** Given Talia turns off the Pro-chosen pause toggle and FEAT-18 restores billing at effectively the same moment, when both clearings are confirmed, then the overall status becomes Active with no inconsistent intermediate state.

**FEAT-27.SPEC-009-AC-09:** Given Talia turns the Pro-chosen pause toggle off, when she next turns it back on, then pause_message and pause_end_date are re-enterable (their prior values were cleared when the toggle turned off).

**FEAT-27.SPEC-009-AC-10:** Given a support operator views the pause screen, when they look for a way to change either pause source, then no actionable control is available -- the toggle and fields are disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-009-AC-11:** Given a client visits the booking page while the account is Paused from either or both sources, when FEAT-05.SPEC-008 evaluates the gate, then the client sees the pause experience (message or default) regardless of which source or combination caused it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Booking Link Forwarding & Reservation Expiry

## Overview

**Name:** Booking Link Forwarding & Reservation Expiry
**ID:** FEAT-27.SPEC-010
**Type:** Automation
**Purpose:** On a booking-link rename, keeps the old link name forwarding to the new one for at least platform parameter: `booking-link-forward-window-months`, reserving it from reuse by any other pro until that window lapses, and then releases the reservation.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Recording the outgoing name as a forwarding entry the moment a rename is accepted
- Keeping that entry resolvable to the Pro's current booking_link_name for at least the forwarding window
- Releasing the reservation (making the name available to other pros) once the window lapses

**Non-Goals:**
- Deciding whether a candidate new name is valid or available -- owned by FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule); this automation only runs after that spec has already accepted the rename
- Resolving a visitor's request against the forwarding table on the public booking page -- owned by FEAT-05.SPEC-001 and FEAT-05.SPEC-008; this automation only maintains the record those specs read
- Notifying Talia that her old link is still working -- the rename screen's own success feedback (FEAT-27.SPEC-002) states the forwarding guarantee at save time; no separate notification is defined for this ongoing state

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking link rename accepted | FEAT-27.SPEC-002 (Booking Link Rename), via FEAT-27.SPEC-007's acceptance | Fires every time a rename passes FEAT-27.SPEC-007's format, length, and uniqueness checks and is saved | The outgoing booking_link_name, the new booking_link_name, the Pro Account reference, and the current timestamp |
| Forwarding-reservation window lapses | Schedule-based (system clock) | Fires once per forwarding entry, at least platform parameter: `booking-link-forward-window-months` after that entry's forwarding-start timestamp | The forwarding entry's outgoing name, its Pro Account reference, and its forwarding-start timestamp |

## Processing Logic

1. On a rename acceptance: record a new forwarding entry for the outgoing name, storing the outgoing name, the current (new) booking_link_name it should resolve to, the Pro Account reference, and the forwarding-start timestamp (the moment of this rename).
2. While a forwarding entry is within its window: any lookup by the outgoing name (from FEAT-05.SPEC-001 or FEAT-05.SPEC-008) resolves to the Pro Account's current booking_link_name at the time of the lookup -- not necessarily the name recorded at forwarding-start, since a Pro may rename more than once.
3. If the Pro renames again while an earlier forwarding entry is still active, chain resolution continues to work: each outgoing name always resolves forward to whatever the Pro's booking_link_name is at lookup time, not to the immediately-next name in the chain.
4. Continuously (or on each scheduled evaluation), check every forwarding entry's age against the forwarding window (platform parameter: `booking-link-forward-window-months`).
5. When a forwarding entry's age meets or exceeds the window, release its reservation: the outgoing name becomes available for use by any other pro (per FEAT-27.SPEC-007's uniqueness check), and it stops resolving on the public booking page.
6. A released name remains available to the Pro who originally owned it (per FEAT-27.SPEC-007's own-name reclaim exception) exactly as it would be to any other pro, once released -- release does not distinguish original ownership.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Forwarding entry created | A rename is accepted | New forwarding entry recorded for the outgoing name | None from this automation directly -- FEAT-27.SPEC-002's own save-success dialog already states the guarantee | FEAT-27.SPEC-002 (Booking Link Rename), FEAT-05.SPEC-001 / FEAT-05.SPEC-008 (resolve forwarded links) |
| Forwarding entry resolved (client visits old name) | A client visits a link inside its forwarding window | None -- read-only lookup | Client is forwarded transparently with no visible difference (XBR-27) | FEAT-05.SPEC-001, FEAT-05.SPEC-008 |
| Reservation released | A forwarding entry's age reaches the window | The outgoing name is removed from the active-reservation set; it stops resolving | None -- silent, since no client should be visiting a name this old, and the Pro receives no notification for this event (per Non-Goals) | FEAT-27.SPEC-007 (uniqueness check now sees the name as available) |
| No-op (automation runs, no entries due) | Scheduled evaluation finds no forwarding entry at or past its window | None | None | -- |
| Automation failure (release evaluation cannot complete) | A processing error prevents the scheduled release check from completing | No forwarding entries are released for that run; entries remain reserved (fail-safe, not fail-open) | None -- reservation continuing slightly longer than the minimum window is never client-visible or harmful, since the guarantee is "at least" the window, not "exactly" | -- |

## Data Model

**Reads:** Pro Account -- booking_link_name (current value, to resolve forwarding), previous_link_names / forwarding entries.
**Creates:** A forwarding entry per accepted rename -- outgoing name, Pro Account reference, forwarding-start timestamp.
**Updates:** Forwarding entry -- marked released once its window lapses.
**Deletes:** None -- a released entry is marked inactive, not erased, preserving the historical record of the Pro's own previous_link_names.

## Business Rules

- XBR-27: a renamed booking link keeps forwarding from the old name for at least platform parameter: `booking-link-forward-window-months`; a closed, paused-to-closure, or mistyped link shows the plain "this booking page isn't available" message, never another pro's page.
- The window is a floor, not a ceiling -- "at least" the stated duration, per BRIEF.md's and the Brief's own wording; a release evaluation failure that delays release past the window causes no incorrect behavior, since the guarantee is never violated by forwarding for longer.
- A forwarding entry always resolves to the Pro's current booking_link_name at lookup time, never to a stale intermediate name, even across multiple renames of the same account.
- Release makes a name available to any pro, including its original owner, on equal footing with any other candidate -- no owner-priority survives release.

## Edge Cases

- **A client visits an old link name the instant its forwarding window lapses** -- Whichever completes first is authoritative: if the release evaluation has already marked the entry inactive, the client sees the "this booking page isn't available" message (FEAT-05.SPEC-008); if the lookup completes microseconds before release, the client is forwarded normally. Either outcome is correct, since the window is a floor.
- **Talia renames her link twice within the same forwarding window** -- Two independent forwarding entries exist (one per outgoing name), each with its own forwarding-start timestamp and its own release time; both resolve forward to Talia's current booking_link_name until each individually lapses.
- **A forwarding entry's Pro Account is closed (FEAT-29) while the entry is still within its window** -- The forwarding entry itself is unaffected by this automation; FEAT-05.SPEC-008 separately renders the account's closed state to any visitor, including one arriving via a still-active forwarding entry, consistent with XBR-27's "closed... link shows the plain message" behavior.
- **Concurrent trigger firing (two renames on two different Pro Accounts at effectively the same time)** -- Each rename creates its own independent forwarding entry against its own account; there is no shared state between different pros' forwarding entries, so no conflict is possible between them.
- **Trigger fires while a previous run is in flight (the scheduled release evaluation is still processing when its next scheduled run would fire)** -- The next scheduled run for the same evaluation is skipped until the in-flight run completes, since re-running against partially-updated state could double-process the same entries; the following scheduled run picks up any entries the skipped run would have caught, and the "at least" floor guarantee is preserved regardless.
- **The release evaluation processes the same forwarding entry twice due to a retry after a partial failure** -- Marking an already-released entry as released again is a no-op; no duplicate release, no error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Booking Link Rename) | Triggered by (inbound) | An accepted rename save fires this automation |
| FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule) | Triggered by (inbound) / Affects (outbound) | Only a rename FEAT-27.SPEC-007 accepts reaches this automation; the release outcome feeds back into FEAT-27.SPEC-007's uniqueness check |
| FEAT-05.SPEC-001 (Public Booking Page & Booking Flow, landing/service list) | Affects (outbound) | Resolves forwarded links using this automation's active entries |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | Affects (outbound) | Resolves forwarded links, and renders the "isn't available" message for a lapsed or otherwise invalid link name |

## Analytics and Success Signals

- **booking_link_forward_created** (has_prior_forward: yes/no) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so forwarding-entry creation is observable
- **booking_link_reservation_released** (age_at_release_days) -- N/A -- no connected success-metrics.md metric; retained so the reservation lifecycle is observable rather than invisible

## Acceptance Criteria

**FEAT-27.SPEC-010-AC-01:** Given Talia renames her link from "talia-lashes" to "talia-beauty" and FEAT-27.SPEC-007 accepts it, when the rename saves, then a forwarding entry for "talia-lashes" is created, recording the current timestamp as its forwarding-start.

**FEAT-27.SPEC-010-AC-02:** Given a client visits "talia-lashes" within its forwarding window, when the lookup resolves, then the client is forwarded transparently to Talia's current booking page with no visible difference.

**FEAT-27.SPEC-010-AC-03:** Given Talia renames again from "talia-beauty" to "talia-studio" while "talia-lashes" is still within its own forwarding window, when a client visits "talia-lashes", then it resolves to Talia's current name, "talia-studio" -- not to the intermediate "talia-beauty".

**FEAT-27.SPEC-010-AC-04:** Given a forwarding entry's age reaches platform parameter: `booking-link-forward-window-months`, when the scheduled release evaluation runs, then the entry is marked released and the name becomes available to any pro via FEAT-27.SPEC-007's uniqueness check.

**FEAT-27.SPEC-010-AC-05:** Given a forwarding entry has just been released, when a client attempts to visit that name, then FEAT-05.SPEC-008 shows "this booking page isn't available" rather than resolving to any pro's page.

**FEAT-27.SPEC-010-AC-06:** Given a forwarding entry's original owner later wants that same name back after release, when she checks its availability via FEAT-27.SPEC-002, then it shows as available to her on the same footing as to any other pro.

**FEAT-27.SPEC-010-AC-07:** Given the scheduled release evaluation encounters a processing error mid-run, when the run fails, then no forwarding entries are released for that run, and the next scheduled run re-evaluates them.

**FEAT-27.SPEC-010-AC-08:** Given two different pros each rename their links at effectively the same time, when both renames are processed, then each creates its own independent forwarding entry with no interaction between them.

**FEAT-27.SPEC-010-AC-09:** Given a Pro Account with an active forwarding entry is closed via FEAT-29 while the entry is still within its window, when a client visits the old name, then FEAT-05.SPEC-008 shows the account's closed-state message, not a normal booking page.

**FEAT-27.SPEC-010-AC-10:** Given the release evaluation is re-run against an already-released forwarding entry (for example, after a retried partial failure), when it processes that entry again, then marking it released a second time has no effect and produces no error.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (rename accepted, window lapses) | 2 |
| Outcome Paths | 5 (entry created, resolved, released, no-op, failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Automatic Pause Resume

## Overview

**Name:** Automatic Pause Resume
**ID:** FEAT-27.SPEC-011
**Type:** Automation
**Purpose:** Resumes bookings automatically when a Pro-chosen pause reaches its end date, without disturbing a still-active system-imposed pause.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Detecting that a Pro-chosen pause's end date has been reached
- Clearing the Pro-chosen pause source and its message/end date
- Leaving a system-imposed pause untouched, per the precedence rule

**Non-Goals:**
- The precedence logic itself (what "resumed" means when a system-imposed pause coexists) -- owned by FEAT-27.SPEC-009 (Pause State Precedence Rule); this automation only executes the clearing action that rule defines
- Setting or lifting the system-imposed pause -- owned by FEAT-18 (Pro Subscription Billing & Account Management); this automation never reads or writes that source's own trigger conditions, only whether it is currently present
- Notifying Talia that her pause has resumed -- product-features.md's Communications field for this feature states "changes to settings send no client messages," and no Pro-facing resume notification is named in Stage 2; the resumed state is visible the next time she opens FEAT-27.SPEC-004 or FEAT-12

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pause end date reached | Schedule-based (system clock, evaluated against the Pro Account's pause_end_date) | Fires once, at the start of the day following the Pro-chosen pause's pause_end_date, in the Pro's own timezone (XBR-25) | The Pro Account reference, the Pro-chosen pause's message and end date, and the current system-imposed pause flag |
| Pause end date cleared or changed before it is reached | FEAT-27.SPEC-004 (Pause Bookings) | The Pro edits or removes the end date, or turns the pause off manually, before this automation's scheduled fire | The updated pause state, which supersedes the originally-scheduled fire |

## Processing Logic

1. For every Pro Account with an active Pro-chosen pause carrying a pause_end_date, evaluate whether that date has been reached, using the Pro's own account timezone (XBR-25).
2. If the date has been reached: clear the Pro-chosen pause flag, and clear pause_message and pause_end_date (per FEAT-27.SPEC-009's Cross-Field Rules, these are re-enterable next time the Pro turns the pause on again).
3. Check the current system-imposed pause flag (set independently by FEAT-18).
4. If the system-imposed pause flag is false: the account's overall status becomes Active -- bookings resume.
5. If the system-imposed pause flag is true: the account's overall status remains Paused, per FEAT-27.SPEC-009's precedence rule; only the Pro-chosen source has cleared.
6. If the Pro manually turned off the pause or changed its end date before this evaluation runs, this automation takes no action for that account on this cycle -- the manual action already superseded the scheduled one.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Fully resumed | Pause end date reached and no system-imposed pause is active | Pro-chosen pause flag, pause_message, and pause_end_date cleared; overall status becomes Active | Booking page shows available times again immediately; FEAT-27.SPEC-004 shows "Taking bookings" on next view | FEAT-27.SPEC-004 (Pause Bookings), FEAT-05.SPEC-008 (Booking Page Availability Gate) |
| Partially resumed (system-imposed pause remains) | Pause end date reached but a system-imposed pause is active | Pro-chosen pause flag, pause_message, and pause_end_date cleared; overall status stays Paused | FEAT-27.SPEC-004 shows "Paused -- billing needs attention" on next view, no longer "Paused by you and by billing" | FEAT-27.SPEC-004, FEAT-05.SPEC-008 |
| No-op (already superseded) | The Pro manually cleared or changed the pause before this evaluation ran | None -- the manual action already stands | None from this automation | FEAT-27.SPEC-004 |
| No-op (no pause due) | No Pro Account has a pause_end_date reached on this evaluation cycle | None | None | -- |
| Automation failure | A processing error prevents the scheduled evaluation from completing for one or more accounts | No change for the affected accounts on this cycle; they remain paused | None -- a pause continuing slightly past its chosen date due to a processing hiccup is corrected on the next scheduled evaluation, and is never worse than resuming a booking Talia did not intend to take | -- |

## Data Model

**Reads:** Pro Account -- status (Pro-chosen pause flag, system-imposed pause flag), pause_message, pause_end_date.
**Creates:** None.
**Updates:** Pro Account -- status (Pro-chosen pause flag cleared), pause_message (cleared), pause_end_date (cleared).
**Deletes:** None.

## Business Rules

- FEAT-27.SPEC-009 governs the precedence outcome this automation executes: clearing the Pro-chosen source never implies clearing a system-imposed one.
- XBR-14: while paused (from either source, before or after this automation runs), existing bookings and their reminders, refunds, and client self-service remain unaffected -- this automation touches only the pause state, never any Booking record.
- The evaluation uses the Pro's own account timezone (XBR-25) to determine when the end date is reached, so "today" for this purpose always matches what the Pro sees on her own settings screen.
- A manual pause change by the Pro always takes precedence over this automation's scheduled action for the same cycle -- the automation never overwrites a state the Pro has already changed.

## Edge Cases

- **Talia's pause_end_date is today, and she also manually turns off the pause a few minutes before this automation's scheduled evaluation** -- No-op: her manual action already cleared the Pro-chosen pause; this automation finds nothing left to do for her account on this cycle.
- **Talia's pause_end_date is today, and a system-imposed pause begins on the very same day (subscription lapses past its grace period)** -- Whichever is evaluated at the automation's fire time is authoritative: if the system-imposed pause is already flagged by then, this automation clears the Pro-chosen source but the account stays Paused; if it is flagged moments after this automation runs, the account briefly shows Active before the system-imposed pause takes effect on its own trigger.
- **Concurrent trigger firing (this automation's scheduled evaluation and a manual pause-toggle save from FEAT-27.SPEC-004 happen at effectively the same time)** -- Whichever commits first wins for that account; the second sees the already-updated state and either finds nothing to do (if the automation already cleared it) or overwrites a redundant clear (if the Pro's manual action already cleared it) -- either order yields the same final state, since both actions clear the identical fields.
- **Trigger fires while a previous run is in flight (the scheduled evaluation is still processing a large batch when its next scheduled cycle would fire)** -- The next scheduled cycle is skipped until the in-flight run completes, since re-running against partially-updated accounts could double-process the same pause; accounts not yet reached by the in-flight run are caught by the following cycle, and no pause resumes later than at most one evaluation cycle after its due date.
- **A Pro Account has a pause_end_date but the Pro Account itself was closed (FEAT-29) before the date is reached** -- This automation still runs its evaluation against the record, but FEAT-05.SPEC-008 already shows the account's closed-state message to any visitor regardless of this automation's outcome, so the resume has no visible effect for a closed account.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-004 (Pause Bookings) | Triggered by (inbound) / Affects (outbound) | A saved pause_end_date arms this automation; its outcome updates what the screen shows on next view |
| FEAT-27.SPEC-009 (Pause State Precedence Rule) | References (inbound) | Defines the precedence outcome this automation executes |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | References (inbound) | Supplies the system-imposed pause flag this automation checks before fully resuming |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | Affects (outbound) | Reflects the resumed (or still-paused) state on the public booking page immediately |

## Analytics and Success Signals

- **bookings_resumed** (trigger: automatic, still_paused_by_system: yes/no) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so automatic-resume activity, and how often a system-imposed pause outlives the Pro's own, is observable
- **automatic_resume_superseded** (reason: manual_change_first) -- N/A -- no connected success-metrics.md metric; retained so the frequency of manual pre-emption is observable

## Acceptance Criteria

**FEAT-27.SPEC-011-AC-01:** Given Talia has a Pro-chosen pause with an end date of today and no system-imposed pause, when this automation's scheduled evaluation runs, then the pause clears fully, her account becomes Active, and the booking page shows available times again.

**FEAT-27.SPEC-011-AC-02:** Given Talia has a Pro-chosen pause with an end date of today and a system-imposed pause is also active, when this automation runs, then the Pro-chosen source clears but her account stays Paused, showing "Paused -- billing needs attention" on next view.

**FEAT-27.SPEC-011-AC-03:** Given Talia manually turns off her pause a few minutes before this automation's scheduled evaluation, when the automation runs, then it finds nothing to do for her account.

**FEAT-27.SPEC-011-AC-04:** Given Talia's pause_end_date has not yet been reached, when this automation's scheduled evaluation runs, then her account is left unchanged.

**FEAT-27.SPEC-011-AC-05:** Given the scheduled evaluation encounters a processing error for a batch of accounts, when the run fails, then those accounts remain paused unchanged, and the next scheduled evaluation re-evaluates them.

**FEAT-27.SPEC-011-AC-06:** Given a system-imposed pause is lifted by FEAT-18 after this automation already cleared Talia's Pro-chosen pause on an earlier cycle, when the lift is confirmed, then her account becomes Active without any further action from this automation.

**FEAT-27.SPEC-011-AC-07:** Given this automation's scheduled evaluation and a manual pause-toggle save happen at effectively the same time for the same account, when both are processed, then the final state is the pause cleared exactly once, with no duplicate or conflicting write.

**FEAT-27.SPEC-011-AC-08:** Given Talia's pause_end_date evaluation uses her account's own timezone, when her local date reaches the chosen end date, then the automation fires for her account at that local boundary, not at a different pro's timezone boundary.

**FEAT-27.SPEC-011-AC-09:** Given a Pro Account with a due pause_end_date was closed before this automation's evaluation runs, when the automation processes it, then no visible change results, since FEAT-05.SPEC-008 already shows the closed-account message to any visitor regardless.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (end date reached, superseded by manual change) | 2 |
| Outcome Paths | 5 (fully resumed, partially resumed, no-op superseded, no-op none due, failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Integration Spec: Profile Photo Storage Capability

## Overview

**Name:** Profile Photo Storage Capability
**ID:** FEAT-27.SPEC-012
**Type:** Integration
**Purpose:** The product stores, replaces, and serves Talia's profile photo through an external file-storage capability, so the booking page still works correctly even when no photo has been set.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Uploading and replacing the Pro's profile photo within its size and format limits
- Serving the stored photo to the public booking page
- User-facing behavior when the storage capability is slow, unavailable, or rejects an upload
- Disclosure to the Pro about what is shared with this capability

**Non-Goals:**
- Choosing the storage vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for file storage.
- Editing, cropping, or otherwise transforming the photo before upload -- product-features.md's Validation & Limits names only a size/format limit, not an editing capability; no cropping or filter tool is defined anywhere in Stage 2.
- Storing any file other than the Pro's own profile photo -- no other file-upload capability exists anywhere in this product's feature set (studio_address and every other field in this feature are plain text).
- The screen mechanics of triggering an upload -- owned by FEAT-27.SPEC-001 (Profile & Booking Page Settings); this spec defines only the storage-capability behavior that screen surfaces.

## Capability Category

**Category:** File storage
**Dependency Source:** ASMP-35 -- "File storage capability for pro profile photos" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "File storage -- Pro profile photos" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-27, FEAT-05)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia uploads or replaces her profile photo from settings | Edit the public profile -- display name, photo, short intro, general area | FEAT-27.SPEC-001 (Profile & Booking Page Settings) |
| The public booking page displays Talia's photo, or works correctly with just her name when none is set | Preview the booking page as a client sees it; the booking loop is unaffected by photo absence, per product-features.md's Rationale | FEAT-05 (Public Booking Page & Booking Flow) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Photo file | Pro Account -- photo (the raw uploaded file) | Talia uploads or replaces her photo on FEAT-27.SPEC-001 | The capability must have the file to store and later serve it |

No other Pro Account field, and no Client or Booking data, ever leaves the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Stored-photo reference (a servable location for the file) | The capability confirms the upload succeeded | Pro Account -- photo (the servable reference FEAT-05 reads to display the photo) |
| Upload rejection reason (size / format) | The capability rejects an upload | Not persisted -- shown inline on FEAT-27.SPEC-001 and discarded; the Pro Account's photo field is left unchanged |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Upload succeeded | The capability finishes storing an uploaded or replacement photo | Pro Account -- photo set to the new stored-photo reference | FEAT-27.SPEC-001 shows the new photo in place; the public booking page (FEAT-05) reflects it immediately | FEAT-27.SPEC-001, FEAT-05 |
| Upload rejected (size exceeded) | The uploaded file exceeds the size limit (platform parameter: `profile-photo-max-file-size-mb`) | None -- Pro Account's photo field is unchanged | FEAT-27.SPEC-001 shows inline: "This photo is too large. Choose one under {the limit}." | FEAT-27.SPEC-001 |
| Upload rejected (unsupported format) | The uploaded file is not a common photo file type the capability accepts | None -- Pro Account's photo field is unchanged | FEAT-27.SPEC-001 shows inline: "This file type isn't supported. Choose a common photo format." | FEAT-27.SPEC-001 |
| Stored photo becomes unservable (capability-side loss or corruption, reported after the fact) | The capability reports it can no longer serve a previously-stored photo | Pro Account -- photo cleared back to empty | The public booking page (FEAT-05) falls back to showing Talia's name only, exactly as the Empty state already handles no-photo; FEAT-27.SPEC-001 shows the "Add a photo" prompt again on next view | FEAT-27.SPEC-001, FEAT-05 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | The photo element shows an uploading state; after 10 seconds a note appears: "Still uploading -- this is taking longer than usual." The rest of the settings screen remains fully usable and other fields can still be saved. | The photo element shows: "Photo upload is temporarily unavailable. The rest of your settings are unaffected -- try the photo again in a few minutes." Talia can still save every other field on this screen. | The exact rejection reason (size or format) is shown inline below the photo element, per Inbound Events above; the previous photo (or empty state) remains unaffected. |
| FEAT-05 (Public Booking Page & Booking Flow, photo display) | The booking page renders without waiting on the photo -- text content (name, services) appears immediately; the photo fades in once it loads, or the page proceeds with no photo if it does not load within the page's own rendering budget. | The booking page shows Talia's name and services with no photo -- identical to the empty-photo state; no error is shown to the client, since a missing photo is never a client-facing failure. | N/A -- FEAT-05 only reads a already-stored photo; it never submits an upload that could be rejected. |

## Consent and Disclosure

- **First photo upload disclosure** -- The first time Talia uploads a photo, a notice appears before the upload begins: "Your photo is shared with an external file-storage service so it can be displayed on your public booking page." Options: "Continue" and "Cancel." Shown once; afterwards no repeat notice appears for subsequent replacements, since the same storage relationship already applies.
- **What is never shared** -- No Client data, no Booking data, and no Pro Account field other than the photo file itself ever leaves the product through this capability. This boundary is stated in the disclosure notice.
- **Client-facing exposure** -- The stored photo is publicly servable on Talia's booking page by design (it is a public profile field, per the Pro Account entity's Data Sensitivity note); no additional client-facing disclosure applies beyond what FEAT-05 already states about the page being public.

## Edge Cases

- **Upload succeeds but the storage capability reports the photo unservable moments later** -- Handled as the "Stored photo becomes unservable" inbound event above: the field clears and both settings and the public page revert to the no-photo appearance, with no error blame directed at Talia.
- **The same upload-succeeded event is delivered twice (a retried confirmation)** -- The second delivery changes nothing: the photo field already holds the correct stored-photo reference, and no duplicate upload or duplicate feedback occurs.
- **An upload-rejected event arrives after Talia has already navigated away from FEAT-27.SPEC-001** -- No feedback is shown (there is no screen open to show it on); her photo field remains at its previous value, and she sees the empty/previous state normally on her next visit, with no attempt she is unaware of having silently succeeded.
- **Capability goes down mid-upload** -- If the upload was not confirmed stored, Talia's Pro Account photo field is unaffected -- no half-uploaded or broken reference is ever written; FEAT-05 continues serving whatever photo (or no photo) was already confirmed before the outage.
- **Talia uploads a replacement photo while the public booking page is being viewed by a client at that exact moment** -- The client's already-loaded page view is unaffected mid-view; the next page load (or refresh) shows the new photo, consistent with the Feature Breakdown Brief's Non-Functional Notes ("the change is visible on the public booking page immediately" -- immediacy applies to new loads, not to a page already rendered in a client's browser).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Triggered by (inbound) | The photo element's upload/replace action initiates a storage request |
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Affects (outbound) | Upload outcomes, degradation states, and the disclosure notice surface here |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | The stored photo (or its absence) is served here on every page load |

## Analytics and Success Signals

- **profile_photo_uploaded** (outcome: success / failure; failure_reason: size / format / unavailable) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so photo-upload adoption and friction are observable
- **profile_photo_became_unservable** () -- N/A -- no connected success-metrics.md metric; retained so this rare capability-side event is observable rather than silently degrading a Pro's page

## Acceptance Criteria

**FEAT-27.SPEC-012-AC-01:** Given Talia has never uploaded a photo, when she taps to upload one for the first time, then the data-sharing notice appears with "Continue" and "Cancel", and no file leaves the product until she chooses "Continue".

**FEAT-27.SPEC-012-AC-02:** Given Talia uploads a valid photo within the size and format limits, when the capability confirms storage, then her photo updates on FEAT-27.SPEC-001 and on the public booking page (FEAT-05).

**FEAT-27.SPEC-012-AC-03:** Given Talia uploads a photo exceeding platform parameter: `profile-photo-max-file-size-mb`, when the capability rejects it, then she sees "This photo is too large. Choose one under {the limit}." and her previous photo (or empty state) is unaffected.

**FEAT-27.SPEC-012-AC-04:** Given Talia uploads a file in an unsupported format, when the capability rejects it, then she sees "This file type isn't supported. Choose a common photo format." and her previous photo is unaffected.

**FEAT-27.SPEC-012-AC-05:** Given a client visits Talia's booking page and she has never set a photo, when the page loads, then it shows her name and services with no photo and no error.

**FEAT-27.SPEC-012-AC-06:** Given the storage capability is unavailable when Talia attempts an upload, when the request cannot be sent, then she sees "Photo upload is temporarily unavailable. The rest of your settings are unaffected -- try the photo again in a few minutes." and can still save her other fields.

**FEAT-27.SPEC-012-AC-07:** Given Talia's already-stored photo becomes unservable at the capability, when that is reported, then her Pro Account's photo field clears and the public booking page falls back to showing her name only.

**FEAT-27.SPEC-012-AC-08:** Given an upload-succeeded confirmation is delivered twice for the same upload, when the second delivery arrives, then nothing changes and no duplicate feedback appears.

**FEAT-27.SPEC-012-AC-09:** Given the storage capability goes down mid-upload before confirming success, when Talia checks her settings afterward, then her photo field is unaffected -- no broken or half-uploaded reference exists.

**FEAT-27.SPEC-012-AC-10:** Given Talia's photo upload is rejected after she has already left FEAT-27.SPEC-001, when the rejection event arrives, then no feedback is shown on any screen, and her photo field remains at its previous value.

**FEAT-27.SPEC-012-AC-11:** Given Talia replaces her photo while a client is already viewing her booking page, when the client's page was loaded before the replacement, then the client's current view is unaffected; the new photo appears on the client's next page load.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 5 (2 screens; 1 N/A cell excluded) | 5 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |



# Notification Spec: Help Request Acknowledgment

## Overview

**Name:** Help Request Acknowledgment
**ID:** FEAT-27.SPEC-013
**Type:** Notification
**Purpose:** Confirms to Talia that her help request was received, carrying the reference support's read-only lookup (FEAT-19) points to.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- The acknowledgment delivered the moment a help request is submitted, on every channel it uses
- Preference, retry, and expiry behavior for this acknowledgment

**Non-Goals:**
- Any follow-up communication about the request's resolution or status -- excluded per scope-boundaries.md SC-05 and coordination note 8: any request-status lifecycle belongs to FEAT-19's support-side handling, entirely out of scope for this feature; this notification is a one-time receipt confirmation only.
- Deciding whether the request was created -- owned by FEAT-27.SPEC-006 (Help Request); this notification begins where that screen's successful submit fires it.
- Batching multiple help requests into one acknowledgment -- product-features.md's Communications field describes a single acknowledgment per request, and each help-seeking moment is treated as its own distinct event worth its own confirmation, not a nag to be consolidated.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when the acknowledgment is delivered | Talia is on the settings screen at the moment she submits, so an immediate in-app confirmation matches the on-screen "Your request was sent" feedback already shown by FEAT-27.SPEC-006 |
| Text | When Talia's notification preferences (FEAT-27.SPEC-005) include text | Talia does nearly all of her Chairtime use on her phone in short bursts (user-persona.md, Behavioral Context); a text reaches her even if she has already closed the app after submitting |
| Email | When Talia's notification preferences (FEAT-27.SPEC-005) include email, or as the fallback when a text send fails (per FEAT-08.SPEC-009's product-wide retry/fallback pattern) | Matches the product-wide email-fallback pattern already established for every other Pro notification |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Help request submitted | FEAT-27.SPEC-006 (Help Request) | Fires immediately when a Help Request record is successfully created | The Pro Account reference, the Help Request's reference (the identifier FEAT-19's support lookup opens against), and the submission timestamp |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- the sole recipient, since a help request is her own submission and this acknowledgment confirms receipt to her alone; no other role is entitled to it (Support's own visibility into the request is a distinct, read-only capability owned by FEAT-19, not a copy of this notification).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Pro notification channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text | FEAT-27.SPEC-005 (Notification Preferences) |

**Quiet Hours:** N/A -- no quiet-hours window is defined anywhere in Stage 2 for Pro-directed notifications; a help request is a Pro-initiated action taken in the moment, so an immediate acknowledgment on every configured channel is the expected and least surprising behavior, with no reason to hold it.

## Content Definition

**In-app:**
- **Title:** We got your message
- **Body:** Your request was received. We'll be in touch.
- **CTA:** None -- this is a receipt confirmation with nothing further for Talia to act on; she returns to settings on her own.

**Text:**
- **Body:** Chairtime: We got your help request and will be in touch soon.

**Email:**
- **Subject:** We got your help request
- **Body:**
  Hi {pro_first_name},

  We received your message and will be in touch soon.

  Reference: {help_request_reference}
- **CTA:** None.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_first_name} | Pro Account -- display_name (first token) | Talia | Greeting renders as "Hi," -- display_name is required and non-empty on every Pro Account (FEAT-27.SPEC-001 validation), so this fallback is defensive only |
| {help_request_reference} | Help Request -- reference (the identifier created by FEAT-27.SPEC-006) | HR-20260927-0142 | Never empty -- this notification only fires after a Help Request record is successfully created, which always carries a reference |

## Delivery Rules

**Batching:** None -- each help request produces exactly one acknowledgment, delivered independently, per this spec's Non-Goals.
**Deduplication:** At most one acknowledgment per Help Request record. A retried delivery attempt after a transient failure never produces a second acknowledgment for the same request -- delivery is retried, not re-triggered.
**Retry on failure:** Text delivery failure is retried once, then falls back to email, per the product-wide message-delivery discipline (platform parameter: `message-delivery-retry-count`, FEAT-08.SPEC-009); after the final failure on every configured channel, the in-app acknowledgment (which has no delivery-failure mode of its own, since it renders directly in the product) stands as the delivery of record, and no alarming failure message is shown to Talia.
**Expiry:** This acknowledgment never expires undelivered in a way that changes its content -- if text and email both ultimately fail, the in-app confirmation remains available indefinitely the next time Talia opens the product, since it carries no time-sensitive action.

## Edge Cases

- **Talia's notification preferences are changed between her request's submission and this acknowledgment's delivery (for example, she turns off text moments after submitting)** -- The preference in effect at delivery time governs, consistent with the product-wide rule that preferences are evaluated at delivery time rather than trigger time (FEAT-27.SPEC-005's own Business Rules); if text is turned off before the send completes, it is not attempted on that channel, and the in-app confirmation (no off switch, per Preference Controls) still delivers.
- **Talia submits a help request while offline, and the request only reaches the product once connectivity returns** -- The acknowledgment fires only after the Help Request record is actually created (per its Trigger), so no acknowledgment is ever sent for a submission that has not yet succeeded; there is no separate "held" state for this notification to manage.
- **Both text and email delivery fail for this acknowledgment** -- Per Delivery Rules, the in-app confirmation stands as the delivery of record and no failure is surfaced to Talia as an alarming error; the product-wide retry/fallback discipline (FEAT-08.SPEC-009) already governs the underlying channel mechanics.
- **Talia submits two help requests in quick succession** -- Each Help Request record produces its own independent acknowledgment; per Non-Goals, they are never batched into one, since each is treated as its own distinct receipt.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-006 (Help Request) | Triggered by (inbound) | A successful submission fires this notification |
| FEAT-27.SPEC-005 (Notification Preferences) | References (inbound) | Preference control governs channel selection |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (outbound) | Carries this notification's text send when text is a configured channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (outbound) | Carries this notification's email send when email is a configured channel, or as the text-failure fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Supplies the retry-then-fallback discipline this notification's Delivery Rules follow |
| FEAT-19 (Platform Support Read-Only Access) | References (outbound) | The help_request_reference this notification carries is the identifier FEAT-19's support lookup opens against |

## Analytics and Success Signals

- **help_request_acknowledgment_delivered** (channel: in_app / text / email) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so acknowledgment delivery is observable
- **help_request_acknowledgment_opened** (channel) -- N/A -- no connected success-metrics.md metric; retained so engagement with the receipt is observable
- **help_request_acknowledgment_channel_failed** (channel, fallback_used: yes/no) -- N/A -- no connected success-metrics.md metric; retained so channel-level delivery friction is observable rather than invisible

## Acceptance Criteria

**FEAT-27.SPEC-013-AC-01:** Given Talia has default notification preferences (In-app + text) and submits a help request, when the request is created, then she receives an in-app confirmation titled "We got your message" and a text reading "Chairtime: We got your help request and will be in touch soon."

**FEAT-27.SPEC-013-AC-02:** Given Talia has set her preferences to "In-app + email", when she submits a help request, then she receives the in-app confirmation and an email with the subject "We got your help request" carrying her help request's reference.

**FEAT-27.SPEC-013-AC-03:** Given Talia has set her preferences to "In-app only", when she submits a help request, then she receives only the in-app confirmation, with no text or email sent.

**FEAT-27.SPEC-013-AC-04:** Given Talia's text delivery for this acknowledgment fails, when the retry also fails, then the acknowledgment falls back to email, per FEAT-08.SPEC-009.

**FEAT-27.SPEC-013-AC-05:** Given both text and email delivery fail for this acknowledgment, when the final failure is reached, then the in-app confirmation stands as the delivery of record and no alarming failure message is shown to Talia.

**FEAT-27.SPEC-013-AC-06:** Given Talia changes her notification preferences from "In-app + text" to "In-app only" moments after submitting a help request but before this acknowledgment is delivered, when delivery occurs, then only the in-app confirmation is delivered, per the delivery-time preference rule.

**FEAT-27.SPEC-013-AC-07:** Given Talia submits two help requests in quick succession, when both are created, then she receives two independent acknowledgments, each carrying its own request's reference -- never batched into one.

**FEAT-27.SPEC-013-AC-08:** Given Talia's email acknowledgment is delivered, when she opens it, then the body reads "Hi {pro_first_name}, We received your message and will be in touch soon." followed by her request's reference.

**FEAT-27.SPEC-013-AC-09:** Given a support operator later opens Talia's account via FEAT-19 using this acknowledgment's carried reference, when they look up the request, then it resolves to the exact Help Request record this notification's reference names.

**FEAT-27.SPEC-013-AC-10:** Given Talia's help request submission has not yet succeeded (for example, she is offline when she taps Send), when she checks for this acknowledgment, then none has been sent, since the trigger only fires after the Help Request record is actually created.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 1 | 1 |
| Preference States | 4 (in-app only, in-app + text, in-app + email, in-app + text + email) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |

