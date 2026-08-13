# AfterImpact — Security

This document owns the security posture: security requirements, provisional controls, and trust-boundary enforcement. `REQUIREMENTS.md` owns WHAT the system does; `ARCHITECTURE.md` owns components, boundaries, and data flow; `DESIGN.md` owns client-facing interaction conventions. No implementation exists yet; every rule here is prospective. The threat model is TO BE DONE LATER and will attach to the boundaries and rules below.

Markers: `UNKNOWN` = no input source provides the fact; `TO BE DECIDED` = decision not yet made; `ASSUMPTION` = inference not directly stated by an input source. Provisional defaults are never presented as confirmed requirements.

## Required Security Inputs

- **Requirements source:** `REQUIREMENTS.md`
- **Design source:** `DESIGN.md`
- **Architecture source:** `ARCHITECTURE.md`
- **System purpose:** Help a person who has had a recent car accident manage paperwork, track accident-related health issues, and get life back on track (REQUIREMENTS.md).
- **Application profile:** Consumer application managing a single user's accident cases, uploaded documents, health records, tasks, expenses, and journal (REQUIREMENTS.md); realized per ARCHITECTURE.md as web + mobile clients (Compose Multiplatform) over a single backend REST API (Quarkus/Kotlin).
- **Users / actors / roles:** One actor: the accident victim, sole account owner; single role; strict per-user data isolation (D-2, FR-1.9). No admin, helper, or professional roles in v1.
- **Public interfaces and trust boundaries:** (1) Client ↔ API over TLS — the only untrusted-client boundary and the single enforcement point; all authorization and validation server-side; (2) API ↔ stores (RDBMS, file store) — private network boundary crossed only by the API application and background worker; (3) System ↔ email provider — the only outbound external boundary (ARCHITECTURE.md **Trust boundaries**).
- **Sensitive or regulated data:** All user data is treated as sensitive personal data because it includes health information (D-5): health issues, treatments, progress updates, appointments, uploaded medical/legal/insurance documents, contacts, expenses, journal entries, plus credential material and the security event log.
- **External integrations:** None in v1 except outbound email (verification, account mail, reminders); exports substitute for all other integrations (D-7).
- **Authentication model:** In conflict. REQUIREMENTS.md FR-1.1–FR-1.8 specify email + password with mandatory email verification and optional TOTP; ARCHITECTURE.md mandates passkey-only (WebAuthn) as the architectural direction (CQ-1). Passkeys are treated here as the explicitly selected direction; every password-specific control is unresolved pending CQ-1 (SQ-2).
- **Authorization model:** Owner-only: every create/read/update/delete/export operation is permitted only for the authenticated owner of the data; a request targeting another user's record fails without disclosing whether it exists (FR-1.9).
- **Session model:** Server-managed sessions with immediate sign-out invalidation (FR-1.5) and default lifetimes of 30 days absolute / 24 h idle (FR-1.10; values TO BE DECIDED with OQ-1). Session token mechanics (cookie vs. bearer) TO BE DECIDED (ARCHITECTURE.md).
- **Deployment and CI/CD model:** Infrastructure managed with Terraform (ARCHITECTURE.md). Hosting platform TO BE DECIDED (SQ-5); CI/CD system UNKNOWN.
- **Applicable privacy or regulatory obligations:** The security notes direct that this is health-care and accident data and privacy is critical, naming HIPAA and other privacy regulation; REQUIREMENTS.md leaves target regimes TO BE DECIDED (OQ-1). Binding obligations are therefore TO BE DECIDED and HIPAA applicability is unconfirmed (SQ-1). D-5 already requires treating all data as sensitive regardless of regime.
- **Security assurance target:** TO BE DECIDED. NFR-1.8 mandates verification against OWASP ASVS (current version) Level 2; confirming ASVS 5.0.0 Level 2 as the formal target is SQ-8. No conformance is claimed for the unbuilt system.
- **Security verification reference:** OWASP ASVS 5.0.0 (`REF-ASVS-5`).
- **Threat model status:** TO BE DONE LATER.

## Selected Security References and Prompt Imports

Sources actually located and read on 2026-08-13. Rules below are synthesized from them, not copied. ASVS requirement identifiers are cited only in verified versioned form; none have been verified individually yet, so ASVS 5.0.0 is referenced generally. OWASP Cheat Sheet Series pages are unversioned living documents (accessed 2026-08-13).

### Public references

| ID | Title (version) | URL |
| --- | --- | --- |
| `REF-ASVS-5` | OWASP Application Security Verification Standard 5.0.0 | <https://github.com/OWASP/ASVS/releases/tag/v5.0.0_release> |
| `REF-PC-2024` | OWASP Top 10 Proactive Controls (2024) | <https://top10proactive.owasp.org/archive/2024/the-top-10/> |
| `REF-API-2023` | OWASP Top 10 API Security Risks – 2023 | <https://owasp.org/API-Security/editions/2023/en/0x11-t10/> |
| `REF-REST` | REST Security Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html> |
| `REF-AUTH` | Authentication Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html> |
| `REF-SESSION` | Session Management Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html> |
| `REF-XSS` | Cross Site Scripting Prevention Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html> |
| `REF-INPUT` | Input Validation Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html> |
| `REF-SECRETS` | Secrets Management Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html> |
| `REF-LOG` | Logging Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html> |
| `REF-ERROR` | Error Handling Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Error_Handling_Cheat_Sheet.html> |
| `REF-SSDF` | NIST SP 800-218, Secure Software Development Framework (SSDF) Version 1.1 (February 2022) | <https://csrc.nist.gov/pubs/sp/800/218/final> |
| `REF-CICD` | CI/CD Security Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/CI_CD_Security_Cheat_Sheet.html> |
| `REF-SUPPLY` | Software Supply Chain Security Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html> |
| `REF-IAC` | Infrastructure as Code Security Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Infrastructure_as_Code_Security_Cheat_Sheet.html> |
| `REF-VULNDEP` | Vulnerable Dependency Management Cheat Sheet | <https://cheatsheetseries.owasp.org/cheatsheets/Vulnerable_Dependency_Management_Cheat_Sheet.html> |
| `REF-63B-4` | NIST SP 800-63B Revision 4, Digital Identity Guidelines (August 2025) | <https://pages.nist.gov/800-63-4/sp800-63b.html> |
| `REF-FIDO` | FIDO Alliance — Passkeys | <https://fidoalliance.org/passkeys/> |
| `REF-WEBAUTHN` | Web Authentication: An API for accessing Public Key Credentials — Level 2, W3C Recommendation (8 April 2021) | <https://www.w3.org/TR/webauthn/> |

`REF-63B-4`, `REF-FIDO`, and `REF-WEBAUTHN` are selected because ARCHITECTURE.md explicitly selects passkeys, not because the model is undecided.

### Local prompt imports

Base path: `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/`. Each file below was located and read in full; its guidance was converted into the project-specific rules cited by `PRM-*` ID.

| ID | Path (relative to base) | Purpose as applied here |
| --- | --- | --- |
| `PRM-QUALITY` | `Code Quality/00 General Code Quality Prompts/PROMPT.md` | Fail-closed security decisions, centralized boundary validation, testable isolated security logic |
| `PRM-QUARKUS` | `Backend Frameworks/Java/07 Secure Quarkus Developer/PROMPT.md` | Quarkus API hardening: endpoint auth, parameterized data access, safe serialization, headers (OIDC/LLM portions not applicable) |
| `PRM-KMP` | `Mobile/04 Secure Mobile Kotlin Multiplatform Developer/PROMPT.md` | Compose MP client rules: platform secure storage, HTTPS-only shared client, strict deserialization |
| `PRM-ABAC` | `Authorization/02 ABAC Architect/PROMPT.md` | Enforcement doctrine only (deny-by-default, fail-closed, single enforcement layer); full ABAC/policy-engine content not applicable to the single-role model |
| `PRM-PASSWORDLESS` | `Authentication/06 Secure Passwordless Authentication Developer/PROMPT.md` | WebAuthn/passkey ceremony validation, opaque user handles, credential lifecycle management |
| `PRM-RECOVERY` | `Authentication/03 Secure Account Recovery Developer/PROMPT.md` | Recovery/verification token hygiene, enumeration-safe recovery responses, session invalidation on recovery |
| `PRM-XSS` | `Web and API Security/03 Secure XSS Prevention Developer/PROMPT.md` | Text-only rendering of user data, security headers for browser delivery |
| `PRM-SQLI` | `Web and API Security/04 Secure SQL Injection Prevention Developer/PROMPT.md` | Parameterized queries only, identifier allowlists, least-privilege DB account |
| `PRM-CSRF` | `Web and API Security/05 Secure CSRF Prevention Developer/PROMPT.md` | Conditional CSRF rules, applicable only if cookie sessions are selected (SQ-3) |
| `PRM-API` | `Web and API Security/06 Secure API Developer/PROMPT.md` | Object-level authorization, DTO binding against mass assignment, no-store on sensitive responses |
| `PRM-UPLOAD` | `Web and API Security/08 Secure File Upload Developer/PROMPT.md` | Content inspection, server-generated storage names, non-executing delivery |
| `PRM-SSWEB` | `Web and API Security/12 Secure Server-Side Web Application Developer/PROMPT.md` | Server-side baseline: fail-closed authorization, server-side sessions, production configuration (password-specific portions superseded by CQ-1) |
| `PRM-RATE` | `Web and API Security/13 Secure API Rate Limiting Developer/PROMPT.md` | Limits on authentication, upload, export, and email-triggering endpoints |
| `PRM-TF` | `Infrastructure/Terraform/00 Secure Terraform Developer/PROMPT.md` | Provider-agnostic Terraform rules: no secrets in code/state, encrypted state, least-privilege identities, IaC scanning |

Not imported: `Web and API Security` sub-prompts for SSRF, XXE, database encryption, CORS, JWT, WebSocket, CSP, rate-limited API keys, webhooks, tRPC, and OpenAPI validation (not applicable now, or blocked on SQ-3/SQ-4/SQ-5 decisions); provider-specific Terraform prompts (AWS/Azure/GCP/CDK) pending the hosting decision (SQ-5); `Authentication/` sub-prompts for password storage, MFA, credential-stuffing defense, and SSO (password-centric or not applicable under the passkey-only direction, pending CQ-1) and session management (covered by `REF-SESSION`/`PRM-SSWEB`, revisited at SQ-3).

## Provisional Security Rules

Grouped by capability; one verifiable behavior per rule. Per-rule status is in **Requirement and Architecture Traceability**; `TO BE DECIDED` inside a rule marks a value no input document provides.

### Trust boundaries and server-side enforcement

- **SEC-TRUST-1** The REST API application MUST enforce every business rule, validation, and authorization decision server-side; client-side checks are responsiveness aids only and MUST NOT be relied on at the Client ↔ API boundary.
  - **Applies to:** All operations crossing trust boundary 1
  - **Verification:** API-level tests exercising every rule with the client bypassed (direct HTTP)
  - **References:** `REF-ASVS-5`, `REF-PC-2024`, `REF-INPUT`
- **SEC-TRUST-2** The RDBMS and file store MUST be reachable only by the API application and background worker; no client, public network path, or other component may cross trust boundary 2.
  - **Applies to:** API ↔ stores boundary
  - **Verification:** Infrastructure review plus network reachability tests from outside the private boundary
  - **References:** `REF-ASVS-5`, `REF-IAC`, `PRM-TF`
- **SEC-TRUST-3** Content originating from user uploads MUST be treated as hostile whenever it crosses back toward clients (NFR-1.5).
  - **Applies to:** Document view/download paths through the API
  - **Verification:** Tests that uploaded HTML/SVG/script content never executes in the app origin
  - **References:** `REF-XSS`, `PRM-UPLOAD`

### Authentication

Passkeys (WebAuthn) are the explicitly selected architectural direction; password/TOTP requirements FR-1.2–FR-1.8 are unresolved pending CQ-1 (SQ-2).

- **SEC-AUTHN-1** Authentication MUST use passkeys (WebAuthn); the server MUST validate each registration and assertion ceremony fully server-side — challenge freshness and single use, origin and RP ID binding, signature — and MUST NOT accept client-asserted authentication results.
  - **Applies to:** Identity & session handling; sign-in and passkey registration ceremonies
  - **Verification:** Negative tests with replayed challenges, wrong origin/RP ID, and tampered assertions
  - **References:** `REF-WEBAUTHN`, `REF-FIDO`, `REF-63B-4`, `PRM-PASSWORDLESS`
- **SEC-AUTHN-2** WebAuthn user handles MUST be opaque random values that encode no email address or other identifying data, and stored credential material MUST be limited to what verification requires (credential ID, public key, per-owner binding).
  - **Applies to:** Identity & session handling credential storage
  - **Verification:** Schema review; test that user handles are random and non-identifying
  - **References:** `REF-WEBAUTHN`, `PRM-PASSWORDLESS`
- **SEC-AUTHN-3** The system MUST require email verification before an account becomes usable for sign-in, and re-verification when the email address changes (FR-1.1, FR-1.7 carry over regardless of credential type).
  - **Applies to:** Registration and email-change flows
  - **Verification:** Functional tests: unverified accounts cannot sign in; email change requires re-verification
  - **References:** `REF-AUTH`, `PRM-RECOVERY`
- **SEC-AUTHN-4** The system MUST throttle failed authentication attempts per account (FR-1.4, adapted to passkeys; passkey-flow thresholds TO BE DECIDED) and MUST log each lockout to the security event log.
  - **Applies to:** Identity & session handling
  - **Verification:** Automated tests driving repeated failures, asserting lockout and log entry
  - **References:** `REF-AUTH`, `PRM-RATE`
- **SEC-AUTHN-5** All authentication and account-mail flows MUST respond identically whether or not an account exists; no response content, observable behavior, or email side effect may disclose account existence (FR-1.3 intent; reinforced for WebAuthn flows).
  - **Applies to:** Sign-in, registration, recovery, email change
  - **Verification:** Response-equivalence tests for existing vs. non-existing accounts
  - **References:** `REF-AUTH`, `REF-WEBAUTHN`, `REF-ERROR`
- **SEC-AUTHN-6** The account-recovery flow for lost passkeys is TO BE DECIDED (CQ-1); when designed, it MUST NOT provide a persistent authentication path weaker than the primary passkey flow, MUST NOT rely on emailed codes alone as an authentication factor, and completing recovery MUST invalidate all existing sessions.
  - **Applies to:** Identity & session handling
  - **Verification:** Design review against `REF-63B-4`; session-invalidation test on recovery completion
  - **References:** `REF-63B-4`, `REF-AUTH`, `REF-FIDO`, `PRM-RECOVERY`
- **SEC-AUTHN-7** Account deletion MUST require fresh re-authentication of the account owner (FR-9.3; the passkey equivalent of password re-entry is TO BE DECIDED under CQ-1).
  - **Applies to:** Account deletion flow
  - **Verification:** Test that deletion is refused without a fresh re-authentication ceremony
  - **References:** `REF-AUTH`, `REF-63B-4`
- **SEC-AUTHN-8** The user SHOULD be able to view and revoke their registered passkeys; revoking or losing the last credential MUST NOT strand the account without an authentication path, and credential registration or revocation SHOULD trigger a notification to the verified email address.
  - **Applies to:** Identity & session handling; credential lifecycle (adapted FR-1.7/FR-1.8 territory under CQ-1)
  - **Verification:** Functional tests of the credential list, revocation guard, and notification
  - **References:** `PRM-PASSWORDLESS`, `REF-FIDO`

### Session management

- **SEC-SESSION-1** The server MUST invalidate a session immediately on sign-out (FR-1.5); subsequent requests presenting that session MUST be rejected. Client-side state clearing alone is insufficient.
  - **Applies to:** All authenticated API operations
  - **Verification:** Test replaying a signed-out session credential against protected endpoints
  - **References:** `REF-SESSION`
- **SEC-SESSION-2** The server MUST enforce an absolute session lifetime and SHOULD enforce an idle timeout, both server-side (defaults 30 days / 24 h per FR-1.10; final values TO BE DECIDED with OQ-1).
  - **Applies to:** Identity & session handling
  - **Verification:** Clock-advanced tests asserting expiry regardless of client behavior
  - **References:** `REF-SESSION`, `REF-63B-4`
- **SEC-SESSION-3** Session token mechanics (cookie vs. bearer) are TO BE DECIDED (SQ-3). Whichever is selected, session identifiers MUST be opaque, generated with a cryptographically secure random source, transmitted only over TLS, never placed in URLs, and never written to logs (NFR-1.7). If cookie sessions are selected, CSRF defenses and restrictive cookie attributes MUST be specified before implementation (`PRM-CSRF`); if bearer tokens are selected, the design MUST document why CSRF does not apply and define client-side storage rules.
  - **Applies to:** Client ↔ API boundary
  - **Verification:** Design review at the token-model decision; token-entropy and log-scan checks in implementation
  - **References:** `REF-SESSION`, `REF-REST`, `PRM-CSRF`
- **SEC-SESSION-4** The session identifier MUST be regenerated on successful authentication so a pre-authentication session value never survives into an authenticated session.
  - **Applies to:** Identity & session handling
  - **Verification:** Test comparing session identifiers before and after authentication
  - **References:** `REF-SESSION`
- **SEC-SESSION-5** Completion of account recovery or any credential change MUST invalidate all other active sessions (FR-1.6/FR-1.7 carry over in adapted form under CQ-1).
  - **Applies to:** Identity & session handling
  - **Verification:** Test that concurrent sessions are invalidated after credential change or recovery
  - **References:** `REF-SESSION`
- **SEC-SESSION-6** Mobile clients MUST keep session credentials only in platform secure storage; they MUST NOT be written to shared preferences, plain files, or shared multiplatform code storage.
  - **Applies to:** Mobile client (Compose MP)
  - **Verification:** Code review of the storage abstraction; device-storage inspection tests
  - **References:** `PRM-KMP`, `REF-SESSION`

### Authorization

- **SEC-AUTHZ-1** The API MUST verify on every operation that the authenticated user owns the target record — established server-side from persisted ownership data, never from client-supplied values — before performing it (FR-1.9).
  - **Applies to:** Every CRUD/export operation on every entity
  - **Verification:** Automated per-endpoint tests using owned and non-owned record identifiers
  - **References:** `REF-ASVS-5`, `REF-API-2023`, `PRM-ABAC`
- **SEC-AUTHZ-2** Authorization MUST be deny-by-default and fail closed: a request is refused unless an explicit ownership match succeeds, and any error or unresolvable state in an authorization path MUST result in denial, never a fall-through permit.
  - **Applies to:** REST API application authorization paths
  - **Verification:** Fault-injection tests in authorization code paths asserting denial
  - **References:** `PRM-ABAC`, `PRM-QUALITY`, `PRM-SSWEB`
- **SEC-AUTHZ-3** A request targeting another user's record MUST fail with a response indistinguishable from the record not existing (FR-1.9).
  - **Applies to:** All API responses on authorization failure
  - **Verification:** Response-equivalence tests: non-owned vs. non-existent identifiers
  - **References:** `REF-API-2023`, `REF-ERROR`
- **SEC-AUTHZ-4** Object-level authorization MUST NOT depend on identifiers being unguessable; every file fetch and record access is authorized regardless of how the identifier was obtained (NFR-1.4).
  - **Applies to:** Documents, export artifacts, and all record endpoints
  - **Verification:** Tests fetching known-valid foreign identifiers; static check that no unauthenticated file routes exist
  - **References:** `REF-API-2023`, `REF-ASVS-5`
- **SEC-AUTHZ-5** Ownership checks MUST be centralized in a single enforcement layer that every endpoint passes through, so no individual handler can omit the check.
  - **Applies to:** REST API application
  - **Verification:** Architecture review; static analysis that no endpoint bypasses the enforcement layer
  - **References:** `PRM-ABAC`, `PRM-QUALITY`

### HTTP/API boundary

- **SEC-HTTP-1** All client–server communication MUST use TLS 1.2 or higher; no functionality may be served over cleartext (NFR-1.1). Protocol and cipher configuration beyond that floor is TO BE DECIDED with the hosting platform.
  - **Applies to:** Client ↔ API boundary
  - **Verification:** TLS configuration scan; test that no cleartext endpoint serves functionality
  - **References:** `REF-REST`, `REF-ASVS-5`
- **SEC-HTTP-2** The API MUST validate the HTTP method and declared content type of each request, rejecting unsupported methods and mismatched or unexpected content types rather than content-sniffing (REST is the selected style).
  - **Applies to:** All REST endpoints
  - **Verification:** Tests sending wrong-method and wrong-content-type requests
  - **References:** `REF-REST`
- **SEC-HTTP-3** Browser-delivered responses MUST carry security headers appropriate to the final web client model; the exact header set and CSP directives are TO BE DECIDED after CQ-2 resolves (SQ-4).
  - **Applies to:** Web client delivery and API responses rendered in browsers
  - **Verification:** Header assertions in integration tests once values are decided
  - **References:** `REF-XSS`, `PRM-XSS`, `REF-REST`
- **SEC-HTTP-4** Cross-origin resource sharing MUST NOT be enabled unless a cross-origin consumer is decided; if one emerges, allowed origins MUST be an explicit allowlist (need and origins TO BE DECIDED).
  - **Applies to:** REST API responses
  - **Verification:** Header scan asserting no permissive CORS in default configuration
  - **References:** `REF-REST`
- **SEC-HTTP-5** API responses containing personal data MUST NOT be cacheable by browsers or intermediaries.
  - **Applies to:** All authenticated API responses
  - **Verification:** Cache-header assertions on personal-data endpoints
  - **References:** `PRM-API`, `REF-REST`
- **SEC-HTTP-6** The API SHOULD enforce request-rate and resource limits beyond the FR-1.4 authentication throttle — per-user API rates, and stricter limits on upload, export, and email-triggering operations; limit values are TO BE DECIDED (SQ-9).
  - **Applies to:** All API operations; background jobs that send email
  - **Verification:** Load tests asserting limit enforcement once values are decided
  - **References:** `REF-API-2023`, `PRM-RATE`

### Input validation

- **SEC-INPUT-1** The API MUST validate all untrusted input server-side at trust boundary 1, for both format and business meaning: accident/onset dates not in the future (FR-2.1, FR-4.1), severity integer 0–10 (FR-4.1, FR-4.3), expense amount > 0 with two decimals (FR-7.1), enumerated categories/statuses/roles (FR-3.2, FR-5.2, FR-6.1, FR-7.1), and file type/size/quota (FR-3.1, FR-3.10). Free-text fields (journal, notes) accept broad legitimate character sets; validation MUST NOT substitute for output encoding.
  - **Applies to:** Every write operation at the Client ↔ API boundary
  - **Verification:** Per-field negative tests via direct HTTP
  - **References:** `REF-INPUT`, `REF-PC-2024`, `PRM-QUALITY`
- **SEC-INPUT-2** All database access MUST use parameterized statements or an equivalent mechanism separating code from data; dynamic query construction from user input MUST NOT occur, and dynamic sort/filter identifiers MUST map through a server-side allowlist.
  - **Applies to:** API application and background worker access to the RDBMS
  - **Verification:** Static analysis for string-built queries; injection test suite asserting payloads are treated as literals
  - **References:** `PRM-SQLI`, `REF-INPUT`, `PRM-QUARKUS`
- **SEC-INPUT-3** Upload acceptance MUST be decided by server-side inspection of file content against the allowed type list (FR-3.1), not by file extension or client-declared content type alone, with size limits enforced before full buffering. (PROVISIONAL safe default — inspection mechanism TO BE DECIDED.)
  - **Applies to:** Document upload endpoint
  - **Verification:** Tests uploading mismatched extension/content pairs and oversize streams
  - **References:** `PRM-UPLOAD`, `REF-INPUT`
- **SEC-INPUT-4** Request bodies MUST bind to explicit request models; unknown fields and server-controlled fields (owner, identifiers, timestamps) MUST be rejected or ignored, never bound to persisted entities.
  - **Applies to:** All REST write endpoints
  - **Verification:** Tests submitting extra/privileged fields and asserting they have no effect
  - **References:** `REF-API-2023`, `PRM-API`, `PRM-QUARKUS`

### File and document handling

- **SEC-FILE-1** Uploaded files and export artifacts MUST be stored only in the private file store and retrieved only by streaming through authenticated, owner-authorized API endpoints; no public, unauthenticated, or guessable URLs may exist (NFR-1.4).
  - **Applies to:** File store; document/export download paths
  - **Verification:** Infrastructure review; tests fetching storage paths directly without API authorization
  - **References:** `REF-ASVS-5`, `PRM-UPLOAD`, `REF-API-2023`
- **SEC-FILE-2** Files served to clients MUST be delivered so they cannot execute as active content in the application's origin (NFR-1.5); the exact delivery mechanism (disposition headers, sandboxing, separate origin) is TO BE DECIDED with the web client model (CQ-2).
  - **Applies to:** In-app viewing (FR-3.5) and downloads
  - **Verification:** Tests that uploaded HTML/SVG/JS renders inert in the app origin
  - **References:** `PRM-UPLOAD`, `REF-XSS`, `PRM-QUARKUS`
- **SEC-FILE-3** Stored files MUST use server-generated storage names; the original filename is kept only as sanitized display metadata (FR-3.2 title default) and MUST never be used as a storage path.
  - **Applies to:** File store; upload pipeline
  - **Verification:** Storage-layout review; path-traversal tests with hostile filenames
  - **References:** `PRM-UPLOAD`
- **SEC-FILE-4** If malware scanning is enabled (FR-3.9, SHOULD), a flagged file MUST be rejected with a user notice and MUST NOT become retrievable; the scanning engine and v1 inclusion are TO BE DECIDED (SQ-10).
  - **Applies to:** Upload pipeline; background processing
  - **Verification:** Test with a standard detection-test file once an engine is selected
  - **References:** `PRM-UPLOAD`
- **SEC-FILE-5** Downloads MUST return the byte-identical original (FR-3.5); the system MUST NOT transform stored originals in place, preserving the integrity of evidence documents.
  - **Applies to:** File store; download path
  - **Verification:** Round-trip hash comparison tests
  - **References:** `REF-ASVS-5`

### Output encoding and safe rendering

- **SEC-OUT-1** All user-authored text (titles, notes, tags, journal entries, contact fields, filenames) MUST be rendered as text with context-appropriate encoding in every client surface; no interface may inject user data as raw HTML/DOM markup, and no rich-HTML input may be added without a security review.
  - **Applies to:** Web and mobile clients rendering user data
  - **Verification:** Rendering tests with markup/script payloads in every user-text field
  - **References:** `REF-XSS`, `PRM-XSS`, `REF-PC-2024`
- **SEC-OUT-2** Generated exports MUST treat user data as inert data: CSV output (FR-7.6, FR-9.2) MUST neutralize spreadsheet-formula injection, and PDF generation (FR-9.1) MUST NOT interpret user text as markup or code.
  - **Applies to:** REST API CSV export; background PDF/export pipeline
  - **Verification:** Exports containing formula-prefix and markup payloads open inert
  - **References:** `REF-XSS`, `REF-INPUT`
- **SEC-OUT-3** User-supplied URL values (e.g., contact fields), if ever rendered as links, MUST be restricted to safe schemes; script-capable schemes MUST be rejected.
  - **Applies to:** Clients rendering contact and note fields
  - **Verification:** Rendering tests with hostile scheme payloads
  - **References:** `PRM-XSS`, `REF-XSS`

### Data protection and privacy

- **SEC-DATA-1** All stored personal data — database contents, uploaded files, export artifacts, and backups — MUST be encrypted at rest (NFR-1.2); algorithms and key management are TO BE DECIDED with the database, file-store, and hosting choices (SQ-5).
  - **Applies to:** Data persistence; file store; backups
  - **Verification:** Infrastructure review once technologies are selected
  - **References:** `REF-ASVS-5`, `PRM-TF`
- **SEC-DATA-2** The system MUST collect and store only data serving the functional requirements (NFR-2.3) and MUST treat all of it as sensitive personal data (D-5), regardless of which regulatory regime is ultimately targeted.
  - **Applies to:** All entities and any telemetry
  - **Verification:** Schema and field review against REQUIREMENTS.md **Data Entities**
  - **References:** `REF-ASVS-5`, `REF-63B-4`
- **SEC-DATA-3** Personal data MUST NOT be sold or shared with third parties, and authenticated screens MUST NOT include third-party advertising or marketing trackers (NFR-2.1).
  - **Applies to:** All clients and server integrations
  - **Verification:** Network-traffic inspection of authenticated sessions; dependency review
  - **References:** `REF-ASVS-5`
- **SEC-DATA-4** Data portability and erasure MUST be honored via full-account export (FR-9.2) and account deletion with permanent removal within 30 days, and within 90 days from backups (FR-9.3, NFR-2.4).
  - **Applies to:** Background processing; all stores
  - **Verification:** Deletion tests asserting absence after purge windows, including the file store
  - **References:** `REF-ASVS-5`
- **SEC-DATA-5** Affected users MUST be notified of a personal-data breach without undue delay, consistent with applicable law; the governing jurisdictions are TO BE DECIDED (NFR-2.5, OQ-1, SQ-1) and incident-response ownership is UNKNOWN (SQ-11).
  - **Applies to:** Incident response (process TO BE DECIDED)
  - **Verification:** Incident-response runbook review once jurisdiction is decided
  - **References:** `REF-ASVS-5`
- **SEC-DATA-6** Outbound email MUST carry the minimum personal data needed for its purpose; health data and document contents MUST NOT appear in email bodies or subjects. What reminder emails may reference (e.g., appointment purpose or provider name) is TO BE DECIDED (SQ-12).
  - **Applies to:** System ↔ email provider boundary
  - **Verification:** Template review; tests asserting excluded fields never reach the email boundary
  - **References:** `REF-LOG`, `REF-ASVS-5`

### Secrets and keys

- **SEC-SECRET-1** Secrets (database, file-store, and email-provider credentials; signing keys) MUST NOT appear in source control, client code or build files, logs, or error output, and MUST differ per environment; the storage and injection mechanism is TO BE DECIDED (SQ-6).
  - **Applies to:** API application, background worker, build artifacts, clients
  - **Verification:** Secret-scanning in review and CI; log and error-output scans in tests
  - **References:** `REF-SECRETS`, `PRM-QUALITY`, `PRM-KMP`
- **SEC-SECRET-2** The API application and background worker SHOULD hold separately scoped credentials for the stores and email boundary, so compromise of one component does not expose the other's access.
  - **Applies to:** API ↔ stores boundary; email boundary
  - **Verification:** Credential-scope review once the platform is selected
  - **References:** `REF-SECRETS`
- **SEC-SECRET-3** Terraform configuration and state MUST NOT contain plaintext secrets; state MUST be stored remotely with encryption, locking, and restricted access. The state backend is TO BE DECIDED with the hosting platform (SQ-5).
  - **Applies to:** Deployment (Terraform)
  - **Verification:** State and configuration scans for secret material
  - **References:** `PRM-TF`, `REF-IAC`, `REF-SECRETS`

### Logging and error handling

- **SEC-LOG-1** The system MUST keep a security event log covering at minimum sign-ins, failed sign-ins, lockouts, credential/email changes (and the 2FA-equivalent lifecycle under CQ-1), exports, and account deletions, each with timestamp and source IP, retained at least 12 months (NFR-1.6). Authorization denials SHOULD also be logged. Interaction with FR-9.3 deletion is unresolved (SQ-7).
  - **Applies to:** Identity & session handling; REST API application
  - **Verification:** Tests asserting a log entry per listed event type
  - **References:** `REF-LOG`, `PRM-ABAC`
- **SEC-LOG-2** Application and security logs MUST NOT contain credentials, session tokens, file contents, or health data (NFR-1.7); logs record identifiers and event metadata only.
  - **Applies to:** All server components
  - **Verification:** Log-content scans in integration tests exercising health and document flows
  - **References:** `REF-LOG`
- **SEC-LOG-3** The user SHOULD be able to view their own recent security activity in-app (NFR-1.6).
  - **Applies to:** REST API application; clients
  - **Verification:** Functional test of the security-activity view
  - **References:** `REF-LOG`
- **SEC-ERR-1** Client-facing errors MUST state in plain language what happened and what to do next, and MUST NOT expose stack traces, framework identifiers, queries, or internal paths (NFR-5.3); full diagnostics are retained server-side subject to SEC-LOG-2, using a single centralized exception-handling path.
  - **Applies to:** All API responses and client error surfaces
  - **Verification:** Error-path tests (including malformed uploads and storage failures) asserting sanitized responses
  - **References:** `REF-ERROR`, `PRM-QUALITY`

### External integrations

- **SEC-EXT-1** Outbound email is the only permitted external integration in v1 (D-7); introducing any other external call or third-party service MUST be treated as a scope change requiring a security review and an update to this document.
  - **Applies to:** All server components
  - **Verification:** Dependency and network-egress review per release
  - **References:** `REF-SUPPLY`, `REF-API-2023`
- **SEC-EXT-2** Emailed action links (verification, and account recovery in its eventual CQ-1 form) MUST be single-use and time-limited; the verification-link lifetime for FR-1.1 and the recovery form are TO BE DECIDED.
  - **Applies to:** Identity flows via the email boundary
  - **Verification:** Reuse and expiry tests on emailed links
  - **References:** `REF-AUTH`, `PRM-RECOVERY`
- **SEC-EXT-3** Email-provider credentials are subject to SEC-SECRET-1; the provider, transport protection, and sender-authentication setup are TO BE DECIDED with the provider choice.
  - **Applies to:** System ↔ email provider boundary
  - **Verification:** Configuration review once a provider is selected
  - **References:** `REF-SECRETS`

### Deployment, CI/CD, and infrastructure as code

- **SEC-DEPLOY-1** All infrastructure MUST be defined in version-controlled Terraform and changed only through reviewed changes; manual console changes to production MUST NOT be routine practice, and IaC static scanning SHOULD run from the first deployment.
  - **Applies to:** Deployment model (Terraform, ARCHITECTURE.md)
  - **Verification:** Drift detection between Terraform state and live infrastructure; scanner output in CI
  - **References:** `PRM-TF`, `REF-IAC`, `REF-SSDF`
- **SEC-DEPLOY-2** Service and deployment identities MUST follow least privilege: the API and worker identities access only their own stores and the email boundary; deployment credentials are scoped to what Terraform manages. Specific identity mechanisms are TO BE DECIDED with the hosting platform (SQ-5).
  - **Applies to:** API ↔ stores boundary; deployment pipeline
  - **Verification:** Permission review of each identity once the platform is selected
  - **References:** `REF-IAC`, `REF-CICD`, `PRM-TF`
- **SEC-DEPLOY-3** The CI/CD system is UNKNOWN; when selected, the pipeline MUST protect build integrity, isolate secrets from build logs, run dependency-vulnerability and IaC scans, and gate releases on SEC-DEPLOY-4 (SQ-6).
  - **Applies to:** Future CI/CD pipeline
  - **Verification:** Pipeline configuration review at selection time
  - **References:** `REF-CICD`, `REF-SSDF`, `REF-SUPPLY`
- **SEC-DEPLOY-4** A release MUST NOT ship with open critical- or high-severity security findings (NFR-1.8), and third-party dependencies with known critical vulnerabilities MUST be patched or mitigated before release and within 30 days when discovered post-release (NFR-1.9).
  - **Applies to:** Release process
  - **Verification:** Release checklist gate; dependency-vulnerability scan results
  - **References:** `REF-VULNDEP`, `REF-SSDF`

## Requirement and Architecture Traceability

Component names are exactly those in `ARCHITECTURE.md`: Web client (Compose MP), Mobile client (Compose MP), REST API application (Quarkus/Kotlin), Identity & session handling, Data persistence (relational, 3NF), File store (uploaded originals), Background processing, Outbound email boundary (external). Trust boundaries 1–3 are ARCHITECTURE.md's **Trust boundaries**. Status: `CONFIRMED` = backed by an explicit documented requirement; `PROVISIONAL` = safe default not explicitly required; `PARTIALLY DEFINED` = requirement exists but a material decision is open; `TO BE DECIDED` = not yet mappable or wholly dependent on an open decision.

| Rule | Requirements protected | Boundary / component | Status |
| --- | --- | --- | --- |
| SEC-TRUST-1 | FR-1.9; all FR validation rules | Trust boundary 1; REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-TRUST-2 | NFR-1.4 | Trust boundary 2; Data persistence (relational, 3NF); File store (uploaded originals) | CONFIRMED |
| SEC-TRUST-3 | NFR-1.5 | REST API application (Quarkus/Kotlin); File store (uploaded originals) | CONFIRMED |
| SEC-AUTHN-1 | Replaces FR-1.2/FR-1.3 under CQ-1 | Identity & session handling | PROVISIONAL |
| SEC-AUTHN-2 | NFR-2.3 (supporting) | Identity & session handling | PROVISIONAL |
| SEC-AUTHN-3 | FR-1.1, FR-1.7 | Identity & session handling; Outbound email boundary (external) | CONFIRMED |
| SEC-AUTHN-4 | FR-1.4, NFR-1.6 | Identity & session handling | PARTIALLY DEFINED |
| SEC-AUTHN-5 | FR-1.3, FR-1.6 | Identity & session handling | CONFIRMED |
| SEC-AUTHN-6 | Replaces FR-1.6 under CQ-1 | Identity & session handling | TO BE DECIDED |
| SEC-AUTHN-7 | FR-9.3 | Identity & session handling | PARTIALLY DEFINED |
| SEC-AUTHN-8 | Replaces FR-1.7/FR-1.8 lifecycle under CQ-1 | Identity & session handling; Outbound email boundary (external) | PROVISIONAL |
| SEC-SESSION-1 | FR-1.5 | Identity & session handling | CONFIRMED |
| SEC-SESSION-2 | FR-1.10 | Identity & session handling | PARTIALLY DEFINED |
| SEC-SESSION-3 | FR-1.10, NFR-1.7 | Trust boundary 1; Identity & session handling | TO BE DECIDED |
| SEC-SESSION-4 | FR-1.9 (supporting) | Identity & session handling | PROVISIONAL |
| SEC-SESSION-5 | FR-1.6, FR-1.7 | Identity & session handling | PARTIALLY DEFINED |
| SEC-SESSION-6 | — (no direct requirement; `PRM-KMP` safe default) | Mobile client (Compose MP) | PROVISIONAL |
| SEC-AUTHZ-1 | FR-1.9 | REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-AUTHZ-2 | FR-1.9 | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-AUTHZ-3 | FR-1.9 | REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-AUTHZ-4 | FR-1.9, NFR-1.4 | REST API application (Quarkus/Kotlin); File store (uploaded originals) | CONFIRMED |
| SEC-AUTHZ-5 | FR-1.9 (supporting) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-HTTP-1 | NFR-1.1 | Trust boundary 1 | CONFIRMED |
| SEC-HTTP-2 | — (REST style, ARCHITECTURE.md inputs) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-HTTP-3 | NFR-1.5 (supporting) | Web client (Compose MP); REST API application (Quarkus/Kotlin) | TO BE DECIDED |
| SEC-HTTP-4 | — | REST API application (Quarkus/Kotlin) | TO BE DECIDED |
| SEC-HTTP-5 | D-5, NFR-2.3 (supporting) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-HTTP-6 | FR-5.6, FR-9.2 (abuse surface) | REST API application (Quarkus/Kotlin); Background processing | TO BE DECIDED |
| SEC-INPUT-1 | FR-2.1, FR-3.1, FR-3.10, FR-4.1, FR-4.3, FR-5.2, FR-6.1, FR-7.1 | REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-INPUT-2 | NFR-1.8 (supporting) | REST API application (Quarkus/Kotlin); Background processing; Data persistence (relational, 3NF) | PROVISIONAL |
| SEC-INPUT-3 | FR-3.1 | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-INPUT-4 | FR-1.9, NFR-6.1 (server-controlled fields) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-FILE-1 | NFR-1.4, FR-9.1, FR-9.2 | File store (uploaded originals); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-FILE-2 | NFR-1.5, FR-3.5 | REST API application (Quarkus/Kotlin); Web client (Compose MP) | PARTIALLY DEFINED |
| SEC-FILE-3 | FR-3.2 (filename as metadata) | File store (uploaded originals) | PROVISIONAL |
| SEC-FILE-4 | FR-3.9 | Background processing | PARTIALLY DEFINED |
| SEC-FILE-5 | FR-3.5 | File store (uploaded originals); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-OUT-1 | NFR-1.5 (client-side complement) | Web client (Compose MP); Mobile client (Compose MP) | PROVISIONAL |
| SEC-OUT-2 | FR-7.6, FR-9.1, FR-9.2 | REST API application (Quarkus/Kotlin); Background processing | PROVISIONAL |
| SEC-OUT-3 | FR-6.1 (contact fields, conditional) | Web client (Compose MP); Mobile client (Compose MP) | PROVISIONAL |
| SEC-DATA-1 | NFR-1.2 | Data persistence (relational, 3NF); File store (uploaded originals) | CONFIRMED |
| SEC-DATA-2 | NFR-2.3, D-5 | All components | CONFIRMED |
| SEC-DATA-3 | NFR-2.1 | Web client (Compose MP); Mobile client (Compose MP); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-DATA-4 | FR-9.2, FR-9.3, NFR-2.4 | Background processing; Data persistence (relational, 3NF); File store (uploaded originals) | CONFIRMED |
| SEC-DATA-5 | NFR-2.5 | UNKNOWN (no incident-response owner documented) | PARTIALLY DEFINED |
| SEC-DATA-6 | NFR-1.7 (extension); trust boundary 3 | Outbound email boundary (external) | PROVISIONAL |
| SEC-SECRET-1 | NFR-1.7 (extension) | All server components; Web client (Compose MP); Mobile client (Compose MP) | PROVISIONAL |
| SEC-SECRET-2 | NFR-1.4 (supporting) | Trust boundaries 2 and 3 | PROVISIONAL |
| SEC-SECRET-3 | — (Terraform, ARCHITECTURE.md inputs) | Deployment (no ARCHITECTURE.md component; Terraform model) | PROVISIONAL |
| SEC-LOG-1 | NFR-1.6 | Identity & session handling; REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-LOG-2 | NFR-1.7 | All server components | CONFIRMED |
| SEC-LOG-3 | NFR-1.6 | REST API application (Quarkus/Kotlin); Web client (Compose MP); Mobile client (Compose MP) | CONFIRMED |
| SEC-ERR-1 | NFR-5.3, FR-1.3, FR-1.9 (disclosure) | REST API application (Quarkus/Kotlin); Web client (Compose MP); Mobile client (Compose MP) | CONFIRMED |
| SEC-EXT-1 | D-7 | Outbound email boundary (external) | CONFIRMED |
| SEC-EXT-2 | FR-1.1, FR-1.6 (adapted under CQ-1) | Identity & session handling; Outbound email boundary (external) | PARTIALLY DEFINED |
| SEC-EXT-3 | NFR-1.1 (transport) | Outbound email boundary (external) | TO BE DECIDED |
| SEC-DEPLOY-1 | — (Terraform, ARCHITECTURE.md inputs) | Deployment (Terraform model) | PROVISIONAL |
| SEC-DEPLOY-2 | NFR-1.4 (supporting) | Trust boundary 2; Deployment (Terraform model) | PARTIALLY DEFINED |
| SEC-DEPLOY-3 | NFR-1.8, NFR-1.9 (gating) | Future CI/CD (UNKNOWN) | TO BE DECIDED |
| SEC-DEPLOY-4 | NFR-1.8, NFR-1.9 | Release process | CONFIRMED |

## Dependency Security Rules

Prospective rules for future implementation; no dependency exists yet and none has been assessed. Supply-chain references: `REF-SUPPLY`, `REF-VULNDEP`, `REF-CICD`, `REF-SSDF`.

- **DEP-1** The project MUST NOT add a dependency when the standard library or a small amount of straightforward, non-security-sensitive first-party code is safer and sufficient. The project MUST NOT replace vetted cryptography, authentication, authorization, protocol parsing, output encoding, HTML sanitization, or other security-critical functionality with custom code merely to avoid a dependency.
- **DEP-2** The project SHOULD prefer zero new dependencies. Every new dependency MUST be justified in the pull request description, including its purpose and why existing code or platform functionality is insufficient.
- **DEP-3** A new dependency MUST show evidence of active maintenance through a stable release, security response, or substantive maintainer activity within the previous 12 months. A mature project with less frequent releases requires an explicit documented exception and evidence that security reports are still handled.
- **DEP-4** The project MUST use the latest stable release from the latest actively supported major version unless a documented compatibility constraint requires another actively supported major version. Deprecated, abandoned, end-of-life, or pre-release packages MUST NOT be introduced into production.
- **DEP-5** Before a dependency is added or updated, direct and transitive dependencies MUST be checked for known vulnerabilities. A dependency with a known unpatched vulnerability applicable to the intended use MUST NOT be introduced without explicit, time-bounded risk acceptance, documented compensating controls, and a remediation plan.
- **DEP-6** Dependency review MUST include the complete transitive dependency graph. A small direct dependency with a disproportionately large, opaque, abandoned, or unvetted transitive tree SHOULD be rejected.
- **DEP-7** Production builds MUST resolve dependencies to exact versions through a committed lockfile or equivalent ecosystem mechanism. Production and CI builds MUST use frozen or reproducible dependency resolution and MUST NOT resolve floating versions at build or deployment time.
- **DEP-8** When multiple suitable libraries exist, the project SHOULD prefer the library with the narrowest required scope, smallest dependency tree, active security response process, clear provenance, and established security track record.

## Prompt Placeholders To Resolve

| Placeholder | Path | Status | Notes |
| --- | --- | --- | --- |
| `{{CODE_QUALITY_PROMPT}}` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Code Quality/00 General Code Quality Prompts/PROMPT.md` | RESOLVED | Read in full; imported as `PRM-QUALITY` (fail-closed decisions, centralized validation, testable security logic). |
| `{{API_SECURITY_PROMPT}}` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Web and API Security` | PARTIALLY RESOLVED | Directory. Seven stack-relevant sub-prompts read (`PRM-XSS`, `PRM-SQLI`, `PRM-CSRF`, `PRM-API`, `PRM-UPLOAD`, `PRM-SSWEB`, `PRM-RATE`). Remaining sub-prompts deferred or not applicable pending SQ-3 (JWT, CSRF finalization), SQ-4 (CSP, CORS), and SQ-5 (database encryption); SSRF/XXE/WebSocket/webhook/tRPC/OpenAPI/API-key prompts not applicable to the documented system. |
| `{{BACKEND_FRAMEWORK_PROMPT}}` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Backend Frameworks/Java/07 Secure Quarkus Developer/PROMPT.md` | RESOLVED | Read in full; imported as `PRM-QUARKUS` (Quarkus is the selected server framework). Its OIDC/JWT and LLM-integration portions do not apply (passkey-only direction; no LLM integration documented). |
| `{{FRONTEND_FRAMEWORK_PROMPT}}` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Mobile/04 Secure Mobile Kotlin Multiplatform Developer/PROMPT.md` | PARTIALLY RESOLVED | Read in full; imported as `PRM-KMP` for the Compose MP clients. Web-client-specific guidance remains open until CQ-2 decides the web client implementation (SQ-4). |
| `{{AUTH_PROMPT}}` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Authorization/02 ABAC Architect/PROMPT.md` | PARTIALLY RESOLVED | Read in full; imported as `PRM-ABAC` for enforcement doctrine only (deny-by-default, fail-closed, single enforcement layer); full ABAC/policy-engine architecture is not applicable to the single-role, owner-only model. Because passkeys are the selected mechanism, the adjacent `Authentication/` prompts for passwordless authentication and account recovery were additionally read and imported (`PRM-PASSWORDLESS`, `PRM-RECOVERY`); the remaining `Authentication/` prompts are deferred as noted under **Local prompt imports**. |
| `{{DEPLOYMENT_PROMPT}}` | `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/Infrastructure/Terraform` | PARTIALLY RESOLVED | Directory. Generic prompt `00 Secure Terraform Developer/PROMPT.md` read and imported as `PRM-TF`. Provider-specific prompts (AWS/Azure/GCP/CDK) not applicable until the hosting platform is decided (SQ-5). |

## Open Security Questions

- **SQ-1** Which privacy regimes bind v1? The security notes name HIPAA and other privacy regulation, while REQUIREMENTS.md OQ-1 leaves jurisdictions TO BE DECIDED, and HIPAA's applicability to a consumer-controlled personal-records product (no covered entity documented) is unconfirmed. This is a material tension between the security notes and REQUIREMENTS.md; it drives breach notification (SEC-DATA-5), retention, and session lifetimes (FR-1.10).
- **SQ-2** CQ-1: passkey-only authentication contradicts FR-1.1–FR-1.8's password/TOTP model. Which FR-1 requirements are replaced, what is the account-recovery flow for lost passkeys, and what re-authentication satisfies FR-9.3? Blocks SEC-AUTHN-1/-6/-7 finalization.
- **SQ-3** Session token mechanics: cookie-based or bearer? Decides whether CSRF defenses and cookie attributes apply (SEC-SESSION-3, `PRM-CSRF`) or token-storage rules do, and shapes SEC-HTTP-4.
- **SQ-4** CQ-2: is the web client Compose/Wasm (canvas) or a separate web implementation? Determines the XSS surface, the security-header/CSP set (SEC-HTTP-3), and the NFR-1.5 file-delivery mechanism (SEC-FILE-2).
- **SQ-5** Hosting platform and store technologies (database product, file-store technology): decide encryption-at-rest mechanisms and key management (SEC-DATA-1), network isolation for trust boundary 2, service identities (SEC-DEPLOY-2), the Terraform state backend (SEC-SECRET-3), and which provider-specific IaC guidance applies.
- **SQ-6** CI/CD platform and secret-storage mechanism: blocks SEC-DEPLOY-3 and the injection mechanism in SEC-SECRET-1.
- **SQ-7** NFR-1.6 requires retaining the security event log (with source IPs) at least 12 months, while FR-9.3 requires permanent deletion of all personal data within 30 days of account deletion. Does the security log survive account deletion, on what basis, and in what form? Material conflict between two confirmed requirements.
- **SQ-8** Is the formal assurance target OWASP ASVS 5.0.0 Level 2 (NFR-1.8 says "current version" Level 2), and who performs verification and when?
- **SQ-9** Abuse and resource limits beyond FR-1.4: per-user API rate limits, export-job frequency, and email-sending caps (reminders can trigger unbounded email). Values for SEC-HTTP-6 are TO BE DECIDED.
- **SQ-10** Does malware scanning (FR-3.9, SHOULD) ship in v1, and with which engine?
- **SQ-11** Incident-response ownership and process: who detects, triages, and notifies under NFR-2.5? No input document assigns this.
- **SQ-12** What may reminder and notification emails contain, given appointment purpose and provider names are health-adjacent data crossing the only external boundary (SEC-DATA-6)?
