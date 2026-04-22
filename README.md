# HKTempWeatherMonitor

HKTempWeatherMonitor is a Dockerized Node-RED + MongoDB project for monitoring Hong Kong weather conditions and presenting smart city insights on a dashboard.

The system fetches official HKO open data, stores normalized records in MongoDB, computes derived indicators such as `heatStress`, and renders charts for:

- 24-hour average temperature vs heat stress
- 24-hour maximum rainfall
- 9-day max/min temperature forecast
- correlation summary widgets

## Repository Layout

- `docker-compose.yml`: reproducible runtime for MongoDB and Node-RED
- `SmartCityClimate.Flow.json`: exported Node-RED flow
- `FLOW_OVERVIEW.md`: explanation of where the actual logic lives inside the flow
- `Project_Report_SmartCityClimate.md`: written report for the assignment
- `docker-description.md`: Docker reproduction notes
- `node_red_data/`: preloaded Node-RED workspace so the flow is ready after startup
- `submission/COMP7503C_Group_Assignment_FINAL.zip`: final submission package

## Where The Code Actually Is

This project is not a traditional multi-file JavaScript app. It is a Node-RED project, so most of the implementation logic lives inside the flow JSON files:

- `SmartCityClimate.Flow.json`
- `node_red_data/flows.json`

These two files contain the same flow logic in two forms:

- `SmartCityClimate.Flow.json`: clean exported flow for assignment submission
- `node_red_data/flows.json`: the runtime copy automatically loaded by Node-RED on startup

Inside the flow there are 8 `function` nodes containing the main JavaScript logic, including:

- realtime snapshot normalization
- forecast normalization
- 24-hour MongoDB query construction
- chart formatting
- correlation calculation

The `submission/COMP7503C_Group_Assignment_FINAL.zip` file is only the final hand-in package. It does not contain hidden extra logic beyond the files already present in the repository.

## Data Sources

This project uses public Hong Kong Observatory APIs:

- Current weather report: `rhrread`
- 9-day weather forecast: `fnd`

Both are official open-data resources under the Hong Kong government open data ecosystem.

## Prerequisites

- Docker Desktop installed and running
- Internet access to `data.weather.gov.hk`

## Quick Start

1. Clone this repository.
2. Start Docker Desktop.
3. In the repository root, run:

```bash
docker compose up -d
```

4. Open the Node-RED editor:

```text
http://localhost:1880
```

5. Open the dashboard:

```text
http://localhost:1880/ui/
```

The repository already includes a preloaded `node_red_data/flows.json`, so the Smart Climate flow should appear automatically after startup. The standalone exported flow file is also kept at the root for backup and assignment submission purposes.

## What You Should See

In the editor:

- a flow tab named `Smart Climate Insights`
- scheduled inject nodes for realtime polling and dashboard refresh
- MongoDB, forecast, and dashboard nodes already wired together

In the dashboard:

- `Heat Stress (24h)`
- `Rainfall Pressure (24h)`
- `9-Day Forecast`

## Running Notes

- Realtime data is stored in incremental mode. A new MongoDB document is inserted only when the upstream `updateTime` changes.
- This means you will usually get roughly one point per source update, not one point per minute.
- To accumulate a full 24-hour trend, keep Docker running continuously and avoid sleep/shutdown interruptions.

## Node-RED and MongoDB Details

- MongoDB container name: `smartcity-mongo`
- Node-RED container name: `smartcity-nodered`
- MongoDB URI inside the flow: `mongodb://smartcity-mongo:27017/smartcity`

## Resetting Local Data

If you want to start from a clean database:

```bash
docker compose down
rm -rf mongo_data
docker compose up -d
```

If you also want to reset the committed Node-RED runtime state to a fresh local copy, remove only the generated runtime files and keep the tracked configuration files in `node_red_data/`.

## Troubleshooting

If the dashboard is empty:

- make sure Docker Desktop is actually running
- check `http://localhost:1880` first
- confirm your network can access `https://data.weather.gov.hk`
- wait for the upstream API to publish a new `updateTime`

If MongoDB connection fails:

- ensure both containers are up
- ensure the Docker network from `docker-compose.yml` was created successfully

If charts look sparse:

- that usually means the source API has not updated often, or Docker was not running continuously

## Assignment Files

This repository keeps both the runnable project files and the assignment deliverables:

- runnable project files at the repository root
- final submission package in `submission/`

## Status

The project has been tested locally with:

- Dockerized Node-RED
- Dockerized MongoDB
- dashboard served at `http://localhost:1880/ui/`
- preloaded flow in `node_red_data/`
