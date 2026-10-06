---
document_type: fidelity-report
target: jira
checked_at: 2026-09-29T16:41:35Z
package_version: 4
samples_checked: 0
verdict: fail
---

# Export Fidelity Report — jira

<!-- Rules for this document:
  - Written by the export-fidelity-checker agent (gate 4b) into the target's export
    directory, EXCEPT Section 1: the export workflow appends the 4a bash reconciliation
    results (one row per rule executed, per n2b/references/stage-5/fidelity-rules.md) after
    the gate loop settles. The checker leaves Section 1's table exactly as rendered here.
  - This file and EXPORT-RECEIPT.md are gate artifacts: they are excluded from every
    fidelity scan and from COMBINED-style concatenations.
  - verdict is the 4b verdict: fail if any finding row carries severity fail; a report with
    only warn rows (or none) is a pass. The workflow combines it with the 4a result — both
    must pass before the export-complete transition fires.
  - Every finding must carry both sides of the evidence. "Table truncated" without the
    missing row named is not a finding.
  - On a formatter re-run, this report is rewritten from scratch — it describes the current
    render only.
-->

## 1. Reconciliation Summary (4a — appended by the workflow)

Bash reconciliation per `fidelity-rules.md`, executed by the export workflow. The final
attempt's results are recorded here by the workflow; the fidelity checker never fills this
section.

| Rule | Expected | Found | Result |
|------|----------|-------|--------|
| Step 2.5 Backlog Builder — backlog.json produced | `.n2b/exports/jira/backlog.json` exists and non-empty (backlog-schema.md §7 all rules passing) | not written — builder script exits on §7 rule 5 / §6.2: `FEAT-20.SPEC-002-AC-25` (specifications/FEAT-20-onboarding-first-run-setup/FEAT-20.SPEC-002-onboarding-guided-sequence.md line 258) has no `, then ` delimiter; 3101 ACs parsed vs 3102 canonical | fail |
| R0, U1–U6, JIRA-1..JIRA-9 | — | not executed — no render exists (the formatter consumes backlog.json and was never spawned) | not run |

Attempts: 2 of 3 used (initial build + 1 re-prompt, both failing identically). The run was
halted after the second identical failure: the defect is in a canonical Stage 3 spec line, not
in the export render, so re-running the builder cannot fix it (same class as the §U6
upstream-defect carve-out). Upstream fix: rewrite `FEAT-20.SPEC-002-AC-25` into
`Given …, when …, then …` form via the owning stage, then re-run `/n2b:s5-export jira`.

## 2. Semantic Findings (4b)

**Sample:** none — gate 4b never ran (no render on disk) — 0 features / 0 specs compared.

No semantic findings recorded; the export was halted before the render pass.

## 3. Verdict

**FAIL** — the export could not be rendered: the Backlog Builder's losslessness validation
rejects one canonical acceptance criterion (FEAT-20.SPEC-002-AC-25, no then-clause), an
upstream Stage 3 defect that must be fixed in the spec before this target can render.
