[README.md](https://github.com/user-attachments/files/31725111/README.md)
# SF EMS Emergency Response Data Analysis Project

## Project Overview

Analysis of **San Francisco Fire Department & Emergency Medical Services Dispatched Calls for Service** (Sep 2025 – Aug 2026). The project covers data cleaning, web scraping, predictive modeling, Excel reporting, and interactive dashboards in Power BI and Tableau.

| Property | Value |
|----------|-------|
| **Dataset** | SF Fire Dept & EMS Dispatched Calls for Service |
| **Source** | [data.sfgov.org/d/nuek-vuh3](https://data.sfgov.org/d/nuek-vuh3) |
| **Period** | September 1, 2025 – August 26, 2026 |
| **Records** | 359,542 unit-dispatch rows (API snapshot + web-scraped weather) |
| **Columns** | 56 (cleaned) |
| **Team Size** | 6 people |

---

## Team Roles & Responsibilities

| Role | Member | Primary Responsibility |
|------|--------|----------------------|
| **Team Lead** | P1 | Coordination, code review, final QA, Excel workbook |
| **Data Engineer** | P2 | Download, filter, clean, preprocess, export CSV |
| **Web Scraper** | P3 | Weather data, holidays, events enrichment via web scraping |
| **ML Analyst** | P4 | Linear & Logistic regression models in Python |
| **Power BI Dev** | P5 | Power BI interactive dashboard (3 pages) |
| **Tableau Dev** | P6 | Tableau interactive dashboard (3 sheets + dashboard) |

---

## Dataset Columns

### Identification

| Column | Type | Description |
|--------|------|-------------|
| `Call Number` | int64 | Unique 911 call identifier |
| `Unit ID` | str | Specific unit dispatched (e.g., E08, M572) |
| `Incident Number` | int64 | Groups all units responding to the same call |
| `RowID` | str | Composite key: `CallNumber-UnitID` (e.g., 252443041-E08) |

### Call Classification

| Column | Type | Description |
|--------|------|-------------|
| `Call Type` | str | Specific call type — 40+ categories |
| `Call Type Group` | str | Broad category: Potentially Life-Threatening, Alarm, Non Life-threatening, Fire |
| `Original Priority` | object | Priority when call first received (1, 2, 3, or letter codes A-E) |
| `Priority` | object | Updated priority after triage (1, 2, 3, E, I, A, T, B) |
| `Final Priority` | int64 | Final priority after triage: 2 or 3 only |
| `ALS Unit` | bool | Was an Advanced Life Support unit dispatched? (True/False) |
| `Call Final Disposition` | str | Outcome of the call (see below) |
| `Number of Alarms` | int64 | Alarm level (1, 2, or 3 — most are 1) |

**Call Type Group breakdown:**

| Group | Count | Percentage |
|-------|-------|------------|
| Potentially Life-Threatening | 176,518 | 49% |
| Alarm | 89,896 | 25% |
| Non Life-threatening | 69,223 | 19% |
| Fire | 14,053 | 4% |

**Call Final Disposition (outcome):**

| Disposition | Count | Percentage |
|-------------|-------|------------|
| Code 2 Transport | 135,099 | 38% |
| Fire | 96,851 | 27% |
| Other | 86,005 | 24% |
| Code 3 Transport | 15,343 | 4% |
| Unable to Locate | 11,428 | 3% |
| Cancelled | 8,608 | 2% |
| Medical Examiner | 4,107 | 1% |

### Timestamps (7 time phases)

| Column | Type | Description | Null % |
|--------|------|-------------|--------|
| `Call Date` | datetime | Date of call only | 0% |
| `Watch Date` | datetime | Shift/watch date | 0% |
| `Received DtTm` | str | When 911 call was received | 0% |
| `Entry DtTm` | str | When entered into CAD system | 0% |
| `Dispatch DtTm` | str | When unit was dispatched | 0% |
| `Response DtTm` | str | When unit acknowledged | 2.7% |
| `On Scene DtTm` | str | When unit arrived on scene | 21.7% |
| `Transport DtTm` | str | When patient was transported | 76.0% |
| `Hospital DtTm` | str | When arrived at hospital | 76.4% |
| `Available DtTm` | str | When unit became available again | ~0% |

**Why the high nulls?**
- `Transport DtTm` (76%) — most calls don't require transport (fires, alarms, cancelled)
- `On Scene DtTm` (21.7%) — cancelled/unable-to-locate calls
- `Response DtTm` (2.7%) — minor, likely late arrivals

### Location

| Column | Type | Description |
|--------|------|-------------|
| `Address` | str | Block-level address/intersection |
| `City` | str | City name (mostly "San Francisco") |
| `Zipcode of Incident` | float64 | ZIP code of incident |
| `Battalion` | str | Fire battalion (12 unique) |
| `Station Area` | float64 | Station zone |
| `Box` | object | Dispatch box area |
| `Fire Prevention District` | float64 | Fire prevention zone (1-10) |
| `Supervisor District` | float64 | Supervisor district (1-11) |
| `Neighborhooods - Analysis Boundaries` | str | 41 neighborhoods |
| `case_location` | str | GPS coordinates (GeoJSON POINT) |

### Unit/Resource

| Column | Type | Description |
|--------|------|-------------|
| `Unit Type` | str | Type of unit dispatched (ENGINE, MEDIC, TRUCK, etc.) |
| `Unit sequence in call dispatch` | int64 | Order unit was dispatched (1 = first) |

### Metadata

| Column | Description |
|--------|-------------|
| `data_as_of` | When dataset was last updated |
| `data_loaded_at` | When data was loaded to portal |

---

## Business Questions

| # | Question | Tool | Owner |
|---|----------|------|-------|
| 1 | **When** do the most emergencies happen? (hourly, daily, monthly patterns) | Excel, Power BI | P1, P5 |
| 2 | **Where** are the EMS hotspots? (neighborhood, station, battalion) | Tableau, Power BI | P5, P6 |
| 3 | **What** drives response time? Can we predict it? | Linear Regression | P4 |
| 4 | **Will** a call need ALS? Can we classify it? | Logistic Regression | P4 |
| 5 | **Does weather affect call volume?** | Python + web scraping | P2, P3 |

---

## Project Phases

### Phase 1: Web Scraping & Enrichment

- Pull raw call data from DataSF API (`nuek-vuh3`)
- Scrape daily weather data from Open-Meteo API:
  - Temperature (avg, min, max)
  - Precipitation
  - Wind speed
  - For SF coordinates: 37.77°N, 122.42°W
- Scrape US/CA holidays using `python-holidays` library
- Merge weather + holidays onto main dataset by date
- Export enriched CSV: `sf_ems_weather_merged.csv`

### Phase 2: Data Acquisition & Cleaning
**Owner: P2 (Data Engineer)**

- Read merged CSV from Phase 1
- Filter to Sep 2025 – Aug 2026 using `Received DtTm`
- Convert all timestamp columns from string to datetime
- Handle nulls:
  - `Zipcode` (455 nulls): Impute from `Address` or `case_location` GPS
  - `Original Priority` (3,248 nulls): Fill with `Priority` column
  - `On Scene DtTm` (21.7%): Keep nulls for cancelled calls
  - `Transport DtTm` / `Hospital DtTm` (76%): Keep as-is (expected)
- Check for duplicates using `RowID`
- Export cleaned CSV: `sf_ems_clean_9_2025_to_8_2026.csv`

### Phase 3: Feature Engineering 

Create derived columns:

| New Column | Formula | Used In |
|------------|---------|---------|
| `response_time_min` | `(On Scene DtTm - Received DtTm).total_seconds() / 60` | Linear Regression (target) |
| `dispatch_lag_min` | `(Dispatch DtTm - Entry DtTm).total_seconds() / 60` | Linear Regression (feature) |
| `transport_time_min` | `(Hospital DtTm - Transport DtTm).total_seconds() / 60` | EDA |
| `hour_of_day` | `Received DtTm.hour` | Both models |
| `day_of_week` | `Received DtTm.day_name()` | Both models |
| `month` | `Received DtTm.month` | Both models |
| `is_weekend` | `day_of_week in ['Saturday', 'Sunday']` | Both models |
| `is_als` | `ALS Unit` (bool → int 0/1) | Logistic Regression (target) |

#### IQR Outlier Detection — Flag-Only (standard 1.5× IQR)

Applied in `01_data_cleaning.ipynb` to the derived time columns. Bounds are computed at runtime.

| Column | Fences (real, at runtime) | Flag Column | Flagged |
|--------|--------------|-------------|---------|
| `response_time_min` | [0.00, 23.77] | `is_outlier_response_time_min` | 22,094 (6.2%) |
| `dispatch_lag_min` | [0.00, 2.35] | `is_outlier_dispatch_lag_min` | 41,976 (11.7%) |
| `transport_time_min` | [0.00, 43.46] | `is_outlier_transport_time_min` | 2,522 (0.7%) |

- **Flag-only**: values are NOT modified — the CSV keeps the real measured times (this protects data truth AND model performance).
- Why not cap? Capping response times crushed **R² from 0.80 → 0.56** (see Phase 5). The flags let Excel/Power BI/ML filter outliers (`is_outlier_* == False`) whenever needed.
- Negative times (data inconsistencies) fall under the lower fence → automatically flagged.

### Phase 4: Exploratory Data Analysis


- Call volume trends (hourly, daily, monthly)
- Geographic distribution by neighborhood/battalion
- Response time distribution by priority/battalion
- Correlation analysis with weather data

### Phase 5: Predictive Modeling 


**Linear Regression:**
- **Target**: `response_time_min` (REAL values — outliers flagged, not capped)
- **Features**: `Call Type`, `Call Type Group`, `Priority` (original/final), `Unit Type`, `hour_of_day`, `day_of_week`, `month`, `is_weekend`, `time_period`, `dispatch_lag_min`, `units_per_incident`, `number_of_alarms`, `unit_sequence_in_call_dispatch`, `battalion`, `neighborhoods_analysis_boundaries`, + web-scraped **weather** (`avg/min/max_temperature`, `precipitation`, `max_wind_speed`, `is_holiday`)
- **Drop**: rows where `on_scene_dttm` is null (no target to predict)
- **Results (full data, 281,553 rows)**: **R² = 0.804**, MAE = 3.32 min, RMSE = 5.50 min
- **Why the old R² was 0.56**: the dataset used was the *capped* version — ~22k rows pinned to 23.78 min crush linear fit. Real values → 0.80. Weather adds <0.001 (response time is logistics-driven, not weather-driven).

**Logistic Regression (3-class Fast/Moderate/Slow by 5/10-min cutoffs):**
- **Features**: same set as linear
- **Results**: Accuracy = 0.745, weighted F1 = 0.73; weak on `Fast` (recall 0.24) due to imbalance
- **Note**: README target table below predates the final 3-class design; the notebook is authoritative.

### Phase 6: Excel Analysis 

| Sheet | Content |
|-------|---------|
| `Summary` | Total calls, avg response time, top call types, top neighborhoods |
| `Pivot_CallType_Neighborhood` | Rows: Call Type Group, Cols: Top 10 Neighborhoods, Values: Count |
| `Pivot_Hour_Day` | Rows: Hour of Day, Cols: Day of Week, Values: Avg Calls |
| `Response_Time_Stats` | By Battalion: Mean, Median, P90, P95 response times |
| `Monthly_Trend` | Month × Call Type Group pivot |

### Phase 7: Power BI Dashboard

| Page | Visuals |
|------|---------|
| `Overview` | KPI cards (total calls, avg response time, ALS %), monthly trend line chart |
| `Geography` | Map by neighborhood (color = call volume), drill to station area |
| `Operations` | Call type donut, priority distribution, hour-of-day heatmap, response time box plot by battalion |

### Phase 8: Tableau Dashboard

| Sheet | Visual |
|-------|--------|
| `Time Trends` | Dual-axis: bar (volume) + line (avg response time) by month |
| `Geographic Heatmap` | Filled map by neighborhood, filter by call type |
| `Call Distribution` | Stacked bar: Call Type Group × Battalion |
| `Dashboard` | All 3 sheets combined with filter actions |

### Phase 9: Final QA & Delivery
**Owner: P1 (All Team)**

- Cross-check all deliverables
- Validate model metrics
- Package all outputs into project folder
- Final team walkthrough

---

## Feature Engineering Details

### For Linear Regression

| Feature | Type | Description |
|---------|------|-------------|
| `Final Priority` | int | 2 or 3 |
| `Call Type Group` | categorical | Life-Threatening, Alarm, Non Life-Threatening, Fire |
| `hour_of_day` | int | 0-23 |
| `is_weekend` | bool | Saturday or Sunday |
| `Battalion` | categorical | 12 battalions |
| `Unit Type` | categorical | ENGINE, MEDIC, TRUCK, etc. |

### For Logistic Regression

| Feature | Type | Description |
|---------|------|-------------|
| `Call Type Group` | categorical | 4 categories |
| `Final Priority` | int | 2 or 3 |
| `hour_of_day` | int | 0-23 |
| `is_weekend` | bool | Saturday or Sunday |
| `Battalion` | categorical | 12 battalions |
| `Number of Alarms` | int | 1, 2, or 3 |

---

## Web Scraping Sources

| Source | URL | What to Scrape | Merge Key |
|--------|-----|----------------|-----------|
| Open-Meteo API | `archive-api.open-meteo.com` | Daily weather: temp, precipitation, wind | `date` |
| US Holidays | `python-holidays` library | Federal + CA state holidays | `date` |

---

## Deliverables Checklist

| # | Deliverable | Format | Owner |
|---|------------|--------|-------|
| 1 | `sf_ems_clean_9_2025_to_8_2026.csv` | CSV | P2 & P1 |
| 2 | `sf_ems_weather_merged.csv` (raw + weather) | CSV | P3 |
| 3 | Weather + Events enriched dataset | CSV | P3 |
| 4 | Web scraping scripts | Python | P3 |
| 5 | Linear Regression model + results | Notebook | P4 |
| 6 | Logistic Regression model + results | Notebook | P4 |
| 7 | Excel analysis workbook | .xlsx | P1 |
| 8 | Power BI dashboard | .pbix | P5 |
| 9 | Tableau workbook | .twbx | P6 |
| 10 | Project summary | README.md | P1 |

---

## Data Versions

| File | Contents | Use |
|------|----------|-----|
| `data\raw\*.csv` | Original download (36 cols, 357,612 rows) | Archive |
| `data\version 1\sf_ems_clean_9_2025_to_8_2026.csv` | Cleaned, pre-IQR archive, real values (45 cols) | Truth archive |
| `data\cleaned\sf_ems_clean_9_2025_to_8_2026-updated.csv` | OLD capped experiment (48 cols) | Archive — do not use for ML |
| `data\cleaned\sf_ems_clean_9_2025_to_8_2026.csv` | **WORKING**: cleaned + weather, real values, flag-only (56 cols, 359,542 rows) | Excel, Power BI, Tableau, ML |

The working file is produced end-to-end by `01_data_cleaning.ipynb` from `notebooks\sf_ems_weather_merged.csv` (raw + web-scraped weather/holidays).

---

## Python Libraries

```
pandas, numpy, openpyxl           # Data handling
matplotlib, seaborn               # EDA plots
scikit-learn                      # LinearRegression, LogisticRegression, metrics
requests, beautifulsoup4          # Web scraping
holidays                          # US holidays
```

---

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| Large file slow to load | Read xlsx in chunks or convert to CSV first |
| Timestamp parsing errors | Use `pd.to_datetime(..., errors='coerce')` |
| Model overfitting | Train/test split 80/20, cross-validation |
| Nulls in `response_time` | Only use non-null rows for regression target |
| Web scraping blocked | Use API (Open-Meteo) instead of HTML scraping |

---

## SQL Integration (Future/Optional)

| Property | Value |
|----------|-------|
| **Database** | MS SQL Server |
| **Status** | Deferred — to be revisited after core analysis |
| **Potential Use** | Store cleaned data, run SQL aggregations, Python connection via `pyodbc` |


