---
target: dev-brief
exported_at: 2026-09-28T23:21:28Z
package_version: 4
files:
  - 00-README.md
  - A-product-vision.md
  - B-usage-and-success.md
  - C-scope-and-assumptions.md
  - COMBINED.md
  - D-feature-catalog.md
  - E-feature-specifications/FEAT-01-household-setup-member-profiles.md
  - E-feature-specifications/FEAT-02-dietary-rules-allergy-safety-engine.md
  - E-feature-specifications/FEAT-03-ai-weekly-dinner-plan-generation.md
  - E-feature-specifications/FEAT-04-one-tap-meal-swap.md
  - E-feature-specifications/FEAT-05-pantry-aware-suggestions.md
  - E-feature-specifications/FEAT-06-shared-grocery-list.md
  - E-feature-specifications/FEAT-07-weekly-plan-ready-notification.md
  - E-feature-specifications/FEAT-08-recipe-library-starter-recipes.md
  - E-feature-specifications/FEAT-09-household-invitations-membership.md
  - E-feature-specifications/FEAT-10-recipe-import-from-web-link.md
  - E-feature-specifications/FEAT-11-leftover-rollover-to-lunches.md
  - E-feature-specifications/FEAT-12-meal-rating-preference-learning.md
  - E-feature-specifications/FEAT-13-tonights-dinner-reminder.md
  - E-feature-specifications/FEAT-14-subscription-billing-management.md
  - E-feature-specifications/FEAT-15-member-onboarding.md
  - E-feature-specifications/FEAT-16-units-currency-locale-configuration.md
  - E-feature-specifications/FEAT-17-older-kid-dinner-voting.md
  - E-feature-specifications/FEAT-18-account-data-management.md
  - E-feature-specifications/FEAT-19-weekly-plan-history.md
  - E-feature-specifications/FEAT-20-online-grocery-ordering-handoff.md
  - E-feature-specifications/FEAT-21-family-calendar-sync.md
  - E-feature-specifications/FEAT-22-operator-read-only-support-access.md
  - E-feature-specifications/FEAT-23-manual-weekly-planning.md
  - E-feature-specifications/FEAT-24-invite-another-household.md
  - E-feature-specifications/FEAT-25-weekly-waste-spend-check-in.md
  - F-data-model.md
  - G1-design-layer.md
  - G2-architecture.md
  - G3-database-schema.md
  - H-appendices.md
feature_count: 25
spec_count: 196
ac_count: 2292
fidelity_result: pass
---

# Export Receipt — dev-brief

<!-- Rules for this document:
  - Frontmatter is contract C-29 — exactly these fields, no additions.
  - Written ONLY by the export workflow (n2b/workflows/stage-5/export.md, Step 5), during
    the export-complete transition (n2b/references/tracking-protocol.md) — never by an
    agent. Agents write deliverables; the workflow writes receipts and MANIFEST.md.
  - fidelity_result is always pass: a receipt exists only for an export whose fidelity gate
    (4a reconciliation + 4b semantic review) passed. There is no fail receipt — a failed
    gate leaves no receipt, which is exactly what stage-resume-s5 classification keys on.
  - files lists every rendered file in the export directory except FIDELITY-REPORT.md and
    this receipt.
-->

This receipt certifies that the `dev-brief` export in this directory was rendered from
the canonical blueprint package at `package_version` 4, and passed the export
fidelity gate: bash reconciliation of the FEAT / SPEC / AC / XBR / ADR / SC / ASMP rosters
(4a) and the semantic fidelity review (4b, see `FIDELITY-REPORT.md` alongside this file).

**Staleness** is judged by comparing this receipt's `package_version` against the current
`package_version` in `.n2b/tracking/MANIFEST.md`: equal → this export is current; behind →
the canonical package has changed since this render and the export is stale. Refresh it any
time with `/n2b:s5-export dev-brief` — refreshing one target never touches any other
target's export.

This file is the deliverable-side receipt (its tracking-side counterpart lives at
`.n2b/tracking/stages/s5-export/dev-brief.md`, template
`n2b/templates/tracking/export-target-tracker.md`). Its presence marks the target complete
for `stage-resume-s5` classification; it is written once per passing render and replaced
only by a user-confirmed per-target refresh.
