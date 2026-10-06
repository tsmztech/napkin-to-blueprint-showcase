---
document_type: technical-feasibility
produced_by: feasibility-planner
status: final
stage: 4
feature_count: 33
created: 2026-09-29
---

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
