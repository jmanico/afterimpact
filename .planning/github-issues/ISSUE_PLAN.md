# AfterImpact — issue decomposition plan

**Run inputs:** `Scope: ALL` · `Execution mode: DRAFT_ONLY`
**Generated:** 2026-08-13 · **Target repository:** `jmanico/afterimpact`

## Run summary

| | |
| --- | --- |
| Sources read | `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`, `REQUIREMENT_TEMPLATE.md` (+ `style-guide.html`, `logo.svg`, `CLAUDE.md`) |
| Requirements in scope | 63 functional (`FR-1.1`–`FR-9.5`) + 27 non-functional (`NFR-1.1`–`NFR-2.5` authored in `SECURITY.md`; `NFR-3.1`–`NFR-6.2` in `REQUIREMENTS.md`) = **90** |
| Controls traced | 69 `SEC-*` + 8 `DEP-*` = **77** |
| Issues drafted | 1 epic + 17 workstreams + 175 leaves = **193** |
| Blocking open questions | **none** — all 14 `PQ-*` resolved |
| Questions answered in interview | **54** (`Q1`–`Q54`), none deferred |
| Decisions recorded | `D-18`–`D-52` in `REQUIREMENTS.md`; `SQ-17`–`SQ-19` and `SEC-TRUST-4` in `SECURITY.md`; new technology rows in `ARCHITECTURE.md`; `CQ-4` tracking the one retained marker |
| GitHub mutations performed | **none** — `DRAFT_ONLY` |

**Post-run amendment (2026-08-13).** The decomposition run itself modified no specification file. Afterwards,
the four contradictions it surfaced were decided by the product owner and recorded in `REQUIREMENTS.md` — the
document that owns those `FR-*` rules — as decisions `D-18` through `D-21`, with `FR-2.3`, `FR-3.7`, `FR-3.9`,
`FR-3.10`, `FR-7.5`, and `FR-9.4` amended to match, and the six `REQUIREMENTS.md`-owned gaps recorded as
`OQ-9`–`OQ-14` so the document no longer claims "None open". The affected issue drafts were revised to match.
Decisions were recorded in the owning document, never settled in an issue body or in code.

### Why 14 questions exist in a specification set that declares itself resolved

`REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, and `DESIGN.md` each record every `OQ-*`, `DQ-*`,
`CQ-*`, and `SQ-*` as resolved on 2026-08-13, and no live `TO BE DECIDED`, `UNKNOWN`, or `ASSUMPTION`
marker remains inside any requirement. The questions below are not those markers. They are conflicts and
silences found by reading the four documents against each other — cases where two documents disagree, or
where a requirement depends on a value, definition, or contract that no document states.

Each candidate was independently checked by an adversarial reviewer instructed to refute it by locating a
resolution anywhere in the set and to default to "refuted" when uncertain. Roughly half the candidates
were refuted and discarded; the 14 below survived. They are stated as questions for the owning document
to answer — **not** resolved here, and not to be settled in code.

---

## Open Questions

All 14 were **BLOCKING**: each affects observable behavior, an API contract, data semantics, a security
control, or acceptance testing. Under `DRAFT_ONLY` the affected scope is drafted and marked; under
`CREATE` the open ones must be answered first.

**Status: all 14 resolved. Nothing blocks a CREATE run on a specification question.**

They were closed in two passes. The four true contradictions (`PQ-5`, `PQ-6`, `PQ-11`, `PQ-12`) were
decided first and recorded as `D-18`–`D-21`. A full-repository sweep then collected every remaining open
item — open-question sections, `TO BE DECIDED` / `UNKNOWN` / `ASSUMPTION` markers, provisional defaults,
blank required inputs, and threat-model rows needing a decision — and a 54-question interview settled all
of them, including 20 follow-ups that only became visible as answers landed. Outcomes were recorded in the
document that owns each fact:

| Owning document | What it now records |
| --- | --- |
| `REQUIREMENTS.md` | `D-18`–`D-52`; new `FR-1.12`, `FR-4.10`, `FR-4.11`, `FR-10.1`, `FR-10.2`; a `Device` entity; an entity-level sensitivity classification; field length limits; `OQ-9`–`OQ-14` closed with a resolution map |
| `SECURITY.md` | `SEC-TRUST-4` (boundary-2 TLS); `SQ-17`–`SQ-19` resolved; amended `SEC-AUTHN-7`/`-10`/`-11`, `SEC-SESSION-3`/`-6`/`-7`, `SEC-HTTP-1`/`-3`/`-6`, `SEC-INPUT-1`, `SEC-LOG-1`/`-3`/`-4`, `SEC-EXT-4`, `SEC-DATA-7`/`-8` |
| `ARCHITECTURE.md` | region, compute, database, schema migrations, and test-tooling rows; the paginated list contract; `CQ-4` |
| `DESIGN.md` + `style-guide.html` | masking predicate and reveal scope, trash conventions, second-passkey prompt; the hardcoded mark fills replaced with token references |

**One marker is retained by decision, not omission.** `D-39` commissions a pre-launch legal review, and
until it reports the **Jurisdiction / Regulatory Scope** and **Regulatory** fields stay `TO BE DECIDED`
across the issue set — a guessed regulation would read as an assessed one. `ARCHITECTURE.md` `CQ-4` tracks
it as the single open cross-document question, and the `SEC-DEPLOY-4` release checklist gates it. The
**OWASP ASVS 5.0.0** and **NIST SP 800-53** rows likewise stay unmapped: `REQ-PLATFORM-100`'s
self-assessment produces the ASVS identifiers, and no 800-53 baseline has been selected.

`PQ-*` identifiers are kept stable because the issue drafts cite them when recording their resolution.

### PQ-1 — No mutation or deletion contract for health records
- **Affected**: `FR-4.1`, `FR-4.2`, `FR-4.3`, `FR-4.4`, `FR-4.6`, `FR-4.8`; `REQUIREMENTS.md` §4
- **Conflict**: §4 grants creation plus exactly two updates — `FR-4.2` status change and `FR-4.8` outcome notes. No requirement anywhere grants editing a health issue's name, body area, onset date, severity, or description; editing or cancelling an appointment; editing a progress update or treatment; or deleting any of these four record types. `FR-5.4` grants exactly this for tasks, so the omission reads as unintentional rather than deliberate.
- **Needed**: which of these records are editable and deletable, by which operation, and with what timeline (`FR-8.1`) and confirmation (`NFR-5.5`) consequences.
- **Blocked scope**: `REQ-HEALTH-100` entirely; the edit and delete facets of `REQ-HEALTH-010`, `-030`, `-040`, `-060`.

### PQ-2 — `User.timezone` has no lifecycle, and no rule anchors date-only fields
- **Affected**: `NFR-6.1`; `FR-2.1`, `FR-4.1`, `FR-5.5`, `FR-2.4`; `REQUIREMENTS.md` **Data Entities** (User)
- **Conflict**: The User entity carries `timezone` and `NFR-6.1` requires display in it, but no requirement creates, defaults, or updates that attribute — `FR-1.1`–`FR-1.11` cover registration, credentials, email change, and sessions, and `FR-5.6` covers only the email-reminder toggle. There is no profile or settings requirement. Separately, no rule fixes which timezone decides the boundary for date-only comparisons: "accident date MUST NOT be in the future" (`FR-2.1`), "onset date MUST NOT be in the future" (`FR-4.1`), and "due date has passed" (`FR-5.5`) each give different answers in the user's zone versus UTC.
- **Needed**: how a timezone is set and changed, its default, and which zone evaluates date-only boundaries.
- **Blocked scope**: facets of `REQ-CASE-010`, `REQ-CASE-040`, `REQ-HEALTH-010`, `REQ-TASK-050`, `REQ-DATA-010`.

### PQ-3 — No locale attribute for server-generated output
- **Affected**: `NFR-5.4`, `D-15`; `FR-5.6`, `FR-9.1`, `FR-9.5`, `SEC-AUTHN-11`, `SEC-EXT-2`
- **Conflict**: v1 ships English and Spanish, and the User entity models the parallel per-user display setting `timezone` — but carries no language or locale attribute, and no document states how a locale is chosen. Clients can infer one from the OS or `Accept-Language`, but a large class of output is generated with no client request in flight: reminder and security emails, verification and recovery mail, inactivity warnings, and case-summary PDFs.
- **Needed**: whether User gains a locale attribute, how it is set, and the fallback when none is known.
- **Blocked scope**: facets of `REQ-WEB-030`, `REQ-MOBILE-030`, `REQ-NOTIFY-030`, `REQ-EXPORT-010`, `REQ-AUTH-200`.

### PQ-4 — Reimbursement arithmetic is undefined
- **Affected**: `FR-7.1`, `FR-7.3`, `FR-7.4`; `NFR-6.2`
- **Conflict**: The Expense entity carries both a payment-status enum (`unpaid`, `paid`, `submitted for reimbursement`, `reimbursed`) and separate reimbursement records, and no document relates them. `FR-7.3`'s MUST-level "total not yet reimbursed" could mean the sum of expenses whose status is not `reimbursed`, or the sum of (amount − recorded reimbursements). The two disagree by real money whenever a partial reimbursement exists. It is also unstated whether recording reimbursements that sum to the full amount changes the payment status, and whether reimbursements may exceed the amount.
- **Needed**: the exact formula for both figures, and the relationship between the enum and the records.
- **Blocked scope**: `REQ-EXPENSE-030`, `REQ-EXPENSE-040`; the expense-total facet of `REQ-CASE-040`.

### PQ-5 — Case currency: no value set, a currency-independent decimal rule, contradicted mutability
- **Affected**: `FR-7.5`, `FR-7.1`, `FR-2.1`, `FR-2.3`; `D-15`, `D-8`, `SEC-INPUT-1`
- **Conflict**: Three problems. (a) No document enumerates the currencies a user may choose; `SEC-INPUT-1` lists every other enumeration the API must validate (categories, statuses, roles) and omits currency, and `FR-2.1` does not list currency among case-creation inputs even though `FR-7.5` says it is set at creation. (b) `FR-7.1` fixes amounts at two decimal places, which is not currency-independent. (c) `FR-2.3` allows the user to "edit any case field at any time" while `FR-7.5` fixes currency "at creation" — a direct contradiction.
- **Needed**: the allowed currency set, whether the decimal rule is per-currency, and whether currency is editable after creation.
- **Blocked scope**: `REQ-EXPENSE-050`; facets of `REQ-CASE-010`, `REQ-CASE-030`, `REQ-EXPENSE-010`.
- **PARTLY RESOLVED (2026-08-13) — `D-18`**: the contradiction (c) is settled. Currency is **immutable after
  creation**; `FR-2.3` now carves currency out of "edit any case field" and `FR-7.5` states the prohibition,
  because `D-15` forbids conversion and a change would relabel recorded amounts without restating them.
  **Still open:** (a) no enumerated set of allowed currency values, and (b) `FR-7.1`'s two-decimal rule is not
  currency-independent. Those two are silences, not contradictions, and remain blocking for `REQ-EXPENSE-050`
  and the validation facet of `REQ-EXPENSE-010`.

### PQ-6 — Prior document versions have no retention bound and no reclaim path
- **Affected**: `FR-3.7`, `FR-3.10`
- **Conflict**: `FR-3.7` requires retaining prior versions with no version limit, no retention period, and no operation to delete one. `FR-3.10` counts them against the 25 GB quota and requires the usage breakdown expressly "so space is reclaimable". Trashed documents self-purge after 30 days and export artifacts after 7, but prior versions have neither a timer nor a user action — so on the documents as written that space can never be reclaimed.
- **Needed**: a retention bound, a version cap, or a user-initiated deletion path for prior versions.
- **Blocked scope**: facets of `REQ-DOC-020`, `REQ-DOC-100`.
- **RESOLVED (2026-08-13) — `D-21`**: prior versions are retained indefinitely, **no operation deletes one**,
  and they **do not count toward the `FR-3.10` quota**. Exemption was chosen over a delete operation because
  `FR-3.7`'s retention exists so the authentic original survives a doctored replacement (`T-47`); a user-facing
  delete would hand that capability to anyone holding a session. `FR-3.7` and `FR-3.10` amended.

### PQ-7 — `FR-9.5` never defines "inactive" and nothing measures it
- **Affected**: `FR-9.5`, `D-16`
- **Conflict**: The requirement mandates permanently purging accounts "inactive for 3 years" with warnings at ~60 and ~30 days, but no document says what resets that clock — a successful sign-in, any authenticated request, a data write, a fired reminder, or opening an email — and the User entity carries no last-activity or last-sign-in attribute. Every other purge in the set is anchored to a stated timestamp.
- **Needed**: the definition of activity and the attribute that records it.
- **Blocked scope**: `REQ-AUTH-200` entirely.

### PQ-8 — Step-up re-authentication has no freshness lifetime and no API representation
- **Affected**: `SEC-AUTHN-7`, `SEC-AUTHN-10`; `FR-1.7`, `FR-1.8`, `FR-2.6`, `FR-9.2`, `FR-9.3`
- **Conflict**: Six operations are gated on a "fresh passkey authentication ceremony … completed within the same session immediately before the operation", but no document states a freshness lifetime, how the requirement and its proof are expressed in the API (a distinct refusal status the client can act on, a step-up token, or a session flag), or whether one ceremony can cover more than one guarded operation.
- **Needed**: the freshness window, the wire contract, and the single-versus-multiple-operation rule.
- **Blocked scope**: `REQ-AUTH-160`; the step-up facets of `REQ-CASE-060`, `REQ-EXPORT-020`, `REQ-AUTH-130`, `REQ-AUTH-150`, `REQ-AUTH-180`, and both clients' step-up prompts.

### PQ-9 — Mobile session credential transport is unspecified
- **Affected**: `SEC-SESSION-3`, `SEC-SESSION-6`; `NFR-1.7`, `FR-1.10`
- **Conflict**: `SEC-SESSION-3` fixes the web transport exactly — HttpOnly, Secure, SameSite=Lax cookie plus CSRF defenses — but for mobile states only where the credential is *stored* (`SEC-SESSION-6`: iOS Keychain, Android Keystore), never how it is *presented* to the API. No document names a bearer or `Authorization`-header contract. If the mobile client instead reuses the cookie mechanism, its HTTP-client cookie store is not platform secure storage, which `SEC-SESSION-6` requires.
- **Needed**: the mobile transport contract, and how it satisfies `SEC-SESSION-6` and the CSRF posture.
- **Blocked scope**: facets of `REQ-AUTH-070`, `REQ-MOBILE-010`, `REQ-MOBILE-190`.

### PQ-10 — The device identifier behind unrecognized-sign-in notices is modeled as no data
- **Affected**: `SEC-AUTHN-11`; `SEC-DATA-2`, `SEC-DATA-4`, `SEC-DATA-7`, `SEC-LOG-1`, `FR-9.2`, `FR-9.3`, `FR-9.5`, `NFR-2.3`
- **Conflict**: `SEC-AUTHN-11` introduces a persistent per-account cookie-anchored device identifier plus a history of devices "previously seen for the account", and makes a security email fire on its absence. No document models it: it is not in **Data Entities**, and has no lifetime, no per-account cap, no purge rule on account deletion or inactivity purge, no statement whether it is included in the account export, and no mobile-client definition where there is no cookie. The Notification feed received exactly this treatment under `T-43`; the device identifier did not.
- **Needed**: the entity, its retention and cap, its purge and export treatment, and its mobile equivalent.
- **Blocked scope**: the unrecognized-sign-in facet of `REQ-AUTH-170`.

### PQ-11 — Full-account archives cannot fit the quota they are charged against
- **Affected**: `FR-9.2`, `FR-9.4`, `FR-3.10`; `SEC-HTTP-6`, `NFR-2.4`, `SEC-DATA-4`
- **Conflict**: `FR-9.2` requires the archive to contain "every original uploaded file"; `FR-9.4` requires export artifacts to count toward the `FR-3.10` quota and to persist 7 days; `SEC-HTTP-6` permits 2 full-account exports per day. Against a 25 GB quota this is arithmetically impossible: any user holding more than roughly 12.5 GB of originals cannot generate even one archive without exceeding the quota, and the portability right `NFR-2.4`/`SEC-DATA-4` guarantee becomes unavailable exactly for the users with the most data.
- **Needed**: whether archives are exempt from the quota, whether the quota rises, or whether archives stream without being stored.
- **Blocked scope**: `REQ-EXPORT-060`; facets of `REQ-EXPORT-020`, `REQ-DOC-020`.
- **RESOLVED (2026-08-13) — `D-19`**: full-account archives (`FR-9.2`) **do not count** toward the `FR-3.10`
  quota; case-summary PDFs (`FR-9.1`) and expense CSVs (`FR-7.6`) still do. The `FR-9.4` seven-day artifact
  purge is unchanged, so `T-18`'s stale-aggregate concern is unaffected, and portability under `NFR-2.4` /
  `SEC-DATA-4` now works at any data volume. `FR-3.10` and `FR-9.4` amended.

### PQ-12 — No defined document state during the asynchronous scan window
- **Affected**: `FR-3.9`, `SEC-FILE-4`; `FR-3.5`, `FR-8.1`, `FR-9.1`, `FR-9.2`
- **Conflict**: `FR-3.9` is written synchronously — scan every upload and "reject a flagged file with a notice to the user" — but `ARCHITECTURE.md` places scanning in flow 3, the asynchronous worker, and omits it from flow 2, the upload path. So the upload is acknowledged before it is scanned, and no document defines the document's observable state in between: whether it is listable, downloadable, viewable, linkable, exportable, or counted against the quota, and what the user sees if a file they already used is flagged later.
- **Needed**: the interim state and its effect on listing, retrieval, export, and quota.
- **Blocked scope**: facets of `REQ-DOC-120`, `REQ-DOC-070`, `REQ-DOC-080`.
- **RESOLVED (2026-08-13) — `D-20`**: scanning is **asynchronous with an explicit pending state**. While a
  scan is pending the document is listed and marked pending, and MUST NOT be downloadable, viewable in the app,
  or included in any export; cleared files become fully available and flagged files are quarantined. This keeps
  the `SEC-FILE-6` worker isolation and the `NFR-4.2` upload budget intact while closing the `T-16`
  redistribution window. `FR-3.9` amended.

### PQ-13 — Link-deletion behavior is stated only for Contact
- **Affected**: `FR-6.3`; `FR-3.2`, `FR-4.5`, `FR-7.1`, `FR-3.8`, `FR-2.6`
- **Conflict**: `FR-6.3` is the only rule in the set for what happens to referencing records when a linked record is deleted, and it is contact-specific (retain the name as text). Nothing states the behavior when a Document serving as an Expense's linked receipt is deleted; when a Health Issue linked from Documents, Expenses, and Appointments is deleted; or when an Expense linked from a Document is deleted. Trash (`FR-3.8`) makes this sharper: a trashed document is recoverable for 30 days, so links to it must survive or not, and no document says which.
- **Needed**: the referential rule for each non-Contact link type, including across the soft-delete window.
- **Blocked scope**: facets of `REQ-HEALTH-050`, `REQ-DOC-110`, `REQ-EXPENSE-010`, `REQ-DATA-020`.

### PQ-14 — "Content flagged sensitive" has no defined predicate
- **Affected**: `DESIGN.md` **On-Screen Privacy** (`DQ-6`); `SEC-SESSION-7`, `SEC-DATA-2`, `NFR-5.1`
- **Conflict**: `DESIGN.md` makes masking a default rendering behavior for "content flagged sensitive", with reveal "never the default state". No document defines the flagging predicate: there is no per-field or per-entity sensitivity classification in **Data Entities** or in `SECURITY.md`, and `SEC-SESSION-7`'s related clause ("flag sensitive fields to platform keyboards") reuses the same undefined term. Under `D-5` every record is sensitive, which would mask the entire interface by default and conflict with `NFR-5.1`'s expectations.
- **Needed**: which fields or entities are flagged sensitive, or the rule that decides.
- **Blocked scope**: `REQ-DESIGN-060`; facets of `REQ-MOBILE-200`.

---

### Narrower gaps recorded per issue

The 14 questions above are the ones that block whole behaviors. Drafting surfaced further, smaller
silences that are recorded in the affected issue's **Open Decisions** field rather than promoted to
`PQ-*` — each is scoped to one issue and does not block its neighbours. They were recorded, never
guessed. Examples:

- `REQ-CASE-020` — whether the case list can include archived cases, the tiebreak when two cases share an accident date, and whether the list pages.
- `REQ-CASE-040` — what "upcoming deadlines" counts as, and how many "most recent timeline events" the overview shows.
- `REQ-CASE-050` — whether archive and unarchive emit an `FR-8.1` timeline event.
- `REQ-EXPENSE-040` — `FR-7.4` states only "amount + date" and never restates `FR-7.1`'s `> 0` and two-decimal rules for a reimbursement amount.
- `REQ-TIMELINE-000` / `-040` — `ARCHITECTURE.md` assigns no component the `FR-8.4` journal purge (its worker list names only `FR-2.6` and `FR-3.8`); `FR-8.4` names no surface for the 30-day journal undo, since `FR-3.8`'s trash area is document-specific.

Reviewing these alongside the `PQ-*` list is worthwhile before a `CREATE` run, but they do not gate it.

---

## Defects already assigned to an issue (not open questions)

These are real problems, but the specifications state the governing rule clearly — only the artifact fails
to implement it. They need no decision, so they are work, not questions.

| Defect | Owning rule | Assigned to |
| --- | --- | --- |
| `logo.svg` hardcodes light-theme fills `#234E70` / `#0F766E`, so the mark cannot render with the active theme's tokens | `DESIGN.md` **Brand and Logo** | `REQ-DESIGN-070` |
| `style-guide.html` header mark hardcodes the same fills instead of referencing tokens | `DESIGN.md` **Color Palette** (components reference tokens, never raw hex) | `REQ-DESIGN-080` |
| `style-guide.html` renders no example of the destructive-action confirmation, the one-time journal-authoring notice, or the On-Screen Privacy masking/re-lock panel | `CLAUDE.md` (the two must agree); `DESIGN.md` **Components**, **On-Screen Privacy** | `REQ-DESIGN-080` |

---

## Hierarchy

```
REQ-EPIC-001  Deliver AfterImpact v1
├── REQ-PLATFORM-000  Platform, deployment, and supply chain      (11 leaves)
├── REQ-API-000       REST API foundation and boundary controls   (12 leaves)
├── REQ-DATA-000      Data persistence and protection             (10 leaves)
├── REQ-AUTH-000      Accounts, authentication, and sessions      (21 leaves)
├── REQ-LOG-000       Security event logging and activity view     (5 leaves)
├── REQ-NOTIFY-000    Notification feed and outbound email         (5 leaves)
├── REQ-CASE-000      Accident cases                               (7 leaves)
├── REQ-DOC-000       Documents and paperwork                     (13 leaves)
├── REQ-HEALTH-000    Health issues and appointments              (10 leaves)
├── REQ-TASK-000      Recovery tasks, deadlines, and reminders    (10 leaves)
├── REQ-CONTACT-000   Contacts                                     (4 leaves)
├── REQ-EXPENSE-000   Expenses                                     (6 leaves)
├── REQ-TIMELINE-000  Timeline and journal                         (6 leaves)
├── REQ-EXPORT-000    Export and data management                   (6 leaves)
├── REQ-DESIGN-000    Design language and style guide              (8 leaves)
├── REQ-WEB-000       Web client                                  (19 leaves)
└── REQ-MOBILE-000    Mobile client                               (22 leaves)
```

Capability names are taken from the source documents' own section names: the `FR` group names in
`REQUIREMENTS.md`, component names in `ARCHITECTURE.md`, rule-group names in `SECURITY.md`, and section
names in `DESIGN.md`. The dependency graph is acyclic: platform → API → data → identity → records →
clients, with `REQ-DESIGN-000` preceding both client workstreams.

---

## Requirement coverage

Trust boundaries are those in `ARCHITECTURE.md` **Trust boundaries**: **1** client ↔ API, **2** API ↔
stores, **3** system ↔ email provider, **4** system ↔ identity provider. Status is `COVERED`,
`PARTIALLY COVERED` (implementable except for a named `PQ-*` facet), `BLOCKED` (not implementable at all
until a `PQ-*` is answered), or `OUT OF SCOPE`.

### Functional requirements

| Requirement | Issue IDs | Boundary | Security rules | Design sections | Status |
| --- | --- | --- | --- | --- | --- |
| FR-1.1 | REQ-AUTH-020, REQ-WEB-040, REQ-MOBILE-040 | 1, 3, 4 | SEC-AUTHN-3, SEC-AUTHN-5, SEC-EXT-2 | Components → Authentication and recovery | COVERED |
| FR-1.2 | REQ-AUTH-040, REQ-AUTH-050, REQ-AUTH-210 | 1, 4 | SEC-AUTHN-1, SEC-AUTHN-6, SEC-AUTHN-9 | Components → Authentication and recovery | COVERED |
| FR-1.3 | REQ-AUTH-060, REQ-AUTH-110 | 1, 4 | SEC-AUTHN-1, SEC-AUTHN-5, SEC-SESSION-4 | N/A | COVERED |
| FR-1.4 | REQ-AUTH-100 | 1 | SEC-AUTHN-4, SEC-HTTP-6, SEC-LOG-1 | N/A | COVERED |
| FR-1.5 | REQ-AUTH-080 | 1 | SEC-SESSION-1 | N/A | COVERED |
| FR-1.6 | REQ-AUTH-120 | 1, 3, 4 | SEC-AUTHN-6, SEC-SESSION-5, SEC-EXT-2 | Components → Authentication and recovery | COVERED |
| FR-1.7 | REQ-AUTH-150 | 1, 3 | SEC-AUTHN-3, SEC-AUTHN-10, SEC-AUTHN-11, SEC-SESSION-5 | N/A | COVERED (SQ-17) |
| FR-1.8 | REQ-AUTH-050, REQ-AUTH-130 | 1, 3, 4 | SEC-AUTHN-8, SEC-AUTHN-10, SEC-SESSION-5 | N/A | COVERED (SQ-17) |
| FR-1.9 | REQ-API-020 | 1 | SEC-AUTHZ-1, SEC-AUTHZ-2, SEC-AUTHZ-3, SEC-AUTHZ-4, SEC-AUTHZ-5 | N/A | COVERED |
| FR-1.10 | REQ-AUTH-090 | 1 | SEC-SESSION-2, SEC-SESSION-3 | N/A | COVERED |
| FR-1.11 | REQ-AUTH-140 | 1 | SEC-AUTHN-8 | N/A | COVERED |
| FR-2.1 | REQ-CASE-010 | 1 | SEC-INPUT-1 | N/A | COVERED (D-23, D-18, D-28) |
| FR-2.2 | REQ-CASE-020 | 1 | SEC-INPUT-2 | N/A | COVERED |
| FR-2.3 | REQ-CASE-030 | 1 | SEC-INPUT-1 | N/A | COVERED (D-18) |
| FR-2.4 | REQ-CASE-040 | 1 | SEC-INPUT-2 | N/A | COVERED (D-23) |
| FR-2.5 | REQ-CASE-050 | 1 | N/A | N/A | COVERED |
| FR-2.6 | REQ-CASE-060, REQ-CASE-070 | 1, 2 | SEC-AUTHN-10, SEC-AUTHN-11, SEC-LOG-1, SEC-DATA-8 | Components → Destructive actions | COVERED (SQ-17) |
| FR-3.1 | REQ-DOC-010, REQ-DOC-020 | 1 | SEC-INPUT-3, SEC-INPUT-1 | Components → Form feedback | COVERED |
| FR-3.2 | REQ-DOC-040 | 1 | SEC-INPUT-1, SEC-FILE-3 | Components → Inputs | COVERED |
| FR-3.3 | REQ-DOC-050 | 1 | SEC-INPUT-2 | N/A | COVERED |
| FR-3.4 | REQ-DOC-060 | 1, 2 | SEC-INPUT-2 | N/A | COVERED |
| FR-3.5 | REQ-DOC-070, REQ-DOC-080 | 1, 2 | SEC-FILE-1, SEC-FILE-2, SEC-FILE-5, SEC-TRUST-3 | N/A | COVERED (D-20) |
| FR-3.6 | REQ-DOC-090 | 1 | SEC-INPUT-1 | N/A | COVERED |
| FR-3.7 | REQ-DOC-100, REQ-DATA-030 | 1, 2 | SEC-FILE-5 | N/A | COVERED (D-21) |
| FR-3.8 | REQ-DOC-110 | 1, 2 | SEC-LOG-1, SEC-DATA-8 | Components → Destructive actions | COVERED (D-27, D-43) |
| FR-3.9 | REQ-DOC-120 | 2 | SEC-FILE-4, SEC-FILE-6 | N/A | COVERED (D-20) |
| FR-3.10 | REQ-DOC-020, REQ-EXPORT-060 | 1 | SEC-INPUT-3 | N/A | COVERED (D-19, D-21) |
| FR-4.1 | REQ-HEALTH-010, REQ-HEALTH-100 | 1 | SEC-INPUT-1 | N/A | COVERED (D-22, D-23) |
| FR-4.2 | REQ-HEALTH-020 | 1 | SEC-INPUT-1 | N/A | COVERED |
| FR-4.3 | REQ-HEALTH-030, REQ-WEB-080, REQ-MOBILE-080 | 1 | SEC-INPUT-1 | Accessibility (never colour alone) | COVERED (D-22) |
| FR-4.4 | REQ-HEALTH-040 | 1 | SEC-INPUT-1 | N/A | COVERED (D-22) |
| FR-4.5 | REQ-HEALTH-050 | 1 | N/A | N/A | COVERED (D-27, D-43) |
| FR-4.6 | REQ-HEALTH-060 | 1 | SEC-INPUT-1 | N/A | COVERED (D-22) |
| FR-4.7 | REQ-HEALTH-070 | 1, 2 | N/A | N/A | COVERED |
| FR-4.8 | REQ-HEALTH-080 | 1 | SEC-INPUT-1 | N/A | COVERED |
| FR-4.9 | REQ-HEALTH-090 | 1 | N/A | Brand and Logo (plain language, no urgency) | COVERED |
| FR-5.1 | REQ-TASK-020 | 1 | N/A | Components → User-authored content | COVERED |
| FR-5.2 | REQ-TASK-010 | 1 | SEC-INPUT-1 | N/A | COVERED |
| FR-5.3 | REQ-TASK-030 | 1 | N/A | N/A | COVERED |
| FR-5.4 | REQ-TASK-040 | 1 | N/A | Components → Destructive actions | COVERED |
| FR-5.5 | REQ-TASK-050 | 1 | N/A | Accessibility (never colour alone) | COVERED (D-23) |
| FR-5.6 | REQ-TASK-060, REQ-TASK-070, REQ-TASK-080 | 1, 2, 3 | SEC-DATA-6, SEC-EXT-4, SEC-AUTHN-11 | N/A | COVERED (D-24) |
| FR-5.7 | REQ-TASK-090 | 1, 2, 3 | SEC-DATA-6 | Brand and Logo (no urgency theatrics) | COVERED |
| FR-5.8 | REQ-TASK-100 | 1 | N/A | Components → User-authored content | COVERED |
| FR-6.1 | REQ-CONTACT-010 | 1 | SEC-INPUT-1, SEC-OUT-3 | Components → Links | COVERED |
| FR-6.2 | REQ-CONTACT-010 | 1 | N/A | N/A | COVERED |
| FR-6.3 | REQ-CONTACT-030, REQ-CONTACT-040, REQ-DATA-040 | 1, 2 | N/A | Components → Destructive actions | COVERED |
| FR-6.4 | REQ-CONTACT-020 | 1 | SEC-INPUT-2 | N/A | COVERED |
| FR-7.1 | REQ-EXPENSE-010 | 1 | SEC-INPUT-1 | N/A | COVERED (D-18, D-28, D-27, D-43) |
| FR-7.2 | REQ-EXPENSE-020 | 1 | N/A | Components → Destructive actions | COVERED |
| FR-7.3 | REQ-EXPENSE-030 | 1 | N/A | N/A | COVERED (D-25) |
| FR-7.4 | REQ-EXPENSE-040 | 1 | N/A | N/A | COVERED (D-25) |
| FR-7.5 | REQ-EXPENSE-050 | 1 | SEC-INPUT-1 | N/A | COVERED (D-18, D-28) |
| FR-7.6 | REQ-EXPENSE-060 | 1, 2 | SEC-OUT-2, SEC-LOG-1, SEC-HTTP-6 | N/A | COVERED |
| FR-8.1 | REQ-TIMELINE-010 | 1, 2 | SEC-LOG-1 | N/A | COVERED |
| FR-8.2 | REQ-TIMELINE-020 | 1 | SEC-OUT-1, SEC-INPUT-1 | Components → User-authored content | COVERED |
| FR-8.3 | REQ-TIMELINE-050 | 1 | SEC-INPUT-2 | N/A | COVERED |
| FR-8.4 | REQ-TIMELINE-030, REQ-TIMELINE-040, REQ-TIMELINE-060, REQ-DATA-030 | 1, 2 | SEC-LOG-1 | Components → Journal authoring, Destructive actions | COVERED |
| FR-9.1 | REQ-EXPORT-010 | 2 | SEC-OUT-2, SEC-FILE-6, SEC-HTTP-6 | N/A | COVERED (D-24) |
| FR-9.2 | REQ-EXPORT-020 | 1, 2, 3 | SEC-AUTHN-10, SEC-AUTHN-11, SEC-OUT-2, SEC-FILE-1 | N/A | COVERED (SQ-17) |
| FR-9.3 | REQ-AUTH-180, REQ-AUTH-190 | 1, 2, 3 | SEC-AUTHN-7, SEC-AUTHN-11, SEC-DATA-4, SEC-LOG-1 | Components → Destructive actions | COVERED (SQ-17) |
| FR-9.4 | REQ-EXPORT-040, REQ-EXPORT-050, REQ-EXPORT-060 | 2 | SEC-DATA-4 | Components → Journal authoring | COVERED (D-19) |
| FR-9.5 | REQ-AUTH-200 | 2, 3 | SEC-DATA-4, SEC-DATA-6 | N/A | COVERED (D-26) |

### Non-functional requirements

`NFR-1.*` and `NFR-2.*` are authored in `SECURITY.md` on the control that enforces them; `NFR-3.*`–`NFR-6.*`
are authored in `REQUIREMENTS.md`.

| Requirement | Issue IDs | Boundary | Security rules | Design sections | Status |
| --- | --- | --- | --- | --- | --- |
| NFR-1.1 | REQ-API-030, REQ-PLATFORM-070 | 1 | SEC-HTTP-1 | N/A | COVERED |
| NFR-1.2 | REQ-DATA-050, REQ-PLATFORM-020, REQ-PLATFORM-030, REQ-PLATFORM-040 | 2 | SEC-DATA-1 | N/A | COVERED |
| NFR-1.3 | REQ-AUTH-210 | 4 | SEC-AUTHN-9 | N/A | COVERED |
| NFR-1.4 | REQ-DOC-070, REQ-EXPORT-030, REQ-PLATFORM-050 | 2 | SEC-FILE-1, SEC-TRUST-2, SEC-SECRET-2, SEC-AUTHZ-4 | N/A | COVERED |
| NFR-1.5 | REQ-DOC-080, REQ-API-050, REQ-WEB-180, REQ-MOBILE-180 | 1 | SEC-TRUST-3, SEC-FILE-2, SEC-HTTP-3, SEC-OUT-1 | Components → User-authored content | COVERED (D-20) |
| NFR-1.6 | REQ-LOG-010, REQ-LOG-030, REQ-LOG-050 | 2 | SEC-LOG-1, SEC-LOG-3, SEC-LOG-4 | N/A | COVERED |
| NFR-1.7 | REQ-LOG-020, REQ-AUTH-070, REQ-PLATFORM-060 | 1, 2 | SEC-LOG-2, SEC-SESSION-3, SEC-SECRET-1, SEC-DATA-6 | N/A | COVERED (SQ-18) |
| NFR-1.8 | REQ-PLATFORM-080, REQ-PLATFORM-090, REQ-PLATFORM-100 | N/A | SEC-DEPLOY-3, SEC-DEPLOY-4 | N/A | COVERED |
| NFR-1.9 | REQ-PLATFORM-080, REQ-PLATFORM-100 | N/A | SEC-DEPLOY-3, SEC-DEPLOY-4 | N/A | COVERED |
| NFR-2.1 | REQ-DATA-090 | 1 | SEC-DATA-3 | N/A | COVERED |
| NFR-2.2 | REQ-AUTH-030 | 1 | SEC-DATA-7 | N/A | COVERED |
| NFR-2.3 | REQ-DATA-080 | 2 | SEC-DATA-2 | N/A | COVERED (D-30) |
| NFR-2.4 | REQ-AUTH-190, REQ-EXPORT-020 | 2 | SEC-DATA-4 | N/A | COVERED (D-19) |
| NFR-2.5 | REQ-DATA-100 | 3 | SEC-DATA-5 | N/A | COVERED |
| NFR-3.1 | REQ-PLATFORM-070, REQ-DATA-060 | N/A | N/A | N/A | COVERED |
| NFR-3.2 | REQ-DATA-060, REQ-DATA-070 | 2 | SEC-DATA-8 | N/A | COVERED |
| NFR-3.3 | REQ-PLATFORM-030, REQ-DATA-060, REQ-DOC-130 | 2 | N/A | N/A | COVERED |
| NFR-4.1 | REQ-API-120 | 1 | N/A | N/A | COVERED |
| NFR-4.2 | REQ-DOC-130 | 1 | N/A | N/A | COVERED |
| NFR-4.3 | REQ-DOC-060, REQ-API-120 | 1, 2 | N/A | N/A | COVERED |
| NFR-5.1 | REQ-DESIGN-050, REQ-WEB-190, REQ-MOBILE-210 | 1 | N/A | Accessibility | COVERED (D-31, D-44) |
| NFR-5.2 | REQ-DESIGN-020, REQ-WEB-190, REQ-MOBILE-210 | 1 | N/A | Layout and Spacing | COVERED |
| NFR-5.3 | REQ-API-110, REQ-WEB-170, REQ-MOBILE-170 | 1 | SEC-ERR-1 | Components → Form feedback | COVERED |
| NFR-5.4 | REQ-WEB-030, REQ-MOBILE-030 | 1 | N/A | Layout and Spacing (text expansion) | COVERED (D-24) |
| NFR-5.5 | REQ-DESIGN-040, REQ-WEB-160, REQ-MOBILE-160 | 1 | SEC-DATA-8 | Components → Destructive actions | COVERED |
| NFR-6.1 | REQ-DATA-010 | 2 | SEC-INPUT-4 | N/A | COVERED (D-23) |
| NFR-6.2 | REQ-DATA-010, REQ-EXPENSE-030 | 2 | N/A | N/A | COVERED (D-25) |

**Coverage result:** 90 of 90 requirements map to at least one drafted issue, and **all 90 are now
`COVERED`** — nothing is `BLOCKED` or `PARTIALLY COVERED`, because every specification question the
decomposition raised has been answered and recorded. Nothing is `OUT OF SCOPE`: `Scope: ALL` was
requested and every requirement in `REQUIREMENTS.md` is in the tree.

---

## Control coverage

Supplementary to the requirement table, so that no `SECURITY.md` control lacks an implementing issue.

| Control | Issue IDs | Status |
| --- | --- | --- |
| SEC-TRUST-1 | REQ-API-020, REQ-API-090, REQ-DOC-010 | COVERED |
| SEC-TRUST-2 | REQ-PLATFORM-010, REQ-PLATFORM-050, REQ-DATA-050 | COVERED |
| SEC-TRUST-3 | REQ-DOC-080, REQ-API-050 | COVERED |
| SEC-AUTHN-1 | REQ-AUTH-050, REQ-AUTH-060, REQ-AUTH-010 | COVERED |
| SEC-AUTHN-2 | REQ-AUTH-050 | COVERED |
| SEC-AUTHN-3 | REQ-AUTH-020, REQ-AUTH-150 | COVERED |
| SEC-AUTHN-4 | REQ-AUTH-100, REQ-API-070 | COVERED |
| SEC-AUTHN-5 | REQ-AUTH-110, REQ-AUTH-020 | COVERED |
| SEC-AUTHN-6 | REQ-AUTH-040, REQ-AUTH-120 | COVERED |
| SEC-AUTHN-7 | REQ-AUTH-160, REQ-AUTH-180 | COVERED (SQ-17) |
| SEC-AUTHN-8 | REQ-AUTH-130, REQ-AUTH-140 | COVERED |
| SEC-AUTHN-9 | REQ-AUTH-210 | COVERED |
| SEC-AUTHN-10 | REQ-AUTH-160 | COVERED (SQ-17) |
| SEC-AUTHN-11 | REQ-AUTH-170 | COVERED (D-30) |
| SEC-SESSION-1 | REQ-AUTH-080 | COVERED |
| SEC-SESSION-2 | REQ-AUTH-090 | COVERED |
| SEC-SESSION-3 | REQ-AUTH-010, REQ-AUTH-070 | COVERED (SQ-18) |
| SEC-SESSION-4 | REQ-AUTH-060 | COVERED |
| SEC-SESSION-5 | REQ-AUTH-120, REQ-AUTH-130, REQ-AUTH-150 | COVERED |
| SEC-SESSION-6 | REQ-MOBILE-190 | COVERED (SQ-18) |
| SEC-SESSION-7 | REQ-MOBILE-200, REQ-MOBILE-220, REQ-DESIGN-060 | COVERED (D-31, D-44) |
| SEC-AUTHZ-1 … SEC-AUTHZ-5 | REQ-API-020 | COVERED |
| SEC-HTTP-1 | REQ-API-030, REQ-PLATFORM-070 | COVERED |
| SEC-HTTP-2 | REQ-API-040 | COVERED |
| SEC-HTTP-3 | REQ-API-050 | COVERED |
| SEC-HTTP-4 | REQ-API-060 | COVERED |
| SEC-HTTP-5 | REQ-API-060 | COVERED |
| SEC-HTTP-6 | REQ-API-070 | COVERED |
| SEC-INPUT-1 | REQ-API-090 | COVERED |
| SEC-INPUT-2 | REQ-API-100 | COVERED |
| SEC-INPUT-3 | REQ-DOC-010, REQ-DOC-020 | COVERED |
| SEC-INPUT-4 | REQ-API-080 | COVERED |
| SEC-FILE-1 | REQ-DOC-070, REQ-EXPORT-030, REQ-PLATFORM-030 | COVERED |
| SEC-FILE-2 | REQ-DOC-080, REQ-DOC-070 | COVERED |
| SEC-FILE-3 | REQ-DOC-030 | COVERED |
| SEC-FILE-4 | REQ-DOC-120 | COVERED (D-20) |
| SEC-FILE-5 | REQ-DOC-030, REQ-DOC-070 | COVERED |
| SEC-FILE-6 | REQ-DOC-120, REQ-EXPORT-010 | COVERED |
| SEC-OUT-1 | REQ-WEB-180, REQ-MOBILE-180, REQ-DESIGN-030 | COVERED |
| SEC-OUT-2 | REQ-EXPENSE-060, REQ-EXPORT-010, REQ-EXPORT-020 | COVERED |
| SEC-OUT-3 | REQ-WEB-180, REQ-MOBILE-180 | COVERED |
| SEC-DATA-1 | REQ-DATA-050, REQ-PLATFORM-020, REQ-PLATFORM-030, REQ-PLATFORM-040 | COVERED |
| SEC-DATA-2 | REQ-DATA-080 | COVERED (D-30) |
| SEC-DATA-3 | REQ-DATA-090 | COVERED |
| SEC-DATA-4 | REQ-AUTH-190, REQ-EXPORT-020 | COVERED (D-19) |
| SEC-DATA-5 | REQ-DATA-100 | COVERED |
| SEC-DATA-6 | REQ-NOTIFY-030 | COVERED (D-24) |
| SEC-DATA-7 | REQ-AUTH-030 | COVERED |
| SEC-DATA-8 | REQ-DATA-070 | COVERED |
| SEC-SECRET-1 | REQ-PLATFORM-060 | COVERED |
| SEC-SECRET-2 | REQ-PLATFORM-050 | COVERED |
| SEC-SECRET-3 | REQ-PLATFORM-010 | COVERED |
| SEC-LOG-1 | REQ-LOG-010, REQ-LOG-040 | COVERED |
| SEC-LOG-2 | REQ-LOG-020 | COVERED |
| SEC-LOG-3 | REQ-LOG-050, REQ-WEB-150, REQ-MOBILE-150 | COVERED |
| SEC-LOG-4 | REQ-LOG-030 | COVERED |
| SEC-ERR-1 | REQ-API-110, REQ-WEB-170, REQ-MOBILE-170 | COVERED |
| SEC-EXT-1 | REQ-PLATFORM-090 | COVERED |
| SEC-EXT-2 | REQ-NOTIFY-050, REQ-AUTH-020, REQ-AUTH-120 | COVERED |
| SEC-EXT-3 | REQ-PLATFORM-110 | COVERED |
| SEC-EXT-4 | REQ-PLATFORM-110, REQ-NOTIFY-040 | COVERED |
| SEC-DEPLOY-1 | REQ-PLATFORM-010 | COVERED |
| SEC-DEPLOY-2 | REQ-PLATFORM-050 | COVERED |
| SEC-DEPLOY-3 | REQ-PLATFORM-080 | COVERED |
| SEC-DEPLOY-4 | REQ-PLATFORM-100, REQ-PLATFORM-090 | COVERED |
| DEP-1 … DEP-8 | REQ-PLATFORM-090 | COVERED |

**Control result:** all 69 `SEC-*` controls and all 8 `DEP-*` rules have at least one implementing issue,
and none carries a pending facet. `SEC-TRUST-4` was added during the interview (TLS on every boundary-2
hop) and is covered by `REQ-PLATFORM-020`, `REQ-PLATFORM-030`, and `REQ-DATA-050`, bringing the control
count to 70.

---

## Manifest

Ordinals are the topological creation order — a parent always precedes its children, and a prerequisite
always precedes its dependent. Effort is engineer-days; LOC is estimated changed human-authored lines,
excluding generated files and lockfiles. Workstream (`-000`) issues are containers and carry no estimate
of their own.

Model recommendations: **S** = Claude Sonnet 5 (`claude-sonnet-5`), **O** = Claude Opus 5
(`claude-opus-5`), **F** = Claude Fable 5 (`claude-fable-5`). Each issue body carries the
issue-specific rationale; the pattern is Opus for trust-boundary and credential logic where a subtly
permissive branch is a vulnerability, Fable only where one change spans every entity or component at
once, Sonnet elsewhere.

| # | ID | Title | Effort | LOC | Model |
| --- | --- | --- | --- | --- | --- |
| 000 | REQ-EPIC-001 | Deliver AfterImpact v1 | — | — | F |
| 001 | REQ-PLATFORM-000 | Platform, deployment, and supply chain | — | — | — |
| 002 | REQ-PLATFORM-010 | Establish the Terraform baseline and private network boundary | 1.5–2 | 500–900 | O |
| 003 | REQ-PLATFORM-020 | Provision managed PostgreSQL with KMS encryption and PITR | 0.5–1 | 150–350 | S |
| 004 | REQ-PLATFORM-030 | Provision the private S3 file store with SSE-KMS and lifecycle expiry | 0.5–1 | 150–350 | S |
| 005 | REQ-PLATFORM-040 | Separate KMS key custody from the data plane | 1–1.5 | 200–450 | O |
| 006 | REQ-PLATFORM-050 | Scope least-privilege IAM roles per component | 1–1.5 | 250–500 | O |
| 007 | REQ-PLATFORM-060 | Inject runtime secrets and keep them out of code and logs | 1–1.5 | 250–500 | O |
| 008 | REQ-PLATFORM-070 | Deploy the CloudFront and WAF edge with a pinned TLS policy | 1–1.5 | 300–600 | S |
| 009 | REQ-PLATFORM-080 | Build the CI pipeline with OIDC federation and scanners | 1.5–2 | 400–800 | S |
| 010 | REQ-PLATFORM-090 | Enforce the dependency admission and patching policy | 1–1.5 | 200–450 | S |
| 011 | REQ-PLATFORM-100 | Gate releases on the ASVS 5.0.0 Level 2 self-assessment | 1–1.5 | 300–600 | O |
| 012 | REQ-PLATFORM-110 | Configure the SES sending domain with SPF, DKIM, and DMARC | 0.5–1 | 120–300 | S |
| 013 | REQ-API-000 | REST API foundation and boundary controls | — | — | — |
| 014 | REQ-API-010 | Stand up the Quarkus API with `/v1` URL versioning | 1–1.5 | 400–800 | S |
| 015 | REQ-API-020 | Enforce owner-only authorization in a single centralized layer | 2 | 600–1200 | O |
| 016 | REQ-API-030 | Serve all functionality over TLS 1.2 or higher only | 0.5–1 | 100–250 | S |
| 017 | REQ-API-040 | Validate HTTP method and declared content type | 0.5–1 | 150–350 | S |
| 018 | REQ-API-050 | Emit the browser security header set including a strict CSP | 1–1.5 | 250–500 | O |
| 019 | REQ-API-060 | Keep CORS disabled and mark every response no-store | 0.5 | 80–200 | S |
| 020 | REQ-API-070 | Enforce request-rate and resource limits at the recorded values | 1.5–2 | 450–900 | O |
| 021 | REQ-API-080 | Bind request bodies to explicit models, rejecting unknown fields | 1–1.5 | 300–600 | O |
| 022 | REQ-API-090 | Provide the server-side validation framework | 1.5–2 | 500–1000 | S |
| 023 | REQ-API-100 | Use parameterized statements and allowlisted sort identifiers | 1–1.5 | 250–550 | O |
| 024 | REQ-API-110 | Centralize exception handling and sanitize client errors | 1–1.5 | 300–600 | O |
| 025 | REQ-API-120 | Meet the interactive latency budget under expected load | 1.5–2 | 400–800 | S |
| 026 | REQ-DATA-000 | Data persistence and protection | — | — | — |
| 027 | REQ-DATA-010 | Define the third-normal-form schema for every entity | 2 | 1000–1500 | F |
| 028 | REQ-DATA-020 | Model soft-delete state for the 30-day undo windows | 1–1.5 | 350–700 | S |
| 029 | REQ-DATA-030 | Persist document version lineage and journal entry versions | 1–1.5 | 350–700 | S |
| 030 | REQ-DATA-040 | Snapshot contact names on referencing records | 0.5–1 | 200–400 | S |
| 031 | REQ-DATA-050 | Encrypt all stored personal data at rest | 1–1.5 | 200–450 | O |
| 032 | REQ-DATA-060 | Meet the backup RPO and RTO with tested restores | 1.5–2 | 300–650 | S |
| 033 | REQ-DATA-070 | Reconcile security and privacy state after a restore | 2 | 600–1200 | O |
| 034 | REQ-DATA-080 | Limit collection to data the functional requirements need | 0.5–1 | 150–350 | S |
| 035 | REQ-DATA-090 | Keep personal data unshared and screens free of trackers | 0.5–1 | 120–300 | S |
| 036 | REQ-DATA-100 | Operate the breach-notification runbook | 1–1.5 | 250–500 | O |
| 037 | REQ-AUTH-000 | Accounts, authentication, and sessions | — | — | — |
| 038 | REQ-AUTH-010 | Terminate identity-provider ceremonies server-side | 2 | 600–1200 | O |
| 039 | REQ-AUTH-020 | Register an account and verify its email before sign-in | 1.5–2 | 500–1000 | O |
| 040 | REQ-AUTH-030 | Show the privacy policy before registration and record acceptance | 1–1.5 | 300–600 | S |
| 041 | REQ-AUTH-040 | Bootstrap the first passkey from a restricted email-code session | 2 | 550–1100 | O |
| 042 | REQ-AUTH-050 | Register a passkey and constrain stored credential material | 1.5–2 | 450–900 | O |
| 043 | REQ-AUTH-060 | Authenticate with a passkey and issue a normal session | 1.5–2 | 450–900 | O |
| 044 | REQ-AUTH-070 | Issue and transport opaque server-side session identifiers | 1.5–2 | 400–800 | O |
| 045 | REQ-AUTH-080 | Invalidate a session immediately on sign-out | 0.5–1 | 150–350 | O |
| 046 | REQ-AUTH-090 | Enforce absolute and idle session lifetimes server-side | 1–1.5 | 250–550 | O |
| 047 | REQ-AUTH-100 | Throttle failed authentication behind source-based limits | 1.5–2 | 400–850 | O |
| 048 | REQ-AUTH-110 | Respond identically whether or not an account exists | 1.5–2 | 350–750 | O |
| 049 | REQ-AUTH-120 | Recover from loss of every passkey through the verified address | 2 | 550–1100 | O |
| 050 | REQ-AUTH-130 | List and revoke credentials with a no-lockout guard | 1.5–2 | 400–800 | O |
| 051 | REQ-AUTH-140 | Prompt for a second passkey while only one is registered | 0.5–1 | 150–350 | S |
| 052 | REQ-AUTH-150 | Change the account email address with re-verification | 1.5–2 | 450–900 | O |
| 053 | REQ-AUTH-160 | Require a fresh passkey ceremony before high-impact operations | 1.5–2 | 400–850 | O |
| 054 | REQ-AUTH-170 | Send out-of-band notices for security-significant events | 1.5–2 | 400–800 | O |
| 055 | REQ-AUTH-180 | Delete an account behind re-authentication and a cancellation window | 1.5–2 | 450–900 | O |
| 056 | REQ-AUTH-190 | Complete the account purge within the stated bounds | 1.5–2 | 500–1000 | O |
| 057 | REQ-AUTH-200 | Purge inactive accounts after advance warnings | 1–1.5 | 300–650 | S |
| 058 | REQ-AUTH-210 | Store and accept no user-chosen shared secret | 0.5 | 80–200 | O |
| 059 | REQ-LOG-000 | Security event logging and activity visibility | — | — | — |
| 060 | REQ-LOG-010 | Record the security event log across every listed event type | 1.5–2 | 500–1000 | O |
| 061 | REQ-LOG-020 | Keep credentials, tokens, and health data out of logs | 1–1.5 | 250–550 | O |
| 062 | REQ-LOG-030 | Make the security event log append-only and tamper-evident | 1.5–2 | 350–750 | O |
| 063 | REQ-LOG-040 | Pseudonymize surviving log entries on account deletion | 1–1.5 | 250–550 | O |
| 064 | REQ-LOG-050 | Show the user their own recent security activity in-app | 1–1.5 | 300–600 | S |
| 065 | REQ-NOTIFY-000 | Notification feed and outbound email | — | — | — |
| 066 | REQ-NOTIFY-010 | Serve the in-app notification feed | 1–1.5 | 350–700 | S |
| 067 | REQ-NOTIFY-020 | Tie notification lifetime to its source record, case, and account | 1–1.5 | 300–600 | S |
| 068 | REQ-NOTIFY-030 | Send outbound email carrying only the minimum personal data | 1–1.5 | 300–650 | O |
| 069 | REQ-NOTIFY-040 | Surface persistent email-delivery failure as an in-app notice | 1–1.5 | 250–550 | S |
| 070 | REQ-NOTIFY-050 | Issue and validate one-time email codes under the provider rules | 1.5–2 | 350–750 | O |
| 071 | REQ-CASE-000 | Accident cases | — | — | — |
| 072 | REQ-CASE-010 | Create a case with a validated accident date and derived title | 1–1.5 | 300–600 | S |
| 073 | REQ-CASE-020 | List cases open-first, newest accident date first | 0.5–1 | 200–400 | S |
| 074 | REQ-CASE-030 | Edit any case field under the creation validation rules | 0.5–1 | 200–450 | S |
| 075 | REQ-CASE-040 | Compute the per-case overview | 1.5–2 | 450–900 | S |
| 076 | REQ-CASE-050 | Archive a case and reverse the archive without data loss | 0.5–1 | 180–400 | S |
| 077 | REQ-CASE-060 | Delete a case behind step-up and explicit confirmation | 1.5–2 | 400–850 | O |
| 078 | REQ-CASE-070 | Purge a deleted case permanently after the undo window | 1–1.5 | 300–650 | O |
| 079 | REQ-DOC-000 | Documents and paperwork | — | — | — |
| 080 | REQ-DOC-010 | Accept uploads only after server-side content inspection | 1.5–2 | 400–850 | O |
| 081 | REQ-DOC-020 | Enforce per-file size limits and the per-user storage quota | 1.5–2 | 450–900 | O |
| 082 | REQ-DOC-030 | Store originals under server-generated names with a SHA-256 hash | 1–1.5 | 300–650 | O |
| 083 | REQ-DOC-040 | Record document title, category, and optional metadata | 1–1.5 | 300–600 | S |
| 084 | REQ-DOC-050 | List a case's documents with filtering and sorting | 1–1.5 | 300–600 | S |
| 085 | REQ-DOC-060 | Search document titles, notes, and tags within a case | 1.5–2 | 350–750 | S |
| 086 | REQ-DOC-070 | Stream byte-identical downloads through authorized endpoints | 1.5–2 | 400–800 | O |
| 087 | REQ-DOC-080 | View safe, non-active types inline in a sandboxed viewer | 1.5–2 | 400–850 | O |
| 088 | REQ-DOC-090 | Edit document metadata without altering the stored file | 0.5–1 | 200–450 | S |
| 089 | REQ-DOC-100 | Replace a document file while retaining prior versions | 1.5–2 | 400–850 | O |
| 090 | REQ-DOC-110 | Trash, restore, and purge documents across the 30-day window | 1.5–2 | 400–800 | S |
| 091 | REQ-DOC-120 | Scan uploads for malware in the isolated worker and quarantine | 1.5–2 | 450–900 | O |
| 092 | REQ-DOC-130 | Meet the upload throughput floor at every accepted file size | 1–1.5 | 250–550 | S |
| 093 | REQ-HEALTH-000 | Health issues and appointments | — | — | — |
| 094 | REQ-HEALTH-010 | Record a health issue | 1–1.5 | 300–600 | S |
| 095 | REQ-HEALTH-020 | Track health issue status with a timestamped history | 1–1.5 | 300–600 | S |
| 096 | REQ-HEALTH-030 | Add timestamped progress updates in chronological order | 1–1.5 | 300–600 | S |
| 097 | REQ-HEALTH-040 | Record treatments and medications as recordkeeping only | 0.5–1 | 250–500 | S |
| 098 | REQ-HEALTH-050 | Link a health issue to providers, documents, expenses, appointments | 1–1.5 | 350–700 | S |
| 099 | REQ-HEALTH-060 | Create appointments and list upcoming ones per case | 1–1.5 | 350–700 | S |
| 100 | REQ-HEALTH-070 | Create the default 24-hour reminder on a future appointment | 0.5–1 | 200–450 | S |
| 101 | REQ-HEALTH-080 | Record appointment outcome notes after it occurs | 0.5–1 | 180–400 | S |
| 102 | REQ-HEALTH-090 | Keep health features to recordkeeping with no medical advice | 0.5 | 100–250 | S |
| 103 | REQ-HEALTH-100 | Edit and delete health records | 1–1.5 | 350–700 | S |
| 104 | REQ-TASK-000 | Recovery tasks, deadlines, and reminders | — | — | — |
| 105 | REQ-TASK-010 | Create a task | 1–1.5 | 300–600 | S |
| 106 | REQ-TASK-020 | Offer the starter recovery checklist at case creation | 1–1.5 | 300–600 | S |
| 107 | REQ-TASK-030 | Mark a task complete or not complete with a timestamp | 0.5–1 | 200–450 | S |
| 108 | REQ-TASK-040 | Edit and delete tasks | 0.5–1 | 200–450 | S |
| 109 | REQ-TASK-050 | Flag overdue tasks visibly and in the overview count | 0.5–1 | 200–450 | S |
| 110 | REQ-TASK-060 | Set one or more reminders on a task or appointment | 1–1.5 | 300–650 | S |
| 111 | REQ-TASK-070 | Deliver reminders within five minutes of fire time | 1.5–2 | 450–900 | O |
| 112 | REQ-TASK-080 | Toggle email reminders account-wide | 0.5–1 | 180–400 | O |
| 113 | REQ-TASK-090 | Escalate hard-deadline reminders with a visible countdown | 1–1.5 | 300–650 | S |
| 114 | REQ-TASK-100 | Attach the guidance-not-advice notice to guidance content | 0.5 | 120–300 | S |
| 115 | REQ-CONTACT-000 | Contacts | — | — | — |
| 116 | REQ-CONTACT-010 | Add a case-scoped contact | 1–1.5 | 300–600 | S |
| 117 | REQ-CONTACT-020 | List a case's contacts filterable by role | 0.5–1 | 180–400 | S |
| 118 | REQ-CONTACT-030 | Edit a contact | 0.5–1 | 180–400 | S |
| 119 | REQ-CONTACT-040 | Delete a contact while retaining its name on referencing records | 1–1.5 | 300–650 | S |
| 120 | REQ-EXPENSE-000 | Expenses | — | — | — |
| 121 | REQ-EXPENSE-010 | Record an expense with a validated amount and date | 1–1.5 | 350–700 | S |
| 122 | REQ-EXPENSE-020 | Edit expenses and delete them after confirmation | 0.5–1 | 200–450 | S |
| 123 | REQ-EXPENSE-030 | Compute per-case expense totals | 1–1.5 | 300–650 | S |
| 124 | REQ-EXPENSE-040 | Record partial reimbursements and show the remaining balance | 1–1.5 | 300–650 | S |
| 125 | REQ-EXPENSE-050 | Fix one currency per case and perform no conversion | 0.5–1 | 200–450 | S |
| 126 | REQ-EXPENSE-060 | Export a case's expense list as CSV with inert output | 1–1.5 | 300–650 | O |
| 127 | REQ-TIMELINE-000 | Timeline and journal | — | — | — |
| 128 | REQ-TIMELINE-010 | Write timeline events in the triggering action's transaction | 1.5–2 | 450–900 | O |
| 129 | REQ-TIMELINE-020 | Author journal entries with a backdatable entry date | 1–1.5 | 300–600 | S |
| 130 | REQ-TIMELINE-030 | Edit journal entries and retain prior versions | 1–1.5 | 350–700 | S |
| 131 | REQ-TIMELINE-040 | Delete a journal entry leaving a permanent timeline trace | 1–1.5 | 350–700 | O |
| 132 | REQ-TIMELINE-050 | Show timeline events and journal entries together with filters | 1–1.5 | 350–700 | S |
| 133 | REQ-TIMELINE-060 | Keep system-generated timeline events uneditable | 0.5–1 | 180–400 | O |
| 134 | REQ-EXPORT-000 | Export and data management | — | — | — |
| 135 | REQ-EXPORT-010 | Generate the case summary PDF within its time budget | 2 | 700–1400 | O |
| 136 | REQ-EXPORT-020 | Generate the full-account archive asynchronously | 2 | 700–1400 | F |
| 137 | REQ-EXPORT-030 | Authorize export artifact retrieval | 1–1.5 | 250–550 | O |
| 138 | REQ-EXPORT-040 | Purge export artifacts seven days after generation | 0.5–1 | 200–450 | S |
| 139 | REQ-EXPORT-050 | State export scope to the user on every export | 0.5–1 | 200–450 | S |
| 140 | REQ-EXPORT-060 | Count export artifacts against the storage quota | 1–1.5 | 250–550 | S |
| 141 | REQ-DESIGN-000 | Design language and style guide | — | — | — |
| 142 | REQ-DESIGN-010 | Define the semantic color tokens for both themes | 1–1.5 | 250–550 | S |
| 143 | REQ-DESIGN-020 | Define the type scale, spacing scale, grid, and breakpoints | 1–1.5 | 250–550 | S |
| 144 | REQ-DESIGN-030 | Specify the core component conventions | 1.5–2 | 400–800 | S |
| 145 | REQ-DESIGN-040 | Specify focus, form feedback, and destructive-action conventions | 1–1.5 | 300–600 | S |
| 146 | REQ-DESIGN-050 | Encode the accessibility rules that meet WCAG 2.2 Level AA | 1.5–2 | 350–750 | O |
| 147 | REQ-DESIGN-060 | Specify the on-screen privacy conventions | 1–1.5 | 250–550 | O |
| 148 | REQ-DESIGN-070 | Make the brand mark render with the active theme's tokens | 0.5–1 | 120–300 | S |
| 149 | REQ-DESIGN-080 | Bring `style-guide.html` into agreement with `DESIGN.md` | 1–1.5 | 300–650 | S |
| 150 | REQ-WEB-000 | Web client | — | — | — |
| 151 | REQ-WEB-010 | Build the web app shell, routing, and session handling | 1.5–2 | 500–1000 | O |
| 152 | REQ-WEB-020 | Implement the design system in React | 2 | 700–1400 | S |
| 153 | REQ-WEB-030 | Build i18n infrastructure with English and Spanish | 1.5–2 | 400–850 | S |
| 154 | REQ-WEB-040 | Build the authentication and recovery screens | 2 | 600–1200 | O |
| 155 | REQ-WEB-050 | Build the case screens | 1.5–2 | 500–1000 | S |
| 156 | REQ-WEB-060 | Build the document library and upload screens | 2 | 650–1300 | S |
| 157 | REQ-WEB-070 | Build the in-app PDF and image viewer | 1.5–2 | 400–850 | O |
| 158 | REQ-WEB-080 | Build the health screens with the severity chart | 2 | 600–1200 | S |
| 159 | REQ-WEB-090 | Build the task, deadline, and reminder screens | 1.5–2 | 550–1100 | S |
| 160 | REQ-WEB-100 | Build the contact screens | 1–1.5 | 350–700 | S |
| 161 | REQ-WEB-110 | Build the expense screens and totals | 1.5–2 | 500–1000 | S |
| 162 | REQ-WEB-120 | Build the timeline and journal screens | 1.5–2 | 550–1100 | S |
| 163 | REQ-WEB-130 | Build the export screens with the scope notice | 1–1.5 | 400–800 | S |
| 164 | REQ-WEB-140 | Build the in-app notification feed | 1–1.5 | 350–700 | S |
| 165 | REQ-WEB-150 | Build the security activity view | 1–1.5 | 300–650 | S |
| 166 | REQ-WEB-160 | Implement destructive-action confirmations | 1–1.5 | 300–650 | O |
| 167 | REQ-WEB-170 | Present errors in plain language | 1–1.5 | 300–600 | S |
| 168 | REQ-WEB-180 | Render user content as text and linkify only safe schemes | 1–1.5 | 250–550 | O |
| 169 | REQ-WEB-190 | Meet responsive and WCAG 2.2 AA conformance on web | 2 | 500–1000 | O |
| 170 | REQ-MOBILE-000 | Mobile client | — | — | — |
| 171 | REQ-MOBILE-010 | Build the mobile app shell, navigation, and session handling | 2 | 600–1200 | O |
| 172 | REQ-MOBILE-020 | Implement the design system in Compose Multiplatform | 2 | 700–1400 | S |
| 173 | REQ-MOBILE-030 | Build i18n with English and Spanish | 1.5–2 | 400–850 | S |
| 174 | REQ-MOBILE-040 | Build the authentication screens with platform passkeys | 2 | 650–1300 | O |
| 175 | REQ-MOBILE-050 | Build the case screens | 1.5–2 | 500–1000 | S |
| 176 | REQ-MOBILE-060 | Build the document library and upload screens | 2 | 650–1300 | S |
| 177 | REQ-MOBILE-070 | Build the in-app PDF and image viewer | 1.5–2 | 400–850 | O |
| 178 | REQ-MOBILE-080 | Build the health screens with the severity chart | 2 | 600–1200 | S |
| 179 | REQ-MOBILE-090 | Build the task, deadline, and reminder screens | 1.5–2 | 550–1100 | S |
| 180 | REQ-MOBILE-100 | Build the contact screens | 1–1.5 | 350–700 | S |
| 181 | REQ-MOBILE-110 | Build the expense screens and totals | 1.5–2 | 500–1000 | S |
| 182 | REQ-MOBILE-120 | Build the timeline and journal screens | 1.5–2 | 550–1100 | S |
| 183 | REQ-MOBILE-130 | Build the export screens with the scope notice | 1–1.5 | 400–800 | S |
| 184 | REQ-MOBILE-140 | Build the in-app notification feed | 1–1.5 | 350–700 | S |
| 185 | REQ-MOBILE-150 | Build the security activity view | 1–1.5 | 300–650 | S |
| 186 | REQ-MOBILE-160 | Implement destructive-action confirmations | 1–1.5 | 300–650 | O |
| 187 | REQ-MOBILE-170 | Present errors in plain language | 1–1.5 | 300–600 | S |
| 188 | REQ-MOBILE-180 | Render user content as text and linkify only safe schemes | 1–1.5 | 250–550 | O |
| 189 | REQ-MOBILE-190 | Keep session credentials only in platform secure storage | 1–1.5 | 250–550 | O |
| 190 | REQ-MOBILE-200 | Enforce the mobile platform security posture | 1.5–2 | 400–850 | O |
| 191 | REQ-MOBILE-210 | Meet accessibility and responsive conformance on mobile | 2 | 500–1000 | O |
| 192 | REQ-MOBILE-220 | Distribute through the App Store and Google Play with signing | 1–1.5 | 200–450 | S |

**Totals:** 175 leaves. Effort sums to roughly **210–275 engineer-days**; estimated changed
human-authored lines sum to roughly **63,000–128,000**. Model split: 74 Opus 5, 99 Sonnet 5, 2 Fable 5
(plus the epic). Every leaf sits inside the 0.5–2 day and ≤1,500 line bounds.

### Independent completeness cross-check

The decomposition was checked against an independently generated proposal and a completeness critic that
was asked to enumerate every requirement and control no leaf covered. Its findings and their disposition:

| Critic finding | Disposition |
| --- | --- |
| No leaf covers breach notification / the incident-response runbook (NFR-2.5, SEC-DATA-5) | Covered — `REQ-DATA-100` |
| Nothing *owns* data minimization (NFR-2.3, SEC-DATA-2) | Covered — `REQ-DATA-080` |
| No leaf provisions the SES boundary (SEC-EXT-3) or sender authentication (SEC-EXT-4) | Covered — `REQ-PLATFORM-110` |
| No leaf establishes server-side egress review (SEC-EXT-1) | Covered — `REQ-PLATFORM-090` |
| No leaf builds the outbound email adapter component | Covered — `REQ-NOTIFY-030`, `REQ-NOTIFY-040` |
| **Largest hole: no client screens for health, tasks, contacts, expenses, journal, or export** | Covered — `REQ-WEB-080`–`-130` and `REQ-MOBILE-080`–`-130` |
| No client application scaffolds | Covered — `REQ-WEB-010`, `REQ-MOBILE-010` |
| No mobile distribution / code signing | Covered — `REQ-MOBILE-220` |
| No performance/load-test harness (NFR-4.1–4.3, SEC-HTTP-6 verification) | Covered — `REQ-API-120`, `REQ-DOC-130` |
| NFR-3.1 availability has no owner | Covered — `REQ-PLATFORM-070`, `REQ-DATA-060` |
| Security messages absent from the notification feed | Covered — `REQ-NOTIFY-010` with `REQ-AUTH-170` |
| Lint/format tooling (ktlint + detekt; ESLint + Prettier) unowned | Covered — `REQ-PLATFORM-080`, per `ARCHITECTURE.md` **Technology Decisions** Lint/format row |

No critic finding required a new leaf.

---

## Creation order

The manifest ordinal **is** the topological creation order. It satisfies two rules:

1. **Parent before child** — the epic (000) precedes every workstream; each workstream (`-000`) precedes
   its own leaves, so `--parent` always resolves.
2. **Prerequisite before dependent** — platform → API → data → identity → logging and notification →
   record workstreams → design → clients. Within a workstream, foundational leaves precede the leaves
   that consume them (for example `REQ-DOC-010` upload acceptance precedes `REQ-DOC-020` size and quota,
   which precedes `REQ-DOC-030` storage naming).

The graph is acyclic. Cross-workstream edges all point backwards in ordinal order — every record
workstream depends on `REQ-API-020` (centralized authorization) and `REQ-DATA-010` (schema), both of which
precede them; both clients depend on `REQ-DESIGN-000`, which precedes them.

---

## DRAFT_ONLY — what this run did and did not do

This run was invoked with `Execution mode: DRAFT_ONLY`. Accordingly:

- **No GitHub state was changed.** No `gh issue create`, no `--parent` linking, no labels, no milestones.
  No mutating `gh` command was executed at any point.
- **No specification file was modified.** `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`,
  `DESIGN.md`, `REQUIREMENT_TEMPLATE.md`, `style-guide.html`, `logo.svg`, and `CLAUDE.md` are untouched.
- Task lists inside issue bodies use `{{ISSUE_URL:<ID>}}` placeholders, which a `CREATE` run replaces
  with real issue URLs after each issue is created.

### Before a CREATE run

`CREATE` must **stop before any GitHub change** while blocking questions remain. All 14 `PQ-*` above are
blocking, so the sequence is:

1. Answer `PQ-1` … `PQ-14` and record each resolution **in the document that owns it** — never in code
   and never in an issue body. Per `CLAUDE.md`, requirements go to `REQUIREMENTS.md`, controls and
   security/privacy requirements to `SECURITY.md`, structure to `ARCHITECTURE.md`, design language to
   `DESIGN.md` (with `style-guide.html` updated to match).
2. Re-run the decomposition so the affected bodies lose their `BLOCKED` markers and gain real acceptance
   criteria, since several currently state only what cannot yet be specified.
3. Then run the `CREATE` pre-flight. It was already run read-only on 2026-08-13 with these results:
   `gh auth status` authenticated as `jmanico` with `repo` scope; target repository `jmanico/afterimpact`
   confirmed; `--body-file` supported; **`--parent` not supported by `gh` 2.93.0**; and no existing issue
   reuses a planning ID, because the repository has no issues. Re-run it at CREATE time, and additionally
   confirm the sub-issues REST endpoint is enabled before relying on it for hierarchy.
4. Create in manifest-ordinal order, capturing each URL and substituting it for the matching
   `{{ISSUE_URL:<ID>}}` placeholder before creating the issues that reference it, then link each child to
   its parent through the sub-issues endpoint shown in **Proposed `gh` commands**.

If any step fails, stop, do not auto-delete anything already created, and record the partial state.

---

## Verification results

Re-run against all 193 draft files and the four specifications after the decision interview, 2026-08-14.

| Check | Result |
| --- | --- |
| Files produced | **193** (1 epic + 17 workstreams + 175 leaves) + `ISSUE_PLAN.md` |
| `REQUIREMENT_TEMPLATE.md` conformance | **193/193** — all 10 headings present and in template order |
| Surviving bracketed placeholders | **0** |
| Invented specification IDs | **0** — every `FR-`, `NFR-`, `SEC-`, `DEP-`, `D-`, `T-`, `OQ-`, `DQ-`, `CQ-`, `SQ-` token cited exists verbatim in its owning document |
| Requirements never cited by any issue | **0** |
| Controls never cited by any issue | **0** |
| Dangling `{{ISSUE_URL:…}}` targets | **0** |
| Live `BLOCKED` markers | **0** — every `PQ-*` is resolved; remaining `PQ-` mentions are historical resolution records |
| Unexplained `TO BE DECIDED` | **0** in the specifications and **0** in the drafts. Every surviving marker is either one of the four fields retained by decision (`D-39`, ASVS, 800-53) or prose explaining a resolution |
| Decision numbering | `D-1`…`D-58` — continuous, ordered, no gaps or duplicates |
| Specification files modified | **`REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`, `style-guide.html`** — deliberately, to record the 60 interview decisions in the documents that own them. `REQUIREMENT_TEMPLATE.md`, `logo.svg`, and `CLAUDE.md` are untouched |
| Repository working tree | those five files, plus the new untracked `.planning/` directory |
| Issues in `jmanico/afterimpact` | **none** — no issue was created at any point |
| Mutating `gh` commands executed | **none** |

On the `Regulatory` and `Jurisdiction / Regulatory Scope` fields: 85 drafts retain `TO BE DECIDED` under
`D-39`, 81 carry the standing `D-10` GDPR-grade posture, 21 carry the posture with the regulatory mapping
still pending, and 6 record a justified `N/A` — design tokens and type scales genuinely carry no
regulatory mapping, which is what `REQUIREMENT_TEMPLATE.md` reserves `N/A` for.

## Proposed `gh` commands

**None of these were executed** — this run is `DRAFT_ONLY`.

### Pre-flight results (read-only checks, run 2026-08-13)

| Check | Result |
| --- | --- |
| `gh auth status` | Authenticated as `jmanico` on github.com; token scopes include `repo` |
| `gh --version` | 2.93.0 |
| Target repository | `jmanico/afterimpact` confirmed via `git remote -v` |
| `gh issue create --body-file` | Supported (`-F, --body-file`) |
| `gh issue create --parent` | **NOT SUPPORTED** — no such flag in 2.93.0, and `gh issue edit` has none either |
| Existing issues reusing a planning ID | None — the repository has no issues at all |

### Parent linking must not use `--parent`

The earlier draft of this plan assumed `gh issue create --parent`. That flag does not exist in the installed
`gh`, so those commands would have failed. Create the issues first, then link each child to its parent through
the sub-issues REST endpoint, which takes the child's **issue id** (not its number):

```bash
# after both issues exist:
CHILD_ID=$(gh api repos/jmanico/afterimpact/issues/$CHILD_NUMBER --jq .id)
gh api --method POST repos/jmanico/afterimpact/issues/$PARENT_NUMBER/sub_issues \
  -F sub_issue_id="$CHILD_ID"
```

The CREATE pre-flight must confirm this endpoint is enabled for the repository before relying on it; if it is
not, fall back to recording the hierarchy through the `{{ISSUE_URL:<ID>}}` task lists already present in every
workstream body, which render as a checklist and require no API support.

### Creation commands, in manifest order

Create in this order so every parent exists before its children, then run the linking step above for each
child. Capture each returned URL and substitute it for the matching `{{ISSUE_URL:<ID>}}` placeholder before
creating the issues that reference it.

```bash
gh issue create --repo jmanico/afterimpact --title "[REQ-EPIC-001] Deliver AfterImpact v1" --body-file ".planning/github-issues/000-REQ-EPIC-001.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-000] Platform, deployment, and supply chain" --body-file ".planning/github-issues/001-REQ-PLATFORM-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-010] Establish the Terraform baseline and private network boundary" --body-file ".planning/github-issues/002-REQ-PLATFORM-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-020] Provision managed PostgreSQL with KMS encryption and PITR" --body-file ".planning/github-issues/003-REQ-PLATFORM-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-030] Provision the private S3 file store with SSE-KMS and lifecycle expiry" --body-file ".planning/github-issues/004-REQ-PLATFORM-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-040] Separate KMS key custody from the data plane" --body-file ".planning/github-issues/005-REQ-PLATFORM-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-050] Scope least-privilege IAM roles per component" --body-file ".planning/github-issues/006-REQ-PLATFORM-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-060] Inject runtime secrets and keep them out of code and logs" --body-file ".planning/github-issues/007-REQ-PLATFORM-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-070] Deploy the CloudFront and WAF edge with a pinned TLS policy" --body-file ".planning/github-issues/008-REQ-PLATFORM-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-080] Build the CI pipeline with OIDC federation and scanners" --body-file ".planning/github-issues/009-REQ-PLATFORM-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-090] Enforce the dependency admission and patching policy" --body-file ".planning/github-issues/010-REQ-PLATFORM-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-100] Gate releases on the ASVS 5.0.0 Level 2 self-assessment" --body-file ".planning/github-issues/011-REQ-PLATFORM-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-PLATFORM-110] Configure the SES sending domain with SPF, DKIM, and DMARC" --body-file ".planning/github-issues/012-REQ-PLATFORM-110.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-000] REST API foundation and boundary controls" --body-file ".planning/github-issues/013-REQ-API-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-010] Stand up the Quarkus API with /v1 URL versioning" --body-file ".planning/github-issues/014-REQ-API-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-020] Enforce owner-only authorization in a single centralized layer" --body-file ".planning/github-issues/015-REQ-API-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-030] Serve all functionality over TLS 1.2 or higher only" --body-file ".planning/github-issues/016-REQ-API-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-040] Validate HTTP method and declared content type" --body-file ".planning/github-issues/017-REQ-API-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-050] Emit the browser security header set including a strict CSP" --body-file ".planning/github-issues/018-REQ-API-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-060] Keep CORS disabled and mark every response no-store" --body-file ".planning/github-issues/019-REQ-API-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-070] Enforce request-rate and resource limits at the recorded values" --body-file ".planning/github-issues/020-REQ-API-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-080] Bind request bodies to explicit models, rejecting unknown fields" --body-file ".planning/github-issues/021-REQ-API-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-090] Provide the server-side validation framework" --body-file ".planning/github-issues/022-REQ-API-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-100] Use parameterized statements and allowlisted sort identifiers" --body-file ".planning/github-issues/023-REQ-API-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-110] Centralize exception handling and sanitize client errors" --body-file ".planning/github-issues/024-REQ-API-110.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-API-120] Meet the interactive latency budget under expected load" --body-file ".planning/github-issues/025-REQ-API-120.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-000] Data persistence and protection" --body-file ".planning/github-issues/026-REQ-DATA-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-010] Define the third-normal-form schema for every entity" --body-file ".planning/github-issues/027-REQ-DATA-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-020] Model soft-delete state for the 30-day undo windows" --body-file ".planning/github-issues/028-REQ-DATA-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-030] Persist document version lineage and journal entry versions" --body-file ".planning/github-issues/029-REQ-DATA-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-040] Snapshot contact names on referencing records" --body-file ".planning/github-issues/030-REQ-DATA-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-050] Encrypt all stored personal data at rest" --body-file ".planning/github-issues/031-REQ-DATA-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-060] Meet the backup RPO and RTO with tested restores" --body-file ".planning/github-issues/032-REQ-DATA-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-070] Reconcile security and privacy state after a restore" --body-file ".planning/github-issues/033-REQ-DATA-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-080] Limit collection to data the functional requirements need" --body-file ".planning/github-issues/034-REQ-DATA-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-090] Keep personal data unshared and screens free of trackers" --body-file ".planning/github-issues/035-REQ-DATA-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DATA-100] Operate the breach-notification runbook" --body-file ".planning/github-issues/036-REQ-DATA-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-000] Accounts, authentication, and sessions" --body-file ".planning/github-issues/037-REQ-AUTH-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-010] Terminate identity-provider ceremonies server-side" --body-file ".planning/github-issues/038-REQ-AUTH-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-020] Register an account and verify its email before sign-in" --body-file ".planning/github-issues/039-REQ-AUTH-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-030] Show the privacy policy before registration and record acceptance" --body-file ".planning/github-issues/040-REQ-AUTH-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-040] Bootstrap the first passkey from a restricted email-code session" --body-file ".planning/github-issues/041-REQ-AUTH-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-050] Register a passkey and constrain stored credential material" --body-file ".planning/github-issues/042-REQ-AUTH-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-060] Authenticate with a passkey and issue a normal session" --body-file ".planning/github-issues/043-REQ-AUTH-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-070] Issue and transport opaque server-side session identifiers" --body-file ".planning/github-issues/044-REQ-AUTH-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-080] Invalidate a session immediately on sign-out" --body-file ".planning/github-issues/045-REQ-AUTH-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-090] Enforce absolute and idle session lifetimes server-side" --body-file ".planning/github-issues/046-REQ-AUTH-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-100] Throttle failed authentication behind source-based limits" --body-file ".planning/github-issues/047-REQ-AUTH-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-110] Respond identically whether or not an account exists" --body-file ".planning/github-issues/048-REQ-AUTH-110.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-120] Recover from loss of every passkey through the verified address" --body-file ".planning/github-issues/049-REQ-AUTH-120.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-130] List and revoke credentials with a no-lockout guard" --body-file ".planning/github-issues/050-REQ-AUTH-130.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-140] Prompt for a second passkey while only one is registered" --body-file ".planning/github-issues/051-REQ-AUTH-140.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-150] Change the account email address with re-verification" --body-file ".planning/github-issues/052-REQ-AUTH-150.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-160] Require a fresh passkey ceremony before high-impact operations" --body-file ".planning/github-issues/053-REQ-AUTH-160.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-170] Send out-of-band notices for security-significant events" --body-file ".planning/github-issues/054-REQ-AUTH-170.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-180] Delete an account behind re-authentication and a cancellation window" --body-file ".planning/github-issues/055-REQ-AUTH-180.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-190] Complete the account purge within the stated bounds" --body-file ".planning/github-issues/056-REQ-AUTH-190.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-200] Purge inactive accounts after advance warnings" --body-file ".planning/github-issues/057-REQ-AUTH-200.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-AUTH-210] Store and accept no user-chosen shared secret" --body-file ".planning/github-issues/058-REQ-AUTH-210.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-LOG-000] Security event logging and activity visibility" --body-file ".planning/github-issues/059-REQ-LOG-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-LOG-010] Record the security event log across every listed event type" --body-file ".planning/github-issues/060-REQ-LOG-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-LOG-020] Keep credentials, tokens, and health data out of logs" --body-file ".planning/github-issues/061-REQ-LOG-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-LOG-030] Make the security event log append-only and tamper-evident" --body-file ".planning/github-issues/062-REQ-LOG-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-LOG-040] Pseudonymize surviving log entries on account deletion" --body-file ".planning/github-issues/063-REQ-LOG-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-LOG-050] Show the user their own recent security activity in-app" --body-file ".planning/github-issues/064-REQ-LOG-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-NOTIFY-000] Notification feed and outbound email" --body-file ".planning/github-issues/065-REQ-NOTIFY-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-NOTIFY-010] Serve the in-app notification feed" --body-file ".planning/github-issues/066-REQ-NOTIFY-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-NOTIFY-020] Tie notification lifetime to its source record, case, and account" --body-file ".planning/github-issues/067-REQ-NOTIFY-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-NOTIFY-030] Send outbound email carrying only the minimum personal data" --body-file ".planning/github-issues/068-REQ-NOTIFY-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-NOTIFY-040] Surface persistent email-delivery failure as an in-app notice" --body-file ".planning/github-issues/069-REQ-NOTIFY-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-NOTIFY-050] Issue and validate one-time email codes under the provider rules" --body-file ".planning/github-issues/070-REQ-NOTIFY-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-000] Accident cases" --body-file ".planning/github-issues/071-REQ-CASE-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-010] Create a case with a validated accident date and derived title" --body-file ".planning/github-issues/072-REQ-CASE-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-020] List cases open-first, newest accident date first" --body-file ".planning/github-issues/073-REQ-CASE-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-030] Edit any case field under the creation validation rules" --body-file ".planning/github-issues/074-REQ-CASE-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-040] Compute the per-case overview" --body-file ".planning/github-issues/075-REQ-CASE-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-050] Archive a case and reverse the archive without data loss" --body-file ".planning/github-issues/076-REQ-CASE-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-060] Delete a case behind step-up and explicit confirmation" --body-file ".planning/github-issues/077-REQ-CASE-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CASE-070] Purge a deleted case permanently after the undo window" --body-file ".planning/github-issues/078-REQ-CASE-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-000] Documents and paperwork" --body-file ".planning/github-issues/079-REQ-DOC-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-010] Accept uploads only after server-side content inspection" --body-file ".planning/github-issues/080-REQ-DOC-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-020] Enforce per-file size limits and the per-user storage quota" --body-file ".planning/github-issues/081-REQ-DOC-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-030] Store originals under server-generated names with a SHA-256 hash" --body-file ".planning/github-issues/082-REQ-DOC-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-040] Record document title, category, and optional metadata" --body-file ".planning/github-issues/083-REQ-DOC-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-050] List a case's documents with filtering and sorting" --body-file ".planning/github-issues/084-REQ-DOC-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-060] Search document titles, notes, and tags within a case" --body-file ".planning/github-issues/085-REQ-DOC-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-070] Stream byte-identical downloads through authorized endpoints" --body-file ".planning/github-issues/086-REQ-DOC-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-080] View safe, non-active types inline in a sandboxed viewer" --body-file ".planning/github-issues/087-REQ-DOC-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-090] Edit document metadata without altering the stored file" --body-file ".planning/github-issues/088-REQ-DOC-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-100] Replace a document file while retaining prior versions" --body-file ".planning/github-issues/089-REQ-DOC-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-110] Trash, restore, and purge documents across the 30-day window" --body-file ".planning/github-issues/090-REQ-DOC-110.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-120] Scan uploads for malware in the isolated worker and quarantine" --body-file ".planning/github-issues/091-REQ-DOC-120.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DOC-130] Meet the upload throughput floor at every accepted file size" --body-file ".planning/github-issues/092-REQ-DOC-130.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-000] Health issues and appointments" --body-file ".planning/github-issues/093-REQ-HEALTH-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-010] Record a health issue" --body-file ".planning/github-issues/094-REQ-HEALTH-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-020] Track health issue status with a timestamped history" --body-file ".planning/github-issues/095-REQ-HEALTH-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-030] Add timestamped progress updates in chronological order" --body-file ".planning/github-issues/096-REQ-HEALTH-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-040] Record treatments and medications as recordkeeping only" --body-file ".planning/github-issues/097-REQ-HEALTH-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-050] Link a health issue to providers, documents, expenses, appointments" --body-file ".planning/github-issues/098-REQ-HEALTH-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-060] Create appointments and list upcoming ones per case" --body-file ".planning/github-issues/099-REQ-HEALTH-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-070] Create the default 24-hour reminder on a future appointment" --body-file ".planning/github-issues/100-REQ-HEALTH-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-080] Record appointment outcome notes after it occurs" --body-file ".planning/github-issues/101-REQ-HEALTH-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-090] Keep health features to recordkeeping with no medical advice" --body-file ".planning/github-issues/102-REQ-HEALTH-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-HEALTH-100] Edit and delete health records" --body-file ".planning/github-issues/103-REQ-HEALTH-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-000] Recovery tasks, deadlines, and reminders" --body-file ".planning/github-issues/104-REQ-TASK-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-010] Create a task" --body-file ".planning/github-issues/105-REQ-TASK-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-020] Offer the starter recovery checklist at case creation" --body-file ".planning/github-issues/106-REQ-TASK-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-030] Mark a task complete or not complete with a timestamp" --body-file ".planning/github-issues/107-REQ-TASK-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-040] Edit and delete tasks" --body-file ".planning/github-issues/108-REQ-TASK-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-050] Flag overdue tasks visibly and in the overview count" --body-file ".planning/github-issues/109-REQ-TASK-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-060] Set one or more reminders on a task or appointment" --body-file ".planning/github-issues/110-REQ-TASK-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-070] Deliver reminders within five minutes of fire time" --body-file ".planning/github-issues/111-REQ-TASK-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-080] Toggle email reminders account-wide" --body-file ".planning/github-issues/112-REQ-TASK-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-090] Escalate hard-deadline reminders with a visible countdown" --body-file ".planning/github-issues/113-REQ-TASK-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TASK-100] Attach the guidance-not-advice notice to guidance content" --body-file ".planning/github-issues/114-REQ-TASK-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CONTACT-000] Contacts" --body-file ".planning/github-issues/115-REQ-CONTACT-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CONTACT-010] Add a case-scoped contact" --body-file ".planning/github-issues/116-REQ-CONTACT-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CONTACT-020] List a case's contacts filterable by role" --body-file ".planning/github-issues/117-REQ-CONTACT-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CONTACT-030] Edit a contact" --body-file ".planning/github-issues/118-REQ-CONTACT-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-CONTACT-040] Delete a contact while retaining its name on referencing records" --body-file ".planning/github-issues/119-REQ-CONTACT-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-000] Expenses" --body-file ".planning/github-issues/120-REQ-EXPENSE-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-010] Record an expense with a validated amount and date" --body-file ".planning/github-issues/121-REQ-EXPENSE-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-020] Edit expenses and delete them after confirmation" --body-file ".planning/github-issues/122-REQ-EXPENSE-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-030] Compute per-case expense totals" --body-file ".planning/github-issues/123-REQ-EXPENSE-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-040] Record partial reimbursements and show the remaining balance" --body-file ".planning/github-issues/124-REQ-EXPENSE-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-050] Fix one currency per case and perform no conversion" --body-file ".planning/github-issues/125-REQ-EXPENSE-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPENSE-060] Export a case's expense list as CSV with inert output" --body-file ".planning/github-issues/126-REQ-EXPENSE-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-000] Timeline and journal" --body-file ".planning/github-issues/127-REQ-TIMELINE-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-010] Write timeline events in the triggering action's transaction" --body-file ".planning/github-issues/128-REQ-TIMELINE-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-020] Author journal entries with a backdatable entry date" --body-file ".planning/github-issues/129-REQ-TIMELINE-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-030] Edit journal entries and retain prior versions" --body-file ".planning/github-issues/130-REQ-TIMELINE-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-040] Delete a journal entry leaving a permanent timeline trace" --body-file ".planning/github-issues/131-REQ-TIMELINE-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-050] Show timeline events and journal entries together with filters" --body-file ".planning/github-issues/132-REQ-TIMELINE-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-TIMELINE-060] Keep system-generated timeline events uneditable" --body-file ".planning/github-issues/133-REQ-TIMELINE-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-000] Export and data management" --body-file ".planning/github-issues/134-REQ-EXPORT-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-010] Generate the case summary PDF within its time budget" --body-file ".planning/github-issues/135-REQ-EXPORT-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-020] Generate the full-account archive asynchronously" --body-file ".planning/github-issues/136-REQ-EXPORT-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-030] Authorize export artifact retrieval" --body-file ".planning/github-issues/137-REQ-EXPORT-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-040] Purge export artifacts seven days after generation" --body-file ".planning/github-issues/138-REQ-EXPORT-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-050] State export scope to the user on every export" --body-file ".planning/github-issues/139-REQ-EXPORT-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-EXPORT-060] Count export artifacts against the storage quota" --body-file ".planning/github-issues/140-REQ-EXPORT-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-000] Design language and style guide" --body-file ".planning/github-issues/141-REQ-DESIGN-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-010] Define the semantic color tokens for both themes" --body-file ".planning/github-issues/142-REQ-DESIGN-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-020] Define the type scale, spacing scale, grid, and breakpoints" --body-file ".planning/github-issues/143-REQ-DESIGN-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-030] Specify the core component conventions" --body-file ".planning/github-issues/144-REQ-DESIGN-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-040] Specify focus, form feedback, and destructive-action conventions" --body-file ".planning/github-issues/145-REQ-DESIGN-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-050] Encode the accessibility rules that meet WCAG 2.2 Level AA" --body-file ".planning/github-issues/146-REQ-DESIGN-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-060] Specify the on-screen privacy conventions" --body-file ".planning/github-issues/147-REQ-DESIGN-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-070] Make the brand mark render with the active theme's tokens" --body-file ".planning/github-issues/148-REQ-DESIGN-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-DESIGN-080] Bring style-guide.html into agreement with DESIGN.md" --body-file ".planning/github-issues/149-REQ-DESIGN-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-000] Web client" --body-file ".planning/github-issues/150-REQ-WEB-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-010] Build the web app shell, routing, and session handling" --body-file ".planning/github-issues/151-REQ-WEB-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-020] Implement the design system in React" --body-file ".planning/github-issues/152-REQ-WEB-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-030] Build i18n infrastructure with English and Spanish" --body-file ".planning/github-issues/153-REQ-WEB-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-040] Build the authentication and recovery screens" --body-file ".planning/github-issues/154-REQ-WEB-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-050] Build the case screens" --body-file ".planning/github-issues/155-REQ-WEB-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-060] Build the document library and upload screens" --body-file ".planning/github-issues/156-REQ-WEB-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-070] Build the in-app PDF and image viewer" --body-file ".planning/github-issues/157-REQ-WEB-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-080] Build the health screens with the severity chart" --body-file ".planning/github-issues/158-REQ-WEB-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-090] Build the task, deadline, and reminder screens" --body-file ".planning/github-issues/159-REQ-WEB-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-100] Build the contact screens" --body-file ".planning/github-issues/160-REQ-WEB-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-110] Build the expense screens and totals" --body-file ".planning/github-issues/161-REQ-WEB-110.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-120] Build the timeline and journal screens" --body-file ".planning/github-issues/162-REQ-WEB-120.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-130] Build the export screens with the scope notice" --body-file ".planning/github-issues/163-REQ-WEB-130.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-140] Build the in-app notification feed" --body-file ".planning/github-issues/164-REQ-WEB-140.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-150] Build the security activity view" --body-file ".planning/github-issues/165-REQ-WEB-150.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-160] Implement destructive-action confirmations" --body-file ".planning/github-issues/166-REQ-WEB-160.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-170] Present errors in plain language" --body-file ".planning/github-issues/167-REQ-WEB-170.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-180] Render user content as text and linkify only safe schemes" --body-file ".planning/github-issues/168-REQ-WEB-180.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-WEB-190] Meet responsive and WCAG 2.2 AA conformance on web" --body-file ".planning/github-issues/169-REQ-WEB-190.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-000] Mobile client" --body-file ".planning/github-issues/170-REQ-MOBILE-000.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-010] Build the mobile app shell, navigation, and session handling" --body-file ".planning/github-issues/171-REQ-MOBILE-010.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-020] Implement the design system in Compose Multiplatform" --body-file ".planning/github-issues/172-REQ-MOBILE-020.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-030] Build i18n with English and Spanish" --body-file ".planning/github-issues/173-REQ-MOBILE-030.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-040] Build the authentication screens with platform passkeys" --body-file ".planning/github-issues/174-REQ-MOBILE-040.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-050] Build the case screens" --body-file ".planning/github-issues/175-REQ-MOBILE-050.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-060] Build the document library and upload screens" --body-file ".planning/github-issues/176-REQ-MOBILE-060.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-070] Build the in-app PDF and image viewer" --body-file ".planning/github-issues/177-REQ-MOBILE-070.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-080] Build the health screens with the severity chart" --body-file ".planning/github-issues/178-REQ-MOBILE-080.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-090] Build the task, deadline, and reminder screens" --body-file ".planning/github-issues/179-REQ-MOBILE-090.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-100] Build the contact screens" --body-file ".planning/github-issues/180-REQ-MOBILE-100.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-110] Build the expense screens and totals" --body-file ".planning/github-issues/181-REQ-MOBILE-110.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-120] Build the timeline and journal screens" --body-file ".planning/github-issues/182-REQ-MOBILE-120.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-130] Build the export screens with the scope notice" --body-file ".planning/github-issues/183-REQ-MOBILE-130.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-140] Build the in-app notification feed" --body-file ".planning/github-issues/184-REQ-MOBILE-140.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-150] Build the security activity view" --body-file ".planning/github-issues/185-REQ-MOBILE-150.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-160] Implement destructive-action confirmations" --body-file ".planning/github-issues/186-REQ-MOBILE-160.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-170] Present errors in plain language" --body-file ".planning/github-issues/187-REQ-MOBILE-170.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-180] Render user content as text and linkify only safe schemes" --body-file ".planning/github-issues/188-REQ-MOBILE-180.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-190] Keep session credentials only in platform secure storage" --body-file ".planning/github-issues/189-REQ-MOBILE-190.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-200] Enforce the mobile platform security posture" --body-file ".planning/github-issues/190-REQ-MOBILE-200.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-210] Meet accessibility and responsive conformance on mobile" --body-file ".planning/github-issues/191-REQ-MOBILE-210.md"
gh issue create --repo jmanico/afterimpact --title "[REQ-MOBILE-220] Distribute through the App Store and Google Play with signing" --body-file ".planning/github-issues/192-REQ-MOBILE-220.md"
```
