# EPAS v1.4 — Event-Driven Architecture

> **Status:** Draft — Normative
> This specification defines the Event Plane for EPAS v1.4 platforms. It supersedes Section 11 of the v1.3 scaffold. The most substantive change from v1.3 is the promotion of NATS JetStream to primary event backbone; Kafka and Redpanda remain permitted for compliance-driven retention scenarios.

---

## 1. Purpose of EPAS v1.4 Event-Driven Architecture

EPAS v1.4 defines the Event Plane — the asynchronous messaging backbone through which platform services communicate state changes. The Event Plane is the system of record for **what happened**. The Contract Plane (specification 03) is the system of record for **what was agreed to**. These two planes are orthogonal and MUST NOT be conflated.

This specification is **normative**. Platforms that deploy without an Event Plane, or that implement the Event Plane synchronously, are non-conformant.

---

## 2. Event-Driven Architecture Relationship to v1.3

v1.3 Section 11 defined:

- Event-driven architecture as mandatory.
- A phased rollout: Kafka/Redpanda for Phase 1 (audit retention), NATS JetStream for Phase 2 (latency optimization).
- CloudEvents as the standard envelope.
- AsyncAPI as the contract format.

v1.4 amends this to:

- **NATS JetStream is the recommended primary event backbone** for new deployments.
- **Kafka and Redpanda remain permitted** where regulatory retention requirements favor Kafka's archival ecosystem, or where existing Kafka deployments have substantial investment.
- **CloudEvents and AsyncAPI remain mandatory** without change.
- **Every event MUST carry a `contractid` CloudEvents extension attribute** — this is new in v1.4 and is non-negotiable. CloudEvents 1.0 requires extension attribute names to be lowercase with no separators; the same identifier appears as `contract_id` in JSON envelopes and prose, and as `contractid` when it is the CloudEvents attribute name.

v1.3 Phase 1 / Phase 2 language is retired. New deployments select a backbone on merit, not on phase.

---

## 3. Why Event-Driven is Mandatory

Event-driven architecture is mandatory in EPAS v1.4 for reasons that do not change between minor versions:

- **Decoupling** — Services do not need synchronous knowledge of their consumers.
- **Audit** — Every state change produces a durable record suitable for forensic replay.
- **Resilience** — Consumer failure does not block producers.
- **Replayability** — Downstream systems rebuild derived state from the event log.
- **Multi-consumer economics** — A single event serves multiple consumers without fan-out in the producer.

Platforms that use synchronous RPC as their primary coordination mechanism are not EPAS v1.4 platforms. Synchronous RPC is permitted as an implementation detail within the Internal Service API (specification 04) but MUST NOT be the substrate for cross-service coordination.

---

## 4. Event Bus Selection Criteria

Teams selecting an event bus MUST evaluate candidates against these criteria:

| Criterion | Measurement |
|-----------|-------------|
| **Latency (p99)** | End-to-end producer-to-consumer latency under production load |
| **Throughput (sustained)** | Messages per second per partition or per subject |
| **Durability** | Retention horizon; replication factor; recovery from broker loss |
| **Ordering guarantees** | Per-partition, per-subject, or per-key ordering |
| **Delivery semantics** | At-most-once, at-least-once, exactly-once |
| **Replayability** | Ability to consume from any offset; retention of historical messages |
| **Retention economics** | Cost per TB per month for regulatory retention horizons |
| **Operational maturity** | Community, tooling, observability integrations |
| **Edge suitability** | Native leaf-node or extension-node topology (specification 07) |
| **Protocol openness** | Open-source implementations; open protocol spec |

EPAS v1.4 recommends NATS JetStream as a reasonable default across these criteria, but specific deployments MAY select alternatives when justified against the criteria above.

---

## 5. Primary Event Bus — NATS JetStream

### 5.1 Why NATS JetStream

NATS JetStream satisfies the EPAS v1.4 event bus requirements with specific strengths:

- **Sub-millisecond latency** at the core dispatch layer.
- **Native leaf-node topology** for edge deployments (specification 07).
- **Persistent streams with replay** from any point in the stream.
- **Simple operational footprint** — single binary, minimal external dependencies.
- **ARM and constrained-hardware compatibility** — runs on edge devices the Kafka JVM cannot.
- **MQTT and WebSocket bridges** without requiring a separate broker.

### 5.2 Required NATS JetStream Configuration

Every EPAS v1.4 deployment using NATS JetStream MUST:

- Configure streams with explicit retention policies per stream.
- Enable replication with a minimum replica count of 3 for production streams.
- Configure per-stream maximum age and maximum size aligned with the tenant's retention requirements.
- Separate high-volume activity streams from contract-adjacent streams.
- Use deterministic subject naming per Section 7.

### 5.3 Stream Taxonomy

Every platform deploying NATS JetStream MUST define at minimum the following stream classes:

| Stream Class | Purpose | Retention |
|--------------|---------|-----------|
| `ops.*` | Operational events: task.started, task.succeeded, task.failed | 7–90 days |
| `audit.*` | Audit events referenced by the Contract Plane | Regulatory retention (years) |
| `telemetry.*` | Observability events | 30 days (typical) |
| `alerts.*` | Alert events for AIOps and SRE | 7–30 days |
| `edge.*` | Events originating at edge nodes (specification 07) | Per edge-node policy |

---

## 6. Permitted Alternatives

### 6.1 Kafka / Redpanda (PERMITTED with justification)

Kafka and Redpanda remain permitted where:

- **Regulatory retention** favors Kafka's mature archival and compaction ecosystem (e.g., 7-year financial retention with Tiered Storage).
- **Existing deployment investment** makes migration costly relative to platform benefit.
- **Specific ecosystem integrations** (Kafka Connect, KSQL, Schema Registry) are load-bearing for the deployment.

Platforms using Kafka or Redpanda:

- MUST satisfy all other requirements of this specification (CloudEvents envelope, AsyncAPI contracts, `contractid` extension attribute on every event).
- SHOULD document the justification in `conformance.yaml`.
- SHOULD evaluate NATS JetStream at each major architectural review.

### 6.2 Cloud-Managed Event Buses (PERMITTED)

Cloud-managed event buses (AWS EventBridge, Google Pub/Sub, Azure Event Grid) are permitted for specific use cases where managed service economics clearly favor them. These services are typically adequate for:

- Cross-service notification fan-out.
- Event-driven Lambda or Cloud Function invocation.
- Serverless integration patterns.

These services are typically **inadequate** as the platform's primary event backbone because of throughput, retention, and ordering limitations. Use them for integration layers, not the core event plane.

### 6.3 RabbitMQ / Pulsar (PERMITTED, situational)

RabbitMQ is permitted for task-queue patterns and low-volume integrations; it is not recommended as the primary event backbone for high-throughput platform coordination.

Apache Pulsar is permitted and is a reasonable alternative to Kafka; it satisfies most EPAS v1.4 requirements natively. Pulsar's operational footprint is heavier than NATS JetStream, making it less suitable for edge deployments.

---

## 7. Event Plane Subject and Topic Conventions

All event bus implementations use the same logical subject structure. Concrete mappings to Kafka topic names or NATS subjects differ only in separators.

### 7.1 Canonical Subject Structure

```
{platform}.{tenant}.{environment}.{domain}.{event_type}
```

Components:

- `{platform}` — the platform name (stable identifier).
- `{tenant}` — the tenant slug.
- `{environment}` — `dev`, `test`, `staging`, or `prod`.
- `{domain}` — the functional domain (`contracts`, `tasks`, `agents`, `ops`, `telemetry`, `edge.{node_id}`).
- `{event_type}` — the specific event type (`offered`, `accepted`, `refused`, `started`, etc.).

Example:

```
example-platform.example-corp.prod.contracts.accepted
example-platform.example-corp.prod.tasks.failed
example-platform.example-corp.prod.edge.node-alpha.sensor.anomaly
```

### 7.2 Subject Namespace Rules

- Subjects MUST include tenant and environment. Implicit-tenant or implicit-environment subjects are non-conformant.
- Subjects SHOULD group by domain so consumers can subscribe to domain wildcards.
- Cross-tenant subscriptions are not permitted at the transport layer. Cross-tenant aggregation happens at the Control Plane, after authorization.

---

## 8. CloudEvents Envelope (Mandatory)

Every event MUST be encoded as a CloudEvents 1.0 envelope.

### 8.1 Required CloudEvents Attributes

Every event MUST include the following CloudEvents-specified attributes:

| Attribute | Value |
|-----------|-------|
| `specversion` | `1.0` |
| `id` | Globally unique event identifier (UUIDv7 recommended) |
| `source` | URI reference identifying the producing service and tenant |
| `type` | Reverse-DNS event type (`com.example.platform.contract.accepted`) |
| `time` | RFC 3339 timestamp at production |
| `datacontenttype` | MIME type of the event data payload |

### 8.2 EPAS v1.4 Extension Attributes (Mandatory)

Every EPAS v1.4 event MUST additionally include these extension attributes:

| Attribute | Purpose |
|-----------|---------|
| `tenant` | Tenant identifier |
| `environment` | `dev` / `test` / `staging` / `prod` |
| `correlationid` | Correlates related events within a single operation |
| `causationid` | Identifies the event that directly caused this event |
| `contractid` | The contract under which this event was produced |
| `operationid` | The originating operation |
| `schemaid` | Identifier of the payload schema |
| `schemaversion` | Version of the payload schema |

An event that lacks `contractid` is non-conformant except for events from the Control Plane that configure the platform itself (policy changes, agent registration) where the contract model's evidentiary role is satisfied by a ledger entry rather than a per-event field.

### 8.3 Payload

The CloudEvents `data` attribute contains the event payload. Payload schemas are defined in AsyncAPI specifications per Section 9.

---

## 9. AsyncAPI Contracts

Every producer-consumer relationship MUST be defined by an AsyncAPI 2.x or 3.x specification. The AsyncAPI document:

- Declares channels (subjects or topics) the service publishes to and subscribes from.
- Defines message schemas with explicit required fields.
- Documents event type names and version ranges.
- Is published alongside the service source code.
- Is consumed by CI validators that verify schema compliance at emit time.

Services that emit events not declared in their AsyncAPI specification are non-conformant. Events that violate the declared schema are defective.

---

## 10. Dead Letter Queues and Retry

### 10.1 Dead Letter Semantics

Every consumer MUST handle poison messages with an explicit dead-letter policy:

- Events that fail deterministic validation are routed to a DLQ stream named `dlq.{original_stream}`.
- Events in the DLQ carry the original event plus error metadata (stage, error message, consumer identity, retry count).
- DLQ entries are retained for at minimum the regulatory retention period and trigger operator alerts.

### 10.2 Retry Discipline

Consumers retry transient failures with bounded exponential backoff. Retries:

- Use an exponential backoff with jitter.
- Cap at a declared maximum retry count (typically 3–10, specific to the consumer).
- Escalate to DLQ after the maximum is exhausted.
- Are observable: every retry increments a metric.

### 10.3 Retry-Induced Contract Considerations

Retries do not create new contracts. A retry executes under the original contract. If the original contract has expired, the retry MUST NOT execute — the consumer escalates to DLQ rather than creating an out-of-contract action.

---

## 11. Event Ordering

### 11.1 Per-Key Ordering

Events for a given entity MUST be ordered within their subject or partition. The partition key is the entity identifier:

- Per-contract ordering: partition by `contractid`.
- Per-operation ordering: partition by `operationid`.
- Per-thread or per-session ordering: partition by `threadid` or `sessionid` per specification 04.

Cross-entity ordering is not guaranteed and MUST NOT be assumed by consumers.

### 11.2 Ordering on Edge Reconnect

Edge nodes (specification 07) replay events in per-stream order. Cross-stream ordering during replay is not preserved. Consumers that depend on cross-stream ordering MUST use the correlation and causation fields to reconstruct ordering deterministically.

---

## 12. Schema Evolution

### 12.1 Compatibility Policy

Every payload schema evolves under **backward-compatible** and **forward-compatible** rules within a major version:

- New optional fields are permitted.
- New enum values are permitted if consumers handle unknown values gracefully.
- Existing field types MUST NOT change.
- Existing field names MUST NOT be repurposed.

Breaking changes require a major version bump in `schemaversion`. Multiple schema versions may coexist during migration; consumers declare supported versions.

### 12.2 Schema Registry

Schemas are published to a schema registry reachable by both producers and consumers. CI validates that emitted events match the published schema. Schema drift between declared and emitted schemas is non-conformant.

---

## 13. Event Plane and Contract Plane Interaction

The Event Plane records **what happened**. The Contract Plane records **what was agreed to**. The two planes interact through mandatory cross-references:

- Every task event carries the `contractid` it was dispatched under.
- Every contract ledger entry references the event stream position at which it was recorded.
- The combined audit view joins events and ledger entries by `contractid` and `operationid`.

Platforms that attempt to merge the two planes into a single stream are non-conformant. The two planes have incompatible retention, throughput, and durability requirements per specification 03.

---

## 14. Observability of the Event Plane

The Event Plane is itself observed:

- Per-subject message rate.
- Per-consumer lag.
- DLQ depth.
- Retry rate per consumer.
- Schema validation failure rate.
- Edge-node buffer depth (specification 07).

These metrics feed the Observability Plane. Event Plane failures (broker unavailable, topics corrupted, consumers stalled) trigger alerts and escalate per the tenant's incident-response policy.

---

## 15. Event Plane Migration from v1.3 Phase-Based Guidance

Teams operating v1.3 Phase 1 (Kafka) deployments migrate to v1.4 as follows:

1. **Evaluate** the current Kafka deployment against the NATS JetStream recommendation in Section 5.
2. **Decide** — keep Kafka (with justification per Section 6.1) or migrate.
3. **If migrating** — run dual-producer emit (events published to both buses) during migration, cut consumers over in waves, retire Kafka when all consumers are migrated and retention requirements are met.
4. **If keeping Kafka** — declare the justification in `conformance.yaml` and ensure all other v1.4 event plane requirements are satisfied.

v1.3 Phase 2 deployments that already run NATS JetStream are v1.4-conformant at the event bus layer once `contractid` is added to every event.

---

## 16. Event-Driven Architecture Conformance Requirements

A platform conforms to EPAS v1.4 Event-Driven Architecture when:

1. An event bus is deployed as the primary coordination mechanism.
2. The selected bus is NATS JetStream (recommended) or a permitted alternative per Section 6.
3. Subjects or topics follow the canonical structure in Section 7.1.
4. Every event is encoded as CloudEvents 1.0 with the extension attributes from Section 8.2.
5. Every task and contract-adjacent event includes `contractid`.
6. Every producer-consumer relationship is declared in AsyncAPI.
7. Dead-letter queues are configured with explicit policies (Section 10).
8. Per-entity ordering is preserved via partition keys (Section 11).
9. Schemas evolve under the compatibility rules in Section 12.
10. Event Plane metrics feed the Observability Plane.

Partial conformance during migration is permitted and MUST be declared in `conformance.yaml`.

---

## 17. Event-Driven Architecture Non-Goals

This specification does not:

- Mandate a specific broker vendor or distribution.
- Prescribe the choice between Kafka and Redpanda where either is acceptable.
- Define specific event type names for domains outside the platform's core.
- Replace operational runbooks for brokers.
- Specify payload serialization (JSON, Protobuf, Avro) — this is per-domain.

Payload serialization selection is tenant-specific and out of scope. Consistency within a tenant is required; specific selection is not mandated.
