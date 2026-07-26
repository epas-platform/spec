# EPAS v2.1 — Profile Schemas

> **Status:** Draft — Normative
> This specification defines the machine-readable schema requirements for EPAS v2.1 profile declarations, profile catalogs, and legality rules.

---

## 1. Purpose of EPAS v2.1 Profile Schemas

EPAS v2.1 requires profiles to be human-readable and machine-validated.

The prose specifications define the meaning of profiles. The schema specifications define the machine-readable structure that implementations MUST use when declaring profiles, named bundles, and profile legality rules.

This specification ensures that:

- profile declarations are consistent across repositories
- CI can validate profile completeness and legality
- downstream tooling can reason about implementation maturity, controls, and stack choices

---

## 2. Validation Model

EPAS v2.1 uses a three-level validation model:

1. **Structural validation** - the document matches the declared schema
2. **Legality validation** - the selected profile values and control realizations are allowed together
3. **Implementation validation** - the repository contains the required implementation artifacts and the declared profile matches reality

Level 1 is schema validation.
Level 2 is policy validation.
Level 3 is conformance validation.

---

## 3. Schema Requirement

Every concept defined in the EPAS v2.1 profile system MUST have a schema representation.

EPAS v2.1 therefore requires:

- a schema for implementation profile declarations
- a schema for reusable named bundles or profile catalogs
- a schema for profile legality and cross-field validation rules
- example declarations that are valid against the schemas

Human-readable prose without a machine-readable schema is non-conformant in v2.1.

---

## 4. Canonical Files

The canonical v2.1 schema artifacts are:

| File | Purpose |
|------|---------|
| `schemas/profile-declaration.schema.json` | Schema for implementation profile declarations |
| `schemas/profile-catalog.schema.json` | Schema for reusable profile catalogs and bundles |
| `schemas/profile-rules.schema.json` | Schema for cross-field legality and policy rules |
| `schemas/profile-declaration.example.yaml` | Human-readable example declaration |

Implementations MAY generate language-native bindings from these schemas, but the JSON Schema artifacts are the source of truth.

---

## 5. Profile Declaration Document

An EPAS implementation profile declaration records four kinds of information:

1. **Document metadata** - what declaration is this
2. **Profile composition** - what matrix values define this implementation
3. **Control realization** - how required EPAS concerns are implemented at this maturity level
4. **Technology and conformance** - what stack and conformance state the implementation claims

### 5.1 Canonical Path

The canonical implementation profile declaration SHOULD live at `epas-profile.yaml` in the repository root.

Implementations MAY use an alternate path only if the alternate location is declared in `conformance.yaml` or an equivalent repository-specific conformance artifact.

### 5.2 Required Top-Level Sections

A canonical declaration MUST contain:

- `schema_version`
- `document`
- `epas`
- `profile`
- `classification`
- `principles`
- `controls`
- `technology`
- `reference_stack`
- `conformance`

Optional section:

- `extensions`

---

## 6. Profile Declaration Semantics

### 6.1 `schema_version`

`schema_version` identifies the schema and version used by the declaration.

Draft artifacts MAY use a draft suffix.

Examples:

```yaml
schema_version: "epas.profile/2.1-draft"
schema_version: "epas.profile/2.1"
```

Release-grade declarations MUST use the exact release schema identifier for the v2.1 line.

### 6.2 `document`

`document` records metadata about the declaration itself.

Required fields:

- `id`
- `title`
- `status`
- `owner`
- `updated_at`

### 6.3 `epas`

`epas` records the base EPAS version and the profile-layer version.

Required fields:

- `base_version`
- `profile_version`

### 6.4 `profile`

`profile` records the canonical matrix composition.

Required fields:

- `profile_id`
- `description`
- `composed_from`

`composed_from` MUST contain every mandatory axis from the matrix specification.
It MAY also contain `architecture_traits` and `overlays` when the implementation needs secondary architecture or overlay context.

### 6.5 `classification`

`classification` records additional summary attributes that may influence stack and control interpretation.

Required fields:

- `primary_use_case`
- `tenant_mode`
- `sensitivity`
- `edge_required`
- `offline_required`

### 6.6 `principles`

`principles` records the non-waivable v2.0 concerns and any explicit inheritance behavior.

Required fields:

- `inherited_from_epas_2_0`
- `non_waivable`

### 6.7 `controls`

`controls` records how the implementation realizes the control surface.

Required subsections:

- `authentication`
- `delegation`
- `contract_handling`
- `governance`
- `events`
- `documentation`

Each control subsection that represents a maturity choice MUST declare both a realization level and a mode.

### 6.8 `technology`

`technology` records the implementation technology choices and any alternatives.

Required subsections:

- `ui`
- `backend`
- `database`
- `sdk`
- `realtime`
- `queueing`
- `infrastructure`

Each technology subsection SHOULD include a justification when the declared choice diverges from the default reference stack for that profile family.

### 6.9 `reference_stack`

`reference_stack` records the declared recommended or selected stack composition.

Required fields:

- `family_id`
- `name`
- `components`

### 6.10 `conformance`

`conformance` records the current state and pending upgrades.

Required fields:

- `overall_state`
- `required_declarations`
- `waivers`
- `pending_controls`

---

## 7. Profile Catalog Document

EPAS v2.1 permits reusable named bundles that compose the profile matrix into common patterns.

Examples:

- `entity_native_internal_prototype`
- `customer_mvp_relational`
- `regulated_production_assured`

The profile catalog schema records:

- catalog identifier
- human-readable metadata
- bundle identifier and bundle status
- matrix composition
- optional architecture traits
- overlays
- reference stack family
- expected control posture

Named bundles MUST always remain reducible to canonical matrix values.

---

## 8. Profile Rules Document

EPAS v2.1 includes a separate schema for legality and cross-field rules because JSON Schema alone cannot express every profile constraint cleanly.

The profile rules schema records policy-style validation artifacts such as:

- production with stubbed auth is invalid
- `g3_assured` with simulated contract handling is invalid
- `entity_native_agentic` without an entity-and-relationship-native data model is invalid unless an explicit exception exists
- overlays and classification fields must align

The profile rules document is intentionally machine-readable so that CI can validate policy rules in addition to shape.

---

## 9. Validation Rules

CI MUST validate all of the following:

1. every required top-level section exists
2. every mandatory matrix axis is declared
3. every axis value is from the approved enum
4. control declarations are consistent with the matrix
5. technology declarations are consistent with the profile family where required
6. known-illegal combinations are rejected
7. the declaration and its conformance record agree

Examples of schema-level or policy-level invalid states:

- `environment: production` with `authentication.mode: stubbed`
- `governance: g3_assured` with `contract_handling.realization: simulated`
- `data_model: entity_relationship_native` with an explicit stack that forbids first-class relationships
- `overlay: entity_native_agentic` with `data_model: relational_operational` and no explicit exception

---

## 10. YAML and JSON Relationship

EPAS v2.1 declarations MAY be authored in YAML for readability.

EPAS tooling MUST support validation by one of these methods:

- YAML converted to JSON then validated against JSON Schema
- native YAML validation against a schema-equivalent toolchain

YAML comments are non-semantic and MUST NOT contain required conformance information.

The normative schema artifacts remain JSON Schema documents.

---

## 11. Schema Versioning Rules

Schema versioning follows the release train of the specification set.

- Draft documents MAY use suffixed schema identifiers.
- Release declarations MUST use the exact release identifier for the spec line being implemented.
- Schema files SHOULD preserve backward-compatible field additions within the same major/minor line.
- Breaking changes require a new specification version.

---

## 12. SurrealDB Representation

The schema model MUST explicitly support database-default selection by profile family.

The schema therefore includes:

- a primary database engine
- rationale for that default
- alternative engines
- recommended profile families for each alternative
