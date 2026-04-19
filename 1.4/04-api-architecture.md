# EPAS v1.4 — API Architecture (Command, Query, Internal)

> **Status:** Draft — Normative
> This specification defines the three API surfaces of an EPAS v1.4 platform, their semantic boundaries, and the relationship between APIs, SDKs (specification 02), and contracts (specification 03).

---

## 1. Purpose

EPAS v1.4 API Architecture separates three concerns that are commonly conflated in REST-only designs:

1. **Intent** — how a client declares what it wants to change.
2. **Observation** — how a client reads current and historical state.
3. **Execution** — how services coordinate internally to fulfill intent.

Each concern is served by its own API surface with its own protocol, shape, and semantic rules. The three surfaces are not interchangeable.

This specification is **normative**. Surfaces that blur these boundaries are non-conformant.

---

## 2. Core Architectural Principles

1. **Intent is explicit.** APIs express commands as nouns-and-verbs over resources, not as hidden state changes inside reads.
2. **Execution is asynchronous by default.** APIs do not mutate state synchronously. Acceptance of intent and completion of work are distinct events.
3. **Observation is side-effect-free.** Reads never produce mutations, dispatch tasks, or consume budget against quotas.
4. **All work is governed.** Every API call passes authentication, authorization, delegation verification, and (for mutating calls) contract acceptance before execution begins.
5. **APIs are not the product.** SDKs (specification 02) are the product. APIs are implementation detail and MUST NOT be consumed directly by clients.

---

## 3. API Surface Taxonomy

EPAS v1.4 platforms expose exactly three API surfaces. No additional surfaces are permitted without amendment to this specification.

| Surface | Purpose | Protocol | Consumer |
|---------|---------|----------|----------|
| **Command API** | Express intent to change state | REST over HTTPS, JSON payloads | SDKs only |
| **Query API** | Read and aggregate state | GraphQL over HTTPS | SDKs only |
| **Internal Service API** | Service-to-service orchestration | gRPC with Protocol Buffers | Platform services only |

Each surface has strict semantic rules defined below. Violating the semantic rules — for example, exposing mutations through GraphQL, or letting a CLI call gRPC endpoints directly — is non-conformant.

---

## 4. Command API (REST)

### 4.1 Purpose

The Command API exists solely to **accept intent**. It does not guarantee execution, does not guarantee completion, and does not return domain state.

The Command API MUST:

- Validate identity and delegation.
- Normalize and schema-validate the submitted command.
- Enforce rate limits.
- Return an operation identifier and a contract reference.

The Command API MUST NOT:

- Perform business logic.
- Mutate domain state directly.
- Block on long-running work.
- Return domain query results (use the Query API).

### 4.2 Canonical Endpoint Structure

Every Command API endpoint follows this shape:

```
POST /t/{tenant}/api/rest/v{version}/{resource}/{id}/commands/{command}
```

Components:

- `{tenant}` — the tenant slug, always present, never inferred.
- `v{version}` — the Command API version (`v1`, `v2`, ...). Version bumps are used sparingly.
- `{resource}` — the noun form of the domain entity (`users`, `agents`, `tools`).
- `{id}` — the resource identifier. Collection-level commands MAY omit `{id}`.
- `{command}` — the verb describing intent (`update-email`, `dispatch-task`, `revoke-delegation`).

Example:

```
POST /t/example-corp/api/rest/v1/users/1234/commands/update-email
```

### 4.3 Request Requirements

Every command request MUST include:

- A valid authentication context (OIDC session for humans, workload identity for services, registered agent credential for agents — see specification 09).
- A signed delegation envelope that authorizes the submitted command.
- A correlation identifier (the Endpoint Service generates one if absent).
- An idempotency key. Repeated submissions with the same idempotency key MUST produce the same `contract_id` and MUST NOT create a second contract.

Example request body:

```json
{
  "parameters": {
    "new_email": "user@example.com"
  }
}
```

Command parameters are schema-validated against an OpenAPI specification published per tenant and per version.

### 4.4 Responses

#### 4.4.1 Default Response (Asynchronous)

Command acceptance returns `202 Accepted`:

```json
{
  "operation_id": "op-01HZY4T8P...",
  "contract_id": "ctr-01HZY4T8Q...",
  "status": "offered",
  "resource": "/t/example-corp/user/1234",
  "command": "update-email",
  "refusal_reason": null,
  "expires_at": "2026-04-19T20:15:00Z"
}
```

The response **does not** indicate that the command has executed. It indicates that the command has been recorded as a contract offer. The client inspects the contract via the SDK's `ContractHandle` to determine whether the offer was accepted, refused, or is still under review.

#### 4.4.2 Synchronous Completion (Exception Only)

`200 OK` responses with inline domain state are permitted **only** for:

- Deterministic operations with bounded latency (typically under 100 ms).
- Operations that produce no side effects observable outside the contract.
- Operations explicitly declared synchronous in the OpenAPI specification.

Synchronous responses still carry a `contract_id`. Contract recording is not skipped. Synchronous responses are a latency optimization, not a governance exception.

#### 4.4.3 Refusal Response

When the executor refuses the contract synchronously, the response is `200 OK` with `status: "refused"` and a signed `refusal_reason` per specification 03. Refusal is not an HTTP error.

---

## 5. Query API (GraphQL)

### 5.1 Purpose

The Query API provides read-only visibility into system state, including:

- Current resource state.
- Operation and contract status.
- Historical outcomes.
- Aggregations across entities.

The Query API MUST NOT:

- Trigger commands.
- Cause side effects.
- Consume budget against user quotas for side-effect-inducing operations.
- Return partial data without a declared freshness bound.

### 5.2 Schema Rules

- Every field MUST be resolvable without side effects.
- Mutations MUST NOT be defined. GraphQL's `mutation` root is absent from the schema.
- Subscriptions are permitted for read-only streaming of state changes.
- Operation, contract, and audit entities are first-class query types.

Example query:

```graphql
query UserWithContracts($id: ID!) {
  user(id: $id) {
    email
    status
    contracts(first: 10) {
      edges {
        node {
          contract_id
          status
          offered_at
          decided_at
        }
      }
    }
  }
}
```

### 5.3 Freshness Bounds

Every query response includes a `freshness` envelope declaring how recent the returned data is guaranteed to be:

```json
{
  "data": { ... },
  "freshness": {
    "authoritative": false,
    "max_staleness_ms": 3000,
    "computed_at": "2026-04-19T19:42:11.120Z"
  }
}
```

Callers that require authoritative data (typically audit or compliance flows) set a query parameter to request `authoritative: true`; the Query API MAY incur additional latency to satisfy this.

### 5.4 Endpoint Structure

```
POST /t/{tenant}/api/graphql/v{version}
```

All queries for a tenant target a single endpoint. Schema and resolver versioning happen at the schema level, not through multiple endpoints.

---

## 6. Internal Service API (gRPC)

### 6.1 Purpose

The Internal Service API handles service-to-service communication inside the platform: command normalization, task dispatch, contract ledger writes, telemetry aggregation, and similar coordination work.

### 6.2 Protocol and Shape

- Protocol: gRPC over HTTP/2 with Protocol Buffers.
- Authentication: mutual TLS with workload identity (SPIFFE / X.509 SVIDs or equivalent).
- Authorization: fine-grained RBAC or ReBAC checked at every call.
- No user sessions are propagated. Internal calls carry the originating user's or agent's DID and the active contract reference, not session tokens.

### 6.3 Isolation from External Surfaces

The Internal Service API **MUST NOT** be exposed outside the platform's trust boundary. Specifically:

- The Internal Service API is never reachable from the public internet.
- No SDK issued to external consumers (UI, CLI, automation, agents) calls the Internal Service API.
- No MCP server exposes Internal Service API methods as agent tools.

Attempts to route external traffic to the Internal Service API are configuration defects and MUST be detected by CI validation.

---

## 7. Endpoint Service Responsibilities

The Endpoint Service is the **edge authority** for the Command API and the Query API. It is the first platform-internal service that receives client requests.

### 7.1 Endpoint Service MUST:

- Authenticate the caller (OIDC, workload identity, or agent credential).
- Validate the delegation envelope per specification 09.
- Enforce rate limits and quotas.
- Normalize and schema-validate the command or query.
- Generate a correlation identifier if absent.
- Forward the request to the Gateway with the authenticated principal, delegation chain, and correlation context attached.

### 7.2 Endpoint Service MUST NOT:

- Persist domain data.
- Execute tasks.
- Orchestrate workflows.
- Perform business logic.

The Endpoint Service is stateless (other than rate-limit counters) and horizontally scalable. It is the platform's publicly-routable surface; every other service is reachable only through it.

---

## 8. Gateway and Orchestration Service

### 8.1 Purpose

The Gateway is the **system of record for intent**. The Gateway receives validated commands from the Endpoint Service and coordinates the lifecycle of each operation.

### 8.2 Gateway MUST:

- Authorize commands against the tenant's policy.
- Enforce platform invariants.
- Offer the contract to the appropriate executor (specification 03).
- Record the contract in the ledger upon acceptance or refusal.
- Decompose accepted contracts into tasks and dispatch them.
- Aggregate execution results and produce the final attestation.

### 8.3 Gateway Communication

- Gateway ingress: gRPC from the Endpoint Service.
- Gateway egress to executors: gRPC with explicit contract references.
- Gateway egress to the contract ledger: direct or via a dedicated Contract Service.
- Gateway egress to the event plane: CloudEvents published with the `contract_id` attached.

---

## 9. Task Dispatch and Execution

### 9.1 Task Contracts

Tasks receive a **contract reference**, not raw input. When the Gateway dispatches a task, the task message contains:

- `operation_id` — the parent operation.
- `contract_id` — the accepted contract.
- `action_id` — the specific action within the contract.
- `target` — the target entity reference.
- `constraints` — tenant, environment, location, and other constraint fields inherited from the contract.
- `idempotency_key` — propagated from the original command.

Tasks MUST NOT be dispatched without a `contract_id`. A task without a contract reference is an architectural defect.

### 9.2 Execution Rules

Tasks:

- Are asynchronous unless explicitly declared synchronous.
- MUST be idempotent.
- MUST emit execution events to the Event Plane.
- MUST include the `contract_id` in every emitted event.
- MUST NOT call the Command API, Query API, or any external surface on behalf of the contract. Tasks use the Internal Service API.
- MUST NOT exceed the contract's declared `constraints`.

---

## 10. Eventing and Truth

### 10.1 Canonical Event Types

At minimum, every platform MUST emit:

- `operation.accepted` — the command was accepted and a contract offered.
- `contract.offered`
- `contract.accepted`
- `contract.refused`
- `task.started`
- `task.succeeded`
- `task.failed`
- `operation.completed`
- `contract.attested`
- `contract.terminated`

Each event carries its `operation_id`, `contract_id`, `tenant`, `environment`, `correlation_id`, and `causation_id` (the event that directly caused this one).

### 10.2 Event Plane Authority

Events are append-only and immutable. The event stream is the system of record for **what happened**. The contract ledger is the system of record for **what was agreed to**. Both MUST be consulted during audit; neither substitutes for the other.

Event Plane details are specified in document 08.

---

## 11. Failure Semantics

| Failure Point | Who Detects | Response |
|---------------|-------------|----------|
| Invalid command payload | Endpoint Service | Rejected at ingress with a `Validation Error` |
| Authentication failure | Endpoint Service | Rejected at ingress with a `System Error` (auth-class) |
| Unauthorized command | Gateway | Rejected after ingress with a `Validation Error` (scope-class) |
| Contract refused | Executor | Returned as structured refusal; not an HTTP error |
| Task execution failure | Executor | Recorded as `task.failed`; contract may succeed, partial-succeed, or fail |
| Ledger unavailable | Gateway | Rejected; no execution occurs |
| Partial failure | Gateway | Explicitly represented in the final attestation |

Task failures do **not** invalidate audit intent. The contract remains recorded even when execution fails; the final attestation records the failure.

---

## 12. Relationship to SDK (Specification 02)

The Command API and the Query API are consumed by SDKs, never by clients directly. SDKs:

- Hide REST URL construction and HTTP verb selection.
- Hide GraphQL query construction.
- Construct and attach delegation envelopes.
- Surface `ContractHandle` objects as primary return types from mutating calls.
- Surface freshness bounds from query responses.

A client that bypasses the SDK and calls the Command or Query API directly is non-conformant regardless of how the bypass is implemented. The API surfaces in this specification exist as the protocol implementation behind the SDK; they are not a public surface.

---

## 13. Relationship to Contract Model (Specification 03)

Every Command API call produces a contract offer per specification 03. Specifically:

- Command ingress is the **Offer** stage.
- Endpoint Service + Gateway authorization is the **Review** stage.
- Executor decision is the **Accept/Refuse** stage.
- Ledger write is the **Record** stage.
- Task dispatch is the **Execute** stage.
- Final attestation is the **Attest** stage.
- Contract expiry or completion is the **Terminate** stage.

The API surface and the contract lifecycle are in 1:1 correspondence. An API shape that cannot produce a valid contract envelope is a defective API shape.

---

## 14. Explicit Non-Goals

The API Architecture does not:

- Provide synchronous guarantees by default.
- Support CRUD-style direct mutation.
- Allow clients to bypass SDKs.
- Encode business logic in APIs.
- Offer per-client or per-agent bespoke endpoints outside the three surfaces above.

Additional surfaces, bespoke endpoints, or "admin" shortcuts are non-conformant. Add capability through SDK methods and through the existing three surfaces.

---

## 15. Conformance Requirements

A platform conforms to EPAS v1.4 API Architecture when:

1. Exactly three API surfaces exist: Command (REST), Query (GraphQL), and Internal Service (gRPC).
2. Command API URLs follow the canonical structure in Section 4.2.
3. Every mutating command returns `202 Accepted` (or `200 OK` for explicitly synchronous operations) with a `contract_id`.
4. Query API contains no mutations.
5. Query API responses carry freshness envelopes.
6. Internal Service API is never exposed outside the platform trust boundary.
7. Task dispatch carries a `contract_id` on every message.
8. Emitted events include `contract_id`, `operation_id`, `correlation_id`, and `causation_id`.
9. The API surfaces are consumed only through official SDKs.
10. CI validates that no client package calls the Command, Query, or Internal Service API directly.

Partial conformance during migration is permitted and MUST be declared in `conformance.yaml`.

---

## 16. Summary

EPAS v1.4 APIs exist to **declare intent, observe reality, and enforce governance**.

They do not execute work. They do not mutate state. They do not hide side effects.

Execution belongs to the task plane. Truth belongs to events. Consent belongs to contracts. Authority belongs to delegation.
