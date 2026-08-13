# AfterImpact — Design

This document owns the visual and frontend design language: brand and logo, color palette, typography, layout, component conventions, and the design rules that meet the accessibility target. `style-guide.html` is its rendered reference and must agree with it. `REQUIREMENTS.md` owns WHAT the system does; `ARCHITECTURE.md` owns how it is built; `SECURITY.md` owns the security posture.

The audience is a person recovering from a recent car accident, who may be injured, stressed, or medicated. The clients are a web application and a mobile application built from a shared Compose Multiplatform codebase (`ARCHITECTURE.md`).

## Brand and Logo

AfterImpact is the known product name. Its brand direction and personality are TO BE DECIDED (DQ-1), so the current system is a neutral, readable provisional foundation rather than a claim about tone.

`logo.svg` is the primary mark. Its two offset, rounded paths bracket a central transition: the first path ends and the second continues, an abstract reference to moving forward after a disruptive event. The geometry is intentionally simple enough to hold at small sizes. The mark uses `primary` and `secondary` only and is designed for `background` and `surface` light backgrounds.

Maintain clear space equal to one eighth of the mark's displayed width on all sides. Do not display it below 24 CSS px square. Keep its aspect ratio and supplied colors; do not rotate it, add effects, place it on low-contrast or visually busy backgrounds, rearrange its paths, or treat nearby typeset product text as a fixed wordmark. A future wordmark or lockup is TO BE DECIDED (DQ-1).

## Color Palette

| Token | Hex | Intended use |
| --- | --- | --- |
| `primary` | `#234E70` | Primary actions, links, focus rings, and the first logo path |
| `secondary` | `#0F766E` | Secondary emphasis and the second logo path |
| `background` | `#F6F8FA` | Page background |
| `surface` | `#FFFFFF` | Cards, fields, and raised content areas |
| `text` | `#17202A` | Body text and headings |
| `error` | `#B42318` | Error text, borders, and destructive-status emphasis |
| `success` | `#146C43` | Success text, borders, and confirmation emphasis |

Approved text/background pairs and WCAG contrast ratios are: `text` on `background` 15.45:1; `text` on `surface` 16.45:1; white on `primary` 8.77:1; white on `secondary` 5.47:1; white on `error` 6.57:1; white on `success` 6.45:1; and `primary` links on `background` / `surface` 8.24:1 / 8.77:1. Use only these documented pairings for text.

The palette above is the light theme. Whether a dark theme ships is TO BE DECIDED (DQ-3); no dark-theme token values exist yet.

## Typography

The broader typography direction is TO BE DECIDED (DQ-5). The provisional primary family is `system-ui`, with `-apple-system`, `BlinkMacSystemFont`, `"Segoe UI"`, and `sans-serif` fallbacks. It is neutral, readable, requires no network or font license dependency, and follows the user's platform conventions.

Use weights 400 (body), 600 (controls and small headings), and 700 (major headings). The type scale is `xs` 0.75rem/1rem, `sm` 0.875rem/1.25rem, `base` 1rem/1.5rem, `lg` 1.25rem/1.75rem, `xl` 1.5rem/2rem, `2xl` 2rem/2.5rem, and `3xl` 2.5rem/3rem. Keep body copy at `base` or larger.

## Layout and Spacing

Use a 4px base spacing scale: `0`, `1` 4px, `2` 8px, `3` 12px, `4` 16px, `6` 24px, `8` 32px, `12` 48px, and `16` 64px. Prefer a centered content area no wider than 72rem and readable text lines near 68 characters. Assume a fluid four-column grid at narrow widths and a twelve-column grid where space permits; components span the columns their content needs.

Responsive breakpoints are `sm` 30rem, `md` 48rem, `lg` 64rem, and `xl` 80rem. Layouts must reflow rather than depend on a particular device size, and must remain usable from a 360 px viewport up (NFR-5.2).

## Components

- **Buttons:** Use a filled `primary` treatment for the single most important action in a group and a `surface` treatment with a `primary` border for alternatives. Labels should describe the action. Preserve a minimum 44px control height, clear hover feedback, and the shared focus treatment. Do not communicate state through color alone.
- **Inputs:** Keep a persistent visible label above each field. Use `surface`, `text`, and a visible border; help text may follow the field. A placeholder never replaces a label. Invalid fields use an `error` border plus adjacent, programmatically associated text that explains how to recover.
- **Links:** Use `primary` and underline links in prose. Hover may strengthen the underline; visited behavior must remain distinguishable without reducing contrast.
- **Focus states:** Every interactive element receives a 3px `primary` focus ring with a 2px `surface` offset on `:focus-visible`. Do not remove focus indication.
- **Form feedback:** Place feedback close to the relevant field or action. Prefix messages with a clear text label such as "Error" or "Success," use the matching token as supporting emphasis, and announce time-sensitive updates appropriately. Never rely on red or green alone.
- **Destructive actions:** Every destructive action requires an explicit confirmation step (NFR-5.5). State in the confirmation what will be removed, and label the confirming control with the action rather than a bare "OK."

## Accessibility

These rules implement NFR-5.1 (WCAG 2.2 Level AA) in the design language.

Use only verified text/background pairs at 4.5:1 or better for body text and 3:1 or better for large text; component boundaries and meaningful non-text graphics need at least 3:1 against adjacent colors. Preserve visible focus, logical keyboard order, native semantics, descriptive labels, and keyboard access to every action. Keep pointer targets at least 24×24 CSS px and prefer 44px control height. Respect `prefers-reduced-motion: reduce`: remove nonessential animation and make state changes understandable without motion. Do not use color, position, or motion as the only cue.

Whether the web client can deliver native semantics is unresolved — Compose Multiplatform's web target renders to canvas (CQ-2).

## Open Questions

- **DQ-1** What brand personality should AfterImpact express, and does it call for a wordmark or lockup alongside `logo.svg`?
- **DQ-3** Should the product support light mode, dark mode, or both?
- **DQ-4** Do existing brand assets need to be incorporated or reconciled with `logo.svg`?
- **DQ-5** Should the provisional system-font direction remain, or should an open-licensed brand typeface be selected?
