# Cohort & Retention Analysis — Online Retail II

> **Which customers stopped buying, when did they leave, and why?**
> A full-cycle analytics project on 1M UK e-commerce transactions: Python cleaning → cohort retention matrix → SQL-driven churn drivers → Tableau dashboard.

**Stack:** Python (Pandas, DuckDB) · SQL · Tableau Desktop
**Dataset:** [UCI Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) — UK e-commerce, Dec 2009 → Dec 2011, ~1.07M transactions, 5,878 unique customers
**Dashboard:** *Add your Tableau Public link here once published* → `[View dashboard](https://public.tableau.com/...)`

---

## TL;DR — three churn drivers found

| # | Driver | Headline number |
|---|--------|----------------|
| 1 | **One-and-done customers** | **27.6%** of customers placed exactly one order and never returned |
| 2 | **Geography** | Spain retains at **35%** vs UK baseline **47%** (75% of UK rate); Germany over-performs at **60%** |
| 3 | **Holiday-season acquisitions** | Customers acquired in Nov/Dec retain **9 percentage points** below the rest-of-year average — gift buyers don't become regulars |

**Bonus signal:** customers who returned spent £485 on their first order vs £355 for those who churned. **First-order value is a leading indicator of stickiness.**

---

## The business question

A UK-based online retailer ran from December 2009 to December 2011 selling unique gifts and home goods. After two years of sales they wanted to understand:

> **Which customers stopped buying, when did they leave, and why — and what should we do about it?**

Three deliverables answered the question:

1. A **cohort retention matrix** showing how each acquisition month's customers behave over time.
2. A **churn driver analysis** isolating the structural reasons customers don't come back.
3. A **Tableau dashboard** that tells the story in 30 seconds for non-technical stakeholders.

---

## Approach

| Phase | Tooling | What happens | Output |
|-------|---------|-------------|--------|
| 1. Get the data | — | Download UCI Online Retail II | `data/raw/online_retail_II.xlsx` |
| 2. Clean | Python + Pandas | Drop nulls / cancellations / non-positive rows; add Revenue; parse dates | `data/clean/online_retail_clean.parquet` |
| 3. Cohort matrix | Python + Pandas | Assign each customer a cohort month; compute month-since-acquisition retention | `cohort_retention.csv` + heatmap PNG |
| 4. Churn drivers | SQL via DuckDB | Single-order share · 90-day retention by country · seasonality of acquisition · AOV signal | `churn_drivers_summary.csv` + chart PNG |
| 5. Tableau dashboard | Tableau Desktop | KPI cards + heatmap + country bars + seasonality line | `outputs/tableau_cohort_retention.twbx` |

### Methodology highlights

- **Cohort definition:** a customer's cohort = the month of their first purchase. `CohortIndex = months_between(invoice_month, cohort_month)` — `0` is the acquisition month.
- **Retention metric:** % of a cohort's month-0 customers active in month *N*. Top-left of the heatmap is always 100% by definition.
- **Right-censoring control:** for the churn-driver comparisons, only customers acquired ≥ 90 days before the dataset's last date are eligible. Without this, late-acquired customers look "churned" simply because they didn't have time to come back.
- **SQL via DuckDB:** every driver query is a standalone `.sql` file under [`sql/`](sql/) that reads the cleaned Parquet directly — no separate database needed. The same queries run unchanged on BigQuery / Snowflake.

---

## Findings in detail

### Driver 1 — One-and-done customers (27.6%)

Of 5,878 unique customers, 1,623 placed exactly one invoice and disappeared. They represent **the single largest churn pattern** in the data. Median orders per customer is 3, but the distribution is heavily right-skewed.

### Driver 2 — Geography is bimodal

90-day retention by country (eligible customers only, n ≥ 30):

| Country | Customers | 90d retention | vs UK |
|---------|-----------|---------------|-------|
| 🇩🇪 Germany | 93 | **60.2%** | +13 pp |
| 🇫🇷 France | 76 | **56.6%** | +10 pp |
| 🇬🇧 United Kingdom | 4,813 | **46.9%** | — |
| 🇪🇸 Spain | 34 | **35.3%** | −12 pp |

International isn't uniformly worse: Germany and France over-perform, Spain under-performs. The story isn't "domestic vs export" — it's that **local fit varies by market and the UK book itself has room to grow**.

### Driver 3 — Holiday-season acquisitions don't stick

The right-most panel shows it best: customers acquired in **November / December retain ~37%** versus **~46% the rest of the year**. The most extreme case is the **Dec 2010 cohort (n=76) which retained only 16% within 90 days**. Classic gift-buyer pattern — Christmas shoppers don't become regulars.

### Bonus — First-order value predicts retention

| Outcome | Customers | Avg first-order value | Median first-order value | Avg lifetime revenue |
|---------|-----------|----------------------|--------------------------|---------------------|
| Returned within 90d | 2,491 | **£485** | £316 | **£5,647** |
| Churned (no return in 90d) | 2,790 | £355 | £258 | £1,173 |

Customers who eventually returned spent **37% more** on their first order. First-order value is a free leading indicator the retailer can act on in the first week.

---

## Recommendations

1. **Win-back campaign targeted at the one-and-done segment.** 1,623 customers placed exactly one order. A modest email + discount sequence aimed at this cohort within 60 days of the gap would test whether the churn is product-fit, post-purchase friction, or simple inertia. Even a 5% reactivation rate = ~80 reactivated customers, each historically worth ~£300 lifetime.

2. **Localised onboarding for Spain.** Spain retains at 75% of the UK rate despite higher average first-order value (£629 vs UK's £387). The customers spend more, then leave — suggesting a post-purchase issue (shipping, returns, language) rather than acquisition mismatch. Pilot translated post-purchase emails and a Spain-specific FAQ before scaling acquisition spend.

3. **Treat Nov/Dec acquisitions as a separate cohort with its own playbook.** Don't expect gift buyers to behave like core customers. Run a January re-engagement flow with non-gift product recommendations (the customer probably bought *for* someone else). The Dec 2010 cohort collapse to 16% retention is a known pattern, not a fluke — design for it.

4. **Use first-order value as an early signal.** Customers placing first orders > £400 are 2.4× more likely to return. Reserve premium onboarding (handwritten thank-you note, loyalty perk, early access) for that segment in the first 14 days.

---

## Repository structure

```
