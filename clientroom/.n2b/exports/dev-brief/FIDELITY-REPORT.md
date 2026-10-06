---
document_type: fidelity-report
target: dev-brief
checked_at: 2026-09-29T00:00:00Z
package_version: 4
samples_checked: 220
verdict: pass
---

# Export Fidelity Report — dev-brief

## 1. Reconciliation Summary (4a — appended by the workflow)

Bash reconciliation per `fidelity-rules.md`, executed by the export workflow. The final
attempt's results are recorded here by the workflow; the fidelity checker never fills this
section.

| Rule | Expected | Found | Result |
|------|----------|-------|--------|
| R0 canonical spec cross-check | 220 spec files on disk = technical-profile.md §6 | 220 = 220 | pass |
| U1 FEAT coverage | all 33 FEAT IDs present | 33/33 | pass |
| U2 SPEC coverage | all 220 SPEC IDs present | 220/220 | pass |
| U3 AC count | ≥ 3102 distinct AC IDs | 3102 | pass |
| U4 Cross-cutting rosters | 35 XBR + 31 ADR + 25 SC + 32 ASMP present | 123/123 | pass |
| U5 No introduced placeholders | ≤ 0 TBD/TODO (canonical count; COMBINED.md excluded) | 0 | pass |
| U6 Regression lint | 0 banned-phrase occurrences | 0 | pass |
| DEV-1 Part-file completeness | 11 C-28 part files exist and non-empty | 11/11 | pass |
| DEV-2 E chapter per feature | 33 chapters under E-feature-specifications/ | 33/33 | pass |
| DEV-3 COMBINED.md completeness | first heading of all 43 parts (10 parts + 33 chapters) in COMBINED.md | 43/43 | pass |

Attempt 1 of 3 passed — no re-renders.

## 2. Semantic Findings (4b)

**Sample:** All 17 Core features (FEAT-01 through FEAT-16, FEAT-32) plus the non-Core features FEAT-17, FEAT-18, FEAT-19, FEAT-20, FEAT-21, FEAT-22, FEAT-23, FEAT-24, FEAT-25, FEAT-26, FEAT-27, FEAT-28, FEAT-29, FEAT-30, FEAT-31, FEAT-33 — 33 features / 220 specs compared. The sample was widened to every feature because each canonical file could be compared mechanically: every spec file and every feature-overview (253 files) was frontmatter-stripped, whitespace-normalized and searched as a contiguous block inside its E chapter, and all 253 were found. That covers every AC body, table row, edge-case entry, exact error message and `##` section in the sampled specs.

Whole-export checks performed:
- **Alternatives at equal depth:** `technical-architecture.md` and `G2-architecture.md` each carry 22 `**Alternatives:**` tables. The architecture document is present as a contiguous block, and the sampled tables carry all six axes (Cost Profile, Operational Complexity, Scale Ceiling, Lock-in, Team Skill Demand, Choose instead when).
- **Design posture:** `specifications/design-system/` and `specifications/design-system.md` are both absent on disk, so Posture 3 (design-agnostic) applies. `G1-design-layer.md` states the design-agnostic posture and carries the BRIEF `## Constraints` body. It invents no tokens or styles.
- **COMBINED.md:** rebuilt independently from 00, A–E, F, G1–G3 and H in template order. It is byte-identical (5,772,636 bytes).
- **Part transclusions:** BRIEF, user-persona, user-journeys, success-metrics, scope-boundaries, assumptions-constraints, product-features, the four architecture documents, database-schema and market-research are each present verbatim in their part. The dependency-map sections (Features, Navigation Connections, Cross-Feature Business Rules, External Touchpoints, Shared Data Entities) and the Domain Entity Inventory are verbatim in D and F. The BRIEF Open Questions are verbatim in C.
- **Glue prose:** the 00-README, part intros, bridges, C framing sentence, G1 statement and G2 intro were read. The counts (33 features, 220 specs, 3102 ACs) match `ls` and `grep -c` over the package. The 33 chapter-opener tables match spec frontmatter (id, name, type, AC count) row for row. The per-chapter "K specifications carrying M acceptance criteria" figures are correct. H's AC index has 3,102 rows across 220 spec groups. No invented product or technical claims, no resolved open questions, no stated preference among alternatives.

| # | Severity | Export file (section) | Canonical source | Finding — canonical vs export evidence |
|---|----------|----------------------|------------------|----------------------------------------|
| 1 | warn | E-feature-specifications/*.md, chapter opener paragraph (11 chapters with Important-tier features, e.g. FEAT-17-deliverable-version-history.md, FEAT-22-accounting-export.md) | export template chapter-opener skeleton; feature-overview.md `priority_tier` | Grammar nit in the authored opener with no content lost. Canonical tier is "Important". Export text: "This chapter covers Deliverable Version History, a Important-tier feature." It should read "an Important-tier feature". |

## 3. Verdict

**PASS** — All 220 specs and 33 feature overviews render byte-for-byte from the canonical package. The architecture alternatives tables render at full depth, the design-agnostic posture is correct, COMBINED.md is identical to its parts, and the authored glue only navigates and counts correctly. The only finding is one warn-level grammar nit in the authored chapter openers.
