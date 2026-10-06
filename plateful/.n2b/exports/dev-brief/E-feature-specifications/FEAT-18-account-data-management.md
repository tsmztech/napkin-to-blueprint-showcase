# FEAT-18 — Account & Data Management

This chapter covers FEAT-18, Account & Data Management, a Important-tier feature. It contains 15 specifications carrying 175 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-18.SPEC-001 | Export Household Data | screen | 13 |
| FEAT-18.SPEC-002 | Remove Member Profile | screen | 13 |
| FEAT-18.SPEC-003 | Delete Household | screen | 14 |
| FEAT-18.SPEC-004 | My Account | screen | 16 |
| FEAT-18.SPEC-005 | Contact Support | screen | 9 |
| FEAT-18.SPEC-006 | Export Generation Processing | automation | 9 |
| FEAT-18.SPEC-007 | Member Removal Processing | automation | 9 |
| FEAT-18.SPEC-008 | Household Deletion Processing | automation | 12 |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | automation | 9 |
| FEAT-18.SPEC-010 | Account & Data Validation Rules | logic-rule | 16 |
| FEAT-18.SPEC-011 | Account & Data Authorization Rules | logic-rule | 15 |
| FEAT-18.SPEC-012 | Transactional Email (Account & Data) | integration | 12 |
| FEAT-18.SPEC-013 | Export Ready Notification | notification | 10 |
| FEAT-18.SPEC-014 | Household Deletion Completed Notification | notification | 9 |
| FEAT-18.SPEC-015 | Support Request Acknowledgement | notification | 9 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Export Household Data

## Overview

**Name:** Export Household Data
**ID:** FEAT-18.SPEC-001
**Type:** Screen
**Purpose:** Maya requests a complete, readable copy of the household's data and downloads it once it is ready.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Requesting an export of the household's plans, ratings, lists, dietary rules, and settings
- Showing the export rate-limit state and the last export's status and download link, when one exists
- Showing progress while an export compiles and surfacing a completed download or a reported failure

**Non-Goals:**
- Compiling the export file itself -- owned by FEAT-18.SPEC-006 (Export Generation Processing), which this screen triggers and whose progress and outcomes it displays
- Choosing what the export contains -- the export always covers the complete household record set; scope-boundaries.md establishes no partial-export capability, so no selection controls exist on this screen
- Sending the export-ready confirmation -- owned by FEAT-18.SPEC-013 (Export Ready Notification) and delivered by FEAT-18.SPEC-012 (Transactional Email); this screen only reflects the same ready state in-app
- Deleting or removing household data -- owned by FEAT-18.SPEC-002 (Remove Member Profile) and FEAT-18.SPEC-003 (Delete Household)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-004 (My Account) | Maya taps "Export household data" in the organiser-only Household Data & Deletion section | None -- screen loads the household's current export state |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Request a data export" from the account settings area (feature-dependency-map.md, Navigation Connections) | None -- screen loads the household's current export state |
| FEAT-18.SPEC-013 (Export Ready Notification) | Maya taps "Download export" on the export-ready notification | None -- screen loads the household's current (Ready) export state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Request an export and download a ready export | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable from any navigation available to Sam; a direct attempt shows "Only the household organiser can export household data." and returns him to FEAT-18.SPEC-004 |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable from any navigation available to this login; a direct attempt shows "Only the household organiser can export household data." and returns to the older-kid login's landing area |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, a non-organiser lands on FEAT-18.SPEC-004, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress export request exists to preserve, since a request is submitted and confirmed in a single action |

## Layout and Content

**Header:** Screen title "Export Household Data" with a back arrow (returns to the entry source).

**Body:** A single content column.
- An explanatory line: "Download a complete copy of your household's plans, lists, ratings, and settings."
- A status card reflecting the current export state (see States): a "Request Export" button when no export is in progress and the household is under the rate limit; a progress indicator with the label "Compiling your export..." while one is generating; a "Download Export" button with the file's ready date when the latest export is ready; a rate-limit notice when the household is currently blocked from requesting a new export.
- A "Previous Exports" list below the status card, showing up to the household's most recent completed exports (ready date and a Download link for each), or the line "No exports yet" when none exist.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; the status card and Previous Exports list stack vertically.
- **Medium size class and above:** Content column capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition back |
| Request Export button | Tap | Validates the export rate limit via FEAT-18.SPEC-010, then triggers FEAT-18.SPEC-006 (Export Generation Processing) | Status card switches to the compiling/progress state | Progress indicator with the label "Compiling your export..." |
| Request Export button (rate-limited) | Tap | No action -- button is disabled while the household is under the limit | None | Rate-limit notice text remains visible |
| Download Export button | Tap | Retrieves the completed export file for download | None -- the screen does not change state | The device's standard file-download experience begins |
| Previous Exports "Download" link | Tap | Retrieves the selected prior export file for download | None | The device's standard file-download experience begins |

### Accessibility Notes

- **Focus order:** Back arrow -> explanatory line -> status card's primary action (Request Export or Download Export) -> Previous Exports list entries in ready-date order (newest first).
- **Progress announcements:** When the status card enters the compiling state, "Compiling your export" is announced to assistive technology; when it becomes ready, "Your export is ready to download" is announced.
- **Keyboard alternatives:** Every action on this screen (request, download) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Status card and Previous Exports area show loading placeholders | Screen first opens, before the initial fetch of the household's export state and previous exports completes | Fetch succeeds (-> No export yet, Ready to request, Compiling, Ready, or Rate-limited, whichever matches) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load your export history. Try again." with a Retry button; Previous Exports list and the status card's action are hidden until data loads | The initial fetch of export state and previous exports fails | Maya taps Retry (re-fetches) or the back arrow (returns to the entry source) |
| No export yet | Status card shows "Request Export" button; Previous Exports shows "No exports yet" | Initial fetch succeeds and the household has never requested an export | Maya requests an export |
| Ready to request | Status card shows "Request Export" button, enabled | No export currently compiling and household is under the export rate limit (FEAT-18.SPEC-010) | Maya taps Request Export |
| Compiling | Status card shows a progress indicator labeled "Compiling your export..." | Export request accepted by FEAT-18.SPEC-006 | Export completes (ready) or fails (retries exhausted) |
| Ready | Status card shows "Download Export" with the ready date | FEAT-18.SPEC-006 completes successfully | Maya downloads the file (screen state persists as Ready -- downloading does not consume the export) |
| Rate-limited | Status card shows a disabled Request Export control with the notice: "You've reached this period's export limit. You can request another export once the limit resets." | The household's export count for the current rate-limit window is at or above the limit (FEAT-18.SPEC-010) | The rate-limit window resets |
| Error | Status card shows an error banner: "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support." with a link to FEAT-18.SPEC-005 (Contact Support) | FEAT-18.SPEC-006 reports retries exhausted | Maya requests a new export (once the rate limit allows) |
| Offline/Degraded | Banner "You're offline -- your export request will be sent when you reconnect." at top; Request Export remains tappable and queues the request locally | Connectivity lost while this screen is open | Connectivity restored -- the queued request submits automatically and the screen shows the Compiling state |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules). See that spec for the export rate-limit condition and its exact denied behavior.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | Entry source (FEAT-18.SPEC-004 or FEAT-14.SPEC-003) | FEAT-14 when entered from there |
| "contact support" link (Error state) | FEAT-18.SPEC-005 (Contact Support) | -- |

## Data Model

**Creates:** None directly -- Request Export triggers FEAT-18.SPEC-006, which creates the export file record.
**Reads:** Household -- all fields, for the export summary; the household's prior export records (ready date, download reference) for the Previous Exports list.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Export requests are rate-limited per FEAT-18.SPEC-010 -- the Request Export control is disabled and the rate-limit notice shown whenever the household is at its limit.
- An export always covers the complete household record set at the moment of the request; it is never partial or filtered (product-features.md, Data Notes).
- Only Maya (Organiser) can reach this screen, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).

## Edge Cases

- **Maya taps Request Export twice in quick succession** -- The second tap is ignored while the first request is in flight (button enters a disabled, progress-indicating state on the first tap).
- **Maya navigates away while an export is compiling and returns later** -- The screen re-fetches the export's current status and shows whichever state (Compiling, Ready, or Error) matches that status; the request is not resubmitted.
- **The export becomes ready while Maya is offline** -- The Ready state and download become available once connectivity returns and the screen refreshes; FEAT-18.SPEC-013 delivers the confirmation independently of this screen being open.
- **Maya requests a download of a previous export whose file has since expired from storage** -- The download attempt shows "This export is no longer available. Request a new one." and the Previous Exports entry is marked unavailable rather than removed, preserving the record of when it was generated.
- **Household data changes while an export is compiling** -- The in-progress export reflects the household's data as of the moment the request was accepted; changes made after that moment appear only in a subsequent export. No concurrent-edit conflict entry applies here, since this screen only reads the household summary and never writes to a shared entity that another member could contend for.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Export rate-limit rule governs the Request Export control |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can reach this screen |
| FEAT-18.SPEC-006 (Export Generation Processing) | Triggers (outbound) | Request Export starts export compilation |
| FEAT-18.SPEC-013 (Export Ready Notification) | Affects (outbound) | The notification's CTA deep-links back to this screen |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | Organiser-only entry point into this screen |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (inbound) | Alternate entry point from account settings |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| data_export_requested | rate_limit_state (under_limit) | Maya taps Request Export and the request is accepted | N/A -- no Stage 2 success metric measures export usage; retained so the export path's actual use is observable, per feature-overview.md's Rationale on export as a trust mechanism |
| data_export_rate_limited | current_window_count | Maya attempts to request an export while at the limit | N/A -- no Stage 2 success metric measures rate-limit friction; retained to observe whether the limit is ever a real obstacle for households |
| data_export_downloaded | export_age_days | Maya taps a Download control (current or previous export) | N/A -- no Stage 2 success metric measures export completion; retained to distinguish a requested export from one actually retrieved |

## Acceptance Criteria

**FEAT-18.SPEC-001-AC-01:** Given Maya is on the Export Household Data screen with no export in progress and under the rate limit, when she taps Request Export, then the status card shows "Compiling your export..." and FEAT-18.SPEC-006 begins.

**FEAT-18.SPEC-001-AC-02:** Given Maya's export completes successfully, when she returns to this screen, then the status card shows "Download Export" with the ready date, and tapping it downloads the file.

**FEAT-18.SPEC-001-AC-03:** Given the household is at its export rate limit, when Maya opens this screen, then the Request Export control is disabled and shows "You've reached this period's export limit. You can request another export once the limit resets."

**FEAT-18.SPEC-001-AC-04:** Given Maya's export generation exhausts its retries, when she views this screen, then she sees "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support." with a link to FEAT-18.SPEC-005.

**FEAT-18.SPEC-001-AC-05:** Given Maya loses connectivity and taps Request Export, when she is offline, then the banner "You're offline -- your export request will be sent when you reconnect." appears and the request queues locally.

**FEAT-18.SPEC-001-AC-06:** Given connectivity returns while a request is queued, when the queued request submits automatically, then the status card shows the Compiling state without Maya re-tapping Request Export.

**FEAT-18.SPEC-001-AC-07:** Given Sam attempts to reach this screen directly, when the screen loads, then he sees "Only the household organiser can export household data." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-001-AC-08:** Given an unauthenticated visitor attempts to reach this screen, when the screen loads, then they are redirected to the sign-in screen.

**FEAT-18.SPEC-001-AC-09:** Given Maya's session expires while she is on this screen, when she next interacts with it, then a dialog reads "Your session has expired. Sign in to continue." and no export request was made.

**FEAT-18.SPEC-001-AC-10:** Given Maya's household has three previous completed exports, when she opens this screen, then the Previous Exports list shows all three with their ready dates and working Download links.

**FEAT-18.SPEC-001-AC-11:** Given a previous export's file has expired from storage, when Maya taps its Download link, then she sees "This export is no longer available. Request a new one." and the entry stays listed as unavailable.

**FEAT-18.SPEC-001-AC-12:** Given Maya opens this screen, when the initial fetch of her export state and previous exports is in progress, then the status card and Previous Exports area show loading placeholders instead of any action button.

**FEAT-18.SPEC-001-AC-13:** Given the initial fetch of export state and previous exports fails, when Maya views this screen, then she sees "We couldn't load your export history. Try again." with a Retry button, and tapping Retry either loads the screen normally or shows the same error again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 5 | 5 |
| States | 9 (loading, load error, no export yet, ready to request, compiling, ready, rate-limited, error, offline) | 9 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Remove Member Profile

## Overview

**Name:** Remove Member Profile
**ID:** FEAT-18.SPEC-002
**Type:** Screen
**Purpose:** Maya selects a household member (adult or kid) and permanently removes them and their associated data.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Listing every removable member (every member except the organiser herself)
- Showing what will be lost for the selected member before removal
- The irreversible-action confirmation for removal
- Triggering the removal and reflecting its outcome

**Non-Goals:**
- Performing the cascade deletion of the member's Dietary Rules and Ratings -- owned by FEAT-18.SPEC-007 (Member Removal Processing), which this screen triggers
- Editing a member's profile details -- owned by FEAT-01 (Household Setup & Member Profiles); this screen only removes
- An adult member leaving the household on their own -- a distinct, self-initiated action owned by FEAT-09.SPEC-005 (Leave Household); this screen is organiser-initiated removal of another member (feature-overview.md, Shared Context's flagged ambiguity)
- Removing the organiser herself -- the organiser is never a selectable target on this screen; she must hand over the role (FEAT-09) or delete the household (FEAT-18.SPEC-003) first, per XBR-15

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-004 (My Account) | Maya taps "Remove a member" in the organiser-only Household Data & Deletion section | None -- screen loads the current removable-member list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Select and remove any non-organiser member | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable from any navigation available to Sam; a direct attempt shows "Only the household organiser can remove a member." and returns him to FEAT-18.SPEC-004 |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable through this login; a direct attempt shows "Only the household organiser can remove a member." |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the in-progress selection (before confirmation) is discarded |

## Layout and Content

**Header:** Screen title "Remove a Member" with a back arrow (returns to FEAT-18.SPEC-004).

**Body:** A single-column list, one row per removable member (every household member except Maya): the member's display name, member_type (Other Adult Member, or a kid label with age_band), and a "Remove" action per row. Selecting a row's Remove action opens the confirmation step below the list.

**Confirmation step (appears in place of the list when a member is selected):** A summary naming the selected member and stating plainly what will be lost: their dietary rules, their ratings, and their removal from all future plans -- consistent with the Shared Context's irreversible-action confirmation pattern (also used by FEAT-18.SPEC-003 and within FEAT-18.SPEC-004). Two actions: "Remove {member_name}" (destructive, requires the explicit affirmative tap) and "Cancel" (returns to the list). No default-confirmed state -- neither action is preselected or triggered by any other interaction.

### Responsive Behavior

- **Compact breakpoint:** Single-column list and confirmation step as described, full width.
- **Medium size class and above:** Content capped at a consistent platform-wide list width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-004 | Screen closes | Standard transition back |
| Member row "Remove" action | Tap | Opens the confirmation step for that member | List is replaced by the confirmation step | Confirmation step appears with the member's name and loss summary |
| Confirmation "Remove {member_name}" | Tap | Triggers FEAT-18.SPEC-007 (Member Removal Processing) for the selected member | Confirmation step shows a progress state | Progress indicator; on completion, success toast "{member_name} has been removed." and return to the member list |
| Confirmation "Cancel" | Tap | Discards the selection | Confirmation step is replaced by the member list | List reappears unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> member list rows in display order -> (on confirmation) loss summary -> Remove button -> Cancel button.
- **Confirmation announcement:** Entering the confirmation step announces the member's name and the loss summary to assistive technology, so the destructive action's consequences are read before the Remove control is reachable.
- **Success/failure announcement:** The "has been removed" toast and any error banner are announced on completion.
- **Keyboard alternatives:** Every action (select, confirm, cancel) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | List area shows loading placeholders in place of member rows | Screen first opens, before the initial fetch of the removable-member list completes | Fetch succeeds (-> Member list or Empty, whichever matches) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load the household's members. Try again." with a Retry button; no member rows or Remove actions are shown until data loads | The initial fetch of the removable-member list fails | Maya taps Retry (re-fetches) or the back arrow (returns to FEAT-18.SPEC-004) |
| Member list (default) | List of removable members | Initial fetch succeeds, or Cancel is tapped | Maya taps a member's Remove action |
| Confirmation | Loss summary and Remove/Cancel buttons for the selected member | Maya taps Remove on a member row | Maya taps Remove (confirmed) or Cancel |
| Removing | Confirmation step shows a progress indicator, both buttons disabled | Maya taps the confirmation Remove button | Removal completes or fails |
| Error | Error banner "We couldn't remove {member_name}. Try again." with a Retry button; confirmation step remains | FEAT-18.SPEC-007 reports failure | Maya taps Retry (re-attempts) or Cancel (returns to list, no removal occurred) |
| Empty (no removable members) | List area shows "There's no one else to remove yet -- invite a member from Household Settings to add one." | The household has only the organiser as a member | A member is added elsewhere (FEAT-01 or FEAT-09) |
| Offline/Degraded | Banner "You're offline -- this removal will be sent when you reconnect." at top of the confirmation step; Remove queues the request locally | Connectivity lost while the confirmation step is open | Connectivity restored -- the queued removal submits automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules), which defines the irreversible-action confirmation requirement enforced by the confirmation step above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | FEAT-18.SPEC-004 (My Account) | -- |
| Successful removal | FEAT-18.SPEC-002 (this screen, member list state) | -- |
| "invite a member" link (Empty state) | FEAT-09.SPEC-001 (Household Invitations Manager) | FEAT-09 |

## Data Model

**Creates:** None.
**Reads:** Member Profile -- display_name, member_type, age_band (kid rows), status, for every member except the organiser.
**Updates:** None directly -- removal is performed by FEAT-18.SPEC-007.
**Deletes:** None directly -- deletion of Member Profile, Dietary Rule, and Rating records is performed by FEAT-18.SPEC-007.

## Business Rules

- The organiser is never a selectable target on this screen (XBR-15) -- she does not appear in the removable-member list.
- Removal requires the explicit affirmative confirmation defined by FEAT-18.SPEC-010; there is no one-tap removal from the list row itself.
- Only Maya (Organiser) can reach this screen and remove members, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).
- XBR-16: removing a member deletes their dietary rules and ratings and future plans stop accounting for them, distinct from a member's own self-leave path (FEAT-09.SPEC-005), which anonymises rather than deletes ratings.

## Edge Cases

- **Maya taps Remove on a member while another device session (Maya on a second device) is also viewing this screen** -- Both sessions read the same member list; whichever session's removal completes first wins, and the other session's stale confirmation step (if still open for the same member) is rejected with refresh: "This member was already removed." and returns to the now-updated list. This is the concurrent-edit conflict entry for this screen, consistent with the dependency map's Contention note for Member Profile.
- **Sam edits his own account (FEAT-18.SPEC-004) at the same moment Maya removes him here** -- Per the dependency map's Contention note for Member Profile, removal wins and Sam's concurrent edit is refused with a clear message on his own screen.
- **Maya taps the confirmation Remove button twice rapidly** -- The second tap is ignored while the first removal is in progress (buttons disabled during the Removing state).
- **The selected member is a young kid profile with no login** -- The loss summary and removal proceed identically to an adult member's; no login-specific behavior differs, since the kid profile itself (not a session) is what is removed.
- **Maya navigates away mid-confirmation without tapping Remove or Cancel** -- No removal occurs; returning to this screen later shows the member list with the previously-selected member still present.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Irreversible-action confirmation pattern |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can reach this screen |
| FEAT-18.SPEC-007 (Member Removal Processing) | Triggers (outbound) | Confirmed removal starts this automation |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | Organiser-only entry point |
| FEAT-09.SPEC-001 (Household Invitations Manager) | Navigation (outbound) | Empty-state link to invite a new member |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| member_removal_confirmation_shown | member_type (adult / kid) | Maya opens the confirmation step for a member | N/A -- no Stage 2 success metric measures removal-flow engagement; retained to observe how often the confirmation step is reached versus completed |
| member_removed | member_type (adult / kid) | Removal completes successfully | N/A -- no Stage 2 success metric measures member removal; retained so this lifecycle action's frequency is observable given its cross-feature cascade (XBR-16) |

## Acceptance Criteria

**FEAT-18.SPEC-002-AC-01:** Given Maya is on the Remove a Member screen, when she taps Remove on Sam's row, then the confirmation step shows Sam's name and a summary of what will be lost.

**FEAT-18.SPEC-002-AC-02:** Given Maya is on the confirmation step for Sam, when she taps "Remove Sam", then Sam is removed and she sees the toast "Sam has been removed." and returns to the member list.

**FEAT-18.SPEC-002-AC-03:** Given Maya is on the confirmation step for a kid profile, when she taps "Cancel", then no removal occurs and she returns to the member list unchanged.

**FEAT-18.SPEC-002-AC-04:** Given Maya opens the Remove a Member screen, when the list loads, then her own organiser profile never appears as a removable row.

**FEAT-18.SPEC-002-AC-05:** Given Maya's household has no members besides herself, when she opens this screen, then she sees "There's no one else to remove yet -- invite a member from Household Settings to add one."

**FEAT-18.SPEC-002-AC-06:** Given Maya confirms Sam's removal on one device while Sam is simultaneously editing his own account on another device, when Maya's removal completes first, then Sam's concurrent edit is refused with a clear message, per the dependency map's Contention note.

**FEAT-18.SPEC-002-AC-07:** Given Maya has the confirmation step open for a member that a second organiser session already removed, when she taps Remove, then she sees "This member was already removed." and returns to the updated list.

**FEAT-18.SPEC-002-AC-08:** Given Maya loses connectivity on the confirmation step and taps Remove, when she is offline, then the banner "You're offline -- this removal will be sent when you reconnect." appears and the removal queues locally.

**FEAT-18.SPEC-002-AC-09:** Given FEAT-18.SPEC-007 reports a failure while removing a member, when Maya views the confirmation step, then she sees "We couldn't remove {member_name}. Try again." with a Retry button.

**FEAT-18.SPEC-002-AC-10:** Given Sam attempts to reach this screen directly, when the screen loads, then he sees "Only the household organiser can remove a member." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-002-AC-11:** Given Maya's session expires while she is on the confirmation step, when she next interacts with it, then a dialog reads "Your session has expired. Sign in to continue." and no removal was made.

**FEAT-18.SPEC-002-AC-12:** Given Maya opens this screen, when the initial fetch of the removable-member list is in progress, then the list area shows loading placeholders instead of member rows.

**FEAT-18.SPEC-002-AC-13:** Given the initial fetch of the removable-member list fails, when Maya views this screen, then she sees "We couldn't load the household's members. Try again." with a Retry button, and tapping Retry either loads the list normally or shows the same error again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 4 | 4 |
| States | 8 (loading, load error, member list, confirmation, removing, error, empty, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Delete Household

## Overview

**Name:** Delete Household
**ID:** FEAT-18.SPEC-003
**Type:** Screen
**Purpose:** Maya sees exactly what will be lost and confirms permanent deletion of the entire household.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Showing a summary of the household's members, plans, and history that deletion will remove
- The irreversible-action confirmation for household deletion
- Triggering deletion and showing its immediate, in-progress, and completed states

**Non-Goals:**
- Performing the cascade deletion across every household entity -- owned by FEAT-18.SPEC-008 (Household Deletion Processing), which this screen triggers
- Removing a single member without deleting the whole household -- owned by FEAT-18.SPEC-002 (Remove Member Profile)
- Deleting only the organiser's own account while the household continues -- owned by FEAT-18.SPEC-004 and FEAT-18.SPEC-009; this screen deletes the entire household, every member included
- Any billing cancellation mechanics -- deletion supersedes the household's Subscription regardless of billing_state; FEAT-14 owns billing cancellation as its own standalone action, which this screen does not duplicate

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-004 (My Account) | Maya taps "Delete household" in the organiser-only Household Data & Deletion section | None -- screen loads a fresh deletion-preview summary |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Confirm household deletion | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable from any navigation available to Sam; a direct attempt shows "Only the household organiser can delete the household." and returns him to FEAT-18.SPEC-004 |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable through this login; a direct attempt shows "Only the household organiser can delete the household." |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no deletion was in progress to preserve, since confirmation is a single action |

## Layout and Content

**Header:** Screen title "Delete Household" with a back arrow (returns to FEAT-18.SPEC-004).

**Body:** A single content column.
- A summary card reading exactly what will be lost, built from the household's current data: the number of member profiles (naming each by display_name), the number of weeks of plan history, the number of items on the current grocery list, and the line "Every dietary rule, rating, and support request will be permanently removed too."
- A warning line: "This cannot be undone. Deletion is permanent and there is no way to recover this household's data afterward."
- The confirmation control below the summary: a "Delete Household Permanently" button (destructive, enabled directly -- no precondition beyond the summary having loaded) and a "Cancel" action that returns to FEAT-18.SPEC-004.

**Confirmation step (appears in place of the Preview body when Delete Household Permanently is tapped):** The same summary of what will be lost, restated, plus the warning line, and two actions: "Delete Household Permanently" (destructive, requires the explicit affirmative tap) and "Cancel" (returns to the Preview state). No default-confirmed state -- neither action is preselected or triggered by any other interaction, consistent with the Shared Context's irreversible-action confirmation pattern (also used identically by FEAT-18.SPEC-002 and within FEAT-18.SPEC-004).

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-004 | Screen closes | Standard transition back |
| "Delete Household Permanently" (Preview) | Tap | Opens the confirmation step | Preview body is replaced by the confirmation step | Confirmation step appears restating the loss summary |
| Confirmation "Delete Household Permanently" | Tap | Triggers FEAT-18.SPEC-008 (Household Deletion Processing) | Screen switches to the in-progress state | Progress indicator with the label "Deleting your household..." |
| Confirmation "Cancel" | Tap | Discards the confirmation | Confirmation step is replaced by the Preview body | Preview body reappears unchanged |
| Cancel (Preview) | Tap | Discards the action | Navigates to FEAT-18.SPEC-004 | Standard transition back, no deletion occurs |

### Accessibility Notes

- **Focus order:** Back arrow -> summary card -> warning line -> Delete Household Permanently button -> Cancel button; on the confirmation step -> restated loss summary -> confirmation's Delete Household Permanently button -> Cancel button.
- **Confirmation announcement:** Entering the confirmation step announces the restated loss summary to assistive technology, so the destructive action's consequences are read before the confirmation's Delete Household Permanently control is reachable, identical to FEAT-18.SPEC-002's confirmation-step announcement.
- **Progress announcement:** Entering the in-progress state announces "Deleting your household" to assistive technology.
- **Keyboard alternatives:** Every action on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Summary card area shows loading placeholders in place of member/plan/list counts | Screen first opens, before the initial fetch of the deletion-preview summary completes | Fetch succeeds (-> Preview) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load your household's deletion summary. Try again." with a Retry button; no confirmation controls are shown until the summary loads | The initial fetch of the deletion-preview summary fails | Maya taps Retry (re-fetches) or the back arrow (returns to FEAT-18.SPEC-004) |
| Preview (default) | Summary card, warning, and an enabled Delete Household Permanently button | Deletion-preview summary loads successfully | Maya taps Delete Household Permanently or Cancel |
| Confirmation | Restated loss summary, warning, and Delete Household Permanently/Cancel buttons; no default-confirmed state | Maya taps Delete Household Permanently on the Preview state | Maya taps the confirmation's Delete Household Permanently (confirmed) or Cancel |
| Deleting | Progress indicator "Deleting your household..."; screen is non-interactive except a note that deletion continues even if she closes the app | Maya taps the confirmation's Delete Household Permanently button | Deletion processing (FEAT-18.SPEC-008) begins its cascade |
| Deletion in progress (post-confirmation) | A persistent notice replacing the household's usual areas: "Your household is being deleted. This can take up to 30 days to fully complete; you'll be signed out shortly and a completion email will follow." | Confirmation accepted by FEAT-18.SPEC-008 | The household record is fully removed and Maya is signed out |
| Confirm Error | Error banner "We couldn't start the deletion. Try again." with a Retry button; the confirmation step's summary remains | FEAT-18.SPEC-008 fails to begin processing | Maya taps Retry or Cancel |
| Offline/Degraded | Banner "You're offline -- deletion will begin once you reconnect." at top; the confirmation's Delete Household Permanently button queues the request locally | Connectivity lost while the confirmation step is open | Connectivity restored -- the queued confirmation submits automatically and the screen shows the Deleting state |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules), which defines the irreversible-action confirmation requirement. This screen's confirmation step is the enforcement of that rule for this specific action, identical to FEAT-18.SPEC-002's and FEAT-18.SPEC-004's confirmation steps.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | FEAT-18.SPEC-004 (My Account) | -- |
| Cancel tap | FEAT-18.SPEC-004 (My Account) | -- |
| Deletion confirmed | Signed-out landing (the product's sign-in screen) once deletion processing begins | -- |

## Data Model

**Creates:** None.
**Reads:** Household -- household_name, and a derived summary count of Member Profiles, Weekly Plans, and current Grocery List items belonging to the household.
**Updates:** None directly -- deletion is performed by FEAT-18.SPEC-008.
**Deletes:** None directly -- the Household record and its cascade are deleted by FEAT-18.SPEC-008.

## Business Rules

- Deletion requires the same explicit-affirmative-tap confirmation step used by FEAT-18.SPEC-002 (Remove Member Profile) and FEAT-18.SPEC-004 (My Account's own-account deletion), per FEAT-18.SPEC-010's irreversible-action confirmation rule -- the confirmation step states plainly what will be lost and has no default-confirmed state.
- Only Maya (Organiser) can reach this screen and confirm deletion, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).
- Household deletion supersedes any in-flight edit by any member, per the dependency map's Contention note for Household -- no concurrent edit can block or be lost silently; deletion always wins.
- Deletion is a hard delete with no restore path, distinct from any member's individual departure (FEAT-09), which is restorable in spirit for that member alone.

## Edge Cases

- **Sam is editing household settings on another screen (a capability outside this feature) at the moment Maya confirms deletion** -- Per the dependency map's Contention note for Household, deletion supersedes the in-flight edit; Sam's screen is refreshed to reflect the household's deletion-in-progress state, and any unsaved change of his is discarded. This is the concurrent-edit conflict entry for this screen.
- **Maya taps Delete Household Permanently, then taps Cancel on the confirmation step** -- No deletion occurs; the confirmation step closes back to the Preview state unchanged, identical in mechanics to FEAT-18.SPEC-002's Cancel behavior.
- **Maya closes the app immediately after confirming deletion** -- Deletion processing continues independently of the app being open; FEAT-18.SPEC-008 completes its cascade and FEAT-18.SPEC-014 delivers the completion notification regardless.
- **Maya navigates back to this screen after already confirming deletion in a prior session (deletion still in progress)** -- The screen shows the "Deletion in progress" state directly rather than the Preview summary, since the household is no longer in a state that supports a fresh deletion request.
- **A second organiser session attempts to confirm deletion while a first confirmation is already processing** -- The second confirmation is rejected with "Deletion is already in progress for this household." since only one deletion can be in flight per household at a time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Irreversible-action confirmation pattern |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can reach this screen |
| FEAT-18.SPEC-008 (Household Deletion Processing) | Triggers (outbound) | Confirmed deletion starts this automation |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Affects (outbound) | Delivered once FEAT-18.SPEC-008 completes, independent of this screen |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | Organiser-only entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| household_deletion_preview_viewed | member_count, plan_week_count | Maya opens this screen | N/A -- no Stage 2 success metric measures deletion-flow engagement; retained to observe how often the preview is reached versus confirmed, given the feature's Rationale that a credible export/delete path builds trust |
| household_deleted | -- | Deletion is confirmed and processing begins | N/A -- no Stage 2 success metric measures household deletion directly; retained since this is the terminal lifecycle event for a household and must be observable for the operator's own record-keeping |

## Acceptance Criteria

**FEAT-18.SPEC-003-AC-01:** Given Maya is on the Delete Household screen, when it loads, then the summary card shows her household's member count, plan-history length, and current grocery-list item count, plus the warning that deletion cannot be undone.

**FEAT-18.SPEC-003-AC-02:** Given Maya is on the Preview state, when she taps Delete Household Permanently, then the confirmation step opens restating what will be lost, and no deletion has occurred yet.

**FEAT-18.SPEC-003-AC-03:** Given Maya is on the confirmation step, when she taps the confirmation's Delete Household Permanently button, then the screen shows "Deleting your household..." and FEAT-18.SPEC-008 begins.

**FEAT-18.SPEC-003-AC-04:** Given Maya has confirmed deletion, when processing begins, then she sees "Your household is being deleted. This can take up to 30 days to fully complete; you'll be signed out shortly and a completion email will follow."

**FEAT-18.SPEC-003-AC-05:** Given Maya taps Cancel on the Preview state, when the action is processed, then no deletion occurs and she returns to FEAT-18.SPEC-004.

**FEAT-18.SPEC-003-AC-06:** Given Maya taps Cancel on the confirmation step, when the action is processed, then no deletion occurs and she returns to the Preview state, not FEAT-18.SPEC-004.

**FEAT-18.SPEC-003-AC-07:** Given Sam has an unsaved household-settings edit open when Maya confirms deletion, when deletion begins, then Sam's screen refreshes to the deletion-in-progress state and his unsaved edit is discarded.

**FEAT-18.SPEC-003-AC-08:** Given Maya loses connectivity on the confirmation step, when she taps the confirmation's Delete Household Permanently button, then the banner "You're offline -- deletion will begin once you reconnect." appears and the confirmation queues locally.

**FEAT-18.SPEC-003-AC-09:** Given FEAT-18.SPEC-008 fails to begin processing, when Maya views the confirmation step, then she sees "We couldn't start the deletion. Try again." with a Retry button.

**FEAT-18.SPEC-003-AC-10:** Given Sam attempts to reach this screen directly, when the screen loads, then he sees "Only the household organiser can delete the household." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-003-AC-11:** Given Maya's session expires while she is on this screen, when she next interacts with it, then a dialog reads "Your session has expired. Sign in to continue." and no deletion was confirmed.

**FEAT-18.SPEC-003-AC-12:** Given Maya returns to this screen after a deletion she confirmed in a prior session is still processing, when the screen loads, then it shows the deletion-in-progress state directly, not the Preview summary.

**FEAT-18.SPEC-003-AC-13:** Given Maya opens this screen, when the initial fetch of the deletion-preview summary is in progress, then the summary card area shows loading placeholders and no confirmation control is shown.

**FEAT-18.SPEC-003-AC-14:** Given the initial fetch of the deletion-preview summary fails, when Maya views this screen, then she sees "We couldn't load your household's deletion summary. Try again." with a Retry button, and no confirmation control is shown until it succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 5 | 5 |
| States | 8 (loading, load error, preview, confirmation, deleting, deletion in progress, confirm error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: My Account

## Overview

**Name:** My Account
**ID:** FEAT-18.SPEC-004
**Type:** Screen
**Purpose:** Any adult member views and edits their own name, email, and sign-in, or starts deleting their own account; the organiser additionally reaches household-wide export, removal, and deletion controls from here.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Viewing and editing the signed-in adult's own display_name, email, and sign-in credential
- Starting the signed-in adult's own account deletion
- For Maya only: an organiser-only "Household Data & Deletion" section linking to export, member removal, and household deletion
- Field-level validation and same-screen confirmation for own-account edits

**Non-Goals:**
- Editing another member's profile, dietary rules, budget, or schedule -- owned by FEAT-01 (Household Setup & Member Profiles), never reachable from this screen for anyone but the viewer's own record
- Performing the own-account deletion itself -- owned by FEAT-18.SPEC-009 (Own Account Deletion Processing), which this screen's deletion action triggers after the precondition check
- Editing notification preferences -- owned by FEAT-07 and FEAT-13, reached from household settings rather than this screen
- Kid-profile account management -- neither kid row has a login to manage (scope-boundaries.md SC-02); kid data is managed exclusively through FEAT-01 and this feature's organiser-only removal path (FEAT-18.SPEC-002)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Product's persistent navigation | Any signed-in adult taps "My Account" | None -- screen loads the viewer's own profile |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | New organiser taps "Review your account details" on the hand-over Success state | None -- screen loads the new organiser's own profile, now showing the organiser-only section |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Own profile fields plus the organiser-only Household Data & Deletion section | Edit own name/email/sign-in, delete own account (subject to the hand-over-or-delete-first precondition), reach Contact Support, and reach export/removal/household-deletion | -- |
| Sam (Other Adult Member) | Own profile fields only; the Household Data & Deletion section is not shown | Edit own name/email/sign-in, delete own account, reach Contact Support | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | This login has no Account & Data access (Access Matrix); the screen is not reachable, and a direct attempt shows "This isn't available for your login." |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | Partial (own field edits preserved) | No | Dialog "Your session has expired. Sign in to continue." -- entered field edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "My Account" with a back arrow (returns to the entry source).

**Body:** A single-column form for the viewer's own profile:
- Display Name (text input, required)
- Email (text input, required for adults, used for sign-in)
- "Change sign-in" action, opening a dedicated credential-update step (current credential, new credential, confirmation)
- "Contact Support" action, navigating to FEAT-18.SPEC-005 (Contact Support)
- A "Delete My Account" action at the bottom of the form, visually separated as a destructive action

**Organiser-only section (Maya only, shown below the form, separated by a divider labeled "Household Data & Deletion"):**
- "Export household data" -- links to FEAT-18.SPEC-001
- "Remove a member" -- links to FEAT-18.SPEC-002
- "Delete household" -- links to FEAT-18.SPEC-003

### Responsive Behavior

- **Compact breakpoint:** Single-column form and organiser-only section as described, full width.
- **Medium size class and above:** Form and organiser-only section remain single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition back |
| Display Name field | Type, then blur | Field-level validation runs inline | Error state on invalid input | "Display name is required" if left empty on blur |
| Email field | Type, then blur | Field-level validation runs inline | Error state on invalid input | "Enter a valid email address" on invalid format |
| Save (form-level, appears once a field changes) | Tap | Validates all changed fields, then saves | Button shows a brief loading state | Success: same-screen confirmation "Your account details have been updated." Failure: inline error messages per field |
| "Change sign-in" | Tap | Opens the credential-update step | Form is replaced by the credential-update step | Credential-update fields appear |
| Credential-update "Save" | Tap | Validates and updates the sign-in credential | Returns to the main form | Success: same-screen confirmation "Your sign-in details have been updated." Failure: inline error on the credential fields |
| "Contact Support" | Tap | Navigate to FEAT-18.SPEC-005 | Screen closes | Standard transition |
| "Delete My Account" | Tap | Checks the own-account deletion precondition via FEAT-18.SPEC-010 | Opens the confirmation step, or shows the blocked state | See States and Business Rules |
| Confirmation "Delete My Account" (in the deletion confirmation step) | Tap | Triggers FEAT-18.SPEC-009 (Own Account Deletion Processing) | Screen shows a progress state | Progress indicator; on completion, the viewer is signed out |
| "Export household data" (Maya only) | Tap | Navigate to FEAT-18.SPEC-001 | Screen closes | Standard transition |
| "Remove a member" (Maya only) | Tap | Navigate to FEAT-18.SPEC-002 | Screen closes | Standard transition |
| "Delete household" (Maya only) | Tap | Navigate to FEAT-18.SPEC-003 | Screen closes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Display Name -> Email -> Change sign-in -> Contact Support -> Delete My Account -> (Maya only) Export household data -> Remove a member -> Delete household.
- **Validation announcements:** Field error states are announced to assistive technology and programmatically associated with their field on blur.
- **Save confirmation announcement:** The "Your account details have been updated." confirmation is announced on success; on failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Form area shows loading placeholders in place of the profile fields | Screen first opens, before the initial fetch of the viewer's own profile completes | Fetch succeeds (-> Viewing) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load your account details. Try again." with a Retry button; no form fields or actions are shown until data loads | The initial fetch of the viewer's own profile fails | Viewer taps Retry (re-fetches) or the back arrow (returns to the entry source) |
| Viewing (default) | Form populated with the viewer's current details; Save button hidden until a field changes | Initial fetch succeeds | Viewer edits a field |
| Editing | Save button visible | Viewer changes any field | Viewer taps Save or navigates away |
| Saving | Save button shows a loading state, fields disabled | Viewer taps Save | Save completes or fails |
| Save error | Inline error banner "Couldn't save your changes. Try again." with entered values preserved | Save fails | Viewer taps Save again or corrects a field |
| Deletion blocked (organiser only) | Blocking dialog: "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with actions "Hand over role" (to FEAT-09.SPEC-003) and "Delete household instead" (to FEAT-18.SPEC-003), plus "Cancel" | Maya taps Delete My Account while she is still the organiser with no completed hand-over (FEAT-18.SPEC-010) | Maya taps one of the offered actions or Cancel |
| Deletion confirmation | Loss summary and Delete/Cancel buttons, consistent with the feature's irreversible-action confirmation pattern | Deletion precondition passes (non-organiser adult, or organiser who has handed over) | Viewer confirms or cancels |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; the form remains editable and Save queues the edit locally | Connectivity lost while this screen is open | Connectivity restored -- the queued save submits automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules) for the own-account deletion precondition and the irreversible-action confirmation. Field-level format and required-field rules for display_name, email, and sign-in credential are governed by FEAT-01's field validation rules for Member Profile, referenced here rather than duplicated, since this screen edits the same fields FEAT-01 defines.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | Entry source | -- |
| "Contact Support" tap | FEAT-18.SPEC-005 (Contact Support) | -- |
| "Export household data" (Maya) | FEAT-18.SPEC-001 (Export Household Data) | -- |
| "Remove a member" (Maya) | FEAT-18.SPEC-002 (Remove Member Profile) | -- |
| "Delete household" (Maya) | FEAT-18.SPEC-003 (Delete Household) | -- |
| "Hand over role" (deletion blocked, Maya) | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | FEAT-09 |
| "Delete household instead" (deletion blocked, Maya) | FEAT-18.SPEC-003 (Delete Household) | -- |
| Successful own-account deletion | The product's sign-in screen (viewer is signed out) | -- |

## Data Model

**Creates:** None.
**Reads:** Member Profile -- display_name, sign_in (email component), member_type, status, for the signed-in viewer's own record; for Maya, the Household's organiser field to determine whether the organiser-only section and the hand-over check apply.
**Updates:** Member Profile -- display_name, sign_in, for the viewer's own record only (own-only, per the Access Matrix).
**Deletes:** None directly -- own-account deletion is performed by FEAT-18.SPEC-009.

## Business Rules

- Every adult member may edit only their own display_name and sign_in; there is no path from this screen to another member's fields (Access Matrix: Account & Data, Own-only for Sam).
- The organiser cannot delete her own account while she remains the organiser and has not handed over the role or deleted the household, per XBR-15 and FEAT-18.SPEC-010.
- Only Maya sees the Household Data & Deletion section, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).
- Field-level edits save inline without navigating away, consistent with product-features.md's Primary Flows for own-account management.

## Edge Cases

- **Sam edits his display name on one device while Maya removes him as a member on another device** -- Per the dependency map's Contention note for Member Profile, removal wins and Sam's concurrent save is refused with "This account no longer exists in this household." rather than a generic save error. This is the concurrent-edit conflict entry for this screen.
- **Maya taps Delete My Account immediately after handing over the organiser role to Sam, before Sam's acceptance is recorded** -- The hand-over is not complete until FEAT-09.SPEC-009 finishes processing Sam's acceptance; Maya still sees the Deletion blocked state until that completes, since she remains the organiser of record until then.
- **Maya's household is deleted (FEAT-18.SPEC-008) while she has this screen open** -- Household deletion supersedes any in-flight edit here (dependency map's Contention note for Household); her screen is redirected to the sign-in screen once the household record is removed.
- **Sam changes his sign-in email to one already used by another member of a different household** -- No conflict exists across households (sign-in identities are global, not household-scoped), so this is not a validation case this spec addresses; any global-uniqueness rule on sign-in credentials is governed by FEAT-01's field validation rules.
- **Viewer navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Own-account deletion precondition and confirmation pattern |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs own-only edit scope and the organiser-only section |
| FEAT-18.SPEC-009 (Own Account Deletion Processing) | Triggers (outbound) | Confirmed deletion starts this automation |
| FEAT-18.SPEC-005 (Contact Support) | Navigation (outbound) | Link to submit a general support request |
| FEAT-18.SPEC-001 (Export Household Data) | Navigation (outbound) | Organiser-only link |
| FEAT-18.SPEC-002 (Remove Member Profile) | Navigation (outbound) | Organiser-only link |
| FEAT-18.SPEC-003 (Delete Household) | Navigation (outbound) | Organiser-only link |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Navigation (outbound) | Offered when deletion is blocked for the organiser |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| own_account_updated | fields_changed (name / email / sign_in) | An own-account field edit saves successfully | N/A -- no Stage 2 success metric measures account-detail edits; retained as the basic account-management usage signal |
| own_account_deletion_blocked | reason (organiser_no_handover) | Maya attempts deletion while still organiser with no completed hand-over | N/A -- no Stage 2 success metric measures this precondition's frequency; retained to observe how often the organiser-hand-over gate is actually hit |
| own_account_deleted | role_at_deletion (organiser / other_adult) | Own-account deletion completes | N/A -- no Stage 2 success metric measures own-account deletion; retained as the terminal event for this lifecycle action |

## Acceptance Criteria

**FEAT-18.SPEC-004-AC-01:** Given Sam is on My Account, when he changes his display name and taps Save, then he sees the confirmation "Your account details have been updated." with no navigation away from the screen.

**FEAT-18.SPEC-004-AC-02:** Given Sam leaves the Email field empty and blurs it, when validation runs, then he sees "Enter a valid email address" if the format is invalid, or the field's required-error if it is empty.

**FEAT-18.SPEC-004-AC-03:** Given Sam (Other Adult Member) is on My Account, when the screen loads, then no Household Data & Deletion section is shown.

**FEAT-18.SPEC-004-AC-04:** Given Maya (Organiser) is on My Account, when the screen loads, then the Household Data & Deletion section with Export, Remove a member, and Delete household links is shown.

**FEAT-18.SPEC-004-AC-05:** Given Maya taps Delete My Account while she is still the organiser and has not handed over the role, when the check runs, then she sees "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with links to hand over or delete the household.

**FEAT-18.SPEC-004-AC-06:** Given Sam (never the organiser) taps Delete My Account, when the precondition check runs, then he proceeds directly to the deletion confirmation step.

**FEAT-18.SPEC-004-AC-07:** Given Maya has completed handing over the organiser role, when she taps Delete My Account, then she proceeds to the deletion confirmation step rather than seeing the blocked state.

**FEAT-18.SPEC-004-AC-08:** Given Sam confirms his own account deletion, when he taps the confirmation Delete My Account, then FEAT-18.SPEC-009 processes the deletion and he is signed out on completion.

**FEAT-18.SPEC-004-AC-09:** Given Sam is removed as a member by Maya while he has an unsaved edit open, when he taps Save, then he sees "This account no longer exists in this household." rather than a generic save error.

**FEAT-18.SPEC-004-AC-10:** Given the viewer loses connectivity while editing a field, when they tap Save, then the banner "You're offline -- changes will be saved when you reconnect." appears and the edit queues locally.

**FEAT-18.SPEC-004-AC-11:** Given the older-kid limited login (Later) attempts to reach My Account, when the screen loads, then the message "This isn't available for your login." is shown and no account fields are exposed.

**FEAT-18.SPEC-004-AC-12:** Given the viewer's session expires while editing a field, when they next interact with the form, then a dialog reads "Your session has expired. Sign in to continue." and the entered edit is preserved and restored after re-authentication.

**FEAT-18.SPEC-004-AC-13:** Given the viewer has unsaved changes and taps the back arrow, when the navigation attempt occurs, then a confirmation dialog reads "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-18.SPEC-004-AC-14:** Given the viewer opens My Account, when the initial fetch of their own profile is in progress, then the form area shows loading placeholders instead of profile fields.

**FEAT-18.SPEC-004-AC-15:** Given the initial fetch of the viewer's own profile fails, when they view this screen, then they see "We couldn't load your account details. Try again." with a Retry button, and tapping Retry either loads the form normally or shows the same error again.

**FEAT-18.SPEC-004-AC-16:** Given Sam is on My Account, when he taps Contact Support, then he is navigated to FEAT-18.SPEC-005 (Contact Support).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 11 | 11 |
| States | 9 (loading, load error, viewing, editing, saving, save error, deletion blocked, deletion confirmation, offline) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Contact Support

## Overview

**Name:** Contact Support
**ID:** FEAT-18.SPEC-005
**Type:** Screen
**Purpose:** Any adult member sends a short description of a problem to the operator and receives an emailed acknowledgement.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Submitting a short description of a general support problem
- Creating a Support Request of kind "general support contact"
- Showing the same-screen confirmation that the request was received

**Non-Goals:**
- Reporting a safety concern about a specific meal -- a distinct Support Request kind owned by FEAT-02 (Dietary Rules & Allergy Safety Engine), reached from a planned meal, not from this screen
- Any in-product back-and-forth after submission -- excluded per scope-boundaries.md SC-14: this screen sends a one-way description and the household receives a one-time email acknowledgement; further exchange happens outside the product
- Viewing the status of a submitted request or the operator's access record -- owned by FEAT-01 (organiser's view of open requests and access records) and FEAT-22 (operator's status management)
- Riley (Operator) using this screen -- it is a household-facing submission form only; Riley's support access is a separate, read-only capability (FEAT-22)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Product's persistent navigation | Any signed-in adult taps "Contact Support" | None -- screen starts empty |
| FEAT-18.SPEC-004 (My Account) | Adult member taps "Contact Support" in the account form | None -- screen starts empty |
| FEAT-18.SPEC-001 (Export Household Data) | Maya taps the "contact support" link from the export Error state | None -- screen starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Submit a support description | -- |
| Sam (Other Adult Member) | Full screen | Submit a support description | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | This login has no Account & Data access (Access Matrix); a direct attempt shows "This isn't available for your login." |
| Riley (Operator, support -- from v1) | No | No | Screen is not part of Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | Partial (entered draft preserved) | No | Dialog "Your session has expired. Sign in to continue." -- entered description is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Contact Support" with a back arrow (returns to the entry source).

**Body:** A single-column form:
- An explanatory line: "Tell us what's going wrong and we'll get back to you by email."
- Description (multi-line text input, required, up to 500 characters, with a live character count)
- "Send" action button

**Footer:** None -- Send is the form's sole action.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; the description field grows with content up to a platform-wide maximum visible height before scrolling internally.
- **Medium size class and above:** Form capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition back |
| Description field | Type | Captures text input; live character count updates | Character count reflects remaining characters | Count shown as "{used}/500" |
| Description field | Blur (empty) | Triggers required-field validation via FEAT-18.SPEC-010 | Error state on the field | "Tell us what's going wrong before sending" below the field |
| Send button | Tap | 1. Validates the description via FEAT-18.SPEC-010. 2. Creates a general-support Support Request. 3. Triggers FEAT-18.SPEC-015 (Support Request Acknowledgement). | Button shows a loading state during submission | Success: same-screen confirmation "Thanks -- we've got your message and will follow up by email." and the form clears. Failure: inline error with a Retry option, description preserved. |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> explanatory line -> Description field -> Send button.
- **Character-count announcement:** The remaining-character count is available to assistive technology on request but is not announced on every keystroke, to avoid interrupting typing.
- **Confirmation announcement:** The success confirmation and any error message are announced when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Description field empty, Send button enabled | Screen first opens | Member begins typing |
| Filling | Description field contains text | Member types | Member taps Send or navigates away |
| Sending | Send button shows a loading state, field disabled | Member taps Send with a non-empty description | Submission completes or fails |
| Sent | Form clears; same-screen confirmation "Thanks -- we've got your message and will follow up by email." shown | Submission succeeds | Member navigates away or begins a new message |
| Error | Error banner "We couldn't send your message. Try again." with a Retry button; entered description preserved | Submission fails | Member taps Retry |
| Offline/Degraded | Banner "You're offline -- this message will be sent when you reconnect." at top; Send queues the message locally | Connectivity lost while this screen is open | Connectivity restored -- the queued message submits automatically and the Sent confirmation appears |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules). See that spec for the description's required-field and length rules.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | Entry source | -- |
| Successful send | This screen (Sent state) | -- |

## Data Model

**Creates:** Support Request -- kind "general support contact", raised_by (the submitting member), note (the description), status "Raised".
**Reads:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Every adult member (Maya and Sam) can submit a support description, per FEAT-18.SPEC-011 (Account & Data Authorization Rules); neither kid row can, since neither has Account & Data access.
- A submitted Support Request always creates exactly one acknowledgement (FEAT-18.SPEC-015) -- there is no path to submit without an acknowledgement being scheduled.
- Description length is capped and required, per FEAT-18.SPEC-010.

## Edge Cases

- **Member taps Send twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Member navigates away with an unsent description** -- Confirmation dialog: "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.
- **Description at exactly 500 characters** -- Submission proceeds; the field simply does not accept further input beyond the limit, so no length-error state is reachable from typing alone.
- **Network failure during send** -- Error banner: "We couldn't send your message. Try again." with a Retry button; the description text is preserved.
- **Member submits a second support description before the first's acknowledgement has arrived** -- Each submission creates its own independent Support Request and its own acknowledgement (FEAT-18.SPEC-015); there is no concurrent-edit conflict here, since this screen only creates new records and never updates a shared entity another member could contend for.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Description field validation |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can submit |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | Triggers (outbound) | Successful submission triggers the acknowledgement |
| FEAT-22 (Operator Read-Only Support Access) | Triggers (outbound, cross-feature) | A submitted request opens Riley's read-only support access for the household |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| support_contacted | description_length | A general support description is submitted successfully | N/A -- no Stage 2 success metric measures support contact volume; retained so this trust-facing capability's actual use is observable, per feature-overview.md's Rationale |
| support_contact_failed | reason (validation / network) | Submission fails | N/A -- no Stage 2 success metric measures submission failures; retained to observe whether this path is reliable enough to trust |

## Acceptance Criteria

**FEAT-18.SPEC-005-AC-01:** Given Sam is on the Contact Support screen, when he types a description and taps Send, then a general-support Support Request is created and he sees "Thanks -- we've got your message and will follow up by email."

**FEAT-18.SPEC-005-AC-02:** Given Maya is on the Contact Support screen, when she taps Send with the description field empty, then she sees "Tell us what's going wrong before sending" and no request is created.

**FEAT-18.SPEC-005-AC-03:** Given Sam has typed 500 characters into the description field, when he attempts to type further, then no additional characters are accepted and the field stays at 500.

**FEAT-18.SPEC-005-AC-04:** Given Maya submits a support description successfully, when submission completes, then FEAT-18.SPEC-015 is triggered to send her an acknowledgement.

**FEAT-18.SPEC-005-AC-05:** Given Sam loses connectivity and taps Send, when he is offline, then the banner "You're offline -- this message will be sent when you reconnect." appears and the message queues locally.

**FEAT-18.SPEC-005-AC-06:** Given connectivity returns while a message is queued, when it submits automatically, then Sam sees the Sent confirmation without re-tapping Send.

**FEAT-18.SPEC-005-AC-07:** Given a submission fails due to a network error, when Maya views the screen, then she sees "We couldn't send your message. Try again." with her description preserved.

**FEAT-18.SPEC-005-AC-08:** Given the older-kid limited login (Later) attempts to reach this screen, when the screen loads, then "This isn't available for your login." is shown and no form is exposed.

**FEAT-18.SPEC-005-AC-09:** Given Maya has an unsent description and taps the back arrow, when the navigation attempt occurs, then a confirmation dialog reads "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 5 | 5 |
| States | 6 (empty, filling, sending, sent, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Export Generation Processing

## Overview

**Name:** Export Generation Processing
**ID:** FEAT-18.SPEC-006
**Type:** Automation
**Purpose:** Compiles all of a household's records into a readable export file and makes it available for download.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Compiling the household's plans, ratings, lists, dietary rules, and settings into one readable export file
- Retrying on failure and reporting exhausted retries clearly
- Making the completed file available for download and signaling readiness for the export-ready notification

**Non-Goals:**
- Deciding when an export may be requested -- the rate-limit condition is governed by FEAT-18.SPEC-010 (Account & Data Validation Rules) and enforced before this automation is triggered by FEAT-18.SPEC-001
- Delivering the export-ready confirmation -- owned by FEAT-18.SPEC-013 (Export Ready Notification), which this automation's success outcome triggers
- Deleting any data -- excluded per product-features.md: export is read-only compilation; it never removes or modifies the source records it reads

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Export requested | FEAT-18.SPEC-001 (Export Household Data) | Fires when Maya taps Request Export and FEAT-18.SPEC-010's rate-limit check passes | Household reference, request timestamp |

## Processing Logic

1. Receive the export request for the household (household reference, request timestamp).
2. Read every record belonging to the household across its owning features: Household settings, every Member Profile (including kid profiles' minimal data), every Dietary Rule, every Weekly Plan and its Planned Meals, the current and archived Grocery Lists, every Rating, and Support Request history.
3. Compile the read records into one readable export file, organized by record type, in a format a household member can open and read without specialized software.
4. Mark the export as ready and record its ready date once compilation completes.
5. Make the completed file available for download from FEAT-18.SPEC-001 and retain it as a previous export entry.
6. Signal FEAT-18.SPEC-013 (Export Ready Notification) that the export is ready.
7. If compilation fails at any step, retry automatically up to the automation's retry limit before reporting failure.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Export ready | Compilation completes successfully | A new export file record is created for the household, marked ready with its ready date | FEAT-18.SPEC-001 shows "Download Export"; FEAT-18.SPEC-013 delivers the ready confirmation | FEAT-18.SPEC-001, FEAT-18.SPEC-013, FEAT-18.SPEC-012 |
| Retry in progress | Compilation fails on an attempt but retries remain | No export file record is finalized yet | FEAT-18.SPEC-001 continues showing "Compiling your export..." -- retries are not surfaced individually as a distinct visible state | FEAT-18.SPEC-001 |
| Export failed | Compilation fails and the retry limit is exhausted | No export file record is created for this request | FEAT-18.SPEC-001 shows "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support." | FEAT-18.SPEC-001, FEAT-18.SPEC-005 |

## Data Model

**Reads:** Household -- all fields; Member Profile -- all fields for every household member; Dietary Rule -- all fields; Weekly Plan and Planned Meal -- all fields; Grocery List and Grocery List Item -- all fields; Rating -- all fields; Support Request -- all fields.
**Creates:** An export file record for the household, with its ready date and download reference.
**Updates:** None -- compilation is read-only against the source records.
**Deletes:** None.

## Business Rules

- Export generation is asynchronous and always shows progress rather than an indefinite wait, per product-features.md's States field.
- A failed compilation is retried automatically; only after retries are exhausted is the failure reported to the household, per product-features.md's States field.
- Each successful compilation produces exactly one export file record, retained as a previous export the household can re-download.
- Export generation never modifies or removes any source record it reads (product-features.md, Data Notes: export is derived data, never a mutation).

## Edge Cases

- **A household's data volume is unusually large (years of accumulated plans and ratings)** -- Compilation still runs to completion; the household sees the same "Compiling your export..." progress state for as long as it takes, per the Non-Functional Notes' expectation that generation stays reasonably fast even for a household with years of history.
- **A member's profile is removed (FEAT-18.SPEC-007) while an export is compiling** -- The export reflects the household's data as of the moment compilation began; a member removed mid-compilation may or may not appear in the resulting file depending on exactly when their records were read, and this is an accepted characteristic of a point-in-time export rather than a defect.
- **Concurrent trigger firing (two export requests for the same household at effectively the same time)** -- Cannot occur in practice: FEAT-18.SPEC-010's rate-limit check and FEAT-18.SPEC-001's disabled-while-compiling state together ensure only one export request per household is accepted while a prior one is in flight.
- **Trigger fires while a previous run is in flight** -- A second request for the same household is rejected before reaching this automation, per FEAT-18.SPEC-001's Compiling state disabling further requests; this automation therefore never runs two compilations for the same household concurrently.
- **Household is deleted while an export is compiling** -- Household deletion (FEAT-18.SPEC-008) supersedes the in-flight export per the dependency map's Contention note for Household; the compilation is cancelled and no export file is finalized or delivered.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-001 (Export Household Data) | Triggered by (inbound) | Request Export starts this automation |
| FEAT-18.SPEC-001 (Export Household Data) | Affects (outbound) | Progress, ready, and error states surface here |
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Rate-limit check runs before this automation is triggered |
| FEAT-18.SPEC-013 (Export Ready Notification) | Triggers (outbound) | The export-ready outcome fires this notification |
| FEAT-18.SPEC-008 (Household Deletion Processing) | References (inbound) | Household deletion cancels an in-flight compilation |

## Analytics and Success Signals

- **data_export_completed** (record_count_by_type) -- N/A -- no Stage 2 success metric measures export completion; retained per product-features.md's Signals field (data_export_completed) as the operational record of this lifecycle action
- **data_export_failed** (retries_attempted) -- N/A -- no Stage 2 success metric measures export failures; retained to observe whether the automated-retry-then-report behavior is ever actually exercised

## Acceptance Criteria

**FEAT-18.SPEC-006-AC-01:** Given Maya's export request passes the rate-limit check, when this automation starts, then it reads every record type belonging to her household and begins compiling the export file.

**FEAT-18.SPEC-006-AC-02:** Given compilation completes successfully, when the export file is finalized, then FEAT-18.SPEC-001 shows "Download Export" and FEAT-18.SPEC-013 is triggered.

**FEAT-18.SPEC-006-AC-03:** Given compilation fails on its first attempt, when a retry remains, then the automation retries automatically without reporting failure to Maya.

**FEAT-18.SPEC-006-AC-04:** Given compilation fails and retries are exhausted, when the final attempt fails, then FEAT-18.SPEC-001 shows "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support."

**FEAT-18.SPEC-006-AC-05:** Given a completed export file exists for a household, when Maya requests a new export later, then a new, independent export file is created and the prior one remains available as a previous export.

**FEAT-18.SPEC-006-AC-06:** Given a household is deleted while its export is compiling, when household deletion processing (FEAT-18.SPEC-008) completes its cascade, then the in-flight compilation is cancelled and no export file is finalized.

**FEAT-18.SPEC-006-AC-07:** Given no export request currently exists for a household, when a rate-limited request attempt is rejected by FEAT-18.SPEC-010, then this automation is never triggered.

**FEAT-18.SPEC-006-AC-08:** Given a household's data spans several years of plans and ratings, when this automation compiles the export, then it produces one complete export file covering the full history, with progress shown throughout.

**FEAT-18.SPEC-006-AC-09:** Given Maya's household already has an export compiling, when a second export request for the same household is attempted, then it never reaches this automation, since FEAT-18.SPEC-001's disabled Compiling state prevents the second trigger.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (ready, retry in progress, failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Member Removal Processing

## Overview

**Name:** Member Removal Processing
**ID:** FEAT-18.SPEC-007
**Type:** Automation
**Purpose:** On confirmed member removal, permanently deletes the member's dietary rules and ratings and stops future plans from accounting for them.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Deleting the removed member's Member Profile, Dietary Rules, and Ratings
- Ensuring future plan generation and manual planning stop accounting for the removed member
- Reporting success or failure back to the triggering screen

**Non-Goals:**
- The removal confirmation UI itself -- owned by FEAT-18.SPEC-002 (Remove Member Profile), which triggers this automation
- A member's own self-initiated departure -- a distinct outcome (anonymised ratings, not deleted) owned by FEAT-09.SPEC-008 (Member Departure Processing)
- Removing the organiser -- never a valid input to this automation; FEAT-18.SPEC-002 never offers the organiser as a removable target (XBR-15)
- Household-wide deletion -- owned by FEAT-18.SPEC-008 (Household Deletion Processing), a distinct, larger-scope automation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member removal confirmed | FEAT-18.SPEC-002 (Remove Member Profile) | Fires when Maya confirms removal in the irreversible-action confirmation step | Member Profile reference (the member being removed), household reference |

## Processing Logic

1. Receive the confirmed removal request (member reference, household reference).
2. Re-verify the target member is still an active, non-organiser member of the household (guards against the race described in Edge Cases).
3. Delete every Dietary Rule belonging to the member.
4. Delete every Rating authored by the member.
5. Set the Member Profile's status to Removed and remove it from the household's active member list.
6. Ensure the removed member is excluded from any future plan generation (FEAT-03) or manual planning (FEAT-23) pick lists and from the safety check's per-member evaluation, from this point forward.
7. Signal FEAT-18.SPEC-002 that removal completed, so the screen can show its success toast and refresh the member list.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Removal completed | All steps succeed | Member Profile status set to Removed; the member's Dietary Rules and Ratings deleted | FEAT-18.SPEC-002 shows "{member_name} has been removed." and returns to the member list | FEAT-18.SPEC-002, FEAT-01, FEAT-12 |
| Target no longer valid | The re-verification step finds the member already removed or no longer non-organiser (a race, per Edge Cases) | No data changes | FEAT-18.SPEC-002 shows "This member was already removed." | FEAT-18.SPEC-002 |
| Removal failed | A deletion step fails partway | No partial state is left visible: the member remains Active and none of its Dietary Rules or Ratings are deleted until the whole sequence can complete | FEAT-18.SPEC-002 shows "We couldn't remove {member_name}. Try again." with a Retry button | FEAT-18.SPEC-002 |

## Data Model

**Reads:** Member Profile -- status, member_type, for the target member and to confirm the requester is the organiser.
**Creates:** None.
**Updates:** Member Profile -- status (set to Removed).
**Deletes:** Dietary Rule -- every rule belonging to the removed member; Rating -- every rating authored by the removed member.

## Business Rules

- XBR-16: removing a member deletes their dietary rules and ratings and future plans stop accounting for them -- this is a hard delete, distinct from FEAT-09's self-leave path, which anonymises rather than deletes ratings.
- The organiser can never be the target of this automation (XBR-15); FEAT-18.SPEC-002 never offers her as a removable member, and this automation treats an organiser target as an invalid input it must never receive.
- This automation is non-reversible: once Dietary Rules and Ratings are deleted, no restore path exists (product-features.md, Primary Flows & Alternates: "permanently removed").
- The removed member's data feeds no future plan or safety check from the moment removal completes (Cross-Feature Touchpoints: FEAT-01, FEAT-12).

## Edge Cases

- **Concurrent trigger firing (two organiser sessions confirm removal of the same member at effectively the same time)** -- The re-verification step in Processing Logic step 2 ensures only the first-committed removal proceeds; the second run finds the member already Removed and reports "Target no longer valid" rather than attempting a duplicate deletion.
- **Trigger fires while a previous run is in flight for a different member** -- Runs for different members proceed independently; each member's Dietary Rule and Rating deletions are scoped to that member alone and do not block or interact with a concurrent removal of someone else.
- **Sam is mid-edit of his own account (FEAT-18.SPEC-004) when this automation removes him** -- Per the dependency map's Contention note for Member Profile, this automation's removal wins; Sam's concurrent save is refused with "This account no longer exists in this household."
- **The removed member has a rating pending recording by another adult (e.g., an adult was about to record a young kid's rating on the removed kid's behalf)** -- Once the removal completes, no new Rating can be created for the removed member; an in-flight rating submission for that member is rejected with the same "This member no longer exists" style message.
- **Household is deleted while a member removal is still processing** -- Household deletion (FEAT-18.SPEC-008) supersedes any in-flight member removal per the dependency map's Contention note for Household; the removal's remaining steps are superseded by the household-wide cascade rather than run to completion separately.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-002 (Remove Member Profile) | Triggered by (inbound) | Confirmed removal starts this automation |
| FEAT-18.SPEC-002 (Remove Member Profile) | Affects (outbound) | Success, race, and failure feedback surface here |
| FEAT-18.SPEC-008 (Household Deletion Processing) | References (inbound) | Household deletion supersedes an in-flight member removal |
| FEAT-01 (Household Setup & Member Profiles) | Affects (outbound, cross-feature) | The removed member's profile and dietary rules disappear from FEAT-01's lists |
| FEAT-12 (Meal Rating & Preference Learning) | Affects (outbound, cross-feature) | The removed member's ratings no longer feed learned preferences |

## Analytics and Success Signals

- **member_deleted** (member_type: adult / kid) -- N/A -- no Stage 2 success metric measures member removal directly; retained per product-features.md's Signals field (member_deleted) as the operational record of this lifecycle action
- **member_removal_failed** (step_failed) -- N/A -- no Stage 2 success metric measures removal failures; retained to observe whether the no-partial-state guarantee is ever actually exercised

## Acceptance Criteria

**FEAT-18.SPEC-007-AC-01:** Given Maya confirms removal of Sam on FEAT-18.SPEC-002, when this automation processes the request, then Sam's Dietary Rules and Ratings are deleted and his Member Profile status is set to Removed.

**FEAT-18.SPEC-007-AC-02:** Given a kid profile is removed, when this automation completes, then the kid's Dietary Rules and Ratings are deleted identically to an adult member's.

**FEAT-18.SPEC-007-AC-03:** Given a member is removed, when the next plan generation or manual planning session runs, then the removed member is excluded from all planning and safety-check evaluation.

**FEAT-18.SPEC-007-AC-04:** Given two organiser sessions confirm removal of the same member at effectively the same time, when the second run's re-verification runs, then it finds the member already Removed and FEAT-18.SPEC-002 shows "This member was already removed." on that session.

**FEAT-18.SPEC-007-AC-05:** Given Sam has an unsaved own-account edit open when this automation removes him, when he attempts to save, then he sees "This account no longer exists in this household." rather than a generic error.

**FEAT-18.SPEC-007-AC-06:** Given a deletion step fails partway through processing, when the failure is reported, then no partial state is visible -- the member remains Active with all Dietary Rules and Ratings intact -- and FEAT-18.SPEC-002 shows "We couldn't remove {member_name}. Try again."

**FEAT-18.SPEC-007-AC-07:** Given a household is deleted while a member removal for that household is in flight, when household deletion processing (FEAT-18.SPEC-008) runs its cascade, then the member removal's remaining steps are superseded by the household-wide deletion.

**FEAT-18.SPEC-007-AC-08:** Given this automation is invoked (defensively) with the organiser as its target, when processing begins, then it treats this as an invalid input and takes no action, since FEAT-18.SPEC-002 never offers the organiser as removable.

**FEAT-18.SPEC-007-AC-09:** Given two different members are removed by two separate confirmed requests at the same time, when both run, then each completes independently without blocking or interacting with the other.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (completed, target no longer valid, failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Household Deletion Processing

## Overview

**Name:** Household Deletion Processing
**ID:** FEAT-18.SPEC-008
**Type:** Automation
**Purpose:** On confirmed household deletion, permanently removes every member profile, plan, list, rule, and rating within 30 days.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Signing out every household member once deletion begins
- Cascading the deletion across every household entity: Member Profile, Weekly Plan, Planned Meal, Grocery List, Grocery List Item, Pantry Item, Rating, Dietary Rule, Support Request, Subscription
- Disconnecting the household's calendar connection, if one exists, via FEAT-21.SPEC-002 (Family Calendar Integration)
- Completing the full cascade within 30 days and triggering the completion notification

**Non-Goals:**
- The deletion confirmation UI -- owned by FEAT-18.SPEC-003 (Delete Household), which triggers this automation
- Cancelling billing separately -- deletion supersedes the Subscription regardless of billing_state; this automation does not duplicate FEAT-14's own cancellation flow, it simply removes the Subscription record as part of the cascade
- Removing a single member without deleting the household -- owned by FEAT-18.SPEC-007 (Member Removal Processing)
- Any restore path -- this is an intentional hard delete with no recovery mechanism, per product-features.md's Primary Flows & Alternates and the Non-Goals in feature-overview.md

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household deletion confirmed | FEAT-18.SPEC-003 (Delete Household) | Fires when Maya taps the confirmation step's explicit affirmative "Delete Household Permanently" action | Household reference |

## Processing Logic

1. Receive the confirmed deletion request (household reference).
2. Mark the Household's status as Closed/Deleted immediately, so no further edits, plans, or actions can be made against it from this moment.
3. Sign out every member currently signed in to the household within a short window of confirmation.
4. Begin cascading deletion across every owned entity: every Member Profile, every Weekly Plan and its Planned Meals, the Grocery List and its Grocery List Items, every Pantry Item, every Rating, every Dietary Rule, every Support Request, and the Subscription.
5. Disconnect the household's calendar connection, if one exists, via FEAT-21.SPEC-002 (Family Calendar Integration) -- sent as part of the cascade regardless of the connection's current status; a household with no calendar connection has nothing to disconnect and this step completes as a no-op.
6. Continue the cascade until every listed entity is fully removed, completing within 30 days of confirmation.
7. Once the cascade completes, permanently remove the Household record itself.
8. Signal FEAT-18.SPEC-014 (Household Deletion Completed Notification) that deletion has fully completed.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Deletion started | Confirmation accepted | Household status set to Closed/Deleted; every member signed out | FEAT-18.SPEC-003 shows "Your household is being deleted..." before members are signed out | FEAT-18.SPEC-003 |
| Deletion completed | The full cascade finishes within the 30-day window | Every listed entity permanently removed; any active calendar connection disconnected (FEAT-21.SPEC-002); the Household record itself removed | FEAT-18.SPEC-014 delivers the completion confirmation to Maya | FEAT-18.SPEC-014, FEAT-18.SPEC-012, FEAT-21.SPEC-002 |
| Deletion start failed | Step 2 or 3 cannot be completed (e.g., the status change itself fails) | No change -- the household remains Active and reachable | FEAT-18.SPEC-003 shows "We couldn't start the deletion. Try again." | FEAT-18.SPEC-003 |

## Data Model

**Reads:** Household -- all fields, to identify every owned entity for the cascade.
**Creates:** None.
**Updates:** Household -- status (set to Closed/Deleted at the start of processing).
**Deletes:** Member Profile, Weekly Plan, Planned Meal, Grocery List, Grocery List Item, Pantry Item, Rating, Dietary Rule, Support Request, Subscription -- every record belonging to the household; and, on cascade completion, the Household record itself.

## Business Rules

- Household deletion supersedes any in-flight edit by any member, per the dependency map's Contention note for Household -- once step 2 marks the household Closed/Deleted, no concurrent action on any owned entity is honored.
- Deletion completes within 30 days of confirmation and no deleted data is retained in any form usable for other purposes (product-features.md, Validation & Limits; ASMP-27) -- the 30-day window is a stated product decision, not an instant purge, and is a fixed, feature-level value stated as a concrete number because it is specific to this feature's own definition, not a platform-wide policy value delegated to build time.
- This is a hard delete with no restore path (XBR-16), distinct from a member's individual self-leave path (FEAT-09), which is restorable in spirit for that member alone.
- Only one household deletion runs per household at a time -- a second confirmation attempt while one is already processing is rejected, per FEAT-18.SPEC-003's Edge Cases.

## Edge Cases

- **Concurrent trigger firing (two confirmation attempts for the same household at effectively the same time)** -- Only the first-committed confirmation proceeds; a second attempt is rejected with "Deletion is already in progress for this household." per FEAT-18.SPEC-003's Edge Cases, since the household's status is already Closed/Deleted by the time the second attempt is evaluated.
- **Trigger fires while a previous run is in flight** -- Cannot occur for the same household: once status is Closed/Deleted, FEAT-18.SPEC-003 no longer offers a fresh confirmation. Runs for different households proceed entirely independently.
- **An export (FEAT-18.SPEC-006) is compiling when deletion begins** -- Deletion supersedes the in-flight export per the dependency map's Contention note for Household; the export compilation is cancelled and no export file is finalized.
- **A member is mid-swap, mid-plan-approval, or otherwise mid-action on any household entity when deletion begins** -- Every such in-flight action is superseded; the acting member's screen refreshes to reflect the household's Closed/Deleted state, and no partial data change from that in-flight action persists.
- **The household's Subscription is mid-grace-period (a payment failure) when deletion is confirmed** -- The Subscription record is deleted as part of the cascade regardless of billing_state; no separate cancellation flow is required to complete first.
- **The household has no calendar connection (never connected, or already disconnected) when deletion runs** -- The disconnect step (FEAT-21.SPEC-002) finds no active connection to disconnect and completes as a no-op; the rest of the cascade proceeds unaffected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-003 (Delete Household) | Triggered by (inbound) | Confirmed deletion starts this automation |
| FEAT-18.SPEC-003 (Delete Household) | Affects (outbound) | Deletion-in-progress and error feedback surface here |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Triggers (outbound) | The completed outcome fires this notification |
| FEAT-18.SPEC-006 (Export Generation Processing) | Affects (outbound) | An in-flight export is cancelled by this cascade |
| FEAT-18.SPEC-007 (Member Removal Processing) | Affects (outbound) | An in-flight member removal is superseded by this cascade |
| FEAT-21.SPEC-002 (Family Calendar Integration) | Triggers (outbound, cross-feature) | Cascade disconnects any active household calendar connection |
| FEAT-01, FEAT-03, FEAT-06, FEAT-05, FEAT-12 (cross-feature) | Affects (outbound, cross-feature) | Every entity these features own within the household is removed by this cascade |

## Analytics and Success Signals

- **household_deleted** (member_count_at_deletion) -- N/A -- no Stage 2 success metric measures household deletion directly; retained per product-features.md's Signals field (household_deleted) as the terminal record of this lifecycle action
- **household_deletion_completed** (days_to_complete) -- N/A -- no Stage 2 success metric measures deletion completion timing; retained to confirm the 30-day commitment is actually met in practice

## Acceptance Criteria

**FEAT-18.SPEC-008-AC-01:** Given Maya confirms household deletion, when this automation starts, then the Household status is set to Closed/Deleted and every signed-in member is signed out shortly after.

**FEAT-18.SPEC-008-AC-02:** Given deletion has started, when the cascade runs, then every Member Profile, Weekly Plan, Grocery List, Pantry Item, Rating, Dietary Rule, and Support Request belonging to the household is permanently removed.

**FEAT-18.SPEC-008-AC-03:** Given the cascade completes within 30 days, when the Household record is finally removed, then FEAT-18.SPEC-014 delivers the completion confirmation to Maya.

**FEAT-18.SPEC-008-AC-04:** Given a second confirmation attempt is made for a household already marked Closed/Deleted, when it is evaluated, then it is rejected with "Deletion is already in progress for this household."

**FEAT-18.SPEC-008-AC-05:** Given an export is compiling for a household when deletion is confirmed, when deletion processing begins, then the export compilation is cancelled and no export file is finalized.

**FEAT-18.SPEC-008-AC-06:** Given Sam is mid-edit of a shared entity when deletion is confirmed, when the household status changes to Closed/Deleted, then his screen refreshes to reflect the closed household and his in-flight action is superseded.

**FEAT-18.SPEC-008-AC-07:** Given the household's Subscription is in a Payment failed grace period when deletion is confirmed, when the cascade runs, then the Subscription record is deleted regardless of its billing_state.

**FEAT-18.SPEC-008-AC-08:** Given step 2 (marking the household Closed/Deleted) fails, when this occurs, then the household remains Active and FEAT-18.SPEC-003 shows "We couldn't start the deletion. Try again."

**FEAT-18.SPEC-008-AC-09:** Given a member removal for the household is in flight when household deletion is confirmed, when the household-wide cascade runs, then the individual member removal's remaining steps are superseded by it.

**FEAT-18.SPEC-008-AC-10:** Given two different households each confirm deletion at the same time, when both are processed, then each household's cascade completes independently without interacting with the other.

**FEAT-18.SPEC-008-AC-11:** Given a household with an active calendar connection confirms deletion, when the cascade runs, then this automation signals FEAT-21.SPEC-002 (Family Calendar Integration) to disconnect that connection as part of the cascade.

**FEAT-18.SPEC-008-AC-12:** Given a household with no calendar connection confirms deletion, when the cascade reaches the calendar-disconnect step, then the step finds nothing to disconnect and completes as a no-op without affecting the rest of the cascade.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (started, completed, start failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Own Account Deletion Processing

## Overview

**Name:** Own Account Deletion Processing
**ID:** FEAT-18.SPEC-009
**Type:** Automation
**Purpose:** Permanently deletes an adult's own account, blocked for the organiser unless the role was handed over or the household deleted first.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Re-checking the organiser hand-over-or-delete-first precondition immediately before deletion
- Permanently deleting the requesting adult's own Member Profile, Dietary Rules, and Ratings
- Reporting success or a blocked outcome back to FEAT-18.SPEC-004

**Non-Goals:**
- The precondition's definition -- owned by FEAT-18.SPEC-010 (Account & Data Validation Rules), which this automation enforces rather than redefines
- Organiser role hand-over itself -- owned by FEAT-09 (Household Invitations & Membership); this automation only checks whether a hand-over has completed
- Deleting a member other than the requester's own account -- that is organiser-initiated removal, owned by FEAT-18.SPEC-007 (Member Removal Processing)
- Household-wide deletion -- owned by FEAT-18.SPEC-008 (Household Deletion Processing); an organiser may choose that path instead of hand-over, but this automation only deletes one adult's own account

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Own-account deletion confirmed | FEAT-18.SPEC-004 (My Account) | Fires when the adult member confirms deletion in the irreversible-action confirmation step, after the precondition check has already passed once at that point | Member Profile reference (the requester), household reference |

## Processing Logic

1. Receive the confirmed own-account deletion request (member reference, household reference).
2. Re-check the organiser-hand-over-or-delete-first precondition (FEAT-18.SPEC-010): if the requester is currently the household's organiser and no hand-over has completed and the household has not been deleted, block the deletion.
3. If the precondition passes, delete every Dietary Rule belonging to the requester.
4. Delete every Rating authored by the requester.
5. Set the requester's Member Profile status to Removed and remove it from the household's active member list.
6. Ensure the requester is excluded from any future plan generation, manual planning, or safety-check evaluation from this point forward.
7. Sign the requester out.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Deletion completed | Precondition passes and all steps succeed | Member Profile status set to Removed; requester's Dietary Rules and Ratings deleted | Requester is signed out; FEAT-18.SPEC-004 shows a signed-out landing | FEAT-18.SPEC-004, FEAT-01, FEAT-12 |
| Precondition blocked | Requester is still the organiser with no completed hand-over and the household has not been deleted (re-checked at processing time, not just at the initial screen check) | No data changes | FEAT-18.SPEC-004 shows the deletion-blocked message again, since the precondition state changed between the screen check and processing | FEAT-18.SPEC-004 |
| Deletion failed | A deletion step fails partway | No partial state is left visible: the requester's Member Profile remains Active and none of its Dietary Rules or Ratings are deleted | FEAT-18.SPEC-004 shows "We couldn't delete your account. Try again." with a Retry button | FEAT-18.SPEC-004 |

## Data Model

**Reads:** Member Profile -- status, member_type, for the requester; Household -- organiser, status, to evaluate the precondition.
**Creates:** None.
**Updates:** Member Profile -- status (set to Removed).
**Deletes:** Dietary Rule -- every rule belonging to the requester; Rating -- every rating authored by the requester.

## Business Rules

- XBR-15: the organiser cannot delete her own account while she remains the organiser and has not handed over the role or deleted the household first -- this automation re-checks that precondition at processing time, not only at the screen-level check, since the household's organiser state could change between the two moments.
- Own-account deletion is a hard delete with no restore path (product-features.md, Primary Flows & Alternates), distinct from a member's self-leave path (FEAT-09.SPEC-005), which anonymises rather than deletes ratings.
- The deleted requester's data feeds no future plan or safety check from the moment deletion completes.
- Only the requesting adult's own account can be the target of this automation -- it is never invoked with a target other than the session's own Member Profile.

## Edge Cases

- **Concurrent trigger firing (the organiser starts a hand-over acceptance on one device while confirming her own deletion on another)** -- The re-check in Processing Logic step 2 reads the household's organiser state at the moment this automation runs; whichever change (hand-over completion or deletion confirmation) is durably recorded first determines the outcome, consistent with first-committed-wins.
- **Trigger fires while a previous run is in flight for the same requester (double confirmation)** -- FEAT-18.SPEC-004's disabled confirmation button during the Deleting state prevents a second trigger for the same requester while the first is processing.
- **The organiser's hand-over completes seconds before she confirms deletion, but the completion has not yet propagated to her own screen's precondition check** -- Because this automation re-checks the precondition at processing time (step 2), a hand-over that completed before this automation runs is honored even if FEAT-18.SPEC-004's earlier screen-level check was stale.
- **The requester is removed by the organiser (FEAT-18.SPEC-007) at the same moment they confirm their own deletion** -- Per the dependency map's Contention note for Member Profile, whichever change is recorded first wins; if the organiser's removal lands first, this automation finds the requester already Removed and reports a blocked-equivalent outcome ("This account no longer exists in this household.") rather than double-deleting.
- **Household is deleted while own-account deletion is processing** -- Household deletion (FEAT-18.SPEC-008) supersedes this automation per the dependency map's Contention note for Household; the household-wide cascade completes the requester's removal as part of its own scope.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-004 (My Account) | Triggered by (inbound) | Confirmed own-account deletion starts this automation |
| FEAT-18.SPEC-004 (My Account) | Affects (outbound) | Completed, blocked, and failure feedback surface here |
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Defines the hand-over-or-delete-first precondition this automation enforces |
| FEAT-09 (Household Invitations & Membership) | References (inbound, cross-feature) | Organiser hand-over completion satisfies the precondition |
| FEAT-18.SPEC-008 (Household Deletion Processing) | References (inbound) | Household deletion supersedes an in-flight own-account deletion |

## Analytics and Success Signals

- **own_account_deleted** (role_at_deletion: organiser / other_adult) -- N/A -- no Stage 2 success metric measures own-account deletion directly; retained per product-features.md's Signals field (own_account_deleted) as the operational record of this lifecycle action
- **own_account_deletion_blocked_at_processing** (reason: organiser_no_handover) -- N/A -- no Stage 2 success metric measures this race outcome; retained to observe whether the screen-level and processing-time precondition checks ever genuinely disagree

## Acceptance Criteria

**FEAT-18.SPEC-009-AC-01:** Given Sam (never the organiser) confirms his own account deletion, when this automation processes the request, then his Dietary Rules and Ratings are deleted, his Member Profile status is set to Removed, and he is signed out.

**FEAT-18.SPEC-009-AC-02:** Given Maya has completed handing over the organiser role and then confirms her own deletion, when this automation re-checks the precondition, then it passes and her account is deleted.

**FEAT-18.SPEC-009-AC-03:** Given Maya is still the organiser with no completed hand-over when this automation processes a deletion request for her, when the precondition check runs, then the deletion is blocked and FEAT-18.SPEC-004 shows the deletion-blocked message again.

**FEAT-18.SPEC-009-AC-04:** Given Maya's hand-over completes between her screen-level check and this automation's processing, when the automation re-checks the precondition, then it passes based on the now-current organiser state.

**FEAT-18.SPEC-009-AC-05:** Given Maya is removed by another organiser session (a defensive, unusual case) at the same moment she confirms her own deletion, and the removal is recorded first, when this automation runs, then it finds her already Removed and reports "This account no longer exists in this household." rather than deleting a second time.

**FEAT-18.SPEC-009-AC-06:** Given a deletion step fails partway through processing, when the failure is reported, then no partial state is visible -- the requester's Member Profile remains Active -- and FEAT-18.SPEC-004 shows "We couldn't delete your account. Try again."

**FEAT-18.SPEC-009-AC-07:** Given a household is deleted while a requester's own-account deletion is in flight, when household deletion processing (FEAT-18.SPEC-008) runs its cascade, then the requester's removal is completed as part of that cascade instead.

**FEAT-18.SPEC-009-AC-08:** Given Sam has already confirmed his own deletion once and it is processing, when he attempts to confirm again, then FEAT-18.SPEC-004's disabled Deleting state prevents a second trigger.

**FEAT-18.SPEC-009-AC-09:** Given a deleted adult's account is later checked against any future plan generation, when planning runs, then the deleted account is excluded from all planning and safety-check evaluation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (completed, precondition blocked, failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Account & Data Validation Rules

## Overview

**Name:** Account & Data Validation Rules
**ID:** FEAT-18.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs export rate-limiting, irreversible-action confirmation, the 30-day purge window, the organiser hand-over-or-delete-first precondition, offline queuing, and the support-contact description's field rules.
**Parent Feature:** FEAT-18 -- Account & Data Management
**Governed Entity:** Household and Member Profile (the export, removal, and deletion lifecycle actions), plus Support Request (the general-support description)

## Scope and Non-Goals

**In Scope:**
- The export rate-limit condition and its denied behavior
- The irreversible-action confirmation pattern shared by member removal, household deletion, and own-account deletion
- The 30-day purge/no-other-use rule for deleted data
- The organiser hand-over-or-delete-first precondition for own-account deletion
- Offline queuing behavior for every action this feature defines
- Field validation for the Support Request description submitted through FEAT-18.SPEC-005

**Non-Goals:**
- Who may perform each action -- role-based authorization is governed by FEAT-18.SPEC-011 (Account & Data Authorization Rules), not this spec
- Field validation for a member's own display_name, email, and sign-in credential -- governed by FEAT-01's field validation rules for Member Profile, which this feature's My Account screen (FEAT-18.SPEC-004) references rather than duplicates
- Actually performing export compilation, member removal, household deletion, or own-account deletion -- owned respectively by FEAT-18.SPEC-006, FEAT-18.SPEC-007, FEAT-18.SPEC-008, and FEAT-18.SPEC-009, which enforce the rules defined here

## Governed Entity

**Entity:** Household and Member Profile (lifecycle actions), plus Support Request (creation)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Household.status | enum | Active or Closed/Deleted; set to Closed/Deleted at the start of household deletion processing |
| Member Profile.status | enum | Active, Invited, Left, or Removed (the shared enum, per the Feature Dependency Map); this feature sets Removed for both organiser-initiated removal and own-account deletion, distinguished only by which flow performed it |
| Household.organiser | reference | The Member Profile currently holding the organiser role; read to evaluate the own-account deletion precondition |
| Support Request.kind | enum | "general support contact" for requests created by this feature |
| Support Request.note | text | The submitted description, up to 500 characters for support contact |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-18.SPEC-001 | Export Household Data | On Request Export tap (rate-limit condition) |
| FEAT-18.SPEC-002 | Remove Member Profile | On the confirmation step's Remove action (irreversible-action confirmation) |
| FEAT-18.SPEC-003 | Delete Household | On the confirmation step's Delete Household Permanently action (irreversible-action confirmation) |
| FEAT-18.SPEC-004 | My Account | On Delete My Account tap (own-account deletion precondition) and on its own confirmation step (irreversible-action confirmation) |
| FEAT-18.SPEC-005 | Contact Support | On description field blur and on Send submit (field validation) |
| FEAT-18.SPEC-006 | Export Generation Processing | On processing (30-day-no-other-use rule governs how the export is compiled and retained) |
| FEAT-18.SPEC-007 | Member Removal Processing | On processing (30-day purge rule for the removed member's data) |
| FEAT-18.SPEC-008 | Household Deletion Processing | On processing (30-day purge window for the full cascade) |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | On processing (re-check of the organiser hand-over-or-delete-first precondition) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|---------------|------------------|-----------|
| Support Request.note | Required, non-empty | Always (kind: general support contact) | On blur, on submit | "Tell us what's going wrong before sending" | Yes |
| Support Request.note | Max 500 characters | Always | On change (input is capped, not error-shown) | -- input simply stops accepting further characters at 500 -- | No -- the field enforces the cap by not accepting further input, so no separate over-limit error state is reachable |
| Support Request.kind | No validation beyond data type | Always -- system-assigned as "general support contact" for every request created through FEAT-18.SPEC-005 | -- | -- | -- |
| Household.status | No validation beyond data type | Always -- system-managed transition, never directly entered by any user | -- | -- | -- |
| Member Profile.status | No validation beyond data type | Always -- system-managed transition, never directly entered by any user | -- | -- | -- |
| Household.organiser | No validation beyond data type | Always -- read-only for this spec's purposes; changed only through FEAT-09's hand-over flow | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|-------------------|-------|-------------------|
| Household closure supersedes member status | Household.status, Member Profile.status | Once Household.status is Closed/Deleted, no Member Profile.status transition through any flow other than the household-deletion cascade itself is honored | "Your household has been deleted." (shown on any screen attempting an action against a closed household) |
| Own-account deletion precondition | Household.organiser, Member Profile (the requester) | Own-account deletion for the requester is permitted only when the requester is not the current Household.organiser, or the household itself has been deleted | "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." |

## Authorization Rules

N/A -- role-based authorization for every action this feature defines (export, remove member, delete household, manage own account, contact support) is governed entirely by FEAT-18.SPEC-011 (Account & Data Authorization Rules), which is this feature's single authoritative home for the role-action matrix. This spec governs only the entity-level, rate-limit, confirmation, precondition, and offline rules above.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-----------------------|-----------------|--------------------------|
| Household.status | Active | On household creation (owned by FEAT-01); set to Closed/Deleted only by FEAT-18.SPEC-008 | No |
| Member Profile.status (on removal or own-account deletion) | Set to Removed, whether organiser-initiated or self-initiated | On confirmed removal (FEAT-18.SPEC-007) or confirmed own-account deletion (FEAT-18.SPEC-009) | No |
| Export rate-limit window | Rolls forward on a fixed cadence: platform parameter: `data-export-rate-limit-window` | Always, recalculated once the current window elapses | No |
| Export rate-limit count | Resets to zero at the start of each new rate-limit window | On window rollover | No |

## Business Rules

- **Export rate limit:** For Maya, the Request Export action is allowed only while the household's export count for the current rate-limit window is below platform parameter: `data-export-rate-limit-count`. At the limit, further export requests are blocked until the window resets, with the denied behavior: "You've reached this period's export limit. You can request another export once the limit resets." (feature-overview.md, Validation & Limits: "export requests are rate-limited to a reasonable frequency to prevent abuse"; the exact count and window are platform-set values, per contract C-36, since no specific number is stated in Stage 2.)
- **Irreversible-action confirmation:** Member removal (FEAT-18.SPEC-002), household deletion (FEAT-18.SPEC-003), and own-account deletion (within FEAT-18.SPEC-004) each require the same explicit affirmative action from Maya (or the requesting adult for own-account deletion): a confirmation step that states plainly and specifically what will be lost, with an explicit affirmative tap and no default-confirmed state on any of the three (feature-overview.md, Shared Context: Irreversible-action confirmation, "Spec Writers for all three should describe the pattern identically").
- **30-day purge / no-other-use rule:** Deleted data (from member removal, own-account deletion, or household deletion) is fully removed within 30 days of confirmation and is not retained in any form usable for any other purpose during or after that window (product-features.md, Validation & Limits; ASMP-27). This 30-day figure is a fixed, feature-level product decision stated as a concrete number, not a platform-wide policy value delegated to build time, since it is already established in Stage 2 (scope-boundaries.md SC-18, product-features.md).
- **Organiser hand-over-or-delete-first precondition (XBR-15):** The organiser must hand over the role to an active adult member (FEAT-09) or delete the household (FEAT-18.SPEC-003) before her own account can be deleted. This precondition is checked once at the FEAT-18.SPEC-004 screen level and re-checked at FEAT-18.SPEC-009's processing time, since the organiser state can change between the two moments.
- **Offline queuing:** Any export request, member removal confirmation, household deletion confirmation, own-account deletion confirmation, own-account field edit, or support-contact submission made without connectivity queues locally on the requesting device and completes automatically once connectivity returns (product-features.md, States field: Offline-degraded). This rule is governed once here and referenced by every screen in this feature (FEAT-18.SPEC-001 through FEAT-18.SPEC-005) rather than re-described.

## Edge Cases

- **Household reaches exactly the export rate limit at the same moment two organiser sessions each attempt a request** -- The check reads the current window's count at the moment each request's validation runs; whichever request's check runs after the count reaches the limit is blocked, even if its submission started slightly before the count-reaching request completed (first-decision-wins, consistent with the processing order in FEAT-18.SPEC-006).
- **Export rate-limit window rolls over while a request is mid-validation** -- The validation uses the count as of the moment it runs; a request evaluated just after rollover is checked against the new window's (reset) count.
- **Maya opens the confirmation step on FEAT-18.SPEC-003, then taps Cancel instead of the confirmation's Delete Household Permanently button** -- No deletion occurs; the confirmation step closes back to the Preview state, since no default-confirmed state exists and only the explicit affirmative tap proceeds.
- **Maya's organiser status changes (hand-over completes) between her screen-level precondition check and her processing-time confirmation** -- The processing-time re-check in FEAT-18.SPEC-009 uses the household's organiser state as of that moment, so a hand-over completed in between is honored even though the earlier screen-level check reflected the old state.
- **A support description is submitted at exactly 500 characters** -- Passes validation; the field's input cap means 501 characters is never reachable through typing, so no separate "too long" error state exists for this field.
- **A household is deleted while a member's own-account deletion precondition check is in flight** -- Per the Cross-Field Rules entry, the household's Closed/Deleted status supersedes; the pending own-account deletion is unnecessary once the whole household is gone, and FEAT-18.SPEC-009 treats this as already-satisfied (no further action needed for that member specifically).

## Acceptance Criteria

**FEAT-18.SPEC-010-AC-01:** Given Maya's household is under the export rate limit, when she requests an export, then the request is accepted and FEAT-18.SPEC-006 begins.

**FEAT-18.SPEC-010-AC-02:** Given Maya's household is at the export rate limit, when she attempts a new export request, then it is blocked with "You've reached this period's export limit. You can request another export once the limit resets."

**FEAT-18.SPEC-010-AC-03:** Given two organiser sessions each attempt an export request as the household's count is exactly at the limit, when both checks run, then the request whose check runs after the count reaches the limit is blocked, even if it started slightly earlier.

**FEAT-18.SPEC-010-AC-04:** Given Maya is on the Remove Member Profile confirmation step, when she has not yet tapped the explicit Remove button, then no removal has occurred, since no default-confirmed state exists.

**FEAT-18.SPEC-010-AC-05:** Given Maya taps Delete Household Permanently on FEAT-18.SPEC-003's Preview state, when the confirmation step opens, then no deletion has occurred, since no default-confirmed state exists.

**FEAT-18.SPEC-010-AC-06:** Given Maya is on FEAT-18.SPEC-003's confirmation step, when she taps Cancel instead of the confirmation's Delete Household Permanently button, then no deletion occurs and she returns to the Preview state.

**FEAT-18.SPEC-010-AC-07:** Given a member's data has been deleted through removal or own-account deletion, when the 30-day window is checked, then no form of that data is used for any purpose other than the removal itself during or after that window.

**FEAT-18.SPEC-010-AC-08:** Given Maya is still the household's organiser and has not handed over the role, when she attempts to delete her own account, then the precondition blocks her with the hand-over-or-delete-first message.

**FEAT-18.SPEC-010-AC-09:** Given Maya has handed over the organiser role, when the precondition check runs again, then it passes and her own-account deletion proceeds.

**FEAT-18.SPEC-010-AC-10:** Given Maya's hand-over completes between her screen-level check and this rule's processing-time re-check, when the re-check runs, then it reflects the now-current organiser state and passes.

**FEAT-18.SPEC-010-AC-11:** Given Sam loses connectivity while editing his own account, when he saves, then the edit queues locally and completes automatically once connectivity returns.

**FEAT-18.SPEC-010-AC-12:** Given Maya loses connectivity while confirming a household deletion, when she confirms, then the request queues locally and begins processing once connectivity returns.

**FEAT-18.SPEC-010-AC-13:** Given Sam submits a support description with the field empty, when he attempts to send it, then he sees "Tell us what's going wrong before sending" and no request is created.

**FEAT-18.SPEC-010-AC-14:** Given Sam has typed exactly 500 characters into the support description, when he attempts to type further, then no additional characters are accepted.

**FEAT-18.SPEC-010-AC-15:** Given a household is marked Closed/Deleted, when any other flow attempts a Member Profile status transition against it, then the attempt is refused with "Your household has been deleted."

**FEAT-18.SPEC-010-AC-16:** Given a member's own-account deletion precondition check is in flight when the household is deleted, when the household-wide deletion completes, then the member's own-account deletion is treated as already satisfied by the household-wide cascade.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 0 (N/A -- covered by FEAT-18.SPEC-011) | 0 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Account & Data Authorization Rules

## Overview

**Name:** Account & Data Authorization Rules
**ID:** FEAT-18.SPEC-011
**Type:** Logic/Rule
**Purpose:** Governs who can export household data, remove members, delete the household, manage their own account, or contact support, as the single authoritative home for role-gated behavior across every screen in this feature.
**Parent Feature:** FEAT-18 -- Account & Data Management
**Governed Entity:** Household and Member Profile (the access dimension -- which role may view or act on each, across FEAT-18.SPEC-001 through FEAT-18.SPEC-005)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for every action this feature defines: export household data, remove a member, delete the household, manage own account (edit, delete), and contact support
- The exact unauthorized experience for every role denied an action
- What an unauthenticated visitor and an expired session see across this feature's screens

**Non-Goals:**
- Field-level validation, rate-limiting, confirmation patterns, the 30-day purge rule, and the organiser hand-over-or-delete-first precondition -- governed by FEAT-18.SPEC-010, which is more specific to those entity-level and process rules
- Authorization for features outside this one (e.g., who may edit household budget or schedule) -- each feature owning its own entities defines its own authorization; this spec covers only the actions FEAT-18 itself defines
- Operator (Riley) authentication or the mechanics of the read-only support view itself -- owned by FEAT-22 (Operator Read-Only Support Access); this spec only states that Riley has no access of any kind to this feature's actions or data (Access Matrix: Account & Data column is None for Riley), not even a read-only view

## Governed Entity

**Entity:** Household and Member Profile (access dimension)
**Source:** Feature Dependency Map; Access Matrix in user-persona.md

| Field | Data Type | Description |
|-------|-----------|-------------|
| (No new fields -- this spec governs actions on the existing Household and Member Profile fields already defined in FEAT-18.SPEC-010) | -- | -- |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-18.SPEC-001 | Export Household Data | On screen entry and on Request Export |
| FEAT-18.SPEC-002 | Remove Member Profile | On screen entry and on the confirmation Remove action |
| FEAT-18.SPEC-003 | Delete Household | On screen entry and on the confirmation Delete Household Permanently action |
| FEAT-18.SPEC-004 | My Account | On screen entry (own-only field scope and the organiser-only section's visibility) and on every save/delete action |
| FEAT-18.SPEC-005 | Contact Support | On screen entry and on Send |
| FEAT-18.SPEC-007 | Member Removal Processing | On processing (re-check the requester is the organiser and the target is not the organiser) |
| FEAT-18.SPEC-008 | Household Deletion Processing | On processing (re-check the requester is the organiser) |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | On processing (re-check the requester acts only on their own account) |

## Field Validation Rules

N/A -- this spec governs authorization (who may act), not field-level data validation, which is FEAT-18.SPEC-010's scope. No field in the governed entities carries validation rules distinct from those already defined there.

## Cross-Field Rules

N/A -- no cross-field data rule applies at the authorization layer; every rule here is a role-action-condition triple, captured in full in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Export household data | Maya (Organiser) | Under the export rate limit (FEAT-18.SPEC-010) | Screen entry point (FEAT-18.SPEC-001) not shown to Sam at all; a direct navigation attempt shows "Only the household organiser can export household data." and returns him to FEAT-18.SPEC-004 |
| Remove a member (adult or kid, excluding the organiser) | Maya (Organiser) | The target is not the organiser (XBR-15) and irreversible-action confirmation is completed (FEAT-18.SPEC-010) | Screen entry point (FEAT-18.SPEC-002) not shown to Sam; a direct navigation attempt shows "Only the household organiser can remove a member." and returns him to FEAT-18.SPEC-004 |
| Delete the household | Maya (Organiser) | Irreversible-action confirmation is completed (FEAT-18.SPEC-010) | Screen entry point (FEAT-18.SPEC-003) not shown to Sam; a direct navigation attempt shows "Only the household organiser can delete the household." and returns him to FEAT-18.SPEC-004 |
| View own account details | Maya, Sam | Always, for the viewer's own record only | -- |
| Edit own name, email, or sign-in | Maya, Sam | Own-only -- only the requesting member's own Member Profile fields | No control exists on FEAT-18.SPEC-004 to edit any field of a Member Profile other than the viewer's own; there is no "select another member" path on this screen |
| Delete own account | Sam | Always, for Sam's own account, subject to no additional precondition | -- |
| Delete own account | Maya | Only when she is not the current organiser or the household has been deleted (FEAT-18.SPEC-010's precondition) | Blocking dialog: "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with links to FEAT-09.SPEC-003 and FEAT-18.SPEC-003 |
| View the Household Data & Deletion section (Export, Remove Member, Delete Household links) | Maya (Organiser) | Always | Section is not rendered at all on FEAT-18.SPEC-004 for Sam -- not hidden-but-present, absent from the layout entirely |
| Contact support | Maya, Sam | Always, for any adult member | -- |
| View, export, or manage household data in any form | Jordan (young kid profile, no login -- MVP) | Never | N/A -- no login exists for a young kid profile; no session can reach any screen in this feature |
| View, export, or manage household data in any form | Jordan (older kid, limited login -- Later) | Never | This login has no Account & Data access (Access Matrix: None); a direct attempt shows "This isn't available for your login." |
| View or act on any action this feature defines | Riley (Operator, support) | Never | N/A -- Riley's access is limited entirely to the separate read-only support view (FEAT-22, XBR-14); no screen or action in this feature has any operator path, not even a read-only one |
| Manage another member's account details (edit or delete) | Maya | Never -- the organiser cannot edit or delete another adult's own account through this feature | No control exists on any FEAT-18 screen for the organiser to edit or delete another member's own-account fields; removal (a distinct action, deleting the whole profile including data) is the only organiser-initiated action against another member, via FEAT-18.SPEC-002 |

## Defaults and Derivations

N/A -- this spec assigns no default or derived field values of its own; all defaults relevant to this feature are defined in FEAT-18.SPEC-010.

## Business Rules

- Every screen in this feature (FEAT-18.SPEC-001 through FEAT-18.SPEC-005) references this spec for its role-gated behavior rather than restating access rules per screen, per this feature's Shared Validation section.
- An unauthorized visitor -- anyone not signed in as a member of the household -- sees only the sign-in screen; no screen in this feature ever renders household or account data to an unauthenticated session.
- A session that expires mid-flow on any screen in this feature shows the dialog "Your session has expired. Sign in to continue." and returns the user to the sign-in screen, preserving in-progress own-account field edits (FEAT-18.SPEC-004 only, since it is the sole screen in this feature with editable, non-destructive form fields).
- Organiser-only authorization (export, remove member, delete household, own-account-deletion precondition) is enforced at both the screen layer (FEAT-18.SPEC-001, FEAT-18.SPEC-002, FEAT-18.SPEC-003, FEAT-18.SPEC-004 each check the acting member's current role or ownership) and the processing layer (FEAT-18.SPEC-007, FEAT-18.SPEC-008, FEAT-18.SPEC-009 each re-check at the moment of processing, not only at screen entry), since a role can change between screen load and action -- for example, an organiser hand-over completing mid-session.
- Roles named in this spec trace exactly to the Access Matrix in user-persona.md: Maya (Organiser), Sam (Other Adult Member), Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later), and Riley (Operator, support, from v1), plus the unauthenticated and expired-session states this feature's own screens define. No role beyond these is ever introduced by this feature.

## Edge Cases

- **An unauthenticated visitor attempts to navigate directly to any FEAT-18 screen** -- Redirected to the sign-in screen; no household or account data is ever rendered, even momentarily, during the redirect.
- **Sam attempts to bypass the hidden organiser-only controls via a direct navigation URL to FEAT-18.SPEC-001, FEAT-18.SPEC-002, or FEAT-18.SPEC-003** -- The destination screen itself enforces the same authorization check on entry, showing the exact denied message for that action regardless of how the screen was reached.
- **The organiser role transfers to Sam while Maya still has FEAT-18.SPEC-004 open with the Household Data & Deletion section visible** -- Her next attempt to use an organiser-only link on that screen (e.g., tapping Delete Household) is rejected at the destination screen's own entry check, showing the exact denied message, since authorization is evaluated at action time, not at screen-load time.
- **Riley's operator session somehow reaches a FEAT-18 URL directly (defensive case)** -- No such entry point exists for the operator role in this feature; no household or account data is ever rendered to an operator session outside FEAT-22's own read-only support view.
- **A kid profile somehow attempts a direct action against this feature (defensive case, e.g., a stale token)** -- No login exists for a young kid profile and the older-kid login (Later) has no entitlement to this feature's actions (Access Matrix: None); no action succeeds under either identity.
- **Maya attempts to reach another member's own-account edit fields through any means (e.g., a manipulated request)** -- No control or path exists on any FEAT-18 screen to expose another member's own-account fields to the organiser; the only organiser-initiated action against another member's data is removal (FEAT-18.SPEC-002), a distinct, whole-profile action, never a field-level edit.

## Acceptance Criteria

**FEAT-18.SPEC-011-AC-01:** Given Maya (Organiser) is signed in and under the export rate limit, when she opens FEAT-18.SPEC-001, then she has full access to request and download an export.

**FEAT-18.SPEC-011-AC-02:** Given Sam (Other Adult Member) attempts to navigate directly to FEAT-18.SPEC-001, when the request is made, then he sees "Only the household organiser can export household data." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-011-AC-03:** Given Sam attempts to navigate directly to FEAT-18.SPEC-002, when the request is made, then he sees "Only the household organiser can remove a member." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-011-AC-04:** Given Sam attempts to navigate directly to FEAT-18.SPEC-003, when the request is made, then he sees "Only the household organiser can delete the household." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-011-AC-05:** Given Sam opens FEAT-18.SPEC-004, when the screen loads, then no Household Data & Deletion section is rendered anywhere on the screen.

**FEAT-18.SPEC-011-AC-06:** Given Maya opens FEAT-18.SPEC-004, when the screen loads, then the Household Data & Deletion section with Export, Remove a member, and Delete household links is shown.

**FEAT-18.SPEC-011-AC-07:** Given Sam is on FEAT-18.SPEC-004, when he attempts to delete his own account, then no precondition blocks him and he proceeds to the confirmation step.

**FEAT-18.SPEC-011-AC-08:** Given Maya is still the organiser and attempts to delete her own account, when the check runs, then she sees "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with links to FEAT-09.SPEC-003 and FEAT-18.SPEC-003.

**FEAT-18.SPEC-011-AC-09:** Given Maya and Sam are both on FEAT-18.SPEC-005, when either submits a support description, then it is accepted, since Contact Support is available to any adult member.

**FEAT-18.SPEC-011-AC-10:** Given the older-kid limited login (Later) attempts to reach any FEAT-18 screen, when the screen loads, then "This isn't available for your login." is shown and no account or household data is exposed.

**FEAT-18.SPEC-011-AC-11:** Given the organiser role transfers to Sam while Maya still has FEAT-18.SPEC-004 open, when Maya (now Other Adult Member) attempts to tap Delete Household, then it is rejected at FEAT-18.SPEC-003's own entry check with "Only the household organiser can delete the household."

**FEAT-18.SPEC-011-AC-12:** Given an unauthenticated visitor attempts to open any FEAT-18 screen directly, when the request is made, then they are redirected to the sign-in screen with no household data rendered.

**FEAT-18.SPEC-011-AC-13:** Given Riley (Operator) is signed in only through the separate read-only support view (FEAT-22), when any FEAT-18 screen is requested under his identity, then no such entry point exists and no household or account data is returned.

**FEAT-18.SPEC-011-AC-14:** Given a young kid profile has no login, when any request is made under that profile's identity against any action in this feature, then no action succeeds, since no valid session can exist for it.

**FEAT-18.SPEC-011-AC-15:** Given Sam attempts to bypass the hidden organiser-only controls via a direct URL to FEAT-18.SPEC-002, then the destination screen itself blocks the action with "Only the household organiser can remove a member.", regardless of the navigation path used.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- covered by FEAT-18.SPEC-010) | 0 |
| Cross-Field Rules | 0 (N/A) | 0 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 0 (N/A) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Integration Spec: Transactional Email (Account & Data)

## Overview

**Name:** Transactional Email (Account & Data)
**ID:** FEAT-18.SPEC-012
**Type:** Integration
**Purpose:** Product boundary to the transactional email capability used to deliver export-ready, deletion-completed, and support-acknowledgement email.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Sending the export-ready email (FEAT-18.SPEC-013's email variant)
- Sending the household-deletion-completed email (FEAT-18.SPEC-014's email variant)
- Sending the support-request-acknowledgement email (FEAT-18.SPEC-015's email variant)
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send
- Disclosure of what account, household-summary, and support-message data is shared with the capability to deliver these emails

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Any other transactional email this product sends (account/recovery, safety reports, plan-ready fallback, billing confirmations) -- each is owned by the feature whose Communications require it (FEAT-01.SPEC-017, FEAT-02.SPEC-010, FEAT-07.SPEC-006, FEAT-14.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table); this spec covers only the account-and-data emails named in this feature's own Communications
- Deciding the exact wording of each email's subject and body -- owned by FEAT-18.SPEC-013, FEAT-18.SPEC-014, and FEAT-18.SPEC-015, whose Content Definition sections this spec delivers verbatim
- The in-app channel for any of the three notifications -- owned directly by FEAT-18.SPEC-013, FEAT-18.SPEC-014, and FEAT-18.SPEC-015; this spec covers only the email channel
- Attaching or delivering the export file itself through email -- the export is downloaded in-app from FEAT-18.SPEC-001; this spec's export-ready email links back to that screen rather than carrying the file as an attachment

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional email capability -- Required for ... data-export and deletion confirmations, ... and support acknowledgements ..." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec, FEAT-18.SPEC-012, covers the export-ready, deletion-completed, and support-acknowledgement portion)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya receives an email once her requested export is ready to download | Export household data | FEAT-18.SPEC-013 (Export Ready Notification) |
| Maya receives an email once her household's deletion has fully completed | Delete the household | FEAT-18.SPEC-014 (Household Deletion Completed Notification) |
| The member who contacted support receives an email acknowledging their message | Contact support | FEAT-18.SPEC-015 (Support Request Acknowledgement) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|-------------------|-----------|------------|
| Maya's email address | Member Profile -- sign_in (email component, organiser only) | FEAT-18.SPEC-013 or FEAT-18.SPEC-014 is triggered | The capability needs a destination address to deliver the email |
| Maya's first name / display name | Member Profile -- display_name (organiser) | Same as above | Personalizes the greeting, per the exact content templates in FEAT-18.SPEC-013 and FEAT-18.SPEC-014 |
| Export readiness content (ready date) | Derived -- the completed export's ready date, no export content itself | FEAT-18.SPEC-013 is triggered | The email's exact content, per FEAT-18.SPEC-013's Content Definition |
| Household deletion completion content (completion date) | Derived -- the completed deletion's completion date, no deleted household content itself | FEAT-18.SPEC-014 is triggered | The email's exact content, per FEAT-18.SPEC-014's Content Definition |
| Reporting member's email address and first name | Member Profile -- sign_in (email component), display_name (the member who submitted the support description) | FEAT-18.SPEC-015 is triggered | The capability needs a destination address and a personalization value for the acknowledgement |

No export file contents, no Dietary Rule data (including any child's allergy information), no other household member's data, no payment details, and no support-description text beyond what FEAT-18.SPEC-015's own content template states ever leaves the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|------------------|-------------------------------|
| Delivery outcome (delivered / bounced / failed) | The capability reports the send result | No entity field is updated by a successful delivery; a bounced or failed send updates an internal delivery-status flag on the pending email attempt (not a Household, Member Profile, or Support Request field), used only to decide whether to retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|----------------|-------------------|--------------------|
| Export-ready email delivered | The capability confirms the email reached the recipient's inbox | None -- delivery confirmation is not surfaced as a user-visible change | None -- FEAT-18.SPEC-001's in-app Ready state already reflects readiness regardless of email delivery status | FEAT-18.SPEC-013 |
| Deletion-completed email delivered | The capability confirms the email reached the recipient's inbox | None | None -- deletion is already complete regardless of email delivery status | FEAT-18.SPEC-014 |
| Support-acknowledgement email delivered | The capability confirms the email reached the recipient's inbox | None | None -- the reporting member's screen-level confirmation (FEAT-18.SPEC-005) already showed a same-screen success message | FEAT-18.SPEC-015 |
| Send failed | The capability reports it could not deliver any of the three emails (bounced, rejected, or a hard failure) | The pending email attempt's internal delivery-status flag is set to failed | None immediately -- see Degradation Behavior; each triggering notification's in-app channel is unaffected | FEAT-18.SPEC-013, FEAT-18.SPEC-014, FEAT-18.SPEC-015 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-------------------|--------------------|------------------------|
| FEAT-18.SPEC-001 (Export Household Data) | No user-visible effect -- the in-app Ready state appears regardless of email send speed | No user-visible effect -- the in-app Ready state is unaffected; the export-ready email is queued to send once the capability recovers, up to its own retry limit | No user-visible effect on Maya's session -- a rejected send does not block or alter the in-app Ready state, since email is a durable-record channel here, not a required confirmation gate |
| FEAT-18.SPEC-003 (Delete Household) | No user-visible effect -- the deletion-in-progress and (later) completion state is driven by FEAT-18.SPEC-008's own processing, not by email | No user-visible effect on the in-app channel; the completion email is queued to send once the capability recovers | No user-visible effect on Maya's session -- the in-app completion state remains the primary, always-delivered record |
| FEAT-18.SPEC-005 (Contact Support) | No user-visible effect -- the same-screen "we've got your message" confirmation appears regardless of email send speed | No user-visible effect on the in-app confirmation; the acknowledgement email is queued to send once the capability recovers | No user-visible effect on the reporting member's session -- the same-screen confirmation already told them the message was received |

## Consent and Disclosure

- **Export-ready and deletion-completed email disclosure** -- Sending Maya a confirmation email at her own registered address once an export is ready or a deletion completes is standard product behavior for an action she just took; no separate consent prompt interrupts the request or confirmation flow, since she already provided that email address for exactly this kind of account communication (per FEAT-01.SPEC-017's account-level disclosure).
- **Support-acknowledgement disclosure** -- FEAT-18.SPEC-005's same-screen confirmation, "Thanks -- we've got your message and will follow up by email.", is itself the disclosure that an email will follow to the reporting member's registered address.
- **What is never shared** -- Export file contents, Dietary Rule data (including any child's allergy information), any other household member's data, and payment details never leave the product through this integration; only the recipient's own email, first name, and the specific content named in Data Exchanged are included.

## Edge Cases

- **An export-ready or deletion-completed email send event arrives for a household deleted between the trigger and the send** -- For the export-ready email, the event is discarded silently if the household no longer exists (a household deletion in the interim would have cancelled the in-flight export per FEAT-18.SPEC-006's Edge Cases); the deletion-completed email is unaffected by this scenario, since it fires only once deletion has already fully completed.
- **The same "send failed" event is delivered twice for one attempt** -- The second delivery changes nothing: the delivery-status flag is already failed, and no duplicate retry beyond the standard policy is triggered.
- **Confirmation and failure events arrive out of order (failure reported, then a late "delivered" event for the same attempt)** -- The most recent event by its own timestamp governs the delivery-status flag; a late "delivered" event arriving after a "failed" event corrects the flag back to delivered, since it reflects a true, if delayed, outcome.
- **Capability goes down mid-send for a support acknowledgement** -- If the send was not confirmed initiated, it is treated as not yet sent and is queued for retry once the capability recovers; the reporting member's in-app confirmation is unaffected either way.
- **Maya requests a second export while the first export's ready email is still queued (capability was down)** -- Each export's ready email is tracked as its own independent send attempt; the second export's own readiness triggers its own email once it, too, is ready, without interfering with the first's queued retry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-013 (Export Ready Notification) | Triggered by (inbound) | The export-ready email channel is sent through this integration |
| FEAT-18.SPEC-013 (Export Ready Notification) | Affects (outbound) | Degradation behavior surfaces here (as no user-visible effect, by design) |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Triggered by (inbound) | The deletion-completed email channel is sent through this integration |
| FEAT-18.SPEC-014 (Household Deletion Completed Notification) | Affects (outbound) | Degradation behavior surfaces here |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | Triggered by (inbound) | The acknowledgement email channel is sent through this integration |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | Affects (outbound) | Degradation behavior surfaces here |
| FEAT-01.SPEC-017 (Transactional Email Integration -- Account Recovery) | References (inbound) | Shares the account-level email disclosure this spec relies on |

## Analytics and Success Signals

- **account_data_email_sent** (notification: export_ready / deletion_completed / support_ack; delivery outcome: delivered / failed) -- N/A -- no Stage 2 success metric measures this feature's email delivery directly; retained per product-features.md's Signals field (data_export_completed, household_deleted, support_contacted) as the operational record of these confirmations actually reaching households
- **account_data_email_degradation_shown** (condition: slow / down / rejected; notification: export_ready / deletion_completed / support_ack) -- N/A -- no Stage 2 success metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable

## Acceptance Criteria

**FEAT-18.SPEC-012-AC-01:** Given Maya's export becomes ready, when FEAT-18.SPEC-013 fires, then this integration sends the export-ready email to her registered address.

**FEAT-18.SPEC-012-AC-02:** Given Maya's household deletion fully completes, when FEAT-18.SPEC-014 fires, then this integration sends the deletion-completed email to her registered address.

**FEAT-18.SPEC-012-AC-03:** Given Sam submits a support description, when FEAT-18.SPEC-015 fires, then this integration sends the acknowledgement email to Sam's registered address.

**FEAT-18.SPEC-012-AC-04:** Given the transactional email capability is slow, when an export-ready email is triggered, then Maya's in-app Ready state is unaffected by the delay.

**FEAT-18.SPEC-012-AC-05:** Given the transactional email capability is down, when the deletion-completed email is triggered, then Maya's in-app completion state is unaffected and the email is queued to send once the capability recovers.

**FEAT-18.SPEC-012-AC-06:** Given the transactional email capability rejects a support-acknowledgement send, when this occurs, then the reporting member's same-screen confirmation on FEAT-18.SPEC-005 remains unaffected.

**FEAT-18.SPEC-012-AC-07:** Given an export-ready email send event arrives for a household deleted since the trigger, when this integration processes it, then no email is sent and no user feedback fires.

**FEAT-18.SPEC-012-AC-08:** Given a "send failed" event for any of this spec's three emails is delivered twice, when the second delivery arrives, then the delivery-status flag remains failed and no duplicate retry beyond the standard policy occurs.

**FEAT-18.SPEC-012-AC-09:** Given a "delivered" event arrives after an earlier "failed" event for the same email attempt, when it is processed, then the delivery-status flag corrects to delivered.

**FEAT-18.SPEC-012-AC-10:** Given the capability goes down mid-send for a support acknowledgement that was not confirmed initiated, when this occurs, then the send is queued for retry and the reporting member's in-app confirmation is unaffected.

**FEAT-18.SPEC-012-AC-11:** Given Maya requests a second export while the first export's ready email is still queued, when the second export becomes ready, then its own email is sent independently without disrupting the first's queued retry.

**FEAT-18.SPEC-012-AC-12:** Given Maya has never seen a data-sharing prompt interrupt an export or deletion action, when she receives an export-ready or deletion-completed email, then she recognizes it as standard account communication to the address she already provided, per FEAT-01.SPEC-017's account-level disclosure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 9 (3 screens x 3 conditions) | 9 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Export Ready Notification

## Overview

**Name:** Export Ready Notification
**ID:** FEAT-18.SPEC-013
**Type:** Notification
**Purpose:** Tells the organiser once a requested export is ready to download, since compilation is asynchronous and she may not be watching the screen while it runs.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- The confirmation delivered when an export completes, on both its channels (in-app and email)
- Preference, retry, and expiry behavior for this notification

**Non-Goals:**
- Deciding when an export is actually ready -- owned by FEAT-18.SPEC-006 (Export Generation Processing); this spec begins where that automation's success outcome fires
- The email channel's delivery mechanics -- owned by FEAT-18.SPEC-012 (Transactional Email, Account & Data), which this notification's email variant is sent through
- Reminding the organiser again if she never downloads the ready export -- excluded per scope-boundaries.md SC-14's spirit and this feature's general tone: the export remains available on FEAT-18.SPEC-001 indefinitely as a previous export, so no repeated nagging is needed

## Channels

| Channel | Used When | Rationale |
|---------|-----------|--------------|
| In-app | Always when the export becomes ready | Maya may have the app open or return to it soon after requesting the export; the in-app channel surfaces readiness the next time she is in the product |
| Email | Always when the export becomes ready | Export generation can take a noticeable time; Maya's day includes long stretches away from the product, and email catches the readiness moment even if she has closed the app |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Export ready | FEAT-18.SPEC-006 (Export Generation Processing) | Fires when compilation completes successfully | Household reference, organiser's Member Profile reference, export ready date |

## Audience and Preferences

**Recipients:** Maya -- the organiser, per the Access Matrix's Account & Data column (Full for Maya, no access for any other role). Only the household member who requested the export receives this notification, since export is an organiser-only action.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|------------------------|
| N/A -- this notification has no on/off control | -- | Always on | -- |

No preference exists for this notification: it confirms an action Maya herself just took, not a proactive product-initiated message, so the product does not offer an opt-out (consistent with the account/recovery and billing confirmation emails' own no-preference treatment elsewhere in the product).

**Quiet Hours:** N/A -- this notification fires only once, in direct response to Maya's own recent request, at whatever time compilation happens to finish; the product defines no quiet-hours window for a confirmation of an action the recipient initiated herself.

## Content Definition

**In-app:**
- **Title:** Your export is ready
- **Body:** Your household data export finished compiling on {ready_date}. Download it from Account & Data.
- **CTA:** Download export -- deep-links to FEAT-18.SPEC-001 (Export Household Data)

**Email:**
- **Subject:** Your Plateful data export is ready
- **Body:**
  Hi {organiser_first_name},

  The household data export you requested is ready to download.

  Open Plateful and go to Account & Data to download your export.
- **CTA (button):** Download your export -- deep-links to FEAT-18.SPEC-001 (Export Household Data)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|-----------------------------|-------------------|----------------------------|
| {ready_date} | Derived -- the completed export's ready date | September 27, 2026 | Never empty -- the export ready date is always set the moment compilation completes, before this notification fires |
| {organiser_first_name} | Member Profile -- display_name (organiser) | Maya | Greeting renders as "Hi," |

## Delivery Rules

**Batching:** No batching applies -- each export request produces its own independent notification, since only one export can be compiling per household at a time (FEAT-18.SPEC-010's rate limit and FEAT-18.SPEC-001's disabled-while-compiling state).
**Deduplication:** At most one export-ready notification per completed export. A retried compilation (FEAT-18.SPEC-006's automatic retry) never fires this notification until the export truly succeeds; retries themselves produce no notification.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-18.SPEC-012's Degradation Behavior. After the final failure, the in-app notification stands as the delivery of record and no error is shown to Maya -- a confirmation of her own successful action must never surface an alarming failure message of its own. In-app delivery has no retry: it is delivered when Maya next opens the product.
**Expiry:** This notification does not expire in the usual sense -- the export it confirms remains available indefinitely as a previous export on FEAT-18.SPEC-001, so there is no "too late to matter" cutoff; an email delayed by retries is still useful whenever it arrives, since the export itself has not gone anywhere.

## Edge Cases

- **Household deleted between export completion and notification delivery** -- Cannot occur: household deletion supersedes and cancels any in-flight export (FEAT-18.SPEC-006's Edge Cases), so an export never completes for a household that has since been deleted.
- **Maya is already viewing FEAT-18.SPEC-001 when the export becomes ready** -- The in-app notification still fires (delivered to the product's standard notification surface), while the screen itself also transitions to its Ready state simultaneously; the two are complementary, not duplicative, since one is a persistent notification and the other is the screen's own live state.
- **The email is still queued (capability was down) when Maya downloads the export in-app first** -- The queued email is still delivered once the capability recovers; downloading in-app does not cancel or suppress the email, since the email's purpose (reaching her even if she is away from the product) is independent of whether she has already acted on the in-app notification.
- **A second export completes while the first export's notification has not yet been opened** -- Each export's notification is independent and both remain available; there is no batching or replacement, since each represents a genuinely separate completed export.
- **Maya has no registered email address (a defensive, unusual case)** -- Cannot occur in practice: every adult member's sign_in requires an email at account creation (FEAT-01); if this were ever true, the email channel would be skipped silently and the in-app notification alone would stand as the delivery of record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-006 (Export Generation Processing) | Triggered by (inbound) | The export-ready outcome fires this notification |
| FEAT-18.SPEC-012 (Transactional Email, Account & Data) | Triggers (outbound) | The email channel is sent through this integration |
| FEAT-18.SPEC-001 (Export Household Data) | Navigation (outbound) | Both channels' CTA deep-links here |

## Analytics and Success Signals

- **export_ready_notification_delivered** (channel: in_app / email) -- N/A -- no Stage 2 success metric measures export-notification delivery directly; retained per product-features.md's Signals field (data_export_completed) as the operational confirmation this milestone reaches Maya
- **export_ready_notification_cta_tapped** (channel) -- N/A -- no Stage 2 success metric measures export download follow-through from this notification; retained to observe how often the notification actually drives a download

## Acceptance Criteria

**FEAT-18.SPEC-013-AC-01:** Given Maya's export completes, when FEAT-18.SPEC-006 fires this notification, then she receives an in-app notification titled "Your export is ready" and an email with the subject "Your Plateful data export is ready".

**FEAT-18.SPEC-013-AC-02:** Given Maya receives the in-app notification, when she taps "Download export", then she lands on FEAT-18.SPEC-001 with her export ready to download.

**FEAT-18.SPEC-013-AC-03:** Given Maya receives the export-ready email, when she taps "Download your export", then she lands on FEAT-18.SPEC-001.

**FEAT-18.SPEC-013-AC-04:** Given the export-ready email fails to send on its first attempt, when a retry remains within the 6-hour window, then it is retried automatically without any error shown to Maya.

**FEAT-18.SPEC-013-AC-05:** Given the export-ready email's retries are exhausted, when this occurs, then no failure message is shown to Maya and the in-app notification stands as the delivery of record.

**FEAT-18.SPEC-013-AC-06:** Given Maya has no preference control for this notification, when an export becomes ready, then both channels are always used, since no opt-out exists.

**FEAT-18.SPEC-013-AC-07:** Given Maya is actively viewing FEAT-18.SPEC-001 when her export becomes ready, when the export completes, then the screen transitions to its Ready state and the in-app notification is also delivered.

**FEAT-18.SPEC-013-AC-08:** Given Maya downloads her export in-app before the queued email arrives, when the email capability later recovers, then the queued email is still delivered.

**FEAT-18.SPEC-013-AC-09:** Given Maya requests and completes two separate exports in succession, when both complete, then she receives two independent notifications, one per export.

**FEAT-18.SPEC-013-AC-10:** Given FEAT-18.SPEC-006 retries a failed compilation before eventually succeeding, when the retries occur, then no export-ready notification fires until the export truly succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Household Deletion Completed Notification

## Overview

**Name:** Household Deletion Completed Notification
**ID:** FEAT-18.SPEC-014
**Type:** Notification
**Purpose:** Tells the organiser once household deletion has fully completed, since the cascade can take up to 30 days and she is signed out well before it finishes.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- The confirmation delivered when household deletion processing fully completes, on its one channel (email, since the organiser is already signed out by then)
- Retry and expiry behavior for this notification

**Non-Goals:**
- Deciding when deletion is fully complete -- owned by FEAT-18.SPEC-008 (Household Deletion Processing); this spec begins where that automation's completed outcome fires
- The email channel's delivery mechanics -- owned by FEAT-18.SPEC-012 (Transactional Email, Account & Data), which this notification's email is sent through
- The immediate, in-app "Your household is being deleted..." message shown at confirmation time -- owned directly by FEAT-18.SPEC-003; this notification is the separate, later confirmation that the full cascade has finished

## Channels

| Channel | Used When | Rationale |
|---------|-----------|--------------|
| Email | Always, once deletion fully completes | Maya is signed out of the household well before the cascade finishes (deletion can take up to 30 days), so no in-app surface exists to reach her; email is the only channel that can still reach her at her own address after the household -- and her session in it -- no longer exists |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household deletion completed | FEAT-18.SPEC-008 (Household Deletion Processing) | Fires when the full cascade finishes and the Household record is permanently removed | Organiser's email address and first name (captured at the start of deletion processing, before the Member Profile record itself is removed), deletion completion date |

## Audience and Preferences

**Recipients:** Maya -- the organiser at the time deletion was confirmed. She is the sole recipient, since she is the one who confirmed the deletion and the household no longer has any other addressable member by the time this notification fires.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|------------------------|
| N/A -- this notification has no on/off control | -- | Always on | -- |

No preference exists for this notification: it confirms an irreversible action Maya herself explicitly confirmed, and it is the sole surviving record that the deletion she asked for actually finished -- the product does not offer an opt-out for it.

**Quiet Hours:** N/A -- this notification fires once, whenever the multi-day cascade happens to finish; the product defines no quiet-hours window for a confirmation of an irreversible action the recipient explicitly confirmed, since delaying it would only prolong her uncertainty about whether the deletion actually completed.

## Content Definition

**Email:**
- **Subject:** Your Plateful household has been deleted
- **Body:**
  Hi {organiser_first_name},

  The household you asked us to delete has now been fully and permanently removed, along with every member profile, plan, list, rule, and rating it contained.

  This action cannot be undone, and no data from this household is retained in any form usable for another purpose.

  If you didn't request this, contact support at {support_contact_reference}.
- **CTA:** N/A -- no in-product destination exists to link to, since the household and every account tied to it have been removed; the email is the final, standalone record of completion.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|-----------------------------|-------------------|----------------------------|
| {organiser_first_name} | Member Profile -- display_name (organiser, captured before removal) | Maya | Greeting renders as "Hi," |
| {support_contact_reference} | Derived -- the product's general support contact reference (not the in-product Contact Support screen, which requires a signed-in session Maya no longer has) | support@plateful.example | Never empty -- a general support contact reference is a fixed part of every account-related email's footer, independent of any specific household |

## Delivery Rules

**Batching:** No batching applies -- each household deletion produces exactly one completion notification, since only one deletion can be in progress per household at a time (FEAT-18.SPEC-008's Business Rules).
**Deduplication:** At most one deletion-completed notification per household deletion. The cascade completes exactly once; there is no retry-of-the-whole-cascade path that could re-trigger this notification.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-18.SPEC-012's Degradation Behavior. After the final failure, no further channel exists to deliver this notification -- there is no in-app fallback, since the household and Maya's session in it no longer exist. The failure is logged internally so the product's own record-keeping reflects that the confirmation could not be confirmed as received, but no further user-facing action follows.
**Expiry:** This notification does not expire in the usual sense; because it has no fallback channel, delivery continues to be attempted for the full retry window (up to 6 hours) regardless of how long the underlying deletion took, since a late-but-eventually-delivered confirmation is still meaningfully better than none.

## Edge Cases

- **Maya's email address is captured before the Member Profile record is removed, but the send itself happens after removal** -- The email address and first name are captured once, at the start of deletion processing (per this spec's Trigger's Available Data), and carried through to send time independently of the Member Profile record's own lifecycle, so the notification can still be sent after the underlying record no longer exists.
- **The email capability is down for the entire 6-hour retry window** -- The notification is never delivered; this is logged internally, and no further fallback exists, since the household and every account tied to it are already gone.
- **Maya attempts to sign back in immediately after confirming deletion, before the cascade finishes** -- She cannot: her Member Profile was set to a removed state as part of the deletion's early steps (FEAT-18.SPEC-008), so no valid session exists for her to sign back into during or after the cascade; the completion email remains her only route to confirmation.
- **Maya never opens the completion email** -- No further reminder is sent; this notification, per its own Delivery Rules, fires exactly once and does not repeat, since a household that no longer exists has nothing further to report.
- **A support inquiry (via the general support contact reference) arrives asking whether a deletion completed, while the cascade is still within its 30-day window** -- This is outside this notification's own scope; it is answered through the product's general support process (outside this feature's in-product Contact Support screen, since Maya's household no longer exists to submit through it), not by re-sending this notification early.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-008 (Household Deletion Processing) | Triggered by (inbound) | The deletion-completed outcome fires this notification |
| FEAT-18.SPEC-012 (Transactional Email, Account & Data) | Triggers (outbound) | The email channel is sent through this integration |
| FEAT-18.SPEC-003 (Delete Household) | References (inbound) | The immediate in-app confirmation this notification follows, at completion rather than confirmation time |

## Analytics and Success Signals

- **household_deletion_completed_notification_delivered** (delivery outcome: delivered / failed) -- N/A -- no Stage 2 success metric measures this notification's delivery directly; retained per product-features.md's Signals field (household_deleted) as the sole confirmation record for a deletion, given it has no in-app fallback
- **household_deletion_completed_notification_failed** (retries_attempted) -- N/A -- no Stage 2 success metric measures this notification's failure rate; retained since a failure here means the household's final confirmation was never received at all

## Acceptance Criteria

**FEAT-18.SPEC-014-AC-01:** Given Maya's household deletion fully completes, when FEAT-18.SPEC-008 fires this notification, then she receives an email with the subject "Your Plateful household has been deleted".

**FEAT-18.SPEC-014-AC-02:** Given Maya receives the deletion-completed email, when she reads it, then it states the deletion cannot be undone and that no data is retained for another purpose.

**FEAT-18.SPEC-014-AC-03:** Given the deletion-completed email fails to send on its first attempt, when a retry remains within the 6-hour window, then it is retried automatically.

**FEAT-18.SPEC-014-AC-04:** Given the deletion-completed email's retries are exhausted, when this occurs, then no further delivery attempt or fallback channel is used, and the failure is logged internally.

**FEAT-18.SPEC-014-AC-05:** Given Maya has no preference control for this notification, when her household deletion completes, then the email is always sent, since no opt-out exists.

**FEAT-18.SPEC-014-AC-06:** Given Maya's Member Profile was already removed as part of the early steps of deletion processing, when this notification is sent later, then it still reaches her registered email address, since that address was captured at the start of processing.

**FEAT-18.SPEC-014-AC-07:** Given Maya attempts to sign back in after confirming deletion but before the cascade finishes, when she tries, then no valid session exists for her, and the completion email remains her only route to confirmation.

**FEAT-18.SPEC-014-AC-08:** Given Maya never opens the completion email, when time passes, then no further reminder or repeat of this notification is sent.

**FEAT-18.SPEC-014-AC-09:** Given a household's deletion completes, when this notification fires, then no in-app CTA is offered, since no in-product destination exists for a deleted household.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Support Request Acknowledgement

## Overview

**Name:** Support Request Acknowledgement
**ID:** FEAT-18.SPEC-015
**Type:** Notification
**Purpose:** Sends the member who contacted support a transactional-email acknowledgement that their message was received.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- The acknowledgement delivered when a general support description is submitted, on its one channel (email)
- Retry and expiry behavior for this notification

**Non-Goals:**
- Any further reply, status update, or resolution message -- excluded per scope-boundaries.md SC-14: this feature's Contact Support screen sends a one-way description and receives a one-time acknowledgement; any further exchange happens outside the product, not as an additional in-product notification
- Deciding when a support description is submitted -- owned by FEAT-18.SPEC-005 (Contact Support), whose successful submission triggers this notification
- The email channel's delivery mechanics -- owned by FEAT-18.SPEC-012 (Transactional Email, Account & Data), which this notification's email is sent through
- The safety-concern acknowledgement -- a distinct communication for a distinct Support Request kind, owned by FEAT-02 (Dietary Rules & Allergy Safety Engine)

## Channels

| Channel | Used When | Rationale |
|---------|-----------|--------------|
| Email | Always, once a general support description is submitted | The submitting member already saw a same-screen confirmation on FEAT-18.SPEC-005; the email is the durable record of the acknowledgement and the channel through which any eventual operator follow-up will reach them, per scope-boundaries.md SC-14 |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Support Request created | FEAT-18.SPEC-005 (Contact Support) | Fires when a general-support-contact Support Request is created successfully | Reporting member's email address and first name, submitted description text, submission date |

## Audience and Preferences

**Recipients:** The reporting adult member (Maya or Sam) -- whoever submitted the support description on FEAT-18.SPEC-005. The acknowledgement is delivered only to the member who raised the specific request, since each request is a private, one-way communication with the operator.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|------------------------|
| N/A -- this notification has no on/off control | -- | Always on | -- |

No preference exists for this notification: it confirms an action the member themselves just took, and withholding the acknowledgement would leave them uncertain whether their message was received at all.

**Quiet Hours:** N/A -- this notification fires immediately after submission, whenever that happens to occur; the product defines no quiet-hours window for a confirmation of an action the recipient initiated themselves, since holding it would only delay the reassurance the acknowledgement exists to give.

## Content Definition

**Email:**
- **Subject:** We've got your message
- **Body:**
  Hi {member_first_name},

  Thanks for reaching out. Here's what you sent us on {submission_date}:

  "{submitted_description}"

  We'll follow up by email if we need more information or once we've looked into it.
- **CTA:** N/A -- no in-product destination exists to link to, since this feature defines no status-tracking screen for a submitted request (scope-boundaries.md SC-14); any further exchange happens by email outside the product.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|-----------------------------|-------------------|----------------------------|
| {member_first_name} | Member Profile -- display_name (the reporting member) | Sam | Greeting renders as "Hi," |
| {submission_date} | Support Request -- derived from its creation date | September 27, 2026 | Never empty -- a Support Request always carries a creation date the moment it is created |
| {submitted_description} | Support Request -- note | The grocery list won't load on my phone. | Never empty -- FEAT-18.SPEC-010's required-field rule prevents an empty description from ever being submitted |

## Delivery Rules

**Batching:** No batching applies -- each submitted support description creates its own Support Request and its own independent acknowledgement, even if the same member submits more than one in quick succession (per FEAT-18.SPEC-005's Edge Cases).
**Deduplication:** At most one acknowledgement per Support Request. A Support Request is created exactly once per submission; there is no re-processing path that could re-trigger this notification for the same request.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-18.SPEC-012's Degradation Behavior. After the final failure, the reporting member's same-screen confirmation on FEAT-18.SPEC-005 stands as the sole delivery-of-record indicator that their message was received in-product, and no further user-facing error is shown for the email specifically.
**Expiry:** This notification does not expire in the usual sense -- an acknowledgement delivered late is still meaningfully useful to the reporting member, since it confirms receipt of a message that remains open regardless of when the email arrives; delivery continues to be attempted for the full retry window.

## Edge Cases

- **The reporting member is removed (FEAT-18.SPEC-007) or deletes their own account (FEAT-18.SPEC-009) between submission and this notification's delivery** -- The email address and first name are captured at submission time (per this spec's Trigger's Available Data) and carried through to send time independently of the Member Profile record's later lifecycle, so the acknowledgement is still delivered to the address that was valid at submission.
- **The household is deleted between submission and this notification's delivery** -- The Support Request itself is retained through household deletion's own record-keeping only up to the point the cascade removes it (FEAT-18.SPEC-008); if the acknowledgement has not yet been sent when the household record is fully removed, it is still delivered using the captured address and content, since the acknowledgement concerns the member's own submission, not the household's continued existence.
- **The email capability is down for the entire 6-hour retry window** -- The acknowledgement is never delivered by email; the reporting member's same-screen confirmation from FEAT-18.SPEC-005 remains the only confirmation they receive, and the failure is logged internally.
- **The submitted description contains characters that render unusually in an email body (e.g., emoji or unusual punctuation)** -- The description is included verbatim as submitted; no sanitization beyond what the platform's standard email-rendering handles is applied by this notification.
- **Two adults in the same household each submit a separate support description around the same time** -- Each submission produces its own independent Support Request and its own acknowledgement addressed to its own submitting member; the two are never merged or cross-referenced.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-005 (Contact Support) | Triggered by (inbound) | Successful submission fires this notification |
| FEAT-18.SPEC-012 (Transactional Email, Account & Data) | Triggers (outbound) | The email channel is sent through this integration |
| FEAT-22 (Operator Read-Only Support Access) | References (inbound, cross-feature) | The Support Request this notification acknowledges is the same record that opens Riley's read-only support access |

## Analytics and Success Signals

- **support_acknowledgement_delivered** (delivery outcome: delivered / failed) -- N/A -- no Stage 2 success metric measures acknowledgement delivery directly; retained per product-features.md's Signals field (support_contacted) as the operational confirmation this trust-facing message actually reaches the reporting member
- **support_acknowledgement_failed** (retries_attempted) -- N/A -- no Stage 2 success metric measures acknowledgement failures; retained to observe whether the reporting member's only durable confirmation ever fails to arrive

## Acceptance Criteria

**FEAT-18.SPEC-015-AC-01:** Given Sam submits a general support description, when FEAT-18.SPEC-005 creates the Support Request, then he receives an email with the subject "We've got your message" quoting his submitted description.

**FEAT-18.SPEC-015-AC-02:** Given Maya submits a support description, when the acknowledgement email is sent, then it addresses her by her own first name and includes her submission date.

**FEAT-18.SPEC-015-AC-03:** Given the acknowledgement email fails to send on its first attempt, when a retry remains within the 6-hour window, then it is retried automatically.

**FEAT-18.SPEC-015-AC-04:** Given the acknowledgement email's retries are exhausted, when this occurs, then no further error is shown to the reporting member, and their same-screen confirmation on FEAT-18.SPEC-005 stands as the record of receipt.

**FEAT-18.SPEC-015-AC-05:** Given Sam has no preference control for this notification, when he submits a support description, then the acknowledgement email is always sent, since no opt-out exists.

**FEAT-18.SPEC-015-AC-06:** Given the reporting member is removed from the household after submitting but before this notification is sent, when it is sent, then it still reaches the email address captured at submission time.

**FEAT-18.SPEC-015-AC-07:** Given two adults in the same household each submit a separate support description around the same time, when both are processed, then each receives their own independent acknowledgement addressed to themselves.

**FEAT-18.SPEC-015-AC-08:** Given a submitted support description, when the acknowledgement email is generated, then no in-product CTA is offered, since this feature defines no in-product status-tracking screen for the request.

**FEAT-18.SPEC-015-AC-09:** Given the submitting member's household is deleted before the acknowledgement email is sent, when the send is attempted, then it still delivers using the address and content captured at submission time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
