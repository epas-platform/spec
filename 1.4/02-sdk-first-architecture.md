# EPAS v1.4 — SDK-First Architecture

> **Status:** Draft — Normative
> This specification defines the SDK-first architecture for EPAS v1.4 platforms. It specifies how humans, automation, agents, and third-party systems interact with an EPAS platform without bypassing governance, contracts, or cryptographic authority.

---

## 1. Purpose of SDK-First Architecture

EPAS v1.4 SDK-First Architecture ensures that:

- There is exactly **one correct way** to interact with an EPAS platform.
- Every client expresses **intent**, not transport implementation.
- Contracts, refusals, and audit evidence are first-class objects in every language binding.
- User interfaces, command-line tools, automation scripts, and AI agents remain behaviorally consistent.

This specification is **normative**. Implementations that deviate from the rules in this document are non-conformant.

---

## 2. Foundational Principle

> **SDKs are the product. APIs are an implementation detail.**

Any interaction with an EPAS platform that cannot be performed through an official SDK is not a supported operation. Operations that require direct API consumption are architectural defects and MUST be resolved by extending the SDK, not by permitting the direct call.

This principle is non-negotiable. Exceptions require explicit amendment to this specification.

---

## 3. SDK Client Taxonomy

EPAS v1.4 recognizes four client classes. All four are equal under the SDK model. No client class is granted privileges that another lacks.

| Client Class | Examples | SDK Required |
|--------------|----------|--------------|
| Web UI | Browser-delivered single-page applications | Yes (TypeScript SDK) |
| Command-Line Interface | Terminal tools and operator consoles | Yes (Language-native SDK) |
| Automation | CI/CD jobs, scheduled workflows, integration scripts | Yes (Python or equivalent SDK) |
| AI Agent | Supervisor agents, worker agents, tool-use agents | Yes (Python or TypeScript SDK) |

### 3.1 Client Class Invariants

- **No client class calls raw platform APIs directly.** Every client class uses the SDK.
- **No client class possesses a private shortcut.** If a capability exists in the SDK, it exists for every client class that imports that SDK.
- **Agents are ordinary clients.** AI agents receive no special authority, no special protocol, and no special error semantics. Agents use the same SDK as humans.

---

## 4. SDK Responsibilities

Every EPAS v1.4 SDK **MUST**:

1. Encapsulate the REST Command API so clients never construct HTTP requests directly.
2. Encapsulate the GraphQL Query API so clients never construct GraphQL documents directly.
3. Construct and attach authenticated delegation envelopes on every mutating call.
4. Manage correlation identifiers, idempotency keys, and retry policy internally.
5. Surface contracts (see specification 03) as first-class return types on mutating calls.
6. Represent refusal as a structured, inspectable outcome rather than an exception.
7. Provide deterministic error semantics classified into the taxonomy defined in Section 10.
8. Emit structured audit events for every call made through the SDK.

Every EPAS v1.4 SDK **MUST NOT**:

- Expose raw HTTP endpoints, GraphQL strings, or transport-layer primitives to the caller.
- Permit arbitrary parameter mutation after a call has been constructed.
- Permit bypass of contract enforcement for any reason.
- Silently retry refused operations.
- Cache authority decisions beyond the contract's declared lifetime.

---

## 5. Domain-Oriented SDK Design

SDKs expose **domain-level methods**, not transport-level calls. The caller works in the platform's domain vocabulary, never in REST paths or GraphQL field names.

### 5.1 Required Method Shape

A mutating SDK method has the following required shape, expressed here in pseudocode:

```
handle = client.<resource>.<action>(<domain_parameters>)
```

Where:

- `<resource>` is a domain noun (`users`, `agents`, `contracts`, `tools`).
- `<action>` is a domain verb (`update_email`, `offer_contract`, `dispatch_task`).
- `<domain_parameters>` are validated, typed parameters in the platform's schema.
- `handle` is a `ContractHandle` object (see Section 6).

### 5.2 Prohibited Method Shapes

The following method shapes are non-conformant:

- `client.http.post("/users/1234/email", {...})` — exposes transport.
- `client.graphql("query { ... }")` — exposes query language.
- `client.raw_request(method, path, body)` — exposes bypass.

Callers MUST NOT see REST paths, HTTP verbs, GraphQL queries, or internal retry state.

---

## 6. ContractHandle — The Primary Return Type

Every mutating SDK method returns a `ContractHandle`. A `ContractHandle` represents the contract issued for the requested operation and its lifecycle (see specification 03).

### 6.1 Required Fields

A `ContractHandle` MUST expose:

| Field | Type | Purpose |
|-------|------|---------|
| `contract_id` | string | Stable identifier for the contract |
| `operation_id` | string | Stable identifier for the operation the contract authorizes |
| `status` | enum | One of: `offered`, `accepted`, `refused`, `in_progress`, `completed`, `failed`, `expired` |
| `refusal_reason` | structured or null | Present when `status = refused`; signed reason |
| `expires_at` | timestamp | When the contract ceases to be valid |
| `events_url` | URI or null | Stream endpoint for execution events, when provided |

### 6.2 Required Methods

A `ContractHandle` MUST provide:

| Method | Behavior |
|--------|----------|
| `wait()` | Blocks until the contract reaches a terminal state (`completed`, `failed`, `refused`, `expired`) |
| `status()` | Returns the current state without blocking |
| `refused()` | Returns true when the contract is in the `refused` terminal state |
| `cancel()` | Requests contract termination when the contract is revocable |
| `events()` | Returns a stream of execution events when supported; optional |

### 6.3 Refusal Is Not an Exception

SDKs MUST NOT raise exceptions for refusal. Refusal is a valid, expected outcome of any mutating call. Callers inspect the `ContractHandle` to determine whether the operation was accepted, refused, or failed.

```python
# Conformant: refusal is inspected, not thrown
handle = client.users.update_email(user_id="1234", new_email="new@example.com")
if handle.refused():
    log_refusal(handle.refusal_reason)
else:
    handle.wait()

# Non-conformant: refusal raised as exception
try:
    client.users.update_email(user_id="1234", new_email="new@example.com")
except RefusedError as e:  # ← wrong
    ...
```

---

## 7. Query Model

SDKs expose read-only access to platform state through domain methods that return typed values.

### 7.1 Read Method Shape

```
value = client.<resource>.<query>(<domain_parameters>)
```

Read methods do not return `ContractHandle` objects. Reads never cause side effects. Reads never produce contract ledger entries.

### 7.2 Read Method Constraints

- Read methods MUST NOT mutate platform state.
- Read methods MUST NOT trigger commands, workflows, or tool executions.
- Read methods MAY return eventually-consistent data with a declared freshness bound.
- Read methods MUST declare their freshness bound in SDK documentation (`<= 5s`, `authoritative`, etc.).

---

## 8. SDK Authentication and Identity

SDKs handle authentication internally. Callers provide an identity reference at client construction time and do not manage tokens manually.

### 8.1 Identity Reference Types

| Identity Type | Used By | Construction |
|---------------|---------|--------------|
| OIDC session | Humans (UI, CLI) | Interactive login or session file |
| Workload identity | Services, CI jobs | Mounted SPIFFE SVID, IAM role, or equivalent |
| Agent identity | AI agents | Registered agent credential per specification 09 |

### 8.2 Delegation Envelopes

Every mutating SDK call MUST construct and attach a signed delegation envelope that declares:

- Requester identity (DID or equivalent cryptographic handle)
- Delegation chain from the requester to the invoking principal
- Requested action and resource scope
- Timestamp and nonce

Delegation envelopes are defined in specification 09. SDKs construct envelopes; callers do not.

### 8.3 Token Handling

SDKs obtain transport credentials (bearer tokens, cookies, mTLS certificates) as needed and discard them after identity extraction. Authority in EPAS v1.4 is expressed through signed delegation, not through bearer tokens.

---

## 9. CLI Architecture

A command-line interface is a **thin wrapper** over a language-native SDK. The CLI contains no business logic, no retry logic, and no error handling beyond presentation.

### 9.1 CLI Structure

```
CLI command handler  →  SDK method  →  ContractHandle  →  Output formatter
```

### 9.2 CLI Invariants

- The CLI MUST NOT call platform APIs directly. The CLI calls the SDK.
- The CLI MUST reflect SDK refusal semantics verbatim. A CLI that swallows refusal is non-conformant.
- The CLI MUST expose `--output json` or equivalent machine-readable output so that CLI calls are scriptable without parsing prose.

### 9.3 Example

A conformant CLI invocation:

```
platform user update-email --id 1234 --email new@example.com --output json
```

Produces JSON that includes the `contract_id`, `status`, and `refusal_reason` exactly as returned by the SDK.

---

## 10. UI Architecture

Web and mobile user interfaces import a TypeScript or Dart SDK. The UI never calls platform APIs directly.

### 10.1 UI Invariants

- The UI imports the TypeScript SDK (for web) or a platform-appropriate SDK (for mobile).
- The UI treats `ContractHandle` as first-class UI state. Pending, refused, and completed contracts have distinct UI affordances.
- Optimistic UI patterns are permitted but MUST roll back when the underlying contract is refused.
- The UI MUST NOT retry refused operations without explicit user action and a new contract.

### 10.2 UI State Model

UI state machines MUST include at minimum these states for any mutating action:

- `idle` — no pending operation
- `offering` — contract is being offered
- `accepted` — contract accepted; execution in progress
- `refused` — contract refused; refusal reason displayed
- `completed` — operation succeeded
- `failed` — contract accepted but execution failed
- `expired` — contract expired without execution

---

## 11. AI Agent Consumption Model

AI agents are ordinary SDK consumers. An agent that bypasses the SDK is non-conformant regardless of how that bypass is implemented (direct HTTP, shell-out to a CLI that bypasses the SDK, MCP server that bypasses the SDK, etc.).

### 11.1 Agent Invariants

- Agents use the same SDK as humans and automation.
- Agents receive `ContractHandle` objects from mutating calls and MUST inspect them.
- Agents MUST NOT retry refused operations without explicit re-authorization through a new contract.
- Supervisor agents operate exclusively on `ContractHandle` objects when coordinating worker agents; they do not construct work items in any other form.

### 11.2 MCP Integration

When platform capabilities are exposed to agents through the Model Context Protocol (MCP), the MCP server implementation MUST itself be an SDK consumer. The MCP server translates tool invocations into SDK calls and returns structured results. MCP servers that bypass the SDK are non-conformant.

---

## 12. SDK Error Model

SDKs MUST classify errors into exactly four categories:

| Category | Meaning | Client Action |
|----------|---------|---------------|
| **Validation Error** | Client-constructed request is invalid | Fix and retry |
| **Refusal** | Contract was not accepted by an executor | Do not retry; obtain new contract if retry is intended |
| **Execution Failure** | Contract was accepted but execution failed | Retry permitted within contract validity; new contract otherwise |
| **System Error** | Platform is unavailable or degraded | Retry with backoff permitted |

Transport-layer errors (timeouts, TLS failures, DNS resolution failures) are mapped to **System Error** by the SDK. Callers do not see raw transport errors.

---

## 13. SDK Versioning

### 13.1 Version Independence

- SDK versions are independent of platform API versions.
- The platform MAY evolve internal API versions freely provided SDK contracts remain stable.
- SDK major version bumps indicate breaking changes to SDK consumers.

### 13.2 Version Policy

- SDKs MUST follow semantic versioning.
- SDKs MUST maintain backward compatibility within a major version.
- Deprecated methods MUST emit a warning for at least one minor version before removal.
- Deprecated methods MUST remain functional for at least two minor versions after deprecation.

### 13.3 Version Discovery

Every SDK MUST expose a `client.version` property that returns the SDK version and a `client.platform_version()` method that returns the negotiated platform version. Version negotiation happens on client construction.

---

## 14. SDK Package Structure

Every service in an EPAS v1.4 platform MUST publish the following packages:

| Package | Purpose | Consumers |
|---------|---------|-----------|
| `{service}-api` | Core REST / GraphQL service implementation | SDK only |
| `{service}-sdk-{language}` | Language-native client library | CLI, UI, automation, agents |
| `{service}-mcp` | MCP server exposing SDK methods as agent tools | AI agents |
| `{service}-cli` | Command-line interface | Humans and scripts |

### 14.1 Repository Layout

```
service/
├── llms.txt                       # Agent discovery index (REQUIRED)
├── CLAUDE.md                      # Agent context file (REQUIRED)
├── README.md                      # Human overview
├── conformance.yaml               # Declared EPAS v1.4 conformance state
│
├── packages/
│   ├── api/                       # Core service
│   ├── sdk/                       # Language-native SDKs (subdirectories per language)
│   ├── mcp/                       # MCP server
│   ├── cli/                       # CLI
│   └── web/                       # UI (where applicable)
│
├── docs/
│   ├── architecture/              # C4 diagrams, ADRs
│   ├── api/                       # OpenAPI / GraphQL schema
│   └── mcp-tools.md               # MCP tool documentation
│
└── .ci/
    └── conformance-check.yaml     # CI validation of SDK-first compliance
```

---

## 15. SDK Generation Rules

SDKs MAY be hand-written or generated from OpenAPI specifications. Generated SDKs MUST be post-processed to satisfy the domain-oriented method shape requirements in Section 5.

A raw generated SDK that exposes transport paths as method names is non-conformant. Generation is a starting point, not a finish line.

---

## 16. SDK-First Conformance Requirements

A platform conforms to EPAS v1.4 SDK-First Architecture when:

1. Every client class in Section 3 consumes the platform through an official SDK.
2. No client class calls platform APIs directly.
3. Every mutating SDK call returns a `ContractHandle` satisfying Section 6.
4. Refusal is represented as a structured outcome, never as an exception.
5. CLI, UI, MCP server, and automation consume the SDK without privileged shortcuts.
6. Every service publishes the packages listed in Section 14.
7. CI validates SDK-first compliance on every pull request.

Partial conformance is permitted during migration and MUST be declared in `conformance.yaml`.

---

## 17. SDK-First Non-Goals

This specification does not:

- Optimize for minimal network latency or message size.
- Specify a language, framework, or runtime for SDKs.
- Expose raw protocol surfaces for performance reasons.
- Support partial or opportunistic implementations that bypass any of the invariants above.

Correctness, consistency, and auditability take precedence over performance in SDK design. Latency concerns are addressed in the service layer, not by weakening the SDK contract.
