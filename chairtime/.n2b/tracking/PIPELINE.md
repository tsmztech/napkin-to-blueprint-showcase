---
pipeline_status: blueprint-complete
active_stage: 0
last_completed_stage: 5
last_gate_result: stage-4-gate-4-passed
project_name: "Chairtime"
started_at: 2026-09-26T18:33:46Z
last_updated: 2026-10-02T16:35:30Z
---

# n2b Pipeline

- [x] Stage 1: Intake — Completed 2026-09-26 | BRIEF.md produced
- [x] Stage 2: Define Features — Completed 2026-09-26 | 30 features, 2 gates passed
- [x] Stage 3: Create Specifications — Completed 2026-09-28 | 219 specs across 30 features
- [x] Stage 4: Technical Architecture — Completed 2026-09-29 | blueprint package complete
- [x] Stage 5: Export — First export 2026-09-29 | dev-brief

## Stage History

### Stage 1: Intake
- Completed: 2026-09-26T18:35:49Z
- Gate: passed — Gate 0 Brief Validation
- Output: .n2b/BRIEF.md
- Performance: 0 agents, 3 min, 0 retries
- Detail: → .n2b/tracking/stages/s1-init/STAGE.md

### Stage 2: Define Features
- Completed: 2026-09-26T19:06:47Z
- Gate 1 (drafts): passed (7/7 files, frontmatter valid)
- Gate 2 (finals): passed (7/7 files, 176 modification markers, depth checks passed)
- Output: .n2b/features/ (7 documents)
- Detail: → .n2b/tracking/stages/s2-define/STAGE.md
- Performance: 3 agents, 28 min, 0 retries

### Stage 3: Create Specifications
- Completed: 2026-09-28T23:55:46Z
- Gate: passed — Gate A (6-category structural validation), 4 soft warnings
- Output: .n2b/specifications/ (219 specs, 30 features)
- Performance: ~186 agents, 3155 min, 2 retries
- Detail: → .n2b/tracking/stages/s3-specify/STAGE.md

### Stage 4: Technical Architecture
- Completed: 2026-09-29T01:05:02Z
- Gate A (metrics): passed (features=30, specs=219 — matches filesystem)
- Gate B (landscape): passed (21 decision areas, 103 options, all Sources populated)
- Gate 4 (architecture): passed (14/14 sections, 18 entities, 30 features, 70 routes, 30 ADRs)
- Output: .n2b/architecture/ (5 documents)
- Detail: → .n2b/tracking/stages/s4-architect/STAGE.md
- Performance: 5 agents, 43 min, 0 retries

### Stage 5: Export
- Completed: 2026-09-29T01:18:58Z (first export)
- Target: dev-brief — package version 4, 41 files, fidelity pass
- Detail: → .n2b/tracking/stages/s5-export/STAGE.md

## Artifact Lineage

| Feature | Stage 2 | Stage 3 Specs | Stage 4 Mapping |
|---------|---------|---------------|-----------------|
| FEAT-01 Service & Pricing Management | ✅ Defined | ✅ 6 specs (3 Screen, 1 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-02 Availability & Working Hours Setup | ✅ Defined | ✅ 5 specs (2 Screen, 2 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-03 Real-Time Slot Availability Engine | ✅ Defined | ✅ 7 specs (0 Screen, 4 Auto, 2 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-04 Two-Way Calendar Sync | ✅ Defined | ✅ 8 specs (2 Screen, 3 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-05 Public Booking Page & Booking Flow | ✅ Defined | ✅ 9 specs (5 Screen, 1 Auto, 3 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-06 Client Booking Identity | ✅ Defined | ✅ 8 specs (4 Screen, 1 Auto, 2 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-07 Deposit Payment at Booking | ✅ Defined | ✅ 5 specs (1 Screen, 1 Auto, 2 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-08 Automated Booking Messaging | ✅ Defined | ✅ 13 specs (1 Screen, 4 Auto, 1 Logic, 2 Integ, 5 Notif) | ✅ Mapped |
| FEAT-09 Cancellation & No-Show Policy Engine | ✅ Defined | ✅ 6 specs (1 Screen, 1 Auto, 3 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-10 Client-Initiated Cancel/Reschedule | ✅ Defined | ✅ 6 specs (3 Screen, 1 Auto, 1 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-11 No-Show Marking & Deposit Forfeiture | ✅ Defined | ✅ 4 specs (1 Screen, 2 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-12 Pro Daily Schedule Dashboard | ✅ Defined | ✅ 8 specs (3 Screen, 2 Auto, 3 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-13 Client Record Management | ✅ Defined | ✅ 6 specs (3 Screen, 1 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-14 Messaging Consent Management | ✅ Defined | ✅ 9 specs (2 Screen, 3 Auto, 3 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-15 Pro Onboarding & Setup Wizard | ✅ Defined | ✅ 8 specs (3 Screen, 2 Auto, 2 Logic, 0 Integ, 1 Notif) | ✅ Mapped |
| FEAT-16 Booking & Payment Activity Record | ✅ Defined | ✅ 5 specs (1 Screen, 2 Auto, 1 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-17 Manual Time Blocking | ✅ Defined | ✅ 8 specs (3 Screen, 4 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-18 Pro Subscription Billing & Account Management | ✅ Defined | ✅ 7 specs (2 Screen, 2 Auto, 1 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-19 Platform Support Read-Only Access | ✅ Defined | ✅ 4 specs (2 Screen, 1 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-20 Waitlist for Cancelled Slots | ✅ Defined | ✅ 9 specs (2 Screen, 3 Auto, 2 Logic, 0 Integ, 2 Notif) | ✅ Mapped |
| FEAT-21 Recurring/Standing Appointments | ✅ Defined | ✅ 10 specs (3 Screen, 2 Auto, 2 Logic, 0 Integ, 3 Notif) | ✅ Mapped |
| FEAT-22 In-App Balance Payment | ✅ Defined | ✅ 5 specs (1 Screen, 1 Auto, 2 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-23 Tipping at Checkout | ✅ Defined | ✅ 3 specs (1 Screen, 0 Auto, 2 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-24 Client List Search & Filter | ✅ Defined | ✅ 2 specs (1 Screen, 0 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-25 Booking & Revenue Insights | ✅ Defined | ✅ 4 specs (1 Screen, 2 Auto, 1 Logic, 0 Integ, 0 Notif) | ✅ Mapped |
| FEAT-26 WhatsApp Reminders | ✅ Defined | ✅ 4 specs (1 Screen, 1 Auto, 1 Logic, 1 Integ, 0 Notif) | ✅ Mapped |
| FEAT-27 Pro Profile & Booking Page Settings | ✅ Defined | ✅ 13 specs (6 Screen, 2 Auto, 3 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-28 Payout Account Connection & Payout Visibility | ✅ Defined | ✅ 7 specs (2 Screen, 1 Auto, 2 Logic, 1 Integ, 1 Notif) | ✅ Mapped |
| FEAT-29 Pro Sign-In & Account Lifecycle | ✅ Defined | ✅ 17 specs (5 Screen, 5 Auto, 3 Logic, 0 Integ, 4 Notif) | ✅ Mapped |
| FEAT-30 Pro Booking Management | ✅ Defined | ✅ 13 specs (5 Screen, 4 Auto, 1 Logic, 1 Integ, 2 Notif) | ✅ Mapped |

## Export History

<!-- Append-only. Rows are appended by the export-complete transition; the gatekeeper's re-run confirm marks rows stale; the status workflow reads them for staleness. Never edit or remove rows. Status values: current | stale. -->

| # | Target | Package version | Artifacts | Completed at | Status |
|---|--------|-----------------|-----------|--------------|--------|
| 1 | dev-brief | 4 | 41 | 2026-09-29T01:18:58Z | current |
| 2 | lovable-pack | 4 | 269 | 2026-10-02T16:35:30Z | current |
