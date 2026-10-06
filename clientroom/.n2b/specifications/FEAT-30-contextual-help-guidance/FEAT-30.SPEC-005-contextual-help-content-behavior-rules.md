---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-30.SPEC-005
spec_name: Contextual Help Content & Behavior Rules
spec_slug: contextual-help-content-behavior-rules
parent_feature: FEAT-30
parent_feature_name: Contextual Help & Guidance
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 22
---

# Logic/Rule Spec: Contextual Help Content & Behavior Rules

## Overview

**Name:** Contextual Help Content & Behavior Rules
**ID:** FEAT-30.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs which guidance content each role may see, that guidance is always advisory and never blocking, and that a dismissed tip is suppressed on every future render.
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance
**Governed Entity:** Help-Tip Dismissal State (feature-local state held on the Freelancer Account or Client Contact record)

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Help-Tip Dismissal State entity
- The role-content-scoping rule that determines which tips and reference topics a given role may ever be shown
- The advisory-only (never-blocking) rule
- The dismissal-suppression rule
- Authorization rules for every action on the Help-Tip Dismissal State, for every role
- Default values and derivations for the entity's fields

**Non-Goals:**
- The visual layout or interaction pattern of the tooltip or reference screens -- owned by FEAT-30.SPEC-001, FEAT-30.SPEC-002, and FEAT-30.SPEC-003, which reference this spec for the rules they enforce.
- The persistence mechanics of writing a dismissal -- owned by FEAT-30.SPEC-004, which this spec's rules govern but does not itself execute.
- A general-purpose chat, messaging, or live-support channel -- excluded per scope-boundaries.md SC-15, consistent with the rest of FEAT-30.
- Operator-side dismissal, reset, or editing of a user's guidance state -- excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only, and she never acts as, or on behalf of, a freelancer or client contact.

## Governed Entity

**Entity:** Help-Tip Dismissal State
**Source:** Feature Dependency Map (feature-local state layered on Freelancer Account or Client Contact, per FEAT-30's Entity-Lifecycle Coverage Matrix)

| Field | Data Type | Description |
|-------|-----------|-------------|
| tip_id | text | Identifier of the specific contextual tip or reference topic this record's dismissal state applies to; must reference an entry in the fixed, deploy-time help-content catalog |
| host_record | enum (Freelancer Account reference \| Client Contact reference) | The single account or contact this dismissal flag is physically stored on |
| dismissed | boolean | True once the user has permanently dismissed this tip; defaults to false |
| dismissed_at | date/time | The moment dismissed became true; unset while dismissed is false |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-30.SPEC-001 | Contextual Help Tooltip | Immediately before rendering any tip's affordance, and on the "Don't show this again" action |
| FEAT-30.SPEC-002 | Freelancer Help Reference | Immediately before rendering any reference topic's dismiss-eligibility, and on the "Don't show this again" action |
| FEAT-30.SPEC-003 | Client Portal Help Reference | Immediately before rendering the role-scoped topic list and any reference topic's dismiss-eligibility, and on the "Don't show this again" action |
| FEAT-30.SPEC-004 | Help Tip Dismissal Recording | On every write to the Help-Tip Dismissal State (default initialization and the dismissal transition) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tip_id | Must reference an entry in the current fixed help-content catalog | Always | On every read and write | N/A -- an unrecognized tip_id is silently treated as ineligible to render (never shown); see FEAT-30.SPEC-004 Edge Cases | No |
| host_record | Required; must reference exactly one Freelancer Account or exactly one Client Contact, never both | Always | On write (creation) | N/A -- this is a system-set reference, never user-entered, so no user-facing error message applies | No |
| dismissed | Boolean; no validation beyond data type | Always | -- | -- | -- |
| dismissed_at | Must be set if and only if dismissed is true | Always | On write | See Cross-Field Rules below | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Dismissal timestamp consistency | dismissed, dismissed_at | dismissed_at is set exactly when dismissed transitions to true, and remains unset whenever dismissed is false; the two fields can never disagree | N/A -- this is a system-enforced write-time invariant with no user-facing input, so no error message is shown to any user; FEAT-30.SPEC-004 enforces it structurally by only ever setting both together |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read own dismissal state (used internally to decide whether to render a tip or reference-entry dismiss action) | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Always, and only their own record (own tip_id + own host_record) | -- |
| Read dismissal state | Dana (Support Operator) | Never | The contextual help layer is never rendered in a support session; per the Access Matrix, Dana's Notifications & Help entitlement is limited to delivery warnings, so this entity is never read on her behalf. |
| View own dismissal history (a list of previously dismissed tips) | Nadia, Owen, Priya | Never -- no such capability exists | No screen or control exposes a list of past dismissals to any role; the product defines only forward, permanent dismissal, never a review-or-restore list (Entity-Lifecycle Coverage Matrix, Read (list) row). |
| View any user's dismissal history | Dana | Never | No screen or control exposes any user's dismissal history to Dana; her Notifications & Help entitlement is limited to delivery warnings, and this capability does not exist for any role in the first place. |
| Dismiss a tip (set dismissed = true) | Nadia, Owen, Priya | Always, only their own record, and only in the forward direction (not-dismissed -> dismissed) | -- |
| Dismiss a tip on another user's behalf | Nadia, Owen, Priya | Never | No control exists anywhere in the product for one user to dismiss a tip for another; each role sees and dismisses only its own guidance state. |
| Dismiss a tip | Dana | Never | No dismiss control exists in a support session; scope-boundaries.md SC-04 bars the operator from acting as, or on behalf of, a freelancer or client contact. |
| Undismiss or reset a tip (own) | Nadia, Owen, Priya | Never -- no reverse transition is defined | No undismiss or reset control exists anywhere in the product; dismissal is permanent by design (Entity-Lifecycle Coverage Matrix, State Transition row). |
| Undismiss or reset a tip on someone else's behalf | Dana | Never | FEAT-30's Non-Goals explicitly exclude operator-side dismissal, reset, or editing of a user's guidance state. |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| dismissed | Defaults to false | On create -- the first time a given tip_id is eligible to render for a given user (FEAT-30.SPEC-004) | No (the only user-facing transition is the forward dismissal action, which sets it to true; there is no override of the default itself) |
| dismissed_at | Unset by default; set to the current date/time the moment dismissed transitions to true | On the dismissal write only (FEAT-30.SPEC-004) | No |
| tip_id | Direct assignment from the tip or reference topic the user is viewing -- not a default or derived value | On create | No -- it is fixed by which tip triggered the record |
| host_record | Direct assignment from the acting user's own Freelancer Account or Client Contact reference -- not a default or derived value | On create | No -- always the acting user's own record; see Authorization Rules |

## Business Rules

- **Advisory-only, never blocking:** No control's core action is ever gated by whether its tip has been opened or dismissed. The presence, absence, or dismissal state of a tip never changes what a user can do on the host screen.
- **Role-content scoping (XBR-08):** A tip or reference topic is eligible to render for a role only if that role has an entitlement to the control or action it explains. Priya (Reviewer) is never shown a tip or topic explaining a Primary-only action (accept, approve, pay, download invoice, invite a Reviewer) because her role has no entitlement to those actions in the first place -- the host screen never renders the underlying control to her, so no tip attaches to it either.
- **Eligibility to render (the single definition used by FEAT-30.SPEC-001, FEAT-30.SPEC-002, and FEAT-30.SPEC-003):** A tip_id is eligible to render for a user if and only if all three conditions hold: (1) the user's role is entitled to the control or action the tip explains (role-content scoping, XBR-08); (2) the tip_id exists in the current fixed help-content catalog; and (3) the user's Help-Tip Dismissal State for that tip_id has dismissed = false or no record yet. An "encounter" is any render of a host screen region that contains the tip's control. The first encounter of an eligible tip is its first-encounter moment, and the tip's inline affordance is offered at that encounter and at every later encounter -- a temporary close ("Got it," the close (X), or an outside tap) records nothing and never ends eligibility -- until the user permanently dismisses it via "Don't show this again." There is no view count, time limit, or automatic expiry after which an undismissed, entitled tip stops being offered.
- **Dismissal suppression:** Immediately before FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 would render a given tip_id for a given user, the eligibility rule above is checked. If the Help-Tip Dismissal State for that pairing has dismissed = true, one behavior applies on every surface: (a) the inline affordance (FEAT-30.SPEC-001) is not rendered; (b) on a reference topic (FEAT-30.SPEC-002, FEAT-30.SPEC-003) the "Don't show this again" action is not rendered and is replaced by a static, non-interactive note reading "Inline tips for this topic are turned off," with no restore control; and (c) the reference topic itself stays in the list with its full explanation, always visible and expandable regardless of dismissal state -- dismissal never suppresses the underlying reference content.
- **One-directional lifecycle:** Once dismissed, a tip's state never reverts (Entity-Lifecycle Coverage Matrix, State Transition row); this spec defines no combination of conditions under which dismissed reverts to false.
- **Inherited retention:** This spec defines no purge policy of its own for the Help-Tip Dismissal State; its retention is entirely inherited from its host record's own delete/erasure path (FEAT-18's contact erasure, FEAT-24's account deletion).

## Edge Cases

- **tip_id references a tip retired from the current help-content catalog** -- Treated as ineligible to render regardless of its dismissal state; no error is produced anywhere in the product.
- **The same tip is evaluated twice within a single render (e.g., appears on two regions of a busy screen)** -- The read is idempotent; both evaluations see the same dismissal state and reach the same eligibility decision.
- **A Client Contact's role changes mid-session (Reviewer promoted to Primary, or vice versa demoted, by FEAT-18)** -- Content eligibility re-evaluates against the new role on the next render; dismissal states are tracked per tip_id and are unaffected by the role change -- a newly eligible tip (now accessible under the new role) has no prior dismissal record and renders as not-yet-dismissed by default; a tip that becomes ineligible under a demotion is simply no longer offered, regardless of its dismissal state.
- **A dismissal write is attempted for a tip_id not present in the current catalog (stale cached client)** -- Accepted as a no-op with no error, per FEAT-30.SPEC-004; this spec's eligibility rule would never have rendered that tip_id in the first place.
- **dismissed is true but dismissed_at is somehow missing (a boundary the cross-field rule is designed to prevent)** -- Never occurs under normal operation because FEAT-30.SPEC-004 sets both fields together in the same write; there is no code or user path in this product definition that sets one without the other.
- **Dana's support session views a host screen that, for the account owner, would show a contextual tip** -- The tip's affordance is never rendered in a support session at all, regardless of the account owner's own dismissal state for it, per the Authorization Rules' "Read dismissal state: Dana, Never" row.

## Acceptance Criteria

**FEAT-30.SPEC-005-AC-01:** Given a tip_id exists in the current help-content catalog, when FEAT-30.SPEC-001 checks its record, then the read succeeds and eligibility is decided from the dismissed field.

**FEAT-30.SPEC-005-AC-02:** Given a tip_id has been retired from the current help-content catalog, when any of FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 evaluate it, then it is treated as ineligible to render, with no error shown anywhere.

**FEAT-30.SPEC-005-AC-03:** Given a Help-Tip Dismissal State record is created for Nadia's own tip, when the host_record is checked, then it references Nadia's Freelancer Account and no other account.

**FEAT-30.SPEC-005-AC-04:** Given a tip has never been dismissed, when its record is checked, then dismissed is false and dismissed_at is unset.

**FEAT-30.SPEC-005-AC-05:** Given a tip has just been dismissed, when its record is checked immediately after, then dismissed is true and dismissed_at holds the moment of dismissal -- the two fields are never found in disagreement.

**FEAT-30.SPEC-005-AC-06:** Given Nadia is viewing her own dismissal state indirectly through FEAT-30.SPEC-001's render check, when the check runs, then it succeeds because she is reading only her own record.

**FEAT-30.SPEC-005-AC-07:** Given Dana is in a support session, when the host screen would otherwise check a dismissal state to render a tip, then that read never occurs and no tip is rendered to her.

**FEAT-30.SPEC-005-AC-08:** Given Nadia, Owen, or Priya looks for a list of tips they have previously dismissed, when they search the product, then no such screen or control exists anywhere.

**FEAT-30.SPEC-005-AC-09:** Given Owen has not yet dismissed a tip, when he taps "Don't show this again," then the transition from not-dismissed to dismissed succeeds.

**FEAT-30.SPEC-005-AC-10:** Given Priya wants to dismiss a tip on Owen's behalf, when she looks for a way to do so, then no such control exists -- each contact dismisses only their own guidance state.

**FEAT-30.SPEC-005-AC-11:** Given Dana wants to dismiss a tip for Nadia during a support session, when she looks for a dismiss control, then none is shown to her, consistent with SC-04.

**FEAT-30.SPEC-005-AC-12:** Given Owen has permanently dismissed a tip, when he looks for a way to undismiss or restore it, then no such control exists anywhere in the product.

**FEAT-30.SPEC-005-AC-13:** Given Dana wants to reset a dismissed tip for Nadia, when she looks for a reset control, then none exists, per FEAT-30's Non-Goals.

**FEAT-30.SPEC-005-AC-14:** Given a tip is eligible to render for Nadia for the first time, when FEAT-30.SPEC-004 initializes its record, then dismissed defaults to false with no user action required.

**FEAT-30.SPEC-005-AC-15:** Given Priya (Reviewer) is viewing a milestone screen, when tip eligibility is evaluated for the Approve control, then no tip is offered to her, because her role has no entitlement to that control at all (XBR-08).

**FEAT-30.SPEC-005-AC-16:** Given a tip has been dismissed by Nadia, when the host screen that would show it renders again, then the tip's affordance is suppressed while the rest of the screen renders normally and unaffected.

**FEAT-30.SPEC-005-AC-17:** Given Owen has dismissed a tip explaining the pay-invoice action, when he next opens the invoice screen, then he can still pay the invoice without restriction -- the dismissal never gates the underlying action, consistent with the advisory-only rule.

**FEAT-30.SPEC-005-AC-18:** Given Priya is promoted from Reviewer to Primary contact (FEAT-18) mid-session, when guidance eligibility is next evaluated for her, then Primary-scoped tips and topics become eligible for the first time, with no retroactive change to her existing dismissal records.

**FEAT-30.SPEC-005-AC-19:** Given a Client Contact's details are erased on request (FEAT-18), when the erasure completes, then their Help-Tip Dismissal State records are removed together with the rest of the erased record, since this spec defines no independent retention for them.

**FEAT-30.SPEC-005-AC-20:** Given Dana is in a support session, when she looks for a way to view Nadia's (or any user's) dismissal history, then no such screen or control exists, consistent with her Notifications & Help entitlement being limited to delivery warnings.

**FEAT-30.SPEC-005-AC-21:** Given Nadia is entitled to a control's tip, the tip_id is in the catalog, and she has tapped "Got it" on it without ever choosing "Don't show this again," when she next encounters the control on any later render, then the affordance is offered again -- and it is offered again on every subsequent encounter until she permanently dismisses it.

**FEAT-30.SPEC-005-AC-22:** Given Owen has permanently dismissed the tip for a topic that appears in the Client Portal Help Reference, when he opens that topic, then the topic and its full explanation are still shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
