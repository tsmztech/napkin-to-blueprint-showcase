# Playbook: Build the Next Feature

## Overview

Build the next unbuilt feature-spec item of Plateful from this blueprint workspace:
pick it from `feature_list.json`, implement it per its specification under
`docs/blueprint/`, verify its acceptance criteria, and deliver.

## Procedure

1. Read `PROGRESS.md` and the recent git history to see where the build stands.
2. Open `feature_list.json` and select the FIRST item whose `"passes"` is `false` (the
   list is already in build order — see `BUILD-ORDER.md`). If the user named a specific
   item, use that one instead.
3. Read the item's `spec_file` in full, plus the architecture sections it touches.
4. Implement the item per its specification and `OPERATING-RULES.md`.
5. Verify every ID in the item's `ac_ids` end-to-end, as a user would exercise it.
6. Append a `PROGRESS.md` entry (date, item id, what was done, verification evidence).
7. Flip the item's `"passes"` to `true` — only after step 5 passed.
8. Commit and deliver a PR with a descriptive summary naming the item id and its AC IDs.

If no code exists yet, first scaffold the project per
`docs/blueprint/architecture/technical-architecture.md` (the RECOMMENDED choices), create
`init.sh`, and commit the baseline.

## Specifications

- Every ID in the item's `ac_ids` verified end-to-end
- `PROGRESS.md` appended with verification evidence
- The item's `"passes"` flipped to `true` in `feature_list.json`
- A PR delivered containing the implementation and the two file updates above

## Advice

- The spec is the contract — read it fully before writing code; its edge cases and error
  messages are requirements, not suggestions.
- Keep the session small: one `feature_list.json` item (≤ ~3 engineer-hours). A larger
  scope means stopping and asking the user to split the work.

## Forbidden Actions

- Never choose a documented architecture alternative over the recommendation
- Never implement anything on the `OPERATING-RULES.md` DO-NOT-BUILD list
- Never edit anything under `docs/blueprint/`
- Never edit `feature_list.json` item `id`s, `ac_ids`, or descriptions, and never flip
  `"passes"` without end-to-end verification

## Required from User

- Access to this repository
- Which item to build, if not the next `"passes": false` one
