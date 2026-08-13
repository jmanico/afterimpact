# AfterImpact — Architecture

This document owns HOW the system is built: components, interfaces, data flow, trust boundaries, technology choices and rationale. `REQUIREMENTS.md` owns WHAT the system does; `DESIGN.md` owns the visual language, interaction conventions, and accessibility expectations the clients must implement. Detailed security controls belong to `SECURITY.md` (to be created next); this document only locates the boundaries those controls will attach to.

Markers: `UNKNOWN` = no input source provides the fact; `TO BE DECIDED` = decision not yet made; `ASSUMPTION` = an inference not directly stated by an input source.

## Required Architecture Inputs

- Requirements source: REQUIREMENTS.md
- Design source: DESIGN.md
- System purpose: See REQUIREMENTS.md
- Primary use cases: See REQUIREMENTS.md
- Target users / actors: See REQUIREMENTS.md
- Runtime environment: web application and mobile application with a backend API
- Server framework: Quarkus/Kotlin
- Client framework: Compose MP
- API style and integration model: REST
- Authentication and session model: passkey (no passwords at all, passkey only flow)
- Data model expectations: third normal form RBDMS
- Deployment model: terraform
- Scale expectations: modest
- Security expectations: TO BE DECIDED — addressed in a later security specification, SECURITY.md

**Input conflict (must be resolved before implementation):** the architecture notes mandate a passkey-only authentication model, but `REQUIREMENTS.md` FR-1.1–FR-1.8 specify email + password registration, password rules, password reset, and optional TOTP. These cannot both hold. This document treats the notes' passkey-only decision as the architectural direction (notes are the root source of truth) and marks every password-specific requirement as unresolved in **Requirement Traceability**. `REQUIREMENTS.md` needs a revision pass; email verification (FR-1.1) and account-lifecycle flows remain required regardless of credential type. See CQ-1 below.

## Initial Architecture (Provisional)

Logical architecture only. Where a technology is named above (Quarkus/Kotlin, Compose MP, REST, relational 3NF storage, Terraform) it is decided by the architecture notes; everything else — database product, file-store technology, hosting platform, email provider, malware-scan engine — is TO BE DECIDED and this document stays neutral on it.

### Shape of the system

Two client applications (web, mobile) built from a shared Compose Multiplatform codebase talk to a single backend REST API over TLS. The API is the sole trust boundary: clients hold no business rules beyond input affordances and local validation for responsiveness; every rule in FR-1–FR-9 (ownership checks, validation, status lifecycles, totals, timeline generation) is enforced server-side. Behind the API sit a relational database (3NF) for structured records, a private file store for uploaded originals, and a background worker for time-driven and long-running work. Outbound email is the only external integration (required by FR-1.1, FR-1.6, FR-5.6; D-7 excludes all others).

Primary flows:

1. **Interactive CRUD** — client → REST API → authorization check (FR-1.9) → validation and business rules → RDBMS → response. Timeline events (FR-8.1) are written in the same transaction as the action that triggers them.
2. **Document upload/download** — client → REST API → type/size/quota checks (FR-3.1, FR-3.10) → file store (original bytes) + RDBMS (metadata); downloads stream the byte-identical original through authenticated API endpoints only (NFR-1.4), served so uploads can never execute as active content in the app origin (NFR-1.5).
3. **Time-driven and async work** — the background worker fires reminders (FR-5.6, FR-4.7, FR-5.7), runs export jobs (FR-9.1, FR-9.2), purges soft-deleted data after its 30-day windows (FR-2.6, FR-3.8) and completes account deletion (FR-9.3), and performs malware scanning if enabled (FR-3.9). Results surface as in-app notifications and, per user preference, email.

### Components

- **Web client (Compose MP)**
  - **Responsibility:** All user-facing screens for the browser, implementing `DESIGN.md`'s tokens, components, focus/keyboard behavior, and WCAG 2.2 AA conformance (NFR-5.1); responsive from 360 px up (NFR-5.2) per the DESIGN.md grid and breakpoints; in-app viewing of PDFs and images (FR-3.5); severity-over-time charts (FR-4.3); in-app notification display (FR-5.6, FR-9.2); plain-language error presentation (NFR-5.3) and confirmation of every destructive action (NFR-5.5).
  - **Inputs:** User interaction; REST API responses; in-app notification feed.
  - **Outputs:** REST API requests; file downloads initiated for the user.
  - **Data owned or accessed:** Owns nothing durable. Accesses the signed-in user's data via the API; holds session credentials and transient UI state only.
  - **Open decisions:** Compose MP's web target (Kotlin/Wasm-Canvas) renders to canvas, which is in tension with DESIGN.md's native-semantics accessibility expectations and system-font direction — whether the web client is Compose/Wasm or a separate implementation of the shared design needs an explicit decision (CQ-2). Light/dark mode (DQ-3).

- **Mobile client (Compose MP)**
  - **Responsibility:** Same responsibilities as the web client on mobile form factors, from the shared Compose MP codebase. In-app notifications only — push channels are out of scope v1 (OQ-4).
  - **Inputs:** User interaction; REST API responses; in-app notification feed.
  - **Outputs:** REST API requests; file downloads/shares initiated by the user.
  - **Data owned or accessed:** Owns nothing durable; same access as the web client. Any offline caching is out of scope — no requirement demands it (ASSUMPTION: online-only v1).
  - **Open decisions:** Target OSes (iOS/Android/both) UNKNOWN; distribution model UNKNOWN.

- **REST API application (Quarkus/Kotlin)**
  - **Responsibility:** The single enforcement point for all business rules: per-owner authorization on every operation (FR-1.9), input validation (dates not in future, severity 0–10, amount > 0, file type/size, quota), status lifecycles, derived values (case overview counts FR-2.4, expense totals FR-7.3, remaining reimbursement FR-7.4), automatic timeline event generation (FR-8.1), immutability of timeline events (FR-8.4), CSV export of expenses (FR-7.6), search over document metadata (FR-3.4), and the security event log (NFR-1.6) with log hygiene per NFR-1.7. Serves the in-app notification feed. Exact monetary arithmetic (NFR-6.2) lives here and in the schema, nowhere else.
  - **Inputs:** Authenticated HTTPS/REST requests from the clients; job-completion signals from the background worker.
  - **Outputs:** JSON responses; streamed file downloads; rows in the RDBMS; originals into the file store; job requests to the background worker; email requests to the email boundary.
  - **Data owned or accessed:** Owns the business rules for every entity; reads/writes all RDBMS data and file-store objects on behalf of the authenticated owner only.
  - **Open decisions:** API versioning scheme TO BE DECIDED. Whether search extends to file contents/OCR (FR-3.4 MAY) TO BE DECIDED.

- **Identity & session handling**
  - **Responsibility:** Registration with email verification (FR-1.1), passkey (WebAuthn) registration/authentication per the architecture notes, sign-out with immediate session invalidation (FR-1.5), session lifetime enforcement (FR-1.10), account-level throttling of failed attempts (FR-1.4, adapted to passkeys), email change with re-verification (FR-1.7), and re-authentication before account deletion (FR-9.3, currently phrased as password re-entry). Logically part of the API application, not a separate service — a single-role, single-user product at modest scale does not justify an external identity provider (ASSUMPTION).
  - **Inputs:** Credential ceremonies and session tokens from clients.
  - **Outputs:** Sessions; entries in the security event log; verification/reset emails via the email boundary.
  - **Data owned or accessed:** Owns the User entity's credential material (passkey public keys, session records, verification tokens); nothing else.
  - **Open decisions:** CQ-1 — the passkey-only decision invalidates FR-1.2, FR-1.3 (password path), FR-1.6, FR-1.7 (password change), FR-1.8 (TOTP as second factor); account-recovery flow for lost passkeys is TO BE DECIDED and is the critical gap the requirements revision must fill. Session token mechanics TO BE DECIDED (SECURITY.md).

- **Data persistence (relational, 3NF)**
  - **Responsibility:** Durable storage of all structured entities in **Data Entities** (REQUIREMENTS.md) in third normal form; referential integrity, including FR-6.3's rule that deleting a Contact preserves the contact's name as text on referencing records (denormalized name snapshot — a deliberate, documented 3NF exception); UTC timestamps on every record (NFR-6.1); exact decimal types for money (NFR-6.2); soft-delete state for the 30-day undo windows; document version lineage (FR-3.7) and journal entry versions (FR-8.4).
  - **Inputs:** SQL from the API application and background worker only — no other component reaches the database.
  - **Outputs:** Query results; backup artifacts meeting NFR-3.2 (RPO ≤ 24 h, RTO ≤ 12 h).
  - **Data owned or accessed:** System of record for every entity except uploaded file bytes.
  - **Open decisions:** Database product TO BE DECIDED. Whether document search uses the database's text-search capability or a separate index is TO BE DECIDED (a separate search system is not currently justified at modest scale — ASSUMPTION: start with database search, revisit against NFR-4.3).

- **File store (uploaded originals)**
  - **Responsibility:** Durable, private storage of original uploaded bytes and prior versions (FR-3.5, FR-3.7), export artifacts (FR-9.1, FR-9.2), byte-identical retrieval, durability against single-component failure (NFR-3.3), encryption at rest (NFR-1.2), and no direct client access — every fetch goes through the API's authorization (NFR-1.4).
  - **Inputs:** Writes/reads from the API application and background worker.
  - **Outputs:** File bytes to the API for streaming to clients.
  - **Data owned or accessed:** Owns file bytes and export artifacts; metadata stays in the RDBMS.
  - **Open decisions:** Technology (object store vs. filesystem vs. database blobs) TO BE DECIDED; NFR-3.3's single-component-failure durability is the deciding constraint.

- **Background processing**
  - **Responsibility:** Reminder scheduling and delivery within 5 minutes of fire time (FR-5.6), default and escalating reminders (FR-4.7, FR-5.7); asynchronous export jobs with in-app ready notification (FR-9.2) and PDF case-summary generation within its 60-second budget (FR-9.1); permanent purge after undo windows (FR-2.6, FR-3.8) and account-deletion completion within 30/90 days (FR-9.3); malware scanning of uploads if enabled (FR-3.9).
  - **Inputs:** Job records and schedules from the RDBMS (Reminder, Export Job, soft-delete timestamps); files from the file store.
  - **Outputs:** Delivery-state updates and notifications via the API's data; export artifacts to the file store; emails via the email boundary; deletions in RDBMS and file store.
  - **Data owned or accessed:** Owns Reminder delivery state and Export Job lifecycle; reads all case data for exports; deletes across all stores for purges.
  - **Open decisions:** Execution model (in-process Quarkus scheduler vs. separate worker process) TO BE DECIDED — modest scale suggests in-process (ASSUMPTION); revisit if FR-9.2 export sizes threaten API responsiveness. No message broker is introduced: database-backed job state satisfies current requirements.

- **Outbound email boundary (external)**
  - **Responsibility:** The system's only external integration: delivery of verification emails (FR-1.1, FR-1.7), account-recovery mail (formerly FR-1.6; form TO BE DECIDED under CQ-1), email reminders honoring the account-wide toggle (FR-5.6), and breach notification if ever required (NFR-2.5).
  - **Inputs:** Send requests from the API application and background worker.
  - **Outputs:** Email to the user's verified address; delivery outcomes back to the caller.
  - **Data owned or accessed:** Transports message content; stores nothing (NFR-1.7's log hygiene applies to what is handed to it).
  - **Open decisions:** Provider/transport TO BE DECIDED.

### Trust boundaries

1. **Client ↔ API:** untrusted client to trusted server; TLS 1.2+ only (NFR-1.1); all authorization and validation server-side (FR-1.9).
2. **API ↔ stores (RDBMS, file store):** private network boundary; only the API application and background worker cross it. Uploaded content crossing back toward clients is treated as hostile (NFR-1.5).
3. **System ↔ email provider:** the only outbound external boundary; carries minimal personal data.

Controls at these boundaries are specified in `SECURITY.md`.

## Requirement Traceability

Statuses: `SUPPORTED` (a named component is responsible), `PARTIALLY DEFINED` (responsibility assigned but a material decision is open), `TO BE DECIDED` (no resolved responsibility).

| Requirement group | Responsible component(s) | Status | Notes |
| --- | --- | --- | --- |
| FR-1.1, FR-1.5, FR-1.9, FR-1.10 | Identity & session handling; REST API application | SUPPORTED | FR-1.9 enforced on every API operation; FR-1.10 lifetimes remain TO BE DECIDED in REQUIREMENTS.md itself. |
| FR-1.2, FR-1.3, FR-1.4, FR-1.6, FR-1.7, FR-1.8 | Identity & session handling | TO BE DECIDED | Password/TOTP-specific requirements conflict with the passkey-only decision (CQ-1); no component can be responsible for them as written. Lockout (FR-1.4) and re-verification on email change (FR-1.7) carry over in adapted form. |
| FR-2.1–FR-2.6 (Accident Cases) | REST API application; Data persistence; Background processing (FR-2.6 purge) | SUPPORTED | Case overview counts computed server-side (FR-2.4). |
| FR-3.1–FR-3.8, FR-3.10 (Documents) | REST API application; File store; Data persistence; Background processing (FR-3.8 purge) | SUPPORTED | FR-3.4 file-content/OCR indexing (MAY) TO BE DECIDED. |
| FR-3.9 (malware scan) | Background processing | PARTIALLY DEFINED | SHOULD-level; scanning engine and whether it ships in v1 TO BE DECIDED. |
| FR-4.1–FR-4.9 (Health & Appointments) | REST API application; Data persistence; clients (FR-4.3 chart); Background processing (FR-4.7 reminder) | SUPPORTED | FR-4.9 is a content constraint on clients and API alike. |
| FR-5.1–FR-5.8 (Tasks & Reminders) | REST API application; Background processing (FR-5.6, FR-5.7 delivery); clients (in-app display) | SUPPORTED | Email channel via outbound email boundary; checklist content TO BE DECIDED (OQ-3). |
| FR-6.1–FR-6.4 (Contacts) | REST API application; Data persistence | SUPPORTED | FR-6.3 name-snapshot rule owned by the persistence design. |
| FR-7.1–FR-7.6 (Expenses) | REST API application; Data persistence | SUPPORTED | Exact decimal handling per NFR-6.2; CSV export generated by the API (FR-7.6). |
| FR-8.1–FR-8.4 (Timeline & Journal) | REST API application; Data persistence | SUPPORTED | Timeline events written transactionally with their triggering action. |
| FR-9.1, FR-9.2 (Exports) | Background processing; File store; REST API application | SUPPORTED | FR-9.1's 60-second budget is a performance constraint on the PDF pipeline. |
| FR-9.3 (Account deletion) | Identity & session handling; Background processing | PARTIALLY DEFINED | Re-authentication step currently specified as password re-entry; passkey equivalent pending CQ-1. Purge timelines owned by background processing. |
| NFR-1 (Security), NFR-2 (Privacy) | All components; specified in SECURITY.md | TO BE DECIDED | Boundaries located here (NFR-1.1, NFR-1.2, NFR-1.4, NFR-1.5, NFR-1.6, NFR-1.7 assigned above); controls deferred to SECURITY.md per the inputs section. |
| NFR-3 (Reliability) | Data persistence; File store | PARTIALLY DEFINED | NFR-3.2/NFR-3.3 constrain store technology choices still TO BE DECIDED; NFR-3.1 availability target depends on the UNKNOWN hosting platform. |
| NFR-4 (Performance) | REST API application; Data persistence; File store | SUPPORTED | NFR-4.1/4.3 budgets sized for modest scale (OQ-5); NFR-4.2 constrains the upload path. |
| NFR-5 (Usability & Accessibility) | Web client; Mobile client | PARTIALLY DEFINED | DESIGN.md implemented by the clients; CQ-2 (Compose web accessibility) must resolve before NFR-5.1 can be claimed on web. |
| NFR-6 (Data Integrity) | Data persistence; REST API application | SUPPORTED | UTC storage, exact decimals in schema and API. |

## Cross-document questions

- **CQ-1** The passkey-only authentication decision contradicts FR-1.1–FR-1.8's password/TOTP model. REQUIREMENTS.md needs revision: which FR-1 requirements are replaced, what is the passkey account-recovery flow, and what re-authentication satisfies FR-9.3?
- **CQ-2** Compose Multiplatform's web target renders to canvas; DESIGN.md's accessibility expectations (native semantics, WCAG 2.2 AA, system fonts) may not be satisfiable there today. Decide: accept Compose/Wasm with its accessibility layer, or use a different web client implementation of the shared design.
- **CQ-3** DESIGN.md lists platform targets as TO BE DECIDED (DQ-2); the architecture notes decide web + mobile. DESIGN.md should be updated to record that decision.
