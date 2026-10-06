---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-006
spec_name: Time Zone & Local Date/Time Display Rule
spec_slug: time-zone-local-date-time-display-rule
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 3
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Time Zone & Local Date/Time Display Rule

## Overview

**Name:** Time Zone & Local Date/Time Display Rule
**ID:** FEAT-15.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs how dates, due dates, and times are rendered in each viewer's own time zone and familiar format, and establishes the freelancer's time zone as the basis for reminder day counts.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Freelancer Account (the `time_zone` field, as the basis for reminder counting) and the display rule applied wherever any date, due date, or time appears across the product

## Scope and Non-Goals

**In Scope:**
- The rendering rule: any date, due date, or time shown on any screen (across this and other features) displays in that viewer's own time zone and a familiar date format
- The counting rule: elapsed-day computations for reminders use the freelancer's time zone specifically, never the viewer's
- Establishing this spec as the cross-feature authority the Automated Payment Reminders schedule (FEAT-11) and Invoice Generation & Sending (FEAT-09) consume for their own date/time display and counting

**Non-Goals:**
- Setting or changing the freelancer's own time zone value -- owned entirely by FEAT-15.SPEC-002 (Freelancer Time Zone Setting); this spec only consumes the value that screen captures
- Translating date or time text into a language other than English -- excluded per scope-boundaries.md (SC-20): "English only at launch"; this spec adapts *format* (day/month order, 12- vs 24-hour convention as derived from locale) and *time zone*, never language
- The reminder schedule's own trigger logic, pause states, or send cadence -- owned entirely by Automated Payment Reminders (FEAT-11); this spec supplies only the time-zone basis that schedule's day counts are computed against
- Determining a client contact's own time zone value -- the product infers it from the viewing device/browser at render time rather than capturing and storing it as an explicit setting, since no screen in the product definition offers a client contact a time zone preference to set

## Governed Entity

**Entity:** Freelancer Account
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| time_zone | enum (recognized time zone) | The freelancer's own time zone, set via FEAT-15.SPEC-002; the basis this spec uses for reminder day counts |

**Note on scope:** the *rendering* half of this spec's purpose applies to every date, due date, and time shown on any screen product-wide, not to a single stored entity field -- there is no separate "display format" entity to enumerate; the field table above covers the one stored value (`time_zone`) this spec's rules read.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | Any date shown on this screen (none currently -- this screen carries no dates) is N/A; listed for completeness of the feature's own screens |
| FEAT-09 (Invoice Generation & Sending) | Invoice detail, invoice list, and any invoice email | Issue dates, due dates, and reminder history render in each viewer's own time zone and date format whenever displayed |
| FEAT-11 (Automated Payment Reminders) | The reminder schedule's day-3/day-10 evaluation | Elapsed-day counts are computed against the freelancer's time zone, not the viewer's, per XBR-15 |
| FEAT-12 (Freelancer Financial Dashboard) | Dashboard and drill-down date displays | Any date shown (e.g., invoice issue/due dates in a drill-down) renders in Nadia's own time zone and format |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| time_zone | Must be a recognized time zone (validated by FEAT-15.SPEC-002's screen-level input, which always presents a value from a closed list) | Always | On read by this spec | N/A -- this spec only reads the value; validation of the value itself belongs to FEAT-15.SPEC-002 | No |

## Cross-Field Rules

Not applicable -- this spec's rendering rule operates on individual date/time fields across many other entities (Invoice issue_date/due_date, Milestone target_date, Reminder Log scheduled_for/sent_at, etc.), each rendered independently in the viewer's own time zone; there is no cross-field interaction between them for the purposes of this display rule.

## Authorization Rules

Not applicable -- this is a display and computation rule, not an access-gated action. Every role that can view a date, due date, or time anywhere in the product (per that surface's own Access Matrix rules) sees it rendered under this rule; this spec grants no additional visibility and restricts none.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| Rendered date/time (any date, due date, or time on any screen) | Derived: the underlying stored instant, converted to the viewing party's own time zone and shown in a familiar date format for that viewer | Every time a date, due date, or time is displayed, on every screen | No -- the rendering is automatic and not a per-screen setting; a viewer changes what they see only by changing their own time zone (Nadia via FEAT-15.SPEC-002; a client contact via their device/browser) |
| Reminder day count (day 3 / day 10 elapsed-day evaluation) | Derived: elapsed calendar days since the invoice's due date, counted using the freelancer's `time_zone` regardless of which viewer or process is evaluating it | Every time the reminder schedule (FEAT-11) evaluates whether a reminder is due | No -- the freelancer's time zone is always the counting basis; it is not evaluated in the viewer's time zone under any circumstance |

## Business Rules

- XBR-15: overdue reminders are counted in the freelancer's time zone, not the viewer's -- this spec is the rule's sole definition; FEAT-11 consumes it.
- The rendering rule applies uniformly across every feature that displays a date, due date, or time: this spec is the cross-feature authority both FEAT-09 and FEAT-12 reference for their own date displays, rather than each feature deriving its own time-zone behavior.
- A viewer's own time zone for *display* purposes (as opposed to Nadia's for *counting* purposes) comes from the viewing device/browser for client contacts, and from the stored `time_zone` field (FEAT-15.SPEC-002) for Nadia herself -- the two are the same source when Nadia is the viewer of her own data.
- Date *format* adapts to each viewer's familiar convention (e.g., day/month/year vs. month/day/year ordering); this is a formatting adaptation only and carries no translated text, consistent with scope-boundaries.md (SC-20).

## Edge Cases

- **Nadia and Owen are in different time zones and both view the same invoice's due date at the same moment** -- Each sees the due date rendered independently in their own time zone; the two displayed values may show different calendar dates for the same underlying due date/time, and this is expected, not a defect.
- **Nadia changes her time zone (FEAT-15.SPEC-002) while a reminder's day count is mid-evaluation** -- The evaluation in flight completes using the time zone value it read at the start of that evaluation cycle; the next evaluation cycle uses the newly saved value. No reminder is double-counted or skipped due to the change.
- **A client contact's device/browser reports no detectable time zone (e.g., a misconfigured environment)** -- The portal falls back to a neutral reference time zone (UTC) for that viewer's session only, with a plain indicator that displayed times may not match their local time; Nadia's own display and the reminder day-count basis are unaffected, since neither depends on a client contact's time zone.
- **A due date lands exactly at a day boundary when converted between the freelancer's time zone and a viewer's time zone** -- Each viewer's rendering uses its own boundary independently; the freelancer's reminder day count uses only the freelancer's own boundary, so the two computations do not need to agree with each other.
- **Nadia views her own dashboard, where she is simultaneously the freelancer (reminder-count basis) and the viewer (display basis)** -- Both computations use the same stored `time_zone` value, so her displayed dates and her reminder day counts are always mutually consistent for her own view.

## Acceptance Criteria

**FEAT-15.SPEC-006-AC-01:** Given Nadia has set her time zone via FEAT-15.SPEC-002, when she views an invoice's due date, then it renders in her own time zone and a familiar date format for her.

**FEAT-15.SPEC-006-AC-02:** Given Owen is in a different time zone than Nadia, when he views the same invoice's due date, then it renders in his own time zone, independent of how it renders for Nadia.

**FEAT-15.SPEC-006-AC-03:** Given an invoice is overdue, when the reminder schedule (FEAT-11) evaluates whether a day-3 reminder is due, then the elapsed-day count is computed using Nadia's time zone, not Owen's.

**FEAT-15.SPEC-006-AC-04:** Given Nadia changes her time zone, when the next reminder evaluation cycle runs, then it uses the newly saved time zone as its counting basis.

**FEAT-15.SPEC-006-AC-05:** Given a reminder evaluation is already in progress when Nadia's time zone change saves, when that in-progress evaluation completes, then it finishes using the time zone value it started with, and only the next cycle picks up the new value.

**FEAT-15.SPEC-006-AC-06:** Given Priya opens a milestone with a target date, when she views it, then the target date renders in her own time zone and familiar format, per FEAT-15's Key Capability for time zones and local formats.

**FEAT-15.SPEC-006-AC-07:** Given a client contact's device reports no detectable time zone, when they view any date or time in the portal, then it falls back to a neutral reference time zone with a plain indicator that the displayed time may not match their local time.

**FEAT-15.SPEC-006-AC-08:** Given Nadia views her own dashboard, when she compares a displayed due date against a reminder's day count for the same invoice, then both are computed from the same time zone value and are mutually consistent.

**FEAT-15.SPEC-006-AC-09:** Given a due date falls on different calendar dates for Nadia and Owen after each converts it to their own time zone, when both view it, then each sees their own correctly converted date with no error or warning -- this divergence is expected behavior.

**FEAT-15.SPEC-006-AC-10:** Given Dana views a date inside a support session, when the date renders, then it renders in Dana's own viewing time zone, consistent with the rule applying to every viewer, not only client-facing roles.

**FEAT-15.SPEC-006-AC-11:** Given no product surface offers a client contact a time zone setting of their own, when Owen or Priya looks for one, then none exists -- their display time zone is always inferred from their device/browser, never a stored preference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 0 (N/A -- no cross-field interaction in this rule) | 0 |
| Authorization Rules | 0 (N/A -- display rule, not access-gated) | 0 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
