# FEAT-29 — Pro Sign-In & Account Lifecycle

This chapter covers Pro Sign-In & Account Lifecycle (FEAT-29), a Important-tier feature. It carries 17 specifications carrying 204 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-29.SPEC-001 | Sign-In Screen | screen | 13 |
| FEAT-29.SPEC-002 | Account Recovery Screen | screen | 10 |
| FEAT-29.SPEC-003 | Account & Sign-In Settings Screen | screen | 19 |
| FEAT-29.SPEC-004 | Data Export Screen | screen | 12 |
| FEAT-29.SPEC-005 | Account Closure & Reopening Screen | screen | 15 |
| FEAT-29.SPEC-006 | Session & Device Management | automation | 14 |
| FEAT-29.SPEC-007 | Data Export Generation | automation | 12 |
| FEAT-29.SPEC-008 | Account Closure Orchestration | automation | 12 |
| FEAT-29.SPEC-009 | Account Reopening | automation | 9 |
| FEAT-29.SPEC-010 | Contact-Detail Change Processing | automation | 12 |
| FEAT-29.SPEC-011 | Sign-In & Recovery Rules | logic-rule | 12 |
| FEAT-29.SPEC-012 | Contact-Change Confirmation Rules | logic-rule | 11 |
| FEAT-29.SPEC-013 | Account Closure & Retention Rules | logic-rule | 12 |
| FEAT-29.SPEC-014 | Sign-In Code Notification | notification | 10 |
| FEAT-29.SPEC-015 | New-Device Sign-In Alert | notification | 9 |
| FEAT-29.SPEC-016 | Contact-Change Confirmation Notification | notification | 11 |
| FEAT-29.SPEC-017 | Account Closure & Deletion Notifications | notification | 11 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Sign-In Screen

## Overview

**Name:** Sign-In Screen
**ID:** FEAT-29.SPEC-001
**Type:** Screen
**Purpose:** Talia enters her sign-in email or mobile number, requests a one-time code, and enters it to sign in -- and it is the screen every unauthenticated visitor to a Pro-only area lands on (XBR-29).
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Collecting the sign-in identifier (email or mobile number) and requesting a one-time code
- The code-entry step and its generic failure handling
- Serving as the redirect destination for every unauthenticated visitor to a Pro-only screen (XBR-29)
- Serving as the onboarding entry point where a brand-new Pro first establishes their sign-in identity (email, mobile, first device)

**Non-Goals:**
- Recovering access when one contact method is lost -- owned by FEAT-29.SPEC-002 (Account Recovery Screen); this screen assumes the Pro still has access to at least the one contact method they enter here
- Verifying the submitted code and creating the signed-in device -- owned by FEAT-29.SPEC-006 (Session & Device Management); this screen only collects input and displays the outcome
- Password-based sign-in or a "remember this password" pattern -- excluded per product-features.md's Key Capabilities ("a one-time code rather than a password to remember") and ASMP-30
- Full Pro Account profile creation -- owned by FEAT-15.SPEC-004, which attaches this screen's established sign-in identity to the new account record; this screen establishes sign-in identity only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15 (Pro Onboarding & Setup Wizard, account setup step) | A brand-new Pro starts setup | None -- first-time sign-in creation, no existing identifier |
| FEAT-12, FEAT-01, FEAT-02, FEAT-13, FEAT-15 (SPEC-001/003), FEAT-17, FEAT-27, FEAT-28 (SPEC-001/002), FEAT-30, and every other Pro-facing feature's own access-authorization check | A visitor with no valid session reaches a Pro-only screen (XBR-29) | The originally requested destination, so sign-in returns the Pro there rather than always to the dashboard |
| External / direct link | Pro opens the product with no active session (e.g., bookmarked link, new device) | None |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Talia confirms "Sign out everywhere" | None -- every session has ended; a fresh sign-in starts |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter sign-in identifier, request a code, enter the code | -- |
| The Client (Riley) | No | No | Clients never hold a Pro-style sign-in (FEAT-06 gives them a password-free identity instead); a client who reaches this URL directly sees the same generic screen but has no path that leads a signed-in client session here |
| Platform Operator (Support) | No | No | Support has no sign-in of their own on this screen; support access is a separate, internal entry point (FEAT-19) that never uses this Pro sign-in flow |
| Unauthenticated | Yes -- this is the destination screen | Yes -- this is the only action available | This is where every unauthenticated visitor to a Pro-only screen lands (XBR-29); no further redirect |
| Expired session | Yes -- redirected here | Yes | Dialog "Your session has expired. Sign in to continue." appears once on arrival; any in-progress, unsaved work on the screen the Pro was on is not preserved (per that screen's own Edge Cases), except read-only schedule data cached for offline viewing (ASMP-27), which remains available after re-authentication |

## Layout and Content

**Header:** Chairtime wordmark, centered, no navigation controls (this is an entry screen with no "back").

**Body -- Identifier step (default):**
- A single text input labeled "Email or mobile number" (accepts either format, required)
- A primary action button "Send code"
- Below the button, a line of text: "We'll text or email you a one-time code to sign in -- no password needed."

**Body -- Code step (after a code is requested):**
- A confirmation line: "We sent a code to {masked identifier}" (e.g., "t***a@gmail.com" or "(***) ***-1234")
- A single numeric code input, 6 digits
- A primary action button "Sign in"
- A secondary text action "Send a new code"
- A secondary text action "Use a different email or mobile number" (returns to the identifier step)

**Footer:** A single text link, "Lost access to this contact method?", navigating to FEAT-29.SPEC-002 (Account Recovery Screen).

### Responsive Behavior

- **Compact breakpoint:** Single-column, vertically centered content, full-width inputs and buttons.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Identifier input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Identifier input | Blur (empty) | Triggers field validation | Error state on field | "Enter your email or mobile number" below field |
| "Send code" button | Tap | Requests a one-time code via FEAT-29.SPEC-006 (Session & Device Management), which triggers FEAT-29.SPEC-014 (Sign-In Code Notification) | Button shows loading state; on success, screen transitions to the Code step | Screen transitions to the Code step showing the masked identifier |
| Code input | Type | Captures numeric input (6 digits) | Field shows entered digits | Standard input focus state; auto-submits when 6 digits are entered |
| "Sign in" button | Tap | Submits the code for validation via FEAT-29.SPEC-011 (Sign-In & Recovery Rules), then FEAT-29.SPEC-006 (Session & Device Management) on success | Button shows loading state | Success: navigates to FEAT-12.SPEC-001 (Today's Upcoming Schedule) or the originally requested Pro-only screen. Failure: generic error shown per FEAT-29.SPEC-011 |
| "Send a new code" link | Tap | Requests a fresh code, invalidating the previous one, via FEAT-29.SPEC-006 | Code input clears | Confirmation text: "New code sent" |
| "Use a different email or mobile number" link | Tap | Returns to the Identifier step | Screen reverts to Identifier step, code input cleared | Identifier field is empty and focused |
| "Lost access to this contact method?" link | Tap | Navigate to FEAT-29.SPEC-002 (Account Recovery Screen) | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Identifier input -> "Send code" -> (Code step) Code input -> "Sign in" -> "Send a new code" -> "Use a different email or mobile number" -> "Lost access to this contact method?".
- **Validation announcements:** Field errors and the generic "that code didn't work" message are announced to assistive technology and programmatically associated with the relevant field.
- **Transition announcements:** The transition from the Identifier step to the Code step is announced ("Code sent to {masked identifier}"), and the numeric code input receives focus automatically.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures. Code auto-submit on 6 digits does not remove the explicit "Sign in" button as a keyboard-operable alternative.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Identifier (default) | Identifier input empty, "Send code" enabled once non-empty | Screen first opens | Pro submits a valid identifier |
| Requesting code | "Send code" shows loading spinner | Pro taps "Send code" | Code is sent (success) or request fails |
| Code entry | Masked identifier shown, code input focused and empty | Code sent successfully | Pro submits a code, requests a new one, or goes back |
| Validating code | "Sign in" shows loading spinner | Pro submits a 6-digit code | Validation succeeds or fails |
| Error | Generic error message shown per FEAT-29.SPEC-011 ("That code didn't work. Try again or send a new code.") below the code input; code input cleared for re-entry | Code is wrong or expired | Pro re-enters a code or requests a new one |
| Locked | Code input and "Send a new code" disabled; message "Too many attempts. Try again in {remaining minutes} minutes." | Fifth consecutive failed attempt (FEAT-29.SPEC-011) | platform parameter: `sign-in-lockout-pause-minutes` elapses |
| Offline/Degraded | Banner "You're offline -- signing in needs a live connection." at top; both "Send code" and "Sign in" are disabled while offline | Connectivity lost while screen is open | Connectivity restored -- controls re-enable, no request was queued (signing in is never queued, per ASMP-27) |

## Validation Rules

Validation of the code itself (expiry, lockout, anti-enumeration) is governed by FEAT-29.SPEC-011 (Sign-In & Recovery Rules). This screen applies the following inline input validation.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Identifier | Required, non-empty | On blur, on submit | "Enter your email or mobile number" |
| Identifier | Must resemble a valid email or mobile-number format | On submit | "Enter a valid email address or mobile number" |
| Code | Exactly 6 digits | On submit (or auto-submit) | "Enter the 6-digit code" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful sign-in (default landing) | FEAT-12.SPEC-001 (Today's Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |
| Successful sign-in (redirected from a Pro-only screen) | The originally requested screen | Varies -- the feature that redirected here |
| "Lost access to this contact method?" tap | FEAT-29.SPEC-002 (Account Recovery Screen) | -- |

## Data Model

**Creates:** On first-time onboarding use only -- the sign-in identity slice of the Pro Account (sign_in_email, sign_in_mobile) and the first signed_in_devices entry, established by FEAT-29.SPEC-006 once the first code is verified; the full Pro Account record is then created by FEAT-15.SPEC-004 around this identity.
**Reads:** None displayed -- this screen never shows existing account data, only the masked identifier the Pro just entered.
**Updates:** None directly -- code verification and device creation/refresh are performed by FEAT-29.SPEC-006.
**Deletes:** None.

## Business Rules

- Sign-in code behavior (expiry, lockout, anti-enumeration) is governed entirely by FEAT-29.SPEC-011 -- this screen never defines or duplicates those rules.
- A failed sign-in never reveals whether an account exists for the entered identifier (XBR-29, ASMP-30): the same generic "that code didn't work" message and the same code-request behavior appear whether or not the identifier matches an account.
- XBR-29: every Pro-facing screen requires a signed-in Pro; this screen is the universal redirect destination for anyone who is not.
- On successful onboarding-time sign-in creation, FEAT-29.SPEC-006 hands the established identity to FEAT-15.SPEC-004, which creates the Pro Account record around it.

## Edge Cases

- **Pro enters an identifier with no matching account** -- The screen behaves identically to a matching identifier: a code is "sent" (or silently discarded server-side) and the Code step appears normally, so no enumeration signal is given (XBR-29).
- **Pro submits the code twice rapidly** -- Second submission is ignored while the first validation is in progress ("Sign in" in loading state).
- **Pro navigates away mid-code-entry and returns** -- The Code step re-appears with the code input empty; a previously requested code remains valid until it expires (platform parameter: `sign-in-code-expiry-minutes`) or a new one is requested.
- **Pro is redirected here from a deep Pro-only screen and abandons sign-in** -- No further redirect loop; the Pro simply remains on this screen until they sign in or leave the product.
- **Network failure while requesting a code** -- Error banner "Could not send code. Check your connection and try again." with a Retry action; the Identifier step is preserved.
- **No concurrent-edit conflict applies** -- This screen creates no record content of its own (only requests and validates a code); there is nothing here for another actor to have changed concurrently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggers (outbound) | Requesting a code and submitting a code both invoke this automation |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Code expiry, lockout, and anti-enumeration rules applied to this screen's code step |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | Triggers (outbound) | A requested code is delivered through this notification |
| FEAT-29.SPEC-002 (Account Recovery Screen) | Navigation (outbound) | "Lost access to this contact method?" link |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound) | Onboarding's account setup step hands the Pro into sign-in creation here |
| FEAT-12.SPEC-001 (Today's Upcoming Schedule) | Navigation (outbound) | Default landing after sign-in |
| FEAT-12, FEAT-01, FEAT-02, FEAT-13, FEAT-15, FEAT-17, FEAT-27, FEAT-28, FEAT-30 | Navigation (inbound) | Every Pro-facing feature's own access-authorization check redirects an unauthenticated visitor here (XBR-29) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| pro_signed_in | entry context (default / redirected), new_device (yes/no) | Sign-in succeeds | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so sign-in reliability remains observable |
| sign_in_code_failed | reason (wrong_code / expired / locked_out) | A code attempt fails | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so sign-in failure patterns remain observable |
| sign_in_code_requested | request_type (initial / resend) | Pro requests a code | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so code-delivery volume is observable |

## Acceptance Criteria

**FEAT-29.SPEC-001-AC-01:** Given Talia is on the Sign-In Screen's Identifier step, when she enters her sign-in email and taps "Send code", then the screen transitions to the Code step showing "We sent a code to t***a@gmail.com".

**FEAT-29.SPEC-001-AC-02:** Given Talia is on the Identifier step, when she taps "Send code" with the field empty, then the field shows the error "Enter your email or mobile number" and no request is sent.

**FEAT-29.SPEC-001-AC-03:** Given Talia is on the Code step with a valid, unexpired code sent, when she enters the correct 6 digits, then she is signed in and lands on FEAT-12.SPEC-001 (Today's Upcoming Schedule).

**FEAT-29.SPEC-001-AC-04:** Given Talia enters an incorrect code, when she submits it, then the screen shows the generic message "That code didn't work. Try again or send a new code." and the code input clears.

**FEAT-29.SPEC-001-AC-05:** Given Talia has failed 5 consecutive attempts, when she tries to enter another code, then the screen shows "Too many attempts. Try again in 15 minutes." and further submission is disabled for platform parameter: `sign-in-lockout-pause-minutes`.

**FEAT-29.SPEC-001-AC-06:** Given an unauthenticated visitor reaches FEAT-12.SPEC-001 directly, when the dashboard's access check runs, then they are redirected to this Sign-In Screen, and on successful sign-in they land back on FEAT-12.SPEC-001.

**FEAT-29.SPEC-001-AC-07:** Given Talia enters an identifier with no matching account, when she taps "Send code", then the screen behaves identically to a matching identifier (transitions to the Code step) with no indication that the account does not exist.

**FEAT-29.SPEC-001-AC-08:** Given Talia is on the Code step, when she taps "Send a new code", then a fresh code is requested, the previous code becomes invalid, and the code input clears with the confirmation "New code sent".

**FEAT-29.SPEC-001-AC-09:** Given Talia signs in on a device not seen before, when sign-in succeeds, then FEAT-29.SPEC-015 (New-Device Sign-In Alert) is triggered.

**FEAT-29.SPEC-001-AC-10:** Given Talia loses connectivity while on this screen, when she is on either the Identifier or Code step, then the banner "You're offline -- signing in needs a live connection." appears and both action buttons are disabled until connectivity returns.

**FEAT-29.SPEC-001-AC-11:** Given Talia has an expired session while she had unsaved changes on another Pro screen, when she is redirected here, then the dialog "Your session has expired. Sign in to continue." appears once, and after she signs in successfully she is returned to the screen she was on (with unsaved changes handled per that screen's own Edge Cases).

**FEAT-29.SPEC-001-AC-12:** Given Talia taps "Lost access to this contact method?", when the tap registers, then she is navigated to FEAT-29.SPEC-002 (Account Recovery Screen).

**FEAT-29.SPEC-001-AC-13:** Given a brand-new Pro reaches this screen from FEAT-15's account setup step with no prior sign-in identity, when she completes the code step for the first time, then FEAT-29.SPEC-006 establishes her sign-in identity and first device, and FEAT-15.SPEC-004 attaches it to a newly created Pro Account record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 5 (requesting, code entry, error, locked, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Account Recovery Screen

## Overview

**Name:** Account Recovery Screen
**ID:** FEAT-29.SPEC-002
**Type:** Screen
**Purpose:** Talia, having lost access to one of her two sign-in contact methods, regains sign-in through whichever contact method she still controls.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Accepting whichever surviving contact method (email or mobile) the Pro still has
- Requesting and validating a one-time code on that surviving contact, identically to ordinary sign-in
- Returning the Pro to a signed-in state without support involvement

**Non-Goals:**
- Ordinary sign-in when both contact methods are still accessible -- handled by FEAT-29.SPEC-001 (Sign-In Screen); this screen exists only for the lost-access path
- Support-assisted recovery -- excluded per scope-boundaries.md SC-05: support has view-only access and can never see sign-in codes or sign in as the Pro; a Pro who loses both contact methods must regain one to recover, with no alternate path
- Changing a sign-in contact detail -- owned by FEAT-29.SPEC-010 (Contact-Detail Change Processing), reached from FEAT-29.SPEC-003 after the Pro is signed back in; this screen only restores sign-in access, it does not update account data
- Code expiry, lockout, and anti-enumeration rules -- governed by FEAT-29.SPEC-011 (Sign-In & Recovery Rules), which this screen references rather than redefines

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-001 (Sign-In Screen) | Pro taps "Lost access to this contact method?" | None -- recovery starts fresh with no pre-filled identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter the surviving contact method, request and enter a code | -- |
| The Client (Riley) | No | No | Clients never hold a Pro-style sign-in and have no recovery need on this account (FEAT-06 covers client access independently) |
| Platform Operator (Support) | No | No | Recovery is a self-service Pro action only (SC-05); support has no role in it and no view of this screen |
| Unauthenticated | Yes -- this is a pre-sign-in screen | Yes | This screen is itself reachable without a session, by design |
| Expired session | Yes -- reachable via FEAT-29.SPEC-001 | Yes | No special handling beyond the standard expired-session dialog on the Sign-In Screen it is reached from |

## Layout and Content

**Header:** Screen title "Recover your account" with a back arrow (returns to FEAT-29.SPEC-001, Sign-In Screen).

**Body -- Contact step (default):**
- Introductory text: "Enter whichever email or mobile number you still have access to."
- A single text input labeled "Email or mobile number" (required)
- A primary action button "Send code"

**Body -- Code step (after a code is requested):**
- A confirmation line: "We sent a code to {masked identifier}"
- A single numeric code input, 6 digits
- A primary action button "Sign in"
- A secondary text action "Send a new code"

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width inputs and buttons, matching FEAT-29.SPEC-001's layout treatment per the Feature Breakdown Brief's Shared UI Patterns (code entry pattern).
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-29.SPEC-001 (Sign-In Screen) | Screen closes | Standard navigation transition |
| Contact input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Contact input | Blur (empty) | Triggers field validation | Error state on field | "Enter your email or mobile number" below field |
| "Send code" button | Tap | Requests a one-time code on the entered contact via FEAT-29.SPEC-006 (Session & Device Management), which triggers FEAT-29.SPEC-014 (Sign-In Code Notification) | Button shows loading state; on success, screen transitions to the Code step | Screen transitions to the Code step showing the masked identifier |
| Code input | Type | Captures numeric input (6 digits) | Field shows entered digits | Standard input focus state; auto-submits at 6 digits |
| "Sign in" button | Tap | Submits the code for validation via FEAT-29.SPEC-011 (Sign-In & Recovery Rules), then FEAT-29.SPEC-006 on success | Button shows loading state | Success: navigates to FEAT-12.SPEC-001 (Today's Upcoming Schedule). Failure: generic error per FEAT-29.SPEC-011 |
| "Send a new code" link | Tap | Requests a fresh code, invalidating the previous one | Code input clears | Confirmation text: "New code sent" |

### Accessibility Notes

- **Focus order:** Back arrow -> Contact input -> "Send code" -> (Code step) Code input -> "Sign in" -> "Send a new code".
- **Validation announcements:** Field errors and the generic failure message are announced to assistive technology, identically to FEAT-29.SPEC-001's code-entry pattern.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Contact (default) | Contact input empty, "Send code" enabled once non-empty | Screen first opens | Pro submits a valid contact identifier |
| Requesting code | "Send code" shows loading spinner | Pro taps "Send code" | Code is sent or request fails |
| Code entry | Masked identifier shown, code input focused and empty | Code sent successfully | Pro submits a code or requests a new one |
| Validating code | "Sign in" shows loading spinner | Pro submits a 6-digit code | Validation succeeds or fails |
| Error | Generic message "That code didn't work. Try again or send a new code." shown below the code input; code input cleared | Code is wrong or expired | Pro re-enters a code or requests a new one |
| Locked | Code input and "Send a new code" disabled; message "Too many attempts. Try again in {remaining minutes} minutes." | Fifth consecutive failed attempt (FEAT-29.SPEC-011) | platform parameter: `sign-in-lockout-pause-minutes` elapses |
| Offline/Degraded | Banner "You're offline -- recovery needs a live connection." at top; both action buttons disabled | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Code validation (expiry, lockout, anti-enumeration) governed by FEAT-29.SPEC-011 (Sign-In & Recovery Rules). This screen applies the same inline input validation as FEAT-29.SPEC-001.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Contact input | Required, non-empty | On blur, on submit | "Enter your email or mobile number" |
| Contact input | Must resemble a valid email or mobile-number format | On submit | "Enter a valid email address or mobile number" |
| Code | Exactly 6 digits | On submit (or auto-submit) | "Enter the 6-digit code" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-29.SPEC-001 (Sign-In Screen) | -- |
| Successful recovery sign-in | FEAT-12.SPEC-001 (Today's Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |

## Data Model

**Creates:** None directly -- a successful recovery refreshes or creates a signed-in device via FEAT-29.SPEC-006, identically to ordinary sign-in.
**Reads:** None displayed.
**Updates:** None directly -- code verification and device handling performed by FEAT-29.SPEC-006.
**Deletes:** None.

## Business Rules

- Recovery uses the identical code/lockout/anti-enumeration rules as ordinary sign-in (FEAT-29.SPEC-011) -- there is no separate, weaker recovery-specific validation path.
- A recovery attempt on an identifier with no matching account behaves identically to a matching one (XBR-29, ASMP-30): a code step always follows a "Send code" tap.
- SC-05: recovery is entirely self-service; no support-assisted path exists at any point in this flow.
- Recovering access does not, by itself, change any sign-in contact detail -- the Pro's other, still-valid contact remains on the account exactly as before; changing it is a separate action through FEAT-29.SPEC-003 and FEAT-29.SPEC-010.

## Edge Cases

- **Pro enters the contact method that was actually lost, not the surviving one** -- No special detection exists; if that contact is genuinely unreachable, the code never arrives and the Pro cannot complete recovery through it. The Pro returns to the Contact step (via "Send a new code" or re-entry) and tries the other contact method instead.
- **Pro has lost both contact methods** -- No recovery path exists (SC-05, Non-Goal); the Pro cannot complete this screen and remains signed out. This is an accepted product boundary, not a screen defect.
- **Pro submits the code twice rapidly** -- Second submission is ignored while the first validation is in progress.
- **Network failure while requesting a code** -- Error banner "Could not send code. Check your connection and try again." with a Retry action; the Contact step is preserved.
- **Pro navigates away mid-code-entry and returns** -- The Code step re-appears with the code input empty; a previously requested code remains valid until expiry (platform parameter: `sign-in-code-expiry-minutes`) or a new one is requested.
- **No concurrent-edit conflict applies** -- This screen creates no record content of its own; there is nothing here for another actor to have changed concurrently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Sign-In Screen) | Navigation (inbound/outbound) | Entry point via "Lost access" link; back arrow returns there |
| FEAT-29.SPEC-006 (Session & Device Management) | Triggers (outbound) | Requesting and validating the recovery code |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Code expiry, lockout, and anti-enumeration rules |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | Triggers (outbound) | A requested code is delivered through this notification |
| FEAT-12.SPEC-001 (Today's Upcoming Schedule) | Navigation (outbound) | Default landing after successful recovery |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_recovery_started | -- | Pro submits a contact identifier on this screen | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so recovery usage remains observable |
| account_recovery_succeeded | -- | Recovery code validates and the Pro is signed in | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so recovery reliability remains observable |

## Acceptance Criteria

**FEAT-29.SPEC-002-AC-01:** Given Talia has lost her old phone but still has her sign-in email, when she enters her email on this screen and taps "Send code", then the screen transitions to the Code step showing "We sent a code to t***a@gmail.com".

**FEAT-29.SPEC-002-AC-02:** Given Talia is on the Code step with a valid, unexpired recovery code, when she enters the correct 6 digits, then she is signed in and lands on FEAT-12.SPEC-001.

**FEAT-29.SPEC-002-AC-03:** Given Talia enters an incorrect recovery code, when she submits it, then the screen shows "That code didn't work. Try again or send a new code." and the code input clears.

**FEAT-29.SPEC-002-AC-04:** Given Talia has failed 5 consecutive recovery attempts, when she tries to enter another code, then the screen shows "Too many attempts. Try again in 15 minutes." for platform parameter: `sign-in-lockout-pause-minutes`.

**FEAT-29.SPEC-002-AC-05:** Given Talia enters a contact identifier with no matching account, when she taps "Send code", then the screen behaves identically to a matching identifier with no indication the account does not exist.

**FEAT-29.SPEC-002-AC-06:** Given Talia has lost both her sign-in contact methods, when she attempts recovery, then no recovery path completes for her, and no support-assisted alternative is offered anywhere on this screen.

**FEAT-29.SPEC-002-AC-07:** Given Talia taps the back arrow, when the tap registers, then she is navigated to FEAT-29.SPEC-001 (Sign-In Screen).

**FEAT-29.SPEC-002-AC-08:** Given Talia is on the Code step, when she taps "Send a new code", then a fresh code is requested, the previous code becomes invalid, and the confirmation "New code sent" appears.

**FEAT-29.SPEC-002-AC-09:** Given Talia loses connectivity while on this screen, when she is on either step, then the banner "You're offline -- recovery needs a live connection." appears and both action buttons are disabled.

**FEAT-29.SPEC-002-AC-10:** Given Talia completes recovery successfully, when sign-in succeeds, then her other, still-valid sign-in contact remains unchanged on her account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 5 (requesting, code entry, error, locked, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Account & Sign-In Settings Screen

## Overview

**Name:** Account & Sign-In Settings Screen
**ID:** FEAT-29.SPEC-003
**Type:** Screen
**Purpose:** Talia's ongoing hub for her signed-in devices, signing out everywhere, and starting a sign-in-contact change, data export, or account closure; also Support's view-only entry point for account status.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Displaying the Pro's sign-in contacts (masked), signed-in devices, and account status
- Starting a sign-in-contact change (email or mobile), handed to FEAT-29.SPEC-010
- The "sign out everywhere" action
- Navigating to Data Export (FEAT-29.SPEC-004) and Account Closure (FEAT-29.SPEC-005)
- Support's view-only entry point for account status during a help request

**Non-Goals:**
- Editing profile fields (display name, photo, studio address, timezone, currency, notification preferences) -- owned by FEAT-27 (Pro Profile & Booking Page Settings); this screen owns only sign-in and account-lifecycle settings
- Performing the dual-confirmation contact change itself -- owned by FEAT-29.SPEC-010 (Contact-Detail Change Processing); this screen only starts the change and shows its pending state
- Viewing or editing subscription and billing details -- owned by FEAT-18 (Pro Subscription Billing & Account Management), reached via a navigation link from this screen
- Support performing any account action -- excluded per scope-boundaries.md SC-05: support's access here is view-only, status only, and never includes sign-in codes or the ability to sign in as the Pro

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard, navigation) | Pro opens settings | None |
| FEAT-29.SPEC-009 (Account Reopening) | Pro's account is restored to Active during the cooling-off period | Confirmation banner context (account reopened) |
| FEAT-19 (Platform Support Read-Only Access) | Support opens the Pro's account after a help request | Support's view-only session context; no sign-in codes ever included |
| FEAT-29.SPEC-015 (New-Device Sign-In Alert) | Talia taps "Manage sign-in" in a new-device sign-in alert email | None -- opens at the devices section |
| FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) | Talia taps "Manage sign-in" in a contact-change confirmation email | None -- opens at the sign-in contacts section |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | All actions: start a contact change, sign out everywhere, sign out one device, open export, open closure | -- |
| The Client (Riley) | No | No | Clients never reach a Pro settings screen; there is no client-facing entry point into it |
| Platform Operator (Support) | Account status only (Active / Paused / Closing) and the account activity log entry for their own view (XBR-24); never sign-in contacts, codes, or devices | No actions -- entirely view-only (Access Matrix: Profile & Account Settings = View, scoped to status) | Any attempt to act (there are no actionable controls rendered for Support) is prevented because the screen renders none for this role; Support sees a status-only reduced layout, never the full Pro layout |
| Unauthenticated | No | No | Redirected to FEAT-29.SPEC-001 (Sign-In Screen) per XBR-29; after signing in, the Pro lands back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress contact-change entry is not preserved; the Pro restarts the change after re-authentication |

## Layout and Content

**Header:** Screen title "Account & Sign-In" with a back arrow (returns to FEAT-12).

**Body, section 1 -- Sign-in contacts (Pro view only):**
- "Email" row showing the masked sign-in email, with an "Edit" action
- "Mobile number" row showing the masked sign-in mobile, with an "Edit" action
- Each Edit action starts a contact change via FEAT-29.SPEC-010; while a change is pending, the row expands into the pending-confirmation area, showing two code-entry steps side by side -- the Shared UI Pattern's code entry, identical in form to FEAT-29.SPEC-001 (sign-in) and FEAT-29.SPEC-002 (recovery):
  - **Confirm from your current {field}:** "We sent a code to {masked old value}", a 6-digit numeric code input, a primary "Confirm" action, and a secondary "Send a new code" action
  - **Confirm from your new {field}:** "We sent a code to {masked new value}", a 6-digit numeric code input, a primary "Confirm" action, and a secondary "Send a new code" action
  - Both steps are shown together, not sequentially, since the Pro typically holds both contacts and may enter either code first; a step that has been confirmed shows a checkmark and "Confirmed" in place of its inputs
  - A wrong or expired code on either step shows the generic message "That code didn't work. Try again or send a new code." under that step, matching FEAT-29.SPEC-001/SPEC-002's code-entry pattern exactly
  - Once both steps show Confirmed, the row updates to the new masked value on next load

**Body, section 2 -- Signed-in devices (Pro view only):**
- A list of signed-in devices, one row per device: device description, approximate location/type if available, last-active time, and an individual "Sign out" action per device
- Below the list, a single "Sign out everywhere" action

**Body, section 3 -- Data and account (Pro view only):**
- "Download my data" row navigating to FEAT-29.SPEC-004 (Data Export Screen)
- "Close account" row navigating to FEAT-29.SPEC-005 (Account Closure & Reopening Screen)
- "Billing & subscription" row navigating to FEAT-18 (Pro Subscription Billing & Account Management)

**Support view (reduced layout, Platform Operator only):**
- A single status line: "Account status: {Active / Paused / Closing}." No sign-in contacts, devices, or actionable controls are rendered.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order above, full width.
- **Medium size class and above:** Sections remain stacked, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Email "Edit" | Tap (Pro only) | Starts a contact change via FEAT-29.SPEC-010, governed by FEAT-29.SPEC-012 | Inline entry field appears for the new email; on submission the row expands into the two code-entry steps | Each step shows "We sent a code to {masked identifier}" |
| Mobile "Edit" | Tap (Pro only) | Starts a contact change via FEAT-29.SPEC-010, governed by FEAT-29.SPEC-012 | Inline entry field appears for the new mobile number; on submission the row expands into the two code-entry steps | Each step shows "We sent a code to {masked identifier}" |
| Pending-confirmation code input (old or new side) | Type | Captures numeric input (6 digits) | That step's field shows entered digits | Standard input focus state; auto-submits at 6 digits |
| "Confirm" button (old or new side) | Tap | Submits that side's code for validation via FEAT-29.SPEC-012, through FEAT-29.SPEC-010 | That step's button shows loading state | Success: that step shows a checkmark and "Confirmed"; once both steps show Confirmed, the row updates to the new masked value on next load. Failure: "That code didn't work. Try again or send a new code." shown under that step; that step's code input clears |
| "Send a new code" link (old or new side) | Tap | Requests a fresh code for that side only, invalidating the previous one, via FEAT-29.SPEC-010 | That side's code input clears; its failed-attempt count resets | Confirmation text: "New code sent" |
| Individual device "Sign out" | Tap (Pro only) | Expires that one signed-in device via FEAT-29.SPEC-006 | Device row is removed from the list | Toast: "Signed out of {device description}" |
| "Sign out everywhere" | Tap (Pro only) | Expires every signed-in device via FEAT-29.SPEC-006, including the current one | Every device row is removed | Confirmation dialog before the action, then the Pro is signed out and redirected to FEAT-29.SPEC-001 |
| "Download my data" row | Tap (Pro only) | Navigate to FEAT-29.SPEC-004 (Data Export Screen) | Screen closes | Standard navigation transition |
| "Close account" row | Tap (Pro only) | Navigate to FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Screen closes | Standard navigation transition |
| "Billing & subscription" row | Tap (Pro only) | Navigate to FEAT-18 (Pro Subscription Billing & Account Management) | Screen closes | Standard navigation transition |
| Back arrow | Tap | Navigate to FEAT-12 | Screen closes | Standard navigation transition |

### Accessibility Notes

- **Focus order (Pro view):** Back arrow -> Email Edit -> Mobile Edit -> (while pending) old-side code input -> old-side Confirm -> old-side "Send a new code" -> new-side code input -> new-side Confirm -> new-side "Send a new code" -> device list (each device row, then its Sign out action) -> Sign out everywhere -> Download my data -> Close account -> Billing & subscription.
- **Dynamic-change announcements:** Each code-entry step's appearance, its "Confirmed" state, the generic failure message, the lockout message, and "Signed out of {device}" are announced to assistive technology when they appear.
- **Code-entry transition announcement:** When a contact change starts, the transition from the Edit field to the two code-entry steps is announced ("Codes sent to {masked old value} and {masked new value}"), matching FEAT-29.SPEC-001's transition-announcement pattern.
- **Sign out everywhere confirmation:** The confirmation dialog is a focus-trapping modal announced on open, with its two options reachable by keyboard.
- **Support view:** The reduced status-only layout has its own simple focus order: back arrow -> status line; no interactive controls follow.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (Pro) | Full three-section layout with current data | Screen opens for the Pro | Pro navigates away |
| Loaded (Support) | Reduced status-only layout | Screen opens for Support | Support navigates away |
| Loading | Section placeholders shown while account and device data load | Screen first opens | Data finishes loading |
| Contact-change code entry (per side) | That side's step shows the masked identifier and an empty code input, awaiting confirmation | A contact change starts (FEAT-29.SPEC-010) and that side has not yet entered a correct code | Correct code entered (step shows Confirmed) or that side locks |
| Contact-change code error (per side) | Generic message "That code didn't work. Try again or send a new code." shown under that step; that step's code input cleared | A submitted code for that side is wrong or expired | Pro re-enters a code for that side or requests a new one |
| Contact-change code locked (per side) | That step's code input and "Send a new code" disabled; message "Too many attempts. Try again in {remaining minutes} minutes." | Fifth consecutive failed attempt on that side (FEAT-29.SPEC-012) | platform parameter: `contact-change-code-lockout-pause-minutes` elapses |
| Contact change pending (overall) | Both code-entry steps visible, at least one not yet Confirmed | A contact change is started (FEAT-29.SPEC-010) | Both sides confirm (row updates to the new value) or the overall confirmation window expires (row reverts, FEAT-29.SPEC-012) |
| Error | Banner "Couldn't load your account settings. Try again." with retry | Data load fails | Pro taps Retry and load succeeds |
| Offline/Degraded | Existing loaded data remains visible read-only; all action controls (Edit, Sign out, Sign out everywhere, navigation into export/closure) are disabled with the inline note "Requires a live connection" | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Validation of the contact-change entry (format of the new email/mobile) and of each side's confirmation code (correctness, expiry, lockout) is governed by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules). This screen applies the following inline input validation on top of that.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Pending-confirmation code (old or new side) | Exactly 6 digits | On submission (or auto-submit) | "Enter the 6-digit code" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's Upcoming Schedule) | FEAT-12 (Pro Daily Schedule Dashboard) |
| "Download my data" tap | FEAT-29.SPEC-004 (Data Export Screen) | -- |
| "Close account" tap | FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | -- |
| "Billing & subscription" tap | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | FEAT-18 (Pro Subscription Billing & Account Management) |
| "Sign out everywhere" confirmed | FEAT-29.SPEC-001 (Sign-In Screen) | -- |

## Data Model

**Creates:** None.
**Reads:** Pro Account -- sign_in_email, sign_in_mobile (masked), signed_in_devices, status (for the Support view). Fields displayed match the dependency map's Pro Account entity exactly.
**Updates:** Pro Account.signed_in_devices (device list changes on individual/everywhere sign-out, via FEAT-29.SPEC-006); Pro Account.sign_in_email / sign_in_mobile (only once a contact change fully commits, via FEAT-29.SPEC-010 -- this screen never writes the field directly).
**Deletes:** None.

## Business Rules

- Support's view is limited to account status only (Active/Paused/Closing); sign-in codes, contacts, and devices are never rendered for this role (XBR-24, ASMP-30).
- Every Support view of this screen is logged in the Pro's visible account activity (XBR-24) -- the Pro can see when and that support looked, through the activity record (FEAT-16).
- A contact change is never committed by this screen directly -- it only starts the change (FEAT-29.SPEC-010) and renders its two code-entry steps and their pending/confirmed/locked/committed state, governed by FEAT-29.SPEC-012.
- Each code-entry step's failure and lockout behavior is independent per side -- a locked or repeatedly wrong code on one side never blocks entry on the other side.
- "Sign out everywhere" (FEAT-29.SPEC-006) always requires an explicit confirmation dialog before it executes, since it also signs out the Pro's current session.

## Edge Cases

- **Pro signs out an individual device while viewing this screen from that same device** -- The current device can be signed out individually like any other; doing so signs the Pro out of the current session immediately and redirects to FEAT-29.SPEC-001, identical in effect to "Sign out everywhere" for that one session.
- **A pending contact change's overall confirmation window expires while the Pro is viewing this screen** -- The row silently reverts from the code-entry steps to the prior value on next data refresh, per FEAT-29.SPEC-012's expiry rule; no error is shown, since an expired pending change is a normal outcome, not a failure.
- **The old-side code locks from repeated wrong entries while the new side has already confirmed** -- The new side's Confirmed state persists; once the lockout pause elapses, the Pro can retry the old side without the new side needing to reconfirm, since each side's state is independent (FEAT-29.SPEC-012).
- **Support opens this screen while the Pro is simultaneously viewing it** -- No conflict: Support's view is read-only and status-only, so nothing the Pro sees or does is affected; the Pro's next activity-record check shows the Support view logged (XBR-24).
- **Pro taps "Sign out everywhere" twice rapidly** -- Second tap is ignored while the first request is in progress (confirmation dialog and subsequent action are not re-entrant).
- **Pro reopens the account during the cooling-off period and lands here** -- The screen reflects the restored Active status immediately (FEAT-29.SPEC-009); the "Close account" row still leads to FEAT-29.SPEC-005, which shows no more pending closure.
- **Concurrent-edit conflict -- device list changed on another signed-in device while this screen is open** -- The device list is a live-updating snapshot, not stale-write-prone (sign-out actions are idempotent per device); if a device this Pro is trying to sign out was already signed out from elsewhere, the action is a no-op and the row is simply already absent on next refresh -- no error shown.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggers (outbound) | Individual sign-out and sign-out-everywhere actions |
| FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Triggers (outbound) | Starting a contact change |
| FEAT-29.SPEC-012 (Contact-Change Confirmation Rules) | References (inbound) | Governs the pending-state display and expiry |
| FEAT-29.SPEC-004 (Data Export Screen) | Navigation (outbound) | "Download my data" |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Navigation (outbound) | "Close account" |
| FEAT-29.SPEC-009 (Account Reopening) | Navigation (inbound) | Landing here after a reopening restores the account |
| FEAT-18 (Pro Subscription Billing & Account Management) | Navigation (outbound) | "Billing & subscription" |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's view-only entry point |
| FEAT-12.SPEC-001 (Today's Upcoming Schedule) | Navigation (inbound/outbound) | Entry from and return to the dashboard |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| signed_out_everywhere | device_count | Pro confirms "Sign out everywhere" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |
| contact_change_started | field (email / mobile) | Pro submits a new value for a contact field | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so contact-change initiation remains observable |

## Acceptance Criteria

**FEAT-29.SPEC-003-AC-01:** Given Talia opens Account & Sign-In from the dashboard, when the screen loads, then she sees her masked email and mobile, her signed-in devices, and the data/account section.

**FEAT-29.SPEC-003-AC-02:** Given Talia taps "Edit" next to her email, when she submits a new email, then FEAT-29.SPEC-010 starts the change and the row expands into two code-entry steps, each showing "We sent a code to {masked identifier}".

**FEAT-29.SPEC-003-AC-03:** Given Talia's old-email code-entry step is showing, when she enters the correct 6-digit code sent to her old email, then that step shows a checkmark and "Confirmed".

**FEAT-29.SPEC-003-AC-04:** Given Talia enters an incorrect code on her new-email step, when she submits it, then that step shows "That code didn't work. Try again or send a new code." and that step's code input clears.

**FEAT-29.SPEC-003-AC-05:** Given Talia's new-email step has failed 5 consecutive code attempts, when she tries another code on that step, then it shows "Too many attempts. Try again in {remaining minutes} minutes." while her old-email step remains available.

**FEAT-29.SPEC-003-AC-06:** Given Talia is on her old-email code-entry step, when she taps "Send a new code", then a fresh code is requested for that side only, that side's input clears with the confirmation "New code sent", and her new-email step is unaffected.

**FEAT-29.SPEC-003-AC-07:** Given both of Talia's code-entry steps show Confirmed, when she next loads this screen, then the email row shows her new masked email in place of the code-entry steps.

**FEAT-29.SPEC-003-AC-08:** Given Talia taps "Sign out" on one listed device, when the confirmation is not required for a single device, then that device is removed from the list and the toast "Signed out of {device description}" appears.

**FEAT-29.SPEC-003-AC-09:** Given Talia taps "Sign out everywhere", when she confirms in the dialog, then every device is signed out, including her current session, and she is redirected to FEAT-29.SPEC-001.

**FEAT-29.SPEC-003-AC-10:** Given Talia taps "Sign out everywhere", when the confirmation dialog appears, then choosing "Cancel" leaves every device signed in unchanged.

**FEAT-29.SPEC-003-AC-11:** Given Talia taps "Download my data", when the tap registers, then she is navigated to FEAT-29.SPEC-004 (Data Export Screen).

**FEAT-29.SPEC-003-AC-12:** Given Talia taps "Close account", when the tap registers, then she is navigated to FEAT-29.SPEC-005 (Account Closure & Reopening Screen).

**FEAT-29.SPEC-003-AC-13:** Given Platform Operator (Support) opens this screen during a help request, when the screen loads, then only the account status line is shown, with no sign-in contacts, codes, or devices rendered, and no actionable controls present.

**FEAT-29.SPEC-003-AC-14:** Given Platform Operator (Support) views this screen, when the view completes, then it is logged in the Pro's visible account activity per XBR-24.

**FEAT-29.SPEC-003-AC-15:** Given an unauthenticated visitor reaches this screen's URL directly, when the access check runs, then they are redirected to FEAT-29.SPEC-001 (Sign-In Screen).

**FEAT-29.SPEC-003-AC-16:** Given Talia has a contact change pending confirmation, when its overall confirmation window expires (FEAT-29.SPEC-012) before she returns to this screen, then the row reverts to the prior value with no error shown.

**FEAT-29.SPEC-003-AC-17:** Given Talia loses connectivity while viewing this screen, when the connection drops, then her already-loaded data remains visible read-only and every action control shows "Requires a live connection".

**FEAT-29.SPEC-003-AC-18:** Given Talia's account was reopened during the cooling-off period (FEAT-29.SPEC-009), when she lands on this screen afterward, then the account status reflects Active immediately.

**FEAT-29.SPEC-003-AC-19:** Given Talia taps "Billing & subscription", when the tap registers, then she is navigated to FEAT-18.SPEC-002 (Billing & Subscription Management Screen).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 11 | 11 |
| States | 7 (loading, code entry, code error, code locked, contact pending, error, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Screen Spec: Data Export Screen

## Overview

**Name:** Data Export Screen
**ID:** FEAT-29.SPEC-004
**Type:** Screen
**Purpose:** Talia requests and downloads a spreadsheet-friendly file of her own clients and booking history.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Requesting a data export and showing generation progress
- Downloading the generated file once ready
- Explaining exactly what the export covers

**Non-Goals:**
- Assembling the export file's contents -- owned by FEAT-29.SPEC-007 (Data Export Generation); this screen only requests it and surfaces its state
- Defining the export's scope (which fields, which entities) -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this screen references for its explanatory copy rather than redefining
- Importing data from any source -- excluded per scope-boundaries.md SC-09: the product has no import path for prior-tool data; this screen exports outward only
- Exporting the Pro's private client notes -- excluded per the Feature Breakdown Brief's Data Notes disposition: private_note is Pro-only working notes, not a client or booking record proper, and is deliberately out of the export's scope

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Download my data" | None |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Request and download the export | -- |
| The Client (Riley) | No | No | Clients never reach a Pro settings screen |
| Platform Operator (Support) | No | No | Support's view-only access to account status does not extend to the Pro's own data export; support never sees or triggers a Pro's export |
| Unauthenticated | No | No | Redirected to FEAT-29.SPEC-001 (Sign-In Screen) per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress export request already submitted continues generating server-side and remains available on the next visit; the screen itself must be reopened after re-authentication |

## Layout and Content

**Header:** Screen title "Download my data" with a back arrow (returns to FEAT-29.SPEC-003).

**Body:**
- On open, the screen briefly checks for an existing or in-progress export before showing any action (see States: Loading)
- Explanatory text: "This includes your clients (name, phone, email) and booking history (appointments, deposits, and outcomes) as a spreadsheet-friendly file. It does not include your private client notes."
- A single primary action button "Request export"
- Below the button (once a request has been made), progress or download state per the States section

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width text and button.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Screen closes | Standard navigation transition |
| "Request export" button | Tap | Triggers FEAT-29.SPEC-007 (Data Export Generation) | Button replaced by an in-progress indicator | "Preparing your file..." shown |
| "Download" button (once ready) | Tap | Downloads the generated file to the Pro's device | None -- file download begins | Standard file-download feedback |
| "Try again" button (on generation failure) | Tap | Re-triggers FEAT-29.SPEC-007's export generation | Returns to Generating state | "Preparing your file..." shown |
| "Try again" button (on status-fetch failure) | Tap | Re-fetches current export status from FEAT-29.SPEC-007 | Returns to Loading state | "Checking for an in-progress export..." shown |

### Accessibility Notes

- **Focus order:** Back arrow -> (Loading has no actionable control besides the back arrow) -> "Request export" (or "Download" / "Try again", whichever is current) .
- **Progress announcements:** The transition from Loading to the resolved state (Ready to request, Generating, or Ready to download), from "Request export" to the in-progress indicator, and from in-progress to "Download" or the failure message, is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Placeholder content with the text "Checking for an in-progress export..." | Screen first opens (every visit, including a return after navigating away) | The status fetch from FEAT-29.SPEC-007 resolves to Generating (an export is already in flight), Ready to download (a completed export exists and has not been superseded), Ready to request (no export exists), or Error (the fetch itself fails) |
| Ready to request (default) | "Request export" button enabled | Loading resolves with no existing export | Pro taps "Request export" |
| Generating | In-progress indicator with the text "Preparing your file... this can take a few minutes for a large amount of history." | Pro taps "Request export"; or Loading resolves with an export already in flight | FEAT-29.SPEC-007 completes (success or failure) |
| Ready to download | "Download" button shown with the file's generation timestamp | Export generation completes successfully; or Loading resolves with a completed export already available | Pro downloads the file, or requests a new export (replacing the previous one) |
| Error | Banner "Couldn't prepare your export. Try again." with a "Try again" action (export generation failure); or banner "Couldn't check your export status. Try again." with a "Try again" action (status-fetch failure) | Export generation fails; or the Loading status fetch fails | Pro taps "Try again" (retries the operation that failed -- generation or the status fetch) |
| Offline/Degraded | Existing "Ready to download" state (if reached before disconnecting) remains available for download if already fully downloaded locally; "Request export" and "Try again" are disabled with the inline note "Requires a live connection" | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Export scope (which fields and entities are included) is governed by FEAT-29.SPEC-013 (Account Closure & Retention Rules). This screen has no user-input fields to validate -- it is a single-action request/download flow.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | -- |

## Data Model

**Creates:** None persisted by this screen -- the export file itself is a derived, on-request artifact created by FEAT-29.SPEC-007.
**Reads:** Client, Booking, Deposit Transaction (via FEAT-29.SPEC-007) -- scoped to the Pro's own records only, per the dependency map's Client/Booking/Deposit Transaction relationships (each belongs to exactly one Pro Account). Current export status (via FEAT-29.SPEC-007), read once on every screen open to resolve the Loading state.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The export covers only the Pro's own clients and bookings (FEAT-29.SPEC-013) -- structured fields of Client, Booking, and Deposit Transaction, excluding Client.private_note.
- Requesting a new export while a previous file exists supersedes it -- the screen always reflects only the most recently completed export.
- Export generation is non-blocking to the rest of the product: the Pro can navigate away while generation runs and return later to find it ready (FEAT-29.SPEC-007).
- The screen never assumes no export exists: every open (first visit or a return) checks current export status with FEAT-29.SPEC-007 before showing "Request export", so an in-progress or completed export from an earlier visit is always reflected rather than overwritten by a fresh default.

## Edge Cases

- **Export generation is still running when the Pro checks back after navigating away** -- Reopening the screen enters the Loading state, which fetches current export status from FEAT-29.SPEC-007 and finds the same in-progress request; the screen enters the Generating state for that request and picks it up; no duplicate request is started.
- **Pro requests a second export while the first is still generating** -- The second request is ignored while one is in flight; the screen stays in the Generating state for the original request.
- **Pro has no bookings or clients yet** -- The export still generates successfully, producing a file with headers only and no data rows; the Download state is reached normally, with no special empty-state messaging beyond the file itself being empty of rows.
- **Network failure during download (file already generated)** -- The browser's standard failed-download handling applies; the "Download" button remains available for a retry, since the generated file persists server-side until superseded by a new request.
- **The on-open status fetch itself fails (e.g., the Pro opens the screen while briefly offline or the read errors)** -- The screen shows the Error state with the message "Couldn't check your export status. Try again."; tapping "Try again" re-fetches the status rather than assuming any particular prior state.
- **No concurrent-edit conflict applies** -- This screen has no editable record; there is nothing here for another actor to have changed concurrently. A concurrent write to the Pro's own Client or Booking records elsewhere (e.g., a new booking arriving mid-generation) is reflected only in the next requested export, never retroactively into one already generating.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (inbound) | Entry point |
| FEAT-29.SPEC-007 (Data Export Generation) | Reads (outbound) | On every screen open, fetches current export status to resolve the Loading state |
| FEAT-29.SPEC-007 (Data Export Generation) | Triggers (outbound) | "Request export" starts assembly of the file |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Defines the export's scope |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| data_export_requested | -- | Pro taps "Request export" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |
| data_export_downloaded | file_generation_duration_bucket | Pro taps "Download" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |

## Acceptance Criteria

**FEAT-29.SPEC-004-AC-01:** Given Talia is on this screen, when she taps "Request export", then the screen shows "Preparing your file..." and FEAT-29.SPEC-007 begins assembling it.

**FEAT-29.SPEC-004-AC-02:** Given Talia's export finishes generating successfully, when the screen updates, then a "Download" button appears with the generation timestamp.

**FEAT-29.SPEC-004-AC-03:** Given Talia taps "Download", when the file is ready, then the file downloads to her device.

**FEAT-29.SPEC-004-AC-04:** Given Talia's export generation fails, when the failure is reported, then the banner "Couldn't prepare your export. Try again." appears with a "Try again" action.

**FEAT-29.SPEC-004-AC-05:** Given Talia has requested an export and navigates away before it finishes, when she returns to this screen, then it shows the same in-progress Generating state rather than starting a new request.

**FEAT-29.SPEC-004-AC-06:** Given Talia already has a completed export ready to download, when she taps "Request export" again, then a new export supersedes the old one and the screen returns to the Generating state.

**FEAT-29.SPEC-004-AC-07:** Given Talia has no bookings or clients yet, when she requests an export, then generation completes successfully and produces a file with no data rows.

**FEAT-29.SPEC-004-AC-08:** Given Talia's export is complete, when she inspects what it contains, then it never includes any client's private_note field.

**FEAT-29.SPEC-004-AC-09:** Given Talia loses connectivity while on this screen, when the connection drops, then "Request export" and "Try again" are disabled with the note "Requires a live connection".

**FEAT-29.SPEC-004-AC-10:** Given Talia taps the back arrow, when the tap registers, then she is navigated to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen).

**FEAT-29.SPEC-004-AC-11:** Given Talia opens this screen, when it first loads, then it shows "Checking for an in-progress export..." while fetching current export status from FEAT-29.SPEC-007, then resolves to Ready to request, Generating, or Ready to download based on what that status reports -- with no export request started merely by opening the screen.

**FEAT-29.SPEC-004-AC-12:** Given the export-status fetch fails when Talia opens this screen, when the failure is reported, then the banner "Couldn't check your export status. Try again." appears with a "Try again" action, and tapping it re-fetches the status.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loading, generating, ready to download, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Account Closure & Reopening Screen

## Overview

**Name:** Account Closure & Reopening Screen
**ID:** FEAT-29.SPEC-005
**Type:** Screen
**Purpose:** Talia reviews upcoming bookings and confirms closure, or -- during the cooling-off period -- reopens her account with everything intact.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Showing upcoming bookings and offering one-step cancellation with full refunds before closure can proceed
- Confirming account closure and showing the resulting cooling-off state
- Reopening the account during the cooling-off period

**Non-Goals:**
- Executing the cancellation, subscription cancellation, booking-page takedown, and deletion sequencing -- owned by FEAT-29.SPEC-008 (Account Closure Orchestration); this screen only confirms the Pro's intent and displays resulting state
- Executing the bulk cancellation and refunds themselves -- owned by FEAT-30 (Pro Booking Management); this screen hands off to it and shows the resulting booking count, it does not perform the cancellation
- Resuming the booking page after reopening -- owned by FEAT-27 (Pro Profile & Booking Page Settings); reopening restores the account only, the Pro separately resumes bookings through FEAT-27
- Defining the cooling-off period length and retention scope -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this screen references for its explanatory copy rather than redefining

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Close account" | None -- fresh review of current upcoming bookings |
| FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) | Talia taps "Manage account" in an account closure/deletion notice email | None -- fresh review of the account's closure state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Confirm closure (with the cancel-upcoming-bookings step if applicable), or reopen during cooling-off | -- |
| The Client (Riley) | No | No | Clients never reach a Pro settings screen; a client whose booking is cancelled by this flow is notified separately (FEAT-08) but never sees this screen |
| Platform Operator (Support) | No | No | Support's view-only access to account status (FEAT-29.SPEC-003) shows "Closing" as a status value, but never this screen's confirmation flow or upcoming-bookings detail |
| Unauthenticated | No | No | Redirected to FEAT-29.SPEC-001 (Sign-In Screen) per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress closure confirmation (not yet submitted) is discarded; the Pro restarts the review after re-authentication |

## Layout and Content

**Header:** Screen title "Close account" (Active/Paused state) or "Your account is closing" (Closing state), with a back arrow (returns to FEAT-29.SPEC-003).

**Body -- Active/Paused state (pre-closure):**
- If upcoming bookings exist: a list summarizing them (count and next appointment date), with the text "Closing your account cancels these {N} upcoming bookings and refunds every deposit in full." and a single "Cancel bookings and continue" action
- If no upcoming bookings exist: the review step is skipped and the explanatory text and confirmation control (below) render directly
- Explanatory text: "Closing your account cancels your subscription, takes your booking page down, and deletes your data after a {cooling-off period} cooling-off period, during which you can sign back in to reopen it with everything intact."
- A single destructive-styled action "Close my account"

**Body -- Closing state (during cooling-off):**
- A status banner: "{Days remaining} days left in your cooling-off period. Your subscription is cancelled and your booking page is down. Everything else is intact."
- A single primary action "Reopen my account"

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width text and buttons.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Screen closes | Standard navigation transition |
| "Cancel bookings and continue" (when upcoming bookings exist) | Tap | Hands off to FEAT-30's bulk cancellation with full refunds for every upcoming booking | Upcoming-bookings list is replaced by the explanatory text and "Close my account" action | Confirmation text: "{N} bookings cancelled and refunded" |
| "Close my account" | Tap | Confirmation dialog, then triggers FEAT-29.SPEC-008 (Account Closure Orchestration) | Screen transitions to the Closing state | Confirmation dialog: "This starts your {cooling-off period} cooling-off period. Continue?" with "Continue" and "Keep my account" options |
| "Reopen my account" | Tap | Triggers FEAT-29.SPEC-009 (Account Reopening) | Screen transitions back to the Active/Paused state | Confirmation text: "Your account is reopened" then navigation to FEAT-29.SPEC-003 |
| "Retry" (data-load failure) | Tap | Re-fetches upcoming bookings and Pro Account status | Screen re-attempts the Loading state | Success: screen renders the current state; failure: the Error banner reappears |
| "Retry" (closure-start failure) | Tap | Re-submits the closure request to FEAT-29.SPEC-008 (Account Closure Orchestration) | Button shows loading state | Success: screen transitions to the Closing state; failure: the Error banner reappears with the same message |

### Accessibility Notes

- **Focus order (pre-closure with bookings):** Back arrow -> upcoming-bookings summary -> "Cancel bookings and continue" -> (after) "Close my account".
- **Focus order (Closing state):** Back arrow -> status banner -> "Reopen my account".
- **Confirmation dialog:** The closure confirmation dialog is a focus-trapping modal announced on open, with both options keyboard-reachable.
- **Transition announcements:** The transition from pre-closure to Closing state, and from Closing to reopened, is announced to assistive technology.
- **Error announcement:** The Error banner (data-load failure or closure-start failure) is announced to assistive technology when it appears, and its Retry action receives focus.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Placeholder content shown while upcoming bookings and Pro Account status are fetched | Screen first opens | Data finishes loading (Reviewing/Ready to confirm/Closing, per current status) or the fetch fails (Error) |
| Reviewing upcoming bookings | Bookings list and "Cancel bookings and continue" shown | Loading completes with the account Active/Paused and upcoming bookings existing | Pro completes the cancel-and-continue step |
| Ready to confirm | Explanatory text and "Close my account" shown | Loading completes with no upcoming bookings, or the cancel-and-continue step completes | Pro confirms closure |
| Closing (cooling-off) | Status banner with days remaining and "Reopen my account" shown | Loading completes with the account already Closing, or closure is confirmed (FEAT-29.SPEC-008 starts the cooling-off clock) | Pro reopens, or the cooling-off period expires and the account is deleted (screen is no longer reachable -- see Edge Cases) |
| Error | Banner "Couldn't load your account status and upcoming bookings. Try again." with a Retry action (initial fetch failure); or, after confirming closure, banner "Couldn't close your account. Try again." with a Retry action (FEAT-29.SPEC-008 closure-start failure) | The initial data fetch fails, or FEAT-29.SPEC-008 reports a closure-start failure | Pro taps Retry and it succeeds (screen renders the resulting state); a repeated failure re-shows the same Error banner |
| Offline/Degraded | Existing loaded state remains visible read-only; all action controls are disabled with the inline note "Requires a live connection" | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Cooling-off period length and closure sequencing are governed by FEAT-29.SPEC-013 (Account Closure & Retention Rules). This screen has no free-text input to validate -- every action is a confirmed, discrete choice.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | -- |
| "Cancel bookings and continue" tap | FEAT-30 bulk cancellation flow, then back to this screen's Ready to confirm state | FEAT-30 (Pro Booking Management) |
| Reopening completes | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | -- |

## Data Model

**Creates:** None directly.
**Reads:** Booking -- upcoming bookings for this Pro Account (state, service, start_time), to show the pre-closure review list. Pro Account -- status, to determine which state (pre-closure or Closing) to render.
**Updates:** Pro Account.status (Active/Paused -> Closing on confirmation, Closing -> Active on reopening) -- executed by FEAT-29.SPEC-008 and FEAT-29.SPEC-009 respectively; this screen triggers the transition, it does not write the field directly.
**Deletes:** None directly -- permanent deletion is executed by FEAT-29.SPEC-008 once the cooling-off period expires unreversed.

## Business Rules

- Closure with upcoming bookings requires the one-step cancel-and-continue action first (XBR-20) -- "Close my account" is not shown until the bookings list is empty or has been explicitly cleared through that action.
- Every deposit for a booking cancelled through this flow is refunded in full (FEAT-30, XBR-09) -- there is no partial-refund or forfeiture path here regardless of the cancellation policy's ordinary window.
- Reopening restores the account to Active with everything intact except the booking page, which stays down until the Pro separately resumes it through FEAT-27 (per the Feature Breakdown Brief's Cross-Feature Touchpoints).
- The cooling-off period length and what survives deletion are governed entirely by FEAT-29.SPEC-013 -- this screen only displays the resulting state.
- A closure-start failure reported by FEAT-29.SPEC-008 never advances Pro Account.status -- the account remains fully Active/Paused, and this screen's Error state offers Retry rather than any partial-closure display.

## Edge Cases

- **A new booking arrives between opening this screen and confirming closure** -- The upcoming-bookings review is re-evaluated at confirmation time; if a new booking appeared, the Pro is shown the updated list and must repeat "Cancel bookings and continue" for the newly added booking before "Close my account" proceeds.
- **The cooling-off period expires while the Pro is not signed in** -- Data deletion (FEAT-29.SPEC-008) proceeds without this screen being open; the Pro's next sign-in attempt after deletion is treated as a new account, since no account record survives (Non-Goal: no reopening path exists after permanent deletion).
- **Pro taps "Close my account" twice rapidly** -- Second tap is ignored while the confirmation dialog or the closure request is in progress.
- **Pro taps "Reopen my account" twice rapidly** -- Second tap is ignored while the reopening request is in progress.
- **Support views account status while the Pro is on this screen mid-flow** -- No conflict: Support's view is read-only status only; the Pro's own confirmation flow is unaffected.
- **Concurrent-edit conflict -- Pro Account status changed by an automated process (e.g., FEAT-18's subscription-lapse pause) while this screen is open** -- Resolution: reject-with-refresh, per the dependency map's Pro Account Contention note; the screen re-fetches the current status before executing "Close my account" or "Reopen my account" and shows the current state if it changed.
- **The initial data fetch fails repeatedly** -- Each Retry re-attempts the same fetch; there is no retry-count limit or lockout on this screen's Retry action, since fetching read-only review data carries no risk of a duplicate side effect.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (inbound) | Entry point |
| FEAT-30 (Pro Booking Management) | Triggers (outbound) | "Cancel bookings and continue" hands off bulk cancellation with full refunds |
| FEAT-29.SPEC-008 (Account Closure Orchestration) | Triggers (outbound) | "Close my account" starts the closure sequence |
| FEAT-29.SPEC-008 (Account Closure Orchestration) | Affects (inbound) | A closure-start failure reported by FEAT-29.SPEC-008 drives this screen's Error state and Retry action |
| FEAT-29.SPEC-009 (Account Reopening) | Triggers (outbound) | "Reopen my account" restores the account |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Cooling-off period and retention scope shown in explanatory copy |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (outbound, implied) | Resuming the booking page after reopening happens through FEAT-27, not this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_closure_requested | had_upcoming_bookings (yes/no), cancelled_booking_count | Pro confirms "Close my account" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |
| account_reopened | days_remaining_at_reopen | Pro confirms "Reopen my account" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |

## Acceptance Criteria

**FEAT-29.SPEC-005-AC-01:** Given Talia has 3 upcoming bookings and opens this screen, when it loads, then she sees those 3 bookings summarized with the text explaining they will be cancelled and refunded, and no "Close my account" action yet.

**FEAT-29.SPEC-005-AC-02:** Given Talia sees her 3 upcoming bookings, when she taps "Cancel bookings and continue", then FEAT-30 cancels and fully refunds all 3, the confirmation "3 bookings cancelled and refunded" appears, and "Close my account" becomes available.

**FEAT-29.SPEC-005-AC-03:** Given Talia has no upcoming bookings, when she opens this screen, then it shows the explanatory text and "Close my account" directly, with no review step.

**FEAT-29.SPEC-005-AC-04:** Given Talia taps "Close my account", when the confirmation dialog appears, then choosing "Continue" starts FEAT-29.SPEC-008 and the screen transitions to the Closing state; choosing "Keep my account" leaves her account unchanged.

**FEAT-29.SPEC-005-AC-05:** Given Talia's account is in the Closing state with 12 days remaining, when she opens this screen, then the banner shows "12 days left in your cooling-off period..." and "Reopen my account" is available.

**FEAT-29.SPEC-005-AC-06:** Given Talia's account is Closing, when she taps "Reopen my account", then FEAT-29.SPEC-009 restores her account to Active, the confirmation "Your account is reopened" appears, and she is navigated to FEAT-29.SPEC-003.

**FEAT-29.SPEC-005-AC-07:** Given Talia's account is reopened, when she checks her booking page, then it remains down until she separately resumes it through FEAT-27.

**FEAT-29.SPEC-005-AC-08:** Given a new booking arrives after Talia opened this screen but before she confirms closure, when she taps "Close my account", then the updated bookings list is shown and she must clear it via "Cancel bookings and continue" before closure proceeds.

**FEAT-29.SPEC-005-AC-09:** Given Talia's cooling-off period has already expired and her data was permanently deleted, when she attempts to sign in again, then no reopening path exists for the deleted account.

**FEAT-29.SPEC-005-AC-10:** Given Talia loses connectivity while on this screen, when the connection drops, then all action controls show "Requires a live connection" and are disabled.

**FEAT-29.SPEC-005-AC-11:** Given Talia taps "Close my account" twice rapidly, when the first tap is already processing, then the second tap has no additional effect.

**FEAT-29.SPEC-005-AC-12:** Given Talia's Pro Account status was changed to Paused by FEAT-18's subscription-lapse automation while this screen was open, when she taps "Close my account", then the screen re-fetches the current status before proceeding and reflects it if it changed.

**FEAT-29.SPEC-005-AC-13:** Given Talia taps the back arrow, when the tap registers, then she is navigated to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen).

**FEAT-29.SPEC-005-AC-14:** Given the initial fetch of Talia's upcoming bookings and account status fails, when the screen attempts to load, then the banner "Couldn't load your account status and upcoming bookings. Try again." appears with a Retry action, and tapping Retry that succeeds renders the correct state (Reviewing, Ready to confirm, or Closing).

**FEAT-29.SPEC-005-AC-15:** Given Talia confirms closure and FEAT-29.SPEC-008 reports a closure-start failure, when the failure is received, then this screen shows "Couldn't close your account. Try again." with a Retry action, her Pro Account.status remains unchanged, and tapping Retry re-submits the closure request.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (loading, reviewing/ready to confirm, closing, error, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Session & Device Management

## Overview

**Name:** Session & Device Management
**ID:** FEAT-29.SPEC-006
**Type:** Automation
**Purpose:** Requests and verifies one-time sign-in codes, creates or refreshes a signed-in device on successful verification, expires devices after inactivity, and executes "sign out everywhere."
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Generating and delivering a one-time code on request (sign-in, recovery, or onboarding)
- Verifying a submitted code against the rules FEAT-29.SPEC-011 defines
- Creating the sign-in identity and first signed-in device on a brand-new Pro's first successful onboarding code entry
- Creating or refreshing a signed-in device on every successful verification thereafter
- Expiring a device after 30 days of inactivity (platform parameter: `session-inactivity-expiry-days`)
- Executing "sign out everywhere"

**Non-Goals:**
- Defining code expiry, the failed-attempt lockout, session duration, or the anti-enumeration rule -- owned by FEAT-29.SPEC-011 (Sign-In & Recovery Rules); this automation enforces those rules, it does not define them
- Creating the full Pro Account record -- owned by FEAT-15.SPEC-004, which attaches the sign-in identity this automation establishes to the new account record
- Delivering the code content itself -- owned by FEAT-29.SPEC-014 (Sign-In Code Notification); this automation only triggers that delivery
- Alerting the Pro to a new-device sign-in -- owned by FEAT-29.SPEC-015 (New-Device Sign-In Alert); this automation only detects the new-device condition and triggers that notification

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests a one-time code (sign-in) | FEAT-29.SPEC-001 (Sign-In Screen) | Pro submits an identifier and taps "Send code" | Entered identifier (email or mobile format) |
| Pro requests a one-time code (recovery) | FEAT-29.SPEC-002 (Account Recovery Screen) | Pro submits a surviving contact identifier | Entered identifier |
| Pro requests a one-time code (onboarding) | FEAT-29.SPEC-001 (Sign-In Screen, onboarding entry) | Brand-new Pro establishing sign-in for the first time | Entered identifier, no existing account reference |
| Pro submits a code | FEAT-29.SPEC-001 or FEAT-29.SPEC-002 | Pro enters 6 digits and submits (or auto-submits) | Submitted code, the identifier the code was requested for, device fingerprint of the requesting browser/device |
| Pro chooses "sign out everywhere" | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro confirms the "Sign out everywhere" dialog | Pro Account reference |
| Pro signs out an individual device | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Sign out" on one device row | Pro Account reference, the specific device identifier |
| Scheduled inactivity check | System | Runs on a recurring schedule to find signed-in devices past the inactivity limit | Every signed_in_devices entry across all Pro Accounts, with each entry's last-active timestamp |

## Processing Logic

**Code request path:**
1. Receive the identifier (email or mobile) submitted from the triggering screen.
2. Look up whether the identifier matches an existing Pro Account's sign_in_email or sign_in_mobile. Regardless of the result, proceed identically (per FEAT-29.SPEC-011's anti-enumeration rule).
3. Generate a new one-time code, superseding any previously issued, unexpired code for this identifier.
4. Record the code's issue time (for the expiry check on submission) against the identifier.
5. Trigger FEAT-29.SPEC-014 (Sign-In Code Notification) to deliver the code to the entered contact.
6. Signal the triggering screen that a code was sent (identical signal whether or not a matching account exists).

**Code submission path:**
1. Receive the submitted code, the identifier it was requested for, and the requesting device's fingerprint.
2. Evaluate the submission against FEAT-29.SPEC-011's rules: expiry, remaining lockout attempts.
3. If invalid (wrong or expired): increment the failed-attempt counter for this identifier and signal the generic failure back to the triggering screen. If this increment reaches the lockout threshold (FEAT-29.SPEC-011), begin the lockout pause.
4. If valid and the identifier matches an existing Pro Account: check whether the requesting device fingerprint matches an existing entry in signed_in_devices.
   - If it matches an existing entry: refresh that entry's last-active timestamp.
   - If it does not match any existing entry: create a new signed_in_devices entry (device description, approximate location/type if available, creation timestamp as last-active) and trigger FEAT-29.SPEC-015 (New-Device Sign-In Alert).
5. If valid and no Pro Account matches (brand-new onboarding sign-in): establish the sign-in identity (sign_in_email and/or sign_in_mobile, whichever was entered) and create the first signed_in_devices entry; hand this identity off to FEAT-15.SPEC-004 to attach to the new Pro Account record it creates. No new-device alert is sent for this first device (there is no prior device to alert from, and no established account yet to receive it on).
6. Reset the failed-attempt counter for this identifier on any successful verification.
7. Signal the triggering screen that sign-in succeeded.

**Sign-out path:**
1. Receive the sign-out request (individual device or everywhere) and the Pro Account reference.
2. For "sign out everywhere": remove every entry from signed_in_devices, including the requesting session's own entry.
3. For an individual device: remove only the specified entry from signed_in_devices.
4. Signal the triggering screen (and, for "everywhere," every other active session) that the affected device(s) are signed out.

**Scheduled inactivity path:**
1. On each scheduled run, read every signed_in_devices entry across all Pro Accounts.
2. For each entry, compare its last-active timestamp against the current time.
3. If the gap exceeds platform parameter: `session-inactivity-expiry-days`, remove that entry silently.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Code sent | Code request processed, regardless of account match | New code recorded against the identifier | Screen transitions to the Code step | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-014 |
| Code sent, request fails | Notification capability cannot be reached | No code state change persists as delivered | Error banner "Could not send code. Check your connection and try again." | FEAT-29.SPEC-001, FEAT-29.SPEC-002 |
| Verification succeeds, existing device | Correct, unexpired code; fingerprint matches an existing entry | signed_in_devices entry's last-active refreshed | Pro is signed in and navigated onward | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-12.SPEC-001 |
| Verification succeeds, new device | Correct, unexpired code; fingerprint matches no existing entry | New signed_in_devices entry created | Pro is signed in and navigated onward; FEAT-29.SPEC-015 fires separately | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-015 |
| Verification succeeds, first-time onboarding | Correct, unexpired code; no existing Pro Account for this identifier | Sign-in identity and first device established; handed to FEAT-15.SPEC-004 | Pro is signed in and continues onboarding | FEAT-15.SPEC-004 |
| Verification fails (wrong/expired) | Code does not match or has passed expiry | Failed-attempt counter incremented | Generic message: "That code didn't work. Try again or send a new code." | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-011 |
| Verification locked out | Failed-attempt counter reaches the lockout threshold | Lockout pause begins for this identifier | "Too many attempts. Try again in {remaining minutes} minutes." | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-011 |
| Individual device signed out | Pro confirms sign-out on one device row | That signed_in_devices entry removed | Device row disappears; toast confirms | FEAT-29.SPEC-003 |
| Signed out everywhere | Pro confirms "Sign out everywhere" | Every signed_in_devices entry removed | Every session (including the current one) is redirected to FEAT-29.SPEC-001 | FEAT-29.SPEC-003, FEAT-29.SPEC-001 |
| Device expired by inactivity | Scheduled check finds an entry past the inactivity limit | That entry removed | No direct user feedback at the moment of expiry; the device simply no longer appears in the list and requires a fresh sign-in next time it is used | FEAT-29.SPEC-003 |
| Automation failure (verification path) | Processing error during verification | No sign-in state changes | Error banner "Something went wrong. Try again." on the triggering screen; the Pro is not signed in | FEAT-29.SPEC-001, FEAT-29.SPEC-002 |

## Data Model

**Reads:** Pro Account -- sign_in_email, sign_in_mobile, signed_in_devices (for lookup and matching during code request and verification).
**Creates:** Pro Account's signed_in_devices entries (on new-device verification, and the first entry on onboarding); the sign-in identity slice (sign_in_email / sign_in_mobile) on first-time onboarding verification.
**Updates:** Pro Account.signed_in_devices (last-active refresh on existing-device verification; entry removal on sign-out or inactivity expiry).
**Deletes:** Pro Account.signed_in_devices entries (individual sign-out, sign-out-everywhere, inactivity expiry).

## Business Rules

- XBR-29: every Pro-facing screen requires a signed-in Pro; this automation is the sole mechanism that establishes that signed-in state.
- Code expiry, lockout threshold and pause duration, and the anti-enumeration guarantee are all defined by FEAT-29.SPEC-011 and enforced here without exception.
- A new-device sign-in always triggers FEAT-29.SPEC-015, except for the very first device created during onboarding (there is no prior device or established account to alert).
- "Sign out everywhere" always includes the requesting session's own device -- there is no way to sign out every device except the current one.
- Device inactivity expiry (platform parameter: `session-inactivity-expiry-days`) is silent -- it produces no notification, since it is a routine housekeeping outcome, not a security event.

## Edge Cases

- **Two devices submit the correct code for the same identifier at nearly the same time (e.g., a code shared or intercepted)** -- Both verifications succeed independently against the same valid code (the code itself is not single-use once issued, only time-limited); each creates or refreshes its own device entry. A resulting unexpected device is visible to the Pro on FEAT-29.SPEC-003 and can be signed out individually.
- **Pro requests a new code while a previous one is still valid** -- The new code supersedes the previous one; the previous code becomes invalid immediately, so a Pro who has an old code page open and a new one open only succeeds with the latest.
- **Concurrent trigger firing -- two code requests for the same identifier fire at effectively the same time (e.g., "Send code" tapped, then "Send a new code" tapped in quick succession before the first delivery completes)** -- Each request generates its own code and supersedes the previous; only the most recently generated code validates. Both notification sends are attempted independently; a duplicate arriving is a minor inconvenience, never a validation error.
- **Trigger fires while a previous run is in flight -- a code submission is in progress when the scheduled inactivity check runs against the same Pro Account** -- The inactivity check operates only on last-active timestamps unaffected by an in-flight verification; the verification's device-entry write (refresh or creation) and the inactivity check's entry removal cannot target the same entry in a way that conflicts, since a device actively completing verification has just updated its last-active timestamp and so cannot simultaneously qualify as inactive.
- **Scheduled inactivity check runs while the Pro is actively using a device that is, by clock skew, borderline past the limit** -- Any activity (a successful verification, which the check does not target, or an in-product action that refreshes last-active) updates the timestamp before the check can remove the entry; the check only ever removes entries with no qualifying activity at all in the window.
- **A device is removed by "sign out everywhere" while its scheduled inactivity check is also about to run** -- The sign-out removal is immediate and idempotent; the scheduled check finds no matching entry and takes no further action for that device.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Sign-In Screen) | Triggered by (inbound) | Code request and code submission |
| FEAT-29.SPEC-002 (Account Recovery Screen) | Triggered by (inbound) | Code request and code submission for recovery |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Triggered by (inbound) | Individual sign-out and sign-out-everywhere |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Code expiry, lockout, anti-enumeration rules enforced here |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | Affects (outbound) | Triggered to deliver every requested code |
| FEAT-29.SPEC-015 (New-Device Sign-In Alert) | Affects (outbound) | Triggered when a new device is created (post-onboarding) |
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Affects (outbound) | Receives the established sign-in identity for a brand-new Pro Account |

## Analytics and Success Signals

- **sign_in_code_requested** (path: sign_in / recovery / onboarding) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so code-delivery volume is observable
- **pro_signed_in** (new_device: yes/no, path: sign_in / recovery / onboarding) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so sign-in success remains observable
- **sign_in_code_failed** (reason: wrong_code / expired / locked_out) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so failure patterns remain observable
- **signed_out_everywhere** (device_count) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **device_expired_inactivity** (-- no user-facing properties, a silent housekeeping event) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so inactivity-expiry volume remains observable for operational monitoring

## Acceptance Criteria

**FEAT-29.SPEC-006-AC-01:** Given Talia submits her sign-in email on FEAT-29.SPEC-001, when the request is processed, then a new code is generated and FEAT-29.SPEC-014 is triggered to deliver it, regardless of whether an account matches.

**FEAT-29.SPEC-006-AC-02:** Given Talia submits the correct, unexpired code from a device that already has a signed_in_devices entry, when verification runs, then that entry's last-active timestamp is refreshed and no new-device alert fires.

**FEAT-29.SPEC-006-AC-03:** Given Talia submits the correct, unexpired code from a device with no existing entry, when verification runs, then a new signed_in_devices entry is created and FEAT-29.SPEC-015 (New-Device Sign-In Alert) is triggered.

**FEAT-29.SPEC-006-AC-04:** Given a brand-new Pro submits the correct code for the first time during onboarding, when verification runs, then her sign-in identity and first device are established and handed to FEAT-15.SPEC-004, with no new-device alert sent.

**FEAT-29.SPEC-006-AC-05:** Given Talia submits an incorrect code, when verification runs, then her failed-attempt counter increments and the generic failure message is returned.

**FEAT-29.SPEC-006-AC-06:** Given Talia's failed-attempt counter reaches the lockout threshold (FEAT-29.SPEC-011), when she attempts another submission, then the lockout pause begins and further attempts are blocked for platform parameter: `sign-in-lockout-pause-minutes`.

**FEAT-29.SPEC-006-AC-07:** Given Talia taps "Sign out" on one device, when the request is processed, then only that device's entry is removed from signed_in_devices.

**FEAT-29.SPEC-006-AC-08:** Given Talia confirms "Sign out everywhere", when the request is processed, then every signed_in_devices entry is removed, including her current session's.

**FEAT-29.SPEC-006-AC-09:** Given a signed_in_devices entry has had no activity for longer than platform parameter: `session-inactivity-expiry-days`, when the scheduled inactivity check runs, then that entry is removed silently with no notification.

**FEAT-29.SPEC-006-AC-10:** Given a signed_in_devices entry had activity within platform parameter: `session-inactivity-expiry-days`, when the scheduled inactivity check runs, then that entry is left unchanged.

**FEAT-29.SPEC-006-AC-11:** Given Talia requests a new code while a previously issued code for the same identifier is still unexpired, when the new code is generated, then the previous code becomes invalid immediately.

**FEAT-29.SPEC-006-AC-12:** Given two devices submit the same valid, unexpired code for the same identifier at nearly the same time, when both verifications process, then both succeed and each creates or refreshes its own device entry independently.

**FEAT-29.SPEC-006-AC-13:** Given the verification path encounters a processing error, when the failure occurs, then the triggering screen shows "Something went wrong. Try again." and the Pro is not signed in.

**FEAT-29.SPEC-006-AC-14:** Given a device is removed via "Sign out everywhere" at the same time its scheduled inactivity check would otherwise run, when the scheduled check executes, then it finds no matching entry and takes no further action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 7 | 7 |
| Outcome Paths | 11 | 11 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Data Export Generation

## Overview

**Name:** Data Export Generation
**ID:** FEAT-29.SPEC-007
**Type:** Automation
**Purpose:** Assembles the requested spreadsheet-friendly file from the Pro's own Client, Booking, and Deposit Transaction records on request.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Reading the Pro's own Client, Booking, and Deposit Transaction records
- Assembling a spreadsheet-friendly file scoped exactly per FEAT-29.SPEC-013's export-scope rule
- Reporting generation progress and completion (or failure) back to the requesting screen
- Reporting the current export status (no export, in progress, or ready) on request, without starting a new generation

**Non-Goals:**
- Requesting the export and presenting the download -- owned by FEAT-29.SPEC-004 (Data Export Screen); this automation only assembles the file
- Defining what the export includes or excludes -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules); this automation implements that scope, it does not decide it
- Importing any data -- excluded per scope-boundaries.md SC-09: no import capability exists in the product definition
- Including the Client entity's private_note field -- excluded per the Feature Breakdown Brief's Data Notes disposition: private_note is Pro-only working notes, not a client or booking record proper

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests an export | FEAT-29.SPEC-004 (Data Export Screen) | Pro taps "Request export" with no other export currently generating for this Pro Account | Pro Account reference |
| Screen requests current export status | FEAT-29.SPEC-004 (Data Export Screen) | The screen opens, on every visit (first open or a return after navigating away) | Pro Account reference |

## Processing Logic

**Generation path:**
1. Receive the export request with the requesting Pro Account's reference.
2. Read all Client records belonging to this Pro Account: name, phone, email, booking_history reference (private_note is never read for this purpose).
3. Read all Booking records belonging to this Pro Account: service, start_time, duration, price_agreed, deposit_amount, state, cancellation/reschedule timestamps, source.
4. Read all Deposit Transaction records tied to those Bookings: amount, currency, status, outcome_reason, timestamps.
5. Assemble the three record sets into a single spreadsheet-friendly file, with one sheet or section per entity, using the field names above as column headers.
6. Report the file as ready for download, along with its generation timestamp.

**Status-check path:**
1. Receive the status request with the requesting Pro Account's reference. This path never reads Client, Booking, or Deposit Transaction data and never starts a new generation.
2. Determine the Pro Account's current export state: a generation currently in progress, a previously completed file that has not been superseded, or no export on record.
3. Report that state back to the requesting screen (plus the generation timestamp, when a completed file exists).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Export ready | Assembly completes successfully | The generated file is stored, superseding any previously generated file for this Pro Account | FEAT-29.SPEC-004 shows the "Download" action with the generation timestamp | FEAT-29.SPEC-004 |
| Export empty (no data) | Pro has zero Clients and zero Bookings | A file with headers only and no data rows is stored | FEAT-29.SPEC-004 shows the "Download" action normally -- no special empty-state messaging | FEAT-29.SPEC-004 |
| Export failure | Assembly encounters a processing error | No new file replaces the previous one, if any | FEAT-29.SPEC-004 shows "Couldn't prepare your export. Try again." with a retry action | FEAT-29.SPEC-004 |
| Status reported | Screen requests current export status (any state: none, in progress, or a completed file) | None -- this is a read-only report | FEAT-29.SPEC-004 routes its Loading state to Ready to request, Generating, or Ready to download, matching the reported state exactly | FEAT-29.SPEC-004 |
| Status check failed | The status query itself fails (e.g., a data-store read error) | None -- no generation is started as a side effect of a failed status check | FEAT-29.SPEC-004 shows "Couldn't check your export status. Try again." with a retry action | FEAT-29.SPEC-004 |

## Data Model

**Reads:** Client (name, phone, email -- excluding private_note), Booking (service, start_time, duration, price_agreed, deposit_amount, state, cancellation/reschedule timestamps, source), Deposit Transaction (amount, currency, status, outcome_reason, timestamps) -- all scoped to the requesting Pro Account, per the dependency map's entity relationships (each belongs to exactly one Pro Account). The status-check path reads only this Pro Account's export-file record (in-progress/completed/none plus generation timestamp), never the Client/Booking/Deposit Transaction data itself.
**Creates:** The generated export file, associated with the requesting Pro Account and a generation timestamp.
**Updates:** None to the source entities -- this automation never modifies Client, Booking, or Deposit Transaction data.
**Deletes:** The previously generated export file for this Pro Account, when a new one supersedes it.

## Business Rules

- Export scope is fixed to structured fields of Client, Booking, and Deposit Transaction, excluding Client.private_note (FEAT-29.SPEC-013).
- The export never includes any other Pro Account's data -- scoping is by Pro Account reference on every read, with no cross-account query path (dependency map: "No record is ever shared between two Pro Accounts").
- Generation is asynchronous relative to the requesting screen: the Pro may navigate away and the export continues; only one export generates at a time per Pro Account.
- A newly requested export always supersedes a previously completed one -- there is no archive of past exports.
- The status-check path is read-only and idempotent -- it never creates, modifies, or deletes the export file, and never starts a generation; only the "Pro requests an export" trigger can start one.

## Edge Cases

- **Pro has years of accumulated history at the upper end of the stated scale (roughly 100-500 clients, several years of bookings)** -- Generation completes reliably at this volume (ASMP-22), showing progress rather than a blank wait for however long assembly takes.
- **A booking is created or changed while generation is running** -- The in-progress export reflects the data as read at the moment each entity was read; a change arriving mid-generation is not guaranteed to appear in that same file and is captured only by a subsequently requested export.
- **Concurrent trigger firing -- the Pro requests an export from two open sessions (e.g., two signed-in devices) at nearly the same time** -- Only one generation runs per Pro Account; the second request is treated as a no-op while the first is in flight (per FEAT-29.SPEC-004's Edge Cases), so no duplicate generation occurs and no conflicting file is produced.
- **Trigger fires while a previous run is in flight** -- A second request for the same Pro Account while generation is already running does not start a new run; it is ignored until the in-flight run completes, at which point a genuinely new request may be made.
- **Generation fails partway through (e.g., a read of one entity succeeds, another fails)** -- The partial result is discarded entirely; no partial file is ever presented as ready. The failure outcome applies and the previous completed file (if any) remains available for download until a new request succeeds.
- **The screen requests status while a generation is mid-flight** -- The status check reports "in progress" without interfering with or restarting that generation; the status-check path never mutates the export-file record.
- **The screen requests status repeatedly (e.g., the Pro reopens the screen several times in a row)** -- Each status request is independently read-only and idempotent; repeated checks never start a generation and never affect one already running.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-004 (Data Export Screen) | Triggered by (inbound) | "Request export" starts this automation |
| FEAT-29.SPEC-004 (Data Export Screen) | Triggered by (inbound) | On every screen open, the status-check path reports current export status to resolve the screen's Loading state |
| FEAT-29.SPEC-004 (Data Export Screen) | Affects (outbound) | Reports progress, ready-to-download, and failure states |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Defines the export's scope, which this automation implements |

## Analytics and Success Signals

- **data_export_generation_completed** (record_counts: client_count, booking_count) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so export reliability remains observable
- **data_export_generation_failed** (-- no properties beyond the failure itself) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so export failures remain observable rather than silent

## Acceptance Criteria

**FEAT-29.SPEC-007-AC-01:** Given Talia requests an export with existing clients and bookings, when generation completes, then a file is produced containing her Client, Booking, and Deposit Transaction records, scoped to her own Pro Account.

**FEAT-29.SPEC-007-AC-02:** Given Talia's export is generated, when its contents are inspected, then no client's private_note field appears anywhere in the file.

**FEAT-29.SPEC-007-AC-03:** Given Talia has zero clients and zero bookings, when she requests an export, then generation completes successfully producing a file with headers only and no data rows.

**FEAT-29.SPEC-007-AC-04:** Given Talia has several years of accumulated history at the upper end of the product's stated scale, when she requests an export, then generation completes reliably, showing progress rather than an indefinite blank wait.

**FEAT-29.SPEC-007-AC-05:** Given a new booking is created while a previously requested export is still generating, when generation completes, then the new booking is not guaranteed to appear in that file, and a subsequently requested export captures it.

**FEAT-29.SPEC-007-AC-06:** Given Talia requests a second export from another signed-in device while the first is still generating, when the second request arrives, then it is ignored until the first completes -- no duplicate generation runs.

**FEAT-29.SPEC-007-AC-07:** Given a processing error occurs partway through assembly, when the failure is detected, then no partial file is produced and FEAT-29.SPEC-004 shows the failure state with a retry action.

**FEAT-29.SPEC-007-AC-08:** Given Talia already has a completed export file, when she requests a new one that completes successfully, then the new file supersedes and replaces the previous one.

**FEAT-29.SPEC-007-AC-09:** Given Talia's export reads only her own Pro Account's records, when generation runs, then no other Pro's Client, Booking, or Deposit Transaction data is included under any condition.

**FEAT-29.SPEC-007-AC-10:** Given Talia's export completes, when FEAT-29.SPEC-004 checks its state, then it reports the generation timestamp alongside the ready-to-download state.

**FEAT-29.SPEC-007-AC-11:** Given Talia opens the Data Export Screen when no export has ever been requested, when the screen's status check runs, then this automation reports no export exists, no generation is started, and FEAT-29.SPEC-004 resolves to Ready to request.

**FEAT-29.SPEC-007-AC-12:** Given a status check for Talia's Pro Account fails (e.g., a data-store read error), when FEAT-29.SPEC-004 requests current status, then this automation reports the status-check failure, starts no generation as a side effect, and FEAT-29.SPEC-004 shows its Error state with a retry action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Automation Spec: Account Closure Orchestration

## Overview

**Name:** Account Closure Orchestration
**ID:** FEAT-29.SPEC-008
**Type:** Automation
**Purpose:** Sequences closure -- cancels the subscription, takes the booking page down, starts the 30-day cooling-off clock, and executes deletion (retaining only legally required de-identified financial records) when the cooling-off period expires unreversed.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Sequencing the closure steps once the Pro confirms closure with no upcoming bookings remaining
- Starting and tracking the cooling-off clock
- Executing permanent deletion when the cooling-off period expires unreversed
- Sending the closure confirmation and, later, the deletion notice

**Non-Goals:**
- Cancelling upcoming bookings with full refunds -- owned by FEAT-30 (Pro Booking Management), invoked by FEAT-29.SPEC-005 before this automation ever starts; this automation assumes no upcoming bookings remain
- Reversing closure during the cooling-off period -- owned by FEAT-29.SPEC-009 (Account Reopening); this automation only starts and completes the one-way sequence, or stops if reopening intervenes
- Defining the cooling-off period length and retention scope -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this automation implements
- Deleting the legally required de-identified financial history -- excluded per scope-boundaries.md SC-22: permanent deletion never removes the de-identified records the law requires to be retained

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro confirms account closure | FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Pro confirms "Close my account" with zero upcoming bookings remaining | Pro Account reference |
| Cooling-off period expires | System (scheduled check against Pro Account.status = Closing and its closure request date) | The account has been in Closing status for platform parameter: `account-closure-cooling-off-days` with no reopening | Pro Account reference, closure request date |

## Processing Logic

**Closure-start path:**
1. Receive the closure confirmation with the Pro Account reference.
2. Verify no upcoming bookings remain for this Pro Account (a final safety check; FEAT-29.SPEC-005 has already ensured this through FEAT-30).
3. Cancel the Subscription via FEAT-18.SPEC-006 (subscription-billing capability).
4. Take the booking page down (Pro Account status change signals FEAT-05.SPEC-008 to render "this booking page isn't available").
5. Set Pro Account.status to Closing and record the closure request date.
6. Trigger FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) to send the closure confirmation.

**Deletion path (cooling-off expiry):**
1. On the scheduled check, find every Pro Account whose status is Closing and whose closure request date is at least platform parameter: `account-closure-cooling-off-days` in the past, with no reopening having occurred.
2. For each such account: hard-delete the Pro's personal data -- sign_in_email, sign_in_mobile, signed_in_devices, and profile fields (display_name, photo, intro, studio_address, and other Pro Account fields).
3. Hard-delete Client contact details and notes for every Client belonging to this Pro Account, following the same semantics as XBR-19 (FEAT-13's client deletion).
4. De-identify Activity Events belonging to this Pro Account per FEAT-16.SPEC-005.
5. Retain Booking and Deposit Transaction history in de-identified form only, as legally required (SC-22) -- no further fields beyond what the law requires are kept.
6. Set Pro Account.status to Closed.
7. Trigger FEAT-29.SPEC-017 to send the final deletion notice.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Closure started | Confirmation received with zero upcoming bookings | Subscription cancelled; booking page taken down; Pro Account.status set to Closing; closure request date recorded | FEAT-29.SPEC-005 shows the Closing state with days remaining | FEAT-29.SPEC-005, FEAT-18, FEAT-05.SPEC-008, FEAT-29.SPEC-017 |
| Closure blocked | Upcoming bookings still exist at the moment of processing (a race with a newly created booking) | No status change | FEAT-29.SPEC-005 re-shows the updated upcoming-bookings list and requires the cancel-and-continue step again | FEAT-29.SPEC-005 |
| Permanent deletion executed | Cooling-off period expires with no reopening | Personal data hard-deleted; Client contact details hard-deleted; Activity Events de-identified; Booking/Deposit Transaction retained de-identified; Pro Account.status set to Closed | FEAT-29.SPEC-017 sends the final deletion notice; no in-product screen remains to show this state to the (now-deleted) account | FEAT-29.SPEC-017, FEAT-16.SPEC-005, FEAT-13 |
| Deletion skipped (reopened) | Reopening (FEAT-29.SPEC-009) occurred before the scheduled deletion check runs | No deletion occurs for this account | -- | FEAT-29.SPEC-009 |
| Closure-start failure | The subscription-cancel or booking-page-takedown step fails | No status change; the account remains Active/Paused | FEAT-29.SPEC-005 shows "Couldn't close your account. Try again." with a retry action | FEAT-29.SPEC-005 |

## Data Model

**Reads:** Pro Account (status, closure request date); Booking (to verify no upcoming bookings remain); Subscription (to cancel).
**Creates:** None.
**Updates:** Pro Account.status (Active/Paused -> Closing -> Closed); Subscription.status (-> Cancelled via FEAT-18.SPEC-006).
**Deletes:** Pro Account's personal fields (sign_in_email, sign_in_mobile, signed_in_devices, profile fields) on permanent deletion; Client contact details and notes for every Client of this Pro Account on permanent deletion. Booking and Deposit Transaction records are never deleted -- only de-identified.

## Business Rules

- XBR-20: upcoming bookings must first be cancelled with full refunds; the subscription is cancelled and the booking page taken down before the cooling-off clock starts; data is deleted only after the cooling-off period, keeping only legally required de-identified financial records.
- The cooling-off period is fixed at platform parameter: `account-closure-cooling-off-days` (FEAT-29.SPEC-013) -- this automation never shortens or extends it per account.
- Deletion never removes the de-identified financial records the law requires to be retained (SC-22) -- there is no "delete everything, no exceptions" option.
- Client contact-detail deletion on account closure follows the same hard-delete-with-de-identified-history semantics as XBR-19 (FEAT-13).
- Reopening (FEAT-29.SPEC-009) at any point before the scheduled deletion check runs cancels the deletion outcome entirely for that account -- there is no partial or in-progress deletion state that reopening must unwind.

## Edge Cases

- **A booking is created for this Pro Account between FEAT-29.SPEC-005's review step and this automation's own final safety check** -- Closure is blocked and the Pro is returned to the review step to clear the new booking; no partial closure (e.g., subscription cancelled but booking page still live) is ever left in place.
- **The subscription-cancel step succeeds but the booking-page-takedown step fails** -- The entire closure-start sequence is treated as a single unit: if any step fails, the automation reports failure and Pro Account.status is not advanced to Closing, so the account remains fully Active/Paused rather than left in a partially-closed state. A retry re-attempts the full sequence from the top, including re-invoking FEAT-18.SPEC-006's subscription cancellation; that call is idempotent against a subscription already in Cancelled status (cancelling an already-cancelled subscription is a no-op that leaves it Cancelled and returns success), so the retry never double-cancels or errors on the already-completed step -- it simply proceeds to re-attempt the booking-page takedown that failed.
- **Concurrent trigger firing -- the Pro confirms closure from two devices at nearly the same time** -- Only the first confirmation to commit transitions the account to Closing; the second is a no-op against an account already in Closing (idempotent), and its screen reflects the Closing state on next load.
- **Trigger fires while a previous run is in flight -- the scheduled deletion check runs while a closure-start is still processing for the same account** -- The deletion check only ever considers accounts already in Closing status with a recorded closure request date; an account whose closure-start has not yet committed that status cannot be selected by the same run, so no conflict arises.
- **The Pro reopens the account in the same window the scheduled deletion check is evaluating it** -- Reopening (FEAT-29.SPEC-009) is the authoritative status change; if it commits before the deletion check reads the account's status, the account is no longer Closing and is excluded from that run. If the deletion check has already begun processing that specific account in the same run, reopening is refused with the message defined in FEAT-29.SPEC-009's own Edge Cases, since deletion for that account is by then irreversible.
- **A refund tied to this Pro Account's bulk cancellation (FEAT-30) is still "in progress" when the cooling-off clock starts** -- The cooling-off clock and the refund's own completion are independent; the refund continues per its own retry rules (XBR-10) regardless of the account's Closing status, and its resulting Deposit Transaction record is retained (de-identified, if the account reaches permanent deletion) exactly like any other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Triggered by (inbound) | Confirmation of closure starts this automation |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Affects (outbound) | Shows the resulting Closing state or failure |
| FEAT-18 (Pro Subscription Billing & Account Management) | Affects (outbound) | Subscription cancellation via FEAT-18.SPEC-006 |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | Affects (outbound) | Booking page takedown |
| FEAT-29.SPEC-009 (Account Reopening) | References (inbound) | Reopening cancels this automation's deletion outcome |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Cooling-off period and retention scope |
| FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) | Affects (outbound) | Closure confirmation and final deletion notice |
| FEAT-16.SPEC-005 (Activity Record Immutability & Visibility Rules) | Affects (outbound) | De-identification of Activity Events on deletion |
| FEAT-13 (Client Record Management) | Affects (outbound) | Client contact-detail deletion, following XBR-19 semantics |

## Analytics and Success Signals

- **account_closure_requested** (had_upcoming_bookings: yes/no) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **account_deleted** (-- no properties beyond the event itself, since the account no longer exists to attach further context to) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so permanent deletion remains observable for operational and compliance monitoring

## Acceptance Criteria

**FEAT-29.SPEC-008-AC-01:** Given Talia confirms closure with zero upcoming bookings, when this automation runs, then her Subscription is cancelled, her booking page is taken down, and Pro Account.status is set to Closing with today's date recorded.

**FEAT-29.SPEC-008-AC-02:** Given Talia's closure-start commits successfully, when the sequence completes, then FEAT-29.SPEC-017 sends the closure confirmation.

**FEAT-29.SPEC-008-AC-03:** Given a new booking appears for Talia's account between screen review and this automation's final check, when the automation runs, then closure is blocked and no status change occurs.

**FEAT-29.SPEC-008-AC-04:** Given the booking-page-takedown step fails during closure-start, when the failure is detected, then Pro Account.status is not advanced to Closing and the account remains fully Active/Paused.

**FEAT-29.SPEC-008-AC-05:** Given Talia's account has been Closing for exactly platform parameter: `account-closure-cooling-off-days` with no reopening, when the scheduled deletion check runs, then her personal data and her clients' contact details are hard-deleted, her Activity Events are de-identified, and Pro Account.status is set to Closed.

**FEAT-29.SPEC-008-AC-06:** Given Talia's account reaches permanent deletion, when the deletion completes, then her Booking and Deposit Transaction history is retained in de-identified form only, per SC-22.

**FEAT-29.SPEC-008-AC-07:** Given Talia's account reaches permanent deletion, when FEAT-29.SPEC-009 (Account Reopening) is attempted afterward, then no reopening path exists -- deletion is irreversible.

**FEAT-29.SPEC-008-AC-08:** Given Talia reopens her account (FEAT-29.SPEC-009) before the scheduled deletion check runs, when the check next runs, then her account is excluded from deletion because its status is no longer Closing.

**FEAT-29.SPEC-008-AC-09:** Given Talia confirms closure from two devices at nearly the same time, when both confirmations process, then only the first commits the Closing transition and the second is a no-op against an account already Closing.

**FEAT-29.SPEC-008-AC-10:** Given a goodwill refund from Talia's bulk cancellation is still "in progress" when her cooling-off clock starts, when the refund's own retry cycle completes, then it proceeds independently of the account's Closing status per XBR-10.

**FEAT-29.SPEC-008-AC-11:** Given Talia's closure-start sequence fails partway (subscription cancelled, booking-page takedown fails), when the automation reports the failure, then FEAT-29.SPEC-005 shows "Couldn't close your account. Try again." and no partially-closed state is left visible.

**FEAT-29.SPEC-008-AC-12:** Given the scheduled deletion check has already begun processing Talia's account in the current run, when a reopening attempt arrives in that same window, then it is refused per FEAT-29.SPEC-009's own edge-case handling, since deletion is by then irreversible.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Account Reopening

## Overview

**Name:** Account Reopening
**ID:** FEAT-29.SPEC-009
**Type:** Automation
**Purpose:** Restores a Closing account to Active when the Pro signs back in during the cooling-off period, leaving the booking page down until the Pro separately resumes it.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Restoring Pro Account.status from Closing to Active on the Pro's explicit request during the cooling-off period
- Confirming that reopening leaves the subscription cancelled and the booking page down, both unchanged by this automation

**Non-Goals:**
- Resuming the booking page itself -- owned by FEAT-27 (Pro Profile & Booking Page Settings); reopening restores the account only, the Pro separately resumes bookings through FEAT-27's pause/resume control
- Restarting the subscription -- owned by FEAT-18 (Pro Subscription Billing & Account Management); a reopened Pro who wants to take bookings again subscribes again through FEAT-18, since XBR-26 requires an active subscription for the booking link to go live
- Executing the original closure or its cooling-off clock -- owned by FEAT-29.SPEC-008 (Account Closure Orchestration); this automation only reverses that clock's outcome before it fires
- Reopening after permanent deletion has already occurred -- excluded per this feature's Non-Goals: a cross-account or shared sign-in identity does not exist, and once data is deleted there is no account record left to restore

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests reopening | FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Pro Account.status is Closing (cooling-off period not yet expired) and the Pro taps "Reopen my account" | Pro Account reference |

## Processing Logic

1. Receive the reopening request with the Pro Account reference.
2. Verify Pro Account.status is currently Closing and the cooling-off period has not yet expired (the deletion check in FEAT-29.SPEC-008 has not begun processing this account).
3. Set Pro Account.status to Active, clearing the recorded closure request date.
4. Leave the Subscription in its cancelled state and the booking page down -- neither is touched by this step.
5. Signal FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) that the account is restored, for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Reopened | Status is Closing and the cooling-off period has not expired | Pro Account.status set to Active; closure request date cleared | FEAT-29.SPEC-005 shows "Your account is reopened" and navigates to FEAT-29.SPEC-003 | FEAT-29.SPEC-005, FEAT-29.SPEC-003 |
| Reopening refused (deletion already begun) | The scheduled deletion check (FEAT-29.SPEC-008) has already started processing this account in the current run | No status change; the account proceeds to permanent deletion | FEAT-29.SPEC-005 shows "This account can no longer be reopened." (no further Pro-facing surface exists once deletion completes) | FEAT-29.SPEC-008 |
| Reopening no-op (already Active) | Status is already Active (e.g., a second reopening attempt from another device) | No data change | FEAT-29.SPEC-005 reflects the already-Active state | FEAT-29.SPEC-005 |

## Data Model

**Reads:** Pro Account.status, closure request date.
**Creates:** None.
**Updates:** Pro Account.status (Closing -> Active); Pro Account's closure request date (cleared).
**Deletes:** None.

## Business Rules

- Reopening restores everything intact except the booking page, which stays down until the Pro separately resumes it through FEAT-27 (Cross-Feature Touchpoints) -- this automation never flips the booking page's own pause/resume state.
- Reopening never restarts the Subscription -- a reopened Pro subscribes again through FEAT-18 if they want to take bookings again, consistent with XBR-26's go-live prerequisite.
- Reopening is available at any point during the cooling-off period and becomes unavailable the instant the scheduled deletion check has begun processing that account (FEAT-29.SPEC-008) -- there is no partial reopening state, per the reopening cutoff defined in FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this automation enforces.
- Once permanent deletion completes, no reopening path exists for that account (Non-Goal) -- a returning Pro after deletion has no prior account to reach.

## Edge Cases

- **Pro requests reopening from two devices at nearly the same time** -- The first request to commit transitions the account to Active; the second is a no-op against an already-Active account (idempotent), and its screen reflects the current state.
- **Concurrent trigger firing -- reopening is requested at the exact moment the scheduled deletion check (FEAT-29.SPEC-008) begins processing the same account** -- Whichever commits first is authoritative: if reopening's status write lands before the deletion check reads the account's status for its own run, the account is excluded from that run's deletion and reopening succeeds; if the deletion check has already selected the account for processing in the current run, the reopening request is refused with "This account can no longer be reopened."
- **Trigger fires while a previous run is in flight -- two reopening requests from the same session in quick succession** -- The second is ignored while the first is processing (FEAT-29.SPEC-005's own double-tap guard); no duplicate status write occurs.
- **Pro reopens then immediately requests closure again** -- Treated as a fresh closure request; FEAT-29.SPEC-005's upcoming-bookings review runs again from a clean state, and a new closure (if confirmed) starts a new cooling-off clock from the current date, not a continuation of the previous one.
- **Pro reopens the account and finds their booking page still down** -- Expected behavior, not an error: the Pro is directed to FEAT-27's pause/resume control (and, if they want new bookings, FEAT-18's subscription flow) to fully restore live operation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Triggered by (inbound) | "Reopen my account" starts this automation |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Affects (outbound) | Shows the resulting Active state or refusal |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Affects (outbound) | Reflects the restored account after reopening |
| FEAT-29.SPEC-008 (Account Closure Orchestration) | References (inbound) | This automation reverses that automation's cooling-off outcome, when it has not yet fired |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Rule spec that defines the cooling-off cutoff after which reopening is refused; this automation enforces that cutoff |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (outbound, implied) | The Pro separately resumes the booking page here after reopening |
| FEAT-18 (Pro Subscription Billing & Account Management) | References (outbound, implied) | The Pro separately re-subscribes here if they want to take bookings again |

## Analytics and Success Signals

- **account_reopened** (days_remaining_at_reopen) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list

## Acceptance Criteria

**FEAT-29.SPEC-009-AC-01:** Given Talia's account is Closing with 12 days remaining in the cooling-off period, when she requests reopening, then Pro Account.status is set to Active and the closure request date is cleared.

**FEAT-29.SPEC-009-AC-02:** Given Talia's account is reopened, when she checks her Subscription, then it remains cancelled -- reopening does not restart it.

**FEAT-29.SPEC-009-AC-03:** Given Talia's account is reopened, when she checks her booking page, then it remains down until she separately resumes it through FEAT-27.

**FEAT-29.SPEC-009-AC-04:** Given the scheduled deletion check has already begun processing Talia's account in the current run, when she attempts reopening in that same window, then the request is refused with "This account can no longer be reopened." and deletion proceeds.

**FEAT-29.SPEC-009-AC-05:** Given Talia's account status is already Active (a second reopening attempt from another device), when the second request processes, then no data change occurs and the screen reflects the already-Active state.

**FEAT-29.SPEC-009-AC-06:** Given Talia's account has already been permanently deleted, when she attempts to reopen it, then no reopening path exists.

**FEAT-29.SPEC-009-AC-07:** Given Talia reopens her account and later requests closure again, when the new closure is confirmed, then a new cooling-off clock starts from the current date, not a continuation of the previous one.

**FEAT-29.SPEC-009-AC-08:** Given Talia requests reopening twice in quick succession from the same session, when the second request arrives while the first is processing, then it is ignored and no duplicate status write occurs.

**FEAT-29.SPEC-009-AC-09:** Given Talia requests reopening from two devices at nearly the same time, when both process, then only the first commits the Active transition and the second is a no-op.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Contact-Detail Change Processing

## Overview

**Name:** Contact-Detail Change Processing
**ID:** FEAT-29.SPEC-010
**Type:** Automation
**Purpose:** Carries a sign-in email or mobile-number change through code-entry confirmation on both the old and the new contact -- the same code-entry pattern used at sign-in (FEAT-29.SPEC-001) and recovery (FEAT-29.SPEC-002) -- before committing it.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Starting a pending contact-detail change when the Pro submits a new email or mobile number
- Generating and delivering an independent one-time code to both the old and new contact
- Validating a submitted code against the correct side and handling wrong/expired entries and per-side lockout
- Handling a "send a new code" request for either side
- Committing the change only once both sides confirm with a correct code
- Invalidating outstanding recovery state and, when the mobile number changes, client access links tied to the old identity

**Non-Goals:**
- Defining the confirmation codes' expiry, lockout, and overall confirmation-window rules -- owned by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules); this automation enforces those rules, it does not define them
- Starting the change from the Settings screen, or rendering the code-entry steps -- owned by FEAT-29.SPEC-003 (Account & Sign-In Settings Screen); this automation begins where that screen's submission ends
- Delivering the code content itself -- owned by FEAT-29.SPEC-016 (Contact-Change Confirmation Notification); this automation only triggers that delivery
- Changing any profile field other than sign_in_email or sign_in_mobile -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: this feature updates only sign-in contacts, devices, and closure status; other profile fields are FEAT-27's

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro submits a new sign-in email | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Edit" on the email row and submits a new value | New email value, current sign_in_email |
| Pro submits a new sign-in mobile number | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Edit" on the mobile row and submits a new value | New mobile value, current sign_in_mobile |
| Confirmation code submitted | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | The Pro enters a 6-digit code on either the old-contact or new-contact code-entry step | Which side, the code entered, the pending change reference |
| "Send a new code" requested | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | The Pro taps "Send a new code" on either side's code-entry step | Which side, the pending change reference |

## Processing Logic

1. Receive the new contact value and identify which field (sign_in_email or sign_in_mobile) is changing.
2. Create a pending contact-change record referencing the current value (old contact) and the submitted value (new contact), with both sides unconfirmed.
3. Generate an independent 6-digit code and expiry for each side per FEAT-29.SPEC-012, and trigger FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) to deliver each side's code to its own contact.
4. On a code submission for a side, validate it against that side's current, unexpired code per FEAT-29.SPEC-012:
   - Correct and unexpired: mark that side confirmed.
   - Incorrect or expired: increment that side's failed-attempt count; FEAT-29.SPEC-003 shows the generic failure message for that side; at 5 consecutive failures for that side, that side locks per FEAT-29.SPEC-012's lockout rule while the other side remains open.
5. On a "send a new code" request for a side: generate a fresh code and expiry for that side only, invalidate the prior code for that side, reset that side's failed-attempt count, and trigger FEAT-29.SPEC-016 to deliver the fresh code -- without affecting the other side or the overall confirmation window's start time.
6. When both sides have confirmed: commit the change -- update the Pro Account's sign_in_email or sign_in_mobile to the new value, invalidate any outstanding recovery code tied to the old identifier, and, if the mobile number changed, invalidate every outstanding Access Link tied to the old mobile number that a client might reuse to reach this Pro's account settings context (this automation invalidates only this feature's own recovery state; client-facing Access Link invalidation for the changed mobile number is a Client entity concern outside this feature's write scope and is not performed here).
7. Trigger FEAT-29.SPEC-016 to send the final confirmation of the committed change to both contacts.
8. If the overall confirmation window elapses per FEAT-29.SPEC-012 before both sides confirm, discard the pending change entirely, leaving the prior contact detail in place.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Change started | Pro submits a new email or mobile number | Pending contact-change record created; a code generated for each side | FEAT-29.SPEC-003 shows both code-entry steps, each with "We sent a code to {masked identifier}" | FEAT-29.SPEC-003, FEAT-29.SPEC-016 |
| Code entry failed | Pro submits a wrong or expired code for one side | That side's failed-attempt count incremented | FEAT-29.SPEC-003 shows "That code didn't work. Try again or send a new code." under that side's step; that side's code input clears | FEAT-29.SPEC-003 |
| Side locked | A side reaches 5 consecutive failed code attempts | That side is temporarily locked | FEAT-29.SPEC-003 shows "Too many attempts. Try again in {remaining minutes} minutes." on that side; the other side remains active | FEAT-29.SPEC-003 |
| New code requested | Pro taps "Send a new code" on one side | Fresh code generated for that side; prior code for that side invalidated; that side's failed-attempt count reset | FEAT-29.SPEC-003 clears that side's code input with the confirmation "New code sent"; FEAT-29.SPEC-016 delivers the fresh code | FEAT-29.SPEC-003, FEAT-29.SPEC-016 |
| Change committed | Both sides enter a correct, unexpired code | sign_in_email or sign_in_mobile updated; outstanding recovery code invalidated | FEAT-29.SPEC-003 shows the new value once loaded; FEAT-29.SPEC-016 sends the committed-change confirmation to both contacts | FEAT-29.SPEC-003, FEAT-29.SPEC-016 |
| Change expired | Neither side (or only one side) completes confirmation within FEAT-29.SPEC-012's overall confirmation window | Pending change discarded; prior value unchanged | FEAT-29.SPEC-003 row reverts to the prior value on next load | FEAT-29.SPEC-003 |
| Change failed (processing error) | An error occurs while committing | No update to sign_in_email/sign_in_mobile | FEAT-29.SPEC-003 shows "Couldn't update your contact details. Try again." | FEAT-29.SPEC-003 |

## Data Model

**Reads:** Pro Account.sign_in_email, sign_in_mobile (current values).
**Creates:** A pending contact-change record (old value, new value, and per side: current code, code expiry, failed-attempt count, confirmed flag).
**Updates:** Pro Account.sign_in_email or sign_in_mobile (on commit only); the pending record's per-side code, expiry, failed-attempt count, and confirmed flag throughout processing.
**Deletes:** The pending contact-change record (on commit or expiry -- it never persists once resolved).

## Business Rules

- A change is never committed until both the old and the new contact each enter a correct, unexpired code (FEAT-29.SPEC-012) -- there is no single-sided confirmation path.
- A side that fails 5 consecutive code attempts locks for platform parameter: `contact-change-code-lockout-pause-minutes` independent of the other side, which remains open for entry throughout.
- Changing a sign-in contact invalidates outstanding recovery state for the old identifier immediately on commit -- a code requested against the old identifier before the change can no longer be used to sign in afterward.
- Only one pending change per field may exist at a time -- submitting a new value while a change for the same field is already pending replaces the earlier pending change (and its codes) with a new one, restarting confirmation for both sides.
- This automation writes only sign_in_email and sign_in_mobile on the Pro Account -- it never touches any other Pro Account field or any Client field directly.

## Edge Cases

- **Pro submits a new email while a previous email change is still pending confirmation** -- The earlier pending change and its codes are discarded (no confirmation collected for it counts toward the new one) and a fresh pending change with fresh codes starts for the newly submitted value, per FEAT-29.SPEC-012.
- **The old contact enters a correct code but the new contact never enters any code** -- The change remains pending until FEAT-29.SPEC-012's overall confirmation window elapses, then is discarded; the prior contact detail stays in place, and the Pro is never left without a valid sign-in contact.
- **Concurrent trigger firing -- both sides' correct codes arrive at effectively the same time** -- Each code submission is processed independently and idempotently; when the second of the two is recorded, the commit step runs exactly once (the commit logic checks that both sides are now confirmed before proceeding, so a race between the two arrivals cannot produce two commits or none).
- **Trigger fires while a previous run is in flight -- the Pro submits a second contact change for the same field while the automation is mid-commit for the first** -- The mid-commit run is allowed to finish; only once it resolves (commit or expiry) does the newly submitted change become the active pending record, per the "only one pending change per field" rule.
- **The mobile number changes and a client holds an Access Link tied to the old number** -- Any client-facing Access Link scoping is governed and invalidated by FEAT-06's own phone-number-match rules (dependency map: Client Contention -- "a Pro phone-number change invalidates access links and requires fresh texting consent"); this automation's own recovery-state invalidation is limited to this feature's sign-in recovery codes, not client Access Links, which is why that boundary is stated explicitly in the Processing Logic.
- **A code arrives for a side that has already locked from 5 failed attempts** -- The submission is not evaluated against the code while that side is locked; FEAT-29.SPEC-003 continues to show the lockout message until platform parameter: `contact-change-code-lockout-pause-minutes` elapses, after which normal code entry resumes for that side.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Triggered by (inbound) | Submitting a new email or mobile number, a code entry, and a "send a new code" request all trigger this automation |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Affects (outbound) | Shows each side's code-entry, error, locked, committed, or reverted state |
| FEAT-29.SPEC-012 (Contact-Change Confirmation Rules) | References (inbound) | Governs code expiry, per-side lockout, and the overall confirmation window |
| FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) | Affects (outbound) | Sends each side's code and the final committed-change confirmation |

## Analytics and Success Signals

- **contact_details_changed** (field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **contact_change_code_failed** (field: email / mobile, side: old / new) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so wrong/expired code attempts remain observable
- **contact_change_expired** (field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so incomplete contact changes remain observable rather than silently dropped

## Acceptance Criteria

**FEAT-29.SPEC-010-AC-01:** Given Talia submits a new sign-in email on FEAT-29.SPEC-003, when the submission processes, then a pending contact-change record is created and a one-time code is sent to both her old and new email addresses.

**FEAT-29.SPEC-010-AC-02:** Given both Talia's old and new email enter their correct codes, when the second correct code is recorded, then Pro Account.sign_in_email is updated to the new value and both contacts receive the committed-change confirmation.

**FEAT-29.SPEC-010-AC-03:** Given only Talia's old email enters its correct code and her new email never responds, when FEAT-29.SPEC-012's overall confirmation window expires, then the pending change is discarded and her sign-in email remains the old value.

**FEAT-29.SPEC-010-AC-04:** Given Talia's new-email side submits a wrong code, when the entry is processed, then FEAT-29.SPEC-003 shows "That code didn't work. Try again or send a new code." for that side and her sign-in email remains the old value.

**FEAT-29.SPEC-010-AC-05:** Given Talia's mobile-number change commits successfully, when the change is committed, then any outstanding sign-in recovery code tied to the old mobile number can no longer be used to sign in.

**FEAT-29.SPEC-010-AC-06:** Given Talia has a pending email change awaiting confirmation, when she submits another new email before it resolves, then the earlier pending change and its codes are discarded and a fresh one starts for the newly submitted value.

**FEAT-29.SPEC-010-AC-07:** Given both of Talia's sides submit their correct codes at effectively the same time, when both are processed, then the change commits exactly once.

**FEAT-29.SPEC-010-AC-08:** Given a processing error occurs while committing Talia's contact change, when the failure is detected, then FEAT-29.SPEC-003 shows "Couldn't update your contact details. Try again." and no field is updated.

**FEAT-29.SPEC-010-AC-09:** Given Talia's mobile number changes, when a client's Access Link tied to the old number is evaluated, then its invalidation is governed by FEAT-06's own phone-number-match rules, not by this automation directly.

**FEAT-29.SPEC-010-AC-10:** Given Talia submits a second contact change for the same field while an earlier one is mid-commit, when the mid-commit run resolves, then only then does the newly submitted change become the active pending record.

**FEAT-29.SPEC-010-AC-11:** Given a change this automation writes, when the write is inspected, then it touches only Pro Account.sign_in_email or sign_in_mobile, never any other field.

**FEAT-29.SPEC-010-AC-12:** Given Talia's old-email side has locked after 5 consecutive wrong codes, when she taps "Send a new code" on that side, then a fresh code is generated, that side's failed-attempt count resets, and her new-email side's code and confirmed state are unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 7 | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Sign-In & Recovery Rules

## Overview

**Name:** Sign-In & Recovery Rules
**ID:** FEAT-29.SPEC-011
**Type:** Logic/Rule
**Purpose:** Governs one-time-code expiry, the failed-attempt lockout, session duration, and the anti-enumeration rule that a failed sign-in never reveals whether an account exists.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle
**Governed Entity:** Pro Account (sign-in slice: sign_in_email, sign_in_mobile, signed_in_devices) and the transient one-time-code state associated with a sign-in or recovery attempt

## Scope and Non-Goals

**In Scope:**
- One-time-code expiry
- Failed-attempt lockout threshold and pause duration
- Signed-in device (session) inactivity duration
- The anti-enumeration rule applied to every sign-in and recovery attempt
- Authorization for every action on the sign-in identity slice of the Pro Account

**Non-Goals:**
- Contact-detail change confirmation rules -- owned by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules); this spec governs sign-in and recovery only, not changing the sign-in contacts themselves
- Account closure and retention rules -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules)
- The step-by-step processing of a code request or verification -- owned by FEAT-29.SPEC-006 (Session & Device Management), which enforces these rules; this spec defines the rules, it does not execute them
- Support-assisted recovery -- excluded per scope-boundaries.md SC-05: no rule in this spec creates a path for support to see codes, sign in as the Pro, or bypass lockout on the Pro's behalf

## Governed Entity

**Entity:** Pro Account (sign-in slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| sign_in_email | text | Required sign-in email; each change confirmed through old and new contact (governed by FEAT-29.SPEC-012, not this spec) |
| sign_in_mobile | text | Required sign-in mobile number; each change confirmed through old and new contact (governed by FEAT-29.SPEC-012, not this spec) |
| signed_in_devices | derived (list) | Active sign-ins, up to platform parameter: `session-inactivity-expiry-days` of inactivity each |
| one_time_code (transient) | derived | The current unexpired code issued for a sign-in or recovery attempt against a given identifier; not a persisted Pro Account field |
| failed_attempt_count (transient) | derived | Count of consecutive failed code submissions for a given identifier since the last success or lockout reset |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-29.SPEC-001 | Sign-In Screen | On code submission; anti-enumeration applied on every code request regardless of match |
| FEAT-29.SPEC-002 | Account Recovery Screen | On code submission; anti-enumeration applied on every code request regardless of match |
| FEAT-29.SPEC-006 | Session & Device Management | During code generation, verification, and device inactivity expiry -- the automation that actually applies these rules |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| one_time_code | Must be submitted within platform parameter: `sign-in-code-expiry-minutes` of issue | Always | On code submission | "That code didn't work. Try again or send a new code." | Yes |
| one_time_code | Must exactly match the most recently issued code for the identifier | Always | On code submission | "That code didn't work. Try again or send a new code." | Yes |
| failed_attempt_count | Blocks further submission once it reaches platform parameter: `sign-in-lockout-threshold` | Always | On each submission attempt | "Too many attempts. Try again in {remaining minutes} minutes." | Yes |
| signed_in_devices entry | Expires after platform parameter: `session-inactivity-expiry-days` of no activity | Always | On the scheduled inactivity check (FEAT-29.SPEC-006) | No error message shown -- silent expiry per Business Rules | No (a housekeeping removal, not a validation failure) |
| sign_in_email / sign_in_mobile | No validation beyond data type in this spec | Always | -- | -- | -- (format and dual-confirmation validation for a change is FEAT-29.SPEC-012's) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Anti-enumeration | one_time_code, sign_in_email, sign_in_mobile | Whether or not the submitted identifier matches an existing Pro Account, the code-request and code-verification behavior (timing, messaging, and outcome shape) is identical | "That code didn't work. Try again or send a new code." (same message whether or not an account exists) |
| Lockout resets on success | failed_attempt_count, one_time_code | A successful verification resets failed_attempt_count to zero for that identifier | -- |
| New code supersedes prior code | one_time_code | Requesting a new code invalidates any previously issued, unexpired code for the same identifier immediately | -- |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Request a one-time code | The Pro (Talia) -- anyone submitting an identifier, since the action must not reveal account existence | Always | -- |
| Submit a one-time code | The Pro (Talia) | Only while failed_attempt_count is below platform parameter: `sign-in-lockout-threshold` | "Too many attempts. Try again in {remaining minutes} minutes." -- code input and further submission disabled for platform parameter: `sign-in-lockout-pause-minutes` |
| View signed-in devices | The Pro (Talia) | Only their own account's device list | -- |
| View signed-in devices | Platform Operator (Support) | Never -- Support sees account status only, never sign-in codes or devices (XBR-24, ASMP-30) | The signed-in device list is not rendered on Support's view of FEAT-29.SPEC-003; only the status line appears |
| Sign out a device | The Pro (Talia) | Only their own account's devices | -- |
| Sign out a device | Platform Operator (Support) | Never | No sign-out control is rendered anywhere on Support's view |
| Sign in as the Pro | Platform Operator (Support) | Never (SC-05) | No such action exists in the product at all -- there is no control, screen, or code path by which Support can assume the Pro's session |
| View another Pro's sign-in identity or devices | The Pro (Talia) | Never -- a sign-in identity belongs to exactly one Pro Account (dependency map: Pro Account Relationships) | No cross-account query path exists; the action is unreachable rather than denied with a message |
| View another Pro's sign-in identity or devices | Platform Operator (Support) | Never -- Support views one Pro account at a time, only after that Pro's own help request (XBR-24) | Support's view is always scoped to the single account opened via a help request; there is no listing or search across accounts |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| failed_attempt_count | Starts at zero for a new identifier or after a successful verification | On first code request for an identifier, and on every successful verification | No |
| signed_in_devices entry's last-active timestamp | Set to the current time on creation; refreshed to the current time on every successful verification from that device | On device creation and every subsequent successful verification | No |

## Business Rules

- One-time codes expire after platform parameter: `sign-in-code-expiry-minutes` (10 minutes, per product-features.md's stated Validation & Limits).
- After platform parameter: `sign-in-lockout-threshold` (5) consecutive failed attempts, further attempts are paused for platform parameter: `sign-in-lockout-pause-minutes` (15 minutes).
- A signed-in device stays active for up to platform parameter: `session-inactivity-expiry-days` (30) days of inactivity, after which FEAT-29.SPEC-006 expires it silently.
- XBR-29: every Pro-facing screen requires a signed-in Pro; anyone else is sent to FEAT-29.SPEC-001, and a failed sign-in never reveals whether an account exists.
- ASMP-30: account protection rests on one-time-code sign-in with new-device alerts (FEAT-29.SPEC-015); no password or remembered credential ever exists as an alternative path.

## Edge Cases

- **Code submitted at exactly platform parameter: `sign-in-code-expiry-minutes` after issue** -- Treated as expired; the boundary itself does not validate. A code must be submitted strictly before the expiry boundary.
- **Fifth failed attempt arrives at the exact same moment a valid code would have been accepted (e.g., the Pro's fifth submission is actually correct)** -- The lockout threshold is evaluated on failed attempts only; a correct fifth submission is a success, not a failure, and resets failed_attempt_count to zero rather than triggering lockout. Lockout triggers only when the fifth attempt is itself also incorrect.
- **Pro attempts a code submission during an active lockout pause** -- Rejected immediately with the lockout message, without evaluating the code's own correctness or expiry (the lockout gate is checked first).
- **A device's last-active timestamp sits at exactly platform parameter: `session-inactivity-expiry-days`** -- Treated as still active (not yet expired); expiry requires the gap to exceed the limit, not merely equal it.
- **Two failed attempts for the same identifier arrive from two different devices at nearly the same time** -- failed_attempt_count is a per-identifier counter, not per-device; both increments apply, and the lockout, once reached, blocks further submission attempts for that identifier from any device.
- **Support attempts to reach a control that would reveal a sign-in code or allow signing in as the Pro** -- No such control exists anywhere in the product for this role (SC-05); this is a structural absence, not a runtime denial to test against.

## Acceptance Criteria

**FEAT-29.SPEC-011-AC-01:** Given Talia's code was issued 9 minutes ago, when she submits it correctly, then verification succeeds.

**FEAT-29.SPEC-011-AC-02:** Given Talia's code was issued exactly platform parameter: `sign-in-code-expiry-minutes` ago, when she submits it, then it is treated as expired and the generic failure message appears.

**FEAT-29.SPEC-011-AC-03:** Given Talia has failed 4 consecutive attempts, when she submits a 5th incorrect code, then failed_attempt_count reaches platform parameter: `sign-in-lockout-threshold` and the lockout pause begins.

**FEAT-29.SPEC-011-AC-04:** Given Talia has failed 4 consecutive attempts, when she submits a 5th, correct code, then verification succeeds and failed_attempt_count resets to zero -- no lockout is triggered.

**FEAT-29.SPEC-011-AC-05:** Given Talia is inside an active lockout pause, when she attempts another submission, then it is rejected with "Too many attempts. Try again in {remaining minutes} minutes." without evaluating the code itself.

**FEAT-29.SPEC-011-AC-06:** Given Talia enters an identifier with no matching Pro Account, when she requests a code, then the request behaves identically (timing and messaging) to a matching identifier.

**FEAT-29.SPEC-011-AC-07:** Given a signed-in device's last activity was exactly platform parameter: `session-inactivity-expiry-days` ago, when the scheduled inactivity check runs, then the device is treated as still active.

**FEAT-29.SPEC-011-AC-08:** Given a signed-in device's last activity was one day more than platform parameter: `session-inactivity-expiry-days` ago, when the scheduled inactivity check runs, then the device is expired.

**FEAT-29.SPEC-011-AC-09:** Given Talia (the Pro) requests to view her own signed-in devices, when the request is made, then her full device list is shown.

**FEAT-29.SPEC-011-AC-10:** Given Platform Operator (Support) views a Pro's account, when the screen renders, then no signed-in device list, sign-in codes, or sign-out controls are shown to this role.

**FEAT-29.SPEC-011-AC-11:** Given two failed attempts for the same identifier arrive from two different devices in quick succession, when both are processed, then both increments apply to the same per-identifier failed_attempt_count.

**FEAT-29.SPEC-011-AC-12:** Given Support has an active help-request view of a Pro's account, when Support looks for any way to sign in as that Pro, then no such action exists anywhere in the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Contact-Change Confirmation Rules

## Overview

**Name:** Contact-Change Confirmation Rules
**ID:** FEAT-29.SPEC-012
**Type:** Logic/Rule
**Purpose:** Governs the dual-confirmation requirement for sign-in-contact changes -- each side proving control of its contact by entering a one-time code, identically to the Shared UI Pattern's code-entry step used at sign-in (FEAT-29.SPEC-001) and recovery (FEAT-29.SPEC-002) -- and what a partial or abandoned change leaves in place.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- The dual-confirmation requirement (both old and new contact must each enter a correct code)
- Per-side code generation, expiry, and failed-attempt lockout
- The overall confirmation window and its expiry
- What happens when only one side confirms and the other never responds

**Non-Goals:**
- Ordinary sign-in code rules -- owned by FEAT-29.SPEC-011 (Sign-In & Recovery Rules); this spec governs contact-change confirmation codes only, a distinct action from signing in, with its own code-expiry and lockout parameters
- Executing the change (updating the field, invalidating recovery state) -- owned by FEAT-29.SPEC-010 (Contact-Detail Change Processing), which this spec's rules govern
- The confirmation code's message content and delivery channel -- owned by FEAT-29.SPEC-016 (Contact-Change Confirmation Notification); this spec defines when a code is valid and what counts as confirmation, not the message wording
- Recovering access when a contact method is entirely lost -- owned by FEAT-29.SPEC-002/FEAT-29.SPEC-011; a contact change requires access to both current contacts, which is a different situation from recovery
- The code-entry screen's layout and interactions -- owned by FEAT-29.SPEC-003 (Account & Sign-In Settings Screen), which surfaces this spec's rules as the Shared UI Pattern's code-entry step

## Governed Entity

**Entity:** Pending contact-change record
**Source:** Feature Dependency Map (Pro Account, sign-in identity slice)

| Field | Data Type | Description |
|-------|-----------|--------------|
| field | enum (sign_in_email \| sign_in_mobile) | Which sign-in contact is being changed |
| old_value | text | The current value at the time the change was started |
| new_value | text | The submitted new value |
| old_contact_code | opaque, not displayed | The current one-time code sent to the old contact |
| old_contact_code_expires_at | derived | Expiry timestamp for the old contact's current code |
| old_contact_failed_attempts | integer | Consecutive failed code attempts on the old-contact side since its last successful or reset state |
| old_contact_confirmed | boolean | Whether the old contact has entered a correct, unexpired code |
| new_contact_code | opaque, not displayed | The current one-time code sent to the new contact |
| new_contact_code_expires_at | derived | Expiry timestamp for the new contact's current code |
| new_contact_failed_attempts | integer | Consecutive failed code attempts on the new-contact side since its last successful or reset state |
| new_contact_confirmed | boolean | Whether the new contact has entered a correct, unexpired code |
| started_at | derived | Timestamp the change was started, for the overall confirmation-window expiry |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-29.SPEC-003 | Account & Sign-In Settings Screen | On starting a contact change and rendering each side's code-entry step, generic failure message, lockout state, and reverted/committed state |
| FEAT-29.SPEC-010 | Contact-Detail Change Processing | During code generation, code validation, commit, and expiry processing of a pending change |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| new_value | Must resemble a valid email or mobile-number format matching the field being changed | Always | On submission (FEAT-29.SPEC-003) | "Enter a valid email address" / "Enter a valid mobile number" | Yes |
| new_value | Must differ from old_value | Always | On submission | "This is already your current {email/mobile number}" | Yes |
| old_contact_code / new_contact_code entry | Must be exactly 6 digits and match that side's current, unexpired code | Always | On submission (or auto-submit at 6 digits) on FEAT-29.SPEC-003 | "That code didn't work. Try again or send a new code." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Dual confirmation required | old_contact_confirmed, new_contact_confirmed | The change commits only when both are true; neither side's confirmation alone is sufficient | -- (no commit occurs; the pending state persists) |
| Overall window supersedes partial confirmation | started_at, old_contact_confirmed, new_contact_confirmed | If platform parameter: `contact-change-confirmation-window-hours` elapses from started_at with at most one side confirmed, the pending change is discarded regardless of which side confirmed | -- (silent discard, no error) |
| Per-side code expiry | old_contact_code_expires_at, new_contact_code_expires_at | Each side's code expires independently, platform parameter: `contact-change-code-expiry-minutes` after it was (re)sent; a code entered after its own side's expiry produces the same generic failure as a wrong code | "That code didn't work. Try again or send a new code." |
| Per-side lockout | old_contact_failed_attempts, new_contact_failed_attempts | After 5 consecutive failed attempts on one side, that side alone is locked for platform parameter: `contact-change-code-lockout-pause-minutes`; the other side's code entry is unaffected and can still be completed | "Too many attempts. Try again in {remaining minutes} minutes." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Start a contact change | The Pro (Talia) | Only for their own account's sign_in_email or sign_in_mobile | -- |
| Start a contact change | Platform Operator (Support) | Never (SC-05: support cannot change a Pro's sign-in details) | No control to start a contact change is rendered on Support's view of any screen |
| Enter a confirmation code for a side | Whoever holds the old contact, and whoever holds the new contact | Each side's code can only confirm that same side (an old-contact code cannot confirm the new-contact side, and vice versa) | A code entered against the wrong side is evaluated against that side's own current code and, not matching, produces the generic failure message like any other wrong code |
| View a pending change's status | The Pro (Talia) | Only their own account's pending change, as the code-entry steps and their confirmed/pending state on FEAT-29.SPEC-003 | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| old_contact_confirmed | false | On creation of the pending change | No (only set true by a correct, unexpired code entry) |
| new_contact_confirmed | false | On creation of the pending change | No (only set true by a correct, unexpired code entry) |
| old_contact_code / new_contact_code | Randomly generated 6-digit code, independent per side | On creation of the pending change, and again whenever "Send a new code" is used for that side | No |
| old_contact_code_expires_at / new_contact_code_expires_at | started_at (or the time of the most recent "Send a new code" for that side) plus platform parameter: `contact-change-code-expiry-minutes` | Recomputed every time that side's code is (re)generated | No |
| started_at | Current time | On creation of the pending change; not reset by a "Send a new code" on either side | No |

## Business Rules

- A contact change is not committed until both the old and the new contact each enter a correct, unexpired code (FEAT-29.SPEC-010) -- there is no single-sided confirmation path under any circumstance.
- A side that fails 5 consecutive code attempts is locked for platform parameter: `contact-change-code-lockout-pause-minutes`, independent of the other side, which remains open for entry throughout.
- An unconfirmed pending change (one or both sides never enter a correct code) expires after platform parameter: `contact-change-confirmation-window-hours` from when it started; on expiry, the prior contact detail remains in place with no error shown to the Pro.
- Starting a new contact change for the same field while one is already pending discards the earlier pending change (and its codes) and starts fresh codes and confirmation for both sides (FEAT-29.SPEC-010's Business Rules).

## Edge Cases

- **The old contact enters a correct code, then the new contact never responds** -- The pending change remains pending, with the old side shown as confirmed, until platform parameter: `contact-change-confirmation-window-hours` elapses; it then discards regardless of the old side's earlier success, and the prior contact detail remains in place.
- **Both sides' codes are entered correctly at effectively the same time** -- Each side's confirmation is processed independently and idempotently; the change commits the moment both are true, regardless of which arrived first, and commits exactly once.
- **The Pro starts a change, then starts a completely different change for the other field (email pending, then starts a mobile change) before the first resolves** -- These are independent pending changes on different fields with independent codes; both may be pending simultaneously without conflict, since the "only one pending change per field" rule (FEAT-29.SPEC-010) applies per field, not across fields.
- **A correct code is entered twice from the same side** -- The second entry is a no-op; that side's confirmed state is already true and is not re-evaluated or reset by a duplicate correct entry.
- **A side enters a wrong code repeatedly** -- That side's failed_attempts increments each time and shows the generic failure message; at 5 consecutive failures that side alone locks for platform parameter: `contact-change-code-lockout-pause-minutes`, while the other side's code entry remains available and unaffected throughout.
- **"Send a new code" is requested for one side** -- A fresh code and expiry are generated for that side only, that side's failed_attempts resets to zero, and the prior code for that side becomes invalid; the other side's code, confirmed state, and the overall started_at (and therefore the overall confirmation-window deadline) are unaffected.
- **The overall confirmation window expires at the exact same moment the second side's code is entered** -- Whichever is processed first wins: if the code entry is recorded before the expiry check runs, the change commits; if the expiry check runs first, the change is discarded and a subsequently entered code is evaluated as a wrong/expired entry against a pending change that no longer exists.
- **The Pro's old contact is also the value used elsewhere as a recovery contact** -- Confirming via the old contact's code and account recovery via the same contact (FEAT-29.SPEC-002) are independent actions; entering a confirmation code does not itself constitute a sign-in or recovery attempt, and vice versa.

## Acceptance Criteria

**FEAT-29.SPEC-012-AC-01:** Given Talia's old email enters its correct code for a pending email change, when only that one side has confirmed, then the change remains pending and does not commit.

**FEAT-29.SPEC-012-AC-02:** Given both Talia's old and new email enter their correct codes for a pending change, when the second correct code is recorded, then the change commits.

**FEAT-29.SPEC-012-AC-03:** Given Talia's new email enters a wrong code, when the entry is submitted, then the generic message "That code didn't work. Try again or send a new code." appears for that side and the pending change is not discarded.

**FEAT-29.SPEC-012-AC-04:** Given Talia's new-email side has failed 5 consecutive code attempts, when she tries to enter another code on that side, then that side shows "Too many attempts. Try again in {remaining minutes} minutes." while the old-email side remains available for entry.

**FEAT-29.SPEC-012-AC-05:** Given only one side of Talia's pending change confirms and platform parameter: `contact-change-confirmation-window-hours` elapses from when the change started, when expiry is evaluated, then the pending change is discarded with no error shown.

**FEAT-29.SPEC-012-AC-06:** Given Talia's old contact enters its correct code first and then her new contact enters its correct code, when the second correct code arrives, then the change commits; confirming order does not matter.

**FEAT-29.SPEC-012-AC-07:** Given Talia has a pending email change and a separately pending mobile change at the same time, when either resolves, then the other is unaffected -- the two pending changes are independent.

**FEAT-29.SPEC-012-AC-08:** Given Talia's old contact enters its correct code twice, when the second entry is processed, then it is a no-op and does not affect the pending change's state.

**FEAT-29.SPEC-012-AC-09:** Given Support attempts to start a contact change on a Pro's behalf, when the attempt is made, then no such control exists anywhere on Support's view.

**FEAT-29.SPEC-012-AC-10:** Given the confirmation window is about to expire at the same moment the second side's code is entered, when the code entry is recorded before the expiry check runs, then the change commits.

**FEAT-29.SPEC-012-AC-11:** Given Talia taps "Send a new code" on her old-email side after two wrong attempts, when the fresh code is generated, then that side's failed-attempt count resets to zero, the prior code for that side is invalidated, and her new-email side's code and confirmed state are unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Account Closure & Retention Rules

## Overview

**Name:** Account Closure & Retention Rules
**ID:** FEAT-29.SPEC-013
**Type:** Logic/Rule
**Purpose:** Governs the 30-day cooling-off period, the closure sequencing required before deletion, the scope of what deletion removes versus retains, and the scope of the data export.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle
**Governed Entity:** Pro Account (status/lifecycle slice), and by reference the Client, Booking, and Deposit Transaction entities as they are affected by closure, deletion, and export

## Scope and Non-Goals

**In Scope:**
- The cooling-off period length and what it protects
- The mandated sequencing of closure steps (bookings cancelled and refunded, subscription cancelled, booking page down, then the cooling-off clock starts)
- What permanent deletion removes and what it retains
- The scope of the data export (which fields, which entities, and the explicit private_note exclusion)
- Authorization for closure, reopening, and export actions

**Non-Goals:**
- Sign-in code and session rules -- owned by FEAT-29.SPEC-011; this spec governs account lifecycle and data retention, not sign-in itself
- Contact-change confirmation -- owned by FEAT-29.SPEC-012
- Executing the bulk cancellation, subscription cancellation, or booking-page takedown -- owned by FEAT-30, FEAT-18, and FEAT-05.SPEC-008 respectively; this spec states the sequencing rule that FEAT-29.SPEC-008 orchestrates against
- Deleting the legally required de-identified financial history -- excluded per scope-boundaries.md SC-22: this spec explicitly carves out that retention as a legal requirement, not a discretionary choice

## Governed Entity

**Entity:** Pro Account (status/lifecycle slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| status | enum (Active \| Paused \| Closing \| Closed) | This feature owns only the Closing/Closed/reopened-to-Active slice of this shared field |
| closure_request_date (derived) | date | The date closure was confirmed, used to compute cooling-off expiry |

**Referenced entities (by reference, not owned by this spec):** Client (contact details, notes -- deletion scope), Booking and Deposit Transaction (retention scope after deletion), Activity Event (de-identification scope after deletion).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-29.SPEC-004 | Data Export Screen | Displays the export-scope explanation to the Pro |
| FEAT-29.SPEC-005 | Account Closure & Reopening Screen | Displays the cooling-off period and closure sequencing to the Pro |
| FEAT-29.SPEC-007 | Data Export Generation | Enforces the export scope when assembling the file |
| FEAT-29.SPEC-008 | Account Closure Orchestration | Enforces closure sequencing, cooling-off timing, and deletion scope |
| FEAT-29.SPEC-009 | Account Reopening | Enforces the cutoff after which reopening is no longer possible |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | Transition to Closing is only valid when zero upcoming Bookings remain for this Pro Account | On closure confirmation | On closure confirmation (FEAT-29.SPEC-008) | -- (the screen re-shows the upcoming-bookings review rather than an error message; see FEAT-29.SPEC-005) | Yes |
| status | Transition from Closing to Active (reopening) is only valid before the scheduled deletion check has begun processing the account | On reopening request | On reopening request (FEAT-29.SPEC-009) | "This account can no longer be reopened." | Yes |
| closure_request_date | Set once, at the moment status transitions to Closing; cleared on reopening | Always | On closure confirmation and on reopening | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Cooling-off expiry | status, closure_request_date | Deletion is eligible only when status is Closing and the current date is at least platform parameter: `account-closure-cooling-off-days` past closure_request_date | -- |
| Closure sequencing | status, Subscription.status, Booking Page availability | The account cannot transition to Closing until upcoming bookings are cleared (cancelled and refunded); Subscription cancellation and booking-page takedown occur as part of the same closure-start step, before the cooling-off clock starts (XBR-20) | -- |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-----------------------------------------------|
| Request account closure | The Pro (Talia) | Only for their own account; only once zero upcoming bookings remain | -- |
| Request account closure | Platform Operator (Support) | Never (SC-05) | No closure control is rendered on Support's view |
| Reopen the account | The Pro (Talia) | Only for their own account, only while status is Closing and deletion has not yet begun processing it | "This account can no longer be reopened." once deletion has begun |
| Reopen the account | Platform Operator (Support) | Never | No reopening control is rendered on Support's view |
| Request a data export | The Pro (Talia) | Only for their own account's own data | -- |
| Request a data export | Platform Operator (Support) | Never -- support never sees or triggers a Pro's export | No export control is rendered on Support's view |
| View account status (Active/Paused/Closing/Closed) | Platform Operator (Support) | Always, view-only, after a Pro's help request (XBR-24) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| closure_request_date | Current date | On closure confirmation | No |
| Deletion eligibility date | closure_request_date + platform parameter: `account-closure-cooling-off-days` | Whenever status is Closing | No |

## Business Rules

- XBR-20 (account closure): upcoming bookings must first be cancelled with full refunds; the subscription is cancelled and the booking page taken down; data is deleted after a platform parameter: `account-closure-cooling-off-days` cooling-off period, keeping only legally required de-identified financial records.
- Permanent deletion hard-deletes: the Pro's sign-in email/mobile, signed-in devices, and profile fields; and, for every Client of this Pro Account, contact details and notes (following XBR-19 semantics).
- Permanent deletion never removes: legally required de-identified financial records (Booking and Deposit Transaction history, retained de-identified per SC-22); Activity Events are de-identified rather than deleted (FEAT-16.SPEC-005), preserving dispute-evidence integrity without retaining identifiable personal data.
- The data export's scope is exactly: the structured fields of Client (name, phone, email, booking_history reference), Booking (service, timing, price, deposit, state, timestamps), and Deposit Transaction (amount, status, outcome, timestamps) for the requesting Pro's own account only -- explicitly excluding Client.private_note (Coordination Note 4: private_note is Pro-only working notes, not a client or booking record proper).
- A closure or export action is never available to any role but the Pro whose account it concerns; Support's role throughout this spec is view-only status, per XBR-24.

## Edge Cases

- **Deletion eligibility date falls exactly on the current date** -- Eligible; the rule is "at least" the cooling-off period, so a request evaluated on the exact boundary day qualifies for deletion processing.
- **The Pro requests an export the same day the cooling-off period is about to expire** -- The export is still generated normally per its own scope rule; requesting an export has no effect on the cooling-off clock or deletion eligibility, since the two are independent actions.
- **A Client record has an upcoming booking at the moment of a de-identification pass triggered by this Pro's account deletion** -- Cannot occur under normal sequencing, since XBR-20 requires all upcoming bookings to be cancelled and refunded before the cooling-off clock can even start; if a data inconsistency were somehow found, deletion processing would not proceed for that Client until the inconsistency is resolved, consistent with FEAT-13's own deletion-eligibility rule (XBR-19: deletion is refused while an upcoming booking exists).
- **Two competing intentions -- the Pro requests reopening and, in the same visit, immediately requests a fresh export** -- Independent actions; reopening changes status to Active, and an export request that follows reads the now-Active account's current data normally.
- **The de-identification pass for Activity Events runs concurrently with the hard-delete pass for Client contact details** -- Both are part of the same deletion sequence (FEAT-29.SPEC-008) and operate on disjoint fields (Activity Event content vs. Client contact fields), so no field-level conflict exists between them; the sequence completes both before setting Pro Account.status to Closed.
- **A legally required de-identified financial record is later needed for a dispute after the Pro's account is deleted** -- It remains available in de-identified form indefinitely (SC-22); no further purge ever removes it, since this spec explicitly carves it out of every deletion pass.

## Acceptance Criteria

**FEAT-29.SPEC-013-AC-01:** Given Talia's account has zero upcoming bookings, when she confirms closure, then status transitions to Closing and closure_request_date is set to today.

**FEAT-29.SPEC-013-AC-02:** Given Talia's account has upcoming bookings, when she attempts to confirm closure without clearing them, then the transition to Closing is refused and the upcoming-bookings review is shown instead.

**FEAT-29.SPEC-013-AC-03:** Given Talia's account has been Closing for exactly platform parameter: `account-closure-cooling-off-days`, when the deletion eligibility check runs, then the account is eligible for deletion.

**FEAT-29.SPEC-013-AC-04:** Given Talia's account has been Closing for one day fewer than platform parameter: `account-closure-cooling-off-days`, when the deletion eligibility check runs, then the account is not yet eligible.

**FEAT-29.SPEC-013-AC-05:** Given Talia's account reaches permanent deletion, when the deletion executes, then her sign-in email/mobile, signed-in devices, and profile fields are hard-deleted, along with her clients' contact details and notes.

**FEAT-29.SPEC-013-AC-06:** Given Talia's account reaches permanent deletion, when the deletion executes, then her Booking and Deposit Transaction history is retained in de-identified form only, and her Activity Events are de-identified rather than deleted.

**FEAT-29.SPEC-013-AC-07:** Given Talia requests a data export, when the file is generated, then it includes only structured Client, Booking, and Deposit Transaction fields for her own account, excluding Client.private_note.

**FEAT-29.SPEC-013-AC-08:** Given deletion has already begun processing Talia's account, when she attempts to reopen it, then the request is refused with "This account can no longer be reopened."

**FEAT-29.SPEC-013-AC-09:** Given Platform Operator (Support) views a Pro's account, when they look for a closure, reopening, or export control, then none is rendered anywhere on their view.

**FEAT-29.SPEC-013-AC-10:** Given Talia requests an export on the same day her cooling-off period is set to expire, when the export generates, then it completes normally with no effect on the deletion eligibility clock.

**FEAT-29.SPEC-013-AC-11:** Given Talia's account has already been permanently deleted, when a card-issuer dispute later requires her historical financial records, then the de-identified Booking and Deposit Transaction history remains available.

**FEAT-29.SPEC-013-AC-12:** Given the deletion sequence for Talia's account runs, when both the Activity Event de-identification pass and the Client contact-detail hard-delete pass execute, then both complete before Pro Account.status is set to Closed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: Sign-In Code Notification

## Overview

**Name:** Sign-In Code Notification
**ID:** FEAT-29.SPEC-014
**Type:** Notification
**Purpose:** Delivers the one-time sign-in code to whichever contact method Talia is signing in with.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Delivering the one-time code on the exact contact method (email or mobile) the Pro entered, whether for ordinary sign-in, recovery, or first-time onboarding sign-in
- Delivery failure and retry behavior for this time-critical message

**Non-Goals:**
- Generating the code or validating its correctness -- owned by FEAT-29.SPEC-006 (Session & Device Management) and FEAT-29.SPEC-011 (Sign-In & Recovery Rules); this spec only delivers the code once generated
- Alerting the Pro to a new-device sign-in after successful verification -- owned by FEAT-29.SPEC-015 (New-Device Sign-In Alert), a distinct notification sent after this one, on success
- Batching multiple codes into one message -- a code request is always a single, immediate, individually time-sensitive action; batching would defeat the purpose of a just-in-time sign-in code

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | The Pro entered her sign-in email as the identifier | Matches exactly the contact method the Pro chose to use for this attempt -- there is no reason to also text a code to a different contact |
| SMS (text) | The Pro entered her sign-in mobile number as the identifier | Matches exactly the contact method the Pro chose to use; a Pro signing in from her phone expects the code to arrive where she is already looking |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A one-time code is generated for a code request | FEAT-29.SPEC-006 (Session & Device Management) | Fires every time a code request is processed, on sign-in, recovery, or onboarding, regardless of whether the identifier matches an existing account | The generated code, the contact method and value it was requested on, the request path (sign-in / recovery / onboarding) |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- specifically, whoever holds the contact method entered on FEAT-29.SPEC-001 or FEAT-29.SPEC-002, since the code is delivered to exactly the identifier submitted, never to any other contact on file. Per the Access Matrix, this is a Pro-only notification; no other role receives it.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this notification cannot be turned off | -- | Always delivered | -- |

A sign-in code cannot be opted out of: it is the sign-in mechanism itself, not a discretionary communication, so no preference control governs it (consistent with FEAT-29's Non-Goal excluding password-based sign-in as the only alternative).

**Quiet Hours:** N/A -- a sign-in code is delivered immediately regardless of time of day, since it is requested by the Pro in the moment she is actively trying to sign in; holding it for quiet hours would break the sign-in flow she just initiated.

## Content Definition

**Email:**
- **Subject:** Your Chairtime sign-in code: {code}
- **Body:**
  Your one-time code to sign in to Chairtime is:

  {code}

  This code expires in {code_expiry_minutes} minutes. If you didn't request this, you can ignore this email.
- **CTA:** None -- the code itself is the actionable content; the Pro returns to the screen where she requested it and enters the code there.

**SMS (text):**
- **Body:** Your Chairtime sign-in code is {code}. Expires in {code_expiry_minutes} min. Didn't request this? Ignore this text.
- **CTA:** None -- same reasoning as email.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {code} | Derived -- the one-time code generated by FEAT-29.SPEC-006 for this request | 482913 | Never empty -- this notification is triggered only once a code has been generated; there is no code-less firing |
| {code_expiry_minutes} | Derived -- platform parameter: `sign-in-code-expiry-minutes` | 10 | Never empty -- always present, one value for every Pro |

## Delivery Rules

**Batching:** None -- every code request produces its own individual, immediate message. A Pro who requests a new code before the previous one arrives simply receives two messages, the second carrying the current valid code (the earlier one having been superseded per FEAT-29.SPEC-011's rule that a new code invalidates the prior one).
**Deduplication:** None beyond natural supersession -- each code request is a distinct event and produces its own delivery; there is no cross-request deduplication window, since a Pro genuinely requesting two codes in succession (e.g., "send a new code") expects and needs the latest one delivered.
**Retry on failure:** SMS delivery failure is retried once through the same channel; if it still fails, the Pro sees the request-failed state on FEAT-29.SPEC-001/FEAT-29.SPEC-002 ("Could not send code. Check your connection and try again.") rather than a silent fallback to email, since the Pro entered a specific contact method and switching channels on her behalf could deliver the code somewhere she is not looking. Email delivery failure is retried once through the same channel with the same on-screen failure behavior if it still fails.
**Expiry:** The notification itself has no independent expiry beyond the code's own expiry (platform parameter: `sign-in-code-expiry-minutes`); once the code expires, a message that has not yet arrived is still delivered if it eventually does, but it will simply no longer validate -- the Pro requests a new code instead.

## Edge Cases

- **The Pro requests a new code before the previous one has been delivered** -- Both messages may still arrive, but only the code in the most recently generated message will validate (FEAT-29.SPEC-011); the Pro is not confused about which to use because entering the older, superseded code produces the same generic "that code didn't work" message that prompts requesting a fresh one again.
- **Delivery fails on the entered contact method and the Pro has no alternative contact entered for this attempt** -- No automatic fallback to the other contact method occurs, since the Pro must be the one who chooses which contact to sign in with; she sees the request-failed state and can retry or switch to entering her other contact method herself on FEAT-29.SPEC-001.
- **Quiet hours collide with expiry** -- N/A -- this notification has no quiet-hours hold (see Audience and Preferences), so no such collision can occur.
- **The underlying "record" (the code itself) becomes invalid before delivery completes** -- If the Pro requests a new code while the first is still in flight for delivery, the first message, if it arrives late, still shows the (by-then superseded) code; entering it produces the standard "that code didn't work" message, exactly as any expired or superseded code would.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggered by (inbound) | Every code generation fires this notification |
| FEAT-29.SPEC-001 (Sign-In Screen) | References (inbound) | The screen this notification's code is entered back into |
| FEAT-29.SPEC-002 (Account Recovery Screen) | References (inbound) | The screen this notification's code is entered back into for recovery |
| FEAT-29.SPEC-011 (Sign-In & Recovery Rules) | References (inbound) | Defines the code's expiry duration shown in this notification's content |

## Analytics and Success Signals

- **sign_in_code_delivered** (channel: email / sms, path: sign_in / recovery / onboarding) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so code-delivery reliability remains observable
- **sign_in_code_delivery_failed** (channel: email / sms, path: sign_in / recovery / onboarding) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so delivery failures are never silent

## Acceptance Criteria

**FEAT-29.SPEC-014-AC-01:** Given Talia enters her sign-in email on FEAT-29.SPEC-001 and requests a code, when the code is generated, then she receives an email with the subject "Your Chairtime sign-in code: {code}" carrying that exact code.

**FEAT-29.SPEC-014-AC-02:** Given Talia enters her sign-in mobile number and requests a code, when the code is generated, then she receives a text with her code and its expiry stated in minutes.

**FEAT-29.SPEC-014-AC-03:** Given Talia requests a code during account recovery (FEAT-29.SPEC-002), when the code is generated, then the same content and delivery rules apply as for ordinary sign-in.

**FEAT-29.SPEC-014-AC-04:** Given Talia requests a code for an identifier with no matching account, when the request is processed, then delivery behavior (timing, and whether anything is actually sent) is indistinguishable from a matching identifier, per the anti-enumeration rule.

**FEAT-29.SPEC-014-AC-05:** Given Talia's text delivery fails once, when the automatic retry also fails, then FEAT-29.SPEC-001 shows "Could not send code. Check your connection and try again." rather than silently falling back to email.

**FEAT-29.SPEC-014-AC-06:** Given Talia requests a new code before the previous one arrives, when both messages eventually arrive, then only the code from the most recently generated message validates.

**FEAT-29.SPEC-014-AC-07:** Given this notification fires at any hour, when it is triggered, then it is delivered immediately with no quiet-hours hold.

**FEAT-29.SPEC-014-AC-08:** Given Talia's code expires before a delayed message arrives, when she enters the code from that delayed message, then she sees the generic "that code didn't work" message, identical to any other expired code.

**FEAT-29.SPEC-014-AC-09:** Given Talia signs in for the first time during onboarding, when the code is requested, then she receives this same notification on whichever contact method she entered.

**FEAT-29.SPEC-014-AC-10:** Given Talia's code delivery fails on both the initial attempt and its retry, when the failure is final, then a sign_in_code_delivery_failed event is recorded so the failure is never silent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: New-Device Sign-In Alert

## Overview

**Name:** New-Device Sign-In Alert
**ID:** FEAT-29.SPEC-015
**Type:** Notification
**Purpose:** Alerts Talia on her existing contact methods when her account is signed in on a device not seen before, so an unrecognized sign-in is never silent.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Alerting the Pro on both her existing sign-in contacts (email and mobile) when a new device successfully signs in
- Letting the alert direct the Pro to her devices list if the sign-in was not her own

**Non-Goals:**
- Detecting whether a device is new -- owned by FEAT-29.SPEC-006 (Session & Device Management); this spec only delivers the alert once that determination is made
- Blocking or reversing the sign-in -- this alert is informational; the sign-in has already succeeded by the time this notification fires (product-features.md's Access field: an account-protection measure that surfaces awareness, not a blocking gate)
- Alerting on the very first device created during onboarding -- excluded per FEAT-29.SPEC-006's Business Rules: there is no prior device or established account to alert from at that moment

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to the Pro's sign-in email | The email address is a channel the Pro is likely to still control even if the new sign-in used a different, possibly unauthorized device, making it a reliable independent alert surface |
| SMS (text) | Always, to the Pro's sign-in mobile number | Text reaches the Pro on her phone quickly, matching her behavioral pattern of checking Chairtime between clients (Behavioral Context, user-persona.md) |

Both channels are used together (not one-or-the-other) because this is a security alert: reaching the Pro reliably matters more than minimizing message volume for this one, infrequent event.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A new signed-in device is created | FEAT-29.SPEC-006 (Session & Device Management) | Fires whenever a successful code verification creates a signed_in_devices entry that did not previously exist, except for the first device created during onboarding | Device description, approximate location/type if available, sign-in time |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- delivered to both her current sign_in_email and sign_in_mobile, since either could be the contact she notices the alert on first. Per the Access Matrix, this is a Pro-only notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this alert cannot be turned off | -- | Always delivered | -- |

A new-device alert is a core account-protection measure (ASMP-30) and is never optional, consistent with there being no discretionary security notification the Pro can silence.

**Quiet Hours:** N/A -- a new-device sign-in alert is delivered immediately regardless of time of day, since a delayed security alert defeats its purpose; a Pro who is the victim of unauthorized access needs to know as soon as possible.

## Content Definition

**Email:**
- **Subject:** New sign-in to your Chairtime account
- **Body:**
  Your Chairtime account was just signed in on a device we haven't seen before ({device_description}) at {sign_in_time}.

  If this was you, you can ignore this message. If it wasn't, open your account settings and sign out everywhere right away.
- **CTA:** Manage sign-in -- deep-links to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen), the devices section

**SMS (text):**
- **Body:** Chairtime: new sign-in on a device we haven't seen before ({device_description}) at {sign_in_time}. If this wasn't you, sign out everywhere in Account & Sign-In settings.
- **CTA:** None on SMS itself (text has no interactive deep-link surface in this product); the Pro follows up in-app via FEAT-29.SPEC-003.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {device_description} | Pro Account.signed_in_devices -- the new entry's device description | iPhone, San Francisco area | "a device" -- used when no descriptive detail is available, so the sentence still reads naturally: "signed in on a device we haven't seen before (a device)" is avoided in favor of omitting the parenthetical entirely when the fallback applies |
| {sign_in_time} | Pro Account.signed_in_devices -- the new entry's creation timestamp, in the Pro's timezone | 3:42 PM | Never empty -- the entry always has a creation timestamp |

## Delivery Rules

**Batching:** None -- each new-device sign-in is its own distinct security event and is delivered individually the moment it is detected; batching a security alert would delay the Pro's awareness of a specific event.
**Deduplication:** At most one alert per new signed_in_devices entry -- the same device signing in again later (an existing device refreshing its last-active timestamp) does not re-trigger this alert, since it is by definition no longer new.
**Retry on failure:** Both email and SMS are retried once on failure. Unlike the Sign-In Code Notification, a failure on one channel does not block the other -- both channels are attempted independently regardless of whether the other succeeds, since this alert's reliability matters more for a security notification than avoiding channel overlap.
**Expiry:** This notification never expires undelivered -- a delayed security alert is still worth delivering whenever it can be, unlike a time-sensitive sign-in code; there is no cutoff after which delivery is abandoned.

## Edge Cases

- **The new device belongs to the Pro herself (e.g., a new phone)** -- The alert still fires; there is no way to distinguish a Pro's own new device from an unauthorized one at the moment of sign-in, so the alert is deliberately unconditional, and the Pro simply disregards it when it was her own action.
- **The new device is signed out (individually or via "sign out everywhere") before this notification is delivered** -- The alert is still delivered; it describes a sign-in event that already occurred, not the device's current state, so its informational value stands regardless of what happens to the device afterward.
- **Quiet hours collide with expiry** -- N/A -- this notification has no quiet-hours hold and no expiry (see Delivery Rules), so no such collision can occur.
- **Two new devices sign in within moments of each other** -- Each produces its own independent alert; they are not batched together, since each represents a distinct event the Pro needs to evaluate on its own.
- **Both delivery channels fail (email and SMS both fail after retry)** -- No further fallback channel exists for this notification; the failure is recorded (see Analytics) so it remains observable for operational monitoring, but the Pro has no in-product indicator of a missed new-device alert beyond her devices list on FEAT-29.SPEC-003, which always shows the device regardless of alert delivery success.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggered by (inbound) | New-device detection fires this notification |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (outbound) | The CTA deep-links to the devices section |

## Analytics and Success Signals

- **new_device_alert_delivered** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so this security alert's delivery reliability remains observable
- **new_device_alert_delivery_failed** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so a missed security alert is never silent, given both channels have no further fallback

## Acceptance Criteria

**FEAT-29.SPEC-015-AC-01:** Given Talia signs in successfully from a device not previously seen, when the sign-in succeeds, then she receives both an email and a text alert describing the new sign-in.

**FEAT-29.SPEC-015-AC-02:** Given Talia signs in from a device already in her signed_in_devices list, when the sign-in succeeds, then no new-device alert is sent.

**FEAT-29.SPEC-015-AC-03:** Given a brand-new Pro completes her first-ever sign-in during onboarding, when her first device is created, then no new-device alert is sent for that first device.

**FEAT-29.SPEC-015-AC-04:** Given Talia receives the email alert, when she taps "Manage sign-in", then she is navigated to FEAT-29.SPEC-003, devices section.

**FEAT-29.SPEC-015-AC-05:** Given the new device that triggered this alert is signed out before the alert is delivered, when delivery proceeds, then the alert is still sent describing the sign-in event that occurred.

**FEAT-29.SPEC-015-AC-06:** Given this notification fires at any hour, when it is triggered, then it is delivered immediately with no quiet-hours hold and no expiry.

**FEAT-29.SPEC-015-AC-07:** Given Talia's email delivery fails, when the retry also fails, then the SMS delivery is still attempted independently regardless of the email outcome.

**FEAT-29.SPEC-015-AC-08:** Given two new devices sign in within moments of each other, when both are detected, then two separate alerts are sent, not one batched alert.

**FEAT-29.SPEC-015-AC-09:** Given both email and SMS delivery fail after retry, when the failure is final, then a new_device_alert_delivery_failed event is recorded, and Talia's devices list on FEAT-29.SPEC-003 still shows the device regardless.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Contact-Change Confirmation Notification

## Overview

**Name:** Contact-Change Confirmation Notification
**ID:** FEAT-29.SPEC-016
**Type:** Notification
**Purpose:** Delivers the one-time confirmation code for a sign-in email or mobile-number change to each side -- the same code-entry pattern used to deliver a sign-in code (FEAT-29.SPEC-014) -- and confirms the change to both the old and the new contact once it commits.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- The initial code-delivery variant, sent to both the old contact and the new contact when a change is started, and re-sent to one side on a "send a new code" request
- The committed-change confirmation variant, sent to both contacts once both sides confirm
- Delivery rules for both variants

**Non-Goals:**
- Deciding when a code is valid, when a side locks, or when a change commits or expires -- owned by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules); this spec only delivers the messages that rules produce
- Starting the change itself, or the code-entry steps a Pro submits a code into -- owned by FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) and FEAT-29.SPEC-010 (Contact-Detail Change Processing)
- Any notice when a change is not confirmed or expires -- excluded per product-features.md's Communications field, which names only "confirmations of contact-detail changes to both old and new contacts," not a separate expiry notice; an unconfirmed or expired change simply produces no further message beyond the two variants defined here

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | The contact being confirmed (old or new) is an email address | Delivers to exactly the contact method being confirmed -- the entire point of dual confirmation is that each side proves control of its own contact method by entering the code that arrived there |
| SMS (text) | The contact being confirmed (old or new) is a mobile number | Same reasoning as email, for the mobile side of the change |

Each of the two code deliveries (old-contact, new-contact) uses whichever channel matches that specific contact's type -- an email change sends two email messages (one to old, one to new); a mobile change sends two texts. A "send a new code" request re-sends only to the requested side, on that side's channel.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A pending contact change is started | FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Fires the initial code-delivery variant to both the old and new contact | Old contact value, new contact value, which field is changing, each side's code |
| A "send a new code" request is processed for one side | FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Fires the initial code-delivery variant again, to the requested side only | Which side, that side's fresh code |
| A pending contact change commits | FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Fires the committed-change confirmation variant to both contacts, once both sides have confirmed | Old contact value, new contact value, which field changed, commit time |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- specifically whoever holds the old contact and whoever holds the new contact for this change. In the ordinary case both are Talia herself; the notification is addressed to each contact independently since confirmation depends on control of the contact, not identity per se.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this notification cannot be turned off | -- | Always delivered | -- |

Confirming a sign-in contact change is a security-relevant action with no discretionary opt-out, consistent with the other account-protection notifications in this feature.

**Quiet Hours:** N/A -- both variants are delivered immediately regardless of time of day; the initial code requires timely entry within FEAT-29.SPEC-012's confirmation window, and holding it for quiet hours would erode that window without the Pro's knowledge.

## Content Definition

**Initial code delivery -- Email (sent to whichever side is an email address):**
- **Subject:** Your Chairtime code to confirm this change
- **Body:**
  You're changing the {field_label} on your Chairtime sign-in from {old_value_masked} to {new_value_masked}.

  Enter this code in the Chairtime app to confirm: {confirmation_code}

  If you didn't request this, ignore this message and no change will be made.
- **CTA:** None -- the code is entered on FEAT-29.SPEC-003 (Account & Sign-In Settings Screen), not through a link in this message, identically to the sign-in code pattern (FEAT-29.SPEC-014)

**Initial code delivery -- SMS (sent to whichever side is a mobile number):**
- **Body:** Chairtime: your code to confirm changing your sign-in {field_label} is {confirmation_code}. Enter it in the app. Didn't request this? Ignore this text.
- **CTA:** None -- the code is entered on FEAT-29.SPEC-003

**Committed-change confirmation -- Email:**
- **Subject:** Your Chairtime sign-in {field_label} has changed
- **Body:**
  Your Chairtime sign-in {field_label} was changed from {old_value_masked} to {new_value_masked} on {commit_date}.

  If you didn't make this change, open your account settings and sign out everywhere right away.
- **CTA:** Manage sign-in -- deep-links to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen)

**Committed-change confirmation -- SMS:**
- **Body:** Chairtime: your sign-in {field_label} changed to {new_value_masked} on {commit_date}. Didn't do this? Sign out everywhere in Account & Sign-In settings.
- **CTA:** None on SMS itself; the Pro follows up in-app via FEAT-29.SPEC-003.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {field_label} | Derived -- "email address" or "mobile number" depending on which field is changing | mobile number | Never empty -- always one of the two known field labels |
| {confirmation_code} | Pending contact-change record.old_contact_code or .new_contact_code, whichever side this delivery is for | 482913 | Never empty -- generated whenever a code is sent or resent for that side (FEAT-29.SPEC-012) |
| {old_value_masked} | Pending contact-change record.old_value, masked | t***a@gmail.com | Never empty -- a pending change always has an old_value |
| {new_value_masked} | Pending contact-change record.new_value, masked | (***) ***-9821 | Never empty -- a pending change always has a new_value |
| {commit_date} | Derived -- the date the change committed, in the Pro's timezone | September 27, 2026 | Never empty -- present only on the committed variant, which fires only once commit occurs |

## Delivery Rules

**Batching:** None -- each pending change produces exactly one initial code delivery per side (plus one more per side for each "send a new code" request) and, if it commits, exactly one committed confirmation per side; these are never batched with any other notification.
**Deduplication:** At most one active code per side at a time -- a "send a new code" request invalidates the prior code for that side and produces exactly one fresh delivery to that side, never a resend of the invalidated code. At most one committed confirmation per pending change per side. Starting a fresh pending change for the same field (superseding an earlier one, per FEAT-29.SPEC-010's Business Rules) produces its own fresh codes and deliveries -- the earlier pending change's codes are not resent or reused.
**Retry on failure:** Both email and SMS are retried once on failure, independently per side and per variant. A failed code delivery to one side does not block delivery to the other side, since each side's confirmation is independent (FEAT-29.SPEC-012).
**Expiry:** Each delivered code expires per platform parameter: `contact-change-code-expiry-minutes` (FEAT-29.SPEC-012). A code entered after its own expiry is evaluated as a wrong/expired entry producing the generic failure message on FEAT-29.SPEC-003, not a silent no-op; the Pro can then request a new code for that side.

## Edge Cases

- **The contact receiving the code is not actually the Pro (e.g., the new mobile number was mistyped and belongs to someone else)** -- That recipient sees a code but has no reason to enter it into an app they do not use; if they ignore it, the change never commits and expires per FEAT-29.SPEC-012, leaving the Pro's contact details unchanged. If they mistakenly relay or enter the code somewhere, the change still requires the Pro's own old-contact code before committing, limiting the impact of a single mistaken entry.
- **The underlying pending change is discarded (superseded by a fresh change, or expired) before the other side's code is delivered or entered** -- A code delivered or entered against a pending change that no longer exists is evaluated as a wrong/expired entry, per FEAT-29.SPEC-012's handling of a code against an already-resolved change; no separate error state is needed beyond the generic failure message.
- **Preferences changing between trigger and delivery** -- N/A -- this notification has no preference control to change (see Audience and Preferences).
- **Quiet hours colliding with expiry** -- N/A -- this notification has no quiet-hours hold (see Delivery Rules), so no such collision can occur.
- **Both the old and new contact happen to be the same channel type (e.g., changing between two email-like identifiers is not possible in this product, but a change from email to email is impossible since email and mobile are distinct fields) -- N/A** -- each contact-change record is scoped to exactly one field (sign_in_email or sign_in_mobile), so old and new are always the same type (both email addresses, for an email change; both mobile numbers, for a mobile change); this is a closed, well-defined case, not an edge case requiring special handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Triggered by (inbound) | The start, a "send a new code" request, and the commit of a pending change each fire this notification's respective variant |
| FEAT-29.SPEC-012 (Contact-Change Confirmation Rules) | References (inbound) | Governs each code's expiry and whether a prompt still matters |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | References (inbound) | Shares the same code-entry-in-app pattern rather than a link or reply action |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (outbound) | The committed-confirmation email's CTA deep-links here; codes themselves are entered here |

## Analytics and Success Signals

- **contact_change_code_delivered** (side: old / new, channel: email / sms, field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so code delivery remains observable
- **contact_change_confirmed_notification_delivered** (side: old / new, channel: email / sms, field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list

## Acceptance Criteria

**FEAT-29.SPEC-016-AC-01:** Given Talia starts a mobile-number change, when the pending change is created, then both her old and new mobile numbers receive a text with a one-time code to confirm the change.

**FEAT-29.SPEC-016-AC-02:** Given Talia starts an email change, when the pending change is created, then both her old and new email addresses receive an email with a one-time code to confirm the change.

**FEAT-29.SPEC-016-AC-03:** Given both sides of Talia's pending change enter their correct codes, when the change commits, then both her old and new contact receive a committed-change confirmation.

**FEAT-29.SPEC-016-AC-04:** Given Talia's committed-change confirmation email arrives, when she taps "Manage sign-in", then she is navigated to FEAT-29.SPEC-003.

**FEAT-29.SPEC-016-AC-05:** Given the old-contact code delivery fails to send, when the retry also fails, then the new-contact code delivery is still attempted independently.

**FEAT-29.SPEC-016-AC-06:** Given a pending change is discarded (superseded or expired) before the new-contact code is delivered, when that code eventually arrives and is entered, then the entry is evaluated as a wrong/expired code, since the pending change no longer exists.

**FEAT-29.SPEC-016-AC-07:** Given a fresh pending change supersedes an earlier one for the same field, when the fresh change starts, then it produces its own new codes rather than resending the earlier ones.

**FEAT-29.SPEC-016-AC-08:** Given this notification fires at any hour, when either variant is triggered, then it is delivered immediately with no quiet-hours hold.

**FEAT-29.SPEC-016-AC-09:** Given Talia's contact change is never confirmed by either side and the overall confirmation window expires, when expiry occurs (FEAT-29.SPEC-012), then no further notification is sent beyond the codes and any committed confirmation already delivered.

**FEAT-29.SPEC-016-AC-10:** Given a new mobile number was mistyped and belongs to someone unrelated, when that person receives the code and does not enter it anywhere, then the change never commits and the Pro's contact details remain unchanged.

**FEAT-29.SPEC-016-AC-11:** Given Talia taps "Send a new code" on her old-email side, when the fresh code is generated, then only her old email receives the new code, and the prior code for that side is invalidated.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 3 (start, resend, commit) | 3 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Account Closure & Deletion Notifications

## Overview

**Name:** Account Closure & Deletion Notifications
**ID:** FEAT-29.SPEC-017
**Type:** Notification
**Purpose:** Sends the account-closure confirmation when closure is requested and the final notice when data is permanently deleted.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- The closure-confirmation variant, sent when Talia's account transitions to Closing
- The final deletion-notice variant, sent when her account transitions to Closed and data is permanently deleted

**Non-Goals:**
- A notice when the account is reopened -- excluded per product-features.md's Communications field, which names only "an account-closure confirmation and a final notice when data is permanently deleted"; reopening's confirmation is an in-product message on FEAT-29.SPEC-005 ("Your account is reopened"), not a standalone Notification spec
- Deciding the cooling-off timing itself -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules); this spec only delivers messages at the moments FEAT-29.SPEC-008 triggers
- Notifying the Pro's clients about the closure -- out of scope for this feature; any client-facing consequence (e.g., the booking page becoming unavailable) is handled by FEAT-05's own messaging, not this spec

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to the Pro's sign-in email as recorded at the moment each message fires | A closure or deletion confirmation is a significant, non-urgent-to-act-on record the Pro may want to keep; email is the durable, reviewable channel for it |
| SMS (text) | Always, to the Pro's sign-in mobile number as recorded at the moment each message fires | Matches the Pro's behavioral pattern of checking her phone, ensuring she is aware even if she does not check email promptly |

Both channels are used together for both variants, consistent with this being a significant, infrequent, irreversible-consequence event.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Account closure is confirmed | FEAT-29.SPEC-008 (Account Closure Orchestration) | Fires the closure-confirmation variant once the closure-start sequence commits (subscription cancelled, booking page down, status set to Closing) | Closure request date, cooling-off period length |
| Permanent deletion executes | FEAT-29.SPEC-008 (Account Closure Orchestration) | Fires the final deletion-notice variant once the cooling-off period expires unreversed and deletion completes | Deletion date |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- delivered to her sign-in email and mobile number as recorded at the moment each message fires (the closure confirmation uses the contacts on file when closure starts; the deletion notice is sent immediately before or as the final deletion step removes those same contacts, so it must be dispatched using the still-current values at that instant).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this notification cannot be turned off | -- | Always delivered | -- |

Both variants confirm significant, account-defining state changes with no discretionary opt-out; a Pro closing or permanently losing their account data must be told regardless of any notification preference.

**Quiet Hours:** N/A -- both variants are delivered as soon as their triggering step completes, regardless of time of day; a closure or deletion confirmation is not time-sensitive in the way a sign-in code is, but holding it artificially would only delay a Pro's awareness of a significant, irreversible-approaching change with no corresponding benefit.

## Content Definition

**Closure confirmation -- Email:**
- **Subject:** Your Chairtime account is closing
- **Body:**
  We've started closing your Chairtime account. Your subscription is cancelled and your booking page is down.

  You have {cooling_off_days} days to change your mind. Sign back in any time before then to reopen your account with everything intact. After that, your data is permanently deleted, except the financial records we're required by law to keep.
- **CTA:** Manage account -- deep-links to FEAT-29.SPEC-005 (Account Closure & Reopening Screen)

**Closure confirmation -- SMS:**
- **Body:** Chairtime: your account is closing. You have {cooling_off_days} days to sign back in and reopen it before your data is permanently deleted.
- **CTA:** None on SMS itself; the Pro follows up in-app via FEAT-29.SPEC-005.

**Deletion notice -- Email:**
- **Subject:** Your Chairtime account data has been deleted
- **Body:**
  Your cooling-off period has ended, and your Chairtime account data has now been permanently deleted, as you requested when you closed your account.

  We've kept only the financial records the law requires us to retain, in de-identified form. This account can no longer be reopened.
- **CTA:** None -- this is a final, terminal notice with no further in-product action available for this account.

**Deletion notice -- SMS:**
- **Body:** Chairtime: your account data has been permanently deleted after your cooling-off period ended. This account can no longer be reopened.
- **CTA:** None.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {cooling_off_days} | Derived -- platform parameter: `account-closure-cooling-off-days` | 30 | Never empty -- always present, one value for every Pro |

## Delivery Rules

**Batching:** None -- each variant is a single, standalone message tied to a specific state transition; neither is ever batched with any other notification.
**Deduplication:** At most one closure confirmation per closure event, and at most one deletion notice per deletion event. A Pro who closes, reopens, and later closes again generates a fresh closure confirmation for the new closure event -- these are independent lifecycle events, not duplicates of the same message.
**Retry on failure:** Both email and SMS are retried once on failure for the closure confirmation, independently per channel. For the deletion notice specifically, delivery is attempted before the sign-in contacts are removed as part of the same deletion sequence (FEAT-29.SPEC-008's Processing Logic), so a failure here has no further retry once the contact details themselves are gone -- the notice is best-effort at that final moment, and its delivery outcome is recorded (see Analytics) so a failure is never silent even though it cannot be retried against a now-deleted contact.
**Expiry:** Neither variant expires undelivered -- both are significant, one-time lifecycle confirmations that remain worth delivering whenever they can be sent, with no cutoff.

## Edge Cases

- **The Pro reopens the account before the closure confirmation is even delivered** -- The confirmation, if it still arrives, describes an accurate historical fact (that closure was started at that moment); it causes no confusion, since the Pro's current in-product state (Active, per the reopening) is shown correctly on FEAT-29.SPEC-003 regardless of a delayed confirmation message.
- **The deletion notice's delivery must complete using contact details that are about to be deleted in the same operation** -- Delivery is sequenced to fire before (or as part of) the same deletion step that removes sign_in_email and sign_in_mobile, ensuring the notice is dispatched to valid contacts; per FEAT-29.SPEC-008's Processing Logic, this notification is explicitly one of the deletion sequence's own steps, not an afterthought following it.
- **Quiet hours colliding with expiry** -- N/A -- neither variant has a quiet-hours hold or an expiry (see Delivery Rules), so no such collision can occur.
- **The Pro closes, reopens, and closes again within a short period** -- Each closure produces its own closure confirmation; there is no deduplication across genuinely separate closure events, since each represents the Pro's own distinct decision.
- **Both delivery channels fail for the deletion notice** -- No further fallback exists (the contacts are being deleted in the same step); the failure is recorded for operational visibility, but there is no further Pro-facing surface to retry against once the account is Closed and its contacts are gone.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-008 (Account Closure Orchestration) | Triggered by (inbound) | Fires both variants at their respective moments |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Navigation (outbound) | The closure-confirmation email's CTA deep-links here |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Defines the cooling-off period length shown in the closure confirmation |

## Analytics and Success Signals

- **account_closure_confirmation_delivered** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **account_deletion_notice_delivered** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list, and especially so given this notice cannot be retried once the contact details are gone

## Acceptance Criteria

**FEAT-29.SPEC-017-AC-01:** Given Talia confirms account closure with no upcoming bookings remaining, when FEAT-29.SPEC-008's closure-start sequence commits, then she receives the closure confirmation on both email and text, stating platform parameter: `account-closure-cooling-off-days` days remain to reopen.

**FEAT-29.SPEC-017-AC-02:** Given Talia's closure confirmation email arrives, when she taps "Manage account", then she is navigated to FEAT-29.SPEC-005.

**FEAT-29.SPEC-017-AC-03:** Given Talia's account reaches the end of its cooling-off period unreversed, when permanent deletion executes, then she receives the deletion notice on both email and text, stating the account can no longer be reopened.

**FEAT-29.SPEC-017-AC-04:** Given Talia's deletion notice must reach her sign-in contacts before those same contacts are deleted, when the deletion sequence runs, then delivery is attempted as part of that same sequence, before the contact fields are removed.

**FEAT-29.SPEC-017-AC-05:** Given Talia reopens her account before the closure confirmation is delivered, when the delayed confirmation eventually arrives, then it causes no confusion, since her in-product status correctly shows Active.

**FEAT-29.SPEC-017-AC-06:** Given Talia closes, reopens, and closes her account again, when each closure commits, then each produces its own independent closure confirmation.

**FEAT-29.SPEC-017-AC-07:** Given either notification fires at any hour, when it is triggered, then it is delivered immediately with no quiet-hours hold.

**FEAT-29.SPEC-017-AC-08:** Given Talia's closure-confirmation email delivery fails, when the retry also fails, then the SMS delivery is still attempted independently.

**FEAT-29.SPEC-017-AC-09:** Given both delivery channels fail for Talia's deletion notice, when the failure is final, then no further retry occurs against her now-deleted contact details, and the failure is recorded for operational visibility.

**FEAT-29.SPEC-017-AC-10:** Given Talia's deletion notice is delivered, when she reads it, then it states that only legally required de-identified financial records are retained.

**FEAT-29.SPEC-017-AC-11:** Given Talia's account is permanently deleted, when she reads the deletion notice, then it states explicitly that the account can no longer be reopened.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 2 (closure, deletion) | 2 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |

