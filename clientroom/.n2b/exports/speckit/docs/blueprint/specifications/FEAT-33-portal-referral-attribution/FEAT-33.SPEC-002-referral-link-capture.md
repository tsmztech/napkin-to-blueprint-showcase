---
document_type: spec
spec_type: automation
spec_id: FEAT-33.SPEC-002
spec_name: Referral Link Capture
spec_slug: referral-link-capture
parent_feature: FEAT-33
parent_feature_name: Portal Referral Attribution
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Referral Link Capture

## Overview

**Name:** Referral Link Capture
**ID:** FEAT-33.SPEC-002
**Type:** Automation
**Purpose:** Captures the referring freelancer's portal identifier the moment a visitor follows the "Made with Clientroom" mark, and carries that reference forward toward sign-up.
**Parent Feature:** FEAT-33 -- Portal Referral Attribution

## Scope and Non-Goals

**In Scope:**
- Capturing which Freelancer Account's portal page or email the mark was followed from
- Holding that reference for the length of the visit, scoped to the visitor's own session
- Routing the visitor to the Referral Landing Page (FEAT-33.SPEC-003) immediately after the click
- Carrying the captured reference forward into sign-up (FEAT-20.SPEC-001) if the visitor proceeds there, so FEAT-20.SPEC-004 can hand it off for recording
- Emitting `referral_mark_clicked` for every follow of the mark

**Non-Goals:**
- Creating the Referral Attribution record -- excluded per the dependency map's lifecycle line ("Created by FEAT-33 at sign-up"); this automation only captures and carries the reference, FEAT-33.SPEC-004 persists it once sign-up completes
- Rendering the mark itself -- owned by FEAT-33.SPEC-001; this automation begins only once the mark has already been followed
- Displaying the landing page's content -- owned by FEAT-33.SPEC-003; this automation only routes the visitor there
- Retaining the captured reference beyond the visit if the visitor never signs up -- excluded per the Non-Functional Notes' data-volume line ("volume tracks the sign-up rate, not portal traffic"); a click that never converts leaves no persisted record

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Visitor follows the mark on a client-facing portal page | FEAT-33.SPEC-001 (Referral Mark Display), rendered on FEAT-05 portal pages | Fires on every follow of the mark, regardless of the visitor's own sign-in state | The owning Freelancer Account reference for the portal page the mark rendered on |
| Visitor follows the mark in a client-facing email | FEAT-33.SPEC-001 (Referral Mark Display), rendered in FEAT-14 emails | Fires on every follow of the mark from an email | The owning Freelancer Account reference the email was composed for (FEAT-14.SPEC-001) |

## Processing Logic

1. Receive the click on the mark, along with the Freelancer Account reference of the surface it appeared on (the portal's owning account, or the email's addressed-for account).
2. Capture that reference as the "referring portal" for this visit, held in a session-scoped context tied to the visitor's browser session -- not yet a persisted record.
3. Route the visitor to the Referral Landing Page (FEAT-33.SPEC-003), passing the captured reference along in that same session context.
4. Emit `referral_mark_clicked`, noting which surface type it was followed from (portal page or email).
5. If the visitor later reaches sign-up (FEAT-20.SPEC-001), either directly from the landing page's "Sign up" action or by navigating there afterward within the same session, carry the same captured reference forward so it is available to FEAT-20.SPEC-004's hand-off.
6. If the captured reference's session context has expired before sign-up is reached (the visitor's session lifetime, per Business Rules, has elapsed), proceed to sign-up with no referring-portal reference, rather than attempting to recover or re-derive one.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|-----------------|--------------------|
| Capture succeeds | The mark is followed and the owning Freelancer Account reference is captured | A session-scoped reference is held for this visit; no persisted record yet | None -- the visitor is simply taken to the Referral Landing Page as expected | FEAT-33.SPEC-003 (Referral Landing Page), FEAT-20.SPEC-001 (Sign-Up & Account Creation) |
| Capture succeeds and carries through to sign-up | The visitor proceeds to sign-up within the reference's session lifetime | The reference is passed into FEAT-20.SPEC-001 and onward to FEAT-20.SPEC-004 | None -- invisible to the visitor; the sign-up form behaves identically with or without a captured reference | FEAT-20.SPEC-001, FEAT-20.SPEC-004, FEAT-33.SPEC-004 |
| Capture fails or the owning Freelancer Account no longer resolves (e.g., deleted, FEAT-24) | The mark's underlying account reference cannot be resolved at click time | No reference is captured for this visit | None -- the visitor still reaches the Referral Landing Page normally; the eventual sign-up simply carries no referring-portal reference | FEAT-33.SPEC-003, FEAT-33.SPEC-004 (records "unknown") |
| Captured reference expires before sign-up | The session-scoped reference's lifetime (platform parameter: `referral-capture-session-window`) elapses before the visitor reaches sign-up | The reference is discarded | None -- sign-up proceeds normally with no referring-portal reference, identical to a visitor who never followed a mark | FEAT-20.SPEC-001, FEAT-33.SPEC-004 (records "unknown") |

## Data Model

**Reads:** Freelancer Account -- reads only the account reference of the surface the mark was rendered on (the portal's owner, or the email's addressee-freelancer), to resolve which portal referred the visit. No other Freelancer Account fields are read.
**Creates:** None -- this automation holds a session-scoped reference only; it creates no persisted record. The persisted Referral Attribution record is created later, by FEAT-33.SPEC-004.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Capture is best-effort and non-blocking: a failure to resolve the referring account never prevents the visitor from reaching the Referral Landing Page or, later, signing up.
- The captured reference is held only for the length of the visit, scoped to the visitor's own session, and expires after platform parameter: `referral-capture-session-window` with no further activity -- it is never persisted independently of a completed sign-up.
- Only one referring-portal reference is held per visitor session at a time; following a second mark during the same session replaces the first (last-click-wins), since a sign-up can be attributed to at most one referring portal (dependency map: Referral Attribution's `referring_portal` is a single optional reference).
- This automation never re-derives or infers a referring portal from any other signal (browsing history, IP address, or similar) -- the reference is captured only from an explicit mark follow, per XBR-32's scope.

## Edge Cases

- **The mark's underlying Freelancer Account has since been deleted (FEAT-24) between the mark's earlier render and the visitor's click** -- Capture fails to resolve; the visitor still reaches the Referral Landing Page, and any later sign-up records the referring portal as unknown.
- **The visitor follows the mark, closes the browser, and returns later in a new, unrelated session** -- The earlier session-scoped reference is gone; the new session carries no referring-portal reference, and a subsequent sign-up in that new session records it as unknown.
- **The visitor follows a mark on Portal A, then later in the same session follows a mark on Portal B before signing up** -- The reference is overwritten: Portal B is what is captured and eventually attributed, per the last-click-wins rule.
- **The visitor follows the mark from an email that was addressed to a client contact but opened by someone else who forwarded it** -- Capture still resolves to the email's addressed-for Freelancer Account, since the reference is tied to the surface's own composition context, not to who personally clicks it.
- **Concurrent trigger firing (two different visitors follow marks on different portals at the same time)** -- Each fires its own independent capture, scoped to its own visitor session; neither affects the other.
- **Trigger fires while a previous run is in flight (the same visitor double-clicks the mark rapidly)** -- The second click's capture simply re-captures the same reference (or, if it was a different mark, overwrites per last-click-wins); there is no queued or conflicting state, since each capture is a single, immediate session write.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-33.SPEC-001 (Referral Mark Display) | Triggered by (inbound) | Following the rendered mark, on either a portal page or an email, fires this capture |
| FEAT-33.SPEC-003 (Referral Landing Page) | Affects (outbound) | Every successful click routes the visitor here immediately after capture |
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Affects (outbound) | The captured reference, when present, is carried forward into sign-up entry |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | References (outbound) | Supplies the referring-portal reference this hand-off later passes to FEAT-33.SPEC-004 |
| FEAT-33.SPEC-004 (Referral Attribution Recording) | Affects (outbound) | The reference this automation captures (or its absence) is what FEAT-33.SPEC-004 eventually records, via FEAT-20.SPEC-004's hand-off |

## Analytics and Success Signals

- **referral_mark_clicked** (source surface: portal page / email) -- supports success-metrics.md: "Growth Through Referral"
- **referral_capture_expired** (had reached the landing page: yes/no) -- N/A -- no Stage 2 metric measures capture expiry directly; retained so a silently dropped capture is observable rather than invisible

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
