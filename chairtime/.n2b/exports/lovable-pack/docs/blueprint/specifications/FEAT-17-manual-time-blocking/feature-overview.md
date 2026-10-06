---
document_type: feature-overview
feature_number: FEAT-17
feature_name: Manual Time Blocking
feature_slug: manual-time-blocking
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 3
automation_count: 4
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Manual Time Blocking

## Summary

**Feature:** Manual Time Blocking
**ID:** FEAT-17
**Description:** The Pro can block off a span of time — a doctor's appointment, a vacation day, a personal commitment — removing it from bookable availability without needing to edit their recurring working hours.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** A direct, near-universal need once recurring hours exist: a Pro's actual availability always has one-off exceptions. Without it, the Pro would be forced to edit recurring hours for a single day, which is error-prone and easy to forget to revert. Important rather than Core: the product still functions on recurring hours alone at a pinch, but reliability (a hallmark of this brief) suffers without it. MVP phase: this is a day-one operational need, not a later refinement. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Block a span of time on a specific date (or a recurring pattern, e.g., "every Sunday")
- Remove a block to restore availability
- See blocked time reflected immediately in the slot engine

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-17.SPEC-001 | Create/Edit Time Block | Screen | The Pro | Pro sets a span of time on a specific date, or a recurring pattern, with an optional private label, to create a new block or edit an existing one |
| FEAT-17.SPEC-002 | Manage Time Blocks | Screen | The Pro, Platform Operator (Support) | Pro (Full) and Support (View-only) see the list of upcoming time blocks, with an empty state when none exist and an entry point to edit or remove each one |
| FEAT-17.SPEC-003 | Time Block Conflict Review | Screen | The Pro | Pro sees the confirmed bookings a new or edited block would conflict with and chooses whether to cancel them, reschedule them, or keep the block with those bookings as an exception |
| FEAT-17.SPEC-004 | Time Block Save Commit & Conflict Detection | Automation | The Pro, The Client | Validates and commits a created or edited Time Block, checking it against existing confirmed bookings and routing to the Conflict Review screen when any are found |
| FEAT-17.SPEC-005 | Recurring Time Block Occurrence Generation | Automation | The Pro | Generates and maintains the future dated occurrences of a recurring block pattern (e.g., every Sunday), running each new occurrence through the same conflict detection as a single-date block |
| FEAT-17.SPEC-006 | Time Block Conflict Resolution Commit | Automation | The Pro, The Client | Commits the Pro's explicit choice on a conflicting booking set — hand off to cancellation, hand off to reschedule, or mark the booking as a kept exception — and finalizes the block once every conflicting booking has a resolved outcome |
| FEAT-17.SPEC-007 | Time Block Removal & Expiry | Automation | The Pro | Deletes a block the Pro removes early, or automatically retires a block once its end time has passed, in either case restoring that time to bookable availability immediately |
| FEAT-17.SPEC-008 | Time Block Validation & Conflict Handling Rules | Logic/Rule | All | The shared rules every screen and automation in this feature references: end-after-start validation, what counts as a conflicting booking, the never-silently-affect-a-booking rule, and who can see or act on a block (Pro Full, Client none, Support view-only, labels private) |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Block a span of time on a specific date (or a recurring pattern, e.g., "every Sunday") | FEAT-17.SPEC-001, FEAT-17.SPEC-004, FEAT-17.SPEC-005 | SPEC-001 is the create form (single span or recurrence pattern, optional label); SPEC-004 validates and commits the record; SPEC-005 generates the future occurrences a recurring pattern implies | Phase 2 (Explicit) |
| Remove a block to restore availability | FEAT-17.SPEC-002, FEAT-17.SPEC-007 | SPEC-002 is the entry point (select an existing block from the list); SPEC-007 deletes the record, freeing the time immediately | Phase 2 (Explicit) |
| See blocked time reflected immediately in the slot engine | FEAT-17.SPEC-004, FEAT-17.SPEC-007, FEAT-03.SPEC-001 | SPEC-004 and SPEC-007 write and remove the Time Block record the instant the Pro acts; FEAT-03.SPEC-001 (already validated) computes the live open-slot list by reading Time Block directly, with no caching, so the record's own immediacy is what this capability actually depends on | Phase 2 (Explicit) / cross-feature (already-validated FEAT-03.SPEC-001) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-17.SPEC-002 | Manage Time Blocks | Phase 3 (Entity-Lifecycle) | The Connected Entities line lists Time Block as create/update/delete; the CRUD matrix's Read (list) cell had no covering spec until this screen -- a Pro cannot remove a block, or find one to edit, without first seeing the set of existing blocks |
| FEAT-17.SPEC-003 | Time Block Conflict Review | Phase 2 (Explicit, Alternate flow) / Phase 6 (Failure Analysis) | The Primary Flows & Alternates field's own Alternate line ("a block is added over an already-booked slot... the Pro is warned and must explicitly decide") and the Sick Day journey's step 1 both describe a distinct decision screen, not an inline dialog, given the three-way choice and full-booking-list review it requires |
| FEAT-17.SPEC-004 | Time Block Save Commit & Conflict Detection | Phase 4 (Trigger-Response) | Saving a block is not a simple write: it requires cross-entity checking against every confirmed Booking in the block's range before it can commit, and different outcomes (immediate commit vs. routing to SPEC-003) depending on what it finds -- past the inline-write threshold |
| FEAT-17.SPEC-005 | Recurring Time Block Occurrence Generation | Phase 3 (Entity-Lifecycle) / Phase 4 (Time-based triggers) | The Key Capabilities line names a recurring pattern ("every Sunday") but no capability describes how a single recurrence rule becomes concrete dated occurrences the slot engine can read; this is a distinct generation/maintenance process from the single-date creation in SPEC-001/SPEC-004 |
| FEAT-17.SPEC-006 | Time Block Conflict Resolution Commit | Phase 4 (Trigger-Response) | The Alternate flow's three-way choice (cancel, reschedule, or leave as exception) is cross-feature and cross-entity processing once the Pro decides -- it hands off to FEAT-30 for two of the three outcomes and must track a per-booking resolved/unresolved state for the third, exceeding a simple inline consequence |
| FEAT-17.SPEC-007 | Time Block Removal & Expiry | Phase 3 (Entity-Lifecycle) / Phase 4 (Time-based triggers) | The dependency map's Time Block lifecycle line states a block is "Deleted by FEAT-17 (or expires once its time has passed)" -- a time-based automatic retirement that no Key Capability names but that the entity's own lifecycle definition requires, alongside the explicit Pro-initiated removal |
| FEAT-17.SPEC-008 | Time Block Validation & Conflict Handling Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field's two rules (end after start; a block cannot silently delete a conflicting booking) govern every screen and automation in this feature identically, and the three-way conflict resolution is conditional logic shared across SPEC-003, SPEC-004, and SPEC-006 -- past the inline threshold for a rule referenced by more than one spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Time Block**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-17.SPEC-001, FEAT-17.SPEC-004 | SPEC-001 captures start/end (or recurrence pattern) and an optional private label; SPEC-004 validates and writes the record after conflict detection | Recurring occurrences beyond the first are created by SPEC-005, not SPEC-001/SPEC-004 directly |
| Read (single) | FEAT-17.SPEC-001, FEAT-17.SPEC-002 | SPEC-001 loads an existing block when the Pro opens it to edit; SPEC-002 reads one row at a time from the list | -- |
| Read (list) | FEAT-17.SPEC-002 | Manage Time Blocks screen lists upcoming blocks in date order, with the plain "no time blocked" empty state when none exist | -- |
| Update | FEAT-17.SPEC-001, FEAT-17.SPEC-004 | Pro edits an existing block's span, recurrence, or label on SPEC-001; SPEC-004 re-validates and re-runs conflict detection before committing the change | A block edit that newly conflicts with a booking routes to SPEC-003 exactly as a create does |
| Delete/Archive | FEAT-17.SPEC-002, FEAT-17.SPEC-007 | SPEC-002 is the Pro's removal entry point; SPEC-007 executes it -- a hard delete, restoring the time to bookable availability immediately, with no restore/undo path since removing a block is a non-destructive action to the Pro's own data; deletion never cascades to a conflicting booking (Validation & Limits: a block cannot silently delete a booking); no retention/purge policy applies since a removed or expired block carries no history requirement of its own (Data Notes: "Displayed: on the Pro's own schedule view" only, unlike Booking's SC-22 retention) -- recorded as an explicit non-goal below | -- |
| State Transition | FEAT-17.SPEC-005, FEAT-17.SPEC-007 | A recurring block's individual occurrences move Active -> Expired as their own end time passes, generated and retired by SPEC-005/SPEC-007 respectively; a single-date block has no intermediate state -- it exists until removed or its end passes | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-17.SPEC-004, FEAT-17.SPEC-003, FEAT-17.SPEC-006 | Conflict detection reads confirmed bookings inside the block's range (SPEC-004); the conflict review screen reads the full conflicting set to show the Pro (SPEC-003); the resolution commit reads each booking's current state before finalizing its outcome (SPEC-006) |
| Pro Account | FEAT-17.SPEC-001 | Reads the account timezone so the block's start/end is entered and interpreted consistently with the Pro's other schedule data (Time Block field: "Pro's timezone") |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro submits a new or edited block (single-date or recurrence pattern) | Validate end-after-start and check the range against existing confirmed bookings | Standalone Automation | FEAT-17.SPEC-004 |
| Pro submits a new or edited block | Apply the shared validation and conflict rules | Standalone Logic/Rule | FEAT-17.SPEC-008 |
| Save finds no conflicting bookings | Commit the block immediately; show success confirmation | Inline in triggering screen | FEAT-17.SPEC-001 |
| Save finds one or more conflicting confirmed bookings | Route the Pro to the conflict review screen before the block commits | Standalone Screen | FEAT-17.SPEC-003 |
| Pro chooses "cancel them" on the conflict review screen | Commit the block; hand the conflicting booking set to bulk cancellation | Standalone Automation / Cross-feature | FEAT-17.SPEC-006; FEAT-30.SPEC-005 responsibility |
| Pro chooses "reschedule" for one conflicting booking | Commit the block; hand that booking to Pro-initiated reschedule | Standalone Automation / Cross-feature | FEAT-17.SPEC-006; FEAT-30.SPEC-002 responsibility |
| Pro chooses "keep as an exception" for one conflicting booking | Commit the block; flag the kept booking as an exception, feeding a dashboard attention flag per XBR-11 | Standalone Automation / Cross-feature | FEAT-17.SPEC-006; FEAT-12.SPEC-005 responsibility |
| Pro sets a recurrence pattern on a new block | Generate the pattern's future dated occurrences, each run through the same conflict detection as a single-date block | Standalone Automation | FEAT-17.SPEC-005 |
| A recurring occurrence's own end time passes | Retire that occurrence automatically, freeing its time | Standalone Automation | FEAT-17.SPEC-007 |
| A single-date block's end time passes | Retire the block automatically, freeing its time | Standalone Automation | FEAT-17.SPEC-007 |
| Pro removes a block early from the Manage Time Blocks screen | Delete the record immediately, restoring the time to bookable availability | Standalone Automation | FEAT-17.SPEC-007 |
| Any Time Block is created, updated, removed, or expires | Freed or blocked time is reflected the next time the open-slot list is computed -- no separate propagation step on this feature's side | Cross-feature -- logged in touchpoints | FEAT-03.SPEC-001 responsibility |
| A client's checkout and a Pro's block save contend for the same instant of time | First committed action wins; the loser is refreshed | Cross-feature -- logged in touchpoints | FEAT-03.SPEC-005 responsibility |
| Pro's save fails (e.g., connectivity) | Entered values are preserved on-screen; Pro can retry | Inline in triggering screen | FEAT-17.SPEC-001 |
| Pro attempts to create, edit, or remove a block while offline | N/A -- this is a setup-style action requiring connectivity for correctness; a plain message that connectivity is required, nothing submitted | Inline in triggering screen | FEAT-17.SPEC-001, FEAT-17.SPEC-002 |
| Pro opens Manage Time Blocks with no blocks set | Show the plain "no time blocked" empty state | Inline in triggering screen | FEAT-17.SPEC-002 |

## Shared Context

**Shared Entities:**
- Time Block -- created and updated by SPEC-001 (form) and SPEC-004 (commit); generated as recurring occurrences by SPEC-005; listed by SPEC-002; deleted or expired by SPEC-007; validated by SPEC-008. Fields (functional): start / end (Pro's timezone), recurrence pattern (optional), label (optional, private to the Pro).
- Booking (read-only slice) -- read by SPEC-003, SPEC-004, and SPEC-006 for conflict detection, review, and resolution hand-off; never created, updated, or deleted by this feature.

**Shared UI Patterns:**
- Single/recurring span form -- SPEC-001 uses one form for both a single-date block and a recurring pattern, and for both create and edit; Spec Writers should describe the recurrence toggle and its fields (e.g., day-of-week) consistently whether the Pro is creating a new block or editing an existing one.
- "See the outcome before confirming" pattern -- SPEC-003 shows every conflicting booking and the consequence of each of the three choices before the Pro commits, matching the same ordering convention used by FEAT-10 and FEAT-30's own confirmation screens.

**Shared Validation:**
- FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) is referenced, not duplicated, by SPEC-001 (inline field validation), SPEC-004 (save-time validation and conflict scope), SPEC-003 (what counts as a conflict to show), and SPEC-006 (the never-silently-affect-a-booking rule governing every resolution outcome).

## Internal Dependency Map

```
SPEC-002 (Manage Time Blocks) -> [Pro taps "add a block"] -> SPEC-001 (Create/Edit Time Block)
SPEC-002 (Manage Time Blocks) -> [Pro taps an existing block] -> SPEC-001 (Create/Edit Time Block) [edit mode]
SPEC-001 (Create/Edit Time Block) -> [validates fields using] -> SPEC-008 (Time Block Validation & Conflict Handling Rules)
SPEC-001 (Create/Edit Time Block) -> [Pro saves] -> SPEC-004 (Time Block Save Commit & Conflict Detection)
SPEC-004 (Time Block Save Commit & Conflict Detection) -> [checks conflict scope using] -> SPEC-008 (Time Block Validation & Conflict Handling Rules)
SPEC-004 (Time Block Save Commit & Conflict Detection) -> [no conflicts found] -> commits the block -> [read live by] -> FEAT-03.SPEC-001
SPEC-004 (Time Block Save Commit & Conflict Detection) -> [conflicts found] -> SPEC-003 (Time Block Conflict Review)
SPEC-003 (Time Block Conflict Review) -> [Pro chooses cancel / reschedule / exception] -> SPEC-006 (Time Block Conflict Resolution Commit)
SPEC-006 (Time Block Conflict Resolution Commit) -> [cancel chosen, one or more bookings] -> FEAT-30.SPEC-005 (Cancel Several Bookings at Once)
SPEC-006 (Time Block Conflict Resolution Commit) -> [reschedule chosen, one booking] -> FEAT-30.SPEC-002 (Reschedule Booking (Pro-Initiated))
SPEC-006 (Time Block Conflict Resolution Commit) -> [exception chosen] -> flags the booking -> [surfaced by] -> FEAT-12.SPEC-005 (Attention Flag Aggregation)
SPEC-006 (Time Block Conflict Resolution Commit) -> [every conflicting booking resolved] -> finalizes the block -> [read live by] -> FEAT-03.SPEC-001
SPEC-001 (Create/Edit Time Block) -> [Pro sets a recurrence pattern] -> SPEC-005 (Recurring Time Block Occurrence Generation)
SPEC-005 (Recurring Time Block Occurrence Generation) -> [each generated occurrence checked using] -> SPEC-008 (Time Block Validation & Conflict Handling Rules) -> [conflicts found] -> SPEC-003 (Time Block Conflict Review)
SPEC-002 (Manage Time Blocks) -> [Pro removes a block] -> SPEC-007 (Time Block Removal & Expiry)
SPEC-007 (Time Block Removal & Expiry) -> [block's end time passes] -> [automatic retirement] -> SPEC-007
SPEC-007 (Time Block Removal & Expiry) -> [block removed or expired] -> [read live by] -> FEAT-03.SPEC-001
```

**Default Entry:** This feature has no single default landing screen -- the Pro arrives at SPEC-001 (Create/Edit Time Block) via FEAT-12's "block time" entry point on the schedule view to create a new block, or at SPEC-002 (Manage Time Blocks) to see, edit, or remove an existing one.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-17.SPEC-004 / SPEC-007 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Time Block records this feature writes, updates, or removes are read live by FEAT-03.SPEC-001's slot computation -- no separate propagation step on this feature's side | Any block created, updated, removed, or expired |
| FEAT-17.SPEC-004 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | A block save and a client checkout contending for the same instant of time resolve by FEAT-03.SPEC-005's first-committed-wins rule | A client is mid-checkout on a slot the Pro is simultaneously blocking |
| FEAT-17.SPEC-001 / SPEC-002 | Inbound | FEAT-12 (Pro Daily Schedule Dashboard) | Pro taps the "block time" entry point on the schedule view | Pro taps block time |
| FEAT-17 (Time Block) | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Time Blocks are shown on the Pro's schedule (FEAT-12.SPEC-001) alongside bookings so a blocked period reads as occupied, not empty | Any block created or removed |
| FEAT-17.SPEC-006 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | A booking the Pro keeps as an exception, or leaves unresolved, feeds an attention flag (FEAT-12.SPEC-005) per XBR-11 | Pro chooses "keep as an exception," or leaves a conflict unresolved |
| FEAT-17.SPEC-006 | Outbound | FEAT-30 (Pro Booking Management) | Hands the conflicting booking set to bulk cancellation (FEAT-30.SPEC-005) when the Pro chooses to cancel them together | Pro chooses "cancel them" |
| FEAT-17.SPEC-006 | Outbound | FEAT-30 (Pro Booking Management) | Routes a single affected booking to Pro-initiated reschedule (FEAT-30.SPEC-002) | Pro chooses "reschedule" for one booking |
| FEAT-17.SPEC-001 | Inbound | FEAT-02 (Availability & Working Hours Setup) | FEAT-02.SPEC-001 directs a one-off closed day here instead of editing the recurring weekly rule | Pro wants to close a single day rather than change working hours |
| FEAT-17.SPEC-001, SPEC-002 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | Every Pro-facing screen in this feature requires a signed-in Pro; anyone else is sent to sign-in (XBR-29) | Any navigation to this feature |

## Non-Functional Notes

**Data volumes / growth:** A solo Pro's time-block set stays small -- occasional one-off blocks plus a handful of recurring patterns -- consistent with the feature's own States field describing the dataset as small enough that loading is instant and needs no progress indicator.

**Responsiveness:** A block disappears from, or reappears in, the bookable slot list within roughly one second of the Pro saving or removing it (ASMP-21, applied because blocked time must disappear from the live slot list right away); block times are entered and interpreted in the Pro's own timezone (ASMP-25).

**Data sensitivity / privacy:** A Time Block is private to the Pro -- its optional label can describe a personal commitment (e.g., a doctor's appointment) and is never shown to clients, who see only the resulting absence of slots; Platform Operator (Support) has view-only access to blocks, consistent with its read-only role everywhere else (dependency map, Time Block Data Sensitivity line).

**Compliance flags:** N/A -- this feature applies no compliance regime of its own; creating or removing a block is a correctness action needing a live connection (ASMP-27) but carries no health, payment, or identity data of its own.

**Signals:** This feature emits time_block_added on a successful commit (Time Block Save Commit & Conflict Detection, SPEC-004, including each generated recurring occurrence via SPEC-005), time_block_removed on an explicit Pro removal or an automatic expiry (Time Block Removal & Expiry, SPEC-007), and time_block_conflict_flagged when a save finds a conflicting confirmed booking (Time Block Conflict Review, SPEC-003, and its resolution outcome via SPEC-006) -- these three signals are the analytics basis for tracking how often blocking collides with existing bookings, feeding the Zero Double-Booking Confidence success metric's contention-handling half.

## Non-Goals

- **A bulk-reschedule flow for several conflicting bookings at once** -- Excluded per the Requirements Architect's coordination note: FEAT-30's validated Brief provides a bulk cancel (SPEC-005/SPEC-008) but reschedules only one booking at a time (SPEC-002); this feature routes each rescheduled booking individually to FEAT-30.SPEC-002 rather than inventing a bulk-reschedule capability FEAT-30 does not define.
- **Automatic purge or retention policy for removed or expired Time Blocks** -- Intentional lifecycle decision surfaced by the CRUD matrix: a Time Block that the Pro removes, or that expires, is hard-deleted with no retention window, because the product definition gives it no historical-record requirement of its own (Data Notes: displayed only on the Pro's own schedule view, unlike Booking's SC-22 permanent-history requirement).
- **Deriving a Time Block automatically from the Pro's personal calendar busy time** -- Excluded per the dependency map's own Dependencies note: busy time synced from the Pro's personal calendar is FEAT-04's responsibility (ASMP-33) and is never written as, or converted into, a Time Block; the two remain distinct availability inputs that FEAT-03 combines separately.
- **Multi-staff or shared/team time blocking** -- Excluded per scope-boundaries.md (SC-01): the product is strictly single-operator, so a Time Block belongs to exactly one Pro Account with no concept of a shared calendar, delegated blocking, or a second operator's schedule to coordinate against.
- **The product adjudicating which of the three conflict-resolution choices is "correct"** -- Excluded per the Alternate flow's own wording and XBR-11: cancel, reschedule, or keep-as-exception are equally legitimate outcomes; the Pro's explicit choice is the only path, and this feature never defaults to one automatically or silently.
