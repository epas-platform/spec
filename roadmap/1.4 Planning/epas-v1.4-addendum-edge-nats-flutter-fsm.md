# EPAS v1.4.0 Addendum — Edge Sensors, NATS, Flutter & State Machines

> Addendum to EPAS v1.3.0 and the v1.4 update plan. Decisions finalized 2026-04-05.
> Reference implementation: hlidskjalf/huginn (Distributed SDR Sensor Platform)

---

## 1. Event Bus: NATS JetStream (Confirmed)

The Kafka Phase 1 → NATS Phase 2 migration path from v1.3.0 is collapsed.
Greenfield projects start at NATS JetStream directly.

**Rationale:**
- NATS leaf node topology is purpose-built for edge-to-hub deployments (IoT, mesh networks, mobile nodes)
- Sub-millisecond latency vs Kafka throughput-first model
- Single ~12MB binary, runs on ARM (RPi, embedded Linux) without JVM
- Native MQTT bridge — IoT sensors publish to NATS subjects without a separate broker
- JetStream provides persistence and replay when edge nodes reconnect after link drops

**Subject naming convention:**
```
<platform>.<node_id>.<event_type>

huginn.node-alpha.scan
huginn.node-alpha.alert
huginn.node-alpha.device_status
```

Tenant isolation is enforced at the NATS **account** level, not the subject level.

**Kafka** remains appropriate for projects requiring Kafka Connect or ksqlDB.

---

## 2. Flutter — 5th Client Class (via Aggregator Proxy)

The client taxonomy from §3 of the main spec is extended:

| Client Type | Examples | SDK Required |
|-------------|----------|--------------|
| Web UI | React, Next.js | Yes (TypeScript SDK) |
| CLI | Go binary | Yes (Go SDK) |
| Automation | Python scripts, CI jobs | Yes (Python SDK) |
| Agents | Supervisor, worker agents | Yes (Python / TS SDK) |
| **Mobile/Embedded** | **Flutter (iOS/Android/Desktop)** | **Via aggregator proxy** |

Flutter does **not** receive a Dart SDK. Flutter clients communicate with a **Go aggregator**
that is itself a Go SDK consumer. The aggregator translates ContractHandle lifecycle events
into a Flutter-friendly WebSocket protocol.

**Principle**: Flutter expresses intent to the aggregator. The aggregator holds contracts,
manages ANP envelopes, and emits simplified state events downstream. Flutter never sees
raw contract objects.

**Flutter-facing protocol** (WebSocket JSON):
```json
{ type: sweep_started, sweep_id: ..., node_id: ..., ts: 0 }
{ type: alert, level: ANOMALY, freq_mhz: 433.9, node_id: ..., ts: 0 }
{ type: contract_refused, reason: ..., operation: start_sweep }
```

---

## 3. State Machine Requirement for Contract Lifecycle

All SDK implementations MUST model the ContractHandle lifecycle as a finite state machine.

**Contract FSM:**
```
         +----------------------------------+
         v                                  |
[submitted] -> [accepted] -> [in_progress] -> [completed]
                  |                          |
                  +--------> [refused]       +---> [failed]
```

**Recommended implementation per SDK:**
- TypeScript SDK: XState v5
- Go SDK: lightweight FSM (hand-rolled or `looplab/fsm`)
- Python SDK: `transitions` library or hand-rolled dataclass FSM
- Flutter/aggregator: Go FSM on aggregator side; Flutter receives state events

**State machine scope extends to domain workflows**, not just contracts.
Example — sweep lifecycle:
```
idle -> baseline_capturing -> scanning -> stopping -> reporting -> idle
```

Each domain SDK method initiating a multi-step operation SHOULD return a workflow handle
backed by a FSM, not just a ContractHandle.

---

## 4. Edge Node Deployment Pattern

EPAS now recognizes **edge sensor nodes** as a deployment class.

**Characteristics:**
- Runs on constrained hardware (ARM SBC, embedded Linux)
- Intermittent connectivity (mesh networks, LTE failover)
- Local operation capability (continues when disconnected from hub)
- Reconnects and replays buffered events via NATS JetStream

**Reference implementation**: hlidskjalf/huginn (SDR sensor node — Python/FastAPI)

**Architecture pattern:**
```
Edge Node (Python/FastAPI + NATS leaf)
    | NATS leaf node connection (auto-reconnect + JetStream replay)
    v
Hub Aggregator (Go + NATS hub)
    | Go SDK
    v
Ravenhelm Platform (Bifrost, Freyr, contracts)
```

**Tactical mesh variant** (BATMAN-adv):
```
N x Edge Nodes (BATMAN-adv mesh)
    -> NATS leaf nodes (auto-route through mesh)
        -> Go Aggregator (hub node)
            -> Flutter COP (tablets/phones)
            -> ATAK (Android devices)
```

---

## 5. ATAK / CoT Integration Pattern

For deployments requiring tactical situational awareness, the Go aggregator bridges
Ravenhelm events to the TAK ecosystem.

**Stack:**
- FreeTAKServer (open source, Python) — TAK protocol router
- CoT (Cursor on Target) — XML event format used by ATAK/WinTAK/iTAK/WebTAK
- Go aggregator — translates NATS alert events to CoT XML, POSTs to FreeTAKServer REST API

**Event mapping:**

| Platform event | CoT type | ATAK appearance |
|---|---|---|
| Node GPS position | `b-m-p-s-p-op` | Sensor/OP marker |
| Alert (anomaly) | `b-r-f-h` | Hostile RF contact |
| Alert (tracker/bug) | `b-r-f-h` | Hostile contact, red |
| Sweep start/stop | `t-x-m-c` | Mission checkpoint |

---

## 6. Canonical URL Addressing for Edge Services

Edge node local APIs are internal. All external access routes through the Go aggregator
following the Ravenhelm URL Namespace Schema.

**Aggregator endpoints (platform-side, tenant-scoped):**
```
/t/{tenant}/api/{platform}/nodes                         -> list nodes
/t/{tenant}/api/{platform}/nodes/{node_id}/alerts        -> alert queries
/t/{tenant}/api/{platform}/nodes/{node_id}/sweep         -> sweep mutations
/t/{tenant}/api/{platform}/nodes/{node_id}/mode          -> mode control
```

**Flutter WebSocket:**
```
wss://{env}.{location}.{substrate}.ravenhelm.ai/t/{tenant}/app/{platform}/ws
```

**NATS namespace (per tenant NATS account):**
```
{platform}.{node_id}.{event_type}
```
