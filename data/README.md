# Where Does Revenue Stop Becoming Profit?

**A 3-page strategic Power BI report on profitability, staffing and structural cost for a 60-store UK coffee chain.**

Built for the Onyx Data challenge by Emmanuel Idowu (Senior Data Analyst, Power BI developer).

> Screenshots: add images to `/docs/screenshots` and link them here.
> `![Page 1 - Profit Leakage](docs/screenshots/page1.png)`

---

## Table of contents

1. [Project overview](#project-overview)
2. [Business problem](#business-problem)
3. [Key findings](#key-findings)
4. [Report structure](#report-structure)
5. [Data model](#data-model)
6. [KPIs and DAX measures](#kpis-and-dax-measures)
7. [Data quality findings](#data-quality-findings)
8. [Design decisions](#design-decisions)
9. [Tech stack](#tech-stack)
10. [Repository structure](#repository-structure)
11. [How to use](#how-to-use)
12. [Limitations](#limitations)
13. [Author](#author)

---

## Project overview

The report answers one question: **where does revenue stop turning into profit?**

It covers 60 stores across 12 UK regions over 24 months (January 2023 to December 2024). Each page tackles one problem, then ends with a dynamic text box that states the finding and a recommended action.

| Page | Problem | Core question |
|---|---|---|
| 1. Profit Leakage | Revenue hides thin margins | Which stores earn their revenue, and where does each £1 go? |
| 2. Peak Demand vs Staffing | Labour is not matched to demand | Is staffing, the biggest controllable cost, aligned with demand? |
| 3. Structural Cost and Action Plan | Rent and staffing lock in losses | Which stores need action, and which cost should be fixed first? |

The report is **strategic, not descriptive**. Every visual has a question as its title, a subtitle for drill-down guidance, an info overlay explaining how to read it, and a KPI card with a one-line reference insight.

## Business problem

The challenge brief described an urban food-delivery operation (delivery speed, traffic, weather, SLA breaches, food temperature). The dataset supplied is a **coffee-store chain** (stores, footfall, staffing, rent, loyalty). The delivery-specific fields do not exist in the data, so building them would have meant inventing data.

I kept the brief's strategic structure and applied it to the schema provided:

- High volume does not always mean healthy profit.
- Peak-period capacity is hard to manage.
- Structural cost (rent and staffing) creates pressure that volume cannot fix.
- Cross-dimensional patterns (store, region, location, rent band) hide the riskiest stores.

## Key findings

All figures are network totals for the full 24 months, with no slicers applied.

| Finding | Evidence |
|---|---|
| Revenue and margin are flat | £89.7M revenue and £16.9M net profit; margin 18.8% overall (18.84% in 2023, 18.80% in 2024). |
| Fixed costs take 41p of every £1 | Staffing 31.0% and rent 10.4% of revenue, on top of COGS at 37.9% and loyalty at 1.9%. |
| 1 in 6 store-months makes a loss | 15.6% of store-months are loss-making. |
| Staffing is the main margin driver | Staffing % of revenue vs net margin has a correlation of about -0.96 at store-month level. |
| Labour is not matched to demand | Revenue per staff hour is about £49 across every shift pattern, and staff hours are uncorrelated with revenue. |
| Rent is a London problem | High rent band margin is 10.6% against 21.6% for Low; 143 of the 151 High-band store-months are in London, which has the weakest regional margin (8.5%). |
| Loyalty is not the margin lever | Loyalty share of spend vs margin has a correlation of about -0.01, and redemption cost is about 1.9% of revenue at every store. |
| A short list of stores needs action | 9 stores exceed the network on margin gap, staffing and rent at the same time. |

## Report structure

### Page 1: Profit Leakage
*Where profit leaks: fixed costs take 41p of every £1.*

| Visual | Question it answers |
|---|---|
| Scatter (revenue vs net margin, coloured by store status) | Which stores earn their revenue, and which only generate volume? |
| Waterfall (revenue to net profit) | Where does each £1 of revenue go before it becomes profit? |
| Revenue by location type, drill to region and city | Where does revenue concentrate? |
| Line and column (revenue and margin by month) | Is revenue growth actually improving margin? |

KPIs: Revenue, Net Profit, Net Margin %, Fixed Cost % Rev, Loss-Making Stores, Loss Store Revenue Share.

### Page 2: Peak Demand vs Staffing
*Staffing is the biggest controllable cost, and it is not matched to demand.*

| Visual | Question it answers |
|---|---|
| Matrix heatmap (day by shift, revenue per staff hour) | Does productivity change with the day or the shift? |
| Donut (peak-to-dead ratio by store format) | Does peak-hour pressure differ by format? |
| Scatter (staffing % vs margin, coloured by store status) | Does higher staffing cost erode margin? |
| Ribbon (revenue per staff hour by month, legend store status) | Does any store status earn consistently more per staff hour? |

KPIs: Revenue per Staff Hour, Staffing % Rev, Peak-to-Dead Ratio, Avg Dwell Time, Transactions per Staff Hour, Revenue per Footfall.

### Page 3: Structural Cost and Action Plan
*Structural losses sit in a few stores, and rent and staffing are the cause, not loyalty.*

| Visual | Question it answers |
|---|---|
| Clustered column (margin and rent % by rent band) | Does rent band drive losses? |
| Scatter (loyalty share vs margin, coloured by store status) | Does loyalty adoption change margin? |
| Net margin and rent % by location type, drill to region and city | Where do margin and rent pressure diverge? |
| Table (risk score, primary cost driver, priority action) | Which stores need action, and which cost do we fix first? |

KPIs: Loyalty Revenue %, Loyalty Cost % Rev, Rent % Rev, Gross Margin %, Stores Needing Action, Flagged Stores Margin Gap.

## Data model

Star schema with five tables. All relationships are one-to-many, single direction. `dim_date` is marked as the date table.

| Table | Grain | Rows |
|---|---|---|
| `dim_store` | One row per store | 60 |
| `dim_date` | One row per day (1 Jan 2023 to 30 Dec 2024) | 730 |
| `dim_product` | One row per product | 40 |
| `fact_store_monthly_profitability` | Store by month | 1,440 |
| `fact_daily_sales` | Store by day by product (**a sample**, see limitations) | 10,000 |

Relationships:

- `dim_store[store_id]` to both fact tables
- `dim_date[date_id]` to both fact tables
- `dim_product[product_id]` to `fact_daily_sales`

All revenue, cost, profit and margin measures use `fact_store_monthly_profitability`. `fact_daily_sales` is used only for staff hours, dwell time, footfall and day-type analysis.

A disconnected table, `Waterfall Lines`, supplies the bars for the waterfall visual.

## KPIs and DAX measures

A sample of the core measures:

```DAX
Revenue       = SUM(fact_store_monthly_profitability[total_monthly_revenue_gbp])
Net Profit    = SUM(fact_store_monthly_profitability[net_profit_gbp])
Net Margin %  = DIVIDE([Net Profit], [Revenue])
Fixed Cost % Rev = [Rent % Rev] + [Staffing % Rev]
Gross Margin %   = DIVIDE([Revenue] - [COGS], [Revenue])

Loss Months % =
DIVIDE(
    CALCULATE(COUNTROWS(fact_store_monthly_profitability),
              fact_store_monthly_profitability[net_profit_gbp] < 0),
    COUNTROWS(fact_store_monthly_profitability)
)

Store Risk Score =
VAR a = IF([Net Margin %] < [Network Margin %], 1, 0)
VAR b = IF([Staffing % Rev] > CALCULATE([Staffing % Rev], ALL(dim_store)), 1, 0)
VAR c = IF([Rent % Rev]     > CALCULATE([Rent % Rev],     ALL(dim_store)), 1, 0)
RETURN a + b + c

Priority Action =
SWITCH([Store Risk Score], 3, "Restructure", 2, "Monitor", "Maintain")
```

**Dynamic insight text.** Each page has a text box driven by a DAX measure, and each KPI card has a one-line reference measure. When no store filter is active, reference lines compare against the median store, because comparing a number with the whole network that contains it returns zero.

```DAX
Ref Rent % =
VAR bench = IF([Store Filtered],
                CALCULATE([Rent % Rev], ALL(dim_store)),
                MEDIANX(ALL(dim_store[store_id]), [Rent % Rev]))
VAR lbl = IF([Store Filtered], "network", "median store")
VAR gap = [Rent % Rev] - bench
RETURN FORMAT(ABS(gap) * 100, "0.0") & " pts " & IF(gap > 0, "above", "below") &
       " " & lbl & " (" & FORMAT(bench, "0.0%") & ")"
```

The full measure library is in [`/dax/measures.md`](dax/measures.md).

## Data quality findings

I audited the workbook before building and changed the design wherever the data did not support the original plan.

| Finding | Impact | Action taken |
|---|---|---|
| `fact_daily_sales` holds 10,000 rows covering about 20% of store-days (8,910 of 43,800), and its revenue totals £20.5M against £89.7M in the monthly table. | The two revenue totals cannot be compared directly. | All P&L measures use the monthly table; the daily table is labelled "sample basis". |
| Store-month revenue in the two fact tables is uncorrelated (r about 0). | The tables cannot be reconciled below network level. | Daily table used for ratios and patterns only. |
| Net profit equals revenue minus COGS, rent, staffing and loyalty cost exactly. | No unexplained cost. | The waterfall has no "other" bar. |
| Product category shows identical margin (about 63%) and revenue stream mix. | A product matrix would show no differences. | Product analysis removed from Page 3. |
| Store status is a current snapshot, but 3 non-active stores trade in all 24 months. | Status colour cannot be read as history. | Overlays state this explicitly. |
| Loyalty redemption cost is about 1.9% of revenue at every store. | A loyalty flag in the risk score was noise. | Risk score uses margin gap, staffing and rent only. |
| `store_format` has 58 High-Street and 2 Transport-Hub stores. | A format slicer is nearly useless. | `location_type` and `region` used for slicing. |
| Dimension rent and staffing differ from monthly values by up to 16% and 28%. | Double counting risk. | Monthly fact used for all costs. |
| 17 store-months have margins as low as -339%. | Outliers distort axes. | Axes capped; medians used where relevant. |

Figures in this README were cross-checked in Python against the source workbook. DAX results should be confirmed in your own Power BI model.

## Design decisions

- **One problem per page,** with at most 4 visuals plus KPI cards, so each page can be read in seconds.
- **Question-led titles.** Every visual title is the question it answers; subtitles guide drill-down.
- **Info overlays.** Each visual has an "i" button (bookmark-driven) explaining what it shows, how to read it, its data basis and its caveats.
- **Honest benchmarks.** Reference lines use a median-store benchmark when unfiltered.
- **Sample basis labels** wherever the daily table is used.
- **Restrained palette:** one neutral, one accent, and red reserved for losses.

## Tech stack

- Power BI Desktop (data model, DAX, bookmarks, drill-through)
- DAX (measures, calculated columns, dynamic text)
- Excel workbook as the source
- Python (pandas) for data validation

## Repository structure

Suggested layout:

```
.
├── README.md
├── report/
│   └── coffee-profitability.pbix
├── data/
│   └── data.xlsx
├── dax/
│   └── measures.md
└── docs/
    └── screenshots/
```

## How to use

1. Download or clone the repository.
2. Open `report/coffee-profitability.pbix` in Power BI Desktop.
3. If prompted, point the data source to `data/data.xlsx` (Home > Transform data > Data source settings).
4. Rename the monthly table to `fact_store_monthly_profitability`. Excel truncates the sheet name to `fact_store_monthly_profitabilit` (31-character limit).
5. Use the page slicers (Year, Region, rent band, location type) and the drill arrows on the location visuals.
6. Click the "i" buttons for guidance on each visual.

## Limitations

- **Daily sales is a sample** (about 20% of store-days). Treat staff-hour, dwell and day-type findings as patterns, not totals.
- **Peak and dead-hour transaction counts are partial** for each month, so only their ratio is meaningful.
- **Store status is a snapshot**, not a history.
- **Two years of data** limit year-on-year comparison.
- **The original brief described food delivery.** Delivery-specific analysis (traffic, weather, delivery time, food temperature) could not be built from the data supplied.
- Findings are observational correlations, not proof of cause.

## Author

**Emmanuel Idowu**
Senior Data Analyst and Power BI Developer | Founder, Manifest Data

- LinkedIn: _add link_
- YouTube: _add link_
- GitHub: _add link_

Built for the Onyx Data challenge.
