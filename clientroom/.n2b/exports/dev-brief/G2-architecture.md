# Part G2 — Architecture: Recommendation & Alternatives

This part carries the four architecture documents in reading order: evidence, option space, feasibility, then decisions. The recommendation in each decision area is the default build; alternatives are documented at equal depth with `Choose instead when` conditions, for this team to weigh and adopt deliberately where its own constraints differ.

## The Evidence — Technical Profile


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


## The Option Space — Technology Landscape


# Technology Landscape

## 1. Research Scope

Derived mechanically from `technical-profile.md`: 11 always-active areas, 9 signal-activated areas (AI & Intelligent Behavior and Geo & Maps stay inactive: AI/ML behavior = No, Geo/maps = No), and 2 extension areas triggered textually by profile Section 7.

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
| File & Object Storage | File upload (FEAT-06.SPEC-001, FEAT-06.SPEC-003, FEAT-16.SPEC-002, FEAT-16.SPEC-007, FEAT-17.SPEC-001; ASMP-22, ASMP-30); Import/export (FEAT-22.SPEC-002, FEAT-24.SPEC-003) |
| Email & Messaging Delivery | Notifications (email/push/SMS) (26 Notification specs, e.g. FEAT-02.SPEC-011, FEAT-05.SPEC-008; FEAT-14.SPEC-001, FEAT-14.SPEC-002; ASMP-29) |
| Payments & Billing | Payments/billing (FEAT-10.SPEC-003, FEAT-23.SPEC-003, FEAT-32.SPEC-002, FEAT-09.SPEC-001 to FEAT-09.SPEC-010; ASMP-28, ASMP-31) |
| Search | Search (FEAT-28.SPEC-001, FEAT-28.SPEC-002, FEAT-28.SPEC-003, FEAT-28.SPEC-004) |
| Background Jobs & Scheduling | Background processing (FEAT-11.SPEC-001, FEAT-12.SPEC-004, FEAT-14.SPEC-003, FEAT-24.SPEC-003, FEAT-24.SPEC-005); Import/export (FEAT-22.SPEC-002, FEAT-24.SPEC-003); Notifications (email/push/SMS) (FEAT-14.SPEC-002) |
| Caching & Performance | Scale hints (ASMP-21, ASMP-22, ASMP-26; BRIEF.md ## Constraints budget); Offline (FEAT-07.SPEC-008, FEAT-29.SPEC-004; ASMP-27) |
| Real-time & Collaboration | Collaboration/concurrency (FEAT-03.SPEC-003, FEAT-08.SPEC-003, FEAT-10.SPEC-004, FEAT-10.SPEC-005, FEAT-11.SPEC-001, FEAT-18.SPEC-006; Real-time = No) |
| Analytics & Product Telemetry | Scale hints (ASMP-22; BRIEF.md ## Scale & Non-Functional Expectations) |
| Internationalization | Internationalization (FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-006, FEAT-15.SPEC-008, FEAT-22.SPEC-003) |
| Electronic Signature Attestation | Product-mandated — feature-dependency-map.md External Touchpoints: "Electronic-signature attestation for legally binding proposal signing — v1 phase ... FEAT-26.SPEC-005 (Electronic-Signature Attestation Capability — submits the signature data captured by FEAT-26.SPEC-002 for attestation ... the jurisdictional standard it must meet is left to Stage 4)" |
| Custom Domain Verification & TLS Serving | Product-mandated — assumptions-constraints.md ASMP-32: "Domain-verification capability (Later phase) — Custom Domain per Freelancer (FEAT-27) requires the ability to verify that a freelancer controls a domain and serve her portal securely at it." (FEAT-27.SPEC-002) |

## 2. Decision Area Landscapes

### Frontend Framework

Serves 18 user-facing features and 65 Screen specs (profile Section 1), a mobile-first client portal with ~2 s interactivity (ASMP-21), and forms of at most 7 fields (Complex forms = No).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Next.js (App Router, v16) | React meta-framework | SSR/RSC for fast mobile first paint; server actions for form flows; largest hiring pool and ecosystem | Open source (MIT); free to self-host | Low — largest ecosystem of integrations and SDK examples | Very mature; softest lock-in is to Vercel-specific features (self-hosting supported) | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| React Router v7 (Remix lineage) | React framework | Loader/action model suited to form-rich, data-heavy flows and progressive enhancement | Open source (MIT) | Low–medium — smaller ecosystem of turnkey templates than Next.js | Mature; Remix merged into React Router v7; runtime-agnostic | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| SvelteKit (Svelte 5) | Svelte meta-framework | Small bundles and strong runtime performance for mobile clients; form actions built in | Open source (MIT) | Medium — smaller component and SDK ecosystem; different mental model from React | Mature; adapters for many hosts; Svelte-specific code does not port to other frameworks | https://prismic.io/blog/sveltekit-vs-nextjs (accessed 2026-09-29) |
| Nuxt 4 | Vue meta-framework | Full SSR/hybrid rendering with Vue ecosystem; mature i18n and form modules | Open source (MIT) | Medium — Vue-specific ecosystem | Mature; Vue-specific code does not port | https://medium.com/@aryavr2030/next-js-vs-nuxt-vs-remix-which-meta-framework-should-you-learn-in-2026-77c5eb8d3c86 (accessed 2026-09-29) |

### Backend / API Layer

Serves 62 Automation specs, 7 Integration specs (webhooks from payments/email/storage) and role-based portals (profile Sections 1 and 3); no real-time transport needed.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Framework server layer (Next.js route handlers and server actions) | Framework-bundled backend | Covers request/response, webhooks and form mutations in the same deployable as the frontend; long-running work must go to a job runner | Included in framework hosting cost | Low — no separate service | Tied to the frontend framework's runtime and hosting model | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| Hono on Node.js / edge runtimes | TypeScript API framework | Lightweight API routing with middleware, portable across Node, Workers and Bun | Open source (MIT) | Low — TypeScript, works with Drizzle/Kysely | Younger but widely adopted; runtime-portable | knowledge-based — no Hono-specific page fetched this run; characteristics from model knowledge |
| NestJS (Node.js) | TypeScript backend framework | Opinionated modules, DI, guards for role/authorization rules and scheduled tasks | Open source (MIT) | Medium — framework conventions to learn | Mature; large enterprise use | knowledge-based — no NestJS-specific page fetched this run; characteristics from model knowledge |
| Django + Django REST Framework (Python) | Python full-stack framework | Batteries-included ORM, admin, auth, migrations; strong for record-heavy systems | Open source (BSD) | Medium — separate Python service and separate frontend | Very mature; Python-side data-access tooling only | knowledge-based — no Django-specific page fetched this run; characteristics from model knowledge |

### Database

Serves 23 entities, 60 relationships, 35 cross-feature rules, immutable financial records (ASMP-25) and contention on 12 entities (profile Sections 2, 3); infrastructure budget under roughly $100/month (BRIEF.md ## Constraints).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Neon (serverless Postgres) | Managed Postgres | Relational transactions, row-level constraints, branching for environments; scale-to-zero | Free tier; Launch $0.106/CU-hour; Scale $0.222/CU-hour; storage $0.35/GB-month | Low — standard Postgres drivers | Standard Postgres; exit by dump/restore | https://neon.com/pricing (accessed 2026-09-29) |
| Supabase Postgres | Managed Postgres platform | Postgres plus bundled auth, storage and realtime; row-level security for client isolation | Pro $25/month incl. 8 GB DB, 250 GB egress; $0.125/GB beyond 8 GB | Low — Postgres wire protocol plus optional SDK | Standard Postgres; platform features add coupling | https://supabase.com/pricing (accessed 2026-09-29) |
| Amazon RDS for PostgreSQL | Cloud-managed Postgres | Production Postgres with Multi-AZ, backups and read replicas | Instance-hour plus storage; small instances from roughly tens of USD/month | Medium — VPC, IAM and parameter setup | Very mature; Postgres portability; ties to AWS networking | knowledge-based — AWS RDS pricing page not fetched this run; figures from model knowledge |
| Render Managed Postgres | Managed Postgres | Fixed-size instances with predictable billing | From $40/month for 1 vCPU / 2 GB | Low — connection string | Standard Postgres; exit by dump/restore | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |

### ORM / Data Access

Serves a relational model of 23 entities with reject-with-refresh concurrency and an immutable-record model (profile Sections 2, 3) — needs transactions, migrations and typed queries.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Drizzle ORM (v1) | TypeScript ORM / query builder | SQL-close typed queries; built-in migration generation (drizzle-kit); works in edge runtimes | Open source (Apache-2.0) | Low — TypeScript-native schema | v1 stable; code-first schema; TypeScript-only | https://www.pkgpulse.com/guides/drizzle-orm-v1-vs-prisma-6-vs-kysely-2026 (accessed 2026-09-29) |
| Prisma ORM (v6) | TypeScript ORM | Schema-first with generated client, Prisma Studio and Prisma Migrate; wide database support | Open source (Apache-2.0); optional paid Accelerate/Postgres services | Low — own schema language and codegen | Very mature; proprietary schema language; edge runtimes need Accelerate | https://makerkit.dev/blog/tutorials/drizzle-vs-prisma (accessed 2026-09-29) |
| Kysely | TypeScript query builder | Type-safe SQL builder for CTEs and window functions (financial aggregates); no migrations built in beyond a basic migrator | Open source (MIT) | Medium — schema types and migrations managed separately | Mature; thin abstraction, low lock-in | https://www.pkgpulse.com/guides/drizzle-orm-v1-vs-prisma-6-vs-kysely-2026 (accessed 2026-09-29) |
| SQLAlchemy + Alembic | Python ORM and migration tool | Full-featured ORM with mature migrations; Python backend only | Open source (MIT) | Medium — pairs only with a Python backend | Very mature; Python-only | knowledge-based — no SQLAlchemy page fetched this run; characteristics from model knowledge |

### CSS / Styling

Serves mobile-first client screens and freelancer-supplied brand colours/logo that must stay legible (FEAT-19.SPEC-001, FEAT-19.SPEC-003; ASMP-27), requiring token-driven theming.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Tailwind CSS v4 | Utility-first CSS framework | Fast builds (Oxide engine); CSS-variable theme tokens map to per-freelancer brand colours | Open source (MIT) | Low — first-class in all mainstream frameworks | Very mature; utility classes in markup are a migration cost | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-29) |
| CSS Modules | Scoped CSS files | Plain CSS with local scope; tokens as CSS custom properties; zero runtime | Built into major bundlers | Low — no extra dependency | Very mature; web-standard, minimal lock-in | https://www.pkgpulse.com/guides/css-modules-vs-tailwind-2026 (accessed 2026-09-29) |
| vanilla-extract | Build-time TypeScript CSS | Type-safe tokens and themes compiled to static CSS | Open source (MIT) | Medium — bundler plugin per framework | Mature; TypeScript-authored styles | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-29) |
| Panda CSS | Build-time CSS-in-JS | Type-safe tokens, variants and conditions; static extraction | Open source (MIT) | Medium — codegen step | Newer; smaller ecosystem (about 300K weekly downloads per source) | https://www.pkgpulse.com/guides/vanilla-extract-vs-panda-css-vs-tailwind-2026 (accessed 2026-09-29) |

Component-layer candidates (profile evidence: data tables in invoice/deliverable lists, dialogs for confirmations, no command palette; no user-supplied design system named in profile):
- Radix UI primitives (headless, React)
- shadcn/ui (copy-in components on Radix and Tailwind, React)
- Headless UI (headless, React and Vue)
- Melt UI (headless, Svelte)

### State Management

Serves mostly server-derived state (invoices, milestones, comments) with snapshot screens (Real-time = No), reject-with-refresh concurrency, and an offline comment queue (FEAT-07.SPEC-008).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| TanStack Query v5 | Server-state cache library | Fetching, caching, invalidation, retries and offline mutation persistence; multi-framework adapters | Open source (MIT) | Low | Very mature; server-state only | https://saschb2b.com/blog/react-state-management-2026 (accessed 2026-09-29) |
| Zustand | Client-store library (React) | Minimal store for UI state and the offline comment queue | Open source (MIT) | Low | Mature; React-only | https://dev.to/digitalunicon/state-management-in-2026-redux-vs-zustand-vs-react-context-5336 (accessed 2026-09-29) |
| Redux Toolkit (with RTK Query) | State store and data-fetching library (React) | Explicit action flow and RTK Query caching for interdependent client state | Open source (MIT) | Medium — slices, store setup | Very mature; React/Redux ecosystem | https://dev.to/digitalunicon/state-management-in-2026-redux-vs-zustand-vs-react-context-5336 (accessed 2026-09-29) |
| SWR | Data-fetching hook library (React) | Stale-while-revalidate caching for read-mostly screens | Open source (MIT) | Low | Mature; React-only, fewer mutation features | https://saschb2b.com/blog/react-state-management-2026 (accessed 2026-09-29) |

### Build Tooling

Serves a 33-feature web app on a single deployable; framework-bundled pipeline is the default (guide Section 5).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Turbopack (bundled with Next.js 16) | Framework-bundled bundler | Stabilized for production builds in Next.js 16 | Included with Next.js (open source) | None — bundled | Newer than webpack; tied to Next.js | https://dev.to/pockit_tools/nextjs-vs-remix-vs-astro-vs-sveltekit-in-2026-the-definitive-framework-decision-guide-lp5 (accessed 2026-09-29) |
| Vite | Bundler / dev server | Default pipeline for React Router, SvelteKit and Nuxt; fast HMR | Open source (MIT) | Low | Very mature; plugin-based, low lock-in | knowledge-based — no Vite page fetched this run; characteristics from model knowledge |
| pnpm (package manager) | Package manager | Fast, disk-efficient installs; workspace support | Open source (MIT) | Low | Mature; lockfile format is pnpm-specific | knowledge-based — no pnpm page fetched this run; characteristics from model knowledge |
| Bun (package manager and bundler) | JS runtime and toolchain | All-in-one installer, bundler and test runner | Open source (MIT) | Low–medium — runtime compatibility to verify | Younger; runtime-specific behavior differences | knowledge-based — no Bun page fetched this run; characteristics from model knowledge |

### Authentication & Identity

Serves email magic-link sign-in for client contacts (FEAT-05.SPEC-001 to FEAT-05.SPEC-007), freelancer sign-up and login (FEAT-20.SPEC-001, FEAT-21.SPEC-003), sign-out of other sessions (FEAT-21.SPEC-006), four roles, and strict client isolation.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Clerk | Managed identity provider | Magic links, sessions and multi-session management; hosted UI components | Free to 50K monthly retained users; then $25/month plus per-MRU overage | Low — SDKs and prebuilt components | Mature; users held in vendor store, export possible | https://clerk.com/pricing (accessed 2026-09-29) |
| Better Auth | Open-source auth library | Magic-link plugin, 2FA, organizations; runs in own database | Open source (MIT); infrastructure cost only | Medium — own the sessions and tables | Newer; low lock-in (data in own DB) | https://dev.to/thiago_alvarez_a7561753aa/clerk-vs-better-auth-2026-we-verified-every-price-so-you-dont-have-to-13pk (accessed 2026-09-29) |
| Supabase Auth | Managed auth (open-source GoTrue) | Magic links, social login, MFA; ties to Postgres row-level security | Free 50,000 MAU; Pro $25/month incl. 100,000 MAU; $0.00325/MAU beyond | Low — SDK; best with Supabase Postgres | Open-source GoTrue gives an exit path | https://www.buildmvpfast.com/api-costs/authentication (accessed 2026-09-29) |
| WorkOS AuthKit | Managed identity provider | Passwordless and social login; enterprise SSO modules | Free to 1M MAU; SSO from $125/month per connection | Low — SDKs | Mature; hosted user store | https://www.buildmvpfast.com/api-costs/authentication (accessed 2026-09-29) |

### Hosting & Environments

Serves a web app with mobile-first clients worldwide (BRIEF.md Geography), webhook endpoints, scheduled work, and an infrastructure budget under roughly $100/month; no uptime target (ASMP-26).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Vercel | Managed frontend/serverless platform | Native Next.js hosting, preview environments per branch, CDN, cron | Pro $20/seat/month with $20 usage credit, 1 TB transfer; Hobby is non-commercial only | Low — Git-based deploys | Mature; platform-specific features raise coupling | https://vercel.com/docs/plans/pro-plan (accessed 2026-09-29) |
| Fly.io | Container/VM platform | Long-running containers, regions near users, no cold starts | Per-second machine billing; no free tier; sample Node+Postgres stack ~$12/month | Medium — CLI-first, Dockerfile | Mature; standard containers, low lock-in | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |
| Railway | Managed app platform | Git deploys, one-click Postgres and workers | Hobby $5/month, Pro $20/month plus usage; sample stack ~$37/month | Low | Younger platform; standard containers | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |
| Render | Managed app platform | Web services, cron jobs, background workers, managed Postgres | Fixed instance prices plus workspace fee; sample stack ~$30/month | Low | Mature; standard containers | https://dev.to/pavel-hostim/render-vs-railway-vs-flyio-pricing-compared-2026-2e5p (accessed 2026-09-29) |

### CI/CD & Delivery

Serves a 33-feature codebase with financial-correctness rules that require automated checks before promotion (ASMP-25) and multiple environments.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| GitHub Actions | Hosted CI/CD | Native to GitHub repos; environments with approval gates | Private repos: 2,000 min/month Free, 3,000 Team; Linux $0.006/min | Low | Very mature; workflow YAML is GitHub-specific | https://docs.github.com/billing/managing-billing-for-github-actions/about-billing-for-github-actions (accessed 2026-09-29) |
| GitLab CI/CD | Hosted CI/CD | Integrated pipelines and environments in GitLab | Free 400 min/month; Premium from $29/user/month | Medium — requires GitLab hosting of code | Very mature; pipeline syntax is GitLab-specific | https://www.eesel.ai/blog/gitlab-pricing (accessed 2026-09-29) |
| CircleCI | Hosted CI/CD | Fast parallel pipelines, reusable orbs | Free 6,000 credits/month; Performance credits $15 per 25,000 | Low–medium | Mature; config is CircleCI-specific | https://circleci.com/pricing/ (accessed 2026-09-29) |
| Platform-native deploy pipelines (e.g. Vercel / Render Git deploys) | Hosting-bundled CD | Build-and-deploy on push with preview environments; tests still need a separate runner | Included with hosting plan | Low | Tied to the hosting platform | https://vercel.com/docs/plans/pro-plan (accessed 2026-09-29) |

### Observability & Operations

Serves failed-email visibility within minutes (ASMP-26), an immutable audit trail (FEAT-13) and GDPR-class data in logs (Compliance/privacy = Yes) on a small budget.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Sentry | SaaS error and performance monitoring | Error reporting, tracing and replay; framework SDKs | Team $26/month incl. 50K errors; tiered overage from $0.0003625/error | Low — SDK per framework | Very mature; SDK is Sentry-specific, self-hosting available | https://blog.struct.ai/sentry-pricing-error-monitoring-2026/ (accessed 2026-09-29) |
| Grafana Cloud | SaaS metrics, logs and traces | Loki logs, Prometheus metrics, alerting in one bill | Logs from ~$0.50/GB ingested; free tier available | Medium — OpenTelemetry or agent setup | Mature; open-source components limit lock-in | https://kanopylabs.com/blog/axiom-vs-datadog-vs-grafana-cloud (accessed 2026-09-29) |
| Datadog | SaaS observability suite | Infra, APM and logs in one product | Infrastructure $15/host/month, APM $31/host/month, logs $0.10/GB | Medium | Very mature; proprietary agent and dashboards | https://blog.railway.com/p/best-cloud-observability-tools-2026 (accessed 2026-09-29) |
| Axiom | SaaS log and trace store | Serverless-friendly log ingestion for logs and traces at low cost | Usage-based; source describes it as a fraction of Datadog's cost | Low — HTTP ingest and platform integrations | Younger; scope narrower than full suites | https://kanopylabs.com/blog/axiom-vs-datadog-vs-grafana-cloud (accessed 2026-09-29) |

### File & Object Storage

Serves deliverables of tens of MB to over 1 GB with resumable upload, version history for the account's life, storage limits, purge on deletion, and generated export archives (FEAT-06.SPEC-003, FEAT-16.SPEC-002, FEAT-16.SPEC-004, FEAT-16.SPEC-006, FEAT-17, FEAT-24.SPEC-003); storage and bandwidth cost must fit the budget (BRIEF.md ## Scale & Non-Functional Expectations).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Cloudflare R2 | S3-compatible object storage | Multipart upload with presigned URLs; zero egress fees suit large downloads | $0.015/GB-month; no egress; Class A $4.50/M, Class B $0.36/M; 10 GB free | Low — S3 API | Mature; S3 API keeps exit path open | https://egresscost.com/cloudflare/ (accessed 2026-09-29) |
| Amazon S3 | Cloud object storage | Multipart and resumable patterns, lifecycle rules, deepest tooling | Standard $0.023/GB-month first 50 TB plus requests and data-transfer-out | Low–medium — IAM and CORS | Very mature; egress cost makes leaving costly | https://aws.amazon.com/s3/pricing/ (accessed 2026-09-29) |
| Backblaze B2 | S3-compatible object storage | Low-cost storage; free egress up to 3x stored data and free via Cloudflare Bandwidth Alliance | $0.00695/GB-month; egress beyond 3x at $0.01/GB | Low — S3-compatible API | Mature; S3-compatible | https://www.backblaze.com/cloud-storage/pricing (accessed 2026-09-29) |
| Supabase Storage | Managed storage on Postgres platform | Resumable (TUS) uploads, access rules tied to row-level security | Counted against plan egress quota (Pro 250 GB, $0.09/GB beyond) plus storage overage | Low if Supabase is in use | Couples to Supabase platform | https://supabase.com/pricing (accessed 2026-09-29) |

### Email & Messaging Delivery

Serves email-only notification delivery (26 Notification specs; FEAT-14.SPEC-001) with delivery and bounce status reporting (FEAT-14.SPEC-003) and branded presentation (FEAT-14.SPEC-005); push and SMS are not named in any spec.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Resend | Managed email API | Developer-first API, webhooks for delivery events, React email templates | Free 3,000 emails/month; Pro $20/month for 50,000 | Low — REST and SDKs | Newer entrant; SMTP interface limits lock-in | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |
| Postmark | Managed transactional email | Deliverability-focused, message streams, bounce webhooks | $15/month base plus $1.80 per extra 1K emails | Low — REST and SDKs | Long established; proprietary streams | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |
| Amazon SES | Cloud email service | Very low-cost sending with event publishing via SNS | $0.10 per 1,000 emails plus $0.12/GB attachments | Medium — IAM, domain and reputation setup | Very mature; ties event handling to AWS | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |
| SendGrid (Twilio) | Managed email platform | Transactional and marketing email, event webhooks | About $0.40 per 1,000 at paid tiers per comparison source | Low–medium | Very mature; Twilio account coupling | https://www.buildmvpfast.com/api-costs/email (accessed 2026-09-29) |

### Payments & Billing

Serves card and bank-transfer payment into each freelancer's own processor account with the platform never touching card data (FEAT-10.SPEC-003, FEAT-32.SPEC-002; ASMP-24, ASMP-28), reversal notices (FEAT-25.SPEC-005), and separate recurring billing of freelancers for their own plan (FEAT-23.SPEC-003; ASMP-31); no cut of payments (BRIEF.md ## Business Context).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Stripe (Connect Standard with direct charges + Stripe Billing) | Payment platform | Freelancers connect their own account; funds land in their account; webhooks for status, disputes; Billing covers own-plan subscriptions | Direct charges on Standard accounts: Stripe fees charged to the connected account, no Connect fee to platform; 2.9% + 30c cards; Billing 0.7% of billing volume | Medium — OAuth/onboarding, webhooks, idempotency | Very mature; broadest SDK coverage; payment method and customer data are portable only via Stripe processes | https://docs.stripe.com/connect/charges (accessed 2026-09-29) |
| Adyen for Platforms | Payment platform | Global acquiring for marketplaces; sales-led onboarding | Interchange++ pricing with monthly minimums | High — sales process and heavier integration | Very mature; enterprise-oriented | https://www.chargeflow.io/blog/stripe-vs-adyen (accessed 2026-09-29) |
| PayPal Commerce Platform | Payment platform | Payment acceptance in 200+ markets; marketplace onboarding | Transaction fees per PayPal rate card | Medium | Very mature; consumer-brand recognition | https://www.inflowpay.com/blog/top-6-paypal-alternatives (accessed 2026-09-29) |
| Mollie Connect | Payment platform | Strong European local payment methods; blended per-transaction fees | Blended per-transaction fee | Medium | Mature; Europe-focused | https://www.mollie.com/growth/mollie-vs-adyen (accessed 2026-09-29) |
| Paddle (own-plan subscriptions only) | Merchant of Record billing | Handles subscription billing and worldwide tax for the platform's own plan, not per-freelancer payouts | About 5% + 50c per transaction | Medium | Mature; separates from client-payment processor | https://aliteq.com/stripe-vs-paddle-vs-lemon-squeezy-ai-saas-2026 (accessed 2026-09-29) |

### Search

Serves FEAT-28 cross-entity global search scoped by access rules (FEAT-28.SPEC-003) with rule-based ranking (FEAT-28.SPEC-004); no full-text or faceted keywords in specs; a few thousand freelancers with 3–15 clients each.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostgreSQL full-text search (tsvector/pg_trgm) | Database feature | Search inside existing data store, honoring same access rules | Included with database | Low — SQL queries and indexes | Mature; standard Postgres | knowledge-based — no Postgres documentation fetched this run; characteristics from model knowledge |
| Typesense | Open-source search engine / cloud | Typo-tolerant instant search; self-host or cloud | Cloud from ~$14/month or free self-hosted | Medium — sync index with database | Mature; open source | https://www.buildmvpfast.com/api-costs/search (accessed 2026-09-29) |
| Meilisearch | Open-source search engine / cloud | Relevance-focused search with simple API | Cloud from ~$20–30/month or free self-hosted | Medium — sync index | Mature; open source | https://www.buildmvpfast.com/api-costs/search (accessed 2026-09-29) |
| Algolia | Managed search service | Full-featured hosted search | ~$0.50 per 1K records and $0.40 per 1K searches on developer tier | Medium — indexing pipeline | Very mature; proprietary | https://www.buildmvpfast.com/api-costs/search (accessed 2026-09-29) |

### Background Jobs & Scheduling

Serves reminder schedules (FEAT-11.SPEC-001), totals refresh (FEAT-12.SPEC-004), email retry (FEAT-14.SPEC-003), export archive generation (FEAT-24.SPEC-003), legal retention purge (FEAT-24.SPEC-005), support-session auto-close (FEAT-31.SPEC-004) and reliable webhook processing.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Inngest | Managed durable-function platform | Event-driven steps, retries, cron; calls own serverless endpoints | Free 25K runs/month; Basic $30/month; Pro $300/month | Low — SDK | Managed only, no self-host; proprietary function model | https://www.buildmvpfast.com/api-costs/background-jobs (accessed 2026-09-29) |
| Trigger.dev v3 | Managed / self-hostable job platform | Long-running tasks without serverless timeouts; schedules | Free 50K runs/month; paid from $10/month | Low–medium | Open source (MIT); self-host via Docker | https://www.buildmvpfast.com/api-costs/background-jobs (accessed 2026-09-29) |
| Upstash QStash | Managed HTTP message queue and scheduler | Scheduled and retried HTTP calls to endpoints | Usage-based per message | Low — HTTP | Managed; simple model, low lock-in | https://www.buildmvpfast.com/api-costs/background-jobs (accessed 2026-09-29) |
| pg-boss / BullMQ workers | Open-source queue libraries | Postgres-backed (pg-boss) or Redis-backed (BullMQ) queues with cron; needs a long-running worker | Free; infrastructure cost of worker (and Redis for BullMQ) | Medium — operate workers | Mature; open source | knowledge-based — no pg-boss or BullMQ page fetched this run; characteristics from model knowledge |

### Caching & Performance

Serves ~2 s mobile interactivity (ASMP-21), dashboard totals within 1–2 s (ASMP-21) and degraded/offline read states (FEAT-29.SPEC-004; ASMP-27) at a few thousand freelancers.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Upstash Redis | Managed serverless Redis | Rate limiting and cached aggregates over HTTP | Free 256 MB / 500K commands; pay-as-you-go $0.20 per 100K commands; fixed from $10/month | Low — REST and SDK | Managed; Redis-protocol exit | https://upstash.com/pricing (accessed 2026-09-29) |
| Redis Cloud | Managed Redis | Conventional Redis with persistence options | Plan-based | Low–medium | Mature; Redis protocol portability | https://upstash.com/blog/redis-pricing-comparison-every-major-provider-in-2026-with-numbers (accessed 2026-09-29) |
| CDN and framework cache (Cloudflare CDN / Next.js cache) | Edge/HTTP caching | Static assets, cached public pages and revalidation without a separate service | Included with hosting/CDN plans | Low | Framework/CDN-specific semantics | https://vercel.com/docs/plans/pro-plan (accessed 2026-09-29) |
| Database-level caching (materialized views / read replicas) | Database feature | Precomputed financial totals without another service | Included with database, replica cost extra | Low–medium | Standard Postgres | knowledge-based — no Postgres documentation fetched this run; characteristics from model knowledge |

### Real-time & Collaboration

Real-time = No; the active signal is Collaboration/concurrency: exactly-once acceptance (FEAT-03.SPEC-003), approval concurrency guard (FEAT-08.SPEC-003) and reject-with-refresh on 12 contended entities, between one freelancer and client contacts rather than shared editing.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Database optimistic concurrency and transactions (row versions, unique constraints) | Database technique | Reject-with-refresh and exactly-once semantics inside the data store, no transport service | Included with database | Low–medium — implemented in data layer | Standard SQL | knowledge-based — technique not tied to a vendor page; from model knowledge |
| Supabase Realtime | Managed realtime over Postgres changes | Push updates of row changes if live refresh is wanted | Included in Supabase plans, usage quotas apply | Low if Supabase in use | Couples to Supabase | https://supabase.com/pricing (accessed 2026-09-29) |
| Ably | Managed pub/sub | Message fan-out with presence and history | Free 6M messages/month; consumption $2.50 per million | Low — SDK | Mature; proprietary protocol | https://ably.com/compare/ably-vs-pusher (accessed 2026-09-29) |
| Pusher Channels | Managed pub/sub | Simple channels for change notifications | From $49/month; free tier 200 connections | Low — SDK | Mature; tiered plans | https://ably.com/compare/ably-vs-pusher (accessed 2026-09-29) |

### Analytics & Product Telemetry

Serves product success metrics for a few thousand freelancers (ASMP-22) with GDPR-class data handling (ASMP-24); referral attribution capture exists as a product feature (FEAT-33).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| PostHog | SaaS / self-hostable product analytics | Events, funnels, feature flags, session replay | Free 1M events/month; then from $0.00005/event | Low — SDK | Open source; self-host option | https://posthog.com/pricing (accessed 2026-09-29) |
| Mixpanel | SaaS product analytics | Event funnels and retention | Free 20M events/month; Growth ~$0.00028/event | Low | Very mature; proprietary | https://www.buildmvpfast.com/api-costs/analytics (accessed 2026-09-29) |
| Amplitude | SaaS product analytics | Behavioral analysis and experimentation | Free basic tier; growth from $49/month | Low | Very mature; proprietary | https://www.buildmvpfast.com/api-costs/analytics (accessed 2026-09-29) |
| Plausible | Privacy-first web analytics | Cookie-less page-level analytics; light product event support | Subscription by pageviews | Low — one script | Mature; open source, self-hostable | https://www.buildmvpfast.com/api-costs/analytics (accessed 2026-09-29) |

### Internationalization

Serves worldwide currencies, tax and time zones (FEAT-15.SPEC-001, FEAT-15.SPEC-002, FEAT-15.SPEC-006, FEAT-15.SPEC-008); English only at launch, no translation specs; multi-currency dashboard does not aggregate (FEAT-15.SPEC-007).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Native Intl APIs (Intl.NumberFormat, Intl.DateTimeFormat) | Platform standard | Currency, date and time-zone formatting with no translation layer | Free (built into runtimes) | Low | Web standard; no lock-in | https://tolgee.io/blog/react-i18n-libraries-comparison (accessed 2026-09-29) |
| next-intl | Library (Next.js) | Server Component friendly message and format handling for future locales | Open source (MIT) | Low — Next.js-specific | Mature; Next.js-bound | https://www.pkgpulse.com/guides/next-intl-vs-react-i18next-vs-lingui-react-i18n-2026 (accessed 2026-09-29) |
| i18next / react-i18next | Library | Framework-agnostic translation and formatting | Open source (MIT) | Low | Very mature; ~2.8M weekly downloads per source | https://tolgee.io/blog/react-i18n-libraries-comparison (accessed 2026-09-29) |
| Lingui | Library | ICU-standard, compile-time extraction, ~3KB runtime | Open source (MIT) | Medium — macros and extraction | Mature; ICU/PO workflows | https://www.pkgpulse.com/guides/next-intl-vs-react-i18next-vs-lingui-react-i18n-2026 (accessed 2026-09-29) |

### Electronic Signature Attestation

Serves FEAT-26.SPEC-005: attesting signature data captured in-portal so signed acceptance has legal weight beyond a self-recorded timestamp; jurisdictional standard left to this stage; v1 phase; sent to both parties as signed copy (FEAT-26.SPEC-004).

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Documenso | Open-source e-signature platform | Embedded signing and API; white-labeling on Platform tier; self-hostable | Free 5 docs/month; Individual $30/month; Platform $250/month | Medium — embed and API | Newer; open source with self-host exit | https://dev.to/beton/documenso-pricing-teardown-2026-3ic6 (accessed 2026-09-29) |
| DocuSign eSignature API | Managed e-signature API | Embedded signing with widely recognized audit certificates | Essentials $75/month for 50 requests; Standard $250/month for 100; embedded signing on a ~$480/month tier | Medium–high | Very mature; per-envelope pricing | https://signb.ee/blog/embedded-esignature-api-cost-calculator (accessed 2026-09-29) |
| Dropbox Sign API | Managed e-signature API | Embedded signing, templates, webhooks | Subscription from ~$100/month billed monthly | Medium | Mature; per-request tiers | https://www.signwell.com/resources/dropbox-sign-api/ (accessed 2026-09-29) |
| SignWell API | Managed e-signature API | Embedded signing and API for lower-volume use | Plan-based | Medium | Mature; SaaS | https://www.signwell.com/resources/dropbox-sign-api/ (accessed 2026-09-29) |

### Custom Domain Verification & TLS Serving

Serves FEAT-27.SPEC-002: verify a freelancer controls a domain (DNS check) and serve the portal over HTTPS at it, with shared default address as fallback (FEAT-27.SPEC-003); Later phase, at most one per freelancer, a few thousand accounts.

| Option | Type | Capability Fit | Pricing Model | Integration Effort | Maturity & Lock-in | Sources |
|---|---|---|---|---|---|---|
| Cloudflare for SaaS (Custom Hostnames) | Managed custom-hostname service | Automated certificate issuance and validation via API; fallback origin | 100 hostnames free on Free/Pro/Business; then $0.10/hostname/month to 50,000 | Medium — API and DNS setup | Mature; requires traffic through Cloudflare | https://domainee.dev/blog/cloudflare-for-saas-pricing (accessed 2026-09-29) |
| Vercel Domains API | Hosting-platform domain management | Add and verify domains per project via API with automatic TLS | Included in Vercel plans; limits per plan | Low if hosted on Vercel | Couples to Vercel | knowledge-based — Vercel domains documentation not fetched this run; from model knowledge |
| Caddy on-demand TLS | Open-source web server | Issues certificates on demand for verified hosts; self-hosted | Free; server infrastructure cost | Medium–high — operate proxy | Mature; open source | knowledge-based — Caddy documentation not fetched this run; from model knowledge |
| Fly.io custom domain certificates | Hosting-platform certificates | Certificates for custom hostnames via API on Fly apps | Included with Fly usage; certificate limits per app | Medium | Couples to Fly.io | knowledge-based — Fly.io certificate documentation not fetched this run; from model knowledge |

## 3. Cross-Area Compatibility Notes

- Drizzle ORM, Prisma and Kysely (ORM / Data Access) are TypeScript/Node data-access layers; they pair with the Node-runtime options in Backend / API Layer (framework server layer, Hono, NestJS) and not with Django. SQLAlchemy is Python-only and pairs with Django, not a Node backend.
- Every Postgres option in Database (Neon, Supabase Postgres, Amazon RDS, Render Postgres) is standard Postgres, supported by Drizzle, Prisma, Kysely and SQLAlchemy.
- Prisma cannot run in edge runtimes without Prisma Accelerate; Drizzle and Kysely run on edge runtimes (Vercel Edge, Workers).
- TanStack Query and Zustand have React bindings and TanStack Query has adapters for Vue and Svelte; SWR and Redux Toolkit are React-only, so they pair only with Next.js or React Router in Frontend Framework. Svelte projects use Svelte-native stores.
- shadcn/ui assumes React plus Tailwind CSS; Radix primitives are React; Headless UI supports React and Vue; Melt UI is Svelte.
- Tailwind CSS, CSS Modules and vanilla CSS approaches pair with all Frontend Framework options; next-intl is Next.js-specific, whereas i18next and Lingui support several frameworks.
- Next.js ships Turbopack; choosing Vite for a Next.js project overrides the bundled pipeline. React Router, SvelteKit and Nuxt use Vite by default.
- Vercel is native for Next.js; Fly.io, Railway and Render run any container. Vercel Hobby is limited to non-commercial use.
- Supabase Auth, Storage and Realtime bind to Supabase Postgres; Clerk, Better Auth and WorkOS AuthKit are independent of the database choice, and Better Auth stores users in the application's own database via the ORM.
- Cloudflare R2, Backblaze B2 and Amazon S3 share the S3 API, so one S3 client library works against all three; B2 egress via Cloudflare is free through the Bandwidth Alliance.
- Cloudflare for SaaS requires custom-hostname traffic to pass through Cloudflare; Vercel Domains API and Fly.io certificates require the portal to run on that platform; Caddy on-demand TLS requires a self-operated proxy.
- Stripe, Resend, Postmark, SendGrid, Amazon SES, Clerk, Sentry, PostHog and Inngest all publish official Node/TypeScript SDKs; Documenso and DocuSign offer HTTP APIs usable from any backend language. SDK availability for Python backends is present for Stripe, Sentry, PostHog, Amazon SES and Postmark; Inngest offers a Python SDK.
- Stripe Connect Standard with direct charges places Stripe fees on the connected account and does not apply platform pricing tools; Stripe Billing for the platform's own subscription is a separate charge stream from client payments; Paddle covers only the platform's own subscription and does not replace a client-payment processor.
- Trigger.dev and Inngest run job code against the application's own endpoints or compute; pg-boss requires Postgres and BullMQ requires Redis, which links Background Jobs & Scheduling to the Database and Caching & Performance options.
- Supabase Realtime and Supabase Storage are only available on Supabase; Ably and Pusher work with any backend.

## 4. Research Log

| Decision Area | Method | Queries & Key Sources | Access Date |
|---|---|---|---|
| Frontend Framework | web | "Next.js vs SvelteKit vs React Router Remix vs Nuxt 2026 comparison full-stack framework"; dev.to, prismic.io, medium.com comparisons | 2026-09-29 |
| Backend / API Layer | web (partial) with knowledge-based fallback | Same framework comparison query for framework server layer. Hono, NestJS and Django rows are knowledge-based — no dedicated page fetched this run; characteristics from model knowledge | 2026-09-29 |
| Database | web (partial) with knowledge-based fallback | "Neon Postgres pricing Launch Scale plan"; neon.com/pricing; supabase.com/pricing; Render via hosting comparison. Amazon RDS row knowledge-based — AWS RDS pricing page not fetched this run | 2026-09-29 |
| ORM / Data Access | web (partial) with knowledge-based fallback | "Drizzle ORM vs Prisma vs Kysely 2026"; makerkit.dev, pkgpulse.com. SQLAlchemy + Alembic row knowledge-based — no SQLAlchemy page fetched this run | 2026-09-29 |
| CSS / Styling | web | "Tailwind CSS v4 vs CSS Modules vs Panda CSS vs vanilla-extract 2026"; pkgpulse.com guides. Component-layer note list from decision guide Section 6 and model knowledge (no prices claimed) | 2026-09-29 |
| State Management | web | "TanStack Query vs Zustand vs Redux Toolkit vs SWR 2026"; saschb2b.com, dev.to | 2026-09-29 |
| Build Tooling | web (partial) with knowledge-based fallback | Turbopack status from the Next.js framework comparison. Vite, pnpm and Bun rows knowledge-based — no dedicated pages fetched this run | 2026-09-29 |
| Authentication & Identity | web | "Clerk pricing monthly active users magic link Auth.js Better Auth"; "Auth0 vs Supabase Auth vs WorkOS AuthKit pricing 2026"; clerk.com/pricing; buildmvpfast.com/api-costs/authentication; dev.to Clerk vs Better Auth | 2026-09-29 |
| Hosting & Environments | web | "Vercel pricing Pro plan"; "Fly.io Railway Render pricing 2026"; vercel.com/docs/plans/pro-plan; dev.to pricing comparison | 2026-09-29 |
| CI/CD & Delivery | web | "GitHub Actions pricing 2026; CircleCI pricing; GitLab CI"; docs.github.com billing; circleci.com/pricing; eesel.ai GitLab pricing | 2026-09-29 |
| Observability & Operations | web | "Sentry pricing Team plan 2026"; "Better Stack vs Grafana Cloud vs Datadog vs Axiom pricing"; blog.struct.ai; kanopylabs.com; blog.railway.com | 2026-09-29 |
| File & Object Storage | web | "Cloudflare R2 pricing"; "Backblaze B2 pricing; Amazon S3 pricing"; egresscost.com; backblaze.com/cloud-storage/pricing; aws.amazon.com/s3/pricing; supabase.com/pricing | 2026-09-29 |
| Email & Messaging Delivery | web | "Resend Postmark Amazon SES pricing transactional email 2026"; buildmvpfast.com/api-costs/email | 2026-09-29 |
| Payments & Billing | web | "Stripe Connect pricing standard accounts direct charges fees"; "Stripe Billing pricing ... Paddle Lemon Squeezy"; "Adyen for Platforms vs PayPal Commerce vs Mollie"; docs.stripe.com/connect/charges; chargeflow.io; mollie.com; aliteq.com. PayPal and Adyen pricing figures not retrieved as numbers; rows describe the pricing shape only | 2026-09-29 |
| Search | web (partial) with knowledge-based fallback | "Meilisearch Typesense Algolia pricing 2026"; buildmvpfast.com/api-costs/search. PostgreSQL full-text search row knowledge-based — no Postgres documentation fetched this run | 2026-09-29 |
| Background Jobs & Scheduling | web (partial) with knowledge-based fallback | "Inngest Trigger.dev pricing 2026"; buildmvpfast.com/api-costs/background-jobs. pg-boss / BullMQ row knowledge-based — no dedicated page fetched this run | 2026-09-29 |
| Caching & Performance | web (partial) with knowledge-based fallback | "Upstash Redis pricing 2026"; upstash.com/pricing; upstash.com blog. Database-level caching row knowledge-based — no Postgres documentation fetched this run | 2026-09-29 |
| Real-time & Collaboration | web (partial) with knowledge-based fallback | "Ably Pusher Channels Liveblocks pricing 2026"; ably.com/compare/ably-vs-pusher; supabase.com/pricing. Optimistic-concurrency row knowledge-based — technique not tied to a vendor page | 2026-09-29 |
| Analytics & Product Telemetry | web | "PostHog pricing"; "Plausible vs Mixpanel vs Amplitude pricing 2026"; posthog.com/pricing; buildmvpfast.com/api-costs/analytics. Plausible pricing figure not retrieved | 2026-09-29 |
| Internationalization | web | "next-intl vs i18next vs Lingui vs FormatJS 2026"; tolgee.io; pkgpulse.com | 2026-09-29 |
| Electronic Signature Attestation | web | "Documenso vs DocuSign API vs Dropbox Sign API pricing embedded signing 2026"; dev.to Documenso teardown; signb.ee; signwell.com. SignWell pricing figure not retrieved | 2026-09-29 |
| Custom Domain Verification & TLS Serving | web (partial) with knowledge-based fallback | "Cloudflare for SaaS custom hostnames pricing per hostname"; domainee.dev. Vercel Domains API, Caddy on-demand TLS and Fly.io certificate rows knowledge-based — documentation pages not fetched this run | 2026-09-29 |


## Feasibility Assessment


# Technical Feasibility Assessment

## 1. Feasibility Summary

| Feature | Verdict | Driving Factors |
|---------|---------|-----------------|
| FEAT-01 (Client & Project Management) | Straightforward | Stateful CRUD with commit-time re-checks on archive/delete (FEAT-01.SPEC-007 ## Processing Logic; feature-dependency-map.md Client **Contention:**) and a plan-limit check at add time (FEAT-01.SPEC-008); no external service |
| FEAT-02 (Proposal Creation & Sending) | Straightforward | Versioned draft/send/void-and-resend lifecycle (FEAT-02.SPEC-005, FEAT-02.SPEC-006 ## Processing Logic) with immutability once sent (XBR-04); email goes through the shared FEAT-14 capability |
| FEAT-03 (Proposal Acceptance) | Straightforward | Exactly-once acceptance is a single atomic conditional write (FEAT-03.SPEC-003 ## Processing Logic step 5) plus a schedule-as-of-acceptance deposit trigger (XBR-01) — standard transactional technique |
| FEAT-04 (Milestone & Payment Schedule Setup) | Straightforward | Small single-project editor (feature-overview.md ## Non-Functional Notes) with a queued offline save (FEAT-04.SPEC-001 Offline/Degraded state, AC-15) and non-retroactive dated adjustments (Payment Schedule **Contention:**) |
| FEAT-05 (Client Portal Access (Magic-Link Login)) | Standard-with-integration | Passwordless single-use, time-limited, per-freelancer tokens (FEAT-05.SPEC-004 ## Processing Logic; XBR-28) delivered through the transactional email capability (FEAT-05.SPEC-008) under strict isolation (FEAT-05.SPEC-007) |
| FEAT-06 (Deliverable Upload & Sharing) | Standard-with-integration | Resumable large-file ingestion via the storage capability (FEAT-06.SPEC-003; FEAT-16.SPEC-007 ## Degradation Behavior) and outbound reachability checks on Figma/Drive/Dropbox links (FEAT-06.SPEC-004 ## Processing Logic) |
| FEAT-07 (Deliverable Review & Feedback) | Straightforward | Append-only threads with no contention (Comment **Contention:** None) plus a device-local offline comment queue replayed on reconnect (FEAT-07.SPEC-008 ## Processing Logic) |
| FEAT-08 (Milestone Approval) | Straightforward | Approval guarded by a live-state-vs-shown-state comparison and exactly-once write (FEAT-08.SPEC-003 ## Processing Logic steps 4–6) — optimistic concurrency, no external service |
| FEAT-09 (Invoice Generation & Sending) | Hard | Gapless per-freelancer sequential numbering across automatic and manual paths (FEAT-09.SPEC-007 "never reused, never skipped"), generation fired atomically from three other features' actions (FEAT-09.SPEC-004 ## Trigger Definition) and evidentiary immutability with credit-note correction (FEAT-09.SPEC-008) |
| FEAT-10 (Invoice Payment Processing) | Hard | Multi-writer invoice state with asynchronous, duplicated and out-of-order processor events that are authoritative over manual records (FEAT-10.SPEC-003 ## Edge Cases; Invoice and Payment **Contention:**) |
| FEAT-11 (Automated Payment Reminders) | Straightforward | Scheduled day-3/day-10 evaluation in the freelancer's time zone with a send-time eligibility re-check (FEAT-11.SPEC-001 ## Trigger Definition, ## Processing Logic step 4) on shared background-job machinery |
| FEAT-12 (Freelancer Financial Dashboard) | Straightforward | Event-triggered per-currency recomputation (FEAT-12.SPEC-004 ## Trigger Definition) at small per-account volume with a 1–2 s target (ASMP-21) |
| FEAT-13 (Immutable Activity & Audit Trail) | Straightforward | Append-only, deduplicated entry recording (FEAT-13.SPEC-003 ## Processing Logic step 3) and immutability rules (FEAT-13.SPEC-004); printable copy (FEAT-13.SPEC-002) has no landscape coverage |
| FEAT-14 (Notifications (Email)) | Standard-with-integration | Transactional email integration with delivery/bounce webhooks, idempotent out-of-order status handling and a 3-retry / 6-hour retry policy (FEAT-14.SPEC-001 ## Edge Cases; FEAT-14.SPEC-003 ## Processing Logic) |
| FEAT-15 (Currency & Tax Handling) | Straightforward | Freelancer-configured currency and tax label/rate with no automatic calculation (feature-overview.md ## Non-Functional Notes, SC-16), lock after first invoice (FEAT-15.SPEC-004) and time-zone display (FEAT-15.SPEC-006) |
| FEAT-16 (Large File Handling & Storage) | Standard-with-integration | Chunked resumable transfer up to a 2 GB ceiling (FEAT-16.SPEC-002 ## Processing Logic; platform-parameters.md `deliverable-file-size-ceiling`) and byte-range delivery via an object-storage capability within a ~$100/month budget (FEAT-16.SPEC-007 ## Capability Category) |
| FEAT-17 (Deliverable Version History) | Straightforward | Immutable append-only versions (Deliverable Version **Contention:** None; FEAT-17.SPEC-004) consuming FEAT-16's storage; comments anchored per version (XBR-13) |
| FEAT-18 (Client Contact Management & Roles) | Straightforward | Contact CRUD with unique-email-per-client reject-with-refresh and commit-time last-Primary check (Client Contact **Contention:**; FEAT-18.SPEC-006), plus erasure-with-evidence retention (FEAT-18.SPEC-009) |
| FEAT-19 (Freelancer Branding) | Straightforward | Small logo upload (2 MB ceiling) and contrast-ratio legibility adjustment (FEAT-19.SPEC-002; platform-parameters.md `branding-color-legibility-contrast-ratio`) |
| FEAT-20 (Onboarding / First-Run Setup) | Straightforward | Guided multi-step sequence with completion detection (FEAT-20.SPEC-002, FEAT-20.SPEC-003) over sign-up handled by the shared identity capability (FEAT-20.SPEC-001) |
| FEAT-21 (Settings & Account Management) | Straightforward | Configuration forms plus session-list/sign-out-others and re-verified email change (FEAT-21.SPEC-005, FEAT-21.SPEC-006) served by the shared identity capability |
| FEAT-22 (Accounting Export) | Straightforward | Bounded date-range read grouped per currency into CSV or QuickBooks/Xero-compatible files (FEAT-22.SPEC-002 ## Processing Logic); target-format specifics not covered by the landscape |
| FEAT-23 (Subscription Plan & Billing Management) | Standard-with-integration | Recurring billing capability with asynchronous, out-of-order charge/renewal/period-end events and acknowledgment-gated cancellation (FEAT-23.SPEC-003 ## Degradation Behavior, ## Edge Cases) |
| FEAT-24 (Data Export & Account Deletion) | Hard | Whole-account archive aggregation (FEAT-24.SPEC-003 ## Processing Logic step 4) and a staged, never-half-deleted deletion spanning database, object storage and the payment connection, with a 7-year retention purge sweep (FEAT-24.SPEC-004, FEAT-24.SPEC-005) |
| FEAT-25 (Refund & Cancelled Project Handling) | Standard-with-integration | Manual refund/cancellation recording is plain state change (FEAT-25.SPEC-003, FEAT-25.SPEC-004); chargeback recording consumes processor reversal notices relayed through FEAT-32 (FEAT-25.SPEC-005; XBR-21) |
| FEAT-26 (Legally Binding E-Signature for Proposals) | Research-spike recommended | The jurisdictional standard and the attest-in-portal-captured-signature interaction model are unresolved (FEAT-26.SPEC-005 ## Capability Category; feature-overview.md ## Non-Functional Notes, Compliance flags) |
| FEAT-27 (Custom Domain per Freelancer) | Standard-with-integration | DNS verification and automated TLS for per-freelancer hostnames with fallback to the shared address (FEAT-27.SPEC-002 ## Degradation Behavior; XBR-35); Later phase |
| FEAT-28 (Global Search Across Clients & Projects) | Straightforward | Account-scoped cross-entity matching with rule-based ranking over a small per-account corpus (FEAT-28.SPEC-002 ## Processing Logic; FEAT-28.SPEC-004; feature-overview.md ## Non-Functional Notes) |
| FEAT-29 (In-App Notification Center) | Straightforward | 90-day rolling feed with device-cached last-loaded fallback (FEAT-29.SPEC-004 ## Processing Logic); Later phase |
| FEAT-30 (Contextual Help & Guidance) | Straightforward | Static content plus one bounded dismissal flag per tip per user (FEAT-30.SPEC-004; feature-overview.md ## Non-Functional Notes) |
| FEAT-31 (Operator Support Access) | Straightforward | Read-only, one-account-at-a-time operator session with inactivity auto-close (FEAT-31.SPEC-003, FEAT-31.SPEC-004 ## Processing Logic; XBR-29) — an established impersonation pattern, with product-wide enforcement coverage as the risk |
| FEAT-32 (Payment Account Connection) | Standard-with-integration | Per-freelancer processor-account onboarding hand-off and authoritative status/reversal events (FEAT-32.SPEC-002 ## Degradation Behavior, ## Edge Cases) |
| FEAT-33 (Portal Referral Attribution) | Straightforward | Referral reference captured with a 30-minute inactivity window and recorded once at sign-up (FEAT-33.SPEC-002, FEAT-33.SPEC-004; platform-parameters.md `referral-capture-session-window`) |

## 2. Per-Feature Assessments

### FEAT-01 — Client & Project Management

**Verdict:** Straightforward — stateful CRUD with commit-time re-validation (FEAT-01.SPEC-007 ## Processing Logic; FEAT-01.SPEC-009, FEAT-01.SPEC-010) and a derived project stage (FEAT-01.SPEC-011); no external service is part of the feature's definition.

**Required Capabilities:**
- Relational client/project records with derived stage and archive/delete eligibility rules (FEAT-01.SPEC-009 Client Delete Eligibility; FEAT-01.SPEC-011 Project Stage Derivation; XBR-24)
- Active-client limit enforcement read against the current plan at add/reactivate time (FEAT-01.SPEC-008; XBR-23)
- Completion-invoice trigger handed to FEAT-09 (FEAT-01.SPEC-006; XBR-03)
- Concurrency: last-write-wins on descriptive fields; archive/delete/complete/cancel are reject-with-refresh if open items or state changed since load (feature-dependency-map.md, Client and Project **Contention:**); plan-change races resolved at the moment of add (Subscription Plan **Contention:**)
- Offline/degraded: N/A — no offline mandate beyond ASMP-27's "say plainly when an action needs a connection"; roster renders skeleton rows while loading (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: 3–15 active clients per freelancer, a few thousand freelancers in year one; roster stays responsive at that size from MVP (feature-overview.md ## Non-Functional Notes; technical-profile.md Section 7, ASMP-22)

**Candidate Approaches:** Relational storage on any Postgres option in the Database area (Neon, Supabase Postgres, Amazon RDS for PostgreSQL, Render Managed Postgres), accessed via Drizzle ORM, Prisma ORM or Kysely (ORM / Data Access); commit-time re-checks map to the "Database optimistic concurrency and transactions" technique (Real-time & Collaboration area), answering the profile's Collaboration/concurrency signal. Mutations can run in the Framework server layer, Hono or NestJS (Backend / API Layer); roster caching and invalidation fit TanStack Query v5 or SWR (State Management). The options differ mainly in how naturally row-version checks and transactions are expressed (Kysely/Drizzle are SQL-close; Prisma abstracts more).

**Risks & Unknowns:** Plan-limit check racing a subscription status event (Subscription Plan **Contention:**) needs the check and the client insert in one transaction to avoid over-limit clients. Client billing name/address is GDPR-class (feature-overview.md ## Non-Functional Notes, Data sensitivity) and must be removed on deletion except where attached invoices are retained (XBR-33).

**Spike Recommendation:** None

### FEAT-02 — Proposal Creation & Sending

**Verdict:** Straightforward — a versioned document lifecycle (draft → sent → voided/accepted) with void-and-resend on edit (FEAT-02.SPEC-006 ## Processing Logic; XBR-06) and validation rules (FEAT-02.SPEC-010); email reaches the client through the shared FEAT-14 capability rather than an integration owned here.

**Required Capabilities:**
- Proposal versioning with at most one active proposal per project and immutability once Sent/Voided/Accepted (XBR-04, XBR-06; feature-overview.md ## Non-Functional Notes, Data sensitivity)
- Send gated on a Primary contact existing (XBR-07) and on currency being set (XBR-17)
- Reuse picker over the freelancer's cross-project proposal history (FEAT-02.SPEC-004)
- Resend cooldown of 10 minutes (FEAT-02.SPEC-007; platform-parameters.md `proposal-resend-cooldown-minutes`)
- Email dispatch of the proposal link (FEAT-02.SPEC-011 ## Channels, ## Delivery Rules — Dedup, retry per `transactional-email-retry-count`)
- Concurrency: Nadia editing/voiding while Owen is viewing or accepting — reject-with-refresh; edit after acceptance refused (feature-dependency-map.md, Proposal **Contention:**)
- Offline/degraded: send never pretends to succeed offline (feature-overview.md ## Non-Functional Notes, Responsiveness; ASMP-27)
- Scale: one proposal per project plus voided versions accumulated over years; the reuse picker must stay responsive as history grows (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Version rows in any Database-area Postgres option with a status column and a partial unique constraint for "one active proposal per project", enforced through Drizzle ORM, Prisma ORM or Kysely; send/void-and-resend as a single transaction in the Framework server layer, Hono or NestJS. Reuse-picker filtering at this volume is satisfiable by plain indexed queries or PostgreSQL full-text search (Search area). Email delivery is shared with FEAT-14 (Resend, Postmark, Amazon SES, SendGrid — Email & Messaging Delivery).

**Risks & Unknowns:** Void-and-resend must atomically void the prior version and create the new one so Owen can never accept a voided version (Proposal **Contention:**). Proposal scope/price are commercially confidential and hidden from Reviewers entirely (feature-overview.md ## Non-Functional Notes) — authorization must be enforced server-side, not only in UI.

**Spike Recommendation:** None

### FEAT-03 — Proposal Acceptance

**Verdict:** Straightforward — exactly-once acceptance is a single atomic conditional update with write-time re-check (FEAT-03.SPEC-003 ## Processing Logic steps 2–5), and the deposit trigger reads the schedule as it stood at acceptance (XBR-01); both are standard transactional techniques.

**Required Capabilities:**
- Atomic accept write (`status`, `accepted_at`, `accepted_by`) with already-accepted and voided outcomes (FEAT-03.SPEC-003 ## Processing Logic)
- Deposit-invoice trigger to FEAT-09 using the schedule snapshot at acceptance (FEAT-03.SPEC-003 step 6; XBR-01)
- Change-request recording as a proposal comment that never alters the proposal (FEAT-03.SPEC-004; XBR-26)
- Primary-only access and client isolation (FEAT-03.SPEC-005; XBR-08, XBR-09)
- Concurrency: two Primary contacts accepting at once yield one acceptance; accept against a voided version refused (feature-dependency-map.md, Proposal **Contention:**)
- Offline/degraded: accept never pretends to succeed offline (ASMP-27; feature-overview.md ## Non-Functional Notes)
- Scale: N/A — no data-volume concern beyond Proposal/Comment growth (feature-overview.md ## Non-Functional Notes, Data volumes); the review screen must be interactive within ~2 s on mobile (ASMP-21)

**Candidate Approaches:** Conditional `UPDATE … WHERE status = 'Sent'` or row-version checks under the "Database optimistic concurrency and transactions" technique (Real-time & Collaboration), on any Database-area Postgres option via Drizzle ORM, Prisma ORM or Kysely. The deposit invoice can be created in the same transaction or dispatched as an idempotent follow-up job (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers — Background Jobs & Scheduling); the in-transaction route gives "appears immediately" semantics, the job route isolates invoice-generation failures. Mobile first paint within ~2 s is served by SSR options in Frontend Framework (Next.js, React Router v7, SvelteKit, Nuxt 4) plus CDN and framework cache (Caching & Performance).

**Risks & Unknowns:** If acceptance and deposit-invoice creation are split across a job boundary, a failed invoice generation leaves an accepted proposal without its deposit invoice — the retry/idempotency contract with FEAT-09.SPEC-004 must be explicit. Accepting contact identity is GDPR-class but must survive erasure as evidence (XBR-27; ASMP-20 legal basis still to be confirmed per FEAT-24 overview).

**Spike Recommendation:** None

### FEAT-04 — Milestone & Payment Schedule Setup

**Verdict:** Straightforward — single-project editor at human scale (feature-overview.md ## Non-Functional Notes, Data volumes) with validation/edit-lock rules (FEAT-04.SPEC-003) and a reorder recalculation (FEAT-04.SPEC-004); the offline queued save is one bounded mutation.

**Required Capabilities:**
- Milestone and payment-structure editing (deposit / per-milestone / completion / mix) with locks on approved or invoiced milestones (FEAT-04.SPEC-003; XBR-10)
- Dated, non-retroactive schedule adjustments; triggers read the schedule as of the triggering action (Payment Schedule **Contention:**)
- Client read-only timeline (FEAT-04.SPEC-002)
- Concurrency: Nadia's edits race Owen's approvals and acceptance triggers — reject-with-refresh; same-freelancer two-session edits reject-with-refresh (feature-dependency-map.md, Milestone and Payment Schedule **Contention:**)
- Offline/degraded: Save queues locally while offline and submits automatically on reconnect (FEAT-04.SPEC-001 Offline/Degraded state; FEAT-04.SPEC-001-AC-15)
- Scale: a handful of milestones per project; no scaling beyond a single project view (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Schedule snapshots or effective-dated rows in a Database-area Postgres option (Neon, Supabase Postgres, Amazon RDS for PostgreSQL, Render Managed Postgres) via Drizzle ORM, Prisma ORM or Kysely. The queued offline save fits TanStack Query v5's offline mutation persistence or a small Zustand store (State Management), both named in the landscape for offline queuing; row-version checks on replay use the "Database optimistic concurrency and transactions" technique (Real-time & Collaboration).

**Risks & Unknowns:** A queued offline save replayed after Owen approved a milestone must be rejected with refresh rather than silently re-pricing an approved milestone (XBR-10; Milestone **Contention:**) — the replay needs the loaded version token. Effective-dated schedule semantics ("as it stood at the moment of the triggering action") must be honored identically by FEAT-03, FEAT-08 and FEAT-01 triggers.

**Spike Recommendation:** None

### FEAT-05 — Client Portal Access (Magic-Link Login)

**Verdict:** Standard-with-integration — passwordless sign-in depends on an identity capability issuing single-use, time-limited tokens per Client Contact record (FEAT-05.SPEC-004 ## Processing Logic; FEAT-05.SPEC-006; XBR-28) and on the transactional email capability delivering the link (FEAT-05.SPEC-008); the work is integration, not invention.

**Required Capabilities:**
- Token issuance per matching Client Contact (one per freelancer for a shared email), invalidating earlier unused tokens, neutral no-match outcome (FEAT-05.SPEC-004 ## Processing Logic steps 2–6)
- Verification landing with expired/used-link recovery (FEAT-05.SPEC-002, FEAT-05.SPEC-005); 24-hour expiry (platform-parameters.md `magic-link-expiry-window`)
- Strict per-company, per-freelancer isolation (FEAT-05.SPEC-007; XBR-09) and first-view capture into the trail (FEAT-05.SPEC-009)
- Concurrency: re-requests invalidate prior unused links (XBR-28); no shared-record contention beyond the `last_sign_in` overwrite (feature-overview.md ## Non-Functional Notes, Data volumes)
- Offline/degraded: email delay/failure degrades sign-in; recovery is one additional request (feature-overview.md ## Non-Functional Notes, Responsiveness — ≥95% first-try success)
- Scale: bounded by a few thousand freelancers × 3–15 clients × a handful of contacts (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** Authentication & Identity area options: Clerk or WorkOS AuthKit (managed, hosted user store), Supabase Auth (binds to Supabase Postgres and row-level security), or Better Auth (library, sessions in the application database). The per-freelancer-contact token model (one person, several separate portal identities) differs from a typical one-user-one-identity model — Better Auth or a thin custom token table keeps that mapping in the application's own schema, while managed providers need the contact-per-freelancer identity mapped onto their user model. Link delivery rides the Email & Messaging Delivery options (Resend, Postmark, Amazon SES, SendGrid).

**Risks & Unknowns:** Email-client link prefetchers can consume single-use links before the contact clicks (FEAT-05.SPEC-006 single-use rule) — an interstitial confirm step may be needed to hit the 95% first-try metric. Managed identity providers hold contact emails (GDPR-class, feature-overview.md ## Non-Functional Notes) in a vendor store — a processing consideration. Deliverability of the sign-in email directly gates access (ASMP-29).

**Spike Recommendation:** None

### FEAT-06 — Deliverable Upload & Sharing

**Verdict:** Standard-with-integration — resumable ingestion and delivery run through the large-file storage capability (FEAT-06.SPEC-003; FEAT-16.SPEC-007 ## Degradation Behavior rows for FEAT-06.SPEC-001/002), and linked assets require outbound reachability checks against third-party hosts (FEAT-06.SPEC-004 ## Processing Logic step 3).

**Required Capabilities:**
- File upload up to the 2 GB ceiling with real progress, pause and auto-resume (FEAT-06.SPEC-003; platform-parameters.md `deliverable-file-size-ceiling`; XBR-12)
- Link attach with server-side reachability check; flagged links never shown to clients (FEAT-06.SPEC-004 ## Processing Logic steps 3–5)
- Deliverable-ready email only after full completion (FEAT-06.SPEC-006; XBR-12)
- Removal eligibility re-checked against approval state (FEAT-06.SPEC-005; XBR-11)
- Concurrency: removal/replacement re-checked at commit and rejected if the milestone was approved meanwhile (feature-dependency-map.md, Deliverable **Contention:**)
- Offline/degraded: dropped connection pauses and resumes; capability-down shows Upload Failed with the file still selected (FEAT-16.SPEC-007 ## Degradation Behavior)
- Scale: files typically tens of MB, sometimes >1 GB, retained for the account's life (feature-overview.md ## Non-Functional Notes; ASMP-22)

**Candidate Approaches:** File & Object Storage options — Cloudflare R2, Amazon S3, Backblaze B2 (S3-compatible multipart upload with presigned URLs) or Supabase Storage (resumable TUS uploads, tied to Supabase). Direct-to-storage browser uploads keep large bytes off the Backend / API Layer (Framework server layer, Hono, NestJS), which only issues upload sessions and records completion. The reachability check is a short outbound HTTP call that fits inline in the backend or as a queued job (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers — Background Jobs & Scheduling).

**Risks & Unknowns:** Figma, Google Drive and Dropbox share links frequently return a successful response that is actually a sign-in page, so "reachable" may be misjudged (FEAT-06.SPEC-004 step 3 — "not requiring credentials the product does not hold"); the landscape has no option covering provider-specific link introspection — recorded as an open question. Operator must list but never download files (XBR-29), so download URLs must be authorization-scoped per viewer. Deliverables may contain personal data (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-07 — Deliverable Review & Feedback

**Verdict:** Straightforward — append-only threads with no concurrent modification (Comment **Contention:** None), a 5-minute edit window and retraction rule (FEAT-07.SPEC-006) and role-scoped visibility (FEAT-07.SPEC-007); the offline queue (FEAT-07.SPEC-008) is a bounded client-side replay of one record type.

**Required Capabilities:**
- Deliverable-version-pinned and milestone-level threads (FEAT-07.SPEC-001, FEAT-07.SPEC-002; XBR-13)
- Device-local offline queue replayed in order on reconnect, re-running validation and authorization, `posted_at` set at sync time (FEAT-07.SPEC-008 ## Processing Logic steps 1–6)
- Per-comment emails to freelancer and client, no batching, dedup one per comment (FEAT-07.SPEC-003, FEAT-07.SPEC-004 ## Delivery Rules)
- Concurrency: many authors append concurrently; ordering by posted time only (feature-dependency-map.md, Comment **Contention:** None)
- Offline/degraded: queued-for-send state never shown as posted (FEAT-07.SPEC-008 step 2; feature-overview.md ## Non-Functional Notes, Compliance flags); file stream unavailable leaves thread usable (FEAT-16.SPEC-007 ## Degradation Behavior, FEAT-07.SPEC-001 row)
- Scale: uncapped thread length must stay responsive; review page interactive within ~2 s on mobile (feature-overview.md ## Non-Functional Notes; ASMP-21)

**Candidate Approaches:** Queue persistence fits TanStack Query v5 offline mutation persistence or a Zustand store (State Management — both named for the offline comment queue), answering the profile's Offline signal; server writes on any Database-area Postgres option via Drizzle ORM, Prisma ORM or Kysely. Threads are snapshot screens (Real-time = No), so live push (Supabase Realtime, Ably, Pusher Channels) is optional rather than demanded. Mobile interactivity is served by the SSR Frontend Framework options and CDN and framework cache (Caching & Performance).

**Risks & Unknowns:** Device-local queues are lost if the browser storage is cleared before reconnect; the spec's queued indicator mitigates misperception but not loss. A queued comment replayed after the author's role was removed must fail authorization (FEAT-07.SPEC-008 step 5). Comment text and authorship are GDPR-class and included in export/removed on deletion (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-08 — Milestone Approval

**Verdict:** Straightforward — the approval guard compares live state with the state shown and writes exactly once (FEAT-08.SPEC-003 ## Processing Logic steps 4–6); the next-invoice trigger (FEAT-08.SPEC-004) and logged reopen (FEAT-08.SPEC-005) are standard state transitions.

**Required Capabilities:**
- Stale-attempt detection on approve (status, deliverable, price changed) and exactly-once approval (FEAT-08.SPEC-003)
- Next-invoice trigger per schedule as it stood at approval (FEAT-08.SPEC-004; XBR-02)
- Freelancer-only reopen as a new logged event (FEAT-08.SPEC-005; XBR-04, XBR-10)
- Concurrency: Nadia re-pricing/removing vs Owen approving — reject-with-refresh; approval exactly-once (feature-dependency-map.md, Milestone **Contention:**)
- Offline/degraded: approve refuses without connectivity/session and never pretends to succeed (FEAT-08.SPEC-003 step 2; ASMP-27)
- Scale: single-digit milestones per project — no scale concern (feature-overview.md ## Non-Functional Notes); approval screen interactive within ~2 s on mobile (ASMP-21)

**Candidate Approaches:** Row-version or state-hash comparison under "Database optimistic concurrency and transactions" (Real-time & Collaboration) on a Database-area Postgres option via Drizzle ORM, Prisma ORM or Kysely; invoice creation either in-transaction or as an idempotent job on Inngest, Trigger.dev v3, Upstash QStash or pg-boss / BullMQ workers (Background Jobs & Scheduling), with the same trade-off described under FEAT-03.

**Risks & Unknowns:** The "state shown" token must cover the deliverable set and price, not only milestone status (FEAT-08.SPEC-003 step 5), or a re-priced milestone could be approved. Approver identity is evidentiary personal data retained after erasure (feature-overview.md ## Non-Functional Notes, Compliance flags; XBR-27).

**Spike Recommendation:** None

### FEAT-09 — Invoice Generation & Sending

**Verdict:** Hard — three demands combine: gapless per-freelancer sequential numbering across automatic and manual creation paths where a conflict is a system-level retry (FEAT-09.SPEC-007 "never reused, never skipped"); generation fired atomically by other features' actions with schedule-as-of semantics and distinct blocked outcomes (FEAT-09.SPEC-004 ## Trigger Definition, ## Processing Logic step 3; XBR-01..03, XBR-16, XBR-17); and evidentiary immutability with correction only via credit notes (FEAT-09.SPEC-008; ASMP-25).

**Required Capabilities:**
- Automatic generation from deposit, milestone-approval and completion triggers; manual invoice and credit-note issuance (FEAT-09.SPEC-004, FEAT-09.SPEC-005, FEAT-09.SPEC-003)
- Gapless sequential numbering per freelancer (FEAT-09.SPEC-007)
- Invoice content compliance: both parties' details, issue/due dates, tax line (feature-overview.md ## Non-Functional Notes, Compliance flags; ASMP-24)
- Pay-link availability derived from payment-connection status with no-account fallback instructions (FEAT-09.SPEC-009; XBR-19)
- Printable/downloadable copy of invoice, credit note and receipt (FEAT-09.SPEC-002 Access table)
- Invoice-issued email to Primary contacts with copy confirmation (FEAT-09.SPEC-010 ## Delivery Rules)
- Concurrency: several writers on one invoice (payment, manual record, refund, reminders, processor reports), resolved reject-with-refresh against current status (feature-dependency-map.md, Invoice **Contention:**); concurrent creation paths contend on the number sequence
- Offline/degraded: generation is server-side and near-instant (feature-overview.md ## Non-Functional Notes, Responsiveness); when payments are not connected, invoices still issue with direct-payment instructions (XBR-19)
- Scale: a handful of invoices per project; retained for the account's life and up to 7 years after deletion (feature-overview.md ## Non-Functional Notes; platform-parameters.md `financial-record-legal-retention-period`)

**Candidate Approaches:** Gapless numbering can be implemented as a per-freelancer counter row locked inside the invoice-creating transaction, or as a unique `(freelancer, number)` constraint with retry-on-conflict — both expressible on any Database-area Postgres option (Neon, Supabase Postgres, Amazon RDS for PostgreSQL, Render Managed Postgres) via Drizzle ORM, Prisma ORM or Kysely, under the "Database optimistic concurrency and transactions" technique (Real-time & Collaboration). Immutability can be enforced in the application layer only or additionally by database-level guards (Postgres permissions/triggers — standard Postgres behavior common to all Database options). Trigger delivery from FEAT-01/03/08 can be synchronous in the originating transaction or via durable jobs (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers); pg-boss keeps the enqueue in the same Postgres transaction as the triggering write, while the managed options need an outbox-style hand-off to get the same atomicity (Cross-Area Compatibility Notes, Background Jobs ↔ Database).

**Risks & Unknowns:** Sequence gaps from rolled-back transactions (native Postgres sequences skip values) would violate "never skipped" — native sequences alone do not meet the spec. Splitting trigger and generation across a job boundary risks duplicate or missing invoices without an idempotency key per triggering event. Printable invoice/receipt copies need a document-rendering capability that no landscape area covers (gap — Section 5). Legal retention length is "to be confirmed with legal counsel" (platform-parameters.md).

**Spike Recommendation:** None

### FEAT-10 — Invoice Payment Processing

**Verdict:** Hard — beyond wiring a processor, the invoice's paid state is written by four actors while processor events arrive duplicated, out of order and per-attempt, and processor-confirmed status must override a conflicting manual record with a discrepancy notice (FEAT-10.SPEC-003 ## Edge Cases; FEAT-10.SPEC-004; feature-dependency-map.md, Invoice and Payment **Contention:**; XBR-20, XBR-22).

**Required Capabilities:**
- Card and bank-transfer payment into the freelancer's own processor account; platform never touches card data (FEAT-10.SPEC-003 ## Capability Category; ASMP-24, ASMP-28)
- Payment-attempt-level event correlation, idempotent duplicate handling, pending bank transfers (FEAT-10.SPEC-003 ## Edge Cases; FEAT-10.SPEC-004)
- Off-platform manual payment recording: full amount, not future-dated (FEAT-10.SPEC-005, FEAT-10.SPEC-006; XBR-20)
- Payment confirmation email (FEAT-10.SPEC-007 ## Delivery Rules — dedup)
- Concurrency: Owen paying vs Nadia recording manually; first confirmed full payment wins; processor reports applied in order and authoritative (Payment **Contention:**)
- Offline/degraded: slow → "Still processing" after 10 s with no second submission; down → pay disabled, rest of invoice usable; rejects → decline reason and retry, no ambiguous Payment (FEAT-10.SPEC-003 ## Degradation Behavior)
- Scale: at most one successful Payment per invoice; ordinary invoice volume (feature-overview.md ## Non-Functional Notes); pay screen interactive within ~2 s on mobile (ASMP-21)

**Candidate Approaches:** Payments & Billing area: Stripe (Connect Standard with direct charges) places funds and fees on the connected account and emits webhooks for status and disputes; Adyen for Platforms (sales-led, interchange++ with minimums), PayPal Commerce Platform and Mollie Connect (European local methods) are the alternatives; they differ in onboarding effort, bank-transfer coverage by region and minimum commitments relative to the worldwide-from-day-one geography (technical-profile.md Section 7). Webhook ingestion lands in the Backend / API Layer (Framework server layer, Hono, NestJS) with durable processing on Background Jobs & Scheduling options (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers); the per-attempt state machine uses "Database optimistic concurrency and transactions" (Real-time & Collaboration).

**Risks & Unknowns:** Processor-authoritative override of a manual "Paid" can produce a double payment that the freelancer must refund in her own processor account (FEAT-10.SPEC-003 ## Edge Cases, third bullet) — the discrepancy notice is the only safeguard. Bank-transfer method availability varies by country and processor; worldwide coverage for "bank transfer" is not established by the landscape. Webhook signature verification and replay handling are security-critical. Payment records are GDPR-class and retained under legal retention (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-11 — Automated Payment Reminders

**Verdict:** Straightforward — schedule-based evaluation at day 3 and day 10 in the freelancer's time zone, with a send-time eligibility re-check and one-entry-per-threshold idempotency (FEAT-11.SPEC-001 ## Trigger Definition, ## Processing Logic steps 2–5; FEAT-11.SPEC-002) — a well-trodden scheduled-job pattern on shared job machinery.

**Required Capabilities:**
- Periodic evaluation of every overdue-capable invoice with time-zone-aware day counting (FEAT-11.SPEC-001; FEAT-15.SPEC-006; XBR-15)
- Pause per invoice, bank-transfer-pending pause, manual reminder limited to one per invoice per day (FEAT-11.SPEC-003, FEAT-11.SPEC-005; XBR-15)
- One email per Reminder Log entry, no batching, no quiet hours (FEAT-11.SPEC-004 ## Delivery Rules)
- Concurrency: schedule vs Nadia's pause/resume/manual send — re-check immediately before send; second concurrent manual send refused (feature-dependency-map.md, Reminder Log **Contention:**)
- Offline/degraded: failed sends retried and surfaced, not lost (feature-overview.md ## Non-Functional Notes, Compliance flags; ASMP-26)
- Scale: at most two automatic entries per overdue invoice; small per freelancer (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Background Jobs & Scheduling options: Inngest or Trigger.dev v3 (managed cron plus durable steps), Upstash QStash (scheduled HTTP calls), or pg-boss / BullMQ workers (open-source queues with cron; pg-boss requires Postgres, BullMQ requires Redis such as Upstash Redis or Redis Cloud from Caching & Performance), answering the profile's Background processing signal. Hosting-native cron (Vercel, Render cron jobs — Hosting & Environments) is a lighter trigger for a periodic sweep. Time-zone arithmetic fits Native Intl APIs (Internationalization) or equivalent database time-zone functions.

**Risks & Unknowns:** DST transitions and sweep cadence can make "exactly 3 elapsed days" miss or double-fire if evaluation is not idempotent on the (invoice, threshold) pair (FEAT-11.SPEC-001 "only if no day 3 Reminder Log entry already exists"). A missed sweep window (host cron outage) must catch up rather than skip, since triggers fire on "exactly 3".

**Spike Recommendation:** None

### FEAT-12 — Freelancer Financial Dashboard

**Verdict:** Straightforward — totals are recomputed per affected scope and currency on invoice/payment events (FEAT-12.SPEC-004 ## Trigger Definition, ## Processing Logic) over a small per-account dataset, never converting currencies (FEAT-12.SPEC-003; XBR-18).

**Required Capabilities:**
- Earned/outstanding/overdue per currency, account-wide and per client/project drill-down (FEAT-12.SPEC-001..003; XBR-18, XBR-22)
- Event-triggered recomputation with retries (3 × 2 s) and a Retry control on error (FEAT-12.SPEC-004; platform-parameters.md `dashboard-aggregation-retry-count`)
- Concurrency: N/A — read-only aggregation; recomputes read current records at run time (FEAT-12.SPEC-004 step 2)
- Offline/degraded: shows last loaded totals with an error/retry state (technical-profile.md Section 3, Offline row — FEAT-12.SPEC-001 "showing your last loaded totals")
- Scale: 3–15 active clients with unlimited history; totals within ~1–2 s (feature-overview.md ## Non-Functional Notes; ASMP-21)

**Candidate Approaches:** On-demand SQL aggregation over indexed Invoice/Payment rows (Kysely named for window functions and CTEs; Drizzle ORM or Prisma ORM also viable — ORM / Data Access), or held totals via "Database-level caching (materialized views / read replicas)" or a cached aggregate in Upstash Redis / Redis Cloud (Caching & Performance), answering the profile's Scale hints signal. The held-totals approach matches FEAT-12.SPEC-004's "currently held Financial Totals" model; on-demand aggregation removes the refresh job at this data size.

**Risks & Unknowns:** Held totals drift if any invoice/payment mutation path fails to emit the refresh trigger (FEAT-12.SPEC-004 lists triggers from FEAT-09, FEAT-10, FEAT-11, FEAT-25). Overdue status is time-driven, so totals must reflect time passing even without a write event.

**Spike Recommendation:** None

### FEAT-13 — Immutable Activity & Audit Trail

**Verdict:** Straightforward — append-only entries with a fixed event vocabulary and duplicate-report guard (FEAT-13.SPEC-003 ## Processing Logic steps 2–3), immutability and attribution rules (FEAT-13.SPEC-004) and role-scoped visibility (FEAT-13.SPEC-005) are established audit-log patterns.

**Required Capabilities:**
- Append-only recording from ~15 event sources with unlimited write retry (FEAT-13.SPEC-003; platform-parameters.md `activity-entry-write-retry-interval`; XBR-05)
- No edit/delete of entries (FEAT-13.SPEC-004; XBR-04; ASMP-25)
- Printable, unalterable copy of a trail or an entry (FEAT-13.SPEC-002)
- Retention and account-deletion purge subject to legal financial-record retention (FEAT-13.SPEC-006; SC-24)
- Concurrency: none — concurrent writers only append independent entries (feature-dependency-map.md, Activity Log Entry **Contention:** None)
- Offline/degraded: entry writes retried until success so no event is lost (FEAT-13.SPEC-003; `activity-entry-write-retry-interval`); trail shows a loading indicator for long histories (feature-overview.md ## Non-Functional Notes)
- Scale: hundreds of entries per long-running project; no depth limit (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** An insert-only table on any Database-area Postgres option, with immutability enforced in the data-access layer (Drizzle ORM, Prisma ORM, Kysely) and optionally by database privileges/triggers; ingestion inline with the triggering transaction or via durable jobs (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers — Background Jobs & Scheduling) for the retry-until-success rule. Operational logs remain separate in Observability & Operations options (Sentry, Grafana Cloud, Datadog, Axiom); these are not substitutes for the evidentiary trail.

**Risks & Unknowns:** "Unalterable" printable copy implies a rendered document (FEAT-13.SPEC-002); no landscape area covers document/PDF rendering (gap — Section 5). Erasure-versus-evidence (names kept on entries after a contact's erasure, XBR-27) has an unconfirmed legal basis (ASMP-20 per FEAT-24 feature-overview.md ## Non-Functional Notes). If entries are written by a job outside the originating transaction, a crash between event and enqueue can lose an entry unless an outbox is used.

**Spike Recommendation:** None

### FEAT-14 — Notifications (Email)

**Verdict:** Standard-with-integration — a transactional email service with delivery/bounce webhooks is the feature's core (FEAT-14.SPEC-001 ## Capability Category), wrapped in idempotent, event-time-ordered status handling and a bounded retry policy (FEAT-14.SPEC-001 ## Edge Cases; FEAT-14.SPEC-003 ## Processing Logic).

**Required Capabilities:**
- Composition and dispatch for 26 notification types with recipient entitlement by role (FEAT-14.SPEC-002, FEAT-14.SPEC-004; XBR-08, XBR-30)
- Branded, recognizable presentation with freelancer logo/colour and the referral mark (FEAT-14.SPEC-005; XBR-31, XBR-32)
- Delivery-status tracking: bounce never retried; transient failures retried up to 3 times within 6 hours; warning to freelancer on exhaustion (FEAT-14.SPEC-003; FEAT-14.SPEC-006; platform-parameters.md `transactional-email-retry-count`, `transactional-email-retry-window`)
- Concurrency: none on Notification (feature-dependency-map.md, Notification **Contention:** None); duplicate and out-of-order provider events resolved by event time (FEAT-14.SPEC-001 ## Edge Cases)
- Offline/degraded: no screen waits on this capability; outages leave Notifications Queued and retried (FEAT-14.SPEC-001 ## Degradation Behavior)
- Scale: volume scales with total product activity across a few thousand freelancers (feature-overview.md ## Non-Functional Notes, Data volumes); failures surfaced within minutes (ASMP-26)

**Candidate Approaches:** Email & Messaging Delivery options: Resend (developer API, delivery webhooks, React email templates), Postmark (deliverability focus, message streams, bounce webhooks), Amazon SES (lowest cost, events via SNS, more setup) or SendGrid (Twilio). Queued dispatch and scheduled retries fit Background Jobs & Scheduling options (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers). Template theming for brand colours can share tokens with CSS / Styling options. The options differ chiefly in deliverability reputation tooling, webhook event richness and cost at volume.

**Risks & Unknowns:** ASMP-26 asks for failures surfaced "within minutes", yet the transient-failure retry window is 6 hours before the warning fires (FEAT-14.SPEC-003 step 6; `transactional-email-retry-window`) — the guarantee holds only for bounces (Section 5). Per-freelancer branded "from" identity and deliverability across thousands of freelancers on a shared sending domain is a reputation risk (feature-overview.md Rationale: spam complaints across competitors). Recipient data in the provider is GDPR-class (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-15 — Currency & Tax Handling

**Verdict:** Straightforward — currency, tax label/rate and time zone are small configuration values with no automatic jurisdictional calculation (feature-overview.md ## Non-Functional Notes, Compliance flags; SC-16), locked after the first invoice (FEAT-15.SPEC-004) and rendered per viewer (FEAT-15.SPEC-006).

**Required Capabilities:**
- Per-project currency and tax line configuration with validation (FEAT-15.SPEC-001, FEAT-15.SPEC-003; XBR-17)
- Lock after first invoice and access rules (FEAT-15.SPEC-004, FEAT-15.SPEC-005)
- Freelancer time zone; viewer-local date/time display; reminder day counts in freelancer time zone (FEAT-15.SPEC-002, FEAT-15.SPEC-006; XBR-15)
- Tax line applied to invoices; multi-currency non-aggregation (FEAT-15.SPEC-008, FEAT-15.SPEC-007; XBR-18)
- Concurrency: currency edit racing first-invoice issuance must be refused once the invoice is sent (FEAT-15.SPEC-004); otherwise last-write-wins on client fields (Client **Contention:**)
- Offline/degraded: N/A — configuration is local and requires connectivity to persist (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: N/A — a fixed small set of fields per project and one time zone per account (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Formatting via Native Intl APIs (Intl.NumberFormat, Intl.DateTimeFormat), or through next-intl, i18next / react-i18next or Lingui (Internationalization) if future locales are anticipated; English-only launch means the translation layer is optional. Money amounts stored as integer minor units or Postgres numeric in any Database-area option via Drizzle ORM, Prisma ORM or Kysely.

**Risks & Unknowns:** Currencies with non-2-decimal minor units (0 or 3 decimals) must be handled in rounding of tax lines and in payment-processor amounts (FEAT-15.SPEC-008 ↔ FEAT-10.SPEC-003). The product makes no tax-correctness claim (SC-16); that boundary must be visible in UI copy.

**Spike Recommendation:** None

### FEAT-16 — Large File Handling & Storage

**Verdict:** Standard-with-integration — chunked resumable ingestion, byte-range delivery and deletion purge run against an external object-storage capability (FEAT-16.SPEC-007 ## Capability Category, ## Degradation Behavior; FEAT-16.SPEC-002 ## Processing Logic); the patterns are established, the work is integration plus cost control.

**Required Capabilities:**
- Chunked transfer with persisted resume point, progress and ETA, auto-resume on reconnect (FEAT-16.SPEC-002 ## Processing Logic steps 3–6)
- Reliable streaming/download delivery (FEAT-16.SPEC-003)
- Per-file 2 GB ceiling; storage allowance 5 GB free / 100 GB paid with 80% warning (FEAT-16.SPEC-004, FEAT-16.SPEC-005; platform-parameters.md)
- Stored-file purge on account deletion with idempotent confirmations (FEAT-16.SPEC-006; FEAT-16.SPEC-007 ## Edge Cases)
- Concurrency: duplicate/out-of-order transfer events ignored once finalized (FEAT-16.SPEC-007 ## Edge Cases); allowance check against last aggregated total (FEAT-16.SPEC-002 step 2)
- Offline/degraded: slow → progress with lengthening estimate; down → "Uploads aren't available right now" with file kept; no half-created versions (FEAT-16.SPEC-007 ## Degradation Behavior)
- Scale: ≥98% of uploads incl. >500 MB complete without restart; stored bytes grow for the account's life within ~$100/month infrastructure (feature-overview.md ## Non-Functional Notes; BRIEF.md ## Constraints)

**Candidate Approaches:** File & Object Storage options: Cloudflare R2 ($0.015/GB-month, zero egress — suits large downloads), Backblaze B2 ($0.00695/GB-month, free egress up to 3× stored or via Cloudflare), Amazon S3 (deepest tooling, egress billed) and Supabase Storage (TUS resumable, egress against plan quota). R2, B2 and S3 share the S3 multipart API (Cross-Area Compatibility Notes), so one client library covers them. Usage aggregation (FEAT-16.SPEC-005) and purge fit Background Jobs & Scheduling options (Inngest, Trigger.dev v3, Upstash QStash, pg-boss / BullMQ workers).

**Risks & Unknowns:** Budget arithmetic is tight: 1,000 paid freelancers averaging 20 GB is ~20 TB, about $300/month at R2 list price and ~$140/month on B2 before requests — above the ~$100/month infrastructure constraint (BRIEF.md ## Constraints; landscape File & Object Storage pricing); the allowance parameters (`paid-tier-storage-allowance` 100 GB) and the budget cannot both hold at scale without plan revenue offsetting cost (Section 5). Resume across a browser reload (not just a connectivity drop) requires client-side persistence of multipart upload IDs. Egress on non-zero-egress options scales with client streaming of video.

**Spike Recommendation:** None

### FEAT-17 — Deliverable Version History

**Verdict:** Straightforward — each re-upload appends an immutable version that no one edits concurrently (Deliverable Version **Contention:** None; FEAT-17.SPEC-003, FEAT-17.SPEC-004), with storage delegated to FEAT-16 (FEAT-17.SPEC-003 → FEAT-16.SPEC-007; XBR-13).

**Required Capabilities:**
- New version upload creating the next round number; prior version untouched on failure (FEAT-17.SPEC-001, FEAT-17.SPEC-003; FEAT-16.SPEC-007 ## Degradation Behavior FEAT-17.SPEC-001 row)
- Version browser opening any round alongside the latest (FEAT-17.SPEC-002)
- Comment anchoring per version and access rules (FEAT-17.SPEC-005; XBR-13)
- Concurrency: none — versions are append-only and created only by Nadia (feature-dependency-map.md, Deliverable Version **Contention:** None)
- Offline/degraded: upload interruption behaves as FEAT-16; switching versions shows a visible loading indicator (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: versions retained uncapped for the account's life and are the main driver of year-one storage volume (feature-overview.md ## Non-Functional Notes, Data volumes); client-facing browser within ~2 s on mobile (ASMP-21)

**Candidate Approaches:** Version metadata rows on any Database-area Postgres option; bytes via the File & Object Storage options named under FEAT-16 (Cloudflare R2, Amazon S3, Backblaze B2, Supabase Storage), with S3-style object versioning or distinct object keys per round. Side-by-side viewing on mobile is a Frontend Framework concern (Next.js, React Router v7, SvelteKit, Nuxt 4).

**Risks & Unknowns:** Uncapped retention is the dominant storage-cost driver (feature-overview.md ## Non-Functional Notes) — ties directly to the FEAT-16 budget risk. Opening two large video versions side by side on mobile may exceed the ~2 s target on typical connections; the spec does not define preview renditions, and no landscape area covers media transcoding/thumbnailing (Section 5).

**Spike Recommendation:** None

### FEAT-18 — Client Contact Management & Roles

**Verdict:** Straightforward — contact CRUD with Primary/Reviewer roles (FEAT-18.SPEC-007), forward-only role changes (FEAT-18.SPEC-008), commit-time last-Primary check (FEAT-18.SPEC-006) and erasure that preserves evidence (FEAT-18.SPEC-009); no external service beyond shared email.

**Required Capabilities:**
- Add/edit/remove contacts; client-side colleague invitation by Primary contacts (FEAT-18.SPEC-002..004)
- Role authorization consumed product-wide (FEAT-18.SPEC-007; XBR-08)
- Erasure that ends access immediately and removes details while names remain on evidence (FEAT-18.SPEC-009; XBR-27)
- Invitation and primary-invited-colleague emails (FEAT-18.SPEC-010, FEAT-18.SPEC-011)
- Concurrency: Nadia and Owen adding the same email concurrently — unique-per-client reject-with-refresh; last-Primary removal re-checked at commit (feature-dependency-map.md, Client Contact **Contention:**)
- Offline/degraded: N/A — freelancer-side forms require connectivity (ASMP-27); SPEC-004 is client-facing and follows the ~2 s mobile target (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: a handful of contacts per client; no special scaling (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Unique `(client, email)` constraint and transactional last-Primary check under "Database optimistic concurrency and transactions" (Real-time & Collaboration) on any Database-area Postgres option via Drizzle ORM, Prisma ORM or Kysely. Session revocation on removal depends on the Authentication & Identity choice (Clerk, Better Auth, Supabase Auth, WorkOS AuthKit) — library-held sessions (Better Auth) are revoked by row deletion, managed providers via their session APIs.

**Risks & Unknowns:** "Ends access immediately" (XBR-27) requires revoking live portal sessions and outstanding magic links, not only the contact row. Keeping the name on evidence records while erasing contact details requires denormalizing the display name onto evidentiary records (FEAT-03, FEAT-08, FEAT-13) — the legal basis is flagged for confirmation (ASMP-20).

**Spike Recommendation:** None

### FEAT-19 — Freelancer Branding

**Verdict:** Straightforward — one logo (≤2 MB) and one colour per account with contrast-ratio adjustment and neutral fallback (FEAT-19.SPEC-002, FEAT-19.SPEC-003; platform-parameters.md `branding-logo-file-size-ceiling`, `branding-color-legibility-contrast-ratio`).

**Required Capabilities:**
- Logo upload and validation; colour selection with automatic legibility adjustment (FEAT-19.SPEC-001, FEAT-19.SPEC-002)
- Application to every client-facing screen and email with fallback and referral mark alongside (FEAT-19.SPEC-003; XBR-31)
- Concurrency: last-write-wins across Nadia's sessions (feature-dependency-map.md, Branding Profile **Contention:** None)
- Offline/degraded: N/A — small configuration form, no loading state (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: one profile per account; logo size limited to keep client pages fast on mobile (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Runtime theme tokens through CSS custom properties — Tailwind CSS v4 (CSS-variable theme tokens mapped per freelancer), CSS Modules, vanilla-extract or Panda CSS (CSS / Styling); logo stored on a File & Object Storage option (Cloudflare R2, Amazon S3, Backblaze B2, Supabase Storage) and served through CDN and framework cache (Caching & Performance). Email branding shares the templates of the Email & Messaging Delivery options.

**Risks & Unknowns:** Email clients handle CSS variables and SVG logos inconsistently, so branded emails need inline colours and raster logos. Build-time CSS options (vanilla-extract, Panda CSS) still need a runtime variable layer for per-freelancer colours.

**Spike Recommendation:** None

### FEAT-20 — Onboarding / First-Run Setup

**Verdict:** Straightforward — a guided multi-step sequence with completion detection and exit criteria (FEAT-20.SPEC-002, FEAT-20.SPEC-003, FEAT-20.SPEC-005) over sign-up served by the shared identity capability (FEAT-20.SPEC-001) and a welcome email (FEAT-20.SPEC-006).

**Required Capabilities:**
- Sign-up and account creation (FEAT-20.SPEC-001) with free-plan auto-provisioning hand-off (FEAT-23.SPEC-002)
- Guided sequence: first client, branding, first proposal, optional payment connection (FEAT-20.SPEC-002; FEAT-32.SPEC-002 hand-off)
- Referral attribution capture hand-off and "how did you hear" answer (FEAT-20.SPEC-004; XBR-32)
- Concurrency: N/A — single user, runs once per account (feature-overview.md ## Non-Functional Notes, Data volumes)
- Offline/degraded: N/A — no offline mandate; welcome-email failure surfaced per ASMP-26 (feature-overview.md ## Non-Functional Notes, Compliance flags)
- Scale: runs once per new account; a few thousand in year one; median sign-up-to-draft under 15 minutes (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Freelancer sign-up via Authentication & Identity options (Clerk hosted components, Better Auth, Supabase Auth, WorkOS AuthKit); step state persisted on the account in any Database-area option so a resumed session continues; product funnels measured with Analytics & Product Telemetry options (PostHog, Mixpanel, Amplitude, Plausible) against the First-Session Activation metric.

**Risks & Unknowns:** Sign-up, free-plan provisioning and referral recording span several features (FEAT-20, FEAT-23, FEAT-33); partial failure must not leave an account without a plan record. Analytics tooling ingests GDPR-class identifiers unless configured otherwise (ASMP-24).

**Spike Recommendation:** None

### FEAT-21 — Settings & Account Management

**Verdict:** Straightforward — standard configuration forms (FEAT-21.SPEC-001, FEAT-21.SPEC-002, FEAT-21.SPEC-004) with validation and completeness gates (FEAT-21.SPEC-007, FEAT-21.SPEC-009), plus session management and re-verified email change (FEAT-21.SPEC-005, FEAT-21.SPEC-006) served by the shared identity capability.

**Required Capabilities:**
- Profile, business details (name, address, tax ID) and default payment terms feeding invoices (FEAT-21.SPEC-004; XBR-16)
- Notification preferences limited to optional emails (FEAT-21.SPEC-002, FEAT-21.SPEC-008; XBR-30)
- Sign-in email change pending re-verification for 24 hours; sign-out of other sessions (FEAT-21.SPEC-005, FEAT-21.SPEC-006; platform-parameters.md `email-change-reverification-window`)
- Operator read-only scope that never exposes credentials (FEAT-21.SPEC-010)
- Concurrency: last-write-wins per field across Nadia's sessions, except sign-in email change (feature-dependency-map.md, Freelancer Account **Contention:**)
- Offline/degraded: N/A — settings require connectivity to persist; failed save keeps unsaved fields (feature-overview.md ## Non-Functional Notes, Compliance flags and Responsiveness)
- Scale: one account record per freelancer (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Session listing and revocation from the Authentication & Identity area — Clerk (multi-session management built in), Better Auth (sessions in the application database), Supabase Auth or WorkOS AuthKit; forms and optimistic updates with TanStack Query v5 or SWR (State Management); data in any Database-area option.

**Risks & Unknowns:** Email change must also update the identity provider's record atomically with the account record; an expired re-verification must revert cleanly (FEAT-21.SPEC-005). Tax ID and address are GDPR-class (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-22 — Accounting Export

**Verdict:** Straightforward — a bounded date-range read of invoices and related payments, grouped per currency and written to CSV or a QuickBooks/Xero-compatible file (FEAT-22.SPEC-002 ## Processing Logic steps 3–7; FEAT-22.SPEC-003), with no live sync (technical-profile.md Section 7, BRIEF.md ## Ecosystem & Integrations).

**Required Capabilities:**
- File generation with per-currency totals including refunds, reversals and manual payments (FEAT-22.SPEC-002; XBR-18, XBR-22)
- Authoritative range/authorization re-check at processing time; operator excluded (FEAT-22.SPEC-003; XBR-29)
- Retry without corrupt partial files (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Concurrency: N/A — read-only over already-stored records; stale requests refused at processing time (FEAT-22.SPEC-002 step 2)
- Offline/degraded: N/A beyond retry-without-corruption; progress shown for large ranges (feature-overview.md ## Non-Functional Notes)
- Scale: bounded by one freelancer's invoice history; no independent growth (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Synchronous generation in the Backend / API Layer (Framework server layer, Hono, NestJS) at this volume, or a job on Background Jobs & Scheduling options (Inngest, Trigger.dev v3 for long-running tasks, Upstash QStash, pg-boss / BullMQ workers) for large ranges; the file held in memory for the screen session per FEAT-22.SPEC-002 or placed briefly on a File & Object Storage option.

**Risks & Unknowns:** "QuickBooks/Xero-compatible" is not a single format — QuickBooks Online and Xero import templates differ in columns, date formats and multi-currency handling; the landscape covers no accounting-format option, so target templates are an open question (Section 5). Exported content is GDPR-class (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-23 — Subscription Plan & Billing Management

**Verdict:** Standard-with-integration — the freelancer's own recurring subscription runs on an external subscription-billing capability whose charge, renewal and period-end events arrive asynchronously and out of order (FEAT-23.SPEC-003 ## Degradation Behavior, ## Edge Cases), applied by a plan-state sync (FEAT-23.SPEC-004).

**Required Capabilities:**
- Upgrade, downgrade offer, cancel with acknowledgment-gated state (FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-006)
- Free-plan auto-provisioning; plan limits gating client adds (FEAT-23.SPEC-002, FEAT-23.SPEC-007; XBR-23)
- 7-day grace window after failed renewal, manual retries, lapse (platform-parameters.md `subscription-charge-grace-window-days`; FEAT-23.SPEC-004)
- Stop-billing relay retried every 15 minutes until acknowledged (FEAT-23.SPEC-003 ## Edge Cases; `stop-billing-relay-retry-interval`)
- Plan & billing emails with dedup (FEAT-23.SPEC-008)
- Concurrency: plan change vs client add vs billing status reports; billing-capability reports authoritative (feature-dependency-map.md, Subscription Plan **Contention:**)
- Offline/degraded: slow → progress then "Still working"; down → action disabled, plan unaffected; rejects → reason inline, plan unchanged (FEAT-23.SPEC-003 ## Degradation Behavior)
- Scale: one plan record per freelancer (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Payments & Billing area: Stripe Billing (0.7% of billing volume; separate charge stream from client payments on Connect — Cross-Area Compatibility Notes) or Paddle (Merchant of Record, ~5% + 50c, handles worldwide sales tax on the platform's own plan, does not replace the client-payment processor). The two differ mainly in who carries tax/VAT obligations for Clientroom's own subscription revenue. Webhook processing via Backend / API Layer plus Background Jobs & Scheduling options for retries.

**Risks & Unknowns:** Worldwide-from-day-one selling of the platform's own plan creates VAT/GST collection obligations for Clientroom itself (technical-profile.md Section 7, Geography); a non-MoR option leaves that to the operator. Stray renewal charges after lapse need refunding in the billing partner (FEAT-23.SPEC-003 ## Edge Cases). Failed-charge alerts must reach Nadia within minutes (ASMP-26).

**Spike Recommendation:** None

### FEAT-24 — Data Export & Account Deletion

**Verdict:** Hard — two materially demanding operations: aggregating every record and version history a multi-year account holds into one archive with bounded retries (FEAT-24.SPEC-003 ## Processing Logic step 4; platform-parameters.md `data-export-generation-retry-count`), and a staged classify → hold → purge deletion that must never leave the account half-deleted while spanning the database, object storage (FEAT-16.SPEC-006), the payment connection (XBR-33) and a 7-year retained-record purge sweep (FEAT-24.SPEC-004 ## Processing Logic steps 2–4; FEAT-24.SPEC-005).

**Required Capabilities:**
- Full-account export archive stored and delivered via FEAT-16, one active archive per account, 7-day download window (FEAT-24.SPEC-003; `data-export-download-window`)
- Deletion with explicit "DELETE" confirmation, pre-deletion warnings and retention classification (FEAT-24.SPEC-002, FEAT-24.SPEC-006)
- Reversible hold phase, commit point, finalization retried every 15 minutes (FEAT-24.SPEC-004; `deletion-finalization-retry-interval`)
- Daily legal-retention purge sweep for retained invoices/payments (FEAT-24.SPEC-005; `legal-retention-purge-sweep-interval`, `financial-record-legal-retention-period`)
- Export-ready and final-warning emails (FEAT-24.SPEC-008, FEAT-24.SPEC-009)
- Concurrency: in-flight events (payments, reversals, email statuses, storage confirmations) arriving during or after deletion are discarded (FEAT-10/14/16/23/32 Integration ## Edge Cases); operator excluded (XBR-29)
- Offline/degraded: failed export retried without corrupt output; failed deletion leaves the account fully intact (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: sized to a substantial multi-year account including deliverables of tens of MB to >1 GB (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Long-running archive builds fit Trigger.dev v3 (long-running tasks without serverless timeouts), Inngest (durable steps), or pg-boss / BullMQ workers on a long-running host (Fly.io, Railway, Render — Hosting & Environments); Upstash QStash suits only short HTTP steps. Archives stream to a File & Object Storage option (Cloudflare R2, Amazon S3, Backblaze B2, Supabase Storage). Staged deletion maps to a durable multi-step workflow (Inngest steps or Trigger.dev tasks) or a state-machine table driven by pg-boss; retained records sit in a restricted state on any Database-area option. Payment disconnect calls the Payments & Billing option chosen for FEAT-32.

**Risks & Unknowns:** Whether the archive includes deliverable bytes (FEAT-24.SPEC-003 step 4 lists Deliverable Version "as version history") determines whether it is a metadata file or a multi-GB bundle — this changes serverless-timeout fit and storage cost (Section 5). Object-store deletion and database deletion cannot share a transaction, so the "never half-deleted" guarantee depends on the reversible hold phase and idempotent finalization. Erasure-versus-evidence legal basis (ASMP-20) and the 7-year period are both "to be confirmed". Backups retaining deleted personal data are not addressed in the specs.

**Spike Recommendation:** None

### FEAT-25 — Refund & Cancelled Project Handling

**Verdict:** Standard-with-integration — manual refund, partial refund and cancellation recording are plain logged transitions (FEAT-25.SPEC-003, FEAT-25.SPEC-004, FEAT-25.SPEC-006), but chargeback recording consumes processor reversal notices relayed through FEAT-32's integration (FEAT-25.SPEC-005; XBR-21; feature-dependency-map.md ## External Touchpoints).

**Required Capabilities:**
- Mark refunded / partially refunded with amount ≤ amount paid (FEAT-25.SPEC-001, FEAT-25.SPEC-003; XBR-20)
- Mark project cancelled preserving history (FEAT-25.SPEC-002, FEAT-25.SPEC-004; XBR-25)
- Disputed status alongside original Paid record on reversal (FEAT-25.SPEC-005; XBR-21)
- Refund/cancellation and reversal emails (FEAT-25.SPEC-007, FEAT-25.SPEC-008)
- Concurrency: refund vs payment vs processor reports on one invoice — reject-with-refresh against current status (feature-dependency-map.md, Invoice **Contention:**)
- Offline/degraded: updates require connectivity and say so plainly; failed updates retried without ambiguous state (feature-overview.md ## Non-Functional Notes, Offline/Degraded and Responsiveness)
- Scale: infrequent relative to invoice volume (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Reversal/dispute webhooks from the Payments & Billing option serving FEAT-10/FEAT-32 (Stripe Connect Standard emits dispute events for connected accounts; Adyen for Platforms, PayPal Commerce Platform and Mollie Connect provide equivalent notifications with different event models), processed via Background Jobs & Scheduling options; status transitions under "Database optimistic concurrency and transactions" (Real-time & Collaboration).

**Risks & Unknowns:** A refund issued in the freelancer's processor account is not automatically reflected (manual marking per spec), so recorded and actual refunds can diverge; processor refund events, if emitted, are not consumed by any spec. Receiving connected-account dispute events requires the platform to subscribe to connected-account webhooks, which depends on the processor's connection model.

**Spike Recommendation:** None

### FEAT-26 — Legally Binding E-Signature for Proposals

**Verdict:** Research-spike recommended — two resolvable unknowns block confident planning: (1) which jurisdictional standard the attestation must meet is explicitly left open ("the jurisdictional standard it must meet is left to Stage 4" — FEAT-26.SPEC-005 ## Capability Category; feature-overview.md ## Non-Functional Notes, Compliance flags), and (2) whether any landscape provider accepts signature data captured in Clientroom's own signing step and returns an attestation, rather than requiring its own embedded signing ceremony (FEAT-26.SPEC-001, FEAT-26.SPEC-002 → FEAT-26.SPEC-005).

**Required Capabilities:**
- In-portal signing step capturing full legal name, Primary-only, opt-in per proposal (FEAT-26.SPEC-001, FEAT-26.SPEC-003; XBR-34)
- Attestation round-trip with write-time re-check for voided proposals and first-outcome-wins handling (FEAT-26.SPEC-002; FEAT-26.SPEC-005 ## Edge Cases)
- Signed-copy confirmation to both parties (FEAT-26.SPEC-004)
- Concurrency: voiding between submission and confirmation discards the confirmation; duplicate/out-of-order outcomes ignored after first (FEAT-26.SPEC-005 ## Edge Cases)
- Offline/degraded: slow → "Still working" after 10 s; down/rejects → inline error with name preserved; no half-written signature (FEAT-26.SPEC-005 ## Degradation Behavior)
- Scale: N/A — at most one signature per opted-in accepted proposal (feature-overview.md ## Non-Functional Notes, Data volumes); signing within the ~2 s client-facing window (ASMP-21)

**Candidate Approaches:** Electronic Signature Attestation area: Documenso (open-source, embedded signing and API, white-label on Platform tier, self-host exit), DocuSign eSignature API (widely recognized audit certificates; embedded signing on a ~$480/month tier), Dropbox Sign API (embedded signing, webhooks, from ~$100/month) and SignWell API (lower-volume plans). All four are envelope/ceremony-based services; they differ in how much of the signing UI can stay inside the branded portal, in recognized audit-trail strength, and in cost against the ~$100/month infrastructure budget (BRIEF.md ## Constraints).

**Risks & Unknowns:** The spec's model (Clientroom captures the signature, provider only attests) may not match any listed provider's API, forcing either an embedded provider ceremony (changing FEAT-26.SPEC-001's screen) or a self-built attestation — the latter has no landscape coverage. DocuSign's embedded-signing tier alone exceeds the infrastructure budget. Signer identity is GDPR-class and must survive erasure as evidence (XBR-27). Nice-to-Have, v1 phase — not on the MVP critical path.

**Spike Recommendation:** Bounded (about 3–5 days) investigation that (a) fixes the target legal standard(s) for the launch markets — e.g. US ESIGN/UETA simple e-signature versus EU eIDAS advanced signature — with a product/legal decision, and (b) tests Documenso, Dropbox Sign and SignWell APIs (DocuSign as the reference) for whether an embedded signing flow can run inside the branded portal within the ~2 s interaction target and whether a provider will attest externally captured signature data. The answer that unblocks the build team: the named standard plus one provider-compatible interaction model (external-capture attestation or embedded provider ceremony), with the per-signature cost at projected v1 volume.

### FEAT-27 — Custom Domain per Freelancer

**Verdict:** Standard-with-integration — DNS ownership verification and automated TLS for per-freelancer hostnames are provided by a domain-verification capability (FEAT-27.SPEC-002 ## Capability Category, ## Degradation Behavior), with the shared default address always serving as fallback (FEAT-27.SPEC-003; XBR-35); Later phase.

**Required Capabilities:**
- Add/replace/remove one domain per account; verification states Added → Verifying → Verified / Verification Failed with re-check (FEAT-27.SPEC-001, FEAT-27.SPEC-003)
- Secure serving of the portal and client-facing links at the verified domain (FEAT-27.SPEC-002; XBR-35)
- Verified confirmation email with dedup (FEAT-27.SPEC-004)
- Concurrency: none between humans (feature-dependency-map.md, Custom Domain Record **Contention:** None); replaced-domain in-flight results disregarded; event-time ordering (FEAT-27.SPEC-002 ## Edge Cases)
- Offline/degraded: capability down disables add/re-check while the shared address keeps serving (FEAT-27.SPEC-002 ## Degradation Behavior)
- Scale: at most one domain per freelancer, a few thousand accounts (feature-overview.md ## Non-Functional Notes; landscape Custom Domain area)

**Candidate Approaches:** Custom Domain Verification & TLS Serving area: Cloudflare for SaaS Custom Hostnames (100 free, then $0.10/hostname/month; traffic must pass through Cloudflare), Vercel Domains API (requires hosting on Vercel), Caddy on-demand TLS (self-operated proxy) or Fly.io custom domain certificates (requires Fly.io hosting). The choice is coupled to the Hosting & Environments selection (Cross-Area Compatibility Notes).

**Risks & Unknowns:** Magic-link emails and session cookies are domain-scoped — a contact signed in at the shared address is not signed in at the custom domain, and links must consistently use one host (XBR-35 "change only the address, never the experience"). Per-plan domain limits on hosting-native options are not quantified in the landscape.

**Spike Recommendation:** None

### FEAT-28 — Global Search Across Clients & Projects

**Verdict:** Straightforward — account-scoped matching across Client, Project, Proposal, Deliverable and Invoice fields with rule-based ranking (FEAT-28.SPEC-002 ## Processing Logic step 3; FEAT-28.SPEC-004) over a small per-account corpus (feature-overview.md ## Non-Functional Notes, Data volumes); no full-text or faceted demand (technical-profile.md Section 3, Search row).

**Required Capabilities:**
- 2+ character debounced query, cross-entity matching with secondary context (FEAT-28.SPEC-001, FEAT-28.SPEC-002)
- Scope rules: own account only; operator limited to the one account in her open session (FEAT-28.SPEC-003; XBR-29)
- Rule-based relevance ranking (FEAT-28.SPEC-004)
- Concurrency: N/A — read-only; captures nothing (feature-overview.md ## Non-Functional Notes, Compliance flags)
- Offline/degraded: automatic retry then manual Retry (FEAT-28.SPEC-002 ## Trigger Definition); lightweight in-progress indicator (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: 3–15 active clients with full history per account; responsive as the user types (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** PostgreSQL full-text search (tsvector/pg_trgm) inside the existing data store, honoring the same access rules without a sync pipeline (Search area), answering the profile's Search signal; or an external engine — Typesense, Meilisearch or Algolia (Search area) — for typo tolerance at the cost of an index-sync pipeline and per-tenant filtering in the index. At a few hundred records per account, the database option avoids a second copy of GDPR-class data.

**Risks & Unknowns:** An external search index holds copies of billing names and proposal scope (feature-overview.md ## Non-Functional Notes, Data sensitivity), adding a processor and a deletion-propagation path for FEAT-24. Tenant filters in an external index are a single point of isolation failure (ASMP-23).

**Spike Recommendation:** None

### FEAT-29 — In-App Notification Center

**Verdict:** Straightforward — a 90-day rolling feed composed from existing records (FEAT-29.SPEC-002; platform-parameters.md `notification-feed-retention-window`), read/unread toggles (FEAT-29.SPEC-003) and a device-cached last-loaded fallback (FEAT-29.SPEC-004 ## Processing Logic); Later phase.

**Required Capabilities:**
- Feed composition over Notification and Activity Log Entry content with 90-day window (FEAT-29.SPEC-002)
- Mark read/unread (FEAT-29.SPEC-003)
- Last-successfully-loaded feed kept on device; read-only offline state (FEAT-29.SPEC-004 steps 1–6)
- Concurrency: read/unread from two sessions is a per-item flag, last-write-wins (FEAT-29.SPEC-003); no cross-role contention
- Offline/degraded: cached feed shown read-only with toggles inert when offline; error banner with Retry on failure (FEAT-29.SPEC-004)
- Scale: rolling window bounds footprint (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Snapshot fetch with TanStack Query v5 cache persistence or SWR stale-while-revalidate (State Management) for the last-loaded fallback, answering the profile's Offline signal; feed query over any Database-area option. Push refresh via Supabase Realtime, Ably or Pusher Channels (Real-time & Collaboration) is optional — the specs describe load/refresh, not live push (technical-profile.md Section 3, Real-time = No).

**Risks & Unknowns:** Cached feed content on the device holds GDPR-class notification content (feature-overview.md ## Non-Functional Notes); clearing it on sign-out matters on shared devices.

**Spike Recommendation:** None

### FEAT-30 — Contextual Help & Guidance

**Verdict:** Straightforward — static product content with no loading state plus one bounded dismissal flag per tip per user (FEAT-30.SPEC-004, FEAT-30.SPEC-005; feature-overview.md ## Non-Functional Notes); Later phase.

**Required Capabilities:**
- Tooltips on freelancer and client screens; freelancer and client help references (FEAT-30.SPEC-001..003)
- Permanent per-user dismissal recording (FEAT-30.SPEC-004)
- Concurrency: N/A — per-user flag written only by that user (feature-overview.md ## Non-Functional Notes, Data volumes)
- Offline/degraded: N/A — static, already-rendered content (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: N/A — bounded flag set riding on existing account/contact records (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Static content bundled in the Frontend Framework (Next.js, React Router v7, SvelteKit, Nuxt 4) and cached by CDN and framework cache (Caching & Performance); accessible tooltip primitives from the component-layer candidates listed under CSS / Styling (Radix UI primitives, shadcn/ui, Headless UI, Melt UI). Dismissal flags on the Freelancer Account / Client Contact records in any Database-area option.

**Risks & Unknowns:** Tooltips overlaying client pages must not add to the ~2 s interactivity budget (ASMP-21) or break screen-reader flow (ASMP-27). Dismissal flags must be erased with contact erasure (feature-overview.md ## Non-Functional Notes, Compliance flags).

**Spike Recommendation:** None

### FEAT-31 — Operator Support Access

**Verdict:** Straightforward — a read-only, account-scoped operator session with inactivity auto-close, email announcement and trail logging (FEAT-31.SPEC-003, FEAT-31.SPEC-004 ## Processing Logic; FEAT-31.SPEC-005; XBR-29) is an established impersonation pattern; the demand is enforcement coverage, captured as a risk.

**Required Capabilities:**
- Contact-support request and operator console queue (FEAT-31.SPEC-001, FEAT-31.SPEC-002)
- Session open with read-only enforcement across every feature; excludes file downloads and data/accounting exports (FEAT-31.SPEC-003, FEAT-31.SPEC-005; XBR-29)
- Auto-close after 15 minutes of inactivity; close-write failure keeps session open and read-only (FEAT-31.SPEC-004; `support-session-inactivity-timeout-minutes`)
- Request confirmation and session-opened notice emails (FEAT-31.SPEC-006, FEAT-31.SPEC-007)
- Concurrency: none — one operator, one account at a time, record never edited after close (feature-dependency-map.md, Support Access Session **Contention:** None)
- Offline/degraded: N/A — connectivity-required operator action (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: sessions scale with support demand, opened one at a time by a solo operator (feature-overview.md ## Non-Functional Notes, Data volumes)

**Candidate Approaches:** Centralized authorization middleware in the Backend / API Layer — NestJS guards, Hono middleware or the Framework server layer — that rejects every mutation while a support session is active; identity for the operator via Authentication & Identity options (Clerk and WorkOS AuthKit offer hosted user management; Better Auth and Supabase Auth keep sessions in the application's control). Optional database-level read-only roles or Supabase row-level security (Supabase Postgres) add a second enforcement layer. Inactivity close fits a timer job on Background Jobs & Scheduling options or a check-on-next-request pattern.

**Risks & Unknowns:** Read-only must hold "in every feature" (XBR-29): any mutation path or signed download URL that bypasses the central check breaks ASMP-23's privacy posture — coverage across 33 features is the real effort. Inactivity close evaluated only on the next request leaves `closed_at` unset for idle sessions unless a timer also runs.

**Spike Recommendation:** None

### FEAT-32 — Payment Account Connection

**Verdict:** Standard-with-integration — the freelancer connects her own processor account through an external hand-off, and authoritative readiness, restriction and reversal events flow back asynchronously (FEAT-32.SPEC-002 ## Capability Category, ## Degradation Behavior, ## Edge Cases; FEAT-32.SPEC-003).

**Required Capabilities:**
- Connect/reconnect hand-off with interim Connecting record and hand-off timeout (FEAT-32.SPEC-001, FEAT-32.SPEC-002; `payment-connect-handoff-timeout`, `payment-connection-handoff-slow-threshold`)
- Readiness status, attention reason, available payment methods; status authority for pay-link readiness (FEAT-32.SPEC-003; XBR-19)
- Disconnect with hard-deleted reference and open-invoice warning (FEAT-32.SPEC-004; XBR-19)
- Relay of reversal/chargeback notices to FEAT-25 (XBR-21)
- Connection status emails (FEAT-32.SPEC-006)
- Concurrency: Nadia's connect/disconnect vs processor status events; processor authoritative; disconnect does not cancel a submitted payment (feature-dependency-map.md, Payment Account Connection **Contention:**)
- Offline/degraded: slow → "Still checking" after 5 minutes; down → connect disabled, last status kept; reject → Error state with interim record deleted (FEAT-32.SPEC-002 ## Degradation Behavior)
- Scale: one connection per freelancer; connect in under 5 minutes; ≥90% connected before first invoice (feature-overview.md ## Non-Functional Notes)

**Candidate Approaches:** Payments & Billing area: Stripe (Connect Standard with direct charges — hosted onboarding, account-status webhooks, fees on the connected account), Adyen for Platforms (sales-led onboarding, heavier integration), PayPal Commerce Platform (marketplace onboarding, consumer recognition) or Mollie Connect (Europe-focused). They differ in self-serve onboarding speed (the 5-minute target), country coverage for worldwide freelancers, and whether a standard/own-account model exists so the platform never holds funds (ASMP-28). Status events processed via Backend / API Layer plus Background Jobs & Scheduling options.

**Risks & Unknowns:** Processor onboarding for some countries requires identity verification that cannot finish in 5 minutes, threatening the Payment Readiness metric (feature-overview.md ## Non-Functional Notes). Processor country availability limits "worldwide from day one" (technical-profile.md Section 7) — freelancers in unsupported countries fall back to XBR-19's direct-payment instructions. The connection reference is GDPR-class and hidden from the operator (feature-overview.md ## Non-Functional Notes).

**Spike Recommendation:** None

### FEAT-33 — Portal Referral Attribution

**Verdict:** Straightforward — a discreet mark on client-facing pages and emails (FEAT-33.SPEC-001), a referral reference captured with a 30-minute inactivity window (FEAT-33.SPEC-002; `referral-capture-session-window`) and one attribution record at sign-up (FEAT-33.SPEC-004), with aggregate-only access (FEAT-33.SPEC-005).

**Required Capabilities:**
- Mark rendering on every client page and email on every plan without leaking client/project/freelancer data (FEAT-33.SPEC-001; XBR-32)
- Referral capture and landing page (FEAT-33.SPEC-002, FEAT-33.SPEC-003)
- Attribution recording once per sign-up; aggregate-only use (FEAT-33.SPEC-004, FEAT-33.SPEC-005)
- Concurrency: none — recorded once and never edited (feature-dependency-map.md, Referral Attribution **Contention:** None)
- Offline/degraded: N/A — static lightweight surfaces (feature-overview.md ## Non-Functional Notes, Responsiveness)
- Scale: one record per sign-up, not per view (feature-overview.md ## Non-Functional Notes, Data volumes); landing page within ~2 s on mobile (ASMP-21)

**Candidate Approaches:** Referral reference carried in the link and held in a first-party cookie or session on the Frontend Framework (Next.js, React Router v7, SvelteKit, Nuxt 4); landing page served from CDN and framework cache (Caching & Performance). Aggregate growth-loop measurement via Analytics & Product Telemetry options — Plausible (cookie-less) or PostHog, Mixpanel, Amplitude (event funnels).

**Risks & Unknowns:** The referral reference must identify the referring account without exposing it in a way that reveals the client or project (XBR-32) — an opaque token rather than a readable account slug. Cookie-based capture may require consent disclosure in some jurisdictions (ASMP-24 GDPR-class posture).

**Spike Recommendation:** None

## 3. Cross-Feature Technical Themes

| Theme / Shared Subsystem | Features Involved | Evidence That Makes It Shared |
|--------------------------|-------------------|-------------------------------|
| Transactional email dispatch, status tracking and retry | FEAT-02, FEAT-03, FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-14, FEAT-18, FEAT-20, FEAT-21, FEAT-23, FEAT-24, FEAT-25, FEAT-26, FEAT-27, FEAT-31, FEAT-32 | 26 Notification specs all carry Dedup rules and the same `transactional-email-retry-count` / `-window` parameters in ## Delivery Rules (e.g. FEAT-02.SPEC-011, FEAT-11.SPEC-004, FEAT-32.SPEC-006); single delivery path FEAT-14.SPEC-001 / FEAT-14.SPEC-003 (feature-dependency-map.md ## External Touchpoints, email row) |
| Background job execution and scheduled sweeps | FEAT-09, FEAT-11, FEAT-12, FEAT-13, FEAT-14, FEAT-16, FEAT-23, FEAT-24, FEAT-31, FEAT-32 | Reminder schedule (FEAT-11.SPEC-001), totals refresh (FEAT-12.SPEC-004), activity-entry write retry (FEAT-13.SPEC-003), email retry (FEAT-14.SPEC-003), storage aggregation (FEAT-16.SPEC-005), stop-billing relay (FEAT-23.SPEC-003), archive generation and retention purge (FEAT-24.SPEC-003, FEAT-24.SPEC-005), support auto-close (FEAT-31.SPEC-004), connection-status apply retry (FEAT-32.SPEC-003; `connection-status-apply-retry-interval`) |
| Idempotent, event-time-ordered inbound webhook ingestion | FEAT-10, FEAT-14, FEAT-16, FEAT-23, FEAT-25, FEAT-26, FEAT-27, FEAT-32 | Every Integration spec's ## Edge Cases specifies "same event delivered twice changes nothing" and "most recent by event time, not arrival" (FEAT-10.SPEC-003, FEAT-14.SPEC-001, FEAT-16.SPEC-007, FEAT-23.SPEC-003, FEAT-26.SPEC-005, FEAT-27.SPEC-002, FEAT-32.SPEC-002); FEAT-25.SPEC-005 consumes relayed reversal notices (XBR-21) |
| Reject-with-refresh optimistic concurrency and exactly-once writes | FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-15, FEAT-18, FEAT-23, FEAT-25, FEAT-32 | 12 of 21 entities carry non-None **Contention:** lines resolved reject-with-refresh (feature-dependency-map.md ## Shared Data Entities; technical-profile.md Section 3 Collaboration/concurrency); exactly-once acceptance (FEAT-03.SPEC-003) and approval (FEAT-08.SPEC-003) |
| Evidentiary immutability and append-only records | FEAT-02, FEAT-03, FEAT-08, FEAT-09, FEAT-10, FEAT-13, FEAT-17, FEAT-25, FEAT-26 | XBR-04 names accepted proposals, approvals, sent invoices and trail entries as immutable; FEAT-09.SPEC-008, FEAT-13.SPEC-004, FEAT-17.SPEC-004 immutability rules; ASMP-25 |
| Large-file storage and delivery | FEAT-06, FEAT-16, FEAT-17, FEAT-19, FEAT-24 | FEAT-16.SPEC-007 ## Degradation Behavior rows cover FEAT-06 and FEAT-17 screens; FEAT-24.SPEC-003 stores archives via FEAT-16.SPEC-007; FEAT-19.SPEC-002 logo upload; storage allowance shared (XBR-13, XBR-14) |
| Role-based authorization and strict client isolation | FEAT-03, FEAT-05, FEAT-07, FEAT-08, FEAT-09, FEAT-13, FEAT-14, FEAT-18, FEAT-21, FEAT-28, FEAT-31 | XBR-08 (Access Matrix everywhere), XBR-09 (client isolation); authorization rule specs FEAT-05.SPEC-007, FEAT-07.SPEC-007, FEAT-08.SPEC-006, FEAT-09.SPEC-006, FEAT-13.SPEC-005, FEAT-14.SPEC-004, FEAT-18.SPEC-007, FEAT-21.SPEC-010, FEAT-28.SPEC-003, FEAT-31.SPEC-005 |
| Payment-processor integration (connection, payments, reversals) | FEAT-09, FEAT-10, FEAT-20, FEAT-25, FEAT-32 | feature-dependency-map.md ## External Touchpoints payment row (FEAT-10.SPEC-003, FEAT-32.SPEC-002; XBR-19, XBR-21) |
| Financial derivation from Invoice/Payment truth, per currency | FEAT-12, FEAT-15, FEAT-22, FEAT-25 | XBR-18 (never convert or add currencies), XBR-22 (totals only from Invoice and Payment incl. refunds/reversals); FEAT-12.SPEC-003, FEAT-15.SPEC-007, FEAT-22.SPEC-002 step 6–7 |
| Offline/degraded client-side state (queue or last-loaded cache) | FEAT-04, FEAT-07, FEAT-12, FEAT-29 | Queued save (FEAT-04.SPEC-001 Offline/Degraded), offline comment queue (FEAT-07.SPEC-008), last loaded totals (FEAT-12.SPEC-001), cached feed (FEAT-29.SPEC-004); ASMP-27 forbids record-creating actions pretending to succeed offline |
| Personal-data erasure and legal retention | FEAT-13, FEAT-16, FEAT-18, FEAT-24, FEAT-30, FEAT-33 | XBR-27 (erasure keeps names on evidence), XBR-33 (deletion keeps only legally retained financial records); FEAT-13.SPEC-006, FEAT-16.SPEC-006, FEAT-18.SPEC-009, FEAT-24.SPEC-004/005; dismissal flags and referral records deleted with accounts (FEAT-30, FEAT-33 feature-overview.md ## Non-Functional Notes) |
| Time-zone-aware scheduling and display | FEAT-11, FEAT-15, FEAT-24 | Reminder day counts in freelancer time zone (FEAT-11.SPEC-001; FEAT-15.SPEC-006; XBR-15); retention periods measured from deletion completion (FEAT-24.SPEC-005) |
| Per-freelancer branding across portal and email | FEAT-14, FEAT-19, FEAT-27, FEAT-33 | XBR-31 (logo/colour on every client screen and email, referral mark alongside), FEAT-14.SPEC-005 branded presentation, XBR-35 custom domain changes only the address |
| Printable / downloadable record documents | FEAT-09, FEAT-13 | Download a printable copy of invoice, credit note or receipt (FEAT-09.SPEC-002 Access table); printable, unalterable trail copy (FEAT-13.SPEC-002) — no landscape area covers document rendering |

## 4. Key Technical Risks

| Risk | Features Affected | Driving Evidence | Possible Mitigation Directions |
|------|-------------------|------------------|--------------------------------|
| Storage cost exceeds the ~$100/month infrastructure budget as versions accumulate uncapped under a 100 GB paid allowance | FEAT-16, FEAT-17, FEAT-06, FEAT-24 | BRIEF.md ## Constraints; platform-parameters.md `paid-tier-storage-allowance` 100 GB, `free-tier-storage-allowance` 5 GB; FEAT-17 feature-overview.md ## Non-Functional Notes (versions uncapped, main volume driver); landscape File & Object Storage pricing | Zero- or low-egress storage options (Cloudflare R2, Backblaze B2) as a cost criterion for the Architect; revisiting allowance parameters against plan revenue ($15/month); modeling actual year-one stored bytes before launch |
| Invoice numbering gaps or duplicates under concurrent automatic and manual creation | FEAT-09, FEAT-03, FEAT-08, FEAT-01 | FEAT-09.SPEC-007 ("never reused, never skipped"); FEAT-09.SPEC-004 three trigger sources; Invoice **Contention:** | Per-freelancer locked counter inside the creating transaction; unique constraint with retry; avoiding native sequences that skip on rollback |
| Missed, duplicated or out-of-order processor events corrupt invoice/payment state | FEAT-10, FEAT-25, FEAT-32, FEAT-23 | FEAT-10.SPEC-003, FEAT-32.SPEC-002, FEAT-23.SPEC-003 ## Edge Cases; Invoice and Payment **Contention:** (processor authoritative) | Idempotency keys per event and per payment attempt; event-time ordering; durable webhook queue with replay; periodic reconciliation against processor state |
| Cross-system deletion leaves partial state (DB deleted, objects retained, or vice versa) | FEAT-24, FEAT-16, FEAT-32 | FEAT-24.SPEC-004 staged hold/commit/finalize; FEAT-16.SPEC-006; XBR-33; feature-overview.md "never half-deleted" | Durable multi-step workflow with idempotent finalization; object-deletion ledger checked by sweep; treating backups within the retention story |
| Operator read-only enforcement gap exposes or mutates freelancer data | FEAT-31, all features with mutations | XBR-29 ("read-only in every feature", excludes downloads/exports); ASMP-23 | Central mutation guard in one backend layer plus database-level read-only role; automated tests enumerating every mutation endpoint under a support session |
| Email failures not surfaced "within minutes" for transient failures | FEAT-14, FEAT-02, FEAT-05, FEAT-09, FEAT-11 | ASMP-26 vs FEAT-14.SPEC-003 step 6 and `transactional-email-retry-window` 6 hours | An early "delayed" warning before retry exhaustion; shorter retry window; separating bounce (immediate) from transient failure messaging |
| Magic links consumed by email security scanners, failing the 95% first-try target | FEAT-05, FEAT-18 | FEAT-05.SPEC-006 single-use rule; feature-overview.md ## Non-Functional Notes (≥95% first-try) | Confirm-click interstitial before token consumption; tolerance window for scanner GETs; measurement during beta |
| E-signature attestation model has no matching provider or exceeds budget | FEAT-26 | FEAT-26.SPEC-005 ## Capability Category (standard left open); landscape Electronic Signature Attestation pricing | The Section 2 spike; fallback to the timestamped Accept (FEAT-03, XBR-34) that already meets the brief's evidence requirement |
| Payment-processor country coverage and onboarding time undercut worldwide launch and the 5-minute connect target | FEAT-32, FEAT-10, FEAT-20 | technical-profile.md Section 7 (worldwide from day one); FEAT-32 feature-overview.md ## Non-Functional Notes (under 5 minutes, ≥90% before first invoice) | Processor coverage as a selection criterion; XBR-19 direct-payment fallback for unsupported countries; more than one processor as a later option |
| Linked-asset reachability check misclassifies private Figma/Drive/Dropbox links | FEAT-06 | FEAT-06.SPEC-004 ## Processing Logic step 3 | Heuristics on provider response content; provider-specific public-link checks; allowing Nadia to confirm a flagged link |

## 5. Open Questions for the Build Team

| # | Question | Why It Matters | What Would Resolve It |
|---|----------|----------------|------------------------|
| 1 | Which legal e-signature standard(s) must FEAT-26 meet, and does any landscape provider attest externally captured signatures? | Determines FEAT-26's verdict resolution, provider, screen design and cost (FEAT-26.SPEC-005 ## Capability Category) | The FEAT-26 spike plus a product/legal decision on target jurisdictions |
| 2 | How do the ~$100/month budget and the 100 GB paid / 5 GB free storage allowances reconcile at year-one volume? | FEAT-16/17 cost can exceed budget; affects storage-option choice and plan pricing (platform-parameters.md; BRIEF.md ## Constraints) | A cost model from expected freelancer mix and average stored bytes, and a product decision on allowances or budget |
| 3 | Does the full data-export archive include deliverable file bytes or only metadata/version history? | Changes FEAT-24 archive size from MB to many GB, job-runtime fit and storage cost (FEAT-24.SPEC-003 step 4) | A product decision against GDPR data-portability expectations |
| 4 | Which document-rendering approach produces printable invoices, credit notes, receipts and trail copies? | FEAT-09.SPEC-002 and FEAT-13.SPEC-002 require printable, unalterable copies; no landscape area covers PDF/document rendering | A landscape extension or build-team choice of a rendering approach |
| 5 | Which exact QuickBooks and Xero import templates must the export match? | FEAT-22's "QuickBooks/Xero-compatible" is not one format; multi-currency import rules differ (FEAT-22.SPEC-003) | Product decision on target products/editions, validated by test imports |
| 6 | Is the 6-hour email retry window compatible with "surfaced within minutes" (ASMP-26)? | Affects FEAT-14 warning timing and every feature relying on email delivery | A product decision on an early-warning threshold or a shorter window |
| 7 | Which countries must payment connection and bank transfer support at launch? | Drives processor choice and FEAT-32/FEAT-10 feasibility for "worldwide from day one" (technical-profile.md Section 7) | Product decision on launch markets checked against processor country lists |
| 8 | Who carries VAT/GST obligations on Clientroom's own subscription revenue? | Separates Merchant-of-Record billing (Paddle) from direct billing (Stripe Billing) for FEAT-23 | A business/tax decision by the operator |
| 9 | What is the confirmed legal retention period and the legal basis for keeping erased contacts' names on evidence? | FEAT-24.SPEC-005 sweep, FEAT-13.SPEC-006 and XBR-27 depend on it; platform-parameters.md says "to be confirmed with legal counsel"; ASMP-20 | Qualified privacy/legal advice before launch |
| 10 | Are preview renditions (thumbnails, transcoded video) expected for large deliverables on mobile? | FEAT-17.SPEC-002 side-by-side viewing and the ~2 s mobile target (ASMP-21) may be unachievable streaming originals; no landscape area covers media processing | A product decision on preview expectations, then a landscape extension if needed |
| 11 | How should a linked asset that returns a sign-in page be treated? | FEAT-06.SPEC-004 reachability outcome for private Figma/Drive/Dropbox links; no landscape option covers provider link introspection | A product decision on flag-vs-warn behavior, informed by testing the three providers |
| 12 | Should portal sessions and links be shared between the default address and a verified custom domain? | FEAT-27 cookie/session scoping and FEAT-05 magic-link host (XBR-35) | A product decision on canonical host behavior once a domain is verified |


## The Decisions — Technical Architecture

Section 1 below repeats the technical profile — by design; it is the architecture document's own embedded evidence base.


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
