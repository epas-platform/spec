# EPAS v1.4 Specification — Document Set

Enterprise Multi-Platform Architecture Specification, version 1.4.

Version 1.4 is published as a **document set**: one monolithic scaffold file (`Enterprise_MultiPlatform_Architecture_Scaffold_v1.4.md`) that indexes into a collection of self-contained normative specifications in this directory. Each specification here is authoritative for its topic; the scaffold provides narrative and cross-references.

This structure serves two readers equally: human auditors who want the full scaffold in one place, and AI agents that retrieve only the spec they need.

---

## v1.4 Document Set

Foundational specifications (constitutional layer):

| # | Document | Status |
|---|----------|--------|
| 00 | [v1.4 Overview](./00-overview.md) | Draft |
| 01 | [Core Principles and Architectural Tenets](./01-core-principles-and-tenets.md) | Draft |
| 02 | [SDK-First Architecture](./02-sdk-first-architecture.md) | Draft |
| 03 | [Contract-Based Trust Model](./03-contract-based-trust-model.md) | Draft |
| 04 | [API Architecture (Command, Query, Internal)](./04-api-architecture.md) | Draft |
| 05 | [Documentation for Machine Readers](./05-documentation-for-machine-readers.md) | Draft |
| 06 | [Agentic Governance and Guardrails](./06-agentic-governance.md) | Draft |
| 07 | [Edge, NATS, and Flutter Clients](./07-edge-nats-flutter.md) | Draft |
| 08 | [Event-Driven Architecture (v1.4 revision)](./08-event-driven-architecture.md) | Draft |
| 09 | [Identity, Delegation, and Cryptographic Authority](./09-identity-and-delegation.md) | Draft |

Amendments to v1.3 sections (applied in the monolithic scaffold):

- Section 15 — Security Architecture: adds agentic security and guardrails subsection
- Section 21 — Developer Experience: adds agent-readiness metrics and setup requirements
- Appendix F — Documentation Checklist (new)

Monolithic scaffold: [Enterprise_MultiPlatform_Architecture_Scaffold_v1.4.md](../Enterprise_MultiPlatform_Architecture_Scaffold_v1.4.md)

Conformance declaration template: [conformance.yaml.template](./conformance.yaml.template) — copy to any implementing repository root as `conformance.yaml` and fill in.

---

## v1.4 Naming and Positioning

v1.4 is the **AI-first and SDK-first** revision of EPAS. It changes what the platform is *for*, not merely what it is *made of*.

Where v1.3 asked "how do we build a multi-tenant enterprise platform," v1.4 asks "how do we build a platform whose primary consumers are AI agents, and whose primary safety property is provable consent."

The two additions that make this possible are:

1. **SDK-First Architecture** — the SDK is the product; APIs are implementation detail. Every client (UI, CLI, automation, agent) consumes the platform through the same SDK surface.
2. **Contract-Based Trust Model** — no work is performed without an explicit, signed, ledger-recorded contract. Refusal is a first-class, auditable outcome.

Everything else in v1.4 — machine-readable documentation, agentic governance, edge nodes, Flutter clients — depends on these two tentpoles.

---

## Versioning

- **v1.3.0** — November 2025, monolithic scaffold, environment-first architecture
- **v1.4.0** — In progress, document set, AI-first and SDK-first, contract-based trust

The v1.3 scaffold file remains in the repository root until v1.4 is complete. Upon completion, the v1.4 monolithic file supersedes it; the v1.3 file moves to `1.3 Archive/`.

---

## Editing This Document Set

Every specification in this directory MUST follow the RAG-optimized markdown rules defined in [05-documentation-for-machine-readers.md](./05-documentation-for-machine-readers.md):

- Headers include the entity being described, not generic words
- No orphan pronouns — restate the entity at paragraph boundaries
- Strict H1 → H2 → H3 hierarchy, no skipped levels
- Self-contained paragraphs that survive chunk boundaries
- Front-loaded key information

Violations of these rules block merge.

All specifications are vendor-neutral. Vendor-specific implementations (product names, reference architectures from adopters) MUST NOT appear in specification text. Concrete examples use generic placeholder names (`example-corp`, `tenant-foo`).

---

## Cross-References

- [v1.4 Update Plan](../1.4%20Planning/epas-v1.4-update-plan.md) — the original amendment plan that informed this document set
- [v1.4 Edge/NATS/Flutter Addendum](../1.4%20Planning/epas-v1.4-addendum-edge-nats-flutter-fsm.md) — subsumed by document 07
- [v1.3 Scaffold](../Enterprise_MultiPlatform_Architecture_Scaffold.md) — the prior monolithic spec; v1.4 supersedes on completion
