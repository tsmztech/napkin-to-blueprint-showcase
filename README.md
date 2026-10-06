<p align="center">
  <img src="https://raw.githubusercontent.com/tsmztech/napkin-to-blueprint/main/assets/n2b-banner.svg" alt="napkin-to-blueprint: an idea on a napkin becomes a product blueprint" width="100%">
</p>

# napkin-to-blueprint showcase

**Real product ideas, run end to end. Published exactly as the pipeline wrote them.**

Each run here starts with a napkin: a founder's plain-language write-up of an app idea, usually about a page
long. [napkin-to-blueprint](https://github.com/tsmztech/napkin-to-blueprint) (**n2b**) turns it into a full
product blueprint: a validated brief, a product definition, a spec for every feature, an architecture with a
database schema, and export packs you can hand to Lovable, a coding agent, Spec Kit or a dev team.

**Vibe code the build. Blueprint the product first.** The prompt you type into a build tool is only as good as
what you know about your product. Each run shows what "knowing your product" looks like for an app that seems
simple and isn't.

Nothing under any `.n2b/` folder is edited by a human. New runs are added over time: other ideas, other
runtimes, other model profiles.

## All runs

| Run | The idea | Runtime · profile | Features | Specs | Acceptance criteria | Build it with |
|---|---|---|---|---|---|---|
| [Chairtime](chairtime/) | Booking link + card deposits for solo beauty pros | Claude Code · balanced | 30 | 219 | 2,996 | [lovable-pack](chairtime/.n2b/exports/lovable-pack/README.md) |
| [Plateful](plateful/) | Family meal planner with AI suggestions | Claude Code · balanced | 25 | 196 | 2,292 | [agent-workspace](plateful/.n2b/exports/agent-workspace/README.md) |
| [Clientroom](clientroom/) | Client portal + invoicing for freelancers | Claude Code · balanced | 33 | 220 | 3,102 | [speckit](clientroom/.n2b/exports/speckit/README.md) |

Every run also has a `dev-brief` export for human teams. Every acceptance criterion is carried word for word
from spec to export, and a fidelity gate checks that before any export is marked done.

<!-- Adding a run: append one row above (keep the columns), then one card below, then the run's own README.md. -->

## Build one

Everything you need to build a run is in one folder: `<run>/.n2b/exports/<pack>/`. Open that pack's README
(the links in the table above). It says what each file is for and the order to use them in. Each pack carries
its own copy of the blueprint under `docs/blueprint/`, so you don't need anything else from the run.

The rest of the run folder (napkin, brief, features, specs, architecture, run log) is there to read: it shows
how the blueprint was made and what it decided.

## The runs

### [Chairtime](chairtime/): "a booking link that takes a deposit"

> *"They lose real money to no-shows and last-minute cancellations, and they spend their evenings going back
> and forth in DMs to agree on a time."*

It looks like a calendar plus a form. The blueprint found the parts a first draft misses: a money flow
(deposit → balance → no-show forfeit → refund window), two-way calendar sync, opt-in for text messages,
time zones, and making sure two clients can never book the same slot.

**Start here:** [napkin](chairtime/napkin.md) ·
[brief](chairtime/.n2b/BRIEF.md) ·
[all 30 features](chairtime/.n2b/features/product-features.md) ·
[sample spec: no-show + deposit forfeit](chairtime/.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-002-no-show-marking-deposit-forfeiture.md) ·
[architecture](chairtime/.n2b/architecture/technical-architecture.md) ·
**[Lovable prompts](chairtime/.n2b/exports/lovable-pack/PROMPTS.md)**

### [Plateful](plateful/): "an AI meal planner for my family"

> *"…or 'AI meal plan' toys that suggest a walnut salad to a family with a nut allergy."*

The most-built genre among vibe coders. The blueprint made allergy safety a separate engine with
14 specs, including a fail-closed rule for missing ingredient data, so the AI is never the last line of
defence. It also covers a shared grocery list that works offline, kids' data, and keeping AI costs per
household small.

**Start here:** [napkin](plateful/napkin.md) ·
[brief](plateful/.n2b/BRIEF.md) ·
[all 25 features](plateful/.n2b/features/product-features.md) ·
[sample spec: fail-closed allergy policy](plateful/.n2b/specifications/FEAT-02-dietary-rules-allergy-safety-engine/FEAT-02.SPEC-007-ingredient-data-completeness-and-fail-closed-policy.md) ·
[architecture](plateful/.n2b/architecture/technical-architecture.md) ·
**[coding-agent workspace](plateful/.n2b/exports/agent-workspace/README.md)**

### [Clientroom](clientroom/): "a client portal so I stop chasing invoices"

> *"…when a client disputes scope there's no record of what they signed off."*

The "I tried to build this in Notion" classic. The blueprint added an activity log that can't be edited after
the fact (the freelancer's evidence in a scope dispute), contact roles for the client side, tax and currency
per country, files over 1 GB, and payments that never touch the platform's own account.

**Start here:** [napkin](clientroom/napkin.md) ·
[brief](clientroom/.n2b/BRIEF.md) ·
[all 33 features](clientroom/.n2b/features/product-features.md) ·
[sample spec: tamper-proof approvals](clientroom/.n2b/specifications/FEAT-13-immutable-activity-audit-trail/FEAT-13.SPEC-004-entry-immutability-content-attribution-rules.md) ·
[architecture](clientroom/.n2b/architecture/technical-architecture.md) ·
**[Spec Kit export](clientroom/.n2b/exports/speckit/README.md)**

## How to read a run

Each folder is a normal n2b project, the same as one you would get on your own machine.

```
<idea>/
  README.md          the idea, run facts, where to start
  napkin.md          what the founder typed, verbatim
  session-notes.md   the Stage 1 interview: every question n2b asked and the answer given
  ANSWERS.md         the founder answer key used for that interview (see "How these were made")
  RUN-LOG.md         every pipeline step: gates passed, retries, deviations, timings
  .n2b/
    BRIEF.md         Stage 1  validated brief
    features/        Stage 2  product definition (7 docs)
    specifications/  Stage 3  one folder per feature, one file per spec
    architecture/    Stage 4  5 architecture docs incl. database schema
    exports/         Stage 5  build-ready packs + FIDELITY-REPORT + EXPORT-RECEIPT
    tracking/        pipeline state, gate results, artifact fingerprints
```

`.n2b/` is a dot-folder. On GitHub it shows normally. After cloning, use `ls -a`.

## How runs are made

- **Napkin in, blueprint out.** Every run starts from the napkin in its folder and goes through all n2b stages
  to at least one export. The runtime, model profile and n2b version are listed in each run's README.
- **Hands off.** The founder's interview answers are written in advance (`ANSWERS.md`) so the run can proceed
  unattended. Where they are silent, n2b records an open question instead of guessing.
- **Gates, not vibes.** Every stage passes completeness and fidelity checks before moving on. Every gate result
  is in `RUN-LOG.md` and `.n2b/tracking/`.
- **Honest record.** Each run README lists its deviations as logged, including any time a gate refused to
  ship an export. Those refusals stay in the repo. See
  [Clientroom's "gate that said no"](clientroom/README.md#the-gate-that-said-no) for an example.

## Run your own

```bash
npx napkin-to-blueprint@latest
```

Then, in Claude Code, run `/n2b:s1-init` and paste your napkin. n2b also installs into Codex, OpenCode and Cursor.
See the [n2b README](https://github.com/tsmztech/napkin-to-blueprint) and
[napkintoblueprint.com](https://napkintoblueprint.com).

If a run here helped you build something, a ⭐ on [n2b](https://github.com/tsmztech/napkin-to-blueprint)
helps other founders find it.

## License

MIT, same as n2b. The product ideas are illustrative. Build them if you like.
