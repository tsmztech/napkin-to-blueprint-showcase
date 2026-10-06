---
document_type: fidelity-report
target: agent-workspace
checked_at: 2026-09-29T00:00:00Z
package_version: 4
samples_checked: 114
verdict: pass
---

# Export Fidelity Report — agent-workspace

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
| U5 no introduced placeholders | ≤ 0 TBD/TODO (canonical 0; OPERATING-RULES.md excluded) | 0 | pass |
| U6 regression lint | 0 banned-phrase hits | 0 | pass |
| AWS-1 harness-file completeness | 9 C-30 harness files non-empty | 9 of 9 | pass |
| AWS-2 verbatim-copy byte fidelity | 237 inventory rows byte-identical; 237 files under docs/blueprint/ | 237 identical / 237 files | pass |
| AWS-3 feature_list.json integrity | parses; 196 items; 2292 distinct ac_ids; 0 non-false passes; id set = SPEC roster | parses; 196; 2292; 0; id set equal | pass |
| AWS-4 BUILD-ORDER coverage | 25 order-table rows, one per FEAT | 25 rows, 25 of 25 FEAT | pass |
| AWS-5 AGENTS.md guardrails | ≤ 12000 chars; 0 bare @-tokens; CLAUDE.md first line "@AGENTS.md" | 2776 chars; 0; "@AGENTS.md" | pass |
| AWS-6 mutual-disclosure symmetry | canonical mutual pairs: FEAT-03 ↔ FEAT-12 | disclosed: FEAT-03 ↔ FEAT-12 | pass |

Attempt 1 of 3 passed — no re-renders used.

## 2. Semantic Findings (4b)

**Sample:** FEAT-01 (Core), FEAT-02 (Core), FEAT-03 (Core), FEAT-04 (Core), FEAT-05 (Core), FEAT-06 (Core), FEAT-07 (Core), FEAT-08 (Core), FEAT-09 (Core), FEAT-23 (Core), FEAT-10 (Important), FEAT-15 (Important), FEAT-19 (Nice-to-Have) — 13 features / 114 specs compared (99 Core specs, 15 non-Core specs).

Method note: the agent-workspace contract is a verbatim byte copy, so each sampled spec was compared against its canonical source with `cmp` (a stricter test than text comparison of ACs, tables, edge cases and error messages). Additionally, all 237 manifest inventory rows were `cmp`-checked (237 rows, 237 copied files, 0 differences) and no extra files exist under `docs/blueprint/`. `technical-architecture.md` is byte-identical to canonical, so every ADR's Alternatives content is carried at the same depth as the canonical source (equal depth by construction). The work list and harness files were checked separately:

- `feature_list.json`: 25 features / 196 items / 2,292 ACs. The AC count independently matches the 2,292 distinct AC-definition IDs in the canonical specs. For all 196 items, `id`, `name`, `type` and `ac_ids` match the copied spec frontmatter and AC definition lines; `passes` is false on every item.
- `BUILD-ORDER.md`: 25 rows (one per FEAT). Values match the dependency map's Features table. The single dependency cycle (FEAT-03 and FEAT-12) is disclosed as MUTUAL, and the ↔ glyph appears only on that line.
- `OPERATING-RULES.md`: the `## Explicit Exclusions` section (SC-01..SC-19, 42 non-blank lines) is verbatim from `scope-boundaries.md`, with every line matched.
- Design posture: no `specifications/design-system/` directory and no legacy `design-system.md` exist on disk. Section 5 states the design-agnostic posture (Posture 3) and invents no tokens or styles. This is the correct branch.
- Glue prose (README, AGENTS, CLAUDE, PROGRESS, BUILD-ORDER intro, playbook, wiki.json): counts (25/196/2292/237) agree with the script metadata and the package. No invented product or technical claims, no resolved open questions, and no stated preference among architecture alternatives. AGENTS.md is 2,776 characters with zero bare `@`-path tokens. CLAUDE.md's first line is `@AGENTS.md`.
- No `COMBINED`-style artifact exists for this target (not applicable). No per-tool rule directories and no `init.sh` were emitted, as required.

No semantic findings. All 114 sampled specs render their acceptance criteria verbatim, tables at full row counts, edge cases and exact error messages intact; alternatives render at equal depth; design posture verified as design-agnostic (no design system supplied); glue prose adds navigation only.

## 3. Verdict

**PASS** — All 114 sampled specs across 13 features (every Core-tier feature plus three non-Core features) are byte-identical to canonical, and the script-built work list, build order, verbatim SC exclusions, design posture and authored harness prose all agree with the canonical package.
