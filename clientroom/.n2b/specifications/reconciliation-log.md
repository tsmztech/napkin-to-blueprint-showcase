---
document_type: reconciliation-log
produced_by: cross-reference-reconciler
status: final
created: 2026-09-29
specs_checked: 220
alignment_edits: 54
structural_gaps: 12
missing_specs: 0
final_pass: complete
gaps_resolved: 9
gaps_open_map_only: 3
---

# Reconciliation Log

## Scope and Method

All 220 spec files across 33 feature folders were indexed by shell (65 screen, 62 automation, 60 logic-rule, 7 integration, 26 notification), together with the 33 Feature Breakdown Briefs and the Feature Dependency Map, including its External Touchpoints table. All 14 checks were run. Checks 1, 2, 3, 5, 6, 8, 11, 12, 13 and 14 were run by script across the whole index, and every hit was then read in context. Checks 4, 7, 9 and 10 were run by script where the structure allowed it (entity lifecycle cross-matching), and otherwise by targeted reading of the entities, rules and specs involved in the Pass C findings and the script hits. The five cross-spec findings routed from Pass C were each checked against the spec text and classified.

## Check Results

| Check | Result |
|-------|--------|
| 1 -- Brief completeness | Pass. Every Brief's Spec Inventory matches its folder exactly, and every `spec_count` matches. |
| 2 -- Cross-feature references resolve | Pass: no dangling `FEAT-NN.SPEC-NNN` reference in any spec, Brief, the dependency map or the registry. Separately, 11 stale "not yet specified in this run" references now point to real specs (edits 11-15). |
| 3 -- Intra-feature references resolve | Pass. |
| 4 -- Shared entity consistency | Freelancer Account, Payment Account Connection and Invoice/Reminder Log field mismatches with the dependency map (SG-03, SG-11, SG-12). These are logged, not edited, because the map is not editable by this agent. |
| 5 -- Bidirectional navigation | 26 one-way links were resolved by adding or naming entry points, and one wrong CTA destination was corrected (edits 16-30). Two remain structural (SG-08, SG-09). Back or return-to-origin rows were treated as satisfied when the destination is already a documented origin of the source. |
| 6 -- Bidirectional triggers | FEAT-32.SPEC-002 now names FEAT-25.SPEC-005 (edit 11). Structural: FEAT-16.SPEC-007 has no inbound event for FEAT-24.SPEC-003 (SG-07), and FEAT-13.SPEC-003 does not list FEAT-23 (SG-04). |
| 7 -- Logic/Rule vs Screen | FEAT-23.SPEC-007 authorization row aligned with its own valid-state table and with XBR-23 (edit 9). |
| 8 -- Spec ID uniqueness | Pass (220 unique IDs; every file name matches its `spec_id`). |
| 9 -- Cross-feature rule consistency | XBR-23 aligned (edit 9) and XBR-33 cascade ordering aligned (edit 10). The notification-type registry is incomplete (SG-10). |
| 10 -- Entity lifecycle completeness | Pass. Every create/update/delete in the map has a covering spec. The only hits were Activity Log Entry and Notification creation, which the map attributes to the triggering features but which FEAT-13.SPEC-003 and FEAT-14.SPEC-002 actually create; these are false positives. |
| 11 -- External Touchpoints vs Integration specs | Pass in both directions. All 7 Integration specs appear in a touchpoint row, and every Integration ID cited in the table exists. |
| 12 -- Notification trigger sources | All 26 notifications cite existing sources. Three screens acknowledged their notifications only in Connected Specs; their Interactions rows now name them (edits 31-33). |
| 13 -- Degradation screen references | Pass. Every Degradation Behavior screen ID exists and is a Screen spec. |
| 14 -- Platform-parameter registry | 2 near-miss markers normalized. 15 bare "3 times over 6 hours" policy literals in 6 specs rewritten to markers. No duplicate slugs. `platform-parameters.md` written with 33 slugs and 295 marker sites. |

## Edits

### Edit 1 -- Check 14 near-miss marker
- **File:** `.n2b/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-004-storage-limit-size-ceiling-rules.md`
- **Section:** Enforced By (FEAT-06.SPEC-005 row)
- **Before:** "(restates this spec's ceiling value via the shared platform parameter)"
- **After:** "(restates this spec's ceiling value via the shared platform parameter: `deliverable-file-size-ceiling`)"
- **Rationale:** Check 14 near-miss normalization. The bare phrase had no slug; this spec is the authoritative owner of the ceiling (XBR-14, FEAT-16 authority), and its value is the `deliverable-file-size-ceiling` marker it and FEAT-06.SPEC-005 already use.

### Edit 2 -- Check 14 near-miss marker
- **File:** `.n2b/specifications/FEAT-24-data-export-account-deletion/FEAT-24.SPEC-005-legal-retention-purge.md`
- **Section:** Trigger Definition
- **Before:** "interval: platform parameter `legal-retention-purge-sweep-interval`"
- **After:** "interval: platform parameter: `legal-retention-purge-sweep-interval`"
- **Rationale:** Check 14 near-miss normalization. The colon was missing, so the strict sweep and Gate A could not see the marker.

### Edits 3-8 -- Check 14 concrete platform-wide policy numbers
- **Files and sections:**
  - Edit 3: `FEAT-03-proposal-acceptance/FEAT-03.SPEC-006-acceptance-confirmation-notification.md` (Delivery Rules: Retry on failure; AC-06)
  - Edit 4: `FEAT-03-proposal-acceptance/FEAT-03.SPEC-007-change-request-notification.md` (Delivery Rules: Retry on failure; AC-06)
  - Edit 5: `FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-006-deliverable-ready-notification.md` (Delivery Rules: Retry on failure; AC-07)
  - Edit 6: `FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-004-signed-copy-confirmation-notification.md` (Delivery Rules: Retry on failure; AC-07)
  - Edit 7: `FEAT-31-operator-support-access/FEAT-31.SPEC-006-support-request-confirmation.md` (Delivery Rules: Retry on failure and Expiry; AC-03)
  - Edit 8: `FEAT-31-operator-support-access/FEAT-31.SPEC-007-support-session-opened-notice.md` (Delivery Rules: Retry on failure and Expiry; AC-05)
- **Before:** "retried up to 3 times over 6 hours" / "up to 3 retries occur over 6 hours" / "after the final retry (6 hours)"
- **After:** "retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`" / "up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window`" / "after the final retry (the end of platform parameter: `transactional-email-retry-window`)"
- **Rationale:** Check 14 lint. A spec stating a concrete platform-wide policy number is an alignment issue. FEAT-14.SPEC-001 and FEAT-14.SPEC-003 own transactional email retry (XBR-30, FEAT-14 authority), and they plus 20 sibling notification specs already use these two markers. The six outliers were aligned to the authoritative marker form. 15 sites were changed in total.

### Edit 9 -- Pass C finding 3 (FEAT-23), XBR-23
- **File:** `.n2b/specifications/FEAT-23-subscription-plan-billing-management/FEAT-23.SPEC-007-plan-limit-access-authorization-rules.md`
- **Section:** Authorization Rules, row "Add or reactivate an active client beyond platform parameter: `free-tier-active-client-limit`"
- **Before:** "Only while status is Active or Charge failed (within the grace window) and tier is Paid"
- **After:** "Only while tier is Paid and status is Active, Charge failed (within the grace window), or Cancelled -- ends at period end (through the stored period end date, while the plan is still Paid and fully usable)"
- **Rationale:** Rule 3 and rule 4. Other content in the same Logic/Rule spec defines Paid + "Cancelled -- ends at period end" as "Paid and fully usable through the period end" (its valid-state table). FEAT-23.SPEC-006 and FEAT-23.SPEC-001 treat it the same way, and XBR-23 (authority FEAT-23) blocks growth only "when a paid plan ends". The authorization row was the single contradicting statement. FEAT-01.SPEC-008 already enforces "unlimited on a paid plan", so no change was needed there.

### Edit 10 -- Pass C finding 5 (FEAT-24 / FEAT-16), XBR-33
- **File:** `.n2b/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-006-stored-file-purge-on-account-deletion.md`
- **Sections:** Trigger Definition; Processing Logic step 6; Business Rules (4th bullet); Edge Cases (1st bullet)
- **Before:**
  - Trigger: "Fires once Nadia's account deletion has been confirmed and the cascade begins processing every data-holding feature"
  - Step 6: "...or, if this step fails, so the cascade can halt per FEAT-24's own failure handling."
  - Business rule: "FEAT-24.SPEC-004's own account-intact-on-failure guarantee depends on every cascade step, including this one, either succeeding or being retried to success..."
  - Edge case: "a transient failure at the storage layer does not by itself halt the cascade."
- **After:**
  - Trigger: "Fires once Nadia's account deletion has been confirmed and the cascade has passed its commit point (the payment-account disconnect, FEAT-24.SPEC-004 step 5), as the irreversible stored-byte purge finalization step (FEAT-24.SPEC-004 step 8); never invoked while a revert of the account is still possible"
  - Step 6: "...or, if this step fails, so FEAT-24.SPEC-004 retries this step until it succeeds per its own post-commit failure handling (the cascade is never halted or reverted at this point)."
  - Business rule: "because this step runs only after FEAT-24.SPEC-004's commit point, a failure here never reverts the account to Active; FEAT-24.SPEC-004 keeps the account's data hidden and retries this step until it succeeds before the account is considered deleted."
  - Edge case: "a failure at the storage layer never halts or reverts the cascade, which has already passed its commit point; FEAT-24.SPEC-004 retries the step until it succeeds."
- **Rationale:** Business-rule contradiction on cascade ordering. XBR-33 names FEAT-24 as the authority for account deletion, and FEAT-24.SPEC-004 (the triggering automation) owns the cascade order: a reversible hold phase, a commit point at step 5, then irreversible finalization at steps 7-9, retried and never reverted. The consuming step's description was aligned to that ordering. No new behaviour was added.

### Edit 11 -- Check 6 trigger bidirectionality (stale reference)
- **File:** `.n2b/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-002-payment-account-connection-status-reporting.md`
- **Sections:** Product Behaviors Enabled (chargeback row); Inbound Events ("Reversal or chargeback reported" row)
- **Before:** "FEAT-25 (Refund & Cancelled Project Handling -- not yet specified in this run)" (in both rows); "the notice is relayed to FEAT-25"
- **After:** "FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording), which fires FEAT-25.SPEC-008 (Payment Reversal Notification)"; the Inbound Events row now reads "relayed to FEAT-25.SPEC-005", with Affected Specs "FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording)"
- **Rationale:** Check 6. FEAT-25.SPEC-005's Trigger Definition cites this Integration spec as its external-event source, but the Inbound Events row named only the feature and marked it unspecified. The automation's declaration is authoritative, so the integration row was aligned to it.

### Edit 12 -- Check 2 stale references (FEAT-14 notification registry)
- **File:** `.n2b/specifications/FEAT-14-notifications-email/FEAT-14.SPEC-004-notification-type-recipient-entitlement-rules.md`
- **Sections:** Scope and Non-Goals; Enforced By; notification-type registry rows (contact_invitation, welcome_email, support_session_notice, chargeback_notice); Business Rules (last bullet)
- **Before:** "owned by FEAT-21 (Settings & Account Management), not yet specified in this run"; "| FEAT-21 (Settings & Account Management) | Notification Preferences (not yet specified in this run) |"; "FEAT-18 (not yet specified in this run)"; "FEAT-20 (not yet specified in this run)"; "FEAT-31 (not yet specified in this run)"; "FEAT-25 / FEAT-32 (not yet specified in this run)"; "FEAT-21's (not yet specified) notification-preferences screen"
- **After:** "owned by FEAT-21 (Settings & Account Management) as FEAT-21.SPEC-002 (Notification Preferences)"; "| FEAT-21.SPEC-002 | Notification Preferences (FEAT-21) |"; "FEAT-18 (FEAT-18.SPEC-010)"; "FEAT-20 (FEAT-20.SPEC-006)"; "FEAT-31 (FEAT-31.SPEC-007)"; "FEAT-25 (FEAT-25.SPEC-008), from the reversal notice FEAT-32.SPEC-002 relays"; "FEAT-21's notification-preferences screen (FEAT-21.SPEC-002)"
- **Rationale:** Checks 2 and 12. Those specs now exist and each declares itself the source of the named type, so the declaring specs are authoritative and the registry was aligned to them. Registry rows missing for other notification specs are logged as SG-10, since adding rows is new content.

### Edit 13 -- Check 2 stale reference
- **File:** `.n2b/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-005-payment-connection-authorization-validation-rules.md`
- **Section:** Enforced By
- **Before:** "FEAT-31.SPEC-002 (Operator Support Access -- not yet specified in this run)"
- **After:** "FEAT-31.SPEC-002 (Operator Support Session Console, FEAT-31)"
- **Rationale:** The referenced spec exists, and its own name replaces the stale marker.

### Edit 14 -- Check 2 / Check 6 stale reference
- **File:** `.n2b/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-004-disconnect-payment-account.md`
- **Section:** Trigger Definition (account-deletion row)
- **Before:** "FEAT-24.SPEC-004 (Data Export & Account Deletion -- not yet specified in this run) | Fires when FEAT-24's own warn-then-remove sequence reaches the payment-account step (XBR-33);"
- **After:** "FEAT-24.SPEC-004 (Account Deletion Processing, FEAT-24) | Fires when FEAT-24's own warn-then-remove sequence reaches the payment-account step, which is that cascade's commit point (FEAT-24.SPEC-004 step 5; XBR-33);"
- **Rationale:** The triggering automation exists and is authoritative for cascade order (XBR-33), so the trigger description was aligned to its step 5 commit point.

### Edit 15 -- Check 2 / Check 5 stale navigation destination
- **File:** `.n2b/specifications/FEAT-32-payment-account-connection/FEAT-32.SPEC-001-payment-connection-screen.md`
- **Sections:** Interactions ("Contact support" link); Navigation Out; Connected Specs
- **Before:** "Navigate to FEAT-31 (Operator Support Access -- not yet specified in this run)"; "| FEAT-31 (support request entry -- not yet specified in this run) |"; "| FEAT-31 (Operator Support Access) | Navigation (outbound) |"
- **After:** "Navigate to FEAT-31.SPEC-001 (Contact Support Screen)" in all three places
- **Rationale:** FEAT-31.SPEC-001 is the freelancer's support-request entry screen. The outbound declaration was made precise, and its destination entry point was added in edit 16.

### Edit 16 -- Check 5 entry point
- **File:** `.n2b/specifications/FEAT-31-operator-support-access/FEAT-31.SPEC-001-contact-support-screen.md`
- **Section:** Entry Points
- **Before:** Only the row "Notifications & Help (product-wide entry)"
- **After:** A row was added: "FEAT-32.SPEC-001 (Payment Connection Screen) | Nadia taps "Contact support" from the Needs attention state | None -- form starts empty"
- **Rationale:** Rule 2. The outbound navigation declared by FEAT-32.SPEC-001 is authoritative.

### Edit 17 -- Check 5 CTA destination (FEAT-02.SPEC-011)
- **Files:** `FEAT-02-proposal-creation-sending/FEAT-02.SPEC-011-proposal-sent-resent-email.md` (Content Definition: three CTA lines; Connected Specs) and `FEAT-03-proposal-acceptance/FEAT-03.SPEC-001-proposal-review-accept.md` (Entry Points)
- **Before:** "deep-links to FEAT-02.SPEC-003 (Proposal Detail) [rendered as Owen's own portal proposal view, owned by FEAT-03]", with the Connected Specs row pointing to FEAT-02.SPEC-003; the FEAT-03.SPEC-001 entry read "External -- emailed proposal link (FEAT-02)"
- **After:** "deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's own portal proposal view, owned by FEAT-03]", with the Connected Specs row pointing to FEAT-03.SPEC-001; the entry now reads "External -- emailed proposal link (FEAT-02.SPEC-011, Proposal Sent/Resent Email)"
- **Rationale:** Rule 2 and data consistency. The notification's own bracketed text names Owen's portal proposal view (FEAT-03), and FEAT-02.SPEC-003's Access table denies Owen access ("Owen's equivalent is his own portal proposal view (FEAT-03), not this screen"). The ID was aligned to the destination the declaration itself describes, and that destination already listed the email as an entry.

### Edits 18-30 -- Check 5 entry points for declared outbound navigation and notification CTAs
Each edit adds or names the source in the destination's **Entry Points** table. Rule 2: the spec declaring outbound navigation or a CTA deep link is authoritative.

| # | File (destination) | Before | After (row added or changed) |
|---|--------------------|--------|------------------------------|
| 18 | `FEAT-03-proposal-acceptance/FEAT-03.SPEC-001-proposal-review-accept.md` | No row for the acceptance email | Added "FEAT-03.SPEC-006 (Acceptance Confirmation Notification) \| Owen taps the confirmation email's "View proposal" CTA" |
| 19 | `FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-001-deliverable-comment-thread.md` | No rows for the deliverable-ready or reply emails | Added FEAT-06.SPEC-006 ("Review deliverable" CTA) and FEAT-07.SPEC-004 ("Open thread" CTA) |
| 20 | `FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-002-milestone-comment-thread.md` | No row for the reply email | Added FEAT-07.SPEC-004 ("Open thread" CTA, milestone reply) |
| 21 | `FEAT-08-milestone-approval/FEAT-08.SPEC-001-milestone-review-approval-screen.md` | No rows for the approval email or the client milestone timeline | Added FEAT-08.SPEC-007 ("View milestone" CTA) and FEAT-04.SPEC-002 (Owen taps a ready milestone row) |
| 22 | `FEAT-10-invoice-payment-processing/FEAT-10.SPEC-001-pay-invoice-screen.md` | "FEAT-11 (Automated Payment Reminders) -- reminder email link"; no payment-confirmation row | "FEAT-11.SPEC-004 (Overdue Reminder Email, FEAT-11 Automated Payment Reminders) -- reminder email link"; added FEAT-10.SPEC-007 ("View invoice" CTA) |
| 23 | `FEAT-18-client-contact-management-roles/FEAT-18.SPEC-001-client-contact-list.md` | No row for the colleague alert | Added FEAT-18.SPEC-011 ("View contacts" CTA) |
| 24 | `FEAT-21-settings-account-management/FEAT-21.SPEC-003-login-security.md` | No row for the account-change email | Added FEAT-21.SPEC-011 ("Go to Login & Security" CTA) |
| 25 | `FEAT-24-data-export-account-deletion/FEAT-24.SPEC-001-data-export-screen.md` | No row for the export-ready email | Added FEAT-24.SPEC-008 ("Download your export" CTA) |
| 26 | `FEAT-09-invoice-generation-sending/FEAT-09.SPEC-002-invoice-detail.md` | No rows for the refund, reversal, trail or search sources | Added FEAT-25.SPEC-007 and FEAT-25.SPEC-008 ("View invoice" CTAs), FEAT-13.SPEC-001 (invoice trail entry link) and FEAT-28.SPEC-001 (Invoice search result) |
| 27 | `FEAT-26-legally-binding-e-signature-for-proposals/FEAT-26.SPEC-001-signature-signing-step.md` | No row for the signed-copy email | Added FEAT-26.SPEC-004 ("View signed proposal" CTA) |
| 28 | `FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-001-activity-trail.md` | Only Dana's support-session entry (feature-level FEAT-31) | Added FEAT-31.SPEC-007 (Nadia taps "View activity trail" CTA) |
| 29 | `FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-002-deliverable-list-management.md` | No rows for the milestone timeline or global search | Added FEAT-04.SPEC-002 (Nadia taps a not-yet-ready milestone row; restricted to Nadia because this screen's Access table denies Owen and Priya, see SG-09) and FEAT-28.SPEC-001 (Deliverable search result) |
| 30 | `FEAT-05-client-portal-access-magic-link-login/FEAT-05.SPEC-002-link-verification-landing.md` | "...invitation email's embedded link from FEAT-02, FEAT-06, or FEAT-18" | "...from FEAT-02 (FEAT-02.SPEC-011), FEAT-06 (FEAT-06.SPEC-006), or FEAT-18 (FEAT-18.SPEC-010, New Contact Invitation Email)" |

### Edits 31-33 -- Check 12 source acknowledgment in Interactions
| # | File | Section | Before | After |
|---|------|---------|--------|-------|
| 31 | `FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-001-deliverable-comment-thread.md` | Interactions -- Post button | "2. If valid and online, creates the Comment pinned to this Deliverable Version." | "...; the successful write fires FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) when the author is Owen or Priya, or FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) when the author is Nadia." |
| 32 | `FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-002-milestone-comment-thread.md` | Interactions -- Post button | "2. If valid and online, creates the Comment pinned to this Milestone." | Same addition as edit 31 |
| 33 | `FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-001-sign-up-account-creation.md` | Interactions -- "Create my account" | "2. If valid, create the Freelancer Account and transition it to Active." | "...(which fires FEAT-20.SPEC-006, Welcome Email, independently of this screen's navigation)." |

- **Rationale:** Check 12. Each Notification spec's Trigger cites these screens as its source. The screens already acknowledged this in Connected Specs and Business Rules; the Interactions rows now reference the Notification spec ID too, with no behaviour change.

## Gaps Returned to the Orchestrator

These are not edited: each needs new content, a redesign, or a change to a document this agent may not modify.

- **SG-01 [STRUCTURAL-GAP]** FEAT-07.SPEC-001 and FEAT-07.SPEC-002 (target) vs FEAT-07.SPEC-008 (source). **Missing:** a "Sync Failed" state for a queued offline comment, and a Discard control (with confirmation) for a queued comment, on both thread screens. **Evidence:** FEAT-07.SPEC-008 step 9 and Outcome Definitions ("the entry remains in the queue as Sync Failed"; "The triggering screen shows the entry with an error and offers the author a chance to edit or discard it"), its edge case "The author discards a queued comment before it syncs", and AC-10 ("when she confirms the discard"). The States and Interactions sections of SPEC-001 and SPEC-002 define neither (grep: no "Sync Failed" or queued-item "Discard" in either screen).
- **SG-02 [STRUCTURAL-GAP]** FEAT-20.SPEC-002, FEAT-20.SPEC-003 and FEAT-20.SPEC-005 vs FEAT-20.SPEC-004. **Missing:** a single writer for `onboarding_how_did_you_hear_resolved`. SPEC-004 should be the only setter (compare-and-set), and callers should only trigger it. **Evidence:** SPEC-002 Interactions rows "How did you hear" Continue and Skip ("Sets onboarding_how_did_you_hear_resolved to true, triggers FEAT-20.SPEC-004") and AC-02. SPEC-003 step 6, its Outcome row and AC-14 ("set it true and trigger FEAT-20.SPEC-004"). SPEC-004 step 1 ("if onboarding_how_did_you_hear_resolved is already true, stop -- the hand-off has already fired"). As written, the FEAT-33 referral hand-off can never fire. SPEC-005's Enforced By row and AC-17 also assign the write to SPEC-002. The fix changes Processing Logic and ACs across four specs, which is a redesign rather than an alignment.
- **SG-03 [STRUCTURAL-GAP]** feature-dependency-map.md, Shared Data Entities > Freelancer Account, vs FEAT-20.SPEC-001, FEAT-20.SPEC-002 and FEAT-20.SPEC-005. **Missing:** the map's field list lacks the onboarding-progress fields `onboarding_referring_portal_ref`, `onboarding_ready_acknowledged`, `onboarding_welcome_warning_dismissed` and `onboarding_how_did_you_hear_resolved` (and onboarding status/current step). **Evidence:** FEAT-20.SPEC-005 Governed Entity table (lines 56-59) and Defaults table (lines 117-120); the map's Freelancer Account field list (name/email, business fields, payment terms, time zone, notification preferences, devices, help-tip dismissals only). The Reconciler may not modify the map.
- **SG-04 [STRUCTURAL-GAP]** FEAT-13.SPEC-003 (Activity Entry Recording) vs FEAT-23.SPEC-002, FEAT-23.SPEC-004, FEAT-23.SPEC-005 and FEAT-23.SPEC-006. **Missing:** Trigger Definition rows for the subscription-plan events FEAT-23 reports (upgrade, downgrade, cancellation, lapse, charge failure). **Evidence:** FEAT-23.SPEC-006 step 6 ("report the cancellation as a record-worthy event ... to FEAT-13.SPEC-003"), and the other three FEAT-23 automations also cite FEAT-13.SPEC-003. The FEAT-13.SPEC-003 trigger table lists sources from FEAT-03, 05, 06, 08, 09, 10, 11, 18, 25 and 31 only, with no FEAT-23 row.
- **SG-05 [STRUCTURAL-GAP]** FEAT-23 feature-overview.md (Brief; Feature Analyst re-spawn) vs FEAT-23.SPEC-004 and FEAT-23.SPEC-007. **Missing:** the Brief should use the specs' state model. **Evidence:** Entity-Lifecycle State Transition row "Paid -> Downgraded on accepted downgrade" (line 80) and user-flow line "[tier set to Downgraded]" (line 133). FEAT-23.SPEC-007 defines tier as Free or Paid only, and an accepted downgrade results in Free / Active. The Reconciler may not modify Briefs.
- **SG-06 [STRUCTURAL-GAP]** FEAT-32.SPEC-001, FEAT-32.SPEC-003 and FEAT-32.SPEC-005. **Missing:** an exit path from the Connecting state when the payment processor never reports an outcome (for example a timeout or a Nadia-initiated cancel of the pending hand-off), with matching ACs in SPEC-001 and SPEC-003 and the authorization in SPEC-005. **Evidence:** SPEC-001 Connecting state: "No action buttons are shown"; the only exits are processor ready, restricted, or failed/abandoned events via SPEC-003 steps 5 and 70. SPEC-005 hides Connect, Reconnect and Disconnect "while Connecting" (lines 88, 95, 97). The slow-threshold note (`payment-connection-handoff-slow-threshold`) informs but offers no action.
- **SG-07 [STRUCTURAL-GAP]** FEAT-16.SPEC-007 (Integration, Inbound Events) vs FEAT-24.SPEC-003. **Missing:** an inbound event for "download delivery of an archive completed" whose Affected Specs names FEAT-24.SPEC-003. **Evidence:** FEAT-24.SPEC-003 Trigger Definition: "Archive successfully delivered | FEAT-16.SPEC-007 | Fires when the storage & delivery capability reports the download transfer for this archive completed". FEAT-16.SPEC-007 Inbound Events define only upload "Transfer completed" (FEAT-16.SPEC-002) and delivery byte-range, paused and failed events (FEAT-16.SPEC-003), with no delivery-completed event and no FEAT-24 reference.
- **SG-08 [STRUCTURAL-GAP]** FEAT-21 settings navigation (FEAT-21.SPEC-001 to SPEC-004, Navigation Out) vs FEAT-32.SPEC-001 (Entry Points and Navigation Out). **Missing:** a "Payment account" navigation-shell entry into FEAT-32.SPEC-001, and a concrete Back destination for FEAT-32.SPEC-001. **Evidence:** FEAT-32.SPEC-001 Entry Points: "FEAT-21 (Settings & Account Management -- not yet specified in this run) | Nadia opens "Payment account" from Settings". Its Navigation Out has "Back arrow tap | FEAT-21 (Settings home -- not yet specified in this run)". No FEAT-21 screen's Navigation Out mentions FEAT-32 or a payment account (grep). The stale wording was left in place because there is no destination to align it to.
- **SG-09 [STRUCTURAL-GAP]** FEAT-04.SPEC-002 (Milestone Timeline (Client View), Navigation Out) vs FEAT-06.SPEC-002 (Access and Visibility). **Missing:** a destination for Owen and Priya when they tap a milestone whose deliverables are not yet ready for approval. It should presumably be FEAT-07.SPEC-001 or FEAT-07.SPEC-002. **Evidence:** FEAT-04.SPEC-002 row "Milestone row tap (deliverables not yet ready for approval) | FEAT-06.SPEC-002", on a screen whose Access table lets Owen "navigate into a milestone's deliverables". FEAT-06.SPEC-002 Access: "Owen ... No | No | This screen is never part of the client portal's navigation ... the equivalent client-facing view is FEAT-07". The entry point added in edit 29 was restricted to Nadia.
- **SG-10 [STRUCTURAL-GAP]** FEAT-14.SPEC-004 (notification-type registry) vs 10 notification specs. **Missing:** registry rows (notification_type, recipient, Transactional/Optional classification) for FEAT-18.SPEC-011, FEAT-21.SPEC-011, FEAT-23.SPEC-008, FEAT-24.SPEC-008, FEAT-24.SPEC-009, FEAT-25.SPEC-007, FEAT-26.SPEC-004, FEAT-27.SPEC-004, FEAT-31.SPEC-006 and FEAT-32.SPEC-006. **Evidence:** the registry has 16 rows covering 16 of the 26 Notification specs. Its Business Rules state "Each notification_type maps to exactly one triggering event" and the registry is "the single source of truth" FEAT-21.SPEC-002 lists from, so unregistered types have no classification.
- **SG-11 [STRUCTURAL-GAP]** feature-dependency-map.md, Shared Data Entities > Payment Account Connection, vs FEAT-32.SPEC-005. **Missing:** the interim `Connecting` status value and the `handoff_started_at` field. **Evidence:** the map's status list is "Not connected, Connected, Needs attention, Disconnected". FEAT-32.SPEC-005 Governed Entity (lines 51, 53) adds Connecting (persisted) and `handoff_started_at`. FEAT-32 is the creating feature and is authoritative, but the Reconciler may not modify the map.
- **SG-12 [STRUCTURAL-GAP]** feature-dependency-map.md (Invoice field `reminder_paused -- (FEAT-11)`) and FEAT-09.SPEC-006 (Governed Entity row `reminder_paused`) vs FEAT-11.SPEC-003 and FEAT-11.SPEC-005 (writers of Reminder Log `pause_state`). **Missing:** a single agreed home for the per-invoice reminder pause. No spec writes `Invoice.reminder_paused`; FEAT-11 writes `Reminder Log.pause_state`. **Evidence:** FEAT-11.SPEC-003 Data Model "Updates: Reminder Log -- `pause_state`"; FEAT-11 Brief's own "Flagged discrepancy" note (line 83); FEAT-09.SPEC-006 line 55 governs visibility of a field that no spec sets.

**[MISSING-SPEC] findings:** none. Every referenced automation, screen, rule, integration and notification exists.

## Pass C Routed Findings -- Disposition

| # | Finding | Classification | Where |
|---|---------|----------------|-------|
| 1 | FEAT-07 Sync Failed state and Discard control | [STRUCTURAL-GAP] | SG-01 |
| 2 | FEAT-20 flag ordering blocks the FEAT-33 hand-off, and new account fields are absent from the map | [STRUCTURAL-GAP] (two gaps) | SG-02, SG-03 |
| 3 | FEAT-23.SPEC-007 authorization excludes Cancelled--ends at period end | [ALIGNMENT], resolved | Edit 9 |
| 3 | FEAT-13.SPEC-003 omits FEAT-23 as a writer | [STRUCTURAL-GAP] | SG-04 |
| 3 | FEAT-23 Brief "Downgraded" wording | [STRUCTURAL-GAP] (Brief; Feature Analyst) | SG-05 |
| 4 | FEAT-32 Connecting state has no exit if the processor never reports | [STRUCTURAL-GAP] | SG-06 |
| 5 | FEAT-16.SPEC-006 "cascade halt" wording vs reordered FEAT-24.SPEC-004 | [ALIGNMENT], resolved | Edit 10 |

## Observations Not Acted On

- A "Still working -- this is taking longer than usual" note after **10 seconds** appears consistently in FEAT-10.SPEC-003, FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-26.SPEC-001 and FEAT-26.SPEC-005. It is treated as a uniform interface-feedback timing, not a business policy value, so no marker was minted. A downstream designer may still choose to parameterize it.
- Day-3 and day-10 reminder timing, the 1-2,000-character comment length, and the "roughly 2 seconds" responsiveness target are product rules or targets carried from BRIEF.md, XBR-15 and ASMP-21. They are not platform-set policy values and were left as written.

---

## Final Reconciliation Pass (after gap routing) -- 2026-09-29

This pass ran after the 12 structural gaps above were routed to the Spec Writers and the Feature Analyst and their fixes landed. It is alignment only: no re-spawns were requested. Every file changed by the gap fixes was checked against its counterparts. The checks covered bidirectional navigation and triggers, dangling references, frontmatter `acceptance_criteria_count` against the actual ACs, and Coverage Summary counts. The whole-index checks were then run again.

### Re-run Check Results

| Check | Result |
|-------|--------|
| 1 -- Brief completeness | Pass. There are still 220 spec files, and the FEAT-23 Brief now uses the five plan states (SG-05). |
| 2 / 3 -- References resolve | Pass (script): no `FEAT-NN.SPEC-NNN` reference in any spec, the dependency map or the registry points to a missing file. |
| 4 -- Shared entity consistency | Freelancer Account (SG-03), Payment Account Connection (SG-11) and Invoice `reminder_paused` (SG-12, map side) still differ between the dependency map and the specs. The map is outside this agent's edit authority, so these are logged below with evidence. |
| 5 -- Bidirectional navigation | Four one-way or imprecise links left by the gap fixes were aligned (edits 42, 43, 48-50). |
| 6 -- Bidirectional triggers | FEAT-07.SPEC-001/002 Retry and edit-Save now appear as a trigger source in FEAT-07.SPEC-008 (edit 51). The "Start over" trigger from FEAT-32.SPEC-001 to FEAT-32.SPEC-003 is acknowledged in both specs (edit 45). The FEAT-16.SPEC-007 event "Archive download delivered" and the FEAT-24.SPEC-003 trigger now match in both directions (edit 47). FEAT-13.SPEC-003 lists all four FEAT-23 sources, and each FEAT-23 automation cites it back. |
| 7 -- Logic/Rule vs Screen | FEAT-20.SPEC-005 now matches the redesigned FEAT-20.SPEC-002 hand-off ordering (edits 34-36). FEAT-13.SPEC-004 now matches FEAT-13.SPEC-003 (edits 38-41). |
| 8 -- Spec ID uniqueness | Pass (220 unique IDs). |
| 9 -- Cross-feature rules | XBR-19 and XBR-29 wording is consistent across the FEAT-32 exit path. The notification registry has 29 rows covering 26 of 26 Notification specs, and FEAT-14.SPEC-002 references all 26 (SG-10). |
| 10 -- Lifecycle completeness | Pass. |
| 11 -- External Touchpoints | Pass in both directions (7 Integration specs). |
| 12 -- Notification trigger sources | Pass. |
| 13 -- Degradation screen references | Pass. |
| 14 -- Platform parameters | One new slug, `payment-connect-handoff-timeout` (19 sites in FEAT-32.SPEC-001, SPEC-003 and SPEC-005), was added to the registry (edit 46). The registry now has 34 rows and 314 marker sites. It matches the shell sweep exactly: every slug is present, each Referenced-by list equals the set of spec files carrying that slug, the rows are alphabetical, and every row is `decide-before-build`. The near-miss sweep found 0 hits: no capitalised phrase, no marker missing its colon or backticks, no uppercase or underscore slug, no marker wrapped across a line. One bare "fixed platform-wide" phrase was found and rewritten (edit 37). |
| AC counts | Pass. Every spec's frontmatter `acceptance_criteria_count` equals its AC count. |
| Template residue | 10 lines that were only a brace-wrapped template instruction were unwrapped (edit 54). The remaining brace-only lines are email body placeholders such as `{deposit_invoice_line}`, each defined in its spec's Placeholders table, so they are content rather than instructions. |

### Final-Pass Edits

#### Edits 34-36 -- SG-02 open point (FEAT-20 "How did you hear" ordering)
- **Files:** `FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-005-onboarding-step-sequencing-exit-criteria-rules.md` (Cross-Field Rules "One-time question" row (edit 34); Business Rules R-06 (edit 35)); `FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-002-onboarding-guided-sequence.md` (States, Welcome row entry condition (edit 36))
- **Before:** "While onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress, onboarding_current_step stays "How did you hear" and the question is shown again on every return; once true, the step is never re-entered". R-06: "It is shown on every open of FEAT-20.SPEC-002 while onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress". Welcome entry: "onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress".
- **After:** onboarding_current_step stays "How did you hear" until Nadia answers or skips. On answer or skip, FEAT-20.SPEC-002 advances the step at once and triggers FEAT-20.SPEC-004 without waiting for it, and the flag becomes true when that hand-off attempt finishes (or at R-09). The question is shown only while onboarding_current_step is "How did you hear", the flag is false and the status is In Progress, so it is never shown again while the hand-off is still running. The Welcome entry condition carries the same three-part test.
- **Rationale:** Rule 4, plus the gap re-spawn's authoritative design. The re-spawned FEAT-20.SPEC-002 and FEAT-20.SPEC-004 make FEAT-20.SPEC-004 the only compare-and-set writer, and FEAT-20.SPEC-002 advances without waiting. The rule's old wording tied the step to the flag, which contradicted that ordering during the hand-off. The rule's intent is unchanged: the question is shown until answered or skipped and never again after that.

#### Edit 37 -- Check 14 lint ("fixed platform-wide" without a marker)
- **File:** `FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-005-onboarding-step-sequencing-exit-criteria-rules.md` (Business Rules R-07)
- **Before:** "The five-step sequence and its mandatory/optional split are fixed platform-wide;"
- **After:** "The five-step sequence and its mandatory/optional split are the same for every account;"
- **Rationale:** Check 14 lint. The sentence describes a product-structure rule (SC-11), not a policy value, so no slug was minted. The phrase was reworded so it no longer reads as an unmarked platform parameter.

#### Edits 38-41 -- SG-04 follow-up (FEAT-13.SPEC-004 aligned to FEAT-13.SPEC-003)
- **File:** `FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-004-entry-immutability-content-attribution-rules.md`
- **Edit 38 (Business Rules, closed vocabulary):** **Before:** "...manual payment recorded, support session opened, support session closed. No other `event_type`..." **After:** the list also includes "the four FEAT-23 plan event types: plan created, plan tier/status changed, downgrade offer raised, plan cancellation recorded".
- **Edit 39 (Field Validation "project" row; Cross-Field "Project required unless account-level"; Edge Case "project is omitted"):** **Before:** only "support session opened" and "support session closed" were exempt from the project requirement. **After:** the four plan event types are exempt too, because they are account-level (FEAT-13.SPEC-003).
- **Edit 40 (Governed Entity `affected_record`; Cross-Field "Affected-record type must match event type"):** **Before:** the examples listed proposal, milestone, invoice and deliverable only. **After:** "or the freelancer's Subscription Plan for a plan event", and the mapping states that the four plan event types must reference the freelancer's Subscription Plan.
- **Edit 41 (Authorization "Create entry" row; Business Rules):** **Before:** "one of the ten writer features". **After:** "one of the eleven writer features listed in FEAT-13.SPEC-003".
- **Rationale:** Business-rule consistency within the feature (Check 7 and Check 9). The re-spawned FEAT-13.SPEC-003 classifies FEAT-23's four plan events against "the fixed vocabulary defined by FEAT-13.SPEC-004", records them without a project, and names eleven writers. Left as it was, FEAT-13.SPEC-004 would have rejected every plan entry as "unrecognized event type" or "project reference required". The Logic/Rule spec was aligned to the event set that its own enforcer and the four FEAT-23 automations already define. No new behaviour was added.

#### Edits 42-45 -- SG-08 and SG-06 follow-ups (FEAT-32.SPEC-001)
- **File:** `FEAT-32-payment-account-connection/FEAT-32.SPEC-001-payment-connection-screen.md`
- **Edit 42 (Layout header; Interactions "Back arrow"; Connected Specs FEAT-21 row):** **Before:** "a back arrow returning to Settings & Account Management (FEAT-21)"; "Navigate to FEAT-21 (Settings & Account Management)"; "| FEAT-21 (Settings & Account Management) | Navigation (inbound/outbound) | Entry point via Settings; back arrow returns there |". **After:** "...returning to FEAT-21.SPEC-001 (Account Profile, the Settings default entry of Settings & Account Management)"; "Navigate to FEAT-21.SPEC-001 (Account Profile, the Settings default entry)"; "| FEAT-21.SPEC-001 / SPEC-002 / SPEC-003 / SPEC-004 (Settings & Account Management) | ... | Entry point via the Settings "Payment account" navigation item; back arrow returns to FEAT-21.SPEC-001 |".
- **Edit 43 (Entry Points and Connected Specs, FEAT-20 row):** **Before:** "FEAT-20 (Onboarding / First-Run Setup) | Nadia reaches the optional "Connect payments" onboarding step". **After:** "FEAT-20.SPEC-002 (Onboarding Guided Sequence, FEAT-20 Onboarding / First-Run Setup) | Nadia taps "Connect payments" on the optional onboarding step", and the Connected Specs row names FEAT-20.SPEC-002.
- **Edit 44 (Access and Visibility, Nadia row; Layout "Connecting"):** **Before:** "Can Act: Connect, Reconnect, Disconnect (her own account only)"; "No action buttons are shown." **After:** "Connect, Reconnect, Disconnect, and Start over on a stalled hand-off (her own account only, per FEAT-32.SPEC-005)"; "Until the start-over threshold below, no action buttons are shown."
- **Edit 45 (Connected Specs, FEAT-32.SPEC-003 row):** **Before:** "References (inbound) | Supplies the current status ... after a failed attempt". **After:** "References (inbound); Triggers (outbound) | ...; a confirmed "Start over" on a stalled hand-off triggers its abandoned-hand-off handling".
- **Rationale:** Rule 2. FEAT-21.SPEC-001's Navigation Out and AC-17, and FEAT-20.SPEC-002's "Connect payments" row, are the authoritative outbound declarations, and the destination now names them by spec ID. Rule 4: FEAT-32.SPEC-005 authorizes Start over for Nadia, so the screen's access row and its "no action buttons" sentence were aligned to it. Check 6: FEAT-32.SPEC-003 lists FEAT-32.SPEC-001 as a trigger source, and the screen now acknowledges that trigger.

#### Edit 46 -- Check 14 registry (SG-06)
- **File:** `.n2b/specifications/platform-parameters.md`
- **Section:** Registry (new row between `paid-tier-storage-allowance` and `payment-connection-handoff-slow-threshold`); frontmatter `parameter_count` 33 -> 34, `marker_site_count` 295 -> 314; intro count 33 -> 34
- **Before:** no row for `payment-connect-handoff-timeout`.
- **After:** `payment-connect-handoff-timeout` | FEAT-32.SPEC-001, FEAT-32.SPEC-003, FEAT-32.SPEC-005 | proposed 30 minutes (non-binding) | `decide-before-build`.
- **Rationale:** Check 14. This is a distinct parameter from `payment-connection-handoff-slow-threshold`: the slow threshold only adds a note, while this one unlocks "Start over". They are not duplicate slugs, so both are kept. The proposed default sits well above the 5-minute slow-note proposal, which is itself grounded in the FEAT-20 success metric.

#### Edit 47 -- SG-07 reciprocal (FEAT-24.SPEC-003)
- **File:** `FEAT-24-data-export-account-deletion/FEAT-24.SPEC-003-data-export-archive-generation.md` (Connected Specs, FEAT-16.SPEC-007 row)
- **Before:** "Triggers (outbound) | Performs the actual storage, delivery, and purge of the archive file"
- **After:** "Triggers (outbound) / Triggered by (inbound) | ...; its "Archive download delivered" inbound event triggers the Ready -> Downloaded transition"
- **Rationale:** Check 6. The Trigger Definition already cited FEAT-16.SPEC-007, which now emits that event. The Connected Specs row was aligned to show the inbound direction.

#### Edits 48-50 -- SG-09 reciprocals (FEAT-04.SPEC-002 destinations)
- **Edit 48:** `FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-002-milestone-comment-thread.md`. In Entry Points, a row was added: "FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | Owen or Priya taps a milestone row that has no deliverable ready for review yet | The Milestone reference". A matching Connected Specs row (Navigation (inbound)) was also added. **Before:** no FEAT-04.SPEC-002 row in either table.
- **Edit 49:** `FEAT-08-milestone-approval/FEAT-08.SPEC-001-milestone-review-approval-screen.md`, Entry Points. **Before:** "Owen taps a milestone row whose deliverable is ready for review". **After:** "Owen or Priya taps a milestone row whose deliverable is ready for review (Priya sees no Approve control)".
- **Edit 50:** `FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-002-deliverable-list-management.md`, Entry Points. **Before:** "Nadia taps a milestone row while its deliverables are not yet ready for approval" (the edit-29 restriction). **After:** "Nadia or Dana taps a milestone row (any milestone status; Dana's rendering is read-only inside her support session)".
- **Rationale:** Rule 2. The re-spawned FEAT-04.SPEC-002 Navigation Out (rows for Nadia/Dana to FEAT-06.SPEC-002, and for Owen/Priya to FEAT-07.SPEC-002 or FEAT-08.SPEC-001) is authoritative, and each destination's entry point now matches it.

#### Edits 51-52 -- SG-01 reciprocal (FEAT-07.SPEC-008)
- **File:** `FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-008-offline-comment-queue-sync.md`
- **Edit 51 (Trigger Definition; Connected Specs FEAT-07.SPEC-001 row; Coverage Summary Trigger Paths):** A trigger row was added: "Author retries or re-saves an unsent entry | FEAT-07.SPEC-001 or FEAT-07.SPEC-002 | Fires when the author taps Retry on a Sync Failed entry (repeated server error) or saves a valid edit to a validation-failed entry while online; the entry is processed from step 4 exactly like a reconnect sync". The Connected Specs description now names the Retry and edit-Save re-triggers and the Sync Failed Edit/Retry/Discard display. Trigger Paths changed from 2 to 3. **Before:** only the "offline Post" and "connectivity returns" triggers were listed.
- **Edit 52 (Outcome "Sync failed -- server error"; Processing Logic step 9):** **Before:** "...in which case the screen offers a manual retry"; step 9: "...or the write in step 6 fails, mark that entry with a Sync Failed state". **After:** "...in which case the entry is marked Sync Failed and the screen offers a manual Retry and Discard (FEAT-07.SPEC-001 / FEAT-07.SPEC-002 Sync Failed state)"; step 9: "...or the write in step 6 fails repeatedly across connectivity events (a single transient write failure leaves the entry Queued for automatic retry)".
- **Rationale:** Check 6. The re-spawned screens state that their Retry control "asks FEAT-07.SPEC-008 to attempt the entry's submission now" and that Save "returns the entry to Queued for FEAT-07.SPEC-008 to submit". The screens' declarations are authoritative, so the automation now lists them as a trigger source. Step 9 contradicted the automation's own server-error outcome row, and the screens' state (c) uses the "repeated server error" wording, so step 9 was aligned to it.

#### Edit 53 -- Coverage Summary counts (SG-08 files)
- **Files:** `FEAT-21-settings-account-management/FEAT-21.SPEC-001-account-profile.md` and `FEAT-21-settings-account-management/FEAT-21.SPEC-004-business-details-payment-terms.md` (Coverage Summary, States row)
- **Before:** "7 (loaded, editing, saving, validation error, error, delivery warning, offline) | 7" and "7 (loaded, editing, saving, validation error, incomplete notice, error, offline) | 7"
- **After:** "8 (..., read-only (Dana), offline) | 8" in both
- **Rationale:** Each States table has 8 rows, including "Read-only (Dana)". The summary count was aligned to the spec's own table.

#### Edit 54 -- Brace-wrapped template-instruction lines unwrapped
- **Files and lines:** `FEAT-12.SPEC-003` (Field Validation intro), `FEAT-13.SPEC-005` (Field Validation intro), `FEAT-18.SPEC-005` (Authorization intro), `FEAT-18.SPEC-006` (Field Validation and Authorization intros), `FEAT-18.SPEC-007` (Field Validation intro), `FEAT-18.SPEC-008` (Field Validation and Authorization intros), `FEAT-24.SPEC-006` (Authorization intro), `FEAT-30.SPEC-001` (Entry Points intro). That is 10 lines in 9 files.
- **Before:** each line was a single sentence wrapped in `{...}` (template-instruction form), for example "{This entity captures no user input -- every field is entirely derived. ...}"
- **After:** the same sentence with the enclosing braces removed. The wording is unchanged.
- **Rationale:** Template-residue hygiene. Each sentence is real spec content, but the brace wrapping makes it read as an unfilled template instruction. No content was added or removed.

### Structural Gaps -- Final Status

| Gap | Final status | Evidence |
|-----|--------------|----------|
| SG-01 | Resolved by re-spawn, plus alignment (edits 51-52) | FEAT-07.SPEC-001 and SPEC-002 define the Sync Failed state with Edit (validation failure), Retry (repeated server error) and Discard-with-confirmation. Their AC counts are 23 and 22, matching the frontmatter. FEAT-07.SPEC-008 AC-10 matches the Discard control, and SPEC-008 now lists the screens' Retry and re-save as a trigger. |
| SG-02 | Resolved by re-spawn, plus alignment (edits 34-36) | FEAT-20.SPEC-004 is the sole compare-and-set writer of `onboarding_how_did_you_hear_resolved`, and FEAT-20.SPEC-002, SPEC-003 and SPEC-005 only trigger or describe it. The SPEC-005 "One-time question" wording now matches SPEC-002 advancing without waiting. |
| SG-03 | Unresolved with evidence (map only; this agent may not modify the dependency map) | `feature-dependency-map.md` > Shared Data Entities > Freelancer Account > Fields still lists only name/email, business fields, payment terms, time zone, notification preferences, devices and help-tip dismissals. `grep -c onboarding_ feature-dependency-map.md` returns 0. FEAT-20.SPEC-005's Governed Entity and Defaults tables define `onboarding_referring_portal_ref`, `onboarding_ready_acknowledged`, `onboarding_welcome_warning_dismissed`, `onboarding_how_did_you_hear_resolved` (plus onboarding status and current step). The specs agree with each other; only the map is stale. Owner: whoever may edit the map (orchestrator or Stage 3 map step). |
| SG-04 | Resolved by re-spawn, plus alignment (edits 38-41) | FEAT-13.SPEC-003 has four FEAT-23 trigger rows and Connected Specs rows. FEAT-23.SPEC-002, 004, 005 and 006 each cite FEAT-13.SPEC-003. FEAT-13.SPEC-004's vocabulary, project exemption and affected-record mapping now include the four plan event types. |
| SG-05 | Resolved by re-spawn | The FEAT-23 Brief's State Transition row uses the five plan states (Free+Active, Paid+Active, Paid+Charge failed, Paid+Cancelled (ends at period end), Free+Lapsed) and states that "Downgraded" is not a tier or status. This is consistent with FEAT-23.SPEC-007 and the dependency map. |
| SG-06 | Resolved by re-spawn, plus alignment (edits 44-46) | FEAT-32.SPEC-001, SPEC-003 and SPEC-005 define the "Start over" exit from Connecting after platform parameter: `payment-connect-handoff-timeout`, and the slug is now registered. |
| SG-07 | Resolved by re-spawn, plus alignment (edit 47) | The FEAT-16.SPEC-007 inbound event "Archive download delivered" has Affected Specs FEAT-24.SPEC-003 and AC-16. FEAT-24.SPEC-003's Trigger Definition and Connected Specs cite it back. |
| SG-08 | Resolved by re-spawn, plus alignment (edits 42-43) | FEAT-21.SPEC-001 to SPEC-004 each have a "Payment account" Navigation Out row to FEAT-32.SPEC-001. FEAT-32.SPEC-001 now names FEAT-21.SPEC-001 in its header, back-arrow Interaction, Navigation Out and Connected Specs, with no bare "FEAT-21 (Settings & Account Management)" destination left. |
| SG-09 | Resolved by re-spawn, plus alignment (edits 48-50) | FEAT-04.SPEC-002 routes Owen and Priya to FEAT-07.SPEC-002 or FEAT-08.SPEC-001, and Nadia and Dana to FEAT-06.SPEC-002. All three destinations list FEAT-04.SPEC-002 as an entry point with the matching roles. |
| SG-10 | Resolved by re-spawn | The FEAT-14.SPEC-004 registry has 29 rows covering 26 of 26 Notification specs (script check), and FEAT-14.SPEC-002 references all 26. Its Coverage Summary reads "6 (plus the 29-row registry covering all 26 Notification specs)". |
| SG-11 | Unresolved with evidence (map only; this agent may not modify the dependency map) | `feature-dependency-map.md` > Payment Account Connection > status still reads "Not connected, Connected ..., Needs attention ..., Disconnected", with no `Connecting` value and no `handoff_started_at` field (grep: 0 hits). FEAT-32.SPEC-005 (the creating feature's rule spec, which is authoritative) and FEAT-32.SPEC-001 and SPEC-003 all use `Connecting` and `handoff_started_at`. Owner: map editor. |
| SG-12 | Spec side resolved by re-spawn; map side unresolved with evidence (map only) | Specs: FEAT-09.SPEC-006 now reads Reminder Log `pause_state` (read-only), and no spec references `Invoice.reminder_paused` any more. The only remaining `reminder_paused` in a spec is an analytics event name in FEAT-11.SPEC-003, which is not a field. Map: `feature-dependency-map.md` Invoice > Fields still has "reminder_paused -- (FEAT-11)" (line 205), while the map's own Reminder Log entity already carries `pause_state`. The FEAT-11 Brief's "Flagged discrepancy" note (line 83) also still refers to it. Owner: map editor (and the Feature Analyst for the Brief note). |

**Residual observations (not structural gaps; no re-spawn requested):**
- `feature-dependency-map.md` > Activity Log Entry > Lifecycle ("Created by FEAT-13 on behalf of FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-25, FEAT-31") and the FEAT-13 Brief ("ten other features", lines 43, 52, 66, 120, 128) do not yet include FEAT-23 as the eleventh writer. The specs (FEAT-13.SPEC-003 and SPEC-004) now agree on eleven. This agent may not edit the map or Briefs, so it is logged for the same map and Brief editor as SG-03, SG-11 and SG-12.
- `feature-dependency-map.md` External Touchpoints, "Large-file storage and delivery" row: the Features Involved column lists FEAT-06, FEAT-16 and FEAT-17, but FEAT-24 (archive delivery and account-deletion purge) also relies on FEAT-16.SPEC-007. Check 11 still passes, because its Integration spec IDs agree in both directions. This is informational only.
- Coverage Summary tables in some specs not touched by the gap fixes list fewer states or interactions than their tables contain, for example FEAT-08.SPEC-001 Interactions (5 listed vs 6 rows), FEAT-13.SPEC-001/002 and FEAT-16.SPEC-001 States. These predate the gap fixes and are per-spec quality items (Pass C scope), so they were left as written.
