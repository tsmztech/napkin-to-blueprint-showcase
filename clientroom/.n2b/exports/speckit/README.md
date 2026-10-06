# Clientroom — Spec Kit Bundle

This directory is a ready-made GitHub Spec Kit workspace rendered from the Clientroom blueprint package (version 4, rendered 2026-10-02): one pre-authored feature spec per blueprint feature (33 features, 220 specifications, 3102 acceptance criteria — every criterion carried verbatim), a project constitution carrying the blueprint's scope and architecture rules, pre-seeded per-feature research, and the complete blueprint under docs/blueprint/. You skip /speckit.specify entirely and go straight to /speckit.plan.

## Build order

Features are ordered topologically — Kahn's algorithm over the blueprint dependency
map, deterministic cycle-break restricted to features inside a dependency cycle
(lowest tier rank Core < Important < Nice-to-Have, then lowest FEAT number), breaks
disclosed below. Directory prefixes match this table; work top to bottom.

| # | Feature | Name | Priority | Depends on | Specs | ACs |
|---|---------|------|----------|------------|-------|-----|
| 001 | FEAT-01 | Client & Project Management | Core | — | 11 | 114 |
| 002 | FEAT-15 | Currency & Tax Handling | Core | — | 8 | 90 |
| 003 | FEAT-02 | Proposal Creation & Sending | Core | FEAT-01, FEAT-15 | 11 | 116 |
| 004 | FEAT-03 | Proposal Acceptance | Core | FEAT-02 | 7 | 81 |
| 005 | FEAT-04 | Milestone & Payment Schedule Setup | Core | FEAT-03 | 4 | 86 |
| 006 | FEAT-16 | Large File Handling & Storage | Core | — | 7 | 84 |
| 007 | FEAT-06 | Deliverable Upload & Sharing | Core | FEAT-04, FEAT-16 | 6 | 79 |
| 008 | FEAT-07 | Deliverable Review & Feedback | Core | FEAT-06 | 8 | 119 |
| 009 | FEAT-08 | Milestone Approval | Core | FEAT-06, FEAT-07 | 7 | 105 |
| 010 | FEAT-17 | Deliverable Version History | Important | FEAT-06, FEAT-16 | 5 | 83 |
| 011 | FEAT-18 | Client Contact Management & Roles | Important | FEAT-01 | 11 | 144 |
| 012 | FEAT-05 | Client Portal Access (Magic-Link Login) | Core | FEAT-18 | 9 | 99 |
| 013 | FEAT-19 | Freelancer Branding | Important | — | 3 | 52 |
| 014 | FEAT-23 | Subscription Plan & Billing Management | Important | FEAT-01 | 8 | 164 |
| 015 | FEAT-26 | Legally Binding E-Signature for Proposals | Nice-to-Have | FEAT-03 | 5 | 81 |
| 016 | FEAT-27 | Custom Domain per Freelancer | Nice-to-Have | FEAT-05, FEAT-19 | 4 | 68 |
| 017 | FEAT-32 | Payment Account Connection | Core | — | 6 | 119 |
| 018 | FEAT-09 | Invoice Generation & Sending | Core | FEAT-01, FEAT-03, FEAT-08, FEAT-15, FEAT-21, FEAT-32 | 10 | 131 |
| 019 | FEAT-10 | Invoice Payment Processing | Core | FEAT-09, FEAT-32 | 7 | 106 |
| 020 | FEAT-11 | Automated Payment Reminders | Core | FEAT-09, FEAT-10, FEAT-15 | 5 | 66 |
| 021 | FEAT-12 | Freelancer Financial Dashboard | Core | FEAT-01, FEAT-09, FEAT-10, FEAT-15 | 4 | 68 |
| 022 | FEAT-22 | Accounting Export | Important | FEAT-09, FEAT-10 | 3 | 47 |
| 023 | FEAT-25 | Refund & Cancelled Project Handling | Important | FEAT-09, FEAT-10, FEAT-32 | 8 | 121 |
| 024 | FEAT-28 | Global Search Across Clients & Projects | Nice-to-Have | FEAT-01, FEAT-02, FEAT-06, FEAT-09 | 4 | 50 |
| 025 | FEAT-13 | Immutable Activity & Audit Trail | Core | FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-25, FEAT-31 | 6 | 101 |
| 026 | FEAT-14 | Notifications (Email) | Core | FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-19, FEAT-25, FEAT-31, FEAT-32 | 6 | 98 |
| 027 | FEAT-21 | Settings & Account Management | Important | FEAT-14 | 11 | 134 |
| 028 | FEAT-29 | In-App Notification Center | Nice-to-Have | FEAT-13, FEAT-14 | 4 | 60 |
| 029 | FEAT-31 | Operator Support Access | Important | FEAT-13, FEAT-14 | 7 | 82 |
| 030 | FEAT-33 | Portal Referral Attribution | Important | FEAT-05, FEAT-14 | 5 | 57 |
| 031 | FEAT-20 | Onboarding / First-Run Setup | Important | FEAT-01, FEAT-02, FEAT-19, FEAT-32, FEAT-33 | 6 | 100 |
| 032 | FEAT-24 | Data Export & Account Deletion | Important | FEAT-01, FEAT-33 | 9 | 126 |
| 033 | FEAT-30 | Contextual Help & Guidance | Nice-to-Have | FEAT-05, FEAT-08, FEAT-20 | 5 | 71 |

### Dependency cycle breaks

- FEAT-09 depends on FEAT-21, which the order places later (FEAT-09 was sequenced as
  part of breaking a dependency cycle) — build FEAT-09 against a stub of the
  FEAT-21-facing interface and complete the wiring when FEAT-21 is built.
- FEAT-13 ↔ FEAT-31 are mutually dependent (each lists the other in the dependency
  map) — the order places FEAT-13 first; build iteratively, stubbing the not-yet-built
  counterpart's interface and completing it when its turn comes.
- FEAT-14 ↔ FEAT-31 are mutually dependent (each lists the other in the dependency
  map) — the order places FEAT-14 first; build iteratively, stubbing the not-yet-built
  counterpart's interface and completing it when its turn comes.

## Quickstart

1. **Copy this directory into your project repository** (or work in it directly).
2. **Install Spec Kit and initialize in place:**

   ```
   uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
   specify init --here --integration <your-agent>
   ```

   `init` will warn before merging into a non-empty directory — proceed: `specs/` and
   the shipped constitution are preserved (Spec Kit never modifies `specs/` on init or
   upgrade, and an existing constitution is kept). Pick your agent from the supported
   integrations (Claude Code, Cursor, Copilot, Codex, Windsurf, …).
3. **Skip `/speckit.specify` — the specs are pre-authored.** Feature 001 is pre-selected
   in `.specify/feature.json`.
4. **Run `/speckit.plan`** with a stack hint, e.g.: "Follow the resolved decisions in
   research.md; the recommended architecture is
   docs/blueprint/architecture/technical-architecture.md." (Markdown-command agents use
   the dot form `/speckit.plan`; skills-mode agents — current Claude Code, Cursor, Codex
   layouts — use the hyphen form `/speckit-plan`.)
5. **Then `/speckit.tasks` → `/speckit.implement`.** `/speckit.analyze` is an optional
   consistency check. When a feature is done (constitution Principle V), switch to the
   next one and repeat from step 4.

## Switching features

Edit `.specify/feature.json` to name the next directory (e.g.
`{"feature_directory": "specs/002-currency-tax-handling"}`), or set the `SPECIFY_FEATURE_DIRECTORY`
environment variable (it takes precedence). This file is the state `/speckit.specify`
would normally write — shipped pre-filled so you never need that command here.

## Two kinds of SC IDs

`SC-XX` (two digits) = blueprint scope-boundary IDs — the DO-NOT-BUILD list in
`.specify/memory/constitution.md`. `SC-001…` (three digits) = Spec Kit's per-feature
success-criteria numbering inside each spec.md's Measurable Outcomes. Different ID
spaces; never merge or renumber them.

## Git is optional

Spec Kit no longer creates git branches, and nothing in this bundle keys off branch
names — initialize a repository if you want history (recommended), and add
branch-per-feature only if your team opts into that extension.

## Full depth

Every spec.md is a faithful condensation-plus-verbatim-ACs render; the complete
blueprint — full specifications, architecture with alternatives, database schema,
product research — lives byte-identical under `docs/blueprint/`. `research.md` in each
feature directory resolves the architecture decisions so `/speckit.plan` finds no
unknowns.
