---
document_type: feature-overview
feature_number: FEAT-29
feature_name: Pro Sign-In & Account Lifecycle
feature_slug: pro-sign-in-account-lifecycle
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 17
screen_count: 5
automation_count: 5
logic_rule_count: 3
integration_count: 0
notification_count: 4
---

# Feature Breakdown Brief: Pro Sign-In & Account Lifecycle

## Summary

**Feature:** Pro Sign-In & Account Lifecycle
**ID:** FEAT-29
**Description:** The Pro signs in securely from their phone and stays signed in between clients, can get back in if they lose access to one of their contact methods, can take a copy of their clients and booking history with them at any time, and can close their account with their data deleted afterward.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The draft referred to "the Pro's own dashboard login" (FEAT-06) but no feature owned signing in, recovering access, or closing the account. The Pro's account holds every client's contact details, so protecting it is part of BRIEF.md's Privacy requirement ("a client's data is visible only to their pro"), and BRIEF.md's Business Context promises "cancel anytime," which is only honest if a Pro can leave with their records. Important rather than Core because it guards access to the Core features rather than delivering the booking value itself; MVP because a Pro cannot reach their dashboard without it. [AUDIT-ADDED: 4 -- Account Management, Data Management, and Security and Privacy Posture concerns were not owned by any feature; sign-in, recovery, data export and account closure are grouped here]

**Key Capabilities:**
- Create a sign-in during onboarding using an email address and a mobile number, with a one-time code rather than a password to remember
- Stay signed in on their own phone between clients, and sign out of every device at once
- Recover access through whichever of the two contact methods they still have
- Change the sign-in email or phone, confirmed through both the old and the new contact
- Download a copy of their client list and booking history as a spreadsheet-friendly file
- Close the account -- subscription cancelled, booking page taken down, data deleted after a cooling-off period

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-29.SPEC-001 | Sign-In Screen | Screen | The Pro | Pro enters their email or mobile number, requests a one-time code, and enters it to sign in (also the screen every unauthenticated visitor to a Pro-only area lands on, per XBR-29) |
| FEAT-29.SPEC-002 | Account Recovery Screen | Screen | The Pro | Pro who has lost access to one contact method regains sign-in through whichever contact method they still control |
| FEAT-29.SPEC-003 | Account & Sign-In Settings Screen | Screen | The Pro, Platform Operator (Support) | Pro's ongoing hub for signed-in devices, signing out everywhere, and starting a sign-in-contact change, data export, or account closure; Support's view-only entry point for account status |
| FEAT-29.SPEC-004 | Data Export Screen | Screen | The Pro | Pro requests and downloads a spreadsheet-friendly file of their own clients and booking history |
| FEAT-29.SPEC-005 | Account Closure & Reopening Screen | Screen | The Pro | Pro reviews upcoming bookings and confirms closure, or -- during the cooling-off period -- reopens the account with everything intact |
| FEAT-29.SPEC-006 | Session & Device Management | Automation | The Pro | Verifies a submitted one-time code, creates or refreshes a signed-in device on success, expires devices after 30 days of inactivity, and executes "sign out everywhere" |
| FEAT-29.SPEC-007 | Data Export Generation | Automation | The Pro | Assembles the requested spreadsheet-friendly file from the Pro's own Client, Booking and Deposit Transaction records on request |
| FEAT-29.SPEC-008 | Account Closure Orchestration | Automation | The Pro | Sequences closure: hands upcoming bookings to bulk cancellation with full refunds, cancels the subscription, takes the booking page down, starts the 30-day cooling-off clock, and executes deletion (retaining only legally required de-identified financial records) when it expires unreversed |
| FEAT-29.SPEC-009 | Account Reopening | Automation | The Pro | Restores a Closing account to Active when the Pro signs back in during the cooling-off period, leaving the booking page down until the Pro separately resumes it |
| FEAT-29.SPEC-010 | Contact-Detail Change Processing | Automation | The Pro | Carries a sign-in email or mobile-number change through confirmation on both the old and the new contact before committing it |
| FEAT-29.SPEC-011 | Sign-In & Recovery Rules | Logic/Rule | The Pro | Governs one-time-code expiry, the failed-attempt lockout, session duration, and the anti-enumeration rule that a failed sign-in never reveals whether an account exists |
| FEAT-29.SPEC-012 | Contact-Change Confirmation Rules | Logic/Rule | The Pro | Governs the dual-confirmation requirement for sign-in-contact changes and what a partial or abandoned change leaves in place |
| FEAT-29.SPEC-013 | Account Closure & Retention Rules | Logic/Rule | The Pro, Platform Operator (Support) | Governs the 30-day cooling-off period, the closure sequencing required before deletion, the scope of what deletion removes versus retains, and the scope of the data export |
| FEAT-29.SPEC-014 | Sign-In Code Notification | Notification | The Pro | Delivers the one-time sign-in code to whichever contact method the Pro is signing in with |
| FEAT-29.SPEC-015 | New-Device Sign-In Alert | Notification | The Pro | Alerts the Pro on their existing contact methods when their account is signed in on a device not seen before |
| FEAT-29.SPEC-016 | Contact-Change Confirmation Notification | Notification | The Pro | Sends confirmation of a sign-in email or mobile-number change to both the old and the new contact |
| FEAT-29.SPEC-017 | Account Closure & Deletion Notifications | Notification | The Pro | Sends the account-closure confirmation when closure is requested and the final notice when data is permanently deleted |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Create a sign-in during onboarding using an email address and a mobile number, with a one-time code rather than a password to remember | FEAT-29.SPEC-001, FEAT-29.SPEC-006, FEAT-29.SPEC-014 | The Sign-In Screen collects both contacts and the code during the FEAT-15 onboarding step; Session & Device Management verifies the code and creates the first signed-in device; the code itself is delivered by the Sign-In Code Notification | Phase 2 (Explicit) |
| Stay signed in on their own phone between clients, and sign out of every device at once | FEAT-29.SPEC-003, FEAT-29.SPEC-006 | The Settings screen lists devices and offers "sign out everywhere"; Session & Device Management holds each device active for up to 30 days of inactivity and clears every device on that action | Phase 2 (Explicit) |
| Recover access through whichever of the two contact methods they still have | FEAT-29.SPEC-002, FEAT-29.SPEC-011 | The Recovery screen accepts either surviving contact; Sign-In & Recovery Rules govern the same code/lockout/anti-enumeration behavior as ordinary sign-in | Phase 2 (Explicit) |
| Change the sign-in email or phone, confirmed through both the old and the new contact | FEAT-29.SPEC-003, FEAT-29.SPEC-010, FEAT-29.SPEC-012, FEAT-29.SPEC-016 | The Settings screen starts the change; Contact-Detail Change Processing runs the dual confirmation the Rules spec governs; the Notification spec confirms to both contacts | Phase 2 (Explicit) |
| Download a copy of their client list and booking history as a spreadsheet-friendly file | FEAT-29.SPEC-004, FEAT-29.SPEC-007, FEAT-29.SPEC-013 | The Export screen requests and downloads the file; Data Export Generation assembles it from Client, Booking and Deposit Transaction; the Retention & Export Rules spec bounds its scope to the Pro's own records | Phase 2 (Explicit) |
| Close the account -- subscription cancelled, booking page taken down, data deleted after a cooling-off period | FEAT-29.SPEC-005, FEAT-29.SPEC-008, FEAT-29.SPEC-009, FEAT-29.SPEC-013, FEAT-29.SPEC-017 | The Closure screen shows upcoming bookings and confirms closure or reopening; Closure Orchestration sequences the hand-offs and the eventual deletion; Reopening reverses it during the cooling-off window; the Rules spec governs timing and retention scope; the Notification spec confirms closure and, later, deletion | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a single Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-29.SPEC-006 | Session & Device Management | Phase 3 (Entity-Lifecycle) and Phase 4 (Trigger-Response) | The Pro Account CRUD matrix has no spec creating the sign-in identity or the `signed_in_devices` list, and a successful code entry is itself a state-changing trigger (no session exists, then one does) that needed a standalone automation rather than being folded into the Sign-In screen |
| FEAT-29.SPEC-009 | Account Reopening | Phase 3 (Entity-Lifecycle, state transition) and Phase 6 (Negative/Failure, "changes their mind" alternate) | The Primary Flows & Alternates field's reopening alternate is a distinct Closing -> Active state transition with its own cross-feature coordination (booking page stays down until FEAT-27 resume) that the Closure screen alone cannot execute |
| FEAT-29.SPEC-010 | Contact-Detail Change Processing | Phase 4 (Trigger-Response, cross-entity effect) | Changing a sign-in contact invalidates outstanding recovery state and (per the Client dependency map's Contention line) can invalidate client access links tied to the old identity if the mobile number is reused elsewhere -- a cross-entity effect too consequential to leave inline in the Settings screen |
| FEAT-29.SPEC-011 | Sign-In & Recovery Rules | Phase 5 (Rule Discovery) | The Validation & Limits field names four interacting rules (10-minute code expiry, 5-attempt lockout with a 15-minute pause, 30-day session duration, anti-enumeration) shared across the Sign-In and Recovery screens -- past the inline-validation threshold |
| FEAT-29.SPEC-012 | Contact-Change Confirmation Rules | Phase 5 (Rule Discovery) | The dual-confirmation requirement is conditional logic (a change is not committed until both confirmations arrive, and either one can be declined or can expire) shared between the Settings screen and the Change Processing automation |
| FEAT-29.SPEC-013 | Account Closure & Retention Rules | Phase 5 (Rule Discovery) | The Validation & Limits and Data Notes fields name a 30-day cooling-off period, a mandated closure sequence (XBR-20), a retention exception (legally required de-identified financial records), and an export-scope boundary (the Pro's own clients and bookings only) -- four interacting, non-trivial rules shared across three specs |
| FEAT-29.SPEC-015 | New-Device Sign-In Alert | Phase 4 (Notification surfacing) | The Communications field names "an alert when the account is signed in on a new device" as a message with a real audience and delivery rule, not a same-screen confirmation |
| FEAT-29.SPEC-016 | Contact-Change Confirmation Notification | Phase 4 (Notification surfacing) | The Communications field names "confirmations of contact-detail changes to both old and new contacts" -- a two-audience delivery rule requiring a standalone Notification spec |
| FEAT-29.SPEC-017 | Account Closure & Deletion Notifications | Phase 4 (Notification surfacing) | The Communications field names both "an account-closure confirmation" and "a final notice when data is permanently deleted" as messages with real timing rules spanning the 30-day cooling-off period |

## Entity-Lifecycle Coverage Matrix

**Entity: Pro Account** (scope note: per the Requirements Architect's coordination notes, this feature's Create responsibility is limited to establishing the sign-in identity -- sign-in email, sign-in mobile, and the first signed-in device -- which FEAT-15.SPEC-004 then attaches to the new account record it creates. Full account-record creation, profile fields and go-live gating belong to FEAT-15/FEAT-27.)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-29.SPEC-001, FEAT-29.SPEC-006 | Sign-In Screen collects email, mobile and the verified one-time code during onboarding; Session & Device Management establishes the sign-in identity and the first signed-in device that FEAT-15.SPEC-004 then attaches to the new Pro Account record | Do not claim full Pro Account creation -- see scope note above and Coordination Note 1 |
| Read (single) | FEAT-29.SPEC-003 | Settings screen displays the Pro's own account status and signed-in devices; Platform Operator (Support) reads account status view-only through FEAT-19 | Support never sees sign-in codes (SC-05, XBR-24) |
| Read (list) | N/A | Exactly one Pro Account per sign-in identity, so no list view is meaningful | -- |
| Update | FEAT-29.SPEC-003, FEAT-29.SPEC-006, FEAT-29.SPEC-010 | Sign-in contacts changed via SPEC-010; signed-in device list maintained by SPEC-006; both surfaced and initiated from SPEC-003 | This feature updates only sign-in contacts, devices, and closure status; profile, link name, timezone, currency and notification preferences are FEAT-27's (dependency map, Pro Account Lifecycle) |
| Delete/Archive | FEAT-29.SPEC-008 | Hard delete of the Pro's personal data (sign-in email/mobile, devices, profile, private client notes) once the 30-day cooling-off period expires unreversed; restorable in full during the cooling-off window via FEAT-29.SPEC-009; no cascade delete of financial records -- Booking and Deposit Transaction history is retained de-identified as required by law (SC-22); Client contact details are hard-deleted per XBR-19 (FEAT-13) semantics, with de-identified booking/financial history surviving | Retention/purge: after deletion, only legally required de-identified financial records are kept indefinitely; nothing further is purged |
| State Transition | FEAT-29.SPEC-008, FEAT-29.SPEC-009, FEAT-29.SPEC-013 | Active -> Closing (cooling-off) -> Closed (deleted), or Closing -> Active (reopened); the Paused state is FEAT-18/FEAT-27's, not this feature's | Status field is shared with FEAT-18 (subscription lapse pause) and FEAT-27 (Pro-chosen pause) -- this feature owns only Closing/Closed/reopened-to-Active |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-29.SPEC-004, FEAT-29.SPEC-007 | Data export reads the Pro's own client list; export excludes nothing named in Data Notes as a display-only exclusion (see Shared Context for the private-notes disposition) |
| Booking | FEAT-29.SPEC-004, FEAT-29.SPEC-007, FEAT-29.SPEC-005 | Export reads booking history; the Closure screen reads upcoming bookings to show the Pro before hand-off to FEAT-30 |
| Deposit Transaction | FEAT-29.SPEC-004, FEAT-29.SPEC-007 | Export reads deposit outcomes tied to the Pro's bookings |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro requests a one-time code (sign-in, recovery, or onboarding) | Generate and deliver a time-limited code to the requested contact | Standalone Automation, then Standalone Notification | FEAT-29.SPEC-006, FEAT-29.SPEC-014 |
| Pro submits a code | Validate against expiry and attempt-count rules; on success create/refresh a signed-in device | Standalone Logic/Rule, then Standalone Automation | FEAT-29.SPEC-011, FEAT-29.SPEC-006 |
| Code is wrong or expired | Show a generic "that code didn't work" message; count the failed attempt toward the 5-attempt lockout | Inline in triggering screen, governed by Standalone Logic/Rule | FEAT-29.SPEC-001 / SPEC-002, FEAT-29.SPEC-011 |
| Fifth consecutive failed attempt | Pause further attempts for 15 minutes | Standalone Logic/Rule | FEAT-29.SPEC-011 |
| Sign-in succeeds on a device not seen before | Send a new-device alert to the Pro's existing contacts | Standalone Notification | FEAT-29.SPEC-015 |
| Pro chooses "sign out everywhere" | Expire every signed-in device immediately | Standalone Automation | FEAT-29.SPEC-006 |
| A signed-in device reaches 30 days of inactivity | Expire that device silently | Standalone Automation | FEAT-29.SPEC-006 |
| Pro starts a sign-in-contact change | Send a confirmation prompt to both the old and the new contact; commit the change only once both confirm | Standalone Automation, governed by Standalone Logic/Rule | FEAT-29.SPEC-010, FEAT-29.SPEC-012 |
| A contact-detail change commits | Notify both the old and the new contact of the change | Standalone Notification | FEAT-29.SPEC-016 |
| One side of a contact-detail change is never confirmed | Leave the prior contact detail in place; the pending change expires | Standalone Logic/Rule | FEAT-29.SPEC-012 |
| Pro requests a data export | Assemble a spreadsheet-friendly file from the Pro's own Client, Booking and Deposit Transaction records | Standalone Automation | FEAT-29.SPEC-007 |
| Export generation is still running when the Pro checks back | Show an in-progress indicator, then a download action once ready | Inline in triggering screen | FEAT-29.SPEC-004 |
| Pro requests account closure while upcoming bookings exist | Show those bookings and offer one-step cancellation with full refunds before closure proceeds | Cross-feature (bulk cancellation and refunds owned by FEAT-30) | FEAT-29.SPEC-005, hands off to FEAT-30 |
| Pro confirms closure | Cancel the subscription, take the booking page down, start the 30-day cooling-off clock, send the closure confirmation | Standalone Automation, cross-feature (subscription cancel via FEAT-18, booking page takedown via FEAT-05), then Standalone Notification | FEAT-29.SPEC-008, FEAT-29.SPEC-017 |
| Cooling-off period expires with no reopening | Permanently delete personal data; retain only legally required de-identified financial records; send the final deletion notice | Standalone Automation, then Standalone Notification | FEAT-29.SPEC-008, FEAT-29.SPEC-017 |
| Pro signs in during the cooling-off period | Restore the account to Active with everything intact; the booking page stays down until the Pro separately resumes it | Standalone Automation, cross-feature (resume owned by FEAT-27) | FEAT-29.SPEC-009 |
| Pro who is already signed in loses connectivity | Keep read-only access to their last loaded schedule; block any action that requires signing in fresh, paying, or changing account state, with a plain message that a live connection is required | Inline in triggering screen | FEAT-29.SPEC-001, FEAT-29.SPEC-003, FEAT-29.SPEC-005 |
| An unauthenticated visitor reaches any Pro-only screen | Redirect to the Sign-In Screen | Cross-feature (every Pro-facing feature enforces this at its own entry point per XBR-29) | FEAT-29.SPEC-001 |
| Support opens a Pro's account for a help request | Show account status only (Active/Paused/Closing); never sign-in codes or the ability to sign in as the Pro | Inline in triggering screen, governed by Standalone Logic/Rule | FEAT-29.SPEC-003, FEAT-29.SPEC-013 |

## Shared Context

**Shared Entities:**
- Pro Account (sign-in slice) -- sign-in identity and first device created by SPEC-001/SPEC-006; devices maintained by SPEC-006; sign-in contacts updated by SPEC-010; closure/reopening status set by SPEC-008/SPEC-009; displayed by SPEC-003. Fields touched by this feature: sign_in_email, sign_in_mobile, signed_in_devices, status (Closing/Closed/reopened-to-Active slice only).
- Client, Booking, Deposit Transaction -- read only, exclusively by SPEC-004 and SPEC-007 for export, and Booking additionally by SPEC-005 to show upcoming bookings before closure. No spec in this feature writes any of these three entities.

**Shared UI Patterns:**
- Code entry pattern -- the same "enter the code we sent" step, generic failure message, and "send a new code" action appears on SPEC-001 (sign-in), SPEC-002 (recovery), and as the confirmation step inside SPEC-010's contact-change flow (surfaced from SPEC-003). Spec Writers for all three should describe the pattern identically rather than as separate components.
- Device list pattern -- SPEC-003 shows one consistent device-list treatment (device, approximate location/type if available, last-active time, individual sign-out) alongside the single "sign out everywhere" action that SPEC-006 executes.
- Cooling-off status banner -- SPEC-005 shows one consistent Closing-state treatment (days remaining, what stays intact, the reopen action) that SPEC-008 and SPEC-009 keep current.

**Shared Validation:**
- SPEC-011 defines the one-time-code expiry, lockout, session-duration, and anti-enumeration rules once; SPEC-001, SPEC-002, and SPEC-006 all reference it rather than duplicating any of these rules.
- SPEC-012 defines the dual-confirmation rule once; SPEC-003 and SPEC-010 both reference it.
- SPEC-013 defines the cooling-off period, closure sequencing, retention scope, and export scope once; SPEC-004, SPEC-005, SPEC-007, SPEC-008, and SPEC-009 all reference it rather than restating the rules.

**Data Notes disposition (Coordination Note 4):** Stage 2 does not say whether the export includes the Pro's private client notes. This feature resolves it as an explicit scope decision rather than leaving it silent: the export is scoped, per the feature's Validation & Limits field ("the data export covers the Pro's own clients and bookings only") and Data Notes ("generated on request from Client, Booking and Deposit Transaction records"), to the structured fields of those three entities -- name, phone, email, booking history, and deposit outcomes -- and **excludes the Client entity's private_note field**, since that field is described product-wide (Client entity, Data Sensitivity) as "Pro-only" working notes rather than a client or booking record proper, and a spreadsheet-friendly export is not the notes' intended surface. SPEC-013 records this as the authoritative export-scope rule; SPEC-007 implements it.

## Internal Dependency Map

```
SPEC-001 (Sign-In Screen) -> [Pro requests a code] -> SPEC-006 (Session & Device Management) -> [code sent] -> SPEC-014 (Sign-In Code Notification)
SPEC-001 (Sign-In Screen) -> [Pro submits code] -> SPEC-011 (Sign-In & Recovery Rules) -> [valid] -> SPEC-006 (Session & Device Management) -> [new device] -> SPEC-015 (New-Device Sign-In Alert)
SPEC-002 (Account Recovery Screen) -> [Pro requests a code on the surviving contact] -> SPEC-006 (Session & Device Management) -> [code sent] -> SPEC-014 (Sign-In Code Notification)
SPEC-002 (Account Recovery Screen) -> [validates code using] -> SPEC-011 (Sign-In & Recovery Rules)
SPEC-003 (Account & Sign-In Settings Screen) -> [Pro chooses "sign out everywhere"] -> SPEC-006 (Session & Device Management)
SPEC-003 (Account & Sign-In Settings Screen) -> [Pro starts a contact change] -> SPEC-010 (Contact-Detail Change Processing) -> [governed by] -> SPEC-012 (Contact-Change Confirmation Rules) -> [both confirmed] -> SPEC-016 (Contact-Change Confirmation Notification)
SPEC-003 (Account & Sign-In Settings Screen) -> [Pro opens export] -> SPEC-004 (Data Export Screen)
SPEC-003 (Account & Sign-In Settings Screen) -> [Pro opens closure] -> SPEC-005 (Account Closure & Reopening Screen)
SPEC-004 (Data Export Screen) -> [Pro requests export] -> SPEC-007 (Data Export Generation) -> [governed by] -> SPEC-013 (Account Closure & Retention Rules) -> [file ready] -> SPEC-004
SPEC-005 (Account Closure & Reopening Screen) -> [Pro confirms closure] -> SPEC-008 (Account Closure Orchestration) -> [governed by] -> SPEC-013 (Account Closure & Retention Rules) -> [closure confirmed] -> SPEC-017 (Account Closure & Deletion Notifications)
SPEC-008 (Account Closure Orchestration) -> [cooling-off expires unreversed] -> [data deleted] -> SPEC-017 (Account Closure & Deletion Notifications)
SPEC-005 (Account Closure & Reopening Screen) -> [Pro reopens during cooling-off] -> SPEC-009 (Account Reopening) -> [restored] -> SPEC-003 (Account & Sign-In Settings Screen)
```

**Default Entry:** SPEC-001 (Sign-In Screen) -- the screen shown to any signed-out visitor to a Pro-only area, and the onboarding entry point for a brand-new Pro's sign-in creation.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-29.SPEC-001 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Onboarding's account setup step hands the Pro into sign-in creation | Pro starts setup |
| FEAT-29.SPEC-006 | Outbound | FEAT-15 (Pro Onboarding & Setup Wizard) | The established sign-in identity is attached to the new Pro Account record created by FEAT-15.SPEC-004 | Pro completes sign-in during onboarding |
| FEAT-29.SPEC-001 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Default landing after sign-in (FEAT-12.SPEC-001) | Pro signs in / opens the app |
| FEAT-29.SPEC-001 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard), FEAT-01, FEAT-02, FEAT-13, FEAT-15 (SPEC-001/003), FEAT-17, FEAT-27, FEAT-28 (SPEC-001/002), FEAT-30 | Every Pro-facing feature's own dashboard-access-authorization check (e.g., FEAT-12.SPEC-008) sends an unauthenticated visitor here (XBR-29) | Visitor with no valid session reaches a Pro-only screen |
| FEAT-29.SPEC-005 | Outbound | FEAT-30 (Pro Booking Management) | Closure with upcoming bookings hands them to bulk cancellation with full refunds | Pro requests closure while bookings are upcoming |
| FEAT-29.SPEC-008 | Outbound | FEAT-18 (Pro Subscription Billing & Account Management) | Closure invokes subscription cancellation | Pro confirms account closure |
| FEAT-29.SPEC-003 | Outbound | FEAT-18 (Pro Subscription Billing & Account Management) | Pro opens billing from account settings | Pro opens billing |
| FEAT-29.SPEC-008 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Closure takes the booking page down; FEAT-05.SPEC-008 renders "this booking page isn't available" | Pro confirms account closure |
| FEAT-29.SPEC-009 | Outbound | FEAT-27 (Pro Profile & Booking Page Settings) | Reopening restores the account but leaves the booking page down until the Pro separately resumes it through FEAT-27's pause/resume control, rather than this feature defining a second pause mechanism | Pro reopens during the cooling-off period |
| FEAT-29.SPEC-008 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Permanent deletion de-identifies Activity Events per FEAT-16.SPEC-005 | Cooling-off period expires unreversed |
| FEAT-29.SPEC-008 | Outbound | FEAT-13 (Client Record Management) | Client deletion on account closure follows the same hard-delete-with-de-identified-history semantics as XBR-19 | Cooling-off period expires unreversed |
| FEAT-29.SPEC-003 | Outbound | FEAT-19 (Platform Support Read-Only Access) | Support's view-only read of account status (Active/Paused/Closing), never sign-in codes | Support opens a Pro account during a help request |
| FEAT-29.SPEC-014, SPEC-015, SPEC-016, SPEC-017 | Outbound | FEAT-08 (Automated Booking & Messaging) | All four notification specs are sent through the text/email delivery capability (FEAT-08.SPEC-012 text, FEAT-08.SPEC-013 email fallback) | Each notification's trigger fires |
| FEAT-29.SPEC-013 | N/A -- capability dependency owned elsewhere | (ASMP-31, payment-processing capability) | This feature has no Integration spec of its own for payment processing; subscription cancellation runs through FEAT-18.SPEC-006 and refunds through FEAT-30.SPEC-011 | Referenced only for completeness of the Dependencies slice |

## Non-Functional Notes

**Data volumes / growth:** A Pro's export covers up to several years of accumulated Client, Booking and Deposit Transaction history at the product-wide scale of roughly 100-500 clients and 20-40 bookings a week (ASMP-22); export generation must stay reliable at that multi-year volume even though it runs on request rather than continuously.

**Responsiveness:** Signing in with a fresh code is a brief, connectivity-dependent action with an in-place loading indicator while the code is sent or checked (States field); a Pro already signed in keeps read-only access to their last loaded schedule when offline (ASMP-27); data export generation may take longer than an instant read and shows progress rather than a blank wait (States field, Loading).

**Data sensitivity / privacy:** This feature's own fields -- sign-in email, sign-in mobile, and signed-in devices -- are the single most sensitive surface in the product, since they gate access to every client's contact details across the account (Data Notes; dependency map Data Sensitivity for Pro Account); protected by one-time-code sign-in with new-device alerts (ASMP-30); Support sees account status only, never sign-in codes (ASMP-30, XBR-24, SC-05).

**Compliance flags:** Account deletion retains only the financial records the law requires, in de-identified form (Validation & Limits field; SC-22) -- a legal-retention obligation rather than a discretionary choice; no other named compliance regime applies specifically to this feature.

## Non-Goals

- **Support acting on a Pro's sign-in or account** -- Excluded per SC-05: Platform Operator (Support) has view-only access to account status and can never see sign-in codes, sign in as the Pro, or change sign-in details; a Pro who loses both contact methods must regain one to recover -- there is no support-assisted recovery path.
- **Password-based sign-in or a remembered credential** -- Excluded per product-features.md's Key Capabilities ("a one-time code rather than a password to remember") and ASMP-30's account-protection basis: this feature never introduces a password field or a "remember this password" pattern.
- **Deleting the legally required financial history** -- Intentional lifecycle decision surfaced by the CRUD matrix and required by SC-22: permanent deletion never removes the de-identified financial records the law requires to be retained; there is no "delete everything, no exceptions" option.
- **Importing sign-in or account data from a prior tool** -- Excluded per SC-09: this feature exports a Pro's own data outward on request but has no path for bringing in accounts, clients, or bookings from an informal prior process (DMs, paper diaries, spreadsheets).
- **A cross-account or shared sign-in identity** -- Intentional lifecycle decision grounded in the dependency map's Pro Account relationships ("No record is ever shared between two Pro Accounts"): each sign-in identity created by this feature belongs to exactly one Pro Account, with no mechanism for one sign-in to reach more than one account.
