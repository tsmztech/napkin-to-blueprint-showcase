---
document_type: feature-overview
feature_number: FEAT-08
feature_name: Milestone Approval
feature_slug: milestone-approval
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 2
automation_count: 3
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

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
