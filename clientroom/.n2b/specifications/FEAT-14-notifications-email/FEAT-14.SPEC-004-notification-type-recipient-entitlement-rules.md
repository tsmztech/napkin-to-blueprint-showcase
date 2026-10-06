---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-004
spec_name: Notification Type & Recipient Entitlement Rules
spec_slug: notification-type-recipient-entitlement-rules
parent_feature: FEAT-14
parent_feature_name: Notifications (Email)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 27
acceptance_criteria_count: 21
---

# Logic/Rule Spec: Notification Type & Recipient Entitlement Rules

## Overview

**Name:** Notification Type & Recipient Entitlement Rules
**ID:** FEAT-14.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs the one-type-per-event mapping, Access-Matrix-limited recipients, and which notifications a preference can switch off versus which always send.
**Parent Feature:** FEAT-14 -- Notifications (Email)
**Governed Entity:** Notification

## Scope and Non-Goals

**In Scope:**
- The complete registry of notification types, one per triggering event across the product, with its entitled recipient role(s)
- Recipient entitlement checking against the Access Matrix for every notification type
- The transactional-versus-optional classification for every notification type, and the preference-override rule (XBR-30)
- Field-level rules for the Notification entity and authorization rules for every action on it

**Non-Goals:**
- Composing the email's actual subject, body, sender name, or branding -- owned by FEAT-14.SPEC-002 (dispatch) and FEAT-14.SPEC-005 (presentation rules); this spec decides *who* receives *which type*, not what the email says or looks like.
- The mechanics of creating the Notification record or handing it to the delivery capability -- owned by FEAT-14.SPEC-002; this spec supplies the rules that automation calls.
- The screen where Nadia actually toggles an optional preference -- owned by FEAT-21 (Settings & Account Management) as FEAT-21.SPEC-002 (Notification Preferences); this spec defines which types are eligible to be toggled and enforces the outcome, not the settings screen itself.
- Delivery, bounce, and retry status tracking -- owned by FEAT-14.SPEC-001 and FEAT-14.SPEC-003; this spec governs only which Notification gets created and for whom, not what happens to it afterward.

## Governed Entity

**Entity:** Notification
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| notification_type | enum | Exactly one triggering event per type (required) |
| recipient | reference | A Client Contact or the Freelancer Account entitled to the triggering event per the Access Matrix (required) |
| sent_at | date | Timestamp of the dispatch attempt (required, system-derived) |
| delivery_status | enum | Queued, Sent, Delivered, Failed, Bounced (required, system-derived) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | On every triggering event, before a Notification record is created -- resolves notification_type and entitled recipient(s), and checks preference state for any type this spec classifies Optional |
| FEAT-14.SPEC-003 | Delivery Status Tracking & Retry | Reads delivery_status transitions this spec's field rules constrain to a one-directional terminal state machine |
| FEAT-21.SPEC-002 | Notification Preferences (FEAT-21) | Lists only the notification types this spec classifies Optional for Nadia to toggle; a type this spec classifies Transactional is never offered as a toggle |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| notification_type | Must be one of the fixed registry of types defined in Business Rules below -- exactly one type per triggering event, never a free-text or ad hoc value | Always | On create (FEAT-14.SPEC-002) | Internal invariant, never user-facing: "notification_type must map to a registered triggering event" -- a type outside the registry is a defect in the triggering feature's own spec, not a runtime condition a user encounters | Yes |
| recipient | Must be a Client Contact or the Freelancer Account that is entitled to this notification_type per the Authorization Rules and registry below | Always | On create (FEAT-14.SPEC-002) | Internal invariant, never user-facing: "recipient must be entitled to notification_type" -- an unentitled recipient is never resolved in the first place, so no Notification record for them is ever attempted (see FEAT-14.SPEC-002's No Entitled Recipients outcome) | Yes |
| sent_at | No validation beyond data type -- system-derived, never user-entered | Always | -- | -- | -- |
| delivery_status | Must be one of Queued, Sent, Delivered, Failed, Bounced, and may only transition toward a terminal state (never regress from Delivered to an earlier state), per FEAT-14.SPEC-003's state machine | Always | On update (FEAT-14.SPEC-003) | Internal invariant, never user-facing: "delivery_status transitions must be forward-only" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Recipient-entitlement-by-type | notification_type, recipient | The recipient's role (Nadia, Owen, Priya) must appear in the entitled-role set for that notification_type in the registry (Business Rules); a role not in that set is never resolved as a recipient for that type | Internal invariant, never user-facing -- a non-entitled contact simply never has a Notification record created for that type (FEAT-14.SPEC-002's No Entitled Recipients / Partial Dispatch outcomes); no error is ever shown to any human |
| Preference-gates-optional-types-only | notification_type, recipient (Freelancer Account -- notification_preferences) | A preference toggle may govern a notification_type only if that type is classified Optional in the registry; a Transactional type ignores the preference state entirely and always resolves its entitled recipients (XBR-30) | Internal invariant -- FEAT-21's preferences screen is never offered a toggle for a Transactional type in the first place, so this condition cannot be violated through the product's own UI |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a Notification record | System, via FEAT-14.SPEC-002, following this spec's entitlement check | Always, when at least one recipient is entitled | -- |
| Receive (be resolved as the recipient of) a Notification | Nadia (Freelancer) | For every notification_type whose registry entry entitles the Freelancer Account (all types addressed to her practice or account) | Not applicable -- Nadia is entitled to every notification type that names her; no denial path exists for her own account's notifications |
| Receive (be resolved as the recipient of) a Notification | Owen (Client Primary Contact) | Only for notification_types whose registry entry entitles the Primary Contact role, and only for his own client company (Own-only) | He is simply never resolved as a recipient for a type that does not entitle Primary contacts (e.g., an invoice-content type addressed only within the freelancer's own account is not applicable here, but any type restricted to Nadia); no error is shown anywhere, since the concept of "denial" does not surface to a contact who was never a candidate recipient |
| Receive (be resolved as the recipient of) a Notification | Priya (Client Reviewer Contact) | Only for notification_types whose registry entry entitles the Reviewer Contact role, and only for her own client company (Own-only); she is never entitled to proposal-content or invoicing-content types, per the Access Matrix | She is silently never resolved as a recipient for a type outside her entitlement (e.g., proposal_sent, invoice_issued, payment_confirmation, overdue_reminder); no error or "hidden" state is shown to her, since she is not a candidate recipient for those types in the first place |
| Receive (be resolved as the recipient of) a Notification | Dana (Support Operator) | Never -- Dana is never a recipient of any Notification; her Notifications & Help access is "View (delivery warnings only)" | Dana is never resolved as a recipient by this spec's rules under any notification_type |
| View delivery status / a delivery warning | Nadia (Freelancer) | Full -- sees delivery warnings for any Notification tied to her own account's projects | -- |
| View delivery status / a delivery warning | Dana (Support Operator) | View only, delivery warnings only (FEAT-14.SPEC-006), inside a logged support session (FEAT-31); never the underlying subject/body content of the notification itself | A direct attempt to view a Notification's full content is never offered to Dana; her support session surfaces only the delivery-warning summary defined by FEAT-14.SPEC-006 |
| View delivery status / a delivery warning | Owen, Priya (Client Contacts) | Never -- client contacts have no view into Notification records or delivery status as data; they only ever receive the email itself | No delivery-status view exists anywhere in the client portal; the concept of a Notification record is invisible to client contacts entirely |
| Update delivery_status | System, via FEAT-14.SPEC-003, following the defined state machine | Always, forward-only | -- |
| Toggle a notification_type's preference on/off | Nadia (Freelancer) | Only for a notification_type classified Optional in the registry below | An attempt to toggle a Transactional type is never offered as a control in FEAT-21's preferences screen; it does not exist as an actionable element |
| Delete a Notification record | No one, in-feature | Never -- Notification records are removed only by FEAT-24 (account deletion) | No delete action exists anywhere in the product for an individual Notification record |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| notification_type | Derived from the registry mapping (Business Rules) based on which triggering feature/spec fired | On create | No -- never manually chosen by any user |
| recipient | Derived from entitlement resolution against the Access Matrix and, for an Optional type, the current preference state | On create | No -- never manually chosen by any user |
| delivery_status | Defaults to Queued | On create | No |
| sent_at | Set to the time FEAT-14.SPEC-001 confirms a transmission attempt was made | On the first dispatch attempt | No |

## Business Rules

**Notification Type Registry** -- the complete, closed set of notification types this product defines in MVP, one per triggering event (Validation & Limits, feature-overview.md), with its entitled recipient role(s) and its transactional-versus-optional classification:

| Notification Type | Triggering Feature / Spec | Entitled Recipient Role(s) | Classification |
|---|---|---|---|
| proposal_sent | FEAT-02 (FEAT-02.SPEC-011) | Owen (Primary) | Transactional |
| proposal_accepted_confirmation | FEAT-03 (FEAT-03.SPEC-006) | Owen (Primary), Nadia | Transactional |
| proposal_change_requested | FEAT-03 (FEAT-03.SPEC-007) | Nadia | Transactional |
| magic_link_sign_in | FEAT-05 (FEAT-05.SPEC-008) | Owen (Primary) or Priya (Reviewer) -- whichever contact requested | Transactional |
| deliverable_ready | FEAT-06 (FEAT-06.SPEC-006) | Owen (Primary), Priya (Reviewer) | Transactional |
| client_comment_alert | FEAT-07 (FEAT-07.SPEC-003) | Nadia | Transactional |
| freelancer_reply_alert | FEAT-07 (FEAT-07.SPEC-004) | Owen (Primary), Priya (Reviewer) | Transactional |
| milestone_approved_confirmation | FEAT-08 (FEAT-08.SPEC-007) | Owen (Primary), Nadia | Transactional |
| invoice_issued | FEAT-09 (FEAT-09.SPEC-010) | Owen (Primary), Nadia | Transactional |
| payment_confirmation | FEAT-10 (FEAT-10.SPEC-007) | Owen (Primary), Nadia | Transactional |
| overdue_reminder | FEAT-11 (FEAT-11.SPEC-004) | Owen (Primary) | Transactional |
| contact_invitation | FEAT-18 (FEAT-18.SPEC-010) | The newly invited contact (Owen or Priya) | Transactional |
| welcome_email | FEAT-20 (FEAT-20.SPEC-006) | Nadia | Transactional |
| support_session_notice | FEAT-31 (FEAT-31.SPEC-007) | Nadia | Transactional (XBR-29 mandates it is "always announced to the freelancer by email") |
| chargeback_notice | FEAT-25 (FEAT-25.SPEC-008), from the reversal notice FEAT-32.SPEC-002 relays | Nadia | Transactional (XBR-21) |
| delivery_failure_warning | FEAT-14.SPEC-006 (this feature) | Nadia (Dana views it read-only, never as a recipient) | Transactional (cannot itself be turned off, or a lost email's warning could itself be silently lost) |
| primary_invited_colleague_alert | FEAT-18 (FEAT-18.SPEC-011), fired by FEAT-18.SPEC-004 | Nadia (never a client contact) | Transactional (access-relevant change to who can see her work; parallel to XBR-29) |
| sign_in_email_change_confirmation | FEAT-21 (FEAT-21.SPEC-011), fired by FEAT-21.SPEC-005 | Nadia -- one copy to her prior sign-in email and one to her new sign-in email, both her own account | Transactional (XBR-30; security-sensitive credential change) |
| plan_change_confirmation | FEAT-23 (FEAT-23.SPEC-008), fired by FEAT-23.SPEC-004 and FEAT-23.SPEC-006 | Nadia | Transactional (XBR-30) -- one type covering the upgrade, downgrade, cancellation-confirmed, paid-plan-ended and lapsed variants, since each is a different outcome of the same plan-change event |
| subscription_charge_failed | FEAT-23 (FEAT-23.SPEC-008), fired by FEAT-23.SPEC-003 via FEAT-23.SPEC-004 | Nadia | Transactional (XBR-30; account-standing alert) |
| data_export_ready | FEAT-24 (FEAT-24.SPEC-008), fired by FEAT-24.SPEC-003 | Nadia | Transactional (XBR-30; confirms an action she requested) |
| account_deletion_final_warning | FEAT-24 (FEAT-24.SPEC-009), fired by FEAT-24.SPEC-002 | Nadia | Transactional (XBR-30; record of an irreversible action) |
| refund_recorded | FEAT-25 (FEAT-25.SPEC-007), fired by FEAT-25.SPEC-003 | Owen (Primary) only -- never Priya (no billing visibility), never Nadia (the actor) | Transactional (XBR-30) |
| project_cancelled | FEAT-25 (FEAT-25.SPEC-007), fired by FEAT-25.SPEC-004 | Owen (Primary) only -- never Priya, never Nadia | Transactional (XBR-30) |
| signed_copy_confirmation | FEAT-26 (FEAT-26.SPEC-004), fired by FEAT-26.SPEC-002 | Owen (Primary, the signing contact), Nadia | Transactional (XBR-30; evidentiary record, sent in addition to proposal_accepted_confirmation) |
| custom_domain_verified_confirmation | FEAT-27 (FEAT-27.SPEC-004), fired by FEAT-27.SPEC-002 | Nadia | Transactional (XBR-30) |
| support_request_confirmation | FEAT-31 (FEAT-31.SPEC-006), fired by FEAT-31.SPEC-001 | Nadia | Transactional (XBR-30; receipt of her own request) |
| payment_connection_connected | FEAT-32 (FEAT-32.SPEC-006), fired by FEAT-32.SPEC-003 | Nadia | Transactional (XBR-30) |
| payment_connection_needs_attention | FEAT-32 (FEAT-32.SPEC-006), fired by FEAT-32.SPEC-003 | Nadia | Transactional (XBR-30; affects whether she can get paid) |

**Zero notification types are currently classified Optional.** Every type registered above is either an evidentiary or action-required record for a client (proposal, deliverable, approval, invoice, payment, reminder), a security- or access-critical message (magic-link sign-in, contact invitation), a transparency mandate the product itself commits to (support-session notice, XBR-29), a financial-integrity notice (chargeback, XBR-21), or an account-critical confirmation to Nadia (sign-in email change, plan and billing changes, payment-connection status, custom-domain verification, data export, account deletion, support-request receipt, invited-colleague alert, refund and cancellation records, signed copy) -- none is a discretionary, marketing-style alert. This is consistent with every sibling Notification spec in this run (FEAT-02.SPEC-011, FEAT-03.SPEC-006/007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003/004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004, FEAT-18.SPEC-010/011, FEAT-20.SPEC-006, FEAT-21.SPEC-011, FEAT-23.SPEC-008, FEAT-24.SPEC-008/009, FEAT-25.SPEC-007/008, FEAT-26.SPEC-004, FEAT-27.SPEC-004, FEAT-31.SPEC-006/007, FEAT-32.SPEC-006), each of which independently states it is transactional and cannot be disabled, citing XBR-30. The preference mechanism itself is still fully specified and enforced here (Cross-Field Rules, Authorization Rules) exactly as XBR-30 requires, so any future notification type this product adds can be classified Optional by an update to this registry without any change to the dispatch or entitlement machinery.

- Registry coverage is complete: every Notification spec in the product (26 across FEAT-02 through FEAT-32 and FEAT-14.SPEC-006) has at least one row above (29 rows in all). A spec that reports two distinct triggering events (FEAT-23.SPEC-008: plan change and failed charge; FEAT-25.SPEC-007: refund and cancellation; FEAT-32.SPEC-006: connected and needs attention) has one row per event; a spec whose variants are outcomes of one event (FEAT-23.SPEC-008's plan-change variants, FEAT-11.SPEC-004's day-3, day-10 and manual reminders) has a single row.
- Each notification_type maps to exactly one triggering event, avoiding duplicate or missing emails for the same underlying occurrence (feature-overview.md, Validation & Limits).
- Recipients are limited to contacts entitled to the event per the Access Matrix (XBR-08) -- a Reviewer contact never receives invoice or proposal-content emails; a client never receives another client's information (ASMP-23).
- A Transactional type's recipient resolution never consults notification_preferences; an Optional type's recipient resolution consults the Freelancer Account's current notification_preferences at the moment FEAT-14.SPEC-002 runs its entitlement check, not at some earlier moment (XBR-30).
- Dana (Support Operator) is never entitled as a recipient of any notification_type; her only relationship to a Notification is viewing its delivery-warning summary, read-only, inside a logged support session (FEAT-31), per the Access Matrix's "View (delivery warnings only)."
- This registry is the single source of truth FEAT-21's notification-preferences screen (FEAT-21.SPEC-002) lists from; that screen only ever offers a toggle for a type classified Optional here.

## Edge Cases

- **A notification_type is entitled to both a Primary and a Reviewer contact, but the client currently has no Reviewer contact on record** -- The registry entry's entitlement is unaffected; a Notification record is simply created only for the contact(s) that currently exist and are entitled (e.g., freelancer_reply_alert creates a record for Owen alone if no Priya-equivalent contact exists for that client).
- **A client has two Primary contacts** -- Every current Primary contact receives their own Notification record and their own copy of the email for a type entitling the Primary role, per FEAT-02.SPEC-011's own edge case; this spec's entitlement rule applies per-contact, not per-role-slot.
- **A contact's role changes (Reviewer promoted to Primary, or vice versa) between the triggering event firing and this spec's entitlement check running** -- The role in effect at the moment FEAT-14.SPEC-002 runs the entitlement check governs, consistent with the preference-evaluated-at-dispatch-time rule; a role change made a moment before the check is honored, one made a moment after is not (XBR-08, FEAT-18 owns role changes).
- **A notification_type this registry classifies Transactional is mistakenly offered as a toggle by a future change to FEAT-21** -- This spec's Cross-Field Rule (Preference-gates-optional-types-only) is the authoritative boundary: even if a toggle were surfaced, the entitlement check here ignores preference state entirely for a Transactional type, so the type continues to send regardless.
- **A future notification type is added to the product with no registry entry yet** -- Per Field Validation Rules, notification_type must map to a registered triggering event; an unregistered type is a defect in the triggering feature's own spec to be caught before that feature's specs pass review, not a runtime condition this spec's entitlement engine is expected to handle gracefully.

## Acceptance Criteria

**FEAT-14.SPEC-004-AC-01:** Given a triggering event fires with a notification_type that matches an entry in the registry, when FEAT-14.SPEC-002 checks this spec's rules, then the type is accepted and entitlement resolution proceeds.

**FEAT-14.SPEC-004-AC-02:** Given a Notification record is being created, when its recipient is resolved, then the recipient's role must appear in that notification_type's entitled-role set, or no record is created for that would-be recipient.

**FEAT-14.SPEC-004-AC-03:** Given a Notification's delivery_status is Delivered, when any subsequent event is processed, then delivery_status never regresses to an earlier state (Queued, Sent, Failed, or Bounced).

**FEAT-14.SPEC-004-AC-04:** Given a proposal_sent event fires for a client with Owen as Primary and Priya as Reviewer, when recipients are resolved, then a Notification record is created for Owen and none for Priya, since proposal_sent entitles only the Primary role.

**FEAT-14.SPEC-004-AC-05:** Given a deliverable_ready event fires, when recipients are resolved, then Notification records are created for both Owen and Priya, since deliverable_ready entitles both roles.

**FEAT-14.SPEC-004-AC-06:** Given a proposal_change_requested event fires, when recipients are resolved, then a Notification record is created only for Nadia; Owen and Priya are never resolved as recipients for this type.

**FEAT-14.SPEC-004-AC-07:** Given Nadia (Freelancer) is the subject of a milestone_approved_confirmation event, when recipients are resolved, then she is always entitled and a Notification record is created for her alongside Owen's.

**FEAT-14.SPEC-004-AC-08:** Given Dana (Support Operator) is inside a logged support session, when she looks for a way to view a Notification's full content, then no such view exists -- she can see only the delivery-warning summary defined by FEAT-14.SPEC-006.

**FEAT-14.SPEC-004-AC-09:** Given Owen looks for a way to see his own or another Notification's delivery status as data, when he looks in the portal, then no such view exists for client contacts.

**FEAT-14.SPEC-004-AC-10:** Given every notification_type in the registry is currently classified Transactional, when Nadia opens her notification preferences (FEAT-21), then no toggle is offered for any of them.

**FEAT-14.SPEC-004-AC-11:** Given a hypothetical future notification_type were classified Optional, when Nadia turns it off in her preferences, then FEAT-14.SPEC-002's entitlement check for that type returns zero recipients for her account going forward.

**FEAT-14.SPEC-004-AC-12:** Given a Transactional notification_type, when FEAT-14.SPEC-002 checks entitlement, then the Freelancer Account's notification_preferences are never consulted, and every entitled recipient always resolves.

**FEAT-14.SPEC-004-AC-13:** Given a support_session_notice event fires (Dana opens a session), when recipients are resolved, then Nadia is always entitled, per XBR-29's mandate that support sessions are always announced by email.

**FEAT-14.SPEC-004-AC-14:** Given a chargeback_notice event fires, when recipients are resolved, then Nadia is always entitled, per XBR-21.

**FEAT-14.SPEC-004-AC-15:** Given a new contact is invited via FEAT-18, when the contact_invitation event fires, then the newly invited contact (Owen or Priya, whichever role they were assigned) is the sole entitled recipient.

**FEAT-14.SPEC-004-AC-16:** Given a client has two Primary contacts, when a notification_type entitling the Primary role fires, then each Primary contact receives their own independent Notification record.

**FEAT-14.SPEC-004-AC-17:** Given a contact's role changes from Reviewer to Primary a moment before FEAT-14.SPEC-002 runs its entitlement check, when the check runs, then the updated (Primary) role governs entitlement for that occurrence.

**FEAT-14.SPEC-004-AC-18:** Given no Reviewer contact currently exists for a client, when a notification_type entitling the Reviewer role fires, then a Notification record is created only for the currently existing entitled contact(s), with no error for the missing role.

**FEAT-14.SPEC-004-AC-19:** Given a notification_type outside the registry is somehow supplied to this spec's rules, when FEAT-14.SPEC-002 checks it, then the type is rejected as an internal defect and no Notification record is created under an unregistered type.

**FEAT-14.SPEC-004-AC-20:** Given anyone attempts to delete an individual Notification record within the product, when they look for such an action, then none exists -- Notification records are removed only by FEAT-24's account deletion.

**FEAT-14.SPEC-004-AC-21:** Given any of the 26 Notification specs in the product fires its triggering event (for example FEAT-24.SPEC-008's export-ready event, FEAT-26.SPEC-004's signature-recorded event, or FEAT-32.SPEC-006's needs-attention event), when FEAT-14.SPEC-002 looks up its notification_type, then a registry row exists naming that spec, and recipients resolve per that row (data_export_ready to Nadia only; signed_copy_confirmation to Owen and Nadia; payment_connection_needs_attention to Nadia only; refund_recorded to Owen only).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 11 | 11 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 (plus the 29-row registry covering all 26 Notification specs) | 6 |
| Edge Cases | 5 | 5 |
