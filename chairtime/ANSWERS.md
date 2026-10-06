# Stage 1 answer key — Chairtime

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
| Product in two sentences | "A booking link for one solo beauty pro that takes a card deposit and sends reminders. Client books in a minute; pro never chases a no-show again." |
| Who the *first* user is | A lash tech or barber I personally know, who currently books via Instagram DMs and asks for Venmo deposits by hand, ~30 clients a week, home studio or rented chair. |
| Trigger for usage | Client side: they saw the pro's work on Instagram and tap the bio link. Pro side: opens it between clients to see who's next and who's paid; sets up services once. |
| Whether "single operator" is truly confirmed | Yes, confirmed. Multi-chair salons are out of scope for v1 and probably forever for this product. |
| Platform-operator / support role | Read-only support view of a pro's account for helping them; not a product role, no client-facing access. Fine to note as a minimal admin role or leave as open question. |
| Deposit vs balance money flow | Deposit paid by card at booking, goes to the pro (processor handles payout to the pro). Balance is due at the appointment; whether it can also be paid in-app is an open question. No-show or cancellation inside the window: deposit forfeited to the pro. Cancellation outside the window: deposit refunded automatically. Platform takes no cut of any of it. |
| How the platform makes money | Flat monthly subscription paid by the pro, card-based, cancel anytime. One price tier for v1. |
| Why now | Post-pandemic wave of solo pros on Instagram; card deposits normalised; the market gap between salon suites and free calendar links. |
| Order of magnitude | A few hundred pros year one; up to ~500 clients each; 20–40 bookings per pro per week. |
| Devices / platforms | Mobile-first web for both roles; must behave inside the Instagram in-app browser; desktop is a bonus for the pro's setup screens. |
| Geography | US first; UK / Canada / Australia next, so timezone and currency stay configurable. |
| Availability / performance expectations | Booking and payment must be correct, always. No specific uptime number; "it must never silently double-book". |
| Calendar relationship | Two-way: external busy blocks availability; bookings written to the pro's calendar. Google and Apple both matter. |
| Reminders channel | SMS in v1 with explicit opt-in at booking; WhatsApp later. Email confirmations are fine as a fallback. |
| Regulated-domain confirmation | Yes: payments (processor-owned card data), US SMS consent rules, personal data deletion on request. No health data — massage/tattoo intake forms are out of scope for v1. |
| Design preferences / brand | No design system supplied. Preference only: clean, warm, looks good on a phone, each pro can add their name, logo/photo, and an accent colour. No artifacts to ingest. |
| Constraints question ("anything non-negotiable?") | Repeat the HARD BOUNDARIES list; nothing else. |
| Feature discussion (Path D) | Decline for this showcase — pick Path C so the repo shows n2b's own discovery. |
