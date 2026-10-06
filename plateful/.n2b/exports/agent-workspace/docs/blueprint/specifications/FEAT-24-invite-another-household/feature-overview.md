---
document_type: feature-overview
feature_number: FEAT-24
feature_name: Invite Another Household
feature_slug: invite-another-household
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 7
screen_count: 2
automation_count: 3
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

# Feature Breakdown Brief: Invite Another Household

## Summary

**Feature:** Invite Another Household
**ID:** FEAT-24
**Description:** Any adult in a household can share Plateful with another family through a personal invite link. When that family sets up its own household from the link, Plateful records which household invited them.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Business Context expects growth "mostly from households inviting other households," and BRIEF.md, Success Criteria states "most paying households were invited by another household." Household Invitations & Membership (FEAT-09) adds people to one household; nothing in the draft let one household bring in another or recorded where a new household came from, so the brief's growth criterion could be neither supported nor measured. Important rather than Core because the plan-and-list loop works without it; MVP because the first users are parents from the founder's kids' school and online parenting groups, where word of mouth starts on day one. No rewards or credits are attached — the brief names none.

**Key Capabilities:**
- Share an invite link — An adult member shares a personal link through any messaging they already use
- Start a household from a link — A new family following the link begins its own household setup, with the referral recorded
- See who joined — The inviting member sees how many families have set up a household from their link

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-24.SPEC-001 | Invite Another Household Screen | Screen | Maya, Sam | Adult member sees their share link, the number of families who joined, and shares the link through their own messaging |
| FEAT-24.SPEC-002 | Referral Welcome Screen | Screen | Unauthorized Visitor | A visitor following the link sees a welcome page naming the inviter and starts their own household setup, or is told they already have one |
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | Automation | Maya, Sam | Creates and persists an adult member's one reusable personal link the first time it is needed |
| FEAT-24.SPEC-004 | Household Referral Recording | Automation | Maya, Sam | Attributes a completed new-household setup to the inviting household's link, enforcing the one-referral and no-self-referral rules |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | Automation | Maya, Sam | Updates a recorded referral's upgraded flag when the referred household's subscription becomes paid |
| FEAT-24.SPEC-006 | Household Referral Rules | Logic/Rule | All | Governs the one-link-per-member limit, single-attribution and no-self-referral rules, the 30-day counting window, the already-has-a-household disposition, and who may see or act on referrals |
| FEAT-24.SPEC-007 | Referral Joined Notification | Notification | Maya, Sam | Tells the inviting member when a family they invited finishes setting up its household |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Share an invite link | FEAT-24.SPEC-001, FEAT-24.SPEC-003 | Screen shows and shares the link; provisioning automation creates the one reusable link per adult member | Phase 2 (Explicit) |
| Start a household from a link | FEAT-24.SPEC-002, FEAT-24.SPEC-004, FEAT-24.SPEC-006 | Welcome screen hands the visitor off into FEAT-01 setup; recording automation attributes the referral once setup completes; rules govern the 30-day window, self-referral block, and already-has-household disposition | Phase 2 (Explicit) |
| See who joined | FEAT-24.SPEC-001, FEAT-24.SPEC-005, FEAT-24.SPEC-007 | Screen displays the count of joined families; upgrade tracking derives whether they went on to pay; notification tells the inviter the moment one finishes setup | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a single Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-24.SPEC-003 | Personal Referral Link Provisioning | Phase 4 (Trigger-Response — default data generation) | Validation & Limits states "one reusable personal link per adult member"; nothing in the explicit capabilities names the process that creates and persists that link the first time a member needs it |
| FEAT-24.SPEC-004 | Household Referral Recording | Phase 4 (Trigger-Response, cross-entity and cross-feature effect) | A new household's setup completing in FEAT-01 must attribute a referral against the inviting household (XBR-20) — a cross-feature effect with enough processing logic (attribution, self-referral and multi-attribution checks) to need its own Automation rather than living inline on either screen |
| FEAT-24.SPEC-005 | Referral Upgrade Tracking | Phase 3 (Entity-Lifecycle Analysis — Update) | The Household Referral CRUD matrix's Update operation was empty until the `upgraded` field's source — the referred household's Subscription, per Data Notes and the dependency map's Household Referral lifecycle — was traced to a spec |
| FEAT-24.SPEC-006 | Household Referral Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field's three interacting conditions (one link per member, single referring household, no self-referral) plus the 30-day counting window (XBR-20) and the Access field's role split (Maya/Sam Full, both Jordan rows and Riley None, unauthorized visitor link-only) cross the standalone-spec threshold |
| FEAT-24.SPEC-007 | Referral Joined Notification | Phase 4 (Notification surfacing) | The Communications field states "the inviting member gets an in-app note when a family they invited finishes setting up" — a message with a defined audience, content, and trigger, not a same-screen toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Household Referral**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-24.SPEC-004 | System writes the record once a new household's setup completes within 30 days of opening the link | Governed by FEAT-24.SPEC-006's attribution rules |
| Read (single) | N/A | No screen surfaces an individual referral record; only the aggregate count and derived upgrade share are shown, per the Data Notes field — an explicit non-goal, not an omission | -- |
| Read (list) | FEAT-24.SPEC-001, FEAT-24.SPEC-005 | Invite screen reads the count of referrals for display; upgrade tracking reads records to evaluate against Subscription changes | -- |
| Update | FEAT-24.SPEC-005 | Sets the `upgraded` flag when the referred household's Subscription becomes paid | The only field this feature ever updates on the record |
| Delete/Archive | N/A | The record is written once and never edited by any role beyond the `upgraded` flag (dependency map, Household Referral Contention) and is kept as the product's only record of household-to-household growth for success measurement (dependency map, Household Referral Data Sensitivity) — no deletion or purge mechanism is defined, recorded as an explicit non-goal. Cascade behavior when a referenced household is deleted is owned by FEAT-18 (Account & Data Management), not this feature — flagged here, not resolved, per this feature's decision authority | -- |
| State Transition | N/A | The record carries no lifecycle states beyond its one field update (`upgraded` false → true); it is never itself transitioned through statuses | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-24.SPEC-001, FEAT-24.SPEC-002, FEAT-24.SPEC-004, FEAT-24.SPEC-006 | Screens and rules read the inviting household's identity and member/link ownership, and check whether a visitor already belongs to a household |
| Subscription | FEAT-24.SPEC-005 | Upgrade tracking reads the referred household's subscription tier to determine whether it went on to pay |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Adult member opens the Invite Another Household screen and no personal link exists yet | System provisions and persists the member's one reusable personal link | Standalone Automation | FEAT-24.SPEC-003 |
| Member taps share or copy on an existing link | Link is shared through the member's own messaging, outside the product | Inline in triggering screen | FEAT-24.SPEC-001 |
| Link cannot be created | Retry is offered | Inline in triggering screen | FEAT-24.SPEC-001 |
| Visitor opens the link | Welcome page shown with the inviter's first name and a start-setup option | Inline in triggering screen | FEAT-24.SPEC-002 |
| Visitor already belongs to a household | Visitor is told they already have one; nothing is recorded | Standalone Logic/Rule | FEAT-24.SPEC-006 |
| A new household completes setup from a link within 30 days | Referral is recorded against the inviting household; self-referral and multi-attribution are blocked | Standalone Automation | FEAT-24.SPEC-004 |
| A new household completes setup from a link after 30 days | No referral is recorded | Standalone Logic/Rule | FEAT-24.SPEC-006 |
| A referral is recorded | Inviting member is sent an in-app note once the referred family finishes setting up | Standalone Notification | FEAT-24.SPEC-007 |
| Referred household's subscription changes to paid | Household Referral's `upgraded` flag is updated | Standalone Automation | FEAT-24.SPEC-005 |
| Referring or referred household is deleted (FEAT-18) | Cascade behavior for its Household Referral records | Cross-feature — logged in touchpoints | FEAT-18 responsibility |

## Shared Context

**Shared Entities:**
- Household Referral — created by SPEC-004 under SPEC-006's attribution rules, updated by SPEC-005, read in aggregate by SPEC-001 and SPEC-005. Fields: referring_household, referring_member_link, new_household, created_date, upgraded.

**Shared UI Patterns:**
- Public link-landing pattern — SPEC-002's welcome page follows the same "no household data exposed to an unauthorized visitor" pattern the Access Matrix notes apply across the product (e.g., FEAT-09's invitation-acceptance screen); Spec Writers should describe the visitor experience consistently with that constraint.

**Shared Validation:**
- SPEC-006 defines the one-link-per-member limit, single-attribution and no-self-referral rules, the 30-day window, and role access. SPEC-001, SPEC-002, and SPEC-004 all reference SPEC-006 rather than duplicating these rules.

## Internal Dependency Map

```
SPEC-001 (Invite Another Household Screen) -> [member opens screen, no link yet] -> SPEC-003 (Personal Referral Link Provisioning)
SPEC-001 (Invite Another Household Screen) -> [member taps share] -> [member's own messaging, outside the product]
SPEC-001 (Invite Another Household Screen) -> [validated by] -> SPEC-006 (Household Referral Rules)
SPEC-002 (Referral Welcome Screen) -> [visitor taps "start setup"] -> FEAT-01 (Household Setup & Member Profiles)
SPEC-002 (Referral Welcome Screen) -> [governed by] -> SPEC-006 (Household Referral Rules)
FEAT-01 (Household Setup & Member Profiles) -> [new household's setup completes] -> SPEC-004 (Household Referral Recording)
SPEC-004 (Household Referral Recording) -> [validated by] -> SPEC-006 (Household Referral Rules)
SPEC-004 (Household Referral Recording) -> [record created] -> SPEC-001 (Invite Another Household Screen) [count updates]
SPEC-004 (Household Referral Recording) -> [record created] -> SPEC-007 (Referral Joined Notification)
FEAT-14 (Subscription & Billing Management) -> [referred household's subscription becomes paid] -> SPEC-005 (Referral Upgrade Tracking)
SPEC-005 (Referral Upgrade Tracking) -> [updates] -> Household Referral record (read by SPEC-001, and by success-metrics.md's growth measurement)
```

**Default Entry:** SPEC-001 (Invite Another Household Screen) for a signed-in adult household member; SPEC-002 (Referral Welcome Screen) for an unauthorized visitor arriving from a shared link.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-24.SPEC-001 | Inbound | FEAT-25 (Weekly Waste & Spend Check-In) | Entry into the Invite Another Household screen from the check-in trend card | User taps "invite another family" on the check-in card (End-of-Week Check-In & Inviting Another Family, step 3) |
| FEAT-24.SPEC-002 | Outbound | FEAT-01 (Household Setup & Member Profiles) | Visitor begins their own new household's setup from the welcome page | Visitor taps the start-setup option on the Referral Welcome Screen |
| FEAT-24.SPEC-004 | Inbound | FEAT-01 (Household Setup & Member Profiles) | A new household's completed setup is the trigger that creates the Household Referral record | New household's setup completes within 30 days of the link being opened |
| FEAT-24.SPEC-005 | Inbound | FEAT-14 (Subscription & Billing Management) | A referred household's subscription tier change is read to update the referral's upgraded flag | Referred household's Subscription transitions to a paid tier |
| Household Referral (Delete/Archive) | Inbound | FEAT-18 (Account & Data Management) | Cascade behavior for Household Referral records when a referencing household is deleted is owned by FEAT-18, not this feature | Household deletion completes |

## Non-Functional Notes

**Data volumes / growth:** Household Referral records grow at roughly the same order of magnitude as new households — several thousand households in the first year (assumptions-constraints.md ASMP-24, scope-boundaries.md SC-15) — with each household producing at most one inbound referral record and one outbound personal link per adult member; volumes stay small relative to the product's other entities.

**Responsiveness:** The personal link is ready instantly once created (feature's States field); the Invite Another Household screen's count and referral-joined note need no faster response, since neither is a real-time collaborative surface like the grocery list (assumptions-constraints.md ASMP-22 does not apply here).

**Data sensitivity / privacy:** Low — the Household Referral record links two households by identity and date only; an unauthorized visitor sees only the inviting member's first name, never any other household data (feature's Access field; dependency map, Household Referral Data Sensitivity). The record is used only for the product's own growth measurement and is never sold or used for advertising (assumptions-constraints.md ASMP-14, ASMP-26).

**Compliance flags:** N/A — the Household Referral record holds no children's data and no payment detail (that sits with the Subscription entity FEAT-14 owns); general personal-data rights (export, deletion) apply to the household records it links, not to this feature's own record content, per assumptions-constraints.md ASMP-27.

## Non-Goals

- **Rewards, credits, or discounts for inviting other households** — Excluded per scope-boundaries.md (SC-10): the brief expects households to invite households but names no incentive; this feature records referrals without paying for them, keeping money flows limited to the household subscription.
- **Multiple households per account / re-attribution** — Excluded per scope-boundaries.md (SC-03) and XBR-20: there is one household per account in v1, so a visitor who already has a household is told so and nothing is recorded; a new household can never be attributed to more than one referring household.
- **Cross-household social or community features (public profiles, recipe-sharing feeds)** — Excluded per scope-boundaries.md (SC-09): growth runs through direct household-to-household invite links, not a social layer; the Referral Welcome Screen shows only the inviter's first name and a start-setup option, never a profile or feed.
- **Individual referral record detail view** — Intentional design decision surfaced by the CRUD matrix: the feature's Data Notes name only the aggregate count and the derived paying share as displayed; no capability or journey step asks the inviting member to inspect a single referral's detail, so no Read (single) spec exists.
- **Deletion or purge of Household Referral records** — Intentional lifecycle decision surfaced by the CRUD matrix: the record is written once and never edited beyond its upgrade flag (dependency map, Household Referral Contention) and is retained as the product's only record of referral-driven growth (dependency map, Household Referral Data Sensitivity); no retention window applies. Cascade behavior on deletion of a household it references belongs to FEAT-18, not this feature.
