---
target: "jira"
status: in-progress
exported_at: null
package_version: null
files_rendered: 0
fidelity_result: fail
---

<!-- Per-target tracker for s5-export/jira.md (C-29 tracking-side counterpart of the deliverable-side EXPORT-RECEIPT.md). Lifecycle: not-started -> in-progress (run active) -> done (export-complete fired). At status: done this file is a WRITE-LOCKED RECEIPT — do not modify — until a user-confirmed per-target refresh resets it for a re-render (tracking-protocol.md). The s5-export STAGE.md dashboard stays live; this file is the receipt. -->

## Target

- **Target key:** jira
- **Output directory:** .n2b/exports/jira/
- **Registry row:** n2b/references/stage-5/export-target-registry.md

## Run Log

(Append-only, newest last. One line per event: `{ISO timestamp} — {event}` — run started, resume classification result, formatter completed, fidelity gate 4a/4b outcome (with round number on retries), export-complete fired, stale marked by upstream re-run, user-confirmed refresh reset.)

- 2026-09-29T16:38:00Z — run started (fresh; package version 4)
- 2026-09-29T16:40:20Z — Backlog Builder failed (round 1): backlog.json not written — FEAT-20.SPEC-002-AC-25 has no then-delimiter (schema §6.2 / §7 rule 5); 3101 ACs parsed vs 3102 canonical
- 2026-09-29T16:41:10Z — Backlog Builder failed (round 2, re-prompt 1 of 3): identical failure — upstream canonical defect, re-run cannot fix
- 2026-09-29T16:41:35Z — run halted (fidelity_result: fail); formatter and gate 4a/4b never ran; upstream fix required in FEAT-20.SPEC-002 before re-running /n2b:s5-export jira

(On completion, `export-complete` sets frontmatter: status: done, exported_at, package_version — the MANIFEST.md value this export was rendered against, the staleness key — files_rendered, and fidelity_result: pass. Values mirror the deliverable-side EXPORT-RECEIPT.md.)
