# AfterImpact — Security

This document owns the security posture: the threat model, all security and privacy requirements, the controls that enforce them, and trust-boundary enforcement. `REQUIREMENTS.md` owns WHAT the system does; `ARCHITECTURE.md` owns components, boundaries, and data flow, and defines the `UNKNOWN` / `TO BE DECIDED` / `ASSUMPTION` markers used here; `DESIGN.md` owns client-facing interaction conventions.

Security requirements carry `NFR-1.*` (security) and `NFR-2.*` (privacy) IDs, stated inline on the rule that owns each one. Controls carry `SEC-*` IDs; dependency rules carry `DEP-*`. No implementation exists yet, so every rule here is prospective and no conformance is claimed.

## Security Profile

- **Sensitive data:** Every record the system holds is treated as sensitive personal data because it includes health information (D-5) — the entities in `REQUIREMENTS.md` **Data Entities**, plus passkey credential material and the security event log. Contact records and journal narrative additionally hold personal data about **third parties** (the other driver, witnesses, employer, adjusters) who are not the account owner; whether the OQ-1/SQ-1 regime creates obligations toward these non-user data subjects is SQ-15 (T-49, T-50).
- **Actors and authorization model:** One actor, the accident victim, as sole account owner; a single role; owner-only access to every record (D-2, FR-1.9).
- **Authentication model:** Passkey-only (FR-1.2, D-9); there is no shared-secret credential to protect (SEC-AUTHN-9).
- **Trust boundaries:** The three boundaries in `ARCHITECTURE.md` **Trust boundaries**, referred to below as boundaries 1, 2, and 3.
- **External attack surface:** Boundary 1 (clients) and boundary 3 (outbound email) only; no inbound third-party integration exists in v1 (D-7).
- **Regulatory obligations:** TO BE DECIDED (OQ-1, narrowed by SQ-1). D-5 requires treating all data as sensitive regardless of the outcome.
- **Assurance target:** NFR-1.8 mandates OWASP ASVS (current version) Level 2; confirming ASVS 5.0.0 Level 2 as the formal target, and who verifies it, is SQ-8.
- **Threat model:** AUTHORED (2026-08-13). The **Threat Model** section below enumerates threats `T-1`–`T-51` against the documented system, naming the product-specific adversaries — the hostile opposing party in the insurance/legal dispute, the household adversary with device access, the attacker controlling the user's email account, the malicious or compromised infrastructure operator, and automated/supply-chain abuse — alongside the everyday actors. Each threat cites its existing controls or the open question it waits on. The rules below are the controls the table references (T-45).

## References

### Public references

Cheat Sheet Series pages are unversioned living documents, accessed 2026-08-13. ASVS 5.0.0 is cited generally; no individual ASVS requirement identifier has been verified against the release yet.

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

### Local prompt imports

Base path: `/Users/jmanico/Dropbox/github/platform/context/prompts/code security/`. Guidance from each file was converted into the project-specific rules that cite its `PRM-*` ID; nothing is copied verbatim.

| ID | Path (relative to base) | Purpose as applied here |
| --- | --- | --- |
| `PRM-QUALITY` | `Code Quality/00 General Code Quality Prompts/PROMPT.md` | Fail-closed security decisions, centralized boundary validation, testable isolated security logic |
| `PRM-QUARKUS` | `Backend Frameworks/Java/07 Secure Quarkus Developer/PROMPT.md` | Quarkus API hardening: endpoint auth, parameterized data access, safe serialization, headers (OIDC/JWT and LLM portions not applicable) |
| `PRM-KMP` | `Mobile/04 Secure Mobile Kotlin Multiplatform Developer/PROMPT.md` | Compose MP client rules: platform secure storage, HTTPS-only shared client, strict deserialization |
| `PRM-ABAC` | `Authorization/02 ABAC Architect/PROMPT.md` | Enforcement doctrine only (deny-by-default, fail-closed, single enforcement layer); full ABAC/policy-engine content not applicable to the single-role model |
| `PRM-PASSWORDLESS` | `Authentication/06 Secure Passwordless Authentication Developer/PROMPT.md` | WebAuthn/passkey ceremony validation, opaque user handles, credential lifecycle management |
| `PRM-RECOVERY` | `Authentication/03 Secure Account Recovery Developer/PROMPT.md` | Recovery/verification token hygiene, enumeration-safe recovery responses, session invalidation on recovery |
| `PRM-XSS` | `Web and API Security/03 Secure XSS Prevention Developer/PROMPT.md` | Text-only rendering of user data, security headers for browser delivery |
| `PRM-SQLI` | `Web and API Security/04 Secure SQL Injection Prevention Developer/PROMPT.md` | Parameterized queries only, identifier allowlists, least-privilege DB account |
| `PRM-CSRF` | `Web and API Security/05 Secure CSRF Prevention Developer/PROMPT.md` | Conditional CSRF rules, applicable only if cookie sessions are selected (SQ-3) |
| `PRM-API` | `Web and API Security/06 Secure API Developer/PROMPT.md` | Object-level authorization, DTO binding against mass assignment, no-store on sensitive responses |
| `PRM-UPLOAD` | `Web and API Security/08 Secure File Upload Developer/PROMPT.md` | Content inspection, server-generated storage names, non-executing delivery |
| `PRM-SSWEB` | `Web and API Security/12 Secure Server-Side Web Application Developer/PROMPT.md` | Server-side baseline: fail-closed authorization, server-side sessions, production configuration (password-specific portions not applicable under D-9) |
| `PRM-RATE` | `Web and API Security/13 Secure API Rate Limiting Developer/PROMPT.md` | Limits on authentication, upload, export, and email-triggering endpoints |
| `PRM-TF` | `Infrastructure/Terraform/00 Secure Terraform Developer/PROMPT.md` | Provider-agnostic Terraform rules: no secrets in code/state, encrypted state, least-privilege identities, IaC scanning |

Deferred or not applicable: `Web and API Security` sub-prompts for SSRF, XXE, database encryption, CORS, JWT, WebSocket, CSP, rate-limited API keys, webhooks, tRPC, and OpenAPI validation — either outside the documented system or blocked on SQ-3, SQ-4, or SQ-5. Provider-specific Terraform prompts (AWS/Azure/GCP/CDK) await the hosting decision (SQ-5). `Authentication/` sub-prompts for password storage, MFA, credential-stuffing defense, and SSO are password-centric or not applicable under D-9; session management is covered by `REF-SESSION` and `PRM-SSWEB`, to be revisited at SQ-3.

## Security and Privacy Rules

Grouped by capability; one verifiable behavior per rule. A rule that carries an `NFR-*` ID **is** that requirement — it is stated here and nowhere else. Per-rule status is in **Traceability**; `TO BE DECIDED` inside a rule marks a value no document provides.

### Trust boundaries and server-side enforcement

- **SEC-TRUST-1** The REST API application MUST enforce every business rule, validation, and authorization decision server-side; client-side checks are responsiveness aids only and MUST NOT be relied on at boundary 1.
  - **Applies to:** All operations crossing trust boundary 1
  - **Verification:** API-level tests exercising every rule with the client bypassed (direct HTTP)
  - **References:** `REF-ASVS-5`, `REF-PC-2024`, `REF-INPUT`
- **SEC-TRUST-2** The RDBMS, the file store, backup artifacts, and any additional store that holds personal data (for example a separate search index, if the database-search ASSUMPTION in `ARCHITECTURE.md` is revisited) MUST be reachable only by the API application and background worker; no client, public network path, or other component may cross boundary 2. The rule binds any store holding personal data, not only the two originally named (T-19, T-31).
  - **Applies to:** API ↔ stores boundary; backup storage; any derived personal-data store
  - **Verification:** Infrastructure review plus network reachability tests from outside the private boundary
  - **References:** `REF-ASVS-5`, `REF-IAC`, `PRM-TF`
- **SEC-TRUST-3** (**NFR-1.5**) User-uploaded content MUST be treated as hostile whenever it crosses back toward clients, and MUST be delivered such that it cannot execute as active content (scripts/HTML) in the application's origin.
  - **Applies to:** Document view/download paths through the API
  - **Verification:** Tests that uploaded HTML/SVG/script content never executes in the app origin
  - **References:** `REF-XSS`, `PRM-UPLOAD`

### Authentication

- **SEC-AUTHN-1** Authentication MUST use passkeys (WebAuthn) (FR-1.2); the server MUST validate each registration and assertion ceremony fully server-side — challenge freshness and single use, origin and RP ID binding, signature — and MUST NOT accept client-asserted authentication results.
  - **Applies to:** Identity & session handling; sign-in and passkey registration ceremonies
  - **Verification:** Negative tests with replayed challenges, wrong origin/RP ID, and tampered assertions
  - **References:** `REF-WEBAUTHN`, `REF-FIDO`, `REF-63B-4`, `PRM-PASSWORDLESS`
- **SEC-AUTHN-2** WebAuthn user handles MUST be opaque random values that encode no email address or other identifying data, and stored credential material MUST be limited to what verification requires (credential ID, public key, per-owner binding).
  - **Applies to:** Identity & session handling credential storage
  - **Verification:** Schema review; test that user handles are random and non-identifying
  - **References:** `REF-WEBAUTHN`, `PRM-PASSWORDLESS`
- **SEC-AUTHN-3** The system MUST require email verification before an account becomes usable for sign-in, and re-verification when the email address changes (FR-1.1, FR-1.7).
  - **Applies to:** Registration and email-change flows
  - **Verification:** Functional tests: unverified accounts cannot sign in; email change requires re-verification
  - **References:** `REF-AUTH`, `PRM-RECOVERY`
- **SEC-AUTHN-4** The system MUST throttle failed authentication attempts per account at the FR-1.4 thresholds and MUST log each lockout to the security event log. Because a purely account-scoped block is a renewable denial-of-service lever against the owner (T-1), the system MUST also apply source-based and anti-automation throttling ahead of the account block (values under SQ-9), and the account block MUST NOT prevent a successful valid passkey ceremony by the owner (a valid assertion is proof of possession, not a guess).
  - **Applies to:** Identity & session handling
  - **Verification:** Automated tests driving repeated failures, asserting lockout, source-throttling, and log entry; test that a valid ceremony succeeds during an account block
  - **References:** `REF-AUTH`, `PRM-RATE`
- **SEC-AUTHN-5** All authentication and account-mail flows MUST respond identically whether or not an account exists; no response content, observable behavior, or email side effect may disclose account existence (FR-1.3).
  - **Applies to:** Sign-in, registration, recovery, email change
  - **Verification:** Response-equivalence tests for existing vs. non-existing accounts
  - **References:** `REF-AUTH`, `REF-WEBAUTHN`, `REF-ERROR`
- **SEC-AUTHN-6** The account-recovery flow required by FR-1.6 is TO BE DECIDED (SQ-2). When designed, it MUST NOT provide a persistent authentication path weaker than the primary passkey flow, MUST NOT rely on emailed codes alone as an authentication factor, and completing recovery MUST invalidate all existing sessions.
  - **Applies to:** Identity & session handling
  - **Verification:** Design review against `REF-63B-4`; session-invalidation test on recovery completion
  - **References:** `REF-63B-4`, `REF-AUTH`, `REF-FIDO`, `PRM-RECOVERY`
- **SEC-AUTHN-7** Account deletion MUST require a fresh passkey authentication ceremony by the account owner, completed within the same session and immediately before deletion (FR-9.3).
  - **Applies to:** Account deletion flow
  - **Verification:** Test that deletion is refused without a fresh re-authentication ceremony
  - **References:** `REF-AUTH`, `REF-63B-4`
- **SEC-AUTHN-8** Credential registration and revocation (FR-1.8) MUST be available to the signed-in user, MUST refuse a revocation that would leave the account with no usable authentication path, and MUST send a notification to the verified email address.
  - **Applies to:** Identity & session handling; credential lifecycle
  - **Verification:** Functional tests of the credential list, revocation guard, and notification
  - **References:** `PRM-PASSWORDLESS`, `REF-FIDO`
- **SEC-AUTHN-9** (**NFR-1.3**) The system MUST NOT store or accept any user-chosen shared secret — password, PIN, or knowledge-based answer — as an authentication factor. If D-9 is ever revisited, introducing one requires a security review, and any such credential MUST be stored using a memory-hard adaptive hashing scheme consistent with current OWASP Password Storage guidance, never recoverable by the system.
  - **Applies to:** Identity & session handling
  - **Verification:** Schema review asserting no shared-secret credential field exists
  - **References:** `REF-63B-4`, `REF-AUTH`, `PRM-PASSWORDLESS`
- **SEC-AUTHN-10** High-impact account-security and full-data-egress operations MUST require a fresh passkey authentication ceremony by the owner, completed within the same session immediately before the operation — extending the SEC-AUTHN-7 pattern (which already guards account deletion) to: email-address change (FR-1.7), passkey registration and revocation (FR-1.8), and full-account export (FR-9.2). This closes the asymmetry whereby a transient session compromise could silently take over the account or exfiltrate the entire record set without re-proving possession of a passkey (T-3, T-11).
  - **Applies to:** Identity & session handling; REST API application; Background processing (export)
  - **Verification:** Tests that each listed operation is refused without a fresh re-authentication ceremony in the same session
  - **References:** `REF-AUTH`, `REF-63B-4`, `PRM-PASSWORDLESS`
- **SEC-AUTHN-11** The system MUST send an out-of-band notification to the account's verified email address when a security-significant event occurs, so an attacker operating a stolen session cannot act entirely silently through the in-app feed they control (T-3, T-4, T-11, T-27, T-46). At minimum: email-address change (to **both** the prior and the new address), full-account export creation (FR-9.2), account-deletion initiation (FR-9.3), and case deletion (FR-2.6); passkey registration and revocation are already notified under SEC-AUTHN-8. Whether a sign-in from a not-previously-recognized credential or context also notifies, and the recognition heuristic, is TO BE DECIDED (SQ-16). These are security messages: they MUST NOT be suppressible by the FR-5.6 email-reminder toggle and MUST honor SEC-DATA-6 (no health or document content in the mail).
  - **Applies to:** Identity & session handling; Background processing; Outbound email boundary (external)
  - **Verification:** Tests asserting a notice per listed event, delivery to the prior address on email change, and non-suppression by the reminder toggle
  - **References:** `REF-AUTH`, `PRM-PASSWORDLESS`, `PRM-RECOVERY`

### Session management

- **SEC-SESSION-1** The server MUST invalidate a session immediately on sign-out (FR-1.5); subsequent requests presenting that session MUST be rejected. Client-side state clearing alone is insufficient.
  - **Applies to:** All authenticated API operations
  - **Verification:** Test replaying a signed-out session credential against protected endpoints
  - **References:** `REF-SESSION`
- **SEC-SESSION-2** The server MUST enforce an absolute session lifetime and SHOULD enforce an idle timeout, both server-side, at the FR-1.10 values.
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
- **SEC-SESSION-5** Completion of account recovery (FR-1.6), an email-address change (FR-1.7), or any credential registration or revocation (FR-1.8) MUST invalidate all other active sessions. (Email change added per T-3: without it, an attacker who changes the address from a stolen session keeps every other session alive.)
  - **Applies to:** Identity & session handling
  - **Verification:** Test that concurrent sessions are invalidated after account recovery, email change, or credential change
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
- **SEC-AUTHZ-4** Object-level authorization MUST NOT depend on identifiers being unguessable; every file fetch and record access is authorized regardless of how the identifier was obtained.
  - **Applies to:** Documents, export artifacts, and all record endpoints
  - **Verification:** Tests fetching known-valid foreign identifiers; static check that no unauthenticated file routes exist
  - **References:** `REF-API-2023`, `REF-ASVS-5`
- **SEC-AUTHZ-5** Ownership checks MUST be centralized in a single enforcement layer that every endpoint passes through, so no individual handler can omit the check.
  - **Applies to:** REST API application
  - **Verification:** Architecture review; static analysis that no endpoint bypasses the enforcement layer
  - **References:** `PRM-ABAC`, `PRM-QUALITY`

### HTTP/API boundary

- **SEC-HTTP-1** (**NFR-1.1**) All client–server communication MUST use TLS 1.2 or higher; the system MUST NOT serve any functionality over cleartext connections. Protocol and cipher configuration beyond that floor is TO BE DECIDED with the hosting platform (SQ-5).
  - **Applies to:** Client ↔ API boundary
  - **Verification:** TLS configuration scan; test that no cleartext endpoint serves functionality
  - **References:** `REF-REST`, `REF-ASVS-5`
- **SEC-HTTP-2** The API MUST validate the HTTP method and declared content type of each request, rejecting unsupported methods and mismatched or unexpected content types rather than content-sniffing.
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

- **SEC-INPUT-1** The API MUST validate all untrusted input server-side at boundary 1, for both format and business meaning: accident/onset dates not in the future (FR-2.1, FR-4.1), severity integer 0–10 (FR-4.1, FR-4.3), expense amount > 0 with two decimals (FR-7.1), enumerated categories/statuses/roles (FR-3.2, FR-5.2, FR-6.1, FR-7.1), and file type/size/quota (FR-3.1, FR-3.10). Free-text fields (journal, notes) accept broad legitimate character sets; validation MUST NOT substitute for output encoding.
  - **Applies to:** Every write operation at the Client ↔ API boundary
  - **Verification:** Per-field negative tests via direct HTTP
  - **References:** `REF-INPUT`, `REF-PC-2024`, `PRM-QUALITY`
- **SEC-INPUT-2** All database access MUST use parameterized statements or an equivalent mechanism separating code from data; dynamic query construction from user input MUST NOT occur, and dynamic sort/filter identifiers MUST map through a server-side allowlist.
  - **Applies to:** API application and background worker access to the RDBMS
  - **Verification:** Static analysis for string-built queries; injection test suite asserting payloads are treated as literals
  - **References:** `PRM-SQLI`, `REF-INPUT`, `PRM-QUARKUS`
- **SEC-INPUT-3** Upload acceptance MUST be decided by server-side inspection of file content against the allowed type list (FR-3.1), not by file extension or client-declared content type alone, with size limits enforced before full buffering. The inspection mechanism is TO BE DECIDED.
  - **Applies to:** Document upload endpoint
  - **Verification:** Tests uploading mismatched extension/content pairs and oversize streams
  - **References:** `PRM-UPLOAD`, `REF-INPUT`
- **SEC-INPUT-4** Request bodies MUST bind to explicit request models; unknown fields and server-controlled fields (owner, identifiers, timestamps) MUST be rejected or ignored, never bound to persisted entities.
  - **Applies to:** All REST write endpoints
  - **Verification:** Tests submitting extra/privileged fields and asserting they have no effect
  - **References:** `REF-API-2023`, `PRM-API`, `PRM-QUARKUS`

### File and document handling

- **SEC-FILE-1** (**NFR-1.4**) Uploaded files and export artifacts MUST NOT be retrievable via unauthenticated or non-authorized requests: they MUST be stored only in the private file store and retrieved only by streaming through authenticated, owner-authorized API endpoints, with no public or guessable URLs. Every file fetch is subject to FR-1.9.
  - **Applies to:** File store; document/export download paths
  - **Verification:** Infrastructure review; tests fetching storage paths directly without API authorization
  - **References:** `REF-ASVS-5`, `PRM-UPLOAD`, `REF-API-2023`
- **SEC-FILE-2** Files served to clients MUST satisfy SEC-TRUST-3; the exact delivery mechanism (disposition headers, sandboxing, separate origin) is TO BE DECIDED with the web client model (CQ-2, SQ-4).
  - **Applies to:** In-app viewing (FR-3.5) and downloads
  - **Verification:** Tests that uploaded HTML/SVG/JS renders inert in the app origin
  - **References:** `PRM-UPLOAD`, `REF-XSS`, `PRM-QUARKUS`
- **SEC-FILE-3** Stored files MUST use server-generated storage names; the original filename is kept only as sanitized display metadata (FR-3.2 title default) and MUST never be used as a storage path.
  - **Applies to:** File store; upload pipeline
  - **Verification:** Storage-layout review; path-traversal tests with hostile filenames
  - **References:** `PRM-UPLOAD`
- **SEC-FILE-4** A file flagged by malware scanning (FR-3.9) MUST NOT become retrievable. The scanning engine and whether scanning ships in v1 are TO BE DECIDED (SQ-10).
  - **Applies to:** Upload pipeline; background processing
  - **Verification:** Test with a standard detection-test file once an engine is selected
  - **References:** `PRM-UPLOAD`
- **SEC-FILE-5** Downloads MUST return the byte-identical original (FR-3.5); the system MUST NOT transform stored originals in place, preserving the integrity of evidence documents.
  - **Applies to:** File store; download path
  - **Verification:** Round-trip hash comparison tests
  - **References:** `REF-ASVS-5`
- **SEC-FILE-6** Server-side processing that parses attacker-influenced file bytes — malware scanning (FR-3.9), any FR-3.4 OCR or file-content indexing, and the embedding of user files and text during FR-9.1/FR-9.2 generation — MUST run isolated from the API application with the least store access the job requires, so a parser or decoder exploit cannot gain the background worker's read-all/delete-all reach across boundary 2 (T-16, T-17). The in-app viewing path (FR-3.5) MUST be covered by the SEC-FILE-2 delivery-mechanism decision, which MUST address client decoder/codec exploitation, not only script execution. Because SEC-FILE-5 forbids sanitizing transforms, a format-valid weaponized upload is stored and re-served intact and may be bundled into exports; whether malware scanning (FR-3.9, SEC-FILE-4) ships in v1 to reduce redistribution is SQ-10, and if it is deferred the residual redistribution risk MUST be recorded as accepted. Isolation mechanism is TO BE DECIDED with the execution-model and hosting choices (SQ-5; `ARCHITECTURE.md` background-processing open decision).
  - **Applies to:** Background processing; File store; Document upload/download flow
  - **Verification:** Architecture review of parsing isolation and store-scope; SEC-FILE-2 mechanism review for decoder risk; SQ-10 risk-acceptance record if scanning is deferred
  - **References:** `PRM-UPLOAD`, `REF-INPUT`, `PRM-QUARKUS`

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

- **SEC-DATA-1** (**NFR-1.2**) All stored personal data — database contents, uploaded files, export artifacts, backups, and any additional store that holds personal data (e.g., a future search index) — MUST be encrypted at rest; algorithms and key management are TO BE DECIDED with the database, file-store, and hosting choices (SQ-5), which MUST weigh whether key custody can be separated from the data plane so at-rest encryption also constrains a privileged infrastructure operator (T-13, T-19, T-20).
  - **Applies to:** Data persistence; file store; backups
  - **Verification:** Infrastructure review once technologies are selected
  - **References:** `REF-ASVS-5`, `PRM-TF`
- **SEC-DATA-2** (**NFR-2.3**) The system MUST collect and store only data that serves the functional requirements, and MUST treat all of it as sensitive personal data (D-5) regardless of which regulatory regime is ultimately targeted.
  - **Applies to:** All entities and any telemetry
  - **Verification:** Schema and field review against REQUIREMENTS.md **Data Entities**
  - **References:** `REF-ASVS-5`, `REF-63B-4`
- **SEC-DATA-3** (**NFR-2.1**) Personal data MUST be used solely to provide the service to its owner; it MUST NOT be sold or shared with third parties, and authenticated screens MUST NOT include third-party advertising or marketing trackers.
  - **Applies to:** All clients and server integrations
  - **Verification:** Network-traffic inspection of authenticated sessions; dependency review
  - **References:** `REF-ASVS-5`
- **SEC-DATA-4** (**NFR-2.4**) Data portability and erasure MUST be honored through full-account export (FR-9.2) and account deletion (FR-9.3), within the timelines those requirements state.
  - **Applies to:** Background processing; all stores
  - **Verification:** Deletion tests asserting absence after purge windows, including the file store
  - **References:** `REF-ASVS-5`
- **SEC-DATA-5** (**NFR-2.5**) In the event of a personal-data breach, affected users MUST be notified without undue delay, consistent with applicable law. The governing jurisdictions are TO BE DECIDED (OQ-1, SQ-1) and incident-response ownership is UNKNOWN (SQ-11).
  - **Applies to:** Incident response (process TO BE DECIDED)
  - **Verification:** Incident-response runbook review once jurisdiction is decided
  - **References:** `REF-ASVS-5`
- **SEC-DATA-6** Outbound email MUST carry the minimum personal data needed for its purpose; health data and document contents MUST NOT appear in email bodies or subjects. What reminder emails may reference (e.g., appointment purpose or provider name) is TO BE DECIDED (SQ-12).
  - **Applies to:** System ↔ email provider boundary
  - **Verification:** Template review; tests asserting excluded fields never reach the email boundary
  - **References:** `REF-LOG`, `REF-ASVS-5`
- **SEC-DATA-7** (**NFR-2.2**) A privacy policy MUST be viewable before registration, and registration MUST record the user's acceptance of it. The policy MUST disclose the security event log's retention (SEC-LOG-1) once its survival of account deletion is resolved (SQ-7), and a material change to the policy MUST re-notify users and record fresh acceptance where the change is material (T-41).
  - **Applies to:** Registration flow; Web client; Mobile client
  - **Verification:** Functional test that registration is refused without recorded acceptance; test that a material policy change triggers re-notification and re-acceptance
  - **References:** `REF-ASVS-5`
- **SEC-DATA-8** A production restore under NFR-3.2 MUST NOT silently reintroduce security or privacy state the user changed since the backup point (T-32, T-48). On restore the system MUST invalidate all sessions and MUST re-apply, where the affected records survive, the credential revocations (FR-1.8) and account/record deletions (FR-2.6, FR-3.8, FR-9.3) recorded after the backup point; it SHOULD notify the affected user that a restore occurred so they can re-check their credential list. Backups MUST bound how long record-level deletions persist — a purged case or document MUST NOT remain recoverable from backups indefinitely (retention value TO BE DECIDED with SQ-5) — and restore-test copies materialized under NFR-3.2 MUST meet production data controls and be destroyed after verification (T-31, T-33). Responsibility for post-restore reconciliation is assigned to Background processing (`ARCHITECTURE.md`).
  - **Applies to:** Data persistence; File store; Background processing; backups
  - **Verification:** Restore drills asserting post-backup revocations and deletions do not resurrect and that all sessions are invalidated; backup-retention and restore-test-destruction review
  - **References:** `REF-ASVS-5`, `REF-63B-4`

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

- **SEC-LOG-1** (**NFR-1.6**) The system MUST keep a security event log covering at minimum sign-ins, failed sign-ins, lockouts, credential registration and revocation (FR-1.8), email changes, account recovery, exports, account deletions, and — added per T-11/T-12 — case, document, and record deletion (FR-2.6, FR-3.8), trash restoration, and permanent purge execution (FR-2.6, FR-3.8, FR-9.3); each with timestamp and source IP, retained at least 12 months. "Exports" covers every export form — full-account archive (FR-9.2), case-summary PDF (FR-9.1), and expense CSV (FR-7.6) — so no export escapes the reviewable record. Authorization denials SHOULD also be logged. Interaction with FR-9.3 deletion is unresolved (SQ-7).
  - **Applies to:** Identity & session handling; REST API application
  - **Verification:** Tests asserting a log entry per listed event type
  - **References:** `REF-LOG`, `PRM-ABAC`
- **SEC-LOG-2** (**NFR-1.7**) Application and security logs MUST NOT contain credentials, session tokens, file contents, or health data; logs record identifiers and event metadata only.
  - **Applies to:** All server components
  - **Verification:** Log-content scans in integration tests exercising health and document flows
  - **References:** `REF-LOG`
- **SEC-LOG-3** The user MUST be able to view their own recent security activity in-app (raised from SHOULD per T-27), giving them a first-party channel to detect account misuse that an attacker operating a stolen session cannot suppress. The view MUST surface at minimum the SEC-LOG-1 events attributable to the account (sign-ins, credential and email changes, exports, deletions).
  - **Applies to:** REST API application; clients
  - **Verification:** Functional test of the security-activity view showing the listed event types
  - **References:** `REF-LOG`
- **SEC-LOG-4** The security event log MUST be tamper-evident and append-only, with write access separated from the components that generate its entries, so a compromise of the REST API application or a privileged infrastructure operator cannot silently rewrite or delete the audit trail that would prove an exfiltration or takeover (T-34) — integrity that also matters as the victim's own evidence in an adversarial dispute. The mechanism is TO BE DECIDED with the store and hosting choices (SQ-5).
  - **Applies to:** Identity & session handling; REST API application; Data persistence
  - **Verification:** Tests that log entries cannot be modified or deleted through the API; design review of write-path separation
  - **References:** `REF-LOG`, `PRM-ABAC`
- **SEC-ERR-1** Client-facing errors MUST NOT expose stack traces, framework identifiers, queries, internal paths, or any other internal state; they carry only the plain-language content NFR-5.3 requires. Full diagnostics are retained server-side subject to SEC-LOG-2, produced through a single centralized exception-handling path.
  - **Applies to:** All API responses and client error surfaces
  - **Verification:** Error-path tests (including malformed uploads and storage failures) asserting sanitized responses
  - **References:** `REF-ERROR`, `PRM-QUALITY`

### External integrations

- **SEC-EXT-1** Outbound email is the only permitted external integration in v1 (D-7); introducing any other external call or third-party service MUST be treated as a scope change requiring a security review and an update to this document.
  - **Applies to:** All server components
  - **Verification:** Dependency and network-egress review per release
  - **References:** `REF-SUPPLY`, `REF-API-2023`
- **SEC-EXT-2** Emailed action links (verification, and account recovery in its eventual SQ-2 form) MUST be single-use and time-limited, and their token values MUST be stored server-side only as one-way hashes — the usable value existing solely in the outbound email — so database or backup read access cannot redeem a live link (T-6). An unverified account and its verification link MUST expire together; a registration attempt against an already-registered address MUST behave enumeration-safely (SEC-AUTHN-5) and MUST NOT create a second bindable account; and verification mail MUST state that inaction cancels the request (T-5). The verification-link lifetime for FR-1.1 is TO BE DECIDED.
  - **Applies to:** Identity flows via the email boundary
  - **Verification:** Reuse and expiry tests on emailed links; schema check that only token hashes are stored; test that unverified accounts expire and re-registration creates no second account
  - **References:** `REF-AUTH`, `PRM-RECOVERY`
- **SEC-EXT-3** Email-provider credentials are subject to SEC-SECRET-1; the provider, transport protection, and sender-authentication setup are TO BE DECIDED with the provider choice.
  - **Applies to:** System ↔ email provider boundary
  - **Verification:** Configuration review once a provider is selected
  - **References:** `REF-SECRETS`
- **SEC-EXT-4** The sending domain MUST enforce sender authentication (an SPF/DKIM/DMARC-class configuration; exact mechanism TO BE DECIDED with the provider under SEC-EXT-3) so AfterImpact-branded mail cannot be exact-domain-spoofed to an injured or medicated victim (T-28). Legitimate system mail MUST NOT solicit secrets, credentials, or passkey ceremonies performed anywhere but the app's own origin — a constraint the SQ-2 recovery design MUST honor. Delivery outcomes are returned by the boundary (`ARCHITECTURE.md`); persistent failure of reminder-email delivery MUST surface to the user as an in-app notification so a silently-dead email channel — whether from provider trouble or a mailbox attacker filtering messages — does not cause missed insurance/legal deadlines (T-49, and the mail-bomb/reputation-loss abuse of T-30). Channel redundancy beyond in-app plus email remains OQ-4.
  - **Applies to:** Outbound email boundary (external); Background processing; Web client; Mobile client
  - **Verification:** DNS/sender-authentication configuration review at provider selection; template review that no mail solicits secrets; test that persistent reminder-delivery failure raises an in-app notice
  - **References:** `REF-AUTH`, `REF-SUPPLY`, `REF-SECRETS`

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
- **SEC-DEPLOY-4** (**NFR-1.8**, **NFR-1.9**) The application MUST verify against OWASP ASVS (current version) Level 2, and a release MUST NOT ship with open critical- or high-severity security findings. Third-party dependencies with known critical vulnerabilities MUST be patched or mitigated before release, and within 30 days when discovered post-release.
  - **Applies to:** Release process
  - **Verification:** Release checklist gate; dependency-vulnerability scan results
  - **References:** `REF-VULNDEP`, `REF-SSDF`

## Threat Model

Authored 2026-08-13 against the documented system only (no implementation exists). Threats `T-1`–`T-51` were produced by six methodology lenses (STRIDE over identity/session, over the API/data/file core, and over background/email/deployment; LINDDUN privacy analysis; attack-tree analysis of the four highest-value adversary objectives; and document/architecture absorption), consolidated, and adversarially verified against these documents. Each row names a specific documented element, describes the threat as actor + capability + impact, cites only existing controls that genuinely cover it, and states the gap with a proposed change at the level the documents support. Severity and any pending-decision dependency (`SQ-*`/`OQ-*`/`CQ-*`/`DQ-*`) are stated in the final column; mechanism specifics remain `TO BE DECIDED` where a decision is open. Rows whose gap is `NONE` record that the existing controls already close the threat — an accurate control mapping, not a finding.

**Product-specific adversaries** named across the table, beyond the everyday unauthenticated network attacker and the user themselves: the **hostile opposing party** in the insurance/legal dispute (motivated to read, copy, destroy, or tamper with evidence, or to impugn its integrity later); the **household adversary** with physical or unlocked-device access (who may also be a party to the accident); the **attacker controlling the user's email account**; the **malicious or compromised hosting/infrastructure operator**; and **automated abuse / supply-chain** actors. Contact records and journal narrative also hold personal data about **third parties** who are not the account owner (SQ-15, T-49).

| Threat ID | Category | Element (component / boundary / flow) | Threat description (actor · capability · impact) | Existing mitigation | Gap and proposed change · severity · depends on |
|---|---|---|---|---|---|
| T-1 | STRIDE: DoS / Attack-path | Identity & session handling (FR-1.4 lockout) | Anyone who knows the victim's email drives 5 failed ceremonies per 15 min to renew the account-scoped block indefinitely, locking the owner out of evidence at a claim deadline; passkey-only makes the lockout near-worthless for brute force but a reliable renewable DoS. | FR-1.4, SEC-AUTHN-4, SEC-AUTHN-5, SEC-HTTP-6, SEC-LOG-1, SEC-LOG-3 | Lockout is account-scoped only; no source throttling/anti-automation, no cap on consecutive locks, and the docs were silent on whether a valid ceremony bypasses the block. Change: FR-1.4 clarified (a valid passkey ceremony is not blocked); SEC-AUTHN-4 strengthened to require source-based throttling ahead of account lockout; SQ-9 sets values. **High** · SQ-9 |
| T-2 | STRIDE: Spoofing/DoS / Takeover | Identity & session handling — account recovery (FR-1.6) | While SQ-2 is open: (a) an email-anchored recovery flow lets an attacker holding the mailbox seize the whole account and suppress notices in the same inbox; (b) a user who loses every passkey has no flow and permanently loses irreplaceable records — D-9's acknowledged cost. | SEC-AUTHN-6, SEC-EXT-2, SEC-SESSION-5, SEC-AUTHN-5, SEC-AUTHN-8, FR-1.6, FR-1.2, D-9 | Flow undesigned (SQ-2); no rule prompts registering a second passkey while only one exists. Change: resolve SQ-2 satisfying SEC-AUTHN-6, naming the factor beyond mailbox control (specifics TBD); consider a rule prompting a second passkey. **High** · SQ-2 |
| T-3 | STRIDE: EoP/Tampering / LINDDUN: Unawareness / Takeover | Identity & session handling — email change (FR-1.7) + credential lifecycle (FR-1.8) | An attacker with a stolen session or unlocked device changes the account email (only the new address is verified, no notice to the prior one, no session invalidation, no fresh ceremony), then registers their own passkey and revokes the victim's — SEC-AUTHN-8's zero-credential guard is satisfied by the attacker's key and its notice goes to the changed address. Transient compromise becomes durable, silent, full ownership. | SEC-AUTHN-3, SEC-AUTHN-8, SEC-SESSION-5, SEC-LOG-1, SEC-LOG-3 | No notice to the prior address; no step-up ceremony for email/credential changes (inconsistent with SEC-AUTHN-7 for deletion); email change absent from SEC-SESSION-5. Change: **added SEC-AUTHN-10** (step-up), **SEC-AUTHN-11** (prior-address notice), SEC-SESSION-5 += email change. **High** · SQ-2, SQ-3 |
| T-4 | STRIDE: Spoofing / Takeover | Identity & session handling — passkey model (D-9) | An attacker who compromises the user's platform account (Apple/Google) — or a household member via device PIN/platform recovery — presents a genuinely valid synced passkey; the ceremony is indistinguishable from the owner and every downstream guard is satisfied. Sign-in security equals platform-account security, outside the system's control. | SEC-LOG-1, SEC-LOG-3, FR-1.8 | This D-9 residual was recorded nowhere and no control gives a proactive new-sign-in signal. Change: recorded here (T-4) and noted on D-9; **SEC-AUTHN-11** adds an unrecognized-sign-in notice (heuristic TBD, SQ-16). **Medium** · — |
| T-5 | STRIDE: Spoofing | Identity & session handling — registration + email verification (FR-1.1) | A hostile party or automated abuser pre-registers the victim's email bound to the attacker's passkey; if the stressed victim later clicks the verification mail, a verified account under their email exists on the attacker's credential, poisoning future email-anchored recovery. | SEC-AUTHN-3, SEC-AUTHN-5, SEC-EXT-2 | No unverified-account lifecycle; behavior for re-registration of an existing email unspecified; link lifetime TBD. Change: **SEC-EXT-2 strengthened** — unverified accounts expire with their link, re-registration is enumeration-safe and creates no second bindable account, mail states inaction cancels; decide a short lifetime. **Medium** · SQ-2, SEC-EXT-2 lifetime (TBD) |
| T-6 | STRIDE: Information Disclosure | Identity & session handling; Data persistence | A malicious/compromised operator or anyone with DB or backup read access reads stored verification/recovery token values and redeems a live email-change or (eventual SQ-2) recovery link without touching the mailbox — nothing forbade plaintext server-side token storage. | SEC-EXT-2, SEC-DATA-1, SEC-TRUST-2, SEC-INPUT-2 | Single-use/expiry limits but does not close the read-path redemption window. Change: **SEC-EXT-2 strengthened** — emailed token values stored server-side only as one-way hashes; plaintext exists solely in the mail. **Medium** · SQ-2, SQ-5 |
| T-7 | STRIDE: Spoofing/Tampering | Trust boundary 1 — session token mechanics (SQ-3) | If bearer tokens are chosen and stored script-readable, any web-client injection (a surface itself open under CQ-2/SQ-4) exfiltrates the token for replay for up to FR-1.10's 30-day absolute lifetime (idle timeout only SHOULD). If cookies are chosen, CSRF applies — conditionally covered by SEC-SESSION-3's precondition. | SEC-SESSION-1, SEC-SESSION-2, SEC-SESSION-3, SEC-HTTP-1, SEC-HTTP-2, SEC-HTTP-4, SEC-OUT-1 | Web-client storage rules deferred to SQ-3; no token rotation/short-lived scheme, so the long lifetime maximizes a stolen token's value. Change: when resolving SQ-3 add storage constraints and consider rotation in SEC-SESSION-3, without weakening FR-1.10. **Medium** · SQ-3, SQ-4, CQ-2, FR-1.10 (TBD) |
| T-8 | STRIDE/LINDDUN: Information Disclosure | Identity & session handling; Mobile client | A household member or physical-access attacker uses the standing session (valid up to 30 days; idle expiry only SHOULD) on an unlocked device to read/alter health, journal, and legal notes; DESIGN.md defines no content masking or app re-lock, and reminder mail surfaces in OS notifications (SQ-12). | SEC-SESSION-1, SEC-SESSION-2, SEC-SESSION-6, SEC-AUTHN-7, FR-1.10, NFR-5.5, SEC-DATA-6, FR-5.6 | Idle timeout SHOULD with value undecided; no re-auth for sensitive read/write short of deletion; no on-screen-privacy convention. Change: raise idle timeout to MUST when OQ-1 resolves FR-1.10; **DESIGN DQ-6** + convention for masking/re-lock (SQ-14); resolve SQ-12 to minimal content. **Medium** · FR-1.10 (TBD), OQ-1, SQ-12, OQ-4 |
| T-9 | STRIDE: Info Disclosure / LINDDUN: Detectability | Identity & session handling — enumeration resistance (SEC-AUTHN-5) | A hostile party or household adversary probes sign-in/registration/recovery/email-change flows — including deliberately tripping the FR-1.4 lockout, which only a real account can enter — to learn the victim holds an AfterImpact account, itself revealing they are organizing dispute evidence. | SEC-AUTHN-5, SEC-ERR-1, FR-1.3 | Narrow: SEC-AUTHN-5 verification is response-equivalence only; timing side channels and the lockout oracle are not explicitly reconciled. Change: **SEC-AUTHN-5 verification extended** to timing-equivalence and an explicit statement that FR-1.4 throttling presents identically for nonexistent accounts; carried into SQ-2. **Low** · SQ-2 |
| T-10 | STRIDE: Spoofing/Tampering / LINDDUN: Linkability | Identity & session handling — WebAuthn ceremonies (FR-1.2/1.3) | Challenge replay, wrong origin/RP-ID, client-asserted results, session fixation, post-sign-out replay, unauthorized deletion, and cross-party correlation of stored passkey material are all blocked by the cited controls — a control-mapping row, not a gap. | SEC-AUTHN-1, SEC-AUTHN-2, SEC-AUTHN-7, SEC-SESSION-1, SEC-SESSION-3, SEC-SESSION-4, SEC-HTTP-1, SEC-LOG-2, FR-1.2, FR-1.3, FR-9.3, D-9 | **NONE.** **Low** · SQ-3 |
| T-11 | STRIDE: Info Disclosure / LINDDUN: Disclosure / Exfiltration | Background processing — export jobs (FR-9.1/9.2, FR-7.6); export artifacts | An attacker with a stolen session or unlocked device triggers the full-account export (or case PDF, or expense CSV) and downloads the entire health/legal corpus in one authorized operation — no fresh ceremony (contrast SEC-AUTHN-7 for deletion), no email notice (contrast SEC-AUTHN-8), completion announced only in the in-app feed the attacker controls. | SEC-FILE-1, SEC-AUTHZ-1, SEC-AUTHZ-4, SEC-LOG-1, SEC-LOG-3, SEC-SESSION-1, SEC-SESSION-2, FR-1.10, SEC-HTTP-6 | No step-up or out-of-band notice for exports; SEC-LOG-1's "exports" scope across FR-7.6/9.1/9.2 undefined; frequency limits SHOULD (SQ-9). Change: **SEC-AUTHN-10** (step-up before FR-9.2), **SEC-AUTHN-11** (export notice), **SEC-LOG-1 enumerated** to cover all three export forms. **High** · SQ-3, SQ-9 |
| T-12 | STRIDE: Repudiation/Tampering / Audit-gap | REST API application; Background processing — purge jobs (FR-2.6/3.8); security event log | An attacker with a stolen session or unlocked device deletes whole cases or key evidence documents (client confirmations do not bind direct API requests per SEC-TRUST-1); deletions are absent from SEC-LOG-1 and FR-8.1, no notice is sent, undo windows are SHOULD, and after 30 days the worker purges permanently. Asymmetry: account deletion needs a fresh ceremony (SEC-AUTHN-7) but deleting every case needs only a click. | FR-2.6, FR-3.8, NFR-5.5, SEC-TRUST-1, SEC-AUTHN-7, SEC-SESSION-1, SEC-SESSION-2, FR-1.10, SEC-LOG-1, FR-8.4 | Deletion/restore/purge absent from SEC-LOG-1 and FR-8.1; no re-auth or notice for case deletion; undo windows SHOULD not MUST. Change: **SEC-LOG-1 += deletion/restore/purge events**; **SEC-AUTHN-11** notifies on case deletion; **FR-8.1 += deletions**; SQ-16 decides case-deletion step-up; consider raising FR-2.6/3.8 undo to MUST. **High** · SQ-3, OQ-4 |
| T-13 | Exfiltration / STRIDE: Information Disclosure | Data persistence; File store | A malicious/compromised infrastructure operator with runtime privilege reads the DB, file store, process memory, and keys, exfiltrating every user's health/legal data; all processing is server-side so no application-layer barrier sits above the operator. | SEC-DEPLOY-2, SEC-SECRET-2, SEC-DATA-1, SEC-TRUST-2 | No defense against a fully-privileged operator; the actor was absent from the (unauthored) model; key custody TBD (SQ-5) — if co-located with data, at-rest encryption does not bind the operator; IR ownership unassigned (SQ-11). Change: operator threat recorded here; **SQ-5 extended** to weigh key-custody separation from the data plane; assign IR under SQ-11. **High** · SQ-5, SQ-11 |
| T-14 | STRIDE: Tampering/Repudiation / Evidentiary-integrity | Data persistence; REST API application | (a) An attacker with the victim's session silently edits evidence-bearing records (severity, amounts, metadata) or replaces files — outside case edits (FR-8.1), status changes (FR-4.2), and journal/version history (SHOULD), nothing logs or versions the change; journal-entry deletability is entirely unspecified. (b) Even unattacked, records carry no independent integrity evidence, so an opposing party can argue a record was rewritten. | FR-8.4, FR-8.1, FR-4.2, FR-3.7, NFR-6.1, SEC-INPUT-4, SEC-FILE-5, SEC-TRUST-2, SEC-DEPLOY-2, SEC-DATA-1, SEC-LOG-1 | No edit audit or version history for structured records; journal deletion unspecified; no tamper-evidence (content hashes, protected history, trusted timestamps); FR-8.4 retention SHOULD. Change: **new SQ-13** (is tamper-evidence a v1 goal); **OQ-8** (journal deletability); FR-8.1 edit/replacement coverage. **Medium** · SQ-13, OQ-8 |
| T-15 | STRIDE: EoP/Tampering | Document upload/download flow; Web client | A hostile party supplies a document with active content (HTML/SVG/script or a PDF+HTML polyglot) via correspondence the victim uploads; if it executed in the app origin on in-app view (FR-3.5) it would run with the victim's session. SEC-TRUST-3 firmly mandates non-executing delivery, but the mechanism (SEC-FILE-2) and headers (SEC-HTTP-3) are TBD pending the web-client model. | SEC-TRUST-3, SEC-FILE-2, SEC-HTTP-3, SEC-INPUT-3, FR-3.1 | Requirement-level: **NONE** (SEC-TRUST-3 CONFIRMED) — but the enforcing mechanism is unspecified until CQ-2/SQ-4 resolve, so the control cannot yet be verified. Change: resolve CQ-2/SQ-4, then specify SEC-FILE-2 + SEC-HTTP-3 before any upload/view build. **Medium** · CQ-2, SQ-4 |
| T-16 | Attack-path / STRIDE: EoP | File store; Document flow; Background processing (FR-3.9) | A hostile party sends format-valid weaponized files (PDF/DOCX/HEIC/WebP exploiting reader/decoder bugs) as an "accident report"; the victim uploads them, the system stores and re-serves them byte-identical (SEC-FILE-5) and bundles them into exports handed to attorneys/insurers, and in-app view exposes client decoders. The sole content control (FR-3.9 scanning) is SHOULD with engine/v1 undecided. | FR-3.9, SEC-FILE-4, SEC-INPUT-3, FR-3.1, SEC-TRUST-3, SEC-FILE-2 | If SQ-10 defers scanning, no control or recorded risk-acceptance covers stored-malware redistribution; no rule hardens the in-app decoder path. Change: resolve SQ-10 weighing the redistribution path (record accepted risk if deferred); **SEC-FILE-6** requires the SEC-FILE-2 decision to address decoder exploitation. **Medium** · SQ-10, CQ-2, SQ-4 |
| T-17 | STRIDE: EoP / Attack-path | Background processing | The worker is the most privileged data actor (reads all case data, deletes across all stores) yet parses attacker-influenced bytes: scanning (FR-3.9), any FR-3.4 OCR (memory-corruption-prone extraction libs), and export embedding. A parser exploit executes inside boundary 2 with read-all/delete-all reach; if the execution model is in-process (ASSUMPTION), the blast radius includes the API and session handling. | DEP-3, DEP-4, DEP-5, DEP-6, SEC-DEPLOY-2, SEC-DEPLOY-4, SEC-SECRET-2, SEC-INPUT-3, SEC-OUT-2 | No rule isolates server-side hostile-content parsing or scopes its store access; the execution-model decision is not informed by this risk. Change: **added SEC-FILE-6** (isolated parsing, least store access); ARCHITECTURE notes parsing as a deciding factor for the execution model. **Medium** · SQ-10, SQ-5, FR-3.4 OCR (TBD), execution model (TBD) |
| T-18 | DOC-GAP / STRIDE: Info Disclosure / LINDDUN: Disclosure | Export artifacts (FR-9.1/9.2) | Full-account archives and case PDFs persist in the file store with no documented expiry — maximum-sensitivity aggregates enlarging every later compromise; a stale archive silently retains records the user deleted under FR-2.6/3.8, and FR-9.2's scope over prior versions/trash and its quota treatment are unspecified. | SEC-DATA-1, SEC-FILE-1, SEC-AUTHZ-4, SEC-DATA-4, FR-9.3 | No artifact retention/expiry rule outside account deletion; no reconciliation vs deletion; export scope/quota undocumented. Change: **new FR-9.4** (artifact lifetime, export scope declared to the user, quota treatment); **OQ-7 extended** to name artifacts. **Medium** · OQ-7 |
| T-19 | DOC-GAP / STRIDE: Info Disclosure | Data persistence; Trust boundary 2 | Search runs in-DB by ASSUMPTION, to be revisited against NFR-4.3. A separate search index (or FR-3.4 content indexing) would replicate sensitive titles/notes/tags/extracted text into a new store that SEC-TRUST-2 (which names only "RDBMS and file store") and SEC-DATA-1's enumeration do not clearly bind — inviting a store outside the boundary-2 regime. | SEC-TRUST-2, SEC-DATA-1, SEC-DATA-2, SEC-DATA-4, FR-9.3 | Boundary-2/at-rest rules enumerate named stores rather than any personal-data store; nothing forces a SECURITY update when the search ASSUMPTION lands. Change: **SEC-TRUST-2 + SEC-DATA-1 generalized** to bind any store holding personal data. **Low** · DB-search ASSUMPTION, FR-3.4 (TBD) |
| T-20 | STRIDE: Information Disclosure | Trust boundary 2 | An unauthenticated attacker or compromised co-tenant attempts a direct DB/file-store connection, bypassing API authorization — covered by SEC-TRUST-2's private network, scoped credentials, least privilege, and at-rest encryption. Residual: if SQ-5 custodies keys with the same operator as the data, encryption does not bind privileged runtime access. | SEC-TRUST-2, SEC-SECRET-1, SEC-SECRET-2, SEC-DEPLOY-1, SEC-DEPLOY-2, SEC-DATA-1 | Key management undecided; no key-custody-separation requirement. Change: **SQ-5 extended** to specify key custody separated from the data plane where feasible. **Low** · SQ-5 |
| T-21 | STRIDE: Denial of Service | REST API application; File store; Background processing | (a) An unauthenticated attacker floods the public API (no edge protection documented). (b) Automated abuse registers many verified accounts (registration is unthrottled) and fills each 10 GB quota or repeatedly triggers exports, monopolizing the assumed in-process worker. (c) A session-holding attacker fills the victim's quota with junk the 30-day trash window prevents reclaiming, blocking new evidence uploads before deadlines. | FR-3.10, FR-3.1, FR-1.4, SEC-AUTHN-3, SEC-HTTP-6, SEC-INPUT-3 | SEC-HTTP-6 SHOULD with values TBD and no registration/anti-automation clause; no edge/volumetric protection; FR-3.10 quota accounting during the trash window undefined. Change: **SQ-9 extended** (registration-rate/anti-automation, export caps); edge protection with SQ-5; **FR-3.10 quota accounting** specified. **Medium** · SQ-9, SQ-5, OQ-5, in-process worker ASSUMPTION |
| T-22 | STRIDE: EoP/Tampering/Info Disclosure | REST API application | A hostile party with their own account submits the victim's record/file identifiers across every entity (IDOR) or posts extra JSON fields hoping they bind (mass assignment) — comprehensively covered by server-side ownership, deny-by-default, not-found-equivalent responses, no-unguessable-ID reliance, a single enforcement layer, and explicit request models. | FR-1.9, SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-AUTHZ-3, SEC-AUTHZ-4, SEC-AUTHZ-5, SEC-FILE-1, SEC-INPUT-4, SEC-TRUST-1, NFR-6.1, D-2 | **NONE.** **Low** · — |
| T-23 | STRIDE: Tampering | REST API application | An attacker supplies SQL metacharacters via write fields, document search (FR-3.4), or dynamic sort/filter (FR-3.3) to read/modify rows — covered by SEC-INPUT-2's parameterized statements + allowlisted identifiers, SEC-ERR-1, and SEC-TRUST-1. | SEC-INPUT-1, SEC-INPUT-2, SEC-TRUST-1, SEC-ERR-1 | **NONE.** **Low** · — |
| T-24 | STRIDE: Tampering | Document flow; File store | An attacker submits type-mismatched files, oversize bodies, or hostile filenames (path traversal) — covered by server-side content inspection, pre-buffering size limits, quotas, and server-generated storage names with the filename kept only as sanitized display metadata. | FR-3.1, FR-3.10, SEC-INPUT-3, SEC-FILE-3, SEC-FILE-1, SEC-HTTP-2, SEC-TRUST-3, SEC-OUT-1 | **NONE.** **Low** · SEC-INPUT-3 mechanism (TBD), FR-3.1 defaults (TBD) |
| T-25 | STRIDE: Tampering/Info Disclosure | In-app notification feed; export artifacts (FR-7.6/9.1/9.2) | Hostile-party text recorded by the victim (contact names, reference numbers, notes with =/+/-/@ or markup) targets stored XSS in the feed/clients, spreadsheet-formula execution in CSV exports, and markup interpretation in PDF generation — covered by text-only encoded rendering, CSV formula neutralization, inert PDF pipelines, safe-scheme links, and exact decimal arithmetic. | SEC-OUT-1, SEC-OUT-2, SEC-OUT-3, SEC-INPUT-1, NFR-6.2 | **NONE.** **Low** · — |
| T-26 | STRIDE: Repudiation / LINDDUN: Non-compliance | Security event log (SEC-LOG-1) | SQ-7's recorded conflict: purge the log with FR-9.3 deletion and the record of sign-ins/lockouts/exports vanishes (destroying abuse investigation and the user's own evidence in an adversarial dispute); retain it and a deleted user's 12-month IP/activity history outlives the erasure promise as a breach and subpoena surface, possibly violating erasure rights. | SEC-LOG-1, SEC-LOG-2, SEC-DATA-4, SEC-DATA-1, FR-9.3 | SQ-7 unresolved: no decided retention basis, surviving form, or SEC-DATA-7 disclosure. Change: resolve SQ-7 with OQ-1/SQ-1; amend SEC-LOG-1 to state what survives deletion, minimized and on what basis, and require SEC-DATA-7 to disclose it. **High** · SQ-7, OQ-1, SQ-1 |
| T-27 | LINDDUN: Unawareness / Notification | In-app notification feed | An attacker with a stolen session or unlocked device operates undetected for the session's life: sign-ins, full exports, and deletions surface at most in the in-app feed the attacker controls; out-of-band mail is required only for credential changes (SEC-AUTHN-8), the activity view is SHOULD and pull-only, and SMS/push are out of scope. The victim cannot learn their evidence is being read or destroyed in time to use the undo windows. | SEC-AUTHN-8, SEC-LOG-1, SEC-LOG-3, FR-2.6, FR-3.8, FR-9.2, SEC-SESSION-2 | No proactive out-of-band notice for sign-ins/exports/destructive acts; SEC-LOG-3 SHOULD. Change: **SEC-LOG-3 raised to MUST**; **SEC-AUTHN-11** requires proactive notice for exports, deletions, and (heuristic TBD, SQ-16) unrecognized sign-ins. **High** · OQ-4, SQ-3 |
| T-28 | STRIDE: Spoofing | Outbound email boundary; Trust boundary 3 | A hostile party or commodity phisher spoofs AfterImpact-branded mail to an injured/medicated victim with fake "verify"/"export ready"/"recover" lures; sender authentication is TBD (SEC-EXT-3), so nothing prevents exact-domain spoofing, and if SQ-2 recovery involves any email step a spoofed lure becomes the practical takeover path. | SEC-AUTHN-1, SEC-AUTHN-9, SEC-EXT-2, SEC-EXT-3, SEC-AUTHN-6, FR-1.2, D-9 | Sender auth undecided; no rule constrains what legitimate mail may ask the user to do. Change: **added SEC-EXT-4** (enforce sender authentication; legitimate mail must not solicit secrets/ceremonies), binding the SQ-2 design. **Medium** · SEC-EXT-3 sender auth (TBD), SQ-2 |
| T-29 | STRIDE/LINDDUN: Disclosure/Linkability | Trust boundary 3; Outbound email boundary | A compromised/malicious email provider, a network interceptor (transport TBD), or a mailbox-controlling household adversary reads everything crossing boundary 3; if SQ-12 admits appointment purpose or provider names, health conditions leak, and even minimal content lets message timing link the address to accident-case management. Mailbox copies outlive FR-9.3 deletion. | SEC-DATA-6, SEC-DATA-3, SEC-EXT-1, SEC-EXT-2, SEC-EXT-3, SEC-AUTHN-6, SEC-AUTHN-9, SEC-SESSION-5, FR-5.6, D-7, D-9 | SQ-12 unresolved; SEC-EXT-3 transport/provider undecided; no requirement the provider be bound by processing terms consistent with SEC-DATA-3/D-5. Change: **SQ-12 resolved toward a minimal-content default in SEC-DATA-6**; **SEC-EXT-4** requires transport protection + processing terms at provider selection. **Medium** · SQ-12, SEC-EXT-3 (TBD), SQ-2, OQ-1 |
| T-30 | STRIDE: Denial of Service | Outbound email boundary; Background processing (FR-5.6) | Automated abuse or a scripted hijacked session drives registration/email-change against arbitrary addresses or creates unbounded reminders, mail-bombing targets and burning sender reputation until the domain is throttled; independently, a provider outage removes the only external channel (verification, recovery, notices, deadline reminders) with no fallback. | SEC-HTTP-6, FR-5.6, NFR-3.1 | Email caps SHOULD with values TBD; no per-recipient caps; no documented failure handling/monitoring for the email dependency; sole-channel risk not recorded against SQ-2. Change: **SQ-9** per-address/per-source caps (email portion of SEC-HTTP-6 to MUST); **SEC-EXT-4** + ARCHITECTURE document delivery-failure handling; SQ-2 must state behavior when email is down. **Medium** · SQ-9, SQ-2, OQ-4, SEC-EXT-3 provider (TBD) |
| T-31 | STRIDE: Info Disclosure / LINDDUN: Non-compliance | Backups | A malicious/compromised operator or an attacker reaching backup storage obtains full-dataset backup artifacts; SEC-DATA-1 encrypts them (defeating media theft if SQ-5 keeps keys separate), but SEC-TRUST-2 names only the live stores, and NFR-3.2 quarterly restore tests materialize full personal-data copies with no documented access/handling/destruction rule. | SEC-DATA-1, NFR-3.2, FR-9.3, SEC-DATA-4, SEC-TRUST-2, SEC-DEPLOY-2, SEC-SECRET-1 | No rule restricts backup-artifact access or governs restore-test copies. Change: **SEC-TRUST-2 generalized** to cover backup artifacts; **SEC-DATA-8** requires restore-test copies to meet production controls and be destroyed after verification. **Medium** · SQ-5 |
| T-32 | STRIDE: Tampering / LINDDUN: Non-compliance | Backups; Background processing — purge jobs | A restore (RTO ≤ 12 h) from a backup up to 90 days old reintroduces data the user permanently deleted and resurrects passkeys they revoked and sessions they invalidated — silently re-arming a credential revoked precisely because it was compromised. No component owns replaying deletions/revocations/invalidations after a restore. | FR-9.3, NFR-3.2, SEC-DATA-4, SEC-SESSION-2 | No owner/procedure for post-restore reconciliation; the restore path is absent from every traceability table. Change: **added SEC-DATA-8** (restore MUST re-apply deletions/revocations and invalidate sessions); **ARCHITECTURE assigns Background processing** and adds CQ-3. **Medium** · — |
| T-33 | LINDDUN: Non-compliance / DOC-GAP | Backups; Time-driven work (flow 3) | FR-9.3's 90-day backup bound applies only to whole-account deletion; deleted cases/documents (FR-2.6/3.8) have no backup-persistence bound and rotation is unspecified — an operator or discovery adversary can recover records the user destroyed, indefinitely. Compounding: FR-2.6/3.8 say "removed"/"permanent" while data is recoverable for 30 days — an unrecorded contradiction that misleads user and implementer. | FR-9.3, FR-2.6, FR-3.8, NFR-5.5, SEC-DATA-1, SEC-DATA-4 | No backup-retention bound for record-level deletions; removed-vs-recoverable tension unrecorded; confirmations don't disclose the undo window. Change: **SEC-DATA-8** bounds backup persistence of purged records; **REQUIREMENTS**: confirmations state the undo window; **ARCHITECTURE CQ-3** records the contradiction. **Medium** · SQ-5, OQ-1, OQ-7 |
| T-34 | STRIDE: Repudiation/Tampering | Security event log | An attacker who compromises the REST API (which writes the log) or a privileged operator edits/deletes log entries to erase the trace of an exfiltration or takeover; in the dispute, a log whose integrity cannot be shown weakens the victim's ability to prove compromise. Nothing requires the log to be append-only or access-separated from the components it monitors. | SEC-LOG-1, SEC-DEPLOY-2 | No integrity/tamper-evidence/write-separation requirement; the event-generating component can rewrite its own trail. Change: **added SEC-LOG-4** (tamper-evident, append-only, write access separated from the API path; mechanism TBD with SQ-5). **Medium** · SQ-5 |
| T-35 | Attack-path / STRIDE: Tampering (supply chain) | REST API; Future CI/CD; Terraform state | A supply-chain attacker compromises a Quarkus/Kotlin dependency, the (UNKNOWN) CI/CD pipeline, or the Terraform state backend — each yielding code execution or store credentials behind boundary 2 and thus full reach, since the API is the single enforcement point. The documented DEP-*/SEC-DEPLOY-*/SEC-SECRET-* posture covers this prospectively. | DEP-1..DEP-8, SEC-DEPLOY-1, SEC-DEPLOY-2, SEC-DEPLOY-3, SEC-DEPLOY-4, SEC-SECRET-1, SEC-SECRET-2, SEC-SECRET-3, SEC-EXT-1, SEC-TRUST-2, SEC-DATA-1, SEC-LOG-1 | **NONE** — covered prospectively, though none is actionable until SQ-6 selects a pipeline and SQ-5 a platform. **Medium** · SQ-5, SQ-6 |
| T-36 | DOC-GAP / STRIDE: Spoofing | Mobile client | Target OSes and distribution model are UNKNOWN; nothing establishes how the victim obtains an authentic client, so a supply-chain attacker or hostile party could induce installing a tampered build (sideload, fake listing) that harvests the session and displayed records. Server-side origin/RP-ID validation limits ceremony abuse, but no SQ tracks client signing/update integrity. | SEC-DEPLOY-3, SEC-AUTHN-1 | No open question tracks distribution/signing/update integrity for the mobile client. Change: **new SQ-14** requires the distribution decision to specify signing, store channel, and update integrity. **Medium** · target-OS/distribution UNKNOWN, SQ-6 |
| T-37 | DOC-GAP / LINDDUN: Disclosure | Mobile client | The mobile client is online-only by ASSUMPTION and owns nothing durable; SEC-SESSION-6 protects only session credentials. If offline caching is introduced (plausible for appointments with poor connectivity), health records would rest on-device with no at-rest control, readable by a household member or thief; no SQ tracks the suppressed question. | SEC-SESSION-6, SEC-DATA-2 | No rule governs client-side persistence of personal data if online-only is revisited. Change: **SQ-14** adds a client data-at-rest rule tied to the assumption's revisit. **Low** · online-only ASSUMPTION |
| T-38 | DOC-GAP | REST API application | API versioning is TBD with no security consideration; when introduced, an unmaintained old version could become an authorization or patching bypass reachable by any network attacker unless every live version routes through the SEC-AUTHZ-5 layer and the SEC-DEPLOY-4 gate. | SEC-AUTHZ-5, SEC-DEPLOY-4 | The versioning decision carries no recorded requirement that all live versions stay behind the enforcement layer and patch gate. Change: **ARCHITECTURE note** on the versioning open decision. **Informational** · versioning scheme (TBD) |
| T-39 | LINDDUN: Non-compliance | Outbound email boundary; Trust boundary 3 | If a personal-data breach occurs, no one detects, triages, or notifies: SEC-DATA-5 requires notice and ARCHITECTURE assigns delivery to email, but IR ownership is UNKNOWN (SQ-11) and jurisdictions/deadlines are TBD (OQ-1/SQ-1). The victim — whose whole record set is sensitive (D-5) — could remain unnotified, compounding harm and regulatory exposure. | SEC-DATA-5, SEC-LOG-1 | SQ-11: no IR owner/process; deadlines unresolvable until OQ-1/SQ-1. Change: resolve SQ-11 by assigning IR ownership + process under SEC-DATA-5; set timelines once OQ-1/SQ-1 decide. **High** · SQ-11, OQ-1, SQ-1 |
| T-40 | LINDDUN: Non-compliance | Data persistence | The system retains an inactive/abandoned account's full sensitive record set indefinitely (OQ-7): over years this maximizes breach blast radius for a user who may have forgotten the account (or died) and likely violates storage-limitation principles under regimes OQ-1/SQ-1 may select. | SEC-DATA-1, SEC-DATA-2, SEC-DATA-4 | OQ-7 unresolved: no retention period or explicit indefinite-retention decision. Change: **OQ-7 resolved** with either a retention period (advance email warning before purge) or an explicit indefinite-retention decision, aligned with OQ-1. **Medium** · OQ-7, OQ-1, SQ-1 |
| T-41 | LINDDUN: Unawareness/Non-compliance | Web client; Mobile client | The user accepts processing they cannot fully evaluate: SEC-DATA-7 requires a viewable pre-registration policy with recorded acceptance, but its substance is unresolvable while regimes are undecided, no re-acceptance on change is required, and it need not disclose SEC-LOG-1's 12-month IP retention (and its unresolved survival of deletion, SQ-7). | SEC-DATA-7 | No policy-change re-notification/re-acceptance; no requirement to disclose security-log retention; content blocked on OQ-1/SQ-1. Change: **SEC-DATA-7 extended** to require change re-notification (re-acceptance where material) and disclosure of SEC-LOG-1 retention once SQ-7 resolves. **Low** · OQ-1, SQ-1, SQ-7 |
| T-42 | LINDDUN: Non-repudiation/Unawareness | Export artifacts; Data persistence | The user cannot repudiate what their records retain: FR-8.4's permanent journal timestamps (deliberate evidentiary protection) let an opposing party receiving a case PDF impugn recollection via late authoring dates, and FR-6.3's name-snapshot means a deleted contact's name flows into exports — with no disclosure at authoring/deletion time and export inclusion of this metadata unspecified. Informed-choice failures, not control failures; timestamps must not be weakened. | FR-8.4, NFR-5.5 | Export inclusion of journal authorship metadata unspecified; no user-facing disclosure at journal authoring or contact deletion. Change: **REQUIREMENTS**: FR-9.1/9.2 explicit about authorship metadata (FR-8.4 unchanged); FR-6.3 deletion confirmation states the name persists; clients disclose permanence at authoring. **Low** · — |
| T-43 | LINDDUN: Non-compliance / DOC-GAP | In-app notification feed | The feed is an input to both clients and served by the API, yet no Data Entity models feed items, so no retention/purge/export/minimization rule attaches; reminder notices carry health-adjacent text that may outlive its deleted source record or case, escaping the deletion guarantees, and SQ-12's content question has no in-app analogue. | SEC-DATA-2, SEC-DATA-4 | Feed items unmodeled; lifecycle after source/case/account deletion and export inclusion unspecified. Change: **REQUIREMENTS adds a Notification entity** tying purge to FR-2.6/9.3 and scope to FR-9.2. **Low** · SQ-12 |
| T-44 | LINDDUN: Disclosure | Web client; Mobile client | No telemetry/analytics is decided; if any were introduced (developer convenience or a transitive dependency), screen-level usage would disclose health-adjacent behavior to a third party. As documented this is fully constrained (functional-need minimization, no third-party trackers on authenticated screens, external-call scope-change review, dependency rules). Mobile OS platform telemetry remains outside these controls. | SEC-DATA-2, SEC-DATA-3, SEC-EXT-1, D-7, DEP-1, DEP-2 | **NONE.** **Informational** · target-OS UNKNOWN |
| T-45 | DOC-GAP | SECURITY.md Security Profile — threat model section | The threat-model section was empty by its own admission, so the system-specific attack chains (lockout weaponization, the email-change-then-credential-swap takeover, platform-account passkey sync, the hostile-party/household-adversary actors) had no documented home and could not be traced to acceptance or mitigation. | — | Section was empty; product-specific actor set undocumented. Change: **this Threat Model section authored** (T-1–T-51), naming the adversaries and cross-referencing each threat to its controls and open questions. **Informational** · resolved |
| T-46 | STRIDE: DoS / Deletion-in-progress | Identity & session handling — account deletion (FR-9.3); Background processing | A household adversary who can complete a platform-authenticator ceremony on the unlocked device — or the injured/medicated user themselves — initiates FR-9.3 deletion, irreversibly destroying the whole record. Unlike FR-2.6/3.8 (30-day undo), FR-9.3 documents no cancellation window and no email notice, so the victim may learn only when sign-in fails, purge already running. | SEC-AUTHN-7, NFR-5.5, FR-9.3, SEC-LOG-1 | No email notice on deletion initiation (SEC-AUTHN-8 covers only credentials); no user-facing cancellation/grace window before purge. Change: **FR-9.3 gains a cancellation window** (value TBD, OQ-1/SQ-1); **SEC-AUTHN-11** notifies on deletion initiation. **Medium** · OQ-1, SQ-1 |
| T-47 | STRIDE: Tampering/Repudiation / Evidentiary-integrity | Document flow; File store — document replacement (FR-3.7) | An attacker with a stolen session or unlocked device uses FR-3.7 to replace an evidence file (police report, medical record) with a doctored version; FR-8.1's timeline list covers "added" but not replaced/edited, so the swap leaves no timeline trace, and FR-3.7 prior-version retention is SHOULD, so the authentic original may not survive — the doctored file then flows into exports as authentic. | FR-3.7, FR-8.4, NFR-6.1, SEC-FILE-5 | Replacement and metadata edits absent from FR-8.1; prior-version retention SHOULD not MUST; SEC-FILE-5 forbids only system-initiated transforms. Change: **FR-8.1 += file replacement and metadata edits**; SQ-13 decides raising FR-3.7 retention to MUST. **Medium** · SQ-13 |
| T-48 | STRIDE: Spoofing / Restore-from-backup | Backups; Identity & session handling | A restore (RPO ≤ 24 h) rolls back the DB holding passkey keys, session records, and tokens, resurrecting revoked credentials and invalidated sessions, resetting FR-1.4 lockout, and losing log entries written after the backup point — so a household adversary regains access after the victim believed the compromise was remediated, and the record of what they did is erased. Complements T-32 (this is credential/session/log state). | NFR-3.2, SEC-SESSION-5, SEC-AUTHN-8, SEC-LOG-1 | No post-restore reconciliation of credential/session state or user notice of the restore. Change: **SEC-DATA-8** — a restore MUST invalidate all sessions, re-apply post-backup revocations where recoverable, and SHOULD notify the user; log-continuity handling noted. **Low** · SQ-5 |
| T-49 | STRIDE: DoS / LINDDUN: Unawareness | Outbound email boundary; Background processing (FR-5.6/5.7) | An attacker controlling the victim's mailbox silently filters/deletes reminder and hard-deadline emails, sabotaging the insurance/legal calendar so filings lapse; the same arises non-adversarially from bounces/blocking. ARCHITECTURE says delivery outcomes return to the caller, but no rule acts on failure and OQ-4 defers other channels. | FR-5.6, FR-5.7, FR-5.5, FR-2.4 | No handling of email delivery failure despite outcomes being available. Change: **SEC-EXT-4** — persistent reminder-mail failure MUST surface as an in-app notification; channel redundancy stays OQ-4. **Low** · OQ-4 |
| T-50 | LINDDUN: Non-compliance/Identifiability | Data persistence — Contact entity; export artifacts | The user, doing exactly what the product intends, stores personal data of non-consenting third parties (other driver, witnesses, employer, adjusters) and narrates them in the journal, then hands it to insurers/attorneys via exports; under the OQ-1/SQ-1 regime these third parties may be data subjects with notification/erasure rights the system never considers — every rule treats the owner as the sole subject. | SEC-DATA-2, SEC-DATA-3, D-5 | No document acknowledges third-party data subjects; no OQ/SQ tracks it. Change: **Security Profile records third-party data**; **new SQ-15** tracks whether the regime creates obligations toward non-user subjects. **Low** · OQ-1, SQ-1 |
| T-51 | LINDDUN: Disclosure / Platform-residue | Mobile client | A household/physical-access attacker — or one reaching the device-level cloud backup — reads health data through OS residue the app never chose to persist: task-switcher screenshots of health/journal screens, OS-backup inclusion of app storage, and clipboard/keyboard caches learning typed health terms. SEC-SESSION-6 covers only session credentials. | SEC-SESSION-6 | No rule governs OS-level residue of rendered/typed health data (snapshot handling, OS-backup exclusion, sensitive-field input hints). Change: **SQ-14** adds a PRM-KMP-derived mobile residue rule; mechanisms TBD with the target-OS decision. **Low** · target-OS/distribution UNKNOWN |

**Cross-methodology confirmations.** The silent full-account export path (T-11) was independently surfaced by five of the six lenses; the case/document evidence-destruction path (T-12) by four; the FR-1.4 lockout DoS (T-1) and the SQ-7 log-vs-deletion conflict (T-26) by three each. These convergences are recorded as high-confidence findings. Several rows deliberately restate an already-recorded open question as its threat statement (SQ-2 → T-2, SQ-7 → T-26, SQ-9 → T-1/T-21, SQ-10 → T-16, SQ-11 → T-39, SQ-12 → T-29, OQ-7 → T-40, CQ-2/SQ-4 → T-15), citing the existing ID rather than raising a duplicate.

## Traceability

Component names are exactly those in `ARCHITECTURE.md`. Trust boundaries 1–3 are its **Trust boundaries**. The Requirements column lists what each rule enforces; an ID in **bold** means the rule *is* that requirement, stated here and nowhere else. Status: `CONFIRMED` = backed by an explicit documented requirement; `PROVISIONAL` = safe default not explicitly required; `PARTIALLY DEFINED` = requirement exists but a material decision is open; `TO BE DECIDED` = wholly dependent on an open decision.

| Rule | Requirements | Boundary / component | Status |
| --- | --- | --- | --- |
| SEC-TRUST-1 | FR-1.9; all FR validation rules | Trust boundary 1; REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-TRUST-2 | **NFR-1.4** (supporting) | Trust boundary 2; Data persistence (relational, 3NF); File store (uploaded originals) | CONFIRMED |
| SEC-TRUST-3 | **NFR-1.5** | REST API application (Quarkus/Kotlin); File store (uploaded originals) | CONFIRMED |
| SEC-AUTHN-1 | FR-1.2, FR-1.3, D-9 | Identity & session handling | CONFIRMED |
| SEC-AUTHN-2 | NFR-2.3 (supporting) | Identity & session handling | PROVISIONAL |
| SEC-AUTHN-3 | FR-1.1, FR-1.7 | Identity & session handling; Outbound email boundary (external) | CONFIRMED |
| SEC-AUTHN-4 | FR-1.4, NFR-1.6 | Identity & session handling | CONFIRMED |
| SEC-AUTHN-5 | FR-1.3 | Identity & session handling | CONFIRMED |
| SEC-AUTHN-6 | FR-1.6 | Identity & session handling | TO BE DECIDED |
| SEC-AUTHN-7 | FR-9.3 | Identity & session handling | CONFIRMED |
| SEC-AUTHN-8 | FR-1.8 | Identity & session handling; Outbound email boundary (external) | CONFIRMED |
| SEC-AUTHN-9 | **NFR-1.3**, D-9 | Identity & session handling | CONFIRMED |
| SEC-AUTHN-10 | FR-1.7, FR-1.8, FR-9.2 (T-3, T-11) | Identity & session handling; REST API application (Quarkus/Kotlin); Background processing | PROVISIONAL |
| SEC-AUTHN-11 | FR-1.7, FR-9.2, FR-9.3, FR-2.6 (T-3, T-4, T-11, T-27, T-46) | Identity & session handling; Background processing; Outbound email boundary (external) | PROVISIONAL |
| SEC-SESSION-1 | FR-1.5 | Identity & session handling | CONFIRMED |
| SEC-SESSION-2 | FR-1.10 | Identity & session handling | PARTIALLY DEFINED |
| SEC-SESSION-3 | FR-1.10, NFR-1.7 | Trust boundary 1; Identity & session handling | TO BE DECIDED |
| SEC-SESSION-4 | FR-1.9 (supporting) | Identity & session handling | PROVISIONAL |
| SEC-SESSION-5 | FR-1.6, FR-1.7, FR-1.8 | Identity & session handling | CONFIRMED |
| SEC-SESSION-6 | — (`PRM-KMP` safe default) | Mobile client (Compose MP) | PROVISIONAL |
| SEC-AUTHZ-1 | FR-1.9 | REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-AUTHZ-2 | FR-1.9 | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-AUTHZ-3 | FR-1.9 | REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-AUTHZ-4 | FR-1.9, NFR-1.4 | REST API application (Quarkus/Kotlin); File store (uploaded originals) | CONFIRMED |
| SEC-AUTHZ-5 | FR-1.9 (supporting) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-HTTP-1 | **NFR-1.1** | Trust boundary 1 | CONFIRMED |
| SEC-HTTP-2 | — (REST style, ARCHITECTURE.md) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-HTTP-3 | NFR-1.5 (supporting) | Web client (Compose MP); REST API application (Quarkus/Kotlin) | TO BE DECIDED |
| SEC-HTTP-4 | — | REST API application (Quarkus/Kotlin) | TO BE DECIDED |
| SEC-HTTP-5 | D-5, NFR-2.3 (supporting) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-HTTP-6 | FR-5.6, FR-9.2 (abuse surface) | REST API application (Quarkus/Kotlin); Background processing | TO BE DECIDED |
| SEC-INPUT-1 | FR-2.1, FR-3.1, FR-3.10, FR-4.1, FR-4.3, FR-5.2, FR-6.1, FR-7.1 | REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-INPUT-2 | NFR-1.8 (supporting) | REST API application (Quarkus/Kotlin); Background processing; Data persistence (relational, 3NF) | PROVISIONAL |
| SEC-INPUT-3 | FR-3.1 | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-INPUT-4 | FR-1.9, NFR-6.1 (server-controlled fields) | REST API application (Quarkus/Kotlin) | PROVISIONAL |
| SEC-FILE-1 | **NFR-1.4**, FR-1.9, FR-9.1, FR-9.2 | File store (uploaded originals); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-FILE-2 | NFR-1.5, FR-3.5 | REST API application (Quarkus/Kotlin); Web client (Compose MP) | PARTIALLY DEFINED |
| SEC-FILE-3 | FR-3.2 (filename as metadata) | File store (uploaded originals) | PROVISIONAL |
| SEC-FILE-4 | FR-3.9 | Background processing | PARTIALLY DEFINED |
| SEC-FILE-5 | FR-3.5 | File store (uploaded originals); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-FILE-6 | FR-3.9, FR-3.4, FR-9.1, FR-9.2 (T-16, T-17) | Background processing; File store (uploaded originals); Document upload/download flow | PARTIALLY DEFINED |
| SEC-OUT-1 | NFR-1.5 (client-side complement) | Web client (Compose MP); Mobile client (Compose MP) | PROVISIONAL |
| SEC-OUT-2 | FR-7.6, FR-9.1, FR-9.2 | REST API application (Quarkus/Kotlin); Background processing | PROVISIONAL |
| SEC-OUT-3 | FR-6.1 (contact fields, conditional) | Web client (Compose MP); Mobile client (Compose MP) | PROVISIONAL |
| SEC-DATA-1 | **NFR-1.2** | Data persistence (relational, 3NF); File store (uploaded originals) | CONFIRMED |
| SEC-DATA-2 | **NFR-2.3**, D-5 | All components | CONFIRMED |
| SEC-DATA-3 | **NFR-2.1** | Web client (Compose MP); Mobile client (Compose MP); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-DATA-4 | **NFR-2.4**, FR-9.2, FR-9.3 | Background processing; Data persistence (relational, 3NF); File store (uploaded originals) | CONFIRMED |
| SEC-DATA-5 | **NFR-2.5** | UNKNOWN (no incident-response owner documented) | PARTIALLY DEFINED |
| SEC-DATA-6 | NFR-1.7 (extension); trust boundary 3 | Outbound email boundary (external) | PROVISIONAL |
| SEC-DATA-7 | **NFR-2.2** | Web client (Compose MP); Mobile client (Compose MP); REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-DATA-8 | FR-9.3, FR-2.6, FR-3.8, FR-1.8, NFR-3.2 (T-31, T-32, T-33, T-48) | Data persistence (relational, 3NF); File store (uploaded originals); Background processing | PARTIALLY DEFINED |
| SEC-SECRET-1 | NFR-1.7 (extension) | All server components; Web client (Compose MP); Mobile client (Compose MP) | PROVISIONAL |
| SEC-SECRET-2 | NFR-1.4 (supporting) | Trust boundaries 2 and 3 | PROVISIONAL |
| SEC-SECRET-3 | — (Terraform, ARCHITECTURE.md) | Deployment (Terraform model) | PROVISIONAL |
| SEC-LOG-1 | **NFR-1.6** | Identity & session handling; REST API application (Quarkus/Kotlin) | CONFIRMED |
| SEC-LOG-2 | **NFR-1.7** | All server components | CONFIRMED |
| SEC-LOG-3 | NFR-1.6 (T-27) | REST API application (Quarkus/Kotlin); Web client (Compose MP); Mobile client (Compose MP) | CONFIRMED |
| SEC-LOG-4 | NFR-1.6 (integrity extension; T-34) | Identity & session handling; REST API application (Quarkus/Kotlin); Data persistence (relational, 3NF) | PARTIALLY DEFINED |
| SEC-ERR-1 | NFR-5.3, FR-1.3, FR-1.9 (disclosure) | REST API application (Quarkus/Kotlin); Web client (Compose MP); Mobile client (Compose MP) | CONFIRMED |
| SEC-EXT-1 | D-7 | Outbound email boundary (external) | CONFIRMED |
| SEC-EXT-2 | FR-1.1, FR-1.6 | Identity & session handling; Outbound email boundary (external) | PARTIALLY DEFINED |
| SEC-EXT-3 | NFR-1.1 (transport) | Outbound email boundary (external) | TO BE DECIDED |
| SEC-EXT-4 | FR-1.1, FR-1.6, FR-5.6 (T-28, T-30, T-49) | Outbound email boundary (external); Background processing; Web client (Compose MP); Mobile client (Compose MP) | PARTIALLY DEFINED |
| SEC-DEPLOY-1 | — (Terraform, ARCHITECTURE.md) | Deployment (Terraform model) | PROVISIONAL |
| SEC-DEPLOY-2 | NFR-1.4 (supporting) | Trust boundary 2; Deployment (Terraform model) | PARTIALLY DEFINED |
| SEC-DEPLOY-3 | **NFR-1.8**, **NFR-1.9** (gating) | Future CI/CD (UNKNOWN) | TO BE DECIDED |
| SEC-DEPLOY-4 | **NFR-1.8**, **NFR-1.9** | Release process | CONFIRMED |

## Dependency Security Rules

Prospective rules; no dependency exists yet and none has been assessed. Supply-chain references: `REF-SUPPLY`, `REF-VULNDEP`, `REF-CICD`, `REF-SSDF`.

- **DEP-1** The project MUST NOT add a dependency when the standard library or a small amount of straightforward, non-security-sensitive first-party code is safer and sufficient. The project MUST NOT replace vetted cryptography, authentication, authorization, protocol parsing, output encoding, HTML sanitization, or other security-critical functionality with custom code merely to avoid a dependency.
- **DEP-2** The project SHOULD prefer zero new dependencies. Every new dependency MUST be justified in the pull request description, including its purpose and why existing code or platform functionality is insufficient.
- **DEP-3** A new dependency MUST show evidence of active maintenance through a stable release, security response, or substantive maintainer activity within the previous 12 months. A mature project with less frequent releases requires an explicit documented exception and evidence that security reports are still handled.
- **DEP-4** The project MUST use the latest stable release from the latest actively supported major version unless a documented compatibility constraint requires another actively supported major version. Deprecated, abandoned, end-of-life, or pre-release packages MUST NOT be introduced into production.
- **DEP-5** Before a dependency is added or updated, direct and transitive dependencies MUST be checked for known vulnerabilities. A dependency with a known unpatched vulnerability applicable to the intended use MUST NOT be introduced without explicit, time-bounded risk acceptance, documented compensating controls, and a remediation plan. Ongoing patching of known-vulnerable dependencies is governed by SEC-DEPLOY-4.
- **DEP-6** Dependency review MUST include the complete transitive dependency graph. A small direct dependency with a disproportionately large, opaque, abandoned, or unvetted transitive tree SHOULD be rejected.
- **DEP-7** Production builds MUST resolve dependencies to exact versions through a committed lockfile or equivalent ecosystem mechanism. Production and CI builds MUST use frozen or reproducible dependency resolution and MUST NOT resolve floating versions at build or deployment time.
- **DEP-8** When multiple suitable libraries exist, the project SHOULD prefer the library with the narrowest required scope, smallest dependency tree, active security response process, clear provenance, and established security track record.

## Open Security Questions

- **SQ-1** Does HIPAA, or any regime with health-data-specific obligations, apply to a consumer-controlled personal-records product with no documented covered entity? Narrows OQ-1 and drives breach notification (SEC-DATA-5), retention (OQ-7), and session lifetimes (FR-1.10).
- **SQ-2** What is the account-recovery flow for a user who has lost every registered passkey (FR-1.6)? This is the critical gap in the passkey-only model (D-9) and blocks SEC-AUTHN-6 and SEC-EXT-2.
- **SQ-3** Session token mechanics: cookie-based or bearer? Decides whether CSRF defenses and cookie attributes apply (SEC-SESSION-3, `PRM-CSRF`) or token-storage rules do, and shapes SEC-HTTP-4.
- **SQ-4** Following CQ-2, is the web client Compose/Wasm (canvas) or a separate web implementation? Determines the XSS surface, the security-header/CSP set (SEC-HTTP-3), and the file-delivery mechanism (SEC-FILE-2).
- **SQ-5** Hosting platform and store technologies (database product, file-store technology): decide encryption-at-rest mechanisms and key management (SEC-DATA-1), network isolation for trust boundary 2, service identities (SEC-DEPLOY-2), the Terraform state backend (SEC-SECRET-3), and which provider-specific IaC guidance applies. Also decide whether encryption key custody can be separated from the data plane so at-rest encryption constrains a privileged infrastructure operator (T-13, T-20), and the mechanisms for security-log integrity (SEC-LOG-4, T-34), backup-artifact access and restore reconciliation (SEC-DATA-8, T-31–T-33, T-48), and hostile-content-parsing isolation (SEC-FILE-6, T-17), all of which depend on the platform.
- **SQ-6** CI/CD platform and secret-storage mechanism: blocks SEC-DEPLOY-3 and the injection mechanism in SEC-SECRET-1.
- **SQ-7** SEC-LOG-1 requires retaining the security event log (with source IPs) at least 12 months, while FR-9.3 requires permanent deletion of all personal data within 30 days of account deletion. Does the security log survive account deletion, on what basis, and in what form (e.g., pseudonymized with identity linkage severed)? A material conflict between two confirmed requirements; its erasure-vs-audit and legal-discovery facets are T-26, and SEC-DATA-7 must disclose the outcome (T-41).
- **SQ-8** Is the formal assurance target OWASP ASVS 5.0.0 Level 2 (SEC-DEPLOY-4 says "current version" Level 2), and who performs verification and when?
- **SQ-9** Abuse and resource limits beyond FR-1.4: per-user API rate limits, export-job frequency, and email-sending caps (reminders can trigger unbounded email). Values for SEC-HTTP-6 are TO BE DECIDED. This MUST also cover source-based and anti-automation throttling ahead of account lockout (T-1), registration-rate limits against multi-account abuse (T-21), and per-address / per-source caps on email-triggering operations so a scripted flow cannot mail-bomb third parties or burn sender reputation (T-11, T-30).
- **SQ-10** Does malware scanning (FR-3.9, SHOULD) ship in v1, and with which engine?
- **SQ-11** Incident-response ownership and process: who detects, triages, and notifies under SEC-DATA-5? No document assigns this.
- **SQ-12** What may reminder and notification emails contain, given appointment purpose and provider names are health-adjacent data crossing the only external boundary (SEC-DATA-6)? The same content question applies to the in-app notification feed, which has no analogue yet and whose items are unmodeled (T-29, T-43); a household adversary or mailbox observer can read whatever reaches the mail app's OS notifications (T-8). A minimal-content default (reference only existence and time, pointing to in-app detail) is the proposed resolution.
- **SQ-13** Does v1 provide tamper-evidence for evidentiary records, given the product exists to hold accident evidence for an adversarial insurance/legal dispute (T-14, T-47)? Candidates: upload-time content hashes substantiating SEC-FILE-5's byte-identical claim; protected/append-only timeline and journal history; and an audit trail (SEC-LOG-1, FR-8.1) over structured-record edits, document replacement (FR-3.7), and metadata edits (FR-3.6). This also decides whether FR-8.4 version retention and FR-3.7 prior-version retention are raised to MUST, and interacts with journal-entry deletability (REQUIREMENTS.md OQ-8).
- **SQ-14** What is the mobile client's security posture once `ARCHITECTURE.md` resolves the UNKNOWN target OSes and distribution model? Must specify client distribution, code-signing, and update integrity so the victim obtains an authentic client (T-36); on-screen privacy for shared or observed devices — sensitive-content masking and app re-lock, needing a `DESIGN.md` convention (DQ-6) (T-8); platform content-residue handling — task-switcher snapshot redaction, exclusion of app data from OS backups, and sensitive-field input hints (T-51); and, should the online-only ASSUMPTION be revisited, at-rest protection (platform secure storage/encryption) for any personal data cached on-device (T-37).
- **SQ-15** Contact records (the other driver, witnesses, employer, adjusters) and journal narrative hold personal data about third parties who are not the account owner, and exports (FR-7.6, FR-9.1, FR-9.2) hand it to insurers and attorneys (T-50). Under the OQ-1/SQ-1 regime, do these non-user data subjects acquire rights — breach notification (SEC-DATA-5), erasure — the system must honor? Every current rule treats the account owner as the sole data subject.
- **SQ-16** Beyond the step-up re-authentication (SEC-AUTHN-10) and out-of-band notifications (SEC-AUTHN-11) now required: does case deletion (FR-2.6) also warrant a fresh passkey ceremony, given it can destroy an entire case's evidence with only a confirmation click (T-12), and by what heuristic is a sign-in judged "not previously recognized" for the SEC-AUTHN-11 sign-in notice (T-4, T-27)?
