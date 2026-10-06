# Part G2 — Architecture: Recommendation & Alternatives

This part carries the four architecture documents in reading order: evidence, option space, feasibility, decisions. The recommendation is the default build; alternatives are documented in full, with `Choose instead when` conditions, for this team to weigh and adopt deliberately where its constraints differ.

## The Evidence — Technical Profile


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


## The Option Space — Technology Landscape


# Technology Landscape

## 1. Research Scope

| Decision Area | Activated By |
|---|---|
| Frontend Framework | Always active |
| Backend / API Layer | Always active |
| Database | Always active |
| ORM / Data Access | Always active |
| CSS / Styling | Always active |
| State Management | Always active |
| Build Tooling | Always active |
| Authentication & Identity | Always active |
| Hosting & Environments | Always active |
| CI/CD & Delivery | Always active |
| Observability & Operations | Always active |
| File & Object Storage | File upload (FEAT-27.SPEC-001, FEAT-27.SPEC-012; ASMP-35); Import/export (FEAT-29.SPEC-004, FEAT-29.SPEC-007, FEAT-16.SPEC-004, FEAT-08.SPEC-001) |
| Email & Messaging Delivery | Notifications (email/push/SMS) (24 Notification specs, e.g. FEAT-08.SPEC-001, FEAT-29.SPEC-014; Integration specs FEAT-08.SPEC-012, FEAT-08.SPEC-013, FEAT-26.SPEC-002) |
| Payments & Billing | Payments/billing (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-30.SPEC-011; ASMP-31) |
| Search | Search (FEAT-24.SPEC-001, FEAT-24.SPEC-002) |
| Background Jobs & Scheduling | Background processing (31 scheduled/timed Automation specs, e.g. FEAT-03.SPEC-003, FEAT-08.SPEC-007, FEAT-18.SPEC-003); Import/export (FEAT-29.SPEC-007); Notifications (email/push/SMS) (FEAT-08.SPEC-007) |
| Caching & Performance | Scale hints (ASMP-21, ASMP-22, ASMP-26); Offline (ASMP-27; FEAT-12.SPEC-001, FEAT-13.SPEC-001) |
| Real-time & Collaboration | Real-time (FEAT-03.SPEC-001, FEAT-07.SPEC-002, FEAT-08.SPEC-005, FEAT-22.SPEC-002, FEAT-28.SPEC-002; ASMP-21); Collaboration/concurrency (16 of 18 entities with contention; FEAT-03.SPEC-005, XBR-01) |
| Analytics & Product Telemetry | Scale hints (ASMP-21, ASMP-22, ASMP-26; BRIEF.md Scale & Non-Functional Expectations) |
| Internationalization | Internationalization (FEAT-07.SPEC-001, FEAT-15.SPEC-002, FEAT-22.SPEC-001, FEAT-27.SPEC-003, FEAT-27.SPEC-008; ASMP-25) |
| Calendar Sync | Product-mandated — "Pro's personal calendar (Google Calendar and Apple Calendar — both matter): two-way. Busy times there block Chairtime availability, and bookings made in Chairtime appear there." (BRIEF.md, ## Ecosystem & Integrations); ASMP-33 "Calendar-sync capability (reading and writing to a pro's personal calendar)" (assumptions-constraints.md, ## Dependencies); FEAT-04.SPEC-003, FEAT-03.SPEC-006 |

Inactive (not in scope): AI & Intelligent Behavior (AI/ML behavior = No), Geo & Maps (Geo/maps = No).

## 2. Decision Area Landscapes

### Frontend Framework

Serves a mobile-first web app for pros and clients, including the Instagram in-app browser (BRIEF.md Devices & platforms), 70 Screen specs, an 8-step setup wizard (FEAT-15), a multi-step booking flow (FEAT-05) and roughly one-second slot refresh (FEAT-03.SPEC-001).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Next.js (App Router) | Framework (React meta-framework) | SSR/SSG for fast public booking page loads, server actions and route handlers, large component ecosystem for wizard and dashboard screens | Open source (MIT); hosting cost separate | Low — mainstream React skills; first-class on Vercel, deployable elsewhere via Node | Very mature; framework conventions are Vercel-shaped but portable to Node hosts | knowledge-based — not fetched this run; figures from model knowledge |
| React Router v7 (framework mode, formerly Remix) | Framework (React) | Loader/action model suits form-heavy flows and progressive enhancement; SSR | Open source (MIT) | Low–medium — React skills; adapters for several runtimes | Mature; portable across Node, edge and serverless adapters | knowledge-based — not fetched this run; figures from model knowledge |
| SvelteKit | Framework (Svelte) | Small client bundles helpful in in-app browsers; form actions; smaller ecosystem than React | Open source (MIT) | Medium — Svelte skills less common; adapters for Node, Vercel, Cloudflare | Mature; ecosystem smaller, component libraries Svelte-specific | knowledge-based — not fetched this run; figures from model knowledge |
| Nuxt | Framework (Vue meta-framework) | SSR/hybrid rendering, Vue composition API for forms and wizard flows | Open source (MIT) | Medium — Vue skills; Nitro server adapters | Mature; Vue-specific ecosystem | knowledge-based — not fetched this run; figures from model knowledge |
| Astro with island components | Framework (content-first, multi-framework islands) | Fast static public pages; app-like dashboards need islands and more client code | Open source (MIT) | Medium — application state across islands needs extra design | Mature for content sites; less conventional for app-heavy dashboards | knowledge-based — not fetched this run; figures from model knowledge |

### Backend / API Layer

Serves 59 Automation specs, 13 Integration specs with inbound webhooks (payments, SMS status, calendar), idempotent payment outcomes (FEAT-07.SPEC-004) and hold/contention logic (FEAT-03.SPEC-002, FEAT-03.SPEC-005).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Framework server layer (Next.js route handlers and server actions) | Framework-bundled server | Single deployable for UI and API; webhooks as route handlers; long-running work must be delegated to a job runner | No separate cost; runs on framework host (serverless function pricing) | Low — no separate service | Coupled to the chosen frontend framework and its host runtime limits | knowledge-based — not fetched this run; figures from model knowledge |
| Hono on Node/edge runtimes | Library/framework (TypeScript HTTP) | Lightweight typed API and webhook endpoints; runs on Node, Cloudflare, Vercel, Deno, Bun | Open source (MIT) | Low — small API surface; shares TypeScript with frontend | Newer, fast-growing; portable across runtimes | knowledge-based — not fetched this run; figures from model knowledge |
| NestJS | Framework (TypeScript, opinionated) | Modular structure, DI and guards for role enforcement across 30 features; queue/scheduler modules | Open source (MIT) | Medium — framework conventions to learn | Mature; heavier than minimal frameworks; Node-bound | knowledge-based — not fetched this run; figures from model knowledge |
| FastAPI (Python) | Framework (Python async) | Typed API and webhook handling; strong for data-heavy jobs; separate language from a TypeScript frontend | Open source (MIT) | Medium — separate service and language | Mature; pairs with SQLAlchemy, not Prisma/Drizzle | knowledge-based — not fetched this run; figures from model knowledge |
| Ruby on Rails | Framework (full-stack MVC) | Batteries-included jobs, mailers, ActiveRecord; full-stack conventions for CRUD-heavy admin | Open source (MIT) | Medium — separate stack from a JS frontend unless used full-stack | Very mature; strong convention lock-in | knowledge-based — not fetched this run; figures from model knowledge |

### Database

Serves 18 entities, 56 named relationships, high-contention Booking records (feature-dependency-map.md), multi-year history for 100–500 clients per pro (ASMP-22), and the correctness bar of never silently double-booking or losing a deposit (ASMP-26); per-pro data isolation (ASMP-23).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Postgres | Managed service (Postgres plus platform) | Full Postgres (transactions, exclusion constraints, row-level security for per-pro isolation); bundled auth, storage, realtime | Free $0 (500 MB); Pro $25/mo (8 GB, then $0.125/GB) | Low — standard Postgres drivers plus supabase-js | Postgres-standard data, exit via pg_dump; platform extras add coupling | https://supabase.com/pricing (accessed 2026-09-29) |
| Neon | Managed service (serverless Postgres) | Standard Postgres with autoscaling and branching; scale-to-zero | Free $0 (0.5 GB); Launch pay-as-you-go $0.106/CU-hour, $0.35/GB-month | Low — standard drivers, serverless HTTP driver | Postgres-standard; exit via dump/restore | https://neon.com/pricing (accessed 2026-09-29) |
| Crunchy Bridge | Managed service (Postgres on AWS/Azure/GCP) | Plain managed Postgres with PITR and SOC 2 Type 2 | From ~$10/mo; storage $0.10/GB/mo | Low — standard drivers | Postgres-standard; exit via dump/restore | https://www.crunchydata.com/pricing (accessed 2026-09-29) |
| Amazon RDS / Aurora PostgreSQL | Managed service (cloud provider) | Postgres with Multi-AZ, read replicas, PITR | Instance-hour plus storage; entry instances roughly tens of dollars per month | Medium — VPC, IAM and parameter setup | Very mature; Postgres-standard data, AWS operational coupling | knowledge-based — not fetched this run; figures from model knowledge |
| Self-hosted PostgreSQL | Self-hosted | Full control of extensions and tuning | VM/host cost plus operations time | High — backups, failover, upgrades owned by team | Open-source standard; no vendor lock-in | knowledge-based — not fetched this run; figures from model knowledge |

### ORM / Data Access

Serves 18 entities with concurrency-sensitive writes (first-committed-wins, reject-with-refresh per feature-dependency-map.md) that need explicit transactions and versioning, plus schema migrations over many features.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Prisma | ORM (TypeScript) | Typed client, declarative schema, migrations; raw SQL escape hatch for constraints and locking | Open source (Apache-2.0) | Low — Node/TypeScript only | Mature; schema DSL is proprietary but SQL is exportable | knowledge-based — not fetched this run; figures from model knowledge |
| Drizzle ORM | ORM / query builder (TypeScript) | SQL-like typed queries, migration tooling, close to SQL semantics for transactions | Open source (Apache-2.0) | Low — Node/TypeScript, edge-friendly drivers | Newer; schemas in TypeScript, low lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Kysely | Query builder (TypeScript) | Type-safe SQL builder; no schema management (migrations separate) | Open source (MIT) | Low–medium — bring migration tool | Mature; minimal abstraction, low lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| SQLAlchemy with Alembic | ORM and migration tool (Python) | Full ORM and explicit transaction control | Open source (MIT) | Medium — Python backends only | Very mature; Python-only | knowledge-based — not fetched this run; figures from model knowledge |
| supabase-js / PostgREST client | Managed data API client | Direct client-to-database API guarded by row-level security; complex multi-step transactions require SQL functions | Included in Supabase plan (https://supabase.com/pricing) | Low — generated types available | Ties data access to Supabase/PostgREST; SQL functions remain portable | https://supabase.com/pricing (accessed 2026-09-29) |

### CSS / Styling

Serves mobile-first screens readable at phone width with scalable text and contrast (ASMP-28), 70 Screen specs, and a design-system passthrough that must map to tokens.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Tailwind CSS | Utility-class framework | Design tokens map to theme config; fast responsive layouts; works with all mainstream frameworks | Open source (MIT) | Low — build plugin | Very mature; utility markup couples templates to Tailwind | knowledge-based — not fetched this run; figures from model knowledge |
| CSS Modules with CSS custom properties | Vanilla CSS approach | Tokens as custom properties; scoped styles, no runtime | Free (web standard) | Low — supported by all bundlers | Standards-based; no lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Panda CSS | Build-time CSS-in-JS | Typed token-driven styles with zero runtime | Open source (MIT) | Medium — codegen step | Newer; moderate lock-in to its API | knowledge-based — not fetched this run; figures from model knowledge |
| vanilla-extract | Build-time CSS-in-TypeScript | Typed themes and tokens compiled to static CSS | Open source (MIT) | Medium — bundler plugin | Mature; TypeScript-authored styles | knowledge-based — not fetched this run; figures from model knowledge |

Component-layer candidates (evidence: multi-step wizard, booking flow, date/time pickers, dialogs and sheets across 70 Screen specs; ASMP-28 screen-reader support):
- Radix UI primitives (headless; React) — accessible unstyled behavior skinnable to tokens.
- shadcn/ui (copy-in components; React plus Tailwind CSS).
- React Aria Components (headless; React) — accessibility-first date and time components.
- Headless UI (headless; React and Vue).
- Custom components on the styling system alone (no library).

### State Management

Serves server-driven data (slot lists refreshed about every second, bookings, money list), an 8-step wizard with resumable progress (FEAT-15.SPEC-004) and multi-step booking flow state.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| TanStack Query | Library (server-cache) | Polling/refetch, caching, optimistic updates for slot lists and dashboards; framework adapters for React, Vue, Svelte | Open source (MIT) | Low | Very mature; no lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Zustand | Library (client store, React) | Small store for wizard and booking-flow state | Open source (MIT) | Low | Mature; React-oriented | knowledge-based — not fetched this run; figures from model knowledge |
| Redux Toolkit (with RTK Query) | Library (store, React-oriented) | Predictable global state and data-fetching layer | Open source (MIT) | Medium — more boilerplate | Very mature | knowledge-based — not fetched this run; figures from model knowledge |
| SWR | Library (data fetching, React) | Stale-while-revalidate polling for live lists | Open source (MIT) | Low | Mature; React-only | knowledge-based — not fetched this run; figures from model knowledge |
| Framework built-ins (React state/context, server components, URL state) | Framework built-in | Sufficient for form state and URL-carried wizard steps; polling written by hand | Free | Low | No lock-in; more hand-written code | knowledge-based — not fetched this run; figures from model knowledge |

### Build Tooling

Serves a single web application in one repository with a TypeScript codebase across 30 features and 219 specs.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Framework-bundled pipeline (Turbopack/Next.js, SvelteKit via Vite, Nuxt via Vite) | Framework-bundled build | No separate bundler config; pairs with the chosen framework | Included with framework | Low | Mature; overriding needs a documented driver | knowledge-based — not fetched this run; figures from model knowledge |
| Vite | Bundler / dev server | Fast dev server; used for SPA or framework builds | Open source (MIT) | Low | Very mature | knowledge-based — not fetched this run; figures from model knowledge |
| pnpm | Package manager | Strict, fast installs, workspace support | Open source (MIT) | Low | Mature | knowledge-based — not fetched this run; figures from model knowledge |
| npm | Package manager | Ships with Node; universal | Free | Low | Very mature | knowledge-based — not fetched this run; figures from model knowledge |
| Turborepo | Monorepo build orchestrator | Task caching if the product splits into multiple packages | Open source (MIT); remote cache optional paid | Medium | Mature; optional Vercel-coupled remote cache | knowledge-based — not fetched this run; figures from model knowledge |

### Authentication & Identity

Serves Pro one-time-code sign-in with new-device alerts and session management (FEAT-29.SPEC-001, .002, .006, .014, .015; ASMP-30), client short-lived access links scoped to one pro (FEAT-06), and a read-only Platform Operator role (FEAT-19).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Auth | Managed service | Email/phone OTP, sessions, row-level-security integration; custom access-link flow needs own code | 50,000 MAUs free; Pro $25/mo with 100,000 MAUs then $0.00325/MAU | Low if Supabase Database used | Coupled to Supabase; open-source GoTrue server | https://supabase.com/pricing (accessed 2026-09-29) |
| Clerk | Managed service (identity SaaS) | Email/SMS OTP, passkeys, session and device management, organizations/roles | Hobby free; Pro $25/mo; 50,000 MRUs included then $0.02/MRU; SMS OTP $0.01 (US/CA) | Low — SDKs and prebuilt components | User data hosted by vendor; export possible | https://clerk.com/pricing (accessed 2026-09-29) |
| WorkOS AuthKit | Managed service | Magic auth, passkeys, MFA, SSO from one integration | First 1,000,000 MAUs free | Low–medium | Vendor-hosted user store; enterprise-oriented | https://workos.com/pricing (accessed 2026-09-29) |
| Better Auth / Auth.js | Library (self-hosted in app) | Framework-native sessions and email OTP plugins; full control of access-link and device-alert logic | Open source; delivery costs (email/SMS) separate | Medium — own session tables and security review | No vendor; team owns security upkeep | knowledge-based — not fetched this run; figures from model knowledge |
| Auth0 | Managed service | Passwordless OTP, sessions, roles, actions | Free tier plus paid plans by MAU | Low–medium | Mature; migrating users out requires export work | knowledge-based — not fetched this run; figures from model knowledge |

### Hosting & Environments

Serves a mobile web app with public booking pages, webhook receivers, scheduled jobs, one-second slot response (ASMP-21), and US-first launch with UK, Canada and Australia later (BRIEF.md Geography).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Vercel | Managed platform (serverless/edge) | Preview environments per branch, native Next.js hosting, cron jobs on all plans | Hobby $0; Pro $20/mo; Enterprise custom | Low | Mature; serverless function limits and platform coupling | https://vercel.com/pricing (accessed 2026-09-29) |
| Render | Managed platform (containers, workers, cron) | Web services, background workers, cron and managed Postgres in one place | knowledge-based — pricing page fetch returned no price content | Low | Mature; container-based, portable | knowledge-based — Render pricing page fetched 2026-09-29 returned navigation content only; figures from model knowledge |
| Fly.io | Managed platform (Machines, multi-region) | Long-running processes near users; multi-region posture for future markets | Usage-based machines and volumes | Medium — Dockerfile and fly config | Container-based, portable | knowledge-based — fetch redirected and was not completed this run |
| AWS (ECS Fargate / App Runner) | Cloud provider | Full control, VPC, IAM, multi-region | Usage-based compute and networking | High — infrastructure-as-code needed | Very mature; AWS service coupling | knowledge-based — not fetched this run; figures from model knowledge |
| Cloudflare Workers / Pages | Managed edge platform | Edge execution, close to R2, KV and cron triggers; Workers runtime differs from Node | Free tier; paid plans usage-based | Medium — runtime constraints | Mature; Workers APIs are Cloudflare-specific | knowledge-based — not fetched this run; figures from model knowledge |

### CI/CD & Delivery

Serves 30 features shipped without regression given the correctness bar (ASMP-26): automated tests on booking, payment and refund logic, and dev/staging/prod promotion.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| GitHub Actions | SaaS platform (CI/CD) | Workflows on pull requests, matrix tests, environment protections and deploy hooks | 2,000 free min (Free), 3,000 (Pro/Team); Linux 2-core $0.006/min | Low | Mature; workflow YAML is GitHub-specific | https://docs.github.com/en/billing/managing-billing-for-your-products/about-billing-for-github-actions (accessed 2026-09-29) |
| GitLab CI/CD | SaaS platform / self-hosted | Pipelines with environments and approvals | Tiered per user plus compute minutes | Medium — needs GitLab repository | Mature; GitLab-specific config | knowledge-based — not fetched this run; figures from model knowledge |
| CircleCI | SaaS platform | Fast parallel pipelines and caching | Credit-based with free tier | Low–medium | Mature; config is CircleCI-specific | knowledge-based — not fetched this run; figures from model knowledge |
| Hosting-platform native deploys (e.g. Vercel Git integration) | Platform feature | Automatic preview and production deploys per branch; tests run elsewhere | Included with hosting plan (https://vercel.com/pricing) | Low | Coupled to hosting platform | https://vercel.com/pricing (accessed 2026-09-29) |
| Buildkite | SaaS orchestration with self-hosted agents | Pipelines on own infrastructure | Per-user seats plus own compute | Medium | Mature; agents self-operated | knowledge-based — not fetched this run; figures from model knowledge |

### Observability & Operations

Serves a correctness-first product (ASMP-26) needing error reporting, tracing across webhook/job chains, and alerting on failed refunds, sync health (FEAT-04.SPEC-006) and delivery failures (FEAT-08.SPEC-009); compliance posture on client data (ASMP-23, ASMP-24).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Sentry | SaaS platform (error and performance monitoring) | Error tracking, tracing, session replay | Developer free; Team $26/mo; Business $80/mo | Low — SDKs for JS and Python | Mature; proprietary backend, open-source SDKs | https://sentry.io/pricing/ (accessed 2026-09-29) |
| Better Stack | SaaS platform (uptime, logs, incident alerts) | Uptime checks, log management, on-call | knowledge-based — pricing fetch returned HTTP 503 | Low | Newer; log export possible | knowledge-based — Better Stack pricing page returned HTTP 503 on 2026-09-29 |
| Datadog | SaaS platform (full observability) | APM, logs, metrics, alerting in one product | Per-host and per-GB usage pricing | Medium | Very mature; costs scale with usage, proprietary | knowledge-based — not fetched this run; figures from model knowledge |
| Grafana Cloud | SaaS platform (metrics, logs, traces) | OpenTelemetry-native metrics, logs and traces with free tier | Free tier plus usage-based | Medium | Mature; open standards limit lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| OpenTelemetry with Honeycomb | Instrumentation standard plus SaaS backend | Vendor-neutral tracing across request, webhook and job chains | Free tier plus event-volume pricing | Medium — instrumentation work | Standards-based instrumentation | knowledge-based — not fetched this run; figures from model knowledge |

### File & Object Storage

Serves the Pro profile photo upload with size and format limits (FEAT-27.SPEC-001, FEAT-27.SPEC-012; ASMP-35) and generated artifacts: data export (FEAT-29.SPEC-007), dispute summary download (FEAT-16.SPEC-004).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Supabase Storage | Managed service | Buckets with row-level-security policies; image transformations | 1 GB free; Pro 100 GB then $0.0213/GB | Low with Supabase stack | S3-compatible protocol; platform coupling | https://supabase.com/pricing (accessed 2026-09-29) |
| Cloudflare R2 | Managed object storage | S3-compatible with zero egress fees | Zero egress; storage per GB (pricing details not on fetched page) | Low–medium | S3 API limits lock-in | https://www.cloudflare.com/developer-platform/products/r2/ (accessed 2026-09-29) |
| Amazon S3 | Managed object storage | Industry-standard object store with pre-signed uploads | Per-GB storage plus request and egress fees | Medium — IAM and CORS | Very mature; S3 API is de facto standard | knowledge-based — not fetched this run; figures from model knowledge |
| Vercel Blob | Managed object storage | Simple uploads from serverless functions | Usage-based storage and transfer | Low — on Vercel | Coupled to Vercel | knowledge-based — not fetched this run; figures from model knowledge |
| Cloudinary | SaaS platform (media management) | Image upload, resize and format conversion | Free tier plus credit-based plans | Low | Proprietary transformation URLs | knowledge-based — not fetched this run; figures from model knowledge |

### Email & Messaging Delivery

Serves transactional SMS (opt-in, STOP handling, 8am–9pm timing, delivery status) and email fallback (FEAT-08.SPEC-012, FEAT-08.SPEC-013, FEAT-14.SPEC-004; ASMP-24, ASMP-29, ASMP-32) across 24 Notification specs; sign-in codes (FEAT-29.SPEC-014); WhatsApp later (FEAT-26.SPEC-002).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Twilio (Programmable Messaging and WhatsApp) | Managed service (SMS, WhatsApp) | SMS with delivery status, inbound STOP replies, A2P 10DLC registration; WhatsApp channel through the same provider | SMS $0.0083/segment plus carrier fees and 10DLC onboarding fees; WhatsApp $0.005/message plus Meta template fees from $0.0034 | Low — SDKs and webhooks | Very mature; number and 10DLC registrations tied to the account | https://www.twilio.com/en-us/sms/pricing/us and https://www.twilio.com/en-us/whatsapp/pricing (accessed 2026-09-29) |
| Telnyx | Managed service (SMS) | US SMS with 10DLC support, delivery reports | $0.004 per part plus carrier passthrough | Low–medium — REST API and SDKs | Mature; number registrations tied to account | https://telnyx.com/pricing/messaging (accessed 2026-09-29) |
| Resend | Managed email API | Transactional email for the fallback and code emails; Node SDK | Free 3,000/mo (100/day); Pro $20–35/mo for 50,000–100,000 | Low | Newer; SMTP limits lock-in | https://resend.com/pricing (accessed 2026-09-29) |
| Postmark | Managed email API | Transactional-first email with delivery-status tracking | Free 100/mo; from $15/mo for 10,000 | Low | Long-established | https://postmarkapp.com/pricing (accessed 2026-09-29) |
| Amazon SES with SNS | Cloud email service | Low-cost email with bounce/complaint notifications; SMS via separate AWS service | Per-1,000 email pricing | Medium — IAM and sender verification | Very mature; AWS coupling | knowledge-based — not fetched this run; figures from model knowledge |

### Payments & Billing

Serves deposit card charges with payout routing to the Pro, refunds, balance payments, tips, connected-account identity/bank verification, dispute notices, and Pro subscription billing (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-16.SPEC-003, FEAT-18.SPEC-006; ASMP-31); BRIEF.md: "Established card payment processor: takes client deposits and pays out to the pro, and owns all card data. Also used for the pro's subscription billing." Idempotency and outcome consistency (FEAT-07.SPEC-004).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Stripe (Connect plus Billing) | Managed payment platform | Connected accounts with hosted identity/bank onboarding, destination charges, refunds, dispute webhooks, subscriptions | Card 2.9% + 30¢; Connect account fee $2 per monthly active account (platform-handled pricing) and payouts 0.25% + 25¢ | Low — SDKs in major languages; idempotency keys | Very mature; card data and vault tokens tied to Stripe | https://stripe.com/connect/pricing (accessed 2026-09-29) |
| Adyen for Platforms | Managed payment platform | Marketplace onboarding, split payments, payouts and disputes | Interchange-plus with per-transaction fee; enterprise sales process | Medium–high | Very mature; enterprise-oriented | knowledge-based — not fetched this run; figures from model knowledge |
| PayPal Complete Payments (Braintree/PayPal marketplaces) | Managed payment platform | Marketplace payments with seller onboarding and payouts | Per-transaction percentage plus fixed fee | Medium | Very mature; onboarding coverage varies by market | knowledge-based — not fetched this run; figures from model knowledge |
| Square (Payments and Connect) | Managed payment platform | Card payments with OAuth-based seller accounts | Per-transaction percentage plus fixed fee | Medium — seller accounts authorize the platform | Mature; seller-account model differs from platform-held funds | knowledge-based — not fetched this run; figures from model knowledge |

### Search

Serves client list search by match and filter (FEAT-24.SPEC-001, FEAT-24.SPEC-002) over 100–500 clients per pro (ASMP-22); no full-text or faceted search is specified.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostgreSQL built-ins (ILIKE, pg_trgm, full-text search) | Database feature | Match and filter over a per-pro record set with no extra service | Included in database cost | Low — SQL queries and indexes | Postgres-standard | knowledge-based — not fetched this run; figures from model knowledge |
| Typesense | Open-source engine / managed cloud | Typo-tolerant instant search | Self-hosted free; cloud priced via calculator (not retrieved) | Medium — data sync | Open-source; self-host exit | knowledge-based — Typesense pricing URL redirected to a calculator page not fetched this run |
| Meilisearch | Open-source engine / managed cloud | Typo-tolerant search with filters | Cloud from $20/mo; self-hosting free | Medium — data sync | Open-source; self-host exit | https://www.meilisearch.com/pricing (accessed 2026-09-29) |
| Algolia | SaaS platform (hosted search) | Hosted instant search; per-pro data isolation requires secured API keys | Free 10K requests/mo, 50K records; Grow $0.50 per additional 1K requests | Low–medium | Mature; proprietary, records must be synced | https://www.algolia.com/pricing (accessed 2026-09-29) |
| OpenSearch / Elasticsearch | Self-hosted or managed engine | Full-text and faceted search; heavy for the stated need | Managed cluster hourly pricing | High | Mature; operationally heavy | knowledge-based — not fetched this run; figures from model knowledge |

### Background Jobs & Scheduling

Serves 31 scheduled or timed Automation specs: slot hold expiry (FEAT-03.SPEC-003), reminder scheduling within timing windows (FEAT-08.SPEC-007), subscription renewals (FEAT-18.SPEC-003), auto-completion sweep (FEAT-12.SPEC-004), sync reconciliation (FEAT-04.SPEC-006), data export (FEAT-29.SPEC-007), retry semantics for messages and refunds (FEAT-08.SPEC-009, FEAT-09.SPEC-006).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Inngest | Managed workflow/job service | Durable step functions, delayed events, retries, cron | Free 50k executions/mo; Pro from $99/mo (1M executions) | Low — SDK and HTTP endpoint | Newer; step model is vendor-specific | https://www.inngest.com/pricing (accessed 2026-09-29) |
| Trigger.dev | Managed job platform (open-source core) | Long-running tasks, scheduling, retries | Free $5 credit; Hobby $10/mo; Pro $50/mo; per-run $0.000025 plus compute | Low–medium | Newer; open-source core allows self-hosting | https://trigger.dev/pricing (accessed 2026-09-29) |
| Postgres-native queue and cron (pg_cron with pgmq or similar) | Database-resident scheduler/queue | Jobs in same transactional store as bookings; no extra service | Included in database cost | Medium — polling worker or edge function | Postgres-standard | knowledge-based — not fetched this run; figures from model knowledge |
| BullMQ with Redis | Library plus self/managed Redis | Delayed and repeatable jobs with retries | Open source; Redis hosting cost (Upstash: https://upstash.com/pricing/redis) | Medium — needs a long-running worker | Mature; Node-specific | https://upstash.com/pricing/redis (accessed 2026-09-29) |
| Platform cron (Vercel Cron Jobs) | Platform feature | Scheduled HTTP invocations for sweeps; no built-in retry or queue | Cron included on Hobby, Pro and Enterprise | Low | Coupled to host; limited durability | https://vercel.com/pricing (accessed 2026-09-29) |

### Caching & Performance

Serves slot lists within roughly one second (ASMP-21), growth in per-pro history (ASMP-22), and read-only offline posture for the most recently loaded schedule, client list and money list (ASMP-27).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Database indexing and query design (no extra cache layer) | Database practice | Booking data per pro is small; indexed queries can serve slot computation | Included in database cost | Low | No lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Upstash Redis | Managed service (serverless Redis) | Short-lived cache, rate limiting and hold counters | Free 256 MB; pay-as-you-go $0.20 per 100K commands; fixed $10–$1,500/mo | Low — REST and Redis clients | Redis protocol limits lock-in | https://upstash.com/pricing/redis (accessed 2026-09-29) |
| Client-side cache (TanStack Query persistence, service worker) | Client library | Keeps last-loaded schedule readable offline | Free | Medium — persistence and invalidation | No lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| CDN/edge caching of public booking page (framework cache, Vercel or Cloudflare) | Platform feature | Fast public page shell; slot data still uncached | Included in hosting plan | Low | Platform-specific cache controls | https://vercel.com/pricing (accessed 2026-09-29) |
| Self-managed Redis / Amazon ElastiCache | Managed or self-hosted | General cache and locks | Instance-hour pricing | Medium — provisioning | Redis standard | knowledge-based — not fetched this run; figures from model knowledge |

### Real-time & Collaboration

Serves slot list refresh about every second while viewing (FEAT-03.SPEC-001), banners and notices updating without refresh (FEAT-08.SPEC-005, FEAT-28.SPEC-002); concurrent writes with last-write-wins, reject-with-refresh and first-committed-wins (feature-dependency-map.md, XBR-01). No WebSocket or collaborative editing is named in the profile.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Polling with TanStack Query / SWR | Client technique | Meets the roughly one-second refresh without extra infrastructure; load scales with viewers | No service cost | Low | No lock-in | knowledge-based — not fetched this run; figures from model knowledge |
| Supabase Realtime | Managed service | Postgres change broadcast and channels | Free 200 connections / 2M messages; Pro 500 connections / 5M messages | Low with Supabase | Coupled to Supabase | https://supabase.com/pricing (accessed 2026-09-29) |
| Ably | Managed service (pub/sub) | Channels and presence at scale | Free 200 connections; Standard $29/mo plus usage ($2.50/M messages) | Low–medium | Proprietary protocol with SDKs | https://ably.com/pricing (accessed 2026-09-29) |
| Pusher Channels | Managed service (pub/sub) | Simple channel push | Free tier plus plans by connections | Low | Proprietary | knowledge-based — not fetched this run; figures from model knowledge |
| Server-Sent Events over own server | Self-managed | One-way pushes from a long-running backend | Server cost | Medium — needs a stateful runtime | Standard web protocol | knowledge-based — not fetched this run; figures from model knowledge |

### Analytics & Product Telemetry

Serves the product's success metrics at a few hundred pros in year one (ASMP-22) and the one-minute booking and one-second slot benchmarks (ASMP-21); privacy posture limits client data flows (ASMP-23).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostHog | SaaS platform (open-source core) | Event analytics, funnels, session replay, feature flags | Free 1M events/mo; usage-based beyond | Low | Self-hosting available | https://posthog.com/pricing (accessed 2026-09-29) |
| Mixpanel | SaaS platform | Funnels and retention | Free 1M events/mo; Growth up to 20M events | Low | Proprietary | https://mixpanel.com/pricing/ (accessed 2026-09-29) |
| Amplitude | SaaS platform | Product analytics and funnels | Free tier plus paid plans | Low | Proprietary | knowledge-based — not fetched this run; figures from model knowledge |
| Google Analytics 4 | SaaS platform | Web traffic and funnels; privacy-consent overhead | Free | Low | Google-hosted | knowledge-based — not fetched this run; figures from model knowledge |
| Plausible Analytics | SaaS platform / self-hosted | Privacy-friendly traffic metrics; limited product funnels | Paid subscription by pageviews; self-hosting option | Low | Open-source core | knowledge-based — not fetched this run; figures from model knowledge |

### Internationalization

Serves per-account timezone and currency, never hard-coded (ASMP-25; FEAT-27.SPEC-003, FEAT-27.SPEC-008, FEAT-07.SPEC-001, FEAT-22.SPEC-001), with US first then UK, Canada, Australia; no multi-language behavior is specified.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Native Intl APIs (Intl.NumberFormat, Intl.DateTimeFormat) | Web platform standard | Currency and timezone formatting with no dependency | Free | Low | Standard | knowledge-based — not fetched this run; figures from model knowledge |
| date-fns with date-fns-tz | Library | Timezone-aware date arithmetic | Open source (MIT) | Low | Mature | knowledge-based — not fetched this run; figures from model knowledge |
| Luxon | Library | Timezone-aware date/time with IANA zones | Open source (MIT) | Low | Mature | knowledge-based — not fetched this run; figures from model knowledge |
| next-intl | Library (React/Next.js i18n) | Message catalogs plus formatting for later language support | Open source (MIT) | Low–medium | Mature; Next.js-specific | knowledge-based — not fetched this run; figures from model knowledge |
| i18next / react-i18next | Library | Framework-agnostic translations and plugins | Open source (MIT) | Medium | Very mature | knowledge-based — not fetched this run; figures from model knowledge |

### Calendar Sync

Serves two-way sync with a Pro's Google Calendar and Apple Calendar: busy time blocks availability and bookings appear there (FEAT-04.SPEC-003, FEAT-03.SPEC-006; ASMP-33), with sync health monitoring and degraded mode.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Direct integration (Google Calendar API plus CalDAV for iCloud) | Provider APIs, own code | Full control; Google deprecated CalDAV for third parties in 2019 so two protocols to build; iCloud via CalDAV | Free API usage | High — two protocols, push/watch channels, reconciliation | No vendor; more maintenance | https://www.nylas.com/blog/best-calendar-apis/ (accessed 2026-09-29) |
| Cronofy | Managed unified calendar API | Real-time sync and free/busy across Google, Microsoft and Apple | Base API from $819/month | Low–medium | Mature; vendor holds calendar tokens | https://www.cronofy.com/pricing and https://www.cronofy.com/blog/best-calendar-apis (accessed 2026-09-29) |
| Nylas | Managed unified calendar and email API | Reads and writes Google, Outlook, Exchange and iCloud (CalDAV wrapped behind JSON API) | Free; Essentials $15/mo; Pro $49/mo; per-account overage $1.35–$2.25 (calendar-only $1.35–$1.70) | Low–medium | Mature | https://www.nylas.com/pricing/ and https://developer.nylas.com/docs/cookbook/calendar/apple-calendar-api/ (accessed 2026-09-29) |
| OneCal Unified Calendar API | Managed unified calendar API | Unified API across calendar providers | Not retrieved | Low–medium | Smaller vendor | https://www.onecal.io/unified-calendar-api (accessed 2026-09-29, listed in search results, page not fetched) |
| Truto unified API | Managed unified API platform | Unified API covering Google Calendar, Outlook and Apple | Not retrieved | Medium | Smaller vendor | https://truto.one/blog/unified-api-for-google-calendar-outlook-and-apple-2026-architecture-guide/ (accessed 2026-09-29, listed in search results, page not fetched) |

## 3. Cross-Area Compatibility Notes

- Prisma, Drizzle and Kysely (ORM / Data Access) are TypeScript/Node data-access layers; they pair with the Node/TypeScript candidates in Backend / API Layer (Next.js server layer, Hono, NestJS), not with FastAPI. SQLAlchemy pairs with FastAPI, not with Node backends. Rails uses ActiveRecord.
- Every Postgres candidate in Database (Supabase, Neon, Crunchy Bridge, RDS/Aurora, self-hosted) exposes standard Postgres drivers supported by Prisma, Drizzle, Kysely and SQLAlchemy. supabase-js/PostgREST works only with Supabase.
- Supabase Auth, Storage and Realtime share one project with Supabase Postgres; row-level-security policies apply across them. Using them with another database splits identity and data.
- Clerk, WorkOS AuthKit, Better Auth/Auth.js and Auth0 keep identity separate from the database, so per-pro isolation is enforced in application code or database policies keyed to their user IDs.
- SWR and Redux Toolkit (State Management) are React-only; TanStack Query has adapters for React, Vue and Svelte; Zustand is React-oriented. Next.js is React; Nuxt is Vue; SvelteKit is Svelte.
- shadcn/ui and Radix UI assume React; shadcn/ui also assumes Tailwind CSS. Headless UI supports React and Vue. Tailwind CSS and CSS Modules pair with every Frontend Framework candidate.
- Vercel is the native host for Next.js; Next.js, React Router v7, SvelteKit, Nuxt and Astro all have adapters for other hosts. Cloudflare Workers uses a non-Node runtime, so Node-only libraries (some ORMs, BullMQ) may need compatible drivers.
- Vercel Cron Jobs (Background Jobs & Scheduling) run only on Vercel; Inngest and Trigger.dev call into an HTTP endpoint or run tasks on their own compute, so work with any host in Hosting & Environments.
- BullMQ requires a long-running worker process and Redis (Upstash, ElastiCache or self-managed); serverless-only hosting does not run it.
- Stripe, Twilio, Telnyx, Resend, Postmark, Cronofy and Nylas provide official SDKs or REST/webhook APIs usable from Node/TypeScript and Python; Adyen, PayPal and Square also have SDKs for both.
- Twilio provides both SMS and WhatsApp under one account (FEAT-26 channel), while Telnyx, Resend and Postmark cover a single channel each. Inbound STOP replies (FEAT-14.SPEC-004) and delivery-status callbacks require a public webhook endpoint in Backend / API Layer for every messaging candidate.
- Stripe Connect webhooks (disputes, payout status, renewals) and refund retry rely on idempotent handlers (FEAT-07.SPEC-004, FEAT-09.SPEC-006); every Backend / API Layer candidate can host them, and Background Jobs & Scheduling candidates can supply retries.
- Google deprecated CalDAV for third-party apps in 2019, so direct integration for Calendar Sync needs the Google Calendar API for Google and CalDAV for Apple; Cronofy and Nylas normalize both.
- Postgres exclusion constraints and transactions (Database candidates) can enforce non-overlapping slots at the data layer independent of the Real-time & Collaboration transport chosen.
- Supabase Realtime uses Postgres change streams from Supabase Postgres only; Ably and Pusher are database-independent.

## 4. Research Log

Every row marked `knowledge-based` in Section 2 was not fetched in this run: the pass fetched a bounded set of vendor pricing pages and covered the remaining candidates from model knowledge, with each affected Sources cell marked. Two fetches failed or were incomplete (Render pricing page returned navigation only; Better Stack returned HTTP 503), and two redirects (Typesense, Fly.io) were not followed.

| Decision Area | Method | Queries & Key Sources | Access Date |
|---|---|---|---|
| Frontend Framework | knowledge-based — web fetches were spent on pricing-bearing areas; open-source frameworks have no pricing pages | Candidates from model knowledge; no URLs cited | — |
| Backend / API Layer | knowledge-based — open-source frameworks; no pricing pages fetched | Candidates from model knowledge; no URLs cited | — |
| Database | web (one knowledge-based row for RDS and self-hosted) | supabase.com/pricing; neon.com/pricing; crunchydata.com/pricing; RDS and self-hosted rows knowledge-based | 2026-09-29 |
| ORM / Data Access | knowledge-based (one web-sourced row) — open-source libraries | supabase.com/pricing for supabase-js row; others from model knowledge | 2026-09-29 |
| CSS / Styling | knowledge-based — open-source libraries | No URLs cited | — |
| State Management | knowledge-based — open-source libraries | No URLs cited | — |
| Build Tooling | knowledge-based — open-source tooling | No URLs cited | — |
| Authentication & Identity | web (two knowledge-based rows) | supabase.com/pricing; clerk.com/pricing; workos.com/pricing; Better Auth/Auth.js and Auth0 from model knowledge | 2026-09-29 |
| Hosting & Environments | web (three knowledge-based rows) | vercel.com/pricing; Render pricing page fetch returned navigation content only; Fly.io fetch redirected to docs.fly.io and was not completed; AWS and Cloudflare from model knowledge | 2026-09-29 |
| CI/CD & Delivery | web (three knowledge-based rows) | docs.github.com billing page for Actions; vercel.com/pricing; GitLab, CircleCI and Buildkite from model knowledge | 2026-09-29 |
| Observability & Operations | web (four knowledge-based rows) | sentry.io/pricing; Better Stack pricing returned HTTP 503; Datadog, Grafana Cloud, OpenTelemetry/Honeycomb from model knowledge | 2026-09-29 |
| File & Object Storage | web (three knowledge-based rows) | supabase.com/pricing; cloudflare.com R2 product page (egress only, no storage price); S3, Vercel Blob, Cloudinary from model knowledge | 2026-09-29 |
| Email & Messaging Delivery | web (one knowledge-based row) | twilio.com SMS and WhatsApp pricing; telnyx.com messaging pricing; resend.com/pricing; postmarkapp.com/pricing; Amazon SES from model knowledge | 2026-09-29 |
| Payments & Billing | web (three knowledge-based rows) | stripe.com/connect/pricing; Adyen, PayPal, Square from model knowledge | 2026-09-29 |
| Search | web (three knowledge-based rows) | algolia.com/pricing; meilisearch.com/pricing; Typesense pricing URL redirected to cloud.typesense.org calculator, not fetched; Postgres built-ins and OpenSearch from model knowledge | 2026-09-29 |
| Background Jobs & Scheduling | web (one knowledge-based row) | inngest.com/pricing; trigger.dev/pricing; upstash.com/pricing/redis; vercel.com/pricing; Postgres-native queue from model knowledge | 2026-09-29 |
| Caching & Performance | web (three knowledge-based rows) | upstash.com/pricing/redis; vercel.com/pricing; other rows from model knowledge | 2026-09-29 |
| Real-time & Collaboration | web (three knowledge-based rows) | supabase.com/pricing; ably.com/pricing; polling, Pusher, SSE from model knowledge | 2026-09-29 |
| Analytics & Product Telemetry | web (three knowledge-based rows) | posthog.com/pricing; mixpanel.com/pricing; Amplitude, GA4, Plausible from model knowledge | 2026-09-29 |
| Internationalization | knowledge-based — open-source libraries and web standards | No URLs cited | — |
| Calendar Sync | web | Search "two-way calendar sync API Google Calendar and Apple iCloud CalDAV support Cronofy Nylas"; cronofy.com/pricing; nylas.com/pricing; result URLs from that search (nylas.com/blog/best-calendar-apis, cronofy.com/blog/best-calendar-apis, developer.nylas.com Apple cookbook, onecal.io, truto.one); OneCal and Truto pricing not retrieved | 2026-09-29 |


## Feasibility Assessment


# Technical Feasibility Assessment

## 1. Feasibility Summary

| Feature | Verdict | Driving Factors |
|---------|---------|-----------------|
| FEAT-01 (Service & Pricing Management) | Straightforward | Small per-Pro CRUD with validation and a price/deposit snapshot at booking time (FEAT-01.SPEC-004, FEAT-01.SPEC-005 ## Field Validation Rules); no external service |
| FEAT-02 (Availability & Working Hours Setup) | Straightforward | Versioned rule records plus a conflict-flagging check on save (FEAT-02.SPEC-003, FEAT-02.SPEC-004 ## Processing Logic); timezone arithmetic is library-grade work |
| FEAT-03 (Real-Time Slot Availability Engine) | Hard | Roughly one-second slot computation over five data sources plus first-committed-wins hold contention with a never-double-book bar (FEAT-03.SPEC-001, FEAT-03.SPEC-002, FEAT-03.SPEC-005 ## Edge Cases; ASMP-21, ASMP-26) |
| FEAT-04 (Two-Way Calendar Sync) | Research-spike recommended | Two-way sync with Google and Apple within "a couple of minutes" (FEAT-04.SPEC-003 ## Inbound Events); Apple/iCloud change-detection latency and authorization model are not knowable from the specs or landscape |
| FEAT-05 (Public Booking Page & Booking Flow) | Standard-with-integration | Multi-step flow inside the Instagram in-app browser that embeds payment and reads a stored photo (FEAT-05.SPEC-004, FEAT-05.SPEC-006 ## Processing Logic; FEAT-27.SPEC-012 ## Degradation Behavior) |
| FEAT-06 (Client Booking Identity) | Standard-with-integration | Bearer-credential access links delivered by text/email with rate limiting (FEAT-06.SPEC-006 ## Channels, FEAT-06.SPEC-007 ## Authorization Rules) |
| FEAT-07 (Deposit Payment at Booking) | Standard-with-integration | Card authorization/capture with payout routing and exactly-once outcome (FEAT-07.SPEC-005 ## Degradation Behavior, FEAT-07.SPEC-004 ## Edge Cases) |
| FEAT-08 (Automated Booking Messaging) | Standard-with-integration | SMS and email providers with delivery-status webhooks, daytime-window scheduling and retry-then-fallback (FEAT-08.SPEC-012, FEAT-08.SPEC-013 ## Degradation Behavior; FEAT-08.SPEC-007, FEAT-08.SPEC-009) |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Standard-with-integration | Processor refunds with indefinite idempotent retry (FEAT-09.SPEC-005 ## Degradation Behavior, FEAT-09.SPEC-006 ## Edge Cases) |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Standard-with-integration | Booking commit racing Pro actions under reject-with-refresh, triggering refunds or a new deposit charge (FEAT-10.SPEC-004 ## Processing Logic; dependency map Booking **Contention:** High) |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Straightforward | State transitions on already-captured money with a 24-hour undo window (FEAT-11.SPEC-002, FEAT-11.SPEC-003, FEAT-11.SPEC-004); no processor call |
| FEAT-12 (Pro Daily Schedule Dashboard) | Straightforward | Read-heavy dashboard, an auto-completion sweep and flag aggregation (FEAT-12.SPEC-004, FEAT-12.SPEC-005 ## Trigger Definition); read-only offline posture (feature-overview.md ## Non-Functional Notes) |
| FEAT-13 (Client Record Management) | Straightforward | CRUD plus hard-delete with de-identified retention cascade (FEAT-13.SPEC-004 ## Processing Logic, FEAT-13.SPEC-006) |
| FEAT-14 (Messaging Consent Management) | Standard-with-integration | Inbound STOP replies from the text provider honored on the very next message (FEAT-14.SPEC-004 ## Trigger Definition; FEAT-14.SPEC-006 ## Edge Cases) |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Straightforward | Resumable progress record and a go-live rule evaluator (FEAT-15.SPEC-004, FEAT-15.SPEC-005, FEAT-15.SPEC-007); integrations are reached through FEAT-28/FEAT-18 |
| FEAT-16 (Booking & Payment Activity Record) | Standard-with-integration | Inbound processor dispute webhooks plus an append-only log and downloadable summary (FEAT-16.SPEC-003 ## Inbound Events, FEAT-16.SPEC-005) |
| FEAT-17 (Manual Time Blocking) | Straightforward | Block CRUD, recurring-occurrence generation and conflict detection against bookings (FEAT-17.SPEC-004, FEAT-17.SPEC-005 ## Processing Logic) |
| FEAT-18 (Pro Subscription Billing & Account Management) | Standard-with-integration | Processor-run subscription with inbound renewal outcomes and a 7-day grace pause (FEAT-18.SPEC-006 ## Degradation Behavior, FEAT-18.SPEC-003) |
| FEAT-19 (Platform Support Read-Only Access) | Straightforward | Role-scoped read-only session with field masking and access logging (FEAT-19.SPEC-004 ## Authorization Rules, FEAT-19.SPEC-002) |
| FEAT-20 (Waitlist for Cancelled Slots) | Standard-with-integration | Immediate opening notices via messaging with a 30-minute priority window layered onto slot availability (FEAT-20.SPEC-004, FEAT-20.SPEC-005, FEAT-20.SPEC-008) |
| FEAT-21 (Recurring/Standing Appointments) | Standard-with-integration | Scheduled occurrence generation, per-occurrence deposit links and release at cut-off (FEAT-21.SPEC-004, FEAT-21.SPEC-005 ## Trigger Definition) |
| FEAT-22 (In-App Balance Payment) | Standard-with-integration | Second card charge with payout routing and cancellation-race refunds (FEAT-22.SPEC-005 ## Degradation Behavior, FEAT-22.SPEC-004) |
| FEAT-23 (Tipping at Checkout) | Standard-with-integration | Tip rides on the FEAT-22 charge and refund (FEAT-23.SPEC-003; dependency map ## External Touchpoints) |
| FEAT-24 (Client List Search & Filter) | Straightforward | Partial match and filter over 100–500 records per Pro (FEAT-24.SPEC-002; ASMP-22) |
| FEAT-25 (Booking & Revenue Insights) | Straightforward | Rolling per-period aggregates maintained on events (FEAT-25.SPEC-004 ## Trigger Definition) |
| FEAT-26 (WhatsApp Reminders) | Standard-with-integration | WhatsApp send/status channel with fallback to text/email (FEAT-26.SPEC-002 ## Degradation Behavior, FEAT-26.SPEC-003) |
| FEAT-27 (Pro Profile & Booking Page Settings) | Standard-with-integration | Photo storage capability plus globally-unique link names with 12-month forwarding (FEAT-27.SPEC-012 ## Degradation Behavior, FEAT-27.SPEC-007, FEAT-27.SPEC-010) |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Standard-with-integration | Processor-hosted identity/bank verification and inbound status/money events (FEAT-28.SPEC-006 ## Degradation Behavior, ## Edge Cases) |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Standard-with-integration | One-time-code delivery over SMS/email, device/session management, data export and closure orchestration (FEAT-29.SPEC-006, FEAT-29.SPEC-007, FEAT-29.SPEC-008, FEAT-29.SPEC-014) |
| FEAT-30 (Pro Booking Management) | Standard-with-integration | Pro cancel/reschedule/goodwill/bulk commits calling processor refunds with per-booking retry (FEAT-30.SPEC-008, FEAT-30.SPEC-011 ## Edge Cases) |

## 2. Per-Feature Assessments

### FEAT-01 — Service & Pricing Management

**Verdict:** Straightforward — validated CRUD over a small per-Pro list (FEAT-01.SPEC-001..003 ## States, FEAT-01.SPEC-004 ## Field Validation Rules) with a booking-time snapshot of price/duration/deposit (FEAT-01.SPEC-005 ## Cross-Field Rules) and a synchronous impact check on archive (FEAT-01.SPEC-006 ## Processing Logic); no external service.

**Required Capabilities:**
- Deposit-rule validation (fixed or percentage, at or above the `minimum-chargeable-deposit` platform parameter) in the Pro's account currency (FEAT-01.SPEC-004; XBR-05, XBR-25)
- Immutable price/deposit snapshot copied onto each Booking at booking time so later edits never alter confirmed bookings (FEAT-01.SPEC-005; XBR-04)
- Archive impact query for upcoming bookings referencing a service (FEAT-01.SPEC-006 ## Processing Logic)
- Service data read live by the booking page and slot engine immediately after save (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Concurrency: last-write-wins between the Pro's own sessions; FEAT-01 and FEAT-02 write disjoint fields (buffer_override) so concurrent saves never overlap; a client mid-checkout keeps the price shown (feature-dependency-map.md, Service **Contention:**)
- Offline/degraded: N/A — Pro-only settings writes need a live connection per ASMP-27; no degraded mode is specified beyond the product-wide plain "needs connection" message
- Scale: a handful to a few dozen services per Pro; "no growth pattern here threatens responsiveness" (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Plain relational tables in any Database-area Postgres option (Supabase Postgres, Neon, Crunchy Bridge, Amazon RDS / Aurora PostgreSQL) with the snapshot enforced by copying fields into the Booking row inside the booking transaction; data access via Prisma, Drizzle ORM or Kysely (ORM / Data Access) for typed models, or supabase-js / PostgREST client with row-level security for per-Pro isolation. Currency formatting via Native Intl APIs (Internationalization). Form state via Framework built-ins or Zustand (State Management).

**Risks & Unknowns:** The `minimum-chargeable-deposit` value must sit at or above the chosen processor's own minimum charge in each currency (platform-parameters.md); this is a decide-before-build value, not a technical blocker.

**Spike Recommendation:** None

### FEAT-02 — Availability & Working Hours Setup

**Verdict:** Straightforward — settings screens (FEAT-02.SPEC-001, FEAT-02.SPEC-002), append-only rule versioning (FEAT-02.SPEC-003 ## Processing Logic) and a post-save conflict-flagging pass (FEAT-02.SPEC-004 ## Trigger Definition) are well-trodden; the only subtlety is timezone interpretation (FEAT-02.SPEC-005 ## Field Validation Rules).

**Required Capabilities:**
- Dated versions of the Availability Rule retained while any booking references them (FEAT-02.SPEC-003; feature-overview.md ## Entity-Lifecycle Coverage Matrix)
- Post-save check of confirmed bookings against new hours that flags, never cancels (FEAT-02.SPEC-004; XBR-11)
- Timezone-aware window interpretation in the Pro's IANA zone, including daylight-saving transitions (FEAT-02.SPEC-005; ASMP-25, XBR-25)
- Concurrency: last-write-wins between the Pro's sessions; a slot held under a previous version is re-validated at confirmation (feature-dependency-map.md, Availability Rule **Contention:**)
- Offline/degraded: N/A — setup writes require a live connection (ASMP-27); no degraded mode specified
- Scale: one active rule plus a slowly growing set of superseded versions per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Versioned rows (effective-from column) in any Database-area Postgres option; the conflict-flagging check can run in-request inside the Backend / API Layer (Framework server layer, Hono, NestJS) at this data size, or be enqueued via a Background Jobs & Scheduling option (Inngest, Trigger.dev, Postgres-native queue and cron). Timezone math via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** Daylight-saving boundaries (non-existent and repeated local hours) can produce off-by-one-hour windows if rules are stored as UTC offsets rather than IANA zone plus local time (FEAT-02.SPEC-005; ASMP-25).

**Spike Recommendation:** None

### FEAT-03 — Real-Time Slot Availability Engine

**Verdict:** Hard — the engine must compute genuinely free slots from working hours, buffers, bookings, time blocks, recurring reservations, holds and calendar busy time within roughly one second and refresh within a second of a slot being taken (FEAT-03.SPEC-001 ## Processing Logic; feature-overview.md ## Non-Functional Notes; ASMP-21), while guaranteeing exactly one winner for simultaneous holds and refusing both parties when commit order is ambiguous (FEAT-03.SPEC-005 ## Edge Cases; ASMP-26 correctness bar).

**Required Capabilities:**
- Slot computation merging six sources with duration-plus-buffer fit, minimum notice, horizon and Pro-only exceptions, labeled in the Pro's timezone (FEAT-03.SPEC-001, FEAT-03.SPEC-004 ## Cross-Field Rules; XBR-01, XBR-03, XBR-25)
- Atomic slot-hold creation at checkout and for Pro-created deposit requests (up to 24 hours or 2 hours before the appointment) (FEAT-03.SPEC-002, FEAT-03.SPEC-007; XBR-02)
- Timed hold expiration releasing slots back to public availability after `checkout-hold-timeout-minutes` (FEAT-03.SPEC-003 ## Trigger Definition)
- Live refresh of the client's slot list roughly every second while viewing (FEAT-03.SPEC-001; technical-profile.md Section 3, Real-time signal)
- Consumption of calendar busy periods with a Pro-only reduced-confidence flag (FEAT-03.SPEC-006 ## Data Exchanged)
- Concurrency: first-committed-wins across two clients, client vs. Pro deposit-request hold, and hold vs. Time Block; ambiguous commit order refuses both (FEAT-03.SPEC-005 ## Edge Cases; XBR-01; feature-dependency-map.md, Booking **Contention:** High, Time Block **Contention:**)
- Offline/degraded: calendar-sync outage falls back to Chairtime-only data, invisible to clients, with a Pro-visible confidence banner; stale or duplicate busy data never retroactively alters a returned result (FEAT-03.SPEC-006 ## Degradation Behavior, ## Edge Cases)
- Scale: a few hundred pros, 20–40 bookings a week each, with computation "equally fast as the volume of historical Bookings it must exclude" grows over years (feature-overview.md ## Non-Functional Notes; ASMP-22); concurrent viewers per Pro are the polling multiplier

**Candidate Approaches:** Data-layer enforcement of non-overlap via Postgres exclusion constraints and transactions (landscape Section 3 note) on any Database-area Postgres option, with row locking expressed through Prisma raw SQL, Drizzle ORM or Kysely (ORM / Data Access), or SQL functions called via supabase-js / PostgREST client. Hold counters or short-lived locks could alternatively sit in Upstash Redis or Self-managed Redis / Amazon ElastiCache (Caching & Performance), at the cost of a second source of truth. Refresh transport: Polling with TanStack Query / SWR (Real-time & Collaboration) meets the one-second cadence without extra infrastructure; Supabase Realtime, Ably or Pusher Channels push changes instead, reducing per-viewer query load; Server-Sent Events over own server needs a stateful runtime. Computation can stay uncached on Database indexing and query design (Caching & Performance) given small per-Pro data, while the page shell uses CDN/edge caching. Hold expiry can be lazy (holds carry an expires-at and are ignored once past) or swept via Postgres-native queue and cron, Inngest delayed events, Trigger.dev, or Platform cron (Vercel Cron Jobs) (Background Jobs & Scheduling).

**Risks & Unknowns:** One-second polling multiplies database reads by concurrent viewers; a viral Instagram post could put many clients on one Pro's page at once (ASMP-21; BRIEF.md Devices & platforms) — the per-viewer query cost at that burst is not quantified upstream. Serverless cold starts on Vercel or Cloudflare Workers (Hosting & Environments) can consume much of the one-second budget. Correctness under concurrent writes depends on the constraint being enforced in the database, not application code (FEAT-03.SPEC-005 ## Edge Cases "the underlying single-Pro-Account data store enforces this"). DST and multi-version rule boundaries inside one requested range (FEAT-03.SPEC-001 ## Edge Cases) are a correctness trap.

**Spike Recommendation:** Build a thin prototype of hold creation plus slot computation on a Postgres candidate with an exclusion constraint, then load-test concurrent hold attempts on one slot and N simultaneous one-second pollers on one Pro. The spike answers: (a) whether database-level exclusion alone yields exactly one winner with no double hold under concurrency, and (b) the viewer count at which polling breaches the one-second target on the candidate hosting — which tells the Architect whether a push transport from Real-time & Collaboration is needed at launch.

### FEAT-04 — Two-Way Calendar Sync

**Verdict:** Research-spike recommended — the specs require both Google and Apple calendars to reflect busy periods and Chairtime bookings "within a couple of minutes" in both directions (FEAT-04.SPEC-003 ## Inbound Events; feature-overview.md ## Non-Functional Notes; success-metrics Calendar Sync Reliability). Google offers push channels, but whether Apple/iCloud busy-time changes can be detected within that window — through direct CalDAV polling or through a unified provider (Nylas, Cronofy) — and what authorization flow iCloud imposes on a Pro are not knowable from the specs or the landscape (landscape Calendar Sync area: "two protocols to build").

**Required Capabilities:**
- Account-linking handshake for Google and Apple, one connection per kind (FEAT-04.SPEC-001, FEAT-04.SPEC-003 ## Data Exchanged, FEAT-04.SPEC-008)
- Busy/free pull only (never event titles) feeding FEAT-03 (FEAT-04.SPEC-003, FEAT-04.SPEC-004; feature-overview.md Data sensitivity)
- Write/move/remove of minimal booking events on create/reschedule/cancel, queued and retried, never blocking the booking (FEAT-04.SPEC-005 ## Processing Logic; FEAT-04.SPEC-003 ## Degradation Behavior)
- Health monitoring, revoked-permission detection and reconciliation on reconnect (FEAT-04.SPEC-006 ## Trigger Definition); in-app reconnection banner (FEAT-04.SPEC-007 ## Channels)
- Concurrency: a Pro disconnect wins over in-flight sync; health status last-write-wins; out-of-order write/remove events resolved by Chairtime event timestamp; duplicate busy updates idempotent (feature-dependency-map.md, Calendar Connection **Contention:**; FEAT-04.SPEC-003 ## Edge Cases)
- Offline/degraded: slow provider keeps last synced busy periods; provider down falls back to Chairtime-only availability with Pro-only confidence narrowing; failed writes queue for retry; reconciliation keeps status at Syncing until drift is resolved (FEAT-04.SPEC-003 ## Degradation Behavior)
- Scale: at most two connections per Pro, a few hundred pros in year one; near-immediate sync latency in both directions (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Direct integration (Google Calendar API plus CalDAV for iCloud) (Calendar Sync) gives full control and no per-account fee but means two protocols, Google watch channels, CalDAV polling and own reconciliation, with a Background Jobs & Scheduling option (Inngest, Trigger.dev, Postgres-native queue and cron, BullMQ with Redis) driving polling and retries. Nylas (Calendar Sync) wraps Google and iCloud CalDAV behind one JSON API with per-account pricing ($1.35–$1.70 calendar-only overage), shrinking the protocol work. Cronofy (Calendar Sync) offers real-time sync and free/busy across Google and Apple at a base of $819/month, which is heavy against a few hundred pros. OneCal Unified Calendar API and Truto unified API are listed in the landscape but with pricing not retrieved. Webhook receipt for provider pushes can sit in any Backend / API Layer candidate.

**Risks & Unknowns:** Apple/iCloud has no third-party push channel noted in the landscape, so near-immediate busy-time detection may require frequent polling per connected Pro — a cost and rate-limit exposure not quantified upstream. iCloud CalDAV access typically depends on an app-specific password rather than a consent screen, which may conflict with the one-tap "authorize" flow the Setup screen describes (FEAT-04.SPEC-001) — unconfirmed in the landscape. A stale Apple busy period is the most direct path to the double-booking the product must never allow (ASMP-33, ASMP-26). Managed providers hold calendar tokens (landscape Maturity & Lock-in), a data-sensitivity consideration for a "Sensitive" personal calendar grant (feature-overview.md Data sensitivity).

**Spike Recommendation:** Connect one Google and one iCloud test calendar through (a) direct Google Calendar API plus CalDAV and (b) Nylas, and measure: the end-to-end latency from a personal-calendar change to busy time visible in Chairtime, the authorization steps a Pro must perform for iCloud, and the polling/request volume per connection per hour. The answer that unblocks the build team: whether the "within a couple of minutes" target is achievable for Apple at an acceptable per-Pro cost, and which Calendar Sync option does it — or whether the product owner must accept a longer Apple latency with the Pro-visible confidence banner as mitigation.

### FEAT-05 — Public Booking Page & Booking Flow

**Verdict:** Standard-with-integration — the multi-step flow (FEAT-05.SPEC-001..005 ## States) is conventional, but it embeds the payment step (FEAT-05.SPEC-004 → FEAT-07.SPEC-005), creates the checkout hold and Pending Payment booking (FEAT-05.SPEC-006 ## Processing Logic) and reads the stored profile photo (FEAT-27.SPEC-012 ## Degradation Behavior), all inside the Instagram in-app browser (feature-overview.md ## Non-Functional Notes).

**Required Capabilities:**
- Fast-loading public page per booking link, including forwarded old link names and the "not accepting / not available" gate (FEAT-05.SPEC-001, FEAT-05.SPEC-008; XBR-14, XBR-27)
- Live slot list with hold placed at "Acknowledge & continue" and re-validated before payment (FEAT-05.SPEC-002, FEAT-05.SPEC-006)
- Consent capture with never-pre-checked opt-in and the exact wording stored as evidence (FEAT-05.SPEC-003, FEAT-05.SPEC-007; ASMP-24)
- Policy-version binding and integrity check between acknowledgment and payment (FEAT-05.SPEC-009; XBR-08)
- Accessible, phone-width UI inside an in-app browser (ASMP-28)
- Concurrency: a slot taken, a hold expired, or a policy edited mid-flow is refused with a refreshed list or re-acknowledgment (FEAT-05.SPEC-006, FEAT-05.SPEC-009 ## Edge Cases; feature-dependency-map.md, Cancellation Policy **Contention:**)
- Offline/degraded: booking and paying require a live connection and say so plainly; an already-loaded page stays visible (technical-profile.md Section 3, Offline signal: FEAT-05.SPEC-001, .004, .005); photo outage shows the no-photo page (FEAT-27.SPEC-012 ## Degradation Behavior)
- Scale: highest-traffic surface; slots within ~1 second and full booking in under one minute (ASMP-21; feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Server-rendered public pages via Next.js (App Router), React Router v7, SvelteKit or Nuxt (Frontend Framework), with SvelteKit's smaller bundles relevant to in-app browsers and Astro with island components suited to the static shell but less to the stateful flow. Page shell via CDN/edge caching of public booking page (Caching & Performance) with slot data fetched live. Flow state via Zustand, TanStack Query or Framework built-ins (State Management); accessible pickers via React Aria Components, Radix UI primitives or shadcn/ui (CSS / Styling component layer). Payment element per FEAT-07's Payments & Billing options; photo from FEAT-27's File & Object Storage options.

**Risks & Unknowns:** Embedded card entry, 3-D Secure challenges and wallet buttons can behave differently inside the Instagram in-app browser (BRIEF.md Devices & platforms); the landscape does not record per-processor in-app-browser behavior. Funnel analytics in the flow must respect the privacy posture (ASMP-23) — relevant when choosing among Analytics & Product Telemetry options.

**Spike Recommendation:** None

### FEAT-06 — Client Booking Identity

**Verdict:** Standard-with-integration — password-free access links are delivered through the transactional text and email capabilities (FEAT-06.SPEC-006 ## Channels, ## Delivery Rules; FEAT-08.SPEC-012, FEAT-08.SPEC-013) and governed by expiry, single-use and per-phone rate-limit rules (FEAT-06.SPEC-007 ## Authorization Rules).

**Required Capabilities:**
- Unguessable bearer tokens: on-demand single-use for 30 minutes, booking-specific valid until the appointment passes (FEAT-06.SPEC-007; XBR-18; ASMP-30)
- Per-phone-number request rate limiting at issuance (FEAT-06.SPEC-007 ## Authorization Rules)
- Phone-to-Client matching scoped to one Pro, with a hard no-cross-client boundary (FEAT-06.SPEC-008)
- Link delivery by text with consent, otherwise email, with retry on failure (FEAT-06.SPEC-006)
- Concurrency: a single-use link opened on two devices — first use wins, the second sees "request a new link" (feature-dependency-map.md, Access Link **Contention:**; FEAT-06.SPEC-002 ## Edge Cases)
- Offline/degraded: online-only identity check by design; already-loaded bookings list/detail stay visible (feature-overview.md ## Non-Functional Notes; technical-profile.md Section 3, Offline: FEAT-06.SPEC-001, .003, .004, .005); delivery failure follows FEAT-08.SPEC-009 retry/fallback
- Scale: active link volume bounded by 30-minute/per-appointment life, tracking booking volume (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Custom token issuance on any Backend / API Layer candidate with tokens hashed in a Database-area Postgres option; Supabase Auth or Better Auth / Auth.js (Authentication & Identity) can supply session plumbing after redemption, while the landscape notes the custom access-link flow "needs own code" under Supabase Auth. Rate limiting via Upstash Redis (Caching & Performance) or a Postgres counter table. Delivery via Twilio, Telnyx (text) and Resend, Postmark or Amazon SES with SNS (email) (Email & Messaging Delivery).

**Risks & Unknowns:** SMS pumping/toll fraud against the link-request endpoint can generate paid texts; the rate limit mitigates but its value is not specified. Link previews in messaging apps can "open" single-use links before the client taps them, consuming the single use (FEAT-06.SPEC-007 single-use rule) — a known pattern the redemption design must tolerate. The Access Link is classified Security-sensitive (dependency map Data Sensitivity).

**Spike Recommendation:** None

### FEAT-07 — Deposit Payment at Booking

**Verdict:** Standard-with-integration — card authorization/capture, payout routing to the Pro's connected account and fee reporting are a payment-processor contract (FEAT-07.SPEC-005 ## Capability Category, ## Data Exchanged); exactly-once outcomes via idempotency (FEAT-07.SPEC-004 ## Edge Cases) are established processor patterns.

**Required Capabilities:**
- Hosted/embedded card entry so the product never holds card data (FEAT-07.SPEC-001; SC-11; ASMP-31)
- Charge exactly the locked deposit, route to the Pro's payout account with zero platform fee, report the processor fee (FEAT-07.SPEC-003, FEAT-07.SPEC-005; XBR-05, XBR-07)
- Atomic Pending Payment → Confirmed flip with one Deposit Transaction per booking on the capture event (FEAT-07.SPEC-002 ## Processing Logic)
- Idempotent webhook handling for duplicate and out-of-order capture/decline events (FEAT-07.SPEC-005 ## Edge Cases)
- Concurrency: payment completing against an expiring hold or a Pro action — first committed wins; a capture for a booking that has left Pending Payment is resolved by FEAT-07.SPEC-004, not dropped (FEAT-07.SPEC-005 ## Edge Cases; feature-dependency-map.md, Deposit Transaction **Contention:**)
- Offline/degraded: slow processor shows "Processing payment, do not close this page"; processor down disables Pay while the hold keeps counting down; outcome unknown creates no Deposit Transaction until a result arrives (FEAT-07.SPEC-005 ## Degradation Behavior, ## Edge Cases)
- Scale: at most one deposit per booking at 20–40 bookings/week per Pro; payment step within the one-minute booking budget (feature-overview.md ## Non-Functional Notes; ASMP-21)

**Candidate Approaches:** Stripe (Connect plus Billing) (Payments & Billing) offers connected accounts, destination charges and idempotency keys in one platform that also covers FEAT-18; Adyen for Platforms offers split payments with an enterprise sales process; PayPal Complete Payments offers marketplace payouts with varying market coverage; Square (Payments and Connect) uses seller-authorized accounts, a different funds model from platform-held connected accounts. Webhook receipt on any Backend / API Layer candidate; failure tracing via Sentry or OpenTelemetry with Honeycomb (Observability & Operations).

**Risks & Unknowns:** The "zero platform fee" rule (XBR-07) interacts with connected-account pricing — Stripe Connect lists a $2 per monthly active account fee and payout fees (landscape Payments & Billing) that the platform absorbs; whether that fits the `subscription-price` is a business question. A capture arriving after hold expiry and a re-sold slot is the path to double-booking or a lost deposit (ASMP-26); FEAT-07.SPEC-004 governs it but the refund-or-honor behavior depends on FEAT-03's constraint design. In-app-browser 3-D Secure behavior is unverified (see FEAT-05).

**Spike Recommendation:** None

### FEAT-08 — Automated Booking Messaging

**Verdict:** Standard-with-integration — the feature is defined by two external capabilities, text and email, each with delivery-status reporting (FEAT-08.SPEC-012, FEAT-08.SPEC-013 ## Inbound Events, ## Degradation Behavior), driven by scheduled reminders in an 8am–9pm window (FEAT-08.SPEC-007 ## Processing Logic) and retry-once-then-email fallback (FEAT-08.SPEC-009).

**Required Capabilities:**
- Transactional SMS with delivery status (Queued/Sent/Delivered/Failed) and a timeout that converts silence into Failed (FEAT-08.SPEC-012 ## Edge Cases)
- Transactional email fallback with delivery status (FEAT-08.SPEC-013)
- Reminder scheduling at `reminder-lead-time-days`, shifted into the Pro's local daytime window; confirmations within about a minute of payment (FEAT-08.SPEC-007; XBR-16; ASMP-29)
- Consent-based channel selection read on every send (FEAT-08.SPEC-011; XBR-15)
- One-tap reply routing and booking-specific manage-link issuance (FEAT-08.SPEC-008, FEAT-08.SPEC-010); .ics add-to-calendar link in the confirmation (FEAT-08.SPEC-001)
- In-app Pro notifications and attention alerts updating without refresh (FEAT-08.SPEC-005, FEAT-08.SPEC-006; technical-profile.md Section 3, Real-time)
- Concurrency: duplicate and out-of-order status events resolved by provider event time; a reply tap after the booking changed is evaluated against current state (FEAT-08.SPEC-012 ## Edge Cases)
- Offline/degraded: every send is asynchronous; provider slow/down records Failed after timeout and hands off to retry/fallback; the in-app copy never depends on the provider (FEAT-08.SPEC-012, FEAT-08.SPEC-013 ## Degradation Behavior)
- Scale: at least a confirmation and a reminder per booking at 20–40 bookings/week per Pro; immutable Message records accumulate over years (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Text via Twilio (Programmable Messaging and WhatsApp) — which also carries FEAT-26's WhatsApp channel under one account — or Telnyx at a lower per-part price (Email & Messaging Delivery); email via Resend, Postmark (transactional-first delivery tracking) or Amazon SES with SNS. Scheduling and retry via Inngest (delayed events and retries), Trigger.dev, Postgres-native queue and cron, or BullMQ with Redis where a long-running worker exists; Platform cron (Vercel Cron Jobs) offers sweeps without built-in retry (Background Jobs & Scheduling). In-app updates via Polling with TanStack Query / SWR or Supabase Realtime / Ably / Pusher Channels (Real-time & Collaboration). Local-hour computation via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** US A2P 10DLC brand/campaign registration (landscape Twilio and Telnyx rows) gates any production texting and can take time; unregistered traffic is filtered by carriers. Carrier fees on top of per-segment price are not totaled upstream. No landscape option names a library for generating the .ics attachment/link (FEAT-08.SPEC-001) — a small landscape gap recorded in Section 5. Messages carry the Pro's studio address, which may be a home address (feature-overview.md Data sensitivity), so provider log retention matters.

**Spike Recommendation:** None

### FEAT-09 — Cancellation & No-Show Policy Engine

**Verdict:** Standard-with-integration — immutable policy versions and binary outcome rules are straightforward (FEAT-09.SPEC-002, FEAT-09.SPEC-003), but automatic full refunds through the processor with indefinite, idempotent retry (FEAT-09.SPEC-005 ## Degradation Behavior, FEAT-09.SPEC-006 ## Edge Cases) make an external payment contract part of the feature.

**Required Capabilities:**
- Immutable Cancellation Policy versions with each booking bound to the acknowledged version and a rendered cut-off time in the Pro's timezone (FEAT-09.SPEC-002; XBR-08)
- Instant outcome evaluation on every cancel/reschedule/no-show/Pro cancel (FEAT-09.SPEC-004; XBR-09)
- Refund request drawing on the Pro's payout account, retried every `refund-retry-interval-hours` until success, never dropped (FEAT-09.SPEC-005, FEAT-09.SPEC-006; XBR-10)
- Hand-off of a paid balance for full refund even when the deposit is forfeited (FEAT-09.SPEC-005 ## Edge Cases; XBR-23)
- Concurrency: one terminal outcome per deposit; automation and manual retries tied to the same idempotency reference; out-of-order refund events resolved to the true state (feature-dependency-map.md, Deposit Transaction **Contention:**; FEAT-09.SPEC-005 ## Edge Cases)
- Offline/degraded: processor unreachable leaves the deposit at Refund in Progress, shown as "in progress" to both parties, with a Pro attention flag (FEAT-09.SPEC-005 ## Degradation Behavior)
- Scale: outcome updates track cancellation/no-show volume, well within 20–40 bookings/week per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Refunds via Stripe (Connect plus Billing) with idempotency keys and refund-from-connected-account semantics, or Adyen for Platforms, PayPal Complete Payments or Square (Payments & Billing), each differing in how a refund draws on the seller's balance. Retry cadence via Inngest durable steps, Trigger.dev, Postgres-native queue and cron, or BullMQ with Redis (Background Jobs & Scheduling). Alerting on stuck refunds via Sentry, Better Stack or Grafana Cloud (Observability & Operations).

**Risks & Unknowns:** A refund drawn on a connected account with insufficient balance (because deposits were already paid out) is the most likely "cannot complete" cause (FEAT-09.SPEC-005 ## Degradation Behavior, Rejects column); retry succeeds only once the Pro's balance recovers, and processor rules for negative balances differ across Payments & Billing options. Indefinite retry needs monitoring to avoid silent long-tail refunds.

**Spike Recommendation:** None

### FEAT-10 — Client-Initiated Cancel/Reschedule

**Verdict:** Standard-with-integration — the commit coordinates deposit outcome, a new deposit charge on a late reschedule, calendar mirroring, activity logging and freed-slot hand-off (FEAT-10.SPEC-004 ## Processing Logic; FEAT-10.SPEC-003), relying on payment and calendar capabilities owned by FEAT-07/FEAT-09/FEAT-04; the contention rule is explicit and implementable (feature-dependency-map.md, Booking **Contention:**).

**Required Capabilities:**
- Eligibility and live cancellation-window countdown against the bound policy version (FEAT-10.SPEC-005)
- Reschedule through the same live slot list and hold machinery as a new booking (FEAT-10.SPEC-002; FEAT-03)
- Late reschedule: deposit kept plus a new deposit charged before confirming (FEAT-10.SPEC-003; XBR-09)
- Multi-side-effect commit: outcome, refund hand-off, calendar move/remove, activity event, freed slot to waitlist (FEAT-10.SPEC-004; XBR-13, XBR-21, XBR-28); client and Pro notices (FEAT-10.SPEC-006)
- Concurrency: High — client action races Pro cancel/reschedule/no-show/complete and automations; first committed transition wins, the other actor sees current state (feature-dependency-map.md, Booking **Contention:**)
- Offline/degraded: cancel/reschedule require a live connection; loaded booking screens stay visible (technical-profile.md Section 3, Offline: FEAT-10.SPEC-001..003); downstream refund/calendar failures do not roll back the commit (FEAT-04.SPEC-003, FEAT-09.SPEC-005 ## Degradation Behavior)
- Scale: target of 85% of cancellations/reschedules self-served — a meaningful share of weekly traffic; slot refresh within ~1 second (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** State-guarded transitions (compare-and-set on booking status/version) inside a transaction on any Database-area Postgres option, via Prisma, Drizzle ORM, Kysely or SQL functions through supabase-js / PostgREST client (ORM / Data Access). Side effects after commit either via an outbox table drained by Postgres-native queue and cron, or via events to Inngest or Trigger.dev (Background Jobs & Scheduling). New-deposit charge through FEAT-07's Payments & Billing option.

**Risks & Unknowns:** Late reschedule couples a new card charge to a slot hold — if the charge fails after the old booking is cancelled the client could lose both slots; FEAT-10.SPEC-003/SPEC-004 order the steps, but atomicity across processor and database needs care. Side-effect fan-out must be idempotent so retries do not double-notify or double-refund (XBR-10).

**Spike Recommendation:** None

### FEAT-11 — No-Show Marking & Deposit Forfeiture

**Verdict:** Straightforward — marking transitions Booking to No-Show and Deposit Transaction to Forfeited in one step, with a 24-hour undo restoring prior states (FEAT-11.SPEC-002, FEAT-11.SPEC-003 ## Processing Logic); the deposit is already captured, so no processor call is made (feature-overview.md Compliance flags).

**Required Capabilities:**
- Single-transaction dual-entity transition with outcome derived from the bound policy (FEAT-11.SPEC-002; XBR-09)
- Time-bounded undo within `no-show-undo-grace-window-hours` and marking window closing at auto-completion (FEAT-11.SPEC-004; XBR-12)
- Own-bookings-only authorization for the Pro (FEAT-11.SPEC-004 ## Authorization Rules)
- Concurrency: marking races a client cancel or a goodwill refund — reject-with-refresh, one terminal outcome per deposit (feature-dependency-map.md, Booking and Deposit Transaction **Contention:**)
- Offline/degraded: marking needs a live connection and says so; a failed write is retried and flagged, never silently queued (feature-overview.md ## Non-Functional Notes; ASMP-27; technical-profile.md Section 3, Offline: FEAT-11.SPEC-001)
- Scale: a subset of 20–40 bookings/week per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Transactional status updates with version checks on any Database-area Postgres option through Prisma, Drizzle ORM, Kysely or supabase-js / PostgREST client SQL functions (ORM / Data Access); undo-window enforcement by timestamp comparison at request time, needing no scheduler.

**Risks & Unknowns:** None identified — no external call and small volume; the only coupling is the dispute overlay (FEAT-16) that must coexist with Forfeited, which the Deposit Transaction contention rule already specifies.

**Spike Recommendation:** None

### FEAT-12 — Pro Daily Schedule Dashboard

**Verdict:** Straightforward — a read-heavy dashboard (FEAT-12.SPEC-001..003 ## States), a daily auto-completion sweep (FEAT-12.SPEC-004 ## Trigger Definition), attention-flag aggregation from other features' states (FEAT-12.SPEC-005) and masked support views (FEAT-12.SPEC-008); no external service.

**Required Capabilities:**
- Today/upcoming list with paid badge, balance due, "I'll be there" status and sync-reliability indicator (FEAT-12.SPEC-001, FEAT-12.SPEC-007)
- Auto-completion 7 days after appointment (FEAT-12.SPEC-004; XBR-12)
- De-duplicated attention items from sync health, delivery failures, refunds, disputes, setup conflicts (FEAT-12.SPEC-005)
- Past bookings browse by date over multi-year history (FEAT-12.SPEC-003)
- Live-updating notices without manual refresh (technical-profile.md Section 3, Real-time: FEAT-08.SPEC-005 citing FEAT-12)
- Concurrency: actions from the dashboard participate in the Booking reject-with-refresh rule (feature-dependency-map.md, Booking **Contention:**; FEAT-12.SPEC-006)
- Offline/degraded: most recently loaded schedule readable offline; actions require reconnecting (feature-overview.md ## Non-Functional Notes, Offline / degraded posture; ASMP-27)
- Scale: next booking's status identifiable within a few seconds; past browse equally responsive as history grows (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Offline readability via Client-side cache (TanStack Query persistence, service worker) (Caching & Performance). Live notices via Polling with TanStack Query / SWR, or Supabase Realtime, Ably or Pusher Channels (Real-time & Collaboration). Sweep via Platform cron (Vercel Cron Jobs), Postgres-native queue and cron, Inngest or Trigger.dev (Background Jobs & Scheduling). Aggregation either computed at read time with Database indexing and query design, or materialized by event handlers.

**Risks & Unknowns:** Service-worker persistence is unreliable in some in-app and private browsing contexts, so the offline-read promise (ASMP-27) may not hold uniformly on every Pro device; the Pro mostly uses a normal mobile browser, which lowers this risk.

**Spike Recommendation:** None

### FEAT-13 — Client Record Management

**Verdict:** Straightforward — contact/note CRUD (FEAT-13.SPEC-001, FEAT-13.SPEC-002), per-Pro phone uniqueness (FEAT-13.SPEC-005) and a hard-delete cascade that keeps de-identified financial and timeline records (FEAT-13.SPEC-004 ## Processing Logic, FEAT-13.SPEC-006) are established patterns.

**Required Capabilities:**
- Private-note field excluded from every Support view (FEAT-13.SPEC-005 ## Authorization Rules; XBR-24)
- Deletion eligibility blocked by upcoming bookings; irreversible delete with consent cascade and de-identification of retained records (FEAT-13.SPEC-003, FEAT-13.SPEC-004, FEAT-13.SPEC-006; XBR-19)
- Phone change invalidates access links and texting consent (feature-dependency-map.md, Client **Contention:**; XBR-15)
- Concurrency: Pro edits vs. client self-edits last-write-wins; two first bookings with the same phone merge to one record; a concurrent edit after deletion is refused with refresh (feature-dependency-map.md, Client **Contention:**)
- Offline/degraded: last loaded client list readable offline; opening, editing or deleting a record requires connection (feature-overview.md ## Non-Functional Notes, Offline / degraded posture)
- Scale: 100–500 clients per Pro; record loads instantly (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Row-level security on Supabase Postgres, or application-layer tenant scoping with Prisma, Drizzle ORM or Kysely against Neon, Crunchy Bridge or Amazon RDS / Aurora PostgreSQL (Database, ORM / Data Access); a unique index on (pro, normalized phone) handles the merge race. De-identification as a transactional update of retained rows. Offline list via Client-side cache (Caching & Performance).

**Risks & Unknowns:** De-identification must also reach copies outside the primary database — messaging-provider logs, analytics events, error-tracking payloads (Observability & Operations, Analytics & Product Telemetry options) — for the deletion commitment (feature-overview.md Compliance flags) to be complete.

**Spike Recommendation:** None

### FEAT-14 — Messaging Consent Management

**Verdict:** Standard-with-integration — consent state is simple, but revocation arrives as inbound "STOP" replies through the text provider (FEAT-14.SPEC-004 ## Trigger Definition; FEAT-08.SPEC-012) and must be honored on the very next message (feature-overview.md ## Non-Functional Notes; XBR-15).

**Required Capabilities:**
- Consent record with state, timestamp, channel and exact wording shown (FEAT-14.SPEC-003; ASMP-24)
- Immediate revocation on opt-out link or inbound STOP; opt-out confirmation message (FEAT-14.SPEC-004, FEAT-14.SPEC-009)
- Re-grant from client preferences; phone change invalidates consent (FEAT-14.SPEC-005, FEAT-14.SPEC-008)
- Textability determination consumed by every send (FEAT-14.SPEC-007)
- Concurrency: STOP vs. in-app re-grant resolved by most recent explicit action timestamp, defaulting to no-text when uncertain (FEAT-14.SPEC-006; feature-dependency-map.md, Messaging Consent **Contention:**)
- Offline/degraded: preference screen needs a live connection; if consent state is uncertain the no-text state applies (feature-dependency-map.md, Messaging Consent **Contention:**; technical-profile.md Section 3, Offline: FEAT-14.SPEC-001)
- Scale: one consent record per client–Pro pair plus transition history (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Inbound STOP via Twilio (Programmable Messaging and WhatsApp) or Telnyx webhooks (Email & Messaging Delivery) received by any Backend / API Layer candidate (landscape Section 3: inbound STOP needs a public webhook). Twilio and Telnyx also apply carrier-level opt-out handling on their side, which the product's own consent record must mirror. Append-only consent history in any Database-area Postgres option.

**Risks & Unknowns:** Provider-level STOP handling and product-level consent can diverge (e.g., the provider blocks a number the product believes re-granted) — reconciliation between the two is not specified. STOP replies arrive on the sending number, so a shared number across all pros must map a STOP to the right client–Pro pair; number strategy (shared vs. per-Pro) is not decided upstream.

**Spike Recommendation:** None

### FEAT-15 — Pro Onboarding & Setup Wizard

**Verdict:** Straightforward — a wizard shell over other features' screens (FEAT-15.SPEC-001), a resumable progress record (FEAT-15.SPEC-004 ## Processing Logic) and a go-live rule re-evaluated on step completion or upstream status change (FEAT-15.SPEC-005, FEAT-15.SPEC-007; XBR-26); payout and subscription integrations are owned by FEAT-28 and FEAT-18.

**Required Capabilities:**
- Progress record with exact resume point, optional calendar step (FEAT-15.SPEC-004, FEAT-15.SPEC-006)
- Go-live evaluation triggered by asynchronous upstream events (payout verification, subscription) (FEAT-15.SPEC-005)
- First Cancellation Policy version with default window (FEAT-15.SPEC-002; `cancellation-window-default-hours`)
- Welcome confirmation when the link goes live (FEAT-15.SPEC-008)
- Concurrency: step completion and upstream status events can arrive together; go-live re-evaluation must be idempotent (FEAT-15.SPEC-005 ## Edge Cases; feature-dependency-map.md, Pro Account **Contention:**)
- Offline/degraded: failed step preserves entered values with retry; go-live preview screen keeps loaded content (feature-overview.md ## Non-Functional Notes; technical-profile.md Section 3, Offline: FEAT-15.SPEC-003)
- Scale: one progress record per new Pro, a few hundred in year one (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Wizard state carried in URL and server progress record via Framework built-ins, or Zustand/TanStack Query (State Management), on any Frontend Framework candidate. Go-live evaluation as a function called from payout/subscription webhook handlers on any Backend / API Layer candidate, or as an event consumer in Inngest or Trigger.dev (Background Jobs & Scheduling).

**Risks & Unknowns:** Payout verification can stay pending for days (FEAT-15.SPEC-003 pending hand-over), so the go-live trigger depends on FEAT-28's inbound events being reliable; a missed event would silently keep a ready Pro offline.

**Spike Recommendation:** None

### FEAT-16 — Booking & Payment Activity Record

**Verdict:** Standard-with-integration — the append-only log (FEAT-16.SPEC-002, FEAT-16.SPEC-005) is conventional, but inbound card-issuer dispute notices from the processor (FEAT-16.SPEC-003 ## Inbound Events, ## Degradation Behavior) and a downloadable dispute summary (FEAT-16.SPEC-004) add an external contract and file generation.

**Required Capabilities:**
- Immutable, append-only Activity Events written by many features (FEAT-16.SPEC-002; XBR-21)
- Dispute webhook: flag booking, set Disputed overlay without erasing outcome, record event (FEAT-16.SPEC-003; XBR-22)
- Plain-language dispute summary assembled from loaded timeline and handed over as a file (FEAT-16.SPEC-004)
- Concurrency: none on entries (append-only); duplicate and out-of-order dispute events idempotent, "concluded" held until the original notice arrives (feature-dependency-map.md, Activity Event **Contention:** None; FEAT-16.SPEC-003 ## Edge Cases)
- Offline/degraded: processor dispute channel down means notices arrive late with original timestamps; timeline shows only known disputes with no error state (FEAT-16.SPEC-003 ## Degradation Behavior); loaded timeline readable offline (technical-profile.md Section 3, Offline: FEAT-16.SPEC-001)
- Scale: a handful of events per booking over multi-year history; timeline loads instantly (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Append-only table with update/delete privileges revoked, on any Database-area Postgres option; dispute webhooks from Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments or Square (Payments & Billing). The summary can be generated on the fly and streamed, or stored in Supabase Storage, Cloudflare R2, Amazon S3 or Vercel Blob (File & Object Storage).

**Risks & Unknowns:** The file format of the dispute summary is not specified and the landscape names no document-generation library — recorded as a gap in Section 5. Retention after client deletion keeps only de-identified events (XBR-19), so event payloads must avoid embedding raw personal data that cannot be scrubbed.

**Spike Recommendation:** None

### FEAT-17 — Manual Time Blocking

**Verdict:** Straightforward — block CRUD (FEAT-17.SPEC-001, FEAT-17.SPEC-002), conflict detection against confirmed bookings (FEAT-17.SPEC-004 ## Processing Logic), recurring occurrence generation (FEAT-17.SPEC-005) and automatic expiry (FEAT-17.SPEC-007) are standard scheduling patterns; the one-second visibility bar rides on FEAT-03.

**Required Capabilities:**
- One-off and recurring blocks in the Pro's timezone with private labels (FEAT-17.SPEC-001, FEAT-17.SPEC-008; ASMP-25)
- Conflict review and explicit resolution (cancel, reschedule, keep as exception), never silent (FEAT-17.SPEC-003, FEAT-17.SPEC-006; XBR-11)
- Rolling generation of future occurrences with the same conflict detection (FEAT-17.SPEC-005)
- Concurrency: block vs. client checkout — first committed wins; block edits between Pro sessions last-write-wins (feature-dependency-map.md, Time Block **Contention:**; FEAT-03.SPEC-005 ## Edge Cases)
- Offline/degraded: creating/removing a block needs a live connection; loaded block list readable (feature-overview.md Compliance flags; technical-profile.md Section 3, Offline: FEAT-17.SPEC-001, .002)
- Scale: small per-Pro set; block appears/disappears from slot list within ~1 second (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Blocks stored as ranges in any Database-area Postgres option and included in the same exclusion-constraint/transaction scheme as FEAT-03. Recurring generation either materialized on a schedule (Postgres-native queue and cron, Inngest, Trigger.dev, Platform cron (Vercel Cron Jobs) — Background Jobs & Scheduling) or expanded at read time from the pattern; recurrence math via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** Materialized recurring occurrences create conflicts at generation time that the Pro was not present for (FEAT-17.SPEC-005); these route to the dashboard, which is specified, but the generation horizon choice affects how far ahead conflicts surface.

**Spike Recommendation:** None

### FEAT-18 — Pro Subscription Billing & Account Management

**Verdict:** Standard-with-integration — subscribe/update/cancel calls and inbound renewal outcomes through the payment processor (FEAT-18.SPEC-006 ## Data Exchanged, ## Degradation Behavior), with a 7-day grace clock and pause trigger (FEAT-18.SPEC-003, FEAT-18.SPEC-004).

**Required Capabilities:**
- Single-tier recurring subscription with card on file at the processor (FEAT-18.SPEC-001, FEAT-18.SPEC-005; SC-11)
- Renewal outcome processing and grace-period timer leading to a system-imposed pause (FEAT-18.SPEC-003, FEAT-18.SPEC-004; XBR-14)
- Cancellation at period end, including cancellation invoked by account closure (FEAT-18.SPEC-006; XBR-20)
- Billing notifications incl. 30-day price-change notice (FEAT-18.SPEC-007)
- Concurrency: payment-method update during a renewal retry — the processor's recorded outcome is authoritative; stale screen refused with refresh (feature-dependency-map.md, Subscription **Contention:**; FEAT-18.SPEC-006 ## Edge Cases)
- Offline/degraded: processor down disables subscribe/update/cancel with plain messages; a down processor during renewal is inconclusive, not a failure (FEAT-18.SPEC-006 ## Degradation Behavior); last-known plan status readable offline (feature-overview.md ## Non-Functional Notes)
- Scale: one subscription per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Stripe (Connect plus Billing) (Payments & Billing) runs the renewal schedule itself and reports outcomes by webhook, sharing a platform with deposits; Adyen for Platforms, PayPal Complete Payments or Square would pair with their own recurring-billing products (not detailed in the landscape). Grace-period timer via Inngest delayed events, Trigger.dev, Postgres-native queue and cron or Platform cron (Vercel Cron Jobs) (Background Jobs & Scheduling).

**Risks & Unknowns:** If the processor runs renewals, the product's grace clock must key off processor events rather than its own schedule, or two clocks disagree (FEAT-18.SPEC-003 vs. FEAT-18.SPEC-006). The landscape does not detail recurring-billing capabilities for non-Stripe options.

**Spike Recommendation:** None

### FEAT-19 — Platform Support Read-Only Access

**Verdict:** Straightforward — a role with structurally no write path, one-account-at-a-time scoping and field masking (FEAT-19.SPEC-004 ## Authorization Rules), plus view logging to the Pro-visible access log (FEAT-19.SPEC-002, FEAT-19.SPEC-003); no external service.

**Required Capabilities:**
- Read-only Platform Operator role; opening a new lookup ends the prior session (FEAT-19.SPEC-001, FEAT-19.SPEC-004)
- Exclusion of private notes, bank/identity details and sign-in codes from every support surface (XBR-24; ASMP-30)
- Support-view events logged with reason/ticket reference (FEAT-19.SPEC-002; XBR-21)
- Concurrency: N/A — support is view-only; its only writes are append-only log events (feature-dependency-map.md, Activity Event **Contention:** None)
- Offline/degraded: N/A — no offline behavior is specified; the view "loads instantly" on a live connection (feature-overview.md ## Non-Functional Notes)
- Scale: bounded by help-request count (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Role and claims from Supabase Auth, Clerk (organizations/roles), WorkOS AuthKit, Auth0 or Better Auth / Auth.js (Authentication & Identity); read-only enforcement either via row-level-security policies on Supabase Postgres or a dedicated read-only database role/connection for support queries on any Postgres option; masking in the Backend / API Layer.

**Risks & Unknowns:** "Structurally read-only" (FEAT-19.SPEC-004) is only as strong as the enforcement layer; application-level checks alone leave a regression path across 30 features' write endpoints.

**Spike Recommendation:** None

### FEAT-20 — Waitlist for Cancelled Slots

**Verdict:** Standard-with-integration — matching on freed-slot signals and a 30-minute claim window (FEAT-20.SPEC-004, FEAT-20.SPEC-005 ## Trigger Definition) depend on immediate delivery of opening notices through the messaging capability (FEAT-20.SPEC-008 ## Delivery Rules; FEAT-08.SPEC-012/013).

**Required Capabilities:**
- Event-driven matching on cancellation, Requested → Notified transitions (FEAT-20.SPEC-005; XBR-28)
- 30-minute priority window layered onto FEAT-03 availability, then return to general availability (FEAT-20.SPEC-004; XBR-02)
- Claim conversion through the ordinary booking flow; expiry of unclaimed and elapsed entries (FEAT-20.SPEC-006, FEAT-20.SPEC-007)
- Opening notice sent immediately (daytime rule read as not applying); expiry notice within the daytime window (FEAT-20.SPEC-008, FEAT-20.SPEC-009; feature-overview.md Compliance flags)
- Concurrency: several notified clients race for one slot — first to complete booking wins; leave wins over pending notification (feature-dependency-map.md, Waitlist Entry **Contention:**; XBR-01)
- Offline/degraded: messaging failures follow FEAT-08.SPEC-009 retry/fallback; My Waitlists screen readable when loaded (technical-profile.md Section 3, Offline: FEAT-20.SPEC-002)
- Scale: at most 3 active entries per client per Pro, low hundreds per Pro (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Freed-slot events delivered to a handler via Inngest or Trigger.dev events, or an outbox drained by Postgres-native queue and cron (Background Jobs & Scheduling); claim-window expiry via delayed jobs or lazy expiry checks. Priority window modeled as a waitlist-scoped hold in the same data-layer scheme as FEAT-03. Notices via FEAT-08's Email & Messaging Delivery options.

**Risks & Unknowns:** The priority window must be enforced by the slot engine, otherwise a public client can book a slot during the window — this extends FEAT-03's contention surface. Retry-then-email fallback delay eats into the 30-minute window (FEAT-08.SPEC-009).

**Spike Recommendation:** None

### FEAT-21 — Recurring/Standing Appointments

**Verdict:** Standard-with-integration — occurrence generation within the horizon with slot validation (FEAT-21.SPEC-004 ## Processing Logic), per-occurrence deposit links and release at cut-off (FEAT-21.SPEC-005 ## Trigger Definition) and three notification types (FEAT-21.SPEC-007..009) depend on payment and messaging capabilities and scheduled jobs.

**Required Capabilities:**
- Series with 1–12 week interval; occurrences generated up to the booking horizon (FEAT-21.SPEC-001, FEAT-21.SPEC-003)
- Generation through the same slot validation; conflicts produce advance notice with re-pick prompt (FEAT-21.SPEC-004, FEAT-21.SPEC-008)
- Deposit request `recurring-occurrence-deposit-lead-days` before each occurrence; release if unpaid at the cancellation cut-off (FEAT-21.SPEC-005; XBR-02, XBR-05)
- Whole-series vs. single-occurrence cancellation (FEAT-21.SPEC-006); Pro-side management (FEAT-21.SPEC-010)
- Concurrency: client and Pro change series/occurrence — reject-with-refresh, first committed wins (feature-dependency-map.md, Recurring Series **Contention:**)
- Offline/degraded: generation retried `occurrence-generation-retry-count` times before flagging the gap to the Pro (platform-parameters.md); messaging degradation per FEAT-08.SPEC-009
- Scale: working set bounded by the horizon (default 8 weeks); small relative to ordinary bookings (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Rolling generation via Postgres-native queue and cron, Inngest cron plus steps, Trigger.dev schedules or Platform cron (Vercel Cron Jobs) (Background Jobs & Scheduling). Deposit links as processor-hosted payment links or the FEAT-07 payment page with Stripe (Connect plus Billing) or other Payments & Billing options — each occurrence paid fresh since card data is never stored (XBR-05). Recurrence math via date-fns with date-fns-tz or Luxon (Internationalization).

**Risks & Unknowns:** Occurrences reserving slots at generation (FEAT-03.SPEC-001 ## Edge Cases) means a failed or late generation run opens those slots to the public; generation reliability therefore touches the double-booking bar. Interval arithmetic across DST changes can shift local times by an hour.

**Spike Recommendation:** None

### FEAT-22 — In-App Balance Payment

**Verdict:** Standard-with-integration — a second card charge with payout routing and full refunds on cancellation through the processor (FEAT-22.SPEC-005 ## Degradation Behavior, ## Edge Cases), plus a payment/cancellation race rule (FEAT-22.SPEC-004).

**Required Capabilities:**
- Balance computed once (price − deposit) and charged exactly (FEAT-22.SPEC-003; XBR-23)
- Capture → Balance Payment record → booking marked fully paid, visible on the dashboard without refresh (FEAT-22.SPEC-002; technical-profile.md Section 3, Real-time)
- Full refund of a paid balance on any cancellation, with idempotent indefinite retry (FEAT-22.SPEC-005)
- Concurrency: client paying while the Pro cancels — cancellation first blocks payment; payment first is refunded in full (feature-dependency-map.md, Balance Payment **Contention:**; FEAT-22.SPEC-004)
- Offline/degraded: processor down disables Pay with an in-person fallback; unknown outcome creates no record until resolved; refund "in progress" states (FEAT-22.SPEC-005 ## Degradation Behavior)
- Scale: at most one balance payment per booking, a fraction of booking volume (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Same Payments & Billing options as FEAT-07 — Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments, Square — reusing FEAT-07's webhook and idempotency machinery; refund retry via FEAT-09's Background Jobs & Scheduling option.

**Risks & Unknowns:** Balance refunds drawn on a connected account after payout share FEAT-09's insufficient-balance exposure. Processor minimum charge applies to small balances (`minimum-chargeable-deposit` reused, platform-parameters.md).

**Spike Recommendation:** None

### FEAT-23 — Tipping at Checkout

**Verdict:** Standard-with-integration — the tip is charged and refunded as part of FEAT-22's balance payment through FEAT-22.SPEC-005 (feature-dependency-map.md ## External Touchpoints; FEAT-23.SPEC-003), so it inherits an external payment contract while adding only validation and a UI step (FEAT-23.SPEC-001, FEAT-23.SPEC-002).

**Required Capabilities:**
- Optional, never-defaulted, non-negative tip step inside the balance flow (FEAT-23.SPEC-001, FEAT-23.SPEC-002)
- Tip passed wholly to the Pro with zero platform cut, refunded with the balance on cancellation (FEAT-23.SPEC-003; XBR-07, XBR-23)
- Concurrency: inherits FEAT-22's payment-vs-cancellation rule (feature-dependency-map.md, Balance Payment **Contention:**)
- Offline/degraded: inherits FEAT-22.SPEC-005 ## Degradation Behavior; a tip must never be silently dropped (feature-overview.md ## Non-Functional Notes; ASMP-26)
- Scale: N/A — one optional field on Balance Payment with no independent growth (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Tip added to the balance charge amount as one charge, or as a separately itemized line, on any Payments & Billing option (Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments, Square); the choice affects fee reporting on the money list (FEAT-28.SPEC-005).

**Risks & Unknowns:** Product description mentions tipping "at deposit or at balance payment" but FEAT-23.SPEC-001 places it in the balance flow only; if tipping at deposit is later wanted it touches FEAT-07's charge amount and lock rules.

**Spike Recommendation:** None

### FEAT-24 — Client List Search & Filter

**Verdict:** Straightforward — partial name/phone matching plus recency and upcoming-booking filters over 100–500 clients per Pro (FEAT-24.SPEC-001 ## States, FEAT-24.SPEC-002; ASMP-22); the profile classifies Search as "Simple filter" (technical-profile.md Section 3).

**Required Capabilities:**
- Instant narrowing as the Pro types, partial phone and name match (FEAT-24.SPEC-001, FEAT-24.SPEC-002)
- Recency (`client-recency-filter-window-days`) and upcoming-booking derivation from booking history (FEAT-24.SPEC-002)
- Private note never surfaced in results; Support sees masked results (feature-overview.md Data sensitivity; XBR-24)
- Concurrency: N/A — read-only feature; underlying record edits follow FEAT-13's rules
- Offline/degraded: loaded client list readable and filterable offline (technical-profile.md Section 3, Offline: FEAT-24.SPEC-001; ASMP-27)
- Scale: 100–500 clients per Pro with multi-year booking history; instant results (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** PostgreSQL built-ins (ILIKE, pg_trgm, full-text search) (Search) need no extra service at this volume; alternatively the whole per-Pro list can be loaded and filtered client-side with TanStack Query cache (State Management), which also serves offline filtering. Typesense, Meilisearch or Algolia (Search) add typo tolerance at the cost of syncing personal data to another service (Algolia needs secured API keys for per-Pro isolation).

**Risks & Unknowns:** None identified — volume is small and the landscape offers database-native options; an external search service would add a personal-data processor (ASMP-23).

**Spike Recommendation:** None

### FEAT-25 — Booking & Revenue Insights

**Verdict:** Straightforward — period summaries from rolling aggregates maintained as bookings and outcomes occur (FEAT-25.SPEC-002, FEAT-25.SPEC-004 ## Trigger Definition) with derivation rules (FEAT-25.SPEC-003); no external service.

**Required Capabilities:**
- Rolling per-period aggregates updated on booking, deposit outcome, no-show and payout events, retried `insights-aggregate-retry-count` times (FEAT-25.SPEC-004)
- Period summary with "saved" figure and most-booked services, "not enough data" threshold (FEAT-25.SPEC-001, FEAT-25.SPEC-003)
- Last successful result retained when a fresh computation fails (FEAT-25.SPEC-002)
- Concurrency: concurrent event updates to the same aggregate row must not lose increments (technical-profile.md Section 3, Collaboration: FEAT-25.SPEC-004)
- Offline/degraded: loaded summary stays visible; aggregate failure retains the last good result (FEAT-25.SPEC-002; technical-profile.md Section 3, Offline: FEAT-25.SPEC-001)
- Scale: aggregates over a multi-year history of 20–40 bookings/week without degrading (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Aggregates computed at read time with Database indexing and query design (Caching & Performance) — viable at this per-Pro size — or maintained incrementally by event handlers in Inngest, Trigger.dev or Postgres-native queue and cron (Background Jobs & Scheduling), or as Postgres materialized views refreshed on schedule. Product-level analytics (PostHog, Mixpanel — Analytics & Product Telemetry) serve the founder's metrics, not the Pro-facing view.

**Risks & Unknowns:** Incremental aggregates drift from source records if an event is missed; a periodic recompute pass would bound drift but is not specified.

**Spike Recommendation:** None

### FEAT-26 — WhatsApp Reminders

**Verdict:** Standard-with-integration — WhatsApp send with delivery status (FEAT-26.SPEC-002 ## Degradation Behavior) and fallback to text or email per consent (FEAT-26.SPEC-003), governed by channel-aware consent (FEAT-26.SPEC-004).

**Required Capabilities:**
- WhatsApp send and inbound status for confirmation, reminder and change-notice content (FEAT-26.SPEC-002)
- Channel preference and explicit channel-scoped consent separate from texting consent (FEAT-26.SPEC-001, FEAT-26.SPEC-004; ASMP-24)
- Automatic fallback on failure or unavailability, with a delivery flag (FEAT-26.SPEC-003)
- Concurrency: duplicate/out-of-order status events resolved by provider event time (FEAT-26.SPEC-002 ## Edge Cases); preference changes follow Messaging Consent resolution (feature-dependency-map.md, Messaging Consent **Contention:**)
- Offline/degraded: every send asynchronous; timeout converts silence to Failed and triggers fallback; preference screen never blocked by provider outage (FEAT-26.SPEC-002 ## Degradation Behavior)
- Scale: N/A at MVP (Later phase); then redistributes FEAT-08 volume across channels (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Twilio (Programmable Messaging and WhatsApp) (Email & Messaging Delivery) is the only landscape option carrying WhatsApp, sharing the account and webhook handling with SMS; Telnyx, Resend and Postmark cover a single channel each (landscape Section 3). Fallback orchestration via the same Background Jobs & Scheduling option as FEAT-08.

**Risks & Unknowns:** Business-initiated WhatsApp messages require pre-approved templates and Meta per-template fees (landscape Twilio row: "Meta template fees from $0.0034"); template approval delays are outside the product's control. The landscape offers a single WhatsApp-capable option, so provider choice for this channel is effectively constrained.

**Spike Recommendation:** None

### FEAT-27 — Pro Profile & Booking Page Settings

**Verdict:** Standard-with-integration — the photo storage capability is an Integration spec (FEAT-27.SPEC-012 ## Capability Category: File storage; ## Degradation Behavior); the rest is settings CRUD plus globally unique link names with 12-month forwarding (FEAT-27.SPEC-007, FEAT-27.SPEC-010) and pause precedence (FEAT-27.SPEC-009, FEAT-27.SPEC-011).

**Required Capabilities:**
- Photo upload within `profile-photo-max-file-size-mb` and format limits, replace and serve; page works without a photo (FEAT-27.SPEC-012; ASMP-35)
- Cross-Pro unique booking-link names with reject-with-refresh and 12-month reservation/forwarding of old names (FEAT-27.SPEC-007, FEAT-27.SPEC-010; XBR-27)
- Timezone/currency settings with currency locked at first deposit (FEAT-27.SPEC-003, FEAT-27.SPEC-008; XBR-25)
- Pro pause with end date and automatic resume; system-imposed pause precedence (FEAT-27.SPEC-004, FEAT-27.SPEC-009, FEAT-27.SPEC-011; XBR-14)
- Notification preferences; help request with acknowledgment (FEAT-27.SPEC-005, FEAT-27.SPEC-006, FEAT-27.SPEC-013)
- Concurrency: link name claimed by another pro first or currency locked mid-edit — reject-with-refresh; other fields last-write-wins between the Pro's devices (feature-dependency-map.md, Pro Account **Contention:**)
- Offline/degraded: photo service slow/down leaves other fields savable; booking page shows no photo without error (FEAT-27.SPEC-012 ## Degradation Behavior)
- Scale: one Pro Account per pro; changes visible on the public page immediately (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Photo storage via Supabase Storage (image transformations, RLS policies), Cloudflare R2 (zero egress), Amazon S3 (pre-signed uploads), Vercel Blob, or Cloudinary (resize and format conversion) (File & Object Storage). Unique names via a unique index plus a reservation table in any Database-area Postgres option. Scheduled forwarding expiry and auto-resume via Platform cron (Vercel Cron Jobs), Postgres-native queue and cron, Inngest or Trigger.dev, or lazy evaluation at read time (Background Jobs & Scheduling). Immediate public-page reflection interacts with CDN/edge caching of public booking page (Caching & Performance) — cache invalidation on save.

**Risks & Unknowns:** Phone photos often carry EXIF location metadata; stripping it matters because the studio address may be a home address (feature-overview.md Data sensitivity) — not specified upstream. Edge caching of the public page conflicts with "shows the change immediately" unless invalidation is wired.

**Spike Recommendation:** None

### FEAT-28 — Payout Account Connection & Payout Visibility

**Verdict:** Standard-with-integration — hand-off into the processor's own identity and bank verification, action-required resolution, and inbound status/payout/fee reporting (FEAT-28.SPEC-006 ## Data Exchanged, ## Degradation Behavior, ## Edge Cases) are the processor's hosted-onboarding pattern.

**Required Capabilities:**
- Processor-hosted onboarding launch and return; no bank or identity data held (FEAT-28.SPEC-001, FEAT-28.SPEC-006; SC-11)
- Status processing driving the go-live gate and notifications (FEAT-28.SPEC-003, FEAT-28.SPEC-007; XBR-06)
- One payout account per Pro, country/currency match (FEAT-28.SPEC-004; XBR-25)
- Money list of deposits, refunds, processor fees and bank payouts with net per period (FEAT-28.SPEC-002, FEAT-28.SPEC-005; XBR-07)
- Banner clearing without manual refresh (technical-profile.md Section 3, Real-time: FEAT-28.SPEC-002)
- Concurrency: processor-reported status authoritative, last-write-wins by processor event time; stale/out-of-order reports discarded (feature-dependency-map.md, Payout Account **Contention:**; FEAT-28.SPEC-006 ## Edge Cases)
- Offline/degraded: dashboard shows last-known data with Retry; resolution flow shows capability-down message leaving Action Required unchanged (FEAT-28.SPEC-006 ## Degradation Behavior); money list readable offline (ASMP-27)
- Scale: any item in the last 90 days findable within a few seconds over multi-year history (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Stripe (Connect plus Billing) hosted identity/bank onboarding and account/payout webhooks; Adyen for Platforms marketplace onboarding; PayPal Complete Payments seller onboarding with market-dependent coverage; Square (Payments and Connect) OAuth seller accounts (Payments & Billing). Money list either mirrored locally from webhooks into any Database-area Postgres option or fetched live from the processor's reporting API; live banner via Polling with TanStack Query / SWR or a push option (Real-time & Collaboration).

**Risks & Unknowns:** Expansion to UK, Canada and Australia (BRIEF.md Geography) depends on the processor's connected-account coverage in each country — varying across options (landscape PayPal row). Mirroring payout and fee data locally risks drift from the processor's ledger; fetching live couples money-list responsiveness to processor latency.

**Spike Recommendation:** None

### FEAT-29 — Pro Sign-In & Account Lifecycle

**Verdict:** Standard-with-integration — one-time codes delivered by SMS and email (FEAT-29.SPEC-014 ## Channels; FEAT-08.SPEC-012/013), device/session management with new-device alerts (FEAT-29.SPEC-006, FEAT-29.SPEC-015), data export generation (FEAT-29.SPEC-007) and closure orchestration with a 30-day cooling-off and deletion (FEAT-29.SPEC-008, FEAT-29.SPEC-013) combine identity and messaging capabilities with scheduled jobs.

**Required Capabilities:**
- Email or mobile OTP sign-in with `sign-in-code-expiry-minutes`, lockout, anti-enumeration (FEAT-29.SPEC-001, FEAT-29.SPEC-011; XBR-29; ASMP-30)
- Signed-in device list, 30-day inactivity expiry, sign-out everywhere, new-device alerts (FEAT-29.SPEC-006, FEAT-29.SPEC-015)
- Recovery via remaining contact; dual-confirmation contact change (FEAT-29.SPEC-002, FEAT-29.SPEC-010, FEAT-29.SPEC-012)
- Spreadsheet-friendly export of clients, bookings and deposits with progress (FEAT-29.SPEC-004, FEAT-29.SPEC-007)
- Closure: bulk cancel with refunds, subscription cancel, page down, cooling-off, deletion keeping de-identified financial records; reopening (FEAT-29.SPEC-008, FEAT-29.SPEC-009; XBR-20)
- Concurrency: sessions on two devices; closure sequencing against in-flight bookings and refunds (feature-dependency-map.md, Pro Account **Contention:**; FEAT-29.SPEC-013)
- Offline/degraded: signed-in Pro keeps read-only access to last loaded schedule; code sending shows in-place indicator; code delivery failures follow FEAT-08.SPEC-009 (feature-overview.md ## Non-Functional Notes)
- Scale: export covers multi-year history (100–500 clients, 20–40 bookings/week) and must stay reliable (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Supabase Auth (email/phone OTP, sessions, RLS integration), Clerk (OTP, session and device management, SMS OTP pricing), WorkOS AuthKit (magic auth), Auth0 (passwordless OTP) or Better Auth / Auth.js (full control of device-alert logic, own session tables) (Authentication & Identity); managed options differ in whether new-device alerts, lockout and anti-enumeration rules match the spec out of the box. Export as a job in Inngest, Trigger.dev (long-running tasks), BullMQ with Redis or Postgres-native queue and cron (Background Jobs & Scheduling), stored in Supabase Storage, Cloudflare R2, Amazon S3 or Vercel Blob (File & Object Storage). Code delivery via Twilio/Telnyx and Resend/Postmark/Amazon SES with SNS (Email & Messaging Delivery).

**Risks & Unknowns:** Managed auth products may not natively support all spec'd rules (dual-confirmation contact change, 30-day device inactivity, new-device alert on existing contacts), pushing custom code around them. Deletion after cooling-off must also purge data held by third parties (auth vendor user store, messaging logs, file storage). SMS OTP is exposed to toll-fraud abuse.

**Spike Recommendation:** None

### FEAT-30 — Pro Booking Management

**Verdict:** Standard-with-integration — Pro cancel/reschedule/goodwill/bulk commits (FEAT-30.SPEC-007..010 ## Processing Logic) call processor refunds with per-booking independent retry (FEAT-30.SPEC-011 ## Degradation Behavior, ## Edge Cases) and deliver deposit requests by text, email or on-screen scan code (FEAT-30.SPEC-013); the contention rule is explicit (feature-dependency-map.md, Booking **Contention:**).

**Required Capabilities:**
- Pro cancel with full refund; Pro reschedule inside notice/horizon exception with deposit carried over and fresh manage link (FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-006; XBR-03, XBR-18)
- Goodwill refund until completion (FEAT-30.SPEC-003, FEAT-30.SPEC-009; XBR-12)
- Bulk cancellation with per-booking outcome reporting (FEAT-30.SPEC-005, FEAT-30.SPEC-008)
- Book-client-in with a deposit-request hold up to 24h or 2h before appointment (FEAT-30.SPEC-004, FEAT-30.SPEC-010; FEAT-03.SPEC-007)
- Deposit request delivered as link or on-screen scan code; delivery stops on expiry (FEAT-30.SPEC-013)
- Concurrency: High — Pro actions race client self-service and automations; first committed wins; bulk siblings refunded independently (feature-dependency-map.md, Booking and Deposit Transaction **Contention:**; FEAT-30.SPEC-011 ## Edge Cases)
- Offline/degraded: actions require connection; a refund that cannot complete shows "Cancelled — refund in progress", never a failure, and never reverses the committed cancellation (FEAT-30.SPEC-011 ## Degradation Behavior); loaded booking-in screen readable (technical-profile.md Section 3, Offline: FEAT-30.SPEC-004)
- Scale: cancel/reschedule in under 30 seconds from the dashboard; a bulk action can cover a full day's bookings (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Refunds via Stripe (Connect plus Billing), Adyen for Platforms, PayPal Complete Payments or Square (Payments & Billing) with idempotency per Deposit Transaction. Bulk fan-out with per-item retry via Inngest step functions, Trigger.dev, Postgres-native queue and cron or BullMQ with Redis (Background Jobs & Scheduling). Deposit-request links as processor-hosted payment links or the FEAT-07 payment page.

**Risks & Unknowns:** The on-screen scan code (FEAT-30.SPEC-013) needs QR-code generation, for which the landscape names no option — recorded as a gap in Section 5. Bulk cancellation on a sick day concentrates refunds against one connected account, raising the insufficient-balance exposure noted for FEAT-09.

**Spike Recommendation:** None

## 3. Cross-Feature Technical Themes

| Theme / Shared Subsystem | Features Involved | Evidence That Makes It Shared |
|--------------------------|-------------------|-------------------------------|
| Slot reservation and booking-contention core (holds, exclusion, first-committed-wins) | FEAT-03, FEAT-05, FEAT-07, FEAT-10, FEAT-17, FEAT-20, FEAT-21, FEAT-30 | FEAT-03.SPEC-002/SPEC-005/SPEC-007 holds and contention; FEAT-05.SPEC-006 checkout hold; FEAT-17 block vs. hold (Time Block **Contention:**); FEAT-20.SPEC-004 priority window; FEAT-21 occurrence reservations (FEAT-03.SPEC-001 ## Edge Cases); FEAT-30.SPEC-010 deposit-request hold; XBR-01, XBR-02 |
| Background job execution, timers and retries | FEAT-02, FEAT-03, FEAT-04, FEAT-08, FEAT-09, FEAT-12, FEAT-17, FEAT-18, FEAT-20, FEAT-21, FEAT-25, FEAT-27, FEAT-29, FEAT-30 | 31 scheduled/timed Automation specs (technical-profile.md Section 3, Background processing), e.g. FEAT-03.SPEC-003 hold expiry, FEAT-08.SPEC-007 reminders, FEAT-09.SPEC-006 refund retry, FEAT-12.SPEC-004 sweep, FEAT-18.SPEC-003 renewals, FEAT-29.SPEC-008 cooling-off |
| Outbound client/Pro messaging with consent gating, timing window and retry-then-fallback | FEAT-06, FEAT-08, FEAT-10, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | All deliver through FEAT-08.SPEC-012/SPEC-013 (feature-dependency-map.md ## External Touchpoints, text and email rows); consent rule XBR-15, timing XBR-16, fallback XBR-17 |
| Payment-processor integration: charges, refunds, connected accounts, disputes, subscriptions, idempotent webhooks | FEAT-07, FEAT-09, FEAT-16, FEAT-18, FEAT-22, FEAT-23, FEAT-28, FEAT-30 | Seven Payment-processing Integration specs (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-30.SPEC-011), all specifying duplicate and out-of-order event handling in ## Edge Cases; ASMP-31 |
| Refund execution with indefinite idempotent retry and "in progress" surfacing | FEAT-09, FEAT-22, FEAT-23, FEAT-30, FEAT-12, FEAT-29 | FEAT-09.SPEC-006, FEAT-22.SPEC-005, FEAT-30.SPEC-011 share `refund-retry-interval-hours` and identical Degradation Behavior wording; FEAT-12.SPEC-005 attention flag; FEAT-29.SPEC-008 bulk refunds on closure; XBR-10 |
| Inbound webhook ingestion with event-time ordering and deduplication | FEAT-04, FEAT-07, FEAT-08, FEAT-14, FEAT-16, FEAT-18, FEAT-22, FEAT-26, FEAT-28 | Every Integration spec's ## Edge Cases lists "same event delivered twice" and "events arrive out of order … by its own event time" (e.g., FEAT-04.SPEC-003, FEAT-08.SPEC-012, FEAT-28.SPEC-006); landscape Section 3 notes public webhook endpoints are needed |
| Booking commit side-effect fan-out (deposit outcome, calendar mirror, activity event, freed slot, notices) | FEAT-05, FEAT-07, FEAT-10, FEAT-11, FEAT-16, FEAT-20, FEAT-30, FEAT-04 | FEAT-10.SPEC-004 and FEAT-30.SPEC-007/SPEC-008 coordinate the same side effects; FEAT-04.SPEC-005 mirrors every create/reschedule/cancel (XBR-13); FEAT-16.SPEC-002 records every event (XBR-21); XBR-28 freed-slot hand-off |
| Live-updating views (slot list, notices, banners) | FEAT-03, FEAT-05, FEAT-08, FEAT-10, FEAT-12, FEAT-22, FEAT-28 | technical-profile.md Section 3, Real-time: FEAT-03.SPEC-001, FEAT-07.SPEC-002, FEAT-08.SPEC-005, FEAT-22.SPEC-002, FEAT-28.SPEC-002; ASMP-21 |
| Read-only offline cache of last-loaded Pro data | FEAT-12, FEAT-13, FEAT-18, FEAT-24, FEAT-28, FEAT-29 | ASMP-27 schedule, client list and money list readable offline; feature-overview.md ## Non-Functional Notes of FEAT-12, FEAT-13, FEAT-18, FEAT-29; technical-profile.md Section 3, Offline |
| Per-Pro data isolation and role-scoped access (Pro, Client, read-only Support with masking) | FEAT-06, FEAT-12, FEAT-13, FEAT-16, FEAT-19, FEAT-24, FEAT-25, FEAT-27, FEAT-28, FEAT-29 | ASMP-23 privacy posture; XBR-24 support restrictions; FEAT-06.SPEC-008 isolation rule; FEAT-12.SPEC-008, FEAT-19.SPEC-004 authorization rules |
| Deletion and de-identified retention across stores | FEAT-13, FEAT-16, FEAT-29, FEAT-14 | XBR-19 client deletion and XBR-20 account closure; FEAT-13.SPEC-004 cascade; FEAT-16 retention (SC-22); FEAT-29.SPEC-013 retention rules |
| Timezone- and currency-aware computation | FEAT-02, FEAT-03, FEAT-07, FEAT-08, FEAT-17, FEAT-21, FEAT-22, FEAT-27 | XBR-25; ASMP-25; FEAT-02.SPEC-005, FEAT-03.SPEC-004 Pro-timezone labeling, FEAT-08.SPEC-007 local daytime window, FEAT-27.SPEC-003/SPEC-008 |
| Generated downloadable files | FEAT-16, FEAT-29 | FEAT-16.SPEC-004 dispute summary download; FEAT-29.SPEC-007 spreadsheet-friendly export; landscape File & Object Storage activation |

## 4. Key Technical Risks

| Risk | Features Affected | Driving Evidence | Possible Mitigation Directions |
|------|-------------------|------------------|--------------------------------|
| Double-booking under concurrent holds, blocks, deposit requests or waitlist claims | FEAT-03, FEAT-05, FEAT-17, FEAT-20, FEAT-21, FEAT-30 | FEAT-03.SPEC-005 ## Edge Cases; XBR-01; ASMP-26 ("never silently double-book"); Booking **Contention:** High | Enforce non-overlap in the database (exclusion constraints or serializable transactions) rather than application checks; the FEAT-03 spike measures the chosen approach under load |
| Apple/iCloud busy time not detected within "a couple of minutes", letting a client book over a personal commitment | FEAT-04, FEAT-03 | FEAT-04.SPEC-003 ## Inbound Events; landscape Calendar Sync area (CalDAV for iCloud, no push noted); ASMP-33 | Run the FEAT-04 spike; directions include frequent CalDAV polling, a unified provider (Nylas, Cronofy), or accepting a longer Apple latency with the Pro-visible confidence banner |
| One-second slot refresh breaks under burst traffic or serverless cold starts | FEAT-03, FEAT-05, FEAT-10 | ASMP-21; FEAT-03 feature-overview.md ## Non-Functional Notes; landscape Real-time & Collaboration polling row ("load scales with viewers") | Push transport options (Supabase Realtime, Ably, Pusher Channels) or warm/long-running hosting (Render, Fly.io); measured in the FEAT-03 spike |
| Refunds cannot complete because the Pro's connected balance was already paid out | FEAT-09, FEAT-22, FEAT-30, FEAT-29 | FEAT-09.SPEC-005 and FEAT-30.SPEC-011 ## Degradation Behavior (Rejects column); XBR-10 | Processor-specific negative-balance/debit behavior as a selection criterion in Payments & Billing; payout-delay settings; monitoring on long-running "in progress" refunds |
| Lost or out-of-order webhooks create wrong payment, consent or calendar state | FEAT-04, FEAT-07, FEAT-14, FEAT-16, FEAT-18, FEAT-22, FEAT-28 | Duplicate/out-of-order cases in every Integration spec's ## Edge Cases; landscape Section 3 idempotent-handler note | Event-time comparison and idempotency keys stored per event; periodic reconciliation against provider APIs; tracing via Observability & Operations options |
| US texting blocked or filtered until A2P 10DLC registration completes; STOP mapping ambiguous on shared numbers | FEAT-08, FEAT-14, FEAT-06, FEAT-29 | landscape Email & Messaging Delivery (10DLC onboarding); FEAT-14.SPEC-004; ASMP-24 | Start registration early; decide shared vs. per-Pro sending numbers; email fallback path (XBR-17) already covers interim delivery |
| Payment and card entry misbehave inside the Instagram in-app browser | FEAT-05, FEAT-07, FEAT-22 | BRIEF.md Devices & platforms; FEAT-05 feature-overview.md ## Non-Functional Notes; ASMP-28 | Device testing of the candidate processor's hosted/embedded elements and 3-D Secure in Instagram's browser before selection; hosted-page fallback |
| Personal data persists in third-party stores after deletion | FEAT-13, FEAT-29, FEAT-08, FEAT-16 | XBR-19, XBR-20; ASMP-23; landscape Observability & Operations and Analytics & Product Telemetry vendors | Minimize personal data sent to vendors; provider data-deletion APIs in the closure/deletion orchestration; scrubbing in error-tracking SDKs |
| Correctness regressions across 30 features with dense cross-feature rules | All | ASMP-26; 29 XBRs and 320 touchpoint rows (technical-profile.md Section 2) | Automated tests on booking, payment and refund logic in CI/CD & Delivery options; concurrency tests derived from the Contention lines |

## 5. Open Questions for the Build Team

| # | Question | Why It Matters | What Would Resolve It |
|---|----------|----------------|------------------------|
| 1 | Can Apple/iCloud busy time be detected within "a couple of minutes", and what does a Pro have to do to authorize iCloud access? | Determines whether FEAT-04 leaves Research-spike status and which Calendar Sync option fits (FEAT-04.SPEC-001, FEAT-04.SPEC-003) | The FEAT-04 spike outcome; if not achievable, a product decision on an accepted Apple latency |
| 2 | Does database-level exclusion deliver exactly one winner under concurrent holds, and at what viewer count does one-second polling breach ASMP-21? | Confirms the FEAT-03 Hard path and whether a push transport is needed at launch | The FEAT-03 spike (load test on a Postgres candidate and the candidate hosting) |
| 3 | Which payment processor's connected-account model, negative-balance/refund behavior and country coverage fit the zero-platform-fee rule and UK/CA/AU expansion? | Affects FEAT-07, FEAT-09, FEAT-22, FEAT-28, FEAT-30 refund feasibility and XBR-07 economics | Architect's Payments & Billing selection informed by processor documentation on connected-account refunds and supported countries |
| 4 | Shared sending number or per-Pro numbers for SMS, and how are inbound STOP replies mapped to a client–Pro pair? | FEAT-14.SPEC-004 revocation correctness and FEAT-08 10DLC registration scope | A product/build decision plus confirmation of the chosen provider's opt-out behavior |
| 5 | What file format do the dispute summary (FEAT-16.SPEC-004) and data export (FEAT-29.SPEC-007, "spreadsheet-friendly") take, and which generation library produces them? | The landscape names storage options but no document/CSV generation library — a landscape gap | A build-team choice of format and library; CSV likely needs none, a formatted summary may |
| 6 | How are the .ics add-to-calendar link (FEAT-08.SPEC-001) and the on-screen deposit-request scan code (FEAT-30.SPEC-013) generated? | No landscape option covers iCalendar or QR-code generation — a landscape gap affecting FEAT-08 and FEAT-30 | A build-team choice of small libraries or hand-written generation |
| 7 | Does the processor run subscription renewals (with the product following its events), or does the product schedule renewals itself? | Avoids two disagreeing grace clocks in FEAT-18.SPEC-003 / FEAT-18.SPEC-006 | Architect decision aligned with the Payments & Billing selection |
| 8 | What values are chosen for the 33 decide-before-build platform parameters, notably `checkout-hold-timeout-minutes`, `refund-retry-interval-hours` and the access-link rate limit? | Hold duration shapes FEAT-03 contention exposure; retry cadence shapes FEAT-09/FEAT-22/FEAT-30 job load; rate limit shapes FEAT-06 abuse exposure | Product-owner decisions recorded in platform-parameters.md |
| 9 | Is tipping at deposit time in scope, or only at balance payment? | FEAT-23 description mentions both; FEAT-23.SPEC-001 covers balance only; deposit tipping would touch FEAT-07 amount-lock rules | A product decision |
| 10 | Must EXIF metadata (including location) be stripped from uploaded profile photos? | Studio address may be a home address (FEAT-27 feature-overview.md Data sensitivity); affects FEAT-27.SPEC-012 processing | A product/privacy decision; File & Object Storage options differ in built-in image processing (Supabase Storage, Cloudinary) |


## The Decisions — Technical Architecture

Section 1 below repeats the technical profile — by design; it is the architecture document's own embedded evidence base.


# Technical Architecture -- Chairtime

## 1. Project Technical Profile

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

## 2. Non-Functional Requirements & Scale Design

| Dimension | Expectation | Source |
|-----------|-------------|--------|
| Launch load | "a few hundred pros in year one. Each pro has roughly 100–500 clients and 20–40 bookings a week." At ~300 pros × ~30 bookings that is roughly 9,000 bookings a week (~1,300 a day), 30,000–150,000 client records, and at least two outbound messages per booking. Assumption — the first launch months carry tens of pros ramping toward that figure (no upstream ramp curve). | Profile Section 7, BRIEF.md Scale & Non-Functional Expectations (Volume); ASMP-22. The derived weekly/daily figures are arithmetic on the quote |
| Growth trajectory | "the product staying equally responsive as pros accumulate history over multiple years"; "the UK, Canada and Australia are the obvious next markets". Assumption — low thousands of pros within two years if expansion markets open; no numeric growth target exists upstream. | Profile Section 7, ASMP-22 and BRIEF.md Geography; growth figure is "Assumption — extrapolated from the four named markets" |
| Burst load on one booking page | Assumption — up to ~50 simultaneous viewers polling one Pro's slot list after a viral Instagram post; feasibility (FEAT-03 Risks) names the burst but no figure exists upstream. | Assumption — technical-feasibility.md FEAT-03 Risks & Unknowns ("not quantified upstream") |
| Performance targets | "available slots appear within roughly one second of a service selection, and a full booking (selection through paid confirmation) completes in under one minute"; confirmations "arrive within about a minute of payment" | Profile Section 7, ASMP-21, ASMP-29 |
| Availability posture | "Reliability is expressed as a correctness bar, not a numeric uptime target: the product must never silently double-book a slot or lose a deposit." Assumption — single-region managed hosting (US East) with provider-standard uptime is adequate; correctness mechanisms (database constraints, idempotent retries) matter more than redundancy. | Profile Section 7, ASMP-26 and BRIEF.md ("No specific uptime number was given"); hosting posture is "Assumption — no numeric target upstream" |
| Offline & loading posture | "anything that books, pays, cancels, refunds or marks a no-show needs a live connection …; the pro's most recently loaded schedule, client list and money list stay readable offline" | Profile Section 7, ASMP-27 |
| Devices & accessibility | "mobile-first web for both roles … the client books from inside the Instagram in-app browser"; "fully operable at phone width inside a social-media in-app browser … full use by screen-reader users" | Profile Section 7, BRIEF.md Devices & platforms; ASMP-28 |
| Security posture | "a pro's account … is protected by a one-time-code sign-in with new-device alerts; client access links are short-lived and open only that client's bookings with that one pro"; "a client's data is visible only to their own pro and to themselves" | Profile Section 7, ASMP-30, ASMP-23 |
| Compliance obligations | "US SMS-consent rules apply to all client texting (explicit opt-in captured at booking, honored immediately on opt-out); no health-data regime applies"; the payment processor "owns all card data" (PCI scope held by the processor); "a pro can permanently delete a client's record on request". Assumption — UK expansion would add UK-GDPR-class handling; the deletion and minimization design below already satisfies its core obligations. | Profile Section 7, ASMP-24, ASMP-31, ASMP-23; UK note is "Assumption — expansion market named in BRIEF.md Geography" |
| Localization | "timezone and currency are per-account configuration from day one, never hard-coded"; no multi-language behavior is specified | Profile Section 7, ASMP-25; profile Section 3 Internationalization |

The binding drivers are **correctness under concurrency** and **one-second slot responsiveness inside the Instagram in-app browser**. ASMP-26 ("never silently double-book a slot or lose a deposit") combined with the profile's Collaboration/concurrency signal (16 of 18 entities with contention, Booking contention High, XBR-01) pushes the architecture toward a relational database that enforces non-overlap and state transitions itself (exclusion constraints, compare-and-set updates, transactional outbox) and toward a durable job runner with idempotent retries for the 31 scheduled/timed Automation specs and the refund/webhook chains. ASMP-21 plus the in-app-browser constraint shapes rendering (server-rendered public pages with small client bundles), hosting region co-location with the database, and the polling-first live-update transport. Privacy (ASMP-23, ASMP-30) is the third binding driver: per-Pro isolation must be enforced below application code, and personal data sent to vendors must be minimized so deletion (XBR-19, XBR-20) can be complete.

Launch and year-one load are modest — roughly 1,300 bookings a day across all pros and at most 500 clients per Pro — and sit comfortably inside every candidate's scale ceiling. Cost efficiency at small scale and low operational burden for a small team matter more than scale ceilings; the only load dimension that could force a change is burst polling on a single Pro's page, which the FEAT-03 spike measures before launch.

## 3. Technology Stack Decisions

### Frontend Framework

| Field | Value |
|-------|-------|
| Context | Scale: Large (30 features, 219 specs); 70 Screen specs; Interaction Complexity: Large (59 Automation, Real-time and Collaboration/concurrency signals present); Complex forms signal: 8-step wizard FEAT-15.SPEC-001..003 and multi-step booking flow FEAT-05.SPEC-001..009; Section 2: one-second slots (ASMP-21), mobile-first inside the Instagram in-app browser, screen-reader operable (ASMP-28); feasibility FEAT-05 Standard-with-integration (server-rendered public pages), FEAT-03 Hard |
| Recommended | Next.js 15 (App Router, React 19, TypeScript) |
| Rationale | The public booking page is the highest-traffic surface (feasibility FEAT-05) and must load fast in an in-app browser: App Router server components render the landing, service list and slot list on the server and ship minimal client JavaScript, while server actions and route handlers serve the 70 Screen specs' mutations and the 13 Integration specs' webhooks from one deployable. The landscape entry cites "large component ecosystem for wizard and dashboard screens" — the React ecosystem supplies the accessible date/time and dialog primitives ASMP-28 demands (Section 9). Mainstream React skills lower team risk for a Large-scope build. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| React Router v7 (framework mode) | Open source; host cost only | Low–Medium — adapter per runtime | High — same React runtime limits | Low — portable across Node, edge and serverless adapters | Mainstream React plus loader/action model | The team wants progressive-enhancement forms as the default and plans to host on a non-Vercel Node platform (e.g. Render, Fly.io) from day one |
| SvelteKit | Open source; host cost only | Low–Medium | High | Medium — Svelte-specific component ecosystem | Specialist — Svelte skills less common | Instagram in-app-browser measurements show React bundle size breaching the one-second slot target (ASMP-21) and the team already knows Svelte |
| Nuxt | Open source; host cost only | Low–Medium — Nitro adapters | High | Medium — Vue-specific ecosystem | Mainstream Vue | The implementing team is Vue-first; every other decision below remains valid with Vue equivalents (TanStack Query Vue adapter, Headless UI) |

### Backend / API Layer

| Field | Value |
|-------|-------|
| Context | Frontend: Next.js 15 (ADR-001); 59 Automation specs; 13 Integration specs with inbound webhooks (payments, SMS status and STOP, calendar); idempotent payment outcomes (FEAT-07.SPEC-004); hold/contention logic (FEAT-03.SPEC-002, FEAT-03.SPEC-005); feasibility theme "Inbound webhook ingestion with event-time ordering and deduplication" across 9 features |
| Recommended | Next.js server layer (route handlers for webhooks and the JSON endpoints the client polls, server actions for form mutations) on the Node.js runtime, with all long-running, delayed and retried work delegated to the Background Jobs selection (ADR-012) |
| Rationale | The landscape entry: "Single deployable for UI and API; webhooks as route handlers; long-running work must be delegated to a job runner". A separate service adds a second deployable and network hop inside the one-second budget without adding capability the profile demands; the 31 scheduled/timed Automation specs go to the durable job runner rather than a standing backend. Domain logic lives in plain TypeScript modules under `src/features/*` and `src/server/*` (Section 5), so extracting a standalone service later is a move, not a rewrite. Node runtime (not edge) because the ORM driver, Stripe and Twilio SDKs are Node-first (landscape Cross-Area Compatibility Notes). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| NestJS (separate Node service) | Open source; second host (~$7–25/month container) | Medium — second deployable, inter-service auth | High | Low — Node-bound | Medium — DI/module conventions | The team grows past ~5 backend engineers or a native mobile app joins, and a stable versioned API with DI-enforced guards across 30 features outweighs single-deployable simplicity |
| Hono on Node | Open source; runs inside or beside the Next.js host | Low | High | Low — portable across runtimes | Low — shares TypeScript | Webhook and polling endpoints need to move to a warm long-running host (FEAT-03 spike shows serverless cold starts breach ASMP-21) while the UI stays on Vercel |
| Ruby on Rails | Open source; container host | Medium — separate stack from the React UI | High | High — convention lock-in | Specialist Ruby | The implementing team is a Rails shop and would rather build the whole product server-rendered with Hotwire than use React |

### Database

| Field | Value |
|-------|-------|
| Context | 18 entities, 56 relationships, Data Complexity: Medium; Collaboration/concurrency: 16 of 18 entities with contention, Booking High, XBR-01 first-committed-wins; Section 2: never double-book/lose a deposit (ASMP-26), per-Pro isolation (ASMP-23), multi-year history (ASMP-22); feasibility FEAT-03 Hard — "Correctness under concurrent writes depends on the constraint being enforced in the database"; Key Risk 1 mitigation "exclusion constraints or serializable transactions" |
| Recommended | Supabase Postgres (Pro tier, US East region), used as standard Postgres 15+ with the `btree_gist` extension for exclusion constraints and row-level security as defense-in-depth for per-Pro isolation |
| Rationale | Postgres exclusion constraints on `tstzrange` columns enforce non-overlap of bookings, holds and time blocks at the data layer — the landscape Section 3 note and feasibility FEAT-03's recommended mitigation. The Pro tier is always-on (no scale-to-zero cold start against the one-second budget, unlike serverless Postgres), costs $25/month at year-one data volumes (landscape: 8 GB included), and the same project supplies the File & Object Storage selection (ADR-008) and a push-transport upgrade path (Supabase Realtime, ADR-014) without a second vendor. Data stays Postgres-standard: exit is `pg_dump` to any landscape Postgres option. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Neon | Free 0.5 GB; Launch pay-as-you-go $0.106/CU-hour, $0.35/GB-month | Low — serverless, branching per preview | High — autoscaling compute | Low — standard Postgres | Mainstream SQL | Per-pull-request database branches matter more than bundled storage, and the FEAT-03 spike shows compute wake-up stays inside the one-second target with autosuspend disabled |
| Amazon RDS / Aurora PostgreSQL | Instance-hour plus storage; tens of dollars/month at entry | Medium — VPC, IAM, parameter groups | Very high — Multi-AZ, read replicas | Low data lock-in; AWS operational coupling | AWS operations skills | Hosting moves to AWS (e.g. compliance audit demands a single cloud boundary) or Multi-AZ failover becomes a stated availability requirement |
| Crunchy Bridge | From ~$10/month; storage $0.10/GB/month | Low | High | Low — plain Postgres | Mainstream SQL | The team wants plain managed Postgres with PITR and SOC 2 Type 2 evidence and no platform extras |

### ORM / Data Access

| Field | Value |
|-------|-------|
| Context | Database: Supabase Postgres (ADR-003); backend: Next.js on Node (ADR-002); 18 entities; concurrency-sensitive writes needing explicit transactions, `SELECT … FOR UPDATE`, compare-and-set on version columns, and exclusion constraints (feasibility FEAT-03, FEAT-10, FEAT-11 Candidate Approaches); migrations across 30 features |
| Recommended | Drizzle ORM (with drizzle-kit migrations) over the `postgres` (postgres.js) driver through Supabase's connection pooler (transaction mode) |
| Rationale | Landscape: "SQL-like typed queries, migration tooling, close to SQL semantics for transactions" — the booking core is written as explicit transactions with row locks and guarded updates, which Drizzle expresses directly while keeping full type safety across 18 entities. Schemas in TypeScript give low lock-in; custom SQL (exclusion constraints, RLS policies, append-only grants on Activity Event) lives in the same migration files. Pairs with the Node backend per the compatibility rules. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Prisma | Open source | Low | High | Medium — proprietary schema DSL | Low — most familiar TS ORM | The team already runs Prisma and accepts writing the contention paths (holds, exclusion constraints, `FOR UPDATE`) as raw SQL escape hatches |
| Kysely | Open source | Low–Medium — bring a migration tool | High | Low | Medium — SQL fluency | The team prefers a pure query builder and wants migrations as hand-written SQL files managed separately |
| supabase-js / PostgREST client | Included in Supabase plan | Low | High | High — ties data access to PostgREST | Low–Medium; multi-step transactions become SQL functions | Most reads move to the browser under RLS and the team accepts expressing every multi-step booking commit as a Postgres function |

### CSS / Styling

| Field | Value |
|-------|-------|
| Context | 70 Screen specs, mobile-first; ASMP-28 (text that scales, contrast, tap targets, screen-reader use); design_system_source: none (design-agnostic, Section 9); Frontend: Next.js (ADR-001) |
| Recommended | Tailwind CSS v4 (CSS-first `@theme` configuration with CSS custom properties as the token layer) |
| Rationale | Landscape: "fast responsive layouts; works with all mainstream frameworks" and tokens map to theme config — the downstream builder can install any visual design as CSS variables without restructuring components. No runtime cost in the in-app browser (ASMP-21). Tailwind is also the prerequisite for the selected component layer (shadcn/ui, ADR-023). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| CSS Modules with CSS custom properties | Free (web standard) | Low | High | None — standards-based | Mainstream CSS | The downstream builder brings a hand-crafted visual design and prefers scoped stylesheets over utility markup |
| vanilla-extract | Open source | Medium — bundler plugin | High | Medium — TS-authored styles | Medium | A supplied design system arrives later with a large typed token set that benefits from compile-time checked themes |
| Panda CSS | Open source | Medium — codegen step | High | Medium | Medium | The team wants typed, token-driven style props with zero runtime and accepts a codegen step |

### State Management

| Field | Value |
|-------|-------|
| Context | Real-time signal: ~1-second slot refresh (FEAT-03.SPEC-001), banners/notices updating without refresh (FEAT-08.SPEC-005, FEAT-28.SPEC-002); Collaboration/concurrency: reject-with-refresh everywhere; Offline signal: read-only cache of schedule, client list and money list (ASMP-27); Complex forms: 8-step resumable wizard (FEAT-15.SPEC-004), multi-step booking flow; Interaction Complexity: Large |
| Recommended | TanStack Query v5 for all server state (polling, cache, invalidation, offline persistence) plus framework built-ins (React state, URL search params, server-persisted wizard progress) for client state; no global client store |
| Rationale | Landscape: "Polling/refetch, caching, optimistic updates for slot lists and dashboards". `refetchInterval` delivers the one-second slot cadence; query invalidation after a server action implements reject-with-refresh; the persist plugin keeps the last-loaded schedule/client list/money list readable offline (ASMP-27, ADR-013). Wizard progress is server-persisted by design (FEAT-15.SPEC-004) and booking-flow state is carried in the URL plus the server-side hold, so a separate store would duplicate state. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| SWR | Open source | Low | High | Low — React-only | Low | The team wants the smallest possible polling layer and does not need offline persistence (ASMP-27 dropped) |
| Zustand (alongside TanStack Query) | Open source | Low | High | Low | Low | Booking-flow or wizard state must survive in-page without URL/server round-trips (e.g. a future offline-first draft mode) |
| Redux Toolkit with RTK Query | Open source | Medium — boilerplate | High | Low | Medium | The team standardizes on Redux across products and wants one store for server and client state |

### Build Tooling

| Field | Value |
|-------|-------|
| Context | Frontend: Next.js 15 (ADR-001); one web application in one repository; TypeScript across 30 features and 219 specs |
| Recommended | Framework-bundled pipeline (Next.js with Turbopack for dev and build) with pnpm as the package manager |
| Rationale | Landscape: "No separate bundler config; pairs with the chosen framework" — overriding the meta-framework's pipeline needs a documented driver and the profile provides none. pnpm's strict dependency resolution prevents phantom dependencies in a codebase of this scope. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| npm (with the framework pipeline) | Free | Low | High | None | Lowest | The team or CI image standardizes on npm and strictness is enforced by lint rules instead |
| Turborepo (with pnpm workspaces) | Open source; optional paid remote cache | Medium | High | Low; optional Vercel-coupled cache | Medium | The product splits into multiple packages (e.g. a separate worker or a native app sharing domain code) — see ADR-018 |

## 4. Platform & Service Decisions

### File & Object Storage

| Field | Value |
|-------|-------|
| Context | File upload signal: FEAT-27.SPEC-001, FEAT-27.SPEC-012 (profile photo within size/format limits; ASMP-35); Import/export signal: FEAT-29.SPEC-007 data export, FEAT-16.SPEC-004 dispute summary; database: Supabase (ADR-003); feasibility FEAT-27 Risks: EXIF location metadata (Open Question 10), public page must reflect changes immediately |
| Recommended | Supabase Storage (same project as the database): a public `profile-photos` bucket served through Supabase image transformations, and a private `exports` bucket for data exports and dispute summaries delivered by short-lived signed URLs |
| Rationale | Landscape: "Buckets with row-level-security policies; image transformations" at 100 GB included on the Pro plan — photos and generated files for a few hundred pros are far below that, so it adds no vendor and no cost. Uploads go through a server action that validates size/format, re-encodes the image to strip EXIF metadata (answering feasibility Open Question 10 conservatively, since the studio address may be a home address), and writes a content-addressed key so the booking page picks up a new photo immediately without CDN invalidation. Exports and dispute summaries are private by default and expire. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Cloudflare R2 | Storage per GB, zero egress | Low–Medium — separate account, S3 API | Very high | Low — S3 API | Mainstream | The database moves off Supabase, or public-photo egress grows large enough that zero-egress pricing matters |
| Amazon S3 | Per-GB storage plus request and egress fees | Medium — IAM and CORS | Very high | Low — de facto standard | AWS skills | Hosting moves to AWS (ADR-028 alternative) |
| Cloudinary | Free tier plus credit-based plans | Low | High | Medium — proprietary transformation URLs | Low | Photo handling expands (multiple portfolio images, cropping UI, format negotiation) beyond a single profile photo |

### Email & Messaging Delivery

| Field | Value |
|-------|-------|
| Context | Notifications signal: 24 Notification specs (Text and Email most; In-app 7; Email and SMS for FEAT-29.SPEC-014..017); Integration specs FEAT-08.SPEC-012 (text), FEAT-08.SPEC-013 (email), FEAT-26.SPEC-002 (WhatsApp, Later); ASMP-24 US SMS consent with STOP honored immediately; ASMP-29 8am–9pm window, confirmation within a minute; BRIEF "WhatsApp is a nice-to-have later"; feasibility risks: A2P 10DLC registration, shared-number STOP mapping (Open Question 4), SMS toll fraud |
| Recommended | Twilio Programmable Messaging (one Messaging Service with a registered A2P 10DLC brand/campaign on a shared platform number pool, Advanced Opt-Out enabled; WhatsApp sender added on the same account for FEAT-26 later) for text, and Postmark (transactional message stream, delivery and bounce webhooks) for email |
| Rationale | Twilio is the only landscape option that also carries WhatsApp (feasibility FEAT-26: "provider choice for this channel is effectively constrained"), so choosing it now avoids a second SMS/WhatsApp integration later; it provides delivery-status callbacks and inbound STOP webhooks (FEAT-08.SPEC-012, FEAT-14.SPEC-004) at $0.0083/segment. Shared-number decision (Open Question 4): one platform campaign keeps 10DLC registration to a single brand; because carrier-level STOP on a shared number blocks that phone for every Pro sending from it, an inbound STOP revokes texting consent on every Client record with that phone number, which applies FEAT-14.SPEC-006's "default to no-text when uncertain" rule — a client re-grants per Pro from preferences, and the product re-enables with Twilio via START. Postmark is transactional-first with delivery tracking (landscape) and keeps email reputation separate from any marketing sender; $15/month covers 10,000 emails. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Telnyx (SMS) with Resend (email) | SMS $0.004/part plus carrier passthrough; Resend free 3,000/month, Pro $20–35/month | Low–Medium | High | Medium — number registrations tied to account | Low | SMS spend becomes the dominant cost line (year-one volume approaches 100,000+ texts/month) and WhatsApp stays out of scope |
| Twilio (SMS) with Resend (email) | As Twilio; Resend free tier then $20/month | Low | High | Medium | Low | The team prefers Resend's React-email developer experience and accepts a newer email vendor |
| Amazon SES with SNS | Per-1,000 email pricing; SMS via separate AWS service | Medium — IAM, sender verification | Very high | Medium — AWS coupling | AWS skills | Hosting moves to AWS and consolidating vendors outweighs 10DLC and WhatsApp convenience |

### Payments & Billing

| Field | Value |
|-------|-------|
| Context | Payments/billing signal: 44 payment-related specs; 7 Payment-processing Integration specs (FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-28.SPEC-006, FEAT-30.SPEC-011); ASMP-31 and BRIEF "Established card payment processor: takes client deposits and pays out to the pro, and owns all card data. Also used for the pro's subscription billing."; XBR-07 zero platform fee; refunds "drawing on the Pro's payout account"; UK/CA/AU expansion; feasibility Open Questions 3 and 7; Key Risk: refunds after payout |
| Recommended | Stripe Connect (connected accounts with Stripe-hosted onboarding for identity/bank verification; direct charges on the Pro's connected account with no application fee, so processor fees, refunds and disputes sit on the Pro's balance) plus Stripe Billing on the platform account for the Pro's monthly subscription |
| Rationale | Landscape: "Connected accounts with hosted identity/bank onboarding, destination charges, refunds, dispute webhooks, subscriptions" in one platform, at card 2.9% + 30¢, with idempotency keys supporting FEAT-07.SPEC-004 and FEAT-09.SPEC-006. Direct charges put the charge, the processor fee report (FEAT-07.SPEC-005), refunds "drawing on the Pro's payout account" (FEAT-09.SPEC-005, FEAT-30.SPEC-011) and card-issuer disputes (FEAT-16.SPEC-003) on the Pro's account, which is exactly the spec'd value flow with zero platform fee (XBR-07); the per-active-account Connect fee ($2/month under platform-handled pricing) is a business cost to confirm against `subscription-price`. Open Question 3: Stripe supports connected accounts in the US, UK, Canada and Australia, covering the named expansion markets; negative-balance refunds are retried per `refund-retry-interval-hours` and a payout-schedule delay is configured on connected accounts to reduce the insufficient-balance risk. Open Question 7: Stripe Billing runs renewals and the product's grace clock (FEAT-18.SPEC-003) keys off Stripe's `invoice.payment_failed`/`invoice.paid` events — one clock, not two. Card entry uses Stripe's hosted Payment Element so the product holds no card data (SC-11). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Adyen for Platforms | Interchange-plus with per-transaction fee; enterprise sales | Medium–High | Very high | High — card tokens with vendor | Specialist | Volume reaches enterprise scale where interchange-plus pricing beats blended 2.9% + 30¢ and a sales-led contract is acceptable |
| PayPal Complete Payments | Per-transaction percentage plus fixed fee | Medium | High | High | Medium | Client-side PayPal wallet adoption proves material for the Instagram audience and seller onboarding covers all target markets |
| Square (Payments and Connect) | Per-transaction percentage plus fixed fee | Medium — seller OAuth accounts | High | High | Medium | Most target pros already run Square for in-person payments and prefer authorizing their existing seller account over new onboarding |

### AI & Intelligent Behavior

Not activated — profile Section 3: "No specs reference AI or machine-learning behavior (keyword hits in FEAT-03.SPEC-001, FEAT-16.SPEC-004, FEAT-30.SPEC-003 and FEAT-02.SPEC-004 are negations or rule-evaluation wording, for example "not a recommendation engine"); BRIEF.md and assumptions-constraints.md Dependencies name no AI capability"

### Search

| Field | Value |
|-------|-------|
| Context | Search signal: "Simple filter -- FEAT-24.SPEC-001, FEAT-24.SPEC-002 … No spec names full-text or faceted search"; 100–500 clients per Pro (ASMP-22); privacy posture ASMP-23; feasibility FEAT-24 Straightforward ("an external search service would add a personal-data processor") |
| Recommended | PostgreSQL built-ins: `pg_trgm` GIN indexes on normalized client name and phone, scoped by `pro_account_id`, with `ILIKE` partial matching; the loaded per-Pro list is also filtered client-side from the TanStack Query cache for offline filtering |
| Rationale | Landscape: "Match and filter over a per-pro record set with no extra service" at no added cost. At ≤500 clients per Pro every query is a single-tenant index scan well inside "instant" results, and no client personal data leaves the primary database — keeping deletion (XBR-19) complete. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Meilisearch (Cloud) | Cloud from $20/month; self-hosting free | Medium — data sync | High | Low — open source | Medium | Pros ask for typo-tolerant search across clients, notes and bookings together, and a data-processing agreement covers the personal data synced |
| Typesense | Self-hosted free; cloud via calculator | Medium — data sync | High | Low | Medium | Same trigger as Meilisearch, with a preference for self-hosting inside the same cloud region |
| Algolia | Free 10K requests/month; $0.50 per additional 1K | Low–Medium — secured API keys per Pro | Very high | High — proprietary | Low | Search becomes a primary UX surface across many entities and per-Pro secured keys are acceptable |

### Background Jobs & Scheduling

| Field | Value |
|-------|-------|
| Context | Background processing signal: 31 scheduled/timed Automation specs (e.g. FEAT-03.SPEC-003 hold expiry, FEAT-08.SPEC-007 reminders, FEAT-09.SPEC-006 refund retry, FEAT-12.SPEC-004 sweep, FEAT-18.SPEC-003 renewals, FEAT-29.SPEC-008 30-day cooling-off); Import/export (FEAT-29.SPEC-007); feasibility themes: booking-commit side-effect fan-out, refund retry "never dropped", bulk cancellation with per-item retry (FEAT-30.SPEC-008); hosting: serverless (ADR-028) — no long-running worker |
| Recommended | Inngest (managed durable functions: event-triggered steps, `sleepUntil` delays, cron, per-step retries and concurrency keys), invoked through a Next.js route handler, fed by a transactional outbox table (ADR-022) |
| Rationale | Landscape: "Durable step functions, delayed events, retries, cron" and it "calls into an HTTP endpoint", so it runs on serverless hosting where BullMQ cannot. Durable steps map one-to-one onto spec'd chains: reminders scheduled at `reminder-lead-time-days` shifted into the 8am–9pm window (`sleepUntil`), refund retry every `refund-retry-interval-hours` until success, bulk cancellation fan-out with independent per-booking retry, 30-day closure cooling-off, and data export generation. Concurrency keys per `pro_account_id` serialize per-Pro work. Free 50k executions/month covers launch; Pro from $99/month at year-one volume. Correctness-critical expiries do not depend on the job runner: slot holds carry `expires_at` and are ignored once past (lazy expiry), with a job only reclaiming rows. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Trigger.dev | Free $5 credit; Hobby $10/month; Pro $50/month plus per-run compute | Low–Medium | High | Low–Medium — open-source core, self-hostable | Low | Long-running tasks (large multi-year exports) exceed serverless step durations, or self-hosting the job platform becomes a requirement |
| Postgres-native queue and cron (pg_cron with pgmq) | Included in database cost | Medium — polling worker or scheduled invocations | Medium — queue load shares the primary database | Low — Postgres-standard | Medium — SQL-heavy | The team wants zero additional vendors and accepts building retry/backoff and delayed-step orchestration itself |
| BullMQ with Redis (Upstash) | Open source plus Redis hosting | Medium — needs a long-running worker | High | Low–Medium — Node-specific | Medium | Hosting moves to a platform with long-running workers (Render, Fly.io) and Redis is already present |

### Caching & Performance

| Field | Value |
|-------|-------|
| Context | Scale hints: ASMP-21 (slots within ~1 second), ASMP-22 (equally responsive over multi-year history), ASMP-26; Offline signal: ASMP-27 read-only cache of schedule, client list and money list (33 specs with an Offline/Degraded already-loaded row); feasibility FEAT-03 ("Computation can stay uncached on Database indexing and query design … given small per-Pro data"), FEAT-12 (service-worker persistence unreliable in in-app browsers), FEAT-27 (edge caching vs immediate changes) |
| Recommended | Database indexing and query design as the only server-side performance layer (composite indexes leading with `pro_account_id`, GiST indexes on time ranges, no separate cache service), plus client-side TanStack Query cache persisted to IndexedDB for the Pro's offline read-only views |
| Rationale | Landscape: "Booking data per pro is small; indexed queries can serve slot computation" — a Pro's slot computation touches one week-scale window of ≤40 bookings, rules, blocks and busy periods, so it is index-bound, not cache-bound, and history growth does not slow it when every query is bounded by `pro_account_id` and a time range. Adding Redis would create a second source of truth for holds, which feasibility FEAT-03 flags as a correctness cost. ASMP-27's offline promise is met by the persisted query cache (the Pro uses a normal mobile browser, lowering FEAT-12's persistence risk). Public booking-page responses are dynamic (no CDN caching of slot or profile data), satisfying FEAT-27's "shows the change immediately". |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Upstash Redis | Free 256 MB; $0.20 per 100K commands | Low | High | Low — Redis protocol | Low | The FEAT-03 spike shows burst polling saturates the database; cache computed slot lists per Pro/service/day for ~1 second and invalidate on any booking, hold or block write |
| CDN/edge caching of the public booking page | Included in hosting plan | Low | Very high | Medium — platform cache controls | Low | Public-page traffic grows to where server rendering the shell per request is material; cache the shell with tag-based revalidation on profile save and keep slots uncached |
| Client-side cache with a service worker | Free | Medium — worker lifecycle and invalidation | High | None | Medium | Pros need the app shell itself to open offline (not only the last-loaded data), e.g. an installable PWA |

### Real-time & Collaboration

| Field | Value |
|-------|-------|
| Context | Real-time signal: FEAT-03.SPEC-001 (~1-second slot refresh), FEAT-07.SPEC-002, FEAT-08.SPEC-005, FEAT-22.SPEC-002, FEAT-28.SPEC-002 — "No WebSocket or collaborative-editing behavior is named"; Collaboration/concurrency: 16 of 18 entities with contention, resolution styles last-write-wins / reject-with-refresh / first-committed-wins; feasibility FEAT-03 spike (b): viewer count at which polling breaches one second |
| Recommended | Polling with TanStack Query (1-second `refetchInterval` on the client's slot list only while visible; 15-second polling on the Pro dashboard notices and banners, refetch-on-focus everywhere), with conflicts resolved server-side by database constraints and version-checked writes (reject-with-refresh) |
| Rationale | Landscape: "Meets the roughly one-second refresh without extra infrastructure" at no service cost. Contention correctness never depends on the transport — the Postgres exclusion constraint and compare-and-set updates decide winners (landscape Section 3 note). The slot endpoint is a small, per-Pro, index-bound query; polling pauses when the tab is hidden. Push is a documented upgrade (Supabase Realtime, already in the same project) triggered by the FEAT-03 spike result, not a launch dependency. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Supabase Realtime | Free 200 connections / 2M messages; Pro 500 connections / 5M messages | Low with Supabase | High | Medium — coupled to Supabase | Low | The FEAT-03 spike shows polling breaches ASMP-21 below the assumed ~50 concurrent viewers per Pro; broadcast a "slots changed" signal per Pro and let clients refetch |
| Ably | Free 200 connections; Standard $29/month plus $2.50/M messages | Low–Medium | Very high | Medium — proprietary protocol | Low | Push is needed and the database leaves Supabase, or presence/typing features are added |
| Server-Sent Events over own server | Server cost | Medium — needs a stateful runtime | Medium | None | Medium | Hosting moves to long-running containers (Render, Fly.io) and a vendor-free push channel is preferred |

### Analytics & Product Telemetry

| Field | Value |
|-------|-------|
| Context | Scale hints: ASMP-22 (a few hundred pros in year one), ASMP-21 (one-minute booking and one-second slot benchmarks to measure); privacy posture ASMP-23 limits client data flows; feasibility FEAT-05 ("funnel analytics … must respect the privacy posture") and Key Risk "Personal data persists in third-party stores after deletion" |
| Recommended | PostHog Cloud (US region) with person profiles only for Pros (keyed by internal `pro_account_id`), anonymous events for the client booking funnel, no client names/phones/emails in any event property, and session replay disabled on client-facing routes |
| Rationale | Landscape: "Event analytics, funnels, session replay, feature flags" with 1M events/month free — enough for year-one volume. Funnels measure the one-minute booking (ASMP-21) and go-live conversion; feature flags support staged rollout of Later-phase features (FEAT-26). Keeping client personal data out of events means client deletion (XBR-19) needs no third-party purge; Pro closure (XBR-20) calls PostHog's person-deletion API. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Plausible Analytics | Paid by pageviews; self-hosting option | Low | High | Low — open-source core | Low | Only aggregate traffic and simple goals are needed and cookie-free, no-personal-data analytics is preferred over funnels |
| Mixpanel | Free 1M events/month; Growth up to 20M | Low | Very high | Medium — proprietary | Low | The team already runs Mixpanel and needs its retention cohorts over feature flags |
| Amplitude | Free tier plus paid plans | Low | Very high | Medium — proprietary | Low | Product analytics maturity grows to experimentation programs Amplitude specializes in |

### Geo & Maps

Not activated — profile Section 3: "No specs reference location or mapping behavior; the studio address (FEAT-27.SPEC-001) and general area are stored and displayed as text fields, with no geocoding, distance or route behavior; BRIEF.md, ## Ecosystem & Integrations and assumptions-constraints.md name no mapping capability"

### Internationalization

| Field | Value |
|-------|-------|
| Context | Internationalization signal: "Currency and timezone are per-account settings, never hard-coded (ASMP-25)"; FEAT-07.SPEC-001, FEAT-15.SPEC-002, FEAT-22.SPEC-001, FEAT-27.SPEC-003, FEAT-27.SPEC-008; "no spec describes multi-language or locale-translation behavior"; feasibility FEAT-02 and FEAT-21 risks: DST boundaries shift local times by an hour |
| Recommended | date-fns v4 with date-fns-tz for IANA-timezone arithmetic (availability windows, reminders' 8am–9pm window, recurrence, cut-offs), Native Intl APIs (`Intl.NumberFormat`, `Intl.DateTimeFormat`) for all currency and date display; amounts stored as integer minor units with an ISO 4217 currency code; no translation catalog at launch |
| Rationale | Landscape: date-fns-tz for "Timezone-aware date arithmetic" and Native Intl for "Currency and timezone formatting with no dependency". Rules are stored as IANA zone plus local wall-clock time (never UTC offsets), which is feasibility FEAT-02's DST mitigation; all instants are stored as `timestamptz`. With no multi-language behavior specified (SC-10), a message-catalog library would be unused weight; strings live in one module per feature so a catalog can be introduced later. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Luxon | Open source | Low | High | Low | Low | The team prefers an immutable DateTime object model over date-fns' function style |
| next-intl | Open source | Low–Medium | High | Medium — Next.js-specific | Low | Multi-language UI enters scope (e.g. Canadian French) |
| i18next / react-i18next | Open source | Medium | High | Low | Medium | Translations must be shared with a future non-Next.js client (native app) |

### Calendar Sync

| Field | Value |
|-------|-------|
| Context | Product-mandated — "Pro's personal calendar (Google Calendar and Apple Calendar — both matter): two-way. Busy times there block Chairtime availability, and bookings made in Chairtime appear there." (BRIEF.md, ## Ecosystem & Integrations); ASMP-33; FEAT-04.SPEC-003, FEAT-03.SPEC-006; feasibility FEAT-04 Research-spike recommended (Apple/iCloud latency and authorization unknown); at most two connections per Pro, a few hundred pros (ASMP-22) |
| Recommended | Nylas (calendar-only usage on the Pro plan, $49/month plus ~$1.35–1.70 per connected account): one JSON API and webhook stream for Google Calendar and iCloud (CalDAV wrapped), used for busy/free reads and minimal booking-event writes |
| Rationale | Landscape: Nylas "Reads and writes Google, Outlook, Exchange and iCloud (CalDAV wrapped behind JSON API)" — it removes building and operating two protocols (Google watch channels plus CalDAV polling and reconciliation), which the landscape marks High effort and the feasibility assessment names as the most direct path to a double-booking via stale Apple busy time. At a few hundred pros with one or two connections each, per-account pricing is an order of magnitude below Cronofy's $819/month base. The FEAT-04 spike remains required for integration mechanics — measuring iCloud change-detection latency against "within a couple of minutes" and the app-specific-password step — and its outcome can swap to the direct-integration alternative; vendor selection is decided here. Only free/busy is read (never event titles), minimizing data held by the vendor. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Direct integration (Google Calendar API plus CalDAV for iCloud) | Free API usage; engineering and polling compute | High — two protocols, watch-channel renewal, reconciliation | High | None | Specialist — CalDAV and OAuth push channels | The FEAT-04 spike shows Nylas adds no iCloud latency advantage over direct CalDAV polling, or per-account fees exceed ~$1,000/month as pros grow |
| Cronofy | Base API from $819/month | Low–Medium | Very high | Medium — vendor holds calendar tokens | Low | Pro count grows into the thousands, making a flat enterprise contract cheaper than per-account pricing, or real-time Apple sync proves better on Cronofy in the spike |
| OneCal Unified Calendar API | Not retrieved | Low–Medium | Unknown — smaller vendor | Medium | Low | Its pricing and iCloud support, once verified, undercut Nylas at equal latency |

## 5. Project Structure

Suggested starting structure for the recommended stack (Next.js 15 App Router, ADR-001), adapted from the Next.js reference tree: route groups separate the four audiences (public booking, client self-service, Pro app, support), non-routing feature logic lives under `src/features/`, and server-only infrastructure (database, integrations, jobs, notifications, webhooks) lives under `src/server/`. Stage 3 folder slugs below are derived from the feature names; align them with the actual `FEAT-NN-*` folder names if they differ.

### Directory Tree

```
chairtime/
├── src/
│   ├── app/                                        # Next.js App Router — file-based routing
│   │   ├── layout.tsx                              # Root layout: html/body, fonts, QueryClientProvider, Toaster
│   │   ├── globals.css                             # Tailwind v4 @import + @theme tokens (CSS custom properties)
│   │   ├── error.tsx / not-found.tsx               # Root error boundary and 404
│   │   ├── (auth)/sign-in/                         # FEAT-29 sign-in and recovery (public, unauthenticated)
│   │   ├── (public)/[handle]/                      # FEAT-05 public booking page + flow, FEAT-20 join waitlist, FEAT-06 access-link request
│   │   │   ├── page.tsx                            # Landing & service list (route: /{handle})
│   │   │   ├── book/…                              # Slot selection, details, checkout, confirmation
│   │   │   └── waitlist/[serviceId]/page.tsx
│   │   ├── (links)/                                # Tokenized one-tap landings (manage link, opt-out, reminder reply, pay request)
│   │   │   ├── m/[token]/route.ts                  # Manage-link redemption → client session → redirect
│   │   │   ├── o/[token]/page.tsx                  # FEAT-14 opt-out landing
│   │   │   ├── r/[token]/page.tsx                  # FEAT-08 reminder reply acknowledgment
│   │   │   └── pay/[token]/page.tsx                # FEAT-07 deposit payment for Pro-created/recurring requests
│   │   ├── (client)/c/                             # Client self-service (access-link session, one client × one Pro)
│   │   │   ├── layout.tsx                          # Client shell; requires client session
│   │   │   ├── bookings/…                          # FEAT-06/10/22/23 list, detail, cancel, reschedule, balance, tip
│   │   │   ├── waitlists/ · recurring/ · preferences/
│   │   ├── (pro)/app/                              # Pro app (Pro session required)
│   │   │   ├── layout.tsx                          # App shell: bottom nav (mobile) / sidebar (desktop)
│   │   │   ├── schedule/ · attention/ · bookings/  # FEAT-12, FEAT-30, FEAT-11, FEAT-16
│   │   │   ├── clients/ · services/ · availability/ · time-blocks/ · recurring/
│   │   │   ├── insights/ · payouts/ · billing/ · setup/ · help/
│   │   │   └── settings/                           # profile, region, pause, notifications, policy, calendar, account
│   │   ├── (support)/support/                      # FEAT-19 read-only support console (Platform Operator role)
│   │   └── api/
│   │       ├── slots/[handle]/route.ts             # GET polled slot list (1-second cadence)
│   │       ├── notices/route.ts                    # GET polled Pro notices/banners
│   │       ├── inngest/route.ts                    # Inngest function endpoint (ADR-012)
│   │       ├── auth/[...all]/route.ts              # Better Auth handler (ADR-025)
│   │       └── webhooks/                           # stripe/ · twilio/ · postmark/ · nylas/ (ADR-022)
│   │
│   ├── features/                                   # Feature business logic (NOT routing), one folder per FEAT-NN
│   │   ├── feat-01-service-pricing-management/
│   │   │   ├── components/                         # ServiceForm.tsx, ServiceCard.tsx
│   │   │   ├── actions/                            # Server actions + Automation specs (archive-impact-check.ts)
│   │   │   ├── rules/                              # Logic/Rule specs (deposit-rule-validation.ts)
│   │   │   ├── queries.ts                          # Read models for this feature's screens
│   │   │   └── schemas.ts                          # Zod input schemas shared client/server
│   │   ├── feat-03-real-time-slot-availability-engine/
│   │   │   ├── rules/ · actions/ · jobs/           # jobs/ = Inngest functions owned by this feature
│   │   └── … feat-02 … feat-30 (same shape)
│   │
│   ├── server/                                     # Server-only infrastructure ('server-only' import guard)
│   │   ├── db/
│   │   │   ├── client.ts                           # Drizzle instance (postgres.js via Supabase pooler)
│   │   │   ├── schema/                             # One file per entity group (see database-schema.md)
│   │   │   ├── tenant.ts                           # withProScope(proAccountId, tx) — sets RLS context
│   │   │   └── outbox.ts                           # Transactional outbox writer (ADR-022)
│   │   ├── auth/                                   # Better Auth config, client access-link sessions, support role
│   │   ├── integrations/                           # One client module per external service
│   │   │   ├── stripe.ts · twilio.ts · postmark.ts · nylas.ts · storage.ts
│   │   ├── notifications/                          # Delivery per channel + templates
│   │   │   ├── sms.ts · email.ts · in-app.ts · whatsapp.ts
│   │   │   └── templates/                          # One template module per Notification spec
│   │   ├── jobs/                                   # Inngest client, event catalog, cron registry
│   │   ├── webhooks/                               # Verify signature → dedupe → persist inbound event → emit
│   │   └── observability.ts                        # Sentry helpers, PII scrubbing
│   │
│   ├── shared/                                     # Cross-feature client+server code
│   │   ├── components/ui/                          # shadcn/ui copy-in components (Button, Dialog, Sheet, …)
│   │   ├── components/                             # AppShell, BottomNav, EmptyState, OfflineBanner, Skeletons
│   │   ├── hooks/                                  # useOnlineStatus, usePolling
│   │   └── lib/                                    # money.ts, time.ts (date-fns-tz), ids.ts, errors.ts, query-client.ts
│   │
│   └── db/seed.ts                                  # Local/staging seed data
│
├── drizzle/                                        # Generated + hand-edited SQL migrations (ADR-019)
├── drizzle.config.ts
├── tests/                                          # e2e (Playwright) and concurrency tests on the booking core
├── public/
├── next.config.ts
├── instrumentation.ts                              # Sentry server init
├── package.json · pnpm-lock.yaml · tsconfig.json
└── .github/workflows/                              # CI (ADR-029)
```

### Feature-to-Directory Mapping

| Feature | Stage 3 Folder | Source Directory | Notes |
|---------|---------------|-----------------|-------|
| FEAT-01 (Service & Pricing Management) | FEAT-01-service-pricing-management/ | `src/features/feat-01-service-pricing-management/`; routes `src/app/(pro)/app/services/` | Price/deposit snapshot copied inside the booking transaction (called from FEAT-05/FEAT-30) |
| FEAT-02 (Availability & Working Hours Setup) | FEAT-02-availability-working-hours-setup/ | `src/features/feat-02-availability-working-hours-setup/`; routes `src/app/(pro)/app/availability/` | Versioned rules; conflict flagging as an Inngest job |
| FEAT-03 (Real-Time Slot Availability Engine) | FEAT-03-real-time-slot-availability-engine/ | `src/features/feat-03-real-time-slot-availability-engine/`; endpoint `src/app/api/slots/[handle]/route.ts` | No screens of its own; owns hold creation, exclusion-constraint SQL, lazy expiry |
| FEAT-04 (Two-Way Calendar Sync) | FEAT-04-two-way-calendar-sync/ | `src/features/feat-04-two-way-calendar-sync/`; routes `src/app/(pro)/app/settings/calendar/` | Uses `src/server/integrations/nylas.ts`; webhooks `src/app/api/webhooks/nylas/` |
| FEAT-05 (Public Booking Page & Booking Flow) | FEAT-05-public-booking-page-booking-flow/ | `src/features/feat-05-public-booking-page-booking-flow/`; routes `src/app/(public)/[handle]/` | Server-rendered; slot list polled from `/api/slots/[handle]` |
| FEAT-06 (Client Booking Identity) | FEAT-06-client-booking-identity/ | `src/features/feat-06-client-booking-identity/`; routes `src/app/(public)/[handle]/access/`, `src/app/(client)/c/`, `src/app/(links)/m/` | Access-link issuance/redemption in `src/server/auth/client-session.ts` |
| FEAT-07 (Deposit Payment at Booking) | FEAT-07-deposit-payment-at-booking/ | `src/features/feat-07-deposit-payment-at-booking/`; routes `src/app/(public)/[handle]/book/checkout/`, `src/app/(links)/pay/` | Stripe Payment Element; capture handled from Stripe webhook |
| FEAT-08 (Automated Booking Messaging) | FEAT-08-automated-booking-messaging/ | `src/features/feat-08-automated-booking-messaging/`; `src/server/notifications/`; route `src/app/(links)/r/` | Owns channel selection, reminder scheduling, retry-then-fallback |
| FEAT-09 (Cancellation & No-Show Policy Engine) | FEAT-09-cancellation-no-show-policy-engine/ | `src/features/feat-09-cancellation-no-show-policy-engine/`; routes `src/app/(pro)/app/settings/policy/` | Refund execution + indefinite idempotent retry as an Inngest function |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | FEAT-10-client-initiated-cancel-reschedule/ | `src/features/feat-10-client-initiated-cancel-reschedule/`; routes `src/app/(client)/c/bookings/[bookingId]/cancel/`, `…/reschedule/` | Commit writes outbox events for side effects |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | FEAT-11-no-show-marking-deposit-forfeiture/ | `src/features/feat-11-no-show-marking-deposit-forfeiture/`; routes `src/app/(pro)/app/bookings/[bookingId]/no-show/` | Undo window checked at request time |
| FEAT-12 (Pro Daily Schedule Dashboard) | FEAT-12-pro-daily-schedule-dashboard/ | `src/features/feat-12-pro-daily-schedule-dashboard/`; routes `src/app/(pro)/app/schedule/`, `src/app/(pro)/app/attention/` | Offline-persisted queries; auto-completion sweep cron |
| FEAT-13 (Client Record Management) | FEAT-13-client-record-management/ | `src/features/feat-13-client-record-management/`; routes `src/app/(pro)/app/clients/[clientId]/` | Deletion cascade + de-identification job |
| FEAT-14 (Messaging Consent Management) | FEAT-14-messaging-consent-management/ | `src/features/feat-14-messaging-consent-management/`; routes `src/app/(client)/c/preferences/messaging/`, `src/app/(links)/o/` | Inbound STOP from Twilio webhook |
| FEAT-15 (Pro Onboarding & Setup Wizard) | FEAT-15-pro-onboarding-setup-wizard/ | `src/features/feat-15-pro-onboarding-setup-wizard/`; routes `src/app/(pro)/app/setup/` | Wizard shell embeds other features' form components |
| FEAT-16 (Booking & Payment Activity Record) | FEAT-16-booking-payment-activity-record/ | `src/features/feat-16-booking-payment-activity-record/`; routes `src/app/(pro)/app/bookings/[bookingId]/activity/` | Append-only writer used by all features; dispute webhook |
| FEAT-17 (Manual Time Blocking) | FEAT-17-manual-time-blocking/ | `src/features/feat-17-manual-time-blocking/`; routes `src/app/(pro)/app/time-blocks/` | Blocks share the exclusion-constraint scheme with holds |
| FEAT-18 (Pro Subscription Billing & Account Management) | FEAT-18-pro-subscription-billing-account-management/ | `src/features/feat-18-pro-subscription-billing-account-management/`; routes `src/app/(pro)/app/billing/` | Grace clock driven by Stripe Billing events |
| FEAT-19 (Platform Support Read-Only Access) | FEAT-19-platform-support-read-only-access/ | `src/features/feat-19-platform-support-read-only-access/`; routes `src/app/(support)/support/`, `src/app/(pro)/app/settings/account/support-log/` | Read-only DB role connection `src/server/db/support-client.ts` |
| FEAT-20 (Waitlist for Cancelled Slots) | FEAT-20-waitlist-for-cancelled-slots/ | `src/features/feat-20-waitlist-for-cancelled-slots/`; routes `src/app/(public)/[handle]/waitlist/`, `src/app/(client)/c/waitlists/` | Priority window modeled as a waitlist-scoped hold (FEAT-03) |
| FEAT-21 (Recurring/Standing Appointments) | FEAT-21-recurring-standing-appointments/ | `src/features/feat-21-recurring-standing-appointments/`; routes `src/app/(client)/c/recurring/`, `src/app/(pro)/app/recurring/` | Rolling occurrence generation cron |
| FEAT-22 (In-App Balance Payment) | FEAT-22-in-app-balance-payment/ | `src/features/feat-22-in-app-balance-payment/`; routes `src/app/(client)/c/bookings/[bookingId]/pay-balance/` | Reuses FEAT-07 webhook/idempotency machinery |
| FEAT-23 (Tipping at Checkout) | FEAT-23-tipping-at-checkout/ | `src/features/feat-23-tipping-at-checkout/`; routes `src/app/(client)/c/bookings/[bookingId]/pay-balance/tip/` | Tip is a line on the FEAT-22 charge |
| FEAT-24 (Client List Search & Filter) | FEAT-24-client-list-search-filter/ | `src/features/feat-24-client-list-search-filter/`; routes `src/app/(pro)/app/clients/` | pg_trgm query + client-side filter of cached list |
| FEAT-25 (Booking & Revenue Insights) | FEAT-25-booking-revenue-insights/ | `src/features/feat-25-booking-revenue-insights/`; routes `src/app/(pro)/app/insights/` | Aggregates maintained by Inngest event handlers |
| FEAT-26 (WhatsApp Reminders) | FEAT-26-whatsapp-reminders/ | `src/features/feat-26-whatsapp-reminders/`; routes `src/app/(client)/c/preferences/whatsapp/`; `src/server/notifications/whatsapp.ts` | Later phase; behind a PostHog feature flag |
| FEAT-27 (Pro Profile & Booking Page Settings) | FEAT-27-pro-profile-booking-page-settings/ | `src/features/feat-27-pro-profile-booking-page-settings/`; routes `src/app/(pro)/app/settings/`, `src/app/(pro)/app/help/` | Photo upload via `src/server/integrations/storage.ts` |
| FEAT-28 (Payout Account Connection & Payout Visibility) | FEAT-28-payout-account-connection-payout-visibility/ | `src/features/feat-28-payout-account-connection-payout-visibility/`; routes `src/app/(pro)/app/payouts/` | Stripe-hosted onboarding; money list mirrored from webhooks |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | FEAT-29-pro-sign-in-account-lifecycle/ | `src/features/feat-29-pro-sign-in-account-lifecycle/`; routes `src/app/(auth)/sign-in/`, `src/app/(pro)/app/settings/account/` | Better Auth configuration in `src/server/auth/` |
| FEAT-30 (Pro Booking Management) | FEAT-30-pro-booking-management/ | `src/features/feat-30-pro-booking-management/`; routes `src/app/(pro)/app/bookings/` | Bulk cancel fan-out via Inngest steps |

### Spec-Type-to-Location Mapping

| Spec Type | File Location Pattern | Example |
|-----------|----------------------|---------|
| Screen | `src/app/({group})/{route}/page.tsx` + `src/features/feat-{NN}-{slug}/components/` | FEAT-12.SPEC-001 → `src/app/(pro)/app/schedule/page.tsx` + `src/features/feat-12-pro-daily-schedule-dashboard/components/ScheduleList.tsx` |
| Automation | Synchronous commits: `src/features/feat-{NN}-{slug}/actions/{action}.ts`; scheduled/timed/retried: `src/features/feat-{NN}-{slug}/jobs/{job}.ts` (Inngest functions registered in `src/server/jobs/registry.ts`) | FEAT-10.SPEC-004 → `src/features/feat-10-client-initiated-cancel-reschedule/actions/commit-booking-update.ts`; FEAT-08.SPEC-007 → `src/features/feat-08-automated-booking-messaging/jobs/schedule-reminder.ts` |
| Logic/Rule | `src/features/feat-{NN}-{slug}/rules/{rule}.ts` (pure functions, unit-tested); rules used by several features in `src/shared/lib/` | FEAT-09.SPEC-003 → `src/features/feat-09-cancellation-no-show-policy-engine/rules/deposit-outcome.ts` |
| Integration | `src/server/integrations/{service}.ts` (one client module per external service) + inbound handlers in `src/server/webhooks/{service}.ts` behind `src/app/api/webhooks/{service}/route.ts` | FEAT-07.SPEC-005 → `src/server/integrations/stripe.ts` + `src/server/webhooks/stripe.ts` |
| Notification | Channel delivery in `src/server/notifications/{channel}.ts`; one template module per spec in `src/server/notifications/templates/{spec-slug}.ts`; the feature's trigger in its `jobs/` | FEAT-08.SPEC-002 → `src/server/notifications/templates/appointment-reminder.ts`, sent via `src/server/notifications/sms.ts` |

## 6. Data Layer Design

See `.n2b/architecture/database-schema.md` for the complete schema design.

**Migration strategy:** migration-based. With 18 entities, 56 relationships and correctness enforced by database objects that schema diffing does not fully own (exclusion constraints with `btree_gist`, RLS policies, append-only grants, a read-only support role), every change is generated by `drizzle-kit generate` into reviewed, checked-in SQL files (hand-edited where needed) and applied by `drizzle-kit migrate` in CI to staging then production — never `push` against shared environments (ADR-019).

## 7. API & Routing Architecture

Four audiences, four route groups: public booking pages at the root (`/{handle}`, the link a Pro puts in their Instagram bio), client self-service under `/c`, the Pro app under `/app`, and the support console under `/support`. Tokenized one-tap links from messages use short root prefixes (`/m`, `/o`, `/r`, `/pay`). Booking-link handles are validated against a reserved-word list (`app`, `c`, `api`, `sign-in`, `support`, `m`, `o`, `r`, `pay`, …) as part of FEAT-27.SPEC-007 (ADR-020).

### Route Map

| Screen Spec | URL Path | Parameters | Data Requirements |
|-------------|----------|------------|-------------------|
| FEAT-01.SPEC-001 (Service List) | /app/services | -- | Pro's services (active/archived), prices, deposits |
| FEAT-01.SPEC-002 (Add Service) | /app/services/new | -- | Account currency, deposit rule limits (`minimum-chargeable-deposit`) |
| FEAT-01.SPEC-003 (Edit Service) | /app/services/[serviceId]/edit | serviceId | Service record + version, upcoming bookings count |
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | /app/availability | -- | Current Availability Rule version, Pro timezone |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | /app/availability/buffers | -- | Services with buffer overrides |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | /app/settings/calendar/connect | ?provider=google\|apple | Existing connections, provider auth URL (Nylas hosted auth) |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | /app/settings/calendar | -- | Calendar Connections with health status, last sync |
| FEAT-05.SPEC-001 (Public Booking Page (Landing & Service List)) | /[handle] | handle | Public profile fields, photo URL, active services, availability gate state (paused/closed/forwarded) |
| FEAT-05.SPEC-002 (Slot Selection) | /[handle]/book/[serviceId] | handle, serviceId, ?date= | Polled slot list from `/api/slots/[handle]?service=&from=&to=` |
| FEAT-05.SPEC-003 (Client Details & Consent) | /[handle]/book/[serviceId]/details | handle, serviceId, ?start= | Selected slot, consent wording version |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | /[handle]/book/checkout/[holdId] | handle, holdId | Hold (expires_at), current Cancellation Policy version, deposit amount |
| FEAT-05.SPEC-005 (Booking Confirmation) | /[handle]/book/confirmed/[bookingRef] | handle, bookingRef | Booking summary, .ics link, manage link |
| FEAT-06.SPEC-001 (Access Link Request) | /[handle]/access | handle | -- (phone/email entry; rate-limited action) |
| FEAT-06.SPEC-003 (My Bookings List) | /c/bookings | -- | Client-session-scoped upcoming/past bookings with this Pro |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | /c/bookings/[bookingId] | bookingId | Booking, deposit status, cut-off time, balance due |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | /c/preferences | -- | Client contact, email preference, consent summary |
| FEAT-07.SPEC-001 (Deposit Payment) | /[handle]/book/checkout/[holdId]/pay (booking flow); /pay/[token] (Pro-created and recurring deposit requests) | holdId or token | Stripe PaymentIntent client secret, amount in account currency, hold countdown |
| FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) | /r/[token] | token | Reply intent, booking current state |
| FEAT-09.SPEC-001 (Cancellation Policy Setup) | /app/settings/policy | -- | Current policy version, version history |
| FEAT-10.SPEC-001 (Cancel Booking) | /c/bookings/[bookingId]/cancel | bookingId | Bound policy version, live window countdown, deposit outcome preview |
| FEAT-10.SPEC-002 (Reschedule -- Select New Time) | /c/bookings/[bookingId]/reschedule | bookingId, ?date= | Polled slot list for the booking's service |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | /c/bookings/[bookingId]/reschedule/confirm | bookingId, ?holdId= | Hold, deposit outcome (kept + new deposit if late) |
| FEAT-11.SPEC-001 (No-Show Mark & Undo Prompt) | /app/bookings/[bookingId]/no-show | bookingId | Booking status/version, marking window, undo deadline |
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | /app/schedule | ?date= | Bookings for the day/week with paid badge, balance due, sync-confidence flag |
| FEAT-12.SPEC-002 (Attention List) | /app/attention | -- | Aggregated attention items |
| FEAT-12.SPEC-003 (Past Bookings Browse) | /app/schedule/past | ?before= (cursor) | Paginated past bookings by date |
| FEAT-13.SPEC-001 (Client Record Detail) | /app/clients/[clientId] | clientId | Client, booking history, consent state, private note |
| FEAT-13.SPEC-002 (Client Contact Edit) | /app/clients/[clientId]/edit | clientId | Client record + version |
| FEAT-13.SPEC-003 (Client Deletion Confirmation) | /app/clients/[clientId]/delete | clientId | Deletion eligibility (upcoming bookings) |
| FEAT-14.SPEC-001 (Consent & Preferences) | /c/preferences/messaging | -- | Messaging Consent per channel with wording and timestamps |
| FEAT-14.SPEC-002 (Opt-Out Link Landing) | /o/[token] | token | Consent record resolved from token |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance) | /app/setup/[step] | step | Setup progress record, step completion states |
| FEAT-15.SPEC-002 (Cancellation Policy Default & First-Version Setup Step) | /app/setup/policy | -- | `cancellation-window-default-hours`, account currency |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | /app/setup/go-live | -- | Go-live evaluation, booking link, payout/subscription status |
| FEAT-16.SPEC-001 (Booking Activity Timeline) | /app/bookings/[bookingId]/activity | bookingId | Activity Events for the booking, dispute overlay; summary download action |
| FEAT-17.SPEC-001 (Create/Edit Time Block) | /app/time-blocks/new; /app/time-blocks/[blockId]/edit | blockId (edit) | Block record, recurrence pattern, Pro timezone |
| FEAT-17.SPEC-002 (Manage Time Blocks) | /app/time-blocks | -- | Upcoming one-off and recurring blocks |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | /app/time-blocks/[blockId]/conflicts | blockId | Conflicting confirmed bookings with resolution options |
| FEAT-18.SPEC-001 (Subscribe Screen) | /app/billing/subscribe | -- | Plan price, Stripe Checkout/Payment Element session |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | /app/billing | -- | Subscription status, next renewal, grace state, invoices |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | /support | ?q= | Pro lookup results (masked), reason/ticket entry |
| FEAT-19.SPEC-003 (Support Access Log) | /app/settings/account/support-log | -- | Support-view events for this Pro account |
| FEAT-20.SPEC-001 (Join Waitlist) | /[handle]/waitlist/[serviceId] | handle, serviceId | Service, requested window options, entry limits |
| FEAT-20.SPEC-002 (My Waitlists) | /c/waitlists | -- | Client's active waitlist entries with this Pro |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | /c/recurring/new | ?fromBooking= | Source booking, interval options (1–12 weeks), horizon |
| FEAT-21.SPEC-002 (My Recurring Series) | /c/recurring | -- | Client's series and upcoming occurrences |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | /app/recurring | ?clientId= | Pro's series, occurrence and deposit states |
| FEAT-22.SPEC-001 (Balance Payment) | /c/bookings/[bookingId]/pay-balance | bookingId | Balance amount (price − deposit), Stripe PaymentIntent |
| FEAT-23.SPEC-001 (Tip Selection) | /c/bookings/[bookingId]/pay-balance/tip | bookingId | Balance, tip presets (never defaulted) |
| FEAT-24.SPEC-001 (Client Search & Filter) | /app/clients | ?q=, ?filter= | Per-Pro client list (cached for offline filtering) |
| FEAT-25.SPEC-001 (Insights Summary Screen) | /app/insights | ?period= | Period aggregates, "not enough data" state |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | /c/preferences/whatsapp | -- | Channel preference, WhatsApp consent |
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | /app/settings/profile | -- | Profile fields, photo URL, studio address |
| FEAT-27.SPEC-002 (Booking Link Rename) | /app/settings/profile/link | -- | Current handle, reserved/forwarded names |
| FEAT-27.SPEC-003 (Timezone & Currency Settings) | /app/settings/region | -- | Timezone, currency, currency-lock state |
| FEAT-27.SPEC-004 (Pause Bookings) | /app/settings/pause | -- | Pause state, end date, system-imposed pause |
| FEAT-27.SPEC-005 (Notification Preferences) | /app/settings/notifications | -- | Pro notification preferences |
| FEAT-27.SPEC-006 (Help Request) | /app/help | -- | -- (help request form) |
| FEAT-28.SPEC-001 (Payout Account Connection) | /app/payouts/connect | -- | Payout account status, Stripe onboarding link |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | /app/payouts | ?period= | Money list (deposits, refunds, fees, payouts), net per period, status banner |
| FEAT-29.SPEC-001 (Sign-In Screen) | /sign-in | ?next= | -- (email/phone entry, code entry) |
| FEAT-29.SPEC-002 (Account Recovery Screen) | /sign-in/recover | -- | -- (remaining-contact recovery) |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | /app/settings/account | -- | Contacts, signed-in devices/sessions |
| FEAT-29.SPEC-004 (Data Export Screen) | /app/settings/account/export | -- | Export job status, signed download URL |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | /app/settings/account/close | -- | Closure prerequisites, cooling-off state (reopen reachable from sign-in when closed) |
| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) | /app/bookings/[bookingId]/cancel | bookingId | Booking + version, refund preview |
| FEAT-30.SPEC-002 (Reschedule Booking (Pro-Initiated)) | /app/bookings/[bookingId]/reschedule | bookingId | Slot list with Pro-only exceptions |
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | /app/bookings/[bookingId]/refund | bookingId | Deposit Transaction state, refundable amount |
| FEAT-30.SPEC-004 (Book Client In) | /app/bookings/new | ?clientId=, ?start= | Clients, services, slot list, deposit-request hold options |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | /app/bookings/bulk-cancel | ?date= | Bookings in range with per-booking refund previews |

### API Endpoint Convention

Mutations from screens are Next.js **server actions** colocated in each feature's `actions/` folder; they accept a Zod-validated input, run inside `withProScope` (or the client-session scope), and return a discriminated result `{ ok: true, data } | { ok: false, code, message, refresh?: true }` — `refresh: true` signals reject-with-refresh so the client refetches the affected queries. JSON **route handlers** exist only where a URL is needed: polled reads (`GET /api/slots/{handle}?service={id}&from={iso}&to={iso}`, `GET /api/notices`), inbound webhooks (`POST /api/webhooks/{stripe|twilio|postmark|nylas}`), the Inngest endpoint (`/api/inngest`), and the auth handler (`/api/auth/*`). Resource names are plural kebab-case nouns, IDs are opaque (UUIDv7 internally, short public refs for anything placed in a URL a client sees), times are ISO 8601 UTC with the Pro's IANA zone returned alongside, and money is `{ amountMinor: number, currency: "USD" }`. Errors use HTTP status plus `{ code, message }`; webhooks return 2xx only after the inbound event is durably persisted.

### Data Fetching Strategy

| Route Type | Fetching Approach | Rationale |
|-----------|-------------------|-----------|
| Public booking pages (`/[handle]`, flow steps) | Server components render the profile and service list per request (dynamic, no CDN cache of data); the slot list hydrates into TanStack Query polling `/api/slots` every 1 second while visible | ASMP-21 one-second slots inside the Instagram in-app browser: first paint is server HTML with a small client bundle; FEAT-27 changes appear immediately |
| Pro list pages (schedule, clients, services, money list) | Server component prefetch into TanStack Query (`HydrationBoundary`), then client-side cache with 15-second polling for notices/banners and refetch on focus; persisted to IndexedDB | ASMP-27 offline read of last-loaded schedule/client list/money list; Real-time signal for banners (FEAT-28.SPEC-002, FEAT-08.SPEC-005) |
| Detail pages (booking, client, activity) | Server component render + TanStack Query for fields that change under contention (booking status/version) | Reject-with-refresh on High-contention Booking needs a fresh version before any action |
| Form submissions | Server actions with `useActionState`; on `refresh: true` the client invalidates affected queries and shows the current state; actions that book/pay/cancel/refund/no-show are disabled when offline | Collaboration/concurrency signal (16 of 18 entities); ASMP-27 "needs a live connection and says so plainly" |
| Payments | Stripe Payment Element on the page; the booking flips to Confirmed only from the Stripe webhook (`payment_intent.succeeded`), while the page polls booking status | FEAT-07.SPEC-002/SPEC-004 exactly-once outcome; never trust the client redirect |

### Navigation Model

- **Client (public + `/c`)**: linear, step-based flows with no global nav — booking-page landing → service → slot → details/consent → policy & pay → confirmation. After an access link or manage link is redeemed, a minimal header links My Bookings, My Waitlists, Recurring and Preferences for that one Pro. Every flow step is URL-addressable so the in-app browser's back button works.
- **Pro (`/app`)**: mobile bottom navigation with five destinations — Schedule (home, `/app/schedule`), Clients, Bookings actions (Book client in), Money (`/app/payouts`), More (services, availability, time blocks, recurring, insights, billing, settings, help); a sidebar with the same items on desktop. The Attention List is reached from a badge on Schedule. Until go-live, `/app` redirects to `/app/setup/[step]` (FEAT-15).
- **Support (`/support`)**: lookup → one Pro account at a time at `/support/accounts/[proAccountId]/…`, rendering the Pro's read screens in read-only mode; opening a new lookup ends the prior session (FEAT-19.SPEC-004).

## 8. Integration Architecture

| Service | Purpose | Data Exchanged | Direction | Rate/Quota Notes | Failure-Mode Handling | Sandbox/Test Path |
|---------|---------|----------------|-----------|------------------|----------------------|-------------------|
| Stripe Connect (payments) | Client card charges and refunds for deposits, balances and tips; payout routing to connected accounts; card-issuer disputes (External Touchpoints: payment processing rows; FEAT-07.SPEC-005, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-22.SPEC-005, FEAT-30.SPEC-011; selected in Section 4, Payments & Billing) | Outbound: amounts, currency, connected-account ID, idempotency keys, booking reference metadata (no client PII beyond what Stripe's Payment Element collects directly). Inbound: `payment_intent.*`, `charge.refunded`, `refund.*`, `charge.dispute.*` events | Bidirectional | API rate limit ~100 req/s live (far above need); card 2.9% + 30¢ on the Pro's account | Per FEAT-07.SPEC-005 Degradation Behavior: slow → "Processing payment, do not close this page"; down → Pay disabled while the hold counts down; unknown outcome creates no Deposit Transaction until the webhook arrives. Refunds per FEAT-09.SPEC-005/FEAT-30.SPEC-011: "Refund in Progress" + indefinite idempotent Inngest retry every `refund-retry-interval-hours`. Disputes per FEAT-16.SPEC-003: late notices recorded with original timestamps | Stripe test mode keys + test cards (incl. 3-D Secure cards); Stripe CLI `stripe listen` forwards webhooks locally |
| Stripe Connect (payout accounts) | Hosted identity/bank onboarding and account/payout status (FEAT-28.SPEC-006; FEAT-15 getting-paid step) | Outbound: account-link creation. Inbound: `account.updated`, `payout.*`, balance/fee reporting. No bank or identity data stored by Chairtime | Bidirectional | Connect account fee ~$2 per monthly active account (platform-handled pricing) | Per FEAT-28.SPEC-006 Degradation Behavior: dashboard shows last-known data with Retry; resolution flow shows capability-down message leaving Action Required unchanged; out-of-order status resolved by event time | Test-mode connected accounts with test verification data |
| Stripe Billing | Pro monthly subscription: subscribe, update payment method, cancel (incl. account closure), renewal outcomes (FEAT-18.SPEC-006) | Outbound: customer, price, subscription actions. Inbound: `invoice.paid`, `invoice.payment_failed`, `customer.subscription.*` | Bidirectional | Included in Stripe Billing pricing (per-invoice percentage) | Per FEAT-18.SPEC-006 Degradation Behavior: processor down disables subscribe/update/cancel with plain messages; down during renewal is inconclusive, not a failure; grace clock keys off Stripe events only | Stripe test clocks to simulate renewals and grace expiry |
| Twilio Programmable Messaging | Transactional SMS for every product text incl. access links, sign-in codes, STOP handling (FEAT-08.SPEC-012; External Touchpoints: transactional text messaging; selected in Section 4, Email & Messaging Delivery) | Outbound: client/Pro phone number and message body (booking time, Pro name, studio address, links). Inbound: delivery-status callbacks, inbound STOP/START/replies | Bidirectional | ~$0.0083/segment plus carrier fees; 10DLC campaign throughput limits; per-phone rate limit on access-link and sign-in-code requests (toll-fraud mitigation) | Per FEAT-08.SPEC-012 Degradation Behavior: every send asynchronous; silence past timeout recorded as Failed and handed to FEAT-08.SPEC-009 retry-once-then-email; in-app copy never depends on the provider. Duplicate/out-of-order status resolved by provider event time | Twilio test credentials + magic numbers; local webhook via tunnel; staging uses a separate Messaging Service |
| Twilio WhatsApp (Later) | WhatsApp confirmations, reminders, change notices (FEAT-26.SPEC-002; External Touchpoints: WhatsApp row) | Outbound: pre-approved template messages. Inbound: delivery status | Bidirectional | ~$0.005/message plus Meta template fees from $0.0034 | Per FEAT-26.SPEC-002 Degradation Behavior: timeout converts silence to Failed and FEAT-26.SPEC-003 falls back to text (with consent) or email | Twilio WhatsApp Sandbox |
| Postmark | Transactional email fallback and email-channel notices, sign-in codes by email (FEAT-08.SPEC-013; External Touchpoints: transactional email) | Outbound: recipient email and message content. Inbound: delivery, bounce, spam-complaint webhooks | Bidirectional | From $15/month for 10,000 emails | Per FEAT-08.SPEC-013 Degradation Behavior: asynchronous; failure recorded and surfaced as a Pro attention item; in-app copy unaffected | Postmark test API token (`POSTMARK_API_TEST`) + a sandbox server |
| Nylas | Two-way calendar sync with Google Calendar and iCloud: busy/free reads, booking event write/move/remove (FEAT-04.SPEC-003, FEAT-03.SPEC-006; External Touchpoints: calendar sync; selected in Section 4, Calendar Sync) | Outbound: minimal booking events (time, "Chairtime booking" label, no client PII beyond first name if the Pro opts in). Inbound: free/busy periods only (never event titles), change webhooks, grant-expired notices | Bidirectional | Per connected account (~$1.35–1.70/month); API rate limits per grant | Per FEAT-04.SPEC-003 Degradation Behavior: slow keeps last synced busy periods; down falls back to Chairtime-only availability with the Pro-only confidence banner (FEAT-03.SPEC-006); failed writes queue for Inngest retry; reconciliation keeps status at Syncing until drift resolves | Nylas sandbox application with test Google and iCloud accounts (FEAT-04 spike) |
| Supabase Storage | Pro profile photos; private data exports and dispute summaries (FEAT-27.SPEC-012; External Touchpoints: file storage; selected in Section 4, File & Object Storage) | Outbound: re-encoded photo (EXIF stripped), export files. Inbound: served image / signed download URL | Bidirectional | 100 GB included on Pro plan | Per FEAT-27.SPEC-012 Degradation Behavior: slow/down leaves other profile fields savable; booking page shows the no-photo layout without error; export shows retry | Local Supabase stack (`supabase start`) or a staging project |
| Better Auth (in-app) | Pro OTP sign-in, sessions, device list (selected in Section 11) | No external runtime exchange — runs inside the app on the primary database; codes are delivered through the Twilio and Postmark rows above | -- (internal) | N/A | Code delivery failure follows FEAT-08.SPEC-009 retry/fallback; sign-in screen shows in-place "sending" indicator | Local database |
| Inngest | Durable jobs, delays, cron, retries (selected in Section 4, Background Jobs & Scheduling) | Outbound: event payloads (IDs only, no client PII). Inbound: function invocations to `/api/inngest` | Bidirectional | Free 50k executions/month; Pro from $99/month | Events are written first to the Postgres outbox (ADR-022), so an Inngest outage delays side effects but loses none; a relay cron re-sends undelivered outbox rows | Inngest Dev Server (`npx inngest-cli dev`) locally; branch environments per preview |
| Sentry | Error tracking and tracing (selected in Section 13) | Outbound: stack traces, request metadata with PII scrubbed | Outbound | Team tier event quota | SDK buffers and drops on outage; never blocks a request | Separate DSN per environment |
| PostHog | Product analytics and feature flags (selected in Section 4, Analytics & Product Telemetry) | Outbound: product events keyed by internal IDs; no client names/phones/emails | Outbound | Free 1M events/month | Client SDK fails silent; feature flags default to off when unreachable | Separate project per environment |

**Inbound webhook ingestion pattern (ADR-022).** Every webhook route verifies the provider signature, inserts the raw event into an `inbound_events` table with a unique `(provider, provider_event_id)` constraint (duplicates become no-ops), returns 2xx, and emits an Inngest event. Handlers apply state changes only if the event's provider timestamp is newer than the last applied event for that object (event-time ordering, per every Integration spec's Edge Cases). Outbound side effects of booking commits are written to an `outbox` table in the same transaction as the state change and relayed to Inngest, so a commit never loses its refund, calendar mirror, activity event, freed-slot hand-off or notice.

## 9. Design System Implementation Plan

**Design system posture:** None (`design_system_source: none`) — this blueprint is design-agnostic; the downstream builder owns visual design, honoring the design preferences recorded in the brief's Constraints. Styling and component decisions below are driven by product needs alone (70 Screen specs, the ASMP-28 accessibility baseline, mobile-first in the Instagram in-app browser).

**Styling-system implementation approach:** Tailwind CSS v4 (ADR-005). All visual values the builder chooses — colors, type scale, spacing, radii — are defined once as CSS custom properties in `src/app/globals.css` under `@theme`, and components reference only semantic tokens (e.g. `bg-surface`, `text-muted`, `border-danger`), never raw values, so a visual design can be installed without touching components. Accessibility requirements that are architecture, not design: rem-based type that scales with user settings, minimum 44×44 px tap targets enforced in the base button/input variants, visible focus rings, and status conveyed by text plus color (ASMP-28 "the paid badge also carries a word").

### Component Library Decision

**shadcn/ui (copy-in components built on Radix UI primitives, React + Tailwind CSS)**, with the date and time-slot pickers hand-built on the same primitives (ADR-023).

Evidence: 70 Screen specs including an 8-step wizard (FEAT-15), a multi-step booking flow (FEAT-05), dialogs and confirmation sheets across cancel/reschedule/no-show/refund flows (FEAT-10, FEAT-11, FEAT-30), and ASMP-28's full screen-reader operability — the landscape's component-layer note ("Radix UI primitives — accessible unstyled behavior skinnable to tokens"; "shadcn/ui (copy-in components; React plus Tailwind CSS)") fits exactly. Copy-in means the code lives in `src/shared/components/ui/`, carries no visual lock-in, and restyles entirely from the `@theme` tokens — the right posture for a design-agnostic package. It satisfies the compatibility rules (React via ADR-001, Tailwind via ADR-005). React Aria Components remains the documented alternative if the builder wants a library-provided accessible calendar grid; a full styled suite was rejected because its visual opinions conflict with the design-agnostic posture.

## 10. Shared Infrastructure Patterns

| Pattern | When Used | Approach | File Location | Dependencies |
|---------|-----------|----------|---------------|-------------|
| Layout system | Every route | Route-group layouts: `(public)` minimal shell with Pro header; `(client)` shell with client nav for one Pro; `(pro)` app shell with bottom nav (mobile) / sidebar (≥ md); `(support)` read-only shell with a persistent "Support view — read only" banner. Layouts fetch the session once and pass it via context | `src/app/(public)/[handle]/layout.tsx`, `src/app/(client)/c/layout.tsx`, `src/app/(pro)/app/layout.tsx`, `src/app/(support)/support/layout.tsx`, `src/shared/components/AppShell.tsx` | Next.js App Router, Better Auth session helpers, Tailwind |
| Navigation | Pro and client shells | Declarative nav config per audience; active state from `usePathname()`; Attention badge count from the polled notices query; setup-incomplete Pros redirected to `/app/setup/[step]` in the `(pro)` layout | `src/shared/components/BottomNav.tsx`, `src/shared/lib/nav.ts` | Next.js navigation, TanStack Query |
| Error handling | All routes, actions, jobs, webhooks | Route-segment `error.tsx` boundaries with a retry button; server actions return typed `{ ok:false, code }` results (never throw to the UI for expected outcomes like "slot taken"); unexpected errors captured to Sentry with PII scrubbing (client name/phone/email/notes removed in `beforeSend`); Inngest failures after final retry raise a Pro attention flag where the spec requires (refunds, calendar, delivery) | `src/app/**/error.tsx`, `src/shared/lib/errors.ts`, `src/server/observability.ts` | Sentry (ADR-030), Inngest |
| Loading/empty states | Every screen that waits (ASMP-27) | `loading.tsx` skeletons per route segment shaped like the content; interactive controls rendered disabled until data resolves ("nothing appears tappable before real data has loaded"); `EmptyState` component with a spec-provided message and primary action; `OfflineBanner` shown from `navigator.onLine` + failed-fetch detection, with book/pay/cancel/refund/no-show buttons disabled and labelled "Needs a connection" | `src/app/**/loading.tsx`, `src/shared/components/Skeletons.tsx`, `src/shared/components/EmptyState.tsx`, `src/shared/components/OfflineBanner.tsx`, `src/shared/hooks/useOnlineStatus.ts` | TanStack Query persistence (ADR-013), shadcn/ui Skeleton |
| Form handling | All create/edit screens, booking flow, wizard steps | React 19 `useActionState` + server actions; one Zod schema per form in the feature's `schemas.ts`, used for client-side inline validation and re-validated on the server; every edit form submits the record's `version` so stale saves come back as reject-with-refresh; failed submissions preserve entered values (FEAT-15 feature-overview) | `src/features/*/schemas.ts`, `src/features/*/actions/`, `src/shared/components/ui/form.tsx` | Zod (ADR-024), shadcn/ui form primitives |
| Toast/notification | Action feedback | shadcn/ui Sonner toaster mounted in the root layout; bottom-center on mobile above the nav, auto-dismiss after 4 s, persistent (manual dismiss) for errors and for "refund in progress"; toasts announced via an ARIA live region. In-app Pro notifications (FEAT-08.SPEC-005/006, FEAT-28.SPEC-007) are a separate polled notices list, not toasts | `src/app/layout.tsx`, `src/shared/components/ui/sonner.tsx`, `src/shared/hooks/useToast.ts` | shadcn/ui, TanStack Query |
| Tenant scoping | Every Pro-data query and mutation | `withProScope(proAccountId, fn)` opens a transaction, sets `app.pro_account_id` via `set_config` for RLS, and passes a scoped Drizzle handle; lint rule forbids importing the raw db client outside `src/server/db/` | `src/server/db/tenant.ts` | Drizzle (ADR-004), Postgres RLS (ADR-003) |

## 11. Authentication & Access Architecture

### Authentication & Identity

| Field | Value |
|-------|-------|
| Context | Authentication signal: "Role-based -- Pro one-time-code sign-in with new-device alerts (FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-014, FEAT-29.SPEC-015; ASMP-30), client access links (FEAT-06), and a read-only Platform Operator (Support) role (FEAT-19)"; 21 driving specs; Access Matrix: 3 roles; Section 2 security posture (ASMP-30, ASMP-23); feasibility FEAT-29 Risk: "Managed auth products may not natively support all spec'd rules (dual-confirmation contact change, 30-day device inactivity, new-device alert on existing contacts)"; feasibility FEAT-06: access-link flow "needs own code" under a managed provider |
| Recommended | Framework-native auth: Better Auth (self-hosted library in the Next.js app) with the email-OTP and phone-number OTP plugins for Pros, database-backed sessions in Supabase Postgres, codes delivered through Twilio/Postmark (ADR-009); a custom, hashed-token access-link session for Clients (FEAT-06) and an admin-role Better Auth user with a read-only database role for Support (FEAT-19) |
| Rationale | Landscape: "Framework-native sessions and email OTP plugins; full control of access-link and device-alert logic". The spec'd rules — lockout, anti-enumeration (XBR-29), 30-day device inactivity expiry, new-device alerts (FEAT-29.SPEC-015), dual-confirmation contact change (FEAT-29.SPEC-012), sign-out everywhere — and the client access links (FEAT-06.SPEC-007) are custom logic under every managed option, so a managed provider would add a second user store to purge on closure (feasibility FEAT-29 Risk) without removing the custom work. Keeping identity in the primary database lets RLS and the tenant scope key directly on `pro_account_id`, and OTP delivery reuses the messaging stack already selected. No per-MAU fee. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Supabase Auth | 50,000 MAUs free; Pro $25/month includes 100,000 MAUs | Low with the Supabase project already selected | High | Medium — coupled to Supabase (open-source GoTrue) | Low | The team wants Supabase-managed OTP and sessions with RLS `auth.uid()` integration and accepts building device alerts, contact-change and access links around it |
| Clerk | Hobby free; Pro $25/month; 50,000 MRUs included; SMS OTP $0.01 (US/CA) | Low — prebuilt components | High | Medium–High — vendor-hosted user store | Low | Session/device management UI out of the box outweighs spec fidelity, and the product owner accepts Clerk's default new-device and contact-change behavior |
| WorkOS AuthKit | First 1,000,000 MAUs free | Low–Medium | Very high | Medium — vendor-hosted | Low–Medium | Enterprise features (SSO for multi-chair salons) enter scope |

### Session Model

- **Pro sessions (Better Auth):** opaque session token in an `HttpOnly`, `Secure`, `SameSite=Lax` cookie; session rows in Postgres (`session` table) carrying device label, user agent, IP-derived coarse location, created/last-active timestamps. Lifetime: sliding 30-day inactivity expiry (FEAT-29.SPEC-006) — each request older than 24 hours since last refresh extends it. New device = no prior session with the same device fingerprint cookie → FEAT-29.SPEC-015 alert queued via Inngest. "Sign out everywhere" deletes all session rows. Short-lived cookie cache (5 minutes) avoids a database read per request.
- **Client sessions (custom, FEAT-06):** access-link tokens are 256-bit random, stored only as SHA-256 hashes with scope `(pro_account_id, client_id, booking_id?)`. On-demand links are single-use and expire after 30 minutes; booking-specific manage links are valid until the appointment passes (FEAT-06.SPEC-007). Redemption is a `POST` from an interstitial page (so messaging-app link previews issuing `GET` do not consume single-use links — feasibility FEAT-06 risk) and mints a signed, `HttpOnly` client-session cookie scoped to that client × Pro for the remaining link life (max 24 hours for on-demand links).
- **Support sessions:** Better Auth session for a user with `role = 'support'`, plus a support-session record `(operator_id, pro_account_id, reason, started_at, ended_at)`; opening a new lookup ends the prior one (FEAT-19.SPEC-004).

### User Model Fields

- `user` (Better Auth, identity): `id`, `email` (unique, nullable), `email_verified_at`, `phone_e164` (unique, nullable), `phone_verified_at`, `role` (`pro` | `support`), `failed_code_attempts`, `locked_until`, `created_at`, `updated_at`.
- `pro_account` (profile, 1:1 with a `pro` user): `id`, `user_id`, `handle` (unique, reserved-word checked), `display_name`, `photo_key`, `studio_address`, `timezone` (IANA), `currency` (ISO 4217), `currency_locked_at`, `status` (`setup` | `live` | `paused` | `system_paused` | `closing` | `closed`), `closure_requested_at`, `stripe_connected_account_id`, `stripe_customer_id`.
- `session`: `id`, `user_id`, `token_hash`, `device_label`, `user_agent`, `ip_region`, `created_at`, `last_active_at`, `expires_at`.
- `verification` (OTP): `id`, `identifier`, `code_hash`, `purpose` (`sign_in` | `recovery` | `contact_change_old` | `contact_change_new`), `expires_at` (`sign-in-code-expiry-minutes`), `attempts`.
- `access_link` (FEAT-06): `id`, `pro_account_id`, `client_id`, `booking_id` (nullable), `token_hash`, `kind` (`on_demand` | `manage`), `expires_at`, `used_at`.
- `support_session`: `id`, `operator_user_id`, `pro_account_id`, `reason`, `ticket_ref`, `started_at`, `ended_at`.
Exact column types, constraints and indexes are owned by `.n2b/architecture/database-schema.md`.

### Protected Routes

| Route / Route Group | Access Requirement | Unauthenticated Experience |
|--------------------|--------------------|----------------------------|
| `/[handle]`, `/[handle]/book/**`, `/[handle]/waitlist/**`, `/[handle]/access` | Public (FEAT-05, FEAT-20.SPEC-001, FEAT-06.SPEC-001); gated by the booking-page availability rule (FEAT-05.SPEC-008) | Closed/mistyped handle → plain "this booking page isn't available" message; paused Pro → "not accepting bookings" |
| `/pay/[token]`, `/o/[token]`, `/r/[token]`, `/m/[token]` | Valid unexpired token scoped to one client × Pro (FEAT-06.SPEC-007, FEAT-14.SPEC-002, FEAT-08.SPEC-003) | Expired/invalid token → "request a new link" prompt |
| `/c/**` | Client session for one client × one Pro (FEAT-06.SPEC-003/004/005, FEAT-10, FEAT-14.SPEC-001, FEAT-20.SPEC-002, FEAT-21, FEAT-22, FEAT-23, FEAT-26) | "Request a new link" prompt (FEAT-06.SPEC-001), never another client's data |
| `/sign-in`, `/sign-in/recover` | Public (FEAT-29.SPEC-001/002); anti-enumeration responses | -- |
| `/app/**` | Authenticated Pro session; data scoped to own `pro_account_id` (FEAT-12.SPEC-008, FEAT-11.SPEC-004 own-bookings-only) | Redirect to `/sign-in?next=…` (Access Matrix "Unauthorized access") |
| `/app/setup/**` vs rest of `/app` | Pro with `status = setup` is redirected into the wizard (FEAT-15) | -- |
| `/support/**` | Role-gated (`support`); read-only DB role; one Pro account at a time (FEAT-19.SPEC-001, FEAT-19.SPEC-004) | Redirect to `/sign-in`; a Pro user receives 404 |
| `/api/webhooks/**` | Provider signature verification (Stripe, Twilio, Postmark, Nylas) | 401 on bad signature |
| `/api/inngest` | Inngest signing key | 401 |
| `/api/slots/[handle]` | Public, rate-limited per IP | -- |

### Role & Permission Mapping

| Role (Access Matrix) | Application Representation | Capability Access Summary | Enforcement Point |
|----------------------|---------------------------|---------------------------|-------------------|
| The Pro (Talia) | `user.role = 'pro'` + owning `pro_account.id` in the session | Full: Service & Availability Setup, Booking & Payment, Client Records, Cancellation & No-Show Handling, Messaging & Consent (can see, never override client texting consent), Subscription & Billing, Profile & Account Settings, Payouts, Recurring Appointments. View: Activity Record & Insights (append-only), Waitlist, Support Access Log | `(pro)` layout route guard; `withProScope` in every server action/query; Postgres RLS policies on `pro_account_id`; append-only grants on Activity Event |
| The Client (Riley) | Custom client-session cookie carrying `(pro_account_id, client_id, booking_id?)` from a redeemed access link; no user account | Own-only: Booking & Payment, Cancellation & No-Show Handling, Messaging & Consent (their own consent), Waitlist, Recurring Appointments. View: public profile only (Profile & Account Settings). None: Service & Availability Setup, Client Records, Subscription & Billing, Payouts, Activity Record & Insights, Support Access Log | `(client)` layout guard; client-scope query helper filtering by `client_id` AND `pro_account_id` (FEAT-06.SPEC-008); RLS client policy |
| Platform Operator (Support) | `user.role = 'support'` + active `support_session.pro_account_id` | View-only across all 12 capability groups for one Pro at a time; never the Pro's private notes, bank/identity details or sign-in codes (XBR-24) | `(support)` role guard; queries run on a separate connection as a Postgres role with `SELECT`-only grants and column-masked views (no write path exists structurally — feasibility FEAT-19 risk); every view writes a support-view Activity Event (FEAT-19.SPEC-002) |

The identity decision introduces no external runtime service (Better Auth runs in-app); its code-delivery dependencies are the Twilio and Postmark rows already in Section 8.

## 12. Development Conventions

### Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Files (components) | PascalCase `.tsx`, one exported component per file | `SlotList.tsx` |
| Files (utilities) | kebab-case `.ts`; server actions named verb-first | `deposit-outcome.ts`, `cancel-booking.ts` |
| Components | PascalCase, noun-first, feature-prefixed only when ambiguous | `export function BookingStatusBadge()` |
| Functions | camelCase, verb-first; rules as pure `evaluate*`/`is*` functions | `evaluateDepositOutcome(booking, policy)` |
| CSS classes | Tailwind utilities using semantic token names only; `cn()` for conditional merges | `className={cn("bg-surface text-fg", isPaid && "border-success")}` |
| Database tables | snake_case, singular; foreign keys `{entity}_id`; timestamps `*_at` (`timestamptz`) | `deposit_transaction.booking_id` |
| Routes | kebab-case segments; dynamic params camelCase in brackets | `/app/bookings/[bookingId]/bulk-cancel` |
| Inngest events | `{domain}/{past-tense-verb}` | `booking/cancelled`, `refund/requested` |
| Env vars | SCREAMING_SNAKE, `NEXT_PUBLIC_` only for browser-safe values | `STRIPE_SECRET_KEY`, `NEXT_PUBLIC_POSTHOG_KEY` |

### Component Structure

Order inside a component file: (1) imports, (2) exported prop types, (3) component function, (4) small private sub-components, (5) no default exports. Server components are the default; add `"use client"` only to leaf components that need state, effects or browser APIs, and never import `src/server/**` from a client component (enforced by the `server-only` package).

```tsx
import { formatMoney } from "@/shared/lib/money";
import type { Booking } from "@/features/feat-12-pro-daily-schedule-dashboard/queries";

export type BookingRowProps = { booking: Booking };

export function BookingRow({ booking }: BookingRowProps) {
  return <li>{booking.clientFirstName} · {formatMoney(booking.balanceDue)}</li>;
}
```

### Import Ordering

Groups separated by a blank line, enforced by ESLint `import/order`: (1) Node/React/Next built-ins, (2) third-party packages, (3) `@/server/*`, (4) `@/features/*`, (5) `@/shared/*`, (6) relative imports, (7) type-only imports last.

```ts
import { cache } from "react";

import { and, eq } from "drizzle-orm";

import { withProScope } from "@/server/db/tenant";

import { evaluateDepositOutcome } from "@/features/feat-09-cancellation-no-show-policy-engine/rules/deposit-outcome";

import { toProTime } from "@/shared/lib/time";

import type { CancelInput } from "./schemas";
```

### TypeScript Usage

`strict: true` plus `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` (ADR-027). Prefer `type` aliases for data shapes and discriminated unions (booking/deposit status machines are unions, exhaustively `switch`ed with a `never` check); `interface` only for extendable component props. `any` is banned (`@typescript-eslint/no-explicit-any` error); use `unknown` and narrow via Zod at every boundary (form input, webhook payload, job event). Money is a branded `MinorUnits` number type; IDs are branded per entity (`BookingId`, `ClientId`) so cross-entity mix-ups fail to compile.

### Path Aliases

Defined in `tsconfig.json` `compilerOptions.paths`:

```json
{
  "@/app/*": ["./src/app/*"],
  "@/features/*": ["./src/features/*"],
  "@/server/*": ["./src/server/*"],
  "@/shared/*": ["./src/shared/*"],
  "@/db/*": ["./src/server/db/*"]
}
```

## 13. Deployment & Environments

### Hosting & Environments

| Field | Value |
|-------|-------|
| Context | Frontend/backend: Next.js 15 server layer (ADR-001, ADR-002); database Supabase Postgres US East (ADR-003); Section 2: a few hundred pros in year one, one-second slots (ASMP-21), availability expressed as correctness not uptime (ASMP-26), US first then UK/CA/AU (ASMP-25); feasibility FEAT-03 risk: "Serverless cold starts … can consume much of the one-second budget" |
| Recommended | Vercel (Pro plan, $20/month per member) with functions pinned to `iad1` (Washington, D.C.) co-located with the Supabase US East database, Fluid compute enabled to keep instances warm, production + per-branch preview deployments |
| Rationale | Landscape: "Preview environments per branch, native Next.js hosting, cron jobs on all plans" at $20/month. Co-locating functions with the database keeps the slot query round-trip to single-digit milliseconds; Fluid compute's instance reuse addresses the cold-start risk, and the FEAT-03 spike verifies the one-second target on this exact setup. Zero server operations for a small team; the stack stays portable (Next.js runs on any Node host) if the spike argues for a long-running host. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Render | Container instances by size; managed cron and workers | Low | High | Low — container-based | Low | The FEAT-03 spike shows serverless cold starts breach ASMP-21, or long-running workers (BullMQ, SSE) are adopted |
| Fly.io | Usage-based Machines and volumes | Medium — Dockerfile and fly config | High — multi-region | Low — containers | Medium | UK/AU expansion requires serving slots from regions near those users with read replicas |
| AWS (ECS Fargate / App Runner) | Usage-based compute and networking | High — IaC required | Very high | Medium — AWS service coupling | AWS/IaC specialists | A compliance audit or enterprise customer requires a single-cloud boundary with VPC isolation |

### CI/CD & Delivery

| Field | Value |
|-------|-------|
| Context | Hosting: Vercel (ADR-028); ASMP-26 correctness bar across 30 features, 29 XBRs and 320 touchpoint rows (feasibility Key Risk "Correctness regressions … Automated tests on booking, payment and refund logic"); migration-based schema changes (ADR-019) |
| Recommended | GitHub Actions for checks (typecheck, lint, unit tests on Logic/Rule modules, integration tests against a Postgres service container including concurrency tests on holds/exclusion constraints, Playwright e2e against the preview URL) + Vercel Git integration for preview and production deploys; `drizzle-kit migrate` runs as a gated Actions job before production promotion |
| Rationale | Landscape: GitHub Actions "Workflows on pull requests, matrix tests, environment protections and deploy hooks" with 2,000–3,000 free minutes, and Vercel's native deploys give a preview per pull request for free. Concurrency tests derived from the spec Contention lines run on every PR because double-booking is the product's failure mode (ASMP-26). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Vercel Git integration only (tests elsewhere/none) | Included with hosting plan | Lowest | Medium | Medium — platform-coupled | Low | Never for production given ASMP-26; acceptable only for a throwaway prototype |
| GitLab CI/CD | Tiered per user plus compute minutes | Medium — needs GitLab repository | High | Medium | Medium | The organization's source of truth is GitLab |
| CircleCI | Credit-based with free tier | Low–Medium | High | Medium | Medium | Test suite duration grows past ~15 minutes and CircleCI's parallelism and test splitting pays for itself |

**Environment topology:**
- **Development (local):** each engineer runs Next.js, a local Supabase stack (Postgres + Storage), the Inngest Dev Server, Stripe CLI webhook forwarding and test-mode keys for Twilio/Postmark/Nylas. Seed data from `src/db/seed.ts`.
- **Preview (per pull request):** Vercel preview deployment against the **staging** Supabase project with an isolated schema reset per run for e2e tests; Inngest branch environment; Stripe/Twilio/Postmark test keys. No real messages leave the system (Twilio test credentials; Postmark test token).
- **Staging (`main` branch):** a persistent Vercel environment and a separate Supabase project mirroring production configuration; Stripe test mode with test clocks; a real Twilio test Messaging Service restricted to team phone numbers; Nylas sandbox. Migrations apply here first.
- **Production (tagged release promoted from `main`):** Vercel production, production Supabase project (Pro tier, daily backups + PITR add-on recommended once real money flows), live keys. Promotion requires green CI plus a manual approval in a GitHub Environment.
- **Secrets:** stored per environment in Vercel Environment Variables (and GitHub Environment secrets for the migration job); never in the repo; live keys exist only in Production. Webhook signing secrets differ per environment.

**Infrastructure as code:** Not warranted at launch — the footprint is five managed accounts (Vercel, Supabase, Inngest, Stripe, Twilio/Postmark/Nylas) configured through dashboards with the settings recorded in a `docs/infrastructure.md` checklist, and the database schema is already code (Drizzle migrations). Growth trigger: adopting Terraform (Vercel and Supabase providers) when a second production region (UK/AU) is added or when more than two engineers change platform configuration.

### Observability & Operations

| Field | Value |
|-------|-------|
| Context | Section 2: correctness bar ASMP-26, no numeric uptime target; Background processing (31 timed Automation specs) and 13 Integration specs; feasibility Key Risks: stuck refunds ("Indefinite retry needs monitoring"), lost/out-of-order webhooks, sync health (FEAT-04.SPEC-006), delivery failures (FEAT-08.SPEC-009); Compliance/privacy: error payloads must not leak client data (feasibility FEAT-13 Risk) |
| Recommended | Sentry (Team tier) for error tracking and performance tracing across requests, server actions, webhooks and Inngest functions, with PII scrubbing; Better Stack for uptime checks on the public booking page, `/api/slots` and webhook endpoints plus log retention via a Vercel log drain; Inngest's run dashboard for job failures; SQL-based correctness alerts (a scheduled Inngest check alerting on refunds "in progress" older than 48 hours, outbox rows undelivered older than 5 minutes, and any overlapping confirmed bookings) |
| Rationale | Landscape: Sentry "Error tracking, tracing, session replay" at $26/month Team; Better Stack "Uptime checks, log management, on-call". Because reliability is a correctness bar rather than an uptime number, the alerts that matter are domain invariants (stuck refunds, undelivered side effects, double-bookings) — cheap scheduled queries surfaced through the same alerting channel — rather than a full APM suite. Sentry's `beforeSend` scrubbing keeps client personal data out of a third-party store (XBR-19). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Grafana Cloud (OpenTelemetry) | Free tier plus usage-based | Medium — instrumentation | High | Low — open standards | Medium | The team wants vendor-neutral OTel traces spanning webhook → outbox → job chains and metrics dashboards in one place |
| Datadog | Per-host and per-GB usage | Medium | Very high | High — proprietary | Medium | The organization already runs Datadog and wants APM, logs and on-call unified |
| OpenTelemetry with Honeycomb | Free tier plus event-volume pricing | Medium — instrumentation work | High | Low — standards-based | Medium | Debugging cross-service event chains (webhook ordering, refund retries) becomes the dominant operational pain |

### Indicative Cost Model

Order-of-magnitude only; launch = first months with tens of pros; growth tier = the year-one volume of a few hundred pros (Section 2). Card processing fees (2.9% + 30¢) are borne by each Pro's connected account under direct charges and are excluded.

| Component | Launch (order of magnitude) | Growth tier |
|-----------|-----------------------------|-------------|
| Vercel (Pro) | ~$20–40/month (1–2 seats) | ~$40–150/month (seats + usage) |
| Supabase (Pro: Postgres + Storage) | ~$25/month | ~$25–100/month (compute add-on, PITR) |
| Inngest | free tier ($0, 50k executions/month) | ~$99–200/month (Pro) |
| Twilio SMS (+10DLC fees) | ~$50–150/month | ~$1,000–1,500/month (~100,000+ texts/month at ~$0.0083 + carrier fees) |
| Twilio WhatsApp (Later) | $0 (not in v1) | ~$50–300/month usage-scaled |
| Postmark | ~$15/month | ~$15–50/month |
| Stripe Connect active-account fee | ~$20–100/month (~$2 × 10–50 active pros) | ~$400–800/month (~$2 × 200–400 active pros) |
| Stripe Billing (subscriptions) | ~$5–20/month (per-invoice percentage) | ~$50–150/month |
| Nylas (calendar) | ~$49–100/month | ~$300–700/month (per connected account) |
| Sentry (Team) | ~$26/month | ~$26–80/month |
| Better Stack | free tier ($0) | ~$25–50/month |
| PostHog | free tier ($0, 1M events/month) | ~$0–100/month |
| GitHub Actions | free tier ($0, 2,000–3,000 minutes) | ~$0–50/month |

### Local Development Setup

Prerequisites: Node.js 22 LTS, pnpm 9+, Docker (for the local Supabase stack), the Supabase CLI, the Stripe CLI, and accounts with test keys for Twilio, Postmark and Nylas. Path from clone to running: `pnpm install` → `supabase start` (local Postgres + Storage) → copy `.env.example` to `.env.local` and fill test keys → `pnpm db:migrate && pnpm db:seed` → in separate terminals run `pnpm dev`, `npx inngest-cli dev`, and `stripe listen --forward-to localhost:3000/api/webhooks/stripe`. Local substitutes: Supabase CLI for Postgres/Storage, Inngest Dev Server for jobs, Stripe test mode, Twilio test credentials (no real SMS), Postmark test token (no real email); calendar sync needs a Nylas sandbox app with test Google/iCloud accounts, and can be disabled via a feature flag so the slot engine runs on Chairtime-only data.

## 14. Decision Log

| ID | Category | Decision | Rationale (one-line) | Profile Driver |
|----|----------|----------|---------------------|---------------|
| ADR-001 | Frontend Framework | Next.js 15 (App Router, React 19, TypeScript) | Server-rendered public booking pages with small client bundles for the in-app browser; one deployable for 70 screens and webhooks | Scale: Large (30 features, 219 specs); 70 Screen specs; Interaction Complexity: Large; ASMP-21 one-second slots |
| ADR-002 | Backend / API Layer | Next.js server layer (route handlers + server actions, Node runtime) with long-running work delegated to the job runner | No second deployable or network hop inside the one-second budget; webhooks as route handlers | 59 Automation specs; 13 Integration specs with inbound webhooks; FEAT-07.SPEC-004 idempotency |
| ADR-003 | Database | Supabase Postgres (Pro tier, US East) with btree_gist exclusion constraints and RLS defense-in-depth | Database-enforced non-overlap and per-Pro isolation; always-on (no cold start); bundles storage and a push upgrade path | 18 entities; Collaboration/concurrency (16 of 18 entities, Booking High); ASMP-26; ASMP-23; feasibility FEAT-03 Hard |
| ADR-004 | ORM / Data Access | Drizzle ORM + drizzle-kit over postgres.js via the Supabase pooler | SQL-close typed transactions, row locks and compare-and-set for the contention core; custom SQL in migrations | 18 entities; 56 relationships; first-committed-wins / reject-with-refresh contention styles |
| ADR-005 | CSS / Styling | Tailwind CSS v4 with @theme CSS custom properties as the token layer | Zero-runtime responsive styling; a builder-supplied visual design installs as tokens | 70 Screen specs; ASMP-28 accessibility baseline; design_system_source: none |
| ADR-006 | State Management | TanStack Query v5 for server state + framework built-ins (URL, React state, server-persisted wizard) | Polling, invalidation for reject-with-refresh, and offline persistence in one library | Real-time signal (FEAT-03.SPEC-001 ~1s refresh); Offline signal (ASMP-27); Complex forms (FEAT-15 8-step wizard) |
| ADR-007 | Build Tooling | Framework-bundled Next.js/Turbopack pipeline with pnpm | Meta-framework pipeline needs no override; strict installs for a large codebase | Scale: Large (219 specs, 30 features); single web app |
| ADR-008 | File & Object Storage | Supabase Storage: public profile-photos bucket (EXIF stripped, content-addressed) + private exports bucket with signed URLs | Same project as the database, 100 GB included; immediate photo updates; private generated files | File upload signal (FEAT-27.SPEC-012; ASMP-35); Import/export signal (FEAT-29.SPEC-007, FEAT-16.SPEC-004) |
| ADR-009 | Email & Messaging Delivery | Twilio Programmable Messaging (shared 10DLC Messaging Service; WhatsApp later on same account) + Postmark for email; inbound STOP revokes texting consent on all Client records with that phone | Only landscape option carrying WhatsApp; delivery status and STOP webhooks; transactional-first email | Notifications signal (24 specs); FEAT-08.SPEC-012/013, FEAT-26.SPEC-002; ASMP-24; ASMP-29 |
| ADR-010 | Payments & Billing | Stripe Connect (hosted onboarding, direct charges, zero application fee) + Stripe Billing for Pro subscriptions; grace clock follows Stripe invoice events | One platform for charges, refunds on the Pro's balance, disputes, payouts and subscriptions in US/UK/CA/AU | Payments/billing signal (44 specs, 7 Payment-processing Integration specs); ASMP-31; XBR-07 |
| ADR-011 | Search | PostgreSQL built-ins (pg_trgm GIN on name/phone, per-Pro scope) + client-side filtering of the cached list | Simple filter over ≤500 clients per Pro needs no second personal-data store | Search signal: Simple filter (FEAT-24.SPEC-001/002); ASMP-22 (100–500 clients per pro); ASMP-23 |
| ADR-012 | Background Jobs & Scheduling | Inngest durable functions (steps, sleepUntil, cron, retries, per-Pro concurrency keys) fed by a Postgres outbox; holds expire lazily | Durable delayed/retried chains on serverless hosting; correctness-critical expiry never depends on the runner | Background processing signal (31 timed Automation specs); FEAT-09.SPEC-006 indefinite retry; FEAT-30.SPEC-008 bulk fan-out |
| ADR-013 | Caching & Performance | Database indexing and query design (no server cache service) + TanStack Query cache persisted to IndexedDB for offline reads | Per-Pro data is small and index-bound; a Redis hold cache would be a second source of truth | Scale hints (ASMP-21, ASMP-22); Offline signal (ASMP-27, 33 already-loaded specs) |
| ADR-014 | Real-time & Collaboration | Polling via TanStack Query (1s slot list while visible, 15s Pro notices) with DB-enforced conflict resolution; Supabase Realtime as spike-triggered upgrade | Meets ~1s refresh with no extra infrastructure; correctness independent of transport | Real-time signal (FEAT-03.SPEC-001, FEAT-08.SPEC-005, FEAT-28.SPEC-002; no WebSocket named); Collaboration/concurrency signal |
| ADR-015 | Analytics & Product Telemetry | PostHog Cloud (US) with Pro-only person profiles, anonymous client funnel, no client PII in events | Funnels for the one-minute booking and feature flags within the free tier, without a client-deletion purge burden | Scale hints (ASMP-22 few hundred pros; ASMP-21 benchmarks); ASMP-23 privacy |
| ADR-016 | Internationalization | date-fns v4 + date-fns-tz for IANA-zone arithmetic, Native Intl for display, integer minor units + ISO 4217 currency; no translation catalog | Per-account timezone/currency never hard-coded; DST-safe local-time rules; no multi-language spec'd | Internationalization signal (ASMP-25; FEAT-27.SPEC-003, FEAT-27.SPEC-008, FEAT-07.SPEC-001, FEAT-22.SPEC-001) |
| ADR-017 | Calendar Sync | Nylas (calendar-only, Pro plan, per-connected-account) for Google Calendar + iCloud free/busy reads and booking writes; FEAT-04 spike confirms iCloud latency | One API for both mandated providers instead of two protocols; far below Cronofy's $819/month base at a few hundred pros | Product-mandated (BRIEF Ecosystem & Integrations; ASMP-33); FEAT-04 Research-spike verdict; ASMP-22 |
| ADR-018 | Project Structure | Single Next.js package (no monorepo) with route groups per audience, src/features/feat-NN-* logic folders and a server-only src/server layer | One deployable matches ADR-001/ADR-002; feature isolation mirrors the 30 Stage 3 folders | 30 features; 219 specs; 4 audiences (Access Matrix 3 roles + public visitors) |
| ADR-019 | Data Layer Design | Migration-based schema changes (drizzle-kit generate → reviewed SQL → migrate in CI; never push to shared environments) | Exclusion constraints, RLS, append-only grants and the support role must be versioned and reviewed | 18 entities; 56 relationships; Collaboration/concurrency signal; ASMP-26 |
| ADR-020 | API & Routing Architecture | Root-level public booking pages /{handle} with reserved-word list; /c client, /app Pro, /support, short token prefixes /m /o /r /pay | Short Instagram-bio links; audience separation by route group for guards and layouts | 70 Screen specs; Access Matrix 3 roles; BRIEF Instagram booking-link placement |
| ADR-021 | API & Routing Architecture | Server components + server actions for reads/mutations, TanStack Query polling for live data, JSON route handlers only for polled reads, webhooks, jobs and auth; payment confirmation only from webhook | Fast first paint in the in-app browser; typed reject-with-refresh results; exactly-once payment outcome | ASMP-21; Real-time signal; FEAT-07.SPEC-002/SPEC-004; Collaboration/concurrency signal |
| ADR-022 | Integration Architecture | Inbound webhooks persisted to an inbound_events table (unique provider event ID, event-time ordering) before 2xx; booking side effects via a transactional outbox relayed to Inngest | Duplicate/out-of-order events become no-ops; no commit loses a refund, calendar mirror or notice | 13 Integration specs (all with duplicate/out-of-order Edge Cases); feasibility themes "Inbound webhook ingestion" and "Booking commit side-effect fan-out" |
| ADR-023 | CSS / Styling | Component library: shadcn/ui (copy-in, Radix primitives) with hand-built date/slot pickers on the same primitives | Accessible dialogs, sheets and wizard primitives with no visual lock-in for a design-agnostic package | 70 Screen specs; FEAT-15 8-step wizard; FEAT-05 multi-step flow; ASMP-28 screen-reader operability |
| ADR-024 | State Management | Form handling: React 19 useActionState + server actions with Zod schemas shared client/server (landscape deviation: the landscape researched no schema-validation library; Zod is the de facto TypeScript runtime validator with first-class Next.js/shadcn form support) | One validation source for 53 Logic/Rule specs' field rules on both sides of the wire; version field enables reject-with-refresh | 53 Logic/Rule specs; Complex forms signal (FEAT-15, FEAT-05); Collaboration/concurrency signal |
| ADR-025 | Authentication & Identity | Better Auth (framework-native) with email and phone OTP for Pros, DB sessions in Postgres; custom hashed-token access-link sessions for Clients; support role with read-only DB role | Spec'd OTP, device, contact-change and access-link rules are custom under every managed option; identity stays in the primary database for RLS | Authentication signal: Role-based (21 specs; FEAT-29, FEAT-06, FEAT-19); Access Matrix: 3 roles; ASMP-30 |
| ADR-026 | Authentication & Identity | Session model: HttpOnly DB-backed Pro sessions with 30-day sliding inactivity expiry and new-device alerts; client sessions minted by POST redemption of 30-minute single-use or per-appointment manage links | Matches FEAT-29.SPEC-006 and FEAT-06.SPEC-007 lifetimes; POST redemption survives messaging-app link previews | Authentication signal (FEAT-29.SPEC-006, FEAT-29.SPEC-015, FEAT-06.SPEC-007); ASMP-30 |
| ADR-027 | Development Conventions | TypeScript strict + noUncheckedIndexedAccess + exactOptionalPropertyTypes; no any; branded IDs and money types; exhaustive status unions | Compile-time guards against cross-entity ID mix-ups and unhandled booking/deposit states | ASMP-26 correctness bar; 18 entities; 29 XBRs; Collaboration/concurrency signal |
| ADR-028 | Hosting & Environments | Vercel Pro, functions pinned to iad1 co-located with Supabase US East, Fluid compute, production + preview deployments | Zero-ops native Next.js hosting; co-location and warm instances protect the one-second slot budget | Scale hints (ASMP-22 few hundred pros); ASMP-21; ASMP-26 (no numeric uptime target); ASMP-25 US first |
| ADR-029 | CI/CD & Delivery | GitHub Actions (typecheck, lint, unit, Postgres concurrency tests, Playwright e2e on previews, gated migrations) + Vercel Git deploys; dev/preview/staging/prod topology | Every PR proves the booking core against double-booking before merge | ASMP-26; 29 XBRs; 320 cross-feature touchpoint rows; Booking contention High |
| ADR-030 | Observability & Operations | Sentry Team (errors + tracing, PII scrubbed) + Better Stack uptime/log drain + Inngest dashboard + scheduled domain-invariant alerts (stuck refunds, undelivered outbox, overlapping bookings) | Reliability is a correctness bar, so invariant alerts matter more than full APM | ASMP-26; Background processing (31 timed specs); 13 Integration specs; Compliance/privacy signal |

