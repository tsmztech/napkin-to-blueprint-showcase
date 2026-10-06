---
document_type: fidelity-report
target: lovable-pack
checked_at: 2026-10-02T16:32:21Z
package_version: 4
samples_checked: 119
verdict: pass
---

# Export Fidelity Report — lovable-pack

## 1. Reconciliation Summary (4a — appended by the workflow)

Bash reconciliation per `fidelity-rules.md`, executed by the export workflow. The final
attempt's results are recorded here by the workflow; the fidelity checker never fills this
section.

| Rule | Expected | Found | Result |
|------|----------|-------|--------|
| R0 canonical cross-check | 219 spec files on disk = technical-profile.md §6 count | 219 = 219 | pass |
| U1 FEAT coverage | all 30 FEAT IDs present | 30/30 present, 0 missing | pass |
| U2 SPEC coverage | all 219 SPEC IDs present | 219/219 present, 0 missing | pass |
| U3 AC count | ≥ 2996 distinct AC IDs | 2996 | pass |
| U4 cross-cutting rosters | 29 XBR + 30 ADR + 22 SC + 35 ASMP present | 116/116 present, 0 missing | pass |
| U5 no introduced placeholders | export TBD/TODO ≤ 0 canonical (AGENTS.md, PROMPTS.md excluded) | 0 ≤ 0 | pass |
| U6 regression lint | 0 banned-phrase hits | 0 | pass |
| VP-1 file completeness | README.md, KNOWLEDGE.md, PROMPTS.md, AGENTS.md non-empty | 4/4 present | pass |
| VP-2 knowledge budget + pointers | ≤ 10,000 bytes; ≥ 1 bullet; 0 unreferenced bullets | 8,584 bytes; 90 bullets; 0 unreferenced | pass |
| VP-3 prompt-sequence coverage | Prompt 0 + 30 per-feature headers, set = roster, no duplicates | Prompt 0 present; 30/30 headers; 0 duplicates; 0 difference | pass |
| VP-4 AC attribution + DoD counts | 0 foreign SPEC/AC IDs per prompt; per-feature SPEC and DoD sets = canonical; totals sum to 2996 | 0 bleed; 30/30 features reconciled; DoD sum 2996 | pass |
| VP-5 scope + posture surfaces | 22 SC IDs in KNOWLEDGE.md and AGENTS.md; binding + design-posture lines | 22/22 + 22/22; both lines present | pass |
| VP-6 blueprint-copy byte fidelity | 265 inventory files byte-identical, 265 files copied | 265/265 identical, 265 copied | pass |
| VP-7 no secrets / no leaked settings | 0 credential-shaped tokens; .lovable/plan.md and .bolt/config.json absent | 0; both absent | pass |
| VP-8 mutual-disclosure symmetry | disclosed ↔ pairs = canonical mutual pairs | 3 = 3 (FEAT-03↔21, FEAT-05↔06, FEAT-05↔07) | pass |

Attempt 1 of 3 passed (this run; resumed after an earlier interrupted run under n2b 0.4.0 whose two attempts failed 4b on the Purpose-verbatim vs VP-4 conflict since resolved in 0.4.1).

## 2. Semantic Findings (4b)

**Sample:** FEAT-01 (Core), FEAT-02 (Core), FEAT-03 (Core), FEAT-04 (Core), FEAT-05 (Core), FEAT-06 (Core), FEAT-07 (Core), FEAT-08 (Core), FEAT-09 (Core), FEAT-10 (Core), FEAT-11 (Core), FEAT-12 (Core), FEAT-28 (Core), FEAT-30 (Core), FEAT-16 (Important), FEAT-19 (Important), FEAT-23 (Nice-to-Have), FEAT-24 (Nice-to-Have) — 18 features / 119 specs compared (all 14 Core-tier features plus 4 non-Core).

Contract applied: lovable-pack is a distilled-render + verbatim-copy target. Acceptance criteria, tables, edge cases and error messages are not transcluded into PROMPTS.md; they live in the byte-identical docs/blueprint/ copy. The comparison therefore checks (a) the copy of every sampled spec, (b) each sampled spec's digest Purpose, definition-of-done line (count, first/last AC ID, path) and per-feature totals in PROMPTS.md, and (c) the authored layers. No AC text appears in PROMPTS.md, so there were no exemplar ACs to compare.

Checks performed (fresh from disk):
- docs/blueprint/: all 265 MANIFEST inventory rows present, none extra, `cmp` byte-identical to canonical (covers every AC, table row, edge case and error string of every sampled spec, plus the architecture ADR alternatives tables).
- PROMPTS.md: the pinned parse-back (`check` mode) prints PARSEBACK OK 30 2996. All 219 spec-digest bullets (all 119 sampled specs included) carry the canonical Purpose byte-for-byte and the canonical name and type. All 219 definition-of-done lines equal the canonical AC count, first/last AC ID and copied-spec path, and the cited totals sum to 2996. Dependency lines for all 30 features match the dependency map (forced breaks disclosed; the glyph appears only on mutual edges).
- AGENTS.md: the `## Explicit Exclusions` section (SC-01..SC-22 with rationale) is byte-identical to scope-boundaries.md; framing, conventions and file map match the template.
- Alternatives at equal depth: the architecture is carried through the byte-identical copy of architecture/technical-architecture.md (nothing is summarised); KNOWLEDGE.md and AGENTS.md state that the recommendation is binding and that alternatives are informational only, expressing no preference among alternatives.
- Design posture: neither specifications/design-system/ nor specifications/design-system.md exists on disk (design-agnostic posture). KNOWLEDGE.md and AGENTS.md section 4 state the design-agnostic posture and point to the brief's Constraints; no tokens or styles are invented.
- KNOWLEDGE.md: 8,584 characters (within the 10,000 limit), no open-item placeholder tokens, and every bullet carries an ID or an [S:] tag. ADR choices, XBR rules, ASMP-26, the SC roster and the glossary were spot-checked against the canonical architecture, dependency map and scope boundaries; no invented product or technical claim was found.
- Glue prose (README, PROMPTS intro, Prompt 0, feature summaries, architecture-rules lines) orients only; the Prompt 0 order matches the computed build order. The README counts (30 / 219 / 2996, version 4) match the package.
- Prior-run stale report: the earlier fail verdict was against a different render. This report describes the current render only.

| # | Severity | Export file (section) | Canonical source | Finding — canonical vs export evidence |
|---|----------|----------------------|------------------|----------------------------------------|
| 1 | warn | KNOWLEDGE.md (Domain glossary, "Deposit rule") | specifications/feature-dependency-map.md (FEAT-01 deposit_rule field) vs features/product-features.md (FEAT-01) | Source tag nit, no content lost. Export: "per-service fixed amount or 1–100% of price ... [S: features/product-features.md]". The "1–100%" bound is stated in feature-dependency-map.md ("fixed amount no greater than the price, or 1–100% of price"), while product-features.md says only "fixed amount or percentage". The claim is canonical but its tag points at the looser file. |
| 2 | warn | KNOWLEDGE.md (Domain glossary, "Slot hold") | features/product-features.md (FEAT-03 alternate flow: "the first to complete payment wins it") | Source tag nit, no content lost. Export tags the claim [S: specifications/feature-dependency-map.md]. The wording matches product-features.md, not that file. |

## 3. Verdict

**PASS** — all 119 sampled specs (every Core feature plus four non-Core) reproduce faithfully: the blueprint copy is byte-identical, every digest Purpose and definition-of-done value matches canonical, the posture is correct for the on-disk design-agnostic package, and the glue prose adds navigation only. Only two warn-level source-tag nits remain, and neither loses content.
