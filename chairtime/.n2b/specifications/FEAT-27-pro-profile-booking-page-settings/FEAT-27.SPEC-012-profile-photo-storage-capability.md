---
document_type: spec
spec_type: integration
spec_id: FEAT-27.SPEC-012
spec_name: Profile Photo Storage Capability
spec_slug: profile-photo-storage-capability
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Integration Spec: Profile Photo Storage Capability

## Overview

**Name:** Profile Photo Storage Capability
**ID:** FEAT-27.SPEC-012
**Type:** Integration
**Purpose:** The product stores, replaces, and serves Talia's profile photo through an external file-storage capability, so the booking page still works correctly even when no photo has been set.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Uploading and replacing the Pro's profile photo within its size and format limits
- Serving the stored photo to the public booking page
- User-facing behavior when the storage capability is slow, unavailable, or rejects an upload
- Disclosure to the Pro about what is shared with this capability

**Non-Goals:**
- Choosing the storage vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for file storage.
- Editing, cropping, or otherwise transforming the photo before upload -- product-features.md's Validation & Limits names only a size/format limit, not an editing capability; no cropping or filter tool is defined anywhere in Stage 2.
- Storing any file other than the Pro's own profile photo -- no other file-upload capability exists anywhere in this product's feature set (studio_address and every other field in this feature are plain text).
- The screen mechanics of triggering an upload -- owned by FEAT-27.SPEC-001 (Profile & Booking Page Settings); this spec defines only the storage-capability behavior that screen surfaces.

## Capability Category

**Category:** File storage
**Dependency Source:** ASMP-35 -- "File storage capability for pro profile photos" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "File storage -- Pro profile photos" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-27, FEAT-05)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia uploads or replaces her profile photo from settings | Edit the public profile -- display name, photo, short intro, general area | FEAT-27.SPEC-001 (Profile & Booking Page Settings) |
| The public booking page displays Talia's photo, or works correctly with just her name when none is set | Preview the booking page as a client sees it; the booking loop is unaffected by photo absence, per product-features.md's Rationale | FEAT-05 (Public Booking Page & Booking Flow) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Photo file | Pro Account -- photo (the raw uploaded file) | Talia uploads or replaces her photo on FEAT-27.SPEC-001 | The capability must have the file to store and later serve it |

No other Pro Account field, and no Client or Booking data, ever leaves the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Stored-photo reference (a servable location for the file) | The capability confirms the upload succeeded | Pro Account -- photo (the servable reference FEAT-05 reads to display the photo) |
| Upload rejection reason (size / format) | The capability rejects an upload | Not persisted -- shown inline on FEAT-27.SPEC-001 and discarded; the Pro Account's photo field is left unchanged |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Upload succeeded | The capability finishes storing an uploaded or replacement photo | Pro Account -- photo set to the new stored-photo reference | FEAT-27.SPEC-001 shows the new photo in place; the public booking page (FEAT-05) reflects it immediately | FEAT-27.SPEC-001, FEAT-05 |
| Upload rejected (size exceeded) | The uploaded file exceeds the size limit (platform parameter: `profile-photo-max-file-size-mb`) | None -- Pro Account's photo field is unchanged | FEAT-27.SPEC-001 shows inline: "This photo is too large. Choose one under {the limit}." | FEAT-27.SPEC-001 |
| Upload rejected (unsupported format) | The uploaded file is not a common photo file type the capability accepts | None -- Pro Account's photo field is unchanged | FEAT-27.SPEC-001 shows inline: "This file type isn't supported. Choose a common photo format." | FEAT-27.SPEC-001 |
| Stored photo becomes unservable (capability-side loss or corruption, reported after the fact) | The capability reports it can no longer serve a previously-stored photo | Pro Account -- photo cleared back to empty | The public booking page (FEAT-05) falls back to showing Talia's name only, exactly as the Empty state already handles no-photo; FEAT-27.SPEC-001 shows the "Add a photo" prompt again on next view | FEAT-27.SPEC-001, FEAT-05 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | The photo element shows an uploading state; after 10 seconds a note appears: "Still uploading -- this is taking longer than usual." The rest of the settings screen remains fully usable and other fields can still be saved. | The photo element shows: "Photo upload is temporarily unavailable. The rest of your settings are unaffected -- try the photo again in a few minutes." Talia can still save every other field on this screen. | The exact rejection reason (size or format) is shown inline below the photo element, per Inbound Events above; the previous photo (or empty state) remains unaffected. |
| FEAT-05 (Public Booking Page & Booking Flow, photo display) | The booking page renders without waiting on the photo -- text content (name, services) appears immediately; the photo fades in once it loads, or the page proceeds with no photo if it does not load within the page's own rendering budget. | The booking page shows Talia's name and services with no photo -- identical to the empty-photo state; no error is shown to the client, since a missing photo is never a client-facing failure. | N/A -- FEAT-05 only reads a already-stored photo; it never submits an upload that could be rejected. |

## Consent and Disclosure

- **First photo upload disclosure** -- The first time Talia uploads a photo, a notice appears before the upload begins: "Your photo is shared with an external file-storage service so it can be displayed on your public booking page." Options: "Continue" and "Cancel." Shown once; afterwards no repeat notice appears for subsequent replacements, since the same storage relationship already applies.
- **What is never shared** -- No Client data, no Booking data, and no Pro Account field other than the photo file itself ever leaves the product through this capability. This boundary is stated in the disclosure notice.
- **Client-facing exposure** -- The stored photo is publicly servable on Talia's booking page by design (it is a public profile field, per the Pro Account entity's Data Sensitivity note); no additional client-facing disclosure applies beyond what FEAT-05 already states about the page being public.

## Edge Cases

- **Upload succeeds but the storage capability reports the photo unservable moments later** -- Handled as the "Stored photo becomes unservable" inbound event above: the field clears and both settings and the public page revert to the no-photo appearance, with no error blame directed at Talia.
- **The same upload-succeeded event is delivered twice (a retried confirmation)** -- The second delivery changes nothing: the photo field already holds the correct stored-photo reference, and no duplicate upload or duplicate feedback occurs.
- **An upload-rejected event arrives after Talia has already navigated away from FEAT-27.SPEC-001** -- No feedback is shown (there is no screen open to show it on); her photo field remains at its previous value, and she sees the empty/previous state normally on her next visit, with no attempt she is unaware of having silently succeeded.
- **Capability goes down mid-upload** -- If the upload was not confirmed stored, Talia's Pro Account photo field is unaffected -- no half-uploaded or broken reference is ever written; FEAT-05 continues serving whatever photo (or no photo) was already confirmed before the outage.
- **Talia uploads a replacement photo while the public booking page is being viewed by a client at that exact moment** -- The client's already-loaded page view is unaffected mid-view; the next page load (or refresh) shows the new photo, consistent with the Feature Breakdown Brief's Non-Functional Notes ("the change is visible on the public booking page immediately" -- immediacy applies to new loads, not to a page already rendered in a client's browser).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Triggered by (inbound) | The photo element's upload/replace action initiates a storage request |
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Affects (outbound) | Upload outcomes, degradation states, and the disclosure notice surface here |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | The stored photo (or its absence) is served here on every page load |

## Analytics and Success Signals

- **profile_photo_uploaded** (outcome: success / failure; failure_reason: size / format / unavailable) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so photo-upload adoption and friction are observable
- **profile_photo_became_unservable** () -- N/A -- no connected success-metrics.md metric; retained so this rare capability-side event is observable rather than silently degrading a Pro's page

## Acceptance Criteria

**FEAT-27.SPEC-012-AC-01:** Given Talia has never uploaded a photo, when she taps to upload one for the first time, then the data-sharing notice appears with "Continue" and "Cancel", and no file leaves the product until she chooses "Continue".

**FEAT-27.SPEC-012-AC-02:** Given Talia uploads a valid photo within the size and format limits, when the capability confirms storage, then her photo updates on FEAT-27.SPEC-001 and on the public booking page (FEAT-05).

**FEAT-27.SPEC-012-AC-03:** Given Talia uploads a photo exceeding platform parameter: `profile-photo-max-file-size-mb`, when the capability rejects it, then she sees "This photo is too large. Choose one under {the limit}." and her previous photo (or empty state) is unaffected.

**FEAT-27.SPEC-012-AC-04:** Given Talia uploads a file in an unsupported format, when the capability rejects it, then she sees "This file type isn't supported. Choose a common photo format." and her previous photo is unaffected.

**FEAT-27.SPEC-012-AC-05:** Given a client visits Talia's booking page and she has never set a photo, when the page loads, then it shows her name and services with no photo and no error.

**FEAT-27.SPEC-012-AC-06:** Given the storage capability is unavailable when Talia attempts an upload, when the request cannot be sent, then she sees "Photo upload is temporarily unavailable. The rest of your settings are unaffected -- try the photo again in a few minutes." and can still save her other fields.

**FEAT-27.SPEC-012-AC-07:** Given Talia's already-stored photo becomes unservable at the capability, when that is reported, then her Pro Account's photo field clears and the public booking page falls back to showing her name only.

**FEAT-27.SPEC-012-AC-08:** Given an upload-succeeded confirmation is delivered twice for the same upload, when the second delivery arrives, then nothing changes and no duplicate feedback appears.

**FEAT-27.SPEC-012-AC-09:** Given the storage capability goes down mid-upload before confirming success, when Talia checks her settings afterward, then her photo field is unaffected -- no broken or half-uploaded reference exists.

**FEAT-27.SPEC-012-AC-10:** Given Talia's photo upload is rejected after she has already left FEAT-27.SPEC-001, when the rejection event arrives, then no feedback is shown on any screen, and her photo field remains at its previous value.

**FEAT-27.SPEC-012-AC-11:** Given Talia replaces her photo while a client is already viewing her booking page, when the client's page was loaded before the replacement, then the client's current view is unaffected; the new photo appears on the client's next page load.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 5 (2 screens; 1 N/A cell excluded) | 5 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
