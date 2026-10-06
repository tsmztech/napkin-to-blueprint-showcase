---
document_type: fidelity-report
target: dev-brief
checked_at: 2026-09-28T23:20:58Z
package_version: 4
samples_checked: 196
verdict: pass
---

# Export Fidelity Report — dev-brief

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
| R0 cross-check (spec files vs technical-profile §6) | 196 = 196 | 196 files / 196 in §6 | pass |
| U1 FEAT coverage | all 25 FEAT IDs | 25 of 25 present | pass |
| U2 SPEC coverage | all 196 SPEC IDs | 196 of 196 present | pass |
| U3 AC count | ≥ 2292 distinct AC IDs | 2292 distinct AC IDs | pass |
| U4 cross-cutting rosters | 110 IDs (20 XBR, 34 ADR, 19 SC, 37 ASMP) | 110 of 110 present | pass |
| U5 no introduced placeholders | ≤ 0 TBD/TODO (canonical 0; COMBINED.md excluded) | 0 | pass |
| U6 regression lint | 0 banned-phrase hits | 0 | pass |
| DEV-1 part-file completeness | 11 C-28 part files non-empty | 11 of 11 | pass |
| DEV-2 E chapter per feature | 25 chapters | 25 of 25 | pass |
| DEV-3 COMBINED.md completeness | 35 part first-headings in COMBINED.md (10 parts + 25 E chapters) | 35 of 35 | pass |

Attempt 1 of 3 passed — no re-renders used.

## 2. Semantic Findings (4b)

**Sample:** All 25 features (exceeds the minimum of all Core + 3 non-Core). Core: FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-23. Non-Core: FEAT-10 through FEAT-22, FEAT-24, FEAT-25 — 25 features / 196 specs compared (plus 25 feature overviews).

Method: every canonical spec and feature-overview body (frontmatter stripped) was tested as an exact contiguous substring of its export chapter; all 221 files matched byte-for-byte, which subsumes AC wording, table row counts, edge-case rows, exact error strings and section completeness. Also verified: all 25 ADR `**Alternatives:**` tables (six axes plus Choose-instead column) are present and identical in G2-architecture.md; Parts A, B, C, D, F, G3, H transclusions match their sources (including the four dependency-map sections in D, Shared Data Entities and Domain Entity Inventory in F, and Open Questions in C); COMBINED.md contains every part and chapter verbatim; E chapter openers' spec and AC counts match disk (196 specs, 2292 ACs); H index holds 2292 AC rows; design posture verified as design-agnostic (no specifications/design-system/ directory and no legacy design-system.md exist; G1 invents no tokens or styles).

| # | Severity | Export file (section) | Canonical source | Finding — canonical vs export evidence |
|---|----------|----------------------|------------------|----------------------------------------|
| 1 | warn | G1-design-layer.md, "Stated Preferences" | BRIEF.md ## Constraints | Bridge heading "## Stated Preferences (from the Brief's Constraints)" is immediately followed by the extracted "## Constraints" heading line (template calls for the section body). Content is complete; the duplicate heading is a navigation nit. |
| 2 | warn | F-data-model.md, "Entities Shared Across Features" | feature-dependency-map.md ## Shared Data Entities | Bridge heading "## Entities Shared Across Features" is immediately followed by the extracted "## Shared Data Entities" heading. No content lost; navigation nit. |
| 3 | warn | E chapter openers, FEAT-10..18, 24, 25 (line 3) | n/a (authored glue) | Grammar: "a Important-tier feature" (11 chapters). No content impact. |

## 3. Verdict

**PASS** — All 196 specs (25 of 25 features, every Core feature included) render byte-identical to the canonical package, alternatives render at equal depth, the design-agnostic posture is correct, and glue prose only orients and counts; only three no-loss navigation warnings were found.
