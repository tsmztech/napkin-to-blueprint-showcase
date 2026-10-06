# FEAT-09 — Household Invitations & Membership

This chapter covers FEAT-09, Household Invitations & Membership, a Core-tier feature. It contains 14 specifications carrying 137 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-09.SPEC-001 | Household Invitations Manager | screen | 12 |
| FEAT-09.SPEC-002 | Invitation Acceptance | screen | 11 |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | screen | 10 |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | screen | 10 |
| FEAT-09.SPEC-005 | Leave Household | screen | 9 |
| FEAT-09.SPEC-006 | Invitation Expiry | automation | 8 |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | automation | 9 |
| FEAT-09.SPEC-008 | Member Departure Processing | automation | 8 |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | automation | 8 |
| FEAT-09.SPEC-010 | Invitation & Membership Validation Rules | logic-rule | 15 |
| FEAT-09.SPEC-011 | Household Invitations & Membership Authorization Rules | logic-rule | 14 |
| FEAT-09.SPEC-012 | Invitation Accepted Confirmation | notification | 8 |
| FEAT-09.SPEC-013 | Member Left Household Notification | notification | 7 |
| FEAT-09.SPEC-014 | Organiser Hand-Over Request Notification | notification | 8 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Household Invitations & Membership

## Summary

**Feature:** Household Invitations & Membership
**ID:** FEAT-09
**Description:** The organiser invites other adults to join the household, so everyone sees the same plan and the same grocery list.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief states plainly that "everyone in a household sees the same plan and grocery list" (BRIEF.md, Target Users & Roles) — membership within a household is the mechanism that makes shared use possible at all. Growth between households is owned by Invite Another Household (FEAT-24); this feature covers membership within one household only.

**Key Capabilities:**
- Send an invitation — Organiser invites another adult by a simple, shareable method
- Accept an invitation — Invited adult joins the household and gains their role's access
- Manage outstanding invitations — Organiser can see and revoke a pending invitation
- Hand over the organiser role — The organiser makes another adult member the organiser, for example before stepping back from planning
- Leave the household — An other adult member leaves on their own; their ratings stay only as anonymous influence on future plans and their profile is removed

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-09.SPEC-001 | Household Invitations Manager | Screen | Maya | Organiser sends a new invitation and sees, resends, or revokes every outstanding and accepted invitation |
| FEAT-09.SPEC-002 | Invitation Acceptance | Screen | Unauthorized Visitor, Sam | Invited adult opens their invitation link and accepts it, or sees why it is no longer valid |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | Screen | Maya | Organiser picks an active adult member and starts handing over the organiser role to them |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | Screen | Sam | The chosen adult member accepts or declines becoming the new organiser |
| FEAT-09.SPEC-005 | Leave Household | Screen | Sam | An other adult member confirms and completes leaving the household on their own |
| FEAT-09.SPEC-006 | Invitation Expiry | Automation | Maya | A pending invitation automatically expires 14 days after it was sent |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | Automation | Sam, Maya | On acceptance, creates the new Member Profile with Other Adult Member access and routes the invitee into first-use onboarding |
| FEAT-09.SPEC-008 | Member Departure Processing | Automation | Sam, Maya | On a member leaving, anonymises their ratings, removes their Member Profile from active membership, and stops future plans from accounting for them |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | Automation | Maya, Sam | On hand-over acceptance, transfers the Household's organiser field and swaps both members' roles atomically |
| FEAT-09.SPEC-010 | Invitation & Membership Validation Rules | Logic/Rule | Maya, Sam | Governs contact-detail requirement, re-invite blocking, the 14-day expiry window, and the accept/revoke/expiry race resolution |
| FEAT-09.SPEC-011 | Household Invitations & Membership Authorization Rules | Logic/Rule | All | Governs who can send, revoke, or hand over invitations, who can leave on their own, and the unauthorized-visitor experience |
| FEAT-09.SPEC-012 | Invitation Accepted Confirmation | Notification | Maya | Tells the organiser once an invited adult has accepted and joined |
| FEAT-09.SPEC-013 | Member Left Household Notification | Notification | Maya | Tells the organiser when an other adult member leaves on their own |
| FEAT-09.SPEC-014 | Organiser Hand-Over Request Notification | Notification | Sam | Tells the chosen adult member that the organiser has asked them to accept the organiser role |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Send an invitation | FEAT-09.SPEC-001, FEAT-09.SPEC-010, FEAT-09.SPEC-011 | Invitations Manager screen creates the invitation; validation governs contact-detail and re-invite rules; authorization restricts the action to Maya | Phase 2 (Explicit) |
| Accept an invitation | FEAT-09.SPEC-002, FEAT-09.SPEC-007, FEAT-09.SPEC-012 | Acceptance screen; processing automation creates the Member Profile and hands off to onboarding; organiser receives a confirmation | Phase 2 (Explicit) |
| Manage outstanding invitations | FEAT-09.SPEC-001, FEAT-09.SPEC-006 | Invitations Manager lists, resends, and revokes; expiry automation ages out unaccepted invitations | Phase 2 (Explicit) |
| Hand over the organiser role | FEAT-09.SPEC-003, FEAT-09.SPEC-004, FEAT-09.SPEC-009, FEAT-09.SPEC-014 | Initiation screen, recipient acceptance screen, the processing automation that transfers the role, and the request notification | Phase 2 (Explicit) |
| Leave the household | FEAT-09.SPEC-005, FEAT-09.SPEC-008, FEAT-09.SPEC-013 | Leave-confirmation screen, departure processing (anonymise ratings, remove profile), and the organiser's departure notification | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a single Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-09.SPEC-006 | Invitation Expiry | Phase 4 (Trigger-Response — time-based trigger) | The Validation & Limits field states invitations "expire after 14 days"; nothing in the explicit capabilities names the automatic process that ages a Sent invitation to Expired |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | Phase 4 (Trigger-Response, cross-entity and cross-feature effect) | Acceptance is a cross-entity effect (Invitation status change creates a new Member Profile) and a cross-feature hand-off (FEAT-15 onboarding, per XBR-18) — too much processing logic to stay inline in the Acceptance screen |
| FEAT-09.SPEC-008 | Member Departure Processing | Phase 4 (Trigger-Response, cross-entity effect) | Leaving anonymises Ratings and removes the Member Profile from active membership (XBR-16) — a cross-entity effect that belongs off the Leave Household screen |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | Phase 4 (Trigger-Response, cross-entity effect) | Hand-over acceptance updates the Household's organiser field and both members' roles together, guaranteeing XBR-15's "exactly one organiser" invariant — this atomic, cross-entity update does not belong inline in either hand-over screen |
| FEAT-09.SPEC-010 | Invitation & Membership Validation Rules | Phase 5 (Rule-Constraint Discovery) | Contact-detail requirement, re-invite blocking, the 14-day expiry window, hand-over eligibility (active adult member who must accept), and the accept/revoke/expiry race resolution are conditional rules shared across SPEC-001, SPEC-002, SPEC-003, SPEC-006, and SPEC-007 — crosses the standalone-spec threshold |
| FEAT-09.SPEC-011 | Household Invitations & Membership Authorization Rules | Phase 5 (Rule-Constraint Discovery — authorization) | Every screen in this feature behaves differently by role (Maya Full on Household Invitations, Sam None for sending/revoking but Own-only for leaving, both Jordan rows and Riley None, unauthorized visitors link-only) — a cross-cutting rule set shared, not duplicated, per screen |
| FEAT-09.SPEC-012 | Invitation Accepted Confirmation | Phase 4 (Notification surfacing) | The Communications field states "a confirmation to the organiser once it is accepted" — a message with a defined audience and content, not a same-screen toast |
| FEAT-09.SPEC-013 | Member Left Household Notification | Phase 4 (Notification surfacing) | The Communications field states "the organiser is also told when a member leaves" — a distinct message from the acceptance confirmation |
| FEAT-09.SPEC-014 | Organiser Hand-Over Request Notification | Phase 4 (Notification surfacing) | The Communications field states "the new organiser is asked to accept a hand-over" — the recipient must be alerted to open FEAT-09.SPEC-004, which is a delivery event distinct from the acceptance screen itself |

## Entity-Lifecycle Coverage Matrix

**Entity: Invitation**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-09.SPEC-001 | Organiser enters a contact detail and sends a new invitation | Also re-created functionally by "resend" on an expired invitation, per Primary Flows & Alternates |
| Read (single) | FEAT-09.SPEC-002 | Invitation Acceptance screen loads the invitation addressed to the visitor's link | -- |
| Read (list) | FEAT-09.SPEC-001 | Invitations Manager lists every outstanding and accepted invitation for the household | Empty state: a plain "invite someone" prompt when no invitations exist (States field) |
| Update | FEAT-09.SPEC-001, FEAT-09.SPEC-006, FEAT-09.SPEC-007 | Organiser revokes or resends (SPEC-001); status auto-transitions to Expired (SPEC-006); status transitions to Accepted (SPEC-007) | All transitions governed by FEAT-09.SPEC-010 |
| Delete/Archive | N/A | Invitation records are never hard-deleted; every invitation is retained with its terminal status (Accepted, Revoked, or Expired) as household history for the life of the account (assumptions-constraints.md ASMP-24) — an explicit non-goal, no purge policy applies | -- |
| State Transition | FEAT-09.SPEC-001, FEAT-09.SPEC-006, FEAT-09.SPEC-007, FEAT-09.SPEC-010 | Sent → Accepted / Revoked / Expired; an Expired invitation can be resent as a fresh Sent invitation; a race between acceptance and revocation/expiry resolves reject-with-refresh (whichever change lands first wins) | -- |

**Entity: Member Profile (this feature's operations only — full lifecycle owned jointly with FEAT-01/FEAT-18)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-09.SPEC-007 | Invitation acceptance creates a new Member Profile with Other Adult Member access | Other creation paths (organiser, kid profiles) belong to FEAT-01 |
| Read (single) | N/A — owned by FEAT-01 (Member Profile Detail) | This feature only reads member records in list form to support its own decisions | -- |
| Read (list) | FEAT-09.SPEC-001, FEAT-09.SPEC-003 | Invitations Manager checks existing members to block re-inviting an already-active member; Hand-Over Initiation lists active adult members as eligible recipients | -- |
| Update | FEAT-09.SPEC-008, FEAT-09.SPEC-009 | Departure processing sets status to Left; hand-over processing swaps member_type between Organiser and Other Adult Member for both parties | -- |
| Delete/Archive | FEAT-09.SPEC-008 | Soft removal: on leaving, the profile's status is set to Left and it is hidden from the active member list; no restore path — a re-invited former member onboards again rather than being restored to old data (XBR-18); no cascade to the profile's Dietary Rules (member-removal cascade is FEAT-18's concern per XBR-16); retained indefinitely as historical record consistent with the household's kept history (ASMP-24) | Distinct from FEAT-18's hard removal path, which this feature never performs |
| State Transition | FEAT-09.SPEC-008, FEAT-09.SPEC-009 | Invited → Active is set at creation (SPEC-007); Active → Left is set on leaving (SPEC-008); Organiser ↔ Other Adult Member is set on hand-over (SPEC-009) | -- |

**Entity: Rating (this feature's operation only — full lifecycle owned by FEAT-12)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Update | FEAT-09.SPEC-008 | A leaving member's ratings are anonymised so they influence future plans without being attributed to the departed member | The only Rating operation this feature performs; create/read/other updates belong to FEAT-12 |

**Entity: Household (this feature's operation only — full lifecycle owned by FEAT-01/FEAT-18)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Update | FEAT-09.SPEC-009 | The Household's organiser field is repointed to the newly accepted recipient | The only Household field this feature ever changes; every other Household field is FEAT-01's or FEAT-16's |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-09.SPEC-001, FEAT-09.SPEC-003 | Screens read the current organiser to gate access and to exclude the organiser from the hand-over recipient list |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Organiser submits a contact detail to send an invitation | Contact detail and already-active-member checks run before the invitation is created | Standalone Logic/Rule | FEAT-09.SPEC-010 |
| Invitation sends successfully | Confirmation shown within a couple of seconds; entered contact detail preserved on failure with a retry offered | Inline in triggering screen | FEAT-09.SPEC-001 |
| Invitation reaches 14 days unaccepted | Status automatically transitions to Expired | Standalone Automation | FEAT-09.SPEC-006 |
| Invited adult opens an accepted/revoked/expired link | "This invitation is no longer valid" message shown; no household data exposed | Standalone Logic/Rule | FEAT-09.SPEC-010, FEAT-09.SPEC-011 |
| Invited adult accepts a still-valid invitation | New Member Profile created with Other Adult Member access; invitee routed to first-use onboarding | Standalone Automation | FEAT-09.SPEC-007 |
| Invitation is accepted | Organiser is sent a confirmation | Standalone Notification | FEAT-09.SPEC-012 |
| Acceptance races a revocation or expiry | Reject-with-refresh: whichever state change lands first wins; the loser sees the current, correct state | Standalone Logic/Rule | FEAT-09.SPEC-010 |
| Organiser selects a recipient and starts a hand-over | Recipient is alerted to accept | Standalone Notification | FEAT-09.SPEC-014 |
| Recipient accepts the hand-over | Household's organiser field and both members' roles are updated together, atomically | Standalone Automation | FEAT-09.SPEC-009 |
| Recipient declines the hand-over | Organiser keeps the role; a clear message is shown; organiser may choose a different recipient | Inline in triggering screen | FEAT-09.SPEC-004 |
| Organiser tries to leave or delete their own account without having handed over the role | Action is blocked; organiser is directed to hand over the role or delete the household through FEAT-18 | Standalone Logic/Rule | FEAT-09.SPEC-011 |
| Other adult member confirms leaving | Ratings are anonymised, Member Profile is soft-removed from active membership, and future plans stop accounting for them | Standalone Automation | FEAT-09.SPEC-008 |
| Member leaves | Organiser is sent a notification | Standalone Notification | FEAT-09.SPEC-013 |
| A leaving or removed member's future plan and grocery-list accounting | Plan generation and the grocery list stop counting that member | Cross-feature — FEAT-03/FEAT-06 responsibility, this feature only marks the member departed | FEAT-03, FEAT-06 (logged in touchpoints) |

## Shared Context

**Shared Entities:**
- Invitation — created and updated (revoke, resend) by FEAT-09.SPEC-001; read by FEAT-09.SPEC-002; updated by FEAT-09.SPEC-006 (expiry) and FEAT-09.SPEC-007 (acceptance). Fields: contact_detail, status, sent_by.
- Member Profile — created by FEAT-09.SPEC-007; read (list) by FEAT-09.SPEC-001 and FEAT-09.SPEC-003; updated by FEAT-09.SPEC-008 (departure) and FEAT-09.SPEC-009 (hand-over). This feature never edits display_name, age_band, or notification_preferences — those stay FEAT-01's and FEAT-07/FEAT-13's.
- Household.organiser — read by FEAT-09.SPEC-001/003 for gating and recipient exclusion; updated only by FEAT-09.SPEC-009.
- Rating — updated (anonymised only) by FEAT-09.SPEC-008; every other Rating operation belongs to FEAT-12.

**Shared UI Patterns:**
- Invalid-invitation state — the "this invitation is no longer valid" message (FEAT-09.SPEC-002) and the empty-invitations "invite someone" prompt (FEAT-09.SPEC-001) share the same plain, non-technical tone required by the States field; Spec Writers should describe both consistently rather than as generic errors.
- Confirmation-required destructive/consequential action — the leave-household confirmation (FEAT-09.SPEC-005) and the hand-over acceptance (FEAT-09.SPEC-004) share the same modal pattern: state plainly what will happen (ratings become anonymous influence; role and its access transfer), require an explicit affirmative tap, no default-confirmed state.
- Draft-preserving invitation form — FEAT-09.SPEC-001's send form preserves the entered contact detail and offers a retry on a failed send, and holds a drafted invitation locally when composed offline until connectivity returns (States field, Offline-degraded).

**Shared Validation:**
- FEAT-09.SPEC-010 defines contact-detail requirement, re-invite blocking, the 14-day expiry window, hand-over-recipient eligibility, and the accept/revoke/expiry race resolution. FEAT-09.SPEC-001, SPEC-002, SPEC-003, SPEC-006, and SPEC-007 all reference SPEC-010 rather than duplicating these rules.
- FEAT-09.SPEC-011 defines who may send, revoke, hand over, or leave, and the unauthorized-visitor experience. Every screen in this feature (FEAT-09.SPEC-001 through 005) references SPEC-011 for its role-gated behavior.

**Flagged product ambiguity (not resolved by this Brief):** The dependency map's XBR-16 states that a member *removed* by the organiser (FEAT-18) has their dietary rules deleted, and that a member who *leaves on their own* (this feature) keeps their ratings only as anonymous influence. Neither XBR-16 nor this feature's Stage 2 fields state whether a self-leaving member's Dietary Rules are deleted, anonymised, or retained. FEAT-09.SPEC-008 (Member Departure Processing) is scoped to Ratings and Member Profile status only; Dietary Rule disposition on self-leave is flagged here for resolution at Stage 4 or by a future synthesis pass, not decided by this Analyst.

## Internal Dependency Map

```
SPEC-001 (Household Invitations Manager) -> [organiser sends invitation] -> SPEC-010 (Validation Rules) -> [valid] -> SPEC-001 [invitation created]
SPEC-001 (Household Invitations Manager) -> [14 days pass, unaccepted] -> SPEC-006 (Invitation Expiry) -> [status expired] -> SPEC-001 [visible as Expired, resend enabled]
SPEC-001 (Household Invitations Manager) -> [organiser revokes] -> SPEC-010 (Validation Rules) -> [status revoked]
SPEC-002 (Invitation Acceptance) -> [invitee opens link] -> SPEC-010 (Validation Rules) -> [valid / invalid]
SPEC-002 (Invitation Acceptance) -> [invitee accepts, still valid] -> SPEC-007 (Invitation Acceptance Processing) -> [Member Profile created] -> FEAT-15 (Member Onboarding, first-use landing)
SPEC-007 (Invitation Acceptance Processing) -> [acceptance recorded] -> SPEC-012 (Invitation Accepted Confirmation) -> [delivered to] Maya
SPEC-003 (Organiser Hand-Over Initiation) -> [organiser selects recipient] -> SPEC-010 (Validation Rules) -> [recipient eligible] -> SPEC-014 (Hand-Over Request Notification) -> [delivered to] Sam
SPEC-014 (Hand-Over Request Notification) -> [recipient opens request] -> SPEC-004 (Organiser Hand-Over Acceptance)
SPEC-004 (Organiser Hand-Over Acceptance) -> [recipient accepts] -> SPEC-009 (Organiser Hand-Over Processing) -> [organiser field + roles updated] -> SPEC-001 (Invitations Manager now shows the new organiser's access)
SPEC-004 (Organiser Hand-Over Acceptance) -> [recipient declines] -> SPEC-003 (Organiser Hand-Over Initiation) [organiser notified, may pick another recipient]
SPEC-005 (Leave Household) -> [member confirms] -> SPEC-008 (Member Departure Processing) -> [ratings anonymised, profile soft-removed] -> SPEC-013 (Member Left Household Notification) -> [delivered to] Maya
SPEC-001 through SPEC-005 -> [gated by] -> SPEC-011 (Authorization Rules)
```

**Default Entry:** FEAT-09.SPEC-001 (Household Invitations Manager) for the organiser navigating to this feature area; FEAT-09.SPEC-002 (Invitation Acceptance) for a visitor opening an invitation link.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-09.SPEC-001 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Organiser sends a partner an invitation from within guided setup | Tap "invite" during setup (First Household Setup journey, step 5) |
| FEAT-09.SPEC-007 | Outbound | FEAT-01 (Household Setup & Member Profiles) | The new Member Profile created on acceptance joins the member list FEAT-01 manages | Invitee accepts |
| FEAT-09.SPEC-007 | Outbound | FEAT-15 (Member Onboarding) | Accepted invitee is routed directly into first-use onboarding, landing on the current week's plan and grocery list | Invitee accepts (Invite & Join Household journey, step 2–3) |
| FEAT-09.SPEC-011 | Outbound | FEAT-18 (Account & Data Management) | An organiser who wants to leave or delete their own account must first hand over the role, or delete the household through FEAT-18 (XBR-15) | Organiser attempts to leave or delete their account while still organiser |
| FEAT-09.SPEC-008 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation), FEAT-06 (Shared Grocery List) | A departed member stops being accounted for in future plan generation and list quantities | Member leaves |
| FEAT-09.SPEC-008 | Outbound | FEAT-12 (Meal Rating & Preference Learning) | A departed member's ratings continue to feed the learning model anonymously rather than being deleted | Member leaves |

## Non-Functional Notes

**Data volumes / growth:** A household holds 1–12 member profiles and may carry multiple outstanding invitations at once (Validation & Limits field); invitation history accumulates over the life of the account alongside thousands of households (assumptions-constraints.md ASMP-24).

**Responsiveness:** Sending an invitation confirms within a couple of seconds; composing an invitation requires connectivity to send, and a drafted invitation is held locally until it can be sent (States field, Loading and Offline-degraded).

**Data sensitivity / privacy:** An invitee's contact detail is personal data about a non-member of the household, used only to deliver the invitation and never sold or used for marketing (feature-dependency-map.md, Invitation Data Sensitivity; assumptions-constraints.md ASMP-14). Once accepted, the new Member Profile carries only the standard adult member data (display name, optional sign-in email) that FEAT-01 already classifies; this feature introduces no children's data.

**Compliance flags:** No children's-privacy-class concerns apply directly to this feature — invitations and hand-overs involve only adult members; general personal-data rights (export, deletion) for any member this feature creates are delivered through Account & Data Management (FEAT-18), not this feature (assumptions-constraints.md ASMP-27).

## Non-Goals

- **Household-to-household invitations and referral growth** — Excluded per scope-boundaries.md SC-09 and SC-10: growth between separate households, including any referral link or incentive, belongs to Invite Another Household (FEAT-24); this feature covers membership within one existing household only, per the feature's own [MODIFIED] rationale.
- **Inviting or logging in kid profiles** — Excluded per scope-boundaries.md SC-02: young kid profiles have no login in v1 (every Access Matrix cell None), and an older-kid limited login is a distinct, Later-phase capability (FEAT-17); neither kid row can send or receive an invitation through this feature.
- **Other adult members sending, revoking, or handing over invitations** — Excluded per the Access Matrix (Household Invitations: None for Sam) and scope-boundaries.md SC-04's role split: only Maya (Organiser) can send, revoke, or hand over; Sam can only accept an invitation addressed to him or leave on his own.
- **Restoring a re-invited former member's old data** — Excluded per XBR-18: a re-invited former member onboards again as a new Member Profile rather than being restored to their prior profile, dietary rules, or history.
- **Hard deletion or purge of Invitation records** — Intentional lifecycle decision surfaced by the CRUD matrix: every invitation is retained with its terminal status (Accepted, Revoked, Expired) as household history for the life of the account (assumptions-constraints.md ASMP-24); no purge policy applies.
- **In-product messaging between the organiser and an invitee about the invitation** — Excluded per scope-boundaries.md SC-14: the invitation itself (a shareable link or message) and its confirmations are the only communications this feature sends; general chat between members is explicitly out of scope.



# Screen Spec: Household Invitations Manager

## Overview

**Name:** Household Invitations Manager
**ID:** FEAT-09.SPEC-001
**Type:** Screen
**Purpose:** The organiser sends a new invitation and sees, resends, or revokes every outstanding and accepted invitation for the household.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Sending a new invitation by entering a contact detail
- Listing every outstanding (Sent), Accepted, Revoked, and Expired invitation for the household
- Revoking a Sent invitation
- Resending (re-creating) an Expired invitation
- Displaying the generated shareable invitation link/message for the organiser to send through her own channel of choice

**Non-Goals:**
- Accepting an invitation -- handled by FEAT-09.SPEC-002 (Invitation Acceptance); this screen is organiser-only
- Automatically delivering the invitation by email or text on the product's behalf -- excluded per scope-boundaries.md SC-14 (no in-product messaging layer) and the feature's zero Integration-spec footprint (feature-dependency-map.md, External Touchpoints): the product generates a unique shareable link and the organiser sends it herself, exactly as household-to-household referral links work in FEAT-24
- Automatically expiring a Sent invitation -- owned by FEAT-09.SPEC-006 (Invitation Expiry); this screen only reflects the resulting status
- Organiser hand-over -- a distinct action owned by FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), reached from elsewhere in household settings, not from this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Invite a partner" during guided setup, step 5 | None -- screen opens directly into the send-invitation form, empty invitation list |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens "Household Invitations" from settings | None -- screen opens to the invitation list |
| Default entry | Organiser navigates to this feature area directly | None |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Organiser taps the "No eligible recipients" link | None -- screen opens to the invitation list so she can invite an adult first |
| FEAT-18.SPEC-002 (Remove Member Profile) | Organiser taps the "invite a member" link from that screen's empty state | None -- screen opens to the invitation list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Send, resend, and revoke invitations | -- |
| Sam (Other Adult Member) | No | No | Screen entry point is not shown in household settings; a direct navigation attempt shows "Only the organiser can manage invitations" and returns Sam to FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- young kid profiles have no login and no household-settings access of any kind |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Household Invitations is outside the older-kid login's entitlements (Access Matrix: None) |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14); this screen has no operator path |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In); no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; any in-progress, unsent invitation draft is preserved and restored to this screen after re-authentication |

## Layout and Content

**Header:** Screen title "Household Invitations" with a back arrow (returns to FEAT-01.SPEC-010) and a "+ Invite" action button (top right).

**Body:**
- **Send Invitation panel** (shown expanded when the list is empty, or opened by "+ Invite" otherwise): a single text field labeled "Contact detail (email or phone)" and a "Send Invitation" button. This is a label for the organiser's own reference and for re-invite blocking (FEAT-09.SPEC-010) -- the product does not deliver anything to this contact detail itself.
- **Invitation List**: one row per invitation, most recent first, each showing:
  - The contact detail entered for that invitation
  - A status badge: Sent, Accepted, Revoked, or Expired
  - The date sent
  - Row actions, shown per status: Sent shows "Revoke" and "Copy link"; Expired shows "Resend"; Accepted and Revoked show no actions (history only)
- **Shareable link confirmation** (appears immediately after a successful send): a read-only field containing the generated invitation link and a "Copy link" button, with the text "Share this with {contact_detail} yourself -- we don't send it for you."

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Send Invitation panel and list stack full width, single column; row actions collapse into a per-row overflow menu.
- **Medium size class and above:** Content area caps at a consistent platform-wide width and is horizontally centered; row actions remain inline (no overflow menu needed); no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 (Household Settings Hub) | Screen closes | Standard navigation transition |
| "+ Invite" button | Tap | Opens the Send Invitation panel | Panel expands | Panel animates open, focus moves to the contact detail field |
| Contact detail field | Type | Captures text input | Field shows entered text | Standard input focus state |
| Contact detail field | Blur | Validates via FEAT-09.SPEC-010 | Error state if invalid | "Enter an email address or phone number" below the field |
| "Send Invitation" button | Tap | 1. Validate contact detail and re-invite blocking via FEAT-09.SPEC-010. 2. If valid, create the Invitation (status: Sent) and generate its shareable link. | Button shows loading state; on success the new invitation appears at the top of the list | Success (within a couple of seconds): "Invitation ready to share" confirmation with the Shareable link confirmation panel. Failure: inline error, entered contact detail preserved, "Try again" retry offered |
| "Copy link" button (per Sent row, and in the send confirmation) | Tap | Copies the invitation's shareable link to the clipboard | None | "Link copied" toast |
| "Revoke" button (Sent rows only) | Tap | Opens a confirmation dialog: "Revoke this invitation? {contact_detail} will no longer be able to join with this link." | Dialog appears | Confirm triggers the revoke (via FEAT-09.SPEC-010); Cancel closes with no change |
| Revoke confirmation | Tap "Revoke" | Sets the invitation's status to Revoked via FEAT-09.SPEC-010 | Row status badge updates to Revoked, row actions clear | Toast: "Invitation revoked" |
| "Resend" button (Expired rows only) | Tap | Creates a fresh Sent invitation to the same contact detail via FEAT-09.SPEC-010, replacing the Expired one at the top of the list | New Sent row appears; the prior Expired row remains as history | Toast: "New invitation ready to share" with the Shareable link confirmation panel |
| "Send Invitation" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> "+ Invite" -> contact detail field (when panel open) -> "Send Invitation" -> invitation list rows, each row's actions in reading order.
- **Validation announcements:** Field errors are announced to assistive technology and programmatically associated with the contact detail field.
- **Dynamic announcements:** "Invitation ready to share," "Link copied," "Invitation revoked," and "New invitation ready to share" are all announced as they appear.
- **Keyboard alternatives:** Every action, including Copy link, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Plain prompt: "No invitations yet -- invite someone to share the plan and list with you." Send Invitation panel shown expanded by default. | Household has zero invitations of any status | An invitation is sent |
| Loaded (with invitations) | Invitation List populated, Send Invitation panel collapsed behind "+ Invite" | One or more invitations exist | Always the resting state once invitations exist |
| Loading (list) | Skeleton rows in place of the invitation list | Screen first opens, list is being fetched | List loads successfully or fails |
| List load error | Error banner: "Couldn't load invitations. Try again." with a Retry button | Fetching the invitation list fails | User taps Retry and the fetch succeeds |
| Sending | "Send Invitation" button shows loading state, contact detail field disabled | User taps Send Invitation with valid input | Send completes or fails |
| Send error | Inline error banner above the form; entered contact detail preserved; "Try again" retry offered | Sending the invitation fails | User taps Retry and the send succeeds, or corrects the contact detail |
| Offline/Degraded | Banner: "You're offline -- this invitation will send once you're back online." The contact detail field remains editable; tapping Send queues the invitation locally rather than failing | Connectivity lost while composing an invitation | Connectivity restored -- queued invitation sends automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules). This screen applies validation on field blur and on Send Invitation submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| Successful send, resend, revoke | Stays on this screen (list updates in place) | -- |

## Data Model

**Creates:** Invitation -- contact_detail (organiser input), status (set to Sent), sent_by (the current organiser, Maya). A shareable link is generated for the new record.
**Reads:** Invitation -- all fields, filtered to this household, for every status, ordered newest first. Household.organiser and the household's active Member Profile list (read to run the re-invite-blocking check in FEAT-09.SPEC-010).
**Updates:** Invitation -- status (Sent to Revoked on revoke).
**Deletes:** None -- invitations are never hard-deleted (feature-overview.md, Entity-Lifecycle Coverage Matrix; assumptions-constraints.md ASMP-24).

## Business Rules

- Contact-detail requirement, format, and re-invite blocking are governed by FEAT-09.SPEC-010 -- this screen never restates them.
- Only Maya (Organiser) reaches this screen at all, per FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules).
- Resending an Expired invitation creates a new Sent invitation to the same contact detail rather than reviving the expired record; the expired record remains as household history (feature-overview.md, Entity-Lifecycle Coverage Matrix).
- A household may hold multiple outstanding (Sent) invitations at once (feature-overview.md, Validation & Limits).

## Edge Cases

- **Organiser revokes an invitation that was accepted moments earlier on another device** -- Revoke is rejected with refresh: the row updates to show Accepted and the toast "This invitation was already accepted" appears instead of "Invitation revoked." Resolution: reject-with-refresh, per the dependency map's Contention note for the Invitation entity.
- **Organiser resends an invitation that expired moments ago vs. one that is still Sent** -- "Resend" is shown only for rows already in Expired status; if the underlying invitation is still Sent when tapped (stale row), the action is rejected with refresh and the row updates to its current status.
- **Organiser taps Revoke twice rapidly** -- Second tap is ignored while the first revoke is in progress (dialog and button disabled).
- **Organiser sends an invitation to a contact detail matching an existing active member** -- Blocked per FEAT-09.SPEC-010 with the error "This person is already a member of your household."
- **Organiser navigates away with the Send Invitation panel open and unsent text** -- No confirmation dialog; the draft contact detail is not destructive state and is simply discarded, consistent with the screen's non-blocking send flow.
- **Two devices signed in as Maya both viewing this screen** -- The list is a snapshot refreshed on entry and after each action on this screen; an invitation sent, revoked, or resent on one device is not live-pushed to the other, so the other device's next action against a changed row hits the reject-with-refresh path above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Contact-detail validation, re-invite blocking, and the accept/revoke/expiry race resolution |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Restricts this screen to Maya (Organiser) |
| FEAT-09.SPEC-006 (Invitation Expiry) | Affects (inbound) | Automatically transitions Sent rows to Expired, shown on this screen |
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Affects (inbound) | Transitions a row to Accepted, shown on this screen |
| FEAT-09.SPEC-002 (Invitation Acceptance) | References (outbound) | The shareable link this screen generates opens that screen for the invitee |
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound) | Guided setup step 5 opens directly into this screen's send form |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point from settings; back arrow returns there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invitation_sent | origin (new / resend), entry source (guided setup / settings) | An invitation is successfully created (new or resend) | N/A -- no Stage 2 metric measures the invitation-sending step itself; the connected metric, "Household Member Participation," measures the invited adult's subsequent joining and use, tracked at acceptance (FEAT-09.SPEC-007) |
| invitation_revoked | time outstanding before revoke | Organiser confirms a revoke | N/A -- no Stage 2 metric measures revocations; retained as a funnel-diagnostic signal |
| invitation_send_failed | failure reason | A send attempt fails | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-001-AC-01:** Given Maya is on the Household Invitations Manager with no existing invitations, when the screen loads, then she sees the empty-state prompt "No invitations yet -- invite someone to share the plan and list with you." with the Send Invitation panel already expanded.

**FEAT-09.SPEC-001-AC-02:** Given Maya enters a valid email address and taps "Send Invitation", when the send succeeds, then a new Sent row appears at the top of the list within a couple of seconds and the Shareable link confirmation panel appears with a "Copy link" button.

**FEAT-09.SPEC-001-AC-03:** Given Maya enters a contact detail matching an active member of her household, when she taps "Send Invitation", then the error "This person is already a member of your household." appears and no invitation is created.

**FEAT-09.SPEC-001-AC-04:** Given Maya taps "Revoke" on a Sent invitation and confirms, when the revoke completes, then the row's status badge updates to Revoked and the toast "Invitation revoked" appears.

**FEAT-09.SPEC-001-AC-05:** Given Maya sees an Expired invitation, when she taps "Resend", then a new Sent row is created for the same contact detail and the toast "New invitation ready to share" appears with a fresh shareable link.

**FEAT-09.SPEC-001-AC-06:** Given Sam attempts to navigate directly to this screen, when the request is made, then he sees "Only the organiser can manage invitations" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-001-AC-07:** Given Maya's send attempt fails due to a connectivity error, when the failure occurs, then her entered contact detail is preserved and a "Try again" retry option appears.

**FEAT-09.SPEC-001-AC-08:** Given Maya loses connectivity while composing an invitation, when she taps "Send Invitation", then the banner "You're offline -- this invitation will send once you're back online." appears and the invitation sends automatically once connectivity returns.

**FEAT-09.SPEC-001-AC-09:** Given Maya taps "Revoke" on a Sent invitation that was accepted moments earlier from another device, when the revoke request completes, then she sees "This invitation was already accepted" and the row refreshes to show Accepted status.

**FEAT-09.SPEC-001-AC-10:** Given the invitation list fails to load, when the screen opens, then the error banner "Couldn't load invitations. Try again." appears with a Retry button.

**FEAT-09.SPEC-001-AC-11:** Given Maya taps "Copy link" on a Sent invitation's row, when the tap registers, then the invitation's shareable link is copied and the toast "Link copied" appears.

**FEAT-09.SPEC-001-AC-12:** Given Maya double-taps "Revoke" rapidly on the confirmation dialog, when the first tap is already processing, then the second tap has no additional effect and only one revoke is applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 7 (empty, loaded, loading, list load error, sending, send error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Invitation Acceptance

## Overview

**Name:** Invitation Acceptance
**ID:** FEAT-09.SPEC-002
**Type:** Screen
**Purpose:** An invited adult opens their invitation link, creates their sign-in, and accepts the invitation to join the household -- or sees why the invitation is no longer valid.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Loading the specific invitation addressed to the link the visitor opened
- Showing the still-valid invitation's content (who is inviting them, to what) and an Accept action
- Creating the invitee's sign-in credentials (display name, sign-in email, protected sign-in) as part of accepting
- Showing the "no longer valid" experience for an Accepted, Revoked, or Expired invitation, or a malformed/unknown link
- Handing off to FEAT-09.SPEC-007 (Invitation Acceptance Processing) on a successful accept

**Non-Goals:**
- Sending or managing invitations -- owned by FEAT-09.SPEC-001 (Household Invitations Manager); this screen is invitee-only
- Creating the Member Profile record or routing into onboarding -- owned by FEAT-09.SPEC-007, which this screen triggers but does not perform itself
- General sign-in for an adult who already has an account -- owned by FEAT-01.SPEC-001 (Account Sign-Up & Sign-In); this screen exists only for a first-time invitee opening a link addressed to them
- Password recovery for an invitee who already accepted a prior invitation and forgot their sign-in -- owned by FEAT-01.SPEC-002 (Password Recovery), a general account concern outside this feature

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Invitation link (external) | Invitee opens the shareable link generated by FEAT-09.SPEC-001 | The specific Invitation record referenced by the link |
| Invitation link, already signed in as a different account | Invitee opens the link while signed in to an unrelated account | Same invitation record; screen shows the account-mismatch edge case below |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Unauthorized visitor (the invitee, not yet a member) | Yes -- only the specific invitation addressed to the link they hold | Accept (if the invitation is still Sent) | -- |
| Maya (Organiser) | No | No | Organiser reaches invitation management through FEAT-09.SPEC-001, not this screen; opening someone else's invitation link while signed in as Maya shows the account-mismatch edge case |
| Sam (Other Adult Member, already a member) | No | No | An already-active member opening any invitation link sees "You're already a member of this household" and is routed to the current plan, not this acceptance flow |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles are never invited through this feature (scope-boundaries.md SC-02) |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- kid profiles are never invited through this feature |
| Riley (Operator, support) | No | No | N/A -- operator support access has no path through this screen (FEAT-22, XBR-14) |
| Expired session (mid-acceptance) | Partial -- invitation details remain visible | No, until re-authenticated | Dialog "Your session has expired. Sign in to continue."; entered sign-in details are preserved and the invitation is not consumed until acceptance is re-confirmed |

## Layout and Content

**Header:** Product name/logo, centered. No back navigation (this is a direct-link entry screen).

**Body -- valid invitation:**
- A short line naming the inviting household and organiser: "{organiser display name} invited you to join their household on Plateful."
- A brief explanation of what joining means: seeing the same weekly plan and shared grocery list, with Other Adult Member access
- A sign-in creation form: Display Name (text input, required), Sign-in Email (text input, required, pre-filled and editable if the invitation's contact detail is an email), Password (with a visible strength indicator, required)
- "Accept & Join" button

**Body -- invalid invitation (Accepted, Revoked, Expired, or link does not resolve to any invitation):**
- Plain message: "This invitation is no longer valid." with a one-line reason where known ("It was already accepted." / "It was revoked by the organiser." / "It expired after 14 days.") -- the same plain, non-technical tone as the Household Invitations Manager's empty state (Feature Breakdown Brief, Shared UI Patterns)
- No household data of any kind is shown in this state
- A link to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) for a visitor who wants to create their own household instead

**Footer:** Legal text line linking to terms and privacy information (static content).

### Responsive Behavior

- **Compact breakpoint:** Form fields and message stack full width.
- **Medium size class and above:** Content area caps at a consistent platform-wide narrow width and is horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Display Name field | Type | Captures text input | Field shows entered text | Standard input focus state |
| Display Name field | Blur | Validates via FEAT-09.SPEC-010 (required, non-empty) | Error state if invalid | "Enter your name" below the field |
| Sign-in Email field | Type / Blur | Captures and validates email format via FEAT-01.SPEC-014 | Error state if invalid | "Enter a valid email address" below the field |
| Password field | Type | Captures the password; strength indicator updates live | Indicator reflects strength | Live indicator update, no blocking feedback while typing |
| Password field | Blur | Validates minimum protection requirement via FEAT-01.SPEC-014 | Error state if requirement not met | "Choose a password that is harder to guess" below the field |
| "Accept & Join" button | Tap | 1. Validate all fields. 2. Re-check invitation validity via FEAT-09.SPEC-010 (race resolution). 3. If still valid, trigger FEAT-09.SPEC-007 (Invitation Acceptance Processing). | Button shows loading state | Success: routed into first-use onboarding (FEAT-15). Failure (validity race lost): switches to the invalid-invitation body with the specific reason; entered form data is discarded since there is nothing left to join |
| "Accept & Join" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Create your own household instead" link (invalid-invitation body only) | Tap | Navigate to FEAT-01.SPEC-001 | Screen changes | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Display Name -> Sign-in Email -> Password -> "Accept & Join". In the invalid-invitation body: the message, then the "Create your own household instead" link.
- **Validation announcements:** Field errors are announced to assistive technology and programmatically associated with their field.
- **Body-switch announcement:** The transition from the valid-invitation form to the invalid-invitation message (e.g., after losing the accept/revoke/expiry race) is announced so the change is not silently missed.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton in place of the body while the invitation is fetched | Screen first opens | Invitation resolves to valid, invalid, or not-found |
| Valid invitation | Sign-in creation form shown as described in Layout and Content | The link resolves to an Invitation with status Sent | User submits Accept & Join, or the invitation becomes invalid while the screen is open (race lost) |
| Invalid invitation | "This invitation is no longer valid." message with reason | The link resolves to an Invitation with status Accepted, Revoked, or Expired, or resolves to no invitation at all | User navigates away (no further transition possible from this screen) |
| Submitting | "Accept & Join" button shows loading state, form fields disabled | User taps Accept & Join with valid input | Acceptance completes or the validity race is lost |
| Error | Inline error banner; form data preserved | Acceptance processing (FEAT-09.SPEC-007) fails for a reason other than invalidity (e.g., a transient failure) | User taps Retry and acceptance succeeds |
| Offline/Degraded | Banner: "You're offline. Joining a household needs a connection -- try again once you're back online." Form remains visible and editable but "Accept & Join" is disabled | Connectivity is lost while this screen is open | Connectivity returns -- banner clears and the button re-enables |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) for invitation validity and the display name field, and by FEAT-01.SPEC-014 (Household & Member Field Validation Rules) for sign-in email and password format. This screen applies field validation on blur and full validation on Accept & Join submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful acceptance | First-use landing (routed by FEAT-09.SPEC-007) | FEAT-15 (Member Onboarding) |
| "Create your own household instead" tap | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | FEAT-01 |

## Data Model

**Creates:** None directly -- the sign-in credentials and Member Profile are created by FEAT-09.SPEC-007 once this screen's Accept & Join succeeds.
**Reads:** Invitation -- status and the inviting organiser's display name (Member Profile.display_name for the household's organiser), to render the valid-invitation body.
**Updates:** None directly -- status transition to Accepted is performed by FEAT-09.SPEC-007.
**Deletes:** None.

## Business Rules

- Invitation validity (Sent vs. Accepted/Revoked/Expired) and the accept/revoke/expiry race resolution are governed by FEAT-09.SPEC-010 -- this screen re-checks validity at Accept & Join time, not only at page load.
- A visitor who already has an account for a different household sees the account-mismatch edge case rather than silently joining a second household, consistent with scope-boundaries.md SC-03 (one household per account in v1).
- Accepting creates a Member Profile with Other Adult Member access exactly once per accepted invitation (XBR-18); this screen cannot be re-submitted after a successful accept.

## Edge Cases

- **Invitation is revoked or expires between page load and Accept & Join tap** -- Reject-with-refresh: the accept fails, the screen switches to the invalid-invitation body with the current reason ("It was revoked by the organiser." or "It expired after 14 days."), and no Member Profile is created. Resolution: reject-with-refresh, per the dependency map's Contention note for the Invitation entity.
- **Two people open the same link and both tap Accept & Join at effectively the same time** -- Only the first accept to be recorded succeeds; the second sees the invalid-invitation body with "It was already accepted."
- **Visitor is already signed in as a member of a different household** -- The screen shows: "You're signed in as {current account email}, which already belongs to a household. Sign out to accept this invitation with a different account." with a sign-out action; the current session is never silently switched.
- **Visitor is already an active member of this same household (e.g., re-opens an old link after joining)** -- "You're already a member of this household." with a link to the current plan, not the acceptance form.
- **Link does not resolve to any invitation (malformed or tampered)** -- Same invalid-invitation body, generic reason: "This invitation is no longer valid." with no further detail, so no information about whether any invitation ever existed is leaked.
- **Visitor navigates away mid-entry and returns via the same link later** -- The invitation's current status is re-fetched fresh; if still Sent, the form is empty again (no draft persistence for this one-time flow, since the invitation itself is durable and re-openable until it changes state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Household Invitations Manager) | Navigation (inbound) | Generates the shareable link that opens this screen |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Invitation validity and the accept/revoke/expiry race resolution |
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Triggers (outbound) | Accept & Join triggers Member Profile creation and onboarding routing |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Sign-in email and password format rules |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (outbound) | "Create your own household instead" link |
| FEAT-15 (Member Onboarding) | Navigation (outbound) | Successful acceptance routes here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invitation_link_opened | invitation status at open time (sent / accepted / revoked / expired / not_found) | The screen resolves the invitation link | N/A -- no Stage 2 metric measures link opens directly; retained as a funnel-diagnostic signal ahead of the metric-bearing acceptance event |
| invitation_accept_attempted | -- | Visitor taps "Accept & Join" | N/A -- diagnostic signal only |
| invitation_accept_failed | reason (validity_race_lost / processing_error) | Acceptance fails after submission | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-002-AC-01:** Given an invited visitor opens a Sent invitation's link, when the screen loads, then it shows "{organiser display name} invited you to join their household on Plateful." with the sign-in creation form.

**FEAT-09.SPEC-002-AC-02:** Given the visitor fills in a valid display name, sign-in email, and password, when they tap "Accept & Join", then FEAT-09.SPEC-007 processes the acceptance and the visitor is routed into first-use onboarding (FEAT-15).

**FEAT-09.SPEC-002-AC-03:** Given the visitor opens a link to an already-Accepted invitation, when the screen loads, then it shows "This invitation is no longer valid. It was already accepted." with no household data.

**FEAT-09.SPEC-002-AC-04:** Given the visitor opens a link to a Revoked invitation, when the screen loads, then it shows "This invitation is no longer valid. It was revoked by the organiser."

**FEAT-09.SPEC-002-AC-05:** Given the visitor opens a link to an Expired invitation, when the screen loads, then it shows "This invitation is no longer valid. It expired after 14 days."

**FEAT-09.SPEC-002-AC-06:** Given the visitor is filling in the form when the organiser revokes the invitation from another device, when the visitor then taps "Accept & Join", then the accept fails and the screen switches to "This invitation is no longer valid. It was revoked by the organiser." with no Member Profile created.

**FEAT-09.SPEC-002-AC-07:** Given the visitor is signed in as a member of a different household, when they open an invitation link, then they see "You're signed in as {current account email}, which already belongs to a household. Sign out to accept this invitation with a different account."

**FEAT-09.SPEC-002-AC-08:** Given the visitor leaves the display name field empty, when they tap "Accept & Join", then the display name field shows "Enter your name" and the acceptance does not proceed.

**FEAT-09.SPEC-002-AC-09:** Given the visitor loses connectivity while filling in the form, when they attempt to tap "Accept & Join", then the banner "You're offline. Joining a household needs a connection -- try again once you're back online." appears and the button is disabled.

**FEAT-09.SPEC-002-AC-10:** Given the visitor opens a link that does not resolve to any invitation, when the screen loads, then it shows the generic "This invitation is no longer valid." message with no further detail.

**FEAT-09.SPEC-002-AC-11:** Given a member who already belongs to this household opens the invitation link again, when the screen loads, then it shows "You're already a member of this household." with a link to the current plan instead of the acceptance form.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (loading, valid, invalid, submitting, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Organiser Hand-Over Initiation

## Overview

**Name:** Organiser Hand-Over Initiation
**ID:** FEAT-09.SPEC-003
**Type:** Screen
**Purpose:** The organiser picks an active adult member and starts handing over the organiser role to them.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Listing every Active adult member eligible to receive the organiser role (excludes the organiser herself, excludes kid profiles, excludes Invited/Left/Removed members)
- Selecting a recipient and sending the hand-over request (triggers FEAT-09.SPEC-014, the recipient's notification)
- Showing the status of an outstanding hand-over request (pending, or the outcome of a decline)
- Cancelling an outstanding hand-over request before the recipient responds

**Non-Goals:**
- The recipient's accept/decline decision -- owned by FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance); this screen only initiates and tracks the request
- Actually transferring the organiser role and swapping member types -- owned by FEAT-09.SPEC-009 (Organiser Hand-Over Processing), which runs only after the recipient accepts
- Household deletion as an alternative to handing over -- owned by FEAT-18 (Account & Data Management); this screen offers hand-over only, per XBR-15's two paths for an organiser who wants to leave or delete their own account

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens "Hand over organiser role" from settings | None |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Recipient declines the hand-over | The declined recipient's name, so the organiser sees the outcome and can pick someone else |
| FEAT-18.SPEC-004 (My Account) | Organiser taps "Hand over role" on the Deletion blocked dialog after attempting to delete her own account while still organiser (XBR-15) | A prompt explaining she must hand over the role first, linking here |
| FEAT-09.SPEC-005 (Leave Household) | Organiser attempts to leave while still organiser and follows the organiser-blocked message link | A prompt explaining she must hand over the role first |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Select a recipient, send the request, cancel a pending request | -- |
| Sam (Other Adult Member) | No | No | Screen entry point is not shown in household settings; a direct navigation attempt shows "Only the organiser can hand over the role" and returns Sam to FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles have no household-settings access of any kind |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Household Invitations is outside the older-kid login's entitlements (Access Matrix: None) |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001; no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; a selected-but-not-yet-sent recipient choice is preserved and restored after re-authentication |

## Layout and Content

**Header:** Screen title "Hand Over Organiser Role" with a back arrow (returns to FEAT-01.SPEC-010).

**Body -- no pending request:**
- A brief explanation: "Handing over makes {selected name} the organiser. You'll keep full access as an Other Adult Member -- they'll take over planning, budget, and settings."
- A list of eligible Active adult members (radio selection, one at a time)
- "Send Request" button, enabled once a recipient is selected

**Body -- pending request outstanding:**
- Status line: "Waiting for {recipient name} to accept." with the date sent
- "Cancel Request" button

**Body -- request declined (organiser's next visit after a decline):**
- Status line: "{recipient name} declined. You're still the organiser." with a "Choose Someone Else" button that returns to the no-pending-request body

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Recipient list and body stack full width, single column.
- **Medium size class and above:** Content area caps at a consistent platform-wide width and is horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 | Screen closes | Standard navigation transition |
| Recipient radio row | Tap | Selects that member as the candidate recipient | Selected row highlighted, "Send Request" enabled | Standard selection state |
| "Send Request" button | Tap | 1. Validate recipient eligibility via FEAT-09.SPEC-010. 2. If eligible, record the pending hand-over request and trigger FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification). | Button shows loading state; on success the body switches to the pending-request state | Success: "Request sent to {recipient name}." Failure: inline error, selection preserved |
| "Cancel Request" button (pending state) | Tap | Opens a confirmation dialog: "Cancel this hand-over request?" | Dialog appears | Confirm clears the pending request and returns to the no-pending-request body with the toast "Request cancelled"; Cancel closes the dialog with no change |
| "Choose Someone Else" button (declined state) | Tap | Clears the declined-request status | Body switches to the no-pending-request state | Immediate transition |
| "Send Request" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> recipient list (radio group) -> "Send Request", or -> "Cancel Request" / "Choose Someone Else" depending on state.
- **Validation announcements:** A failed send is announced along with its inline error text.
- **State-change announcements:** The switch between no-pending, pending, and declined bodies is announced so the outcome is not silently missed.
- **Keyboard alternatives:** The recipient list is a standard radio group, fully keyboard-operable; every action is reachable by keyboard.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| No eligible recipients | Message: "You need at least one other active adult member to hand over the role. Invite someone first." with a link to FEAT-09.SPEC-001 | Household has zero Active Other Adult Members | A member becomes Active (accepts an invitation) |
| No pending request | Recipient list with "Send Request" | No hand-over request is currently outstanding and none was just declined | Organiser sends a request |
| Sending | "Send Request" button shows loading state, list disabled | Organiser taps Send Request with a valid selection | Send completes or fails |
| Pending | Status line and "Cancel Request" as described in Layout and Content | A hand-over request has been sent and not yet accepted, declined, or cancelled | Recipient accepts (screen leaves this feature area), declines (switches to Declined), or organiser cancels (returns to No pending request) |
| Declined | Status line and "Choose Someone Else" as described in Layout and Content | The recipient declined the most recent request | Organiser taps "Choose Someone Else" |
| Send error | Inline error banner; recipient selection preserved | Sending the request fails for a reason other than eligibility (e.g., a transient failure) | Organiser retries and the send succeeds |
| Offline/Degraded | Banner: "You're offline -- sending a hand-over request needs a connection." Recipient selection remains but "Send Request" is disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears and the button re-enables |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules), specifically the hand-over-recipient eligibility rule (must be an Active adult member, not the current organiser).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| "No eligible recipients" link | FEAT-09.SPEC-001 (Household Invitations Manager) | -- |

## Data Model

**Creates:** Household -- pending_organiser_handover (a workflow-only marker this feature introduces to track an outstanding request: the requested recipient's Member Profile reference and the request date; it is not one of the dependency map's listed Household fields because no other feature reads or writes it, but it does not conflict with any listed field).
**Reads:** Household.organiser (to exclude the current organiser from the recipient list) and the household's Member Profile list, filtered to Active adult members (Other Adult Member type).
**Updates:** Household -- pending_organiser_handover is cleared on cancel or decline; it is cleared (and Household.organiser transferred) by FEAT-09.SPEC-009 on acceptance.
**Deletes:** None.

## Business Rules

- Only one hand-over request may be outstanding at a time (XBR-15's "exactly one organiser" invariant): the recipient list and "Send Request" are only reachable from the No pending request state.
- A recipient must be an Active adult member (Other Adult Member type) who has not left or been removed; kid profiles of any kind are never eligible (feature-overview.md, Validation & Limits).
- Cancelling a pending request does not notify the recipient beyond removing their pending action from FEAT-09.SPEC-004 -- there is no cancellation notification, since the recipient simply no longer has anything to act on.
- This screen is one of the two paths XBR-15 requires before an organiser may leave or delete her own account; the other is household deletion through FEAT-18.

## Edge Cases

- **Recipient's status changes to Left or Removed while a request is pending** -- The pending request is silently cancelled and the screen returns to the No pending request state with the note "{recipient name} is no longer an active member; the request was cancelled."
- **Organiser cancels the request at the same moment the recipient accepts it** -- Reject-with-refresh: whichever change is recorded first wins. If the accept was recorded first, the cancel is rejected and the screen shows the hand-over as completed (organiser role already transferred, this screen no longer applies to her). If the cancel was recorded first, the recipient's accept attempt fails per FEAT-09.SPEC-004's own edge case.
- **Two eligible recipients exist and the organiser selects one while another is added mid-session** -- The recipient list is a snapshot loaded on screen entry; a newly Active member does not appear until the organiser reopens the screen, with no effect on an already-selected candidate.
- **Organiser taps "Send Request" twice rapidly** -- Second tap is ignored while the first send is in progress (button in loading state).
- **Household has exactly one eligible recipient and they decline** -- The Declined state is shown exactly as with any recipient; "Choose Someone Else" returns to a recipient list that, if still only one eligible member exists, shows that same person available to re-select.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Hand-over-recipient eligibility |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Restricts this screen to Maya (Organiser) |
| FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification) | Triggers (outbound) | "Send Request" fires the recipient's notification |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Navigation (inbound) | A decline returns the organiser here with the outcome |
| FEAT-09.SPEC-009 (Organiser Hand-Over Processing) | Affects (inbound) | An acceptance elsewhere transfers the role, ending this screen's relevance to the former organiser |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point from settings; back arrow returns there |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | XBR-15's own-account deletion gate (Deletion blocked state) links here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric | 
|-------|-----------|--------------|-----------------|
| organiser_handover_requested | -- | Organiser sends a hand-over request | N/A -- no Stage 2 metric measures hand-over activity; "Household Member Participation" measures a member's joining and use, not the organiser role's transfer |
| organiser_handover_cancelled | reason (organiser_cancelled / recipient_no_longer_eligible) | A pending request is cleared without a decision | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-003-AC-01:** Given Maya has one Active Other Adult Member (Sam), when she opens this screen, then Sam appears as the sole eligible recipient and "Send Request" is disabled until she selects him.

**FEAT-09.SPEC-003-AC-02:** Given Maya selects Sam and taps "Send Request", when the send succeeds, then the screen switches to "Waiting for Sam to accept." and Sam receives the notification (FEAT-09.SPEC-014).

**FEAT-09.SPEC-003-AC-03:** Given Maya has a pending request to Sam, when she taps "Cancel Request" and confirms, then the request clears, the screen returns to the recipient list, and the toast "Request cancelled" appears.

**FEAT-09.SPEC-003-AC-04:** Given Sam declines Maya's hand-over request, when Maya next opens this screen, then she sees "Sam declined. You're still the organiser." with a "Choose Someone Else" button.

**FEAT-09.SPEC-003-AC-05:** Given Sam is removed or leaves while Maya's request to him is pending, when the change is recorded, then the request is silently cancelled and Maya sees "Sam is no longer an active member; the request was cancelled."

**FEAT-09.SPEC-003-AC-06:** Given Maya's household has no Active Other Adult Members, when she opens this screen, then she sees "You need at least one other active adult member to hand over the role. Invite someone first." with a link to FEAT-09.SPEC-001.

**FEAT-09.SPEC-003-AC-07:** Given Sam attempts to navigate directly to this screen, when the request is made, then he sees "Only the organiser can hand over the role" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-003-AC-08:** Given Maya cancels her pending request at the same moment Sam accepts it and Sam's acceptance is recorded first, when Maya's cancel is processed, then it is rejected and the screen reflects that the organiser role has already transferred.

**FEAT-09.SPEC-003-AC-09:** Given Maya loses connectivity while a recipient is selected, when she attempts to tap "Send Request", then the banner "You're offline -- sending a hand-over request needs a connection." appears and the button is disabled.

**FEAT-09.SPEC-003-AC-10:** Given Maya taps "Send Request" twice rapidly, when the first tap is already processing, then the second tap has no additional effect and only one request is sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 (no eligible recipients, no pending, sending, pending, declined, send error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Organiser Hand-Over Acceptance

## Overview

**Name:** Organiser Hand-Over Acceptance
**ID:** FEAT-09.SPEC-004
**Type:** Screen
**Purpose:** The chosen adult member accepts or declines becoming the new organiser.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Showing the outstanding hand-over request addressed to the signed-in member
- Accepting the request, which triggers FEAT-09.SPEC-009 (Organiser Hand-Over Processing)
- Declining the request, which returns the organiser to FEAT-09.SPEC-003 with the outcome

**Non-Goals:**
- Selecting who receives a hand-over request -- owned by FEAT-09.SPEC-003 (Organiser Hand-Over Initiation)
- Performing the actual role transfer and member-type swap -- owned by FEAT-09.SPEC-009, which this screen triggers on accept but does not perform itself
- Cancelling a request from the organiser's side -- owned by FEAT-09.SPEC-003; this screen only offers the recipient's own accept/decline choice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification) | Recipient opens the notification | The pending hand-over request addressed to them |
| FEAT-01.SPEC-010 (Household Settings Hub) | Recipient has a pending request and taps the pending hand-over entry ("{organiser display name} wants to make you the organiser") | Same pending request |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Sam (Other Adult Member, addressed recipient) | Full screen | Accept or decline | -- |
| Sam (Other Adult Member, not the addressed recipient -- e.g., a future household with more than one other adult) | No | No | N/A -- there is at most one outstanding hand-over request per household (XBR-15); a member with no pending request addressed to them sees no entry point |
| Maya (Organiser) | No | No | The organiser tracks her own request through FEAT-09.SPEC-003, not this screen; opening this screen while still organiser (e.g., stale link) shows "You're the organiser -- there's nothing to accept here." |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles are never eligible hand-over recipients |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Household Invitations is outside the older-kid login's entitlements |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001; no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the pending request is unaffected and re-shown after re-authentication |

## Layout and Content

**Header:** Screen title "Become the Organiser?" with no back navigation (this is a decision screen reached from a notification or a settings entry, not a browsing flow).

**Body:**
- Plain statement of what accepting means, matching the same confirmation-required modal pattern as Leave Household (FEAT-09.SPEC-005): "{organiser display name} wants to make you the organiser. You'll take over the weekly budget, schedule, and household settings. {organiser display name} keeps full access as an Other Adult Member."
- "Accept" and "Decline" buttons, no default-confirmed state

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Message and buttons stack full width, "Accept" above "Decline."
- **Medium size class and above:** Content area caps at a consistent platform-wide narrow width and is horizontally centered; buttons sit side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Accept" button | Tap | 1. Re-check the request is still outstanding via FEAT-09.SPEC-010. 2. If still outstanding, trigger FEAT-09.SPEC-009 (Organiser Hand-Over Processing). | Button shows loading state | Success: screen transitions to the Success state with "You're now the organiser." confirmation. Failure (request no longer outstanding): switches to the withdrawn-request message |
| "Decline" button | Tap | Opens a confirmation dialog: "Decline becoming the organiser? {organiser display name} will keep the role." | Dialog appears | Confirm clears the request and navigates back to FEAT-01.SPEC-010 with the toast "You declined."; Cancel closes the dialog with no change |
| "Accept" / "Decline" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Go to Household Settings" button (Success state) | Tap | Navigates to FEAT-01.SPEC-010 with organiser-level access | Screen unmounts | Household Settings Hub loads with the new organiser's elevated access |
| "Review your account details" link (Success state) | Tap | Navigates to FEAT-18.SPEC-004 (My Account), now showing the organiser-only Household Data & Deletion section | Screen unmounts | My Account loads the new organiser's own profile, with the organiser-only section now visible |

### Accessibility Notes

- **Focus order:** Message, then "Accept", then "Decline". On the Success state, focus moves to the confirmation message, then "Go to Household Settings", then "Review your account details".
- **Confirmation announcements:** "You're now the organiser." and "You declined." are announced when they appear.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Pending decision | Message and Accept/Decline buttons as described | Screen loads with a request still outstanding, addressed to this member | Member taps Accept or Decline |
| Accepting | "Accept" button shows loading state, both buttons disabled | Member taps Accept | Processing completes or the request is found no longer outstanding |
| Success | "You're now the organiser." confirmation, with "Go to Household Settings" and "Review your account details" buttons | FEAT-09.SPEC-009 (Organiser Hand-Over Processing) completes the role transfer | Member taps "Go to Household Settings" (routes to FEAT-01.SPEC-010) or "Review your account details" (routes to FEAT-18.SPEC-004) |
| Declining | Confirmation dialog shown | Member taps Decline | Member confirms or cancels the dialog |
| Withdrawn | Message: "This request is no longer available -- {organiser display name} may have cancelled it or the role has already changed hands." with a single "Back to Household" button | The request is found no longer outstanding when Accept is attempted, or the screen is opened after the request already resolved | Member taps "Back to Household" |
| Error | Inline error banner; Pending decision body remains | Accept processing fails for a reason other than the request being withdrawn (e.g., a transient failure) | Member retries and processing succeeds |
| Offline/Degraded | Banner: "You're offline -- accepting or declining needs a connection." Buttons disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears and buttons re-enable |

## Validation Rules

Validation governed by FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) for whether the hand-over request is still outstanding at the moment of accept or decline.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Go to Household Settings" (Success state) | FEAT-01.SPEC-010 (Household Settings Hub), now with organiser access | FEAT-01 |
| "Review your account details" (Success state) | FEAT-18.SPEC-004 (My Account), now showing the organiser-only section | FEAT-18 |
| Decline confirmed | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| "Back to Household" (withdrawn state) | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |

## Data Model

**Creates:** None directly -- Household.organiser and both members' member_type are updated by FEAT-09.SPEC-009 on accept.
**Reads:** Household -- pending_organiser_handover (the workflow marker defined in FEAT-09.SPEC-003) to confirm the request addressed to this member and to display the current organiser's display name.
**Updates:** Household -- pending_organiser_handover is cleared on decline (processing owns clearing it on accept, per FEAT-09.SPEC-009).
**Deletes:** None.

## Business Rules

- Only the addressed recipient can act on a given request; there is never more than one outstanding hand-over request per household (XBR-15).
- Declining leaves the organiser role unchanged and does not notify the organiser through a separate Notification spec -- the outcome surfaces inline the next time she opens FEAT-09.SPEC-003 (feature-overview.md, Side-Effect Inventory).
- Accepting is the sole trigger for FEAT-09.SPEC-009's atomic role transfer; no partial state exists between accept and the transfer completing.

## Edge Cases

- **Organiser cancels the request at the same moment the recipient taps Accept** -- Reject-with-refresh: whichever change is recorded first wins. If the cancel is recorded first, the accept fails and the screen switches to the Withdrawn state; if the accept is recorded first, it proceeds normally regardless of the near-simultaneous cancel attempt.
- **Recipient's own account changes state while the request is pending (e.g., they leave the household through another session)** -- The request becomes void along with their membership; opening this screen shows the Withdrawn state, since a departed member cannot become organiser.
- **Recipient taps Accept and Decline in quick succession (double interaction)** -- The first action to register is processed; the second is ignored once the button/dialog for the first action is in progress.
- **Recipient opens the screen after the request already resolved (stale notification link)** -- Withdrawn state, with the specific outcome unknown to this screen (it does not distinguish "already accepted by someone else" from "cancelled," since at most one recipient and one outcome exist at a time -- the message stays generic).
- **Recipient loses connectivity mid-accept** -- The offline banner appears and Accept/Decline are disabled; no partial role transfer occurs, since FEAT-09.SPEC-009 only runs after a successfully recorded accept.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-014 (Organiser Hand-Over Request Notification) | Navigation (inbound) | The notification's CTA opens this screen |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | References (inbound), Navigation (outbound) | Declining reports the outcome there |
| FEAT-09.SPEC-009 (Organiser Hand-Over Processing) | Triggers (outbound) | Accept fires the atomic role transfer |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Whether the request is still outstanding |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | Destination after accept, decline, or withdrawal |
| FEAT-18.SPEC-004 (My Account) | Navigation (outbound) | The Success state offers a link back to the new organiser's own account details |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| organiser_handover_accepted | -- | Recipient's accept is recorded | N/A -- no Stage 2 metric measures hand-over outcomes; "Household Member Participation" measures a member's joining and use, not the organiser role's transfer |
| organiser_handover_declined | -- | Recipient's decline is recorded | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-004-AC-01:** Given Sam has a pending hand-over request from Maya, when he opens this screen, then he sees "Maya wants to make you the organiser..." with "Accept" and "Decline" buttons.

**FEAT-09.SPEC-004-AC-02:** Given Sam taps "Accept", when the acceptance is processed, then FEAT-09.SPEC-009 transfers the organiser role and Sam sees the Success state with "You're now the organiser." along with "Go to Household Settings" and "Review your account details" buttons.

**FEAT-09.SPEC-004-AC-03:** Given Sam taps "Decline" and confirms, when the decline is recorded, then he is returned to FEAT-01.SPEC-010 with the toast "You declined." and Maya remains the organiser.

**FEAT-09.SPEC-004-AC-04:** Given Maya cancels the request at the same moment Sam taps Accept, and the cancel is recorded first, when Sam's accept is processed, then it fails and Sam sees the Withdrawn state.

**FEAT-09.SPEC-004-AC-05:** Given Sam opens this screen after the request has already resolved, when the screen loads, then he sees "This request is no longer available -- Maya may have cancelled it or the role has already changed hands." with a "Back to Household" button.

**FEAT-09.SPEC-004-AC-06:** Given Maya (still organiser) opens this screen directly via a stale or mistaken link, when the screen loads, then she sees "You're the organiser -- there's nothing to accept here."

**FEAT-09.SPEC-004-AC-07:** Given Sam loses connectivity while viewing the pending request, when he attempts to tap "Accept" or "Decline", then the banner "You're offline -- accepting or declining needs a connection." appears and both buttons are disabled.

**FEAT-09.SPEC-004-AC-08:** Given Sam leaves the household from another session while this request is pending to him, when he (or anyone) opens this screen for that request, then it shows the Withdrawn state.

**FEAT-09.SPEC-004-AC-09:** Given Sam taps "Accept" and then rapidly taps "Decline" before the accept completes, when the accept is already processing, then the decline tap has no effect and only the accept outcome applies.

**FEAT-09.SPEC-004-AC-10:** Given Sam is on the Success state after accepting the organiser role, when he taps "Review your account details", then he is navigated to FEAT-18.SPEC-004 (My Account), which loads his own profile now showing the organiser-only Household Data & Deletion section.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (pending, accepting, declining, success, withdrawn, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Leave Household

## Overview

**Name:** Leave Household
**ID:** FEAT-09.SPEC-005
**Type:** Screen
**Purpose:** An other adult member confirms and completes leaving the household on their own.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The confirmation step for an Other Adult Member choosing to leave the household on their own
- Triggering FEAT-09.SPEC-008 (Member Departure Processing) on confirmation
- The organiser-blocked experience when the current organiser attempts to reach this action (XBR-15)

**Non-Goals:**
- Actually anonymising ratings and soft-removing the Member Profile -- owned by FEAT-09.SPEC-008, which this screen triggers but does not perform itself
- Removing a member by the organiser's own action -- owned by FEAT-18 (Account & Data Management); this screen is exclusively the self-service leave path
- Deleting the leaving member's own account entirely (closing the account, not just membership) -- owned by FEAT-18; leaving the household and deleting one's account are distinct actions per the Access Matrix's separate Household Invitations and Account & Data columns

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Other Adult Member opens "Leave household" from settings | None |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Sam (Other Adult Member) | Full screen | Confirm leaving | -- |
| Maya (Organiser) | No | No | "Leave household" entry point is not shown to the organiser in settings; a direct navigation attempt shows "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18, per XBR-15 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- kid profiles have no household-settings access of any kind |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- Account & Data is outside the older-kid login's entitlements (Access Matrix: None) |
| Riley (Operator, support) | No | No | N/A -- Riley's access is limited to the separate read-only support view (FEAT-22, XBR-14) |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001; no household data is rendered |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; no destructive action has occurred yet, so nothing needs to be preserved |

## Layout and Content

**Header:** Screen title "Leave Household" with a back arrow (returns to FEAT-01.SPEC-010).

**Body:** A confirmation panel using the same modal pattern as Organiser Hand-Over Acceptance (FEAT-09.SPEC-004): a plain statement of consequences, an explicit affirmative action, no default-confirmed state.
- "If you leave, you'll lose access to this household's plan, grocery list, and settings. Your past ratings will keep quietly influencing future meal choices, but they'll no longer be tied to your name."
- "Leave Household" button (destructive styling) and "Cancel" button

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Message and buttons stack full width, "Cancel" above "Leave Household" to avoid an accidental destructive tap being the first reachable action.
- **Medium size class and above:** Content area caps at a consistent platform-wide narrow width and is horizontally centered; buttons sit side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 | Screen closes | Standard navigation transition |
| "Cancel" button | Tap | Navigate to FEAT-01.SPEC-010, no change made | Screen closes | Standard navigation transition |
| "Leave Household" button | Tap | Opens a final confirmation dialog: "Are you sure? This can't be undone." | Dialog appears | Standard dialog transition |
| Final confirmation dialog, "Yes, Leave" | Tap | Triggers FEAT-09.SPEC-008 (Member Departure Processing) | Button shows loading state | Success: signed out of the household and routed to FEAT-01.SPEC-001 with the message "You've left the household." Failure: inline error, member remains a household member |
| Final confirmation dialog, "Stay" | Tap | Closes the dialog, no change made | Dialog closes | Returns to the confirmation panel |
| "Yes, Leave" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> message -> "Cancel" -> "Leave Household" -> (dialog) "Stay" -> "Yes, Leave".
- **Destructive-action announcement:** The final confirmation dialog is announced as a destructive, irreversible action when it opens.
- **Completion announcement:** "You've left the household." is announced on success.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Confirming | Confirmation panel as described in Layout and Content | Screen first opens | Member taps Cancel, Leave Household, or the back arrow |
| Final confirmation | Dialog: "Are you sure? This can't be undone." | Member taps "Leave Household" | Member taps "Yes, Leave" or "Stay" |
| Leaving | "Yes, Leave" button shows loading state, dialog non-dismissible | Member taps "Yes, Leave" | Departure processing completes or fails |
| Error | Inline error banner; confirmation panel remains, member is still a household member | Departure processing fails (e.g., a transient failure) | Member retries and the leave succeeds |
| Offline/Degraded | Banner: "You're offline -- leaving needs a connection." "Leave Household" is disabled | Connectivity lost while this screen is open | Connectivity restored -- banner clears and the button re-enables |

## Validation Rules

Validation governed by FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules): only an Other Adult Member may reach and complete this action; the current organiser is blocked per XBR-15.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow / Cancel tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 |
| Successful leave | FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | FEAT-01 |
| Organiser-blocked message links | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), FEAT-18 (Account & Data Management) | FEAT-18 |

## Data Model

**Creates:** None.
**Reads:** The signed-in member's own Member Profile.member_type, to confirm eligibility (Other Adult Member only) before the confirmation panel renders.
**Updates:** None directly -- Member Profile.status and Rating anonymisation are performed by FEAT-09.SPEC-008.
**Deletes:** None directly.

## Business Rules

- Only an Other Adult Member (never the current organiser) can complete this action, per XBR-15; the organiser must hand over the role (FEAT-09.SPEC-003) or delete the household (FEAT-18) first.
- Leaving is final and offers no restore path: a re-invited former member onboards again as a new Member Profile rather than being restored to their prior profile or history (XBR-18, feature-overview.md Entity-Lifecycle Coverage Matrix).
- The leaving member's Dietary Rules disposition is not decided by this screen or by FEAT-09.SPEC-008 -- feature-overview.md flags this as an open product ambiguity for resolution outside this feature; this screen's confirmation message therefore covers only what this feature itself performs (ratings, membership), never implying a Dietary Rules outcome it does not control.

## Edge Cases

- **Member's role changes to organiser (accepts a hand-over) while this screen is open in another tab** -- The next action on this screen (Leave Household or the final confirmation) is rejected with "You're the organiser now -- hand over the role or delete the household to leave." since eligibility is re-checked at action time, not screen-load time.
- **Member taps "Yes, Leave" twice rapidly** -- Second tap is ignored while the first departure is in progress (dialog and button disabled).
- **Departure processing fails after the member has already been shown the final confirmation** -- The member remains a full household member; the inline error is shown and no partial departure state exists (Rating anonymisation and Member Profile removal happen atomically together in FEAT-09.SPEC-008).
- **Member loses connectivity between the final confirmation and processing completing** -- The offline banner replaces the loading state; the leave has not taken effect and the member remains a household member until connectivity returns and the action completes or is retried.
- **Only member (aside from the organiser) leaves, dropping the household to a single active adult** -- Leaving proceeds normally; the household continuing with only the organiser is a valid state (feature-overview.md, Household holds 1-12 members with no stated minimum beyond the one required organiser).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-008 (Member Departure Processing) | Triggers (outbound) | "Yes, Leave" fires ratings anonymisation and membership removal |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Restricts this screen to Other Adult Members, blocks the organiser per XBR-15 |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Entry point from settings; Cancel and back arrow return there |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (outbound) | Destination after a successful leave |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), FEAT-18 (Account & Data Management) | Navigation (outbound) | Organiser-blocked message's alternative paths |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_left_household_confirmed | -- | Member completes the final confirmation and departure processing succeeds | N/A -- "Household Member Participation" measures a household gaining and keeping an engaged other adult member; a departure is the inverse signal and is not itself a positive contribution to that target, so no Stage 2 metric is fed directly by this event; it is retained to explain drops in household participation when reading that metric |
| leave_blocked_organiser | -- | The organiser attempts to reach this action and is blocked | N/A -- diagnostic signal only |

## Acceptance Criteria

**FEAT-09.SPEC-005-AC-01:** Given Sam (Other Adult Member) opens this screen, when it loads, then he sees the confirmation panel explaining that ratings become anonymous influence and access is lost.

**FEAT-09.SPEC-005-AC-02:** Given Sam taps "Leave Household" and then "Yes, Leave" on the final confirmation, when departure processing completes, then he is signed out and routed to FEAT-01.SPEC-001 with "You've left the household."

**FEAT-09.SPEC-005-AC-03:** Given Sam taps "Leave Household", when the final confirmation dialog appears, then it reads "Are you sure? This can't be undone." with "Yes, Leave" and "Stay" options and no default-confirmed state.

**FEAT-09.SPEC-005-AC-04:** Given Maya (Organiser) attempts to navigate directly to this screen, when the request is made, then she sees "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18.

**FEAT-09.SPEC-005-AC-05:** Given Sam taps "Cancel" on the confirmation panel, when the tap registers, then he returns to FEAT-01.SPEC-010 with no change to his membership.

**FEAT-09.SPEC-005-AC-06:** Given Sam accepts a hand-over and becomes organiser in another tab while this screen is open, when he then taps "Leave Household", then the action is rejected with "You're the organiser now -- hand over the role or delete the household to leave."

**FEAT-09.SPEC-005-AC-07:** Given Sam loses connectivity on the confirmation panel, when he attempts to tap "Leave Household", then the banner "You're offline -- leaving needs a connection." appears and the button is disabled.

**FEAT-09.SPEC-005-AC-08:** Given Sam's departure processing fails after he confirms, when the failure occurs, then he remains a full household member and an inline error is shown with no partial departure state.

**FEAT-09.SPEC-005-AC-09:** Given Sam taps "Yes, Leave" twice rapidly on the final confirmation, when the first tap is already processing, then the second tap has no additional effect and departure processing runs exactly once.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (confirming, final confirmation, leaving, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Invitation Expiry

## Overview

**Name:** Invitation Expiry
**ID:** FEAT-09.SPEC-006
**Type:** Automation
**Purpose:** A pending invitation automatically expires 14 days after it was sent, so a stale invitation link stops working and the organiser sees its true state.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Transitioning a Sent invitation to Expired once 14 days have passed since it was sent
- Making the expired invitation's link resolve to the "no longer valid" experience on FEAT-09.SPEC-002
- Making the Expired status and a Resend action available on FEAT-09.SPEC-001

**Non-Goals:**
- Resending an expired invitation -- a distinct, user-initiated action on FEAT-09.SPEC-001 (Household Invitations Manager), owned by that screen, not this automation
- Notifying the organiser when an invitation expires -- feature-overview.md's Communications field and Side-Effect Inventory name a confirmation on acceptance and on a member leaving, but no notification for expiry; the organiser sees the Expired status passively on FEAT-09.SPEC-001, per that screen's Empty/List states
- Deleting the expired invitation record -- invitations are never hard-deleted, retained with their terminal status as household history (assumptions-constraints.md ASMP-24)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| 14 days elapsed since an invitation was sent | system (schedule-based) | Fires once per Invitation, 14 days after its sent date, only if its status is still Sent at that moment | Invitation contact_detail, status, sent_by, sent date |

## Processing Logic

1. On each scheduled run, identify every Invitation whose status is Sent and whose sent date is 14 or more days in the past.
2. For each identified invitation, re-confirm its status is still Sent at the moment of processing (it may have been accepted or revoked since the schedule last ran).
3. If still Sent, set the invitation's status to Expired.
4. If no longer Sent (already Accepted or Revoked), skip it -- no action, no error.
5. No notification is sent for any transition performed by this automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Invitation expired | Invitation was Sent and 14+ days have elapsed since it was sent | Invitation.status set to Expired | None immediately; the organiser sees the Expired badge and a "Resend" action the next time she opens FEAT-09.SPEC-001; an invitee opening the link afterward sees "This invitation is no longer valid. It expired after 14 days." on FEAT-09.SPEC-002 | FEAT-09.SPEC-001, FEAT-09.SPEC-002 |
| No action (already resolved) | Invitation reached 14 days but was already Accepted or Revoked before this run | None | None -- the invitation already reflects its resolved status | -- |
| Automation failure | Processing error during a scheduled run | None -- no partial expiry is ever applied | None immediately; the affected invitations remain Sent and are picked up on the next scheduled run | FEAT-09.SPEC-001 (continues to show the invitation as Sent, including one that is functionally overdue, until the next successful run) |

## Data Model

**Reads:** Invitation -- status and sent date, across all households, to find every Sent invitation past 14 days.
**Creates:** None.
**Updates:** Invitation -- status (Sent to Expired).
**Deletes:** None.

## Business Rules

- The 14-day expiry window is fixed at 14 days from the invitation's sent date (feature-overview.md, Validation & Limits) -- this is a feature-level product decision stated directly in the product definition, not a platform-wide policy value.
- Expiry never reverses an Accepted or Revoked invitation -- it applies only to invitations still in the Sent status at the moment of processing (FEAT-09.SPEC-010 governs this precedence).
- An Expired invitation can be resent as a fresh Sent invitation (a new Invitation record via FEAT-09.SPEC-001), never revived in place.

## Edge Cases

- **Invitation is accepted at almost exactly the same moment this automation would expire it** -- Reject-with-refresh, per the dependency map's Contention note for Invitation: whichever change is recorded first wins. If the acceptance is recorded first, this automation's re-confirmation step (Processing Logic, step 2) finds the invitation no longer Sent and skips it.
- **Invitation is revoked at almost exactly the same moment this automation would expire it** -- Same resolution: whichever change is recorded first wins; if the revoke is recorded first, this automation skips the invitation.
- **Scheduled run is delayed or missed** -- Invitations remain Sent past their 14-day window until the next successful run, which processes them as overdue at that point; no invitation expires "early," and no expiry is silently lost.
- **Concurrent trigger firing (two scheduled runs overlap)** -- Each invitation's expiry transition is idempotent: setting an already-Expired invitation to Expired again has no observable effect, so an overlapping run produces no double transition or duplicate side effect (there is none to duplicate, since no notification is sent).
- **Trigger fires while a previous run is still in flight** -- The next scheduled run is skipped if the prior run has not completed, preventing two runs from re-confirming and transitioning the same invitation simultaneously; a skipped run's candidates are picked up whole on the following run.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-001 (Household Invitations Manager) | Affects (outbound) | Expired status and Resend action shown there |
| FEAT-09.SPEC-002 (Invitation Acceptance) | Affects (outbound) | An expired invitation's link resolves to the "no longer valid" experience there |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Defines the 14-day window and the accept/revoke/expiry race resolution this automation follows |

## Analytics and Success Signals

- **invitation_expired** (days_outstanding: 14) -- N/A -- no Stage 2 metric measures invitation expiry; "Household Member Participation" measures a member's joining and use, not the invitation funnel's attrition, so this signal is retained only as a funnel-diagnostic event.

## Acceptance Criteria

**FEAT-09.SPEC-006-AC-01:** Given an invitation was sent 14 days ago and is still Sent, when this automation runs, then the invitation's status becomes Expired.

**FEAT-09.SPEC-006-AC-02:** Given an invitation was sent 13 days ago and is still Sent, when this automation runs, then the invitation remains Sent.

**FEAT-09.SPEC-006-AC-03:** Given an invitation reached 14 days but was accepted moments before this automation runs, when the automation processes it, then it finds the invitation already Accepted and takes no action.

**FEAT-09.SPEC-006-AC-04:** Given an invitation reached 14 days but was revoked moments before this automation runs, when the automation processes it, then it finds the invitation already Revoked and takes no action.

**FEAT-09.SPEC-006-AC-05:** Given Maya opens FEAT-09.SPEC-001 after an invitation has expired, when the list loads, then that invitation shows the Expired badge and a Resend action.

**FEAT-09.SPEC-006-AC-06:** Given an invitee opens the link to an invitation that has since expired, when FEAT-09.SPEC-002 loads, then they see "This invitation is no longer valid. It expired after 14 days."

**FEAT-09.SPEC-006-AC-07:** Given two scheduled runs overlap for the same invitation, when both attempt the expiry transition, then the invitation ends in Expired status exactly once with no duplicate side effect.

**FEAT-09.SPEC-006-AC-08:** Given a scheduled run is missed and an invitation is now 20 days past its sent date, when the next successful run occurs, then that invitation is still correctly transitioned to Expired.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (expired, no action, automation failure) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Invitation Acceptance Processing

## Overview

**Name:** Invitation Acceptance Processing
**ID:** FEAT-09.SPEC-007
**Type:** Automation
**Purpose:** On acceptance, creates the new Member Profile with Other Adult Member access and routes the invitee into first-use onboarding.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Re-validating the invitation is still Sent at the moment of processing (the accept/revoke/expiry race)
- Creating the new Member Profile record with Other Adult Member access, using the sign-in details captured on FEAT-09.SPEC-002
- Transitioning the Invitation's status to Accepted
- Triggering FEAT-09.SPEC-012 (Invitation Accepted Confirmation) to the organiser
- Routing the newly created member into FEAT-15 (Member Onboarding)

**Non-Goals:**
- Collecting the invitee's display name, sign-in email, and password -- captured by FEAT-09.SPEC-002 (Invitation Acceptance) and passed into this automation, not collected here
- The onboarding experience itself (first-use landing on the plan and list) -- owned by FEAT-15 (Member Onboarding); this automation only performs the hand-off
- Restoring a re-invited former member's prior profile, dietary rules, or history -- excluded per XBR-18; every acceptance creates a genuinely new Member Profile, even for someone who was previously a member of this same household

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invitee taps "Accept & Join" | FEAT-09.SPEC-002 (Invitation Acceptance) | Fires on successful form validation, before the invitation's current validity is re-confirmed | Invitation reference, invitee's display name, sign-in email, password |

## Processing Logic

1. Receive the invitation reference and the invitee's entered display name, sign-in email, and password from FEAT-09.SPEC-002.
2. Re-check the invitation's current status per FEAT-09.SPEC-010's race resolution: proceed only if it is still Sent.
3. If no longer Sent (already Accepted, Revoked, or Expired since the invitee loaded the screen), stop processing and return the current status and reason to FEAT-09.SPEC-002 -- no Member Profile is created.
4. If still Sent, create a new Member Profile: display_name from the invitee's input, member_type set to Other Adult Member, sign_in set to the invitee's entered email and protected sign-in, status set to Active, notification_preferences set to the household's stated defaults (plan-ready on, nightly nudge on).
5. Set the Invitation's status to Accepted.
6. Trigger FEAT-09.SPEC-012 (Invitation Accepted Confirmation) to the household's current organiser.
7. Route the newly created member into FEAT-15 (Member Onboarding), landing in context on the household's current Weekly Plan and Grocery List.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Acceptance succeeds | Invitation is still Sent at processing time | New Member Profile created (Active, Other Adult Member); Invitation.status set to Accepted | Invitee is routed into FEAT-15 first-use onboarding; organiser receives FEAT-09.SPEC-012 | FEAT-09.SPEC-002, FEAT-09.SPEC-001, FEAT-15 (FEAT-15.SPEC-002), FEAT-09.SPEC-012 |
| Acceptance rejected -- race lost | Invitation is no longer Sent (Accepted, Revoked, or Expired) when processing runs | No data changes | FEAT-09.SPEC-002 switches to the invalid-invitation body with the specific current reason; no Member Profile is created | FEAT-09.SPEC-002 |
| Automation failure | Processing error after the race check passes but before the Member Profile is durably created | No partial Member Profile persists; the Invitation remains Sent | FEAT-09.SPEC-002 shows an inline error and offers Retry; the invitee's entered sign-in details are preserved for the retry | FEAT-09.SPEC-002 |

## Data Model

**Reads:** Invitation -- status, contact_detail, sent_by; Household -- the organiser reference, to address FEAT-09.SPEC-012.
**Creates:** Member Profile -- display_name, member_type (Other Adult Member), sign_in, status (Active), notification_preferences (household defaults).
**Updates:** Invitation -- status (Sent to Accepted).
**Deletes:** None.

## Business Rules

- An accepted invitation creates a Member Profile with Other Adult Member access and triggers first-use onboarding exactly once per accepted invitation (XBR-18); this automation never runs its creation step more than once for the same invitation.
- A re-invited former member (someone who previously left or was removed) onboards again as a genuinely new Member Profile rather than being restored to old data (XBR-18) -- this automation performs no lookup against any prior profile for the same person.
- The race between acceptance and a concurrent revocation or expiry resolves reject-with-refresh: whichever change lands first wins (FEAT-09.SPEC-010).
- Notification preferences on the new Member Profile default to the household's stated defaults; the new member can change them afterward through FEAT-07/FEAT-13's own settings, not through this automation.

## Edge Cases

- **Invitation is revoked between the invitee's tap and this automation's re-check** -- Reject-with-refresh: the revoke wins if recorded first, this automation's race check (Processing Logic, step 2-3) finds the invitation Revoked and stops, and no Member Profile is created.
- **Invitation expires (FEAT-09.SPEC-006) between the invitee's tap and this automation's re-check** -- Same resolution: whichever change lands first wins; if expiry is recorded first, this automation stops with no Member Profile created.
- **Two people somehow attempt to accept the same invitation link at effectively the same time** -- Only the first acceptance to be recorded succeeds and creates the Member Profile and transitions the Invitation to Accepted; the second is processed against the now-Accepted invitation and is rejected via the race-lost outcome, exactly as an ordinary "already accepted" case.
- **Concurrent trigger firing (this automation and FEAT-09.SPEC-006's expiry fire on the same invitation at effectively the same time)** -- Whichever transition is recorded first wins per FEAT-09.SPEC-010; the loser's spec (this automation, or FEAT-09.SPEC-006) takes its respective no-action/rejected path with no partial or conflicting Invitation state.
- **Trigger fires while a previous run is still in flight for the same invitation** -- A second acceptance attempt for the same invitation cannot start meaningfully while the first is in flight, since FEAT-09.SPEC-002's Accept & Join button is disabled during processing on that screen; a second attempt from a different session for the same invitation processes against whatever status the first run leaves behind (Accepted, if the first run succeeded), taking the race-lost path.
- **Member Profile creation succeeds but the notification trigger (FEAT-09.SPEC-012) fails** -- The acceptance itself is not rolled back; the organiser simply does not receive the confirmation and instead sees the new member on FEAT-09.SPEC-001 (Household Invitations Manager) directly, since a missed confirmation must never undo a completed membership change.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invitation Acceptance) | Triggered by (inbound) | Accept & Join fires this automation |
| FEAT-09.SPEC-002 (Invitation Acceptance) | Affects (outbound) | Returns success or the specific invalidity reason |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | The accept/revoke/expiry race resolution |
| FEAT-09.SPEC-001 (Household Invitations Manager) | Affects (outbound) | The accepted invitation and new member become visible there |
| FEAT-09.SPEC-012 (Invitation Accepted Confirmation) | Triggers (outbound) | Notifies the organiser of the successful acceptance |
| FEAT-15 (Member Onboarding) | Affects (outbound) | Routes the new member into first-use onboarding |
| FEAT-09.SPEC-006 (Invitation Expiry) | References (inbound) | Shares the same race-resolution outcome space on the Invitation entity |

## Analytics and Success Signals

- **invitation_accepted** (time from sent to accepted, in days) -- supports success-metrics.md: "Household Member Participation" (the acceptance is the moment a household gains the other adult member the metric requires)
- **invitation_accept_rejected** (reason: revoked / expired / already_accepted) -- N/A -- no Stage 2 metric measures rejected acceptance attempts; retained as a funnel-diagnostic signal
- **member_profile_created** (member_type: other_adult) -- supports success-metrics.md: "Household Member Participation" (creation of the joined member's profile is the durable record the metric's "joined" condition is evaluated against)

## Acceptance Criteria

**FEAT-09.SPEC-007-AC-01:** Given Sam submits valid sign-in details on FEAT-09.SPEC-002 for a still-Sent invitation, when this automation processes the acceptance, then a new Member Profile is created for Sam with Other Adult Member access and Active status.

**FEAT-09.SPEC-007-AC-02:** Given the automation successfully creates Sam's Member Profile, when creation completes, then the Invitation's status is set to Accepted and Sam is routed into FEAT-15 (Member Onboarding) landing on the current Weekly Plan.

**FEAT-09.SPEC-007-AC-03:** Given the automation successfully processes Sam's acceptance, when processing completes, then Maya (the organiser) receives FEAT-09.SPEC-012 (Invitation Accepted Confirmation).

**FEAT-09.SPEC-007-AC-04:** Given the invitation was revoked by Maya moments before this automation's re-check, when the automation processes Sam's submission, then it stops with no Member Profile created and FEAT-09.SPEC-002 shows the revoked-invitation message.

**FEAT-09.SPEC-007-AC-05:** Given the invitation expired moments before this automation's re-check, when the automation processes the submission, then it stops with no Member Profile created and FEAT-09.SPEC-002 shows the expired-invitation message.

**FEAT-09.SPEC-007-AC-06:** Given Sam previously left this same household and is being re-invited, when he accepts the new invitation, then a genuinely new Member Profile is created for him with no data restored from his prior profile.

**FEAT-09.SPEC-007-AC-07:** Given two acceptance attempts for the same invitation are processed at effectively the same time, when the first is recorded, then the second is rejected as already-accepted with no second Member Profile created.

**FEAT-09.SPEC-007-AC-08:** Given the Member Profile creation succeeds but the confirmation trigger to Maya fails, when this is detected, then Sam's membership remains in effect and Maya instead sees the new member directly on FEAT-09.SPEC-001.

**FEAT-09.SPEC-007-AC-09:** Given a processing error occurs after the race check passes but before the Member Profile is durably created, when the failure occurs, then no partial Member Profile exists, the Invitation remains Sent, and FEAT-09.SPEC-002 shows an inline error with Sam's entered details preserved for retry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, rejected -- race lost, automation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Member Departure Processing

## Overview

**Name:** Member Departure Processing
**ID:** FEAT-09.SPEC-008
**Type:** Automation
**Purpose:** On a member leaving, anonymises their ratings, removes their Member Profile from active membership, and stops future plans from accounting for them.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Re-validating the leaving member is eligible to leave (an Other Adult Member, never the current organiser) at the moment of processing
- Anonymising every Rating recorded by the leaving member, so it keeps influencing future plans without attribution
- Setting the leaving member's Member Profile status to Left and removing it from the active member list
- Triggering FEAT-09.SPEC-013 (Member Left Household Notification) to the organiser
- Ending the leaving member's session

**Non-Goals:**
- Deciding whether to leave -- the confirmation is owned by FEAT-09.SPEC-005 (Leave Household), which triggers this automation
- Deleting or anonymising the leaving member's Dietary Rules -- feature-overview.md's Shared Context flags this as an open product ambiguity not resolved by this feature; this automation touches only Ratings and Member Profile status, never Dietary Rule records
- Removing a member by the organiser's own action, or cascading a member's data on removal -- owned by FEAT-18 (Account & Data Management), a distinct path from this self-service leave (XBR-16 draws this exact line between removal and self-leaving)
- Stopping future plan and grocery-list accounting for the departed member -- the plan-generation and list-recalculation logic itself belongs to FEAT-03 and FEAT-06, which read the updated Member Profile status this automation sets; this automation only performs the status change, not the downstream recalculation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member confirms leaving | FEAT-09.SPEC-005 (Leave Household) | Fires on the final confirmation ("Yes, Leave"), before eligibility is re-confirmed | Leaving member's Member Profile reference |

## Processing Logic

1. Receive the leaving member's Member Profile reference from FEAT-09.SPEC-005.
2. Re-check eligibility per FEAT-09.SPEC-011: proceed only if the member's current member_type is Other Adult Member (never the current organiser).
3. If not eligible (the member has become organiser since the confirmation screen loaded), stop processing and return the block reason to FEAT-09.SPEC-005 -- no data changes.
4. If eligible, locate every Rating recorded by this member and anonymise it: the rating's value is retained for its influence on future meal selection, but its association with this specific member is removed.
5. Set the Member Profile's status to Left.
6. Remove the Member Profile from the household's active member list (it no longer appears in member counts, plan-participant lists, or grocery-list "added by" attributions).
7. Trigger FEAT-09.SPEC-013 (Member Left Household Notification) to the household's current organiser.
8. End the leaving member's session and sign them out of the household.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Departure succeeds | Member is an Other Adult Member (not organiser) at processing time | Ratings anonymised; Member Profile.status set to Left and removed from the active list | Member is signed out and routed to FEAT-01.SPEC-001 with "You've left the household."; organiser receives FEAT-09.SPEC-013 | FEAT-09.SPEC-005, FEAT-01.SPEC-001, FEAT-09.SPEC-013, FEAT-03, FEAT-06, FEAT-12 |
| Departure blocked -- now organiser | Member's member_type has changed to Organiser since the confirmation screen loaded (a hand-over completed in another session) | No data changes | FEAT-09.SPEC-005 shows "You're the organiser now -- hand over the role or delete the household to leave." | FEAT-09.SPEC-005 |
| Automation failure | Processing error after eligibility passes but before both the rating anonymisation and the status change are durably applied | No partial change persists -- rating anonymisation and Member Profile status change apply together or not at all | FEAT-09.SPEC-005 shows an inline error; the member remains a full household member | FEAT-09.SPEC-005 |

## Data Model

**Reads:** Member Profile -- member_type (eligibility check); Rating -- every record where member is the leaving member.
**Creates:** None.
**Updates:** Rating -- member association removed (anonymised) on every rating recorded by the leaving member; Member Profile -- status set to Left.
**Deletes:** None -- the Member Profile record and its anonymised ratings are retained, not purged (feature-overview.md, Entity-Lifecycle Coverage Matrix: soft removal, retained indefinitely as historical record).

## Business Rules

- A member who leaves on their own keeps their ratings only as anonymous influence on future plans; this is distinct from organiser-initiated removal (FEAT-18), which deletes the departing member's dietary rules and ratings outright (XBR-16). This automation never deletes a Rating.
- The organiser can never trigger this automation against herself; XBR-15 requires a hand-over or household deletion first, enforced by FEAT-09.SPEC-011 and re-checked here at processing time, not only at screen entry.
- The leaving member's Member Profile has no restore path: a later re-invitation of the same person creates a genuinely new Member Profile rather than reactivating this one (XBR-18).
- Rating anonymisation and the Member Profile status change apply together as one unit -- no state exists where ratings are anonymised but the profile is still Active, or the profile is Left but ratings still carry attribution.

## Edge Cases

- **Member is handed the organiser role (accepts via FEAT-09.SPEC-004) at almost exactly the same moment they confirm leaving via FEAT-09.SPEC-005** -- Reject-with-refresh: whichever change is recorded first wins. If the hand-over acceptance is recorded first, this automation's eligibility re-check (Processing Logic, step 2-3) finds the member is now organiser and blocks the departure.
- **The member has multiple ratings across several past weeks' plans** -- Every rating recorded by the member is anonymised in the same processing run; none are left attributed and none are skipped.
- **The member has never rated any meal** -- Anonymisation step (Processing Logic, step 4) has no records to act on; the Member Profile status change (step 5) still proceeds normally.
- **The leaving member is the only Other Adult Member, leaving the household with only the organiser** -- Departure proceeds normally; a household with only its organiser as an active adult member is a valid state.
- **Concurrent trigger firing (defensive case -- the same member somehow triggers this automation twice, e.g., a double network retry)** -- The second run finds the Member Profile already Left (idempotent check against current status) and takes no further action; ratings are not anonymised a second time since the association removal is already applied and has no further effect to repeat.
- **Trigger fires while a previous run is still in flight for the same member** -- FEAT-09.SPEC-005's "Yes, Leave" button is disabled during processing, preventing a second trigger from the same session; a second trigger from a different session for the same member processes against the Member Profile's current status and, if already Left, takes the idempotent no-further-action path above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-005 (Leave Household) | Triggered by (inbound) | The final "Yes, Leave" confirmation fires this automation |
| FEAT-09.SPEC-005 (Leave Household) | Affects (outbound) | Returns success or the organiser-blocked reason |
| FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules) | References (inbound) | Eligibility re-check (Other Adult Member only) |
| FEAT-09.SPEC-013 (Member Left Household Notification) | Triggers (outbound) | Notifies the organiser of the departure |
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Affects (outbound) | Destination after the leaving member's session ends |
| FEAT-03 (AI Weekly Dinner Plan Generation), FEAT-06 (Shared Grocery List) | Affects (outbound) | Read the updated Member Profile status so future plans and lists stop accounting for the departed member |
| FEAT-12 (Meal Rating & Preference Learning) | Affects (outbound) | Continues to read the member's ratings, now anonymised, as ongoing influence on selection |

## Analytics and Success Signals

- **member_left_household** (ratings_anonymised_count) -- N/A -- "Household Member Participation" measures a household gaining and keeping an engaged other adult member; a departure is the inverse signal and does not itself feed a positive contribution to that target, so this event is retained only to explain drops in household participation when reading that metric, not cited as a direct contributor
- **member_departure_blocked_organiser** -- N/A -- diagnostic signal only, confirming XBR-15's block is exercised correctly

## Acceptance Criteria

**FEAT-09.SPEC-008-AC-01:** Given Sam (Other Adult Member) confirms leaving on FEAT-09.SPEC-005, when this automation processes the departure, then every rating he recorded is anonymised and his Member Profile status is set to Left.

**FEAT-09.SPEC-008-AC-02:** Given Sam's departure is processed successfully, when processing completes, then he is signed out and routed to FEAT-01.SPEC-001, and Maya (the organiser) receives FEAT-09.SPEC-013.

**FEAT-09.SPEC-008-AC-03:** Given Sam accepts a hand-over and becomes organiser in another session at almost the same moment he confirms leaving, and the hand-over is recorded first, when this automation runs its eligibility check, then the departure is blocked with "You're the organiser now -- hand over the role or delete the household to leave."

**FEAT-09.SPEC-008-AC-04:** Given Sam has never rated any meal, when his departure is processed, then his Member Profile status is still set to Left and no rating anonymisation step has any records to act on.

**FEAT-09.SPEC-008-AC-05:** Given Sam is the household's only Other Adult Member, when his departure is processed, then the household is left with only Maya as an active adult member, which is a valid resulting state.

**FEAT-09.SPEC-008-AC-06:** Given a processing error occurs after eligibility passes but before both the rating anonymisation and the status change are durably applied, when the failure occurs, then Sam remains a full active member with no partial anonymisation or status change.

**FEAT-09.SPEC-008-AC-07:** Given this automation is somehow triggered twice for the same already-Left member, when the second run processes, then it finds the profile already Left and takes no further action, leaving the anonymised ratings unaffected.

**FEAT-09.SPEC-008-AC-08:** Given Sam's departure completes, when FEAT-03 next generates or FEAT-06 next recalculates a plan, then neither accounts for Sam, since his Member Profile no longer appears in the active member list.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, blocked -- now organiser, automation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Organiser Hand-Over Processing

## Overview

**Name:** Organiser Hand-Over Processing
**ID:** FEAT-09.SPEC-009
**Type:** Automation
**Purpose:** On hand-over acceptance, transfers the Household's organiser field and swaps both members' roles atomically.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- Re-validating the hand-over request is still outstanding and addressed to the accepting member at the moment of processing
- Transferring Household.organiser to the accepting member
- Setting the accepting member's member_type to Organiser and the former organiser's member_type to Other Adult Member, as one atomic update
- Clearing the pending hand-over request

**Non-Goals:**
- The recipient's accept/decline decision itself -- owned by FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance), which triggers this automation only on accept
- Initiating a hand-over or selecting a recipient -- owned by FEAT-09.SPEC-003 (Organiser Hand-Over Initiation)
- Notifying either party of the completed transfer beyond what each screen shows inline (FEAT-09.SPEC-004's own success confirmation to the new organiser) -- feature-overview.md's Communications field names a request notification (FEAT-09.SPEC-014) but no separate "hand-over completed" notification; this automation triggers none

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Recipient taps "Accept" | FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Fires on the recipient's accept action, before the request's current standing is re-confirmed | Household reference, accepting member's Member Profile reference, former organiser's Member Profile reference |

## Processing Logic

1. Receive the household reference and the accepting member's identity from FEAT-09.SPEC-004.
2. Re-check per FEAT-09.SPEC-010 that a hand-over request is still outstanding for this household and is still addressed to this accepting member.
3. If no longer outstanding (cancelled by the organiser, or the accepting member is no longer eligible), stop processing and return the withdrawn-request status to FEAT-09.SPEC-004 -- no data changes.
4. If still outstanding, set Household.organiser to the accepting member's Member Profile reference.
5. Set the accepting member's member_type to Organiser.
6. Set the former organiser's member_type to Other Adult Member.
7. Clear the household's pending hand-over request.
8. All of steps 4-7 apply together as one atomic update -- no household state exists with the organiser field pointing to one member while member_type fields have not yet caught up, or vice versa.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Hand-over succeeds | Request is still outstanding and addressed to the accepting member at processing time | Household.organiser transferred; both members' member_type swapped; pending request cleared | New organiser sees "You're now the organiser." and is routed to FEAT-01.SPEC-010 with organiser-level access; former organiser's next FEAT-01 action is now gated as Other Adult Member | FEAT-09.SPEC-004, FEAT-01.SPEC-010, FEAT-09.SPEC-003, FEAT-01.SPEC-016 |
| Hand-over rejected -- request withdrawn | Request was cancelled by the organiser, or the accepting member is no longer an eligible Active adult member | No data changes | FEAT-09.SPEC-004 shows the Withdrawn state | FEAT-09.SPEC-004 |
| Automation failure | Processing error after the outstanding-request check passes but before the atomic update completes | No partial change persists -- the organiser field and both member_type fields remain exactly as they were before this run | FEAT-09.SPEC-004 shows an inline error; the request remains outstanding and can be retried | FEAT-09.SPEC-004 |

## Data Model

**Reads:** Household -- organiser, pending_organiser_handover (the workflow marker defined in FEAT-09.SPEC-003); Member Profile -- the accepting member's and the former organiser's current member_type and status.
**Creates:** None.
**Updates:** Household -- organiser (transferred to the accepting member), pending_organiser_handover (cleared); Member Profile -- member_type on both the accepting member (to Organiser) and the former organiser (to Other Adult Member).
**Deletes:** None.

## Business Rules

- The household always has exactly one organiser (XBR-15): this automation's atomic update (Processing Logic, step 8) guarantees no moment exists with zero or two organisers.
- Only an outstanding request addressed to the accepting member can be processed; a stale or already-resolved request is rejected, never silently re-applied.
- The former organiser retains full household access as an Other Adult Member immediately after the swap -- nothing about her access is revoked beyond the organiser-only entitlements themselves (feature-overview.md, Primary Flows & Alternates).
- This automation never runs against a household with no outstanding hand-over request; FEAT-09.SPEC-004 has no entry point without one.

## Edge Cases

- **Organiser cancels the request at almost exactly the same moment the recipient accepts** -- Reject-with-refresh: whichever change is recorded first wins. If the cancel is recorded first, this automation's re-check (Processing Logic, step 2-3) finds no outstanding request and stops with the Withdrawn outcome; if the accept is recorded first, the transfer proceeds and the near-simultaneous cancel attempt (FEAT-09.SPEC-003's own edge case) is itself rejected against the now-completed transfer.
- **Accepting member's status changed to Left between the request being sent and the accept being processed (defensive case, since FEAT-09.SPEC-003 already cancels the request when the recipient leaves)** -- The eligibility re-check finds the recipient no longer an Active adult member and rejects with the Withdrawn outcome, as a second line of defense behind FEAT-09.SPEC-003's own cancellation.
- **This automation runs twice for the same accepted request (defensive case, e.g., a double network retry)** -- The second run finds no outstanding request (already cleared by the first run) and takes the Withdrawn path with no further effect; the organiser field and member_type fields are not swapped a second time.
- **Former organiser has an organiser-only screen open at the moment the swap completes** -- Her next action against an organiser-only control is rejected per FEAT-01.SPEC-016's own edge case ("Only the organiser can change this"), since authorization is evaluated at action time, not at screen-load time; this automation itself performs no live push to her open screen.
- **Trigger fires while a previous run is still in flight for the same household** -- FEAT-09.SPEC-004's Accept button is disabled during processing, preventing a second trigger from the same session; a second trigger for the same household (e.g., a retried request) processes against the household's current state and, if the request is already cleared, takes the Withdrawn path above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Triggered by (inbound) | Accept fires this automation |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Affects (outbound) | Returns success or the withdrawn-request status |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Whether the request is still outstanding and addressed to the accepting member |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Affects (outbound) | The initiating organiser's screen no longer reflects an outstanding request afterward |
| FEAT-01.SPEC-010 (Household Settings Hub) | Affects (outbound) | Reflects the new organiser's access immediately after the swap |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Governs the former organiser's now-restricted access to organiser-only actions |

## Analytics and Success Signals

- **organiser_role_transferred** (household member count) -- N/A -- no Stage 2 metric measures organiser role transfers; "Household Member Participation" measures a member joining and using the shared plan and list, not which member holds the organiser role, so this signal is retained for operational visibility only

## Acceptance Criteria

**FEAT-09.SPEC-009-AC-01:** Given Sam accepts a still-outstanding hand-over request from Maya, when this automation processes the acceptance, then Household.organiser is set to Sam, Sam's member_type becomes Organiser, and Maya's member_type becomes Other Adult Member.

**FEAT-09.SPEC-009-AC-02:** Given the atomic update in AC-01 completes, when Sam next opens FEAT-01.SPEC-010, then he has full organiser-level access and Maya has Other Adult Member-level access.

**FEAT-09.SPEC-009-AC-03:** Given Maya cancels the request at almost exactly the same moment Sam accepts it, and the cancel is recorded first, when this automation processes Sam's accept, then it finds no outstanding request and FEAT-09.SPEC-004 shows the Withdrawn state.

**FEAT-09.SPEC-009-AC-04:** Given Sam's accept is recorded before Maya's near-simultaneous cancel, when both are processed, then the transfer completes and Maya's cancel attempt against the now-completed transfer is itself rejected.

**FEAT-09.SPEC-009-AC-05:** Given a processing error occurs after the outstanding-request check passes but before the atomic update completes, when the failure occurs, then Household.organiser and both members' member_type remain exactly as before, and FEAT-09.SPEC-004 shows an inline error with the request still outstanding.

**FEAT-09.SPEC-009-AC-06:** Given this automation is somehow triggered twice for the same already-processed request, when the second run executes, then it finds no outstanding request and performs no further change.

**FEAT-09.SPEC-009-AC-07:** Given Maya (the former organiser) has FEAT-01.SPEC-008 (an organiser-only screen) open at the moment the swap completes, when she next attempts to save a budget change, then it is rejected with "Only the organiser can change this," per FEAT-01.SPEC-016.

**FEAT-09.SPEC-009-AC-08:** Given the transfer completes successfully, when Maya next opens FEAT-09.SPEC-003, then it no longer shows an outstanding request and behaves as a non-organiser attempting to reach that screen (per FEAT-09.SPEC-011).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (succeeds, rejected -- withdrawn, automation failure) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Invitation & Membership Validation Rules

## Overview

**Name:** Invitation & Membership Validation Rules
**ID:** FEAT-09.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs the contact-detail requirement, re-invite blocking, the 14-day expiry window, hand-over-recipient eligibility, and the accept/revoke/expiry race resolution.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership
**Governed Entity:** Invitation (plus the hand-over eligibility conditions on Member Profile and Household.organiser)

## Scope and Non-Goals

**In Scope:**
- Field validation for the Invitation entity's contact_detail
- Re-invite blocking against existing active/invited members and outstanding invitations
- The 14-day expiry window's definition (its enforcement is FEAT-09.SPEC-006's)
- Hand-over-recipient eligibility (an Active adult member, never the current organiser)
- The race resolution between acceptance, revocation, and expiry on the same Invitation
- Default values for Invitation fields at creation

**Non-Goals:**
- Who may send, revoke, or hand over -- role-based authorization is governed by FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules), not this spec
- Sign-in email and password format for the invitee's new account -- governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules), a general account-field concern this feature references rather than duplicates
- Actually performing the expiry transition, the acceptance processing, or the hand-over transfer -- owned respectively by FEAT-09.SPEC-006, FEAT-09.SPEC-007, and FEAT-09.SPEC-009, which enforce the rules defined here

## Governed Entity

**Entity:** Invitation
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| contact_detail | text | The invitee's contact detail, entered by the organiser as a label for the invitation and for re-invite blocking |
| status | enum | Sent, Accepted, Revoked, or Expired |
| sent_by | reference | The organiser who created the invitation |
| sent_date | date | When the invitation was created (used to compute the 14-day expiry window) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Household Invitations Manager | On contact detail field blur and on Send Invitation submit (creation); on Revoke and Resend actions |
| FEAT-09.SPEC-002 | Invitation Acceptance | On screen load (invitation validity) and on Accept & Join submit (re-check) |
| FEAT-09.SPEC-006 | Invitation Expiry | On each scheduled run (expiry window and precedence against a concurrent accept/revoke) |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | On acceptance processing (race resolution) |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | On recipient selection and on Send Request submit (hand-over-recipient eligibility) |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | On Accept submit (whether the request is still outstanding) |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | On processing (re-check that the request is still outstanding) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| contact_detail | Required, non-empty | Always | On blur, on submit | "Enter an email address or phone number" | Yes |
| contact_detail | Must be a valid email address format OR a valid phone number format | Always | On blur | "Enter a valid email address or phone number" | Yes |
| contact_detail | Max 254 characters | Always | On blur | "That's too long -- enter a shorter email address or phone number" | Yes |
| status | No validation beyond data type | Always -- system-managed, never directly entered by any user | -- | -- | -- |
| sent_by | No validation beyond data type | Always -- system-assigned from the current session | -- | -- | -- |
| sent_date | No validation beyond data type | Always -- system-assigned at creation | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Re-invite blocking against active members | contact_detail | If the entered contact_detail matches the sign_in email of an existing Active Member Profile in this household, the invitation is blocked | "This person is already a member of your household." |
| Duplicate outstanding invitation | contact_detail, status | If the entered contact_detail exactly matches the contact_detail of another invitation with status Sent for this household, the new invitation is blocked | "There's already an outstanding invitation for this contact -- resend it instead of sending a new one." |

## Authorization Rules

N/A -- role-based authorization for every action this feature defines (send, revoke, resend, accept, hand over, leave) is governed entirely by FEAT-09.SPEC-011 (Household Invitations & Membership Authorization Rules), which is this feature's single authoritative home for the role-action matrix. This spec governs only the entity-level and race-condition rules above.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Sent | On create only | No |
| sent_by | The current organiser's Member Profile reference | On create only | No |
| sent_date | Current date | On create only | No |
| expiry_date | Derived: sent_date + 14 days | Always, recalculated whenever sent_date is set (including on a fresh Sent invitation created by Resend) | No |

## Business Rules

- Invitations expire after 14 days from their sent date (feature-overview.md, Validation & Limits) -- this is a fixed, feature-level product decision, stated as a concrete number because it is specific to this feature's own definition, not a platform-wide policy value delegated to build time.
- A household may hold multiple outstanding (Sent) invitations at once (feature-overview.md, Validation & Limits) -- the duplicate-outstanding-invitation rule above blocks only an exact contact-detail repeat, not multiple invitations to different people.
- **Accept/revoke/expiry race resolution:** the Invitation entity's status transition follows first-committed-wins. Whichever of an acceptance (FEAT-09.SPEC-007), a revocation (FEAT-09.SPEC-001), or an automatic expiry (FEAT-09.SPEC-006) is durably recorded first against a given invitation determines its outcome; every other concurrent attempt against the same invitation is rejected and shown the invitation's now-current status (reject-with-refresh), per the dependency map's Contention note for Invitation.
- **Hand-over-recipient eligibility:** a hand-over recipient must be an Active adult member with member_type Other Adult Member; the current organiser can never be selected as her own recipient, and a member with status Invited, Left, or Removed is never eligible.
- **Hand-over request standing:** at most one outstanding hand-over request exists per household at a time (XBR-15); a request becomes non-outstanding when the organiser cancels it, the recipient accepts or declines it, or the recipient's eligibility is lost (e.g., they leave) while it is pending.
- An Expired invitation resend creates a new Invitation record (new sent_date, new expiry_date, status Sent) rather than reviving the expired one in place; the expired record is retained as-is.

## Edge Cases

- **Entered contact detail matches an Active member's sign_in email exactly but differs in letter case** -- The comparison is case-insensitive for email addresses; the invitation is still blocked as an already-active member.
- **Entered contact detail is a phone number with no matching Member Profile field to compare against** -- Re-invite blocking against active members applies only when the entered contact detail is in email format (the only contact channel stored on Member Profile.sign_in); a phone-number contact detail is checked only against other outstanding invitations' contact_detail for the duplicate-outstanding rule, never against member sign-in emails.
- **Invitation reaches exactly 14 days and 0 hours since its sent_date at the same moment a scheduled expiry run and an acceptance both arrive** -- First-committed-wins applies exactly as with any other race: whichever transition is recorded first stands, with no special-cased tie-break beyond commit order.
- **Organiser resends an invitation whose original contact_detail no longer matches any real recipient concern (e.g., a typo)** -- Resend uses the original invitation's contact_detail as entered; correcting a typo requires revoking the Expired invitation's record is unnecessary since it is already terminal, and sending a fresh invitation with the corrected contact detail is a new Send action on FEAT-09.SPEC-001, not a Resend.
- **Hand-over recipient list has exactly one eligible member and that member is mid-way through their own Leave Household confirmation (FEAT-09.SPEC-005) when selected** -- If the member's departure (FEAT-09.SPEC-008) is recorded before the hand-over request is sent, they no longer appear as eligible and the organiser's stale selection is rejected with refresh, showing the now-empty eligible-recipient state.
- **Duplicate-outstanding-invitation check runs against a contact detail that matches an invitation revoked seconds earlier** -- A Revoked invitation does not block a new Send; the duplicate check applies only to invitations currently in Sent status.

## Acceptance Criteria

**FEAT-09.SPEC-010-AC-01:** Given Maya leaves the contact detail field empty, when she attempts to send an invitation, then she sees "Enter an email address or phone number" and the invitation is not created.

**FEAT-09.SPEC-010-AC-02:** Given Maya enters text that is neither a valid email nor a valid phone number, when she blurs the field, then she sees "Enter a valid email address or phone number."

**FEAT-09.SPEC-010-AC-03:** Given Maya enters a contact detail matching Sam's sign_in email exactly, when she attempts to send the invitation, then it is blocked with "This person is already a member of your household."

**FEAT-09.SPEC-010-AC-04:** Given Maya enters a contact detail matching Sam's sign_in email but in different letter case, when she attempts to send the invitation, then it is still blocked, since the email comparison is case-insensitive.

**FEAT-09.SPEC-010-AC-05:** Given a Sent invitation already exists for "jane@example.com", when Maya attempts to send a second invitation to the same contact detail, then it is blocked with "There's already an outstanding invitation for this contact -- resend it instead of sending a new one."

**FEAT-09.SPEC-010-AC-06:** Given an invitation was sent exactly 14 days ago and remains Sent, when FEAT-09.SPEC-006 evaluates it, then it is eligible for expiry.

**FEAT-09.SPEC-010-AC-07:** Given an invitation was sent 13 days ago and remains Sent, when FEAT-09.SPEC-006 evaluates it, then it is not yet eligible for expiry.

**FEAT-09.SPEC-010-AC-08:** Given an invitation is revoked at the same moment it becomes eligible for expiry, and the revoke is recorded first, when both are processed, then the invitation's final status is Revoked, not Expired.

**FEAT-09.SPEC-010-AC-09:** Given an invitation is accepted at the same moment Maya revokes it, and the acceptance is recorded first, when both are processed, then the invitation's final status is Accepted and Maya's revoke is rejected with the current (Accepted) status shown.

**FEAT-09.SPEC-010-AC-10:** Given Maya selects herself as a hand-over recipient (defensive case -- she is excluded from the eligible list), when the recipient list is built, then Maya never appears as a selectable option.

**FEAT-09.SPEC-010-AC-11:** Given Maya selects a member whose status is Left, when the eligibility check runs, then the selection is rejected as ineligible.

**FEAT-09.SPEC-010-AC-12:** Given a household already has one outstanding hand-over request, when the organiser attempts to view FEAT-09.SPEC-003's recipient list, then no new request can be sent until the existing one resolves or is cancelled (XBR-15).

**FEAT-09.SPEC-010-AC-13:** Given an Expired invitation is resent, when the resend is processed, then a new Invitation record is created with a fresh sent_date, a recalculated expiry_date 14 days later, and status Sent, leaving the original Expired record unchanged.

**FEAT-09.SPEC-010-AC-14:** Given Maya enters a contact detail that is a phone number matching no Member Profile field, when she attempts to send the invitation, then the active-member re-invite check does not block it (no email field exists to compare), while the duplicate-outstanding-invitation check still applies.

**FEAT-09.SPEC-010-AC-15:** Given a hand-over request's sole eligible recipient completes their own departure (FEAT-09.SPEC-008) before the request is sent, when Maya's stale selection is submitted, then it is rejected with refresh and the recipient list reflects no eligible members.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 0 (N/A -- covered by FEAT-09.SPEC-011) | 0 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Household Invitations & Membership Authorization Rules

## Overview

**Name:** Household Invitations & Membership Authorization Rules
**ID:** FEAT-09.SPEC-011
**Type:** Logic/Rule
**Purpose:** Governs who can send, revoke, or hand over invitations, who can leave on their own, and the unauthorized-visitor experience, as the single authoritative home for role-gated behavior across every screen in this feature.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership
**Governed Entity:** Invitation and Member Profile (the access dimension -- which role may view or act on each, across FEAT-09.SPEC-001 through FEAT-09.SPEC-005)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for every action this feature defines: send/revoke/resend an invitation, accept an invitation, initiate/accept/decline a hand-over, and leave the household
- The exact unauthorized experience for every role denied an action
- What an unauthenticated visitor and an unauthorized visitor (someone with no invitation addressed to them) see across this feature's screens
- The organiser's leave/delete block per XBR-15

**Non-Goals:**
- Field-level validation and the accept/revoke/expiry race resolution -- governed by FEAT-09.SPEC-010, which is more specific to the Invitation entity's own data rules
- Authorization for features outside this one (e.g., who may edit a member's dietary rules) -- each feature owning its own entities defines its own authorization; this spec covers only the actions FEAT-09 itself defines
- Operator (Riley) authentication or the mechanics of the read-only support view itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only states that Riley has no access of any kind to this feature's actions or data (Access Matrix: Household Invitations column is None for Riley), not even a read-only view

## Governed Entity

**Entity:** Invitation and Member Profile (access dimension)
**Source:** Feature Dependency Map; Access Matrix in user-persona.md

| Field | Data Type | Description |
|-------|-----------|-------------|
| (No new fields -- this spec governs actions on the existing Invitation and Member Profile fields already defined in FEAT-09.SPEC-010) | -- | -- |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Household Invitations Manager | On screen entry and on every send/revoke/resend action |
| FEAT-09.SPEC-002 | Invitation Acceptance | On screen entry (unauthorized-visitor and already-a-member states) |
| FEAT-09.SPEC-003 | Organiser Hand-Over Initiation | On screen entry and on Send Request |
| FEAT-09.SPEC-004 | Organiser Hand-Over Acceptance | On screen entry and on Accept/Decline |
| FEAT-09.SPEC-005 | Leave Household | On screen entry and on the final leave confirmation |
| FEAT-09.SPEC-007 | Invitation Acceptance Processing | On processing (re-check that acceptance is not attempted by an already-authenticated household member) |
| FEAT-09.SPEC-008 | Member Departure Processing | On processing (re-check the leaving member is not the organiser) |
| FEAT-09.SPEC-009 | Organiser Hand-Over Processing | On processing (re-check the accepting member is not already the organiser) |

## Field Validation Rules

N/A -- this spec governs authorization (who may act), not field-level data validation, which is FEAT-09.SPEC-010's scope. No field in the governed entities carries validation rules distinct from those already defined there.

## Cross-Field Rules

N/A -- no cross-field data rule applies at the authorization layer; every rule here is a role-action-condition triple, captured in full in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Send an invitation | Maya (Organiser) | No matching duplicate (FEAT-09.SPEC-010) | Screen entry point (FEAT-09.SPEC-001) not shown to Sam at all; a direct navigation attempt shows "Only the organiser can manage invitations" |
| Revoke a Sent invitation | Maya (Organiser) | Always, on any Sent invitation for her household | Same as above -- Sam never reaches the screen where this action lives |
| Resend an Expired invitation | Maya (Organiser) | Always, on any Expired invitation for her household | Same as above |
| View the invitation list and history | Maya (Organiser) | Always | Sam, both Jordan rows, and Riley see no entry point (Access Matrix: Household Invitations is None for all four) |
| Accept an invitation | The specific unauthorized visitor addressed by the invitation link | The invitation referenced by the link is still Sent (FEAT-09.SPEC-010) | An invitation no longer Sent shows "This invitation is no longer valid" with the specific reason; the invitation content itself is only ever shown to the holder of its own link, never to anyone else |
| Accept an invitation | Any already-authenticated household member (Maya, Sam) | Never -- an existing member has nothing to accept | "You're already a member of this household." with a link to the current plan, not the acceptance form |
| Accept an invitation | Kid profiles (young, no login; older, limited login, Later) | Never | N/A -- kid profiles are never invited through this feature (scope-boundaries.md SC-02); no kid-addressed invitation link can exist |
| Initiate a hand-over | Maya (Organiser) | At least one Active Other Adult Member exists and no hand-over request is already outstanding | Screen entry point (FEAT-09.SPEC-003) not shown to Sam; a direct navigation attempt shows "Only the organiser can hand over the role" |
| Accept or decline a hand-over | The specific Active adult member the outstanding request is addressed to | A hand-over request is outstanding and addressed to them | A member with no request addressed to them has no entry point to FEAT-09.SPEC-004; the current organiser opening it directly sees "You're the organiser -- there's nothing to accept here." |
| Leave the household on her own | Sam (Other Adult Member) | Always, for any Active Other Adult Member -- her member_type is not Organiser | -- |
| Leave the household on her own | Maya (Organiser) | Never, while she holds the organiser role (XBR-15) | Entry point (FEAT-09.SPEC-005) not shown in settings to the organiser; a direct navigation attempt shows "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18 |
| Leave the household on her own | Kid profiles (young, no login; older, limited login, Later) | Never | N/A -- kid profiles are not household members with an account of their own to leave from (Access Matrix: Household Invitations and Account & Data are None) |
| View or act on any action this feature defines | Riley (Operator, support) | Never | N/A -- Riley's access is limited entirely to the separate read-only support view (FEAT-22, XBR-14); no screen or action in this feature has any operator path, not even a read-only one |

## Defaults and Derivations

N/A -- this spec assigns no default or derived field values of its own; all defaults for Invitation fields are defined in FEAT-09.SPEC-010.

## Business Rules

- Every screen in this feature (FEAT-09.SPEC-001 through FEAT-09.SPEC-005) references this spec for its role-gated behavior rather than restating access rules per screen, per this feature's Shared Validation section.
- An unauthorized visitor -- anyone not signed in as a member of the household, and holding no invitation link addressed to them -- sees only a sign-in screen (FEAT-01.SPEC-001) or the specific invitation-acceptance screen for a link genuinely addressed to them (FEAT-09.SPEC-002), never any other household data, per user-persona.md's Access Matrix notes.
- A session that expires mid-flow on any screen in this feature returns the user to FEAT-01.SPEC-001 (or, for an unauthenticated invitee, remains on FEAT-09.SPEC-002 with the invitation preserved) with the message "Your session has expired. Sign in to continue."
- XBR-15 (exactly one organiser at all times) is enforced at both the screen layer (Leave Household and Organiser Hand-Over Initiation each check the acting member's current role) and the processing layer (FEAT-09.SPEC-008 and FEAT-09.SPEC-009 each re-check at the moment of processing, not only at screen entry), since a role can change between screen load and action.
- Roles named in this spec trace exactly to the Access Matrix in user-persona.md: Maya (Organiser), Sam (Other Adult Member), Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later), and Riley (Operator, support, from v1), plus the unauthorized-visitor and unauthenticated states this feature's own screens define. No role beyond these is ever introduced by this feature.

## Edge Cases

- **An unauthenticated visitor attempts to navigate directly to FEAT-09.SPEC-001** -- Redirected to FEAT-01.SPEC-001; no household or invitation data is ever rendered, even momentarily, during the redirect.
- **Sam attempts to bypass the hidden invitation-management controls via a direct navigation URL to FEAT-09.SPEC-001** -- The destination screen itself enforces the same authorization check on entry, showing "Only the organiser can manage invitations" regardless of how the screen was reached.
- **The organiser role changes hands mid-session (FEAT-09.SPEC-009 completes while the former organiser has FEAT-09.SPEC-001 open)** -- Her next action against an organiser-only control on that screen (e.g., attempting to revoke an invitation) is rejected with "Only the organiser can manage invitations," since authorization is evaluated at action time, not at screen-load time.
- **A visitor holds a link to an invitation addressed to someone else (guessed or shared incorrectly)** -- The invitation is shown only to whoever holds its unique link; this spec does not distinguish "wrong person" from "right person" at the authorization layer, since the link itself is the sole credential FEAT-09.SPEC-002 checks -- possession of a valid, still-Sent link is what "addressed to them" means functionally.
- **Riley's operator session somehow reaches a FEAT-09 URL directly (defensive case)** -- No such entry point exists for the operator role in this feature; no invitation or membership data is ever rendered to an operator session outside FEAT-22's own read-only support view.
- **A kid profile somehow attempts a direct action against this feature (defensive case, e.g., a stale token)** -- No login exists for a young kid profile and the older-kid login (Later) has no entitlement to this feature's actions (Access Matrix: None); no action succeeds under either identity.

## Acceptance Criteria

**FEAT-09.SPEC-011-AC-01:** Given Maya (Organiser) is signed in, when she opens FEAT-09.SPEC-001, then she has full access to send, revoke, and resend invitations.

**FEAT-09.SPEC-011-AC-02:** Given Sam (Other Adult Member) attempts to navigate directly to FEAT-09.SPEC-001, when the request is made, then he sees "Only the organiser can manage invitations" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-011-AC-03:** Given an unauthorized visitor opens a link to an invitation addressed to them that is still Sent, when FEAT-09.SPEC-002 loads, then they can view and accept it.

**FEAT-09.SPEC-011-AC-04:** Given Sam (already a household member) opens any invitation link, when FEAT-09.SPEC-002 loads, then he sees "You're already a member of this household." instead of the acceptance form.

**FEAT-09.SPEC-011-AC-05:** Given Maya (Organiser) attempts to navigate directly to FEAT-09.SPEC-005 (Leave Household), when the request is made, then she sees "You're the organiser -- hand over the role or delete the household to leave." with links to FEAT-09.SPEC-003 and FEAT-18.

**FEAT-09.SPEC-011-AC-06:** Given Sam (Other Adult Member) opens FEAT-09.SPEC-005, when the screen loads, then he can proceed to confirm leaving.

**FEAT-09.SPEC-011-AC-07:** Given Sam attempts to navigate directly to FEAT-09.SPEC-003 (Organiser Hand-Over Initiation), when the request is made, then he sees "Only the organiser can hand over the role" and is returned to FEAT-01.SPEC-010.

**FEAT-09.SPEC-011-AC-08:** Given Maya (still organiser) opens FEAT-09.SPEC-004 directly via a stale link, when the screen loads, then she sees "You're the organiser -- there's nothing to accept here."

**FEAT-09.SPEC-011-AC-09:** Given the organiser role transfers to Sam while Maya still has FEAT-09.SPEC-001 open, when Maya (now Other Adult Member) attempts to revoke an invitation, then it is rejected with "Only the organiser can manage invitations."

**FEAT-09.SPEC-011-AC-10:** Given an unauthenticated visitor attempts to open FEAT-09.SPEC-003 directly, when the request is made, then they are redirected to FEAT-01.SPEC-001 with no household data rendered.

**FEAT-09.SPEC-011-AC-11:** Given Riley (Operator) is signed in only through the separate read-only support view (FEAT-22), when any FEAT-09 screen is requested under his identity, then no such entry point exists and no invitation or membership data is returned.

**FEAT-09.SPEC-011-AC-12:** Given a young kid profile has no login, when any request is made under that profile's identity against any action in this feature, then no action succeeds, since no valid session can exist for it.

**FEAT-09.SPEC-011-AC-13:** Given Sam attempts to bypass the hidden invitation-management controls via a direct URL to FEAT-09.SPEC-001, then the destination screen itself blocks the action with "Only the organiser can manage invitations," regardless of the navigation path used.

**FEAT-09.SPEC-011-AC-14:** Given a member with no hand-over request addressed to them opens FEAT-09.SPEC-004 (defensive case, e.g., a guessed URL), then no request is shown and no accept/decline action is available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- covered by FEAT-09.SPEC-010) | 0 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: Invitation Accepted Confirmation

## Overview

**Name:** Invitation Accepted Confirmation
**ID:** FEAT-09.SPEC-012
**Type:** Notification
**Purpose:** Tells the organiser once an invited adult has accepted and joined, so she knows her household is now shared.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The confirmation delivered to the organiser when an invitation she sent is accepted
- Its single channel (in-app), content, and delivery behavior

**Non-Goals:**
- Delivery to the newly joined member -- their own confirmation is the first-use onboarding experience itself (FEAT-15), not this notification, which is addressed only to the organiser
- Email or push delivery -- this feature carries no Integration spec for transactional email or device-notification delivery (feature-dependency-map.md, External Touchpoints); the invitation itself is a self-shared link (FEAT-09.SPEC-001), and this confirmation is a lightweight in-app signal consistent with that same pattern
- The acceptance processing itself (creating the Member Profile, routing to onboarding) -- owned by FEAT-09.SPEC-007, which triggers this notification only after that processing succeeds

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when an invitation is accepted | Maya (the organiser) is the household's primary in-product user and checks the product regularly for the plan and list; an in-app signal reaches her without adding an email dependency this feature does not otherwise carry |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invitation acceptance succeeds | FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Fires once, immediately after the new Member Profile is created and the Invitation's status is set to Accepted | New member's display_name, the organiser's Member Profile reference |

## Audience and Preferences

**Recipients:** Maya -- the household's current organiser at the moment of acceptance, per the Access Matrix in user-persona.md. Only the organiser receives this confirmation; no other role is entitled to it, since sending and tracking invitations is Full only for the organiser (Access Matrix, Household Invitations column).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this confirmation has no dedicated on/off preference | -- | Always on | -- |

There is no preference toggle for this notification: feature-overview.md's Communications field states it plainly as a standing confirmation the organiser receives whenever an invitation is accepted, with no stated opt-out, distinct from the general plan-ready/nightly-nudge preferences owned by FEAT-07/FEAT-13.

**Quiet Hours:** N/A -- this notification is in-app only and reflects a durable state change (a new member has joined) rather than a time-sensitive interruption; there is no quiet-hours concept for an in-app confirmation the organiser sees on her next visit, and the product defines no quiet-hours window for this feature's notifications.

## Content Definition

**In-app:**
- **Title:** {new_member_name} joined your household
- **Body:** {new_member_name} accepted your invitation and now sees the same plan and grocery list.
- **CTA:** View household -- deep-links to FEAT-01.SPEC-010 (Household Settings Hub) for this household

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {new_member_name} | Member Profile -- display_name (the newly created member) | Sam | Never empty -- display_name is required at acceptance (FEAT-09.SPEC-002, FEAT-09.SPEC-010) |

## Delivery Rules

**Batching:** No batching -- each accepted invitation produces exactly one confirmation, since an organiser accepting multiple invitations sends each one deliberately and expects to see each acceptance distinctly.
**Deduplication:** At most one confirmation per accepted Invitation record. FEAT-09.SPEC-007 creates the Member Profile and transitions the Invitation to Accepted exactly once per invitation (XBR-18); this notification fires exactly once per that single transition, never re-fired by a later re-read of the same Invitation.
**Retry on failure:** N/A -- an in-app notification has no separate delivery step to retry; it renders directly from the current state of the household's notification list the next time the organiser opens the product.
**Expiry:** This confirmation does not expire -- it remains visible in the organiser's in-app notification history until she dismisses or reads it, since a record of who joined and when is durable household information, not a time-sensitive alert.

## Edge Cases

- **Organiser's role has been handed over (FEAT-09.SPEC-009) between the invitation being sent and being accepted** -- The confirmation is delivered to whoever is the household's organiser at the moment of acceptance, not to whoever originally sent the invitation; this matches the Recipients definition above (the current organiser).
- **The newly created Member Profile is somehow removed (FEAT-18) moments after acceptance, before the organiser opens the notification** -- The confirmation still renders normally; it reports a fact that occurred (a member joined) and is not retracted by a later, unrelated removal.
- **Organiser is signed in on two devices when the acceptance occurs** -- The confirmation appears in the organiser's in-app notification list on both devices, since it belongs to the household's organiser identity, not to a single device session.
- **Two invitations are accepted in quick succession** -- Each produces its own confirmation with its own new_member_name; they are not merged, per the no-batching rule above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Triggered by (inbound) | Successful acceptance fires this notification |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | The CTA deep-links here |
| FEAT-09.SPEC-001 (Household Invitations Manager) | References (inbound) | The organiser can also see the accepted invitation directly on that screen without opening this notification |

## Analytics and Success Signals

- **invitation_accepted_confirmation_delivered** (-- ) -- supports success-metrics.md: "Household Member Participation" (confirms the organiser is informed of the exact moment her household gains the participating other adult member the metric measures)
- **invitation_accepted_confirmation_opened** (-- ) -- N/A -- no Stage 2 metric distinguishes a delivered confirmation from an opened one; retained as an engagement-diagnostic signal

## Acceptance Criteria

**FEAT-09.SPEC-012-AC-01:** Given Sam's acceptance of Maya's invitation is processed successfully, when FEAT-09.SPEC-007 completes, then Maya receives the in-app notification "Sam joined your household."

**FEAT-09.SPEC-012-AC-02:** Given Maya opens the confirmation, when she taps "View household", then she is taken to FEAT-01.SPEC-010.

**FEAT-09.SPEC-012-AC-03:** Given Maya has handed over the organiser role to a prior recipient before a separate invitation she originally sent is accepted, when that acceptance completes, then the confirmation is delivered to the household's current organiser, not to Maya.

**FEAT-09.SPEC-012-AC-04:** Given two invitations are accepted within moments of each other, when both complete, then Maya receives two separate confirmations, each naming its own new member.

**FEAT-09.SPEC-012-AC-05:** Given Maya is signed in on two devices, when an invitation is accepted, then the confirmation appears in her in-app notification list on both devices.

**FEAT-09.SPEC-012-AC-06:** Given this notification has no preference toggle, when an invitation is accepted, then the confirmation is always delivered regardless of any other notification setting the organiser has configured.

**FEAT-09.SPEC-012-AC-07:** Given the confirmation has been delivered and sits unread in Maya's in-app notification list, when a week passes with no dismissal, then it remains visible, since this notification does not expire.

**FEAT-09.SPEC-012-AC-08:** Given the newly joined member is removed from the household moments after acceptance, when Maya later opens the still-undismissed confirmation, then it still renders normally, reporting the acceptance that did occur.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no toggle) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: Member Left Household Notification

## Overview

**Name:** Member Left Household Notification
**ID:** FEAT-09.SPEC-013
**Type:** Notification
**Purpose:** Tells the organiser when an other adult member leaves on their own, so she understands why the household's membership changed.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The notification delivered to the organiser when an Other Adult Member completes leaving the household on their own
- Its single channel (in-app), content, and delivery behavior

**Non-Goals:**
- Delivery to the leaving member -- their own confirmation is the in-screen success message on FEAT-09.SPEC-005 itself, not this notification, which is addressed only to the organiser
- Email or push delivery -- consistent with FEAT-09.SPEC-012, this feature carries no Integration spec for transactional email or device-notification delivery (feature-dependency-map.md, External Touchpoints)
- Notifying the organiser when a member is removed by her own action (FEAT-18) -- that is the organiser's own action and needs no notification to herself; this spec covers only a member leaving on their own initiative
- The departure processing itself (anonymising ratings, removing the profile) -- owned by FEAT-09.SPEC-008, which triggers this notification only after that processing succeeds

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a member's self-initiated departure is processed | Maya (the organiser) needs to understand why her household's membership changed, and an in-app signal reaches her without adding an email dependency this feature does not otherwise carry |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member departure processing succeeds | FEAT-09.SPEC-008 (Member Departure Processing) | Fires once, immediately after the leaving member's ratings are anonymised and their Member Profile status is set to Left | Leaving member's display_name (as it was before status changed) |

## Audience and Preferences

**Recipients:** Maya -- the household's current organiser at the moment of departure, per the Access Matrix in user-persona.md. Only the organiser receives this notification; no other role is entitled to it, since household membership changes are Full-visibility only for the organiser.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this notification has no dedicated on/off preference | -- | Always on | -- |

There is no preference toggle for this notification: feature-overview.md's Communications field states plainly that "the organiser is also told when a member leaves," with no stated opt-out, distinct from the general plan-ready/nightly-nudge preferences owned by FEAT-07/FEAT-13.

**Quiet Hours:** N/A -- this notification is in-app only and reflects a durable state change (a member has left) rather than a time-sensitive interruption; there is no quiet-hours concept for an in-app notification the organiser sees on her next visit, and the product defines no quiet-hours window for this feature's notifications.

## Content Definition

**In-app:**
- **Title:** {member_name} left your household
- **Body:** {member_name} has left. Their household access has ended, and their past ratings continue to quietly influence future meal choices.
- **CTA:** View household -- deep-links to FEAT-01.SPEC-010 (Household Settings Hub) for this household

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {member_name} | Member Profile -- display_name, captured at the moment of departure before the record's status changes | Sam | Never empty -- display_name is a required field for every Member Profile (FEAT-01.SPEC-014) |

## Delivery Rules

**Batching:** No batching -- each departure produces exactly one notification, since each departure is a distinct household event the organiser should see individually.
**Deduplication:** At most one notification per departure. FEAT-09.SPEC-008 sets a member's status to Left exactly once per member (its own idempotency rule prevents a repeat run from re-anonymising or re-transitioning an already-Left profile); this notification fires exactly once per that single transition.
**Retry on failure:** N/A -- an in-app notification has no separate delivery step to retry; it renders directly from the current state of the household's notification list the next time the organiser opens the product.
**Expiry:** This notification does not expire -- it remains visible in the organiser's in-app notification history until she dismisses or reads it, since a record of who left and when is durable household information, not a time-sensitive alert.

## Edge Cases

- **Organiser's role has been handed over between when the leaving member started their confirmation and when departure processing completes** -- The notification is delivered to whoever is the household's organiser at the moment processing completes, not to whoever was organiser earlier in the session; this matches the Recipients definition above (the current organiser).
- **Organiser is signed in on two devices when the departure is processed** -- The notification appears in the organiser's in-app notification list on both devices, since it belongs to the household's organiser identity, not to a single device session.
- **The leaving member is later re-invited and rejoins as a genuinely new Member Profile (XBR-18)** -- This notification's original record is unaffected; it remains a historical account of the earlier departure and is never retroactively updated or removed by a later, unrelated re-invitation.
- **Two members leave in quick succession (e.g., after a household disagreement)** -- Each produces its own notification with its own member_name; they are not merged, per the no-batching rule above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-008 (Member Departure Processing) | Triggered by (inbound) | Successful departure processing fires this notification |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | The CTA deep-links here |

## Analytics and Success Signals

- **member_left_notification_delivered** (-- ) -- N/A -- "Household Member Participation" measures a household gaining and keeping an engaged other adult member; this notification reports the inverse event (a departure) and is not itself a contribution to that target, so it is retained only for organiser-communication completeness, not cited as a metric contributor
- **member_left_notification_opened** (-- ) -- N/A -- no Stage 2 metric measures notification engagement for this event

## Acceptance Criteria

**FEAT-09.SPEC-013-AC-01:** Given Sam's departure is processed successfully by FEAT-09.SPEC-008, when processing completes, then Maya receives the in-app notification "Sam left your household."

**FEAT-09.SPEC-013-AC-02:** Given Maya opens the notification, when she taps "View household", then she is taken to FEAT-01.SPEC-010.

**FEAT-09.SPEC-013-AC-03:** Given Maya has handed over the organiser role before a member's departure that was already in progress completes, when departure processing finishes, then the notification is delivered to the household's current organiser, not to Maya.

**FEAT-09.SPEC-013-AC-04:** Given Maya is signed in on two devices, when a member's departure is processed, then the notification appears in her in-app notification list on both devices.

**FEAT-09.SPEC-013-AC-05:** Given this notification has no preference toggle, when a member leaves, then the notification is always delivered regardless of any other notification setting the organiser has configured.

**FEAT-09.SPEC-013-AC-06:** Given the notification has been delivered and sits unread, when a week passes with no dismissal, then it remains visible, since this notification does not expire.

**FEAT-09.SPEC-013-AC-07:** Given the departed member is later re-invited and rejoins as a new Member Profile, when Maya later views the original notification, then it still shows the earlier departure unchanged, unaffected by the later re-invitation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no toggle) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: Organiser Hand-Over Request Notification

## Overview

**Name:** Organiser Hand-Over Request Notification
**ID:** FEAT-09.SPEC-014
**Type:** Notification
**Purpose:** Tells the chosen adult member that the organiser has asked them to accept the organiser role, so they know to open the request and respond.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The notification delivered to the selected recipient when the organiser initiates a hand-over request
- Its single channel (in-app), content, and delivery behavior, including what happens if the request is withdrawn before the recipient acts

**Non-Goals:**
- The recipient's accept/decline decision itself -- owned by FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance), which this notification's CTA opens
- Notifying the organiser of the outcome (accept/decline) -- feature-overview.md's Side-Effect Inventory routes a decline back inline into FEAT-09.SPEC-003, and an acceptance's outcome is visible there directly; neither is a separate Notification spec
- Email or push delivery -- consistent with FEAT-09.SPEC-012 and FEAT-09.SPEC-013, this feature carries no Integration spec for transactional email or device-notification delivery (feature-dependency-map.md, External Touchpoints)

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a hand-over request is sent | Sam (the recipient) is an active household member who uses the product regularly for the shared plan and list; an in-app signal reaches him without adding an email or push dependency this feature does not otherwise carry, and a role hand-over is a considered decision, not an urgent same-minute interruption |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Organiser sends a hand-over request | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Fires once, immediately after the request is recorded as outstanding | Current organiser's display_name, recipient's Member Profile reference |

## Audience and Preferences

**Recipients:** Sam -- the specific Active adult member the organiser selected as the hand-over recipient, per the Access Matrix in user-persona.md. Only the addressed recipient receives this notification; no other role sees it, since at most one hand-over request is outstanding per household (XBR-15) and it is addressed to exactly one member.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this notification has no dedicated on/off preference | -- | Always on | -- |

There is no preference toggle for this notification: a hand-over request is a direct, deliberate action from the organiser to one named recipient, and feature-overview.md's Communications field states plainly that "the new organiser is asked to accept a hand-over," with no stated opt-out.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this feature's notifications; a hand-over request has no stated urgency window that quiet hours would need to hold against, since the recipient can act on it whenever they next open the product.

## Content Definition

**In-app:**
- **Title:** {organiser_name} wants you to become the organiser
- **Body:** {organiser_name} has asked you to take over as the household's organiser -- managing the weekly budget, schedule, and settings. Open this to accept or decline.
- **CTA:** Respond -- deep-links to FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) for this request

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {organiser_name} | Member Profile -- display_name (the current organiser at the moment the request is sent) | Maya | Never empty -- display_name is a required field for every Member Profile (FEAT-01.SPEC-014) |

## Delivery Rules

**Batching:** No batching -- at most one hand-over request is ever outstanding per household (XBR-15), so this notification never has more than one pending instance to batch.
**Deduplication:** At most one notification per outstanding request. FEAT-09.SPEC-003 creates one pending request per Send Request action; if the organiser cancels and sends a new request (to the same or a different recipient), that is a new request and produces its own new notification -- the prior one is withdrawn per the Edge Cases below.
**Retry on failure:** N/A -- an in-app notification has no separate delivery step to retry; it renders directly from the current state of the recipient's notification list the next time they open the product.
**Expiry:** This notification does not expire on a timer -- unlike an Invitation, a hand-over request has no stated 14-day window (feature-overview.md defines the 14-day expiry only for Invitations, not for hand-over requests). It remains actionable until the organiser cancels it, the recipient responds, or the recipient loses eligibility, at which point the notification is withdrawn per the Edge Cases below.

## Edge Cases

- **Organiser cancels the request before the recipient responds** -- The notification is withdrawn: if still unread, it no longer appears in the recipient's notification list; if already read but not yet acted on, opening its CTA now shows FEAT-09.SPEC-004's Withdrawn state rather than a live decision. A request must never remain actionable after the organiser has cancelled it.
- **Recipient's eligibility is lost while the notification is pending (e.g., they leave the household)** -- The notification is withdrawn along with the request itself (FEAT-09.SPEC-003's own edge case); a departed member is never shown a still-open request to become organiser of a household they no longer belong to.
- **Organiser cancels one request and immediately sends a new one to the same recipient** -- The first notification is withdrawn and a second, distinct notification is delivered for the new request; they are never merged or treated as an update to the same instance.
- **Recipient reads the notification but does not act, and the organiser reaches for the request's own screen (FEAT-09.SPEC-003) meanwhile** -- The organiser sees the request is still Pending; nothing about the notification's read/unread state changes the request's own outstanding status, which is governed entirely by FEAT-09.SPEC-010.
- **Recipient is signed in on two devices when the request is sent** -- The notification appears in the recipient's in-app notification list on both devices, since it belongs to the recipient's identity, not to a single device session.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Triggered by (inbound) | Send Request fires this notification |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | References (inbound) | A cancel withdraws this notification |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Navigation (outbound) | The CTA deep-links here |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Whether the underlying request remains outstanding |

## Analytics and Success Signals

- **handover_request_notification_delivered** (-- ) -- N/A -- no Stage 2 metric measures hand-over activity; "Household Member Participation" measures a member's joining and use, not the organiser role's transfer
- **handover_request_notification_opened** (-- ) -- N/A -- no Stage 2 metric measures notification engagement for this event

## Acceptance Criteria

**FEAT-09.SPEC-014-AC-01:** Given Maya sends a hand-over request to Sam, when the request is recorded, then Sam receives the in-app notification "Maya wants you to become the organiser."

**FEAT-09.SPEC-014-AC-02:** Given Sam opens the notification, when he taps "Respond", then he is taken to FEAT-09.SPEC-004 with the request shown.

**FEAT-09.SPEC-014-AC-03:** Given Maya cancels the request before Sam has responded, when the cancellation is recorded, then the notification is withdrawn from Sam's unread list, or shows the Withdrawn state if he opens it afterward.

**FEAT-09.SPEC-014-AC-04:** Given Sam leaves the household while the request is pending to him, when his departure is processed, then the notification is withdrawn along with the request.

**FEAT-09.SPEC-014-AC-05:** Given Maya cancels her request to Sam and immediately sends a new request to him, when the new request is recorded, then Sam receives a second, distinct notification, and the first is withdrawn.

**FEAT-09.SPEC-014-AC-06:** Given Sam is signed in on two devices, when the request is sent, then the notification appears in his in-app notification list on both devices.

**FEAT-09.SPEC-014-AC-07:** Given this notification has no preference toggle, when a request is sent, then it is always delivered regardless of any other notification setting Sam has configured.

**FEAT-09.SPEC-014-AC-08:** Given the notification has been delivered and sits unread for several days with no organiser cancellation, when Sam eventually opens it, then it remains actionable, since this notification does not expire on a timer.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no toggle) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
