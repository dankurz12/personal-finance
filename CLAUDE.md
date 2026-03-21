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
- API supports: `GET ?action=getAll`, `GET ?action=get&year=Y&month=M`, `POST {action:'save', year, month, data}`, `POST {action:'delete', year, month}`
- Data stored as rows: `year (int) | month (int, 0-indexed) | data (JSON string)`
- All data loaded into `allData` object on init, keyed as `year_month`

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
- Font: DM Mono (monospace), DM Sans (body) — loaded from Google Fonts
- Background palette: `#0a0a0f` (bg), `#111118` (bg2), `#1a1a24` (bg3), `#222230` (bg4)
- Border: `#2a2a3a` (border), `#3a3a4a` (border2)
- Text: `#e8e8f0` (text), `#9090a8` (text2), `#5a5a72` (text3)
- Accents: `#00d4aa` (green), `#ff4d6a` (red), `#f5c842` (yellow), `#4d9fff` (blue), `#7b5ea7` (purple), `#00c4c4` (teal)
- High-density layout — tight padding, small font sizes, minimal whitespace
- All colors defined as CSS variables in `:root`

## Tabs / Sections
1. **Overview** — KPI cards, net worth trend chart (canvas), asset allocation bars, savings rate, FIRE progress bars, milestones
2. **FIRE** — settings panel, KPI cards, FIRE goals, investment milestones, CMGR, SWR projection matrix, Coast FI table
3. **P&L** — income statement table, spending bars
4. **Balance Sheet** — assets table, liabilities + net worth table
5. **Cash Flow** — month-over-month comparison table
6. **Spending** — ranked category breakdown
7. **Data Entry** — form to input monthly data, saves to Google Sheets

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
Stored in `FIRE_CONFIG` object near top of JS. User can edit via settings panel in FIRE tab. Key fields: `currentAge`, `withdrawalRate`, `returnRate`, `retirementAge`, `annualContribution`, `annualRetirementSpend`, `leanFire`, `fire`, `fatFire`, `coastAge`, `coastFiAt65`, `cmgrTotal`, `cmgrLtm`, `saved2025`, `savings2025Rate`.

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
