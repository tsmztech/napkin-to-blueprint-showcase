---
pipeline_status: blueprint-complete
active_stage: 0
last_completed_stage: 5
last_gate_result: stage-4-gate-4-passed
project_name: "Clientroom"
started_at: 2026-09-26T19:18:07Z
last_updated: 2026-10-02T17:10:02Z
---

# n2b Pipeline

- [x] Stage 1: Intake — Completed 2026-09-26 | BRIEF.md produced
- [x] Stage 2: Define Features — Completed 2026-09-26 | 33 features, 2 gates passed
- [x] Stage 3: Create Specifications — Completed 2026-09-29 | 220 specs across 33 features
- [x] Stage 4: Technical Architecture — Completed 2026-09-29 | blueprint package complete
- [x] Stage 5: Export — First export 2026-09-29 | dev-brief

## Stage History

### Stage 1: Intake
- Completed: 2026-09-26T19:20:23Z
- Gate: passed — Gate 0 Brief Validation
- Output: .n2b/BRIEF.md
- Performance: 0 agents, 2 min, 0 retries
- Detail: → .n2b/tracking/stages/s1-init/STAGE.md

### Stage 2: Define Features
- Completed: 2026-09-26T19:53:20Z
- Gate 1 (drafts): passed (7/7 files, frontmatter valid)
- Gate 2 (finals): passed (7/7 files, 241 modification markers, depth checks passed)
- Output: .n2b/features/ (7 documents)
- Detail: → .n2b/tracking/stages/s2-define/STAGE.md
- Performance: 3 agents, 31 min, 0 retries

### Stage 3: Create Specifications
- Completed: 2026-09-29T04:45:51Z
- Gate: passed — Gate A (6-category structural validation)
- Output: .n2b/specifications/ (220 specs, 33 features)
- Performance: 177 agents, 3388 min, 0 retries
- Detail: → .n2b/tracking/stages/s3-specify/STAGE.md

### Stage 4: Technical Architecture
- Completed: 2026-09-29T13:43:20Z
- Gate A (metrics): passed (features=33, specs=220 — matches filesystem)
- Gate B (landscape): passed (22 decision areas, 89 options, all Sources populated)
- Gate 4 (architecture): passed (14/14 sections, 23 entities, 33 features, 65 routes, 31 ADRs)
- Output: .n2b/architecture/ (5 documents)
- Detail: → .n2b/tracking/stages/s4-architect/STAGE.md
- Performance: 5 agents, 50 min, 0 retries

### Stage 5: Export
- Completed: 2026-09-29T16:04:10Z (first export)
- Target: dev-brief — 44 files, fidelity pass (package version 4)
- Detail: → .n2b/tracking/stages/s5-export/STAGE.md

## Artifact Lineage

| Feature | Stage 2 | Stage 3 Specs | Stage 4 Mapping |
|---------|---------|---------------|-----------------|
| FEAT-01 Client & Project Management | ✅ Defined | ✅ 11 specs (5 Screen, 2 Auto, 4 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-02 Proposal Creation & Sending | ✅ Defined | ✅ 11 specs (4 Screen, 5 Auto, 1 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-03 Proposal Acceptance | ✅ Defined | ✅ 7 specs (2 Screen, 2 Auto, 1 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-04 Milestone & Payment Schedule Setup | ✅ Defined | ✅ 4 specs (2 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-05 Client Portal Access (Magic-Link Login) | ✅ Defined | ✅ 9 specs (3 Screen, 3 Auto, 2 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-06 Deliverable Upload & Sharing | ✅ Defined | ✅ 6 specs (2 Screen, 2 Auto, 1 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-07 Deliverable Review & Feedback | ✅ Defined | ✅ 8 specs (2 Screen, 1 Auto, 3 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-08 Milestone Approval | ✅ Defined | ✅ 7 specs (2 Screen, 3 Auto, 1 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-09 Invoice Generation & Sending | ✅ Defined | ✅ 10 specs (3 Screen, 2 Auto, 4 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-10 Invoice Payment Processing | ✅ Defined | ✅ 7 specs (2 Screen, 2 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-11 Automated Payment Reminders | ✅ Defined | ✅ 5 specs (1 Screen, 2 Auto, 1 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-12 Freelancer Financial Dashboard | ✅ Defined | ✅ 4 specs (2 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-13 Immutable Activity & Audit Trail | ✅ Defined | ✅ 6 specs (2 Screen, 1 Auto, 3 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-14 Notifications (Email) | ✅ Defined | ✅ 6 specs (0 Screen, 2 Auto, 2 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-15 Currency & Tax Handling | ✅ Defined | ✅ 8 specs (2 Screen, 1 Auto, 5 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-16 Large File Handling & Storage | ✅ Defined | ✅ 7 specs (1 Screen, 4 Auto, 1 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-17 Deliverable Version History | ✅ Defined | ✅ 5 specs (2 Screen, 1 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-18 Client Contact Management & Roles | ✅ Defined | ✅ 11 specs (4 Screen, 1 Auto, 4 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-19 Freelancer Branding | ✅ Defined | ✅ 3 specs (1 Screen, 0 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-20 Onboarding / First-Run Setup | ✅ Defined | ✅ 6 specs (2 Screen, 2 Auto, 1 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-21 Settings & Account Management | ✅ Defined | ✅ 11 specs (4 Screen, 2 Auto, 4 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-22 Accounting Export | ✅ Defined | ✅ 3 specs (1 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-23 Subscription Plan & Billing Management | ✅ Defined | ✅ 8 specs (1 Screen, 4 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-24 Data Export & Account Deletion | ✅ Defined | ✅ 9 specs (2 Screen, 3 Auto, 2 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-25 Refund & Cancelled Project Handling | ✅ Defined | ✅ 8 specs (2 Screen, 3 Auto, 1 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-26 Legally Binding E-Signature for Proposals | ✅ Defined | ✅ 5 specs (1 Screen, 1 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-27 Custom Domain per Freelancer | ✅ Defined | ✅ 4 specs (1 Screen, 0 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-28 Global Search Across Clients & Projects | ✅ Defined | ✅ 4 specs (1 Screen, 1 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-29 In-App Notification Center | ✅ Defined | ✅ 4 specs (1 Screen, 2 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-30 Contextual Help & Guidance | ✅ Defined | ✅ 5 specs (3 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-31 Operator Support Access | ✅ Defined | ✅ 7 specs (2 Screen, 2 Auto, 1 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-32 Payment Account Connection | ✅ Defined | ✅ 6 specs (1 Screen, 2 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-33 Portal Referral Attribution | ✅ Defined | ✅ 5 specs (1 Screen, 2 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |

## Export History

<!-- Append-only. Rows are appended by the export-complete transition; the gatekeeper's re-run confirm marks rows stale; the status workflow reads them for staleness. Never edit or remove rows. Status values: current | stale. -->

| # | Target | Package version | Artifacts | Completed at | Status |
|---|--------|-----------------|-----------|--------------|--------|
| 1 | dev-brief | 4 | 44 | 2026-09-29T16:04:10Z | current |
| 2 | speckit | 4 | 338 | 2026-10-02T17:10:02Z | current |
