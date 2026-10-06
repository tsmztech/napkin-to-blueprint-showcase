---
document_type: feature-overview
feature_number: FEAT-15
feature_name: Currency & Tax Handling
feature_slug: currency-tax-handling
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 2
automation_count: 1
logic_rule_count: 5
integration_count: 0
notification_count: 0
---

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
