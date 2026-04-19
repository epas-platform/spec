# EPAS v1.4 — Core Principles and Architectural Tenets

> **Status:** Draft — Constitutional Layer
> **Normative.** This specification defines the governing principles and the machine-readable architectural tenets for EPAS v1.4. All subsequent v1.4 specifications are subordinate to the principles defined here.

---

## 1. Purpose

This specification establishes the **governing principles** and **architectural tenets** that every EPAS v1.4 platform MUST observe. Principles express intent; tenets encode that intent as machine-checkable constraints.

Principles are prose. Tenets are structured. Both are authoritative.

---

## 2. Relationship to v1.3

EPAS v1.3 defined ten core principles. EPAS v1.4 **preserves all ten** and adds five. No v1.3 principle is removed.

The five additions reflect the v1.4 constitutional shift: the platform's primary consumer is an AI agent, and the platform's primary safety property is provable consent.

---

## 3. Core Principles (v1.4)

The v1.4 core principles drive every architectural decision. Principles are listed in precedence order. When two principles conflict, the earlier principle wins.

### 3.1 Principles Carried from v1.3

- **Defense in depth** — The platform enforces multiple layers of protection: edge, service, data, tooling.
- **Least privilege by default** — Services, agents, and tools receive the minimum access necessary to perform their function.
- **Zero-trust boundaries** — Every internal call is authenticated and authorized. The platform grants no implicit trust based on network position.
- **Deterministic orchestration** — Agents and tools execute through schema-validated flows. Ad-hoc orchestration is non-conformant.
- **Environment isolation mandatory** — Development, testing, staging, and production environments are isolated at the infrastructure level, not at the configuration level.
- **Dev/prod parity** — Local development mirrors cloud deployment behavior. Platform code MUST NOT branch on environment.
- **Evidence-first architecture** — Every control MUST be proven with logs, configurations, and traces. Undocumented controls are considered absent.
- **Privacy and security by default** — Encryption is enforced everywhere. Data minimization and strong access control are defaults, not options.
- **Cost visibility mandatory** — Every agent-to-agent call, LLM invocation, and resource usage is tracked. Untracked cost is non-conformant.
- **Multi-model by default** — The platform supports multiple LLM providers to enable cost optimization and vendor resilience.

### 3.2 Principles New in v1.4

- **Machine-readable by default** — Documentation serves AI agents as primary consumers. Human readability is preserved but not prioritized over agent comprehension. Repositories without machine-readable discovery indices (`llms.txt`) and agent context files are non-conformant.
- **SDK-first consumption** — SDKs are the product. APIs are implementation detail. Every client MUST consume the platform through an official SDK. Direct API consumption by clients is non-conformant.
- **Contract-based execution** — No work is performed without an explicit, signed, ledger-recorded contract. Execution without a contract is non-conformant regardless of outcome.
- **Refusal is a first-class outcome** — An executor's refusal to perform contracted work is a valid, auditable, structured outcome — not an exception and not a failure mode. Platforms that treat refusal as an error are non-conformant.
- **Event-as-truth, contract-as-consent** — The event stream records what happened. The contract ledger records what was agreed to. Events and contracts are orthogonal. Neither substitutes for the other.

---

## 4. Architectural Tenets (v1.4)

Architectural tenets are the machine-readable encoding of the principles above. Each tenet is a Boolean assertion that the platform either satisfies or does not. Tenets are consumed by CI validation, conformance tests, and architecture review.

```yaml
architectural_tenets:

  # Tenancy and Isolation (unchanged from v1.3)
  - tenant_isolation_by_design
  - environment_first_architecture
  - business_unit_flexibility

  # Configuration and Development (unchanged from v1.3)
  - profile_based_environments
  - lazy_credential_loading
  - environment_agnostic_code

  # Security and Trust (unchanged from v1.3)
  - zero_trust_internal_calls
  - defense_in_depth
  - mcp_for_tool_integration_only

  # Orchestration and Events (unchanged from v1.3)
  - deterministic_orchestration
  - event_plane_mandatory
  - event_bus_pluggable

  # AI and Cost (unchanged from v1.3)
  - a2a_cost_tracking_mandatory
  - multi_model_by_default
  - ai_cost_tracking_built_in
  - token_attribution_per_agent

  # Standards and Operations (unchanged from v1.3)
  - open_standards_everywhere
  - infrastructure_as_code
  - local_dev_equals_cloud
  - devsecops_and_agentops_required

  # Compliance and Governance (unchanged from v1.3)
  - compliance_as_code
  - evidence_first
  - regulatory_alignment_built_in
  - supply_chain_transparency
  - itil_v4_service_management

  # Documentation for Machine Readers (NEW in v1.4)
  - documentation_dual_audience
  - llms_txt_mandatory
  - agent_context_files_required
  - rag_optimized_markdown
  - machine_readable_by_default

  # SDK-First Architecture (NEW in v1.4)
  - sdk_first_architecture
  - no_direct_api_consumption
  - openapi_sdk_generation
  - sdk_version_independent_of_api

  # Contract-Based Trust (NEW in v1.4)
  - contract_based_execution
  - contract_ledger_mandatory
  - contract_ledger_append_only
  - contract_ledger_tamper_evident
  - contract_ledger_externally_anchored     # Merkle roots to operator-independent medium
  - contract_modeling_declared              # Formal or envelope-based, stated in conformance.yaml
  - contract_substrate_declared             # Permissioned / CT-log / event-store / witness
  - refusal_is_first_class
  - contract_envelope_signed

  # Agentic Governance (NEW in v1.4)
  - agentic_governance_enabled
  - agent_operations_classified
  - agent_guardrails_ci_validated
  - agent_guardrails_runtime_enforced

  # Edge and Resilience (NEW in v1.4)
  - edge_deployment_supported
  - event_replay_on_reconnect
  - offline_tolerant_execution
```

---

## 5. Tenet Enforcement

Every tenet MUST be enforced at one or more of the following layers:

| Layer | Enforcement Mechanism | Examples |
|-------|----------------------|----------|
| **Compile-time** | Type system, linter, code generation | `sdk_first_architecture`, `openapi_sdk_generation` |
| **CI-time** | Repository validation, schema checks | `llms_txt_mandatory`, `agent_guardrails_ci_validated`, `rag_optimized_markdown` |
| **Deploy-time** | Configuration validation, manifest checks | `environment_first_architecture`, `infrastructure_as_code` |
| **Runtime** | Authorization, audit, ledger | `contract_based_execution`, `zero_trust_internal_calls`, `agent_guardrails_runtime_enforced` |

A tenet enforced only by documentation or developer discipline is non-conformant. Enforcement MUST be mechanical.

---

## 6. Five-Plane Architecture

EPAS v1.4 organizes the platform into **five logical planes**. v1.3 defined four planes; v1.4 adds the **Contract Plane**.

### 6.1 Plane Definitions

- **Control Plane** — Configuration of tenants, organizations, business units, projects, model catalogs, tool catalogs, policies, guardrails, access control, observability, and cost budgets.
- **Data Plane** — Runtime agents, workflows, connectors, tools, operational storage, and multi-tenant data isolation.
- **Event Plane** — Asynchronous messaging backbone, CloudEvents propagation, AsyncAPI contracts, dead-letter queues, retry logic. The Event Plane records **what happened**.
- **Contract Plane** *(new in v1.4)* — Contract ledger, delegation chains, refusal records, attestations. The Contract Plane records **what was agreed to**. The Contract Plane is orthogonal to the Event Plane and MUST NOT be conflated with it.
- **Observability Plane** — Logs, metrics, traces, security and audit logs, evaluation runs, cost tracking, AI decision logging, explainability.

### 6.2 Plane Separation Invariants

- **No plane mutates state owned by another plane.** The Observability Plane does not write to the Data Plane. The Event Plane does not write to the Contract Plane. Violations are architectural defects.
- **Planes communicate through well-defined interfaces.** The Control Plane distributes configuration through versioned schemas. The Contract Plane produces attestations that the Event Plane may reference by ID but does not copy.
- **Contracts are not events, and events are not contracts.** Events are high-volume, ephemeral, and noisy. Contracts are low-volume, durable, and authoritative. Systems that conflate them fail under audit.

### 6.3 Plane Diagram

```
┌───────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                           │
│  Tenants · Orgs · BUs · Projects · Catalogs · Policies · FinOps│
└─────────────────────────┬─────────────────────────────────────┘
                          │ configures
         ┌────────────────┼────────────────┐
         ▼                ▼                ▼
┌─────────────┐   ┌──────────────┐   ┌──────────────┐
│ DATA PLANE  │   │ EVENT PLANE  │   │ CONTRACT     │
│             │   │              │   │ PLANE        │
│ Agents      │──▶│ CloudEvents  │   │              │
│ Workflows   │   │ AsyncAPI     │   │ Ledger       │
│ Tools       │◀──│ DLQ/Retry    │   │ Delegations  │
│ Storage     │   │              │   │ Refusals     │
└──────┬──────┘   └──────┬───────┘   │ Attestations │
       │                 │           └──────┬───────┘
       │                 │                  │
       │                 │                  │ records
       │                 │                  ▼
       │                 │         (what was agreed to)
       │                 │
       │                 ▼
       │        (what happened)
       │
       ▼
┌───────────────────────────────────────────────────────────────┐
│                   OBSERVABILITY PLANE                          │
│  Logs · Metrics · Traces · Audit · Cost · AI Decisions        │
└───────────────────────────────────────────────────────────────┘
```

---

## 7. Conformance

A platform conforms to EPAS v1.4 core principles and architectural tenets when:

1. Every principle in Section 3 is observably upheld.
2. Every tenet in Section 4 is enforced at the appropriate layer per Section 5.
3. The five-plane separation in Section 6 is architecturally distinct.
4. No v1.3 principle has been weakened or removed.
5. CI validates tenet conformance on every pull request.

Partial conformance is permitted during migration from v1.3 and is tracked explicitly in a `conformance.yaml` file at the repository root. Drift from declared conformance is a platform incident.

---

## 8. Precedence

When specifications in this document set conflict:

1. Core Principles (this document, Section 3) are highest precedence.
2. Architectural Tenets (this document, Section 4) are next.
3. Specifications 02 through 09 in the v1.4 document set follow.
4. Amendments to v1.3 sections in the monolithic scaffold are lowest precedence.

A later specification that contradicts an earlier principle is defective. Defects are resolved by amending the later specification, not by weakening the principle.

---

## 9. Explicit Non-Goals

This specification does not:

- Define specific vendors, products, or open-source projects
- Define specific programming languages or frameworks
- Define specific cloud providers or deployment topologies
- Define specific AI models or inference providers

Those decisions live in the sections that follow and remain vendor-neutral throughout.
