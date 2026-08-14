# AfterImpact — open-question interview

Decisions captured in session on 2026-08-13. **All 60 are answered and applied to the documents that own
them; nothing was deferred.** See **Status** at the end for the full picture, including the one marker
retained deliberately.

This file is the record of what was decided and why. It is not itself a specification — every rule here
also lives, normatively, in `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, or `DESIGN.md`.

---

## Batch 1 — requirements data model (ANSWERED)

### Q1 · Timezone lifecycle and date-only boundaries
*Was OQ-10 / PQ-2. Owning document: `REQUIREMENTS.md` (Data Entities, NFR-6.1, FR-2.1, FR-4.1, FR-5.5, FR-2.4).*

**Answer:** The client detects the user's IANA timezone at registration and stores it on `User`; the user
can change it later. **Date-only boundaries evaluate in the user's timezone** — "MUST NOT be in the
future" (FR-2.1, FR-4.1), "due date has passed" (FR-5.5), and the FR-2.4 seven-day window all resolve
against the user's local day, not UTC. Storage stays UTC per NFR-6.1.

**Consequences to record:** a user profile/settings capability now exists (API + both clients) that did
not previously appear in any FR; `SEC-INPUT-1` gains timezone as a validated field; the FR-2.4 overview
and FR-5.5 overdue flag become timezone-dependent.

### Q2 · User locale for server-generated output
*Was OQ-11 / PQ-3. Owning document: `REQUIREMENTS.md` (Data Entities, NFR-5.4).*

**Answer:** Add a **`locale` attribute to `User`**, seeded from the client at registration and editable in
settings, alongside timezone. All server-generated output — reminder and security email, verification and
recovery mail, FR-9.5 inactivity warnings, FR-9.1 case-summary PDFs — renders in it. **Fall back to
English when unset.**

**Consequences to record:** shares the settings surface introduced by Q1; email templates and the PDF
pipeline become locale-aware.

### Q3 · Edit and delete for health records
*Was OQ-9 / PQ-1. Owning document: `REQUIREMENTS.md` §4.*

**Answer:** **Full edit and delete for all four** record types (Health Issue, Appointment, Progress
Update, Treatment). Health Issue and Appointment deletion carries the **30-day undo window** already
established by FR-2.6 and FR-3.8. **Deleting a Health Issue cascades** to its Progress Updates and
Treatments. This matches how tasks (FR-5.4), contacts (FR-6.3), and expenses (FR-7.2) already behave.

**Consequences to record:** new FRs in §4 for edit and delete; FR-8.1 timeline events for each; NFR-5.5
confirmations; SEC-LOG-1 deletion entries; an undo/restore surface for health records.

### Q4 · What happens to a link when its target is deleted
*Was OQ-14 / PQ-13. Owning document: `REQUIREMENTS.md` (FR-3.2, FR-4.5, FR-6.3, FR-7.1).*

**Answer:** **Generalize FR-6.3 to every link type.** The link is severed and the referencing record
**retains the target's name or title as plain text**, exactly as a deleted contact's name is retained
today. Applies to a deleted Document that was an Expense's receipt, a deleted Health Issue linked from
Documents/Expenses/Appointments, and a deleted Expense linked from a Document.

**Consequences to record:** the FR-6.3 name-snapshot rule becomes general; the persistence design gains
snapshot columns on each referencing record (a documented 3NF exception, as FR-6.3 already is);
the NFR-5.5 deletion confirmations must disclose the retention, as FR-6.3's already does.

---

## Batch 2 — money, account lifecycle, list contract (ANSWERED)

### Q5 · Reimbursement arithmetic
*Was OQ-12 / PQ-4. Owning document: `REQUIREMENTS.md` (FR-7.1, FR-7.3, FR-7.4).*

**Answer:** Money is **computed from reimbursement records; payment status is a user-owned label**.
Remaining balance = `amount − sum(reimbursements)`. FR-7.3's "total not yet reimbursed" = the sum of those
balances across the case. Payment status never changes itself. A reimbursement amount MUST be `> 0`, use
the currency's minor unit, and MUST NOT exceed the outstanding balance.

### Q6 · Allowed currency set and precision
*Was the PQ-5 residual. Owning document: `REQUIREMENTS.md` (FR-7.1, FR-7.5).*

**Answer:** Currency is an **ISO 4217 three-letter code, full set**. FR-7.1's "two decimal places" becomes
**"the currency's ISO 4217 minor unit"**, so JPY (0) and KWD (3) are handled correctly. NFR-6.2's exact
arithmetic is unchanged.

### Q7 · Definition of "inactive"
*Was OQ-13 / PQ-7. Owning document: `REQUIREMENTS.md` (FR-9.5, Data Entities).*

**Answer:** **A successful sign-in, and nothing else, resets the clock.** Store `last_sign_in_at` on
`User`. FR-1.10's 7-day absolute session lifetime means any genuinely active user resets it naturally.

### Q8 · List pagination, default sort, and tiebreak
*New, raised by many drafts. Owning document: `REQUIREMENTS.md`, with the API shape in `ARCHITECTURE.md`.*

**Answer:** **Every list endpoint paginates**, with a server-set default page size and a documented
maximum. **Every ordering ends with a deterministic tiebreak** on created timestamp then record
identifier, so pages never repeat or skip entries. Documents default to **upload date descending** when
the client requests no sort. Protects the NFR-4.1 latency budget as a case accumulates evidence.

---

## Batch 3 — security contracts (ANSWERED)

### Q9 · Step-up freshness and its API representation
*Was PQ-8. Owning document: `SECURITY.md` (SEC-AUTHN-7, SEC-AUTHN-10).*

**Answer:** A successful ceremony **stamps the server-side session**; any guarded operation within a
**5-minute window** is allowed, and one ceremony may cover several. A request lacking a fresh ceremony is
refused with a **distinct machine-readable refusal code** so the client can prompt rather than infer.

### Q10 · Mobile session credential transport
*Was PQ-9. Owning document: `SECURITY.md` (SEC-SESSION-3, SEC-SESSION-6).*

**Answer:** The opaque session identifier travels in an **`Authorization` bearer header** and rests in
platform secure storage. Because nothing is transmitted ambiently, **CSRF defenses remain web-only**,
which is why SEC-SESSION-3 already scopes them to the cookie transport.

### Q11 · Device-identifier data model
*Was PQ-10. Owning document: `SECURITY.md` (SEC-AUTHN-11) and `REQUIREMENTS.md` (Data Entities).*

**Answer:** Add a **`Device` entity**: opaque random identifier, first-seen and last-seen timestamps, a
per-account cap with least-recently-seen eviction, **purged with the account** (FR-9.3) and by the
inactivity purge (FR-9.5), **included in the account export** (FR-9.2), and **disclosed in the privacy
policy** (SEC-DATA-7). On mobile the app generates the identifier and holds it in platform secure storage.

### Q12 · "Content flagged sensitive" predicate
*Was PQ-14. Owning document: `DESIGN.md` **On-Screen Privacy**, with the classification in
`REQUIREMENTS.md` **Data Entities**.*

**Answer:** **Entity-level classification.** Health issues, progress updates, treatments, appointments,
journal entries, and document titles and notes are flagged sensitive. Case, task, contact, and expense
metadata is not. Masking applies to the flagged set, and SEC-SESSION-7's keyboard-flag rule consumes the
same classification.

---

## Batch 4 — architecture and platform (ANSWERED)

*All four belong in `ARCHITECTURE.md` **Technology Decisions**, except Q16 which is a new `SECURITY.md` rule.*

### Q13 · AWS region
**Answer:** **Single EU region, `eu-west-1`.** All stores, workers, and backups in Ireland, so personal
data never leaves the EU under D-10's GDPR-grade bar. CloudFront still terminates near users worldwide.

### Q14 · Compute runtime
**Answer:** **ECS on Fargate, two separate services** — one for the REST API application, one for the
background worker, with separate task definitions and IAM roles. This *is* the SEC-FILE-6 parsing
isolation and the SEC-SECRET-2 least-privilege split, expressed in infrastructure.

### Q15 · Managed PostgreSQL edition
**Answer:** **Amazon RDS for PostgreSQL, Multi-AZ.** Point-in-time recovery and automated backups meet
NFR-3.2; Multi-AZ supports NFR-3.1 and NFR-3.3. Standard PostgreSQL, so the 3NF schema, exact NUMERIC
money, and full-text search are unaffected.

### Q16 · Transport protection inside boundary 2
*New rule for `SECURITY.md`.*

**Answer:** **TLS is required on every boundary-2 hop** — database connections use TLS with full
certificate verification, and file-store access is HTTPS-only. Closes the gap against T-13's malicious or
compromised infrastructure operator and complements SEC-DATA-1's at-rest encryption.

---

## Batch 5 — tooling and operational values (ANSWERED)

### Q17 · Test tooling
*New rows for `ARCHITECTURE.md` **Technology Decisions**.*

**Answer:** **Vitest** (web unit/component), **Playwright** (web end-to-end, and the SEC-FILE-2 inert-render
tests), **JUnit 5** (Quarkus/Kotlin), the **Compose Multiplatform test framework** (mobile), **axe-core**
(automated NFR-5.1 checks), **k6** (NFR-4.1/NFR-4.3 budgets and SEC-HTTP-6's load-test verification).

### Q18 · TLS policy and WAF configuration
*For `SECURITY.md` alongside SEC-HTTP-1, and `ARCHITECTURE.md`'s Edge row.*

**Answer:** Pin CloudFront and the load balancer to the AWS-managed **`TLSv1.2_2021`** policy. WAF runs the
**AWS Managed Rules Common and Known Bad Inputs** sets plus a **rate-based rule** for volumetric protection.

### Q19 · Security header values
*For `SECURITY.md` SEC-HTTP-3.*

**Answer:** `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`;
`Referrer-Policy: strict-origin-when-cross-origin`; `Permissions-Policy` denying camera, microphone,
geolocation, and payment; CSP **`default-src 'none'`** with explicit self-only allowances for script,
style, image, and connect, plus `frame-ancestors 'none'`.

### Q20 · Field length limits
*For `REQUIREMENTS.md` **Data Entities**, enforced by SEC-INPUT-1.*

**Answer:** Titles and names **200**; notes and outcome notes **10,000**; journal entries **50,000**; tags
**50 characters each, at most 25 per document**; reference numbers **100**.

---

## Batch 6 — logging, email, and rate-limit semantics (ANSWERED)

### Q21 · Rate-limit windows and suppressed security notices
*For `SECURITY.md` SEC-HTTP-6.*

**Answer:** All "per day" limits are **rolling 24-hour windows**. **SEC-AUTHN-11 security notices are
exempt from the 20/day per-recipient email cap** — they are the channel that warns a victim of a takeover,
and SEC-AUTHN-11 already forbids suppressing them. Reminder and other mail stays capped.

### Q22 · "Persistent" delivery failure
*For `SECURITY.md` SEC-EXT-4.*

**Answer:** **Any hard bounce raises the in-app notice immediately; soft failures require three consecutive
failures** to the same address. Avoids false alarms from a transient hiccup while still catching a dead
mailbox well before a filing deadline (T-49).

### Q23 · Who writes the security event log, and what happens if the write fails
*For `SECURITY.md` SEC-LOG-1 / SEC-LOG-4, with the component assignment in `ARCHITECTURE.md`.*

**Answer:** **The component performing the action writes its own entry** through a shared append-only
client — so Background processing logs its own purges, deletions, and exports. If the entry cannot be
written, **security-significant operations are refused** (fail-closed, per SEC-AUTHZ-2's doctrine), with a
**short bounded local buffer** so a momentary sink outage does not stop the product.

### Q24 · Security activity view
*For `SECURITY.md` SEC-LOG-3, with the read path in `ARCHITECTURE.md`.*

**Answer:** The view reads a **first-party projection in PostgreSQL**, written alongside the append-only
log, so the user-facing query path never holds read access to the tamper-evident store. It shows the
**full 12-month** SEC-LOG-1 retention window, with **source IPs truncated** as SEC-LOG-1 already truncates
them on deletion.

---

## Batch 7 — product behavior (ANSWERED)

### Q25 · Case overview specifics
*For `REQUIREMENTS.md` FR-2.4.*

**Answer:** The overview shows the **10 most recent timeline events**, and **"upcoming deadlines" means
tasks flagged as hard deadlines** (FR-5.7) falling due within 7 days — not every dated task, which the
overdue count already covers.

### Q26 · Second-passkey prompt cadence
*For `REQUIREMENTS.md` FR-1.11 and a `DESIGN.md` prompt convention.*

**Answer:** A **dismissible banner on the case list after each sign-in** while exactly one passkey is
registered; dismissal lasts for that session only.

### Q27 · Archiving
*For `REQUIREMENTS.md` FR-2.5 and FR-8.1.*

**Answer:** **Archive and unarchive each emit an FR-8.1 timeline event**, so a case leaving the default
list is never a silent state change (T-12). Archived cases are reached through an **explicit filter on the
existing case list**, not a separate screen.

### Q28 · Undo and restore surface
*For `REQUIREMENTS.md` FR-2.6, FR-3.8, FR-8.4, and the new §4 deletions from Q3.*

**Answer:** **One Trash surface per case** listing every soft-deleted record in it — documents, journal
entries, health records, tasks, contacts, expenses — each with its remaining window and a restore action;
**deleted cases live in an account-level trash**. Generalizes FR-3.8's trash area rather than creating
parallel per-type concepts.

---

## Batch 8 — lifecycle, measurement, distribution, legal (ANSWERED)

### Q29 · Provider credential removal and purge alerting
*For `REQUIREMENTS.md` FR-9.3 and `ARCHITECTURE.md`'s Identity component.*

**Answer:** Purge execution **deletes the Cognito user first**, removing passkey credential material inside
the FR-9.3 30-day bound, then clears first-party stores — so a partial failure cannot leave usable
credentials behind. **A purge that misses the 30-day or 90-day bound raises a high-severity operator alert.**

### Q30 · Load profile
*Recorded with the k6 configuration in the repository, not in a specification document.*

**Answer:** Fix a reference profile — read-weighted request mix, realistic think time, and a representative
account of a few hundred records and dozens of documents — **versioned alongside the load-test config** so
runs stay comparable. A test-design value, so **no requirement changes** and NFR-4.1's budget is untouched.

### Q31 · Mobile signing custody
*For `ARCHITECTURE.md` (Mobile client / distribution) and `SECURITY.md` SEC-SESSION-7.*

**Answer:** **Google Play App Signing and Apple-managed certificates hold the distribution keys; CI holds
only upload keys**, injected from AWS Secrets Manager over the existing GitHub Actions OIDC path
(SEC-SECRET-1). Store-listing metadata is version-controlled in-repo. A CI compromise (T-35) cannot then
produce a signed release impersonating the app.

### Q32 · Regulatory and legal review
*For `REQUIREMENTS.md` and `SECURITY.md`, gated on the SEC-DEPLOY-4 release checklist.*

**Answer:** **Commission one pre-launch legal review** covering health-content and unlicensed-practice
exposure, the sufficiency of the FR-4.9 and FR-5.8 notices, and accessibility obligations in the launch
markets. Outcomes are recorded in the owning documents. Until it reports, the **Regulatory fields stay
`TO BE DECIDED`** — an honest unassessed state rather than a guessed mapping.

---

## Batch 9 — follow-ups raised by the answers (ANSWERED)

### Q33 · Soft-delete for tasks, contacts, and expenses
*For `REQUIREMENTS.md` FR-5.4, FR-6.3, FR-7.2, and NFR-5.5.*

**Answer:** **Yes — all three gain the same 30-day undo window.** One deletion semantic across every record
type, the trash surface from Q28 is complete, and an expense (evidence of real money) survives a mis-tap.
NFR-5.5's confirmation copy extends to state the window for these three.

### Q34 · Device cap
*For `REQUIREMENTS.md` **Data Entities** (`Device`) and `SECURITY.md` SEC-AUTHN-11.*

**Answer:** **10 devices per account**, least-recently-seen eviction. Covers a phone, tablet, laptop, work
machine, and several browser profiles, so eviction is rare and a re-notification is worth attention.

## Batch 10 — raised by applying the decisions (ANSWERED)

### Q35 · Bounds for the remaining short free-text fields
*For `REQUIREMENTS.md` **Data Entities**.*

**Answer:** A **short-text class of 500 characters** covering Case and Appointment location, Appointment
purpose, Health Issue body area, and Contact address; **phone and email at 200**. Completes the table with
one more class rather than a bespoke number per field.

### Q36 · NFR-6.2 wording after the minor-unit change
*For `REQUIREMENTS.md` NFR-6.2.*

**Answer:** Restate as **"accurate to the case currency's ISO 4217 minor unit"**. A wording correction —
"to the cent" became literally false for JPY and KWD once Q6 landed. No behavior changes for a
two-decimal currency.

### Q37 · Whether restoring a record re-establishes severed links
*For `REQUIREMENTS.md` FR-6.3 and the trash requirements.*

**Answer:** **Restoring within the window re-establishes the links** severing removed, so an undo genuinely
undoes. The text snapshot remains as the fallback once a record is purged for good.

### Q38 · Undo for Progress Updates and Treatments
*For `REQUIREMENTS.md` FR-4.11.*

**Answer:** **They get the same 30-day undo** and appear in the case trash. One deletion semantic across
every record type, and a progress note is health evidence a user may need back. Supersedes the agent's
flagged inference that these two deleted immediately.

---

## Batch 11 — final gaps (ANSWERED)

### Q39 · Page size
*For `ARCHITECTURE.md` (API shape), referenced by `REQUIREMENTS.md` FR-10.1.*

**Answer:** **Default 50, maximum 200.**

### Q40 · How long a reveal of masked content lasts
*For `DESIGN.md` **On-Screen Privacy**.*

**Answer:** **Per-element, and re-masks on navigation.** Matches the DQ-6 shoulder-surfing threat directly:
an observer sees at most the single field the user deliberately opened.

### Q41 · Where the trash surfaces live
*For `DESIGN.md`, consumed by the client issues.*

**Answer:** The **per-case trash is a destination inside the case**, alongside its other sections; the
**account-level trash for deleted cases lives in account settings**. Each surface sits at the level of the
thing it restores.

---

## Batch 12 — residuals surfaced while propagating into the drafts (ANSWERED)

These were invisible in the original scan; they only became visible once each issue was specified in
detail. The drafting agents recorded them rather than inventing answers.

### Q42 · Future journal entry dates
*For `REQUIREMENTS.md` FR-8.2 and SEC-INPUT-1.* **Answer: reject them** — the same not-in-future rule
FR-2.1 and FR-4.1 already carry. A journal is a contemporaneous record; a mistyped year would otherwise
pin an entry above everything real in the FR-8.3 ordering forever.

### Q43 · Whether a timeline event survives the purge of its source record
*For `REQUIREMENTS.md` FR-8.1 / FR-8.4.* **Answer: deletion events survive the purge**, as D-17 already
rules for journal entries; other events go with their source. Keeps the timeline honest about what was
removed — the T-12 anti-tampering purpose — without retaining the content.

### Q44 · Notification `type` value set
*For `REQUIREMENTS.md` **Data Entities**.* **Answer: a closed set of the four kinds the specs already
describe** — reminder, security message, export ready, delivery failure — validated by SEC-INPUT-1 like
every other enumeration.

### Q45 · How a notification becomes read
*For `REQUIREMENTS.md` **Data Entities** / the feed requirement.* **Answer: explicit per-item, plus a
mark-all-read action.** An item becomes read when the user opens it or its target record. Nothing is
silently consumed by glancing at the feed — which matters when the feed carries a takeover warning (T-27).

### Q46 · Readiness signal for the case-summary PDF
*For `REQUIREMENTS.md` FR-9.1.* **Answer: the same asynchronous generation plus in-app ready
notification FR-9.2 uses.** One export model for both, and a 60-second budget stops being an HTTP wait an
injured user watches on a spinner.

### Q47 · Whether the export scope statement is embedded in the artifact
*For `REQUIREMENTS.md` FR-9.4.* **Answer: yes — in-app *and* inside the PDF and archive.** T-18's harm is
a recipient drawing conclusions from an export whose limits they cannot see; only the embedded copy
reaches the insurer or attorney.

### Q48 · Default for the account-wide email-reminder toggle
*For `REQUIREMENTS.md` FR-5.6.* **Answer: on by default.** The product exists to stop an injured user
missing deadlines, and FR-5.7's escalating reminders are worth little if the channel is silent until
someone finds a settings screen.

### Q49 · Logging a notification-preference change
*For `SECURITY.md` SEC-LOG-1.* **Answer: add it to the logged minimum** so it appears in the SEC-LOG-3
activity view — no additional SEC-AUTHN-11 email. Closes the T-49 silent-sabotage path without diluting
the security notices.

### Q50 · Scope of the delivery-failure in-app notice
*For `SECURITY.md` SEC-EXT-4.* **Answer: extend it to security and account mail**, not reminders only.
Persistent failure of any outbound message to the account address raises the notice — a mailbox attacker
filtering security notices (T-29, T-49) is exactly what the control exists for.

### Q51 · Whether restore reconciliation re-revokes at the identity provider
*For `SECURITY.md` SEC-DATA-8.* **Answer: yes.** The re-apply step covers credential revocations at the
provider, not just first-party rows; otherwise T-48 stands — a restore silently re-arms a passkey the user
revoked because it was compromised.

### Q52 · Schema migration tool
*For `ARCHITECTURE.md` **Technology Decisions**.* **Answer: Flyway.** Versioned plain-SQL migrations with
first-class Quarkus integration, reviewable against the 3NF schema — which matters when a migration
touches health data.

### Q53 · Category on starter-checklist tasks
*For `REQUIREMENTS.md` FR-5.1.* **Answer: yes — each of the nine carries its natural FR-5.2 category**
(notify insurance → insurance, obtain the police report → legal, seek a medical evaluation → medical, and
so on). The checklist is most users' first encounter with tasks and should model the categorisation the
feature relies on.

### Q54 · Timeline event when a task completion is reverted
*For `REQUIREMENTS.md` FR-5.3 / FR-8.1.* **Answer: yes — reverting emits its own event.** FR-8.1 exists so
state changes leave no silent trace (T-12); a completion that quietly disappears from a deadline-tracking
product is exactly what the timeline is for. Symmetric with the completion event already required.

---

## Batch 16–17 — final residuals from the second propagation (ANSWERED)

### Q55 · Completion budget for uploads above 25 MB
*For `REQUIREMENTS.md` NFR-4.2.* **Answer: restate the budget as a throughput floor** — 25 MB within 60
seconds on 10 Mbps, and larger files at no worse than that effective rate, giving 500 MB a derived bound
of roughly 20 minutes. One rule covers every size and stays testable.

### Q56 · A reminder whose target is in the trash at fire time
*For `REQUIREMENTS.md` FR-5.6.* **Answer: suppressed while trashed, and not fired retroactively on
restore.** The user deleted the record; a notification about it would confuse, and a restore should not
produce a burst of stale alerts.

### Q57 · A reminder missed because the worker was down
*For `REQUIREMENTS.md` FR-5.6.* **Answer: delivered late within a one-hour grace window**, marked late;
older ones are skipped and logged. A deadline product should not silently swallow a reminder, but a
day-old alert for a passed appointment is noise.

### Q58 · How the client learns of new feed items
*For `ARCHITECTURE.md` (client components).* **Answer: poll on an interval while foregrounded, plus
refresh on navigation**, at an interval comfortably inside FR-5.6's five-minute budget. No long-lived
connection through the edge, and it degrades gracefully on poor mobile connectivity.

### Q59 · Flagging a hard deadline fewer than 30 days out
*For `REQUIREMENTS.md` FR-5.7.* **Answer: skip escalation points already passed**; schedule only those
still ahead. Firing "30 days remaining" for something due in 10 would mislead a stressed user.

### Q60 · Editing the due date of a hard-deadline task
*For `REQUIREMENTS.md` FR-5.7.* **Answer: recompute the ladder against the new date**, applying the same
skip rule; escalations already delivered are not re-sent. A deadline that moves carries its warnings with
it.

---

**Interview closed at Q60. Every question is answered; nothing is deferred.**

---

## Bookkeeping corrections (no decision needed)

Two sentences in `REQUIREMENTS.md` went stale because the four documents were updated in parallel:
its claim that `DESIGN.md`'s DQ-6 treatment is incomplete (DESIGN.md now defines the predicate), and its
reference to "sixteen" security questions (`SECURITY.md` now records nineteen, SQ-17–SQ-19 having been
added and resolved). Both corrected.

## Editorial placements (no decision needed)

- The user profile/settings capability implied by Q1 and Q2 (set timezone, set locale) is recorded as a new
  requirement in `REQUIREMENTS.md` §1 Accounts & Authentication, where the other account-level operations
  already live, rather than as a new section.

## Status

**Closed.** 60 questions asked across 17 batches; all 60 answered, none deferred.

The interview ran in four waves, each exposing the next:

1. **Q1–Q32** — the initial repository sweep: open-question sections, `TO BE DECIDED` / `UNKNOWN` /
   `ASSUMPTION` markers, provisional defaults, blank required inputs, and threat-model rows needing a
   decision, deduplicated from 175 drafts.
2. **Q33–Q41** — raised by the answers themselves, and by applying them to the specifications.
3. **Q42–Q54** — surfaced while propagating into the issue drafts, when each requirement was specified in
   enough detail for the gap to become visible.
4. **Q55–Q60** — the last residuals, found the same way in the second propagation pass.

Recorded as `D-18`–`D-58` in `REQUIREMENTS.md`; `SQ-17`–`SQ-19` and `SEC-TRUST-4` in `SECURITY.md`; new
technology rows, the paginated list contract, and the feed polling contract in `ARCHITECTURE.md`; the
masking predicate, reveal scope, and trash conventions in `DESIGN.md` with `style-guide.html` updated to
match. `OQ-9`–`OQ-14` are closed with a resolution map, and `CQ-4` tracks the one marker retained by
decision.

**The single retained marker.** `D-39` commissions a pre-launch legal review; until it reports, the
**Jurisdiction / Regulatory Scope** and **Regulatory** fields stay `TO BE DECIDED` across the issue set,
because a guessed regulation would read as an assessed one. The **OWASP ASVS 5.0.0** and **NIST SP 800-53**
rows stay unmapped for the same reason — `REQ-PLATFORM-100`'s self-assessment produces the ASVS
identifiers, and no 800-53 baseline has been selected. These are decisions to retain a marker, not
omissions.
