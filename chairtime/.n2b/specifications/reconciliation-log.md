---
document_type: reconciliation-log
produced_by: cross-reference-reconciler
status: final
created: 2026-09-28
specs_reviewed: 219
edits_applied: 195
files_edited: 71
structural_gaps: 17
missing_specs: 1
final_alignment_pass: complete
gaps_resolved: 14
gaps_carried_forward_stage4: 4
---

# Reconciliation Log — Pass D

Cross-reference reconciliation of all 218 specs across 30 features (69 screen, 59 automation, 53 logic-rule, 13 integration, 24 notification), the 30 Feature Breakdown Briefs and the Feature Dependency Map (including External Touchpoints). Every check was run by script over the full spec set, not by recall. Every alignment issue found was resolved by a direct edit (logged per edit below, each with status **applied**); every gap outside the Reconciler's edit authority is classified `[STRUCTURAL-GAP]` or `[MISSING-SPEC]` and **routed to the orchestrator** (status recorded per entry). No entry is left without a disposition.

## Checks Run

| # | Check | Result | Disposition |
|---|-------|--------|-------------|
| 1 | Feature Breakdown Brief completeness | 218 spec files ↔ 30 Brief inventories: 0 orphan files, 0 phantom inventory entries. Five Brief sentences use shorthand sibling-feature IDs (e.g. "FEAT-16.SPEC-002/SPEC-005", "FEAT-08 (SPEC-001, …)") that read as phantom IDs to a naive parser | Pass; Brief shorthand noted, Briefs not editable by the Reconciler |
| 2 | Cross-feature references resolve | 0 dangling `FEAT-NN.SPEC-NNN` IDs in specs, Briefs or the map; 14 malformed/feature-level references (`FEAT-05.SPEC`, `FEAT-10.SPEC`, `FEAT-11.SPEC`, `FEAT-30.SPEC`, `FEAT-15.SPEC`, `FEAT-05.SPEC-*`, `FEAT-05.SPEC-001-equivalent`) | Applied (14 edits) |
| 3 | Intra-feature references resolve | 1 sibling reference ("SPEC-005" in FEAT-19.SPEC-003 meaning FEAT-16.SPEC-005) | Applied (1 edit) |
| 4 | Shared entity consistency | Booking.state "Expired" vs "Expired (unpaid)" (FEAT-03.SPEC-007, FEAT-30.SPEC-010, FEAT-11.SPEC-004); Pro Account `sign-in_email`/`sign-in_mobile` spelling (FEAT-19.SPEC-001) | Applied (12 edits) |
| 5 | Bidirectional navigation (incl. Notification CTA deep links) | 18 forward screen→screen links and 11 notification CTA deep links with no matching Entry Points row; 20 feature-level source/destination references. Back-arrow returns, embedded sections (FEAT-23.SPEC-001 in FEAT-22.SPEC-001) and placeholder mentions were excluded as non-navigation | Applied (51 edits, Rule 2); 6 source-side gaps routed (SG-04, SG-05, SG-07, SG-10) |
| 6 | Bidirectional automation triggers (incl. Integration external-event sources) | 35 one-way links closed by tightening feature-level Connected Specs / trigger-source rows to exact IDs | Applied (35 edits); remaining one-way links routed (SG-06, SG-11, SG-12) |
| 7 | Logic/Rule consistency | No contradictory enforcement wording found in spot-checked pairs; one intra-feature count mismatch (FEAT-15.SPEC-007 "eight required steps" vs its own seven conditions); 38 Enforced-By pairs lack a back-reference | Applied (1 edit); back-references routed (SG-13) |
| 8 | Spec ID uniqueness | 218 unique IDs; every file name matches its frontmatter `spec_id` | Pass |
| 9 | Cross-feature business rules | XBR-23 balance-refund routing (FEAT-30.SPEC-007/008 cited FEAT-30.SPEC-011 instead of FEAT-22.SPEC-005); XBR-12 auto-completion window stated two ways (FEAT-11 marker vs FEAT-12 concrete 7 days); FEAT-18 terminal cancellation signal name vs Stage 2 Signals | Applied (6 edits + FEAT-12/FEAT-11 alignment under Check 14); FEAT-09.SPEC-005 balance trigger routed (SG-09); hold timing routed (SG-03) |
| 10 | Entity lifecycle completeness | Every map lifecycle operation has a spec in the named feature except Recurring Series "created by … Pro via FEAT-30" (no Pro-side screen) and Waitlist Entry read by FEAT-19 (support view). Specs also perform writes the map omits | MS-01 routed; map omissions routed (SG-01, SG-02) |
| 11 | External Touchpoints ↔ Integration specs | 9 touchpoint rows; all 13 Integration specs (FEAT-03.SPEC-006, FEAT-04.SPEC-003, FEAT-07.SPEC-005, FEAT-08.SPEC-012, FEAT-08.SPEC-013, FEAT-09.SPEC-005, FEAT-16.SPEC-003, FEAT-18.SPEC-006, FEAT-22.SPEC-005, FEAT-26.SPEC-002, FEAT-27.SPEC-012, FEAT-28.SPEC-006, FEAT-30.SPEC-011) cited, and every cited Integration spec exists. Non-integration IDs in the table (notification/automation specs "delivered through" a row) all exist | Pass (both directions) |
| 12 | Notification trigger sources | 24 Notification specs: every trigger source exists; FEAT-08.SPEC-001/004/005 had feature-level or unacknowledged sources | Applied (9 edits); now every source names the notification in its Outcome Definitions / Inbound Events / Interactions |
| 13 | Degradation Behavior screen references | Every spec ID named in the 13 Degradation Behavior sections exists; every screen-surface ID is a Screen spec | Pass |
| 14 | Platform-parameter markers → registry | 289 strict marker sites, 33 distinct slugs after normalization; near-misses normalized (capitalized marker, slug without prefix, hyphenated "platform-parameter value", "fixed platform-wide", unmarked policy numbers) | Applied (40 edits); `platform-parameters.md` written (33 rows, all `decide-before-build`) |

## Resolved Carry-Forward Items (from STAGE.md Deviations)

| Item | Disposition |
|------|-------------|
| FEAT-11 `booking-auto-completion-window-days` marker vs FEAT-12's concrete 7 days | Applied: FEAT-12 (XBR-12 authority) keeps the testable "7 days" and now carries the same marker; FEAT-11 keeps the marker and states "7 days per XBR-12" where it had "fixed platform-wide". One registry row, proposed default 7 days |
| FEAT-22 balance-refund ownership vs FEAT-30.SPEC-007 / FEAT-09.SPEC-005 | Applied for FEAT-30.SPEC-007/008 (now route via FEAT-22.SPEC-005); FEAT-09.SPEC-005 half routed as SG-09 |
| FEAT-19.SPEC-001 `sign-in_email` hyphenation | Applied (sign_in_email / sign_in_mobile) |
| FEAT-18 analytics names vs Stage 2 Signals | Applied (FEAT-18.SPEC-006 terminal event renamed `subscription_cancelled`) |
| FEAT-15.SPEC-005 payout trigger source (FEAT-28.SPEC-003 vs map FEAT-28.SPEC-006) | Consistent chain confirmed: FEAT-28.SPEC-006 (Integration, inbound status) → FEAT-28.SPEC-003 (automation) → FEAT-15.SPEC-005; the map row cites FEAT-28.SPEC-006 for the processor hand-off only. FEAT-28.SPEC-003 Connected Specs tightened to name FEAT-15.SPEC-005 |
| FEAT-05/FEAT-30 merge-on-match | Confirmed: FEAT-05.SPEC-003 (Business Rules, Interactions) and FEAT-30.SPEC-004 (Edge Cases) resolve a phone match within a Pro to the existing Client, never a duplicate |
| FEAT-24.SPEC-001 → FEAT-13.SPEC-002 reference | Confirmed correct: FEAT-13.SPEC-002 is Client Contact Edit, the Client-update owner |
| FEAT-22 reuse of `minimum-chargeable-deposit` | Confirmed deliberate; one registry row; near-miss marker sentence normalized |
| FEAT-05.SPEC-001/008 bare "12 months" | Applied: markers `booking-link-forward-window-months` added |
| FEAT-03.SPEC-001 "FEAT-05.SPEC-001-equivalent" / bare "FEAT-10.SPEC"; FEAT-04.SPEC-005 feature-level trigger sources | Applied (exact IDs) |
| FEAT-16/FEAT-19 feature-level references "not yet specified in this run" | Applied (FEAT-19.SPEC-001/002/003 IDs) |
| FEAT-06 Brief omits SPEC-002 Booking read; FEAT-16 Brief omits SPEC-004 governed-by SPEC-005; FEAT-18 Brief matrix names SPEC-006 as price writer | Brief-level, routed as SG-15 |
| FEAT-06.SPEC-004 Cancellation Policy reader missing from map; FEAT-10 as Booking creator | Map-level, routed as SG-01 |
| FEAT-26.SPEC-001 entry point as prospective addition to FEAT-06.SPEC-005 | Routed as SG-10 |

## Gap Report (routed to the orchestrator)

Each entry was routed to the orchestrator in Pass D (the Reconciler cannot add spec content, edit Briefs or edit the dependency map; the orchestrator re-spawns the Spec Writer for `[STRUCTURAL-GAP]` or the Feature Analyst for `[MISSING-SPEC]`). The Status column shows the disposition after gap routing, as re-verified against the files on disk in the Final Alignment Pass below: `resolved` (with what closed it) or `carried-forward (Stage 4)` (with evidence). SG-16 and SG-17 were found in the final pass.

| ID | Type | Source | Target | What is missing | Evidence | Status |
|----|------|--------|--------|-----------------|----------|--------|
| SG-01 | [STRUCTURAL-GAP] | feature-dependency-map.md (Shared Data Entities) | FEAT-10.SPEC-004; FEAT-20.SPEC-001; FEAT-06.SPEC-004; FEAT-18.SPEC-001/FEAT-18.SPEC-006; FEAT-14.SPEC-001/FEAT-14.SPEC-005; FEAT-27.SPEC-006 | Entity lifecycle entries the map does not carry (map is outside Reconciler edit authority): Booking creators omit FEAT-10 (FEAT-10.SPEC-004 creates a new Booking on a late reschedule); Client creators omit FEAT-20 (FEAT-20.SPEC-001 creates/matches a Client on waitlist join); Cancellation Policy readers omit FEAT-06 (FEAT-06.SPEC-004 reads it); Subscription is created by FEAT-18.SPEC-001/FEAT-18.SPEC-006 (reached from FEAT-15), not FEAT-15; Messaging Consent re-grant is owned by FEAT-14.SPEC-001/FEAT-14.SPEC-005, not FEAT-06; the Help Request entity created by FEAT-27.SPEC-006 and read by FEAT-19 has no Shared Data Entities entry | Spec files state these writes/reads; map Lifecycle lines list different or no features (STAGE.md Pass A/B/C carry-forwards) | carried-forward (Stage 4) — dependency-map lifecycle deltas; the map is outside Reconciler, producer and analyst authority, so the orchestrator deliberately carries these to Stage 4 as known map deltas (passd-gap-decisions.md) |
| SG-02 | [STRUCTURAL-GAP] | feature-dependency-map.md (entity states) | FEAT-18.SPEC-004/FEAT-18.SPEC-005; FEAT-21.SPEC-003/FEAT-21.SPEC-006 | State lists differ: Subscription status in the map lacks the post-grace lapsed/paused outcome FEAT-18.SPEC-004 drives; Recurring Series lists a Paused state that FEAT-21 excludes as a Non-Goal (Active/Ended only) | Map Subscription.status = Active \| Payment Failed \| Cancelled; Recurring Series.state = Active \| Paused \| Ended; FEAT-21.SPEC-003/006 Non-Goals | carried-forward (Stage 4) — dependency-map state-list deltas (Subscription lapsed/paused outcome; Recurring Series Paused vs FEAT-21 Active/Ended); map outside re-spawn authority, carried to Stage 4 by orchestrator decision |
| SG-03 | [STRUCTURAL-GAP] | FEAT-05.SPEC-006 | FEAT-03.SPEC-002 | Checkout-hold timing conflict: FEAT-05.SPEC-006 requests the hold (and creates the Pending Payment Booking) at slot pick, while FEAT-03.SPEC-002 (XBR-02 authority) creates it when the client advances into the payment step. One moment must be chosen and both specs aligned; the Reconciler cannot rewrite either processing sequence without changing behavior | FEAT-05.SPEC-006 Processing Logic step 1 ("On a slot-pick trigger: request a checkout Slot Hold"); FEAT-03.SPEC-002 Trigger Definition ("advances into the deposit payment step") | resolved — FEAT-05.SPEC-006 now places the hold and creates the Pending Payment Booking when the client advances into the payment step (Purpose, Trigger Definition, Processing step 1, Business Rules), matching FEAT-03.SPEC-002 (XBR-02 owner); FEAT-05.SPEC-002/004 aligned. Final pass E-180/E-181 aligned a stale step number and state name |
| SG-04 | [STRUCTURAL-GAP] | FEAT-05.SPEC-004 | FEAT-07.SPEC-001 | FEAT-07.SPEC-001 lists FEAT-05.SPEC-004 as its entry point (map: FEAT-05 policy acknowledgment -> FEAT-07 deposit payment), but FEAT-05.SPEC-004 declares no navigation to FEAT-07.SPEC-001 and goes straight to FEAT-05.SPEC-005 on successful payment; both specs describe card entry (FEAT-07 should own it) | FEAT-05.SPEC-004 Navigation Out (Back / Successful payment / Slot lost); FEAT-07.SPEC-001 Entry Points | resolved — FEAT-05.SPEC-004 Navigation Out "Acknowledge & continue (hold placed) → FEAT-07.SPEC-001"; card entry removed from FEAT-05.SPEC-004 (Non-Goals, AC-05) and owned by FEAT-07.SPEC-001, which returns to FEAT-05.SPEC-005 on success. Final pass E-170 names FEAT-05.SPEC-006 at FEAT-07.SPEC-001's pre-charge check |
| SG-05 | [STRUCTURAL-GAP] | FEAT-05.SPEC-003 | FEAT-06.SPEC-001 | FEAT-06.SPEC-001 claims entry from FEAT-05.SPEC-003 when a returning phone number is recognized, but FEAT-05.SPEC-003 performs the lookup in place (name pre-fill) and declares no navigation to FEAT-06.SPEC-001; either the entry row or a FEAT-05.SPEC-003 navigation row must go | FEAT-05.SPEC-003 Interactions (Phone input blur) and Navigation Out; FEAT-06.SPEC-001 Entry Points (ID tightened from "FEAT-05.SPEC-*" in this pass) | resolved — FEAT-05.SPEC-003 performs the lookup in place and says it navigates to no FEAT-06 screen; FEAT-06.SPEC-001 Entry Points no longer claim FEAT-05.SPEC-003 (Connected Specs row "Related (no navigation)") |
| SG-06 | [STRUCTURAL-GAP] | FEAT-30.SPEC-007 | FEAT-08.SPEC-010 | XBR-18: a Pro reschedule issues a fresh manage link. FEAT-08.SPEC-010 is triggered by FEAT-30.SPEC-007 (tightened in this pass), but FEAT-30.SPEC-007 never names FEAT-08.SPEC-010 or the link reissue in its outcomes/Connected Specs | FEAT-30.SPEC-007 Outcome "Reschedule committed" and Connected Specs; FEAT-08.SPEC-010 Trigger Definition | resolved — FEAT-30.SPEC-007 Processing step 8, "Reschedule committed" outcome, Business Rules, Connected Specs and AC-15 name FEAT-08.SPEC-010 and the fresh manage link (XBR-18) |
| SG-07 | [STRUCTURAL-GAP] | FEAT-08.SPEC-004 / FEAT-08.SPEC-005 | FEAT-10.SPEC-006; FEAT-30.SPEC-012; FEAT-30.SPEC-013; FEAT-14.SPEC-001 | Duplicate communication/screen ownership needing a canonical owner: FEAT-10.SPEC-006 and FEAT-30.SPEC-012 are trigger-and-audience contracts for content FEAT-08.SPEC-004/005 own, but FEAT-08.SPEC-004/005 do not list them in Connected Specs; FEAT-30.SPEC-013 expiry notice overlaps FEAT-03.SPEC-007/FEAT-08.SPEC-006; FEAT-14.SPEC-001 duplicates FEAT-06.SPEC-005 (FEAT-06.SPEC-003/004 "Preferences" navigate to FEAT-06.SPEC-005 while FEAT-14.SPEC-001 claims the same entry) | FEAT-10.SPEC-006 carry-forward note; FEAT-30.SPEC-012 RECONCILIATION NOTE; FEAT-14.SPEC-001 Entry Points vs FEAT-06.SPEC-003/004 Navigation Out | resolved — FEAT-08.SPEC-004/005 Trigger and Connected Specs name FEAT-10.SPEC-006 and FEAT-30.SPEC-012 as trigger-and-audience contracts; FEAT-30.SPEC-013 rescoped to reference FEAT-03.SPEC-007 (trigger) and FEAT-08.SPEC-006 (content); FEAT-14.SPEC-001 rescoped as the consent section hosted in FEAT-06.SPEC-005, which is the single Preferences screen |
| SG-08 | [STRUCTURAL-GAP] | FEAT-30.SPEC-010 | FEAT-03.SPEC-007 | Both automations transition the same Booking to Expired (unpaid) on an unpaid deposit-request hold (XBR-02 names FEAT-03 owner); one writer must be chosen | FEAT-03.SPEC-007 Processing step 10; FEAT-30.SPEC-010 Processing step 7 | resolved — FEAT-30.SPEC-010 Processing step 7, Data Model and Business Rules defer the Expired (unpaid) write to FEAT-03.SPEC-007 as sole writer; final pass E-182/E-183 aligned the remaining state name and signal citation |
| SG-09 | [STRUCTURAL-GAP] | FEAT-09.SPEC-005 | FEAT-22.SPEC-005 | XBR-23: a paid balance is refunded when a client or automatic cancellation happens; FEAT-22.SPEC-005 lists FEAT-09 as "Triggered by (inbound)" but FEAT-09.SPEC-005 has no Balance Payment refund trigger/outcome | FEAT-22.SPEC-005 cross-feature note; FEAT-09.SPEC-005 Outcome Definitions (deposit only) | resolved — FEAT-09.SPEC-005 adds the Balance Payment refund hand-off to FEAT-22.SPEC-005 (Scope, Outcome Definitions "Balance refund due"/"reported back", Edge Cases, Connected Specs, AC-14/AC-15), XBR-23 |
| SG-10 | [STRUCTURAL-GAP] | FEAT-05.SPEC-002; FEAT-05.SPEC-005; FEAT-06.SPEC-003; FEAT-06.SPEC-005; FEAT-02.SPEC-001 | FEAT-20.SPEC-001; FEAT-21.SPEC-001; FEAT-21.SPEC-002; FEAT-26.SPEC-001; FEAT-17.SPEC-001 | Destination screens declare entry points that the source screens do not offer: FEAT-05.SPEC-002 still says waitlist join is "not built" (v1 FEAT-20.SPEC-001 now exists); FEAT-05.SPEC-005 is marked terminal with no recurring offer (FEAT-21.SPEC-001); FEAT-06.SPEC-003 has no recurring-series section (FEAT-21.SPEC-002); FEAT-06.SPEC-005 lacks the "Message channel" element and its Non-Goal excludes WhatsApp (FEAT-26.SPEC-001, prospective addition); FEAT-02.SPEC-001 lists FEAT-17 only as an informational row, no navigation to FEAT-17.SPEC-001 | Source Navigation Out / Layout sections vs destination Entry Points | resolved — source-side navigation present: FEAT-05.SPEC-002 → FEAT-20.SPEC-001; FEAT-05.SPEC-005 → FEAT-21.SPEC-001; FEAT-06.SPEC-003 → FEAT-21.SPEC-002; FEAT-06.SPEC-005 Message channel → FEAT-26.SPEC-001 (WhatsApp Non-Goal removed); FEAT-02.SPEC-001 → FEAT-17.SPEC-001 |
| SG-11 | [STRUCTURAL-GAP] | FEAT-05.SPEC-005, FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-006, FEAT-09.SPEC-002, FEAT-19.SPEC-002; FEAT-07.SPEC-002, FEAT-10.SPEC-004, FEAT-11.SPEC-003, FEAT-12.SPEC-004, FEAT-12.SPEC-006, FEAT-30.SPEC-007; FEAT-05.SPEC-003, FEAT-05.SPEC-004; FEAT-04.SPEC-002; FEAT-07.SPEC-002, FEAT-11.SPEC-002, FEAT-30.SPEC-010 | FEAT-16.SPEC-002; FEAT-25.SPEC-004; FEAT-14.SPEC-003; FEAT-04.SPEC-004; FEAT-04.SPEC-005 | One-way trigger links where the source has no feature-level row to tighten: source specs never name the automation that their event triggers (FEAT-16.SPEC-002 activity recording; FEAT-25.SPEC-004 insights aggregates; FEAT-14.SPEC-003 consent capture; FEAT-04.SPEC-004 disconnect cleanup; FEAT-04.SPEC-005 calendar write). FEAT-16.SPEC-002 additionally excludes support-view logging as a Non-Goal while FEAT-19.SPEC-002 writes through it | c6 sweep (TRIG-1WAY); FEAT-16.SPEC-002 Scope and Non-Goals | resolved — shell re-sweep: every source spec cited in the Trigger Definitions of FEAT-16.SPEC-002, FEAT-25.SPEC-004, FEAT-14.SPEC-003, FEAT-04.SPEC-004 and FEAT-04.SPEC-005 now names that automation; FEAT-16.SPEC-002 lists FEAT-19.SPEC-002 as a trigger and no longer excludes support-view logging |
| SG-12 | [STRUCTURAL-GAP] | FEAT-04.SPEC-003; FEAT-09.SPEC-005; FEAT-18.SPEC-006; FEAT-07.SPEC-005; FEAT-28.SPEC-006; FEAT-30.SPEC-011; FEAT-08.SPEC-012; FEAT-08.SPEC-013 | FEAT-12.SPEC-005; FEAT-15.SPEC-005; FEAT-25.SPEC-004; FEAT-06.SPEC-002 | Integration Inbound Events do not name the automations that cite them as external-event trigger sources (FEAT-04.SPEC-003/FEAT-09.SPEC-005 -> FEAT-12.SPEC-005; FEAT-18.SPEC-006 -> FEAT-15.SPEC-005; FEAT-07.SPEC-005/FEAT-09.SPEC-005/FEAT-28.SPEC-006/FEAT-30.SPEC-011 -> FEAT-25.SPEC-004; FEAT-08.SPEC-012/013 -> FEAT-06.SPEC-002). Adding Inbound Events rows is new content | Automation Trigger Definitions cite the integration; integration Inbound Events tables omit them | resolved — shell re-sweep: every Integration spec cited as an external-event trigger source now names the citing automation in its Inbound Events (0 omissions) |
| SG-13 | [STRUCTURAL-GAP] | FEAT-08.SPEC-011 and 25 other enforcing specs | FEAT-14.SPEC-006/007/008, FEAT-26.SPEC-004 and 13 other Logic/Rule specs | Logic/Rule "Enforced By" tables name enforcing specs that never cite the rule back (38 pairs, full list below). Most material: FEAT-08.SPEC-011 does not cite FEAT-14.SPEC-006/007/008 (XBR-15 authority FEAT-14) nor note that FEAT-26.SPEC-004 runs before it. No contradictory enforcement wording was found in the spot-checked pairs | c7 sweep (ENF-1WAY) | resolved — shell re-sweep: 37 of the 38 Enforced-By pairs now cite the rule back, including FEAT-08.SPEC-011 citing FEAT-14.SPEC-006/007/008 and noting FEAT-26.SPEC-004 runs first; the one remaining rule-to-rule pair is split out as SG-17 |
| SG-14 | [STRUCTURAL-GAP] | FEAT-26.SPEC-001 | Messaging Consent / FEAT-14 | The WhatsApp channel preference is written by FEAT-26.SPEC-001 but its storage entity/field is unstated (FEAT-26 never writes Messaging Consent; owner FEAT-06 or FEAT-14 undecided) | FEAT-26.SPEC-001 Data Model; STAGE.md Pass A batch 8 carry-forward | resolved — FEAT-26.SPEC-001 writes Client.preferred_message_channel (SMS or WhatsApp, default SMS), kept separate from Messaging Consent; FEAT-06.SPEC-005 reads and links to it |
| SG-15 | [STRUCTURAL-GAP] | feature-overview.md Briefs (FEAT-04, FEAT-06, FEAT-16, FEAT-18, FEAT-24) | FEAT-04.SPEC-003/005/006; FEAT-06.SPEC-002; FEAT-16.SPEC-004/005; FEAT-18.SPEC-003; FEAT-24.SPEC-001 | Brief-level discrepancies (Briefs are outside Reconciler edit authority -- Feature Analyst re-spawn): FEAT-06 Referenced Entities omits FEAT-06.SPEC-002 as a Booking reader; FEAT-16 Shared Validation / Internal Dependency Map omit FEAT-16.SPEC-004 governed by FEAT-16.SPEC-005; FEAT-18 Entity-Lifecycle Matrix still names FEAT-18.SPEC-006 as a Subscription.price writer (mechanism is FEAT-18.SPEC-003 only); FEAT-04 matrix attributes busy_periods/last_successful_sync writes to FEAT-04.SPEC-004; FEAT-24 says entry is "from FEAT-13's client list" though FEAT-24.SPEC-001 is the canonical client list; FEAT-10 Brief says the feature never creates a Booking (FEAT-10.SPEC-004 does on a late reschedule) | STAGE.md Pass C carry-forwards; Brief tables | resolved — Briefs corrected: FEAT-06 names SPEC-002 as a Booking reader; FEAT-16 records SPEC-004 governed by SPEC-005; FEAT-18 matrix makes SPEC-003 the only price writer; FEAT-04 matrix attributes busy_periods/last_successful_sync to SPEC-003/SPEC-006; FEAT-24 entry is its own list (SPEC-001); FEAT-10 records SPEC-004 creating a Booking on a late reschedule |
| MS-01 | [MISSING-SPEC] | FEAT-30 (Pro Booking Management) or FEAT-21 (Recurring/Standing Appointments) | -- | No Screen spec lets the Pro set up or manage a client's recurring series / cancel an occurrence at the chair (Pro = Full access for Recurring Appointments). FEAT-21.SPEC-003 Enforced By cites "FEAT-30 -- Pro's own series setup screen" and FEAT-21.SPEC-006 cites "FEAT-30 -- Pro's own series/occurrence management screen", but FEAT-30's Brief lists recurring appointments as a Non-Goal and has no such spec; the map says Recurring Series is created "by FEAT-21 (client, or Pro at the chair via FEAT-30)" | FEAT-21.SPEC-003 line 61, FEAT-21.SPEC-006 line 59; FEAT-30 feature-overview.md Non-Goals; map Recurring Series Lifecycle | resolved — FEAT-21.SPEC-010 (Pro Recurring Series Management) created and listed in the FEAT-21 Brief; FEAT-21.SPEC-003/006 Enforced By repointed to it; FEAT-30.SPEC-001 links to it and FEAT-30 keeps recurring as a Non-Goal. Final pass E-184..E-188 tightened its FEAT-30 entry point to FEAT-30.SPEC-001 |
| SG-16 | [STRUCTURAL-GAP] | FEAT-21.SPEC-010 | FEAT-08.SPEC-004 | Found in the final alignment pass. FEAT-21.SPEC-010 (new) declares "Triggers (outbound)" to FEAT-08.SPEC-004 to tell the client about each occurrence the Pro cancels, but FEAT-08.SPEC-004's Trigger table has no row for a Pro occurrence cancellation from FEAT-21.SPEC-010 (its Pro-side rows arrive via FEAT-30.SPEC-012). Adding a trigger row is new content, outside Reconciler authority | FEAT-21.SPEC-010 Business Rules, Connected Specs, AC-11; FEAT-08.SPEC-004 Trigger (no FEAT-21 source) | carried-forward (Stage 4) — one-way notification trigger link; Stage 4 wires the Pro occurrence-cancel event to FEAT-08.SPEC-004 (directly or through a trigger contract) |
| SG-17 | [STRUCTURAL-GAP] | FEAT-16.SPEC-005 | FEAT-19.SPEC-004 | Residual SG-13 pair. FEAT-19.SPEC-004 (Support Session Scope & Access Rules) lists FEAT-16.SPEC-005 in Enforced By as jointly enforcing the private-notes exclusion, but FEAT-16.SPEC-005 cites XBR-24 without naming FEAT-19.SPEC-004. Both are Logic/Rule specs and their rules agree; only the back-citation is absent (new content) | FEAT-19.SPEC-004 Enforced By; FEAT-16.SPEC-005 Business Rules (XBR-24 line) | carried-forward (Stage 4) — rule-to-rule back-citation; no contradictory enforcement wording |

**SG-13 detail — Enforced-By pairs without a back-reference (rule spec → enforcing spec):** FEAT-01.SPEC-004 → FEAT-01.SPEC-001; FEAT-01.SPEC-004 → FEAT-01.SPEC-006; FEAT-05.SPEC-008 → FEAT-05.SPEC-002; FEAT-05.SPEC-008 → FEAT-05.SPEC-003; FEAT-12.SPEC-007 → FEAT-12.SPEC-002; FEAT-13.SPEC-005 → FEAT-13.SPEC-003; FEAT-13.SPEC-005 → FEAT-13.SPEC-004; FEAT-13.SPEC-006 → FEAT-13.SPEC-001; FEAT-13.SPEC-006 → FEAT-13.SPEC-002; FEAT-14.SPEC-006 → FEAT-08.SPEC-011; FEAT-14.SPEC-007 → FEAT-08.SPEC-011; FEAT-14.SPEC-007 → FEAT-14.SPEC-002; FEAT-14.SPEC-007 → FEAT-14.SPEC-004; FEAT-14.SPEC-008 → FEAT-13.SPEC-002; FEAT-14.SPEC-008 → FEAT-14.SPEC-007; FEAT-15.SPEC-006 → FEAT-04.SPEC-001; FEAT-15.SPEC-006 → FEAT-15.SPEC-005; FEAT-16.SPEC-005 → FEAT-16.SPEC-003; FEAT-17.SPEC-008 → FEAT-17.SPEC-002; FEAT-17.SPEC-008 → FEAT-17.SPEC-007; FEAT-18.SPEC-005 → FEAT-18.SPEC-004; FEAT-19.SPEC-004 → FEAT-13.SPEC-005; FEAT-19.SPEC-004 → FEAT-16.SPEC-005; FEAT-19.SPEC-004 → FEAT-28.SPEC-002; FEAT-20.SPEC-004 → FEAT-03.SPEC-005; FEAT-23.SPEC-002 → FEAT-22.SPEC-001; FEAT-23.SPEC-003 → FEAT-22.SPEC-005; FEAT-23.SPEC-003 → FEAT-28.SPEC-005; FEAT-23.SPEC-003 → FEAT-30.SPEC-011; FEAT-26.SPEC-004 → FEAT-08.SPEC-001; FEAT-26.SPEC-004 → FEAT-08.SPEC-002; FEAT-26.SPEC-004 → FEAT-08.SPEC-004; FEAT-26.SPEC-004 → FEAT-08.SPEC-011; FEAT-27.SPEC-008 → FEAT-07.SPEC-003; FEAT-27.SPEC-009 → FEAT-05.SPEC-008; FEAT-29.SPEC-013 → FEAT-29.SPEC-009; FEAT-30.SPEC-006 → FEAT-03.SPEC-007.

## Edits Applied

Each edit records the file, section, exact before/after text and the conflict-resolution rule applied. Status of every entry: **applied**.

### E-001 — FEAT-11.SPEC-004 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-004-no-show-marking-window-authorization-rules.md`
- **Section:** Defaults and Derivations
- **Before:** | No -- fixed platform-wide, owned by FEAT-12 |
- **After:** | No -- one value for every Pro (7 days per XBR-12), owned by FEAT-12 |
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change.
- **Status:** applied

### E-002 — FEAT-11.SPEC-004 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-004-no-show-marking-window-authorization-rules.md`
- **Section:** Defaults and Derivations
- **Before:** | No -- fixed platform-wide, the same span for every Pro and every booking |
- **After:** | No -- one value for every Pro and every booking (24 hours per XBR-12) |
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change.
- **Status:** applied

### E-003 — FEAT-30.SPEC-006 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-006-pro-booking-action-rules.md`
- **Section:** Defaults and Derivations
- **Before:** and (appointment start time − platform parameter: `deposit-request-hold-appointment-cutoff-hours`) | Computed by FEAT-03.SPEC-007 whenever FEAT-30.SPEC-010 creates a Pro-created deposit request | No -- fixed platform-wide |
- **After:** and (appointment start time − platform parameter: `deposit-request-hold-appointment-cutoff-hours`) | Computed by FEAT-03.SPEC-007 whenever FEAT-30.SPEC-010 creates a Pro-created deposit request | No -- one value for every Pro and every booking |
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change.
- **Status:** applied

### E-004 — FEAT-29.SPEC-014 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-014-sign-in-code-notification.md`
- **Section:** Content Definition (Placeholders)
- **Before:** | 10 | Never empty -- a fixed, always-present platform value |
- **After:** | 10 | Never empty -- always present, one value for every Pro |
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change.
- **Status:** applied

### E-005 — FEAT-29.SPEC-017 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-017-account-closure-deletion-notifications.md`
- **Section:** Content Definition (Placeholders)
- **Before:** | 30 | Never empty -- a fixed, always-present platform value |
- **After:** | 30 | Never empty -- always present, one value for every Pro |
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change.
- **Status:** applied

### E-006 — FEAT-18.SPEC-007 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-007-subscription-billing-notifications.md`
- **Section:** Content Definition (Placeholders)
- **Before:** | $39/month | Never empty -- fixed platform-wide value |
- **After:** | $39/month (illustrative only) | Never empty -- one price for every Pro |
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change. Example value labelled illustrative because the price itself is undecided (see platform-parameters.md).
- **Status:** applied

### E-007 — FEAT-20.SPEC-008 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-008-waitlist-opening-notification.md`
- **Section:** Content Definition (Placeholders)
- **Before:** | Platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- a fixed platform-wide value |
- **After:** | platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- one value for every Pro |
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Capitalized marker lowercased and "fixed platform-wide" phrasing removed.
- **Status:** applied

### E-008 — FEAT-20.SPEC-009 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-009-waitlist-expiry-notification.md`
- **Section:** Content Definition (Placeholders)
- **Before:** | Platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- a fixed platform-wide value |
- **After:** | platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- one value for every Pro |
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Capitalized marker lowercased and "fixed platform-wide" phrasing removed.
- **Status:** applied

### E-009 — FEAT-08.SPEC-007 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-007-reminder-scheduling-timing-window-enforcement.md`
- **Section:** Business Rules
- **Before:** (platform parameter: `reminder-window-start-hour` / `reminder-window-end-hour`)
- **After:** (platform parameter: `reminder-window-start-hour` / platform parameter: `reminder-window-end-hour`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Second slug lacked the marker prefix.
- **Status:** applied

### E-010 — FEAT-08.SPEC-007 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-007-reminder-scheduling-timing-window-enforcement.md`
- **Section:** Scope and Non-Goals
- **Before:** the lead time and window are fixed platform-wide values (XBR-16), not Pro-configurable preferences
- **After:** the lead time and window are single values for every Pro (XBR-16; platform parameter: `reminder-lead-time-days`, platform parameter: `reminder-window-start-hour`, platform parameter: `reminder-window-end-hour`), not Pro-configurable preferences
- **Rationale:** Check 14 lint: "fixed platform-wide" phrasing without the marker (alignment issue) rewritten; the marker in the same cell already names the parameter; no behavior change.
- **Status:** applied

### E-011 — FEAT-08.SPEC-007 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-007-reminder-scheduling-timing-window-enforcement.md`
- **Section:** Overview
- **Before:** keeps every reminder inside the platform's fixed 8am--9pm daytime send window in the Pro's timezone
- **After:** keeps every reminder inside the daytime send window (roughly 8am--9pm per XBR-16; platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`) in the Pro's timezone
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-012 — FEAT-22.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-003-balance-amount-eligibility-rules.md`
- **Section:** Business Rules
- **Before:** - The `minimum-chargeable-deposit` platform parameter is reused verbatim
- **After:** - The marker platform parameter: `minimum-chargeable-deposit` is reused verbatim
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Reuse confirmed: one registry row covers both the deposit and balance floor.
- **Status:** applied

### E-013 — FEAT-22.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-003-balance-amount-eligibility-rules.md`
- **Section:** Business Rules
- **Before:** Reconciliation (Pass D) should confirm this reuse against the parameter registry rather than treat it as a naming mismatch.
- **After:** Reconciliation (Pass D) confirmed this reuse: the registry carries one row for this slug covering both floors.
- **Rationale:** Check 14 slug consolidation: deliberate reuse confirmed; no second slug minted.
- **Status:** applied

### E-014 — FEAT-18.SPEC-005 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md`
- **Section:** Enforced By
- **Before:** writing the new platform-parameter price value into
- **After:** writing the new value of platform parameter: `subscription-price` into
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Hyphenated near-miss "platform-parameter price value".
- **Status:** applied

### E-015 — FEAT-18.SPEC-005 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md`
- **Section:** Defaults and Derivations
- **Before:** re-written to the new platform-parameter value on
- **After:** re-written to the new value of platform parameter: `subscription-price` on
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-016 — FEAT-18.SPEC-005 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md`
- **Section:** Acceptance Criteria
- **Before:** then its price is set to the single platform-set tier and
- **After:** then its price is set to the single tier (platform parameter: `subscription-price`) and
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Platform-set value phrasing without marker.
- **Status:** applied

### E-017 — FEAT-18.SPEC-005 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md`
- **Section:** Acceptance Criteria
- **Before:** price field is updated to the new platform-set value by FEAT-18.SPEC-003
- **After:** price field is updated to the new value of platform parameter: `subscription-price` by FEAT-18.SPEC-003
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-018 — FEAT-18.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-003-subscription-renewal-payment-failure-processing.md`
- **Section:** Scope and Non-Goals
- **Before:** price field to the new platform-parameter value on
- **After:** price field to the new value of platform parameter: `subscription-price` on
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-019 — FEAT-18.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-003-subscription-renewal-payment-failure-processing.md`
- **Section:** Trigger Definition
- **Before:** | Subscription reference, new platform-parameter price value, effective_date |
- **After:** | Subscription reference, new value of platform parameter: `subscription-price`, effective_date |
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-020 — FEAT-18.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-003-subscription-renewal-payment-failure-processing.md`
- **Section:** Data Model
- **Before:** price (updated to the new platform-parameter value on
- **After:** price (updated to the new value of platform parameter: `subscription-price` on
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-021 — FEAT-18.SPEC-005 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md`
- **Section:** Overview
- **Before:** the 7-day grace threshold, cancellation-at-period-end timing, the 30-day price-change notice rule
- **After:** the 7-day grace threshold (platform parameter: `subscription-payment-failure-grace-period-days`), cancellation-at-period-end timing, the 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Concrete platform policy numbers now carry their markers.
- **Status:** applied

### E-022 — FEAT-18.SPEC-005 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-005-subscription-billing-rules.md`
- **Section:** Scope and Non-Goals
- **Before:** - The 30-day price-change notice rule
- **After:** - The 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-023 — FEAT-18.SPEC-007 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-007-subscription-billing-notifications.md`
- **Section:** Scope and Non-Goals
- **Before:** - The price-change notice, sent at least 30 days before a price change applies
- **After:** - The price-change notice, sent at least 30 days (platform parameter: `subscription-price-change-notice-days`) before a price change applies
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-024 — FEAT-29.SPEC-006 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-006-session-device-management.md`
- **Section:** Scope and Non-Goals
- **Before:** - Expiring a device after 30 days of inactivity
- **After:** - Expiring a device after 30 days of inactivity (platform parameter: `session-inactivity-expiry-days`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-025 — FEAT-11.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-003-no-show-mark-undo.md`
- **Section:** Overview
- **Before:** **Purpose:** Within the fixed 24-hour grace window,
- **After:** **Purpose:** Within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`),
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-026 — FEAT-11.SPEC-001 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-001-no-show-mark-undo-prompt.md`
- **Section:** Overview
- **Before:** or -- within the 24-hour grace window -- undo
- **After:** or -- within the 24-hour grace window (platform parameter: `no-show-undo-grace-window-hours`) -- undo
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-027 — FEAT-11.SPEC-004 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-004-no-show-marking-window-authorization-rules.md`
- **Section:** Overview
- **Before:** and the fixed 24-hour undo grace window, shared
- **After:** and the 24-hour undo grace window (platform parameter: `no-show-undo-grace-window-hours`), shared
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-028 — FEAT-03.SPEC-007 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md`
- **Section:** Overview
- **Before:** holding it up to 24 hours or until 2 hours before the appointment (whichever comes first)
- **After:** holding it up to 24 hours (platform parameter: `deposit-request-hold-max-hours`) or until 2 hours before the appointment (platform parameter: `deposit-request-hold-appointment-cutoff-hours`), whichever comes first
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-029 — FEAT-03.SPEC-002 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-002-slot-hold-creation-checkout-reservation.md`
- **Section:** Scope and Non-Goals
- **Before:** (longer-lived, up to 24 hours)
- **After:** (longer-lived, up to platform parameter: `deposit-request-hold-max-hours`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-030 — FEAT-03.SPEC-003 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-003-slot-hold-expiration.md`
- **Section:** Scope and Non-Goals
- **Before:** (up to 24 hours or 2 hours before the appointment)
- **After:** (up to platform parameter: `deposit-request-hold-max-hours` or platform parameter: `deposit-request-hold-appointment-cutoff-hours` before the appointment)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-031 — FEAT-05.SPEC-001 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-001-public-booking-page-landing-service-list.md`
- **Section:** Business Rules
- **Before:** from its previous name for at least 12 months (XBR-27)
- **After:** from its previous name for at least 12 months (platform parameter: `booking-link-forward-window-months`) (XBR-27)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. Carried-forward Pass B item: FEAT-05 adopts the FEAT-27 slug.
- **Status:** applied

### E-032 — FEAT-05.SPEC-001 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-001-public-booking-page-landing-service-list.md`
- **Section:** Entry Points
- **Before:** renamed away from within the last 12 months (XBR-27)
- **After:** renamed away from within the last 12 months (platform parameter: `booking-link-forward-window-months`) (XBR-27)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-033 — FEAT-05.SPEC-008 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-008-booking-page-availability-gate.md`
- **Section:** Scope and Non-Goals
- **Before:** forwarding a renamed link for at least 12 months
- **After:** forwarding a renamed link for at least 12 months (platform parameter: `booking-link-forward-window-months`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-034 — FEAT-05.SPEC-008 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-008-booking-page-availability-gate.md`
- **Section:** Field Validation Rules
- **Before:** (within 12 months of the rename)
- **After:** (within 12 months of the rename, platform parameter: `booking-link-forward-window-months`)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-035 — FEAT-05.SPEC-008 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-008-booking-page-availability-gate.md`
- **Section:** Business Rules
- **Before:** keeps forwarding from the old name for at least 12 months; a closed
- **After:** keeps forwarding from the old name for at least 12 months (platform parameter: `booking-link-forward-window-months`); a closed
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change.
- **Status:** applied

### E-036 — FEAT-12.SPEC-004 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-004-auto-completion-sweep.md`
- **Section:** Overview
- **Before:** marks a Booking Completed 7 days after its appointment time
- **After:** marks a Booking Completed 7 days (platform parameter: `booking-auto-completion-window-days`) after its appointment time
- **Rationale:** Check 14 + carried-forward FEAT-11/FEAT-12 alignment: FEAT-12 (XBR-12 authority) keeps the concrete 7 days required by its Pass C review; the marker FEAT-11 already uses is added beside it so both features name the same value the same way and the registry row is shared (Rule 3 tie on priority resolved by XBR-12 authority).
- **Status:** applied

### E-037 — FEAT-12.SPEC-004 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-004-auto-completion-sweep.md`
- **Section:** Business Rules
- **Before:** - The completion window is 7 days after `start_time`, per XBR-12 and FEAT-12.SPEC-006.
- **After:** - The completion window is 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time`, per XBR-12 and FEAT-12.SPEC-006.
- **Rationale:** Same as above (FEAT-11/FEAT-12 auto-completion window alignment).
- **Status:** applied

### E-038 — FEAT-12.SPEC-004 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-004-auto-completion-sweep.md`
- **Section:** Business Rules
- **Before:** (it is a platform-wide parameter, not a per-account setting)
- **After:** (it is one value for every Pro (platform parameter: `booking-auto-completion-window-days`), not a per-account setting)
- **Rationale:** Check 14 near-miss/unmarked platform value (alignment issue): normalized to the exact marker shape `platform parameter: `slug`` so the registry and Gate A sweep see every site; no behavior change. "platform-wide parameter" near-miss phrasing.
- **Status:** applied

### E-039 — FEAT-12.SPEC-006 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-006-booking-completion-rules.md`
- **Section:** Cross-Field Rules
- **Before:** `Awaiting Outcome` 7 days after `start_time` with no Pro action
- **After:** `Awaiting Outcome` 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time` with no Pro action
- **Rationale:** FEAT-11/FEAT-12 auto-completion window alignment (see FEAT-12.SPEC-004 entry).
- **Status:** applied

### E-040 — FEAT-12.SPEC-006 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-006-booking-completion-rules.md`
- **Section:** Business Rules
- **Before:** automatically 7 days after `start_time` (XBR-12).
- **After:** automatically 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time` (XBR-12).
- **Rationale:** FEAT-11/FEAT-12 auto-completion window alignment (see FEAT-12.SPEC-004 entry).
- **Status:** applied

### E-041 — FEAT-02.SPEC-001 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-001-working-hours-buffer-notice-horizon-setup.md`
- **Section:** Connected Specs
- **Before:** | FEAT-17 (Manual Time Blocking) |
- **After:** | FEAT-17.SPEC-001 (Create/Edit Time Block), FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) -- within FEAT-17 (Manual Time Blocking) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-042 — FEAT-03.SPEC-003 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-003-slot-hold-expiration.md`
- **Section:** Connected Specs
- **Before:** | FEAT-05 (Public Booking Page & Booking Flow) |
- **After:** | FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout), FEAT-05.SPEC-002 (Slot Selection) -- within FEAT-05 (Public Booking Page & Booking Flow) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-043 — FEAT-07.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-002-deposit-capture-booking-confirmation.md`
- **Section:** Connected Specs
- **Before:** | FEAT-08 (Automated Booking Messaging) |
- **After:** | FEAT-08.SPEC-001 (Booking Confirmation Message), FEAT-08.SPEC-005 (Pro Booking Activity Notification), FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) -- within FEAT-08 (Automated Booking Messaging) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-044 — FEAT-07.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-002-deposit-capture-booking-confirmation.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-045 — FEAT-07.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-002-deposit-capture-booking-confirmation.md`
- **Section:** Connected Specs
- **Before:** | FEAT-30 (Pro Booking Management) |
- **After:** | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) -- within FEAT-30 (Pro Booking Management) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-046 — FEAT-08.SPEC-005 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-005-pro-booking-activity-notification.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-047 — FEAT-08.SPEC-009 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-009-message-delivery-retry-fallback.md`
- **Section:** Connected Specs
- **Before:** | FEAT-12 (Pro Daily Schedule Dashboard) |
- **After:** | FEAT-12.SPEC-005 (Attention Flag Aggregation) -- within FEAT-12 (Pro Daily Schedule Dashboard) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-048 — FEAT-08.SPEC-009 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-009-message-delivery-retry-fallback.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-049 — FEAT-10.SPEC-004 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-004-booking-update-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-050 — FEAT-10.SPEC-004 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-004-booking-update-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-20 (Waitlist for Cancelled Slots) |
- **After:** | FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-051 — FEAT-10.SPEC-004 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-004-booking-update-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-04 (Two-Way Calendar Sync) |
- **After:** | FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-052 — FEAT-11.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-002-no-show-marking-deposit-forfeiture.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-053 — FEAT-11.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-002-no-show-marking-deposit-forfeiture.md`
- **Section:** Connected Specs
- **Before:** | FEAT-25 (Booking & Revenue Insights) |
- **After:** | FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-054 — FEAT-11.SPEC-003 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-003-no-show-mark-undo.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-055 — FEAT-13.SPEC-004 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-004-client-deletion-execution.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-056 — FEAT-18.SPEC-006 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-006-subscription-billing-integration.md`
- **Section:** Connected Specs
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) |
- **After:** | FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) -- within FEAT-15 (Pro Onboarding & Setup Wizard) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-057 — FEAT-28.SPEC-003 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-003-payout-account-status-processing.md`
- **Section:** Connected Specs
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) |
- **After:** | FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) -- within FEAT-15 (Pro Onboarding & Setup Wizard) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-058 — FEAT-28.SPEC-003 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-003-payout-account-status-processing.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-059 — FEAT-30.SPEC-007 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-007-pro-cancel-reschedule-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-060 — FEAT-30.SPEC-007 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-007-pro-cancel-reschedule-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-20 (Waitlist for Cancelled Slots) |
- **After:** | FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-061 — FEAT-30.SPEC-007 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-007-pro-cancel-reschedule-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-04 (Two-Way Calendar Sync) |
- **After:** | FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-062 — FEAT-30.SPEC-008 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-008-bulk-cancellation-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-16 (Booking & Payment Activity Record) |
- **After:** | FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-063 — FEAT-30.SPEC-008 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-008-bulk-cancellation-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-20 (Waitlist for Cancelled Slots) |
- **After:** | FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-064 — FEAT-30.SPEC-008 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-008-bulk-cancellation-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-04 (Two-Way Calendar Sync) |
- **After:** | FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-065 — FEAT-05.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-002-slot-selection.md`
- **Section:** Connected Specs
- **Before:** | FEAT-03 (Real-Time Slot Availability Engine) |
- **After:** | FEAT-03.SPEC-001 (Slot Availability Computation) -- within FEAT-03 (Real-Time Slot Availability Engine) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-066 — FEAT-10.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-10-client-initiated-cancel-reschedule/FEAT-10.SPEC-002-reschedule-select-new-time.md`
- **Section:** Connected Specs
- **Before:** | FEAT-03 (Real-Time Slot Availability Engine) |
- **After:** | FEAT-03.SPEC-001 (Slot Availability Computation) -- within FEAT-03 (Real-Time Slot Availability Engine) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-067 — FEAT-16.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-001-booking-activity-timeline.md`
- **Section:** Connected Specs
- **Before:** | FEAT-19 (Platform Support Read-Only Access -- not yet specified in this run) | Navigation (inbound)
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry), FEAT-19.SPEC-002 (Support View Logging), FEAT-19.SPEC-003 (Support Access Log) -- within FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound)
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change. Stale "not yet specified in this run" note removed (FEAT-19 specs now exist).
- **Status:** applied

### E-068 — FEAT-03.SPEC-001 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-001-slot-availability-computation.md`
- **Section:** Trigger Definition
- **Before:** | FEAT-05.SPEC-001-equivalent (Public Booking Page & Booking Flow, service selection step) |
- **After:** | FEAT-05.SPEC-002 (Slot Selection), reached when the client picks a service on FEAT-05.SPEC-001 (Public Booking Page & Booking Flow, service selection step) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-069 — FEAT-03.SPEC-001 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-001-slot-availability-computation.md`
- **Section:** Trigger Definition
- **Before:** | FEAT-10.SPEC (Client-Initiated Cancel/Reschedule) |
- **After:** | FEAT-10.SPEC-002 (Reschedule -- Select New Time) (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-070 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking confirmed | FEAT-05.SPEC (Public Booking Page & Booking Flow) |
- **After:** | Booking confirmed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation), completing the FEAT-05 (Public Booking Page & Booking Flow) checkout |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-071 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking confirmed (Pro-initiated) | FEAT-30.SPEC (Pro Booking Management) |
- **After:** | Booking confirmed (Pro-initiated) | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold), confirmed through FEAT-07.SPEC-002 (Pro Booking Management) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-072 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking rescheduled (client-initiated) | FEAT-10.SPEC (Client-Initiated Cancel/Reschedule) |
- **After:** | Booking rescheduled (client-initiated) | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-073 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking rescheduled (Pro-initiated) | FEAT-30.SPEC (Pro Booking Management) |
- **After:** | Booking rescheduled (Pro-initiated) | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) (Pro Booking Management) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-074 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking cancelled (client-initiated) | FEAT-10.SPEC (Client-Initiated Cancel/Reschedule) |
- **After:** | Booking cancelled (client-initiated) | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-075 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking cancelled (Pro-initiated) | FEAT-30.SPEC (Pro Booking Management) |
- **After:** | Booking cancelled (Pro-initiated) | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit) (Pro Booking Management) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-076 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | Booking marked no-show | FEAT-11.SPEC (No-Show Marking & Deposit Forfeiture) |
- **After:** | Booking marked no-show | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-077 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | FEAT-05.SPEC (Public Booking Page & Booking Flow) | Triggered by (inbound) |
- **After:** | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) -- within FEAT-05 (Public Booking Page & Booking Flow) checkout | Triggered by (inbound) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-078 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | FEAT-10.SPEC (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) |
- **After:** | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-079 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | FEAT-11.SPEC (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) |
- **After:** | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-080 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition / Connected Specs
- **Before:** | FEAT-30.SPEC (Pro Booking Management) | Triggered by (inbound) |
- **After:** | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), FEAT-30.SPEC-008 (Bulk Cancellation Commit), FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) (Pro Booking Management) | Triggered by (inbound) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change.
- **Status:** applied

### E-081 — FEAT-04.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-001-calendar-connection-setup.md`
- **Section:** Entry Points
- **Before:** | FEAT-15.SPEC (Pro Onboarding & Setup Wizard, calendar step) |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard, calendar step) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change. Rule 2: FEAT-15.SPEC-001 Navigation Out declares this destination.
- **Status:** applied

### E-082 — FEAT-06.SPEC-001 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-001-access-link-request.md`
- **Section:** Entry Points
- **Before:** | FEAT-05.SPEC-* (Public Booking Page & Booking Flow) |
- **After:** | FEAT-05.SPEC-003 (Client Details & Consent) (Public Booking Page & Booking Flow) |
- **Rationale:** Check 2 malformed/feature-level reference (alignment): bare "FEAT-NN.SPEC" or "-equivalent" reference replaced by the exact existing spec ID(s) that perform the described event (carried-forward Pass B/C item); no behavior change. Wildcard ID replaced by the only FEAT-05 screen that captures the phone number; see STRUCTURAL-GAP SG-05 (FEAT-05.SPEC-003 declares no matching navigation).
- **Status:** applied

### E-083 — FEAT-19.SPEC-003 (Intra-feature references)

- **File:** `.n2b/specifications/FEAT-19-platform-support-read-only-access/FEAT-19.SPEC-003-support-access-log.md`
- **Section:** Scope and Non-Goals
- **Before:** FEAT-16.SPEC-002/SPEC-005's own lifecycle
- **After:** FEAT-16.SPEC-002/FEAT-16.SPEC-005's own lifecycle
- **Rationale:** Check 3 (alignment): abbreviated "SPEC-005" resolved as FEAT-19.SPEC-005 (nonexistent) by an intra-feature reader; written as the full FEAT-16.SPEC-005 ID it means.
- **Status:** applied

### E-084 — FEAT-01.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-002-add-service.md`
- **Section:** Connected Specs
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard, Setup Wizard step) |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard, Setup Wizard step) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-085 — FEAT-02.SPEC-001 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-001-working-hours-buffer-notice-horizon-setup.md`
- **Section:** Connected Specs
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-086 — FEAT-04.SPEC-001 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-001-calendar-connection-setup.md`
- **Section:** Connected Specs
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-087 — FEAT-28.SPEC-001 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-001-payout-account-connection.md`
- **Section:** Connected Specs
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-088 — FEAT-03.SPEC-007 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md`
- **Section:** Connected Specs
- **Before:** | FEAT-30 (Pro Booking Management) |
- **After:** | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) -- within FEAT-30 (Pro Booking Management) |
- **Rationale:** Check 6 bidirectional trigger (alignment): source named the counterpart only at feature level; tightened to the exact spec ID(s) whose Trigger Definition already cites this spec, so both ends name each other. Original feature-level wording retained. Rule 2 analogue (declared link is authoritative); no behavior change.
- **Status:** applied

### E-089 — FEAT-09.SPEC-005 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-005-automatic-deposit-refund.md`
- **Section:** Connected Specs
- **Before:** | FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) |
- **After:** | FEAT-12.SPEC-005 (Attention Flag Aggregation) -- within FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) |
- **Rationale:** Check 6 bidirectional trigger (alignment): FEAT-12.SPEC-005 Trigger Definition cites this spec; the feature-level row is tightened to that spec ID. No behavior change.
- **Status:** applied

### E-090 — FEAT-05.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-002-slot-selection.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-07.SPEC-001)
- **After:** | FEAT-07.SPEC-001 (Deposit Payment) | The checkout hold expires on the deposit payment screen (with or without a prior decline) | Same chosen service; slot list refreshed; plain expiry message shown |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-07.SPEC-001 declares outbound navigation to this screen ("The checkout hold expires on the deposit payment screen (with or without a prior decline)"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-091 — FEAT-05.SPEC-005 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-005-booking-confirmation.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-07.SPEC-001)
- **After:** | FEAT-07.SPEC-001 (Deposit Payment) | Deposit payment succeeds and the Booking reaches Confirmed | The confirmed Booking reference |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-07.SPEC-001 declares outbound navigation to this screen ("Deposit payment succeeds and the Booking reaches Confirmed"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-092 — FEAT-06.SPEC-004 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-004-booking-detail-via-manage-link.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-10.SPEC-001)
- **After:** | FEAT-10.SPEC-001 (Cancel Booking) | Client completes a cancellation, taps "Keep my booking", or taps back | The same Booking reference, showing its current state |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-10.SPEC-001 declares outbound navigation to this screen ("Client completes a cancellation, taps "Keep my booking", or taps back"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-093 — FEAT-06.SPEC-004 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-004-booking-detail-via-manage-link.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-10.SPEC-003)
- **After:** | FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client confirms an outside-window reschedule successfully | The same Booking reference with its new start_time |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-10.SPEC-003 declares outbound navigation to this screen ("Client confirms an outside-window reschedule successfully"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-094 — FEAT-28.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-001-payout-account-connection.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-15.SPEC-003)
- **After:** | FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Talia taps "Finish verifying" on the go-live screen | None -- the existing Payout Account and its current verification status load fresh |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-15.SPEC-003 declares outbound navigation to this screen ("Talia taps "Finish verifying" on the go-live screen"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-095 — FEAT-17.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-003-time-block-conflict-review.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-17.SPEC-001)
- **After:** | FEAT-17.SPEC-001 (Create/Edit Time Block) | A save finds one or more conflicting confirmed bookings | The pending block values and the conflicting Booking references (via FEAT-17.SPEC-004) |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-17.SPEC-001 declares outbound navigation to this screen ("A save finds one or more conflicting confirmed bookings"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-096 — FEAT-17.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-002-manage-time-blocks.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-17.SPEC-003)
- **After:** | FEAT-17.SPEC-003 (Time Block Conflict Review) | Conflict review confirms with every remaining booking kept as an exception | None -- list reloads to reflect the saved block |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-17.SPEC-003 declares outbound navigation to this screen ("Conflict review confirms with every remaining booking kept as an exception"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-097 — FEAT-30.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-002-reschedule-booking-pro-initiated.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-17.SPEC-003)
- **After:** | FEAT-17.SPEC-003 (Time Block Conflict Review) | Conflict review confirms with a booking chosen "Reschedule" | The conflicting Booking reference |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-17.SPEC-003 declares outbound navigation to this screen ("Conflict review confirms with a booking chosen "Reschedule""); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-098 — FEAT-01.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-003-edit-service.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-19.SPEC-001)
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Services entry during an active support session | The Pro account under review; screen renders read-only for Support with no action controls |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-19.SPEC-001 declares outbound navigation to this screen ("Support taps the Services entry during an active support session"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-099 — FEAT-05.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-001-public-booking-page-landing-service-list.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-19.SPEC-001)
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Booking Page Preview entry during an active support session | Preview mode flag -- identical screens render read-only, no real payment is taken |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-19.SPEC-001 declares outbound navigation to this screen ("Support taps the Booking Page Preview entry during an active support session"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-100 — FEAT-12.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-001-todays-upcoming-schedule.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-19.SPEC-001)
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-19.SPEC-001 declares outbound navigation to this screen ("Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008)"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-101 — FEAT-12.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-002-attention-list.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-19.SPEC-001)
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-19.SPEC-001 declares outbound navigation to this screen ("Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008)"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-102 — FEAT-12.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-003-past-bookings-browse.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-19.SPEC-001)
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-19.SPEC-001 declares outbound navigation to this screen ("Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008)"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-103 — FEAT-16.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-001-booking-activity-timeline.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-19.SPEC-003)
- **After:** | FEAT-19.SPEC-003 (Support Access Log) | Support taps an access-log entry that references a booking | The referenced Booking identifier; Support read-only variant |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-19.SPEC-003 declares outbound navigation to this screen ("Support taps an access-log entry that references a booking"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-104 — FEAT-06.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-001-access-link-request.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-20.SPEC-001)
- **After:** | FEAT-20.SPEC-001 (Join Waitlist) | Client taps the access-link link on the waitlist Confirmed state | None -- form starts empty |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-20.SPEC-001 declares outbound navigation to this screen ("Client taps the access-link link on the waitlist Confirmed state"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-105 — FEAT-18.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-002-billing-subscription-management-screen.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-27.SPEC-004)
- **After:** | FEAT-27.SPEC-004 (Pause Bookings) | Talia taps "Resolve it in Billing" on the Pause Bookings screen while a subscription lapse pause is in force | The Pro's account reference |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-27.SPEC-004 declares outbound navigation to this screen ("Talia taps "Resolve it in Billing" on the Pause Bookings screen while a subscription lapse pause is in force"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-106 — FEAT-29.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-001-sign-in-screen.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-29.SPEC-003)
- **After:** | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Talia confirms "Sign out everywhere" | None -- every session has ended; a fresh sign-in starts |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-29.SPEC-003 declares outbound navigation to this screen ("Talia confirms "Sign out everywhere""); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-107 — FEAT-30.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-001-cancel-booking-pro-initiated.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-30.SPEC-005)
- **After:** | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Talia taps "View" on a booking that failed in a bulk cancellation | The failed Booking reference |
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2): FEAT-30.SPEC-005 declares outbound navigation to this screen ("Talia taps "View" on a booking that failed in a bulk cancellation"); the outbound-declaring spec is authoritative, so the destination Entry Points table gains the matching row. Context carried is taken from the source row.
- **Status:** applied

### E-108 — FEAT-06.SPEC-004 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-004-booking-detail-via-manage-link.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-10.SPEC-006)
- **After:** | FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) | Client taps "Manage my booking" in the cancellation/reschedule notice | The affected Booking reference (the original booking for a cancellation, the new booking for a late reschedule) |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-10.SPEC-006's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-109 — FEAT-15.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-003-go-live-preview-booking-link-hand-over.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-15.SPEC-008)
- **After:** | FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | Talia taps "View my link" in the go-live welcome confirmation | None -- this Pro's own go-live state and booking link load fresh |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-15.SPEC-008's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-110 — FEAT-05.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-002-slot-selection.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-20.SPEC-008)
- **After:** | FEAT-20.SPEC-008 (Waitlist Opening Notification) | Client taps the claim link in a waitlist opening notice within the claim window | The matched service and opened slot; priority claim context |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-20.SPEC-008's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-111 — FEAT-05.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-001-public-booking-page-landing-service-list.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-20.SPEC-009)
- **After:** | FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "Join the waitlist again" in a waitlist expiry notice | None -- page loads for this Pro's booking_link_name |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-20.SPEC-009's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-112 — FEAT-06.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-001-access-link-request.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-20.SPEC-009)
- **After:** | FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "View my waitlists" when a fresh access link could not be issued at send time | None -- form starts empty so the client can request a new link |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-20.SPEC-009's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-113 — FEAT-20.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-20-waitlist-for-cancelled-slots/FEAT-20.SPEC-002-my-waitlists.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-20.SPEC-009)
- **After:** | FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "View my waitlists" in a waitlist expiry notice | The client's access-link identity; list loads that client's own entries with this Pro |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-20.SPEC-009's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-114 — FEAT-21.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-002-my-recurring-series.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-21.SPEC-007)
- **After:** | FEAT-21.SPEC-007 (Occurrence Generated Notification) | Client taps "View my series" in an occurrence-generated notice | The client's series reference |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-21.SPEC-007's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-115 — FEAT-28.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-002-payout-status-money-dashboard.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-28.SPEC-007)
- **After:** | FEAT-28.SPEC-007 (Payout Status Notification) | Talia taps "View payouts" or "Resolve now" in a payout status notification | None -- opens to the default view; for "Resolve now" the action-required item is surfaced for FEAT-28.SPEC-006's resolution flow |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-28.SPEC-007's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-116 — FEAT-29.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-003-account-sign-in-settings-screen.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-29.SPEC-015)
- **After:** | FEAT-29.SPEC-015 (New-Device Sign-In Alert) | Talia taps "Manage sign-in" in a new-device sign-in alert email | None -- opens at the devices section |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-29.SPEC-015's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-117 — FEAT-29.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-003-account-sign-in-settings-screen.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-29.SPEC-016)
- **After:** | FEAT-29.SPEC-016 (Contact-Change Confirmation Notification) | Talia taps "Manage sign-in" in a contact-change confirmation email | None -- opens at the sign-in contacts section |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-29.SPEC-016's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-118 — FEAT-29.SPEC-005 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-29-pro-sign-in-account-lifecycle/FEAT-29.SPEC-005-account-closure-reopening-screen.md`
- **Section:** Entry Points
- **Before:** (no row for FEAT-29.SPEC-017)
- **After:** | FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) | Talia taps "Manage account" in an account closure/deletion notice email | None -- fresh review of the account's closure state |
- **Rationale:** Check 5 notification CTA (alignment, Rule 2): FEAT-29.SPEC-017's CTA deep-links to this screen, which is navigation; the destination Entry Points table gains the matching row.
- **Status:** applied

### E-119 — FEAT-01.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-01-service-pricing-management/FEAT-01.SPEC-002-add-service.md`
- **Section:** Entry Points
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) | Pro reaches the services step
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard) | Pro reaches the services step
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-120 — FEAT-02.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-02-availability-working-hours-setup/FEAT-02.SPEC-001-working-hours-buffer-notice-horizon-setup.md`
- **Section:** Entry Points
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard) -- setup step: hours |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard) -- setup step: hours |
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-121 — FEAT-28.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-001-payout-account-connection.md`
- **Section:** Entry Points
- **Before:** | FEAT-15 (Pro Onboarding & Setup Wizard), setup step: getting paid |
- **After:** | FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard), setup step: getting paid |
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-122 — FEAT-30.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-003-goodwill-deposit-refund.md`
- **Section:** Entry Points
- **Before:** | FEAT-16 (Booking & Payment Activity Record) | Talia decides
- **After:** | FEAT-16.SPEC-001 (Booking Activity Timeline) (Booking & Payment Activity Record) | Talia decides
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-123 — FEAT-30.SPEC-005 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-005-cancel-several-bookings-at-once.md`
- **Section:** Entry Points
- **Before:** | FEAT-17 (Manual Time Blocking) | Talia's new time block
- **After:** | FEAT-17.SPEC-003 (Time Block Conflict Review) (Manual Time Blocking) | Talia's new time block
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-124 — FEAT-17.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-17-manual-time-blocking/FEAT-17.SPEC-002-manage-time-blocks.md`
- **Section:** Entry Points
- **Before:** | FEAT-19 (Platform Support Read-Only Access) | Support opens
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-125 — FEAT-18.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-002-billing-subscription-management-screen.md`
- **Section:** Entry Points
- **Before:** | FEAT-19 (Platform Support Read-Only Access) | Support opens
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-126 — FEAT-28.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-28-payout-account-connection-payout-visibility/FEAT-28.SPEC-002-payout-status-money-dashboard.md`
- **Section:** Entry Points
- **Before:** | FEAT-19 (Platform Support Read-Only Access) | Support opens
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-127 — FEAT-16.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-16-booking-payment-activity-record/FEAT-16.SPEC-001-booking-activity-timeline.md`
- **Section:** Entry Points
- **Before:** | FEAT-19 (Platform Support Read-Only Access -- not yet specified in this run) | Support opens
- **After:** | FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination. Stale "not yet specified in this run" note removed.
- **Status:** applied

### E-128 — FEAT-13.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-001-client-record-detail.md`
- **Section:** Entry Points
- **Before:** | FEAT-24 (Client List Search & Filter, v1) |
- **After:** | FEAT-24.SPEC-001 (Client Search & Filter) (Client List Search & Filter, v1) |
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-129 — FEAT-05.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-001-public-booking-page-landing-service-list.md`
- **Section:** Entry Points
- **Before:** | FEAT-27 (Pro Profile & Booking Page Settings) | Pro taps "preview"
- **After:** | FEAT-27.SPEC-001 (Profile & Booking Page Settings) (Pro Profile & Booking Page Settings) | Pro taps "preview"
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-130 — FEAT-18.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-002-billing-subscription-management-screen.md`
- **Section:** Entry Points
- **Before:** | FEAT-29 (Pro Sign-In & Account Lifecycle, account settings) |
- **After:** | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) (Pro Sign-In & Account Lifecycle, account settings) |
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-131 — FEAT-12.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-001-todays-upcoming-schedule.md`
- **Section:** Entry Points
- **Before:** | FEAT-29 (Pro Sign-In & Account Lifecycle) | Pro signs in / opens the app |
- **After:** | FEAT-29.SPEC-001 (Sign-In Screen) / FEAT-29.SPEC-002 (Account Recovery Screen) (Pro Sign-In & Account Lifecycle) | Pro signs in / opens the app |
- **Rationale:** Check 5 (alignment, Rule 2): Entry Points row named the source only at feature level; tightened to the exact source spec whose Navigation Out declares this destination.
- **Status:** applied

### E-132 — FEAT-06.SPEC-004 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-004-booking-detail-via-manage-link.md`
- **Section:** Navigation Out
- **Before:** | "Cancel or Reschedule" tap | Cancel/reschedule flow | FEAT-10 (Client-Initiated Cancel/Reschedule) |
- **After:** | "Cancel or Reschedule" tap | Cancel/reschedule flow, FEAT-10.SPEC-001 (Cancel Booking) | FEAT-10 (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-133 — FEAT-12.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-001-todays-upcoming-schedule.md`
- **Section:** Navigation Out
- **Before:** | Client name tap | FEAT-13 (Client Record Management) | FEAT-13 |
- **After:** | Client name tap | FEAT-13.SPEC-001 (Client Record Detail) | FEAT-13 |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-134 — FEAT-12.SPEC-002 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-002-attention-list.md`
- **Section:** Navigation Out
- **Before:** | Dispute card tap | FEAT-16 (Booking & Payment Activity Record) | FEAT-16 |
- **After:** | Dispute card tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-135 — FEAT-12.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-003-past-bookings-browse.md`
- **Section:** Navigation Out
- **Before:** | Booking row tap (elsewhere) | FEAT-16 (Booking & Payment Activity Record) | FEAT-16 |
- **After:** | Booking row tap (elsewhere) | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-136 — FEAT-06.SPEC-004 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-004-booking-detail-via-manage-link.md`
- **Section:** Navigation Out
- **Before:** | "Pay Balance" tap (v1) | Balance payment flow | FEAT-22 (In-App Balance Payment) |
- **After:** | "Pay Balance" tap (v1) | Balance payment flow, FEAT-22.SPEC-001 (Balance Payment) | FEAT-22 (In-App Balance Payment) |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-137 — FEAT-12.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-12-pro-daily-schedule-dashboard/FEAT-12.SPEC-001-todays-upcoming-schedule.md`
- **Section:** Navigation Out
- **Before:** | Navigation -- Insights | FEAT-25 (Booking & Revenue Insights) | FEAT-25 |
- **After:** | Navigation -- Insights | FEAT-25.SPEC-001 (Insights Summary Screen) | FEAT-25 |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-138 — FEAT-15.SPEC-001 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-001-setup-wizard-shell-step-navigation-guidance.md`
- **Section:** Navigation Out
- **Before:** | Continue on profile step | Profile & studio location screen | FEAT-27 (Pro Profile & Booking Page Settings) |
- **After:** | Continue on profile step | Profile & studio location screen (FEAT-27.SPEC-001) | FEAT-27 (Pro Profile & Booking Page Settings) |
- **Rationale:** Check 5 (alignment, Rule 2): Navigation Out named the destination only at feature level; tightened to the exact destination spec whose Entry Points already list this screen. No behavior change.
- **Status:** applied

### E-139 — FEAT-07.SPEC-002 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-002-deposit-capture-booking-confirmation.md`
- **Section:** Outcome Definitions
- **Before:** | FEAT-07.SPEC-001, FEAT-05.SPEC-005, FEAT-08, FEAT-12, FEAT-16, FEAT-30 |
- **After:** | FEAT-07.SPEC-001, FEAT-05.SPEC-005, FEAT-08 (FEAT-08.SPEC-001, FEAT-08.SPEC-005, FEAT-08.SPEC-007), FEAT-12, FEAT-16 (FEAT-16.SPEC-002), FEAT-30 (FEAT-30.SPEC-010) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-140 — FEAT-08.SPEC-004 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-004-booking-change-refund-notice.md`
- **Section:** Trigger
- **Before:** | Client cancels or reschedules their own booking | FEAT-10 (Client-Initiated Cancel/Reschedule) |
- **After:** | Client cancels or reschedules their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-141 — FEAT-08.SPEC-004 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-004-booking-change-refund-notice.md`
- **Section:** Trigger
- **Before:** | Pro cancels or reschedules a client's booking | FEAT-30 (Pro Booking Management) |
- **After:** | Pro cancels or reschedules a client's booking | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit), via their trigger contract FEAT-30.SPEC-012 (Pro Booking Management) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-142 — FEAT-08.SPEC-004 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-004-booking-change-refund-notice.md`
- **Section:** Trigger
- **Before:** | Pro issues a goodwill refund | FEAT-30 (Pro Booking Management) |
- **After:** | Pro issues a goodwill refund | FEAT-30.SPEC-009 (Goodwill Refund Commit) / FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution), via their trigger contract FEAT-30.SPEC-012 (Pro Booking Management) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-143 — FEAT-08.SPEC-004 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-004-booking-change-refund-notice.md`
- **Section:** Trigger
- **Before:** | Automatic refund succeeds, enters progress, or fails | FEAT-09 (Cancellation & No-Show Policy Engine) |
- **After:** | Automatic refund succeeds, enters progress, or fails | FEAT-09.SPEC-005 (Automatic Deposit Refund) (Cancellation & No-Show Policy Engine) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-144 — FEAT-09.SPEC-005 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-005-automatic-deposit-refund.md`
- **Section:** Connected Specs
- **Before:** | FEAT-08 (Automated Booking Messaging) | Affects (outbound) |
- **After:** | FEAT-08.SPEC-004 (Booking Change & Refund Notice) -- within FEAT-08 (Automated Booking Messaging) | Affects (outbound) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-145 — FEAT-09.SPEC-005 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-09-cancellation-no-show-policy-engine/FEAT-09.SPEC-005-automatic-deposit-refund.md`
- **Section:** Outcome Definitions
- **Before:** The outcome is included in the relevant confirmation message (FEAT-08), not a separate standalone notice
- **After:** The outcome is included in the relevant confirmation message (FEAT-08.SPEC-004), not a separate standalone notice
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-146 — FEAT-08.SPEC-005 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-005-pro-booking-activity-notification.md`
- **Section:** Trigger
- **Before:** | Client cancels their own booking | FEAT-10 (Client-Initiated Cancel/Reschedule) |
- **After:** | Client cancels their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-147 — FEAT-08.SPEC-005 (Notification trigger sources)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-005-pro-booking-activity-notification.md`
- **Section:** Trigger
- **Before:** | Client reschedules their own booking | FEAT-10 (Client-Initiated Cancel/Reschedule) |
- **After:** | Client reschedules their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) |
- **Rationale:** Check 12 notification trigger source (alignment): trigger source / acknowledgment named only at feature level; tightened to the exact spec IDs already cross-citing each other (the trigger-contract specs FEAT-10.SPEC-006 and FEAT-30.SPEC-012 already name this notification as the content owner). No behavior change.
- **Status:** applied

### E-148 — FEAT-03.SPEC-002 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-002-slot-hold-creation-checkout-reservation.md`
- **Section:** Trigger Definition
- **Before:** | Client begins the payment step for a chosen slot | FEAT-05 (Public Booking Page & Booking Flow) / FEAT-07 (Deposit Payment at Booking) |
- **After:** | Client begins the payment step for a chosen slot | FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) -- within FEAT-05 (Public Booking Page & Booking Flow) / FEAT-07 (Deposit Payment at Booking) |
- **Rationale:** Check 6 bidirectional trigger (alignment): FEAT-05.SPEC-006 declares "Triggers (outbound)" to this spec; the trigger source is tightened from feature level to that spec ID. The hold-timing difference between the two specs is routed to the orchestrator as STRUCTURAL-GAP SG-03.
- **Status:** applied

### E-149 — FEAT-30.SPEC-007 (Business-rule consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-007-pro-cancel-reschedule-commit.md`
- **Section:** Processing Logic
- **Before:** request its full refund (XBR-23) through the same payment-processing capability path FEAT-30.SPEC-011 uses for other Pro-triggered refunds.
- **After:** request its full refund (XBR-23) through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund), which owns the outbound balance refund; the deposit leg continues to run through its own refund path.
- **Rationale:** Check 9 business-rule consistency (alignment, Rule 3 / XBR-23 authority FEAT-22): the balance-refund leg is owned by FEAT-22.SPEC-005 (Brief, External Touchpoints row and XBR-23 authority all name FEAT-22); FEAT-30 wording now cites that spec instead of FEAT-30.SPEC-011. Carried-forward Pass B/C item.
- **Status:** applied

### E-150 — FEAT-30.SPEC-007 (Business-rule consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-007-pro-cancel-reschedule-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-22 (In-App Balance Payment) | Affects (outbound, v1) |
- **After:** | FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound, v1) |
- **Rationale:** Check 9 business-rule consistency (alignment, Rule 3 / XBR-23 authority FEAT-22): the balance-refund leg is owned by FEAT-22.SPEC-005 (Brief, External Touchpoints row and XBR-23 authority all name FEAT-22); FEAT-30 wording now cites that spec instead of FEAT-30.SPEC-011. Carried-forward Pass B/C item. Also resolves Check 6: FEAT-22.SPEC-005 lists FEAT-30 as "Triggered by (inbound)".
- **Status:** applied

### E-151 — FEAT-30.SPEC-008 (Business-rule consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-008-bulk-cancellation-commit.md`
- **Section:** Processing Logic
- **Before:** d. Check whether a Balance Payment exists with status Succeeded (v1); if so, request its full refund (XBR-23).
- **After:** d. Check whether a Balance Payment exists with status Succeeded (v1); if so, request its full refund (XBR-23) through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund).
- **Rationale:** Check 9 business-rule consistency (alignment, Rule 3 / XBR-23 authority FEAT-22): the balance-refund leg is owned by FEAT-22.SPEC-005 (Brief, External Touchpoints row and XBR-23 authority all name FEAT-22); FEAT-30 wording now cites that spec instead of FEAT-30.SPEC-011. Carried-forward Pass B/C item.
- **Status:** applied

### E-152 — FEAT-30.SPEC-008 (Business-rule consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-008-bulk-cancellation-commit.md`
- **Section:** Connected Specs
- **Before:** | FEAT-22 (In-App Balance Payment) | Affects (outbound, v1) |
- **After:** | FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound, v1) |
- **Rationale:** Check 9 business-rule consistency (alignment, Rule 3 / XBR-23 authority FEAT-22): the balance-refund leg is owned by FEAT-22.SPEC-005 (Brief, External Touchpoints row and XBR-23 authority all name FEAT-22); FEAT-30 wording now cites that spec instead of FEAT-30.SPEC-011. Carried-forward Pass B/C item.
- **Status:** applied

### E-153 — FEAT-22.SPEC-005 (Business-rule consistency)

- **File:** `.n2b/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-005-balance-charge-payout-routing-refund.md`
- **Section:** Connected Specs (cross-feature note)
- **Before:** the wording in FEAT-30.SPEC-007 should be reconciled to cite FEAT-22.SPEC-005 rather than FEAT-30.SPEC-011 for the balance leg specifically.
- **After:** Pass D reconciliation has aligned FEAT-30.SPEC-007 (and FEAT-30.SPEC-008) to cite FEAT-22.SPEC-005 for the balance leg specifically.
- **Rationale:** Check 9: note updated to reflect the alignment edit made in FEAT-30.SPEC-007/008; the FEAT-09.SPEC-005 half of the note is routed to the orchestrator as STRUCTURAL-GAP SG-09.
- **Status:** applied

### E-154 — FEAT-19.SPEC-001 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-19-platform-support-read-only-access/FEAT-19.SPEC-001-pro-account-lookup-support-session-entry.md`
- **Section:** Layout and Content
- **Before:** accepts the Pro's booking_link_name, sign-in_email, sign-in_mobile, or display_name
- **After:** accepts the Pro's booking_link_name, sign_in_email, sign_in_mobile, or display_name
- **Rationale:** Check 4 shared-entity field consistency (alignment, Rule 1): Pro Account field names follow the dependency map / creating feature spelling (sign_in_email, sign_in_mobile).
- **Status:** applied

### E-155 — FEAT-22.SPEC-005 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-22-in-app-balance-payment/FEAT-22.SPEC-005-balance-charge-payout-routing-refund.md`
- **Section:** Connected Specs
- **Before:** | FEAT-30 (Pro Booking Management) | Triggered by (inbound) |
- **After:** | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), FEAT-30.SPEC-008 (Bulk Cancellation Commit) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) |
- **Rationale:** Check 6/9 (alignment): inbound trigger tightened to the two FEAT-30 commits that now cite this spec for the balance refund leg.
- **Status:** applied

### E-156 — FEAT-18.SPEC-006 (Business-rule consistency)

- **File:** `.n2b/specifications/FEAT-18-pro-subscription-billing-account-management/FEAT-18.SPEC-006-subscription-billing-integration.md`
- **Section:** Analytics and Success Signals
- **Before:** - **subscription_cancellation_confirmed_by_processor** (invoked via: self-service / account closure)
- **After:** - **subscription_cancelled** (invoked via: self-service / account closure; fires when the payment-processing capability confirms the cancellation)
- **Rationale:** Carried-forward Pass C item (FEAT-18 analytics names vs Stage 2 Signals): product-features.md FEAT-18 Signals name the terminal event subscription_cancelled; the processor-confirmed cancellation is that terminal moment, so the event is renamed to the Stage 2 name. subscription_cancel_confirmed (FEAT-18.SPEC-002) stays as the distinct UI-intent event. No behavior change.
- **Status:** applied

### E-157 — FEAT-03.SPEC-007 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md`
- **Section:** Scope and Non-Goals
- **Before:** marking the associated Booking Expired, with Pro notification
- **After:** marking the associated Booking Expired (unpaid), with Pro notification
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-158 — FEAT-03.SPEC-007 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md`
- **Section:** Processing Logic
- **Before:** 10. Mark the associated Booking's state as Expired.
- **After:** 10. Mark the associated Booking's state as Expired (unpaid).
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-159 — FEAT-03.SPEC-007 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md`
- **Section:** Business Rules
- **Before:** associated Booking is marked Expired, never silently deleted
- **After:** associated Booking is marked Expired (unpaid), never silently deleted
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-160 — FEAT-03.SPEC-007 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-03-real-time-slot-availability-engine/FEAT-03.SPEC-007-pro-created-deposit-request-hold-expiration.md`
- **Section:** Acceptance Criteria
- **Before:** the associated Booking is marked Expired, and Talia
- **After:** the associated Booking is marked Expired (unpaid), and Talia
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-161 — FEAT-30.SPEC-010 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Scope and Non-Goals
- **Before:** Reflecting the Booking's transition to Expired when
- **After:** Reflecting the Booking's transition to Expired (unpaid) when
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-162 — FEAT-30.SPEC-010 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Processing Logic
- **Before:** transition this Booking's state to Expired and surface
- **After:** transition this Booking's state to Expired (unpaid) and surface
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-163 — FEAT-30.SPEC-010 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Outcome Definitions
- **Before:** | Booking.state -> Expired |
- **After:** | Booking.state -> Expired (unpaid) |
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-164 — FEAT-30.SPEC-010 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Data Model
- **Before:** or -> Expired (on unpaid hold expiry)
- **After:** or -> Expired (unpaid) (on unpaid hold expiry)
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-165 — FEAT-30.SPEC-010 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Acceptance Criteria
- **Before:** this Booking's state transitions to Expired and Talia
- **After:** this Booking's state transitions to Expired (unpaid) and Talia
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-166 — FEAT-11.SPEC-004 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-004-no-show-marking-window-authorization-rules.md`
- **Section:** Governed Entity
- **Before:** \| Rescheduled \| Expired -- the marking window
- **After:** \| Rescheduled \| Expired (unpaid) -- the marking window
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-167 — FEAT-11.SPEC-004 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-11-no-show-marking-deposit-forfeiture/FEAT-11.SPEC-004-no-show-marking-window-authorization-rules.md`
- **Section:** Scope and Non-Goals
- **Before:** Cancelled, Rescheduled, Expired, or the auto-completion
- **After:** Cancelled, Rescheduled, Expired (unpaid), or the auto-completion
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" per the creating feature FEAT-05 (FEAT-05.SPEC-006) and the dependency map; bare "Expired" aligned. No behavior change.
- **Status:** applied

### E-168 — FEAT-08.SPEC-010 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-08-automated-booking-messaging/FEAT-08.SPEC-010-booking-specific-manage-link-issuance.md`
- **Section:** Trigger Definition
- **Before:** | A Pro-initiated reschedule occurs | FEAT-30 (Pro Booking Management) |
- **After:** | A Pro-initiated reschedule occurs | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) (Pro Booking Management) |
- **Rationale:** Check 6 (alignment): feature-level trigger source tightened to the only FEAT-30 spec that commits a Pro-initiated reschedule. Reverse acknowledgment missing in FEAT-30.SPEC-007 is logged as STRUCTURAL-GAP SG-06.
- **Status:** applied

### E-169 — FEAT-15.SPEC-007 (Logic/Rule consistency)

- **File:** `.n2b/specifications/FEAT-15-pro-onboarding-setup-wizard/FEAT-15.SPEC-007-go-live-prerequisite-rule-xbr-26-authority.md`
- **Section:** Scope and Non-Goals
- **Before:** Validation & Limits field: the eight required steps are a hard gate
- **After:** Validation & Limits field: the seven required conditions (the eight wizard steps less the optional calendar step) are a hard gate
- **Rationale:** Check 7/9 intra-feature consistency (alignment, Rule 4): this Logic/Rule spec defines seven mandatory conditions (Business Rules, Cross-Field Rules) and FEAT-15.SPEC-005 was already aligned to seven in Pass C; the stray "eight required steps" Non-Goal wording (Pass C should-fix) is aligned. No behavior change.
- **Status:** applied

## Final Alignment Pass (post gap-routing verification)

Run after the orchestrator routed SG-01..SG-15 and MS-01 (Feature Analyst Brief corrections for FEAT-04/05/06/10/14/16/18/21/24/30; producer revisions across FEAT-01..19, 21, 22, 26, 28, 29, 30; new spec FEAT-21.SPEC-010), under the ownership decisions the orchestrator recorded for those re-spawns. Scope: alignment edits only. No spec content added, no Brief or dependency-map edits. Every check was re-run by script over all 219 specs (70 screen, 59 automation, 53 logic-rule, 13 integration, 24 notification), the 30 Briefs and the map.

### Checks Re-Run

| # | Check | Result | Disposition |
|---|-------|--------|-------------|
| 1 | Brief completeness | 219 spec files ↔ 30 Brief inventories; FEAT-21.SPEC-010 is listed in the FEAT-21 Brief; 0 orphans, 0 dangling Brief IDs | Pass |
| 2/3 | Cross-/intra-feature references | 0 dangling `FEAT-NN.SPEC-NNN` IDs in specs, Briefs or map; 0 malformed IDs; 2 wrong spec names beside correct IDs (FEAT-25.SPEC-004 → FEAT-07.SPEC-005; FEAT-04.SPEC-005 → FEAT-07.SPEC-002) | Applied (E-173, E-174, E-189) |
| 4 | Shared entity consistency | Booking creation cited to FEAT-05.SPEC-009 instead of the creating spec FEAT-05.SPEC-006 (FEAT-07.SPEC-003); acknowledgment fields described as written "on the Booking" before it exists (FEAT-05.SPEC-009); two bare "Expired" Booking states | Applied (E-171, E-172, E-175..E-179, E-181, E-182) |
| 5 | Bidirectional navigation (incl. CTA deep links) | New forward link FEAT-30.SPEC-001 → FEAT-21.SPEC-010 was named at feature level on the destination and used a different label; FEAT-06.SPEC-003 → FEAT-20.SPEC-002 and FEAT-13.SPEC-003 → FEAT-24.SPEC-001 named the destination at feature level. The other one-way hits are back-arrow returns, embedded sections (FEAT-23.SPEC-001 in FEAT-22.SPEC-001; FEAT-14.SPEC-001 in FEAT-06.SPEC-005) or CTAs that route through an intermediary (FEAT-06.SPEC-006 via FEAT-06.SPEC-002; FEAT-20.SPEC-009 via FEAT-05.SPEC-001). These were excluded as in Pass D | Applied (E-184..E-188, E-190..E-193) |
| 6 | Bidirectional automation triggers (incl. Integration Inbound Events) | FEAT-05.SPEC-006 cites FEAT-07.SPEC-001 (Pay tap) as its pre-charge trigger source, and FEAT-07.SPEC-001 did not name it; Integration Inbound Events: 0 omissions | Applied (E-170). SG-11/SG-12 are closed |
| 7 | Logic/Rule consistency | Enforced-By back-references: 1 of the 38 SG-13 pairs remains (rule-to-rule); stale step number in FEAT-05.SPEC-006 | Applied (E-180); SG-17 carried forward |
| 8 | Spec ID uniqueness | 219 unique IDs; file names match frontmatter | Pass |
| 9 | Cross-feature business rules | Hold timing (XBR-02) consistent across FEAT-03.SPEC-002, FEAT-05.SPEC-002/003/004/006; Expired (unpaid) has a single writer (FEAT-03.SPEC-007); XBR-23 balance refund routed from FEAT-09.SPEC-005 to FEAT-22.SPEC-005; FEAT-30.SPEC-010 cited a signal pairing FEAT-03.SPEC-007 does not have | Applied (E-183) |
| 10 | Entity lifecycle completeness | Recurring Series Pro-side create/manage now has FEAT-21.SPEC-010 (MS-01 closed); map lifecycle and state lines unchanged | SG-01/SG-02 carried forward |
| 11 | External Touchpoints ↔ Integration specs | 13 Integration specs, all cited; every cited ID exists | Pass |
| 12 | Notification trigger sources | Every notification Source Spec exists and acknowledges it, or (FEAT-08.SPEC-004/005) acknowledges it through its trigger-and-audience contract FEAT-10.SPEC-006 / FEAT-30.SPEC-012, as SG-07 decided. FEAT-21.SPEC-010 → FEAT-08.SPEC-004 is one-way | SG-16 carried forward |
| 13 | Degradation Behavior screen references | Every ID in the 13 Degradation Behavior sections exists | Pass |
| 14 | Check 14 markers → registry | Strict sweep: 294 marker sites, 33 distinct slugs. Near-miss sweep over every spec file (case-insensitive, hyphen or space): 0 lines carrying a marker phrase in any shape other than the exact lowercase marker with a backticked kebab slug; 0 unmarked "fixed" platform-set value phrasings in specs or registry (the only remaining occurrences under specifications/ are the quoted Before text of Pass D edits in this log). 1 unmarked concrete policy number added in revision (FEAT-05.SPEC-008 "7-day grace") | Applied (E-194, E-195); registry is 33 rows, one per slug, all `decide-before-build` |

### Final Pass Edits Applied

Every entry has status **applied**. E-170..E-194 are spec edits and E-195 is the registry rebuild. Total: 26 edits across 12 files.

### E-170 — FEAT-07.SPEC-001 (Bidirectional triggers)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-001-deposit-payment.md`
- **Section:** Interactions
- **Before:** 1. Confirm the Booking is still Pending Payment and the checkout hold is still active.
- **After:** 1. Confirm the Booking is still Pending Payment and the checkout hold is still active (the pre-charge hold re-validation run by FEAT-05.SPEC-006).
- **Rationale:** Check 6 bidirectional trigger (alignment): FEAT-05.SPEC-006 Trigger Definition cites this screen's Pay tap as its "Client submits payment" source (the pre-charge hold-still-active check); the existing Pay-button step that performs that check now names the automation that runs it. No behavior change.
- **Status:** applied

### E-171 — FEAT-07.SPEC-003 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-003-deposit-amount-eligibility-rules.md`
- **Section:** Field Validation Rules
- **Before:** at the moment the Booking is created (FEAT-05.SPEC-009); never recomputed
- **After:** at the moment the Booking is created (FEAT-05.SPEC-006, from the amount FEAT-05.SPEC-009 computes); never recomputed
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): the Booking is created by FEAT-05.SPEC-006 when the client advances into the payment step (SG-03 resolution); FEAT-05.SPEC-009 computes the amount but does not create the Booking. Creation citation aligned to the creating spec. No behavior change.
- **Status:** applied

### E-172 — FEAT-07.SPEC-003 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-07-deposit-payment-at-booking/FEAT-07.SPEC-003-deposit-amount-eligibility-rules.md`
- **Section:** Defaults and Derivations
- **Before:** On Booking creation only (FEAT-05.SPEC-009); read-only thereafter
- **After:** On Booking creation only (FEAT-05.SPEC-006, from the amount FEAT-05.SPEC-009 computes); read-only thereafter
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): same as the preceding edit -- Booking creation is FEAT-05.SPEC-006's. No behavior change.
- **Status:** applied

### E-173 — FEAT-25.SPEC-004 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-25-booking-revenue-insights/FEAT-25.SPEC-004-historical-aggregate-maintenance.md`
- **Section:** Trigger Definition
- **Before:** \| FEAT-07.SPEC-005 (Deposit Payment Processing Integration) \| Fires when a deposit capture succeeds
- **After:** \| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) \| Fires when a deposit capture succeeds
- **Rationale:** Check 2 cross-feature reference (alignment): the cited spec name did not match FEAT-07.SPEC-005's actual name ("Card Deposit Charge & Payout Routing"); the ID was already correct. No behavior change.
- **Status:** applied

### E-174 — FEAT-25.SPEC-004 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-25-booking-revenue-insights/FEAT-25.SPEC-004-historical-aggregate-maintenance.md`
- **Section:** Connected Specs
- **Before:** \| FEAT-07.SPEC-005 (Deposit Payment Processing Integration) \| Triggered by (inbound)
- **After:** \| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) \| Triggered by (inbound)
- **Rationale:** Check 2 cross-feature reference (alignment): spec name aligned to FEAT-07.SPEC-005's actual name. No behavior change.
- **Status:** applied

### E-175 — FEAT-05.SPEC-009 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-009-policy-acknowledgment-capture-integrity-check.md`
- **Section:** Overview (Purpose)
- **Before:** records the acknowledged cancellation policy version and wording on the Booking, and re-validates
- **After:** captures the acknowledged cancellation policy version and wording into the in-progress checkout (carried onto the Booking when FEAT-05.SPEC-006 creates it), and re-validates
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): the Booking does not exist when the acknowledgment box is checked -- FEAT-05.SPEC-006 (the Booking's creating spec) creates it when the client advances into the payment step and fixes the acknowledged version, wording and timestamp on it at creation (its Processing step 2). Wording aligned: the acknowledgment is captured into the in-progress checkout and carried onto the Booking at creation. No behavior change.
- **Status:** applied

### E-176 — FEAT-05.SPEC-009 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-009-policy-acknowledgment-capture-integrity-check.md`
- **Section:** Scope and Non-Goals (In Scope)
- **Before:** Recording the acknowledged Cancellation Policy version, its exact plain-language wording, and the acknowledgment timestamp onto the Booking, the moment the client checks the acknowledgment box
- **After:** Recording the acknowledged Cancellation Policy version, its exact plain-language wording, and the acknowledgment timestamp into the in-progress checkout the moment the client checks the acknowledgment box; FEAT-05.SPEC-006 carries them onto the Booking when it creates it
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): the Booking does not exist when the acknowledgment box is checked -- FEAT-05.SPEC-006 (the Booking's creating spec) creates it when the client advances into the payment step and fixes the acknowledged version, wording and timestamp on it at creation (its Processing step 2). Wording aligned: the acknowledgment is captured into the in-progress checkout and carried onto the Booking at creation. No behavior change.
- **Status:** applied

### E-177 — FEAT-05.SPEC-009 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-009-policy-acknowledgment-capture-integrity-check.md`
- **Section:** Edge Cases
- **Before:** only the most recent acknowledgment is ever recorded on the Booking.
- **After:** only the most recent acknowledgment is carried onto the Booking when FEAT-05.SPEC-006 creates it.
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): the Booking does not exist when the acknowledgment box is checked -- FEAT-05.SPEC-006 (the Booking's creating spec) creates it when the client advances into the payment step and fixes the acknowledged version, wording and timestamp on it at creation (its Processing step 2). Wording aligned: the acknowledgment is captured into the in-progress checkout and carried onto the Booking at creation. No behavior change.
- **Status:** applied

### E-178 — FEAT-05.SPEC-009 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-009-policy-acknowledgment-capture-integrity-check.md`
- **Section:** Acceptance Criteria (AC-04)
- **Before:** when the check registers, then the Booking's policy_version, acknowledged_wording, and acknowledgment_timestamp are set to
- **After:** when the check registers, then the in-progress checkout's policy_version, acknowledged_wording, and acknowledgment_timestamp (carried onto the Booking when FEAT-05.SPEC-006 creates it) are set to
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): the Booking does not exist when the acknowledgment box is checked -- FEAT-05.SPEC-006 (the Booking's creating spec) creates it when the client advances into the payment step and fixes the acknowledged version, wording and timestamp on it at creation (its Processing step 2). Wording aligned: the acknowledgment is captured into the in-progress checkout and carried onto the Booking at creation. No behavior change.
- **Status:** applied

### E-179 — FEAT-05.SPEC-009 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-009-policy-acknowledgment-capture-integrity-check.md`
- **Section:** Acceptance Criteria (AC-06)
- **Before:** against the refreshed wording, then the Booking's policy_version, acknowledged_wording, and acknowledgment_timestamp are overwritten
- **After:** against the refreshed wording, then the in-progress checkout's policy_version, acknowledged_wording, and acknowledgment_timestamp are overwritten
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): the Booking does not exist when the acknowledgment box is checked -- FEAT-05.SPEC-006 (the Booking's creating spec) creates it when the client advances into the payment step and fixes the acknowledged version, wording and timestamp on it at creation (its Processing step 2). Wording aligned: the acknowledgment is captured into the in-progress checkout and carried onto the Booking at creation. No behavior change.
- **Status:** applied

### E-180 — FEAT-05.SPEC-006 (Logic/Rule consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-006-slot-hold-re-validation-at-checkout.md`
- **Section:** Edge Cases
- **Before:** Step 4's re-validation surfaces this
- **After:** Step 5's re-validation surfaces this
- **Rationale:** Check 7 intra-spec consistency (alignment): after the SG-03 revision the pre-charge re-validation is Processing Logic step 5 (step 4 is the hand-off signal); the stale step number is aligned. No behavior change.
- **Status:** applied

### E-181 — FEAT-05.SPEC-006 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-006-slot-hold-re-validation-at-checkout.md`
- **Section:** Outcome Definitions
- **Before:** Booking transitions to Confirmed via FEAT-07 instead of Expired \|
- **After:** Booking transitions to Confirmed via FEAT-07 instead of Expired (unpaid) \|
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): Booking.state value named "Expired (unpaid)" as in the rest of this spec and the dependency map. No behavior change.
- **Status:** applied

### E-182 — FEAT-30.SPEC-010 (Shared entity consistency)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Business Rules
- **Before:** An expired Pro-created booking is marked Expired (by FEAT-03.SPEC-007)
- **After:** An expired Pro-created booking is marked Expired (unpaid) (by FEAT-03.SPEC-007)
- **Rationale:** Check 4 shared-entity consistency (alignment, Rule 1): bare "Expired" aligned to the Booking.state value "Expired (unpaid)". No behavior change.
- **Status:** applied

### E-183 — FEAT-30.SPEC-010 (Cross-feature business rules)

- **File:** `.n2b/specifications/FEAT-30-pro-booking-management/FEAT-30.SPEC-010-pro-created-booking-deposit-request-hold.md`
- **Section:** Analytics and Success Signals
- **Before:** is measured against deposit_request_expired, per FEAT-03.SPEC-007's same signal pairing)
- **After:** is measured against deposit_request_expired, emitted below when this automation reads the Expired (unpaid) state FEAT-03.SPEC-007 writes)
- **Rationale:** Check 2/9 consistency (alignment): FEAT-03.SPEC-007 emits no deposit_request_expired signal, so the "same signal pairing" citation pointed at nothing; after SG-08 this automation emits the signal on reading the state FEAT-03.SPEC-007 writes (its own Processing step 7 and the signal list below), which also matches the FEAT-30 Brief Signals line. No behavior change.
- **Status:** applied

### E-184 — FEAT-21.SPEC-010 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-010-pro-recurring-series-management.md`
- **Section:** Entry Points
- **Before:** \| FEAT-30 (Pro Booking Management) -- Pro booking detail outbound link \| Talia opens a client's booking on her schedule and taps "Standing appointment" (labelled "Repeat this every N weeks" when the booking has no series) \|
- **After:** \| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) -- within FEAT-30 (Pro Booking Management), Pro booking detail outbound link \| Talia opens a client's booking on her schedule and taps the "Recurring series" link ("Repeat this booking" when the booking has no series, "Manage recurring series" when it belongs to one) \|
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) declares the outbound "Recurring series" link to this screen ("Repeat this booking" / "Manage recurring series"), but this new spec named the source only at feature level and used a different control label. Source ID tightened and label aligned to the declaring spec. No behavior change.
- **Status:** applied

### E-185 — FEAT-21.SPEC-010 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-010-pro-recurring-series-management.md`
- **Section:** Interactions
- **Before:** Navigate to the originating screen: FEAT-30 (Pro Booking Management) booking detail,
- **After:** Navigate to the originating screen: FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) booking detail,
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) declares the outbound "Recurring series" link to this screen ("Repeat this booking" / "Manage recurring series"), but this new spec named the source only at feature level and used a different control label. Source ID tightened and label aligned to the declaring spec. No behavior change.
- **Status:** applied

### E-186 — FEAT-21.SPEC-010 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-010-pro-recurring-series-management.md`
- **Section:** Navigation Out
- **Before:** \| Back arrow tap (arrived from a booking) \| FEAT-30 (Pro Booking Management) booking detail \|
- **After:** \| Back arrow tap (arrived from a booking) \| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) booking detail \|
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) declares the outbound "Recurring series" link to this screen ("Repeat this booking" / "Manage recurring series"), but this new spec named the source only at feature level and used a different control label. Source ID tightened and label aligned to the declaring spec. No behavior change.
- **Status:** applied

### E-187 — FEAT-21.SPEC-010 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-010-pro-recurring-series-management.md`
- **Section:** Connected Specs
- **Before:** \| FEAT-30 (Pro Booking Management) \| Navigation (inbound/outbound) \| Talia arrives from a client's booking detail outbound link
- **After:** \| FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) -- within FEAT-30 (Pro Booking Management) \| Navigation (inbound/outbound) \| Talia arrives from the "Recurring series" link on a client's booking detail
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) declares the outbound "Recurring series" link to this screen ("Repeat this booking" / "Manage recurring series"), but this new spec named the source only at feature level and used a different control label. Source ID tightened and label aligned to the declaring spec. No behavior change.
- **Status:** applied

### E-188 — FEAT-21.SPEC-010 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-21-recurring-standing-appointments/FEAT-21.SPEC-010-pro-recurring-series-management.md`
- **Section:** Acceptance Criteria (AC-01)
- **Before:** Given Talia opens Riley's eligible booking in FEAT-30 and taps "Repeat this every N weeks",
- **After:** Given Talia opens Riley's eligible booking in FEAT-30.SPEC-001 and taps "Repeat this booking",
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-30.SPEC-001 (Cancel Booking (Pro-Initiated)) declares the outbound "Recurring series" link to this screen ("Repeat this booking" / "Manage recurring series"), but this new spec named the source only at feature level and used a different control label. Source ID tightened and label aligned to the declaring spec. No behavior change.
- **Status:** applied

### E-189 — FEAT-04.SPEC-005 (Cross-feature references)

- **File:** `.n2b/specifications/FEAT-04-two-way-calendar-sync/FEAT-04.SPEC-005-booking-to-calendar-sync.md`
- **Section:** Trigger Definition
- **Before:** confirmed through FEAT-07.SPEC-002 (Pro Booking Management)
- **After:** confirmed through FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation)
- **Rationale:** Check 2 cross-feature reference (alignment): the parenthetical named the wrong spec (FEAT-07.SPEC-002 is Deposit Capture & Booking Confirmation; "Pro Booking Management" is FEAT-30, already cited by FEAT-30.SPEC-010 in the same cell). No behavior change.
- **Status:** applied

### E-190 — FEAT-06.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-003-my-bookings-list.md`
- **Section:** Interactions
- **Before:** Confirms intent, then navigates to FEAT-20 (Waitlist for Cancelled Slots) to remove the entry
- **After:** Confirms intent, then navigates to FEAT-20.SPEC-002 (My Waitlists) within FEAT-20 (Waitlist for Cancelled Slots) to remove the entry
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-20.SPEC-002 (My Waitlists) lists this screen's Waitlist section as its entry point, but this screen named the destination only at feature level; tightened to the exact spec ID that performs the removal. No behavior change.
- **Status:** applied

### E-191 — FEAT-06.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-003-my-bookings-list.md`
- **Section:** Navigation Out
- **Before:** \| Waitlist "Leave" confirmed \| Waitlist removal \| FEAT-20 (Waitlist for Cancelled Slots) \|
- **After:** \| Waitlist "Leave" confirmed \| FEAT-20.SPEC-002 (My Waitlists), waitlist removal \| FEAT-20 (Waitlist for Cancelled Slots) \|
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-20.SPEC-002 (My Waitlists) lists this screen's Waitlist section as its entry point, but this screen named the destination only at feature level; tightened to the exact spec ID that performs the removal. No behavior change.
- **Status:** applied

### E-192 — FEAT-06.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-06-client-booking-identity/FEAT-06.SPEC-003-my-bookings-list.md`
- **Section:** Connected Specs
- **Before:** \| FEAT-20 (Waitlist for Cancelled Slots) \| Navigation (outbound) \|
- **After:** \| FEAT-20.SPEC-002 (My Waitlists) -- within FEAT-20 (Waitlist for Cancelled Slots) \| Navigation (outbound) \|
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-20.SPEC-002 (My Waitlists) lists this screen's Waitlist section as its entry point, but this screen named the destination only at feature level; tightened to the exact spec ID that performs the removal. No behavior change.
- **Status:** applied

### E-193 — FEAT-13.SPEC-003 (Bidirectional navigation)

- **File:** `.n2b/specifications/FEAT-13-client-record-management/FEAT-13.SPEC-003-client-deletion-confirmation.md`
- **Section:** Navigation Out
- **Before:** \| Successful deletion \| The screen the Pro arrived at before opening the client record \| FEAT-12 or FEAT-24 \|
- **After:** \| Successful deletion \| The screen the Pro arrived at before opening the client record \| FEAT-12 or FEAT-24 (FEAT-24.SPEC-001, Client Search & Filter) \|
- **Rationale:** Check 5 bidirectional navigation (alignment, Rule 2 (declared outbound navigation is authoritative; destination aligned)): FEAT-24.SPEC-001 lists this screen's post-deletion return as an entry point, but this row named FEAT-24 only at feature level; tightened to the exact spec ID. No behavior change.
- **Status:** applied

### E-194 — FEAT-05.SPEC-008 (Platform-parameter markers)

- **File:** `.n2b/specifications/FEAT-05-public-booking-page-booking-flow/FEAT-05.SPEC-008-booking-page-availability-gate.md`
- **Section:** Business Rules
- **Before:** (subscription lapse after the 7-day grace, or a Pro-chosen pause;
- **After:** (subscription lapse after the 7-day grace, platform parameter: `subscription-payment-failure-grace-period-days`, or a Pro-chosen pause;
- **Rationale:** Check 14 (alignment): a concrete platform-wide policy number (the 7-day subscription grace period, XBR-14) introduced in the gap-routing revision without the marker; the existing slug is attached. The Stage 2 number is kept for readability, as elsewhere. No behavior change.
- **Status:** applied

### E-195 — platform-parameters.md (Check 14 registry)

- **File:** `.n2b/specifications/platform-parameters.md`
- **Section:** Registry (Referenced by) and frontmatter
- **Before:** `checkout-hold-timeout-minutes` referenced by FEAT-03.SPEC-002, FEAT-03.SPEC-003, FEAT-05.SPEC-002, FEAT-05.SPEC-006; `subscription-payment-failure-grace-period-days` referenced by FEAT-18.SPEC-003, FEAT-18.SPEC-005, FEAT-18.SPEC-007; marker_site_count 289
- **After:** `checkout-hold-timeout-minutes` adds FEAT-05.SPEC-003 (marker added in the SG-03 revision); `subscription-payment-failure-grace-period-days` adds FEAT-05.SPEC-008 (E-194) and FEAT-18.SPEC-004 (marker added in the gap-routing revision); every row's Referenced-by rebuilt by shell from the specs and sorted by spec ID; marker_site_count 294
- **Rationale:** Check 14: the registry is rebuilt from the shell sweep, never by recall. 33 distinct slugs in the specs and 33 registry rows, a one-to-one match; every row keeps `decide-before-build`; proposed defaults unchanged (non-binding).
- **Status:** applied
