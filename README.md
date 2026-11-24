# **Effect of weather and climate change on commodity prices.**

Team 24 data science project on **how weather and climate change affect commodity prices**. 

## Project stages

| Stage | Section | Status |
|---|---|---|
| 1. Data collection | [1. Data collection](#1-data-collection) | In progress |
| 2. Cleaning | [2. Cleaning](#2-cleaning) | To do |
| 3. Analysis | [3. Analysis](#3-analysis) | To do |

Every choice that shapes the data or the results is recorded, with its reasons, in the **[decision log](docs/decision_doc.md)**.

## Scope

- **Frequency:** monthly
- **Period:** January 1990 to October 2026 (442 months)
- **Commodities:** maize, US HRW wheat, soybeans, arabica coffee, cocoa, world sugar, US and European natural gas; Brent crude is a control variable
- **Weather:** one point per key producing region (crop commodities) or demand centre (natural gas)
- **Climate:** ENSO (ONI and RONI) and the global temperature anomaly (GISTEMP)
- **Focus:** cocoa, with Côte d'Ivoire and Ghana as the producing regions. The other commodities and regions are kept for comparison.

## 1. Data collection

Raw data from eight public datasets, plus pre-processed tables that the cleaning stage starts from. Each table covers one thing; there is no joined panel.

### Datasets

Eight public datasets, all free.

| # | Dataset | Publisher | What we take from it | Native frequency | Access | Licence | Raw file(s) |
|---|---|---|---|---|---|---|---|
| 1 | **Commodity Price Data ("Pink Sheet"), monthly historical** | World Bank, Prospects Group | Nominal USD monthly average prices for the 9 series in [Prices](#prices) | Monthly | Manual download of `CMO-Historical-Data-Monthly.xlsx`, sheet *Monthly Prices* ([page](https://www.worldbank.org/commodities)) | CC BY 4.0 | `data/raw/prices/CMO-Historical-Data-Monthly.xlsx` |
| 2 | **ERA5 / ERA5-Land reanalysis**, via the **Open-Meteo Historical Weather API** | ECMWF Copernicus Climate Change Service; served by Open-Meteo | Daily 2 m mean, max and min temperature and daily precipitation at 10 points ([Weather](#weather)), aggregated to monthly | Daily (from hourly) | `archive-api.open-meteo.com/v1/archive` | CC BY 4.0 | `data/raw/weather/weather_daily_<region>.csv` |
| 3 | **Oceanic Niño Index (ONI), ERSSTv5** | NOAA Climate Prediction Center | 3-month running Niño 3.4 SST anomaly (°C) | Monthly (overlapping 3-month seasons) | `cpc.ncep.noaa.gov/data/indices/oni.ascii.txt` | US public domain | `data/raw/climate/oni.ascii.txt` |
| 4 | **Relative Oceanic Niño Index (RONI)** | NOAA Climate Prediction Center | Niño 3.4 anomaly relative to the tropical-mean SST anomaly (°C). NOAA's official ENSO index since Feb 2026. | Monthly (overlapping 3-month seasons) | HTML table at the CPC RONI page | US public domain | `data/raw/climate/roni_table.html` |
| 5 | **GISS Surface Temperature Analysis v4 (GISTEMP)**, global land-ocean means | NASA Goddard Institute for Space Studies | Global mean surface temperature anomaly vs 1951–1980 (°C) | Monthly | `data.giss.nasa.gov/gistemp/tabledata_v4/GLB.Ts+dSST.csv` | US public domain | `data/raw/climate/GLB.Ts+dSST.csv` |
| 6 | **US price indices: CPI-U and PPI** | US Bureau of Labor Statistics (BLS) | CPI all items (`CUUR0000SA0`, the deflator), CPI candy and chewing gum (`CUUR0000SEFR02`), PPI chocolate and chocolate-type confectionery (`WPU02550301`) | Monthly | BLS public API v1 (`api.bls.gov/publicAPI/v1`) | US public domain | `data/raw/price_indices/bls_<start>_<end>.json` |
| 7 | **Commitments of Traders (COT), Disaggregated, futures only**: ICE cocoa | US Commodity Futures Trading Commission (CFTC) | Open interest and long/short positions of managed money (funds), producers/merchants and swap dealers in cocoa futures | Weekly (Tuesday positions) | CFTC Public Reporting API, dataset `72hh-3qpy`, contract code `073732` | US public domain | `data/raw/positions/cot_cocoa_disaggregated_weekly.csv` |
| 8 | **FAOSTAT Crops and livestock products (QCL)**: cocoa beans | UN Food and Agriculture Organization (FAO) | Production (t), area harvested (ha) and yield (kg/ha) of cocoa beans in Côte d'Ivoire and Ghana | Annual | FAOSTAT bulk download (the API needs a login) | CC BY 4.0 | `data/raw/production/faostat_cocoa_civ_ghana.csv` |

**How each maps to the research question:**
- **(1)** gives the outcome variables: prices of weather-exposed crops and of natural gas, a heating/cooling-demand commodity. Brent crude is the control for general energy and macro moves.
- **(2)** gives local weather shocks in the main producing regions (crops) or demand centres (gas).
- **(3) and (4)** give the dominant mode of interannual climate variability. El Niño and La Niña shift rainfall and temperature across the tropics and the Americas.
- **(5)** gives the slow warming trend, the "climate change" signal.
- **(6)** follows a cocoa shock down the chain: cocoa beans → chocolate makers' prices (PPI) → what shoppers pay for candy (CPI). CPI all items removes general inflation from every price.
- **(7)** shows whether investors (managed money) bet on cocoa before or after weather events.
- **(8)** is the step between weather and price: bad weather → smaller harvest → higher price.

**Suggested citations:** World Bank (2026), *Commodity Markets Outlook*, Pink Sheet data. Hersbach et al. (2020), "The ERA5 global reanalysis", *QJRMS* 146:1999–2049, accessed via Open-Meteo (Zippenfeld, 2023). NOAA CPC ONI and RONI. GISTEMP Team (2026), GISS Surface Temperature Analysis v4, NASA GISS. US BLS CPI and PPI. CFTC Commitments of Traders. FAO (2025), FAOSTAT Crops and livestock products.

### Data folders

```
data/
├── raw/                  untouched downloads, exactly as the source published them
│   ├── prices/           World Bank Pink Sheet (1 file)
│   ├── weather/          daily weather, one file per region (10 files)
│   ├── climate/          ONI, RONI, GISTEMP (3 files)
│   ├── price_indices/    US CPI and PPI from BLS (4 files)
│   ├── positions/        CFTC cocoa futures positions (1 file)
│   └── production/       FAOSTAT cocoa production, Côte d'Ivoire and Ghana (1 file)
└── pre_processed/        reshaped to one common format, NOT cleaned (7 files)
```

#### Raw vs pre-processed

`raw/` is the evidence: each file is exactly what the source served, in its own layout (Excel with title rows, daily rows, a text table, an HTML page). These files are never edited.

`pre_processed/` holds the same information converted into one common shape. Every file is a CSV, keyed by `month` (the first day of the month, `YYYY-MM-01`), starting in January 1990, with short snake_case column names. The one exception is `cocoa_production_annual.csv`, keyed by `year` because FAO publishes yearly. Each file stays a separate table; nothing is joined. The table below lists everything that was done between raw and pre-processed. Anything not listed was **not** done.

| | Done in pre-processing | Left for cleaning |
|---|---|---|
| **Prices** | Kept 9 of the 71 Pink Sheet series and renamed them. Converted `1990M01` to `1990-01-01`. Dropped title rows and data before 1990. Turned the Pink Sheet's missing markers (`..`, `…`) into empty cells. Moved units into a separate file. | Prices are **nominal** USD, not deflated, logged or differenced. Units differ between series ($/mt, $/kg, $/mmbtu, $/bbl). |
| **Weather** | Aggregated daily values to monthly (rules below). Added degree days, heat-stress days, dry days and anomalies against 1991–2020. Stacked the 10 regions into one long table. | No outlier checks. One grid point stands in for each region. No crop-calendar or seasonal weighting. |
| **Climate** | Mapped each 3-month ENSO season to its centre month (DJF→Jan, JFM→Feb, …, NDJ→Dec). Turned GISTEMP's year × month grid into one row per month. Dropped GISTEMP's annual and seasonal columns and its `***` missing markers. | The latest ENSO values are provisional; NOAA revises them for about two months. |
| **Price indices** | Kept only the monthly values (dropped BLS's annual averages). Put the three series side by side, one row per month, 1990-01 to 2026-10. BLS's "not collected" marker (`-`) became an empty cell. | Indices, not prices in dollars, with different base periods. Not rebased or deflated. |
| **Positions** | Averaged the weekly reports within each calendar month. Kept 8 of the 194 CFTC columns (open interest; long, short and spread positions by trader type). No new columns calculated. | The first and last months are partial (fewer weekly reports; see `n_reports`). |
| **Production** | Kept cocoa beans for Côte d'Ivoire and Ghana from 1990. Put production, area harvested and yield side by side, one row per year, using FAO's values as published. Kept FAO's flag for each production value. | Annual only, so it does not line up with the monthly tables. Ghana 2022–2024 are flagged `X` (figures from an external organization, not official). |

**In every table, missing values are left as empty cells (NaN).** Nothing is imputed.

Weather aggregation rules (daily → monthly):

| Monthly column | From the daily data |
|---|---|
| `temp_mean_c` | mean of daily mean temperature |
| `temp_max_c` / `temp_min_c` | highest daily max / lowest daily min in the month |
| `precip_mm` | sum of daily precipitation |
| `hdd` / `cdd` | sum over days of max(0, 18.3 − daily mean) / max(0, daily mean − 18.3) |
| `heat_days` | count of days with daily max ≥ 35 °C |
| `dry_days` | count of days with precipitation < 1 mm |
| `temp_anom_c` / `precip_anom_mm` | monthly value minus the 1991–2020 average for the same calendar month at that region |

#### `data/raw/prices/`

| File | Contents |
|---|---|
| `CMO-Historical-Data-Monthly.xlsx` | World Bank Pink Sheet, update of 3 November 2026. We use the sheet **Monthly Prices**: rows are months from `1960M01` (format `YYYYMmm`), columns are 71 commodities. The first rows are a title block, then a row of series names and a row of units. The other sheets (*Monthly Indices*, *Description*, *Index Weights*, *Mismatch Details*) are not used. |

#### `data/raw/weather/`

Ten files named `weather_daily_<region>.csv`, one per region in the [region table](#weather). All ten have the same layout: one row per day from 1990-01-01 to 2026-10-31 (13,453 rows).

| Column | Meaning |
|---|---|
| `time` | date (UTC) |
| `temperature_2m_mean` / `_max` / `_min` | daily mean, maximum and minimum air temperature at 2 m (°C) |
| `precipitation_sum` | daily total precipitation (mm) |

#### `data/raw/climate/`

| File | Contents |
|---|---|
| `oni.ascii.txt` | NOAA CPC ONI, plain-text table from 1950. Columns: `SEAS` (3-month season, e.g. `DJF`), `YR`, `TOTAL` (absolute Niño 3.4 SST, °C), `ANOM` (anomaly, °C). Only `ANOM` is used. Ends at ASO 2026. |
| `roni_table.html` | Saved copy of the NOAA CPC RONI web page. The data is an HTML table with rows = years from 1950 and columns = the 12 seasons `DJF` … `NDJ`. Ends at AMJ 2026. |
| `GLB.Ts+dSST.csv` | NASA GISTEMP v4 global land-ocean anomaly, from 1880. The first line is a title. After that, rows = years; columns = `Jan` … `Dec`, then annual (`J-D`, `D-N`) and seasonal (`DJF`, `MAM`, `JJA`, `SON`) means. `***` means not yet available. Only the 12 monthly columns are used. |

#### `data/raw/price_indices/`

| File | Contents |
|---|---|
| `bls_1990_1999.json`, `bls_2000_2009.json`, `bls_2010_2019.json`, `bls_2020_2026.json` | Untouched BLS API responses, one per decade (the keyless API returns at most 10 years per request). Each holds the three series with one entry per month (`period` `M01`–`M12`) plus annual averages (`M13`), and footnotes such as "Preliminary". |

#### `data/raw/positions/`

| File | Contents |
|---|---|
| `cot_cocoa_disaggregated_weekly.csv` | CFTC Disaggregated Commitments of Traders, futures only, for cocoa on ICE Futures U.S. (called New York Board of Trade before Sep 2007). One row per weekly report, from 13 June 2006 to 17 November 2026 (1,067 rows), with all 194 CFTC columns. Positions are numbers of contracts (1 contract = 10 tonnes). |

#### `data/raw/production/`

| File | Contents |
|---|---|
| `faostat_cocoa_civ_ghana.csv` | FAOSTAT rows for cocoa beans (item 661) in Côte d'Ivoire (area 107) and Ghana (area 81), 1961–2024, all elements (384 rows). Columns as FAO publishes them: `Area`, `Item`, `Element` (Production, Area harvested, Yield), `Year`, `Unit`, `Value`, `Flag` (A = official, E = estimated, I = imputed, X = external organization), `Note`. |

#### `data/pre_processed/`

| File | Shape | Contents |
|---|---|---|
| `prices_monthly.csv` | 442 rows × 10 cols | `month` plus the 9 price columns (see [Prices](#prices)). No missing values. |
| `prices_units.csv` | 9 rows | `commodity`, `unit`: the unit of each price column, copied from the Pink Sheet. |
| `weather_monthly.csv` | 4,420 rows (442 months × 10 regions) | **Long format.** `month`, `region`, `commodity` (what the region proxies), `lat`, `lon`, then `temp_mean_c`, `temp_max_c`, `temp_min_c`, `precip_mm`, `hdd`, `cdd`, `heat_days`, `dry_days`, `temp_anom_c`, `precip_anom_mm`. No missing values. |
| `climate_monthly.csv` | 442 rows × 4 cols | `month`, `oni`, `roni`, `gistemp_anom_c`. `roni` is empty for 2026-06 to 2026-10 and `oni` for 2026-10 (not yet published). |
| `price_indices_monthly.csv` | 442 rows × 4 cols | `month`, `cpi_all` (CPI all items, 1982–84 = 100), `cpi_candy` (CPI candy and chewing gum, Dec 1997 = 100), `ppi_chocolate` (PPI chocolate products, Jun 1983 = 100). `cpi_candy` starts in Dec 1997, when BLS created it. Both CPIs are empty for Oct 2025 (not collected during the US government shutdown). The latest 4 PPI months are preliminary. |
| `cot_cocoa_monthly.csv` | 246 rows × 10 cols | `month` (2006-06 to 2026-11); `open_interest`; `mm_long`, `mm_short`, `mm_spread` (managed money, i.e. hedge funds); `prod_long`, `prod_short` (producers, merchants, processors); `swap_long`, `swap_short` (swap dealers); `n_reports` (weekly reports averaged). All positions in contracts. |
| `cocoa_production_annual.csv` | 35 rows × 9 cols | `year` (1990–2024); for each of `civ_` and `ghana_`: `production_t` (tonnes), `area_harvested_ha` (hectares), `yield_kg_ha` (kg per hectare); `civ_production_t_flag`, `ghana_production_t_flag` (FAO flag, see above). No missing values. |

### Data dictionary

#### Prices
Source: World Bank Commodity Price Data ("Pink Sheet"), monthly averages in nominal USD. See `prices_units.csv` for each series' unit.

| Column | Pink Sheet series | Unit |
|---|---|---|
| `maize` | Maize | $/mt |
| `wheat_us_hrw` | Wheat, US HRW | $/mt |
| `soybeans` | Soybeans | $/mt |
| `coffee_arabica` | Coffee, Arabica | $/kg |
| `cocoa` | Cocoa | $/kg |
| `sugar_world` | Sugar, world | $/kg |
| `natgas_us` | Natural gas, US (Henry Hub) | $/mmbtu |
| `natgas_europe` | Natural gas, Europe | $/mmbtu |
| `crude_brent` | Crude oil, Brent | $/bbl |

#### Climate

| Column | Meaning | Source |
|---|---|---|
| `oni` | Oceanic Niño Index: 3-month running mean of Niño 3.4 SST anomalies (°C), assigned to the centre month | NOAA CPC |
| `roni` | Relative ONI. NOAA's official ENSO index since February 2026. ≥ +0.5 means El Niño; ≤ −0.5 means La Niña. | NOAA CPC |
| `gistemp_anom_c` | Global land-ocean temperature anomaly vs the 1951–1980 mean (°C) | NASA GISTEMP v4 |

#### Weather
Source: ERA5 / ERA5-Land reanalysis via the Open-Meteo archive API (CC BY 4.0).

| Feature | Meaning |
|---|---|
| `temp_mean_c` | Monthly mean of daily mean 2 m temperature (°C) |
| `temp_anom_c` | `temp_mean_c` minus the 1991–2020 mean for that calendar month |
| `precip_mm` | Total monthly precipitation (mm) |
| `precip_anom_mm` | `precip_mm` minus the 1991–2020 mean for that calendar month |
| `hdd` / `cdd` | Heating / cooling degree days, base 18.3 °C (65 °F) |
| `heat_days` | Days with daily max ≥ 35 °C (crop heat stress) |
| `dry_days` | Days with < 1 mm precipitation |

| Region | Coordinates | Proxies |
|---|---|---|
| `us_cornbelt_iowa` | 41.59, −93.62 | maize, soybeans |
| `us_cornbelt_illinois` | 40.12, −88.24 | maize, soybeans |
| `us_kansas` | 37.69, −97.34 | wheat |
| `ukraine_poltava` | 49.59, 34.55 | wheat, maize |
| `russia_krasnodar` | 45.04, 38.98 | wheat |
| `brazil_minas_gerais` | −21.55, −45.43 | arabica coffee |
| `ivory_coast_daloa` | 6.88, −6.45 | cocoa |
| `ghana_kumasi` | 6.69, −1.62 | cocoa |
| `us_chicago` | 41.88, −87.63 | natural gas demand |
| `us_new_york` | 40.71, −74.01 | natural gas demand |

## 2. Cleaning

*To be added by the cleaning stage.*

## 3. Analysis

*To be added by the analysis stage.*

## Known limitations

- **Single points stand in for whole regions.** A better version would average several points per region, or weight them by production.
- **Prices are nominal.** Deflate them (for example by US CPI) before comparing across decades.
- **Prices are monthly averages,** so short weather shocks within a month get smoothed out.
- **Recent ENSO values are provisional.** NOAA revises the latest values for up to two months. In the current build, the CPC RONI table stops at AMJ 2026 (May), so `roni` is NaN for June to October 2026. `oni` stops at ASO (September), so October 2026 is NaN.
- **Seasonality matters.** Weather in the growing season matters far more than in the off-season, so the analysis should account for crop calendars.
- **Natural gas depends on more than weather.** Prices also respond to storage levels, LNG exports and the 2022 Europe shock. Consider controls or a dummy for 2022.

## Sources

- World Bank Pink Sheet: https://www.worldbank.org/commodities
- Open-Meteo Historical Weather API: https://open-meteo.com/en/docs/historical-weather-api
- NOAA CPC ONI: https://www.cpc.ncep.noaa.gov/products/analysis_monitoring/enso/oni/v6/
- NOAA CPC RONI: https://www.cpc.ncep.noaa.gov/products/analysis_monitoring/enso/roni/v5/
- NASA GISTEMP v4: https://data.giss.nasa.gov/gistemp/
- BLS CPI: https://www.bls.gov/cpi/ and PPI: https://www.bls.gov/ppi/
- CFTC Commitments of Traders: https://www.cftc.gov/MarketReports/CommitmentsofTraders/index.htm
- FAOSTAT Crops and livestock products: https://www.fao.org/faostat/en/#data/QCL
