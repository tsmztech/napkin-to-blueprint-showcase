---
document_type: feature-overview
feature_number: FEAT-27
feature_name: Custom Domain per Freelancer
feature_slug: custom-domain-per-freelancer
priority_tier: Nice-to-Have
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 4
screen_count: 1
automation_count: 0
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

# Feature Breakdown Brief: Custom Domain per Freelancer

## Summary

**Feature:** Custom Domain per Freelancer
**ID:** FEAT-27
**Description:** A freelancer points her own domain at her portal so clients see the freelancer's brand end-to-end.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Platform
**Rationale:** BRIEF.md, Ecosystem & Integrations: "ideally their own custom domain (timing is an open question)." Nice-to-Have because the shared portal already fulfills the branding promise (FEAT-19) without it; phased Later as the brief itself defers the timing question. Fully white-labeled portals are an established, well-received pattern in the market, which keeps this on the roadmap, while the phase stays Later because BRIEF.md leaves its timing open and the MVP branding (FEAT-19) already delivers the brand promise.

**Key Capabilities:**
- Add a domain — freelancer enters her own domain
- Verify and go live — once verified, the portal is reachable at the freelancer's own domain

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-27.SPEC-001 | Custom Domain Settings | Screen | Nadia (Full); Dana (View, read-only inside a logged support session per FEAT-31) | Nadia adds, views, retries, and removes her custom domain and sees its current verification state |
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Integration | Nadia (Full — configures and monitors); Dana (View, read-only via FEAT-31); Owen, Priya (experience whichever domain this spec determines is active, with no direct control) | Verifies that Nadia controls the domain she added and serves her portal securely at it once verified, reporting verification, failure, and re-check results back |
| FEAT-27.SPEC-003 | Custom Domain Validation & Fallback Rule | Logic/Rule | Nadia (Full); Owen, Priya (protected by the always-available fallback, with no direct control) | Enforces one domain per freelancer account and guarantees the shared default domain always remains reachable as a fallback |
| FEAT-27.SPEC-004 | Custom Domain Verified Confirmation | Notification | Nadia (sole recipient) | Emails Nadia once her custom domain is verified and live |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Add a domain | FEAT-27.SPEC-001, FEAT-27.SPEC-003 | The settings screen captures the domain name; the validation rule enforces the one-domain-per-account limit and domain format before the record is created | Phase 2 (Explicit) |
| Verify and go live | FEAT-27.SPEC-001, FEAT-27.SPEC-002, FEAT-27.SPEC-004 | The Integration spec verifies control of the domain and serves the portal at it; the settings screen surfaces pending/verified/failed status; the confirmation email fires once live | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-32) names a domain-verification and secure-serving capability this feature requires; the feature-dependency-map.md External Touchpoints row names FEAT-27 as the expected owner of this Integration spec |
| FEAT-27.SPEC-003 | Custom Domain Validation & Fallback Rule | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field's one-domain-per-account limit and always-available fallback is the authority behind cross-feature rule XBR-35 (owned by FEAT-27) and governs both the add flow and the removal/failure paths; its cross-feature reach (FEAT-05 depends on it) crossed the threshold for a standalone spec rather than an inline note |
| FEAT-27.SPEC-004 | Custom Domain Verified Confirmation | Phase 4 (Notification surfacing) | The Communications field names a confirmation email with a defined trigger (verified and live), audience (Nadia), and channel (email) -- this crosses the inline-toast threshold and requires a standalone Notification spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Custom Domain Record**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-27.SPEC-001 | Add-domain form on the Custom Domain Settings screen; the entered domain is checked against FEAT-27.SPEC-003 before the record is created | -- |
| Read (single) | FEAT-27.SPEC-001 | The settings screen displays the one configured domain and its current verification state | -- |
| Read (list) | N/A | At most one Custom Domain Record exists per freelancer account (feature-dependency-map.md, Relationships) -- there is nothing to list | -- |
| Update | FEAT-27.SPEC-002 | The domain-verification capability writes verification_state as it progresses (Added -> Verifying -> Verified / Verification Failed with a specific reason); FEAT-27.SPEC-001 also updates domain_name if Nadia replaces her domain, which re-triggers verification via FEAT-27.SPEC-002 | -- |
| Delete/Archive | FEAT-27.SPEC-001 | Hard delete: Nadia removes her configured domain from the settings screen and the portal immediately reverts to the shared default domain (FEAT-27.SPEC-003 fallback rule). No restore path -- re-adding the same domain starts verification from scratch. No cascade -- nothing else references the record. No retention/purge window: the record carries no personal data (feature-dependency-map.md, Data Sensitivity: None), so deletion is immediate and complete. Also deleted as part of a full account deletion, owned by FEAT-24 | Cross-feature: FEAT-24 (Data Export & Account Deletion) also deletes this record |
| State Transition | FEAT-27.SPEC-002 | Added -> Verifying -> Verified, or Added -> Verifying -> Verification Failed (with a specific reason), reported by the domain-verification capability; a re-check from Verification Failed returns to Verifying | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Branding Profile | FEAT-27.SPEC-001 | The settings screen presents the custom domain alongside the freelancer's existing logo and colour, since a verified domain is typically adopted together with Freelancer Branding (FEAT-19) to complete the white-label experience |

## Side-Effect Inventory

This feature produces zero standalone Automation specs: its only entity-creation side-effect (verification) crosses the product boundary to an external capability and is therefore dispositioned directly to the Integration spec (FEAT-27.SPEC-002) per the Phase 4 decision rule, rather than to an intermediate internal Automation.

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia submits a new domain | Validate against the one-domain-per-account limit and domain format | Standalone Logic/Rule | FEAT-27.SPEC-003 |
| Domain passes validation | Create the Custom Domain Record and initiate verification with the domain-verification capability | Standalone Integration | FEAT-27.SPEC-002 |
| Domain-verification capability reports the domain verified | Update the record to Verified, begin serving the portal securely at the custom domain | Standalone Integration (inbound event) | FEAT-27.SPEC-002 |
| Domain verified and live | Send Nadia a confirmation email | Standalone Notification | FEAT-27.SPEC-004 |
| Domain-verification capability reports a failure | Update the record with the specific failure reason; screen names the problem to Nadia and offers re-check, while the shared default domain keeps serving the portal | Standalone Integration (inbound event), surfaced inline on FEAT-27.SPEC-001 | FEAT-27.SPEC-002 / FEAT-27.SPEC-001 |
| Nadia taps re-check on a failed verification | Re-trigger verification with the domain-verification capability | Standalone Integration | FEAT-27.SPEC-002 |
| Nadia removes her configured domain | Portal and client-facing links revert to the shared default domain, no disruption to client access | Inline in triggering screen (governed by FEAT-27.SPEC-003's fallback rule) | FEAT-27.SPEC-001 |
| Dana opens a logged support session on this account | Views the domain's verification status, read-only | Cross-feature -- logged in touchpoints | FEAT-31 responsibility |

## Shared Context

**Shared Entities:**
- Custom Domain Record -- created and deleted by SPEC-001 (with SPEC-003 gating creation), read by SPEC-001, verification-state updated by SPEC-002. Fields: domain_name (required, one per freelancer), verification_state (Added, Verifying, Verified, Verification Failed with a specific failure reason).

**Shared UI Patterns:**
- Single settings surface -- SPEC-001 is the only screen and carries every state of the domain lifecycle (Empty, Loading/pending, Verified/live, Error with re-check) on one view rather than splitting add/verify/remove into separate screens, consistent with this feature's small, linear Key Capability set.

**Shared Validation:**
- SPEC-003 defines the one-domain-per-account limit, domain format checks, and the fallback guarantee. SPEC-001 and SPEC-002 both reference SPEC-003 rather than duplicating the rule: SPEC-001 enforces it at submission and at removal-time fallback display; SPEC-002 relies on it to know there is at most one record to verify per freelancer.

## Internal Dependency Map

```
SPEC-001 (Custom Domain Settings) -> [Nadia submits a domain] -> SPEC-003 (Custom Domain Validation & Fallback Rule) -> [valid] -> SPEC-002 (Domain Verification & Secure Serving)
SPEC-002 (Domain Verification & Secure Serving) -> [verification succeeds] -> SPEC-001 (status updates to Verified/Live)
SPEC-002 (Domain Verification & Secure Serving) -> [verification succeeds] -> SPEC-004 (Custom Domain Verified Confirmation)
SPEC-002 (Domain Verification & Secure Serving) -> [verification fails] -> SPEC-001 (status names the problem, offers re-check)
SPEC-001 (Custom Domain Settings) -> [Nadia taps re-check] -> SPEC-002 (Domain Verification & Secure Serving)
SPEC-001 (Custom Domain Settings) -> [Nadia removes her domain] -> SPEC-003 (Custom Domain Validation & Fallback Rule) -> [reverts to shared default domain]
```

**Default Entry:** SPEC-001 (Custom Domain Settings) -- the screen shown when Nadia navigates into custom domain setup from her branding settings (FEAT-19).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-27.SPEC-001 | Inbound | FEAT-19 (Freelancer Branding) | Navigation into custom domain setup from branding settings | Nadia goes further than logo and colour |
| FEAT-27.SPEC-001 | Outbound | FEAT-19 (Freelancer Branding) | Reads the Branding Profile to present the custom domain alongside existing branding | Settings screen display |
| FEAT-27.SPEC-002 | Outbound | FEAT-05 (Client Portal Access) | Determines the address -- custom domain or shared default -- that the portal and client-facing links are served at (XBR-35) | Domain verified/live, removed, or failed |
| FEAT-27.SPEC-004 | Outbound | FEAT-14 (Notifications (Email)) | Confirmation email sent through the transactional email delivery capability | Domain verified and live |
| FEAT-27.SPEC-001 / SPEC-002 | Outbound | FEAT-31 (Operator Support Access) | Dana views the domain's verification status, read-only, inside a logged support session | Support session opened |
| FEAT-27.SPEC-001 | Inbound | FEAT-24 (Data Export & Account Deletion) | Custom Domain Record deleted as part of a full account deletion cascade | Nadia deletes her account |

## Non-Functional Notes

**Data volumes / growth:** N/A -- at most one Custom Domain Record exists per freelancer account (feature-dependency-map.md, Relationships), so there is no meaningful volume or growth pattern to plan for.

**Responsiveness:** This is a one-time configuration step for Nadia, not an ongoing client-facing dependency (product-features.md, States: "Offline-degraded: N/A -- this is a one-time configuration step, not an ongoing runtime dependency for either party's session"), so the client-facing responsiveness targets in ASMP-21 do not apply to it directly. The settings screen itself shows real progress while verification is pending (records can take time to propagate) rather than a fixed time target, per the loading-state convention in ASMP-27. Once live, every client-facing page served at the custom domain still meets the same responsiveness expectations named for FEAT-05 (ASMP-21), since XBR-35 requires the domain to change only the address, never the experience.

**Data sensitivity / privacy:** The domain name and verification state carry no personal data -- a domain name is public information (feature-dependency-map.md, Data Sensitivity: None). The resulting confirmation email carries the same personal-data classification as every Notification record (recipient identity and message content, GDPR-class), per the dependency map's Notification entity definition; it is addressed to Nadia only.

**Compliance flags:** N/A beyond the general GDPR-class handling that already applies to the confirmation Notification -- no domain-specific compliance regime is named for this feature in assumptions-constraints.md, and ASMP-23's privacy posture (strict client isolation, operator access read-only and visible to the freelancer) continues to hold at the custom domain exactly as XBR-09 and XBR-31 require.

## Non-Goals

- **More than one domain, or per-team-member subdomains** -- Excluded per the Validation & Limits field ("one custom domain per freelancer account," product-features.md) and SC-01: Clientroom has no internal-staff seat model, so there is no second user whose work would need a distinct subdomain.
- **Dana adding, changing, or removing a domain** -- Excluded per SC-04: support sessions (FEAT-31) are strictly read-only and logged; Dana can only view the verification status, never act on it.
- **Ongoing monitoring or alerting for a domain that later breaks after going live** -- Excluded per the States field's own framing: this feature is "a one-time configuration step, not an ongoing runtime dependency for either party's session" (product-features.md, States), so continuous post-verification health monitoring is not part of this feature's specification surface.
- **A public directory or discovery surface listing freelancers' custom domains** -- Excluded per SC-03: portal access is never public or anonymous, and the only public-facing surface adjacent to branding is the referral mark's product page (FEAT-33), which never exposes portal content; a custom domain creates no new public discovery surface.
