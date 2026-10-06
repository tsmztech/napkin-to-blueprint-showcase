---
document_type: spec
spec_type: integration
spec_id: FEAT-10.SPEC-005
spec_name: Web Page Recipe Extraction
spec_slug: web-page-recipe-extraction
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Web Page Recipe Extraction

## Overview

**Name:** Web Page Recipe Extraction
**ID:** FEAT-10.SPEC-005
**Type:** Integration
**Purpose:** Reads a pasted web page through the product's web-page recipe extraction capability and returns ingredients, steps, and cook time, or reports that extraction failed.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Sending a submitted web link to the web-page recipe extraction capability for a single recipe import
- Receiving extracted ingredients, steps, and cook time, or a failure report
- User-facing behavior on FEAT-10.SPEC-001 when this capability is slow, unavailable, or rejects a link
- Disclosure to the user about what is shared with the capability (the pasted link itself)

**Non-Goals:**
- Choosing the web-page recipe extraction vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability.
- Reviewing or editing the extracted draft -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe); this spec only defines what crosses the boundary with the extraction capability.
- Determining whether extracting content from a given third-party site is legally permitted -- BRIEF.md's Open Questions names this as an unresolved product/legal decision to settle before v1 ships; this spec defines the functional extraction contract approved for build, not that resolution.
- Bulk extraction of multiple links in one request -- excluded per scope-boundaries.md SC-12: the brief names only per-link recipe import; this integration handles exactly one link per request.

## Capability Category

**Category:** Web-page recipe extraction
**Dependency Source:** ASMP-36 -- "Web-page recipe extraction capability, v1 -- Required to read a pasted recipe link and extract ingredients, steps, and cook time" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Web-page recipe extraction (ASMP-36, v1)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-10; Integration Spec: FEAT-10.SPEC-005)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A household member pastes a web link and receives a pre-filled draft of ingredients, steps, and cook time to review | Import by link | FEAT-10.SPEC-001 (Import by Link), FEAT-10.SPEC-002 (Review Extracted Recipe) |
| A member sees a brief, explained progress indicator while the page is read, rather than a bare wait | Import by link | FEAT-10.SPEC-001 (Import by Link) |
| A link that cannot be parsed prompts the member to enter the recipe details manually instead of failing silently | Import by link | FEAT-10.SPEC-003 (Manual Recipe Entry) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| The pasted web link | Not a stored entity at send time -- the raw link text the member entered on FEAT-10.SPEC-001 | Member taps Import and FEAT-10.SPEC-007's well-formedness and weekly-limit checks pass | The capability must fetch and parse the page at this address to extract recipe content |

No household data, member data, dietary rule data, or any other personal or account data ever leaves the product through this capability -- only the pasted link itself is sent.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Extracted ingredients (each with a quantity and unit) | Extraction succeeds | Recipe draft (not yet persisted) -- ingredients, held by FEAT-10.SPEC-002 for review before save |
| Extracted steps | Extraction succeeds | Recipe draft (not yet persisted) -- steps, held by FEAT-10.SPEC-002 for review before save |
| Extracted cook time | Extraction succeeds | Recipe draft (not yet persisted) -- cook_time, held by FEAT-10.SPEC-002 for review before save |
| Extraction failure report | Extraction fails or the page layout cannot be parsed | No Recipe draft is populated; FEAT-10.SPEC-001 routes the member to FEAT-10.SPEC-003 (Manual Recipe Entry) instead |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Extraction succeeded | The capability successfully reads and parses the linked page | No persisted data changes -- the extracted ingredients, steps, and cook time populate an unsaved draft | FEAT-10.SPEC-001's progress indicator completes and the member is navigated to FEAT-10.SPEC-002 with the draft pre-filled | FEAT-10.SPEC-001, FEAT-10.SPEC-002 |
| Extraction failed | The capability cannot reach the page, or the page layout cannot be parsed into recipe content | No persisted data changes; no draft is created | FEAT-10.SPEC-001 shows "We couldn't read that page." with "Enter details manually" and "Try a different link" | FEAT-10.SPEC-001, FEAT-10.SPEC-003 |

Both outcomes are direct consequences (populate a draft, or route to a fallback screen) with no branching decisions or cross-entity effects, so processing stays inline in this spec rather than routing to a standalone Automation spec.

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-10.SPEC-001 (Import by Link) | The progress indicator "Reading the recipe... this usually takes a few seconds." remains visible; if the wait exceeds what the feature's own timing contract describes as typical, no separate warning replaces it -- the indicator continues until the capability responds or the request ultimately times out into the Extraction failed path. The link input and Import button stay disabled during the wait. | The request cannot be sent; the screen shows "We couldn't read that page." with "Enter details manually" and "Try a different link" -- the same experience as an extraction failure, since the member's next useful action is identical either way. | The capability reports that the page's layout could not be parsed; the screen shows "We couldn't read that page." with "Enter details manually" and "Try a different link". |

No other screen sends requests to or displays results from this capability -- FEAT-10.SPEC-002 and FEAT-10.SPEC-003 only consume the draft or the failure outcome already delivered here; they cannot themselves experience a degradation state from this capability.

## Consent and Disclosure

- **Link-sharing disclosure** -- The helper line on FEAT-10.SPEC-001 ("We'll pull the ingredients, steps, and cook time from the page -- you'll be able to review and edit before it's saved.") discloses, in plain terms, that the pasted page is read to extract content, at the moment the member is about to submit a link. No separate consent gate blocks submission -- pasting a link and tapping Import is the member's affirmative action to proceed, consistent with this being a link the member has chosen to share, not incidental personal data.
- **What is never shared** -- No household data, member data, dietary rule data, or any other personal or account data crosses this boundary in either direction; only the pasted link leaves the product, and only extracted recipe content (ingredients, steps, cook time) or a failure report returns.

## Edge Cases

- **Extraction succeeds after the member has already navigated away from FEAT-10.SPEC-001** -- The result is held for the same import attempt; if the member returns to the flow by re-submitting the same link, extraction runs again rather than reusing a stale prior result, since no draft persists across navigation away from an in-progress request.
- **The same link is submitted twice in quick succession (double tap already prevented on the screen, but two separate submissions)** -- Each submission is an independent extraction request; whichever completes is shown to the member on FEAT-10.SPEC-002, and duplicate handling at save time is FEAT-10.SPEC-006's responsibility, not this integration's.
- **Extraction request times out mid-read** -- Treated as an Extraction failed event; the member is routed to the same "We couldn't read that page." experience as any other failure, with no half-populated draft ever shown.
- **Capability goes down mid-extraction** -- If no result was confirmed received, FEAT-10.SPEC-001 shows the extraction-failed message and no draft is created -- no half-created Recipe state, since nothing is persisted until FEAT-10.SPEC-002 or FEAT-10.SPEC-003's own save action.
- **Member submits a link while offline** -- Per FEAT-10.SPEC-001's Offline/Degraded state, the link is held as a draft locally and this integration is not invoked until connectivity returns, at which point the request proceeds normally.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Import by Link) | Triggered by (inbound) | Import tap, after validation, requests extraction |
| FEAT-10.SPEC-001 (Import by Link) | Affects (outbound) | Progress indicator, success navigation, and failure messaging surface here |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Affects (outbound) | Successful extraction populates this screen's draft |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Affects (outbound) | Extraction failure, if the member proceeds manually, routes here |

## Analytics and Success Signals

- **recipe_import_failed** (failure reason category: unreachable / unparseable / timeout) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained so extraction reliability -- a known competitor weak point named in product-features.md's Rationale -- is observable.
- **recipe_extraction_completed** (outcome: succeeded / failed; duration bracket) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained to monitor whether the "typically a few seconds" timing contract stated in the feature's own States field is being met.

## Acceptance Criteria

**FEAT-10.SPEC-005-AC-01:** Given Maya submits a well-formed link on FEAT-10.SPEC-001, when the web-page recipe extraction capability successfully parses the page, then ingredients, steps, and cook time populate the draft shown on FEAT-10.SPEC-002.

**FEAT-10.SPEC-005-AC-02:** Given Sam submits a link whose page cannot be parsed, when the capability reports the failure, then FEAT-10.SPEC-001 shows "We couldn't read that page." with "Enter details manually" and "Try a different link".

**FEAT-10.SPEC-005-AC-03:** Given Maya submits a link to a page the capability cannot reach, when the capability is down, then she sees the same "We couldn't read that page." message and can proceed to FEAT-10.SPEC-003.

**FEAT-10.SPEC-005-AC-04:** Given Sam submits a link while the capability is responding slowly, when the response has not yet returned, then the progress indicator "Reading the recipe... this usually takes a few seconds." remains visible and the Import button stays disabled.

**FEAT-10.SPEC-005-AC-05:** Given Maya submits a link, when extraction is requested, then only the pasted link itself is sent to the capability -- no household, member, or dietary data is included.

**FEAT-10.SPEC-005-AC-06:** Given Sam is about to submit his first link, when he views FEAT-10.SPEC-001, then the helper line discloses that the page will be read to extract ingredients, steps, and cook time before any link is sent.

**FEAT-10.SPEC-005-AC-07:** Given an extraction request times out mid-read, when the timeout occurs, then Maya sees the standard extraction-failed experience and no partially populated draft is ever shown.

**FEAT-10.SPEC-005-AC-08:** Given the capability goes down after Sam's request is sent but before any result is confirmed, when the request fails to complete, then no Recipe record or draft exists in any half-created state.

**FEAT-10.SPEC-005-AC-09:** Given Maya submits the same link twice in quick succession as two separate requests, when both complete, then each is treated as an independent extraction result, with duplicate handling deferred to FEAT-10.SPEC-006 at save time.

**FEAT-10.SPEC-005-AC-10:** Given Sam submits a link while offline, when the screen is offline, then this capability is not invoked until connectivity returns, per FEAT-10.SPEC-001's Offline/Degraded state.

**FEAT-10.SPEC-005-AC-11:** Given Maya navigates away from FEAT-10.SPEC-001 after submitting a link but before extraction completes, when she later re-submits the same link, then a fresh extraction request runs rather than reusing any prior result.

**FEAT-10.SPEC-005-AC-12:** Given extraction succeeds for Sam's link, when the draft is populated, then no data beyond ingredients, steps, and cook time is carried into FEAT-10.SPEC-002 from this integration.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 2 | 2 |
| Degradation Paths | 3 (1 screen) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
