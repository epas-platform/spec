# EPAS v1.4 — Edge Nodes, NATS JetStream, and Flutter Clients

> **Status:** Draft — Normative
> This specification defines the Edge Node deployment class, the NATS JetStream leaf-node event topology that supports it, and the Go-aggregator / Flutter-client pattern used for mobile and tactical deployments. This specification supersedes the v1.4 Edge/NATS/Flutter Addendum in `1.4 Planning/`.

---

## 1. Purpose of Edge Deployment Specification

EPAS v1.4 extends the platform to deployments where the central hub assumption breaks. Three scenarios motivate this specification:

1. **Constrained hardware** — embedded systems, single-board computers, and mobile sensor nodes that cannot run the full platform stack.
2. **Intermittent connectivity** — environments where wide-area network availability cannot be assumed and cannot be made reliable.
3. **Mobile and tactical clients** — Flutter applications on iOS, Android, and desktop that must continue operating through disconnection and later reconcile with the central platform.

This specification is **normative** for platforms that claim Edge conformance. Central-hub-only platforms are not required to implement this specification but MUST NOT label themselves Edge-conformant without doing so.

---

## 2. Edge Deployment Relationship to Other Specifications

- **Specification 02 (SDK-First)** — Edge clients consume the platform through an SDK, typically through a language binding appropriate for the constrained runtime (TypeScript for browser, Dart for Flutter, Python for Python-capable edge nodes).
- **Specification 03 (Contract-Based Trust)** — Edge contracts buffer locally when disconnected and are recorded on the central ledger upon reconnection. Edge nodes MUST NOT fabricate contracts.
- **Specification 08 (Event-Driven Architecture)** — NATS JetStream is the primary event backbone in v1.4. This specification defines the leaf-node topology used at the edge.
- **Specification 09 (Identity and Delegation)** — Edge agents carry the same DID-based identity as central agents. Disconnected operation does not suspend identity verification.

---

## 3. Edge Node Deployment Class

An **Edge Node** is a platform deployment that:

- Runs on constrained or intermittently-connected hardware.
- Publishes events to a central hub via NATS JetStream leaf-node replication.
- Maintains local state sufficient to operate during disconnection.
- Reconciles state and replays buffered events when connectivity is restored.

Edge Nodes are distinct from Central Services (which assume continuous connectivity) and from Client Applications (which consume the platform via SDK). Edge Nodes are themselves platform participants that produce events, accept contracts, and maintain local authority.

### 3.1 Edge Node Reference Architecture

```
┌───────────────────────────────────────────────┐
│              EDGE NODE                         │
│                                                │
│  ┌──────────────┐   ┌──────────────────────┐  │
│  │  Local SDK   │   │  NATS JetStream      │  │
│  │              │   │  leaf node           │  │
│  │  Commands    │◀──┤                      │  │
│  │  Queries     │   │  ● Local streams     │  │
│  │  Events      │──▶│  ● Persistent buffer │  │
│  └──────┬───────┘   │  ● Auto-reconnect    │  │
│         │           └──────────┬───────────┘  │
│         ▼                      │              │
│  ┌──────────────┐              │              │
│  │ Local State  │              │              │
│  │ (SQLite or   │              │              │
│  │  equivalent) │              │              │
│  └──────────────┘              │              │
└────────────────────────────────┼──────────────┘
                                 │ leaf-node bridge
                                 ▼
┌───────────────────────────────────────────────┐
│              CENTRAL HUB                       │
│  NATS JetStream cluster                       │
│  Contract Ledger                              │
│  Central Services                             │
└───────────────────────────────────────────────┘
```

### 3.2 Edge Node Invariants

- **Local events persist to durable storage** before acknowledgment. An edge node whose buffer is in-memory only is non-conformant.
- **Buffered events replay on reconnect** with exactly-once semantics against central consumers.
- **Local contracts carry a tentative status** until recorded in the central ledger. Tentative contracts are ledger-recordable upon reconnection.
- **Local state reconciles deterministically** when conflicting central events arrive (central wins by default; conflict resolution is explicit).

---

## 4. NATS JetStream Leaf Node Topology

EPAS v1.4 selects NATS JetStream as the primary event backbone (see specification 08). At the edge, NATS leaf nodes provide the offline-tolerant transport.

### 4.1 Leaf Node Properties

A NATS leaf node:

- Runs as a local NATS server on the edge hardware.
- Holds persistent streams that buffer events locally.
- Connects to the central NATS cluster as a leaf (downstream-initiated, upstream-authenticated).
- Replays buffered messages to the central cluster when the connection is available.
- Receives relevant messages from the central cluster for local consumption.

### 4.2 Stream Configuration

Every edge node MUST define at minimum three local streams:

| Stream Name | Purpose | Retention |
|-------------|---------|-----------|
| `edge.{node_id}.events` | Events originating at this edge node | Regulatory retention or 30 days minimum |
| `edge.{node_id}.contracts_tentative` | Contracts offered locally, pending central ledger recording | Until reconciled with central ledger |
| `central.inbox.{node_id}` | Events and commands from the central hub targeting this node | 7 days minimum |

Stream retention is tuned per deployment but MUST satisfy regulatory requirements of the tenant.

### 4.3 Subject Conventions

Edge events are published to subjects following a consistent namespace:

```
{platform}.{tenant}.{environment}.edge.{node_id}.{event_type}
```

Example:

```
example-platform.example-corp.prod.edge.node-alpha.sensor.scan
```

Central services subscribe to wildcard patterns:

```
example-platform.example-corp.prod.edge.*.sensor.scan
```

### 4.4 Reconnection and Replay

When an edge node reconnects to the central hub:

1. The leaf node replays locally-buffered events in original order.
2. The central hub ingests the events, deduplicating by message identifier.
3. Tentative contracts produced at the edge are promoted to full contract-ledger entries per specification 03.
4. Central-to-edge messages that accumulated during the disconnect are delivered to the edge.
5. State reconciliation runs per Section 3.2.

Replay MUST preserve per-stream ordering. Cross-stream ordering is not preserved and MUST NOT be assumed by consumers.

### 4.5 Security Requirements

- Leaf-to-hub authentication uses workload identity (mutual TLS with workload certificates per specification 09).
- Leaf-to-hub traffic is encrypted in transit.
- Local storage on the edge node MUST be encrypted at rest where the edge hardware supports it.
- The edge node's NATS credentials rotate on a declared schedule and on suspected compromise.

---

## 5. Edge Contract Handling

### 5.1 Contract Offer at the Edge

When an edge node offers a contract while disconnected:

1. The edge SDK produces the contract envelope with a `status: "offered_tentative"` flag.
2. The tentative contract is written to `edge.{node_id}.contracts_tentative`.
3. Local execution MAY proceed if the contract is classified as locally-authoritative (see Section 5.3).
4. On reconnect, the tentative contract is submitted to the central ledger for recording.
5. Central recording promotes the tentative contract to a full ledger entry with `status: "accepted"` or `status: "refused"`.

### 5.2 Tentative Contract Revocation

A tentative contract that is refused by the central ledger on reconnect is revoked. Local state changes produced under the revoked contract are rolled back where possible, and a violation event is emitted to the governance plane per specification 06.

### 5.3 Locally-Authoritative Contracts

A contract is **locally-authoritative** when the tenant's policy declares that the edge node's executor has authority to accept the contract without central ledger acknowledgment. Locally-authoritative contracts:

- Are declared in the tenant's agentic governance policy (specification 06).
- Are bounded by the edge node's declared scope (resources, actions, time window).
- Produce ledger entries that are still externally anchored on the central platform's schedule (specification 03, Section 8.3).

Contracts outside the locally-authoritative scope MUST NOT execute during disconnection. They are held in the tentative queue until the central ledger is reachable.

---

## 6. Go Aggregator Pattern

For deployments that expose platform capabilities to mobile, tactical, or low-resource clients, EPAS v1.4 recognizes an intermediate service — the **Go Aggregator** — that bridges NATS JetStream to client-friendly protocols.

### 6.1 Go Aggregator Responsibilities

The Go Aggregator:

- Runs as a horizontally-scalable service (Go is specified as a reference language; other languages satisfying the requirements are acceptable).
- Subscribes to relevant NATS streams for the tenants it serves.
- Translates NATS events into client-friendly formats (WebSocket JSON messages, Server-Sent Events, Protobuf-over-WebSocket).
- Serves WebSocket connections to Flutter and web clients.
- Optionally bridges platform events to tactical protocols (see Section 8).
- Holds no authoritative state — the aggregator is stateless relative to contracts and events.

### 6.2 Aggregator URL Namespace

Aggregator endpoints follow a tenant-scoped URL structure:

```
wss://aggregator.{platform}.example/t/{tenant}/ws/{stream_type}/{entity_id}
```

Example:

```
wss://aggregator.example-platform.example/t/example-corp/ws/node-events/node-alpha
```

### 6.3 Aggregator as SDK Consumer

The aggregator is itself an SDK consumer (specification 02). It consumes the platform through a Go SDK and exposes platform capabilities through its own language-specific client bindings. Flutter clients consume a Dart SDK that calls the aggregator; the aggregator calls the Go SDK; the Go SDK calls the platform.

A Flutter client does not call the platform directly. The aggregator is the trust-boundary-aware intermediary that makes offline-tolerant mobile clients possible.

---

## 7. Flutter Client Architecture

### 7.1 Flutter SDK Requirements

Flutter applications targeting EPAS v1.4 consume a Dart SDK that:

- Implements the SDK-first contract from specification 02.
- Returns `ContractHandle` objects from mutating calls.
- Surfaces refusal as a structured outcome.
- Handles WebSocket reconnection to the aggregator transparently.
- Buffers outgoing commands locally when disconnected.

### 7.2 Flutter Finite State Machine Requirement

Client state management for contract lifecycles MUST be implemented as a finite state machine. Specifically:

- Every `ContractHandle` in a Flutter client is modeled as an FSM with states from specification 02 Section 10.2.
- State transitions are explicit, not implicit.
- The FSM is implemented using a declarative state-machine library appropriate for the language:
  - **Dart / Flutter**: `bloc`, `flutter_bloc`, or an equivalent FSM library.
  - **TypeScript**: `xstate` v5 or equivalent.
  - **Python**: `transitions` or equivalent.
  - **Go**: hand-rolled FSM with explicit state types.

Ad-hoc state management that allows impossible state transitions is non-conformant. The FSM requirement prevents the entire class of bugs where a contract appears to be simultaneously accepted and refused, or where the UI displays "pending" for a terminal state.

### 7.3 Flutter Offline Mode

Flutter clients operating offline:

- Continue to render cached state from the aggregator's last successful sync.
- Queue outgoing commands locally with durable storage.
- Display a clear offline indicator.
- Attempt reconnection on a bounded exponential backoff.
- Replay queued commands on reconnection, respecting idempotency keys (specification 04).

---

## 8. Tactical Integration (Optional)

For deployments in tactical, operational-awareness, or emergency-response contexts, EPAS v1.4 defines an optional bridge to the Cursor-on-Target (CoT) ecosystem.

### 8.1 Tactical Bridge Responsibilities

The tactical bridge is an optional component of the Go Aggregator that:

- Translates platform events (especially anomaly, alert, and status events) into CoT XML messages.
- Publishes to a standards-compliant CoT server (Team Awareness Kit server).
- Receives CoT events from tactical systems and translates them to platform events.
- Maps platform identity to CoT callsigns under a deterministic scheme.

### 8.2 Tactical Bridge Constraints

The tactical bridge:

- Consumes the platform through the SDK like any other client.
- Does not bypass the Contract-Based Trust Model — CoT-triggered operations produce contract offers on the platform side.
- Maintains the tenancy boundary: tactical events for one tenant are not exposed to another tenant's CoT feed.

Tactical integration is **optional**. Platforms without tactical use cases do not implement this section.

---

## 9. URL Namespace at the Edge

Edge deployments use the canonical URL namespace from specification 04 with edge-specific extensions:

```
# Central platform (unchanged)
https://{platform}.example/t/{tenant}/api/rest/v1/{resource}/commands/{command}

# Edge node direct access (limited)
https://edge-{node_id}.{tenant}.{platform}.example/t/{tenant}/edge/api/v1/{resource}

# Aggregator WebSocket
wss://aggregator.{platform}.example/t/{tenant}/ws/{stream}/{entity}

# Tactical bridge (optional)
https://cot-bridge.{platform}.example/t/{tenant}/cot/feed
```

Edge-specific endpoints MUST carry the tenant identifier in the path. Implicit-tenant edge endpoints are non-conformant.

---

## 10. Reconciliation and Conflict Resolution

### 10.1 Default Resolution Rules

When an edge node reconnects and the central hub reports conflicting state:

- **Contract conflicts** — The central ledger is authoritative. Tentative contracts that cannot be recorded are revoked.
- **Configuration conflicts** — The central configuration is authoritative. The edge node reloads.
- **Event conflicts** — Events are append-only; the central hub accepts all events that pass deduplication. Apparent "conflicts" in observed state are resolved by event ordering at the central hub.
- **Resource conflicts** — Resource-level conflicts (e.g., two edges claiming the same resource) resolve by tenant-defined policy. Default is earliest-timestamp wins.

### 10.2 Explicit Conflict Resolution

Tenants MAY declare conflict resolution policies per resource type in the control plane. Declared policies override the defaults. Undeclared conflicts fall through to the defaults above.

---

## 11. Throughput and Capacity

### 11.1 Edge Node Sizing

An edge node typically handles:

- 10 to 10,000 events per second local publish rate, depending on deployment.
- 1,000 to 1,000,000 events in the local buffer during a multi-hour disconnection.
- 1 to 100 tentative contracts per hour.

Edge hardware is sized for the local event rate plus a disconnect-buffer multiplier. Sizing under-provisions the reconnect replay burst; conformant deployments declare a minimum reconnect bandwidth adequate for the expected buffer size.

### 11.2 Aggregator Sizing

An aggregator typically handles:

- 100 to 10,000 concurrent WebSocket connections per instance.
- 100 to 100,000 NATS messages per second, per tenant.
- Translation overhead is negligible when message formats are structurally similar; nontrivial when formats differ substantially.

Aggregator instances are horizontally scalable. Connection affinity is per-client; event delivery guarantees are at-least-once.

---

## 12. Observability at the Edge

### 12.1 Edge Node Telemetry

Edge nodes emit telemetry to the central observability plane over the same NATS streams used for platform events. Disconnected edge nodes buffer telemetry alongside domain events; telemetry replays on reconnect.

### 12.2 Edge Dashboards

The central observability plane MUST provide visibility into:

- Edge node connectivity state (connected, disconnected, degraded).
- Edge buffer depth per node.
- Tentative contract count per node.
- Reconnect replay status.
- Reconciliation conflict rate.

---

## 13. Edge Deployment Conformance Requirements

A platform conforms to EPAS v1.4 Edge, NATS, and Flutter requirements when:

1. Edge nodes run NATS JetStream leaf nodes with durable local streams per Section 4.2.
2. Edge subject naming follows the canonical scheme in Section 4.3.
3. Edge-initiated contracts are handled per Section 5, with tentative-contract semantics and central-ledger reconciliation on reconnect.
4. Go Aggregator (or equivalent) is SDK-based and stateless per Section 6.
5. Flutter clients implement contract lifecycles as FSMs per Section 7.2.
6. Edge URLs carry tenant identifiers explicitly per Section 9.
7. Reconciliation policies are declared or default-selected per Section 10.
8. Edge telemetry flows through the same observability plane as central services.
9. Edge and central ledger entries share the same external anchoring scheme per specification 03.

Platforms without edge deployment MAY declare `edge_deployment_supported: false` in `conformance.yaml` and skip this specification. Platforms claiming edge conformance MUST satisfy all items above.

---

## 14. Edge Deployment Non-Goals

This specification does not:

- Mandate specific edge hardware, operating systems, or form factors.
- Specify the exact mechanism for local encryption at rest (implementation-specific).
- Define mobile-platform-specific UI patterns.
- Replace cellular, satellite, or mesh networking specifications.
- Prescribe specific CoT servers, tactical software, or military standards.

Deployment concerns — hardware selection, transport selection, operating system hardening — remain tenant-specific and out of scope for this specification.
