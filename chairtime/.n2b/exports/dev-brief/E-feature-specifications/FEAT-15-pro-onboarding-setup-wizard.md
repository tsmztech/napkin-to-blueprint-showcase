# FEAT-15 — Pro Onboarding & Setup Wizard

This chapter covers Pro Onboarding & Setup Wizard (FEAT-15), a Important-tier feature. It carries 8 specifications carrying 103 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-15.SPEC-001 | Setup Wizard Shell, Step Navigation & Guidance | screen | 14 |
| FEAT-15.SPEC-002 | Cancellation Policy Default & First-Version Setup Step | screen | 12 |
| FEAT-15.SPEC-003 | Go-Live Preview & Booking Link Hand-Over | screen | 13 |
| FEAT-15.SPEC-004 | Setup Progress Tracking & Resume | automation | 12 |
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | automation | 11 |
| FEAT-15.SPEC-006 | Setup Step Order & Optional-Step Rules | logic-rule | 15 |
| FEAT-15.SPEC-007 | Go-Live Prerequisite Rule (XBR-26 Authority) | logic-rule | 14 |
| FEAT-15.SPEC-008 | Onboarding Welcome Confirmation | notification | 12 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Setup Wizard Shell, Step Navigation & Guidance

## Overview

**Name:** Setup Wizard Shell, Step Navigation & Guidance
**ID:** FEAT-15.SPEC-001
**Type:** Screen
**Purpose:** The persistent wizard frame that shows Talia her step order and progress, hands her into each step's owning-feature screen in sequence, and surfaces a short plain-language tip for whichever step she is on.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- The persistent step-progress frame (step list, current position, completed/current/upcoming/skipped status per step)
- Launching the Pro into each step's owning-feature screen in the fixed order defined by FEAT-15.SPEC-006
- The plain-language "what this step asks for" tip shown for the current step
- The always-available booking-page preview entry point (hands off to FEAT-05's preview mode)
- Reflecting resume position on return, as computed by FEAT-15.SPEC-004

**Non-Goals:**
- Collecting any step's actual field data (name, price, hours, bank details, card number) -- each step's owning feature (FEAT-29, FEAT-27, FEAT-01, FEAT-02, FEAT-28, FEAT-04, FEAT-18) owns its own form and validation; this shell only launches into it and reads back completion
- Creating the first Cancellation Policy version -- owned by FEAT-15.SPEC-002, this feature's own step screen, not the shell
- Computing or storing setup-progress state -- owned by FEAT-15.SPEC-004; this shell only displays what that automation reports
- Deciding whether the booking link may go live -- owned by FEAT-15.SPEC-007 (rule) and FEAT-15.SPEC-005 (automation); this shell only navigates to FEAT-15.SPEC-003 once told setup is complete
- Multi-staff or multi-chair setup paths -- excluded per SC-01: the product is strictly single-operator, so the shell has exactly one linear path for one Pro

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29 (Pro sign-in creation) | Talia signs in for the first time with zero completed setup steps | None -- shell opens at step 1 (account & sign-in, already satisfied by the sign-in that just completed), so the shell opens showing step 2 (profile) as current |
| FEAT-29 (Pro sign-in) | Talia signs in with setup already in progress | Resume point computed by FEAT-15.SPEC-004 -- the shell opens directly at the first incomplete step with all earlier answers preserved |
| Any step's owning screen (FEAT-27, FEAT-01.SPEC-002, FEAT-02.SPEC-001, FEAT-28.SPEC-001, FEAT-04.SPEC-001, FEAT-18's subscribe screen) | Talia completes that step and the owning screen hands control back | Updated progress state (that step now marked complete); shell advances to the next step in order |
| FEAT-15.SPEC-002 (Cancellation Policy step, owned by this feature) | Talia saves the first Cancellation Policy version | Updated progress state; shell advances to the next step |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Navigate into any step in fixed order, review a completed step, tap the preview entry point | -- |
| The Client (Riley) | No | No | This screen is never reached by a client path; a client following any wizard-shaped link is sent to the public booking page (FEAT-05) or a "this booking page isn't available" message if the link does not resolve to a Pro |
| Platform Operator (Support) | No (Support does not use this wizard screen) | No | Support views a Pro's setup progress read-only through Platform Support Read-Only Access (FEAT-19), which reads the same progress state (FEAT-15.SPEC-004) surfaced in a support-facing view -- Support never opens this Pro-facing shell |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); after signing in, a brand-new account lands on this shell at step 2, and a returning incomplete account lands at its resume point |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no wizard data is lost because every completed step was already persisted by FEAT-15.SPEC-004 at the moment it completed; after re-authentication, the shell reopens at the same resume point it showed before the session expired |

## Layout and Content

**Header:** A step-progress indicator reading "Step {current_number} of 8: {current_step_name}" with a horizontal list of all 8 steps beneath it, each rendered with one of four states: Completed (checkmark), Current (highlighted), Upcoming (dimmed, not tappable), Skipped (shown only for the calendar step once Talia has explicitly skipped it, per FEAT-15.SPEC-006). A "Preview my booking page" action sits at the top-right of the header, available from the first step onward.

**Body:** A single content card for the current step, containing:
- The step's title (e.g., "Set your working hours")
- The plain-language tip: one to three sentences explaining, in non-technical terms, what this step asks for and why (e.g., for the cancellation-policy step: "Choose how far ahead of an appointment a client can cancel and still get their deposit back. Most pros start with 24 hours -- you can change this any time later.")
- A "Continue" button that launches the step's owning-feature screen

Completed steps in the header list are tappable: tapping one navigates into that step's owning screen in its own edit mode (e.g., tapping the completed "Services" step opens FEAT-01's Service List so Talia can review or add another service before continuing) without disturbing later steps' saved answers. Upcoming steps are not tappable -- their titles are visible but dimmed, communicating the fixed order without allowing Talia to jump ahead.

**Footer:** None -- the "Continue" action lives in the body card, and "Preview my booking page" lives in the header.

### Responsive Behavior

- **Compact breakpoint (phone width):** The step list collapses to a horizontally scrollable strip above the current-step card; the current step is always scrolled into view on load. The current-step card and its Continue button remain full width.
- **Medium size class and above:** The step list renders as a vertical rail to the left of the current-step card rather than a horizontal strip; no other structural change.
- **Preview action:** Remains a single tappable label at every size; it never collapses into an icon-only control, since a first-time Pro must recognize it by its words.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Step list item (completed step) | Tap | Navigate into that step's owning-feature screen in review/edit mode | Shell frame remains visible around the destination screen | Destination screen opens; on return, the shell re-displays with progress unchanged unless the Pro made a new change there |
| Step list item (current step) | Tap | Same as tapping "Continue" for the current step | -- | Owning-feature screen opens |
| Step list item (upcoming step) | Tap | No action -- not interactive | None | Dimmed appearance communicates it is not yet reachable; no error is shown, since this is expected, not a mistake |
| Step list item (skipped calendar step) | Tap | Navigate into FEAT-04's calendar connection screen (FEAT-04.SPEC-001) | -- | Talia can connect the calendar at any point after skipping it, from here or later from settings |
| "Continue" button | Tap | Navigate to the current step's owning-feature screen (FEAT-29, FEAT-27, FEAT-01.SPEC-002, FEAT-02.SPEC-001, FEAT-15.SPEC-002, FEAT-28.SPEC-001, FEAT-04.SPEC-001, or FEAT-18's subscribe screen, per the fixed order in FEAT-15.SPEC-006) | Shell hands control to the destination screen | Destination screen opens |
| "Preview my booking page" | Tap | Navigate to FEAT-05's preview mode for this Pro's current (possibly incomplete) profile and services | None on the shell itself | FEAT-05 opens in preview mode, showing the booking page exactly as a client would see it today, with no real payment possible |
| Step's owning screen completes and hands back | System event (not a direct tap) | FEAT-15.SPEC-004 updates progress; shell re-reads progress and advances the current-step indicator | Header progress list updates; body card shows the next step's title and tip | Brief transition to the next step's card |
| Final required step completes | System event | FEAT-15.SPEC-005 re-evaluates readiness (FEAT-15.SPEC-007); if satisfied, the shell navigates automatically | Shell is replaced by FEAT-15.SPEC-003 | Talia lands on the Go-Live Preview & Booking Link Hand-Over screen |

### Accessibility Notes

- **Focus order:** Step-progress list (left to right / top to bottom) -> "Preview my booking page" -> current-step title -> tip text -> "Continue" button.
- **Dynamic-change announcements:** When the shell advances to a new step after a hand-off screen returns, the new step's title and tip are announced to assistive technology as a single update, so Talia does not have to re-discover her position by re-reading the whole header.
- **Keyboard alternatives:** Every step-list item and the "Continue" and "Preview" actions are reachable and activatable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Fresh start | Header shows step 2 of 8 as current (step 1, account & sign-in, is already satisfied by the sign-in that opened this screen); all later steps upcoming | Talia signs in for the first time with zero completed steps | Talia taps Continue on step 2 |
| In progress | Header reflects whichever steps are complete, current, upcoming, or skipped, per FEAT-15.SPEC-004's progress record | Talia returns to the shell with some steps already complete | Talia advances a step, or the automatic hand-off to FEAT-15.SPEC-003 fires |
| Awaiting hand-off return | Shell frame persists in the background conceptually while a step's owning screen is open (from the Pro's perspective, the owning screen has the focus) | Talia taps Continue or a step-list item | The owning screen hands control back to the shell |
| Complete, hand-off pending | Brief transition state after the final required step reports complete and before FEAT-15.SPEC-005/007 finish evaluating readiness | The final required step (subscription, if it is completed last) reports complete | Readiness confirmed and the shell navigates to FEAT-15.SPEC-003 |
| Error | The shell itself shows no error state of its own -- a failed step submission is handled entirely inside that step's owning screen (each preserves entered values and offers retry, consistent with every setup screen in this product); the shell only ever shows a step as "not yet complete" until the owning screen reports success | A step's owning screen fails to save | Talia retries and succeeds inside the owning screen, which then reports completion back to the shell |
| Offline/Degraded | N/A -- setup is a deliberate, connected session on a stable connection between clients, not an in-the-moment mobile flow (feature's own States field); the shell requires connectivity to read and advance progress | -- | -- |

## Validation Rules

Validation governed by FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) for step sequencing and the one skippable step. This shell performs no field-level validation of its own -- every step's data validation is owned by that step's owning-feature spec (e.g., FEAT-01.SPEC-004 for service fields, FEAT-02.SPEC-005 for availability fields).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Continue on account & sign-in step | Pro sign-in creation screen | FEAT-29 (Pro Sign-In & Account Lifecycle) |
| Continue on profile step | Profile & studio location screen (FEAT-27.SPEC-001) | FEAT-27 (Pro Profile & Booking Page Settings) |
| Continue on service step | Add Service screen (FEAT-01.SPEC-002) | FEAT-01 (Service & Pricing Management) |
| Continue on hours step | Working Hours setup screen (FEAT-02.SPEC-001) | FEAT-02 (Availability & Working Hours Setup) |
| Continue on cancellation-policy step | FEAT-15.SPEC-002 (Cancellation Policy Default & First-Version Setup Step) | -- |
| Continue on payout step | Payout Account Connection screen (FEAT-28.SPEC-001) | FEAT-28 (Payout Account Connection & Payout Visibility) |
| Continue on calendar step (or tapping the skipped step later) | Calendar Connection Setup screen (FEAT-04.SPEC-001) | FEAT-04 (Two-Way Calendar Sync) |
| Continue on subscription step | Subscribe screen | FEAT-18 (Pro Subscription Billing & Account Management) |
| "Preview my booking page" | Booking page preview mode | FEAT-05 (Public Booking Page & Booking Flow) |
| All required steps satisfied | FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | -- |

## Data Model

**Creates:** None -- this shell creates no data of its own; account and progress creation is owned by FEAT-15.SPEC-004.
**Reads:** Pro Account's setup-progress state (which of the 8 steps are complete/current/skipped, per FEAT-15.SPEC-004's record) -- read on every load and after every hand-off return.
**Updates:** None -- the shell never writes progress state directly; it only triggers FEAT-15.SPEC-004 to re-read and re-render after a step's owning screen reports completion.
**Deletes:** None.

## Business Rules

- Step order and the calendar step's optional/skippable status are governed entirely by FEAT-15.SPEC-006 -- this shell never re-derives or overrides the sequence.
- Progress is computed and persisted entirely by FEAT-15.SPEC-004 -- this shell is a read-and-navigate surface, not a state owner.
- Go-Live readiness (XBR-26) is evaluated entirely by FEAT-15.SPEC-007 via FEAT-15.SPEC-005 -- the shell hands off to FEAT-15.SPEC-003 the moment it is told readiness is reached, and never itself decides that setup is "done enough."
- The booking-page preview (FEAT-05's preview mode) is available from the first step onward and never requires setup to be complete, consistent with the Key Capability "a preview of the booking page exactly as a client will see it."

## Edge Cases

- **Talia is signed in on two devices and completes different steps on each at nearly the same time** -- Each device's owning-feature screen (e.g., FEAT-01.SPEC-002 on one device, FEAT-02.SPEC-001 on the other) saves its own step independently; FEAT-15.SPEC-004 records both completions (they touch disjoint progress fields, so there is no overwrite). On next load, each shell re-reads progress and shows both steps as complete -- no conflict dialog is needed because step completions are additive, not competing edits to the same field, consistent with the Pro Account entity's last-write-wins resolution for non-overlapping fields.
- **Talia navigates directly to the shell URL without having started setup** -- Not reachable: the shell is entered only through the FEAT-29 sign-in hand-off or a resume from a signed-in session; a signed-in Pro with zero progress is shown step 2 as current, per Entry Points.
- **Talia taps "Preview my booking page" before any service exists** -- FEAT-05's preview mode shows the "temporarily not accepting bookings" empty state that a zero-service booking page shows, so Talia can see what an incomplete page looks like to a client and is motivated to keep going.
- **Talia backs out of a step's owning screen without saving (e.g., closes the browser mid-form)** -- That step's owning screen preserves entered values and treats this the same as any other unsaved-exit case for that screen; the shell still shows the step as incomplete on her return, and she resumes exactly where FEAT-15.SPEC-004 last recorded her.
- **A step is completed out of the displayed order through a settings deep link (for example, Talia reaches FEAT-27 profile settings directly after account creation, from a source outside the wizard)** -- FEAT-15.SPEC-004 still records the completion; the shell's step list marks that step complete even though it was not reached by tapping Continue, and the current-step indicator advances to the next incomplete step in fixed order.
- **The final required step completes while Talia's payout account is still Verification Pending** -- The shell still hands off to FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over), which shows the "finish verifying to start taking bookings" waiting state rather than a live link, per FEAT-15.SPEC-007's payout condition.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | References (inbound) | The shell reads current progress and resume point from this automation on every load and after every hand-off return |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Governs the fixed step order and the calendar step's skip behavior shown in the header list |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Triggers (outbound) | Firing after each step completion; the shell hands off to FEAT-15.SPEC-003 once this automation reports readiness |
| FEAT-15.SPEC-002 (Cancellation Policy Default & First-Version Setup Step) | Navigation (outbound) | Continue on the cancellation-policy step launches this sibling screen |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Navigation (outbound) | Automatic hand-off once all required steps are satisfied |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound and outbound) | Sign-in creation hands control to this shell; the account & sign-in step's Continue launches into FEAT-29 |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (outbound) | Profile step's Continue launches into FEAT-27 |
| FEAT-01.SPEC-002 (Add Service) | Navigation (outbound) | Service step's Continue launches into FEAT-01's Add Service screen |
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Navigation (outbound) | Hours step's Continue launches into this screen |
| FEAT-28.SPEC-001 (Payout Account Connection) | Navigation (outbound) | Payout step's Continue launches into this screen |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Navigation (outbound) | Calendar step's Continue (or a later tap on the skipped step) launches into this screen |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | "Preview my booking page" launches FEAT-05's preview mode |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| onboarding_started | none | Talia's first entry into this shell after sign-in with zero completed steps | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_step_completed | step name, step number, whether it was reached via Continue or a direct deep link | Any step's owning screen reports completion back to the shell | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_preview_from_wizard_tapped | current step number at time of tap | Talia taps "Preview my booking page" | supports success-metrics.md: "Setup-to-Live-Link Completion" |

## Acceptance Criteria

**FEAT-15.SPEC-001-AC-01:** Given Talia has just completed sign-in for the first time, when the wizard shell opens, then it shows "Step 2 of 8: Profile & Studio Location" as the current step with all later steps shown upcoming and dimmed.

**FEAT-15.SPEC-001-AC-02:** Given Talia is on the current-step card for working hours, when she reads the tip text, then it explains in plain language what setting working hours means for her booking page, with no technical terms.

**FEAT-15.SPEC-001-AC-03:** Given Talia taps "Continue" on the current step, when the tap registers, then she is navigated to that step's owning-feature screen (for example, FEAT-01.SPEC-002 for the service step).

**FEAT-15.SPEC-001-AC-04:** Given Talia completes the working-hours step on FEAT-02.SPEC-001 and is handed back to the shell, when the shell re-renders, then "Working Hours" shows as Completed in the step list and "Step 5 of 8: Cancellation Policy" becomes current.

**FEAT-15.SPEC-001-AC-05:** Given Talia looks at an upcoming step in the header list, when she taps it, then nothing happens -- the step remains dimmed and not navigable.

**FEAT-15.SPEC-001-AC-06:** Given Talia has already completed the service step, when she taps that completed step in the header list, then she is navigated to FEAT-01's Service List in review/edit mode without disturbing her progress on later steps.

**FEAT-15.SPEC-001-AC-07:** Given Talia is on any step of the wizard, when she taps "Preview my booking page", then FEAT-05 opens in preview mode showing the booking page exactly as a client would see it, with no real payment possible.

**FEAT-15.SPEC-001-AC-08:** Given Talia skips the calendar-connection step per FEAT-15.SPEC-006, when the shell re-renders, then that step shows as "Skipped" in the header list rather than blocking progress to the subscription step.

**FEAT-15.SPEC-001-AC-09:** Given Talia later taps the skipped calendar step from the header list, when the tap registers, then she is navigated into FEAT-04.SPEC-001 to connect her calendar.

**FEAT-15.SPEC-001-AC-10:** Given Talia completes the final required step (subscription) and her payout account is already Active, when FEAT-15.SPEC-005 confirms readiness, then the shell automatically navigates her to FEAT-15.SPEC-003.

**FEAT-15.SPEC-001-AC-11:** Given Talia completes the final required step but her payout account is still Verification Pending, when the shell hands off, then she still lands on FEAT-15.SPEC-003, which shows the "finish verifying to start taking bookings" waiting state rather than a live link.

**FEAT-15.SPEC-001-AC-12:** Given Talia is completing setup on her phone with a stable connection and then loses connectivity, when she tries to advance a step, then the shell requires connectivity to proceed (per its Offline/Degraded state), consistent with setup being a deliberate connected session.

**FEAT-15.SPEC-001-AC-13:** Given an unauthenticated visitor reaches the wizard shell URL, when the screen loads, then they are redirected to the Pro sign-in screen (FEAT-29) and never see any wizard content.

**FEAT-15.SPEC-001-AC-14:** Given Talia's session expires while the wizard shell is open, when she next interacts with the screen, then a dialog reads "Your session has expired. Sign in to continue." and, after she signs back in, the shell reopens at the exact resume point it showed before expiry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (fresh start, in progress, awaiting hand-off return, complete/hand-off pending, error, offline/degraded) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Cancellation Policy Default & First-Version Setup Step

## Overview

**Name:** Cancellation Policy Default & First-Version Setup Step
**ID:** FEAT-15.SPEC-002
**Type:** Screen
**Purpose:** Talia accepts or adjusts a recommended cancellation window and saves it, creating the first Cancellation Policy version for her account.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- Presenting the recommended default cancellation window and plain-language wording
- Letting Talia accept the default or adjust the window within the product's allowed range
- Saving the first Cancellation Policy version (version 1) by applying FEAT-09.SPEC-001's validation and FEAT-09.SPEC-002's versioning rules
- Reporting step completion back to the wizard shell (FEAT-15.SPEC-001) once saved

**Non-Goals:**
- Editing the policy after this first version is saved -- every subsequent edit creates a new version and is owned entirely by FEAT-09 (FEAT-09.SPEC-002); this screen's involvement ends once version 1 is saved
- Defining the field-level validation and versioning rules themselves -- owned by FEAT-09.SPEC-001 (Cancellation Policy Setup) and FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering); this screen applies those rules rather than restating them
- Partial refunds or tiered cancellation schedules -- excluded per scope-boundaries.md SC-18: the product's cancellation rule is binary (full refund outside the window, deposit kept inside it or on a no-show), so no tiered percentage option is offered here
- Multi-language policy wording -- excluded per scope-boundaries.md SC-10: the near-term geography is entirely English-language, so no translated wording variant is built

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Talia taps "Continue" on the cancellation-policy step, or taps this step in the header list after already completing it | If completing for the first time: none, the form starts with the recommended default pre-filled. If revisiting a completed step: N/A -- this screen has no edit path after version 1 is saved (see Non-Goals); tapping the completed step instead shows the saved version 1 wording read-only with a note that changes are made from Cancellation & No-Show settings later (FEAT-09) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Accept or adjust the recommended window and save version 1 | -- |
| The Client (Riley) | No | No | Never reaches this setup screen; a client sees only the resulting plain-language policy on the public booking page (FEAT-05) once it exists |
| Platform Operator (Support) | No | No | Support has View-only access to the Cancellation & No-Show Handling capability group generally, but this specific setup-step screen is never opened by Support; Support views the saved policy through FEAT-09's own read surfaces if needed for a help request |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); the wizard shell re-enters at this step on return if it was the resume point |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any adjustment Talia made to the window before saving is preserved locally and restored after re-authentication succeeds, since nothing is persisted until Save |

## Layout and Content

**Header:** Step title "Cancellation Policy" with the wizard shell's persistent step-progress indicator above it (owned by FEAT-15.SPEC-001).

**Body:**
- A short explanation in plain language: what the cancellation window controls (when a client can still get their deposit back) and why it protects Talia from last-minute cancellations
- A recommended-window display: "We recommend platform parameter: `cancellation-window-default-hours` hours before the appointment" shown as the pre-filled value in an editable numeric field labeled "Cancellation window (hours before appointment)"
- A live preview of the exact plain-language wording a client will see at booking, updating as Talia adjusts the number (for example: "Cancel or reschedule at least {window} hours before your appointment for a full refund. Cancelling later, or not showing up, means your deposit is kept.")
- A "Save and continue" button

**Footer:** None -- Save is in the body, directly below the preview.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described; the live wording preview sits directly beneath the numeric field, both full width.
- **Medium size class and above:** The numeric field and its live preview render side by side in two columns instead of stacked; no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Cancellation window field | Type or use stepper controls | Captures the numeric hour value | Live wording preview updates to reflect the new value | Preview text updates in place as the field changes |
| Cancellation window field | Blur with an out-of-range value | Triggers validation via FEAT-09.SPEC-001 | Field shows error state | Exact error message from FEAT-09.SPEC-001 (window must be a whole number of hours from 1 to 168) |
| "Save and continue" button | Tap | 1. Validate the window via FEAT-09.SPEC-001. 2. If valid, create Cancellation Policy version 1 via FEAT-09.SPEC-002's versioning rules (version = 1, effective_from = now). 3. Report step completion to FEAT-15.SPEC-004. | Button shows a brief saving state | Success: step marked complete, wizard shell (FEAT-15.SPEC-001) advances to the next step. Failure: inline error, entered value preserved |
| "Save and continue" (while saving) | Tap | No action -- debounced | None | Button remains in its saving state |

### Accessibility Notes

- **Focus order:** Explanation text -> cancellation window field -> live wording preview (read-only, announced as a live region) -> "Save and continue" button.
- **Dynamic-change announcements:** When the field's value changes, the updated wording preview is announced to assistive technology as a polite live-region update, not an interruption. A validation error is announced immediately and associated with the field.
- **Keyboard alternatives:** The numeric field's stepper controls (increment/decrement) are reachable and operable by keyboard (arrow keys), with direct typing as an equivalent alternative.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (recommended) | Field pre-filled with platform parameter: `cancellation-window-default-hours`; preview shows the resulting wording; Save enabled | Screen first opens | Talia edits the field, or taps Save with the default unchanged |
| Adjusted | Field shows Talia's entered value; preview reflects it; Save enabled if valid | Talia types a new value | Talia saves, or clears the field back toward a valid state |
| Validation Error | Field shows error state and message from FEAT-09.SPEC-001 | Entered value fails validation (not 1-168 whole hours) | Talia corrects the value |
| Saving | Save button shows a loading indicator; field disabled | Talia taps Save with a valid value | Save completes or fails |
| Error | Error banner "Could not save your cancellation policy. Check your connection and try again." with a Retry action; entered value preserved | Save operation fails | Talia taps Retry and the save succeeds |
| Offline/Degraded | N/A -- this is a setup screen used on a stable connection between clients, not an in-the-moment mobile flow (feature's own States field); saving requires connectivity | -- | -- |

## Validation Rules

Validation governed by FEAT-09.SPEC-001 (Cancellation Policy Setup). See that spec for the exact window bounds (1-168 whole hours) and error messaging. This screen checks the field on blur and again on Save.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Successful save | FEAT-15.SPEC-001 (Setup Wizard Shell) advances to the next step (payout account) | -- |

## Data Model

**Creates:** Cancellation Policy record -- version 1, with window_hours (the value Talia accepted or adjusted), inside_window_outcome (deposit kept, fixed at binary v1), outside_window_outcome (full refund, fixed), plain_language_wording (rendered from the template shown in the live preview), effective_from (the moment of save). All field names and the entity's versioning behavior match the Cancellation Policy entity in the Feature Dependency Map exactly, applying FEAT-09.SPEC-002's versioning rules.
**Reads:** The recommended default window value (platform parameter: `cancellation-window-default-hours`), pre-filled on load.
**Updates:** None -- this screen's involvement ends at the creation of version 1; every later edit is owned by FEAT-09.SPEC-002.
**Deletes:** None.

## Business Rules

- The window must be a whole number of hours from 1 to 168 (7 days), per FEAT-09.SPEC-001.
- The policy created here is always version 1 with effective_from set to the moment of save, per FEAT-09.SPEC-002's versioning rule -- no earlier version can exist for a new Pro Account.
- The deposit outcome is binary in v1 (full refund outside the window, deposit kept inside it or on a no-show) per BRIEF.md's Business Context; this screen offers no tiered or partial-refund option.
- This is the one Connected Entity FEAT-15 creates outright rather than merely handing off to another feature's screen, per the Brief's Shared Context.

## Edge Cases

- **Talia saves the recommended default without changing it** -- Version 1 is created with window_hours equal to platform parameter: `cancellation-window-default-hours`, identical in every respect to a version she had explicitly typed that same number for.
- **Talia enters 0 hours** -- Rejected by FEAT-09.SPEC-001's validation (minimum 1 hour); the field shows the exact error message from that spec and Save does not proceed.
- **Talia enters 200 hours** -- Rejected by FEAT-09.SPEC-001's validation (maximum 168 hours); same treatment as above.
- **Talia saves, then immediately re-enters this step from the wizard shell's header list** -- Because this screen has no edit path after version 1 is saved (see Non-Goals), she instead sees the saved wording read-only with a note directing further changes to Cancellation & No-Show settings (FEAT-09), never a second "create version 1" form.
- **Network failure during save** -- Error banner appears with a Retry action; the entered window value is preserved exactly as typed, so Talia never has to re-enter it.
- **Talia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in its saving state); only one Cancellation Policy version 1 is ever created.
- **Concurrent-edit conflict** -- Not applicable: the Cancellation Policy entity is created here, not updated, and this is the only screen in the entire product that can create version 1 for a given Pro Account; no other actor can be mid-edit on a record that does not yet exist. Once version 1 exists, every subsequent edit is exclusively owned by FEAT-09.SPEC-002's own screen, which carries its own concurrency handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Navigation (inbound and outbound) | Launches this screen on Continue; receives control back on successful save |
| FEAT-09.SPEC-001 (Cancellation Policy Setup) | References (outbound) | Field-level validation (1-168 whole hours) applied by this screen |
| FEAT-09.SPEC-002 (Policy Versioning & Cutoff Rendering) | References (outbound) | Versioning rule (version 1, effective_from) applied when this screen saves |
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Triggers (outbound) | Successful save reports this step complete |
| FEAT-05 (Public Booking Page & Booking Flow) | References (outbound, indirect) | The plain-language wording created here is later displayed on the public booking page once the link goes live |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| onboarding_cancellation_policy_saved | window_hours saved, whether the recommended default was accepted unchanged or adjusted | Save completes successfully | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_cancellation_policy_validation_failed | attempted value, reason (below minimum / above maximum) | Save attempted with an out-of-range value | N/A -- no Stage 2 metric measures setup-form validation failures directly; retained so this step's friction is observable rather than invisible |

## Acceptance Criteria

**FEAT-15.SPEC-002-AC-01:** Given Talia reaches the cancellation-policy step for the first time, when the screen loads, then the window field is pre-filled with platform parameter: `cancellation-window-default-hours` and the wording preview reflects that value.

**FEAT-15.SPEC-002-AC-02:** Given Talia is on this screen with the default value unchanged, when she taps "Save and continue", then a Cancellation Policy version 1 is created with that window and she advances to the payout step.

**FEAT-15.SPEC-002-AC-03:** Given Talia types "48" into the window field, when the field updates, then the live wording preview immediately reflects "48 hours" in its text.

**FEAT-15.SPEC-002-AC-04:** Given Talia enters "0" into the window field and moves focus away, when validation runs (FEAT-09.SPEC-001), then the field shows an error state with that spec's exact minimum-hours error message and Save does not proceed.

**FEAT-15.SPEC-002-AC-05:** Given Talia enters "200" into the window field, when validation runs, then the field shows FEAT-09.SPEC-001's exact maximum-hours error message.

**FEAT-15.SPEC-002-AC-06:** Given Talia adjusts the window to "72" and taps Save, when the save completes, then the created Cancellation Policy version 1 has window_hours equal to 72 and effective_from set to the moment of save.

**FEAT-15.SPEC-002-AC-07:** Given a network failure occurs during save, when the failure is detected, then an error banner reads "Could not save your cancellation policy. Check your connection and try again." with a Retry action, and the entered window value remains in the field.

**FEAT-15.SPEC-002-AC-08:** Given Talia taps "Save and continue" twice in rapid succession, when the second tap registers, then it is ignored while the first save is in progress, and exactly one Cancellation Policy version 1 is created.

**FEAT-15.SPEC-002-AC-09:** Given Talia has already saved version 1 and returns to this step from the wizard shell's completed-step list, when the screen loads, then it shows the saved wording read-only with a note that further changes are made from Cancellation & No-Show settings, and no new version is created.

**FEAT-15.SPEC-002-AC-10:** Given a client (Riley) attempts to reach this screen directly, when the request is made, then it is refused and Riley is shown no wizard content of any kind.

**FEAT-15.SPEC-002-AC-11:** Given an unauthenticated visitor reaches this screen's URL, when the screen would load, then they are redirected to the Pro sign-in screen (FEAT-29) instead.

**FEAT-15.SPEC-002-AC-12:** Given Talia's session expires while she has adjusted the window but not yet saved, when she next interacts with the screen, then the expiry dialog appears, and after she signs back in, her adjusted (unsaved) value is restored in the field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (default, adjusted, validation error, saving, error, offline/degraded) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Screen Spec: Go-Live Preview & Booking Link Hand-Over

## Overview

**Name:** Go-Live Preview & Booking Link Hand-Over
**ID:** FEAT-15.SPEC-003
**Type:** Screen
**Purpose:** Once every required setup step is satisfied, this screen reveals Talia's live shareable booking link with a plain hand-over note on what to do with it -- or, if her payout account is still verifying, shows a "finish verifying to start taking bookings" waiting state instead.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- The screen Talia lands on once the wizard shell (FEAT-15.SPEC-001) hands off after the final required step
- Displaying the live, shareable booking link once FEAT-15.SPEC-005 has activated it
- The booking-page preview entry point into FEAT-05's preview mode
- The "finish verifying to start taking bookings" waiting state when every other required step is complete but the payout account is still Verification Pending
- The plain "here's what to do with it" hand-over note once the link is live

**Non-Goals:**
- Deciding whether the link may go live -- owned by FEAT-15.SPEC-007 (rule) and FEAT-15.SPEC-005 (automation); this screen only displays the outcome of that decision
- Rendering the actual booking page a client sees -- owned by FEAT-05; this screen only links to it (live) or previews it (FEAT-05's preview mode)
- Resolving a payout verification problem -- owned by FEAT-28's own screens (FEAT-28.SPEC-001, FEAT-28.SPEC-003); this screen only shows the waiting state and a path back into FEAT-28's flow
- Sending the welcome confirmation -- owned by FEAT-15.SPEC-008 (Notification), triggered by the same go-live event this screen displays, delivered outside this screen entirely

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | The final required step completes and FEAT-15.SPEC-005 confirms readiness (or reports the payout condition still pending) | Whether the link is live or the payout-pending waiting state applies, per FEAT-15.SPEC-007's evaluation |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Talia finishes payout verification after having already reached this screen once in the waiting state | Updated payout status; FEAT-15.SPEC-005 re-evaluates and this screen refreshes from waiting to live |
| FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | Talia taps "View my link" in the go-live welcome confirmation | None -- this Pro's own go-live state and booking link load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Tap the preview entry point; copy or share the live link once activated; tap through to finish payout verification while waiting | -- |
| The Client (Riley) | No | No | Never reaches this screen; a client who follows the live link itself is sent to FEAT-05's public booking page, not this hand-over screen |
| Platform Operator (Support) | No | No | Support has no reason to open this screen directly; a Pro's go-live status is visible to Support only through the read-only setup-progress view surfaced by FEAT-19 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- after re-authentication, the Pro returns to this same screen showing the same live-or-waiting state as before, since nothing here depends on unsaved input |

## Layout and Content

**Header:** Screen title, either "You're live!" (link activated) or "Almost there" (payout verification pending), depending on state.

**Body (Live state):**
- The full shareable booking link, displayed as plain text with a "Copy link" action
- A "Preview my booking page" action, opening FEAT-05's preview mode
- The hand-over note: plain-language guidance on what to do next (e.g., "Add this link to your Instagram bio so clients can book and pay their deposit in under a minute.")

**Body (Waiting state -- payout still Verification Pending):**
- A plain message: "Finish verifying your payout account to start taking bookings." (mirrors FEAT-28's own wording for this exact condition)
- A summary confirming every other step is done ("Everything else is ready -- services, hours, cancellation policy, and your subscription are all set.")
- A "Finish verifying" action, navigating into FEAT-28's verification flow
- The "Preview my booking page" action remains available even while waiting, since preview never requires an active payout account

**Footer:** None -- all actions are in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; the link display wraps rather than truncates so Talia can always read it in full.
- **Medium size class and above:** The link display and the hand-over note render side by side in two columns instead of stacked; no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Copy link" (Live state only) | Tap | Copies the booking link to the clipboard | None on screen state | Toast confirmation "Link copied" |
| "Preview my booking page" | Tap | Navigate to FEAT-05's preview mode | None on this screen | FEAT-05 opens in preview mode |
| "Finish verifying" (Waiting state only) | Tap | Navigate into FEAT-28's payout verification flow (FEAT-28.SPEC-001) | None on this screen until Talia returns | FEAT-28's verification screen opens |
| (Automatic) Payout status changes to Active while waiting | System event | FEAT-15.SPEC-005 re-evaluates readiness (FEAT-15.SPEC-007) and, finding it satisfied, activates the link | Screen transitions from Waiting to Live | The "Finish verifying" panel is replaced by the live link and hand-over note; the welcome confirmation (FEAT-15.SPEC-008) is sent independently of this screen's state |

### Accessibility Notes

- **Focus order (Live state):** Screen title -> booking link text -> "Copy link" -> "Preview my booking page" -> hand-over note.
- **Focus order (Waiting state):** Screen title -> summary message -> "Finish verifying" -> "Preview my booking page".
- **Dynamic-change announcements:** The transition from Waiting to Live (when payout verification completes) is announced to assistive technology as a state change, since it can happen without Talia performing an action on this screen (she may have completed verification via a notification link, not by returning here first).
- **Keyboard alternatives:** All actions on this screen are reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Live | Shareable link, Copy action, preview action, and hand-over note shown | FEAT-15.SPEC-005 has activated the booking link (FEAT-15.SPEC-007's conditions all satisfied) | Talia navigates away (this is her setup destination; the wizard is now complete and she typically continues to her daily dashboard, FEAT-12) |
| Waiting (payout pending) | "Finish verifying" panel shown in place of the link; preview action still available | Every other required step is complete but the payout account is Verification Pending | Payout account becomes Active, at which point FEAT-15.SPEC-005 re-evaluates and this screen transitions to Live automatically |
| Loading | N/A -- readiness is evaluated before this screen is reached (by FEAT-15.SPEC-005), so the screen never renders in an intermediate "evaluating" state visible to Talia | -- | -- |
| Error | If the link or status cannot be retrieved on load, the last-known state (live link or waiting message) is shown with the time it was loaded and a Retry action, rather than a blank screen | Status retrieval fails | Talia taps Retry and the current status loads successfully |
| Offline/Degraded | The last-loaded state (live link or waiting message) remains viewable read-only; "Copy link" still works from the cached link text; "Finish verifying" and "Preview my booking page" require connectivity and are shown disabled with a plain explanation until it returns | Connectivity lost while this screen is open | Connectivity restored -- the screen re-checks status and refreshes normally |

## Validation Rules

Validation governed by FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) for whether the link may be Live or must show the Waiting state. This screen performs no field-level input validation of its own -- it has no form fields.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Preview my booking page" | Booking page preview mode | FEAT-05 (Public Booking Page & Booking Flow) |
| "Finish verifying" | Payout Account Connection screen (FEAT-28.SPEC-001) | FEAT-28 (Payout Account Connection & Payout Visibility) |

## Data Model

**Creates:** None.
**Reads:** Pro Account's booking_link_name and go-live status (derived by FEAT-15.SPEC-005 from FEAT-15.SPEC-007's evaluation); Payout Account's status field (Active vs. Verification Pending), read from FEAT-28.
**Updates:** None -- this screen never writes go-live status or payout status directly; both are owned by their respective automations (FEAT-15.SPEC-005 and FEAT-28).
**Deletes:** None.

## Business Rules

- The link is shown as Live only when FEAT-15.SPEC-007's Go-Live Prerequisite Rule (XBR-26) is satisfied -- this screen never shows a partially-live or Pro-overridable state, per the feature's own explicit non-goal.
- Every other required step being complete while payout verification is pending is the one named waiting condition this screen defines, per XBR-06 ("no deposit can be taken, and the booking link cannot go live, unless the Pro's payout account is active").
- The preview entry point (FEAT-05's preview mode) is available in both the Live and Waiting states, since it never requires an active payout account or a live link.

## Edge Cases

- **Talia reaches this screen while payout is Verification Pending, then completes verification on a different device without returning here first** -- The next time she opens this screen (or if she has it open and connectivity permits a status refresh), FEAT-15.SPEC-005 has already re-evaluated and activated the link; the screen shows Live directly, never requiring her to retrigger anything from this screen.
- **The processor later flags Talia's payout account as needing action after the link has already gone live** -- Out of scope for this screen: an already-Active payout account that later needs action is handled entirely by FEAT-28's own attention-flow (dashboard banner and notification); this screen's Waiting state applies only during the original setup sequence, before the link has ever gone live.
- **Talia taps "Copy link" while offline, using the last-loaded link text** -- The copy succeeds from cached text since it requires no network call; if the link was never successfully loaded before going offline, the action is shown disabled with a plain "reconnect to copy your link" explanation instead.
- **Talia's booking link renders with the correct name before she has renamed it from the default suggested at account creation** -- This screen displays whatever booking_link_name currently exists on the Pro Account (owned by FEAT-27); renaming it later (FEAT-27) does not require returning to this screen, and the forwarding behavior for a renamed link (XBR-27) is entirely FEAT-27's concern.
- **Concurrent-edit conflict** -- Not applicable: this screen performs no writes of its own; go-live status and payout status are each owned and serialized by their respective automations (FEAT-15.SPEC-005, FEAT-28), so there is no field on this screen a second actor could be editing concurrently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Navigation (inbound) | Hands off to this screen once the final required step completes |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | References (inbound) | Determines whether this screen shows Live or Waiting, and activates the link this screen displays |
| FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) | References (inbound) | Defines the exact condition set this screen's Live/Waiting split is based on |
| FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | References (inbound, indirect) | Fires from the same go-live event this screen displays, delivered independently of whether Talia is viewing this screen |
| FEAT-28.SPEC-001 (Payout Account Connection) | Navigation (outbound) | "Finish verifying" launches into this screen |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | References (inbound) | Reports the Verification Pending / Active status this screen reads |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | "Preview my booking page" opens FEAT-05's preview mode; the live link itself points here for clients |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| onboarding_link_shared | share method (copy) | Talia taps "Copy link" | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_preview_from_golive_tapped | state at time of tap (live / waiting) | Talia taps "Preview my booking page" from this screen | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_payout_wait_shown | none | This screen renders the Waiting state | supports success-metrics.md: "Setup-to-Live-Link Completion" (a payout-pending wait is friction between "began onboarding" and "reached a live link," directly relevant to this metric's completion window) |

## Acceptance Criteria

**FEAT-15.SPEC-003-AC-01:** Given Talia completes her final required setup step and her payout account is already Active, when FEAT-15.SPEC-005 confirms readiness, then this screen shows "You're live!" with her shareable booking link and a hand-over note.

**FEAT-15.SPEC-003-AC-02:** Given Talia is on the Live state, when she taps "Copy link", then the link is copied to her clipboard and a "Link copied" toast appears.

**FEAT-15.SPEC-003-AC-03:** Given Talia completes her final required step but her payout account is Verification Pending, when this screen loads, then it shows "Almost there" with the message "Finish verifying your payout account to start taking bookings." and no live link is shown.

**FEAT-15.SPEC-003-AC-04:** Given Talia is on the Waiting state, when she taps "Finish verifying", then she is navigated into FEAT-28's payout verification flow (FEAT-28.SPEC-001).

**FEAT-15.SPEC-003-AC-05:** Given Talia is on the Waiting state and completes payout verification, when her payout account becomes Active, then this screen transitions automatically from Waiting to Live without requiring her to take any further action on this screen.

**FEAT-15.SPEC-003-AC-06:** Given Talia is on either the Live or Waiting state, when she taps "Preview my booking page", then FEAT-05 opens in preview mode showing the booking page exactly as a client would see it.

**FEAT-15.SPEC-003-AC-07:** Given Talia's booking link goes live, when the transition occurs, then FEAT-15.SPEC-008's welcome confirmation is sent independently, regardless of whether Talia is currently viewing this screen.

**FEAT-15.SPEC-003-AC-08:** Given status retrieval fails when this screen loads, when the failure is detected, then the last-known state is shown with the time it was loaded and a Retry action, rather than a blank screen.

**FEAT-15.SPEC-003-AC-09:** Given Talia loses connectivity while viewing the Live state, when she taps "Copy link", then the copy still succeeds using the cached link text.

**FEAT-15.SPEC-003-AC-10:** Given Talia loses connectivity while viewing the Waiting state, when she taps "Finish verifying", then the action is shown disabled with a plain explanation to reconnect, rather than silently failing.

**FEAT-15.SPEC-003-AC-11:** Given a client (Riley) follows Talia's live booking link, when the link is opened, then Riley lands on FEAT-05's public booking page, never on this hand-over screen.

**FEAT-15.SPEC-003-AC-12:** Given an unauthenticated visitor reaches this screen's URL directly, when the request is made, then they are redirected to the Pro sign-in screen (FEAT-29).

**FEAT-15.SPEC-003-AC-13:** Given Talia's session expires while she is on this screen, when she next interacts with it, then the expiry dialog appears, and after signing back in she returns to the same Live-or-Waiting state she saw before expiry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (live, waiting, loading, error, offline/degraded) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Setup Progress Tracking & Resume

## Overview

**Name:** Setup Progress Tracking & Resume
**ID:** FEAT-15.SPEC-004
**Type:** Automation
**Purpose:** Creates the Pro Account's setup-progress record the moment Talia first enters the wizard, updates it as each step (including a skipped calendar step) completes, and computes the exact resume point whenever she returns -- the same progress state Support reads read-only.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- Creating the Pro Account record and its setup-progress state on first entry into the wizard
- Updating setup-progress as each of the 8 steps (per FEAT-15.SPEC-006's order) completes, including the calendar step marked complete-as-skipped
- Computing the exact resume point (the first incomplete step) whenever Talia returns to the wizard shell
- Serving the same progress state read-only to Platform Operator (Support) via FEAT-19

**Non-Goals:**
- Establishing sign-in identity itself (email, mobile, one-time code) -- owned by FEAT-29; this automation creates the account envelope that makes resume possible once sign-in has already happened
- Deciding step order or which step is optional -- owned by FEAT-15.SPEC-006; this automation only records completion against that fixed order
- Evaluating or acting on Go-Live readiness -- owned by FEAT-15.SPEC-007 and FEAT-15.SPEC-005; this automation only supplies the progress data those specs evaluate
- Deleting or archiving the Pro Account or its progress state -- excluded per the feature's own explicit non-goal: removal is owned entirely by FEAT-29's 30-day cooling-off closure, with no independent retention/purge window here

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro completes sign-in and enters the wizard for the first time | FEAT-29 (Pro Sign-In & Account Lifecycle) | Fires exactly once per Pro Account, the first time sign-in completes with no existing setup-progress record | Sign-in identity reference from FEAT-29 |
| A setup step's owning screen reports completion | FEAT-15.SPEC-001 (Setup Wizard Shell), by way of any step's owning screen (FEAT-29, FEAT-27, FEAT-01.SPEC-002, FEAT-02.SPEC-001, FEAT-15.SPEC-002, FEAT-28.SPEC-001, FEAT-04.SPEC-001, or FEAT-18's subscribe screen) | Fires each time a step's owning screen signals its data was saved successfully | The step identifier that completed, and any data the automation needs to independently confirm completion (see Processing Logic) |
| Talia reaches the calendar-connection step and chooses "skip and connect later" | FEAT-04.SPEC-001 (Calendar Connection Setup), per FEAT-15.SPEC-006's optional-step rule | Fires when Talia makes the explicit skip choice on FEAT-04's own screen | Skip choice, timestamp |
| Talia returns to the wizard shell (any session after the first) | FEAT-15.SPEC-001 (Setup Wizard Shell) | Fires every time the shell loads for a Pro Account with an existing progress record | Current progress record |
| Platform Operator (Support) opens a Pro's account during a help request | FEAT-19 (Platform Support Read-Only Access) | Fires when Support requests a read of setup-progress state for a specific Pro Account | The same progress record, served read-only |

## Processing Logic

1. On first sign-in completion for a Pro Account with no existing progress record: create the Pro Account record (its identity fields are populated by FEAT-29 separately) and create its setup-progress state with all 8 steps marked incomplete except "account & sign-in," which is marked complete immediately (sign-in having just succeeded).
2. On a step-completion signal from any owning screen: verify the step's underlying data genuinely exists (for example, for the "profile" step, confirm the Pro Account's display_name and studio_address fields -- written by FEAT-27 -- are both populated; for the "services" step, confirm at least one Service record exists for this Pro Account; for the "payout" step, confirm a Payout Account record exists in at least Verification Pending status) before marking that step complete -- this automation never marks a step complete on a screen's say-so alone.
3. Mark the verified step complete in the progress record, with a timestamp.
4. If the step is the calendar-connection step and the signal is a skip choice, mark that step complete-as-skipped rather than complete-as-connected, per FEAT-15.SPEC-006.
5. Emit onboarding_step_completed with the step's name.
6. Re-derive the resume point: the first step, in the fixed order defined by FEAT-15.SPEC-006, that is not yet complete (complete-as-skipped counts as complete for this purpose).
7. Notify FEAT-15.SPEC-005 that a required step has changed state, so it can re-evaluate Go-Live readiness.
8. On a shell-load resume request: return the current progress record and the resume point computed in step 6, with every earlier step's saved answers intact (each step's owning feature retains its own data; this automation returns only completion state, not the underlying field values).
9. On a Support read request: return the same progress record read-only, with no action affordance attached, and log the view per FEAT-19's own logging behavior (this automation does not itself write the log entry; it only serves the data FEAT-19 logs having viewed).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Account and progress created | First sign-in completion for a new Pro Account | Pro Account and setup-progress records created; "account & sign-in" step marked complete | Wizard shell opens showing step 2 as current | FEAT-15.SPEC-001 |
| Step marked complete | A step's owning screen reports success and this automation independently verifies the underlying data exists | Progress record updated for that step | Wizard shell advances to the next step | FEAT-15.SPEC-001, FEAT-15.SPEC-005 |
| Step marked complete-as-skipped | The calendar step's skip choice is signaled | Progress record updated for the calendar step with a skipped flag | Wizard shell shows that step as Skipped rather than blocking | FEAT-15.SPEC-001, FEAT-15.SPEC-006 |
| Verification failed (no-op) | A step-completion signal arrives but the underlying data does not actually exist (for example, a signal fires but no Service record can be found) | No change to the progress record | The step is not advanced; Talia sees the step's owning screen show its own save error rather than a false "complete" state | FEAT-15.SPEC-001 (owning screen's own error handling) |
| Resume point computed | Talia returns to the wizard shell with an existing progress record | None (read-only computation) | Wizard shell opens directly at the first incomplete step with earlier answers preserved | FEAT-15.SPEC-001 |
| Support view served | Support requests a read during a help request | None (read-only) | Support sees the same progress state, with no action available | FEAT-19 |
| Automation failure | Processing error while marking a step complete or computing resume | No partial or inconsistent progress state is ever persisted (see Business Rules) | The triggering screen shows a generic "something went wrong, try again" retry state, consistent with every setup screen's own error handling | FEAT-15.SPEC-001 (owning screen's own error handling) |

## Data Model

**Reads:** Pro Account (display_name, studio_address existence check for the profile step, written by FEAT-27), Service (existence check for the services step), Availability Rule (existence check for the hours step), Cancellation Policy (existence check for the policy step, created by FEAT-15.SPEC-002), Payout Account (status check for the payout step), Calendar Connection (existence or skip-flag check for the calendar step), Subscription (Active status check for the subscription step) -- all read-only, existence/status checks only, per the dependency map's Referenced Entities table for FEAT-15.
**Creates:** Pro Account record (identity envelope; profile fields are written separately by FEAT-27, per the dependency map) and its setup-progress state, on first sign-in completion.
**Updates:** Pro Account's setup-progress state -- the per-step completion flags (complete / complete-as-skipped / incomplete) and timestamps, exclusively owned and written by this automation, per the Brief's Shared Context.
**Deletes:** None -- per the feature's own explicit non-goal, this automation never deletes or archives the Pro Account or its progress state.

## Business Rules

- This automation is the sole writer of the Pro Account's setup-progress state (Brief's Shared Context) -- no other spec, including the wizard shell itself, ever writes progress directly.
- A step is never marked complete on a screen's completion signal alone -- this automation independently verifies the underlying entity exists (or, for the calendar step, that an explicit skip choice was made) before recording completion, so a false-positive "complete" state can never occur from a screen bug.
- The calendar-connection step is the only step that can be marked complete-as-skipped; every other step requires genuine completion of its owning screen's flow, per FEAT-15.SPEC-006.
- Resume always lands on the first incomplete step in the fixed order defined by FEAT-15.SPEC-006 -- Talia is never forced to restart from step 1, and earlier steps' saved answers (held by their owning features) are never touched by a resume.
- Platform Operator (Support) reads this same progress state read-only through FEAT-19, and can never act on a step, per SC-05 and XBR-24.

## Edge Cases

- **A step's owning screen signals completion, but the underlying record was deleted or never actually saved (e.g., a race between two tabs)** -- Step 2 of Processing Logic's independent verification catches this: the step is not marked complete, and the owning screen's own error handling applies rather than the wizard silently advancing on a false signal.
- **Talia completes the same step twice (e.g., adds a second service after the services step was already marked complete)** -- No effect on progress: the step is already marked complete, and this automation does not re-fire or duplicate the completion record; the additional service is simply additional data owned by FEAT-01.
- **Talia abandons setup for weeks and returns** -- The resume point is recomputed exactly as it would be for a same-day return; no step decays or times out, consistent with the feature's own "never forcing a restart" alternate flow.
- **Talia skips the calendar step, then connects a calendar later from settings** -- The progress record's calendar-step flag updates from complete-as-skipped to complete-as-connected; this has no effect on Go-Live readiness (FEAT-15.SPEC-007 treats both as satisfying the optional step) but is reflected accurately for Support's read-only view.
- **Concurrent trigger firing (Talia completes two different steps on two devices at nearly the same time)** -- Each completion signal touches a disjoint field in the progress record (one step's flag each); both are recorded independently with no overwrite, consistent with the Pro Account entity's last-write-wins resolution for non-overlapping fields in the dependency map's Contention note.
- **Trigger fires while a previous run is in flight (two completion signals for the same step arrive close together, e.g., a double-submitted save)** -- The second signal for an already-complete step is a no-op (see the "completes the same step twice" case above); the automation's per-step verification is idempotent, so re-running it against an already-satisfied condition produces the same "complete" outcome without side effects.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Triggered by (inbound) | First sign-in completion triggers account and progress creation |
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Triggered by (inbound), Affects (outbound) | Every step-completion signal and shell-load resume request flows through this automation; the shell reflects this automation's output |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Triggered by (inbound) | The explicit skip choice signals complete-as-skipped for the calendar step |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Defines the fixed step order this automation tracks against and the calendar step's skip semantics |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Affects (outbound) | Notified after every progress change so it can re-evaluate readiness |
| FEAT-19 (Platform Support Read-Only Access) | Affects (outbound) | Serves the same progress record read-only for Support's help-request view |

## Analytics and Success Signals

- **onboarding_started** (none) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_step_completed** (step name, whether connected or skipped where applicable) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_completed** (total elapsed time from onboarding_started, count of steps skipped) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_referral_source** -- N/A -- no field in the Pro Account or any entity this automation reads captures how a new Pro heard about Chairtime; success-metrics.md's "Peer-Referral Growth Share" (also connected to this feature) has no in-product signal available anywhere in FEAT-15 to feed it and is tracked by the founder outside the product (for example, by asking new pros directly), not by an automation event.

## Acceptance Criteria

**FEAT-15.SPEC-004-AC-01:** Given Talia completes sign-in for the very first time, when this automation fires, then a Pro Account record and its setup-progress state are created with the "account & sign-in" step already marked complete.

**FEAT-15.SPEC-004-AC-02:** Given Talia has just added her first service on FEAT-01.SPEC-002, when that screen reports completion, then this automation verifies a Service record exists, marks the services step complete, and emits onboarding_step_completed with "services."

**FEAT-15.SPEC-004-AC-03:** Given a step-completion signal arrives for the payout step but no Payout Account record can be found, when this automation runs its verification, then the step is not marked complete and no false progress is recorded.

**FEAT-15.SPEC-004-AC-04:** Given Talia chooses "skip and connect later" on the calendar step, when FEAT-04.SPEC-001 signals the skip, then this automation marks the calendar step complete-as-skipped.

**FEAT-15.SPEC-004-AC-05:** Given Talia has completed steps 1 through 4 and closes the app, when she signs back in days later, then the wizard shell resumes exactly at step 5 with steps 1 through 4 shown complete.

**FEAT-15.SPEC-004-AC-06:** Given Talia skipped the calendar step during setup and later connects a calendar from settings, when the connection succeeds, then this automation updates the calendar step's flag from complete-as-skipped to complete-as-connected.

**FEAT-15.SPEC-004-AC-07:** Given Platform Operator (Support) opens Talia's account during a help request, when they view her setup progress via FEAT-19, then they see the same progress state read-only, with no action available to them.

**FEAT-15.SPEC-004-AC-08:** Given Talia completes two different steps on two different devices at nearly the same time, when both completion signals are processed, then both steps are recorded as complete with no overwrite of the other.

**FEAT-15.SPEC-004-AC-09:** Given Talia's services step is already marked complete, when a duplicate completion signal for that same step arrives, then the automation makes no change and does not emit a second onboarding_step_completed event for that step.

**FEAT-15.SPEC-004-AC-10:** Given every required step has become complete, when the last one is recorded, then this automation notifies FEAT-15.SPEC-005 to re-evaluate Go-Live readiness.

**FEAT-15.SPEC-004-AC-11:** Given Talia has never used a calendar connection and setup otherwise completes, when readiness is evaluated, then the calendar step's incomplete/skipped state has no bearing on any other step's recorded completion.

**FEAT-15.SPEC-004-AC-12:** Given a completion signal for a step whose underlying data existed a moment before but was deleted before verification ran, when this automation processes the signal, then the step remains marked incomplete and the owning screen's own retry-capable error handling governs what Talia sees next.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 | 5 |
| Outcome Paths | 7 | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Go-Live Evaluation & Booking Link Activation

## Overview

**Name:** Go-Live Evaluation & Booking Link Activation
**ID:** FEAT-15.SPEC-005
**Type:** Automation
**Purpose:** Re-evaluates readiness against the Go-Live Prerequisite Rule after every relevant step completion or upstream status change, and activates Talia's booking link the moment it is satisfied.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- Re-evaluating Go-Live readiness whenever a required step's completion state changes (FEAT-15.SPEC-004) or an upstream status this rule depends on changes (payout account status, subscription status)
- Activating the booking link (making it publicly reachable at FEAT-05) the instant readiness is reached
- Emitting onboarding_completed and the state that makes FEAT-15.SPEC-008's welcome confirmation eligible to fire
- Reporting readiness (or its absence, with the specific gap) to FEAT-15.SPEC-003 for display

**Non-Goals:**
- Defining which steps are required and which is optional -- owned entirely by FEAT-15.SPEC-007 (Go-Live Prerequisite Rule, XBR-26 authority); this automation only consumes that rule's evaluation
- Tracking step completion itself -- owned by FEAT-15.SPEC-004; this automation is notified of changes, it does not compute them
- Rendering the live link or the waiting state to Talia -- owned by FEAT-15.SPEC-003; this automation only supplies the activation event that screen displays
- Deactivating a link once live (for example, on subscription lapse or account pause) -- owned by FEAT-27 (pause state, XBR-14) and FEAT-18 (subscription lapse); this automation's activation is a one-time, one-directional transition from not-live to live, never the reverse

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A required step's completion state changes | FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Fires every time any of the eight steps transitions between complete, complete-as-skipped, and incomplete | Current progress record for all 8 steps |
| Payout account status changes | FEAT-28.SPEC-003 (Payout Account Status Processing) | Fires when the Payout Account's status changes to or from Active | Payout Account status |
| Subscription status changes | FEAT-18.SPEC-006 (FEAT-18's subscription-billing automation; external-event trigger source, per the External Touchpoints table) | Fires when the Subscription's status changes to or from Active | Subscription status |

## Processing Logic

1. On any trigger, read the current state of all seven prerequisites defined by FEAT-15.SPEC-007's Go-Live Prerequisite Rule: sign-in complete, display name and studio location set, at least one Service exists (which by definition includes a deposit rule, per FEAT-01's Service entity), working hours set, the Cancellation Policy version 1 exists, Payout Account status is Active, and Subscription status is Active.
2. Evaluate FEAT-15.SPEC-007's rule against that state: all seven required conditions (the calendar step is explicitly excluded from this evaluation, per XBR-26) must be true.
3. If the booking link is already active, and the rule is still satisfied, take no action (idempotent -- this automation never re-activates an already-live link or emits a duplicate completion event).
4. If the booking link is not yet active and the rule is now satisfied: activate the link (make FEAT-05's public booking page reachable at Talia's booking_link_name), record the activation timestamp, and emit onboarding_completed.
5. If the booking link is not yet active and the rule is not yet satisfied: take no activation action; compute which specific condition(s) are still unmet (for display purposes) and make that gap available to FEAT-15.SPEC-003.
6. If the booking link is already active and a later trigger reports one of the seven conditions has become false (for example, a payout account regresses from Active to Action Required after the link is already live) -- this automation does not deactivate the link: deactivation on an already-live link is out of scope (see Non-Goals) and owned by the account-pause/subscription-lapse mechanisms in FEAT-27 and FEAT-18, not by this one-directional activation automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Link activated | All seven required conditions are satisfied for the first time | Booking link marked active; activation timestamp recorded | Talia sees the Live state on FEAT-15.SPEC-003; the welcome confirmation (FEAT-15.SPEC-008) becomes eligible and fires | FEAT-15.SPEC-003, FEAT-15.SPEC-008, FEAT-05 |
| Readiness still pending | One or more required conditions remain unmet | No change | Talia sees the Waiting state on FEAT-15.SPEC-003 if the payout condition specifically is the gap, or remains inside the wizard shell (FEAT-15.SPEC-001) for any other unmet condition, since the shell does not hand off to FEAT-15.SPEC-003 until FEAT-15.SPEC-001's own final-step completion signal fires | FEAT-15.SPEC-001, FEAT-15.SPEC-003 |
| No-op (already active, still satisfied) | Link is already active and re-evaluation confirms the rule remains satisfied | None | No visible change | -- |
| No-op (already active, a condition regresses) | Link is already active and a later trigger reports a required condition now false | None -- this automation never deactivates a live link | No visible change from this automation; any resulting Pro-facing banner or notification is owned by FEAT-27/FEAT-18's own attention mechanisms, not this spec | FEAT-27, FEAT-18 (out of this spec's scope) |
| Automation failure | Processing error during evaluation | No partial activation state is ever persisted -- the link is either fully active or not active, never partially | Talia sees the wizard shell or FEAT-15.SPEC-003 in its previous known state; a retry occurs on the next trigger (e.g., the next step completion or a manual refresh of FEAT-15.SPEC-003) | FEAT-15.SPEC-001, FEAT-15.SPEC-003 |

## Data Model

**Reads:** Pro Account setup-progress state (FEAT-15.SPEC-004), Payout Account status (FEAT-28), Subscription status (FEAT-18) -- all read-only, per the dependency map's Referenced Entities table for FEAT-15.
**Creates:** None.
**Updates:** The booking link's active/not-active state and activation timestamp on the Pro Account -- owned exclusively by this automation; no other spec activates the link.
**Deletes:** None.

## Business Rules

- FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) determines which steps count toward the seven required conditions on every evaluation; the optional calendar step never counts, whether it is complete, skipped, or incomplete.
- FEAT-15.SPEC-007 (XBR-26) is the sole authority for which conditions gate go-live; this automation never adds, removes, or reinterprets a condition on its own.
- Activation is one-directional: once a link is active, this automation never deactivates it. A regression in an underlying condition (payout action-required, subscription lapse) is handled entirely by the owning feature's own pause/lapse mechanism (FEAT-27, FEAT-18), consistent with XBR-11's "setup changes never silently cancel a confirmed booking" and XBR-14's pause behavior, which apply to an already-live account, not to this feature's one-time go-live transition.
- Evaluation is idempotent -- re-running it against an unchanged state never re-emits onboarding_completed or re-activates an already-active link.
- Re-evaluation is triggered by every relevant upstream change, not on a fixed schedule, so activation happens "the moment" readiness is reached, per the Key Capability's stated behavior, never on a delay.

## Edge Cases

- **Two required conditions become true at nearly the same moment (for example, Talia's subscription payment succeeds at the same instant her payout account finishes verification)** -- Concurrent trigger firing: each trigger independently re-reads the full current state of all seven conditions (step 1 of Processing Logic), so whichever trigger's evaluation runs second sees both conditions already true and activates the link; the first to run either activates it (if it also sees both true) or finds one still pending and takes no action. Exactly one activation ever occurs, never two, because step 3's idempotency check prevents a duplicate activation regardless of which trigger's evaluation "wins."
- **Trigger fires while a previous evaluation run is in flight** -- Because each evaluation run reads a fresh snapshot of all seven conditions independently rather than relying on a value carried over from a prior run, an overlapping second run produces the same correct outcome as if it had waited; both runs converge on the same activation decision, and step 3's idempotency guard prevents any duplicate activation or event.
- **Talia's payout account regresses from Active to Verification Pending again after the link is already live (an unusual processor-side event)** -- Per the Non-Goals and Processing Logic step 6, this automation takes no deactivation action; the link remains live, consistent with "setup changes never silently cancel" behavior owning to other features once an account is live.
- **All seven conditions are satisfied except the deposit rule, because Talia added a service with a name and duration but has not yet set its deposit rule** -- Not possible as a standalone gap: per FEAT-01's Service entity, a deposit rule is a required field of every Service record at creation, so a Service cannot exist without one; "at least one Service exists" and "a deposit rule exists" are therefore always satisfied together, never independently.
- **Talia skips the calendar step and every other required condition is satisfied** -- The link activates normally: the calendar step is explicitly excluded from the seven required conditions per XBR-26, so a skip never blocks or delays activation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | Triggered by (inbound) | Every progress change re-triggers this automation's evaluation |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Enforcing side of its Enforced-By entry: on every readiness evaluation this automation applies FEAT-15.SPEC-006 to decide which steps count toward the seven required conditions (the optional calendar step is excluded) |
| FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) | References (inbound) | Supplies the exact condition set this automation evaluates |
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Affects (outbound) | Readiness confirmation is what causes the shell to hand off to FEAT-15.SPEC-003 |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Affects (outbound) | This automation's activation state is exactly what that screen displays as Live or Waiting |
| FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | Affects (outbound) | Link activation is the trigger event for the welcome confirmation |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggered by (inbound) | Payout status changes re-trigger evaluation |
| FEAT-18 (subscription-billing automation, FEAT-18.SPEC-006) | Triggered by (inbound) | Subscription status changes re-trigger evaluation |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | Activation is what makes FEAT-05's public page reachable at Talia's booking link |

## Analytics and Success Signals

- **onboarding_completed** (elapsed time since onboarding_started, count of steps completed vs. skipped) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **golive_evaluation_run** (outcome: activated / still_pending / no_op; gap conditions if still pending) -- N/A -- no Stage 2 metric directly measures individual evaluation runs; retained internally so a stalled Pro's specific blocking condition is observable to Support (via FEAT-19) rather than invisible.

## Acceptance Criteria

**FEAT-15.SPEC-005-AC-01:** Given Talia has completed every required step except her subscription, when she completes the subscription step, then this automation re-evaluates and, finding all seven conditions satisfied, activates her booking link immediately.

**FEAT-15.SPEC-005-AC-02:** Given Talia's booking link has just activated, when the activation completes, then onboarding_completed is emitted and FEAT-15.SPEC-008's welcome confirmation becomes eligible to send.

**FEAT-15.SPEC-005-AC-03:** Given Talia has completed every required step except her payout account is still Verification Pending, when this automation evaluates readiness, then it does not activate the link, and FEAT-15.SPEC-003 shows the Waiting state.

**FEAT-15.SPEC-005-AC-04:** Given Talia's payout account later becomes Active, when FEAT-28.SPEC-003 reports the status change, then this automation re-evaluates and activates the link without requiring Talia to take any wizard action.

**FEAT-15.SPEC-005-AC-05:** Given Talia's link is already active and a later trigger fires reporting no change in any condition, when this automation re-evaluates, then no duplicate onboarding_completed event is emitted and no re-activation occurs.

**FEAT-15.SPEC-005-AC-06:** Given Talia's payout account regresses from Active to Action Required after her link is already live, when this automation is triggered by that status change, then the link remains live and this automation takes no deactivation action.

**FEAT-15.SPEC-005-AC-07:** Given Talia skips the calendar-connection step and every other required condition is satisfied, when this automation evaluates readiness, then the link activates normally, since the calendar step is excluded from the seven required conditions.

**FEAT-15.SPEC-005-AC-08:** Given two required conditions (subscription and payout) become true at nearly the same instant, when both triggers fire, then exactly one activation occurs and exactly one onboarding_completed event is emitted.

**FEAT-15.SPEC-005-AC-09:** Given a second evaluation run starts while a first run for the same Pro Account is still in flight, when both complete, then they converge on the same activation decision with no duplicate activation.

**FEAT-15.SPEC-005-AC-10:** Given Talia's account has a Service record, when this automation checks the deposit-rule condition, then it is always found satisfied together with the service-exists condition, since a Service cannot exist without a deposit rule.

**FEAT-15.SPEC-005-AC-11:** Given a processing error occurs during evaluation, when the error is detected, then no partial activation state is persisted, and the next relevant trigger re-evaluates from a clean, fully-read state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Setup Step Order & Optional-Step Rules

## Overview

**Name:** Setup Step Order & Optional-Step Rules
**ID:** FEAT-15.SPEC-006
**Type:** Logic/Rule
**Purpose:** Defines the fixed step sequence, which single step (calendar connection) is optional and resumable from settings later, and how a skipped step is represented in progress.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard
**Governed Entity:** Pro Account's setup-progress state (the per-step completion sub-structure of the Pro Account entity)

## Scope and Non-Goals

**In Scope:**
- The fixed order of the 8 setup steps
- Which step is optional (calendar connection) and how a skip is recorded and later resolved
- Authorization for who may view or advance setup progress
- Defaults and derivations for the resume-point computation
- The cross-step "offer a default, let the Pro accept or change it" pattern this feature establishes for every hand-off step

**Non-Goals:**
- Determining Go-Live readiness -- owned by FEAT-15.SPEC-007 (Go-Live Prerequisite Rule, XBR-26); this spec governs sequencing and optionality, not the separate question of when the link may activate
- Writing or reading the progress record -- owned by FEAT-15.SPEC-004 (Setup Progress Tracking & Resume); this spec defines the rules that automation enforces, not the mechanics of persistence
- Field-level validation within any individual step's own form (service fields, hours, payout details) -- each step's owning feature governs its own field rules (e.g., FEAT-01.SPEC-004 for service fields); this spec governs only the sequence and optionality of steps as a whole
- Multi-staff or role-based setup paths -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so exactly one linear sequence exists for one Pro

## Governed Entity

**Entity:** Pro Account (setup-progress state)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| step_account_signin | enum (complete) | Always complete once the wizard is entered, since sign-in (FEAT-29) must succeed first |
| step_profile | enum (incomplete, complete) | Whether display name and studio location have been captured (FEAT-27) |
| step_service | enum (incomplete, complete) | Whether at least one Service exists (FEAT-01) |
| step_hours | enum (incomplete, complete) | Whether working hours have been set (FEAT-02) |
| step_cancellation_policy | enum (incomplete, complete) | Whether Cancellation Policy version 1 exists (FEAT-15.SPEC-002) |
| step_payout | enum (incomplete, complete) | Whether a Payout Account exists in at least Verification Pending status (FEAT-28) |
| step_calendar | enum (incomplete, complete-as-connected, complete-as-skipped) | The one step this spec marks optional (FEAT-04) |
| step_subscription | enum (incomplete, complete) | Whether the Subscription is at least started (FEAT-18) |
| resume_point | derived | The first step in fixed order not yet complete (complete-as-skipped counts as complete) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-15.SPEC-001 | Setup Wizard Shell, Step Navigation & Guidance | On every screen load, to determine which steps are tappable, dimmed, or current, and to display resume_point |
| FEAT-15.SPEC-004 | Setup Progress Tracking & Resume | On every step-completion signal, to determine which step advances and how a skip is recorded |
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | On every readiness evaluation, to determine which steps count toward the seven required conditions (excluding the optional calendar step) |
| FEAT-04.SPEC-001 | Calendar Connection Setup | When Talia is offered the "skip and connect later" choice |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| step_account_signin | No validation beyond data type -- always set to complete on wizard entry | Always | -- | -- | -- |
| step_profile | Must transition only from incomplete to complete; never reverts | Always | On completion signal | -- | -- |
| step_service | Must transition only from incomplete to complete; never reverts once at least one Service has ever existed | Always | On completion signal | -- | -- |
| step_hours | Must transition only from incomplete to complete; never reverts | Always | On completion signal | -- | -- |
| step_cancellation_policy | Must transition only from incomplete to complete; never reverts | Always | On completion signal | -- | -- |
| step_payout | Must transition only from incomplete to complete; never reverts once a Payout Account has ever reached at least Verification Pending | Always | On completion signal | -- | -- |
| step_calendar | May transition from incomplete to complete-as-skipped or complete-as-connected, and from complete-as-skipped to complete-as-connected; never from complete-as-connected back to incomplete or skipped | Always | On completion or skip signal | -- | -- |
| step_subscription | Must transition only from incomplete to complete; never reverts within this feature's scope (a later cancellation is FEAT-18's own concern, not a reversion of this setup step) | Always | On completion signal | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Fixed sequence display | step_account_signin, step_profile, step_service, step_hours, step_cancellation_policy, step_payout, step_calendar, step_subscription | The wizard shell (FEAT-15.SPEC-001) shows steps in this exact order; an upcoming step (one after the first incomplete step) is never made tappable regardless of any individual field's own state | N/A -- this is a display/navigation constraint, not a user-facing validation error |
| Calendar step is excluded from advancement blocking | step_calendar, resume_point | resume_point calculation treats step_calendar as satisfied whenever it is complete-as-skipped OR complete-as-connected, so an incomplete calendar step never becomes the resume_point once explicitly skipped | N/A |
| Resume point derivation | All eight step_* fields | resume_point = the first field in the fixed order (per Field Validation Rules) whose value is still "incomplete"; if all are satisfied, resume_point is null and setup is complete | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own setup progress | The Pro (Talia) | Always, for her own account only | -- |
| Advance a step (via its owning screen) | The Pro (Talia) | Only the step at resume_point may be advanced through the wizard shell's Continue action; a completed step may be re-opened for review through the header list without being able to un-complete it | Attempting to advance a step ahead of resume_point is not exposed as a possible action -- the step-list item for any step after resume_point is rendered non-interactive (dimmed) rather than producing an error |
| Skip the calendar step | The Pro (Talia) | Only the calendar step, only while it is incomplete | No other step exposes a skip choice; the choice control appears only on FEAT-04.SPEC-001's own screen for this one step |
| View setup progress | Platform Operator (Support) | Always, for any Pro Account during an active help request, read-only | -- |
| Advance, skip, or otherwise modify a step on the Pro's behalf | Platform Operator (Support) | Never | No action control of any kind is shown to Support in the progress view surfaced by FEAT-19; per SC-05 and XBR-24, a stuck Pro must resolve their own step |
| View or act on any Pro Account's setup progress | The Client (Riley) | Never | No client-facing surface of any kind exposes setup progress; the concept does not exist in any client-reachable screen |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| step_account_signin | Defaults to complete | On Pro Account creation (first wizard entry) | No -- it is a fact of having reached the wizard at all |
| All other step_* fields | Default to incomplete | On Pro Account creation | No -- they become complete only through their own owning screen's flow |
| resume_point | Derived: first incomplete step in fixed order | Recomputed on every progress change and every wizard shell load | No -- it is entirely derived, never directly settable |
| step_calendar | Defaults to incomplete; the Pro chooses complete-as-connected (by connecting) or complete-as-skipped (by explicit skip) | On reaching the calendar step in the fixed order | Yes -- the Pro may later convert complete-as-skipped to complete-as-connected by connecting from settings (FEAT-04), at any time |

## Business Rules

- The fixed step order is: (1) account & sign-in, (2) profile & studio location, (3) at least one service (including its deposit rule), (4) working hours, (5) cancellation policy, (6) payout account, (7) calendar connection (optional), (8) subscription -- matching the Feature Breakdown Brief's Key Capabilities and the dependency map's Navigation Connections table exactly.
- The calendar-connection step is the only step in the sequence that can be marked complete without its underlying capability having actually been used, per the Brief's Side-Effect Inventory ("Pro reaches the calendar-connection step -> offer the skip choice -> mark the step complete-as-skipped without blocking progress").
- A skipped calendar step never counts against Go-Live readiness (XBR-26; enforced by FEAT-15.SPEC-007) -- this spec only defines that the step is marked satisfied either way; FEAT-15.SPEC-007 is the authority on which steps that satisfaction actually gates.
- The "offer a default, let the Pro accept or change it" pattern established concretely by FEAT-15.SPEC-002 (the cancellation window) is the general pattern every hand-off step's owning feature should also apply to its own defaults (e.g., a suggested buffer time in FEAT-02, a suggested deposit percentage in FEAT-01), so the wizard reads as one consistent experience end to end, per the Brief's Shared Context.
- No step may be advanced out of order through the wizard shell; the shell's own enforcement (FEAT-15.SPEC-001) makes upcoming steps non-interactive rather than presenting them as actionable and then rejecting the action.

## Edge Cases

- **Talia reaches the calendar step, skips it, and never returns to it** -- step_calendar remains complete-as-skipped indefinitely; this has no expiry and never blocks resume_point from advancing past it, since it is treated as satisfied.
- **Talia completes the subscription step (step 8) before every earlier step, by navigating a deep link outside the wizard** -- Not possible through the wizard shell's own navigation (steps are gated in order), but if an owning feature's screen is reached directly and its data is saved (for example, Talia subscribes via a settings-area link before finishing profile setup), FEAT-15.SPEC-004 still records step_subscription as complete out of order; resume_point calculation is unaffected, since it always evaluates the fixed order from the start regardless of which steps happen to already be complete.
- **Two of the eight step fields are updated by two completion signals arriving in the same instant** -- Each field is independent (per Field Validation Rules, no field's transition depends on another field's simultaneous value), so both updates apply cleanly with no conflict.
- **step_calendar is at exactly the boundary between complete-as-skipped and complete-as-connected (Talia connects immediately after skipping, before advancing past the step)** -- The transition from complete-as-skipped to complete-as-connected is explicitly allowed (see Field Validation Rules); the reverse (complete-as-connected back to skipped or incomplete) is never allowed, since disconnecting a calendar afterward is FEAT-04's own disconnect action, not a reversion of this setup step.
- **All eight fields become complete at once (a rare but possible burst, e.g., a batch of upstream statuses resolving together)** -- resume_point correctly computes to null; this spec makes no further claim about what happens next, since Go-Live activation itself is FEAT-15.SPEC-007's and FEAT-15.SPEC-005's authority.
- **Support attempts to view progress for a Pro Account with no help request context** -- Not a rule this spec governs directly; FEAT-19 owns the authorization gate on when Support may open any Pro's account at all. This spec's Authorization Rules apply once Support is validly viewing an account per FEAT-19's own rules.

## Acceptance Criteria

**FEAT-15.SPEC-006-AC-01:** Given Talia has just entered the wizard for the first time, when the progress record is created, then step_account_signin is already complete and all other step_* fields are incomplete.

**FEAT-15.SPEC-006-AC-02:** Given Talia has completed steps 1 through 4, when resume_point is computed, then it evaluates to step 5 (cancellation policy).

**FEAT-15.SPEC-006-AC-03:** Given Talia is at the calendar step, when she chooses "skip and connect later," then step_calendar transitions to complete-as-skipped and resume_point advances to step 8 (subscription).

**FEAT-15.SPEC-006-AC-04:** Given Talia skipped the calendar step, when she later connects a calendar from settings, then step_calendar transitions from complete-as-skipped to complete-as-connected.

**FEAT-15.SPEC-006-AC-05:** Given Talia has already connected her calendar (complete-as-connected), when any process attempts to mark it skipped, then the transition is rejected -- a connected calendar step can never revert to skipped or incomplete.

**FEAT-15.SPEC-006-AC-06:** Given Talia (the Pro) is on the wizard shell, when she looks at a step after resume_point, then it is rendered non-interactive (dimmed), and no action is available to advance it out of order.

**FEAT-15.SPEC-006-AC-07:** Given Talia (the Pro) taps a completed step's header entry, when she views it, then she can review it but no control exists to un-complete it.

**FEAT-15.SPEC-006-AC-08:** Given Platform Operator (Support) opens Talia's account during a help request, when they view her setup progress, then they see the same fields read-only with no action control shown.

**FEAT-15.SPEC-006-AC-09:** Given Platform Operator (Support) is viewing a Pro's setup progress, when they look for any way to advance, skip, or modify a step, then no such control exists anywhere in that view.

**FEAT-15.SPEC-006-AC-10:** Given a client (Riley) attempts to reach any setup-progress surface, when the attempt is made, then no such surface is reachable by a client under any circumstance.

**FEAT-15.SPEC-006-AC-11:** Given every one of the eight step_* fields is satisfied at once, when resume_point is recomputed, then it evaluates to null.

**FEAT-15.SPEC-006-AC-12:** Given Talia's step_service field is already complete, when a duplicate completion signal for the services step arrives, then the field remains complete and does not toggle or reset.

**FEAT-15.SPEC-006-AC-13:** Given the calendar step is skipped, when FEAT-15.SPEC-007 evaluates Go-Live readiness, then the skipped calendar step never counts against it.

**FEAT-15.SPEC-006-AC-14:** Given Talia subscribes via a settings-area link before completing her profile step, when FEAT-15.SPEC-004 records the subscription completion, then resume_point still correctly reflects the profile step as the next incomplete step in fixed order.

**FEAT-15.SPEC-006-AC-15:** Given the wizard shell displays the eight steps, when Talia views the header list, then the order shown exactly matches: account & sign-in, profile, service, hours, cancellation policy, payout, calendar (optional), subscription.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Go-Live Prerequisite Rule (XBR-26 Authority)

## Overview

**Name:** Go-Live Prerequisite Rule (XBR-26 Authority)
**ID:** FEAT-15.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines and owns the exact set of steps that must be complete before the booking link can go live, per XBR-26; consumed by FEAT-15.SPEC-005 and referenced by other features that gate on go-live status.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard
**Governed Entity:** Pro Account's Go-Live readiness state (the derived "may this booking link be active" flag)

## Scope and Non-Goals

**In Scope:**
- The exact, complete list of conditions required before a booking link may go live, per XBR-26
- The single condition (calendar connection) that is explicitly excluded from the requirement
- Authorization for who may act on go-live status and who may only observe it
- The derivation logic that produces the readiness flag itself
- Being the single authoritative source other features (FEAT-05, FEAT-07) reference rather than re-deriving the condition list

**Non-Goals:**
- Tracking whether each individual condition is currently true -- owned by FEAT-15.SPEC-004 (setup-progress state) and by each condition's owning feature (FEAT-01, FEAT-02, FEAT-09, FEAT-28, FEAT-18, FEAT-27, FEAT-29); this spec only defines which conditions matter and how they combine
- Activating the link once the rule is satisfied -- owned by FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation), which consumes this rule's evaluation
- Defining the step sequence or which step is skippable in the wizard's own navigation -- owned by FEAT-15.SPEC-006; this spec defines the go-live gate, not the wizard's walking order (though the two are deliberately aligned)
- Allowing a partial or Pro-overridable go-live state -- excluded per the feature's own explicit non-goal and Validation & Limits field: the seven required conditions (the eight wizard steps less the optional calendar step) are a hard gate with no partial or Pro-overridable go-live state

## Governed Entity

**Entity:** Pro Account's Go-Live readiness state
**Source:** Feature Dependency Map (XBR-26)

| Field | Data Type | Description |
|-------|-----------|-------------|
| condition_signin | boolean | Sign-in is established (FEAT-29) |
| condition_display_name_location | boolean | Display name and studio location are set (FEAT-27) |
| condition_service | boolean | At least one Service exists (FEAT-01), which by definition includes a deposit rule |
| condition_hours | boolean | Working hours are set (FEAT-02) |
| condition_cancellation_policy | boolean | Cancellation Policy version 1 exists (FEAT-15.SPEC-002) |
| condition_payout_active | boolean | Payout Account status is Active (FEAT-28) |
| condition_subscription_active | boolean | Subscription status is Active (FEAT-18) |
| condition_calendar (excluded) | boolean | Calendar connection status (FEAT-04) -- tracked for display purposes only; never included in the readiness computation |
| is_ready | derived (boolean) | True only when all seven required conditions above (excluding calendar) are true |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | Evaluates is_ready on every relevant trigger and activates the link when it becomes true |
| FEAT-15.SPEC-003 | Go-Live Preview & Booking Link Hand-Over | Displays Live vs. Waiting based on is_ready and, specifically, condition_payout_active when that is the sole remaining gap |
| FEAT-05 | Public Booking Page & Booking Flow | Reads whether the link is live (the outcome of this rule via FEAT-15.SPEC-005) rather than re-deriving the condition list, per XBR-26's authority note |
| FEAT-07 | Deposit Payment at Booking | Reads the same live/not-live outcome (via FEAT-28's payout-active check, which is itself one of this rule's seven conditions) to determine whether a deposit may be taken, per XBR-06 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| condition_signin | Must be true; sourced from FEAT-29 | Always | On every readiness evaluation | N/A -- this is an internal derived condition, not a user-facing input field, so it carries no error message of its own | Yes |
| condition_display_name_location | Must be true; sourced from FEAT-27 | Always | On every readiness evaluation | N/A | Yes |
| condition_service | Must be true; sourced from FEAT-01 (existence of at least one Service, which requires a deposit rule by that entity's own definition) | Always | On every readiness evaluation | N/A | Yes |
| condition_hours | Must be true; sourced from FEAT-02 | Always | On every readiness evaluation | N/A | Yes |
| condition_cancellation_policy | Must be true; sourced from FEAT-15.SPEC-002 | Always | On every readiness evaluation | N/A | Yes |
| condition_payout_active | Must be true; sourced from FEAT-28 (status = Active, not merely Verification Pending) | Always | On every readiness evaluation | N/A | Yes |
| condition_subscription_active | Must be true; sourced from FEAT-18 (status = Active) | Always | On every readiness evaluation | N/A | Yes |
| condition_calendar | No validation applied -- explicitly excluded from the readiness computation regardless of its value | Always excluded | Never checked for readiness purposes (tracked separately for display only, per FEAT-15.SPEC-006) | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Readiness is a strict conjunction | condition_signin, condition_display_name_location, condition_service, condition_hours, condition_cancellation_policy, condition_payout_active, condition_subscription_active | is_ready = true only when every one of these seven is true; any single false condition makes is_ready false -- there is no weighted, partial, or majority-based readiness | N/A -- surfaced to the Pro as the Waiting state (FEAT-15.SPEC-003), not as a field-level error |
| Calendar exclusion | condition_calendar, is_ready | condition_calendar's value never participates in the is_ready computation in any way, regardless of whether it is true, false, or undefined | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own Go-Live readiness state | The Pro (Talia) | Always, for her own account | -- |
| Force or override Go-Live activation ahead of readiness | The Pro (Talia) | Never | No control of any kind exists anywhere in the product to activate a link before all seven conditions are true; the feature's own Validation & Limits field states this is a hard gate with no override |
| View Go-Live readiness state | Platform Operator (Support) | Always, for any Pro Account during an active help request, read-only | -- |
| Force or override Go-Live activation on a Pro's behalf | Platform Operator (Support) | Never | No action control is shown to Support in any progress or readiness view; per SC-05, Support cannot act on a Pro's account |
| View or act on Go-Live readiness for any account | The Client (Riley) | Never | No client-facing surface exposes readiness state directly; a client only ever sees the resulting live-or-not-reachable booking page |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| condition_* fields | Default to false (not yet satisfied) on Pro Account creation | On Pro Account creation | No -- each becomes true only through its own owning feature's completion |
| is_ready | Derived: logical AND of all seven required condition_* fields | Recomputed on every trigger listed in FEAT-15.SPEC-005's Trigger Definition | No -- entirely derived, never directly settable by any role |

## Business Rules

- XBR-26 (the cross-feature business rule this spec is the authority for): "The booking link goes live only when sign-in, display name and studio location, one service, working hours, deposit rule, cancellation policy, an active payout account and an active subscription are all in place; calendar connection is the only optional step." This spec's seven required conditions are the literal enumeration of that rule (deposit rule is folded into condition_service, since a Service cannot exist without one, per FEAT-01's own entity definition).
- FEAT-15 is the sole owner of this rule; other features that gate behavior on go-live status (FEAT-05, FEAT-07 via XBR-06) reference this rule's outcome rather than re-deriving or duplicating the condition list, per the dependency map's Authority column for XBR-26.
- There is no partial, weighted, or Pro-overridable readiness state -- every one of the seven conditions is mandatory and none can be waived, per the feature's own Non-Goals.
- The calendar connection condition is permanently excluded from this computation; it can never become a blocking condition under any configuration, present or future within this feature's scope, since XBR-26 names it explicitly as the one optional step.

## Edge Cases

- **All seven required conditions are true except condition_payout_active, which is Verification Pending rather than outright false** -- Treated identically to any other unmet condition: is_ready is false. This specific gap is surfaced distinctly by FEAT-15.SPEC-003 (the "finish verifying" Waiting state) precisely because it is common enough near the end of setup to warrant its own message, but the underlying rule treats it the same as any other unmet condition.
- **condition_subscription_active later becomes false after the link has already gone live (subscription lapses)** -- Out of scope for this rule's own gating logic: this spec governs the one-time transition to live; an already-live link's subsequent pause on subscription lapse is XBR-14's concern, owned by FEAT-27 and FEAT-18, not a re-evaluation of this go-live gate.
- **A new required condition is proposed in a future version (hypothetical: e.g., requiring an intro photo)** -- Not evaluated by this rule as written; any change to the seven-condition set is a change to XBR-26 itself, made explicitly at that authority, never inferred by another feature adding conditions unilaterally.
- **condition_calendar is true (connected) while every other condition is also true** -- No different outcome than if condition_calendar were false or unset: is_ready is already true from the seven required conditions alone; the calendar's own state adds nothing to and subtracts nothing from readiness.
- **Two required conditions are read at slightly different moments within one evaluation (e.g., a fast payout-status read and a slower subscription-status read)** -- The evaluation in FEAT-15.SPEC-005 reads a consistent snapshot of all seven conditions together before computing is_ready (per that spec's Processing Logic), so this rule itself never operates on a torn read across two different points in time.
- **A downstream feature (FEAT-07) needs to know if a deposit may be taken, which depends on payout status alone (XBR-06), not the full go-live rule** -- FEAT-07 reads condition_payout_active (via FEAT-28) directly for its own XBR-06 check; it does not need is_ready as a whole, since a deposit's precondition and a link's go-live precondition are related but distinct questions this spec keeps separately readable.

## Acceptance Criteria

**FEAT-15.SPEC-007-AC-01:** Given all seven required conditions are true, when is_ready is computed, then it evaluates to true.

**FEAT-15.SPEC-007-AC-02:** Given exactly one required condition (payout active) is false and the other six are true, when is_ready is computed, then it evaluates to false.

**FEAT-15.SPEC-007-AC-03:** Given condition_calendar is false (never connected, never skipped -- a theoretical unset state), when is_ready is computed with all seven required conditions true, then it still evaluates to true, since condition_calendar never participates in the computation.

**FEAT-15.SPEC-007-AC-04:** Given condition_calendar is true (connected) and all seven required conditions are also true, when is_ready is computed, then the outcome is the same as if condition_calendar were false -- true either way.

**FEAT-15.SPEC-007-AC-05:** Given Talia's account has a Service record, when condition_service is evaluated, then it is true, and the deposit-rule requirement is satisfied by the same check, since FEAT-01's Service entity cannot exist without one.

**FEAT-15.SPEC-007-AC-06:** Given Talia (the Pro) looks for any way to force her link live before all seven conditions are met, when she searches the product, then no such control exists anywhere.

**FEAT-15.SPEC-007-AC-07:** Given Platform Operator (Support) views a Pro's readiness state during a help request, when they look for an override action, then none is available to them.

**FEAT-15.SPEC-007-AC-08:** Given a client (Riley) has no reachable surface for readiness state, when any client-facing screen is inspected, then readiness state is never exposed directly, only its outcome (the booking page being reachable or not).

**FEAT-15.SPEC-007-AC-09:** Given Talia's subscription lapses after her link is already live, when this rule is consulted, then it is not re-evaluated to deactivate the link -- that behavior is owned by XBR-14 via FEAT-27 and FEAT-18, outside this spec's one-time gating scope.

**FEAT-15.SPEC-007-AC-10:** Given FEAT-05 needs to know whether a Pro's link is live, when it checks, then it reads the outcome of this rule (via FEAT-15.SPEC-005's activation state) rather than re-deriving any condition itself.

**FEAT-15.SPEC-007-AC-11:** Given FEAT-07 needs to know whether a deposit may be taken, when it checks per XBR-06, then it reads condition_payout_active directly rather than requiring the full seven-condition is_ready to be true.

**FEAT-15.SPEC-007-AC-12:** Given all seven required conditions are true except condition_payout_active, which shows Verification Pending, when FEAT-15.SPEC-003 renders, then it shows the "finish verifying" Waiting state, consistent with this rule treating that gap the same as any other unmet condition.

**FEAT-15.SPEC-007-AC-13:** Given an evaluation reads payout status and subscription status at slightly different instants within one run, when is_ready is computed, then the computation uses a single consistent snapshot of all seven conditions rather than a mix of stale and fresh values.

**FEAT-15.SPEC-007-AC-14:** Given all seven required conditions are true and condition_calendar has just transitioned from incomplete to complete-as-skipped, when is_ready is recomputed, then the transition itself has no effect on the already-true is_ready outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Notification Spec: Onboarding Welcome Confirmation

## Overview

**Name:** Onboarding Welcome Confirmation
**ID:** FEAT-15.SPEC-008
**Type:** Notification
**Purpose:** Tells Talia, the moment her booking link goes live, that setup is done and her link is ready to share -- a one-time confirmation, not a recurring nag.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- The one-time welcome confirmation sent the instant the booking link activates
- Its content on both channels the product uses for Pro notifications
- Preference, delivery, retry, and expiry behavior for this one confirmation

**Non-Goals:**
- Any recurring onboarding reminder or nag if a Pro stalls mid-setup -- product-features.md's Communications field for FEAT-15 names exactly one message (the welcome confirmation); the Brief's Side-Effect Inventory defines no "nudge a stalled Pro" communication, so none is built
- Sending this confirmation to the Client -- the audience is exclusively the Pro, per the Brief's own Roles Touched column for this spec ("The Pro")
- The mechanics of text and email delivery themselves -- owned by FEAT-08.SPEC-012 (transactional text) and FEAT-08.SPEC-013 (transactional email), which this spec's content and delivery rules are handed to for actual sending
- Confirming any individual setup step's completion (e.g., "your service was added") -- each step's owning screen (FEAT-01, FEAT-02, FEAT-28, etc.) gives its own in-flow save confirmation; this spec covers only the single, final go-live moment

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, the next time Talia opens the product after go-live if she is not actively viewing FEAT-15.SPEC-003 at that instant; shown immediately if she is | Talia is mid-session in the wizard at the moment this fires (she just completed her final step), so an in-app surface reaches her with zero delay in the common case |
| Text | When Talia has an active texting consent context for her own Pro notifications, per her notification preferences (FEAT-27) -- texting is the default for Pro notifications in this product, matching the founder's own phone-first behavior described in BRIEF.md | Talia does nearly all of her Chairtime use on her phone in short bursts between clients (user-persona.md, Behavioral Context); a text reaches her even if she has already closed the app after finishing setup |
| Email | When Talia's notification preferences (FEAT-27) select email instead of, or in addition to, text for Pro notifications, or as the fallback when a text send fails (per FEAT-08.SPEC-009) | Matches the product-wide email-fallback pattern already established for every other Pro and client notification |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking link activates | FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Fires exactly once, the first time is_ready (FEAT-15.SPEC-007) becomes true for a given Pro Account and the link is activated | Pro's display name, booking_link_name, activation timestamp |

## Audience and Preferences

**Recipients:** The Pro (Talia) only -- traced to the Access Matrix's "Profile & Account Settings" and "Booking & Payment" columns, both Full for the Pro. This is a Pro-facing account-lifecycle notification; no other role in the Access Matrix (Client, Platform Operator Support) is entitled to it or to the data it carries (the Pro's own booking link).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Pro notification channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text | Pro Profile & Booking Page Settings (FEAT-27, notification preferences) |

This confirmation follows the same Pro-notification channel preference every other Pro notification in the product uses (product-features.md, FEAT-08's Key Capabilities: "Notify the Pro of new bookings... and anything needing attention... per the Pro's preferences in FEAT-27"); it introduces no preference control of its own, since it is a one-time event with no independent on/off switch a Pro would need -- it fires exactly once, ever, per account.

**Quiet Hours:** N/A -- this product's quiet-hours window (roughly 8am-9pm) is defined specifically for automatic client-facing appointment reminders (BRIEF.md, XBR-16), which are a recurring, potentially-many-per-day communication about someone else's appointment. This is a single, one-time, self-initiated confirmation that fires as the direct result of an action Talia herself just took (completing her final setup step); holding it until a quiet-hours window would delay Talia's own recognition of her own achievement for no protective purpose, so it is exempt on every channel it uses.

## Content Definition

**In-app:**
- **Title:** You're live, {pro_display_name}!
- **Body:** Your booking link is ready to share: {booking_link}
- **CTA:** View my link -- deep-links to FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) for this Pro's own account

**Text:**
- **Body:** Chairtime: You're live, {pro_display_name}! Your booking link is ready: {booking_link}. Share it anywhere -- your Instagram bio is a great place to start.
- **CTA:** N/A -- the link itself is the action; tapping it opens the live booking page directly (no separate in-product deep link is needed on this channel)

**Email:**
- **Subject:** Your Chairtime booking link is live
- **Body:**
  Hi {pro_display_name},

  You did it -- your Chairtime setup is complete and your booking link is ready to share:

  {booking_link}

  Add it to your Instagram bio, or send it straight to a client, and they'll be able to pick a service, choose a genuinely free time, and pay their deposit in under a minute.
- **CTA (button):** View my link -- deep-links to FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) for this Pro's own account

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | "there" (greeting renders as "Hi there," / "You're live, there!") -- display_name is required before this notification can ever fire, since profile completion is one of the seven required Go-Live conditions (FEAT-15.SPEC-007), so this fallback is a defensive default that is never expected to render |
| {booking_link} | Pro Account -- booking_link_name, rendered as the full shareable URL | chairtime.app/talia-lashes | Never empty -- link activation (the trigger for this notification) cannot occur without a booking_link_name already existing on the Pro Account (FEAT-27, required before go-live) |

## Delivery Rules

**Batching:** None -- this notification is inherently singular (one Pro Account can only go live once, ever), so no batching window or key applies.
**Deduplication:** At most one Onboarding Welcome Confirmation is ever sent per Pro Account. FEAT-15.SPEC-005's activation is itself a one-directional, idempotent transition (its own Business Rules: "activation is one-directional... re-running it against an unchanged state never re-emits onboarding_completed"), so this notification's trigger event can never fire twice for the same account, and no separate deduplication key is needed beyond that guarantee.
**Retry on failure:** Text delivery failure is retried once, then falls back to email, consistent with the product-wide message-delivery pattern (FEAT-08.SPEC-009); if both fail, the in-app notification stands as the delivery of record and no alarming failure message is shown to Talia -- a delayed confirmation is a minor cosmetic gap, never a business-critical loss, since the link itself is already live and visible on FEAT-15.SPEC-003 regardless of whether this notification ever reaches her by text or email.
**Expiry:** The in-app notification never expires undelivered -- it is shown the next time Talia opens the product, however long that takes, since her account only ever goes live once and the moment is worth surfacing whenever she next returns. The text/email attempt itself does not expire in the sense of being withdrawn; per the retry rule above, it either succeeds (directly or via fallback) or silently stands aside for the in-app channel, which never expires.

## Edge Cases

- **Talia's Pro Account is closed (FEAT-29) between activation and delivery** -- Not possible in practice: activation and this notification's dispatch happen within the same processing step (per FEAT-15.SPEC-005's Processing Logic, "activate the link... and emit onboarding_completed"), and account closure requires a Pro Account that has been active for some time first; there is no meaningful window for closure to occur between the two.
- **Talia's notification preferences (FEAT-27) are changed between trigger and delivery (for example, she turns off text mid-send)** -- The preference in effect at delivery time governs, consistent with the product-wide rule that preferences are evaluated at delivery time rather than trigger time; if text is turned off before the send completes, the send is not attempted on that channel and the in-app notification (which has no off switch, per Preference Controls) still delivers.
- **Both text and email delivery fail** -- Per the Retry on failure rule, the in-app notification stands as the delivery of record; Talia is never shown an alarming failure message, since her link is already live and visible regardless of this notification's delivery outcome on any other channel.
- **Talia is actively viewing FEAT-15.SPEC-003 at the exact moment activation occurs** -- The in-app notification and the screen's own Live-state transition happen from the same activation event; Talia experiences this as the screen itself updating to show her live link, with the in-app notification available from wherever the product surfaces in-app notifications (e.g., a notification center), not as a jarring duplicate interruption on top of the screen she is already looking at.
- **Talia's booking_link_name is later renamed (FEAT-27) after this notification was already delivered** -- This notification is never re-sent or corrected; it is a one-time, point-in-time confirmation. The old link continues forwarding per XBR-27, so a Pro who shared the link from this notification's original wording is unaffected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Triggered by (inbound) | Link activation is the sole trigger event for this notification |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Navigation (outbound) | Every channel's CTA (where one exists) deep-links here |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro notification channel preference this spec follows, and the display_name and booking_link_name placeholders |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (outbound) | Actual text send and delivery-status reporting for this notification's text channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (outbound) | Actual email send and delivery-status reporting for this notification's email channel |

## Analytics and Success Signals

- **onboarding_welcome_confirmation_delivered** (channel: in_app / text / email) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_welcome_confirmation_opened** (channel) -- N/A -- no Stage 2 metric measures whether the Pro opens this specific confirmation; retained so a Pro who never engages with it (despite a live link) is observable, distinct from a Pro who never went live at all
- **onboarding_welcome_cta_tapped** (channel; destination: FEAT-15.SPEC-003) -- supports success-metrics.md: "Setup-to-Live-Link Completion"

## Acceptance Criteria

**FEAT-15.SPEC-008-AC-01:** Given Talia's booking link activates and her notification preferences are the default (in-app + text), when this notification fires, then she receives an in-app notification titled "You're live, Talia!" and a text with her booking link.

**FEAT-15.SPEC-008-AC-02:** Given Talia has set her Pro notification preference to email only, when her link activates, then she receives the email version with subject "Your Chairtime booking link is live" and no text is sent.

**FEAT-15.SPEC-008-AC-03:** Given Talia taps "View my link" from the in-app notification, when the tap registers, then she lands on FEAT-15.SPEC-003 showing her live booking link.

**FEAT-15.SPEC-008-AC-04:** Given Talia's link has already activated once, when any later process re-fires FEAT-15.SPEC-005's activation logic against the same already-active state, then this notification is not sent a second time.

**FEAT-15.SPEC-008-AC-05:** Given Talia's text delivery fails, when the retry-once policy is exhausted, then the confirmation falls back to email, consistent with FEAT-08.SPEC-009's product-wide retry pattern.

**FEAT-15.SPEC-008-AC-06:** Given both Talia's text and email delivery fail, when the failures are final, then the in-app notification stands as the delivery of record and no failure message is shown to her.

**FEAT-15.SPEC-008-AC-07:** Given Talia is actively viewing FEAT-15.SPEC-003 at the exact instant her link activates, when the activation occurs, then the screen itself transitions to the Live state and the in-app notification is available without producing a jarring duplicate interruption.

**FEAT-15.SPEC-008-AC-08:** Given Talia has no display_name set at the theoretical moment of this notification (a state that cannot occur per Go-Live's own required conditions), when the placeholder would render, then the fallback "there" is used rather than a broken or empty greeting.

**FEAT-15.SPEC-008-AC-09:** Given Talia changes her notification preference from text to email after activation fires but before the send completes, when delivery is attempted, then the preference in effect at delivery time (email) governs, not the preference at trigger time.

**FEAT-15.SPEC-008-AC-10:** Given this notification has no quiet-hours window, when Talia's link activates at 2am local time, then the text and email sends are attempted immediately rather than held.

**FEAT-15.SPEC-008-AC-11:** Given Talia later renames her booking link after this notification was delivered, when she checks the link included in the original notification, then it still works (via the forwarding behavior owned by FEAT-27/XBR-27), and this notification is never re-sent with the new name.

**FEAT-15.SPEC-008-AC-12:** Given a client (Riley) has no entitlement to this notification, when the trigger fires, then it is delivered exclusively to Talia and never to any client.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 1 | 1 |
| Preference States | 4 (in-app only, in-app + text default, email only, preference changed mid-flight) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |

