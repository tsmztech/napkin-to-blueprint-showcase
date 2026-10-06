---
document_type: feature-overview
feature_number: FEAT-27
feature_name: Pro Profile & Booking Page Settings
feature_slug: pro-profile-booking-page-settings
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 13
screen_count: 6
automation_count: 2
logic_rule_count: 3
integration_count: 1
notification_count: 1
---

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
