---
document_type: technical-profile
produced_by: profile-analyst
status: final
stage: 4
created: 2026-09-29
project_name: Clientroom
---

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
