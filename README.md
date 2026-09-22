# napkin-to-blueprint — showcase

Real product ideas, run end-to-end through [napkin-to-blueprint](https://github.com/tsmztech/napkin-to-blueprint)
(**n2b**), published exactly as the pipeline produced them. Nothing under any `.n2b/` folder is hand-edited.

n2b turns a raw idea into an investment-ready product blueprint inside your coding agent: a validated brief,
a full product definition, per-feature specifications, a recommended architecture with a database schema,
and export packs that Lovable, Bolt, v0, Replit, Jira, or a dev team can build from.
It deliberately does **not** build the product — the blueprint is the deliverable.

Website: [napkintoblueprint.com](https://napkintoblueprint.com) · Install: `npx napkin-to-blueprint@latest`

## How to read a showcase

Each idea folder has the **napkin** — the plain-language write-up a founder typed in, verbatim — and one
folder per run. A run is one runtime (Claude Code, Codex, OpenCode, Cursor) with one model profile.
Same napkin, different brain: compare what each produced.

```
<idea>/
  README.md          the idea and a run-by-run comparison
  napkin.md          what the human typed
  runs/<runtime>--<profile>/.n2b/
    BRIEF.md         Stage 1 — validated brief
    features/        Stage 2 — product definition (7 docs)
    specifications/  Stage 3 — one spec per feature
    architecture/    Stage 4 — architecture docs + DB schema
    exports/         Stage 5 — dev-brief + one build-tool pack (others omitted for repo size)
    tracking/        pipeline state, gates, timings
```

## Index

| Idea | Runtime | Model profile | Stages done | Features | Specs | Run |
|---|---|---|---|---|---|---|
| [Chairtime](chairtime/) — deposit-first booking for solo beauty & wellness pros | — | — | not started | — | — | — |

## License

Blueprint content is published under MIT, same as n2b. Product ideas are illustrative; build them if you like.
