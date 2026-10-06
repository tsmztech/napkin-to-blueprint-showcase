@AGENTS.md

## Claude Code notes

- AGENTS.md above is the complete instruction set; this file only imports it.
- Blueprint paths in AGENTS.md are written plainly or backticked on purpose — nothing
  force-loads into context. Read blueprint files with the Read tool as each item needs.
- `PROGRESS.md` is the cross-session memory: read it at session start, append at session
  end, one `feature_list.json` item per session.
