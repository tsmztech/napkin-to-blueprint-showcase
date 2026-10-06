---
document_type: feature-overview
feature_number: FEAT-15
feature_name: Pro Onboarding & Setup Wizard
feature_slug: pro-onboarding-setup-wizard
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 3
automation_count: 2
logic_rule_count: 2
integration_count: 0
notification_count: 1
---

# Feature Breakdown Brief: Pro Onboarding & Setup Wizard

## Summary

**Feature:** Pro Onboarding & Setup Wizard
**ID:** FEAT-15
**Description:** The guided, one-time setup flow that takes a brand-new Pro from signup to a live, shareable booking link -- services, hours, deposit rule, cancellation policy, and calendar connection, in a sensible order with sensible defaults.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** Every product has a first run, and this one has an unusually high stakes first run: BRIEF.md's Constraints name a three-month runway to the first paying pro, so setup friction directly threatens the founder's timeline. Ranked Important rather than Core because, once complete, the wizard itself is never used again -- the Core features it configures are what deliver ongoing value. MVP phase: a Pro cannot reach any Core feature without it. [RESEARCH-INFORMED: simplicity and fast setup are frequently praised in comparisons aimed at solo operators, contrasted against salon-scale tools, from independent comparison guides (MEDIUM confidence)]

**Key Capabilities:**
- Guided, ordered setup: account and sign-in (FEAT-29) -> profile basics and studio location (FEAT-27) -> at least one service -> working hours -> deposit rule -> cancellation policy -> payout account (FEAT-28) -> (optional) calendar connection -> subscription payment
- A preview of the booking page exactly as a client will see it, plus short plain-language tips at each step
- Sensible defaults offered at each step (e.g., a common cancellation window) that the Pro can accept or change
- A shareable booking link generated the moment setup is minimally complete

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-15.SPEC-001 | Setup Wizard Shell, Step Navigation & Guidance | Screen | The Pro | The persistent wizard frame that shows step order and progress, hands the Pro into each step's owning-feature screen, and surfaces the plain-language "what this step asks for" tip |
| FEAT-15.SPEC-002 | Cancellation Policy Default & First-Version Setup Step | Screen | The Pro | The step where the Pro accepts or adjusts the recommended cancellation window and the first Cancellation Policy version is created |
| FEAT-15.SPEC-003 | Go-Live Preview & Booking Link Hand-Over | Screen | The Pro | The booking-page preview entry point, the live shareable link once activated, and the "here's what to do with it" hand-over note (or, if payout verification is still pending, a "finish verifying to start taking bookings" waiting state) |
| FEAT-15.SPEC-004 | Setup Progress Tracking & Resume | Automation | The Pro, Platform Operator (Support) | Creates the Pro Account's setup-progress record, updates it as each step (including a skipped calendar step) completes, and computes the exact resume point on return; the same progress state is what Support reads read-only |
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | Automation | The Pro | Re-evaluates readiness after every relevant step completion or upstream status change and activates the booking link the moment the Go-Live Prerequisite Rule is satisfied |
| FEAT-15.SPEC-006 | Setup Step Order & Optional-Step Rules | Logic/Rule | The Pro | Defines the fixed step sequence, which single step (calendar connection) is optional and resumable from settings later, and how a skipped step is represented in progress |
| FEAT-15.SPEC-007 | Go-Live Prerequisite Rule (XBR-26 Authority) | Logic/Rule | The Pro | Defines and owns the exact set of steps that must be complete before the booking link can go live, per XBR-26; consumed by SPEC-005 and referenced by other features that gate on go-live status |
| FEAT-15.SPEC-008 | Onboarding Welcome Confirmation | Notification | The Pro | The one-time confirmation sent to the Pro the moment their booking link goes live |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Guided, ordered setup: account and sign-in -> profile basics and studio location -> at least one service -> working hours -> deposit rule -> cancellation policy -> payout account -> (optional) calendar connection -> subscription payment | FEAT-15.SPEC-001, FEAT-15.SPEC-006 | The shell screen displays and drives the sequence; the Logic/Rule spec fixes the order and the one skippable step | Phase 2 (Explicit) |
| A preview of the booking page exactly as a client will see it, plus short plain-language tips at each step | FEAT-15.SPEC-001 (tips), FEAT-15.SPEC-003 (preview entry into FEAT-05's preview mode) | The shell carries the per-step tip copy; the go-live screen is where the full-page preview is entered | Phase 2 (Explicit) |
| Sensible defaults offered at each step (e.g., a common cancellation window) that the Pro can accept or change | FEAT-15.SPEC-002 (the concrete cancellation-window instance FEAT-15 owns), FEAT-15.SPEC-006 (the cross-step "offer a default, let the Pro accept or change it" pattern) | SPEC-002 is the one step whose default value FEAT-15 itself sets and applies (via FEAT-09.SPEC-001); every other step's field-level default (service duration, working hours, etc.) is set by that step's owning feature, and SPEC-006 records that the wizard's role there is hand-off only, not re-derivation | Phase 2 (Explicit) |
| A shareable booking link generated the moment setup is minimally complete | FEAT-15.SPEC-005, FEAT-15.SPEC-007, FEAT-15.SPEC-003 | The rule spec defines "minimally complete"; the automation activates the link the instant it is satisfied; the go-live screen reveals and hands it over | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-15.SPEC-004 | Setup Progress Tracking & Resume | Phase 3 (Entity-Lifecycle) / Phase 4 (Trigger-Response) | The CRUD matrix has no Create spec for the Pro Account's setup-progress state, and the Alternate flow ("wizard resumes exactly where they left off") is a cross-step, cross-session side-effect with no natural home in any single step screen -- too consequential to leave inline |
| FEAT-15.SPEC-006 | Setup Step Order & Optional-Step Rules | Phase 5 (Rule Discovery) | The step order is referenced by SPEC-001 (navigation), SPEC-004 (progress computation) and SPEC-005 (which steps count) -- a rule shared across three specs, past the inline threshold |
| FEAT-15.SPEC-007 | Go-Live Prerequisite Rule (XBR-26 Authority) | Phase 5 (Rule Discovery) | The Validation & Limits field names eight interacting required-step conditions plus one optional step; the dependency map names FEAT-15 as XBR-26's authority, so this rule must be specified once and consumed elsewhere, not re-derived per screen |
| FEAT-15.SPEC-008 | Onboarding Welcome Confirmation | Phase 4 (Notification surfacing) | The Communications field names a message with a real trigger (link goes live) and audience (the Pro); the Phase 4 disposition rule requires a standalone Notification spec rather than an inline confirmation, since it is delivered outside the wizard screen itself (FEAT-08.SPEC-012/013) |

## Entity-Lifecycle Coverage Matrix

**Entity: Pro Account**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-15.SPEC-004 | The account record, plus its setup-progress state, is created the moment the Pro completes sign-in (FEAT-29) and enters the wizard | Sign-in identity itself is established by FEAT-29 (coordination note 4); this feature creates the account envelope that makes resume possible |
| Read (single) | FEAT-15.SPEC-001, FEAT-15.SPEC-004, FEAT-15.SPEC-005 | The shell displays current progress; the tracking automation reads it to compute the resume point; the go-live automation reads it to re-check readiness | -- |
| Read (list) | N/A | Onboarding is a single-account, single-session concern -- there is no list view of Pro Accounts within this feature | -- |
| Update | FEAT-15.SPEC-004 | Setup-progress fields are updated as each step (including a skipped calendar step) completes | Profile fields (display name, photo, studio address, timezone, currency, booking_link_name) are written by FEAT-27, not this feature (coordination note 4) |
| Delete/Archive | N/A -- explicit non-goal | This feature never deletes or archives the Pro Account; removal is owned entirely by FEAT-29's 30-day cooling-off closure, with no independent retention/purge window of its own here | -- |
| State Transition | N/A -- explicit non-goal | The Pro Account's own status field (Active / Paused / Closing / Closed) is governed by FEAT-18 (subscription lapse) and FEAT-29 (closure), not by setup progress; this feature's "readiness" concept (tracked by SPEC-004/007) is a separate, wizard-scoped state that never overwrites account status | -- |

**Entity: Cancellation Policy**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-15.SPEC-002 | The Pro accepts or adjusts the recommended default window; the step applies FEAT-09.SPEC-001's validation (1-168 whole hours, plain-language wording reviewed before saving) and FEAT-09.SPEC-002's versioning (version 1, effective_from) to create the first version | Per coordination note 1: this is the one Connected Entity FEAT-15 creates outright, not merely hands off |
| Read (single) | FEAT-15.SPEC-002 | The step displays the current draft (default or adjusted) before saving | -- |
| Read (list) | N/A | Only one version exists at creation time; version history browsing belongs to FEAT-09, not this feature | -- |
| Update | N/A -- explicit non-goal | Every subsequent edit creates a new version and is owned entirely by FEAT-09 (FEAT-09.SPEC-002); this feature's involvement ends once version 1 is saved | -- |
| Delete/Archive | N/A -- explicit non-goal | Versions are kept while any booking references them (dependency map); this feature never deletes a version | -- |
| State Transition | N/A | The Cancellation Policy entity carries no state field beyond its version/effective_from lineage (dependency map) | -- |

**Referenced Entities (read-only / hand-off only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Service | FEAT-15.SPEC-004, FEAT-15.SPEC-007 | The wizard hands the Pro into FEAT-01.SPEC-002 to create the first service and only reads whether at least one exists, to mark the step complete and to feed the go-live check |
| Availability Rule | FEAT-15.SPEC-004, FEAT-15.SPEC-007 | The wizard hands the Pro into FEAT-02.SPEC-001 and only reads whether working hours are set, to mark the step complete and to feed the go-live check |
| Subscription | FEAT-15.SPEC-004, FEAT-15.SPEC-007 | The wizard hands the Pro into FEAT-18's subscribe capability (coordination note 3) and only reads Active status, to mark the step complete and to feed the go-live check |
| Payout Account | FEAT-15.SPEC-004, FEAT-15.SPEC-005, FEAT-15.SPEC-007 | The wizard hands the Pro into FEAT-28.SPEC-001 and reads status (Active vs. Verification Pending) both to mark the step complete and, per XBR-06/XBR-26, to gate go-live and drive the "finish verifying" waiting state on SPEC-003 |
| Calendar Connection | FEAT-15.SPEC-004, FEAT-15.SPEC-006 | The wizard hands the Pro into FEAT-04.SPEC-001/SPEC-002 and reads whether it was connected or explicitly skipped -- the one step SPEC-006 marks optional |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro completes sign-in and enters the wizard for the first time | Create the Pro Account record and its setup-progress state | Standalone Automation | FEAT-15.SPEC-004 |
| Pro completes any setup step | Update setup-progress state; emit onboarding_step_completed (with step name); compute and show the next step | Standalone Automation | FEAT-15.SPEC-004 |
| Pro closes the app or returns later mid-setup | On return, resume exactly at the saved step with all earlier answers preserved -- never force a restart | Standalone Automation | FEAT-15.SPEC-004 |
| Pro reaches the calendar-connection step | Offer the "skip and connect later from settings" choice; mark the step complete-as-skipped without blocking progress | Standalone Logic/Rule (governs), inline choice in triggering screen (FEAT-04's own screen, per coordination note 2) | FEAT-15.SPEC-006 |
| Pro reaches the cancellation-policy step | Offer the recommended default window; on save, create Cancellation Policy version 1 via FEAT-09.SPEC-001/SPEC-002 rules | Standalone Screen (owns the step), applying an existing spec's rules | FEAT-15.SPEC-002 |
| A required step (sign-in, profile, service, hours, deposit rule, cancellation policy, payout, subscription) becomes complete | Re-evaluate the Go-Live Prerequisite Rule against current progress | Standalone Automation, consuming a Standalone Logic/Rule | FEAT-15.SPEC-005 consuming FEAT-15.SPEC-007 |
| All required steps are satisfied (calendar remains the only optional one) | Activate the booking link the moment readiness is reached; emit onboarding_completed and onboarding_link_shared-eligible state | Standalone Automation | FEAT-15.SPEC-005 |
| Booking link activates | Send the one-time welcome confirmation to the Pro | Standalone Notification | FEAT-15.SPEC-008 |
| Every other required step is complete but the payout account is still Verification Pending | Keep everything else the Pro entered; show "finish verifying to start taking bookings" instead of the live link | Inline in triggering screen, driven by an upstream Integration spec's reported status | FEAT-15.SPEC-003, reading FEAT-28.SPEC-003/SPEC-006 |
| Pro taps the booking-page preview entry point | Render the booking page exactly as a client would see it, without a real payment | Cross-feature (preview mode is FEAT-05's) | FEAT-15.SPEC-003 hands off to FEAT-05.SPEC-001-SPEC-005 |
| A step's submission fails server-side (network error, timeout) | Preserve entered values on that step and offer a retry, consistent with every setup screen in this product | Inline in triggering screen (per the feature's own States field and ASMP-27) | FEAT-15.SPEC-001, FEAT-15.SPEC-002 |
| Platform Operator (Support) opens a Pro's account during a help request | Show read-only setup-progress state; no action is ever available to Support | Inline read access on an existing Automation's data, surfaced by FEAT-19 | FEAT-15.SPEC-004 (data), FEAT-19 (surface) |

The Communications field names exactly one message -- the welcome confirmation once the link goes live -- which is dispositioned into the standalone FEAT-15.SPEC-008 Notification spec, consistent with `notification_count: 1`. This feature reaches every category-level external capability it depends on (payment processing, transactional text/email, calendar sync) entirely through the owning feature's already-specified Integration spec (FEAT-28.SPEC-006, FEAT-18's subscription-billing Integration spec, FEAT-04.SPEC-003, FEAT-08.SPEC-012/013) rather than specifying any of its own, so `integration_count: 0` is the correct, examined disposition rather than an omission.

## Shared Context

**Shared Entities:**
- Pro Account (setup-progress state only) -- created and updated exclusively by SPEC-004; read by SPEC-001 (display), SPEC-005 (readiness check), and SPEC-007 (rule evaluation), and read-only by Platform Operator (Support) via FEAT-19. Profile-identity fields on the same record (display name, studio address, timezone, currency, booking_link_name) belong to FEAT-27, not to this feature's writes.
- Cancellation Policy (version 1 only) -- created by SPEC-002 applying FEAT-09.SPEC-001/SPEC-002 rules; read by SPEC-002 itself before save. No other spec in this feature touches it.

**Shared UI Patterns:**
- Step shell and tip pattern -- SPEC-001 provides the persistent progress/step frame and per-step plain-language tip that every step screen (this feature's own SPEC-002 and SPEC-003, and the hand-off screens owned by FEAT-01, FEAT-02, FEAT-04, FEAT-09, FEAT-18, FEAT-27, FEAT-28, FEAT-29) is shown inside; Spec Writers for those step screens should describe entry into and return from this shell consistently rather than as a bespoke wrapper per step.
- Default-then-adjust pattern -- SPEC-002 shows the one instance FEAT-15 owns (the cancellation window); SPEC-006 records this as the general pattern every hand-off step's owning feature should also follow for its own defaults, so the wizard reads as one consistent experience end to end.

**Shared Validation:**
- SPEC-007 defines the single Go-Live Prerequisite Rule (XBR-26) once; SPEC-005 is its only automation consumer within this feature, and FEAT-07/FEAT-05 read its outcome (link is live or not) rather than re-deriving the condition list.
- SPEC-006 defines step order and the one optional step once; SPEC-001 (navigation), SPEC-004 (progress computation) and SPEC-005 (which steps count toward readiness) all reference it rather than re-stating the sequence.

## Internal Dependency Map

```
FEAT-29 (sign-in) -> [Pro completes sign-in] -> SPEC-004 (Setup Progress Tracking & Resume) -> [Pro Account + progress state created] -> SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance)
SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance) -> [Pro completes a step] -> SPEC-004 (Setup Progress Tracking & Resume) -> [progress updated] -> SPEC-001 (next step shown)
SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance) -> [ordering and skip decisions] -> SPEC-006 (Setup Step Order & Optional-Step Rules)
SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance) -> [reaches cancellation-policy step] -> SPEC-002 (Cancellation Policy Default & First-Version Setup Step) -> [policy version 1 saved] -> SPEC-004 (Setup Progress Tracking & Resume)
SPEC-004 (Setup Progress Tracking & Resume) -> [a required step completes] -> SPEC-007 (Go-Live Prerequisite Rule) -> [readiness re-evaluated] -> SPEC-005 (Go-Live Evaluation & Booking Link Activation)
SPEC-005 (Go-Live Evaluation & Booking Link Activation) -> [link activates] -> SPEC-008 (Onboarding Welcome Confirmation)
SPEC-005 (Go-Live Evaluation & Booking Link Activation) -> [link activates, or payout still pending] -> SPEC-003 (Go-Live Preview & Booking Link Hand-Over)
SPEC-003 (Go-Live Preview & Booking Link Hand-Over) -> [Pro taps preview] -> cross-feature to FEAT-05 (preview mode)
```

**Default Entry:** SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance) -- shown automatically the first time a newly signed-in Pro has no completed setup steps; on any later return with setup incomplete, the shell re-enters at the resume point SPEC-004 computes.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-15.SPEC-001 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Sign-in creation (email, mobile, one-time code) hands control back to the wizard shell | Pro starts setup |
| FEAT-15.SPEC-001 | Outbound | FEAT-27 (Pro Profile & Booking Page Settings) | Wizard hands off into display name, photo, and studio-location capture | Pro completes the sign-in step |
| FEAT-15.SPEC-001 | Outbound | FEAT-01 (Service & Pricing Management) | Wizard hands off into the Add Service screen (FEAT-01.SPEC-002) to create the first service and deposit rule | Pro continues setup |
| FEAT-15.SPEC-001 | Outbound | FEAT-02 (Availability & Working Hours Setup) | Wizard hands off into working hours and buffer capture (FEAT-02.SPEC-001), which reports hours-set back for XBR-26 | Pro continues setup |
| FEAT-15.SPEC-002 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | First Cancellation Policy version is created applying FEAT-09.SPEC-001's validation and FEAT-09.SPEC-002's versioning | Pro saves the cancellation-policy step |
| FEAT-15.SPEC-001 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Wizard hands off into payout account connection (FEAT-28.SPEC-001) through the payment processor's own verification | Pro reaches the getting-paid step |
| FEAT-15.SPEC-005, FEAT-15.SPEC-007 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Payout account Active status (FEAT-28.SPEC-004) satisfies XBR-06/XBR-26's payout condition; a still-pending status drives SPEC-003's waiting state | Payout status is reported by the processor |
| FEAT-15.SPEC-001 | Outbound | FEAT-04 (Two-Way Calendar Sync) | Wizard hands off into calendar connection (FEAT-04.SPEC-001/SPEC-002), the one step marked optional by SPEC-006 | Pro reaches the calendar step (may skip) |
| FEAT-15.SPEC-001, FEAT-15.SPEC-007 | Outbound | FEAT-18 (Pro Subscription Billing & Account Management) | Wizard hands off into the subscribe-with-a-card capability and records it as a go-live prerequisite (coordination note 3) | Pro reaches the subscription step |
| FEAT-15.SPEC-003 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Booking-page preview entry uses FEAT-05's preview mode without a real payment | Pro taps preview |
| FEAT-15.SPEC-008 | Outbound | FEAT-08 (Automated Booking & Messaging) | Welcome confirmation content and audience are defined here; delivery is FEAT-08.SPEC-012 (text) / FEAT-08.SPEC-013 (email) | Booking link goes live |
| FEAT-15.SPEC-004 | Outbound | FEAT-19 (Platform Support Read-Only Access) | Support's view-only surface for how far a Pro has progressed reads this feature's setup-progress state; Support can never act on a step (SC-05) | Support opens a Pro account during a help request |
| FEAT-15.SPEC-001, FEAT-15.SPEC-003 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Anyone not signed in as the Pro is sent to the Pro sign-in screen before reaching any wizard step | Unauthenticated access attempt |

## Non-Functional Notes

**Data volumes / growth:** One setup-progress record and one initial Cancellation Policy version per Pro, created once per account and never repeated; volume scales one-to-one with new Pro sign-ups (a few hundred pros in year one), so no growth pattern of its own beyond normal account creation (assumptions-constraints.md; dependency map, Pro Account lifecycle).

**Responsiveness:** Each step is a simple form with instant local response -- no Loading state applies (feature's own States field); a failed step preserves entered values and offers retry rather than losing work, consistent with ASMP-27's "nothing appears tappable before real data has loaded" and "in-place indicator" rules applied at step-submission granularity.

**Data sensitivity / privacy:** The wizard is the first place several of the product's most sensitive fields are captured -- sign-in email/mobile (FEAT-29), studio address (FEAT-27), and the payout account's identity/bank hand-off (FEAT-28) -- though this feature's own writes are limited to setup-progress state and the (non-sensitive, public) Cancellation Policy wording. Protected end to end by one-time-code sign-in with new-device alerts (ASMP-30); Support's read-only progress view never exposes sign-in codes or bank/identity details (ASMP-30; Access field).

**Compliance flags:** N/A -- no compliance regime attaches to this feature specifically beyond what the capabilities it hands off to already carry (payment-processing identity/bank verification via FEAT-28, per its own Compliance flags entry); this feature never handles card, bank, or identity data itself (SC-11).

## Non-Goals

- **Multi-staff or salon-scale onboarding paths (inviting staff, assigning chairs, role setup)** -- Excluded per SC-01: the product is strictly single-operator, so the wizard has exactly one linear path for one Pro and no staff-invitation or multi-chair step of any kind.
- **Support completing, editing, or advancing a Pro's setup steps on their behalf** -- Excluded per SC-05: Platform Operator (Support) has View-only access to setup progress (Access field); a stuck Pro must resolve their own step, including regaining sign-in access themselves if that is what is blocking them.
- **Importing services, hours, or policies from a prior tool during setup** -- Excluded per SC-09: pros arrive from informal, unstructured processes (DMs, paper diaries, ad hoc Venmo requests) with no structured source worth an import path; every setup field is entered directly by the Pro.
- **Multi-language or translated setup content** -- Excluded per SC-10: the near-term geography (US, UK, Canada, Australia) is entirely English-language, so no translated step content is built.
- **Card, bank, or identity data ever touching this feature's own screens or storage** -- Excluded per SC-11: the getting-paid and subscription steps hand the Pro entirely into the payment processor's own verification and billing flows (FEAT-28, FEAT-18); this feature stores no card, bank, or identity data at any point.
- **Going live with a partial or waived prerequisite (e.g., a link that is "live" without a service, or without an active subscription)** -- Excluded per the feature's own Validation & Limits field (a named Stage 2 decision) and its authority over XBR-26: the eight required steps are a hard gate with no partial or Pro-overridable go-live state; only the calendar step is ever optional.
- **Independent deletion or archival of the Pro Account or its setup-progress state within this feature** -- Intentional lifecycle decision surfaced by the CRUD matrix: this feature never deletes the account it creates; removal is owned entirely by FEAT-29's 30-day cooling-off closure, with no separate retention/purge window of its own.
