# FEAT-12 — Freelancer Financial Dashboard

This chapter covers Freelancer Financial Dashboard, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 4 specifications carrying 68 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-12.SPEC-001 | Dashboard Overview | screen | 17 |
| FEAT-12.SPEC-002 | Client/Project Financial Drill-down | screen | 15 |
| FEAT-12.SPEC-003 | Financial Totals Aggregation | logic-rule | 20 |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | automation | 16 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Freelancer Financial Dashboard

## Summary

**Feature:** Freelancer Financial Dashboard
**ID:** FEAT-12
**Description:** The freelancer sees earned, outstanding, and overdue totals across all clients and projects at a glance, and can drill into a single client or project's financial detail.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Experience narrative: "At month end you open your dashboard and see earned, outstanding and overdue, per client" — this is the brief's explicit description of the freelancer's core recurring need. MVP phase: the "money in one place" promise depends on it.

**Key Capabilities:**
- View aggregate totals — earned, outstanding, overdue across all clients
- Drill into a client or project — see the financial detail behind the aggregate
- See status at a glance — visually distinguish paid, due, and overdue

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-12.SPEC-001 | Dashboard Overview | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia sees aggregate earned/outstanding/overdue totals across every client, filterable by client or period, with a zero-state for a new account |
| FEAT-12.SPEC-002 | Client/Project Financial Drill-down | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia opens one client or project to see the specific invoices behind its aggregate totals |
| FEAT-12.SPEC-003 | Financial Totals Aggregation | Logic/Rule | Nadia (Freelancer), Dana (Support Operator) | Derives earned/outstanding/overdue totals (and their paid/due/overdue status classification) from Invoice and Payment records, per currency, honoring refunds and manual payments |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | Automation | Nadia (Freelancer), Dana (Support Operator) | Recomputes affected totals when an underlying invoice or payment changes elsewhere, and falls back to the last successfully computed totals with retry if aggregation fails |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View aggregate totals — earned, outstanding, overdue across all clients | FEAT-12.SPEC-001, FEAT-12.SPEC-003 | Dashboard Overview is the primary screen; the aggregation rule computes the totals it displays | Phase 2 (Explicit) |
| Drill into a client or project — see the financial detail behind the aggregate | FEAT-12.SPEC-002 | Primary purpose of the Client/Project Financial Drill-down screen | Phase 2 (Explicit) |
| See status at a glance — visually distinguish paid, due, and overdue | FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003 | Status badges/color treatment on both screens, driven by the classification the aggregation rule produces | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Filter by client or period (Primary Flows & Alternates) | FEAT-12.SPEC-001 | Alternate flow on the Dashboard Overview screen narrows totals via FEAT-12.SPEC-003 | Phase 2 (Explicit elaboration of stated alternate flow) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-12.SPEC-003 | Financial Totals Aggregation | Phase 5 (Rule Discovery) | The feature's Data Notes state all totals are "derived... nothing is captured directly in this view," and the Validation & Limits field plus XBR-18/XBR-20/XBR-22 impose a multi-condition derivation (per-currency separation, single-full-payment rule, refunds/reversals/manual payments) that exceeds the inline-validation threshold and is shared by both screens and the refresh automation |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | Phase 4 (Trigger-Response) | The Send a Proposal journey's Step 5 requires the dashboard to reflect a payment "instantly," and the feature's Error state requires the last successfully computed totals plus retry on aggregation failure — a cross-feature, stateful side-effect nobody named as a capability |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity of its own. Its Connected Entities (product-features.md) are Invoice, Payment, and Project, all listed as **read** — the dashboard is a derived view with no create, update, delete, or state-transition surface of its own (Data Notes: "nothing is captured directly in this view"). No CRUD matrix applies; the full lifecycle for each entity is owned and specified by the features named below.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invoice | FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003, FEAT-12.SPEC-004 | Status (Sent, Payment pending, Paid, Overdue, Refunded, Partially refunded, Disputed, Corrected), amount, currency, and due date drive every total and the client/project drill-down list; owned by FEAT-09 (creation, sending) and updated by FEAT-10/FEAT-11/FEAT-25 |
| Payment | FEAT-12.SPEC-003, FEAT-12.SPEC-004 | Succeeded/Failed/Reversed status and amount determine the earned figure and whether a refund or reversal reduces it (XBR-20, XBR-22); owned by FEAT-10 |
| Project | FEAT-12.SPEC-001, FEAT-12.SPEC-002 | Groups invoices for the drill-down and names the client/project the freelancer narrows into; owned by FEAT-01 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia opens the dashboard | Compute aggregate earned/outstanding/overdue totals across every client, per currency | Standalone Logic/Rule | FEAT-12.SPEC-003 |
| Nadia narrows totals to one client or a date range | Recompute filtered totals using the same aggregation rule | Inline in triggering screen (uses SPEC-003) | FEAT-12.SPEC-001 |
| Nadia drills into a specific client or project | Load that entity's invoice-level financial detail | Inline in triggering screen (uses SPEC-003) | FEAT-12.SPEC-002 |
| An invoice or payment underlying the totals changes elsewhere (sent, paid, overdue flag set, refunded, reversed, manually recorded) | Recompute the affected totals so the dashboard reflects the change without a manual refresh | Standalone Automation | FEAT-12.SPEC-004 |
| Aggregation across many clients takes noticeable time | Show a lightweight progress indicator while totals render | Inline in triggering screen | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 |
| Aggregation fails | Show the last successfully computed totals with a retry option — never a blank or misleading number | Standalone Automation | FEAT-12.SPEC-004 |
| Freelancer has no invoices yet | Show an explanatory zero-state with a prompt toward sending a first proposal | Inline in triggering screen (navigates cross-feature to FEAT-02) | FEAT-12.SPEC-001 |
| A filter (client or period) matches no invoices | Show no-results messaging distinct from the true zero-state, with a way to clear the filter | Inline in triggering screen | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 |
| Connectivity is lost while viewing | Keep the most recently loaded totals viewable, read-only | Inline in triggering screen | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 |
| Totals span more than one currency | Keep each currency's totals separate; never convert or sum across currencies (XBR-18) | Standalone Logic/Rule | FEAT-12.SPEC-003 |
| Nadia opens an invoice flagged Overdue on the dashboard | Navigate to that invoice's reminder history and pause control | Cross-feature — owned by FEAT-11 | FEAT-12.SPEC-001 initiates; FEAT-11 owns the destination |
| Nadia opens a specific invoice from the client drill-down | Navigate to invoice detail | Cross-feature — owned by FEAT-09 | FEAT-12.SPEC-002 initiates; FEAT-09 owns the destination |
| Dana opens a logged support session | View the dashboard and drill-down read-only; no filter-driven export control is shown | Cross-feature — owned by FEAT-31 | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 (View), FEAT-31 (session logging) |

## Shared Context

**Shared Entities:**
- **Invoice** (read-only) — read by FEAT-12.SPEC-001 (aggregate and per-currency totals), FEAT-12.SPEC-002 (drill-down invoice list), FEAT-12.SPEC-003 (derivation source), and FEAT-12.SPEC-004 (change detection). Fields read: status, amount, currency, due_date, project.
- **Payment** (read-only) — read by FEAT-12.SPEC-003 and FEAT-12.SPEC-004. Fields read: status, amount, paid_at, invoice.
- **Project** (read-only) — read by FEAT-12.SPEC-001 (client/project grouping) and FEAT-12.SPEC-002 (drill-down scope). Fields read: project_name, client, currency.

**Shared UI Patterns:**
- **Status badge / color treatment (paid, due, overdue)** — the same visual vocabulary appears on FEAT-12.SPEC-001 (aggregate totals) and FEAT-12.SPEC-002 (per-invoice detail), both driven by the classification FEAT-12.SPEC-003 produces; Spec Writers for both screens should describe it identically and never rely on color alone (ASMP-27).
- **Per-currency total block** — both screens render one self-contained total block per currency present, never a combined figure; Spec Writers should describe this the same way on both screens (XBR-18).
- **Empty / Loading / Error / No-Results / Offline-degraded states** — FEAT-12.SPEC-001 and FEAT-12.SPEC-002 share one convention: zero-state prompt toward a first proposal (dashboard only), lightweight progress indicator during aggregation, last-successfully-computed totals with retry on error, distinct no-results messaging for an empty filter result, and read-only viewing of the last-loaded totals when offline.

**Shared Validation:**
- FEAT-12.SPEC-003 (Financial Totals Aggregation) is referenced by FEAT-12.SPEC-001, FEAT-12.SPEC-002, and FEAT-12.SPEC-004 rather than re-deriving the earned/outstanding/overdue formula, the currency-separation rule, or the paid/due/overdue classification in each.

## Internal Dependency Map

```
SPEC-001 (Dashboard Overview) -> [Nadia opens the dashboard] -> SPEC-003 (Financial Totals Aggregation) -> [aggregate totals, per currency] -> SPEC-001
SPEC-001 (Dashboard Overview) -> [Nadia narrows to a client or period] -> SPEC-003 (Financial Totals Aggregation) -> [filtered totals] -> SPEC-001
SPEC-001 (Dashboard Overview) -> [Nadia drills into a client or project] -> SPEC-002 (Client/Project Financial Drill-down)
SPEC-002 (Client/Project Financial Drill-down) -> [loads] -> SPEC-003 (Financial Totals Aggregation) -> [per-client/project totals and invoice list] -> SPEC-002
SPEC-004 (Dashboard Totals Refresh) -> [an Invoice or Payment changes elsewhere] -> SPEC-003 (Financial Totals Aggregation) -> [recomputed totals] -> SPEC-001 / SPEC-002
SPEC-001 (Dashboard Overview) -> [aggregation fails] -> SPEC-004 (Dashboard Totals Refresh) -> [last successfully computed totals + retry] -> SPEC-001
SPEC-002 (Client/Project Financial Drill-down) -> [aggregation fails] -> SPEC-004 (Dashboard Totals Refresh) -> [last successfully computed totals + retry] -> SPEC-002
```

**Default Entry:** SPEC-001 (Dashboard Overview) -- the screen shown when the user navigates to this feature.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-12.SPEC-001 | Outbound | FEAT-11 (Automated Payment Reminders) | Opens an overdue invoice's reminder history and pause control | Nadia opens an invoice flagged Overdue on the dashboard |
| FEAT-12.SPEC-002 | Outbound | FEAT-09 (Invoice Generation & Sending) | Opens a specific invoice's detail | Nadia opens an invoice from the client drill-down |
| FEAT-12.SPEC-001 | Outbound | FEAT-22 (Accounting Export) | Opens the accounting export flow | Nadia generates the month's export from the dashboard |
| FEAT-12.SPEC-001 | Outbound | FEAT-02 (Proposal Creation & Sending) | Zero-state prompt opens a new proposal draft | Freelancer with no invoices yet taps the zero-state prompt |
| FEAT-12.SPEC-001 / FEAT-12.SPEC-002 | Inbound | FEAT-20 (Guided Onboarding) | Onboarding completion lands Nadia on the normal dashboard | Onboarding's first client, project, and draft proposal exist |
| FEAT-12.SPEC-003 / FEAT-12.SPEC-004 | Inbound | FEAT-09 (Invoice Generation & Sending) | Invoice creation, sending, and due-date changes feed the totals | An invoice is generated, sent, or its due date changes |
| FEAT-12.SPEC-003 / FEAT-12.SPEC-004 | Inbound | FEAT-10 (Invoice Payment Processing) | Payment success, failure, and the resulting Paid/Payment-pending status feed the totals | A payment succeeds, fails, or is recorded manually |
| FEAT-12.SPEC-003 | Inbound | FEAT-01 (Client & Project Management) | Client and project roster and archive status determine which totals and drill-down entries appear | Nadia's client/project roster changes |
| FEAT-12.SPEC-003 | Inbound | FEAT-15 (Currency & Tax Handling) | Each project's assigned billing currency determines how totals are segmented | A client/project's currency is set before its first invoice |
| FEAT-12.SPEC-001 / FEAT-12.SPEC-002 | Inbound | FEAT-31 (Support Access) | Dana's logged, read-only support session views the dashboard and drill-down | Dana opens a support session on Nadia's account |

## Non-Functional Notes

**Data volumes / growth:** The dashboard aggregates across a freelancer's full client roster — 3–15 active clients typical, a few thousand freelancers expected in year one (ASMP-22, SC-21) — with no hard limit on history depth (Validation & Limits); the aggregation and drill-down are designed to stay responsive at this scale from MVP onward.

**Responsiveness:** Dashboard totals appear within roughly 1–2 seconds on a typical connection (ASMP-21); the Dashboard Comprehension success metric targets Nadia stating her total earned, outstanding, and overdue amounts within 10 seconds of opening the dashboard, so the aggregation must render before that window elapses; a lightweight progress indicator covers the gap while many clients are aggregated (feature's States field, ASMP-27).

**Data sensitivity / privacy:** Totals are derived entirely from Invoice and Payment records, which carry GDPR-class personal data (client billing name and address, payer identity) per the dependency map's Data Sensitivity lines; the dashboard itself displays no card data (no card numbers or bank credentials are ever captured, ASMP-24/SC-10) and is strictly isolated to Nadia's own account — no client contact ever sees another client's totals or the freelancer's aggregate view (Access field, ASMP-23).

**Compliance flags:** Any operator (Dana) access to this view is read-only, time-limited, and logged in the freelancer's trail (ASMP-23, Access field); financial-record retention (SC-24) governs how long the underlying Invoice and Payment records this feature reads remain available, which this feature has no control over and simply reflects.

## Non-Goals

- **Live, two-way accounting sync from the dashboard** -- Excluded per scope-boundaries.md (SC-07): the brief provides an export file (FEAT-22), not a live sync; this feature's totals stay a read-only view with no accounting-system connection of its own.
- **Converting or summing totals across currencies into one combined figure** -- Excluded per the feature's own Validation & Limits field and XBR-18: amounts in different currencies are never silently converted or added together, even though a single combined number would be a natural dashboard adjacency; each currency's totals are shown separately, always.
- **Recording, editing, or refunding a payment from the dashboard** -- Excluded per the dependency map: Invoice and Payment are Connected Entities marked read-only for this feature; payment recording and refunds are owned exclusively by FEAT-10 and FEAT-25, reachable only by navigating away from the dashboard into invoice/payment detail.
- **Automatic tax calculation or reporting from the dashboard's totals** -- Excluded per scope-boundaries.md (SC-16): tax depth is limited to the freelancer-configured tax label and rate captured elsewhere (FEAT-15); this feature surfaces totals only and performs no tax computation of its own.



# Screen Spec: Dashboard Overview

## Overview

**Name:** Dashboard Overview
**ID:** FEAT-12.SPEC-001
**Type:** Screen
**Purpose:** Nadia sees earned, outstanding, and overdue totals across her entire client roster at a glance, per currency, filterable by client or period, and can drill into any client for detail.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard

## Scope and Non-Goals

**In Scope:**
- Displaying aggregate earned/outstanding/overdue totals across every client, one self-contained block per currency present
- Narrowing the displayed totals to one client or a date range (period)
- A status badge/color treatment on the aggregate totals distinguishing paid, due, and overdue amounts
- Entry point into the per-client/project drill-down (FEAT-12.SPEC-002)
- Entry point into an individual overdue invoice's reminder history and pause control (FEAT-11, cross-feature)
- Entry point into the accounting export flow (FEAT-22, cross-feature)
- The new-account zero-state and its prompt toward a first proposal (FEAT-02, cross-feature)
- Read-only viewing for Dana (Support Operator) during a logged support session

**Non-Goals:**
- Computing the totals themselves -- delegated entirely to FEAT-12.SPEC-003 (Financial Totals Aggregation); this screen only renders what that rule returns
- Converting or summing totals across currencies into one combined figure -- excluded per the feature's Validation & Limits and XBR-18: each currency's totals are shown separately, always, never combined
- Recording, editing, or refunding a payment from this screen -- excluded per the dependency map: Invoice and Payment are read-only Connected Entities for this feature; those actions belong exclusively to FEAT-10 and FEAT-25, reachable only by navigating away from the dashboard
- Automatic tax calculation or reporting -- excluded per scope-boundaries.md (SC-16): this screen surfaces totals only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry (main navigation) | Nadia opens the "Financial Dashboard" / "Dashboard" navigation item | None -- loads with no filter (All Clients, All Time) |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding completes: a first client, project, and draft proposal exist | None -- lands on the normal (non-zero-state) dashboard per the Navigation Connections table |
| FEAT-31 (Operator Support Access) | Dana opens a logged support session on Nadia's account | Session is read-only and scoped to the one freelancer account being supported |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Nadia taps the back arrow | The client/period filter active before drilling in is restored |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | All actions: filter by client/period, drill into a client, open an overdue invoice, generate an accounting export, dismiss the zero-state | -- |
| Owen (Client Primary Contact) | No | No | Owen signs in only to the Client Portal (FEAT-05), which has no route to this screen; the Financial Dashboard exists solely within the freelancer's own account. A portal link manipulated to reach this address is rejected and Owen is redirected to his portal home (FEAT-05.SPEC-003) with no message indicating a freelancer-only screen exists |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya's portal session has no route here and is redirected identically on any manipulated attempt |
| Dana (Support Operator) | Full screen, for the one freelancer account under an open support session; the export action is not shown | Filter by client/period, drill into a client, open an overdue invoice's reminder history; cannot generate an accounting export | The export control is not rendered for Dana. A direct navigation attempt into the export flow (FEAT-22) while in a support session is blocked with "Support sessions cannot generate exports." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, Nadia lands on this screen (or Dana's session resumes its scoped view) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- since this screen has no unsaved input, nothing is preserved beyond the active filter selection, which is restored after re-authentication succeeds |

## Layout and Content

**Header:** Page title "Financial Dashboard." Below the title, two filter controls sit side by side: a Client filter (dropdown, default "All Clients," listing Nadia's active client roster from FEAT-01) and a Period filter (dropdown, default "All Time," with presets including "This Month" and "Last Month," plus a "Custom Range" option that opens a start/end date picker). To the right of the filters, an "Export" action button (Nadia only; hidden for Dana per Access and Visibility) navigates to the accounting export flow.

**Body:** Below the header, one self-contained total block per currency present across the filtered scope (never a combined figure across currencies, per XBR-18). Each currency's block shows three figures side by side -- Earned, Outstanding, Overdue -- each with its amount and the status color/badge treatment shared with FEAT-12.SPEC-002, driven by the classification FEAT-12.SPEC-003 produces.

Below the currency blocks, a "By Client" list: one row per client with financial activity in the filtered scope, each row showing that client's name and its own earned/outstanding/overdue mini-totals with the same badge treatment. A client row with at least one overdue invoice carries a distinct overdue indicator that opens directly into that invoice's reminder history and pause control.

**Footer:** None -- filters and the export action are in the header.

### Responsive Behavior

- **Compact breakpoint:** Client and Period filters stack vertically, full width, above the currency total blocks. Each currency block's three figures (Earned, Outstanding, Overdue) stack vertically within the block rather than sitting side by side. The By Client list remains a single column of full-width rows.
- **Medium size class and above:** Client and Period filters sit side by side as described. Each currency block's three figures sit side by side within the block. The By Client list remains single-column but each row gains more horizontal breathing room; no structural change beyond width.
- **Export button:** Always in the header regardless of breakpoint; at the compact breakpoint it collapses to an icon-only control with the same label available to assistive technology.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Client filter | Select a client (or "All Clients") | Requests totals from FEAT-12.SPEC-003 scoped to the selection | Screen enters Loading, then re-renders with filtered totals | Progress indicator during aggregation; totals and By Client list update to the new scope |
| Period filter | Select a preset or Custom Range | Requests totals from FEAT-12.SPEC-003 scoped to the selection | Screen enters Loading, then re-renders with filtered totals | Same as Client filter |
| Custom Range date pickers | Enter start/end dates | Applies the custom period once both dates are set | Screen enters Loading on confirmation | Same as Client filter; an incomplete range (only one date set) does not trigger a refetch |
| By Client list row | Tap | Navigate to FEAT-12.SPEC-002 (Client/Project Financial Drill-down) for that client | Screen closes | Transition to the drill-down screen for the tapped client |
| Overdue indicator on a client row | Tap | Navigate to that client's overdue invoice's reminder history and pause control (FEAT-11, cross-feature) | Screen closes | Transition to FEAT-11's invoice detail |
| Export button (Nadia only) | Tap | Navigate to the accounting export flow (FEAT-22, cross-feature) | Screen closes | Transition to FEAT-22 |
| Retry button (Error state) | Tap | Triggers FEAT-12.SPEC-004 (Dashboard Totals Refresh) to recompute now | Button shows a brief loading indicator | Success: totals update silently. Failure: banner remains, "Still unable to refresh. Showing your last known totals." |
| Clear Filter button (No-Results state) | Tap | Resets Client and Period filters to their defaults | Screen enters Loading, then re-renders with account-wide totals | Filters visibly reset; totals reload |
| Zero-state "Send your first proposal" prompt | Tap | Navigate to a new proposal draft (FEAT-02, cross-feature) | Screen closes | Transition to FEAT-02's proposal draft editor |
| Currency total blocks | -- | Display-only -- not interactive | None | -- |

### Accessibility Notes

- **Focus order:** Client filter -> Period filter -> (Custom Range date pickers, when shown) -> Export button -> each currency block's three figures in order (Earned, Outstanding, Overdue) -> each By Client row in list order -> Retry/Clear Filter button, when shown.
- **Dynamic announcements:** Entering the Loading state announces "Updating totals" to assistive technology. A successful refresh announces the new Earned/Outstanding/Overdue figures for the active scope. Entering the Error state announces the retry banner text. Entering the No-Results state announces the no-results message.
- **Status badges:** Paid/Due/Overdue status is never conveyed by color alone (ASMP-27) -- each badge carries a text label read by assistive technology alongside its color treatment.
- **Keyboard alternatives:** Every filter, list row, and button on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (zero-state) | Explanatory message that no invoices exist yet, with a prompt and action toward sending a first proposal; no currency blocks or By Client list shown | Screen loads and Nadia's account has never had any invoice in any status | Nadia's first invoice is generated (via any triggering path in FEAT-09) |
| Loading | Currency blocks and By Client list area show a lightweight progress indicator; filters remain interactive but disabled until the in-flight request completes | Screen first opens, or a filter selection changes, or FEAT-12.SPEC-004 recomputes for the active scope | Aggregation completes (success, no-results, or error) |
| Loaded (default) | Currency blocks and By Client list render the current totals for the active filter scope | Aggregation completes successfully with at least one matching total | A filter changes, an underlying invoice/payment changes elsewhere (silent background refresh via FEAT-12.SPEC-004), or connectivity is lost |
| No-Results | Distinct message: "No invoices match this filter." with a "Clear Filter" action; no currency blocks shown | The active Client/Period filter matches zero invoices, while the account overall has invoices | Filter is cleared or changed to a scope with matches |
| Error | The last successfully computed totals for the active scope remain visible, with a banner "Couldn't refresh your totals just now." and a Retry action | FEAT-12.SPEC-003's aggregation fails and FEAT-12.SPEC-004's automatic retries are exhausted for the active scope | Retry succeeds, or a filter change triggers a fresh aggregation attempt |
| Offline/Degraded | The most recently loaded totals for whichever filter was active when connectivity was lost remain visible, read-only, with a persistent banner "You're offline -- showing your last loaded totals." Changing the filter shows the same banner and does not attempt a new fetch; the newly selected filter is remembered | Connectivity is lost while this screen is open | Connectivity is restored -- the remembered filter (whether unchanged or changed while offline) is fetched automatically and the banner clears |

## Validation Rules

**Option B -- Inline (for simple validations not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Client filter | Must be a client from Nadia's own roster (FEAT-01) or "All Clients" | On selection | N/A -- the control only ever offers valid options; no free text entry is possible |
| Period filter | Must be a defined preset or a valid Custom Range | On selection | N/A -- presets are fixed options |
| Custom Range: end date | Must not be earlier than the start date | On selection of the end date | "End date can't be before the start date." |

No user input is captured for the totals themselves (per the feature's Validation & Limits field) -- every value on this screen is derived by FEAT-12.SPEC-003.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| By Client row tap | FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | -- |
| Overdue indicator tap | Invoice reminder history and pause control | FEAT-11 (Automated Payment Reminders) |
| Export button tap | Accounting export flow | FEAT-22 (Accounting Export) |
| Zero-state prompt tap | New proposal draft | FEAT-02 (Proposal Creation & Sending) |

## Data Model

**Creates:** None -- this screen creates no data.
**Reads:** The computed Financial Totals produced by FEAT-12.SPEC-003 (earned/outstanding/overdue amounts per currency, per-client breakdowns, and the paid/due/overdue classification). Indirectly, via that rule: Invoice (status, amount, currency, due_date, project), Payment, and Project (project_name, client, currency) as defined in the Feature Dependency Map. Reads Client roster (FEAT-01) to populate the Client filter's options.
**Updates:** None.
**Deletes:** None.

## Business Rules

- All totals shown on this screen are computed exclusively by FEAT-12.SPEC-003; this screen never derives a figure independently, and never re-implements the earned/outstanding/overdue formula, the currency-separation rule, or the classification.
- XBR-18: amounts in different currencies are never converted or summed into one figure -- each currency renders in its own self-contained block.
- The zero-state, loading indicator, error/retry behavior, no-results messaging, and offline-degraded read-only viewing follow the shared convention declared in the Feature Breakdown Brief's Shared Context, identical in wording and behavior to FEAT-12.SPEC-002.
- The Export action is visible only to Nadia; Dana's read-only support session (FEAT-31, XBR-29) never exposes export generation, consistent with the Access Matrix.
- An invoice or payment change elsewhere in the product does not require Nadia to manually refresh -- FEAT-12.SPEC-004 recomputes affected totals in the background and this screen reflects the update silently if it is open and viewing the affected scope.

## Edge Cases

- **An underlying invoice or payment changes while Nadia is viewing this screen** -- No conflict: this screen only reads. FEAT-12.SPEC-004 recomputes the affected totals in the background and this screen's Loaded state re-renders the new figures without disrupting the active filter selection or list scroll position.
- **A filter (client or period) matches no invoices** -- The No-Results state appears with a "Clear Filter" action, distinct from the true zero-state message.
- **Aggregation across many clients takes noticeable time** -- The Loading state's lightweight progress indicator covers the gap; the Dashboard Comprehension success metric's 10-second target governs how quickly this must resolve.
- **Nadia double-taps the Export button** -- The second tap is ignored while navigation to FEAT-22 is already in progress.
- **Nadia navigates away and back** -- The screen re-fetches totals for whatever filter was last active rather than showing stale cached data, except when returning directly from FEAT-12.SPEC-002 via its back arrow, where the prior filter is restored without a full re-fetch if the data is still current.
- **A client the totals reference is archived (FEAT-01) while this screen is open** -- The archived client's historical totals remain included in the By Client list (archiving never erases records); no error occurs.
- **Custom Range with only a start date entered** -- No refetch is triggered until an end date is also set; the screen remains in its previous Loaded state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Navigation (outbound) | Tapping a By Client row opens that client's detail |
| FEAT-12.SPEC-003 (Financial Totals Aggregation) | References (outbound) | This screen's every displayed figure is sourced from this rule |
| FEAT-12.SPEC-004 (Dashboard Totals Refresh) | Triggers (outbound) | The Retry action, and any background recompute while this screen is open, is handled by this automation |
| FEAT-01 (Client & Project Management) | References (inbound) | Client roster populates the Client filter; project grouping feeds the By Client list |
| FEAT-11 (Automated Payment Reminders) | Navigation (outbound) | Opening an overdue invoice from a client row |
| FEAT-22 (Accounting Export) | Navigation (outbound) | Export button |
| FEAT-02 (Proposal Creation & Sending) | Navigation (outbound) | Zero-state prompt |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Onboarding completion lands Nadia here |
| FEAT-31 (Operator Support Access) | References (inbound) | Dana's read-only support session views this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dashboard_viewed | currency_count_shown, client_count_in_scope, aggregation_duration_ms | This screen finishes loading (Loaded state entered) | supports success-metrics.md: "Dashboard Comprehension" |
| dashboard_filtered | filter_type (client / period), aggregation_duration_ms | Nadia changes the Client or Period filter and the new totals load | supports success-metrics.md: "Dashboard Comprehension" |
| dashboard_empty_state_shown | -- | The zero-state renders | N/A -- this event measures the new-account onboarding funnel; no metric in success-metrics.md targets a zero-state account, since the Dashboard Comprehension metric is defined for an account with existing financial data |
| dashboard_no_results_shown | filter_type (client / period) | The No-Results state renders | N/A -- no Stage 2 metric targets filtered-empty outcomes specifically; this event supports usability monitoring of the filter controls |

## Acceptance Criteria

**FEAT-12.SPEC-001-AC-01:** Given Nadia opens the Financial Dashboard for the first time with an account that has never had any invoice, then she sees the zero-state message with a prompt to send her first proposal, and no currency blocks are shown.

**FEAT-12.SPEC-001-AC-02:** Given Nadia has invoices across three clients in one currency, when she opens the Financial Dashboard, then a single currency block appears showing Earned, Outstanding, and Overdue totals aggregated across all three clients, and the By Client list shows one row per client with financial activity.

**FEAT-12.SPEC-001-AC-03:** Given Nadia has invoices in two different currencies, when she opens the Financial Dashboard, then two separate self-contained currency blocks appear, and no figure ever combines amounts from both currencies.

**FEAT-12.SPEC-001-AC-04:** Given Nadia is viewing the dashboard with "All Clients" selected, when she selects a specific client in the Client filter, then the screen shows a brief loading indicator and re-renders with totals scoped to that client only.

**FEAT-12.SPEC-001-AC-05:** Given Nadia selects "Last Month" in the Period filter, when the selection completes, then the currency blocks and By Client list update to reflect only invoices relevant to last month.

**FEAT-12.SPEC-001-AC-06:** Given Nadia filters to a client with no invoices in the selected period, when the filtered totals load, then the No-Results state appears with the message "No invoices match this filter." and a "Clear Filter" action, distinct from the zero-state.

**FEAT-12.SPEC-001-AC-07:** Given Nadia is viewing the No-Results state, when she taps "Clear Filter", then both filters reset to "All Clients" / "All Time" and the account-wide totals load.

**FEAT-12.SPEC-001-AC-08:** Given Nadia taps a client row in the By Client list, when the tap registers, then she is navigated to FEAT-12.SPEC-002 (Client/Project Financial Drill-down) for that client.

**FEAT-12.SPEC-001-AC-09:** Given a client row shows an overdue indicator, when Nadia taps it, then she is navigated to that overdue invoice's reminder history and pause control (FEAT-11).

**FEAT-12.SPEC-001-AC-10:** Given Nadia taps the Export button, when the tap registers, then she is navigated to the accounting export flow (FEAT-22).

**FEAT-12.SPEC-001-AC-11:** Given Dana (Support Operator) is viewing the dashboard during a logged support session, when she looks for the Export button, then it is not shown, and a direct navigation attempt into the export flow is blocked with "Support sessions cannot generate exports."

**FEAT-12.SPEC-001-AC-12:** Given aggregation for the active scope fails and FEAT-12.SPEC-004's automatic retries are exhausted, when the Error state renders, then the last successfully computed totals remain visible alongside the banner "Couldn't refresh your totals just now." and a Retry action -- never a blank or misleading number.

**FEAT-12.SPEC-001-AC-13:** Given Nadia is viewing the Error state, when she taps Retry and the recomputation succeeds, then the banner clears and the refreshed totals render.

**FEAT-12.SPEC-001-AC-14:** Given Nadia loses connectivity while viewing the dashboard, when the connection drops, then the banner "You're offline -- showing your last loaded totals." appears and the most recently loaded totals remain viewable, read-only.

**FEAT-12.SPEC-001-AC-15:** Given Nadia is offline and changes the Client filter, when the selection is made, then no new fetch is attempted, the offline banner remains, and the newly selected filter is remembered for automatic fetch once connectivity returns.

**FEAT-12.SPEC-001-AC-16:** Given an invoice underlying Nadia's currently displayed totals is marked Paid elsewhere while this screen is open, when FEAT-12.SPEC-004 recomputes the affected totals, then the displayed figures update silently without disrupting Nadia's active filter selection.

**FEAT-12.SPEC-001-AC-17:** Given Nadia with no invoices taps the zero-state prompt, when the tap registers, then she is navigated to a new proposal draft (FEAT-02).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 6 (empty, loading, loaded, no-results, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Screen Spec: Client/Project Financial Drill-down

## Overview

**Name:** Client/Project Financial Drill-down
**ID:** FEAT-12.SPEC-002
**Type:** Screen
**Purpose:** Nadia opens one client to see the specific invoices behind its aggregate totals, and can narrow further to a single project within that client.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard

## Scope and Non-Goals

**In Scope:**
- Showing one client's earned/outstanding/overdue totals, per currency, matching the aggregate shown on FEAT-12.SPEC-001
- Narrowing within the client to a single project
- Listing the specific invoices behind the client's (or project's) totals, each with its paid/due/overdue status
- Entry point into an individual invoice's detail (FEAT-09, cross-feature) and into an overdue invoice's reminder history and pause control (FEAT-11, cross-feature)
- Read-only viewing for Dana (Support Operator) during a logged support session

**Non-Goals:**
- Computing the totals themselves -- delegated entirely to FEAT-12.SPEC-003 (Financial Totals Aggregation), identical to FEAT-12.SPEC-001
- Editing, sending, or voiding a proposal or invoice from this screen -- excluded per the dependency map: this feature's Connected Entities (Invoice, Payment, Project) are all read-only; those actions belong to FEAT-02 and FEAT-09
- Recording a payment, refund, or reversal from this screen -- excluded per the dependency map: Payment is a read-only Connected Entity here; those actions belong exclusively to FEAT-10 and FEAT-25
- Converting or summing this client's totals across currencies -- excluded per XBR-18, identical to FEAT-12.SPEC-001

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Nadia taps a By Client row | The selected client, and the Period filter active on FEAT-12.SPEC-001 at the time |
| FEAT-31 (Operator Support Access) | Dana opens a logged support session and views a client's detail from the dashboard | Session is read-only and scoped to the one freelancer account being supported |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | Narrow to a project, open an invoice, open an overdue invoice's reminder history | -- |
| Owen (Client Primary Contact) | No | No | Owen signs in only to the Client Portal (FEAT-05), which has no route to this screen; a portal link manipulated to reach this address is rejected and Owen is redirected to his portal home (FEAT-05.SPEC-003) with no message indicating a freelancer-only screen exists |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya's portal session has no route here and is redirected identically on any manipulated attempt |
| Dana (Support Operator) | Full screen, for the one freelancer account under an open support session | Narrow to a project, open an invoice, open an overdue invoice's reminder history -- all read-only | -- (this screen has no export control to deny; export is confined to FEAT-12.SPEC-001) |
| Unauthenticated | No | No | Redirected to the sign-in screen; the client context is not preserved across the redirect for security, and Nadia returns to FEAT-12.SPEC-001 after signing in |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- since this screen has no unsaved input, the selected client and project narrowing are restored after re-authentication succeeds |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12.SPEC-001, restoring its prior filter). Client name as the screen title. Below the title, a "Period: {value}" chip carried from FEAT-12.SPEC-001 (or "All Time" if none was active), adjustable inline via the same preset/custom-range control described in that screen. A Project filter (dropdown, default "All Projects" for this client) sits alongside it.

**Body:** Below the header, one self-contained total block per currency present within the client's (or narrowed project's) scope -- Earned, Outstanding, Overdue -- using the same status color/badge treatment as FEAT-12.SPEC-001, per the Shared UI Patterns declared in the Feature Breakdown Brief. Below the totals, an invoice list: one row per invoice in scope, each showing the invoice's project name, amount and currency, due date, and a paid/due/overdue status badge driven by FEAT-12.SPEC-003's classification. A row for an Overdue invoice carries the same overdue indicator used on FEAT-12.SPEC-001.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** The Period chip and Project filter stack vertically below the client name. Each currency block's three figures stack vertically. Invoice list rows show project name and status badge on one line, amount/currency and due date on a second line within the same row.
- **Medium size class and above:** The Period chip and Project filter sit side by side. Each currency block's three figures sit side by side. Invoice list rows show all fields on a single line; no structural change beyond width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001, restoring the filter active before drilling in | Screen closes | Animated transition back to the dashboard |
| Period chip | Select a new preset or Custom Range | Requests totals from FEAT-12.SPEC-003 scoped to this client and the new period | Screen enters Loading, then re-renders | Progress indicator during aggregation; totals and invoice list update |
| Project filter | Select a project (or "All Projects") | Requests totals from FEAT-12.SPEC-003 scoped to this client and the selected project | Screen enters Loading, then re-renders | Same as Period chip |
| Invoice row (not overdue) | Tap | Navigate to that invoice's detail (FEAT-09, cross-feature) | Screen closes | Transition to FEAT-09's invoice detail |
| Overdue indicator on an invoice row | Tap | Navigate to that invoice's reminder history and pause control (FEAT-11, cross-feature) | Screen closes | Transition to FEAT-11's invoice detail |
| Retry button (Error state) | Tap | Triggers FEAT-12.SPEC-004 (Dashboard Totals Refresh) to recompute now | Button shows a brief loading indicator | Success: totals update silently. Failure: banner remains |
| Clear Filter button (No-Results state) | Tap | Resets the Project filter to "All Projects" (Period chip is left as carried from FEAT-12.SPEC-001) | Screen enters Loading, then re-renders | Filter visibly resets; totals reload |
| Currency total blocks | -- | Display-only -- not interactive | None | -- |

### Accessibility Notes

- **Focus order:** Back arrow -> Period chip -> Project filter -> each currency block's three figures in order -> each invoice row in list order -> Retry/Clear Filter button, when shown.
- **Dynamic announcements:** Entering Loading announces "Updating totals" to assistive technology. A successful refresh announces the client's updated Earned/Outstanding/Overdue figures. Entering Error announces the retry banner text. Entering No-Results announces the no-results message.
- **Status badges:** Paid/Due/Overdue status is never conveyed by color alone (ASMP-27) -- each badge carries a text label alongside its color treatment, identical to FEAT-12.SPEC-001.
- **Keyboard alternatives:** Every filter, row, and button on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Currency blocks and invoice list area show a lightweight progress indicator; filters remain visible but disabled until the in-flight request completes | Screen first opens, or the Period chip/Project filter changes, or FEAT-12.SPEC-004 recomputes for the active scope | Aggregation completes (success, no-results, or error) |
| Loaded (default) | Currency blocks and invoice list render the current totals for the active client/project/period scope | Aggregation completes successfully with at least one matching invoice | A filter changes, an underlying invoice/payment changes elsewhere (silent background refresh via FEAT-12.SPEC-004), or connectivity is lost |
| No invoices for this client yet | Message "This client has no invoices yet." with no currency blocks or invoice rows shown | The selected client (with "All Projects," "All Time") genuinely has zero invoices in any status | An invoice is generated for this client (via any triggering path in FEAT-09) |
| No-Results | Distinct message "No invoices match this filter." with a "Clear Filter" action | The active Project/Period scope matches zero invoices while the client overall has invoices | Filter is cleared or changed to a scope with matches |
| Error | The last successfully computed totals for the active scope remain visible, with a banner "Couldn't refresh your totals just now." and a Retry action | FEAT-12.SPEC-003's aggregation fails and FEAT-12.SPEC-004's automatic retries are exhausted for the active scope | Retry succeeds, or a filter change triggers a fresh aggregation attempt |
| Offline/Degraded | The most recently loaded totals and invoice list for the active scope remain visible, read-only, with a persistent banner "You're offline -- showing your last loaded totals." Changing a filter shows the same banner and does not attempt a new fetch | Connectivity is lost while this screen is open | Connectivity is restored -- the remembered filter is fetched automatically and the banner clears |

## Validation Rules

**Option B -- Inline (for simple validations not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Project filter | Must be a project belonging to this client (FEAT-01) or "All Projects" | On selection | N/A -- the control only ever offers valid options |
| Period chip | Same rule as FEAT-12.SPEC-001's Period filter, including the end-date-not-before-start-date check for Custom Range | On selection | "End date can't be before the start date." |

No user input is captured for the totals or invoice list themselves -- every value on this screen is derived by FEAT-12.SPEC-003.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Dashboard Overview) | -- |
| Invoice row tap (not overdue) | Invoice detail | FEAT-09 (Invoice Generation & Sending) |
| Overdue indicator tap | Invoice reminder history and pause control | FEAT-11 (Automated Payment Reminders) |

## Data Model

**Creates:** None -- this screen creates no data.
**Reads:** The computed Financial Totals produced by FEAT-12.SPEC-003 (earned/outstanding/overdue amounts per currency, per-project breakdown, and the paid/due/overdue classification), scoped to the selected client. Indirectly, via that rule: Invoice (status, amount, currency, due_date, project), Payment, and Project (project_name, client, currency) as defined in the Feature Dependency Map. Reads the client's project roster (FEAT-01) to populate the Project filter's options.
**Updates:** None.
**Deletes:** None.

## Business Rules

- All totals and the invoice-level status badges on this screen are computed exclusively by FEAT-12.SPEC-003; this screen never re-derives the earned/outstanding/overdue formula or the currency-separation rule.
- XBR-18: amounts in different currencies within this client are never converted or summed into one figure -- each currency renders in its own self-contained block, identical to FEAT-12.SPEC-001.
- The loading indicator, error/retry behavior, no-results messaging, and offline-degraded read-only viewing follow the shared convention declared in the Feature Breakdown Brief's Shared Context, identical in wording and behavior to FEAT-12.SPEC-001.
- This screen has no export control -- accounting export is confined to FEAT-12.SPEC-001, consistent with the Access Matrix.
- An invoice or payment change elsewhere in the product does not require Nadia to manually refresh -- FEAT-12.SPEC-004 recomputes affected totals in the background and this screen reflects the update silently if it is open and viewing the affected client.

## Edge Cases

- **An underlying invoice or payment changes while Nadia is viewing this client's detail** -- No conflict: this screen only reads. FEAT-12.SPEC-004 recomputes the affected totals in the background and this screen's Loaded state re-renders the new figures without disrupting the active project/period selection.
- **A filter (project or period) matches no invoices** -- The No-Results state appears with a "Clear Filter" action, distinct from the "no invoices for this client yet" message.
- **The client is archived (FEAT-01) while Nadia is viewing its drill-down** -- The client's historical totals and invoice list remain visible read-only (archiving never erases records); no error occurs and no banner is required beyond what FEAT-01's own client detail shows.
- **Nadia navigates directly to a project's invoices, then that project is marked complete or cancelled elsewhere (FEAT-01/FEAT-25)** -- The project's stage label updates silently on the next refresh; its invoices and their statuses remain visible and unaffected by the stage change itself.
- **Nadia double-taps an invoice row** -- The second tap is ignored while navigation to FEAT-09 is already in progress.
- **Nadia narrows to a project, then Nadia's filter is manually cleared while a background recompute triggered by the prior scope is still in flight** -- The in-flight recompute's result is discarded on arrival if it no longer matches the current filter scope; only a recompute matching the currently selected scope updates the displayed totals.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Navigation (inbound) | Nadia arrives here from a By Client row |
| FEAT-12.SPEC-003 (Financial Totals Aggregation) | References (outbound) | This screen's every displayed figure and status badge is sourced from this rule |
| FEAT-12.SPEC-004 (Dashboard Totals Refresh) | Triggers (outbound) | The Retry action, and any background recompute while this screen is open, is handled by this automation |
| FEAT-01 (Client & Project Management) | References (inbound) | Project roster populates the Project filter |
| FEAT-09 (Invoice Generation & Sending) | Navigation (outbound) | Opening a specific invoice from the invoice list |
| FEAT-11 (Automated Payment Reminders) | Navigation (outbound) | Opening an overdue invoice's reminder history |
| FEAT-31 (Operator Support Access) | References (inbound) | Dana's read-only support session views this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_drilldown_viewed | invoice_count_in_scope, currency_count_shown, entry_source (dashboard_row) | This screen finishes loading (Loaded state entered) | N/A -- the drill-down view is a secondary action after the moment the Dashboard Comprehension metric targets (Nadia stating her totals within 10 seconds of opening the dashboard, FEAT-12.SPEC-001); no metric in success-metrics.md targets drill-down usage specifically |
| client_drilldown_filtered | filter_type (project / period) | Nadia changes the Project or Period filter and the new totals load | N/A -- same reason as client_drilldown_viewed |

## Acceptance Criteria

**FEAT-12.SPEC-002-AC-01:** Given Nadia taps a client row on FEAT-12.SPEC-001, when this screen loads, then it shows that client's earned/outstanding/overdue totals per currency and a list of that client's invoices with paid/due/overdue status badges.

**FEAT-12.SPEC-002-AC-02:** Given Nadia is viewing a client's drill-down with "All Projects" selected, when she selects one specific project in the Project filter, then the totals and invoice list narrow to that project only.

**FEAT-12.SPEC-002-AC-03:** Given Nadia opens the drill-down for a client that has never had an invoice, then she sees "This client has no invoices yet." with no currency blocks or invoice rows.

**FEAT-12.SPEC-002-AC-04:** Given Nadia narrows to a project with no invoices in the carried-over period, when the filtered totals load, then the No-Results state appears with "No invoices match this filter." and a "Clear Filter" action.

**FEAT-12.SPEC-002-AC-05:** Given Nadia is viewing the No-Results state, when she taps "Clear Filter", then the Project filter resets to "All Projects" and the client's totals for that period reload.

**FEAT-12.SPEC-002-AC-06:** Given Nadia taps an invoice row that is not overdue, when the tap registers, then she is navigated to that invoice's detail (FEAT-09).

**FEAT-12.SPEC-002-AC-07:** Given an invoice row shows an overdue indicator, when Nadia taps it, then she is navigated to that invoice's reminder history and pause control (FEAT-11).

**FEAT-12.SPEC-002-AC-08:** Given Nadia taps the back arrow, when the tap registers, then she returns to FEAT-12.SPEC-001 with the filter that was active before she drilled in restored.

**FEAT-12.SPEC-002-AC-09:** Given a client has invoices in two currencies, when Nadia views its drill-down, then two separate self-contained currency blocks appear and no figure combines amounts from both.

**FEAT-12.SPEC-002-AC-10:** Given aggregation for the active scope fails and FEAT-12.SPEC-004's automatic retries are exhausted, when the Error state renders, then the last successfully computed totals remain visible alongside a Retry action -- never a blank or misleading number.

**FEAT-12.SPEC-002-AC-11:** Given Nadia loses connectivity while viewing this client's drill-down, when the connection drops, then the banner "You're offline -- showing your last loaded totals." appears and the most recently loaded totals and invoice list remain viewable, read-only.

**FEAT-12.SPEC-002-AC-12:** Given Dana (Support Operator) is viewing a client's drill-down during a logged support session, when she looks at the screen, then all actions available to her are read-only and no export control is present.

**FEAT-12.SPEC-002-AC-13:** Given an invoice underlying this client's currently displayed totals is refunded elsewhere while this screen is open, when FEAT-12.SPEC-004 recomputes the affected totals, then the displayed figures update silently without disrupting Nadia's active project/period selection.

**FEAT-12.SPEC-002-AC-14:** Given the client Nadia is viewing is archived (FEAT-01) while this screen is open, when the archive completes, then the client's historical totals and invoices remain visible, read-only, with no error shown.

**FEAT-12.SPEC-002-AC-15:** Given Nadia clears her project filter while an earlier background recompute for the previously narrowed project is still in flight, when that stale recompute's result arrives, then it is discarded because it no longer matches the currently selected scope.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loading, loaded, empty-for-client, no-results, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Financial Totals Aggregation

## Overview

**Name:** Financial Totals Aggregation
**ID:** FEAT-12.SPEC-003
**Type:** Logic/Rule
**Purpose:** Derives earned, outstanding, and overdue totals -- and the paid/due/overdue status classification behind them -- from Invoice and Payment records, per currency, honoring refunds, reversals, and manually recorded payments.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard
**Governed Entity:** Financial Totals (a derived aggregate view -- not a stored entity; computed entirely from the Invoice and Payment records defined in the Feature Dependency Map, per the feature's Data Notes: "nothing is captured directly in this view")

## Scope and Non-Goals

**In Scope:**
- The derivation formula for earned, outstanding, and overdue totals, per currency, at three scopes: account-wide, per-client, and per-project
- The paid/due/overdue classification applied to each invoice, which drives both the aggregate figures and the per-invoice status badges on FEAT-12.SPEC-001 and FEAT-12.SPEC-002
- Currency-separation rules (XBR-18)
- Honoring refunds, partial refunds, reversals, disputes, and manually recorded payments in the earned figure (XBR-20, XBR-21, XBR-22)
- Which invoice and payment statuses are excluded from the totals entirely

**Non-Goals:**
- Rendering the totals on screen -- owned by FEAT-12.SPEC-001 and FEAT-12.SPEC-002, which reference this spec rather than re-deriving the formula
- Recomputing on a schedule or in response to an underlying change, and retrying on failure -- owned by FEAT-12.SPEC-004 (Dashboard Totals Refresh); this spec defines only the formula, not when it runs
- Determining the exact refunded or reversed amount on a given Payment -- excluded per the dependency map: Payment's refund/reversal fields are owned and written by FEAT-10 and FEAT-25; this spec reads those authoritative values rather than computing them
- Converting or summing totals across currencies into one combined figure -- excluded per the feature's Validation & Limits and XBR-18; each currency is aggregated and reported entirely separately

## Governed Entity

**Entity:** Financial Totals (derived aggregate view)
**Source:** Feature Dependency Map (computed from Invoice and Payment; scoped using Project)

| Field | Data Type | Description |
|-------|-----------|-------------|
| scope | enum (account / client / project) | Which slice of the roster this computation covers |
| currency | text | The currency this total block is computed for; one Financial Totals record exists per currency present within the scope (never combined, XBR-18) |
| earned_total | derived, number | Sum of amounts actually retained by Nadia across Succeeded and Recorded-manually payments in this scope and currency, net of any refunded or reversed amount |
| outstanding_total | derived, number | Sum of amounts on unpaid invoices in this scope and currency that are not yet past their due date |
| overdue_total | derived, number | Sum of amounts on unpaid invoices in this scope and currency that are past their due date |
| per_client_breakdown | derived, list | The same three totals (earned/outstanding/overdue), per currency, scoped to each individual client -- populates FEAT-12.SPEC-001's By Client list |
| per_project_breakdown | derived, list | The same three totals, per currency, scoped to each individual project within one client -- populates FEAT-12.SPEC-002's project narrowing |
| invoice_status_classification | derived, enum (Paid / Due / Overdue) | The at-a-glance status assigned to each invoice contributing to the totals; drives the status badges shared by FEAT-12.SPEC-001 and FEAT-12.SPEC-002 |
| computed_at | derived, timestamp | When this computation ran; carried forward unchanged by FEAT-12.SPEC-004 when a recompute fails, so the screens can keep showing "last known" totals |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Dashboard Overview | On screen load and on every Client/Period filter change |
| FEAT-12.SPEC-002 | Client/Project Financial Drill-down | On screen load and on every Project/Period filter change |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | On every triggered recompute (invoice/payment/roster change, or a manual Retry from either screen) |

## Field Validation Rules

This entity captures no user input -- every field is entirely derived. Each field is addressed below to confirm it was considered, not accidentally skipped.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| scope | No validation -- always one of account / client / project, set by the requesting screen, never free text | Always | -- | -- | No |
| currency | No validation -- entirely derived from the Project.currency of the invoices in scope; see Defaults and Derivations | Always | -- | -- | No |
| earned_total | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| outstanding_total | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| overdue_total | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| per_client_breakdown | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| per_project_breakdown | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| invoice_status_classification | No validation -- entirely derived per invoice; see Defaults and Derivations | Always | -- | -- | No |
| computed_at | No validation -- entirely derived (current time at successful computation); see Defaults and Derivations | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Currency separation (XBR-18) | currency, earned_total, outstanding_total, overdue_total | Each currency present in the scope produces its own complete Financial Totals record; no field on one currency's record is ever added to or converted into another currency's record | N/A -- this is an internal computation guarantee, never a user-facing error; there is no input path that could violate it |
| Mutually exclusive classification | invoice_status_classification, earned_total, outstanding_total, overdue_total | Each invoice's amount contributes to exactly one of earned_total, outstanding_total, or overdue_total in its currency -- never more than one bucket, and never zero for an invoice that is in scope and not excluded (see Business Rules for the excluded statuses) | N/A -- internal computation guarantee |
| Breakdown-to-aggregate consistency | per_client_breakdown, per_project_breakdown, earned_total, outstanding_total, overdue_total | The account-wide totals for a currency equal the sum of that currency's per_client_breakdown entries; a client's total equals the sum of its per_project_breakdown entries | N/A -- internal computation guarantee |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View account-wide financial totals | Nadia (Freelancer) | Always, her own account only | -- |
| View account-wide financial totals | Owen (Client Primary Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-001's Access and Visibility for the exact experience |
| View account-wide financial totals | Priya (Client Reviewer Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-001's Access and Visibility |
| View account-wide financial totals | Dana (Support Operator) | Always, read-only, only during an open support session (FEAT-31) | -- |
| View per-client/per-project financial detail | Nadia (Freelancer) | Always, her own clients and projects only | -- |
| View per-client/per-project financial detail | Owen (Client Primary Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-002's Access and Visibility |
| View per-client/per-project financial detail | Priya (Client Reviewer Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-002's Access and Visibility |
| View per-client/per-project financial detail | Dana (Support Operator) | Always, read-only, only during an open support session (FEAT-31) | -- |
| Narrow totals by client or period | Nadia (Freelancer) | Always | -- |
| Narrow totals by client or period | Dana (Support Operator) | Always, read-only, only during an open support session | -- |
| Narrow totals by client or period | Owen, Priya | Never | Filter controls not shown -- screen not reachable |
| Generate an accounting export from these totals | Nadia (Freelancer) | Always | -- |
| Generate an accounting export from these totals | Dana (Support Operator) | Never (XBR-29: support sessions exclude data/accounting exports) | Export control is not shown; a direct navigation attempt is blocked with "Support sessions cannot generate exports." (see FEAT-12.SPEC-001) |
| Generate an accounting export from these totals | Owen, Priya | Never | Export control not shown -- screen not reachable |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| currency | The distinct set of Project.currency values among invoices in scope; one Financial Totals record is derived per distinct currency found | Every computation | No |
| invoice_status_classification (per invoice in scope) | "Paid" when Invoice.status is Paid, Refunded, Partially refunded, or Disputed (XBR-21 keeps the Disputed marker alongside the original Paid record). "Overdue" when Invoice.status is Overdue, or Invoice.status is Sent/Payment pending and due_date has passed. "Due" when Invoice.status is Sent or Payment pending and due_date has not yet passed. Invoices with status Generated (not yet sent) or Corrected (superseded by a correcting invoice) are excluded from classification and from every total | On every computation | No |
| earned_total (per currency, per scope) | Sum, over invoices in scope classified "Paid," of the associated Payment.amount for Payments with status Succeeded or Recorded manually, minus the refunded or reversed portion FEAT-10/FEAT-25 record against that Payment (a fully reversed or fully refunded invoice contributes zero net; a partially refunded invoice contributes the remaining net amount) | On every computation | No |
| outstanding_total (per currency, per scope) | Sum, over invoices in scope classified "Due," of Invoice.total | On every computation | No |
| overdue_total (per currency, per scope) | Sum, over invoices in scope classified "Overdue," of Invoice.total | On every computation | No |
| per_client_breakdown / per_project_breakdown | The same earned/outstanding/overdue derivation, re-run with scope narrowed to each individual client or project | On every computation | No |
| computed_at | The current time at the moment a computation completes successfully | On every successful computation only -- left unchanged by FEAT-12.SPEC-004 on a failed recompute | No |

## Business Rules

- XBR-18: financial totals are shown per currency; amounts in different currencies are never converted or added together, at any scope (account, client, or project).
- XBR-20: an invoice is paid once and in full only; a refund cannot exceed the amount paid; these facts are enforced by FEAT-10 and simply reflected here as the net earned amount.
- XBR-21: when a paid invoice is later disputed, it keeps its "Paid" classification alongside the Disputed marker (owned by FEAT-25) unless and until the underlying Payment is actually reported Reversed, at which point the earned figure is reduced accordingly.
- XBR-22: Financial Dashboard totals are derived only from Invoice and Payment records, including refunds, reversals, and manually recorded payments -- no other data source contributes to earned/outstanding/overdue.
- Invoices with status Generated (drafted but never sent) carry no financial commitment visible to a client and are excluded from every total; a Corrected invoice is excluded in favor of the invoice that corrects it, so a correction is never double-counted.
- The "Due" vs. "Overdue" boundary is evaluated against the invoice's due_date in Nadia's own time zone (consistent with FEAT-15's time-zone handling and the day-boundary convention FEAT-11 uses for its own reminder schedule, XBR-15) -- an invoice becomes Overdue at the start of the day after its due date, regardless of whether FEAT-11 has yet set the Overdue status flag, so this rule's own due_date comparison never lags behind the true due date.
- The view spans the freelancer's full client roster with no hard limit on history depth (feature's Validation & Limits); this rule does not truncate or paginate the underlying computation -- pagination, if any, is a rendering concern for the consuming screen.

## Edge Cases

- **An invoice's due_date falls exactly on the current day (in Nadia's time zone)** -- Classified "Due," not "Overdue"; the boundary is exclusive of the due date itself, matching the day-3/day-10 reminder convention elsewhere in the product.
- **An invoice is exactly one day past due_date** -- Classified "Overdue" from the start of that next day, even if FEAT-11 has not yet processed its day-3 reminder or set the Overdue status flag.
- **A Partially refunded invoice** -- Remains classified "Paid"; earned_total includes only the net amount after the recorded partial refund, never the full original payment.
- **A fully Refunded invoice** -- Remains classified "Paid" for badge purposes (the client did pay at some point), but contributes zero to earned_total since the full amount was returned.
- **A Disputed invoice whose Payment has not yet been reported Reversed** -- Classified "Paid," and earned_total still includes the full net amount; only an actual Reversed report changes the figure.
- **A Disputed invoice whose Payment is later reported Reversed** -- earned_total for the affected scope(s) drops by that amount on the next computation; classification remains "Paid" per XBR-21's "alongside the original Paid record."
- **An invoice manually recorded as paid off-platform (FEAT-10)** -- Counted identically to a processor-confirmed payment in earned_total, per XBR-22's explicit inclusion of manually recorded payments.
- **An ad hoc invoice still in Generated status (drafted, not yet sent)** -- Excluded from every total; it becomes eligible for classification only once its status moves to Sent.
- **A corrected invoice and its correcting invoice both exist** -- Only the correcting invoice (whichever status it currently holds) contributes to totals; the original Corrected invoice is excluded to prevent double-counting the same underlying amount.
- **A client has invoices in three different currencies** -- Three separate Financial Totals records are produced for that client, one per currency; per_client_breakdown never merges them.
- **A project has zero invoices in scope after a filter narrows to it** -- The derivation returns an empty result set for that scope (zero across earned/outstanding/overdue in every currency); the consuming screen (FEAT-12.SPEC-002) renders this as its No-Results state, not as an error.
- **Two overlapping computations for the same scope run at effectively the same time** -- Each reads current source data independently; see FEAT-12.SPEC-004's Edge Cases for how the consuming automation resolves which result is kept.

## Acceptance Criteria

**FEAT-12.SPEC-003-AC-01:** Given Nadia has invoices in a single currency across three clients, when the account-wide computation runs, then earned_total, outstanding_total, and overdue_total are each returned as one figure for that currency, summed across all three clients.

**FEAT-12.SPEC-003-AC-02:** Given Nadia has invoices in two different currencies, when the account-wide computation runs, then two separate Financial Totals records are returned, one per currency, and neither is combined with the other.

**FEAT-12.SPEC-003-AC-03:** Given an invoice has status Sent with a due_date three days in the future, when classification runs, then it is classified "Due" and its amount contributes to outstanding_total.

**FEAT-12.SPEC-003-AC-04:** Given an invoice has status Sent with a due_date one day in the past, when classification runs, then it is classified "Overdue" and its amount contributes to overdue_total, regardless of whether FEAT-11 has yet set the Overdue status flag.

**FEAT-12.SPEC-003-AC-05:** Given an invoice's due_date is exactly today, when classification runs, then it is classified "Due," not "Overdue."

**FEAT-12.SPEC-003-AC-06:** Given an invoice has status Paid with a Payment of status Succeeded and no refund or reversal recorded, when the computation runs, then it is classified "Paid" and its full Payment.amount contributes to earned_total.

**FEAT-12.SPEC-003-AC-07:** Given an invoice has status Partially refunded, when the computation runs, then it remains classified "Paid" and earned_total includes only the net amount after the recorded partial refund.

**FEAT-12.SPEC-003-AC-08:** Given an invoice has status Refunded (in full), when the computation runs, then it remains classified "Paid" but contributes zero to earned_total.

**FEAT-12.SPEC-003-AC-09:** Given an invoice has status Disputed and its Payment has not been reported Reversed, when the computation runs, then it is classified "Paid" and its full net amount still contributes to earned_total.

**FEAT-12.SPEC-003-AC-10:** Given a previously Disputed invoice's Payment is subsequently reported Reversed, when the next computation runs, then earned_total for the affected scope decreases by that Payment's amount.

**FEAT-12.SPEC-003-AC-11:** Given a payment was recorded manually by Nadia (off-platform) with status Recorded manually, when the computation runs, then the invoice is classified "Paid" and the manually recorded amount contributes to earned_total identically to a processor-confirmed payment.

**FEAT-12.SPEC-003-AC-12:** Given an ad hoc invoice has status Generated (drafted, never sent), when the computation runs, then it is excluded from classification and from every total.

**FEAT-12.SPEC-003-AC-13:** Given an invoice has status Corrected and a correcting invoice exists for the same amount, when the computation runs, then only the correcting invoice contributes to totals and the Corrected invoice is excluded.

**FEAT-12.SPEC-003-AC-14:** Given a client's invoices span two currencies, when a per-client breakdown is computed, then two separate currency records are returned for that client and the account-wide total for each currency equals the sum of all clients' breakdowns in that currency.

**FEAT-12.SPEC-003-AC-15:** Given Nadia is the requesting user, when she requests account-wide or per-client/per-project totals, then the computation always proceeds for her own account.

**FEAT-12.SPEC-003-AC-16:** Given Owen (Client Primary Contact) has no route to this feature's screens, when any attempt to reach them occurs, then the totals are never computed for or shown to him.

**FEAT-12.SPEC-003-AC-17:** Given Dana (Support Operator) has an open support session on Nadia's account, when she requests account-wide, per-client, or per-project totals, then the computation runs identically to Nadia's own view, read-only.

**FEAT-12.SPEC-003-AC-18:** Given Dana (Support Operator) is in a support session, when she attempts to trigger an accounting export from these totals, then the action is denied with "Support sessions cannot generate exports."

**FEAT-12.SPEC-003-AC-19:** Given a project has no invoices in the requested scope, when the computation runs, then it returns an empty result (zero across earned/outstanding/overdue in every currency) rather than an error.

**FEAT-12.SPEC-003-AC-20:** Given the computation completes successfully, when computed_at is set, then it reflects the current time of that successful run, and remains unchanged by FEAT-12.SPEC-004 if a subsequent recompute attempt fails.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 14 | 14 |
| Defaults/Derivations | 7 | 7 |
| Business Rules | 7 | 7 |
| Edge Cases | 12 | 12 |



# Automation Spec: Dashboard Totals Refresh

## Overview

**Name:** Dashboard Totals Refresh
**ID:** FEAT-12.SPEC-004
**Type:** Automation
**Purpose:** Recomputes the affected Financial Totals whenever an underlying invoice or payment changes elsewhere in the product, and falls back to the last successfully computed totals with retry if a recomputation attempt fails.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard

## Scope and Non-Goals

**In Scope:**
- Deciding when to recompute Financial Totals: on an underlying invoice or payment change, and on a manual Retry from either dashboard screen
- Scoping each recompute to the affected client/project and the account-wide aggregate, using FEAT-12.SPEC-003's formula
- Retrying automatically on failure and preserving the last successfully computed totals, per currency, so a viewing screen never shows a blank or misleading number
- Silently refreshing FEAT-12.SPEC-001 and FEAT-12.SPEC-002 when they are open and viewing an affected scope

**Non-Goals:**
- The derivation formula itself (what counts as earned, outstanding, or overdue, and the paid/due/overdue classification) -- owned entirely by FEAT-12.SPEC-003; this automation only decides when to invoke it and what to do with the result
- Notifying Nadia by email or in-app when a recompute succeeds or fails -- excluded per the feature's Communications field: "this is a self-initiated view; it sends no notifications itself." A failure surfaces only as the on-screen Error state defined in FEAT-12.SPEC-001 and FEAT-12.SPEC-002, not as a separate notification
- Generating the accounting export file -- excluded per scope-boundaries.md (SC-07); that is a distinct, on-demand capability owned by FEAT-22, unrelated to this automation's background recompute
- Correcting or reversing the underlying Invoice or Payment records themselves -- excluded per the dependency map: those records are read-only Connected Entities for this feature; corrections belong to FEAT-09, FEAT-10, and FEAT-25

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An invoice is generated, sent, has its due date changed, or is flagged Overdue, Refunded, Partially refunded, Disputed, or Corrected | FEAT-09 (Invoice Generation & Sending) / FEAT-11 (Automated Payment Reminders, for the Overdue flag) | Fires on any Invoice status or due_date change reported by these features | The changed Invoice's id, status, amount, currency, due_date, and project reference |
| A payment succeeds, fails, is recorded manually, or is reversed | FEAT-10 (Invoice Payment Processing) / FEAT-25 (Refund & Cancelled Project Handling, for reversals and refund amounts) | Fires on any Payment status change or a recorded refund/reversal amount | The changed Payment's id, status, amount, paid_at, and the Invoice it belongs to |
| Nadia taps Retry on the Error state | FEAT-12.SPEC-001 (Dashboard Overview) / FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Fires when Nadia manually requests an immediate recomputation attempt after a failure | The scope (account / client / project) and currency the failed computation was for |

## Processing Logic

1. Identify the scope affected by the triggering change: the specific client and/or project the changed Invoice or Payment belongs to, plus the account-wide aggregate (which every change affects). A client/project roster or currency change (FEAT-01, FEAT-15) is not itself a trigger for this automation -- it is a Connected Entity read that FEAT-12.SPEC-003 incorporates automatically the next time this automation recomputes an affected scope (see Edge Cases).
2. Invoke FEAT-12.SPEC-003 (Financial Totals Aggregation) to recompute the Financial Totals for each affected scope and currency, reading current Invoice and Payment records at the moment of the run.
3. If the recomputation completes without error, compare the new result to the currently held Financial Totals for that scope and currency (the totals this automation last successfully computed and is holding). Replace the held totals with the new result and set its computed_at to the current time, whether or not the figures actually changed.
4. If FEAT-12.SPEC-001 or FEAT-12.SPEC-002 is currently open and displaying an affected scope, refresh its displayed totals with the newly held result, silently, without disrupting the screen's active filter selection or scroll position.
5. If the recomputation fails, leave the currently held Financial Totals for that scope and currency unchanged, and schedule an automatic retry.
6. Retry automatically up to platform parameter: `dashboard-aggregation-retry-count` times, spaced platform parameter: `dashboard-aggregation-retry-interval` apart. If any retry succeeds, proceed as in Step 3-4.
7. If all automatic retries are exhausted without success, leave the currently held Financial Totals unchanged and, if a screen is open on the affected scope, surface the Error state (last successfully computed totals plus a manual Retry control, per FEAT-12.SPEC-001 and FEAT-12.SPEC-002's States sections). If no screen is open, the failure is recorded silently and the Error state appears the next time a screen opens on that scope.
8. When Nadia manually taps Retry, attempt one immediate recomputation for that scope and currency regardless of where the automatic retry schedule currently stands; on success, proceed as in Step 3-4; on failure, the Error state remains and the automatic schedule (if still running) continues unaffected.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recompute succeeded, totals changed | Recomputation completes and the new result differs from the currently held one | Held totals replaced with the new result; computed_at updated | Any open, affected screen refreshes silently to the new figures | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Recompute succeeded, no change | Recomputation completes and the new result equals the currently held one | Held totals' computed_at updated; figures unchanged | No visible change on any open screen | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Automatic retry in progress | Recomputation failed and an automatic retry is scheduled within the retry count/interval | Held totals unchanged | Any open, affected screen continues showing its last known totals with no interruption; no retry indicator is shown for an automatic (non-manual) retry, since it is invisible by design | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Automatic retries exhausted | All automatic retries fail | Held totals unchanged | Any open, affected screen shows the Error state: last successfully computed totals plus a manual Retry control | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Manual retry succeeded | Nadia taps Retry and the immediate recomputation succeeds | Held totals replaced with the new result; computed_at updated | The Error state clears and the refreshed totals render | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Manual retry failed | Nadia taps Retry and the immediate recomputation fails | Held totals unchanged | The Error state banner updates to "Still unable to refresh. Showing your last known totals." and the Retry control remains available | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| No screen open when failure occurs | All automatic retries fail while neither FEAT-12.SPEC-001 nor FEAT-12.SPEC-002 is open | Held totals unchanged | Nothing shown at the time of failure; the Error state appears the next time Nadia opens either screen on that scope | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |

## Data Model

**Reads:** Invoice (status, amount, currency, due_date, project) and Payment (status, amount, paid_at, invoice) for the affected scope, per the Feature Dependency Map. Client and Project (currency, archive status) as part of FEAT-12.SPEC-003's aggregation, so a roster or currency change already on record is reflected in whatever scope this automation next recomputes. Consumes FEAT-12.SPEC-003's derivation for the actual computation.
**Creates:** None -- this automation persists no new business record; it only maintains the currently held Financial Totals result FEAT-12.SPEC-003 defines.
**Updates:** The currently held Financial Totals (earned_total, outstanding_total, overdue_total, per-currency, per scope) and its computed_at timestamp.
**Deletes:** None.

## Business Rules

- XBR-22: this automation's triggers must cover every event type Financial Dashboard totals are derived from -- invoice generation/sending/due-date changes, payment success/failure/manual recording, and refunds/reversals/cancellations -- so no underlying change is ever missed.
- The last successfully computed totals are never discarded on failure; a failed recompute leaves the prior held result and its computed_at in place, so a viewing screen never shows a blank or misleading number (feature's Error state).
- This automation never re-derives the earned/outstanding/overdue formula itself -- every computation is delegated to FEAT-12.SPEC-003, so the two specs cannot drift out of sync.
- XBR-18: a recompute for one currency never merges with another currency's held result.
- A recompute triggered by a change to one client's data only recomputes that client's scoped totals and the account-wide aggregate; other clients' unaffected held totals are left untouched, since they did not change.
- A manual Retry always attempts immediately, independent of where the automatic retry schedule stands, so Nadia is never forced to wait out the automatic interval.

## Edge Cases

- **Concurrent trigger firing (invoices for two different clients change at effectively the same time)** -- Each client's recompute runs independently and scoped to its own client; the account-wide aggregate recomputes incorporating whichever changes have landed by the time it runs. Since each run is a full recompute from current source records rather than an incremental delta, no change is lost even if the two client-scoped runs and the account-wide run complete in a different order than they were triggered.
- **A trigger fires while a previous run for the same scope is still in flight** -- Each run reads current source data at the moment it executes and tags its result with that read time. Only a result whose read time is later than the currently held result's computed_at replaces it; an in-flight run that finishes after a newer trigger's run has already produced a fresher result is discarded on arrival rather than clobbering the more current data.
- **Automatic retries are exhausted while neither dashboard screen is open** -- The failure is recorded silently; since this feature sends no notifications of its own (Communications: N/A), Nadia only learns of it if she opens a dashboard screen on the affected scope, at which point the Error state and Retry control appear.
- **The client underlying a triggering change is archived (FEAT-01) mid-recompute** -- The recompute proceeds normally; archiving never erases records, so the archived client's totals remain computable and are still included in the account-wide aggregate and its own historical detail.
- **Nadia taps manual Retry while an automatic retry is already scheduled** -- The manual attempt runs immediately; it is not queued behind the automatic schedule. If the manual attempt itself fails, the automatic schedule continues unaffected from where it left off.
- **A client, project, or currency change (FEAT-01, FEAT-15) introduces a scope that has never been computed before** -- This automation is never triggered directly by the roster or currency change itself (that inbound relationship belongs to FEAT-12.SPEC-003, which reads Client and Project as part of every aggregation); the new scope gets its first held result the first time an invoice or payment event for it fires this automation, or the first time a screen loads it directly. That first result is a first-time computation rather than a recompute of an existing one; there is no prior "last known" total to fall back on, so a failure on this first attempt is treated identically to any other failure (automatic retry, then the Error state if exhausted) rather than as a special case.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Triggered by (inbound) | The manual Retry action fires this automation |
| FEAT-12.SPEC-001 (Dashboard Overview) | Affects (outbound) | Silently refreshes displayed totals on success, or surfaces the Error state on exhausted failure |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Triggered by (inbound) | Same manual Retry relationship |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Affects (outbound) | Same refresh/Error-state relationship |
| FEAT-12.SPEC-003 (Financial Totals Aggregation) | References (outbound) | Invokes this rule for every recomputation; never re-implements the formula |
| FEAT-09 (Invoice Generation & Sending) | Triggered by (inbound) | Invoice lifecycle events (generated, sent, due-date change) |
| FEAT-11 (Automated Payment Reminders) | Triggered by (inbound) | The Overdue status flag being set |
| FEAT-10 (Invoice Payment Processing) | Triggered by (inbound) | Payment success, failure, and manual recording |
| FEAT-25 (Refund & Cancelled Project Handling) | Triggered by (inbound) | Refunds, reversals, and cancellations |

## Analytics and Success Signals

- **dashboard_totals_recomputed** (trigger_type: invoice_change / payment_change / manual_retry; outcome: changed / no_change / failure; affected_scope: account / client / project) -- supports success-metrics.md: "Dashboard Comprehension" (recompute reliability and freshness are what let Nadia state accurate totals whenever she opens the dashboard)
- **dashboard_aggregation_retry_exhausted** (affected_scope: account / client / project, consecutive_failure_count) -- N/A -- no metric in success-metrics.md specifically targets aggregation-failure recovery; this event supports operational reliability monitoring rather than a Stage 2 product metric
- **dashboard_manual_retry_used** (outcome: success / failure) -- N/A -- no metric in success-metrics.md targets manual-retry usage specifically; this event supports monitoring how often the automatic path alone was insufficient

## Acceptance Criteria

**FEAT-12.SPEC-004-AC-01:** Given an invoice Nadia sent moves to status Paid (FEAT-10), when this automation fires, then the affected client's and the account-wide totals are recomputed via FEAT-12.SPEC-003 and the held totals are updated.

**FEAT-12.SPEC-004-AC-02:** Given Nadia is viewing FEAT-12.SPEC-001 with a client's invoice marked Paid elsewhere while she watches, when the recompute completes, then the dashboard's displayed totals update silently without disrupting her active filter.

**FEAT-12.SPEC-004-AC-03:** Given a payment is reported Reversed by FEAT-25 against a previously Disputed invoice, when this automation fires, then the affected scope's earned_total is recomputed downward to reflect the reversal.

**FEAT-12.SPEC-004-AC-04:** Given Nadia adds a new client (FEAT-01) and its first invoice is later generated, when this automation fires on that invoice event, then a first-time computed result is held for that client -- this automation is not fired by the client's addition itself, only by the invoice event that follows it.

**FEAT-12.SPEC-004-AC-05:** Given a recomputation attempt fails, when the automation retries automatically, then it retries up to platform parameter: `dashboard-aggregation-retry-count` times spaced platform parameter: `dashboard-aggregation-retry-interval` apart before surfacing any failure to the user.

**FEAT-12.SPEC-004-AC-06:** Given all automatic retries for a scope are exhausted while FEAT-12.SPEC-001 is open on that scope, when the final retry fails, then the last successfully computed totals remain visible with the Error state's Retry control -- never a blank or misleading number.

**FEAT-12.SPEC-004-AC-07:** Given all automatic retries are exhausted while no dashboard screen is open, when Nadia next opens FEAT-12.SPEC-001 on the affected scope, then she sees the Error state immediately, with the last successfully computed totals and a Retry control.

**FEAT-12.SPEC-004-AC-08:** Given Nadia taps Retry on FEAT-12.SPEC-002's Error state, when the immediate recomputation succeeds, then the held totals update, computed_at advances, and the Error state clears.

**FEAT-12.SPEC-004-AC-09:** Given Nadia taps Retry and the immediate recomputation also fails, when the failure occurs, then the banner updates to "Still unable to refresh. Showing your last known totals." and the Retry control remains.

**FEAT-12.SPEC-004-AC-10:** Given a manual Retry is tapped while an automatic retry is already scheduled for the same scope, when the manual attempt runs, then it executes immediately rather than waiting for the scheduled automatic attempt, and the automatic schedule continues unaffected if the manual attempt fails.

**FEAT-12.SPEC-004-AC-11:** Given invoices for two different clients change at effectively the same time, when both trigger this automation, then each client's recompute runs independently and the account-wide aggregate reflects both changes once each has completed, regardless of completion order.

**FEAT-12.SPEC-004-AC-12:** Given a recompute for one scope is still in flight when a newer trigger for the same scope fires, when both runs eventually complete, then only the result with the later read timestamp replaces the held totals, even if the older run happens to finish last.

**FEAT-12.SPEC-004-AC-13:** Given a client underlying a triggering change is archived mid-recompute (FEAT-01), when the recompute completes, then it succeeds normally and the archived client's totals remain included.

**FEAT-12.SPEC-004-AC-14:** Given a project's currency is set for the first time (FEAT-15) and its first invoice is subsequently generated, when this automation fires on that invoice event, then the new currency's scope is computed and its result is held for that project going forward -- the currency assignment itself does not fire this automation.

**FEAT-12.SPEC-004-AC-15:** Given an invoice's due date is changed before sending (FEAT-09), when this automation fires, then the affected totals are recomputed to reflect the new due date's Due/Overdue classification.

**FEAT-12.SPEC-004-AC-16:** Given a recomputation succeeds but produces figures identical to the currently held result, when the run completes, then computed_at still advances and no visible change appears on any open screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (invoice change, payment change, manual retry) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
