# Moboost Ad Framework — Codex Project Guide

## Overview

Single-file interactive prototype for **Moboost** ad-operations admin (`ad-framework.html`, ~34k lines). All UI, CSS, and JavaScript live in one HTML file with in-memory mock data — no build step, no backend.

Open locally: double-click `ad-framework.html` or serve the folder with any static server.

Repository: `https://github.com/mssuper123/ad-framework-metaone.git`

## File layout

| File | Purpose |
|------|---------|
| `ad-framework.html` | Entire app: layout, styles, mock data, routing, all page logic |
| `标签条件字段清单.csv` | Reference doc for audience/tag filter field definitions |
| `.codex/config.toml` | Project-level Codex config |
| `AGENTS.md` | This file — agent instructions for Codex |

## Architecture

```
ad-framework.html
├── <style>           — CSS variables (--ag-*), component styles, dark mode
├── HTML shell        — sidebar nav, page containers (#xxxPage), modals/drawers
└── <script>          — mock data, MENU_DEF, rolePermissions, page renderers
```

### Navigation

- Sidebar sections use `navChild(el, secId)` / `navTo(el)`.
- Page switching: `setPage(page)` toggles `#xxxPage` visibility and calls `initXxxPage()`.
- Hash routes for Campaign/Offer forms: `#campaign`, `#campaign-outbound`, `#offer`, `#offer/form/:id`.

### Permissions

- `MENU_DEF` — menu tree for role-permission UI.
- `rolePermissions` — per-role `menus`, `buttons`, `dataScope`.
- `applyRolePermissions(roleId)` — hides nav items and gates row actions.
- `PAGE_TO_MENU` — maps `data-page` attrs to permission ids.

### Mock data conventions

- Lists: `let xxxList = [...]` with `nextXxxSeq` or numeric ids.
- Normalize on read: `xxxNormalizeItem(o)` mutates/returns consistent shape.
- Escaping HTML: `campEsc()`, `appMgmtEsc()` — always escape user-facing strings in templates.
- Toast: `showToast(msg)`.

### UI patterns

- Filters: toolbar rows with `.cm-inp` / `.cm-sel`, search button + reset.
- Tables/grids: CSS grid (`.off-grid-*`, `.camp-card-*`) or `<table class="cm-table">`.
- Forms: full-page (`#xxxFormPage`) or drawer (`#xxxDrawer`).
- Modals: `.mo` overlay + `openMo(id)` / `closeMo(id)`.

## Major modules (by page id)

| Section | Page ids | Notes |
|---------|----------|-------|
| Dashboard | `dashboard_monitor`, `dashboard_overview`, `dashboard_ctit`, `dashboard_itet` | Monitor = live metrics; others placeholder |
| CRM | `clues`, `huntlist`, `customer` | Leads, hunt targets, client mgmt |
| Basic | `channel`, `app`, `pid`, `event` | Master data |
| Ad Center | `campaign`, `campaign_outbound`, `offer`, `abtest` | Core ad ops; Offer has batch create + audience targeting |
| Other | `reports`, `finance`, `tools`, `datapackage` | Reports/finance = placeholder; tools/datapackage implemented |
| Perms | `dept`, `roles` | Org + RBAC config |
| Approval | `myapps`, `pendingapps` | App approval center |

## Key implementation areas

### Offer list (recent)

- 18 filter fields incl. Created Type (`manual` / `api`).
- 22 columns incl. Offer ID, Created Type, Campaign link, Ad Type, Billing Type.
- Filter logic: `getFilteredOfferRows()`, `offInitFilterSelects()`.
- Row render: `offerRenderOfferGridRow()`, `OFFER_LIST_COLUMNS`.

### Campaign / Offer forms

- Campaign: regions, budget types (CTV/CTA/VTA), pay events, link param variants.
- Offer: audience config (realtime/offline/mixed), channel config, run config, time curve, link params.
- Batch offer: `offerOpenBatchModal()`, wizard vs copy modes.

### Dashboard monitor

- `initDashboardPage()` → `dashRenderAll()`.
- Sections: metrics, flow, app/PRT/PID monitors.

## Coding rules

1. **Minimal diff** — single-file monolith; change only what the task requires.
2. **Match existing style** — naming (`camp*`, `off*`, `pkg*`), grid/filter patterns, mock data shape.
3. **No build tooling** — plain HTML/CSS/JS; CDN for Font Awesome, Chart.js, Google Fonts.
4. **No new dependencies** unless explicitly requested.
5. **Do not commit** unless the user asks.
6. **Comments** — only for non-obvious business logic.
7. **i18n** — `PAGES`, `I18N`, `L(zh, en)`; update both zh/en when adding labels.

## Search tips for Codex

```bash
# Find a page container
rg 'id=\"offerPage\"' ad-framework.html

# Find init/render functions
rg 'function initOfferPage|function renderOfferTable' ad-framework.html

# Find mock data
rg '^let offerList|^let campaignList' ad-framework.html

# Find CSS for a component
rg '\\.off-grid-' ad-framework.html
```

## Working in Codex

1. Open this folder as a **ChatGPT Project** or run Codex CLI from repo root.
2. Trust level is set in `.codex/config.toml` and `~/.codex/config.toml`.
3. Preview changes by refreshing `ad-framework.html` in a browser.
4. Sync to Git when the user explicitly requests it.

## Out of scope (unless asked)

- Splitting into multi-file or framework (React/Vue).
- Real API/backend integration.
- Automated tests (no test harness today).
