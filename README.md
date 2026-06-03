# TÜİK Forecasting Project — Cinema Audience in Türkiye

## 1. Project Overview

This project produces a one-step-ahead forecast for the annual total cinema audience in Türkiye using time series methods applied to data published by TÜİK. Six applicable forecasting methods are compared on standard accuracy measures, and the most suitable method is selected on both numerical and structural grounds. The forecast target is the next unpublished period: 2026.

## 2. Data Source and TÜİK Connection

Data are accessed directly from the TÜİK Data Portal through the `tuikr` R package. No data file is bundled with this repository; the source Excel sheet is fetched at knit time using the URL returned by `tuikr::statistical_tables()` and downloaded inside R with `httr::GET()`.

| Field | Value |
|---|---|
| TÜİK dataset name | Sinema, Gösterilen Film ve Seyirci Sayısı |
| TÜİK theme/category | Kültür ve Spor (theme_id = 10) |
| TÜİK table name | Sinema, Gösterilen Film ve Seyirci Sayısı |
| `tuikr` dataflow ID | Not available — the table is published as a downloadable file (`istab`), not as an SDMX dataflow. Access URL retrieved via `tuikr::statistical_tables("10")`. |
| Selected variable | Total cinema audience (annual) |
| Data frequency | Annual |
| Time coverage | 2000–2025 |
| Latest available observation | 2025 |
| Forecast target period | 2026 |
| Date of data access | 3 June 2026 |
| R package for data access | `tuikr` |
| Package source | https://github.com/emraher/tuikr |

The cinema table has no SDMX dataflow on the TÜİK portal; only a file-download (`istab`) entry exists. The download URL is obtained programmatically from `tuikr` and the file is fetched via `httr::GET()` inside R. This access pattern was confirmed with the course instructor before adoption.

## 3. Research Objective

The selected variable is the total number of cinema viewers in Türkiye per year. The audience figure is a useful summary indicator of the cinema sector: it reflects both supply-side conditions (number of films released, number of screens) and demand-side conditions (consumer behavior, ticket prices, competing entertainment options). Forecasting it for 2026 is meaningful for the Ministry of Culture and Tourism, for distributors planning release schedules, and for cinema operators sizing their inventory.

## 4. Use of TÜİK Data in R

No manually prepared, edited, or external data file was used. All preparation steps run inside the R notebook:

- **Selected variable:** total annual cinema audience (column 9 of the source Excel).
- **Time variable:** year (column 1 of the source Excel).
- **Frequency confirmation:** all rows represent one calendar year.
- **Period ordering:** rows are sorted by year ascending via `dplyr::arrange()`.
- **Filtering applied:** only data rows (rows 5–30 of the raw sheet) are kept; header rows and footer notes are dropped programmatically by row index.
- **Type conversion:** `year` cast to integer, `audience` cast to numeric.
- **Data quality checks:** missing values, duplicate years, and row count are verified with `stopifnot()` after cleaning.
- **Time series object:** the cleaned vector is wrapped in a `ts()` object with `start = 2000, frequency = 1`.

## 5. Exploratory Time Series Analysis

The series shows four structural periods:

- **Stagnant (2000–2007):** audience oscillates between 14M and 23M; no clear direction.
- **Growth (2008–2019):** sustained upward trend; peak of 68.5M in 2017.
- **COVID shock (2020–2021):** collapse to 17M and then 12M as cinemas close.
- **Recovery (2022–2025):** partial rebound stabilizing around 28–35M, well below pre-pandemic levels.

The series has trend (long-run upward direction interrupted by a sharp 2020 break), no seasonality (data are annual), and two clear outliers (2020, 2021) driven by the pandemic. No missing values; no duplicate years.

## 6. Forecasting Methods Applied

| Method | Status |
|---|---|
| Naïve Forecasting | Applied |
| Moving Average (3-year window) | Applied |
| Weighted Moving Average (0.2/0.3/0.5) | Applied |
| Exponential Smoothing | Applied |
| Trend-Adjusted Exponential Smoothing (Holt) | Applied |
| Linear Trend Projection | Applied |
| Seasonal Indices | Not applicable — annual data, no within-year cycle |
| Additive Decomposition | Not applicable — requires `frequency > 1` |
| Multiplicative Decomposition | Not applicable — same as above |
| Regression w/ Trend and Seasonal Dummies | Not applicable — no seasons to encode |

The four omitted methods all require a seasonal structure that annual data do not have. Each omission is documented and justified in the notebook.

## 7. Forecast Accuracy Comparison

All six applicable methods are evaluated on six measures: Bias / Mean Error, MAD, MSE, MAPE, RSFE, and Tracking Signal. The full comparison table is in `outputs/tables/accuracy_comparison.csv` and is reproduced in the notebook. Each method also has its own actual-vs-forecast plot in `outputs/figures/`.

## 8. Selection of the Superior Method

**Selected superior method: Exponential Smoothing.**

It produces the lowest MAPE among the applicable methods, and its Tracking Signal lies well within the conventional ±4 band, indicating no systematic bias. The choice is also defensible on structural grounds: the optimized smoothing parameter places most of the weight on recent observations, which is the appropriate response to a series with a 2020 regime shift. Methods that treat all observations equally — most notably Linear Trend — produce forecasts of roughly 40M for 2026, well above the 28–35M range the series has actually held since 2022.

## 9. Final Next-Period Forecast

- **Selected superior method:** Exponential Smoothing
- **Date of data access:** 3 June 2026
- **Latest available TÜİK observation:** 2025 — 27,657,591 viewers
- **Forecast target period:** 2026
- **Forecasted value:** approximately 27,658,079 viewers

## 10. Interpretation of Results

The 2026 forecast is essentially flat against 2025. The post-pandemic recovery has plateaued: audience figures since 2022 have hovered between 28M and 36M, with no sign of returning to the 56–68M range observed in 2017–2019. The model reads this as the new equilibrium and projects it forward. Whether the market eventually recovers further or has settled permanently at this lower level is a question only future data can resolve.

## 11. Limitations

- **Short post-break window.** Only four observations exist for the post-COVID regime, limiting how confidently any method can describe the new normal.
- **The winning method nearly collapses to Naïve.** The optimizer chose an α close to 1, meaning the forecast effectively repeats the most recent observation. The numerical win is real, but no method here extracted structure beyond level.
- **Roughly 23% MAPE is high in absolute terms.** Any point forecast comes with a band of several million viewers in uncertainty.
- **Future shocks are not modeled.** Streaming competition, ticket price changes, or another public-health disruption could push 2026 well outside historical patterns.
- **TÜİK methodology shift.** From the 2024 release, the source moved to administrative records from the Ministry of Culture and Tourism. The audience figures appear continuous, but other variables on the same table show large discontinuities of administrative origin.
- **Annual frequency.** Four of the ten required methods are inapplicable on structural grounds — not for lack of effort but for lack of seasonality.

## 12. Reproducibility

To rerun this project from a clean machine:

1. Clone this repository.
2. Open the `.Rproj` file in RStudio.
3. Run `renv::restore()` to install the locked package versions.
4. Knit `forecasting_project.Rmd` to HTML.

No data file is bundled. The Excel sheet is downloaded fresh from TÜİK at knit time using the URL returned by `tuikr::statistical_tables()`. Required packages: `tuikr`, `httr`, `readxl`, `dplyr`, `ggplot2`, `forecast`, `scales`, `knitr`. Versions are pinned in `renv.lock`.

## 13. Repository Structure

```
tuik-forecasting-project/
├── README.md
├── forecasting_project.Rmd
├── forecasting_project.html
├── outputs/
│   ├── tables/
│   │   ├── accuracy_comparison.csv
│   │   └── final_forecast.csv
│   └── figures/
│       ├── actual_series_plot.png
│       ├── naive_forecast_plot.png
│       ├── moving_average_plot.png
│       ├── weighted_moving_average_plot.png
│       ├── exponential_smoothing_plot.png
│       ├── trend_adjusted_smoothing_plot.png
│       ├── trend_projection_plot.png
│       └── superior_method_plot.png
├── R
├── renv.lock
└── .gitignore
```

## 14. Student Information

- **Student name:** Sueda Turgut
- **Student number:** 138721013
- **Course:** Quantitative Analysis for DM
- **My GitHub account:** https://github.com/suedat
