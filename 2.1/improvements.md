# EPAS v2.1 Improvements Review

## Review Scope

This review covers the EPAS v2.1 document set only.

Files reviewed:

- `2.1/README.md`
- `2.1/00-overview.md`
- `2.1/01-profile-matrix.md`
- `2.1/02-profile-schemas.md`
- `2.1/03-reference-stacks.md`
- `2.1/04-migration-and-conformance.md`
- `2.1/schemas/profile-declaration.schema.json`
- `2.1/schemas/profile-catalog.schema.json`
- `2.1/schemas/profile-declaration.example.yaml`

The review does not recommend changes to legacy root-level repository documents.

## Cross-Document Recommendations

### 1. Normalize the normative language model

The v2.1 series currently mixes `MUST`, `SHOULD`, "recommended", "default", "guidance", and "normative defaults." The next pass should define a short terminology rule in `00-overview.md` or `01-profile-matrix.md`.

Recommendation:

- `MUST` means required for conformance.
- `SHOULD` means expected unless justified in the profile declaration.
- `MAY` means permitted.
- `default reference stack` means recommended and deviation requires justification, but deviation is not automatically non-conformant.
- `normative default` should be avoided unless the document defines exactly what makes the default normative.

### 2. Add a release-wide glossary

Several terms appear across files but are not defined once in a canonical place.

Terms to define:

- profile
- profile declaration
- profile catalog
- named bundle
- overlay
- reference stack
- control realization
- entity-and-relationship-native
- entity-native agentic
- profile-aware conformance

Recommendation:

- Add a glossary section to `00-overview.md`.
- Link other documents back to that glossary instead of redefining terms.

### 3. Separate conformance requirements from implementation guidance

The current documents are directionally clear, but some sections mix hard conformance rules with practical stack advice.

Recommendation:

- Keep conformance rules in `01-profile-matrix.md`, `02-profile-schemas.md`, and `04-migration-and-conformance.md`.
- Keep technology recommendations in `03-reference-stacks.md`.
- In `03-reference-stacks.md`, explicitly label every stack item as `default`, `permitted alternative`, or `requires justification`.

### 4. Add a profile declaration lifecycle

The documents say every implementation must declare a profile, but the lifecycle of that declaration is not yet specified.

Recommendation:

- Define profile states: `draft`, `active`, `deprecated`, and possibly `superseded`.
- Define when a profile declaration must be updated.
- Define whether historical releases retain their original profile declarations or point to the current profile.

### 5. Make illegal combinations machine-checkable

The prose names illegal combinations, but the current JSON Schema cannot enforce most of them.

Recommendation:

- Add a separate `profile-rules.schema.json` or a policy validation section for cross-field constraints.
- Include initial rules for production plus stubbed auth, `g3_assured` plus simulated contracts, and `entity_native_agentic` without `entity_relationship_native` or `hybrid_multimodal`.

## File-by-File Recommendations

## `2.1/README.md`

### Current Strength

The README gives a clean entry point into the v2.1 document set and explains the relationship to v2.0 without burying the reader in implementation detail.

### Recommended Tightening

1. Add a "How to use this document set" section.

The README should tell implementers which file to read first based on their task:

- choosing a profile
- writing `epas-profile.yaml`
- selecting a stack
- migrating from v2.0
- validating conformance

2. Clarify schema status.

The README lists schema files, but it should say whether the schemas are normative artifacts in v2.1. The current answer appears to be yes.

Suggested wording:

```markdown
The JSON Schema files in `2.1/schemas/` are normative for machine validation. The prose documents define semantics; the schemas define structure.
```

3. Add release compatibility language.

The README should specify whether v2.1 requires v2.0 as a base.

Suggested wording:

```markdown
EPAS v2.1 profile declarations are valid only when paired with an EPAS v2.0 base conformance declaration.
```

4. Add a short "What v2.1 does not do" paragraph.

This would prevent readers from assuming v2.1 weakens v2.0.

Recommended content:

- v2.1 does not replace the contract model.
- v2.1 does not make prototypes production-conformant.
- v2.1 does not require one universal technology stack.

## `2.1/00-overview.md`

### Current Strength

The overview explains the motivation for profiles clearly and keeps the standard company-neutral. The SurrealDB section is now framed as a profile-family default rather than a company-specific assumption.

### Recommended Tightening

1. Fix the example profile values.

The example currently uses `audience: internal` and `environment: dev`, but the matrix schema defines `internal_tool`, `internal_platform`, `local_dev`, and `shared_dev`.

Recommendation:

- Replace `audience: internal` with `audience: internal_tool` or `audience: internal_platform`.
- Replace `environment: dev` with `environment: local_dev` or `environment: shared_dev`.

2. Add a "Profile layer vs constitutional layer" table.

The overview says v2.1 does not weaken v2.0. A table would make that much sharper.

Suggested columns:

- Concern
- v2.0 role
- v2.1 profile role
- Can a profile relax this?

3. Reframe "Supersedes" metadata.

The header says v2.1 supersedes v2.0 only in implementation profiling. "Supersedes" may be too strong because v2.1 adds a layer rather than replacing a v2.0 section.

Recommendation:

- Change to `Amends: EPAS v2.0 in the area of implementation profiling`.

4. Add a required artifact list.

The overview should list the implementation artifacts v2.1 expects:

- `epas-profile.yaml`
- `conformance.yaml`
- `llms.txt`
- agent context file

5. Add an adoption decision tree.

The overview could include a short table that maps common implementation realities to likely profiles. This would give readers a fast mental model before they enter the matrix.

## `2.1/01-profile-matrix.md`

### Current Strength

The matrix is the heart of the v2.1 release. The axes are useful, composable, and mostly DRY. The document also does a good job preserving prototype flexibility while keeping SDK-first and refusal semantics visible.

### Recommended Tightening

1. Decide whether `architecture` is truly a single-choice axis.

The current matrix says every mandatory axis gets exactly one value. In practice, `modular_monolith`, `event_driven`, and `edge_capable` can overlap. A modular monolith can be event-driven; a service-oriented system can be edge-capable.

Recommendation:

- Keep one `architecture` primary value, but add an `architecture_traits` optional array.
- Move `event_driven` and `edge_capable` to traits or overlays if they are not mutually exclusive.

2. Clarify the relationship between overlays and classification fields.

The schema also has `edge_required`, `offline_required`, `sensitivity`, and `tenant_mode`. The matrix has overlays like `edge`, `regulated`, and `customer_data_sensitive`. The relationship between these is not fully defined.

Recommendation:

- Define whether `edge_required: true` requires the `edge` overlay.
- Define whether `sensitivity: regulated` requires the `regulated` overlay.
- Define whether `customer_data_sensitive` implies specific sensitivity values.

3. Add a lifecycle-by-governance compatibility table.

The current text describes lifecycle and governance independently. A compatibility table would make the matrix more enforceable.

Example:

| Lifecycle | Minimum Governance | Notes |
|-----------|--------------------|-------|
| prototype | `g0_exploratory` | Stubbed auth allowed outside production |
| demo | `g0_exploratory` or `g1_guided` | Visible lifecycle states required |
| mvp | `g1_guided` | Real auth required |
| production | `g2_operational` | `g3_assured` for regulated systems |

4. Add a technology neutrality guardrail.

The SurrealDB default is important, but the standard should explicitly say defaults do not prohibit equivalent technology when justified.

Recommendation:

- Add a short "Reference defaults and alternatives" subsection to the data model axis.

5. Define "important state."

The phrase "everything important is modeled as entities and relationships" is useful but subjective.

Recommendation:

- Define important state as state used for authorization, audit, workflow decisions, agent memory, customer-visible behavior, or durable business records.

6. Expand illegal combinations.

The current list is good but should include:

- `environment: production` plus `governance: g0_exploratory`
- `audience: customer_facing` plus `lifecycle: production` plus `authentication.required: false`
- `overlay: regulated` plus `governance: g0_exploratory`
- `delivery: agent_first` plus missing agent identity controls beyond prototype/demo

## `2.1/02-profile-schemas.md`

### Current Strength

This document correctly states that every concept must have a schema representation. That is the right backbone for making v2.1 real rather than merely advisory.

### Recommended Tightening

1. Add explicit filenames and repository paths for implementers.

The document names canonical schema artifacts, but it should also recommend where implementations place their profile declarations.

Recommendation:

- Standardize `epas-profile.yaml` at repository root for implementation declarations.
- Allow alternative paths only when declared in `conformance.yaml`.

2. Define schema validation levels.

JSON Schema can validate structure, but cross-field rules need policy validation.

Recommendation:

- Define Level 1 validation: JSON Schema structure.
- Define Level 2 validation: profile legality rules.
- Define Level 3 validation: implementation evidence checks.

3. Add required enums to prose.

The prose describes top-level sections but does not mirror the allowed axis enums. Readers should not have to open the JSON schema to know legal values.

Recommendation:

- Link to `01-profile-matrix.md` for the canonical enum set.
- State that schema enums must match the matrix exactly.

4. Define versioning rules for schemas.

The schema pattern allows suffixes like `2.1-draft`. The prose should explain when draft suffixes are allowed and when release declarations must use `epas.profile/2.1`.

5. Clarify YAML authoring and JSON Schema validation.

The document says YAML may be converted to JSON. It should state that comments are non-semantic and must not carry required conformance information.

## `2.1/03-reference-stacks.md`

### Current Strength

The reference stack file is practical and useful. It gives implementers direct guidance while preserving the ability to justify alternatives.

### Recommended Tightening

1. Separate normative defaults from recommendations.

The header says "Guidance with Normative Defaults," but the document later says reference stacks are recommendations. This can be tightened.

Recommendation:

- Rename the status to `Guidance`.
- Add a rule that "default reference stacks are recommended defaults; deviation requires justification but is not non-conformance by itself."

2. Add an explicit stack declaration model.

The reference stacks should map directly to schema fields.

Recommendation:

- For each stack family, add a `stack_family_id`.
- Use IDs like `entity_native_agentic_prototype`, `entity_native_agentic_mvp`, `relational_internal_platform`, `customer_product`, `regulated_production`.

3. Clarify SurrealDB and PostgreSQL decision rules.

The file states default cases, but it should include a decision table.

Suggested columns:

- Dominant state shape
- Recommended primary database
- Secondary storage options
- Justification required for alternatives

4. Add "when not to use SurrealDB" guidance.

The standard will be more credible if it names boundary conditions.

Examples:

- primarily tabular reporting workloads
- teams requiring mature SQL reporting ecosystem as the core system of record
- existing enterprise data platforms where relational interoperability is load-bearing

5. Add object storage guidance.

Agent-centric and multimodal systems often need artifact storage. The stack currently mentions object storage only in hybrid examples.

Recommendation:

- Add object storage as a standard supporting component for artifacts, media, model outputs, and large payloads.

6. Add local development parity guidance.

Reference stacks should say how prototype stacks keep migration paths open.

Recommendation:

- Define the minimum Docker Compose services for the prototype profile.
- Define which services are stubs and which should already match MVP direction.

## `2.1/04-migration-and-conformance.md`

### Current Strength

The migration document is clear and pragmatic. "Truthful incompleteness is conformant. Hidden incompleteness is not." is the right spirit for profile migration.

### Recommended Tightening

1. Add a formal migration checklist.

The minimum migration steps are good. They should become a checklist implementers can copy into issues or PRs.

2. Define conformance states more precisely.

The document uses `partial`, but should define `conformant`, `partial`, and `non_conformant` in the v2.1 context.

3. Define what "truthful incompleteness is conformant" means.

This phrase is strong but needs a boundary.

Recommendation:

- Truthful incompleteness is conformant only when the selected profile permits that incomplete control.
- A production profile cannot truthfully omit real authentication and remain conformant.

4. Add examples of profile upgrades.

Useful transitions:

- `prototype` to `demo`
- `demo` to `mvp`
- `mvp` to `production`
- `relational_operational` to `hybrid_multimodal`
- `g0_exploratory` to `g1_guided`

5. Specify CI expectations as MUST or SHOULD.

The file says CI SHOULD validate several items. Since v2.1 requires schema-backed declarations, at least profile declaration existence and schema validation should likely be MUST.

Recommendation:

- Make profile existence and schema validity `MUST`.
- Keep deeper evidence checks as `SHOULD` until the tooling matures.

## `2.1/schemas/profile-declaration.schema.json`

### Current Strength

The schema provides a solid first machine-readable contract for profile declarations. It already captures the major top-level sections and axis enums.

### Recommended Tightening

1. Add `$defs` for axis enums.

Axis enums are currently inline. Reusable `$defs` would reduce drift between `profile-declaration.schema.json` and `profile-catalog.schema.json`.

Recommendation:

- Define `$defs.lifecycle`, `$defs.audience`, `$defs.environment`, `$defs.governance`, `$defs.infrastructure`, `$defs.data_model`, `$defs.delivery`, `$defs.architecture`, and `$defs.overlay`.

2. Add stronger version patterns.

`base_version` and `profile_version` are arbitrary strings. They should use SemVer-compatible patterns.

3. Add `format` expectations for identifiers.

Fields like `profile_id`, `document.id`, and `reference_stack.name` should define allowed characters.

Recommendation:

- Use lowercase slug patterns where appropriate.

4. Restrict control modes.

The `modeControl.mode` field is currently any string. The prose introduces `simulated`, `lightweight`, `partial`, and `full`, while the example uses domain-specific modes like `stubbed` and `database_backed_event_log`.

Recommendation:

- Split `realization` from `mode`.
- Example: `realization: simulated | lightweight | partial | full`, plus `mode: stubbed | oidc | database_backed_event_log | ...`.

5. Add cross-field constraints where JSON Schema can support them.

Examples:

- If `environment` is `production`, `authentication.mode` must not be `stubbed`.
- If `governance` is `g3_assured`, `contract_handling.mode` must not be `simulated`.
- If overlay contains `entity_native_agentic`, `data_model` should be `entity_relationship_native` or `hybrid_multimodal`.

6. Add `minItems` and `uniqueItems` to more arrays.

Recommended fields:

- `required_declarations`
- `waivers`
- `pending_controls`
- `required_languages`

7. Add a `justification` field for non-default technology choices.

The prose requires justification for divergence, but the schema has no clear place to record that justification.

Recommendation:

- Add `technology.<layer>.justification` or `technology.deviations`.

## `2.1/schemas/profile-catalog.schema.json`

### Current Strength

The catalog schema supports reusable named bundles and keeps bundles reducible to the matrix.

### Recommended Tightening

1. Reuse the same enum definitions as the declaration schema.

The catalog currently allows arbitrary strings for axis values. That creates drift risk.

Recommendation:

- Use the same `$defs` enum structure as `profile-declaration.schema.json`.

2. Add catalog metadata.

Recommended fields:

- `version`
- `status`
- `owner`
- `updated_at`

3. Add bundle status.

Named bundles may evolve. Add `status: draft | active | deprecated`.

4. Add expected minimum controls.

Bundles should be able to declare minimum governance, auth, contract, and event expectations.

5. Add reference stack linkage rules.

`reference_stack_family` is currently just a string. It should either be enum-backed or tied to stack IDs defined in `03-reference-stacks.md`.

## `2.1/schemas/profile-declaration.example.yaml`

### Current Strength

The example is readable and does a good job showing how a prototype can remain EPAS-shaped while relaxing full identity and contract handling.

### Recommended Tightening

1. Align the example with final schema terminology.

If the schema adds `realization`, the example should distinguish:

- `realization: simulated`
- `mode: stubbed`

2. Add comments explaining why controls are relaxed.

The example would be more useful with short comments on why `stubbed` auth and no delegation are acceptable only for an internal prototype outside production.

3. Add a stack deviation example.

A second short example or commented block could show how an implementer justifies using PostgreSQL instead of SurrealDB in an entity-native profile.

4. Include every mandatory axis in `required_declarations`.

The current example lists only six required declarations. It should include all mandatory axes:

- lifecycle
- audience
- environment
- governance
- infrastructure
- data_model
- delivery
- architecture

5. Add a pending control for real event-plane migration.

The example uses DB-backed events. If the target path is NATS JetStream for MVP or production, list that pending control explicitly.

## Suggested Next Pass Order

1. Fix enum/value mismatches in `00-overview.md` and `profile-declaration.example.yaml`.
2. Decide whether `architecture` is single-choice or should be split into primary architecture plus traits.
3. Add validation-level language to `02-profile-schemas.md`.
4. Strengthen schema enums and add reusable `$defs`.
5. Add reference stack IDs to `03-reference-stacks.md`.
6. Add profile/conformance state definitions to `04-migration-and-conformance.md`.
7. Add cross-field legality rules in prose first, then schema or policy validation.

## Release Readiness Assessment

The v2.1 draft is structurally strong and pointed in the right direction. The biggest remaining work is not content volume; it is precision.

The highest-impact tightening items are:

- align examples with schema enums
- remove ambiguity around normative defaults
- make cross-field profile legality enforceable
- clarify architecture axis composition
- add a schema-backed place for stack deviation justifications

After those changes, the document set should be much closer to a release-candidate draft for EPAS v2.1.
