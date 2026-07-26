# EPAS v2.1 Specification — Profile Matrix Document Set

Enterprise Multi-Platform Architecture Specification, version 2.1.

Version 2.1 introduces a mandatory **profile layer** for all EPAS implementations. EPAS v2.0 established the constitutional architecture for AI-first, SDK-first, contract-governed platforms. EPAS v2.1 defines how an implementation declares what kind of EPAS system it is, what controls are required at its current maturity and risk level, and which reference stack patterns are appropriate.

This structure serves two readers equally: human architects who need a release-grade standards document, and AI agents that retrieve only the profile specification or schema they need.

---

## How To Use This Set

- Read [00-overview.md](./00-overview.md) first if you are new to the profile layer.
- Read [01-profile-matrix.md](./01-profile-matrix.md) when choosing or reviewing a profile.
- Read [02-profile-schemas.md](./02-profile-schemas.md) when implementing validation or generating declarations.
- Read [03-reference-stacks.md](./03-reference-stacks.md) when selecting a technology stack family.
- Read [04-migration-and-conformance.md](./04-migration-and-conformance.md) when moving an implementation into v2.1 conformance.
- Read [05-profile-bundles.md](./05-profile-bundles.md) when choosing a reusable named bundle.

---

## v2.1 Document Set

| # | Document | Purpose | Status |
|---|----------|---------|--------|
| 00 | [v2.1 Overview](./00-overview.md) | Introduces the mandatory EPAS profile layer | Draft |
| 01 | [EPAS Profile Matrix](./01-profile-matrix.md) | Defines composable profile axes and profile composition rules | Draft |
| 02 | [EPAS Profile Schemas](./02-profile-schemas.md) | Defines machine-readable schema requirements and declaration structure | Draft |
| 03 | [EPAS Reference Stacks](./03-reference-stacks.md) | Defines recommended implementation stacks by profile family | Draft |
| 04 | [EPAS Migration and Conformance](./04-migration-and-conformance.md) | Defines migration and conformance rules for profile adoption | Draft |
| 05 | [EPAS Profile Bundles](./05-profile-bundles.md) | Defines the initial required named bundle catalog | Draft |

## v2.1 Schema Artifacts

| File | Purpose |
|------|---------|
| [schemas/profile-declaration.schema.json](./schemas/profile-declaration.schema.json) | JSON Schema for implementation-level profile declarations |
| [schemas/profile-catalog.schema.json](./schemas/profile-catalog.schema.json) | JSON Schema for reusable profile catalogs and named bundles |
| [schemas/profile-rules.schema.json](./schemas/profile-rules.schema.json) | JSON Schema for profile legality and cross-field validation rules |
| [schemas/profile-declaration.example.yaml](./schemas/profile-declaration.example.yaml) | Human-readable example declaration |
| [profile-catalog.draft.json](./profile-catalog.draft.json) | Draft catalog of required reusable profile bundles |

---

## v2.1 Positioning

EPAS v2.1 does not replace EPAS v2.0. EPAS v2.1 adds a required application layer on top of EPAS v2.0.

EPAS v2.0 answered:

- What principles define an EPAS platform?
- What architecture is conformant?
- What trust, identity, SDK, governance, and event rules are mandatory?

EPAS v2.1 answers:

- What kind of EPAS implementation is this?
- What controls are mandatory for that implementation profile?
- Which controls may be relaxed in prototype or demo stages without abandoning EPAS architecture?
- Which reference stacks are recommended for agent-centric, entity-native, conventional enterprise, internal, customer-facing, and regulated systems?

---

## v2.1 Does Not Do

EPAS v2.1 does not:

- replace the EPAS v2.0 contract model
- make prototypes production-conformant by default
- require one universal technology stack
- eliminate the need for a separate `conformance.yaml`

EPAS v2.1 exists to classify implementations honestly, not to flatten all implementations into one maturity baseline.

---

## Core v2.1 Rule

Every EPAS implementation MUST declare a profile.

An implementation without a declared profile is non-conformant in v2.1, even if the implementation otherwise attempts to satisfy the v2.0 principles and tenets.

---

## Relationship to v2.0

EPAS v2.1 is subordinate to EPAS v2.0 constitutional principles. A profile MAY tailor controls, reference stacks, and maturity expectations. A profile MUST NOT waive the governing v2.0 principles.

The JSON Schema files in `2.1/schemas/` are normative for machine validation. The prose documents define semantics; the schemas define structure.

Examples:

- A prototype profile MAY use stubbed authentication.
- A prototype profile MUST still preserve SDK-first boundaries.
- A prototype profile MAY simulate contract handling.
- A prototype profile MUST still represent refusal as a first-class outcome.
- An entity-native agent-centric profile MAY default to SurrealDB.
- A conventional enterprise profile MAY default to PostgreSQL.

EPAS v2.1 profile declarations are valid only when paired with an EPAS v2.0 base conformance declaration.

---

## Cross-References

- [EPAS v2.0 Document Set](../2.0/README.md)
- [EPAS v2.0 Overview](../2.0/00-overview.md)
- [EPAS v2.0 Core Principles and Architectural Tenets](../2.0/01-core-principles-and-tenets.md)
- [EPAS v2.0 SDK-First Architecture](../2.0/02-sdk-first-architecture.md)
- [EPAS v2.0 Contract-Based Trust Model](../2.0/03-contract-based-trust-model.md)
- [EPAS v2.0 Migration and Conformance Template](../2.0/conformance.yaml.template)
