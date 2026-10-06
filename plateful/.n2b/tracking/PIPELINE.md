---
pipeline_status: blueprint-complete
active_stage: 0
last_completed_stage: 5
last_gate_result: stage-4-gate-4-passed
project_name: "Plateful"
started_at: 2026-09-26T19:16:37Z
last_updated: 2026-09-29T00:27:25Z
---

# n2b Pipeline

- [x] Stage 1: Intake — Completed 2026-09-26 | BRIEF.md produced
- [x] Stage 2: Define Features — Completed 2026-09-26 | 25 features, 2 gates passed
- [x] Stage 3: Create Specifications — Completed 2026-09-28 | 196 specs across 25 features
- [x] Stage 4: Technical Architecture — Completed 2026-09-28 | blueprint package complete
- [x] Stage 5: Export — First export 2026-09-28 | dev-brief

## Stage History

### Stage 1: Intake
- Completed: 2026-09-26T19:18:36Z
- Gate: passed — Gate 0 Brief Validation
- Output: .n2b/BRIEF.md
- Performance: 0 agents, 3 min, 0 retries
- Detail: → .n2b/tracking/stages/s1-init/STAGE.md

### Stage 2: Define Features
- Completed: 2026-09-26T19:47:08Z
- Gate 1 (drafts): passed (7/7 files, frontmatter valid)
- Gate 2 (finals): passed (7/7 files, 177 modification markers, depth checks passed)
- Output: .n2b/features/ (7 documents)
- Detail: → .n2b/tracking/stages/s2-define/STAGE.md
- Performance: 3 agents, 26 min, 0 retries

### Stage 3: Create Specifications
- Completed: 2026-09-28T04:10:56Z
- Gate: passed — Gate A (6-category structural validation; 10 soft warnings)
- Output: .n2b/specifications/ (196 specs, 25 features)
- Performance: 127 agents, 1915 min, 1 retries
- Detail: → .n2b/tracking/stages/s3-specify/STAGE.md

### Stage 4: Technical Architecture
- Completed: 2026-09-28T14:25:27Z
- Gate A (metrics): passed (features=25, specs=196 — matches filesystem)
- Gate B (landscape): passed (25 decision areas, 104 options, all Sources populated)
- Gate 4 (architecture): passed (14/14 sections, 17 entities, 25 features, 60 routes, 34 ADRs)
- Output: .n2b/architecture/ (5 documents)
- Detail: → .n2b/tracking/stages/s4-architect/STAGE.md
- Performance: 5 agents, 47 min, 0 retries

### Stage 5: Export
- Completed: 2026-09-28T23:21:28Z (first export)
- Target: dev-brief — fidelity pass, 36 files, package version 4
- Detail: → .n2b/tracking/stages/s5-export/STAGE.md

## Artifact Lineage

| Feature | Stage 2 | Stage 3 Specs | Stage 4 Mapping |
|---------|---------|---------------|-----------------|
| FEAT-01 Household Setup & Member Profiles | ✅ Defined | ✅ 18 specs (10 Screen, 2 Auto, 4 Logic, 1 Integ, 1 Notif) — specifications/FEAT-01-household-setup-member-profiles/ | ✅ Mapped |
| FEAT-02 Dietary Rules & Allergy Safety Engine | ✅ Defined | ✅ 14 specs (1 Screen, 4 Auto, 4 Logic, 1 Integ, 4 Notif) — specifications/FEAT-02-dietary-rules-allergy-safety-engine/ | ✅ Mapped |
| FEAT-03 AI Weekly Dinner Plan Generation | ✅ Defined | ✅ 11 specs (2 Screen, 3 Auto, 4 Logic, 2 Integ, 0 Notif) — specifications/FEAT-03-ai-weekly-dinner-plan-generation/ | ✅ Mapped |
| FEAT-04 One-Tap Meal Swap | ✅ Defined | ✅ 10 specs (3 Screen, 2 Auto, 3 Logic, 1 Integ, 1 Notif) — specifications/FEAT-04-one-tap-meal-swap/ | ✅ Mapped |
| FEAT-05 Pantry-Aware Suggestions | ✅ Defined | ✅ 7 specs (1 Screen, 2 Auto, 4 Logic, 0 Integ, 0 Notif) — specifications/FEAT-05-pantry-aware-suggestions/ | ✅ Mapped |
| FEAT-06 Shared Grocery List | ✅ Defined | ✅ 9 specs (1 Screen, 3 Auto, 4 Logic, 1 Integ, 0 Notif) — specifications/FEAT-06-shared-grocery-list/ | ✅ Mapped |
| FEAT-07 Weekly Plan Ready Notification | ✅ Defined | ✅ 6 specs (0 Screen, 1 Auto, 2 Logic, 2 Integ, 1 Notif) — specifications/FEAT-07-weekly-plan-ready-notification/ | ✅ Mapped |
| FEAT-08 Recipe Library (Starter Recipes) | ✅ Defined | ✅ 4 specs (2 Screen, 0 Auto, 1 Logic, 1 Integ, 0 Notif) — specifications/FEAT-08-recipe-library-starter-recipes/ | ✅ Mapped |
| FEAT-09 Household Invitations & Membership | ✅ Defined | ✅ 14 specs (5 Screen, 4 Auto, 2 Logic, 0 Integ, 3 Notif) — specifications/FEAT-09-household-invitations-membership/ | ✅ Mapped |
| FEAT-10 Recipe Import from Web Link | ✅ Defined | ✅ 8 specs (4 Screen, 2 Auto, 1 Logic, 1 Integ, 0 Notif) — specifications/FEAT-10-recipe-import-from-web-link/ | ✅ Mapped |
| FEAT-11 Leftover Rollover to Lunches | ✅ Defined | ✅ 4 specs (1 Screen, 2 Auto, 1 Logic, 0 Integ, 0 Notif) — specifications/FEAT-11-leftover-rollover-to-lunches/ | ✅ Mapped |
| FEAT-12 Meal Rating & Preference Learning | ✅ Defined | ✅ 5 specs (1 Screen, 1 Auto, 3 Logic, 0 Integ, 0 Notif) — specifications/FEAT-12-meal-rating-preference-learning/ | ✅ Mapped |
| FEAT-13 Tonight's Dinner Reminder | ✅ Defined | ✅ 6 specs (0 Screen, 2 Auto, 2 Logic, 0 Integ, 2 Notif) — specifications/FEAT-13-tonights-dinner-reminder/ | ✅ Mapped |
| FEAT-14 Subscription & Billing Management | ✅ Defined | ✅ 12 specs (4 Screen, 2 Auto, 2 Logic, 2 Integ, 2 Notif) — specifications/FEAT-14-subscription-billing-management/ | ✅ Mapped |
| FEAT-15 Member Onboarding | ✅ Defined | ✅ 3 specs (1 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) — specifications/FEAT-15-member-onboarding/ | ✅ Mapped |
| FEAT-16 Units, Currency & Locale Configuration | ✅ Defined | ✅ 4 specs (2 Screen, 0 Auto, 2 Logic, 0 Integ, 0 Notif) — specifications/FEAT-16-units-currency-locale-configuration/ | ✅ Mapped |
| FEAT-17 Older-Kid Dinner Voting | ✅ Defined | ✅ 6 specs (3 Screen, 2 Auto, 1 Logic, 0 Integ, 0 Notif) — specifications/FEAT-17-older-kid-dinner-voting/ | ✅ Mapped |
| FEAT-18 Account & Data Management | ✅ Defined | ✅ 15 specs (5 Screen, 4 Auto, 2 Logic, 1 Integ, 3 Notif) — specifications/FEAT-18-account-data-management/ | ✅ Mapped |
| FEAT-19 Weekly Plan History | ✅ Defined | ✅ 4 specs (2 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) — specifications/FEAT-19-weekly-plan-history/ | ✅ Mapped |
| FEAT-20 Online Grocery Ordering Handoff | ✅ Defined | ✅ 3 specs (1 Screen, 0 Auto, 1 Logic, 1 Integ, 0 Notif) — specifications/FEAT-20-online-grocery-ordering-handoff/ | ✅ Mapped |
| FEAT-21 Family Calendar Sync | ✅ Defined | ✅ 5 specs (1 Screen, 1 Auto, 2 Logic, 1 Integ, 0 Notif) — specifications/FEAT-21-family-calendar-sync/ | ✅ Mapped |
| FEAT-22 Operator Read-Only Support Access | ✅ Defined | ✅ 10 specs (3 Screen, 2 Auto, 3 Logic, 0 Integ, 2 Notif) — specifications/FEAT-22-operator-read-only-support-access/ | ✅ Mapped |
| FEAT-23 Manual Weekly Planning | ✅ Defined | ✅ 6 specs (3 Screen, 1 Auto, 2 Logic, 0 Integ, 0 Notif) — specifications/FEAT-23-manual-weekly-planning/ | ✅ Mapped |
| FEAT-24 Invite Another Household | ✅ Defined | ✅ 7 specs (2 Screen, 3 Auto, 1 Logic, 0 Integ, 1 Notif) — specifications/FEAT-24-invite-another-household/ | ✅ Mapped |
| FEAT-25 Weekly Waste & Spend Check-In | ✅ Defined | ✅ 5 specs (2 Screen, 1 Auto, 2 Logic, 0 Integ, 0 Notif) — specifications/FEAT-25-weekly-waste-spend-check-in/ | ✅ Mapped |

## Export History

<!-- Append-only. Rows are appended by the export-complete transition; the gatekeeper's re-run confirm marks rows stale; the status workflow reads them for staleness. Never edit or remove rows. Status values: current | stale. -->

| # | Target | Package version | Artifacts | Completed at | Status |
|---|--------|-----------------|-----------|--------------|--------|
| 1 | dev-brief | 4 | 36 | 2026-09-28T23:21:28Z | current |
| 2 | agent-workspace | 4 | 246 | 2026-09-29T00:27:25Z | current |
