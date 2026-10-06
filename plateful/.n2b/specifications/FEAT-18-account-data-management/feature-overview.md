---
document_type: feature-overview
feature_number: FEAT-18
feature_name: Account & Data Management
feature_slug: account-data-management
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 15
screen_count: 5
automation_count: 4
logic_rule_count: 2
integration_count: 1
notification_count: 3
---

# Feature Breakdown Brief: Account & Data Management

## Summary

**Feature:** Account & Data Management
**ID:** FEAT-18
**Description:** The organiser can review, export, or permanently delete the household's data, and manage the account itself — including the minimal data held for kid profiles.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** The brief's privacy stance is explicit and repeated: "children's data is minimal, parent-controlled and never used for anything but the family's own plan" and "no ads, ever, and no selling of family data" (BRIEF.md, Constraints: Privacy). A stated privacy promise is not credible without a concrete way to exercise it, so this is included at MVP rather than deferred. Research on meal-planning apps that closed or were folded into other products within about two years (app-alternative reviews, MEDIUM confidence) supports a readable export as a credible answer to families' durability worries.

**Key Capabilities:**
- Export household data — Organiser downloads a copy of the household's plans, ratings, and settings
- Delete a member profile — Organiser removes a member (including a kid profile) and their associated data
- Delete the household — Organiser permanently deletes the entire household and all its data
- Manage own account — Any adult member edits their own name and email, changes their sign-in, or deletes their own account
- Contact support — Any adult member sends a short description of a problem to the operator

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-18.SPEC-001 | Export Household Data | Screen | Maya | Organiser requests and downloads a complete, readable copy of the household's data |
| FEAT-18.SPEC-002 | Remove Member Profile | Screen | Maya | Organiser selects a member (adult or kid) and permanently removes them and their associated data |
| FEAT-18.SPEC-003 | Delete Household | Screen | Maya | Organiser sees exactly what will be lost and confirms permanent deletion of the entire household |
| FEAT-18.SPEC-004 | My Account | Screen | Maya, Sam | Any adult member views and edits their own name, email, and sign-in, or deletes their own account |
| FEAT-18.SPEC-005 | Contact Support | Screen | Maya, Sam | Any adult member sends a short description of a problem to the operator |
| FEAT-18.SPEC-006 | Export Generation Processing | Automation | Maya | Compiles all household records into a readable export file and makes it available for download |
| FEAT-18.SPEC-007 | Member Removal Processing | Automation | Maya, Sam, Jordan (young kid profile, no login — MVP), Jordan (older kid, limited login — Later) | On member removal, deletes the member's dietary rules and ratings and stops future plans from accounting for them |
| FEAT-18.SPEC-008 | Household Deletion Processing | Automation | All | On household deletion, permanently removes every member profile, plan, list, rule, and rating within 30 days |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | Automation | Maya, Sam | Permanently deletes an adult's own account, blocked for the organiser unless the role was handed over or the household deleted first |
| FEAT-18.SPEC-010 | Account & Data Validation Rules | Logic/Rule | Maya, Sam | Governs export rate-limiting, irreversible-action confirmation, the 30-day purge window, and the organiser hand-over-or-delete-first precondition |
| FEAT-18.SPEC-011 | Account & Data Authorization Rules | Logic/Rule | All | Governs who can export, remove members, delete the household, manage their own account, or contact support |
| FEAT-18.SPEC-012 | Transactional Email (Account & Data) | Integration | Maya, Sam | Delivers export-ready, deletion-completed, and support-acknowledgement email through the household's transactional email capability |
| FEAT-18.SPEC-013 | Export Ready Notification | Notification | Maya | Tells the organiser once a requested export is ready to download |
| FEAT-18.SPEC-014 | Household Deletion Completed Notification | Notification | Maya | Tells the organiser once household deletion has fully completed |
| FEAT-18.SPEC-015 | Support Request Acknowledgement | Notification | Maya, Sam | Sends the member who contacted support a transactional-email acknowledgement |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Export household data | FEAT-18.SPEC-001, FEAT-18.SPEC-006, FEAT-18.SPEC-010, FEAT-18.SPEC-012, FEAT-18.SPEC-013 | Export screen requests a copy; generation automation compiles it; rate-limit rule governs frequency; email integration and notification confirm readiness | Phase 2 (Explicit) |
| Delete a member profile | FEAT-18.SPEC-002, FEAT-18.SPEC-007, FEAT-18.SPEC-011 | Removal screen selects the member; processing automation cascades the deletion; authorization restricts the action to Maya | Phase 2 (Explicit) |
| Delete the household | FEAT-18.SPEC-003, FEAT-18.SPEC-008, FEAT-18.SPEC-010, FEAT-18.SPEC-011, FEAT-18.SPEC-012, FEAT-18.SPEC-014 | Confirmation screen shows what will be lost; deletion automation cascades within 30 days; validation enforces explicit confirmation; email and notification confirm completion | Phase 2 (Explicit) |
| Manage own account | FEAT-18.SPEC-004, FEAT-18.SPEC-009, FEAT-18.SPEC-010, FEAT-18.SPEC-011 | My Account screen edits name/email/sign-in and starts self-deletion; processing automation enforces the organiser precondition | Phase 2 (Explicit) |
| Contact support | FEAT-18.SPEC-005, FEAT-18.SPEC-011, FEAT-18.SPEC-012, FEAT-18.SPEC-015 | Contact screen submits a description; authorization opens it to any adult; email integration and acknowledgement confirm receipt | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a single Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-18.SPEC-006 | Export Generation Processing | Phase 4 (Trigger-Response — processing logic) | Compiling "a complete, readable copy" of all household records (Data Notes: "the export file itself, compiled from all household records") is non-trivial processing with its own retry-on-failure behavior (States field) — too much logic to stay inline in the Export screen |
| FEAT-18.SPEC-007 | Member Removal Processing | Phase 4 (Trigger-Response, cross-entity effect) | Removal cascades to Dietary Rule and Rating deletion and to future plan accounting (Primary Flows & Alternates, XBR-16) — a cross-entity effect off the Removal screen |
| FEAT-18.SPEC-008 | Household Deletion Processing | Phase 4 (Trigger-Response, cross-feature effect) | Full deletion cascades across every household entity owned by other features (Household, Member Profile, Weekly Plan, Grocery List, Pantry Item, Rating, Dietary Rule, Support Request) within a stated 30-day window (XBR-16) — an atomic, cross-feature effect that cannot live inline in a screen |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | Phase 4 (Trigger-Response, cross-feature precondition) | Deleting one's own account must check the organiser-hand-over precondition owned by FEAT-09 (XBR-15) before proceeding — a cross-feature gate too complex to stay inline |
| FEAT-18.SPEC-010 | Account & Data Validation Rules | Phase 5 (Rule-Constraint Discovery) | Export rate-limiting, irreversible-action confirmation, the 30-day purge/no-other-use rule, and the organiser hand-over-or-delete-first precondition (Validation & Limits field) are conditional rules shared across SPEC-001 through SPEC-004 and SPEC-006 through SPEC-009 — crosses the standalone-spec threshold |
| FEAT-18.SPEC-011 | Account & Data Authorization Rules | Phase 5 (Rule-Constraint Discovery — authorization) | Every screen in this feature behaves differently by role (Maya Full, Sam Own-only, both Jordan rows None, Riley None per the Access Matrix's Account & Data column) — a cross-cutting rule set shared, not duplicated, per screen |
| FEAT-18.SPEC-012 | Transactional Email (Account & Data) | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-32) names transactional email as a capability this feature relies on for export/deletion confirmations and support acknowledgements; the feature-dependency-map's External Touchpoints table marks this feature's share of that capability "pending — awaiting validated Brief for FEAT-18" |
| FEAT-18.SPEC-013 | Export Ready Notification | Phase 4 (Notification surfacing) | The Communications field states "sends a confirmation once an export is ready" — a message with a defined audience and delivery timing, not a same-screen toast, since export generation is asynchronous |
| FEAT-18.SPEC-014 | Household Deletion Completed Notification | Phase 4 (Notification surfacing) | The Communications field states "sends a confirmation once ... a deletion completes"; household deletion is explicitly asynchronous (up to 30 days, Validation & Limits), so completion needs its own delivered message distinct from the screen's immediate confirmation |
| FEAT-18.SPEC-015 | Support Request Acknowledgement | Phase 4 (Notification surfacing) | The Communications field states "a support contact sends the member an acknowledgement by transactional email" — an explicit channel and audience |

## Entity-Lifecycle Coverage Matrix

**Entity: Household**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A — owned by FEAT-01 | Household creation is FEAT-01's Household Setup responsibility | -- |
| Read (single) | FEAT-18.SPEC-003 | Delete Household screen reads a summary of the household's members, plans, and history so the organiser sees exactly what will be lost | -- |
| Read (list) | N/A | A household has no list form; one household per account (SC-03) | -- |
| Update | N/A — owned by FEAT-01, FEAT-07, FEAT-14, FEAT-16 | This feature never edits household settings, only deletes the record | -- |
| Delete/Archive | FEAT-18.SPEC-003, FEAT-18.SPEC-008 | Hard delete after explicit confirmation of the irreversible action (FEAT-18.SPEC-010); no restore path; cascades to every member profile, plan, list, pantry item, dietary rule, rating, and support request; fully removed within 30 days and not retained in any form usable for other purposes (Validation & Limits, ASMP-27) | Household deletion supersedes any in-flight edit (feature-dependency-map.md, Household Contention) |
| State Transition | FEAT-18.SPEC-008 | Active → Closed/Deleted, set when deletion processing completes | -- |

**Entity: Member Profile** (this feature's operations only — full lifecycle owned jointly with FEAT-01/FEAT-09)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A — owned by FEAT-01, FEAT-09 | Organiser, adult, and kid-profile creation, plus invitation acceptance, are FEAT-01's and FEAT-09's responsibility | -- |
| Read (single) | FEAT-18.SPEC-004 | My Account screen loads the signed-in adult's own profile | -- |
| Read (list) | FEAT-18.SPEC-002 | Remove Member Profile screen lists every removable member (excludes the organiser, per XBR-15) | -- |
| Update | FEAT-18.SPEC-004 | Any adult edits their own display name, email, or sign-in credentials | Own-only per the Access Matrix; Sam cannot edit anyone else's profile through this feature |
| Delete/Archive | FEAT-18.SPEC-002, FEAT-18.SPEC-004, FEAT-18.SPEC-007, FEAT-18.SPEC-009 | Hard delete: organiser-initiated removal (any member, including a kid profile) or self-initiated own-account deletion (adults only); no restore path; cascades to the member's Dietary Rules and Ratings and removes them from future plan accounting (XBR-16); removed within 30 days, not retained for other use | Distinct from FEAT-09's self-leave path, which soft-removes (status Left) and anonymises rather than deletes ratings — see Shared Context for the flagged product ambiguity between the two |
| State Transition | FEAT-18.SPEC-007, FEAT-18.SPEC-009 | Active → Removed (organiser removal); Active → Deleted (self-account deletion) | -- |

**Entity: Support Request**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-18.SPEC-005 | Any adult submits a short description of a problem (general support contact kind) | Distinct from the safety-concern kind created by FEAT-02 |
| Read (single) | N/A — owned by FEAT-01, FEAT-22 | The organiser sees open requests and access records through FEAT-01; Riley views and manages status through FEAT-22 | -- |
| Read (list) | N/A — owned by FEAT-01 | -- | -- |
| Update | N/A — owned by FEAT-22 | Status and access-record changes belong to Operator Read-Only Support Access | -- |
| Delete/Archive | N/A | Support requests are never deleted by this feature except as part of the full household-deletion cascade (FEAT-18.SPEC-008); no independent purge policy exists for an active household — an explicit non-goal, retained as household history | -- |

**Referenced Entities (read-only, or cascade-deleted only):**

| Entity | Read By / Deleted By | Context |
|--------|---------|---------|
| Dietary Rule | FEAT-18.SPEC-007, FEAT-18.SPEC-008 | Cascade-deleted on member removal and on household deletion; full CRUD lifecycle (creation, editing) owned by FEAT-01 |
| Weekly Plan | FEAT-18.SPEC-008 | Cascade-deleted on household deletion only; full lifecycle owned by FEAT-03/FEAT-23 |
| Grocery List | FEAT-18.SPEC-008 | Cascade-deleted on household deletion only; full lifecycle owned by FEAT-06 |
| Pantry Item | FEAT-18.SPEC-008 | Cascade-deleted on household deletion only; full lifecycle owned by FEAT-05 |
| Rating | FEAT-18.SPEC-007, FEAT-18.SPEC-008 | Cascade-deleted on member removal and on household deletion; full lifecycle owned by FEAT-12 |
| Planned Meal | FEAT-18.SPEC-008 | Cascade-deleted on household deletion only, as part of the Weekly Plan it belongs to; full lifecycle owned by FEAT-03/FEAT-23 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Organiser requests a household data export | Rate-limit check runs before a new export is generated | Standalone Logic/Rule | FEAT-18.SPEC-010 |
| Export request accepted | Export begins compiling with visible progress rather than an indefinite wait (States field) | Standalone Automation | FEAT-18.SPEC-006 |
| Export generation fails | Automatically retried; if retries exhaust, clearly reported rather than left ambiguous (States field) | Standalone Automation | FEAT-18.SPEC-006 |
| Export becomes ready | Organiser is sent a confirmation via the shared email capability | Standalone Notification, Standalone Integration | FEAT-18.SPEC-013, FEAT-18.SPEC-012 |
| Organiser selects a member to remove | Irreversible-action confirmation shown, explicit affirmative required | Standalone Logic/Rule | FEAT-18.SPEC-010 |
| Organiser confirms member removal | Dietary Rules and Ratings for that member are deleted; future plan generation stops accounting for them | Standalone Automation | FEAT-18.SPEC-007 |
| Organiser confirms household deletion | Every member profile, plan, list, pantry item, dietary rule, rating, and support request is permanently removed within 30 days | Standalone Automation | FEAT-18.SPEC-008 |
| Household deletion completes | Organiser is sent a completion confirmation via the shared email capability | Standalone Notification, Standalone Integration | FEAT-18.SPEC-014, FEAT-18.SPEC-012 |
| Adult member edits their own name, email, or sign-in | Field-level validation runs; success shown as a same-screen confirmation | Inline in triggering screen | FEAT-18.SPEC-004 |
| Adult member requests their own account be deleted | Organiser precondition checked (must have handed over the role or deleted the household first, per XBR-15) | Standalone Logic/Rule | FEAT-18.SPEC-010 |
| Own-account deletion precondition satisfied and confirmed | Account is permanently deleted | Standalone Automation | FEAT-18.SPEC-009 |
| Own-account deletion precondition fails (organiser, no hand-over) | Action is blocked; member is directed to hand over the role (FEAT-09) or delete the household first | Standalone Logic/Rule | FEAT-18.SPEC-010, FEAT-18.SPEC-011 |
| Adult member submits a support description | Support Request of kind "general support contact" is created | Inline in triggering screen | FEAT-18.SPEC-005 |
| Support Request created | Reporting member receives an acknowledgement via the shared email capability | Standalone Notification, Standalone Integration | FEAT-18.SPEC-015, FEAT-18.SPEC-012 |
| Any export, deletion, or account-management action is attempted offline | Request queues locally and completes once connectivity returns (States field, Offline-degraded) | Standalone Logic/Rule | FEAT-18.SPEC-010 |
| A role or user without Account & Data access attempts any action in this feature | Access denied; kid rows and Riley never see this area, an unauthorized visitor sees only sign-in | Standalone Logic/Rule | FEAT-18.SPEC-011 |

## Shared Context

**Shared Entities:**
- Household — read by FEAT-18.SPEC-003 for the deletion-preview summary; deleted by FEAT-18.SPEC-008. This feature never edits household settings.
- Member Profile — read (single) by FEAT-18.SPEC-004, read (list) by FEAT-18.SPEC-002, updated by FEAT-18.SPEC-004, deleted by FEAT-18.SPEC-002/SPEC-007 (organiser removal) and FEAT-18.SPEC-004/SPEC-009 (self-deletion). Fields touched: display_name, sign_in (email/credentials) for own-account edits; status for removal/deletion.
- Dietary Rule, Rating — never created or edited by this feature, only cascade-deleted by FEAT-18.SPEC-007 and FEAT-18.SPEC-008.
- Support Request — created by FEAT-18.SPEC-005 (general support contact kind only); read, status-managed, and access-recorded entirely by FEAT-01/FEAT-22.

**Shared UI Patterns:**
- Irreversible-action confirmation — Remove Member Profile (FEAT-18.SPEC-002), Delete Household (FEAT-18.SPEC-003), and own-account deletion within My Account (FEAT-18.SPEC-004) share the same modal pattern: state plainly and specifically what will be lost, require an explicit affirmative tap, no default-confirmed state, consistent with the Validation & Limits field's "explicit confirmation of an irreversible action." Spec Writers for all three should describe the pattern identically.
- Progress-not-silence for long-running requests — export generation (FEAT-18.SPEC-001/006) and household deletion (FEAT-18.SPEC-003/008) both show clear progress rather than an indefinite wait, and a failure is either retried automatically or clearly reported, never left in an ambiguous half-completed state (States field).
- Offline queuing — any export, removal, deletion, or account-edit request made without connectivity queues locally and completes once reconnected (States field, Offline-degraded); this pattern is governed once by FEAT-18.SPEC-010 and referenced by every screen rather than re-described.

**Shared Validation:**
- FEAT-18.SPEC-010 defines export rate-limiting, irreversible-action confirmation, the 30-day purge/no-other-use rule, the organiser hand-over-or-delete-first precondition, and offline queuing. FEAT-18.SPEC-001 through FEAT-18.SPEC-004 and FEAT-18.SPEC-006 through FEAT-18.SPEC-009 all reference SPEC-010 rather than duplicating these rules.
- FEAT-18.SPEC-011 defines who may export, remove members, delete the household, manage their own account, or contact support, including the unauthorized-visitor and no-access-role experiences. Every screen in this feature (FEAT-18.SPEC-001 through 005) references SPEC-011 for its role-gated behavior.

**Flagged product ambiguity (not resolved by this Brief):** Stage 2's Key Capabilities list both "delete a member profile" (organiser-initiated, this feature) and, via FEAT-09, "leave the household" (self-initiated departure). Both features also let an adult remove themselves: FEAT-09.SPEC-005 (Leave Household) soft-removes the member and anonymises their ratings, while this feature's own-account deletion (FEAT-18.SPEC-009) is modeled as a hard delete consistent with XBR-16's "removing a member deletes their dietary rules and ratings." Neither XBR-16, the Access Matrix, nor either feature's Functional Depth fields state whether "delete my own account" (FEAT-18) and "leave the household" (FEAT-09) are the same user-facing action under two names or two genuinely distinct actions with different data outcomes for the same self-initiated departure. This Brief treats them as distinct per each feature's own Stage 2 wording (delete vs. leave) but flags the overlap for resolution at Stage 4 or a future synthesis pass, consistent with FEAT-09's own flagged note on the same seam.

## Internal Dependency Map

```
SPEC-001 (Export Household Data) -> [organiser requests export] -> SPEC-010 (Validation Rules) -> [under rate limit] -> SPEC-006 (Export Generation Processing)
SPEC-006 (Export Generation Processing) -> [export ready] -> SPEC-013 (Export Ready Notification) -> [via] SPEC-012 (Transactional Email) -> [delivered to] Maya
SPEC-002 (Remove Member Profile) -> [organiser selects member] -> SPEC-010 (Validation Rules) -> [confirmed] -> SPEC-007 (Member Removal Processing) -> [Dietary Rules + Ratings deleted]
SPEC-003 (Delete Household) -> [organiser confirms] -> SPEC-010 (Validation Rules) -> [confirmed] -> SPEC-008 (Household Deletion Processing) -> [cascade complete, within 30 days] -> SPEC-014 (Household Deletion Completed Notification) -> [via] SPEC-012 (Transactional Email) -> [delivered to] Maya
SPEC-004 (My Account) -> [adult edits own details] -> [saved inline]
SPEC-004 (My Account) -> [adult requests own-account deletion] -> SPEC-010 (Validation Rules) -> [organiser precondition checked] -> SPEC-009 (Own Account Deletion Processing) [or blocked, directed to FEAT-09 hand-over / SPEC-003]
SPEC-005 (Contact Support) -> [member submits description] -> [Support Request created] -> SPEC-015 (Support Request Acknowledgement) -> [via] SPEC-012 (Transactional Email) -> [delivered to] reporting member
SPEC-005 (Contact Support) -> [request created] -> FEAT-22 (Operator Read-Only Support Access, support request raised)
SPEC-001 through SPEC-005 -> [gated by] -> SPEC-011 (Authorization Rules)
```

**Default Entry:** FEAT-18.SPEC-004 (My Account) for any adult member navigating to this feature area; the organiser reaches FEAT-18.SPEC-001, FEAT-18.SPEC-002, and FEAT-18.SPEC-003 from account settings.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-18.SPEC-001 | Inbound | FEAT-14 (Subscription & Billing Management) | Organiser navigates from account settings to request an export | Request an export (Upgrading & Managing the Account journey, step 4) |
| FEAT-18.SPEC-005 | Outbound | FEAT-22 (Operator Read-Only Support Access) | A submitted support description becomes an open Support Request that opens Riley's read-only support access | Any adult submits a support description |
| FEAT-18.SPEC-007 | Outbound | FEAT-01 (Household Setup & Member Profiles) | A removed member's profile and dietary rules disappear from FEAT-01's member and rule lists | Organiser confirms member removal |
| FEAT-18.SPEC-007 | Outbound | FEAT-12 (Meal Rating & Preference Learning) | A removed member's ratings are deleted rather than retained as learning influence, unlike a self-leaving member (FEAT-09) | Organiser confirms member removal |
| FEAT-18.SPEC-008 | Outbound | FEAT-01, FEAT-03/FEAT-23, FEAT-06, FEAT-05, FEAT-12 | Household deletion cascades to every entity these features own (member profiles, plans, lists, pantry items, ratings) | Organiser confirms household deletion |
| FEAT-18.SPEC-009 / SPEC-010 | Inbound | FEAT-09 (Household Invitations & Membership) | An organiser attempting to delete their own account is redirected to FEAT-09's role hand-over flow, or to FEAT-18.SPEC-003, per XBR-15 | Organiser (self) attempts account deletion while still organiser |
| FEAT-18.SPEC-011 | Outbound | FEAT-22 (Operator Read-Only Support Access) | Riley's read-only support access never extends to Account & Data (Access Matrix: Riley None); XBR-14 confirms support access is otherwise read-only and logged | N/A — a standing constraint, not a single trigger |
| FEAT-18.SPEC-012 | Outbound | FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012 | This feature's Integration spec fills the "pending" slot for FEAT-18 in the shared transactional-email External Touchpoint (feature-dependency-map.md), alongside these other features' pieces of the same capability | N/A — a standing capability contract, not a single trigger |

## Non-Functional Notes

**Data volumes / growth:** An export compiles every household record accumulated over the life of the account — plans, lists, ratings, and settings kept indefinitely per SC-18/ASMP-24 — so export generation must stay reasonably fast even for a household with years of history; household deletion must reliably cascade across the same accumulated volume within the stated 30-day window.

**Responsiveness:** Export and deletion requests show clear progress rather than an indefinite wait, and a failure is retried automatically or clearly reported rather than left ambiguous (States field); simple own-account edits and the support-contact submission complete without a perceptible wait, consistent with the product's general responsiveness bar (assumptions-constraints.md ASMP-23).

**Data sensitivity / privacy:** An export contains the household's most sensitive data in one place, including kid profiles' minimal data (first name/nickname, age band, dietary rules) and any allergy information, so it carries children's-privacy-class protection throughout generation and delivery (ASMP-26, ASMP-27); deletion removes personal and children's data alike, and no deleted or exported data is retained in any form usable for advertising or resale, consistent with the brief's "no ads, ever, and no selling of family data" (BRIEF.md, Constraints: Privacy).

**Compliance flags:** This feature is the household's primary mechanism for the data-subject rights named in ASMP-27 — a copy of one's data (export) and deletion — for both general personal data and children's-privacy-class data; verifiable parental consent for kid-profile data is established at creation by FEAT-01 and is not re-verified here, since removal and deletion of a kid profile are organiser-only actions requiring no separate consent step.

## Non-Goals

- **Exporting or deleting data across multiple households from one account** — Excluded per scope-boundaries.md SC-03: there is one household per account in v1, so export and deletion are always scoped to the single household the account holds; no cross-household selection exists.
- **Kid-profile self-service export, deletion, or account management** — Excluded per the Access Matrix (Account & Data: None for both Jordan rows) and scope-boundaries.md SC-02: young kid profiles have no login in v1 and an older-kid limited login is Later-phase; kid-profile data is managed exclusively through Maya's member-removal path (FEAT-18.SPEC-002) or household deletion, never by the kid.
- **Riley (Operator) viewing, exporting, or triggering deletion of household data** — Excluded per the Access Matrix (Account & Data: None for Riley) and XBR-14: operator support access is strictly read-only and scoped to an open Support Request, never extending to export or deletion.
- **Immediate, instantaneous full data purge** — Intentional lifecycle decision from the CRUD matrix: the Validation & Limits field sets a 30-day removal window rather than an instant purge, so a short retention-for-processing period is an explicit design decision, not an accidental gap; no form of the deleted data is usable for any other purpose during that window (ASMP-27).
- **Retaining or repurposing exported or deleted data for analytics, marketing, or resale** — Excluded per BRIEF.md's Constraints: Privacy ("no ads, ever, and no selling of family data") and ASMP-26: an export is a copy handed to the household, and deleted data is not kept in any form usable for other purposes.
- **In-product back-and-forth with support beyond the initial description** — Excluded per scope-boundaries.md SC-14: this feature's Contact Support screen sends a one-way description and receives a one-time email acknowledgement; any further exchange happens outside the product (email), not as in-product messaging.
- **Restoring a deleted member profile or a deleted household** — Intentional lifecycle decision from the CRUD matrix: FEAT-18.SPEC-002/007 (member removal) and FEAT-18.SPEC-003/008 (household deletion) are both hard deletes with no restore path, consistent with the Primary Flows & Alternates field's "permanently deletes" and "permanently removed" language; this contrasts deliberately with FEAT-09's soft, restorable-in-spirit self-leave path (see Shared Context's flagged ambiguity).
