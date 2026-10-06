# Feature Specification: Contextual Help & Guidance

**Blueprint feature:** FEAT-30
**Priority tier:** Nice-to-Have
**Build order:** 033 of 33
**Depends on:** FEAT-05, FEAT-08, FEAT-20
**Blueprint source:** `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Contextual Help Tooltip (Priority: P3)

An inline, on-demand explanation for an unfamiliar control, overlaid on a host screen starting at the user's first encounter with the control and offered on every later encounter until permanently dismissed, with a permanent-dismiss action.

**Acceptance Scenarios:**

**FEAT-30.SPEC-001-AC-01:** Given Nadia is on a first-run setup screen (FEAT-20) and encounters an unfamiliar control for the first time, when she taps the help affordance beside it, then a popover opens showing a brief explanation of that control.

**FEAT-30.SPEC-001-AC-02:** Given Owen is viewing the milestone approval screen (FEAT-08) for the first time, when he taps the help affordance beside the Approve control, then a popover opens explaining what approving does, and this emits a help_tip_shown event with host_feature FEAT-08.

**FEAT-30.SPEC-001-AC-03:** Given Priya (Reviewer) is on the same milestone view Owen sees, when she looks for a help affordance beside an Approve control, then none is shown, because her role never has an Approve control rendered to it in the first place (XBR-08).

**FEAT-30.SPEC-001-AC-04:** Given Nadia has an open help popover, when she taps "Got it," then the popover closes, no dismissal is recorded, and the affordance is offered again at her next encounter of the control and at every later encounter until she chooses "Don't show this again."

**FEAT-30.SPEC-001-AC-05:** Given Owen has an open help popover, when he taps "Don't show this again," then the popover closes and FEAT-30.SPEC-004 records a permanent dismissal for that tip and Owen.

**FEAT-30.SPEC-001-AC-06:** Given Nadia previously dismissed a tip permanently, when she encounters the same control again on any screen, then no affordance is shown for that tip.

**FEAT-30.SPEC-001-AC-07:** Given Dana is in a read-only support session viewing Nadia's dashboard, when the underlying screen renders, then no help affordances or popovers appear anywhere on it.

**FEAT-30.SPEC-001-AC-08:** Given a visitor is not signed in, when they attempt to reach any host screen this overlay would appear on, then they cannot reach that screen at all, and this overlay never renders.

**FEAT-30.SPEC-001-AC-09:** Given Priya has an open help popover, when she taps outside the popover, then it closes the same as tapping "Got it," and the tip remains available for a later encounter.

**FEAT-30.SPEC-001-AC-10:** Given Owen has one help popover open, when he taps a different control's help affordance, then the first popover closes and the second one opens.

**FEAT-30.SPEC-001-AC-11:** Given Nadia loses connectivity while a help popover is open, when she continues reading it, then the already-rendered explanation stays visible and no error appears.

**FEAT-30.SPEC-001-AC-12:** Given Nadia taps "Don't show this again" and the dismissal write fails for a reason other than lost connectivity, so it is not retried, when she encounters the same control again later, then the affordance may still appear, and no error was ever shown to her for the earlier failed attempt.

**FEAT-30.SPEC-001-AC-13:** Given Priya is promoted from Reviewer to Primary contact by FEAT-18 mid-session, when she next encounters the Approve control, then a help affordance for it is offered to her for the first time, since it is now eligible under her new role.

**FEAT-30.SPEC-001-AC-14:** Given Nadia dismisses the same tip from two open sessions at effectively the same time, when both dismissal writes complete, then the tip is recorded as dismissed exactly once in effect, with no error or conflict shown in either session.

**FEAT-30.SPEC-001-AC-15:** Given Owen taps "Don't show this again" while his device has no connectivity, when the popover closes, then the tip is not offered again on that device, no error is shown, and once connectivity returns FEAT-30.SPEC-004 completes the dismissal automatically so the tip stays suppressed on every future render.

### User Story 2 - Freelancer Help Reference (Priority: P3)

A short, browsable help reference covering the freelancer dashboard, reachable from any dashboard screen, so Nadia needs no external documentation.

**Acceptance Scenarios:**

**FEAT-30.SPEC-002-AC-01:** Given Nadia is on any dashboard screen, when she taps the Help entry point, then the Freelancer Help Reference opens showing the full topic list, collapsed.

**FEAT-30.SPEC-002-AC-02:** Given Nadia is on the Freelancer Help Reference, when she taps a collapsed topic, then it expands to show its explanation inline.

**FEAT-30.SPEC-002-AC-03:** Given Nadia has expanded a topic that corresponds to an inline tip, when she taps "Don't show this again," then FEAT-30.SPEC-004 records the dismissal and the corresponding inline tip on the dashboard no longer appears on future encounters.

**FEAT-30.SPEC-002-AC-04:** Given Nadia expands a topic with no corresponding inline tip, when she looks for a dismiss action, then none is shown, and the topic always remains in the list.

**FEAT-30.SPEC-002-AC-05:** Given Nadia is on the empty (zero-state) dashboard, when the dashboard renders, then a zero-state prompt directs her toward this reference alongside the prompt to draft a first proposal (FEAT-02).

**FEAT-30.SPEC-002-AC-06:** Given Owen or Priya is signed into their client portal, when they look for any way to reach the Freelancer Help Reference, then no path exists -- this screen is not part of the client portal.

**FEAT-30.SPEC-002-AC-07:** Given Dana is in a read-only support session viewing Nadia's account, when she looks for a help entry point, then none is shown to her.

**FEAT-30.SPEC-002-AC-08:** Given Nadia loses connectivity while this reference is open, when she continues browsing topics, then every topic's explanation remains visible and expandable with no error.

**FEAT-30.SPEC-002-AC-09:** Given Nadia taps "Don't show this again" while offline, when connectivity is lost at that moment, then the dismissal is held on her device, the topic shows the "Inline tips for this topic are turned off" note meanwhile, and FEAT-30.SPEC-004 retries the write automatically once connectivity returns, with no error shown to her.

**FEAT-30.SPEC-002-AC-11:** Given Nadia has already permanently dismissed the tip for a topic (here or inline), when she opens the Freelancer Help Reference and expands that topic, then the topic and its full explanation are shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

**FEAT-30.SPEC-002-AC-10:** Given Nadia taps the close/back control, when the screen closes, then she returns to the exact dashboard screen she opened this reference from.

### User Story 3 - Client Portal Help Reference (Priority: P3)

A short, browsable help reference covering the client-facing portal, scoped to the viewing contact's role, reachable from any portal screen, so a client contact needs no external documentation.

**Acceptance Scenarios:**

**FEAT-30.SPEC-003-AC-01:** Given Owen is on the portal home, when he taps the Help entry point, then the Client Portal Help Reference opens showing his Primary-scoped topic groups, collapsed.

**FEAT-30.SPEC-003-AC-02:** Given Priya is on a portal screen, when she taps the Help entry point, then the reference opens showing only her Reviewer-scoped groups, with no Proposals & Acceptance, Milestone Approval, or Invoicing & Payments group present.

**FEAT-30.SPEC-003-AC-03:** Given Owen has expanded the topic explaining the Approve control, when he taps "Don't show this again," then FEAT-30.SPEC-004 records the dismissal and the corresponding inline tip no longer appears to him on future encounters.

**FEAT-30.SPEC-003-AC-04:** Given Priya expands a topic with no corresponding inline tip, when she looks for a dismiss action, then none is shown, and the topic always remains in her list.

**FEAT-30.SPEC-003-AC-05:** Given Nadia is signed into her freelancer dashboard, when she looks for any way to reach the Client Portal Help Reference, then no path exists -- she uses FEAT-30.SPEC-002 instead.

**FEAT-30.SPEC-003-AC-06:** Given a visitor is not signed in, when they attempt to reach any client-portal screen, then they are redirected to the magic-link sign-in request page, and this reference is unreachable until they sign in.

**FEAT-30.SPEC-003-AC-07:** Given a contact's magic link has expired, when they try to open the portal, then they see the expired-link page with a fresh-link option, and this reference is unreachable until a new sign-in succeeds.

**FEAT-30.SPEC-003-AC-08:** Given Nadia promotes Priya to Primary contact via FEAT-18, when she next opens this reference, then the Primary-scoped groups appear for her for the first time.

**FEAT-30.SPEC-003-AC-09:** Given Owen loses connectivity while this reference is open, when he continues browsing topics, then every topic's explanation remains visible and expandable with no error.

**FEAT-30.SPEC-003-AC-10:** Given Owen taps "Don't show this again" while offline, when connectivity is lost at that moment, then the dismissal is held on his device, the topic shows the "Inline tips for this topic are turned off" note meanwhile, and FEAT-30.SPEC-004 retries the write automatically once connectivity returns, with no error shown.

**FEAT-30.SPEC-003-AC-11:** Given Priya taps the close/back control, when the screen closes, then she returns to the exact portal screen she opened this reference from.

**FEAT-30.SPEC-003-AC-12:** Given Owen has already permanently dismissed the tip for a topic (here or inline), when he opens the Client Portal Help Reference and expands that topic, then the topic and its full explanation are shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

### User Story 4 - Help Tip Dismissal Recording (Priority: P3)

Owns every write to a user's Help-Tip Dismissal State: the default "not dismissed" record created the first time a tip becomes eligible to render, and the permanent flip to "dismissed" when the user chooses "Don't show this again."

**Acceptance Scenarios:**

**FEAT-30.SPEC-004-AC-01:** Given Nadia encounters a tip for the first time and no Help-Tip Dismissal State record exists yet for it, when FEAT-30.SPEC-005 evaluates it for render, then this automation creates a record with dismissed = false and the tip renders as eligible.

**FEAT-30.SPEC-004-AC-02:** Given a Help-Tip Dismissal State record already exists for a tip and user with dismissed = false, when the same tip is evaluated for render again, then this automation takes no action (no duplicate record is created).

**FEAT-30.SPEC-004-AC-03:** Given Owen taps "Don't show this again" on an inline tip explaining the Approve control (FEAT-08), when this automation processes the dismissal, then it sets dismissed = true and dismissed_at to the current time on Owen's Client Contact record, and emits help_tip_dismissed with host_feature FEAT-08.

**FEAT-30.SPEC-004-AC-04:** Given Priya taps "Don't show this again" on a reference-only topic entry (FEAT-30.SPEC-003) with no corresponding host_feature overlay, when this automation processes the dismissal, then it records the dismissal and emits help_tip_dismissed with the N/A citation, since no overlaid metric applies.

**FEAT-30.SPEC-004-AC-05:** Given a tip is already recorded as dismissed for Nadia, when a second dismissal request for the same tip arrives, then this automation makes no further change and produces no error.

**FEAT-30.SPEC-004-AC-06:** Given Owen taps "Don't show this again" and the write fails for a reason other than lost connectivity, when he next encounters the same control, then the tip may still appear, no retry was made, and no error was shown to him at the time of the failed attempt.

**FEAT-30.SPEC-004-AC-10:** Given Nadia taps "Don't show this again" (from FEAT-30.SPEC-001 or FEAT-30.SPEC-002) while her device has no connectivity, when connectivity is restored, then the held dismissal is retried automatically, dismissed = true and dismissed_at are set, and at no point was an error shown to her; while it was held, the tip stayed suppressed on that device.

**FEAT-30.SPEC-004-AC-11:** Given a tip is evaluated for render while Priya's device has no connectivity and no record exists yet, when the initialization write cannot complete, then nothing is held or retried, and the record is created the next time the tip is evaluated with connectivity.

**FEAT-30.SPEC-004-AC-07:** Given two dismissal requests for the same tip and the same user arrive at effectively the same time, when both are processed, then the end state is dismissed = true exactly once in effect, with no error in either request.

**FEAT-30.SPEC-004-AC-08:** Given a Client Contact's details are erased on request (FEAT-18) after a dismissal write was queued but not yet applied, when the erasure completes first, then the dismissal write is discarded with no error, since the flag's host record no longer exists.

**FEAT-30.SPEC-004-AC-09:** Given a dismissal request references a tip_id no longer present in the current help-content catalog, when this automation processes it, then it is accepted as a no-op and no error is produced.

### User Story 5 - Contextual Help Content & Behavior Rules (Priority: P3)

Governs which guidance content each role may see, that guidance is always advisory and never blocking, and that a dismissed tip is suppressed on every future render.

**Acceptance Scenarios:**

**FEAT-30.SPEC-005-AC-01:** Given a tip_id exists in the current help-content catalog, when FEAT-30.SPEC-001 checks its record, then the read succeeds and eligibility is decided from the dismissed field.

**FEAT-30.SPEC-005-AC-02:** Given a tip_id has been retired from the current help-content catalog, when any of FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 evaluate it, then it is treated as ineligible to render, with no error shown anywhere.

**FEAT-30.SPEC-005-AC-03:** Given a Help-Tip Dismissal State record is created for Nadia's own tip, when the host_record is checked, then it references Nadia's Freelancer Account and no other account.

**FEAT-30.SPEC-005-AC-04:** Given a tip has never been dismissed, when its record is checked, then dismissed is false and dismissed_at is unset.

**FEAT-30.SPEC-005-AC-05:** Given a tip has just been dismissed, when its record is checked immediately after, then dismissed is true and dismissed_at holds the moment of dismissal -- the two fields are never found in disagreement.

**FEAT-30.SPEC-005-AC-06:** Given Nadia is viewing her own dismissal state indirectly through FEAT-30.SPEC-001's render check, when the check runs, then it succeeds because she is reading only her own record.

**FEAT-30.SPEC-005-AC-07:** Given Dana is in a support session, when the host screen would otherwise check a dismissal state to render a tip, then that read never occurs and no tip is rendered to her.

**FEAT-30.SPEC-005-AC-08:** Given Nadia, Owen, or Priya looks for a list of tips they have previously dismissed, when they search the product, then no such screen or control exists anywhere.

**FEAT-30.SPEC-005-AC-09:** Given Owen has not yet dismissed a tip, when he taps "Don't show this again," then the transition from not-dismissed to dismissed succeeds.

**FEAT-30.SPEC-005-AC-10:** Given Priya wants to dismiss a tip on Owen's behalf, when she looks for a way to do so, then no such control exists -- each contact dismisses only their own guidance state.

**FEAT-30.SPEC-005-AC-11:** Given Dana wants to dismiss a tip for Nadia during a support session, when she looks for a dismiss control, then none is shown to her, consistent with SC-04.

**FEAT-30.SPEC-005-AC-12:** Given Owen has permanently dismissed a tip, when he looks for a way to undismiss or restore it, then no such control exists anywhere in the product.

**FEAT-30.SPEC-005-AC-13:** Given Dana wants to reset a dismissed tip for Nadia, when she looks for a reset control, then none exists, per FEAT-30's Non-Goals.

**FEAT-30.SPEC-005-AC-14:** Given a tip is eligible to render for Nadia for the first time, when FEAT-30.SPEC-004 initializes its record, then dismissed defaults to false with no user action required.

**FEAT-30.SPEC-005-AC-15:** Given Priya (Reviewer) is viewing a milestone screen, when tip eligibility is evaluated for the Approve control, then no tip is offered to her, because her role has no entitlement to that control at all (XBR-08).

**FEAT-30.SPEC-005-AC-16:** Given a tip has been dismissed by Nadia, when the host screen that would show it renders again, then the tip's affordance is suppressed while the rest of the screen renders normally and unaffected.

**FEAT-30.SPEC-005-AC-17:** Given Owen has dismissed a tip explaining the pay-invoice action, when he next opens the invoice screen, then he can still pay the invoice without restriction -- the dismissal never gates the underlying action, consistent with the advisory-only rule.

**FEAT-30.SPEC-005-AC-18:** Given Priya is promoted from Reviewer to Primary contact (FEAT-18) mid-session, when guidance eligibility is next evaluated for her, then Primary-scoped tips and topics become eligible for the first time, with no retroactive change to her existing dismissal records.

**FEAT-30.SPEC-005-AC-19:** Given a Client Contact's details are erased on request (FEAT-18), when the erasure completes, then their Help-Tip Dismissal State records are removed together with the rest of the erased record, since this spec defines no independent retention for them.

**FEAT-30.SPEC-005-AC-20:** Given Dana is in a support session, when she looks for a way to view Nadia's (or any user's) dismissal history, then no such screen or control exists, consistent with her Notifications & Help entitlement being limited to delivery warnings.

**FEAT-30.SPEC-005-AC-21:** Given Nadia is entitled to a control's tip, the tip_id is in the catalog, and she has tapped "Got it" on it without ever choosing "Don't show this again," when she next encounters the control on any later render, then the affordance is offered again -- and it is offered again on every subsequent encounter until she permanently dismisses it.

**FEAT-30.SPEC-005-AC-22:** Given Owen has permanently dismissed the tip for a topic that appears in the Client Portal Help Reference, when he opens that topic, then the topic and its full explanation are still shown, the "Don't show this again" action is not rendered, and the static note "Inline tips for this topic are turned off" appears in its place with no restore control.

### Edge Cases

- **FEAT-30.SPEC-001 (Contextual Help Tooltip):** A second quick tap toggles the popover closed with no duplicate, and losing connectivity keeps the already rendered explanation visible with no error. A Don't show this again write that fails for another reason is not retried (the affordance may reappear later), while one chosen offline stays suppressed on the device and is retried on reconnection via FEAT-30.SPEC-004. Source: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-001-contextual-help-tooltip.md` (section: Edge Cases)
- **FEAT-30.SPEC-002 (Freelancer Help Reference):** The reference renders fully offline as static content with only a dismiss held on the device, dismissing the same topic from the reference and its inline tip sets the same state with the second write a no-op, and expanding then collapsing all topics loses nothing. A topic added in a later release appears on next load as not yet dismissed. Source: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-002-freelancer-help-reference.md` (section: Edge Cases)
- **FEAT-30.SPEC-003 (Client Portal Help Reference):** The Primary-scoped topic list renders fully offline, dismissals from the reference and the inline tip are idempotent, and a Reviewer promoted to Primary sees the additional Primary-scoped topics on next open. A newly invited Reviewer sees only Reviewer-scoped groups with every topic not yet dismissed. Source: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-003-client-portal-help-reference.md` (section: Edge Cases)
- **FEAT-30.SPEC-004 (Help Tip Dismissal Recording):** Duplicate or concurrent dismissals for the same tip and user end in the same dismissed state with the second a no-op, and a first-eligible-render trigger racing a dismissal creates the record first. A deferred dismissal found already dismissed from another device is a no-op and the first dismissed_at stands. Source: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-004-help-tip-dismissal-recording.md` (section: Edge Cases)
- **FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules):** A tip retired from the catalog is ineligible to render with no error, evaluating the same tip twice in one render yields the same decision, and a role change re-evaluates content eligibility on the next render while per-tip dismissal states are retained. A dismissal for a tip not in the current catalog is accepted as a no-op. Source: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-005-contextual-help-content-behavior-rules.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-30.SPEC-001** (Contextual Help Tooltip) as specified: An inline, on-demand explanation for an unfamiliar control, overlaid on a host screen starting at the user's first encounter with the control and offered on every later encounter until permanently dismissed, with a permanent-dismiss action. Full spec: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-001-contextual-help-tooltip.md`
- **FR-002**: The system MUST implement **FEAT-30.SPEC-002** (Freelancer Help Reference) as specified: A short, browsable help reference covering the freelancer dashboard, reachable from any dashboard screen, so Nadia needs no external documentation. Full spec: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-002-freelancer-help-reference.md`
- **FR-003**: The system MUST implement **FEAT-30.SPEC-003** (Client Portal Help Reference) as specified: A short, browsable help reference covering the client-facing portal, scoped to the viewing contact's role, reachable from any portal screen, so a client contact needs no external documentation. Full spec: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-003-client-portal-help-reference.md`
- **FR-004**: The system MUST implement **FEAT-30.SPEC-004** (Help Tip Dismissal Recording) as specified: Owns every write to a user's Help-Tip Dismissal State: the default "not dismissed" record created the first time a tip becomes eligible to render, and the permanent flip to "dismissed" when the user chooses "Don't show this again." Full spec: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-004-help-tip-dismissal-recording.md`
- **FR-005**: The system MUST implement **FEAT-30.SPEC-005** (Contextual Help Content & Behavior Rules) as specified: Governs which guidance content each role may see, that guidance is always advisory and never blocking, and that a dismissed tip is suppressed on every future render. Full spec: `docs/blueprint/specifications/FEAT-30-contextual-help-guidance/FEAT-30.SPEC-005-contextual-help-content-behavior-rules.md`

### Key Entities

- N/A — a guidance layer over existing screens, not a data-owning entity.

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Help tips shown and dismissed are each observable as distinct signals (help_tip_shown, help_tip_dismissed); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-27**: Client-facing screens are usable with a screen reader and keyboard and say plainly when an action needs a connection. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-02**: Client contacts act primarily from mobile browsers, which the portal help reference must serve. Full register: `docs/blueprint/features/assumptions-constraints.md`
