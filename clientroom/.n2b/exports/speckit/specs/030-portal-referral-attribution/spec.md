# Feature Specification: Portal Referral Attribution

**Blueprint feature:** FEAT-33
**Priority tier:** Important
**Build order:** 030 of 33
**Depends on:** FEAT-05, FEAT-14
**Blueprint source:** `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Referral Mark Display (Priority: P2)

Governs where, how, and under what conditions the small "Made with Clientroom" mark renders on every client-facing portal page (FEAT-05) and email (FEAT-14), so it stays discreet, consistent, and never overrides the freelancer's own branding.

**Acceptance Scenarios:**

**FEAT-33.SPEC-001-AC-01:** Given Owen opens the Portal Home screen (FEAT-05.SPEC-003), when the page renders, then the "Made with Clientroom" mark appears in the footer region.

**FEAT-33.SPEC-001-AC-02:** Given Nadia has set a custom logo and brand colour in her Branding Profile (FEAT-19), when any client-facing page or email renders, then the mark keeps its own fixed small treatment and never adopts, resizes to match, or visually competes with her brand colour.

**FEAT-33.SPEC-001-AC-03:** Given Nadia has no Branding Profile set (default neutral branding), when a client-facing page renders, then the mark still renders with its own fixed treatment, unchanged by the absence of a logo.

**FEAT-33.SPEC-001-AC-04:** Given Priya opens a client-facing email composed by FEAT-14.SPEC-001, when the email is delivered, then the mark appears in the email's footer alongside Nadia's own branding.

**FEAT-33.SPEC-001-AC-05:** Given Dana opens a read-only support session (FEAT-31) for a freelancer's account, when she views that freelancer's portal pages, then the mark renders the same way it would for any client contact.

**FEAT-33.SPEC-001-AC-06:** Given any visitor views a client-facing portal page or email, when they follow the mark, then they are taken into FEAT-33.SPEC-002 (Referral Link Capture), never directly into portal content.

**FEAT-33.SPEC-001-AC-07:** Given Nadia is on a Free-tier Subscription Plan, when any client-facing surface renders, then the mark is shown -- there is no plan-based control anywhere that can hide it.

**FEAT-33.SPEC-001-AC-08:** Given Nadia searches Settings and Branding (FEAT-19) for a way to hide or customize the mark, when she reviews the available branding controls, then no such control exists -- only her logo and brand colour are configurable there.

**FEAT-33.SPEC-001-AC-09:** Given Owen opens a client-facing email whose images are blocked by his email client, when the email renders, then the mark, being a text link, remains visible and followable.

**FEAT-33.SPEC-001-AC-10:** Given a client-facing portal page is in an offline or degraded state (per FEAT-05's own States), when the page renders its footer, then the mark still appears, unaffected by the degraded data state.

**FEAT-33.SPEC-001-AC-11:** Given Owen follows the mark from Nadia's portal page, when FEAT-33.SPEC-002 receives the click, then the link never carries or exposes any of Nadia's client or project data -- only the fact that this surface belongs to her Freelancer Account.

**FEAT-33.SPEC-001-AC-12:** Given the same client contact receives two separate client-facing emails close together, when both render, then each carries its own independent footer mark per this spec's surface_scope rule.

**FEAT-33.SPEC-001-AC-13:** Given Nadia previews her own portal home while signed in as the freelancer, when the page renders, then the mark appears exactly as it would to a client contact, with no preview-mode suppression.

### User Story 2 - Referral Link Capture (Priority: P2)

Captures the referring freelancer's portal identifier the moment a visitor follows the "Made with Clientroom" mark, and carries that reference forward toward sign-up.

**Acceptance Scenarios:**

**FEAT-33.SPEC-002-AC-01:** Given Owen follows the "Made with Clientroom" mark on Nadia's portal home page, when this automation fires, then it captures Nadia's Freelancer Account as the referring portal for Owen's visit and routes him to the Referral Landing Page (FEAT-33.SPEC-003).

**FEAT-33.SPEC-002-AC-02:** Given a marketing lead follows the mark inside a client-facing email addressed by Nadia's account, when this automation fires, then it captures Nadia's Freelancer Account as the referring portal, identical to a portal-page follow.

**FEAT-33.SPEC-002-AC-03:** Given a visitor's referring-portal reference was captured, when the visitor proceeds from the Referral Landing Page to sign up (FEAT-20.SPEC-001), then the captured reference is carried forward unchanged into the sign-up entry.

**FEAT-33.SPEC-002-AC-04:** Given the mark's underlying Freelancer Account was deleted before the visitor's click resolves, when this automation attempts capture, then it fails silently, the visitor still reaches the Referral Landing Page, and no reference is carried forward.

**FEAT-33.SPEC-002-AC-05:** Given a visitor's captured reference has exceeded platform parameter: `referral-capture-session-window` with no sign-up yet, when the visitor reaches sign-up, then no referring-portal reference is carried forward.

**FEAT-33.SPEC-002-AC-06:** Given a visitor follows a mark on Portal A and later, in the same session, follows a mark on Portal B, when the visitor eventually signs up, then Portal B is the referring portal carried forward, not Portal A.

**FEAT-33.SPEC-002-AC-07:** Given every follow of the mark, when this automation fires, then `referral_mark_clicked` is emitted noting whether the source was a portal page or an email.

**FEAT-33.SPEC-002-AC-08:** Given a visitor closes the browser after following the mark and returns later in an unrelated session, when the visitor signs up in that new session, then no referring-portal reference is present.

**FEAT-33.SPEC-002-AC-09:** Given two different visitors follow marks on two different portals at the same time, when both captures fire, then each is scoped independently to its own visitor session with no interference.

**FEAT-33.SPEC-002-AC-10:** Given a visitor double-clicks the same mark rapidly, when the second click's capture fires, then it re-captures the same reference with no queued or conflicting state.

**FEAT-33.SPEC-002-AC-11:** Given a capture expires before the visitor reaches sign-up and the visitor had already reached the landing page, when the expiry occurs, then `referral_capture_expired` is emitted noting the visitor had reached the landing page.

### User Story 3 - Referral Landing Page (Priority: P2)

A visitor who followed the "Made with Clientroom" mark without wanting to sign up sees a short public product page, with a one-step way back to the portal if they arrived from one.

**Acceptance Scenarios:**

**FEAT-33.SPEC-003-AC-01:** Given Owen follows the mark from Nadia's portal home while his portal session is active, when the Referral Landing Page renders, then he sees the public description, a "Sign up" button, and a "Back to portal" link.

**FEAT-33.SPEC-003-AC-02:** Given an unauthenticated visitor follows the mark from a client-facing email with no portal session of their own, when the page renders, then no "Back to portal" link is shown.

**FEAT-33.SPEC-003-AC-03:** Given Priya is on this page with a captured referring-portal reference, when she taps "Sign up", then she is taken to FEAT-20.SPEC-001 with that reference carried forward silently.

**FEAT-33.SPEC-003-AC-04:** Given Owen is on this page with an active portal session, when he taps "Back to portal", then he is taken directly to Portal Home (FEAT-05.SPEC-003).

**FEAT-33.SPEC-003-AC-05:** Given a client contact's portal session had already expired when they followed the mark, when this page renders and they tap "Back to portal", then they are taken to FEAT-05.SPEC-001 (Request Sign-In Link) rather than directly into the portal.

**FEAT-33.SPEC-003-AC-06:** Given Nadia (already signed in) reaches this page, when it renders, then no "Back to portal" link is shown, since she is not a client contact with a portal session.

**FEAT-33.SPEC-003-AC-07:** Given any visitor is on this page, when they look for any indication of which freelancer's portal referred them, then no such indication is present anywhere on the page.

**FEAT-33.SPEC-003-AC-08:** Given a visitor arrives at this page's address without having followed any mark, when it renders, then it shows the same public content with no "Back to portal" link and no referring-portal reference carried forward.

**FEAT-33.SPEC-003-AC-09:** Given a visitor loses connectivity while this page is open, when they attempt to tap "Sign up" or "Back to portal", then both actions are disabled with the note "This needs a connection. Try again once you're back online." until connectivity is restored.

**FEAT-33.SPEC-003-AC-10:** Given this page renders, when it loads, then `referral_landing_viewed` is emitted noting whether the visitor arrived with an active portal session.

**FEAT-33.SPEC-003-AC-11:** Given a visitor taps "Sign up" twice in rapid succession, when the second tap occurs, then it has no additional effect beyond the first navigation already underway.

### User Story 4 - Referral Attribution Recording (Priority: P2)

Creates the Referral Attribution record once, at sign-up, from the referring portal captured by FEAT-33.SPEC-002 and the self-reported "how did you hear" answer captured by FEAT-20, degrading either half to unknown when it is missing.

**Acceptance Scenarios:**

**FEAT-33.SPEC-004-AC-01:** Given a new freelancer's account was referred by a resolvable portal and she also answered the "How did you hear" question, when this automation fires, then the Referral Attribution record is created with both `referring_portal` and `self_reported_source` populated, and both `signup_attributed_to_portal` and `signup_source_answered` are emitted.

**FEAT-33.SPEC-004-AC-02:** Given a new freelancer arrived with no referring-portal reference but answered the question, when this automation fires, then the record is created with `referring_portal` set to "unknown" and `self_reported_source` populated, and only `signup_source_answered` fires among the two conditional events.

**FEAT-33.SPEC-004-AC-03:** Given a new freelancer arrived via a resolvable referring portal but skipped the question, when this automation fires, then the record is created with `self_reported_source` set to "unknown" and `referring_portal` populated, and only `signup_attributed_to_portal` fires among the two conditional events.

**FEAT-33.SPEC-004-AC-04:** Given a new freelancer arrived with neither a referring portal nor an answer, when this automation fires, then the record is still created with both fields set to "unknown," and `referral_attribution_recorded` fires noting both as unknown, with neither conditional event firing.

**FEAT-33.SPEC-004-AC-05:** Given a referring-portal reference points to a Freelancer Account that was deleted before this sign-up completed, when this automation resolves the reference, then it is recorded as "unknown" rather than causing an error.

**FEAT-33.SPEC-004-AC-06:** Given a Referral Attribution record already exists for a Freelancer Account, when FEAT-20.SPEC-004's hand-off is somehow delivered again for that same account, then this automation takes no action and no second record is created.

**FEAT-33.SPEC-004-AC-07:** Given this automation's recording fails after the Freelancer Account was already created, when the failure occurs, then the Freelancer Account remains fully usable and no error is surfaced to the new freelancer.

**FEAT-33.SPEC-004-AC-08:** Given this automation successfully creates a Referral Attribution record, when the creation completes, then `referral_attribution_recorded` is emitted noting whether each of the two fields was known or unknown.

**FEAT-33.SPEC-004-AC-09:** Given two different new freelancers complete sign-up at effectively the same time, when this automation fires for each, then each creates its own independent record with no shared state or race between them.

**FEAT-33.SPEC-004-AC-10:** Given a Referral Attribution record has been created for a Freelancer Account, when any process attempts to update it later, then no update path exists -- the record remains exactly as recorded at creation.

**FEAT-33.SPEC-004-AC-11:** Given a Freelancer Account whose Referral Attribution record was created, when that account is later deleted (FEAT-24), then the Referral Attribution record is deleted with it, with no independent retention.

**FEAT-33.SPEC-004-AC-12:** Given this automation creates a record with an unknown referring portal, when FEAT-33.SPEC-005 is later consulted for that record's visibility, then no persona -- including Nadia and Dana -- has any path to view the individual record.

### User Story 5 - Referral Data Access Restriction (Priority: P2)

Enforces that no persona browses an individual Referral Attribution record inside the product -- attribution exists only as an aggregate figure for the Growth Through Referral metric, and the referring freelancer is never told who signed up from her portal.

**Acceptance Scenarios:**

**FEAT-33.SPEC-005-AC-01:** Given Nadia is signed in and looks anywhere in Settings, her Dashboard, or Client & Project Management for a way to see who signed up from her portal, when she searches those areas, then no such screen or control exists anywhere.

**FEAT-33.SPEC-005-AC-02:** Given Owen is signed in to his client portal, when he looks for any referral-related data about other freelancers or accounts, then no such capability is shown -- his access to Portal Referral relates only to viewing the mark itself (FEAT-33.SPEC-001), never to attribution data.

**FEAT-33.SPEC-005-AC-03:** Given Priya is signed in to her client portal, when she looks for referral attribution data, then the same result as Owen's applies -- no such capability exists for her role either.

**FEAT-33.SPEC-005-AC-04:** Given Dana opens a read-only support session (FEAT-31) for a freelancer's account that has an associated Referral Attribution record, when she reviews the account's data during that session, then the Referral Attribution record is not shown, even though her session otherwise reads most of that account's data.

**FEAT-33.SPEC-005-AC-05:** Given a Referral Attribution record exists, when any persona attempts to reach a direct view of it (by any means the product exposes), then no route or screen exists to do so.

**FEAT-33.SPEC-005-AC-06:** Given the Growth Through Referral metric is computed in aggregate, when any persona views any in-product screen (Dashboard, Settings, or otherwise), then that aggregate figure is not surfaced there -- it exists only as a success-metrics.md reporting artifact outside the product's own screens.

**FEAT-33.SPEC-005-AC-07:** Given a Referral Attribution record has been created, when any process attempts to create a second one for the same Freelancer Account through any persona-facing control, then no such control exists -- creation is exclusively FEAT-33.SPEC-004's own automated action.

**FEAT-33.SPEC-005-AC-08:** Given a Referral Attribution record exists, when any persona attempts to edit or delete it directly (outside FEAT-24's account-deletion cascade), then no edit or delete control exists anywhere in the product for any role.

**FEAT-33.SPEC-005-AC-09:** Given every field in the Referral Attribution entity, when this spec's Field Validation Rules are reviewed, then each field is explicitly addressed as "no validation beyond data type," confirming none was accidentally skipped.

**FEAT-33.SPEC-005-AC-10:** Given a Freelancer Account with an associated Referral Attribution record is deleted through FEAT-24, when the deletion completes, then the Referral Attribution record is removed with it -- the only removal path this spec recognizes for the entity.

### Edge Cases

- **FEAT-33.SPEC-001 (Referral Mark Display):** The referral mark renders with a fixed small treatment even with default branding, is a text link so image-blocking email clients do not hide it, and still renders in degraded or offline portal states. A freelancer previewing her own portal sees it exactly as a contact would, since it cannot be hidden on any plan. Source: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-001-referral-mark-display.md` (section: Edge Cases)
- **FEAT-33.SPEC-002 (Referral Link Capture):** A referring account deleted before the click fails capture while the visitor still reaches the landing page, and a new unrelated session carries no referring-portal reference. The last mark followed in a session wins (last-click-wins), and a forwarded email still resolves to the account it was addressed for. Source: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-002-referral-link-capture.md` (section: Edge Cases)
- **FEAT-33.SPEC-003 (Referral Landing Page):** A visitor arriving directly renders the page identically to the default state with no referring-portal reference, and a portal session expiring while the page is open hands the Back to portal tap to FEAT-05's expired-link handling. Double taps on Sign up have no extra effect, and no concurrent-edit conflict applies since no shared entity is updated. Source: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-003-referral-landing-page.md` (section: Edge Cases)
- **FEAT-33.SPEC-004 (Referral Attribution Recording):** A referring account deleted before sign-up completes records the source as unknown, a duplicate hand-off from FEAT-20.SPEC-004 is caught by the idempotency check and creates no second record, and whitespace-only sources never arrive since FEAT-20.SPEC-004 normalizes them to unknown. Concurrent sign-ups each record independently. Source: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-004-referral-attribution-recording.md` (section: Edge Cases)
- **FEAT-33.SPEC-005 (Referral Data Access Restriction):** Referral Attribution records never appear in a support session, and no who-referred-this-signup or my-referrals screen, route or record identifier exists anywhere in the product. No referral data of any kind is exposed, so the aggregate cannot be derived by cross-referencing. Source: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-005-referral-data-access-restriction.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-33.SPEC-001** (Referral Mark Display) as specified: Governs where, how, and under what conditions the small "Made with Clientroom" mark renders on every client-facing portal page (FEAT-05) and email (FEAT-14), so it stays discreet, consistent, and never overrides the freelancer's own branding. Full spec: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-001-referral-mark-display.md`
- **FR-002**: The system MUST implement **FEAT-33.SPEC-002** (Referral Link Capture) as specified: Captures the referring freelancer's portal identifier the moment a visitor follows the "Made with Clientroom" mark, and carries that reference forward toward sign-up. Full spec: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-002-referral-link-capture.md`
- **FR-003**: The system MUST implement **FEAT-33.SPEC-003** (Referral Landing Page) as specified: A visitor who followed the "Made with Clientroom" mark without wanting to sign up sees a short public product page, with a one-step way back to the portal if they arrived from one. Full spec: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-003-referral-landing-page.md`
- **FR-004**: The system MUST implement **FEAT-33.SPEC-004** (Referral Attribution Recording) as specified: Creates the Referral Attribution record once, at sign-up, from the referring portal captured by FEAT-33.SPEC-002 and the self-reported "how did you hear" answer captured by FEAT-20, degrading either half to unknown when it is missing. Full spec: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-004-referral-attribution-recording.md`
- **FR-005**: The system MUST implement **FEAT-33.SPEC-005** (Referral Data Access Restriction) as specified: Enforces that no persona browses an individual Referral Attribution record inside the product -- attribution exists only as an aggregate figure for the Growth Through Referral metric, and the referring freelancer is never told who signed up from her portal. Full spec: `docs/blueprint/specifications/FEAT-33-portal-referral-attribution/FEAT-33.SPEC-005-referral-data-access-restriction.md`

### Key Entities

- Referral Attribution (create, read)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least a third of new freelancer signups in year one cite seeing another freelancer's client portal (as a client or peer) as how they heard about the product (metric: Growth Through Referral). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Referral mark clicks, signups attributed to a portal and answered signup sources are each observable as distinct signals (referral_mark_clicked, signup_attributed_to_portal, signup_source_answered). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-12**: The permanent free tier is assumed to feed the portal-driven growth loop. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: Personal data worldwide is treated as GDPR-class, which is why referral data is never exposed to any screen or operator. Full register: `docs/blueprint/features/assumptions-constraints.md`
