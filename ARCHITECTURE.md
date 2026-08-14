# AfterImpact — Architecture

This document owns HOW the system is built: components, interfaces, data flow, trust boundaries, technology choices and rationale. `REQUIREMENTS.md` owns WHAT the system does; `DESIGN.md` owns the visual language and interaction conventions the clients implement; `SECURITY.md` owns the security posture and the controls that attach to the boundaries located here.

Markers used across all specification documents: `UNKNOWN` = no input source provides the fact; `TO BE DECIDED` = decision not yet made; `ASSUMPTION` = an inference not directly stated by an input source.

## Technology Decisions

Decided, and binding on implementation:

| Area | Decision |
| --- | --- |
| Runtime environment | Web application and mobile application over a backend API |
| Server framework | Quarkus / Kotlin |
| Client framework | Compose Multiplatform (mobile: iOS + Android); React + TypeScript (web) |
| API style | REST |
| Authentication | WebAuthn passkeys for normal access; verified-email OTP only for restricted first-passkey bootstrap and lost-passkey recovery (FR-1.2, FR-1.6, D-9) |
| Identity | Amazon Cognito (managed identity provider). Its sign-in policy permits only `WEB_AUTHN` and `EMAIL_OTP`; `PASSWORD` and `SMS_OTP` are disabled, the app client enables `ALLOW_USER_AUTH`, and `AuthSessionValidity` is 10 minutes. All Cognito ceremonies terminate server-side at the API, which issues normal sessions after passkey authentication and restricted bootstrap/recovery sessions after email OTP |
| Data model | Relational, third normal form |
| Database | PostgreSQL (managed) |
| File store | S3 object storage (SSE-KMS, lifecycle rules) |
| Email | AWS SES |
| Malware scanning | ClamAV |
| Upload type inspection | Apache Tika |
| Hosting | AWS (EU regions available) |
| Edge | CloudFront + AWS WAF |
| CI/CD | GitHub Actions with AWS OIDC federation |
| Lint/format | ktlint + detekt (Kotlin); ESLint flat configuration + Prettier (React/TypeScript) |
| Deployment | Terraform |
| Scale | ~1,000 concurrent users (D-14, `REQUIREMENTS.md`) |

No technology choice remains open in this document.

## Initial Architecture

### Shape of the system

A React/TypeScript web client and Compose Multiplatform mobile clients (iOS and Android) talk to a single backend REST API over TLS, fronted by a CloudFront + AWS WAF edge layer. The API is the sole trust boundary: clients hold no business rules beyond input affordances and local validation for responsiveness; every rule in FR-1–FR-9 (ownership checks, validation, status lifecycles, totals, timeline generation) is enforced server-side. Identity is delegated to Amazon Cognito: every Cognito ceremony and token exchange terminates server-side at the API as a backend-for-frontend, and Cognito tokens never reach a client. The API issues its own normal session after WebAuthn, or a separately scoped bootstrap/recovery session after verified-email OTP that can register one passkey and perform no other operation — session mechanics are owned by `SECURITY.md` (SEC-SESSION-3). Behind the API sit a managed PostgreSQL database (3NF) for structured records, a private S3 file store for uploaded originals, and a background worker running as a separate service for time-driven and long-running work — separate rather than in-process so hostile-content parsing stays isolated from the API (SEC-FILE-6). Outbound email goes through AWS SES (required by FR-1.1, FR-1.6, FR-1.7, FR-5.6); per D-7, outbound email and the managed identity service are the system's external service dependencies.

Primary flows:

1. **Interactive CRUD** — client → REST API → authorization check (FR-1.9) → validation and business rules → RDBMS → response. Timeline events (FR-8.1) are written in the same transaction as the action that triggers them.
2. **Document upload/download** — client → REST API → type/size/quota checks (FR-3.1, FR-3.10) → file store (original bytes) + RDBMS (metadata); downloads stream the byte-identical original through authenticated API endpoints only (SEC-FILE-1), served so uploads can never execute as active content in the app origin (SEC-TRUST-3).
3. **Time-driven and async work** — the background worker fires reminders (FR-5.6, FR-4.7, FR-5.7), runs export jobs (FR-9.1, FR-9.2), purges soft-deleted data after its 30-day windows (FR-2.6, FR-3.8) and completes account deletion (FR-9.3), and performs malware scanning (FR-3.9). Results surface as in-app notifications and, per user preference, email.

### Components

- **Web client (React/TypeScript)**
  - **Responsibility:** All user-facing screens for the browser, implementing `DESIGN.md`'s tokens, components, focus/keyboard behavior, and WCAG 2.2 AA conformance (NFR-5.1); responsive from 360 px up (NFR-5.2) per the DESIGN.md grid and breakpoints; in-app viewing of PDFs and images (FR-3.5); severity-over-time charts (FR-4.3); in-app notification display (FR-5.6, FR-9.2); plain-language error presentation (NFR-5.3) and confirmation of every destructive action (NFR-5.5).
  - **Inputs:** User interaction; REST API responses; in-app notification feed.
  - **Outputs:** REST API requests; file downloads initiated for the user.
  - **Data owned or accessed:** Owns nothing durable. Accesses the signed-in user's data via the API; holds session credentials and transient UI state only.
  - **Open decisions:** None. The web client is a separate React/TypeScript implementation of the shared design rather than Compose MP's Wasm-canvas target (CQ-2 resolved — see **Cross-document questions**): React renders native DOM semantics, so DESIGN.md's accessibility expectations and NFR-5.1 are satisfiable on web. Light and dark themes are both supported; DESIGN.md owns the theme conventions (DQ-3 resolved there).

- **Mobile client (Compose MP)**
  - **Responsibility:** Same responsibilities as the web client on mobile form factors, from the shared Compose MP codebase, targeting iOS and Android and distributed through the App Store and Google Play. In-app notifications only — push channels are out of scope v1 (OQ-4).
  - **Inputs:** User interaction; REST API responses; in-app notification feed.
  - **Outputs:** REST API requests; file downloads/shares initiated by the user.
  - **Data owned or accessed:** Owns nothing durable; same access as the web client. The clients are online-only in v1 — no offline caching; nothing rests on the device beyond session credentials (SEC-SESSION-6).
  - **Open decisions:** None.

- **REST API application (Quarkus/Kotlin)**
  - **Responsibility:** The single enforcement point for all business rules: per-owner authorization on every operation (FR-1.9), input validation (dates not in future, severity 0–10, amount > 0, file type/size, quota), status lifecycles, derived values (case overview counts FR-2.4, expense totals FR-7.3, remaining reimbursement FR-7.4), automatic timeline event generation (FR-8.1), immutability of timeline events (FR-8.4), CSV export of expenses (FR-7.6), search over document metadata (FR-3.4), and the security event log (SEC-LOG-1) with log hygiene per SEC-LOG-2. Serves the in-app notification feed. Exact monetary arithmetic (NFR-6.2) lives here and in the schema, nowhere else.
  - **Inputs:** Authenticated HTTPS/REST requests from the clients; job-completion signals from the background worker.
  - **Outputs:** JSON responses; streamed file downloads; rows in the RDBMS; originals into the file store; job requests to the background worker; email requests to the email boundary.
  - **Data owned or accessed:** Owns the business rules for every entity; reads/writes all RDBMS data and file-store objects on behalf of the authenticated owner only.
  - **Open decisions:** None. API versioning is by URL path: the initial surface is `/v1`, and a breaking change mints `/v2`. Every live API version MUST route through the SEC-AUTHZ-5 centralized enforcement layer and remain within the SEC-DEPLOY-4 patch gate, so an unmaintained old version cannot become an authorization or patching bypass reachable by any network attacker (T-38). Search does not extend to file contents/OCR in v1 (per FR-3.4, search stays metadata-only); if ever adopted, the resulting index is a personal-data store bound by SEC-TRUST-2 and SEC-DATA-1 (T-19) and its extraction step is a hostile-content parsing surface under SEC-FILE-6 (T-17).

- **Identity & session handling**
  - **Responsibility:** Registration with email verification (FR-1.1), passkey (WebAuthn) registration and authentication ceremonies (FR-1.2, FR-1.3), restricted verified-email-OTP bootstrap and recovery (FR-1.2, FR-1.6), credential lifecycle — additional registration, listing, revocation (FR-1.8), sign-out with immediate session invalidation (FR-1.5), session lifetime enforcement (FR-1.10; lifetime values owned by REQUIREMENTS.md), account-level throttling of failed attempts (FR-1.4), email change with re-verification (FR-1.7), and re-authentication before account deletion (FR-9.3). Credential ceremonies are delegated to Amazon Cognito under the exact factor policy in **Technology Decisions**; the API application remains the session authority, terminates every Cognito exchange server-side, and issues either a normal passkey-authenticated session or the operation-limited email-OTP session SEC-AUTHN-6 defines.
  - **Inputs:** Passkey and email-OTP ceremonies from clients; server-side responses and tokens from Cognito.
  - **Outputs:** Sessions; entries in the security event log; verification and recovery emails via the email boundary.
  - **Data owned or accessed:** Owns the User entity's session records and verification tokens; passkey credential material is held by Cognito; nothing else.
  - **Open decisions:** None. The email-OTP recovery flow (SQ-2) and session token mechanics (SQ-3) are resolved in SECURITY.md.

- **Data persistence (relational, 3NF)**
  - **Responsibility:** Durable storage of all structured entities in **Data Entities** (REQUIREMENTS.md) in third normal form; referential integrity, including FR-6.3's rule that deleting a Contact preserves the contact's name as text on referencing records (denormalized name snapshot — a deliberate, documented 3NF exception); UTC timestamps on every record (NFR-6.1); exact decimal types for money (NFR-6.2); soft-delete state for the 30-day undo windows; document version lineage (FR-3.7) and journal entry versions (FR-8.4).
  - **Inputs:** SQL from the API application and background worker only — no other component reaches the database.
  - **Outputs:** Query results; backup artifacts meeting NFR-3.2 (RPO ≤ 24 h, RTO ≤ 12 h).
  - **Data owned or accessed:** System of record for every entity except uploaded file bytes.
  - **Open decisions:** None. The database is managed PostgreSQL: NUMERIC types carry exact money (NFR-6.2), and point-in-time recovery meets the NFR-3.2 RPO/RTO. Document search uses PostgreSQL full-text search over metadata (title, description, tags) — no separate index; file-content/OCR search is not in v1 per FR-3.4 — search stays metadata-only, revisited against NFR-4.3.

- **File store (uploaded originals)**
  - **Responsibility:** Durable, private storage of original uploaded bytes and prior versions (FR-3.5, FR-3.7), export artifacts (FR-9.1, FR-9.2), byte-identical retrieval, durability against single-component failure (NFR-3.3), encryption at rest (NFR-1.2), and no direct client access — every fetch goes through the API's authorization (SEC-FILE-1).
  - **Inputs:** Writes/reads from the API application and background worker.
  - **Outputs:** File bytes to the API for streaming to clients.
  - **Data owned or accessed:** Owns file bytes and export artifacts; metadata stays in the RDBMS.
  - **Open decisions:** None. The store is S3 (see **Technology Decisions**): a private bucket with SSE-KMS encryption at rest, lifecycle rules enforcing artifact expiry (FR-9.4), S3 durability meeting NFR-3.3, and access streamed through the API only per SEC-FILE-1.

- **Background processing**
  - **Responsibility:** Reminder scheduling and delivery within 5 minutes of fire time (FR-5.6), default and escalating reminders (FR-4.7, FR-5.7); asynchronous export jobs with in-app ready notification (FR-9.2) and PDF case-summary generation within its 60-second budget (FR-9.1); permanent purge after undo windows (FR-2.6, FR-3.8) and account-deletion completion within 30/90 days (FR-9.3); enforcing the export-artifact lifetime purge (FR-9.4); post-restore reconciliation — re-applying deletions, credential revocations, and session invalidations recorded after a backup point, and invalidating all sessions on restore (SEC-DATA-8); malware scanning of uploads with ClamAV (FR-3.9).
  - **Inputs:** Job records and schedules from the RDBMS (Reminder, Export Job, soft-delete timestamps); files from the file store.
  - **Outputs:** Delivery-state updates and notifications via the API's data; export artifacts to the file store; emails via the email boundary; deletions in RDBMS and file store.
  - **Data owned or accessed:** Owns Reminder delivery state and Export Job lifecycle; reads all case data for exports; deletes across all stores for purges.
  - **Open decisions:** None. The worker runs as a separate service, not in-process with the API: it parses attacker-influenced bytes (malware scanning FR-3.9, export embedding) while holding read-all/delete-all reach, and hostile-content-parsing isolation (SEC-FILE-6) is the recorded rationale — an in-process parser exploit would have reached the API application and session handling (T-17). No message broker is introduced: database-backed job state satisfies current requirements.

- **Outbound email boundary (external)**
  - **Responsibility:** The system's outbound email channel, provided by AWS SES — one of the two external service dependencies D-7 allows (with the identity provider): delivery of verification emails (FR-1.1, FR-1.7), account-recovery mail (FR-1.6; form owned by SECURITY.md, SEC-EXT-2 and SEC-AUTHN-6), credential-lifecycle notices (FR-1.8), email reminders honoring the account-wide toggle (FR-5.6), and breach notification if ever required (NFR-2.5).
  - **Inputs:** Send requests from the API application and background worker.
  - **Outputs:** Email to the user's verified address; delivery outcomes back to the caller.
  - **Data owned or accessed:** Transports message content; stores nothing (SEC-DATA-6 governs what may be handed to it).
  - **Open decisions:** None. The provider is AWS SES (see **Technology Decisions**). Delivery outcomes are returned to the caller; how persistent failure is handled is specified in SEC-DATA-6's neighbor SEC-EXT-4 — at minimum, persistent reminder-delivery failure MUST surface as an in-app notification so a silently-dead channel (provider trouble, or a mailbox attacker filtering messages) does not cause missed deadlines (T-49). Availability of this sole outbound email channel bounds verification, recovery, and reminders (T-30), and sender authentication for the domain is required by SEC-EXT-4 (T-28).

### Trust boundaries

1. **Client ↔ API:** untrusted client to trusted server; TLS only (NFR-1.1); all authorization and validation server-side (FR-1.9). A CloudFront + AWS WAF edge layer sits on this boundary in front of the API, providing volumetric protection (T-21).
2. **API ↔ stores (RDBMS, file store):** private network boundary; only the API application and background worker cross it. Backup artifacts and any further store that holds personal data (e.g., a separate search index, if the database-search decision is ever revisited) sit inside this boundary too (SEC-TRUST-2, T-19, T-31). Uploaded content crossing back toward clients is treated as hostile (NFR-1.5).
3. **System ↔ email provider (AWS SES):** outbound external boundary; carries minimal personal data.
4. **System ↔ identity provider (Amazon Cognito):** external identity dependency; carries passkey and email-OTP ceremonies and their token exchanges. The API terminates every exchange server-side (SEC-SESSION-3), so Cognito tokens never reach a browser or mobile client.

Controls at these boundaries are specified in `SECURITY.md`.

## Requirement Traceability

Statuses: `SUPPORTED` (a named component is responsible), `PARTIALLY DEFINED` (responsibility assigned but a material decision is open), `TO BE DECIDED` (no resolved responsibility).

| Requirement group | Responsible component(s) | Status | Notes |
| --- | --- | --- | --- |
| FR-1.1–FR-1.5, FR-1.7–FR-1.10 (Accounts & Authentication) | Identity & session handling; REST API application | SUPPORTED | Normal access is passkey-only; Cognito email OTP creates only the restricted bootstrap/recovery session D-9 defines. FR-1.9 is enforced on every API operation; FR-1.10 lifetime values are set in REQUIREMENTS.md. |
| FR-1.6 (Account recovery) | Identity & session handling; Outbound email boundary (external) | SUPPORTED | Ten-minute verified-email-OTP recovery and replacement-passkey flow designed in SECURITY.md (SEC-AUTHN-6). |
| FR-2.1–FR-2.6 (Accident Cases) | REST API application; Data persistence; Background processing (FR-2.6 purge) | SUPPORTED | Case overview counts computed server-side (FR-2.4). |
| FR-3.1–FR-3.8, FR-3.10 (Documents) | REST API application; File store; Data persistence; Background processing (FR-3.8 purge) | SUPPORTED | FR-3.4 file-content/OCR indexing is not in v1 — search is metadata-only (revisit against NFR-4.3). |
| FR-3.9 (malware scan) | Background processing | SUPPORTED | Ships in v1: ClamAV in the isolated worker; flagged files quarantined per SEC-FILE-4. |
| FR-4.1–FR-4.9 (Health & Appointments) | REST API application; Data persistence; clients (FR-4.3 chart); Background processing (FR-4.7 reminder) | SUPPORTED | FR-4.9 is a content constraint on clients and API alike. |
| FR-5.1–FR-5.8 (Tasks & Reminders) | REST API application; Background processing (FR-5.6, FR-5.7 delivery); clients (in-app display) | SUPPORTED | Email channel via outbound email boundary; checklist content is set in REQUIREMENTS.md (OQ-3 resolved there). |
| FR-6.1–FR-6.4 (Contacts) | REST API application; Data persistence | SUPPORTED | FR-6.3 name-snapshot rule owned by the persistence design. |
| FR-7.1–FR-7.6 (Expenses) | REST API application; Data persistence | SUPPORTED | Exact decimal handling per NFR-6.2; CSV export generated by the API (FR-7.6). |
| FR-8.1–FR-8.4 (Timeline & Journal) | REST API application; Data persistence | SUPPORTED | Timeline events written transactionally with their triggering action; FR-8.1 now records document replacement/edits and record deletions (T-12, T-47). Journal deletability is set in REQUIREMENTS.md (OQ-8 resolved there); tamper-evidence controls are set in SECURITY.md (SQ-13 resolved there). |
| FR-9.1, FR-9.2, FR-9.4 (Exports) | Background processing; File store; REST API application | SUPPORTED | FR-9.1's 60-second budget constrains the PDF pipeline; FR-9.4 sets artifact lifetime/scope/quota (T-18); full-account export requires step-up re-auth (SEC-AUTHN-10) and out-of-band notice (SEC-AUTHN-11). |
| FR-9.3 (Account deletion) | Identity & session handling; Background processing | SUPPORTED | Re-authentication is a fresh passkey ceremony (SEC-AUTHN-7); deletion initiation is notified (SEC-AUTHN-11) and cancellable within the window FR-9.3 defines (T-46); purge timelines owned by background processing. |
| NFR-1 (Security), NFR-2 (Privacy) | All components | SUPPORTED | Authored with their controls in SECURITY.md; the boundaries they attach to are located in **Trust boundaries** above. |
| NFR-3 (Reliability) | Data persistence; File store; Background processing (restore reconciliation) | SUPPORTED | PostgreSQL point-in-time recovery and S3 durability satisfy NFR-3.2/NFR-3.3; the NFR-3.1 availability target rests on AWS hosting. A restore under NFR-3.2 MUST reconcile post-backup deletions, credential revocations, and session invalidations (SEC-DATA-8, T-32, T-48). |
| NFR-4 (Performance) | REST API application; Data persistence; File store | SUPPORTED | NFR-4.1/4.3 budgets sized for ~1,000 concurrent users (D-14); NFR-4.2 constrains the upload path. |
| NFR-5 (Usability & Accessibility) | Web client; Mobile client | SUPPORTED | DESIGN.md implemented by the clients; the React/TypeScript web client renders native DOM semantics, so NFR-5.1 is claimable on web (CQ-2 resolved). |
| NFR-6 (Data Integrity) | Data persistence; REST API application | SUPPORTED | UTC storage, exact decimals in schema and API. |

## Cross-document questions

None open. Resolved questions are recorded here so the IDs stay traceable:

- **CQ-2 (resolved)** The web client is a separate React/TypeScript implementation of the shared design; Compose Multiplatform serves mobile only. React renders native DOM semantics, so DESIGN.md's accessibility expectations (WCAG 2.2 AA, system fonts) and NFR-5.1 are satisfiable on web. Its security consequences (SQ-4) are resolved in SECURITY.md.
- **CQ-3 (resolved)** The FR-2.6/FR-3.8 confirmation copy is reconciled to state the recovery window, eventual permanent deletion, and backup lag truthfully (copy owned by `DESIGN.md` / `REQUIREMENTS.md`), and SEC-DATA-8 bounds how long purged records may survive in backups (`SECURITY.md`). Its privacy facet, threat T-33, is closed.
