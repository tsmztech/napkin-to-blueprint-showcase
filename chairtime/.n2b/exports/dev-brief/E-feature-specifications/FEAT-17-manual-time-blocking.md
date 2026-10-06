# FEAT-17 — Manual Time Blocking

This chapter covers Manual Time Blocking (FEAT-17), a Important-tier feature. It carries 8 specifications carrying 114 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-17.SPEC-001 | Create/Edit Time Block | screen | 17 |
| FEAT-17.SPEC-002 | Manage Time Blocks | screen | 14 |
| FEAT-17.SPEC-003 | Time Block Conflict Review | screen | 15 |
| FEAT-17.SPEC-004 | Time Block Save Commit & Conflict Detection | automation | 12 |
| FEAT-17.SPEC-005 | Recurring Time Block Occurrence Generation | automation | 12 |
| FEAT-17.SPEC-006 | Time Block Conflict Resolution Commit | automation | 13 |
| FEAT-17.SPEC-007 | Time Block Removal & Expiry | automation | 11 |
| FEAT-17.SPEC-008 | Time Block Validation & Conflict Handling Rules | logic-rule | 20 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Create/Edit Time Block

## Overview

**Name:** Create/Edit Time Block
**ID:** FEAT-17.SPEC-001
**Type:** Screen
**Purpose:** Talia sets a span of time on a specific date, or a recurring weekly pattern, with an optional private label, to create a new Time Block or edit an existing one.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- One form serving four combinations: create single-date, create recurring pattern, edit single-date, edit an existing recurring pattern
- Capturing start, end, an optional recurrence pattern, and an optional private label
- Inline field validation governed by FEAT-17.SPEC-008
- Handing the submitted values to FEAT-17.SPEC-004 on save
- The empty (new-block default), filling, saving, error, and offline/degraded states for this form

**Non-Goals:**
- Listing existing blocks to choose one to edit -- owned by FEAT-17.SPEC-002 (Manage Time Blocks), the entry point into this screen's edit mode
- Detecting or resolving a conflict with an existing booking -- owned by FEAT-17.SPEC-004 (detection) and FEAT-17.SPEC-003 (review); this screen only submits the values and reacts to where FEAT-17.SPEC-004 routes it next
- Generating the individual dated occurrences a recurring pattern implies -- owned by FEAT-17.SPEC-005; this screen only captures the pattern definition
- Multi-staff or shared-calendar blocking -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so a block belongs to exactly one Pro Account with no second operator to coordinate against

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps the "block time" entry point on the schedule view | None -- form starts empty in create mode |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Talia taps "add a block" | None -- form starts empty in create mode |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Talia taps an existing block row | The selected Time Block's start, end, recurrence pattern (if any), and label are loaded into the form in edit mode |
| FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Talia wants to close a single day rather than change her recurring working hours | None -- form starts empty in create mode, pre-focused on the date field |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- her own blocks only | Create and edit her own blocks | -- |
| The Client (Riley) | No | No | This screen exists only inside the Pro's signed-in application; Riley has no navigation path to it at all -- Riley's own experience of a block is limited to its absence from the bookable slot list on the public booking page |
| Platform Operator (Support) | No | No | Support's read-only surface for this feature is the Manage Time Blocks list (FEAT-17.SPEC-002); this create/edit form is outside support's granted session scope, so it is never reachable from the support view |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form values are preserved locally and restored on the form after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Block Time" (create mode) or "Edit Time Block" (edit mode), with a back arrow (returns to the entry screen without saving) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields, in order:
- **Date** (date input, required) -- the date of the block, or the first occurrence date when a recurrence pattern is set
- **Start time** (time input, required)
- **End time** (time input, required)
- **Timezone note** -- a non-interactive line beneath the time fields showing "Times are in your account timezone ({Pro Account timezone})," read from the Pro Account so the block is entered and interpreted consistently with the Pro's other schedule data
- **Repeats** (toggle, default off) -- when turned on, reveals a **day-of-week selector** (single selection, defaulting to the day-of-week of the chosen Date) describing the recurring pattern (e.g., "every Sunday")
- **Label** (text input, optional) -- placeholder text "Private note (only you see this)"; a static caption beneath the field states "Clients never see this label."

Editing an existing recurring pattern shows the same Repeats toggle already on, with its day-of-week selector pre-filled; editing a single already-generated occurrence (opened from a specific dated row on FEAT-17.SPEC-002) shows the Repeats toggle off and hidden entirely, since a single generated occurrence is edited as its own dated instance, not as the pattern.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry screen (FEAT-12, FEAT-17.SPEC-002, or FEAT-02.SPEC-001) discarding unsaved input | Screen closes | If fields were filled, a confirmation dialog appears first (see Edge Cases) |
| Date input | Select a date | Captures the date; if Repeats is on, updates the day-of-week selector's default to match | Field shows selected date | Standard date-picker feedback |
| Start time input | Select a time | Captures start time | Field shows selected time | Standard input feedback |
| End time input | Select a time | Captures end time; triggers end-after-start validation via FEAT-17.SPEC-008 | Field shows selected time | Error state and message if end is not after start |
| Repeats toggle | Tap | Reveals or hides the day-of-week selector | Form layout expands/contracts | Day-of-week selector animates into or out of view |
| Day-of-week selector | Select a day | Captures the recurrence pattern's weekday | Selector shows chosen day | Selected day highlighted |
| Label input | Type | Captures free text, validated via FEAT-17.SPEC-008 (length) | Field shows entered text | Character count shown as the limit is approached |
| Save button | Tap | 1. Validate all fields via FEAT-17.SPEC-008. 2. If valid, submit to FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection). | Button shows loading state during submission | See States: Saving, then Error or navigation per FEAT-17.SPEC-004's outcome |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Date -> Start time -> End time -> Repeats toggle -> Day-of-week selector (when visible) -> Label -> Save.
- **Validation announcements:** When a field enters an error state (end-before-start, label too long), its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** A successful commit's confirmation is announced; on validation or conflict-routing outcomes, focus moves to the first field in error or to the routed screen's heading.
- **Keyboard alternatives:** Every action, including the Repeats toggle and day-of-week selection, is reachable by keyboard; date and time inputs offer a typed-entry alternative to any picker gesture.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (edit mode only) | N/A -- a solo Pro's block set is small enough that the existing block's values load instantly, with no progress indicator needed (Non-Functional Notes: data volumes) | Screen opens in edit mode, before the selected block's values are available | Values load (effectively immediately) and the Loaded state appears |
| Empty (create, default) | All fields empty except Date, which defaults to today; Save enabled | Screen opens in create mode | Talia begins editing any field |
| Loaded (edit, default) | Fields pre-filled from the selected Time Block; Save enabled | Screen opens in edit mode from FEAT-17.SPEC-002 | Talia begins editing any field |
| Filling | Fields contain Talia's input; inline validation runs per FEAT-17.SPEC-008 | Talia types or selects in any field | Talia taps Save or navigates away |
| Saving | Save button shows a loading indicator, fields disabled | Talia taps Save and all inline validation passes | FEAT-17.SPEC-004 returns an outcome (commit, conflict routing, or failure) |
| Error | Error banner at the top of the form: "Couldn't save this time block. Check your connection and try again." with a Retry control; entered values remain in the fields | FEAT-17.SPEC-004 reports a save failure | Talia taps Retry and the save succeeds, or she navigates away |
| Offline/Degraded | Plain message "Connect to the internet to save a time block." replaces the Save button's active state; fields remain visible and editable but Save is disabled | Connectivity is lost while the screen is open, or the screen is opened while already offline | Connectivity is restored -- Save re-enables |

## Validation Rules

Validation governed by FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules). See that spec for all field-level rules, including end-after-start and label length. This screen applies validation on field blur (Date, Start time, End time, Label) and again on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-17.SPEC-002 (Manage Time Blocks), FEAT-12, or FEAT-02.SPEC-001, matching the entry source | Varies by entry source |
| Save succeeds with no conflicting bookings | FEAT-17.SPEC-002 (Manage Time Blocks) | -- |
| Save finds one or more conflicting confirmed bookings | FEAT-17.SPEC-003 (Time Block Conflict Review) | -- |
| Discard confirmation -- "Discard" chosen | Same destination as the back arrow, per entry source | Varies by entry source |

## Data Model

**Creates:** Time Block record -- start, end, recurrence (optional), label (optional), owned by the signed-in Pro Account. Setting a recurrence pattern also triggers FEAT-17.SPEC-005 to generate the pattern's future dated occurrences.
**Reads:** Time Block record (in edit mode, all fields, to pre-fill the form); Pro Account -- timezone field only, to label the time inputs consistently with the Pro's other schedule data.
**Updates:** Time Block record -- start, end, recurrence, label, on an existing block Talia opens to edit.
**Deletes:** None -- removal is owned by FEAT-17.SPEC-002 and FEAT-17.SPEC-007.

## Business Rules

- End-after-start and label-length validation are enforced by FEAT-17.SPEC-008 -- Talia cannot save with invalid values.
- Saving never commits or rejects a conflict decision itself; FEAT-17.SPEC-004 determines whether the save completes immediately or routes to FEAT-17.SPEC-003 (XBR-11: setup changes never silently affect a confirmed booking).
- Editing an existing recurring pattern's span, day-of-week, or label affects that pattern's future not-yet-elapsed occurrences (regenerated by FEAT-17.SPEC-005); already-elapsed occurrences are historical and are never altered.
- A block cannot be created or edited without connectivity -- this is a setup-style action requiring a live connection for correctness (ASMP-27).

## Edge Cases

- **Talia navigates away with unsaved changes** -- Confirmation dialog: "Discard this time block?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner as described in States; entered values are preserved and Talia can retry without re-entering anything.
- **Talia turns Repeats on after already entering a Date in the past relative to today** -- The date field itself is not restricted to future dates (a block can start today), but the day-of-week selector always derives from whichever Date is currently entered.
- **Talia edits a single already-generated occurrence (not the parent pattern) and turns Repeats on** -- Not offered: the Repeats toggle is hidden entirely for a single generated occurrence, since converting one occurrence into a new pattern is not a supported edit; Talia would instead create a new recurring block from this screen's create mode.
- **Concurrent edit -- another session (e.g., a second signed-in device) removes or edits this same block while this form is open** -- On Save, FEAT-17.SPEC-004 re-validates against the current record; if the block no longer exists, the save is rejected with "This time block was removed. Start over?" (Discard returns to FEAT-17.SPEC-002); if the block was edited elsewhere, the last commit to complete wins (last-write-wins, per the dependency map's Contention note for Time Block) and this form's save simply overwrites it, matching the block's own single-owner editing model.
- **Label left empty** -- Save proceeds; the block carries no label and displays with only its time range on FEAT-17.SPEC-002 and the Pro's schedule (FEAT-12).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-002 (Manage Time Blocks) | Navigation (inbound/outbound) | Entry point for create and edit; destination after a conflict-free save |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Navigation (outbound) | Destination when the save finds conflicting bookings |
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | Triggers (outbound) | Save submits the form's values for validation, conflict detection, and commit |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | Triggers (outbound) | A saved or edited recurrence pattern triggers occurrence generation |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Field-level validation rules applied to this form |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound) | "Block time" entry point |
| FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Navigation (inbound) | Directs a one-off closed day here instead of editing recurring hours |

## Analytics and Success Signals

N/A -- this screen only captures and submits values; the block-creation outcome this feature's Signals track (`time_block_added`, `time_block_conflict_flagged`) is emitted by the automation that actually commits or routes the data (FEAT-17.SPEC-004), not by this entry form. Tracking submission here would double-count the same event the commit automation already emits.

## Acceptance Criteria

**FEAT-17.SPEC-001-AC-01:** Given Talia is on the Create Time Block screen with an empty form, when she sets tomorrow's date, a start and end time, and taps Save, then FEAT-17.SPEC-004 validates and commits the block, and Talia is returned to FEAT-17.SPEC-002 with the new block visible.

**FEAT-17.SPEC-001-AC-02:** Given Talia is on the Create Time Block screen, when she sets an end time earlier than the start time, then the end time field shows an error state with the message defined by FEAT-17.SPEC-008 and Save does not proceed.

**FEAT-17.SPEC-001-AC-03:** Given Talia turns on the Repeats toggle, when the day-of-week selector appears, then it defaults to the day-of-week of the currently entered Date.

**FEAT-17.SPEC-001-AC-04:** Given Talia sets a recurring pattern and taps Save, when the save commits, then FEAT-17.SPEC-005 generates the pattern's future dated occurrences.

**FEAT-17.SPEC-001-AC-05:** Given Talia enters a label describing a personal commitment and saves, when the block is later viewed on FEAT-17.SPEC-002 or the Pro's schedule (FEAT-12), then the label is visible only to Talia and never appears on any client-facing screen.

**FEAT-17.SPEC-001-AC-06:** Given Talia's new block's time range overlaps an existing confirmed booking, when she taps Save, then she is routed to FEAT-17.SPEC-003 (Time Block Conflict Review) instead of seeing an immediate success confirmation.

**FEAT-17.SPEC-001-AC-07:** Given Talia is on the Create Time Block screen with unsaved input, when she taps the back arrow, then a confirmation dialog appears asking "Discard this time block?" with "Discard" and "Keep Editing" options.

**FEAT-17.SPEC-001-AC-08:** Given Talia taps Save and it is in progress, when she taps Save again, then the second tap has no effect and the button remains in its loading state.

**FEAT-17.SPEC-001-AC-09:** Given Talia's save fails due to a connectivity error, when the error banner appears, then her entered values remain in every field and she can retry without re-entering anything.

**FEAT-17.SPEC-001-AC-10:** Given Talia loses connectivity while the form is open, when she looks at the Save button, then it is disabled and the message "Connect to the internet to save a time block." is shown.

**FEAT-17.SPEC-001-AC-11:** Given Talia opens an existing single-date block from FEAT-17.SPEC-002, when the form loads, then the Repeats toggle is hidden and the fields are pre-filled with that block's start, end, and label.

**FEAT-17.SPEC-001-AC-12:** Given Talia opens an existing recurring pattern from FEAT-17.SPEC-002 and changes its end time, when she saves, then the pattern's future not-yet-elapsed occurrences are regenerated with the new end time, and already-elapsed occurrences are unchanged.

**FEAT-17.SPEC-001-AC-13:** Given Talia leaves the Label field empty and saves, when the block is created, then it carries no label and displays with only its time range.

**FEAT-17.SPEC-001-AC-14:** Given Talia is filling the form, when she looks below the time fields, then she sees her account's timezone stated so she can confirm the entered times are interpreted correctly.

**FEAT-17.SPEC-001-AC-15:** Given Talia arrives at this screen from FEAT-02.SPEC-001 to close a single day, when the form opens, then it is in create mode with an empty form pre-focused on the date field.

**FEAT-17.SPEC-001-AC-16:** Given another session removed the block Talia currently has open for editing, when she taps Save, then she sees "This time block was removed. Start over?" with the option to discard back to FEAT-17.SPEC-002.

**FEAT-17.SPEC-001-AC-17:** Given the Client (Riley) has no signed-in Pro session, when any attempt is made to reach this screen's URL, then no such navigation path exists in Riley's experience -- Riley's only exposure to a block is its absence from the bookable slot list.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 7 (loading, empty, loaded, filling, saving, error, offline/degraded) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Screen Spec: Manage Time Blocks

## Overview

**Name:** Manage Time Blocks
**ID:** FEAT-17.SPEC-002
**Type:** Screen
**Purpose:** Talia (Full) and Platform Operator Support (View-only) see the list of upcoming Time Blocks, with a plain empty state when none exist and an entry point to edit or remove each one.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Listing upcoming Time Blocks in date order, one row per single-date block and one summary row per recurring pattern
- The "no time blocked" empty state
- Entry points to create a new block, edit an existing one, and remove one
- Support's read-only view of the same list

**Non-Goals:**
- The create/edit form itself -- owned by FEAT-17.SPEC-001, which this screen navigates to
- Executing the removal (the actual delete and slot-freeing) -- owned by FEAT-17.SPEC-007; this screen only provides the entry point and confirmation
- Showing past (already-expired) blocks -- excluded per the feature's own Data Notes: a removed or expired block carries no historical-record requirement of its own, so this list shows upcoming blocks only, not a history view
- Search or filtering across blocks -- excluded per scope-boundaries.md: a solo Pro's block set is small enough (Non-Functional Notes: data volumes) that no search capability is warranted

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia opens her schedule navigation to manage blocks | None -- list loads current upcoming blocks |
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Talia completes a save with no conflicts, or discards an edit | None -- list reloads to reflect any change |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens this feature's view within an active Pro-account review | The Pro account under review; no action controls rendered |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Conflict review confirms with every remaining booking kept as an exception | None -- list reloads to reflect the saved block |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- her own blocks only | Add, edit, and remove her own blocks | -- |
| Platform Operator (Support) | Full list for the one Pro account under active review, including labels (Support has View-only access to Time Block, including its label, per the dependency map's Data Sensitivity note) | View only -- no add, edit, or remove controls rendered | Any write control is simply not present; there is no denial dialog because no write path is ever rendered for Support |
| The Client (Riley) | No | No | This screen exists only inside the Pro's signed-in application (or support's review session); Riley has no navigation path to it |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen to preserve, since it is a viewing/entry-point surface with no form state |

## Layout and Content

**Header:** Screen title "Time Blocks" with a back arrow (returns to FEAT-12) and, for the Pro only, an "Add a block" action button (right-aligned). Support sees the title and back arrow only -- no add action.

**Body:** A single vertically scrolling list of upcoming blocks in date order. Each **row** shows:
- Date (and, for a recurring pattern, "Repeats every {day of week}" instead of a single date, with the next occurrence date shown beneath)
- Time range (start--end)
- Label, when one exists (for the Pro, shown in full; for Support, shown in full per its View-only entitlement to the label)
- A remove affordance, visible to the Pro only

**Empty state:** When Talia has no upcoming blocks, the body shows a plain message: "No time blocked." with the "Add a block" action available from the header. Support sees the same message with no add action.

**Footer:** None -- Add is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described, full width.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard), or exit the review panel for Support | Screen closes | Standard backward transition |
| "Add a block" (Pro only) | Tap | Navigate to FEAT-17.SPEC-001 (Create/Edit Time Block) in create mode | Screen transitions | Standard navigation transition |
| Block row (Pro only) | Tap | Navigate to FEAT-17.SPEC-001 (Create/Edit Time Block) in edit mode, pre-filled with this block's or pattern's values | Screen transitions | Standard navigation transition |
| Block row (Support) | Tap | No action -- display-only for Support | None | Row shows a static, non-interactive treatment |
| Remove affordance (Pro only) | Tap | Prompts a confirmation dialog: "Remove this time block? The time becomes bookable again immediately." | Dialog appears | Confirmation dialog with "Remove" and "Cancel" |
| Remove confirmation -- "Remove" | Tap | Triggers FEAT-17.SPEC-007 (Time Block Removal & Expiry) | Row is removed from the list | List updates in place; if the list becomes empty, the empty state appears |
| Remove confirmation -- "Cancel" | Tap | No action | Dialog closes | Row remains unchanged |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches the upcoming block list | List reloads | Loading indicator during refresh |

### Accessibility Notes

- **Focus order:** Back arrow -> "Add a block" (Pro only) -> each block row in date order (within a row: date/pattern, time range, label, remove affordance) -> refresh control.
- **Dynamic-change announcements:** When a row is removed, its removal from the list is announced to assistive technology; when the list becomes empty, the "No time blocked." message is announced.
- **Keyboard alternatives:** Every action (add, open a row to edit, remove, confirm/cancel, refresh) is reachable without a pointer-only gesture; pull-to-refresh has an equivalent tappable refresh control.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (has blocks) | Rows populated as described in Layout and Content, in date order | Data fetch succeeds with at least one upcoming block | Data changes (new fetch, add, edit, or remove) |
| Empty | "No time blocked." message, with "Add a block" available (Pro only) | Data fetch succeeds with zero upcoming blocks | A block is created |
| Loading | A lightweight in-place indicator; on first-ever load, a brief full-screen lightweight loading indicator | Screen first opens, or a refresh is triggered | Data fetch completes (success or failure) |
| Error | Error banner: "Couldn't load your time blocks. Check your connection and try again." with a Retry control | Data fetch fails | Retry succeeds, or connectivity is restored and an automatic retry succeeds |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded time blocks." at the top; the most recently loaded list remains viewable read-only; add, edit, and remove controls are disabled with a note that they require reconnecting | Connectivity is lost while this screen is open, or the screen is opened while already offline with cached data available | Connectivity is restored -- the banner clears and a fresh fetch runs automatically |

## Validation Rules

This screen has no user-entry form fields. The remove confirmation is a binary choice with no field-level validation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | FEAT-12 (Pro only); support exits its review panel |
| "Add a block" tap | FEAT-17.SPEC-001 (Create/Edit Time Block) | -- |
| Block row tap (Pro) | FEAT-17.SPEC-001 (Create/Edit Time Block) | -- |
| Remove confirmed | Stays on this screen; row removed in place | -- |

## Data Model

**Creates:** None directly -- creation is owned by FEAT-17.SPEC-001/FEAT-17.SPEC-004.
**Reads:** Time Block records -- start, end, recurrence, label, for the signed-in Pro Account (or the Pro account under Support's active review), grouped for display into one row per single-date block and one summary row per recurring pattern.
**Updates:** None directly -- editing is owned by FEAT-17.SPEC-001.
**Deletes:** Time Block record -- triggers FEAT-17.SPEC-007 on confirmed removal.

## Business Rules

- Support's access is read-only in every respect on this screen -- no add, edit, or remove control is ever rendered for Support (Access Matrix: Service & Availability Setup = View for Platform Operator).
- Removing a block never affects a confirmed booking that was kept as an exception to it -- the booking is untouched; only the block itself is deleted (Validation & Limits: a block cannot silently delete a conflicting booking).
- The list shows upcoming blocks only; an expired block is retired by FEAT-17.SPEC-007 and no longer appears here.
- Support's view-only rendering of this screen (no write control rendered, no field validation because there are no form fields) is governed by FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules), which lists this screen as an enforcing spec for its authorization rules.

## Edge Cases

- **Talia removes the last remaining block** -- The list transitions directly to the "No time blocked." empty state.
- **Talia taps remove twice rapidly on the same row** -- The confirmation dialog appears once; a second tap while the dialog is open has no additional effect.
- **A block Talia is viewing expires while this screen is open (its end time passes during the session)** -- On the next refresh (automatic or pull-to-refresh) the expired block no longer appears; no separate expiry notice is shown here, since expiry is a routine background retirement, not an error.
- **Support opens this screen for a Pro account with no blocks** -- The same "No time blocked." message appears, with no add action shown.
- **Network failure during removal** -- The row remains in the list with an inline error: "Couldn't remove this time block. Try again." and the remove affordance remains available to retry.
- **Another session (e.g., a second signed-in device) already removed or edited the block Talia is confirming removal for** -- FEAT-17.SPEC-007 finds the record already gone (or already changed) and reports the already-gone outcome with no error; the row simply disappears from Talia's list on the next refresh, consistent with the dependency map's Contention note for Time Block (last-write-wins between the Pro's own sessions).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Navigation (outbound) | Add and edit entry points |
| FEAT-17.SPEC-007 (Time Block Removal & Expiry) | Triggers (outbound) | Confirmed removal triggers the delete |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Entry point and back destination |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's read-only entry point during an account review |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules) | Governed by | Authorization rules for entry and Support's view-only rendering |

## Analytics and Success Signals

N/A -- this screen is a navigation and confirmation surface; the removal outcome this feature's Signals track (`time_block_removed`) is emitted by FEAT-17.SPEC-007, which performs the actual delete, not by this list screen.

## Acceptance Criteria

**FEAT-17.SPEC-002-AC-01:** Given Talia has two upcoming single-date blocks and one recurring pattern, when she opens Manage Time Blocks, then she sees three rows in date order, the recurring one labeled "Repeats every {day}" with its next occurrence date.

**FEAT-17.SPEC-002-AC-02:** Given Talia has no upcoming blocks, when she opens Manage Time Blocks, then she sees the message "No time blocked." with "Add a block" available.

**FEAT-17.SPEC-002-AC-03:** Given Talia is on Manage Time Blocks, when she taps "Add a block", then she is navigated to FEAT-17.SPEC-001 in create mode.

**FEAT-17.SPEC-002-AC-04:** Given Talia taps an existing block row, when the screen transitions, then FEAT-17.SPEC-001 opens in edit mode pre-filled with that block's values.

**FEAT-17.SPEC-002-AC-05:** Given Talia taps the remove affordance on a block row, when the confirmation dialog appears, then it reads "Remove this time block? The time becomes bookable again immediately." with "Remove" and "Cancel" options.

**FEAT-17.SPEC-002-AC-06:** Given Talia confirms removal of a block, when FEAT-17.SPEC-007 completes the delete, then the row disappears from the list immediately.

**FEAT-17.SPEC-002-AC-07:** Given Talia cancels the remove confirmation dialog, when she taps "Cancel", then the dialog closes and the row remains unchanged.

**FEAT-17.SPEC-002-AC-08:** Given Platform Operator Support is reviewing a Pro's account, when Support opens this screen, then every row is visible, including labels, with no remove or edit control rendered anywhere on the screen.

**FEAT-17.SPEC-002-AC-09:** Given a data fetch fails when Talia opens this screen, when the error appears, then she sees "Couldn't load your time blocks. Check your connection and try again." with a Retry control.

**FEAT-17.SPEC-002-AC-10:** Given Talia loses connectivity while viewing her block list, when the offline banner appears, then her most recently loaded list remains visible and every write control is disabled.

**FEAT-17.SPEC-002-AC-11:** Given Talia removes her only remaining block, when the removal completes, then the screen transitions directly to the "No time blocked." empty state.

**FEAT-17.SPEC-002-AC-12:** Given a network failure occurs while Talia confirms a removal, when the failure is reported, then the row remains in the list with the inline error "Couldn't remove this time block. Try again." and the remove affordance is still available.

**FEAT-17.SPEC-002-AC-13:** Given a block Talia removed had a confirmed booking kept as an exception, when the block is deleted, then that booking is left completely untouched.

**FEAT-17.SPEC-002-AC-14:** Given the Client (Riley) has no signed-in Pro or support session, when any attempt is made to reach this screen, then no such navigation path exists in Riley's experience.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (loaded, empty, loading, error, offline/degraded) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Time Block Conflict Review

## Overview

**Name:** Time Block Conflict Review
**ID:** FEAT-17.SPEC-003
**Type:** Screen
**Purpose:** Talia sees every confirmed booking a new or edited Time Block conflicts with and chooses, per booking, to cancel it, reschedule it, or keep the block with that booking as an exception, before the block commits.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Listing every confirmed booking that conflicts with the block Talia just tried to save
- Capturing Talia's per-booking choice: cancel, reschedule, or keep as an exception
- Showing the consequence of each choice before she confirms, matching the "see the outcome before confirming" pattern shared with FEAT-10 and FEAT-30
- Submitting the confirmed set of choices to FEAT-17.SPEC-006 for commit

**Non-Goals:**
- Detecting which bookings conflict -- owned by FEAT-17.SPEC-004; this screen only displays what that automation already found
- Executing the cancellation or reschedule itself -- owned by FEAT-30.SPEC-005 (bulk cancel) and FEAT-30.SPEC-002 (single reschedule), reached through FEAT-17.SPEC-006's hand-off
- Deciding which of the three choices is "correct" -- excluded per the Alternate flow's own wording and XBR-11: cancel, reschedule, and keep-as-exception are equally legitimate outcomes, and the product never defaults to one automatically
- A bulk-reschedule option for several conflicting bookings at once -- excluded per the Brief's own Non-Goals: FEAT-30 reschedules only one booking at a time, so this screen never offers a "reschedule all" choice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | A single-date block save finds one or more conflicting confirmed bookings | The pending block's values and the full set of conflicting Booking references |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | A generated recurring occurrence conflicts with a confirmed booking | The pending occurrence's values and the conflicting Booking reference(s) for that occurrence |
| FEAT-17.SPEC-001 (Create/Edit Time Block) | A save finds one or more conflicting confirmed bookings | The pending block values and the conflicting Booking references (via FEAT-17.SPEC-004) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- only reachable as part of her own save/generation flow | Choose cancel, reschedule, or keep-as-exception for each listed booking, and confirm | -- |
| The Client (Riley) | No | No | This screen exists only inside the Pro's signed-in application, reached only mid-save; Riley has no navigation path to it and is never shown which of her bookings conflicted -- only the eventual outcome (a cancellation notice, a reschedule notice, or nothing, if kept as an exception) |
| Platform Operator (Support) | No | No | This screen is reachable only inside Talia's own active save flow, not from a read-only account review; support never opens it |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the pending block's values and conflict set are preserved and this screen is restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Review {count} conflicting bookings" with a back arrow (returns to FEAT-17.SPEC-001 without committing the block) and a "Confirm" action button (right-aligned, enabled once every listed booking has a choice).

**Body:** The pending block's own span is restated at the top ("Blocking {date/pattern}, {start}--{end}"), followed by one **row per conflicting booking**, each showing:
- Booking time, service name, and client name
- A three-way choice control: "Cancel," "Reschedule," "Keep as exception" -- exactly one selected per row, with no default pre-selected
- When "Cancel" is selected, an inline note: "Full refund, {client name} is notified."
- When "Reschedule" is selected, an inline note: "You'll pick a new time for {client name} next."
- When "Keep as exception" is selected, an inline note: "This booking stays as booked. It will be flagged on your dashboard as an exception to this block."

**Footer:** None -- Confirm is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column list of rows as described, full width.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-17.SPEC-001 without committing the block or resolving any conflict | Screen closes; the block save is abandoned | Confirmation dialog first (see Edge Cases) |
| Choice control per row | Select one of Cancel / Reschedule / Keep as exception | Captures that booking's chosen outcome | Row shows the corresponding inline consequence note | Selected choice highlighted |
| "Confirm" button | Tap (enabled once every row has a choice) | Submits the full set of per-booking choices to FEAT-17.SPEC-006 (Time Block Conflict Resolution Commit) | Button shows loading state | See Navigation Out for outcome routing |
| "Confirm" button (disabled state) | Tap while one or more rows have no choice | No action | None | Button remains visually disabled; a hint "Choose an outcome for every booking" appears |

### Accessibility Notes

- **Focus order:** Back arrow -> pending block summary -> each conflicting booking row in the order listed (within a row: booking details, choice control, consequence note) -> Confirm.
- **Dynamic-change announcements:** When a row's choice changes, its consequence note update is announced to assistive technology; when Confirm becomes enabled (every row has a choice), that state change is announced.
- **Keyboard alternatives:** Every choice control and the Confirm action are reachable and operable by keyboard.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Awaiting choices | Every row shows its three-way control with nothing selected; Confirm disabled | Screen opens with the conflicting set loaded from FEAT-17.SPEC-004 or FEAT-17.SPEC-005 | Every row receives a choice |
| Ready to confirm | Every row has a choice; Confirm enabled | The last unresolved row receives a choice | Talia taps Confirm, or changes a choice back to none (not offered -- see Edge Cases) |
| Confirming | Confirm button shows a loading indicator, choice controls disabled | Talia taps Confirm | FEAT-17.SPEC-006 returns an outcome |
| Error | Error banner: "Couldn't save your choices. Check your connection and try again." with a Retry control; all selected choices are preserved | FEAT-17.SPEC-006 reports a commit failure | Talia taps Retry and the commit succeeds, or she navigates away |
| Offline/Degraded | Plain message "Connect to the internet to confirm these choices." replaces Confirm's active state; choice controls remain usable but Confirm is disabled | Connectivity is lost while this screen is open, or it is opened while already offline | Connectivity is restored -- Confirm re-enables once every row has a choice |

## Validation Rules

Validation governed by FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules), specifically the never-silently-affect-a-booking rule: Confirm is enabled only once every conflicting booking has an explicit choice, checked on each choice selection and again on Confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-17.SPEC-001 (Create/Edit Time Block) | -- |
| Confirm succeeds, one or more bookings chosen "Cancel" | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | FEAT-30 |
| Confirm succeeds, a booking chosen "Reschedule" | FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | FEAT-30 |
| Confirm succeeds, every remaining booking chosen "Keep as exception" | FEAT-17.SPEC-002 (Manage Time Blocks) | -- |

## Data Model

**Creates:** None directly -- this screen captures choices; FEAT-17.SPEC-006 performs the commit.
**Reads:** Booking (start_time, service, client, state) -- the full conflicting set handed over by FEAT-17.SPEC-004 or FEAT-17.SPEC-005; the pending Time Block's own start, end, and recurrence values.
**Updates:** None directly.
**Deletes:** None directly.

## Business Rules

- Confirm is never enabled while any conflicting booking has no chosen outcome -- the never-silently-affect-a-booking rule (FEAT-17.SPEC-008) is enforced at the UI level here, before FEAT-17.SPEC-006 ever runs.
- Every "Cancel" choice across the set is handed to FEAT-30.SPEC-005 together as one bulk batch; every "Reschedule" choice is handed to FEAT-30.SPEC-002 individually, one booking at a time (the Brief's own Non-Goal: no bulk-reschedule capability exists).
- A "Keep as exception" choice never alters the booking itself -- it only flags it for the Pro's own attention (XBR-11, surfaced via FEAT-12.SPEC-005).

## Edge Cases

- **Talia navigates away (back arrow) with choices already made but not confirmed** -- Confirmation dialog: "Discard these choices and the time block?" with "Discard" and "Keep Reviewing" options; discarding abandons the entire pending block, not just the choices.
- **Talia changes a row's choice after selecting one** -- Allowed at any time before Confirm; the consequence note updates immediately to match the new choice.
- **The conflicting set contains only one booking** -- The screen renders identically with a single row; the header reads "Review 1 conflicting booking."
- **A conflicting booking is cancelled or rescheduled by Talia through an unrelated path (e.g., another device) while this screen is open** -- On Confirm, FEAT-17.SPEC-006 re-validates the set against current booking state; a booking no longer confirmed is dropped from the set with a brief note, and the block commits against the remaining conflicts.
- **Talia taps Confirm twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Network failure during Confirm** -- Error banner as described in States; every selected choice is preserved and Talia can retry without re-selecting anything.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | Navigation (inbound) | Routes here when a single-date save finds conflicts |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | Navigation (inbound) | Routes here when a generated occurrence finds conflicts |
| FEAT-17.SPEC-006 (Time Block Conflict Resolution Commit) | Triggers (outbound) | Confirm submits the full set of per-booking choices |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Never-silently-affect-a-booking rule enforced here |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Navigation (outbound, via FEAT-17.SPEC-006) | Destination for bulk-cancelled bookings |
| FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Navigation (outbound, via FEAT-17.SPEC-006) | Destination for a single rescheduled booking |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Navigation (outbound) | Destination once every conflict is resolved as an exception |

## Analytics and Success Signals

N/A -- this screen captures Talia's per-booking choices; the analytics this feature's Signals track for conflict handling (`time_block_conflict_flagged` and its resolution outcome) are emitted by the automations that detect and commit the resolution (FEAT-17.SPEC-004 and FEAT-17.SPEC-006), not by this review surface itself.

## Acceptance Criteria

**FEAT-17.SPEC-003-AC-01:** Given Talia's new block conflicts with two confirmed bookings, when FEAT-17.SPEC-004 routes her here, then she sees both bookings listed with a three-way choice control each, and Confirm disabled.

**FEAT-17.SPEC-003-AC-02:** Given Talia selects "Cancel" for a conflicting booking, when the row updates, then it shows the note "Full refund, {client name} is notified."

**FEAT-17.SPEC-003-AC-03:** Given Talia selects "Reschedule" for a conflicting booking, when the row updates, then it shows the note "You'll pick a new time for {client name} next."

**FEAT-17.SPEC-003-AC-04:** Given Talia selects "Keep as exception" for a conflicting booking, when the row updates, then it shows the note that the booking stays as booked and will be flagged on her dashboard.

**FEAT-17.SPEC-003-AC-05:** Given Talia has made a choice for every listed booking, when she looks at the header, then the Confirm button is enabled.

**FEAT-17.SPEC-003-AC-06:** Given Talia has left one booking's choice unselected, when she taps the disabled Confirm button, then nothing happens and the hint "Choose an outcome for every booking" appears.

**FEAT-17.SPEC-003-AC-07:** Given Talia has chosen "Cancel" for one booking and "Reschedule" for another and taps Confirm, when FEAT-17.SPEC-006 commits, then she is routed to FEAT-30.SPEC-005 for the cancelled booking's bulk-cancel review.

**FEAT-17.SPEC-003-AC-08:** Given Talia has chosen "Keep as exception" for every conflicting booking and taps Confirm, when FEAT-17.SPEC-006 commits, then the block is created and she is returned to FEAT-17.SPEC-002 with no further screen to visit.

**FEAT-17.SPEC-003-AC-09:** Given Talia is on this screen with choices made but not confirmed, when she taps the back arrow, then a confirmation dialog appears asking "Discard these choices and the time block?"

**FEAT-17.SPEC-003-AC-10:** Given Talia's conflicting set contains exactly one booking, when the screen loads, then the header reads "Review 1 conflicting booking" and one row is shown.

**FEAT-17.SPEC-003-AC-11:** Given a conflicting booking is cancelled through another path while this screen is open, when Talia taps Confirm, then FEAT-17.SPEC-006 drops that booking from the set with a brief note and commits the block against the remaining conflicts.

**FEAT-17.SPEC-003-AC-12:** Given Talia taps Confirm and it is in progress, when she taps Confirm again, then the second tap has no effect and the button remains in its loading state.

**FEAT-17.SPEC-003-AC-13:** Given a network failure occurs during Confirm, when the error banner appears, then every selected choice remains as Talia set it and she can retry without re-selecting.

**FEAT-17.SPEC-003-AC-14:** Given Talia loses connectivity on this screen, when the offline message appears, then choice controls remain usable but Confirm stays disabled until connectivity returns.

**FEAT-17.SPEC-003-AC-15:** Given the Client (Riley) whose booking conflicted with a new block, when Talia's resolution is committed, then Riley is never shown this review screen and only receives the eventual notice matching Talia's chosen outcome (cancellation, reschedule, or nothing if kept as an exception).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (awaiting choices, ready to confirm, confirming, error, offline/degraded) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Time Block Save Commit & Conflict Detection

## Overview

**Name:** Time Block Save Commit & Conflict Detection
**ID:** FEAT-17.SPEC-004
**Type:** Automation
**Purpose:** Validates and commits a created or edited Time Block, checking it against existing confirmed bookings and routing to the Conflict Review screen when any are found.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Re-validating the submitted values against FEAT-17.SPEC-008's field rules at save time
- Checking the block's date/time range against every confirmed Booking for the same Pro Account
- Committing the block immediately when no conflict is found
- Routing to FEAT-17.SPEC-003 when one or more conflicts are found, without committing the block first

**Non-Goals:**
- Generating a recurring pattern's future occurrences -- owned by FEAT-17.SPEC-005, which runs each occurrence through this same conflict scope independently
- Resolving a detected conflict -- owned by FEAT-17.SPEC-006, once Talia's per-booking choices are made on FEAT-17.SPEC-003
- Checking against a client's in-progress checkout hold -- that contention is FEAT-03.SPEC-005's responsibility (first-committed-wins between a block save and a client checkout); this automation only checks against already-confirmed Bookings

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia taps Save on a new or edited single-date block | FEAT-17.SPEC-001 (Create/Edit Time Block) | Fires after the screen's own inline field validation passes | Start, end, label, and (for an edit) the existing Time Block's identity |

## Processing Logic

1. Receive the submitted block values (start, end, optional label) from FEAT-17.SPEC-001, and, for an edit, the identity of the existing Time Block record.
2. Re-validate start, end, and label against FEAT-17.SPEC-008's field rules (end must be after start; label within its length limit).
3. If validation fails, return the specific field error to the triggering screen without proceeding further.
4. For an edit, confirm the existing Time Block record still exists and has not been removed by another session; if it no longer exists, report the removed-record outcome.
5. Read every confirmed Booking (state Confirmed or Awaiting Outcome) belonging to the same Pro Account whose scheduled range overlaps the block's start/end range, per FEAT-17.SPEC-008's definition of a conflicting booking.
6. If zero conflicting bookings are found, commit the Time Block record (create or update) and signal success to the triggering screen.
7. If one or more conflicting bookings are found, hold the block's values in a pending state (not yet committed) and hand the full conflicting set, plus the pending block's values, to FEAT-17.SPEC-003 for Talia's review.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Committed, no conflicts | Zero confirmed bookings overlap the block's range | Time Block record created or updated | Talia is returned to FEAT-17.SPEC-002 with the new/updated block visible | FEAT-17.SPEC-001, FEAT-17.SPEC-002 |
| Routed to conflict review | One or more confirmed bookings overlap the block's range | No commit yet -- the block's values are held pending Talia's choices | Talia is navigated to FEAT-17.SPEC-003 with the conflicting set shown | FEAT-17.SPEC-003 |
| Validation failed | Start/end or label fails FEAT-17.SPEC-008's rules | None | Field-level error shown on FEAT-17.SPEC-001 | FEAT-17.SPEC-001 |
| Edited record no longer exists | The block being edited was removed by another session before this save completed | None | "This time block was removed. Start over?" shown on FEAT-17.SPEC-001 | FEAT-17.SPEC-001 |
| Automation failure | A processing error prevents the save from completing | None | Error banner "Couldn't save this time block. Check your connection and try again." on FEAT-17.SPEC-001, with Retry | FEAT-17.SPEC-001 |

## Data Model

**Reads:** Time Block (existing record, on edit); Booking -- state, start_time, duration, per the Pro Account, to compute overlap against the pending block's range.
**Creates:** Time Block record, on a conflict-free create.
**Updates:** Time Block record, on a conflict-free edit.
**Deletes:** None.

## Business Rules

- Only Bookings in state Confirmed or Awaiting Outcome count as conflicting; Pending Payment, Completed, No-Show, Cancelled, Rescheduled, and Expired (unpaid) bookings never block a save (FEAT-17.SPEC-008: what counts as a conflicting booking).
- The block never commits while a conflict is unresolved -- committing happens either here (zero conflicts) or in FEAT-17.SPEC-006 (once Talia's choices are captured), never both, and never silently (XBR-11, FEAT-17.SPEC-008).
- Validation runs synchronously -- the triggering screen waits for this automation's result before showing any outcome.
- A block save and a client's in-progress checkout for the same instant of time resolve by FEAT-03.SPEC-005's first-committed-wins rule; this automation checks only already-confirmed bookings, not in-progress holds.

## Edge Cases

- **The block's range overlaps a Booking that is Pending Payment (not yet confirmed)** -- Not treated as a conflict; an unpaid, unconfirmed hold does not block the save. If that hold later completes into a Confirmed booking after this block already committed, FEAT-03.SPEC-005's contention rule governs, not this automation.
- **Talia edits a block to shrink its range so a previously-conflicting booking no longer overlaps** -- The re-validation in step 5 finds zero conflicts for the new range and the edit commits immediately, even if the block previously had a conflict on an earlier save attempt.
- **The block's range exactly touches a Booking's start or end with no overlap (adjacent, not overlapping)** -- Not a conflict, per FEAT-17.SPEC-008's boundary definition.
- **Automation processing fails partway through the conflict check** -- No partial commit occurs; the save fails cleanly and the triggering screen shows the automation-failure outcome with entered values preserved.
- **Concurrent trigger firing (Talia saves the same block from two open sessions at effectively the same time)** -- Each save runs its own conflict check independently against the data visible when it starts; the save that commits first wins, and the second save's re-validation (step 4, for an edit) or a subsequent read will reflect the first save's result on its next attempt.
- **Trigger fires while a previous run for the same block is still in flight** -- FEAT-17.SPEC-001's Save button is disabled while a save is in progress, so a second run for the same submission cannot start; a save for a different block proceeds independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Triggered by (inbound) | Fires on Save after field validation passes |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Affects (outbound) | Receives the pending block and conflicting set when conflicts are found |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Field validation and conflicting-booking definition |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | A committed block is read live by the next slot computation |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (inbound) | Governs contention against an in-progress client checkout, outside this automation's own scope |

## Analytics and Success Signals

- **time_block_added** (source: single-date create or edit) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **time_block_conflict_flagged** (conflicting booking count) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-004-AC-01:** Given Talia submits a new block whose range overlaps zero confirmed bookings, when this automation runs, then the block commits immediately and she is returned to FEAT-17.SPEC-002.

**FEAT-17.SPEC-004-AC-02:** Given Talia submits a new block whose range overlaps one confirmed booking, when this automation runs, then the block is held pending and she is routed to FEAT-17.SPEC-003 with that booking listed.

**FEAT-17.SPEC-004-AC-03:** Given Talia submits a block with an end time not after its start time, when this automation re-validates, then the specific field error is returned to FEAT-17.SPEC-001 and nothing commits.

**FEAT-17.SPEC-004-AC-04:** Given Talia edits an existing block that another session already removed, when she saves, then the "removed record" outcome is returned and FEAT-17.SPEC-001 shows "This time block was removed. Start over?"

**FEAT-17.SPEC-004-AC-05:** Given a Booking in the block's range is Pending Payment and not yet confirmed, when the conflict check runs, then that booking is not counted as a conflict and the save proceeds toward commit.

**FEAT-17.SPEC-004-AC-06:** Given Talia edits a block to a smaller range that no longer overlaps a previously conflicting booking, when she saves, then the edit commits immediately with no routing to conflict review.

**FEAT-17.SPEC-004-AC-07:** Given a block's range ends exactly at a confirmed booking's start time, when the conflict check runs, then it is treated as adjacent, not overlapping, and is not a conflict.

**FEAT-17.SPEC-004-AC-08:** Given a processing error occurs during the conflict check, when the automation reports failure, then FEAT-17.SPEC-001 shows "Couldn't save this time block. Check your connection and try again." with entered values preserved.

**FEAT-17.SPEC-004-AC-09:** Given Talia saves the same block from two open sessions at effectively the same time, when both saves run, then the one that commits first succeeds and the second reflects that result on its own re-validation.

**FEAT-17.SPEC-004-AC-10:** Given a block commits with zero conflicts, when the next slot computation runs, then FEAT-03.SPEC-001 reads the new block and removes its time from the bookable list.

**FEAT-17.SPEC-004-AC-11:** Given a block's save is in flight, when FEAT-17.SPEC-001's Save button is tapped again, then no second run starts for that same submission.

**FEAT-17.SPEC-004-AC-12:** Given a conflict is found and Talia is routed to FEAT-17.SPEC-003, when the routing occurs, then the `time_block_conflict_flagged` event is emitted with the conflicting booking count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (committed, routed, validation failed, removed record, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Recurring Time Block Occurrence Generation

## Overview

**Name:** Recurring Time Block Occurrence Generation
**ID:** FEAT-17.SPEC-005
**Type:** Automation
**Purpose:** Generates and maintains the future dated occurrences of a recurring block pattern (e.g., every Sunday), running each new occurrence through the same conflict detection as a single-date block.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Turning a recurrence pattern (day-of-week, start/end time-of-day, label) into individual dated Time Block occurrence records
- Keeping the generated horizon current as time passes (rolling generation)
- Regenerating not-yet-elapsed future occurrences when Talia edits the pattern's span, day-of-week, or label
- Running every newly generated occurrence through the same conflict scope as FEAT-17.SPEC-004

**Non-Goals:**
- Capturing the recurrence pattern itself -- owned by FEAT-17.SPEC-001, which this automation only reads
- Retiring an occurrence once its own end time passes -- owned by FEAT-17.SPEC-007
- Deriving a recurring pattern automatically from the Pro's personal calendar -- excluded per the Brief's own Non-Goals: calendar busy time is FEAT-04's distinct responsibility and is never converted into a Time Block
- Cross-Pro or shared recurrence patterns -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia saves a new recurring pattern | FEAT-17.SPEC-001 (Create/Edit Time Block) | Fires when the Repeats toggle is on and the save commits (via FEAT-17.SPEC-004, zero conflicts on the first occurrence) | Day-of-week, start/end time-of-day, label, first occurrence date |
| Talia edits an existing recurring pattern's span, day-of-week, or label | FEAT-17.SPEC-001 (Create/Edit Time Block) | Fires when the edit to a pattern-defining block commits (via FEAT-17.SPEC-004) | Updated day-of-week, start/end time-of-day, label |
| The Pro Account's booking_horizon setting changes | FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Fires when the horizon is extended, since a longer horizon means further-out occurrences must now be generated | Current booking_horizon value |
| Rolling generation check | System (time-based) | Fires periodically to keep each active pattern's generated occurrences current out to the Pro Account's current booking_horizon | Each active pattern's definition; current date |

## Processing Logic

1. Read the recurrence pattern's definition: day-of-week, start/end time-of-day, label, and the Pro Account's current booking_horizon (Availability Rule).
2. Determine every future date, out to the booking_horizon from today, that matches the pattern's day-of-week and does not already have a generated occurrence.
3. For each such date, construct a candidate occurrence (that date's start/end from the pattern's time-of-day, carrying the pattern's label).
4. Run each candidate occurrence through the same validation and conflict scope as FEAT-17.SPEC-004 (end-after-start already guaranteed by the pattern; conflicting-booking check against confirmed Bookings on that date).
5. If a candidate has zero conflicts, commit it as a Time Block occurrence record referencing the parent pattern.
6. If a candidate conflicts with one or more confirmed bookings, hold that single occurrence pending and hand it to FEAT-17.SPEC-003 for Talia's review, exactly as a single-date save would; generation continues independently for the pattern's other candidate dates.
7. When the pattern itself is edited (span, day-of-week, or label changed), delete every not-yet-elapsed generated occurrence that has not already had a conflict resolved by Talia, and repeat steps 2--6 under the new definition; occurrences whose own end time has already passed are never touched (they carry no retention requirement of their own, but they are also simply historical and out of scope for regeneration).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Occurrences generated, no conflicts | Every candidate date's occurrence overlaps zero confirmed bookings | New Time Block occurrence records created out to the current booking_horizon | New occurrences appear on FEAT-17.SPEC-002 and the Pro's schedule (FEAT-12) on next view | FEAT-17.SPEC-002, FEAT-12 |
| One or more occurrences conflict | A candidate date's occurrence overlaps a confirmed booking | Non-conflicting candidates commit; the conflicting one is held pending | Talia is routed to FEAT-17.SPEC-003 for the conflicting occurrence | FEAT-17.SPEC-003 |
| Pattern edited, occurrences regenerated | Talia changes span, day-of-week, or label on an existing pattern | Not-yet-elapsed, not-yet-conflict-resolved future occurrences are deleted and regenerated under the new definition | Updated occurrences reflect the new definition on next view of FEAT-17.SPEC-002 | FEAT-17.SPEC-002, FEAT-12 |
| Horizon extended, more occurrences generated | The Pro Account's booking_horizon increases | Additional future occurrences generated up to the new horizon | New further-out occurrences appear on next view | FEAT-17.SPEC-002 |
| No action needed | Every date within the current horizon already has a generated occurrence | None | None -- silent, logged internally | -- |
| Generation failure | A processing error prevents an occurrence from being created for one or more candidate dates | No occurrence created for the failed date(s); other candidate dates are unaffected | No blocking user feedback -- the gap is not user-visible until it would matter; surfaced to the Pro only indirectly if a client later books a slot the Pro expected blocked, which is out of this automation's scope to detect | FEAT-17.SPEC-002 |

## Data Model

**Reads:** Time Block (the parent pattern's definition: day-of-week, start/end time-of-day, label; existing generated occurrences to avoid duplicate generation); Availability Rule -- booking_horizon field, per Pro Account; Booking -- state, start_time, duration, for the conflict check on each candidate date.
**Creates:** Time Block occurrence records -- one per generated future date, referencing the parent pattern.
**Updates:** None to the parent pattern record itself; regeneration deletes and recreates affected future occurrence records.
**Deletes:** Not-yet-elapsed, not-yet-conflict-resolved future occurrence records, when the parent pattern is edited (step 7).

## Business Rules

- Occurrences are generated only out to the Pro Account's current booking_horizon (Availability Rule) -- matching the range within which any slot can be booked at all, so a block is never generated for a date no client could book against anyway.
- Each generated occurrence is checked against the same conflicting-booking definition as a single-date block (FEAT-17.SPEC-008) -- recurrence introduces no separate conflict rule.
- An occurrence whose conflict Talia has already resolved (cancel, reschedule, or keep-as-exception) is never silently deleted or regenerated by a later pattern edit; only not-yet-resolved future occurrences are replaced.
- Regeneration never touches an occurrence whose own end time has already passed -- past occurrences are historical, per the entity's own no-retention lifecycle.

## Edge Cases

- **A candidate date falls on a day the Pro Account's booking_horizon does not yet reach** -- Not generated in this run; generated automatically once the rolling generation check runs again and the horizon (relative to today) reaches that date.
- **Talia's booking_horizon shrinks (a narrower horizon is set)** -- Already-generated occurrences beyond the new, shorter horizon are left in place (a Pro-set horizon change never deletes an already-committed block); no new occurrences are generated beyond the new horizon going forward.
- **A pattern is edited while one of its future occurrences has an unresolved conflict on FEAT-17.SPEC-003** -- That occurrence is left untouched by the regeneration (its resolution takes priority); it is included in step 7's exclusion because it has not yet had a conflict resolved by Talia at the moment of the edit, so it is preserved rather than deleted out from under an in-progress review.
- **Concurrent trigger firing (a pattern edit and the rolling generation check for the same pattern run at effectively the same time)** -- The pattern edit's regeneration (steps 2--7) is authoritative and its result is what persists; the rolling check's own run against the same pattern, if it started first, has its results superseded once the edit's regeneration commits.
- **Trigger fires while a previous generation run for the same pattern is still in flight** -- A second run for the same pattern is not started until the first completes; a run for a different pattern proceeds independently.
- **A pattern's every candidate date within the horizon already conflicts with the same recurring confirmed booking** -- Each conflicting date is routed to FEAT-17.SPEC-003 independently as its own occurrence; Talia resolves each one separately, since the Brief provides no bulk-recurring-conflict resolution.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Triggered by (inbound) | A saved or edited recurrence pattern triggers generation or regeneration |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Affects (outbound) | A conflicting generated occurrence routes here |
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | References (inbound) | Shares the same conflict-check logic, applied per candidate occurrence |
| FEAT-17.SPEC-007 (Time Block Removal & Expiry) | Affects (outbound) | Generated occurrences are later retired there as their own end time passes |
| FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Triggered by (inbound) | A booking_horizon change re-triggers generation |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | Each committed occurrence is read live by the next slot computation |

## Analytics and Success Signals

- **time_block_added** (source: recurring occurrence generation) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **time_block_conflict_flagged** (source: recurring occurrence generation) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-005-AC-01:** Given Talia saves a new "every Sunday" pattern with a booking_horizon of several weeks, when this automation runs, then it generates one occurrence for each future Sunday within the horizon with zero conflicts.

**FEAT-17.SPEC-005-AC-02:** Given one candidate Sunday's occurrence conflicts with a confirmed booking, when generation processes that date, then Talia is routed to FEAT-17.SPEC-003 for that occurrence while the other Sundays' occurrences commit normally.

**FEAT-17.SPEC-005-AC-03:** Given Talia edits her existing pattern's end time, when the edit commits, then every not-yet-elapsed, not-yet-conflict-resolved future occurrence is regenerated with the new end time.

**FEAT-17.SPEC-005-AC-04:** Given an occurrence's own end time has already passed, when Talia edits the pattern, then that already-elapsed occurrence is left completely unchanged.

**FEAT-17.SPEC-005-AC-05:** Given Talia extends her booking_horizon, when the rolling generation check next runs, then additional future occurrences are generated out to the new horizon.

**FEAT-17.SPEC-005-AC-06:** Given Talia narrows her booking_horizon, when the change takes effect, then already-generated occurrences beyond the new horizon remain in place.

**FEAT-17.SPEC-005-AC-07:** Given a future occurrence has an unresolved conflict currently open on FEAT-17.SPEC-003, when Talia edits the parent pattern at that moment, then that specific occurrence is excluded from regeneration and left untouched.

**FEAT-17.SPEC-005-AC-08:** Given a generation run for a pattern is still in flight, when the rolling generation check fires again for the same pattern, then no second run starts until the first completes.

**FEAT-17.SPEC-005-AC-09:** Given every date within the horizon already has a generated occurrence, when the rolling generation check runs, then no new occurrences are created and nothing is shown to Talia.

**FEAT-17.SPEC-005-AC-10:** Given a processing error prevents one candidate date's occurrence from being created, when the run completes, then the other candidate dates' occurrences are unaffected and commit normally.

**FEAT-17.SPEC-005-AC-11:** Given a generated occurrence commits with zero conflicts, when the next slot computation runs, then FEAT-03.SPEC-001 reads it and removes that date's time from the bookable list.

**FEAT-17.SPEC-005-AC-12:** Given a pattern edit's regeneration and the rolling generation check for the same pattern run at effectively the same time, when both complete, then the pattern edit's result is what persists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 6 (generated, conflict, regenerated, horizon-extended, no-action, failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Time Block Conflict Resolution Commit

## Overview

**Name:** Time Block Conflict Resolution Commit
**ID:** FEAT-17.SPEC-006
**Type:** Automation
**Purpose:** Commits Talia's explicit choice on a conflicting booking set -- hand off to cancellation, hand off to reschedule, or mark the booking as a kept exception -- and finalizes the block once every conflicting booking has a resolved outcome.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Committing the pending Time Block (or occurrence) once Talia's per-booking choices are submitted from FEAT-17.SPEC-003
- Handing every "Cancel" booking to bulk cancellation as one batch, and every "Reschedule" booking individually to Pro-initiated reschedule
- Marking every "Keep as exception" booking as a resolved exception, flagged for the Pro's attention
- Tracking each conflicting booking's resolved/unresolved state until the whole set is resolved

**Non-Goals:**
- Presenting the choices to Talia -- owned by FEAT-17.SPEC-003, which this automation receives its input from
- Performing the cancellation or reschedule itself -- owned by FEAT-30.SPEC-005 and FEAT-30.SPEC-002 respectively; this automation only hands off and tracks completion
- Deciding a bulk-reschedule outcome -- excluded per the Brief's Non-Goals: FEAT-30 reschedules one booking at a time only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia taps Confirm on the conflict review screen | FEAT-17.SPEC-003 (Time Block Conflict Review) | Fires once every conflicting booking in the set has a chosen outcome | The pending block's (or occurrence's) values, and the per-booking choice (cancel / reschedule / keep as exception) for every conflicting Booking |

## Processing Logic

1. Receive the pending block (or occurrence) values and the full set of per-booking choices from FEAT-17.SPEC-003.
2. Re-validate each conflicting booking's current state (it may have changed since FEAT-17.SPEC-003 loaded it); drop any booking that is no longer Confirmed or Awaiting Outcome from the set, since it is no longer a conflict.
3. Commit the Time Block (or occurrence) record immediately -- the block itself does not wait for the conflicting bookings' outcomes to complete, per XBR-11: setup changes are never blocked on how a conflict resolves.
4. Group every remaining booking chosen "Cancel" into one batch and hand it to FEAT-30.SPEC-005 (Cancel Several Bookings at Once); mark each as pending resolution until FEAT-30.SPEC-005 reports completion.
5. For every booking chosen "Reschedule," hand it individually to FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated); mark each as pending resolution until FEAT-30.SPEC-002 reports completion.
6. For every booking chosen "Keep as exception," mark it resolved immediately as a kept exception, and raise an attention signal for it (consumed by FEAT-12.SPEC-005).
7. Track the resolved/unresolved status of every booking in the original conflicting set.
8. Once every booking in the set reaches a resolved outcome (cancelled, rescheduled, or kept as exception), mark the block fully finalized with no remaining pending conflicts.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Block committed, all bookings resolved immediately | Every booking is "Keep as exception" (no cancel or reschedule hand-offs pending) | Block committed; every booking flagged as a kept exception | Talia is returned to FEAT-17.SPEC-002 | FEAT-17.SPEC-002, FEAT-12.SPEC-005 |
| Block committed, hand-offs pending | One or more bookings chosen "Cancel" or "Reschedule" | Block committed; cancel batch handed to FEAT-30.SPEC-005; reschedule bookings handed individually to FEAT-30.SPEC-002 | Talia is routed to the first pending hand-off screen | FEAT-30.SPEC-005, FEAT-30.SPEC-002 |
| Every pending hand-off completes | All handed-off bookings report a completed outcome from FEAT-30 | Block marked fully finalized | No separate notice -- the block simply shows no pending conflicts on FEAT-17.SPEC-002 | FEAT-17.SPEC-002 |
| A booking dropped from the set before resolution | Re-validation (step 2) finds the booking is no longer Confirmed or Awaiting Outcome | That booking is excluded from any hand-off | No user-visible change beyond a smaller conflicting set | FEAT-17.SPEC-003 |
| A hand-off is abandoned or left incomplete | Talia navigates away from FEAT-30.SPEC-005 or FEAT-30.SPEC-002 before completing it | That booking remains unresolved | The booking is flagged on the dashboard as an unresolved exception (per the touchpoint: "leaves unresolved" feeds an attention flag) until Talia revisits and completes the hand-off | FEAT-12.SPEC-005 |
| Commit failure | A processing error prevents the block itself from committing | No block created; no bookings altered | Error banner on FEAT-17.SPEC-003: "Couldn't save your choices. Check your connection and try again." with Retry | FEAT-17.SPEC-003 |

## Data Model

**Reads:** Booking -- current state, for re-validation before hand-off.
**Creates:** Time Block (or occurrence) record -- committed from the pending values held by FEAT-17.SPEC-004/FEAT-17.SPEC-005.
**Updates:** Booking -- no direct field write by this automation; state changes for cancel/reschedule are owned by FEAT-30.SPEC-005/FEAT-30.SPEC-002. This automation writes only the resolution tracking (resolved/unresolved, and outcome kind) associated with the block's conflict set.
**Deletes:** None.

## Business Rules

- The block never waits for a cancel or reschedule hand-off to complete before it commits -- committing the block and resolving its conflicting bookings are decoupled, so the Pro's setup change is never blocked by a client-facing process (XBR-11).
- A "Keep as exception" booking is never altered in any field -- only a resolution-tracking flag is set (FEAT-17.SPEC-008: never-silently-affect-a-booking).
- An unresolved booking (a hand-off Talia abandoned) is always surfaced on the dashboard -- it is never silently dropped (XBR-11, FEAT-12.SPEC-005).
- Every "Cancel" choice in one Confirm submission is batched into exactly one bulk-cancel hand-off; every "Reschedule" choice is its own individual hand-off -- these are never merged or split further.

## Edge Cases

- **A booking chosen "Cancel" is no longer Confirmed by the time this automation runs (e.g., the client cancelled it themselves moments earlier)** -- Dropped from the cancel batch in step 2; the block commits as if that booking had never conflicted.
- **Talia completes the reschedule hand-off for one booking but abandons the bulk-cancel hand-off for others** -- The rescheduled booking resolves normally; the abandoned cancel batch's bookings remain unresolved and are flagged per the abandoned-hand-off outcome.
- **Every booking in the set is dropped during re-validation (step 2)** -- The block commits with zero remaining conflicts and is immediately finalized, with no hand-off screen shown at all.
- **Concurrent trigger firing (Talia confirms the same conflict set from two open sessions)** -- The first commit to complete wins; the second's re-validation (step 2) finds the block already exists and the bookings already resolved, so it reports the already-finalized outcome rather than duplicating any hand-off.
- **Trigger fires while a previous resolution commit for the same block is still in flight** -- FEAT-17.SPEC-003's Confirm button is disabled while a submission is in progress, so a second run for the same submission cannot start.
- **A kept-exception booking is later cancelled by Talia through an unrelated path (FEAT-30.SPEC-001)** -- The exception flag raised here is cleared by that cancellation's own completion, since the booking it referred to no longer exists in a state that needs flagging; this automation itself performs no further action once the flag has been raised.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Triggered by (inbound) | Confirm submits the per-booking choices |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Affects (outbound) | Receives the bulk-cancel batch |
| FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Affects (outbound) | Receives each individual reschedule hand-off |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | Affects (outbound) | Receives the kept-exception and unresolved-conflict flags |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Never-silently-affect-a-booking rule enforced throughout |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Affects (outbound) | Destination once the block is committed and (fully or partially) resolved |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The committed block is read live once finalized |

## Analytics and Success Signals

- **time_block_conflict_resolved** (outcome: cancel / reschedule / keep_as_exception / unresolved, per booking) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-006-AC-01:** Given Talia confirms choices for two conflicting bookings, one "Cancel" and one "Reschedule," when this automation runs, then the block commits immediately and Talia is routed to the bulk-cancel hand-off first.

**FEAT-17.SPEC-006-AC-02:** Given Talia confirms "Keep as exception" for every conflicting booking, when this automation runs, then the block commits, every booking is flagged as a kept exception, and Talia returns directly to FEAT-17.SPEC-002.

**FEAT-17.SPEC-006-AC-03:** Given a booking chosen "Cancel" is no longer Confirmed when this automation runs, when re-validation occurs, then that booking is dropped from the batch and the block commits as if it had never conflicted.

**FEAT-17.SPEC-006-AC-04:** Given every booking in the conflicting set is dropped during re-validation, when this automation completes, then the block commits with zero remaining conflicts and no hand-off screen is shown.

**FEAT-17.SPEC-006-AC-05:** Given Talia abandons the bulk-cancel hand-off after confirming her choices, when she navigates away without completing it, then those bookings remain unresolved and are flagged on her dashboard via FEAT-12.SPEC-005.

**FEAT-17.SPEC-006-AC-06:** Given every booking in a set eventually reaches a resolved outcome, when the last one resolves, then the block is marked fully finalized with no remaining pending conflicts.

**FEAT-17.SPEC-006-AC-07:** Given Talia keeps a booking as an exception, when the flag is raised, then no field on that Booking record is altered.

**FEAT-17.SPEC-006-AC-08:** Given a commit failure occurs while this automation runs, when the failure is reported, then FEAT-17.SPEC-003 shows "Couldn't save your choices. Check your connection and try again." with Retry, and no block or booking change persists.

**FEAT-17.SPEC-006-AC-09:** Given Talia confirms the same conflict set from two open sessions at effectively the same time, when both submissions run, then the first to complete commits and the second reports the already-finalized outcome without duplicating any hand-off.

**FEAT-17.SPEC-006-AC-10:** Given a resolution commit for a block is already in flight, when FEAT-17.SPEC-003's Confirm is tapped again for that same submission, then no second run starts.

**FEAT-17.SPEC-006-AC-11:** Given a kept-exception booking is later cancelled through FEAT-30.SPEC-001, when that cancellation completes, then the exception flag raised by this automation is cleared.

**FEAT-17.SPEC-006-AC-12:** Given the block commits regardless of pending hand-offs, when Talia checks FEAT-17.SPEC-002 immediately after confirming, then the new block is already visible even before any cancel or reschedule hand-off has completed.

**FEAT-17.SPEC-006-AC-13:** Given a booking's resolution reaches an outcome, when it does, then the `time_block_conflict_resolved` event is emitted naming that outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 (all resolved immediately, hand-offs pending, hand-offs complete, booking dropped, hand-off abandoned, commit failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Time Block Removal & Expiry

## Overview

**Name:** Time Block Removal & Expiry
**ID:** FEAT-17.SPEC-007
**Type:** Automation
**Purpose:** Deletes a block Talia removes early, or automatically retires a block once its end time has passed, in either case restoring that time to bookable availability immediately.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Hard-deleting a Time Block (single-date or one generated occurrence) on Talia's explicit removal
- Automatically retiring a block or occurrence once its own end time passes
- Restoring the freed time to bookable availability immediately in both cases

**Non-Goals:**
- Presenting the removal confirmation to Talia -- owned by FEAT-17.SPEC-002, which triggers this automation
- Any retention, archive, or undo path for a removed or expired block -- excluded per the Brief's own Non-Goals: a Time Block carries no historical-record requirement of its own (Data Notes: displayed only on the Pro's own schedule view), unlike Booking's SC-22 retention
- Retiring the recurrence pattern definition itself -- a pattern has no "end time" of its own to expire; only its individual generated occurrences expire here, one at a time

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms removal of a block | FEAT-17.SPEC-002 (Manage Time Blocks) | Fires when Talia taps "Remove" on the confirmation dialog | The Time Block's identity |
| A block's or occurrence's own end time passes | System (time-based) | Fires when the current time crosses a Time Block record's end value and it has not already been removed | The Time Block's identity and end value |

## Processing Logic

1. Receive the triggering event: either an explicit removal request (with the block's identity) or a scheduled expiry check crossing a block's end time.
2. For an explicit removal, confirm the block still exists (it may have already expired or been removed by another session); if it does not, report the already-gone outcome.
3. For a scheduled expiry, identify every Time Block whose end value has just passed and which has not already been removed.
4. Delete the identified Time Block record (hard delete -- no retention window).
5. Signal the deletion so the next slot computation reads the current, updated set of Time Blocks with the freed time available immediately.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Explicit removal succeeds | Talia confirms removal and the block exists | Time Block record deleted | Row disappears from FEAT-17.SPEC-002 immediately | FEAT-17.SPEC-002, FEAT-03.SPEC-001 |
| Automatic expiry | A block's or occurrence's end time passes | Time Block record deleted | No notice shown -- routine background retirement; the block simply no longer appears on next view of FEAT-17.SPEC-002 or FEAT-12 | FEAT-17.SPEC-002, FEAT-12, FEAT-03.SPEC-001 |
| Already gone | The block targeted for explicit removal no longer exists (already expired or removed elsewhere) | None | Row is simply absent from the list on next refresh; no error shown, since the Pro's intended outcome (the time being free) is already true | FEAT-17.SPEC-002 |
| Removal failure | A processing error prevents the delete from completing | None | Inline error on FEAT-17.SPEC-002: "Couldn't remove this time block. Try again." with the remove affordance still available | FEAT-17.SPEC-002 |

## Data Model

**Reads:** Time Block -- identity and end value, to confirm existence (explicit removal) or to identify newly expired records (scheduled check).
**Creates:** None.
**Updates:** None.
**Deletes:** Time Block record -- hard delete, in both the explicit-removal and automatic-expiry outcomes.

## Business Rules

- Deletion never cascades to a conflicting Booking kept as an exception -- that booking is left completely untouched (Validation & Limits: a block cannot silently delete a booking).
- No retention or undo path applies to a removed or expired block -- this is an intentional lifecycle decision, since the entity carries no historical-record requirement of its own.
- The freed time becomes bookable again within roughly one second of removal or expiry (ASMP-21), since the slot computation reads Time Block data live with no caching (FEAT-03.SPEC-001).
- The Active -> Expired state derivation this automation acts on (a block expires the moment its own end value passes, system-derived only, never overridable by Talia) is defined by FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules), which lists this automation as an enforcing spec.
- A recurring pattern's individual occurrences expire independently of one another and of the pattern definition -- expiring one occurrence never affects the pattern's other future occurrences (owned by FEAT-17.SPEC-005) or already-generated ones.

## Edge Cases

- **Talia removes a block at the exact moment its end time passes (a race between explicit removal and automatic expiry)** -- Whichever action completes first performs the delete; the other finds the block already gone and reports the already-gone outcome with no error.
- **A generated recurring occurrence expires while its parent pattern is being edited on FEAT-17.SPEC-001** -- The already-elapsed occurrence is unaffected by the concurrent edit (FEAT-17.SPEC-005's regeneration never touches occurrences whose end time has passed); this automation's expiry proceeds independently.
- **The scheduled expiry check runs while a large number of occurrences cross their end time at once (e.g., many Sundays' worth reach end-of-day together)** -- Each is deleted independently; there is no ordering dependency between them, since a solo Pro's block set stays small (Non-Functional Notes: data volumes).
- **Concurrent trigger firing (Talia's explicit removal and the scheduled expiry check target the same block at effectively the same time)** -- The first to complete performs the delete; the second's existence check (step 2) or expiry scan (step 3) finds the record already gone and produces no error, since both actions intend the same outcome.
- **Trigger fires while a previous removal run for the same block is still in flight** -- A second run for the same block's identity is not started while the first is in flight; the second, once the first completes, finds the record already gone and reports the already-gone outcome.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-002 (Manage Time Blocks) | Triggered by (inbound) | Confirmed removal fires this automation |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | References (inbound) | Generated occurrences are the records this automation later expires |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | Reads the current Time Block set live -- a deletion here is reflected on the next computation with no separate propagation step |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A removed or expired block no longer appears on the Pro's schedule view |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules) | Governed by | Defines the Active -> Expired state derivation that the expiry path enforces |

## Analytics and Success Signals

- **time_block_removed** (source: explicit removal or automatic expiry) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-007-AC-01:** Given Talia confirms removal of an existing block, when this automation runs, then the record is hard-deleted and the row disappears from FEAT-17.SPEC-002 immediately.

**FEAT-17.SPEC-007-AC-02:** Given a single-date block's end time passes with no Pro action, when the scheduled expiry check runs, then the record is deleted automatically with no notice shown to Talia.

**FEAT-17.SPEC-007-AC-03:** Given a generated recurring occurrence's end time passes, when the scheduled expiry check runs, then only that occurrence is deleted, and the pattern's other future occurrences are unaffected.

**FEAT-17.SPEC-007-AC-04:** Given Talia attempts to remove a block that already expired moments earlier, when this automation checks its existence, then no error is shown and the row is simply absent on the next refresh.

**FEAT-17.SPEC-007-AC-05:** Given a removed block had a confirmed booking kept as an exception, when the deletion completes, then that booking is left completely untouched.

**FEAT-17.SPEC-007-AC-06:** Given a block is deleted (removal or expiry), when the next slot computation runs, then FEAT-03.SPEC-001 reads the current Time Block set and the freed time appears bookable within roughly one second.

**FEAT-17.SPEC-007-AC-07:** Given a processing error occurs during an explicit removal, when the failure is reported, then FEAT-17.SPEC-002 shows "Couldn't remove this time block. Try again." with the remove affordance still available.

**FEAT-17.SPEC-007-AC-08:** Given Talia's explicit removal and the scheduled expiry check target the same block at effectively the same time, when both run, then whichever completes first performs the delete and the other reports no error.

**FEAT-17.SPEC-007-AC-09:** Given many recurring occurrences reach their end time together, when the scheduled expiry check runs, then each is deleted independently with no ordering dependency between them.

**FEAT-17.SPEC-007-AC-10:** Given a removal run for a block is already in flight, when a second removal request for the same block arrives, then no second run starts, and it later finds the record already gone.

**FEAT-17.SPEC-007-AC-11:** Given a block is removed or expires, when the deletion completes, then the `time_block_removed` event is emitted.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 (explicit removal, automatic expiry, already gone, removal failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Time Block Validation & Conflict Handling Rules

## Overview

**Name:** Time Block Validation & Conflict Handling Rules
**ID:** FEAT-17.SPEC-008
**Type:** Logic/Rule
**Purpose:** Defines the shared rules every screen and automation in this feature references: end-after-start validation, what counts as a conflicting booking, the never-silently-affect-a-booking rule, and who can see or act on a block.
**Parent Feature:** FEAT-17 -- Manual Time Blocking
**Governed Entity:** Time Block

## Scope and Non-Goals

**In Scope:**
- Field-level validation rules for every Time Block field
- The end-after-start cross-field rule
- The definition of what counts as a conflicting confirmed booking
- Authorization rules for every action on a Time Block, per role
- Default values and derived fields (state, resolution tracking)
- The never-silently-affect-a-booking rule governing every conflict resolution outcome

**Non-Goals:**
- The processing logic that detects conflicts at save time -- owned by FEAT-17.SPEC-004 and FEAT-17.SPEC-005, which apply the definitions in this spec
- The processing logic that commits a resolution -- owned by FEAT-17.SPEC-006, which applies the never-silently-affect-a-booking rule defined here
- UI layout and interaction behavior for displaying validation errors -- defined in FEAT-17.SPEC-001, FEAT-17.SPEC-002, and FEAT-17.SPEC-003, which reference this spec for the rules but own their own display behavior

## Governed Entity

**Entity:** Time Block
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| start | date/time | The block's start, in the Pro's account timezone |
| end | date/time | The block's end, in the Pro's account timezone; must be after start |
| recurrence | enum (none \| weekly-on-day) | Optional pattern; when set, defines the day-of-week this block repeats on |
| label | text | Optional, private to the Pro; describes the reason for the block |
| state | derived enum (Active \| Expired) | Whether this block (or occurrence) still blocks availability |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-17.SPEC-001 | Create/Edit Time Block | Field validation on blur (Date, Start time, End time, Label) and on form submit |
| FEAT-17.SPEC-002 | Manage Time Blocks | Authorization on screen entry (Support's view-only rendering); no field validation (no form fields) |
| FEAT-17.SPEC-003 | Time Block Conflict Review | The never-silently-affect-a-booking rule, enforced by disabling Confirm until every conflicting booking has a choice |
| FEAT-17.SPEC-004 | Time Block Save Commit & Conflict Detection | Re-validation of start/end/label at save time; the conflicting-booking definition, applied to the save's conflict scope |
| FEAT-17.SPEC-005 | Recurring Time Block Occurrence Generation | The same field validation and conflicting-booking definition, applied to each generated occurrence |
| FEAT-17.SPEC-006 | Time Block Conflict Resolution Commit | The never-silently-affect-a-booking rule, enforced by never altering a Booking field on a kept-exception outcome |
| FEAT-17.SPEC-007 | Time Block Removal & Expiry | The state derivation (Active -> Expired) that this automation acts on |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| start | Required; a valid date and time | Always | On blur, on submit | "Choose a start date and time" | Yes |
| end | Required; a valid date and time; must be strictly after start | Always | On blur, on submit | "End time must be after the start time" | Yes |
| recurrence | Optional; when set, must name exactly one day of the week | When the Repeats toggle is on | On selection, on submit | "Choose which day this repeats on" | Yes |
| label | Optional; no validation beyond a maximum of 100 characters | Always | On blur, on submit | "Label must be 100 characters or fewer" | Yes |
| state | Not user-entered -- system-derived only; no input validation applies | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| End-after-start | start, end | end must be strictly later than start (equal values are invalid; a zero-length block is not permitted) | "End time must be after the start time" |
| Recurrence requires a day | recurrence, start | If the Repeats toggle is on, a day-of-week must be selected; it defaults to the day-of-week of the entered start date but Talia may change it | "Choose which day this repeats on" |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Time Block | The Pro (Talia) | Always, for her own Pro Account | -- |
| Create Time Block | The Client (Riley) | Never | No navigation path to the create screen exists in Riley's experience (FEAT-17.SPEC-001) |
| Create Time Block | Platform Operator (Support) | Never | The create/edit screen is outside support's granted session scope; no control or route to it is ever rendered (FEAT-17.SPEC-001) |
| View Time Block (single or list) | The Pro (Talia) | Always, for her own blocks only | -- |
| View Time Block (single or list), including label | Platform Operator (Support) | Always, for the one Pro account under active review, read-only | -- |
| View Time Block | The Client (Riley) | Never | Riley never sees a block directly -- only its absence from the bookable slot list on the public booking page |
| Edit Time Block (span, recurrence, label) | The Pro (Talia) | Always, for her own blocks only | -- |
| Edit Time Block | Platform Operator (Support) | Never | No edit control is rendered for Support anywhere in this feature |
| Edit Time Block | The Client (Riley) | Never | No navigation path exists in Riley's experience |
| Remove Time Block | The Pro (Talia) | Always, for her own blocks only | -- |
| Remove Time Block | Platform Operator (Support) | Never | No remove control is rendered for Support (FEAT-17.SPEC-002) |
| Remove Time Block | The Client (Riley) | Never | No navigation path exists in Riley's experience |
| Choose a conflict resolution outcome (cancel / reschedule / keep as exception) | The Pro (Talia) | Always, for conflicts against her own blocks | -- |
| Choose a conflict resolution outcome | Platform Operator (Support) | Never | FEAT-17.SPEC-003 is reachable only inside Talia's own active save flow; support never opens it |
| Choose a conflict resolution outcome | The Client (Riley) | Never | Riley is never shown which of her bookings conflicted, or offered a choice; she only receives the eventual notice matching Talia's chosen outcome |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state | Active | On create (single-date block or each generated occurrence) | No -- system-derived only |
| state | Expired | The moment the block's own end value passes (FEAT-17.SPEC-007) | No -- system-derived only |
| recurrence's day-of-week (initial default) | The day-of-week of the entered start date | When the Repeats toggle is first turned on | Yes -- Talia may pick a different day before saving |

## Business Rules

- **What counts as a conflicting booking:** A Booking whose state is Confirmed or Awaiting Outcome, belonging to the same Pro Account, whose scheduled range (start_time through start_time + duration) overlaps the Time Block's [start, end) range. A Booking that is Pending Payment, Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, or Expired (unpaid) never counts as a conflict.
- **Overlap boundary:** Two ranges overlap only when they share at least one instant of time; a block ending exactly at a booking's start (or a booking ending exactly at a block's start) is adjacent, not overlapping, and is not a conflict.
- **The never-silently-affect-a-booking rule:** A confirming Booking flagged as conflicting is never automatically cancelled, rescheduled, or otherwise altered by this feature. Only Talia's explicit, per-booking choice on FEAT-17.SPEC-003 determines its outcome, and until every conflicting booking has a chosen outcome, the conflict remains open and visible (XBR-11).
- **A block never commits over an unresolved conflict silently** -- FEAT-17.SPEC-004 and FEAT-17.SPEC-005 always route a conflicting save or occurrence to FEAT-17.SPEC-003 rather than committing it outright; the block commits only once Talia's choices are captured (FEAT-17.SPEC-006).
- **A "kept as exception" booking's fields are never modified** -- only a resolution-tracking flag is set, and the exception is surfaced to Talia via FEAT-12.SPEC-005 (XBR-11).
- **First-committed-wins against an in-progress client checkout** -- when a block save and a client's checkout hold contend for the same instant, FEAT-03.SPEC-005's contention rule (not this spec) governs which one wins; this spec's conflicting-booking definition applies only to already-confirmed bookings.
- **Recurrence introduces no separate conflict rule** -- every generated occurrence (FEAT-17.SPEC-005) is checked against the exact same conflicting-booking definition as a single-date block.

## Edge Cases

- **start and end are identical** -- Invalid; a zero-length block fails the end-after-start rule ("End time must be after the start time").
- **end is exactly one minute after start** -- Valid; the minimum block length is not otherwise restricted.
- **label at exactly 100 characters** -- Passes validation. 101 characters shows the length error.
- **label left empty** -- Valid; a block with no label is fully supported and displays with only its time range.
- **A block's range touches a confirmed booking's boundary exactly (block end == booking start, or booking end == block start)** -- Not a conflict, per the overlap boundary rule; the ranges are adjacent, not overlapping.
- **A block's range overlaps a Booking by a single minute** -- Counted as a conflict; any nonzero overlap qualifies, there is no minimum-overlap threshold.
- **Recurrence is toggled on and then off again before saving** -- The day-of-week selection is discarded; the block saves as a plain single-date block with no recurrence.
- **Ownership boundary: a block's parent Pro Account is somehow different from the currently signed-in Pro** -- Not a reachable state in this single-operator product (scope-boundaries.md SC-01): every Time Block belongs to exactly one Pro Account, and the signed-in Pro can only ever act on her own.
- **Talia's own conflict choice arrives after the conflicting booking has already changed state through an unrelated path** -- FEAT-17.SPEC-006's re-validation drops that booking from the set before applying any hand-off, per the conflicting-booking definition no longer matching its current state.

## Acceptance Criteria

**FEAT-17.SPEC-008-AC-01:** Given Talia leaves the start field empty on FEAT-17.SPEC-001, when she blurs the field, then she sees "Choose a start date and time."

**FEAT-17.SPEC-008-AC-02:** Given Talia sets an end time equal to the start time, when validation runs, then she sees "End time must be after the start time" and the save is blocked.

**FEAT-17.SPEC-008-AC-03:** Given Talia sets an end time one minute after the start time, when validation runs, then it passes with no error.

**FEAT-17.SPEC-008-AC-04:** Given Talia turns on the Repeats toggle without selecting a day, when she attempts to submit, then she sees "Choose which day this repeats on."

**FEAT-17.SPEC-008-AC-05:** Given Talia turns on the Repeats toggle, when the day-of-week selector appears, then it defaults to the day-of-week of her entered start date.

**FEAT-17.SPEC-008-AC-06:** Given Talia enters a label of exactly 100 characters, when she blurs the field, then no error is shown.

**FEAT-17.SPEC-008-AC-07:** Given Talia enters a label of 101 characters, when she blurs the field, then she sees "Label must be 100 characters or fewer."

**FEAT-17.SPEC-008-AC-08:** Given Talia leaves the label empty and saves, when the block is created, then it is valid with no label.

**FEAT-17.SPEC-008-AC-09:** Given a block's range ends exactly at a confirmed booking's start time, when FEAT-17.SPEC-004 checks for conflicts, then it is not treated as a conflict.

**FEAT-17.SPEC-008-AC-10:** Given a block's range overlaps a confirmed booking by one minute, when FEAT-17.SPEC-004 checks for conflicts, then it is treated as a conflict.

**FEAT-17.SPEC-008-AC-11:** Given a Booking in the block's range is Pending Payment, when the conflict check runs, then it does not count as a conflict.

**FEAT-17.SPEC-008-AC-12:** Given a Booking in the block's range is Cancelled by Client, when the conflict check runs, then it does not count as a conflict.

**FEAT-17.SPEC-008-AC-13:** Given Talia (the Pro) attempts to create, edit, remove, or resolve a conflict on her own block, when she acts, then every action is allowed with no restriction.

**FEAT-17.SPEC-008-AC-14:** Given Platform Operator Support views the Manage Time Blocks list, when the screen renders, then Support sees every block including labels, with no create, edit, remove, or conflict-resolution control anywhere.

**FEAT-17.SPEC-008-AC-15:** Given the Client (Riley) has no navigation path to any FEAT-17 screen, when this is verified, then no create, view, edit, remove, or conflict-resolution capability is ever exposed to her.

**FEAT-17.SPEC-008-AC-16:** Given a conflicting booking exists, when Talia has not yet made a choice for it on FEAT-17.SPEC-003, then the booking is never automatically cancelled, rescheduled, or altered.

**FEAT-17.SPEC-008-AC-17:** Given Talia chooses "Keep as exception" for a conflicting booking, when FEAT-17.SPEC-006 commits, then no field on that Booking record changes.

**FEAT-17.SPEC-008-AC-18:** Given a single-date block commits with zero conflicts, when it is created, then its state is set to Active by default.

**FEAT-17.SPEC-008-AC-19:** Given a block's end value passes, when FEAT-17.SPEC-007's scheduled expiry check runs, then its state derivation moves to Expired and Talia cannot override this transition.

**FEAT-17.SPEC-008-AC-20:** Given a recurring occurrence is checked for conflicts, when FEAT-17.SPEC-005 applies this spec's rules, then it uses the exact same conflicting-booking definition as a single-date block save (FEAT-17.SPEC-004).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 7 | 7 |
| Edge Cases | 9 | 9 |

