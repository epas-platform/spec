# EPAS v1.4 — Documentation for Machine Readers

> **Status:** Draft — Normative
> This specification defines the documentation standards for EPAS v1.4 platforms, with AI coding agents as the primary audience and human engineers as the secondary audience.

---

## 1. Purpose of Machine-Reader Documentation Standards

EPAS v1.4 inverts the traditional documentation audience model. In 2026, the primary consumer of developer documentation is often an **AI coding agent** attempting to resolve an issue autonomously. Human engineers are the secondary audience.

This inversion is consequential. Documentation optimized for human reading (narrative flow, implicit context, pronouns, embedded assumptions) is actively hostile to AI agents, which consume documentation through retrieval-augmented generation pipelines that chunk text into fragments. A paragraph that references "it" in a way that depends on the previous paragraph will be retrieved without its antecedent and answered incorrectly.

This specification defines the mandatory documentation standards that make EPAS platforms comprehensible to both audiences.

This specification is **normative**. Repositories that violate these standards are non-conformant.

---

## 2. Documentation Consumer Model

EPAS v1.4 recognizes two documentation consumer classes. Both are first-class.

### 2.1 Primary Consumers

- **AI coding agents** executing as an interactive assistant or as an autonomous agent.
- **Retrieval-augmented generation pipelines** that chunk documentation into vector-searchable fragments.
- **Model Context Protocol servers** that discover repositories, tools, and capabilities by reading manifests.

### 2.2 Secondary Consumers

- **New engineers** performing onboarding and learning codebases they have not seen before.
- **Support and operations teams** troubleshooting incidents.
- **Auditors and compliance reviewers** verifying controls.

Documentation SHOULD serve both audiences. Where the two audiences' needs conflict, the primary (machine) audience wins.

---

## 3. Required Files

Every EPAS v1.4 repository MUST contain the following files at the repository root or in the documented location:

| File | Location | Purpose | Consumer |
|------|----------|---------|----------|
| `llms.txt` | Repository root | Curated, token-efficient index of repository structure | AI agents, RAG pipelines |
| `CLAUDE.md` | Repository root | Persistent project context and guardrails for AI coding agents | AI coding agents |
| `README.md` | Repository root | Human-readable overview | New engineers |
| `.cursor/rules` | `.cursor/rules` | Cursor-specific agent rules (MAY be a symlink to `CLAUDE.md`) | Cursor |
| `.github/copilot-instructions.md` | `.github/copilot-instructions.md` | GitHub Copilot instructions (MAY be a symlink to `CLAUDE.md`) | GitHub Copilot |
| `docs/mcp-tools.md` | `docs/mcp-tools.md` | MCP tool documentation when the repository publishes an MCP server | AI agents |
| `conformance.yaml` | Repository root | Declared EPAS v1.4 conformance state | CI, auditors |

Repositories without `llms.txt` or `CLAUDE.md` (or the locally-appropriate equivalent per Section 6) are non-conformant.

---

## 4. llms.txt Standard

`llms.txt` provides AI agents with a curated, token-efficient index of repository structure. Think of `llms.txt` as the "sitemap for LLMs."

### 4.1 llms.txt Format

```markdown
# Project Name

> One-line description of what this project does.

## Quick Start
- Setup: /docs/setup.md
- Configuration: /docs/configuration.md
- First deploy: /docs/deploy.md

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
- Local environment: /docs/dev-environment.md

## Agent Context
- Coding rules: /CLAUDE.md
- MCP tools: /docs/mcp-tools.md
- Conformance: /conformance.yaml

## External References
- Published SDK: https://...
- Service dashboard: https://...
```

### 4.2 llms.txt Rules

- Every entry MUST be a path or a URL.
- Every entry MUST be accurate as of the current commit. Stale entries are non-conformant.
- Entries SHOULD be grouped by purpose, not by file-system layout.
- The file SHOULD be under 200 lines. Longer manifests defeat the token-efficiency purpose.

### 4.3 llms-full.txt (Optional)

`llms-full.txt` concatenates the full content of every document referenced in `llms.txt`. Permitted for small repositories where the full context fits comfortably in a typical agent context window. Discouraged for large repositories where the file becomes too large to be useful.

### 4.4 CI Validation

CI MUST verify:

- `llms.txt` exists at the repository root.
- Every path referenced in `llms.txt` exists in the repository.
- Every URL referenced in `llms.txt` returns a non-error response (optional, rate-limited).

---

## 5. Agent Context Files

Agent context files provide persistent, project-specific memory for AI coding agents. The canonical filename is `CLAUDE.md`; platforms MAY maintain additional files with the same content for other agents (`.cursor/rules`, `.github/copilot-instructions.md`), typically as symlinks.

### 5.1 Required Sections

Every `CLAUDE.md` MUST contain the following sections, in this order:

```markdown
# Project Context for AI Agents

## Project Overview
[2-3 sentence description. What does this project do? For whom? In what phase?]

## Tech Stack
- Language: [Python 3.11+, TypeScript 5.x, etc.]
- Framework: [FastAPI, Next.js, etc.]
- Database: [PostgreSQL with pgvector, etc.]
- Auth: [OIDC via X, OpenFGA, etc.]

## Architecture
[One paragraph naming the architectural approach.]
- Services communicate via SDK, never direct API calls
- All database access through repository pattern
- Events published via transactional outbox
- [Additional architecture invariants specific to this project]

## Coding Standards
- [Language-specific rule, e.g., "Always use async/await for I/O operations"]
- [Framework-specific rule, e.g., "Use Pydantic for all data validation"]
- [Project-specific rule, e.g., "Never import directly from data layer in API routes"]

## Common Mistakes to Avoid
- [Project-specific pitfall #1]
- [Project-specific pitfall #2]

## Testing Requirements
- All new code requires unit tests.
- Integration tests for API endpoints.
- Mock LLM calls in tests (never real API).

## Commands
[Copy-pasteable commands grouped by concern. Only document commands that work today.]

## File Naming Conventions
- Models: `models/{entity}.py`
- Schemas: `schemas/{entity}.py`
- [etc.]

## Agent Guardrails

### Allowed Operations
- [Operations this agent may perform autonomously]

### Approval Required
- [Operations requiring human approval]

### Prohibited Operations
- [Operations the agent MUST NOT perform]
```

Sections that do not apply to a specific project MAY be marked `[Not applicable]` but MUST NOT be removed. Absence of a required section is non-conformant.

### 5.2 Agent Guardrails Section

The **Agent Guardrails** section in `CLAUDE.md` is the human-readable expression of the Agentic Governance specified in document 06. The guardrails in `CLAUDE.md` are **soft enforcement** (the agent reads them and complies); the hard enforcement lives in CI and runtime authorization per document 06.

Guardrails in `CLAUDE.md` MUST be consistent with the runtime policy. When they drift, the runtime policy is authoritative and `CLAUDE.md` is out of date.

### 5.3 CLAUDE.md Maintenance

`CLAUDE.md` documents what works today. Aspirational documentation is non-conformant. When an agent reads `CLAUDE.md`, the agent will try to run commands it finds there. Commands that fail because the documentation is stale produce failing implementations.

A platform MUST run CI validation that either:

- Executes the "Commands" section against a test harness, or
- Verifies each command is declared in the repository's CI configuration (so commands in `CLAUDE.md` are a subset of commands CI already runs).

---

## 6. Equivalent Files for Other Agents

Agents other than Claude Code have their own context file conventions:

| Agent | Expected File | Typical Relationship to `CLAUDE.md` |
|-------|---------------|-------------------------------------|
| Claude Code | `CLAUDE.md` | Canonical |
| Cursor | `.cursor/rules` | Symlink or regenerated |
| GitHub Copilot | `.github/copilot-instructions.md` | Symlink or regenerated |
| Gemini CLI | `GEMINI.md` | Symlink or regenerated |
| OpenAI Codex / Codex CLI | `AGENTS.md` | Symlink or regenerated |
| Aider | `.aider.conf` or `CONVENTIONS.md` | Regenerated |

A repository SHOULD maintain one canonical file (`CLAUDE.md`) and symlink or regenerate the others. A repository MUST maintain the context file for every agent the team supports.

---

## 7. RAG-Optimized Markdown

Documentation is consumed by retrieval-augmented generation pipelines that chunk text. A paragraph that loses meaning when isolated from its neighbors will be retrieved and answered incorrectly. EPAS v1.4 requires five rules to ensure documentation survives chunking.

### 7.1 Rule 1 — No Orphan Pronouns

A pronoun (it, they, this, that) MUST NOT reference an antecedent from a previous paragraph. Restate the entity.

```markdown
# Non-conformant (fails when chunk boundary falls here)
## Auth Service Configuration
The Auth Service supports multiple formats. You can configure it via environment variables.

# Conformant
## Auth Service Configuration
The Auth Service supports multiple configuration formats. Auth Service configuration can be set via environment variables.
```

### 7.2 Rule 2 — Explicit Entity Naming in Headers

Headers MUST name the entity being described, not a generic category.

```markdown
# Non-conformant (ambiguous when retrieved in isolation)
## Configuration
## API Reference
## Testing

# Conformant
## Auth Service Configuration
## Auth Service API Reference
## Auth Service Testing Guide
```

### 7.3 Rule 3 — Strict Header Hierarchy

Headers MUST follow strict H1 → H2 → H3 nesting without skipped levels. Recursive markdown splitters produce incoherent chunks when hierarchy is broken.

```markdown
# Non-conformant
# Service Overview
### API Endpoints       ← skipped H2

# Conformant
# Auth Service Overview
## Auth Service API
### Auth Service API Endpoints
```

### 7.4 Rule 4 — Self-Contained Paragraphs

Each paragraph MUST stand alone. Context from the previous paragraph MUST NOT be required to understand the current paragraph.

```markdown
# Non-conformant
The service uses JWT tokens. They expire after 15 minutes.

# Conformant
The Auth Service uses JWT tokens for authentication. Auth Service JWT tokens expire after 15 minutes.
```

### 7.5 Rule 5 — Front-Loaded Key Information

The most important fact in any section MUST appear in the first one or two sentences. Buried key information is lost in retrieval.

```markdown
# Non-conformant
After considering various options and evaluating the trade-offs between
complexity and performance, we decided to use PostgreSQL for the database.

# Conformant
PostgreSQL is the required database for all EPAS v1.4 services. This decision
was made after evaluating trade-offs between complexity and performance.
```

### 7.6 CI Validation

CI MUST validate the five rules above on every markdown file in `docs/` and at the repository root. Validation MAY be implemented via:

- `markdownlint` with a custom ruleset for rules 2 and 3 (hierarchy, entity naming).
- A custom RAG-readiness linter for rules 1, 4, and 5.
- A sampled LLM-as-judge evaluation run on changed files in CI.

Repositories without RAG-readiness validation in CI are non-conformant.

---

## 8. MCP Tool Documentation

Repositories that publish a Model Context Protocol server MUST document every tool in `docs/mcp-tools.md`. Required fields per tool:

```markdown
### Tool Name: {tool_name}

**Purpose:** [One-sentence description of what the tool does.]

**Parameters:**
- `param_a` (string, required) — [Description.]
- `param_b` (integer, optional, default 0) — [Description.]

**Returns:** [Description of return shape.]

**Side Effects:** [None | Writes to <ledger/event plane/etc>]

**Contract Impact:** [How this tool relates to the contract model — does it offer a contract, consume one, or neither?]

**Guardrails:**
- Risk level: [low | medium | high | critical]
- Approval required: [none | single | dual | board]

**Example:**
\```json
{
  "param_a": "example-value",
  "param_b": 42
}
\```
```

Tools without complete documentation MUST NOT be registered with production MCP servers.

---

## 9. Documentation and the Contract Model

Documentation describing any mutating operation MUST describe:

- What contract is offered when the operation is invoked.
- What scope the contract requires.
- What refusal reasons are expected in normal operation.
- What execution events the operation emits.

Documentation that describes mutating operations without contract context is incomplete and non-conformant.

---

## 10. Versioning Machine-Reader Documentation

Documentation versioning follows the code. A documentation change accompanying a code change MUST land in the same pull request as the code change. Documentation drift from code is a regression that MUST be caught by CI.

---

## 11. Machine-Reader Documentation Conformance Requirements

A repository conforms to EPAS v1.4 Documentation for Machine Readers when:

1. `llms.txt` exists at the repository root and is accurate.
2. `CLAUDE.md` (or locally-appropriate equivalent) exists with all required sections from Section 5.1.
3. Equivalent context files exist for every AI agent the team supports (Section 6).
4. Every markdown file in `docs/` follows the five RAG-optimized markdown rules.
5. If the repository publishes MCP tools, `docs/mcp-tools.md` documents every tool per Section 8.
6. CI validates `llms.txt` freshness, `CLAUDE.md` section completeness, RAG-readiness rules, and MCP tool documentation.
7. Documentation describing mutating operations includes contract context per Section 9.

Partial conformance during migration from v1.3 is permitted and MUST be declared in `conformance.yaml`.

---

## 12. Machine-Reader Documentation Non-Goals

This specification does not:

- Define a specific AI coding agent, RAG framework, or vector store.
- Mandate a documentation generation tool.
- Replace internal style guides or tone-of-voice standards.
- Specify documentation content beyond the required sections.

Content quality beyond structural conformance remains the team's responsibility. This specification ensures that documentation is *readable*; making it *good* is out of scope.
