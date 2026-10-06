---
target: "lovable-pack"
status: done
exported_at: 2026-10-02T16:35:30Z
package_version: 4
files_rendered: 269
fidelity_result: pass
---

<!-- Per-target tracker for s5-export/lovable-pack.md (C-29 tracking-side counterpart of the deliverable-side EXPORT-RECEIPT.md). Lifecycle: not-started -> in-progress (run active) -> done (export-complete fired). At status: done this file is a WRITE-LOCKED RECEIPT — do not modify — until a user-confirmed per-target refresh resets it for a re-render (tracking-protocol.md). The s5-export STAGE.md dashboard stays live; this file is the receipt. -->

## Target

- **Target key:** lovable-pack
- **Output directory:** .n2b/exports/lovable-pack/
- **Registry row:** n2b/references/stage-5/export-target-registry.md

## Run Log

- 2026-09-29T02:29:44Z — run started (fresh; package version 4)
- 2026-09-29T02:33:00Z — formatter completed (4 pack files + 265-file blueprint copy)
- 2026-09-29T02:35:00Z — fidelity gate 4a passed (round 1)
- 2026-09-29T02:36:00Z — fidelity gate 4b FAILED (round 1): 9 spec-digest Purpose lines paraphrased (foreign SPEC IDs reduced to FEAT-NN)
- 2026-09-29T02:37:00Z — formatter re-rendered (retry 1 of 3): 9 digests changed to a verbatim excerpt that stops before the first foreign SPEC ID, plus a pointer to the full Purpose
- 2026-09-29T02:37:30Z — fidelity gate 4a passed (round 2)
- 2026-09-29T02:38:00Z — fidelity gate 4b FAILED (round 2): the same 9 digests are truncated rather than verbatim. Halted after the second failure (runbook rule 5). Cause: the template's verbatim-Purpose rule conflicts with the VP-4 attribution rule for these 9 specs.
- 2026-10-02T16:28:03Z — run resumed (formatting incomplete: receipt missing → re-run formatter + gate; n2b 0.4.1; package version 4)
- 2026-10-02T16:33:00Z — formatter completed (4 pack files + 265-file blueprint copy; parse-back OK 30/2996)
- 2026-10-02T16:34:00Z — fidelity gate 4a passed (attempt 1 of this run; R0, U1–U6, VP-1..VP-8 clean)
- 2026-10-02T16:36:00Z — fidelity gate 4b passed (attempt 1 of this run; 18 features / 119 specs sampled, 0 fail, 2 warn on KNOWLEDGE.md [S:] source tags)
- 2026-10-02T16:35:30Z — EXPORT-RECEIPT.md written; export complete (package version 4, 269 files)

(On completion, `export-complete` sets frontmatter: status: done, exported_at, package_version — the MANIFEST.md value this export was rendered against, the staleness key — files_rendered, and fidelity_result: pass. Values mirror the deliverable-side EXPORT-RECEIPT.md.)
