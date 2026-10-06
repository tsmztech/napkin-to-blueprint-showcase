---
document_type: feature-overview
feature_number: FEAT-15
feature_name: Member Onboarding
feature_slug: member-onboarding
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 3
screen_count: 1
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Member Onboarding

## Summary

**Feature:** Member Onboarding
**ID:** FEAT-15
**Description:** An adult who accepts a household invitation is guided from acceptance to seeing the current plan and grocery list for the first time.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The brief's growth model depends on invited members having a smooth first experience — "growth is expected to come mostly from households inviting other households" (BRIEF.md, Business Context) is a household-to-household version of the same principle, and a confusing first join would undermine it. Included at MVP alongside Household Invitations (FEAT-09), since an invitation without a guided first-use is only half the feature.

**Key Capabilities:**
- Land in context — Newly joined member is taken directly to the current plan and grocery list, not a generic empty home screen
- Understand their role — Newly joined member sees a brief explanation of what they can do (view the plan, suggest swaps, shop, rate) versus what the organiser manages

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-15.SPEC-001 | Onboarding Landing | Screen | Other Adult Member | Newly accepted member's first-use view: the current plan and grocery list plus a short role-explanation banner, with an explained empty state and an offline-degraded fallback |
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | Automation | Other Adult Member | Fires on FEAT-09's invitation-acceptance event, routes the new member to the Onboarding Landing exactly once, and emits the feature's signals |
| FEAT-15.SPEC-003 | Onboarding Eligibility & Once-Only Rule | Logic/Rule | Other Adult Member | Governs who is ever routed through onboarding (Other Adult Member only) and that a re-invited former member always onboards fresh rather than being restored to prior data |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Land in context | FEAT-15.SPEC-001 | Primary purpose of the Onboarding Landing screen — displays the current Weekly Plan and Grocery List directly, or the explained empty state if no plan exists yet | Phase 2 (Explicit) |
| Understand their role | FEAT-15.SPEC-001 | Role-explanation banner on the same screen, naming Sam's entitlements (view plan, suggest swaps, shop, rate) against the organiser's (setup, budget, schedule, plan approval, billing) per the Access Matrix | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-15.SPEC-002 | Onboarding Trigger & Completion | Phase 4 (Trigger-Response Analysis) | Neither Key Capability describes how onboarding actually starts; the feature's own Interactions field ("Depends on Household Invitations & Membership (FEAT-09) for the acceptance event it follows") and XBR-18 require a standalone trigger-response spec that crosses a feature boundary — landing does not happen on its own |
| FEAT-15.SPEC-003 | Onboarding Eligibility & Once-Only Rule | Phase 5 (Rule-Constraint Discovery), confirmed by Phase 3 (re-join is a lifecycle question) | The Validation & Limits field ("shown exactly once per newly accepted invitation"), the Access field (only Sam's role, never Maya, either kid row, Riley, or an unauthorized visitor), and XBR-18's re-join clause are three conditional rules shared by both SPEC-001 (guarding direct access) and SPEC-002 (deciding whether to fire) — they cross the standalone-spec threshold as rules shared across multiple specs within the feature |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity of its own. Its two Connected Entities are declared read-only in product-features.md ("Member Profile (read — the newly created one), Invitation (read — the accepted one)"), and the dependency map assigns their full lifecycle to other features: Member Profile is created by FEAT-01/FEAT-09 and Invitation is created and updated by FEAT-09. A full CRUD matrix would therefore be entirely N/A by definition; the Referenced Entities table below is the correct and complete coverage for a feature of this shape. The once-only tracking required by SPEC-003 rides on the Invitation's own one-time acceptance event (an Invitation transitions to Accepted exactly once, per FEAT-09) rather than requiring a new persisted flag on Member Profile — this keeps the feature within its declared read-only access to both entities.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invitation | FEAT-15.SPEC-002, FEAT-15.SPEC-003 | The accepted invitation whose acceptance event (owned by FEAT-09) is the sole trigger for onboarding, and whose one-time Accepted transition is what makes onboarding naturally fire once |
| Member Profile | FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-003 | The newly created Other Adult Member profile the Landing screen personalizes for, and whose member_type/status the Automation and Eligibility Rule check to confirm the role and to distinguish a fresh join from a re-join |
| Weekly Plan | FEAT-15.SPEC-001 | Read directly on the Landing screen; owned and written by FEAT-03 (generated) or FEAT-23 (manual) |
| Grocery List | FEAT-15.SPEC-001 | Read directly on the Landing screen; owned and written by FEAT-06 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Invited adult accepts a household invitation (FEAT-09 event) | Route the new member to the Onboarding Landing, applying the eligibility and once-only rule first | Standalone Automation | FEAT-15.SPEC-002 |
| Onboarding Trigger fires for a former member who was previously removed and is now re-invited | Route through the same onboarding flow again rather than restoring prior household data | Standalone Logic/Rule | FEAT-15.SPEC-003 |
| Onboarding Landing loads and the household has no plan yet | Show the explained empty state instead of a blank or broken view | Inline in triggering screen | FEAT-15.SPEC-001 |
| Onboarding Landing loads with no connectivity | Degrade to the standard offline plan/list view rather than a distinct onboarding-specific offline treatment | Inline in triggering screen | FEAT-15.SPEC-001 |
| Onboarding Landing displays the live current Weekly Plan and Grocery List | Real-time synchronization of plan and list content across devices | Cross-feature — owned by the real-time synchronization boundary already specified elsewhere | FEAT-03.SPEC-011 / FEAT-06.SPEC-005 |
| A role other than Other Adult Member (organiser, either kid row, Riley, or an unauthorized visitor) would otherwise reach the Onboarding Landing | Refuse to route or render; the flow is never shown | Standalone Logic/Rule | FEAT-15.SPEC-003 |

No Notification spec is produced: the feature's Communications field is explicit — "N/A — this is an in-app first-use experience, not a separate notification (the invitation itself, sent by FEAT-09, is the communication)." No Integration spec is produced: the External Touchpoints slice names none for this feature and explicitly instructs "Do not invent an external capability"; the one external-shaped dependency the Landing screen relies on (real-time sync) is already owned by FEAT-03 and FEAT-06's Integration/Automation specs, referenced above rather than duplicated.

## Shared Context

**Shared Entities:**
- Invitation — read by SPEC-002 (as the trigger source) and SPEC-003 (to determine the acceptance event and detect a re-join). Fields used: status (must be Accepted), and implicitly the invited contact detail's history for re-join detection via Member Profile status.
- Member Profile — read by all three specs. Fields used: member_type (must be Other Adult Member for eligibility), status (Invited/Active/Left/Removed — Removed-then-reinvited is the re-join case), display_name (personalizes the role-explanation banner).
- Weekly Plan and Grocery List — read only by SPEC-001, exactly as rendered by FEAT-03/FEAT-23 and FEAT-06; SPEC-001 does not reimplement their display logic, it embeds it.

**Shared UI Patterns:**
- Existing plan/list surfaces — SPEC-001 is not a new rendering of the plan and grocery list; it presents the same views FEAT-03/FEAT-23 and FEAT-06 already specify, adding only the role-explanation banner and the first-use empty-state variant unique to onboarding. The Spec Writer for SPEC-001 should describe the banner and empty-state addition, not redescribe the underlying plan/list layouts.

**Shared Validation:**
- FEAT-15.SPEC-003 is the single source for "who is eligible" and "fires once, and a re-join is always fresh." SPEC-001 references it to refuse rendering for an ineligible role or a second showing; SPEC-002 references it to decide whether to fire at all. Neither spec should restate the eligibility or once-only logic independently.

## Internal Dependency Map

```
[FEAT-09 invitation-acceptance event] -> [invited adult accepts] -> SPEC-002 (Onboarding Trigger & Completion)
SPEC-002 (Onboarding Trigger & Completion) -> [checks eligibility and once-only status via] -> SPEC-003 (Onboarding Eligibility & Once-Only Rule)
SPEC-002 (Onboarding Trigger & Completion) -> [eligible and not yet shown] -> SPEC-001 (Onboarding Landing)
SPEC-001 (Onboarding Landing) -> [guards direct or repeat access via] -> SPEC-003 (Onboarding Eligibility & Once-Only Rule)
SPEC-001 (Onboarding Landing) -> [displays] -> FEAT-03 / FEAT-23 (current Weekly Plan)
SPEC-001 (Onboarding Landing) -> [displays] -> FEAT-06 (current Grocery List)
```

**Default Entry:** SPEC-001 (Onboarding Landing) — but only ever reached by way of SPEC-002's trigger; the feature has no independent navigation entry point since onboarding is never navigated to directly, only routed to on acceptance.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-15.SPEC-002 | Inbound | FEAT-09 (Household Invitations & Membership) | Onboarding starts only when an invitation is accepted and a Member Profile with Other Adult Member access is created | Invited adult accepts their invitation (XBR-18) |
| FEAT-15.SPEC-003 | Inbound | FEAT-09 (Household Invitations & Membership) | Re-join detection reads Member Profile status set by FEAT-09 when a member leaves or is removed and later re-invited | A previously removed member accepts a new invitation |
| FEAT-15.SPEC-001 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Landing shows the current week's AI-generated plan | New member lands in context on a paid-tier household with a generated plan |
| FEAT-15.SPEC-001 | Outbound | FEAT-23 (Manual Weekly Planning) | Landing shows the current manually built week | New member lands in context on a free-tier household with a manually built plan |
| FEAT-15.SPEC-001 | Outbound | FEAT-06 (Shared Grocery List) | Landing shows and opens the shared grocery list alongside the plan | New member views or opens the list from the landing view |

## Non-Functional Notes

**Data volumes / growth:** N/A — onboarding is a single, per-member, per-acceptance event rather than a growing data set; it produces no stored records of its own, only signal emissions, so it carries no independent volume or growth profile (product-features.md, Data Notes: "Derived: none. Source: existing household data").

**Responsiveness:** The Onboarding Landing reuses the same plan and grocery list surfaces the household already relies on, so it inherits their responsiveness bar rather than defining a new one: the list must feel instant (ASMP-22), and — per the Grocery List Live-Update Trust success metric's clause that "no household member ever sees a week's plan without its matching list" — the first-time landing must show the plan and list together, not the plan before the list or vice versa.

**Data sensitivity / privacy:** The feature reads existing household personal data (Weekly Plan, Grocery List — private to the household, never sold, ASMP-14/ASMP-26) and an adult Member Profile (personal sign-in and display data protected under account-protection expectations); it never touches kid-profile data, since neither kid row ever goes through this flow (Access field). No new sensitive data is created — only read and briefly displayed.

**Compliance flags:** N/A — no health, financial, or children's-privacy regime applies to this feature specifically; it is scoped to the Other Adult Member role only and never surfaces kid-profile data (ASMP-26 governs Household Setup & Member Profiles, not this feature's read-only display).

**Signals:** member_onboarding_started and member_onboarding_completed are emitted by FEAT-15.SPEC-002 at the start and successful completion of a routed onboarding; member_onboarding_shown_empty_household is emitted by FEAT-15.SPEC-001 when the empty-state variant renders instead of a populated plan and list. All three trace directly to the feature's Signals field in product-features.md.

## Non-Goals

- **Restoring a re-joined member's prior household data automatically** — Excluded per XBR-18 and the feature's Re-join flow line: "a previously removed member who is re-invited goes through the same onboarding again rather than being silently restored to old data." FEAT-15.SPEC-003 makes this an explicit rule rather than a silent omission.
- **A dedicated onboarding notification or email** — Excluded per the feature's Communications field ("N/A — this is an in-app first-use experience, not a separate notification"). The invitation message itself remains FEAT-09's responsibility; onboarding never sends its own message through any channel.
- **Onboarding for the organiser, either kid row, or Riley (Operator, support)** — Excluded per the Access field: the organiser creates the household and is never shown this flow, young kid profiles never accept invitations, a Later-phase older-kid login gets its own introduction under FEAT-17 (not this feature), and Riley's Household Invitations access is None. Grounded in the Access Matrix (user-persona.md).
- **A distinct onboarding-specific offline experience** — Excluded by the feature's own States field: "the landing view degrades to the standard offline plan/list view if there is no connectivity at the moment of acceptance." Onboarding intentionally reuses FEAT-03/FEAT-06's existing offline behavior rather than building a parallel one.
- **Multiple households or a household-selection step during onboarding** — Excluded per scope-boundaries.md (SC-03): "there is one household per account in v1"; a newly joined member never has more than one household to be routed into, so no selection step is needed.
