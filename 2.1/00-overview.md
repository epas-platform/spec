# EPAS v2.1 Overview

> **Status:** Draft
> **Amends:** EPAS v2.0 in the area of implementation profiling
> **Publication format:** Document set plus schemas

---

## 1. Purpose of EPAS v2.1

EPAS v2.1 defines the **profile layer** for EPAS implementations.

EPAS v2.0 established the constitutional architecture of an EPAS platform: AI-first documentation, SDK-first consumption, contract-based execution, refusal as a first-class outcome, agentic governance, event-driven architecture, and cryptographically provable identity and delegation.

EPAS v2.1 adds a mandatory answer to a different question:

**What kind of EPAS implementation is this, and what controls are required for that kind of system?**

This distinction matters because EPAS implementations exist across a wide range of realities:

- local internal prototypes
- polished internal demos
- customer-facing MVPs
- regulated production systems
- agent-centric platforms
- conventional enterprise applications
- entity-native systems where all important state is modeled as entities with relationships

Without a profile layer, teams either overbuild prototypes to production standards or under-specify production systems with prototype-era shortcuts. EPAS v2.1 prevents both failures.

---

## 2. EPAS v2.1 Layering Model

| Concern | EPAS v2.0 Role | EPAS v2.1 Role | Can a profile relax this? |
|---------|-----------------|----------------|---------------------------|
| SDK-first consumption | Constitutional requirement | Still mandatory | No |
| Contract-based execution | Constitutional requirement | Still mandatory in the long-term model | No |
| Refusal as a first-class outcome | Constitutional requirement | Still mandatory | No |
| Identity and delegation | Constitutional requirement | Control maturity may vary | No |
| Reference stack choice | Not specified | Profile-dependent recommendation | Yes, with justification |
| Auth implementation maturity | Not specified at the profile level | Profile-dependent control realization | Yes, if the profile permits it |

---

## 3. What EPAS v2.1 Adds

EPAS v2.1 adds five mandatory capabilities:

1. **Profile declaration** — Every implementation declares its profile explicitly.
2. **Composable profile matrix** — Profiles are composed from reusable axes rather than invented ad hoc.
3. **Schema-backed declarations** — Every profile declaration is machine-readable and validated by schema.
4. **Reference stacks** — EPAS defines recommended implementation stacks for profile families.
5. **Migration path** — Implementations can move from prototype to demo to MVP to production without abandoning EPAS principles.

---

## 4. What EPAS v2.1 Does Not Change

EPAS v2.1 does not weaken the v2.0 constitutional layer.

The following remain authoritative and non-waivable:

- machine-readable by default
- SDK-first consumption
- contract-based execution as the long-term trust model
- refusal as a first-class outcome
- evidence-first architecture
- event plane mandatory
- governance and identity as required architectural concerns

Profiles may change **how fully** these controls are realized at a given stage. Profiles may not declare those concerns irrelevant.

---

## 5. Core v2.1 Principle

> **Every EPAS implementation MUST declare a profile before it declares conformance.**

EPAS v2.1 makes profiles mandatory because conformance without context is misleading.

A prototype and a regulated production system can both be “EPAS-aligned,” but they must not be held to identical maturity expectations. The prototype still needs the correct shape. The regulated production system needs the full control surface. EPAS v2.1 makes that distinction explicit and auditable.

---

## 6. Profile Categories

EPAS v2.1 defines profiles through a composable matrix of axes.

The initial mandatory axes are:

- lifecycle
- audience
- environment
- governance
- infrastructure
- data model
- delivery
- architecture

Implementations compose a profile from these axes rather than inventing one-off labels.

Example:

```yaml
profile:
  lifecycle: prototype
  audience: internal_platform
  environment: shared_dev
  governance: g0_exploratory
  infrastructure: i0_local_compose
  data_model: entity_relationship_native
  delivery: sdk_first
  architecture: modular_monolith
```

This composable structure keeps EPAS DRY and machine-checkable.

---

## 7. SurrealDB and Data Model Direction

EPAS v2.1 formally recognizes that not all EPAS systems are best modeled as conventional relational application stacks.

EPAS v2.1 therefore defines a first-class **entity-and-relationship-native** data model profile for agent-centric systems. In this profile family:

- all important state is modeled as entities and relationships
- graph-shaped workflow and memory patterns are treated as normal
- multimodal and polymorphic records are treated as native concerns
- SurrealDB is the default reference database

EPAS v2.1 also preserves a conventional enterprise application profile where PostgreSQL remains the default reference database.

EPAS v2.1 is not “SurrealDB only.” EPAS v2.1 is explicit that different profile families justify different database defaults.

---

## 8. Implementation Artifacts

Every EPAS implementation SHOULD keep the following artifacts in the repository root or in the repository location declared by conformance:

- `epas-profile.yaml`
- `conformance.yaml`
- `llms.txt`
- `CLAUDE.md` or the locally supported agent context file equivalent

The `epas-profile.yaml` file is the implementation-level declaration of profile choice and control realization. The `conformance.yaml` file is the implementation-level declaration of conformant, partial, or non-conformant status.

---

## 9. Document Set Structure

### 9.1 Profile Definition Layer

| # | Document | Purpose |
|---|----------|---------|
| 01 | Profile Matrix | Defines the composable axes and legal combinations |
| 02 | Profile Schemas | Defines machine-readable structure and validation |

### 9.2 Implementation Guidance Layer

| # | Document | Purpose |
|---|----------|---------|
| 03 | Reference Stacks | Defines recommended stacks and infrastructure patterns |
| 04 | Migration and Conformance | Defines adoption path and conformance reporting |

---

## 10. Conformance Rule

A platform conforms to EPAS v2.1 when all of the following are true:

1. The platform continues to satisfy the relevant EPAS v2.0 principles and tenets.
2. The platform declares a profile using the v2.1 schema.
3. The platform’s stated controls and technologies are consistent with that profile.
4. CI validates the profile declaration and profile-aware conformance state.

An implementation without a valid profile declaration is non-conformant in v2.1.

---

## 11. Adoption Hints

| If the implementation is... | Start with profile family... |
|-----------------------------|------------------------------|
| a local exploratory tool | `prototype` + `internal_tool` + `local_dev` |
| a polished internal demo | `demo` + `internal_platform` + `shared_dev` |
| a customer-facing product | `mvp` + `customer_facing` + `staging_uat` |
| a regulated production system | `production` + `customer_facing` or `internal_platform` + `production` |
| an entity-native agent system | `entity_relationship_native` + `entity_native_agentic` overlay |

---

## 12. Glossary

### 12.1 Profile

A profile is the canonical EPAS classification of an implementation across lifecycle, audience, environment, governance, infrastructure, data model, delivery, and architecture.

### 12.2 Profile Declaration

A profile declaration is the machine-readable artifact that records the selected profile, control realization, technology choices, and conformance state.

### 12.3 Profile Catalog

A profile catalog is a reusable collection of named bundles that map back to canonical matrix values.

### 12.4 Named Bundle

A named bundle is a derived, reusable profile composition such as a prototype or MVP package.

### 12.5 Overlay

An overlay is an optional profile modifier such as `regulated` or `edge` that refines the interpretation of a matrix composition.

### 12.6 Reference Stack

A reference stack is a recommended technology stack family for a profile.

### 12.7 Control Realization

Control realization is the declared maturity of a control, such as `simulated`, `lightweight`, `partial`, or `full`.

### 12.8 Entity-And-Relationship-Native

Entity-and-relationship-native describes systems that model important state as entities and relationships first, rather than only as tabular rows.

### 12.9 Entity-Native Agentic

Entity-native agentic is the overlay used for agent-centric systems that want the entity-and-relationship-native reference path.

### 12.10 Profile-Aware Conformance

Profile-aware conformance is the requirement that an implementation’s stated profile, stack, and controls agree with one another and with the schema.

---

## 13. Stakeholder Guide

| Role | Primary v2.1 Sections | Why It Matters |
|------|-----------------------|----------------|
| CTO / CIO | 01, 03, 04 | Choose architecture and maturity path |
| CISO | 01, 02, 04 | Evaluate when controls are simulated, lightweight, or full |
| CAIO | 01, 03 | Choose stack and governance posture for agent systems |
| Platform Engineer | 02, 03, 04 | Implement schema, stack, and CI validation |
| Product / Solutions Engineer | 01, 03 | Match product maturity to profile expectations |

---

## 14. Authorship and Control

**Document Owner:** Platform Architecture

Changes to EPAS v2.1 profile definitions, schema requirements, or reference-stack defaults require explicit review because those changes affect conformance behavior across all implementations.
