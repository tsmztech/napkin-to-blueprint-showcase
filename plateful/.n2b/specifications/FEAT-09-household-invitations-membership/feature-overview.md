---
document_type: feature-overview
feature_number: FEAT-09
feature_name: Household Invitations & Membership
feature_slug: household-invitations-membership
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 14
screen_count: 5
automation_count: 4
logic_rule_count: 2
integration_count: 0
notification_count: 3
---

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
