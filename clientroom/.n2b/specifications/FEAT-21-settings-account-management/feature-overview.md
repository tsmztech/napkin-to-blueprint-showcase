---
document_type: feature-overview
feature_number: FEAT-21
feature_name: Settings & Account Management
feature_slug: settings-account-management
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 11
screen_count: 4
automation_count: 2
logic_rule_count: 4
integration_count: 0
notification_count: 1
---

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
