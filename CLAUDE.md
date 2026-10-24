# Personal Finance Dashboard — Claude Code Context

## Project Overview
A single-file personal finance dashboard deployed on GitHub Pages. It tracks income, spending, account balances, net worth, and FIRE progress. Data is stored in Google Sheets via a Google Apps Script web app and loaded on page init.

## Repo
- GitHub: https://github.com/dankurz12/personal-finance
- Live URL: https://dankurz12.github.io/personal-finance/
- Single file: `index.html` (must be lowercase)

## Stack
- Vanilla JavaScript, HTML, CSS — no frameworks, no build tools, no npm
- Single `index.html` file — everything lives in one file (HTML, CSS, JS)
- No external JS dependencies beyond Google Fonts
- Deployed via GitHub Pages (Linux, case-sensitive filenames)

## Backend
- Google Sheets via Google Apps Script web app
- Script URL is stored in `const SCRIPT_URL` near the top of the JS section
- API supports: `GET ?action=getAll`, `GET ?action=hasApiKey`, `POST {action:'save', year, month, data}`, `POST {action:'delete', year, month}`, `POST {action:'setApiKey', key}`, `POST {action:'parseStub', pdf}`
- Every request carries `token` (GET query param / POST body field) — `API_TOKEN`, derived at unlock as SHA-256 of `'pfm-api-v1:' + password` and kept in `sessionStorage`. The Apps Script checks it against its `PFM_TOKEN` script property. Always call the backend through `apiGet()` / `apiPost()`, never raw `fetch(SCRIPT_URL)`
- The Anthropic key lives only in the Apps Script; pay stubs are parsed server-side via `parseStub`. Never send the key to the browser
- Data stored as rows: `year (int) | month (int, 0-indexed) | data (JSON string)`
- All data loaded into `allData` object on init, keyed as `year_month`
- Settings (FIRE config, mortgage inputs) are saved as a sentinel row: year `9999` (`SETTINGS_YEAR`), month 0

## Data Model
Each month's data object looks like:
```json
{
  "salary": 0,
  "bonus": 0,
  "otherincome": 0,
  "checking": 0,
  "savings": 0,
  "brokerage": 0,
  "k401": 0,
  "ira": 0,
  "otherassets": 0,
  "cc": 0,
  "studentloan": 0,
  "otherdebt": 0,
  "spending": {
    "Rent": 0, "Utilities": 0, "Transportation": 0, "Groceries": 0,
    "Restaurants": 0, "Coffee / Snacks": 0, "Alcohol": 0, "Entertainment": 0,
    "Fitness": 0, "Clothing": 0, "Personal": 0, "Gas": 0, "Household": 0,
    "Travel": 0, "Gifts": 0, "Medical/Dental": 0, "Other": 0
  }
}
```

## Spend Categories (exact strings, order matters)
Rent, Utilities, Transportation, Groceries, Restaurants, Coffee / Snacks, Alcohol, Entertainment, Fitness, Clothing, Personal, Gas, Household, Travel, Gifts, Medical/Dental, Other

## Design System
- Fonts: Barlow (body), Barlow Condensed (headings/KPIs), DM Mono (numbers, labels) — loaded from Google Fonts
- Light, muted palette. Background: `#f4f3f0` (bg), `#faf9f6` (bg2), `#efeeea` (bg3), `#e6e5e1` (bg4); border `#e4e2dd` / `#d0cec8`
- Text: `#16181a` (text), `#55555c` (text2), `#8a8a88` (text3)
- Accents: `#5980a6` (blue/accent), `#2f6a43` (green), `#b0413a` (red), `#8a6d2e` (yellow), `#5a6b74` (teal); spending series `#b0705a`
- Square corners, high-density layout — tight padding, small font sizes, minimal whitespace
- All colors defined as CSS variables in `:root`
- Reusable UI: `.seg` segmented toggles (set state, then `segOn(id, dataKey, value)`), `.perf-grid`/`.perf-cell` stat rows, `.verdict` one-line summaries, `.insight` cards, `registerChart()` for canvas hover tooltips and click-to-open-month

## Git Workflow
- **Commit directly to `main`** — do not create feature branches or PRs
- Push to `origin main` after every change so GitHub Pages deploys immediately
- GitHub Pages auto-deploys from `main`; changes are live within ~30 seconds of push

## Tabs / Sections
1. **Overview** — headline strip, household summary (optional), insights, KPI cards, net worth trend (nominal/today's $), allocation, milestones with pacing
2. **Review** — month-end close: verdict, vs prior / 12-month average, category deltas, flags
3. **FIRE** — settings, milestones, growth projection (return scenarios, goal path), FIRE number check vs actual spending, bridge to 59½, tax-bucket mix, CMGR, SWR matrix, Coast FI, contribution impact
4. **Plan** — 12-month cash-flow forecast, scenario planner, Monte Carlo, mortgage, tax-advantaged space, brackets
5. **P&L** — Sankey, income waterfall, income statement, annual P&L
6. **Balance Sheet** — assets, liabilities, monthly net-worth reconciliation
7. **Spending** — breakdown, essentials vs discretionary, YTD comparison, heatmap, category trend
8. **Trends** — trailing-12 KPIs, investment performance (TWR/IRR), income vs spending, savings rate, NW change, YoY, income growth, wealth composition
9. **Data Entry** — form, pay stub import, backfill, saves to Google Sheets

## Key JS Functions
- `init()` — loads all data from Sheets, builds UI
- `refreshAll()` — redraws all sections for current period
- `compute(d)` — derives totals from raw month data object
- `updateFire(d)` — renders FIRE tab, uses `FIRE_CONFIG`
- `saveData()` / `clearMonth()` — write/delete via Sheets API
- `drawNWChart()` — draws net worth trend on canvas element
- `fmt(n, sign)` — formats numbers as currency
- `pct(n, dec)` — formats as percentage
- `showToast(msg, type)` — shows ok/err toast notification

## FIRE Config
Stored in `FIRE_CONFIG` object near top of JS; overridden on load by the saved settings row (`loadSettings()`), saved by `saveSettings()`. User can edit via settings panel in FIRE tab. Key fields: `dob` (age is derived live), `withdrawalRate`, `returnRate`, `retirementAge`, `annualContribution` (via `effectiveContribution()`: LTM actuals or `ctbManual` per `ctbMode`), `annualRetirementSpend`, `leanFire`, `fire`, `fatFire`, `coastAge`, `partnerInclude`, `partnerNw`, `inflationRate` (use this — never hardcode 2%), `healthcarePre65`, `iraRothPct`, `k401RothPct`, `rothBasisPrior`. Any new persisted field must also be added to the `saveSettings()` payload.

Spending categories in `ESSENTIAL_CATS` count as essentials; all others are discretionary.

## Owner Context
- Dan, age 29, works in financial due diligence at Deloitte
- No debt, maxes retirement accounts, invests in taxable brokerage
- FIRE target: $2M, LeanFIRE: $1.6M, FatFIRE: $3M
- Historical data goes back to January 2023

## Coding Conventions
- No semicolons optional — existing code uses semicolons, keep consistent
- CSS classes are short and semantic (`.kpi`, `.tbl`, `.btn`, `.card`, `.g2`, `.g3`)
- JS is compact — prefer inline ternaries and chained operations over verbose code
- Never add external JS libraries or CDN dependencies
- Never split into multiple files — everything stays in `index.html`
- Never add a build step or package.json
- When adding new tabs: add to the nav tabs list, create a `section-{id}` div, add to the `tabs` array in `showSection()`, and add a corresponding `update{Name}()` function called from `refreshAll()`

## What NOT to Do
- Do not add an AI analysis feature — this was intentionally removed
- Do not change the Google Apps Script URL
- Do not add localStorage — all persistence is via Google Sheets
- Do not change the spend category names or order
- Do not add npm, webpack, or any build tooling
- Do not rename `index.html`
