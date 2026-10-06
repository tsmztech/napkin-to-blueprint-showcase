---
document_type: technical-profile
produced_by: profile-analyst
status: final
stage: 4
created: 2026-09-29
project_name: Chairtime
---

# Project Technical Profile

## 1. Scale Metrics

Counts from the `FEAT-*` folders and spec files on disk (`ls -d`, `find`) and from spec frontmatter (`spec_type`). The five spec-type counts sum to the total spec count (70+59+53+13+24 = 219). Feature types are from the Features table in feature-dependency-map.md.

| Metric | Value |
|--------|-------|
| Total features | 30 |
| Total specs | 219 |
| Screen specs | 70 |
| Automation specs | 59 |
| Logic/Rule specs | 53 |
| Integration specs | 13 |
| Notification specs | 24 |
| User-Facing features | 20 |
| Platform features | 7 |
| Lifecycle features | 3 |

## 2. Complexity Metrics

Counted with Bash/grep from product-features.md, feature-dependency-map.md and the feature-overview.md files. Inter-entity relationships: 18 `**Relationships:**` lines have a value other than "None"; summing the distinct entity-to-entity relationships those lines name gives 56 (an entity naming another entity counts once on its own line, so a pair named from both sides counts twice).

| Metric | Value |
|--------|-------|
| Entity count | 18 |
| Inter-entity relationships | 56 (named across 18 Relationships lines) |
| Cross-feature business rules (XBR) | 29 |
| Cross-feature touchpoint rows | 320 |
| Cross-feature integration density | 10.67 (320 / 30) |
| Navigation connections | 46 |
| Hub screens (3+ inbound connections) | 1 screen-level: Live Slot List (FEAT-03), 3 inbound (from FEAT-05 service list, FEAT-07 decline/hold expired, FEAT-10 reschedule). By destination feature, 5 features receive 3 or more inbound connections: FEAT-30 Pro Booking Management (6), FEAT-05 Public Booking Page & Booking Flow (4), FEAT-28 Payout Account Connection & Payout Visibility (3), FEAT-16 Booking & Payment Activity Record (3), FEAT-03 Real-Time Slot Availability Engine (3) |
| Average specs per feature | 7.30 (219 / 30) |

## 3. Capability Signals

Scanned with `grep` across all 219 spec files (all five types, including Integration Capability Category / Data Exchanged / Degradation Behavior sections and Notification Channels / Trigger / Delivery Rules sections), BRIEF.md, assumptions-constraints.md and feature-dependency-map.md. Broad keywords from the scan table (for example "file", "live", "location", "session", "shared", "download") also match generic wording in the specs; the Detail column lists the specs where the behavior itself is described.

| Signal | Present | Detail |
|--------|---------|--------|
| Real-time | Yes | FEAT-03.SPEC-001, FEAT-07.SPEC-002, FEAT-08.SPEC-005, FEAT-22.SPEC-002, FEAT-28.SPEC-002 -- the slot list is recomputed on a refresh cycle of roughly one second while a client is viewing it (FEAT-03.SPEC-001); in-app notices and status banners update without a manual refresh (FEAT-08.SPEC-005 citing FEAT-12's live-updating nature; FEAT-28.SPEC-002 banner clears without a manual refresh). No WebSocket or collaborative-editing behavior is named. All seven FEAT-03 specs belong to the feature named Real-Time Slot Availability Engine. ASMP-21 (assumptions-constraints.md, Non-Functional Expectations) states slots appear within roughly one second |
| Offline | Yes | Read-only offline posture, not offline-first: ASMP-27 (assumptions-constraints.md, Non-Functional Expectations) states booking, paying, cancelling, refunding and no-show marking need a live connection while the most recently loaded schedule, client list and money list stay readable. Specs with an Offline/Degraded row describing an already-loaded view that stays visible: FEAT-04.SPEC-001, FEAT-04.SPEC-002, FEAT-05.SPEC-001, FEAT-05.SPEC-004, FEAT-05.SPEC-005, FEAT-06.SPEC-001, FEAT-06.SPEC-003, FEAT-06.SPEC-004, FEAT-06.SPEC-005, FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-003, FEAT-11.SPEC-001, FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003, FEAT-13.SPEC-001, FEAT-13.SPEC-002, FEAT-13.SPEC-003, FEAT-14.SPEC-001, FEAT-15.SPEC-003, FEAT-16.SPEC-001, FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-18.SPEC-002, FEAT-20.SPEC-002, FEAT-24.SPEC-001, FEAT-25.SPEC-001, FEAT-26.SPEC-001, FEAT-28.SPEC-002, FEAT-29.SPEC-003, FEAT-29.SPEC-005, FEAT-30.SPEC-004 |
| File upload | Yes | FEAT-27.SPEC-001, FEAT-27.SPEC-012 (Pro profile photo upload, size and format limits, FEAT-27.SPEC-012 Integration Category: File storage; ASMP-35) |
| Complex forms (10+ fields) | Yes | No Screen spec enumerates 10 or more fields in one form. Multi-step wizard: FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-003 (FEAT-15 Pro Onboarding & Setup Wizard, 8 setup steps per the Navigation Connections table); multi-step booking flow FEAT-05.SPEC-001 to FEAT-05.SPEC-009 |
| Background processing | Yes | 31 Automation specs with scheduled, timed or expiry triggers: FEAT-02.SPEC-004, FEAT-03.SPEC-003, FEAT-03.SPEC-007, FEAT-04.SPEC-005, FEAT-04.SPEC-006, FEAT-06.SPEC-002, FEAT-08.SPEC-007, FEAT-08.SPEC-010, FEAT-09.SPEC-004, FEAT-10.SPEC-004, FEAT-12.SPEC-004, FEAT-12.SPEC-005, FEAT-16.SPEC-002, FEAT-17.SPEC-004, FEAT-17.SPEC-005, FEAT-17.SPEC-006, FEAT-17.SPEC-007, FEAT-18.SPEC-003, FEAT-18.SPEC-004, FEAT-21.SPEC-004, FEAT-21.SPEC-005, FEAT-25.SPEC-004, FEAT-27.SPEC-010, FEAT-27.SPEC-011, FEAT-29.SPEC-006, FEAT-29.SPEC-008, FEAT-29.SPEC-009, FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-30.SPEC-009, FEAT-30.SPEC-010 |
| Authentication | Yes | Role-based -- Pro one-time-code sign-in with new-device alerts (FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-014, FEAT-29.SPEC-015; ASMP-30), client access links (FEAT-06), and a read-only Platform Operator (Support) role (FEAT-19); Access Matrix roles in specs: The Pro, The Client, Platform Operator (Support). Specs: FEAT-12.SPEC-008, FEAT-15.SPEC-004, FEAT-19.SPEC-001, FEAT-19.SPEC-004, FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-003, FEAT-29.SPEC-004, FEAT-29.SPEC-005, FEAT-29.SPEC-006, FEAT-29.SPEC-007, FEAT-29.SPEC-008, FEAT-29.SPEC-009, FEAT-29.SPEC-010, FEAT-29.SPEC-011, FEAT-29.SPEC-012, FEAT-29.SPEC-013, FEAT-29.SPEC-014, FEAT-29.SPEC-015, FEAT-29.SPEC-016, FEAT-29.SPEC-017 |
| Search | Yes | Simple filter -- FEAT-24.SPEC-001, FEAT-24.SPEC-002 (client search by match and filter derivation, FEAT-24.SPEC-001 / FEAT-24.SPEC-002; Active/Archived filters and list filters elsewhere). No spec names full-text or faceted search |
| Payments/billing | Yes | Payment-related specs: FEAT-01.SPEC-004, FEAT-01.SPEC-005, FEAT-03.SPEC-007, FEAT-05.SPEC-004, FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-003, FEAT-07.SPEC-004, FEAT-07.SPEC-005, FEAT-08.SPEC-004, FEAT-09.SPEC-003, FEAT-09.SPEC-005, FEAT-09.SPEC-006, FEAT-11.SPEC-002, FEAT-12.SPEC-007, FEAT-16.SPEC-003, FEAT-16.SPEC-004, FEAT-18.SPEC-002, FEAT-18.SPEC-003, FEAT-18.SPEC-004, FEAT-18.SPEC-005, FEAT-18.SPEC-006, FEAT-18.SPEC-007, FEAT-21.SPEC-005, FEAT-21.SPEC-009, FEAT-22.SPEC-001, FEAT-22.SPEC-002, FEAT-22.SPEC-003, FEAT-22.SPEC-004, FEAT-22.SPEC-005, FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-23.SPEC-003, FEAT-28.SPEC-001, FEAT-28.SPEC-002, FEAT-28.SPEC-003, FEAT-28.SPEC-004, FEAT-28.SPEC-006, FEAT-28.SPEC-007, FEAT-30.SPEC-003, FEAT-30.SPEC-009, FEAT-30.SPEC-010, FEAT-30.SPEC-011, FEAT-30.SPEC-013. Integration specs with Category: Payment processing: FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-30.SPEC-011; ASMP-31 (assumptions-constraints.md, Dependencies); BRIEF.md, ## Business Context (deposit at booking, flat monthly subscription) and ## Ecosystem & Integrations (established card payment processor) |
| Notifications (email/push/SMS) | Yes | 24 Notification specs: FEAT-04.SPEC-007, FEAT-06.SPEC-006, FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006, FEAT-10.SPEC-006, FEAT-14.SPEC-009, FEAT-15.SPEC-008, FEAT-18.SPEC-007, FEAT-20.SPEC-008, FEAT-20.SPEC-009, FEAT-21.SPEC-007, FEAT-21.SPEC-008, FEAT-21.SPEC-009, FEAT-27.SPEC-013, FEAT-28.SPEC-007, FEAT-29.SPEC-014, FEAT-29.SPEC-015, FEAT-29.SPEC-016, FEAT-29.SPEC-017, FEAT-30.SPEC-012, FEAT-30.SPEC-013. Channels values: Text and Email (most), In-app (FEAT-04.SPEC-007, FEAT-08.SPEC-005, FEAT-08.SPEC-006, FEAT-15.SPEC-008, FEAT-18.SPEC-007, FEAT-27.SPEC-013, FEAT-28.SPEC-007), Email and SMS (text) (FEAT-29.SPEC-014 to FEAT-29.SPEC-017), On-screen code (FEAT-30.SPEC-013). Integration specs FEAT-08.SPEC-012 (text), FEAT-08.SPEC-013 (email), FEAT-26.SPEC-002 (WhatsApp, Later); ASMP-32 (assumptions-constraints.md, Dependencies) |
| Third-party integrations | Yes | 13 Integration specs with Capability Category values: FEAT-03.SPEC-006 (Calendar sync), FEAT-04.SPEC-003 (Calendar sync), FEAT-07.SPEC-005 (Payment processing), FEAT-08.SPEC-012 (Transactional text messaging), FEAT-08.SPEC-013 (Transactional email), FEAT-09.SPEC-005 (Payment processing), FEAT-16.SPEC-003 (Payment processing -- card-issuer dispute notifications), FEAT-18.SPEC-006 (Payment processing), FEAT-22.SPEC-005 (Payment processing), FEAT-26.SPEC-002 (Transactional WhatsApp messaging), FEAT-27.SPEC-012 (File storage), FEAT-28.SPEC-006 (Payment processing), FEAT-30.SPEC-011 (Payment processing). External Touchpoints rows: 9 (quoted in Section 7); BRIEF.md, ## Ecosystem & Integrations; ASMP-31, ASMP-32, ASMP-33, ASMP-35 |
| AI/ML behavior | No | No specs reference AI or machine-learning behavior (keyword hits in FEAT-03.SPEC-001, FEAT-16.SPEC-004, FEAT-30.SPEC-003 and FEAT-02.SPEC-004 are negations or rule-evaluation wording, for example "not a recommendation engine"); BRIEF.md and assumptions-constraints.md Dependencies name no AI capability |
| Geo/maps | No | No specs reference location or mapping behavior; the studio address (FEAT-27.SPEC-001) and general area are stored and displayed as text fields, with no geocoding, distance or route behavior; BRIEF.md, ## Ecosystem & Integrations and assumptions-constraints.md name no mapping capability |
| Import/export | Yes | FEAT-29.SPEC-004 (Data Export Screen), FEAT-29.SPEC-007 (Data Export Generation), FEAT-16.SPEC-004 (Dispute Summary Download), FEAT-08.SPEC-001 (add-to-calendar .ics link built from a booking). No bulk import spec; FEAT-24.SPEC-001 lists bulk export among its non-goals. BRIEF.md names no import or export |
| Collaboration/concurrency | Yes | Every Shared Data Entities Contention line is non-"None" except Message and Activity Event (16 of 18 entities). Resolution styles named: last-write-wins, reject-with-refresh, first committed wins, most recent explicit client action by timestamp wins. Booking (High contention), Deposit Transaction, Waitlist Entry, Recurring Series, Time Block, Access Link (feature-dependency-map.md, Shared Data Entities). Specs: FEAT-01.SPEC-001, FEAT-01.SPEC-003, FEAT-01.SPEC-005, FEAT-01.SPEC-006, FEAT-02.SPEC-001, FEAT-02.SPEC-002, FEAT-02.SPEC-003, FEAT-02.SPEC-004, FEAT-03.SPEC-001, FEAT-03.SPEC-002, FEAT-03.SPEC-003, FEAT-03.SPEC-005, FEAT-03.SPEC-007, FEAT-04.SPEC-001, FEAT-04.SPEC-002, FEAT-04.SPEC-003, FEAT-04.SPEC-004, FEAT-04.SPEC-005, FEAT-04.SPEC-006, FEAT-04.SPEC-008, FEAT-05.SPEC-001, FEAT-05.SPEC-003, FEAT-05.SPEC-006, FEAT-06.SPEC-001, FEAT-06.SPEC-002, FEAT-06.SPEC-003, FEAT-06.SPEC-004, FEAT-06.SPEC-005, FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-007, FEAT-08.SPEC-008, FEAT-08.SPEC-009, FEAT-08.SPEC-010, FEAT-08.SPEC-011, FEAT-09.SPEC-001, FEAT-09.SPEC-002, FEAT-09.SPEC-004, FEAT-10.SPEC-001, FEAT-10.SPEC-002, FEAT-10.SPEC-003, FEAT-10.SPEC-004, FEAT-10.SPEC-005, FEAT-11.SPEC-001, FEAT-11.SPEC-002, FEAT-11.SPEC-003, FEAT-11.SPEC-004, FEAT-12.SPEC-001, FEAT-12.SPEC-003, FEAT-12.SPEC-004, FEAT-12.SPEC-005, FEAT-12.SPEC-006, FEAT-13.SPEC-001, FEAT-13.SPEC-002, FEAT-13.SPEC-004, FEAT-13.SPEC-005, FEAT-13.SPEC-006, FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-004, FEAT-14.SPEC-005, FEAT-14.SPEC-006, FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-003, FEAT-15.SPEC-004, FEAT-15.SPEC-005, FEAT-16.SPEC-001, FEAT-16.SPEC-004, FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-17.SPEC-004, FEAT-17.SPEC-005, FEAT-17.SPEC-006, FEAT-17.SPEC-007, FEAT-18.SPEC-001, FEAT-18.SPEC-002, FEAT-18.SPEC-003, FEAT-18.SPEC-004, FEAT-18.SPEC-005, FEAT-19.SPEC-002, FEAT-19.SPEC-004, FEAT-20.SPEC-005, FEAT-20.SPEC-006, FEAT-20.SPEC-007, FEAT-21.SPEC-002, FEAT-21.SPEC-004, FEAT-21.SPEC-005, FEAT-21.SPEC-006, FEAT-21.SPEC-010, FEAT-22.SPEC-001, FEAT-22.SPEC-002, FEAT-22.SPEC-003, FEAT-22.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-24.SPEC-001, FEAT-25.SPEC-001, FEAT-25.SPEC-002, FEAT-25.SPEC-004, FEAT-26.SPEC-001, FEAT-26.SPEC-003, FEAT-27.SPEC-001, FEAT-27.SPEC-002, FEAT-27.SPEC-004, FEAT-27.SPEC-005, FEAT-27.SPEC-007, FEAT-27.SPEC-010, FEAT-27.SPEC-011, FEAT-28.SPEC-003, FEAT-28.SPEC-004, FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-003, FEAT-29.SPEC-004, FEAT-29.SPEC-005, FEAT-29.SPEC-006, FEAT-29.SPEC-007, FEAT-29.SPEC-008, FEAT-29.SPEC-009, FEAT-29.SPEC-010, FEAT-29.SPEC-013, FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-005, FEAT-30.SPEC-006, FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-30.SPEC-009, FEAT-30.SPEC-010, FEAT-30.SPEC-012; XBR-01 slot tie-break |
| Compliance/privacy | Yes | US texting-consent rules (ASMP-24); privacy posture and deletion on request (ASMP-23); account protection (ASMP-30); card data held by the payment processor (ASMP-31). Data Sensitivity lines on all 18 entities (feature-dependency-map.md, Shared Data Entities), including Client (Personal data), Messaging Consent (Compliance evidence), Deposit Transaction (Financial record), Access Link (Security-sensitive). BRIEF.md, ## Constraints (Messaging consent, Personal data). Specs: FEAT-05.SPEC-003, FEAT-06.SPEC-005, FEAT-06.SPEC-008, FEAT-08.SPEC-011, FEAT-13.SPEC-003, FEAT-13.SPEC-004, FEAT-13.SPEC-006, FEAT-14.SPEC-001, FEAT-14.SPEC-002, FEAT-14.SPEC-003, FEAT-14.SPEC-004, FEAT-14.SPEC-005, FEAT-14.SPEC-006, FEAT-14.SPEC-008, FEAT-14.SPEC-009, FEAT-16.SPEC-005, FEAT-26.SPEC-004, FEAT-29.SPEC-005, FEAT-29.SPEC-008, FEAT-29.SPEC-013, FEAT-29.SPEC-017 |
| Internationalization | Yes | Currency and timezone are per-account settings, never hard-coded (ASMP-25; BRIEF.md, ## Scale & Non-Functional Expectations, Geography). Specs: FEAT-07.SPEC-001, FEAT-15.SPEC-002, FEAT-22.SPEC-001, FEAT-27.SPEC-003, FEAT-27.SPEC-008. FEAT-15.SPEC-002 excludes multi-language policy wording (scope-boundaries.md SC-10); no spec describes multi-language or locale-translation behavior |
| Scale hints | Yes | ASMP-22 (assumptions-constraints.md, Non-Functional Expectations: a few hundred pros in year one, roughly 100-500 clients and 20-40 bookings a week per pro, multi-year history); ASMP-21 (slots within roughly one second, full booking in under one minute); ASMP-26 (correctness bar, no numeric uptime target); BRIEF.md, ## Scale & Non-Functional Expectations (Volume) |

## 4. Derived Classifications

**Classification rule:** the classification describes blueprint scope -- how much product definition the architecture must cover -- not deployment scale. Deployment-scale evidence (users, traffic, data growth) lives in Section 7 and must never be derived from document counts.

### Threshold Definitions

| Axis | Small | Medium | Large |
|------|-------|--------|-------|
| Scale | 1-8 features, <=40 specs | 9-24 features, 41-200 specs | 25+ features, 201+ specs |
| Data Complexity | <=10 entities, few cross-entity rules | 11-30 entities, moderate relationship graph and cross-entity rules | 31+ entities, dense relationship graph |
| Interaction Complexity | Mostly CRUD screens, <=5 automations, no real-time or collaboration signals | Mixed screens with state, 6-15 automations, integration or notification behavior present | Complex state machines, 16+ automations, real-time or collaboration signals present |

### Project Classification

| Axis | Classification | Evidence |
|------|---------------|----------|
| Scale | Large | 30 features (25+ threshold), 219 specs (201+ threshold) |
| Data Complexity | Medium | 18 entities (11-30 band), 56 named inter-entity relationships, 29 cross-feature business rules |
| Interaction Complexity | Large | 59 Automation specs (16+ threshold), 13 Integration specs, 24 Notification specs, Real-time signal present (Section 3), Collaboration/concurrency signal present (Section 3), 70 Screen specs, 53 Logic/Rule specs |

**Summary:** Large / Medium / Large

(Blueprint scope only -- the deployment-scale evidence for this product lives in Section 7.)

## 5. Entity Inventory

Every `### Entity:` block of the Domain Entity Inventory in product-features.md (18). Managing Feature is the entity's "Managed by" line (or "Created by" where Managed by is N/A). Field Count and Relationships come from the entity's Shared Data Entities subsection in feature-dependency-map.md, where all 18 entities appear.

| Entity Name | Managing Feature | Field Count | Relationships |
|-------------|-----------------|-------------|---------------|
| Pro Account | Pro Profile & Booking Page Settings (FEAT-27), Pro Sign-In & Account Lifecycle (FEAT-29), Pro Subscription Billing & Account Management (FEAT-18) | 12 | The root of every other record: one Pro Account owns its Services, Availability Rules, Time Blocks, Calendar Connection(s), Clients, Bookings, Cancellation Policy versions, Subscription and Payout Account. No record is ever shared between two Pro Accounts. |
| Service | Service & Pricing Management (FEAT-01) | 7 | Belongs to one Pro Account; referenced by many Bookings (each Booking keeps the price and deposit agreed at booking time) and by Waitlist Entries. |
| Availability Rule | Availability & Working Hours Setup (FEAT-02) | 6 | Belongs to one Pro Account; combined with Time Blocks, Bookings, Calendar Connection busy time and Recurring Series by FEAT-03 to compute open slots. |
| Time Block | Manual Time Blocking (FEAT-17) | 3 | Belongs to one Pro Account; removes availability in FEAT-03; may overlap confirmed Bookings, which are then handed to FEAT-30 by the Pro's explicit choice. |
| Calendar Connection | Two-Way Calendar Sync (FEAT-04) | 4 | Belongs to one Pro Account; feeds busy time into FEAT-03 and receives Chairtime bookings created or changed by FEAT-05, FEAT-10, FEAT-11 and FEAT-30. |
| Client | Client Record Management (FEAT-13) | 6 | Belongs to exactly one Pro Account (a person booking two pros has two unconnected records, scope-boundaries SC-04); has many Bookings, one Messaging Consent per channel, Access Links and Waitlist Entries. |
| Booking | Client-Initiated Cancel/Reschedule (FEAT-10), No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Daily Schedule Dashboard (FEAT-12), Pro Booking Management (FEAT-30) | 9 | Belongs to one Pro Account and one Client; references one Service and one Cancellation Policy version; has one Deposit Transaction, zero or one Balance Payment, many Messages and many Activity Events; may belong to a Recurring Series. |
| Deposit Transaction | Cancellation & No-Show Policy Engine (FEAT-09), No-Show Marking & Deposit Forfeiture (FEAT-11), Pro Booking Management (FEAT-30) | 4 | Belongs to one Booking; paid out to the Pro's Payout Account; listed in the money list (FEAT-28). |
| Cancellation Policy | Cancellation & No-Show Policy Engine (FEAT-09) | 5 | Belongs to one Pro Account; each Booking points at the version the client acknowledged. |
| Messaging Consent | Messaging Consent Management (FEAT-14) | 4 | Belongs to one Client with one Pro; consulted before every message. |
| Message | Automated Booking Messaging (FEAT-08) | 4 | Belongs to one Booking (or to the Pro Account for Pro notifications). |
| Subscription | Pro Subscription Billing & Account Management (FEAT-18) | 4 | One per Pro Account; its lapse pauses new bookings (XBR-14). |
| Waitlist Entry | Waitlist for Cancelled Slots (FEAT-20) | 3 | Belongs to one Client with one Pro; converts into a Booking. |
| Recurring Series | Recurring/Standing Appointments (FEAT-21) | 4 | Belongs to one Client with one Pro; generates Bookings, each with its own deposit. |
| Payout Account | Payout Account Connection & Payout Visibility (FEAT-28) | 4 | One per Pro Account; receives every Deposit Transaction (and, from v1, Balance Payments and tips). |
| Activity Event | Booking & Payment Activity Record (FEAT-16), written automatically as other features act | 2 | Belongs to one Booking or to the Pro Account (support-view log). |
| Access Link | Client Booking Identity (FEAT-06) | 3 | Belongs to one Client with one Pro (and optionally one Booking). |
| Balance Payment | In-App Balance Payment (FEAT-22), Pro Booking Management (FEAT-30) (refund if the appointment is cancelled after the balance was paid) | 3 | Belongs to one Booking; paid out to the Pro's Payout Account. |

## 6. Raw Spec Index

All 219 specs from the Spec Inventory tables of the 30 feature-overview.md files, in FEAT-NN then SPEC-NNN order. Key Signals cross-reference the Section 3 rows.

| Spec ID | Name | Type | Feature | Key Signals |
|---------|------|------|---------|-------------|
| FEAT-01.SPEC-001 | Service List | Screen | FEAT-01 (Service & Pricing Management) | Collaboration/concurrency |
| FEAT-01.SPEC-002 | Add Service | Screen | FEAT-01 (Service & Pricing Management) | -- |
| FEAT-01.SPEC-003 | Edit Service | Screen | FEAT-01 (Service & Pricing Management) | Collaboration/concurrency |
| FEAT-01.SPEC-004 | Service Field & Deposit Rule Validation | Logic/Rule | FEAT-01 (Service & Pricing Management) | Payments/billing |
| FEAT-01.SPEC-005 | Price & Deposit Lock at Booking Time | Logic/Rule | FEAT-01 (Service & Pricing Management) | Payments/billing; Collaboration/concurrency |
| FEAT-01.SPEC-006 | Archive Impact Check | Automation | FEAT-01 (Service & Pricing Management) | Collaboration/concurrency |
| FEAT-02.SPEC-001 | Working Hours, Buffer, Notice & Horizon Setup | Screen | FEAT-02 (Availability & Working Hours Setup) | Collaboration/concurrency |
| FEAT-02.SPEC-002 | Per-Service Buffer Override | Screen | FEAT-02 (Availability & Working Hours Setup) | Collaboration/concurrency |
| FEAT-02.SPEC-003 | Availability Rule Versioning | Automation | FEAT-02 (Availability & Working Hours Setup) | Collaboration/concurrency |
| FEAT-02.SPEC-004 | Confirmed Booking Conflict Flagging | Automation | FEAT-02 (Availability & Working Hours Setup) | Background processing; Collaboration/concurrency |
| FEAT-02.SPEC-005 | Availability Setup Validation & Limits | Logic/Rule | FEAT-02 (Availability & Working Hours Setup) | -- |
| FEAT-03.SPEC-001 | Slot Availability Computation | Automation | FEAT-03 (Real-Time Slot Availability Engine) | Real-time; Collaboration/concurrency |
| FEAT-03.SPEC-002 | Slot Hold Creation & Checkout Reservation | Automation | FEAT-03 (Real-Time Slot Availability Engine) | Collaboration/concurrency |
| FEAT-03.SPEC-003 | Slot Hold Expiration | Automation | FEAT-03 (Real-Time Slot Availability Engine) | Background processing; Collaboration/concurrency |
| FEAT-03.SPEC-004 | Slot Validation & Timing Rules | Logic/Rule | FEAT-03 (Real-Time Slot Availability Engine) | -- |
| FEAT-03.SPEC-005 | Slot Contention Resolution Rules | Logic/Rule | FEAT-03 (Real-Time Slot Availability Engine) | Collaboration/concurrency |
| FEAT-03.SPEC-006 | Calendar Busy-Time Consumption & Degraded Mode | Integration | FEAT-03 (Real-Time Slot Availability Engine) | Third-party integrations |
| FEAT-03.SPEC-007 | Pro-Created Deposit Request Hold & Expiration | Automation | FEAT-03 (Real-Time Slot Availability Engine) | Background processing; Payments/billing; Collaboration/concurrency |
| FEAT-04.SPEC-001 | Calendar Connection Setup | Screen | FEAT-04 (Two-Way Calendar Sync) | Offline; Collaboration/concurrency |
| FEAT-04.SPEC-002 | Calendar Connection Status & Management | Screen | FEAT-04 (Two-Way Calendar Sync) | Offline; Collaboration/concurrency |
| FEAT-04.SPEC-003 | Calendar Provider Sync | Integration | FEAT-04 (Two-Way Calendar Sync) | Third-party integrations; Collaboration/concurrency |
| FEAT-04.SPEC-004 | Busy-Time Availability Feed | Automation | FEAT-04 (Two-Way Calendar Sync) | Collaboration/concurrency |
| FEAT-04.SPEC-005 | Booking-to-Calendar Sync | Automation | FEAT-04 (Two-Way Calendar Sync) | Background processing; Collaboration/concurrency |
| FEAT-04.SPEC-006 | Sync Health Monitor & Reconciliation | Automation | FEAT-04 (Two-Way Calendar Sync) | Background processing; Collaboration/concurrency |
| FEAT-04.SPEC-007 | Calendar Reconnection Alert | Notification | FEAT-04 (Two-Way Calendar Sync) | Notifications (email/push/SMS) |
| FEAT-04.SPEC-008 | Calendar Connection Rules | Logic/Rule | FEAT-04 (Two-Way Calendar Sync) | Collaboration/concurrency |
| FEAT-05.SPEC-001 | Public Booking Page (Landing & Service List) | Screen | FEAT-05 (Public Booking Page & Booking Flow) | Offline; Collaboration/concurrency |
| FEAT-05.SPEC-002 | Slot Selection | Screen | FEAT-05 (Public Booking Page & Booking Flow) | -- |
| FEAT-05.SPEC-003 | Client Details & Consent | Screen | FEAT-05 (Public Booking Page & Booking Flow) | Collaboration/concurrency; Compliance/privacy |
| FEAT-05.SPEC-004 | Policy Acknowledgment & Deposit Checkout | Screen | FEAT-05 (Public Booking Page & Booking Flow) | Offline; Payments/billing |
| FEAT-05.SPEC-005 | Booking Confirmation | Screen | FEAT-05 (Public Booking Page & Booking Flow) | Offline |
| FEAT-05.SPEC-006 | Slot Hold & Re-Validation at Checkout | Automation | FEAT-05 (Public Booking Page & Booking Flow) | Collaboration/concurrency |
| FEAT-05.SPEC-007 | Booking Details Field Validation | Logic/Rule | FEAT-05 (Public Booking Page & Booking Flow) | -- |
| FEAT-05.SPEC-008 | Booking Page Availability Gate | Logic/Rule | FEAT-05 (Public Booking Page & Booking Flow) | -- |
| FEAT-05.SPEC-009 | Policy Acknowledgment Capture & Integrity Check | Logic/Rule | FEAT-05 (Public Booking Page & Booking Flow) | -- |
| FEAT-06.SPEC-001 | Access Link Request | Screen | FEAT-06 (Client Booking Identity) | Offline; Collaboration/concurrency |
| FEAT-06.SPEC-002 | Access Link Validation & Redemption | Automation | FEAT-06 (Client Booking Identity) | Background processing; Collaboration/concurrency |
| FEAT-06.SPEC-003 | My Bookings List | Screen | FEAT-06 (Client Booking Identity) | Offline; Collaboration/concurrency |
| FEAT-06.SPEC-004 | Booking Detail via Manage Link | Screen | FEAT-06 (Client Booking Identity) | Offline; Collaboration/concurrency |
| FEAT-06.SPEC-005 | Consent & Email Preferences | Screen | FEAT-06 (Client Booking Identity) | Offline; Collaboration/concurrency; Compliance/privacy |
| FEAT-06.SPEC-006 | Access Link Delivery | Notification | FEAT-06 (Client Booking Identity) | Notifications (email/push/SMS) |
| FEAT-06.SPEC-007 | Access Link Lifecycle & Scope Rules | Logic/Rule | FEAT-06 (Client Booking Identity) | -- |
| FEAT-06.SPEC-008 | Client Identity & Privacy Isolation Rule | Logic/Rule | FEAT-06 (Client Booking Identity) | Compliance/privacy |
| FEAT-07.SPEC-001 | Deposit Payment | Screen | FEAT-07 (Deposit Payment at Booking) | Payments/billing; Collaboration/concurrency; Internationalization |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Automation | FEAT-07 (Deposit Payment at Booking) | Real-time; Payments/billing; Collaboration/concurrency |
| FEAT-07.SPEC-003 | Deposit Amount & Eligibility Rules | Logic/Rule | FEAT-07 (Deposit Payment at Booking) | Payments/billing |
| FEAT-07.SPEC-004 | Payment Outcome Consistency & Idempotency | Logic/Rule | FEAT-07 (Deposit Payment at Booking) | Payments/billing |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | Integration | FEAT-07 (Deposit Payment at Booking) | Payments/billing; Third-party integrations |
| FEAT-08.SPEC-001 | Booking Confirmation Message | Notification | FEAT-08 (Automated Booking Messaging) | Notifications (email/push/SMS); Import/export |
| FEAT-08.SPEC-002 | Appointment Reminder Message | Notification | FEAT-08 (Automated Booking Messaging) | Notifications (email/push/SMS) |
| FEAT-08.SPEC-003 | Reminder Reply Acknowledgment | Screen | FEAT-08 (Automated Booking Messaging) | -- |
| FEAT-08.SPEC-004 | Booking Change & Refund Notice | Notification | FEAT-08 (Automated Booking Messaging) | Payments/billing; Notifications (email/push/SMS); Collaboration/concurrency |
| FEAT-08.SPEC-005 | Pro Booking Activity Notification | Notification | FEAT-08 (Automated Booking Messaging) | Real-time; Notifications (email/push/SMS) |
| FEAT-08.SPEC-006 | Pro Attention Alert | Notification | FEAT-08 (Automated Booking Messaging) | Notifications (email/push/SMS) |
| FEAT-08.SPEC-007 | Reminder Scheduling & Timing Window Enforcement | Automation | FEAT-08 (Automated Booking Messaging) | Background processing; Collaboration/concurrency |
| FEAT-08.SPEC-008 | Reminder Reply Routing | Automation | FEAT-08 (Automated Booking Messaging) | Collaboration/concurrency |
| FEAT-08.SPEC-009 | Message Delivery Retry & Fallback | Automation | FEAT-08 (Automated Booking Messaging) | Collaboration/concurrency |
| FEAT-08.SPEC-010 | Booking-Specific Manage Link Issuance | Automation | FEAT-08 (Automated Booking Messaging) | Background processing; Collaboration/concurrency |
| FEAT-08.SPEC-011 | Messaging Consent & Channel Selection Rule | Logic/Rule | FEAT-08 (Automated Booking Messaging) | Collaboration/concurrency; Compliance/privacy |
| FEAT-08.SPEC-012 | Transactional Text Messaging Capability | Integration | FEAT-08 (Automated Booking Messaging) | Third-party integrations |
| FEAT-08.SPEC-013 | Transactional Email Capability | Integration | FEAT-08 (Automated Booking Messaging) | Third-party integrations |
| FEAT-09.SPEC-001 | Cancellation Policy Setup | Screen | FEAT-09 (Cancellation & No-Show Policy Engine) | Collaboration/concurrency |
| FEAT-09.SPEC-002 | Policy Versioning & Cutoff Rendering | Logic/Rule | FEAT-09 (Cancellation & No-Show Policy Engine) | Collaboration/concurrency |
| FEAT-09.SPEC-003 | Deposit Outcome Rules | Logic/Rule | FEAT-09 (Cancellation & No-Show Policy Engine) | Payments/billing |
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | Automation | FEAT-09 (Cancellation & No-Show Policy Engine) | Background processing; Collaboration/concurrency |
| FEAT-09.SPEC-005 | Automatic Deposit Refund | Integration | FEAT-09 (Cancellation & No-Show Policy Engine) | Payments/billing; Third-party integrations |
| FEAT-09.SPEC-006 | Refund Idempotency & Retry Rule | Logic/Rule | FEAT-09 (Cancellation & No-Show Policy Engine) | Payments/billing |
| FEAT-10.SPEC-001 | Cancel Booking | Screen | FEAT-10 (Client-Initiated Cancel/Reschedule) | Offline; Collaboration/concurrency |
| FEAT-10.SPEC-002 | Reschedule -- Select New Time | Screen | FEAT-10 (Client-Initiated Cancel/Reschedule) | Offline; Collaboration/concurrency |
| FEAT-10.SPEC-003 | Reschedule -- Outcome & Confirm | Screen | FEAT-10 (Client-Initiated Cancel/Reschedule) | Offline; Collaboration/concurrency |
| FEAT-10.SPEC-004 | Booking Update Commit | Automation | FEAT-10 (Client-Initiated Cancel/Reschedule) | Background processing; Collaboration/concurrency |
| FEAT-10.SPEC-005 | Cancellation Window & Eligibility Rule | Logic/Rule | FEAT-10 (Client-Initiated Cancel/Reschedule) | Collaboration/concurrency |
| FEAT-10.SPEC-006 | Cancellation/Reschedule Notification | Notification | FEAT-10 (Client-Initiated Cancel/Reschedule) | Notifications (email/push/SMS) |
| FEAT-11.SPEC-001 | No-Show Mark & Undo Prompt | Screen | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Offline; Collaboration/concurrency |
| FEAT-11.SPEC-002 | No-Show Marking & Deposit Forfeiture | Automation | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Payments/billing; Collaboration/concurrency |
| FEAT-11.SPEC-003 | No-Show Mark Undo | Automation | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Collaboration/concurrency |
| FEAT-11.SPEC-004 | No-Show Marking Window & Authorization Rules | Logic/Rule | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Collaboration/concurrency |
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | Screen | FEAT-12 (Pro Daily Schedule Dashboard) | Offline; Collaboration/concurrency |
| FEAT-12.SPEC-002 | Attention List | Screen | FEAT-12 (Pro Daily Schedule Dashboard) | Offline |
| FEAT-12.SPEC-003 | Past Bookings Browse | Screen | FEAT-12 (Pro Daily Schedule Dashboard) | Offline; Collaboration/concurrency |
| FEAT-12.SPEC-004 | Auto-Completion Sweep | Automation | FEAT-12 (Pro Daily Schedule Dashboard) | Background processing; Collaboration/concurrency |
| FEAT-12.SPEC-005 | Attention Flag Aggregation | Automation | FEAT-12 (Pro Daily Schedule Dashboard) | Background processing; Collaboration/concurrency |
| FEAT-12.SPEC-006 | Booking Completion Rules | Logic/Rule | FEAT-12 (Pro Daily Schedule Dashboard) | Collaboration/concurrency |
| FEAT-12.SPEC-007 | Balance Due & Status Display Rules | Logic/Rule | FEAT-12 (Pro Daily Schedule Dashboard) | Payments/billing |
| FEAT-12.SPEC-008 | Dashboard Access Authorization | Logic/Rule | FEAT-12 (Pro Daily Schedule Dashboard) | Authentication |
| FEAT-13.SPEC-001 | Client Record Detail | Screen | FEAT-13 (Client Record Management) | Offline; Collaboration/concurrency |
| FEAT-13.SPEC-002 | Client Contact Edit | Screen | FEAT-13 (Client Record Management) | Offline; Collaboration/concurrency |
| FEAT-13.SPEC-003 | Client Deletion Confirmation | Screen | FEAT-13 (Client Record Management) | Offline; Compliance/privacy |
| FEAT-13.SPEC-004 | Client Deletion Execution | Automation | FEAT-13 (Client Record Management) | Collaboration/concurrency; Compliance/privacy |
| FEAT-13.SPEC-005 | Client Field Validation & Access Rules | Logic/Rule | FEAT-13 (Client Record Management) | Collaboration/concurrency |
| FEAT-13.SPEC-006 | Deletion Eligibility & Retention Rule | Logic/Rule | FEAT-13 (Client Record Management) | Collaboration/concurrency; Compliance/privacy |
| FEAT-14.SPEC-001 | Consent & Preferences | Screen | FEAT-14 (Messaging Consent Management) | Offline; Collaboration/concurrency; Compliance/privacy |
| FEAT-14.SPEC-002 | Opt-Out Link Landing | Screen | FEAT-14 (Messaging Consent Management) | Compliance/privacy |
| FEAT-14.SPEC-003 | Consent Capture at Booking | Automation | FEAT-14 (Messaging Consent Management) | Collaboration/concurrency; Compliance/privacy |
| FEAT-14.SPEC-004 | Opt-Out / STOP Processing | Automation | FEAT-14 (Messaging Consent Management) | Collaboration/concurrency; Compliance/privacy |
| FEAT-14.SPEC-005 | Consent Re-Grant Action | Automation | FEAT-14 (Messaging Consent Management) | Collaboration/concurrency; Compliance/privacy |
| FEAT-14.SPEC-006 | Concurrent Consent Update Resolution | Logic/Rule | FEAT-14 (Messaging Consent Management) | Collaboration/concurrency; Compliance/privacy |
| FEAT-14.SPEC-007 | Textability Determination Rule | Logic/Rule | FEAT-14 (Messaging Consent Management) | -- |
| FEAT-14.SPEC-008 | Phone Number Change Consent Invalidation Rule | Logic/Rule | FEAT-14 (Messaging Consent Management) | Compliance/privacy |
| FEAT-14.SPEC-009 | Opt-Out Confirmation Message | Notification | FEAT-14 (Messaging Consent Management) | Notifications (email/push/SMS); Compliance/privacy |
| FEAT-15.SPEC-001 | Setup Wizard Shell, Step Navigation & Guidance | Screen | FEAT-15 (Pro Onboarding & Setup Wizard) | Complex forms (10+ fields); Collaboration/concurrency |
| FEAT-15.SPEC-002 | Cancellation Policy Default & First-Version Setup Step | Screen | FEAT-15 (Pro Onboarding & Setup Wizard) | Complex forms (10+ fields); Collaboration/concurrency; Internationalization |
| FEAT-15.SPEC-003 | Go-Live Preview & Booking Link Hand-Over | Screen | FEAT-15 (Pro Onboarding & Setup Wizard) | Offline; Complex forms (10+ fields); Collaboration/concurrency |
| FEAT-15.SPEC-004 | Setup Progress Tracking & Resume | Automation | FEAT-15 (Pro Onboarding & Setup Wizard) | Authentication; Collaboration/concurrency |
| FEAT-15.SPEC-005 | Go-Live Evaluation & Booking Link Activation | Automation | FEAT-15 (Pro Onboarding & Setup Wizard) | Collaboration/concurrency |
| FEAT-15.SPEC-006 | Setup Step Order & Optional-Step Rules | Logic/Rule | FEAT-15 (Pro Onboarding & Setup Wizard) | -- |
| FEAT-15.SPEC-007 | Go-Live Prerequisite Rule (XBR-26 Authority) | Logic/Rule | FEAT-15 (Pro Onboarding & Setup Wizard) | -- |
| FEAT-15.SPEC-008 | Onboarding Welcome Confirmation | Notification | FEAT-15 (Pro Onboarding & Setup Wizard) | Notifications (email/push/SMS) |
| FEAT-16.SPEC-001 | Booking Activity Timeline | Screen | FEAT-16 (Booking & Payment Activity Record) | Offline; Collaboration/concurrency |
| FEAT-16.SPEC-002 | Activity Event Recording | Automation | FEAT-16 (Booking & Payment Activity Record) | Background processing |
| FEAT-16.SPEC-003 | Card-Issuer Dispute Integration | Integration | FEAT-16 (Booking & Payment Activity Record) | Payments/billing; Third-party integrations |
| FEAT-16.SPEC-004 | Dispute Summary Download | Automation | FEAT-16 (Booking & Payment Activity Record) | Payments/billing; Import/export; Collaboration/concurrency |
| FEAT-16.SPEC-005 | Activity Record Immutability & Visibility Rules | Logic/Rule | FEAT-16 (Booking & Payment Activity Record) | Compliance/privacy |
| FEAT-17.SPEC-001 | Create/Edit Time Block | Screen | FEAT-17 (Manual Time Blocking) | Offline; Collaboration/concurrency |
| FEAT-17.SPEC-002 | Manage Time Blocks | Screen | FEAT-17 (Manual Time Blocking) | Offline; Collaboration/concurrency |
| FEAT-17.SPEC-003 | Time Block Conflict Review | Screen | FEAT-17 (Manual Time Blocking) | -- |
| FEAT-17.SPEC-004 | Time Block Save Commit & Conflict Detection | Automation | FEAT-17 (Manual Time Blocking) | Background processing; Collaboration/concurrency |
| FEAT-17.SPEC-005 | Recurring Time Block Occurrence Generation | Automation | FEAT-17 (Manual Time Blocking) | Background processing; Collaboration/concurrency |
| FEAT-17.SPEC-006 | Time Block Conflict Resolution Commit | Automation | FEAT-17 (Manual Time Blocking) | Background processing; Collaboration/concurrency |
| FEAT-17.SPEC-007 | Time Block Removal & Expiry | Automation | FEAT-17 (Manual Time Blocking) | Background processing; Collaboration/concurrency |
| FEAT-17.SPEC-008 | Time Block Validation & Conflict Handling Rules | Logic/Rule | FEAT-17 (Manual Time Blocking) | -- |
| FEAT-18.SPEC-001 | Subscribe Screen | Screen | FEAT-18 (Pro Subscription Billing & Account Management) | Collaboration/concurrency |
| FEAT-18.SPEC-002 | Billing & Subscription Management Screen | Screen | FEAT-18 (Pro Subscription Billing & Account Management) | Offline; Payments/billing; Collaboration/concurrency |
| FEAT-18.SPEC-003 | Subscription Renewal & Payment-Failure Processing | Automation | FEAT-18 (Pro Subscription Billing & Account Management) | Background processing; Payments/billing; Collaboration/concurrency |
| FEAT-18.SPEC-004 | Subscription-Lapse Account Pause Trigger | Automation | FEAT-18 (Pro Subscription Billing & Account Management) | Background processing; Payments/billing; Collaboration/concurrency |
| FEAT-18.SPEC-005 | Subscription Billing Rules | Logic/Rule | FEAT-18 (Pro Subscription Billing & Account Management) | Payments/billing; Collaboration/concurrency |
| FEAT-18.SPEC-006 | Subscription Billing Integration | Integration | FEAT-18 (Pro Subscription Billing & Account Management) | Payments/billing; Third-party integrations |
| FEAT-18.SPEC-007 | Subscription Billing Notifications | Notification | FEAT-18 (Pro Subscription Billing & Account Management) | Payments/billing; Notifications (email/push/SMS) |
| FEAT-19.SPEC-001 | Pro Account Lookup & Support Session Entry | Screen | FEAT-19 (Platform Support Read-Only Access) | Authentication |
| FEAT-19.SPEC-002 | Support View Logging | Automation | FEAT-19 (Platform Support Read-Only Access) | Collaboration/concurrency |
| FEAT-19.SPEC-003 | Support Access Log | Screen | FEAT-19 (Platform Support Read-Only Access) | -- |
| FEAT-19.SPEC-004 | Support Session Scope & Access Rules | Logic/Rule | FEAT-19 (Platform Support Read-Only Access) | Authentication; Collaboration/concurrency |
| FEAT-20.SPEC-001 | Join Waitlist | Screen | FEAT-20 (Waitlist for Cancelled Slots) | -- |
| FEAT-20.SPEC-002 | My Waitlists | Screen | FEAT-20 (Waitlist for Cancelled Slots) | Offline |
| FEAT-20.SPEC-003 | Waitlist Entry Validation & Limits | Logic/Rule | FEAT-20 (Waitlist for Cancelled Slots) | -- |
| FEAT-20.SPEC-004 | Waitlist Priority & Claim Window Rule | Logic/Rule | FEAT-20 (Waitlist for Cancelled Slots) | -- |
| FEAT-20.SPEC-005 | Cancellation-Triggered Waitlist Matching | Automation | FEAT-20 (Waitlist for Cancelled Slots) | Collaboration/concurrency |
| FEAT-20.SPEC-006 | Waitlist Claim Conversion | Automation | FEAT-20 (Waitlist for Cancelled Slots) | Collaboration/concurrency |
| FEAT-20.SPEC-007 | Waitlist Entry Expiry | Automation | FEAT-20 (Waitlist for Cancelled Slots) | Collaboration/concurrency |
| FEAT-20.SPEC-008 | Waitlist Opening Notification | Notification | FEAT-20 (Waitlist for Cancelled Slots) | Notifications (email/push/SMS) |
| FEAT-20.SPEC-009 | Waitlist Expiry Notification | Notification | FEAT-20 (Waitlist for Cancelled Slots) | Notifications (email/push/SMS) |
| FEAT-21.SPEC-001 | Set Up Recurring Series | Screen | FEAT-21 (Recurring/Standing Appointments) | -- |
| FEAT-21.SPEC-002 | My Recurring Series | Screen | FEAT-21 (Recurring/Standing Appointments) | Collaboration/concurrency |
| FEAT-21.SPEC-003 | Recurring Series Setup & Generation Limits | Logic/Rule | FEAT-21 (Recurring/Standing Appointments) | -- |
| FEAT-21.SPEC-004 | Occurrence Generation & Conflict Handling | Automation | FEAT-21 (Recurring/Standing Appointments) | Background processing; Collaboration/concurrency |
| FEAT-21.SPEC-005 | Occurrence Deposit Request & Release | Automation | FEAT-21 (Recurring/Standing Appointments) | Background processing; Payments/billing; Collaboration/concurrency |
| FEAT-21.SPEC-006 | Series & Occurrence Cancellation Rules | Logic/Rule | FEAT-21 (Recurring/Standing Appointments) | Collaboration/concurrency |
| FEAT-21.SPEC-007 | Occurrence Generated Notification | Notification | FEAT-21 (Recurring/Standing Appointments) | Notifications (email/push/SMS) |
| FEAT-21.SPEC-008 | Occurrence Time Change Advance Notice | Notification | FEAT-21 (Recurring/Standing Appointments) | Notifications (email/push/SMS) |
| FEAT-21.SPEC-009 | Occurrence Deposit Lifecycle Notification | Notification | FEAT-21 (Recurring/Standing Appointments) | Payments/billing; Notifications (email/push/SMS) |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | Screen | FEAT-21 (Recurring/Standing Appointments) | Collaboration/concurrency |
| FEAT-22.SPEC-001 | Balance Payment | Screen | FEAT-22 (In-App Balance Payment) | Payments/billing; Collaboration/concurrency; Internationalization |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Automation | FEAT-22 (In-App Balance Payment) | Real-time; Payments/billing; Collaboration/concurrency |
| FEAT-22.SPEC-003 | Balance Amount & Eligibility Rules | Logic/Rule | FEAT-22 (In-App Balance Payment) | Payments/billing; Collaboration/concurrency |
| FEAT-22.SPEC-004 | Balance Payment Outcome Consistency & Cancellation Contention | Logic/Rule | FEAT-22 (In-App Balance Payment) | Payments/billing; Collaboration/concurrency |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Integration | FEAT-22 (In-App Balance Payment) | Payments/billing; Third-party integrations |
| FEAT-23.SPEC-001 | Tip Selection | Screen | FEAT-23 (Tipping at Checkout) | Payments/billing; Collaboration/concurrency |
| FEAT-23.SPEC-002 | Tip Amount Validation | Logic/Rule | FEAT-23 (Tipping at Checkout) | Payments/billing |
| FEAT-23.SPEC-003 | Tip Payout & Refund Rule | Logic/Rule | FEAT-23 (Tipping at Checkout) | Payments/billing; Collaboration/concurrency |
| FEAT-24.SPEC-001 | Client Search & Filter | Screen | FEAT-24 (Client List Search & Filter) | Offline; Search; Collaboration/concurrency |
| FEAT-24.SPEC-002 | Search Match & Filter Derivation Rules | Logic/Rule | FEAT-24 (Client List Search & Filter) | Search |
| FEAT-25.SPEC-001 | Insights Summary Screen | Screen | FEAT-25 (Booking & Revenue Insights) | Offline; Collaboration/concurrency |
| FEAT-25.SPEC-002 | Period Insights Aggregation | Automation | FEAT-25 (Booking & Revenue Insights) | Collaboration/concurrency |
| FEAT-25.SPEC-003 | Insights Derivation, Period & Access Rules | Logic/Rule | FEAT-25 (Booking & Revenue Insights) | -- |
| FEAT-25.SPEC-004 | Historical Aggregate Maintenance | Automation | FEAT-25 (Booking & Revenue Insights) | Background processing; Collaboration/concurrency |
| FEAT-26.SPEC-001 | WhatsApp Channel Preference | Screen | FEAT-26 (WhatsApp Reminders) | Offline; Collaboration/concurrency |
| FEAT-26.SPEC-002 | WhatsApp Send & Delivery-Status Capability | Integration | FEAT-26 (WhatsApp Reminders) | Third-party integrations |
| FEAT-26.SPEC-003 | WhatsApp Delivery Fallback | Automation | FEAT-26 (WhatsApp Reminders) | Collaboration/concurrency |
| FEAT-26.SPEC-004 | WhatsApp Channel Eligibility & Consent Rule | Logic/Rule | FEAT-26 (WhatsApp Reminders) | Compliance/privacy |
| FEAT-27.SPEC-001 | Profile & Booking Page Settings | Screen | FEAT-27 (Pro Profile & Booking Page Settings) | File upload; Collaboration/concurrency |
| FEAT-27.SPEC-002 | Booking Link Rename | Screen | FEAT-27 (Pro Profile & Booking Page Settings) | Collaboration/concurrency |
| FEAT-27.SPEC-003 | Timezone & Currency Settings | Screen | FEAT-27 (Pro Profile & Booking Page Settings) | Internationalization |
| FEAT-27.SPEC-004 | Pause Bookings | Screen | FEAT-27 (Pro Profile & Booking Page Settings) | Collaboration/concurrency |
| FEAT-27.SPEC-005 | Notification Preferences | Screen | FEAT-27 (Pro Profile & Booking Page Settings) | Collaboration/concurrency |
| FEAT-27.SPEC-006 | Help Request | Screen | FEAT-27 (Pro Profile & Booking Page Settings) | -- |
| FEAT-27.SPEC-007 | Booking Link Name Validation & Uniqueness Rule | Logic/Rule | FEAT-27 (Pro Profile & Booking Page Settings) | Collaboration/concurrency |
| FEAT-27.SPEC-008 | Currency Lock Rule | Logic/Rule | FEAT-27 (Pro Profile & Booking Page Settings) | Internationalization |
| FEAT-27.SPEC-009 | Pause State Precedence Rule | Logic/Rule | FEAT-27 (Pro Profile & Booking Page Settings) | -- |
| FEAT-27.SPEC-010 | Booking Link Forwarding & Reservation Expiry | Automation | FEAT-27 (Pro Profile & Booking Page Settings) | Background processing; Collaboration/concurrency |
| FEAT-27.SPEC-011 | Automatic Pause Resume | Automation | FEAT-27 (Pro Profile & Booking Page Settings) | Background processing; Collaboration/concurrency |
| FEAT-27.SPEC-012 | Profile Photo Storage Capability | Integration | FEAT-27 (Pro Profile & Booking Page Settings) | File upload; Third-party integrations |
| FEAT-27.SPEC-013 | Help Request Acknowledgment | Notification | FEAT-27 (Pro Profile & Booking Page Settings) | Notifications (email/push/SMS) |
| FEAT-28.SPEC-001 | Payout Account Connection | Screen | FEAT-28 (Payout Account Connection & Payout Visibility) | Payments/billing |
| FEAT-28.SPEC-002 | Payout Status & Money Dashboard | Screen | FEAT-28 (Payout Account Connection & Payout Visibility) | Real-time; Offline; Payments/billing |
| FEAT-28.SPEC-003 | Payout Account Status Processing | Automation | FEAT-28 (Payout Account Connection & Payout Visibility) | Payments/billing; Collaboration/concurrency |
| FEAT-28.SPEC-004 | Payout Account Eligibility & Constraints | Logic/Rule | FEAT-28 (Payout Account Connection & Payout Visibility) | Payments/billing; Collaboration/concurrency |
| FEAT-28.SPEC-005 | Money List Composition & Net Calculation | Logic/Rule | FEAT-28 (Payout Account Connection & Payout Visibility) | -- |
| FEAT-28.SPEC-006 | Payout Account Connection & Verification | Integration | FEAT-28 (Payout Account Connection & Payout Visibility) | Payments/billing; Third-party integrations |
| FEAT-28.SPEC-007 | Payout Status Notification | Notification | FEAT-28 (Payout Account Connection & Payout Visibility) | Payments/billing; Notifications (email/push/SMS) |
| FEAT-29.SPEC-001 | Sign-In Screen | Screen | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Collaboration/concurrency |
| FEAT-29.SPEC-002 | Account Recovery Screen | Screen | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Collaboration/concurrency |
| FEAT-29.SPEC-003 | Account & Sign-In Settings Screen | Screen | FEAT-29 (Pro Sign-In & Account Lifecycle) | Offline; Authentication; Collaboration/concurrency |
| FEAT-29.SPEC-004 | Data Export Screen | Screen | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Import/export; Collaboration/concurrency |
| FEAT-29.SPEC-005 | Account Closure & Reopening Screen | Screen | FEAT-29 (Pro Sign-In & Account Lifecycle) | Offline; Authentication; Collaboration/concurrency; Compliance/privacy |
| FEAT-29.SPEC-006 | Session & Device Management | Automation | FEAT-29 (Pro Sign-In & Account Lifecycle) | Background processing; Authentication; Collaboration/concurrency |
| FEAT-29.SPEC-007 | Data Export Generation | Automation | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Import/export; Collaboration/concurrency |
| FEAT-29.SPEC-008 | Account Closure Orchestration | Automation | FEAT-29 (Pro Sign-In & Account Lifecycle) | Background processing; Authentication; Collaboration/concurrency; Compliance/privacy |
| FEAT-29.SPEC-009 | Account Reopening | Automation | FEAT-29 (Pro Sign-In & Account Lifecycle) | Background processing; Authentication; Collaboration/concurrency |
| FEAT-29.SPEC-010 | Contact-Detail Change Processing | Automation | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Collaboration/concurrency |
| FEAT-29.SPEC-011 | Sign-In & Recovery Rules | Logic/Rule | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication |
| FEAT-29.SPEC-012 | Contact-Change Confirmation Rules | Logic/Rule | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication |
| FEAT-29.SPEC-013 | Account Closure & Retention Rules | Logic/Rule | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Collaboration/concurrency; Compliance/privacy |
| FEAT-29.SPEC-014 | Sign-In Code Notification | Notification | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Notifications (email/push/SMS) |
| FEAT-29.SPEC-015 | New-Device Sign-In Alert | Notification | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Notifications (email/push/SMS) |
| FEAT-29.SPEC-016 | Contact-Change Confirmation Notification | Notification | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Notifications (email/push/SMS) |
| FEAT-29.SPEC-017 | Account Closure & Deletion Notifications | Notification | FEAT-29 (Pro Sign-In & Account Lifecycle) | Authentication; Notifications (email/push/SMS); Compliance/privacy |
| FEAT-30.SPEC-001 | Cancel Booking (Pro-Initiated) | Screen | FEAT-30 (Pro Booking Management) | Collaboration/concurrency |
| FEAT-30.SPEC-002 | Reschedule Booking (Pro-Initiated) | Screen | FEAT-30 (Pro Booking Management) | Collaboration/concurrency |
| FEAT-30.SPEC-003 | Goodwill Deposit Refund | Screen | FEAT-30 (Pro Booking Management) | Payments/billing |
| FEAT-30.SPEC-004 | Book Client In | Screen | FEAT-30 (Pro Booking Management) | Offline |
| FEAT-30.SPEC-005 | Cancel Several Bookings at Once | Screen | FEAT-30 (Pro Booking Management) | Collaboration/concurrency |
| FEAT-30.SPEC-006 | Pro Booking Action Rules | Logic/Rule | FEAT-30 (Pro Booking Management) | Collaboration/concurrency |
| FEAT-30.SPEC-007 | Pro Cancel/Reschedule Commit | Automation | FEAT-30 (Pro Booking Management) | Background processing; Collaboration/concurrency |
| FEAT-30.SPEC-008 | Bulk Cancellation Commit | Automation | FEAT-30 (Pro Booking Management) | Background processing; Collaboration/concurrency |
| FEAT-30.SPEC-009 | Goodwill Refund Commit | Automation | FEAT-30 (Pro Booking Management) | Background processing; Payments/billing; Collaboration/concurrency |
| FEAT-30.SPEC-010 | Pro-Created Booking & Deposit Request Hold | Automation | FEAT-30 (Pro Booking Management) | Background processing; Payments/billing; Collaboration/concurrency |
| FEAT-30.SPEC-011 | Goodwill & Bulk-Cancellation Refund Execution | Integration | FEAT-30 (Pro Booking Management) | Payments/billing; Third-party integrations |
| FEAT-30.SPEC-012 | Pro Action Client Notice | Notification | FEAT-30 (Pro Booking Management) | Notifications (email/push/SMS); Collaboration/concurrency |
| FEAT-30.SPEC-013 | Deposit Request & Expiry Notice | Notification | FEAT-30 (Pro Booking Management) | Payments/billing; Notifications (email/push/SMS) |

## 7. Demand-Side Inputs

Verbatim quotes, each attributed to its file and section. No summary or paraphrase.

### From BRIEF.md -- Scale & Non-Functional Expectations

> - **Volume:** a few hundred pros in year one. Each pro has roughly 100–500 clients and 20–40 bookings a week.
> - **Devices & platforms:** mobile-first web for both roles. The pro checks it on their phone between clients, and the client books from inside the Instagram in-app browser, so it must work well there. Desktop is a bonus for the pro's setup screens.
> - **Geography:** US first; the UK, Canada and Australia are the obvious next markets, so timezone and currency must not be hard-coded.
> - **Correctness & reliability:** reliability matters more than features. Booking and payment must be correct, always. It must never silently double-book or lose a deposit; if it does, the pro leaves and tells their friends. No specific uptime number was given.
> - **Privacy:** a client's data is visible only to their pro (and to the client themselves for their own bookings), never to any other pro or client. A pro can delete a client's record on request.
> -- BRIEF.md, ## Scale & Non-Functional Expectations

### From BRIEF.md -- Ecosystem & Integrations

> - **Pro's personal calendar (Google Calendar and Apple Calendar — both matter):** two-way. Busy times there block Chairtime availability, and bookings made in Chairtime appear there.
> - **SMS text messaging:** confirmations and reminders by text in v1, sent only with the client's explicit opt-in at booking. Email confirmations are acceptable as a fallback. WhatsApp is a nice-to-have later, not v1.
> - **Established card payment processor:** takes client deposits and pays out to the pro, and owns all card data. Also used for the pro's subscription billing.
> - **Instagram:** only the place the pro's booking link lives. No Instagram integration for v1.
> - Otherwise it stands alone.
> -- BRIEF.md, ## Ecosystem & Integrations

### From assumptions-constraints.md -- Non-Functional Expectations

> - **ID:** ASMP-21
> - **Responsiveness: available slots appear within roughly one second of a service selection, and a full booking (selection through paid confirmation) completes in under one minute.** — Basis: BRIEF.md's Vision states the one-minute booking benchmark directly, and its Scale & Non-Functional Expectations section makes correctness and speed the product's defining quality bar.
> - **ID:** ASMP-22
> - **Data volume and growth: a few hundred pros in year one, each with roughly 100–500 clients and 20–40 bookings a week, with the product staying equally responsive as pros accumulate history over multiple years.** — Basis: BRIEF.md's Scale & Non-Functional Expectations, stated directly.
> - **ID:** ASMP-23
> - **Privacy posture: a client's data is visible only to their own pro and to themselves; a pro can permanently delete a client's record on request.** — Basis: BRIEF.md's Privacy and Constraints sections, stated directly.
> - **ID:** ASMP-24
> - **Compliance: US SMS-consent rules apply to all client texting (explicit opt-in captured at booking, honored immediately on opt-out); no health-data regime applies, since intake and clinical data are explicitly out of scope.** — Basis: BRIEF.md's Constraints, "Regulated-domain confirmation" section, stated directly.
> - **ID:** ASMP-25
> - **Geography and localization: timezone and currency are per-account configuration from day one, never hard-coded, so expansion beyond the US requires no structural rework.** — Basis: BRIEF.md's Scale & Non-Functional Expectations: "the UK, Canada and Australia are the obvious next markets, so timezone and currency must not be hard-coded."
> - **ID:** ASMP-26
> - **Reliability is expressed as a correctness bar, not a numeric uptime target: the product must never silently double-book a slot or lose a deposit.** — Basis: BRIEF.md's Scale & Non-Functional Expectations states directly that "no specific uptime number was given" and frames correctness, not uptime percentage, as the requirement. [RESEARCH-INFORMED: glitches and crashes in the core scheduling workflow are reported for two competitors at comparable booking volumes (MEDIUM confidence), so correctness under everyday use is a documented market gap]
> - **ID:** ASMP-27
> - **Offline and loading posture: anything that books, pays, cancels, refunds or marks a no-show needs a live connection and says so plainly when it is missing; the pro's most recently loaded schedule, client list and money list stay readable offline; every screen that waits shows an in-place indicator rather than a blank page, and nothing appears tappable before real data has loaded.** — Basis: BRIEF.md's correctness bar ("never silently double-book or lose a deposit") and the decomposition checklist's Offline and Loading items; decided product-wide so every feature's States field follows one rule. [AUDIT-ADDED: 4 -- Offline/Degraded and Loading concerns needed a product-level decision]
> - **ID:** ASMP-28
> - **Accessibility baseline: every client and pro screen is readable and fully operable at phone width inside a social-media in-app browser, with text that scales, sufficient contrast, controls large enough to tap reliably, and full use by screen-reader users; nothing relies on color alone (for example, the paid badge also carries a word).** — Basis: BRIEF.md's Scale & Non-Functional Expectations (mobile-first, Instagram in-app browser) and the decomposition checklist's Accessibility item. [AUDIT-ADDED: 4 -- Accessibility baseline was not decided in the draft]
> - **ID:** ASMP-29
> - **Messaging timing: automatic reminders reach clients only during reasonable daytime hours (roughly 8am–9pm in the pro's timezone), and confirmations arrive within about a minute of payment.** — Basis: BRIEF.md's Constraints ("reminders must respect that consent," citing US texting rules) and its Vision ("a confirmation text lands immediately"). [AUDIT-ADDED: 4 -- Compliance concern]
> - **ID:** ASMP-30
> - **Account protection: a pro's account, which holds every client's contact details, is protected by a one-time-code sign-in with new-device alerts; client access links are short-lived and open only that client's bookings with that one pro.** — Basis: BRIEF.md's Privacy section ("a client's data is visible only to their pro") and the decomposition checklist's Security and Privacy Posture item. [AUDIT-ADDED: 4 -- Security and Privacy Posture concern]
> -- assumptions-constraints.md, ## Non-Functional Expectations

### From assumptions-constraints.md -- Dependencies

> - **ID:** ASMP-31
> - **Payment-processing capability** — required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription. Without it, the product has no way to collect money or generate revenue at all; BRIEF.md's Constraints require that this capability, not the product's own code, owns all card data. [MODIFIED: connected payout accounts, identity verification, refunds and dispute notifications named explicitly after the value-flow audit]
> - **ID:** ASMP-32
> - **Transactional text-messaging capability, with email as a fallback channel** — required to send booking confirmations and pre-appointment reminders. Without it, the product cannot deliver the automatic-reminder promise that replaces the pro's manual texting habit; BRIEF.md's Ecosystem & Integrations names texting as the v1 channel with email as an acceptable fallback.
> - **ID:** ASMP-33
> - **Calendar-sync capability (reading and writing to a pro's personal calendar)** — required for the two-way sync described in BRIEF.md's Ecosystem & Integrations. Without it, the availability engine cannot account for a pro's real-world commitments outside Chairtime, directly threatening the "never double-book" correctness bar.
> - **ID:** ASMP-34
> - **A searchable, per-pro record store for services, bookings, clients, and payment outcomes** — required for the product to function at all across sessions; without persistent, per-account data, nothing booked, paid, or configured could be relied on the next time either the pro or the client returns.
> - **ID:** ASMP-35
> - **File storage capability for pro profile photos** — required for the photo shown on the booking page (FEAT-27). Without it, the booking page shows the pro's name only; the booking loop itself is unaffected. [AUDIT-ADDED: 3 -- the profile photo captured by the audit-added Pro Profile & Booking Page Settings needs somewhere to live]
> -- assumptions-constraints.md, ## Dependencies

### From feature-dependency-map.md -- External Touchpoints

Traced from `## Dependencies` in assumptions-constraints.md. ASMP-34 (a per-pro record store) is the product's own persistence, not an external capability, so it has no touchpoint row; Stage 4 addresses it directly. The Integration Specs column was completed batch by batch as each feature's Brief was validated, and the final analysis batch confirmed full coverage in both directions: every row below has at least one covering Integration spec, and every Integration spec in every Brief is cited here. The WhatsApp messaging row was added from a validated Brief's Integration spec (FEAT-26.SPEC-002); its citation trail is ASMP-32 and BRIEF.md's Ecosystem & Integrations.

| Capability Category | Features Involved | Integration Specs |
|---------------------|-------------------|-------------------|
| Payment processing — client card charges and refunds (deposits; from v1 balances; Later tips) (ASMP-31) | FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-23 | FEAT-07.SPEC-005 (deposit card authorization and capture, processor fee reporting, zero platform fee), FEAT-09.SPEC-005 (automatic full deposit refund on client cancellation outside the window and every Pro cancellation, drawing on the Pro's payout account, with not-yet-completable refunds reported back for retry), FEAT-30.SPEC-011 (goodwill refunds and the per-booking refund set of a Pro bulk cancellation, drawing on the Pro's payout account, with not-yet-completable refunds reported back for retry; a single Pro cancellation's refund is executed by FEAT-09.SPEC-005), FEAT-22.SPEC-005 (in-app balance card authorization and capture with zero platform fee, and the outbound full refund of a paid balance when either party cancels, per XBR-23); FEAT-23's optional tip is charged and refunded as part of that same balance payment through FEAT-22.SPEC-005 and needs no Integration spec of its own |
| Payment processing — connected payout accounts with identity and bank verification (ASMP-31) | FEAT-28, FEAT-15, FEAT-07, FEAT-22, FEAT-23 | FEAT-07.SPEC-005 (routes each captured deposit to the Pro's connected payout account), FEAT-28.SPEC-006 (hand-off into the processor's own identity and bank verification, action-required resolution, and inbound account-status, payout and processor-fee reporting; FEAT-15 reaches it through its getting-paid step), FEAT-22.SPEC-005 (routes each captured balance, including any FEAT-23 tip, to the Pro's connected payout account with zero platform fee) |
| Payment processing — card-issuer dispute notifications (ASMP-31) | FEAT-16, FEAT-12 | FEAT-16.SPEC-003 (inbound card-issuer dispute notice: flags the booking, sets the Deposit Transaction's Disputed overlay without erasing its outcome, and records the dispute event; FEAT-12 consumes the flag through FEAT-12.SPEC-005 and needs no Integration spec of its own) |
| Payment processing — Pro subscription billing (ASMP-31) | FEAT-18, FEAT-15 | FEAT-18.SPEC-006 (subscribe, payment-method update and cancel requests, including cancellation invoked by FEAT-29 account closure, plus inbound renewal outcomes; FEAT-15 reaches it through its subscription setup step) |
| Transactional text messaging (ASMP-32) | FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-20, FEAT-21, FEAT-26 | FEAT-08.SPEC-012 (text send and delivery-status reporting for every product text, including FEAT-06 access links, the FEAT-14 opt-out confirmation and inbound STOP replies consumed by FEAT-14.SPEC-004, and the FEAT-30 Pro-action notices and deposit requests); FEAT-15's go-live welcome confirmation (FEAT-15.SPEC-008) and FEAT-18's billing notices (FEAT-18.SPEC-007) are also delivered through FEAT-08.SPEC-012; FEAT-20's waitlist opening and expiry notices (FEAT-20.SPEC-008, FEAT-20.SPEC-009) and FEAT-29's sign-in code, new-device alert, contact-change and closure/deletion notices (FEAT-29.SPEC-014 to FEAT-29.SPEC-017) are also delivered through FEAT-08.SPEC-012; FEAT-21's occurrence-generated, occurrence time-change and occurrence deposit-link/release notices (FEAT-21.SPEC-007 to FEAT-21.SPEC-009) are also delivered through FEAT-08.SPEC-012; FEAT-26's WhatsApp fallback (FEAT-26.SPEC-003) re-sends a failed or unavailable WhatsApp message as a text through FEAT-08.SPEC-012 when the client has texting consent |
| Transactional email (fallback channel) (ASMP-32) | FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-18, FEAT-21, FEAT-26 | FEAT-08.SPEC-013 (fallback and consent-declined email send and delivery-status reporting, including FEAT-06 access links, FEAT-14 fallback routing and the FEAT-30 Pro-action notices and deposit requests for clients without texting consent); FEAT-18's billing notices (FEAT-18.SPEC-007) and FEAT-15's go-live welcome confirmation (FEAT-15.SPEC-008) are also delivered through FEAT-08.SPEC-013; FEAT-29's notices (FEAT-29.SPEC-014 to FEAT-29.SPEC-017) are delivered through FEAT-08.SPEC-013 as the email fallback, as are FEAT-20's waitlist notices (FEAT-20.SPEC-008, FEAT-20.SPEC-009) and FEAT-21's occurrence notices (FEAT-21.SPEC-007 to FEAT-21.SPEC-009) for clients without texting consent; FEAT-26's WhatsApp fallback (FEAT-26.SPEC-003) re-sends through FEAT-08.SPEC-013 when the client has no texting consent |
| Transactional WhatsApp messaging — optional client channel for confirmations, reminders and change notices, from Later (ASMP-32; BRIEF.md Ecosystem & Integrations: "WhatsApp is a nice-to-have later, not v1") | FEAT-26, FEAT-08, FEAT-14 | FEAT-26.SPEC-002 (WhatsApp send and inbound delivery-status reporting for FEAT-08's confirmation, reminder and change-notice content when the client has chosen WhatsApp and FEAT-26.SPEC-004 finds the channel eligible under channel-aware Messaging Consent owned by FEAT-14; failed or unavailable sends fall back through FEAT-26.SPEC-003 to FEAT-08.SPEC-012/FEAT-08.SPEC-013) |
| Calendar sync — reading busy time from and writing bookings to a Pro's personal calendar (ASMP-33) | FEAT-04, FEAT-03, FEAT-15 | FEAT-04.SPEC-003 (connection handshake, busy-time pull, booking write/move/remove), FEAT-03.SPEC-006 (busy-time consumption and degraded mode) |
| File storage — Pro profile photos (ASMP-35) | FEAT-27, FEAT-05 | FEAT-27.SPEC-012 (stores, replaces and serves the Pro's profile photo within the size/format limits; FEAT-05 reads the served photo on the booking page and needs no Integration spec of its own; the page still works without a photo) |

> -- feature-dependency-map.md, ## External Touchpoints
