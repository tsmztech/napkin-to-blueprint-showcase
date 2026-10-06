# Feature Specification: Currency & Tax Handling

**Blueprint feature:** FEAT-15
**Priority tier:** Core
**Build order:** 002 of 33
**Depends on:** —
**Blueprint source:** `docs/blueprint/specifications/FEAT-15-currency-tax-handling/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Project Currency & Tax Configuration (Priority: P1)

Nadia sets a project's billing currency and tax label/rate during billing setup, and views the locked, read-only version of that configuration once the project's first invoice has been sent; Dana views the same configuration read-only inside a logged support session.

**Acceptance Scenarios:**

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

### User Story 2 - Freelancer Time Zone Setting (Priority: P1)

Nadia views and changes her own time zone, which grounds every reminder day-count and local-time display across the product.

**Acceptance Scenarios:**

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

### User Story 3 - Currency & Tax Validation Rules (Priority: P1)

Governs which currencies are acceptable and what shape a tax line may take on a project, and blocks invoice generation with a specific, correctable message on an invalid configuration.

**Acceptance Scenarios:**

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

### User Story 4 - Currency & Tax Lock After First Invoice (Priority: P1)

Locks a project's currency and its paired tax line the moment its first invoice is sent, and blocks any later change attempt with an explanation.

**Acceptance Scenarios:**

**FEAT-15.SPEC-004-AC-01:** Given a project has no invoice that has ever reached Sent, when Nadia opens its currency/tax configuration, then the fields render editable and lock_state is Editable.

**FEAT-15.SPEC-004-AC-02:** Given a project's first invoice is sent by FEAT-09, when that event lands, then the project's lock_state transitions to Locked immediately.

**FEAT-15.SPEC-004-AC-03:** Given a project's lock_state is Locked, when Nadia opens its currency/tax configuration, then no editable field is rendered and the explanation "currency and tax are fixed after your first invoice" is shown.

**FEAT-15.SPEC-004-AC-04:** Given a project's lock_state is Locked, when any out-of-band attempt tries to change its currency, then it is refused with "Currency is fixed once billing has begun for this project."

**FEAT-15.SPEC-004-AC-05:** Given a project has a Draft invoice that has never reached Sent, when Nadia opens its currency/tax configuration, then the fields remain editable -- the draft alone does not trigger the lock.

**FEAT-15.SPEC-004-AC-06:** Given a project's would-be first invoice is voided before reaching Sent, when Nadia later opens its currency/tax configuration, then the fields remain editable and no lock has fired.

**FEAT-15.SPEC-004-AC-07:** Given Nadia's save attempt reaches the server after the project's first invoice has already been sent in the interim, when the lock check runs, then the save is refused even though the screen appeared editable when it was opened.

**FEAT-15.SPEC-004-AC-08:** Given a project's currency/tax is locked, when Dana views it inside a support session, then she sees the identical locked, read-only display Nadia would see -- no override is available to her.

**FEAT-15.SPEC-004-AC-09:** Given a project's currency/tax is already locked, when a credit note or a corrected invoice is later issued for that project, then no re-evaluation of the lock occurs and the project remains Locked exactly as before.

### User Story 5 - Currency & Tax Configuration Access Rules (Priority: P1)

Governs who may view or change a project's currency/tax configuration -- Nadia Full, Dana read-only in a logged session, Owen and Priya no direct access to the configuration surface at all.

**Acceptance Scenarios:**

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

### User Story 6 - Time Zone & Local Date/Time Display Rule (Priority: P1)

Governs how dates, due dates, and times are rendered in each viewer's own time zone and familiar format, and establishes the freelancer's time zone as the basis for reminder day counts.

**Acceptance Scenarios:**

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

### User Story 7 - Multi-Currency Dashboard Non-Aggregation Rule (Priority: P1)

Governs that financial totals are always shown grouped per currency and are never converted or summed across currencies.

**Acceptance Scenarios:**

**FEAT-15.SPEC-007-AC-01:** Given Nadia has clients billed in USD and clients billed in EUR, when she opens her financial dashboard, then earned, outstanding, and overdue totals are shown as separate USD and EUR subtotals, never combined into one figure.

**FEAT-15.SPEC-007-AC-02:** Given Nadia's dashboard totals span two currencies, when she looks for a single combined "total across all currencies" figure, then none is shown anywhere on the dashboard.

**FEAT-15.SPEC-007-AC-03:** Given Nadia has configured every active project in the same currency, when she opens her dashboard, then she sees exactly one subtotal group, produced by the same per-currency grouping logic as a multi-currency account.

**FEAT-15.SPEC-007-AC-04:** Given Nadia adds her first client in a new currency, when totals are next computed, then a new per-currency group appears alongside her existing group(s), with no retroactive recomputation of prior totals.

**FEAT-15.SPEC-007-AC-05:** Given Nadia records a refund in EUR against an EUR invoice, when the dashboard recomputes her EUR subtotal, then the refund nets within the EUR group only, never against a USD or other-currency group.

**FEAT-15.SPEC-007-AC-06:** Given Nadia generates an accounting export (FEAT-22) across multiple currencies, when the export file is produced, then it preserves the same per-currency grouping the dashboard uses, with no converted or summed cross-currency figure.

**FEAT-15.SPEC-007-AC-07:** Given Nadia drills down from the dashboard into a specific client's totals, when the client's projects span more than one currency, then that drill-down also shows per-currency subtotals, consistent with the dashboard-level rule.

**FEAT-15.SPEC-007-AC-08:** Given the non-aggregation rule is unconditional, when any future display mode or permission level is considered by a downstream builder, then no exception path exists in this spec that would permit a combined cross-currency figure under any condition.

### User Story 8 - Invoice Currency & Tax Line Application (Priority: P1)

At invoice generation, stamps the project's configured currency and tax label/rate onto the new invoice, computes the tax amount and total, and flags a mismatch if the invoice's currency would not match the project's current configuration.

**Acceptance Scenarios:**

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

### Edge Cases

- **FEAT-15.SPEC-001 (Project Currency & Tax Configuration):** Unsaved currency and tax edits trigger a Discard / Keep Editing confirmation and a double tap on Save is ignored. If the project's first invoice is sent while the screen is open unlocked, the screen is rejected-with-refresh and the in-progress edit is not saved; a locked project renders no editable fields at all. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-001-project-currency-tax-configuration.md` (section: Edge Cases)
- **FEAT-15.SPEC-002 (Freelancer Time Zone Setting):** Unsaved selections trigger the discard confirmation and double saves are ignored. Concurrent changes from two sessions are last-write-wins, and saving the already-selected time zone proceeds normally with the same success feedback. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-002-freelancer-time-zone-setting.md` (section: Edge Cases)
- **FEAT-15.SPEC-003 (Currency & Tax Validation Rules):** Tax rate boundaries are inclusive (0 and 100 pass, 0 is a defined zero-rated treatment distinct from no tax line) while -0.01 and 100.01 fail with the 0-to-100 message; a tax label passes at 60 characters and fails at 61. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-003-currency-tax-validation-rules.md` (section: Edge Cases)
- **FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice):** The lock check re-evaluates at the moment of the save attempt, so sessions open on an unlocked project are rejected-with-refresh once the first invoice is sent. Only an invoice reaching Sent (not Draft, Generated, voided or failed sends) locks the project. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-004-currency-tax-lock-after-first-invoice.md` (section: Edge Cases)
- **FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules):** The portal-home redirect applies to client contacts however they obtained a link, and an operator's support access ends the moment the session closes (FEAT-31). A concurrent read-only support session never conflicts with the freelancer's edits, and roles beyond the four in the Access Matrix are out of scope. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-005-currency-tax-configuration-access-rules.md` (section: Edge Cases)
- **FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule):** Each viewer sees dates in their own time zone independently, so the same due date can show different calendar dates. A reminder evaluation in flight finishes with the time zone it read at its start, a viewer with no detectable time zone falls back to UTC for that session with a plain indicator, and day boundaries are applied per viewer. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-006-time-zone-local-date-time-display-rule.md` (section: Edge Cases)
- **FEAT-15.SPEC-007 (Multi-Currency Dashboard Non-Aggregation Rule):** Totals are always grouped per currency: a single-currency freelancer sees one subtotal, a newly added currency creates a new group without retroactively merging history, and refunds or reversals net only within their own currency. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-007-multi-currency-dashboard-non-aggregation-rule.md` (section: Edge Cases)
- **FEAT-15.SPEC-008 (Invoice Currency & Tax Line Application):** A configuration change between the triggering event and the automation run is flagged as a mismatch rather than silently applied. A no-tax-line configuration or a zero subtotal yields zero tax with no tax_label or tax_rate stored, and concurrent generation triggers for one project each run independently. Source: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-008-invoice-currency-tax-line-application.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-15.SPEC-001** (Project Currency & Tax Configuration) as specified: Nadia sets a project's billing currency and tax label/rate during billing setup, and views the locked, read-only version of that configuration once the project's first invoice has been sent; Dana views the same configuration read-only inside a logged support session. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-001-project-currency-tax-configuration.md`
- **FR-002**: The system MUST implement **FEAT-15.SPEC-002** (Freelancer Time Zone Setting) as specified: Nadia views and changes her own time zone, which grounds every reminder day-count and local-time display across the product. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-002-freelancer-time-zone-setting.md`
- **FR-003**: The system MUST implement **FEAT-15.SPEC-003** (Currency & Tax Validation Rules) as specified: Governs which currencies are acceptable and what shape a tax line may take on a project, and blocks invoice generation with a specific, correctable message on an invalid configuration. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-003-currency-tax-validation-rules.md`
- **FR-004**: The system MUST implement **FEAT-15.SPEC-004** (Currency & Tax Lock After First Invoice) as specified: Locks a project's currency and its paired tax line the moment its first invoice is sent, and blocks any later change attempt with an explanation. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-004-currency-tax-lock-after-first-invoice.md`
- **FR-005**: The system MUST implement **FEAT-15.SPEC-005** (Currency & Tax Configuration Access Rules) as specified: Governs who may view or change a project's currency/tax configuration -- Nadia Full, Dana read-only in a logged session, Owen and Priya no direct access to the configuration surface at all. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-005-currency-tax-configuration-access-rules.md`
- **FR-006**: The system MUST implement **FEAT-15.SPEC-006** (Time Zone & Local Date/Time Display Rule) as specified: Governs how dates, due dates, and times are rendered in each viewer's own time zone and familiar format, and establishes the freelancer's time zone as the basis for reminder day counts. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-006-time-zone-local-date-time-display-rule.md`
- **FR-007**: The system MUST implement **FEAT-15.SPEC-007** (Multi-Currency Dashboard Non-Aggregation Rule) as specified: Governs that financial totals are always shown grouped per currency and are never converted or summed across currencies. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-007-multi-currency-dashboard-non-aggregation-rule.md`
- **FR-008**: The system MUST implement **FEAT-15.SPEC-008** (Invoice Currency & Tax Line Application) as specified: At invoice generation, stamps the project's configured currency and tax label/rate onto the new invoice, computes the tax amount and total, and flags a mismatch if the invoice's currency would not match the project's current configuration. Full spec: `docs/blueprint/specifications/FEAT-15-currency-tax-handling/FEAT-15.SPEC-008-invoice-currency-tax-line-application.md`

### Key Entities

- Invoice (read/update — currency and tax fields)
- Project (read — client's stated currency)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: 100% of invoices display the project's configured currency and tax line correctly, across every currency and tax configuration in active use (metric: Invoice Currency and Tax Correctness). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Currency, tax treatment and time zone configuration changes, and any invoice currency mismatch, are each observable as distinct signals (currency_set, tax_treatment_set, invoice_currency_mismatch_flagged, timezone_set). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-19**: English-only interface at launch, with locale-aware money, dates and time zones never hard-coded. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: Invoices carry the content commonly required of a valid invoice, including a tax line. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-25**: Financial and evidentiary records are timestamped at creation and never silently altered afterward. Full register: `docs/blueprint/features/assumptions-constraints.md`
