# EPAS v2.1 — Migration and Conformance

> **Status:** Draft — Normative
> This specification defines how EPAS implementations adopt the v2.1 profile layer and report conformance.

---

## 1. Purpose of EPAS v2.1 Migration and Conformance

EPAS v2.1 introduces a mandatory profile declaration for every implementation. Existing EPAS-aligned systems therefore need a migration path.

This specification defines:

- how an existing implementation adopts the profile layer
- how a new implementation declares a profile from the beginning
- how conformance is reported when controls are partial, simulated, lightweight, or full
- how CI distinguishes honest partiality from non-conformant ambiguity

---

## 2. Migration Principle

> **A profile declaration is required before an implementation may claim EPAS v2.1 conformance.**

The migration burden is intentionally smaller than a full architectural rewrite.

An implementation does not need to become fully production-grade to adopt v2.1. An implementation does need to become explicit about:

- what it is
- what stage it is in
- what controls are real today
- what controls are deferred

Truthful incompleteness is conformant only when the chosen profile allows that incompleteness. Hidden incompleteness is not conformant.

---

## 3. Migration Checklist

Every implementation adopting v2.1 SHOULD complete the following checklist in order:

1. inventory the current implementation reality
2. choose the closest truthful profile
3. declare the profile in `epas-profile.yaml`
4. declare the conformance state in `conformance.yaml`
5. identify any incomplete controls
6. record the target phase for each deferred control
7. select a reference stack family or justify a deviation
8. wire schema validation into CI
9. wire legality validation into CI
10. wire conformance agreement checks into CI

---

## 4. Adoption Paths

### 4.1 New Implementations

A new implementation SHOULD:

1. choose a matrix composition
2. select a reference stack family or justify a custom stack
3. create a profile declaration
4. create or update conformance state
5. validate both in CI

### 4.2 Existing v2.0 Implementations

An existing v2.0 implementation SHOULD:

1. inventory the actual system reality
2. choose the closest truthful profile
3. declare any incomplete controls explicitly
4. update conformance artifacts to reference the profile
5. plan migrations for deferred controls

---

## 5. Minimum Migration Steps

Every implementation adopting v2.1 MUST:

1. add a valid profile declaration file
2. declare all mandatory matrix axes
3. declare control posture for auth, delegation, contract handling, governance, events, and documentation
4. declare a reference stack or justify deviation
5. update conformance artifacts to reference the profile declaration

---

## 6. Relationship to `conformance.yaml`

EPAS v2.1 does not eliminate `conformance.yaml`.

EPAS v2.1 extends the conformance model so that:

- `conformance.yaml` remains the implementation-level conformance declaration
- the profile declaration becomes a required input to conformance
- CI validates that the profile and conformance documents agree

Future releases MAY merge or regenerate these artifacts differently. In v2.1 they are logically distinct.

---

## 7. Conformance States

### 7.1 `conformant`

An implementation is `conformant` when:

- it has a valid profile declaration
- it has a valid conformance declaration
- its declared controls and technology choices are internally consistent
- it does not contain known-illegal profile combinations
- its required profile-aware repository artifacts are present

### 7.2 `partial`

An implementation is `partial` when:

- it has a valid profile declaration
- it intentionally lacks one or more non-waivable-but-deferred controls for its current stage
- it lists the incomplete controls in the conformance record
- it lists a target phase for those controls

Partial conformance is honest only when the profile itself allows the current stage.

### 7.3 `non_conformant`

An implementation is `non_conformant` when:

- it lacks a profile declaration
- it declares illegal axis combinations
- it claims a control realization that contradicts the selected profile
- it claims production-like maturity while still using prototype-only shortcuts

---

## 8. Control Realization Vocabulary

EPAS v2.1 requires implementations to be explicit about control realization.

The canonical control realization vocabulary is:

- `simulated`
- `lightweight`
- `partial`
- `full`

This vocabulary is intentionally different from profile maturity. A system may be `lifecycle: prototype` and still implement some controls in `full` form. A system may be `lifecycle: mvp` and still have some controls in `partial` form.

---

## 9. CI Validation Expectations

CI MUST validate all of the following:

1. profile declaration file exists
2. profile declaration passes schema validation
3. profile legality rules pass
4. `conformance.yaml` exists where required
5. profile declaration and conformance declaration do not contradict each other
6. known-illegal combinations are rejected
7. required machine-readable files remain present

CI SHOULD additionally verify implementation evidence where that evidence is easy to inspect automatically.

---

## 10. Example Migration Patterns

### 10.1 Internal Prototype

- choose `prototype`
- choose `internal_tool` or `internal_platform`
- choose `local_dev` or `shared_dev`
- declare `g0_exploratory`
- use lightweight contract handling and stubbed auth if truthful

### 10.2 Internal Demo

- choose `demo`
- keep `sdk_first`
- add visible approval and refusal flows
- declare whether auth is real or still lightweight

### 10.3 Customer MVP

- choose `mvp`
- choose `customer_facing`
- choose at least `g1_guided` or `g2_operational`
- real auth required
- durable audit and run history required

### 10.4 Entity-Native Agent Platform

- choose `entity_relationship_native`
- overlay `entity_native_agentic`
- use SurrealDB as default reference database unless explicitly justified otherwise

### 10.5 Relational Enterprise Platform

- choose `relational_operational`
- choose `internal_platform` or `customer_facing`
- use PostgreSQL as the default reference database unless explicitly justified otherwise

---

## 11. Conformance Failure Modes

An implementation is non-conformant in v2.1 if:

- no profile is declared
- required axes are omitted
- illegal axis combinations are declared
- a production environment claims stubbed auth
- a profile claims SDK-first delivery while clients bypass the SDK
- an `entity_native_agentic` profile denies first-class relationship modeling
- the profile and conformance artifacts contradict each other

---

## 12. Recommended Repository Layout

An implementation repository SHOULD include:

- `conformance.yaml`
- `epas-profile.yaml`
- `llms.txt`
- `CLAUDE.md`
- any locally required equivalent agent context files

The exact filename for the profile declaration MAY vary if EPAS later standardizes a different path. The declaration content and schema-valid structure are mandatory in v2.1.

---

## 13. Future Evolution

EPAS v2.1 establishes the profile layer.

Future releases MAY add:

- stronger legality validation
- overlay-specific schemas
- generated conformance reports
- stack capability catalogs

Those additions should extend the v2.1 model, not replace it abruptly.

