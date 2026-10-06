---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-33.SPEC-005
spec_name: Referral Data Access Restriction
spec_slug: referral-data-access-restriction
parent_feature: FEAT-33
parent_feature_name: Portal Referral Attribution
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Referral Data Access Restriction

## Overview

**Name:** Referral Data Access Restriction
**ID:** FEAT-33.SPEC-005
**Type:** Logic/Rule
**Purpose:** Enforces that no persona browses an individual Referral Attribution record inside the product -- attribution exists only as an aggregate figure for the Growth Through Referral metric, and the referring freelancer is never told who signed up from her portal.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution
**Governed Entity:** Referral Attribution

## Scope and Non-Goals

**In Scope:**
- Authorization rules for every action on the Referral Attribution record, for every role in the Access Matrix
- The explicit statement that the record is never queried for display by any screen, including Dana's otherwise broad, read-only support access (FEAT-31)
- The boundary between this restriction and the aggregate Growth Through Referral figure, which is a success-metrics.md reporting concern outside any in-product screen
- Confirming field-level coverage of the governed entity, even though no field carries its own validation rule (validation is owned by the creating automation, FEAT-33.SPEC-004)

**Non-Goals:**
- Validating or deriving the Referral Attribution record's field values -- owned by FEAT-33.SPEC-004 (Referral Attribution Recording), which creates the sole record and sets every field at that moment; this spec governs only who may subsequently access it, never how it is populated
- Whether the record appears inside a full account data export (FEAT-24) -- FEAT-24 owns the scope and content of what a freelancer's own data export contains; this spec restricts only in-product browsing of an individual record, per product-features.md's exact language, and does not extend to an out-of-product export file
- Defining the aggregate Growth Through Referral figure's calculation -- owned by success-metrics.md as a Stage 2 reporting concern; this spec only draws the boundary that no in-product screen surfaces that aggregate to any persona
- Deleting the record -- owned entirely by FEAT-24's account-deletion cascade (dependency map: "Deleted by FEAT-24 with the account"); no persona-initiated deletion path exists for this spec to restrict

## Governed Entity

**Entity:** Referral Attribution
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| referring_portal | derived (reference, or "unknown") | Reference to the referring freelancer's Freelancer Account, or "unknown" when no portal referred the sign-up or the reference could not be resolved |
| self_reported_source | text (or "unknown") | The new freelancer's optional free-text answer to "How did you hear about us?", or "unknown" when skipped |
| recorded_at | date (required) | Timestamp of the record's one-time creation |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-33.SPEC-004 (Referral Attribution Recording) | Portal Referral Attribution | At creation -- the record is written once and exposed to no subsequent read path from any screen or automation this spec governs |
| FEAT-31 (Operator Support Access, referenced) | Operator Support Access | Structural exclusion -- Dana's otherwise broad, read-only support session (View across most entities per the Access Matrix) explicitly excludes Referral Attribution; no support-session screen queries this entity |
| success-metrics.md (Growth Through Referral, referenced) | -- (Stage 2 reporting artifact, not an in-product spec) | The only "read" of this entity anywhere -- an aggregate figure computed and reported outside the product's own screens, never surfaced to any persona through a screen |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| referring_portal | No validation beyond data type -- set once at creation by FEAT-33.SPEC-004; this spec governs access to the field, not its population | Always | -- | -- | -- |
| self_reported_source | No validation beyond data type -- set once at creation by FEAT-33.SPEC-004; this spec governs access to the field, not its population | Always | -- | -- | -- |
| recorded_at | No validation beyond data type -- set automatically at creation and never re-checked, since the record is never edited | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Independent degradation | referring_portal, self_reported_source | Neither field's value (known or "unknown") depends on or constrains the other -- a record may hold any of the four combinations of known/unknown across the two fields | N/A -- this is a creation-time rule owned by FEAT-33.SPEC-004; this spec only confirms no access rule in this spec depends on either field's value |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a Referral Attribution record | -- (no persona) | Never via any persona-initiated action -- creation happens only through FEAT-33.SPEC-004's own automated processing at sign-up | N/A -- no screen or control anywhere in the product offers any persona a way to create this record directly |
| View an individual Referral Attribution record | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never, for all four roles, always | No screen, list, or detail view anywhere in the product exposes an individual record. Nadia specifically is never told who signed up from her portal -- there is no "who did I refer" screen in Settings, the Dashboard, or anywhere else. Dana's otherwise broad support-session read access (FEAT-31) does not extend to this entity; no support screen shows it |
| View the aggregate Growth Through Referral figure | -- (no persona) | Never, via any in-product screen -- the aggregate is a success-metrics.md reporting artifact consumed outside the product's own screens, not a capability exposed to any persona | No screen in the product -- including any Nadia sees on her Dashboard (FEAT-12) or Settings (FEAT-21) -- surfaces this aggregate to her or to any other role |
| Update a Referral Attribution record | -- (no persona) | Never -- the record is immutable from the moment of creation (dependency map: "Never updated") | N/A -- no edit control exists for any persona, since the record has no fields exposed to any screen in the first place |
| Delete a Referral Attribution record | -- (no persona) | Never, directly -- deletion happens only as part of FEAT-24's account-deletion cascade, not as a persona-initiated action within this feature | N/A -- no delete control exists for any persona; the only removal path is the whole-account cascade FEAT-24 owns |

## Defaults and Derivations

No defaults or derivations are governed by this spec. Field defaults and derivations belong entirely to FEAT-33.SPEC-004, which creates the sole record for each Freelancer Account and sets every field's value exactly once, at that moment. This spec governs only who may subsequently read or act on the entity, never how its values are populated.

## Business Rules

- XBR-32: attribution is used only in aggregate to measure the growth loop -- this spec is XBR-32's authority for the access-restriction half of that rule; FEAT-33.SPEC-004 is its authority for how the record is created.
- The dependency map's Referral Attribution lifecycle line states it is "Read by FEAT-33 (aggregate growth measurement only)" -- no other feature, screen, or automation in the product reads this entity for any other purpose, including features that otherwise have broad read access to a freelancer's data (FEAT-13's Activity & Audit Trail, FEAT-31's Operator Support Access).
- Dana's support session (FEAT-31) is otherwise read-only across most of a freelancer's data, per the Access Matrix's "View" entries for her row -- Referral Attribution is a deliberate, explicit exception to that breadth, not an oversight.
- This restriction applies from the moment of creation onward -- there is no transitional period, retroactive exposure, or admin override anywhere in the product definition that surfaces an individual record to any persona.

## Edge Cases

- **Dana opens a support session (FEAT-31) for a freelancer whose portal referred a new sign-up** -- The Referral Attribution record created by that sign-up is not shown anywhere in the support session, even though most of the freelancer's other data is visible to Dana in that same session.
- **Nadia looks for a "who referred this signup" or "my referrals" screen anywhere in Settings, the Dashboard, or Client & Project Management** -- No such screen exists anywhere in the product; there is no control to look for.
- **Someone attempts to reach a direct record identifier or URL for a specific Referral Attribution record** -- Not applicable: no such route exists in the product, since no screen ever renders, links to, or exposes an individual record's identifier.
- **The Growth Through Referral aggregate would, in principle, be derivable by cross-referencing Freelancer Account creation dates with any exposed referral data** -- Not a concern in practice, since no referral data of any kind (individual or partial) is exposed through any screen for any persona to cross-reference in the first place.
- **A future feature request asks for an in-product "who signed up from my portal" view** -- Out of scope for this spec and for the current product definition; product-features.md's Access field and this spec's Authorization Rules table would both need to change through a Brief update, which is outside any spec's authority to make unilaterally.

## Acceptance Criteria

**FEAT-33.SPEC-005-AC-01:** Given Nadia is signed in and looks anywhere in Settings, her Dashboard, or Client & Project Management for a way to see who signed up from her portal, when she searches those areas, then no such screen or control exists anywhere.

**FEAT-33.SPEC-005-AC-02:** Given Owen is signed in to his client portal, when he looks for any referral-related data about other freelancers or accounts, then no such capability is shown -- his access to Portal Referral relates only to viewing the mark itself (FEAT-33.SPEC-001), never to attribution data.

**FEAT-33.SPEC-005-AC-03:** Given Priya is signed in to her client portal, when she looks for referral attribution data, then the same result as Owen's applies -- no such capability exists for her role either.

**FEAT-33.SPEC-005-AC-04:** Given Dana opens a read-only support session (FEAT-31) for a freelancer's account that has an associated Referral Attribution record, when she reviews the account's data during that session, then the Referral Attribution record is not shown, even though her session otherwise reads most of that account's data.

**FEAT-33.SPEC-005-AC-05:** Given a Referral Attribution record exists, when any persona attempts to reach a direct view of it (by any means the product exposes), then no route or screen exists to do so.

**FEAT-33.SPEC-005-AC-06:** Given the Growth Through Referral metric is computed in aggregate, when any persona views any in-product screen (Dashboard, Settings, or otherwise), then that aggregate figure is not surfaced there -- it exists only as a success-metrics.md reporting artifact outside the product's own screens.

**FEAT-33.SPEC-005-AC-07:** Given a Referral Attribution record has been created, when any process attempts to create a second one for the same Freelancer Account through any persona-facing control, then no such control exists -- creation is exclusively FEAT-33.SPEC-004's own automated action.

**FEAT-33.SPEC-005-AC-08:** Given a Referral Attribution record exists, when any persona attempts to edit or delete it directly (outside FEAT-24's account-deletion cascade), then no edit or delete control exists anywhere in the product for any role.

**FEAT-33.SPEC-005-AC-09:** Given every field in the Referral Attribution entity, when this spec's Field Validation Rules are reviewed, then each field is explicitly addressed as "no validation beyond data type," confirming none was accidentally skipped.

**FEAT-33.SPEC-005-AC-10:** Given a Freelancer Account with an associated Referral Attribution record is deleted through FEAT-24, when the deletion completes, then the Referral Attribution record is removed with it -- the only removal path this spec recognizes for the entity.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 0 (explicitly N/A) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
