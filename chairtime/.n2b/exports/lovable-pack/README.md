# Chairtime — Lovable Pack

This directory is a prompt-ready build pack rendered from the Chairtime blueprint package (version 4, rendered 2026-10-02) — 30 features, 219 specifications, 2996 acceptance criteria — for building in Lovable: KNOWLEDGE.md (the distilled project knowledge), PROMPTS.md (a Plan-mode seed plus one build prompt per feature, in dependency order), AGENTS.md (the build constitution with the full DO-NOT-BUILD list), and the complete blueprint verbatim under docs/blueprint/ for reference.

## Set up Lovable

Lovable has no repo or ZIP import (it exports TO GitHub, not from it), so the pack loads by paste:

1. Open your Lovable project → Settings → Knowledge, and paste the full contents of KNOWLEDGE.md into Project Knowledge. The field caps at 10,000 characters; KNOWLEDGE.md is sized to fit with headroom. (Workspace Knowledge is a separate 10k field for owners/admins; if both are set, Project Knowledge wins on conflict.)
2. Keep this pack's folder open beside Lovable — PROMPTS.md is your build script and docs/blueprint/ is your reference when a prompt needs more depth.

## Plan, then build

1. Switch to Plan mode (flat 1 credit per message, writes no code) and paste Prompt 0 from PROMPTS.md. Review the plan it produces.
2. Switch to default mode and feed the feature prompts ONE AT A TIME, in order. Never paste several at once — bulk input degrades output and burns credits.

## Connect GitHub, then commit AGENTS.md

When Lovable has created and connected its GitHub repo, commit this pack's AGENTS.md at the repo root — Lovable always reads a root AGENTS.md once the repo is connected, making the constitution persistent without spending Knowledge budget. Commit docs/blueprint/ alongside it as the in-repo reference.

## Gotchas

- KNOWLEDGE.md and AGENTS.md carry the same rules on purpose — Knowledge covers you before the repo exists; AGENTS.md takes over after.
- Everything under docs/blueprint/ is read-only reference — regenerate upstream if it needs to change.
