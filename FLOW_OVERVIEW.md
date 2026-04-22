# Flow Overview

This repository uses Node-RED, so the main application logic is embedded inside the flow JSON rather than split into many standalone `.js` files.

## Main Logic Files

- `SmartCityClimate.Flow.json`
- `node_red_data/flows.json`

Both files describe the same Node-RED workflow.

## Function Nodes

The flow currently contains 8 function nodes with the core logic:

1. `Build Last Realtime Query`
   - Creates the MongoDB query used to fetch the newest realtime record.

2. `Store Last Realtime updateTime`
   - Saves the last seen upstream `updateTime` into flow context.

3. `Normalize Realtime Snapshot`
   - Parses realtime weather payloads.
   - Computes normalized metrics such as:
     - `avgTemp`
     - `avgHumidity`
     - `maxRainfall`
     - `heatStress`
     - `riskLevel`
   - Inserts only when upstream `updateTime` changes.

4. `Normalize Forecast`
   - Converts 9-day forecast payloads into a cleaner structure for storage and chart rendering.

5. `Build 24h Query`
   - Builds the MongoDB time-range query for the last 24 hours using `collectedAt`.

6. `Format Realtime Charts`
   - Rebuilds MongoDB records into dashboard chart series.
   - Computes:
     - temperature vs heat stress chart data
     - rainfall chart data
     - Pearson correlation summaries

7. `Build Last Forecast Query`
   - Fetches the newest forecast snapshot from MongoDB.

8. `Format Forecast Chart`
   - Converts stored forecast records into the 9-day min/max chart format.

## Why There Are Not Many Source Files

Node-RED projects are visually programmed. That means:

- node wiring is stored as JSON
- each `function` node stores a JavaScript snippet internally
- the full program is spread across connected nodes, not separate modules

So the repository may look “small”, but the actual application logic is already present in the flow files.

## Submission Zip

`submission/COMP7503C_Group_Assignment_FINAL.zip` is only a packaged snapshot for the course submission.

It does not contain secret or additional source code that is missing from the repository.
