# AfterImpact

A car accident does not end when the vehicles are towed. What follows is months of
paperwork, medical appointments, insurance deadlines, receipts, and phone calls — arriving
while the person handling them is injured, and arriving in an order nobody chose.

AfterImpact is a web and mobile application for managing that aftermath. One person, their
own accident, in one place: the documents, the health issues, the recovery tasks and
deadlines, the contacts, the expenses, and a journal of what happened when — plus exports
built for the people who ask for records, namely insurers, attorneys, and providers.

> **Status: specification only.** This repository currently contains no implementation.
> The specifications are complete and every open question in them has been resolved; the
> build is decomposed into [193 GitHub issues](https://github.com/jmanico/afterimpact/issues)
> under a single epic, ready to start.

## What it does

| Capability | Covers |
| --- | --- |
| Accident cases | The accident record everything else hangs from |
| Documents & paperwork | Uploads, malware scanning, in-app viewing, search, storage quota |
| Health issues & appointments | Injuries, treatments, appointments, progress over time |
| Recovery tasks & deadlines | Tasks, due dates, reminders, notifications |
| Contacts | Adjusters, attorneys, providers, and who is linked to what |
| Expenses | Costs, payment status, reimbursement tracking |
| Timeline & journal | A dated record of the whole episode |
| Export & data management | Full-archive export, account deletion, inactivity purge |

`REQUIREMENTS.md` is authoritative for all of it.

## Two things that shape every decision here

**It is single-user.** The accident victim is the only actor. There are no roles, no
sharing, no collaborators, no admin console. Any feature that implies a second user is out
of scope, and authorization reduces to a single question: does this record belong to the
caller?

**Every record counts as health data.** Because the content includes injuries and
treatment, the whole dataset is treated as sensitive personal data rather than sorting
records into sensitive and non-sensitive tiers. That decision is why the security posture
is heavier than the feature count suggests: passkey-only authentication, encryption with
customer-managed keys, EU-only data residency, on-screen masking, and a GDPR-grade
privacy bar applied globally rather than per-jurisdiction.

## Repository layout

Each document owns its domain. Facts are stated once, in the document that owns them, and
referenced everywhere else — so if two places disagree, the owning document wins.

| Document | Owns |
| --- | --- |
| [`REQUIREMENTS.md`](REQUIREMENTS.md) | What the system does — functional requirements, data entities, product decisions |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | Components, boundaries, data flow, technology decisions |
| [`SECURITY.md`](SECURITY.md) | Security and privacy requirements, controls, threat model |
| [`DESIGN.md`](DESIGN.md) | The design language — tokens, scales, components, accessibility |
| [`style-guide.html`](style-guide.html) | The rendered reference for `DESIGN.md`; the two must agree |
| [`REQUIREMENT_TEMPLATE.md`](REQUIREMENT_TEMPLATE.md) | The structure every issue must follow |
| [`CLAUDE.md`](CLAUDE.md) | Contributor rules, and the authoritative statement of the above |

## Technology

Decided and recorded in `ARCHITECTURE.md`, which is the only place to change them:

- **API** — Quarkus / Kotlin, REST, versioned at `/v1`
- **Web** — React + TypeScript · **Mobile** — Compose Multiplatform (iOS + Android)
- **Data** — Amazon RDS for PostgreSQL (Multi-AZ, 3NF schema, Flyway migrations)
- **Files** — S3 with SSE-KMS, scanned with ClamAV, type-inspected with Apache Tika
- **Identity** — Amazon Cognito, restricted to WebAuthn passkeys and email OTP
- **Infrastructure** — ECS on Fargate in `eu-west-1`, CloudFront + WAF, Terraform,
  GitHub Actions with AWS OIDC federation

## The work breakdown

The build is decomposed into one epic and 192 sub-issues, each written against
`REQUIREMENT_TEMPLATE.md` and independently testable — implementation, tests, error
handling, security, and accessibility all land in the same issue.

**Epic: [#12 — Deliver AfterImpact v1](https://github.com/jmanico/afterimpact/issues/12)**

| Workstream | Issues | | Workstream | Issues |
| --- | --- | --- | --- | --- |
| [Platform & supply chain](https://github.com/jmanico/afterimpact/issues/13) | 11 | | [Contacts](https://github.com/jmanico/afterimpact/issues/127) | 4 |
| [REST API foundation](https://github.com/jmanico/afterimpact/issues/25) | 12 | | [Expenses](https://github.com/jmanico/afterimpact/issues/132) | 6 |
| [Data persistence](https://github.com/jmanico/afterimpact/issues/38) | 10 | | [Timeline & journal](https://github.com/jmanico/afterimpact/issues/139) | 6 |
| [Auth & sessions](https://github.com/jmanico/afterimpact/issues/49) | 21 | | [Export & data management](https://github.com/jmanico/afterimpact/issues/146) | 6 |
| [Security logging](https://github.com/jmanico/afterimpact/issues/71) | 5 | | [Design language](https://github.com/jmanico/afterimpact/issues/153) | 8 |
| [Notifications](https://github.com/jmanico/afterimpact/issues/77) | 5 | | [Web client](https://github.com/jmanico/afterimpact/issues/162) | 19 |
| [Accident cases](https://github.com/jmanico/afterimpact/issues/83) | 7 | | [Mobile client](https://github.com/jmanico/afterimpact/issues/182) | 22 |
| [Documents](https://github.com/jmanico/afterimpact/issues/91) | 13 | | [Health issues](https://github.com/jmanico/afterimpact/issues/105) | 10 |
| [Recovery tasks](https://github.com/jmanico/afterimpact/issues/116) | 10 | | | |

`.planning/github-issues/ISSUE_PLAN.md` records how the decomposition was derived and
carries the coverage matrices — every requirement and every security control traced to the
issue that implements it. `.planning/github-issues/ISSUE_MAP.md` maps each planning ID to
its issue number. **The GitHub issues are the live record**; the planning files are a
point-in-time snapshot and are not kept in sync.

## Contributing

`CLAUDE.md` states the rules in full. In short:

- No direct pushes to `main`. Work on a kebab-case topic branch and land it via a pull
  request with CI green. Commit subjects are imperative mood, 72 characters or fewer.
- Read the owning document before changing anything in its domain, and record any decision
  you reach in that document. Code must never be the only record of a behavior or
  technology choice.
- Every new issue follows `REQUIREMENT_TEMPLATE.md`.
- `TO BE DECIDED`, `UNKNOWN`, and `ASSUMPTION` are defined terms with meanings set by
  `ARCHITECTURE.md`. Never quietly replace one with a guess — where the specs are silent,
  say so rather than inventing a requirement.
