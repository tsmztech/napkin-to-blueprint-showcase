---
target: "agent-workspace"
status: done
exported_at: 2026-09-29T00:27:25Z
package_version: 4
files_rendered: 246
fidelity_result: pass
---

<!-- Per-target tracker for s5-export/agent-workspace.md (C-29 tracking-side counterpart of the deliverable-side EXPORT-RECEIPT.md). Lifecycle: not-started -> in-progress (run active) -> done (export-complete fired). At status: done this file is a WRITE-LOCKED RECEIPT — do not modify — until a user-confirmed per-target refresh resets it for a re-render (tracking-protocol.md). The s5-export STAGE.md dashboard stays live; this file is the receipt. -->

## Target

- **Target key:** agent-workspace
- **Output directory:** .n2b/exports/agent-workspace/
- **Registry row:** n2b/references/stage-5/export-target-registry.md

## Run Log

(Append-only, newest last. One line per event: `{ISO timestamp} — {event}` — run started, resume classification result, formatter completed, fidelity gate 4a/4b outcome (with round number on retries), export-complete fired, stale marked by upstream re-run, user-confirmed refresh reset.)

- 2026-09-29T00:23:18Z — run started (fresh, package version 4)

(On completion, `export-complete` sets frontmatter: status: done, exported_at, package_version — the MANIFEST.md value this export was rendered against, the staleness key — files_rendered, and fidelity_result: pass. Values mirror the deliverable-side EXPORT-RECEIPT.md.)
- 2026-09-29T00:24:58Z — formatter completed (246 files)
- 2026-09-29T00:25:56Z — fidelity gate 4a passed (round 1)
- 2026-09-29T00:27:14Z — fidelity gate 4b passed (round 1; 13 features / 114 specs sampled, 0 warnings)
- 2026-09-29T00:27:25Z — EXPORT-RECEIPT.md written; export-complete fired
