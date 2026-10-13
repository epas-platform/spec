# EPAS v2.1 — Reference Stacks

> **Status:** Draft — Guidance
> This specification defines recommended technology stacks for EPAS v2.1 profile families.

---

## 1. Purpose of EPAS v2.1 Reference Stacks

EPAS v2.1 is technology-neutral at the constitutional layer, but implementations still need practical guidance.

The EPAS v2.1 Reference Stacks document defines recommended defaults for profile families so that:

- prototypes can move quickly without architectural dead ends
- MVPs can harden without rewriting every core decision
- agent-centric systems use data stores and infrastructure that fit their relationship-heavy reality
- conventional enterprise systems continue to have a clear relational default path

Reference stacks are recommendations, not hard mandates, unless another v2.x specification explicitly promotes a recommendation to a requirement.

---

## 2. Reference Stack Selection Rule

An implementation MUST either:

1. adopt a reference stack family that matches its profile, or
2. declare and justify a different stack in its profile declaration

An implementation that diverges from the reference stack is not automatically non-conformant. An implementation that diverges without explanation is poorly specified and SHOULD be treated as incomplete.

---

## 3. Guidance Legend

Reference stack items use the following labels:

| Label | Meaning |
|-------|---------|
| `default` | Recommended path for the profile family |
| `permitted alternative` | Valid option without a special waiver |
| `requires justification` | Allowed only when rationale is recorded in the profile declaration |

---

## 4. Stack Family A - Entity-Native Agent-Centric Prototype and Demo

### 4.1 Stack Family ID

`entity_native_agentic_prototype`

### 4.2 Intended Profiles

- `data_model: entity_relationship_native`
- `delivery: sdk_first` or `agent_first`
- `audience: internal_tool` or `internal_platform`
- `lifecycle: prototype` or `demo`
- overlay `entity_native_agentic`

### 4.3 Recommended Stack

| Layer | Recommendation | Status |
|-------|----------------|--------|
| UI | Next.js + TypeScript | default |
| Backend | FastAPI + Python | default |
| Database | SurrealDB | default |
| SDKs | TypeScript SDK and Python SDK | default |
| Realtime | Server-Sent Events | default |
| Queueing | Database-backed jobs or lightweight queue | permitted alternative |
| Eventing | Database-backed event log | default |
| Infrastructure | Docker Compose | default |
| Object storage | Local filesystem or S3-compatible bucket | permitted alternative |

### 4.4 Rationale

This stack is optimized for systems where:

- entities and relationships are the primary modeling abstraction
- agent workflow state and memory matter from the first release
- multimodal or polymorphic records are normal
- a team wants to preserve EPAS architectural shape without implementing full production-grade control surfaces on day one

SurrealDB is the default reference database for this family because SurrealDB matches the entity-and-relationship-native profile directly.

---

## 5. Stack Family B - Entity-Native Agent-Centric MVP

### 5.1 Stack Family ID

`entity_native_agentic_mvp`

### 5.2 Intended Profiles

- `data_model: entity_relationship_native`
- `lifecycle: mvp`
- `governance: g1_guided` or `g2_operational`
- `audience: internal_platform` or `customer_facing`

### 5.3 Recommended Stack

| Layer | Recommendation | Status |
|-------|----------------|--------|
| UI | Next.js + TypeScript | default |
| Backend | FastAPI + Python | default |
| Database | SurrealDB | default |
| SDKs | TypeScript SDK and Python SDK | default |
| Realtime | Server-Sent Events or WebSockets | permitted alternative |
| Queueing | Redis-backed or NATS-backed jobs | permitted alternative |
| Eventing | NATS JetStream preferred; DB-backed event log permitted during transition | requires justification |
| Infrastructure | Simple runtime or platform runtime | default |
| Auth | OIDC or equivalent real auth | default |
| Object storage | S3-compatible object storage for media, artifacts, and large outputs | default |

### 5.4 Rationale

This stack preserves the SurrealDB-centered entity model while adding the stronger execution, auth, and event posture expected for an MVP.

---

## 6. Stack Family C - Conventional Enterprise Internal Platform

### 6.1 Stack Family ID

`relational_internal_platform`

### 6.2 Intended Profiles

- `data_model: relational_operational`
- `audience: internal_platform`
- `delivery: sdk_first`
- `lifecycle: demo`, `mvp`, or `production`

### 6.3 Recommended Stack

| Layer | Recommendation | Status |
|-------|----------------|--------|
| UI | Next.js + TypeScript | default |
| Backend | FastAPI + Python or equivalent typed service stack | default |
| Database | PostgreSQL | default |
| SDKs | TypeScript SDK and Python SDK | default |
| Realtime | Server-Sent Events | default |
| Queueing | Redis or NATS | permitted alternative |
| Eventing | NATS JetStream preferred | permitted alternative |
| Infrastructure | Platform runtime | default |
| Object storage | S3-compatible object storage where durable artifacts are stored | default |

### 6.4 Rationale

This stack fits systems where:

- state is primarily tabular and operational
- teams benefit from the relational maturity, reporting ecosystem, and widespread operational familiarity of PostgreSQL
- entity-and-relationship-native modeling is not the dominant need

PostgreSQL remains the default reference database for this family.

---

## 7. Stack Family D - Customer-Facing Product

### 7.1 Stack Family ID

`customer_product_standard`

### 7.2 Intended Profiles

- `audience: customer_facing`
- `lifecycle: mvp` or `production`

### 7.3 Recommended Stack

| Layer | Recommendation | Status |
|-------|----------------|--------|
| UI | Next.js + TypeScript | default |
| Backend | FastAPI + Python or service-oriented equivalent | default |
| Database | PostgreSQL for conventional systems, SurrealDB for agent-centric entity-native systems | requires justification |
| SDKs | TypeScript SDK mandatory; Python SDK strongly recommended | default |
| Realtime | SSE or WebSockets based on UX need | permitted alternative |
| Eventing | NATS JetStream preferred | default |
| Infrastructure | Platform runtime or enterprise multi-env runtime | default |
| Auth | Full OIDC or equivalent production auth | default |
| Object storage | Managed S3-compatible storage for customer-visible artifacts and uploads | default |

### 7.4 Rationale

Customer-facing systems must harden earlier. EPAS v2.1 therefore recommends stronger auth, observability, deployment, and eventing defaults for this family.

---

## 8. Stack Family E - Regulated Production

### 8.1 Stack Family ID

`regulated_production_assured`

### 8.2 Intended Profiles

- overlay `regulated`
- `lifecycle: production`
- `governance: g3_assured`

### 8.3 Recommended Stack

| Layer | Recommendation | Status |
|-------|----------------|--------|
| UI | Next.js + TypeScript or equivalent enterprise-grade web client | default |
| Backend | Strongly typed service platform with SDK-first boundary | default |
| Database | PostgreSQL or SurrealDB depending data model, with explicit evidence and retention justification | requires justification |
| SDKs | TypeScript SDK and Python SDK | default |
| Eventing | NATS JetStream or justified equivalent | default |
| Infrastructure | Enterprise multi-env runtime | default |
| Auth | Full identity, delegation, and approval-aware runtime | default |
| Object storage | Managed object storage with explicit retention and evidence controls | default |

### 8.4 Rationale

Regulated production is defined more by governance and evidence posture than by a single stack. EPAS v2.1 therefore emphasizes declared justification and control completion rather than one universal database or framework rule.

---

## 9. Database Selection Guidance

### 9.1 SurrealDB Default Cases

SurrealDB is the default reference database when:

- the profile is `entity_relationship_native`
- the overlay is `entity_native_agentic`
- the implementation is agent-centric
- everything important is modeled as entities with relationships
- graph traversal and relationship reasoning are primary operations
- multimodal or polymorphic data is native to the domain

### 9.2 PostgreSQL Default Cases

PostgreSQL is the default reference database when:

- the profile is `relational_operational`
- the application is primarily conventional enterprise workflow software
- tabular reporting, relational joins, and mature enterprise operational tooling dominate

### 9.3 When Not To Use SurrealDB

SurrealDB is usually not the best default when:

- the system is primarily a classic tabular line-of-business application
- mature SQL reporting and enterprise BI integration are the dominant requirements
- the implementation team needs the broadest possible conventional SQL ecosystem as the primary constraint

### 9.4 Hybrid Cases

Hybrid architectures are valid where the profile explicitly declares `hybrid_multimodal`.

Examples:

- SurrealDB for agent memory and workflow graph state
- PostgreSQL for transactional business records
- object storage for large artifacts

Hybrid systems MUST be explicit about which state belongs where.

---

## 10. Frontend Guidance

Next.js plus TypeScript is the default reference UI stack across most EPAS v2.1 profile families because:

- it supports serious product-grade UX
- it aligns with SDK-first web consumption
- it fits both internal and customer-facing products
- it avoids prototype-only UI dead ends

Streamlit remains appropriate for exploratory tooling outside the default product path, but Streamlit is not the primary reference UI for repo-ready EPAS products.

---

## 11. Backend Guidance

FastAPI plus Python is the default reference backend for many EPAS profile families because:

- agent orchestration, AI workflows, and SDK integration are often Python-heavy
- it enables rapid iteration without abandoning typed schemas and serious API discipline
- it aligns well with internal automation and agent clients

EPAS v2.1 does not prohibit other backends. EPAS v2.1 recommends FastAPI as the practical default in the absence of a stronger profile-specific reason.

---

## 12. Event, Queue, and Artifact Guidance

### 12.1 Prototype and Demo

- database-backed jobs are acceptable
- database-backed event logs are acceptable
- NATS JetStream is optional

### 12.2 MVP and Production

- NATS JetStream becomes the preferred event backbone
- retries, DLQ behavior, and event discipline should match v2.0 event-plane expectations

### 12.3 Object Storage Guidance

Object storage should be used whenever the system produces large artifacts, media, exports, model outputs, or evidence bundles.

Object storage may be local in prototype environments and managed in production environments.

### 12.4 Local Development Parity

Prototype stacks SHOULD preserve the shape of the later MVP stack where practical.

At minimum, the prototype environment SHOULD keep:

- the same primary UI framework
- the same primary backend framework
- the same primary database family unless the prototype is explicitly a throwaway spike
- the same SDK boundary pattern

This staged guidance lets teams grow into the full event plane without rebuilding the whole product architecture.

