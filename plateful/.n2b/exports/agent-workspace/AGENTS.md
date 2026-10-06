# Plateful — Agent Instructions

## What this repository is

Build the product described under `docs/blueprint/` — Plateful, a shared family meal
planner with AI weekly dinner plans, allergy-safe suggestions and a live shared grocery
list. It is a complete, implementation-ready blueprint (25 features, 196 specifications,
2292 acceptance criteria) rendered from the Plateful package. This workspace ships the
blueprint and a build harness; it ships no code yet.

## Blueprint map & reading order

| Path | What it is |
|------|-----------|
| `docs/blueprint/BRIEF.md` | The product brief — read this first |
| `docs/blueprint/features/` | Product definition: features, personas, journeys, scope, metrics, research |
| `docs/blueprint/specifications/` | Per-feature specifications — the build contract, with acceptance criteria |
| `docs/blueprint/architecture/` | Recommended architecture, feasibility, database schema |

Read the brief once, then per item: the item's spec in full, plus the architecture and
schema sections it touches. Do not bulk-load the blueprint (237 files) — read what the
current item needs.

## Ground rules

`OPERATING-RULES.md` is binding: the recommended architecture (alternatives are not yours
to choose), the DO-NOT-BUILD list, assumptions awareness, the design posture, and the
read-only blueprint rule. Specs are the contract; blueprint files are read-only inputs.

## Session protocol

Every working session runs the same loop:

1. Read `PROGRESS.md` and the recent `git log` to see where the build stands.
2. Pick the FIRST item in `feature_list.json` whose `"passes"` is `false` — the list is
   already in build order.
3. Read that item's `spec_file` in full.
4. Implement the item.
5. Verify every ID in its `ac_ids` end-to-end, the way a user would exercise it.
6. Commit with a descriptive message.
7. Append a `PROGRESS.md` entry (date, item id, what was done, verification evidence).
8. Flip that item's `"passes"` to `true` — ONLY after step 5's verification.

Work one item per session (or a small coherent group). Never edit item `id`s, `ac_ids`, or
descriptions; never mark `passes` without verification.

**First session ever:** there is no code yet. Scaffold the project per
`docs/blueprint/architecture/technical-architecture.md` (the RECOMMENDED choices), create
`init.sh` for environment startup, and commit the baseline — then start the loop.

## Build order

`BUILD-ORDER.md` explains the dependency order `feature_list.json` follows and discloses
its cycle breaks. Follow the list; do not re-derive the order.

## Definition of done

A spec is done when its acceptance criteria — by ID, per the item's `ac_ids` — pass
end-to-end. Not merely compiling, not unit tests alone: exercised as a user would.
