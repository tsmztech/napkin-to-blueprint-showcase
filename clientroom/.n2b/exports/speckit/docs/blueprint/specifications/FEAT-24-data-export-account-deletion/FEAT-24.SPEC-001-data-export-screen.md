---
document_type: spec
spec_type: screen
spec_id: FEAT-24.SPEC-001
spec_name: Data Export Screen
spec_slug: data-export-screen
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Data Export Screen

## Overview

**Name:** Data Export Screen
**ID:** FEAT-24.SPEC-001
**Type:** Screen
**Purpose:** Nadia requests a full export of all her own data, tracks the archive's progress, and downloads it once ready.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Requesting a full data export and showing its current status (Requested, Ready, Downloaded, Expired)
- Downloading the completed archive
- The path onward to closing the account (navigation to the Account Deletion Screen)

**Non-Goals:**
- Choosing which data types to include -- excluded per product-features.md's Key Capabilities, which name only a whole-account export; no per-category selection control exists on this screen.
- Aggregating the archive's contents or handling generation failure/retry -- owned by FEAT-24.SPEC-003 (Data Export Archive Generation); this screen only reflects that automation's state.
- Deciding who may reach this screen -- owned by FEAT-24.SPEC-007 (Export & Deletion Access Rules); this screen's Access and Visibility table below is consistent with, but does not redefine, that spec.
- Account deletion itself -- owned entirely by FEAT-24.SPEC-002 (Account Deletion Screen); this screen only offers the path there.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia taps "Close account" in the Settings navigation shell | None -- this is the feature's default entry point; the screen loads the current (or absent) Data Export Archive state for her account |
| FEAT-24.SPEC-008 (Export Ready Notification) | Nadia taps the email's "Download your export" CTA | None -- the screen loads the current Data Export Archive state for her account, showing the ready archive |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Request export, download archive, navigate to Account Deletion Screen | -- |
| Owen (Client Primary Contact) | No | No | No navigation path in the client portal reaches this screen; a direct link shows the same out-of-scope explanation used elsewhere in the portal (XBR-09), never this screen's content |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no client-portal surface exists for this feature (feature-overview.md, Non-Goals) |
| Dana (Support Operator) | No | No | This screen is excluded entirely from every read-only support session (XBR-29; FEAT-24.SPEC-007); a direct link during a session shows "This isn't available during a support session," not a read-only view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-12 (Freelancer Financial Dashboard), not this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." No in-progress request is lost, since this screen holds no unsaved input -- after re-authentication the current archive status is reloaded exactly as it stood |

## Layout and Content

**Header:** Screen title "Export Your Data" with a back arrow (returns to FEAT-21.SPEC-001, Account Profile). To the left of the title, the shared Settings navigation shell (also used by FEAT-21.SPEC-001 through SPEC-004 and FEAT-19.SPEC-001) lists "Profile," "Notification Preferences," "Login & Security," "Business Details & Payment Terms," "Branding," and "Close account" at the bottom, visually separated; "Close account" is shown selected/active since this screen is what it routes to.

**Body:** A single-column content area below the header:
- Explanatory text: "Download a complete copy of your clients, projects, proposals, invoices, and activity trail."
- A status card showing the current archive's state (see States below): its requested date, current status, and, once Ready or Downloaded, a download action and the date the archive becomes unavailable.
- A "Request Export" action (button), whose label and enabled state depend on the current archive state.
- Below the status card, a visually separated section: "Want to close your account instead?" with a "Continue to close your account" link.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; the Settings navigation shell collapses to a top row of tabs, consistent with FEAT-21.SPEC-001's own responsive behavior.
- **Medium size class and above:** The Settings navigation shell sits to the left of the main content, consistent with FEAT-21.SPEC-001; the status card and actions remain single-column, capped at a consistent platform-wide content width and horizontally centered within the content area.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-21.SPEC-001 (Account Profile) | Screen closes | Standard navigation transition |
| "Request Export" button | Tap (Empty, Expired, Ready, or Downloaded state) | Triggers FEAT-24.SPEC-003 (Data Export Archive Generation) | Status card enters Requested/Generating | Button shows loading state; status card text updates to "Preparing your export -- this can take a few minutes for accounts with a lot of history." |
| "Request Export" button (while Requested/Generating) | Tap | No action -- debounced | None | Button remains disabled/loading |
| "Download" action (Ready or Downloaded state only) | Tap | Initiates delivery of the archive file via FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Archive transitions from Ready to Downloaded on first successful download (stays downloadable until it expires) | Standard file-download behavior begins; status card shows "Downloaded on {date}" once the transfer completes |
| "Continue to close your account" link | Tap | Navigate to FEAT-24.SPEC-002 (Account Deletion Screen) | Screen leaves Data Export Screen | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Settings navigation shell items -> explanatory text -> status card -> Request Export button -> Download action (when present) -> "Continue to close your account" link.
- **Status announcements:** When the status card's state changes (e.g., Requested/Generating -> Ready, or -> Error), the new status text is announced to assistive technology.
- **Download feedback:** The "Downloaded on {date}" update is announced once a download completes.
- **Keyboard alternatives:** Every action on this screen (Request Export, Download, Continue to close account, back arrow) is a standard activatable control reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no export requested) | Status card reads "You haven't requested an export yet." Request Export button enabled | Screen first opens with no prior or active archive for this account | Nadia taps Request Export |
| Requested/Generating | Status card shows a progress indicator and "Preparing your export -- this can take a few minutes for accounts with a lot of history." Request Export button disabled | FEAT-24.SPEC-003 begins processing the request | FEAT-24.SPEC-003 reports Ready or, after all retries, Error |
| Ready | Status card shows "Your export is ready." with a Download action and "Available until {expiry_date}." Request Export button re-enabled, labeled "Request New Export" | FEAT-24.SPEC-003 reports the archive Ready | Nadia downloads (-> Downloaded), the window elapses unfetched (-> Expired), or she requests a new export (replaces this archive) |
| Downloaded | Status card shows "Downloaded on {date}." with the Download action still available and "Available until {expiry_date}." Request Export button re-enabled, labeled "Request New Export" | Nadia completes a download while the archive is Ready | The window elapses (-> Expired) or she requests a new export |
| Expired | Status card shows "Your last export has expired. Request a new one to download your data again." Request Export button enabled, labeled "Request Export" | The download window elapses without a fresh request superseding it (FEAT-24.SPEC-003) | Nadia requests a new export |
| Error | Error banner: "We couldn't generate your export. Try again." Request Export button re-enabled | FEAT-24.SPEC-003 reports generation failure after exhausting its retries | Nadia requests a new export |
| Offline/Degraded | Banner: "You're offline. Reconnect to request or download your export." Request Export and Download actions disabled; the last-known status card content remains visible | Connectivity lost while this screen is open | Connectivity restored -- the current archive status is re-fetched and the screen returns to the state matching it |

## Validation Rules

N/A -- this screen has no user-entered fields. Its only actions (Request Export, Download, navigate onward) require no input validation; authorization for reaching the screen at all is governed by FEAT-24.SPEC-007 (Export & Deletion Access Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-21.SPEC-001 (Account Profile) | FEAT-21 (Settings & Account Management) |
| "Continue to close your account" link | FEAT-24.SPEC-002 (Account Deletion Screen) | -- |

## Data Model

**Creates:** None directly -- Request Export triggers FEAT-24.SPEC-003, which creates the Data Export Archive record.
**Reads:** Data Export Archive -- status, requested-at, download link/window, displayed in the status card.
**Updates:** None directly -- FEAT-24.SPEC-003 advances the archive's lifecycle states; this screen only reflects them.
**Deletes:** None.

## Business Rules

- At most one active archive exists at a time (feature-overview.md, Entity-Lifecycle Coverage Matrix); requesting a new export while one is Ready, Downloaded, or Expired supersedes it, per FEAT-24.SPEC-003.
- Full data export only -- no per-category selection exists on this screen (product-features.md, Key Capabilities).
- FEAT-24.SPEC-007 confines this entire screen, and every action on it, to Nadia and her own account.

## Edge Cases

- **Nadia taps Request Export twice in quick succession** -- The second tap is ignored while the button is disabled during Requested/Generating (debounced).
- **Network failure while initiating a request** -- Error banner: "Could not start your export. Check your connection and try again." No archive record is created; the screen returns to its prior state (Empty, Expired, Ready, or Downloaded).
- **Nadia has this screen open in two browser tabs and requests an export in one** -- This screen is a snapshot, not live-updating; the second tab continues to show its last-loaded state until Nadia reloads or re-navigates to it, at which point it reflects the newly requested archive. No conflict dialog is shown, since the underlying Data Export Archive is a single-writer, non-shared entity (feature-dependency-map.md notes it as a single-feature entity with no Contention entry) and FEAT-24.SPEC-003 resolves the two requests deterministically (the later request supersedes).
- **The archive expires while this screen is open and idle** -- The status card does not update in real time; it shows Expired on the next load or manual refresh.
- **Nadia navigates away mid-generation and returns later** -- The screen re-fetches and displays whatever state FEAT-24.SPEC-003 has reached (Generating, Ready, or Error); nothing is lost, since generation continues independently of whether this screen is open.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-003 (Data Export Archive Generation) | Triggers (outbound) | Request Export starts this automation; its outcomes drive every state on this screen |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Governs who can reach and act on this screen |
| FEAT-24.SPEC-008 (Export Ready Notification) | References (inbound) | This notification's CTA deep-links back to this screen |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | "Close account" navigation item routes here as the feature's default entry |
| FEAT-24.SPEC-002 (Account Deletion Screen) | Navigation (outbound) | "Continue to close your account" link |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | References (outbound) | Underlying delivery capability the Download action uses |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| data_export_requested | entry state (empty / expired / ready / downloaded) | Nadia taps Request Export | N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; this event is retained because product-features.md's Signals field names it explicitly as this feature's defined signal, kept observable even without a Stage 2 metric consuming it |
| data_export_download_initiated | archive age (days since requested) | Nadia taps Download | N/A -- same reason as above; retained for observability of whether Nadia actually retrieves what she requested |

## Acceptance Criteria

**FEAT-24.SPEC-001-AC-01:** Given Nadia is on FEAT-21.SPEC-001 (Account Profile), when she taps "Close account," then she lands on this screen showing her account's current export status.

**FEAT-24.SPEC-001-AC-02:** Given Nadia is on the Data Export Screen with no prior export, when she taps "Request Export," then FEAT-24.SPEC-003 begins and the status card shows "Preparing your export -- this can take a few minutes for accounts with a lot of history."

**FEAT-24.SPEC-001-AC-03:** Given Nadia's archive is in the Requested/Generating state, when she looks at the Request Export button, then it is disabled and a second tap has no effect.

**FEAT-24.SPEC-001-AC-04:** Given FEAT-24.SPEC-003 reports the archive Ready, when Nadia views this screen, then the status card shows "Your export is ready." with a Download action and "Available until {expiry_date}."

**FEAT-24.SPEC-001-AC-05:** Given Nadia's archive is Ready, when she taps Download, then the file transfer begins through FEAT-16.SPEC-007 and, once it completes, the status card shows "Downloaded on {date}."

**FEAT-24.SPEC-001-AC-06:** Given Nadia's archive has passed its download window unfetched, when she views this screen, then the status card shows "Your last export has expired. Request a new one to download your data again." and the Download action is no longer shown.

**FEAT-24.SPEC-001-AC-07:** Given FEAT-24.SPEC-003 reports generation failure after exhausting its retries, when Nadia views this screen, then she sees the error banner "We couldn't generate your export. Try again." and the Request Export button is enabled.

**FEAT-24.SPEC-001-AC-08:** Given Nadia is on the Data Export Screen, when she taps "Continue to close your account," then she is navigated to FEAT-24.SPEC-002 (Account Deletion Screen).

**FEAT-24.SPEC-001-AC-09:** Given Nadia loses connectivity while this screen is open, when the connection drops, then the banner "You're offline. Reconnect to request or download your export." appears and both Request Export and Download are disabled.

**FEAT-24.SPEC-001-AC-10:** Given Owen (Client Primary Contact) has no navigation path to this screen, when he follows any link in the client portal, then he never reaches it.

**FEAT-24.SPEC-001-AC-11:** Given Dana (Support Operator) is in an active read-only support session, when she attempts to open this screen directly, then she sees "This isn't available during a support session," not a read-only view of Nadia's export.

**FEAT-24.SPEC-001-AC-12:** Given an unauthenticated visitor opens this screen's link, then they are redirected to sign-in and, after signing in, land on FEAT-12 (Freelancer Financial Dashboard), not this screen.

**FEAT-24.SPEC-001-AC-13:** Given Nadia's session expires while this screen is open, when she is shown the expired-session dialog and signs back in, then the screen reloads showing the current archive status exactly as it stood before expiry.

**FEAT-24.SPEC-001-AC-14:** Given Nadia's archive is already Ready, when she taps "Request New Export," then a fresh request supersedes the current archive per FEAT-24.SPEC-003, and the status card returns to Requested/Generating.

**FEAT-24.SPEC-001-AC-15:** Given Nadia has this screen open in two tabs and requests an export in one, when she switches to the other tab without reloading it, then that tab still shows its prior state until she reloads or re-navigates to it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (empty, requested/generating, ready, downloaded, expired, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
