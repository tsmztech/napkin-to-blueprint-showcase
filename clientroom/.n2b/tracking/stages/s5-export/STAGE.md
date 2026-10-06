---
stage: 5
stage_name: "Export"
status: complete
targets_completed: 2
targets_in_progress: 1
files_rendered_total: 382
last_export_at: 2026-10-02T17:10:02Z
---

<!-- LIVE DASHBOARD — Stage 5 exception (tracking-protocol.md): unlike Stage 1–4 STAGE.md files, this dashboard is a live document for the life of the project and is EXEMPT from the receipt write-lock — exports accumulate over time, so it is never sealed. The receipts are the per-target files beside it in s5-export/ (template: export-target-tracker.md), each write-locked once its status is done. Same dashboard-live / per-item-receipt pattern Stage 3 uses per feature.
     status here reflects the current run: in-progress while an export run is active (stage-resume-s5 keys on it), back to a resting value when the run ends. It never blocks a future run — re-export is always legal. -->

This file is the s5-export dashboard. Export runs update it continuously; per-target receipt files carry the write-locked record of each completed export.

## Targets

| Target | Status | Package version | Files | Fidelity | Exported at |
|--------|--------|-----------------|-------|----------|-------------|
| dev-brief | done | 4 | 44 | pass | 2026-09-29T16:04:10Z |
| jira | in-progress | 4 | 0 | fail (Backlog Builder — upstream AC defect, 2 attempts) | — |
| speckit | done | 4 | 338 | pass | 2026-10-02T17:10:02Z |

(One row per export target ever attempted. Status: not-started / in-progress / done / stale. Rows are added on first attempt and updated in place — `export-complete` updates the target's row and the frontmatter counters; the gatekeeper's confirmed upstream re-run flips affected done rows to stale. Package version = the MANIFEST.md `package_version` the export was rendered against.)

## Activity Log

(Append-only, newest last. One line per event: `{ISO timestamp} — {target} — {event}` — run started, resumed (with per-target classification), fidelity gate passed/failed, export complete, marked stale, per-target refresh confirmed.)

- 2026-09-29T15:59:23Z — dev-brief — run started (fresh; package version 4, 269 inventory artifacts verified, 0 drifted)
- 2026-09-29T16:01:15Z — dev-brief — formatter complete (44 files: 11 part files + 33 E chapters)
- 2026-09-29T16:01:35Z — dev-brief — fidelity gate 4a passed (attempt 1; R0, U1–U6, DEV-1..DEV-3 — 0 errors)
- 2026-09-29T16:02:50Z — dev-brief — fidelity gate 4b passed (Export Fidelity Checker: 220/220 specs verbatim, 0 fail findings, 1 warn nit — "a Important-tier" article in 11 chapter openers)
- 2026-09-29T16:04:10Z — dev-brief — receipt written; export complete (package version 4, 44 files, fidelity pass)
- 2026-09-29T16:38:00Z — jira — run started (fresh; package version 4, 269 inventory artifacts verified, 0 drifted)
- 2026-09-29T16:40:20Z — jira — Backlog Builder failed (round 1): FEAT-20.SPEC-002-AC-25 lacks a then-delimiter (backlog-schema §6.2 / §7 rule 5) — backlog.json not written
- 2026-09-29T16:41:10Z — jira — Backlog Builder failed (round 2): identical failure; run halted — upstream Stage 3 spec defect, formatter and fidelity gate never ran
- 2026-10-02T16:59:52Z — speckit — run started (resume classification: dashboard in-progress from jira; speckit tracker missing → not started → fresh run; package version 4, 269 inventory artifacts verified, 0 drifted)
- 2026-10-02T17:09:30Z — speckit — formatter complete (338 files: README, constitution, feature.json, 33 spec.md + 33 research.md, 269-file docs/blueprint copy)
- 2026-10-02T17:10:00Z — speckit — fidelity gate 4a passed (attempt 1; R0, U1–U6, SK-1..SK-7 — 0 errors)
- 2026-10-02T17:10:02Z — speckit — fidelity gate 4b passed (Export Fidelity Checker: 0 fail, 0 warn findings)
- 2026-10-02T17:10:02Z — speckit — receipt written; export complete (package version 4, 338 files, fidelity pass)
