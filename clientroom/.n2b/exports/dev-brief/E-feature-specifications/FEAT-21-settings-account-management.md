# FEAT-21 — Settings & Account Management

This chapter covers Settings & Account Management, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 11 specifications carrying 134 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-21.SPEC-001 | Account Profile | screen | 17 |
| FEAT-21.SPEC-002 | Notification Preferences | screen | 11 |
| FEAT-21.SPEC-003 | Login & Security | screen | 14 |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | screen | 13 |
| FEAT-21.SPEC-005 | Sign-In Email & Login Method Change | automation | 12 |
| FEAT-21.SPEC-006 | Sign-Out Other Sessions | automation | 9 |
| FEAT-21.SPEC-007 | Account Field Validation Rules | logic-rule | 15 |
| FEAT-21.SPEC-008 | Notification Preference Rules | logic-rule | 9 |
| FEAT-21.SPEC-009 | Business Details Completeness Gate | logic-rule | 10 |
| FEAT-21.SPEC-010 | Settings Access & Read-Only Scope Rules | logic-rule | 12 |
| FEAT-21.SPEC-011 | Account-Critical Change Confirmation Email | notification | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Settings & Account Management

## Summary

**Feature:** Settings & Account Management
**ID:** FEAT-21
**Description:** The freelancer edits her account profile, manages notification preferences, and manages her own login recovery (clients use magic links only, per FEAT-05).
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The decomposition checklist's Commonly Forgotten Areas expect a settings surface for any product with accounts. Ranked Important rather than Core because the product's primary loop functions without visiting settings; phased MVP since account-recovery and notification control are needed from first use. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Edit profile — update account details
- Manage notification preferences — control which optional notifications are sent
- Manage login recovery — update email/login method for her own account
- Sign in securely — Nadia signs in to her own account, can see where she is signed in, and can sign out other devices [AUDIT-ADDED: 4 -- security and privacy posture: the freelancer's own sign-in was implied but undefined]
- Business details and payment terms — business name, address, tax ID, and default payment terms printed on her invoices [AUDIT-ADDED: 3 -- inverse check: invoice content (FEAT-09) had no capture point]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-21.SPEC-001 | Account Profile | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia views and edits her name and account profile, and is the Settings entry point navigating to every other section (including Branding and account closure); Dana views the same profile read-only inside a logged support session |
| FEAT-21.SPEC-002 | Notification Preferences | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia turns optional notifications on or off, with transactional record emails always shown as locked-on; Dana views the same preferences read-only inside a logged support session |
| FEAT-21.SPEC-003 | Login & Security | Screen | Nadia (Freelancer) | Nadia starts a sign-in email or login method change, and views and manages her signed-in devices, including signing out other sessions; never surfaced to Dana |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia sets her business name, address, tax ID, and default payment terms that print on every invoice; Dana views the same values read-only inside a logged support session |
| FEAT-21.SPEC-005 | Sign-In Email & Login Method Change | Automation | Nadia (Freelancer) | Processes a pending sign-in email or login method change through re-verification before it takes effect, and expires or lets Nadia resend an unconfirmed change |
| FEAT-21.SPEC-006 | Sign-Out Other Sessions | Automation | Nadia (Freelancer) | Invalidates every one of Nadia's signed-in sessions except the current one and records the event |
| FEAT-21.SPEC-007 | Account Field Validation Rules | Logic/Rule | Nadia (Freelancer) | Enforces required fields, format, and value limits across profile, business details, and payment terms fields |
| FEAT-21.SPEC-008 | Notification Preference Rules | Logic/Rule | Nadia (Freelancer) | Enforces that only optional notifications can be switched off and that transactional record emails always send (XBR-30) |
| FEAT-21.SPEC-009 | Business Details Completeness Gate | Logic/Rule | Nadia (Freelancer) | Tracks whether business details and payment terms are complete and blocks the first invoice send until they are (XBR-16) |
| FEAT-21.SPEC-010 | Settings Access & Read-Only Scope Rules | Logic/Rule | All | Enforces that only Nadia can edit her own account, that Dana's support session sees profile/preference/business values read-only and never sign-in credentials, and that client contacts have no settings surface at all |
| FEAT-21.SPEC-011 | Account-Critical Change Confirmation Email | Notification | Nadia (Freelancer) | Sends Nadia a confirmation email when her sign-in email address or login method actually changes |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Edit profile — update account details | FEAT-21.SPEC-001, FEAT-21.SPEC-007 | Primary purpose of the profile screen; field validation from the Logic/Rule spec | Phase 2 (Explicit) |
| Manage notification preferences | FEAT-21.SPEC-002, FEAT-21.SPEC-008 | Primary purpose of the preferences screen; the transactional-vs-optional rule governs what can be toggled | Phase 2 (Explicit) |
| Manage login recovery — update email/login method | FEAT-21.SPEC-003, FEAT-21.SPEC-005 | Login & Security screen starts the change; the Automation carries it through re-verification | Phase 2 (Explicit) |
| Sign in securely (view sessions, sign out other devices) | FEAT-21.SPEC-003, FEAT-21.SPEC-006 | Login & Security screen lists signed-in devices; the Automation invalidates other sessions on request | Phase 2 (Explicit) |
| Business details and payment terms | FEAT-21.SPEC-004, FEAT-21.SPEC-007, FEAT-21.SPEC-009 | Primary purpose of the business details screen; validation from the shared field-validation rule; completeness tracked by the gate rule that FEAT-09 checks before sending | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-21.SPEC-005 | Sign-In Email & Login Method Change | Phase 4 (Trigger-Response Analysis) | The Validation & Limits field states "email changes require re-verification" -- a pending-then-confirmed process with real failure modes (expiry, abandonment), crossing the standalone-Automation threshold rather than a direct field write |
| FEAT-21.SPEC-006 | Sign-Out Other Sessions | Phase 4 (Trigger-Response Analysis) | The "sign out other devices" capability has a cross-entity effect -- invalidating every other active session at once -- past the threshold for a simple inline write |
| FEAT-21.SPEC-007 | Account Field Validation Rules | Phase 5 (Rule-Constraint Discovery) | Validation rules apply across three separate screens (profile, business details, payment terms) with 5+ distinct field rules (name required, email format, business name/address/tax ID required-before-invoicing, payment-terms value set), crossing the standalone-Logic/Rule threshold |
| FEAT-21.SPEC-008 | Notification Preference Rules | Phase 5 (Rule-Constraint Discovery) | Conditional rule shared with FEAT-14: a preference can switch off delivery only when the notification type is optional; this interacts with FEAT-14's delivery logic and is the authority side of XBR-30 |
| FEAT-21.SPEC-009 | Business Details Completeness Gate | Phase 3 (Entity-Lifecycle Analysis) / Phase 4 (Trigger-Response Analysis) | The Update cell for Freelancer Account surfaced a conditional gate ("Business details are required before the first invoice is sent") that another feature (FEAT-09) checks at send time (XBR-16); this needed a standalone home rather than being buried in the business details screen |
| FEAT-21.SPEC-010 | Settings Access & Read-Only Scope Rules | Phase 5 (Rule-Constraint Discovery) | Authorization-rule analysis found entitlements that vary by role across every screen in this feature (Nadia full, Dana view-only except credentials, client contacts none), shared across four screens -- well past the standalone threshold |
| FEAT-21.SPEC-011 | Account-Critical Change Confirmation Email | Phase 4 (Notification surfacing lens) | The Communications field names a confirmation email with real delivery rules (channel: email; audience: Nadia herself; content: which account-critical change occurred) |

## Entity-Lifecycle Coverage Matrix

**Entity: Freelancer Account**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Created by Onboarding / First-Run Setup (FEAT-20) at sign-up; this feature never creates the account, only updates it (feature-dependency-map.md, Entity: Freelancer Account) | -- |
| Read (single) | FEAT-21.SPEC-001, FEAT-21.SPEC-002, FEAT-21.SPEC-003, FEAT-21.SPEC-004 | Each screen loads its own slice of the current account record when opened (profile, preferences, sign-in/sessions, business details) | Dana's read-only support session also reads SPEC-001/002/004's slices, never SPEC-003's |
| Read (list) | N/A | Exactly one Freelancer Account per freelancer; there is no list view | -- |
| Update | FEAT-21.SPEC-001 (name/profile), FEAT-21.SPEC-002 (notification_preferences), FEAT-21.SPEC-003 via FEAT-21.SPEC-005 (sign-in email/login method, pending until re-verified) and FEAT-21.SPEC-006 (signed-in devices list), FEAT-21.SPEC-004 (business_name, business_address, tax_id, default_payment_terms) | All updates save immediately once validated (FEAT-21.SPEC-007); two open sessions of Nadia's resolve last-write-wins per field, except the sign-in email change, which requires re-verification before it takes effect (feature-dependency-map.md, Contention) | -- |
| Delete/Archive | N/A -- explicit non-goal | Full account deletion is owned entirely by Data Export & Account Deletion (FEAT-24); the feature's own Primary Flows state that a freelancer wanting to close her account is routed to FEAT-24 "rather than duplicating that flow here." This feature never soft- or hard-deletes the Freelancer Account, defines no restore path, and applies no retention/purge policy of its own -- those are FEAT-24's responsibility | -- |
| State Transition | FEAT-21.SPEC-005 | The sign-in email/login method field moves Active -> Pending re-verification -> Active (confirmed) or reverts to the prior value on expiry/abandonment | No other field on this entity carries a workflow state |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Notification | FEAT-21.SPEC-002 | The preferences screen reads the set of notification types (and which are optional vs. transactional) to build the toggle list; it never reads or writes individual Notification instances |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia edits her profile and saves | Validate required fields and format | Standalone Logic/Rule | SPEC-007 |
| Nadia edits business details and/or payment terms and saves | Validate required fields and format; re-check whether business details are now complete | Standalone Logic/Rule | SPEC-007 / SPEC-009 |
| Nadia toggles a notification preference | Check whether the notification type is optional before allowing the toggle; transactional types stay locked on | Standalone Logic/Rule | SPEC-008 |
| Nadia submits a new sign-in email or login method | Start the pending change, send a re-verification link/code through transactional email delivery | Standalone Automation | SPEC-005 |
| Nadia confirms the re-verification link/code | Commit the sign-in email/login method change and send the account-critical change confirmation email | Standalone Automation -> Standalone Notification | SPEC-005 -> SPEC-011 |
| The re-verification link/code expires, or Nadia abandons the change | Revert to the prior sign-in email/login method; the previous value stays active with no data loss; Nadia can restart the change | Standalone Automation (Error/ambiguous-outcome handling) | SPEC-005 |
| Nadia taps "Sign out other devices" | Invalidate every session except the current one immediately | Standalone Automation | SPEC-006 |
| Business details are complete and Nadia (or the system) attempts to send the first invoice for a project | Allow the send to proceed | Standalone Logic/Rule (cross-feature enforcement point) | SPEC-009 |
| Business details are incomplete and the first invoice send is attempted | Block the send and prompt Nadia to complete business details in Settings | Standalone Logic/Rule (cross-feature enforcement point) | SPEC-009 |
| Any save on any screen fails server-side | Retry the save without discarding unsaved fields (feature's own States field: "Error: a failed save is retried without discarding unsaved fields") | Inline in triggering screen (Error state) | SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 |
| Nadia saves any screen successfully | Show a success confirmation; changes save immediately per the Primary Flow | Inline in triggering screen | SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 |
| Dana opens a read-only support session (FEAT-31) on the freelancer's account | Render the profile, notification preferences, and business details screens read-only; never render Login & Security or any sign-in credential | Standalone Logic/Rule | SPEC-010 |
| Owen or Priya attempts to reach any Settings screen | Nothing is shown -- the Access Matrix gives both roles "None" on Branding, Onboarding & Settings | Standalone Logic/Rule (Permission Denied) | SPEC-010 |
| Nadia navigates to "Close account" from Settings | Route to Data Export & Account Deletion (FEAT-24); this feature never duplicates the closure flow | Inline in triggering screen -- cross-feature | SPEC-001 |
| Nadia returns to Settings to finish a previously skipped branding step | Navigate to Freelancer Branding's settings screen (FEAT-19) | Inline in triggering screen -- cross-feature | SPEC-001 |
| A profile edit, preference change, business-details update, email/login change, or sign-out-others completes | Emit the corresponding signal (`settings_updated`, `notification_preference_changed`, `account_email_changed`, `business_details_updated`, `other_sessions_signed_out`) | Inline in triggering spec | SPEC-001 / SPEC-002 / SPEC-004 / SPEC-005 / SPEC-006 |

## Shared Context

**Shared Entities:**
- Freelancer Account -- read by all four screens (SPEC-001, SPEC-002, SPEC-003, SPEC-004), updated by SPEC-001 (profile), SPEC-002 (notification_preferences), SPEC-004 (business details, payment terms), and SPEC-005/SPEC-006 (sign-in email/login method, signed-in devices). Fields touched by this feature: name, sign-in email, business_name, business_address, tax_id, default_payment_terms, notification_preferences, signed-in devices. (time_zone is also a field on this record, but it is set and owned by Currency & Tax Handling (FEAT-15) -- this feature does not expose a manual time-zone control, since the dependency map lists no FEAT-15 connection for FEAT-21.)

**Shared UI Patterns:**
- Settings section layout -- SPEC-001, SPEC-002, SPEC-003, and SPEC-004 share one consistent settings-page pattern (a persistent way to reach every other section, plus the "Close account" and "Branding" navigation items), all reachable from SPEC-001 as the entry point. Spec Writers for all four should describe the shared shell consistently rather than each inventing its own layout.
- Retry-preserving save -- the feature's own States field applies identically to every screen: a failed save is retried without discarding unsaved fields. Spec Writers for SPEC-001 through SPEC-004 should describe this Error-state behavior the same way.
- Read-only rendering for Dana -- SPEC-001, SPEC-002, and SPEC-004 each render the identical read-only treatment (no save controls, no destructive actions) inside a logged FEAT-31 support session, per SPEC-010.

**Shared Validation:**
- SPEC-007 defines field-level validation (required, format, value limits). SPEC-001 and SPEC-004 both reference it rather than duplicating field rules.
- SPEC-010 defines who may see or edit each screen (including that Owen and Priya never see any Settings screen, and that Dana never sees SPEC-003 or any sign-in credential). SPEC-001 through SPEC-004 all reference SPEC-010 for what to show, hide, or lock, rather than each screen defining its own authorization logic.

## Internal Dependency Map

```
SPEC-001 (Account Profile) -> [Nadia opens Settings] -> SPEC-001 (default entry)
SPEC-001 (Account Profile) -> [Nadia navigates to a section] -> SPEC-002 (Notification Preferences) / SPEC-003 (Login & Security) / SPEC-004 (Business Details & Payment Terms)
SPEC-001 (Account Profile) -> [Nadia saves her name/profile] -> SPEC-007 (Account Field Validation Rules) -> [valid] -> SPEC-001 (confirmation)
SPEC-002 (Notification Preferences) -> [Nadia toggles a preference] -> SPEC-008 (Notification Preference Rules) -> [allowed if optional] -> SPEC-002 (confirmation)
SPEC-003 (Login & Security) -> [Nadia submits new sign-in email/login method] -> SPEC-005 (Sign-In Email & Login Method Change)
SPEC-005 (Sign-In Email & Login Method Change) -> [change confirmed] -> SPEC-011 (Account-Critical Change Confirmation Email)
SPEC-005 (Sign-In Email & Login Method Change) -> [expired/abandoned] -> SPEC-003 (reverts, shows restart option)
SPEC-003 (Login & Security) -> [Nadia taps "Sign out other devices"] -> SPEC-006 (Sign-Out Other Sessions) -> [complete] -> SPEC-003 (updated session list)
SPEC-004 (Business Details & Payment Terms) -> [Nadia saves] -> SPEC-007 (Account Field Validation Rules) -> [valid] -> SPEC-009 (Business Details Completeness Gate) -> SPEC-004 (confirmation)
SPEC-001 (Account Profile) -> [Nadia taps "Close account"] -> FEAT-24 (cross-feature)
SPEC-001 (Account Profile) -> [Nadia resumes a skipped branding step] -> FEAT-19.SPEC-001 (cross-feature)
SPEC-010 (Settings Access & Read-Only Scope Rules) -> [governs entitlements referenced by] -> SPEC-001, SPEC-002, SPEC-003, SPEC-004
```

**Default Entry:** SPEC-001 (Account Profile) -- the screen Nadia reaches when she opens Settings; it is also the navigation hub for every other section in this feature and for the outbound routes to Branding (FEAT-19) and account closure (FEAT-24).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-21.SPEC-001 | Outbound | FEAT-19 (Freelancer Branding) | Nadia resumes a previously skipped branding step from Settings | Nadia taps "Branding" from the Settings entry point |
| FEAT-21.SPEC-001 | Outbound | FEAT-24 (Data Export & Account Deletion) | Nadia is routed to account closure rather than this feature duplicating that flow | Nadia taps "Close account" from Settings |
| FEAT-21.SPEC-002 | Outbound | FEAT-14 (Notifications/Email) | Notification preferences set here control which optional emails FEAT-14 sends; transactional emails always send regardless (XBR-30) | A preference is saved |
| FEAT-21.SPEC-005 / FEAT-21.SPEC-011 | Outbound | FEAT-14 (Notifications/Email) | The re-verification message and the account-critical change confirmation are both delivered, and delivery/bounce status reported, through the transactional email delivery capability FEAT-14 owns; this feature validates with no Integration spec of its own for that capability | A sign-in email/login method change starts or completes |
| FEAT-21.SPEC-004 / FEAT-21.SPEC-009 | Outbound | FEAT-09 (Invoice Generation & Sending) | Business details and default payment terms saved here print on every invoice; sending is blocked until both this feature's business details and the client's billing details exist (XBR-16) | An invoice is generated or a send is attempted |
| FEAT-21.SPEC-001 / FEAT-21.SPEC-002 / FEAT-21.SPEC-004 | Inbound | FEAT-31 (Operator Support Access) | Dana views profile, preference, and business-detail values read-only inside a logged support session; she never reaches Login & Security or any sign-in credential | Dana opens a support session and navigates to Settings |
| FEAT-21.SPEC-001 | Outbound | FEAT-23 (Subscription Plan & Billing Management) | Subscription and billing management reads the freelancer's account details (feature-dependency-map.md, Freelancer Account: "Read by ... FEAT-23") | Nadia views her subscription/billing area |
| FEAT-21.SPEC-001 | Outbound | FEAT-33 (Portal Referral Attribution) | Referral attribution reads the freelancer's account (feature-dependency-map.md, Freelancer Account: "Referenced by ... FEAT-33") | The referral mark's attribution is resolved |

## Non-Functional Notes

**Data volumes / growth:** Exactly one Freelancer Account record per freelancer, holding a small, bounded set of profile, business, and preference fields plus a short signed-in-devices list; volume tracks the expected few thousand freelancers in year one and carries no independent growth concern (feature-dependency-map.md, Entity: Freelancer Account).

**Responsiveness:** Settings is a standard configuration form used by Nadia only; it is expected to be reliably available and responsive during ordinary business use, with no specific uptime number committed (assumptions-constraints.md, ASMP-26). The feature's own States field rules out a Loading state beyond "a standard form" and requires that a failed save retry without discarding unsaved fields.

**Data sensitivity / privacy:** Freelancer Account holds personal data of the freelancer herself -- name, sign-in email, business address, tax ID -- treated as GDPR-class personal data, exportable and deletable on the freelancer's own request through FEAT-24 (assumptions-constraints.md, ASMP-23, ASMP-24). Sign-in credentials are never visible to the operator under any circumstance (feature-dependency-map.md, Entity: Freelancer Account, Data Sensitivity), enforced by SPEC-010's read-only scope rule.

**Compliance flags:** Business details captured here (business name, address, tax ID) exist specifically to satisfy invoice-content requirements (sequential number, both parties' business details, issue/due dates, tax line) that FEAT-09 must meet (assumptions-constraints.md, ASMP-24); no card or payment data is ever captured by this feature. Every screen follows the product's accessibility and degraded-state conventions -- readable on mobile, screen-reader and keyboard usable, real progress on loading, and a plain statement that saving needs a connection (assumptions-constraints.md, ASMP-27), consistent with this feature's own States field ("Offline-degraded: N/A -- settings changes require connectivity to persist").

## Non-Goals

- **Duplicating account closure or data export here** -- Excluded by the feature's own Primary Flows: "a freelancer wanting to close her account entirely is routed to Data Export & Account Deletion (FEAT-24) rather than duplicating that flow here." Settings only navigates to FEAT-24; it never deletes or exports the account itself.
- **A settings surface for client contacts** -- Excluded per the Access field: "client contacts have no equivalent settings surface," consistent with scope-boundaries.md SC-02 (only Primary and Reviewer client-contact roles exist, with no additional tiers or self-service account areas for them).
- **Agency, team, or staff-seat settings (e.g., a scoped bookkeeper role)** -- Excluded per scope-boundaries.md SC-01: the product has no internal-staff seat model; Nadia is the sole freelancer-side actor over her own account, and Dana's read-only support role (FEAT-31) is the only other internal identity, never an editor.
- **Operator (Dana) edits of any kind** -- Excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only; she can view profile, preference, and business-detail values but can never change them, and sign-in credentials are hidden from her entirely.
- **Manual time-zone control** -- The Freelancer Account record carries a time_zone field, but the dependency map lists no connection between this feature and Currency & Tax Handling (FEAT-15), which owns that field; this feature exposes no manual time-zone editor, leaving that behavior fully owned by FEAT-15.
- **Help-tip dismissal management surface** -- The Data Notes field lists help-tip dismissals as data this feature's account captures, but the dependency map marks that field "(FEAT-30, Later)"; FEAT-30 is a Later-phase feature not yet built, so this feature stores the field without exposing any management screen or control for it in this MVP scope.
- **Alternate or multiple sign-in methods (e.g., third-party single sign-on) for the freelancer** -- Adjacency exclusion: the persona set establishes that client contacts sign in only by passwordless magic link, and neither BRIEF.md nor the Access Matrix names any alternate authentication method for Nadia's own account; login recovery here covers a single email-based sign-in method only.
- **Automatic purge of the Freelancer Account or its settings history** -- Intentional lifecycle decision surfaced by the CRUD matrix: because deletion and retention are entirely FEAT-24's responsibility, this feature defines no independent retention or purge policy for the account or any of its settings fields.



# Screen Spec: Account Profile

## Overview

**Name:** Account Profile
**ID:** FEAT-21.SPEC-001
**Type:** Screen
**Purpose:** Nadia views and edits her name and account profile, and reaches every other Settings section (Notification Preferences, Login & Security, Business Details & Payment Terms, Branding, and account closure) from here; Dana views her name read-only inside a logged support session (never her sign-in email).
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying and editing Nadia's name and sign-in email display (the email value itself; changing it routes to FEAT-21.SPEC-003)
- The Settings entry shell shared by SPEC-001 through SPEC-004: a persistent way to reach every other section, plus "Branding" and "Close account" navigation items
- Saving profile edits immediately with retry-preserving error handling
- Read-only rendering of the name inside a logged Dana support session (FEAT-31), with no sign-in email row and no "Login & Security" or "Close account" shell items
- The account-critical change delivery-warning banner surfaced from FEAT-21.SPEC-011 (shown to Nadia only)

**Non-Goals:**
- Changing the sign-in email or login method -- that action is started on FEAT-21.SPEC-003 (Login & Security) and carried through by FEAT-21.SPEC-005; this screen only displays the current email as read display text, never as an editable email field
- Editing business details, payment terms, or notification preferences -- owned by FEAT-21.SPEC-004 and FEAT-21.SPEC-002 respectively; this screen only links to them
- Closing or deleting the account -- excluded per the Brief's own Primary Flows: "a freelancer wanting to close her account entirely is routed to Data Export & Account Deletion (FEAT-24) rather than duplicating that flow here"; this screen only navigates to FEAT-24
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02: the persona set establishes only Primary and Reviewer client-contact roles, with no self-service account areas for them

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Global navigation (any authenticated screen) | Nadia selects "Settings" | None -- profile loads from her own account |
| FEAT-21.SPEC-002 / FEAT-21.SPEC-003 / FEAT-21.SPEC-004 | Nadia selects "Profile" from the shared Settings shell | None -- returns to this default entry |
| FEAT-31 (Operator Support Access) | Dana opens a support session and navigates to Settings | Read-only support session context; profile renders without save controls |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | Edit name/profile, navigate to every Settings section, Branding, and Close account | -- |
| Dana (Support Operator) | Screen read-only, inside a logged FEAT-31 support session: the name only -- the sign-in email row, the "Change" link, the "Login & Security" shell item, the "Close account" shell item, and the delivery-warning banner are never rendered (FEAT-21.SPEC-010) | None -- no save controls, no destructive actions shown | Editing controls are not rendered at all for Dana; a direct attempt to submit a change is refused with "Support sessions are read-only." |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, Nadia lands on this screen if Settings was her original destination |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved profile edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Settings" with Nadia's account name as a subheading.

**Navigation shell (shared across SPEC-001 through SPEC-004):** A persistent list of Settings sections, positioned to the left of the main content on wider layouts and as a top row of tabs on narrower ones: "Profile" (this screen, selected), "Notification Preferences" (FEAT-21.SPEC-002), "Login & Security" (FEAT-21.SPEC-003), "Business Details & Payment Terms" (FEAT-21.SPEC-004), "Payment account" (FEAT-32.SPEC-001), "Branding" (FEAT-19.SPEC-001), and "Close account" (routes to FEAT-24) at the bottom of the shell, visually separated from the other items.

**Body:** A single-column form with the following fields:
- Name (text input, required)
- Sign-in email (read-only display text, with a "Change" link that navigates to FEAT-21.SPEC-003) -- Nadia only

Below the form, a "Save" action button.

**Delivery-warning banner (Nadia only):** When FEAT-21.SPEC-011 reports that a security confirmation email for a sign-in email change failed after all retries, a warning banner appears at the top of the body, above the form, reading "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." with a "Go to Login & Security" link (navigates to FEAT-21.SPEC-003) and a "Dismiss" button. The banner names no email address. Several unresolved failures collapse into this single banner.

**Dana's rendering:** shows only the Name value as read-only text -- no sign-in email row, no "Save" button, no "Change" link, no delivery-warning banner -- and the navigation shell is present but omits "Login & Security", "Payment account", and "Close account" (sign-in credentials are never visible to the operator, account closure is never available inside a support session per FEAT-21.SPEC-010, and Dana never reaches the payment connection screen -- her read-only connection status lives on her own FEAT-31 support-session surface, per FEAT-32.SPEC-001).

**Footer:** None -- Save is directly below the form.

### Responsive Behavior

- **Compact breakpoint:** The Settings navigation shell collapses to a horizontal scrolling tab row above the form; the form itself is single-column, full width.
- **Medium size class and above:** The navigation shell renders as a fixed left-hand column; the form is capped at a consistent platform-wide form width and sits to its right. No structural change to the form beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Navigation shell item | Tap | Navigate to the corresponding Settings section (FEAT-21.SPEC-002, SPEC-003, SPEC-004) or FEAT-19.SPEC-001 (Branding) | Screen changes to the selected section | Selected item highlighted in the shell |
| "Payment account" item | Tap | Navigate to FEAT-32.SPEC-001 (Payment Account connection screen); Nadia only | Screen leaves the Settings profile view; FEAT-32.SPEC-001's back arrow returns her to this screen (FEAT-21.SPEC-001) | Standard navigation transition |
| "Close account" item | Tap | Navigate to FEAT-24 (Data Export & Account Deletion) | Screen leaves Settings | Standard navigation transition |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers validation via FEAT-21.SPEC-007 | Error state on field | "Name is required" below field |
| "Change" link (sign-in email) | Tap | Navigate to FEAT-21.SPEC-003 (Login & Security) | Screen changes | Standard navigation transition |
| Save button | Tap | Validate the name field via FEAT-21.SPEC-007; if valid, save the profile | Button shows loading state during save | Success: toast "Profile updated" and the form reflects the saved value. Failure: inline error banner, retry option, entered text preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| Delivery-warning banner: "Go to Login & Security" | Tap | Navigate to FEAT-21.SPEC-003 | Screen changes; banner stays until dismissed | Standard navigation transition |
| Delivery-warning banner: "Dismiss" | Tap | Marks the reported delivery failure(s) as acknowledged | Banner is removed and does not return for those failures | Banner disappears; removal announced to assistive technology |

### Accessibility Notes

- **Focus order:** Navigation shell items (in listed order) -> delivery-warning banner ("Go to Login & Security", "Dismiss"; when shown) -> Name input -> "Change" link -> Save button.
- **Validation announcements:** When the name field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Profile updated" toast is announced on success; on validation failure, focus moves to the name field.
- **Keyboard alternatives:** Every action on this screen, including navigation shell selection, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form pre-filled with Nadia's current name and sign-in email, Save button enabled | Screen opens | User begins editing |
| Editing | Name field shows in-progress input, Save button enabled | User types in the name field | User taps Save or navigates away |
| Saving | Save button shows a loading spinner, name field disabled | User taps Save with valid input | Save completes or fails |
| Validation Error | Name field highlighted with error message below it | Name field is empty on blur or submit | User enters a valid name and re-triggers validation |
| Error | Error banner at top of form: "Could not save your profile. Check your connection and try again." with a Retry button | Save operation fails server-side | User taps Retry; entered name is preserved and resubmitted |
| Delivery warning | Warning banner above the form (text in Layout and Content), form otherwise as Loaded | FEAT-21.SPEC-011 reports a confirmation-email delivery failure after final retry and Nadia has not dismissed it | Nadia taps "Dismiss" |
| Read-only (Dana) | Name shown as read-only text; no sign-in email row, no Save button, no "Change" link, no delivery-warning banner | Dana opens this screen inside a logged FEAT-31 support session | Dana closes the support session or navigates away |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); the form remains visible with current values but Save is disabled and a banner reads "You're offline. Reconnect to save changes." | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; no queued submission occurs |

## Validation Rules

Validation governed by FEAT-21.SPEC-007 (Account Field Validation Rules). See that spec for the name field's required/format rules. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Notification Preferences" | FEAT-21.SPEC-002 (Notification Preferences) | -- |
| Navigation shell: "Login & Security" | FEAT-21.SPEC-003 (Login & Security) | -- |
| Navigation shell: "Business Details & Payment Terms" | FEAT-21.SPEC-004 (Business Details & Payment Terms) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |
| "Change" link (sign-in email) | FEAT-21.SPEC-003 (Login & Security) | -- |
| Successful save | Stays on FEAT-21.SPEC-001 with the confirmation toast | -- |

## Data Model

**Creates:** None.
**Reads:** Freelancer Account -- `name`, sign-in email (displayed read-only to Nadia only; never read into Dana's rendering, per the dependency map's Data Sensitivity note "sign-in credentials never visible to the operator"); the unacknowledged delivery-failure flag reported by FEAT-21.SPEC-011 (Nadia only).
**Updates (acknowledgement):** the delivery-failure flag is cleared when Nadia taps "Dismiss".
**Updates:** Freelancer Account -- `name`.
**Deletes:** None.

## Business Rules

- Field validation (FEAT-21.SPEC-007) is enforced before any save completes -- the user cannot save with an empty name.
- Access and read-only scope (FEAT-21.SPEC-010) governs what Dana sees and cannot act on -- this screen's Access and Visibility table is consistent with that spec: Dana sees the name only, never the sign-in email, and never the "Login & Security" or "Close account" shell items.
- The delivery-warning banner is driven solely by FEAT-21.SPEC-011's failure report; it persists across visits until Nadia dismisses it and is never rendered for Dana.
- The sign-in email itself is never directly editable here; XBR-30 and the Freelancer Account entity's Contention note require the dedicated re-verification path (FEAT-21.SPEC-003 -> FEAT-21.SPEC-005) for any email/login-method change.
- Two open sessions of Nadia's editing the name field resolve last-write-wins, per the dependency map's Contention note for the Freelancer Account entity.

## Edge Cases

- **Nadia navigates away with an unsaved name edit** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Save fails server-side** -- Error banner: "Could not save your profile. Check your connection and try again." with a Retry button; the entered name is preserved and retried without being discarded (product-features.md, States field).
- **Name changed by Nadia in a second open session while this screen is open** -- Save is last-write-wins per the dependency map's Contention note for the Freelancer Account entity: Nadia's save here overwrites whatever the other session saved, with no merge and no warning dialog, consistent with "None across roles ... two open sessions of Nadia's resolve last-write-wins per field."
- **A second confirmation-email delivery failure is reported while the banner is already showing** -- The single banner remains; no second banner is added, and one "Dismiss" acknowledges both.
- **Dana's support session ends while she is viewing this screen** -- The screen redirects her out of the freelancer's account view; no data was ever editable so nothing is lost.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | Name field validation rules |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Governs Dana's read-only rendering and Owen/Priya's total exclusion |
| FEAT-21.SPEC-002 (Notification Preferences) | Navigation (outbound/inbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound/inbound) | Shared Settings navigation shell; "Change" link for sign-in email |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | Navigation (outbound/inbound) | Shared Settings navigation shell |
| FEAT-32.SPEC-001 (Payment Account connection screen) | Navigation (outbound) | "Payment account" navigation item, cross-feature; its back arrow returns to Settings (FEAT-21.SPEC-001) |
| FEAT-19.SPEC-001 (Branding Settings) | Navigation (outbound) | "Branding" navigation item; Nadia resumes a skipped branding step |
| FEAT-24 (Data Export & Account Deletion) | Navigation (outbound) | "Close account" navigation item, cross-feature |
| FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | References (inbound) | Reports confirmation-email delivery failures that this screen surfaces as the delivery-warning banner |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support session renders this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| settings_updated | section: profile; field changed: name | Nadia's profile save completes successfully | N/A -- no success-metrics.md metric is connected to Settings & Account Management (success-metrics.md carries no Connected Feature entry for FEAT-21); retained per product-features.md's own Signals field ("settings_updated") so the save is observable |
| settings_save_failed | section: profile | Save fails server-side | N/A -- no connected success-metrics.md metric; retained to make retry-preserving save failures observable rather than silent |

## Acceptance Criteria

**FEAT-21.SPEC-001-AC-01:** Given Nadia is on the Account Profile screen, when she clears the name field and taps Save, then the name field shows an error state with the message "Name is required" and the save does not proceed.

**FEAT-21.SPEC-001-AC-02:** Given Nadia enters "Nadia Voss" as her name and taps Save, then the profile saves, a "Profile updated" toast appears, and the field shows "Nadia Voss".

**FEAT-21.SPEC-001-AC-03:** Given Nadia is on the Account Profile screen, when she taps "Change" next to her sign-in email, then she is navigated to FEAT-21.SPEC-003 (Login & Security).

**FEAT-21.SPEC-001-AC-04:** Given Nadia taps "Close account", then she is navigated to FEAT-24 (Data Export & Account Deletion) and no closure or deletion happens inside this screen.

**FEAT-21.SPEC-001-AC-05:** Given Nadia taps "Branding" from the navigation shell, then she is navigated to FEAT-19.SPEC-001 (Branding Settings).

**FEAT-21.SPEC-001-AC-06:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the profile, then she sees the current name only -- no sign-in email row, no Save button, no "Change" link, no delivery-warning banner, and neither a "Login & Security" nor a "Close account" item in the navigation shell.

**FEAT-21.SPEC-001-AC-07:** Given Dana (Support Operator) is viewing the profile read-only, when she attempts to submit a change through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-001-AC-08:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-001-AC-09:** Given Nadia's profile save fails server-side, when the failure occurs, then the error banner "Could not save your profile. Check your connection and try again." appears with a Retry button, and her entered name remains in the field.

**FEAT-21.SPEC-001-AC-10:** Given Nadia has an unsaved name edit, when she navigates away, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-21.SPEC-001-AC-11:** Given Nadia saves her name from a second open session while this screen is also open in a first session with a different unsaved name, when the first session's Save is tapped, then the first session's value overwrites the second session's saved value (last-write-wins), with no merge dialog shown.

**FEAT-21.SPEC-001-AC-12:** Given Nadia loses connectivity while on this screen, then the Save button is disabled and a banner reads "You're offline. Reconnect to save changes."

**FEAT-21.SPEC-001-AC-13:** Given Nadia's expired session is detected while she has an unsaved name edit, when the "Your session has expired" dialog is dismissed by signing in again, then her unsaved name edit is restored on this screen.

**FEAT-21.SPEC-001-AC-14:** Given FEAT-21.SPEC-011 has reported a confirmation-email delivery failure after final retry, when Nadia opens this screen, then the banner "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." appears above the form with "Go to Login & Security" and "Dismiss".

**FEAT-21.SPEC-001-AC-15:** Given the delivery-warning banner is showing, when Nadia taps "Go to Login & Security", then she is navigated to FEAT-21.SPEC-003; when she taps "Dismiss", then the banner is removed and does not reappear on later visits for that failure.

**FEAT-21.SPEC-001-AC-16:** Given a delivery failure is unacknowledged, when Dana (Support Operator) opens this screen inside a logged support session, then no delivery-warning banner is rendered for her.

**FEAT-21.SPEC-001-AC-17:** Given Nadia is on the Account Profile screen, when she taps "Payment account" in the navigation shell, then she is navigated to FEAT-32.SPEC-001 (Payment Account connection screen), and when she taps that screen's back arrow she returns to this screen; and given Dana (Support Operator) is inside a logged support session, then no "Payment account" item is rendered in her navigation shell.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 8 (loaded, editing, saving, validation error, error, delivery warning, read-only (Dana), offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Notification Preferences

## Overview

**Name:** Notification Preferences
**ID:** FEAT-21.SPEC-002
**Type:** Screen
**Purpose:** Nadia turns optional notifications on or off, with transactional record emails always shown locked-on; Dana views the same preferences read-only inside a logged support session.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying every notification type the product defines, grouped as optional (toggleable) or transactional (locked-on)
- Toggling an optional notification type on or off, saved immediately
- Read-only rendering of the identical preference list inside a logged Dana support session (FEAT-31)

**Non-Goals:**
- Deciding which notification types exist or their delivery content -- owned by Notifications (Email) (FEAT-14); this screen only reads the type list and writes the on/off preference
- Allowing any preference to disable a transactional record email -- excluded per XBR-30: "Notification preferences can switch off only optional emails; transactional emails core to the record ... always send"; enforced by FEAT-21.SPEC-008
- In-app or push notification channel preferences -- product-features.md and the dependency map define email as the product's sole notification channel; no other channel exists to have a preference
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02, consistent with FEAT-21.SPEC-010

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia selects "Notification Preferences" from the Settings navigation shell | None |
| FEAT-31 (Operator Support Access) | Dana opens a support session and navigates to Notification Preferences | Read-only support session context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own preferences only | Toggle any optional notification type | -- |
| Dana (Support Operator) | Full screen, read-only, inside a logged FEAT-31 support session | None -- toggles rendered as disabled, non-interactive indicators (FEAT-21.SPEC-010) | A direct attempt to change a toggle is refused with "Support sessions are read-only." |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress toggle (already saved instantly, see Interactions) requires no restoration since each toggle persists on change |

## Layout and Content

**Header:** Screen title "Settings" with the shared Settings navigation shell (as described in FEAT-21.SPEC-001), "Notification Preferences" selected.

**Body:** A list of notification types grouped under two headings:
- **Transactional (always on):** one row per transactional notification type (e.g., payment confirmation), each showing the type's name and a locked toggle in the on position with a "Always sent" label beside it -- no interaction available.
- **Optional:** one row per optional notification type, each showing the type's name, a short one-line description of what it notifies about, and an on/off toggle.

For Dana's read-only rendering, both groups render identically but every toggle (locked or optional) is shown as a static, disabled indicator with no interaction affordance.

**Footer:** None -- each toggle saves immediately on change; there is no separate Save action.

### Responsive Behavior

- **Compact breakpoint:** Notification type rows stack in a single column, full width, with the toggle right-aligned on each row.
- **Medium size class and above:** Same single-column list, capped at the platform-wide form width and horizontally centered alongside the navigation shell; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Optional notification toggle | Tap | Validates via FEAT-21.SPEC-008 that the type is optional, then saves the new on/off state immediately | Toggle switches to the new position; shows a brief loading indicator during save | Success: toggle settles in new position with a small "Saved" indicator that fades. Failure: toggle reverts to its prior position and an inline error appears next to the row |
| Transactional notification indicator | Tap | No action -- non-interactive | None | No visual change; the row's "Always sent" label communicates why it cannot be toggled |
| Optional notification toggle (while saving) | Tap | No action -- debounced | None | Toggle remains in its transitional loading state |

### Accessibility Notes

- **Focus order:** Navigation shell items -> Transactional rows (announced as non-interactive) -> Optional toggles, in the order listed on screen.
- **Toggle announcements:** Each toggle's new state and the "Saved" confirmation are announced to assistive technology when a save completes; a save failure announces the reverted state and the inline error message.
- **Locked toggle:** Transactional toggles are exposed to assistive technology as disabled controls with the accessible label "Always sent -- cannot be turned off."
- **Keyboard alternatives:** Every optional toggle is operable by keyboard (space/enter); there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | All notification types listed with their current on/off state | Screen opens | Always the resting state between toggles |
| Toggle Saving | The toggled row shows a brief loading indicator | User taps an optional toggle | Save completes or fails |
| Toggle Save Error | The toggled row reverts to its prior state with an inline error message: "Could not save. Try again." | Save operation fails server-side | User re-taps the toggle and the retry succeeds |
| Read-only (Dana) | All toggles shown as static disabled indicators | Dana opens this screen inside a logged FEAT-31 support session | Dana closes the support session or navigates away |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); toggles are shown disabled with a banner "You're offline. Reconnect to change preferences." | Connectivity lost while screen is open | Connectivity restored -- toggles re-enable |

## Validation Rules

Validation governed by FEAT-21.SPEC-008 (Notification Preference Rules): a toggle attempt on a transactional type is rejected before any save request is made, since the control is non-interactive by construction. This screen applies FEAT-21.SPEC-008's optional/transactional distinction on render (determining which rows show a toggle vs. a locked indicator) and re-checks it on toggle before saving.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Profile" | FEAT-21.SPEC-001 (Account Profile) | -- |
| Navigation shell: "Login & Security" | FEAT-21.SPEC-003 (Login & Security) | -- |
| Navigation shell: "Business Details & Payment Terms" | FEAT-21.SPEC-004 (Business Details & Payment Terms) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |

## Data Model

**Creates:** None.
**Reads:** Notification -- the set of `notification_type` values and which are optional vs. transactional (dependency map, Referenced Entities: "the preferences screen reads the set of notification types ... to build the toggle list; it never reads or writes individual Notification instances"). Freelancer Account -- `notification_preferences` (current on/off state per optional type).
**Updates:** Freelancer Account -- `notification_preferences` (one optional type's on/off state per toggle).
**Deletes:** None.

## Business Rules

- Only optional notification types can be toggled; transactional types are always shown locked-on (XBR-30, FEAT-21.SPEC-008).
- Each toggle saves immediately and independently -- there is no batch save or unsaved-changes state on this screen.
- Access and read-only scope (FEAT-21.SPEC-010) governs Dana's non-interactive rendering.
- A saved preference change takes effect for the next notification of that type sent by FEAT-14 (dependency map, Cross-Feature Touchpoints: "Notification preferences set here control which optional emails FEAT-14 sends").

## Edge Cases

- **Nadia toggles the same preference twice rapidly** -- The second tap is ignored while the first save is in progress (row in loading state).
- **Toggle save fails server-side** -- The toggle reverts to its prior position with the inline error "Could not save. Try again."; the row remains interactive for a retry.
- **The set of notification types changes (a new optional type is added by FEAT-14) while this screen is open** -- The screen reflects the type list as of load; a new type appears the next time the screen is opened or refreshed, with its default preference value applied until Nadia changes it.
- **Nadia toggles a preference in one open session while another of her sessions has the same screen open** -- The other open session's toggle reflects the new state on its next refresh (this is a live preference value, not a form draft, so no merge conflict arises); the dependency map's Contention note for Freelancer Account applies last-write-wins per field, and here each toggle is its own field-level write.
- **Dana's support session ends while she is viewing this screen** -- The screen redirects her out of the freelancer's account view; no toggle was ever interactive so nothing is lost.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-008 (Notification Preference Rules) | References (inbound) | Determines which types are optional vs. transactional and enforces the toggle restriction |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Governs Dana's read-only rendering and Owen/Priya's total exclusion |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Shared Settings navigation shell entry point |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-14 (Notifications (Email)) | References (outbound) | Consumes the toggled preference when deciding whether to send an optional notification |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support session renders this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| notification_preference_changed | notification_type, new_state: on/off | An optional toggle save completes successfully | N/A -- no success-metrics.md metric is connected to Settings & Account Management or to Notifications (Email)'s FEAT-21 side; retained per product-features.md's own Signals field ("notification_preference_changed") so the change is observable |
| notification_preference_save_failed | notification_type | A toggle save fails server-side | N/A -- no connected success-metrics.md metric; retained to make save failures observable rather than silent |

## Acceptance Criteria

**FEAT-21.SPEC-002-AC-01:** Given Nadia is on the Notification Preferences screen, when she views a transactional notification type, then it shows locked-on with an "Always sent" label and no toggle interaction is available.

**FEAT-21.SPEC-002-AC-02:** Given Nadia is on the Notification Preferences screen, when she turns an optional notification type off, then the toggle switches off, briefly shows "Saved", and the preference is saved.

**FEAT-21.SPEC-002-AC-03:** Given Nadia turns an optional notification type back on, then the toggle switches on and is saved.

**FEAT-21.SPEC-002-AC-04:** Given Nadia's toggle save fails server-side, when the failure occurs, then the toggle reverts to its prior position and shows "Could not save. Try again."

**FEAT-21.SPEC-002-AC-05:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the preferences, then every toggle -- transactional and optional -- is shown as a static, disabled indicator.

**FEAT-21.SPEC-002-AC-06:** Given Dana (Support Operator) is viewing preferences read-only, when she attempts to change a toggle through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-002-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-002-AC-08:** Given Nadia taps the same optional toggle twice rapidly, when the first save is still in progress, then the second tap is ignored.

**FEAT-21.SPEC-002-AC-09:** Given Nadia loses connectivity on this screen, then all toggles are disabled and a banner reads "You're offline. Reconnect to change preferences."

**FEAT-21.SPEC-002-AC-10:** Given a new optional notification type is added by FEAT-14 while Nadia's screen is already open, when she reopens or refreshes the screen, then the new type appears with its default preference applied.

**FEAT-21.SPEC-002-AC-11:** Given Nadia toggles a preference in one open session, when a second open session of hers with the same screen is refreshed, then it shows the newly saved state for that preference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (loaded, saving, save error, read-only, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Login & Security

## Overview

**Name:** Login & Security
**ID:** FEAT-21.SPEC-003
**Type:** Screen
**Purpose:** Nadia starts a sign-in email or login method change, views and manages her signed-in devices, and signs out other sessions; never surfaced to Dana or any client contact.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Starting a new sign-in email or login method change, handed off to FEAT-21.SPEC-005 for re-verification
- Showing the state of a pending, unconfirmed change (in progress, with a resend or cancel option)
- Listing Nadia's signed-in devices (current session marked, others listed)
- Triggering "Sign out other devices" (FEAT-21.SPEC-006)

**Non-Goals:**
- Carrying out the re-verification itself (sending the link/code, confirming it, expiring it) -- owned entirely by FEAT-21.SPEC-005; this screen only starts the change and reflects its pending/confirmed/expired state
- Alternate or multiple sign-in methods (e.g., third-party single sign-on) -- excluded per the Brief's Non-Goals: "neither BRIEF.md nor the Access Matrix names any alternate authentication method for Nadia's own account; login recovery here covers a single email-based sign-in method only"
- Any visibility for Dana -- excluded per scope-boundaries.md SC-04 and FEAT-21.SPEC-010: "Dana never sees ... any sign-in credential"; this screen is never rendered inside a support session
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02, consistent with FEAT-21.SPEC-010

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia selects "Login & Security" from the Settings navigation shell | None |
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps "Change" next to her sign-in email | None -- lands on this screen ready to start a change |
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | The pending change expires or Nadia abandons it | Reverted state -- the prior sign-in email/login method is shown active again with a restart option |
| FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | Nadia taps the email's "Go to Login & Security" CTA | None -- the screen loads her account's current sign-in email, login method, and devices |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Start a sign-in email/login method change, view signed-in devices, sign out other devices | -- |
| Dana (Support Operator) | No -- this screen is never rendered inside a support session | No | The Settings navigation shell shown to Dana omits "Login & Security" entirely (FEAT-21.SPEC-010); there is no direct-link path into this screen from a support session |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress, not-yet-submitted email change entry is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Settings" with the shared Settings navigation shell (as described in FEAT-21.SPEC-001), "Login & Security" selected.

**Body, section 1 -- Sign-in email and login method:**
- Current sign-in email (read-only display text)
- "Change sign-in email" action button
- If a change is pending (per FEAT-21.SPEC-005): a status banner showing "Verification pending for {new email}" with "Resend link", "Use a different email", and "Cancel" actions, replacing the "Change sign-in email" button for the duration of the pending state. "Use a different email" is the only control for starting a replacement change while one is pending.

**Body, section 2 -- Signed-in devices:**
- A list of Nadia's active sessions, each row showing: device/browser description, approximate location, and last-active time; the current session is labeled "This device" and cannot be individually signed out from this list
- "Sign out other devices" action button, shown only when at least one other session exists

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Both sections stack vertically, full width; the device list rows stack their device description above location/last-active on narrow widths.
- **Medium size class and above:** Both sections remain single-column, capped at the platform-wide form width alongside the navigation shell; device list rows show device description, location, and last-active in one row.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Change sign-in email" button | Tap | Opens an inline form to enter a new sign-in email | Form appears below the button | Form fields ready for input |
| New sign-in email field | Type / Submit | On submit, triggers FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) via validation from FEAT-21.SPEC-007 | Screen enters the pending state | Success: banner "Verification pending for {new email}" with instructions to check that inbox. Failure: inline error from FEAT-21.SPEC-007 |
| "Resend link" (pending state) | Tap | Triggers FEAT-21.SPEC-005's resend path | Button briefly disabled | Confirmation text "A new verification link has been sent." |
| "Use a different email" (pending state) | Tap | Opens the same inline new-email form as "Change sign-in email"; submitting it triggers FEAT-21.SPEC-005's new-submission path, which replaces the pending change | Inline form appears below the banner; the pending banner stays until the replacement is accepted | On accept: banner updates to "Verification pending for {newest email}". Failure: inline error from FEAT-21.SPEC-007 and the existing pending change is untouched |
| "Cancel" (pending state) | Tap | Triggers FEAT-21.SPEC-005's abandon path | Pending banner clears, prior state restored | Toast "Sign-in email change cancelled." |
| "Sign out other devices" button | Tap | Confirmation dialog, then triggers FEAT-21.SPEC-006 (Sign-Out Other Sessions) | Button shows loading state during processing | Success: toast "All other sessions signed out." and the device list refreshes to show only "This device". Failure: inline error, device list unchanged |

### Accessibility Notes

- **Focus order:** Navigation shell -> "Change sign-in email" (or the pending banner's "Resend link"/"Use a different email"/"Cancel") -> new-email input when open -> device list rows -> "Sign out other devices".
- **Pending-state announcements:** Entering or leaving the pending state is announced to assistive technology (the pending banner's appearance and its removal on cancel/confirm/expiry).
- **Sign-out feedback:** The "All other sessions signed out" confirmation is announced on success; a failure announcement names the reason from FEAT-21.SPEC-006's outcome.
- **Keyboard alternatives:** Every action on this screen, including opening the email-change form and triggering sign-out, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Current sign-in email shown with "Change sign-in email" button; device list loaded | Screen opens with no pending change | User starts a change, or a change becomes pending from elsewhere |
| Change form open | Inline new-email input visible below the "Change sign-in email" button | User taps "Change sign-in email" | User submits, or collapses the form |
| Pending verification | Banner "Verification pending for {new email}" with "Resend link", "Use a different email", and "Cancel"; the "Change sign-in email" button is not shown | FEAT-21.SPEC-005 accepts the submitted change | FEAT-21.SPEC-005 confirms, the change expires, or Nadia cancels; "Use a different email" keeps this state with the newest target |
| Sign-out processing | "Sign out other devices" button shows a loading spinner, device list temporarily non-interactive | User confirms "Sign out other devices" | FEAT-21.SPEC-006 completes or fails |
| Error | Inline error banner: "Could not complete this action. Check your connection and try again." with Retry | Change submission, resend, cancel, or sign-out fails server-side | User taps Retry |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); action buttons are disabled with a banner "You're offline. Reconnect to manage login and security." | Connectivity lost while screen is open | Connectivity restored -- actions re-enable |

## Validation Rules

Validation governed by FEAT-21.SPEC-007 (Account Field Validation Rules) for the new sign-in email's format. This screen applies that validation on submit, before handing off to FEAT-21.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Profile" | FEAT-21.SPEC-001 (Account Profile) | -- |
| Navigation shell: "Notification Preferences" | FEAT-21.SPEC-002 (Notification Preferences) | -- |
| Navigation shell: "Business Details & Payment Terms" | FEAT-21.SPEC-004 (Business Details & Payment Terms) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |

## Data Model

**Creates:** None directly -- submitting a new sign-in email creates a pending change record owned by FEAT-21.SPEC-005.
**Reads:** Freelancer Account -- current sign-in email, signed-in devices list.
**Updates:** None directly -- "Change sign-in email" and "Sign out other devices" delegate their writes to FEAT-21.SPEC-005 and FEAT-21.SPEC-006 respectively.
**Deletes:** None.

## Business Rules

- The sign-in email/login method change is never applied directly from this screen -- it always goes through FEAT-21.SPEC-005's re-verification process (dependency map, Freelancer Account Contention: "the sign-in email change, which requires re-verification before it takes effect").
- Only one pending change can exist at a time; starting a replacement change while one is pending (via the banner's "Use a different email" control) replaces the pending target email (enforced by FEAT-21.SPEC-005 and reflected here as the pending banner updating to the newest target).
- "Sign out other devices" never signs out the current session -- only FEAT-21.SPEC-006's scope (every session except the current one).
- Access and read-only scope (FEAT-21.SPEC-010) excludes this entire screen from Dana's support session and from every client contact role.

## Edge Cases

- **Nadia submits the same email currently active as sign-in email** -- Rejected inline with "This is already your sign-in email." per FEAT-21.SPEC-007; no pending change is created.
- **Nadia starts a second email change while one is already pending (via "Use a different email" on the pending banner)** -- The pending banner updates to reflect the newest target email; the prior pending verification link/code is invalidated by FEAT-21.SPEC-005.
- **The pending change expires while Nadia is on this screen** -- The pending banner clears automatically and the "Change sign-in email" button returns, per FEAT-21.SPEC-005's expiry outcome; no data loss occurs since the prior email remains active throughout.
- **Nadia taps "Sign out other devices" with only her current session active** -- The button is not shown (per Layout and Content: "shown only when at least one other session exists").
- **A device Nadia signs out reconnects mid-action** -- The sign-out is processed against the session list as it stood when FEAT-21.SPEC-006 began; a session that reconnects after the invalidation is treated as a fresh unauthenticated session and must sign in again.
- **Nadia's own current session is somehow included in a stale device-list snapshot** -- The current session is always excluded from "Sign out other devices" by construction (FEAT-21.SPEC-006's scope), regardless of list staleness.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Triggers (outbound) | "Change sign-in email" submission starts the re-verification process |
| FEAT-21.SPEC-006 (Sign-Out Other Sessions) | Triggers (outbound) | "Sign out other devices" invalidates every other session |
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | New sign-in email format validation |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Excludes this screen from Dana's support session entirely |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Shared Settings navigation shell entry point and "Change" link |
| FEAT-21.SPEC-002 (Notification Preferences) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | Navigation (outbound) | Shared Settings navigation shell |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_email_change_started | N/A -- no personal data in the event payload beyond the fact of the attempt | Nadia submits a new sign-in email and FEAT-21.SPEC-005 accepts it as pending | N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("account_email_changed" family) so the start of the flow is observable |
| other_sessions_signed_out | session_count | FEAT-21.SPEC-006 completes successfully | N/A -- no connected success-metrics.md metric; retained per product-features.md's own Signals field so this security action is observable |

## Acceptance Criteria

**FEAT-21.SPEC-003-AC-01:** Given Nadia is on the Login & Security screen, when she taps "Change sign-in email" and enters a new valid email, then FEAT-21.SPEC-005 starts the change and the screen shows "Verification pending for {new email}".

**FEAT-21.SPEC-003-AC-02:** Given Nadia enters her currently active sign-in email as the "new" email, when she submits, then she sees "This is already your sign-in email." and no pending change is created.

**FEAT-21.SPEC-003-AC-03:** Given Nadia has a pending sign-in email change, when she taps "Resend link", then a new verification link is sent and she sees "A new verification link has been sent."

**FEAT-21.SPEC-003-AC-04:** Given Nadia has a pending sign-in email change, when she taps "Cancel", then the pending banner clears, she sees "Sign-in email change cancelled.", and her prior sign-in email remains active.

**FEAT-21.SPEC-003-AC-05:** Given Nadia has a pending change, when she taps "Use a different email" on the pending banner and submits a second, different valid email, then the pending banner updates to "Verification pending for {newest email}" and the "Change sign-in email" button remains hidden.

**FEAT-21.SPEC-003-AC-06:** Given Nadia's pending change expires while she is on this screen, then the pending banner clears automatically and the "Change sign-in email" button reappears.

**FEAT-21.SPEC-003-AC-07:** Given Nadia is signed in on two other devices in addition to her current one, when she opens this screen, then the device list shows all three sessions with the current one labeled "This device".

**FEAT-21.SPEC-003-AC-08:** Given Nadia has one other active session, when she taps "Sign out other devices" and confirms, then FEAT-21.SPEC-006 runs and the device list refreshes to show only "This device", with the toast "All other sessions signed out."

**FEAT-21.SPEC-003-AC-09:** Given Nadia has no other active sessions, when she views this screen, then "Sign out other devices" is not shown.

**FEAT-21.SPEC-003-AC-10:** Given Dana (Support Operator) is inside a logged FEAT-31 support session, when she views the Settings navigation shell, then "Login & Security" is not listed and this screen is not reachable.

**FEAT-21.SPEC-003-AC-11:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-003-AC-12:** Given Nadia's email-change submission fails server-side, when the failure occurs, then the error banner "Could not complete this action. Check your connection and try again." appears with a Retry button.

**FEAT-21.SPEC-003-AC-13:** Given Nadia loses connectivity on this screen, then all actions are disabled and a banner reads "You're offline. Reconnect to manage login and security."

**FEAT-21.SPEC-003-AC-14:** Given Nadia's expired session is detected while she has an unsubmitted new-email entry in the open change form, when she signs in again, then the entry is restored on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (default, change form open, pending, sign-out processing, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Business Details & Payment Terms

## Overview

**Name:** Business Details & Payment Terms
**ID:** FEAT-21.SPEC-004
**Type:** Screen
**Purpose:** Nadia sets her business name, address, tax ID, and default payment terms that print on every invoice; Dana views the same values read-only inside a logged support session.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying and editing business name, business address, tax ID, and default payment terms
- Showing whether business details are complete enough to send a first invoice (per FEAT-21.SPEC-009)
- Saving edits immediately with retry-preserving error handling
- Read-only rendering of the identical fields inside a logged Dana support session (FEAT-31)

**Non-Goals:**
- Deciding whether an invoice send is blocked -- owned by FEAT-21.SPEC-009 (Business Details Completeness Gate), which FEAT-09 (Invoice Generation & Sending) checks at send time; this screen only displays completeness status and lets Nadia fill the gap
- Currency and tax-rate configuration -- owned by Currency & Tax Handling (FEAT-15) at the project level; the tax_id field here is the freelancer's own tax identifier printed on invoices, a distinct field from the project's tax label/rate
- Per-project payment schedule or milestone pricing -- owned by Milestone & Payment Schedule Setup (FEAT-04); "default payment terms" here is the account-wide due-date default, not a project-specific schedule
- A settings surface for client contacts (Owen, Priya) -- excluded per scope-boundaries.md SC-02, consistent with FEAT-21.SPEC-010

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia selects "Business Details & Payment Terms" from the Settings navigation shell | None |
| FEAT-21.SPEC-009 (Business Details Completeness Gate) | Nadia is prompted to complete business details after a blocked first-invoice send attempt (FEAT-09) | A "complete your business details" prompt context, highlighting the incomplete fields |
| FEAT-31 (Operator Support Access) | Dana opens a support session and navigates to Business Details & Payment Terms | Read-only support session context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | Edit business name, address, tax ID, default payment terms | -- |
| Dana (Support Operator) | Full screen, read-only, inside a logged FEAT-31 support session | None -- no save controls (FEAT-21.SPEC-010) | Editing controls are not rendered at all; a direct attempt to submit a change is refused with "Support sessions are read-only." |
| Owen (Client Primary Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Priya (Client Reviewer Contact) | No | No | Settings is not shown in navigation at all (FEAT-21.SPEC-010) |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- unsaved field edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Settings" with the shared Settings navigation shell (as described in FEAT-21.SPEC-001), "Business Details & Payment Terms" selected. When entered from FEAT-21.SPEC-009's incomplete-details prompt, a banner appears at the top: "Complete your business details before sending your first invoice."

**Body:** A single-column form with the following fields in order:
- Business name (text input, required before first invoice)
- Business address (multi-line text input, required before first invoice)
- Tax ID (text input, optional)
- Default payment terms (selection input: "Due on receipt" or "Net {N} days", required before first invoice)

Below the form, a completeness indicator line reading either "Business details are complete." or "Business details are incomplete -- required before your first invoice can be sent." (per FEAT-21.SPEC-009), followed by a "Save" action button.

**Footer:** None -- Save is directly below the form.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width, stacked in the order listed; the completeness indicator and Save button remain below the form.
- **Medium size class and above:** Form remains single-column, capped at the platform-wide form width alongside the navigation shell; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Business name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Business name input | Blur (empty) | Triggers the required-before-invoicing check via FEAT-21.SPEC-007 (advisory, non-blocking) | Advisory notice on field; Save stays enabled | "Business name is required before invoicing" below field, per FEAT-21.SPEC-007 |
| Business address input | Type / Blur | Captures text input; validates via FEAT-21.SPEC-007 (empty is advisory, over-length is blocking) | Field shows entered text, advisory notice (empty), or error state (over 500 characters) | Standard input state, "Business address is required before invoicing" (empty), or "Business address must be 500 characters or fewer" |
| Tax ID input | Type | Captures text input | Field shows entered text | Standard input focus state -- no required-field error, since tax ID is optional |
| Default payment terms selector | Select | Captures the chosen option; validates via FEAT-21.SPEC-007 | Selector shows chosen value | Standard selection feedback |
| Save button | Tap | Validate all fields via FEAT-21.SPEC-007; only blocking rules (length limits) stop the save. Empty required-before-invoicing fields do not block: whatever is filled is saved (partial saves allowed), then completeness is re-checked via FEAT-21.SPEC-009 | Button shows loading state during save | Success: toast "Business details updated" and the completeness indicator updates (complete or incomplete). Blocking validation failure: error state on the offending field, save does not proceed. Server failure: inline error banner, retry option, entered fields preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Navigation shell -> Business name -> Business address -> Tax ID -> Default payment terms selector -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Completeness announcements:** A change in the completeness indicator's text (from incomplete to complete, or the reverse) is announced on save.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form pre-filled with current values (empty for any field never set), Save button enabled | Screen opens | User begins editing |
| Editing | Fields show in-progress input, Save button enabled | User types or selects in any field | User taps Save or navigates away |
| Saving | Save button shows a loading spinner, fields disabled | User taps Save with valid input | Save completes or fails |
| Validation Error | Failed fields highlighted with error messages below them; Save does not proceed | A field exceeds its length limit (business name 200, address 500, tax ID 50) on blur or submit | User corrects the field and re-triggers validation |
| Incomplete notice | Empty required-before-invoicing fields show their advisory message below the field; Save remains enabled | A required-before-invoicing field (business name, address, default payment terms) is empty on blur or after save | User fills the field |
| Error | Error banner at top of form: "Could not save your business details. Check your connection and try again." with a Retry button | Save operation fails server-side | User taps Retry; entered fields are preserved and resubmitted |
| Read-only (Dana) | Form shows current values with no Save button | Dana opens this screen inside a logged FEAT-31 support session | Dana closes the support session or navigates away |
| Offline/Degraded | N/A -- settings changes require connectivity to persist (product-features.md, States field); the form remains visible with current values but Save is disabled and a banner reads "You're offline. Reconnect to save changes." | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; no queued submission occurs |

## Validation Rules

Validation governed by FEAT-21.SPEC-007 (Account Field Validation Rules). See that spec for the required-before-invoicing rules on business name, business address, and default payment terms (advisory and non-blocking: they never stop a save), for the blocking length limits, and for tax ID's optional length rule. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Navigation shell: "Profile" | FEAT-21.SPEC-001 (Account Profile) | -- |
| Navigation shell: "Notification Preferences" | FEAT-21.SPEC-002 (Notification Preferences) | -- |
| Navigation shell: "Login & Security" | FEAT-21.SPEC-003 (Login & Security) | -- |
| Navigation shell: "Payment account" | FEAT-32.SPEC-001 (Payment Account connection screen) | FEAT-32 (Payment Account Connection) |
| Navigation shell: "Branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Navigation shell: "Close account" | FEAT-24 (Data Export & Account Deletion entry screen) | FEAT-24 (Data Export & Account Deletion) |
| Successful save | Stays on FEAT-21.SPEC-004 with the confirmation toast and updated completeness indicator | -- |

## Data Model

**Creates:** None.
**Reads:** Freelancer Account -- `business_name`, `business_address`, `tax_id`, `default_payment_terms`.
**Updates:** Freelancer Account -- `business_name`, `business_address`, `tax_id`, `default_payment_terms`.
**Deletes:** None.

## Business Rules

- Blocking field validation (FEAT-21.SPEC-007 length limits) is enforced before any save completes. Required-before-invoicing fields are non-blocking: Nadia can save with any of business name, address, or default payment terms empty (including a form where only tax ID is filled); the save succeeds and completeness stays incomplete until all three are filled (FEAT-21.SPEC-009).
- Completeness (FEAT-21.SPEC-009) is re-evaluated on every successful save and reflected in the indicator line; the gate itself is enforced at invoice-send time by FEAT-09, not by this screen (XBR-16).
- Access and read-only scope (FEAT-21.SPEC-010) governs what Dana sees and cannot act on.
- Two open sessions of Nadia's editing these fields resolve last-write-wins per field, per the dependency map's Contention note for the Freelancer Account entity.

## Edge Cases

- **Nadia navigates away with unsaved edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Save fails server-side** -- Error banner: "Could not save your business details. Check your connection and try again." with a Retry button; entered fields are preserved and retried without being discarded (product-features.md, States field).
- **Business details changed by Nadia in a second open session while this screen is open** -- Save is last-write-wins per the dependency map's Contention note for the Freelancer Account entity.
- **Nadia completes the last missing required field and saves while a first-invoice send from another screen is in progress against the previously incomplete state** -- FEAT-21.SPEC-009's completeness check runs at the moment FEAT-09 attempts the send, not at the moment this screen loaded, so a send that starts after this save's completeness update proceeds; a send already in flight when this save commits uses the state it read at its own check point (FEAT-21.SPEC-009's own Edge Cases govern the exact race).
- **Nadia clears a previously filled required field, leaving business details incomplete again** -- The save is not blocked; it succeeds with the toast "Business details updated", the completeness indicator switches back to "incomplete", and any future first-invoice send is blocked again per FEAT-21.SPEC-009.
- **Nadia fills only some fields (e.g., only tax ID) and saves** -- The partial values are saved, the toast "Business details updated" appears, the empty required fields keep their advisory messages, and the indicator reads "Business details are incomplete -- required before your first invoice can be sent."

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | Field-level validation for all fields on this screen |
| FEAT-21.SPEC-009 (Business Details Completeness Gate) | Triggers (outbound) | Re-checks completeness on every save; navigates here when a blocked invoice send prompts Nadia to finish |
| FEAT-21.SPEC-010 (Settings Access & Read-Only Scope Rules) | References (inbound) | Governs Dana's read-only rendering and Owen/Priya's total exclusion |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Shared Settings navigation shell entry point |
| FEAT-21.SPEC-002 (Notification Preferences) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound) | Shared Settings navigation shell |
| FEAT-09 (Invoice Generation & Sending) | References (outbound) | Business details and default payment terms print on every invoice; sending is blocked until they exist (XBR-16) |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support session renders this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| business_details_updated | fields_changed (list), completeness_state: complete/incomplete | Nadia's business details save completes successfully | N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("business_details_updated") so the save is observable |
| settings_save_failed | section: business_details | Save fails server-side | N/A -- no connected success-metrics.md metric; retained to make retry-preserving save failures observable rather than silent |

## Acceptance Criteria

**FEAT-21.SPEC-004-AC-01:** Given Nadia is on the Business Details & Payment Terms screen with business name empty, when she taps Save, then the business name field shows the advisory "Business name is required before invoicing", the save still proceeds for the other fields with the toast "Business details updated", and the completeness indicator reads "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-004-AC-02:** Given Nadia fills in business name, business address, and default payment terms, when she taps Save, then the details save, a "Business details updated" toast appears, and the completeness indicator reads "Business details are complete."

**FEAT-21.SPEC-004-AC-03:** Given Nadia leaves the tax ID field empty, when she saves with the other required fields complete, then the save succeeds and the completeness indicator reads "Business details are complete." (tax ID is optional).

**FEAT-21.SPEC-004-AC-04:** Given Nadia arrives from FEAT-21.SPEC-009's incomplete-details prompt, when the screen loads, then the banner "Complete your business details before sending your first invoice." appears above the form.

**FEAT-21.SPEC-004-AC-05:** Given Dana (Support Operator) opens this screen inside a logged FEAT-31 support session, when she views the fields, then she sees the current values with no Save button.

**FEAT-21.SPEC-004-AC-06:** Given Dana (Support Operator) is viewing business details read-only, when she attempts to submit a change through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-004-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation, then none is shown.

**FEAT-21.SPEC-004-AC-08:** Given Nadia's business details save fails server-side, when the failure occurs, then the error banner "Could not save your business details. Check your connection and try again." appears with a Retry button, and her entered fields remain populated.

**FEAT-21.SPEC-004-AC-09:** Given Nadia has an unsaved edit, when she navigates away, then a confirmation dialog appears: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-21.SPEC-004-AC-10:** Given Nadia clears a previously filled business address, leaving it empty, when she saves, then the save succeeds with the toast "Business details updated" and the completeness indicator switches to "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-004-AC-11:** Given Nadia loses connectivity on this screen, then the Save button is disabled and a banner reads "You're offline. Reconnect to save changes."

**FEAT-21.SPEC-004-AC-12:** Given Nadia's expired session is detected while she has unsaved field edits, when she signs in again, then her unsaved edits are restored on this screen.

**FEAT-21.SPEC-004-AC-13:** Given Nadia fills only the tax ID and leaves business name, business address, and default payment terms empty, when she taps Save, then the tax ID is saved, the toast "Business details updated" appears, each empty required field shows its advisory message, and the indicator reads "Business details are incomplete -- required before your first invoice can be sent."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 8 (loaded, editing, saving, validation error, incomplete notice, error, read-only (Dana), offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Automation Spec: Sign-In Email & Login Method Change

## Overview

**Name:** Sign-In Email & Login Method Change
**ID:** FEAT-21.SPEC-005
**Type:** Automation
**Purpose:** Processes a pending sign-in email or login method change through re-verification before it takes effect, and expires or lets Nadia resend an unconfirmed change.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Starting a pending change when Nadia submits a new sign-in email on FEAT-21.SPEC-003
- Sending a re-verification link/code through transactional email delivery
- Confirming the change when Nadia completes re-verification, and committing the new sign-in email
- Expiring an unconfirmed change and reverting to the prior value
- Resending the re-verification link/code on request
- Triggering the account-critical change confirmation email (FEAT-21.SPEC-011) once the change commits

**Non-Goals:**
- Collecting the new sign-in email itself -- that is FEAT-21.SPEC-003's form; this automation begins once a syntactically valid new email is submitted
- Any alternate login method beyond email-based sign-in -- excluded per the Brief's Non-Goals: "login recovery here covers a single email-based sign-in method only"
- Signing out other devices -- a distinct capability owned by FEAT-21.SPEC-006; a sign-in email change does not itself invalidate other sessions
- Validating the new email's format -- owned by FEAT-21.SPEC-007 (Account Field Validation Rules) and enforced by FEAT-21.SPEC-003 before this automation ever starts

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New sign-in email submitted | FEAT-21.SPEC-003 (Login & Security), via "Change sign-in email" or, while a change is pending, the banner's "Use a different email" | Fires when Nadia submits a new sign-in email that passes FEAT-21.SPEC-007 validation and differs from her current sign-in email | New sign-in email, Nadia's current sign-in email, Freelancer Account reference |
| Resend requested | FEAT-21.SPEC-003 (Login & Security) | Fires when Nadia taps "Resend link" while a change is pending | The pending change's target email, Freelancer Account reference |
| Re-verification completed | FEAT-21.SPEC-003 (Login & Security), via the link/code the recipient followed | Fires when the re-verification link/code is followed or entered while the pending change has not expired | The pending change's target email and token/code |
| Cancel requested | FEAT-21.SPEC-003 (Login & Security) | Fires when Nadia taps "Cancel" while a change is pending | The pending change's target email, Freelancer Account reference |
| Pending change expiry | System (scheduled check against the pending change's expiry timestamp) | Fires when the pending change's platform parameter: `email-change-reverification-window` elapses with no completed re-verification | The pending change's target email, Freelancer Account reference |

## Processing Logic

1. On new-email submission: create a pending change record on the Freelancer Account holding the target email, a re-verification token, and an expiry timestamp set to platform parameter: `email-change-reverification-window` from now. If a pending change already existed, replace it and invalidate its prior token.
2. Send the re-verification link/code to the target email through the transactional email delivery capability (FEAT-14.SPEC-001).
3. On resend request: verify a pending change exists and has not expired; if so, issue a fresh token (invalidating the previous one) and resend to the same target email, resetting the expiry to a full platform parameter: `email-change-reverification-window` from the resend moment.
4. On re-verification completion: verify the presented token matches the pending change's current token and has not expired. If valid, commit the target email as the Freelancer Account's sign-in email, clear the pending change, and trigger the account-critical change confirmation email (FEAT-21.SPEC-011).
5. On cancel request: clear the pending change and invalidate its token; the prior sign-in email remains active throughout and requires no reversal since it was never changed.
6. On expiry: clear the pending change and invalidate its token; the prior sign-in email remains active (it was never altered mid-process).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Change started | New email submitted and accepted | Pending change record created on Freelancer Account | FEAT-21.SPEC-003 shows "Verification pending for {new email}" | FEAT-21.SPEC-003 |
| Resend succeeded | Resend requested against a still-pending, non-expired change | Token refreshed, expiry reset | FEAT-21.SPEC-003 shows "A new verification link has been sent." | FEAT-21.SPEC-003 |
| Change confirmed | Valid, non-expired token presented | Freelancer Account sign-in email updated; pending change cleared | FEAT-21.SPEC-003 shows the new sign-in email as active; account-critical change confirmation email sent | FEAT-21.SPEC-003, FEAT-21.SPEC-011 |
| Change cancelled | Nadia taps Cancel on a pending change | Pending change cleared; no account field altered | FEAT-21.SPEC-003 shows "Sign-in email change cancelled." | FEAT-21.SPEC-003 |
| Change expired | platform parameter: `email-change-reverification-window` elapses with no confirmation | Pending change cleared; no account field altered | FEAT-21.SPEC-003's pending banner clears automatically; Nadia can restart the change | FEAT-21.SPEC-003 |
| Automation failure (send or commit) | The re-verification email fails to send, or the commit step fails after a valid token is presented | No account field altered; pending change is retried per FEAT-14.SPEC-001's retry rules for the send case, or remains pending for the commit case | FEAT-21.SPEC-003 shows an inline error: "Could not complete this action. Check your connection and try again." with Retry | FEAT-21.SPEC-003 |

## Data Model

**Reads:** Freelancer Account -- current sign-in email, any existing pending change record.
**Creates:** A pending change record on the Freelancer Account -- target email, re-verification token, expiry timestamp (created on submission, replaced on resend/re-submission, cleared on confirm/cancel/expiry).
**Updates:** Freelancer Account -- sign-in email (set only on confirmed re-verification).
**Deletes:** The pending change record, on confirm, cancel, or expiry.

## Business Rules

- The prior sign-in email stays fully active and usable for sign-in until the moment a new email is confirmed -- there is no window in which Nadia is locked out (dependency map, Freelancer Account State Transition: "Active -> Pending re-verification -> Active (confirmed) or reverts to the prior value on expiry/abandonment").
- Only one pending change can exist at a time; a new submission or resend always replaces and invalidates the prior token (XBR-30's authority feature, FEAT-14, delivers only the current token's email).
- A confirmed change always triggers the account-critical change confirmation email (FEAT-21.SPEC-011) -- this is never skipped or batched with other notifications.
- This automation never signs out other devices on its own; that remains a separate, explicit action (FEAT-21.SPEC-006).

## Edge Cases

- **Nadia submits her current sign-in email as the "new" one** -- Rejected before this automation starts, per FEAT-21.SPEC-007 and FEAT-21.SPEC-003's inline check ("This is already your sign-in email."); no pending change is created.
- **The re-verification link/code is followed twice (e.g., email client pre-fetches it)** -- The first valid use commits the change and clears the pending record; the second use finds no pending change matching that token and shows "This verification link has already been used or has expired." with no further data change.
- **Nadia cancels a change at the exact moment the re-verification link is followed** -- Whichever request the system processes first wins: if the cancel commits first, the pending change is gone and the late verification attempt shows "This verification link has already been used or has expired."; if the verification commits first, the change is confirmed and the later cancel finds nothing pending and is a no-op.
- **The pending change expires at the exact moment re-verification is submitted** -- The expiry check is authoritative: a token presented at or after the expiry timestamp is treated as expired, and the presenter sees "This verification link has expired. Start the change again."
- **Concurrent trigger firing (a resend request and an independent new-email submission arrive at effectively the same time)** -- Whichever request is processed first sets the current pending state (token and target email); the second request either resends against that same state (if it targets the same email) or replaces it (if it targets a different email) -- there is no scenario where two pending changes coexist.
- **Trigger fires while a previous run is in flight (Nadia taps Resend twice rapidly)** -- FEAT-21.SPEC-003's "Resend link" button is disabled during the resend request, per that spec's Interactions; a second resend cannot start until the first completes.
- **The transactional email delivery capability is down when the re-verification email is due to send** -- The send is queued and retried per FEAT-14.SPEC-001's standing retry behavior; the pending change's expiry clock (platform parameter: `email-change-reverification-window`) still runs from the original submission time, so a prolonged outage can expire the change before delivery succeeds, in which case the expiry outcome applies and Nadia is shown the option to restart.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-003 (Login & Security) | Triggered by (inbound) | Every trigger in this automation originates from actions on that screen |
| FEAT-21.SPEC-003 (Login & Security) | Affects (outbound) | Returns pending/confirmed/expired/cancelled state to that screen |
| FEAT-21.SPEC-007 (Account Field Validation Rules) | References (inbound) | New sign-in email format and "not the same as current" checks run before this automation starts |
| FEAT-21.SPEC-011 (Account-Critical Change Confirmation Email) | Triggers (outbound) | Fires once the change commits |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggers (outbound) | Delivers the re-verification link/code and its resends |

## Analytics and Success Signals

- **account_email_change_started** (has_prior_pending: yes/no) -- N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("account_email_changed" family) so the start of the flow is observable
- **account_email_change_confirmed** (time_to_confirm_bucket) -- N/A -- no connected success-metrics.md metric; retained so the completion of a security-sensitive change is observable
- **account_email_change_expired** () -- N/A -- no connected success-metrics.md metric; retained so abandoned changes are observable rather than silently disappearing
- **account_email_change_cancelled** () -- N/A -- no connected success-metrics.md metric; retained for the same reason

## Acceptance Criteria

**FEAT-21.SPEC-005-AC-01:** Given Nadia submits a new, valid sign-in email on FEAT-21.SPEC-003, when this automation processes the submission, then a pending change is created, a re-verification link is sent to the new email, and FEAT-21.SPEC-003 shows "Verification pending for {new email}".

**FEAT-21.SPEC-005-AC-02:** Given Nadia has a pending change, when she taps "Resend link", then a fresh token is issued, a new link is sent, and the pending change's expiry resets to a full platform parameter: `email-change-reverification-window`.

**FEAT-21.SPEC-005-AC-03:** Given Nadia follows a valid, non-expired re-verification link, when this automation processes it, then her sign-in email is updated to the target email, the pending change clears, and FEAT-21.SPEC-011 sends the account-critical change confirmation email.

**FEAT-21.SPEC-005-AC-04:** Given Nadia has a pending change, when she taps "Cancel", then the pending change clears and her prior sign-in email remains active with no confirmation email sent.

**FEAT-21.SPEC-005-AC-05:** Given Nadia's pending change reaches platform parameter: `email-change-reverification-window` with no completed re-verification, when this automation's expiry check runs, then the pending change clears and her prior sign-in email remains active.

**FEAT-21.SPEC-005-AC-06:** Given Nadia has an already-confirmed change, when the same re-verification link is followed a second time, then she sees "This verification link has already been used or has expired." and no further data changes.

**FEAT-21.SPEC-005-AC-07:** Given Nadia taps "Use a different email" on FEAT-21.SPEC-003's pending banner and submits a different valid email while one change is already pending, when this automation processes the new submission, then the prior pending change and its token are invalidated and replaced by the new target email.

**FEAT-21.SPEC-005-AC-08:** Given a re-verification token is presented at or after its expiry timestamp, when this automation checks it, then the presenter sees "This verification link has expired. Start the change again." and no account field changes.

**FEAT-21.SPEC-005-AC-09:** Given the transactional email delivery capability is unavailable when a re-verification email is due to send, when the delivery capability recovers before the pending change's expiry, then the queued email is delivered and the change remains eligible for confirmation.

**FEAT-21.SPEC-005-AC-10:** Given the transactional email delivery capability remains unavailable past the pending change's expiry, when the expiry check runs, then the change expires per FEAT-21.SPEC-005-AC-05, regardless of the undelivered email.

**FEAT-21.SPEC-005-AC-11:** Given Nadia taps "Resend link" while a prior resend request for the same change is still processing, when the second tap occurs, then it is ignored because FEAT-21.SPEC-003 disables the control during the in-flight request.

**FEAT-21.SPEC-005-AC-12:** Given this automation's commit step fails after a valid token is presented, when the failure occurs, then no sign-in email change is applied, the pending change remains active, and FEAT-21.SPEC-003 shows "Could not complete this action. Check your connection and try again." with Retry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (submit, resend, confirm, cancel, expiry) | 5 |
| Outcome Paths | 6 (started, resend succeeded, confirmed, cancelled, expired, failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Automation Spec: Sign-Out Other Sessions

## Overview

**Name:** Sign-Out Other Sessions
**ID:** FEAT-21.SPEC-006
**Type:** Automation
**Purpose:** Invalidates every one of Nadia's signed-in sessions except the current one and records the event, when she taps "Sign out other devices".
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- Invalidating every active session on Nadia's account except the session that issued the request
- Refreshing the signed-in devices list shown on FEAT-21.SPEC-003 once complete
- Recording the sign-out event

**Non-Goals:**
- Signing out any single, individually chosen device -- the Brief's Key Capabilities and FEAT-21.SPEC-003's layout define only an all-other-sessions action, not per-device selection
- Changing the sign-in email or login method -- a distinct capability owned by FEAT-21.SPEC-005; signing out other sessions never alters Nadia's sign-in email
- Signing out Dana's support session or any client contact session -- excluded because neither Dana's Support Access Session (FEAT-31) nor a Client Contact's portal session is a signed-in device on the Freelancer Account; this automation's scope is limited entirely to Nadia's own sessions

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| "Sign out other devices" confirmed | FEAT-21.SPEC-003 (Login & Security) | Fires when Nadia confirms the "Sign out other devices" action, and at least one other active session exists | Freelancer Account reference, current session identifier, list of all other active session identifiers at request time |

## Processing Logic

1. Receive the request from the triggering screen, including the identifier of the current (requesting) session.
2. Read the full list of active sessions on the Freelancer Account.
3. Exclude the current session from the list -- it is never a candidate for invalidation.
4. Invalidate every remaining session on the list immediately.
5. Record the event.
6. Signal the triggering screen that invalidation is complete, with the count of sessions signed out.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Sessions signed out | One or more other sessions existed and were invalidated | All non-current sessions on the Freelancer Account marked invalid; event recorded | FEAT-21.SPEC-003 shows toast "All other sessions signed out." and the device list refreshes to show only "This device" | FEAT-21.SPEC-003 |
| No-action (nothing to sign out) | No other active sessions existed at request time | None | Not user-visible as a distinct outcome -- FEAT-21.SPEC-003 never shows the "Sign out other devices" button when no other session exists, so this path is only reachable if the last other session ended between screen load and the request; in that case the device list simply refreshes with the toast "All other sessions signed out." (zero sessions affected) | FEAT-21.SPEC-003 |
| Automation failure | Invalidation cannot complete (e.g., the session store is unreachable) | No sessions invalidated | FEAT-21.SPEC-003 shows inline error "Could not sign out other sessions. Try again." and the device list remains unchanged | FEAT-21.SPEC-003 |

## Data Model

**Reads:** Freelancer Account -- signed-in devices list, current session identifier.
**Creates:** None -- the sign-out event is recorded, not created as a new tracked entity beyond that record.
**Updates:** Freelancer Account -- signed-in devices list (each non-current session marked invalid/removed).
**Deletes:** None -- session invalidation is a state change, not a deletion of the account record.

## Business Rules

- The current session is never included in the invalidation, by construction of the trigger's available data (the current session identifier is excluded before invalidation runs).
- This action always signs out all other sessions at once -- there is no partial or per-device variant (per the Brief's Key Capabilities: "can sign out other devices").
- A signed-out session cannot be resumed; the device must sign in again from the start, going through the product's standard sign-in path.

## Edge Cases

- **Nadia taps "Sign out other devices" with zero other sessions active (the last one ended between screen load and her tap)** -- The action still completes with zero sessions affected; the toast "All other sessions signed out." still appears since, from Nadia's perspective, the intended end state (only her current session active) is achieved.
- **A device being signed out is mid-action (e.g., another of Nadia's sessions is in the middle of saving a form) at the moment of invalidation** -- That other session's next request is rejected as unauthenticated; any unsaved data in that session is lost, consistent with an ordinary session expiry on that device.
- **Concurrent trigger firing (Nadia taps "Sign out other devices" from two of her own sessions at effectively the same time)** -- Whichever request's invalidation step commits first signs out every other session, including the second triggering session itself if it was not the one processed first; the second request then finds its own session already invalidated and the triggering screen redirects that device to sign in again.
- **Trigger fires while a previous run is in flight** -- A second "Sign out other devices" tap from the same session while the first request is still processing is ignored, since FEAT-21.SPEC-003's button shows a loading state during processing and does not accept a second tap.
- **A device reconnects immediately after being signed out** -- It is treated as a fresh, unauthenticated session and must sign in again; it is not automatically re-admitted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-003 (Login & Security) | Triggered by (inbound) | "Sign out other devices" confirmation fires this automation |
| FEAT-21.SPEC-003 (Login & Security) | Affects (outbound) | Refreshed device list and completion toast/error shown there |

## Analytics and Success Signals

- **other_sessions_signed_out** (session_count) -- N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Signals field ("other_sessions_signed_out") so this security action is observable
- **other_sessions_sign_out_failed** () -- N/A -- no connected success-metrics.md metric; retained so a failed security action is observable rather than silent

## Acceptance Criteria

**FEAT-21.SPEC-006-AC-01:** Given Nadia has two other active sessions in addition to her current one, when she confirms "Sign out other devices", then both other sessions are invalidated, her current session remains active, and she sees "All other sessions signed out."

**FEAT-21.SPEC-006-AC-02:** Given Nadia's other session was already invalidated by the time her confirmed request is processed (it ended on its own between load and request), when this automation runs, then it completes with zero sessions affected and she still sees "All other sessions signed out."

**FEAT-21.SPEC-006-AC-03:** Given a signed-out device attempts its next action after invalidation, then that action is rejected as unauthenticated and the device is returned to the sign-in screen.

**FEAT-21.SPEC-006-AC-04:** Given Nadia taps "Sign out other devices" from two of her own sessions at effectively the same time, when the first request's invalidation commits, then the second triggering session is itself signed out and is redirected to sign in again.

**FEAT-21.SPEC-006-AC-05:** Given Nadia taps "Sign out other devices" a second time while the first request is still processing, then the second tap is ignored because the button is in a loading state.

**FEAT-21.SPEC-006-AC-06:** Given the session invalidation step fails, when the failure occurs, then no sessions are invalidated and FEAT-21.SPEC-003 shows "Could not sign out other sessions. Try again."

**FEAT-21.SPEC-006-AC-07:** Given a device Nadia just signed out attempts to reconnect immediately afterward, then it is treated as a fresh, unauthenticated session and must sign in again.

**FEAT-21.SPEC-006-AC-08:** Given this automation completes successfully, then a sign-out event is recorded for the account.

**FEAT-21.SPEC-006-AC-09:** Given Nadia has no other active sessions and the "Sign out other devices" button is therefore not shown on FEAT-21.SPEC-003, then this automation is never triggered from that state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (signed out, no-action, failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Account Field Validation Rules

## Overview

**Name:** Account Field Validation Rules
**ID:** FEAT-21.SPEC-007
**Type:** Logic/Rule
**Purpose:** Enforces required fields, format, and value limits across profile, business details, and payment terms fields on the Freelancer Account.
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account

## Scope and Non-Goals

**In Scope:**
- Field-level validation for name, business name, business address, tax ID, and default payment terms
- The "not the same as current" check applied to a new sign-in email submission
- Authorization for every action this feature defines on the Freelancer Account

**Non-Goals:**
- The sign-in email's own format and re-verification process -- format is checked here as a field rule applied at submission time on FEAT-21.SPEC-003, but the re-verification workflow itself (send, confirm, expire, resend) is owned entirely by FEAT-21.SPEC-005
- Notification preference toggling rules -- owned by FEAT-21.SPEC-008 (Notification Preference Rules), a distinct rule set for a distinct part of the Freelancer Account
- Business details completeness-for-invoicing determination -- owned by FEAT-21.SPEC-009 (Business Details Completeness Gate), which consumes this spec's field rules but owns the aggregate completeness decision
- Validation of signed-in devices data -- the devices list is system-derived from active sessions, not user-entered data, so no field validation applies to it

## Governed Entity

**Entity:** Freelancer Account
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Freelancer's own name, shown on her profile |
| sign-in email | text | Email used to sign in; changes require re-verification (FEAT-21.SPEC-005) |
| business_name | text | Name printed on invoices as the freelancer's business |
| business_address | text | Address printed on invoices |
| tax_id | text | Freelancer's tax identifier, printed on invoices when set |
| default_payment_terms | enum | Account-wide default due-date term applied to new invoices |
| time_zone | derived | Owned entirely by Currency & Tax Handling (FEAT-15); this feature exposes no editor for it (Brief Non-Goals) -- listed here only because it is a field on the governed entity, with no validation rule of this feature's own |
| notification_preferences | derived | Governed by FEAT-21.SPEC-008, not this spec |
| signed-in devices | derived | System-maintained list of active sessions; no user-entered validation applies |
| help-tip dismissals | derived | Captured for FEAT-30 (Later); this feature exposes no management surface for it (Brief Non-Goals) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Account Profile | On field blur and form submit for `name`; authorization on screen entry and on save |
| FEAT-21.SPEC-003 | Login & Security | On submit for the new sign-in email's format and "not the same as current" check; authorization on screen entry |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | On field blur and form submit for `business_name`, `business_address`, `tax_id`, `default_payment_terms`; authorization on screen entry and on save |
| FEAT-21.SPEC-005 | Sign-In Email & Login Method Change | Re-checks the "not the same as current" condition is still true immediately before starting the pending change |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, max 100 characters | Always | On blur / on submit | "Name is required" / "Name must be 100 characters or fewer" | Yes |
| sign-in email (new value on FEAT-21.SPEC-003) | Valid email format | Always | On submit | "Please enter a valid email address" | Yes |
| sign-in email (new value on FEAT-21.SPEC-003) | Must differ from the current sign-in email | Always | On submit | "This is already your sign-in email." | Yes |
| business_name | Required before the first invoice is sent (advisory: an empty value never blocks a save; it keeps completeness incomplete per FEAT-21.SPEC-009), max 200 characters | Always (required-before-invoicing per XBR-16); length limit always | On blur / on submit | "Business name is required before invoicing" (advisory) / "Business name must be 200 characters or fewer" | No for the required rule (advisory only); Yes for the length limit |
| business_address | Required before the first invoice is sent (advisory, non-blocking as above), max 500 characters | Always (required-before-invoicing per XBR-16); length limit always | On blur / on submit | "Business address is required before invoicing" (advisory) / "Business address must be 500 characters or fewer" | No for the required rule (advisory only); Yes for the length limit |
| tax_id | No length beyond 50 characters; no required-field error | Always | On blur | "Tax ID must be 50 characters or fewer" | Yes (length only; the field itself is optional) |
| default_payment_terms | Required before the first invoice is sent; must be one of the product's defined terms options ("Due on receipt" or "Net {N} days", where {N} is one of the product's fixed offered values) | Always (required-before-invoicing per XBR-16) | On selection / on submit | "Choose a default payment term before invoicing" (advisory) | No -- advisory only; an unselected value never blocks a save, it keeps completeness incomplete (FEAT-21.SPEC-009). A value outside the defined options is not selectable and is rejected (Yes) |
| time_zone | No validation beyond data type -- this feature never writes this field (owned by FEAT-15) | Always | -- | -- | -- |
| notification_preferences | No validation beyond data type in this spec -- see FEAT-21.SPEC-008 | Always | -- | -- | -- |
| signed-in devices | No validation beyond data type -- system-derived, never user-entered | Always | -- | -- | -- |
| help-tip dismissals | No validation beyond data type -- captured for FEAT-30 (Later), no management surface in this feature | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Business details required together before invoicing | business_name, business_address, default_payment_terms | All three must be non-empty/selected before FEAT-21.SPEC-009 marks business details complete (XBR-16); tax_id is not part of this set since it is optional | No blocking error on this screen -- saving with any of the three empty (including only tax ID filled) succeeds; the completeness indicator on FEAT-21.SPEC-004 (owned by FEAT-21.SPEC-009) reads "Business details are incomplete -- required before your first invoice can be sent." while any of the three is missing |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own profile, business details, notification preferences | Nadia (Freelancer) | Always, her own account only | -- |
| Edit name (profile) | Nadia (Freelancer) | Always, her own account only | -- |
| Edit business_name, business_address, tax_id, default_payment_terms | Nadia (Freelancer) | Always, her own account only | -- |
| Start a sign-in email/login method change | Nadia (Freelancer) | Always, her own account only | -- |
| View profile, business details, notification preferences | Dana (Support Operator) | Read-only, inside a logged FEAT-31 support session (FEAT-21.SPEC-010) | -- |
| Edit any field on this entity | Dana (Support Operator) | Never | Save controls are not rendered for Dana on any Settings screen; a direct attempt is refused with "Support sessions are read-only." (FEAT-21.SPEC-010) |
| View or edit sign-in email/login method, signed-in devices | Dana (Support Operator) | Never | Login & Security (FEAT-21.SPEC-003) is never rendered inside a support session, and sign-in credentials are never visible to the operator under any circumstance (feature-dependency-map.md, Freelancer Account, Data Sensitivity) |
| View or edit any field on this entity | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings |
| View or edit any field on this entity | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| name | Set at sign-up (FEAT-20) | On create only | Yes -- Nadia can edit at any time on FEAT-21.SPEC-001 |
| sign-in email | Set at sign-up (FEAT-20) | On create only | Yes -- only through the FEAT-21.SPEC-005 re-verification process, never a direct edit |
| business_name, business_address, tax_id, default_payment_terms | No default -- empty until Nadia fills them, unless captured during FEAT-20's guided setup | On create (if supplied during onboarding) or left empty | Yes -- Nadia can edit at any time on FEAT-21.SPEC-004 |
| time_zone | Set and derived by FEAT-15 | Always | No -- not overridable from this feature |

## Business Rules

- Every rule in this spec applies identically whenever the enforcing screen is open to Nadia -- there is no create-only or edit-only distinction, since the Freelancer Account is created once (FEAT-20) and only ever updated afterward by this feature.
- Business details completeness (XBR-16) is a cross-field, cross-feature outcome computed by FEAT-21.SPEC-009 from this spec's field rules; FEAT-09 checks that outcome at invoice-send time and blocks sending until it is true.
- The sign-in email format and "not the same as current" checks are evaluated at submission time on FEAT-21.SPEC-003; the actual change only takes effect once FEAT-21.SPEC-005's re-verification succeeds (dependency map, Freelancer Account Contention: "the sign-in email change, which requires re-verification before it takes effect").
- Authorization here is consistent with, and never overrides, FEAT-21.SPEC-010's account-wide read-only scope rules for Dana and total exclusion for client contacts.

## Edge Cases

- **Name field at exactly 100 characters** -- Passes validation. 101 characters shows the length error.
- **Business address at exactly 500 characters** -- Passes validation. 501 characters shows the length error.
- **Tax ID left as whitespace only** -- Treated as empty (optional field, no required-field error); trimmed before storage.
- **Nadia submits a new sign-in email that differs from her current one only by letter case (e.g., Nadia@Example.com vs. nadia@example.com)** -- Treated as the same email for the "not the same as current" check (email comparison is case-insensitive), so the submission is rejected with "This is already your sign-in email."
- **Default payment terms cleared after being previously set, then business_name and business_address remain filled** -- The save is not blocked (the required rule is advisory) and succeeds; business details completeness (FEAT-21.SPEC-009) reverts to incomplete, since all three required fields must be non-empty together.
- **Nadia's role or account state changes mid-edit (not possible in this single-role product beyond Nadia herself, but Dana's support session could open concurrently)** -- Dana's concurrently opened read-only session never gains edit controls regardless of timing; Nadia's own edit session is unaffected by a support session opening or closing.

## Acceptance Criteria

**FEAT-21.SPEC-007-AC-01:** Given Nadia clears the name field, when she blurs it, then she sees "Name is required."

**FEAT-21.SPEC-007-AC-02:** Given Nadia enters a name of exactly 100 characters, when she saves, then it is accepted; entering 101 characters shows "Name must be 100 characters or fewer."

**FEAT-21.SPEC-007-AC-03:** Given Nadia submits a malformed new sign-in email, when she submits the change form, then she sees "Please enter a valid email address."

**FEAT-21.SPEC-007-AC-04:** Given Nadia submits her current sign-in email (including a case-only difference) as the "new" one, when she submits, then she sees "This is already your sign-in email." and no pending change is created.

**FEAT-21.SPEC-007-AC-05:** Given Nadia leaves business_name empty and attempts to save Business Details & Payment Terms, when she saves, then she sees the advisory "Business name is required before invoicing." and the save still succeeds (non-blocking), leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-06:** Given Nadia leaves business_address empty and attempts to save, then she sees the advisory "Business address is required before invoicing." and the save still succeeds, leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-07:** Given Nadia does not select a default_payment_terms value and attempts to save, then she sees the advisory "Choose a default payment term before invoicing." and the save still succeeds, leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-08:** Given Nadia leaves tax_id empty and saves with the other required fields complete, then the save succeeds with no required-field error for tax_id.

**FEAT-21.SPEC-007-AC-09:** Given Nadia enters a tax_id of 51 characters, when she blurs the field, then she sees "Tax ID must be 50 characters or fewer."

**FEAT-21.SPEC-007-AC-10:** Given Nadia has filled business_name, business_address, and default_payment_terms, then FEAT-21.SPEC-009 evaluates business details as complete (the cross-field rule's condition is satisfied).

**FEAT-21.SPEC-007-AC-11:** Given Nadia (Freelancer) is on any Settings screen, when she performs an edit action on her own account, then it is always allowed.

**FEAT-21.SPEC-007-AC-12:** Given Dana (Support Operator) is inside a logged support session, when she looks for any save control on any Settings screen, then none is shown, and a direct attempt to submit a change is refused with "Support sessions are read-only."

**FEAT-21.SPEC-007-AC-13:** Given Dana (Support Operator) is inside a logged support session, when she looks for the sign-in email or signed-in devices, then neither is ever shown to her.

**FEAT-21.SPEC-007-AC-14:** Given Owen (Client Primary Contact) attempts to reach any Settings screen, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-007-AC-15:** Given Priya (Client Reviewer Contact) attempts to reach any Settings screen, then she finds none in navigation and no direct access exists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 (name, email format, email-differs, business_name, business_address, tax_id, default_payment_terms, N/A fields noted) | 8 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Notification Preference Rules

## Overview

**Name:** Notification Preference Rules
**ID:** FEAT-21.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces that only optional notifications can be switched off and that transactional record emails always send (XBR-30).
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account (notification_preferences field), read against the Notification entity's type classification

## Scope and Non-Goals

**In Scope:**
- The optional/transactional classification lookup that determines which notification types can be toggled
- The toggle-write rule enforced whenever Nadia changes a preference on FEAT-21.SPEC-002
- Authorization for reading and changing notification preferences

**Non-Goals:**
- Which specific notification types exist and their content -- owned entirely by Notifications (Email) (FEAT-14); this spec only consumes FEAT-14's optional/transactional classification, it does not define it
- Delivery timing, batching, retry, or channel behavior for any notification -- owned by FEAT-14 and by each notification's own spec (e.g., FEAT-21.SPEC-011)
- Field validation for name, business details, and payment terms -- owned by FEAT-21.SPEC-007, a separate rule set for a separate part of the Freelancer Account

## Governed Entity

**Entity:** Freelancer Account (notification_preferences field)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| notification_preferences | derived (a set of per-type on/off values) | Optional notifications only; transactional record emails cannot be disabled (dependency map, Freelancer Account fields) |

**Referenced (read-only):** Notification -- `notification_type` and its optional-vs-transactional classification (dependency map, Referenced Entities: "the preferences screen reads the set of notification types ... to build the toggle list").

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-002 | Notification Preferences | On render (determines which rows show a toggle vs. a locked "Always sent" indicator) and on every toggle, before saving |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | -- (referenced, not enforcing) | Consults the saved preference at send time for optional types only; transactional types are never checked against this preference set at all |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| notification_preferences (per optional type) | Value must be a boolean on/off | Always | On toggle | "Could not save. Try again." (generic save failure -- there is no invalid-value case reachable through the toggle UI) | Yes |
| notification_preferences (per transactional type) | No write path exists -- rejected before any save attempt | Always | On toggle attempt | Not applicable -- FEAT-21.SPEC-002 renders no interactive toggle for a transactional type, so no submission is ever produced for it | Yes (structurally, by omission of the control) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Transactional types are never disableable | notification_preferences, Notification.notification_type classification | A toggle write is accepted only when the targeted notification_type's classification (read from Notification) is Optional; a write targeting a Transactional type is rejected regardless of source | "Transactional emails cannot be turned off." (shown only if a write somehow targets a transactional type outside the normal screen path; the screen itself never exposes this control) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View notification preferences | Nadia (Freelancer) | Always, her own account only | -- |
| Toggle an optional notification type | Nadia (Freelancer) | Always, her own account only | -- |
| Toggle a transactional notification type | Nadia (Freelancer) | Never | No toggle control is rendered for transactional types; the row shows a locked "Always sent" indicator instead |
| View notification preferences | Dana (Support Operator) | Read-only, inside a logged FEAT-31 support session (FEAT-21.SPEC-010) | -- |
| Toggle any notification type | Dana (Support Operator) | Never | Every toggle -- transactional or optional -- renders as a static, disabled indicator; a direct attempt is refused with "Support sessions are read-only." |
| View or toggle any notification type | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings |
| View or toggle any notification type | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| notification_preferences (each optional type) | Defaults to On at account creation (FEAT-20) and for any new optional type FEAT-14 introduces afterward | On create, and when a new optional type is introduced | Yes -- Nadia can turn any optional type off or back on at any time on FEAT-21.SPEC-002 |
| notification_preferences (each transactional type) | Fixed On -- not a stored preference, since it cannot vary | Always | No |

## Business Rules

- XBR-30 is the authority for this spec's central rule: "Notification preferences can switch off only optional emails; transactional emails core to the record ... always send; delivery failures are surfaced to the freelancer as warnings on the affected project" -- the delivery-failure-warning half of XBR-30 is FEAT-14's responsibility, not this spec's.
- A saved preference change is evaluated by FEAT-14 at the moment each individual notification would be sent, not at the moment the preference was changed -- a toggle change never affects a notification already queued or sent before the toggle was saved.
- This spec's classification lookup (Optional vs. Transactional) is authoritative for FEAT-21.SPEC-002's rendering; FEAT-21.SPEC-002 never invents its own classification.
- Authorization here is consistent with, and never overrides, FEAT-21.SPEC-010's account-wide read-only scope rules.

## Edge Cases

- **A notification type FEAT-14 previously classified as Optional is later reclassified as Transactional** -- Any existing off preference for that type is discarded (transactional types have no off state); the row moves from the Optional group to the locked Transactional group on FEAT-21.SPEC-002's next load.
- **A notification type FEAT-14 previously classified as Transactional is later reclassified as Optional** -- The type gains a toggle, defaulting to On, on FEAT-21.SPEC-002's next load, per the Defaults and Derivations row above.
- **Nadia toggles an optional preference in one open session while a second of her sessions has the same screen open** -- The second session reflects the new value on its next refresh; each toggle is its own field-level write, so no merge conflict arises (per FEAT-21.SPEC-002's own Edge Cases).
- **A malformed or forged toggle request targets a transactional type directly (bypassing the screen's UI)** -- Rejected per the Cross-Field Rule above: "Transactional emails cannot be turned off."; the account's transactional types remain On.
- **Nadia turns an optional type off, and a notification of that type was already queued for delivery before the toggle saved** -- The already-queued notification is unaffected (FEAT-14 evaluates the preference at send time for notifications not yet queued); only notifications queued after the toggle saves respect the new Off state.

## Acceptance Criteria

**FEAT-21.SPEC-008-AC-01:** Given Nadia views the Notification Preferences screen, when she looks at a Transactional notification type, then it shows locked-on with no toggle control.

**FEAT-21.SPEC-008-AC-02:** Given Nadia toggles an Optional notification type off, when the toggle saves, then FEAT-14 will not send that type to her going forward until she turns it back on.

**FEAT-21.SPEC-008-AC-03:** Given Nadia toggles an Optional notification type back on, when the toggle saves, then FEAT-14 resumes sending that type to her.

**FEAT-21.SPEC-008-AC-04:** Given a forged or malformed request attempts to toggle a Transactional type off, when this rule evaluates the request, then it is rejected with "Transactional emails cannot be turned off." and the type remains On.

**FEAT-21.SPEC-008-AC-05:** Given FEAT-14 reclassifies a previously Optional type as Transactional, when Nadia's screen next loads, then that type appears in the locked Transactional group with no toggle, regardless of its prior off/on preference.

**FEAT-21.SPEC-008-AC-06:** Given FEAT-14 introduces a new Optional type, when Nadia's screen next loads, then the new type appears with a toggle defaulted to On.

**FEAT-21.SPEC-008-AC-07:** Given Dana (Support Operator) is inside a logged support session, when she views notification preferences, then every toggle -- Transactional or Optional -- is a static, disabled indicator, and a direct change attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-008-AC-08:** Given Owen (Client Primary Contact) attempts to reach the Notification Preferences screen, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-008-AC-09:** Given a notification of an Optional type was already queued before Nadia turns that type off, when the toggle saves, then the already-queued notification is still delivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Business Details Completeness Gate

## Overview

**Name:** Business Details Completeness Gate
**ID:** FEAT-21.SPEC-009
**Type:** Logic/Rule
**Purpose:** Tracks whether business details and payment terms are complete and blocks the first invoice send until they are (XBR-16).
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account (business_name, business_address, default_payment_terms fields)

## Scope and Non-Goals

**In Scope:**
- The completeness determination: which fields must be non-empty for business details to count as complete
- Re-evaluating completeness on every business-details save
- Exposing the completeness state to FEAT-21.SPEC-004 (for the indicator line) and to FEAT-09 (for the send-time gate)

**Non-Goals:**
- Performing the actual block on invoice sending -- that check and its user-facing blocked message are executed by FEAT-09 (Invoice Generation & Sending) at send time; this spec only computes and exposes the true/false completeness state XBR-16 requires
- Field-level validation (required/format/length) for the individual business-detail fields -- owned by FEAT-21.SPEC-007; this spec consumes those fields' current values, it does not validate their format
- Client billing details completeness -- a separate half of XBR-16 owned by FEAT-01 (Client & Project Management); this spec covers only the freelancer's own business details

## Governed Entity

**Entity:** Freelancer Account (business_name, business_address, default_payment_terms fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| business_name | text | Required before the first invoice is sent (dependency map, Freelancer Account fields) |
| business_address | text | Required before the first invoice is sent |
| default_payment_terms | enum | Required before the first invoice is sent |
| tax_id | text | Optional -- not part of the completeness determination (dependency map lists tax_id without a required-before-invoicing note) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-004 | Business Details & Payment Terms | Re-evaluated on every successful save; result drives the screen's completeness indicator line |
| FEAT-09 (Invoice Generation & Sending) | -- | Checked at the moment a first-invoice send is attempted (XBR-16); the send is blocked while this spec's completeness result is false |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| business_name | No validation beyond data type in this spec -- format/required rules owned by FEAT-21.SPEC-007; this spec only reads whether it is non-empty | Always | -- | -- | -- |
| business_address | No validation beyond data type in this spec -- see above | Always | -- | -- | -- |
| default_payment_terms | No validation beyond data type in this spec -- see above | Always | -- | -- | -- |
| tax_id | No validation beyond data type -- not part of the completeness determination | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Business details completeness | business_name, business_address, default_payment_terms | Complete (true) only when all three are non-empty/selected; incomplete (false) if any one is empty | On FEAT-21.SPEC-004: "Business details are incomplete -- required before your first invoice can be sent." On FEAT-09 at send time: the invoice send is blocked with a message directing Nadia to complete business details in Settings (FEAT-09 owns the exact blocked-send message text) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View completeness state | Nadia (Freelancer) | Always, her own account only | -- |
| Change the fields that determine completeness | Nadia (Freelancer) | Always, her own account only, via FEAT-21.SPEC-004 | -- |
| View completeness state | Dana (Support Operator) | Read-only, inside a logged FEAT-31 support session (FEAT-21.SPEC-010) | -- |
| Change the fields that determine completeness | Dana (Support Operator) | Never | No save controls are rendered on FEAT-21.SPEC-004 for Dana; a direct attempt is refused with "Support sessions are read-only." |
| View or change completeness-related fields | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all |
| View or change completeness-related fields | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| completeness (derived) | Computed as business_name non-empty AND business_address non-empty AND default_payment_terms selected | Recomputed on every business-details save and on every FEAT-09 send-time check | No -- it is fully derived, never directly set |

## Business Rules

- XBR-16 is the authority for this spec's central rule: "Every invoice carries a due date from the freelancer's default payment terms ... a unique sequential number per freelancer, her business details, and the client's billing details; sending is blocked until both sets of details exist." This spec owns the freelancer's-own-details half of that check.
- Completeness applies only to the *first* invoice send in the strict sense that once business details are complete, they can later be cleared and re-completed -- the gate re-evaluates the current state at every send attempt (not just the first), so a freelancer who later clears a required field would find sending blocked again, consistent with the field values genuinely being empty at that moment.
- Saving is never blocked by incompleteness: FEAT-21.SPEC-007's required-before-invoicing rules are advisory, so Nadia can save any partial or cleared state on FEAT-21.SPEC-004; the only enforcement of completeness is FEAT-09's send-time block.
- tax_id is deliberately excluded from the completeness determination -- the dependency map records it as optional, with no required-before-invoicing note, unlike business_name, business_address, and default_payment_terms.
- Authorization here is consistent with, and never overrides, FEAT-21.SPEC-010's account-wide read-only scope rules.

## Edge Cases

- **Nadia completes the last missing field and saves while a send attempt from FEAT-09 is already mid-flight against the previously incomplete state** -- FEAT-09's own send-time check is authoritative for that specific attempt; if FEAT-09's check already read "incomplete" before this save committed, that attempt is blocked, and Nadia must retry the send after the save (which will then pass).
- **Nadia clears business_address after it was previously complete, with no first invoice yet sent** -- Completeness reverts to false immediately on that save; any subsequent send attempt is blocked until it is filled again.
- **Nadia clears business_address after the first invoice has already been sent** -- Completeness still reverts to false for the purpose of any *future* first-time gate check (XBR-16 gates the first invoice; a project's first invoice already sent is unaffected retroactively, but any other project of Nadia's whose first invoice has not yet been sent would now be blocked, since completeness is evaluated on the account's current field state, not per project).
- **Nadia saves a partial form (e.g., only tax ID filled, or one required field cleared)** -- The save succeeds (it is never blocked by this rule); completeness is recomputed on that save and is false until all three required fields are filled.
- **All three required fields are filled with only whitespace** -- Treated as empty by FEAT-21.SPEC-007's trim behavior before this spec ever evaluates them, so completeness remains false.
- **Business details were captured during FEAT-20's guided onboarding setup** -- Completeness is evaluated the same way regardless of when the fields were set; onboarding-captured values that satisfy all three conditions yield a complete result with no distinct code path.

## Acceptance Criteria

**FEAT-21.SPEC-009-AC-01:** Given Nadia has business_name, business_address, and default_payment_terms all filled, when this rule evaluates completeness, then the result is complete and FEAT-21.SPEC-004 shows "Business details are complete."

**FEAT-21.SPEC-009-AC-02:** Given Nadia has business_address empty, when this rule evaluates completeness, then the result is incomplete and FEAT-21.SPEC-004 shows "Business details are incomplete -- required before your first invoice can be sent."

**FEAT-21.SPEC-009-AC-03:** Given Nadia's business details are incomplete, when a first-invoice send is attempted through FEAT-09, then FEAT-09 blocks the send and directs her to complete business details in Settings.

**FEAT-21.SPEC-009-AC-04:** Given Nadia's business details are complete, when a first-invoice send is attempted through FEAT-09, then the send proceeds and is not blocked by this rule.

**FEAT-21.SPEC-009-AC-05:** Given Nadia leaves tax_id empty while the three required fields are filled, when this rule evaluates completeness, then the result is complete (tax_id is not part of the determination).

**FEAT-21.SPEC-009-AC-06:** Given Nadia clears default_payment_terms after previously completing business details, when she saves, then the save succeeds (it is not blocked) and completeness reverts to incomplete.

**FEAT-21.SPEC-009-AC-10:** Given Nadia fills only tax_id and leaves business_name, business_address, and default_payment_terms empty, when she saves on FEAT-21.SPEC-004, then the save succeeds and this rule evaluates completeness as incomplete.

**FEAT-21.SPEC-009-AC-07:** Given Dana (Support Operator) is inside a logged support session, when she views the completeness indicator, then she sees its current state read-only with no ability to change the underlying fields.

**FEAT-21.SPEC-009-AC-08:** Given Owen (Client Primary Contact) attempts to reach any Settings screen showing completeness, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-009-AC-09:** Given all three required fields contain only whitespace, when this rule evaluates completeness, then the result is incomplete, since whitespace-only values are treated as empty.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 (all N/A -- owned by FEAT-21.SPEC-007) | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Settings Access & Read-Only Scope Rules

## Overview

**Name:** Settings Access & Read-Only Scope Rules
**ID:** FEAT-21.SPEC-010
**Type:** Logic/Rule
**Purpose:** Enforces that only Nadia can edit her own account, that Dana's support session sees profile/preference/business values read-only and never sign-in credentials, and that client contacts have no settings surface at all.
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account (whole-entity access scope, across every screen in this feature)

## Scope and Non-Goals

**In Scope:**
- The account-wide, cross-screen access and visibility scope for every role touching the Freelancer Account through this feature
- The screen-by-screen read-only rendering rule applied to Dana's support sessions
- The total exclusion rule applied to client contacts

**Non-Goals:**
- Per-field validation rules -- owned by FEAT-21.SPEC-007 (profile/business fields) and FEAT-21.SPEC-008 (notification preferences); this spec governs *who* may reach and act on a screen, not what makes a field value valid
- Opening, closing, or logging a support session -- owned entirely by Operator Support Access (FEAT-31); this spec only consumes the fact that a logged support session is open and applies this feature's own read-only rendering within it
- Any settings capability for a role beyond the four in the Access Matrix -- excluded per scope-boundaries.md SC-01 and SC-02: the product has no internal-staff seat model and no client-side roles beyond Primary and Reviewer

## Governed Entity

**Entity:** Freelancer Account (whole-entity access scope)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| (whole entity) | -- | This spec governs access to the entity as a whole across FEAT-21.SPEC-001 through FEAT-21.SPEC-004, not any single field; per-field rules are FEAT-21.SPEC-007 and FEAT-21.SPEC-008's domain |
| sign-in email, signed-in devices | text / derived | The specific fields this spec singles out as never visible to Dana under any circumstance (feature-dependency-map.md, Freelancer Account, Data Sensitivity: "sign-in credentials never visible to the operator") |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Account Profile | On screen entry (renders read-only for Dana, hidden entirely from client contacts) and on every save attempt |
| FEAT-21.SPEC-002 | Notification Preferences | On screen entry and on every toggle attempt |
| FEAT-21.SPEC-003 | Login & Security | On screen entry -- this screen is never rendered inside a support session and never shown to client contacts |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | On screen entry and on every save attempt |

## Field Validation Rules

No field validation rules -- this spec governs access and visibility scope, not field content. See FEAT-21.SPEC-007 (profile/business fields) and FEAT-21.SPEC-008 (notification preferences) for field validation.

## Cross-Field Rules

No cross-field validation rules -- this spec's cross-cutting concern is expressed entirely through the Authorization Rules table below, which spans all four screens rather than individual fields.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own account (profile, preferences, business details, login & security) | Nadia (Freelancer) | Always, her own account only | -- |
| Edit own account (profile, preferences, business details, login & security) | Nadia (Freelancer) | Always, her own account only | -- |
| View profile, notification preferences, business details | Dana (Support Operator) | Read-only, only inside a logged FEAT-31 support session open on that specific freelancer's account | Outside a logged support session, Dana has no path to any freelancer's Settings at all -- there is no standing access |
| Edit profile, notification preferences, business details | Dana (Support Operator) | Never | No save controls, no destructive actions are rendered on any of these three screens for Dana; a direct attempt to submit a change is refused with "Support sessions are read-only." |
| View sign-in email, signed-in devices, or any Login & Security content | Dana (Support Operator) | Never, under any circumstance | Login & Security (FEAT-21.SPEC-003) is never rendered inside a support session; it does not appear in the Settings navigation shell shown to Dana, and no direct link into it exists from a support session |
| View or edit account closure ("Close account") | Dana (Support Operator) | Never | The "Close account" navigation item is never rendered inside a support session -- account closure is unreachable from a read-only session |
| View or edit any Settings screen or content | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings; a direct URL/deep-link attempt shows the client portal's shared out-of-scope explanation defined in FEAT-05.SPEC-002 (Error state, per XBR-09): the heading "This link isn't valid anymore", the line "Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since.", and a single "Send me a new link" button. No Settings content is rendered and the message never says Settings exists or that access was denied |
| View or edit any Settings screen or content | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings; a direct URL/deep-link attempt shows the identical FEAT-05.SPEC-002 out-of-scope explanation as in Owen's row above (heading "This link isn't valid anymore", the same explanation line, and the "Send me a new link" button) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Dana's Settings visibility scope | Derived from whether a FEAT-31 support session is currently open and logged against the freelancer's account | Evaluated on every screen entry attempt | No -- Dana cannot expand her own scope; it is entirely determined by the support session's state |

## Business Rules

- This spec is the single authoritative home for cross-screen access scope in this feature -- FEAT-21.SPEC-001 through FEAT-21.SPEC-004 each reference this spec rather than defining their own authorization logic (Brief, Shared Validation: "SPEC-001 through SPEC-004 all reference SPEC-010 for what to show, hide, or lock").
- Dana's read-only scope is narrower than "View" on the Access Matrix's own cell for "Subscription & Account Data" in one specific respect: sign-in credentials are excluded entirely, never merely read-only, per the dependency map's explicit Data Sensitivity note for the Freelancer Account entity.
- Every support session Dana opens is itself logged and announced to Nadia by email, per XBR-29 (owned by FEAT-31) -- this spec does not duplicate that logging, it only relies on FEAT-31 to gate when Dana's read-only scope is active at all.
- Client contact exclusion is total and unconditional -- there is no partial, own-only, or view-only tier for Owen or Priya on any Settings screen, unlike their Own-only access elsewhere in the product.

## Edge Cases

- **Dana's support session closes (manually or by inactivity timeout) while she has a Settings screen open** -- Her read-only scope is revoked immediately; the screen redirects her out of the freelancer's account view, consistent with FEAT-31's session lifecycle.
- **Dana attempts to construct a direct link into Login & Security while a support session is open** -- The attempt is refused; FEAT-21.SPEC-003 is never rendered inside a support session regardless of how it is reached, per the "never, under any circumstance" condition above.
- **Owen or Priya follows an old bookmarked Settings URL from before a role change or portal redesign** -- The same "None" denial applies, shown as the FEAT-05.SPEC-002 out-of-scope explanation ("This link isn't valid anymore" with the "Send me a new link" button); no Settings content is ever exposed to a client contact regardless of the path taken to reach it.
- **Two support sessions are somehow opened for the same freelancer account at once (an edge FEAT-31 itself may already prevent)** -- This spec's read-only scope applies identically and independently to each; neither session gains any edit capability, since edit is never granted to Dana under any session state.
- **Nadia opens her own Settings while a Dana support session is also open on her account** -- Nadia's full edit access is entirely unaffected by a concurrent read-only support session; the two views are independent, and Dana's view simply reflects Nadia's most recently saved values on its own refresh.

## Acceptance Criteria

**FEAT-21.SPEC-010-AC-01:** Given Nadia is signed in, when she opens any Settings screen, then she has full view and edit access to her own account.

**FEAT-21.SPEC-010-AC-02:** Given Dana (Support Operator) has no support session open on a freelancer's account, when she attempts to reach that freelancer's Settings, then no path exists -- she has no standing access.

**FEAT-21.SPEC-010-AC-03:** Given Dana (Support Operator) is inside a logged FEAT-31 support session on a freelancer's account, when she views Account Profile, Notification Preferences, or Business Details & Payment Terms, then she sees the current values with no save controls and no destructive actions.

**FEAT-21.SPEC-010-AC-04:** Given Dana (Support Operator) is inside a logged support session, when she attempts to submit a change on any of those three screens through any means, then the attempt is refused with "Support sessions are read-only."

**FEAT-21.SPEC-010-AC-05:** Given Dana (Support Operator) is inside a logged support session, when she looks at the Settings navigation shell, then "Login & Security" does not appear and no direct link reaches it.

**FEAT-21.SPEC-010-AC-06:** Given Dana (Support Operator) is inside a logged support session, when she looks at the Settings navigation shell, then "Close account" does not appear.

**FEAT-21.SPEC-010-AC-07:** Given Owen (Client Primary Contact) is signed in, when he looks for a Settings entry in navigation or follows a direct link to one, then no Settings entry is shown in navigation, and the direct link shows the heading "This link isn't valid anymore" with the line "Sign-in links are single-use and time-limited, and yours has expired, already been used, or was requested again since." and a "Send me a new link" button (FEAT-05.SPEC-002 Error state), with no Settings content exposed.

**FEAT-21.SPEC-010-AC-08:** Given Priya (Client Reviewer Contact) is signed in, when she looks for a Settings entry in navigation or follows a direct link to one, then no Settings entry is shown in navigation, and the direct link shows the identical "This link isn't valid anymore" explanation and "Send me a new link" button, with no Settings content exposed.

**FEAT-21.SPEC-010-AC-09:** Given Dana's support session closes while she has a Settings screen open, when the session ends, then she is redirected out of the freelancer's account view.

**FEAT-21.SPEC-010-AC-10:** Given Nadia has her own Settings open while Dana has a concurrent read-only support session open on the same account, when Nadia saves a change, then Dana's view reflects the new value only on its own next refresh, with no interference to Nadia's save.

**FEAT-21.SPEC-010-AC-11:** Given Dana attempts to construct a direct link into Login & Security while her support session is open, when the link is followed, then the attempt is refused and the screen is not rendered.

**FEAT-21.SPEC-010-AC-12:** Given a client contact follows an old bookmarked Settings URL, when the link is followed, then the same denial applies regardless of the path taken: the heading "This link isn't valid anymore" with the FEAT-05.SPEC-002 explanation line and "Send me a new link" button is shown, and no Settings content is exposed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-21.SPEC-007 / FEAT-21.SPEC-008) | 0 |
| Cross-Field Rules | 0 (N/A -- expressed through Authorization Rules) | 0 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Account-Critical Change Confirmation Email

## Overview

**Name:** Account-Critical Change Confirmation Email
**ID:** FEAT-21.SPEC-011
**Type:** Notification
**Purpose:** Sends Nadia a confirmation email, to both her prior and her new sign-in email address, when her sign-in email address or login method actually changes, so a change she did not make is immediately visible to her at the address she held before the change.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent once a sign-in email/login method change is confirmed by FEAT-21.SPEC-005
- Its single channel (email), content, and delivery behavior

**Non-Goals:**
- The re-verification link/code email sent while the change is still pending -- that is a distinct communication owned entirely by FEAT-21.SPEC-005's own send step (via FEAT-14.SPEC-001), not this spec; this spec fires only after the change is confirmed
- Any notification for profile, notification-preference, or business-details changes -- product-features.md's Communications field names only "account-critical changes (email address change, login method change)" for this confirmation; other Settings saves surface only an in-screen toast (FEAT-21.SPEC-001, FEAT-21.SPEC-002, FEAT-21.SPEC-004), never an email
- In-app or push delivery -- excluded per product-features.md and the dependency map, which define email as the product's sole notification channel; a security-sensitive confirmation particularly needs to reach Nadia outside the product, since it exists precisely for the case where her account access itself may be compromised

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email, sent as two separate messages: one to the prior sign-in email address and one to the new sign-in email address | Always, immediately once the change is confirmed | The prior address is the one an unauthorized change cannot redirect, so it is what makes an unauthorized change visible to Nadia; the new address confirms the change landed where intended. Nadia works from a laptop or desktop but is not necessarily inside the product at the moment a change she did not make takes effect; email is the one channel guaranteed to reach her outside the session in which the change occurred, and is the product's sole notification channel for the account-critical case this confirmation exists to cover |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Sign-in email/login method change confirmed | FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Fires when FEAT-21.SPEC-005's re-verification completes and the new sign-in email is committed to the Freelancer Account | Prior sign-in email, new sign-in email, time of confirmation |

## Audience and Preferences

**Recipient addresses:** exactly two, both belonging to Nadia's own account and each receiving its own copy: (1) `{prior_email}` -- the sign-in email as it stood immediately before the change committed; (2) `{new_email}` -- the sign-in email just committed. No other address (no operator, no client contact) is ever a recipient.

**Recipients:** Nadia (Freelancer) -- the sole role that can hold or change a sign-in email on the Freelancer Account. Per the Access Matrix, no other role (Owen, Priya, Dana) has any sign-in credential to change, so no other role can ever trigger or receive this notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| (none) | -- | Always sent | -- (this is a transactional, record-core communication; XBR-30 places it outside the optional preferences governed by FEAT-21.SPEC-002 and FEAT-21.SPEC-008, since it reports a security-sensitive change to Nadia's own credentials) |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for any notification, and even where a preference model existed, a confirmation of a change to the freelancer's own sign-in credentials is exactly the kind of account-critical event that must never be held; it is delivered the instant the change is confirmed.

## Content Definition

**Email:**
- **Subject:** Your Clientroom sign-in email was changed
- **Body:**
  Hi {freelancer_name},

  Your Clientroom sign-in email was changed from {prior_email} to {new_email} on {change_timestamp}.

  If you made this change, no action is needed.

  If you did not make this change, sign out other devices from Login & Security in Settings immediately and contact support.
- **CTA (button):** Go to Login & Security -- deep-links to FEAT-21.SPEC-003 (Login & Security) for Nadia's own account

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_name} | Freelancer Account -- `name` | Nadia Voss | Greeting renders as "Hi," -- name is never empty in practice since it is required (FEAT-21.SPEC-007), but the fallback exists for defensive completeness |
| {prior_email} | Freelancer Account -- sign-in email, value as it stood immediately before this change committed | nadia@oldstudio.com | Never empty -- a confirmed change always has a prior value, since the Freelancer Account is created with a sign-in email at sign-up (FEAT-20) |
| {new_email} | Freelancer Account -- sign-in email, the value FEAT-21.SPEC-005 just committed | nadia@newstudio.com | Never empty -- FEAT-21.SPEC-005 only fires this notification after committing a non-empty, validated new email |
| {change_timestamp} | Derived -- the moment FEAT-21.SPEC-005 committed the change, shown in Nadia's own time zone (FEAT-15) | September 27, 2026, 2:14 PM | Never empty -- always set at the moment of confirmation |

## Delivery Rules

**Batching:** None -- each confirmed change produces exactly one email per recipient address (two in total: one to `{prior_email}`, one to `{new_email}`), sent individually. A rapid sequence of changes (e.g., Nadia changes her email, then changes it again shortly after) produces one confirmation per confirmed change, never batched, since each is independently security-relevant.
**Deduplication:** At most one confirmation per confirmed change per recipient address. FEAT-21.SPEC-005 fires this notification exactly once, at the moment its commit step succeeds; a re-verification link followed twice for the same change (FEAT-21.SPEC-005's own edge case) triggers this notification only on the first, successful commit.
**Retry on failure:** Each recipient address is delivered and retried independently, so a failure at one address never delays or cancels the other. Delivery failure (including a bounce or invalid-address report) is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure at either address (or both), the failure is surfaced to Nadia as a delivery warning, consistent with XBR-30's "delivery failures are surfaced to the freelancer as warnings"; because this confirmation is account-level rather than project-level, the warning surfaces on her Account Profile screen (FEAT-21.SPEC-001) as the dismissible delivery-warning banner defined there, rather than on a project; one banner covers any number of failed addresses and it never names an email address.
**Expiry:** This notification never expires unsent in a way that discards it -- it is not time-bound to a window the way a reminder is. If delivery ultimately fails after all retries, the failure warning (above) is the surviving signal; the confirmed change itself is never reverted because its confirmation email could not be delivered.

## Edge Cases

- **Nadia changes her sign-in email twice in quick succession (the second change confirms before the first email is delivered)** -- Both confirmed changes are notified independently; each sends its own pair of emails (to that change's prior address and new address), each naming its own prior and new email accurately, so the first change's prior address and the second change's prior address (the first change's new address) each receive the relevant message.
- **The Freelancer Account is deleted (FEAT-24) shortly after a change is confirmed but before this email is delivered** -- Both emails are still delivered, to `{prior_email}` and to `{new_email}`, as a final security record of what happened to the account, since it is a transactional, evidentiary confirmation rather than a live-data view; FEAT-24's account deletion does not retract already-triggered transactional emails.
- **Nadia did not make the change (a compromised session initiated it)** -- The content's explicit "If you did not make this change" guidance is the product's mitigation; the copy sent to `{prior_email}` is the safeguard: it reaches the address the attacker did not control before the change, so Nadia sees the change even if the new address is attacker-controlled. There is no further technical safeguard this notification performs.
- **The transactional email delivery capability reports either the prior or the new email address as invalid or bouncing** -- Treated identically to any other delivery failure for that address: retried per the Retry rule above, then surfaced as the delivery warning on FEAT-21.SPEC-001; the copy to the other address is unaffected and still delivered, and the sign-in email change itself remains committed and in effect regardless of whether either confirmation email could be delivered.
- **Quiet hours or a preference change between trigger and delivery** -- Not applicable: this notification has no preference control and no quiet-hours window (see Audience and Preferences), so there is no collision to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Triggered by (inbound) | Fires this notification exactly once, on confirmed commit |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggers (outbound) | Delivers the email and reports delivery/bounce/failure status |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound) | The CTA deep-links here so Nadia can immediately review sign-in security and sign out other sessions if the change was not hers |
| FEAT-21.SPEC-001 (Account Profile) | References (outbound) | Delivery-failure warnings for this notification surface here |

## Analytics and Success Signals

- **account_critical_change_confirmation_sent** (change_type: email) -- N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Communications and Signals fields so the confirmation is observable
- **account_critical_change_confirmation_delivery_failed** (retry_count) -- N/A -- no connected success-metrics.md metric; retained so a failed security confirmation is observable rather than silent, consistent with XBR-30's delivery-failure-warning requirement
- **account_critical_change_confirmation_cta_tapped** () -- N/A -- no connected success-metrics.md metric; retained to observe whether Nadia acts on a confirmation by reviewing Login & Security

## Acceptance Criteria

**FEAT-21.SPEC-011-AC-01:** Given Nadia's sign-in email change is confirmed by FEAT-21.SPEC-005, when this notification fires, then one email with the subject "Your Clientroom sign-in email was changed" is sent to her prior sign-in address and one to her new sign-in address, each naming her prior and new email accurately.

**FEAT-21.SPEC-011-AC-02:** Given Nadia receives this confirmation email, when she taps "Go to Login & Security", then she lands on FEAT-21.SPEC-003 (Login & Security).

**FEAT-21.SPEC-011-AC-03:** Given Nadia changes her sign-in email twice in quick succession, when both changes confirm, then each confirmed change sends its own separate pair of confirmation emails (prior address and new address), never merged.

**FEAT-21.SPEC-011-AC-04:** Given there is no notification preference for this email, when Nadia's account has every optional notification turned off on FEAT-21.SPEC-002, then this confirmation is still sent for any confirmed sign-in email change.

**FEAT-21.SPEC-011-AC-05:** Given delivery of this email fails once for a transient reason, when the delivery capability retries within platform parameter: `transactional-email-retry-window`, then up to platform parameter: `transactional-email-retry-count` retries occur before any failure is surfaced to Nadia.

**FEAT-21.SPEC-011-AC-06:** Given delivery of this email fails after all retries are exhausted, when the final failure is processed, then the delivery-warning banner "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." appears on Nadia's Account Profile screen (FEAT-21.SPEC-001) until she taps "Dismiss".

**FEAT-21.SPEC-011-AC-07:** Given Nadia's Freelancer Account is deleted shortly after a change is confirmed but before this email is delivered, when delivery proceeds, then the email is still sent to both the prior and the new sign-in email address.

**FEAT-21.SPEC-011-AC-08:** Given FEAT-21.SPEC-005's re-verification link is followed a second time for an already-confirmed change, when that second follow is processed, then no second confirmation email is sent (only the first, successful commit triggers this notification).

**FEAT-21.SPEC-011-AC-09:** Given the new sign-in email address bounces on delivery, when the bounce is reported, then it is handled as any other delivery failure for that address under the Retry on failure rule, the copy to the prior address is unaffected, and the sign-in email change remains in effect regardless.

**FEAT-21.SPEC-011-AC-10:** Given the prior sign-in email address bounces on delivery, when the bounce is reported, then it is retried under the Retry on failure rule, the copy to the new address is still delivered, and after the final failure the delivery-warning banner appears on FEAT-21.SPEC-001.

**FEAT-21.SPEC-011-AC-11:** Given a sign-in email change is confirmed from `{prior_email}` to `{new_email}`, when this notification fires, then the only recipient addresses are `{prior_email}` and `{new_email}`, and no other address receives it.

**FEAT-21.SPEC-011-AC-12:** Given Owen, Priya, or Dana have no sign-in credential on the Freelancer Account to change, then none of them can ever trigger or receive this notification.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always sent, no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
