# EPAS v2.0 — Identity, Delegation, and Cryptographic Authority

> **Status:** Draft — Normative
> This specification defines the identity and delegation model for EPAS v2.0 platforms. It replaces token-passthrough authentication with decentralized identifiers, explicit delegation chains, and per-request signatures. The model is designed to eliminate the confused-deputy class of vulnerabilities that pervade token-bearer authentication in agentic systems.

---

## 1. Purpose of EPAS v2.0 Identity and Delegation Specification

EPAS v2.0 requires **cryptographically provable** identity and delegation. The platform does not infer authority from network position, session token possession, or implicit delegation. Every authority claim is traceable to a signed delegation chain rooted in an accountable principal.

This specification is **normative**. Platforms that rely on bearer tokens as proof of authority are non-conformant in v2.0.

---

## 2. Identity and Delegation Relationship to Other Specifications

- **Specification 03 (Contract-Based Trust Model)** — Contracts name requesters and executors as DIDs. The delegation chain supporting a contract is referenced by `delegation_chain_id` in the contract envelope.
- **Specification 04 (API Architecture)** — Every mutating API call carries a signed delegation envelope. Endpoint Service verifies the envelope before admission.
- **Specification 06 (Agentic Governance)** — Agent identity is verified before guardrails are evaluated. An agent whose identity fails verification has zero permitted operations.
- **Specification 07 (Edge)** — Edge nodes carry workload identity tied to the same DID system as central services. Disconnection does not suspend identity verification.

---

## 3. Identity Model

### 3.1 Decentralized Identifiers (DIDs)

Every identity in an EPAS v2.0 platform is expressed as a decentralized identifier (DID) per the W3C DID specification. A DID is a URI of the form:

```
did:{method}:{method-specific-identifier}
```

Examples:

```
did:web:platform.example:users:alice
did:web:platform.example:services:gateway
did:web:platform.example:agents:agent-builder-001
did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSrGjjD...
```

### 3.2 Permitted DID Methods

EPAS v2.0 permits:

- `did:web` — canonical for platform-operated identities (users, services, agents). Resolves to a DID document served at a well-known URL.
- `did:key` — permitted for ephemeral or transient identities (short-lived delegates, one-off service accounts).
- `did:plc` or equivalent ledger-backed method — permitted where a public, cryptographically-anchored identity registry is required.

`did:web` is the canonical recommended method for platform identities. Other methods are permitted for specific use cases and MUST be declared in `conformance.yaml`.

### 3.3 DID Document Requirements

Every DID resolves to a **DID document** that declares:

- The DID itself.
- One or more public keys (`verificationMethod`) suitable for signature verification.
- The declared purpose of each key (`authentication`, `assertionMethod`, `keyAgreement`).
- Service endpoints where relevant.
- Controller identity (the DID that controls this DID — may be the DID itself for self-controlled identities).

DID documents are fetched over HTTPS and cached per the document's declared TTL. Cached DID documents MUST be re-validated on suspected compromise or on a bounded refresh interval (default 24 hours).

---

## 4. Identity Classes

EPAS v2.0 recognizes four identity classes. Each carries distinct lifecycle and trust properties.

### 4.1 Human Identity

Human identities represent individual people interacting with the platform.

**Properties:**
- DID issued at user provisioning.
- Linked to an external IdP (OIDC) for session authentication.
- Private key material held in a user-controlled credential store (password manager, hardware token, platform-managed per policy).
- Revocation on offboarding, compromise, or role change.

**Authentication flow:**
1. User authenticates to IdP (OIDC).
2. Platform exchanges IdP session for a DID-signed session credential.
3. SDK uses the signed credential to construct delegation envelopes on mutating calls.

### 4.2 Service Identity (Workload Identity)

Service identities represent platform services (Gateway, Contract Service, specific microservices) and external systems integrated with the platform.

**Properties:**
- DID issued at service provisioning.
- Private key material held in a workload identity system (SPIFFE/SPIRE, cloud IAM role federation, or equivalent).
- Automatic rotation per the workload identity system's policy.
- Revocation on service retirement or compromise.

**Authentication flow:**
1. Service obtains a workload credential (SPIFFE SVID, IAM-federated token, or equivalent).
2. Service derives or receives its DID-signed platform credential.
3. Service-to-service calls use mutual TLS anchored in workload identity.

### 4.3 Agent Identity

Agent identities represent AI agents — supervisors, workers, and utility agents.

**Properties:**
- DID issued at agent registration.
- Private key material generated and stored under a declared policy (secrets manager, hardware-attested enclave, or equivalent).
- Cryptographic enrollment per a three-tier model (Section 5).
- Revocation on decommissioning, governance violation, or compromise.

Agent identity is treated with more lifecycle rigor than service identity because agents can be created, tuned, retired, and replaced more frequently than traditional services. Section 5 defines the three-tier agent identity model.

### 4.4 Ephemeral / Delegate Identity

Ephemeral identities represent short-lived delegate capabilities — a temporary identity used to perform a scoped action on behalf of a principal.

**Properties:**
- Typically `did:key` for minimal issuance overhead.
- Lifetime bounded in minutes or hours, not days.
- Always subordinate to a longer-lived principal via explicit delegation (Section 6).
- No independent authority — authority derives entirely from the delegation chain.

Ephemeral identities are common for automation workflows, one-off tool invocations, and cross-service handoffs where a durable DID would be overkill.

---

## 5. Three-Tier Agent Identity Model

Agent identity matures through three tiers. This progression is mandatory for production-deployed agents; experimental and sandbox agents MAY operate with Tier 1 identity only.

### 5.1 Tier 1 — Registration Identity

When an agent is created, it receives:

- A DID minted under `did:web:{platform}:agents:{agent-id}`.
- A declared set of capabilities.
- Registration-time artifact hashes: configuration hash, model hash, tool manifest hash.
- A composite fingerprint derived from the above.

Tier 1 identity is sufficient to be observed but **not** to act on the platform. An agent with only Tier 1 identity cannot offer contracts.

### 5.2 Tier 2 — Operational Identity (Wallet)

When an agent has accumulated sufficient observed behavior to be distinguishable from other agents, it is enrolled into Tier 2:

- The agent generates a signing keypair (Ed25519 recommended).
- The public key is added to the agent's DID document.
- The private key is stored under the platform's declared agent-secret policy.
- An X.509 certificate may be issued that binds the DID to a legal entity namespace, for use in environments that require X.509 chains.

Tier 2 enrollment criteria are declared in the agent's governance policy (specification 06). Typical criteria:

- Minimum observed execution count.
- Behavioral consistency score above a declared threshold.
- No governance violations in the observation window.

Tier 2 identity is sufficient to **offer** contracts. Each offer is signed by the agent with its Tier 2 key.

### 5.3 Tier 3 — Scoped Authority (Contract)

Tier 3 is not a persistent identity tier — it is the authority delegation that enables a Tier 2 agent to **execute** specific contracts. A Tier 2 agent that has not received a delegation for a specific scope cannot execute in that scope, even though it can construct the offer.

Tier 3 delegations are:

- Scoped to specific actions and resources.
- Signed by the delegating authority (a human, a supervisor agent, or a service identity).
- Time-bounded.
- Revocable.
- Recorded in the contract ledger per specification 03.

The Tier 3 delegation is what the contract's `delegation_chain_id` resolves to. Without it, the agent's Tier 2 offer is refused at Review stage.

---

## 6. Delegation Envelope

A delegation envelope declares that one identity grants scoped authority to another. Delegation envelopes compose into chains; chains are what the Contract-Based Trust Model (specification 03) verifies at Review stage.

### 6.1 Delegation Envelope Fields

```yaml
delegation:
  delegation_id: string                   # UUIDv7 recommended
  schema_version: string                  # e.g., "epas.delegation/2.0"

  grantor:
    did: string                           # Who is granting authority
    signature: string                     # Signature over canonical envelope

  grantee:
    did: string                           # Who is receiving authority

  scope:
    actions: list of strings              # e.g., ["contract:offer", "tool:call"]
    resources: list of strings            # e.g., ["tenant:example-corp:agents:*"]
    conditions:                           # Additional constraints
      time_window: duration | null
      rate_limit: integer | null
      environment: list of strings

  lineage:
    parent_delegation_id: string | null   # The delegation that authorized THIS delegation
    root_authority_did: string            # The ultimate human or service authority

  timestamps:
    issued_at: timestamp
    expires_at: timestamp
    revoked_at: timestamp | null

  platform_attestation:
    platform_did: string
    signature: string
    ledger_entry_id: string               # Ledger record of the delegation
```

### 6.2 Canonicalization and Signing

The delegation envelope is canonicalized (deterministic field ordering, consistent encoding) before signing. Signatures are Ed25519 by default; other schemes are permitted with explicit `schema_version` declaration.

### 6.3 Delegation Scope Composition

A delegation MAY NOT grant more authority than the grantor itself holds. A chain `A → B → C` requires:

- The scope of `B → C` is a subset of the scope of `A → B`.
- The time window of `B → C` is within the time window of `A → B`.
- Every conditional constraint in `A → B` is preserved or narrowed in `B → C`.

Delegations that violate subset composition are invalid. Platform verification rejects them at Review stage.

---

## 7. Delegation Chain Verification

### 7.1 Chain Structure

A delegation chain is an ordered list of delegation envelopes where each envelope's `grantee` is the `grantor` of the next envelope. The chain begins with a **root authority** — a human or a service identity that holds direct authorization from the platform's authority source (tenant admin, governance owner, or equivalent).

```
Human (root) → Supervisor Agent → Worker Agent
     (delegation A→B)       (delegation B→C)
```

### 7.2 Verification Steps

When a contract is offered with a `delegation_chain_id`, the platform MUST:

1. Resolve the chain by following the `delegation_chain_id` to the root.
2. Verify each delegation envelope's signatures.
3. Verify each delegation's `grantor` DID resolves and controls the signing key.
4. Verify scope subset composition along the chain (Section 6.3).
5. Verify no delegation in the chain is expired.
6. Verify no delegation in the chain is revoked (per Section 8).
7. Verify the final `grantee` matches the contract's `requester` or `executor` DID as appropriate.
8. Verify the contract's scope is covered by the final delegation's scope.

Failure of any step produces a signed refusal per specification 03 with reason `authority_insufficient`.

### 7.3 Chain Caching

Delegation chains MAY be cached by verification services for performance. Cache TTLs:

- Never exceed the earliest expiration in the chain.
- Never exceed 60 seconds for revocation responsiveness.
- Invalidate on any revocation event received through the event plane.

---

## 8. Delegation Revocation

### 8.1 Revocation Properties

Any delegation in a chain MAY be revoked by its grantor at any time. Revocation:

- Is recorded on the contract ledger.
- Publishes a `delegation.revoked` event to the event plane.
- Takes effect immediately for all contracts dispatched after the revocation is observed.
- Does NOT automatically terminate in-flight contracts — each in-flight contract is evaluated against the revocation per tenant policy.

### 8.2 Revocation Propagation SLA

Platforms MUST observe revocation within a declared propagation SLA (default 30 seconds). Components that cache delegation chains refresh on revocation events.

### 8.3 Revocation Cannot Rewrite History

Revocation invalidates future authority. It does NOT invalidate past contracts. Contracts signed before revocation and recorded on the ledger remain valid historical records. The ledger is append-only; it cannot be rewritten to mask previously-authorized action.

---

## 9. Per-Request Signatures

EPAS v2.0 requires per-request signatures on every mutating API call. The signature:

- Covers the canonicalized request including method, URL, body, and critical headers.
- Is produced by the SDK using the caller's private key.
- Includes a nonce and timestamp to prevent replay.
- Is verified by the Endpoint Service before admission.

### 9.1 Request Signature Envelope

```yaml
request_signature:
  signer_did: string
  delegation_chain_id: string
  nonce: string
  timestamp: timestamp
  signature: string
```

### 9.2 Replay Prevention

- Signatures include a nonce unique to the request.
- Nonces are tracked by the Endpoint Service for at least the duration of the request's skew window (default 5 minutes).
- Requests with duplicate nonces are rejected.

---

## 10. Authentication Token Handling

EPAS v2.0 does not eliminate bearer tokens — OIDC session tokens, workload tokens, and similar are used for initial identity extraction. But tokens do not carry authority after that extraction.

### 10.1 Token Usage Rules

- Tokens prove **identity**, not authority.
- Tokens are discarded after identity extraction at SDK level.
- Authority is expressed solely through signed delegation envelopes.
- Pass-through of raw tokens across service boundaries is **prohibited**.

### 10.2 Confused Deputy Elimination

The confused-deputy vulnerability — where a service holding an elevated credential is induced to perform work on behalf of an unauthorized caller — is structurally prevented by per-request signing and delegation-chain verification. The credential a service holds authorizes the service to operate, not to act on any caller's behalf.

---

## 11. Cross-Tenant and Cross-Platform Identity

### 11.1 Cross-Tenant Identity

Identities are tenant-scoped by default. A user, agent, or service in tenant `example-corp` does not exist in tenant `other-corp`. Explicit cross-tenant delegations are permitted:

- A root authority in tenant A issues a delegation to an identity in tenant B.
- The cross-tenant delegation is recorded on both tenants' ledger projections.
- Cross-tenant scope is narrow and explicitly enumerated.

### 11.2 Cross-Platform Identity

EPAS v2.0 platforms MAY federate identities with other EPAS-conformant platforms through DID methods (`did:web` or ledger-backed methods) that are resolvable across platforms. Federation:

- Respects each platform's tenancy boundaries.
- Requires mutual trust agreements between platform operators.
- Uses delegation envelopes crossing platform boundaries, not shared session tokens.

---

## 12. Identity Events

The following events MUST be emitted to the event plane by EPAS v2.0 platforms:

- `identity.registered` — a new DID is minted.
- `identity.enrolled` — an agent is promoted from Tier 1 to Tier 2.
- `identity.key_rotated` — an identity's signing key is rotated.
- `identity.revoked` — an identity is revoked.
- `delegation.issued` — a new delegation is signed and recorded.
- `delegation.revoked` — a delegation is revoked.
- `signature.verification_failed` — a request signature fails verification.

These events are inputs to security monitoring, governance audit, and anomaly detection.

---

## 13. Identity and Delegation Conformance Requirements

A platform conforms to EPAS v2.0 Identity, Delegation, and Cryptographic Authority when:

1. Every identity is expressed as a DID per Section 3.
2. Every identity class (human, service, agent, ephemeral) has a defined lifecycle and key management policy.
3. Agent identity progresses through the three-tier model of Section 5.
4. Every mutating request carries a signed delegation envelope per Section 6.
5. Delegation chains are verified at Review stage per Section 7.
6. Scope subset composition is enforced along every delegation chain.
7. Revocation propagates within the declared SLA and is recorded on the event plane per Section 12.
8. Per-request signatures are required on all mutating API calls (Section 9).
9. Tokens are used only for initial identity extraction; authority is never inferred from token possession.
10. Identity events listed in Section 12 are emitted and consumed by security monitoring.

Partial conformance during migration from v1.3 is permitted and MUST be declared in `conformance.yaml`.

---

## 14. Identity and Delegation Non-Goals

This specification does not:

- Mandate a specific DID method beyond the permitted set in Section 3.2.
- Mandate a specific signature scheme beyond Ed25519 as the default.
- Specify the operational details of a workload identity system.
- Replace operational secret management (key generation, storage, rotation) specifications.
- Define specific OIDC providers or IdP integration patterns.

Selection of specific DID methods, signature schemes, workload identity systems, and IdPs is tenant-specific and out of scope.
