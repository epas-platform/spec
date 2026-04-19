# ADR 001 — SemVer Correction: v1.4 Working Title to v2.0 Release Label

| Field | Value |
|-------|-------|
| **Status** | Accepted |
| **Date** | 2026-04-19 |
| **Deciders** | Document Owner (Nate Walker) |
| **Supersedes** | N/A |
| **Superseded by** | N/A |

---

## Context

The next EPAS release was drafted under the working title "v1.4". The working title followed the cadence of prior minor-version bumps (v1.0 → v1.1 → v1.2 → v1.3) and matched the existing planning artifacts in `1.4 Planning/` (update plan, edge/NATS/Flutter addendum, `CLAUDE.md`).

During drafting, the specification set was authored with explicit breaking-change language. Multiple "non-conformant" declarations mark behaviors that were permissible under v1.3 and are no longer permissible in the new release:

- **Specification 09 (Identity and Delegation)** declares platforms that rely on bearer tokens as proof of authority non-conformant.
- **Specification 02 (SDK-First Architecture)** declares clients that consume REST or GraphQL APIs directly non-conformant.
- **Specification 03 (Contract-Based Trust Model)** declares mutating operations without contracts non-conformant.
- **Specification 08 (Event-Driven Architecture)** declares events without a `contractid` CloudEvents extension attribute non-conformant.

The specification's own overview (specification 00) describes the release as "not an incremental revision" but "a constitutional shift."

Semantic Versioning (`https://semver.org/`) defines:

> Given a version number MAJOR.MINOR.PATCH, increment the:
> 1. MAJOR version when you make incompatible API changes
> 2. MINOR version when you add functionality in a backward compatible manner
> 3. PATCH version when you make backward compatible bug fixes

A release that marks previously-conformant patterns as non-conformant is, by definition, an incompatible change. SemVer requires a MAJOR bump.

---

## Decision

The release previously labeled **v1.4.0** is relabeled **v2.0.0**.

The rename applies to:

- The specification directory: `1.4/` → `2.0/`.
- The monolithic scaffold file: `Enterprise_MultiPlatform_Architecture_Scaffold_v1.4.md` → `Enterprise_MultiPlatform_Architecture_Scaffold_v2.0.md`.
- Every prose reference to "v1.4" inside the specification set is updated to "v2.0".
- Schema version identifiers embedded in envelopes are updated: `epas.contract/1.4` → `epas.contract/2.0`, `epas.delegation/1.4` → `epas.delegation/2.0`.
- The conformance declaration template's `epas_version` field is updated: `1.4.0` → `2.0.0`.
- The Document Control version history is updated with a note identifying v2.0.0 as the first major version bump and summarizing the breaking changes.

The following items are **preserved** as historical artifacts:

- The `1.4 Planning/` directory, including `epas-v1.4-update-plan.md`, `epas-v1.4-addendum-edge-nats-flutter-fsm.md`, `CLAUDE.md`, and `llms.txt`. The directory records the planning phase under the v1.4 working title and is the archaeological record of the version label correction.
- The `epas-v1.4-*.md` filenames inside `1.4 Planning/`. The filenames are not renamed.
- References from the v2.0 specification set to `1.4 Planning/` artifacts, which preserve "v1.4" in their link text to reflect the artifact's historical title.

---

## Consequences

### Positive

- The specification's version label now accurately reflects the scope of its changes. Downstream implementers, auditors, and dependency resolvers that rely on SemVer discipline receive correct signals.
- The Document Control version history gains a clear first-major-bump marker, which simplifies future MAJOR / MINOR decisions.
- Schema version identifiers in the contract envelope and delegation envelope track the spec version, maintaining the `schemaversion` discipline specified in specification 08 § 12.1.

### Negative

- All references, links, and planning artifacts that refer to "v1.4" in external contexts (pull requests, issues, commits, Linear tickets, external documentation) become historical. External readers encountering "v1.4" need to understand it refers to the working title of what shipped as v2.0.
- The SemVer correction does not retroactively relabel v1.1 through v1.3. Earlier releases may have contained breaking changes (the v1.3 scaffold's "environment-first architecture" and the v1.2 AWS-first deployment blueprint are plausible candidates), but retroactive correction was explicitly declined. See "Related Questions" below.
- The ADR itself is a new convention in this repository. Future decisions of similar scope should follow this template. The ADR convention location is `docs/adr/NNN-short-name.md`.

### Neutral

- The `v1.4-draft` git branch is renamed to `v2.0-draft`. Commits authored under the v1.4 label remain in the branch history; their content is updated in the SemVer-correction commit rather than by rewriting history.

---

## Alternatives Considered

### Alternative 1: Keep "v1.4" as the release label

Rejected. The release cannot satisfy SemVer as v1.4 because the release contains breaking changes. Keeping v1.4 would either violate SemVer (non-conformant behavior from downstream consumers who rely on MINOR bumps being additive) or require redefining what SemVer means for this repository, which is worse.

### Alternative 2: Label the release "v1.4.0-breaking" or "v1.4.0-preview"

Rejected. Pre-release suffixes are intended for in-progress versions of a MAJOR or MINOR version, not to signal breaking changes inside a nominally-MINOR version. The suffix would be linguistic cover for a SemVer violation.

### Alternative 3: Retroactively relabel earlier versions

Rejected. The repository's prior releases (v1.0 through v1.3) are published artifacts referenced in external documentation, RFPs, and customer deliverables. Changing the past is more costly than accepting that the versioning discipline was informal before v2.0 and is now formal.

### Alternative 4: Rename `1.4 Planning/` to `2.0 Planning/`

Rejected. The directory is an accurate record of the planning phase: at the time the directory was named, the release was labeled v1.4. Renaming it would erase that fact and create confusion about why the filenames inside still say `epas-v1.4-*`. Preserving the directory as a historical artifact is more honest.

---

## Related Questions (Deferred)

- **Should the `CLAUDE.md` in `1.4 Planning/` relocate to the repository root or to `2.0/`?** Deferred. The file is the agent context for drafting the specification and continues to serve that role. Relocation is a separate decision.
- **Should the v1.2 and v1.3 releases be retroactively audited for SemVer compliance?** Deferred. Not required for the v2.0 release to ship correctly. A separate ADR may revisit if the audit is valuable.
- **Should this repository adopt `CHANGELOG.md` at the root tracking SemVer per release?** Deferred. Recommended for follow-up.

---

## Implementation

A single commit on the `v2.0-draft` branch performs the rename as a bulk mechanical change on top of the existing v1.4 drafting commits. The commit:

1. `git mv` the directory and monolithic file.
2. Bulk replace `v1.4` → `v2.0` in the spec set, excluding the preserved historical references documented above.
3. Bulk replace schema version identifiers (`epas.contract/1.4` → `epas.contract/2.0`, `epas.delegation/1.4` → `epas.delegation/2.0`).
4. Update the conformance template's `epas_version` field.
5. Update the Document Control version history to note the first major bump.
6. Add this ADR.

The branch is renamed from `v1.4-draft` to `v2.0-draft` at the same time as the commit lands.

---

## References

- [Semantic Versioning 2.0.0](https://semver.org/)
- [v1.4 Update Plan (historical)](../../1.4%20Planning/epas-v1.4-update-plan.md)
- [v1.4 Edge/NATS/Flutter Addendum (historical)](../../1.4%20Planning/epas-v1.4-addendum-edge-nats-flutter-fsm.md)
- [EPAS v2.0 Overview](../../2.0/00-overview.md)
- [EPAS v2.0 Core Principles and Tenets](../../2.0/01-core-principles-and-tenets.md)
- [EPAS v2.0 Monolithic Scaffold](../../Enterprise_MultiPlatform_Architecture_Scaffold_v2.0.md)
