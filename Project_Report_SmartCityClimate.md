# COMP7503C Multimedia Technologies
## Programming Assignment Written Report

- Project Title: Smart Climate Insight Dashboard for Urban Outdoor Planning
- Course: COMP7503C
- Submission Deadline: 2026-04-27 23:55 (HKT)
- Group Members:
  - `<Student 1 Name / UID>`
  - `<Student 2 Name / UID>`
  - `<Student 3 Name / UID (optional)>`

## 1. Project Overview

This project builds a smart city climate analytics system in Node-RED using Hong Kong open data streams from `data.gov.hk` / HKO. The system continuously ingests weather data, stores normalized records in MongoDB, computes derived risk indicators, and presents a decision-oriented dashboard for urban outdoor operations.

The design goal is not only to visualize weather values, but to transform raw feeds into practical insights for scheduling and risk awareness.

## 2. Smart City Use Cases

### 2.1 Use Case A: Outdoor Heat-Risk Monitoring

Target users include event coordinators, municipal field teams, and construction supervisors. They need to know whether thermal stress is rising and whether current conditions remain safe for prolonged outdoor activity.

The dashboard addresses this by combining temperature and humidity into a derived `heatStress` score and showing how the score evolves over time.

### 2.2 Use Case B: Rainfall Disruption Monitoring

Target users need to quickly detect periods where rainfall intensity may affect operations, transport flow, and temporary outdoor infrastructure.

The dashboard tracks the district-level maximum rainfall and displays the 24-hour rainfall trend to identify spikes.

### 2.3 Use Case C: Short-Term Planning with Forecast Envelope

For near-term planning (staffing, logistics, and event preparation), users need an at-a-glance forecast range rather than reading long text bulletins.

The dashboard converts 9-day forecast data into a min/max temperature envelope for fast planning decisions.

## 3. Open Data Streams Used

The system uses two official HKO APIs:

1. Realtime Weather Report (`rhrread`)
   - `https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=rhrread&lang=en`
2. 9-Day Weather Forecast (`fnd`)
   - `https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=fnd&lang=en`

Both APIs are pulled by scheduled Node-RED flows through HTTPS.

## 4. Implementation Details

### 4.1 Runtime and Deployment

- Node-RED container (`nodered/node-red`) for orchestration and dashboard.
- MongoDB container (`mongo:latest`) for persistent storage.
- Docker network-based connectivity (`mongodb://smartcity-mongo:27017/smartcity`).
- Dashboard endpoint: `http://localhost:1880/ui`.

### 4.2 Flow Design

The implementation has four functional pipelines:

1. Realtime ingestion (trigger every 60 seconds)
   - Query latest DB record to get the last seen `updateTime`.
   - Call `rhrread` API.
   - Normalize and compute metrics.
   - Incremental insert rule: only insert when source `updateTime` is newer than last seen value.

2. Forecast ingestion (trigger every 21600 seconds / 6 hours)
   - Call `fnd` API.
   - Normalize 9-day fields.
   - Insert forecast snapshot.

3. Realtime dashboard refresh (trigger every 60 seconds)
   - Query realtime records in last 24 hours using `collectedAt`.
   - Build chart payloads for temperature/stress and rainfall.
   - Compute Pearson correlation values for insight widgets.

4. Forecast dashboard refresh (trigger every 6 hours)
   - Query latest forecast document.
   - Reformat into min/max line chart series.

### 4.3 Data Cleaning and Transformation

Key data processing logic:

- Numeric sanitation: all numeric fields pass a finite-number check (`toNumber`), invalid values become `null`.
- No forced zero filling for missing values in charts; invalid points are skipped to avoid misleading drops.
- Time normalization: source `updateTime` converted to ISO format; dashboard time axis uses `collectedAt` for consistent timeline plotting.

### 4.4 Derived Metrics and Correlation

Derived `heatStress` score (0 to 100) is computed from:

- average temperature,
- average relative humidity,
- capped rainfall contribution.

Risk level mapping:

- `Low` (<45)
- `Moderate` (45-64)
- `High` (65-79)
- `Severe` (>=80)

Correlation analysis (24-hour window):

- `corr(avgTemp, heatStress)`
- `corr(maxRainfall, heatStress)`

Both coefficients are computed in-flow using Pearson correlation and shown in dashboard text widgets.

## 5. Data Storage Approach

### 5.1 Collection 1: `UrbanClimateRealtime`

Each document stores a normalized incremental snapshot:

- `updateTime` (source timestamp)
- `collectedAt` (ingestion timestamp)
- `sourceUpdated` (boolean, true for inserted incremental records)
- `stationCount`
- `avgTemp`, `minTemp`, `maxTemp`
- `avgHumidity`
- `maxRainfall`, `maxRainfallDistrict`
- `hottestStation`
- `heatStress`
- `riskLevel`

Design rationale:

- `updateTime` preserves source chronology.
- `collectedAt` supports dashboard range queries and temporal plotting.
- Aggregated fields reduce dashboard query complexity and make rendering fast.

### 5.2 Collection 2: `UrbanClimateForecast`

Each document stores:

- `updateTime` (ingestion time)
- `sourceUpdateTime` (upstream update timestamp)
- `forecast` array (9 items), each including:
  - `forecastDate`, `week`
  - `minTemp`, `maxTemp`
  - `minRH`, `maxRH`
  - `weather`

Design rationale:

- Storing one full 9-day snapshot per update simplifies retrieval for dashboard rendering.

### 5.3 Update Strategy and Indexing

Update strategy:

- Realtime collection uses incremental insert by `updateTime` change detection.
- Forecast collection inserts per polling cycle (low frequency).

Recommended indexes:

- `UrbanClimateRealtime.updateTime` (for source-time lookups)
- `UrbanClimateRealtime.collectedAt` (for 24-hour range queries)
- `UrbanClimateForecast.updateTime` (for latest snapshot retrieval)

Retention note:

- For assignment demo, snapshots are retained.
- For production, TTL/archive policy should be added.

## 6. Dashboard and Insight Explanation

### 6.1 Dashboard Components

1. **Avg Temp vs Heat Stress (24h)**
   - Dual-series line chart comparing thermal baseline and derived stress.
   - Helps users detect whether discomfort risk rises beyond temperature-only changes.

2. **Max Rainfall (24h)**
   - Line chart for district maximum rainfall.
   - Highlights wet-period intensity likely to impact outdoor operations.

3. **9-Day Max/Min Temperature**
   - Forecast envelope chart (daily min vs max).
   - Supports resource and activity planning.

4. **Text Insight Widgets**
   - Latest update time
   - Current heat stress and risk level
   - District with latest maximum rainfall
   - Pearson correlation values for temperature-stress and rainfall-stress

### 6.2 Observed 24-Hour Results (System Validation Snapshot)

At validation time on **2026-03-05 16:11 HKT** (window: 2026-03-04 16:11 HKT to 2026-03-05 16:11 HKT), the database contained:

- 24 realtime points in the last 24 hours.
- `avgTemp`: min 15.48 C, max 22.93 C, mean 17.51 C.
- `heatStress`: min 53, max 62, mean 57.21.
- `maxRainfall`: max 6 mm, mean 1.08 mm.
- Correlation:
  - `corr(avgTemp, heatStress) = 0.5821`
  - `corr(maxRainfall, heatStress) = 0.3838`
- Risk distribution in this window: 24/24 points were `Moderate`.

Interpretation:

- Temperature has a moderate positive relationship with heat stress in this period.
- Rainfall also shows a positive, weaker relationship with heat stress.
- The dashboard therefore supports a practical conclusion: thermal pressure in this window is stable in the moderate range, with no severe risk episode.

## 7. Reproducibility

The project is fully reproducible with the submitted artifacts:

- `SmartCityClimate.Flow.json`
- `docker-compose.yml`
- `docker-description.md`

Procedure:

1. `docker compose up -d`
2. Open Node-RED at `http://localhost:1880`
3. Import `SmartCityClimate.Flow.json` and Deploy
4. Open dashboard at `http://localhost:1880/ui`

## 8. Limitations and Future Work

1. `heatStress` is a heuristic score; future versions can integrate validated thermal comfort indices (e.g., WBGT/UTCI-related methods).
2. The project currently focuses on climate streams only; future work can integrate transport, crowd, or health-related open data for cross-domain correlation.
3. Alerting and recommendation features can be added (e.g., automatic warning when risk exceeds threshold).

## 9. Conclusion

This project satisfies the assignment requirements by delivering a complete smart city application pipeline: open-data ingestion, cleansing and aggregation, persistent storage, correlation analysis, and dashboard-based insight communication. The final system is reproducible in Docker and provides actionable climate intelligence for urban outdoor planning.
