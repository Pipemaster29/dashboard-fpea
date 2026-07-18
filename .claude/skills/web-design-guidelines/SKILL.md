---
name: web-design-guidelines
description: Review UI code for Web Interface Guidelines compliance (Vercel). Use when asked to "review my UI", "check accessibility", "audit design", "review UX", or "check the site against best practices". For this repo, the default target is index.html.
metadata:
  author: vercel (installed locally for offline use)
  version: "1.0.0"
  argument-hint: <file-or-pattern>
---

# Web Interface Guidelines

Review files for compliance with the Web Interface Guidelines. If no files are specified, review `index.html`.

Optionally fetch the latest rules from
`https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md`;
if unavailable, use the embedded rules below (snapshot).

Read files, check against rules below. Output concise but comprehensive—sacrifice grammar for brevity. High signal-to-noise.

## Rules

### Accessibility

- Icon-only buttons need `aria-label`
- Form controls need `<label>` or `aria-label`
- Interactive elements need keyboard handlers (`onKeyDown`/`onKeyUp`)
- `<button>` for actions, `<a>`/`<Link>` for navigation (not `<div onClick>`)
- Images need `alt` (or `alt=""` if decorative)
- Decorative icons need `aria-hidden="true"`
- Async updates (toasts, validation) need `aria-live="polite"`
- Use semantic HTML (`<button>`, `<a>`, `<label>`, `<table>`) before ARIA
- Headings hierarchical `<h1>`–`<h6>`; include skip link for main content
- `scroll-margin-top` on heading anchors

### Focus States

- Interactive elements need visible focus: `focus-visible` styles
- Never `outline: none` without focus replacement
- Use `:focus-visible` over `:focus` (avoid focus ring on click)
- Group focus with `:focus-within` for compound controls

### Forms

- Inputs need `autocomplete` and meaningful `name`
- Use correct `type` (`email`, `tel`, `url`, `number`) and `inputmode`
- Never block paste (`onPaste` + `preventDefault`)
- Labels clickable (`for`/`htmlFor` or wrapping control)
- Disable spellcheck on emails, codes, usernames
- Checkboxes/radios: label + control share single hit target (no dead zones)
- Submit button stays enabled until request starts; spinner during request
- Errors inline next to fields; focus first error on submit
- Placeholders end with `…` and show example pattern
- Warn before navigation with unsaved changes

### Animation

- Honor `prefers-reduced-motion` (provide reduced variant or disable)
- Animate `transform`/`opacity` only (compositor-friendly)
- Never `transition: all`—list properties explicitly
- Set correct `transform-origin`
- Animations interruptible—respond to user input mid-animation

### Typography

- `…` not `...`; curly quotes
- Non-breaking spaces: `10&nbsp;MB`, `⌘&nbsp;K`, brand names
- Loading states end with `…`: `"Carregando…"`, `"Salvando…"`
- `font-variant-numeric: tabular-nums` for number columns/comparisons
- `text-wrap: balance` on headings (prevents widows)

### Content Handling

- Text containers handle long content: truncate, line-clamp, or break-words
- Flex children need `min-width: 0` to allow text truncation
- Handle empty states—don't render broken UI for empty strings/arrays
- User-generated content: anticipate short, average, and very long inputs

### Images

- `<img>` needs explicit `width` and `height` (prevents CLS)
- Below-fold images: `loading="lazy"`; critical images: `fetchpriority="high"`

### Performance

- Large lists (>50 items): virtualize or `content-visibility: auto`
- No layout reads in render (`getBoundingClientRect`, `offsetHeight`, `scrollTop`)
- Batch DOM reads/writes; avoid interleaving
- Add `<link rel="preconnect">` for CDN/asset domains
- Critical fonts: `<link rel="preload" as="font">` with `font-display: swap`

### Navigation & State

- URL reflects state—filters, tabs, pagination, expanded panels in query params
- Links use `<a>` (Cmd/Ctrl+click, middle-click support)
- Deep-link all stateful UI
- Destructive actions need confirmation modal or undo window—never immediate

### Touch & Interaction

- `touch-action: manipulation` (prevents double-tap zoom delay)
- `-webkit-tap-highlight-color` set intentionally
- `overscroll-behavior: contain` in modals/drawers/sheets
- `autofocus` sparingly—desktop only, single primary input; avoid on mobile

### Safe Areas & Layout

- Full-bleed layouts need `env(safe-area-inset-*)` for notches
- Avoid unwanted scrollbars; fix content overflow
- Flex/grid over JS measurement for layout

### Dark Mode & Theming

- `color-scheme: dark` on `<html>` for dark themes (fixes scrollbar, inputs)
- `<meta name="theme-color">` matches page background
- Native `<select>`: explicit `background-color` and `color` (Windows dark mode)

### Locale & i18n

- Dates/times: `Intl.DateTimeFormat`; numbers/currency: `Intl.NumberFormat`
- Brand names, code tokens: `translate="no"` to prevent garbled auto-translation

### Hover & Interactive States

- Buttons/links need hover state (visual feedback)
- Interactive states increase contrast: hover/active/focus more prominent than rest

### Content & Copy

- Active voice; specific button labels ("Salvar na nuvem", not "Continuar")
- Numerals for counts: "8 lançamentos" not "oito"
- Error messages include fix/next step, not just problem

### Anti-patterns (flag these)

- `user-scalable=no` or `maximum-scale=1` disabling zoom
- `onPaste` with `preventDefault`
- `transition: all`
- `outline: none` without focus-visible replacement
- `<div>`/`<span>`/`<tr>` with click handlers and no keyboard access
- Images without dimensions
- Form inputs without labels; icon buttons without `aria-label`
- Hardcoded date/number formats (use `Intl.*`)

## Output Format

Group by file. Use `file:line` format (VS Code clickable). Terse findings.
State issue + location. Skip explanation unless fix non-obvious. No preamble.

## Project note (dashboard-fpea)

Single-file app (`index.html`), dark theme, pt-BR, Chart.js + SheetJS + Supabase.
Never break the analyses/reconciliation logic while fixing UI findings; after any
change run the Playwright battery (all tabs + DRE check-up all green + zero pageerrors)
before publishing.
