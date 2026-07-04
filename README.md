# VietstockFinance CSTC Comparator

A self-contained, browser-based equity analysis tool that reads two VietstockFinance **"Chỉ số tài chính" (CSTC)** Excel workbooks side-by-side and produces an interactive dashboard covering snapshot metrics, 5-year trends, three valuation methods, and a full metric comparison — entirely client-side, with no data ever leaving your machine.

---

## Quick Start

1. Open `VietstockFinance_CSTC_Comparator_test_3.html` in any modern browser (Chrome, Firefox, Edge, Safari).
2. Drop or click-to-browse **File A** and **File B** — one CSTC `.xlsx` file per slot.
3. The dashboard appears automatically once both files are loaded.
4. Use the **↺ Load different files** button to reset and compare a different pair.

> **No installation, no server, no internet required after the page loads.** The only external dependency is the `xlsx.js` spreadsheet library, fetched from a CDN on first load (with an `unpkg.com` fallback).

---

## Compatible Files

The tool is designed for workbooks exported from **VietstockFinance** using the *Báo cáo tài chính → Chỉ số tài chính (CSTC)* report type. The expected structure is:

| Cell | Content |
|------|---------|
| `A1` | `Chỉ số tài chính - <TICKER>` (e.g. `Chỉ số tài chính - VTO`) |
| `B1:F1` | Fiscal years as text strings, e.g. `"2021"` through `"2025"` |
| `A2` onward | Vietnamese metric row labels followed by 5 annual values |

The parser auto-detects the ticker symbol from `A1`, the year columns from row 1 (handling numbers, text strings, and Excel date serials), and matches metric rows using Vietnamese-language regular expressions — so it is robust to minor row-order variations between exports.

---

## Features

### Theme

A three-way toggle in the top-right corner switches between:

| Mode | Behaviour |
|------|-----------|
| **DARK** | Dark terminal aesthetic (default) |
| **LIGHT** | Clean light theme |
| **AUTO** | Follows the operating system's `prefers-color-scheme` setting and updates live if the OS setting changes |

The selected mode is persisted in `localStorage` across sessions.

---

### Tab 1 — Overview

A quick-glance summary of the two stocks.

- **Snapshot cards** — 10 key metrics for each stock pulled from the latest available year: EPS, BVPS, P/E, P/B, EV/EBIT, ROEA, Net Margin, Revenue Growth, Debt/Equity, and Dividend Yield.
- **Head-to-Head verdict panel** — 8 comparative rules (e.g. *lower P/E = cheaper*, *higher ROEA = better*) applied to the latest year figures. Each rule awards an edge badge to the winning stock, or shows a Tie. A horizontal score bar at the top summarises the overall tally.
- **Hover interaction** — hovering any table row or verdict row highlights it in amber and brightens the figures.

---

### Tab 2 — Trends

A 5-year time-series view for any metric in the dataset.

- **Metric selector** — a dropdown listing all 33 metrics matched across both files.
- **SVG line chart** — rendered without any external charting library. Features:
  - Smooth Catmull-Rom curves through data points
  - Gradient area fill under each line
  - Actual fiscal years on the x-axis (e.g. 2021–2025), not placeholders
  - Amber dashed **crosshair** that snaps to the nearest year on hover
  - **Floating tooltip** showing both stocks' values for the highlighted year
  - Dots enlarge on hover to mark the active point
  - Touch/swipe supported for mobile
- **Raw data tables** — one per stock below the chart, showing the exact values for the selected metric across all years.

---

### Tab 3 — Valuation

Three independent valuation methods, each selectable via toggle buttons. All methods use only data available in the CSTC file (per-share figures). A notice is displayed if required inputs are missing or the model constraints are violated.

#### Method 1 · Relative Valuation (P/E · P/B)

Implied price = multiple × per-share fundamental. Four implied prices per stock:

| Formula | Multiple Used |
|---------|--------------|
| Current P/E × EPS | Latest year P/E |
| Avg P/E × EPS | 5-year average P/E |
| Current P/B × BVPS | Latest year P/B |
| Avg P/B × BVPS | 5-year average P/B |

**Blended Fair Value** = average of all four. An editable **Market Price** input computes live upside/downside vs the blended figure.

*Required fields:* EPS, BVPS, P/E, P/B.

#### Method 2 · DCF (Discounted Cash Flow)

A multi-stage Gordon Growth DCF using CFO per Share (CPS) as a proxy for Free Cash Flow to Equity.

**User-controlled sliders:**

| Slider | Range | Default |
|--------|-------|---------|
| Discount Rate / WACC | 6 % – 25 % | 12 % |
| Terminal Growth Rate | 0 % – 8 % | 3 % |
| Projection Years | 3 – 10 | 5 |

The short-term growth rate is estimated from the historical average of Net Profit Growth, clipped to ±30 % to avoid implausible extrapolation. The output shows a year-by-year FCF/PV table, the PV of explicit cash flows, the PV of the terminal value, and the total **DCF Intrinsic Value**.

*Required fields:* CPS > 0; WACC > terminal growth rate.
*Will show a notice and omit the result if CPS is zero, negative, or missing.*

#### Method 3 · Asset-Based (Residual Income Model)

Single-stage Residual Income Model:

```
IV = BVPS + (EPS − ke × BVPS) / (ke − g)
```

Where `ke` is the required return on equity and `g` is the long-term growth rate. The model values the stock at book value plus the present value of economic profit — i.e. the earnings generated above the cost of equity capital.

**User-controlled sliders:**

| Slider | Range | Default |
|--------|-------|---------|
| Required Return on Equity (ke) | 6 % – 25 % | 12 % |
| Long-term Growth Rate (g) | 0 % – 10 % | 3 % |

The output also shows the **economic spread** (ROEA vs ke), which indicates whether the business is creating or destroying shareholder value.

*Required fields:* BVPS, EPS; ke > g.

---

### Tab 4 — Full Compare

A side-by-side table of all 33 matched metrics for the latest available year, with a fourth column showing the arithmetic difference (File A minus File B).

A **group filter** dropdown narrows the view to one of six categories:

| Group | Metrics included |
|-------|-----------------|
| Valuation | EPS, BVPS, P/E, P/B, P/S, Dividend Yield, Beta, EV/EBIT, EV/EBITDA |
| Profitability | Gross Margin, EBIT Margin, EBITDA Margin, Net Margin, ROEA, ROCE, ROAA |
| Growth | Revenue, Gross Profit, EBT, Net Profit, Total Asset, Equity growth rates |
| Liquidity | Cash, Quick, Current ratios; Interest Coverage |
| Leverage | Debt/Assets, Equity/Assets, Debt/Equity |
| Cash Flow | CFO/Revenue, CFO per Share, COGS/Revenue, SGA/Revenue |

Hovering any row highlights it and brightens the displayed figures. Positive differences are coloured green, negative in red.

---

## Metrics Reference

The tool matches 33 metrics from the Vietnamese CSTC sheet:

| Key | Label | Type |
|-----|-------|------|
| EPS | EPS (VND) | VND/share |
| BVPS | BVPS (VND) | VND/share |
| PE | P/E (x) | Multiple |
| PB | P/B (x) | Multiple |
| PS | P/S (x) | Multiple |
| DivYield | Dividend Yield | % |
| Beta | Beta | Multiple |
| EV_EBIT | EV/EBIT (x) | Multiple |
| EV_EBITDA | EV/EBITDA (x) | Multiple |
| GrossMargin | Gross Margin | % |
| EBITMargin | EBIT Margin | % |
| EBITDAMargin | EBITDA Margin | % |
| NetMargin | Net Margin | % |
| ROEA | Return on Equity (avg) | % |
| ROCE | Return on Capital Employed | % |
| ROAA | Return on Assets (avg) | % |
| RevGrowth | Revenue Growth | % |
| GrossProfitGrowth | Gross Profit Growth | % |
| EBTGrowth | EBT Growth | % |
| NPGrowth | Net Profit Growth | % |
| AssetGrowth | Total Asset Growth | % |
| EquityGrowth | Equity Growth | % |
| CashRatio | Cash Ratio | Multiple |
| QuickRatio | Quick Ratio | Multiple |
| CurrentRatio | Current Ratio | Multiple |
| IntCoverage | Interest Coverage | Multiple |
| DebtToAssets | Debt / Assets | % |
| EquityToAssets | Equity / Assets | % |
| DebtToEquity | Debt / Equity | % |
| CFO_Revenue | CFO / Revenue | % |
| CPS | CFO per Share (VND) | VND/share |
| COGS_Rev | COGS / Revenue | % |
| SGA_Rev | SGA / Revenue | % |

---

## Technical Notes

- **Single HTML file** — all HTML, CSS, and JavaScript are self-contained. No build step required.
- **No data upload** — files are read via the browser's `FileReader` API and processed entirely in memory. Nothing is sent to any server.
- **External dependency** — `xlsx.js` v0.18.5 is loaded from `cdnjs.cloudflare.com`, with `unpkg.com` as a fallback. If both CDNs are unreachable (e.g. fully offline), file parsing will fail with a clear error message.
- **Chart rendering** — the trend chart is drawn with hand-coded SVG and vanilla JavaScript. There is no charting library dependency, so it cannot fail due to a CDN issue.
- **Year parsing** — the parser handles year values stored as plain numbers, text strings (`"2021"`), JavaScript `Date` objects, or Excel date serial numbers, with a direct cell-address fallback (`B1:J1`) if row-scanning finds nothing.
- **Theme persistence** — the selected theme (Dark / Light / Auto) is saved to `localStorage` under the key `cstc_theme`.
- **Responsive layout** — the two-column grid collapses to a single column on screens narrower than 740 px.

---

## Limitations

- Only the **CSTC sheet** is read; income statement, balance sheet, and cash flow statement sheets (if present in the workbook) are not used.
- All valuation methods rely exclusively on **per-share figures** (EPS, BVPS, CPS). Absolute revenue, total assets, and net debt are not available from the CSTC export, so EV-based DCF and NAV liquidation methods are not implemented.
- DCF growth is estimated from historical Net Profit Growth, which is a rough proxy. The result is highly sensitive to WACC and terminal growth assumptions — treat it as a stress-test tool, not a price target.
- The Residual Income model uses a single-stage perpetuity formula. It does not account for changing book value or multi-stage growth.

---

## File Structure

```
VietstockFinance_CSTC_Comparator_test_3.html
│
├── <style>          CSS variables (dark/light themes), layout, components
│
├── <body>
│   ├── <header>     Title + three-way theme toggle
│   ├── .uploader    Two drag-and-drop / click-to-browse file slots
│   └── #appRoot     Dashboard (hidden until both files are loaded)
│       ├── .tabs    Overview · Trends · Valuation · Full Compare
│       └── .views   One <div> per tab, rendered dynamically
│
└── <script>
    ├── Theme        applyTheme(), localStorage persistence, OS media query listener
    ├── Constants    MDEFS (33 metric definitions), GROUPS, VERDICTS
    ├── Parsing      findSheet(), asYear(), parseWb(), readFile()
    ├── Wiring       wireDZ() (dropzones), resetBtn, tab clicks, method switcher, sliders
    ├── Overview     buildOverview() — snapshot cards + verdict panel
    ├── Trends       buildTrends() — metric selector + SVG chart + raw tables
    ├── Valuation    buildValuation() → buildRel(), buildDCF(), buildAB()
    ├── Compare      buildCompare() — filterable full metric table
    └── Chart        drawChart() — SVG line chart with crosshair tooltip
```

---

*Built for Vietnamese equity analysis using VietstockFinance CSTC export data. All processing is local — no data leaves your browser.*
