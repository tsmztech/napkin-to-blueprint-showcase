---
document_type: reconciliation-log
produced_by: cross-reference-reconciler
status: final
created: 2026-09-28
specs_checked: 196
alignment_edits: 57
files_edited: 31
structural_gaps: 9
structural_gaps_resolved: 9
structural_gaps_open: 0
final_pass_alignment_edits: 31
missing_specs: 0
platform_parameters: 9
platform_parameter_marker_sites: 36
---

# Reconciliation Log

The Cross-Reference Reconciler checked all 196 spec files across 25 features: 60 screen, 46 automation, 55 logic-rule, 15 integration and 20 notification specs. It also checked the 25 Feature Breakdown Briefs and the Feature Dependency Map, including its External Touchpoints table. All 14 validation checks were run by shell sweep and targeted reading.

- **Alignment issues:** 57 edits across 31 spec files. Each is logged below with File / Section / Before / After / Rationale.
- **Structural gaps:** 9. They are logged in the Gap Report and returned to the orchestrator for Spec Writer re-spawn.
- **Missing specs:** 0.
- **Platform parameters:** 9 distinct slugs across 33 marker sites. The registry is `.n2b/specifications/platform-parameters.md`.

No Feature Breakdown Brief and no part of the Feature Dependency Map was modified.

## Check Results

| # | Check | Result |
|---|-------|--------|
| 1 | Feature Breakdown Brief completeness | Pass. Every Spec Inventory row has a file, every file has an inventory row, and every inventory type matches its frontmatter `spec_type` (25 Briefs, 196 specs). |
| 2 | Cross-feature references resolve | Pass after edits. Every `FEAT-NN.SPEC-NNN` string in all specs, Briefs and the dependency map resolves to a file. Three references pointed at the wrong kind of target and were realigned: a CTA citing a Notification spec as a screen (FEAT-02.SPEC-013), a hedged "or the feature's equivalent" CTA (FEAT-02.SPEC-014), and a hub row pointing at nonexistent "FEAT-07/FEAT-13 preference screens" (FEAT-01.SPEC-010). |
| 3 | Intra-feature references resolve | Pass. There are no dangling bare `SPEC-NNN` sibling references. |
| 4 | Shared data entity consistency | One finding. Field constraints were swept (household_name 1–60, item names 1–80, safety note 500, invitation expiry 14 days, grace 7 days, 1–12 members, referral 30 days, one week ahead) together with the status enums for Invitation, Subscription billing_state, Swap Suggestion outcome, Weekly Plan and Member Profile. All are consistent except the Member Profile status value "Deleted" (SG-09). |
| 5 | Bidirectional navigation | 36 edits plus 5 shared with Check 2. Destination Entry Points rows were added or corrected per Rule 2, including all Notification CTA deep-link destinations. Six reverse one-way links, where an entry point names a source that declares no outbound navigation, cannot be fixed without writing new source content and are logged as gaps (SG-01, SG-03, SG-04, SG-05, SG-06, SG-07). |
| 6 | Bidirectional automation triggers | 13 edits. Source Outcome Definitions and Interactions now name the automations that cite them as trigger sources. Screen-to-automation and integration-to-automation links are otherwise reciprocal. One external-event link has no cascade content (SG-08). |
| 7 | Logic/Rule vs Screen consistency | Pass, by spot sweep. Badge and disclaimer wording ("Checked against allergies" / "Always check labels"), plan-approval once-per-week, one-open-suggestion-per-member-per-night, vote options 2–3, leftover source within two days and the nudge/correction caps are all described consistently with their Logic/Rule specs. |
| 8 | Spec ID uniqueness | Pass. All 196 IDs are unique and match their filenames and parent features. |
| 9 | Cross-feature business rule consistency | Pass, by spot sweep of XBR-01 to XBR-20 phrasing and numbers across affected specs. |
| 10 | Entity lifecycle completeness | Pass. Every create/update/delete feature named per shared entity in the dependency map has at least one spec that handles the entity. For the Household plan-arrival update attributed to FEAT-07, the screen surface is missing its control (SG-02). |
| 11 | External Touchpoints ↔ Integration specs | Pass. All 15 Integration specs appear in a touchpoint row, every row's cited Integration spec exists, and each spec's Capability Category matches its row. The notification IDs cited in the device-notification row are explanatory "relies on" notes, not claims to be Integration specs. |
| 12 | Notification trigger source consistency | Pass after 1 edit. All 20 Notification specs cite existing source specs, and each source acknowledges the notification. FEAT-04.SPEC-006 also cites FEAT-04.SPEC-010 (a Logic/Rule) as a co-source alongside its screen. This is accepted because the screen, FEAT-04.SPEC-002, now acknowledges it. |
| 13 | Degradation Behavior screen references exist | Pass after 1 edit. Every spec ID in the 15 Degradation Behavior sections exists. For delivery-only capabilities (email, device notification) some "Affected Screen" rows name the Notification, Automation or Logic/Rule spec that is the user-facing surface. The IDs exist and the rows state N/A or name the real screen, so no failure. One row named only a feature and was pinned to FEAT-22.SPEC-001. |
| 14 | Platform-parameter markers → registry | 1 lint edit: an unmarked "platform-set pre-dinner time" in FEAT-13.SPEC-001. The sweep found 0 near-miss markers and no duplicate slugs naming the same parameter. The registry was written with 9 rows and 33 marker sites. Numbers stated concretely (invitation 14 days, grace 7 days, referral and deletion 30 days, import cap 30 per week) trace to Stage 2 decisions in product-features.md or the dependency map, so they are not platform parameters. |

## Edits

### Edit 01 (Check 5)

- **File:** `.n2b/specifications/FEAT-23-manual-weekly-planning/FEAT-23.SPEC-001-weekly-plan-manual-week-builder.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | Organiser taps "Plan this week by hand" on the free-tier placeholder | None -- opens the current/next week to build |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 02 (Check 5)

- **File:** `.n2b/specifications/FEAT-23-manual-weekly-planning/FEAT-23.SPEC-001-weekly-plan-manual-week-builder.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-24.SPEC-002 (Referral Welcome Screen) | A visitor whose own household is on the free tier taps "Go to your plan" | None -- opens the visitor's own current manually built week |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 03 (Check 5)

- **File:** `.n2b/specifications/FEAT-23-manual-weekly-planning/FEAT-23.SPEC-001-weekly-plan-manual-week-builder.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Household member taps the "Tonight: ..." nudge for a manually built week | Scrolled/highlighted to tonight's slot within the current week |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 04 (Check 5)

- **File:** `.n2b/specifications/FEAT-23-manual-weekly-planning/FEAT-23.SPEC-001-weekly-plan-manual-week-builder.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Household member taps the same-day correction for a manually built week | Scrolled/highlighted to tonight's slot, showing the new dinner |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 05 (Check 5)

- **File:** `.n2b/specifications/FEAT-14-subscription-billing-management/FEAT-14.SPEC-002-upgrade-to-paid.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | Maya taps "Upgrade" on the free-tier plan placeholder | None -- the form starts with no plan period preselected |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 06 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-001-account-sign-up-sign-in.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-09.SPEC-002 (Invitation Acceptance) | Invitee taps "Create your own household instead" | None -- screen starts in its default "sign up or sign in" state |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 07 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-001-account-sign-up-sign-in.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-09.SPEC-005 (Leave Household) | Other Adult Member completes leaving the household | None -- the member no longer belongs to a household |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 08 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-001-household-invitations-manager.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Organiser taps the "No eligible recipients" link | None -- screen opens to the invitation list so she can invite an adult first |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 09 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-001-household-invitations-manager.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-18.SPEC-002 (Remove Member Profile) | Organiser taps the "invite a member" link from that screen's empty state | None -- screen opens to the invitation list |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 10 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-003-organiser-hand-over-initiation.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-09.SPEC-005 (Leave Household) | Organiser attempts to leave while still organiser and follows the organiser-blocked message link | A prompt explaining she must hand over the role first |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 11 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-10.SPEC-002 (Review Extracted Recipe) | Member saves an imported recipe, or taps "View existing recipe" when a duplicate is found | The saved (or existing duplicate) recipe's identity |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 12 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-10.SPEC-003 (Manual Recipe Entry) | Member saves a manually entered recipe, or taps "View existing recipe" when a duplicate is found | The saved (or existing duplicate) recipe's identity |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 13 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-19.SPEC-002 (Past Week Detail View) | Household member taps a recipe name in a past week | The recipe's identity |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 14 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-02.SPEC-014 (Safety Concern Resolution Notice) | Household member taps "View recipe" on the resolution notice | The reviewed recipe's identity |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 15 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-001-recipe-library-browse-search.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-10.SPEC-004 (Edit Imported Recipe) | Member removes an imported recipe | None -- list opens without the removed recipe |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 16 (Check 5)

- **File:** `.n2b/specifications/FEAT-18-account-data-management/FEAT-18.SPEC-005-contact-support.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-18.SPEC-001 (Export Household Data) | Maya taps the "contact support" link from the export Error state | None -- screen starts empty |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 17 (Check 5)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-001-weekly-plan-view.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-24.SPEC-002 (Referral Welcome Screen) | A visitor whose own household is on the paid tier taps "Go to your plan" | Current week's plan, no special context |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative; destination Entry Points updated.

### Edit 18 (Check 5)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-001-weekly-plan-view.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Household member taps the same-day correction | Scrolled/highlighted to tonight's dinner, showing the new dinner |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 19 (Check 5)

- **File:** `.n2b/specifications/FEAT-18-account-data-management/FEAT-18.SPEC-001-export-household-data.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-18.SPEC-013 (Export Ready Notification) | Maya taps "Download export" on the export-ready notification | None -- screen loads the household's current (Ready) export state |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 20 (Check 5)

- **File:** `.n2b/specifications/FEAT-04-one-tap-meal-swap/FEAT-04.SPEC-001-meal-swap-direct.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement) | Reporter taps "Show me alternatives" | The emptied slot's night; no current recipe to display since the meal was removed |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 21 (Check 5)

- **File:** `.n2b/specifications/FEAT-04-one-tap-meal-swap/FEAT-04.SPEC-001-meal-swap-direct.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-02.SPEC-012 (Safety Concern Organiser Alert) | Maya taps "Choose a replacement" (in-app or push) | The emptied slot's night; no current recipe to display since the meal was removed |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 22 (Check 5)

- **File:** `.n2b/specifications/FEAT-04-one-tap-meal-swap/FEAT-04.SPEC-002-suggest-a-swap.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| Swap Suggestion Notifications (FEAT-04.SPEC-006) | Sam taps an "accepted", "declined", or "lapsed" notification | The suggestion's slot, showing its outcome |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 23 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-09.SPEC-012 (Invitation Accepted Confirmation) | Organiser taps "View household" | None |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 24 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-09.SPEC-013 (Member Left Household Notification) | Organiser taps "View household" | None |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 25 (Check 5)

- **File:** `.n2b/specifications/FEAT-14-subscription-billing-management/FEAT-14.SPEC-001-plan-tier-overview.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Maya taps "View plan" | None -- loads the household's current Subscription |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 26 (Check 5)

- **File:** `.n2b/specifications/FEAT-14-subscription-billing-management/FEAT-14.SPEC-003-billing-payment-management.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Maya taps "View billing" | None -- loads the household's current Subscription and billing_history |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 27 (Check 5)

- **File:** `.n2b/specifications/FEAT-14-subscription-billing-management/FEAT-14.SPEC-003-billing-payment-management.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Maya taps "Update payment details" | None -- loads the Subscription in its Payment failed billing_state |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 28 (Check 5)

- **File:** `.n2b/specifications/FEAT-22-operator-read-only-support-access/FEAT-22.SPEC-003-household-support-access-record.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-22.SPEC-009 (Support View Recorded Notification) | Maya taps "View record" | None -- screen loads this household's full access history |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 29 (Check 5)

- **File:** `.n2b/specifications/FEAT-22-operator-read-only-support-access/FEAT-22.SPEC-003-household-support-access-record.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-22.SPEC-010 (Support Request Resolved Notification) | Maya taps "View record" | None -- screen loads this household's full access history |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 30 (Check 5)

- **File:** `.n2b/specifications/FEAT-24-invite-another-household/FEAT-24.SPEC-001-invite-another-household-screen.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-24.SPEC-007 (Referral Joined Notification) | Member taps "See who's joined" | None -- screen loads the signed-in member's own link state, including the updated joined-families count |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 31 (Check 5)

- **File:** `.n2b/specifications/FEAT-22-operator-read-only-support-access/FEAT-22.SPEC-001-support-request-queue.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-02.SPEC-013 (Safety Concern Operator Alert) | Riley taps "Review report" in the operator alert email | The household's open safety-concern Support Request, highlighted in the queue |`
- **Rationale:** Rule 2 (Navigation mismatch), Check 5 notification clause: the Notification spec's CTA deep-link is outbound navigation and authoritative; destination Entry Points updated.

### Edit 32 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-005-member-profile-detail.md`
- **Section:** Entry Points
- **Before:** (no row for this source)
- **After:** `| FEAT-01.SPEC-010 (Household Settings Hub) | Any adult taps "Notification preferences" | The signed-in adult's own Member Profile, scrolled to the Notification preferences toggles |`
- **Rationale:** Rule 2 (Navigation mismatch): FEAT-01.SPEC-010 now declares outbound navigation to this screen (see its retargeted row); destination Entry Points updated.

### Edit 33 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-001-household-invitations-manager.md`
- **Section:** Entry Points
- **Before:** `| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Organiser taps "invite" for a partner during guided setup, step 5 |`
- **After:** `| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Invite a partner" during guided setup, step 5 |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative. FEAT-01.SPEC-004 declares the outbound "Invite a partner" link (and FEAT-01's Brief Cross-Feature Touchpoints name FEAT-01.SPEC-004 as the outbound source); FEAT-01.SPEC-003 declares no navigation to FEAT-09.

### Edit 34 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-001-household-invitations-manager.md`
- **Section:** Connected Specs
- **Before:** `| FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) | Navigation (inbound) | Guided setup step 5 opens directly into this screen's send form |`
- **After:** `| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound) | Guided setup step 5 opens directly into this screen's send form |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative. Same correction as the Entry Points row, kept consistent in Connected Specs.

### Edit 35 (Check 5)

- **File:** `.n2b/specifications/FEAT-18-account-data-management/FEAT-18.SPEC-001-export-household-data.md`
- **Section:** Entry Points
- **Before:** `| FEAT-14.SPEC-001 (Plan Tier Overview) | Maya taps "Request an export" from the account settings area (feature-dependency-map.md, Navigation Connections) | None -- screen loads the household's current export state |`
- **After:** `| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Request a data export" from the account settings area (feature-dependency-map.md, Navigation Connections) | None -- screen loads the household's current export state |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative. The only declared outbound export link in FEAT-14 is FEAT-14.SPEC-003's "Request a data export" (Interactions and Navigation Out); FEAT-14.SPEC-001 declares no such navigation.

### Edit 36 (Check 5)

- **File:** `.n2b/specifications/FEAT-18-account-data-management/FEAT-18.SPEC-001-export-household-data.md`
- **Section:** Navigation Out
- **Before:** `| Back arrow tap | Entry source (FEAT-18.SPEC-004 or FEAT-14.SPEC-001) | FEAT-14 when entered from there |`
- **After:** `| Back arrow tap | Entry source (FEAT-18.SPEC-004 or FEAT-14.SPEC-003) | FEAT-14 when entered from there |`
- **Rationale:** Rule 2 (Navigation mismatch): the spec declaring outbound navigation is authoritative. Back-navigation target aligned with the corrected entry source.

### Edit 37 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-02-dietary-rules-allergy-safety-engine/FEAT-02.SPEC-013-safety-concern-operator-alert.md`
- **Section:** Content Definition
- **Before:** `deep-links to FEAT-22.SPEC-009 (Operator Support View, or the feature's equivalent support-visit spec) for this household's open Support Request`
- **After:** `deep-links to FEAT-22.SPEC-001 (Support Request Queue) with this household's open Support Request highlighted`
- **Rationale:** Check 2/5 alignment: the CTA cited FEAT-22.SPEC-009, which is a Notification spec (Support View Recorded Notification), not a screen, and hedged with "or the feature's equivalent". FEAT-22.SPEC-001 is the operator's landing screen and the only entry to FEAT-22.SPEC-002 (per FEAT-22.SPEC-002 Entry Points). Rule 2 applied: destination Entry Points updated in FEAT-22.SPEC-001.

### Edit 38 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-02-dietary-rules-allergy-safety-engine/FEAT-02.SPEC-014-safety-concern-resolution-notice.md`
- **Section:** Content Definition
- **Before:** `deep-links to the recipe's detail view (FEAT-08.SPEC-002 or the feature's equivalent recipe detail spec) for {recipe_name}`
- **After:** `deep-links to FEAT-08.SPEC-002 (Recipe Detail View) for {recipe_name}`
- **Rationale:** Check 2 alignment: removed the hedge "or the feature's equivalent"; FEAT-08.SPEC-002 exists and is the recipe detail screen. Destination Entry Points updated (Rule 2).

### Edit 39 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Layout and Content
- **Before:** `- "Notification preferences" -- links to FEAT-07/FEAT-13 preference screens (every adult, own preferences only)`
- **After:** `- "Notification preferences" -- opens the signed-in adult's own FEAT-01.SPEC-005 (Member Profile Detail) at its plan-ready and nightly-nudge toggles, whose rules are owned by FEAT-07 and FEAT-13 (every adult, own preferences only)`
- **Rationale:** Check 2/5 alignment: the hub pointed at "FEAT-07/FEAT-13 preference screens", which do not exist (FEAT-07 and FEAT-13 define no screens). FEAT-07's Brief (Summary, Cross-Feature Touchpoints) and FEAT-01.SPEC-005 (Layout: Notification preferences toggles, own-only) establish FEAT-01.SPEC-005 as the per-member preference surface. Row retargeted; FEAT-01.SPEC-005 Entry Points updated (Rule 2).

### Edit 40 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Interactions
- **Before:** `| Navigate to the signed-in member's own preference screen (FEAT-07/FEAT-13) | Screen changes | Standard transition, leaving this feature |`
- **After:** `| Navigate to the signed-in member's own FEAT-01.SPEC-005 (Member Profile Detail), Notification preferences toggles (rules owned by FEAT-07/FEAT-13) | Screen changes | Standard transition |`
- **Rationale:** Check 2/5 alignment: the hub pointed at "FEAT-07/FEAT-13 preference screens", which do not exist (FEAT-07 and FEAT-13 define no screens). FEAT-07's Brief (Summary, Cross-Feature Touchpoints) and FEAT-01.SPEC-005 (Layout: Notification preferences toggles, own-only) establish FEAT-01.SPEC-005 as the per-member preference surface. Row retargeted; FEAT-01.SPEC-005 Entry Points updated (Rule 2).

### Edit 41 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Navigation Out
- **Before:** `| "Notification preferences" row tap | -- | FEAT-07 / FEAT-13 |`
- **After:** `| "Notification preferences" row tap | FEAT-01.SPEC-005 (Member Profile Detail, own profile, Notification preferences toggles) | -- (rules owned by FEAT-07 / FEAT-13) |`
- **Rationale:** Check 2/5 alignment: the hub pointed at "FEAT-07/FEAT-13 preference screens", which do not exist (FEAT-07 and FEAT-13 define no screens). FEAT-07's Brief (Summary, Cross-Feature Touchpoints) and FEAT-01.SPEC-005 (Layout: Notification preferences toggles, own-only) establish FEAT-01.SPEC-005 as the per-member preference surface. Row retargeted; FEAT-01.SPEC-005 Entry Points updated (Rule 2).

### Edit 42 (Check 13)

- **File:** `.n2b/specifications/FEAT-02-dietary-rules-allergy-safety-engine/FEAT-02.SPEC-010-transactional-email-delivery-safety-reports.md`
- **Section:** Degradation Behavior
- **Before:** `| FEAT-22 (Operator Read-Only Support Access) | The operator alert email may arrive later`
- **After:** `| FEAT-22.SPEC-001 (Support Request Queue, FEAT-22) | The operator alert email may arrive later`
- **Rationale:** Check 13 alignment: the Affected Screen column named only a feature; the "Support View" through which Riley discovers the open Support Request is FEAT-22.SPEC-001 (Support Request Queue), an existing Screen spec.

### Edit 43 (Check 6)

- **File:** `.n2b/specifications/FEAT-14-subscription-billing-management/FEAT-14.SPEC-008-apply-subscription-change.md`
- **Section:** Outcome Definitions
- **Before:** `| Upgrade applied | Upgrade payment succeeded | tier set to paid; billing_period set to the chosen period; billing_state set to Active; current_period_end_date set one billing_period ahead; billing_history entry appended | FEAT-14.SPEC-001 shows the new Paid tier; FEAT-14.SPEC-010 sends the upgrade confirmation | FEAT-14.SPEC-001, FEAT-14.SPEC-002, FEAT-14.SPEC-010, FEAT-03, FEAT-05, FEAT-12 |`
- **After:** `| Upgrade applied | Upgrade payment succeeded | tier set to paid; billing_period set to the chosen period; billing_state set to Active; current_period_end_date set one billing_period ahead; billing_history entry appended | FEAT-14.SPEC-001 shows the new Paid tier; FEAT-14.SPEC-010 sends the upgrade confirmation | FEAT-14.SPEC-001, FEAT-14.SPEC-002, FEAT-14.SPEC-010, FEAT-03 (FEAT-03.SPEC-004), FEAT-05, FEAT-12, FEAT-24.SPEC-005 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 44 (Check 6)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-003-scheduled-weekly-plan-generation.md`
- **Section:** Outcome Definitions
- **Before:** `| Generation succeeded | Steps 1-13 complete with a full seven-dinner plan fitting the budget | New Weekly Plan (Generated) and seven Planned Meals (Proposed) created; prior Active plan archived | FEAT-03.SPEC-001 shows the new week; FEAT-07.SPEC-001 sends the plan-ready notification; FEAT-06.SPEC-002 recalculates the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002 |`
- **After:** `| Generation succeeded | Steps 1-13 complete with a full seven-dinner plan fitting the budget | New Weekly Plan (Generated) and seven Planned Meals (Proposed) created; prior Active plan archived | FEAT-03.SPEC-001 shows the new week; FEAT-07.SPEC-001 sends the plan-ready notification; FEAT-06.SPEC-002 recalculates the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002, FEAT-11.SPEC-002, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 45 (Check 6)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-003-scheduled-weekly-plan-generation.md`
- **Section:** Outcome Definitions
- **Before:** `| Generation succeeded, over budget | Steps 1-13 complete, but no safe combination of seven dinners fits weekly_budget | Same as above, plus Weekly Plan.over_budget_note set by FEAT-03.SPEC-006 | FEAT-03.SPEC-001 shows the plan with the over-budget note in the weekly total banner | FEAT-03.SPEC-001, FEAT-03.SPEC-006 |`
- **After:** `| Generation succeeded, over budget | Steps 1-13 complete, but no safe combination of seven dinners fits weekly_budget | Same as above, plus Weekly Plan.over_budget_note set by FEAT-03.SPEC-006 | FEAT-03.SPEC-001 shows the plan with the over-budget note in the weekly total banner | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-11.SPEC-002, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 46 (Check 6)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-004-first-plan-generation-on-upgrade.md`
- **Section:** Outcome Definitions
- **Before:** `| First-plan generation succeeded | Steps 1-12 complete with a full seven-dinner plan fitting the budget | First Weekly Plan (Generated) and seven Planned Meals (Proposed) created | FEAT-03.SPEC-001 resolves from its Empty state to a populated plan; FEAT-07.SPEC-001 sends the plan-ready notification with first-plan framing; FEAT-06.SPEC-002 builds the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002 |`
- **After:** `| First-plan generation succeeded | Steps 1-12 complete with a full seven-dinner plan fitting the budget | First Weekly Plan (Generated) and seven Planned Meals (Proposed) created | FEAT-03.SPEC-001 resolves from its Empty state to a populated plan; FEAT-07.SPEC-001 sends the plan-ready notification with first-plan framing; FEAT-06.SPEC-002 builds the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002, FEAT-11.SPEC-002 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 47 (Check 6)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-004-first-plan-generation-on-upgrade.md`
- **Section:** Outcome Definitions
- **Before:** `| First-plan generation succeeded, over budget | Steps 1-12 complete, but no safe week fits weekly_budget | Same as above, plus over_budget_note set | FEAT-03.SPEC-001 shows the plan with the over-budget note | FEAT-03.SPEC-001, FEAT-03.SPEC-006 |`
- **After:** `| First-plan generation succeeded, over budget | Steps 1-12 complete, but no safe week fits weekly_budget | Same as above, plus over_budget_note set | FEAT-03.SPEC-001 shows the plan with the over-budget note | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-11.SPEC-002 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 48 (Check 6)

- **File:** `.n2b/specifications/FEAT-04-one-tap-meal-swap/FEAT-04.SPEC-004-apply-meal-swap.md`
- **Section:** Outcome Definitions
- **Before:** `| Swap applied (direct) | Safety re-check passes, triggered from FEAT-04.SPEC-001 | Planned Meal recipe, status, swap_history updated; any other open suggestion on the slot superseded (FEAT-04.SPEC-010); grocery list signaled | Weekly Plan shows the new dinner in the slot immediately | FEAT-04.SPEC-001, FEAT-06 |`
- **After:** `| Swap applied (direct) | Safety re-check passes, triggered from FEAT-04.SPEC-001 | Planned Meal recipe, status, swap_history updated; any other open suggestion on the slot superseded (FEAT-04.SPEC-010); grocery list signaled | Weekly Plan shows the new dinner in the slot immediately | FEAT-04.SPEC-001, FEAT-06, FEAT-11.SPEC-004, FEAT-13.SPEC-003, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 49 (Check 6)

- **File:** `.n2b/specifications/FEAT-04-one-tap-meal-swap/FEAT-04.SPEC-004-apply-meal-swap.md`
- **Section:** Outcome Definitions
- **Before:** `| Swap applied (accepted suggestion) | Safety re-check passes, triggered from FEAT-04.SPEC-003 | Same as above, plus the accepted Swap Suggestion's outcome set to Accepted | The suggestion card is removed from FEAT-04.SPEC-003's list; the suggesting member is notified their suggestion was accepted (FEAT-04.SPEC-006); Weekly Plan shows the new dinner | FEAT-04.SPEC-003, FEAT-04.SPEC-010, FEAT-04.SPEC-006, FEAT-06 |`
- **After:** `| Swap applied (accepted suggestion) | Safety re-check passes, triggered from FEAT-04.SPEC-003 | Same as above, plus the accepted Swap Suggestion's outcome set to Accepted | The suggestion card is removed from FEAT-04.SPEC-003's list; the suggesting member is notified their suggestion was accepted (FEAT-04.SPEC-006); Weekly Plan shows the new dinner | FEAT-04.SPEC-003, FEAT-04.SPEC-010, FEAT-04.SPEC-006, FEAT-06, FEAT-11.SPEC-004, FEAT-13.SPEC-003, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 50 (Check 6)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-007-invitation-acceptance-processing.md`
- **Section:** Outcome Definitions
- **Before:** `| Acceptance succeeds | Invitation is still Sent at processing time | New Member Profile created (Active, Other Adult Member); Invitation.status set to Accepted | Invitee is routed into FEAT-15 first-use onboarding; organiser receives FEAT-09.SPEC-012 | FEAT-09.SPEC-002, FEAT-09.SPEC-001, FEAT-15, FEAT-09.SPEC-012 |`
- **After:** `| Acceptance succeeds | Invitation is still Sent at processing time | New Member Profile created (Active, Other Adult Member); Invitation.status set to Accepted | Invitee is routed into FEAT-15 first-use onboarding; organiser receives FEAT-09.SPEC-012 | FEAT-09.SPEC-002, FEAT-09.SPEC-001, FEAT-15 (FEAT-15.SPEC-002), FEAT-09.SPEC-012 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 51 (Check 6)

- **File:** `.n2b/specifications/FEAT-23-manual-weekly-planning/FEAT-23.SPEC-006-apply-manual-pick.md`
- **Section:** Outcome Definitions
- **Before:** `| Pick applied | A new pick's create succeeds | New Planned Meal created (status "Picked"); Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06 |`
- **After:** `| Pick applied | A new pick's create succeeds | New Planned Meal created (status "Picked"); Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 52 (Check 6)

- **File:** `.n2b/specifications/FEAT-23-manual-weekly-planning/FEAT-23.SPEC-006-apply-manual-pick.md`
- **Section:** Outcome Definitions
- **Before:** `| Change applied | A change's update succeeds | Existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge overwritten; Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06 |`
- **After:** `| Change applied | A change's update succeeds | Existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge overwritten; Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 53 (Check 6)

- **File:** `.n2b/specifications/FEAT-03-ai-weekly-dinner-plan-generation/FEAT-03.SPEC-005-auto-adoption-at-week-start.md`
- **Section:** Outcome Definitions
- **Before:** `| Plan auto-adopted | Weekly Plan.status is Generated when the week starts | Weekly Plan.status transitions to Active; approval set to auto-adopted | FEAT-03.SPEC-001 shows "Active (adopted)" in place of an Approve control; no blocking message, since the household is never left without a plan | FEAT-03.SPEC-001, FEAT-03.SPEC-011 |`
- **After:** `| Plan auto-adopted | Weekly Plan.status is Generated when the week starts | Weekly Plan.status transitions to Active; approval set to auto-adopted | FEAT-03.SPEC-001 shows "Active (adopted)" in place of an Approve control; no blocking message, since the household is never left without a plan | FEAT-03.SPEC-001, FEAT-03.SPEC-011, FEAT-21.SPEC-003 |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed.

### Edit 54 (Check 6)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-003-household-naming-guided-setup-start.md`
- **Section:** Interactions
- **Before:** `3. Trigger FEAT-01.SPEC-011 (Default Subscription Provisioning). |`
- **After:** `3. Trigger FEAT-01.SPEC-011 (Default Subscription Provisioning). 4. The completed household creation triggers FEAT-24.SPEC-004 (Household Referral Recording), which records a referral only when referral context is present. |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed. FEAT-24.SPEC-004 Trigger Definition cites FEAT-01.SPEC-003 as its trigger source.

### Edit 55 (Check 6)

- **File:** `.n2b/specifications/FEAT-14-subscription-billing-management/FEAT-14.SPEC-003-billing-payment-management.md`
- **Section:** Interactions
- **Before:** `| Submits updated payment details through FEAT-14.SPEC-009 | Button shows loading state |`
- **After:** `| Submits updated payment details through FEAT-14.SPEC-009; while billing_state is Payment failed, the submission also triggers FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) to retry the charge | Button shows loading state |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed. FEAT-14.SPEC-007 Trigger Definition cites FEAT-14.SPEC-003 ("Payment details updated during grace").

### Edit 56 (Check 12)

- **File:** `.n2b/specifications/FEAT-04-one-tap-meal-swap/FEAT-04.SPEC-002-suggest-a-swap.md`
- **Section:** Interactions
- **Before:** `| Submits the suggestion via FEAT-04.SPEC-010 (creates the Swap Suggestion, enforcing one-open-per-member-per-slot) |`
- **After:** `| Submits the suggestion via FEAT-04.SPEC-010 (creates the Swap Suggestion, enforcing one-open-per-member-per-slot); the new suggestion notifies Maya via FEAT-04.SPEC-006 (Swap Suggestion Notifications) |`
- **Rationale:** Check 6/12 bidirectionality: the downstream spec's Trigger Definition (the side declaring the dependency) is authoritative, by analogy with Rule 2; the source spec's acknowledgment is updated to name the spec it triggers. No behavior changed. FEAT-04.SPEC-006 Trigger cites FEAT-04.SPEC-002 as the "Suggestion submitted" source.

### Edit 57 (Check 14)

- **File:** `.n2b/specifications/FEAT-13-tonights-dinner-reminder/FEAT-13.SPEC-001-tonights-nudge-trigger.md`
- **Section:** Scope and Non-Goals
- **Before:** `- Firing once per household per day, at a single platform-set pre-dinner time`
- **After:** `` - Firing once per household per day, at a single platform-set pre-dinner time (platform parameter: `nightly-nudge-send-time`) ``
- **Rationale:** Check 14 lint: a platform-set value phrasing without the marker; rewritten to carry the existing slug used by the same spec's Trigger Definition and by FEAT-13.SPEC-002.

## Gap Report (returned to orchestrator)

These findings describe content that does not exist, so the reconciler cannot resolve them by aligning existing text. Each needs a Spec Writer re-spawn for the named source spec. None needs a new spec.

### SG-01 [STRUCTURAL-GAP] Household Settings Hub has no outbound links to the FEAT-09 membership screens

**Status: RESOLVED (final pass).** Fixed by FEAT-01.SPEC-010. It now has Layout, Interactions, Navigation Out and Connected Specs rows for "Household Invitations" → FEAT-09.SPEC-001, "Hand over organiser role" → FEAT-09.SPEC-003, the pending hand-over entry → FEAT-09.SPEC-004 and "Leave household" → FEAT-09.SPEC-005 (AC-14 to AC-18). All four FEAT-09 Entry Points reciprocate. FEAT-09.SPEC-004's trigger wording was aligned in Final Edit F10.

- **Source spec:** FEAT-01.SPEC-010 (Household Settings Hub)
- **Target specs:** FEAT-09.SPEC-001 (Household Invitations Manager), FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance), FEAT-09.SPEC-005 (Leave Household)
- **What is missing:** Layout rows, Interactions rows and Navigation Out rows in FEAT-01.SPEC-010 for "Household Invitations", "Hand over organiser role", the recipient's pending hand-over banner or entry, and "Leave household".
- **Evidence:** All four FEAT-09 screens list FEAT-01.SPEC-010 as an Entry Point source. FEAT-01.SPEC-010 Scope says "this hub links to invitation management". Its Navigation Out table has rows only for FEAT-01.SPEC-003/004/008, FEAT-16, notification preferences, FEAT-22 and FEAT-21.

### SG-02 [STRUCTURAL-GAP] The plan-arrival day/time control is absent from the Household Settings Hub

**Status: RESOLVED (final pass).** Fixed by FEAT-01.SPEC-010. It now has an inline "Plan arrival" row and picker (organiser edits, Sam read-only), Updates plan_arrival_day_time per FEAT-07.SPEC-004, and carries platform parameter: `plan-arrival-time-slots` (AC-11 to AC-13). FEAT-07.SPEC-004's "linked screen" wording was aligned to the inline picker in Final Edits F01 to F04.

- **Source spec:** FEAT-01.SPEC-010 (Household Settings Hub)
- **Target spec:** FEAT-07.SPEC-004 (Plan-Arrival Day & Time Setting Rule)
- **What is missing:** A settings row and control (or a linked sub-view defined within FEAT-01.SPEC-010) through which the organiser views and changes `plan_arrival_day_time`, and through which Sam sees it read-only.
- **Evidence:** FEAT-07.SPEC-004 Enforced By names FEAT-01.SPEC-010 ("when the organiser opens the linked plan-arrival setting"), its Authorization Rules refer to "FEAT-01.SPEC-010's linked screen", and one AC opens "the plan-arrival setting from FEAT-01.SPEC-010". The FEAT-07 Brief (Side-Effect Inventory) says "Inline in FEAT-01.SPEC-010". FEAT-01.SPEC-010 never mentions `plan_arrival_day_time` or an arrival setting. The dependency map's Navigation Connections also list "FEAT-01 household settings → FEAT-07 plan-ready preference and plan-arrival day".

### SG-03 [STRUCTURAL-GAP] Guided setup never hands off to FEAT-16, and the order before Setup Complete conflicts

**Status: RESOLVED (final pass).** Fixed by FEAT-01.SPEC-003, FEAT-01.SPEC-004, FEAT-01.SPEC-008, FEAT-01.SPEC-009 and FEAT-16.SPEC-001. The single order is FEAT-01.SPEC-003 (Step 1 of 8) → FEAT-01.SPEC-004 (Step 2 of 8) → FEAT-01.SPEC-008 (Step 5 of 8) → FEAT-16.SPEC-001 → FEAT-16.SPEC-002 → FEAT-01.SPEC-009 (Step 8 of 8). FEAT-01.SPEC-008's guided save now goes to FEAT-16.SPEC-001, and FEAT-01.SPEC-009's only entry is FEAT-16.SPEC-002 "Finish". FEAT-16 step-number wording was aligned in Final Edits F24 to F29.

- **Source specs:** FEAT-01.SPEC-003 (Household Naming & Guided Setup Start) and FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup)
- **Target specs:** FEAT-16.SPEC-001 (Units & Currency Settings), FEAT-16.SPEC-002 (Aisle Name Customization), FEAT-01.SPEC-009 (Setup Complete & Next Steps)
- **What is missing:** No FEAT-01 guided-setup screen navigates to FEAT-16.SPEC-001, yet "Set units, currency and aisle layout" is First Household Setup step 4. The step order is also contradictory: FEAT-01.SPEC-008's guided save goes straight to FEAT-01.SPEC-009, while FEAT-16.SPEC-002's guided "Finish" also claims to go to FEAT-01.SPEC-009. FEAT-01.SPEC-009 lists only FEAT-01.SPEC-008 as an entry.
- **Evidence:** FEAT-16.SPEC-001 Entry Points cite FEAT-01.SPEC-003 ("Guided setup reaches step 4"). FEAT-01.SPEC-003 Navigation Out goes only to FEAT-01.SPEC-004 and FEAT-01.SPEC-010. The FEAT-01 Brief's Cross-Feature Touchpoints list FEAT-01.SPEC-003 → FEAT-16 as outbound, and the dependency map has "FEAT-01 guided setup → FEAT-16". This was not auto-aligned because two outbound links (FEAT-01.SPEC-008 and FEAT-16.SPEC-002) both claim to precede FEAT-01.SPEC-009, so Rule 2 cannot pick one. The wizard sequence needs one defined order.

### SG-04 [STRUCTURAL-GAP] Grocery List declares no navigation to Pantry or to the ordering handoff

**Status: RESOLVED (final pass).** Fixed by FEAT-06.SPEC-001. It now has a "Hand off list" header control → FEAT-20.SPEC-001 (gated by FEAT-20.SPEC-003) and a "View Pantry" toast action → FEAT-05.SPEC-001 (gated by FEAT-05.SPEC-007), in Interactions, Navigation Out and AC-17 to AC-20. The destination triggers were aligned in Final Edits F14 and F15.

- **Source spec:** FEAT-06.SPEC-001 (Grocery List)
- **Target specs:** FEAT-05.SPEC-001 (Pantry List & Item Entry), FEAT-20.SPEC-001 (Grocery Handoff Screen)
- **What is missing:** The "view the pantry" follow-up after "Already have it", and the "hand off list" control (Later phase) with its navigation.
- **Evidence:** FEAT-06.SPEC-001 Navigation Out says "This screen has no navigation-out targets", and its Scope says it "only carries the future hand-off point" without defining it. FEAT-05.SPEC-001 and FEAT-20.SPEC-001 both list FEAT-06.SPEC-001 as an Entry Point source. The dependency map's Navigation Connections include FEAT-06 → FEAT-05 ("Tap 'already have it' and add to pantry") and FEAT-06 → FEAT-20 ("Tap 'hand off list'").

### SG-05 [STRUCTURAL-GAP] Recipe Detail View has no "Edit" path for imported recipes

**Status: RESOLVED (final pass).** Fixed by FEAT-08.SPEC-002. It now has a header "Edit" button for imported recipes owned by the household (Maya and Sam only) → FEAT-10.SPEC-004 (AC-13, AC-14). FEAT-10.SPEC-004's entry trigger and stale reconciliation note were aligned in Final Edits F16 and F17. FEAT-08.SPEC-002 now lists FEAT-10.SPEC-004 as an entry (F18).

- **Source spec:** FEAT-08.SPEC-002 (Recipe Detail View)
- **Target spec:** FEAT-10.SPEC-004 (Edit Imported Recipe)
- **What is missing:** An "Edit" interaction and Navigation Out row for imported recipes the member may edit.
- **Evidence:** FEAT-10.SPEC-004 Entry Points list FEAT-08.SPEC-002 ("Household member taps 'Edit' on an imported recipe they can act on"). FEAT-08.SPEC-002 says "View only -- no edit or add-to-plan controls exist on this screen", and its Scope assigns the edit surface to FEAT-10. That leaves FEAT-10.SPEC-004 with no declared way in.

### SG-06 [STRUCTURAL-GAP] Organiser Hand-Over Acceptance never offers the link to My Account

**Status: RESOLVED (final pass).** Fixed by FEAT-09.SPEC-004. Its Success state now offers "Review your account details" → FEAT-18.SPEC-004 (Interactions, States, Navigation Out, AC-10). FEAT-18.SPEC-004's trigger wording was aligned in Final Edit F13.

- **Source spec:** FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance)
- **Target spec:** FEAT-18.SPEC-004 (My Account)
- **What is missing:** The post-accept link that FEAT-18.SPEC-004 says is offered to the new organiser.
- **Evidence:** FEAT-18.SPEC-004 Entry Points: "After accepting the organiser role, the new organiser is offered a link back to review their own account details". FEAT-09.SPEC-004 never references FEAT-18, and its accept outcome navigates only to FEAT-01.SPEC-010.

### SG-07 [STRUCTURAL-GAP] My Account's "Contact Support" link is only conditional

**Status: RESOLVED (final pass).** Fixed by FEAT-18.SPEC-004 and FEAT-18.SPEC-005. My Account now has a definite "Contact Support" row in Layout, Interactions, Navigation Out and AC-16. FEAT-18.SPEC-005's entry is no longer hedged.

- **Source spec:** FEAT-18.SPEC-004 (My Account)
- **Target spec:** FEAT-18.SPEC-005 (Contact Support)
- **What is missing:** A definite "Contact Support" row in FEAT-18.SPEC-004 Layout, Interactions and Navigation Out, or removal of the hedged entry from FEAT-18.SPEC-005.
- **Evidence:** FEAT-18.SPEC-005 Entry Points: "Adult member taps a 'Contact Support' link, if surfaced from account settings". FEAT-18.SPEC-004 Navigation Out has no support row and does not mention support anywhere except the Riley access row. This is low severity: the persistent-navigation entry to FEAT-18.SPEC-005 already exists.

### SG-08 [STRUCTURAL-GAP] Household deletion cascade omits the calendar connection

**Status: RESOLVED (final pass).** Fixed by FEAT-18.SPEC-008. It adds Processing Logic step 5 (disconnect via FEAT-21.SPEC-002, no-op if none), an Outcome acknowledgment, a Connected Specs row, AC-11 and AC-12. FEAT-21.SPEC-002's inbound-event wording was aligned to "during the cascade (step 5)" in Final Edit F31.

- **Source spec:** FEAT-18.SPEC-008 (Household Deletion Processing)
- **Target spec:** FEAT-21.SPEC-002 (Family Calendar Integration)
- **What is missing:** A Processing Logic step and Outcome acknowledgment that disconnects any active Calendar Connection as part of the deletion cascade (Later phase).
- **Evidence:** FEAT-21.SPEC-002 Inbound Events has a "Household deletion cascade" row: "FEAT-18.SPEC-008 (Household Deletion Processing) completes a household deletion ... Any active Calendar Connection for the household is disconnected". FEAT-18.SPEC-008 never mentions the calendar, FEAT-21 or Calendar Connection.

### SG-09 [STRUCTURAL-GAP] Member Profile status value "Deleted" is outside the shared enum

**Status: RESOLVED (final pass).** Fixed by FEAT-18.SPEC-009 and FEAT-18.SPEC-010. Own-account deletion now sets Member Profile status to Removed. FEAT-18.SPEC-010 states the shared enum as Active, Invited, Left, Removed, and says the flow performing the change is what tells the two apart. A sweep finds no remaining Member Profile "Deleted" value. Household "Closed/Deleted" is a separate Household.status enum and is unaffected.

- **Source specs:** FEAT-18.SPEC-009 (Own-Account Deletion Processing), FEAT-18.SPEC-010 (Account Data Validation Rules)
- **Target specs:** FEAT-15.SPEC-003 (Onboarding Eligibility & Once-Only Rule), FEAT-22.SPEC-007 (Kid Profile & Billing Data Visibility Rule); Member Profile entity in the Feature Dependency Map
- **What is missing:** One agreed Member Profile status set. FEAT-18 sets status to "Deleted" on own-account deletion (FEAT-18.SPEC-009 steps and AC-01; FEAT-18.SPEC-010 Governed Entity and Defaults), but the dependency map enum is Invited, Active, Left, Removed. FEAT-15.SPEC-003 and FEAT-22.SPEC-007 enumerate those four values only.
- **Evidence:** Rule 1 names the creating feature (FEAT-01/FEAT-09) as authoritative. Neither enumerates the status values, and the reconciler may not edit the dependency map. Resolving this means choosing between mapping self-deletion to "Removed" in FEAT-18 and adopting "Deleted" as a fifth value in every enumeration. The distinction between self-initiated and organiser-initiated deletion is load-bearing in FEAT-18.SPEC-010, so this is returned rather than auto-aligned.

## Observations (no action required)

- FEAT-21.SPEC-003's trigger row cites FEAT-03.SPEC-008 (a Logic/Rule) as a source for "approved". Approval is initiated on FEAT-03.SPEC-001 ("Trigger approval via FEAT-03.SPEC-008"), so the chain is traceable. Consider citing FEAT-03.SPEC-001 on the next FEAT-21 re-spawn.
- FEAT-03.SPEC-002 lists FEAT-14.SPEC-008 (reversion to free) as an entry context. FEAT-14.SPEC-008's downgrade outcomes cite FEAT-23 rather than FEAT-03.SPEC-002. This is a state-driven routing condition, not navigation, so it is left as is.
- FEAT-01.SPEC-018 and FEAT-02.SPEC-012 carry feature-level CTA targets ("FEAT-04", "FEAT-03"). The corresponding screens (FEAT-04.SPEC-001, FEAT-03.SPEC-001) already carry entries for those flows at feature level.
- Delivery retry figures inside individual Notification Delivery Rules ("3 retries over 6 hours") are per-notification delivery behavior, not a platform-wide policy claim, and are left unmarked.

## Final Reconciliation Pass (alignment-only, after gap routing)

This pass ran after Feature Spec Producers revised FEAT-01.SPEC-003/004/008/009/010, FEAT-16.SPEC-001, FEAT-06.SPEC-001, FEAT-08.SPEC-002, FEAT-09.SPEC-004 and FEAT-18.SPEC-004/005/008/009/010 to close SG-01 to SG-09. It re-ran the alignment checks by shell sweep, scoped to the revised specs and their counterparts: dangling references (Checks 2 and 3), bidirectional navigation and entry points (Check 5), trigger links (Check 6), notification sources (Check 12), Degradation Behavior screen refs (Check 13), External Touchpoints ↔ Integration specs (Check 11), shared-entity fields and enums (Check 4) and the platform-parameter sweep (Check 14). No Feature Breakdown Brief, no part of the dependency map and nothing under `.n2b/tracking/` was modified.

### Structural gap status

| Gap | Status | Fixing specs |
|-----|--------|--------------|
| SG-01 | Resolved | FEAT-01.SPEC-010 |
| SG-02 | Resolved | FEAT-01.SPEC-010 |
| SG-03 | Resolved | FEAT-01.SPEC-003, FEAT-01.SPEC-004, FEAT-01.SPEC-008, FEAT-01.SPEC-009, FEAT-16.SPEC-001 |
| SG-04 | Resolved | FEAT-06.SPEC-001 |
| SG-05 | Resolved | FEAT-08.SPEC-002 |
| SG-06 | Resolved | FEAT-09.SPEC-004 |
| SG-07 | Resolved | FEAT-18.SPEC-004, FEAT-18.SPEC-005 |
| SG-08 | Resolved | FEAT-18.SPEC-008 |
| SG-09 | Resolved | FEAT-18.SPEC-009, FEAT-18.SPEC-010 |

Open structural gaps: 0. New structural gaps: 0. Missing specs: 0.

### Final-pass check results

| # | Check | Result |
|---|-------|--------|
| 2/3 | References resolve | Pass. No dangling `FEAT-NN.SPEC-NNN` reference exists anywhere. One wildcard navigation target, "FEAT-08.SPEC-*" in FEAT-19.SPEC-002, was pinned to FEAT-08.SPEC-002 (F21 to F23). |
| 4 | Shared entity consistency | Pass. The Member Profile status enum is now uniform (SG-09). The new hub reads of `plan_arrival_day_time` and `Household.pending_organiser_handover` match their defining specs (FEAT-07.SPEC-004, FEAT-09.SPEC-003). |
| 5 | Bidirectional navigation | 25 edits (F06 to F30). Every new forward link from a revised spec has a matching destination Entry Points row. Destination triggers now name the actual controls. Feature-level hub, setup-complete and entry rows are pinned to spec IDs. Back-arrow, cancel and edit-mode return routes to a prior screen are treated as returns, not entry points, as in the full pass. |
| 6 | Bidirectional triggers | 1 edit (F31). FEAT-18.SPEC-008 → FEAT-21.SPEC-002 is reciprocal, and the event timing is aligned. |
| 7 | Logic/Rule vs Screen | 4 edits (F01 to F04). FEAT-07.SPEC-004 now describes the hub's inline picker rather than a "linked screen". |
| 11 | External Touchpoints ↔ Integration | Pass. No Integration spec or touchpoint row was added or removed. FEAT-21.SPEC-002 is unchanged in capability. |
| 12 | Notification trigger sources | Pass. FEAT-18.SPEC-013/014/015 sources (FEAT-18.SPEC-006/008/005) still acknowledge their notifications. |
| 13 | Degradation Behavior screen refs | Pass. Every ID in FEAT-21.SPEC-002 Degradation Behavior exists. |
| 14 | Platform-parameter markers → registry | 1 lint edit (F05): a concrete "6:00 pm" example on the hub. Strict sweep: 9 distinct slugs, 36 marker occurrences. Near-miss sweep: 0. No duplicate slugs. New site FEAT-01.SPEC-010 (`plan-arrival-time-slots`, 3 occurrences) was added to the registry, and `marker_site_count` was updated from 33 to 36. |

### Final-pass edits

### Final Edit F01 (Check 5 / Check 7)

- **File:** `.n2b/specifications/FEAT-07-weekly-plan-ready-notification/FEAT-07.SPEC-004-plan-arrival-day-time-setting-rule.md`
- **Section:** Enforced By
- **Before:** | FEAT-01.SPEC-010 | Household Settings Hub | On save, when the organiser opens the linked plan-arrival setting and submits a new day/time pair |
- **After:** | FEAT-01.SPEC-010 | Household Settings Hub | On save, when the organiser expands the inline plan-arrival picker on the hub and submits a new day/time pair |
- **Rationale:** The hub (FEAT-01.SPEC-010, the screen this rule names as owning the edit surface) now defines the plan-arrival control as an inline picker, not a linked screen. Rule 4 keeps value rules here; the surface description is aligned to the screen that owns it.

### Final Edit F02 (Check 7)

- **File:** `.n2b/specifications/FEAT-07-weekly-plan-ready-notification/FEAT-07.SPEC-004-plan-arrival-day-time-setting-rule.md`
- **Section:** Authorization Rules
- **Before:** shown to Sam as a read-only household fact within FEAT-01.SPEC-010's linked screen; no edit control is present.
- **After:** shown to Sam as a read-only household fact on FEAT-01.SPEC-010 (a plain summary row with no chevron); no picker control is present.
- **Rationale:** Aligns with FEAT-01.SPEC-010 Access and Visibility and AC-13 (Sam sees the plan-arrival summary read-only, no chevron, no picker).

### Final Edit F03 (Check 7)

- **File:** `.n2b/specifications/FEAT-07-weekly-plan-ready-notification/FEAT-07.SPEC-004-plan-arrival-day-time-setting-rule.md`
- **Section:** Business Rules
- **Before:** FEAT-01.SPEC-010's edit screen and FEAT-03.SPEC-003's schedule read
- **After:** FEAT-01.SPEC-010's inline plan-arrival picker and FEAT-03.SPEC-003's schedule read
- **Rationale:** Same alignment: the hub edits the value inline, not on a separate edit screen.

### Final Edit F04 (Check 7)

- **File:** `.n2b/specifications/FEAT-07-weekly-plan-ready-notification/FEAT-07.SPEC-004-plan-arrival-day-time-setting-rule.md`
- **Section:** Acceptance Criteria (AC-01)
- **Before:** Given Maya opens the plan-arrival setting from FEAT-01.SPEC-010 with no prior change made, when the screen loads, then it shows
- **After:** Given Maya expands the inline plan-arrival picker on FEAT-01.SPEC-010 with no prior change made, when the picker opens, then it shows
- **Rationale:** Same alignment: the setting is an inline picker on the hub, so the AC opens the picker rather than loading a screen. The tested value is unchanged.

### Final Edit F05 (Check 14)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Layout and Content
- **Before:** current day and time-slot summary (e.g., "Sunday, 6:00 pm")
- **After:** current day and time-slot summary (e.g., "Sunday" with the chosen Evening slot from platform parameter: `plan-arrival-time-slots`)
- **Rationale:** Check 14 lint: the example fixed a concrete platform-wide slot value (6:00 pm) that FEAT-07.SPEC-004 leaves to the `plan-arrival-time-slots` parameter. The example now carries the existing slug.

### Final Edit F06 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Entry Points
- **Before:** | FEAT-09.SPEC-013 (Member Left Household Notification) | Organiser taps "View household" | None |
- **After:** | FEAT-09.SPEC-013 (Member Left Household Notification) | Organiser taps "View household" | None | ⏎ | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | New organiser taps "Go to Household Settings" on the Success state, or the recipient confirms a decline, or taps "Back to Household" on the withdrawn state | None -- the hub loads with the viewer's current role (organiser access after a completed hand-over) |
- **Rationale:** Rule 2: FEAT-09.SPEC-004 Navigation Out declares three forward routes to FEAT-01.SPEC-010. The destination Entry Points now list that source.

### Final Edit F07 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Navigation Out
- **Before:** | "Units, currency & aisles" row tap | -- | FEAT-16 (Units, Currency & Locale Configuration) |
- **After:** | "Units, currency & aisles" row tap | FEAT-16.SPEC-001 (Units & Currency Settings) | FEAT-16 (Units, Currency & Locale Configuration) |
- **Rationale:** The destination FEAT-16.SPEC-001 names this row as an entry. The feature-level destination is pinned to that spec ID so the link resolves in both directions.

### Final Edit F08 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Navigation Out
- **Before:** | "Support access record" row tap | -- | FEAT-22 (Operator Read-Only Support Access) |
- **After:** | "Support access record" row tap | FEAT-22.SPEC-003 (Household Support Access Record) | FEAT-22 (Operator Read-Only Support Access) |
- **Rationale:** The destination FEAT-22.SPEC-003 names this row as an entry. The feature-level destination is pinned to that spec ID.

### Final Edit F09 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-010-household-settings-hub.md`
- **Section:** Navigation Out
- **Before:** | "Connect calendar" row tap (Later) | -- | FEAT-21 (Family Calendar Sync) |
- **After:** | "Connect calendar" row tap (Later) | FEAT-21.SPEC-001 (Calendar Connection Settings) | FEAT-21 (Family Calendar Sync) |
- **Rationale:** The destination FEAT-21.SPEC-001 names this row as its sole entry. The feature-level destination is pinned to that spec ID.

### Final Edit F10 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-004-organiser-hand-over-acceptance.md`
- **Section:** Entry Points
- **Before:** | FEAT-01.SPEC-010 (Household Settings Hub) | Recipient has a pending request and opens a persistent banner/entry in settings | Same pending request |
- **After:** | FEAT-01.SPEC-010 (Household Settings Hub) | Recipient has a pending request and taps the pending hand-over entry ("{organiser display name} wants to make you the organiser") | Same pending request |
- **Rationale:** Rule 2: the trigger now matches the control FEAT-01.SPEC-010 declares (Layout, Interactions, Navigation Out; AC-16).

### Final Edit F11 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-003-organiser-hand-over-initiation.md`
- **Section:** Entry Points
- **Before:** | FEAT-18 (Account & Data Management) | Organiser attempts to leave or delete her own account while still organiser (XBR-15) | A prompt explaining she must hand over the role first, linking here |
- **After:** | FEAT-18.SPEC-004 (My Account) | Organiser taps "Hand over role" on the Deletion blocked dialog after attempting to delete her own account while still organiser (XBR-15) | A prompt explaining she must hand over the role first, linking here |
- **Rationale:** Rule 2: FEAT-18.SPEC-004 Navigation Out declares "Hand over role" (deletion blocked) → FEAT-09.SPEC-003. The feature-level source is pinned to that spec. The leave path is already covered by the FEAT-09.SPEC-005 row.

### Final Edit F12 (Check 5)

- **File:** `.n2b/specifications/FEAT-09-household-invitations-membership/FEAT-09.SPEC-003-organiser-hand-over-initiation.md`
- **Section:** Connected Specs
- **Before:** | FEAT-18 (Account & Data Management) | Navigation (inbound) | XBR-15's leave/delete gate links here |
- **After:** | FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | XBR-15's own-account deletion gate (Deletion blocked state) links here |
- **Rationale:** Same alignment as the Entry Points row.

### Final Edit F13 (Check 5)

- **File:** `.n2b/specifications/FEAT-18-account-data-management/FEAT-18.SPEC-004-my-account.md`
- **Section:** Entry Points
- **Before:** | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | After accepting the organiser role, the new organiser is offered a link back to review their own account details |
- **After:** | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | New organiser taps "Review your account details" on the hand-over Success state |
- **Rationale:** Rule 2: the trigger now names the control FEAT-09.SPEC-004 declares (Interactions, States, Navigation Out; AC-10).

### Final Edit F14 (Check 5)

- **File:** `.n2b/specifications/FEAT-05-pantry-aware-suggestions/FEAT-05.SPEC-001-pantry-list-item-entry.md`
- **Section:** Entry Points
- **Before:** | FEAT-06.SPEC-001 (Grocery List) | Household member taps "already have it" on a grocery list line, then chooses to view the pantry (governed by FEAT-05.SPEC-007) |
- **After:** | FEAT-06.SPEC-001 (Grocery List) | Maya or Sam taps "Already have it" on a plan-derived grocery list line, then taps the "View Pantry" action on the confirmation toast (action offered only to roles with Pantry Input access, per FEAT-05.SPEC-007) |
- **Rationale:** Rule 2: the trigger now matches the "View Pantry" toast action and role gating that FEAT-06.SPEC-001 declares (Interactions, Navigation Out; AC-19, AC-20).

### Final Edit F15 (Check 5)

- **File:** `.n2b/specifications/FEAT-20-online-grocery-ordering-handoff/FEAT-20.SPEC-001-grocery-handoff-screen.md`
- **Section:** Entry Points
- **Before:** | FEAT-06.SPEC-001 (Grocery List) | Household taps "hand off list" |
- **After:** | FEAT-06.SPEC-001 (Grocery List) | Maya or Sam taps the "Hand off list" header control (shown only while the household's region is handoff-eligible, per FEAT-20.SPEC-003) |
- **Rationale:** Rule 2: the trigger now matches the header control FEAT-06.SPEC-001 declares (Layout, Interactions, Navigation Out; AC-17).

### Final Edit F16 (Check 5)

- **File:** `.n2b/specifications/FEAT-10-recipe-import-from-web-link/FEAT-10.SPEC-004-edit-imported-recipe.md`
- **Section:** Entry Points
- **Before:** | FEAT-08.SPEC-002 (Recipe Detail View) | Household member taps "Edit" on an imported recipe they can act on |
- **After:** | FEAT-08.SPEC-002 (Recipe Detail View) | Maya or Sam taps the header "Edit" button on an imported recipe owned by the household |
- **Rationale:** Rule 2: the trigger now matches the "Edit" control FEAT-08.SPEC-002 declares (Access, Layout, Interactions, Navigation Out; AC-13).

### Final Edit F17 (Check 5)

- **File:** `.n2b/specifications/FEAT-10-recipe-import-from-web-link/FEAT-10.SPEC-004-edit-imported-recipe.md`
- **Section:** Entry Points (Cross-feature reconciliation note)
- **Before:** As currently written, FEAT-08.SPEC-002's Access and Visibility table describes Maya and Sam as "View only -- no edit or add-to-plan controls exist on this screen" and its Interactions table shows no Edit control or navigation to FEAT-10. This spec assumes FEAT-08.SPEC-002 will show an "Edit" control (and, per this spec's Footer, a path to "Remove recipe") to Maya and Sam specifically when the recipe's origin is imported -- never on starter recipes, which stay read-only for households. This is flagged here for the Cross-Reference Reconciler (Pass D) to reconcile against FEAT-08.SPEC-002 directly; this spec does not modify that sibling spec.
- **After:** FEAT-08.SPEC-002 now shows an "Edit" control to Maya and Sam specifically when the recipe's origin is imported and it is owned by the household -- never on starter recipes, which stay read-only for households -- and navigates here carrying the recipe's ID and current field values (FEAT-08.SPEC-002 Interactions, Navigation Out and AC-13). The "Remove recipe" path stays on this screen, per this spec's Footer. Reconciled in Pass D (SG-05).
- **Rationale:** The note described FEAT-08.SPEC-002 as lacking the Edit control. That spec now declares it, so the note is aligned to the current source and records the resolution.

### Final Edit F18 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Entry Points
- **Before:** | FEAT-10.SPEC-003 (Manual Recipe Entry) | Member saves a manually entered recipe, or taps "View existing recipe" when a duplicate is found | The saved (or existing duplicate) recipe's identity |
- **After:** | FEAT-10.SPEC-003 (Manual Recipe Entry) | Member saves a manually entered recipe, or taps "View existing recipe" when a duplicate is found | The saved (or existing duplicate) recipe's identity | ⏎ | FEAT-10.SPEC-004 (Edit Imported Recipe) | Member saves an edit to an imported recipe, or taps the back arrow with no unsaved changes | The edited recipe's identity |
- **Rationale:** Rule 2: FEAT-10.SPEC-004 Navigation Out declares "Successful save" (and the back arrow) → FEAT-08.SPEC-002. The destination Entry Points now list that source, matching the rows for FEAT-10.SPEC-002 and SPEC-003.

### Final Edit F19 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Entry Points
- **Before:** | FEAT-03 (AI Weekly Dinner Plan Generation, planned meal) | Household member taps a meal on the weekly plan |
- **After:** | FEAT-03.SPEC-001 (Weekly Plan View) | Household member taps a dinner card on the weekly plan |
- **Rationale:** Rule 2: FEAT-03.SPEC-001 Navigation Out declares "Dinner card tap" → FEAT-08.SPEC-002. The feature-level source is pinned to that spec.

### Final Edit F20 (Check 5)

- **File:** `.n2b/specifications/FEAT-08-recipe-library-starter-recipes/FEAT-08.SPEC-002-recipe-detail-view.md`
- **Section:** Connected Specs
- **Before:** | FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (inbound) | Tapping a planned meal opens this same detail view |
- **After:** | FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (inbound) | Tapping a dinner card opens this same detail view |
- **Rationale:** Same alignment as the Entry Points row.

### Final Edit F21 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-19-weekly-plan-history/FEAT-19.SPEC-002-past-week-detail-view.md`
- **Section:** Interactions
- **Before:** Open recipe detail (FEAT-08.SPEC-* Recipe Library) for the archived recipe
- **After:** Open recipe detail (FEAT-08.SPEC-002 Recipe Detail View) for the archived recipe
- **Rationale:** The wildcard "FEAT-08.SPEC-*" does not resolve to a spec. FEAT-08.SPEC-002 lists FEAT-19.SPEC-002 as an entry ("taps a recipe name in a past week"), so the source is pinned to that destination.

### Final Edit F22 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-19-weekly-plan-history/FEAT-19.SPEC-002-past-week-detail-view.md`
- **Section:** Navigation Out
- **Before:** | Recipe name tap | FEAT-08 Recipe detail | FEAT-08 (Recipe Library) |
- **After:** | Recipe name tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 (Recipe Library) |
- **Rationale:** Same alignment as the Interactions row.

### Final Edit F23 (Check 2 / Check 5)

- **File:** `.n2b/specifications/FEAT-19-weekly-plan-history/FEAT-19.SPEC-002-past-week-detail-view.md`
- **Section:** Connected Specs
- **Before:** | FEAT-08.SPEC-* (Recipe Library) | Navigation (outbound) | Recipe name tap opens recipe detail |
- **After:** | FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Recipe name tap opens recipe detail |
- **Rationale:** Same alignment as the Interactions row.

### Final Edit F24 (Check 5)

- **File:** `.n2b/specifications/FEAT-16-units-currency-locale-configuration/FEAT-16.SPEC-001-units-currency-settings.md`
- **Section:** Entry Points
- **Before:** | FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Guided setup reaches step 4, "Set units, currency and aisle layout," immediately after
- **After:** | FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | Guided setup reaches its units-and-currency step (wizard step 6 of 8; First Household Setup journey step 4, "Set units, currency and aisle layout"), immediately after
- **Rationale:** The guided-setup order is now FEAT-01.SPEC-003 (Step 1 of 8) → 004 (2) → 008 (5) → FEAT-16.SPEC-001 → FEAT-16.SPEC-002 → FEAT-01.SPEC-009 (Step 8 of 8), and FEAT-01.SPEC-009 calls FEAT-16.SPEC-002 "guided setup's step 7". A bare "step 4" read as a wizard position that clashes with FEAT-01.SPEC-004's "Step 2 of 8"/FEAT-01.SPEC-008's "Step 5 of 8". Both numberings are now named.

### Final Edit F25 (Check 5)

- **File:** `.n2b/specifications/FEAT-16-units-currency-locale-configuration/FEAT-16.SPEC-001-units-currency-settings.md`
- **Section:** Entry Points
- **Before:** | FEAT-01.SPEC-010 (Household Settings Hub) | Organiser (or a view-only role) taps the "Units, currency & aisles" row |
- **After:** | FEAT-01.SPEC-010 (Household Settings Hub) | Organiser taps the "Units, currency & aisles" row |
- **Rationale:** Rule 2: FEAT-01.SPEC-010 declares the "Units, currency & aisles" row organiser-only (Layout, Interactions). This spec's Access table for Sam's view-only mode is unchanged, since he can still open it by direct link.

### Final Edit F26 (Check 5)

- **File:** `.n2b/specifications/FEAT-16-units-currency-locale-configuration/FEAT-16.SPEC-001-units-currency-settings.md`
- **Section:** Connected Specs
- **Before:** Guided setup hands off to this screen as step 4, immediately after
- **After:** Guided setup hands off to this screen as wizard step 6 of 8 (journey step 4), immediately after
- **Rationale:** Same step-numbering alignment as the Entry Points row.

### Final Edit F27 (Check 5)

- **File:** `.n2b/specifications/FEAT-16-units-currency-locale-configuration/FEAT-16.SPEC-002-aisle-name-customization.md`
- **Section:** Entry Points
- **Before:** or guided setup advances via Continue after confirming units and currency (step 4 continuation)
- **After:** or guided setup advances via Continue after confirming units and currency (wizard step 7 of 8; journey step 4 continuation)
- **Rationale:** Same step-numbering alignment. FEAT-01.SPEC-009 names this screen as "guided setup's step 7".

### Final Edit F28 (Check 5)

- **File:** `.n2b/specifications/FEAT-16-units-currency-locale-configuration/FEAT-16.SPEC-002-aisle-name-customization.md`
- **Section:** Entry Points
- **Before:** | FEAT-01.SPEC-010 (Household Settings Hub) | Organiser (or a view-only role) navigates to units/currency settings and then to this sub-section |
- **After:** | FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens the hub's "Units, currency & aisles" row, then taps "Customize aisle names" on FEAT-16.SPEC-001 |
- **Rationale:** Rule 2: the hub row is organiser-only and lands on FEAT-16.SPEC-001, so the trigger now names that indirect path.

### Final Edit F29 (Check 5)

- **File:** `.n2b/specifications/FEAT-16-units-currency-locale-configuration/FEAT-16.SPEC-002-aisle-name-customization.md`
- **Section:** Layout and Content (Header)
- **Before:** a "Finish" action instead of a back arrow, since this is the last guided-setup step.
- **After:** a "Finish" action instead of a back arrow, since this is the last configuration step of guided setup (step 7 of 8); Finish hands off to FEAT-01.SPEC-009, the Step 8 of 8 completion screen.
- **Rationale:** FEAT-01.SPEC-009 is now the terminal wizard screen ("Step 8 of 8"). "The last guided-setup step" contradicted that, so it is aligned to the defined order.

### Final Edit F30 (Check 5)

- **File:** `.n2b/specifications/FEAT-01-household-setup-member-profiles/FEAT-01.SPEC-009-setup-complete-next-steps.md`
- **Section:** Navigation Out
- **Before:** | "See plans" tap | -- | FEAT-14 (Subscription & Billing Management) |
- **After:** | "See plans" tap | FEAT-14.SPEC-001 (Plan Tier Overview) | FEAT-14 (Subscription & Billing Management) |
- **Rationale:** The destination FEAT-14.SPEC-001 names "See plans" as an entry. The feature-level destination is pinned to that spec ID.

### Final Edit F31 (Check 6)

- **File:** `.n2b/specifications/FEAT-21-family-calendar-sync/FEAT-21.SPEC-002-family-calendar-integration.md`
- **Section:** Inbound Events
- **Before:** | Household deletion cascade | FEAT-18.SPEC-008 (Household Deletion Processing) completes a household deletion --
- **After:** | Household deletion cascade | FEAT-18.SPEC-008 (Household Deletion Processing) reaches its calendar-disconnect step (Processing Logic step 5) during the household deletion cascade --
- **Rationale:** The source automation is authoritative for its own trigger. FEAT-18.SPEC-008 now sends the disconnect as step 5 of the cascade, before the Household record is removed, not after deletion completes. The inbound event now matches that.

### Final-pass observations (no action required)

- FEAT-01.SPEC-003 says the wizard shell continues through FEAT-16.SPEC-001 and FEAT-16.SPEC-002. Those two headers describe Continue/Finish actions but carry no "Step N of 8" indicator. FEAT-01.SPEC-005/006/007 also carry none, so wizard positions 3 and 4 are not tied to a named screen. No indicator anywhere conflicts with "of 8", so there is nothing to align. Adding indicators would be new content, so this is noted for Stage 4 or UI detailing.
- FEAT-01.SPEC-004 "Invite a partner" and the FEAT-09.SPEC-001 row cite First Household Setup *journey* step 5, not a wizard position. The journey numbering in the Briefs (step 4 = units, currency and aisles; step 6 = completion) is a different scale from the wizard indicator. Both are now named explicitly where FEAT-16 cites a step.
- FEAT-19.SPEC-002 "Reuse this week" → FEAT-19.SPEC-003 (pick target week) was flagged one-way by the sweep. It is outside the revised-spec scope and predates this pass. It is left for any later FEAT-19 re-spawn.
