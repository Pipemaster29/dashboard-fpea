# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Response style

Be extremely concise and objective in chat replies in this repo. Short, direct sentences. No preamble, no
filler, no restating the question, no hedging. Get to the point in as few words/tokens as possible while
staying technically correct.

## What this is

A single-file, client-side FP&A dashboard for GNL Brasil Logística ("Dashboard FP&A"). Users drag in Excel
exports (SAP KSB1 cost extracts, budget spreadsheets, revenue, SG&A ledger); everything is parsed and
rendered entirely in the browser. There is no backend, no build step, and no package.json — the whole app
is `index.html`.

## Files

- `index.html` — the entire application: HTML markup, CSS, and JS all in one file (~2750 lines). One
  `<script>` block starting after the `<head>` styles contains all logic.
- `gnl-logo.png` — header logo, referenced by `index.html`.
- `COMO-PUBLICAR.md` — (Portuguese) end-user instructions for publishing the dashboard via GitHub Pages.
  Describes the spreadsheet formats the app expects (KSB1 realizado, "Consolidado CV-CF-SG&A" orçamento).
- `.claude/` / `.agents/` — installed Claude skills (animation-vocabulary, emil-design-eng,
  review-animations), tracked via `skills-lock.json`. Not project-specific rules.

## Development workflow

There is no package manager, build tool, linter, or test suite in this repo — it's a static HTML file
loading libraries from CDN (`xlsx`, `chart.js` + `chartjs-plugin-datalabels`, `@supabase/supabase-js`,
`pako`, `pptxgenjs`). To work on it:

- **Run locally**: just open `index.html` in a browser, or serve the directory with any static file server
  (e.g. `python3 -m http.server`) so `fetch`/module behavior matches production. There's no dev server
  script.
- **No automated tests.** Verify changes manually in the browser: load sample Excel files through the drop
  zone and click through the tabs.
- **Deploy**: this is published via GitHub Pages directly from `main` (root). There is no CI/build —
  pushing `index.html` to `main` is the entire deploy (see `COMO-PUBLICAR.md`).
- Since the whole app is one file, prefer targeted edits (`Edit`, not rewriting the file) and use line
  numbers/`grep` to navigate — `index.html` is too large to hold in context at once.

## Architecture

### Global state (all in-memory, top of the `<script>` block)

The app has no framework/components — it's plain DOM manipulation over a handful of global arrays that
hold parsed spreadsheet rows:

- `DADOS_RAW` / `DADOS` — realizado (actual costs), full vs. the current YTD-filtered view.
- `ORCAMENTO` — budget (orçado) rows: `{tipo, mes, classeCod, classeNome, valor}`.
- `ORCKPI` — budget R$/km KPIs parsed from an "OBZ" sheet: `{localidade: {mes: {km, custo: {...}}}}`.
- `ORCREC` — revenue budget rows.
- `RECEITA_RAW` / `RECEITA` — actual revenue rows, full vs. YTD-filtered.
- `SGA_RAW` / `SGA` — SG&A actuals from the accounting ledger.

`DADOS`, `RECEITA`, `SGA` are always derived from their `_RAW` counterparts by `aplicarPeriodo()`, which
applies the "Período (YTD)" cutoff-month selector — **every** analysis reads from the filtered (non-`_RAW`)
variable, never `_RAW` directly. `_RAW` is what gets persisted to Supabase.

### Ingest pipeline (`lerArquivo`)

Every uploaded file goes through `XLSX.read` → the app **sniffs the sheet shape** to decide what kind of
file it is, in this priority order: revenue (`ehReceitaGNL`) → SG&A ledger (`ehSGA`) → revenue budget
(`ehOrcReceita`) → OBZ KPI budget (`ehOBZ`) → cost budget (`ehOrcamento`) → otherwise treated as a realizado
(cost) extract via `mapear()`. Each recognized type has a matching `parseX(aoa)` function. Re-uploading a
file **replaces** the previous data for that same source/origem (matched by filename-derived `origem`), it
does not append/duplicate. After ingest, `aplicarPeriodo()` re-applies the period filter and
`agendarSalvamento()` debounces an auto-save to the cloud.

### Tabs and lazy rendering

Top-level views are plain `<div id="...">` sections toggled by `showTab()`/`applyTabs()`: `ovr` (Orçado vs
Real), `rec` (Receita & Viagens), `dre` (DRE/EBITDA), `exec` (Execução por Área), `fixo` (Custos Fixos),
`ops` (Operações & R$/km), `pareto` (Pareto & Gargalos), `dash` (Análise de Custos). A tab only appears in
the nav if `tabHasContent(id)` says it has data to show.

Rendering is lazy and dirty-tracked: `RENDER_TAB` maps each tab id to its render function; `marcarTudoSujo()`
flags all tabs dirty whenever underlying data changes; `renderTab(t)` only actually re-renders a tab the
first time it's shown after becoming dirty. When adding a new tab or changing what invalidates a view, make
sure `marcarTudoSujo()` (or a targeted `_dirty[tab]=true`) is called wherever the source data changes —
otherwise stale views won't refresh.

Some tabs additionally have "subpages" (`.sp` elements + `.subnav` segmented control), tracked per-tab in
`localStorage` via `showSub`/`restoreSubs`.

### Persistence — Supabase as a shared single-document store

There is no auth and no per-user data. `sb` is a Supabase client (anon/publishable key, hardcoded — this is
intentional, it's a client-only anon key) reading/writing a single row (`id:'atual'`) in the `dataset`
table. The entire state (`DADOS_RAW, ORCAMENTO, ORCKPI, ORCREC, RECEITA_RAW, SGA_RAW`) is JSON-serialized,
gzipped with `pako`, base64-encoded (`comprimir`/`descomprimir`, prefixed `gz:`), and stored as one JSON
blob. On page load the app fetches this row and repopulates all globals (`carregarDaNuvem`). Any data
mutation should go through `agendarSalvamento()` (debounced auto-save) so the cloud copy stays in sync —
don't mutate `_RAW` arrays without triggering a save if the change should persist.

`migrarDados()` runs after loading cloud data to backfill fields (e.g. `bloco`/`nat`) that didn't exist in
older saved payloads — when changing a data row's shape, add a migration step here rather than assuming all
persisted data matches the current shape.

### Domain concepts worth knowing before editing calculations

- **Localidade**: the operation is split into two regions inferred from the cost-center code's last letter
  — `localidadeDe()`: ends in "A" → Amazonas, otherwise → Maranhão.
- **Natureza (nat)**: costs are `variavel` or `fixo` (`natDe`); fixed costs are further grouped into
  "blocos" (`blocoDe`/`blocoFixoDe`/`blocoPorNome`).
- **Origem**: which source file a realizado row came from (Serviços, Materiais, Pessoal/Folha, CF O&M),
  inferred from filename via `origemDe()`.
- **PIS/COFINS factor**: budget revenue/cost values are optionally adjusted by a 9.25% tax factor
  (`fatorPisCofins()`) when the "considerar PIS/COFINS" checkbox is on — applied via `orcVal()`.
- **DRE/EBITDA**: `dreDados()`/`dreLinhas()` build the P&L waterfall; costs are counted at "valor CHEIO" per
  a deliberate design decision (manual/non-recurring entries neutralize NR inside cost, not outside it —
  see commit history for the reasoning if touching this).
- Export functions (`exportarXxx`) mirror each render function and re-derive the same aggregates for
  Excel/PNG/PPT output — when changing an aggregation's logic, check whether the corresponding export
  function duplicates that logic and needs the same fix.

## Conventions

- Code, comments, UI copy, and commit messages in this repo are in **Portuguese (pt-BR)**. Match that when
  editing — variable/function names (`renderReceita`, `montarFiltros`, `escolherAba`, etc.) and user-facing
  strings are Portuguese, not translations layered on top.
- Currency/number formatting goes through the shared `BRL`/`INT`/`PCT`/`kBRL` helpers — don't hand-roll
  `toLocaleString` elsewhere.
- Month values are normalized to the `MM/YYYY` string format everywhere (`mesDe()` parses source dates into
  this shape; `mesOrd()` sorts by it) — keep new date handling consistent with this format rather than
  using `Date` objects downstream.
