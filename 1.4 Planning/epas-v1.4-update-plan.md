# EPAS v1.4 Update Plan: AI-First Documentation & SDK Architecture

## Overview

Version 1.4 introduces two major themes:

1. **AI-First Documentation** - Every section considers both human and machine readers
2. **SDK-First Architecture** - All interfaces consume APIs through SDKs

These aren't additive sections - they're woven throughout the entire spec.

---

## What's New in v1.4 (Draft)

**Strategic Shifts**:
1. **Dual-Audience Documentation** - Human engineers AND AI agents as first-class consumers
2. **SDK-First Architecture** - All interfaces (CLI, MCP, integrations) consume APIs through SDKs
3. **Machine-Readable Standards** - llms.txt, CLAUDE.md, RAG-optimized markdown mandatory
4. **Agentic Governance** - Guardrails-as-code for autonomous agents
5. **LocalStack Optional** - OpenBao/MinIO as primary dev stack
6. **NATS as Primary** - Simplified from Kafka Phase 1 → NATS Phase 2

**Key Decisions**:
- Primary documentation consumer: **AI Agent** (secondary: human)
- SDK is **mandatory** - no direct API consumption from interfaces
- llms.txt **required** in all repositories
- CLAUDE.md **required** for all services
- Markdown must pass **RAG-readiness** checks

---

## Section-by-Section Amendments

### Section: Core Principles

**Add New Principle**:
```yaml
- **Machine-readable by default** – Documentation serves AI agents as primary consumers; human readability is preserved but not prioritized over agent comprehension
```

**Expand Existing**:
```yaml
# BEFORE:
- **Evidence-first architecture** – Assume every control must be proven with logs, configs, traces

# AFTER:
- **Evidence-first architecture** – Assume every control must be proven with logs, configs, traces; documentation must be retrievable by RAG systems
```

---

### Section: Architectural Tenets (v1.4)

**Add New Tenets**:
```yaml
architectural_tenets:
  # NEW: Documentation & Machine Readers
  - documentation_dual_audience           # Human + AI consumers
  - llms_txt_mandatory                    # Repository discovery for agents
  - agent_context_files_required          # CLAUDE.md, .cursorrules
  - rag_optimized_markdown                # Semantic chunking compatible
  - agentic_governance_enabled            # Guardrails-as-code

  # NEW: SDK-First Architecture
  - sdk_first_architecture                # All interfaces via SDK
  - no_direct_api_consumption             # CLI/MCP/integrations use SDK
  - openapi_sdk_generation                # SDKs generated from spec
```

---

### NEW Section: Documentation for Machine Readers

**Position**: After "Developer Experience" (Section 21) or as subsection 21.9+

#### 21.9 Documentation Strategy: Dual-Audience

**Principle**: In 2026, the primary consumer of documentation is often an AI agent (Claude Code, Cursor, Copilot) attempting to resolve issues autonomously. Human engineers are the secondary audience.

```yaml
documentation_consumers:
  primary:
    - ai_coding_agents      # Claude Code, Cursor, Copilot
    - rag_pipelines         # Vector search for knowledge retrieval
    - mcp_discovery         # Agent tool discovery

  secondary:
    - new_engineers         # Onboarding
    - support_teams         # Troubleshooting
    - auditors              # Compliance verification
```

#### 21.10 llms.txt Standard (Mandatory)

**Purpose**: Provide AI agents with a curated, token-efficient index of repository structure - the "sitemap for LLMs."

**Required Files**:
```
/llms.txt              # Concise index with key file pointers
/llms-full.txt         # Optional: Concatenated full context (for smaller repos)
```

**llms.txt Format**:
```markdown
# Project Name

> One-line description

## Quick Start
- Setup: /docs/setup.md
- Configuration: /docs/configuration.md

## Architecture
- Overview: /docs/architecture/overview.md
- Data Flow: /docs/architecture/data-flow.md
- API Reference: /docs/api/README.md

## Key Files
- Main entry: /src/main.py
- Configuration: /src/config.py
- Core models: /src/models/

## Development
- Contributing: /CONTRIBUTING.md
- Testing: /docs/testing.md

## Agent Context
- Coding rules: /CLAUDE.md
- MCP tools: /docs/mcp-tools.md
```

**Validation**: CI/CD must verify llms.txt exists and is current.

#### 21.11 Agent Context Files (Mandatory)

**Purpose**: Provide persistent memory for AI coding agents with project-specific rules.

**Required Files**:
```
/CLAUDE.md             # For Claude Code
/.cursor/rules         # For Cursor (symlink to CLAUDE.md if identical)
/.github/copilot-instructions.md  # For GitHub Copilot
```

**CLAUDE.md Structure**:
```markdown
# Project Context for AI Agents

## Project Overview
[2-3 sentence description]

## Tech Stack
- Language: Python 3.11+
- Framework: FastAPI
- Database: PostgreSQL + pgvector
- Auth: OIDC via Zitadel

## Coding Standards
- Always use async/await for I/O operations
- Use Pydantic for all data validation
- Never import directly from data layer in API routes
- All API responses use standard wrapper format

## Architecture Rules
- Services communicate via SDK, never direct API calls
- All database access through repository pattern
- Events published via transactional outbox

## Common Mistakes to Avoid
- Don't hardcode environment-specific values
- Don't skip OpenFGA permission checks
- Don't create new API endpoints without SDK methods

## Testing Requirements
- All new code requires unit tests
- Integration tests for API endpoints
- Mock LLM calls in tests (never real API)

## File Naming Conventions
- Models: `models/{entity}.py`
- Schemas: `schemas/{entity}.py`
- API routes: `api/{entity}.py`
- SDK methods: `sdk/{entity}.py`
```

**Guardrails Section** (Agentic Governance):
```markdown
## Agent Guardrails

### Allowed Operations
- Create/modify files in /src, /tests, /docs
- Run tests via `pytest`
- Run linting via `ruff`

### Prohibited Operations
- Never modify /migrations without explicit approval
- Never commit secrets or API keys
- Never modify CI/CD configuration without review
- Never delete production data

### Approval Required
- Database schema changes
- New external dependencies
- Security-related changes
- Changes to authentication/authorization
```

#### 21.12 RAG-Optimized Markdown Guidelines

**Purpose**: Ensure documentation survives vector search chunking with context intact.

**Rules**:

1. **No Orphan Pronouns**
   ```markdown
   # BAD: Loses context when chunked
   ## Configuration
   It supports multiple formats. You can configure it via environment variables.

   # GOOD: Self-contained chunk
   ## Auth Service Configuration
   The Auth Service supports multiple configuration formats. Auth Service configuration can be set via environment variables.
   ```

2. **Explicit Entity Naming in Headers**
   ```markdown
   # BAD: Ambiguous when isolated
   ## Configuration
   ## API Reference
   ## Testing

   # GOOD: Clear entity scope
   ## Auth Service Configuration
   ## Auth Service API Reference
   ## Auth Service Testing Guide
   ```

3. **Strict Header Hierarchy**
   ```markdown
   # BAD: Broken hierarchy
   # Service Overview
   ### API Endpoints     # Skipped H2!

   # GOOD: Proper nesting for recursive splitters
   # Auth Service Overview
   ## Auth Service API
   ### Auth Service API Endpoints
   ```

4. **Self-Contained Paragraphs**
   ```markdown
   # BAD: Context from previous paragraph required
   The service uses JWT tokens. They expire after 15 minutes.

   # GOOD: Each paragraph stands alone
   The Auth Service uses JWT tokens for authentication. Auth Service JWT tokens expire after 15 minutes.
   ```

5. **Front-Loaded Key Information**
   ```markdown
   # BAD: Key info buried
   After considering various options and evaluating the trade-offs between
   complexity and performance, we decided to use PostgreSQL for the database.

   # GOOD: Key info first
   PostgreSQL is the required database for all services. This decision was made
   after evaluating trade-offs between complexity and performance.
   ```

**Validation**: Add markdownlint rules or custom CI check for RAG-readiness.

---

### NEW Section: SDK-First Architecture

**Position**: New Section 22 or integrate into Architectural Overview

#### 22.1 Interface Layer Requirements

**Principle**: All interfaces consume the core API through a shared SDK. No direct API consumption.

```
┌─────────────────┐
│   Claude/AI     │
│   (via MCP)     │
└────────┬────────┘
         │
┌──────────────┐    ┌────────▼────────┐    ┌──────────────┐
│     CLI      │───▶│   Python SDK    │◀───│  n8n / Other │
│              │    │                 │    │   Services   │
└──────────────┘    └────────┬────────┘    └──────────────┘
                             │
                    ┌────────▼────────┐
                    │    MCP Server   │
                    │  (uses SDK)     │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   Core API      │
                    │   (FastAPI)     │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
┌────────▼────────┐ ┌────────▼────────┐ ┌────────▼────────┐
│   PostgreSQL    │ │     Redis       │ │   Event Bus     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

#### 22.2 Required Packages

Every service MUST provide:

| Package | Purpose | Consumers |
|---------|---------|-----------|
| `{service}-api` | Core REST service | SDK only |
| `{service}-sdk` | Python client library | CLI, MCP, integrations |
| `{service}-mcp` | MCP server (tool exposure) | AI agents |
| `{service}-cli` | Terminal interface | Human operators |

#### 22.3 SDK Design Requirements

```python
# Required SDK structure
class ServiceClient:
    """Async-first, typed client."""

    def __init__(
        self,
        base_url: str = "http://localhost:8000",
        api_key: str | None = None,
        access_token: str | None = None,
    ):
        self._session = httpx.AsyncClient(...)

        # Resource namespaces (mirrors API structure)
        self.users = UsersResource(self)
        self.agents = AgentsResource(self)
        self.tools = ToolsResource(self)

# SDK is the ONLY way to call the API from interfaces
# MCP tools use SDK:
@mcp.tool()
async def get_user(user_id: str) -> User:
    return await client.users.get(user_id)

# CLI uses SDK:
@app.command()
def get_user(user_id: str):
    user = asyncio.run(client.users.get(user_id))
    print(user.model_dump_json())
```

#### 22.4 Benefits

1. **No duplication** - Business logic in SDK, not reimplemented per interface
2. **Type safety** - Pydantic models shared between SDK and API
3. **Versioning** - SDK version tracks API version
4. **Testing** - SDK is the integration test surface
5. **Agent compatibility** - MCP tools are thin wrappers around SDK

---

### Section: Security Architecture Updates

**Add to Section 15 (Security Architecture)**:

#### 15.x Agentic Security & Guardrails

**Scope**: Define what AI agents are permitted to do autonomously.

```yaml
agentic_governance:
  # What agents can do without approval
  autonomous_operations:
    - read_documentation
    - run_tests
    - run_linters
    - create_feature_branches
    - modify_code_in_src_tests_docs

  # What requires human approval
  approval_required:
    - database_migrations
    - security_configuration_changes
    - production_deployments
    - new_external_dependencies
    - changes_to_auth_authz

  # What is prohibited
  prohibited_operations:
    - commit_secrets_or_keys
    - modify_ci_cd_without_review
    - delete_production_data
    - bypass_permission_checks
    - direct_database_access_in_prod
```

**Enforcement**:
- CLAUDE.md defines guardrails (soft enforcement)
- CI/CD validates prohibited operations (hard enforcement)
- OpenFGA checks agent permissions at runtime

---

### Section: Developer Experience Updates

**Update Section 21.1 (One-Command Setup)**:

```bash
# Clone repository
git clone https://github.com/org/service
cd service

# One-command startup
docker-compose up -d

# Verify setup
make check-health

# Agent context ready:
# - /llms.txt (repository index)
# - /CLAUDE.md (coding rules)
# - /docs/mcp-tools.md (available tools)
```

**Add to 21.8 (Developer Productivity Metrics)**:

```yaml
developer_experience_targets:
  # Existing
  time_to_first_commit: <60 minutes
  time_to_productive: <4 hours

  # NEW: Agent readiness
  llms_txt_current: true           # llms.txt reflects actual structure
  claude_md_complete: true         # CLAUDE.md has all required sections
  rag_lint_passing: true           # Markdown passes RAG-readiness checks
  agent_test_coverage: >80%        # MCP tools have test coverage
```

---

### Section: Repository Structure Updates

**Add Standard Repository Layout**:

```
service/
├── llms.txt                    # Agent discovery index (REQUIRED)
├── CLAUDE.md                   # Agent context/rules (REQUIRED)
├── README.md                   # Human-readable overview
├── .cursor/
│   └── rules                   # Cursor rules (symlink to CLAUDE.md)
│
├── packages/
│   ├── api/                    # Core FastAPI service
│   ├── sdk/                    # Python SDK
│   ├── mcp/                    # MCP server
│   ├── cli/                    # CLI tool
│   └── web/                    # Frontend (if applicable)
│
├── docs/
│   ├── architecture/           # C4 diagrams, ADRs
│   ├── api/                    # OpenAPI spec, examples
│   └── mcp-tools.md           # MCP tool documentation
│
├── docker-compose.yml
├── pyproject.toml              # Workspace root
└── .gitlab-ci.yml              # CI/CD pipeline
```

---

### Appendix Updates

**Add Appendix F: Documentation Checklist**

```yaml
documentation_checklist:
  repository_level:
    - [ ] llms.txt exists and is current
    - [ ] CLAUDE.md has all required sections
    - [ ] README.md provides human-readable overview
    - [ ] .cursor/rules exists (can symlink to CLAUDE.md)

  markdown_quality:
    - [ ] No orphan pronouns in isolation
    - [ ] Headers include entity names
    - [ ] Strict H1→H2→H3 hierarchy
    - [ ] Paragraphs are self-contained
    - [ ] Key information is front-loaded

  sdk_documentation:
    - [ ] SDK README with quick start
    - [ ] All public methods have docstrings
    - [ ] Usage examples for common operations
    - [ ] Error handling documentation

  mcp_documentation:
    - [ ] All tools documented in docs/mcp-tools.md
    - [ ] Tool parameters and return types specified
    - [ ] Usage examples for each tool
    - [ ] Guardrails/limitations documented
```

---

## Summary of Changes

| Section | Change Type | Description |
|---------|-------------|-------------|
| Core Principles | Amendment | Add "Machine-readable by default" |
| Architectural Tenets | Amendment | Add 8 new tenets for AI-first |
| 21.9-21.12 | New | Documentation for Machine Readers |
| 22 | New | SDK-First Architecture |
| 15.x | New | Agentic Security & Guardrails |
| 21.1 | Amendment | Add agent context to setup |
| 21.8 | Amendment | Add agent readiness metrics |
| Repository Structure | New | Standard layout with llms.txt, CLAUDE.md |
| Appendix F | New | Documentation checklist |

---

## Implementation Order

1. **Core Principles & Tenets** - Foundation for everything else
2. **SDK-First Architecture (Section 22)** - Structural requirement
3. **Documentation for Machine Readers (21.9-21.12)** - Standards
4. **Agentic Security (15.x)** - Governance
5. **Repository Structure** - Reference implementation
6. **Appendix F** - Validation checklist
