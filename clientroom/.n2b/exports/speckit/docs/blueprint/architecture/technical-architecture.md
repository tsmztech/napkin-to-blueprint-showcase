---
document_type: technical-architecture
produced_by: technical-architect
status: final
stage: 4
prerequisites:
  - .n2b/architecture/technical-profile.md
  - .n2b/architecture/technology-landscape.md
  - .n2b/architecture/technical-feasibility.md
created: 2026-09-29
---

# Technical Architecture -- Clientroom

## 1. Project Technical Profile

# Project Technical Profile

## 1. Scale Metrics

Quantitative scale metrics extracted from Stage 3 outputs. Feature count from the `FEAT-*` folders in `.n2b/specifications/` (`ls -d .n2b/specifications/FEAT-*/ | wc -l`); total specs from the spec files on disk (`find .n2b/specifications/FEAT-*/ -name "FEAT-*.SPEC-*.md" | wc -l`); spec types from spec frontmatter (`spec_type:`) across all five types. The five type counts sum to the total (65 + 62 + 60 + 7 + 26 = 220); the Spec Inventory rows across all feature-overview.md files also total 220 and match the on-disk spec IDs one-to-one.

| Metric | Value |
|--------|-------|
| Total features | 33 |
| Total specs | 220 |
| Screen specs | 65 |
| Automation specs | 62 |
| Logic/Rule specs | 60 |
| Integration specs | 7 |
| Notification specs | 26 |
| User-Facing features | 18 |
| Platform features | 8 |
| Lifecycle features | 7 |

## 2. Complexity Metrics

Structural complexity metrics extracted from product-features.md, feature-dependency-map.md, and feature-overview.md files. All values are counted, not estimated.

| Metric | Value |
|--------|-------|
| Entity count | 23 |
| Inter-entity relationships | 60 (named entity-to-entity relationships, counted directionally, across the 21 `**Relationships:**` lines of the Shared Data Entities section; 21 of the 23 entities appear there, Accounting Export File and Data Export Archive are single-feature entities not listed) |
| Cross-feature business rules (XBR) | 35 |
| Cross-feature touchpoint rows | 326 |
| Cross-feature integration density | 9.88 (326 / 33) |
| Navigation connections | 54 |
| Hub screens (3+ inbound connections) | 1 -- Client Contact List (FEAT-18): 3 inbound (from FEAT-01 client detail, FEAT-02 proposal (sent), FEAT-14 delivery warning on a project). At feature level, destination features with 3+ inbound connections: FEAT-09 (5), FEAT-05 (5), FEAT-18 (4), FEAT-10 (4), FEAT-02 (4), FEAT-13 (3), FEAT-01 (3) |
| Average specs per feature | 6.67 (220 / 33) |

## 3. Capability Signals

Capability signals detected by `grep -rn -i` over all 220 specs (all five types, including Integration Capability Category / Data Exchanged / Inbound Events / Degradation Behavior sections and Notification Channels / Trigger / Delivery Rules sections), plus BRIEF.md and assumptions-constraints.md for the demand-side signals. Where a keyword matches a large share of specs through incidental wording, the Detail column gives the match count and the specs whose primary behavior it is; the per-spec keyword hits are carried in Section 6.

| Signal | Present | Detail |
|--------|---------|--------|
| Real-time | No | No specs reference real-time behavior as a capability. The keyword "live-updating" matches only statements that a screen is a snapshot and "not live-updating" (e.g. FEAT-01.SPEC-003, FEAT-01.SPEC-004, FEAT-03.SPEC-002, FEAT-05.SPEC-003, FEAT-07.SPEC-002); FEAT-31.SPEC-001 excludes "a live chat or real-time conversation" from its scope. No WebSocket or collaborative-editing behavior is specified |
| Offline | Yes | FEAT-07.SPEC-008 (Offline Comment Queue & Sync: comments composed offline are queued and synced on reconnect), FEAT-29.SPEC-004 (Feed Load Fallback & Offline Continuity); offline banners and degraded read-only states appear in 79 specs by keyword match (e.g. FEAT-06.SPEC-003 resumable upload resumes after a connection drop, FEAT-12.SPEC-001 "showing your last loaded totals"). ASMP-27 (assumptions-constraints.md, Non-Functional Expectations): actions that create records (accept, approve, pay) never pretend to succeed offline. Not an offline-first product |
| File upload | Yes | 37 specs match "upload"; primary: FEAT-06.SPEC-001 (Deliverable Upload), FEAT-06.SPEC-003 (Resumable Upload Handling), FEAT-06.SPEC-004 (Linked Asset Reachability Check), FEAT-16.SPEC-002 (Resumable Upload Transfer), FEAT-16.SPEC-003 (Reliable File Delivery), FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules), FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability, Integration), FEAT-17.SPEC-001 (New Version Upload), FEAT-17.SPEC-003 (Version Creation & Preservation), FEAT-19.SPEC-001 (Branding Settings, logo upload), FEAT-19.SPEC-002; ASMP-22 (files tens of MB, sometimes over 1 GB) and ASMP-30 |
| Complex forms (10+ fields) | No | No specs exceed 10 fields: the highest Layout and Content input-control count in any Screen spec is 7 (FEAT-09.SPEC-003 Manual Invoice & Credit Note Issuance), then 6 (FEAT-23.SPEC-001) and 5 (FEAT-20.SPEC-001, FEAT-11.SPEC-003). Multi-step sequence present: FEAT-20.SPEC-002 (Onboarding Guided Sequence) |
| Background processing | Yes | 15 specs match scheduled/cron/daily/weekly/periodic: FEAT-06.SPEC-004, FEAT-09.SPEC-002, FEAT-09.SPEC-003, FEAT-11.SPEC-001 (Reminder Schedule), FEAT-11.SPEC-002, FEAT-11.SPEC-003, FEAT-11.SPEC-005, FEAT-12.SPEC-004 (Dashboard Totals Refresh), FEAT-14.SPEC-003 (Delivery Status Tracking & Retry), FEAT-15.SPEC-006, FEAT-16.SPEC-005 (Storage Usage Aggregation), FEAT-21.SPEC-005, FEAT-23.SPEC-003, FEAT-24.SPEC-003 (Data Export Archive Generation), FEAT-24.SPEC-005 (Legal Retention Purge); also FEAT-04.SPEC-004 and FEAT-31.SPEC-004 (Support Session Auto-Close on Inactivity) are event/timer-driven Automation specs |
| Authentication | Yes | Role-based -- FEAT-05.SPEC-001 (Request Sign-In Link), FEAT-05.SPEC-002 (Link Verification Landing), FEAT-05.SPEC-004 (Magic Link Issuance), FEAT-05.SPEC-005 (Magic Link Verification), FEAT-05.SPEC-006 (Link Validity & Recognition Rules), FEAT-05.SPEC-007 (Portal Access & Isolation Rules), FEAT-20.SPEC-001 (Sign-Up & Account Creation), FEAT-21.SPEC-003 (Login & Security), FEAT-21.SPEC-005 (Sign-In Email & Login Method Change), FEAT-21.SPEC-006 (Sign-Out Other Sessions); roles named across specs: Freelancer (Nadia), Client Primary Contact (Owen), Client Reviewer Contact (Priya), Support Operator (Dana), with role rules in FEAT-18.SPEC-007, FEAT-08.SPEC-006, FEAT-09.SPEC-006, FEAT-13.SPEC-005, FEAT-21.SPEC-010, FEAT-31.SPEC-005. 107 specs match the sign-in keywords in total |
| Search | Yes | Simple filter with a cross-entity search feature -- FEAT-28.SPEC-001 (Global Search), FEAT-28.SPEC-002 (Cross-Entity Search Execution), FEAT-28.SPEC-003 (Search Scope & Access Rules), FEAT-28.SPEC-004 (Search Result Relevance Ranking); list filtering in FEAT-01.SPEC-003, FEAT-02.SPEC-004, FEAT-09.SPEC-001, FEAT-12.SPEC-001 to FEAT-12.SPEC-004, FEAT-29.SPEC-001, FEAT-30.SPEC-002. Keywords "full-text" and "faceted" match no spec; FEAT-28 is Nice-to-Have, phase v1 |
| Payments/billing | Yes | Spec IDs: FEAT-10.SPEC-001, FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing, Integration), FEAT-10.SPEC-004, FEAT-10.SPEC-005, FEAT-09.SPEC-001 to FEAT-09.SPEC-010 (invoicing), FEAT-11.SPEC-001 (reminders), FEAT-23.SPEC-001, FEAT-23.SPEC-003 (Subscription Billing Processing, Integration), FEAT-25.SPEC-001, FEAT-25.SPEC-003, FEAT-25.SPEC-005 (refunds, reversals), FEAT-32.SPEC-001, FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting, Integration); 117 specs match the payment keywords. BRIEF.md, ## Business Context (subscription per freelancer; no cut of payments) and ## Ecosystem & Integrations (payment processor, required); ASMP-28 and ASMP-31 (assumptions-constraints.md, Dependencies) |
| Notifications (email/push/SMS) | Yes | 26 Notification specs, channel email: FEAT-02.SPEC-011, FEAT-03.SPEC-006, FEAT-03.SPEC-007, FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, FEAT-08.SPEC-007, FEAT-09.SPEC-010, FEAT-10.SPEC-007, FEAT-11.SPEC-004, FEAT-14.SPEC-006, FEAT-18.SPEC-010, FEAT-18.SPEC-011, FEAT-20.SPEC-006, FEAT-21.SPEC-011, FEAT-23.SPEC-008, FEAT-24.SPEC-008, FEAT-24.SPEC-009, FEAT-25.SPEC-007, FEAT-25.SPEC-008, FEAT-26.SPEC-004, FEAT-27.SPEC-004, FEAT-31.SPEC-006, FEAT-31.SPEC-007, FEAT-32.SPEC-006; plus FEAT-14.SPEC-001 (Transactional Email Delivery, Integration) and FEAT-14.SPEC-002 (Notification Composition & Dispatch). BRIEF.md, ## Ecosystem & Integrations (Email, required; "all notifications ... go by email"); ASMP-29 (assumptions-constraints.md, Dependencies). Push and SMS: no spec names either channel |
| Third-party integrations | Yes | 7 Integration specs: FEAT-10.SPEC-003 (payment processing), FEAT-14.SPEC-001 (transactional email delivery), FEAT-16.SPEC-007 (large-file storage and delivery), FEAT-23.SPEC-003 (subscription billing), FEAT-26.SPEC-005 (electronic-signature attestation), FEAT-27.SPEC-002 (domain verification and secure serving), FEAT-32.SPEC-002 (payment account connection and status). feature-dependency-map.md External Touchpoints: 6 capability-category rows (payment processing; transactional email; large-file storage; subscription billing; domain verification; e-signature attestation). BRIEF.md, ## Ecosystem & Integrations (payment processor, email, accounting export file, Figma/Google Drive/Dropbox by link); ASMP-28 to ASMP-32 (assumptions-constraints.md, Dependencies); linked-asset reachability check FEAT-06.SPEC-004 |
| AI/ML behavior | No | No specs reference AI or machine-learning behavior; the keyword "classification" matches only the paid/due/overdue invoice status classification (FEAT-12.SPEC-001 to FEAT-12.SPEC-004) and FEAT-28.SPEC-004 covers rule-based search result ranking. BRIEF.md and assumptions-constraints.md Dependencies name no AI capability |
| Geo/maps | No | No specs reference location or mapping behavior; keywords "geolocation", "GPS", "proximity", "geocoding" match nothing in specs, BRIEF.md, or assumptions-constraints.md. The keyword "address" matches billing and email address fields only (e.g. FEAT-01.SPEC-001 billing address text input) |
| Import/export | Yes | FEAT-22.SPEC-001 (Accounting Export Screen), FEAT-22.SPEC-002 (Export File Generation), FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules), FEAT-24.SPEC-001 (Data Export Screen), FEAT-24.SPEC-003 (Data Export Archive Generation), FEAT-24.SPEC-008 (Export Ready Notification), FEAT-13.SPEC-002 (Printable Record Copy); no bulk import spec. BRIEF.md, ## Ecosystem & Integrations (accounting export file, CSV or QuickBooks/Xero-compatible format; no live sync) |
| Collaboration/concurrency | Yes | Reject-with-refresh and last-write-wins resolution appear in 97 specs by keyword match; primary: FEAT-03.SPEC-003 (Acceptance Recording, exactly-once), FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard), FEAT-10.SPEC-004, FEAT-10.SPEC-005, FEAT-11.SPEC-001, FEAT-11.SPEC-002, FEAT-18.SPEC-006, FEAT-01.SPEC-008. feature-dependency-map.md Contention lines with a non-"None" value: 12 of the 21 entities (Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Invoice, Payment, Reminder Log, Subscription Plan, Payment Account Connection); Contention reads "None" for 9 (Deliverable Version, Comment, Activity Log Entry, Branding Profile, Custom Domain Record, Freelancer Account, Notification, Support Access Session, Referral Attribution). Multi-user contention is between one freelancer and client contacts on the same record, not a shared-editing workspace |
| Compliance/privacy | Yes | FEAT-13.SPEC-003, FEAT-13.SPEC-004, FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule), FEAT-18.SPEC-002, FEAT-18.SPEC-003, FEAT-18.SPEC-009 (Contact Removal & Data Erasure), FEAT-24.SPEC-004 (Account Deletion Processing), FEAT-24.SPEC-006, FEAT-05.SPEC-007 (Portal Access & Isolation Rules), FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules), FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules), FEAT-30.SPEC-004, FEAT-30.SPEC-005; 15 specs match GDPR / personal data / erasure and 48 match retention / immutable. Data Sensitivity lines (feature-dependency-map.md) name GDPR-class personal data on Client, Client Contact, Proposal, Milestone, Comment, Invoice, Payment, Activity Log Entry, Freelancer Account, Notification; ASMP-23, ASMP-24, ASMP-25 (assumptions-constraints.md, Non-Functional Expectations); BRIEF.md, ## Scale & Non-Functional Expectations (Correctness & records; Privacy) |
| Internationalization | Yes | FEAT-15.SPEC-001 (Project Currency & Tax Configuration), FEAT-15.SPEC-002 (Freelancer Time Zone Setting), FEAT-15.SPEC-003, FEAT-15.SPEC-004, FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule), FEAT-15.SPEC-007 (Multi-Currency Dashboard Non-Aggregation Rule), FEAT-15.SPEC-008 (Invoice Currency & Tax Line Application), FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules), FEAT-09.SPEC-007; 82 specs match currency / time zone / VAT / GST keywords. Multi-language and translation: no spec references either (English only at launch). BRIEF.md, ## Scale & Non-Functional Expectations (Geography: worldwide; currencies, tax, time zones; English only) |
| Scale hints | Yes | ASMP-22 (a few thousand freelancers in year one, 3-15 active clients each, files tens of MB to over 1 GB, version history for the life of the account); ASMP-21 (interactive within roughly 2 seconds on mobile); ASMP-26 (no uptime target); BRIEF.md, ## Scale & Non-Functional Expectations (Users year one; Data; Availability); BRIEF.md, ## Constraints (Budget: infrastructure under roughly $100/month); FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules), FEAT-16.SPEC-005 (Storage Usage Aggregation), FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules), FEAT-01.SPEC-008 (Active Client Limit Enforcement) |

## 4. Derived Classifications

Three-axis classification based on metrics from Sections 1-3. Each axis classified as Small, Medium, or Large using the threshold definitions below.

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
| Scale | Large | 33 features (25+ threshold), 220 specs (201+ threshold) |
| Data Complexity | Medium | 23 entities (11-30 band), 60 named inter-entity relationships, 35 cross-feature business rules |
| Interaction Complexity | Large | 62 Automation specs (16+ threshold), 7 Integration specs, 26 Notification specs; Collaboration/concurrency signal present (Section 3); Real-time signal absent |

**Summary:** Large / Medium / Large

(Blueprint scope only -- the deployment-scale evidence for this product lives in Section 7.)

## 5. Entity Inventory

Complete listing of all 23 entities from the Domain Entity Inventory in product-features.md (Managing Feature from the "Managed by" line; "Created by" where Managed by is N/A), enriched with the Shared Data Entities detail from feature-dependency-map.md where an entity appears there. Field Count is the number of "Fields (functional)" bullets; entities absent from the Shared Data Entities section carry "--".

| Entity Name | Managing Feature | Field Count | Relationships |
|-------------|-----------------|-------------|---------------|
| Client | FEAT-01 (Client & Project Management) | 6 | Freelancer Account (belongs to one), Project (has many), Client Contact (has many), Subscription Plan (counts toward active-client limit while Active) |
| Client Contact | FEAT-18 (Client Contact Management & Roles) | 7 | Client (belongs to one); actor on acceptances (FEAT-03), approvals (FEAT-08), comments (FEAT-07), payments (FEAT-10); a person who is a contact for several freelancers holds a separate Client Contact per freelancer |
| Project | FEAT-01 (Client & Project Management), FEAT-25 (Refund & Cancelled Project Handling) | 6 | Client (belongs to one); at most one active Proposal, one Payment Schedule; many Milestones, Deliverables, Invoices, Activity Log Entries |
| Proposal | FEAT-02 (Proposal Creation & Sending), FEAT-03 (Proposal Acceptance), FEAT-26 (Legally Binding E-Signature for Proposals) | 8 | Project (belongs to one); sent to the client's Primary contact(s); referenced by Payment Schedule; carries request-changes Comments |
| Milestone | FEAT-04 (Milestone & Payment Schedule Setup), FEAT-08 (Milestone Approval) | 7 | Project (belongs to one); governed by Payment Schedule; many Deliverables and milestone-level Comments; approval creates the next Invoice |
| Payment Schedule | FEAT-04 (Milestone & Payment Schedule Setup) | 4 | Project (one per Project); attached to the accepted Proposal; drives Invoice Generation through its triggers |
| Deliverable | FEAT-06 (Deliverable Upload & Sharing), FEAT-16 (Large File Handling & Storage) | 6 | Milestone (belongs to one); one or more Deliverable Versions; many pinned Comments |
| Deliverable Version | FEAT-17 (Deliverable Version History) | 4 | Deliverable (belongs to one); anchors version-specific Comments; counts against the freelancer's storage allowance (FEAT-16) |
| Comment | FEAT-07 (Deliverable Review & Feedback) | 6 | Pinned to a Deliverable Version, a Milestone, or a Proposal; visible to all contacts of the same client (except proposal notes, which Reviewers cannot see) and to the freelancer |
| Invoice | FEAT-09 (Invoice Generation & Sending), FEAT-10 (Invoice Payment Processing), FEAT-11 (Automated Payment Reminders), FEAT-25 (Refund & Cancelled Project Handling) | 8 | Project (belongs to one); addressed to the client's Primary contact(s); at most one successful Payment; has a Reminder Log; may be corrected by a credit note Invoice |
| Payment | FEAT-10 (Invoice Payment Processing), FEAT-25 (Refund & Cancelled Project Handling, reversals) | 6 | Invoice (belongs to one); lands in the freelancer's own processor account via Payment Account Connection |
| Reminder Log | FEAT-11 (Automated Payment Reminders) | 4 | Invoice (belongs to one); each send writes an Activity Log Entry and a Notification |
| Activity Log Entry | FEAT-13 (Immutable Activity & Audit Trail) | 5 | Freelancer Account (belongs to one) and, usually, one Project; references the record it describes |
| Branding Profile | FEAT-19 (Freelancer Branding) | 2 | Freelancer Account (one per account) |
| Subscription Plan | FEAT-23 (Subscription Plan & Billing Management) | 4 | Freelancer Account (one per account); limits how many Active Clients FEAT-01 allows |
| Accounting Export File | FEAT-22 (Accounting Export) | -- | -- |
| Custom Domain Record | FEAT-27 (Custom Domain per Freelancer) | 2 | Freelancer Account (one per account at most) |
| Freelancer Account | FEAT-21 (Settings & Account Management), FEAT-24 (Data Export & Account Deletion) | 7 | Owns every Client, Project, Branding Profile, Subscription Plan, Payment Account Connection, Referral Attribution, and Activity Log Entry in the account |
| Notification | FEAT-14 (Notifications (Email)) | 4 | References its triggering event and the affected project |
| Payment Account Connection | FEAT-32 (Payment Account Connection) | 3 | Freelancer Account (one per account); every Invoice's pay link and in-portal Payment depends on it |
| Support Access Session | FEAT-31 (Operator Support Access) | 4 | Freelancer Account (belongs to one); writes Activity Log Entries and triggers Notifications |
| Referral Attribution | FEAT-33 (Portal Referral Attribution) | 3 | Freelancer Account (belongs to the new account); references the referring Freelancer Account |
| Data Export Archive | FEAT-24 (Data Export & Account Deletion) | -- | -- |

## 6. Raw Spec Index

Complete listing of all 220 specs compiled from the Spec Inventory tables of all 33 feature-overview.md files, in FEAT-NN then SPEC-NNN order. Key Signals lists the Section 3 signals whose keyword (or, for Type-defined signals, whose spec type) matches that spec; keyword matches are raw grep hits ("--" where none).

| Spec ID | Name | Type | Feature | Key Signals |
|---------|------|------|---------|-------------|
| FEAT-01.SPEC-001 | Add Client | Screen | FEAT-01 (Client & Project Management) | Authentication; Payments/billing; Internationalization |
| FEAT-01.SPEC-002 | Create Project | Screen | FEAT-01 (Client & Project Management) | Authentication; Payments/billing; Internationalization |
| FEAT-01.SPEC-003 | Client & Project Roster | Screen | FEAT-01 (Client & Project Management) | Authentication; Search |
| FEAT-01.SPEC-004 | Client Detail | Screen | FEAT-01 (Client & Project Management) | Authentication; Search; Internationalization; Collaboration/concurrency |
| FEAT-01.SPEC-005 | Project Detail (Open Project) | Screen | FEAT-01 (Client & Project Management) | Authentication; Search; File upload; Collaboration/concurrency |
| FEAT-01.SPEC-006 | Completion Invoice Trigger | Automation | FEAT-01 (Client & Project Management) | Internationalization; Collaboration/concurrency |
| FEAT-01.SPEC-007 | Archive Open-Items Check | Automation | FEAT-01 (Client & Project Management) | Payments/billing; Collaboration/concurrency |
| FEAT-01.SPEC-008 | Active Client Limit Enforcement | Logic/Rule | FEAT-01 (Client & Project Management) | Internationalization; Collaboration/concurrency |
| FEAT-01.SPEC-009 | Client Delete Eligibility | Logic/Rule | FEAT-01 (Client & Project Management) | Internationalization |
| FEAT-01.SPEC-010 | Client Billing Completeness Gate | Logic/Rule | FEAT-01 (Client & Project Management) | Internationalization; Collaboration/concurrency |
| FEAT-01.SPEC-011 | Project Stage Derivation | Logic/Rule | FEAT-01 (Client & Project Management) | Internationalization |
| FEAT-02.SPEC-001 | Proposal Draft Editor | Screen | FEAT-02 (Proposal Creation & Sending) | Authentication; Payments/billing; Internationalization |
| FEAT-02.SPEC-002 | Proposal Preview | Screen | FEAT-02 (Proposal Creation & Sending) | Authentication; Internationalization |
| FEAT-02.SPEC-003 | Proposal Detail | Screen | FEAT-02 (Proposal Creation & Sending) | Authentication; Search; Payments/billing; Internationalization |
| FEAT-02.SPEC-004 | Reuse Proposal Picker | Screen | FEAT-02 (Proposal Creation & Sending) | Authentication; Search; Payments/billing; Internationalization |
| FEAT-02.SPEC-005 | Proposal Send | Automation | FEAT-02 (Proposal Creation & Sending) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-02.SPEC-006 | Proposal Edit-Before-Acceptance Void & Resend | Automation | FEAT-02 (Proposal Creation & Sending) | Internationalization; Collaboration/concurrency |
| FEAT-02.SPEC-007 | Proposal Resend | Automation | FEAT-02 (Proposal Creation & Sending) | Payments/billing; Internationalization |
| FEAT-02.SPEC-008 | Create Draft From Copy | Automation | FEAT-02 (Proposal Creation & Sending) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-02.SPEC-009 | Discard Draft | Automation | FEAT-02 (Proposal Creation & Sending) | Payments/billing; Collaboration/concurrency |
| FEAT-02.SPEC-010 | Proposal Validation & Business Rules | Logic/Rule | FEAT-02 (Proposal Creation & Sending) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-02.SPEC-011 | Proposal Sent/Resent Email | Notification | FEAT-02 (Proposal Creation & Sending) | Authentication; Payments/billing; Internationalization; Notifications (email/push/SMS) |
| FEAT-03.SPEC-001 | Proposal Review & Accept | Screen | FEAT-03 (Proposal Acceptance) | Authentication; Internationalization; Collaboration/concurrency |
| FEAT-03.SPEC-002 | Request Changes | Screen | FEAT-03 (Proposal Acceptance) | Authentication; Payments/billing; Collaboration/concurrency |
| FEAT-03.SPEC-003 | Acceptance Recording | Automation | FEAT-03 (Proposal Acceptance) | Collaboration/concurrency |
| FEAT-03.SPEC-004 | Change-Request Recording | Automation | FEAT-03 (Proposal Acceptance) | Collaboration/concurrency |
| FEAT-03.SPEC-005 | Acceptance & Access Rules | Logic/Rule | FEAT-03 (Proposal Acceptance) | Collaboration/concurrency |
| FEAT-03.SPEC-006 | Acceptance Confirmation Notification | Notification | FEAT-03 (Proposal Acceptance) | Internationalization; Notifications (email/push/SMS) |
| FEAT-03.SPEC-007 | Change-Request Notification | Notification | FEAT-03 (Proposal Acceptance) | Notifications (email/push/SMS) |
| FEAT-04.SPEC-001 | Milestone & Payment Schedule Editor | Screen | FEAT-04 (Milestone & Payment Schedule Setup) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-04.SPEC-002 | Milestone Timeline (Client View) | Screen | FEAT-04 (Milestone & Payment Schedule Setup) | Authentication; Internationalization |
| FEAT-04.SPEC-003 | Milestone & Schedule Validation and Edit Rules | Logic/Rule | FEAT-04 (Milestone & Payment Schedule Setup) | Internationalization; Collaboration/concurrency |
| FEAT-04.SPEC-004 | Milestone Reorder Recalculation | Automation | FEAT-04 (Milestone & Payment Schedule Setup) | -- |
| FEAT-05.SPEC-001 | Request Sign-In Link | Screen | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication; Collaboration/concurrency |
| FEAT-05.SPEC-002 | Link Verification Landing | Screen | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication; Collaboration/concurrency |
| FEAT-05.SPEC-003 | Portal Home | Screen | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-05.SPEC-004 | Magic Link Issuance | Automation | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication |
| FEAT-05.SPEC-005 | Magic Link Verification | Automation | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication |
| FEAT-05.SPEC-006 | Link Validity & Recognition Rules | Logic/Rule | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication; Internationalization |
| FEAT-05.SPEC-007 | Portal Access & Isolation Rules | Logic/Rule | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication; Search |
| FEAT-05.SPEC-008 | Magic Link Sign-In Email | Notification | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication; Notifications (email/push/SMS) |
| FEAT-05.SPEC-009 | Portal Record First-View Capture | Automation | FEAT-05 (Client Portal Access (Magic-Link Login)) | Authentication |
| FEAT-06.SPEC-001 | Deliverable Upload | Screen | FEAT-06 (Deliverable Upload & Sharing) | Authentication; File upload; Payments/billing |
| FEAT-06.SPEC-002 | Deliverable List & Management | Screen | FEAT-06 (Deliverable Upload & Sharing) | Authentication; Search; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-06.SPEC-003 | Resumable Upload Handling | Automation | FEAT-06 (Deliverable Upload & Sharing) | File upload; Payments/billing; Collaboration/concurrency |
| FEAT-06.SPEC-004 | Linked Asset Reachability Check | Automation | FEAT-06 (Deliverable Upload & Sharing) | Authentication; Background processing; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-06.SPEC-005 | Deliverable Validation & Removal Eligibility Rules | Logic/Rule | FEAT-06 (Deliverable Upload & Sharing) | File upload; Collaboration/concurrency |
| FEAT-06.SPEC-006 | Deliverable Ready Notification | Notification | FEAT-06 (Deliverable Upload & Sharing) | Authentication; File upload; Notifications (email/push/SMS) |
| FEAT-07.SPEC-001 | Deliverable Comment Thread | Screen | FEAT-07 (Deliverable Review & Feedback) | Authentication; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-07.SPEC-002 | Milestone Comment Thread | Screen | FEAT-07 (Deliverable Review & Feedback) | Authentication; Payments/billing |
| FEAT-07.SPEC-003 | Client Comment Alert to Freelancer | Notification | FEAT-07 (Deliverable Review & Feedback) | File upload; Notifications (email/push/SMS) |
| FEAT-07.SPEC-004 | Freelancer Reply Alert to Client | Notification | FEAT-07 (Deliverable Review & Feedback) | File upload; Notifications (email/push/SMS) |
| FEAT-07.SPEC-005 | Comment Content & Submission Validation | Logic/Rule | FEAT-07 (Deliverable Review & Feedback) | Search |
| FEAT-07.SPEC-006 | Comment Edit Window & Retraction Rule | Logic/Rule | FEAT-07 (Deliverable Review & Feedback) | Import/export; Collaboration/concurrency |
| FEAT-07.SPEC-007 | Comment Visibility & Authorization Rule | Logic/Rule | FEAT-07 (Deliverable Review & Feedback) | -- |
| FEAT-07.SPEC-008 | Offline Comment Queue & Sync | Automation | FEAT-07 (Deliverable Review & Feedback) | File upload; Payments/billing; Collaboration/concurrency; Offline |
| FEAT-08.SPEC-001 | Milestone Review & Approval Screen | Screen | FEAT-08 (Milestone Approval) | Authentication; File upload; Internationalization; Collaboration/concurrency |
| FEAT-08.SPEC-002 | Milestone Reopen Screen | Screen | FEAT-08 (Milestone Approval) | Authentication; Payments/billing |
| FEAT-08.SPEC-003 | Approval Recording & Concurrency Guard | Automation | FEAT-08 (Milestone Approval) | Internationalization; Collaboration/concurrency |
| FEAT-08.SPEC-004 | Next-Invoice Trigger | Automation | FEAT-08 (Milestone Approval) | Internationalization; Collaboration/concurrency |
| FEAT-08.SPEC-005 | Reopen Recording | Automation | FEAT-08 (Milestone Approval) | -- |
| FEAT-08.SPEC-006 | Approval Authorization & Eligibility Rules | Logic/Rule | FEAT-08 (Milestone Approval) | Internationalization; Collaboration/concurrency |
| FEAT-08.SPEC-007 | Milestone Approval Confirmation Notification | Notification | FEAT-08 (Milestone Approval) | Internationalization; Notifications (email/push/SMS) |
| FEAT-09.SPEC-001 | Invoice List | Screen | FEAT-09 (Invoice Generation & Sending) | Authentication; Search; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-09.SPEC-002 | Invoice Detail | Screen | FEAT-09 (Invoice Generation & Sending) | Authentication; Search; Background processing; Payments/billing; Internationalization |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | Screen | FEAT-09 (Invoice Generation & Sending) | Authentication; Search; Background processing; Payments/billing; Internationalization |
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Automation | FEAT-09 (Invoice Generation & Sending) | Search; Internationalization; Collaboration/concurrency |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | Automation | FEAT-09 (Invoice Generation & Sending) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-09.SPEC-006 | Invoice Access & Role Authorization Rules | Logic/Rule | FEAT-09 (Invoice Generation & Sending) | Payments/billing; Internationalization |
| FEAT-09.SPEC-007 | Invoice Content, Numbering, Amount & Due-Date Rules | Logic/Rule | FEAT-09 (Invoice Generation & Sending) | Internationalization |
| FEAT-09.SPEC-008 | Invoice Immutability & Correction Rules | Logic/Rule | FEAT-09 (Invoice Generation & Sending) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-09.SPEC-009 | Pay-Link Availability & No-Account Fallback Rule | Logic/Rule | FEAT-09 (Invoice Generation & Sending) | -- |
| FEAT-09.SPEC-010 | Invoice Issued & Copy Confirmation Notification | Notification | FEAT-09 (Invoice Generation & Sending) | Internationalization; Notifications (email/push/SMS) |
| FEAT-10.SPEC-001 | Pay Invoice Screen | Screen | FEAT-10 (Invoice Payment Processing) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-10.SPEC-002 | Record Off-Platform Payment Screen | Screen | FEAT-10 (Invoice Payment Processing) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-10.SPEC-003 | Card & Bank-Transfer Payment Processing | Integration | FEAT-10 (Invoice Payment Processing) | Payments/billing; Internationalization; Third-party integrations |
| FEAT-10.SPEC-004 | Payment Confirmation & Invoice Status Sync | Automation | FEAT-10 (Invoice Payment Processing) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-10.SPEC-005 | Record Off-Platform Payment | Automation | FEAT-10 (Invoice Payment Processing) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-10.SPEC-006 | Payment Authorization & Validation Rules | Logic/Rule | FEAT-10 (Invoice Payment Processing) | Search; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-10.SPEC-007 | Payment Confirmation Notification | Notification | FEAT-10 (Invoice Payment Processing) | Payments/billing; Internationalization; Notifications (email/push/SMS) |
| FEAT-11.SPEC-001 | Reminder Schedule | Automation | FEAT-11 (Automated Payment Reminders) | Background processing; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-11.SPEC-002 | Reminder Eligibility Rule | Logic/Rule | FEAT-11 (Automated Payment Reminders) | Background processing; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-11.SPEC-003 | Invoice Reminder Panel | Screen | FEAT-11 (Automated Payment Reminders) | Authentication; Background processing; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-11.SPEC-004 | Overdue Reminder Email | Notification | FEAT-11 (Automated Payment Reminders) | Payments/billing; Compliance/privacy; Internationalization; Notifications (email/push/SMS) |
| FEAT-11.SPEC-005 | Bank-Transfer-Pending Reminder Pause | Automation | FEAT-11 (Automated Payment Reminders) | Background processing; Payments/billing |
| FEAT-12.SPEC-001 | Dashboard Overview | Screen | FEAT-12 (Freelancer Financial Dashboard) | Authentication; Search; Payments/billing; Import/export; Internationalization |
| FEAT-12.SPEC-002 | Client/Project Financial Drill-down | Screen | FEAT-12 (Freelancer Financial Dashboard) | Authentication; Search; Payments/billing; Import/export; Internationalization |
| FEAT-12.SPEC-003 | Financial Totals Aggregation | Logic/Rule | FEAT-12 (Freelancer Financial Dashboard) | Search; Payments/billing; Import/export; Internationalization |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | Automation | FEAT-12 (Freelancer Financial Dashboard) | Search; Background processing; Payments/billing; Import/export; Internationalization |
| FEAT-13.SPEC-001 | Activity Trail | Screen | FEAT-13 (Immutable Activity & Audit Trail) | Authentication; File upload; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-13.SPEC-002 | Printable Record Copy | Screen | FEAT-13 (Immutable Activity & Audit Trail) | Authentication; Import/export |
| FEAT-13.SPEC-003 | Activity Entry Recording | Automation | FEAT-13 (Immutable Activity & Audit Trail) | File upload; Payments/billing; Compliance/privacy |
| FEAT-13.SPEC-004 | Entry Immutability, Content & Attribution Rules | Logic/Rule | FEAT-13 (Immutable Activity & Audit Trail) | File upload; Payments/billing; Compliance/privacy |
| FEAT-13.SPEC-005 | Activity Trail Access & Visibility Rules | Logic/Rule | FEAT-13 (Immutable Activity & Audit Trail) | -- |
| FEAT-13.SPEC-006 | Retention & Account-Deletion Purge Rule | Logic/Rule | FEAT-13 (Immutable Activity & Audit Trail) | Search; Payments/billing; Import/export; Compliance/privacy |
| FEAT-14.SPEC-001 | Transactional Email Delivery | Integration | FEAT-14 (Notifications (Email)) | Authentication; Payments/billing; Third-party integrations |
| FEAT-14.SPEC-002 | Notification Composition & Dispatch | Automation | FEAT-14 (Notifications (Email)) | Authentication; Search; File upload; Payments/billing; Import/export; Internationalization |
| FEAT-14.SPEC-003 | Delivery Status Tracking & Retry | Automation | FEAT-14 (Notifications (Email)) | Background processing |
| FEAT-14.SPEC-004 | Notification Type & Recipient Entitlement Rules | Logic/Rule | FEAT-14 (Notifications (Email)) | Authentication; Payments/billing; Import/export |
| FEAT-14.SPEC-005 | Recognizable & Branded Email Presentation Rules | Logic/Rule | FEAT-14 (Notifications (Email)) | Collaboration/concurrency |
| FEAT-14.SPEC-006 | Delivery Failure Warning to Freelancer | Notification | FEAT-14 (Notifications (Email)) | Notifications (email/push/SMS) |
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | Screen | FEAT-15 (Currency & Tax Handling) | Authentication; Search; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-15.SPEC-002 | Freelancer Time Zone Setting | Screen | FEAT-15 (Currency & Tax Handling) | Authentication; Search; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-15.SPEC-003 | Currency & Tax Validation Rules | Logic/Rule | FEAT-15 (Currency & Tax Handling) | Internationalization |
| FEAT-15.SPEC-004 | Currency & Tax Lock After First Invoice | Logic/Rule | FEAT-15 (Currency & Tax Handling) | Internationalization |
| FEAT-15.SPEC-005 | Currency & Tax Configuration Access Rules | Logic/Rule | FEAT-15 (Currency & Tax Handling) | Internationalization; Collaboration/concurrency |
| FEAT-15.SPEC-006 | Time Zone & Local Date/Time Display Rule | Logic/Rule | FEAT-15 (Currency & Tax Handling) | Background processing; Internationalization |
| FEAT-15.SPEC-007 | Multi-Currency Dashboard Non-Aggregation Rule | Logic/Rule | FEAT-15 (Currency & Tax Handling) | Payments/billing; Import/export; Internationalization |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | Automation | FEAT-15 (Currency & Tax Handling) | Internationalization; Collaboration/concurrency |
| FEAT-16.SPEC-001 | Storage Usage Summary | Screen | FEAT-16 (Large File Handling & Storage) | Authentication; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-16.SPEC-002 | Resumable Upload Transfer | Automation | FEAT-16 (Large File Handling & Storage) | File upload; Payments/billing |
| FEAT-16.SPEC-003 | Reliable File Delivery | Automation | FEAT-16 (Large File Handling & Storage) | File upload; Import/export |
| FEAT-16.SPEC-004 | Storage Limit & Size Ceiling Rules | Logic/Rule | FEAT-16 (Large File Handling & Storage) | File upload |
| FEAT-16.SPEC-005 | Storage Usage Aggregation | Automation | FEAT-16 (Large File Handling & Storage) | Background processing; File upload |
| FEAT-16.SPEC-006 | Stored File Purge on Account Deletion | Automation | FEAT-16 (Large File Handling & Storage) | File upload; Collaboration/concurrency |
| FEAT-16.SPEC-007 | Large-File Storage & Delivery Capability | Integration | FEAT-16 (Large File Handling & Storage) | File upload; Payments/billing; Import/export; Third-party integrations |
| FEAT-17.SPEC-001 | New Version Upload | Screen | FEAT-17 (Deliverable Version History) | Authentication; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Screen | FEAT-17 (Deliverable Version History) | Authentication; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Automation | FEAT-17 (Deliverable Version History) | File upload; Payments/billing; Collaboration/concurrency |
| FEAT-17.SPEC-004 | Version Numbering, Immutability & Retention Rules | Logic/Rule | FEAT-17 (Deliverable Version History) | Search; File upload |
| FEAT-17.SPEC-005 | Version Access & Comment-Anchoring Rules | Logic/Rule | FEAT-17 (Deliverable Version History) | File upload; Collaboration/concurrency |
| FEAT-18.SPEC-001 | Client Contact List | Screen | FEAT-18 (Client Contact Management & Roles) | Authentication |
| FEAT-18.SPEC-002 | Add or Edit Client Contact | Screen | FEAT-18 (Client Contact Management & Roles) | Authentication; Payments/billing; Compliance/privacy; Collaboration/concurrency |
| FEAT-18.SPEC-003 | Remove Client Contact | Screen | FEAT-18 (Client Contact Management & Roles) | Authentication; Compliance/privacy; Collaboration/concurrency |
| FEAT-18.SPEC-004 | Invite Reviewer Colleague | Screen | FEAT-18 (Client Contact Management & Roles) | Authentication; Payments/billing; Collaboration/concurrency |
| FEAT-18.SPEC-005 | Contact Field Validation Rules | Logic/Rule | FEAT-18 (Client Contact Management & Roles) | Authentication |
| FEAT-18.SPEC-006 | Primary Contact Requirement Rule | Logic/Rule | FEAT-18 (Client Contact Management & Roles) | Collaboration/concurrency |
| FEAT-18.SPEC-007 | Role Authorization Rules | Logic/Rule | FEAT-18 (Client Contact Management & Roles) | Collaboration/concurrency |
| FEAT-18.SPEC-008 | Role Change Effective-Timing Rule | Logic/Rule | FEAT-18 (Client Contact Management & Roles) | Collaboration/concurrency |
| FEAT-18.SPEC-009 | Contact Removal & Data Erasure | Automation | FEAT-18 (Client Contact Management & Roles) | Authentication; Compliance/privacy; Collaboration/concurrency |
| FEAT-18.SPEC-010 | New Contact Invitation Email | Notification | FEAT-18 (Client Contact Management & Roles) | Authentication; Notifications (email/push/SMS) |
| FEAT-18.SPEC-011 | Primary-Invited-Colleague Alert | Notification | FEAT-18 (Client Contact Management & Roles) | Notifications (email/push/SMS) |
| FEAT-19.SPEC-001 | Branding Settings | Screen | FEAT-19 (Freelancer Branding) | Authentication; File upload; Payments/billing; Collaboration/concurrency |
| FEAT-19.SPEC-002 | Branding Upload & Legibility Validation Rules | Logic/Rule | FEAT-19 (Freelancer Branding) | File upload; Payments/billing |
| FEAT-19.SPEC-003 | Branding Application & Fallback Rule | Logic/Rule | FEAT-19 (Freelancer Branding) | -- |
| FEAT-20.SPEC-001 | Sign-Up & Account Creation | Screen | FEAT-20 (Onboarding / First-Run Setup) | Authentication; Search; Payments/billing; Internationalization |
| FEAT-20.SPEC-002 | Onboarding Guided Sequence | Screen | FEAT-20 (Onboarding / First-Run Setup) | Authentication; File upload; Collaboration/concurrency |
| FEAT-20.SPEC-003 | Onboarding Completion Detection | Automation | FEAT-20 (Onboarding / First-Run Setup) | -- |
| FEAT-20.SPEC-004 | Referral Attribution Capture Hand-off | Automation | FEAT-20 (Onboarding / First-Run Setup) | -- |
| FEAT-20.SPEC-005 | Onboarding Step Sequencing & Exit-Criteria Rules | Logic/Rule | FEAT-20 (Onboarding / First-Run Setup) | Authentication; File upload |
| FEAT-20.SPEC-006 | Welcome Email | Notification | FEAT-20 (Onboarding / First-Run Setup) | Authentication; Notifications (email/push/SMS) |
| FEAT-21.SPEC-001 | Account Profile | Screen | FEAT-21 (Settings & Account Management) | Authentication; Payments/billing; Collaboration/concurrency |
| FEAT-21.SPEC-002 | Notification Preferences | Screen | FEAT-21 (Settings & Account Management) | Authentication; Collaboration/concurrency |
| FEAT-21.SPEC-003 | Login & Security | Screen | FEAT-21 (Settings & Account Management) | Authentication; Compliance/privacy |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | Screen | FEAT-21 (Settings & Account Management) | Authentication; Payments/billing; Collaboration/concurrency |
| FEAT-21.SPEC-005 | Sign-In Email & Login Method Change | Automation | FEAT-21 (Settings & Account Management) | Authentication; Background processing |
| FEAT-21.SPEC-006 | Sign-Out Other Sessions | Automation | FEAT-21 (Settings & Account Management) | Authentication |
| FEAT-21.SPEC-007 | Account Field Validation Rules | Logic/Rule | FEAT-21 (Settings & Account Management) | Authentication; Collaboration/concurrency |
| FEAT-21.SPEC-008 | Notification Preference Rules | Logic/Rule | FEAT-21 (Settings & Account Management) | Payments/billing |
| FEAT-21.SPEC-009 | Business Details Completeness Gate | Logic/Rule | FEAT-21 (Settings & Account Management) | -- |
| FEAT-21.SPEC-010 | Settings Access & Read-Only Scope Rules | Logic/Rule | FEAT-21 (Settings & Account Management) | Authentication; Collaboration/concurrency |
| FEAT-21.SPEC-011 | Account-Critical Change Confirmation Email | Notification | FEAT-21 (Settings & Account Management) | Authentication; Payments/billing; Internationalization; Notifications (email/push/SMS) |
| FEAT-22.SPEC-001 | Accounting Export Screen | Screen | FEAT-22 (Accounting Export) | Authentication; Payments/billing; Import/export; Internationalization; Collaboration/concurrency |
| FEAT-22.SPEC-002 | Export File Generation | Automation | FEAT-22 (Accounting Export) | Payments/billing; Import/export; Internationalization; Collaboration/concurrency |
| FEAT-22.SPEC-003 | Export Scope, Authorization & Currency Rules | Logic/Rule | FEAT-22 (Accounting Export) | Payments/billing; Import/export; Internationalization |
| FEAT-23.SPEC-001 | Plan & Billing Screen | Screen | FEAT-23 (Subscription Plan & Billing Management) | Authentication; Payments/billing; Collaboration/concurrency |
| FEAT-23.SPEC-002 | Free Plan Auto-Provisioning | Automation | FEAT-23 (Subscription Plan & Billing Management) | Collaboration/concurrency |
| FEAT-23.SPEC-003 | Subscription Billing Processing | Integration | FEAT-23 (Subscription Plan & Billing Management) | Background processing; Payments/billing; Third-party integrations |
| FEAT-23.SPEC-004 | Plan State Sync | Automation | FEAT-23 (Subscription Plan & Billing Management) | Payments/billing; Collaboration/concurrency |
| FEAT-23.SPEC-005 | Downgrade Eligibility Detection | Automation | FEAT-23 (Subscription Plan & Billing Management) | Collaboration/concurrency |
| FEAT-23.SPEC-006 | Cancel Subscription | Automation | FEAT-23 (Subscription Plan & Billing Management) | Payments/billing |
| FEAT-23.SPEC-007 | Plan Limit & Access Authorization Rules | Logic/Rule | FEAT-23 (Subscription Plan & Billing Management) | Payments/billing; Collaboration/concurrency |
| FEAT-23.SPEC-008 | Plan & Billing Notifications | Notification | FEAT-23 (Subscription Plan & Billing Management) | Payments/billing; Notifications (email/push/SMS) |
| FEAT-24.SPEC-001 | Data Export Screen | Screen | FEAT-24 (Data Export & Account Deletion) | Authentication; Payments/billing; Import/export |
| FEAT-24.SPEC-002 | Account Deletion Screen | Screen | FEAT-24 (Data Export & Account Deletion) | Authentication; Payments/billing; Import/export; Collaboration/concurrency |
| FEAT-24.SPEC-003 | Data Export Archive Generation | Automation | FEAT-24 (Data Export & Account Deletion) | Background processing; Payments/billing; Import/export |
| FEAT-24.SPEC-004 | Account Deletion Processing | Automation | FEAT-24 (Data Export & Account Deletion) | Payments/billing; Compliance/privacy; Collaboration/concurrency |
| FEAT-24.SPEC-005 | Legal Retention Purge | Automation | FEAT-24 (Data Export & Account Deletion) | Background processing; Import/export; Collaboration/concurrency |
| FEAT-24.SPEC-006 | Pre-Deletion Warning & Retention Determination Rules | Logic/Rule | FEAT-24 (Data Export & Account Deletion) | File upload; Payments/billing; Compliance/privacy; Internationalization |
| FEAT-24.SPEC-007 | Export & Deletion Access Rules | Logic/Rule | FEAT-24 (Data Export & Account Deletion) | Import/export |
| FEAT-24.SPEC-008 | Export Ready Notification | Notification | FEAT-24 (Data Export & Account Deletion) | Authentication; Payments/billing; Import/export; Notifications (email/push/SMS) |
| FEAT-24.SPEC-009 | Account Deletion Final Warning Notification | Notification | FEAT-24 (Data Export & Account Deletion) | Authentication; Payments/billing; Internationalization; Notifications (email/push/SMS) |
| FEAT-25.SPEC-001 | Mark Invoice Refunded Screen | Screen | FEAT-25 (Refund & Cancelled Project Handling) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-25.SPEC-002 | Mark Project Cancelled Screen | Screen | FEAT-25 (Refund & Cancelled Project Handling) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-25.SPEC-003 | Refund & Partial Refund Recording | Automation | FEAT-25 (Refund & Cancelled Project Handling) | Payments/billing; Import/export; Internationalization; Collaboration/concurrency |
| FEAT-25.SPEC-004 | Project Cancellation Recording | Automation | FEAT-25 (Refund & Cancelled Project Handling) | Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-25.SPEC-005 | Payment Reversal (Chargeback) Recording | Automation | FEAT-25 (Refund & Cancelled Project Handling) | Payments/billing; Import/export; Collaboration/concurrency |
| FEAT-25.SPEC-006 | Refund, Cancellation & Reversal Authorization and Validation Rules | Logic/Rule | FEAT-25 (Refund & Cancelled Project Handling) | Search; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-25.SPEC-007 | Refund & Cancellation Notification | Notification | FEAT-25 (Refund & Cancelled Project Handling) | Payments/billing; Compliance/privacy; Internationalization; Notifications (email/push/SMS) |
| FEAT-25.SPEC-008 | Payment Reversal Notification | Notification | FEAT-25 (Refund & Cancelled Project Handling) | Authentication; Payments/billing; Internationalization; Notifications (email/push/SMS) |
| FEAT-26.SPEC-001 | Signature Signing Step | Screen | FEAT-26 (Legally Binding E-Signature for Proposals) | Authentication; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-26.SPEC-002 | Signature Recording | Automation | FEAT-26 (Legally Binding E-Signature for Proposals) | Collaboration/concurrency |
| FEAT-26.SPEC-003 | E-Signature Opt-In & Signing Access Rules | Logic/Rule | FEAT-26 (Legally Binding E-Signature for Proposals) | Collaboration/concurrency |
| FEAT-26.SPEC-004 | Signed-Copy Confirmation Notification | Notification | FEAT-26 (Legally Binding E-Signature for Proposals) | Internationalization; Notifications (email/push/SMS) |
| FEAT-26.SPEC-005 | Electronic-Signature Attestation Capability | Integration | FEAT-26 (Legally Binding E-Signature for Proposals) | Payments/billing; Third-party integrations |
| FEAT-27.SPEC-001 | Custom Domain Settings | Screen | FEAT-27 (Custom Domain per Freelancer) | Authentication; Payments/billing |
| FEAT-27.SPEC-002 | Domain Verification & Secure Serving | Integration | FEAT-27 (Custom Domain per Freelancer) | Payments/billing; Compliance/privacy; Third-party integrations |
| FEAT-27.SPEC-003 | Custom Domain Validation & Fallback Rule | Logic/Rule | FEAT-27 (Custom Domain per Freelancer) | Payments/billing; Compliance/privacy |
| FEAT-27.SPEC-004 | Custom Domain Verified Confirmation | Notification | FEAT-27 (Custom Domain per Freelancer) | Search; Payments/billing; Notifications (email/push/SMS) |
| FEAT-28.SPEC-001 | Global Search | Screen | FEAT-28 (Global Search Across Clients & Projects) | Authentication; Search; File upload; Payments/billing; Internationalization; Collaboration/concurrency |
| FEAT-28.SPEC-002 | Cross-Entity Search Execution | Automation | FEAT-28 (Global Search Across Clients & Projects) | Search; File upload; Payments/billing |
| FEAT-28.SPEC-003 | Search Scope & Access Rules | Logic/Rule | FEAT-28 (Global Search Across Clients & Projects) | Search |
| FEAT-28.SPEC-004 | Search Result Relevance Ranking | Logic/Rule | FEAT-28 (Global Search Across Clients & Projects) | Search; File upload |
| FEAT-29.SPEC-001 | Notification Center Feed | Screen | FEAT-29 (In-App Notification Center) | Authentication; Search; Payments/billing; Collaboration/concurrency |
| FEAT-29.SPEC-002 | Feed Composition & Retention Rule | Logic/Rule | FEAT-29 (In-App Notification Center) | File upload; Payments/billing |
| FEAT-29.SPEC-003 | Mark Item Read/Unread | Automation | FEAT-29 (In-App Notification Center) | Collaboration/concurrency |
| FEAT-29.SPEC-004 | Feed Load Fallback & Offline Continuity | Automation | FEAT-29 (In-App Notification Center) | Payments/billing; Offline |
| FEAT-30.SPEC-001 | Contextual Help Tooltip | Screen | FEAT-30 (Contextual Help & Guidance) | Authentication |
| FEAT-30.SPEC-002 | Freelancer Help Reference | Screen | FEAT-30 (Contextual Help & Guidance) | Authentication; Search |
| FEAT-30.SPEC-003 | Client Portal Help Reference | Screen | FEAT-30 (Contextual Help & Guidance) | Authentication |
| FEAT-30.SPEC-004 | Help Tip Dismissal Recording | Automation | FEAT-30 (Contextual Help & Guidance) | Payments/billing; Compliance/privacy |
| FEAT-30.SPEC-005 | Contextual Help Content & Behavior Rules | Logic/Rule | FEAT-30 (Contextual Help & Guidance) | Search; Compliance/privacy |
| FEAT-31.SPEC-001 | Contact Support Screen | Screen | FEAT-31 (Operator Support Access) | Authentication; Payments/billing |
| FEAT-31.SPEC-002 | Operator Support Session Console | Screen | FEAT-31 (Operator Support Access) | Authentication; Payments/billing; Import/export; Collaboration/concurrency |
| FEAT-31.SPEC-003 | Support Session Open & Read-Only Enforcement | Automation | FEAT-31 (Operator Support Access) | Import/export |
| FEAT-31.SPEC-004 | Support Session Auto-Close on Inactivity | Automation | FEAT-31 (Operator Support Access) | Collaboration/concurrency |
| FEAT-31.SPEC-005 | Support Access Authorization & Read-Only Rules | Logic/Rule | FEAT-31 (Operator Support Access) | Authentication; Payments/billing; Import/export |
| FEAT-31.SPEC-006 | Support Request Confirmation | Notification | FEAT-31 (Operator Support Access) | Authentication; Notifications (email/push/SMS) |
| FEAT-31.SPEC-007 | Support Session Opened Notice | Notification | FEAT-31 (Operator Support Access) | Internationalization; Notifications (email/push/SMS) |
| FEAT-32.SPEC-001 | Payment Connection Screen | Screen | FEAT-32 (Payment Account Connection) | Authentication; Payments/billing |
| FEAT-32.SPEC-002 | Payment Account Connection & Status Reporting | Integration | FEAT-32 (Payment Account Connection) | Authentication; Payments/billing; Third-party integrations |
| FEAT-32.SPEC-003 | Connection Status Sync | Automation | FEAT-32 (Payment Account Connection) | Payments/billing; Collaboration/concurrency |
| FEAT-32.SPEC-004 | Disconnect Payment Account | Automation | FEAT-32 (Payment Account Connection) | Payments/billing |
| FEAT-32.SPEC-005 | Payment Connection Authorization & Validation Rules | Logic/Rule | FEAT-32 (Payment Account Connection) | Payments/billing; Collaboration/concurrency |
| FEAT-32.SPEC-006 | Connection Status Notifications | Notification | FEAT-32 (Payment Account Connection) | Authentication; Payments/billing; Notifications (email/push/SMS) |
| FEAT-33.SPEC-001 | Referral Mark Display | Logic/Rule | FEAT-33 (Portal Referral Attribution) | Search; Payments/billing |
| FEAT-33.SPEC-002 | Referral Link Capture | Automation | FEAT-33 (Portal Referral Attribution) | Authentication; Payments/billing |
| FEAT-33.SPEC-003 | Referral Landing Page | Screen | FEAT-33 (Portal Referral Attribution) | Authentication; Collaboration/concurrency |
| FEAT-33.SPEC-004 | Referral Attribution Recording | Automation | FEAT-33 (Portal Referral Attribution) | -- |
| FEAT-33.SPEC-005 | Referral Data Access Restriction | Logic/Rule | FEAT-33 (Portal Referral Attribution) | Search; Import/export |

## 7. Demand-Side Inputs

Verbatim-quote evidence of what the product's world demands of the architecture: expected scale, non-functional expectations, and the external systems and capabilities the product must live alongside. Every quote is copied exactly from its source and attributed to file and section.

### From BRIEF.md -- Scale & Non-Functional Expectations

> - **Users (year one):** a few thousand freelancers, each with 3–15 active clients and a handful of contacts per client.
> - **Data:** large files are the norm: design files, videos and PDFs, typically tens of MB and sometimes over 1 GB for video, with version history per deliverable. Storage and bandwidth cost must fit the budget constraint.
> - **Devices / platforms:** web app. Freelancers work on laptop or desktop; clients mostly review on mobile browsers, so the client side must be excellent on mobile. No native apps.
> - **Geography:** worldwide from day one. Currencies, tax on invoices (VAT, GST, US sales tax) and time zones must not be hard-coded. English only at launch.
> - **Correctness & records:** payments and records must be correct. Accepted proposals, approvals and sent invoices are timestamped and never silently altered afterwards.
> - **Privacy:** strict isolation between clients. Freelancers can export and delete their data, and personal-data handling must hold up worldwide (GDPR).
> - **Availability:** no specific uptime number was stated.
>
> -- BRIEF.md, ## Scale & Non-Functional Expectations

### From BRIEF.md -- Ecosystem & Integrations

> - **Payment processor (required):** an established processor takes card and bank-transfer payments directly into each freelancer's own account. The platform never touches card numbers or holds funds.
> - **Email (required):** all notifications to clients and freelancers go by email. Clients will not install an app.
> - **Accounting software (QuickBooks, Xero):** v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync.
> - **Figma, Google Drive, Dropbox:** accepted as deliverables by link, not copied into Clientroom, for v1.
> - **Freelancer branding:** each freelancer's own logo and colours on their portal, and ideally their own custom domain (timing is an open question).
> - Otherwise the product stands alone (confirmed).
>
> -- BRIEF.md, ## Ecosystem & Integrations

### From assumptions-constraints.md -- Non-Functional Expectations

> ASMP-21: "Responsiveness: client-facing pages (deliverable review, approval, invoice payment) become interactive within roughly 2 seconds on a typical mobile connection; dashboard totals appear within roughly 1–2 seconds." — Basis: BRIEF.md, Scale & Non-Functional Expectations ("the client side must be excellent on mobile"). [RESEARCH-INFORMED: slow page loads are the single most cited complaint in negative SuiteDash reviews and are reported for HoneyBook (HIGH)]
>
> ASMP-22: "Data volume and growth: a few thousand freelancers in year one, each with 3–15 active clients and a handful of contacts per client; deliverables typically tens of MB and sometimes over 1 GB, retained with version history for the life of the account." — Basis: BRIEF.md, Scale & Non-Functional Expectations. [MODIFIED: synthesis check — "indefinitely" replaced by "for the life of the account" so it agrees with account deletion (FEAT-24) and scope-boundaries.md]
>
> ASMP-23: "Privacy posture: strict data isolation between clients (a client never sees another client's anything), freelancers can export and delete their own data on request, and any operator access is read-only and visible to the freelancer." — Basis: BRIEF.md, Privacy and Constraints; Target Users & Roles (operator support access).
>
> ASMP-24: "Compliance: personal data of freelancers and client contacts worldwide is treated as GDPR-class personal data; no card or payment data is ever captured or stored by the product itself, since that handling belongs entirely to the payment-processing capability; invoices carry the content commonly required of a valid invoice (sequential number, both parties' business details, issue and due dates, tax line)." — Basis: BRIEF.md, Privacy and Constraints; domain reasoning from the cross-cutting decomposition checklist (Compliance). [AUDIT-ADDED: 4 -- invoice-content compliance added]
>
> ASMP-25: "Correctness of financial and evidentiary records: accepted proposals, approvals, and sent invoices are timestamped at creation and never silently altered afterward, regardless of account or usage scale." — Basis: BRIEF.md, Constraints ("payments and records must be correct").
>
> ASMP-26: "Availability and delivery: no specific uptime target is committed; the product is expected to be reliably available during ordinary business use, and emails that fail to deliver are surfaced to the freelancer within minutes rather than lost." — Basis: BRIEF.md, Scale & Non-Functional Expectations ("no specific uptime number was stated"). [RESEARCH-INFORMED: messages failing to send or landing in spam are reported across four profiled products (HIGH), so delivery visibility is part of the reliability expectation]
>
> ASMP-27: "Accessibility and degraded states: client-facing screens are readable on a phone without zooming, usable with a screen reader and keyboard, never rely on colour alone, and keep brand colours legible; every screen shows real progress while loading, keeps typed input on errors, and says plainly when an action needs a connection — actions that create records (accept, approve, pay) never pretend to succeed offline." — Basis: BRIEF.md, Scale & Non-Functional Expectations (clients mostly on mobile; client side must be excellent on mobile) and Constraints (records must be correct); decomposition checklist Commonly Forgotten Areas (accessibility baseline, loading states, offline posture). [AUDIT-ADDED: 4 -- product-level accessibility, loading, and offline conventions decided, matching each feature's States field]
>
> -- assumptions-constraints.md, ## Non-Functional Expectations

### From assumptions-constraints.md -- Dependencies

> ASMP-28: "Payment-processing capability — The product requires the ability to accept card and bank-transfer payments directly into each freelancer's own account, to let each freelancer connect her own account, and to report payment status, pending bank transfers, and reversals back to the product. Without it, invoices cannot be paid in-portal at all, and the "pay it by card on the spot" experience described in BRIEF.md's Experience narrative cannot exist." [MODIFIED: extended with per-freelancer account connection and status reporting, based on the value-flow walk that added Payment Account Connection (FEAT-32) and reversal handling (FEAT-25)]
>
> ASMP-29: "Transactional email delivery capability — The product requires reliable email delivery for every notification: proposals, deliverable-ready alerts, approval requests, invoices, reminders, and support-session notices, with delivery and bounce status reported back. Without it, no client-facing communication reaches its recipient, since BRIEF.md states plainly that "clients will not install an app.""
>
> ASMP-30: "File storage and delivery capability for large files — The product requires the ability to store and reliably deliver deliverables from a few MB up to 1 GB+, with version history, within a modest budget. Without it, the deliverable-sharing loop central to the product cannot function at the scale BRIEF.md describes."
>
> ASMP-31: "Subscription-billing capability for the freelancer's own plan — The product requires the ability to charge freelancers a recurring subscription once they exceed the free tier. Without it, the business model described in BRIEF.md's Business Context cannot operate, independent of and separate from the client-side payment-processing capability (ASMP-28)."
>
> ASMP-32: "Domain-verification capability (Later phase) — Custom Domain per Freelancer (FEAT-27) requires the ability to verify that a freelancer controls a domain and serve her portal securely at it. Without it, only the shared default portal address is available; nothing in the MVP depends on it." [AUDIT-ADDED: 4 -- external-service integrations concern: the dependency behind a roadmap feature stated in advance]
>
> -- assumptions-constraints.md, ## Dependencies

### From feature-dependency-map.md -- External Touchpoints

| Capability Category | Features Involved | Integration Specs |
|---------------------|-------------------|-------------------|
| Payment processing into each freelancer's own account — connection, card and bank-transfer payment, status, pending transfers, reversals (ASMP-28) | FEAT-09, FEAT-10, FEAT-20, FEAT-25, FEAT-32 | FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing — payment submission, pending bank transfers, outcome reporting). FEAT-32.SPEC-002 (Payment Account Connection & Status Reporting — connect/reconnect hand-off, readiness status, attention reason, available payment methods, and inbound reversal/chargeback notices relayed to FEAT-25 per XBR-21; authority for pay-link readiness per XBR-19). FEAT-25 validated with no Integration spec for this capability: it records reversals by consuming the inbound reversal/chargeback notice relayed by FEAT-32's integration (FEAT-25.SPEC-005 → FEAT-32.SPEC-002, XBR-21). FEAT-09 validated with no Integration spec for this capability: it derives pay-link availability from the connection status (FEAT-09.SPEC-009 → FEAT-32, XBR-19). FEAT-20 validated with no Integration spec for this capability: its optional Connect payments step only navigates into FEAT-32's connection flow (FEAT-20.SPEC-002 → FEAT-32.SPEC-002) |
| Transactional email delivery with delivery and bounce status (ASMP-29) | FEAT-14 (delivery), relied on by FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-20, FEAT-21, FEAT-23, FEAT-24, FEAT-25, FEAT-26, FEAT-27, FEAT-31, FEAT-32 | FEAT-14.SPEC-001 (Transactional Email Delivery — sends every composed email and reports delivery, bounce, and failure status back). FEAT-02 and FEAT-03 validated with no Integration spec for this capability: they use it through Notification specs FEAT-02.SPEC-011, FEAT-03.SPEC-006, and FEAT-03.SPEC-007. FEAT-05, FEAT-06, FEAT-07, and FEAT-08 validated with no Integration spec for this capability: they use it through Notification specs FEAT-05.SPEC-008, FEAT-06.SPEC-006, FEAT-07.SPEC-003, FEAT-07.SPEC-004, and FEAT-08.SPEC-007. FEAT-09, FEAT-10, and FEAT-11 validated with no Integration spec for this capability: they use it through Notification specs FEAT-09.SPEC-010, FEAT-10.SPEC-007, and FEAT-11.SPEC-004. FEAT-18 and FEAT-32 validated with no Integration spec for this capability: they use it through Notification specs FEAT-18.SPEC-010, FEAT-18.SPEC-011, and FEAT-32.SPEC-006. FEAT-20, FEAT-21, and FEAT-23 validated with no Integration spec for this capability: they use it through Notification specs FEAT-20.SPEC-006, FEAT-21.SPEC-011, and FEAT-23.SPEC-008 (FEAT-21.SPEC-005 also sends its re-verification link through it). FEAT-24, FEAT-25, and FEAT-31 validated with no Integration spec for this capability: they use it through Notification specs FEAT-24.SPEC-008, FEAT-24.SPEC-009, FEAT-25.SPEC-007, FEAT-25.SPEC-008, FEAT-31.SPEC-006, and FEAT-31.SPEC-007. FEAT-26 and FEAT-27 validated with no Integration spec for this capability: they use it through Notification specs FEAT-26.SPEC-004 (signed-copy confirmation to both parties) and FEAT-27.SPEC-004 (custom domain verified confirmation) |
| Large-file storage and delivery with version history (ASMP-30) | FEAT-06, FEAT-16, FEAT-17 | FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability — resumable ingestion, streaming/download delivery, reported transfer status, within the stated budget; authority for storage limits per XBR-14). FEAT-17 validated with no Integration spec for this capability: it consumes storage through FEAT-16 (FEAT-17.SPEC-003 → FEAT-16.SPEC-007, XBR-13, XBR-14), mirroring FEAT-06. FEAT-06 validated with no Integration spec for this capability: it consumes storage through FEAT-16 (FEAT-06.SPEC-003 → FEAT-16, XBR-12, XBR-14) |
| Subscription billing for the freelancer's own plan (ASMP-31) | FEAT-23 | FEAT-23.SPEC-003 (Subscription Billing Processing — submits upgrade, downgrade, and cancellation changes to the subscription-billing capability and receives charge outcomes, renewal and period-end events, and failure reasons; FEAT-23.SPEC-004 applies them to the Subscription Plan) |
| Domain verification and secure serving at a freelancer's own domain — Later phase (ASMP-32) | FEAT-27, FEAT-05 | FEAT-27.SPEC-002 (Domain Verification & Secure Serving — verifies Nadia controls the added domain, serves her portal securely at it once verified, and reports verification, failure reason, and re-check results; FEAT-27.SPEC-003 guarantees the shared default address remains the fallback per XBR-35). FEAT-05 validated with no Integration spec for this capability: it only reads the Custom Domain Record (FEAT-05.SPEC-002, FEAT-05.SPEC-003; Later phase) |
| Electronic-signature attestation for legally binding proposal signing — v1 phase (not in the Dependencies section of assumptions-constraints.md; added from the validated FEAT-26 Brief per Step 7.5 rule 4) | FEAT-26 | FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability — submits the signature data captured by FEAT-26.SPEC-002 for attestation and returns the confirmation that gives the signed acceptance legal weight beyond a self-recorded timestamp; the jurisdictional standard it must meet is left to Stage 4) |

## 2. Non-Functional Requirements & Scale Design

| Dimension | Expectation | Source |
|-----------|-------------|--------|
| Launch load | "a few thousand freelancers, each with 3–15 active clients and a handful of contacts per client" — i.e. roughly 10,000–45,000 client companies and a low six-figure count of client contacts in year one; request volume is human-paced (proposals, approvals, invoices, comments), not machine-generated | Profile Section 7, BRIEF.md ## Scale & Non-Functional Expectations (Users year one); ASMP-22. The derived client/contact ranges are arithmetic on the quote |
| Data volume | "deliverables typically tens of MB and sometimes over 1 GB, retained with version history for the life of the account"; per-file ceiling 2 GB; storage allowance 5 GB free / 100 GB paid | Profile Section 7, ASMP-22; feasibility FEAT-16 (platform-parameters.md `deliverable-file-size-ceiling`, storage allowances) |
| Growth trajectory | Assumption — no upstream growth figure beyond year one; the architecture is sized to reach roughly 10× year-one accounts (tens of thousands of freelancers) without re-platforming, with object-storage bytes (not request volume) as the first cost to scale | Assumption — upstream documents state year-one volume only; stored bytes grow for the life of each account (ASMP-22) |
| Performance targets | "client-facing pages (deliverable review, approval, invoice payment) become interactive within roughly 2 seconds on a typical mobile connection; dashboard totals appear within roughly 1–2 seconds" | Profile Section 7, ASMP-21 |
| Availability posture | "no specific uptime target is committed; the product is expected to be reliably available during ordinary business use, and emails that fail to deliver are surfaced to the freelancer within minutes rather than lost"; Assumption — single-region managed hosting with provider-standard uptime and point-in-time database restore is adequate; no multi-region failover | Profile Section 7, ASMP-26 (quote); the hosting/restore posture is an Assumption — no uptime number upstream |
| Security posture | "strict data isolation between clients (a client never sees another client's anything) … any operator access is read-only and visible to the freelancer"; "no card or payment data is ever captured or stored by the product itself" | Profile Section 7, ASMP-23, ASMP-24 |
| Compliance obligations | GDPR-class handling of freelancer and client-contact personal data worldwide; export and deletion on request; valid-invoice content (sequential number, both parties' details, issue/due dates, tax line); "accepted proposals, approvals, and sent invoices are timestamped at creation and never silently altered afterward" | Profile Section 7, ASMP-23, ASMP-24, ASMP-25; BRIEF.md ## Scale & Non-Functional Expectations (Privacy; Correctness & records) |
| Budget constraint | "infrastructure under roughly $100/month" | Profile Section 3, Scale hints row (BRIEF.md ## Constraints) |
| Accessibility & degraded states | Readable on a phone without zooming, screen-reader and keyboard usable, never colour-only, brand colours kept legible; real progress while loading; record-creating actions "never pretend to succeed offline" | Profile Section 7, ASMP-27 |
| Data residency | Assumption — no residency mandate upstream; one hosting region co-located with the database, defaulting to an EU region as the conservative choice for GDPR-class data, with the CDN serving static assets worldwide | Assumption — "personal-data handling must hold up worldwide (GDPR)" (BRIEF.md) names a regime, not a region |

The binding drivers are, in order: (1) **correctness of financial and evidentiary records** (ASMP-25, feasibility verdicts Hard for FEAT-09 and FEAT-10) — gapless invoice numbering, exactly-once acceptance/approval and processor-authoritative payment state push the design toward a single relational database with real transactions, conditional writes and an idempotent inbound-event ledger; (2) **strict client isolation plus a read-only operator** (ASMP-23) — this shapes a single, central authorization layer on the server and rules out client-trusting data access; (3) **the ~$100/month infrastructure budget against large, uncapped-retention files** (ASMP-22, ASMP-30) — this makes storage egress price the decisive axis for object storage and pushes every other layer toward free tiers and scale-to-zero services; and (4) **~2 s mobile interactivity for client pages** (ASMP-21) — this favours server rendering with small client bundles and direct-to-storage file transfer.

Request load itself is modest: a few thousand freelancers with human-paced activity sit comfortably inside every candidate in the landscape, so scale ceilings are rarely the deciding axis. The budget is under real tension only from stored bytes — at year-one volume the feasibility assessment estimates ~20 TB could cost ~$300/month even on zero-egress storage (feasibility Section 4, first risk; Open Question 2) — so the architecture keeps storage cost observable (FEAT-16.SPEC-005 aggregation) and treats the allowance-versus-budget reconciliation as a product decision rather than hiding it.

## 3. Technology Stack Decisions

### Frontend Framework

| Field | Value |
|-------|-------|
| Context | Scale: Large (33 features, 220 specs) and Interaction Complexity: Large (profile Section 4); 65 Screen specs across a freelancer workspace (desktop) and a client portal that "must be excellent on mobile" (Section 7, BRIEF.md); ASMP-21 ~2 s interactivity on mobile for review/approval/payment; Complex forms = No (max 7 inputs, FEAT-09.SPEC-003); Offline = Yes for a comment queue and last-loaded caches (FEAT-07.SPEC-008, FEAT-29.SPEC-004); feasibility names SSR frameworks as the route to the ~2 s target (FEAT-03, FEAT-07 candidate approaches) |
| Recommended | Next.js 16 (App Router, React 19, TypeScript), Node.js runtime for all data-touching routes |
| Rationale | Server Components render the 65 screens with data already in the HTML, keeping client bundles small on mobile for the ASMP-21 target; server actions cover the form-light mutation surface (≤7 fields per form); nested layouts map cleanly onto three distinct shells (freelancer workspace, per-freelancer branded portal, operator console). The landscape rates it "Low" integration effort with the largest SDK ecosystem — which matters here because 7 Integration specs (Stripe, Resend, R2, e-signature, domains) all publish Node/TypeScript SDKs (landscape Cross-Area Compatibility Notes). It is also native to the recommended host (Section 13) and pairs with every other selection (Drizzle, TanStack Query, Tailwind, shadcn/ui). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| React Router v7 (Remix lineage) | Open source (MIT); runs on any Node host | Low–Medium — fewer turnkey templates; explicit loader/action wiring per route | High — runtime-agnostic, no framework ceiling at this load | Low — runtime-agnostic, Vite-based | Mainstream React | The team wants progressive-enhancement forms and a host-neutral runtime (e.g. deploying to Fly.io/Render containers) more than Next.js's RSC model and Vercel-native features |
| SvelteKit (Svelte 5) | Open source (MIT) | Low–Medium — smaller SDK/component ecosystem to fill | High | Medium — Svelte code does not port | Svelte-specific skills | The team is Svelte-first and client-bundle size on low-end phones proves to be the binding ASMP-21 constraint after measurement |
| Nuxt 4 | Open source (MIT) | Medium — Vue-specific ecosystem | High | Medium — Vue code does not port | Vue-specific skills | The implementing team's existing skills are Vue/Nuxt, making React ramp-up the larger risk than ecosystem breadth |

### Backend / API Layer

| Field | Value |
|-------|-------|
| Context | 62 Automation specs and 7 Integration specs (profile Section 1); webhooks from payments, subscription billing, email, e-signature and domain services (feasibility Section 3 theme "Idempotent, event-time-ordered inbound webhook ingestion"); Real-time = No, so no socket server is needed; long-running work exists (FEAT-24.SPEC-003 multi-GB archive, FEAT-24.SPEC-004 staged deletion) that exceeds request timeouts; ADR-001 selects Next.js |
| Recommended | Next.js framework server layer — server actions for user mutations, Route Handlers for webhooks, upload-session and download-grant endpoints — on the Node.js runtime; all scheduled, retried and long-running work delegated to the job runner (ADR-012) |
| Rationale | The landscape's "Framework server layer" option covers request/response, webhooks and form mutations "in the same deployable as the frontend", with the explicit caveat that long-running work goes to a job runner — which ADR-012 provides. At a few thousand freelancers with human-paced traffic (Section 2) a separate API service adds a second deployable, a second auth boundary and more cost against the $100/month budget without adding capability. Keeping one TypeScript codebase lets the central authorization guard (Section 11) and the shared domain rules (60 Logic/Rule specs) run in one process for both server actions and webhooks. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Hono on Node.js | Open source (MIT); needs its own host or function | Medium — second deployable and CORS/auth boundary | High | Low — runtime-portable | TypeScript, middleware patterns | A public API or a native mobile client is added, so the API must be versioned and deployed independently of the web frontend |
| NestJS (Node.js) | Open source (MIT); long-running container host (~$10–40/month) | Medium–High — modules/DI conventions, separate service | High | Low–Medium — framework conventions | NestJS DI/guards | The team grows past ~5 backend engineers and wants enforced module boundaries and guard-based authorization as framework structure |
| Django + Django REST Framework | Open source (BSD); separate Python service | High — two languages, two deploys, separate frontend | High | Medium — Python-only data layer | Python + React split skills | The team is Python-first and wants Django admin for the operator console enough to accept a split stack |

### Database

| Field | Value |
|-------|-------|
| Context | 23 entities, 60 relationships, 35 XBR (Data Complexity: Medium); contention on 12 of 21 entities resolved reject-with-refresh; exactly-once acceptance/approval (FEAT-03.SPEC-003, FEAT-08.SPEC-003); gapless per-freelancer invoice numbering (FEAT-09 verdict Hard) and processor-authoritative multi-writer invoice state (FEAT-10 verdict Hard); immutable evidentiary records (ASMP-25); GDPR-class data with export/erasure (ASMP-23/24); budget under ~$100/month; preview environments per branch (Section 13) |
| Recommended | Neon serverless Postgres — Launch plan at launch (usage-based, $0.106/CU-hour, $0.35/GB-month storage), Scale plan when sustained load or retention needs grow; one project with branches for preview/staging |
| Rationale | Every Hard verdict in the feasibility assessment turns on relational transactions: row-locked counters for gapless numbering, conditional `UPDATE … WHERE status = …` for exactly-once writes, unique constraints for idempotency keys — all standard Postgres. Neon's scale-to-zero and usage pricing keep the launch cost to tens of dollars against the $100/month budget, and its branching gives each preview deployment an isolated copy of the schema. Standard Postgres keeps the exit path open (dump/restore). Supabase's bundled auth/storage/realtime are not needed: storage goes to R2 for egress cost (ADR-008), auth to Better Auth (ADR-026), and Real-time = No. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Supabase Postgres (Pro) | $25/month incl. 8 GB DB, 250 GB egress; $0.125/GB beyond 8 GB | Low — managed; extra platform surface | High for this workload | Medium — standard Postgres, but platform features (RLS-coupled auth, storage) add coupling if adopted | Postgres + Supabase platform | The team wants one vendor for database, auth and row-level-security-enforced client isolation, accepting coupling in exchange for fewer integrations |
| Amazon RDS for PostgreSQL | Instance-hour plus storage; small instances tens of USD/month, always-on | Medium — VPC, IAM, parameter groups | Very high (Multi-AZ, replicas) | Low–Medium — Postgres, AWS networking | AWS operations | An explicit uptime target or Multi-AZ failover requirement is introduced (ASMP-26 currently commits none), or the product consolidates onto AWS |
| Render Managed Postgres | From $40/month for 1 vCPU / 2 GB, fixed | Low | Medium–High | Low — dump/restore | Mainstream Postgres | Hosting moves to Render (Section 13 alternative) and predictable fixed billing beats scale-to-zero savings |

### ORM / Data Access

| Field | Value |
|-------|-------|
| Context | 23 entities with 60 relationships; SQL-level concurrency techniques needed by FEAT-03, FEAT-08, FEAT-09, FEAT-10 (row locks, conditional updates, unique-constraint retries — feasibility Candidate Approaches); per-currency financial aggregation (FEAT-12.SPEC-003); migrations across preview/staging/prod branches; ADR-002 Node runtime, ADR-003 Postgres |
| Recommended | Drizzle ORM v1 with drizzle-kit migrations, using the Neon serverless driver in WebSocket/Pool mode so interactive transactions (`SELECT … FOR UPDATE`) work from Vercel functions and Trigger.dev tasks |
| Rationale | The landscape describes Drizzle as "SQL-close typed queries" with "built-in migration generation" — exactly what the Hard feasibility verdicts need: the gapless counter lock, version-token conditional updates and `ON CONFLICT` idempotency are written as near-SQL with full types, and CTE/window aggregations for the dashboard stay expressible. It runs in any Node or edge runtime without an accelerator (landscape Cross-Area note on Prisma), and its TypeScript-native schema is shared by the web app and job tasks in one package. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Prisma ORM (v6) | Open source; optional paid Accelerate | Low — codegen step; Prisma Migrate | High | Medium — proprietary schema language | Mainstream; Prisma DSL | The team values Prisma Studio and schema-first modelling over SQL-closeness, and is comfortable writing raw SQL for the counter-lock and conditional-update paths |
| Kysely | Open source (MIT) | Medium — schema types and migrations managed separately | High | Low — thin SQL builder | Strong SQL | The team wants maximum SQL control (heavy CTE/window reporting) and is willing to own schema typing and migrations separately |

### CSS / Styling

| Field | Value |
|-------|-------|
| Context | No user-supplied design system (design-agnostic, Section 9); per-freelancer logo and colour on every client screen and email with automatic legibility adjustment (FEAT-19.SPEC-001–003, XBR-31); ASMP-27 accessibility (never colour-only, legible brand colours); 65 Screen specs across three shells; mobile-first client portal (ASMP-21) |
| Recommended | Tailwind CSS v4 with theme tokens defined as CSS custom properties (`@theme`), and a per-freelancer `--brand` / `--brand-foreground` variable pair injected at the portal layout root |
| Rationale | The landscape notes Tailwind v4's "CSS-variable theme tokens map to per-freelancer brand colours" — the one runtime theming need the specs impose (FEAT-19). A static, compiled stylesheet with no runtime CSS-in-JS keeps mobile payloads small for ASMP-21, and Tailwind is the styling base the selected component layer (shadcn/ui, ADR-024) assumes. Build-time token systems (vanilla-extract, Panda) would still need a runtime variable layer for brand colours (feasibility FEAT-19 risk). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| CSS Modules | Built into Next.js | Low | High | Low — web-standard CSS | Plain CSS | The team prefers authored CSS files over utility classes and will hand-build components without shadcn/ui |
| vanilla-extract | Open source (MIT) | Medium — bundler plugin | High | Medium — TS-authored styles | TypeScript styling | Type-checked design tokens become important because a user-supplied design system is later added with a large token set |
| Panda CSS | Open source (MIT) | Medium — codegen step | High | Medium — smaller ecosystem | Panda recipes | The team wants typed variants/recipes as a build-time system and accepts a smaller ecosystem |

### State Management

| Field | Value |
|-------|-------|
| Context | Real-time = No (snapshot screens); Collaboration/concurrency = Yes via reject-with-refresh on 12 entities; Offline = Yes for FEAT-07.SPEC-008 (offline comment queue), FEAT-04.SPEC-001 (queued schedule save, AC-15), FEAT-12.SPEC-001 (last loaded totals), FEAT-29.SPEC-004 (cached feed); Interaction Complexity: Large but driven by server-side automations, not client state; ASMP-27 forbids record-creating actions pretending to succeed offline |
| Recommended | React Server Components as the primary read path, plus TanStack Query v5 on the client islands that need caching or offline behaviour, with the query/mutation cache persisted to IndexedDB (paused mutations for the comment queue and queued schedule save; last-loaded snapshots for dashboard and feed); no global client store |
| Rationale | Almost all state is server-derived (invoices, milestones, comments), so RSC removes client fetching for most of the 65 screens. The landscape lists TanStack Query v5 with "offline mutation persistence" — the one capability that directly serves the Offline signal's two queue specs and two last-loaded specs. Version tokens returned with each query feed the reject-with-refresh guard (ADR-014); a conflict response invalidates the query and re-renders with the fresh record. Accept, approve and pay are never registered as persistable mutations (ASMP-27). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Zustand (with RSC for reads) | Open source (MIT) | Low | High | Low — React-only | Mainstream React | Offline needs grow into richer local drafts (e.g. multi-screen offline editing) that are UI state rather than server-cache state |
| SWR | Open source (MIT) | Low | High | Low — React-only | Mainstream React | The offline mutation queue (FEAT-07.SPEC-008) is dropped and only stale-while-revalidate reads remain |
| Redux Toolkit (RTK Query) | Open source (MIT) | Medium — slices and store setup | High | Medium | Redux patterns | Real-time is later introduced with interdependent client state that needs explicit action flows and devtools-level traceability |

### Build Tooling

| Field | Value |
|-------|-------|
| Context | ADR-001 Next.js 16 (Turbopack bundled and "stabilized for production builds in Next.js 16" per landscape); single deployable for 33 features plus a job-task directory built by the Trigger.dev CLI (ADR-012) |
| Recommended | Turbopack (bundled with Next.js 16) for dev and production builds; pnpm as package manager; TypeScript compiler for type-checking in CI |
| Rationale | The decision guide's rule is that meta-frameworks ship their own pipeline and overriding it needs a documented driver — none exists here. pnpm gives fast, disk-efficient installs and a strict lockfile for reproducible CI builds across preview, staging and production. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Bun (package manager and toolchain) | Open source (MIT) | Low–Medium — runtime compatibility to verify | High | Medium — runtime-specific behaviour | Bun toolchain | Install and test-run speed in CI becomes a measured bottleneck and Next.js/Trigger.dev compatibility is verified |
| Vite | Open source (MIT) | Low | High | Low | Mainstream | The frontend framework changes to React Router v7, SvelteKit or Nuxt (ADR-001 alternatives), whose default pipeline is Vite |

## 4. Platform & Service Decisions

### File & Object Storage

| Field | Value |
|-------|-------|
| Context | File upload = Yes (FEAT-06.SPEC-001, FEAT-06.SPEC-003, FEAT-16.SPEC-002, FEAT-16.SPEC-007, FEAT-17.SPEC-001; ASMP-22, ASMP-30); Import/export = Yes (FEAT-22.SPEC-002, FEAT-24.SPEC-003); files tens of MB to >1 GB, 2 GB ceiling, uncapped version retention for the account's life, "≥98% of uploads incl. >500 MB complete without restart" (feasibility FEAT-16); clients stream/download on mobile; budget ~$100/month; feasibility FEAT-16 risk: ~20 TB ≈ $300/month on R2 |
| Recommended | Cloudflare R2 (Standard storage class) via the S3 API: browser-direct multipart uploads with presigned part URLs (resume state persisted server-side and in IndexedDB), short-lived presigned GET URLs for authorized viewers (byte-range capable), a private bucket for deliverables/archives and a public CDN-fronted bucket for branding logos only |
| Rationale | Egress is the cost that scales with the product's core loop — clients repeatedly streaming large videos and design files — and R2 charges $0 egress at $0.015/GB-month storage with 10 GB free (landscape). Direct-to-storage multipart keeps multi-GB bytes off the serverless backend (feasibility FEAT-06 candidate approach), and the S3 API means one client library also works against B2 or S3 (landscape Cross-Area note), preserving the exit path if storage cost forces a move. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Backblaze B2 | $0.00695/GB-month; free egress up to 3× stored data (and via Cloudflare) | Low — S3-compatible | Very high | Low — S3 API | S3 API | Stored bytes, not egress, dominate cost at the year-one storage model (Open Question 2) — B2 is roughly half R2's storage price (~$140 vs ~$300/month at 20 TB per feasibility) |
| Amazon S3 | $0.023/GB-month plus requests and data-transfer-out | Low–Medium — IAM, CORS | Very high | Medium — egress makes leaving costly | AWS IAM | The product consolidates on AWS or needs S3-only features (e.g. object lifecycle to archive tiers) and egress is fronted by a CDN |
| Supabase Storage | Counted against plan egress (Pro 250 GB, $0.09/GB beyond) plus storage overage | Low if Supabase is adopted | High | Medium — couples to Supabase | Supabase | The Database decision switches to Supabase and TUS resumable uploads with RLS-tied access rules are preferred over presigned URLs |

### Email & Messaging Delivery

| Field | Value |
|-------|-------|
| Context | Notifications (email/push/SMS) = Yes: 26 Notification specs, all email (push/SMS named by no spec); FEAT-14.SPEC-001 delivery with bounce/failure status; FEAT-14.SPEC-003 3 retries within 6 h, bounces never retried; FEAT-14.SPEC-005 branded presentation with freelancer logo/colour; ASMP-26 failures "surfaced … within minutes"; ASMP-29 "reliable email delivery for every notification"; FEAT-05 magic-link sign-in depends on deliverability |
| Recommended | Resend (Pro, $20/month for 50,000 emails; Free 3,000/month during development) with delivery/bounce/complaint webhooks, React Email templates rendered server-side with inline brand colours and raster logos, sent from a platform domain (display name "{Freelancer} via Clientroom", reply-to the freelancer) and separate subdomains for transactional vs sign-in mail |
| Rationale | Resend's developer API, delivery-event webhooks and React email templates fit the Next.js/TypeScript stack (landscape) and let the 26 notification templates share components and brand tokens with the portal. Its price fits the budget at year-one volume where Postmark's $1.80 per extra 1,000 would not. The webhook events feed the same idempotent inbound-event ledger as payments (ADR-023), so bounces surface to the freelancer within minutes (FEAT-14.SPEC-006). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Postmark | $15/month base plus $1.80 per extra 1K emails | Low — message streams, bounce webhooks | High | Medium — proprietary streams | Mainstream | Measured deliverability of sign-in links falls short of FEAT-05's ≥95% first-try target and a deliverability-first sender justifies the higher per-email cost |
| Amazon SES | $0.10 per 1,000 emails | Medium — IAM, domain reputation, SNS events | Very high | Medium — AWS event handling | AWS | Monthly volume grows past a few hundred thousand emails and cost dominates over integration effort |
| SendGrid (Twilio) | ~$0.40 per 1,000 at paid tiers | Low–Medium | Very high | Medium — Twilio account coupling | Mainstream | Marketing email (not in any spec today) is added alongside transactional mail on one platform |

### Payments & Billing

| Field | Value |
|-------|-------|
| Context | Payments/billing = Yes (FEAT-10.SPEC-003, FEAT-23.SPEC-003, FEAT-32.SPEC-002, FEAT-09.SPEC-001 to FEAT-09.SPEC-010; ASMP-28, ASMP-31): card and bank-transfer payments "directly into each freelancer's own account", "the platform never touches card numbers or holds funds", no cut of payments (BRIEF.md ## Business Context); reversals relayed to FEAT-25 (XBR-21); separate recurring billing of the freelancer's own plan; FEAT-10 verdict Hard (duplicated/out-of-order processor events, processor-authoritative state); FEAT-32 connect in <5 minutes, ≥90% before first invoice; worldwide from day one |
| Recommended | Stripe: Connect with Standard accounts and direct charges for client invoice payments (hosted Account Links onboarding; Payment Element embedded in the portal pay screen; connected-account webhooks for payment, dispute and account-status events), plus Stripe Billing on the platform account for the freelancer's own subscription |
| Rationale | Standard accounts with direct charges place funds and Stripe fees on the freelancer's own account with "no Connect fee to platform" (landscape) — matching the no-cut business model and ASMP-28 exactly; the Payment Element keeps card data inside Stripe (ASMP-24). Stripe's self-serve hosted onboarding is the only landscape option aimed at the 5-minute connect target (Adyen is sales-led; Mollie is Europe-focused). One vendor covers both the client-payment and own-plan streams while keeping them separate charge streams (landscape Cross-Area note), and one webhook ledger (ADR-023) handles both. Country coverage and bank-transfer method availability remain launch-market questions (feasibility Open Question 7); unsupported countries fall back to XBR-19's direct-payment instructions. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Paddle (own-plan subscriptions only, Stripe Connect kept for client payments) | ~5% + 50c per subscription transaction (vs Stripe Billing 0.7% + card fees) | Medium — second vendor and webhook stream | High | Medium | Mainstream | The operator decides not to register for and collect VAT/GST on Clientroom's own worldwide subscription revenue (feasibility Open Question 8) — Paddle as Merchant of Record carries it |
| Mollie Connect | Blended per-transaction fees | Medium | High | Medium | Mainstream | Launch markets are confirmed as European-only and local payment methods (iDEAL, Bancontact, SEPA) outrank worldwide coverage |
| Adyen for Platforms | Interchange++ with monthly minimums | High — sales-led onboarding, heavier integration | Very high | Medium–High | Payments specialists | Processing volume reaches enterprise scale where interchange++ pricing and global acquiring outweigh minimums and onboarding effort |

### AI & Intelligent Behavior

Not activated — profile Section 3: "No specs reference AI or machine-learning behavior; the keyword "classification" matches only the paid/due/overdue invoice status classification (FEAT-12.SPEC-001 to FEAT-12.SPEC-004) and FEAT-28.SPEC-004 covers rule-based search result ranking."

### Search

| Field | Value |
|-------|-------|
| Context | Search = Yes: FEAT-28.SPEC-001–004 cross-entity global search with access-scoped results (FEAT-28.SPEC-003) and rule-based ranking (FEAT-28.SPEC-004); keywords "full-text" and "faceted" match no spec; per-account corpus of 3–15 active clients with history (few hundred records); feasibility FEAT-28 notes an external index would duplicate GDPR-class data and become "a single point of isolation failure" (ASMP-23) |
| Recommended | PostgreSQL full-text search on Neon — `tsvector` columns with GIN indexes for word matching plus `pg_trgm` trigram indexes for prefix/partial matching of names and invoice numbers — queried through the same account-scoped data-access functions as every other read; ranking implemented in SQL per FEAT-28.SPEC-004 |
| Rationale | The corpus is tiny per account, so the database option answers "as the user types" latency without a sync pipeline, keeps the operator's one-account scope and client isolation enforced by the same query path (FEAT-28.SPEC-003; XBR-29), and adds no second copy of billing names and proposal scope that FEAT-24 deletion would have to chase. Cost is included in the database (landscape). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Typesense (Cloud) | From ~$14/month or free self-hosted | Medium — index sync, per-tenant filters | High | Low — open source | Search engine ops | Typo tolerance becomes a product requirement (not in FEAT-28 today) and per-account corpora grow into thousands of records |
| Meilisearch (Cloud) | ~$20–30/month or free self-hosted | Medium — index sync | High | Low — open source | Search engine ops | As Typesense, with preference for Meilisearch's relevance tuning |
| Algolia | ~$0.50 per 1K records and $0.40 per 1K searches | Medium — indexing pipeline | Very high | High — proprietary | Mainstream | Search becomes a primary cross-account surface (e.g. operator-wide search) needing hosted instant search with analytics |

### Background Jobs & Scheduling

| Field | Value |
|-------|-------|
| Context | Background processing = Yes (FEAT-11.SPEC-001, FEAT-12.SPEC-004, FEAT-14.SPEC-003, FEAT-24.SPEC-003, FEAT-24.SPEC-005); Import/export = Yes (FEAT-22.SPEC-002, FEAT-24.SPEC-003); Notifications (FEAT-14.SPEC-002); 62 Automation specs; retry intervals from platform-parameters (email 3×/6 h, stop-billing relay every 15 min, deletion finalization every 15 min, daily retention purge, 15-min support auto-close); multi-GB archive build exceeding serverless timeouts (feasibility FEAT-24 Hard); feasibility FEAT-09/FEAT-13 risk: managed runners need an outbox for atomic enqueue |
| Recommended | Trigger.dev v3 (Cloud; Free 50K runs/month, paid from $10/month) for all scheduled, retried and long-running tasks, fed by a Postgres transactional outbox: side-effect jobs are written as `job_outbox` rows inside the same transaction as the triggering write, dispatched immediately after commit, and re-dispatched by a one-minute Trigger.dev sweep for any row not yet acknowledged |
| Rationale | Trigger.dev runs long tasks "without serverless timeouts" and supports schedules (landscape), covering both the minute-level sweeps and the FEAT-24 archive/deletion workflows on one platform; it is open source with a Docker self-host exit (low lock-in), and its free tier covers year-one volume. The outbox closes the atomicity gap the feasibility assessment flags for managed runners: an accepted proposal, approval or payment event can never commit without its email, activity-entry retry or totals refresh being durably queued. Invoice creation itself stays in the originating transaction (ADR-014), so the job boundary never splits a trigger from its invoice. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Inngest | Free 25K runs/month; Basic $30/month; Pro $300/month | Low — SDK, event-driven steps calling own endpoints | High | Medium–High — managed only, no self-host | Mainstream | Step-level durable workflows invoked on the app's own Vercel functions are preferred and all long steps can be chunked under function time limits |
| pg-boss workers | Free; cost of an always-on worker host (~$5–12/month on Fly.io/Railway) | Medium — operate a worker process | Medium–High | Low — open source, Postgres-backed | Mainstream Node | Hosting moves to a container platform (Render/Fly.io) where a long-running worker is already present — pg-boss then enqueues inside the same Postgres transaction with no outbox |
| Upstash QStash | Usage-based per message | Low — HTTP | High | Low | Mainstream | Only short HTTP-shaped jobs remain (e.g. if the data-export archive is decided to be metadata-only, Open Question 3) |

### Caching & Performance

| Field | Value |
|-------|-------|
| Context | Scale hints = Yes (ASMP-21 ~2 s client pages, dashboard totals within 1–2 s; ASMP-22; ASMP-26; budget ~$100/month); Offline = Yes (FEAT-07.SPEC-008, FEAT-29.SPEC-004; ASMP-27); FEAT-12.SPEC-004 "currently held Financial Totals" recomputed on invoice/payment events; feasibility FEAT-12 risk: overdue status is time-driven |
| Recommended | No separate cache service at launch: Vercel CDN plus the Next.js framework cache for static assets and public pages (signup, referral landing, help references); per-request dynamic rendering for all authenticated/private pages; database-level caching as a Postgres `financial_totals` table (per account/client/project and currency) maintained by FEAT-12.SPEC-004 refresh jobs plus an hourly time-zone-aware overdue sweep; client-side last-loaded caches via TanStack Query persistence (ADR-006) |
| Rationale | Both landscape options "CDN and framework cache" and "Database-level caching" are included in plans already paid for, so the ASMP-21 targets are met without adding Redis to the budget. Held per-currency totals make the dashboard a single indexed read (1–2 s target) while honoring the spec's held-totals model; the overdue sweep answers the time-driven drift risk. Private data is never placed in shared caches, protecting client isolation (ASMP-23). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Upstash Redis | Free 256 MB / 500K commands; $0.20 per 100K commands; fixed from $10/month | Low — REST SDK | High | Low — Redis protocol | Mainstream | Rate limiting (magic-link requests, webhook bursts) or hot aggregates outgrow Postgres-backed counters under measured load |
| Postgres materialized views / read replicas | Included; replica compute extra | Low–Medium | Medium–High | Low | SQL | Dashboard reads start contending with write traffic, or totals logic becomes easier to express as a view refreshed on schedule |
| Redis Cloud | Plan-based | Low–Medium | High | Low | Redis ops | A persistent, conventional Redis is needed for a later feature (e.g. real-time presence) beyond HTTP-style access |

### Real-time & Collaboration

| Field | Value |
|-------|-------|
| Context | Real-time = No; Collaboration/concurrency = Yes (FEAT-03.SPEC-003, FEAT-08.SPEC-003, FEAT-10.SPEC-004, FEAT-10.SPEC-005, FEAT-11.SPEC-001, FEAT-18.SPEC-006); reject-with-refresh on 12 of 21 entities; exactly-once acceptance and approval; gapless invoice numbering "never reused, never skipped" (FEAT-09.SPEC-007); contention is "between one freelancer and client contacts on the same record, not a shared-editing workspace" (profile Section 3) |
| Recommended | Database optimistic concurrency and transactions — every contended row carries a `version` integer returned to the client as a token; mutations use conditional `UPDATE … WHERE id = $1 AND version = $2` (zero rows → reject-with-refresh); exactly-once transitions via status-guarded updates; per-freelancer invoice counter row locked `FOR UPDATE` inside the invoice-creating transaction (no native sequences); unique constraints for idempotency keys; no realtime transport |
| Rationale | The landscape's "Database optimistic concurrency and transactions" option delivers reject-with-refresh and exactly-once semantics inside the data store with no extra service, exactly the technique every relevant feasibility Candidate Approach names (FEAT-01, 03, 04, 08, 09, 10, 18). Native Postgres sequences skip values on rollback and would violate FEAT-09.SPEC-007 (feasibility FEAT-09 risk), hence the locked counter. With Real-time = No and snapshot screens, a push transport would add cost without a spec that consumes it. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Supabase Realtime | Included in Supabase plans (usage quotas) | Low if Supabase is adopted | High | Medium — Supabase-only | Supabase | The database moves to Supabase and live refresh of threads/feeds is added as a product requirement |
| Ably | Free 6M messages/month; $2.50 per million | Low — SDK | Very high | Medium — proprietary protocol | Mainstream | A later spec requires live comment threads or presence on the deliverable review screen |
| Pusher Channels | From $49/month; free tier 200 connections | Low — SDK | High | Medium | Mainstream | Simple change-notification push is wanted and connection counts stay within Pusher's tiers |

### Analytics & Product Telemetry

| Field | Value |
|-------|-------|
| Context | Scale hints = Yes (ASMP-22 a few thousand freelancers; BRIEF.md ## Scale & Non-Functional Expectations); success metrics referenced in feature notes (first-session activation, median sign-up-to-draft under 15 minutes for FEAT-20, ≥90% payments connected before first invoice for FEAT-32, ≥95% magic-link first-try success for FEAT-05); GDPR-class data (ASMP-24); FEAT-33 referral attribution is a product feature, measured in aggregate only (FEAT-33.SPEC-005) |
| Recommended | PostHog Cloud (Free 1M events/month; then from $0.00005/event) with server-side event capture for funnel milestones keyed by opaque account IDs, no emails or names in event properties, session replay disabled on client-portal routes, and portal analytics limited to anonymous aggregate events |
| Rationale | PostHog covers events and funnels for the activation metrics above within its free tier at year-one volume, and it is open source with a self-host option (landscape) — a lower lock-in and data-control posture for GDPR-class users. Server-side capture of milestone events avoids cookie-consent friction on client portals and keeps client-contact personal data out of the analytics store (feasibility FEAT-20 risk). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Plausible | Subscription by pageviews | Low — one script | Medium | Low — open source | Minimal | Only cookie-less page-level metrics (referral landing, signup conversion) are wanted and funnel/event analysis is dropped |
| Mixpanel | Free 20M events/month; Growth ~$0.00028/event | Low | Very high | Medium — proprietary | Mainstream | Event volume exceeds PostHog's free tier by a wide margin and retention analysis is the primary need |
| Amplitude | Free basic tier; growth from $49/month | Low | Very high | Medium — proprietary | Analytics specialists | Behavioural cohorts and experimentation become a core growth practice with a dedicated analyst |

### Geo & Maps

Not activated — profile Section 3: "No specs reference location or mapping behavior; keywords "geolocation", "GPS", "proximity", "geocoding" match nothing in specs, BRIEF.md, or assumptions-constraints.md."

### Internationalization

| Field | Value |
|-------|-------|
| Context | Internationalization = Yes (FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-006, FEAT-15.SPEC-008, FEAT-22.SPEC-003): worldwide currencies, tax labels/rates and time zones "must not be hard-coded"; "English only at launch" with no translation spec; multi-currency never aggregated (FEAT-15.SPEC-007, XBR-18); reminder day counts in the freelancer's time zone (XBR-15); feasibility FEAT-15 risk on 0- and 3-decimal currencies |
| Recommended | Native Intl APIs (`Intl.NumberFormat`, `Intl.DateTimeFormat`) for all currency, date and time-zone formatting; money stored as integer minor units plus ISO 4217 code with the minor-unit exponent derived from Intl; IANA time zone stored on the freelancer account; viewer time zone read from the browser (cookie for server rendering); Postgres `AT TIME ZONE` for day-boundary arithmetic in reminder sweeps; English copy kept in per-feature copy modules, no translation framework at launch |
| Rationale | The landscape lists Native Intl as "currency, date and time-zone formatting with no translation layer" at zero cost and zero lock-in — exactly the demand, since no spec needs translation. Integer minor units with Intl-derived exponents handle JPY/KWD-style currencies correctly in tax-line rounding and Stripe amounts (feasibility FEAT-15). Keeping strings in copy modules leaves a clean seam for next-intl if locales are added. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| next-intl | Open source (MIT) | Low | High | Medium — Next.js-bound | Mainstream | A second UI language is scheduled — adopt it then over the existing copy modules |
| i18next / react-i18next | Open source (MIT) | Low | High | Low | Mainstream | Translations must be shared with a non-Next.js surface (e.g. email rendering in another service) |
| Lingui | Open source (MIT) | Medium — macros and extraction | High | Low — ICU/PO | ICU workflows | Several languages with professional translators using PO-file workflows are planned |

### Electronic Signature Attestation

| Field | Value |
|-------|-------|
| Context | Product-mandated — feature-dependency-map.md External Touchpoints: "Electronic-signature attestation for legally binding proposal signing — v1 phase ... FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability — submits the signature data captured by FEAT-26.SPEC-002 for attestation ... the jurisdictional standard it must meet is left to Stage 4)"; feasibility FEAT-26 verdict "Research-spike recommended" on (1) the legal standard and (2) whether a provider attests externally captured signatures; Nice-to-Have, opt-in per proposal (XBR-34), at most one signature per accepted proposal; ~2 s client interaction (ASMP-21); budget ~$100/month |
| Recommended | Documenso via its API — Individual tier ($30/month) for the spike and low v1 volume, Platform tier ($250/month) only if white-labeled embedded signing inside the branded portal is required at volume, with a self-hosted Documenso instance as the cost exit. Target standard for design: simple electronic signature under US ESIGN/UETA and EU eIDAS SES (the spike confirms whether any launch market needs eIDAS Advanced) |
| Rationale | Documenso is the only landscape option that is open source and self-hostable, so it can meet the budget at v1's low volume (at most one signature per opted-in proposal) and gives a path to attest signature data captured in Clientroom's own step if the hosted API cannot — the interaction-model risk the feasibility assessment raises. DocuSign's embedded tier (~$480/month) alone exceeds the infrastructure budget. The vendor is decided here; the FEAT-26 spike (3–5 days) is scoped to integration mechanics: confirming the interaction model (external-capture attestation vs embedded Documenso ceremony) and the per-signature cost. If no provider attests external capture, FEAT-26.SPEC-001 embeds the Documenso signing view; the timestamped Accept (FEAT-03, XBR-34) remains the fallback evidence. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Dropbox Sign API | From ~$100/month billed monthly | Medium — embedded signing, webhooks | High | Medium — SaaS | Mainstream | The spike shows Documenso's embedded flow cannot meet the ~2 s mobile target or its audit certificate is not accepted in a launch market |
| DocuSign eSignature API | Essentials $75/month for 50 requests; embedded signing on a ~$480/month tier | Medium–High | Very high | High — per-envelope model | DocuSign platform | A launch market or enterprise client base requires eIDAS Advanced/Qualified signatures or DocuSign-branded audit certificates, and the budget is raised accordingly |
| SignWell API | Plan-based | Medium | Medium | Medium — SaaS | Mainstream | Volume stays very low and a simpler hosted API with lower-tier pricing than Dropbox Sign is confirmed in the spike |

### Custom Domain Verification & TLS Serving

| Field | Value |
|-------|-------|
| Context | Product-mandated — assumptions-constraints.md ASMP-32: "Domain-verification capability (Later phase) — Custom Domain per Freelancer (FEAT-27) requires the ability to verify that a freelancer controls a domain and serve her portal securely at it." (FEAT-27.SPEC-002); at most one domain per freelancer, a few thousand accounts; shared default address always the fallback (FEAT-27.SPEC-003, XBR-35); landscape Cross-Area note: choice is coupled to hosting |
| Recommended | Vercel Domains API on the production project — add the freelancer's hostname, return the DNS records she must set, poll/re-check verification, automatic TLS issuance; Next.js middleware maps the request host to the freelancer's portal (`/portal/[handle]` rewrite) and falls back to the shared address on any failure |
| Rationale | With hosting on Vercel (ADR-029), the landscape rates this option "Low if hosted on Vercel" and "included in Vercel plans", keeping this Later-phase feature at zero added cost and no extra proxy in the request path. The per-plan domain limit is not quantified in the landscape (feasibility FEAT-27 risk) and must be verified before launch of FEAT-27; the choose-instead condition below covers exceeding it. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Cloudflare for SaaS (Custom Hostnames) | 100 hostnames free; then $0.10/hostname/month to 50,000 | Medium — API, DNS, fallback origin | Very high (50,000 hostnames) | Medium — traffic through Cloudflare | DNS/CDN | Verified custom domains exceed the Vercel plan's domain limit, or hosting moves off Vercel |
| Caddy on-demand TLS | Free; proxy server cost (~$5–12/month) | Medium–High — operate a proxy | High | Low — open source | Proxy/TLS ops | Hosting moves to self-managed containers where a proxy already fronts the app |
| Fly.io custom domain certificates | Included with Fly usage; per-app certificate limits | Medium | Medium–High | Medium — Fly-coupled | Fly platform | Hosting moves to Fly.io (Section 13 alternative) |

## 5. Project Structure

Suggested starting structure for the recommended stack (Next.js 16 App Router, ADR-001), adapted from the reference Next.js tree. It is a recommendation that fits the stack's conventions, not a binding layout. Two adaptations are deliberate (ADR-019, ADR-021): URLs are resource-oriented within three route groups (freelancer workspace, client portal, operator console) instead of feature-named, while all feature logic stays isolated under `src/features/feat-NN-slug/`; and Trigger.dev tasks live in `src/jobs/` inside the same package so jobs and web share one schema and rule set.

### Directory Tree

```
clientroom/
├── src/
│   ├── app/                                   # Next.js App Router — routing only; pages call into src/features
│   │   ├── layout.tsx                         # Root layout: html/body, fonts, providers (TanStack Query, toasts)
│   │   ├── globals.css                        # Tailwind v4 @import + @theme tokens (neutral base palette)
│   │   ├── error.tsx / not-found.tsx          # Root error boundary and 404
│   │   ├── (public)/                          # Unauthenticated pages (CDN-cacheable)
│   │   │   ├── page.tsx                       # Marketing/landing  (/)
│   │   │   ├── signup/page.tsx                # FEAT-20.SPEC-001   (/signup)
│   │   │   ├── sign-in/page.tsx               # Freelancer sign-in (/sign-in)
│   │   │   └── r/[referralToken]/page.tsx     # FEAT-33.SPEC-003   (/r/…)
│   │   ├── (freelancer)/app/                  # Freelancer workspace shell (/app/…) — sidebar + top bar
│   │   │   ├── layout.tsx                     # requireFreelancer(); nav; support-session banner
│   │   │   ├── dashboard/                     # FEAT-12
│   │   │   ├── clients/                       # FEAT-01, FEAT-18 (contacts nested under a client)
│   │   │   ├── projects/[projectId]/          # FEAT-01, 02, 04, 06, 13, 15, 25 (project-scoped screens)
│   │   │   ├── milestones/[milestoneId]/      # FEAT-07, FEAT-08 (freelancer side)
│   │   │   ├── deliverables/[deliverableId]/  # FEAT-07, FEAT-17 (freelancer side)
│   │   │   ├── invoices/                      # FEAT-09, 10, 11, 25
│   │   │   ├── records/…/print/               # FEAT-13.SPEC-002 printable copies
│   │   │   ├── exports/accounting/            # FEAT-22
│   │   │   ├── search/ notifications/ help/ support/ onboarding/   # FEAT-28, 29, 30, 31, 20
│   │   │   └── settings/                      # FEAT-15, 16, 19, 21, 23, 24, 27, 32
│   │   ├── (portal)/portal/[handle]/          # Client portal shell (/portal/{handle}/…), branded per freelancer
│   │   │   ├── layout.tsx                     # resolve freelancer by handle/host; inject --brand vars; referral mark
│   │   │   ├── sign-in/ verify/               # FEAT-05 (public within the portal)
│   │   │   ├── page.tsx                       # FEAT-05.SPEC-003 Portal Home
│   │   │   ├── projects/[projectId]/          # FEAT-03, FEAT-04, FEAT-26
│   │   │   ├── milestones/[milestoneId]/      # FEAT-07, FEAT-08
│   │   │   ├── deliverables/[deliverableId]/  # FEAT-07, FEAT-17
│   │   │   ├── invoices/[invoiceId]/          # FEAT-09, FEAT-10
│   │   │   ├── team/invite/                   # FEAT-18.SPEC-004
│   │   │   └── help/                          # FEAT-30.SPEC-003
│   │   ├── (ops)/ops/                         # Operator console (/ops/…) — requireOperator()
│   │   │   └── support/                       # FEAT-31.SPEC-002
│   │   └── api/                               # Route Handlers — machine endpoints only
│   │       ├── webhooks/[provider]/route.ts   # stripe | stripe-connect | resend | documenso → inbound_event ledger
│   │       ├── uploads/route.ts               # multipart session create/part-URL/complete (FEAT-16)
│   │       ├── files/[versionId]/route.ts     # authorized short-lived download grant (FEAT-16.SPEC-003)
│   │       └── auth/[...all]/route.ts         # Better Auth handler (freelancer/operator realm)
│   ├── middleware.ts                          # custom-domain host → /portal/[handle] rewrite; session cookie presence checks
│   ├── features/                              # One folder per Stage 3 feature (see mapping table)
│   │   └── feat-NN-{slug}/
│   │       ├── components/                    # Feature UI (client/server components)
│   │       ├── queries/                       # Account-scoped read functions used by RSC pages
│   │       ├── actions/                       # Server actions + automation handlers (Automation specs)
│   │       ├── rules/                         # Pure rule functions (Logic/Rule specs), shared client/server
│   │       └── copy.ts                        # English UI strings for the feature
│   ├── jobs/                                  # Trigger.dev v3 tasks and schedules (trigger.config.ts at root)
│   │   ├── outbox-dispatch.ts                 # 1-minute sweep of job_outbox
│   │   ├── inbound-events.ts                  # process webhook ledger rows in event-time order
│   │   └── feat-NN-{slug}/…                   # feature tasks (e.g. feat-11 reminder sweep, feat-24 archive)
│   ├── shared/
│   │   ├── components/ui/                     # shadcn/ui copy-in components (Radix + Tailwind)
│   │   ├── components/                        # AppShell, PortalShell, EmptyState, Skeletons, OfflineBanner…
│   │   ├── hooks/                             # useOnline, useVersionedMutation, useToast
│   │   ├── lib/
│   │   │   ├── db.ts                          # Drizzle client (Neon Pool) + read-only client for support sessions
│   │   │   ├── authz/                         # requireFreelancer/requirePortalContact/requireOperator, defineAction()
│   │   │   ├── auth/                          # Better Auth config; portal token + session helpers
│   │   │   ├── money.ts / datetime.ts         # Intl formatting, minor units, time-zone helpers
│   │   │   ├── outbox.ts                      # enqueue side effects inside a transaction
│   │   │   ├── integrations/                  # stripe.ts, stripe-connect.ts, r2.ts, resend.ts, documenso.ts, vercel-domains.ts, link-check.ts
│   │   │   ├── notifications/                 # email.ts (dispatch), templates/ (React Email, one per Notification spec)
│   │   │   └── telemetry/                     # sentry.ts, posthog.ts (server capture), logger.ts
│   │   └── styles/                            # brand.ts (contrast-adjusted --brand computation, FEAT-19.SPEC-002)
│   └── db/
│       ├── schema/                            # Drizzle schema, one file per entity group (see database-schema.md)
│       └── seed.ts                            # Development seed data
├── drizzle/                                   # drizzle-kit generated SQL migrations (committed)
├── tests/                                     # e2e (Playwright) and integration tests incl. support-session mutation sweep
├── public/                                    # Static assets
├── trigger.config.ts                          # Trigger.dev project config (points at src/jobs)
├── drizzle.config.ts
├── next.config.ts
├── package.json / pnpm-lock.yaml
└── tsconfig.json                              # strict; path aliases (Section 12)
```

### Feature-to-Directory Mapping

| Feature | Stage 3 Folder | Source Directory | Notes |
|---------|---------------|-----------------|-------|
| FEAT-01 (Client & Project Management) | FEAT-01-client-project-management/ | src/features/feat-01-client-project-management/ | Routes: `/app/clients/**`, `/app/projects/[projectId]` |
| FEAT-02 (Proposal Creation & Sending) | FEAT-02-proposal-creation-sending/ | src/features/feat-02-proposal-creation-sending/ | Routes: `/app/projects/[projectId]/proposal/**` |
| FEAT-03 (Proposal Acceptance) | FEAT-03-proposal-acceptance/ | src/features/feat-03-proposal-acceptance/ | Portal routes `/portal/[handle]/projects/[projectId]/proposal/**`; acceptance action creates the deposit invoice in-transaction via FEAT-09 rules |
| FEAT-04 (Milestone & Payment Schedule Setup) | FEAT-04-milestone-payment-schedule-setup/ | src/features/feat-04-milestone-payment-schedule-setup/ | Offline-queued save uses the shared versioned-mutation hook |
| FEAT-05 (Client Portal Access (Magic-Link Login)) | FEAT-05-client-portal-access-magic-link-login/ | src/features/feat-05-client-portal-access-magic-link-login/ | Portal token/session helpers in src/shared/lib/auth/portal.ts |
| FEAT-06 (Deliverable Upload & Sharing) | FEAT-06-deliverable-upload-sharing/ | src/features/feat-06-deliverable-upload-sharing/ | Upload UI uses FEAT-16 multipart client; link reachability check as a job (src/jobs/feat-06-…) |
| FEAT-07 (Deliverable Review & Feedback) | FEAT-07-deliverable-review-feedback/ | src/features/feat-07-deliverable-review-feedback/ | Offline comment queue (persisted TanStack mutations) |
| FEAT-08 (Milestone Approval) | FEAT-08-milestone-approval/ | src/features/feat-08-milestone-approval/ | Approval action = version-guarded update + next invoice in one transaction |
| FEAT-09 (Invoice Generation & Sending) | FEAT-09-invoice-generation-sending/ | src/features/feat-09-invoice-generation-sending/ | `rules/numbering.ts` owns the locked per-freelancer counter; generation callable from FEAT-01/03/08 |
| FEAT-10 (Invoice Payment Processing) | FEAT-10-invoice-payment-processing/ | src/features/feat-10-invoice-payment-processing/ | Payment Element client component; event processing in src/jobs/inbound-events.ts |
| FEAT-11 (Automated Payment Reminders) | FEAT-11-automated-payment-reminders/ | src/features/feat-11-automated-payment-reminders/ | Hourly reminder sweep task in src/jobs/feat-11-automated-payment-reminders/ |
| FEAT-12 (Freelancer Financial Dashboard) | FEAT-12-freelancer-financial-dashboard/ | src/features/feat-12-freelancer-financial-dashboard/ | Totals refresh + overdue sweep jobs |
| FEAT-13 (Immutable Activity & Audit Trail) | FEAT-13-immutable-activity-audit-trail/ | src/features/feat-13-immutable-activity-audit-trail/ | `recordActivity()` helper called inside originating transactions; print routes |
| FEAT-14 (Notifications (Email)) | FEAT-14-notifications-email/ | src/features/feat-14-notifications-email/ | Dispatch/entitlement rules here; delivery module and templates in src/shared/lib/notifications/ |
| FEAT-15 (Currency & Tax Handling) | FEAT-15-currency-tax-handling/ | src/features/feat-15-currency-tax-handling/ | Formatting primitives in src/shared/lib/money.ts and datetime.ts |
| FEAT-16 (Large File Handling & Storage) | FEAT-16-large-file-handling-storage/ | src/features/feat-16-large-file-handling-storage/ | Multipart client + `/api/uploads`, `/api/files`; R2 client in src/shared/lib/integrations/r2.ts |
| FEAT-17 (Deliverable Version History) | FEAT-17-deliverable-version-history/ | src/features/feat-17-deliverable-version-history/ | Reuses FEAT-16 upload path; one object key per version |
| FEAT-18 (Client Contact Management & Roles) | FEAT-18-client-contact-management-roles/ | src/features/feat-18-client-contact-management-roles/ | Removal revokes portal sessions and outstanding tokens in the same transaction |
| FEAT-19 (Freelancer Branding) | FEAT-19-freelancer-branding/ | src/features/feat-19-freelancer-branding/ | Contrast computation in src/shared/styles/brand.ts |
| FEAT-20 (Onboarding / First-Run Setup) | FEAT-20-onboarding-first-run-setup/ | src/features/feat-20-onboarding-first-run-setup/ | Sign-up transaction also provisions the free plan (FEAT-23) and referral record (FEAT-33) |
| FEAT-21 (Settings & Account Management) | FEAT-21-settings-account-management/ | src/features/feat-21-settings-account-management/ | Session list/revocation via Better Auth APIs |
| FEAT-22 (Accounting Export) | FEAT-22-accounting-export/ | src/features/feat-22-accounting-export/ | Format writers per target (CSV, QuickBooks, Xero) in `rules/formats/` |
| FEAT-23 (Subscription Plan & Billing Management) | FEAT-23-subscription-plan-billing-management/ | src/features/feat-23-subscription-plan-billing-management/ | Stripe Billing client; stop-billing relay task |
| FEAT-24 (Data Export & Account Deletion) | FEAT-24-data-export-account-deletion/ | src/features/feat-24-data-export-account-deletion/ | Archive build, staged deletion and retention purge as Trigger.dev tasks |
| FEAT-25 (Refund & Cancelled Project Handling) | FEAT-25-refund-cancelled-project-handling/ | src/features/feat-25-refund-cancelled-project-handling/ | Consumes reversal events relayed from FEAT-32 processing |
| FEAT-26 (Legally Binding E-Signature for Proposals) | FEAT-26-legally-binding-e-signature-for-proposals/ | src/features/feat-26-legally-binding-e-signature-for-proposals/ | Documenso client in src/shared/lib/integrations/documenso.ts |
| FEAT-27 (Custom Domain per Freelancer) | FEAT-27-custom-domain-per-freelancer/ | src/features/feat-27-custom-domain-per-freelancer/ | Host resolution in src/middleware.ts; Vercel Domains client |
| FEAT-28 (Global Search Across Clients & Projects) | FEAT-28-global-search-across-clients-projects/ | src/features/feat-28-global-search-across-clients-projects/ | SQL search + ranking in `queries/search.ts` |
| FEAT-29 (In-App Notification Center) | FEAT-29-in-app-notification-center/ | src/features/feat-29-in-app-notification-center/ | Persisted last-loaded feed; cleared on sign-out |
| FEAT-30 (Contextual Help & Guidance) | FEAT-30-contextual-help-guidance/ | src/features/feat-30-contextual-help-guidance/ | Static help content as MDX/TS modules; tooltip component |
| FEAT-31 (Operator Support Access) | FEAT-31-operator-support-access/ | src/features/feat-31-operator-support-access/ | Read-only enforcement in src/shared/lib/authz/; auto-close task |
| FEAT-32 (Payment Account Connection) | FEAT-32-payment-account-connection/ | src/features/feat-32-payment-account-connection/ | Stripe Connect onboarding links; account-status event handling |
| FEAT-33 (Portal Referral Attribution) | FEAT-33-portal-referral-attribution/ | src/features/feat-33-portal-referral-attribution/ | Referral mark component used by PortalShell and email templates |

### Spec-Type-to-Location Mapping

| Spec Type | File Location Pattern | Example |
|-----------|----------------------|---------|
| Screen | `src/app/(group)/…/page.tsx` (thin route file) + `src/features/feat-NN-{slug}/components/` | FEAT-08.SPEC-001 → `src/app/(portal)/portal/[handle]/milestones/[milestoneId]/page.tsx` rendering `src/features/feat-08-milestone-approval/components/MilestoneApprovalView.tsx` |
| Automation | `src/features/feat-NN-{slug}/actions/{action}.ts` for request-driven automations; `src/jobs/feat-NN-{slug}/{task}.ts` for scheduled/retried/long-running ones | FEAT-03.SPEC-003 → `src/features/feat-03-proposal-acceptance/actions/record-acceptance.ts`; FEAT-11.SPEC-001 → `src/jobs/feat-11-automated-payment-reminders/reminder-sweep.ts` |
| Logic/Rule | `src/features/feat-NN-{slug}/rules/{rule}.ts` (pure functions, imported by actions, jobs and client validation) | FEAT-09.SPEC-007 → `src/features/feat-09-invoice-generation-sending/rules/numbering.ts` |
| Integration | `src/shared/lib/integrations/{service}.ts` (one client module per external service) + webhook entry in `src/app/api/webhooks/[provider]/route.ts` | FEAT-10.SPEC-003 → `src/shared/lib/integrations/stripe-connect.ts`; FEAT-16.SPEC-007 → `src/shared/lib/integrations/r2.ts` |
| Notification | `src/shared/lib/notifications/templates/{notification}.tsx` (React Email template) dispatched by `src/shared/lib/notifications/email.ts`; the feature's action enqueues it via the outbox | FEAT-11.SPEC-004 → `src/shared/lib/notifications/templates/overdue-reminder.tsx` |

## 6. Data Layer Design

See `.n2b/architecture/database-schema.md` for the complete schema design.

**Migration strategy:** migration-based. With 23 entities, 60 relationships and immutable evidentiary records (ASMP-25) on Neon Postgres via Drizzle (ADR-003, ADR-004), every schema change is generated by `drizzle-kit generate` as a reviewed, committed SQL file and applied by `drizzle-kit migrate` in CI to preview branches, staging and production — push-based auto-sync would risk silent destructive changes to financial records and cannot be reviewed or replayed across environments.

## 7. API & Routing Architecture

### Route Map

| Screen Spec | URL Path | Parameters | Data Requirements |
|-------------|----------|------------|-------------------|
| FEAT-01.SPEC-001 (Add Client) | /app/clients/new | -- | Plan active-client count and limit (FEAT-01.SPEC-008); account default currency |
| FEAT-01.SPEC-002 (Create Project) | /app/clients/[clientId]/projects/new | clientId | Client summary; billing completeness (FEAT-01.SPEC-010) |
| FEAT-01.SPEC-003 (Client & Project Roster) | /app/clients | ?status=, ?q= | Clients with active projects, derived project stage; filter state |
| FEAT-01.SPEC-004 (Client Detail) | /app/clients/[clientId] | clientId | Client record + version token, projects, contacts summary, open items |
| FEAT-01.SPEC-005 (Project Detail (Open Project)) | /app/projects/[projectId] | projectId | Project + stage, proposal status, milestones, deliverables, invoices summary, version token |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | /app/projects/[projectId]/proposal/edit | projectId | Current draft + version token; currency set; Primary contact existence |
| FEAT-02.SPEC-002 (Proposal Preview) | /app/projects/[projectId]/proposal/preview | projectId | Draft rendered with branding as the client will see it |
| FEAT-02.SPEC-003 (Proposal Detail) | /app/projects/[projectId]/proposal | projectId, ?version= | Proposal versions, status, acceptance/signature evidence, change-request comments |
| FEAT-02.SPEC-004 (Reuse Proposal Picker) | /app/projects/[projectId]/proposal/reuse | projectId, ?q= | Freelancer's prior proposals (indexed, paginated) |
| FEAT-03.SPEC-001 (Proposal Review & Accept) | /portal/[handle]/projects/[projectId]/proposal | handle, projectId | Sent proposal version + status token; contact role (Primary only); branding |
| FEAT-03.SPEC-002 (Request Changes) | /portal/[handle]/projects/[projectId]/proposal/request-changes | handle, projectId | Proposal id/version; contact identity |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | /app/projects/[projectId]/schedule | projectId | Milestones, payment structure, lock state (approved/invoiced), version tokens |
| FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | /portal/[handle]/projects/[projectId]/milestones | handle, projectId | Milestones with status and due dates (viewer-local time) |
| FEAT-05.SPEC-001 (Request Sign-In Link) | /portal/[handle]/sign-in | handle | Freelancer branding only (neutral outcome, no account existence leak) |
| FEAT-05.SPEC-002 (Link Verification Landing) | /portal/[handle]/verify | handle, ?token= | Token validity check (read-only on GET); confirm button consumes via POST |
| FEAT-05.SPEC-003 (Portal Home) | /portal/[handle] | handle (or custom domain host) | Contact's client projects, items awaiting action, open invoices (role-filtered) |
| FEAT-06.SPEC-001 (Deliverable Upload) | /app/projects/[projectId]/milestones/[milestoneId]/deliverables/new | projectId, milestoneId | Storage allowance and ceiling; multipart session state for resume |
| FEAT-06.SPEC-002 (Deliverable List & Management) | /app/projects/[projectId]/deliverables | projectId | Deliverables by milestone, latest version, link-check status, removal eligibility |
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | /portal/[handle]/deliverables/[deliverableId]/versions/[versionNo] (freelancer: /app/deliverables/[deliverableId]/versions/[versionNo]) | handle, deliverableId, versionNo | Version media grant, version-anchored comments, viewer role, offline queue state |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | /portal/[handle]/milestones/[milestoneId]/comments (freelancer: /app/milestones/[milestoneId]/comments) | handle, milestoneId | Milestone-level comments, viewer role |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | /portal/[handle]/milestones/[milestoneId] | handle, milestoneId | Milestone, deliverable set, price, state token covering all three (FEAT-08.SPEC-003) |
| FEAT-08.SPEC-002 (Milestone Reopen Screen) | /app/milestones/[milestoneId]/reopen | milestoneId | Approved milestone, approval evidence, dependent invoice state |
| FEAT-09.SPEC-001 (Invoice List) | /app/invoices | ?status=, ?client=, ?currency= | Invoices with paid/due/overdue status per currency |
| FEAT-09.SPEC-002 (Invoice Detail) | /app/invoices/[invoiceId] (client: /portal/[handle]/invoices/[invoiceId]) | invoiceId, handle | Immutable invoice, payments, credit notes, pay-link availability (FEAT-09.SPEC-009) |
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | /app/invoices/new | ?projectId=, ?type=invoice\|credit-note, ?corrects= | Project currency/tax, business details completeness, invoice being corrected |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | /portal/[handle]/invoices/[invoiceId]/pay | handle, invoiceId | Invoice amount/currency, Stripe PaymentIntent client secret on the connected account, available methods |
| FEAT-10.SPEC-002 (Record Off-Platform Payment Screen) | /app/invoices/[invoiceId]/record-payment | invoiceId | Invoice status + version token, amount due |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | /app/invoices/[invoiceId]/reminders | invoiceId | Reminder Log, pause state, next scheduled reminder, manual-send eligibility today |
| FEAT-12.SPEC-001 (Dashboard Overview) | /app/dashboard | -- | Held per-currency totals (earned/outstanding/overdue); last-loaded cache |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | /app/dashboard/[scopeType]/[scopeId] | scopeType (client\|project), scopeId | Per-scope per-currency totals and contributing invoices |
| FEAT-13.SPEC-001 (Activity Trail) | /app/projects/[projectId]/activity (account-wide: /app/activity) | projectId | Append-only entries, paginated newest-first, role-filtered |
| FEAT-13.SPEC-002 (Printable Record Copy) | /app/records/[recordType]/[recordId]/print (client copies: /portal/[handle]/records/[recordType]/[recordId]/print) | recordType, recordId, handle | Immutable record snapshot rendered with print stylesheet |
| FEAT-15.SPEC-001 (Project Currency & Tax Configuration) | /app/projects/[projectId]/settings/currency-tax | projectId | Currency, tax label/rate, lock state (first invoice issued) |
| FEAT-15.SPEC-002 (Freelancer Time Zone Setting) | /app/settings/time-zone | -- | Account IANA time zone; zone list |
| FEAT-16.SPEC-001 (Storage Usage Summary) | /app/settings/storage | -- | Last aggregated usage, allowance, 80% warning state |
| FEAT-17.SPEC-001 (New Version Upload) | /app/deliverables/[deliverableId]/versions/new | deliverableId | Current version number, allowance, multipart session state |
| FEAT-17.SPEC-002 (Version Browser & Comparison) | /portal/[handle]/deliverables/[deliverableId]/versions (freelancer: /app/deliverables/[deliverableId]/versions) | handle, deliverableId, ?compare=a,b | Version list, per-version media grants |
| FEAT-18.SPEC-001 (Client Contact List) | /app/clients/[clientId]/contacts | clientId | Contacts with roles, invitation status |
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | /app/clients/[clientId]/contacts/new and /app/clients/[clientId]/contacts/[contactId]/edit | clientId, contactId | Contact record + version token; uniqueness per client |
| FEAT-18.SPEC-003 (Remove Client Contact) | /app/clients/[clientId]/contacts/[contactId]/remove | clientId, contactId | Contact, last-Primary check, evidence records referencing the contact |
| FEAT-18.SPEC-004 (Invite Reviewer Colleague) | /portal/[handle]/team/invite | handle | Inviting Primary contact's client; existing contacts |
| FEAT-19.SPEC-001 (Branding Settings) | /app/settings/branding | -- | Branding Profile (logo URL, colour), computed legible variant, preview |
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | /signup | ?ref= (referral carried in cookie) | Referral capture state; time-zone guess from browser |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | /app/onboarding/[step] | step | Onboarding progress on the account; step prerequisites |
| FEAT-21.SPEC-001 (Account Profile) | /app/settings/profile | -- | Freelancer Account profile fields + version |
| FEAT-21.SPEC-002 (Notification Preferences) | /app/settings/notifications | -- | Optional-email preferences (mandatory types shown locked) |
| FEAT-21.SPEC-003 (Login & Security) | /app/settings/security | -- | Sign-in email, pending change, login methods, active sessions |
| FEAT-21.SPEC-004 (Business Details & Payment Terms) | /app/settings/business | -- | Business name, address, tax ID, default payment terms |
| FEAT-22.SPEC-001 (Accounting Export Screen) | /app/exports/accounting | ?from=, ?to=, ?format= | Invoice/payment date range availability; currencies present |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | /app/settings/plan | -- | Subscription Plan state, grace window, limits, Stripe Billing portal/links |
| FEAT-24.SPEC-001 (Data Export Screen) | /app/settings/data-export | -- | Active archive status, download window |
| FEAT-24.SPEC-002 (Account Deletion Screen) | /app/settings/delete-account | -- | Pre-deletion warnings, retention classification, deletion state |
| FEAT-25.SPEC-001 (Mark Invoice Refunded Screen) | /app/invoices/[invoiceId]/refund | invoiceId | Invoice payments, amount paid, version token |
| FEAT-25.SPEC-002 (Mark Project Cancelled Screen) | /app/projects/[projectId]/cancel | projectId | Project state, open invoices/milestones, version token |
| FEAT-26.SPEC-001 (Signature Signing Step) | /portal/[handle]/projects/[projectId]/proposal/sign | handle, projectId | Proposal version opted in to e-signature, Primary role, attestation status |
| FEAT-27.SPEC-001 (Custom Domain Settings) | /app/settings/domain | -- | Custom Domain Record, verification state, required DNS records |
| FEAT-28.SPEC-001 (Global Search) | /app/search | ?q= | Account-scoped cross-entity results with secondary context |
| FEAT-29.SPEC-001 (Notification Center Feed) | /app/notifications | ?cursor= | 90-day feed items with read state; persisted last-loaded copy |
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | Overlay within host routes /app/** and /portal/[handle]/** (no own URL) | tipId (component prop) | Tip content, per-user dismissal flag |
| FEAT-30.SPEC-002 (Freelancer Help Reference) | /app/help | ?topic= | Static help content |
| FEAT-30.SPEC-003 (Client Portal Help Reference) | /portal/[handle]/help | handle | Static client help content, branding |
| FEAT-31.SPEC-001 (Contact Support Screen) | /app/support | -- | Freelancer's support requests and past sessions |
| FEAT-31.SPEC-002 (Operator Support Session Console) | /ops/support (session view: /ops/support/sessions/[sessionId]) | sessionId | Request queue; active session with target account, inactivity timer |
| FEAT-32.SPEC-001 (Payment Connection Screen) | /app/settings/payments | ?return=, ?refresh= (Stripe onboarding return) | Payment Account Connection status, attention reason, methods |
| FEAT-33.SPEC-003 (Referral Landing Page) | /r/[referralToken] | referralToken (opaque) | Static landing content; sets referral capture cookie (30-minute window) |

### API Endpoint Convention

User-initiated mutations are **server actions**, one per Automation spec entry point, named `verbNoun` (e.g. `acceptProposal`, `recordOffPlatformPayment`), each wrapped in `defineAction({ realm, role, input, run })` which enforces authentication, role, account/client scoping and the support-session read-only rule before `run` executes, and returns a discriminated result: `{ ok: true, data }`, `{ ok: false, kind: 'conflict', current }` (reject-with-refresh, carrying the fresh record and version), `{ ok: false, kind: 'validation', fieldErrors }`, or `{ ok: false, kind: 'forbidden' | 'offline-required' }`. **Route Handlers** exist only for machine-facing endpoints under `/api/`: `POST /api/webhooks/{provider}` (signature-verified, returns 200 after persisting to the inbound-event ledger), `POST /api/uploads` and `POST /api/uploads/{sessionId}/parts|complete` (multipart session control), `GET /api/files/{versionId}` (302 to a short-lived presigned URL after authorization) and `/api/auth/*` (Better Auth). Resource names are plural kebab-case nouns; JSON bodies use camelCase; money is `{ amountMinor: number, currency: 'EUR' }`; timestamps are ISO-8601 UTC strings. No public REST API is exposed.

### Data Fetching Strategy

| Route Type | Fetching Approach | Rationale |
|-----------|-------------------|-----------|
| List pages (roster, invoices, deliverables, activity, reuse picker) | React Server Components call account-scoped `queries/` functions directly; cursor pagination; filters in URL search params; `loading.tsx` skeletons stream while data loads | Keeps client bundles small and HTML complete on first paint for the ~2 s target (ASMP-21); URL-held filters survive reloads; lists are snapshots (Real-time = No) |
| Detail pages (project, proposal, milestone approval, invoice, deliverable version) | RSC fetch of the record plus its version/state token; client islands only where interaction or offline needs demand (comment thread, schedule editor) hydrate TanStack Query with the server data | The version token is what the reject-with-refresh guard compares (ADR-014); hydration avoids a second round-trip on mobile |
| Form submissions | Server actions via `useActionState` with progressive enhancement; the version token is a hidden field; `conflict` results re-render with the current record and keep typed input (ASMP-27); record-creating actions (accept, approve, pay, send) require connectivity and are never queued | Exactly-once and stale-state protection live on the server; ASMP-27 forbids pretending offline success |
| Offline-capable islands (comment thread, schedule save, dashboard, notification feed) | TanStack Query with IndexedDB persistence: paused mutations replay in order on reconnect (FEAT-07.SPEC-008, FEAT-04.SPEC-001 AC-15); last-loaded snapshots shown read-only while offline (FEAT-12.SPEC-001, FEAT-29.SPEC-004) | Serves the Offline signal's four specs without an offline-first architecture |
| File transfer | Browser-direct to R2 via presigned multipart part URLs; downloads via `/api/files/{versionId}` 302 to presigned GET | Keeps multi-GB bytes off serverless functions; authorization checked per viewer, operator excluded (XBR-29) |
| Public pages (landing, signup, referral landing, help references) | Static/ISR via Next.js cache on the Vercel CDN | Fast worldwide first paint at zero extra cost (ADR-013) |

### Navigation Model

Three shells with separate navigation. **Freelancer workspace (`/app`)**: a left sidebar on desktop (collapsing to a bottom-sheet menu under 768 px) with Dashboard, Clients, Invoices, Search, Notifications, Help and Settings; project context is entered from a client or search result and exposes project tabs (Overview, Proposal, Schedule, Deliverables, Invoices, Activity). URLs nest by ownership (`/app/clients/[clientId]/contacts`, `/app/projects/[projectId]/schedule`); records reached from several places (milestones, deliverables, invoices) have flat canonical URLs to keep links in emails stable. The Settings area groups account, business, branding, time zone, notifications, security, plan, payments, storage, domain, data export and deletion. **Client portal (`/portal/[handle]`, or the freelancer's verified custom domain at `/`)**: mobile-first, a single Home listing the contact's projects and "waiting on you" items, with project → proposal / milestones / deliverables / invoices drill-down and a persistent header carrying the freelancer's branding, Help and sign-out; every notification email deep-links to the exact portal URL. **Operator console (`/ops`)**: a request queue and one active session view; while a support session is open, the operator browses the freelancer's `/app` routes in read-only mode with a persistent banner and a close-session control.

## 8. Integration Architecture

| Service | Purpose | Data Exchanged | Direction | Rate/Quota Notes | Failure-Mode Handling | Sandbox/Test Path |
|---------|---------|----------------|-----------|------------------|----------------------|-------------------|
| Stripe Connect (Standard accounts, direct charges) | Payment processing into each freelancer's own account — External Touchpoints row 1; FEAT-10.SPEC-003, FEAT-32.SPEC-002, relayed reversals for FEAT-25.SPEC-005 (ADR-010) | Out: account-link creation, PaymentIntents on the connected account (amount in minor units, currency, invoice reference — no card data); in: connected-account webhooks (payment succeeded/failed/processing, bank-transfer pending, dispute created/closed, account.updated readiness and methods). Payment references are GDPR-class; card data stays in Stripe (ASMP-24) | Bidirectional | Stripe fees charged to the connected account; API rate limits per account apply (verify current limits); webhooks retried by Stripe, deduplicated by event id in the ledger | FEAT-10.SPEC-003 Degradation Behavior: slow → "Still processing" after 10 s with no second submission; down → pay disabled, rest of invoice usable; rejects → decline reason and retry, no ambiguous Payment. FEAT-32.SPEC-002: slow → "Still checking" after 5 minutes; down → connect disabled, last status kept; reject → Error state with interim record deleted. Out-of-order events resolved by event time and payment-attempt id; processor state authoritative with discrepancy notice | Stripe test mode keys and test connected accounts; Stripe CLI `stripe listen` forwards webhooks to local/preview |
| Stripe Billing (platform account) | Subscription billing for the freelancer's own plan — External Touchpoints row 4; FEAT-23.SPEC-003 (ADR-010) | Out: customer, subscription create/update/cancel; in: invoice paid/failed, subscription updated/deleted, period-end events. Freelancer billing details are GDPR-class | Bidirectional | 0.7% of billing volume plus card fees | FEAT-23.SPEC-003 Degradation Behavior: slow → progress then "Still working"; down → action disabled, plan unaffected; rejects → reason inline, plan unchanged. Stop-billing relay retried every 15 minutes until acknowledged; 7-day grace window after failed renewal | Stripe test mode, test clocks for renewal/grace scenarios |
| Resend | Transactional email delivery with delivery and bounce status — External Touchpoints row 2; FEAT-14.SPEC-001 (ADR-009) | Out: rendered emails (recipient address, branded content, deep links incl. magic links); in: delivered/bounced/complained/failed webhooks. Recipient data GDPR-class | Bidirectional | Pro: 50,000 emails/month for $20; Free: 3,000/month | FEAT-14.SPEC-001 Degradation Behavior: no screen waits on email; outages leave Notifications Queued and retried; bounces never retried; transient failures retried up to 3 times within 6 hours, then FEAT-14.SPEC-006 warning to the freelancer. Duplicate/out-of-order events resolved by event time | Resend test API key and test recipient addresses; local preview of React Email templates |
| Cloudflare R2 | Large-file storage and delivery with version history — External Touchpoints row 3; FEAT-16.SPEC-007; export archives (FEAT-24.SPEC-003); branding logos (ADR-008) | Out: multipart create/complete/abort, presigned part and GET URLs, deletes; in: object bytes from browsers (direct), ListParts/HeadObject results. Deliverables may contain personal data | Bidirectional | $0.015/GB-month, zero egress; Class A $4.50/M, Class B $0.36/M; 10 GB free | FEAT-16.SPEC-007 Degradation Behavior: slow → progress with lengthening estimate; down → "Uploads aren't available right now" with the file kept selected; no half-created versions (version row written only after verified completion). Duplicate/out-of-order transfer events ignored once finalized; purge confirmations idempotent | Separate dev/preview bucket with its own API token; MinIO or LocalStack S3 container as local substitute |
| Documenso | Electronic-signature attestation — External Touchpoints row 6; FEAT-26.SPEC-005 (ADR-017) | Out: signed-proposal document, signer full legal name and contact email, signature data; in: attestation/completion webhook, signed copy with audit certificate. Signer identity GDPR-class, retained as evidence (XBR-27) | Bidirectional | Individual tier $30/month; Platform $250/month for white-label embedding | FEAT-26.SPEC-005 Degradation Behavior: slow → "Still working" after 10 s; down/rejects → inline error with the typed name preserved; no half-written signature. Voiding between submission and confirmation discards the confirmation; first outcome wins | Documenso test/free account (5 docs/month) or a self-hosted Documenso container locally |
| Vercel Domains API | Domain verification and secure serving at a freelancer's own domain (Later phase) — External Touchpoints row 5; FEAT-27.SPEC-002 (ADR-018) | Out: add/remove project domain, verification checks; in: verification status, required DNS records, certificate state. Domain name is account data | Bidirectional | Included in Vercel plan; per-plan domain limit to verify before FEAT-27 launch | FEAT-27.SPEC-002 Degradation Behavior: capability down disables add/re-check while the shared default address keeps serving (XBR-35); replaced-domain in-flight results disregarded; event-time ordering | Preview project with a test domain; verification simulated in unit tests |
| Figma / Google Drive / Dropbox public share links | Linked-asset reachability check for deliverables accepted by link (BRIEF.md ## Ecosystem & Integrations; FEAT-06.SPEC-004) | Out: unauthenticated HTTP HEAD/GET to the submitted URL; in: status code and limited body sniff to detect sign-in pages. No credentials held | Outbound | Human-paced (one check per attach plus periodic re-check); per-host concurrency capped at 2 with 10 s timeout | Check runs as a job; timeout or ambiguous (sign-in page) result flags the link and hides it from clients per FEAT-06.SPEC-004; treatment of sign-in pages remains feasibility Open Question 11 | Recorded HTTP fixtures in tests; no sandbox needed |
| Neon (Postgres) | Primary database (ADR-003) — architecture-introduced | All application data over TLS; GDPR-class records | Bidirectional | Launch plan usage-based ($0.106/CU-hour, $0.35/GB-month) | Connection failures surface as retryable errors; record-creating actions fail visibly (ASMP-27); point-in-time restore for recovery | Neon branch per preview deployment; Docker Postgres locally |
| Trigger.dev (Cloud) | Background jobs, schedules and long-running tasks (ADR-012) — architecture-introduced | Out: task triggers with ids and minimal payload (record ids, not personal data); tasks read data from Neon directly | Outbound (tasks call back into Neon/R2/Resend/Stripe) | Free 50K runs/month; paid from $10/month | Outbox rows persist until acknowledged; one-minute sweep re-dispatches; task retries with backoff per spec parameters | Trigger.dev dev environment with `trigger dev` running tasks locally |
| Better Auth (library, no external service) | Freelancer/operator identity (ADR-026) — runs in-process; listed for completeness | Session and account rows in Neon only | -- (in-process) | -- | Auth outage equals app outage; no third party | Same as local database |
| Sentry | Error tracking, tracing, cron and uptime monitors (ADR-031) — architecture-introduced | Out: error events and traces with PII scrubbing (no emails, names or document content) | Outbound | Team $26/month incl. 50K errors | SDK buffers and drops on outage; no user-facing impact | Separate Sentry environment tags for preview/staging |
| Grafana Cloud (Loki) | Structured log retention and alerting (ADR-031) — architecture-introduced | Out: Vercel log drain and Trigger.dev logs, PII-scrubbed | Outbound | Free tier; ~$0.50/GB ingested beyond | Log loss during outage tolerated; errors still in Sentry | Free-tier stack for non-production |
| PostHog Cloud | Product analytics (ADR-015) — architecture-introduced | Out: server-side milestone events keyed by opaque account id; no emails/names | Outbound | Free 1M events/month | Capture is fire-and-forget; never blocks requests | Separate PostHog project for non-production |

## 9. Design System Implementation Plan

**Design system posture:** None (`design_system_source: none`) — this blueprint is design-agnostic; the downstream builder owns visual design, honoring the design preferences recorded in the brief's Constraints. Styling and component decisions below are driven from product needs alone: ASMP-27 (readable on a phone without zooming, screen-reader and keyboard usable, never colour-only, brand colours kept legible), FEAT-19 per-freelancer logo and colour on every client screen and email, and the ~2 s mobile target (ASMP-21).

**Styling-system implementation approach (ADR-005):** Tailwind CSS v4 with a neutral, unbranded base palette, type scale and spacing defined once as `@theme` CSS custom properties in `src/app/globals.css`. The client portal layout computes a contrast-adjusted brand colour server-side (`src/shared/styles/brand.ts`, FEAT-19.SPEC-002 contrast ratio) and injects `--brand` and `--brand-foreground` on the portal root; components consume only those variables, so a freelancer's colour can never make text illegible. Email templates receive the same computed values as inline colours and a raster logo (feasibility FEAT-19 risk). Status is always conveyed by text or icon plus colour (ASMP-27).

### Component Library Decision

**shadcn/ui (copy-in components built on Radix UI primitives and Tailwind CSS), placed in `src/shared/components/ui/`.** Profile evidence justifies a component layer: 65 Screen specs with data tables (invoice list, deliverable list, activity trail), confirmation dialogs for record-creating and destructive actions (accept, approve, remove contact, delete account), popover tooltips (FEAT-30.SPEC-001), toasts, tabs and a multi-step onboarding sequence (FEAT-20.SPEC-002) — while Complex forms = No (≤7 inputs) means no heavyweight form suite is needed. Radix primitives provide the keyboard and screen-reader behaviour ASMP-27 requires, and the copy-in model keeps components visually unopinionated and fully owned, so the downstream builder can restyle them without fighting a vendor theme — the right posture for a design-agnostic package. shadcn/ui assumes React plus Tailwind (landscape Cross-Area note), matching ADR-001 and ADR-005. Alternatives considered from the landscape's component-layer list: raw Radix UI primitives (choose when the builder wants zero pre-styled markup), Headless UI (choose if a Vue frontend is adopted), Melt UI (choose if SvelteKit is adopted).

## 10. Shared Infrastructure Patterns

| Pattern | When Used | Approach | File Location | Dependencies |
|---------|-----------|----------|---------------|-------------|
| Layout system | Every route | Three App Router route-group layouts: `AppShell` (freelancer sidebar/top bar, support-session banner), `PortalShell` (branded header with logo, `--brand` vars, referral mark, help link; mobile-first single column), `OpsShell` (operator console). Nested project layout supplies project tabs. Layouts perform the realm guard (`requireFreelancer`, `requirePortalContact`, `requireOperator`) | `src/app/(freelancer)/app/layout.tsx`, `src/app/(portal)/portal/[handle]/layout.tsx`, `src/app/(ops)/ops/layout.tsx`; shells in `src/shared/components/` | Next.js layouts, Tailwind tokens, `src/shared/lib/authz` |
| Navigation | All shells | Config-driven nav arrays per shell; active state from `usePathname()` prefix match with `aria-current="page"`; desktop sidebar, mobile bottom-sheet (Radix Dialog) under 768 px; portal uses a header with back-links rather than a sidebar | `src/shared/components/nav/` | shadcn/ui Sheet/NavigationMenu, Next.js `Link` |
| Error handling | Every route and action | Route-level `error.tsx` boundaries render a recoverable message with Retry that preserves typed input (forms keep state via `useActionState`); `not-found.tsx` for out-of-scope records returns the same "not available" explanation as a missing record so isolation never leaks existence (Access Matrix note); server actions return typed failures instead of throwing for expected cases; unexpected errors reported to Sentry with PII scrubbing (tooling decision ADR-031) | `src/app/**/error.tsx`, `src/shared/lib/telemetry/sentry.ts`, `src/shared/components/ErrorState.tsx` | Sentry SDK, `defineAction` result types |
| Loading / empty states | Every data-backed screen | `loading.tsx` per route segment renders skeletons matching the final layout (ASMP-27 "real progress while loading"); upload and export flows show determinate progress; `EmptyState` component with one primary next action; `OfflineBanner` driven by `useOnline()` states plainly which actions need a connection | `src/app/**/loading.tsx`, `src/shared/components/Skeletons.tsx`, `EmptyState.tsx`, `OfflineBanner.tsx` | React Suspense streaming, shadcn/ui Skeleton |
| Form handling | All mutation forms (≤7 inputs per profile Complex forms = No) | Server actions + React 19 `useActionState`; validation rules are plain TypeScript functions in each feature's `rules/` run on the client for instant feedback and again on the server as authority; hidden `version` field for reject-with-refresh; submit button disabled while pending to prevent double submission; on `conflict` the form shows the current values and keeps the user's input for re-apply | `src/features/*/components/*Form.tsx`, `src/features/*/rules/`, `src/shared/lib/authz/define-action.ts` | Next.js server actions, shadcn/ui Form primitives (Radix Label), ADR-014 version tokens |
| Toast / notification | Feedback after non-navigating actions and background outcomes | Radix Toast via shadcn/ui, bottom-center on mobile and bottom-right on desktop, `aria-live="polite"` (assertive for failures), auto-dismiss after 5 s except errors which persist until dismissed; never used as the only record of a failure that needs action (those render inline) | `src/shared/components/ui/toast.tsx`, `src/shared/hooks/useToast.ts` | Radix Toast |
| Authorization & support-session guard | Every server action, route handler, query and job touching account data | `defineAction()` / `defineQuery()` wrappers resolve the realm session, enforce role (Access Matrix), inject the account and client scope into data-access calls, and reject all mutations while an operator support session is active; operator reads use a read-only Postgres role connection as a second layer (feasibility FEAT-31 mitigation); a test enumerates every exported action and asserts rejection under a support session | `src/shared/lib/authz/` | Better Auth sessions, portal sessions, Drizzle read-only client |
| Versioned & offline mutations | Contended records and the Offline-signal specs | `useVersionedMutation` hook wraps TanStack Query mutations with the version token and conflict handling; only allow-listed mutation keys (`comment.create`, `schedule.save`) persist while offline, replayed in order and re-authorized on the server; persisted cache is cleared on sign-out | `src/shared/hooks/useVersionedMutation.ts`, `src/shared/lib/query-client.ts` | TanStack Query v5, IndexedDB persister |
| Money & date formatting | Every amount and timestamp shown or emailed | `formatMoney(amountMinor, currency)` and `formatDateTime(instant, tz)` wrappers over Intl; freelancer time zone for schedule semantics, viewer time zone for display (FEAT-15.SPEC-006); per-currency grouping, never summed across currencies (XBR-18) | `src/shared/lib/money.ts`, `src/shared/lib/datetime.ts` | Native Intl APIs |

## 11. Authentication & Access Architecture

### Authentication & Identity

| Field | Value |
|-------|-------|
| Context | Authentication = Yes, role-based (profile Section 3): FEAT-05.SPEC-001–007 magic-link portal access for client contacts, FEAT-20.SPEC-001 sign-up, FEAT-21.SPEC-003/005/006 login & security, email change with 24 h re-verification, sign-out other sessions; 4 roles in the Access Matrix (Freelancer, Client Primary Contact, Client Reviewer Contact, Support Operator); "a person who is a contact for several freelancers holds a separate Client Contact per freelancer" (profile Section 5); strict isolation and read-only operator (ASMP-23); contact erasure "ends access immediately" (XBR-27); feasibility FEAT-05: managed providers need the contact-per-freelancer identity mapped onto their user model and hold GDPR-class emails in a vendor store, while Better Auth or a thin token table keeps the mapping in the app's schema; magic-link prefetch risk to the ≥95% first-try target |
| Recommended | Better Auth (open-source library, sessions and accounts in Neon via Drizzle) for the freelancer and operator realm — email magic link plus email/password sign-in, email verification, multi-session listing and revocation, admin plugin for the operator role; plus an application-owned client-portal realm: hashed single-use magic-link tokens scoped to one Client Contact record (24 h expiry per `magic-link-expiry-window`, earlier unused tokens invalidated on re-request), consumed only by an explicit confirm POST on the verification landing, issuing a database-backed portal session |
| Rationale | Better Auth keeps identity data in the application database (landscape: "low lock-in (data in own DB)"; infrastructure cost only), which removes a GDPR sub-processor and lets contact erasure, session revocation and account deletion happen in the same transaction as the record change (FEAT-18.SPEC-009, FEAT-24.SPEC-004). Client contacts are per-freelancer records, not global users; modelling them as a separate, deliberately small realm keyed by Client Contact id fits that shape exactly, where a managed provider would force one-user-many-identities mapping (feasibility FEAT-05). The confirm-click interstitial mitigates scanner prefetch consumption (feasibility risk table). Clerk's free tier would be affordable, but it adds vendor-held contact data and does not model per-freelancer identities natively. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Clerk | Free to 50K monthly retained users; then $25/month plus per-MRU overage | Low — hosted UI, SDKs, multi-session built in | High | Medium–High — users in vendor store, export possible | Mainstream | The team wants hosted sign-in UI, MFA and bot protection for freelancers out of the box and accepts a vendor holding freelancer emails; the portal realm stays application-owned either way |
| Supabase Auth | Free 50,000 MAU; Pro $25/month incl. 100,000 MAU | Low with Supabase Postgres | High | Medium — GoTrue open source exit | Supabase + RLS | The Database decision switches to Supabase and row-level security becomes the primary client-isolation enforcement point |
| WorkOS AuthKit | Free to 1M MAU; SSO from $125/month per connection | Low — SDKs | Very high | Medium–High — hosted user store | Mainstream | Agency or enterprise freelancers require SAML SSO for their own staff (not in any spec today) |

### Session Model

Two session realms, both **database-backed opaque sessions** in Neon, delivered as `HttpOnly; Secure; SameSite=Lax` cookies with only a random session identifier in the cookie (hash stored server-side), so revocation is immediate by row deletion:

- **Freelancer / operator (Better Auth):** cookie `cr_session` on the app host; Assumption — 30-day rolling lifetime refreshed on activity at most once per day, 12-hour absolute lifetime for operator accounts; sessions listed on Login & Security and revocable individually or all-but-current (FEAT-21.SPEC-006); email change keeps the old address active until the new one is verified within 24 hours (FEAT-21.SPEC-005).
- **Client portal (application-owned):** cookie `cr_portal` scoped to the serving host (shared address or the freelancer's verified custom domain — sessions are per host, per feasibility Open Question 12); one session binds exactly one Client Contact id and therefore one client company of one freelancer; Assumption — 30-day rolling lifetime; role (Primary/Reviewer) is read from the Client Contact row on every request rather than cached in the session, so role changes and removals take effect on the next request (FEAT-18.SPEC-008, XBR-27).
- **Operator support session (FEAT-31):** a `support_access_session` row opened by the operator against one freelancer account; while open, the operator's requests carry the target account scope with read-only enforcement; `last_activity_at` is updated per request and a Trigger.dev task closes sessions after 15 minutes of inactivity (`support-session-inactivity-timeout-minutes`).

Magic-link and portal tokens are stored only as SHA-256 hashes; sign-in email requests are rate-limited per email and IP in Postgres (Assumption — 5 per hour per address).

### User Model Fields

Freelancer/operator identity (`user` table, shared with Better Auth): `id`, `email` (unique, sign-in email), `email_verified_at`, `pending_email`, `pending_email_expires_at`, `display_name`, `role` (`freelancer` | `operator`), `time_zone` (IANA), `onboarding_state`, `created_at`, `deletion_state` (`active` | `hold` | `committed`), `deleted_at`; plus Better Auth `account` (credential/provider rows) and `session` (`id`, `user_id`, `token_hash`, `expires_at`, `created_at`, `last_seen_at`, `user_agent`) tables. The freelancer's business profile lives on the Freelancer Account entity (one-to-one with `user` where `role = freelancer`).

Client-portal identity (on the Client Contact entity, per freelancer): `id`, `freelancer_account_id`, `client_id`, `email` (unique per client), `name`, `portal_role` (`primary` | `reviewer`), `status` (`invited` | `active` | `removed`), `invited_by_contact_id`, `last_sign_in_at`, `erased_at`, `help_dismissals`; `portal_token` (`id`, `client_contact_id`, `token_hash`, `expires_at`, `consumed_at`, `created_at`); `portal_session` (`id`, `client_contact_id`, `host`, `token_hash`, `created_at`, `expires_at`, `last_seen_at`). Support: `support_access_session` (`id`, `operator_user_id`, `freelancer_account_id`, `opened_at`, `last_activity_at`, `closed_at`, `close_reason`). The Schema Designer owns final column types and constraints.

### Protected Routes

| Route / Route Group | Access Requirement | Unauthenticated Experience |
|--------------------|--------------------|----------------------------|
| `/`, `/signup`, `/sign-in`, `/r/[referralToken]` | Public | -- |
| `/app/**` (freelancer workspace; FEAT-01, 02, 04, 06, 08.SPEC-002, 09–13, 15–25, 27–29, 30.SPEC-002, 31.SPEC-001, 32) | Authenticated, role-gated (freelancer) — or operator with an open support session for that account (read-only) | Redirect to `/sign-in?next=…` |
| `/portal/[handle]/sign-in`, `/portal/[handle]/verify` (FEAT-05.SPEC-001, FEAT-05.SPEC-002) | Public within the portal | -- (expired/used link shows recovery with a request-new-link action, FEAT-05.SPEC-002) |
| `/portal/[handle]/**` other routes (FEAT-03, 04.SPEC-002, 05.SPEC-003, 07, 08.SPEC-001, 09.SPEC-002, 10.SPEC-001, 17.SPEC-002, 18.SPEC-004, 26, 30.SPEC-003) | Authenticated portal contact of a client of that freelancer; records filtered to the contact's own client (FEAT-05.SPEC-007) | Redirect to `/portal/[handle]/sign-in` with the target preserved; out-of-scope records show the plain "not available — request a fresh link" explanation, never another company's data |
| `/portal/[handle]/projects/[projectId]/proposal/**` accept, request changes, sign (FEAT-03.SPEC-001, FEAT-03.SPEC-002, FEAT-26.SPEC-001) | Role-gated (Primary) — Reviewers cannot see proposals (FEAT-03.SPEC-005) | Reviewer sees "not available" |
| `/portal/[handle]/milestones/[milestoneId]` approve action (FEAT-08.SPEC-001) | Role-gated (Primary) for approve; Reviewer view and comment only (FEAT-08.SPEC-006) | Approve control hidden for Reviewers |
| `/portal/[handle]/invoices/**` (FEAT-09.SPEC-002, FEAT-10.SPEC-001) | Role-gated (Primary) (FEAT-09.SPEC-006) | Reviewer sees "not available" |
| `/portal/[handle]/team/invite` (FEAT-18.SPEC-004) | Role-gated (Primary) | Reviewer sees "not available" |
| `/ops/**` (FEAT-31.SPEC-002) | Role-gated (operator) | 404 for non-operators; redirect to `/sign-in` when signed out |
| `/api/webhooks/*` | Provider signature verification (no session) | 401 on invalid signature |
| `/api/uploads/*`, `/api/files/*` | Authenticated freelancer (uploads); authorized viewer of the version (files); operator always refused for file downloads (XBR-29) | 401/403 JSON |

### Role & Permission Mapping

| Role (Access Matrix) | Application Representation | Capability Access Summary | Enforcement Point |
|----------------------|---------------------------|---------------------------|-------------------|
| Nadia (Freelancer) | `user.role = 'freelancer'` in the Better Auth realm; owns one Freelancer Account; every query scoped by `freelancer_account_id` | Full on client/project management, proposals, milestones/deliverables, invoicing/payments, dashboard/export, contacts, branding/onboarding/settings, subscription/account data, portal access control and notifications/help; Full (read and share) on the activity trail with entries never editable; View on support sessions on her account and on the portal referral mark | Freelancer layout guard + `defineAction`/`defineQuery` account scoping in the API layer; immutability of evidence enforced in rules and database constraints (database-schema.md) |
| Owen (Client Primary Contact) | Portal session bound to a Client Contact with `portal_role = 'primary'`; scope = that contact's `client_id` under one `freelancer_account_id` | Own-only: view/accept/request changes/sign proposals; view/comment/approve milestones and deliverables; view/pay invoices and download copies; invite Reviewer contacts at own company; own portal access; own emails and help tips; View on the referral mark; None on dashboard/export, activity trail, branding/settings, subscription/account data and support access | Portal layout guard + `requirePortalContact({ role: 'primary' })` in actions; client-scoped query functions (FEAT-05.SPEC-007); role re-read per request |
| Priya (Client Reviewer Contact) | Portal session bound to a Client Contact with `portal_role = 'reviewer'` | Own-only: view and comment on milestones and deliverables (no approve), own portal access, own emails and help tips; View on the referral mark; None on proposals, invoicing/payments, contacts and everything else | Same portal guards; proposal and invoice queries exclude Reviewers server-side (FEAT-02 feasibility risk: enforced server-side, not only in UI) |
| Dana (Support Operator) | `user.role = 'operator'` in the Better Auth realm; account access only through an open `support_access_session` | View on client/project management, proposals, milestones/deliverables (no file downloads), invoicing/payments, dashboard (no export generation), activity trail, contacts, branding/onboarding/settings, subscription (plan status only; no export or deletion), delivery warnings and the referral mark; Full on support access (opens read-only, always-logged sessions); None on client portal access | `requireOperator()` on `/ops`; support-session scope + global mutation rejection in `defineAction`; read-only Postgres role connection for operator reads; `/api/files` refuses operators; every session open/close written to the activity trail and announced by email (FEAT-31.SPEC-007) |

## 12. Development Conventions

### Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Files (components) | PascalCase `.tsx`, one exported component per file | `MilestoneApprovalView.tsx` |
| Files (utilities) | kebab-case `.ts`; actions named verb-noun | `record-acceptance.ts`, `money.ts` |
| Components | PascalCase, suffix by role (`View`, `Form`, `List`, `Dialog`) | `export function InvoiceList()` |
| Functions | camelCase verb-first; server actions end without suffix; queries start with `get`/`list` | `acceptProposal()`, `listInvoicesForAccount()` |
| CSS classes | Tailwind utilities in markup; token names kebab-case CSS variables; no custom class names except `cr-print-*` print helpers | `className="bg-[--brand] text-[--brand-foreground]"` |
| Database tables | snake_case, singular, entity-named; columns snake_case; money columns suffixed `_minor` | `invoice`, `client_contact`, `amount_minor` |
| Routes | kebab-case path segments, plural resource nouns, `[camelCaseId]` params | `/app/invoices/[invoiceId]/record-payment` |

### Component Structure

Order within a component file: (1) `'use client'` directive only when interactivity requires it — Server Components are the default; (2) imports per the ordering below; (3) exported `Props` type; (4) the component function; (5) small private sub-components; (6) no default exports except Next.js route files (`page.tsx`, `layout.tsx`). Data access never happens inside client components — they receive data from a Server Component parent or TanStack Query hooks that call server actions.

```tsx
import { formatMoney } from '@shared/lib/money';
import type { InvoiceSummary } from '../queries/list-invoices';

export type InvoiceRowProps = { invoice: InvoiceSummary };

export function InvoiceRow({ invoice }: InvoiceRowProps) {
  return <li>{invoice.number} · {formatMoney(invoice.amountMinor, invoice.currency)}</li>;
}
```

### Import Ordering

Groups separated by a blank line, enforced by ESLint `import/order`: (1) React/Next.js, (2) third-party packages, (3) `@shared/*` and `@db/*` aliases, (4) `@features/*` (other features only through their public `index.ts`), (5) relative imports, (6) type-only imports last.

```ts
import { notFound } from 'next/navigation';

import { and, eq } from 'drizzle-orm';

import { db } from '@shared/lib/db';
import { defineAction } from '@shared/lib/authz';

import { generateInvoice } from '@features/feat-09-invoice-generation-sending';

import { assertAcceptable } from '../rules/acceptance-rules';

import type { ActionResult } from '@shared/lib/authz';
```

### TypeScript Usage

`strict: true` plus `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes` — financial correctness (ASMP-25) and 60 Logic/Rule specs justify the strictest settings (ADR-028). Prefer `type` aliases for data shapes and discriminated unions (`ActionResult`); use `interface` only for extensible object contracts such as integration client interfaces. `any` is banned (`@typescript-eslint/no-explicit-any: error`); use `unknown` with narrowing at boundaries (webhook payloads are parsed and narrowed before use). Money is a branded type `MinorUnits` so raw numbers cannot be passed as amounts. Database row types are inferred from the Drizzle schema (`typeof invoice.$inferSelect`), never hand-written.

### Path Aliases

Defined in `tsconfig.json` `compilerOptions.paths`:

```json
{
  "@/*": ["./src/*"],
  "@features/*": ["./src/features/*"],
  "@shared/*": ["./src/shared/*"],
  "@db/*": ["./src/db/*"],
  "@jobs/*": ["./src/jobs/*"]
}
```

## 13. Deployment & Environments

### Hosting & Environments

| Field | Value |
|-------|-------|
| Context | ADR-001 Next.js 16 with ADR-002 framework server layer; ADR-012 moves long-running work off the web host; ASMP-26 no uptime target; worldwide clients mostly on mobile (BRIEF.md Geography/Devices) with the ~2 s target (ASMP-21); budget ~$100/month; custom domains via the host (ADR-018); preview environments wanted for a 33-feature codebase |
| Recommended | Vercel Pro ($20/seat/month incl. $20 usage credit and 1 TB transfer) — production and per-branch preview deployments, Node.js runtime functions in one region co-located with the Neon project (Assumption — an EU region by default, Section 2), global CDN for static assets |
| Rationale | The landscape rates Vercel "native Next.js hosting, preview environments per branch, CDN" with Low integration effort; Hobby is non-commercial only, so Pro is the floor. With scheduled and long-running work on Trigger.dev, the host only serves request/response traffic, where serverless scale-to-zero fits human-paced load. Preview deployments paired with Neon branches give every pull request an isolated, realistic environment. Platform coupling is contained: the app is a standard Next.js build that also runs on container hosts (alternatives below). |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Render | Fixed instance prices plus workspace fee; sample stack ~$30/month | Low — web service, cron, workers, managed Postgres in one place | High | Low — standard containers | Mainstream | The team prefers always-on containers with an in-house worker (enabling pg-boss instead of Trigger.dev) and predictable fixed pricing |
| Fly.io | Per-second machine billing; sample Node+Postgres stack ~$12/month | Medium — CLI-first, Dockerfile | High; regions near users | Low — standard containers | Docker/Fly CLI | Multi-region serving close to worldwide clients becomes necessary to hit ASMP-21 after measurement, or cost pressure demands the cheapest compute |
| Railway | Hobby $5/month, Pro $20/month plus usage; sample stack ~$37/month | Low — Git deploys, one-click Postgres and workers | Medium–High | Low | Mainstream | A small team wants container hosting with minimal ops and co-located workers, without Vercel-specific features |

### CI/CD & Delivery

| Field | Value |
|-------|-------|
| Context | 33 features, 220 specs with financial-correctness rules (ASMP-25) requiring automated checks before promotion (landscape CI/CD area); migration-based schema changes (Section 6); Trigger.dev tasks deployed alongside the web app; Vercel hosting (ADR-029) |
| Recommended | GitHub Actions for checks (typecheck, lint, unit tests for Logic/Rule functions, integration tests against a Neon branch incl. the support-session mutation sweep, Playwright e2e against the preview URL) and for migrations and Trigger.dev deploys; Vercel Git integration for build-and-deploy with a preview per pull request; production promotion from `main` gated on a protected GitHub environment |
| Rationale | GitHub Actions is "native to GitHub repos; environments with approval gates" with 2,000 free minutes/month (Free) or 3,000 (Team) — enough for this codebase at launch; platform-native Vercel deploys give previews without writing deploy scripts, while Actions supplies the test runner that platform pipelines lack (landscape note "tests still need a separate runner"). Migrations run in Actions before promotion so schema and code ship together. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| GitLab CI/CD | Free 400 min/month; Premium from $29/user/month | Medium — requires GitLab code hosting | High | Medium — GitLab syntax | GitLab | The team's code already lives on GitLab |
| CircleCI | Free 6,000 credits/month; Performance $15 per 25,000 credits | Low–Medium | High | Medium — CircleCI config | CircleCI | Test suites grow long enough that parallel test splitting beyond Actions' runners becomes the bottleneck |
| Platform-native deploy pipelines only (Vercel Git deploys) | Included with hosting | Low | Medium | Medium — tied to host | Minimal | Never on its own for this product — only acceptable during an initial prototype before financial rules are implemented, since it runs no tests |

**Environment topology:**

- **Local development** — developer machine; Docker Postgres or a personal Neon branch; Stripe, Resend and Documenso test keys; dev R2 bucket or MinIO; `trigger dev` for jobs. Secrets in `.env.local` (never committed).
- **Preview** — one Vercel preview deployment per pull request, paired with an ephemeral Neon branch created from staging data-free schema and seeded by `src/db/seed.ts`; test-mode keys for Stripe, Resend (test key), Documenso; dev R2 bucket; Trigger.dev `preview` environment. Used for review and e2e tests.
- **Staging** — deploys from `main` automatically; its own Neon branch with migrated schema and synthetic data, Stripe test mode with test connected accounts, a staging sending subdomain, a staging R2 bucket; Trigger.dev `staging` environment. Used to verify migrations and webhook flows end to end.
- **Production** — promoted from a green `main` commit via the protected `production` GitHub environment (manual approval); production Neon branch with point-in-time restore; live Stripe keys; production R2 buckets and sending domains; Trigger.dev `prod` environment. Secrets live in Vercel and Trigger.dev environment variables per environment and in GitHub environment secrets for CI; no secret is shared between environments.

**Infrastructure as code:** Not warranted at launch — the stack is five managed services configured through their dashboards and a handful of environment variables, operated by a small team (Section 2 load is modest). The trigger that changes the answer: a second region, a second production project (e.g. EU/US split for data residency), or more than ~10 environment-specific secrets per service — at that point adopt Terraform (Vercel, Cloudflare and Neon providers) for reproducible environments.

### Observability & Operations

| Field | Value |
|-------|-------|
| Context | ASMP-26: no uptime target, but "emails that fail to deliver are surfaced to the freelancer within minutes rather than lost"; Background processing = Yes with minute-level sweeps whose silent failure breaks FEAT-11 "exactly 3 days" reminders (feasibility FEAT-11 risk); Compliance/privacy = Yes (GDPR-class data must not leak into logs); FEAT-13 audit trail is evidentiary and separate from operational logs (feasibility FEAT-13); budget ~$100/month |
| Recommended | Sentry Team ($26/month incl. 50K errors) for error tracking and tracing across Next.js and Trigger.dev, Sentry Cron Monitors on every scheduled task (outbox sweep, reminder sweep, retention purge, support auto-close) and an uptime check on the portal and webhook endpoints; Grafana Cloud free tier (Loki) as the Vercel log-drain and Trigger.dev log destination with alert rules on webhook-processing failures and outbox backlog age; PII scrubbing (emails, names, document content) at the SDK and log-formatter level |
| Rationale | Sentry covers errors, performance and missed-schedule detection in one SDK per framework (landscape: "Low — SDK per framework") at a fixed cost inside the budget; cron monitors directly address the missed-sweep risk behind FEAT-11 and FEAT-24. Grafana Cloud's free tier (then ~$0.50/GB) gives searchable logs and alerting without Datadog's per-host pricing. Operational telemetry never substitutes for the FEAT-13 trail, which lives in Postgres. |

**Alternatives:**

| Alternative | Cost Profile | Operational Complexity | Scale Ceiling | Lock-in | Team Skill Demand | Choose instead when |
|---|---|---|---|---|---|---|
| Axiom (logs and traces, with Sentry for errors) | Usage-based; described as a fraction of Datadog's cost | Low — HTTP ingest, Vercel integration | High | Low–Medium | Minimal | Log volume or query needs outgrow the Grafana free tier and a serverless-native log store with simpler setup is preferred |
| Grafana Cloud as the single stack (no Sentry) | Free tier; logs ~$0.50/GB ingested | Medium — OpenTelemetry setup, build own error views | High | Low — open-source components | OpenTelemetry | Cost pressure forces dropping Sentry's $26/month and the team is comfortable building error triage on logs and traces |
| Datadog | Infrastructure $15/host/month, APM $31/host/month, logs $0.10/GB | Medium | Very high | High — proprietary agent | Datadog | The product moves to container hosting with many hosts and a dedicated ops function wanting one proprietary suite |

### Indicative Cost Model

Order-of-magnitude only; verify current pricing before committing. "Launch" is the first months at hundreds to low thousands of freelancers; "Growth" is the year-one "few thousand freelancers" tier (Section 2).

| Component | Launch (order of magnitude) | Growth tier |
|-----------|-----------------------------|-------------|
| Vercel Pro (hosting, CDN, previews) | ~$20/month (1 seat, within $20 usage credit) | ~$40–100/month (2–3 seats plus usage beyond credit) |
| Neon Postgres (Launch → Scale plan) | ~$10–25/month ($0.106/CU-hour with scale-to-zero; few GB at $0.35/GB-month) | ~$50–160/month (Scale plan, ~1 CU sustained at $0.222/CU-hour) |
| Cloudflare R2 (deliverables, archives, logos) | ~$0–15/month (10 GB free; up to ~1 TB at $0.015/GB-month; zero egress) | ~$150–300/month at 10–20 TB stored (feasibility estimate; storage-allowance decision pending, Open Question 2) |
| Resend (transactional email) | Free (3,000/month) to ~$20/month (Pro, 50,000 emails) | ~$20–90/month, volume-scaled beyond 50,000 emails |
| Trigger.dev (jobs and schedules) | Free (50K runs/month) | ~$10–50/month usage-scaled |
| Sentry Team (errors, crons, uptime) | ~$26/month | ~$26–80/month with error overage |
| Grafana Cloud (logs) | Free tier | ~$5–25/month (~$0.50/GB ingested) |
| PostHog Cloud (analytics) | Free (1M events/month) | ~$0–50/month ($0.00005/event beyond free) |
| GitHub Actions (CI) | Free (2,000 min/month) | ~$4/user/month Team plan plus ~$0–20/month overage at $0.006/min |
| Documenso (e-signature, FEAT-26 v1 only) | ~$30/month (Individual) once FEAT-26 ships | ~$250/month (Platform) if white-label embedding is required, or ~$12/month self-hosted on a small container |
| Stripe Billing + card fees on the freelancer's own plan (revenue-proportional, not infrastructure) | ~$10–50/month (0.7% Billing plus 2.9% + 30c per charge on early subscriptions) | ~$500–1,500/month at ~1,000 paying freelancers on a $15/month plan |
| Stripe Connect on client invoice payments | $0 to the platform (direct charges bill the freelancer's own account) | $0 to the platform |

At launch the infrastructure lines (excluding Stripe's revenue-proportional fees) total roughly $80–130/month, close to the "under roughly $100/month" constraint; at the growth tier object storage alone can exceed the budget, which is the tension surfaced in feasibility Open Question 2 and must be resolved by the storage-allowance or budget decision rather than by the architecture.

### Local Development Setup

Prerequisites: Node.js LTS, pnpm, Docker (for local Postgres and optional MinIO), the Stripe CLI, and the Trigger.dev CLI. Setup path: clone → `pnpm install` → copy `.env.example` to `.env.local` and fill test keys (Stripe test mode, Resend test key, Documenso free/test account, R2 dev bucket or MinIO endpoint, Trigger.dev dev key) → `docker compose up -d postgres minio` (or point `DATABASE_URL` at a personal Neon branch) → `pnpm db:migrate && pnpm db:seed` → in separate terminals `pnpm dev`, `pnpm trigger:dev`, and `stripe listen --forward-to localhost:3000/api/webhooks/stripe-connect`. Local substitutes: Docker Postgres for Neon, MinIO for R2 (S3 API), Stripe test mode and CLI webhooks, Resend test key (emails viewable in the Resend dashboard or React Email preview at `pnpm email:dev`), Trigger.dev local dev runner; Sentry and PostHog are disabled when their DSN/key is unset.

## 14. Decision Log

| ID | Category | Decision | Rationale (one-line) | Profile Driver |
|----|----------|----------|---------------------|---------------|
| ADR-001 | Frontend Framework | Next.js 16 (App Router, React 19, TypeScript) | RSC keeps mobile bundles small for 65 screens and three shells; largest SDK ecosystem for 7 integrations | Scale: Large (33 features, 220 specs); 65 Screen specs; ASMP-21 ~2 s mobile interactivity |
| ADR-002 | Backend / API Layer | Next.js framework server layer (server actions + Route Handlers on Node runtime); long-running work to job runner | One deployable and one auth boundary for human-paced load; webhooks and mutations share domain rules | 62 Automation specs, 7 Integration specs; Real-time = No; ASMP-22 a few thousand freelancers |
| ADR-003 | Database | Neon serverless Postgres (Launch plan, Scale plan at growth) with branches per environment | Relational transactions serve every Hard verdict; scale-to-zero fits the budget; branching for previews | 23 entities, 60 relationships, 35 XBR; contention on 12 entities; budget under ~$100/month |
| ADR-004 | ORM / Data Access | Drizzle ORM v1 + drizzle-kit, Neon Pool driver for interactive transactions | SQL-close typed queries express counter locks, conditional updates and ON CONFLICT idempotency | Data Complexity: Medium (23 entities); FEAT-09 and FEAT-10 feasibility Hard (gapless numbering, multi-writer state) |
| ADR-005 | CSS / Styling | Tailwind CSS v4 with CSS-variable theme tokens and injected per-freelancer --brand variables | Runtime brand colours via CSS variables; static CSS keeps mobile payloads small | FEAT-19.SPEC-001–003 branding; ASMP-27 legibility; ASMP-21 mobile target |
| ADR-006 | State Management | RSC for reads + TanStack Query v5 with IndexedDB persistence on offline/interactive islands; no global store | Server-derived state; persisted mutations/snapshots serve the four Offline-signal specs | Real-time = No; Offline = Yes (FEAT-07.SPEC-008, FEAT-29.SPEC-004); Collaboration/concurrency = Yes |
| ADR-007 | Build Tooling | Turbopack (bundled with Next.js 16) + pnpm | Framework-bundled pipeline with no override driver; strict lockfile for CI reproducibility | Frontend selection ADR-001; 33 features in one deployable |
| ADR-008 | File & Object Storage | Cloudflare R2 with browser-direct multipart uploads and presigned GETs | Zero egress for repeated large-file streaming; S3 API keeps exit to B2/S3 open | File upload = Yes; ASMP-22 files tens of MB to >1 GB; ASMP-30; budget ~$100/month |
| ADR-009 | Email & Messaging Delivery | Resend Pro with delivery/bounce webhooks and React Email branded templates | Fits TypeScript stack and budget; webhooks feed the event ledger for minutes-level failure surfacing | 26 Notification specs (email only); ASMP-26; ASMP-29; FEAT-14.SPEC-003 |
| ADR-010 | Payments & Billing | Stripe Connect Standard with direct charges (Payment Element) + Stripe Billing for own plan | Funds and fees land in the freelancer's account with no platform cut; self-serve onboarding for the 5-minute target | Payments/billing = Yes; ASMP-24, ASMP-28, ASMP-31; FEAT-10 verdict Hard; FEAT-32 connect <5 minutes |
| ADR-011 | Search | PostgreSQL full-text search (tsvector + GIN, pg_trgm) with account-scoped queries | Tiny per-account corpus; no second copy of GDPR-class data; isolation enforced by the same query path | Search = Yes (FEAT-28.SPEC-001–004); no full-text/faceted keywords; 3–15 active clients per account |
| ADR-012 | Background Jobs & Scheduling | Trigger.dev v3 (Cloud) fed by a Postgres transactional outbox with one-minute sweep | Long-running tasks without timeouts plus schedules; outbox gives atomic enqueue; self-host exit | Background processing = Yes (15 specs); FEAT-24 verdict Hard (multi-GB archive, staged deletion); 62 Automation specs |
| ADR-013 | Caching & Performance | Vercel CDN + Next.js cache for public pages; Postgres-held per-currency totals table; no Redis at launch | Meets 2 s / 1–2 s targets with already-paid layers; private data never in shared caches | Scale hints = Yes (ASMP-21 dashboard 1–2 s); Offline = Yes; budget ~$100/month |
| ADR-014 | Real-time & Collaboration | Database optimistic concurrency: version tokens, status-guarded conditional updates, locked per-freelancer invoice counter, unique idempotency keys; no realtime transport | Reject-with-refresh and exactly-once inside the data store; native sequences would skip numbers | Collaboration/concurrency = Yes (12 contended entities); Real-time = No; FEAT-09.SPEC-007 "never reused, never skipped" |
| ADR-015 | Analytics & Product Telemetry | PostHog Cloud (free tier) with server-side milestone events keyed by opaque ids; no replay on portal | Funnels for activation metrics within free tier; open-source self-host option; no client-contact PII | Scale hints = Yes (ASMP-22); ASMP-24 GDPR-class data; FEAT-20/FEAT-32/FEAT-05 success metrics |
| ADR-016 | Internationalization | Native Intl APIs; integer minor units + ISO 4217; IANA time zones; no translation framework at launch | Worldwide currency/tax/time zone without translation need; correct 0/3-decimal handling | Internationalization = Yes (FEAT-15.SPEC-001–008); English only at launch; XBR-18 non-aggregation |
| ADR-017 | Electronic Signature Attestation | Documenso API (Individual tier for spike/v1, Platform tier only if white-label needed, self-host exit); design to ESIGN/UETA and eIDAS SES pending spike | Only open-source, self-hostable option; fits budget at one signature per opted-in proposal; DocuSign embedded tier exceeds budget | Product-mandated (External Touchpoints, FEAT-26.SPEC-005); FEAT-26 verdict Research-spike recommended; budget ~$100/month |
| ADR-018 | Custom Domain Verification & TLS Serving | Vercel Domains API with automatic TLS; middleware host-to-portal mapping with shared-address fallback | Zero added cost and no extra proxy when hosted on Vercel; Cloudflare for SaaS if domain limits are hit | Product-mandated (ASMP-32, FEAT-27.SPEC-002); at most one domain per freelancer, a few thousand accounts |
| ADR-019 | Project Structure | Single Next.js package (no monorepo) with src/features/feat-NN-slug logic folders and src/jobs Trigger.dev tasks sharing one schema | One schema and rule set for web and jobs; feature isolation per Stage 3 folder | 33 features; 60 Logic/Rule specs reused by 62 Automation specs across web and jobs |
| ADR-020 | Migration Strategy | Migration-based: drizzle-kit generate (committed SQL) + migrate in CI per environment | Reviewed, replayable schema changes protect immutable financial records across branches | 23 entities, 60 relationships; ASMP-25 immutable records; Neon branch-per-environment |
| ADR-021 | API & Routing | Resource-oriented URLs in three route groups: /app (freelancer), /portal/[handle] (client, custom-domain rewrite), /ops (operator), plus public pages | Separate shells and guards per realm; stable deep links from emails; custom domains map onto portal routes | 65 Screen specs; 4 Access Matrix roles; 54 navigation connections; ASMP-32 custom domains |
| ADR-022 | API & Routing | RSC reads + server actions with version tokens for mutations; Route Handlers only for webhooks, upload sessions, file grants and auth; TanStack Query only on offline/interactive islands | Complete HTML on first paint; server-side exactly-once and stale-state checks; bytes bypass functions | ASMP-21 ~2 s mobile; Collaboration/concurrency = Yes; ASMP-27 no pretend-offline success |
| ADR-023 | Webhook Ingestion | Signature-verified webhooks persisted to an inbound_event ledger (unique provider event id), acknowledged 200, processed by Trigger.dev in event-time order per subject with processor state authoritative | One idempotent, replayable path for all providers' duplicated and out-of-order events | 7 Integration specs all requiring "same event twice changes nothing" and event-time ordering; FEAT-10 verdict Hard |
| ADR-024 | CSS / Styling | Component library: shadcn/ui (copy-in, Radix primitives + Tailwind) in src/shared/components/ui | Accessible primitives for tables, dialogs, tooltips, toasts; owned and unopinionated for a design-agnostic package | 65 Screen specs with tables/dialogs/tooltips; Complex forms = No; ASMP-27 screen reader/keyboard; design_system_source: none |
| ADR-025 | State Management | Offline pattern: only allow-listed mutations (comment.create, schedule.save) persist and replay with server re-authorization; accept/approve/pay/send never queued; persisted cache cleared on sign-out | Serves offline specs while honoring the no-pretend-success rule and shared-device privacy | Offline = Yes (FEAT-07.SPEC-008, FEAT-04.SPEC-001 AC-15, FEAT-29.SPEC-004); ASMP-27 |
| ADR-026 | Authentication & Identity | Better Auth (freelancer/operator realm, sessions in Neon) + application-owned client-portal magic-link realm keyed by Client Contact with confirm-click consumption | Per-freelancer contact identities, in-transaction erasure/revocation, no vendor holding GDPR-class emails | Authentication = Yes, role-based (FEAT-05.SPEC-001–007, FEAT-21.SPEC-003/005/006); 4 Access Matrix roles; ASMP-23 |
| ADR-027 | Authentication & Identity | Session model: database-backed opaque HttpOnly cookie sessions per realm (30-day rolling, host-scoped portal sessions), role re-read per request, 15-minute inactivity close for support sessions | Immediate revocation by row deletion; role changes and removals effective on next request | FEAT-21.SPEC-006 sign-out others; XBR-27 immediate access end; FEAT-31.SPEC-004 15-minute auto-close |
| ADR-028 | Development Conventions | TypeScript strict + noUncheckedIndexedAccess + exactOptionalPropertyTypes; no any; branded MinorUnits money type; schema-inferred row types | Compile-time guards on money and nullable financial fields | ASMP-25 correctness; 60 Logic/Rule specs; 82 specs matching currency/tax keywords |
| ADR-029 | Hosting & Environments | Vercel Pro, single region co-located with Neon (EU default), per-branch previews, global CDN | Native Next.js hosting with previews at ~$20/month; request/response only since jobs run on Trigger.dev | ASMP-26 no uptime target; ASMP-21 mobile target; budget ~$100/month |
| ADR-030 | CI/CD & Delivery | GitHub Actions for checks, migrations and Trigger.dev deploys + Vercel Git deploys; production gated by protected environment | Automated correctness checks before promotion; previews without deploy scripts | 220 specs with ASMP-25 financial-correctness rules; migration-based schema (ADR-020) |
| ADR-031 | Observability & Operations | Sentry Team (errors, tracing, cron monitors, uptime) + Grafana Cloud free-tier logs with PII scrubbing | Minutes-level detection of failed sweeps and webhooks within budget; audit trail kept separate | ASMP-26 failures surfaced within minutes; Background processing = Yes; Compliance/privacy = Yes |
