---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-005
spec_name: Recognizable & Branded Email Presentation Rules
spec_slug: recognizable-branded-email-presentation-rules
parent_feature: FEAT-14
parent_feature_name: Notifications (Email)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 21
acceptance_criteria_count: 19
---

# Logic/Rule Spec: Recognizable & Branded Email Presentation Rules

## Overview

**Name:** Recognizable & Branded Email Presentation Rules
**ID:** FEAT-14.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs sender name, subject line, branding, and plain, consistent formatting so every email is identifiable as coming from the freelancer's practice and reads as trustworthy, not spam.
**Parent Feature:** FEAT-14 -- Notifications (Email)
**Governed Entity:** Branding Profile (as applied to email presentation; also draws on Freelancer Account's business_name for sender-name composition)

## Scope and Non-Goals

**In Scope:**
- The sender-name composition rule applied to every notification
- The subject-line context requirement applied to every notification
- Where and how the freelancer's Branding Profile (logo, brand colour) is applied to a notification's visual presentation, and the neutral-default fallback
- Where and how the referral mark is included on a client-facing email, without overriding branding (XBR-32)
- The plain, consistent formatting baseline every notification's content must follow

**Non-Goals:**
- Capturing or validating the Branding Profile's own fields (logo size/format limits, brand-colour legibility adjustment) -- owned by FEAT-19 (Freelancer Branding); this spec consumes an already-valid Branding Profile and defines only how it is applied to email presentation.
- The exact subject and body wording of any single notification type -- owned by each triggering feature's own Notification spec (e.g., FEAT-02.SPEC-011), which must satisfy this spec's rules but authors its own content.
- Deciding who receives which notification type -- owned by FEAT-14.SPEC-004; this spec governs how an already-entitled email looks and reads, not who it goes to.
- The referral mark's own content and destination -- owned by FEAT-33 (Portal Referral Attribution); this spec governs only that the mark is included and does not override branding.

## Governed Entity

**Entity:** Branding Profile
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| logo | image (optional) | Freelancer's logo, within size/format limits set by FEAT-19; a neutral default applies when unset |
| brand_colour | colour (optional) | Freelancer's primary brand colour, legibility-adjusted by FEAT-19; a neutral default applies when unset |

**Referenced (not governed) for composition:** Freelancer Account -- business_name, name (sender-name fallback); Project -- project_name, Invoice -- invoice_number, Milestone -- name, Client -- client_name (subject-line context identifiers, whichever is relevant to the notification_type).

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | Applies these rules during its composition step, on every notification, before handing the email to FEAT-14.SPEC-001 |
| FEAT-02.SPEC-011, FEAT-03.SPEC-006, FEAT-03.SPEC-007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004 | Each feature's own Notification spec | Each authors its own subject/body content within the boundaries this spec sets (e.g., the sender-name and branding rules), rather than re-deriving them independently |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| logo | No new validation here -- FEAT-19 already enforces size and format limits before a Branding Profile is saved; this spec only defines the fallback when the field is empty (neutral default) | When absent | At composition time (FEAT-14.SPEC-002) | -- | -- |
| brand_colour | No new validation here -- FEAT-19 already performs legibility adjustment before a Branding Profile is saved; this spec only defines the fallback when the field is empty (neutral default) | When absent | At composition time | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Sender-name composition | Freelancer Account -- business_name, name | The sender display name is business_name; if business_name is not yet set, it falls back to the freelancer's account name, so no email is ever sent with a blank sender identity | Internal invariant -- never user-facing; a Freelancer Account without at least a name cannot exist (FEAT-20 requires it at sign-up) |
| Subject-line context requirement | notification_type, and the record identifier relevant to that type (Project.project_name, Invoice.invoice_number, Milestone.name, or Client.client_name) | Every subject line names at least one specific record identifier matched to the recipient's own context: a client-facing subject (to Owen or Priya) references the project, invoice, or milestone; a freelancer-facing copy (to Nadia) references the client company name, since she manages several clients at once | Internal invariant, enforced by each triggering feature's own Notification spec review, not by a runtime check |
| Branding applies to client-facing recipients only | Branding Profile -- logo, brand_colour; recipient role | Every notification addressed to a client contact (Owen or Priya) applies the freelancer's Branding Profile to its visual presentation, falling back to a neutral default when unset (XBR-31); a notification addressed to Nadia herself does not receive this branding treatment, since branding exists to build trust with an external audience, not to decorate her own inbox | -- |
| Referral mark applies to client-facing recipients only | recipient role | Every notification addressed to a client contact includes the referral mark, without overriding branding (XBR-32); a notification addressed to Nadia herself never carries the referral mark, since it exists to reach potential new freelancers through clients and peers, not her own account | -- |
| Greeting consistency | recipient -- name | Every notification body opens with "Hi {first_name}," using the recipient's own name -- never a generic, unaddressed, or promotional-style opening | Internal invariant -- a missing recipient name is impossible, since name is required at Client Contact creation (FEAT-18) and at Freelancer Account creation (FEAT-20) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read the Branding Profile for email composition (this spec's own action) | System, via FEAT-14.SPEC-002 | Always, for every notification addressed to a client contact | -- |
| Edit the Branding Profile (logo, brand_colour) | Nadia (Freelancer) | Always, her own account -- owned and enforced by FEAT-19, referenced here only for completeness | -- |
| Edit the Branding Profile | Owen, Priya, Dana | Never | Not offered anywhere in their access; Dana's View-only account access (Access Matrix) includes no branding controls |
| Preview how a branded email will render | Nadia (Freelancer) | Full, via each triggering feature's own preview (e.g., FEAT-02.SPEC-002 Proposal Preview) applying this spec's rules -- not a screen this spec itself owns | -- |
| Preview how a branded email will render | Owen, Priya, Dana | Never, as a dedicated preview action -- they see the rendered result only by receiving the actual email (Owen, Priya) or viewing the delivery-warning summary (Dana, FEAT-14.SPEC-006) | No preview action exists for these roles; this is not a denial of an otherwise-available action, since no such standalone preview surface exists for them in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Sender display name | business_name; falls back to Freelancer Account.name if business_name is unset | On every notification send | No -- not a per-email choice; Nadia changes her business_name in FEAT-21 (Settings), which then applies to every future send |
| Header branding (logo, brand_colour) | Branding Profile.logo / brand_colour if set; otherwise a clean neutral default (owned by FEAT-19) | On every notification addressed to a client contact | No -- not a per-email choice; Nadia changes her Branding Profile in FEAT-19, which then applies to every future client-facing send |
| Referral mark presence | Always included on every client-facing notification | On every notification addressed to a client contact | No -- per XBR-32, the referral mark appears on every client-facing email on every plan in MVP; there is no opt-out |
| Subject-line context identifier | Derived from the notification_type's registry entry (FEAT-14.SPEC-004) and the triggering event's own data (project, invoice, milestone, or client reference) | On every notification send | No -- system-derived, not user-chosen |

## Business Rules

- Every notification's sender name identifies the freelancer's business, never a generic or disguised origin, giving the recipient's inbox a recognizable, consistent source across every email they receive from that freelancer (feature-overview.md, Key Capabilities: "every email names the freelancer and the project in its sender name and subject").
- Every subject line names the specific record it concerns, so a recipient managing several concurrent client relationships or freelancer engagements (a client contact who works with more than one freelancer, or Nadia managing several clients) can tell emails apart at a glance without opening each one.
- The freelancer's branding (logo, brand colour) and the referral mark apply to every client-facing email consistently, falling back to a clean neutral default when branding is unset, and the referral mark never overrides the branding (XBR-31, XBR-32).
- Formatting stays plain and consistent across every notification type -- a simple header, a personal greeting, clear body text, one primary call-to-action button, and a plain sign-off -- deliberately avoiding promotional styling, since client-facing emails landing in spam is a documented, recurring failure mode this feature exists to avoid [RESEARCH-INFORMED: spam-folder delivery of client emails is a recurring complaint for HoneyBook and SuiteDash, and message reliability is a category-wide quality bar (G2, Capterra, Trustpilot, HIGH)].
- Branding and the referral mark are deliberately withheld from notifications addressed to Nadia herself: they exist to build external trust and drive growth through clients and peers (XBR-32), not to decorate her own working inbox, so her own copies stay plain and functional.

## Edge Cases

- **The freelancer has not set a Branding Profile at all** -- Every client-facing email falls back cleanly to the neutral default logo and colour, consistent with FEAT-02.SPEC-011's own stated fallback behavior; no email is ever sent with a missing or broken branding element.
- **The freelancer's business_name is not yet set (only her personal account name exists)** -- Sender name and any subject slot that would otherwise use business_name use her personal account name instead, so no email is ever sent with a blank sender or subject slot.
- **A notification is addressed to Nadia herself (e.g., proposal_accepted_confirmation's copy to Nadia)** -- Branding and the referral mark are not applied to her copy; only the sender-name, subject-context, and greeting rules apply, since those support recognizability for her own record-keeping, not external trust-building.
- **The freelancer's brand colour is adjusted for legibility by FEAT-19 after this spec last read it** -- This spec always reads the current, already-legibility-adjusted value at composition time; it never caches or re-derives contrast itself, so a legibility fix Nadia makes in FEAT-19 takes effect on the very next notification sent.
- **A single triggering event addresses more than one role in the same occurrence (e.g., invoice_issued to both Owen and Nadia)** -- Each recipient's copy is composed independently against this spec's rules for their own audience type: Owen's copy is client-facing (branded, with the referral mark); Nadia's copy is not (unbranded, no referral mark), even though both originate from the same triggering event.

## Acceptance Criteria

**FEAT-14.SPEC-005-AC-01:** Given the freelancer has set both a logo and a brand colour, when a client-facing notification is composed, then the email's presentation applies that logo and colour.

**FEAT-14.SPEC-005-AC-02:** Given the freelancer has not set a Branding Profile, when a client-facing notification is composed, then the email falls back to the neutral default logo and colour.

**FEAT-14.SPEC-005-AC-03:** Given the freelancer's brand colour was recently adjusted for legibility by FEAT-19, when the next notification is composed, then it uses the current, already-adjusted colour value.

**FEAT-14.SPEC-005-AC-04:** Given the freelancer has set a business_name, when any notification is composed, then the sender display name is that business_name.

**FEAT-14.SPEC-005-AC-05:** Given the freelancer has not yet set a business_name, when any notification is composed, then the sender display name falls back to her personal account name.

**FEAT-14.SPEC-005-AC-06:** Given a notification is addressed to Owen about a specific project, when its subject line is composed, then it names the project (or the specific invoice/milestone identifier relevant to the event).

**FEAT-14.SPEC-005-AC-07:** Given a notification is addressed to Nadia about a specific client's action, when its subject line is composed, then it names the client company, since Nadia manages several clients.

**FEAT-14.SPEC-005-AC-08:** Given any notification body is composed, when it opens, then it greets the recipient by name ("Hi {first_name},"), never a generic or unaddressed opening.

**FEAT-14.SPEC-005-AC-09:** Given a notification is addressed to Owen or Priya, when it is composed, then it includes the referral mark without overriding the freelancer's branding.

**FEAT-14.SPEC-005-AC-10:** Given a notification is addressed to Nadia herself, when it is composed, then it does not carry the referral mark and does not apply client-facing branding.

**FEAT-14.SPEC-005-AC-11:** Given any notification type, when its content is composed, then it follows the plain, consistent formatting baseline (simple header, personal greeting, clear body, one primary CTA, plain sign-off) rather than promotional styling.

**FEAT-14.SPEC-005-AC-12:** Given Nadia edits her Branding Profile in FEAT-19, when she saves the change, then it is not offered as a per-email override anywhere -- it applies uniformly to every future client-facing send.

**FEAT-14.SPEC-005-AC-13:** Given Nadia looks for a way to turn off the referral mark on client-facing emails, when she checks her settings, then no such control exists, per XBR-32.

**FEAT-14.SPEC-005-AC-14:** Given Dana is inside a logged support session, when she looks for a way to preview a client-facing email's rendered branding, then no such preview action exists for her role.

**FEAT-14.SPEC-005-AC-15:** Given Owen or Priya attempt to edit the freelancer's Branding Profile, when they look for such a control, then none exists anywhere in their portal access.

**FEAT-14.SPEC-005-AC-16:** Given an invoice_issued event addresses both Owen and Nadia in the same occurrence, when each copy is composed, then Owen's copy is branded with the referral mark and Nadia's copy is not.

**FEAT-14.SPEC-005-AC-17:** Given a client has no Reviewer contact on record, when a notification entitling the Reviewer role would otherwise be composed for one, then no presentation rule is violated -- the rule set applies per actual recipient, not per role slot.

**FEAT-14.SPEC-005-AC-18:** Given the freelancer's logo file was accepted by FEAT-19's own size and format validation, when this spec composes an email, then it applies that logo without re-validating its size or format.

**FEAT-14.SPEC-005-AC-19:** Given a Freelancer Account exists (required at sign-up, FEAT-20), when any notification is composed, then a sender name and a recipient greeting name are always available -- neither is ever blank.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
