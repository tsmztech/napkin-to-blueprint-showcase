# Stage 1 session notes — Chairtime

Every question n2b asked during `n2b:s1-init`, and the founder answer given (from `napkin.md` / `ANSWERS.md`).

### Q: What do you want to build? Tell me everything — the idea, the problem, who it's for, how you imagine it working.
The full body of `napkin.md`, verbatim (preserved as `.n2b/inputs/source/pasted-notes.md`).

### Q: Roles — you mentioned "basic support access" for yourself. So that's a read-only support view of a pro's account, not a product role?
Read-only support view of a pro's account for helping them; not a product role, no client-facing access. Fine to note as a minimal admin role.

### Q: Money flow — the card deposit goes to the pro via the processor; a no-show or cancellation inside the window forfeits it to the pro; a cancellation outside the window refunds it automatically; the platform takes no cut. Is that right?
Yes. Deposit paid by card at booking, goes to the pro (processor handles payout to the pro). Balance is due at the appointment; whether it can also be paid in-app is an open question. No-show or cancellation inside the window: deposit forfeited to the pro. Cancellation outside the window: deposit refunded automatically. Platform takes no cut of any of it. Revenue is a flat monthly subscription paid by the pro, card-based, cancel anytime, one price tier for v1.

### Q: Regulated areas — payments (processor owns card data), US SMS consent, personal-data deletion. Any health data from massage or tattoo?
Yes: payments (processor-owned card data), US SMS consent rules, personal data deletion on request. No health data — massage/tattoo intake forms are out of scope for v1.

### Q: Who is the very first pro — one concrete person from your network?
A lash tech or barber I personally know, who currently books via Instagram DMs and asks for Venmo deposits by hand, ~30 clients a week, home studio or rented chair.

### Q: Before I play this back — any hard boundaries I should know about? Timeline, budget, regulations, existing systems, technology commitments, brand rules — anything non-negotiable?
The HARD BOUNDARIES list, nothing else: just me building with AI coding tools, first paying pro within ~3 months; infrastructure under ~$100/month until revenue; card data never stored or handled by my code; clients must explicitly agree to texts at booking (US texting rules); a pro must be able to delete a client's record, and client data is never visible to any other pro or client; mobile-first web only, no native apps or app stores; no technology commitments.

### Q: How would you like to proceed? (after the "Here's what I got" show-back)
Looks good — take it from here (Path C).

### Q: Which AI models should n2b's agents use?
Balanced — smart planning, fast execution. Provider: Claude aliases (default, not asked on Claude Code).
