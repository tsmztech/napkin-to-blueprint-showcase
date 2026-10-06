# FEAT-27 — Custom Domain per Freelancer

This chapter covers Custom Domain per Freelancer, a Nice-to-Have-tier feature. It contains the feature breakdown brief followed by every specification in full: 4 specifications carrying 68 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-27.SPEC-001 | Custom Domain Settings | screen | 29 |
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | integration | 15 |
| FEAT-27.SPEC-003 | Custom Domain Validation & Fallback Rule | logic-rule | 14 |
| FEAT-27.SPEC-004 | Custom Domain Verified Confirmation | notification | 10 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Custom Domain per Freelancer

## Summary

**Feature:** Custom Domain per Freelancer
**ID:** FEAT-27
**Description:** A freelancer points her own domain at her portal so clients see the freelancer's brand end-to-end.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Platform
**Rationale:** BRIEF.md, Ecosystem & Integrations: "ideally their own custom domain (timing is an open question)." Nice-to-Have because the shared portal already fulfills the branding promise (FEAT-19) without it; phased Later as the brief itself defers the timing question. Fully white-labeled portals are an established, well-received pattern in the market, which keeps this on the roadmap, while the phase stays Later because BRIEF.md leaves its timing open and the MVP branding (FEAT-19) already delivers the brand promise.

**Key Capabilities:**
- Add a domain — freelancer enters her own domain
- Verify and go live — once verified, the portal is reachable at the freelancer's own domain

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-27.SPEC-001 | Custom Domain Settings | Screen | Nadia (Full); Dana (View, read-only inside a logged support session per FEAT-31) | Nadia adds, views, retries, and removes her custom domain and sees its current verification state |
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Integration | Nadia (Full — configures and monitors); Dana (View, read-only via FEAT-31); Owen, Priya (experience whichever domain this spec determines is active, with no direct control) | Verifies that Nadia controls the domain she added and serves her portal securely at it once verified, reporting verification, failure, and re-check results back |
| FEAT-27.SPEC-003 | Custom Domain Validation & Fallback Rule | Logic/Rule | Nadia (Full); Owen, Priya (protected by the always-available fallback, with no direct control) | Enforces one domain per freelancer account and guarantees the shared default domain always remains reachable as a fallback |
| FEAT-27.SPEC-004 | Custom Domain Verified Confirmation | Notification | Nadia (sole recipient) | Emails Nadia once her custom domain is verified and live |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Add a domain | FEAT-27.SPEC-001, FEAT-27.SPEC-003 | The settings screen captures the domain name; the validation rule enforces the one-domain-per-account limit and domain format before the record is created | Phase 2 (Explicit) |
| Verify and go live | FEAT-27.SPEC-001, FEAT-27.SPEC-002, FEAT-27.SPEC-004 | The Integration spec verifies control of the domain and serves the portal at it; the settings screen surfaces pending/verified/failed status; the confirmation email fires once live | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-32) names a domain-verification and secure-serving capability this feature requires; the feature-dependency-map.md External Touchpoints row names FEAT-27 as the expected owner of this Integration spec |
| FEAT-27.SPEC-003 | Custom Domain Validation & Fallback Rule | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field's one-domain-per-account limit and always-available fallback is the authority behind cross-feature rule XBR-35 (owned by FEAT-27) and governs both the add flow and the removal/failure paths; its cross-feature reach (FEAT-05 depends on it) crossed the threshold for a standalone spec rather than an inline note |
| FEAT-27.SPEC-004 | Custom Domain Verified Confirmation | Phase 4 (Notification surfacing) | The Communications field names a confirmation email with a defined trigger (verified and live), audience (Nadia), and channel (email) -- this crosses the inline-toast threshold and requires a standalone Notification spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Custom Domain Record**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-27.SPEC-001 | Add-domain form on the Custom Domain Settings screen; the entered domain is checked against FEAT-27.SPEC-003 before the record is created | -- |
| Read (single) | FEAT-27.SPEC-001 | The settings screen displays the one configured domain and its current verification state | -- |
| Read (list) | N/A | At most one Custom Domain Record exists per freelancer account (feature-dependency-map.md, Relationships) -- there is nothing to list | -- |
| Update | FEAT-27.SPEC-002 | The domain-verification capability writes verification_state as it progresses (Added -> Verifying -> Verified / Verification Failed with a specific reason); FEAT-27.SPEC-001 also updates domain_name if Nadia replaces her domain, which re-triggers verification via FEAT-27.SPEC-002 | -- |
| Delete/Archive | FEAT-27.SPEC-001 | Hard delete: Nadia removes her configured domain from the settings screen and the portal immediately reverts to the shared default domain (FEAT-27.SPEC-003 fallback rule). No restore path -- re-adding the same domain starts verification from scratch. No cascade -- nothing else references the record. No retention/purge window: the record carries no personal data (feature-dependency-map.md, Data Sensitivity: None), so deletion is immediate and complete. Also deleted as part of a full account deletion, owned by FEAT-24 | Cross-feature: FEAT-24 (Data Export & Account Deletion) also deletes this record |
| State Transition | FEAT-27.SPEC-002 | Added -> Verifying -> Verified, or Added -> Verifying -> Verification Failed (with a specific reason), reported by the domain-verification capability; a re-check from Verification Failed returns to Verifying | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Branding Profile | FEAT-27.SPEC-001 | The settings screen presents the custom domain alongside the freelancer's existing logo and colour, since a verified domain is typically adopted together with Freelancer Branding (FEAT-19) to complete the white-label experience |

## Side-Effect Inventory

This feature produces zero standalone Automation specs: its only entity-creation side-effect (verification) crosses the product boundary to an external capability and is therefore dispositioned directly to the Integration spec (FEAT-27.SPEC-002) per the Phase 4 decision rule, rather than to an intermediate internal Automation.

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia submits a new domain | Validate against the one-domain-per-account limit and domain format | Standalone Logic/Rule | FEAT-27.SPEC-003 |
| Domain passes validation | Create the Custom Domain Record and initiate verification with the domain-verification capability | Standalone Integration | FEAT-27.SPEC-002 |
| Domain-verification capability reports the domain verified | Update the record to Verified, begin serving the portal securely at the custom domain | Standalone Integration (inbound event) | FEAT-27.SPEC-002 |
| Domain verified and live | Send Nadia a confirmation email | Standalone Notification | FEAT-27.SPEC-004 |
| Domain-verification capability reports a failure | Update the record with the specific failure reason; screen names the problem to Nadia and offers re-check, while the shared default domain keeps serving the portal | Standalone Integration (inbound event), surfaced inline on FEAT-27.SPEC-001 | FEAT-27.SPEC-002 / FEAT-27.SPEC-001 |
| Nadia taps re-check on a failed verification | Re-trigger verification with the domain-verification capability | Standalone Integration | FEAT-27.SPEC-002 |
| Nadia removes her configured domain | Portal and client-facing links revert to the shared default domain, no disruption to client access | Inline in triggering screen (governed by FEAT-27.SPEC-003's fallback rule) | FEAT-27.SPEC-001 |
| Dana opens a logged support session on this account | Views the domain's verification status, read-only | Cross-feature -- logged in touchpoints | FEAT-31 responsibility |

## Shared Context

**Shared Entities:**
- Custom Domain Record -- created and deleted by SPEC-001 (with SPEC-003 gating creation), read by SPEC-001, verification-state updated by SPEC-002. Fields: domain_name (required, one per freelancer), verification_state (Added, Verifying, Verified, Verification Failed with a specific failure reason).

**Shared UI Patterns:**
- Single settings surface -- SPEC-001 is the only screen and carries every state of the domain lifecycle (Empty, Loading/pending, Verified/live, Error with re-check) on one view rather than splitting add/verify/remove into separate screens, consistent with this feature's small, linear Key Capability set.

**Shared Validation:**
- SPEC-003 defines the one-domain-per-account limit, domain format checks, and the fallback guarantee. SPEC-001 and SPEC-002 both reference SPEC-003 rather than duplicating the rule: SPEC-001 enforces it at submission and at removal-time fallback display; SPEC-002 relies on it to know there is at most one record to verify per freelancer.

## Internal Dependency Map

```
SPEC-001 (Custom Domain Settings) -> [Nadia submits a domain] -> SPEC-003 (Custom Domain Validation & Fallback Rule) -> [valid] -> SPEC-002 (Domain Verification & Secure Serving)
SPEC-002 (Domain Verification & Secure Serving) -> [verification succeeds] -> SPEC-001 (status updates to Verified/Live)
SPEC-002 (Domain Verification & Secure Serving) -> [verification succeeds] -> SPEC-004 (Custom Domain Verified Confirmation)
SPEC-002 (Domain Verification & Secure Serving) -> [verification fails] -> SPEC-001 (status names the problem, offers re-check)
SPEC-001 (Custom Domain Settings) -> [Nadia taps re-check] -> SPEC-002 (Domain Verification & Secure Serving)
SPEC-001 (Custom Domain Settings) -> [Nadia removes her domain] -> SPEC-003 (Custom Domain Validation & Fallback Rule) -> [reverts to shared default domain]
```

**Default Entry:** SPEC-001 (Custom Domain Settings) -- the screen shown when Nadia navigates into custom domain setup from her branding settings (FEAT-19).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-27.SPEC-001 | Inbound | FEAT-19 (Freelancer Branding) | Navigation into custom domain setup from branding settings | Nadia goes further than logo and colour |
| FEAT-27.SPEC-001 | Outbound | FEAT-19 (Freelancer Branding) | Reads the Branding Profile to present the custom domain alongside existing branding | Settings screen display |
| FEAT-27.SPEC-002 | Outbound | FEAT-05 (Client Portal Access) | Determines the address -- custom domain or shared default -- that the portal and client-facing links are served at (XBR-35) | Domain verified/live, removed, or failed |
| FEAT-27.SPEC-004 | Outbound | FEAT-14 (Notifications (Email)) | Confirmation email sent through the transactional email delivery capability | Domain verified and live |
| FEAT-27.SPEC-001 / SPEC-002 | Outbound | FEAT-31 (Operator Support Access) | Dana views the domain's verification status, read-only, inside a logged support session | Support session opened |
| FEAT-27.SPEC-001 | Inbound | FEAT-24 (Data Export & Account Deletion) | Custom Domain Record deleted as part of a full account deletion cascade | Nadia deletes her account |

## Non-Functional Notes

**Data volumes / growth:** N/A -- at most one Custom Domain Record exists per freelancer account (feature-dependency-map.md, Relationships), so there is no meaningful volume or growth pattern to plan for.

**Responsiveness:** This is a one-time configuration step for Nadia, not an ongoing client-facing dependency (product-features.md, States: "Offline-degraded: N/A -- this is a one-time configuration step, not an ongoing runtime dependency for either party's session"), so the client-facing responsiveness targets in ASMP-21 do not apply to it directly. The settings screen itself shows real progress while verification is pending (records can take time to propagate) rather than a fixed time target, per the loading-state convention in ASMP-27. Once live, every client-facing page served at the custom domain still meets the same responsiveness expectations named for FEAT-05 (ASMP-21), since XBR-35 requires the domain to change only the address, never the experience.

**Data sensitivity / privacy:** The domain name and verification state carry no personal data -- a domain name is public information (feature-dependency-map.md, Data Sensitivity: None). The resulting confirmation email carries the same personal-data classification as every Notification record (recipient identity and message content, GDPR-class), per the dependency map's Notification entity definition; it is addressed to Nadia only.

**Compliance flags:** N/A beyond the general GDPR-class handling that already applies to the confirmation Notification -- no domain-specific compliance regime is named for this feature in assumptions-constraints.md, and ASMP-23's privacy posture (strict client isolation, operator access read-only and visible to the freelancer) continues to hold at the custom domain exactly as XBR-09 and XBR-31 require.

## Non-Goals

- **More than one domain, or per-team-member subdomains** -- Excluded per the Validation & Limits field ("one custom domain per freelancer account," product-features.md) and SC-01: Clientroom has no internal-staff seat model, so there is no second user whose work would need a distinct subdomain.
- **Dana adding, changing, or removing a domain** -- Excluded per SC-04: support sessions (FEAT-31) are strictly read-only and logged; Dana can only view the verification status, never act on it.
- **Ongoing monitoring or alerting for a domain that later breaks after going live** -- Excluded per the States field's own framing: this feature is "a one-time configuration step, not an ongoing runtime dependency for either party's session" (product-features.md, States), so continuous post-verification health monitoring is not part of this feature's specification surface.
- **A public directory or discovery surface listing freelancers' custom domains** -- Excluded per SC-03: portal access is never public or anonymous, and the only public-facing surface adjacent to branding is the referral mark's product page (FEAT-33), which never exposes portal content; a custom domain creates no new public discovery surface.



# Screen Spec: Custom Domain Settings

## Overview

**Name:** Custom Domain Settings
**ID:** FEAT-27.SPEC-001
**Type:** Screen
**Purpose:** Nadia adds, views, replaces, retries, and removes her custom domain from a single settings surface that shows its current verification state.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer

## Scope and Non-Goals

**In Scope:**
- Adding a domain when none is configured
- Displaying the current domain and its verification state (Added, Verifying, Verified, Verification Failed with its specific reason)
- Replacing the configured domain with a different one, and cancelling that replacement
- Requesting a re-check after a failed verification
- Removing the configured domain and confirming the reversion to the shared default portal address
- The data-sharing disclosure notice shown before the first submission, and the "How this is shared" link that reopens it (content owned by FEAT-27.SPEC-002)
- Surfacing capability trouble (slow, unavailable, rejected) and the confirmation-email delivery-failure warning (owned by FEAT-27.SPEC-002 and FEAT-27.SPEC-004 respectively)
- Dana's read-only render of this same screen inside a logged support session (FEAT-31)
- Presenting the custom domain alongside the freelancer's existing branding (read-only reference to the Branding Profile)

**Non-Goals:**
- Verifying domain control and serving the portal securely at a verified domain -- owned by FEAT-27.SPEC-002 (Domain Verification & Secure Serving); this screen only submits requests to it and displays its reported state
- The one-domain-per-account limit, domain-format validation, and the fallback-to-default guarantee -- owned by FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule); this screen enforces those rules by reference, it does not define them
- Editing the freelancer's logo or brand colour -- owned by FEAT-19.SPEC-001 (Branding Settings); this screen reads the Branding Profile for display only
- Ongoing monitoring or alerting for a domain that later breaks after going live -- excluded per product-features.md's States field: this is "a one-time configuration step, not an ongoing runtime dependency for either party's session," so this screen has no post-verification health-check surface

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-19.SPEC-001 (Branding Settings) | Nadia taps "Go further with your own domain" | None -- screen loads whatever Custom Domain Record currently exists, or the Empty state if none |
| FEAT-31 (Operator Support Access) | Dana opens a logged support session on a freelancer account and navigates here | Read-only render; no action controls are exposed |
| Direct return (default entry) | Nadia navigates back into custom domain setup after a previous visit | Screen reloads the current Custom Domain Record and its verification_state |
| FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Nadia taps "View custom domain settings" in the confirmation email | Screen loads the current Custom Domain Record, normally showing the Verified state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | All actions: add, replace, cancel replace, request re-check (from Verification Failed only), remove, open the "How this is shared" notice | -- |
| Owen (Client Primary Contact) | No | No | This screen is not part of the client portal -- Owen has no route to it; he only experiences whichever domain FEAT-27.SPEC-003's fallback rule resolves as active, with no visibility into this configuration surface |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no route to this screen; she experiences whichever domain is active with no visibility into its configuration |
| Dana (Support Operator) | Full screen, read-only render (domain, verification state, failure reason or verification steps, delivery-failure warning, and Branding Profile preview visible) | None -- Add, Replace, Re-check, Remove, "How this is shared", and warning-dismiss controls are omitted entirely, not merely disabled | Support sessions are strictly read-only and logged (XBR-29, ASMP-18); Dana can reach this screen only inside an active, logged support session opened through FEAT-31 -- outside such a session she has no route to it at all |
| Unauthenticated | No | No | Redirected to the sign-in screen for Nadia's own account; there is no client-facing or public route to this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any domain text typed but not yet submitted is discarded -- this screen holds no draft worth preserving across a session boundary since Add/Replace submit immediately on confirmation |

## Layout and Content

**Header:** Screen title "Custom Domain" with a back arrow (returns to FEAT-19.SPEC-001, Branding Settings).

**Delivery-warning banner (conditional):** Appears at the top of the body only when FEAT-27.SPEC-004 reports that the domain-verified confirmation email failed after all retries: "We couldn't email you the confirmation for {domain_name}. Your domain is still verified and live." with a "Dismiss" control. Not shown otherwise.

**Body:** A single content region that shows exactly one of the following presentations, depending on the initial fetch outcome, the current Custom Domain Record, and its verification_state (per the Shared UI Pattern: one settings surface carries every state of the domain lifecycle rather than splitting add/verify/remove into separate screens):

- **Loading:** While the Custom Domain Record and Branding Profile are being fetched on open, placeholder blocks stand in for the domain area and the branding preview; no action controls are shown.
- **Load error:** If the Custom Domain Record cannot be fetched, the message "We couldn't load your custom domain settings." with a "Try again" button. The back arrow stays available.
- **No domain configured:** A short explanation that the portal is currently reachable only at the shared default portal address, a "Your domain" text input, and an "Add domain" button below it.
- **Replace form (Nadia tapped "Replace domain"):** The current domain name shown as a reference, the "Your domain" input (empty), a "Save" button, and a "Cancel" control.
- **Domain configured (Added, Verifying, Verified, or Verification Failed):** The configured domain name displayed as text, a status badge showing the current verification_state, and, below it:
  - When Added: a note that verification is being requested. This is a brief transitional presentation between a successful submission and FEAT-27.SPEC-002 reporting that verification instructions were issued.
  - When Verifying: the verification steps (the verification instructions FEAT-27.SPEC-002 issued and the product stored on the Custom Domain Record), plus a note that domain records can take time to propagate. No "Re-check" button is shown in this state. If FEAT-27.SPEC-002 reports the verification is slow, the note "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." is added.
  - When Verified: a confirmation line that the portal is now reachable at this domain, alongside the shared default address, which always remains reachable too.
  - When Verification Failed: the specific failure reason FEAT-27.SPEC-002 reports, plus a "Re-check" button (the only state in which Re-check appears, per FEAT-27.SPEC-003).
  - In every configured state: a "Replace domain" link and a "Remove domain" button.
- **Capability-unavailable message:** When FEAT-27.SPEC-002 reports the capability is down, the message "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." appears above the domain area, and Add, Save, and Re-check are shown disabled.

**"How this is shared" link:** A text link under the domain area, shown for Nadia in every presentation except Loading and Load error. It opens the data-sharing notice in view-only form.

**Data-sharing notice (dialog):** Text owned by FEAT-27.SPEC-002 (Consent and Disclosure): "To verify you control this domain and serve your portal securely at it, the domain name you enter is shared with the domain-verification capability. Nothing else about your account, clients, or data is shared." Shown with "Continue" and "Cancel" when it interrupts a first submission, or with a single "Close" button when opened from the "How this is shared" link.

**Branding preview region:** Below the domain area, a read-only preview showing the freelancer's current logo and brand colour (read from the Branding Profile, FEAT-19), with the note that a verified custom domain is typically adopted alongside this branding to complete the white-label experience. If only the Branding Profile fetch fails, this region shows "Branding preview unavailable right now." and the domain area is unaffected.

**Footer:** None -- all actions live in the body.

For Dana's read-only render, the same regions appear (domain name, status badge, verification steps or failure reason, delivery-failure warning, branding preview) with the "Add domain" button, "Your domain" input, "Replace domain" link, "Re-check" button, "Remove domain" button, "How this is shared" link, and "Dismiss" control omitted entirely -- she sees values only, consistent with FEAT-19.SPEC-001's equivalent Dana render.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; domain status badge wraps below the domain name if space is constrained.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (exact value is the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Branding preview region:** Uniform scaling, no structural change -- it renders identically to its equivalent region on FEAT-19.SPEC-001 across breakpoints.

## Interactions

Who writes verification_state: this screen and FEAT-27.SPEC-003 set it to Added when a record is created or its domain is replaced; FEAT-27.SPEC-002 alone sets Verifying (when it accepts a submission and issues verification instructions), Verified, and Verification Failed. This screen displays those reports and never writes them.

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Screen open | Load | Fetches the Custom Domain Record and the Branding Profile | Loading presentation until both settle; then the presentation matching the record (or No domain configured) | Placeholder blocks while loading |
| "Try again" button (Load error) | Tap | Repeats the fetch of the Custom Domain Record | Returns to Loading; then to the matching presentation or, if it fails again, Load error | Placeholder blocks while loading |
| "Your domain" input | Type | Captures the entered domain text | Field shows entered text | Standard input focus state |
| "Your domain" input | Blur | No inline validation at this point -- format is checked on submit (per FEAT-27.SPEC-003) | None | None |
| "Add domain" button | Tap | 1. Validate the entered domain via FEAT-27.SPEC-003. 2. If invalid, show the inline error and stop. 3. If valid and Nadia has not yet acknowledged the data-sharing notice, open the notice with "Continue" and "Cancel" and wait. 4. If valid and already acknowledged (or after "Continue"), create the Custom Domain Record (verification_state Added) and submit the domain to FEAT-27.SPEC-002 as one operation. | Button shows a loading state during the create-and-submit step. On acceptance, badge shows "Added" and then "Verifying" once FEAT-27.SPEC-002 reports verification instructions issued | Success: screen switches to the configured presentation showing the verification steps. Failure (invalid format): inline error message below the input per FEAT-27.SPEC-003's exact text. Failure (rejected or not confirmed sent): see the Rejects and Error (submission) rows below |
| "Add domain" button (while loading) | Tap | No action -- ignored while the create is already in progress | Button remains in its loading state | No additional feedback |
| Data-sharing notice "Continue" (first-submission form) | Tap | Records Nadia's acknowledgment against her account (so the notice is not interrupting again), closes the notice, and proceeds with the pending Add, Replace-save, or Re-check request | Acknowledgment flag set; acting button shows its loading state | Notice closes |
| Data-sharing notice "Cancel" (first-submission form) | Tap | Closes the notice and sends nothing to FEAT-27.SPEC-002; no Custom Domain Record is created or changed and no acknowledgment is recorded | Screen returns to the presentation it was in before the tap, with the typed domain text preserved in the input; the notice will appear again on the next submission attempt | Notice closes; no toast |
| "How this is shared" link | Tap | Opens the same notice text in view-only form (single "Close" button); never sends anything | None | Dialog opens; focus moves into it |
| Data-sharing notice "Close" (link form) | Tap | Closes the notice | None | Dialog closes; focus returns to the link |
| "Replace domain" link | Tap | Reopens the "Your domain" input, pre-empty, in place of the current domain display | Screen shows the Replace form, with the current domain name shown as a reference above it | Standard transition, no page navigation |
| "Save" button (Replace form) | Tap | 1. Validate the new domain via FEAT-27.SPEC-003. 2. If valid, apply the data-sharing notice rule as for "Add domain". 3. Replace domain_name on the existing Custom Domain Record, reset verification_state to Added, and submit the new domain to FEAT-27.SPEC-002 as one operation. | Button shows a loading state. On acceptance, badge shows "Added" and then "Verifying" once verification instructions are issued | Success: screen returns to the configured presentation showing the new domain and its verification steps. Failure (format): inline error message per FEAT-27.SPEC-003. Failure (rejected or not confirmed sent): the prior domain and state stay as they were; see the Rejects and Error (submission) rows |
| "Save" button (while loading) | Tap | No action -- ignored while the replace is already in progress | Button remains in its loading state | No additional feedback |
| "Cancel" control (Replace form) | Tap | Discards any typed text and returns to the configured presentation of the existing domain; no request is made and the record is unchanged | Screen returns to the configured presentation showing the prior domain and state | Standard transition, no toast |
| "Re-check" button | Tap | Visible only when verification_state is Verification Failed. Applies the data-sharing notice rule, then re-submits the same domain_name to FEAT-27.SPEC-002 | Button shows a loading state and the failure reason stays visible until FEAT-27.SPEC-002 accepts the request; on acceptance the badge switches to "Verifying" (written by FEAT-27.SPEC-002), the failure reason is cleared, and the "Re-check" button is no longer shown | Toast on acceptance: "Re-checking your domain." |
| "Re-check" button (while loading) | Tap | No action -- ignored while the re-submission is in flight, between the tap and FEAT-27.SPEC-002's acceptance | Button remains in its loading state | No additional feedback |
| Rejected submission | FEAT-27.SPEC-002 rejects an Add, Save, or Re-check | Shows the reason; nothing else changes. A rejected first Add leaves no Custom Domain Record; a rejected Replace-save or Re-check leaves the prior domain and its state (Verified, Verifying, or Verification Failed) untouched. The typed domain text stays in the input | Acting button leaves its loading state | Inline message: "Verification could not start: {reason}. Check the domain and try again." |
| Submission failure (request not confirmed sent) | The Add, Save, or Re-check request fails before FEAT-27.SPEC-002 confirms receipt (for example, Nadia's connection drops) | Nothing is recorded: no record is created or changed | Acting button leaves its loading state; typed text stays in the input | Inline message: "Something went wrong and your domain wasn't saved. Check your connection and try again." |
| Capability slow | FEAT-27.SPEC-002 reports verification is taking longer than usual while Verifying | Adds the slow note to the Verifying presentation; Remove and Replace stay usable | Badge stays "Verifying" | Note: "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." |
| Capability down | FEAT-27.SPEC-002 reports the capability is unavailable | Disables Add, Save, and Re-check; viewing the current domain and state, Replace (opening the form), Cancel, and Remove remain available | The capability-unavailable message appears; disabled controls stay disabled until the capability is reported available on a later load or retry of the screen | Message: "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." |
| Delivery-warning "Dismiss" control | Tap | Hides the delivery-failure banner for this failure; it does not resend the email | Banner removed | None |
| "Remove domain" button | Tap | Opens a confirmation dialog before any change is made | None yet -- confirmation required first | Dialog: "Remove {domain_name}? Your portal will immediately go back to the shared default address. This can't be undone -- re-adding this domain later starts verification from scratch." with "Remove" and "Keep domain" options |
| Confirmation dialog "Remove" | Tap | Deletes the Custom Domain Record via FEAT-27.SPEC-003 | Screen returns to the "No domain configured" presentation | Toast: "Domain removed. Your portal is reachable at the shared default address again." |
| Confirmation dialog "Keep domain" | Tap | Closes the dialog, no change made | None | Dialog closes |
| Back arrow | Tap | Navigate to FEAT-19.SPEC-001 (Branding Settings); in the Replace form or with unsubmitted text in the "Your domain" input, the typed text is discarded without a prompt | Screen closes | Standard transition |
| Branding preview region | -- | Display-only -- reflects the current Branding Profile; not interactive on this screen | None | None |

### Accessibility Notes

- **Focus order:** Back arrow -> delivery-warning banner (when shown) -> domain area (input or domain display, depending on state) -> Replace/Save/Cancel/Re-check/Remove controls, in the order they appear -> "How this is shared" link -> branding preview (non-interactive, skipped in tab order).
- **Validation announcements:** When the domain input enters an error state after Add/Replace, or an inline rejection or submission-failure message appears, the message is announced to assistive technology and programmatically associated with the input.
- **Status announcements:** A verification_state change (Added -> Verifying, Verifying -> Verified, Verifying -> Verification Failed), the slow note, and the capability-unavailable message are announced to assistive technology as they happen, since the status badge is the primary content of this screen once a domain is configured.
- **Dialogs:** Focus moves into the "Remove {domain_name}?" dialog and into the data-sharing notice on open, and returns to the control that opened it on close (either option).
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (initial fetch) | Placeholder blocks for the domain area and branding preview; no controls | Screen opens or "Try again" is tapped | The Custom Domain Record fetch succeeds (matching presentation) or fails (Load error) |
| Load error | "We couldn't load your custom domain settings." with "Try again"; back arrow available | The Custom Domain Record fetch fails | "Try again" is tapped |
| No domain configured (default) | "Your domain" input and "Add domain" button, with the shared-default-address explanation | Screen opens and no Custom Domain Record exists, or Nadia removes the domain | Nadia successfully adds a domain |
| Replace form | Current domain shown as reference, empty "Your domain" input, "Save" and "Cancel" | Nadia taps "Replace domain" | Save succeeds, Cancel is tapped, or Nadia leaves the screen |
| Added | Domain name with an "Added" badge and a note that verification is being requested | Add or Replace-save is accepted, before FEAT-27.SPEC-002 reports instructions issued | FEAT-27.SPEC-002 reports verification instructions issued (Verifying) |
| Verifying | Domain name with a "Verifying" badge, the verification steps, and the propagation note; no Re-check button | FEAT-27.SPEC-002 reports instructions issued (for an Add, Replace-save, or accepted Re-check) | FEAT-27.SPEC-002 reports the domain verified or failed |
| Verifying (slow) | Verifying presentation plus the slow note | FEAT-27.SPEC-002 reports verification is taking longer than usual | The capability reports the domain verified or failed |
| Verified / Live | Domain name with a "Verified · Live" badge and the confirmation that the portal is reachable there | FEAT-27.SPEC-002 reports verification succeeded | Nadia replaces or removes the domain (returns to Added, or to empty) |
| Verification Failed | Domain name with a "Verification Failed" badge, the specific failure reason, and a "Re-check" button | FEAT-27.SPEC-002 reports verification failed | Re-check is accepted (returns to Verifying), or the domain is replaced or removed |
| Loading (action, transient) | The acting button (Add, Save, or Re-check) shows a loading state; the rest of the screen remains as it was | Add, Save, or Re-check is tapped and any required notice is continued | The action completes (accepted, rejected, or failed to send) |
| Notice open | The data-sharing notice over the screen | First submission needs acknowledgment, or "How this is shared" is tapped | Continue, Cancel, or Close |
| Error (format) | Inline error message below the domain input, per FEAT-27.SPEC-003's exact text | Domain-format validation fails on Add or Save | Nadia corrects the input and resubmits |
| Error (rejected) | Inline "Verification could not start: {reason}. Check the domain and try again." message; prior domain and state unchanged | FEAT-27.SPEC-002 rejects the submission | Nadia edits the input and resubmits, or leaves |
| Error (submission failure) | Inline "Something went wrong and your domain wasn't saved. Check your connection and try again." message; nothing changed | The request fails before FEAT-27.SPEC-002 confirms receipt | Nadia retries the action |
| Degraded (capability down) | Capability-unavailable message; Add, Save, and Re-check disabled; viewing, Replace (opening the form), Cancel, and Remove still work | FEAT-27.SPEC-002 reports the capability unavailable | The capability is reported available again on a later load or retry |
| Delivery warning shown | Banner "We couldn't email you the confirmation for {domain_name}. Your domain is still verified and live." above the body | FEAT-27.SPEC-004 reports delivery failed after all retries | Nadia taps "Dismiss", or the domain is replaced or removed |
| Offline | The fetch and submissions fail per Load error and Error (submission failure); no separate offline presentation exists | Nadia's connection is lost | Connection returns and Nadia retries |

## Validation Rules

Validation governed by FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule). See that spec for the domain-format rule, the one-domain-per-account cardinality rule, and the fallback guarantee. This screen applies validation on Add and Save submit, before the data-sharing notice can appear; there is no on-blur validation for the domain input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| Successful add or replace | Remains on this screen, now showing the configured presentation | -- |
| Confirmed remove | Remains on this screen, now showing the "No domain configured" presentation | -- |
| Replace "Cancel" | Remains on this screen, showing the existing domain | -- |

## Data Model

**Creates:** Custom Domain Record -- domain_name set from the "Your domain" input; verification_state set to Added. Validated by FEAT-27.SPEC-003 before creation, and created only in the same operation that FEAT-27.SPEC-002 accepts.
**Reads:** Custom Domain Record -- domain_name, verification_state, the specific failure reason, and the verification instructions, for display. Branding Profile (FEAT-19) -- logo and brand_colour, for the read-only preview region. Nadia's data-sharing acknowledgment flag on her account, to decide whether the notice interrupts a submission.
**Updates:** Custom Domain Record -- domain_name (on Replace) and verification_state (on Replace, resetting to Added). verification_state values Verifying, Verified, and Verification Failed, the failure reason, and the verification instructions are written by FEAT-27.SPEC-002's reports, which this screen reflects but does not itself write. Nadia's data-sharing acknowledgment flag -- set on "Continue".
**Deletes:** Custom Domain Record -- on a confirmed Remove. Hard delete, no restore path.

## Business Rules

- FEAT-27.SPEC-003 governs domain format, the one-domain-per-account limit, and the fallback guarantee -- this screen enforces those rules by reference and never duplicates them.
- XBR-35: with a verified custom domain, the portal and client-facing links use it, and the shared default address always remains available as a fallback -- this screen's copy in every state reflects that guarantee explicitly (e.g., the Verified state names the default address as still reachable; the Remove confirmation names the immediate reversion to it; the capability-unavailable message names it).
- Replacing a domain (FEAT-27.SPEC-003) resets verification_state to Added and re-submits verification via FEAT-27.SPEC-002 -- Nadia cannot keep a prior Verified state against a new domain name.
- Removing a domain is a hard delete with no cascade: nothing else in the product references the Custom Domain Record directly (dependency map, Relationships), so removal has no downstream cleanup beyond the fallback reversion itself.
- Data-sharing notice: the first time Nadia's account attempts an Add, Replace-save, or Re-check, the notice interrupts after format validation passes and before anything is sent. After "Continue" is chosen once for the account, later submissions send directly and only the "How this is shared" link shows the notice again. Nothing leaves the product while the notice is open or after "Cancel" (FEAT-27.SPEC-002, Consent and Disclosure).
- Re-check availability: "Re-check" is shown only in Verification Failed (FEAT-27.SPEC-003 Authorization Rules). While Verifying, including the slow case, Nadia waits, replaces, or removes; FEAT-27.SPEC-002 resolves every Verifying record to Verified or Verification Failed, after which Re-check is available if needed.

## Edge Cases

- **Nadia taps "Add domain" twice rapidly** -- The second tap is ignored while the first create-and-submit request is in progress (button in loading state).
- **Nadia navigates away while Add or Replace is loading** -- The request completes in the background; the next time she opens this screen it reflects whatever the request resolved to (accepted and Verifying, or unchanged if it was rejected or never confirmed sent).
- **Nadia types a domain in the Replace form (or the Add input) and navigates away before submitting** -- The back arrow leaves immediately with no prompt; the typed text is discarded, nothing is sent, and the existing record is unchanged. Returning shows the configured presentation (or the empty Add input).
- **Nadia taps Cancel in the data-sharing notice** -- Nothing is sent, no record is created or changed, no acknowledgment is recorded, and her typed text stays in the input; the notice reappears on her next submission attempt.
- **Nadia's own two sessions both act on the domain (e.g., one tab open on this screen, another used to remove the domain)** -- There is at most one Custom Domain Record and no other human writer (dependency map, Contention: "None -- only Nadia configures it"), so the second session's view simply reflects whichever action completed last; there is no true conflict to reject, only a stale display that refreshes to the current record the next time the screen loads or an action is attempted against it.
- **Verification is still pending when Nadia navigates away and returns** -- The screen re-fetches the Custom Domain Record on load and shows its current verification_state, whatever FEAT-27.SPEC-002 has reported since her last visit.
- **Nadia taps Remove while verification is still Verifying (not yet Verified or Failed)** -- The same confirmation dialog applies; confirming removes the record and reverts to the shared default address immediately, discarding the in-progress verification.
- **Nadia re-adds the exact domain she just removed** -- Treated as a brand-new Custom Domain Record per FEAT-27.SPEC-003: verification starts from Added, with no memory of the prior verification outcome.
- **The capability goes down after Nadia opens the Replace form** -- Save becomes disabled with the capability-unavailable message; Cancel and the typed text are unaffected.
- **The delivery-failure warning is showing and Nadia replaces or removes the domain** -- The banner is removed, because it described the earlier domain's confirmation email.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Domain Verification & Secure Serving) | Triggers (outbound) | Add, Replace-save, and Re-check submit requests to this integration; this screen displays its reported verification_state, verification instructions, failure reasons, and degradation messages, and shows its data-sharing notice |
| FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule) | References (inbound) | Domain-format validation, the one-domain-per-account limit, Re-check availability, and the fallback guarantee govern Add, Replace, Re-check, and Remove |
| FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Navigation (inbound) and Affects (inbound) | The email's CTA lands here; a delivery failure after all retries surfaces here as the delivery-warning banner |
| FEAT-19.SPEC-001 (Branding Settings) | Navigation (inbound) | Entry point via "Go further with your own domain"; also read for the Branding Profile preview |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only entry point during a logged support session |
| FEAT-24 (Data Export & Account Deletion) | References (informational) | The Custom Domain Record this screen manages is also deleted as part of a full account deletion cascade, never by this screen directly |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| custom_domain_added | none beyond the event itself (domain_name is not included, per data-minimization discipline) | Add is accepted and the Custom Domain Record is created | N/A -- no success-metrics.md metric is connected to Custom Domain per Freelancer (FEAT-27); retained per product-features.md's own Signals field so custom-domain adoption remains observable |
| custom_domain_replaced | none | Replace is accepted | N/A -- no success-metrics.md metric is connected to FEAT-27; retained for the same reason as custom_domain_added |
| custom_domain_removed | none | Remove is confirmed | N/A -- no success-metrics.md metric is connected to FEAT-27; retained so removal (and reversion to the fallback) remains observable |
| custom_domain_recheck_requested | none | Re-check is accepted by FEAT-27.SPEC-002 | N/A -- no success-metrics.md metric is connected to FEAT-27; retained so re-check usage after a failure remains observable |
| custom_domain_disclosure_responded | response: continue / cancel / closed | Nadia chooses an option in the data-sharing notice | N/A -- no success-metrics.md metric is connected to FEAT-27; retained so drop-off at the disclosure remains observable |

## Acceptance Criteria

**FEAT-27.SPEC-001-AC-01:** Given Nadia is on the Custom Domain Settings screen with no domain configured and has already acknowledged the data-sharing notice, when she enters a validly formatted domain and taps "Add domain", then the Custom Domain Record is created with verification_state Added, the domain is submitted to FEAT-27.SPEC-002, and once FEAT-27.SPEC-002 reports verification instructions issued the screen shows the domain with a "Verifying" badge and those steps.

**FEAT-27.SPEC-001-AC-02:** Given Nadia enters an invalidly formatted domain, when she taps "Add domain", then an inline error appears below the input per FEAT-27.SPEC-003's exact text, no Custom Domain Record is created, and the data-sharing notice does not appear.

**FEAT-27.SPEC-001-AC-03:** Given Nadia has a domain in the Verification Failed state with a specific reason shown, when she taps "Re-check" and FEAT-27.SPEC-002 accepts the request, then the badge switches to "Verifying", the failure reason is cleared, the "Re-check" button is no longer shown, and the toast "Re-checking your domain." appears.

**FEAT-27.SPEC-001-AC-04:** Given Nadia has a Verified domain, when she taps "Replace domain" and submits a new, validly formatted domain, then domain_name is updated, verification_state resets to Added, the new domain is submitted to FEAT-27.SPEC-002, and the badge moves to "Verifying" once verification instructions are issued for the new domain.

**FEAT-27.SPEC-001-AC-05:** Given Nadia taps "Remove domain", when she confirms "Remove" in the dialog, then the Custom Domain Record is deleted, the screen returns to the "No domain configured" presentation, and a toast confirms the portal is reachable at the shared default address again.

**FEAT-27.SPEC-001-AC-06:** Given Nadia taps "Remove domain", when she chooses "Keep domain" in the dialog, then the dialog closes and the Custom Domain Record is unchanged.

**FEAT-27.SPEC-001-AC-07:** Given Nadia's domain reaches the Verified state, when the screen displays it, then the copy explicitly states that the shared default address remains reachable too, per XBR-35.

**FEAT-27.SPEC-001-AC-08:** Given Dana opens a logged support session on a freelancer account (FEAT-31) and navigates to this screen, when it renders, then she sees the domain name, verification state, and branding preview with every Add, Replace, Re-check, Remove, "How this is shared", and Dismiss control omitted.

**FEAT-27.SPEC-001-AC-09:** Given Owen or Priya, when they look for any route to this screen from the client portal, then none exists -- they experience only whichever domain FEAT-27.SPEC-003 resolves as active.

**FEAT-27.SPEC-001-AC-10:** Given an unauthenticated visitor requests this screen's address directly, when the request is made, then they are redirected to Nadia's sign-in screen with no client-facing route ever exposed.

**FEAT-27.SPEC-001-AC-11:** Given Nadia's session expires while she is on this screen, when she attempts an action, then the dialog "Your session has expired. Sign in to continue." appears and any unsubmitted domain text is discarded.

**FEAT-27.SPEC-001-AC-12:** Given Nadia taps "Add domain" twice rapidly, when the first request is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-27.SPEC-001-AC-13:** Given Nadia has one browser tab open on this screen while removing the domain from a second tab, when she returns to the first tab and takes any action, then it reflects the domain's current state (removed) rather than the stale state it loaded with.

**FEAT-27.SPEC-001-AC-14:** Given Nadia re-adds the exact domain she previously removed, when Add succeeds, then verification starts fresh from Added and reaches Verifying only when FEAT-27.SPEC-002 issues new instructions, with no memory of the prior verification outcome.

**FEAT-27.SPEC-001-AC-15:** Given this screen is loading on a narrow (compact) viewport, when the domain area renders, then the status badge wraps below the domain name if space is constrained, with no structural change to the single-column layout.

**FEAT-27.SPEC-001-AC-16:** Given Nadia's account has never acknowledged the data-sharing notice and she enters a validly formatted domain, when she taps "Add domain", then the notice appears with "Continue" and "Cancel", and no Custom Domain Record is created and nothing is sent to FEAT-27.SPEC-002 until she taps "Continue".

**FEAT-27.SPEC-001-AC-17:** Given the data-sharing notice is open on a first submission, when Nadia taps "Cancel", then the notice closes, no record is created or changed, nothing is sent, the typed domain text remains in the input, and the notice appears again on her next submission attempt.

**FEAT-27.SPEC-001-AC-18:** Given Nadia has previously tapped "Continue" on the notice, when she later submits a Replace or Re-check, then no notice interrupts, and when she taps "How this is shared" then the same notice text opens with a single "Close" button and sends nothing.

**FEAT-27.SPEC-001-AC-19:** Given Nadia's domain is Verifying and FEAT-27.SPEC-002 reports verification is slow, when the screen displays it, then the note "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." appears, no "Re-check" button is shown, and Replace and Remove remain usable.

**FEAT-27.SPEC-001-AC-20:** Given FEAT-27.SPEC-002 reports the capability is unavailable, when Nadia views the screen, then "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." appears, Add, Save, and Re-check are disabled, and viewing the current domain and Remove remain available.

**FEAT-27.SPEC-001-AC-21:** Given Nadia submits a domain that FEAT-27.SPEC-002 rejects, when the rejection is reported, then "Verification could not start: {reason}. Check the domain and try again." appears inline, her typed text stays in the input, a first Add leaves no Custom Domain Record, and a Replace-save or Re-check leaves the prior domain and its state unchanged.

**FEAT-27.SPEC-001-AC-22:** Given Nadia taps "Add domain" and her connection drops before FEAT-27.SPEC-002 confirms receipt, when the request fails, then "Something went wrong and your domain wasn't saved. Check your connection and try again." appears, no record is created or changed, and her typed text stays in the input.

**FEAT-27.SPEC-001-AC-23:** Given the confirmation email for a verified domain fails delivery after all retries (FEAT-27.SPEC-004), when Nadia opens this screen, then the banner "We couldn't email you the confirmation for {domain_name}. Your domain is still verified and live." appears with a "Dismiss" control, and tapping "Dismiss" hides it without resending the email; Dana sees the banner without a Dismiss control.

**FEAT-27.SPEC-001-AC-24:** Given Nadia opens this screen, when the Custom Domain Record and Branding Profile are still being fetched, then placeholder blocks appear in place of the domain area and branding preview and no action controls are shown.

**FEAT-27.SPEC-001-AC-25:** Given the Custom Domain Record fetch fails on open, when the screen settles, then "We couldn't load your custom domain settings." appears with a "Try again" button that repeats the fetch, and the back arrow still returns to FEAT-19.SPEC-001.

**FEAT-27.SPEC-001-AC-26:** Given Nadia has tapped "Replace domain" and typed a new domain, when she taps "Cancel", then the typed text is discarded, no request is made, and the screen shows her existing domain and state unchanged.

**FEAT-27.SPEC-001-AC-27:** Given Nadia has typed text in the Replace form and has not submitted it, when she taps the back arrow, then she leaves to FEAT-19.SPEC-001 with no prompt, the text is discarded, and her existing record is unchanged when she returns.

**FEAT-27.SPEC-001-AC-28:** Given Nadia's domain is in Added, Verifying, or Verified, when she views the screen, then no "Re-check" button is shown; it appears only in Verification Failed.

**FEAT-27.SPEC-001-AC-29:** Given FEAT-27.SPEC-002 has issued verification instructions for Nadia's domain, when the screen shows the Verifying state, then the verification steps are displayed with the propagation note, and the badge value "Verifying" was set by FEAT-27.SPEC-002's report rather than by this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 26 | 26 |
| States | 17 (loading fetch, load error, no domain, replace form, added, verifying, verifying slow, verified, verification failed, loading action, notice open, error format, error rejected, error submission failure, degraded down, delivery warning, offline) | 17 |
| Business Rules | 6 | 6 |
| Edge Cases | 10 | 10 |



# Integration Spec: Domain Verification & Secure Serving

## Overview

**Name:** Domain Verification & Secure Serving
**ID:** FEAT-27.SPEC-002
**Type:** Integration
**Purpose:** Verifies that Nadia controls the domain she added and, once verified, serves her portal securely at it, reporting verification, failure, and re-check results back to the product.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer

## Scope and Non-Goals

**In Scope:**
- Submitting a domain for control verification when Nadia adds or replaces one, or requests a re-check
- Serving the portal securely at a domain once it is verified
- Accepting a submission and issuing the verification instructions (the specific steps Nadia must complete), which sets the Verifying state
- Reporting verification succeeded, verification failed (with a specific reason), and re-check results back to the product
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to Nadia about what is shared with the capability

**Non-Goals:**
- The Custom Domain Settings screen's own mechanics (input, buttons, confirmation dialogs) -- owned by FEAT-27.SPEC-001; this spec defines only the verification-and-serving behavior that screen surfaces
- The one-domain-per-account limit, domain-format validation, and the fallback-to-default guarantee -- owned by FEAT-27.SPEC-003; this spec relies on that rule to know there is at most one record to verify per freelancer and never re-derives the limit itself
- Ongoing monitoring or alerting for a domain that later breaks after going live -- excluded per product-features.md's States field, which frames this feature as "a one-time configuration step, not an ongoing runtime dependency for either party's session"; this spec verifies once and serves once verified, with no continuous health-check loop
- Choosing the domain-verification vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability

## Capability Category

**Category:** Domain verification and secure serving
**Dependency Source:** ASMP-32 -- "Domain-verification capability (Later phase)" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Domain verification and secure serving at a freelancer's own domain -- Later phase (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-27, FEAT-05)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia's added or replaced domain is checked for her control of it | Verify and go live | FEAT-27.SPEC-001 (Custom Domain Settings) |
| Nadia is shown the specific steps she must complete to prove control, and the record moves to Verifying only once the capability has issued them | Verify and go live | FEAT-27.SPEC-001 (Verifying presentation) |
| Once verified, the portal and every client-facing link serve securely at the custom domain, with the shared default address still reachable (XBR-35) | Verify and go live | FEAT-27.SPEC-001 (status display); FEAT-05 (Client Portal Access) resolves the served address |
| A failed verification names the specific problem to Nadia and offers a re-check, while the shared default domain keeps serving the portal | Verify and go live | FEAT-27.SPEC-001 |
| Nadia is emailed once her domain is verified and live | Verify and go live | FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Domain name | Custom Domain Record -- domain_name | Nadia adds a domain, replaces it, or requests a re-check | The capability must know exactly which domain to check control of and, once verified, which domain to serve the portal at |

Nothing else leaves the product for this capability: no client data, no freelancer personal data, no proposal, deliverable, invoice, or payment content ever crosses this boundary, since the Custom Domain Record carries no personal data (feature-dependency-map.md, Data Sensitivity: "None -- a domain name is public information").

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Submission accepted, with verification instructions (the specific steps Nadia must complete, in plain language) | The capability accepts the submitted domain_name from an Add, Replace-save, or Re-check and issues the steps for it | Custom Domain Record -- verification_state set to Verifying, with the verification instructions stored as system-written detail on the record (alongside the failure reason detail), replacing any earlier instructions |
| Verification succeeded | The capability confirms Nadia controls the submitted domain and secure serving is ready | Custom Domain Record -- verification_state set to Verified |
| Verification failed, with a specific reason | The capability cannot confirm control, or secure serving cannot be established, for the submitted domain | Custom Domain Record -- verification_state set to Verification Failed, with the specific failure reason recorded |
| Re-check result (succeeded or failed, same shape as above) | Nadia requests a re-check on a previously failed domain | Custom Domain Record -- verification_state updated to Verified or Verification Failed accordingly |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Verification instructions issued | The capability accepts a submitted domain_name (from Add, Replace-save, or Re-check) and issues the steps for proving control | Custom Domain Record: verification_state set to Verifying (this integration is the only writer of Verifying); verification instructions stored; on a Re-check, the prior failure reason is cleared | FEAT-27.SPEC-001 switches the badge to "Verifying", shows the verification steps and the propagation note, and (on a Re-check) shows the toast "Re-checking your domain." and stops showing "Re-check" | FEAT-27.SPEC-001 |
| Verification succeeded | The capability confirms control of the submitted domain_name and secure serving is ready | Custom Domain Record: verification_state set to Verified | FEAT-27.SPEC-001 shows a "Verified · Live" badge and confirms the shared default address remains reachable too; FEAT-27.SPEC-004 sends Nadia the confirmation email | FEAT-27.SPEC-001, FEAT-27.SPEC-004, FEAT-05 (resolves the served address per XBR-35) |
| Verification failed | The capability cannot confirm control of the submitted domain_name, or cannot establish secure serving for it | Custom Domain Record: verification_state set to Verification Failed, with the specific failure reason recorded | FEAT-27.SPEC-001 names the specific reason and offers "Re-check"; the shared default domain keeps serving the portal throughout, per XBR-35 | FEAT-27.SPEC-001 |
| Re-check result | Nadia's "Re-check" request (FEAT-27.SPEC-001), after the instructions-issued event above put the record in Verifying, resolves | Custom Domain Record: verification_state set to Verified or Verification Failed with a fresh reason | Same feedback as the corresponding succeeded/failed event above | FEAT-27.SPEC-001, and FEAT-27.SPEC-004 if the re-check succeeds |

No event in this integration involves multi-step branching beyond the direct verification_state update and the resulting screen/notification feedback, so none of these events route to a standalone Automation spec.

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-27.SPEC-001 (Custom Domain Settings) | The "Verifying" badge persists past the usual duration; a note appears: "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check." No "Re-check" button is shown while Verifying (FEAT-27.SPEC-003 makes Re-check available only from Verification Failed); this integration always resolves the record to Verified or Verification Failed. The rest of the screen, including Remove, remains fully usable, and the shared default address keeps serving the portal throughout. | "Add domain", "Replace domain" (save), and "Re-check" are disabled with the message: "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." Viewing the current domain and its last known state, and removing the domain, remain available. | The submitted request is declined with the plain-language reason attached: "Verification could not start: {reason}. Check the domain and try again." The typed domain_name stays in the input exactly as entered. No half-created record results: a rejected first Add leaves no Custom Domain Record, and a rejected Replace-save or Re-check leaves the prior domain and its verification_state (Verified, Verifying, or Verification Failed) unchanged. |

## Consent and Disclosure

- **First domain-submission disclosure** -- The first time Nadia's account submits a domain (add, replace-save, or re-check) on FEAT-27.SPEC-001, and after FEAT-27.SPEC-003's format validation has passed, a notice appears before the request is sent: "To verify you control this domain and serve your portal securely at it, the domain name you enter is shared with the domain-verification capability. Nothing else about your account, clients, or data is shared." Options: "Continue" and "Cancel". "Continue" records Nadia's acknowledgment on her account and lets the request proceed; "Cancel" sends nothing, creates or changes no record, and records no acknowledgment, so the notice appears again on her next attempt. Shown once per account; afterwards a "How this is shared" link on FEAT-27.SPEC-001 reopens the same notice text in view-only form (a single "Close" button, nothing sent).
- **What is never shared** -- Every field on every other entity in the product -- client data, proposals, deliverables, invoices, payments, and the freelancer's own personal details -- stays inside the product. Only the submitted domain_name ever crosses this boundary. This boundary is stated explicitly in the disclosure notice above.

## Edge Cases

- **A verification event arrives for a domain Nadia has since removed** -- The event is discarded silently: no Custom Domain Record exists to update, and no user feedback fires, since there is nothing left on FEAT-27.SPEC-001 to reflect it.
- **The same verification-succeeded event is delivered twice** -- The second delivery changes nothing: a Custom Domain Record already Verified stays Verified, and FEAT-27.SPEC-004's confirmation email is not sent a second time (per that spec's own deduplication rule).
- **Events arrive out of order (a failure result for an earlier submission arrives after a later re-check's success)** -- The record reflects the most recent event by the event's own time, not its arrival time; a stale failure arriving after a newer success does not overwrite the Verified state.
- **The capability goes down mid-verification-request** -- If the request was not confirmed sent, the typed domain_name stays in the input and nothing is recorded: a first Add leaves no record, and a Replace-save or Re-check leaves the prior domain and verification_state as they were; no half-submitted state results. FEAT-27.SPEC-001 shows its submission-failure or capability-unavailable message accordingly.
- **The capability accepts a submission but never issues instructions or a result** -- The record stays Added (no instructions yet); FEAT-27.SPEC-001 keeps its "Added" presentation, Nadia can still Replace or Remove, and the shared default address keeps serving throughout.
- **Nadia replaces her domain while a verification for the previous domain is still in flight** -- The in-flight verification for the old domain_name is disregarded once it resolves (there is no longer a record for it to update, per FEAT-27.SPEC-003's replace behavior); only the new domain_name's verification result is applied.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Custom Domain Settings) | Triggered by (inbound) | Add, Replace-save, and Re-check submit verification requests to this integration |
| FEAT-27.SPEC-001 (Custom Domain Settings) | Affects (outbound) | verification_state, the specific failure reason, and degradation messages surface here |
| FEAT-27.SPEC-003 (Custom Domain Validation & Fallback Rule) | References (inbound) | This integration relies on the one-domain-per-account limit to know there is at most one record to verify per freelancer, and on the fallback guarantee to keep the default address serving throughout verification and failure |
| FEAT-27.SPEC-004 (Custom Domain Verified Confirmation) | Triggers (outbound) | The verification-succeeded event fires this notification |
| FEAT-05 (Client Portal Access (Magic-Link Login)) | Affects (outbound) | Resolves, per XBR-35, which address (custom or shared default) the portal and client-facing links are served at |
| FEAT-31 (Operator Support Access) | References (inbound) | Dana's read-only render of FEAT-27.SPEC-001 during a logged support session displays this integration's reported verification_state, with no control over it |

## Analytics and Success Signals

- **custom_domain_verification_succeeded** (none beyond the event itself) -- N/A -- no success-metrics.md metric is connected to Custom Domain per Freelancer (FEAT-27); retained per product-features.md's own Signals field so verification outcomes remain observable
- **custom_domain_verification_failed** (failure_reason_category) -- N/A -- no success-metrics.md metric is connected to FEAT-27; retained for the same reason, so failure patterns remain observable to the product even without a Stage 2 metric measuring them
- **custom_domain_degradation_shown** (condition: slow / down / rejected; screen: FEAT-27.SPEC-001) -- N/A -- no success-metrics.md metric measures degradation frequency for this feature; retained so the product's tolerance for capability trouble on a Nice-to-Have, Later-phase feature stays observable rather than invisible

## Acceptance Criteria

**FEAT-27.SPEC-002-AC-01:** Given Nadia adds a domain on FEAT-27.SPEC-001, when the capability confirms she controls it, then the Custom Domain Record's verification_state is set to Verified, the screen shows "Verified · Live", and Nadia receives the FEAT-27.SPEC-004 confirmation email.

**FEAT-27.SPEC-002-AC-02:** Given Nadia adds a domain, when the capability cannot confirm control, then verification_state is set to Verification Failed with the specific reason recorded, and FEAT-27.SPEC-001 names that reason and offers "Re-check".

**FEAT-27.SPEC-002-AC-03:** Given Nadia's domain is in Verification Failed, when she taps "Re-check" and the capability now confirms control, then verification_state updates to Verified and the confirmation email is sent.

**FEAT-27.SPEC-002-AC-04:** Given Nadia's domain is Verifying, when the capability's response takes longer than usual, then FEAT-27.SPEC-001 shows "This is taking longer than usual -- domain records can take time to propagate. Check back later; if verification does not succeed you will be able to re-check.", shows no "Re-check" button, and the shared default address keeps serving the portal.

**FEAT-27.SPEC-002-AC-05:** Given the capability is unavailable, when Nadia taps "Add domain", "Replace domain" (save), or "Re-check", then that control is disabled with "Domain verification is temporarily unavailable right now. Try again in a few minutes -- your portal keeps working at the shared default address the whole time." and no Custom Domain Record is left half-created.

**FEAT-27.SPEC-002-AC-06:** Given the capability rejects a submitted domain, when the rejection is reported, then FEAT-27.SPEC-001 shows "Verification could not start: {reason}. Check the domain and try again.", the typed domain_name stays in the input, and no record is created or altered by the rejected request.

**FEAT-27.SPEC-002-AC-07:** Given Nadia's account has never submitted a domain before, when she taps "Add domain" with a validly formatted domain for the first time, then the data-sharing notice appears with "Continue" and "Cancel", and no domain data leaves the product until she chooses "Continue"; if she chooses "Cancel", nothing is sent and the notice appears again on her next attempt.

**FEAT-27.SPEC-002-AC-08:** Given a Custom Domain Record is already Verified, when the same verification-succeeded event is delivered again, then nothing changes and no duplicate confirmation email is sent.

**FEAT-27.SPEC-002-AC-09:** Given Nadia removes her domain, when a verification event for that removed domain later arrives, then it is discarded silently with no user feedback and no record updated.

**FEAT-27.SPEC-002-AC-10:** Given a stale failure event for an earlier submission arrives after a newer verification-succeeded event, when both have been received, then the record reflects the more recent event by event time and stays Verified.

**FEAT-27.SPEC-002-AC-11:** Given Nadia replaces her domain while the previous domain's verification is still in flight, when the previous verification later resolves, then it is disregarded and only the new domain's verification result is applied.

**FEAT-27.SPEC-002-AC-12:** Given a verification-succeeded event arrives for any freelancer's domain, when it is processed, then FEAT-05's portal and client-facing links resolve to that domain per XBR-35, while the shared default address remains reachable.

**FEAT-27.SPEC-002-AC-13:** Given Dana is viewing FEAT-27.SPEC-001 inside a logged support session (FEAT-31), when this integration's reported verification_state changes, then she sees the updated state with no control over it.

**FEAT-27.SPEC-002-AC-14:** Given the capability accepts the domain Nadia submitted via Add, Replace-save, or Re-check, when it issues the verification instructions, then the Custom Domain Record's verification_state is set to Verifying by this integration, the instructions are stored on the record, and FEAT-27.SPEC-001 shows the "Verifying" badge with those steps (on a Re-check, the prior failure reason is cleared).

**FEAT-27.SPEC-002-AC-15:** Given Nadia's record is Verifying, Verified, or Added, when she views FEAT-27.SPEC-001, then no "Re-check" button is shown; it is available only after this integration reports Verification Failed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (1 screen x 3 conditions) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Custom Domain Validation & Fallback Rule

## Overview

**Name:** Custom Domain Validation & Fallback Rule
**ID:** FEAT-27.SPEC-003
**Type:** Logic/Rule
**Purpose:** Enforces the domain-format rule and the one-domain-per-account limit on the Custom Domain Record, and guarantees the shared default portal address always remains reachable as a fallback.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer
**Governed Entity:** Custom Domain Record

## Scope and Non-Goals

**In Scope:**
- Field validation for domain_name (format, required)
- The one-domain-per-account cardinality limit, and how Add versus Replace is resolved against it
- Authorization rules for every action on the Custom Domain Record, per role
- Default values and derivations for verification_state
- The fallback guarantee: the shared default portal address always remains reachable, regardless of the Custom Domain Record's state

**Non-Goals:**
- Actually verifying that Nadia controls the submitted domain, or serving the portal securely once verified -- owned by FEAT-27.SPEC-002 (Domain Verification & Secure Serving); this spec defines the entry conditions and the fallback guarantee, not the verification mechanism itself
- The Custom Domain Settings screen's own layout, buttons, and confirmation dialogs -- owned by FEAT-27.SPEC-001; this spec's rules are applied there by reference
- Resolving which address (custom or shared default) FEAT-05's portal and client-facing links actually render at -- owned by FEAT-05 (Client Portal Access), which consumes this spec's verification_state and fallback guarantee per XBR-35, but implements the resolution itself
- Multiple domains, or per-team-member subdomains -- excluded per the Validation & Limits field ("one custom domain per freelancer account," product-features.md) and SC-01: Clientroom has no internal-staff seat model, so there is no second user whose work would need a distinct subdomain

## Governed Entity

**Entity:** Custom Domain Record
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| domain_name | text | The domain Nadia has configured to serve her portal; required, and limited to one per Freelancer Account |
| verification_state | enum | Added, Verifying, Verified, or Verification Failed (carrying a specific failure reason when Failed, and the verification instructions when Verifying; both are system-written detail from FEAT-27.SPEC-002). Added is set here and on FEAT-27.SPEC-001's create and replace; Verifying, Verified, and Verification Failed are written only by FEAT-27.SPEC-002 |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-27.SPEC-001 | Custom Domain Settings | domain_name format validated on Add and Replace-save submit; the one-domain-per-account limit governs whether "Add" or "Replace" applies; authorization on screen entry (which controls are shown at all) and on every action attempt |
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Relies on the one-domain-per-account limit to know there is at most one record to verify per freelancer; relies on the fallback guarantee to keep the shared default address serving throughout verification, failure, and re-check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|---------------|-----------------|-----------|
| domain_name | Required, non-empty | Always | On submit (Add and Replace-save) | "Enter your domain to continue." | Yes |
| domain_name | Must be a recognizable domain name: letters, numbers, and hyphens grouped into labels separated by dots, with no spaces, no protocol prefix (e.g. no "http://" or "https://"), and no path or query text | Always | On submit (Add and Replace-save) | "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." | Yes |
| verification_state | No validation beyond data type -- this field is never set directly from user input; it is set to Added on creation and thereafter only by FEAT-27.SPEC-002's reported events -- instructions issued (Verifying), succeeded, or failed (see Defaults and Derivations) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|------------------|
| Replacing the domain resets verification | domain_name, verification_state | When domain_name is changed on an existing Custom Domain Record (Replace), verification_state is reset to Added in the same operation, and verification is re-triggered via FEAT-27.SPEC-002 for the new value | N/A -- this is an automatic reset, not a rejected input; no error message applies |
| One record per account | domain_name (record cardinality) | At most one Custom Domain Record may exist per Freelancer Account (XBR-35). Submitting a domain when a record already exists is never a second create -- it is resolved as a Replace of the existing record's domain_name, never a second, parallel record | N/A -- there is no invalid-state error here; FEAT-27.SPEC-001 always presents this as "Replace domain" once a record exists, so the ambiguity never reaches the user as an error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|-------------------------------------------------|
| Add domain (create the Custom Domain Record) | Nadia (Freelancer) | Only when no Custom Domain Record currently exists for her account | -- |
| View domain and verification status | Nadia (Freelancer) | Always | -- |
| View domain and verification status | Dana (Support Operator) | Only inside an active, logged support session opened through FEAT-31 (ASMP-18, XBR-29) | Outside a support session, Dana has no route to this data at all -- the screen and the data it would show are simply not reachable |
| Replace domain (update domain_name on an existing record) | Nadia (Freelancer) | Always, when a record exists | -- |
| Request re-check on a failed verification | Nadia (Freelancer) | Only when verification_state is Verification Failed | The "Re-check" control is not shown for any other verification_state, including Verifying (even when FEAT-27.SPEC-002 reports the verification is slow); Nadia waits, replaces, or removes, and FEAT-27.SPEC-002 resolves the record to Verified or Verification Failed |
| Remove domain (delete the Custom Domain Record) | Nadia (Freelancer) | Always, when a record exists | -- |
| Add domain | Owen (Client Primary Contact) | Never | No route to this screen or control exists from the client portal; client contacts have no custom-domain surface at all |
| Replace domain | Owen (Client Primary Contact) | Never | Same as Add domain -- no route exists |
| Request re-check | Owen (Client Primary Contact) | Never | Same as Add domain -- no route exists |
| Remove domain | Owen (Client Primary Contact) | Never | Same as Add domain -- no route exists |
| View domain and verification status | Owen (Client Primary Contact) | Never | Owen has no visibility into this configuration surface; he only experiences whichever domain FEAT-27.SPEC-003's fallback resolves as active (feature-overview.md) |
| Add domain | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| Replace domain | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| Request re-check | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| Remove domain | Priya (Client Reviewer Contact) | Never | Same as Owen -- no route exists |
| View domain and verification status | Priya (Client Reviewer Contact) | Never | Same as Owen -- no visibility into this configuration surface |
| Add domain | Dana (Support Operator) | Never | Support sessions are strictly read-only in every feature (XBR-29, ASMP-18): no domain-configuration control is ever shown to Dana, inside or outside a support session |
| Replace domain | Dana (Support Operator) | Never | Same as Add domain -- read-only boundary (XBR-29, ASMP-18) |
| Request re-check | Dana (Support Operator) | Never | Same as Add domain -- read-only boundary (XBR-29, ASMP-18) |
| Remove domain | Dana (Support Operator) | Never | Same as Add domain -- read-only boundary (XBR-29, ASMP-18) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-----------------------|-----------------|--------------------|
| verification_state | Set to Added | On create (a new Custom Domain Record) | No -- this is the fixed starting state; it is never user-settable |
| verification_state | Reset to Added | On update, whenever domain_name changes (Replace) | No -- the reset is automatic, per the Replacing the domain resets verification cross-field rule |
| verification_state | Set to Verifying, with the verification instructions stored as system-written detail | Whenever FEAT-27.SPEC-002 reports its "verification instructions issued" event after accepting a submission (Add, Replace-save, or Re-check); FEAT-27.SPEC-002 is the only writer of Verifying | No -- Nadia can only trigger a new attempt, never set the state directly |
| verification_state | Set to Verified, or Verification Failed (with a specific failure reason when Failed) | Whenever FEAT-27.SPEC-002 reports a verification succeeded, verification failed, or re-check result | No -- this field is entirely system-derived from FEAT-27.SPEC-002's reports; Nadia can only trigger a new attempt (Add, Replace, Re-check), never set the resulting state directly |

## Business Rules

- XBR-35: With a verified custom domain, the portal and client-facing links use it, and the shared default address always remains available as a fallback -- this fallback holds in every verification_state, including Added, Verifying, and Verification Failed, so the portal is never unreachable while a custom domain is unverified, failed, being replaced, or being removed.
- Removing the Custom Domain Record immediately reverts portal and client-facing links to the shared default address (FEAT-27.SPEC-001, per XBR-35). There is no restore path: re-adding the exact same domain_name later starts verification from Added, with no memory of the prior verification outcome.
- No cascade: nothing else in the product references the Custom Domain Record directly (dependency map, Relationships), so a Remove or a Replace has no downstream records to clean up beyond the fallback reversion itself.
- The Custom Domain Record is also deleted as part of a full account deletion, owned by FEAT-24 (Data Export & Account Deletion) -- that deletion path is FEAT-24's own cascade, not an action this spec's Authorization Rules govern.
- No retention or purge window applies to a removed Custom Domain Record: it carries no personal data (dependency map, Data Sensitivity: None), so deletion is immediate and complete, unlike the legal-retention exception that applies to financial records elsewhere in the product.

## Edge Cases

- **domain_name submitted with a leading or trailing "http://" or "https://"** -- Rejected by the format rule with "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." even though the underlying domain portion may otherwise be valid; the rule does not attempt to strip and salvage a protocol prefix.
- **domain_name submitted with only whitespace** -- Treated as empty; rejected with "Enter your domain to continue."
- **domain_name at the shortest valid form (two labels separated by one dot, e.g. "ab.co")** -- Passes validation; the format rule sets no minimum label count beyond requiring at least one dot separating two non-empty labels.
- **Nadia submits the domain she already has configured, unchanged, via "Replace domain"** -- Treated as a normal Replace: verification_state resets to Added and verification re-triggers, even though domain_name did not actually change in value; the reset is keyed to the Replace action, not to whether the value differs.
- **Nadia attempts to add a domain while one already exists (e.g. by returning to a stale "Add domain" view)** -- The one-record-per-account rule resolves this as a Replace of the existing record, never a second, parallel Custom Domain Record; FEAT-27.SPEC-001 never actually presents an "Add" control once a record exists, so this scenario can only arise from a stale or replayed submission, and it is handled identically to a normal Replace.
- **Dana's support session closes while she is viewing this data** -- Access is revoked immediately; any further attempt to view or act on the Custom Domain Record is denied per the View row's condition (no active session) with no route to the data at all, consistent with FEAT-31's session-close behavior.
- **A Remove and a Replace are both attempted from Nadia's own two open sessions at nearly the same moment** -- There is at most one Custom Domain Record and no other human writer (dependency map, Contention: "None -- only Nadia configures it"), so whichever action reaches the record first completes normally, and the second session's subsequent action operates on the record's new state (e.g., a Replace submitted just after a Remove completed creates a fresh record from Added, rather than updating a record that no longer exists).

## Acceptance Criteria

**FEAT-27.SPEC-003-AC-01:** Given Nadia has no Custom Domain Record, when she submits "myportal.com" via Add domain, then the domain-format rule passes and a Custom Domain Record is created with verification_state Added.

**FEAT-27.SPEC-003-AC-02:** Given Nadia submits an empty domain field, when she taps Add domain, then she sees "Enter your domain to continue." and no record is created.

**FEAT-27.SPEC-003-AC-03:** Given Nadia submits "http://myportal.com", when she taps Add domain, then she sees "Enter a valid domain name (e.g., myportal.com) -- without \"http://\" or any extra text." and no record is created.

**FEAT-27.SPEC-003-AC-04:** Given Nadia already has a Custom Domain Record for "myportal.com", when she submits "newdomain.com" via Replace domain, then domain_name updates to "newdomain.com" and verification_state resets to Added, re-triggering verification.

**FEAT-27.SPEC-003-AC-05:** Given Nadia's Custom Domain Record is in Verification Failed, when she taps Re-check, then the request is allowed and verification is re-triggered for the same domain_name.

**FEAT-27.SPEC-003-AC-06:** Given Nadia's Custom Domain Record is Verified or Verifying (including a slow verification), when she looks for a Re-check control, then none is shown -- Re-check is available only from Verification Failed.

**FEAT-27.SPEC-003-AC-07:** Given Nadia (Freelancer), when she attempts to add, replace, request a re-check on, or remove her domain, then every action is allowed without further condition beyond the record-existence and verification-state conditions stated above.

**FEAT-27.SPEC-003-AC-08:** Given Owen or Priya, when either looks for any control to add, replace, re-check, or remove a custom domain, then no such control or route exists anywhere in the client portal.

**FEAT-27.SPEC-003-AC-09:** Given Dana is inside an active, logged support session opened via FEAT-31, when she views FEAT-27.SPEC-001, then she can see the domain and its verification state but has no Add, Replace, Re-check, or Remove control available to her.

**FEAT-27.SPEC-003-AC-10:** Given Dana's support session has closed, when she attempts to view the Custom Domain Record again, then she has no route to it at all.

**FEAT-27.SPEC-003-AC-11:** Given a domain is verified and reachable, when a client contact opens a portal link, then it resolves to the verified custom domain while the shared default address remains reachable too, per XBR-35.

**FEAT-27.SPEC-003-AC-12:** Given Nadia's domain is in Verification Failed, when a client contact opens a portal link during that time, then the portal is still fully reachable at the shared default address, per the fallback guarantee.

**FEAT-27.SPEC-003-AC-13:** Given Nadia removes her verified domain, when the removal is confirmed, then verification_state and domain_name are cleared (the record is deleted), and portal links immediately resolve to the shared default address again.

**FEAT-27.SPEC-003-AC-14:** Given Nadia re-adds the exact domain she just removed, when the new record is created, then verification_state starts at Added with no memory of the prior verification outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 20 | 20 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Notification Spec: Custom Domain Verified Confirmation

## Overview

**Name:** Custom Domain Verified Confirmation
**ID:** FEAT-27.SPEC-004
**Type:** Notification
**Purpose:** Emails Nadia once her custom domain is verified and live, so she knows without checking the settings screen that her portal now serves at her own domain.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent once FEAT-27.SPEC-002 reports the domain verified and live
- Its single channel (email), content, and delivery behavior

**Non-Goals:**
- Deciding whether a domain is verified -- owned by FEAT-27.SPEC-002 (Domain Verification & Secure Serving); this spec begins only once that spec's verification-succeeded event fires
- Any notification for a failed or re-checked-and-still-failing verification -- product-features.md's Communications field for this feature names only "Confirmation email once the domain is verified and live"; a failure is surfaced inline on FEAT-27.SPEC-001, never by email, so Nadia is not alerted twice for the same event
- In-app or push delivery -- excluded per product-features.md and the dependency map, which define email as the product's sole notification channel; there is no in-app notification surface in MVP (FEAT-29, In-App Notification Center, is Later phase and not connected to this feature)
- Notifying Owen or Priya -- excluded per feature-overview.md: this is Nadia's own configuration event, and Owen and Priya "experience whichever domain this spec determines is active, with no direct control" and no entitlement to be told about the change

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, once verification succeeds | Nadia works from a laptop or desktop and is not necessarily on the Custom Domain Settings screen at the moment verification completes, since propagation can take time; email is the product's sole notification channel and reaches her the moment the one-time configuration step she started actually finishes |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Domain verified and live | FEAT-27.SPEC-002 (Domain Verification & Secure Serving) | Fires once, when the verification-succeeded inbound event sets verification_state to Verified (including a re-check that succeeds) | Freelancer Account identity, Custom Domain Record's domain_name |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient. Per the Access Matrix, this event concerns Nadia's own configuration of her account; Owen and Priya hold no entitlement to it (they only experience whichever domain is active, with no visibility into its configuration, per feature-overview.md), and Dana (Support Operator) never receives product notifications addressed to the freelancer -- her Access Matrix entry for Notifications & Help is limited to viewing delivery warnings, not receiving this or any other freelancer-addressed email.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| (none) | -- | Always sent | -- (this is a transactional, record-core confirmation of a change to how Nadia's own portal is served; XBR-30 places it outside the optional preferences governed by FEAT-21.SPEC-002 and FEAT-21.SPEC-008, alongside the product's other account-critical confirmations) |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for any notification (see FEAT-21.SPEC-011's equivalent decision for the product's other transactional confirmation), and this confirmation in particular reports the completion of a step Nadia herself initiated, so holding it back would only leave her wondering whether it finished.

## Content Definition

**Email:**
- **Subject:** Your custom domain is verified and live
- **Body:**
  Hi {freelancer_name},

  Your domain {domain_name} is now verified. Your portal is reachable there, and it's still reachable at your shared default address too.

  Nothing else changes -- your clients see the same portal, now under your own domain.
- **CTA (button):** View custom domain settings -- deep-links to FEAT-27.SPEC-001 (Custom Domain Settings) showing this domain's Verified state

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_name} | Freelancer Account -- name | Nadia Voss | Greeting renders as "Hi," -- name is never empty in practice since it is required (dependency map, Freelancer Account fields), but the fallback exists for defensive completeness |
| {domain_name} | Custom Domain Record -- domain_name, the value that just reached verification_state Verified | myportal.com | Never empty -- this notification only fires once FEAT-27.SPEC-002 reports a specific domain verified; there is no case where verification succeeds for an unset domain_name |

## Delivery Rules

**Batching:** None -- each verification-succeeded event produces exactly one email, sent individually. Since at most one Custom Domain Record exists per freelancer account (dependency map, Relationships), there is never more than one domain's verification to batch.
**Deduplication:** At most one confirmation per verification-succeeded event. A verification-succeeded event delivered twice for the same already-Verified record (FEAT-27.SPEC-002's own edge case) does not trigger a second email; only the transition into Verified sends this notification, not every subsequent report that the state remains Verified.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning, consistent with XBR-30's "delivery failures are surfaced to the freelancer as warnings"; because this confirmation is account-level rather than project-level, the warning surfaces on the Custom Domain Settings screen (FEAT-27.SPEC-001) rather than on a project.
**Expiry:** This notification never expires unsent in a way that discards it -- the underlying fact (the domain is verified and live) remains true and visible on FEAT-27.SPEC-001 regardless of whether the email itself is ever delivered. If delivery ultimately fails after all retries, the delivery-failure warning above is the surviving signal.

## Edge Cases

- **The Custom Domain Record is removed shortly after verification succeeds but before this email is delivered** -- The email is still delivered, since it reports a real, completed event (the domain was in fact verified); it is a historical confirmation, not a live status view, so Nadia may receive confirmation for a domain she has since removed. The email's content is unaffected because domain_name is captured at trigger time, not re-read at delivery time.
- **Nadia replaces her domain again before this email is delivered** -- The pending email for the first domain's verification is still delivered as written, describing that domain; the replacement domain's own eventual verification (if it succeeds) produces its own, separate confirmation email.
- **The verification-succeeded event fires twice for the same domain (e.g., a delayed duplicate report)** -- Per the Deduplication rule, only the first transition into Verified sends this notification; the duplicate report changes nothing and sends nothing.
- **Quiet hours or a preference change between trigger and delivery** -- Not applicable: this notification has no preference control and no quiet-hours window (see Audience and Preferences), so there is no collision to resolve.
- **Delivery fails because the domain-related confirmation email itself is misrouted by an overzealous spam filter tuned to the new domain** -- Handled identically to any other delivery failure: retried per the Retry rule above, then surfaced as a delivery warning on FEAT-27.SPEC-001; the domain's Verified state and live serving are entirely unaffected by whether this confirmation email is delivered.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Domain Verification & Secure Serving) | Triggered by (inbound) | The verification-succeeded event fires this notification exactly once per transition into Verified |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggers (outbound) | Delivers the email and reports delivery/bounce/failure status |
| FEAT-27.SPEC-001 (Custom Domain Settings) | Navigation (outbound) | The CTA deep-links here, showing the domain's Verified state; delivery-failure warnings for this notification also surface here |

## Analytics and Success Signals

- **custom_domain_verified_confirmation_sent** (channel: email) -- N/A -- no success-metrics.md metric is connected to Custom Domain per Freelancer (FEAT-27); retained per product-features.md's own Communications and Signals fields so the confirmation remains observable
- **custom_domain_verified_confirmation_delivery_failed** (retry_count) -- N/A -- no connected success-metrics.md metric; retained so a failed confirmation is observable rather than silent, consistent with XBR-30's delivery-failure-warning requirement
- **custom_domain_verified_confirmation_cta_tapped** () -- N/A -- no connected success-metrics.md metric; retained to observe whether Nadia opens her domain settings after being confirmed live

## Acceptance Criteria

**FEAT-27.SPEC-004-AC-01:** Given Nadia's domain reaches verification_state Verified, when this notification fires, then she receives an email with the subject "Your custom domain is verified and live" naming her verified domain.

**FEAT-27.SPEC-004-AC-02:** Given Nadia receives this confirmation email, when she taps "View custom domain settings", then she lands on FEAT-27.SPEC-001 showing her domain's Verified state.

**FEAT-27.SPEC-004-AC-03:** Given a verification-succeeded event is delivered twice for the same already-Verified record, when the second delivery is processed, then no second confirmation email is sent.

**FEAT-27.SPEC-004-AC-04:** Given there is no notification preference for this email, when Nadia has every optional notification turned off on FEAT-21.SPEC-002, then this confirmation is still sent whenever her domain verifies.

**FEAT-27.SPEC-004-AC-05:** Given delivery of this email fails once for a transient reason, when the delivery capability retries within platform parameter: `transactional-email-retry-window`, then up to platform parameter: `transactional-email-retry-count` retries occur before any failure is surfaced to Nadia.

**FEAT-27.SPEC-004-AC-06:** Given delivery of this email fails after all retries are exhausted, when the final failure is processed, then a delivery-failure warning appears on FEAT-27.SPEC-001.

**FEAT-27.SPEC-004-AC-07:** Given Nadia removes her domain shortly after verification but before this email is delivered, when delivery proceeds, then the email is still sent describing the domain that was verified.

**FEAT-27.SPEC-004-AC-08:** Given Nadia's domain fails verification (or a re-check still fails), then this notification never fires for that event -- only a successful transition into Verified triggers it.

**FEAT-27.SPEC-004-AC-09:** Given Owen, Priya, or Dana, then none of them ever receives this notification, since only Nadia is entitled to it.

**FEAT-27.SPEC-004-AC-10:** Given Nadia replaces her domain again before this email for the first domain is delivered, when delivery proceeds, then the email is delivered describing the first, already-verified domain, unaffected by the later replacement.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always sent) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
