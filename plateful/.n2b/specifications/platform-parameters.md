---
document_type: platform-parameters
produced_by: cross-reference-reconciler
status: final
created: 2026-09-28
parameter_count: 9
marker_site_count: 36
---

# Platform Parameters — Decide Before Build

## What this is

The specifications reference 9 platform-wide policy values by name
instead of fixing numbers — deliberately: these are business decisions the blueprint
surfaces for an explicit decision rather than deciding silently. Each row below names
one parameter, every spec that depends on it, and a **proposed default with rationale**
— a suggestion, not a commitment. **Decide every row before build.**

## Registry

| Parameter | Referenced by | What it governs | Proposed default | Rationale | Status |
|-----------|---------------|-----------------|------------------|-----------|--------|
| `calendar-sync-retry-interval` | FEAT-21.SPEC-004 | How far apart successive automatic retries of a failed per-night calendar entry are spaced | 1 hour | Family Calendar Sync is a Later-phase nice-to-have that must never block the plan (BRIEF.md, Ecosystem: "a nice-to-have ... not v1"); an hourly retry repairs a same-evening outage without spending the under-$100/month infrastructure budget (BRIEF.md, Constraints) on tight polling | decide-before-build |
| `calendar-sync-retry-max-attempts` | FEAT-21.SPEC-004 | The ceiling for both counters: a night's own consecutive failures before it shows Failing, and the household's consecutive fully-failed sync runs before the connection is treated as Disconnected | 5 attempts | With the 1-hour interval this gives about five hours of recovery before the household is asked to reconnect. Market research shows that calendar reliability drives strong loyalty and strong backlash (Cozi's shared-calendar category, market-research.md), so the product should prefer a clear "reconnect" prompt to failing silently for days | decide-before-build |
| `data-export-rate-limit-count` | FEAT-18.SPEC-010 | How many household data exports the organiser may request within one rate-limit window before further requests are blocked | 3 exports per window | Exports serve general personal-data rights (ASMP-27) and are rare, deliberate acts. Three allows a retry after a failed download while bounding generation cost under the bootstrapped infrastructure budget (BRIEF.md, Constraints: under roughly $100/month until revenue) | decide-before-build |
| `data-export-rate-limit-window` | FEAT-18.SPEC-010 | The fixed, rolling-forward cadence over which export requests are counted | 24 hours | A daily window resets quickly enough that a family with a genuine need is never blocked for long, and it still caps any repeated or automated requesting. This matches the low-frequency, privacy-first stance in BRIEF.md ("no selling of family data") | decide-before-build |
| `nightly-nudge-send-time` | FEAT-13.SPEC-001, FEAT-13.SPEC-002 | The single daily time, in household local time, at which the "Tonight: ..." dinner nudge fires | 4:00 pm household local time | BRIEF.md places the nudge in the evening, before cooking, for families with "30-minute weeknights". A mid-afternoon send leaves time to shop or start any early prep that FEAT-13.SPEC-006 surfaces, and it still falls in waking hours, so no quiet-hours window is needed (FEAT-13.SPEC-002) | decide-before-build |
| `plan-arrival-time-slots` | FEAT-01.SPEC-010, FEAT-07.SPEC-004 | The small set of selectable Morning and Evening time slots for the organiser's plan-arrival day and time, and which Evening slot is the default | Morning: 7:00, 8:00, 9:00 am; Evening: 5:00, 6:00, 7:00 pm; default Evening slot 6:00 pm (with Sunday as the default day) | BRIEF.md's anchor scenario is "It's Sunday evening and a notification arrives: 'Next week's plan is ready.'" Six hourly slots keep the choice small (the Stage 2 "small set" constraint), and 6:00 pm Sunday lands in the planning moment the BRIEF describes ("Every Sunday night families face the same question") | decide-before-build |
| `repeated-dislike-rating-count` | FEAT-12.SPEC-005 | How many thumbs-down ratings by the same member on the same meal turn it into a learned soft dislike | 3 down-ratings | Learned dislikes are soft and never block a suggestion (BRIEF.md: "Dislikes are soft and learned from ratings"; XBR-17), so a modest threshold is low-risk. Three is enough to tell a real dislike apart from one off night, especially for kids' ratings recorded by an adult. Market research names the soft-preference layer learned over time as a differentiator (market-research.md), and it needs a signal that shows up within a few weeks | decide-before-build |
| `subscription-price-monthly` | FEAT-14.SPEC-002, FEAT-14.SPEC-008, FEAT-14.SPEC-009 | The paid household subscription price when billed monthly, in each supported currency | US$5.99 / £4.99 per month | This sits below Samsung Food+ ($6.99/mo), which gates the same class of AI meal plans and pantry features, and above Mealime Pro's former $2.99/mo (market-research.md, pricing table). It suits a freemium market that gates advanced capability behind a low-cost subscription, and a paid tier that must cover per-household AI cost (BRIEF.md, Constraints) | decide-before-build |
| `subscription-price-yearly` | FEAT-14.SPEC-002, FEAT-14.SPEC-008, FEAT-14.SPEC-009 | The paid household subscription price when billed yearly, in each supported currency | US$49.99 / £41.99 per year | About 30% below twelve monthly payments, which matches the freemium-with-annual-subscription pattern that dominates the category (market-research.md). It stays under Samsung Food+ yearly ($59.99) and above list-only household tiers (AnyList household $14.99/yr, Cozi Gold $39/yr), since Plateful's paid tier adds AI planning | decide-before-build |

## Contract

These values are deliberately not decided by the blueprint. Every spec that references
a row's slug behaves per its own acceptance criteria *given* the value; the value
itself is the product owner's pre-build decision. When a value is decided, record it in
the Status column of your working copy (`decided: {value}`) — the specs need no edits.
