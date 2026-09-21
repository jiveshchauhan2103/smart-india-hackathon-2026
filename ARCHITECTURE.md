# SIH26060 Technical Architecture

## Design principles

### 1. Disconnection is a normal operating state
The station must not become useless because a mainland link disappears.

### 2. Intent over raw configuration
Researchers ask for a high-level change such as `collect every 6 hours`. They do not edit Kubernetes YAML.

### 3. Station is the final authority
Central systems can request a change; local station policy decides whether it may execute.

### 4. Durable synchronization
Requests have stable IDs and are safe to retry.

### 5. Simulation-first
The edge agent mirrors the interfaces needed by a future Kubernetes/hardware adapter.

## State machine

```text
REQUESTED
    │
    ├── offline ──► QUEUED ── link returns ──► SENT
    │                                         │
    └─────────────────────────────────────────┘
                                              ▼
                                           CHECKED
                                         /         \
                                    rejected       allowed
                                       ▼              ▼
                                   REJECTED        APPLIED
                                                     │
                                                     ▼
                                                  CONFIRMED
```

## Components

- `researcher-control-tower`: operator UX (Digital Twin, Safe Operations, Requests & Sync, Logistics Plan, AI Insights, Audit Trail)
- `central-control-plane`: request lifecycle, audit, station registry, telemetry, logistics planning API, AI insights API
- `sync-engine`: store-and-forward transport abstraction
- `station-edge-agent`: local policy + local state
- `ai-intelligence-core`: risk/workload models — reused live by `central-control-plane`'s `aiController.js` against real telemetry, in addition to being runnable standalone for decision-support demos
- `offline-edge-engine`: dependency-free local decision logic

Every entry point (`server.js`, `agent.js`, `sync.js`, `edgeDecision.js`, the `ai-intelligence-core` scripts, and the dashboard's dev/build/start scripts) runs a Node-version guard first, so an unsupported Node install fails with one clear message instead of a cryptic, version-dependent crash.

## Logistics planning

`logisticsPlan(stationId)` in `central-control-plane/data/store.js` turns raw inventory levels into a forecast: a daily burn rate per consumable, days remaining until a critical threshold, and a resupply-window recommendation gated on live wind/visibility/ice-risk — so a flight or traverse is only proposed when it is actually safe. This is distinct from the inventory *monitoring* already shown on the Digital Twin.

## Predictive analytics / AI insights

`central-control-plane/controllers/aiController.js` normalizes live station telemetry (wind, ice risk, simulated link bandwidth, request queue depth, station health) into the feature space expected by `ai-intelligence-core`'s existing causal risk model and workload advisor, and exposes the result as connectivity risk, workload recommendation, priority recommendation and safe-scheduling recommendation via `/api/ai/insights`. Previously these models only ran as standalone CLI scripts disconnected from the live app.

## Constrained satellite bandwidth

Each station simulates a narrow, Iridium-class satellite link (single-digit to low-double-digit kbps) that degrades further under high wind/ice risk and drops to zero when the station is off the link. Every telemetry sync logs its JSON payload size to the audit trail, so the demo can show — not just claim — that synchronization is designed for a severely bandwidth-constrained link.

## Safety

MVP policy:
- collection interval: 6, 12 or 24 hours
- urgent safety state cannot be overridden by normal requests
- request ID prevents duplicate application
- rollback is a compensating request

Production should add signed commands, approvals, hardware interlocks and station-specific operating envelopes.

## SIH pitch

> “We don't build a cloud dashboard that assumes Antarctica is always online. We build an edge operating layer where disconnection is expected. The station keeps working, requests are stored and prioritized, and synchronization happens when communication permits.”

## Why it scales

The same central control model can support future stations because station-specific policies and adapters stay at the edge.
