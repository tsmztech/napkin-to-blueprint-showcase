# Build Order

The features below are ordered topologically — Kahn's algorithm over the dependency map's
Features-table `Depends On` column, with a deterministic cycle-break restricted to features
inside a dependency cycle (lowest priority-tier rank Core < Important < Nice-to-Have, then
lowest FEAT number) whose broken edges are disclosed at the end.
`feature_list.json` follows this exact order — features as listed here, specs within a
feature in SPEC number order. Follow the list; do not re-derive the order.

| # | Feature | Name | Priority | Depends on | Specs | ACs |
|---|---------|------|----------|------------|-------|-----|
| 1 | FEAT-01 | Household Setup & Member Profiles | Core | — | 18 | 180 |
| 2 | FEAT-02 | Dietary Rules & Allergy Safety Engine | Core | FEAT-01 | 14 | 158 |
| 3 | FEAT-08 | Recipe Library (Starter Recipes) | Core | FEAT-02 | 4 | 54 |
| 4 | FEAT-09 | Household Invitations & Membership | Core | FEAT-01 | 14 | 137 |
| 5 | FEAT-10 | Recipe Import from Web Link | Important | FEAT-02, FEAT-08 | 8 | 105 |
| 6 | FEAT-14 | Subscription & Billing Management | Important | — | 12 | 151 |
| 7 | FEAT-05 | Pantry-Aware Suggestions | Core | FEAT-14 | 7 | 78 |
| 8 | FEAT-15 | Member Onboarding | Important | FEAT-09 | 3 | 29 |
| 9 | FEAT-16 | Units, Currency & Locale Configuration | Important | — | 4 | 50 |
| 10 | FEAT-18 | Account & Data Management | Important | FEAT-01, FEAT-09 | 15 | 175 |
| 11 | FEAT-23 | Manual Weekly Planning | Core | FEAT-01, FEAT-02, FEAT-08, FEAT-10 | 6 | 85 |
| 12 | FEAT-24 | Invite Another Household | Important | FEAT-01, FEAT-14 | 7 | 69 |
| 13 | FEAT-25 | Weekly Waste & Spend Check-In | Important | FEAT-01, FEAT-16 | 5 | 76 |
| 14 | FEAT-03 | AI Weekly Dinner Plan Generation | Core | FEAT-01, FEAT-02, FEAT-05, FEAT-08, FEAT-10, FEAT-12, FEAT-14, FEAT-16 | 11 | 135 |
| 15 | FEAT-04 | One-Tap Meal Swap | Core | FEAT-02, FEAT-03, FEAT-23 | 10 | 114 |
| 16 | FEAT-06 | Shared Grocery List | Core | FEAT-03, FEAT-04, FEAT-05, FEAT-16, FEAT-23 | 9 | 104 |
| 17 | FEAT-07 | Weekly Plan Ready Notification | Core | FEAT-03 | 6 | 71 |
| 18 | FEAT-11 | Leftover Rollover to Lunches | Important | FEAT-03, FEAT-04 | 4 | 47 |
| 19 | FEAT-12 | Meal Rating & Preference Learning | Important | FEAT-01, FEAT-03, FEAT-14, FEAT-23 | 5 | 63 |
| 20 | FEAT-13 | Tonight's Dinner Reminder | Important | FEAT-03, FEAT-04, FEAT-23 | 6 | 60 |
| 21 | FEAT-17 | Older-Kid Dinner Voting | Important | FEAT-02, FEAT-03 | 6 | 74 |
| 22 | FEAT-19 | Weekly Plan History | Nice-to-Have | FEAT-02, FEAT-03, FEAT-06 | 4 | 49 |
| 23 | FEAT-20 | Online Grocery Ordering Handoff | Nice-to-Have | FEAT-06 | 3 | 50 |
| 24 | FEAT-21 | Family Calendar Sync | Nice-to-Have | FEAT-03, FEAT-04 | 5 | 71 |
| 25 | FEAT-22 | Operator Read-Only Support Access | Nice-to-Have | FEAT-01, FEAT-02, FEAT-03, FEAT-06, FEAT-18 | 10 | 107 |

## Dependency cycle breaks

- FEAT-03 ↔ FEAT-12 are mutually dependent (each lists the other in the dependency
  map) — the order places FEAT-03 first and interleaves them; build iteratively,
  stubbing the not-yet-built counterpart's interface and completing it when its turn
  comes.
