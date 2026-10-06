---
document_type: feature-overview
feature_number: FEAT-21
feature_name: Family Calendar Sync
feature_slug: family-calendar-sync
priority_tier: Nice-to-Have
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 5
screen_count: 1
automation_count: 1
logic_rule_count: 2
integration_count: 1
notification_count: 0
---

# Feature Breakdown Brief: Family Calendar Sync

## Summary

**Feature:** Family Calendar Sync
**ID:** FEAT-21
**Description:** The week's dinners can appear on the household's existing family calendar, so meal plans show up alongside everything else the family has scheduled.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Platform
**Rationale:** The brief names this directly as "a nice-to-have for showing dinner on the family calendar, not v1" (BRIEF.md, Ecosystem & Integrations). Phased to Later exactly per the brief's own stated timing. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Sync the week's dinners — Household connects a calendar capability so each night's dinner appears as an entry
- Keep it current — A swapped meal updates the corresponding calendar entry automatically

This feature is entirely an additive, background-integration feature: it defines exactly one screen of its own (the connection control point) and otherwise operates as a system-to-system sync with no other in-app surface. Per the feature's own States field, it is additive only — a failed sync, a lost connection, or the capability never being connected at all leaves the in-app Weekly Plan (owned by FEAT-03 and FEAT-23) entirely unaffected.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-21.SPEC-001 | Calendar Connection Settings | Screen | Maya | Maya connects or disconnects the household's calendar capability and sees its current connection status |
| FEAT-21.SPEC-002 | Family Calendar Integration | Integration | Maya, Sam, Jordan (young kid), Jordan (older kid) | The connection contract with the external calendar capability — connect/disconnect, entry create/update, and the inbound sync-outcome and disconnect events, per the family-calendar capability category (ASMP-37) |
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | Automation | Maya, Sam, Jordan (young kid), Jordan (older kid) | Watches the Weekly Plan for dinners to sync and for swaps that change an already-synced night, and drives the Integration spec to create or update the matching calendar entry |
| FEAT-21.SPEC-004 | Calendar Connection & Sync Governance Rules | Logic/Rule | Maya, Sam, Jordan (young kid), Jordan (older kid) | Governs the one-connection-per-household limit, retry-on-failure behavior, the additive-only/non-blocking guarantee for the in-app plan, and what disconnecting does and does not affect |
| FEAT-21.SPEC-005 | Calendar Entry Content Derivation Rule | Logic/Rule | Maya, Sam, Jordan (young kid), Jordan (older kid) | Derives what a synced calendar entry contains (night, dish, timing) from Weekly Plan and Planned Meal data, and keeps one entry per planned night stable across swaps |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Sync the week's dinners | FEAT-21.SPEC-001, FEAT-21.SPEC-002, FEAT-21.SPEC-003, FEAT-21.SPEC-005 | Maya connects the calendar capability from Settings; the sync automation walks the week's Planned Meals and drives the Integration spec to create one entry per dinner, with content derived by SPEC-005 | Phase 2 (Explicit) |
| Keep it current | FEAT-21.SPEC-003, FEAT-21.SPEC-004, FEAT-21.SPEC-005 | A meal swap re-fires the sync automation, which updates the same night's existing calendar entry (per SPEC-005's stable per-night mapping) rather than creating a duplicate | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-21.SPEC-002 | Family Calendar Integration | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-37) names family-calendar capabilities as a category-level external dependency this feature relies on, and the dependency map's External Touchpoints row for it was left "pending — awaiting validated Brief for FEAT-21"; this Brief resolves that row with a standalone Integration spec covering connect, disconnect, entry create/update, and the inbound sync-outcome and disconnect events |
| FEAT-21.SPEC-004 | Calendar Connection & Sync Governance Rules | Phase 5 (Rule-Constraint Discovery — conditional logic, shared across specs) | The Validation & Limits field (one connection per household), the States field's Error and Offline-degraded behavior (auto-retry, additive-only), and the Primary Flows & Alternates field's disconnect behavior together form five-plus interacting rules shared by SPEC-001, SPEC-002, and SPEC-003 rather than duplicated in each |
| FEAT-21.SPEC-005 | Calendar Entry Content Derivation Rule | Phase 5 (Rule-Constraint Discovery — derivation) | The Data Notes field names a derived field ("calendar entry content, computed from Weekly Plan data") sourced from two other features' output (FEAT-03, FEAT-04); the derivation and the per-night entry-identity rule that makes "update, not duplicate" possible are non-trivial enough to cross the standalone-spec threshold rather than living inline in SPEC-003 |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity through the create/read/update/delete lifecycle. Its sole Connected Entity, Weekly Plan, is read-only for this feature (product-features.md Connected Entities: "Weekly Plan (read)") — its creation, approval, and updates are owned entirely by other features (FEAT-03, FEAT-23, FEAT-04). Per Phase 3's rule for read-only Connected Entities, Weekly Plan is carried below as a Referenced Entity rather than given a full CRUD matrix; Planned Meal is included alongside it because the Data Notes field traces synced calendar-entry content down to individual dinners, not just the week as a whole.

The household's calendar connection itself (whether a household is currently connected, and to what) is not a Connected Entity named in product-features.md, so it is not given a Domain Entity CRUD table here. Its lifecycle — establishing a connection, reading its status, and tearing it down — is instead specified as part of FEAT-21.SPEC-002's Integration contract and governed by FEAT-21.SPEC-004's rules, consistent with Phase 4's guidance that a capability's connect/disconnect contract belongs to the Integration spec that owns it.

**Referenced Entities (read-only for this feature):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-21.SPEC-003 | The sync automation reads the household's current (and up to one week ahead) Weekly Plan to know which nights have a dinner to sync |
| Planned Meal | FEAT-21.SPEC-003, FEAT-21.SPEC-005 | Reads each dinner's night, recipe/dish name, and swap history so the sync automation can create or update the matching calendar entry with SPEC-005's derived content |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya connects the calendar capability | Establish the connection (rejecting a second simultaneous connection per SPEC-004's one-connection-per-household limit), then run an initial sync of the current week's dinners | Standalone Integration | FEAT-21.SPEC-002 |
| Maya disconnects the calendar capability | Tear down the connection; already-created calendar entries are left as-is on the external calendar, and no further sync occurs | Standalone Integration | FEAT-21.SPEC-002 |
| A dinner is proposed, picked, or approved into the Weekly Plan for a connected household | Sync automation creates the corresponding calendar entry via the Integration spec | Standalone Automation | FEAT-21.SPEC-003 |
| A meal swap (FEAT-04) changes an already-synced night | Sync automation updates the existing calendar entry for that night rather than creating a duplicate | Standalone Automation | FEAT-21.SPEC-003 |
| A sync attempt fails (connectivity or provider error) | Automatically retry; the in-app Weekly Plan is entirely unaffected in the meantime | Standalone Logic/Rule | FEAT-21.SPEC-004 |
| The external calendar reports the connection was revoked or lost outside the product | Integration spec's inbound event marks the household disconnected; in-app plan is unaffected | Standalone Integration (inbound event) | FEAT-21.SPEC-002 |
| A household is deleted (FEAT-18 account deletion cascade) | The calendar connection, if any, is disconnected as part of the cascade | Cross-feature | FEAT-18 responsibility, inbound event handled by FEAT-21.SPEC-002 |
| Calendar entry content is computed for a dinner | Derive night, dish name, and timing from the Planned Meal/Weekly Plan; no other household data (allergies, cost, notes) is placed on the entry | Standalone Logic/Rule | FEAT-21.SPEC-005 |
| Maya opens Calendar Connection Settings while never connected | Show the not-connected empty state with a "Connect calendar" call to action | Inline in triggering screen | FEAT-21.SPEC-001 |

## Shared Context

**Shared Entities:**
- Weekly Plan — read only, by FEAT-21.SPEC-003, to know which nights in the current (and up to one week ahead) plan need a synced entry. No fields are created or updated by this feature.
- Planned Meal — read only, by FEAT-21.SPEC-003 and FEAT-21.SPEC-005, for the night, recipe/dish name, and swap history that become calendar-entry content.

**Shared UI Patterns:**
- N/A — this feature defines exactly one screen (FEAT-21.SPEC-001), so there is no pattern to keep consistent across multiple screens within this feature. The screen is reached only from FEAT-01's Household Settings Hub, per the dependency map's navigation connection.

**Shared Validation:**
- FEAT-21.SPEC-004 defines the one-connection-per-household limit, the retry-on-failure behavior, and the additive-only/non-blocking guarantee; FEAT-21.SPEC-001, FEAT-21.SPEC-002, and FEAT-21.SPEC-003 all reference it rather than restating the rules.
- FEAT-21.SPEC-005 defines calendar-entry content derivation and the stable per-night entry identity that makes updates (rather than duplicates) possible; FEAT-21.SPEC-003 references it rather than deriving content itself.

## Internal Dependency Map

```
FEAT-21.SPEC-001 (Calendar Connection Settings) -> [Maya taps "Connect calendar"] -> FEAT-21.SPEC-002 (Family Calendar Integration)
FEAT-21.SPEC-001 (Calendar Connection Settings) -> [Maya taps "Disconnect"] -> FEAT-21.SPEC-002 (Family Calendar Integration)
FEAT-21.SPEC-002 (Family Calendar Integration) -> [enforces limit & retry rules from] -> FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules)
FEAT-21.SPEC-002 (Family Calendar Integration) -> [connection established] -> FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync)
FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) -> [reads content from] -> FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule)
FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) -> [creates/updates entries through] -> FEAT-21.SPEC-002 (Family Calendar Integration)
FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) -> [governed by non-blocking & retry rules in] -> FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules)
FEAT-21.SPEC-002 (Family Calendar Integration) -> [inbound: connection revoked externally] -> FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules) -> [household shown disconnected] -> FEAT-21.SPEC-001 (Calendar Connection Settings)
```

**Default Entry:** FEAT-21.SPEC-001 (Calendar Connection Settings) -- the only screen this feature owns, reached from FEAT-01's Household Settings Hub via the "connect calendar" navigation link (dependency map, Later phase).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-21.SPEC-001 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Reached by tapping "connect calendar" in household settings | Maya opens Household Settings Hub and taps the calendar connection link |
| FEAT-21.SPEC-003 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Reads AI-generated and approved Planned Meals as sync source content | New week generated or approved for a connected household |
| FEAT-21.SPEC-003 | Inbound | FEAT-23 (Manual Weekly Planning) | Reads manually picked Planned Meals as sync source content | A dinner is picked or changed in a manually built week for a connected household |
| FEAT-21.SPEC-003 | Inbound | FEAT-04 (One-Tap Meal Swap) | A completed swap (FEAT-04.SPEC-004, Apply Meal Swap) is the sole trigger for updating an already-synced night's entry | Swap completes for a night that already has a synced calendar entry |
| FEAT-21.SPEC-002 | Inbound | FEAT-18 (Account & Data Management) | Household deletion cascades to disconnecting any active calendar connection | Household deletion completes |

## Non-Functional Notes

**Data volumes / growth:** At most one calendar entry per planned dinner per connected household, capped by the Weekly Plan's own limit of up to seven dinners plus leftover-lunch slots per week, planned at most one week ahead (Validation & Limits, Weekly Plan fields); this feature stores no growing dataset of its own beyond the single connection state per household (Validation & Limits: "one calendar connection per household at a time").

**Responsiveness:** States field: "Loading: N/A — sync happens automatically in the background once connected" — sync is not a user-waited-for interaction; the only user-facing wait is the connect action itself on FEAT-21.SPEC-001, which should complete or clearly show a retrying/error state rather than leaving Maya uncertain.

**Data sensitivity / privacy:** Calendar entries carry only the dish/night content that FEAT-21.SPEC-005 derives — household personal data about what the family eats (ASMP-14, ASMP-26), never sold or used for advertising. No allergy, cost, or member-identifying detail is placed on the entry; this keeps children's data (ASMP-26) out of the synced content even though the resulting entries are visible to Sam and both kid rows outside the product, on the calendar itself (Access field).

**Compliance flags:** N/A — this feature carries no health or financial data on the synced entries, and Riley (Operator, support) has no access to this feature at all (Access field), so no support-access exposure applies here.

## Non-Goals

- **Two-way sync from the external calendar back into the Weekly Plan** — Excluded per the feature's own Primary Flows & Alternates field, which describes only an outbound flow ("each night's planned dinner appears as a calendar entry") and per the States field's "this integration is additive only": nothing the household does on the external calendar changes the in-app plan.
- **Multiple simultaneous calendar connections per household** — Excluded per the Validation & Limits field: "One calendar connection per household at a time." A household must disconnect before connecting a different calendar capability.
- **Automatic cleanup of past calendar entries** — Intentional lifecycle decision: neither the feature entry nor the dependency map's Weekly Plan lifecycle (archived, not deleted, when a week ends) calls for removing already-created calendar entries; entries created for past weeks remain on the external calendar indefinitely, consistent with this feature's additive-only posture (States field).
- **In-app viewing of the synced calendar** — Excluded per the Data Notes field: "Displayed: N/A within the product beyond a connection status." The product shows only whether a calendar is connected (FEAT-21.SPEC-001); the calendar itself, and its entries, are viewed entirely outside the product on the household's own calendar.
- **Sam or either kid row connecting or disconnecting the calendar** — Excluded per the Access field, which reserves this action to Maya (Organiser) alone; Sam and both kid rows only ever see the resulting entries outside the product, and Riley (Operator, support) has no access to this feature at all.
