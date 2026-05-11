# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal finance web app for a couple (Giovanna & Bruno), hosted on GitHub Pages. No build step — the entire app is a single `index.html` file. "Database" is a Google Sheets spreadsheet accessed through a Google Apps Script web app endpoint.

## Development

No build, no package manager, no dependencies to install. Edit `index.html` directly and open it in a browser to test. To preview changes live, use VS Code Live Server or any local HTTP server:

```bash
npx serve .
# or
python -m http.server 8080
```

Deploy by pushing to `main` — GitHub Pages serves `index.html` automatically.

## Architecture

Everything lives in `index.html` (~130 KB):

- **CSS** — top `<style>` block. Uses CSS custom properties for theming. Four themes (Aurora, Pulse, Cocoon, Glow) × three profiles (Giovanna `prof-gi`, Bruno `prof-br`, Casal `prof-ca`) controlled by classes on `<body>`.
- **HTML** — three screens: `#tela-login`, `#tela-ind` (individual dashboard), `#tela-casal` (couple dashboard). Modals are present in the DOM at all times, shown/hidden via JS.
- **JavaScript** — bottom `<script>` block. No modules, no framework. Global state in plain variables (`stateGi`, `stateBr`, `stateCa`). ~160+ functions, all global scope.

### Data flow

```
Google Sheets ← Google Apps Script web app → fetch() → local JS state → DOM render
```

1. User selects a profile → `selProfile()` → `loadDash()` → `api('getDashboardData', {usuario})`.
2. Response populates `stateGi` or `stateBr` (objects matching mock shape below).
3. `renderInd()` / `renderCasal()` re-renders the DOM from state.
4. CRUD operations call `api()` with write actions, then re-fetch or patch local state.

### State shape

```js
{
  usuario, gastos, entradas, recorrentes, investimentos,
  reserva, aportes, metaHistorico, evolucao,
  config: { meta_reserva }
}
```

Mock data (`MOCK_GI`, `MOCK_BR`) mirrors this shape and is used as fallback/dev data.

### Apps Script endpoint

Stored in the `api()` function as a constant (`SCRIPT_URL`). All requests go to this single URL with an `action` query param. The Apps Script handles routing by action name.

### localStorage

Used only to persist per-user "paid" status for transactions across sessions. Key pattern: `paid_{usuario}_{year}_{month}`.

## Key constraints

- **No framework, no bundler** — keep it that way. Adding npm/webpack would complicate the GitHub Pages deploy.
- **Single file** — all CSS, JS, and HTML stay in `index.html`. Do not split into separate files unless the user explicitly requests it.
- **Portuguese (pt-BR)** — all UI text, variable names for domain concepts, and user-facing strings are in Portuguese.
- **Apps Script is the only backend** — there is no server, no auth service, no database other than Google Sheets. Data mutations must go through `api()`.
