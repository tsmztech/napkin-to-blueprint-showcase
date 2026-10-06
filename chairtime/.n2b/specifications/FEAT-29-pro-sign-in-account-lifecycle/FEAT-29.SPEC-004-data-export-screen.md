---
document_type: spec
spec_type: screen
spec_id: FEAT-29.SPEC-004
spec_name: Data Export Screen
spec_slug: data-export-screen
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Data Export Screen

## Overview

**Name:** Data Export Screen
**ID:** FEAT-29.SPEC-004
**Type:** Screen
**Purpose:** Talia requests and downloads a spreadsheet-friendly file of her own clients and booking history.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Requesting a data export and showing generation progress
- Downloading the generated file once ready
- Explaining exactly what the export covers

**Non-Goals:**
- Assembling the export file's contents -- owned by FEAT-29.SPEC-007 (Data Export Generation); this screen only requests it and surfaces its state
- Defining the export's scope (which fields, which entities) -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this screen references for its explanatory copy rather than redefining
- Importing data from any source -- excluded per scope-boundaries.md SC-09: the product has no import path for prior-tool data; this screen exports outward only
- Exporting the Pro's private client notes -- excluded per the Feature Breakdown Brief's Data Notes disposition: private_note is Pro-only working notes, not a client or booking record proper, and is deliberately out of the export's scope

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Download my data" | None |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Request and download the export | -- |
| The Client (Riley) | No | No | Clients never reach a Pro settings screen |
| Platform Operator (Support) | No | No | Support's view-only access to account status does not extend to the Pro's own data export; support never sees or triggers a Pro's export |
| Unauthenticated | No | No | Redirected to FEAT-29.SPEC-001 (Sign-In Screen) per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress export request already submitted continues generating server-side and remains available on the next visit; the screen itself must be reopened after re-authentication |

## Layout and Content

**Header:** Screen title "Download my data" with a back arrow (returns to FEAT-29.SPEC-003).

**Body:**
- On open, the screen briefly checks for an existing or in-progress export before showing any action (see States: Loading)
- Explanatory text: "This includes your clients (name, phone, email) and booking history (appointments, deposits, and outcomes) as a spreadsheet-friendly file. It does not include your private client notes."
- A single primary action button "Request export"
- Below the button (once a request has been made), progress or download state per the States section

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width text and button.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Screen closes | Standard navigation transition |
| "Request export" button | Tap | Triggers FEAT-29.SPEC-007 (Data Export Generation) | Button replaced by an in-progress indicator | "Preparing your file..." shown |
| "Download" button (once ready) | Tap | Downloads the generated file to the Pro's device | None -- file download begins | Standard file-download feedback |
| "Try again" button (on generation failure) | Tap | Re-triggers FEAT-29.SPEC-007's export generation | Returns to Generating state | "Preparing your file..." shown |
| "Try again" button (on status-fetch failure) | Tap | Re-fetches current export status from FEAT-29.SPEC-007 | Returns to Loading state | "Checking for an in-progress export..." shown |

### Accessibility Notes

- **Focus order:** Back arrow -> (Loading has no actionable control besides the back arrow) -> "Request export" (or "Download" / "Try again", whichever is current) .
- **Progress announcements:** The transition from Loading to the resolved state (Ready to request, Generating, or Ready to download), from "Request export" to the in-progress indicator, and from in-progress to "Download" or the failure message, is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Placeholder content with the text "Checking for an in-progress export..." | Screen first opens (every visit, including a return after navigating away) | The status fetch from FEAT-29.SPEC-007 resolves to Generating (an export is already in flight), Ready to download (a completed export exists and has not been superseded), Ready to request (no export exists), or Error (the fetch itself fails) |
| Ready to request (default) | "Request export" button enabled | Loading resolves with no existing export | Pro taps "Request export" |
| Generating | In-progress indicator with the text "Preparing your file... this can take a few minutes for a large amount of history." | Pro taps "Request export"; or Loading resolves with an export already in flight | FEAT-29.SPEC-007 completes (success or failure) |
| Ready to download | "Download" button shown with the file's generation timestamp | Export generation completes successfully; or Loading resolves with a completed export already available | Pro downloads the file, or requests a new export (replacing the previous one) |
| Error | Banner "Couldn't prepare your export. Try again." with a "Try again" action (export generation failure); or banner "Couldn't check your export status. Try again." with a "Try again" action (status-fetch failure) | Export generation fails; or the Loading status fetch fails | Pro taps "Try again" (retries the operation that failed -- generation or the status fetch) |
| Offline/Degraded | Existing "Ready to download" state (if reached before disconnecting) remains available for download if already fully downloaded locally; "Request export" and "Try again" are disabled with the inline note "Requires a live connection" | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Export scope (which fields and entities are included) is governed by FEAT-29.SPEC-013 (Account Closure & Retention Rules). This screen has no user-input fields to validate -- it is a single-action request/download flow.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | -- |

## Data Model

**Creates:** None persisted by this screen -- the export file itself is a derived, on-request artifact created by FEAT-29.SPEC-007.
**Reads:** Client, Booking, Deposit Transaction (via FEAT-29.SPEC-007) -- scoped to the Pro's own records only, per the dependency map's Client/Booking/Deposit Transaction relationships (each belongs to exactly one Pro Account). Current export status (via FEAT-29.SPEC-007), read once on every screen open to resolve the Loading state.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The export covers only the Pro's own clients and bookings (FEAT-29.SPEC-013) -- structured fields of Client, Booking, and Deposit Transaction, excluding Client.private_note.
- Requesting a new export while a previous file exists supersedes it -- the screen always reflects only the most recently completed export.
- Export generation is non-blocking to the rest of the product: the Pro can navigate away while generation runs and return later to find it ready (FEAT-29.SPEC-007).
- The screen never assumes no export exists: every open (first visit or a return) checks current export status with FEAT-29.SPEC-007 before showing "Request export", so an in-progress or completed export from an earlier visit is always reflected rather than overwritten by a fresh default.

## Edge Cases

- **Export generation is still running when the Pro checks back after navigating away** -- Reopening the screen enters the Loading state, which fetches current export status from FEAT-29.SPEC-007 and finds the same in-progress request; the screen enters the Generating state for that request and picks it up; no duplicate request is started.
- **Pro requests a second export while the first is still generating** -- The second request is ignored while one is in flight; the screen stays in the Generating state for the original request.
- **Pro has no bookings or clients yet** -- The export still generates successfully, producing a file with headers only and no data rows; the Download state is reached normally, with no special empty-state messaging beyond the file itself being empty of rows.
- **Network failure during download (file already generated)** -- The browser's standard failed-download handling applies; the "Download" button remains available for a retry, since the generated file persists server-side until superseded by a new request.
- **The on-open status fetch itself fails (e.g., the Pro opens the screen while briefly offline or the read errors)** -- The screen shows the Error state with the message "Couldn't check your export status. Try again."; tapping "Try again" re-fetches the status rather than assuming any particular prior state.
- **No concurrent-edit conflict applies** -- This screen has no editable record; there is nothing here for another actor to have changed concurrently. A concurrent write to the Pro's own Client or Booking records elsewhere (e.g., a new booking arriving mid-generation) is reflected only in the next requested export, never retroactively into one already generating.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (inbound) | Entry point |
| FEAT-29.SPEC-007 (Data Export Generation) | Reads (outbound) | On every screen open, fetches current export status to resolve the Loading state |
| FEAT-29.SPEC-007 (Data Export Generation) | Triggers (outbound) | "Request export" starts assembly of the file |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Defines the export's scope |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| data_export_requested | -- | Pro taps "Request export" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |
| data_export_downloaded | file_generation_duration_bucket | Pro taps "Download" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |

## Acceptance Criteria

**FEAT-29.SPEC-004-AC-01:** Given Talia is on this screen, when she taps "Request export", then the screen shows "Preparing your file..." and FEAT-29.SPEC-007 begins assembling it.

**FEAT-29.SPEC-004-AC-02:** Given Talia's export finishes generating successfully, when the screen updates, then a "Download" button appears with the generation timestamp.

**FEAT-29.SPEC-004-AC-03:** Given Talia taps "Download", when the file is ready, then the file downloads to her device.

**FEAT-29.SPEC-004-AC-04:** Given Talia's export generation fails, when the failure is reported, then the banner "Couldn't prepare your export. Try again." appears with a "Try again" action.

**FEAT-29.SPEC-004-AC-05:** Given Talia has requested an export and navigates away before it finishes, when she returns to this screen, then it shows the same in-progress Generating state rather than starting a new request.

**FEAT-29.SPEC-004-AC-06:** Given Talia already has a completed export ready to download, when she taps "Request export" again, then a new export supersedes the old one and the screen returns to the Generating state.

**FEAT-29.SPEC-004-AC-07:** Given Talia has no bookings or clients yet, when she requests an export, then generation completes successfully and produces a file with no data rows.

**FEAT-29.SPEC-004-AC-08:** Given Talia's export is complete, when she inspects what it contains, then it never includes any client's private_note field.

**FEAT-29.SPEC-004-AC-09:** Given Talia loses connectivity while on this screen, when the connection drops, then "Request export" and "Try again" are disabled with the note "Requires a live connection".

**FEAT-29.SPEC-004-AC-10:** Given Talia taps the back arrow, when the tap registers, then she is navigated to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen).

**FEAT-29.SPEC-004-AC-11:** Given Talia opens this screen, when it first loads, then it shows "Checking for an in-progress export..." while fetching current export status from FEAT-29.SPEC-007, then resolves to Ready to request, Generating, or Ready to download based on what that status reports -- with no export request started merely by opening the screen.

**FEAT-29.SPEC-004-AC-12:** Given the export-status fetch fails when Talia opens this screen, when the failure is reported, then the banner "Couldn't check your export status. Try again." appears with a "Try again" action, and tapping it re-fetches the status.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loading, generating, ready to download, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
