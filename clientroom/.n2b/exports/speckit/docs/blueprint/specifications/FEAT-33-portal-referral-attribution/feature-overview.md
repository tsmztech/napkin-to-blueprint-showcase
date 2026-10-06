---
document_type: feature-overview
feature_number: FEAT-33
feature_name: Portal Referral Attribution
feature_slug: portal-referral-attribution
priority_tier: Important
feature_type: Lifecycle
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 5
screen_count: 1
automation_count: 2
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Portal Referral Attribution

## Summary

**Feature:** Portal Referral Attribution
**ID:** FEAT-33
**Description:** Every client-facing portal page and email carries a small, discreet "Made with Clientroom" link. A client contact, or a fellow freelancer shown the portal, can follow it to start an account, and every new sign-up records how the freelancer found the product.
**Priority:** Important
**Phase:** MVP
**Type:** Lifecycle
**Rationale:** BRIEF.md, Business Context: "The intended growth loop is that new freelancers arrive after a client or peer sees someone else's portal," and Success Criteria: "Most new freelancers arrive because a client or peer saw someone else's portal." No draft feature gave a portal viewer a way to reach the product or measured whether the loop works. Research shows white-labeled branding is highly valued (SuiteDash, HIGH), so the mark stays small and never competes with the freelancer's own brand. Important rather than Core because the client-facing loop works without it; MVP because the brief's growth success criterion must be measurable from the very first portal. [AUDIT-ADDED: 1 -- the brief-goal walk found no feature delivering or measuring the portal-driven growth loop named in BRIEF.md's Success Criteria]

**Key Capabilities:**
- Discreet product mark -- a small "Made with Clientroom" link at the foot of client-facing pages and emails
- Start from a portal -- a visitor who follows the link lands on sign-up, with the referring portal recorded
- Tell us how you heard -- each new freelancer is asked, optionally, how they found Clientroom

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-33.SPEC-001 | Referral Mark Display | Logic/Rule | All | Governs where and how the "Made with Clientroom" mark renders on every client-facing page and email, without overriding the freelancer's own branding |
| FEAT-33.SPEC-002 | Referral Link Capture | Automation | Nadia | Captures the referring freelancer's portal identifier when a visitor follows the mark, and carries it toward sign-up |
| FEAT-33.SPEC-003 | Referral Landing Page | Screen | All | Visitor who follows the mark without wanting to sign up sees a short public product page and can return to the portal in one step |
| FEAT-33.SPEC-004 | Referral Attribution Recording | Automation | Nadia | Creates the Referral Attribution record at sign-up from the captured portal and/or the self-reported "how did you hear" answer, degrading to unknown on either gap |
| FEAT-33.SPEC-005 | Referral Data Access Restriction | Logic/Rule | Nadia, Dana | Enforces that no persona browses individual referral records inside the product; attribution is exposed only in aggregate to the growth metric |

Frontmatter counts match this table: spec_count: 5, screen_count: 1, automation_count: 2, logic_rule_count: 2, integration_count: 0, notification_count: 0.

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Discreet product mark | FEAT-33.SPEC-001 | Placement and rendering rule for the mark on every client-facing page and email | Phase 2 (Explicit) |
| Start from a portal | FEAT-33.SPEC-002, FEAT-33.SPEC-003, FEAT-33.SPEC-004 | Link capture on click, a landing page for viewers who decline to sign up, and attribution recorded once sign-up completes | Phase 2 (Explicit), elaborated in Phase 4 (Trigger-Response) |
| Tell us how you heard | FEAT-33.SPEC-004 | The self-reported answer is recorded into the Referral Attribution entity; the question itself is presented inside FEAT-20's onboarding screen (cross-feature -- see Cross-Feature Touchpoints) | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-33.SPEC-005 | Referral Data Access Restriction | Phase 5 (Rule-Constraint Discovery) | The Access field states plainly that "no persona browses referral data inside the product" and that Nadia is never told who signed up from her portal; this authorization/visibility constraint governs both SPEC-004's output and every role's access, so it needed its own rule spec rather than a buried note |
| FEAT-33.SPEC-002 | Referral Link Capture | Phase 4 (Trigger-Response Analysis) | The feature description states the referring portal is recorded but never states the mechanism; following a link and later completing sign-up are separate events, so a capture-and-carry automation is required between them |

## Entity-Lifecycle Coverage Matrix

**Entity: Referral Attribution**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-33.SPEC-004 | Created once, at sign-up, from the captured referring portal (SPEC-002) and/or the self-reported source answer (asked in FEAT-20); either half may be "unknown" | Matches the dependency map's lifecycle line: "Created by FEAT-33 at sign-up (answer captured in FEAT-20)" |
| Read (single) | N/A | No screen in this feature (or elsewhere) exposes an individual Referral Attribution record | Explicit product decision, not a gap -- see FEAT-33.SPEC-005 and the Access field: "No persona browses referral data inside the product" |
| Read (list) | N/A | No browsable list of referral records exists; the entity is read only in aggregate to compute the Growth Through Referral metric in success-metrics.md | Explicit product decision -- see FEAT-33.SPEC-005; aggregation is a success-metrics reporting concern, not a feature screen |
| Update | N/A | The dependency map states the entity is "Never updated" | Explicit non-goal, sourced from feature-dependency-map.md's Referral Attribution lifecycle line |
| Delete/Archive | N/A within this feature | Hard-deleted by FEAT-24 as part of full account deletion, per the dependency map's lifecycle line ("Deleted by FEAT-24 with the account"); no restore path and no independent retention window apply -- the record's lifetime is bound entirely to the owning Freelancer Account's lifetime | Cross-feature ownership, not a gap -- see Cross-Feature Touchpoints |
| State Transition | N/A | The entity carries no status field and passes through no states after creation (product-features.md, Data Notes: recorded_at only, never edited) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Freelancer Account | FEAT-33.SPEC-002 | Resolves which freelancer's portal a followed link belongs to, to capture the referring-portal reference |
| Freelancer Account | FEAT-33.SPEC-004 | Attaches the new Referral Attribution record to the Freelancer Account being created at sign-up |
| Branding Profile | FEAT-33.SPEC-001 | Confirms the mark's placement never overrides the freelancer's logo or brand colour (XBR-31) |
| Notification | FEAT-33.SPEC-001 | Confirms the mark's footer placement inside the emails FEAT-14 already sends, rather than this feature composing its own message |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Visitor follows the "Made with Clientroom" mark | Capture the referring freelancer's portal identifier for the session and fire the referral_mark_clicked signal | Standalone Automation | FEAT-33.SPEC-002 |
| New freelancer completes sign-up (FEAT-20) | Create the Referral Attribution record from the captured referring portal and the self-reported source answer, firing signup_attributed_to_portal and/or signup_source_answered | Standalone Automation | FEAT-33.SPEC-004 |
| Attribution cannot be recorded (e.g., the captured portal reference has expired or is missing) | Sign-up continues normally; the source is recorded as unknown rather than blocking account creation | Inline in triggering automation | FEAT-33.SPEC-004 |
| Anyone attempts to browse or open an individual referral record inside the product | The capability is not shown; only the aggregate growth figure exists, in success-metrics.md's reporting | Standalone Logic/Rule | FEAT-33.SPEC-005 |
| Freelancer account is deleted | Referral Attribution record is deleted with the account (no independent retention) | Cross-feature | FEAT-24 responsibility |

## Shared Context

**Shared Entities:**
- Referral Attribution -- created by SPEC-004 from data captured by SPEC-002 and the onboarding answer owned by FEAT-20; its visibility is restricted by SPEC-005. Fields: referring_portal (reference to a Freelancer Account, or unknown), self_reported_source (free-text answer, or unknown), recorded_at (required, immutable).
- Freelancer Account -- read (not owned) by SPEC-002 to resolve the referring portal and by SPEC-004 to attach the new attribution record to the freelancer being created; owned end-to-end by FEAT-20 (create) and FEAT-21 (update).

**Shared UI Patterns:**
- The mark itself (SPEC-001) is a single small, consistent footer element reused verbatim across every FEAT-05 portal page and every FEAT-14 email -- Spec Writers for both surfaces should treat its rendering rule as authored once here and referenced, not restated per surface.
- The Referral Landing Page (SPEC-003) and the sign-up entry point it links to (FEAT-20) share the same "return to portal in one step" affordance for a visitor who arrived via the mark but has a portal to return to.

**Shared Validation:**
- SPEC-005 defines the access restriction that SPEC-004's output must respect; no other spec in this feature re-derives who may view attribution data.

## Internal Dependency Map

```
SPEC-001 (Referral Mark Display) -> [visitor clicks the mark on a portal page or email] -> SPEC-002 (Referral Link Capture)
SPEC-002 (Referral Link Capture) -> [visitor chooses to sign up] -> FEAT-20 (Onboarding / First-Run Setup) sign-up flow
SPEC-002 (Referral Link Capture) -> [visitor declines to sign up] -> SPEC-003 (Referral Landing Page)
SPEC-003 (Referral Landing Page) -> [client contact returns to the portal] -> FEAT-05 (Client Portal Access)
SPEC-004 (Referral Attribution Recording) -> [reads the portal captured by] -> SPEC-002 (Referral Link Capture)
SPEC-004 (Referral Attribution Recording) -> [reads the self-reported answer from] -> FEAT-20 (Onboarding / First-Run Setup)
SPEC-004 (Referral Attribution Recording) -> [output visibility governed by] -> SPEC-005 (Referral Data Access Restriction)
```

**Default Entry:** SPEC-001 (Referral Mark Display) -- this feature has no in-product screen a user navigates to directly; it is entered only by a visitor following the mark that SPEC-001 places on FEAT-05 portal pages and FEAT-14 emails.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-33.SPEC-001 | Outbound | FEAT-05 (Client Portal Access [Magic-Link Login]) | The mark renders at the foot of every client-facing portal page | Portal page renders |
| FEAT-33.SPEC-001 | Outbound | FEAT-14 (Notifications [Email]) | The mark renders at the foot of every client-facing email | Email is composed for sending |
| FEAT-33.SPEC-001 | Inbound | FEAT-19 (Freelancer Branding) | The mark's rendering rule never overrides the freelancer's logo or brand colour (XBR-31) | Every client-facing render |
| FEAT-33.SPEC-002 | Outbound | FEAT-20 (Onboarding / First-Run Setup) | The captured referring portal is carried into the sign-up and first-run flow | Visitor who followed the mark chooses to sign up |
| FEAT-33.SPEC-003 | Outbound | FEAT-05 (Client Portal Access [Magic-Link Login]) | One-step return to the portal from the public landing page | Client contact taps "back to portal" |
| FEAT-33.SPEC-004 | Inbound | FEAT-20 (Onboarding / First-Run Setup) | The optional "How did you hear about us?" answer, asked during onboarding, is passed to attribution recording | New freelancer answers or skips the question |
| FEAT-33 (Referral Attribution entity) | Outbound | FEAT-24 (Data Export & Account Deletion) | The Referral Attribution record is deleted when the owning Freelancer Account is deleted | Freelancer requests account deletion |

## Non-Functional Notes

**Data volumes / growth:** One Referral Attribution record is created per new freelancer sign-up at most (never per portal view or per mark click) -- volume tracks the sign-up rate, not portal traffic; no distinct volume expectation beyond the freelancer base itself is set in assumptions-constraints.md.

**Responsiveness:** The mark and the Referral Landing Page are static, lightweight surfaces reached from a client-facing page a visitor is often viewing on a phone; they fall under ASMP-21's general expectation that client-facing pages become interactive within roughly 2 seconds on a typical mobile connection (assumptions-constraints.md, Non-Functional Expectations).

**Data sensitivity / privacy:** Low personal data -- a link between two Freelancer Accounts and a free-text answer, never client or project data (feature-dependency-map.md, Referral Attribution Data Sensitivity line); used only in aggregate, and the referring freelancer is never told who signed up (product-features.md, Access).

**Compliance flags:** GDPR-class handling applies because the record references personal Freelancer Account data (ASMP-24); it carries no independent retention window and is deleted entirely when the referencing account is deleted (FEAT-24), so no separate purge policy is needed for this feature.

## Non-Goals

- **An in-product referral analytics view for the referring freelancer** -- Excluded per product-features.md, Access: "Nadia is never told who signed up from her portal, and attribution is used only in aggregate to measure the growth loop." SPEC-005 enforces this as a rule rather than as an absent screen.
- **Hiding the mark on any current plan** -- Excluded per product-features.md, Validation & Limits: "The mark is shown on every plan in MVP and cannot be hidden." Whether paid plans may hide it later is an open pricing question the brief defers (BRIEF.md, Open Questions -- pricing), not a decision this feature can make.
- **Any exposure of client, project, or freelancer portal content through the mark or landing page** -- Excluded per scope-boundaries.md (SC-03): public or anonymous portal access is out of scope, and the mark leads only to a public product page, never to portal content.
- **A localized or multi-language public landing page** -- Excluded per scope-boundaries.md (SC-20): the product is English-only at launch, so SPEC-003's landing page carries no localization behavior.
- **A native-app deep link for the referral mark** -- Adjacency exclusion per scope-boundaries.md (SC-06): the product ships no native apps, so the mark and landing page are web-only, reached through the mobile browser like every other client-facing surface.
