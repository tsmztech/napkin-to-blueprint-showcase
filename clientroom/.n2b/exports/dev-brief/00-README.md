# Clientroom — Development Brief

> Rendered from the Clientroom blueprint package, version 4, on 2026-09-29.
> This brief reorganizes the package for human readers; it adds navigation, never content.
> For print or PDF, use `COMBINED.md` — the same parts in one file.

## Executive Summary

Clientroom is a branded client portal for solo freelancers — designers, developers, copywriters and video editors — that puts proposals, milestones, deliverables with approvals, and invoices behind one link per client project. It serves two audiences: the freelancer, who sees every client, project and payment in one place with an unalterable record of what was agreed, approved and billed, and the client contacts, who sign in by magic link, review work on their phones, and pay. This package contains 33 features specified in 220 specifications carrying 3102 acceptance criteria, a recommended architecture with documented alternatives, and a complete database schema.

## Package Map

| Part | File | What it contains | Built from |
|------|------|------------------|-----------|
| 00 | 00-README.md | This guide | authored |
| A | A-product-vision.md | Vision, problem, brief, persona set + access matrix | BRIEF.md · user-persona.md |
| B | B-usage-and-success.md | User journeys, success metrics | user-journeys.md · success-metrics.md |
| C | C-scope-and-assumptions.md | Exclusions (SC), assumptions (ASMP), NFRs, open items | scope-boundaries.md · assumptions-constraints.md · BRIEF Open Questions |
| D | D-feature-catalog.md | Full feature catalog (FEAT), dependencies, cross-feature rules (XBR) | product-features.md · feature-dependency-map.md |
| E | E-feature-specifications/ | One chapter per feature: breakdown brief + every spec, verbatim | specifications/FEAT-NN-{slug}/ |
| F | F-data-model.md | Domain entities, shared data entities | product-features.md · feature-dependency-map.md |
| G1 | G1-design-layer.md | Design posture (design-agnostic; stated preferences from the brief) | BRIEF Constraints |
| G2 | G2-architecture.md | Technical profile, technology landscape, feasibility, recommended architecture + alternatives | architecture/ (4 documents) |
| G3 | G3-database-schema.md | Complete database schema | database-schema.md |
| H | H-appendices.md | Market research, consolidated acceptance-criteria index | market-research.md · generated index |

## Reading Order by Role

- **Product manager / project lead:** 00 → A → C → D → B — the product, its boundaries, and its open items before anything technical.
- **Designer:** A (persona set + access matrix) → G1 (design posture; the team owns visual design) → B (journeys) → E chapters for the screen specs of the features being designed.
- **Backend engineer:** A → D → F → G3 → G2 — then the E chapters in dependency order (Part D's Features table carries Depends On).
- **Frontend engineer:** A → B → G1 → G2 (architecture §5 structure + §7 routes) → E chapters, screen specs first.
- **QA:** C (what is out of scope) → E chapters' Acceptance Criteria sections → H (consolidated AC index — every criterion in one table, with source pointers).

## What To Do First

1. **Resolve the open items.** Part C ends with the brief's open questions — the decisions the team owns before or during early build.
2. **Confirm the architecture recommendation.** Part G2 records a recommended option plus documented alternatives for every decision area, with `Choose instead when` conditions. The recommendation is the default; the alternatives exist so this team can adapt choices to its own constraints — confirm or substitute deliberately, per area.
3. **Pick your tooling exports.** This brief is one rendering of the package. The same package exports to other consumers (tracker backlogs, AI coding agents) — `/n2b:s5-export` lists what is available.
