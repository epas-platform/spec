Enterprise Multi‑Platform Architecture Scaffold
Version 1.4.0 — AI-First, SDK-First, Contract-Based Trust

⸻

# 🎯 What's New in v1.4.0

**Strategic Shifts** (v1.3 → v1.4):

1. **AI-First Documentation** – Documentation serves AI agents as primary consumers. `llms.txt`, agent context files, and RAG-optimized markdown are mandatory in every repository.
2. **SDK-First Architecture** – SDKs are the product; APIs are implementation detail. Every client (UI, CLI, automation, agent) consumes the platform exclusively through an official SDK.
3. **Contract-Based Trust Model** – No work is performed without an explicit, signed, ledger-recorded contract. Refusal is a first-class, auditable outcome.
4. **Externally Anchored Ledger** – The contract ledger is anchored via Merkle commitments to an operator-independent public medium so integrity is verifiable without platform cooperation.
5. **Five-Plane Architecture** – v1.4 adds a **Contract Plane** alongside Control, Data, Event, and Observability. Contracts are not events; the two planes have distinct substrates and retention profiles.
6. **Agentic Governance** – Autonomous agent operations are classified as allowed, approval-required, or prohibited. Classification is enforced at four layers (soft / CI / runtime / ledger).
7. **Edge-Capable Deployment** – A formally specified Edge Node class supports constrained hardware and intermittent connectivity. NATS JetStream leaf nodes provide offline-tolerant transport; Flutter clients implement contract lifecycles as FSMs.
8. **NATS JetStream Primary** – NATS JetStream is promoted to recommended primary event backbone. Kafka and Redpanda remain permitted for compliance-driven retention scenarios.
9. **DID-Based Identity and Delegation** – Bearer tokens prove identity only; authority is expressed through signed delegation chains with per-request signatures.

**v1.4 Preserves Every v1.3 Principle.** No v1.3 principle is removed or weakened. v1.4 is additive.

**For Stakeholders:**

- **CTO / CIO**: See document set specifications 02, 04, 07, 08 for the architectural tentpoles.
- **CISO**: See specifications 03, 06, 09 for the trust model, agentic governance, and identity.
- **CAIO**: See specifications 03, 05, 06 for contracts, AI-first documentation, and agent governance.
- **CFO / COO**: Cost tracking, DR/BC, and vendor management carry forward from v1.3 without material change.
- **CRO / CLO**: See specification 03 for non-repudiable contract-ledger evidence.

⸻

# Document Control

## Version History

- **v1.0.0** – Initial reference architecture and multi-tenant model
- **v1.1.0** – Event-driven extensions, A2A architecture, AI governance primitives, initial cost tracking model
- **v1.2.0** – Environment & profile strategy, AWS-first deployment blueprint, full data handling & security baseline
- **v1.3.0** – Environment-first hierarchy, comprehensive A2A cost management, regulatory compliance integration, multi-model LLM strategy, expanded event architecture options, endpoint security
- **v1.4.0** – AI-first documentation, SDK-first architecture, contract-based trust model with external anchoring, five-plane model (adds Contract Plane), agentic governance, edge deployment class, NATS JetStream primary, DID-based identity and delegation

## Audience & Stakeholder Roles (RACI)

Carried forward from v1.3 without change. See [Enterprise_MultiPlatform_Architecture_Scaffold.md](./Enterprise_MultiPlatform_Architecture_Scaffold.md#audience--stakeholder-roles-raci) for the full RACI table.

## Non-Goals

- This is not legal advice or a certification document.
- This is a technical scaffold that is aligned with internal policies, legal opinions, and regulatory programs.
- Implementation details are illustrative; organizations adapt to their specific context.
- v1.4 is vendor-neutral. Reference implementations inform the specification but are not named in specification text.

## Conformance Declaration

Every service or platform claiming EPAS v1.4 conformance MUST publish a `conformance.yaml` at the repository root declaring its conformance state. Partial conformance during migration from v1.3 is permitted and MUST be explicitly declared. A template is published at [1.4/conformance.yaml.template](./1.4/conformance.yaml.template).

⸻

# Document Set Map

EPAS v1.4 publishes as a **document set** under `1.4/`. This scaffold file provides narrative, amendments to sections not broken out as standalone specs, and stakeholder cross-references. Each document in the set is normative and retrievable on its own.

| # | Document | Scope |
|---|----------|-------|
| [00](./1.4/00-overview.md) | v1.4 Overview | Strategic positioning, 8 shifts, migration plan, stakeholder guide |
| [01](./1.4/01-core-principles-and-tenets.md) | Core Principles and Architectural Tenets | 15 principles, machine-readable tenet list, five-plane model |
| [02](./1.4/02-sdk-first-architecture.md) | SDK-First Architecture | Normative SDK specification; `ContractHandle` as primary return type |
| [03](./1.4/03-contract-based-trust-model.md) | Contract-Based Trust Model | Contract lifecycle, envelope schema, three-concern ledger decomposition (modeling / substrate / external anchoring) |
| [04](./1.4/04-api-architecture.md) | API Architecture (Command, Query, Internal) | REST / GraphQL / gRPC taxonomy, canonical URL structure, task dispatch under contract |
| [05](./1.4/05-documentation-for-machine-readers.md) | Documentation for Machine Readers | `llms.txt`, `CLAUDE.md` required sections, five RAG-optimization rules, MCP tool documentation |
| [06](./1.4/06-agentic-governance.md) | Agentic Governance and Guardrails | Three-tier classification, four enforcement layers, emergency override, trust accumulation |
| [07](./1.4/07-edge-nats-flutter.md) | Edge, NATS, and Flutter Clients | Edge Node deployment class, NATS leaf-node topology, Go aggregator, Flutter FSM requirement |
| [08](./1.4/08-event-driven-architecture.md) | Event-Driven Architecture | NATS JetStream primary, Kafka/Redpanda permitted with justification, CloudEvents `contractid` extension, AsyncAPI mandatory |
| [09](./1.4/09-identity-and-delegation.md) | Identity, Delegation, and Cryptographic Authority | DID-based identity, four identity classes, three-tier agent model, signed delegation chains, per-request signatures |

**How to read v1.4:**

- **Executives**: Read the "What's New" section above, then [00 Overview](./1.4/00-overview.md).
- **Architects**: Read [01 Core Principles](./1.4/01-core-principles-and-tenets.md), then [02](./1.4/02-sdk-first-architecture.md), [03](./1.4/03-contract-based-trust-model.md), and [04](./1.4/04-api-architecture.md) in sequence.
- **Security**: Read [03 Contract-Based Trust Model](./1.4/03-contract-based-trust-model.md), [06 Agentic Governance](./1.4/06-agentic-governance.md), and [09 Identity and Delegation](./1.4/09-identity-and-delegation.md).
- **Platform engineers**: Read [02 SDK-First](./1.4/02-sdk-first-architecture.md), [04 API Architecture](./1.4/04-api-architecture.md), [05 Documentation](./1.4/05-documentation-for-machine-readers.md).
- **Edge / mobile engineers**: Read [07 Edge, NATS, and Flutter Clients](./1.4/07-edge-nats-flutter.md) after the architectural tentpoles.

⸻

# Core Principles

Normative specification: [1.4/01-core-principles-and-tenets.md](./1.4/01-core-principles-and-tenets.md)

v1.4 defines 15 core principles: the 10 carried forward from v1.3 and 5 new principles that enable the AI-first and SDK-first shift. Principles are listed in precedence order; earlier principles win conflicts.

**v1.3 principles preserved (brief form):** defense in depth; least privilege; zero-trust boundaries; deterministic orchestration; environment isolation mandatory; dev/prod parity; evidence-first architecture; privacy and security by default; cost visibility mandatory; multi-model by default.

**v1.4 additions:**

- **Machine-readable by default** — documentation targets AI agents as primary consumers.
- **SDK-first consumption** — every client consumes the platform through an official SDK.
- **Contract-based execution** — no work is performed without an explicit, signed, ledger-recorded contract.
- **Refusal is a first-class outcome** — refusal is a structured, signed, auditable outcome, never an exception.
- **Event-as-truth, contract-as-consent** — the event stream records what happened; the contract ledger records what was agreed to. The two planes are orthogonal and non-substitutable.

See [01 Core Principles and Tenets](./1.4/01-core-principles-and-tenets.md) for the full normative text and the complete machine-readable tenet list.

⸻

# Architectural Overview

## Five Planes (v1.4)

Normative specification: [1.4/01-core-principles-and-tenets.md § 6](./1.4/01-core-principles-and-tenets.md#6-five-plane-architecture)

v1.3 defined four planes: Control, Data, Event, Observability. v1.4 adds a fifth: the **Contract Plane**. Contracts are distinct from events: contracts have low volume and long retention; events have high volume and operational retention. Systems that conflate the two planes fail under audit.

```
Control Plane ─────────┐
                       │ configures
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
   Data Plane ──▶ Event Plane     Contract Plane
   (runtime)     (what happened)  (what was agreed)
       │             │                  │
       └─────────────┴────── records ───┘
                         │
                         ▼
                 Observability Plane
```

## Environment-First Hierarchy

Carried forward from v1.3 without change. The authoritative definition remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § Environment-First Hierarchy](./Enterprise_MultiPlatform_Architecture_Scaffold.md#environment-first-hierarchy).

```
environment → organization → business_unit → team → project → resource
```

All v1.3 rules apply: separate infrastructure per environment; environment-agnostic code; no environment-specific branching; DevSecOps + AgentOps mandatory.

## Architectural Tenets

Normative specification: [1.4/01-core-principles-and-tenets.md § 4](./1.4/01-core-principles-and-tenets.md#4-architectural-tenets-v14).

v1.4 defines ~35 tenets grouped as follows:

- Tenancy and Isolation (unchanged from v1.3)
- Configuration and Development (unchanged from v1.3)
- Security and Trust (unchanged from v1.3)
- Orchestration and Events (unchanged from v1.3)
- AI and Cost (unchanged from v1.3)
- Standards and Operations (unchanged from v1.3)
- Compliance and Governance (unchanged from v1.3)
- **Documentation for Machine Readers** (NEW in v1.4)
- **SDK-First Architecture** (NEW in v1.4)
- **Contract-Based Trust** (NEW in v1.4)
- **Agentic Governance** (NEW in v1.4)
- **Edge and Resilience** (NEW in v1.4)

Every tenet is enforced at one or more of: compile-time, CI-time, deploy-time, runtime. Tenets enforced only by documentation or developer discipline are non-conformant. See the full tenet list and enforcement layer mapping in [01 § 4–5](./1.4/01-core-principles-and-tenets.md#4-architectural-tenets-v14).

⸻

# Identity, Scopes, Roles, and Authorities

## Scopes

Scope hierarchy is unchanged from v1.3: `environment → organization → business_unit → team → project → resource`. See [Enterprise_MultiPlatform_Architecture_Scaffold.md § Scopes](./Enterprise_MultiPlatform_Architecture_Scaffold.md#scopes-environment-first-hierarchy) for the full v1.3 treatment of scope IDs, parent pointers, lifecycle, compliance tags, residency tags, and cost allocation tags.

## Authorities and Roles

Authorities (atomic immutable permissions) and Roles (configurable bundles) are unchanged from v1.3. ABAC augmentation is unchanged.

## v1.4 Amendment — Delegation Chains and DID-Based Identity

v1.4 replaces token-passthrough authentication with cryptographically provable identity and delegation. The authoritative specification is [1.4/09-identity-and-delegation.md](./1.4/09-identity-and-delegation.md).

Key amendments to v1.3:

- Every identity is a **decentralized identifier (DID)** resolvable to a DID document with public keys.
- Every mutating request carries a **signed delegation envelope** that declares the requester's DID, the delegation chain to the root authority, and the requested scope.
- Tokens (OIDC sessions, workload credentials) prove identity only. Authority never derives from token possession.
- Agent identity progresses through a **three-tier model**: registration identity (DID at creation), operational identity (wallet enrollment with signing keypair), scoped authority (contract-specific delegation).
- **Per-request signatures** prevent confused-deputy vulnerabilities by binding authority to the specific request being made.
- **Revocation** takes effect within a declared propagation SLA (default 30 seconds); cached chains are invalidated on revocation events.

See [09 Identity and Delegation](./1.4/09-identity-and-delegation.md) for the full envelope schema, chain verification rules, and revocation semantics.

⸻

# Section 9. A2A Cost Tracking & FinOps

Carried forward from v1.3 without material change. See [Enterprise_MultiPlatform_Architecture_Scaffold.md § 9 (Cost Tracking Architecture)](./Enterprise_MultiPlatform_Architecture_Scaffold.md#cost-tracking-architecture) for the full v1.3 specification covering:

- Inter-agent cost attribution
- MCP protocol overhead accounting
- Per-agent token attribution
- Real-time budget alerts at 75% (warning) and 95% (critical) thresholds
- Chargeback to business units

v1.4 does not amend the cost-tracking mechanics. The v1.4 contract model does contribute new attribution possibilities — every cost event can now be attributed to the contract under which it was incurred — but this attribution is additive and does not change the v1.3 cost tracking architecture.

⸻

# Section 10. Multi-Model AI Strategy

Carried forward from v1.3 without change. See [Enterprise_MultiPlatform_Architecture_Scaffold.md § 10](./Enterprise_MultiPlatform_Architecture_Scaffold.md#provider-selection-matrix) covering:

- Provider selection matrix
- Recommended multi-model stack
- Model routing and fallback logic
- Model registry and versioning
- Fine-tuned model governance

v1.4 amends only the approval authority: fine-tuned model deployments now require a governance-owner signed policy change per [06 Agentic Governance § 5](./1.4/06-agentic-governance.md#5-declarative-classification) when the deployment affects agent behavior. The v1.3 CAIO approval model remains authoritative for model selection; the v1.4 amendment applies only when the model is consumed by an autonomous agent.

⸻

# Section 11. Event-Driven Architecture

Superseded by [1.4/08-event-driven-architecture.md](./1.4/08-event-driven-architecture.md).

v1.4 changes:

- NATS JetStream is promoted to recommended primary event backbone.
- Kafka and Redpanda remain permitted with declared justification (compliance retention, existing deployment, specific ecosystem integrations).
- Every event MUST carry `contractid`, `operationid`, `correlationid`, and `causationid` CloudEvents extension attributes.
- AsyncAPI contracts remain mandatory.
- Schema evolution follows backward-compatible + forward-compatible rules within a major version.
- Dead-letter policies and retry discipline are explicitly specified.
- Per-entity ordering via partition keys is mandatory; cross-entity ordering is not guaranteed.

The v1.3 Phase 1 / Phase 2 phased rollout language is retired. Teams select a backbone on merit against the criteria in [08 § 4](./1.4/08-event-driven-architecture.md#4-event-bus-selection-criteria).

⸻

# Section 12. Observability

Carried forward from v1.3 without change. OpenTelemetry + LGTM stack remains the recommended reference. See [Enterprise_MultiPlatform_Architecture_Scaffold.md](./Enterprise_MultiPlatform_Architecture_Scaffold.md) for the v1.3 treatment.

v1.4 amendment: the Event Plane emits metrics (per-subject message rate, consumer lag, DLQ depth, retry rate, schema validation failures, edge buffer depth) that feed the Observability Plane. See [08 § 14](./1.4/08-event-driven-architecture.md#14-observability-of-the-event-plane).

⸻

# Section 13. Compliance & Accessibility

Carried forward from v1.3 without change.

⸻

# Section 14. Regulatory Compliance & AI Governance

Carried forward from v1.3 with amendments. The authoritative v1.3 content covering HIPAA, GDPR, EU AI Act, BIPA, ISO/IEC 42001, NIST AI RMF, and ITIL v4 ITSM alignment remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 14](./Enterprise_MultiPlatform_Architecture_Scaffold.md#regulatory-landscape-overview).

v1.4 amendments:

- **Contract ledger as evidence** — the contract ledger (specification 03) and its external anchoring (03 § 8.3) provide non-repudiable evidence for audit scenarios that v1.3 satisfied through log aggregation. Regulators requiring cryptographic evidence of consent and execution can verify directly against the external anchor without platform cooperation.
- **EU AI Act high-risk governance** — high-risk AI system governance requirements (human oversight, bias testing, logging) map to v1.4's Agentic Governance (specification 06). Approval-required operations are the human oversight mechanism; contract-ledger refusal records provide the bias-relevant audit trail.
- **Evidence matrix extension** — v1.3's Evidence & Equivalency Matrix is extended with columns for contract-ledger anchoring, agentic-governance classification, and SDK conformance attestation.

⸻

# Section 15. Security Architecture & Hardening

Carried forward from v1.3 with one major amendment. The authoritative v1.3 content covering Defense in Depth, Encryption Standards (TLS 1.3, AES-256), Key & Secret Management, Network Segmentation & Zero Trust, LangGraph Security Patterns, MCP Hardening, FastAPI Security Baseline, AWS-Aligned Integration, and Supply Chain Security remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 15](./Enterprise_MultiPlatform_Architecture_Scaffold.md#defense-in-depth-model).

v1.4 amendment — new subsection **15.x Agentic Security & Guardrails**, superseded by the standalone specification [1.4/06-agentic-governance.md](./1.4/06-agentic-governance.md).

**Summary of 15.x:**

```yaml
agentic_governance:
  autonomous_operations:
    - read_documentation
    - run_tests
    - run_linters
    - create_feature_branches
    - modify_files_in: [/src, /tests, /docs]

  approval_required:
    - database_migrations
    - security_configuration_changes
    - production_deployments
    - new_external_dependencies
    - data_exports_above_threshold
    - cross_tenant_operations

  prohibited:
    - commit_secrets
    - modify_ci_cd_without_review
    - delete_production_data
    - bypass_permission_checks
    - modify_ledger_entries
    - modify_anchoring_config
    - grant_new_authorities
    - disable_audit_logging
    - disable_guardrails
```

Enforcement layers: soft (agent context file), CI-time (build validation), runtime (SDK + Gateway checks), ledger (post-hoc detection). All four layers MUST be present; any single layer is defense-in-depth, the combination is governance.

⸻

# Section 16. Incident Response & Breach Notification (ITIL v4 Aligned)

Carried forward from v1.3 without change. The authoritative v1.3 specification covering P0–P4 classification, SLA targets, and ITIL Major Incident Management remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 16](./Enterprise_MultiPlatform_Architecture_Scaffold.md#16-incident-response--breach-notification-itil-v4-aligned).

v1.4 amendment: security incidents involving contract-ledger tampering, agentic guardrail violations, or delegation-chain compromise are automatically P0 regardless of scope.

⸻

# Section 17. Disaster Recovery & Business Continuity

Carried forward from v1.3 without change. The authoritative v1.3 specification covering RPO 15-minute, RTO 4-hour targets, multi-region replication, and quarterly DR drills remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 17](./Enterprise_MultiPlatform_Architecture_Scaffold.md#17-disaster-recovery--business-continuity--detailed).

v1.4 amendment: the contract ledger and its external anchoring have DR requirements distinct from operational data. External anchoring is not a DR substitute for the ledger — the platform must recover its own ledger substrate from DR backups, and the external anchor provides integrity verification that the DR-restored ledger matches the pre-incident state.

⸻

# Section 18. Performance & Scalability Targets

Carried forward from v1.3 without change. The authoritative v1.3 specification covering production SLAs (p95 response 2000ms/1500ms/1000ms by tier), uptime targets (99.95%/99.995%), concurrent conversation capacity, and scaling triggers remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 18](./Enterprise_MultiPlatform_Architecture_Scaffold.md#18-performance--scalability-targets--detailed).

v1.4 amendment: the Contract Plane's Record-stage latency (5–50ms for CT-style logs; 10–100ms for immutable event stores; 50–500ms for witness networks) contributes to end-to-end command acceptance latency. Platforms SHOULD select ledger substrate based on the tenant tier's latency budget.

⸻

# Section 19. Vendor & Subprocessor Management

Carried forward from v1.3 without change. The authoritative v1.3 specification covering GDPR Article 28 alignment, SOC 2 / ISO 27001 / penetration test validation, Data Processing Agreements, and subprocessor registry remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 19](./Enterprise_MultiPlatform_Architecture_Scaffold.md#19-vendor--subprocessor-management--detailed).

⸻

# Section 20. Change Management & Release Governance (ITIL v4 Aligned)

Carried forward from v1.3 without change. The authoritative v1.3 specification covering CAB/ECAB processes and standard/normal/emergency changes remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 20](./Enterprise_MultiPlatform_Architecture_Scaffold.md#20-change-management--release-governance-itil-v4-aligned).

v1.4 amendment: agentic governance policy changes are change-managed as **normal changes** (or **emergency changes** with post-hoc review) and produce contract-ledger entries per [06 § 5.2](./1.4/06-agentic-governance.md#52-policy-versioning).

⸻

# Section 21. Developer Experience & Local Development

Carried forward from v1.3 with amendments. The authoritative v1.3 specification covering one-command setup, LocalStack, mkcert, hot-reload, and developer productivity metrics remains in [Enterprise_MultiPlatform_Architecture_Scaffold.md § 21](./Enterprise_MultiPlatform_Architecture_Scaffold.md#21-developer-experience--local-development--detailed).

v1.4 amendments:

**§ 21.1 One-Command Setup** — extended with agent context:

```bash
# Clone repository
git clone https://github.com/org/service
cd service

# One-command startup
docker-compose up -d

# Verify setup
make check-health

# Agent context automatically present:
# - /llms.txt          (repository index for AI agents)
# - /CLAUDE.md         (coding rules and guardrails)
# - /docs/mcp-tools.md (available MCP tools)
# - /conformance.yaml  (declared EPAS v1.4 conformance state)
```

**§ 21.8 Developer Productivity Metrics** — extended with agent-readiness metrics:

```yaml
developer_experience_targets:
  # Existing (v1.3)
  time_to_first_commit: "<60 minutes"
  time_to_productive: "<4 hours"

  # New (v1.4)
  llms_txt_current: true                  # llms.txt reflects current repo state
  claude_md_complete: true                # CLAUDE.md has all required sections
  rag_lint_passing: true                  # Markdown passes RAG-readiness checks
  agent_test_coverage: ">80%"             # MCP tools have test coverage
  sdk_first_compliance: true              # No client bypasses the SDK
  conformance_yaml_current: true          # conformance.yaml reflects reality
```

**§ 21.x Standard Repository Layout (NEW):**

```
service/
├── llms.txt                       # Agent discovery index (REQUIRED)
├── CLAUDE.md                      # Agent context file (REQUIRED)
├── README.md                      # Human-readable overview
├── conformance.yaml               # EPAS v1.4 conformance declaration (REQUIRED)
├── .cursor/rules                  # Cursor rules (symlink to CLAUDE.md)
├── .github/copilot-instructions.md # Copilot instructions (symlink to CLAUDE.md)
│
├── packages/
│   ├── api/                       # Core FastAPI / Rust / Go service
│   ├── sdk/                       # Language-native SDKs
│   │   ├── python/
│   │   ├── typescript/
│   │   └── go/
│   ├── mcp/                       # MCP server (SDK consumer)
│   ├── cli/                       # CLI (SDK consumer)
│   └── web/                       # UI (SDK consumer)
│
├── docs/
│   ├── architecture/              # C4 diagrams, ADRs
│   ├── api/                       # OpenAPI / GraphQL / AsyncAPI specs
│   └── mcp-tools.md               # MCP tool documentation
│
├── docker-compose.yml
├── pyproject.toml                 # Workspace root (or equivalent)
└── .gitlab-ci.yml                 # CI with conformance validation
```

⸻

# Section 22. Endpoint Security & Device Management

Carried forward from v1.3 without change.

⸻

# Roadmap

v1.4 introduces substantial additions on top of v1.3. The following phasing is recommended for new and migrating deployments.

## M0 — Foundation (v1.3 complete)

All v1.3 M0 deliverables remain applicable.

## M1 — Documentation and Tenets (weeks 1–2)

- Add `llms.txt` and `CLAUDE.md` to every repository.
- Convert existing documentation to RAG-optimized markdown (specification 05).
- Update `architectural_tenets` configuration to the v1.4 list.
- Publish `conformance.yaml` at the repository root declaring current state.

## M2 — SDK-First Migration (weeks 3–8)

- Extract SDK packages per service.
- Update CLI, MCP server, UI, and automation to consume SDK exclusively.
- Deprecate direct API consumption paths.
- Generate SDKs from OpenAPI where applicable and post-process to domain shape.

## M3 — Contract Ledger Introduction (weeks 6–16)

- Select contract modeling approach (formal or envelope-based) per specification 03 § 8.1.
- Select ledger substrate per specification 03 § 8.2.
- Implement external anchoring per specification 03 § 8.3.
- Instrument mutating operations to write contract entries.
- Update SDKs to return `ContractHandle` from mutating calls.
- Introduce refusal as a first-class SDK outcome.

## M4 — Agentic Governance (weeks 12–20)

- Classify every agent operation as allowed, approval-required, or prohibited.
- Implement CI validation of prohibited operations.
- Implement runtime authorization checks.
- Configure approval routing and notification channels.
- Integrate the agentic governance policy with the contract ledger.

## M5 — Edge and NATS (weeks 16–24, where applicable)

- Migrate event bus to NATS JetStream if Kafka was used solely for latency (not retention).
- For edge-applicable deployments: introduce Edge Node class, NATS leaf nodes, Go aggregator, Flutter FSM clients.

## M6 — Identity and Delegation Hardening (weeks 18–26)

- Mint DIDs for every identity.
- Convert to per-request signatures on mutating calls.
- Retire token-as-authority patterns.
- Implement three-tier agent identity.

## M7 — Production Hardening and Conformance Attestation (weeks 24–32)

- Close partial-conformance declarations in `conformance.yaml`.
- Conduct third-party audit of contract-ledger integrity.
- Validate external anchoring against independent auditor reproduction.
- Publish EPAS v1.4 conformance attestation.

⸻

# Summary

v1.4 is not an incremental revision. It is the framework-level acknowledgment that AI agents are first-class platform consumers, and that their consumption must be governed by cryptographically provable consent. Everything else — SDK-first, machine-readable documentation, five-plane architecture, external anchoring, edge deployment, DID-based identity — derives from that acknowledgment.

v1.4 is also additive: nothing in v1.3 is removed or weakened. A v1.3-conformant platform can migrate to v1.4 incrementally without abandoning operational properties that are already working.

Reference implementations may publish their own mapping documents that trace v1.4 requirements to specific products, technologies, and naming conventions. Such mappings are out of scope for this specification. v1.4 remains vendor-neutral.

⸻

# Appendices

## Appendix A. Cost Model Example

Carried forward from v1.3 without change. See [Enterprise_MultiPlatform_Architecture_Scaffold.md § Appendix A](./Enterprise_MultiPlatform_Architecture_Scaffold.md#a-cost-model-example-a2a-voice-platform---2025).

## Appendix B. Regulatory Checklist

Carried forward from v1.3 without change.

## Appendix C. Event Bus Matrix

Superseded by [1.4/08-event-driven-architecture.md § 4](./1.4/08-event-driven-architecture.md#4-event-bus-selection-criteria) (selection criteria) and § 5–6 (recommended primary and permitted alternatives).

## Appendix D. Security Checklist

Carried forward from v1.3 with amendments for agentic security per specification 06.

## Appendix E. Decisions Log

Carried forward from v1.3. v1.4 additions:

- v1.4-D1: SDK-first is mandatory; direct API consumption by clients is non-conformant.
- v1.4-D2: Contract ledger is mandatory for mutating operations.
- v1.4-D3: Contract ledger MUST be externally anchored; public blockchain anchoring is the recommended external medium.
- v1.4-D4: Storing full contracts on a public blockchain (as opposed to anchoring Merkle roots) is non-conformant.
- v1.4-D5: NATS JetStream is the recommended primary event bus; Kafka/Redpanda are permitted with declared justification.
- v1.4-D6: Refusal is a first-class contract outcome, represented as a structured signed object, never as an exception.
- v1.4-D7: Bearer tokens prove identity only; authority is always expressed through signed delegation.

## Appendix F. Documentation Checklist (NEW in v1.4)

Every EPAS v1.4 repository MUST satisfy the following checklist. CI validates the checklist on every pull request.

```yaml
documentation_checklist:
  repository_level:
    - id: llms_txt_exists
      description: "llms.txt exists at the repository root"
      required: true
    - id: llms_txt_current
      description: "Every path in llms.txt resolves to an existing file"
      required: true
    - id: claude_md_exists
      description: "CLAUDE.md (or equivalent per specification 05 § 6) exists"
      required: true
    - id: claude_md_sections_complete
      description: "CLAUDE.md contains all required sections from specification 05 § 5.1"
      required: true
    - id: readme_exists
      description: "README.md provides human-readable overview"
      required: true
    - id: conformance_yaml_exists
      description: "conformance.yaml exists at the repository root"
      required: true
    - id: cursor_rules
      description: ".cursor/rules exists when team supports Cursor"
      required: false
    - id: copilot_instructions
      description: ".github/copilot-instructions.md exists when team supports Copilot"
      required: false

  markdown_quality:
    - id: no_orphan_pronouns
      description: "Paragraphs do not rely on pronouns referencing prior paragraph antecedents"
      required: true
    - id: entity_named_headers
      description: "Headers include the entity being described, not generic words"
      required: true
    - id: strict_header_hierarchy
      description: "Strict H1 → H2 → H3 nesting with no skipped levels"
      required: true
    - id: self_contained_paragraphs
      description: "Each paragraph stands alone when chunked by a RAG pipeline"
      required: true
    - id: front_loaded_key_information
      description: "Key information appears in the first one or two sentences of each section"
      required: true

  sdk_documentation:
    - id: sdk_readme
      description: "SDK README with quick-start exists per language binding"
      required: true
    - id: sdk_public_methods_documented
      description: "All public SDK methods have docstrings"
      required: true
    - id: sdk_usage_examples
      description: "Usage examples for common operations are present"
      required: true
    - id: sdk_error_handling
      description: "Error handling and refusal semantics are documented"
      required: true

  mcp_documentation:
    - id: mcp_tools_documented
      description: "All MCP tools documented in docs/mcp-tools.md per specification 05 § 8"
      required: true
    - id: mcp_params_and_returns
      description: "Tool parameters and return types are specified"
      required: true
    - id: mcp_usage_examples
      description: "Usage examples present for each MCP tool"
      required: true
    - id: mcp_guardrails_documented
      description: "Risk level and approval requirements documented per tool"
      required: true

  contract_documentation:
    - id: mutating_ops_have_contract_context
      description: "Every documented mutating operation describes its contract offer, scope, expected refusal reasons, and emitted events"
      required: true
```

## Appendix G. Conformance Declaration Template

See [1.4/conformance.yaml.template](./1.4/conformance.yaml.template) for the machine-readable template. Every repository claiming EPAS v1.4 conformance publishes a completed `conformance.yaml` at the repository root.

⸻

**Enterprise Multi‑Platform Architecture Scaffold v1.4.0 — Draft**
