---
document_type: spec
spec_type: screen
spec_id: FEAT-16.SPEC-001
spec_name: Storage Usage Summary
spec_slug: storage-usage-summary
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Storage Usage Summary

## Overview

**Name:** Storage Usage Summary
**ID:** FEAT-16.SPEC-001
**Type:** Screen
**Purpose:** Nadia sees her total stored bytes against her freelancer storage allowance, with warning styling as she approaches the limit and a path into her plan view; Dana sees the same figures read-only during a logged support session.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Displaying the freelancer's current total stored bytes against her storage allowance
- Warning styling when usage approaches or reaches the allowance
- A link into the plan view when usage is near or at the limit
- Dana's read-only view of the same figures during a logged support session

**Non-Goals:**
- Calculating or recalculating the total stored-bytes figure -- owned by FEAT-16.SPEC-005 (Storage Usage Aggregation); this screen only displays the most recently aggregated total
- Determining the allowance amount, the warning threshold, or enforcing any limit -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this screen only displays the values that spec defines
- Changing or purchasing a plan -- owned by FEAT-23 (Subscription Plan & Billing Management); this screen only links into FEAT-23.SPEC-001 (Plan & Billing Screen)
- Listing or previewing individual stored files -- excluded per this feature's own Summary: "its own user-facing surface is limited to the storage-usage visibility the Validation & Limits field requires"; per-deliverable browsing belongs to FEAT-06.SPEC-002 (Deliverable List & Management) and FEAT-17 (Deliverable Version History)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia opens "Storage" from her account settings | None -- screen loads and fetches the current total and allowance itself |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Nadia opens the storage detail from a usage indicator on her plan view | None -- screen loads independently |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana, inside an open support session on a named freelancer's account, opens that freelancer's Storage screen | The freelancer account the session is scoped to (FEAT-31.SPEC-003); screen renders that account's figures read-only |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen: total stored bytes, allowance, warning styling, link into the plan view | Navigate to FEAT-23.SPEC-001 (Plan & Billing Screen) | -- |
| Owen (Client Primary Contact) | No | No | Storage usage is not part of any client-facing surface; Owen's Subscription & Account Data entitlement is None (Access Matrix) -- no navigation path in the client portal ever reaches this screen |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya's Subscription & Account Data entitlement is None |
| Dana (Support Operator) | Full screen, read-only: total stored bytes, allowance, warning styling -- only while an active support session on that freelancer's account is open (FEAT-31.SPEC-003) | No actions -- the "Review your plan" link is not shown to Dana, since changing a plan is not part of a read-only session | Outside an active support session this screen is unreachable; there is no separate denied experience, since no navigation path to it exists without one |
| Unauthenticated | No | No | Redirected to sign-in; entered no data on this read-only screen to lose |
| Expired session | No | No | "Your session has expired. Sign in to continue." -- no in-progress data to preserve, since this screen accepts no input |

## Layout and Content

**Header:** Screen title "Storage" within the account settings / plan area, with a back control returning to the entry screen (Account Profile or Plan & Billing).

**Body:** A single stat panel, vertically centered in the content area:
- A storage meter (horizontal progress-style indicator) showing bytes used as a proportion of the allowance
- A numeric label beneath the meter: "{used amount} of {allowance amount} used" (e.g., "6.2 GB of 10 GB used")
- A warning banner, shown only in the Near Limit or At Capacity states, with the message defined by FEAT-16.SPEC-004's threshold rule and a "Review your plan" link (Nadia only -- not shown to Dana)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Stat panel stacks vertically, full width; meter spans the full content width; warning banner, when present, appears directly below the meter, full width.
- **Medium size class and above:** Stat panel is capped at a consistent platform-wide card width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Storage meter | Screen load | Fetches the most recently aggregated total from FEAT-16.SPEC-005 and the current allowance and threshold from FEAT-16.SPEC-004 | Meter renders at the loaded proportion | Visual meter fill plus the numeric label |
| "Review your plan" link | Tap (Nadia only, Near Limit or At Capacity state) | Navigate to FEAT-23.SPEC-001 (Plan & Billing Screen) | Screen closes | Standard navigation transition |
| Screen re-opened | Nadia or Dana returns to this screen after navigating away | Re-fetches the latest aggregated total and allowance | Meter and label update to the current values | Meter reflects the freshest known total; the screen does not live-update while left open in the background |

### Accessibility Notes

- **Focus order:** Back control -> storage meter (announced with its numeric percentage) -> warning banner and "Review your plan" link (when present).
- **Dynamic announcements:** When the warning banner becomes visible on load, its message is announced to assistive technology.
- **Keyboard alternatives:** The meter is a display-only element; "Review your plan" and the back control are both reachable and actionable by keyboard, with no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton meter and label while the current total and allowance load | Screen first opens | Load completes (success or error) |
| Normal | Meter shows usage below the warning threshold; no warning banner | Load completes with usage below platform parameter: `storage-usage-warning-threshold-percent` of the allowance | Usage crosses the warning threshold on a later reload |
| Near Limit | Meter and label show the current usage; warning banner: "You're approaching your storage allowance. {used amount} of {allowance amount} used." with "Review your plan" (Nadia) | Load completes with usage at or above platform parameter: `storage-usage-warning-threshold-percent` but below the full allowance | Usage drops back below the threshold, or reaches full allowance, on a later reload |
| At Capacity | Meter shows a full or near-full fill; warning banner: "You've reached your storage allowance. New uploads won't complete until you free up space or upgrade your plan." with "Review your plan" (Nadia) | Load completes with usage at or effectively at the full allowance | Usage drops below the full allowance on a later reload (e.g., after a removal or purge) |
| Error | Error banner: "Couldn't load your storage usage. Try again." with a Retry control | The total or allowance fails to load | Retry succeeds and the screen re-enters Loading, then Normal/Near Limit/At Capacity |
| Offline/Degraded | Banner: "You're offline -- showing your last known storage usage." above the meter; the last successfully loaded figures remain displayed; "Review your plan" is disabled while offline | Connectivity is lost while this screen is open, or the screen is opened without connectivity and a cached figure exists | Connectivity returns and the screen re-fetches automatically |

## Validation Rules

Not applicable -- this screen accepts no user input; every displayed value is read-only, sourced from FEAT-16.SPEC-004 and FEAT-16.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | FEAT-21.SPEC-001 (Account Profile) or FEAT-23.SPEC-001 (Plan & Billing Screen) -- whichever this screen was entered from | FEAT-21 or FEAT-23 |
| "Review your plan" tap (Nadia, Near Limit or At Capacity) | FEAT-23.SPEC-001 (Plan & Billing Screen) | FEAT-23 |

## Data Model

**Creates:** None.
**Reads:** The freelancer's current aggregated total stored bytes (maintained by FEAT-16.SPEC-005, derived from every non-purged Deliverable Version's stored file size). Subscription Plan -- tier, used by FEAT-16.SPEC-004 to resolve the applicable allowance and warning threshold.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The allowance amount, the warning threshold, and the At Capacity condition are all defined by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) -- this screen displays those values without redefining them.
- The displayed total is FEAT-16.SPEC-005's most recently completed aggregation; a transfer still in progress is not reflected until it completes and the aggregation recalculates (XBR-14).
- XBR-29: Dana's read-only support session never offers a file-download or plan-change action on this screen.

## Edge Cases

- **Total shown while an upload is mid-transfer** -- The meter reflects only FEAT-16.SPEC-005's last completed aggregation, not bytes from an in-progress transfer; the figure can briefly lag behind a transfer that has not yet finished. This is a display characteristic, not a rule violation, since FEAT-16.SPEC-004 always re-checks the live total at the moment any transfer is actually evaluated against the allowance.
- **Nadia's plan changes while this screen is open in a background tab** -- The allowance shown does not update until the screen is re-opened or reloaded; a stale allowance figure is a display lag only, since FEAT-16.SPEC-004 always re-reads the current plan at the moment of real enforcement.
- **Dana opens this screen for a freelancer account with zero stored bytes** -- The meter shows 0 of {allowance}, in the Normal state, with no warning banner.
- **Usage figure is exactly at the warning threshold** -- The screen enters the Near Limit state; the boundary is inclusive of the threshold value.

This screen updates no shared entity, so no concurrent-edit conflict entry applies -- every value here is read-only and sourced from FEAT-16.SPEC-004 and FEAT-16.SPEC-005.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) | References (inbound) | Supplies the allowance amount, the warning threshold, and the At Capacity condition this screen displays |
| FEAT-16.SPEC-005 (Storage Usage Aggregation) | References (inbound) | Supplies the current aggregated total stored bytes |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Navigation (outbound) | "Review your plan" link when Nadia is near or at her allowance |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Settings entry point into this screen |
| FEAT-31.SPEC-002 (Operator Support Session Console) / FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | Gates Dana's read-only, session-scoped access to this screen (XBR-29) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| storage_limit_warning_shown | threshold_crossed (near_limit / at_capacity), percent_used | Screen enters the Near Limit or At Capacity state on load | N/A -- no Stage 2 metric measures storage-warning visibility directly; retained so how often freelancers approach their allowance is observable |
| storage_usage_summary_viewed | percent_used bucket, viewer_role (Nadia / Dana) | Screen finishes loading successfully | N/A -- no Stage 2 metric measures views of this screen; retained to observe usage-checking frequency ahead of any future metric needing it |

## Acceptance Criteria

**FEAT-16.SPEC-001-AC-01:** Given Nadia opens the Storage screen from Account Profile, when the figures load, then the meter shows her current total stored bytes against her allowance with the numeric label "{used} of {allowance} used".

**FEAT-16.SPEC-001-AC-02:** Given Nadia's usage is below platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when the screen loads, then no warning banner is shown.

**FEAT-16.SPEC-001-AC-03:** Given Nadia's usage is at or above platform parameter: `storage-usage-warning-threshold-percent` but below her full allowance, when the screen loads, then the Near Limit warning banner appears with the "Review your plan" link.

**FEAT-16.SPEC-001-AC-04:** Given Nadia taps "Review your plan" while the Near Limit banner is shown, when the tap registers, then she is navigated to FEAT-23.SPEC-001 (Plan & Billing Screen).

**FEAT-16.SPEC-001-AC-05:** Given Nadia's usage reaches her full allowance, when the screen loads, then the At Capacity banner appears stating that new uploads won't complete until space is freed or the plan is upgraded.

**FEAT-16.SPEC-001-AC-06:** Given the storage figures fail to load, when the load fails, then the error banner "Couldn't load your storage usage. Try again." appears with a Retry control.

**FEAT-16.SPEC-001-AC-07:** Given Nadia loses connectivity while this screen is open, when connectivity drops, then the banner "You're offline -- showing your last known storage usage." appears, the last-loaded figures remain visible, and "Review your plan" is disabled.

**FEAT-16.SPEC-001-AC-08:** Given Dana opens a support session on a freelancer's account and navigates to Storage, when the screen loads, then she sees the same total and allowance figures read-only, with no "Review your plan" link shown.

**FEAT-16.SPEC-001-AC-09:** Given Owen or Priya is signed in to the client portal, when they look for any path to a storage screen, then none exists -- this screen is never reachable from the client-facing portal.

**FEAT-16.SPEC-001-AC-10:** Given an unauthenticated visitor requests this screen's address directly, when the request is made, then they are redirected to sign-in.

**FEAT-16.SPEC-001-AC-11:** Given Nadia's usage is exactly at platform parameter: `storage-usage-warning-threshold-percent`, when the screen loads, then it enters the Near Limit state (the boundary is inclusive).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (Loading, Near Limit, At Capacity, Error, Offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
