# EPAS v2.1 — Profile Matrix

> **Status:** Draft — Normative
> This specification defines the composable EPAS profile matrix that every EPAS implementation MUST use when declaring its profile.

---

## 1. Purpose of the EPAS v2.1 Profile Matrix

The EPAS v2.1 Profile Matrix defines the legal dimensions along which an EPAS implementation is classified.

The EPAS v2.1 Profile Matrix exists to solve a recurring problem:

- a prototype should not be forced to implement every production control immediately
- a production system should not inherit prototype shortcuts by accident
- a customer-facing system should not be judged by the same maturity baseline as an internal exploratory tool
- an agent-centric entity-and-relationship-native platform should not be forced into the same data-model assumptions as a conventional enterprise CRUD application
- a system with event-driven behavior should not be forced to treat that as the same thing as its primary architectural boundary

The EPAS v2.1 Profile Matrix provides one DRY mechanism for expressing these differences.

---

## 2. Profile Matrix Rule

Every EPAS implementation MUST declare exactly one value from each mandatory axis.

Implementations MAY additionally declare optional overlays and architecture traits where EPAS defines them.

Implementations MUST NOT invent private axis names or private axis values in their canonical conformance declaration. Private internal metadata MAY exist outside the EPAS declaration, but it is not part of EPAS conformance.

---

## 3. Profile Declaration Lifecycle

The canonical profile declaration is a document with its own lifecycle.

### 3.1 Allowed Declaration States

| State | Meaning |
|-------|---------|
| `draft` | The profile declaration is being prepared or reviewed |
| `active` | The profile declaration is currently authoritative for the implementation |
| `deprecated` | The profile declaration remains readable but should not be used for new work |
| `superseded` | The profile declaration has been replaced by a newer declaration |

### 3.2 Lifecycle Rules

- A declaration MAY move from `draft` to `active` when it becomes the authoritative profile record.
- A declaration MAY move from `active` to `deprecated` or `superseded` when a newer declaration replaces it.
- Historical implementation releases SHOULD retain the declaration that was true for that release.
- A current implementation SHOULD point to the active declaration, not the historical one.

---

## 4. Mandatory Axes

EPAS v2.1 defines the following mandatory profile axes:

| Axis | Purpose |
|------|---------|
| `lifecycle` | Declares the maturity stage of the implementation |
| `audience` | Declares who the implementation primarily serves |
| `environment` | Declares the deployment environment for the declaration |
| `governance` | Declares the strength of agentic governance and trust controls |
| `infrastructure` | Declares the infrastructure-operating posture |
| `data_model` | Declares the dominant data-model shape |
| `delivery` | Declares the primary consumption model |
| `architecture` | Declares the system-boundary pattern |

---

## 5. Lifecycle Axis

The `lifecycle` axis declares implementation maturity.

### 4.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `prototype` | Exploratory implementation proving workflow, architecture, or user value |
| `demo` | Polished demonstration implementation proving coherent operator or user experience |
| `mvp` | Real-use implementation with durable data and supported user paths |
| `production` | Hardened implementation intended for sustained operational use |

### 4.2 Lifecycle Rules

- `prototype` MAY use simulated contract handling and stubbed authentication.
- `prototype` MUST still preserve SDK-first boundaries and refusal semantics.
- `demo` MUST present real lifecycle states to the user, even if some controls remain lightweight.
- `mvp` MUST implement real authentication and durable execution records.
- `production` MUST implement full profile-required controls and profile-aware conformance evidence.

### 4.3 Lifecycle and Governance Compatibility

| Lifecycle | Minimum Governance | Notes |
|-----------|--------------------|-------|
| `prototype` | `g0_exploratory` | Stubbed auth allowed outside production |
| `demo` | `g0_exploratory` or `g1_guided` | Visible lifecycle states required |
| `mvp` | `g1_guided` | Real auth required |
| `production` | `g2_operational` | `g3_assured` for regulated systems |

---

## 6. Audience Axis

The `audience` axis declares who the implementation primarily serves.

### 5.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `internal_tool` | Built primarily for operators or internal builders |
| `internal_platform` | Shared internal platform used by multiple teams or services |
| `customer_facing` | Product surface directly used by customers |
| `partner_embedded` | Consumed primarily through partner or embedded integrations |

### 5.2 Audience Rules

- `internal_tool` profiles may optimize for speed of iteration.
- `internal_platform` profiles must prioritize consistency, SDK discipline, and shared-operability concerns.
- `customer_facing` profiles must prioritize real auth, durable state, and user-facing operational clarity earlier in the lifecycle.
- `partner_embedded` profiles must prioritize SDK quality, versioning discipline, and integration ergonomics.

---

## 7. Environment Axis

The `environment` axis declares the environment context for the profile declaration.

### 6.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `local_dev` | Local workstation or isolated developer runtime |
| `preview` | Ephemeral preview or branch environment |
| `shared_dev` | Shared non-production team environment |
| `staging_uat` | Pre-production validation and acceptance environment |
| `production` | Production runtime serving real operations |

### 6.2 Environment Rules

- Environment declarations are infrastructure-level, not merely configuration labels.
- `production` MAY NOT rely on stubbed auth or simulated contracts.
- `local_dev` MAY use synthetic identities and local-only secrets patterns.
- `staging_uat` SHOULD resemble production in auth, deployment, and data-flow behavior as closely as practical.

---

## 8. Governance Axis

The `governance` axis declares the maturity of contract, identity, and agentic governance controls.

### 7.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `g0_exploratory` | Lightweight exploratory governance |
| `g1_guided` | Guided governance with visible policy and approval semantics |
| `g2_operational` | Real operational governance with durable approvals and auth |
| `g3_assured` | High-assurance governance with strong delegation and contract rigor |

### 7.2 Governance Rules

#### `g0_exploratory`

- operation classification required
- prohibited operations required
- simulated or lightweight contract handling allowed
- cryptographic signatures not required
- stubbed identity allowed outside production

#### `g1_guided`

- operation classification required
- approval-required flows required
- durable audit trail required
- basic or real auth required depending on environment

#### `g2_operational`

- real auth required
- durable contract lifecycle required
- runtime governance enforcement required
- approval flow evidence required

#### `g3_assured`

- full identity and delegation rigor required
- per-request signatures required
- full contract-plane strength required
- strongest audit and evidence posture required

---

## 9. Infrastructure Axis

The `infrastructure` axis declares the operating posture of the system.

### 8.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `i0_local_compose` | Local or isolated Docker Compose style runtime |
| `i1_simple_runtime` | Simple VM or container runtime with managed backups and reverse proxy |
| `i2_platform_runtime` | Structured platform runtime such as Kubernetes or equivalent |
| `i3_enterprise_multi_env` | Fully segmented enterprise multi-environment operating model |

### 8.2 Infrastructure Rules

- `i0_local_compose` is appropriate for prototypes and many demos.
- `i1_simple_runtime` is appropriate for polished demos and early MVPs.
- `i2_platform_runtime` is appropriate when runtime decomposition, observability, and shared operations matter.
- `i3_enterprise_multi_env` is appropriate for regulated and large-scale production systems.

---

## 10. Data Model Axis

The `data_model` axis declares the dominant state-modeling style of the platform.

### 9.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `relational_operational` | Conventional tabular enterprise application model |
| `entity_relationship_native` | Entity-and-relationship-native model for graph-shaped and agent-centric systems |
| `hybrid_multimodal` | Mixed state model spanning relational, entity, relationship, and multimodal structures |

### 9.2 Data Model Rules

- `relational_operational` profiles default to PostgreSQL in reference stacks unless another database is justified.
- `entity_relationship_native` profiles default to SurrealDB in reference stacks for entity-native and agent-centric systems.
- `hybrid_multimodal` profiles may combine SurrealDB with relational or object storage where justified.

### 9.3 Reference Defaults and Alternatives

Defaults do not prohibit other technology choices.

- A profile MAY choose an alternative database, but the profile declaration MUST justify the deviation.
- An entity-native agentic profile SHOULD use SurrealDB unless the profile declaration states why a different engine is superior.
- A relational-operational profile SHOULD use PostgreSQL unless the profile declaration states why a different engine is superior.

### 9.4 SurrealDB Position

EPAS v2.1 explicitly recognizes SurrealDB as the default reference database for `entity_relationship_native` profiles where:

- everything important is modeled as entities and relationships
- agent memory and agent workflow state are central concerns
- multimodal and polymorphic records are routine
- relationship traversal is a primary system operation

Important state is the state used for authorization, audit, workflow decisions, agent memory, customer-visible behavior, or durable business records.

This position is normative for EPAS reference-stack guidance. It does not prohibit other databases, but it sets the default reference path.

---

## 11. Delivery Axis

The `delivery` axis declares the primary product-consumption model.

### 10.1 Allowed Values

| Value | Meaning |
|-------|---------|
| `ui_first` | Primary experience is a human-operated user interface |
| `api_first` | Primary experience is service integration |
| `sdk_first` | Official SDKs are the product boundary |
| `agent_first` | AI agents are the dominant operational consumer |
| `integration_first` | Workflow and external-system integration is the dominant mode |

### 10.2 Delivery Rules

- All EPAS v2.1 implementations MUST still satisfy the v2.0 SDK-first principle.
- `ui_first` MAY describe product emphasis, but MUST NOT justify direct UI-to-platform mutation paths that bypass SDKs.
- `sdk_first` is the recommended default for most EPAS systems.
- `agent_first` SHOULD be paired with `entity_relationship_native` or `hybrid_multimodal` data profiles unless justified otherwise.

---

## 12. Architecture Axis

The `architecture` axis declares the primary implementation boundary pattern.

### 11.1 Primary Architecture Values

| Value | Meaning |
|-------|---------|
| `single_service` | A single deployable service |
| `modular_monolith` | One deployable unit with strong internal module boundaries |
| `service_oriented` | Multiple services with explicit service boundaries |

### 11.2 Architecture Traits

Architecture traits are optional refinements that describe coordination or operating characteristics without replacing the primary architecture value.

Allowed traits:

- `event_driven`
- `edge_capable`

Architecture traits MUST be treated as additive, not as replacements for the primary architecture value.

### 11.3 Architecture Rules

- `single_service` and `modular_monolith` are both valid EPAS starts.
- `event_driven` does not require many services; it requires that asynchronous event semantics are architecturally real.
- `edge_capable` requires compliance with edge-specific expectations in EPAS v2.0 specification 07.
- `architecture_traits` MAY be used to record secondary architecture characteristics when the primary architecture value does not fully capture the deployment shape.
- If `architecture_traits` contains `event_driven`, the implementation SHOULD preserve an explicit event plane in the profile and implementation artifacts.
- If `architecture_traits` contains `edge_capable`, the implementation SHOULD also declare the `edge` overlay or explain the edge-related exception.

---

## 13. Optional Overlays

EPAS v2.1 defines optional overlays for recurring conditions that cut across the mandatory axes.

### 12.1 Allowed Overlays

- `regulated`
- `edge`
- `entity_native_agentic`
- `customer_data_sensitive`

### 12.2 Overlay Rules

- Overlays refine requirements. Overlays do not replace mandatory axis values.
- `entity_native_agentic` SHOULD usually imply `entity_relationship_native` unless an exception is documented.
- `regulated` increases evidence, retention, approval, and audit expectations.
- If `edge_required: true`, the implementation SHOULD declare the `edge` overlay.
- If `edge` is declared, the implementation SHOULD set `edge_required: true`.
- If `sensitivity: regulated`, the implementation SHOULD declare the `regulated` overlay.
- If `customer_data_sensitive` is declared, the implementation SHOULD set sensitivity to `customer_confidential` or `regulated`.
- If `entity_native_agentic` is declared, the implementation SHOULD also declare `entity_relationship_native` unless the profile explicitly documents why it is not the dominant data model.

---

## 14. Named Bundles

Named bundles are reusable compositions of the matrix. Named bundles are derived artifacts, not the primary model.

Example:

```yaml
bundle_id: entity_native_internal_prototype
composed_from:
  lifecycle: prototype
  audience: internal_platform
  environment: shared_dev
  governance: g0_exploratory
  infrastructure: i0_local_compose
  data_model: entity_relationship_native
  delivery: sdk_first
  architecture: modular_monolith
  architecture_traits:
    - event_driven
  overlays:
    - entity_native_agentic
```

Bundles are permitted for reuse. Bundles MUST remain reducible to the canonical matrix.

The initial draft bundle catalog is defined in [05-profile-bundles.md](./05-profile-bundles.md) and [profile-catalog.draft.json](./profile-catalog.draft.json).

---

## 15. Illegal Combinations

Some combinations are non-conformant.

Examples:

- `environment: production` plus `authentication: stubbed`
- `governance: g3_assured` plus simulated contract handling
- `delivery: sdk_first` plus direct client API mutation paths
- `entity_native_agentic` plus a declaration that relationships are not first-class state
- `environment: production` plus `governance: g0_exploratory`
- `audience: customer_facing` plus `lifecycle: production` plus `authentication.required: false`
- `overlay: regulated` plus `governance: g0_exploratory`
- `agent_first` plus no declared identity and delegation posture beyond prototype or demo
- `architecture_traits: edge_capable` plus `edge_required: false` without an exception

The schema layer and CI layer SHOULD validate known-illegal combinations mechanically.

---

## 16. Conformance Requirements

A valid EPAS v2.1 profile declaration MUST:

1. include every mandatory axis
2. use only defined axis values
3. include any declared overlays from the approved overlay set
4. remain consistent with the implementation’s control and stack declarations
5. be validated against the EPAS v2.1 profile schema

An implementation that fails any of these rules is non-conformant.
