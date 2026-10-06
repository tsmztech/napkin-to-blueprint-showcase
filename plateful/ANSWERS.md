# Stage 1 answer key — Plateful

You are the founder. Everything in `napkin.md` counts as *given*. If n2b still asks, answer from the
table below in the founder's voice. Don't volunteer more than asked. Where this is silent, choose the
conventional option and let n2b record it as an open question.

## Fixed choices

| Prompt | Choice |
|---|---|
| Open capture ("tell me about your idea") | The full body of `napkin.md`, verbatim |
| Fork after the show-back | **Looks good — take it from here** (Path C). Decline feature discussion (Path D). |
| Model profile | **balanced** |
| Provider | **Claude aliases** (default) |
| Design-system artifacts? | None. Blueprint ships design-agnostic; the preference below goes into Constraints. |
| Existing `.n2b/` found | Should not happen in a fresh run — if it does, stop and report. |

## Follow-up answers

| If it asks about… | Answer |
|---|---|
| Product in two sentences | "A shared family meal planner where AI proposes a realistic week of dinners that respects everyone's allergies, diets, schedule and budget. One tap to swap, one shared grocery list the whole household ticks off live." |
| Who the *first* user is | A working parent like me: two kids, one with a food allergy, a partner with a different diet, 30-minute weeknights, currently planning in a notes app and a group chat. |
| Trigger for usage | Organiser: Sunday "plan is ready" notification. Everyone else: the shared grocery list in the store, the evening "tonight's dinner" nudge, rating meals after dinner. |
| Roles confirmed? | Yes: organiser, other adult members, and kids (represented somehow — open question). One household per account in v1. |
| Operator / support role | Read-only support access to help a household; not a product role. |
| Kids' accounts | Open question. Leaning: young kids are parent-managed profiles with no login; older kids might get a limited login later. Keep kids' data minimal. |
| Allergy handling | Hard rule, not preference. Every AI suggestion is checked by the app against the household allergy list before anyone sees it; the AI is never the last line of defence. Show a "checked against allergies" badge; standard "always check labels" disclaimer. |
| Dietary preferences vs rules | Allergies and religious rules (e.g. halal) are hard filters; vegetarian can be per-person with a "vegetarian option" on a shared meal; dislikes are soft and learned from ratings. |
| How the AI plan works | Weekly, on demand or scheduled Sunday; respects time per night, budget, pantry items the user says they have, variety (no repeats within 2 weeks), and past ratings. |
| Grocery list | One combined list per week, quantities merged and units converted, grouped by aisle, shared live across household, works offline in the store and syncs later. Manual items can be added by anyone. |
| How it makes money | Free tier (manual planning + shared list); paid household subscription (monthly/yearly) for AI planning, pantry-aware suggestions and learning. No ads, no data selling. |
| AI cost | Must stay small per household: roughly one weekly plan plus a few swaps; free tier gets no AI. |
| Why now | AI can finally plan around real constraints; food prices up so waste hurts; every family already shares a grocery list somewhere awful. |
| Order of magnitude | Several thousand households year one, 2–6 people each. |
| Devices / platforms | Responsive web, mobile-first; organiser may use a laptop for setup. No native apps in v1. |
| Geography | US and UK first; units, currency and aisle names configurable. |
| Availability / performance | Shared list must feel instant and survive bad supermarket signal. No specific uptime number. |
| Recipes | Starter library plus save-from-link; legality of importing is an open question. |
| Integrations | AI model (any). Online grocery ordering and calendar are later, not v1. |
| Regulated-domain confirmation | Yes: children's data (minimal, parent-controlled), food-allergy safety, no ads/data selling. Nutrition claims are an open question — no medical or diet advice. |
| Design preferences / brand | No design system supplied. Preference only: friendly, calm, food-photo-led, big tap targets for one-handed use in a shop. No artifacts to ingest. |
| Constraints question ("anything non-negotiable?") | Repeat the HARD BOUNDARIES list; nothing else. |
| Feature discussion (Path D) | Decline for this showcase — pick Path C so the repo shows n2b's own discovery. |
