# Session notes — Stage 1 intake (n2b:s1-init)

Every question n2b asked during intake, and the answer given as the founder (from `napkin.md` / `ANSWERS.md`).

### Q: What do you want to build? Tell me everything — the idea, the problem, who it's for, how you imagine it working. If you already have a written brief, paste it.
The entire body of `napkin.md`, verbatim (preserved by n2b at `.n2b/inputs/source/pasted-notes.md`).

### Q: (Interpretation check) Dietary rules — are allergies and religious rules hard filters, with vegetarian per person and dislikes as soft preferences?
Yes. Allergies and religious rules (e.g. halal) are hard filters; vegetarian can be per-person with a "vegetarian option" on a shared meal; dislikes are soft and learned from ratings.

### Q: (Interpretation check) Allergy safety — should the app itself check every AI suggestion against the allergy list, rather than trusting the AI?
Hard rule, not preference. Every AI suggestion is checked by the app against the household allergy list before anyone sees it; the AI is never the last line of defence. Show a "checked against allergies" badge; standard "always check labels" disclaimer.

### Q: (Interpretation check) The operator — you only need support access, not a product role?
Read-only support access to help a household; not a product role.

### Q: (Interpretation check) Grocery list — one combined weekly list, merged quantities, shared live, usable offline?
One combined list per week, quantities merged and units converted, grouped by aisle, shared live across household, works offline in the store and syncs later. Manual items can be added by anyone.

### Q: You said the AI proposes a "realistic" week — what should it weigh, and how often does it run?
Weekly, on demand or scheduled Sunday; respects time per night, budget, pantry items the user says they have, variety (no repeats within 2 weeks), and past ratings. AI cost must stay small per household: roughly one weekly plan plus a few swaps; free tier gets no AI.

### Q: Children use this and allergies are involved — confirming this touches regulated/sensitive ground (children's data, food-allergy safety, nutrition)?
Yes: children's data (minimal, parent-controlled), food-allergy safety, no ads/data selling. Nutrition claims are an open question — no medical or diet advice.

### Q: Any brand or design preferences — and do you have design artifacts (files, a URL, tokens)?
No design system supplied. Preference only: friendly, calm, food-photo-led, big tap targets for one-handed use in a shop. No artifacts to ingest.

### Q: Before I play this back — any hard boundaries I should know about? Timeline, budget, regulations, existing systems, technology commitments, brand rules — anything non-negotiable?
The HARD BOUNDARIES list, nothing else:
- Team: just me, building with AI coding tools. I want paying households within about three months.
- Budget: infrastructure under roughly $100/month until there's revenue — which means AI costs per household have to stay small.
- Safety: allergies are a hard rule, not a preference. The AI must never be the last line of defence; the app has to check its suggestions against the household's allergy list.
- Privacy: children use it, so their data is minimal and never used for anything but the family's own plan. No ads, no data selling.
- Platform: responsive web app for v1. No native apps, no app stores.
- No technology commitments — build it with whatever the blueprint recommends.

### Q: (After show-back) How would you like to proceed? [Looks good — take it from here / Close, but I want to add more / Not quite — let me correct / Let's outline key features too]
Looks good — take it from here (Path C).

### Q: Which AI models should n2b's agents use? [Balanced / Quality / Budget]
Balanced — smart planning, fast execution. (Provider: Claude aliases — the default; no provider question on Claude Code.)
