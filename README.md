# SIH26060 — Antarctic EdgeOps (Final Prototype)

Digital Twin + offline-first remote management platform for India's Antarctic research stations **Maitri** and **Bharati**, built for **SIH26060 — "Digital Platform for efficient remote management of Indian Antarctic Research Stations"** (Ministry of Earth Sciences / NCPOR).

## Problem statement → implementation map
| SIH26060 requirement | Where it's implemented |
|---|---|
| Digital Twin for Maitri & Bharati | `researcher-control-tower` → Digital Twin tab, backed by `central-control-plane` station model |
| Infrastructure monitoring | Digital Twin → INFRASTRUCTURE domain (temperature, HVAC, structural health, air quality, water system) |
| Energy management | Digital Twin → ENERGY domain (generator, renewable generation, battery, fuel, load) |
| **Logistics planning** | **Logistics Plan tab** — consumption burn-rate forecasting + weather-gated resupply window recommendation (`/api/logistics`), distinct from raw inventory monitoring |
| Environmental monitoring | Digital Twin → ENVIRONMENT domain (wind, pressure, visibility, ice/weather risk) |
| **Predictive analytics** | **AI Insights tab** — connectivity-risk score, workload recommendation, priority recommendation and safe-scheduling recommendation, computed live from telemetry via `/api/ai/insights` (reuses the causal risk model in `ai-intelligence-core`, which used to run only as a disconnected CLI demo) |
| Remote station synchronization under severely constrained satellite bandwidth | Offline-first store-and-forward request queue + sync engine; each station simulates a narrowband Iridium-class link (kbps shown live) that degrades further under bad weather; every sync logs its payload size to the audit trail |

## What this final prototype demonstrates
- Station Digital Twin for Maitri and Bharati
- Infrastructure, energy, environment and logistics/inventory monitoring
- **Logistics planning**: burn-rate forecasts and safe resupply-window recommendations, not just stock levels
- **Live AI insights**: connectivity risk, workload risk, priority and safe-scheduling recommendations, computed from real station state (not a static demo script)
- Simulated constrained satellite bandwidth (kbps) that degrades with weather, with payload size logged on every sync
- Offline-first store-and-forward request queue
- Local station policy validation (6h / 12h / 24h collection intervals)
- Request lifecycle: REQUESTED → QUEUED → SENT → CHECKED → APPLIED → CONFIRMED
- Rejection and rollback demonstrations
- Audit trail and connectivity events

## Stack
100% JavaScript / Node.js. **Node 18.18+ required (Node 20 LTS recommended)** — see [Node.js troubleshooting](#nodejs-troubleshooting) below. Next.js, React, Express. No Python, pip, Docker or native database required for the demo.

## Architecture
```text
Researcher Control Tower (Next.js)
          │
          ▼
Central Control Plane (Node/Express)
   ├── request queue + audit
   ├── telemetry API
   ├── logistics planning API
   ├── AI insights API
   └── station connectivity
          │
   intermittent / low-bandwidth (simulated kbps) link
          │
   ┌──────┴────────┐
   ▼               ▼
Maitri Edge     Bharati Edge
   │               │
local policy    local policy
telemetry       telemetry
cache           cache
   └──────┬────────┘
          ▼
Store-and-forward Sync Engine
```

## Node.js troubleshooting
Every service in this project now checks its own Node.js version on startup and fails with a clear message — instead of a confusing crash — if it's too old:

```
NODE.JS VERSION TOO OLD  —  SIH26060 Antarctic EdgeOps
--------------------------------------------------------------
  Detected Node.js : v16.20.0
  Required         : >= 18.18.0   (Node 20 LTS recommended)
```

If you see this, install Node 20 LTS from https://nodejs.org/en/download (or `nvm install 20 && nvm use 20`) and re-run `npm start` / `npm run dev`.

**Why this matters:** two modules (`offline-edge-engine`, `ai-intelligence-core`) previously had no `package.json`, so their ES-module syntax only ran because newer Node versions (20.19+/22+) auto-detect module syntax in ambiguous `.js` files. On the documented minimum of Node 18, the identical files would crash with `SyntaxError: Unexpected token 'export'` with no useful explanation. Both folders now declare `"type": "module"` and an `engines` field explicitly, and every entry point imports a shared `nodeVersionGuard.js` first, so the failure mode is now a clear, actionable message rather than a version-dependent syntax error or a bare `fetch is not defined`.

## Windows quick start
Open 5 PowerShell/CMD windows.

### 1. Central API
```powershell
cd central-control-plane
npm install
npm start
```
API: http://localhost:8000

### 2. Maitri edge
```powershell
cd station-edge-agent
npm install
npm run maitri
```

### 3. Bharati edge
```powershell
cd station-edge-agent
npm run bharati
```

### 4. Sync engine
```powershell
cd sync-engine
npm install
npm start
```

### 5. Dashboard (another terminal)
```powershell
cd researcher-control-tower
npm install
npm run dev
```
Open http://localhost:3000

If you only need the UI, run only the dashboard. It includes demo controls and mock fallback data.

## SIH demo sequence
1. Open Digital Twin and show Maitri/Bharati telemetry, including the live satellite link kbps.
2. Show Infrastructure, Energy, Environment domain cards.
3. Open Logistics Plan — show burn-rate forecast and the safe-resupply-window recommendation.
4. Go to Safe Operations.
5. Simulate Maitri satellite link loss.
6. Submit a 6-hour research intent.
7. Show QUEUED / local autonomy.
8. Restore the link and sync.
9. Show SENT → CHECKED → APPLIED → CONFIRMED, and the payload-size line in the audit trail.
10. Submit the 1-hour test request and show station policy rejection.
11. Use rollback on a confirmed change.
12. Open AI Insights — show live connectivity risk, workload recommendation, priority and safe-scheduling recommendation.
13. Open Audit Trail to show traceability.

## Scope
This is a hackathon simulation/MVP. Telemetry, satellite transport and station hardware are simulated. Production deployment should use signed command envelopes, mTLS/OIDC/RBAC, persistent encrypted edge storage, real satellite transport, real sensor/PLC adapters, PostgreSQL/TimescaleDB and observability.

