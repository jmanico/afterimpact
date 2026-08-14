# AfterImpact

AfterImpact helps a person manage the aftermath of a car accident: paperwork, accident-related health issues, recovery tasks and deadlines, contacts, expenses, and a journal/timeline, plus exports for insurers, attorneys, and providers. It is single-user — the accident victim is the only actor — and every record it holds counts as sensitive personal data because it includes health information.

The repository currently holds specifications only; no implementation exists yet.

## Where knowledge lives

Read the owning document before changing anything in its domain. Each fact and rule lives in exactly one document — reference it, never copy it somewhere else.

| Document | Owns | Read it before |
| --- | --- | --- |
| `REQUIREMENTS.md` | WHAT the system does — `FR-*`, non-functional `NFR-3`–`NFR-6`, decisions `D-*`, open questions `OQ-*` | changing observable behavior |
| `DESIGN.md` | The design language — tokens, type and spacing scales, components, accessibility; `style-guide.html` is its rendered reference; open questions `DQ-*` | UI work |
| `ARCHITECTURE.md` | Components, boundaries, data flow, technology decisions, marker definitions; cross-document questions `CQ-*` | structural changes |
| `SECURITY.md` | The security posture — security and privacy requirements `NFR-1.*`/`NFR-2.*`, controls `SEC-*`, dependency rules `DEP-*`, threat model, open questions `SQ-*` | touching auth, input handling, data protection, or any trust boundary |
| `REQUIREMENT_TEMPLATE.md` | The required structure of a requirement | opening a GitHub issue |

All security and privacy requirements are authored in `SECURITY.md`, stated on the control that enforces them. `REQUIREMENTS.md` does not restate them.

## Rules

- Every new GitHub issue MUST follow `REQUIREMENT_TEMPLATE.md`, so each issue is a structured, independently testable requirement.
- `TO BE DECIDED`, `UNKNOWN`, and `ASSUMPTION` carry the meanings `ARCHITECTURE.md` defines. Never quietly replace one with a guess. Where the specs are silent, say so rather than inventing a requirement, control, or decision.
- The specs record open questions and unresolved conflicts by ID (`OQ-*`, `DQ-*`, `CQ-*`, `SQ-*`). Do not settle one in code — get the decision made, then record it in the document that owns it.
- Any decision reached while implementing goes into its owning document. Code MUST NOT be the only record of a behavior or technology choice.
- `style-guide.html` must agree with `DESIGN.md`: changing a token, scale, or component convention means changing both.
- IDs are the cross-document reference mechanism and appear in traceability tables. Keep existing IDs stable; when a spec changes, update what points at it.

## Workflow

- Build, test, and run commands: deferred until the first implementation PR lands — no implementation or build tooling exists yet. Record them here in that PR.
- CI/CD, linting, and formatting: the platform and tooling choices are owned by `ARCHITECTURE.md`'s Technology Decisions (CI/CD and Lint/format rows); the pipeline's security posture (secret handling, scanning) is owned by `SECURITY.md` (SEC-DEPLOY-3, SEC-SECRET-1 — the `SQ-6` resolution).
- Branch, commit, and pull-request conventions: no direct pushes to `main`; work on kebab-case topic branches; every change lands via a pull request with CI green; commit subjects are imperative mood, 72 characters or fewer; squash-merge is optional.
