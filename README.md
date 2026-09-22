# napkin-to-blueprint — showcase

Real product ideas, run end-to-end through [napkin-to-blueprint](https://github.com/tsmztech/napkin-to-blueprint)
(**n2b**), published exactly as the pipeline produced them. Nothing under any `.n2b/` folder is hand-edited.

n2b turns a raw idea into an investment-ready product blueprint inside your coding agent: a validated brief,
a full product definition, per-feature specifications, a recommended architecture with a database schema,
and export packs that Lovable, Bolt, v0, Replit, Jira, or a dev team can build from.
It deliberately does **not** build the product — the blueprint is the deliverable.

Website: [napkintoblueprint.com](https://napkintoblueprint.com) · Install: `npx napkin-to-blueprint@latest`

## How to read a showcase

Every top-level folder is one run: a plain n2b project, set up exactly as any user would. It holds the
**napkin** — the plain-language write-up a founder typed in, verbatim — and the `.n2b/` output the pipeline
produced from it. When the same napkin is re-run on another runtime (Claude Code, Codex, OpenCode, Cursor)
or model profile, it gets a sibling folder named `<idea>--<runtime>`. Same napkin, different brain.

```
<idea>/
  README.md          the idea, this run's facts, links to sibling runs
  napkin.md          what the human typed
  .n2b/
    BRIEF.md         Stage 1 — validated brief
    features/        Stage 2 — product definition (7 docs)
    specifications/  Stage 3 — one spec per feature
    architecture/    Stage 4 — architecture docs + DB schema
    exports/         Stage 5 — dev-brief + one build-tool pack (others omitted for repo size)
    tracking/        pipeline state, gates, timings
```

## Index

| Run | Idea | Runtime | Model profile | Stages done | Features | Specs |
|---|---|---|---|---|---|---|
| [chairtime](chairtime/) | Deposit-first booking for solo beauty & wellness pros | Claude Code | balanced (claude-aliases) | not started | — | — |

## License

Blueprint content is published under MIT, same as n2b. Product ideas are illustrative; build them if you like.
