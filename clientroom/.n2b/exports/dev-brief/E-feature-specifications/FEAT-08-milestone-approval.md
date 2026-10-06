# FEAT-08 — Milestone Approval

This chapter covers Milestone Approval, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 7 specifications carrying 105 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-08.SPEC-001 | Milestone Review & Approval Screen | screen | 22 |
| FEAT-08.SPEC-002 | Milestone Reopen Screen | screen | 17 |
| FEAT-08.SPEC-003 | Approval Recording & Concurrency Guard | automation | 12 |
| FEAT-08.SPEC-004 | Next-Invoice Trigger | automation | 10 |
| FEAT-08.SPEC-005 | Reopen Recording | automation | 10 |
| FEAT-08.SPEC-006 | Approval Authorization & Eligibility Rules | logic-rule | 22 |
| FEAT-08.SPEC-007 | Milestone Approval Confirmation Notification | notification | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Milestone Approval

## Summary

**Feature:** Milestone Approval
**ID:** FEAT-08
**Description:** The client's Primary Contact approves a milestone once satisfied; approval is timestamped and permanent, and approving automatically issues the next invoice per the payment schedule.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Experience narrative: "the founder hits 'Approve,' and approving the milestone automatically issues the next invoice" — this is the product's central automation and the record cited as evidence in scope disputes (BRIEF.md, Success Criteria). MVP phase: the core billing loop depends on it. [RESEARCH-INFORMED: a lightweight approve-then-auto-invoice loop is not delivered as a first-class flow by any profiled competitor (derived from Dubsado's missing milestone sequencing, MEDIUM) — this is the product's clearest differentiator] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Approve a milestone — a timestamped, permanent decision
- Automatic next invoice — approval triggers the next invoice in the payment schedule
- Reopen (freelancer only) — a logged, non-silent way to reopen a milestone if needed

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-08.SPEC-001 | Milestone Review & Approval Screen | Screen | Owen (Primary Contact), Priya (Reviewer) | Owen reviews the deliverable and comment thread and approves the milestone; Priya sees the same status with no Approve control |
| FEAT-08.SPEC-002 | Milestone Reopen Screen | Screen | Nadia (Freelancer) | Nadia reopens an approved milestone through a deliberate, logged action, never a silent edit |
| FEAT-08.SPEC-003 | Approval Recording & Concurrency Guard | Automation | Owen (Primary Contact), Nadia (Freelancer) | Records the exactly-once, immutable approval timestamp and identity, refusing approval if the milestone changed underneath Owen since it was shown |
| FEAT-08.SPEC-004 | Next-Invoice Trigger | Automation | Owen (Primary Contact), Nadia (Freelancer) | On a successful approval, automatically fires the next invoice in the payment schedule with no freelancer action |
| FEAT-08.SPEC-005 | Reopen Recording | Automation | Nadia (Freelancer), Owen (Primary Contact) | Writes the reopen as a distinct logged event, resets the milestone to Reopened, and leaves the original approval record untouched |
| FEAT-08.SPEC-006 | Approval Authorization & Eligibility Rules | Logic/Rule | Owen (Primary Contact), Priya (Reviewer), Nadia (Freelancer) | Governs who may approve or reopen, when approval is allowed, and the exactly-once/immutability constraints on the action |
| FEAT-08.SPEC-007 | Milestone Approval Confirmation Notification | Notification | Owen (Primary Contact), Nadia (Freelancer) | Sends a confirmation email to Owen and Nadia the moment an approval is recorded |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Approve a milestone — a timestamped, permanent decision | FEAT-08.SPEC-001, FEAT-08.SPEC-003, FEAT-08.SPEC-006 | The screen exposes the Approve control; the automation writes the immutable timestamp/identity exactly once; the rule spec governs who may act and when | Phase 2 (Explicit) |
| Automatic next invoice — approval triggers the next invoice in the payment schedule | FEAT-08.SPEC-004 | Automation fires immediately on a successful approval, reading the Payment Schedule as it stood at approval time (XBR-02) | Phase 2 (Explicit) |
| Reopen (freelancer only) — a logged, non-silent way to reopen a milestone if needed | FEAT-08.SPEC-002, FEAT-08.SPEC-005, FEAT-08.SPEC-006 | The screen gives Nadia the reopen action; the automation records it as a distinct logged event; the rule spec restricts reopen to Nadia alone | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-08.SPEC-006 | Approval Authorization & Eligibility Rules | Phase 5 (Rule-Constraint Discovery) | The Access, Validation & Limits, and Contention fields together produce 5+ interacting rules (role gate, exactly-once, immutability, precondition on deliverable/thread load, reject-with-refresh concurrency, connectivity requirement) that govern both SPEC-001 and SPEC-003/005 — past the inline-validation threshold |
| FEAT-08.SPEC-007 | Milestone Approval Confirmation Notification | Phase 4 (Notification surfacing) | The Communications field names a confirmation email to Owen and Nadia with a defined audience and trigger — this carries delivery rules and cannot stay an inline toast |

## Entity-Lifecycle Coverage Matrix

**Entity: Milestone**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Milestones are created by Milestone & Payment Schedule Setup (FEAT-04); this feature never creates one | -- |
| Read (single) | FEAT-08.SPEC-001 | Milestone Review & Approval Screen loads the single milestone, its deliverable, and its comment thread | Also read (view-only) by Dana inside FEAT-31's logged support session, not by a screen this feature owns |
| Read (list) | N/A | This feature has no milestone list view; browsing the project's milestones is owned by FEAT-04/FEAT-05 | -- |
| Update | FEAT-08.SPEC-003, FEAT-08.SPEC-005 | SPEC-003 writes `approved_at`/`approved_by` on approval; SPEC-005 writes the Reopened status and its own logged event | Milestone `status`, `approved_at`, `approved_by` fields (feature-dependency-map.md, Entity: Milestone) |
| Delete/Archive | N/A | This feature never deletes a milestone; FEAT-04 may delete one only if it was never approved or invoiced, and XBR-10 forbids removal or re-pricing of an approved/invoiced milestone — deletion after approval is an explicit non-goal enforced by this feature's rules (see Non-Goals) | -- |
| State Transition | FEAT-08.SPEC-003 (-> Approved), FEAT-08.SPEC-005 (-> Reopened) | Approved and Reopened are the two transitions this feature owns; Defined and Deliverable Uploaded are owned by FEAT-04 and FEAT-06 | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deliverable | FEAT-08.SPEC-001 | The approval screen displays the milestone's current deliverable so Owen can review it before approving |
| Comment | FEAT-08.SPEC-001 | The approval screen displays the existing comment thread; the Approve control stays disabled until this has fully loaded |
| Payment Schedule | FEAT-08.SPEC-004 | The next-invoice trigger reads the schedule, as it stood at the moment of approval, to determine which invoice fires next |
| Client Contact | FEAT-08.SPEC-001, FEAT-08.SPEC-003, FEAT-08.SPEC-006 | Identifies the approving contact (Owen), enforces the role gate (Primary vs. Reviewer), and supplies the `approved_by` identity |
| Invoice | FEAT-08.SPEC-004 | Create is triggered by SPEC-004, but the Invoice record itself is created, numbered, and sent by Invoice Generation & Sending (FEAT-09); this feature never reads, updates, or deletes an Invoice directly |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Owen clicks Approve on a loaded, not-yet-approved milestone | Validate role, preconditions, and current milestone state; if valid, write the immutable `approved_at`/`approved_by` record exactly once | Standalone Automation | SPEC-003 |
| Owen clicks Approve but the milestone was re-priced/removed by Nadia since it was shown | Refuse the approval and show Owen the refreshed milestone instead of recording it | Inline in SPEC-003 (reject-with-refresh outcome), governed by SPEC-006 | SPEC-003 / SPEC-006 |
| Approval is successfully recorded | Automatically generate and send the next invoice in the payment schedule, with no freelancer action (XBR-02) | Standalone Automation | SPEC-004 |
| Approval is successfully recorded | Send a confirmation email to Owen and Nadia | Standalone Notification | SPEC-007 |
| Approval is successfully recorded | Write an append-only Activity Log Entry for the approval (XBR-05) | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| Nadia reopens an approved milestone | Write a distinct, logged reopen event; set milestone status to Reopened; leave the original approval record untouched | Standalone Automation | SPEC-005 |
| Nadia reopens an approved milestone | Write an append-only Activity Log Entry for the reopen (XBR-05); this is how the event stays "non-silent" -- no separate email is named for reopen in this feature's Communications field | Cross-feature -- owned by FEAT-13 | FEAT-13 responsibility |
| A failed approval action is retried by Owen | Retry without double-recording; the exactly-once guard in SPEC-003 prevents a second approval record | Inline in SPEC-001 (Error state), enforced by SPEC-003 | SPEC-001 / SPEC-003 |
| Approve is attempted while offline | Show a clear "reconnect to approve" state; the action never appears to succeed without connectivity | Inline in SPEC-001 (Offline/Degraded state) | SPEC-001 |
| Priya opens the milestone view | Show milestone status with no working Approve control, per her Reviewer role | Inline in SPEC-001, governed by SPEC-006 | SPEC-001 / SPEC-006 |

## Shared Context

**Shared Entities:**
- Milestone -- read by SPEC-001 for display; updated by SPEC-003 (Approved) and SPEC-005 (Reopened). Fields touched: `status`, `approved_at`, `approved_by`.
- Payment Schedule -- read by SPEC-004 to resolve the next invoice trigger; never written by this feature.
- Client Contact -- read by SPEC-001, SPEC-003, and SPEC-006 to establish the acting contact's identity and role (Primary vs. Reviewer).

**Shared UI Patterns:**
- Milestone review layout -- SPEC-001 presents the deliverable and comment thread identically to Owen and Priya; only the presence of a working Approve control differs by role. Spec Writers should describe one screen with a role-conditional control, not two screens.
- "This records your approval and issues the next invoice" consent copy -- required verbatim in spirit on SPEC-001's Approve control per the Accessibility expectation in the feature's States field; SPEC-007's confirmation email should echo the same plain description of what happened.

**Shared Validation:**
- SPEC-006 defines the authorization and eligibility rules (role gate, exactly-once, immutability, load-precondition, concurrency, connectivity). SPEC-001, SPEC-003, and SPEC-005 all reference SPEC-006 rather than restating these rules.

## Internal Dependency Map

```
SPEC-001 (Milestone Review & Approval Screen) -> [Owen clicks Approve] -> SPEC-006 (Authorization & Eligibility Rules) -> [pass] -> SPEC-003 (Approval Recording & Concurrency Guard)
SPEC-003 (Approval Recording & Concurrency Guard) -> [approval written] -> SPEC-004 (Next-Invoice Trigger)
SPEC-003 (Approval Recording & Concurrency Guard) -> [approval written] -> SPEC-007 (Milestone Approval Confirmation Notification)
SPEC-003 (Approval Recording & Concurrency Guard) -> [state changed] -> SPEC-001 (Milestone Review & Approval Screen shows "Approved on {date}")
SPEC-002 (Milestone Reopen Screen) -> [Nadia confirms reopen] -> SPEC-006 (Authorization & Eligibility Rules) -> [pass] -> SPEC-005 (Reopen Recording)
SPEC-005 (Reopen Recording) -> [milestone reopened] -> SPEC-001 (Milestone Review & Approval Screen returns to an unapproved state, Approve control re-enabled)
SPEC-001 (Milestone Review & Approval Screen) -> [validates access using] -> SPEC-006 (Authorization & Eligibility Rules)
```

**Default Entry:** SPEC-001 (Milestone Review & Approval Screen) -- the screen a client contact reaches from the milestone in the project view (navigated to from FEAT-07's deliverable/comment view) once a deliverable exists for the milestone.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-08.SPEC-001 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | The approval screen only exists once a deliverable has been uploaded for the milestone | Deliverable upload completes |
| FEAT-08.SPEC-001 | Inbound | FEAT-07 (Deliverable Review & Feedback) | Owen navigates from the deliverable/comment review view into the approval screen | Owen finishes reviewing the round and chooses Approve |
| FEAT-08.SPEC-004 | Outbound | FEAT-09 (Invoice Generation & Sending) | Approval fires the trigger; FEAT-09 owns invoice creation, numbering, and sending (XBR-02) | Milestone approval is successfully recorded |
| FEAT-08.SPEC-003, FEAT-08.SPEC-005 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Both the approval and the reopen write append-only Activity Log Entries (XBR-05) that Nadia later locates and shares as evidence | Approval or reopen is recorded |
| FEAT-08.SPEC-007 | Outbound | FEAT-14 (Notifications & Delivery) | The confirmation email is sent through the product's transactional email delivery capability, whose Integration spec is owned by FEAT-14 (this feature carries no Integration spec of its own for that capability, matching FEAT-02/FEAT-03's disposition) | Approval is recorded |
| FEAT-08.SPEC-001 | Inbound | FEAT-31 (Support Access) | Dana views the same milestone data read-only, inside a logged support session that FEAT-31 owns; this feature's screen is not shown to Dana directly | Dana opens a support session |
| FEAT-08.SPEC-003 | Inbound | FEAT-04 (Milestone & Payment Schedule Setup) | An approval attempt is refused, and the refreshed milestone shown, if Nadia re-priced or removed the milestone since Owen last saw it | Nadia edits the milestone concurrently with Owen's approval attempt |
| FEAT-08.SPEC-005 | Outbound | FEAT-04 (Milestone & Payment Schedule Setup) | A reopened milestone becomes eligible for Nadia's edits again; her edit attempts on a still-approved milestone are refused until a reopen occurs | Nadia attempts to edit an approved milestone |

## Non-Functional Notes

**Data volumes / growth:** Each milestone is approved at most once, with an occasional reopen; volume scales with a project's milestone count (typically single digits per project), so this feature carries no meaningful growth or scale concern of its own (assumptions-constraints.md, ASMP-21 context).

**Responsiveness:** The approval screen is a client-facing page and must become interactive within roughly 2 seconds on a typical mobile connection; the Approve action itself completes without a perceptible wait once pressed (assumptions-constraints.md, ASMP-21).

**Data sensitivity / privacy:** The approval record captures the approving contact's identity, which is personal data (assumptions-constraints.md, ASMP-24), alongside business-confidential milestone pricing; the record is evidentiary and immutable once written (assumptions-constraints.md, ASMP-25; feature-dependency-map.md, Entity: Milestone, Data Sensitivity).

**Compliance flags:** The approver's identity is GDPR-class personal data (assumptions-constraints.md, ASMP-24); if the approving contact later files an erasure request, their contact details are removed but their name stays on this approval record as evidence, per XBR-27 (owned by FEAT-18).

## Non-Goals

- **Client-side reversal or editing of a recorded approval** -- Excluded per the feature's own Validation & Limits field and BRIEF.md's record-immutability constraint (assumptions-constraints.md, ASMP-25): approval cannot be reversed by the client under any circumstance; only Nadia's logged reopen can change the milestone's course, and that is a new event, never an edit to the original record.
- **A multi-approver or staged sign-off flow** -- Excluded per scope-boundaries.md (SC-02): the persona set defines exactly two client-contact roles (Primary, Reviewer) with no additional client-side tiers, so there is no second approver to route a sign-off through.
- **A configurable approval workflow or conditional routing** -- Excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior (approve triggers the next invoice) rather than a configurable workflow or automation builder.
- **Automatic purge of approval records** -- Intentional lifecycle decision confirmed by the CRUD matrix: approval records are retained for the life of the freelancer's account with no automatic purge, per scope-boundaries.md (SC-24); they are removed only on account deletion (FEAT-24), subject to legal financial-record retention.
- **Reversing or cancelling an already-issued auto-invoice from this feature** -- Excluded: once SPEC-004 fires the trigger, correcting or cancelling the resulting invoice is owned entirely by Invoice Generation & Sending (FEAT-09) via a credit note or new invoice (XBR-04); this feature has no invoice-editing capability of its own.



# Screen Spec: Milestone Review & Approval Screen

## Overview

**Name:** Milestone Review & Approval Screen
**ID:** FEAT-08.SPEC-001
**Type:** Screen
**Purpose:** Owen reviews a milestone's current deliverable and comment thread and approves it with a single, plainly explained, timestamped action; Priya sees the identical status and content with no working Approve control.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Displaying the milestone's current Deliverable and its Comment thread for review before a decision
- The Approve control and its confirmation copy, available to Owen only
- Showing the milestone's approval state ("Approved on {date}") once approved
- The loading, error, offline, and stale-refresh states around the Approve action

**Non-Goals:**
- Posting or retracting comments -- handled by Deliverable Review & Feedback (FEAT-07); this screen displays the existing thread read-only and links out for participation.
- Uploading, replacing, or removing the deliverable -- handled by Deliverable Upload & Sharing (FEAT-06); this screen displays the current deliverable read-only.
- Reopening an approved milestone -- handled by FEAT-08.SPEC-002 (Milestone Reopen Screen), a freelancer-only, freelancer-facing screen that this one never exposes to any client contact.
- Defining the exact authorization, eligibility, exactly-once, and concurrency rules behind the Approve action -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this screen enforces and reflects rather than restates.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Owen or Priya finishes reviewing the current deliverable and its comments and navigates to the milestone-level view | The milestone reference; the deliverable and comment data already loaded is reused where possible to avoid a second full load |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Owen or Priya finishes reviewing milestone-level feedback and navigates to the approval view | The milestone reference |
| FEAT-06.SPEC-006 (Deliverable Ready Notification email) | A client contact opens the "deliverable ready for review" email and signs in via FEAT-05 | The milestone reference, landing directly on this screen after magic-link sign-in |
| FEAT-08.SPEC-005 (Reopen Recording) | A milestone Nadia just reopened returns Owen to an unapproved state on a screen he already had open | The same milestone reference; the screen re-renders in place with the Approve control restored |
| FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification) | Owen or Nadia taps the confirmation email's "View milestone" CTA | The approved milestone reference |
| FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | Owen or Priya taps a milestone row whose deliverable is ready for review (Priya sees no Approve control) | The milestone reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Owen (Client Primary Contact) | Full screen, for milestones belonging to his own client company (Own-only) | Approve, once the deliverable and comment thread have fully loaded and connectivity is present, and only while the milestone is not already Approved | -- |
| Priya (Client Reviewer Contact) | Full screen, for milestones belonging to her own client company (Own-only) -- identical deliverable, comment thread, and status display to Owen | None -- no Approve control is rendered for her at all | There is no denial message shown, because the control simply does not exist for this role; the screen otherwise behaves identically to Owen's read-only content |
| Nadia (Freelancer) | Not applicable -- this screen is client-facing only; Nadia never opens this exact screen | None | N/A -- Nadia's equivalent view of the same milestone lives in her own project view (FEAT-01) and the Reopen Screen (FEAT-08.SPEC-002), not this spec |
| Dana (Support Operator) | Full screen content, read-only, inside a logged support session (FEAT-31) -- no file downloads | None | Every action affordance (Approve) is rendered disabled with "Support access is read-only." |
| Unauthenticated | No | No | Redirected to FEAT-05's sign-in request page; after a fresh magic-link sign-in, the contact lands back on this exact milestone |
| Expired session | No | No | FEAT-05's "expired or invalid link" page with a plain explanation and a one-step request for a fresh sign-in link; no in-progress state exists to preserve, since this screen has no draft input |

## Layout and Content

**Header:** The project and milestone name, with the milestone's current status shown as a plain label ("Ready for your review," "Approved on {date}," or "Reopened -- new deliverable pending"). A back control returns to the client portal home (FEAT-05).

**Body, in order from top to bottom:**
- **Deliverable panel:** The milestone's current, Active Deliverable -- its kind (uploaded file or linked external asset), a preview where feasible, and a link to open it in full via FEAT-06's viewing surface. If the deliverable is a linked asset, the panel shows the link and its reachability state as recorded by FEAT-06.
- **Comment thread panel, below the deliverable panel:** The milestone's comment thread (Comment entity, target: milestone), read-only on this screen, in post-time order, each entry showing author name, role (Nadia, Owen, or Priya), and posted time. A "Reply in thread" link navigates out to FEAT-07.SPEC-002 (Milestone Comment Thread) for anyone who wants to post.
- **Decision area, at the bottom of the body (Owen only):** The Approve control and, directly beneath it, the fixed consent line: "This records your approval and issues the next invoice." Neither the control nor the line is shown to Priya.
- **Approved marker (once approved, replaces the Decision area for every role):** "Approved on {date}" with the approving contact's name, shown to Owen, Priya, and Dana alike.

**Footer:** None -- Approve sits inline in the body's decision area, not in a persistent footer, since it is a single deliberate action rather than a recurring form action.

### Responsive Behavior

- **Compact size class:** Single column, full width, in the stacking order described above; the deliverable preview scales to the available width and the comment thread scrolls independently beneath it.
- **Medium size class and above:** The deliverable panel and comment thread panel sit side by side (deliverable left, comments right), each independently scrollable; the decision area (or Approved marker) spans the full width beneath both, so the consent line always reads across the whole screen width regardless of size class.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-05 (portal home) | Screen closes | Standard navigation transition |
| Deliverable panel "Open" link | Tap | Navigate to FEAT-06's viewing surface for the current deliverable, or open the linked external asset in a new context | This screen remains open behind the navigation | Standard navigation or new-context transition |
| "Reply in thread" link | Tap | Navigate to FEAT-07.SPEC-002 (Milestone Comment Thread) | This screen closes | Standard navigation transition |
| Comment thread entries | Display only | None -- read-only on this screen | None | No interaction; posting happens on FEAT-07.SPEC-002 |
| Approve control (Owen only) | Tap, while enabled | 1. Disable the control and show a brief in-progress state. 2. Trigger FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard). 3. Await the outcome. | Control shows an in-progress state during the request | Success: the Decision area is replaced by the "Approved on {date}" marker. Stale/already-approved: refresh dialog described in States, then the marker. Connectivity/failure: the relevant error/offline state described in States. |
| Approve control (Owen only) | Tap, while disabled (loading or offline) | No action -- the control is inert until its enabling condition is met | None | The control's disabled appearance itself communicates why (loading spinner, or the offline banner already visible above it) |

### Accessibility Notes

- **Focus order:** Back control -> Deliverable panel's Open link -> comment thread entries (in post order) -> "Reply in thread" link -> Approve control (when present) -> Approved marker (when present).
- **Consent line association:** The "This records your approval and issues the next invoice" line is programmatically associated with the Approve control, so assistive technology announces it as part of the control's own description, not as separate incidental text.
- **Dynamic announcements:** A successful approval's transition to the "Approved on {date}" marker is announced to assistive technology as a live region update. The stale-refresh dialog and any error/offline banner are announced immediately when they appear.
- **Keyboard alternatives:** Every action on this screen (navigation links, Approve) is reachable and actionable by keyboard alone; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this view exists only once a deliverable has been uploaded for the milestone (product-features.md, States); a milestone with no deliverable yet has no route into this screen, so there is no empty-state rendering to define here | Never entered -- excluded by the milestone's own lifecycle (Defined -> Deliverable Uploaded before this screen becomes reachable) | N/A |
| Loading | Deliverable and comment thread panels show loading placeholders; the Approve control (if the role would otherwise see one) is disabled | Screen first opens | Deliverable and comment thread both finish loading |
| Ready for review (default, Owen) | Deliverable and comments fully loaded; Approve control enabled with the consent line beneath it | Loading completes for a milestone not yet Approved, viewed by Owen | Owen taps Approve, or the milestone's status changes underneath him (reopen elsewhere, or another approval race) |
| Ready for review (Priya / read-only) | Deliverable and comments fully loaded; no Approve control anywhere in the layout | Loading completes for a milestone not yet Approved, viewed by Priya or Dana | Milestone's status changes to Approved |
| Approving (in progress) | Approve control shows an in-progress state; the rest of the screen remains visible and stable | Owen taps Approve | FEAT-08.SPEC-003 returns an outcome |
| Approved | Decision area replaced by "Approved on {date}" with the approving contact's name, visible to every role that can view this screen | A successful approval outcome is returned, or the screen loads for a milestone already Approved | A subsequent reopen (FEAT-08.SPEC-005) changes status away from Approved |
| Stale refresh (Owen) | A dialog: "This milestone has changed since you opened it. Here's the current version." with an acknowledgment action that reloads the deliverable, comments, and current status | FEAT-08.SPEC-003 returns the stale-attempt or unauthorized-because-already-approved outcome | Owen acknowledges the dialog, and the screen reloads to the live state |
| Error (approval failed) | An error banner above the Decision area: "Something went wrong recording your approval. Try again." with a Retry action; the milestone's displayed status is unchanged | FEAT-08.SPEC-003 returns the write-failure outcome | Owen taps Retry (re-triggers the Approve interaction) or navigates away |
| Load error | An error banner in place of the deliverable and/or comment panel that failed to load, with a Retry action; the Approve control (if applicable) stays disabled until loading succeeds | The deliverable or comment thread fails to load | Owen or Priya taps Retry and loading succeeds |
| Offline/Degraded | A persistent banner: "You're offline. Reconnect to approve this milestone." Already-loaded deliverable and comment content remain visible and readable; the Approve control is disabled for the duration, and no approval is queued for later -- an approval never appears to succeed without a live connection | Connectivity is lost while the screen is open, or the screen loads without connectivity | Connectivity is restored; the Approve control re-enables once the milestone's current state is confirmed fresh |

## Validation Rules

Validation and eligibility governed by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules). See that spec for the full role, state, and concurrency conditions gating the Approve action; this screen renders the resulting Access and Visibility and States behavior above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | FEAT-05 (portal home) | FEAT-05 -- Client Portal Access |
| Deliverable "Open" link tap | FEAT-06 (deliverable viewing surface) | FEAT-06 -- Deliverable Upload & Sharing |
| "Reply in thread" link tap | FEAT-07.SPEC-002 (Milestone Comment Thread) | FEAT-07 -- Deliverable Review & Feedback |
| Successful approval | Stays on this screen, now showing the Approved state | -- |

## Data Model

**Creates:** None directly -- the approval write itself is performed by FEAT-08.SPEC-003, which this screen triggers.
**Reads:** Milestone -- `name`, `status`, `approved_at`, `approved_by`. Deliverable -- the milestone's current, Active deliverable: `kind`, `file`/`link`, `link_status`. Comment -- the milestone's comment thread: `text`, `author`, `posted_at`, `status` (Posted entries only; Retracted entries are omitted per FEAT-07's own display rule).
**Updates:** None directly -- Milestone `status`/`approved_at`/`approved_by` are updated only by FEAT-08.SPEC-003, triggered from this screen.
**Deletes:** None.

## Business Rules

- The Approve control's eligibility, exactly-once, immutability, and concurrency behavior are governed entirely by FEAT-08.SPEC-006 and enforced by FEAT-08.SPEC-003; this screen only reflects their outcomes.
- XBR-09: an out-of-scope or expired milestone link never reveals another client company's data -- it shows a plain explanation and a fresh-link path, handled by FEAT-05.
- Approving is a permanent, evidentiary act (BRIEF.md, Constraints: record immutability) -- once the Approved marker is shown, there is no client-side action anywhere on this screen that reverses it; only Nadia's reopen (FEAT-08.SPEC-005), on a screen this feature never exposes to a client contact, changes the state again.
- The consent line ("This records your approval and issues the next invoice") is required, verbatim, wherever the Approve control appears -- it is the product's stated accessibility and informed-consent commitment for this irreversible action (product-features.md, States: Accessibility).

## Edge Cases

- **Owen taps Approve twice in rapid succession** -- The control disables on the first tap; the second tap has no effect while disabled, so exactly one approval request is ever sent from this screen (the exactly-once guarantee itself is enforced server-side by FEAT-08.SPEC-003 regardless).
- **The milestone was re-priced or had its deliverable removed by Nadia since this screen loaded, and Owen taps Approve** -- FEAT-08.SPEC-003 refuses the write; this screen shows the stale-refresh dialog and reloads the deliverable, comments, and current status rather than recording the approval against what Owen was actually shown.
- **The milestone is approved by an already-completed request from another of Owen's sessions while this screen is open** -- The Approve tap on this screen is refused as stale (already approved); the stale-refresh dialog reloads to the current "Approved on {date}" state.
- **Owen navigates away mid-approval (Approving state) and returns** -- The screen re-loads fresh on return and reflects whatever outcome the in-flight request ultimately produced (Approved, or still awaiting his action if the request had not yet completed) -- there is no separate draft or unsaved-approval state to restore, since Approve has no intermediate input to lose.
- **Priya opens this screen for a milestone Owen has already approved** -- She sees the "Approved on {date}" marker identically to Owen; nothing about her role changes once approved, since she never had an Approve control to begin with.
- **Nadia reopens the milestone while Owen has this screen open** -- The screen's displayed status is a snapshot from load; Owen sees the reopened state only on his next load or a live-update refresh, at which point the Approve control reappears -- this screen does not silently flip state without a reload, since an approval decision must be made against a state the reviewer has actually seen refreshed.
- **A milestone that Owen already approved is later reopened, and he returns to this screen** -- He sees the Approve control again, exactly as he would for a milestone approved for the first time; the prior approval's history is not shown inline here (it lives in the Activity Log via FEAT-13), only the current cycle's status.
- **Concurrent-edit conflict on the underlying Milestone entity (the shared-entity contention case this screen must cover, per the dependency map's Milestone Contention note)** -- Resolution is reject-with-refresh, identical to the stale-approval edge case above: any mismatch between what Owen was shown and the milestone's live state at the moment he acts refuses the write and reloads him to the current state, rather than recording his decision against data that has since changed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Navigation (inbound) | Owen or Priya arrives here after reviewing the deliverable and its comments |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Navigation (inbound / outbound) | Arrival after milestone-level review; "Reply in thread" navigates back out to post |
| FEAT-06 (Deliverable Upload & Sharing) | Navigation (outbound) / References (inbound) | Displays the current deliverable read-only and links to its full viewing surface |
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Triggers (outbound) | The Approve interaction fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Governs whether the Approve control renders and is actionable |
| FEAT-08.SPEC-005 (Reopen Recording) | References (inbound) | A successful reopen returns this screen to its unapproved, Approve-enabled state |
| FEAT-05 (Client Portal Access) | Navigation (inbound / outbound) | Entry after magic-link sign-in; exit via the back control |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_review_screen_viewed | role (Owen / Priya / Dana), milestone status at view time | Screen finishes loading | supports success-metrics.md: "Milestone Approval Turnaround" |
| milestone_approve_tapped | time since deliverable marked ready for review | Owen taps an enabled Approve control | supports success-metrics.md: "Milestone Approval Turnaround" |
| milestone_approval_confirmed_on_screen | time from tap to confirmed outcome | This screen renders the "Approved on {date}" marker following a successful approval | supports success-metrics.md: "Milestone Approval Turnaround" |
| milestone_approval_stale_refresh_shown | trigger reason (re-priced / deliverable removed / already approved) | The stale-refresh dialog is shown to Owen | N/A -- no Stage 2 metric measures refused-attempt friction directly; retained so a pattern of stale refreshes (which would slow real-world approval turnaround) is visible rather than indistinguishable from ordinary slow review |

## Acceptance Criteria

**FEAT-08.SPEC-001-AC-01:** Given Owen navigates from FEAT-07.SPEC-001 to a milestone whose deliverable and comments have not yet finished loading, when the screen opens, then it shows the Loading state and the Approve control is disabled.

**FEAT-08.SPEC-001-AC-02:** Given Owen is on this screen for a milestone with status "Deliverable Uploaded," when loading completes, then the deliverable panel, comment thread, and an enabled Approve control with its consent line are all shown.

**FEAT-08.SPEC-001-AC-03:** Given Priya is on this screen for the same milestone, when loading completes, then she sees the identical deliverable and comment content, but no Approve control anywhere in the layout.

**FEAT-08.SPEC-001-AC-04:** Given Owen taps the enabled Approve control, when the tap registers, then the control disables, shows an in-progress state, and triggers FEAT-08.SPEC-003.

**FEAT-08.SPEC-001-AC-05:** Given FEAT-08.SPEC-003 returns a successful outcome, when the response arrives, then the Decision area is replaced by "Approved on {date}" with Owen's name, announced to assistive technology as a live update.

**FEAT-08.SPEC-001-AC-06:** Given FEAT-08.SPEC-003 returns a stale-attempt outcome because Nadia changed the milestone since load, when the response arrives, then the stale-refresh dialog appears and, once acknowledged, the screen reloads to the current deliverable, comments, and status.

**FEAT-08.SPEC-001-AC-07:** Given FEAT-08.SPEC-003 returns a write-failure outcome, when the response arrives, then an error banner with a Retry action appears above the Decision area, and the milestone's displayed status is unchanged.

**FEAT-08.SPEC-001-AC-08:** Given Owen loses connectivity while this screen is open, when the loss is detected, then the offline banner "You're offline. Reconnect to approve this milestone." appears, the Approve control disables, and no approval is queued.

**FEAT-08.SPEC-001-AC-09:** Given Owen regains connectivity after the offline banner appeared, when the screen confirms the milestone's current state is fresh, then the Approve control re-enables (if the milestone is still eligible).

**FEAT-08.SPEC-001-AC-10:** Given Owen taps Approve twice in rapid succession, when the first tap disables the control, then the second tap has no effect and only one request is sent.

**FEAT-08.SPEC-001-AC-11:** Given the milestone's status is already "Approved" when this screen first loads, when loading completes, then the "Approved on {date}" marker is shown immediately with no Decision area, for every role that can view the screen.

**FEAT-08.SPEC-001-AC-12:** Given Dana is in a logged, read-only support session viewing this milestone, when the screen loads, then she sees the full deliverable and comment content with every control shown disabled and labeled "Support access is read-only."

**FEAT-08.SPEC-001-AC-13:** Given an unauthenticated visitor follows a milestone link, when the link resolves, then they are redirected to FEAT-05's sign-in request page.

**FEAT-08.SPEC-001-AC-14:** Given a client contact's session has expired, when they attempt to open this screen, then they see FEAT-05's expired-link page with a fresh-link request option.

**FEAT-08.SPEC-001-AC-15:** Given the deliverable panel fails to load, when the failure occurs, then a load-error banner with Retry appears in its place, and the Approve control stays disabled until it succeeds.

**FEAT-08.SPEC-001-AC-16:** Given Owen taps "Reply in thread," when the tap registers, then he navigates to FEAT-07.SPEC-002 and this screen closes.

**FEAT-08.SPEC-001-AC-17:** Given Nadia reopens a milestone that Owen previously approved, when Owen reloads this screen, then he sees the Approve control again instead of the "Approved on {date}" marker.

**FEAT-08.SPEC-001-AC-18:** Given Priya follows a link to a milestone belonging to a different client company than her own, when the link resolves, then she sees a plain explanation and a fresh-link option, never that company's data.

**FEAT-08.SPEC-001-AC-19:** Given Owen approves a milestone whose live state matches what he was shown, when the approval succeeds, then the milestone_approval_confirmed_on_screen event fires with the elapsed time from tap to confirmation.

**FEAT-08.SPEC-001-AC-20:** Given two of Owen's own sessions both have this screen open for the same milestone and one approves it first, when the second session's Approve tap is processed, then FEAT-08.SPEC-003 refuses it as stale and this screen shows the stale-refresh dialog, resolving to the current Approved state -- the concurrent-edit conflict on the shared Milestone entity.

**FEAT-08.SPEC-001-AC-21:** Given Owen or Priya taps the Back control, when the tap registers, then they navigate to FEAT-05's portal home and this screen closes, with no data change of any kind.

**FEAT-08.SPEC-001-AC-22:** Given Owen or Priya taps the Deliverable panel's "Open" link, when the tap registers, then FEAT-06's viewing surface for the current deliverable opens (or the linked external asset opens in a new context), and this screen remains open behind it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 10 (empty, loading, ready-owen, ready-readonly, approving, approved, stale refresh, error, load error, offline) | 10 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |



# Screen Spec: Milestone Reopen Screen

## Overview

**Name:** Milestone Reopen Screen
**ID:** FEAT-08.SPEC-002
**Type:** Screen
**Purpose:** Nadia reopens an approved milestone through a deliberate, explained, logged confirmation -- never a silent status edit -- when genuinely necessary.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- The reopen confirmation flow: what reopening does, and Nadia's deliberate confirmation of it
- Showing the milestone's current approval details (who approved it, when) before Nadia confirms
- The loading, error, offline, and already-changed states around the reopen action

**Non-Goals:**
- Approving a milestone -- the reverse action belongs entirely to Owen on FEAT-08.SPEC-001; Nadia has no approve capability anywhere in the product.
- Editing the milestone's name, price, or schedule once reopened -- handled by FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor), which becomes reachable again only after this screen's reopen action succeeds.
- Reversing or cancelling the invoice the original approval generated -- excluded per the Brief's Non-Goals: correcting that invoice is owned entirely by FEAT-09 via a credit note or new invoice (XBR-04); this screen has no invoice-editing surface.
- Defining the exact authorization and eligibility rule behind reopen -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this screen enforces and reflects rather than restates.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (project view, milestones area) | Nadia opens an Approved milestone from her own project view and chooses Reopen | The milestone reference |
| FEAT-04.SPEC-002 (Milestone Timeline & Client View, freelancer's own equivalent view) | Nadia sees a milestone marked Approved and chooses Reopen from its detail | The milestone reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, for any of her own milestones currently Approved | Confirm reopen, once the milestone's live status is confirmed as Approved and connectivity is present | -- |
| Owen (Client Primary Contact) | None -- this screen has no client-facing route | None | This screen is never reachable from the client portal; no navigation path or deep link into it exists for a client contact |
| Priya (Client Reviewer Contact) | None | None | Same as Owen -- unreachable from the client portal |
| Dana (Support Operator) | None -- reopening is a write action, and Dana's support access is read-only everywhere (ASMP-18); she has no reason to reach this specific confirmation screen | None | This screen is not part of the read-only support session surface; Dana views milestone status through FEAT-08.SPEC-001's read-only rendering instead |
| Unauthenticated | No | No | Redirected to Nadia's own sign-in; this is a freelancer-only screen, not a client-portal one |
| Expired session | No | No | Redirected to sign-in with a session-expired notice; no in-progress state exists to preserve |

## Layout and Content

**Header:** "Reopen this milestone?" with the milestone's name and project, and a back control that returns to the milestone's detail view without reopening anything.

**Body, in order from top to bottom:**
- **Current approval summary:** "Approved on {date} by {approving contact's name}." Read-only.
- **Explanation block:** Fixed copy explaining the consequence: "Reopening lets you make changes to this milestone again. It will show as Reopened to {client company name}, and this action is recorded. The original approval stays on record." (the client company name is the milestone's owning Client's `client_name`).
- **Confirm area:** A "Reopen Milestone" confirmation control and a "Cancel" control, side by side.

**Footer:** None -- the confirm/cancel pair sits in the body, since this screen has only one decision to make.

### Responsive Behavior

- **Compact size class:** Single column, full width, in the order described above; the Confirm and Cancel controls stack full-width, Reopen above Cancel.
- **Medium size class and above:** Uniform scaling, no structural change -- the content is capped at a consistent platform-wide narrow-form width (the design layer's decision) and centered; Confirm and Cancel sit side by side rather than stacked.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate away without reopening | Screen closes | Standard navigation transition |
| Cancel control | Tap | Discard the reopen attempt, no write occurs | Screen closes, returning to the milestone's detail view | Standard navigation transition |
| "Reopen Milestone" control | Tap, while enabled | 1. Disable both controls and show an in-progress state. 2. Trigger FEAT-08.SPEC-005 (Reopen Recording). 3. Await the outcome. | Confirm control shows an in-progress state during the request | Success: navigate to the milestone's detail view, now showing "Reopened." Already-changed: refresh message described in States. Connectivity/failure: the relevant error/offline state described in States. |
| "Reopen Milestone" control | Tap, while disabled (loading or offline) | No action -- inert until its enabling condition is met | None | The control's disabled appearance communicates why |

### Accessibility Notes

- **Focus order:** Back control -> Current approval summary (read, not focusable) -> Explanation block (read, not focusable) -> Reopen Milestone control -> Cancel control.
- **Confirmation announcement:** A successful reopen's navigation away from this screen is preceded by an announced confirmation so Nadia is not left uncertain whether the action completed before the screen changes.
- **Dynamic announcements:** The already-changed message and any error/offline banner are announced immediately when they appear.
- **Keyboard alternatives:** Every action on this screen (Back, Cancel, Reopen) is reachable and actionable by keyboard alone; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen exists only for a milestone that is currently Approved, reached exclusively from an Approved milestone's own detail view; there is no list or browsing surface within this screen for an empty condition to apply to | Never entered -- excluded by this screen's own entry points, which only route in from an already-Approved milestone | N/A |
| Loading | Approval summary shows a loading placeholder; Reopen and Cancel controls disabled | Screen first opens | The milestone's current approval details finish loading |
| Ready to confirm (default) | Approval summary, explanation, and both controls fully rendered and enabled | Loading completes for a milestone still "Approved" | Nadia taps Reopen or Cancel, or the milestone's status changes underneath her |
| Confirming (in progress) | Reopen control shows an in-progress state; Cancel is also disabled during the request | Nadia taps Reopen Milestone | FEAT-08.SPEC-005 returns an outcome |
| Already changed | A message in place of the confirm area: "This milestone's status has changed. Here's the current state." with an acknowledgment action that navigates to the milestone's current detail view | FEAT-08.SPEC-005 returns the already-changed outcome | Nadia acknowledges the message and navigates away |
| Error (reopen failed) | An error banner above the confirm area: "Something went wrong reopening this milestone. Try again." with a Retry action | FEAT-08.SPEC-005 returns the write-failure outcome | Nadia taps Retry or navigates away |
| Load error | An error banner in place of the approval summary, with a Retry action; the Reopen control stays disabled until it succeeds | The current approval details fail to load | Nadia taps Retry and loading succeeds |
| Offline/Degraded | A persistent banner: "You're offline. Reconnect to reopen this milestone." The Reopen control is disabled for the duration; no reopen is queued for later -- reopening never appears to succeed without a live connection | Connectivity is lost while the screen is open, or the screen loads without connectivity | Connectivity is restored and the milestone's current state is confirmed fresh |

## Validation Rules

Validation and eligibility governed by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules). See that spec for the milestone-state precondition and the rule that this action is Nadia-only; this screen renders the resulting Access and Visibility and States behavior above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | The milestone's detail view (FEAT-01 project view or FEAT-04.SPEC-002) | -- |
| Cancel tap | Same as Back | -- |
| Successful reopen | The milestone's detail view, now showing "Reopened" | -- |
| Already-changed acknowledgment | The milestone's current detail view | -- |

## Data Model

**Creates:** None directly -- the reopen write itself is performed by FEAT-08.SPEC-005, which this screen triggers.
**Reads:** Milestone -- `name`, `status`, `approved_at`, `approved_by`. Client -- `client_name`, for the explanation copy.
**Updates:** None directly -- Milestone `status` is updated only by FEAT-08.SPEC-005, triggered from this screen.
**Deletes:** None.

## Business Rules

- Reopen is Nadia-only and requires the milestone to currently be Approved -- governed entirely by FEAT-08.SPEC-006 and enforced by FEAT-08.SPEC-005; this screen only reflects their outcomes.
- Reopening is a deliberate, logged action, never a silent edit (Key Capabilities) -- this screen exists specifically so reopening always passes through an explicit confirmation with a stated consequence, rather than a status field a freelancer could change incidentally from a list view.
- The original approval record is never altered by a reopen (XBR-04) -- the explanation copy states this plainly so Nadia understands the reopen adds a new event rather than erasing the prior one.

## Edge Cases

- **Nadia taps Reopen Milestone twice in rapid succession** -- The controls disable on the first tap; the second tap has no effect while disabled, so exactly one request is ever sent from this screen.
- **The milestone was already reopened (or re-approved and reopened again) by an action from another of Nadia's sessions since this screen loaded** -- FEAT-08.SPEC-005 returns the already-changed outcome; this screen shows the acknowledgment message and, once acknowledged, sends Nadia to the milestone's current state rather than claiming a reopen that did not happen as she expected.
- **Nadia navigates away mid-confirmation (Confirming state) and returns** -- The screen reloads fresh on return and reflects whatever outcome the in-flight request ultimately produced -- there is no separate draft state to restore, since Reopen has no intermediate input to lose.
- **Nadia opens this screen for a milestone that is not currently Approved (reached through a stale link or a shared bookmark)** -- Loading resolves with the live status already not "Approved"; this screen shows the already-changed message immediately rather than a confirm area for an action that cannot succeed.
- **Nadia cancels after reading the explanation** -- No write occurs; she returns to the milestone's detail view exactly as it was before she opened this screen.
- **The client company (Owen or Priya) opens the milestone at the exact moment Nadia's reopen commits** -- Not a conflict for this screen: Nadia's Confirm/Cancel are the only writes this screen ever attempts, and the client side has no write path into this same action; the client contact's next load of FEAT-08.SPEC-001 simply reflects the reopened state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01 (Client & Project Management) | Navigation (inbound) | Nadia arrives here from an Approved milestone in her project view |
| FEAT-04.SPEC-002 (Milestone Timeline & Client View) | Navigation (inbound) | Nadia arrives here from the milestone's freelancer-side detail |
| FEAT-08.SPEC-005 (Reopen Recording) | Triggers (outbound) | The Reopen Milestone confirmation fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Governs whether reopen is actionable and who may reach it |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | References (outbound) | A successful reopen here returns that screen to its unapproved, Approve-enabled state |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Navigation (outbound) | Becomes reachable for edits on this milestone again only after a successful reopen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_reopen_screen_viewed | time since the original approval | Screen finishes loading in the Ready-to-confirm state | N/A -- no Stage 2 metric measures reopen-screen views; retained as the funnel step preceding milestone_reopened_by_freelancer (FEAT-08.SPEC-005), which is itself N/A for the same reason -- reopen frequency has no connected success metric, but the event pair keeps the reopen path observable |
| milestone_reopen_confirmed | time since the original approval | Nadia's reopen succeeds | N/A -- see above; the reopen path has no connected Stage 2 metric, and this event is retained purely for observability of the Key Capability's usage, not for a specific target |
| milestone_reopen_cancelled | -- | Nadia taps Cancel | N/A -- no Stage 2 metric measures abandoned reopens; retained so a pattern of hesitation before reopening (which could indicate the explanation copy is unclear) remains visible |

## Acceptance Criteria

**FEAT-08.SPEC-002-AC-01:** Given Nadia opens this screen for a milestone whose approval details have not yet finished loading, when the screen opens, then it shows the Loading state and both controls are disabled.

**FEAT-08.SPEC-002-AC-02:** Given Nadia is on this screen for a milestone that is currently Approved, when loading completes, then she sees "Approved on {date} by {approving contact's name}," the explanation block, and enabled Reopen Milestone and Cancel controls.

**FEAT-08.SPEC-002-AC-03:** Given Nadia taps the enabled Reopen Milestone control, when the tap registers, then both controls disable, an in-progress state shows, and FEAT-08.SPEC-005 is triggered.

**FEAT-08.SPEC-002-AC-04:** Given FEAT-08.SPEC-005 returns a successful outcome, when the response arrives, then Nadia is navigated to the milestone's detail view, now showing "Reopened."

**FEAT-08.SPEC-002-AC-05:** Given FEAT-08.SPEC-005 returns an already-changed outcome, when the response arrives, then the acknowledgment message "This milestone's status has changed. Here's the current state." appears, and acknowledging it navigates to the milestone's current detail view.

**FEAT-08.SPEC-002-AC-06:** Given FEAT-08.SPEC-005 returns a write-failure outcome, when the response arrives, then an error banner with a Retry action appears, and the milestone's approval details remain unchanged.

**FEAT-08.SPEC-002-AC-07:** Given Nadia loses connectivity while this screen is open, when the loss is detected, then the offline banner "You're offline. Reconnect to reopen this milestone." appears and the Reopen control disables.

**FEAT-08.SPEC-002-AC-08:** Given Nadia regains connectivity after the offline banner appeared, when the screen confirms the milestone is still Approved, then the Reopen control re-enables.

**FEAT-08.SPEC-002-AC-09:** Given Nadia taps Cancel, when the tap registers, then no write occurs and she returns to the milestone's detail view unchanged.

**FEAT-08.SPEC-002-AC-10:** Given Nadia taps Reopen Milestone twice in rapid succession, when the first tap disables the controls, then the second tap has no effect and only one request is sent.

**FEAT-08.SPEC-002-AC-11:** Given Owen or Priya attempts to reach this screen through any path, when the attempt is made, then no such route exists in the client portal.

**FEAT-08.SPEC-002-AC-12:** Given Dana is in a support session, when she views this milestone, then she sees its status through FEAT-08.SPEC-001's read-only rendering, never through this screen.

**FEAT-08.SPEC-002-AC-13:** Given an unauthenticated visitor attempts to reach this screen, when the attempt is made, then they are redirected to Nadia's own sign-in.

**FEAT-08.SPEC-002-AC-14:** Given Nadia's session has expired, when she attempts to open this screen, then she is redirected to sign-in with a session-expired notice.

**FEAT-08.SPEC-002-AC-15:** Given the current approval details fail to load, when the failure occurs, then a load-error banner with Retry appears in place of the approval summary, and the Reopen control stays disabled.

**FEAT-08.SPEC-002-AC-16:** Given Nadia reaches this screen through a stale link for a milestone that is no longer Approved, when loading completes, then she sees the already-changed message immediately, with no confirm area shown.

**FEAT-08.SPEC-002-AC-17:** Given Nadia taps the Back control, when the tap registers, then no write occurs and she navigates to the milestone's detail view unchanged, identically to Cancel.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 8 (empty, loading, ready, confirming, already changed, error, load error, offline) | 8 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Approval Recording & Concurrency Guard

## Overview

**Name:** Approval Recording & Concurrency Guard
**ID:** FEAT-08.SPEC-003
**Type:** Automation
**Purpose:** Records a milestone's approval exactly once, atomically writing the immutable timestamp and approving contact's identity, and refuses the attempt -- showing the refreshed milestone instead -- if the milestone changed underneath the approving contact since it was shown.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Re-validating Owen's authorization and the milestone's eligibility at the moment of the Approve attempt, not just at screen load
- Atomically writing `status` (-> Approved), `approved_at`, and `approved_by` exactly once
- Detecting and refusing an approval attempt made against a stale milestone view (reject-with-refresh)
- Firing the downstream next-invoice trigger and confirmation notification on a successful write
- Firing the Activity Log entry for the approval event

**Non-Goals:**
- Rendering the Approve control, the loading/offline/error states the client sees, and the client-visible retry affordance -- owned by FEAT-08.SPEC-001 (Milestone Review & Approval Screen), which this automation reports its outcome back to.
- Defining who may approve, under what milestone state, and what "already approved" or "stale" means -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this automation enforces rather than restates.
- Generating the next invoice itself -- owned by FEAT-08.SPEC-004 (Next-Invoice Trigger), which this automation only fires on success; invoice numbering, content, and sending belong entirely to FEAT-09.
- Recording a reopen or any reversal of an approval -- owned by FEAT-08.SPEC-005 (Reopen Recording); this automation has no path back out of "Approved" once it writes.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Owen taps Approve | FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Fires whenever the Approve control is actionable and Owen taps it, whether or not the underlying milestone has changed since the screen loaded | The milestone reference and the version/state Owen's screen last loaded (for the staleness check), Owen's authenticated Client Contact identity, the current timestamp |

## Processing Logic

1. Receive the Approve request from FEAT-08.SPEC-001, carrying the milestone reference, the milestone state as last loaded by Owen's screen, and Owen's authenticated Client Contact identity.
2. Confirm connectivity and an authenticated session are present; if either is missing, stop and return the connectivity/session failure outcome without touching the milestone record.
3. Re-check authorization per FEAT-08.SPEC-006: confirm the requesting identity is a Primary Contact (Owen's role) for the milestone's own client company. If this check fails, stop and return the unauthorized outcome.
4. Read the milestone's current, live state (not the state Owen's screen last loaded).
5. Compare the live state against the state Owen's screen last loaded, specifically: `status`, and whether the current Deliverable and its price/eligibility fields differ from what was shown.
6. If the live state differs from what was shown (the milestone was re-priced, its deliverable was removed, or `status` is no longer "Deliverable Uploaded" or "Reopened"), stop and return the stale-attempt outcome -- no write occurs.
7. If the live state matches, atomically write, in a single operation: `status` -> "Approved", `approved_at` -> the current timestamp, `approved_by` -> Owen's identity.
8. Confirm the write committed successfully before reporting success -- a write that cannot be confirmed as committed is treated as a failure, never as an assumed success.
9. On a confirmed successful write, fire FEAT-08.SPEC-004 (Next-Invoice Trigger) and FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification).
10. On a confirmed successful write, fire the append-only Activity Log entry to FEAT-13 (event type "milestone approved," actor Owen, the timestamp from step 7, the Milestone as the affected record) per XBR-05.
11. Return the outcome (success, stale, unauthorized, or failure) to FEAT-08.SPEC-001 for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Approval recorded | Live state matches what Owen's screen loaded; write commits and is confirmed | Milestone `status`, `approved_at`, `approved_by` set atomically | FEAT-08.SPEC-001 shows "Approved on {date}" in place of the Approve control | FEAT-08.SPEC-001, FEAT-08.SPEC-004, FEAT-08.SPEC-007, FEAT-13 |
| Stale attempt refused | Milestone was re-priced, its deliverable removed, or its status changed since Owen's screen loaded | None -- no partial write | FEAT-08.SPEC-001 shows a refresh dialog: "This milestone has changed since you opened it. Here's the current version." and reloads to the live state; Owen's tap is not recorded as an approval | FEAT-08.SPEC-001 |
| Unauthorized | Requesting identity is not a Primary Contact for the milestone's own client company, or the milestone is already "Approved" at the time of the attempt | None | Treated identically to the stale-attempt outcome from Owen's perspective when caused by a race (already approved by the time this request lands); if reached through any other path, no control was ever shown to begin with (FEAT-08.SPEC-006) | FEAT-08.SPEC-001 |
| Connectivity/session failure | Connectivity or an authenticated session is not present at the moment of the attempt | None | FEAT-08.SPEC-001 shows the offline "reconnect to approve" state; Owen's tap never appears to succeed | FEAT-08.SPEC-001 |
| Write failure (system error after checks pass) | All eligibility checks pass, but the write itself cannot be confirmed as committed | None -- the milestone is left in its pre-attempt state, never partially written | FEAT-08.SPEC-001 shows an error banner with a Retry option; retrying re-runs this automation from step 3, so a retried attempt is never double-recorded | FEAT-08.SPEC-001 |

## Data Model

**Reads:** Milestone -- `status`, `price`, `no_separate_charge`, current Deliverable reference and its status, for the staleness comparison. Client Contact -- role and client-company reference, to establish Owen's identity and authorization.
**Creates:** None.
**Updates:** Milestone -- `status`, `approved_at`, `approved_by`, written together in one atomic operation on a successful outcome only.
**Deletes:** None.

## Business Rules

- Exactly-once and immutability (FEAT-08.SPEC-006): a confirmed write is never issued a second time against the same pre-approval state; `approved_at` and `approved_by` are never altered by any subsequent action other than a future reopen-then-approve cycle.
- Reject-with-refresh, never last-write-wins or merge, on the Milestone entity (feature-dependency-map.md, Milestone Contention note): the comparison in step 5 is authoritative, and any mismatch refuses the write rather than attempting to reconcile it.
- Approval requires connectivity (FEAT-08.SPEC-006): this automation never queues an approval for later delivery -- a request received without a confirmed connection and session is treated as a connectivity failure, not a pending approval.
- A failed approval action is retried by Owen without double-recording (product-features.md, States: Error): because the write is a single atomic, idempotent-by-precondition operation, retrying after a write failure re-evaluates eligibility fresh and cannot produce two approval records for the same cycle.
- The next-invoice trigger and the confirmation notification fire only after the write is confirmed committed (step 8) -- never optimistically before commit, so a failed or stale attempt can never issue an invoice or send a confirmation.

## Edge Cases

- **Owen retries after a write-failure error banner** -- The retry re-runs the full check from step 3; if the milestone is still eligible, the retry succeeds and records exactly one approval; if another approval was recorded in the meantime, the retry is refused as a stale attempt.
- **The milestone's deliverable is removed by Nadia between screen load and Owen's tap** -- Refused as a stale attempt per step 6; Owen sees the refreshed milestone, which has no reviewable deliverable and no Approve control.
- **The milestone is re-priced by Nadia between screen load and Owen's tap, with no status change** -- Still refused as a stale attempt: any live-state mismatch from what Owen was shown refuses the write, not only a status change, so his approval is never recorded against pricing he did not actually see.
- **Connectivity drops between Owen's tap and the write's confirmation** -- Treated as a write failure (step 8): the automation does not assume success without confirmation, and Owen sees the error/offline state rather than a false "Approved" confirmation.
- **Concurrent trigger firing -- Owen taps Approve on two of his own sessions at effectively the same time** -- Both requests reach step 4-6; whichever commits first wins, moving `status` out of the eligible range. The second request's comparison in step 6 then finds a mismatch (status is already "Approved") and is refused as a stale attempt, showing the now-Approved milestone. Exactly one approval is ever recorded.
- **A second Approve request arrives while the first is still mid-write (trigger fires while a previous run is in flight)** -- The second request's read of live state (step 4) either observes the pre-write state (and, if it then wins the race to write, is caught by the same single-writer commit guarantee so only one write ultimately commits) or observes the already-committed state (and is refused as stale). No interleaving of the two requests can produce two committed approvals or a partially-written record.
- **The Activity Log write (step 10) fails after the Milestone write (step 7-8) already succeeded** -- Non-blocking: the approval itself stands (Owen sees "Approved on {date}," the next invoice still fires); the audit-trail write is retried by FEAT-13's own handling, per the pattern used elsewhere in this product for a downstream logging failure that must never unwind an already-committed evidentiary record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Triggered by (inbound) | Owen's Approve tap fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Defines the authorization, eligibility, exactly-once, immutability, and concurrency rules this automation enforces |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Affects (outbound) | Returns the success, stale, unauthorized, connectivity, or failure outcome for display |
| FEAT-08.SPEC-004 (Next-Invoice Trigger) | Affects (outbound) | Fired immediately after a confirmed successful approval write |
| FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification) | Affects (outbound) | Fired immediately after a confirmed successful approval write |
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | References (inbound) | This automation's staleness check reads the same Milestone state that FEAT-04.SPEC-003's edit-lock rule protects |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the approval event (XBR-05) |

## Analytics and Success Signals

- **milestone_approval_recorded** (time from deliverable-ready to approval, milestone reference) -- supports success-metrics.md: "Milestone Approval Turnaround"
- **milestone_approval_refused_stale** (reason: re-priced / deliverable-removed / already-approved) -- N/A -- no Stage 2 metric measures refused attempts directly; retained so a client-visible refusal remains observable rather than a silent dead end, since a repeatedly refused approval would otherwise look like a slow turnaround with no diagnosable cause.
- **milestone_approval_write_failed** (retry_count) -- N/A -- no Stage 2 metric covers infrastructure write failures; retained to distinguish a genuinely slow approval from one blocked by a technical failure, which "Milestone Approval Turnaround" alone cannot tell apart.

## Acceptance Criteria

**FEAT-08.SPEC-003-AC-01:** Given Owen taps Approve on a milestone whose live state matches what his screen loaded, when the write commits, then `status`, `approved_at`, and `approved_by` are all set together and FEAT-08.SPEC-001 shows "Approved on {date}."

**FEAT-08.SPEC-003-AC-02:** Given a successful approval write, when it commits, then FEAT-08.SPEC-004 (Next-Invoice Trigger) and FEAT-08.SPEC-007 (Confirmation Notification) both fire.

**FEAT-08.SPEC-003-AC-03:** Given a successful approval write, when it commits, then an Activity Log entry is fired to FEAT-13 with event type "milestone approved," Owen as actor, and the Milestone as the affected record.

**FEAT-08.SPEC-003-AC-04:** Given Nadia removed the milestone's deliverable after Owen's screen loaded, when Owen taps Approve, then the attempt is refused as stale and Owen is shown the refreshed milestone with no write to `status`, `approved_at`, or `approved_by`.

**FEAT-08.SPEC-003-AC-05:** Given Nadia re-priced the milestone after Owen's screen loaded, with no status change, when Owen taps Approve, then the attempt is still refused as stale.

**FEAT-08.SPEC-003-AC-06:** Given the milestone is already "Approved" by the time Owen's request reaches this automation, when the comparison in step 6 runs, then the attempt is refused and Owen is shown the current "Approved on {date}" state.

**FEAT-08.SPEC-003-AC-07:** Given Owen has no connectivity at the moment he taps Approve, when the request is evaluated, then it is refused as a connectivity failure and no write is attempted.

**FEAT-08.SPEC-003-AC-08:** Given all eligibility checks pass but the write cannot be confirmed as committed, when Owen sees the resulting error, then he can retry, and the retry re-evaluates eligibility fresh rather than assuming the prior attempt partially succeeded.

**FEAT-08.SPEC-003-AC-09:** Given Owen taps Approve twice in rapid succession from the same session, when the first tap's write is already committing, then the second tap's request is refused as stale, showing the now-Approved milestone, and only one approval is ever recorded.

**FEAT-08.SPEC-003-AC-10:** Given Owen approves the same milestone from two of his own sessions at effectively the same time, when both requests are processed, then exactly one write commits and the other is refused as stale.

**FEAT-08.SPEC-003-AC-11:** Given a second Approve request arrives while a first request's write is still in flight, when both are processed, then no interleaving produces two committed approvals or a partially written record.

**FEAT-08.SPEC-003-AC-12:** Given the Milestone write for an approval succeeds but the subsequent Activity Log write to FEAT-13 fails, when this is observed, then the approval itself still stands (Owen still sees "Approved on {date}," the invoice trigger still fires), and the audit-trail write is retried by FEAT-13's own handling.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (recorded, stale, unauthorized, connectivity failure, write failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Next-Invoice Trigger

## Overview

**Name:** Next-Invoice Trigger
**ID:** FEAT-08.SPEC-004
**Type:** Automation
**Purpose:** On a successful milestone approval, automatically hands off to invoice generation for the next invoice in the payment schedule, with no freelancer action, reading the schedule exactly as it stood at the moment of approval.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Firing immediately and only on a confirmed successful approval write
- Resolving which invoice, if any, the approved milestone's `payment_trigger` maps to in the Payment Schedule as it stood at the moment of approval
- Handing that resolved trigger off to invoice generation, with no freelancer action required
- Defining the no-action outcome for a milestone with no invoicing consequence

**Non-Goals:**
- Creating, numbering, formatting, or sending the invoice itself -- owned entirely by Invoice Generation & Sending (FEAT-09.SPEC-004, Automatic Invoice Generation), per XBR-02; this automation only fires the trigger and hands off the resolved schedule reference.
- Deciding whether the approval itself is valid or should be recorded -- owned by FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard); this automation only runs after that spec confirms a successful write.
- Correcting, cancelling, or crediting an invoice already issued by this trigger -- excluded per the Brief's Non-Goals: once this trigger fires, correcting the resulting invoice is FEAT-09's responsibility alone (credit note or new invoice, XBR-04); this feature has no invoice-editing capability of its own.
- Adjusting the Payment Schedule itself -- owned by FEAT-04 (Milestone & Payment Schedule Setup); this automation only reads the schedule as it stood at the moment of approval and never writes to it.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Milestone approval successfully recorded | FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Fires immediately and only after FEAT-08.SPEC-003 confirms the approval write committed | The approved Milestone's reference, its `payment_trigger` value, and its Project reference (to resolve the owning Payment Schedule) |

## Processing Logic

1. Receive the confirmed-approval event from FEAT-08.SPEC-003, carrying the approved Milestone's reference and its `payment_trigger` value at the moment of approval.
2. Read the Project's Payment Schedule exactly as it currently stands (which, per the Payment Schedule Contention rule, is guaranteed to be the schedule as it stood at the moment of approval, since a later Nadia edit applies only to future triggers and never retroactively).
3. Determine whether the approved milestone's `payment_trigger` marks it as issuing an invoice on approval, per the schedule's structure.
4. If it does, hand off to FEAT-09.SPEC-004 (Automatic Invoice Generation) with the Milestone reference, its price (or the schedule's relevant amount), the Project, and the triggering event type "milestone approval."
5. If it does not (the milestone carries `no_separate_charge` or its `payment_trigger` is not set to invoice on approval), take no invoicing action and record the no-action outcome.
6. Return the outcome (handed off, or no action) for observability; this automation does not itself confirm invoice creation, numbering, or sending -- those confirmations belong to FEAT-09.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Invoice generation triggered | The approved milestone's `payment_trigger` marks it as an invoice-on-approval milestone in the schedule as it stood at approval | None made directly by this automation -- FEAT-09.SPEC-004 owns the Invoice record itself | Nadia and Owen see the new invoice appear and its own notification arrive shortly after approval, through FEAT-09's own screens and notifications; no separate confirmation from this automation is shown | FEAT-09.SPEC-004 |
| No invoicing action (milestone carries no payment trigger) | The approved milestone's `payment_trigger` is not set to invoice on approval, or `no_separate_charge` is set | None | Nothing is shown -- approving a milestone with no payment trigger completes exactly like any other approval, with no invoice-related feedback of any kind | None -- silent no-action, consistent with the milestone carrying no charge |
| Hand-off failure (the automation cannot resolve or reach invoice generation) | A system error prevents resolving the schedule or reaching FEAT-09.SPEC-004 after the approval has already been recorded | None to the Milestone or Invoice; the approval itself is never rolled back | The approval stands ("Approved on {date}" remains visible to Owen); the missing invoice surfaces to Nadia as a delivery/processing warning on the project (consistent with XBR-30's pattern for a downstream failure that must never unwind an already-recorded evidentiary approval), and the hand-off is retried automatically | FEAT-08.SPEC-001 (approval display unaffected), FEAT-09 (retried hand-off) |

## Data Model

**Reads:** Milestone -- `payment_trigger`, `price`, `no_separate_charge`, Project reference. Payment Schedule -- `structure`, `deposit_amount`, `completion_amount`, as they stand at the moment of approval.
**Creates:** None -- Invoice creation belongs entirely to FEAT-09.SPEC-004.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-02: Approving a milestone automatically generates and sends the next invoice in the payment schedule, with no freelancer action -- this automation is the FEAT-08 half of that rule; FEAT-09 owns invoice creation itself.
- The schedule used is the schedule as it stood at the moment of approval (feature-dependency-map.md, Payment Schedule Contention note): a schedule edit Nadia saves afterward is dated and applies only to later triggers, never retroactively to this one.
- This automation fires unconditionally on every confirmed successful approval -- it is never skipped, delayed, or made optional by a freelancer setting, per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior rather than a configurable workflow.
- A milestone with no payment trigger produces no invoice and no error -- silence is the correct behavior for a milestone that was never meant to bill separately.
- A hand-off failure never reverses or delays display of the already-recorded approval (XBR-04): the approval's immutability and evidentiary status do not depend on whether the downstream invoice succeeds.

## Edge Cases

- **The approved milestone carries `no_separate_charge`** -- No invoice is generated; the approval completes with no invoicing feedback of any kind.
- **The Payment Schedule has no payment trigger configured at all at the time of approval** -- No invoice is generated; this is a pre-existing condition surfaced earlier by FEAT-04.SPEC-003's non-blocking payment-trigger indicator, not a new failure introduced here.
- **Nadia edits the Payment Schedule at the exact moment Owen's approval is being recorded** -- Per the Payment Schedule Contention rule, this trigger resolves against the schedule as it stood at approval; Nadia's concurrent edit is dated and takes effect only for triggers that fire after her edit commits, never for this one.
- **Concurrent trigger firing -- two different milestones in the same project are approved by Owen at effectively the same time** -- Each approval is a separate, independently confirmed write (FEAT-08.SPEC-003's exactly-once guarantee is per-milestone); this automation runs once per approval and hands off two independent invoice-generation requests to FEAT-09, which processes them independently.
- **This trigger fires while a previous run for a different milestone is still in flight** -- Runs for different milestones proceed independently; there is no shared state between them beyond both reading the same Project's Payment Schedule, which is read-only from this automation's perspective and therefore never contended between the two runs.
- **FEAT-09.SPEC-004 is unreachable at the moment this automation attempts the hand-off** -- The approval remains recorded and visible; the hand-off is retried automatically, and if retries are exhausted the missing invoice surfaces to Nadia as a project-level warning rather than being silently lost.
- **The milestone is later reopened and re-approved** -- Each approval independently re-evaluates the schedule as it stands at that moment and, if the milestone's `payment_trigger` still applies, hands off again; this automation carries no memory of a prior hand-off from an earlier approval cycle on the same milestone, since a genuinely new approval is a new triggering event.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Triggered by (inbound) | A confirmed successful approval write fires this automation |
| FEAT-09.SPEC-004 (Automatic Invoice Generation) | Affects (outbound) | Hands off the resolved trigger and milestone/schedule data for invoice creation, numbering, and sending (XBR-02) |
| FEAT-04 (Milestone & Payment Schedule Setup) | References (inbound) | Reads the Payment Schedule and the milestone's `payment_trigger`, as they stood at the moment of approval |

## Analytics and Success Signals

- **milestone_invoice_auto_generated** (milestone reference, invoice trigger type) -- supports success-metrics.md: "Milestone Approval Turnaround" (a fired trigger with no delay is the visible half of a fast approval-to-billing loop; a stalled or failed hand-off would otherwise look identical to a slow approval from the freelancer's perspective)
- **milestone_invoice_trigger_skipped** (reason: no_payment_trigger / no_separate_charge) -- N/A -- no Stage 2 metric measures milestones with no invoicing consequence; retained so a milestone approval with no invoice is distinguishable from a hand-off failure.
- **milestone_invoice_handoff_failed** (retry_count) -- N/A -- no Stage 2 metric covers infrastructure hand-off failures directly; retained because "Invoice Auto-Generation Accuracy" (product-features.md, FEAT-09) measures correctness of invoices that were generated, not invoices that failed to generate at all -- this event is this feature's only signal of that failure mode.

## Acceptance Criteria

**FEAT-08.SPEC-004-AC-01:** Given Owen's approval of a milestone with `payment_trigger` set to invoice on approval is successfully recorded, when FEAT-08.SPEC-003 confirms the write, then this automation hands off to FEAT-09.SPEC-004 immediately, with no action required from Nadia.

**FEAT-08.SPEC-004-AC-02:** Given the approved milestone carries `no_separate_charge`, when the approval is recorded, then no invoice hand-off occurs and no invoice-related feedback appears anywhere.

**FEAT-08.SPEC-004-AC-03:** Given the Payment Schedule has no configured payment trigger at all, when a milestone under it is approved, then no invoice hand-off occurs.

**FEAT-08.SPEC-004-AC-04:** Given Nadia edits the Payment Schedule at the same moment Owen's approval is being recorded, when this automation resolves the trigger, then it uses the schedule exactly as it stood at the moment of approval, not Nadia's concurrent edit.

**FEAT-08.SPEC-004-AC-05:** Given a schedule edit Nadia saved earlier is dated after the approval being processed, when this automation runs, then it is unaffected by that later edit, consistent with the non-retroactive rule.

**FEAT-08.SPEC-004-AC-06:** Given Owen approves two different milestones in the same project at effectively the same time, when both approvals are confirmed, then this automation fires once per approval and hands off two independent invoice-generation requests.

**FEAT-08.SPEC-004-AC-07:** Given FEAT-09.SPEC-004 is unreachable at the moment of hand-off, when the failure occurs, then the approval remains recorded and visible to Owen, the hand-off is retried automatically, and Nadia sees a project-level warning if retries are exhausted.

**FEAT-08.SPEC-004-AC-08:** Given a milestone is reopened and re-approved, when the second approval is confirmed, then this automation re-evaluates the schedule fresh and hands off again if the milestone's `payment_trigger` still applies, independent of any hand-off from the first approval cycle.

**FEAT-08.SPEC-004-AC-09:** Given a milestone's `payment_trigger` marks it to invoice on approval, when the automation completes its hand-off, then the milestone_invoice_auto_generated event is emitted with the milestone reference and trigger type.

**FEAT-08.SPEC-004-AC-10:** Given this automation runs for two milestones approved at effectively the same time, when both runs read the Project's Payment Schedule, then neither run's read is contended by the other, since the schedule is read-only from this automation's perspective.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (triggered, no action, hand-off failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Reopen Recording

## Overview

**Name:** Reopen Recording
**ID:** FEAT-08.SPEC-005
**Type:** Automation
**Purpose:** Writes Nadia's reopen of an approved milestone as a distinct, logged event, resetting the milestone to Reopened status while leaving the original approval record's `approved_at` and `approved_by` fields untouched.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- Re-validating Nadia's authorization and the milestone's eligibility (status = Approved) at the moment of the Reopen attempt
- Atomically writing `status` -> "Reopened" without altering `approved_at` or `approved_by`
- Firing the Activity Log entry for the reopen event, which is how the event stays non-silent
- Making the milestone eligible again for Nadia's edits (FEAT-04) once reopened

**Non-Goals:**
- Rendering the Reopen control and the confirmation the freelancer sees before committing to reopen -- owned by FEAT-08.SPEC-002 (Milestone Reopen Screen), which this automation reports its outcome back to.
- Defining who may reopen and under what milestone state -- owned by FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules), which this automation enforces rather than restates.
- Sending any notification about the reopen -- the Brief's Communications field names no separate email for reopen; the append-only Activity Log entry this automation fires is how the event stays "non-silent" (Key Capabilities), not a notification.
- Reversing or cancelling an invoice that was already issued by the approval being reopened -- excluded per the Brief's Non-Goals: correcting or cancelling that invoice is owned entirely by FEAT-09 via a credit note or new invoice (XBR-04); this automation never touches the Invoice entity.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms reopen | FEAT-08.SPEC-002 (Milestone Reopen Screen) | Fires when Nadia confirms the reopen action on an approved milestone | The milestone reference, Nadia's authenticated identity, the current timestamp |

## Processing Logic

1. Receive the reopen request from FEAT-08.SPEC-002, carrying the milestone reference and Nadia's authenticated identity.
2. Confirm connectivity and an authenticated session are present; if either is missing, stop and return the connectivity/session failure outcome without touching the milestone record.
3. Re-check authorization per FEAT-08.SPEC-006: confirm the requesting identity is Nadia (the Freelancer) for this milestone's own account.
4. Read the milestone's current, live status.
5. If the live status is not "Approved," stop and return the already-changed outcome -- there is nothing to reopen.
6. If the live status is "Approved," write `status` -> "Reopened," leaving `approved_at` and `approved_by` exactly as they are.
7. Confirm the write committed successfully before reporting success -- an unconfirmed write is treated as a failure.
8. On a confirmed successful write, fire the append-only Activity Log entry to FEAT-13 (event type "milestone reopened," actor Nadia, the timestamp from step 6, the Milestone as the affected record) per XBR-05.
9. Return the outcome (reopened, already-changed, or failure) to FEAT-08.SPEC-002 for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Reopen recorded | Live status is "Approved" at the moment of the confirmed request | Milestone `status` -> "Reopened"; `approved_at` and `approved_by` unchanged | FEAT-08.SPEC-002 shows the milestone as Reopened; FEAT-08.SPEC-001 now shows the Approve control again instead of the "Approved on {date}" marker; FEAT-04.SPEC-001 shows the milestone as editable again | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-04.SPEC-001, FEAT-13 |
| Already changed (status is no longer Approved) | The milestone's status changed (e.g., a fresh approval cycle, or an earlier reopen already recorded) since Nadia's screen last loaded | None | FEAT-08.SPEC-002 shows a refresh: "This milestone's status has changed. Here's the current state." and reloads to the live status | FEAT-08.SPEC-002 |
| Connectivity/session failure | Connectivity or an authenticated session is not present at the moment of the attempt | None | FEAT-08.SPEC-002 shows a connectivity error; Nadia's confirmation never appears to succeed | FEAT-08.SPEC-002 |
| Write failure (system error after checks pass) | Eligibility checks pass but the write cannot be confirmed as committed | None -- the milestone is left in its pre-attempt "Approved" state | FEAT-08.SPEC-002 shows an error banner with a Retry option; retrying re-runs this automation from step 3 | FEAT-08.SPEC-002 |

## Data Model

**Reads:** Milestone -- `status`. Client Contact / Freelancer Account -- identity, to confirm Nadia is the requester.
**Creates:** None (the reopen event itself is captured as an Activity Log entry by FEAT-13, not as a new record on the Milestone entity).
**Updates:** Milestone -- `status` only, written on a successful outcome.
**Deletes:** None.

## Business Rules

- Reopen never rewrites approval history (FEAT-08.SPEC-006): this automation writes `status` only; `approved_at` and `approved_by` are never cleared, edited, or reset by a reopen.
- Reopen is Nadia-only (Key Capabilities: "Reopen (freelancer only)"): no client contact role can trigger this automation under any condition.
- Reopen requires the milestone to currently be "Approved" (FEAT-08.SPEC-006): there is no reopen of a milestone that was never approved, and no double-reopen of one already Reopened.
- A reopen is itself a logged, non-silent event (XBR-05, Key Capabilities): the Activity Log entry in step 8 is the mechanism that satisfies "never a silent edit" -- there is no separate reopen confirmation sent to any client contact, since reopening is a freelancer-side correction, not a client-facing event.
- Once reopened, the milestone becomes eligible again for Nadia's edits in FEAT-04 (feature-dependency-map.md, Cross-Feature Touchpoints): her edit attempts on the still-Approved milestone were refused until this automation's write commits.

## Edge Cases

- **Nadia confirms reopen twice in rapid succession (double-submit)** -- The first confirmation's write begins moving `status` out of "Approved" immediately; the second is evaluated against the now-"Reopened" state in step 5 and returns the already-changed outcome, never recording a second reopen event.
- **The milestone is re-approved (a new approval cycle) between Nadia's screen load and her reopen confirmation** -- Refused as already-changed only if status moved away from "Approved" in the interim; if it is still "Approved" (just a fresh approval cycle with new `approved_at`/`approved_by`), the reopen proceeds normally against the current approval.
- **Connectivity drops between Nadia's confirmation and the write's confirmation** -- Treated as a write failure; the milestone remains "Approved" and Nadia sees the connectivity/error state rather than an ambiguous "maybe reopened" state.
- **Concurrent trigger firing -- Nadia reopens the same milestone from two of her own sessions at effectively the same time** -- Whichever request's write commits first wins; the second request's check in step 5 then finds the milestone already "Reopened" and returns the already-changed outcome. Exactly one reopen event is ever recorded.
- **A reopen request arrives while a previous reopen run for the same milestone is still in flight** -- The second request's read of live status either observes the pre-write "Approved" state (and, if it also attempts to write, is subject to the same single-writer commit guarantee so only one write ultimately commits) or observes the already-"Reopened" state (and returns already-changed). No interleaving produces two reopen events for one approval cycle.
- **The Activity Log write (step 8) fails after the Milestone write (step 6-7) already succeeded** -- Non-blocking: the reopen itself stands (the milestone shows "Reopened," FEAT-04 edits become available); the audit-trail write is retried by FEAT-13's own handling, since the reopen's non-silence depends on that entry eventually landing, but a transient logging failure must never unwind an already-committed status change.
- **Nadia reopens a milestone and immediately attempts to edit it before the reopen's write is confirmed** -- FEAT-04.SPEC-003's edit-lock rule still reads "Approved" until this automation's write commits; her edit is refused until the reopen is confirmed, after which the same edit attempt succeeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Milestone Reopen Screen) | Triggered by (inbound) | Nadia's confirmed reopen action fires this automation |
| FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules) | References (inbound) | Defines the Reopen authorization and eligibility rules this automation enforces |
| FEAT-08.SPEC-002 (Milestone Reopen Screen) | Affects (outbound) | Returns the reopened, already-changed, connectivity, or failure outcome for display |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Affects (outbound) | A successful reopen returns the milestone to an unapproved state, re-enabling the Approve control there |
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | Affects (outbound) | A successful reopen lifts the edit-lock, making Nadia's edits on this milestone allowed again |
| FEAT-13 (Immutable Activity & Audit Trail) | Triggers (outbound) | Fires the append-only trail entry for the reopen event (XBR-05) |

## Analytics and Success Signals

- **milestone_reopened_by_freelancer** (milestone reference, time since original approval) -- N/A -- no Stage 2 metric measures reopen frequency directly; retained because the Brief's Key Capabilities treat "a logged, non-silent way to reopen" as a first-class product guarantee, and this event is the only signal that the guarantee is being exercised as intended rather than never used or, at the other extreme, used so often it signals an approval-quality problem worth the founder's attention.
- **milestone_reopen_refused_already_changed** (milestone reference) -- N/A -- no Stage 2 metric measures refused reopen attempts; retained for the same observability reason as the analogous refused-approval event in FEAT-08.SPEC-003.

## Acceptance Criteria

**FEAT-08.SPEC-005-AC-01:** Given Nadia confirms reopen on a milestone whose live status is "Approved," when the write commits, then `status` becomes "Reopened" and `approved_at`/`approved_by` remain unchanged.

**FEAT-08.SPEC-005-AC-02:** Given a successful reopen write, when it commits, then an Activity Log entry is fired to FEAT-13 with event type "milestone reopened," Nadia as actor, and the Milestone as the affected record.

**FEAT-08.SPEC-005-AC-03:** Given a successful reopen, when Owen next opens FEAT-08.SPEC-001 for that milestone, then he sees the Approve control again instead of "Approved on {date}."

**FEAT-08.SPEC-005-AC-04:** Given a successful reopen, when Nadia opens FEAT-04.SPEC-001 for that milestone, then its edit controls are enabled again.

**FEAT-08.SPEC-005-AC-05:** Given the milestone's status is no longer "Approved" by the time Nadia's confirmed request is processed, when the check in step 5 runs, then the attempt returns the already-changed outcome with no write.

**FEAT-08.SPEC-005-AC-06:** Given Nadia has no connectivity at the moment she confirms reopen, when the request is evaluated, then it is refused as a connectivity failure and no write is attempted.

**FEAT-08.SPEC-005-AC-07:** Given all eligibility checks pass but the write cannot be confirmed as committed, when Nadia sees the resulting error, then she can retry, and the retry re-evaluates eligibility fresh.

**FEAT-08.SPEC-005-AC-08:** Given Nadia confirms reopen twice in rapid succession, when the first confirmation's write is already committing, then the second is refused as already-changed and only one reopen event is ever recorded.

**FEAT-08.SPEC-005-AC-09:** Given Nadia reopens the same milestone from two of her own sessions at effectively the same time, when both requests are processed, then exactly one write commits and the other returns the already-changed outcome.

**FEAT-08.SPEC-005-AC-10:** Given the Milestone write for a reopen succeeds but the subsequent Activity Log write to FEAT-13 fails, when this is observed, then the reopen itself still stands, and the audit-trail write is retried by FEAT-13's own handling.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (reopened, already-changed, connectivity failure, write failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Approval Authorization & Eligibility Rules

## Overview

**Name:** Approval Authorization & Eligibility Rules
**ID:** FEAT-08.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may approve or reopen a milestone, the milestone-state and load preconditions that gate each action, the exactly-once and immutability guarantees on an approval, and the concurrency and connectivity conditions that decide whether an attempt is honored or refused.
**Parent Feature:** FEAT-08 -- Milestone Approval
**Governed Entity:** Milestone

## Scope and Non-Goals

**In Scope:**
- Authorization rules for the Approve and Reopen actions on Milestone, per role in the Access Matrix
- Eligibility preconditions that gate Approve (milestone state, deliverable state, comment-thread load) and Reopen (milestone state)
- The exactly-once and immutability guarantees on `approved_at` and `approved_by`
- The concurrency (reject-with-refresh) and connectivity rules that govern a contested or interrupted approval attempt
- Derivation of `status`, `approved_at`, and `approved_by` on a successful approval or reopen

**Non-Goals:**
- Field validation and authorization rules for Milestone's descriptive and pricing fields (name, order, price, no_separate_charge, payment_trigger, target_date) and for Create/View/Edit/Remove actions -- owned by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules); this spec only reads `status` as a precondition and never re-defines those rules.
- The atomic write mechanics and refresh behavior of recording an approval -- specified by FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard), which implements the rules this spec defines.
- The atomic write mechanics of recording a reopen -- specified by FEAT-08.SPEC-005 (Reopen Recording), which implements the reopen rules this spec defines.
- A multi-approver or staged sign-off workflow -- excluded per scope-boundaries.md (SC-02): the persona set defines exactly two client-contact roles (Primary, Reviewer) with no additional client-side tiers, so there is no second approver to route a sign-off through.
- A configurable approval workflow or conditional routing -- excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior (approve triggers the next invoice) rather than a configurable workflow or automation builder.

## Governed Entity

**Entity:** Milestone
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The milestone's name |
| order | number | Its position within the project's sequence |
| price | number | The milestone's price, when it carries a separate charge |
| no_separate_charge | boolean | Flag marking the milestone as carrying no separate charge (mutually exclusive with price) |
| payment_trigger | enum | Whether this milestone's approval issues an invoice, per the Payment Schedule |
| target_date | date | Optional date shown in each viewer's own time zone |
| status | enum | Defined, Deliverable Uploaded, Approved, Reopened |
| approved_at | date/time | Timestamp of the current approval cycle, written once per approval and never otherwise altered |
| approved_by | text | Identity of the contact who recorded the current approval, written once per approval and never otherwise altered |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-08.SPEC-001 | Milestone Review & Approval Screen | On screen entry (whether the Approve control renders at all, per role) and on every Approve attempt (eligibility and connectivity preconditions) |
| FEAT-08.SPEC-002 | Milestone Reopen Screen | On screen entry (whether the Reopen control renders, per role) and on every Reopen attempt (milestone-state precondition) |
| FEAT-08.SPEC-003 | Approval Recording & Concurrency Guard | Reads this spec's exactly-once, immutability, and reject-with-refresh rules to decide whether an Approve attempt is recorded or refused |
| FEAT-08.SPEC-005 | Reopen Recording | Reads this spec's Reopen eligibility and immutability rules to decide whether a Reopen attempt is recorded |
| FEAT-04.SPEC-003 | Milestone & Schedule Validation and Edit Rules | Reads this spec's Milestone edit-lock outcome (status = Approved) to refuse a concurrent edit or removal attempt by Nadia (XBR-10) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| order | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| price | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| no_separate_charge | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| payment_trigger | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| target_date | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| status | Must be "Deliverable Uploaded" or "Reopened" for an Approve attempt to be recorded; must be "Approved" for a Reopen attempt to be recorded. Any other status refuses the respective action. | Always | On Approve attempt (FEAT-08.SPEC-003); on Reopen attempt (FEAT-08.SPEC-005) | "This milestone can't be approved right now -- it may have already been approved or its deliverable removed. Refreshing to show the current state." (Approve) / "This milestone can only be reopened once it has been approved." (Reopen) | Yes |
| approved_at | Set exactly once per approval cycle by FEAT-08.SPEC-003 at the moment an approval is successfully recorded; never directly editable by any role, on any screen | Always | On every Approve attempt (write path only) | N/A -- no direct-entry field exists for this value | Yes (write-protected) |
| approved_by | Set exactly once per approval cycle by FEAT-08.SPEC-003 to the identity of the approving Client Contact; never directly editable by any role, on any screen | Always | On every Approve attempt (write path only) | N/A -- no direct-entry field exists for this value | Yes (write-protected) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Approval fields are set atomically or not at all | status, approved_at, approved_by | On a successful Approve, all three change together in one write: status -> "Approved", approved_at -> current timestamp, approved_by -> the approving contact's identity. A refused attempt (state mismatch, concurrency conflict, or connectivity loss) leaves all three fields exactly as they were -- there is no partial write. | N/A -- enforced as an atomic operation, not surfaced as a field error |
| Reopen never rewrites approval history | status, approved_at, approved_by | A successful Reopen changes status -> "Reopened" only; it never clears or edits approved_at or approved_by. Those fields continue to display the most recent approval's data until a fresh Approve overwrites them together (per the rule above). | N/A -- enforced as a scoped write, not surfaced as a field error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Approve milestone | Owen (Client Primary Contact) | Milestone belongs to Owen's own client company (Own-only, XBR-09), status is "Deliverable Uploaded" or "Reopened" (not already "Approved"), the milestone's current Deliverable and its comment thread have fully loaded, and Owen has connectivity at the moment he acts | The Approve control is disabled while any precondition (load, connectivity) is unmet, with the specific reason shown per FEAT-08.SPEC-001's States; if the milestone is already Approved, the control is replaced entirely by the "Approved on {date}" marker -- there is no separate denial message because the action is not offered |
| Approve milestone | Priya (Client Reviewer Contact) | Never | The Approve control is not rendered for Priya at all; she sees the same milestone status as Owen with no path to an approve action, per the Access Matrix's Reviewer entitlement |
| Approve milestone | Nadia (Freelancer) | Never | Nadia has no approval surface for her own client's milestone -- the action belongs exclusively to the client's Primary Contact (Key Capabilities); no control is shown on any freelancer-facing screen |
| Approve milestone | Dana (Support Operator) | Never | Dana's read-only support session (FEAT-31) never renders the Approve control; if a support-session render path were reached in error, the control would show disabled with "Support access is read-only." |
| Reopen milestone | Nadia (Freelancer) | Milestone status is "Approved" | The Reopen control is not shown while status is "Defined," "Deliverable Uploaded," or already "Reopened" -- there is nothing to reopen; a direct attempt outside the "Approved" state shows "This milestone can only be reopened once it has been approved." |
| Reopen milestone | Owen, Priya (Client Contacts) | Never | No reopen surface exists in the client portal for either role, per the Key Capability "Reopen (freelancer only)" -- the action is never offered to a client contact under any circumstance |
| Reopen milestone | Dana (Support Operator) | Never | Dana's read-only support session never renders the Reopen control; if reached in error, the control shows disabled with "Support access is read-only." |
| View approval status ("Approved on {date}" marker, or current pending state) | Nadia (Freelancer) | Always, across all of her own projects | -- |
| View approval status | Owen, Priya (Client Contacts) | Own-only -- their own client company's projects (XBR-09) | An out-of-scope milestone link is never reachable; it shows a plain explanation and a fresh-link option, never another company's data |
| View approval status | Dana (Support Operator) | Always, inside a logged, read-only support session (FEAT-31) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Derived to "Approved" when FEAT-08.SPEC-003 successfully records an approval | On a successful Approve action only | No -- always system-derived from the recorded outcome |
| status | Derived to "Reopened" when FEAT-08.SPEC-005 successfully records a reopen | On a successful Reopen action only | No -- always system-derived from the recorded outcome |
| approved_at | Set to the current timestamp at the moment the approval write commits | On a successful Approve action only | No -- never user-entered, never editable afterward |
| approved_by | Set to the identity (name and Client Contact reference) of the approving contact, resolved from the authenticated session at the moment the approval write commits | On a successful Approve action only | No -- never user-entered, never editable afterward |

## Business Rules

- **Exactly-once per approval cycle (XBR-04):** A single successful Approve action fully consumes the eligibility window for that cycle -- `status` moves out of "Deliverable Uploaded"/"Reopened" the instant the write commits, so no second Approve attempt against the same pre-approval state can ever be recorded; a retried or double-submitted attempt is refused by FEAT-08.SPEC-003's atomic check-and-write, never double-recorded.
- **Immutability of the current approval record (ASMP-25, XBR-04):** Once set, `approved_at` and `approved_by` are never altered, deleted, or reassigned by any role on any screen -- not by Owen, not by Nadia, not by Dana. The only way the milestone's approval state changes at all is a new Reopen-then-Approve cycle, which is itself a new, separately logged event (FEAT-13, XBR-05), never an edit to the prior one.
- **Reopen is a distinct event, never a silent edit:** A reopen changes only `status`; it does not retroactively alter the approval record it supersedes for display. The full, permanent history of every approval and reopen -- including a milestone approved, reopened, and approved again -- lives in the immutable Activity Log (FEAT-13); the Milestone entity's own `approved_at`/`approved_by` fields reflect only the current cycle's most recent approval, consistent with the entity's "written once on approval, never altered" definition (feature-dependency-map.md, Entity: Milestone).
- **Concurrency is reject-with-refresh, never last-write-wins or merge (feature-dependency-map.md, Milestone Contention note):** An Approve attempt is evaluated against the milestone state Owen was shown when the screen loaded, not the state at click time. If Nadia re-priced, removed, or otherwise altered the milestone since Owen's screen loaded, the attempt is refused and Owen is shown the refreshed milestone instead of having his approval recorded against stale data.
- **Approval requires connectivity:** The Approve action can be initiated only while the client's device has connectivity; it is never queued or optimistically recorded offline, because an approval is a timestamped, permanent, evidentiary act that must reflect the moment it genuinely occurred (BRIEF.md, Constraints: record immutability).
- **Approval requires a fully loaded review context:** The Approve control is not actionable until the milestone's current Deliverable and its comment thread have both finished loading on FEAT-08.SPEC-001 -- an approval decided before the reviewer has seen everything they were shown to see is not a genuine decision.
- **XBR-10 (read-only reference):** A milestone that has been approved or invoiced cannot be removed or re-priced by Nadia; this spec's exactly-once and immutability rules are the FEAT-08 side of that same cross-feature guarantee -- FEAT-04.SPEC-003 owns the edit-lock rule itself.

## Edge Cases

- **Owen taps Approve twice in rapid succession (double-submit)** -- The first attempt's write begins moving `status` out of the eligible state immediately; the second attempt is evaluated against the now-current (or in-flight) state and is refused as if the milestone were already approved, never recorded as a second approval.
- **Owen's Approve attempt arrives after Nadia removed the milestone's only Deliverable in the meantime** -- Refused: the Deliverable precondition is no longer met. Owen is shown the refreshed milestone, which now has no reviewable deliverable and no Approve control.
- **Nadia attempts to reopen a milestone that was never approved (status "Defined" or "Deliverable Uploaded")** -- Refused with "This milestone can only be reopened once it has been approved." -- there is no prior approval to undo.
- **Nadia attempts to reopen a milestone that is already "Reopened"** -- Refused with the same message; a milestone can be reopened only once per completed approval cycle, and a second reopen has nothing further to do until a new approval occurs.
- **A milestone is reopened and re-approved, then reopened again** -- Each cycle is independent: the second approval's `approved_at`/`approved_by` overwrote the first atomically (per the Cross-Field Rule above), and the second reopen again changes only `status`. All four events remain individually visible and unaltered in the Activity Log (FEAT-13).
- **Owen loses connectivity between tapping Approve and the response returning** -- The action is never optimistically applied; if the write cannot be confirmed as committed, the milestone shows its prior state and Owen sees the offline/reconnect experience (FEAT-08.SPEC-001, States) rather than an ambiguous "maybe approved" state.
- **Two of Owen's own sessions (e.g., laptop and phone) both attempt to approve the same milestone at effectively the same time** -- Exactly-once holds regardless of which session sent the first commit: the first write to commit succeeds: the second is evaluated against the now-"Approved" state and refused with the same "already approved" experience Priya-adjacent stale attempts receive, showing the now-Approved milestone.
- **Dana's read-only session is open on a milestone at the exact moment Owen approves it** -- No conflict: Dana never has a write path, so her session simply reflects the updated state on next read; her view is never itself in contention.

## Acceptance Criteria

**FEAT-08.SPEC-006-AC-01:** Given Owen (Client Primary Contact) is viewing a milestone in his own client company's project with status "Deliverable Uploaded" and its deliverable and comment thread fully loaded, when he taps Approve while connected, then the approval is recorded and status moves to "Approved."

**FEAT-08.SPEC-006-AC-02:** Given Owen is viewing a milestone whose status is already "Approved," when the screen renders, then no Approve control is shown -- only the "Approved on {date}" marker.

**FEAT-08.SPEC-006-AC-03:** Given Priya (Client Reviewer Contact) opens the same milestone Owen can approve, when the screen renders, then no Approve control appears anywhere for her.

**FEAT-08.SPEC-006-AC-04:** Given Nadia (Freelancer) opens her own client's milestone review context, when she looks for an approve action, then none exists on any freelancer-facing screen.

**FEAT-08.SPEC-006-AC-05:** Given Dana (Support Operator) is in a logged, read-only support session viewing the milestone, when the screen renders, then no Approve control is shown.

**FEAT-08.SPEC-006-AC-06:** Given Nadia is viewing a milestone with status "Approved," when she looks for a Reopen action, then it is available and enabled.

**FEAT-08.SPEC-006-AC-07:** Given Nadia is viewing a milestone with status "Defined" or "Deliverable Uploaded," when she looks for a Reopen action, then none is shown, and a direct attempt shows "This milestone can only be reopened once it has been approved."

**FEAT-08.SPEC-006-AC-08:** Given Owen or Priya are in the client portal, when either looks for a way to reopen any milestone, then no such action exists anywhere in their portal.

**FEAT-08.SPEC-006-AC-09:** Given Dana is in a support session, when she looks for a Reopen action, then none is shown.

**FEAT-08.SPEC-006-AC-10:** Given Owen views his own client company's milestone, when the screen loads, then he can see its current status and history, since View approval status is always allowed for him on his own company's data.

**FEAT-08.SPEC-006-AC-11:** Given Priya follows a milestone link belonging to a different client company, when the link resolves, then she sees a plain explanation and a fresh-link option, never that company's data (XBR-09).

**FEAT-08.SPEC-006-AC-12:** Given Owen taps Approve twice in rapid succession, when the first tap's write is already in flight, then the second tap is refused as already-approved and no second approval is recorded.

**FEAT-08.SPEC-006-AC-13:** Given Nadia re-prices or removes the milestone's deliverable after Owen's approval screen loaded but before he taps Approve, when Owen taps Approve, then the attempt is refused and Owen is shown the refreshed milestone instead of having his approval recorded.

**FEAT-08.SPEC-006-AC-14:** Given the milestone's deliverable and comment thread have not yet finished loading on Owen's screen, when he looks for the Approve control, then it is disabled until loading completes.

**FEAT-08.SPEC-006-AC-15:** Given Owen has no connectivity, when he attempts to tap Approve, then the action does not proceed and he sees the offline "reconnect to approve" state rather than any success feedback.

**FEAT-08.SPEC-006-AC-16:** Given a milestone is successfully approved, when the write commits, then `status`, `approved_at`, and `approved_by` all change together in the same operation -- no partial state is ever observed.

**FEAT-08.SPEC-006-AC-17:** Given a milestone is successfully reopened, when the write commits, then only `status` changes to "Reopened"; `approved_at` and `approved_by` retain the values from the approval being reopened.

**FEAT-08.SPEC-006-AC-18:** Given a milestone was approved, reopened, and approved again, when the second approval commits, then `approved_at` and `approved_by` reflect the second approval, and both the first and second approval events remain individually visible, unaltered, in the Activity Log (FEAT-13).

**FEAT-08.SPEC-006-AC-19:** Given a milestone is already "Approved," when any role attempts to directly alter `approved_at` or `approved_by`, then no such control or path exists on any spec -- the fields are write-protected outside the approval and reopen automations.

**FEAT-08.SPEC-006-AC-20:** Given the same milestone receives two Approve attempts from two of Owen's own sessions at effectively the same moment, when the first commits, then the second is evaluated against the now-"Approved" state and refused, showing the now-Approved milestone.

**FEAT-08.SPEC-006-AC-21:** Given a milestone has been approved and invoiced, when Nadia attempts to edit or remove it from FEAT-04.SPEC-001, then FEAT-04.SPEC-003's edit-lock rule refuses the attempt, consistent with this spec's exactly-once and immutability guarantees (XBR-10).

**FEAT-08.SPEC-006-AC-22:** Given Dana has a read-only support session open on a milestone at the moment Owen approves it, when Dana next reads the milestone, then she sees the updated "Approved" state with no conflict or error of her own.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |



# Notification Spec: Milestone Approval Confirmation Notification

## Overview

**Name:** Milestone Approval Confirmation Notification
**ID:** FEAT-08.SPEC-007
**Type:** Notification
**Purpose:** Confirms to both Owen and Nadia, the moment an approval is recorded, exactly what happened -- so the client-side record and the freelancer-side record of the same event match from the start.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent to Owen and to Nadia when a milestone approval is successfully recorded
- Preference, retry, and expiry behavior for this confirmation

**Non-Goals:**
- Notifying anyone about the resulting invoice -- the auto-issued invoice sends its own notification, owned by Invoice Generation & Sending (FEAT-09); this spec covers only the approval confirmation itself (product-features.md, Communications).
- Notifying anyone about a reopen -- the Brief names no separate email for reopen; the reopen's non-silence is carried entirely by its Activity Log entry (FEAT-13), not a notification (Side-Effect Inventory).
- Notifying Priya -- she is not the approving contact and the Access Matrix limits invoice-and-approval-adjacent communications to the Primary Contact; a Reviewer receiving a financial-commitment confirmation would exceed her role's entitlement (XBR-08).
- In-app notification center delivery -- product-features.md's Communications field for this feature names only the email channel; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase), which is out of scope for this MVP-phase spec.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, when an approval is recorded | Owen's portal sessions are short and triggered by a specific email link (user-persona.md, Behavioral Context) -- he is not routinely inside the product to see a status change happen live; Nadia works from her own inbox and desktop workflow and needs the same confirmation as her defensible, timestamped record without having to reopen the project to check |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Milestone approval successfully recorded | FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Fires immediately and only after FEAT-08.SPEC-003 confirms the approval write committed | Milestone name and project, `approved_at`, `approved_by` (Owen's identity), Client `client_name`, Freelancer Account contact details for Nadia |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) and Nadia (the Freelancer), per the Brief's Communications field ("Confirmation email to Owen and Nadia when approval is recorded"). Both are entitled to this content under the Access Matrix: Owen approved the milestone himself, and Nadia's Full access to Milestones & Deliverables covers every event on her own account's milestones.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional record email, not an optional notification) |

This confirmation is a transactional email core to the record (XBR-30): it cannot be switched off by either recipient's notification preferences, in the same way a payment confirmation cannot -- it is the evidentiary echo of a permanent, timestamped decision, not a discretionary update.

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this confirmation is transactional and sends immediately regardless of the time of day for either recipient, consistent with the approval event itself being permanent and time-stamped at the moment it occurred.

## Content Definition

**Email (to Owen):**
- **Subject:** You approved {milestone_name}
- **Body:**
  Hi {owen_first_name},

  This confirms your approval of {milestone_name} on {project_name}, recorded on {approved_at_formatted}.

  This records your approval and issues the next invoice in the payment schedule, if one applies. You'll receive it separately if so.
- **CTA (button):** View milestone -- deep-links to FEAT-08.SPEC-001 (Milestone Review & Approval Screen) for this milestone

**Email (to Nadia):**
- **Subject:** {client_name} approved {milestone_name}
- **Body:**
  Hi {nadia_first_name},

  {approved_by_name} at {client_name} approved {milestone_name} on {project_name} on {approved_at_formatted}.

  If the payment schedule includes an invoice for this milestone, it has been generated and sent automatically -- no action needed from you.
- **CTA (button):** View milestone -- deep-links to the milestone's detail in her own project view (FEAT-01)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {milestone_name} | Milestone -- name | Homepage Redesign | Never empty -- required at milestone creation (FEAT-04.SPEC-003) |
| {project_name} | Project -- project_name | Acme Rebrand | Never empty -- required at project creation (FEAT-01) |
| {approved_at_formatted} | Milestone -- approved_at, rendered in each recipient's own time zone (FEAT-15) | September 27, 2026, 3:14 PM | Never empty -- set atomically by FEAT-08.SPEC-003 at the moment this notification's trigger fires |
| {approved_by_name} | Client Contact -- name (the approving contact, from Milestone.approved_by) | Owen Marsh | Never empty -- set atomically by FEAT-08.SPEC-003 alongside approved_at |
| {client_name} | Client -- client_name | Acme Co. | Never empty -- required at client creation (FEAT-01) |
| {owen_first_name} | Client Contact -- name (first name portion) | Owen | Greeting renders as "Hi," |
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |

## Delivery Rules

**Batching:** None -- each approval is its own distinct, permanent event and is confirmed individually; two milestones approved close together each produce their own separate confirmation to each recipient, never merged into one summary email.
**Deduplication:** At most one confirmation email per recipient per approval event. FEAT-08.SPEC-003's exactly-once guarantee on the approval write itself is the deduplication boundary: this notification's trigger fires once per confirmed write, so a retried or refused approval attempt (stale, unauthorized, connectivity, or write failure, per FEAT-08.SPEC-003) never produces a confirmation, since the trigger condition -- a confirmed committed write -- was never met.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the project (XBR-30) -- for the copy addressed to Owen as well as her own, since a client contact never sees the freelancer's own delivery-warning surface and Nadia is the one positioned to notice and follow up.
**Expiry:** This confirmation never expires undelivered in the sense of becoming pointless to send late -- the approval it confirms is a permanent record, so a delayed delivery (after retries) still carries accurate, still-true information whenever it eventually lands. There is no cutoff after which the email is withheld; the retry window in the rule above is the only limit, after which delivery is treated as failed (surfaced as a warning) rather than expired.

## Edge Cases

- **The Milestone record is later reopened after this confirmation was sent but before Nadia or Owen reads it** -- The confirmation remains accurate as sent: it describes the approval event that genuinely occurred at that timestamp, and a later reopen is a separate, new event that does not retroactively make the original confirmation false or worth recalling.
- **Owen's or Nadia's email address changes between the approval and delivery** -- Not applicable in practice, since this notification fires and is handed to the delivery capability immediately upon the confirmed write (no batching or delay); if a bounce nonetheless occurs because an address was already invalid, the standard retry-then-warning rule above applies.
- **Owen's copy fails to deliver but Nadia's succeeds (or vice versa)** -- Each recipient's copy is tracked and retried independently; one recipient's successful delivery has no bearing on the other's retry count or warning surfacing.
- **The milestone carries `no_separate_charge` (no invoice will follow)** -- Owen's copy still states the general consent language ("issues the next invoice ... if one applies") accurately, since it is conditional wording, not a promise of an invoice that will not arrive; Nadia's copy is equally accurate for the same reason.
- **A second, later approval (after a reopen) is recorded for the same milestone** -- This notification fires again as its own, independent trigger instance, with its own `approved_at`/`approved_by` values; it is never treated as a duplicate of the first confirmation, since each approval is a genuinely distinct, permanent event.
- **Quiet hours colliding with expiry** -- N/A, since this transactional confirmation observes neither quiet hours nor an expiry cutoff (both stated above); there is no collision to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Triggered by (inbound) | A confirmed successful approval write fires this notification |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Navigation (outbound) | Owen's CTA deep-links here |
| FEAT-01 (Client & Project Management) | Navigation (outbound) | Nadia's CTA deep-links to the milestone's detail in her own project view |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **milestone_approval_confirmation_delivered** (recipient: owen / nadia) -- supports success-metrics.md: "Notification Delivery Reliability"
- **milestone_approval_confirmation_delivery_failed** (recipient: owen / nadia, retry_count) -- supports success-metrics.md: "Notification Delivery Reliability"
- **milestone_approval_confirmation_opened** (recipient: owen / nadia) -- N/A -- no Stage 2 metric measures open rates for this specific confirmation; retained as a standard delivery-quality signal alongside the delivered/failed pair above.

## Acceptance Criteria

**FEAT-08.SPEC-007-AC-01:** Given Owen's approval of a milestone is successfully recorded, when FEAT-08.SPEC-003 confirms the write, then Owen receives an email with subject "You approved {milestone_name}" and Nadia receives an email with subject "{client_name} approved {milestone_name}."

**FEAT-08.SPEC-007-AC-02:** Given Owen opens his confirmation email, when he taps "View milestone," then he lands on FEAT-08.SPEC-001 for that milestone.

**FEAT-08.SPEC-007-AC-03:** Given Nadia opens her confirmation email, when she taps "View milestone," then she lands on that milestone's detail in her own project view.

**FEAT-08.SPEC-007-AC-04:** Given an approval attempt is refused by FEAT-08.SPEC-003 as stale, unauthorized, a connectivity failure, or a write failure, when the refusal occurs, then this notification never fires for that attempt.

**FEAT-08.SPEC-007-AC-05:** Given neither Owen nor Nadia has any way to opt out of this confirmation, when their respective notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-08.SPEC-007-AC-06:** Given an approval is recorded at any hour, when this notification fires, then it sends immediately regardless of either recipient's configured quiet hours, since this confirmation is transactional.

**FEAT-08.SPEC-007-AC-07:** Given delivery of Owen's copy fails, when the failure occurs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees a delivery warning on the project.

**FEAT-08.SPEC-007-AC-08:** Given delivery of Nadia's copy fails while Owen's copy succeeds, when this is observed, then Nadia's copy is retried independently of Owen's successful delivery.

**FEAT-08.SPEC-007-AC-09:** Given the approved milestone carries `no_separate_charge`, when Owen's confirmation is generated, then its consent language about issuing the next invoice remains accurate as conditional wording, with no false promise of an invoice.

**FEAT-08.SPEC-007-AC-10:** Given a milestone is reopened and approved a second time, when the second approval is confirmed, then this notification fires again as an independent instance with the second approval's own `approved_at` and `approved_by` values.

**FEAT-08.SPEC-007-AC-11:** Given Priya is a Reviewer contact on the same client company, when a milestone is approved by Owen, then Priya receives no copy of this confirmation.

**FEAT-08.SPEC-007-AC-12:** Given this confirmation is delivered successfully to both recipients, when delivery completes, then the milestone_approval_confirmation_delivered event fires once per recipient.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
