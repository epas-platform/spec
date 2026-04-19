# EPAS v1.4 — Contract-Based Trust Model

> **Status:** Draft — Normative
> This specification defines the Contract-Based Trust Model for EPAS v1.4. It specifies how work is offered, accepted, refused, recorded, executed, and proven. The model produces non-repudiable, audit-grade evidence that a specific action was requested by an authorized party, accepted by a bounded executor, and executed within agreed constraints.

---

## 1. Purpose

EPAS v1.4 replaces implicit trust with **provable agreement**. This specification defines the contract lifecycle, the contract envelope, the contract ledger, and the relationship between contracts and the event plane.

The Contract-Based Trust Model is the constitutional addition that distinguishes EPAS v1.4 from v1.3. Platforms that execute work without explicit contracts are non-conformant.

This specification is **normative**. Implementations that deviate from the rules in this document are non-conformant.

---

## 2. Core Principle

> **No work is performed without an explicit contract.**

A contract is not a formality. A contract is a cryptographic agreement between three parties:

1. **Requester** — The party expressing intent. A human user, a service, or an agent acting under delegated authority.
2. **Executor** — The agent, worker, or service that agrees to perform the work.
3. **Platform** — The system that attests to the agreement and records it in the contract ledger.

A contract that is not recorded in the ledger is not valid. An operation that lacks a contract is an unauthorized action regardless of outcome.

---

## 3. Relationship to v1.3

EPAS v1.3 did not define a contract model. v1.3 relied on:

- Zero-trust internal authentication
- Scope-based authorization (environment → organization → business unit → ...)
- Event-driven orchestration with CloudEvents
- Audit logs in the Observability Plane

EPAS v1.4 **preserves all of the above** and adds a distinct Contract Plane. Authentication proves identity; authorization proves permission; **a contract proves consent**. These are three orthogonal concerns.

Platforms that attempt to satisfy v1.4 by repurposing authorization records or event streams as contracts are non-conformant. Contracts have a distinct lifecycle, a distinct envelope, and a distinct storage discipline.

---

## 4. Contract Lifecycle

Every contract follows the same seven-stage lifecycle:

```
┌────────────┐     ┌────────────┐     ┌────────────────────┐
│  1. OFFER  │────▶│  2. REVIEW │────▶│ 3. ACCEPT / REFUSE │
└────────────┘     └────────────┘     └─────────┬──────────┘
                                                │
                                                ▼
                                      ┌──────────────────┐
                                      │  4. RECORD       │
                                      │  (ledger write)  │
                                      └─────────┬────────┘
                                                │
                                   ┌────────────┴─────────┐
                                   │                      │
                                   ▼                      ▼
                          ┌────────────────┐    ┌────────────────┐
                          │  5. EXECUTE    │    │  REFUSAL       │
                          │  (if accepted) │    │  (if refused)  │
                          └────────┬───────┘    └────────┬───────┘
                                   │                     │
                                   ▼                     │
                          ┌────────────────┐             │
                          │  6. ATTEST     │             │
                          └────────┬───────┘             │
                                   │                     │
                                   ▼                     ▼
                          ┌───────────────────────────────────┐
                          │         7. TERMINATE              │
                          │  (completed, expired, or revoked) │
                          └───────────────────────────────────┘
```

### 4.1 Stage Definitions

- **Offer** — The requester proposes a contract with declared scope, constraints, and terms. The offer is signed by the requester.
- **Review** — The platform validates the offer: identity, delegation, scope, risk tier, and environmental constraints.
- **Accept or Refuse** — The executor signs acceptance or produces a signed refusal. Both outcomes are valid and terminal with respect to the offer.
- **Record** — The platform writes the contract (whether accepted or refused) to the contract ledger. A contract that is not recorded is not valid.
- **Execute** — If the contract was accepted, tasks are dispatched under the contract. Tasks MUST carry the contract identifier.
- **Attest** — Executors and the platform produce signed attestations of outcome (success, partial success, or failure). Attestations are appended to the ledger.
- **Terminate** — The contract reaches a terminal state: completed, failed, refused, expired, or revoked.

### 4.2 Lifecycle Invariants

- A contract cannot skip the **Record** stage. Ledger write failure terminates the lifecycle and MUST reject the offer.
- A contract cannot transition from **Refuse** to **Execute**. Refusal is terminal.
- A contract that expires during **Execute** enters **Terminate** immediately; in-flight tasks MUST be cancelled.
- The same offer MAY be re-submitted after refusal, but doing so produces a new contract. Retrying the refused contract is prohibited.

---

## 5. Contract Envelope

A contract is expressed as a canonical, signed envelope. The envelope is implementation-agnostic: it MAY be serialized as JSON, Protobuf, or CBOR provided the structure and signature scheme are preserved.

### 5.1 Required Envelope Fields

```yaml
contract:
  contract_id: string                     # UUIDv7 recommended
  operation_id: string                    # UUIDv7 recommended
  schema_version: string                  # e.g., "epas.contract/1.4"

  tenant: string                          # Tenant identifier
  environment: string                     # dev | test | staging | prod
  location: string                        # Data residency region

  requester:
    did: string                           # Decentralized identifier of requester
    signature: string                     # Signature over canonical envelope

  executor:
    did: string                           # Decentralized identifier of executor
    accepted: boolean                     # true = accepted, false = refused
    signature: string                     # Signature over canonical envelope
    refusal_reason: structured | null     # Required when accepted = false

  authority:
    scope: list of strings                # Fine-grained permission strings
    risk_tier: enum                       # low | medium | high | critical
    approval: enum                        # none | single | dual | board
    delegation_chain_id: string | null    # Link to Identity & Delegation spec

  constraints:
    time_window: duration                 # e.g., "PT5M" = 5 minutes
    max_actions: integer                  # Upper bound on task count
    allowed_locations: list of strings    # Data residency restrictions
    forbidden_actions: list of strings    # Explicit denials

  terms:
    description: string                   # Human-readable intent
    expected_outcome: string              # Declared success criterion
    revocable: boolean                    # True if requester may revoke

  timestamps:
    offered_at: timestamp
    reviewed_at: timestamp | null
    decided_at: timestamp                 # Accept or refuse
    expires_at: timestamp
    terminated_at: timestamp | null

  platform_attestation:
    platform_did: string
    signature: string                     # Platform countersignature
    ledger_entry_id: string               # Points to ledger record
```

### 5.2 Canonicalization

The envelope MUST be canonicalized (deterministic field ordering, consistent numeric representation, UTF-8 encoding) before signing. All implementations MUST use the same canonicalization rules to ensure signature verification is portable across languages.

### 5.3 Signature Scheme

- Each party signs the canonicalized envelope with their private key.
- Signatures are Ed25519 by default. Other signature schemes are permitted if declared in the `schema_version` field.
- The platform countersigns after acceptance or refusal, producing the final ledger-bound artifact.

---

## 6. Refusal Semantics

Refusal is a **first-class, valid, and expected outcome**. Refusal is recorded in the ledger with the same rigor as acceptance.

### 6.1 Refusal Reasons

An executor MAY refuse a contract for any of the following reasons:

| Reason | Description |
|--------|-------------|
| `scope_mismatch` | Contract scope exceeds executor capability or job class |
| `authority_insufficient` | Delegation chain does not authorize the requested scope |
| `environment_forbidden` | Contract targets an environment the executor cannot serve |
| `location_forbidden` | Contract targets a data residency region the executor cannot serve |
| `risk_tier_exceeds_policy` | Contract risk tier exceeds executor's accepted tier |
| `constraint_violation` | Contract violates executor's declared operational constraints |
| `expired` | Contract expired before acceptance could complete |
| `platform_unavailable` | Required platform subsystems are unavailable |
| `executor_capability_mismatch` | Executor lacks the specific capability named in `authority.scope` |
| `policy_violation` | Executor's local policy prohibits this contract |

Refusal reasons MAY be extended by implementations provided extensions do not mask the categories above.

### 6.2 Refusal Envelope Fields

```yaml
refusal_reason:
  code: string                            # One of the enum values above
  message: string                         # Human-readable explanation
  evidence: structured | null             # Supporting data (e.g., policy ID)
  retryable: boolean                      # True if same offer could succeed later
  executor_signature: string              # Signs the refusal reason
```

### 6.3 Retry Prohibition

An executor that refused a contract MUST NOT be re-asked to accept the same contract. Retry requires a new contract with a new `contract_id`. Retrying a refused contract is a trust-model violation and MUST be rejected at platform ingress.

### 6.4 Refusal Visibility

Refusals are surfaced to:

- The requester, synchronously or via the requester's SDK `ContractHandle`
- The contract ledger, with full signed envelope
- The observability plane, as a structured audit event
- Tenant administrators, via their configured notification channels

---

## 7. Contract Ledger Requirements

The contract ledger is the system of record for consent. The ledger is distinct from and orthogonal to the event plane.

### 7.1 Non-Negotiable Ledger Requirements

The contract ledger **MUST**:

- Be **append-only**. No ledger entry is ever modified or deleted.
- Be **tamper-evident** via hash chaining or an equivalent cryptographic structure.
- Support **cryptographic verification** of individual entries.
- Allow **independent replay** and verification by auditors without platform cooperation.
- Provide **ordered records per tenant and per operation**.
- Support **retention and archival** aligned with regulatory obligations.
- Produce **signed tree heads** or equivalent commitments at defined intervals.
- Be **externally anchored** so that ledger integrity is verifiable against a medium the platform does not control (see Section 8.3).

The contract ledger **MUST NOT**:

- Participate in the execution path of individual tasks.
- Gate real-time task dispatch.
- Require global consensus.
- Store execution payloads or large data blobs.

### 7.2 Contract Ledger vs. Event Stream

| Property | Contract Ledger | Event Stream |
|----------|-----------------|--------------|
| Volume | Low | High |
| Purpose | Records consent | Records activity |
| Mutability | Append-only | Append-only |
| Retention | Regulatory (years) | Operational (days to weeks) |
| Verification | Cryptographic | Operational |
| Write frequency | Per contract boundary | Per state change |
| Reader | Auditors, compliance | Agents, operators, UI |

Systems that attempt to merge the ledger and the event stream are non-conformant. The two structures have incompatible durability and throughput requirements.

### 7.3 Ledger Entry Shape

Each ledger entry records a single lifecycle event for a contract:

```yaml
ledger_entry:
  ledger_entry_id: string
  contract_id: string
  entry_type: enum                        # offer | accept | refuse | attest | terminate
  contract_envelope: structured           # Full signed envelope at time of entry
  previous_hash: string                   # Hash of previous entry (hash chain)
  entry_hash: string                      # Hash of this entry
  timestamp: timestamp
  platform_signature: string
```

---

## 8. Ledger Architecture: Modeling, Substrate, and External Anchoring

EPAS v1.4 treats the contract ledger as a **composition of three concerns**, each with its own conformance requirements:

1. **Contract Modeling** — how a contract is represented semantically (the language of templates, parties, and state transitions).
2. **Ledger Substrate** — what stores the contract records durably (the persistent structure).
3. **External Anchoring** — how ledger integrity is independently verifiable against a medium the platform does not control.

Earlier revisions of this specification conflated these concerns. v1.4 separates them. A conformant platform MUST select an approach for each concern and declare the selection in `conformance.yaml`.

---

### 8.1 Contract Modeling

Contract modeling defines how obligations, parties, and permitted state transitions are expressed. Two approaches are recognized.

#### 8.1.1 Formal Contract Modeling (RECOMMENDED)

A formal contract modeling language expresses contracts as **templates** (schemas with declared parties and fields) and **choices** (authorized state transitions with explicit invariants). The language compiles to executable contract logic that the ledger substrate enforces mechanically.

**Required properties of a formal modeling language:**

- Contracts declare their **parties** explicitly (requester, executor, observers, regulators).
- Contracts declare their **signatories** — parties whose authorization is required for any state transition.
- State transitions are expressed as **choices** with declared preconditions, authorization requirements, and resulting state.
- Privacy follows from party visibility: a party sees only contracts where it is a stakeholder or declared observer.
- Contract semantics are **verifiable offline** — the same contract produces the same behavior regardless of which participant runs it.

**Why this approach is recommended:**

Formal contract modeling eliminates the class of bugs where a contract's implementation disagrees with its written terms. The model *is* the contract. Regulators and auditors can read the model and trace every permitted action back to an authorized choice. Reference implementations of this approach have been deployed to govern tens of billions of dollars in financial workflows and are recognized by financial regulators as acceptable evidence.

**Implementation note (non-normative):** Template-and-choice modeling languages designed for multi-party permissioned ledgers are the canonical reference pattern. Implementations that satisfy the required properties above are conformant regardless of the specific language selected.

#### 8.1.2 Envelope-Based Modeling (ACCEPTABLE)

An envelope-based model represents contracts as **canonical signed documents** (JSON, CBOR, or Protobuf) with explicit state transitions encoded by additional envelopes.

**Properties:**

- The envelope shape is the one defined in Section 5 of this specification.
- State transitions produce new envelopes that reference the prior envelope by hash.
- Authorization is checked at ingest by the platform, not by the ledger substrate.

**Caveats:**

- Envelope-based modeling is adequate for **single-operator** deployments where the operator is the sole source of ledger truth.
- Multi-party deployments requiring party-level privacy, cross-organization contract negotiation, or regulator-embedded observers SHOULD use formal contract modeling (8.1.1) instead.
- Envelope-based deployments require more platform-side code to enforce invariants that formal modeling enforces by construction.

---

### 8.2 Ledger Substrate

The ledger substrate persists contract records. Four substrate classes are recognized.

#### 8.2.1 Permissioned Formal-Contract Ledger (RECOMMENDED for multi-party)

A distributed ledger designed to execute formal contract models with party-level visibility and cryptographic authorization. The substrate and the modeling language are tightly coupled: the substrate enforces the modeling language's invariants natively.

**Required properties:**

- Append-only storage with deterministic transaction ordering.
- Party-level privacy: each participant's node holds only the subset of contracts for which that party is a stakeholder or observer.
- Cryptographic authorization: state transitions cannot execute without signatures from all required signatories.
- Native transaction log suitable for audit without additional derivation.
- Support for external participants (customer nodes, regulator nodes, governance nodes) operated independently.

**Why this substrate fits:**

Permissioned formal-contract ledgers are the industrial-strength pattern for multi-party governance. They are the substrate that finance, custody, and inter-enterprise settlement systems use for exactly this class of problem: parties do not trust each other fully, regulators must observe, and privacy across tenants must be mechanical rather than policy-enforced.

#### 8.2.2 Certificate-Transparency-Style Log (RECOMMENDED for single-operator)

A hash-chained append-only log with periodic signed tree heads, following the Certificate Transparency pattern.

**Properties:**

- Append-only Merkle-tree log with signed tree heads.
- Inclusion proofs for any entry against any published tree head.
- Multiple independent log operators may run parallel logs for cross-verification.
- Simple verification: given an entry and a tree head, a third party verifies inclusion with no platform cooperation.

**When to select this substrate:**

CT-style logs are the right substrate when the platform is operated by a single organization, formal multi-party contract semantics are not required, and the primary audit use case is "prove this specific contract was recorded in the sequence I think it was."

#### 8.2.3 Immutable Event Store with Explicit Hash Chaining (ACCEPTABLE)

A log-structured event store with an added hash-chaining layer.

**Properties:**

- Append-only storage with strong ordering.
- Explicit hash chain computed and stored alongside events.
- Contracts stored in a dedicated stream separate from high-volume activity events.
- Signed tree heads produced periodically.

**Caveats:**

- The platform MUST add explicit hash chaining; raw log order is insufficient cryptographic evidence.
- Contracts MUST be stored in a stream separate from the high-volume activity event stream. A shared stream is non-conformant.
- Operators MUST implement an independent verifier that reproduces the hash chain from exported data.

#### 8.2.4 Witness Network (ACCEPTABLE, composable)

A threshold-signature model where multiple independent witnesses co-sign ledger entries. Composable with any of 8.2.1–8.2.3 — a witness network adds cross-entity attestation to a primary substrate; it does not replace the substrate.

**Properties:**

- k-of-n signatures required for entry finalization.
- Witnesses operated by independent entities (auditor, regulator, customer, insurer).
- Flexible trust model tunable to industry requirements.

**Caveats:**

- Coordinating witnesses adds latency to the Record stage.
- Witness selection and rotation MUST be transparent to auditors.
- Witness compromise policy MUST be defined in advance and executable without platform cooperation.

---

### 8.3 External Anchoring

External anchoring is the mechanism by which ledger integrity is verifiable against a medium the platform does not control. **External anchoring is REQUIRED** for any substrate in Section 8.2; it is the property that distinguishes a detective control from a self-reported log.

#### 8.3.1 Anchoring Principle

A platform operator can, in principle, tamper with any substrate the operator controls. External anchoring defeats this by committing ledger roots to an independent, tamper-evident, operator-external medium at defined intervals.

The external medium does not store contracts. The external medium stores **cryptographic commitments** to contracts — Merkle roots of batched ledger entries, plus minimal metadata identifying the batch.

The contract itself remains on the platform's substrate, governed by the substrate's privacy model. No contract payload, no tenant data, and no party identity leaves the platform's control. Only hashes do.

#### 8.3.2 Anchoring Protocol

```
Ledger Entries (8.2)
        │
        ▼
┌────────────────────────┐
│ 1. Hash each entry     │
│    (canonicalize first)│
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 2. Maintain per-stream │
│    hash chain          │
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 3. Batch by time or    │
│    sequence window     │
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 4. Build Merkle tree   │
│    over batch          │
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 5. Publish Merkle root │
│    to external anchor  │
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ 6. Record the anchor's │
│    transaction back on │
│    the internal ledger │
│    (cross-reference)   │
└────────────────────────┘
```

Step 6 closes the loop: the internal ledger records the external anchor's identifier (e.g., on-chain transaction hash, block number), so any subsequent audit can trace **substrate transaction → contract event → event hash → Merkle proof → on-chain anchor** (the specific trace shape depends on the substrate class selected in Section 8.2).

#### 8.3.3 Anchoring Substrate

The external anchoring medium MUST satisfy:

- **Independence from the platform operator.** Neither the platform nor any single customer operates the anchor substrate.
- **Tamper evidence.** The anchor substrate's own integrity is cryptographic, not reputational.
- **Public verifiability.** Any third party MUST be able to verify that a given Merkle root was committed at a given time without privileged access.
- **Long-term availability.** The anchor substrate's retention horizon MUST exceed the regulatory retention horizon of the contracts it anchors.

A **public blockchain** (Ethereum or an EVM-compatible L2 is the canonical reference; equivalent public chains are acceptable) is the recommended external anchor substrate. Public blockchains satisfy all four properties by construction.

Alternative anchor substrates are permitted where equivalent properties can be demonstrated:

- Third-party notarization services with cryptographic receipts.
- Cross-signatures by a witness network (see 8.2.4) when witnesses are genuinely independent of the platform operator.
- Regulatory submission systems that produce signed timestamped receipts.

#### 8.3.4 Anchoring Granularity

Two granularities are permitted:

- **Global batches** — All contracts across tenants roll into a single Merkle tree per interval. Produces fewer on-chain transactions and a simpler narrative. Appropriate when cross-tenant event leakage is not a concern at the root level.
- **Tenant-scoped batches** — One Merkle tree per tenant (or per environment, or per regulatory domain) per interval. Produces more on-chain transactions but allows per-tenant proof packages without exposing other tenants' root structure.

Granularity selection is a cost-versus-isolation trade-off and MUST be declared in `conformance.yaml`. Changing granularity is permitted but produces a versioned anchoring scheme.

#### 8.3.5 Anchoring Interval

Anchoring intervals are tuned to balance:

- Record-stage latency (the platform does not wait for anchor confirmation; acceptance returns before anchoring completes)
- External-medium cost (per-anchor transaction cost)
- Auditor expectations (how fresh must the external commitment be to satisfy regulatory proof?)

Typical intervals are hourly (for high-frequency deployments) to daily (for lower-frequency deployments). Intervals longer than daily SHOULD be justified in `conformance.yaml`.

Anchoring is asynchronous relative to contract execution. A contract accepted at time T is executed under the contract's internal validity; its anchor commitment appears in the external medium at time T + interval. External anchoring proves that the historical record has not been rewritten; it is not on the per-contract critical path.

#### 8.3.6 Audit and Verification

A conformant platform MUST publish:

- **Export tooling** — Given a period, agent, or incident, produce a signed export of ledger entries with all fields necessary to recompute hashes.
- **Verification utilities** — Recompute per-entry hashes from the export, reconstruct Merkle trees for relevant batches, and confirm roots match those committed to the external anchor.
- **Cross-reference traces** — Given a contract, produce the full chain: substrate transaction identifier → entry → hash → Merkle path → anchor transaction.
- **Proof packages** — Given an incident or audit scope, produce a self-contained bundle including all entries, Merkle proofs, and external anchor references sufficient for offline verification.

An auditor MUST be able to verify a contract's integrity **without the platform's cooperation at verification time**, using only the auditor's copy of the export and the publicly-verifiable external anchor.

---

### 8.4 Anti-Pattern: Public Blockchain as Primary Ledger

Storing full contracts (envelopes, payloads, or party-level data) directly on a public blockchain is **non-conformant**. This anti-pattern combines the costs of a public chain with the privacy failures of a public chain and delivers none of the properties that justify the substrate classes in Section 8.2.

**Specifically:**

- Confirmation latency (seconds to minutes) is incompatible with Record-stage semantics.
- Per-entry transaction cost is operationally unsustainable at tenant scale.
- Public visibility violates tenancy and confidentiality obligations in every regulated industry.
- Global consensus is unnecessary for per-tenant contract ordering.

The correct pattern is: **substrate on-platform, anchor off-platform**. The public chain records Merkle commitments. The platform's substrate records contracts. The two together produce finance-grade governance with enterprise-grade privacy.

---

### 8.5 Selection Matrix

A conformant platform selects one modeling approach and one substrate. External anchoring is required regardless of selection.

| Deployment | Recommended Modeling | Recommended Substrate | Anchoring |
|------------|---------------------|----------------------|-----------|
| Multi-party governance (customers, regulators, platform co-sign) | 8.1.1 Formal | 8.2.1 Permissioned formal-contract ledger | 8.3, hourly or daily |
| Single operator, regulated industry | 8.1.1 Formal *or* 8.1.2 Envelope | 8.2.2 CT-style log | 8.3, hourly |
| Single operator, moderate assurance | 8.1.2 Envelope | 8.2.3 Immutable event store (dedicated stream) | 8.3, daily |
| Cross-organization attestation required in addition to platform ledger | Either | Primary substrate + 8.2.4 witness network | 8.3, hourly |

Hybrid substrates (e.g., permissioned formal-contract ledger with witness network overlay) are permitted and MUST be declared in `conformance.yaml`.

---

## 9. Throughput and Scalability

The contract ledger is sized for consent, not for activity.

### 9.1 Throughput Reality

Contracts are low-frequency, high-value artifacts. Even under aggressive automation:

- 1,000 concurrent human users
- Multi-agent supervisor systems at steady state
- Automated workflows producing derived contracts

The ledger will typically see **tens to hundreds of writes per second per tenant**, not tens of thousands. This is orders of magnitude below:

- Kafka or NATS JetStream throughput
- PostgreSQL append-only write capacity
- Merkle log implementations

Platforms that design the contract ledger for activity-volume throughput have misunderstood the model.

### 9.2 What Is Not Written to the Ledger

The ledger records **contract lifecycle boundaries only**:

- Offer
- Acceptance or refusal
- Final attestation
- Termination

The ledger **MUST NOT** record:

- Individual task events within an accepted contract
- Intermediate execution telemetry
- Tool invocations internal to an accepted contract
- Observability data of any kind

All of the above belong on the Event Plane or in the Observability Plane.

### 9.3 Write Path Latency

Expected write-path latency for the Record stage:

- CT-style log: 5–50 ms (local write, periodic head signing)
- Immutable event store with hash chain: 10–100 ms
- Witness network: 50–500 ms depending on k and n

The Record stage is on the critical path for offer-to-execute latency. SDKs MAY surface the `offered` state to callers before Record completes, but MUST NOT surface `accepted` until Record is durable.

---

## 10. Integration with the Event Plane

The contract ledger and the event plane are orthogonal but collaborate:

```
Command from SDK
   │
   ▼
┌──────────────┐
│ 1. Offer     │──────┐
│    Review    │      │
│    Accept/   │      │
│    Refuse    │      │
└──────┬───────┘      │
       │              ▼
       │     ┌─────────────────────┐
       │     │ LEDGER: record      │
       │     │ offer + acceptance  │
       │     └──────────┬──────────┘
       │                │
       │                │ (Record durable)
       ▼                │
┌──────────────┐        │
│ 2. Dispatch  │◀───────┘
│    tasks     │
└──────┬───────┘
       │
       ▼
┌──────────────────────────────────┐
│ EVENT PLANE: task.started,        │
│ task.succeeded, task.failed       │
└─────────────────┬────────────────┘
                  │
                  ▼
          ┌──────────────┐
          │ 3. Attest    │
          │    outcome   │
          └──────┬───────┘
                 │
                 ▼
          ┌─────────────────────┐
          │ LEDGER: record      │
          │ final attestation   │
          └─────────────────────┘
```

Every task event on the event plane MUST include the `contract_id` it was dispatched under. Events without a `contract_id` are defective.

---

## 11. Failure Semantics

### 11.1 Ledger Write Failure

- Ledger write failure during the Record stage **terminates the lifecycle**.
- The offer is rejected and the requester's SDK receives a `System Error` classification.
- No execution occurs.

### 11.2 Ledger Read Failure

- Ledger read failure during authority verification **pauses execution**.
- In-flight tasks complete under their existing contract.
- New dispatches within the contract are held until ledger read recovers or the contract expires.

### 11.3 Event Plane Failure

- Event plane failures do not invalidate contracts.
- Contracts remain valid for their declared time window.
- Executors MAY produce out-of-band attestations if the event plane is degraded.

### 11.4 Ledger Availability Window

The ledger MUST be highly available at **contract boundaries** (offer, accept, refuse, attest, terminate). The ledger is NOT required to be available continuously for task execution within an accepted contract.

This relaxation is what makes the ledger design tractable: the ledger is consulted tens of times per hour of agent activity, not tens of thousands of times.

---

## 12. Requester Delegation

Requesters MAY act through delegation. A requester with delegated authority produces offers whose envelopes include a `delegation_chain_id` that resolves to a signed chain of delegations from the ultimate authority down to the proximate requester.

Delegation chains are defined in specification 09 (Identity, Delegation, and Cryptographic Authority). This specification requires only that every contract offer carry a resolvable delegation chain reference when the proximate requester is not the ultimate authority.

---

## 13. Executor Refusal Capacity

Executors MUST implement a refusal path. An executor that cannot refuse is non-conformant — the ability to refuse is what distinguishes a bounded executor from a privileged rubber-stamp.

### 13.1 Mandatory Refusal Checks

Before accepting, an executor MUST verify:

1. The contract scope falls within the executor's declared job class.
2. The environment and location are permitted for the executor.
3. The authority tier is within the executor's accepted tier.
4. The contract has not expired.
5. The executor's local policy does not prohibit the contract.

Failure of any check produces a structured refusal. Silent acceptance followed by silent inaction is non-conformant.

### 13.2 Refusal Signing

Refusals are cryptographically signed by the executor with the same discipline as acceptances. An unsigned refusal is indistinguishable from an absent response and is non-conformant.

---

## 14. Contract Ledger as Detective Control

The contract ledger is a **detective control**, not a preventative control. The ledger does not prevent unauthorized action — it makes unauthorized action visible, attributable, and unforgettable.

Prevention lives in authorization (the scope system), identity (the delegation chain), and executor refusal. The ledger catches what prevention missed, and makes what prevention permitted provable after the fact.

Platforms that attempt to use the ledger as a runtime gate for every task are non-conformant: they conflate prevention with detection and produce systems that fail under load.

---

## 15. Conformance Requirements

A platform conforms to EPAS v1.4 Contract-Based Trust Model when:

1. Every mutating operation is preceded by a contract offer.
2. Every contract offer receives an explicit acceptance or signed refusal.
3. Every contract is recorded in a ledger satisfying Section 7.
4. The contract modeling approach is declared (Section 8.1) and one of 8.1.1 or 8.1.2 is selected.
5. The ledger substrate is declared (Section 8.2) and one of 8.2.1 through 8.2.4 is selected.
6. The ledger is externally anchored per Section 8.3 at a declared interval and granularity.
7. No contract payload, party identity, or tenant data is written to the external anchoring medium — only Merkle roots and minimal batch metadata.
8. Refusals are structured, signed, and ledger-recorded.
9. Event-plane task records carry the originating `contract_id`.
10. Contract-plane and event-plane data are architecturally separate.
11. The platform publishes an auditor's guide and verification tooling satisfying Section 8.3.6, enabling independent replay without platform cooperation.

Partial conformance during migration from v1.3 is permitted and MUST be declared in `conformance.yaml`. External anchoring MAY begin as a best-effort scheme during migration but MUST reach production discipline (defined intervals, signed commitments, independent verifiability) before full conformance is claimed.

---

## 16. Explicit Non-Goals

The Contract-Based Trust Model does not:

- Replace the event plane.
- Replace operational databases.
- Execute logic or run workflows.
- Store task payloads or business data.
- Approve or reject changes — approval is an upstream concern that produces offers.
- Enforce policy — policy enforcement happens at review and in executor refusal.

The ledger exists solely to prove agreement and outcome. Mission scope beyond that is a specification defect to be resolved by amendment.

---

## 17. Summary

EPAS v1.4 Contract-Based Trust Model replaces implicit trust with provable agreement. Agents do not act without consent. Platforms do not execute without evidence. Auditors do not infer intent.

The contract ledger is not a performance bottleneck. It is a court record.
