# EPAS Specification - Agent Context

## Project Overview

The Enterprise Platform Architecture Specification (EPAS) is a comprehensive guide for building enterprise-grade multi-tenant SaaS applications. EPAS defines standards for authentication, authorization, security, observability, event-driven architecture, and AI agent safety.

## Tech Stack (Specified in EPAS)

- **Language**: Python 3.11+
- **API Framework**: FastAPI
- **Frontend**: Next.js 15+ with React 19+
- **Database**: PostgreSQL with pgvector
- **Auth**: OIDC via Zitadel + OpenFGA (ReBAC)
- **Secrets**: OpenBao (HashiCorp Vault fork)
- **Events**: NATS JetStream with CloudEvents
- **Observability**: Prometheus, Loki, Tempo, Grafana

## Document Standards

### RAG-Optimized Markdown
- No orphan pronouns (always name the entity)
- Headers include entity names (e.g., "Auth Service Configuration" not "Configuration")
- Strict H1 -> H2 -> H3 hierarchy
- Self-contained paragraphs
- Key information front-loaded

### When Editing the Specification
- Maintain the existing section structure
- Use YAML code blocks for machine-readable configs
- Include ASCII diagrams for architecture visualization
- Reference section numbers when cross-linking
- Keep examples concrete and implementable

## v1.4 Update Themes

1. **AI-First Documentation** - Documentation serves AI agents as primary consumers
2. **SDK-First Architecture** - All interfaces consume APIs through SDKs
3. **Machine-Readable Standards** - llms.txt, CLAUDE.md, RAG-optimized markdown mandatory
4. **Agentic Governance** - Guardrails-as-code for autonomous agents

## Agent Guardrails

### Allowed Operations
- Edit specification sections
- Add new sections following numbering scheme
- Update diagrams and examples
- Fix typos and clarify language

### Approval Required
- Changing core principles
- Modifying security requirements
- Adding new technology mandates
- Removing existing standards

### Prohibited Operations
- Breaking the section numbering scheme
- Removing security controls
- Adding vendor-specific implementations (keep vendor-agnostic)
