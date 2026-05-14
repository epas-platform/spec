# EPAS v2.1 - Profile Bundles

> **Status:** Draft - Normative candidate
> This document defines the initial required named bundle set for EPAS v2.1 profile adoption.

---

## 1. Purpose

Profile bundles are reusable compositions of the EPAS v2.1 profile matrix.

The matrix remains the source of truth. A bundle is only valid if it can be reduced to the mandatory matrix axes and optional traits or overlays defined in [01-profile-matrix.md](./01-profile-matrix.md).

The canonical machine-readable draft catalog is [profile-catalog.draft.json](./profile-catalog.draft.json).

---

## 2. Bundle Rules

Every required bundle MUST declare:

- `bundle_id`
- `status`
- `description`
- `composed_from.lifecycle`
- `composed_from.audience`
- `composed_from.environment`
- `composed_from.governance`
- `composed_from.infrastructure`
- `composed_from.data_model`
- `composed_from.delivery`
- `composed_from.architecture`
- `reference_stack_family`

Bundles MAY declare architecture traits and overlays where they refine the canonical matrix values.

Bundles MUST NOT waive EPAS v2.0 constitutional principles. Bundles MAY define lower control realization for prototypes and demos only where the matrix permits it.

---

## 3. Initial Required Bundle Set

The v2.1 draft catalog starts with ten bundles. They cover the common progression from internal prototype to customer production across relational and entity-native systems, plus partner and regulated variants.

| Bundle | Lifecycle | Audience | Data Model | Reference Stack |
|--------|-----------|----------|------------|-----------------|
| `relational_internal_prototype` | `prototype` | `internal_tool` | `relational_operational` | `relational_internal_platform` |
| `relational_internal_demo` | `demo` | `internal_platform` | `relational_operational` | `relational_internal_platform` |
| `relational_customer_mvp` | `mvp` | `customer_facing` | `relational_operational` | `customer_product_standard` |
| `relational_customer_production` | `production` | `customer_facing` | `relational_operational` | `customer_product_standard` |
| `entity_native_internal_prototype` | `prototype` | `internal_platform` | `entity_relationship_native` | `entity_native_agentic_prototype` |
| `entity_native_internal_demo` | `demo` | `internal_platform` | `entity_relationship_native` | `entity_native_agentic_prototype` |
| `entity_native_customer_mvp` | `mvp` | `customer_facing` | `entity_relationship_native` | `entity_native_agentic_mvp` |
| `entity_native_customer_production` | `production` | `customer_facing` | `entity_relationship_native` | `customer_product_standard` |
| `partner_embedded_mvp` | `mvp` | `partner_embedded` | `relational_operational` | `customer_product_standard` |
| `regulated_production_assured` | `production` | `customer_facing` | `hybrid_multimodal` | `regulated_production_assured` |

---

## 4. Drafting Notes

The first starter implementation may expose friendlier bundle IDs such as `relational-prototype` or `entity-native-demo`. Those starter IDs are aliases. The canonical EPAS catalog IDs use matrix-bearing names so that the audience, lifecycle, and dominant data model are visible without opening the bundle.

Future catalog work SHOULD add:

- explicit alias mapping between starter IDs and canonical catalog IDs
- a dedicated relational prototype reference stack, or a note explaining why `relational_internal_platform` remains sufficient
- partner-specific reference-stack guidance
- an edge-specific bundle family for offline and field deployments
- CI fixtures that validate every catalog bundle against `schemas/profile-catalog.schema.json`
