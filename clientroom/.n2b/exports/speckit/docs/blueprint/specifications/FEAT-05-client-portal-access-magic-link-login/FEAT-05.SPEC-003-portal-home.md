---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-003
spec_name: Portal Home
spec_slug: portal-home
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Portal Home

## Overview

**Name:** Portal Home
**ID:** FEAT-05.SPEC-003
**Type:** Screen
**Purpose:** The contact sees their company's projects, each project's current stage and milestones, and exactly what is waiting on them, scoped to what their role allows.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- Listing the contact's client company's projects with each project's derived stage
- Showing each project's milestones and what is currently waiting on the contact (accept, review, approve, pay), limited to what the contact's role allows
- The empty state for a contact with no active projects
- Navigation into the waiting item's own feature (accept a proposal, review a deliverable, approve a milestone, pay an invoice, or invite a Reviewer colleague)

**Non-Goals:**
- Accepting proposals, reviewing deliverables, approving milestones, or paying invoices -- each is owned by its own feature (FEAT-03, FEAT-07, FEAT-08, FEAT-10); this screen only surfaces that something is waiting and links to where the action happens.
- Determining which role sees which action -- owned by FEAT-05.SPEC-007 (Portal Access & Isolation Rules), the single source of truth for role-scoped display; this screen enforces its output rather than defining it.
- Managing client contacts or roles -- excluded from this screen's scope per the Entity-Lifecycle Coverage Matrix: contact creation, role assignment, and invitation belong to FEAT-18, even though Owen can launch an invite from this screen.
- A general activity feed or notification center -- excluded per scope-boundaries.md's Feature Scope Exclusions and the Later-phase status of In-App Notification Center (FEAT-29); this screen shows current project state, not a historical feed.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-002 (Link Verification Landing) | Verification succeeds | The verified Client Contact's identity, scoped freelancer, and client company |
| FEAT-33 (Portal Referral Attribution) | A contact with a live session returns via the public product page | Existing session -- no new link request |
| Any portal page (session lapse recovery) | N/A -- a lapsed session returns the contact to FEAT-05.SPEC-001, not here | N/A |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full project list for his own client company: stage, milestones, and everything waiting on him | Tap through to accept, approve, pay, or review; invite a Reviewer colleague | -- |
| Priya (Client Reviewer Contact) | Full project list for her own client company: stage and milestones, and review items waiting on her | Tap through to review and comment only; no Approve, Accept, or Pay control is shown | If Priya reaches a waiting item that is Owen-only (a proposal or invoice), no such item ever appears in her waiting list, so there is no disabled control to encounter (FEAT-05.SPEC-007) |
| Nadia (Freelancer) | None -- this is a client-facing screen; Nadia never signs in as a client contact | None | Nadia has no route to this screen at all; she has her own freelancer-side project view (FEAT-01) |
| Dana (Support Operator) | None -- Dana never signs in as a client contact and has no portal access (SC-04, Access Matrix: Client Portal Access -- None) | None | Dana has no route to this screen; her read-only support session (FEAT-31) views a freelancer's account, never a client's portal |
| Unauthenticated | No | No | Redirected to FEAT-05.SPEC-001 (Request Sign-In Link) |
| Expired session | No | No | Redirected to FEAT-05.SPEC-001; in-progress reading state on this screen is simply lost, since Portal Home holds no user-entered data to preserve |

## Layout and Content

**Header:** The owning freelancer's Branding Profile logo (or neutral default), left-aligned, per XBR-31. For Owen only, an "Invite a colleague" action (right-aligned) that opens FEAT-18's invite flow.

**Body:** A vertical list of project cards, one per project in the contact's client company. Each card shows:
- Project name
- Stage label (Draft, In Progress, Complete, Cancelled, Archived -- the Project entity's derived `stage` field)
- A "Waiting on you" region, present only when at least one item needs the contact's attention, listing each waiting item (a proposal to accept, a deliverable to review, a milestone to approve, or an invoice to pay) with its type and a tap target
- Milestone summary: a compact list of the project's milestones with each one's status (Defined, Deliverable Uploaded, Approved, Reopened)

Cards are ordered with the most recent "Waiting on you" activity first, then by project recency.

**Footer:** The discreet "Made with Clientroom" referral mark (FEAT-33).

### Responsive Behavior

- **Compact breakpoint (phone width):** Project cards stack in a single column, full width; the milestone summary within a card collapses to a scrollable horizontal strip of milestone chips; the "Invite a colleague" header action collapses to an icon-only button for Owen.
- **Medium size class and above:** Project cards remain single-column but cap at a consistent platform-wide content width and center horizontally; the milestone summary within a card shows as a wrapped list instead of a horizontal strip.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Invite a colleague" (Owen only) | Tap | Navigates to FEAT-18's invite-a-Reviewer flow | Screen navigates away | Standard navigation transition |
| Project card | Tap | Expands or navigates to that project's underlying detail within whichever waiting-action feature applies (see Navigation Out) | Depends on destination | Standard navigation transition |
| "Waiting on you" item | Tap | Navigates directly to the specific action screen (accept, review, approve, or pay) for that item | Screen navigates away | Standard navigation transition |
| Milestone chip | Tap | Navigates to FEAT-07 (Deliverable Review & Feedback) for that milestone's deliverables | Screen navigates away | Standard navigation transition |
| "Made with Clientroom" referral mark | Tap | Navigates externally to the public product page (FEAT-33); does not affect this screen's own state | This screen's state is unchanged | Standard external navigation |

### Accessibility Notes

- **Focus order:** Header logo (skippable) -> "Invite a colleague" (Owen only) -> project cards in list order, each card's "Waiting on you" items before its milestone summary.
- **Dynamic content announcement:** When Portal Home loads, the presence of any "Waiting on you" items is announced as a summary (e.g., "2 items waiting on you") so a screen-reader user does not have to traverse every card to learn whether action is needed.
- **Keyboard alternatives:** Every card, waiting item, and milestone chip is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loaded (default) | Project list as described above | Screen loads with at least one project | User navigates away |
| Loading | Skeleton placeholders in place of project cards | Screen first opens, before project data returns | Data load completes |
| Empty | Plain message "Nothing here yet -- {freelancer name} hasn't sent you anything to review." (no error styling) | Contact has no active projects (before any proposal has been sent) | A project reaches a state the contact can see |
| Error | Error banner "We couldn't load your projects. Try again." with a Retry button | Project data fails to load | User taps Retry and load succeeds |
| Offline/Degraded | Banner "You're offline -- showing the last loaded view." above the project list; the list shown is the last successfully loaded snapshot, read-only (tapping a waiting item that requires a live connection shows the same offline notice rather than navigating) | Connectivity lost while viewing a previously loaded Portal Home | Connectivity restored -- banner clears and the screen refreshes automatically |

## Validation Rules

Validation governed by FEAT-05.SPEC-007 (Portal Access & Isolation Rules) for what a given role's session may display. This screen has no user-entered fields, so no field-level validation applies.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Owen taps a waiting proposal | FEAT-03 (Proposal Acceptance) | FEAT-03 |
| Owen or Priya taps a waiting deliverable | FEAT-07 (Deliverable Review & Feedback) | FEAT-07 |
| Owen taps a waiting milestone | FEAT-08 (Milestone Approval) | FEAT-08 |
| Owen taps a waiting invoice | FEAT-10 (Invoice Payment Processing) | FEAT-10 |
| Owen taps "Invite a colleague" | FEAT-18 (Client Contact Management & Roles) | FEAT-18 |

## Data Model

**Creates:** None.
**Reads:** Project -- `project_name`, `stage`, `currency` (for display formatting) for every project belonging to the contact's client company; Milestone -- `name`, `order`, `status` for each project's milestones; Branding Profile -- `logo`, `brand_colour`. Proposal, Invoice, and Milestone waiting-state fields are read indirectly through each owning feature's own "waiting on" determination (Proposal `status`, Invoice `status`, Milestone `status`) to compose the "Waiting on you" region.
**Updates:** None -- this screen is read-only.
**Deletes:** None.

## Business Rules

- XBR-08 and FEAT-05.SPEC-007: Priya's "Waiting on you" region never lists a proposal or invoice item, since Reviewer contacts cannot accept, approve, or pay; only Owen's region can include those item types.
- XBR-09 and FEAT-05.SPEC-007: the project list shown is scoped to exactly the contact's own client company under exactly the freelancer whose portal they signed into; a contact who is also a contact for a different freelancer sees that freelancer's projects only after a separate sign-in to that freelancer's portal (FEAT-05.SPEC-007).
- A project's `stage` label shown here is the same derived value defined in the dependency map's Project entity -- this screen never computes its own stage logic.

## Edge Cases

- **Contact has projects but none currently have anything waiting on them** -- Each project card shows its stage and milestones with no "Waiting on you" region; the screen is not treated as empty, since projects exist.
- **A waiting item is resolved by someone else while Portal Home is open (e.g., Owen approves a milestone Priya was also viewing)** -- Portal Home is a snapshot, not live-updating; the resolved item still shows as waiting until the contact reloads or navigates back to this screen, at which point the refreshed data no longer lists it. No error occurs if the contact taps the now-resolved item -- the destination screen (e.g., FEAT-08) shows its own current state.
- **Project count is large enough to require scrolling** -- The list scrolls normally; no pagination or truncation is applied, since the dependency map's Non-Functional Notes bound this feature's data volume to a few thousand freelancers with a handful of client contacts each, well within a single scrollable list.
- **Contact reaches Portal Home for a client company that was archived after the link was issued but before verification completed** -- Treated the same as "no active projects": the Empty state's message is shown rather than an error, since archiving does not delete the underlying record.
- **This screen reads shared entities (Project, Milestone) but never writes them, so no concurrent-edit conflict applies here** -- N/A: Portal Home is read-only; a change made elsewhere (by Nadia, or by the contact's own action on another screen) is reflected only on the next load, per the snapshot behavior above, not as an in-place conflict.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (inbound) | Successful verification lands the contact here |
| FEAT-05.SPEC-007 (Portal Access & Isolation Rules) | References (inbound) | Governs role-scoped display and client isolation for everything shown here |
| FEAT-03 (Proposal Acceptance) | Navigation (outbound) | Waiting-proposal item deep-links here |
| FEAT-07 (Deliverable Review & Feedback) | Navigation (outbound) | Waiting-deliverable item and milestone chips deep-link here |
| FEAT-08 (Milestone Approval) | Navigation (outbound) | Waiting-milestone item deep-links here |
| FEAT-10 (Invoice Payment Processing) | Navigation (outbound) | Waiting-invoice item deep-links here |
| FEAT-18 (Client Contact Management & Roles) | Navigation (outbound) | Owen's "Invite a colleague" action deep-links here |
| FEAT-05.SPEC-009 (Portal Record First-View Capture) | Triggers (outbound) | Navigating from a waiting item into a proposal, deliverable, or invoice screen is the moment SPEC-009 detects a first view |
| FEAT-33 (Portal Referral Attribution) | Navigation (outbound) | The footer referral mark links out to the public product page |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|------------------|
| portal_home_viewed | role (Owen / Priya), project_count, waiting_item_count, load_time_ms | Portal Home successfully loads | supports success-metrics.md: "Client Portal Mobile Responsiveness" |
| portal_home_waiting_item_opened | item_type (proposal / deliverable / milestone / invoice), role | Contact taps a "Waiting on you" item | supports success-metrics.md: "Client Portal Login Success" (a completed sign-in that leads into the waiting action is the flow this metric ultimately protects) |

## Acceptance Criteria

**FEAT-05.SPEC-003-AC-01:** Given Owen signs in and lands on Portal Home, when the screen loads, then he sees his client company's projects, each with its stage, milestone summary, and any items waiting on him.

**FEAT-05.SPEC-003-AC-02:** Given Priya signs in and lands on Portal Home, when she views her "Waiting on you" region, then it lists only deliverables waiting for her review and never a proposal or invoice item.

**FEAT-05.SPEC-003-AC-03:** Given Owen has a project with a proposal waiting on him, when he taps that waiting item, then he is navigated to FEAT-03 (Proposal Acceptance) for that specific proposal.

**FEAT-05.SPEC-003-AC-04:** Given Owen is on Portal Home, when he taps "Invite a colleague," then he is navigated into FEAT-18's invite-a-Reviewer flow.

**FEAT-05.SPEC-003-AC-05:** Given Priya is on Portal Home, when she looks for an "Invite a colleague" action, then it is not shown, since only Owen (Primary contact) can invite.

**FEAT-05.SPEC-003-AC-06:** Given a contact with no active projects lands on Portal Home, when the screen loads, then it shows "Nothing here yet -- {freelancer name} hasn't sent you anything to review." rather than an error.

**FEAT-05.SPEC-003-AC-07:** Given Owen's project data fails to load, when the load fails, then he sees "We couldn't load your projects. Try again." with a Retry button.

**FEAT-05.SPEC-003-AC-08:** Given Priya loses connectivity while viewing a previously loaded Portal Home, when connectivity drops, then the banner "You're offline -- showing the last loaded view." appears and the last-loaded project list remains visible.

**FEAT-05.SPEC-003-AC-09:** Given Priya's connectivity returns after the offline banner appeared, when connectivity is restored, then the banner clears and the screen refreshes automatically.

**FEAT-05.SPEC-003-AC-10:** Given Owen is a client contact for two different freelancers, when he signs into each freelancer's portal separately, then each Portal Home shows only that freelancer's projects under that freelancer's own branding, never both together.

**FEAT-05.SPEC-003-AC-11:** Given Owen taps a project card with no items currently waiting on him, when the card is shown, then it displays the project's stage and milestone summary with no "Waiting on you" region.

**FEAT-05.SPEC-003-AC-12:** Given Portal Home renders for either Owen or Priya, when the page becomes interactive, then it emits `portal_home_viewed` with the load time, supporting the mobile responsiveness target.

**FEAT-05.SPEC-003-AC-13:** Given Owen is on Portal Home, when he taps the "Made with Clientroom" referral mark, then he is navigated externally to the public product page (FEAT-33) and this screen's own state is unaffected.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loaded, loading, empty, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
