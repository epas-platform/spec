# EPAS — Enterprise Platform Architecture Specification

> A proven enterprise blueprint for building scalable, reliable, performative, and cost-efficient SaaS platforms.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](./LICENSE)
[![Spec: v2.0](https://img.shields.io/badge/spec-v2.0-green.svg)](./2.0/)
[![Status: Published](https://img.shields.io/badge/status-published-success.svg)](./2.0/00-overview.md)

Open-source · vendor-neutral · machine-checkable conformance.

---

## What This Is

EPAS is a normative, version-controlled architecture specification for enterprise SaaS platforms. It codifies the patterns, controls, and decisions required to ship a multi-tenant, multi-region, AI-capable platform that enterprises will buy and auditors will accept.

The specification is **vendor-neutral** in spec text — no cloud, language, or model is privileged — and **mechanically enforced**: every architectural tenet is validated at compile / CI / deploy / runtime. Controls that live only in documentation are non-conformant.

## Why It Exists

Cloud Well-Architected frameworks were written for stateless web services. AI governance documents were written by lawyers. The result in most organizations is a stitched-together stack of cloud pillars, vendor patterns, and bolt-on compliance — coherent in slides, brittle in production, expensive in audit.

EPAS is the integrated specification that closes the gap. One document set covers tenancy, identity, APIs, events, observability, FinOps, AI governance, security, compliance, DR/BC, ITIL operations, and developer experience.

## Releases

| Version | Status | Location |
|---|---|---|
| v2.1 | Draft release branch | [`2.1/`](./2.1/) |
| v2.0 | Published spec set (canonical) | [`2.0/`](./2.0/) |
| v1.3 | Production-ready scaffold | [`Enterprise_MultiPlatform_Architecture_Scaffold.md`](./Enterprise_MultiPlatform_Architecture_Scaffold.md) |

## The Four Pillars

### Scalable
- Environment-first hierarchy: `environment → org → BU → team → project → resource`
- Multi-tenant by design; logical or physical isolation per tenant tier
- SDK-first consumption eliminates per-client integration drift across UI, CLI, automation, and agents
- Event-driven core; NATS JetStream primary, Kafka / Redpanda permitted with declared justification
- Edge-capable: formal Edge Node class with leaf-node topology for distributed, offline-tolerant deployments

### Reliable
- Field-validated SLAs: p95 1000–2000ms by tier, 99.95–99.995% uptime
- Disaster recovery: RPO 15-minute, RTO 4-hour, multi-region replication, quarterly drills
- ITIL v4 alignment across Incident (P0–P4), Change (CAB / ECAB), and Service Management
- Machine-readable conformance: ~35 architectural tenets validated in CI on every PR

### Performative
- OpenTelemetry + LGTM observability with per-entity latency budgets
- Event substrate selection criteria account for latency, retention, and ecosystem fit — not opinion
- Capacity planning and scaling triggers specified per workload class
- Developer experience targets measured: <60 min time-to-first-commit, <4 hr time-to-productive

### Cost-Efficient
- A2A cost attribution with per-agent token tracking and BU chargeback
- Real-time budget alerts at 75% warning / 95% critical thresholds
- Multi-model AI strategy: 8 providers with cost-based routing and declared fallback
- Reference workload achieves 24.5% optimization through caching + routing — full cost model published

## Built-In, Not Bolted On

- **Security:** TLS 1.3, AES-256, mTLS, zero-trust internal calls, DID-based identity with signed delegation, SBOM, supply-chain provenance
- **Compliance:** HIPAA, GDPR, EU AI Act, BIPA, ISO/IEC 42001, NIST AI RMF; vendor management aligned to GDPR Article 28 / SOC 2 / ISO 27001
- **AI Governance:** agent operations classified as allowed / approval-required / prohibited; enforced at four layers; non-repudiable audit trail via append-only contract ledger
- **Developer Experience:** one-command setup, LocalStack, mkcert, hot-reload, language-native SDKs (Python / TypeScript / Go)

## Quick Start

Where to begin depends on your role:

- **Executives** — [v2.0 Overview](./2.0/00-overview.md)
- **Architects** — [Core Principles & Tenets](./2.0/01-core-principles-and-tenets.md) → [SDK-First](./2.0/02-sdk-first-architecture.md) → [Contract Trust](./2.0/03-contract-based-trust-model.md) → [API Architecture](./2.0/04-api-architecture.md)
- **Security & Compliance** — [Contract Trust Model](./2.0/03-contract-based-trust-model.md) · [Agentic Governance](./2.0/06-agentic-governance.md) · [Identity & Delegation](./2.0/09-identity-and-delegation.md)
- **Platform Engineers** — [SDK-First Architecture](./2.0/02-sdk-first-architecture.md) · [API Architecture](./2.0/04-api-architecture.md) · [Documentation for Machine Readers](./2.0/05-documentation-for-machine-readers.md)
- **Edge / Mobile Engineers** — [Edge, NATS, and Flutter Clients](./2.0/07-edge-nats-flutter.md)
- **Historical foundation** — [Enterprise Architecture Scaffold v1.3](./Enterprise_MultiPlatform_Architecture_Scaffold.md) · [Technology Index](./TECHNOLOGY_INDEX.md) (~250 technologies cataloged)

## Conformance

Any platform claiming EPAS conformance publishes a `conformance.yaml` at the repository root. Tenets are enforced across four layers:

| Layer | Mechanism |
|---|---|
| Compile-time | Type system, linter, code generation |
| CI-time | Repository validation, schema checks |
| Deploy-time | Configuration validation, manifest checks |
| Runtime | Authorization, audit, ledger |

Partial conformance during migration is permitted and explicitly declared. Drift from declared conformance is a platform incident.

Template: [`2.0/conformance.yaml.template`](./2.0/conformance.yaml.template)

## What This Is Not

Not a certification. Not legal advice. Not a vendor pitch deck. Reference implementations exist but are explicitly out of scope for normative documents.

## License

Apache 2.0 — see [`LICENSE`](./LICENSE).

## Author

**Nate Walker** — Founder, Ravenhelm. ~20 years in enterprise AI presales, solution architecture, and platform strategy at IPsoft, Amelia, SoundHound, and Quant.ai. Speaker: Gartner, Forrester, NVIDIA GTC, HIMSS, CCW.

📧 [nate@ravenhelm.co](mailto:nate@ravenhelm.co) · 🔗 [github.com/epas-platform/spec](https://github.com/epas-platform/spec)
