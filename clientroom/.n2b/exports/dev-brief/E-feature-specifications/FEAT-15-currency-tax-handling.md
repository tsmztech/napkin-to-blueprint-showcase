# FEAT-15 — Currency & Tax Handling

This chapter covers Currency & Tax Handling, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 8 specifications carrying 90 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | screen | 17 |
| FEAT-15.SPEC-002 | Freelancer Time Zone Setting | screen | 11 |
| FEAT-15.SPEC-003 | Currency & Tax Validation Rules | logic-rule | 12 |
| FEAT-15.SPEC-004 | Currency & Tax Lock After First Invoice | logic-rule | 9 |
| FEAT-15.SPEC-005 | Currency & Tax Configuration Access Rules | logic-rule | 10 |
| FEAT-15.SPEC-006 | Time Zone & Local Date/Time Display Rule | logic-rule | 11 |
| FEAT-15.SPEC-007 | Multi-Currency Dashboard Non-Aggregation Rule | logic-rule | 8 |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | automation | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Currency & Tax Handling

## Summary

**Feature:** Currency & Tax Handling
**ID:** FEAT-15
**Description:** The freelancer sets the currency for each client/project and shows an appropriate tax line on invoices, without any single country's currency or tax rules hard-coded into the product.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Scale & Non-Functional Expectations: "Currencies, tax on invoices (VAT, GST, US sales tax) and time zones must not be hard-coded" for a worldwide-from-day-one product. MVP phase: every invoice needs a correct currency and tax line from the first one issued. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Set project currency — choose the billing currency per client/project
- Configure a tax line — set a tax label or rate (VAT, GST, US sales tax, or none) per client/region
- Time zones and local formats — dates, due dates, and times show in each viewer's own time zone and familiar date format, while reminder day counts follow the freelancer's time zone [AUDIT-ADDED: 4 -- internationalization: BRIEF.md says time zones must not be hard-coded, but no feature owned them]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia sets a project's billing currency and tax label/rate during billing setup; Dana views it read-only inside a logged support session |
| FEAT-15.SPEC-002 | Freelancer Time Zone Setting | Screen | Nadia (Freelancer) | Nadia sets her own time zone, which grounds every reminder count and local-time display across the product |
| FEAT-15.SPEC-003 | Currency & Tax Validation Rules | Logic/Rule | Nadia (Freelancer) | Governs which currencies are acceptable and what shape a tax line may take, and blocks invoice generation with a specific, correctable message on an invalid configuration |
| FEAT-15.SPEC-004 | Currency & Tax Lock After First Invoice | Logic/Rule | Nadia (Freelancer) | Locks a project's currency (and its paired tax line) the moment its first invoice is sent, and blocks any later change attempt |
| FEAT-15.SPEC-005 | Currency & Tax Configuration Access Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Governs who may view or change project currency/tax configuration: Nadia Full, Dana read-only in a logged session, Owen and Priya no direct access to the configuration surface at all |
| FEAT-15.SPEC-006 | Time Zone & Local Date/Time Display Rule | Logic/Rule | All | Governs how dates, due dates, and times are rendered in each viewer's own time zone and familiar format, and establishes the freelancer's time zone as the basis for reminder day counts |
| FEAT-15.SPEC-007 | Multi-Currency Dashboard Non-Aggregation Rule | Logic/Rule | Nadia (Freelancer) | Governs that financial totals are always shown grouped per currency and are never converted or summed across currencies |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | Automation | Nadia (Freelancer), Owen (Client Primary Contact) | At invoice generation, stamps the project's configured currency and tax label/rate onto the new invoice, computes the tax amount and total, and flags a mismatch if the invoice's currency would not match the project's current configuration |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Set project currency — choose the billing currency per client/project | FEAT-15.SPEC-001, FEAT-15.SPEC-003, FEAT-15.SPEC-004, FEAT-15.SPEC-008 | The configuration screen captures the choice; the validation rule enforces a recognized world currency; the lock rule fixes it after the first invoice; the application automation stamps it onto every generated invoice | Phase 2 (Explicit) |
| Configure a tax line — set a tax label or rate (VAT, GST, US sales tax, or none) per client/region | FEAT-15.SPEC-001, FEAT-15.SPEC-003, FEAT-15.SPEC-008 | The same configuration screen captures the tax label/rate or none; the validation rule enforces the label-and-rate shape (no automatic per-jurisdiction calculation); the application automation applies it to the invoice subtotal | Phase 2 (Explicit) |
| Time zones and local formats — dates, due dates, and times show in each viewer's own time zone and familiar date format, while reminder day counts follow the freelancer's time zone | FEAT-15.SPEC-002, FEAT-15.SPEC-006 | The time zone setting screen captures Nadia's own time zone; the display rule renders every date/time in each viewer's own time zone and format and grounds reminder day counts in the freelancer's time zone | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-15.SPEC-004 | Currency & Tax Lock After First Invoice | Phase 3 (Entity-Lifecycle Analysis) | The Validation & Limits field states "a project's currency cannot change once its first invoice is sent" but names no mechanism; the CRUD matrix's State Transition cell for Project was otherwise empty |
| FEAT-15.SPEC-005 | Currency & Tax Configuration Access Rules | Phase 5 (Rule-Constraint Discovery) | The Access field's four fully differentiated role behaviors (Nadia Full, Owen view-only-on-invoices, Priya none, Dana view-in-session) exceed the inline-authorization threshold and govern a screen two other roles must never reach directly |
| FEAT-15.SPEC-007 | Multi-Currency Dashboard Non-Aggregation Rule | Phase 5 (Rule-Constraint Discovery) | XBR-17/XBR-18 name FEAT-15 as the authority owning currency and its non-aggregation rule; the dependency map's cross-feature business rule needed a standalone home rather than being left implicit on FEAT-12's dashboard |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | Phase 4 (Trigger-Response Analysis) | The Invoice entity's lifecycle line records "Updated by ... FEAT-15 (currency and tax fields at generation)" — a system side-effect of invoice generation that the feature description never states explicitly but the dependency map requires |

## Entity-Lifecycle Coverage Matrix

**Entity: Project** *(this feature owns the currency and tax fields; all other Project fields and the record's own creation/archival belong to FEAT-01)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | The Project record itself is created by Client & Project Management (FEAT-01); this feature never creates a Project, only sets two of its fields once it exists | — |
| Read (single) | FEAT-15.SPEC-001 | The configuration screen loads a project's current currency, tax label, and tax rate when Nadia opens billing setup | — |
| Read (list) | N/A | This feature has no list view of its own; per-project currency/tax values are read in list context only by other features (e.g., the Financial Dashboard, FEAT-12) | — |
| Update | FEAT-15.SPEC-001 | Nadia sets or changes the project's currency and tax label/rate on the configuration screen, subject to the lock in SPEC-004 | — |
| Delete/Archive | N/A | This feature never deletes or archives a Project. Project archival and deletion are owned exclusively by FEAT-01 (archive) and FEAT-24 (account deletion) — an explicit non-goal, not a silent omission | — |
| State Transition | FEAT-15.SPEC-004 | Once the project's first invoice is sent, its currency and tax fields transition from editable to locked; the lock is permanent for that project (no unlock path) | — |

**Entity: Freelancer Account** *(this feature owns only the `time_zone` field; every other account field belongs to Settings & Account Management, FEAT-21)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | The account record is created at sign-up (FEAT-20), which sets an initial time zone default; this feature only lets Nadia change it afterward | — |
| Read (single) | FEAT-15.SPEC-002 | The time zone setting screen loads Nadia's current time zone value | — |
| Read (list) | N/A | There is only ever one account per freelancer session; no list applies | — |
| Update | FEAT-15.SPEC-002 | Nadia changes her time zone; the new value takes effect for all future reminder-day-count and local-time-display computation (FEAT-15.SPEC-006) | — |
| Delete/Archive | N/A | This feature never deletes or archives the account or the field; the whole account is deleted only through FEAT-24 | — |
| State Transition | N/A | `time_zone` is a plain value with no lifecycle states of its own | — |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invoice | FEAT-15.SPEC-004 | Checked to determine whether a project already has a sent invoice, which is what triggers the currency/tax lock |
| Invoice | FEAT-15.SPEC-008 | Read at generation time as the record onto which the project's currency and tax fields are stamped (this is also the one Update this feature makes to Invoice, per the dependency map's Invoice lifecycle line) |
| Client | FEAT-15.SPEC-001 | The configuration screen shows the owning client's name for context while Nadia sets a project's currency/tax |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia saves a project's currency and tax configuration | Validate the currency against the recognized world-currency list and the tax line against the label-and-rate-or-none shape | Standalone Logic/Rule | FEAT-15.SPEC-003 |
| Nadia attempts to save an invalid currency or tax configuration | Block the save with a specific, correctable error message; no invoice can be generated from an invalid configuration | Standalone Logic/Rule, surfaced inline in the triggering screen (Error state) | FEAT-15.SPEC-003 / FEAT-15.SPEC-001 |
| A project's first invoice is sent (event from Invoice Generation & Sending, FEAT-09) | Lock the project's currency and tax fields; any later attempt to change them is refused with an explanation | Standalone Logic/Rule | FEAT-15.SPEC-004 |
| Nadia attempts to change currency/tax on a project whose first invoice has already been sent | Refuse the change and explain that the currency is fixed once billing has begun | Standalone Logic/Rule, surfaced inline in the configuration screen | FEAT-15.SPEC-004 / FEAT-15.SPEC-001 |
| An invoice is generated (automatic or ad hoc, FEAT-09) for a project with a valid currency/tax configuration | Stamp the invoice with the project's currency, tax label, and tax rate; compute the tax amount and total from the subtotal | Standalone Automation | FEAT-15.SPEC-008 |
| An invoice generation attempt reaches a project with no currency/tax configured yet | Block generation and prompt Nadia to complete configuration first (Empty state) | Inline in FEAT-15.SPEC-001 (Empty state), enforced by FEAT-15.SPEC-003 | FEAT-15.SPEC-001 / FEAT-15.SPEC-003 |
| An invoice is generated whose stamped currency would not match the project's currently configured currency (e.g., a race between a configuration change and a concurrent generation) | Flag the discrepancy (`invoice_currency_mismatch_flagged`) rather than silently issuing a mismatched invoice | Standalone Automation (ambiguous-outcome handling) | FEAT-15.SPEC-008 |
| Nadia sets or changes her time zone | Recompute the basis used for every future reminder day-count and local-time display for her account | Standalone Logic/Rule | FEAT-15.SPEC-006 |
| Any viewer opens a screen that shows a date, due date, or time (across this and other features) | Render it in that viewer's own time zone and familiar date format | Standalone Logic/Rule, applied inline wherever a date/time appears | FEAT-15.SPEC-006 |
| The automated reminder schedule (FEAT-11) evaluates whether a day-3 or day-10 reminder is due | Count elapsed days using the freelancer's time zone, not the viewer's | Standalone Logic/Rule, cross-feature | FEAT-15.SPEC-006 |
| The Financial Dashboard (FEAT-12) aggregates earned/outstanding/overdue totals across clients in different currencies | Show totals grouped per currency; never convert or add amounts across currencies | Standalone Logic/Rule, cross-feature | FEAT-15.SPEC-007 |
| Owen or Priya attempts to reach the currency/tax configuration screen | Show nothing — the capability is not shown at all to either role | Inline in the triggering screen (Permission Denied state), governed by | FEAT-15.SPEC-005 / FEAT-15.SPEC-001 |
| Dana opens a project inside a logged support session (FEAT-31) | Show the currency/tax configuration read-only, with no save control | Inline in the triggering screen, governed by, cross-feature | FEAT-15.SPEC-005 / FEAT-15.SPEC-001 |
| Nadia saves a valid configuration or a valid time zone | Emit the corresponding signal (`currency_set`, `tax_treatment_set`, `timezone_set`) | Inline in triggering spec | FEAT-15.SPEC-001 / FEAT-15.SPEC-002 |

## Shared Context

**Shared Entities:**
- Project — currency and tax_label/tax_rate fields are read and updated by FEAT-15.SPEC-001, validated by FEAT-15.SPEC-003, locked by FEAT-15.SPEC-004, and read at invoice-generation time by FEAT-15.SPEC-008. No other spec in this feature writes to Project.
- Freelancer Account — the `time_zone` field is read and updated by FEAT-15.SPEC-002 and consumed by FEAT-15.SPEC-006 for every reminder-count and local-time computation.
- Invoice — read by FEAT-15.SPEC-004 (lock check) and updated once, at generation, by FEAT-15.SPEC-008 (currency and tax fields stamped on).

**Shared UI Patterns:**
- Locked-configuration banner — once a project's first invoice is sent, FEAT-15.SPEC-001 replaces the editable currency/tax fields with a plain, non-editable display and a short explanation ("currency and tax are fixed after your first invoice"); Spec Writers should describe this as the same visual treatment FEAT-09.SPEC-002 uses for "not edited after sending" so the product speaks with one voice about immutability.
- Role-gated screen — FEAT-15.SPEC-001 is entirely absent from Owen's and Priya's navigation (Access: None), and rendered strictly read-only for Dana inside a support session; FEAT-15.SPEC-005 is the single source of truth both other specs reference rather than restating the gate.

**Shared Validation:**
- FEAT-15.SPEC-003 defines the single source of truth for what counts as a valid currency and a valid tax line. FEAT-15.SPEC-001 (screen-level save) and FEAT-15.SPEC-008 (generation-time application) both apply it rather than re-deriving the rule.

**Flagged Stage 2 inconsistency (not resolved by the Analyst, per decision authority):** the Client entity's Fields list in the dependency map slice also names "currency and tax treatment ... set through FEAT-15," while this feature's own Connected Entities line names only Invoice and Project, and the Project entity's lifecycle line (not the Client entity's) is the one that credits FEAT-15 as an updater. This Brief elaborates the configuration as living on Project (matching "per client/project" billing and the Project lifecycle line), consistent with the Connected Entities field; the Client entity's parallel mention is a Stage 2 documentation inconsistency, flagged here for the Requirements Architect rather than resolved by re-deriving Stage 2 scope. Separately, the Freelancer Account entity's own lifecycle line in the dependency map credits only FEAT-21 as its updater, while this feature's Data Notes, Signals, and Key Capabilities assign the `time_zone` field and its `timezone_set` signal to FEAT-15; this Brief follows the Stage 2 Functional Depth fields (which are authoritative input per this feature's contract) and gives FEAT-15.SPEC-002 ownership of the time zone value, flagging rather than silently overriding the dependency map's lifecycle line.

## Internal Dependency Map

```
FEAT-01 (project billing setup) -> [Nadia navigates to configure currency/tax] -> FEAT-15.SPEC-001 (Project Currency & Tax Configuration) [cross-feature entry point]
FEAT-15.SPEC-001 (Project Currency & Tax Configuration) -> [Nadia saves] -> FEAT-15.SPEC-003 (Currency & Tax Validation Rules)
FEAT-15.SPEC-001 (Project Currency & Tax Configuration) -> [validates access using] -> FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules)
FEAT-15.SPEC-001 (Project Currency & Tax Configuration) -> [checks lock state using] -> FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice)
FEAT-09 (Automatic Invoice Generation) -> [first invoice sent] -> FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice) [cross-feature trigger]
FEAT-09 (Automatic or Manual Invoice Generation) -> [invoice generated] -> FEAT-15.SPEC-008 (Invoice Currency & Tax Line Application) [cross-feature trigger]
FEAT-15.SPEC-008 (Invoice Currency & Tax Line Application) -> [enforces] -> FEAT-15.SPEC-003 (Currency & Tax Validation Rules)
FEAT-21 (Settings & Account Management) -> [Nadia navigates to time zone preference] -> FEAT-15.SPEC-002 (Freelancer Time Zone Setting) [cross-feature entry point]
FEAT-15.SPEC-002 (Freelancer Time Zone Setting) -> [Nadia saves] -> FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule)
FEAT-11 (Automated Payment Reminders) -> [computes day-3/day-10 counts using] -> FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) [cross-feature reference]
FEAT-12 (Freelancer Financial Dashboard) -> [aggregates totals using] -> FEAT-15.SPEC-007 (Multi-Currency Dashboard Non-Aggregation Rule) [cross-feature reference]
FEAT-31 (Operator Support Access) -> [Dana opens a read-only session] -> FEAT-15.SPEC-001 (Project Currency & Tax Configuration) [governed by FEAT-15.SPEC-005]
```

**Default Entry:** FEAT-15.SPEC-001 (Project Currency & Tax Configuration) is reached from FEAT-01's project billing setup and has no independent entry point of its own; FEAT-15.SPEC-002 (Freelancer Time Zone Setting) is reached separately, from the freelancer's account settings area (FEAT-21).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-15.SPEC-001 | Inbound | FEAT-01 (Client & Project Management) | Project billing setup is the sole entry point into this feature's configuration screen | Nadia opens a project's billing setup for the first time or to review it |
| FEAT-15.SPEC-001, FEAT-15.SPEC-008 | Outbound | FEAT-02 (Proposal Creation & Sending) | Proposal prices must be denominated in the project's already-configured currency (XBR-17) | Nadia drafts a proposal for a project |
| FEAT-15.SPEC-004 | Inbound | FEAT-09 (Invoice Generation & Sending) | Sending a project's first invoice fires the currency/tax lock | Invoice status set to Sent |
| FEAT-15.SPEC-008 | Inbound | FEAT-09 (Invoice Generation & Sending) | Every invoice-generation trigger (deposit, milestone approval, completion, ad hoc) invokes this automation to stamp currency and tax onto the new invoice | Invoice generated |
| FEAT-15.SPEC-002 | Inbound | FEAT-21 (Settings & Account Management) | The freelancer's account settings area is the entry point into the time zone setting screen | Nadia opens her time zone preference |
| FEAT-15.SPEC-006 | Outbound | FEAT-11 (Automated Payment Reminders) | Day-3 and day-10 reminder counts are computed against the freelancer's time zone, not the viewer's (XBR-15) | Reminder schedule evaluates whether a reminder is due |
| FEAT-15.SPEC-006 | Outbound | FEAT-09 (Invoice Generation & Sending) | Issue dates, due dates, and reminder history on invoice screens render in each viewer's own time zone and date format | Invoice detail viewed by Nadia or Owen |
| FEAT-15.SPEC-007 | Outbound | FEAT-12 (Freelancer Financial Dashboard) | Dashboard earned/outstanding/overdue totals are grouped and shown per currency, never converted or summed (XBR-18) | Dashboard totals computed |
| FEAT-15.SPEC-005 | Outbound | FEAT-31 (Operator Support Access) | Dana views project currency/tax configuration read-only, inside a logged, time-limited support session | Support session opened on a freelancer's account |

## Non-Functional Notes

**Data volumes / growth:** Currency and tax configuration is a small, fixed set of fields per project (one currency, one tax label, one tax rate) and one time zone value per freelancer account; volume tracks the freelancer's 3–15 active clients (assumptions-constraints.md, ASMP-22-scale context implied by feature-dependency-map.md, Client entity) and carries no growth concern of its own.

**Responsiveness:** Configuration is entirely local with no real-time dependency (feature's own States field: "Loading: N/A — configuration is local"); saving a project's currency/tax or the freelancer's time zone completes without a perceptible wait, consistent with the product-wide loading and progress-feedback baseline (assumptions-constraints.md, ASMP-27).

**Data sensitivity / privacy:** The currency, tax label/rate, and time zone values are business-configuration data, not personal data in themselves (feature-dependency-map.md, Entity: Project, Data Sensitivity: "no special-category data"); they become part of the Invoice record once applied, and the Invoice is GDPR-class financial data carrying personal data about the freelancer and the client contact (assumptions-constraints.md, ASMP-24).

**Compliance flags:** Every invoice must carry the content commonly required of a valid invoice worldwide, including a tax line (assumptions-constraints.md, ASMP-24); this feature supplies that tax line as a freelancer-configured label and rate only — it performs no automatic jurisdictional tax calculation and makes no compliance claim about correctness under any specific country's tax law (scope-boundaries.md, SC-16), so the freelancer remains responsible for setting a tax line that is correct for her own situation.

## Non-Goals

- **Automatic tax calculation per country or region** — Excluded per scope-boundaries.md (SC-16): the tax line is a freelancer-configured label and rate (or none); the product computes no jurisdiction-specific tax rules of its own. BRIEF.md's Open Questions left tax depth undecided, and tax tooling appears in only one of five profiled competitors, US-specific there, which a solo founder could not extend worldwide (Validation & Limits field, product-features.md).
- **Interfaces or tax/currency labels in languages other than English** — Excluded per scope-boundaries.md (SC-20): "English only at launch." This feature adapts currency symbols, tax-line values, time zones, and date *formats* to each user, but no product text is translated.
- **Automatic currency conversion or live exchange-rate lookups** — Adjacency exclusion: the assumptions-constraints.md Dependencies slice names no external capability this feature relies on, and XBR-18 (owned by this feature) requires that financial totals across currencies are shown grouped and never converted or summed; building a conversion capability would directly contradict that rule.
- **A single invoice spanning multiple currencies** — Adjacency exclusion: XBR-17 (owned by this feature) ties every invoice to its project's one configured currency; splitting a single invoice's line items across currencies is not a capability the product definition describes anywhere and would break the "carries its own currency consistently" guarantee invoices give clients.
- **Versioned history of past currency/tax configurations** — Intentional lifecycle decision confirmed by the Entity-Lifecycle Coverage Matrix: because a project's currency and tax line become permanently locked after its first invoice (Validation & Limits field), there is no scenario in which a project accumulates multiple historical configurations to track, so no versioning or audit-of-changes capability is built beyond the lock itself.



# Screen Spec: Project Currency & Tax Configuration

## Overview

**Name:** Project Currency & Tax Configuration
**ID:** FEAT-15.SPEC-001
**Type:** Screen
**Purpose:** Nadia sets a project's billing currency and tax label/rate during billing setup, and views the locked, read-only version of that configuration once the project's first invoice has been sent; Dana views the same configuration read-only inside a logged support session.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling

## Scope and Non-Goals

**In Scope:**
- Displaying and capturing a project's currency, tax label, and tax rate (or no tax) during billing setup
- Showing the owning client's name for context while Nadia sets the values
- Replacing the editable fields with a locked, read-only display once the project's first invoice has been sent
- Surfacing the empty-configuration prompt that blocks invoice generation until currency/tax is set
- Surfacing validation and lock errors inline, sourced from FEAT-15.SPEC-003 and FEAT-15.SPEC-004
- Dana's read-only view of this same configuration inside a support session

**Non-Goals:**
- Deciding which currencies and tax shapes are valid -- owned entirely by FEAT-15.SPEC-003 (Currency & Tax Validation Rules); this screen only surfaces that spec's outcome
- Deciding when the configuration becomes locked -- owned entirely by FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice); this screen only renders the locked state
- Automatic, jurisdiction-specific tax calculation -- excluded per scope-boundaries.md (SC-16): the tax line is a freelancer-configured label and rate only, never a computed jurisdictional rule
- Creating, renaming, or archiving the Project record itself -- owned entirely by Client & Project Management (FEAT-01); this screen only sets two of that record's fields
- Currency conversion of any kind -- excluded per XBR-18 and this feature's own Non-Goals: the product never converts between currencies, so no conversion control ever appears here

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Client & Project Management, project billing setup) | Nadia opens a project's billing setup for the first time or to review it | Project reference, owning client reference |
| FEAT-31 (Operator Support Access) | Dana opens a project's billing setup inside a logged, read-only support session | Project reference, read-only session context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen for any of her own projects | Set or change currency and tax label/rate while unlocked (FEAT-15.SPEC-003 validates every save); view-only once locked (FEAT-15.SPEC-004) | -- |
| Owen (Client Primary Contact) | None -- this screen is entirely absent from his navigation | None | A direct link to this screen redirects to his portal home with no partial content ever rendered, per FEAT-15.SPEC-005 |
| Priya (Client Reviewer Contact) | None -- this screen is entirely absent from her navigation | None | A direct link to this screen redirects to her portal home with no partial content ever rendered, per FEAT-15.SPEC-005 |
| Dana (Support Operator) | Full content, read-only, inside a logged support session (FEAT-31) | View only -- no save control is ever rendered | The Save control and every editable input are not rendered; a direct attempt to submit a change (e.g., a replayed request) is refused and the screen re-renders in its read-only form, per FEAT-15.SPEC-005 |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Redirected to sign-in; any unsaved currency/tax entry is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Currency & Tax" with the owning client's name shown directly beneath it for context, and a back control returning to the project's billing setup area (FEAT-01).

**Body, unlocked state (before the first invoice is sent):**
- **Currency selector** (selection input, required): a searchable list of recognized world currencies; no value is preselected.
- **Tax treatment group:**
  - "No tax line" toggle -- when on, the tax label and rate inputs below are hidden and no tax line is applied.
  - Tax label (text input, required unless "No tax line" is on) -- freelancer-entered text such as "VAT," "GST," or "Sales Tax."
  - Tax rate (numeric percentage input, required unless "No tax line" is on).
- **Save button**, at the bottom of the form.

**Body, locked state (once the project's first invoice has been sent):**
- The currency, tax label, and tax rate fields render as a plain, non-editable display (the same visual treatment FEAT-09 uses for "not edited after sending") with a short explanation directly beneath: "Currency and tax are fixed after your first invoice."
- No Save button is rendered.

**Body, empty-configuration prompt (surfaced when Nadia or the system reaches invoice generation before this screen has ever been completed):**
- A banner above the form: "Set a currency and tax line before you can generate an invoice for this project," present until the first successful save.

**Body, Dana's read-only view:**
- The same locked-state layout regardless of whether the project's first invoice has been sent, with a small "Viewing read-only in a support session" label beneath the header and no Save button.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains directly below the last field.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-01's project billing setup | Screen closes | Standard transition |
| Currency selector | Select | Captures the chosen currency | Field shows chosen currency | Standard selection state |
| "No tax line" toggle | Toggle on | Hides tax label and rate inputs; clears any entered values | Tax label/rate inputs disappear | Fields collapse from the form |
| "No tax line" toggle | Toggle off | Shows tax label and rate inputs, empty | Tax label/rate inputs reappear | Fields expand into the form |
| Tax label input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Tax rate input | Type | Captures numeric input | Field shows entered value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-15.SPEC-003. 2. If valid, save the project's currency and tax fields. | Button shows loading state during save | Success: toast "Currency and tax saved" and the screen re-renders with the saved values. Failure: inline field-level errors per FEAT-15.SPEC-003. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back control -> empty-configuration banner (when present) -> currency selector -> "No tax line" toggle -> tax label input (when visible) -> tax rate input (when visible) -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Lock announcement:** When the screen renders in its locked state, the "currency and tax are fixed after your first invoice" explanation is announced on load.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | N/A -- per product-features.md's States field for FEAT-15 ("Loading: N/A -- configuration is local"), this screen has no distinct loading state of its own: currency, tax label, and tax rate are fields on the Project record already fetched as part of opening the project's billing setup (FEAT-01), so the form renders immediately into Empty, Configured, or Locked with no separate fetch-in-progress appearance | N/A | N/A |
| Empty (unlocked, never configured) | Empty-configuration banner shown above an empty form; Save enabled once required fields are filled | Screen first opens on a project with no currency/tax ever saved | Nadia begins filling the form |
| Filling | Form fields contain user input, Save button enabled | Nadia types or selects in any field | Nadia taps Save or navigates away |
| Saving | Save button shows loading spinner, form fields disabled | Nadia taps Save | Save completes or fails |
| Validation Error | Failed fields highlighted with error messages below them, per FEAT-15.SPEC-003 | Validation fails on save | Nadia corrects the field and re-saves |
| Configured (unlocked) | Form fields show the saved currency, tax label, and tax rate; Save remains enabled for further edits | Save succeeds and the project has not yet had a first invoice sent | Nadia edits and re-saves, or the project's first invoice is sent (transitions to Locked) |
| Locked | Plain, non-editable display of currency/tax with the "fixed after your first invoice" explanation; no Save control | The project's first invoice is sent (FEAT-15.SPEC-004) | Never -- the lock is permanent for this project |
| Error | Error banner "Couldn't save your currency and tax settings. Check your connection and try again." with Retry | Save operation fails for a reason other than validation (e.g., connectivity) | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- currency and tax changes can't be saved right now." at top; form remains viewable but Save is disabled | Connectivity is lost while the screen is open | Connectivity returns and Save re-enables |

## Validation Rules

Validation governed by FEAT-15.SPEC-003 (Currency & Tax Validation Rules). See that spec for all field-level rules on currency and tax label/rate. This screen applies validation on Save; the lock check (FEAT-15.SPEC-004) is applied before the form even renders as editable.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control tap | Project billing setup | FEAT-01 |
| Successful save | This screen, re-rendered with the saved values | -- |

## Data Model

**Creates:** None -- the Project record itself is created by FEAT-01; this screen never creates a Project.
**Reads:** Project -- `currency`, `tax_label`, `tax_rate`, `client` (for the owning client's name shown in the header). Client -- `client_name`, read only for header context.
**Updates:** Project -- `currency`, `tax_label`, `tax_rate`, only while the project is unlocked (FEAT-15.SPEC-004) and only with values that pass FEAT-15.SPEC-003.
**Deletes:** None.

## Business Rules

- Currency and tax field validation (FEAT-15.SPEC-003) is enforced on every save -- Nadia cannot save an unrecognized currency or an invalid tax shape.
- The lock rule (FEAT-15.SPEC-004) governs whether this screen renders editable or locked; once the project's first invoice is sent, this screen never shows an editable field for that project again.
- Access to this screen is governed entirely by FEAT-15.SPEC-005 -- Owen and Priya never reach it, and Dana reaches it only read-only inside a logged support session.
- XBR-17: this screen is the sole place a project's currency and tax line are set, and every downstream invoice (FEAT-09) and proposal (FEAT-02) for this project uses the value saved here.
- An invoice cannot be generated for a project with no currency/tax configured yet -- the empty-configuration banner on this screen is the correction path, enforced by FEAT-15.SPEC-003.
- Two concurrent unlocked edits to the same project's currency/tax resolve last-write-wins, consistent with the dependency map's Project Contention note treating currency/tax as a non-lifecycle field (like renames); this is superseded by the lock the moment the project's first invoice is sent, which always wins over a stale unlocked save.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved currency and tax changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **The project's first invoice is sent by an automation (FEAT-09) while Nadia has this screen open, unlocked** -- The screen is rejected-with-refresh: any in-progress edit is not saved, a message appears ("This project's first invoice was just sent -- currency and tax are now fixed."), and the screen re-renders in its Locked state. Resolution: reject-with-refresh, since a locked configuration can never be overwritten by a stale unlocked edit.
- **Nadia attempts to change currency/tax on a project whose first invoice has already been sent** -- The screen renders no editable fields at all for a locked project, so this is structurally prevented; a direct, out-of-band change attempt (e.g., a replayed request) is refused with the same "fixed after your first invoice" explanation.
- **Nadia toggles "No tax line" on after entering a tax label and rate** -- The entered tax label and rate are discarded from the form; if she toggles it off again, both fields are empty and must be re-entered.
- **Dana opens this screen inside a support session for a project that is not yet locked** -- She sees the current (possibly unconfigured) values read-only; if the empty-configuration state applies, she sees the same "not yet configured" banner but with no ability to act on it.
- **Nadia has this screen open in two sessions at once, and both save a currency/tax change to the same unlocked project before either reloads** -- Resolution: last-write-wins, consistent with the dependency map's Project Contention note (currency/tax are non-lifecycle Project fields, resolved the same way as renames -- the reject-with-refresh treatment there is reserved for the complete/cancel/archive lifecycle transitions only). Both saves succeed individually; the later save's values are what persists on the Project record, and the earlier session's form reflects the newer values the next time it loads or re-saves (it is not live-pushed mid-edit). If the project's first invoice is sent between the two saves, the lock (FEAT-15.SPEC-004) takes precedence: the second save is rejected-with-refresh into the Locked state per the "first invoice sent while open" edge case above, rather than being applied as a last-write-wins update to a now-locked project.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-003 (Currency & Tax Validation Rules) | References (inbound) | Governs every field-level validation rule this screen enforces on save |
| FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice) | References (inbound) | Governs whether this screen renders editable or locked |
| FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules) | References (inbound) | Governs the Access and Visibility table |
| FEAT-15.SPEC-008 (Invoice Currency & Tax Line Application) | References (outbound) | The values saved here are the ones this automation stamps onto every generated invoice |
| FEAT-01 (Client & Project Management) | Navigation (inbound) | Sole entry point into this screen, from project billing setup |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only support-session entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| currency_set | project reference, currency code | Nadia's save succeeds with a currency value (first save or a change while unlocked) | supports success-metrics.md: "Invoice Currency and Tax Correctness" |
| tax_treatment_set | project reference, tax treatment (label + rate, or none) | Nadia's save succeeds with a tax treatment | supports success-metrics.md: "Invoice Currency and Tax Correctness" |
| currency_tax_config_blocked_invoice | project reference | Nadia or the system reaches invoice generation for this project while the empty-configuration banner is showing | N/A -- no success-metrics.md metric measures blocked-generation attempts directly; retained since product-features.md's States field for FEAT-15 names this as the Empty-state behavior this screen must surface |

## Acceptance Criteria

**FEAT-15.SPEC-001-AC-01:** Given Nadia opens billing setup for a project with no currency/tax ever saved, when the screen loads, then she sees the empty-configuration banner above an empty form.

**FEAT-15.SPEC-001-AC-02:** Given Nadia selects a recognized currency and enters a tax label "VAT" and rate "20", when she taps Save, then the configuration is validated via FEAT-15.SPEC-003, saved, and she sees a "Currency and tax saved" toast.

**FEAT-15.SPEC-001-AC-03:** Given Nadia toggles "No tax line" on, when she taps Save with only a currency selected, then the project saves with no tax label or rate and no error is shown.

**FEAT-15.SPEC-001-AC-04:** Given Nadia enters an unrecognized currency, when she taps Save, then the currency field shows the exact error message defined by FEAT-15.SPEC-003 and the save does not proceed.

**FEAT-15.SPEC-001-AC-05:** Given Nadia is viewing a project whose first invoice has already been sent, when the screen loads, then currency and tax render as a plain, non-editable display with the explanation "currency and tax are fixed after your first invoice," and no Save button is shown.

**FEAT-15.SPEC-001-AC-06:** Given Owen attempts to reach this screen directly, when the link resolves, then he is redirected to his portal home with no currency/tax content ever rendered.

**FEAT-15.SPEC-001-AC-07:** Given Priya attempts to reach this screen directly, when the link resolves, then she is redirected to her portal home with no currency/tax content ever rendered.

**FEAT-15.SPEC-001-AC-08:** Given Dana opens this screen inside a support session, when the screen loads, then she sees the full current configuration read-only, with no Save control rendered.

**FEAT-15.SPEC-001-AC-09:** Given Nadia has unsaved changes on this screen, when she taps the back control, then a confirmation dialog appears asking "You have unsaved currency and tax changes. Discard?"

**FEAT-15.SPEC-001-AC-10:** Given Nadia has this screen open and unlocked, when the project's first invoice is sent by FEAT-09 while she is mid-edit, then her edit is not saved, a message explains the lock just took effect, and the screen re-renders Locked.

**FEAT-15.SPEC-001-AC-11:** Given Nadia loses connectivity while filling the form, when she looks at the Save button, then it is disabled and the banner "You're offline -- currency and tax changes can't be saved right now." is shown.

**FEAT-15.SPEC-001-AC-12:** Given Nadia taps Save and the save operation fails for a reason other than validation (e.g., connectivity), when the failure occurs, then she sees "Couldn't save your currency and tax settings. Check your connection and try again." with a Retry action, and the form fields retain her entered values. This screen has no separate load-failure state: per product-features.md's States field for FEAT-15, loading is N/A because currency/tax are fields on the already-fetched Project record.

**FEAT-15.SPEC-001-AC-13:** Given Nadia toggles "No tax line" on after entering a tax label and rate, when she toggles it off again, then both fields are shown empty rather than restoring the discarded values.

**FEAT-15.SPEC-001-AC-14:** Given a project has never had currency/tax configured, when Nadia or an automation reaches invoice generation for it, then generation is blocked and this screen's empty-configuration banner is the correction path.

**FEAT-15.SPEC-001-AC-15:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-15.SPEC-001-AC-16:** Given Nadia successfully saves a valid configuration, when the save completes, then the currency_set and tax_treatment_set events are emitted with the project reference and the values saved.

**FEAT-15.SPEC-001-AC-17:** Given Nadia has this screen open in two sessions on the same unlocked project, when she saves a currency/tax change in each session before either reloads, then both saves succeed and the later save's values persist on the Project record (last-write-wins), unless the project's first invoice was sent between the two saves, in which case the second save is rejected-with-refresh into the Locked state instead.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 9 (loading (N/A), empty, filling, saving, validation error, configured, locked, error, offline) | 9 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Screen Spec: Freelancer Time Zone Setting

## Overview

**Name:** Freelancer Time Zone Setting
**ID:** FEAT-15.SPEC-002
**Type:** Screen
**Purpose:** Nadia views and changes her own time zone, which grounds every reminder day-count and local-time display across the product.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling

## Scope and Non-Goals

**In Scope:**
- Displaying Nadia's current time zone
- Letting Nadia change her time zone
- Saving the new value so it takes effect for all future reminder-day-count and local-time-display computation

**Non-Goals:**
- Rendering any individual date, due date, or time in a viewer's own time zone -- owned entirely by FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule); this screen only captures Nadia's own value
- Any other account setting (name, sign-in email, business details, notification preferences) -- owned entirely by Settings & Account Management (FEAT-21); this screen owns only the `time_zone` field
- Setting a time zone for a client contact -- the product defines time zone only for the freelancer's account; a client contact's own device/browser supplies their local time zone for FEAT-15.SPEC-006's rendering, with no setting screen of their own
- Historical tracking of past time zone values -- the entity carries a single current value with no versioned history, consistent with this feature's Entity-Lifecycle Coverage Matrix, which records only Create/Read/Update for this field

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21 (Settings & Account Management) | Nadia opens her time zone preference from her account settings area | None -- screen loads her current value |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Change and save her own time zone | -- |
| Owen (Client Primary Contact) | None -- this is a freelancer-account setting with no client-facing surface | None | This screen has no route reachable from Owen's portal; a direct attempt resolves the same as any out-of-scope client link, per XBR-09 |
| Priya (Client Reviewer Contact) | None -- this is a freelancer-account setting with no client-facing surface | None | This screen has no route reachable from Priya's portal; a direct attempt resolves the same as any out-of-scope client link, per XBR-09 |
| Dana (Support Operator) | Full screen, read-only (Freelancer Account is View-only for Dana per the Access Matrix) | View only | The time zone selector and Save control are not rendered; the current value is shown as plain text |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Redirected to sign-in; any unsaved selection is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Time Zone" with a back control returning to account settings (FEAT-21).

**Body:** A single field:
- Time zone selector (selection input, required, always has a value): a searchable list of recognized time zones, showing the current UTC offset next to each option. Preselected to Nadia's currently saved time zone (or the default captured at sign-up, FEAT-20, if never changed).
- Helper text directly beneath the selector: "This sets the time zone used for your payment reminder countdowns and any dates or times shown to you."

**Footer:** Save button.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout, full width; Save remains directly below the helper text.
- **Medium size class and above:** Uniform scaling, no structural change -- the form is capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigate to FEAT-21 (account settings) | Screen closes | Standard transition |
| Time zone selector | Select | Captures the chosen time zone | Field shows chosen time zone and its UTC offset | Standard selection state |
| Save button | Tap | Saves the freelancer account's `time_zone` field | Button shows loading state during save | Success: toast "Time zone updated" and the selector reflects the saved value. Failure: error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back control -> time zone selector -> Save.
- **Save feedback:** The "Time zone updated" toast is announced on success; on save failure, focus moves to the error banner.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; the time zone selector's search input accepts typed queries with keyboard-navigable results.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Selector preselected to Nadia's current time zone, Save enabled | Screen first opens | Nadia changes the selection |
| Changed | Selector shows the newly chosen time zone, Save enabled | Nadia selects a different time zone | Nadia taps Save or navigates away |
| Saving | Save button shows loading spinner, selector disabled | Nadia taps Save | Save completes or fails |
| Error | Error banner "Couldn't update your time zone. Check your connection and try again." with Retry | Save operation fails | Nadia taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this change will be saved when you reconnect." at top; selector remains usable, Save queues the change locally | Connectivity lost while the screen is open | Connectivity restored -- queued save submits automatically and the standard success feedback appears |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Time zone | Required -- must be a recognized time zone from the selector's list; the field is never left blank since a value is always preselected | On submit | "Choose a time zone to continue." |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back control tap | Account settings | FEAT-21 |
| Successful save | This screen, re-rendered with the saved value | -- |

## Data Model

**Creates:** None -- the Freelancer Account record itself is created at sign-up (FEAT-20), which sets an initial time zone default; this screen only lets Nadia change it afterward.
**Reads:** Freelancer Account -- `time_zone`.
**Updates:** Freelancer Account -- `time_zone`.
**Deletes:** None.

## Business Rules

- Saving a new time zone recomputes the basis used for every future reminder-day-count and local-time display for Nadia's account, governed by FEAT-15.SPEC-006 -- this screen does not itself recompute anything, only writes the new value.
- Time zone is the only Freelancer Account field this feature owns; every other account field is out of scope here and lives in FEAT-21.
- There is no concurrent-edit conflict handling beyond last-write-wins: the dependency map's Contention note for Freelancer Account states two open sessions of Nadia's resolve last-write-wins per field, and `time_zone` is not the sign-in-email exception that requires re-verification.

## Edge Cases

- **Nadia navigates away with an unsaved selection** -- Confirmation dialog: "You have an unsaved time zone change. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Nadia changes her time zone from two open sessions in quick succession** -- Last-write-wins, consistent with the dependency map's Contention note for the Freelancer Account entity: the later save's value is what persists, and the earlier session's selector re-renders to the newer value on its next load.
- **Nadia saves the same time zone she already had selected** -- Save proceeds normally and shows the same success feedback; no distinct "no change" state is defined.
- **Network failure during save** -- Error banner: "Couldn't update your time zone. Check your connection and try again." with a Retry button. Selection is preserved.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | Triggers (outbound) | The value saved here is the basis this rule uses for every reminder-day-count and local-time computation grounded in Nadia's time zone |
| FEAT-21 (Settings & Account Management) | Navigation (inbound) | Sole entry point into this screen, from the freelancer's account settings area |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| timezone_set | new time zone value | Nadia's save succeeds | N/A -- no success-metrics.md metric measures time zone configuration directly; retained since product-features.md's Signals field for FEAT-15 names `timezone_set` explicitly as a required signal, and the value this event carries is what makes "Invoice Currency and Tax Correctness"-adjacent local-time correctness verifiable downstream in FEAT-15.SPEC-006 |

## Acceptance Criteria

**FEAT-15.SPEC-002-AC-01:** Given Nadia opens her time zone preference for the first time, when the screen loads, then the selector shows the time zone default captured at her sign-up.

**FEAT-15.SPEC-002-AC-02:** Given Nadia selects a different time zone, when she taps Save, then the change is saved and she sees a "Time zone updated" toast.

**FEAT-15.SPEC-002-AC-03:** Given Nadia has just saved a new time zone, when any future reminder day-count or local-time display is computed for her account, then it uses the newly saved value, per FEAT-15.SPEC-006.

**FEAT-15.SPEC-002-AC-04:** Given Dana opens this screen inside a support session, when the screen loads, then she sees Nadia's current time zone as plain text with no selector or Save control rendered.

**FEAT-15.SPEC-002-AC-05:** Given Nadia has an unsaved time zone selection, when she taps the back control, then a confirmation dialog appears asking "You have an unsaved time zone change. Discard?"

**FEAT-15.SPEC-002-AC-06:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-15.SPEC-002-AC-07:** Given Nadia changes her time zone in one session while an older session of hers is also open on this screen, when both saves land, then the later save's value persists and the older session's selector reflects it on its next load (last-write-wins).

**FEAT-15.SPEC-002-AC-08:** Given the save operation fails for a connectivity reason, when the failure occurs, then Nadia sees "Couldn't update your time zone. Check your connection and try again." with a Retry action.

**FEAT-15.SPEC-002-AC-09:** Given Nadia loses connectivity after selecting a new time zone, when she taps Save, then the banner "You're offline -- this change will be saved when you reconnect." appears and the change is submitted automatically once connectivity returns.

**FEAT-15.SPEC-002-AC-10:** Given Nadia saves the same time zone she already had, when the save completes, then it succeeds with the same success feedback as any other save.

**FEAT-15.SPEC-002-AC-11:** Given Nadia successfully saves a time zone, when the save completes, then the timezone_set event is emitted with the new value.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (loaded, changed, saving, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Currency & Tax Validation Rules

## Overview

**Name:** Currency & Tax Validation Rules
**ID:** FEAT-15.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs which currencies are acceptable and what shape a tax line may take on a project, and blocks invoice generation with a specific, correctable message on an invalid configuration.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Project (the `currency`, `tax_label`, and `tax_rate` fields)

## Scope and Non-Goals

**In Scope:**
- Field-level validation for the project's `currency`, `tax_label`, and `tax_rate` fields
- The cross-field rule governing when `tax_label`/`tax_rate` are required versus optional
- Blocking invoice generation when a project's currency/tax configuration is invalid or missing
- The single source of truth both FEAT-15.SPEC-001 (screen-level save) and FEAT-15.SPEC-008 (generation-time application) apply, per this feature's Shared Validation note

**Non-Goals:**
- Automatic, jurisdiction-specific tax calculation -- excluded per scope-boundaries.md (SC-16): this spec validates the *shape* of a freelancer-entered tax line, never computes a rate from a country or region
- Who may change a project's currency/tax -- owned entirely by FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules); this spec governs data shape, not access
- Whether a project's configuration can still be changed at all -- owned entirely by FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice); this spec's rules apply only while a project remains unlocked
- Currency conversion or exchange-rate validation -- excluded per XBR-18: the product never converts between currencies, so no conversion-related rule exists here

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| currency | enum (recognized world currency code) | The project's billing currency |
| tax_label | text | Freelancer-entered tax line label (e.g., "VAT," "GST," "Sales Tax"), or absent when no tax applies |
| tax_rate | number (percentage) | Freelancer-entered tax rate applied to the invoice subtotal, or absent when no tax applies |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | On Save, while the project is unlocked |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | At invoice-generation time, as a precondition before stamping currency/tax onto the new invoice |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| currency | Must be a recognized world currency | Always | On submit | "Choose a recognized currency to continue." | Yes |
| tax_label | Required, non-empty, max 60 characters | When "No tax line" is not selected | On submit | "Enter a tax label (for example, VAT, GST, or Sales Tax)." / "Tax label must be 60 characters or fewer." | Yes |
| tax_label | No validation beyond data type | When "No tax line" is selected | -- | -- | -- |
| tax_rate | Required; numeric; between 0 and 100 inclusive | When "No tax line" is not selected | On submit | "Enter a tax rate between 0 and 100." | Yes |
| tax_rate | No validation beyond data type | When "No tax line" is selected | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Tax shape completeness | tax_label, tax_rate | Both must be present together, or both absent ("no tax line"); a project can never carry a label with no rate, or a rate with no label | "A tax line needs both a label and a rate -- or select 'No tax line'." |
| Configuration completeness for generation | currency, tax_label, tax_rate | A project must have a valid currency and a complete tax treatment (label + rate, or explicitly none) before any invoice can be generated for it | "Set this project's currency and tax line before generating an invoice." |

## Authorization Rules

Authorization for who may reach and act on this configuration is owned entirely by FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules), which is this feature's single authoritative home for the role-action matrix on the Project currency/tax fields (per this feature's Shared UI Patterns note: "FEAT-15.SPEC-005 is the single source of truth both other specs reference rather than restating the gate"). This spec's rules below apply only after FEAT-15.SPEC-005 has already permitted the action.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set or change currency/tax (subject to these validation rules) | See FEAT-15.SPEC-005 | See FEAT-15.SPEC-005 | See FEAT-15.SPEC-005 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| currency | No default -- the field starts empty and must be explicitly chosen | On create (first save) | Yes -- Nadia always chooses explicitly |
| tax_label / tax_rate | No default -- starts empty; "No tax line" is not preselected either way, so Nadia makes an explicit choice | On create (first save) | Yes -- Nadia always chooses explicitly |

## Business Rules

- XBR-17: a project's currency and tax line must be set before its first invoice, and the currency cannot change once the first invoice is sent (the lock itself is FEAT-15.SPEC-004's rule; this spec supplies the validity check that must pass before that first invoice can exist).
- This spec is the single source of truth for currency and tax validity across the feature -- FEAT-15.SPEC-001 and FEAT-15.SPEC-008 both apply it rather than re-deriving the rule, per this feature's Shared Validation note.
- Validation runs identically whether the save originates from Nadia's screen action (FEAT-15.SPEC-001) or from the generation-time precondition check (FEAT-15.SPEC-008) -- the product definition establishes no separate rule set for either context.
- A tax rate of exactly 0 with a tax label present is valid (e.g., a zero-rated tax category) and is distinct from "No tax line": the former still carries a label and appears on the invoice; the latter carries neither.

## Edge Cases

- **Tax rate entered as exactly 0** -- Passes validation when a tax label is also present (a zero-rated line is still a defined tax treatment, distinct from "No tax line").
- **Tax rate entered as exactly 100** -- Passes validation (upper boundary is inclusive).
- **Tax rate entered as 100.01 or -0.01** -- Fails validation with "Enter a tax rate between 0 and 100."
- **Tax label at exactly 60 characters** -- Passes validation. 61 characters shows the length error.
- **Tax label left as whitespace only** -- Treated as empty; fails the "required" rule with the same message as a blank field.
- **"No tax line" toggled after a valid label and rate were entered, then generation is attempted immediately** -- The cross-field completeness rule evaluates the current state (no tax line, explicitly chosen) as complete; generation is not blocked.
- **Currency chosen but tax fields left in an inconsistent state (label present, rate absent) due to a partial client-side interruption** -- The cross-field "Tax shape completeness" rule catches this on submit and blocks the save with its exact error message, regardless of how the inconsistent state arose.

## Acceptance Criteria

**FEAT-15.SPEC-003-AC-01:** Given Nadia selects a recognized world currency, when she submits, then the currency field passes validation with no error.

**FEAT-15.SPEC-003-AC-02:** Given Nadia leaves the currency field unset, when she submits, then she sees "Choose a recognized currency to continue." and the save is blocked.

**FEAT-15.SPEC-003-AC-03:** Given Nadia has "No tax line" off and leaves the tax label empty, when she submits, then she sees "Enter a tax label (for example, VAT, GST, or Sales Tax)." and the save is blocked.

**FEAT-15.SPEC-003-AC-04:** Given Nadia enters a tax rate of 150, when she submits, then she sees "Enter a tax rate between 0 and 100." and the save is blocked.

**FEAT-15.SPEC-003-AC-05:** Given Nadia enters a tax rate of exactly 0 alongside a tax label "Zero-Rated VAT", when she submits, then the configuration passes validation and saves.

**FEAT-15.SPEC-003-AC-06:** Given Nadia selects "No tax line", when she submits with only a currency chosen, then the configuration passes validation with no tax label or rate.

**FEAT-15.SPEC-003-AC-07:** Given a project reaches an inconsistent state with a tax label present but no rate, when a save or generation-time check evaluates it, then it is blocked with "A tax line needs both a label and a rate -- or select 'No tax line'."

**FEAT-15.SPEC-003-AC-08:** Given a project has never had currency or tax configured, when invoice generation (FEAT-15.SPEC-008) checks its configuration, then generation is blocked with "Set this project's currency and tax line before generating an invoice."

**FEAT-15.SPEC-003-AC-09:** Given a project has a valid currency and a complete tax treatment, when invoice generation checks its configuration, then the precondition passes and generation proceeds.

**FEAT-15.SPEC-003-AC-10:** Given Nadia enters a tax label of exactly 60 characters, when she submits, then it passes validation; at 61 characters it is blocked with the length error message.

**FEAT-15.SPEC-003-AC-11:** Given the same currency and tax rules apply on both the configuration screen (FEAT-15.SPEC-001) and generation-time application (FEAT-15.SPEC-008), when either context evaluates an identical configuration, then the outcome (pass or fail, and the exact error) is identical.

**FEAT-15.SPEC-003-AC-12:** Given Nadia enters a tax rate of exactly 100, when she submits, then it passes validation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 1 (delegated to FEAT-15.SPEC-005) | 1 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Currency & Tax Lock After First Invoice

## Overview

**Name:** Currency & Tax Lock After First Invoice
**ID:** FEAT-15.SPEC-004
**Type:** Logic/Rule
**Purpose:** Locks a project's currency and its paired tax line the moment its first invoice is sent, and blocks any later change attempt with an explanation.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Project (the `currency`, `tax_label`, and `tax_rate` fields; the lock state itself is derived from Invoice)

## Scope and Non-Goals

**In Scope:**
- The state transition from editable to locked, fired by a project's first invoice being sent
- The permanent, no-unlock nature of the lock for that project
- Blocking any change attempt on a locked project's currency/tax fields, with an explanation
- Reading Invoice to determine whether a project already has a sent invoice

**Non-Goals:**
- Deciding what counts as a valid currency or tax shape -- owned entirely by FEAT-15.SPEC-003 (Currency & Tax Validation Rules); this spec governs *whether a change may be attempted at all*, not whether an attempted value would be valid
- Who may attempt a change while a project is unlocked -- owned entirely by FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules); this spec governs the lock state, not role entitlement
- Sending the first invoice itself, or any other invoice lifecycle behavior -- owned entirely by Invoice Generation & Sending (FEAT-09); this spec only reacts to the "first invoice sent" event as a trigger
- Any versioned history of pre-lock configuration changes -- excluded per this feature's Non-Goals: because the lock is permanent and irreversible, no scenario produces multiple historical configurations to version

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| currency | enum (recognized world currency code) | The project's billing currency; the field this rule locks |
| tax_label | text | Freelancer-entered tax line label, or absent; the field this rule locks |
| tax_rate | number (percentage) | Freelancer-entered tax rate, or absent; the field this rule locks |
| (derived) lock_state | derived boolean | Whether the project's currency/tax fields are currently editable or locked; derived from whether any Invoice belonging to this project has ever reached status Sent or later |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | On screen load (to decide editable vs. locked rendering) and on any save attempt |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | Immediately after a project's first invoice is sent, to trigger the state transition |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| currency | No new value may be written once the project's lock_state is Locked | Project has a sent invoice | On any change attempt | "Currency is fixed once billing has begun for this project." | Yes |
| tax_label | No new value may be written once the project's lock_state is Locked | Project has a sent invoice | On any change attempt | "Tax label is fixed once billing has begun for this project." | Yes |
| tax_rate | No new value may be written once the project's lock_state is Locked | Project has a sent invoice | On any change attempt | "Tax rate is fixed once billing has begun for this project." | Yes |
| lock_state | No validation beyond data type -- this is a system-derived field, never directly entered by any user | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Locked triad | currency, tax_label, tax_rate | The lock applies to all three fields together -- there is no partial lock where one field remains editable while the others are fixed | "Currency and tax are fixed once billing has begun for this project." |

## Authorization Rules

The role-action matrix for who may attempt a currency/tax change is owned by FEAT-15.SPEC-005. This spec's own authorization concern is narrower: whether the lock state itself blocks even an otherwise-permitted actor.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Change currency/tax on a locked project | Nadia (Freelancer) | Never, once lock_state is Locked -- even though FEAT-15.SPEC-005 grants Nadia Full access to this configuration while unlocked | No editable field is ever rendered for a locked project (FEAT-15.SPEC-001); a direct, out-of-band attempt is refused with "Currency and tax are fixed once billing has begun for this project." |
| Change currency/tax on an unlocked project | Nadia (Freelancer) | Always, while lock_state is Editable (subject to FEAT-15.SPEC-003's validation and FEAT-15.SPEC-005's access grant) | -- |
| View a locked project's currency/tax | Nadia (Freelancer), Dana (Support Operator) | Always -- the lock never restricts viewing, only writing | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| lock_state | Derived: Editable by default; transitions to Locked the moment any Invoice belonging to this project first reaches status Sent | Recomputed at the moment the project's first invoice is sent (FEAT-15.SPEC-008); read on every access to FEAT-15.SPEC-001 | No -- lock_state is never directly set by any user, and once Locked it is never overridable by anyone, including Nadia |

## Business Rules

- XBR-17: the currency cannot change once the first invoice is sent -- this spec is the rule's sole enforcement point.
- The lock is permanent for a project: there is no unlock path, no administrative override, and no scenario (including account support access) that reopens a locked project's currency/tax fields, per this feature's Entity-Lifecycle Coverage Matrix.
- The lock fires from a cross-feature trigger: "a project's first invoice is sent (event from Invoice Generation & Sending, FEAT-09)," per this feature's Side-Effect Inventory and Internal Dependency Map.
- The lock check (this spec) always runs before the validation check (FEAT-15.SPEC-003): a locked project blocks any change attempt outright, regardless of whether the attempted new value would itself be valid.
- A credit note or a corrected/re-sent invoice on the same project does not create a second "first invoice sent" event and does not re-lock or re-evaluate an already-locked project -- the lock is keyed to the project's *first* invoice reaching Sent, a one-time, non-repeating transition.

## Edge Cases

- **Two of Nadia's sessions both have the currency/tax screen open, unlocked, when the project's first invoice is sent by FEAT-09** -- Both sessions are rejected-with-refresh: neither in-flight edit is saved, and both re-render to the Locked state on their next interaction (consistent with FEAT-15.SPEC-001's Edge Cases).
- **Nadia attempts to save a currency/tax change at the exact moment the first invoice's send is being processed (race between save and lock)** -- The lock check re-evaluates at the moment of the save attempt, not at the moment the screen was loaded: if the invoice's Sent status has landed by the time the save is processed, the save is refused even though the screen appeared editable when it was opened.
- **A project has an unsent (Draft or Generated, not yet Sent) first invoice** -- lock_state remains Editable; only a status of Sent or later on any invoice belonging to the project triggers the lock. A draft invoice never locks the project.
- **The project's would-be first invoice is voided or fails to send before reaching Sent** -- No lock fires; the project remains Editable until an invoice genuinely reaches Sent.
- **Dana views a locked project's currency/tax inside a support session** -- She sees it exactly as Nadia would (read-only display with the lock explanation); the lock applies identically regardless of viewer, and Dana's session grants no override of any kind.

## Acceptance Criteria

**FEAT-15.SPEC-004-AC-01:** Given a project has no invoice that has ever reached Sent, when Nadia opens its currency/tax configuration, then the fields render editable and lock_state is Editable.

**FEAT-15.SPEC-004-AC-02:** Given a project's first invoice is sent by FEAT-09, when that event lands, then the project's lock_state transitions to Locked immediately.

**FEAT-15.SPEC-004-AC-03:** Given a project's lock_state is Locked, when Nadia opens its currency/tax configuration, then no editable field is rendered and the explanation "currency and tax are fixed after your first invoice" is shown.

**FEAT-15.SPEC-004-AC-04:** Given a project's lock_state is Locked, when any out-of-band attempt tries to change its currency, then it is refused with "Currency is fixed once billing has begun for this project."

**FEAT-15.SPEC-004-AC-05:** Given a project has a Draft invoice that has never reached Sent, when Nadia opens its currency/tax configuration, then the fields remain editable -- the draft alone does not trigger the lock.

**FEAT-15.SPEC-004-AC-06:** Given a project's would-be first invoice is voided before reaching Sent, when Nadia later opens its currency/tax configuration, then the fields remain editable and no lock has fired.

**FEAT-15.SPEC-004-AC-07:** Given Nadia's save attempt reaches the server after the project's first invoice has already been sent in the interim, when the lock check runs, then the save is refused even though the screen appeared editable when it was opened.

**FEAT-15.SPEC-004-AC-08:** Given a project's currency/tax is locked, when Dana views it inside a support session, then she sees the identical locked, read-only display Nadia would see -- no override is available to her.

**FEAT-15.SPEC-004-AC-09:** Given a project's currency/tax is already locked, when a credit note or a corrected invoice is later issued for that project, then no re-evaluation of the lock occurs and the project remains Locked exactly as before.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Currency & Tax Configuration Access Rules

## Overview

**Name:** Currency & Tax Configuration Access Rules
**ID:** FEAT-15.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may view or change a project's currency/tax configuration -- Nadia Full, Dana read-only in a logged session, Owen and Priya no direct access to the configuration surface at all.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Project (the `currency`, `tax_label`, and `tax_rate` fields, and the configuration screen that surfaces them)

## Scope and Non-Goals

**In Scope:**
- The complete role-action matrix for viewing and changing a project's currency/tax configuration
- The exact experience for each role that is denied access, including the two roles with no access at all (Owen, Priya)
- This spec is the single source of truth FEAT-15.SPEC-001 (screen) and FEAT-15.SPEC-003 (validation) reference rather than restating the gate, per this feature's Shared UI Patterns note

**Non-Goals:**
- What shape a valid currency or tax line takes -- owned entirely by FEAT-15.SPEC-003 (Currency & Tax Validation Rules); this spec governs who may attempt an action, not whether the attempted value is valid
- Whether a project's configuration can still be changed at all, independent of role -- owned entirely by FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice); this spec governs role entitlement, not lock state
- Dana's support session lifecycle (how it opens, how long it lasts, how it is announced) -- owned entirely by Operator Support Access (FEAT-31); this spec only states what Dana may do with currency/tax data once inside a session
- Access to invoice content that happens to display a project's currency and tax line after generation -- owned by Invoice Generation & Sending (FEAT-09, e.g. its own Access Rules); this spec governs only the configuration surface, not the resulting invoice

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| currency | enum (recognized world currency code) | The project's billing currency |
| tax_label | text | Freelancer-entered tax line label, or absent |
| tax_rate | number (percentage) | Freelancer-entered tax rate, or absent |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | On screen entry (deciding whether the screen renders at all, and in which mode) and on every save attempt |

## Field Validation Rules

Not applicable -- this spec governs role access to the configuration screen and its fields, not the shape of the values themselves. Field-level validation is defined entirely by FEAT-15.SPEC-003.

## Cross-Field Rules

Not applicable -- no cross-field rule in this spec; cross-field validation is defined entirely by FEAT-15.SPEC-003.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View project currency/tax configuration | Nadia (Freelancer) | Always, for her own projects | -- |
| View project currency/tax configuration | Dana (Support Operator) | Only inside a logged, time-limited support session (FEAT-31) | Outside an open support session, this screen is not reachable at all -- no route exists for Dana without an active session |
| View project currency/tax configuration | Owen (Client Primary Contact) | Never | The configuration screen is not shown at all -- absent from his navigation entirely; a direct link redirects to his portal home with no partial content ever rendered |
| View project currency/tax configuration | Priya (Client Reviewer Contact) | Never | The configuration screen is not shown at all -- absent from her navigation entirely; a direct link redirects to her portal home with no partial content ever rendered |
| Set or change project currency/tax | Nadia (Freelancer) | For her own projects, while lock_state is Editable (FEAT-15.SPEC-004) and the attempted value passes FEAT-15.SPEC-003 | Once locked, no editable control is shown; a direct out-of-band attempt is refused per FEAT-15.SPEC-004's exact message |
| Set or change project currency/tax | Dana (Support Operator) | Never | No Save control or any editable input is ever rendered in a support session, regardless of the project's lock state; a direct, out-of-band change attempt is refused and the screen re-renders in its read-only form |
| Set or change project currency/tax | Owen (Client Primary Contact) | Never | The configuration screen is not shown at all, so no change control is ever reachable |
| Set or change project currency/tax | Priya (Client Reviewer Contact) | Never | The configuration screen is not shown at all, so no change control is ever reachable |

## Defaults and Derivations

Not applicable -- this spec governs access, not data defaults. Field defaults and derivations are defined entirely by FEAT-15.SPEC-003.

## Business Rules

- This spec is the single authoritative home for the role-action matrix on the Project currency/tax configuration; FEAT-15.SPEC-001 and FEAT-15.SPEC-003 reference it rather than restating the gate, per this feature's Shared UI Patterns note.
- Owen and Priya's exclusion is total, not merely read-restricted: the configuration screen is not shown at all, distinct from a "view but not edit" pattern -- consistent with the Brief's Side-Effect Inventory entry "Owen or Priya attempts to reach the currency/tax configuration screen -> Show nothing."
- Dana's access is always read-only and always session-scoped: she never reaches this screen outside an active support session, and even inside one she holds no change capability of any kind, consistent with XBR-29 (operator sessions are read-only in every feature).
- This matrix governs the configuration surface only; it does not govern what Owen sees on an invoice once one has been generated (FEAT-09 owns that surface and its own access rules) -- Owen's Access field in product-features.md states he sees the resulting amounts and tax line on his invoices, which is a distinct, downstream surface from this one.

## Edge Cases

- **Owen's support-request context somehow includes a link to this configuration screen (e.g., forwarded from an internal tool)** -- The redirect-to-portal-home behavior applies regardless of how the link was obtained; no context bypasses the role check.
- **Dana's support session expires while she is viewing this screen** -- The screen becomes unreachable the moment the session closes (governed by FEAT-31); any further attempt to view or interact resolves as if she had never had a session.
- **Nadia is viewing this screen when a support session for her account happens to open (Dana starts viewing concurrently)** -- No conflict: Dana's session is read-only and makes no writes, so Nadia's own editing (subject to FEAT-15.SPEC-003 and FEAT-15.SPEC-004) proceeds unaffected by Dana's concurrent, separate view.
- **A future role is added to the product that the Access Matrix does not yet name** -- Out of scope for this spec: per pipeline-rules.md's grounded-roles constraint, this spec's matrix covers exactly the four roles the Access Matrix in user-persona.md establishes today (Nadia, Owen, Priya, Dana), and no invented role is added here.

## Acceptance Criteria

**FEAT-15.SPEC-005-AC-01:** Given Nadia opens the currency/tax configuration for one of her own projects, when the screen loads, then she has full view and change access, subject to lock state.

**FEAT-15.SPEC-005-AC-02:** Given Dana has no active support session on a freelancer's account, when she attempts to reach that account's currency/tax configuration screen, then no route to it exists.

**FEAT-15.SPEC-005-AC-03:** Given Dana has an active, logged support session open, when she views a project's currency/tax configuration, then she sees it read-only with no Save control or editable input rendered.

**FEAT-15.SPEC-005-AC-04:** Given Owen attempts to reach the currency/tax configuration screen by any direct link, when the link resolves, then he is redirected to his portal home with no configuration content ever rendered.

**FEAT-15.SPEC-005-AC-05:** Given Priya attempts to reach the currency/tax configuration screen by any direct link, when the link resolves, then she is redirected to her portal home with no configuration content ever rendered.

**FEAT-15.SPEC-005-AC-06:** Given Dana is inside a support session, when she attempts an out-of-band change to a project's currency, then it is refused and the screen re-renders in its read-only form.

**FEAT-15.SPEC-005-AC-07:** Given Nadia's project is unlocked, when she attempts to change its currency/tax, then the action is allowed, subject to passing FEAT-15.SPEC-003's validation.

**FEAT-15.SPEC-005-AC-08:** Given Nadia's project is locked (FEAT-15.SPEC-004), when she attempts to change its currency/tax, then no editable control is shown and any out-of-band attempt is refused per FEAT-15.SPEC-004's exact message.

**FEAT-15.SPEC-005-AC-09:** Given Dana's support session expires while she is viewing this screen, when the session closes, then the screen becomes unreachable to her as if no session had ever existed.

**FEAT-15.SPEC-005-AC-10:** Given Owen later opens an invoice generated for one of his company's projects, when he views its content, then he sees the resulting currency and tax line on the invoice itself -- a distinct surface governed by FEAT-09, not by this spec's configuration-screen access rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-15.SPEC-003) | 0 |
| Cross-Field Rules | 0 (N/A -- owned by FEAT-15.SPEC-003) | 0 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-15.SPEC-003) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Time Zone & Local Date/Time Display Rule

## Overview

**Name:** Time Zone & Local Date/Time Display Rule
**ID:** FEAT-15.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs how dates, due dates, and times are rendered in each viewer's own time zone and familiar format, and establishes the freelancer's time zone as the basis for reminder day counts.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Freelancer Account (the `time_zone` field, as the basis for reminder counting) and the display rule applied wherever any date, due date, or time appears across the product

## Scope and Non-Goals

**In Scope:**
- The rendering rule: any date, due date, or time shown on any screen (across this and other features) displays in that viewer's own time zone and a familiar date format
- The counting rule: elapsed-day computations for reminders use the freelancer's time zone specifically, never the viewer's
- Establishing this spec as the cross-feature authority the Automated Payment Reminders schedule (FEAT-11) and Invoice Generation & Sending (FEAT-09) consume for their own date/time display and counting

**Non-Goals:**
- Setting or changing the freelancer's own time zone value -- owned entirely by FEAT-15.SPEC-002 (Freelancer Time Zone Setting); this spec only consumes the value that screen captures
- Translating date or time text into a language other than English -- excluded per scope-boundaries.md (SC-20): "English only at launch"; this spec adapts *format* (day/month order, 12- vs 24-hour convention as derived from locale) and *time zone*, never language
- The reminder schedule's own trigger logic, pause states, or send cadence -- owned entirely by Automated Payment Reminders (FEAT-11); this spec supplies only the time-zone basis that schedule's day counts are computed against
- Determining a client contact's own time zone value -- the product infers it from the viewing device/browser at render time rather than capturing and storing it as an explicit setting, since no screen in the product definition offers a client contact a time zone preference to set

## Governed Entity

**Entity:** Freelancer Account
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| time_zone | enum (recognized time zone) | The freelancer's own time zone, set via FEAT-15.SPEC-002; the basis this spec uses for reminder day counts |

**Note on scope:** the *rendering* half of this spec's purpose applies to every date, due date, and time shown on any screen product-wide, not to a single stored entity field -- there is no separate "display format" entity to enumerate; the field table above covers the one stored value (`time_zone`) this spec's rules read.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | Any date shown on this screen (none currently -- this screen carries no dates) is N/A; listed for completeness of the feature's own screens |
| FEAT-09 (Invoice Generation & Sending) | Invoice detail, invoice list, and any invoice email | Issue dates, due dates, and reminder history render in each viewer's own time zone and date format whenever displayed |
| FEAT-11 (Automated Payment Reminders) | The reminder schedule's day-3/day-10 evaluation | Elapsed-day counts are computed against the freelancer's time zone, not the viewer's, per XBR-15 |
| FEAT-12 (Freelancer Financial Dashboard) | Dashboard and drill-down date displays | Any date shown (e.g., invoice issue/due dates in a drill-down) renders in Nadia's own time zone and format |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| time_zone | Must be a recognized time zone (validated by FEAT-15.SPEC-002's screen-level input, which always presents a value from a closed list) | Always | On read by this spec | N/A -- this spec only reads the value; validation of the value itself belongs to FEAT-15.SPEC-002 | No |

## Cross-Field Rules

Not applicable -- this spec's rendering rule operates on individual date/time fields across many other entities (Invoice issue_date/due_date, Milestone target_date, Reminder Log scheduled_for/sent_at, etc.), each rendered independently in the viewer's own time zone; there is no cross-field interaction between them for the purposes of this display rule.

## Authorization Rules

Not applicable -- this is a display and computation rule, not an access-gated action. Every role that can view a date, due date, or time anywhere in the product (per that surface's own Access Matrix rules) sees it rendered under this rule; this spec grants no additional visibility and restricts none.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| Rendered date/time (any date, due date, or time on any screen) | Derived: the underlying stored instant, converted to the viewing party's own time zone and shown in a familiar date format for that viewer | Every time a date, due date, or time is displayed, on every screen | No -- the rendering is automatic and not a per-screen setting; a viewer changes what they see only by changing their own time zone (Nadia via FEAT-15.SPEC-002; a client contact via their device/browser) |
| Reminder day count (day 3 / day 10 elapsed-day evaluation) | Derived: elapsed calendar days since the invoice's due date, counted using the freelancer's `time_zone` regardless of which viewer or process is evaluating it | Every time the reminder schedule (FEAT-11) evaluates whether a reminder is due | No -- the freelancer's time zone is always the counting basis; it is not evaluated in the viewer's time zone under any circumstance |

## Business Rules

- XBR-15: overdue reminders are counted in the freelancer's time zone, not the viewer's -- this spec is the rule's sole definition; FEAT-11 consumes it.
- The rendering rule applies uniformly across every feature that displays a date, due date, or time: this spec is the cross-feature authority both FEAT-09 and FEAT-12 reference for their own date displays, rather than each feature deriving its own time-zone behavior.
- A viewer's own time zone for *display* purposes (as opposed to Nadia's for *counting* purposes) comes from the viewing device/browser for client contacts, and from the stored `time_zone` field (FEAT-15.SPEC-002) for Nadia herself -- the two are the same source when Nadia is the viewer of her own data.
- Date *format* adapts to each viewer's familiar convention (e.g., day/month/year vs. month/day/year ordering); this is a formatting adaptation only and carries no translated text, consistent with scope-boundaries.md (SC-20).

## Edge Cases

- **Nadia and Owen are in different time zones and both view the same invoice's due date at the same moment** -- Each sees the due date rendered independently in their own time zone; the two displayed values may show different calendar dates for the same underlying due date/time, and this is expected, not a defect.
- **Nadia changes her time zone (FEAT-15.SPEC-002) while a reminder's day count is mid-evaluation** -- The evaluation in flight completes using the time zone value it read at the start of that evaluation cycle; the next evaluation cycle uses the newly saved value. No reminder is double-counted or skipped due to the change.
- **A client contact's device/browser reports no detectable time zone (e.g., a misconfigured environment)** -- The portal falls back to a neutral reference time zone (UTC) for that viewer's session only, with a plain indicator that displayed times may not match their local time; Nadia's own display and the reminder day-count basis are unaffected, since neither depends on a client contact's time zone.
- **A due date lands exactly at a day boundary when converted between the freelancer's time zone and a viewer's time zone** -- Each viewer's rendering uses its own boundary independently; the freelancer's reminder day count uses only the freelancer's own boundary, so the two computations do not need to agree with each other.
- **Nadia views her own dashboard, where she is simultaneously the freelancer (reminder-count basis) and the viewer (display basis)** -- Both computations use the same stored `time_zone` value, so her displayed dates and her reminder day counts are always mutually consistent for her own view.

## Acceptance Criteria

**FEAT-15.SPEC-006-AC-01:** Given Nadia has set her time zone via FEAT-15.SPEC-002, when she views an invoice's due date, then it renders in her own time zone and a familiar date format for her.

**FEAT-15.SPEC-006-AC-02:** Given Owen is in a different time zone than Nadia, when he views the same invoice's due date, then it renders in his own time zone, independent of how it renders for Nadia.

**FEAT-15.SPEC-006-AC-03:** Given an invoice is overdue, when the reminder schedule (FEAT-11) evaluates whether a day-3 reminder is due, then the elapsed-day count is computed using Nadia's time zone, not Owen's.

**FEAT-15.SPEC-006-AC-04:** Given Nadia changes her time zone, when the next reminder evaluation cycle runs, then it uses the newly saved time zone as its counting basis.

**FEAT-15.SPEC-006-AC-05:** Given a reminder evaluation is already in progress when Nadia's time zone change saves, when that in-progress evaluation completes, then it finishes using the time zone value it started with, and only the next cycle picks up the new value.

**FEAT-15.SPEC-006-AC-06:** Given Priya opens a milestone with a target date, when she views it, then the target date renders in her own time zone and familiar format, per FEAT-15's Key Capability for time zones and local formats.

**FEAT-15.SPEC-006-AC-07:** Given a client contact's device reports no detectable time zone, when they view any date or time in the portal, then it falls back to a neutral reference time zone with a plain indicator that the displayed time may not match their local time.

**FEAT-15.SPEC-006-AC-08:** Given Nadia views her own dashboard, when she compares a displayed due date against a reminder's day count for the same invoice, then both are computed from the same time zone value and are mutually consistent.

**FEAT-15.SPEC-006-AC-09:** Given a due date falls on different calendar dates for Nadia and Owen after each converts it to their own time zone, when both view it, then each sees their own correctly converted date with no error or warning -- this divergence is expected behavior.

**FEAT-15.SPEC-006-AC-10:** Given Dana views a date inside a support session, when the date renders, then it renders in Dana's own viewing time zone, consistent with the rule applying to every viewer, not only client-facing roles.

**FEAT-15.SPEC-006-AC-11:** Given no product surface offers a client contact a time zone setting of their own, when Owen or Priya looks for one, then none exists -- their display time zone is always inferred from their device/browser, never a stored preference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 0 (N/A -- no cross-field interaction in this rule) | 0 |
| Authorization Rules | 0 (N/A -- display rule, not access-gated) | 0 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Multi-Currency Dashboard Non-Aggregation Rule

## Overview

**Name:** Multi-Currency Dashboard Non-Aggregation Rule
**ID:** FEAT-15.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs that financial totals are always shown grouped per currency and are never converted or summed across currencies.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Invoice and Payment (read-only, as the source records behind any financial total this rule governs)

## Scope and Non-Goals

**In Scope:**
- The grouping rule: any financial total spanning more than one project or client is grouped per currency
- The prohibition rule: amounts in different currencies are never converted to a common currency, and never added together
- Establishing this spec as the cross-feature authority the Freelancer Financial Dashboard (FEAT-12) consumes when it aggregates earned/outstanding/overdue totals

**Non-Goals:**
- Computing the totals themselves (which invoices/payments count toward earned, outstanding, or overdue) -- owned entirely by Freelancer Financial Dashboard (FEAT-12); this spec only governs how those already-computed per-currency totals may be combined for display, which is: not at all
- Any currency conversion or live exchange-rate lookup capability -- excluded per this feature's Non-Goals: the assumptions-constraints.md Dependencies slice names no external capability this feature relies on, and building conversion would directly contradict this spec's own prohibition
- A single invoice spanning multiple currencies -- excluded per this feature's Non-Goals: XBR-17 ties every invoice to its project's one configured currency, so no per-invoice multi-currency scenario exists for this rule to govern
- Accounting Export's per-currency handling -- Accounting Export (FEAT-22) is a separate consumer of Invoice/Payment records with its own export-format rules; this spec only names it in the Business Rules below as one of the two features the underlying prohibition (XBR-18) governs

## Governed Entity

**Entity:** Invoice, Payment (read-only)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Invoice.currency | enum (recognized world currency code) | The currency an invoice's amount, tax, and total are denominated in (stamped from the project's configuration, FEAT-15.SPEC-008) |
| Invoice.total | number | The invoice total in its own `currency`; the figure this rule's grouping applies to |
| Payment.amount | number | A payment's amount, always equal to its invoice's full amount and therefore in that invoice's `currency` |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-12 (Freelancer Financial Dashboard) | Financial totals aggregation and dashboard/drill-down display | Whenever earned, outstanding, or overdue totals are computed or displayed across more than one client or project |
| FEAT-22 (Accounting Export) | Export file generation | Whenever exported totals span more than one currency, the export preserves the per-currency grouping rather than summing across currencies |

## Field Validation Rules

Not applicable beyond data type -- this spec places no new validation rule on Invoice.currency, Invoice.total, or Payment.amount; each is validated where it is written (Invoice.currency by FEAT-15.SPEC-003 at generation time, per FEAT-15.SPEC-008). This spec governs only how these already-valid, already-stamped values may be combined for display.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Currency grouping | Invoice.currency, Invoice.total, Payment.amount | Any total that would span invoices or payments in more than one currency is computed and shown as separate per-currency subtotals, never as one combined figure | N/A -- this is a display-composition rule, not a validation failure; there is no user-facing error, only the correct grouped presentation |

## Authorization Rules

Not applicable -- this is a computation and display-composition rule, not an access-gated action. Whether Nadia (or any role) may view a given total at all is governed by the consuming spec's own Access Matrix rules (e.g., FEAT-12's own Access and Visibility); this spec governs only how a visible total is composed once access is already granted.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| Grouped total (per currency) | Derived: for each distinct currency present among the underlying Invoice/Payment records, sum only the amounts in that currency; present each currency's sum as its own subtotal | Every time a financial total spanning more than one project or client is computed for display (FEAT-12) or export (FEAT-22) | No -- there is no setting or toggle that produces a combined, converted total; the grouped presentation is the only presentation the product defines |

## Business Rules

- XBR-18: financial totals are shown per currency; amounts in different currencies are never converted or added together -- this spec is the rule's sole definition; FEAT-12 and FEAT-22 consume it.
- This rule exists specifically because a freelancer serving clients in different countries sets a different currency per client (this feature's Primary Flows & Alternates), so any aggregate view that spans clients must, by construction, potentially span currencies.
- The non-aggregation rule is unconditional: there is no threshold, permission level, or display mode under which the product ever shows a single combined figure across currencies -- not even an approximate or clearly-labeled estimate.
- A single-currency freelancer (one who has configured every project in the same currency) sees what looks like one combined total, but this is the grouping rule producing exactly one group, not an exception to the rule.

## Edge Cases

- **A freelancer has exactly one currency across all active projects** -- The grouped presentation naturally collapses to a single subtotal, which is visually indistinguishable from "one combined total" but is still produced by the same per-currency grouping logic, not a special case.
- **A freelancer adds a client in a new currency after months of single-currency operation** -- The next time totals are computed, a second per-currency group appears; no historical total is retroactively recomputed or merged.
- **A project's currency configuration is locked (FEAT-15.SPEC-004) but an older, pre-lock invoice exists in a different currency due to a since-corrected configuration error** -- Not applicable: FEAT-15.SPEC-004 locks currency at the first invoice, so no project ever produces invoices in more than one currency; this scenario cannot occur under the product's own rules, and is noted here to confirm no such edge case exists.
- **A total includes both a positive invoice amount and a negative refund/reversal figure in the same currency** -- Both net within their shared currency's subtotal normally; a refund or reversal in one currency never offsets or nets against a total in a different currency.
- **The Accounting Export (FEAT-22) is generated for a freelancer with multiple currencies** -- The export preserves the same per-currency grouping this spec defines, rather than summing across currencies into the export's file.

## Acceptance Criteria

**FEAT-15.SPEC-007-AC-01:** Given Nadia has clients billed in USD and clients billed in EUR, when she opens her financial dashboard, then earned, outstanding, and overdue totals are shown as separate USD and EUR subtotals, never combined into one figure.

**FEAT-15.SPEC-007-AC-02:** Given Nadia's dashboard totals span two currencies, when she looks for a single combined "total across all currencies" figure, then none is shown anywhere on the dashboard.

**FEAT-15.SPEC-007-AC-03:** Given Nadia has configured every active project in the same currency, when she opens her dashboard, then she sees exactly one subtotal group, produced by the same per-currency grouping logic as a multi-currency account.

**FEAT-15.SPEC-007-AC-04:** Given Nadia adds her first client in a new currency, when totals are next computed, then a new per-currency group appears alongside her existing group(s), with no retroactive recomputation of prior totals.

**FEAT-15.SPEC-007-AC-05:** Given Nadia records a refund in EUR against an EUR invoice, when the dashboard recomputes her EUR subtotal, then the refund nets within the EUR group only, never against a USD or other-currency group.

**FEAT-15.SPEC-007-AC-06:** Given Nadia generates an accounting export (FEAT-22) across multiple currencies, when the export file is produced, then it preserves the same per-currency grouping the dashboard uses, with no converted or summed cross-currency figure.

**FEAT-15.SPEC-007-AC-07:** Given Nadia drills down from the dashboard into a specific client's totals, when the client's projects span more than one currency, then that drill-down also shows per-currency subtotals, consistent with the dashboard-level rule.

**FEAT-15.SPEC-007-AC-08:** Given the non-aggregation rule is unconditional, when any future display mode or permission level is considered by a downstream builder, then no exception path exists in this spec that would permit a combined cross-currency figure under any condition.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- no new validation beyond data type) | 0 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 0 (N/A -- computation/display rule, not access-gated) | 0 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Invoice Currency & Tax Line Application

## Overview

**Name:** Invoice Currency & Tax Line Application
**ID:** FEAT-15.SPEC-008
**Type:** Automation
**Purpose:** At invoice generation, stamps the project's configured currency and tax label/rate onto the new invoice, computes the tax amount and total, and flags a mismatch if the invoice's currency would not match the project's current configuration.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling

## Scope and Non-Goals

**In Scope:**
- Reading the project's configured currency, tax label, and tax rate at the moment an invoice is generated
- Applying FEAT-15.SPEC-003's validation as a precondition -- blocking generation on an invalid or missing configuration
- Stamping currency, tax label, and tax rate onto the new invoice, and computing the tax amount and total from the subtotal
- Detecting and flagging a currency mismatch between the invoice being generated and the project's currently configured currency

**Non-Goals:**
- Deciding when an invoice is generated at all (deposit, milestone approval, completion, ad hoc, or correction) -- owned entirely by Invoice Generation & Sending (FEAT-09); this automation only reacts to the generation event and applies currency/tax to whatever invoice FEAT-09 is creating
- Computing the invoice subtotal itself (the amount before tax) -- owned entirely by FEAT-09 and its triggering source (Payment Schedule, Milestone, or ad hoc entry); this automation reads the subtotal as an input and applies tax to it
- Locking the project's currency/tax after the first invoice -- owned entirely by FEAT-15.SPEC-004; this automation triggers that lock's evaluation by being the point at which "first invoice sent" becomes possible, but the lock rule itself lives there
- Automatic, jurisdiction-specific tax calculation -- excluded per scope-boundaries.md (SC-16): this automation applies the freelancer-configured label and rate exactly as stored; it computes no rate of its own

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invoice generated (deposit) | FEAT-09 (Invoice Generation & Sending), fired from acceptance per XBR-01 | Fires whenever FEAT-09 creates a new Invoice record from a proposal acceptance with a deposit in the payment schedule | Project reference, invoice subtotal, triggering event type |
| Invoice generated (milestone approval) | FEAT-09, fired from milestone approval per XBR-02 | Fires whenever FEAT-09 creates a new Invoice record from an approved milestone | Project reference, invoice subtotal, triggering event type |
| Invoice generated (project completion) | FEAT-09, fired from marking a project complete per XBR-03 | Fires whenever FEAT-09 creates a new Invoice record from project completion | Project reference, invoice subtotal, triggering event type |
| Invoice generated (ad hoc) | FEAT-09 | Fires whenever Nadia issues an ad hoc invoice outside the schedule | Project reference, invoice subtotal, triggering event type |
| Invoice generated (correction) | FEAT-09, fired from a credit note or corrected invoice | Fires whenever FEAT-09 creates a new Invoice record as a correction to a prior invoice | Project reference, invoice subtotal, triggering event type, reference to the invoice being corrected |

## Processing Logic

1. Receive the new invoice's project reference and subtotal from the triggering generation event (FEAT-09).
2. Read the project's currently configured `currency`, `tax_label`, and `tax_rate` (or "no tax line").
3. Apply FEAT-15.SPEC-003's validation to the project's current configuration as a precondition: if the configuration is invalid or was never set, halt processing and signal the blocking outcome back to FEAT-09 before any invoice record is finalized.
4. If the configuration is valid, compare the invoice's expected currency (the one the triggering event assumed, e.g., what a proposal or payment schedule was denominated in) against the project's currently configured currency.
5. If the two currencies do not match, flag the discrepancy (do not silently substitute either value) and proceed to the mismatch-flagged outcome rather than the success outcome.
6. If the two currencies match (the normal case), stamp the invoice's `currency`, `tax_label`, and `tax_rate` fields with the project's configured values.
7. Compute the tax amount: the subtotal (`amount`) multiplied by `tax_rate` (or zero, when "no tax line" applies). The tax amount itself is a computed figure used to derive the total and to display the tax line on the invoice; it is not a separately stored Invoice field beyond the `tax_label`/`tax_rate` pair that determines it.
8. Compute the invoice total: `amount` plus the computed tax amount.
9. Write the computed `total` onto the invoice record alongside the stamped `currency`, `tax_label`, and `tax_rate` fields.
10. Signal completion back to FEAT-09, which proceeds with the remainder of invoice generation (numbering, due date, sending).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Applied successfully | Project has a valid, complete currency/tax configuration and the invoice's expected currency matches it | Invoice's `currency`, `tax_label`, `tax_rate`, and `total` are set (tax amount is computed as part of deriving `total` and displayed from `amount` and `tax_rate`, not stored separately) | None directly -- the invoice simply carries correct values when Nadia or Owen next views it (FEAT-09.SPEC-002) | FEAT-09 (proceeds with generation), FEAT-09.SPEC-002 (displays the result) |
| Generation blocked -- no valid configuration | Project's currency/tax configuration is missing or fails FEAT-15.SPEC-003's validation | No invoice record is finalized | Nadia sees the empty-configuration prompt on FEAT-15.SPEC-001, per that screen's Empty state, directing her to complete configuration before generation can proceed | FEAT-15.SPEC-001 (empty state), FEAT-09 (generation halted) |
| Currency mismatch flagged | The invoice's expected currency does not match the project's currently configured currency at the moment of generation (e.g., a race between a configuration change and a concurrent generation) | Invoice is created with the `invoice_currency_mismatch_flagged` signal recorded against it rather than silently issuing a mismatched invoice; the invoice is not automatically sent | Nadia sees a flagged-invoice indicator on the invoice before it can be sent, prompting her to review and confirm the correct currency before sending | FEAT-09 (generation proceeds to a held state rather than auto-send), FEAT-09.SPEC-002 (shows the flagged indicator) |
| Automation failure (processing error unrelated to configuration validity) | Reading the project's configuration or computing the tax amount/total fails for a reason other than an invalid configuration | No invoice record is finalized | Nadia sees a generation failure notice from FEAT-09 with a retry option; the failure is not attributed to her configuration, since the configuration was never evaluated to invalid | FEAT-09 (generation retried or surfaced as a failure) |

## Data Model

**Reads:** Project -- `currency`, `tax_label`, `tax_rate`. Invoice (the one being generated) -- the subtotal and expected currency supplied by the triggering event from FEAT-09.
**Creates:** None -- the Invoice record itself is created by FEAT-09; this automation supplies field values onto it during that creation.
**Updates:** Invoice -- `currency`, `tax_label`, `tax_rate`, `total` (this is this feature's one Update to Invoice, per the dependency map's Invoice lifecycle line: "Updated by ... FEAT-15 (currency and tax fields at generation)"). The tax amount is computed from `amount` and `tax_rate` for the total and for the invoice's displayed tax line; it is not a separately stored field.
**Deletes:** None.

## Business Rules

- XBR-17: proposal prices are in the project's already-configured currency, and this automation is the point at which that same configuration is applied to every invoice for the project.
- This automation enforces FEAT-15.SPEC-003 as a precondition -- it never stamps a configuration that would fail validation, and it never invents a default when configuration is missing.
- The tax amount is always computed as subtotal times tax rate, with "no tax line" treated as a tax rate of zero for this computation only (the invoice's `tax_label` remains genuinely absent, not "0%", to preserve the distinction FEAT-15.SPEC-003 establishes between a zero-rated line and no tax line at all).
- A currency mismatch is never resolved automatically by picking one value over the other -- XBR-18's non-conversion rule means there is no computation that could reconcile two different currencies, so the only safe response is to flag and hold, never to guess.
- This automation is also the trigger point that makes FEAT-15.SPEC-004's lock possible: it is the successful "applied" outcome, once the resulting invoice reaches Sent, that fires the first-invoice lock for the project.

## Edge Cases

- **The project's configuration changes between when the triggering event (e.g., milestone approval) captured the expected currency and when this automation actually runs** -- This is exactly the mismatch condition: flagged, not silently applied, per the Outcome Definitions above.
- **The project has "no tax line" configured and the subtotal is a whole number** -- Tax amount computes to zero, total equals the subtotal exactly, and the invoice carries no `tax_label` or `tax_rate` value at all (not a zero value).
- **The subtotal is zero (e.g., a fully-discounted ad hoc invoice)** -- Tax amount computes to zero regardless of the configured tax rate (zero times any rate is zero); the total is zero. No error is raised for a zero subtotal by this automation.
- **Two invoice-generation triggers fire for the same project at effectively the same time (e.g., a milestone approval and an ad hoc invoice issued moments apart)** -- Each generation event runs this automation independently against its own invoice record; both read the project's configuration at their own respective moments, so each is internally consistent even if the two invoices could theoretically read different configuration snapshots (which itself could only happen if a change landed on the project between the two runs, in which case the later one would be evaluated against the newer configuration).
- **This automation's run for one invoice is still in flight when a second generation trigger fires for the same project** -- The two runs process independently and do not queue behind each other; each reads the project's configuration at its own start time and writes only to its own invoice record, so neither run's completion depends on the other's.
- **A correction (credit note) is generated for an already-locked project** -- The project's currency/tax remains as locked by FEAT-15.SPEC-004; this automation applies that same locked configuration to the credit note exactly as it would to any other invoice on that project, since a locked project always has a valid, unchanging configuration to read.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-003 (Currency & Tax Validation Rules) | References (inbound) | Supplies the precondition check this automation applies before stamping any invoice |
| FEAT-15.SPEC-001 (Project Currency & Tax Configuration) | Affects (outbound) | The empty-configuration and validation-failure outcomes surface on that screen |
| FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice) | Triggers (outbound) | A successful application whose resulting invoice reaches Sent is the event that fires the first-invoice lock |
| FEAT-09 (Invoice Generation & Sending) | Triggered by (inbound) | Every invoice-generation trigger (deposit, milestone approval, completion, ad hoc, correction) invokes this automation |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Supplies the stamped currency/tax fields, computed tax amount, and total that FEAT-09's invoice record and detail screen (FEAT-09.SPEC-002) display |

## Analytics and Success Signals

- **currency_tax_applied** (project reference, invoice reference, currency, tax treatment applied) -- supports success-metrics.md: "Invoice Currency and Tax Correctness"
- **invoice_currency_mismatch_flagged** (project reference, invoice reference, expected currency, project's currently configured currency) -- supports success-metrics.md: "Invoice Currency and Tax Correctness" (a flagged mismatch is exactly the failure mode this metric measures the absence of)
- **currency_tax_application_blocked** (project reference, reason: missing configuration / invalid configuration) -- supports success-metrics.md: "Invoice Currency and Tax Correctness" (a blocked generation is the automation successfully preventing an incorrect invoice from ever being issued)

## Acceptance Criteria

**FEAT-15.SPEC-008-AC-01:** Given a project has a valid currency and tax configuration, when its next milestone approval triggers invoice generation, then the new invoice is stamped with that currency, tax label, and tax rate, and its tax amount and total are computed from the subtotal.

**FEAT-15.SPEC-008-AC-02:** Given a project has "no tax line" configured, when an invoice is generated for it, then the tax amount is zero, the total equals the subtotal, and no `tax_label` or `tax_rate` value is set.

**FEAT-15.SPEC-008-AC-03:** Given a project has never had currency/tax configured, when a generation trigger fires for it, then generation is blocked and Nadia sees the empty-configuration prompt on FEAT-15.SPEC-001.

**FEAT-15.SPEC-008-AC-04:** Given a project's configuration changes after a triggering event captured its expected currency but before this automation runs, when the mismatch is detected, then the resulting invoice is created with `invoice_currency_mismatch_flagged` set and is not automatically sent.

**FEAT-15.SPEC-008-AC-05:** Given a subtotal of zero on an ad hoc invoice, when this automation computes tax and total, then both are zero and no error is raised.

**FEAT-15.SPEC-008-AC-06:** Given two generation triggers fire for the same project at effectively the same time, when both run this automation, then each processes its own invoice independently and neither is blocked by the other.

**FEAT-15.SPEC-008-AC-07:** Given this automation's run for one invoice is still in flight, when a second generation trigger fires for the same project, then the second run proceeds independently rather than queuing behind the first.

**FEAT-15.SPEC-008-AC-08:** Given the accepted proposal's deposit trigger fires per XBR-01, when this automation applies currency and tax to the resulting deposit invoice, then the applied values match the project's configuration at that moment.

**FEAT-15.SPEC-008-AC-09:** Given a project's currency/tax is already locked, when a credit note is generated for it, then this automation applies the same locked configuration to the credit note exactly as it would to any other invoice on that project.

**FEAT-15.SPEC-008-AC-10:** Given this automation successfully applies currency and tax to an invoice, when the resulting invoice is later sent as the project's first invoice, then that Sent event is the trigger FEAT-15.SPEC-004 uses to lock the project's configuration.

**FEAT-15.SPEC-008-AC-11:** Given a processing error unrelated to configuration validity occurs (e.g., the configuration read itself fails), when this automation cannot complete, then Nadia sees a generation failure notice from FEAT-09 with a retry option, distinct from the empty-configuration message.

**FEAT-15.SPEC-008-AC-12:** Given this automation successfully applies currency and tax, when the application completes, then the currency_tax_applied event is emitted with the project and invoice references and the values applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (deposit, milestone approval, completion, ad hoc, correction) | 5 |
| Outcome Paths | 4 (applied successfully, blocked -- no configuration, mismatch flagged, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
