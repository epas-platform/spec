# EPAS v1.4 — Agentic Governance and Guardrails

> **Status:** Draft — Normative
> This specification defines how EPAS v1.4 platforms govern autonomous AI agent operations through a three-tier classification, the enforcement mechanisms that back each tier, and the integration between agentic guardrails and the Contract-Based Trust Model.

---

## 1. Purpose

EPAS v1.4 Agentic Governance defines what autonomous AI agents are permitted to do without human intervention, what requires human approval, and what is prohibited regardless of authorization.

The specification exists because AI agents are neither services (fully constrained by code) nor humans (constrained by judgment and accountability). An AI agent operates somewhere between — capable of taking action at speed, but without the embodied accountability that human actors carry. Platforms that treat agents as either extreme produce systems that either constrain agents into uselessness or cede too much authority to them.

This specification is **normative**. Platforms that deploy autonomous agents without agentic governance are non-conformant.

---

## 2. Relationship to Other Specifications

- **Specification 02 (SDK-First)** — Agents consume the platform through the same SDK as any other client. Agentic governance constrains *what* agents do through the SDK, not *how* they access the SDK.
- **Specification 03 (Contract-Based Trust)** — Approval-required and prohibited operations interact with the contract model at Review and Refuse stages. Approval-required operations produce contract offers that are held pending human approval. Prohibited operations produce immediate refusals.
- **Specification 05 (Machine-Readable Docs)** — The human-readable expression of an agent's guardrails lives in `CLAUDE.md` and equivalent context files. Runtime enforcement lives here (specification 06).
- **Specification 09 (Identity and Delegation)** — Agent identity is proven before guardrails are evaluated. An agent whose identity cannot be proven has no permitted operations.

---

## 3. Three-Tier Classification

Every operation an agent might perform MUST be classified into exactly one of three tiers.

| Tier | Meaning | Enforcement |
|------|---------|-------------|
| **Allowed** | The agent MAY perform this operation autonomously | Runtime authorization check passes silently |
| **Approval-Required** | The agent MAY offer this operation but MUST NOT execute it without human approval | Runtime creates a pending approval; blocks execution until resolved |
| **Prohibited** | The agent MUST NOT perform this operation under any circumstance | Runtime refuses the contract offer; no approval path exists |

Operations not explicitly classified are treated as **Prohibited** by default. Absence of classification is not permission.

---

## 4. Classification Rules

### 4.1 Allowed Operations

Allowed operations are typically:

- **Read-only operations** against non-sensitive data.
- **Reversible operations** against isolated resources.
- **Operations scoped to the agent's own working artifacts** (branches, feature files, sandbox environments).
- **Operations against explicitly-declared agent-controlled resources** (test databases, dev environments, ephemeral compute).

Example allowed operations:

- Read tenant configuration (the agent's own tenant).
- Create or modify files in `/src`, `/tests`, `/docs` on a feature branch.
- Run tests, linters, type checkers.
- Create feature branches.
- Query telemetry and observability data within the agent's tenant.
- Dispatch worker agents under contracts the agent holds.

### 4.2 Approval-Required Operations

Approval-required operations are typically:

- **Operations that affect shared infrastructure** visible to other teams or tenants.
- **Operations that consume budget** above a declared threshold.
- **Operations that cannot be cheaply reversed** but are not catastrophic.
- **Operations with regulatory or compliance implications** (changes to audit-relevant configuration, data retention settings).
- **Operations that cross tenant boundaries** for administrative purposes.

Example approval-required operations:

- Database schema migrations.
- New external dependencies or vendor integrations.
- Security-related configuration changes.
- Changes to authentication or authorization policy.
- Production deployments.
- Data exports above a declared row threshold.
- Cross-tenant administrative operations.

### 4.3 Prohibited Operations

Prohibited operations are typically:

- **Destructive operations that cannot be undone** without off-platform intervention.
- **Operations that would compromise the trust model** (bypassing the contract ledger, forging signatures, modifying audit records).
- **Operations that produce privacy or regulatory violations** (exfiltrating tenant data, disabling audit logging).
- **Operations against the governance system itself** (modifying guardrails, granting new authorities, tampering with the ledger).

Example prohibited operations:

- Committing secrets or API keys.
- Modifying CI/CD configuration without review.
- Deleting production data.
- Bypassing permission checks.
- Direct database access in production bypassing the SDK.
- Modifying or deleting contract ledger entries.
- Modifying external anchoring configuration.
- Granting new authorities to the agent itself or to other agents.
- Disabling audit logging, observability, or guardrail enforcement.

---

## 5. Declarative Classification

Every agent's classification is declared in a machine-readable policy file. The policy is the source of truth; human-readable guardrails in `CLAUDE.md` MUST be consistent with the policy.

### 5.1 Policy Shape

```yaml
# agentic_governance.yaml
agent_class: worker                    # supervisor | worker | utility
agent_id: svc-agent-builder

autonomous_operations:
  - read_documentation
  - run_tests
  - run_linters
  - create_feature_branches
  - modify_files_in:
      - /src
      - /tests
      - /docs
  - query_telemetry:
      scopes: ["tenant:example-corp"]

approval_required:
  - database_migrations:
      approver_count: 1
      sla: "30m"
  - security_configuration_changes:
      approver_count: 2
      sla: "2h"
  - production_deployments:
      approver_count: 1
      sla: "15m"
  - new_external_dependencies:
      approver_count: 1
      sla: "4h"
  - data_exports_above:
      threshold_rows: 10000
      approver_count: 1
      sla: "1h"
  - cross_tenant_operations:
      approver_count: 2
      sla: "4h"

prohibited:
  - commit_secrets
  - modify_ci_cd
  - delete_production_data
  - bypass_permission_checks
  - direct_db_access_in_prod
  - modify_ledger_entries
  - modify_anchoring_config
  - grant_new_authorities
  - disable_audit_logging
  - disable_guardrails

effective_from: "2026-04-19T00:00:00Z"
review_due_by: "2026-07-19T00:00:00Z"
signed_by:
  - did: "did:example:governance:owner-1"
  - did: "did:example:governance:owner-2"
```

### 5.2 Policy Versioning

- Every policy is versioned. Changes produce a new policy version.
- Every policy change is signed by the declared governance owners.
- Policy changes are recorded on the contract ledger per specification 03.
- The signed policy and its ledger entry together constitute evidence of governance.

### 5.3 Policy Discovery

The runtime authorization layer resolves the applicable policy for an agent at every check. Policy caching is permitted for performance, but the cache TTL MUST NOT exceed one minute, and policy changes MUST propagate to all enforcement points within the declared propagation SLA (typically 30 seconds).

---

## 6. Enforcement Layers

Agentic guardrails are enforced at **four layers**. All four layers MUST be present for conformance. Any single layer is defense-in-depth; the combination is governance.

### 6.1 Soft Enforcement — Agent Context File

The agent's own `CLAUDE.md` (specification 05) describes the guardrails the agent is expected to follow. The agent reads this on task start and is expected to comply.

**Characteristics:**
- Readable by the agent.
- Influences the agent's behavior directly.
- NOT sufficient on its own — agents may be induced (by prompt injection, by flawed reasoning, by corrupted context) to deviate.

### 6.2 CI Enforcement — Build-Time Validation

CI validates prohibited operations before code merges. CI-layer enforcement catches attempts to commit secrets, modify CI configuration, or touch protected files.

**Characteristics:**
- Deterministic.
- Blocks merge when violations are detected.
- Effective only for operations visible in code changes. Runtime operations require runtime enforcement.

### 6.3 Runtime Enforcement — SDK and Gateway Checks

The platform runtime evaluates every mutating operation against the applicable agent policy before producing a contract offer.

**Flow:**
1. Agent invokes an SDK method.
2. SDK constructs the delegation envelope including the agent's identity and the target operation.
3. Gateway receives the command at Review stage (specification 03).
4. Gateway resolves the agent's policy.
5. Gateway classifies the operation into Allowed / Approval-Required / Prohibited.
6. Gateway produces the appropriate response per Section 7.

### 6.4 Ledger Enforcement — Post-Hoc Detection

Prohibited operations that somehow reach execution are detected by the governance audit service, which reads the contract ledger (specification 03) and compares executed operations against policy. Violations produce:

- Immutable violation records in the audit ledger.
- Escalation to the declared governance owner.
- Automatic revocation of the agent's identity if configured (specification 09).
- Regulatory reporting per the tenant's compliance requirements.

Ledger enforcement is a detective control. It catches what runtime enforcement missed. A platform that relies solely on ledger enforcement (as its runtime enforcement is absent or weak) is non-conformant.

---

## 7. Runtime Responses by Classification

### 7.1 Allowed

The Gateway offers the contract to the executor, the executor accepts, the contract is recorded, and execution proceeds per specification 03. No additional human step is inserted.

### 7.2 Approval-Required

The Gateway offers the contract with a `status: "pending_approval"` state. The contract is recorded with an acceptance pending human approval. The approval request is routed to the configured approvers per the policy. The agent's `ContractHandle` surfaces the pending state.

**Approval states:**
- `pending_approval` — approval has been requested.
- `approved` — required approvers have signed; contract moves to Accept.
- `rejected` — approver declined; contract moves to Refuse with a signed rejection reason.
- `expired` — SLA elapsed without approval; contract moves to Refuse with `expired` reason.

Approval signatures are part of the contract envelope and are recorded in the ledger alongside the contract. Approvals themselves are contract events, not side-channel approvals — a rubber-stamp approval outside the ledger is non-conformant.

### 7.3 Prohibited

The Gateway refuses the contract offer at Review stage with `refusal_reason: "policy_violation"` and evidence identifying the policy and the clause violated. The refusal is signed by the Gateway, recorded in the ledger, and surfaced to the agent's `ContractHandle`.

Prohibited operations do not produce approval requests. There is no approval path for a prohibited operation. A request to change the classification of a specific operation from prohibited to approval-required requires a governance-owner policy change, not a per-request override.

---

## 8. Emergency Override

Governance systems that cannot be overridden in genuine emergencies produce brittle operations. Governance systems that can be overridden without evidence produce fraud. EPAS v1.4 requires a narrow emergency override path with maximum evidence.

### 8.1 Override Properties

An override:

- Is invoked by a named human actor with declared override authority (not by an agent, never by an agent).
- Names the specific prohibited operation being overridden.
- Names the specific justification (free text + structured incident reference).
- Has a duration bounded in minutes, not hours.
- Is automatically revoked on timeout.
- Is cryptographically signed by the override authority.
- Is recorded on the contract ledger as an **exception** entry per specification 03.
- Is reviewed after the fact by the governance owner.

### 8.2 Overrides Do Not Override Everything

Even during an active override, some operations remain prohibited:

- Modifying the contract ledger (tampering with the evidence of the override itself).
- Modifying external anchoring configuration.
- Disabling audit logging or observability.
- Modifying the override policy itself.

A governance system that allows its own evidence to be tampered with during an emergency is not a governance system.

---

## 9. Multi-Agent Coordination

Agents that dispatch sub-agents (supervisor-worker patterns) operate under combined authority. The supervisor's contract with the platform authorizes the supervisor's scope. When the supervisor dispatches a worker, the worker receives a **derived contract** whose authority is bounded by the intersection of:

- The supervisor's contract scope.
- The worker's declared capabilities.
- The operation's classification under the worker's policy.

Derived contracts are recorded in the ledger with an explicit parent reference. The governance system can therefore trace any worker action back to the supervisor's originating contract and, from there, to the human or service that authorized the supervisor.

A worker cannot elevate beyond its supervisor. A worker that encounters an operation outside its supervisor's scope produces a structured refusal per specification 03.

---

## 10. Agent Trust Accumulation

Over time, an agent accumulates behavioral history — contracts offered, contracts refused, policy violations, approvals granted, approvals rejected. EPAS v1.4 permits but does not mandate **trust accumulation**: adjusting an agent's effective classification based on observed history.

### 10.1 Trust Accumulation Properties

Trust accumulation MUST:

- Be declared explicitly in the agent's policy.
- Move operations **only from approval-required to a lower approval-count threshold**, never from approval-required to allowed, and never from prohibited to any lower tier.
- Be transparent to auditors: the agent's current effective classification MUST be queryable.
- Be reversible by the governance owner at any time.
- Be recorded in the ledger when it changes.

### 10.2 Trust Erosion

Repeated refusals, approval rejections, or violations erode trust and may raise an operation's effective classification. Trust erosion is not punitive — it is the same mechanism as trust accumulation, operating in the opposite direction.

Platforms that implement trust accumulation MUST implement trust erosion. Asymmetric implementations produce systems that get looser over time with no corrective mechanism.

---

## 11. Guardrails and the Contract Ledger

Every guardrail evaluation — allowed, approval-required, or prohibited — produces a ledger-recorded event. The contract ledger is therefore the authoritative record of:

- Every agent operation the platform considered.
- Every classification applied.
- Every approval requested and the approver's decision.
- Every refusal and the signed reason.
- Every override and the authority that invoked it.

An auditor reading the ledger can reconstruct the full governance history of every agent in the platform. This is what makes agentic governance in EPAS v1.4 distinct from policy engines in earlier architectures: governance decisions are not ephemeral, they are durable cryptographic evidence.

---

## 12. Conformance Requirements

A platform conforms to EPAS v1.4 Agentic Governance when:

1. Every agent has a signed, versioned agentic governance policy per Section 5.
2. Every agent operation is classified as allowed, approval-required, or prohibited.
3. Unclassified operations default to prohibited.
4. All four enforcement layers (soft, CI, runtime, ledger) are present.
5. Approval-required operations produce ledger-recorded approval events before execution.
6. Prohibited operations produce signed refusals with evidence pointing to the violated policy clause.
7. Emergency overrides follow the properties in Section 8.
8. Multi-agent coordination produces derived contracts with explicit parent references.
9. The human-readable `CLAUDE.md` guardrails are consistent with the runtime policy.
10. The current effective policy for every agent is queryable by auditors.

Partial conformance during migration from v1.3 is permitted and MUST be declared in `conformance.yaml`.

---

## 13. Explicit Non-Goals

Agentic Governance does not:

- Define which specific operations belong in which tier for a given platform. Classification is tenant-specific and policy-driven.
- Replace code review, architecture review, or human oversight of agent-produced changes.
- Eliminate the need for traditional security controls (authentication, authorization, network isolation, encryption).
- Substitute for the Contract-Based Trust Model. Guardrails constrain which contracts agents may offer; the trust model governs how those contracts are recorded and executed.
- Prescribe specific approval UI, notification channels, or escalation paths.

Implementation of the approval workflow UI, notification routing, and escalation is out of scope for this specification. Integration patterns (notification platforms, ticketing systems, paging) remain vendor-neutral.
