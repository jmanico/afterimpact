# AfterImpact — Design

This document owns the visual and frontend design language: brand and logo, color palette, typography, layout, component conventions, and the design rules that meet the accessibility target. `style-guide.html` is its rendered reference and must agree with it. `REQUIREMENTS.md` owns WHAT the system does; `ARCHITECTURE.md` owns how it is built; `SECURITY.md` owns the security posture.

The audience is a person recovering from a recent car accident, who may be injured, stressed, or medicated. The clients are a web application and a mobile application; `ARCHITECTURE.md` owns the client technology choices.

## Brand and Logo

AfterImpact is the product name, and its brand personality is calm, competent, and reassuring — a steady hand after a bad day (resolves DQ-1). Write in plain language and avoid urgency theatrics: the interface's job is to reduce stress, never to demand attention. There are no brand assets beyond `logo.svg`; it is the only mark (resolves DQ-4).

`logo.svg` is the primary mark. Its two offset, rounded paths bracket a central transition: the first path ends and the second continues, an abstract reference to moving forward after a disruptive event. The geometry is intentionally simple enough to hold at small sizes. The mark uses the `primary` and `secondary` tokens only and sits on `background` and `surface`; in each theme its paths render with that theme's token values.

Maintain clear space equal to one eighth of the mark's displayed width on all sides. Do not display it below 24 CSS px square. Keep its aspect ratio and token colors; do not rotate it, add effects, place it on low-contrast or visually busy backgrounds, or rearrange its paths. The product name typeset in the primary UI family serves as the interim wordmark; a designed wordmark or lockup is deliberately post-v1 work (resolves DQ-1).

## Color Palette

Both a light and a dark theme ship (resolves DQ-3). Clients follow the operating-system color-scheme preference by default and offer a manual override. Tokens are semantic and paired one-to-one across themes: each token keeps its name and role and takes its value from the active theme's table below, so components reference tokens, never raw hex values. Filled controls (`primary`, `secondary`, `error`, and `success` fills) use `surface` label ink in the light theme and `background` label ink in the dark theme, per the documented pairs.

Light theme:

| Token | Hex | Intended use |
| --- | --- | --- |
| `primary` | `#234E70` | Primary actions, links, focus rings, and the first logo path |
| `secondary` | `#0F766E` | Secondary emphasis and the second logo path |
| `background` | `#F6F8FA` | Page background |
| `surface` | `#FFFFFF` | Cards, fields, and raised content areas |
| `text` | `#17202A` | Body text and headings |
| `error` | `#B42318` | Error text, borders, and destructive-status emphasis |
| `success` | `#146C43` | Success text, borders, and confirmation emphasis |

Approved light-theme text/background pairs and WCAG contrast ratios are: `text` on `background` 15.45:1; `text` on `surface` 16.45:1; white on `primary` 8.77:1; white on `secondary` 5.47:1; white on `error` 6.57:1; white on `success` 6.45:1; and `primary` links on `background` / `surface` 8.24:1 / 8.77:1. Use only these documented pairings for text.

Dark theme:

| Token | Hex | Intended use |
| --- | --- | --- |
| `primary` | `#8FB8DC` | Primary actions, links, focus rings, and the first logo path |
| `secondary` | `#5CC8BE` | Secondary emphasis and the second logo path |
| `background` | `#0F1720` | Page background |
| `surface` | `#1A2530` | Cards, fields, and raised content areas |
| `text` | `#E7EDF3` | Body text and headings |
| `error` | `#F5928A` | Error text, borders, and destructive-status emphasis |
| `success` | `#6FCF97` | Success text, borders, and confirmation emphasis |

Approved dark-theme text/background pairs and WCAG contrast ratios are: `text` on `background` 15.30:1; `text` on `surface` 13.18:1; `background` ink on `primary` 8.64:1; `background` ink on `secondary` 8.99:1; `background` ink on `error` 8.04:1; `background` ink on `success` 9.49:1; `primary` links on `background` / `surface` 8.64:1 / 7.44:1; and `error` / `success` text on `background` 8.04:1 / 9.49:1 and on `surface` 6.93:1 / 8.18:1. Use only these documented pairings for text.

## Typography

The primary family is the system font stack — `system-ui`, with `-apple-system`, `BlinkMacSystemFont`, `"Segoe UI"`, and `sans-serif` fallbacks — and this direction is final (resolves DQ-5). It is neutral, readable, requires no network or font license dependency, and follows the user's platform conventions.

Use weights 400 (body), 600 (controls and small headings), and 700 (major headings). The type scale is `xs` 0.75rem/1rem, `sm` 0.875rem/1.25rem, `base` 1rem/1.5rem, `lg` 1.25rem/1.75rem, `xl` 1.5rem/2rem, `2xl` 2rem/2.5rem, and `3xl` 2.5rem/3rem. Keep body copy at `base` or larger.

## Layout and Spacing

Use a 4px base spacing scale: `0`, `1` 4px, `2` 8px, `3` 12px, `4` 16px, `6` 24px, `8` 32px, `12` 48px, and `16` 64px. Prefer a centered content area no wider than 72rem and readable text lines near 68 characters. Assume a fluid four-column grid at narrow widths and a twelve-column grid where space permits; components span the columns their content needs.

Responsive breakpoints are `sm` 30rem, `md` 48rem, `lg` 64rem, and `xl` 80rem. Layouts must reflow rather than depend on a particular device size, and must remain usable from a 360 px viewport up (NFR-5.2).

The product ships localized (NFR-5.4 in `REQUIREMENTS.md` owns the locales). Design components to absorb roughly 30% text expansion relative to English: avoid fixed-width labels, and never depend on truncation to make a layout work.

## Components

- **Buttons:** Use a filled `primary` treatment for the single most important action in a group and a `surface` treatment with a `primary` border for alternatives. Labels should describe the action. Preserve a minimum 44px control height, clear hover feedback, and the shared focus treatment. Do not communicate state through color alone.
- **Inputs:** Keep a persistent visible label above each field. Use `surface`, `text`, and a visible border; help text may follow the field. A placeholder never replaces a label. Invalid fields use an `error` border plus adjacent, programmatically associated text that explains how to recover.
- **Links:** Use `primary` and underline links in prose. Hover may strengthen the underline; visited behavior must remain distinguishable without reducing contrast. User-supplied values become links only as SEC-OUT-3 permits; otherwise render them as ordinary text.
- **User-authored content:** v1 content is plain text as SEC-OUT-1 specifies. Preserve line breaks where meaningful, but do not present Markdown or rich-text affordances that imply formatting will be interpreted.
- **Authentication and recovery:** Email-code screens MUST explain that the code creates only a restricted setup/recovery state and that the user must register a passkey before account data becomes available (FR-1.2, FR-1.6, SEC-AUTHN-6). Show the security-owned expiry in plain language, offer a clearly labeled resend action, and avoid countdown theatrics or urgency styling.
- **Focus states:** Every interactive element receives a 3px `primary` focus ring with a 2px `surface` offset on `:focus-visible`. Do not remove focus indication.
- **Form feedback:** Place feedback close to the relevant field or action. Prefix messages with a clear text label such as "Error" or "Success," use the matching token as supporting emphasis, and announce time-sensitive updates appropriately. Never rely on red or green alone.
- **Destructive actions:** Every destructive action requires an explicit confirmation step (NFR-5.5). State in the confirmation what will be removed and the true recovery timeline NFR-5.5 requires — the recovery window, when deletion becomes permanent, and the backup lag — and label the confirming control with the action rather than a bare "OK."
- **Journal authoring:** The first time a user creates or edits a journal entry, show a one-time notice that entry creation and edit timestamps are permanent and appear in exports (T-42; the export behavior is FR-9.4 in `REQUIREMENTS.md`).

## On-Screen Privacy

These conventions protect health, journal, and legal content from shoulder-surfing and shared or observed devices (resolves DQ-6). The platform mechanisms that enforce them, and the mobile security posture generally, are owned by `SECURITY.md` (SQ-14).

- **Re-lock:** After the background interval SEC-SESSION-7 (`SECURITY.md`) sets, a client locks and requires biometric or passkey confirmation to resume. The lock screen is a calm, content-free panel — the mark, the product name, and an unlock action — with no record data, counts, or notification content.
- **Task-switcher snapshot:** The snapshot the OS captures for the app switcher shows the same content-free panel, never live content.
- **Sensitive-content masking:** Content flagged sensitive renders masked — an obscured, neutral placeholder — until the user explicitly reveals it. The reveal control is labeled, keyboard accessible, and never the default state; masked content is announced to assistive technology as hidden, not read out.

## Accessibility

These rules implement NFR-5.1 (WCAG 2.2 Level AA) in the design language.

Use only verified text/background pairs at 4.5:1 or better for body text and 3:1 or better for large text; component boundaries and meaningful non-text graphics need at least 3:1 against adjacent colors. Preserve visible focus, logical keyboard order, native semantics, descriptive labels, and keyboard access to every action. Keep pointer targets at least 24×24 CSS px and prefer 44px control height. Respect `prefers-reduced-motion: reduce`: remove nonessential animation and make state changes understandable without motion. Do not use color, position, or motion as the only cue.

The web client is a React/TypeScript implementation (`ARCHITECTURE.md`, where CQ-2's resolution is recorded), so it delivers native DOM semantics and these rules apply to web and mobile alike, without caveat.
