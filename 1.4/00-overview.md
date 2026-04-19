# EPAS v1.4 Overview

> **Status:** Draft
> **Supersedes:** EPAS v1.3.0 (November 2025)
> **Publication format:** Document set + monolithic scaffold

---

## 1. Purpose of EPAS v1.4

EPAS v1.4 defines the architectural specification for enterprise multi-platform systems whose **primary consumers are autonomous AI agents**, and whose **primary safety property is provable consent**.

v1.4 is not an incremental revision. It is a constitutional shift.

Where EPAS v1.3 specified how to build a multi-tenant enterprise platform with AI components, v1.4 specifies how to build a platform whose operational reality is shaped by AI agents acting under explicit, auditable authority. Every platform surface — API, SDK, event, storage, human interface — is evaluated against two load-bearing questions:

1. **Can an AI agent consume this surface correctly without privileged access?**
2. **Can every action taken through this surface be proven to have been consented to?**

If either answer is no, the surface is non-conformant with v1.4.

---

## 2. Strategic Shifts from v1.3

| # | Shift | Impact |
|---|-------|--------|
| 1 | **AI-First Documentation** | Documentation is authored for AI agents as primary consumers; human readability is preserved but not prioritized over agent comprehension. |
| 2 | **SDK-First Architecture** | SDKs become the product. All clients (UI, CLI, automation, agents) consume APIs exclusively through SDKs. |
| 3 | **Contract-Based Trust Model** | No work is performed without an explicit, signed, ledger-recorded contract. Refusal is a first-class, auditable outcome. The contract ledger is externally anchored (Merkle commitments to an operator-independent public medium) so integrity is verifiable without platform cooperation. |
| 4 | **Machine-Readable Standards** | `llms.txt`, agent context files, and RAG-optimized markdown are mandatory in every repository. |
| 5 | **Agentic Governance** | Guardrails-as-code define what agents may do autonomously, what requires approval, and what is prohibited. |
| 6 | **Edge-Capable Deployment** | A formally specified Edge Node deployment class supports constrained hardware and intermittent connectivity. |
| 7 | **NATS JetStream as Primary Event Backbone** | NATS JetStream replaces Kafka as the recommended primary event bus. Kafka/Redpanda remains permitted for compliance-driven retention scenarios. |
| 8 | **Five-Plane Architecture** | The platform adds a **Contract Plane** alongside Control, Data, Event, and Observability planes. Contracts are orthogonal to events and are the source of consent, not activity. |

---

## 3. What v1.4 Keeps from v1.3

The following v1.3 decisions remain authoritative and are carried forward unchanged:

- Environment-first hierarchy: `environment → organization → business unit → team → project → resource`
- Environment isolation at infrastructure level (not configuration)
- A2A cost tracking with per-agent token attribution
- Multi-model AI with cost-based routing
- Regulatory alignment with HIPAA, GDPR, EU AI Act, NIST AI RMF, ITIL v4
- Evidence-first architecture
- Disaster recovery SLAs (RPO 15 min, RTO 4 hr)
- Performance SLAs by tier (standard / premium / enterprise)
- Vendor and subprocessor management with GDPR Article 28 alignment
- ITIL v4 incident response, change management, and problem tracking

v1.4 amends these sections with agent-readiness and contract-plane integration but does not replace them.

---

## 4. Document Set Structure

v1.4 publishes as a **document set**, not a single monolithic file. Each specification is self-contained and retrievable by AI agents without loading the full scaffold. The monolithic scaffold (`Enterprise_MultiPlatform_Architecture_Scaffold_v1.4.md`) indexes into the set for human readers who prefer linear reading.

### 4.1 Constitutional Layer (Foundation)

| # | Document | Purpose |
|---|----------|---------|
| 01 | Core Principles and Architectural Tenets | Amended constitutional principles and the machine-readable tenet list |
| 02 | SDK-First Architecture | Normative specification for SDK design and client consumption rules |
| 03 | Contract-Based Trust Model | Normative specification for contracts, refusal, and the contract ledger |

### 4.2 Interface and Surface Layer

| # | Document | Purpose |
|---|----------|---------|
| 04 | API Architecture (Command, Query, Internal) | Three-surface API taxonomy: REST commands, GraphQL queries, internal gRPC |
| 05 | Documentation for Machine Readers | `llms.txt`, agent context files, RAG-optimized markdown rules |
| 06 | Agentic Governance and Guardrails | Allowed / approval-required / prohibited operations for autonomous agents |

### 4.3 Deployment and Integration Layer

| # | Document | Purpose |
|---|----------|---------|
| 07 | Edge, NATS, and Flutter Clients | Edge node deployment class, NATS JetStream leaf nodes, Go aggregator / Flutter client pattern |
| 08 | Event-Driven Architecture (v1.4 revision) | Updated event bus selection with NATS JetStream as primary |
| 09 | Identity, Delegation, and Cryptographic Authority | DID-based identity, explicit delegation chains, per-request signatures |

### 4.4 Preserved Sections (v1.3 carried forward with amendments)

The following v1.3 sections remain in the monolithic scaffold with targeted amendments:

- Identity, Scopes, Roles, Authorities (amended with delegation-chain requirements)
- A2A Cost Tracking and FinOps (carried forward)
- Multi-Model AI Strategy (carried forward)
- Regulatory Compliance and AI Governance (amended with contract-ledger evidence)
- Security Architecture and Hardening (amended with agentic security subsection)
- Incident Response and Breach Notification (carried forward)
- Disaster Recovery and Business Continuity (carried forward)
- Performance and Scalability Targets (carried forward)
- Vendor and Subprocessor Management (carried forward)
- Change Management and Release Governance (carried forward)
- Developer Experience and Local Development (amended with agent-readiness metrics)
- Endpoint Security and Device Management (carried forward)

---

## 5. Stakeholder Guide

| Role | Primary v1.4 Sections | What Changed |
|------|----------------------|--------------|
| **CTO/CIO** | 02 (SDK-First), 04 (API Architecture), 07 (Edge/NATS), 08 (Events) | Architecture tentpoles; SDK is now the product |
| **CISO** | 03 (Contract-Based Trust), 06 (Agentic Governance), 09 (Identity) | Trust model is cryptographic; agents operate under explicit contracts |
| **CAIO** | 03 (Contract-Based Trust), 05 (Machine-Readable Docs), 06 (Agentic Governance) | Agents are governed by contracts; documentation serves agents first |
| **CFO/COO** | Sections carried from v1.3 (cost tracking, vendor management) | No material change |
| **CRO/CLO** | 03 (Contract-Based Trust), Regulatory Compliance (amended) | Contract ledger provides non-repudiable evidence |
| **Developer / Platform Engineer** | 02 (SDK-First), 04 (API), 05 (Machine-Readable Docs), 07 (Edge) | Service template now includes SDK, MCP, CLI packages by default |

---

## 6. Implementation Readiness

A platform conforms to EPAS v1.4 when all of the following are demonstrable:

1. **SDK-First Compliance** — Every interface (UI, CLI, automation, agent) consumes the platform through an official SDK. No client calls REST or GraphQL endpoints directly.
2. **Contract Ledger Operational** — Every mutating operation produces a contract ledger entry before execution begins. Refusals are recorded with signed reasons.
3. **Agent-Readable Documentation** — Every service repository contains a current `llms.txt` and an agent context file (`CLAUDE.md` or equivalent) validated in CI.
4. **Agentic Guardrails Enforced** — Autonomous agent operations are classified as allowed, approval-required, or prohibited, and the classification is enforced by CI and by runtime authorization checks.
5. **Five-Plane Separation** — Control, Data, Event, Contract, and Observability planes are architecturally distinct; no plane mutates state owned by another.
6. **Edge-Deployable** — At least one reference service can be deployed to an Edge Node with intermittent connectivity and replay its events to the central hub on reconnection.

Partial conformance is acceptable during migration from v1.3. Full conformance is required for new services.

---

## 7. EPAS v1.4 Migration from v1.3

Existing v1.3 platforms migrate to v1.4 in four phases:

### 7.1 Phase 1 — Documentation and Tenets (Weeks 1–2)

- Add `llms.txt` and `CLAUDE.md` (or equivalent) to every repository
- Convert existing documentation to RAG-optimized markdown rules
- Update architectural tenets file to the v1.4 list

### 7.2 Phase 2 — SDK-First Migration (Weeks 3–8)

- Extract SDK packages from each service
- Update CLI, MCP server, and UI to consume SDK exclusively
- Deprecate direct API consumption paths
- Generate SDKs from OpenAPI specs where possible

### 7.3 Phase 3 — Contract Ledger Introduction (Weeks 6–16)

- Implement the contract ledger (Certificate-Transparency-style log or immutable event store with hash chaining)
- Instrument mutating operations to write contract offer / acceptance / attestation entries
- Update SDKs to return `ContractHandle` objects for mutating calls
- Introduce refusal as a first-class response

### 7.4 Phase 4 — Agentic Governance and Edge (Weeks 12–24)

- Classify all agent operations as allowed / approval-required / prohibited
- Implement CI validation of guardrails
- Introduce Edge Node deployment class where applicable
- Migrate event bus to NATS JetStream if Kafka was used solely for latency (not retention)

---

## 8. Non-Goals

EPAS v1.4 does not:

- Provide legal advice or regulatory certification
- Prescribe specific vendors, products, or open-source projects
- Encode business logic for specific industries
- Replace internal policies, engineering standards, or security programs

v1.4 is a technical scaffold aligned with internal policies, legal opinions, and regulatory programs. Organizations adapt v1.4 to their specific context.

---

## 9. Authorship and Control

**Document Owner:** Platform Architecture
**Control:** Changes to Core Principles, Architectural Tenets, Contract Model, or SDK Architecture require explicit approval per the v1.4 agent context file (`1.4 Planning/CLAUDE.md`).

---

## 10. EPAS v1.4 Relationship to Reference Implementations

EPAS v1.4 is informed by one or more reference implementations that have materially influenced the specification. Reference implementations are **not named** in v1.4 specification text. The specification remains vendor-neutral and deployment-neutral.

Reference implementations may publish their own mapping documents that trace EPAS v1.4 requirements to their specific services, technologies, and naming conventions. Such mapping documents are out of scope for this repository.
