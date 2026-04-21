# HKTempWeatherMonitor

Smart city climate monitoring project built with Node-RED, MongoDB, and Docker.

## Included Files

- `SmartCityClimate.Flow.json`: exported Node-RED flow
- `Project_Report_SmartCityClimate.md`: written report
- `docker-compose.yml`: reproducible local runtime
- `docker-description.md`: environment reproduction notes
- `COMP7503C_Group_Assignment_FINAL.zip`: final submission package

## Quick Start

1. Start Docker Desktop.
2. Run `docker compose up -d`.
3. Open `http://localhost:1880` for the Node-RED editor.
4. Open `http://localhost:1880/ui/` for the dashboard.

## Project Summary

The application fetches Hong Kong weather data, stores normalized records in MongoDB, performs trend and correlation analysis, and presents insights on a Node-RED dashboard for smart city outdoor planning use cases.
