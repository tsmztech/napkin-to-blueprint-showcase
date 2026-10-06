# Plateful — Agent Build Workspace

This directory is a build-ready workspace rendered from the Plateful blueprint package
(version 4, rendered 2026-09-29): the complete blueprint — 25 features, 196
specifications, 2292 acceptance criteria — verbatim under docs/blueprint/, plus the
harness files a coding agent needs to build it across many sessions. It contains no code
yet; the first build session scaffolds the project.

## Make it a repository

```
git init && git add -A && git commit -m "Plateful blueprint workspace"
```

## Start your coding agent

**Claude Code** — open the repository and start a session: Claude Code reads `CLAUDE.md`,
which imports `AGENTS.md`. Ask it to build the next feature.

**Cursor / GitHub Copilot / Windsurf (Devin Desktop)** — open the repository: each reads
the root `AGENTS.md` natively. Ask it to follow the session protocol.

**Devin (cloud)** — connect the repository: `AGENTS.md` is auto-ingested into Devin's
Knowledge, and the wiki indexes `docs/blueprint/` (steered by `.devin/wiki.json`). Attach
`playbooks/build-next-feature.devin.md` at session start, or save it under Settings &
Library → Playbooks. Run small sessions — one `feature_list.json` item each, sized ≤ ~3
engineer-hours.

## What's in the root

| File | What it is |
|------|-----------|
| `AGENTS.md` | Canonical agent instructions — every major coding agent reads it |
| `CLAUDE.md` | Claude Code bridge — imports AGENTS.md |
| `OPERATING-RULES.md` | Binding rules: architecture authority, DO-NOT-BUILD list, design posture |
| `BUILD-ORDER.md` | Dependency-ordered feature sequence, cycle breaks disclosed |
| `feature_list.json` | Machine-checkable work list — every item starts `"passes": false` |
| `PROGRESS.md` | Append-only session log — the build's cross-session memory |
| `playbooks/build-next-feature.devin.md` | Devin playbook for one build session |
| `docs/blueprint/` | The complete blueprint package, byte-identical |
