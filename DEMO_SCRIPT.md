# SIH26060 — 3.5 Minute Judge Demo

## 0:00 — Problem

“Antarctic research stations are remote. The mainland cannot assume a continuous communication link. A normal remote-management system can therefore fail exactly when operators need it.”

## 0:20 — Control tower

Show Maitri and Bharati.

Point out:
- link state and simulated satellite bandwidth (kbps)
- current collection schedule
- pending requests
- station health
- synchronization status

## 0:45 — Failure

Set Maitri to OFFLINE.

Say:
“We are now simulating a satellite communication outage. The station is still autonomous.”

Request:
`Maitri → collection every 6 hours`

Show:
`QUEUED`

## 1:20 — Recovery

Restore the link.

Click:
`Sync`

Show:
`QUEUED → SENT → CHECKED → APPLIED → CONFIRMED`

Point out that the current station value is now 6h, and that the audit trail logs the exact payload size of the sync — this is a bandwidth-constrained link, not a normal cloud connection.

## 1:50 — Safety

Request:
`1 hour`

Sync it.

Show:
`REJECTED — station policy`

Say:
“The central system never bypasses the local safety checkpoint.”

## 2:15 — Logistics planning

Open Logistics Plan.

Show:
- burn-rate forecast per consumable (food, medical, spares, science)
- days remaining until each hits a critical threshold
- the safe resupply-window recommendation, gated on live wind/visibility/ice risk

## 2:45 — Predictive analytics

Open AI Insights.

Show:
- connectivity risk score
- workload recommendation
- priority recommendation
- safe scheduling recommendation

Say: “This isn't a static demo script — it's computed live from the station's current telemetry, queue depth and simulated link quality.”

## 3:15 — Closing

“Our innovation is the operating model: intent-based control, local autonomy, durable synchronization under constrained bandwidth, station-side safety, logistics planning and live predictive analytics. A satellite outage delays a change; it does not stop the station.”
