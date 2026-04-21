# Docker Environment Description (COMP7503C Programming Assignment)

This file is the reproducible Docker description required by the assignment.

## 1) Docker Hub ID

- Docker Hub ID: `<YOUR_DOCKER_HUB_ID>`
- If your group does not submit Docker Hub image, keep this document in the zip as the reproducible environment description.

## 2) Runtime Architecture

- `smartcity-nodered` (image: `nodered/node-red`)
- `smartcity-mongo` (image: `mongo:latest`)
- Shared Docker network: `smartcity-net`
- Dashboard URL: `http://localhost:1880/ui`

## 3) One-Command Startup (Recommended)

In the `deliverables` folder:

```bash
docker compose up -d
```

This compose setup will:

1. Start MongoDB container.
2. Start Node-RED container.
3. Auto-install required Node-RED modules inside container:
   - `node-red-dashboard`
   - `node-red-contrib-mongodb3`

## 4) Import and Deploy Flow

1. Open Node-RED editor: `http://localhost:1880`
2. Menu -> Import -> select `SmartCityClimate.Flow.json`
3. Click Deploy
4. Open dashboard: `http://localhost:1880/ui`

The provided flow already uses Docker-network-safe Mongo URI:

- `mongodb://smartcity-mongo:27017/smartcity`

## 5) Expected MongoDB Collections

- `UrbanClimateRealtime`
- `UrbanClimateForecast`

## 6) Verification Commands

```bash
# container health
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Node-RED logs
docker logs --tail=100 smartcity-nodered

# MongoDB connectivity test
docker exec -it smartcity-mongo mongosh --quiet --eval "db.adminCommand({ ping: 1 })"
```

## 7) Common Errors and Fixes

1. `unknown types` in imported flow:
   - Ensure `node-red-dashboard` and `node-red-contrib-mongodb3` are installed.
2. `MongoNetworkError`:
   - Ensure both containers are in `smartcity-net`.
   - Ensure flow URI is `mongodb://smartcity-mongo:27017/smartcity`.
3. Dashboard empty after deployment:
   - Wait for scheduled polling to collect data.
   - Check Node-RED debug sidebar and container logs.

## 8) Stop and Cleanup

```bash
docker compose down
```

Optional full cleanup (including data volumes in local folders):

```bash
rm -rf ./mongo_data ./node_red_data
```
