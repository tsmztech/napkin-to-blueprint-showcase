---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-001
spec_name: Export Household Data
spec_slug: export-household-data
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Export Household Data

## Overview

**Name:** Export Household Data
**ID:** FEAT-18.SPEC-001
**Type:** Screen
**Purpose:** Maya requests a complete, readable copy of the household's data and downloads it once it is ready.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Requesting an export of the household's plans, ratings, lists, dietary rules, and settings
- Showing the export rate-limit state and the last export's status and download link, when one exists
- Showing progress while an export compiles and surfacing a completed download or a reported failure

**Non-Goals:**
- Compiling the export file itself -- owned by FEAT-18.SPEC-006 (Export Generation Processing), which this screen triggers and whose progress and outcomes it displays
- Choosing what the export contains -- the export always covers the complete household record set; scope-boundaries.md establishes no partial-export capability, so no selection controls exist on this screen
- Sending the export-ready confirmation -- owned by FEAT-18.SPEC-013 (Export Ready Notification) and delivered by FEAT-18.SPEC-012 (Transactional Email); this screen only reflects the same ready state in-app
- Deleting or removing household data -- owned by FEAT-18.SPEC-002 (Remove Member Profile) and FEAT-18.SPEC-003 (Delete Household)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-004 (My Account) | Maya taps "Export household data" in the organiser-only Household Data & Deletion section | None -- screen loads the household's current export state |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Maya taps "Request a data export" from the account settings area (feature-dependency-map.md, Navigation Connections) | None -- screen loads the household's current export state |
| FEAT-18.SPEC-013 (Export Ready Notification) | Maya taps "Download export" on the export-ready notification | None -- screen loads the household's current (Ready) export state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Request an export and download a ready export | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable from any navigation available to Sam; a direct attempt shows "Only the household organiser can export household data." and returns him to FEAT-18.SPEC-004 |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable from any navigation available to this login; a direct attempt shows "Only the household organiser can export household data." and returns to the older-kid login's landing area |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, a non-organiser lands on FEAT-18.SPEC-004, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress export request exists to preserve, since a request is submitted and confirmed in a single action |

## Layout and Content

**Header:** Screen title "Export Household Data" with a back arrow (returns to the entry source).

**Body:** A single content column.
- An explanatory line: "Download a complete copy of your household's plans, lists, ratings, and settings."
- A status card reflecting the current export state (see States): a "Request Export" button when no export is in progress and the household is under the rate limit; a progress indicator with the label "Compiling your export..." while one is generating; a "Download Export" button with the file's ready date when the latest export is ready; a rate-limit notice when the household is currently blocked from requesting a new export.
- A "Previous Exports" list below the status card, showing up to the household's most recent completed exports (ready date and a Download link for each), or the line "No exports yet" when none exist.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; the status card and Previous Exports list stack vertically.
- **Medium size class and above:** Content column capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition back |
| Request Export button | Tap | Validates the export rate limit via FEAT-18.SPEC-010, then triggers FEAT-18.SPEC-006 (Export Generation Processing) | Status card switches to the compiling/progress state | Progress indicator with the label "Compiling your export..." |
| Request Export button (rate-limited) | Tap | No action -- button is disabled while the household is under the limit | None | Rate-limit notice text remains visible |
| Download Export button | Tap | Retrieves the completed export file for download | None -- the screen does not change state | The device's standard file-download experience begins |
| Previous Exports "Download" link | Tap | Retrieves the selected prior export file for download | None | The device's standard file-download experience begins |

### Accessibility Notes

- **Focus order:** Back arrow -> explanatory line -> status card's primary action (Request Export or Download Export) -> Previous Exports list entries in ready-date order (newest first).
- **Progress announcements:** When the status card enters the compiling state, "Compiling your export" is announced to assistive technology; when it becomes ready, "Your export is ready to download" is announced.
- **Keyboard alternatives:** Every action on this screen (request, download) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Status card and Previous Exports area show loading placeholders | Screen first opens, before the initial fetch of the household's export state and previous exports completes | Fetch succeeds (-> No export yet, Ready to request, Compiling, Ready, or Rate-limited, whichever matches) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load your export history. Try again." with a Retry button; Previous Exports list and the status card's action are hidden until data loads | The initial fetch of export state and previous exports fails | Maya taps Retry (re-fetches) or the back arrow (returns to the entry source) |
| No export yet | Status card shows "Request Export" button; Previous Exports shows "No exports yet" | Initial fetch succeeds and the household has never requested an export | Maya requests an export |
| Ready to request | Status card shows "Request Export" button, enabled | No export currently compiling and household is under the export rate limit (FEAT-18.SPEC-010) | Maya taps Request Export |
| Compiling | Status card shows a progress indicator labeled "Compiling your export..." | Export request accepted by FEAT-18.SPEC-006 | Export completes (ready) or fails (retries exhausted) |
| Ready | Status card shows "Download Export" with the ready date | FEAT-18.SPEC-006 completes successfully | Maya downloads the file (screen state persists as Ready -- downloading does not consume the export) |
| Rate-limited | Status card shows a disabled Request Export control with the notice: "You've reached this period's export limit. You can request another export once the limit resets." | The household's export count for the current rate-limit window is at or above the limit (FEAT-18.SPEC-010) | The rate-limit window resets |
| Error | Status card shows an error banner: "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support." with a link to FEAT-18.SPEC-005 (Contact Support) | FEAT-18.SPEC-006 reports retries exhausted | Maya requests a new export (once the rate limit allows) |
| Offline/Degraded | Banner "You're offline -- your export request will be sent when you reconnect." at top; Request Export remains tappable and queues the request locally | Connectivity lost while this screen is open | Connectivity restored -- the queued request submits automatically and the screen shows the Compiling state |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules). See that spec for the export rate-limit condition and its exact denied behavior.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | Entry source (FEAT-18.SPEC-004 or FEAT-14.SPEC-003) | FEAT-14 when entered from there |
| "contact support" link (Error state) | FEAT-18.SPEC-005 (Contact Support) | -- |

## Data Model

**Creates:** None directly -- Request Export triggers FEAT-18.SPEC-006, which creates the export file record.
**Reads:** Household -- all fields, for the export summary; the household's prior export records (ready date, download reference) for the Previous Exports list.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Export requests are rate-limited per FEAT-18.SPEC-010 -- the Request Export control is disabled and the rate-limit notice shown whenever the household is at its limit.
- An export always covers the complete household record set at the moment of the request; it is never partial or filtered (product-features.md, Data Notes).
- Only Maya (Organiser) can reach this screen, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).

## Edge Cases

- **Maya taps Request Export twice in quick succession** -- The second tap is ignored while the first request is in flight (button enters a disabled, progress-indicating state on the first tap).
- **Maya navigates away while an export is compiling and returns later** -- The screen re-fetches the export's current status and shows whichever state (Compiling, Ready, or Error) matches that status; the request is not resubmitted.
- **The export becomes ready while Maya is offline** -- The Ready state and download become available once connectivity returns and the screen refreshes; FEAT-18.SPEC-013 delivers the confirmation independently of this screen being open.
- **Maya requests a download of a previous export whose file has since expired from storage** -- The download attempt shows "This export is no longer available. Request a new one." and the Previous Exports entry is marked unavailable rather than removed, preserving the record of when it was generated.
- **Household data changes while an export is compiling** -- The in-progress export reflects the household's data as of the moment the request was accepted; changes made after that moment appear only in a subsequent export. No concurrent-edit conflict entry applies here, since this screen only reads the household summary and never writes to a shared entity that another member could contend for.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Export rate-limit rule governs the Request Export control |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can reach this screen |
| FEAT-18.SPEC-006 (Export Generation Processing) | Triggers (outbound) | Request Export starts export compilation |
| FEAT-18.SPEC-013 (Export Ready Notification) | Affects (outbound) | The notification's CTA deep-links back to this screen |
| FEAT-18.SPEC-004 (My Account) | Navigation (inbound) | Organiser-only entry point into this screen |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (inbound) | Alternate entry point from account settings |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| data_export_requested | rate_limit_state (under_limit) | Maya taps Request Export and the request is accepted | N/A -- no Stage 2 success metric measures export usage; retained so the export path's actual use is observable, per feature-overview.md's Rationale on export as a trust mechanism |
| data_export_rate_limited | current_window_count | Maya attempts to request an export while at the limit | N/A -- no Stage 2 success metric measures rate-limit friction; retained to observe whether the limit is ever a real obstacle for households |
| data_export_downloaded | export_age_days | Maya taps a Download control (current or previous export) | N/A -- no Stage 2 success metric measures export completion; retained to distinguish a requested export from one actually retrieved |

## Acceptance Criteria

**FEAT-18.SPEC-001-AC-01:** Given Maya is on the Export Household Data screen with no export in progress and under the rate limit, when she taps Request Export, then the status card shows "Compiling your export..." and FEAT-18.SPEC-006 begins.

**FEAT-18.SPEC-001-AC-02:** Given Maya's export completes successfully, when she returns to this screen, then the status card shows "Download Export" with the ready date, and tapping it downloads the file.

**FEAT-18.SPEC-001-AC-03:** Given the household is at its export rate limit, when Maya opens this screen, then the Request Export control is disabled and shows "You've reached this period's export limit. You can request another export once the limit resets."

**FEAT-18.SPEC-001-AC-04:** Given Maya's export generation exhausts its retries, when she views this screen, then she sees "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support." with a link to FEAT-18.SPEC-005.

**FEAT-18.SPEC-001-AC-05:** Given Maya loses connectivity and taps Request Export, when she is offline, then the banner "You're offline -- your export request will be sent when you reconnect." appears and the request queues locally.

**FEAT-18.SPEC-001-AC-06:** Given connectivity returns while a request is queued, when the queued request submits automatically, then the status card shows the Compiling state without Maya re-tapping Request Export.

**FEAT-18.SPEC-001-AC-07:** Given Sam attempts to reach this screen directly, when the screen loads, then he sees "Only the household organiser can export household data." and is returned to FEAT-18.SPEC-004.

**FEAT-18.SPEC-001-AC-08:** Given an unauthenticated visitor attempts to reach this screen, when the screen loads, then they are redirected to the sign-in screen.

**FEAT-18.SPEC-001-AC-09:** Given Maya's session expires while she is on this screen, when she next interacts with it, then a dialog reads "Your session has expired. Sign in to continue." and no export request was made.

**FEAT-18.SPEC-001-AC-10:** Given Maya's household has three previous completed exports, when she opens this screen, then the Previous Exports list shows all three with their ready dates and working Download links.

**FEAT-18.SPEC-001-AC-11:** Given a previous export's file has expired from storage, when Maya taps its Download link, then she sees "This export is no longer available. Request a new one." and the entry stays listed as unavailable.

**FEAT-18.SPEC-001-AC-12:** Given Maya opens this screen, when the initial fetch of her export state and previous exports is in progress, then the status card and Previous Exports area show loading placeholders instead of any action button.

**FEAT-18.SPEC-001-AC-13:** Given the initial fetch of export state and previous exports fails, when Maya views this screen, then she sees "We couldn't load your export history. Try again." with a Retry button, and tapping Retry either loads the screen normally or shows the same error again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 5 | 5 |
| States | 9 (loading, load error, no export yet, ready to request, compiling, ready, rate-limited, error, offline) | 9 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
