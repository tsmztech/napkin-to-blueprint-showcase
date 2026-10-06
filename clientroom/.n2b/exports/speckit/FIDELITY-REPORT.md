---
document_type: fidelity-report
target: speckit
checked_at: 2026-10-02T17:09:37Z
package_version: 4
samples_checked: 220
verdict: pass
---

# Export Fidelity Report — speckit

## 1. Reconciliation Summary (4a — appended by the workflow)

Bash reconciliation per `fidelity-rules.md`, executed by the export workflow. The final
attempt's results are recorded here by the workflow; the fidelity checker never fills this
section.

| Rule | Expected | Found | Result |
|------|----------|-------|--------|
| R0 canonical spec cross-check | 220 spec files = technical-profile.md §6 | 220 = 220 | pass |
| U1 FEAT coverage | all 33 FEAT IDs | 33/33 present | pass |
| U2 SPEC coverage | all 220 SPEC IDs | 220/220 present | pass |
| U3 AC count | ≥ 3102 distinct AC IDs | 3102 | pass |
| U4 cross-cutting rosters | 35 XBR + 31 ADR + 25 SC + 32 ASMP | all present | pass |
| U5 no introduced placeholders | ≤ 0 TBD/TODO (canonical 0; excl. constitution.md) | 0 | pass |
| U6 regression lint | 0 banned-phrase hits | 0 | pass |
| SK-1 workspace files | README.md, constitution.md, feature.json non-empty | all 3 present | pass |
| SK-2 feature-dir coverage | 33 dirs, NNN 001..033, 33 unique Blueprint-feature headers | 33 dirs, exact set, 0 dups | pass |
| SK-3 AC verbatim coverage (render-side) | 3102 distinct AC IDs, set-equal | 3102, set-equal | pass |
| SK-4 per-spec mapping | 220 SPEC IDs in owning spec.md | 220/220 | pass |
| SK-5 blueprint-copy byte fidelity | 269 inventory files byte-identical | 269 copied, 0 cmp diffs | pass |
| SK-6 constitution + feature.json | 25 SC IDs + footer; feature_directory → 001-* | 25/25, footer present, specs/001-client-project-management | pass |
| SK-7 mutual-disclosure symmetry | 2 canonical mutual pairs | 2 disclosed, symmetric | pass |

Attempt 1 of 3 passed.

## 2. Semantic Findings (4b)

**Sample:** all 17 Core features (FEAT-01, 02, 03, 04, 05, 06, 07, 08, 09, 10, 11, 12, 13, 14, 15, 16, 32; 122 specs) plus all 16 non-Core features (FEAT-17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 33; 98 specs) — 33 features / 220 specs compared. The verbatim-bearing parts were checked for every spec by script. The authored condensation layers were read in full against canonical text for FEAT-01 and FEAT-10 (Core) and FEAT-30 (non-Core). The other features' authored layers were checked by structure, ID, metric-attribution, entity and ADR cross-checks, not line-by-line prose reading.

Checks performed (all read-only, canonical files read fresh from disk):

- **ACs verbatim:** all 3,102 canonical AC-definition lines appear as exact full-line matches in the matching spec.md. 0 missing, 0 altered. Every spec's `**Purpose:**` line appears verbatim, and every SPEC has an FR entry plus an Edge Cases digest bullet with a `Source:` pointer. All 220 pointer paths resolve.
- **Mechanical sections independently re-derived:** the template's pinned build script was re-run against the canonical package into scratch space. For all 33 features, the rendered spec.md starts with the script's `01-header-stories.md` byte-for-byte and contains `02-requirements.md` byte-for-byte. The ORDER table in README.md matches the script's ORDER rows, and the three disclosed cycle breaks (FEAT-09 FORWARD, FEAT-13 MUTUAL, FEAT-14 MUTUAL) match its BROKEN rows. `.specify/feature.json` names `specs/001-client-project-management`, which exists.
- **Tables and edge cases:** per the template, spec.md carries condensed Edge Cases digests, not tables. Canonical tables, edge-case rows and exact error strings live in `docs/blueprint/`. All 269 MANIFEST inventory files were compared with `cmp` against `docs/blueprint/{rel}`. 0 differ, and the copied-file count is 269, equal to INV_COUNT.
- **Edge-case digests (FEAT-01 all 11 specs, FEAT-10 all 7, FEAT-30 all 5):** every digest is a faithful condensation of the canonical Edge Cases. No condition or behaviour was invented and the pointers are intact.
- **Measurable Outcomes attribution:** for all 22 cited metric references, the metric's `**Connected Feature:**` resolves to the rendered feature. Target values match the canonical metric Targets, and the cited target text is the canonical text or a faithful trim of it. Features with no connected metric (e.g. FEAT-30) say so and ground the outcome in Signals alone. All 21 canonical metrics are cited in exactly the features they connect to. Every ASMP ID cited exists in `assumptions-constraints.md`.
- **research.md (all 33):** every ADR ID cited exists in the `## 14. Decision Log`. Every "Decided stack" line and every feature-specific ADR line carries the register's decision text verbatim, with the correct category. Every entity named under "Data model" exists in `database-schema.md`.
- **Constitution:** the `## Explicit Exclusions` block (SC-01..SC-25, rationales included) is identical to the canonical extraction (`diff` clean). The footer reads `Version: 1.0.0 | Ratified: 2026-10-02 | Last Amended: 2026-10-02`. Design posture IV states the design-agnostic branch.
- **Design posture:** neither `specifications/design-system/` nor `specifications/design-system.md` exists in the package, so the design-agnostic posture is the correct branch. The export states it and invents no tokens or styles.
- **Alternatives at equal depth:** the speckit target does not re-render architecture. `technical-architecture.md` is carried byte-identical in `docs/blueprint/architecture/`, so its full alternatives tables are preserved at canonical depth. research.md and the constitution state the recommendation as binding and point to the full document. This matches the template contract.
- **Glue prose:** README.md and the constitution frames follow the template skeleton. Counts (33 / 220 / 3102 / version 4) agree with canonical and script totals. No new product or technical claim, open-question resolution or scope softening was found. Zero TBD/TODO occurrences were introduced in authored files.

No semantic findings. All 220 specs render their acceptance criteria verbatim; the blueprint copy preserves tables at full row counts, edge cases and exact error messages byte-identically; alternatives render at equal depth through the byte-identical architecture copy; the design posture was verified as design-agnostic (no design system on disk); and glue prose adds navigation only.

## 3. Verdict

**PASS** — all 3,102 acceptance criteria, all 220 Purpose lines, every header/story/FR fragment and all 269 blueprint files reproduce the canonical package exactly. The authored layers sampled (Edge Cases digests, Measurable Outcomes, Assumptions, research.md, constitution, README) add no invented claims and misattribute no metric.
