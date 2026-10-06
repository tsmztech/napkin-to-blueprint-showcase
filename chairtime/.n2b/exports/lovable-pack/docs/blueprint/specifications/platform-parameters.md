---
document_type: platform-parameters
produced_by: cross-reference-reconciler
status: final
created: 2026-09-28
parameter_count: 33
marker_site_count: 294
---

# Platform Parameters — Decide Before Build

## What this is

The specifications reference 33 platform-wide policy values by name
instead of fixing numbers — deliberately: these are business decisions the blueprint
surfaces for an explicit decision rather than deciding silently. Each row below names
one parameter, every spec that depends on it, and a **proposed default with rationale**
— a suggestion, not a commitment. **Decide every row before build.**

Every proposed default below is non-binding. Where Stage 2 (product-features.md or a
cross-feature rule in the dependency map) already names a number, the proposal repeats
it so that a decision to keep it is a one-line confirmation; where nothing upstream names
a number, the proposal is grounded in `features/market-research.md` and BRIEF.md
constraints. Some specs quote the Stage 2 number next to the marker for readability;
the decided value replaces it everywhere.

## Registry

| Parameter | Referenced by | What it governs | Proposed default | Rationale | Status |
|-----------|---------------|-----------------|------------------|-----------|--------|
| `account-closure-cooling-off-days` | FEAT-29.SPEC-008, FEAT-29.SPEC-013, FEAT-29.SPEC-017 | How long a closing Pro Account can be reopened before its data is permanently deleted | 30 days | XBR-20 and product-features.md FEAT-29 state a 30-day cooling-off; long enough to cover a monthly billing cycle for a solo pro who changes their mind | decide-before-build |
| `booking-auto-completion-window-days` | FEAT-11.SPEC-001, FEAT-11.SPEC-002, FEAT-11.SPEC-004, FEAT-12.SPEC-004, FEAT-12.SPEC-006 | How long after start_time an unmarked booking auto-completes, which also closes the no-show marking window | 7 days | XBR-12 and product-features.md FEAT-11/FEAT-12 state 7 days; covers a week of between-clients admin for a pro with 20-40 bookings a week (BRIEF.md Scale) | decide-before-build |
| `booking-link-forward-window-months` | FEAT-05.SPEC-001, FEAT-05.SPEC-008, FEAT-27.SPEC-002, FEAT-27.SPEC-007, FEAT-27.SPEC-010 | How long a renamed booking link keeps forwarding (and stays reserved) from its old name | 12 months | XBR-27 sets "at least 12 months"; Instagram bio links persist in old posts and screenshots, and the link is the pro's only booking channel (BRIEF.md Success Criteria) | decide-before-build |
| `cancellation-window-default-hours` | FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-15.SPEC-002 | The cancellation window pre-filled for a new Pro during onboarding (the Pro may change it within 1-168 hours) | 24 hours | Stage 2 asks for "a common cancellation window" as the onboarding default (ASMP policy-default constraint); 24 hours is the plain-language norm across the deposit-policy tools profiled in market-research.md, where unclear terms drive disputes | decide-before-build |
| `checkout-hold-timeout-minutes` | FEAT-03.SPEC-002, FEAT-03.SPEC-003, FEAT-05.SPEC-002, FEAT-05.SPEC-003, FEAT-05.SPEC-006 | How long a client's checkout hold reserves a slot while they pay the deposit | 10 minutes | XBR-02 calls for "a few minutes"; BRIEF.md expects checkout in under a minute but inside the Instagram in-app browser, so 10 minutes leaves room for card entry and bank verification without locking the slot for long | decide-before-build |
| `client-recency-filter-window-days` | FEAT-24.SPEC-002 | The look-back span for the client list's "booked recently" filter | 30 days | product-features.md FEAT-24 gives "booked in the last 30 days" as the example filter | decide-before-build |
| `contact-change-code-expiry-minutes` | FEAT-29.SPEC-012, FEAT-29.SPEC-016 | How long each confirmation code for a sign-in contact change stays valid | 10 minutes | Mirrors the 10-minute sign-in code expiry in product-features.md FEAT-29 so the Pro meets one code pattern (ASMP-30) | decide-before-build |
| `contact-change-code-lockout-pause-minutes` | FEAT-29.SPEC-003, FEAT-29.SPEC-010, FEAT-29.SPEC-012 | How long one side of a contact-change confirmation is paused after repeated wrong codes | 15 minutes | Mirrors the 15-minute sign-in lockout pause in product-features.md FEAT-29 (ASMP-30) | decide-before-build |
| `contact-change-confirmation-window-hours` | FEAT-29.SPEC-012 | How long a pending sign-in contact change waits for both old- and new-contact confirmations before it is discarded | 24 hours | Long enough for a busy pro to confirm between clients on the same working day, short enough that a stale pending change cannot be completed later by someone holding an old device (ASMP-30) | decide-before-build |
| `deposit-request-hold-appointment-cutoff-hours` | FEAT-03.SPEC-003, FEAT-03.SPEC-007, FEAT-30.SPEC-006, FEAT-30.SPEC-010, FEAT-30.SPEC-013 | How close to the appointment a Pro-created deposit-request hold must lapse | 2 hours | XBR-02 states "until 2 hours before the appointment" | decide-before-build |
| `deposit-request-hold-max-hours` | FEAT-03.SPEC-002, FEAT-03.SPEC-003, FEAT-03.SPEC-007, FEAT-30.SPEC-006, FEAT-30.SPEC-010, FEAT-30.SPEC-013 | The longest a Pro-created deposit-request hold reserves a slot | 24 hours | XBR-02 states "up to 24 hours" | decide-before-build |
| `insights-aggregate-retry-count` | FEAT-25.SPEC-004 | How many times a failed insights aggregate update is retried before it is logged as a gap | 3 | Insights are non-blocking (FEAT-25 is Nice-to-Have); a small fixed retry count keeps the figures correct under brief faults without competing with booking and payment work, which BRIEF.md ranks above features | decide-before-build |
| `insights-minimum-history-threshold` | FEAT-25.SPEC-003 | How many occurred bookings (Completed or No-Show) a Pro needs before insights show figures instead of "not enough data yet" | 10 bookings | product-features.md FEAT-25 wants a plain "not enough data yet" state for pros with little history; at 20-40 bookings a week (BRIEF.md Scale) 10 occurred bookings is under a week of use | decide-before-build |
| `message-delivery-retry-count` | FEAT-08.SPEC-009, FEAT-14.SPEC-004, FEAT-14.SPEC-009, FEAT-18.SPEC-007, FEAT-20.SPEC-008, FEAT-20.SPEC-009, FEAT-27.SPEC-013 | How many times a failed text is retried before falling back to email | 1 | XBR-17 and BRIEF.md: a failed text is "retried once, then sent by email" | decide-before-build |
| `minimum-chargeable-deposit` | FEAT-01.SPEC-004, FEAT-07.SPEC-003, FEAT-22.SPEC-003 | The smallest deposit (and in-app balance) amount the product will ask a card to pay, in the account currency | 1.00 in the account currency | Must sit at or above the payment processor's own minimum card charge (ASMP-31); BRIEF.md's example deposit is $20, so a 1.00 floor never blocks a real deposit rule while rejecting amounts the processor fee would consume | decide-before-build |
| `no-show-undo-grace-window-hours` | FEAT-11.SPEC-001, FEAT-11.SPEC-003, FEAT-11.SPEC-004 | How long after marking a no-show the Pro can undo it | 24 hours | XBR-12 and product-features.md FEAT-11 fix the undo grace period at 24 hours | decide-before-build |
| `occurrence-generation-retry-count` | FEAT-21.SPEC-004 | How many times a failed recurring-occurrence generation is retried before the gap is flagged to the Pro | 3 | Matches the other background-retry counts; a missed standing appointment must surface on the dashboard quickly rather than retry silently (BRIEF.md: reliability over features) | decide-before-build |
| `profile-photo-max-file-size-mb` | FEAT-27.SPEC-012 | The largest profile photo file a Pro can upload | 10 MB | product-features.md FEAT-27 asks for "a standard image under a reasonable size limit"; 10 MB accepts a photo straight from a modern phone camera (mobile-first, BRIEF.md Platform) while staying trivial against the under-$100/month infrastructure budget for a few hundred pros | decide-before-build |
| `recurring-occurrence-deposit-lead-days` | FEAT-21.SPEC-005 | How many days before a recurring occurrence its deposit-request link is sent | 10 days | An unpaid occurrence is released at its cancellation cut-off (XBR-02), and a Pro's window can be up to 168 hours (7 days); a 10-day lead always leaves the client at least 3 days to pay before release | decide-before-build |
| `refund-retry-interval-hours` | FEAT-09.SPEC-006, FEAT-22.SPEC-005, FEAT-30.SPEC-011 | How often a refund that could not complete is retried until it succeeds | 6 hours | XBR-10 requires automatic retry until success while the refund shows "in progress"; a 6-hour cadence clears most payout-balance shortfalls within the same day (market-research.md notes next-business-day payouts on comparable tools) without hammering the processor | decide-before-build |
| `reminder-lead-time-days` | FEAT-08.SPEC-007 | How far before the appointment the automatic reminder is sent | 2 days | BRIEF.md Experience: "Two days before, a reminder arrives" | decide-before-build |
| `reminder-window-end-hour` | FEAT-08.SPEC-007, FEAT-20.SPEC-009, FEAT-21.SPEC-007, FEAT-21.SPEC-008, FEAT-21.SPEC-009 | The latest local hour (Pro's timezone) an automatic reminder may be sent | 21:00 (9pm) | XBR-16: reminders go out "only between roughly 8am and 9pm in the Pro's timezone" | decide-before-build |
| `reminder-window-start-hour` | FEAT-08.SPEC-007, FEAT-20.SPEC-009, FEAT-21.SPEC-007, FEAT-21.SPEC-008, FEAT-21.SPEC-009 | The earliest local hour (Pro's timezone) an automatic reminder may be sent | 08:00 (8am) | XBR-16: reminders go out "only between roughly 8am and 9pm in the Pro's timezone" | decide-before-build |
| `session-inactivity-expiry-days` | FEAT-29.SPEC-006, FEAT-29.SPEC-011 | How long a signed-in device stays signed in without use | 30 days | product-features.md FEAT-29: "a sign-in stays active on a device for up to 30 days of inactivity" | decide-before-build |
| `sign-in-code-expiry-minutes` | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-011, FEAT-29.SPEC-014 | How long a one-time sign-in code stays valid | 10 minutes | product-features.md FEAT-29: "One-time codes expire after 10 minutes" | decide-before-build |
| `sign-in-lockout-pause-minutes` | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-006, FEAT-29.SPEC-011 | How long sign-in code entry is paused after the lockout threshold is reached | 15 minutes | product-features.md FEAT-29: "further attempts are paused for 15 minutes" | decide-before-build |
| `sign-in-lockout-threshold` | FEAT-29.SPEC-011 | How many consecutive wrong sign-in codes trigger the lockout pause | 5 attempts | product-features.md FEAT-29: "after 5 failed attempts" | decide-before-build |
| `subscription-payment-failure-grace-period-days` | FEAT-05.SPEC-008, FEAT-18.SPEC-003, FEAT-18.SPEC-004, FEAT-18.SPEC-005, FEAT-18.SPEC-007 | How long a Pro has to fix a failed renewal before new bookings pause | 7 days | XBR-14 and product-features.md FEAT-18 state a 7-day grace period | decide-before-build |
| `subscription-price` | FEAT-18.SPEC-001, FEAT-18.SPEC-002, FEAT-18.SPEC-003, FEAT-18.SPEC-005, FEAT-18.SPEC-006, FEAT-18.SPEC-007 | The single all-inclusive monthly subscription price every Pro pays | 29 per month (USD at US launch) | BRIEF.md leaves the price open ("a single saved no-show pays for the month"); market-research.md shows solo tiers at about $28 (GlossGenius Standard), $29.99 (Booksy), about $30 (Vagaro) and $35 (StyleSeat), and fee unpredictability, not price, is the main complaint; $29 sits at the low end of that band and below the $65 example service in BRIEF.md | decide-before-build |
| `subscription-price-change-notice-days` | FEAT-18.SPEC-003, FEAT-18.SPEC-005, FEAT-18.SPEC-007 | How far in advance a Pro is told about a subscription price change | 30 days | Stage 2 (FEAT-18) names a 30-day notice; market-research.md shows that pricing changes made with little warning (Fresha 2025) caused the strongest trust backlash in this market | decide-before-build |
| `waitlist-claim-window-minutes` | FEAT-20.SPEC-004, FEAT-20.SPEC-005, FEAT-20.SPEC-007, FEAT-20.SPEC-008, FEAT-20.SPEC-009 | How long a notified waitlisted client has priority to claim a freed slot before it returns to general availability | 30 minutes | XBR-02 and XBR-28: "a 30-minute priority window" | decide-before-build |
| `waitlist-join-range-max-days` | FEAT-20.SPEC-001, FEAT-20.SPEC-003 | The longest date range one waitlist entry can cover | 7 days | product-features.md FEAT-20: "one day (or a range of up to 7 days)" | decide-before-build |
| `waitlist-max-active-entries-per-pro` | FEAT-20.SPEC-003 | How many active waitlist entries one client may hold with one Pro | 3 | product-features.md FEAT-20: "a client may hold at most 3 active waitlist entries per Pro" | decide-before-build |

## Contract

These values are deliberately not decided by the blueprint. Every spec that references
a row's slug behaves per its own acceptance criteria *given* the value; the value
itself is the product owner's pre-build decision. When a value is decided, record it in
the Status column of your working copy (`decided: {value}`) — the specs need no edits.
