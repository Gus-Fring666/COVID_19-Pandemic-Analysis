# COVID-19 Pandemic Analysis

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Matplotlib-Visualization-blue?logo=plotly&logoColor=white" alt="Matplotlib">
  <img src="https://img.shields.io/badge/Scipy-Statistics-blue" alt="Scipy">
  <img src="https://img.shields.io/badge/Status-Completed-green" alt="Status">
</p>

<p align="center">
  <strong>An end-to-end exploratory data analysis and visualization of the COVID-19 pandemic</strong><br>
  Combining JHU CSSE case data with World Bank socioeconomic indicators to uncover trends across time, geography, and population health metrics.
</p>

---

## 📑 Table of Contents

- [About This Project](#about-this-project)
- [Problem Statement](#problem-statement)
- [Data Sources](#data-sources)
- [Project Structure](#project-structure)
- [Methodology](#methodology)
  - [1. Data Acquisition](#1-data-acquisition)
  - [2. Data Wrangling](#2-data-wrangling)
  - [3. Feature Engineering](#3-feature-engineering)
  - [4. Visualization](#4-visualization)
- [Key Findings](#key-findings)
- [Technical Specifications](#technical-specifications)
- [How to Reproduce](#how-to-reproduce)
- [Notebook Guide](#notebook-guide)
- [Author](#author)

---

## About This Project

This project provides a comprehensive analysis of the SARS-CoV-2 pandemic using open-source data. The analysis is organized into five Jupyter notebooks covering the full data pipeline — from raw data ingestion to publication-ready visualizations.

The project demonstrates:
- **Automated data pipelines** for daily epidemic surveillance
- **Multi-granularity analysis** (global, continental, national)
- **Socioeconomic correlation studies** linking pandemic metrics to World Bank indicators
- **Reproducible visualization architecture** via a custom `CovidDataViz` class

---

## Problem Statement

The COVID-19 pandemic spread rapidly across the world, but the pace and severity varied significantly across countries and continents. Key questions this project addresses:

1. How did the pandemic evolve globally and regionally from December 2019 to October 2020?
2. Which countries and continents were most severely impacted?
3. How do socioeconomic factors (GDP, healthcare expenditure, rural population, life expectancy) correlate with case rates and mortality?
4. What patterns emerge when examining mortality rates across different country profiles?

---

## Data Sources

| Source | Description | URL |
|--------|-------------|-----|
| JHU CSSE COVID-19 Data | Daily time series of confirmed, recovered, dead, and active cases by country | [GitHub](https://github.com/CSSEGISandData/COVID-19) |
| World Bank — GDP per capita PPP | Economic indicator by country | [data.worldbank.org](https://data.worldbank.org/indicator/NY.GDP.PCAP.PP.CD) |
| World Bank — Population | Total population | [data.worldbank.org](https://data.worldbank.org/indicator/SP.POP.TOTL) |
| World Bank — Urban Population | Urban population percentage | [data.worldbank.org](https://data.worldbank.org/indicator/SP.URB.TOTL.IN.ZS) |
| World Bank — Slum Population | Population living in slums | [data.worldbank.org](https://data.worldbank.org/indicator/EN.POP.SLUM.UR.ZS) |
| World Bank — Rural Population | Rural population percentage | [data.worldbank.org](https://data.worldbank.org/indicator/SP.RUR.TOTL.ZS) |
| World Bank — Life Expectancy | Life expectancy at birth | [data.worldbank.org](https://data.worldbank.org/indicator/SP.DYN.LE00.IN) |
| World Bank — Healthcare Expenditure | Current healthcare expenditure (% of GDP) | [data.worldbank.org](https://data.worldbank.org/indicator/SH.XPD.CHEX.GD.ZS) |
| Country & Continent Codes | Country-to-continent mapping | [Datahub](https://datahub.io/JohnSnowLabs/country-and-continent-codes-list) |

---

## Project Structure

```
covid19-pandemic-analysis/
├── data/
│   └── download_data.py              # Automated data download script
├── features/                         # Data processing pipeline
│   ├── make_all.py                   # Master pipeline runner
│   ├── make_cases.py                 # JHU CSSE case ingestion
│   ├── make_cases_daily_change.py    # Daily differencing
│   ├── make_cases_since_t0.py        # Reindexed time series
│   ├── make_continents.py            # Continent aggregation
│   ├── make_coordinates.py           # Geo coordinate mapping
│   ├── make_country_stats.py         # Country-level summaries
│   ├── make_country_to_continent.py  # Country-to-continent mapping
│   ├── make_mortality.py             # Mortality rate calculation
│   ├── make_world_bank.py            # World Bank indicator ingestion
│   └── utils.py                      # Shared utilities
├── visualizations/
│   └── covid_data_viz.py             # Reusable CovidDataViz plotting class
├── notebooks/                        # Analysis notebooks
│   ├── Data-wrangling.ipynb          # Data pipeline documentation
│   ├── Exploratory-analysis-globally.ipynb   # World & continent trends
│   ├── Exploratory_analysis_fancy_plot.ipynb  # Publication-ready visual
│   ├── Exploratory-analysis-mortality.ipynb   # Country-level mortality
│   └── Exploratory_analysis_socioeconomic.ipynb # World Bank correlations
└── tests/
```

---

## Methodology

### 1. Data Acquisition

The data acquisition pipeline is fully automated via `download_data.py` and the `features/` scripts:

- **JHU CSSE data** is fetched from the [CSSEGISandData/COVID-19](https://github.com/CSSEGISandData/COVID-19) repository. Daily CSV files (time_series_19-covid-*.csv) are downloaded and processed.
- **World Bank indicators** are retrieved using the `wbdata` Python library. The last available value for each indicator is extracted per country.
- **Country metadata** (names, codes, continents) is downloaded from Datahub and cached locally.

All raw and processed data is stored under `data/processed/` with the following outputs:

| Output File | Content |
|-------------|---------|
| `confirmed_cases.csv` | Cumulative confirmed cases by country over time |
| `confirmed_cases_daily_change.csv` | Daily new confirmed cases |
| `confirmed_cases_since_t0.csv` | Reindexed time series (days since 100th case) |
| `recovered_cases.csv` | Cumulative recovered cases |
| `dead_cases.csv` | Cumulative deaths |
| `active_cases.csv` | Active cases = confirmed − recovered − dead |
| `mortality_rate.csv` | Mortality rate = dead / confirmed |
| `coordinates.csv` | Country geo coordinates |
| `continents.csv` | Continent assignments |
| `country_to_continent.csv` | Country-to-continent lookup |
| `country_stats.csv` | Country-level summary statistics |
| `world_bank.csv` | World Bank socioeconomic indicators |

### 2. Data Wrangling

The raw JHU CSSE data undergoes several transformations:

- **Grouping:** Province/state-level rows are aggregated to country level.
- **Filtering:** Cases occurring on boats/cruise ships are excluded.
- **Derived metrics:**
  - `mortality rate = dead / confirmed`
  - `active cases = confirmed − recovered − dead`
- **Date handling:** Time series are parsed and aligned across all countries.
- **Missing values:** Forward-filled where appropriate; countries with insufficient data are flagged.

### 3. Feature Engineering

| Feature | Description |
|---------|-------------|
| `Confirmed chg` | Daily change in confirmed cases (first differencing) |
| `Confirmed t0` | Days since the 100th confirmed case (reindexing) |
| `Mortality` | Case fatality rate = dead / confirmed |
| `Country stats` | Aggregated totals and peak metrics per country |
| `World bank` | Merged socioeconomic indicators (GDP, population, healthcare, life expectancy) |
| `Ctry to cont` | Country-to-continent mapping for regional aggregation |

### 4. Visualization

The `CovidDataViz` class provides a unified interface for all plots:

- **World-level:** `plot_world_cases()` — stacked area chart of global cases
- **Continent-level:** `plot_continent_cases(continent)` — breakdown by continent
- **Country-level:** `plot_country_cases(country)`, `plot_country_cases_chg(country)` — time series and daily changes
- **Rankings:** `plot_highest_country_stats(metric)` — top-N countries by confirmed, recovered, dead, active, or mortality
- **Growth analysis:** `plot_growth(countries, periods)` — growth factor trends with configurable averaging windows
- **Socioeconomic:** `show_corr_mat()` — correlation matrix; `plot_with_slope(x, y)` — scatter plots with regression lines

---

## Key Findings

### Global & Continental Trends
- By **October 2020**, global confirmed cases approached **35 million**.
- **Asia** exhibited sustained exponential growth, driven largely by **India**.
- **Australia** had the lowest confirmed case count among continents.
- Most countries were able to suppress exponential growth within **30–40 days** of reaching their 100th case.
- **Italy** experienced a massive early surge; the **United Kingdom** showed evidence of a second wave during the analysis period.

### Country-Level Patterns
- Sharp jumps in time series reflect varying data collection methodologies across countries.
- Countries with highest confirmed cases: US, India, Brazil, Russia, and several European nations.
- Mortality rates varied widely, with outliers where risk-group demographics (e.g., elderly population density) played a significant role.

### Socioeconomic Correlations
| Indicator Pair | Correlation | Interpretation |
|----------------|-------------|----------------|
| Rural population % vs. Cases per million | **-0.46** | Virus spread more easily in densely populated urban areas |
| GDP healthcare expenditure vs. Deaths per million | **+0.38** | Likely reflects better testing/reporting capacity rather than worse outcomes |
| Life expectancy vs. Mortality % | Non-linear | Outliers to the right suggest risk-group demographics influence |

- **Yemen** was identified as a severe outlier and excluded from correlation analysis.
- No strong linear relationships were found between individual socioeconomic indicators and mortality, suggesting the need for continent-level or multivariate analysis.

### Key Metrics Summary
| Metric | Value | Context |
|--------|-------|---------|
| Analysis window | Dec 2019 – Oct 2020 | Early pandemic phase |
| Global confirmed cases | ~35 million | As of 2020-10-05 |
| Countries analyzed | 200+ | All countries with reported cases |
| Socioeconomic indicators | 7 | GDP, population, healthcare, life expectancy, rural/urban |
| Strongest correlation | -0.46 | Rural population % vs. cases per million |

---

## Limitations

- **Data latency:** JHU CSSE data reflects reported cases, not true infections; testing availability varied drastically across countries.
- **Socioeconomic snapshot:** World Bank indicators are static, year-level values — they do not capture within-year policy changes or healthcare system strain during the pandemic.
- **Correlation ≠ causation:** The observed correlations (e.g., rural population vs. cases) are ecological and do not imply individual-level causality.
- **Outlier sensitivity:** Yemen and other low-reporting countries were excluded, potentially biasing correlation estimates.
- **Time window:** Analysis is limited to the first 10 months of the pandemic; long-term trends and vaccine effects are not captured.

---

## Next Steps

- **Extend time window** to include 2021–2022 data and compare pre-/post-vaccination dynamics.
- **Multivariate regression** to isolate the effect of each socioeconomic indicator while controlling for confounders.
- **Phylogenetic / mobility data integration** (e.g., Google Mobility Reports) to model spread mechanics.
- **Interactive dashboard** using Plotly Dash or Streamlit for real-time exploration.
- **Automated daily update pipeline** deployed as a scheduled GitHub Actions workflow.

---

## Dataset Version

This analysis is pinned to the **JHU CSSE snapshot from October 2020**. To reproduce exact numbers:
- Use the `data/download_data.py` script with `--date 2020-10-05` or manually download the time series files from the [CSSEGISandData/COVID-19](https://github.com/CSSEGISandData/COVID-19) repository at commit `5d333ad`.

---

## Technical Specifications

### Data Pipeline Architecture
```
Raw JHU CSSE CSVs
       ↓
make_cases.py → confirmed_cases.csv, recovered_cases.csv, dead_cases.csv
       ↓
make_cases_daily_change.py → confirmed_cases_daily_change.csv
       ↓
make_cases_since_t0.py → confirmed_cases_since_t0.csv
       ↓
make_mortality.py → mortality_rate.csv
       ↓
make_continents.py + make_country_to_continent.py → continents.csv, country_to_continent.csv
       ↓
make_world_bank.py → world_bank.csv
       ↓
CovidDataViz class → reusable plotting methods
```

### Processing Notes
- **Boat cases removed:** Cruise ship incidents (Diamond Princess, etc.) excluded from country totals.
- **Date alignment:** All time series aligned to common date index.
- **Missing data handling:** Forward-fill for countries with intermittent reporting; zero-fill for unrecovered/dead where appropriate.

---

## How to Reproduce

### Prerequisites

```bash
pip install -r requirements.txt
```

### Step 1: Download Raw Data

```bash
python data/download_data.py
```

This script fetches:
- JHU CSSE daily case reports
- World Bank indicator metadata
- Country/continent code mappings

### Step 2: Generate Processed Features

```bash
python features/make_all.py
```

Or run individual feature scripts in dependency order:

```bash
python features/make_cases.py
python features/make_cases_daily_change.py
python features/make_cases_since_t0.py
python features/make_mortality.py
python features/make_continents.py
python features/make_coordinates.py
python features/make_country_stats.py
python features/make_country_to_continent.py
python features/make_world_bank.py
```

### Step 3: Run Analysis Notebooks

Open Jupyter Notebook or JupyterLab and execute the notebooks in `notebooks/`:

```bash
jupyter notebook
```

---

## Notebook Guide

| Notebook | Description | Key Outputs |
|----------|-------------|-------------|
| `Data-wrangling.ipynb` | Documents the complete data pipeline from raw sources to processed CSVs | Processed data files in `data/processed/` |
| `Exploratory-analysis-globally.ipynb` | World and continent-level trends; early pandemic dynamics | World case plots, continent breakdowns, growth factor analysis |
| `Exploratory-analysis-mortality.ipynb` | Country-level mortality analysis; time series of cases and daily changes | Country case plots, daily change plots, mortality rankings |
| `Exploratory_analysis_socioeconomic.ipynb` | World Bank correlations; mortality vs. socioeconomic indicators | Correlation matrix, scatter plots with regression slopes |
| `Exploratory_analysis_fancy_plot.ipynb` | Publication-ready heatmap visualization for repository README | `covid_tiles.png` (1600×800, 200 DPI) |

---

## Author

**Sarvesh Kumar Sharma**

- GitHub: [@shsarv](https://github.com/shsarv)
- LinkedIn: [in/shsarv](https://linkedin.com/in/shsarv)

---

<p align="center">
  <a href="../README.md">← Back to repository root</a>
</p>
