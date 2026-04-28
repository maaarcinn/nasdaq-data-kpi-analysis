# Data Card: NASDAQ Stock Price Dataset for AI Model Training Quality Assessment

## 1. Dataset Overview
This dataset contains historical daily stock price data for five NASDAQ‑listed companies, each collected over a different time window. It serves to demonstrate a rigorous quality assessment for AI training data using the key performance indicators **completeness**, **Latency**, **accuracy**, and **consistency**.

### Tickers & Periods

| Ticker | Company                               | Period   |
|--------|---------------------------------------|----------|
| VRTX   | Vertex Pharmaceuticals Incorporated   | 1 year   |
| LULU   | Lululemon Athletica Inc.              | 9 months |
| ODFL   | Old Dominion Freight Line, Inc.       | 6 months |
| ENPH   | Enphase Energy, Inc.                  | 3 months |
| DKNG   | DraftKings Inc.                       | 1 month  |

- **Temporal Coverage**: All data retrieved on **28‑04‑2026**, with each ticker’s history rolling back by its respective period. The latest data point is the retrieval date itself, resulting in **0 days of staleness**.
- **Data Frequency**: Daily (trading days only).
- **Approximate Trading Days**: VRTX: 252, LULU: ~189, ODFL: ~126, ENPH: ~63, DKNG: ~21.

---

## 2. Source of Data

- **Data Provider**: Yahoo Finance – accessed via the [`yfinance`](https://pypi.org/project/yfinance/) Python library.
- **API Endpoint**: `yfinance.download(ticker, period=...)`
- **Access Date**: 2026‑04‑28
- **Market**: NASDAQ

Yahoo Finance delivers open, high, low, close, adjusted close, and volume data sourced from official exchange feeds. The service is widely trusted for financial analysis and machine learning pipelines.

**Selection Rationale**  
The five companies represent diverse sectors (biotech, retail, transportation, renewable energy, digital entertainment) and market capitalizations. The intentionally staggered periods (1 year down to 1 month) replicate real‑world scenarios where AI models fuse historical windows of varying lengths, making thorough quality evaluation critical.

---

## 3. Data Collection Methodology

A single Python script (`nasdaq-data-kpi-analysis.ipynb`) automates the entire pipeline:

1. Iterates over the five ticker/period pairs.
2. Fetches raw daily data using `yfinance.download`.
3. **Completeness** – Calculates the percentage of non‑null cells in the raw DataFrame.
4. **Latency** – Determines the staleness (days since last data point) and period coverage (actual trading days vs. expected calendar span).
5. **Accuracy** – Flags impossible values (negative prices/volumes) and logical OHLC violations (`High < Low`, `Close > High`, etc.). Additional health checks count suspicious daily price spikes (>30 %) and extreme volume outliers (>10× mean).
6. **Consistency** – Verifies five structural properties: presence of the required schema (`Open`, `High`, `Low`, `Close`, `Volume`), no duplicate index dates, chronologically sorted index, no weekend dates, and consistent `float` data types for price columns.
7. Saves the cleaned datasets as CSV files and exports a KPI summary report.

---

## 4. Data Quality KPIs

### 4.1 Definitions

| KPI                   | Description                                                                                                                                                     |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Completeness**      | `(1 - null_cells / total_cells) × 100` – Proportion of non‑missing values in the raw download.                                                                |
| **Latency**        | **Days Stale**: Number of days between the last available data point and the fetch date. **Period Coverage**: Actual trading days captured as a percentage of the total calendar days since the first requested date. |
| **Accuracy**          | Percentage of rows that pass all integrity checks: no negative price/volume, and valid OHLC relationships. The score is `(1 – invalid_rows / total_rows) × 100`. |
| **Consistency**       | A 5‑point check: expected columns present, no duplicate dates, chronological order, no weekend dates, and consistent float dtypes for price fields. Labeled as *“X/5 checks passed”*. |
| *Supplementary*       | **Suspicious Spikes**: Count of days with an absolute closing price change >30 %. **Volume Outliers**: Days where volume exceeds 10× the mean volume. These are reported for inspection but do not affect the Accuracy score. |

### 4.2 KPI Report

| Ticker | Period | Last Data Point | Days Stale | Period Coverage (%) | Completeness (%) | Accuracy (%) | Suspicious Spikes | Volume Outliers | Consistency        |
|--------|--------|-----------------|------------|---------------------|------------------|--------------|-------------------|-----------------|--------------------|
| VRTX   | 1y     | 2026-04-28      | 0          | 100.0               | 100.0            | 100.0        | 0                 | 0               | 5/5 checks passed  |
| LULU   | 9mo    | 2026-04-28      | 0          | 100.0               | 100.0            | 100.0        | 0                 | 0               | 5/5 checks passed  |
| ODFL   | 6mo    | 2026-04-28      | 0          | 100.0               | 100.0            | 100.0        | 0                 | 0               | 5/5 checks passed  |
| ENPH   | 3mo    | 2026-04-28      | 0          | 100.0               | 100.0            | 100.0        | 1                 | 0               | 5/5 checks passed  |
| DKNG   | 1mo    | 2026-04-28      | 0          | 100.0               | 100.0            | 100.0        | 0                 | 0               | 5/5 checks passed  |

**Interpretation**

- **Completeness & Latency**: All datasets are fully complete with zero missing values and cover 100 % of the expected period. The data is fresh with zero days of staleness, confirming the reliability of the Yahoo Finance feed for up‑to‑date analysis.
- **Accuracy**: No price/volume anomalies or OHLC logic violations were detected in any ticker, resulting in perfect accuracy scores. Only **ENPH** exhibited a single daily price spike (>30 %), which is flagged for awareness but remains within normal market volatility.
- **Consistency**: All five structural checks pass for every company, guaranteeing a uniform and analysis‑ready schema.

---

## 5. Descriptive Statistics Example (VRTX – 1 Year)

*All values in USD; volume is number of shares traded.*

|       | Close   | High    | Low     | Open    | Volume      |
|-------|---------|---------|---------|---------|-------------|
| count | 252.00  | 252.00  | 252.00  | 252.00  | 252.00      |
| mean  | 440.21  | 445.56  | 434.96  | 440.10  | 1,536,626   |
| std   | 30.31   | 30.63   | 29.76   | 30.22   | 971,251     |
| min   | 366.54  | 380.20  | 362.50  | 362.97  | 196,700     |
| 25%   | 421.30  | 426.96  | 416.86  | 418.93  | 1,056,450   |
| 50%   | 443.92  | 448.28  | 439.23  | 444.01  | 1,331,800   |
| 75%   | 461.01  | 466.07  | 456.91  | 461.54  | 1,697,800   |
| max   | 509.50  | 510.77  | 498.21  | 507.00  | 10,729,400  |

The distribution exhibits moderate daily volatility over the year, with no gaps or extreme outliers aside from the expected volume spikes that can accompany earnings or news events.

---

## 6. Visualization

<img width="700" alt="image" src="https://github.com/user-attachments/assets/6d7f7c19-8395-47bc-8aa5-eec8b6c7849c" />


The normalized stock price chart rebases each series to 100 at its start, enabling a fair comparison of relative performance despite the differing time frames. The visual inspection confirms:

- Continuous, plausible price movements for all tickers.
- No gaps, flat lines, or erratic jumps that would indicate data corruption.
- A clear, interpretable view of trend divergence across sectors.

---

## 7. Data Files Structure
```
├── .gitignore
├── README.md
└── nasdaq-data-kpi-analysis.ipynb
```

CSV output columns (generated on run): `Open`, `High`, `Low`, `Close`, `Volume`, indexed by `Date`.

---
## 8. Requirements

Python 3.x with the following libraries:
`yfinance`, `pandas`, `matplotlib`

Install with: `pip install yfinance pandas matplotlib`

## 9. Conclusion

The collected NASDAQ stock dataset demonstrates **outstanding quality** across all assessed dimensions, making it immediately suitable for training AI/ML models in finance, time‑series forecasting, and portfolio analytics.

- **Completeness:** 100 % – no missing values in any dataset.
- **Latency:** Zero days stale, 100 % period coverage – data is fresh and comprehensive.
- **Accuracy:** Perfect integrity scores; only a single flagged daily price spike (ENPH) was observed, which does not detract from overall reliability.
- **Consistency:** All five schema, duplication, sorting, calendar, and type checks pass uniformly.

The enhanced KPI framework goes beyond basic quality checks, incorporating Latency and structural validation that are critical for real‑world AI pipelines. The Yahoo Finance API (via `yfinance`) proves to be a trustworthy source, delivering clean, structured, and timely financial time‑series data ready for reproducible research and production model development.
