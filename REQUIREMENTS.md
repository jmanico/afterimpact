# AfterImpact — Requirements

AfterImpact helps a person manage the aftermath of a car accident: paperwork, accident-related health issues, and getting life back on track.

This document owns WHAT the system must do. Security and privacy requirements are authored in `SECURITY.md`; the visual and interaction language is `DESIGN.md`; components, technology choices, and data flow are `ARCHITECTURE.md`. Requirements elaborate the recorded decisions in **Decisions**; `TO BE DECIDED` marks an unresolved value inside an otherwise clear requirement.

## Scope

In scope (v1): everything in **Functional Requirements** below — single-user accounts, accident cases, document storage, health tracking, tasks/deadlines/reminders, contacts, expenses, journal/timeline, and export.

Out of scope (v1):

- Multi-user access of any kind: sharing, collaboration, helper/attorney accounts (D-2, D-11).
- Integrations with insurers, healthcare systems, or any other third-party data source or service (D-7; the two external service dependencies D-7 permits — email delivery and the managed identity service — are infrastructure, not integrations).
- Automated claim filing or form generation for specific insurers/agencies.
- Medical or legal advice of any kind (FR-4.9, FR-5.8).
- Payments or monetization features.
- Languages other than English and Spanish (NFR-5.4); currencies other than one per-case currency (FR-7.5) (D-15).
- SMS and push notification channels (D-13).

## Data Entities

Logical model only; storage design belongs to `ARCHITECTURE.md`.

- **User** — email (unique), registered passkey credentials, notification preferences, timezone.
- **Case** — title, accident date (required), accident time/location/description (optional), currency (one per case, set at creation — FR-7.5), status (open, archived), owned by one User. All entities below belong to exactly one Case (except User).
- **Document** — original file, title, category, optional document date, notes, tags, links to Contact/Health Issue/Expense.
- **Health Issue** — name, optional body area, onset date, severity (0–10), status (active, improving, resolved), description; has Progress Updates and Treatments.
- **Progress Update** — timestamp, note, optional severity (0–10).
- **Treatment** — name, start/end dates, prescribing provider (Contact), notes.
- **Appointment** — date/time, provider (Contact, optional), location, purpose, linked Health Issues, outcome notes.
- **Task** — title, optional due date, category, notes, done flag + completion timestamp, hard-deadline flag, linked records.
- **Reminder** — target record (Task or Appointment), fire datetime, delivery state.
- **Contact** — name, role (insurance adjuster, insurance company, attorney, medical provider, repair shop, other driver, witness, employer, other), phone/email/address, reference numbers (e.g., policy #, claim #), notes.
- **Expense** — amount, date, category, payment status, optional payee (Contact), linked receipt Document, linked Health Issue, reimbursement records.
- **Journal Entry** — user-authored text, entry date (backdatable), created/edited timestamps, optional linked Contact/records.
- **Timeline Event** — system-generated record of a case action (see FR-8.1).
- **Export Job** — requested scope, status, produced artifact.
- **Notification** — an in-app feed item (type, target record, created timestamp, read state): a reminder or a security message (SEC-AUTHN-11) surfaced to the user. A notification tied to a case or record is removed with it (FR-2.6) and with the account (FR-9.3), and notifications are included in the account export (FR-9.2). Modeling the feed as a data entity lets retention, purge, and export rules attach to items that may carry health-adjacent text (T-43).

## Functional Requirements

### 1. Accounts & Authentication

The credential ceremonies, session mechanics, and enumeration-resistance controls that implement this section are specified in `SECURITY.md` **Authentication** and **Session management**.

- **FR-1.1** The system MUST allow a person to register with an email address and MUST send a verification email; the account MUST NOT be usable for sign-in until the email address is verified.
- **FR-1.2** The system MUST authenticate users with passkeys (WebAuthn) and MUST NOT offer passwords or any other shared-secret credential (D-9).
- **FR-1.3** The system MUST sign in a user who completes a valid authentication ceremony with a registered credential on a verified account, and MUST show the same generic error for every failed sign-in (SEC-AUTHN-5 owns what a failure may disclose).
- **FR-1.4** After 5 consecutive failed authentication attempts for an account within 15 minutes, the system MUST block further attempts for that account for at least 15 minutes, and MUST log the event (SEC-LOG-1). The block applies to failed or invalid attempts and MUST NOT prevent a successful valid passkey ceremony by the owner — a valid WebAuthn assertion is cryptographic proof of possession, not a guess (T-1). Because an account-scoped block is otherwise a denial-of-service lever anyone who knows the email can renew, source-based and anti-automation throttling applies ahead of the account block (SEC-AUTHN-4; `SECURITY.md` owns the limit values).
- **FR-1.5** The system MUST let a signed-in user sign out, immediately invalidating that session.
- **FR-1.6** The system MUST provide account recovery via one-time recovery codes for a user who has lost access to every registered passkey (D-9). The recovery flow itself is specified in `SECURITY.md` (SEC-AUTHN-6).
- **FR-1.7** The system MUST let a signed-in user change their email address, requiring verification of the new address before the change takes effect.
- **FR-1.8** The system MUST let a signed-in user register additional passkeys, view their registered credentials, and revoke any of them (SEC-AUTHN-8 governs the revocation guard and notification; SEC-SESSION-5 the session consequences).
- **FR-1.9** Every create/read/update/delete/export operation MUST be permitted only for the authenticated owner of the data. SEC-AUTHZ-1 through SEC-AUTHZ-5 own how that is enforced and what a denied request may disclose.
- **FR-1.10** Sessions MUST expire after 7 days regardless of activity and MUST expire after 1 hour of inactivity.
- **FR-1.11** While an account has exactly one registered passkey, the system MUST prompt the signed-in user to register a second one (FR-1.8), so the loss of a single credential does not force account recovery (T-2).

### 2. Accident Cases

- **FR-2.1** The system MUST allow a user to create a case with an accident date (required, MUST NOT be in the future) and optional time, location, description, and title; when no title is given the system MUST derive one from the accident date.
- **FR-2.2** The system MUST support multiple cases per user and MUST list them with open cases first, newest accident date first.
- **FR-2.3** The system MUST allow the user to edit any case field at any time, subject to the same validation as creation.
- **FR-2.4** The system MUST show a per-case overview containing at minimum: count of open tasks, count of overdue tasks, next upcoming appointment, upcoming deadlines within 7 days, expense totals (FR-7.3), and the most recent timeline events.
- **FR-2.5** The system MUST allow the user to archive a case (hiding it from the default list) and to reverse the archive at any time with no data loss.
- **FR-2.6** The system MUST allow the user to delete a case only after a fresh passkey re-authentication (as FR-9.3 requires for account deletion; `SECURITY.md` owns the step-up ceremony) and an explicit confirmation that states all contained records will be removed; deletion MUST remove all case data, and MUST be undoable for 30 days before becoming permanent.

### 3. Documents & Paperwork

- **FR-3.1** The system MUST let the user upload files of types PDF, JPEG, PNG, HEIC, WebP, DOCX, XLSX, TXT, MP4, and MOV, up to 25 MB per file — except video (MP4, MOV), which may be up to 500 MB per file (dashcam and scene clips are in scope) — and MUST reject any other type or oversize file with an error naming the violated rule.
- **FR-3.2** Each document MUST have a title (defaulting to the filename) and exactly one category from: police & legal, insurance, medical record, bill/invoice, repair & vehicle, correspondence, receipt, identity, other; and MAY have a document date, notes, tags, and links to a contact, health issue, or expense.
- **FR-3.3** The system MUST list a case's documents with filtering by category, tag, and date range, and sorting by document date or upload date.
- **FR-3.4** The system MUST provide text search over document titles, notes, and tags within a case. File contents are not indexed in v1 (no OCR); content search is revisited against NFR-4.3, and `ARCHITECTURE.md` owns the search technology.
- **FR-3.5** The system MUST let the user view PDFs and images in the app and download any document as the byte-identical original file.
- **FR-3.6** The system MUST let the user edit a document's metadata at any time without altering the stored file.
- **FR-3.7** The system MUST let the user replace a document's file while retaining access to prior versions.
- **FR-3.8** Deleting a document MUST require confirmation; deleted documents MUST be recoverable from a trash area for 30 days before permanent removal.
- **FR-3.9** The system MUST scan every upload for malware and MUST reject a flagged file with a notice to the user (SEC-FILE-4 owns quarantine handling; `ARCHITECTURE.md` owns the scanning engine).
- **FR-3.10** The system MUST enforce a per-user storage quota of 25 GB, MUST reject uploads that would exceed it with an error showing current usage, and MUST display current usage on request. Trashed documents awaiting purge (FR-3.8), retained prior versions (FR-3.7), and generated export artifacts (FR-9.4) all count toward usage, and the usage display MUST break usage down across these so space is reclaimable — otherwise a session-holding attacker could silently exhaust the quota with unreclaimable space and block new evidence uploads before a deadline (T-21).

### 4. Health Issues & Appointments

- **FR-4.1** The system MUST let the user record a health issue with a name (required) and optional body area, onset date (MUST NOT be in the future), severity (integer 0–10), and description.
- **FR-4.2** A health issue MUST carry a status of active, improving, or resolved; the user MUST be able to change status at any time, and each change MUST be timestamped and visible in the issue's history.
- **FR-4.3** The system MUST let the user add timestamped progress updates (note plus optional severity 0–10) to a health issue and MUST display them in chronological order; severity over time SHOULD be shown as a chart.
- **FR-4.4** The system SHOULD let the user record treatments/medications on a health issue (name, start/end dates, prescribing provider, notes) as recordkeeping only.
- **FR-4.5** The system MUST let the user link providers (contacts), documents, expenses, and appointments to a health issue and navigate between linked records.
- **FR-4.6** The system MUST let the user create appointments with date/time (required; past or future), and optional provider (contact), location, purpose, and linked health issues; and MUST show an upcoming-appointments list per case.
- **FR-4.7** Creating a future appointment MUST create a default reminder 24 hours before it (via FR-5.6), which the user MAY change or remove.
- **FR-4.8** The system MUST let the user record outcome notes on an appointment after it occurs.
- **FR-4.9** The system MUST NOT present diagnoses, treatment recommendations, or any other medical advice; health features are recordkeeping only.

### 5. Recovery Tasks, Deadlines & Reminders

- **FR-5.1** On case creation the system MUST offer a starter recovery checklist that the user can accept in full, in part, or skip; it MUST include at least: notify insurance company; obtain the police/accident report; photograph vehicle damage; seek a medical evaluation; obtain repair estimates; arrange alternate transportation; notify employer and track missed work; review policy coverage and filing deadlines; start tracking expenses. This nine-item list is the final v1 content (D-12); the FR-5.8 guidance notice applies.
- **FR-5.2** The system MUST let the user create a task with a title (required) and optional due date, notes, category (insurance, legal, medical, vehicle, financial, personal), and links to case records.
- **FR-5.3** The system MUST let the user mark a task complete or not complete; completion MUST be timestamped and MUST appear on the timeline (FR-8.1).
- **FR-5.4** The system MUST let the user edit and delete tasks.
- **FR-5.5** The system MUST visibly flag tasks whose due date has passed without completion, and include them in the case overview's overdue count (FR-2.4).
- **FR-5.6** The user MUST be able to set one or more reminder date-times on any task or appointment; the system MUST deliver an in-app notification within 5 minutes of each reminder time and SHOULD also deliver email; the user MUST be able to turn email reminders on or off account-wide.
- **FR-5.7** The user SHOULD be able to flag a dated task as a hard deadline, causing a visible countdown and escalating reminders at 30, 7, and 1 day(s) before the due date.
- **FR-5.8** All guidance content (including the starter checklist) MUST be accompanied by a notice that it is general organizational guidance, not legal or medical advice.

### 6. Contacts

- **FR-6.1** The system MUST let the user add a contact to a case with a name (required), one role from the set in **Data Entities**, and optional phone, email, address, reference numbers (e.g., policy or claim numbers), and notes.
- **FR-6.2** Contacts are case-scoped: a contact MUST belong to exactly one case; a person involved in more than one case is entered in each (D-8).
- **FR-6.3** The system MUST let the user edit and delete contacts; deleting a contact MUST NOT delete journal entries, documents, or other records that referenced it — those records MUST retain the contact's name as text. The deletion confirmation (NFR-5.5) MUST disclose that the contact's name is retained as text on referencing records and therefore continues to appear in exports handed to third parties (T-42).
- **FR-6.4** The system MUST list a case's contacts filterable by role.

### 7. Expenses

- **FR-7.1** The system MUST let the user record an expense with amount (required, > 0, two decimal places) and date (required), one category from: medical, vehicle repair, rental/transport, lost wages, legal, insurance, other; a payment status of unpaid, paid, submitted for reimbursement, or reimbursed; and optional payee (contact), notes, linked receipt document, and linked health issue.
- **FR-7.2** The system MUST let the user edit expenses and delete them after confirmation.
- **FR-7.3** The system MUST show per-case expense totals: overall, by category, by payment status, and the total not yet reimbursed.
- **FR-7.4** The system SHOULD support recording partial reimbursements (amount + date) against an expense and MUST, where supported, show the remaining balance.
- **FR-7.5** Each case records exactly one currency, chosen at case creation; all amounts and totals (FR-7.3) within a case use that currency, and the system MUST NOT perform exchange-rate conversion (D-15).
- **FR-7.6** The system MUST export a case's expense list as CSV including all fields in FR-7.1 and the totals in FR-7.3.

### 8. Timeline & Journal

- **FR-8.1** The system MUST automatically record a timestamped timeline event when, at minimum: a case is created or edited; a document is added, has its file replaced (FR-3.7), or has its metadata edited (FR-3.6); a task is completed; an appointment is created or given outcome notes; an expense is added; a health issue is created or changes status; and any record is deleted while its case survives. Recording document replacement, metadata edits, and deletions closes a silent-tampering gap: without them, an attacker with a stolen session could substitute a doctored evidence file or delete records leaving no user-visible trace on the timeline (T-12, T-47). Records additionally carry independent tamper-evidence (content hashes, protected history) as specified in `SECURITY.md` (SEC-FILE-5).
- **FR-8.2** The system MUST let the user write journal entries (free text) with an entry date that MAY be in the past (backdating for events recalled later), and optional links to a contact or other case records — e.g., a summary of a phone call with an adjuster.
- **FR-8.3** The system MUST display timeline events and journal entries together in date order (newest first by default) with filtering by type and date range.
- **FR-8.4** Journal entries MUST visibly show both their creation timestamp and last-edited timestamp; the system MUST retain and display prior versions of edited entries. Users MUST NOT be able to edit or delete system-generated timeline events (they are removed only with their case or source record). A user MAY delete a journal entry after explicit confirmation (NFR-5.5); the deletion MUST be undoable for 30 days, and a permanent timeline event MUST record that an entry was deleted — the content is gone, the deletion stays visible (D-17, T-14).

### 9. Export & Data Management

- **FR-9.1** The system MUST generate a case summary as a PDF containing user-selected sections from: case details, contacts, document index, health issues with progress history, appointments, task status, expense report (FR-7.3), and the full timeline; generation MUST complete within 60 seconds and the file MUST be downloadable in-app.
- **FR-9.2** The system MUST let the user export all of their account data as a downloadable archive containing a machine-readable form (JSON or CSV) of every record plus every original uploaded file; the export runs asynchronously and MUST notify the user in-app when ready.
- **FR-9.3** The system MUST let the user delete their account after a fresh re-authentication and an explicit confirmation; all personal data MUST be permanently deleted within 30 days, and within 90 days from backups. The system MUST provide a 14-day cancellation window before purge begins — deletion initiated, notification email sent to the verified address (SEC-AUTHN-11), cancellable in-app for 14 days with a one-click cancel offered at sign-in, then purge — so that neither an adversary with the unlocked device nor the injured user in error can irreversibly destroy the entire evidentiary record with no notice and no way back (T-46). The window fits inside the 30/90-day deletion bounds above.
- **FR-9.4** Generated export artifacts stored in the file store — full-account archives (FR-9.2), case-summary PDFs (FR-9.1), and expense CSVs (FR-7.6) — MUST be purged by the background worker 7 days after generation; the user MAY regenerate any export at any time. Exports cover current records only: prior document versions (FR-3.7), prior journal versions (FR-8.4), and soft-deleted or trashed records are excluded, and each export MUST state this scope to the user, since a stale archive is a maximum-sensitivity aggregate that can silently retain records the user later deleted and can leak superseded drafts to attorneys and insurers if its scope is undeclared (T-18). Journal creation and last-edited timestamps (FR-8.4) ARE included in exports, and the system MUST show a one-time notice at journal-authoring time that these timestamps are permanent and appear in exports (T-42; `DESIGN.md` owns the notice convention). Export artifacts count toward the FR-3.10 quota.
- **FR-9.5** The system MUST purge accounts that have been inactive for 3 years, and MUST send warning emails to the verified address approximately 60 days and approximately 30 days before the purge (D-16).

## Non-Functional Requirements

Security (`NFR-1`) and privacy (`NFR-2`) requirements are authored in `SECURITY.md`, which owns them along with the controls that enforce them. Their IDs are unchanged and remain the reference mechanism.

### 3. Reliability

- **NFR-3.1** The service MUST target at least 99.5% monthly availability, excluding announced maintenance.
- **NFR-3.2** Automated backups MUST bound data loss to at most 24 hours (RPO ≤ 24 h) and restoration time to at most 12 hours (RTO ≤ 12 h); restores MUST be tested at least quarterly.
- **NFR-3.3** An upload acknowledged to the user as successful MUST remain durable and retrievable despite the failure of any single storage component.

### 4. Performance

- **NFR-4.1** Interactive reads and writes (excluding uploads, search, and exports) MUST complete in under 2 seconds at the 95th percentile under normal load of approximately 1,000 concurrent users (D-14).
- **NFR-4.2** A 25 MB upload MUST complete within 60 seconds on a 10 Mbps connection.
- **NFR-4.3** Document search (FR-3.4) MUST return results within 3 seconds at the 95th percentile.

### 5. Usability & Accessibility

- **NFR-5.1** All user-facing screens MUST conform to WCAG 2.2 Level AA. `DESIGN.md` **Accessibility** owns the design-language rules that achieve this.
- **NFR-5.2** All functionality MUST be usable on both mobile (viewport ≥ 360 px wide) and desktop form factors.
- **NFR-5.3** Error messages shown to users MUST state in plain language what happened and what the user can do next; raw technical errors MUST NOT be shown.
- **NFR-5.4** v1 ships in English and Spanish (D-15); user-facing copy SHOULD target roughly an 8th-grade reading level in every locale, since users may be injured, stressed, or medicated.
- **NFR-5.5** Every destructive action (delete case/document/contact/expense/account) MUST require explicit confirmation before execution. The confirmation copy MUST truthfully state the recovery timeline: the 30-day recovery window where one applies (FR-2.6, FR-3.8, FR-8.4), that deletion becomes permanent afterward, and that purged records persist in backups only for the bounded period SEC-DATA-8 sets (resolves CQ-3's copy conflict, T-33).

### 6. Data Integrity

- **NFR-6.1** Every record MUST carry created and last-updated timestamps, stored in UTC and displayed in the user's timezone.
- **NFR-6.2** Monetary values MUST be stored and computed exactly (no floating-point rounding errors); all totals MUST be accurate to the cent.

## Decisions

Decisions that requirements elaborate. Any may be revisited; requirements cite them for traceability.

- **D-1** "Paperwork" = user-uploaded files with metadata, categories, search, and export (FR-3), plus date-driven obligations handled as tasks/deadlines (FR-5). Structured claim-form authoring is out of scope v1.
- **D-2** v1 is strictly single-user: one actor (the accident victim), one role, authenticated per FR-1, hard per-user isolation per FR-1.9. Confirmed for v1 (D-11); sharing/helper access is revisited after v1.
- **D-3** Health issue model per FR-4/**Data Entities**: named issue with severity 0–10, status lifecycle active→improving→resolved, timestamped progress updates, optional treatments, linked providers/documents/expenses/appointments.
- **D-4** "Get your life back on track" = guided starter checklist + tasks/deadlines/reminders (FR-5) + expense tracking (FR-7) + journal/timeline (FR-8) + shareable exports (FR-9).
- **D-5** All user data is treated as sensitive personal data (it includes health information) and protected per `SECURITY.md` under the GDPR-grade global bar (D-10); health-specific regime applicability is recorded in `SECURITY.md` (SQ-1 resolution).
- **D-6** A user can manage multiple accidents; the Case is the anchor entity (FR-2).
- **D-7** v1 has no integrations with insurers, healthcare systems, or any other third-party data source; all data is manually entered, and exports (FR-9) serve insurers/attorneys/providers instead. The external service dependencies are outbound email delivery (FR-1.1, FR-1.7, FR-5.6) and the managed identity service behind authentication (FR-1.2); `ARCHITECTURE.md` owns the products. Platform infrastructure (hosting, CDN) is not an integration.
- **D-8** Values originally chosen as provisional defaults to make requirements testable are final, each stated in its owning requirement: accepted file types and per-file size limits (FR-3.1), storage quota (FR-3.10), per-case currency (FR-7.5), v1 locales (NFR-5.4), case-scoped contacts (FR-6.2), session lifetimes (FR-1.10), starter checklist content (FR-5.1).
- **D-9** Passkeys (FR-1.2) were chosen over passwords for a product holding health data: there is no shared secret to phish, reuse, or expose in a breach dump, and no second factor to bolt on. The cost is that account recovery needs an explicit design — FR-1.6 provides it via recovery codes, with the flow specified in `SECURITY.md` (SEC-AUTHN-6).
- **D-10** v1 accepts users anywhere and designs to a GDPR-grade bar globally — lawful basis, data-subject rights, 72-hour breach notice, storage limitation — so jurisdiction never needs re-litigating; `SECURITY.md` owns the controls that implement the bar (resolves OQ-1).
- **D-11** v1 stays strictly single-user (D-2 confirmed): no sharing of any kind; exports (FR-9) are the sharing mechanism; helper/attorney access is revisited after v1 (resolves OQ-2).
- **D-12** The nine-item starter checklist in FR-5.1 is the final v1 content, presented as general guidance only (FR-4.9, FR-5.8); domain review by insurance/legal expertise is a post-v1 improvement, not a launch blocker (resolves OQ-3).
- **D-13** Notification channels are in-app and email only; SMS and push stay out of scope for v1, and no quiet-hours feature ships. The in-app feed serves as the notice of email delivery failure (SEC-EXT-4) (resolves OQ-4).
- **D-14** v1 is sized for a marketed public launch at approximately 1,000 concurrent users; NFR-4 budgets and the hosting sizing in `ARCHITECTURE.md` cite this figure (resolves OQ-5).
- **D-15** v1 ships localized: i18n infrastructure with English and Spanish locales (NFR-5.4), locale-aware date and number formatting, and one currency per case chosen at creation with no exchange-rate math (FR-7.5) (resolves OQ-6).
- **D-16** Accounts inactive for 3 years are purged after advance email warnings (FR-9.5); this satisfies the storage-limitation half of D-10's bar and anchors the export-artifact lifetime (FR-9.4) (resolves OQ-7).
- **D-17** Journal entries are deletable: explicit confirmation (NFR-5.5), a 30-day undo window, and a permanent timeline event recording the deletion — content gone, deletion visible (FR-8.4). System-generated timeline events remain undeletable (resolves OQ-8).

## Open Questions

None open. OQ-1 through OQ-8 are resolved and recorded as decisions above (OQ-1 → D-10, OQ-2 → D-11, OQ-3 → D-12, OQ-4 → D-13, OQ-5 → D-14, OQ-6 → D-15, OQ-7 → D-16, OQ-8 → D-17); the OQ IDs remain valid for traceability.
