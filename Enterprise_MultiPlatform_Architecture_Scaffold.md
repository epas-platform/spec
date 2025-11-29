Enterprise Multi‑Platform Architecture Scaffold
Version 1.3.0 – Environment-First, A2A Cost Management, Regulatory Alignment

⸻

# 🎯 What's New in v1.3.0

**Strategic Shifts**:
1. **Environment-First Architecture** – Environments are top-level infrastructure isolation, not config
2. **A2A Cost Tracking Mandatory** – Inter-agent MCP overhead, per-agent attribution, real-time alerts
3. **Multi-Model AI by Default** – 8 LLM providers with cost-based routing
4. **Regulatory Compliance Built-In** – HIPAA, EU AI Act, GDPR, BIPA controls embedded
5. **ITIL v4 Service Management** – Incident & change aligned to industry-standard ITSM practices
6. **Phased Event Bus Strategy** – Kafka Phase 1 (audit), NATS Phase 2 (latency optimization)

**Key Decisions**:
- Cost alerts at **75% (warning)** and **95% (critical)** budget utilization
- Model approval: **CAIO** (informed by CISO, CIO; CTO made aware)
- Event bus: **Kafka/Redpanda** for dev/Phase 1, **NATS/JetStream** when squeezing 100ms matters
- Business Unit: P&L-capable but customer-flexible (NOT knowledge domain)

**Detailed Sections Added**:
- Section 9: A2A Cost Tracking & FinOps (comprehensive)
- Section 10: Multi-Model AI Strategy
- Section 11: Event-Driven Architecture (5 options compared)
- Section 14: Regulatory Compliance & AI Governance
- Section 15: Security Architecture & Hardening
- Section 16: Incident Response (ITIL v4 aligned) 🆕
- Section 17: Disaster Recovery & Business Continuity 🆕
- Section 19: Vendor & Subprocessor Management 🆕
- Section 20: Change Management (ITIL v4 aligned) 🆕
- Section 21: Developer Experience & Local Development

**For Stakeholders**:
- **CFO/COO**: See Section 9 (Cost Tracking), Appendix A (Cost Model)
- **CISO**: See Section 15 (Security), Appendix D (Security Checklist)
- **CAIO**: See Section 10 (Multi-Model AI), Section 14 (AI Governance)
- **CTO/CIO**: See Section 11 (Event Bus), Section 21 (Developer Experience)
- **CRO/CLO**: See Section 14 (Regulatory Compliance), Appendix B (Regulatory Checklist)

⸻

# Document Control

## Version History
- **v1.0.0** – Initial reference architecture and multi-tenant model
- **v1.1.0** – Event-driven extensions, A2A architecture, AI governance primitives, initial cost tracking model
- **v1.2.0** – Environment & profile strategy, AWS-first deployment blueprint, full data handling & security baseline
- **v1.3.0** – Environment-first hierarchy, comprehensive A2A cost management, regulatory compliance integration, multi-model LLM strategy, expanded event architecture options, endpoint security

## Audience & Stakeholder Roles (RACI)

| Role | Responsibilities | Examples |
|------|------------------|----------|
| **Responsible** (owns decisions) | Architecture decisions, implementation choices, technical standards | CTO, CIO, CISO |
| **Accountable** (budget & risk oversight) | Budget approval, risk acceptance, strategic alignment | CFO, COO, CAIO (Chief AI Officer) |
| **Consulted** (domain expertise) | Compliance guidance, data governance, regulatory interpretation | Chief Data Officer, Chief Compliance Officer |
| **Informed** (strategic awareness) | Business impact, market positioning, legal implications | CEO, CRO (Chief Revenue Officer), CLO (Chief Legal Officer) |

**Note**: CAIO (Chief AI Officer) is an emerging C-suite role responsible for AI strategy, governance, and ethical deployment across the enterprise.

## Non-Goals
- This is not legal advice or a certification document
- It is a technical scaffold that can be aligned with internal policies, legal opinions, and regulatory programs
- Implementation details are illustrative; adapt to your organizational context

⸻

# Core Principles

These principles drive every architectural decision:

- **Defense in depth** – Multiple layers of protection: edge → service → data → tooling
- **Least privilege by default** – Services, agents, and tools get minimum access necessary
- **Zero-trust boundaries** – Every internal call authenticated and authorized; no implicit trust
- **Deterministic orchestration** – Agents and tools run through schema-validated flows
- **Environment isolation mandatory** – Dev/prod separation at infrastructure level, not configuration
- **Dev/prod parity** – Local development mirrors cloud deployments (LocalStack, DNS, TLS)
- **Evidence-first architecture** – Assume every control must be proven with logs, configs, traces
- **Privacy & security by default** – Encryption everywhere, data minimization, strong access controls
- **Cost visibility mandatory** – Every A2A call, LLM invocation, and resource usage tracked
- **Multi-model by default** – Support multiple LLM providers for cost optimization and resilience

⸻

# Architectural Overview

## Planes

The platform is logically split into four planes:

- **Control Plane**
  - Tenant/org/business unit/project configuration
  - Model & tool catalogs
  - Policies, guardrails, access control
  - Observability configuration
  - Cost management & budget enforcement

- **Data Plane**
  - Runtime agents and workflows (LangGraph-based A2A)
  - Connectors and tools (APIs, MCP, webhooks)
  - Storage and search for operational data
  - Multi-tenant data isolation

- **Event Plane** (Mandatory)
  - Asynchronous messaging backbone (Kafka, NATS, or equivalent)
  - CloudEvents-based event propagation
  - AsyncAPI contracts for event schemas
  - Dead letter queues and retry logic

- **Observability Plane**
  - Logs, metrics, traces (OpenTelemetry)
  - Security and audit logs
  - Evaluation runs and cost tracking
  - AI decision logging and explainability

## Environment-First Hierarchy

**CRITICAL ARCHITECTURAL DECISION**: Environments are isolated at the infrastructure level, not configuration level.

```
┌─────────────────────────────────────────┐
│ ENVIRONMENT (Top-Level Isolation)      │
│  ├─ dev                                 │
│  ├─ test                                │
│  ├─ staging                             │
│  └─ prod                                │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ ORGANIZATION (Customer/Tenant)          │
│  org_acme, org_initech, org_cyberdyne   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ BUSINESS UNIT (P&L or Logical Grouping)│
│  "Sales", "Engineering", "Operations"   │
│  Note: Customer-defined taxonomy        │
│  ⚠️ NOT knowledge domain                │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ TEAM (Functional Group)                 │
│  "Platform Team", "Data Team"           │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ PROJECT (Solution/Product)              │
│  "Voice AI", "Customer Portal"          │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ RESOURCE (Agents, Tools, Workflows)     │
│  agent_voice_io, tool_weather, etc.     │
└─────────────────────────────────────────┘
```

### Environment Isolation Requirements

**All code MUST be environment-agnostic**. Environmental differences are abstracted via:

1. **Separate Infrastructure**: Each environment has dedicated:
   - AWS accounts (or equivalent cloud isolation)
   - VPCs and network boundaries
   - KMS keys and encryption boundaries
   - Secrets Manager / Parameter Store instances
   - Identity providers (Zitadel/Entra instances)

2. **Configuration Abstraction**:
   ```python
   # ✅ CORRECT: Environment-agnostic code
   secret_value = await secrets_client.get_secret("api_key")
   
   # ❌ WRONG: Environment-specific logic
   if env == "prod":
       api_key = "prod_key"
   ```

3. **DevSecOps + AgentOps Mandatory**:
   - CI/CD pipelines promote artifacts across environments
   - Immutable container images (same SHA in dev → prod)
   - Configuration injected at deployment time
   - Agent behavior identical across environments
   - Cost tracking per environment for comparison

⸻

# Architectural Tenets (v1.3)

```yaml
architectural_tenets:
  # Tenancy & Isolation
  - tenant_isolation_by_design
  - environment_first_architecture        # NEW: Env > Org hierarchy
  - business_unit_flexibility            # NEW: P&L-capable but customer-defined
  
  # Configuration & Development
  - profile_based_environments
  - localstack_mandatory_for_dev
  - lazy_credential_loading
  - environment_agnostic_code            # NEW: No env-specific logic
  
  # Security & Trust
  - zero_trust_internal_calls
  - defense_in_depth
  - mcp_for_tool_integration_only
  
  # Orchestration & Events
  - deterministic_orchestration
  - event_plane_mandatory                # NEW: Not optional
  - event_bus_pluggable                  # NEW: Multiple options
  
  # AI & Cost
  - a2a_cost_tracking_mandatory          # NEW: Inter-agent call costs
  - multi_model_by_default               # NEW: Multiple LLM providers
  - ai_cost_tracking_built_in
  - token_attribution_per_agent          # NEW: Which agent used what
  
  # Standards & Operations
  - open_standards_everywhere
  - infrastructure_as_code
  - local_dev_equals_cloud
  - devsecops_and_agentops_required      # NEW: Both mandatory
  
  # Compliance & Governance
  - compliance_as_code
  - evidence_first
  - regulatory_alignment_built_in        # NEW: HIPAA, EU AI Act, etc.
  - supply_chain_transparency            # NEW: SBOM mandatory
  - itil_v4_service_management           # NEW: Change, incident, problem aligned to ITIL
```

⸻

# Identity, Scopes, Roles, Authorities

## Scopes (Environment-First Hierarchy)

**Definition**: Design-time taxonomy with explicit hierarchical inheritance.

```
environment → organization → business_unit → team → project → resource
```

**Properties**:
- Unique scope IDs (UUIDv7 recommended for sortability)
- Parent pointer for inheritance
- Lifecycle (create/rename/merge/retire)
- Compliance tags (HIPAA, PCI, SOC2)
- Residency tags (us-east-1, eu-central-1)
- Cost allocation tags (for chargeback)

**Business Unit Nuances**:
- **Primary Use**: P&L center with budget and cost allocation
- **Flexibility**: Customers can use for logical grouping ("Sales", "Engineering")
- **⚠️ NOT Knowledge Domain**: Don't confuse with data classification or subject matter
- **Authority Inheritance**: Business Unit Admin MAY inherit Team-level authorities (configurable)

## Authorities (Immutable)

**Concept**: Atomic, immutable permissions aligned to scope + CRUD+X (create/read/update/delete/execute) over resources and administrative actions.

**Examples**:
- `ORG_ADMIN`, `ORG_CONFIGURE`, `ORG_BILLING`
- `BUSINESS_UNIT_ADMIN`, `BUSINESS_UNIT_VIEW_COSTS`
- `TEAM_MANAGE_MEMBERS`, `TEAM_CONFIGURE`
- `PROJECT_CONFIGURE`, `PROJECT_DEPLOY`
- `RESOURCE_ADMIN`, `RESOURCE_EXECUTE`
- `AUDIT_READ`, `DATA_MANAGEMENT`
- `SYSTEM_RBAC`, `SYSTEM_ADMIN`
- `COST_VIEW`, `COST_BUDGET_MANAGE` (NEW)
- `AI_MODEL_APPROVE`, `AI_MONITOR` (NEW)

## Roles (Configurable)

**Concept**: Named collections of authorities; admin-configurable, supports custom roles.

**System Roles** (ship with platform):
- System Admin
- Organization Admin
- Business Unit Admin (NEW)
- Team Admin
- Project Admin
- Auditor
- Cost Analyst (NEW)
- AI Governance Officer (NEW)
- Support
- Billing Admin
- Read-Only

**Custom Roles**: Customers can define via UI + API; maintain change-controlled role catalogs with diffs & approvals.

## ABAC Augmentation

Optional attribute guards (region, data classification, business hours, cost budget) evaluated at request-time; implement via policy engine (Cedar/OPA/OpenFGA) with decision auditability.

⸻

# A2A Cost Tracking & FinOps (MANDATORY)

## Overview

In an Agent-to-Agent (A2A) architecture, **cost visibility is critical**. Unlike traditional microservices where costs are infrastructure-centric, A2A systems have:
- LLM API costs per agent invocation
- Inter-agent communication overhead (MCP protocol)
- Multi-model routing decisions
- Token usage varying by agent complexity

**Business Impact**: Without cost tracking, A2A systems can incur runaway costs. Example from SAAA project:
- Direct LLM call: 1590ms, ~$0.026/minute
- A2A mesh call: 1990ms (+400ms overhead), potentially 2-3x cost due to orchestrator + agent LLM calls

## Cost Tracking Architecture

### 1. Cost Event Schema

```python
class CostEvent(BaseModel):
    """Emitted for every cost-bearing operation"""
    event_id: UUID
    timestamp: datetime
    environment: EnvironmentType  # dev, test, staging, prod
    
    # Hierarchy
    org_id: UUID
    business_unit_id: Optional[UUID]
    team_id: Optional[UUID]
    project_id: UUID
    
    # Resource context
    resource_type: str  # "agent", "tool", "llm_call", "mcp_call"
    resource_id: str    # "voice-io-agent", "weather-tool"
    
    # Cost dimensions
    cost_type: CostType  # llm_tokens, mcp_overhead, storage, compute
    provider: str  # "anthropic", "openai", "deepgram", "elevenlabs"
    model: str  # "claude-sonnet-4", "gpt-4o", "nova-2", "turbo-v2.5"
    
    # Usage metrics
    tokens_input: Optional[int]
    tokens_output: Optional[int]
    tokens_total: Optional[int]
    duration_ms: Optional[int]
    
    # Calculated cost
    cost_usd: Decimal
    cost_calculation_method: str  # "provider_api", "estimated", "metered"
    
    # Trace context
    trace_id: str
    span_id: str
    parent_span_id: Optional[str]
    
    # Metadata
    metadata: Dict[str, Any]  # Agent-specific context
```

### 2. Per-Agent Token Attribution

**Problem**: In A2A systems, a single user request may trigger multiple agent calls. Which agent "owns" the cost?

**Solution**: Hierarchical cost attribution

```python
# Example: User asks voice agent a weather question
# Trace:
#   voice-io-agent (orchestrator)
#     ├─ STT: $0.004 (Deepgram)
#     ├─ orchestrator-llm: $0.015 (Claude Sonnet)
#     ├─ MCP call → weather-agent
#     │   └─ weather-agent-llm: $0.008 (GPT-4o mini)
#     └─ TTS: $0.009 (ElevenLabs)
#
# Total: $0.036
# Attribution:
#   - voice-io-agent: $0.028 (direct costs)
#   - weather-agent: $0.008 (called via MCP)
```

**Implementation**:
```python
async def track_agent_cost(
    agent_id: str,
    cost_usd: Decimal,
    trace_id: str,
    parent_agent_id: Optional[str] = None,
):
    """Emit cost event with attribution chain"""
    await cost_tracker.emit(CostEvent(
        resource_type="agent",
        resource_id=agent_id,
        cost_usd=cost_usd,
        trace_id=trace_id,
        metadata={
            "parent_agent_id": parent_agent_id,
            "attribution_type": "direct" if not parent_agent_id else "indirect"
        }
    ))
```

### 3. Inter-Agent MCP Overhead Tracking

**Problem**: MCP calls add latency and LLM cost (orchestrator must reason about which agent to call).

**Measurement**:
- Base latency: Direct LLM call (750ms, $0.015)
- A2A latency: MCP call (1100ms, $0.023)
- **Overhead**: +350ms, +$0.008 (orchestrator reasoning + network)

**Tracking**:
```python
@track_mcp_cost
async def call_remote_agent(agent_id: str, tool_name: str, params: Dict):
    start_time = time.time()
    start_cost_snapshot = await get_current_cost()
    
    result = await mcp_client.call_tool(agent_id, tool_name, params)
    
    duration_ms = (time.time() - start_time) * 1000
    overhead_cost = await get_current_cost() - start_cost_snapshot
    
    await cost_tracker.emit(CostEvent(
        resource_type="mcp_call",
        resource_id=f"{agent_id}/{tool_name}",
        cost_type=CostType.MCP_OVERHEAD,
        duration_ms=duration_ms,
        cost_usd=overhead_cost,
        metadata={"agent_id": agent_id, "tool_name": tool_name}
    ))
```

### 4. Multi-Model Cost Comparison Matrix

**Goal**: Enable cost-based routing decisions.

| Provider | Model | Input ($/1M tokens) | Output ($/1M tokens) | Latency (p50) | Use Case |
|----------|-------|--------------------:|---------------------:|--------------:|----------|
| **Anthropic** | Claude Sonnet 4 | $3.00 | $15.00 | 800ms | High-quality reasoning |
| **Anthropic** | Claude Haiku 3.5 | $0.25 | $1.25 | 400ms | Fast classification |
| **OpenAI** | GPT-4o | $2.50 | $10.00 | 1000ms | General purpose |
| **OpenAI** | GPT-4o-mini | $0.15 | $0.60 | 300ms | Simple tasks |
| **AWS Bedrock** | Claude Sonnet (Bedrock) | $3.00 | $15.00 | 900ms | AWS-native |
| **Azure OpenAI** | GPT-4o (Azure) | $2.50 | $10.00 | 1100ms | Enterprise SSO |
| **Google** | Gemini 1.5 Pro | $1.25 | $5.00 | 850ms | Multimodal |
| **Local** | Llama 3.1 70B (self-hosted) | $0.10* | $0.10* | 1200ms | Data residency |

*Self-hosted costs include infrastructure amortization

**Cost-Based Routing**:
```python
class ModelRouter:
    async def select_model(self, task_type: str, budget_usd: Decimal) -> str:
        if task_type == "classification" and budget_usd < 0.005:
            return "gpt-4o-mini"  # $0.0015/classification
        elif task_type == "reasoning" and budget_usd < 0.02:
            return "claude-haiku-3.5"  # $0.008/reasoning
        else:
            return "claude-sonnet-4"  # $0.018/reasoning (high quality)
```

### 5. Chargeback & Showback to Business Units

**Goal**: Allocate AI costs to P&L owners for accountability.

```sql
-- Monthly Cost Summary by Business Unit
CREATE TABLE business_unit_cost_summary (
    business_unit_id UUID NOT NULL,
    period_start TIMESTAMPTZ NOT NULL,
    period_end TIMESTAMPTZ NOT NULL,
    
    -- Cost breakdown
    llm_cost_usd DECIMAL(10, 2),
    stt_cost_usd DECIMAL(10, 2),
    tts_cost_usd DECIMAL(10, 2),
    mcp_overhead_cost_usd DECIMAL(10, 2),
    compute_cost_usd DECIMAL(10, 2),
    storage_cost_usd DECIMAL(10, 2),
    total_cost_usd DECIMAL(10, 2),
    
    -- Usage metrics
    total_agent_calls INTEGER,
    total_tokens BIGINT,
    total_minutes_audio DECIMAL(10, 2),
    
    -- Budget tracking
    budget_allocated_usd DECIMAL(10, 2),
    budget_remaining_usd DECIMAL(10, 2),
    budget_utilization_pct DECIMAL(5, 2),
    
    PRIMARY KEY (business_unit_id, period_start)
) PARTITION BY RANGE (period_start);
```

**Chargeback vs Showback**:
- **Showback**: Report costs for visibility (no actual financial transfer)
- **Chargeback**: Allocate costs to BU P&L (requires finance integration)

### 6. Real-Time Cost Dashboards & Budget Alerts

**Dashboard Metrics**:
```yaml
cost_dashboard:
  real_time:
    - current_hourly_burn_rate_usd
    - projected_monthly_cost_usd
    - top_5_cost_agents
    - cost_per_business_unit
    - budget_utilization_by_project
  
  historical:
    - cost_trend_last_30_days
    - cost_by_model_provider
    - token_efficiency_by_agent
    - mcp_overhead_trend
  
  alerts:
    - hourly_burn_rate_threshold: 10.00  # Alert if >$10/hr
    - business_unit_budget_warning: 0.75  # Warning at 75% budget
    - business_unit_budget_critical: 0.95  # Critical at 95% budget
    - agent_cost_anomaly: 2.0x  # Alert if agent cost >2x normal
    - project_budget_exceeded: true
```

**Alerting Logic**:
```python
async def check_cost_alerts():
    """Run every 5 minutes"""
    for bu in await get_business_units():
        summary = await get_current_cost_summary(bu.id)
        
        if summary.budget_utilization_pct >= 75 and summary.budget_utilization_pct < 95:
            # WARNING: Approaching budget limit
            await send_alert(
                severity="warning",
                message=f"BU {bu.name} at {summary.budget_utilization_pct}% budget",
                recipients=[bu.finance_owner, bu.technical_owner]
            )
        
        if summary.budget_utilization_pct >= 95:
            # CRITICAL: Near or exceeded budget, throttle agents
            await throttle_business_unit(bu.id, throttle_pct=0.5)
            await send_alert(
                severity="critical",
                message=f"BU {bu.name} at {summary.budget_utilization_pct}% budget - THROTTLING ENABLED",
                recipients=[bu.finance_owner, bu.technical_owner, cfo_email, caio_email]
            )
```

### 7. Cost Optimization Strategies

**Techniques**:

1. **Prompt Caching** (Anthropic, OpenAI)
   ```python
   # Cache system prompts to reduce input token costs
   system_prompt = "You are a helpful assistant..."  # 500 tokens
   # First call: $0.0015
   # Cached calls: $0.0001 (10x reduction)
   ```

2. **Prompt Compression**
   ```python
   # Before: 2000 tokens, $0.006
   original = "The following is a detailed analysis of..."
   
   # After: 500 tokens, $0.0015 (4x reduction)
   compressed = prompt_compressor.compress(original)
   ```

3. **Model Cascading**
   ```python
   # Try cheap model first, fallback to expensive if needed
   result = await try_model("gpt-4o-mini", prompt)  # $0.0015
   if result.confidence < 0.8:
       result = await try_model("claude-sonnet-4", prompt)  # $0.018
   ```

4. **Batch Processing**
   ```python
   # Batch 10 requests into 1 LLM call
   # Individual: 10 calls × $0.015 = $0.15
   # Batched: 1 call × $0.045 = $0.045 (3x reduction)
   ```

5. **Token Budgets per Agent**
   ```python
   class AgentTokenBudget:
       max_input_tokens: int = 1000
       max_output_tokens: int = 500
       max_cost_per_call_usd: Decimal = Decimal("0.05")
   ```

**Cost Optimization Dashboard**:
```yaml
optimization_metrics:
  - cache_hit_rate: 0.65  # 65% of calls use cached prompts
  - avg_tokens_per_call: 1200 → 800 (33% reduction)
  - model_routing_savings_usd: 1200.00/month
  - batch_processing_savings_usd: 800.00/month
```

⸻

# Multi-Model AI Strategy

## Provider Selection Matrix

**Decision Factors**:
1. **Cost**: $/token varies 20x between models
2. **Latency**: 300ms (GPT-4o-mini) to 1200ms (self-hosted)
3. **Quality**: Reasoning depth, accuracy, hallucination rate
4. **Compliance**: Data residency, training data usage, SOC 2
5. **Availability**: SLA, rate limits, fallback options

## Recommended Multi-Model Stack

```yaml
llm_strategy:
  primary:
    provider: anthropic
    model: claude-sonnet-4
    use_case: high-quality reasoning, complex multi-step tasks
    cost: $3.00/$15.00 per 1M tokens (input/output)
    
  fast:
    provider: anthropic
    model: claude-haiku-3.5
    use_case: classification, simple Q&A, routing decisions
    cost: $0.25/$1.25 per 1M tokens
    
  fallback:
    provider: openai
    model: gpt-4o
    use_case: when primary is unavailable
    cost: $2.50/$10.00 per 1M tokens
    
  cost_optimized:
    provider: openai
    model: gpt-4o-mini
    use_case: high-volume simple tasks
    cost: $0.15/$0.60 per 1M tokens
    
  enterprise_sso:
    provider: azure_openai
    model: gpt-4o
    use_case: customers requiring Entra ID integration
    cost: $2.50/$10.00 per 1M tokens
    
  data_residency:
    provider: aws_bedrock
    model: claude-sonnet-4
    use_case: EU data residency requirements
    cost: $3.00/$15.00 per 1M tokens
    
  multimodal:
    provider: google
    model: gemini-1.5-pro
    use_case: image/video/audio analysis
    cost: $1.25/$5.00 per 1M tokens
    
  self_hosted:
    provider: local
    model: llama-3.1-70b
    use_case: extreme data sensitivity, regulated industries
    cost: $0.10/$0.10 per 1M tokens* (*infrastructure cost)
```

## Model Routing & Fallback Logic

```python
class LLMOrchestrator:
    """Routes requests to optimal model based on requirements"""
    
    async def route_request(
        self,
        prompt: str,
        requirements: ModelRequirements
    ) -> LLMResponse:
        # 1. Check data residency
        if requirements.data_residency == "EU":
            models = [self.bedrock_eu, self.azure_eu]
        else:
            models = [self.claude_sonnet, self.gpt4o, self.claude_haiku]
        
        # 2. Check budget
        if requirements.max_cost_usd < 0.005:
            models = [self.gpt4o_mini, self.claude_haiku]
        
        # 3. Check latency SLA
        if requirements.max_latency_ms < 500:
            models = [self.claude_haiku, self.gpt4o_mini]
        
        # 4. Try models with fallback
        for model in models:
            try:
                return await model.generate(prompt, requirements)
            except (RateLimitError, ServiceUnavailableError) as e:
                logger.warning(f"Model {model.name} unavailable: {e}")
                continue
        
        raise AllModelsUnavailableError("No models available")
```

## Model Registry & Versioning

```sql
CREATE TABLE ai_model_registry (
    model_id UUID PRIMARY KEY,
    provider TEXT NOT NULL,  -- anthropic, openai, aws_bedrock, etc.
    model_name TEXT NOT NULL,  -- claude-sonnet-4, gpt-4o
    model_version TEXT NOT NULL,  -- 20241120, gpt-4o-2024-11-20
    
    -- Capabilities
    max_context_tokens INTEGER,
    supports_streaming BOOLEAN,
    supports_function_calling BOOLEAN,
    supports_vision BOOLEAN,
    
    -- Pricing (per 1M tokens)
    cost_input_usd DECIMAL(10, 4),
    cost_output_usd DECIMAL(10, 4),
    
    -- Performance
    median_latency_ms INTEGER,
    p95_latency_ms INTEGER,
    
    -- Compliance
    data_residency TEXT[],  -- ['US', 'EU', 'GLOBAL']
    training_data_cutoff DATE,
    soc2_certified BOOLEAN,
    hipaa_eligible BOOLEAN,
    
    -- Governance
    approved_for_production BOOLEAN DEFAULT FALSE,
    risk_tier TEXT,  -- high, limited, minimal
    approval_date DATE,
    approved_by UUID,
    
    -- Status
    status TEXT DEFAULT 'active',  -- active, deprecated, experimental
    deprecated_date DATE,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Fine-Tuned Model Governance

**Approval Workflow (RACI)**:

| Stage | Responsible | Accountable | Consulted | Informed |
|-------|-------------|-------------|-----------|----------|
| **Training Data Review** | Data Science | CAIO | Legal, Compliance | - |
| **Bias Testing** | AI Governance Team | CAIO | CISO, CIO | - |
| **Risk Assessment** | CAIO | CAIO | CISO, CIO | CTO |
| **Production Approval** | **CAIO** | CAIO | **CISO, CIO** | **CTO** |
| **Ongoing Monitoring** | ML Ops | CAIO | CISO | CTO |
| **Incident Response** | CAIO | CISO | CIO, Legal | CTO, CEO |

**Approval Authority**: CAIO is the approver, informed by CISO (security implications) and CIO (infrastructure impact). CTO is made aware but not a blocker.

**Process**:
1. **Training Data Approval**: Compliance review of training corpus (Legal + CAIO)
2. **Model Validation**: Bias testing, accuracy benchmarks (AI Governance Team)
3. **Risk Assessment**: NIST AI RMF categorization (CAIO with CISO/CIO input)
4. **Approval Gate**: CAIO approves for production after consulting CISO/CIO, CTO notified
5. **Monitoring**: Ongoing drift detection and performance tracking (ML Ops)
6. **Retirement**: Graceful deprecation with fallback models (CAIO approval required)

```python
class FineTunedModelLifecycle:
    STAGES = {
        "training": "Model training in progress",
        "validation": "Bias and accuracy testing",
        "risk_assessment": "CAIO reviewing with CISO/CIO input",
        "awaiting_approval": "CAIO decision pending",
        "cto_notification": "CTO being notified",
        "staging": "Deployed to staging environment",
        "production": "CAIO approved for production",
        "monitoring": "Active monitoring for drift",
        "deprecated": "Scheduled for retirement",
        "retired": "No longer available"
    }
    
    APPROVAL_REQUIRED_FROM = ["CAIO"]
    CONSULTED_STAKEHOLDERS = ["CISO", "CIO"]
    INFORMED_STAKEHOLDERS = ["CTO"]
```

⸻

# Event-Driven Architecture (Expanded Options)

## Why Event-Driven is Mandatory

In a multi-agent system:
- Agents don't know who needs their output
- Workflows span multiple services (orchestrator → agents → tools)
- Audit requires immutable event log
- Replay and debugging need event history

**Events are the source of truth, not API calls.**

## Event Bus Selection Criteria

| Criterion | Kafka/Redpanda | NATS/JetStream | AWS EventBridge | RabbitMQ | Apache Pulsar |
|-----------|----------------|----------------|-----------------|----------|---------------|
| **Throughput** | 100K+ msg/s | 50K+ msg/s | 10K+ msg/s | 20K+ msg/s | 100K+ msg/s |
| **Latency (p99)** | <100ms | <10ms | <500ms | <50ms | <100ms |
| **Ordering** | Partition-level | Stream-level | Best-effort | Queue-level | Topic-level |
| **Retention** | Days-years | Hours-days | 24 hours | Hours | Days-years |
| **Multi-Tenancy** | Topics | Streams/Accounts | Event buses | vHosts | Tenants |
| **Cost** | Medium | Low | Pay-per-event | Low | Medium |
| **Ops Complexity** | Medium | Low | Managed | Low | High |
| **Use Case** | Event sourcing, audit | Lightweight, cloud-native | AWS-native | Task queues | Multi-tenant SaaS |

## Recommended Event Bus Strategy (Phased Approach)

**Phase 1 (Dev & Initial Production)**: **Kafka/Redpanda**
- Proven for audit retention (7-year compliance)
- Rich ecosystem (Schema Registry, Kafka Streams)
- Easier to find experienced engineers
- Redpanda for simpler operations vs vanilla Kafka

**Phase 2 (Latency Optimization)**: **NATS/JetStream**
- When <10ms latency is required (vs Kafka's <100ms)
- **Savings**: ~90ms p99 latency reduction through tuning
- Use case: Real-time voice, high-frequency trading, gaming
- Migration path: Dual-write during transition

**Decision Logic**:
```python
if latency_requirement_p99 < 20ms:
    event_bus = "NATS/JetStream"  # Every 100ms matters
elif compliance_retention > 365:
    event_bus = "Kafka/Redpanda"  # Long-term audit
elif aws_exclusive_deployment:
    event_bus = "AWS EventBridge + SQS"
else:
    event_bus = "Kafka/Redpanda"  # Default
```

## Recommended Event Bus per Scenario

### Scenario 1: Enterprise SaaS (Multi-Tenant)
**Recommendation**: **Kafka/Redpanda**

- Mature multi-tenant topic partitioning
- Long retention for audit (7-year compliance)
- Exactly-once semantics
- Ecosystem tooling (Schema Registry, Kafka Streams)

```yaml
kafka_config:
  brokers: 3  # HA cluster
  replication_factor: 3
  min_in_sync_replicas: 2
  retention_days: 2555  # 7 years
  
  topics:
    - agent.lifecycle.*
    - agent.decision.*
    - cost.events
    - audit.* # Compliance
```

### Scenario 2: Cloud-Native Startup
**Recommendation**: **NATS/JetStream**

- Lightweight, easy operations
- Sub-10ms latency
- Built-in clustering
- Lower cost than Kafka

```yaml
nats_config:
  cluster_size: 3
  jetstream_enabled: true
  storage: file
  retention: 7d
  
  streams:
    - agent_events
    - cost_events
    - audit_events
```

### Scenario 3: AWS-Exclusive Deployment
**Recommendation**: **AWS EventBridge + SQS**

- Fully managed (no ops overhead)
- Native IAM integration
- Pay-per-event pricing
- Integrates with Lambda, Step Functions

```yaml
eventbridge_config:
  event_buses:
    - name: agent-events
      rules:
        - pattern: {"source": ["saaa.agent"]}
          targets: [sqs_queue, lambda_function]
```

### Scenario 4: Simple Async Tasks
**Recommendation**: **RabbitMQ**

- Mature, battle-tested
- Good for task queues (webhooks, notifications)
- Lower throughput, but sufficient for non-event-sourcing use cases

```yaml
rabbitmq_config:
  exchanges:
    - name: agent_tasks
      type: topic
  queues:
    - name: webhooks
      durable: true
      max_priority: 10
```

### Scenario 5: Massive Scale (100K+ agents)
**Recommendation**: **Apache Pulsar**

- Designed for multi-tenancy at scale
- Geo-replication built-in
- Separate compute/storage layers

```yaml
pulsar_config:
  tenants:
    - org_acme
    - org_initech
  namespaces:
    - org_acme/production
    - org_acme/dev
  topics:
    - persistent://org_acme/production/agent-events
```

## CloudEvents Standard (Mandatory)

All events MUST use CloudEvents 1.0 envelope:

```json
{
  "specversion": "1.0",
  "type": "com.saaa.agent.decision.made",
  "source": "saaa/agent/voice-io",
  "id": "a3f5b2c1-...",
  "time": "2024-11-29T10:00:00Z",
  "datacontenttype": "application/json",
  "subject": "session/abc123",
  "data": {
    "agent_id": "voice-io-agent",
    "decision": "route_to_weather_agent",
    "confidence": 0.95,
    "cost_usd": 0.015
  },
  "traceparent": "00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01"
}
```

## AsyncAPI Contracts

```yaml
asyncapi: 3.0.0
info:
  title: SAAA Agent Events
  version: 1.0.0

channels:
  agent.decision.made:
    address: agent.decision.made
    messages:
      AgentDecisionMade:
        payload:
          type: object
          properties:
            agent_id:
              type: string
            decision:
              type: string
            cost_usd:
              type: number
```

⸻

# Regulatory Compliance & AI Governance

## Regulatory Landscape Overview

The platform is designed to support architectures aligned with:

| Region | Regulation | Status | Key Requirements |
|--------|-----------|--------|------------------|
| **U.S. Federal** | HIPAA/HITECH | Active | PHI encryption, BAA, breach notification |
| **U.S. Federal** | FTC Act § 5 | Active | Unfair/deceptive practices prohibition |
| **U.S. Federal** | FCRA/ECOA | Active | Credit decisioning fairness, explainability |
| **U.S. State** | IL BIPA | Active | Biometric consent, retention, deletion |
| **U.S. State** | CO AI Act | Enacted (delayed) | High-risk AI disclosure, bias testing |
| **U.S. State** | UT AI Policy Act | Active | State agency AI governance |
| **EU** | GDPR | Active | Data subject rights, DPIAs, cross-border |
| **EU** | AI Act | In Force (staged) | High-risk AI systems, transparency, oversight |
| **UK** | Data Protection Act | Active | UK GDPR equivalent |
| **Canada** | PIPEDA (post-C-27) | Active | Consent, breach notification |
| **Brazil** | LGPD | Active | Data protection authority, penalties |
| **Global** | ISO/IEC 42001 | Standard | AI management systems |
| **Global** | NIST AI RMF | Framework | Risk management lifecycle |

## AI Use-Case Risk Classification (EU AI Act Aligned)

```python
class AIRiskTier(Enum):
    PROHIBITED = "prohibited"  # Social scoring, manipulative AI
    HIGH = "high"              # Employment, credit, law enforcement
    LIMITED = "limited"        # Chatbots, emotion recognition (disclosure)
    MINIMAL = "minimal"        # Spam filters, recommendation engines
```

**Mapping to Platform**:
```python
class AgentGovernanceMetadata:
    risk_tier: AIRiskTier
    human_oversight: HumanOversightMode  # HITL, HOTL, NONE
    data_classifications_handled: List[DataClassification]
    regulatory_tags: List[str]  # ["HIPAA", "EU_AI_ACT_HIGH_RISK"]
    bias_test_passed: bool
    bias_test_date: datetime
    approval_authority: str
    approval_date: datetime
```

## Built-In Compliance Controls

### 1. Data Subject Rights (GDPR/CCPA)

```python
# Right to Access
async def export_user_data(user_id: UUID) -> UserDataExport:
    """Compile all user data across all levels"""
    return UserDataExport(
        user_profile=await state.get_user_data(user_id),
        conversation_history=await get_sessions(user_id),
        ai_decisions=await get_ai_decision_log(user_id),
        cost_history=await get_cost_summary(user_id),
    )

# Right to Erasure
async def delete_user_data(user_id: UUID) -> DeletionReport:
    """Delete user data while preserving audit logs"""
    deleted_count = 0
    deleted_count += await state.delete_level("user", user_id)
    deleted_count += await anonymize_ai_decisions(user_id)
    deleted_count += await delete_biometric_data(user_id)
    # Preserve audit logs (legal requirement)
    await audit_log.record_deletion(user_id, deleted_count)
    return DeletionReport(deleted_records=deleted_count)
```

### 2. Consent Management (BIPA)

```python
class ConsentToken:
    """Cryptographic consent token for biometric data"""
    user_id: UUID
    consent_type: str  # "voice_processing", "facial_recognition"
    granted_at: datetime
    expires_at: Optional[datetime]
    purpose: str
    signature: str  # HMAC-SHA256 of fields
    
    def is_valid(self) -> bool:
        return (
            self.expires_at is None or datetime.now() < self.expires_at
        ) and self.verify_signature()
```

### 3. AI Decision Logging & Explainability

```sql
CREATE TABLE ai_decision_log (
    decision_id UUID PRIMARY KEY,
    timestamp TIMESTAMPTZ DEFAULT NOW(),
    
    -- Context
    org_id UUID NOT NULL,
    business_unit_id UUID,
    user_id UUID,
    session_id UUID,
    
    -- Agent
    agent_id TEXT NOT NULL,
    agent_version TEXT NOT NULL,
    model_provider TEXT NOT NULL,
    model_name TEXT NOT NULL,
    
    -- Decision
    decision_type TEXT NOT NULL,  -- "route", "approve", "deny", "classify"
    input_summary TEXT,  -- Truncated/masked input
    output_summary TEXT,
    confidence_score DECIMAL(5, 4),
    
    -- Explainability
    reasoning_trace JSONB,  -- Chain of thought
    feature_importance JSONB,  -- What influenced the decision
    
    -- Governance
    risk_tier TEXT NOT NULL,
    human_reviewed BOOLEAN DEFAULT FALSE,
    human_reviewer_id UUID,
    human_review_timestamp TIMESTAMPTZ,
    
    -- Cost
    cost_usd DECIMAL(10, 6),
    tokens_used INTEGER,
    
    -- Trace
    trace_id TEXT NOT NULL,
    span_id TEXT NOT NULL,
    
    -- Compliance tags
    regulatory_tags TEXT[],
    
    PRIMARY KEY (decision_id)
) PARTITION BY RANGE (timestamp);

-- Retention: 7 years for regulated industries
```

### 4. Bias Testing & Monitoring

```python
class BiasTestSuite:
    """Required for EU AI Act high-risk systems"""
    
    async def test_demographic_parity(
        self,
        agent_id: str,
        test_dataset: List[BiasTestCase]
    ) -> BiasTestResult:
        """Test if outcomes are independent of protected attributes"""
        results = []
        for case in test_dataset:
            decision = await agent.make_decision(case.input)
            results.append((case.protected_attribute, decision.outcome))
        
        # Statistical analysis
        parity_score = calculate_demographic_parity(results)
        
        return BiasTestResult(
            agent_id=agent_id,
            test_type="demographic_parity",
            score=parity_score,
            passed=parity_score >= 0.80,  # 80% parity threshold
            tested_at=datetime.now()
        )
```

### 5. Human Oversight Modes

```python
class HumanOversightMode(Enum):
    NONE = "none"  # Minimal risk AI
    MONITORING = "monitoring"  # Human-on-the-loop (notification)
    APPROVAL = "approval"  # Human-in-the-loop (blocking)
    
async def enforce_oversight(
    agent_id: str,
    decision: AgentDecision
) -> AgentDecision:
    governance = await get_agent_governance(agent_id)
    
    if governance.human_oversight == HumanOversightMode.APPROVAL:
        # Block until human approves
        approval = await wait_for_human_approval(decision)
        if not approval.approved:
            raise DecisionRejectedError(approval.reason)
        decision.approved_by = approval.reviewer_id
        decision.approved_at = approval.timestamp
    
    elif governance.human_oversight == HumanOversightMode.MONITORING:
        # Notify but don't block
        await notify_humans(decision)
    
    return decision
```

## IT Service Management Framework Alignment

**Primary Framework**: **ITIL v4** (Information Technology Infrastructure Library)

The platform aligns operational processes with ITIL v4 practices:

| ITIL v4 Practice | Platform Implementation | Section Reference |
|------------------|-------------------------|-------------------|
| **Incident Management** | P0-P4 classification, SLA tracking, major incident process | Section 16 |
| **Problem Management** | Root cause analysis, permanent fixes, known error database | Section 16 |
| **Change Enablement** | Standard/Normal/Emergency changes, CAB, ECAB | Section 20 |
| **Release Management** | Environment promotion, blue/green deployments, PIR | Section 20 |
| **Service Level Management** | SLA/SLO definitions, performance targets | Section 18 (v1.4) |
| **Continual Improvement** | Post-incident reviews, retrospectives, metrics | Sections 16, 20 |

**Alternative/Complementary Frameworks**:
- **ISO/IEC 20000**: IT service management standard (compatible with ITIL)
- **COBIT**: Governance and management framework (enterprise-level)
- **NIST Cybersecurity Framework**: Incident response lifecycle (security-focused)

**Rationale**: ITIL v4 is the de facto standard for IT service management. We align with established practices rather than inventing new processes. This provides:
1. Common language with enterprise IT teams
2. Proven processes for incident and change management
3. Integration with existing ITSM tooling (ServiceNow, Jira Service Management)
4. Professional certifications and training materials available

**Adaptation for A2A Architecture**:
- Agent-aware incident classification (which agents affected?)
- AI cost impact in change risk assessment
- Model deployment as a formal change type (CAIO approval required)
- Bias testing as part of change validation gates

## Evidence & Equivalency Matrix

Maintain a mapping of:
- Regulatory requirement → Platform capability → Evidence artifact

```yaml
evidence_matrix:
  - regulation: HIPAA Security Rule § 164.312(a)(2)(iv)
    requirement: Encryption and decryption
    platform_capability: TLS 1.3, AES-256-GCM at rest
    evidence_artifacts:
      - config/encryption_policy.yaml
      - reports/penetration_test_2024Q4.pdf
      - logs/kms_audit_trail.json
  
  - regulation: EU AI Act Article 14
    requirement: Human oversight for high-risk AI
    platform_capability: HumanOversightMode.APPROVAL
    evidence_artifacts:
      - code/base_agent.py#L234
      - reports/bias_test_results.json
      - logs/ai_decision_log (human_reviewed=true)
  
  - itil_v4_practice: Change Enablement
    requirement: Risk-based change authorization
    platform_capability: CAB approval for normal changes, CAIO for AI
    evidence_artifacts:
      - itsm/change_records.json
      - itsm/cab_meeting_minutes.pdf
      - git/release_tags (immutable deployments)
```

⸻

# Security Architecture & Hardening (From Agent Foundry)

## Defense in Depth Model

```
┌─────────────────────────────────────────┐
│ EDGE TIER                               │
│  - WAF (ModSecurity, AWS WAF)           │
│  - DDoS protection (Cloudflare, AWS)    │
│  - TLS 1.3 termination                  │
│  - Rate limiting (per-tenant)           │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ SERVICE TIER                            │
│  - FastAPI with OIDC (Zitadel)          │
│  - OpenFGA authorization                │
│  - MCP gateway (authenticated)          │
│  - Agent workers (SPIRE identity)       │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ DATA TIER                               │
│  - PostgreSQL (TLS + RLS)               │
│  - Redis (TLS + AUTH)                   │
│  - S3 (SSE-KMS)                         │
│  - Secrets Manager (KMS-encrypted)      │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ TOOL TIER                               │
│  - LangGraph tools (schema-validated)   │
│  - MCP servers (mTLS)                   │
│  - External APIs (policy-governed)      │
└─────────────────────────────────────────┘
```

## Encryption Standards

### In Transit (TLS 1.3)
```yaml
tls_config:
  min_version: "1.3"  # TLS 1.3 mandatory
  cipher_suites:
    - TLS_AES_256_GCM_SHA384
    - TLS_CHACHA20_POLY1305_SHA256
  disable:
    - TLS 1.0, 1.1, 1.2
    - SSLv2, SSLv3
  features:
    - HSTS: max-age=31536000; includeSubDomains; preload
    - OCSP_stapling: true
    - Session_resumption: true
```

### At Rest (AES-256)
```yaml
encryption_at_rest:
  postgres:
    method: RDS_encryption
    algorithm: AES-256-GCM
    kms_key: alias/saaa-prod-db
  
  redis:
    at_rest: true
    algorithm: AES-256
  
  s3:
    sse: SSE-KMS
    kms_key: alias/saaa-prod-storage
    bucket_policy: enforce_encryption
  
  secrets_manager:
    kms_key: alias/saaa-prod-secrets
    rotation: 90_days
```

## Key & Secret Management

### Production (AWS)
```python
class SecretsClient:
    """Production: AWS Secrets Manager"""
    
    async def get_secret(self, secret_name: str) -> str:
        response = await self.secretsmanager_client.get_secret_value(
            SecretId=f"saaa/prod/{secret_name}"
        )
        # Secret is automatically decrypted by AWS KMS
        return response['SecretString']
    
    async def rotate_secret(self, secret_name: str):
        """Trigger automatic rotation"""
        await self.secretsmanager_client.rotate_secret(
            SecretId=f"saaa/prod/{secret_name}",
            RotationLambdaARN=self.rotation_lambda_arn
        )
```

### Development (LocalStack)
```python
class LocalSecretsClient:
    """Development: LocalStack (AWS-compatible)"""
    
    def __init__(self):
        self.client = boto3.client(
            'secretsmanager',
            endpoint_url='http://localstack:4566',  # LocalStack endpoint
            aws_access_key_id='test',
            aws_secret_access_key='test',
            region_name='us-east-1'
        )
    
    async def seed_dev_secrets(self):
        """Populate LocalStack with dev secrets"""
        await self.client.create_secret(
            Name='saaa/dev/deepgram_api_key',
            SecretString='dev_key_...'
        )
```

**Key Principle**: Code is identical in dev and prod; only endpoint URL differs.

## Network Segmentation & Zero Trust

### Network Tiers
```yaml
network_topology:
  edge_tier:
    - nginx_ingress
    - aws_alb
    allowed_inbound: [internet]
    allowed_outbound: [app_tier]
  
  app_tier:
    - fastapi_backend
    - mcp_gateway
    - agent_workers
    allowed_inbound: [edge_tier]
    allowed_outbound: [data_tier, external_apis]
  
  data_tier:
    - postgres
    - redis
    - zitadel
    - openfga
    allowed_inbound: [app_tier]
    allowed_outbound: []  # No internet access
```

### Zero Trust Implementation (SPIRE)
```yaml
spire_config:
  trust_domain: saaa.local
  
  workload_identities:
    - spiffe_id: spiffe://saaa.local/agent/voice-io
      selector: k8s:ns:saaa,k8s:sa:voice-io-agent
      ttl: 3600  # 1 hour, auto-rotated
    
    - spiffe_id: spiffe://saaa.local/agent/orchestrator
      selector: k8s:ns:saaa,k8s:sa:orchestrator-agent
      ttl: 3600
  
  policies:
    - source: spiffe://saaa.local/agent/orchestrator
      destination: spiffe://saaa.local/agent/voice-io
      allowed_operations: [call_tool, list_tools]
```

## LangGraph Security Patterns

### Tool Surface Hardening
```python
class SecureWeatherTool:
    """Example: Hardened tool with validation"""
    
    schema = {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "pattern": "^[a-zA-Z0-9\\s,.-]{1,100}$",  # Restrict input
                "maxLength": 100
            }
        },
        "required": ["location"],
        "additionalProperties": False  # Reject unexpected fields
    }
    
    async def execute(self, params: Dict) -> Dict:
        # 1. Schema validation (Pydantic)
        validated = WeatherToolInput(**params)
        
        # 2. Sanitization
        location = sanitize_location(validated.location)
        
        # 3. Rate limiting
        if not await rate_limiter.allow(f"weather:{location}"):
            raise RateLimitError("Too many requests")
        
        # 4. Execute safely
        result = await self.weather_api.get_current(location)
        
        # 5. Audit log
        await audit_log.record_tool_call(
            tool="weather/current",
            input={"location": location},
            output_summary="weather_data",
            cost_usd=0.001
        )
        
        return result
```

### Forbidden Tools in Production
```python
# ❌ NEVER in production:
class ForbiddenTools:
    - ArbitraryCodeExecutionTool  # exec(), eval()
    - ShellExecutionTool  # subprocess.run()
    - SQLInjectionRiskTool  # f"SELECT * FROM {user_input}"
    - FileSystemWriteTool  # Unrestricted file writes
    - NetworkScanTool  # Port scanning, probing
```

## MCP Hardening

```python
class SecuredMCPServer:
    """MCP server with authentication and authorization"""
    
    def __init__(self):
        self.auth_middleware = JWTAuthMiddleware()
        self.authz_client = OpenFGAClient()
    
    @require_auth
    async def handle_tool_call(
        self,
        caller_identity: Identity,
        tool_name: str,
        params: Dict
    ) -> Dict:
        # 1. Authorization check
        allowed = await self.authz_client.check(
            user=caller_identity.user_id,
            relation="can_execute",
            object=f"tool:{tool_name}"
        )
        if not allowed:
            raise AuthorizationError(f"User {caller_identity.user_id} cannot execute {tool_name}")
        
        # 2. Rate limiting (per-tenant)
        if not await rate_limiter.allow(caller_identity.org_id, tool_name):
            raise RateLimitError()
        
        # 3. Execute tool
        result = await self.execute_tool(tool_name, params)
        
        # 4. Audit log
        await audit_log.record_mcp_call(
            caller=caller_identity,
            tool=tool_name,
            params_hash=hash(json.dumps(params)),
            result_hash=hash(json.dumps(result))
        )
        
        return result
```

## FastAPI Security Baseline

```python
# security.py
from fastapi import FastAPI, Security
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from slowapi import Limiter
from slowapi.util import get_remote_address

app = FastAPI(
    # Disable docs in production
    docs_url=None if settings.environment == "prod" else "/docs",
    redoc_url=None if settings.environment == "prod" else "/redoc",
    
    # Security headers
    swagger_ui_parameters={"persistAuthorization": False}
)

# CORS (strict)
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,  # Not "*"
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
    max_age=3600
)

# Trusted hosts
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=settings.ALLOWED_HOSTS
)

# Rate limiting
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.post("/api/v1/agent/call")
@limiter.limit("100/minute")  # Per-IP
async def call_agent(
    request: Request,
    identity: Identity = Security(get_current_identity)  # OIDC JWT
):
    # Multi-tenant authz
    if not await authz.check(identity.user_id, "execute", f"agent:{request.agent_id}"):
        raise HTTPException(status_code=403, detail="Forbidden")
    
    # Execute
    result = await agent_executor.call(request.agent_id, request.params)
    
    return result
```

## AWS-Aligned Integration Model

```python
class AWSIntegrationLayer:
    """Abstract AWS services for environment independence"""
    
    def __init__(self, environment: EnvironmentType):
        if environment == EnvironmentType.DEV:
            # LocalStack endpoints
            self.endpoint_url = "http://localstack:4566"
        else:
            # Real AWS
            self.endpoint_url = None
        
        self.secrets_client = boto3.client(
            'secretsmanager',
            endpoint_url=self.endpoint_url
        )
        self.ssm_client = boto3.client(
            'ssm',
            endpoint_url=self.endpoint_url
        )
        self.s3_client = boto3.client(
            's3',
            endpoint_url=self.endpoint_url
        )
    
    async def get_secret(self, name: str) -> str:
        """Same code for dev and prod"""
        response = await self.secrets_client.get_secret_value(
            SecretId=f"saaa/{self.environment}/{name}"
        )
        return response['SecretString']
```

## Supply Chain Security

```yaml
supply_chain:
  sbom_generation:
    tool: syft
    format: cyclonedx
    output: sbom.json
    
  dependency_scanning:
    - tool: snyk
      frequency: daily
      fail_on: high_severity
    
    - tool: safety  # Python
      command: safety check --json
    
    - tool: npm_audit  # Node.js
      command: npm audit --audit-level=high
  
  container_scanning:
    - tool: trivy
      scan_targets: [os_packages, app_dependencies]
      severity_threshold: CRITICAL
    
  code_scanning:
    - tool: semgrep
      rulesets: [owasp-top-10, security-audit]
    
    - tool: bandit  # Python
      config: .bandit.yaml
```

⸻

# Missing Sections (Headings + Skeleton for v1.4+)

**v1.4 Prioritization (Recommended)**:
1. 🔴 **Section 17** (Disaster Recovery) - Critical for production commitments
2. 🔴 **Section 18** (Performance & Scalability) - Needed for SLAs
3. 🟡 **Section 21** (Developer Experience) - High productivity impact
4. 🟡 **Section 16** (Incident Response) - Regulatory requirement
5. 🟢 **Section 19, 20, 22** (Vendor, Change, Endpoint) - Lower urgency

---

## 16. Incident Response & Breach Notification (ITIL v4 Aligned)

**Scope**: Align platform incident management with **ITIL v4 Incident Management** and **Problem Management** practices.

**Framework Alignment**:
- **ITIL v4 Incident Management**: Restore normal service operation as quickly as possible
- **ITIL v4 Problem Management**: Root cause analysis and permanent fixes
- **NIST Cybersecurity Framework**: Incident response lifecycle (Identify → Protect → Detect → Respond → Recover)
- **Regulatory**: HIPAA breach notification (60 days), GDPR (72 hours), state breach laws

**ITIL v4 Integration**:
```yaml
itil_v4_practices:
  incident_management:
    priority_matrix: # Impact × Urgency
      P0_critical: "Service down, revenue impact >$10K/hr"
      P1_high: "Major function unavailable, workaround exists"
      P2_medium: "Service degraded, <100 users affected"
      P3_low: "Minor issue, no business impact"
    
    sla_targets:
      P0: response_15min, resolution_4hours
      P1: response_1hour, resolution_24hours
      P2: response_4hours, resolution_3days
      P3: response_24hours, resolution_7days
  
  problem_management:
    trigger: "3+ incidents with same root cause"
    rca_required: true
    permanent_fix_tracking: true
  
  major_incident_management:
    trigger: "P0 or security incident"
    war_room: required
    stakeholders: [CTO, CIO, CISO, affected_customers]
```

**Platform-Specific Extensions**:
- **AI Incident Classification**: Auto-detect anomalies via cost spikes, error rate changes
- **Agent-Aware Runbooks**: Include which agents affected, recent deployments
- **Compliance Notification**: Auto-trigger breach workflows if PHI/PII exposed

**Tooling Integration**:
- SIEM → PagerDuty/Opsgenie (ITIL incident creation)
- Audit logs → Forensic analysis (ITIL problem investigation)
- Cost anomaly detection → Potential security indicator
- ServiceNow/Jira Service Management for ITSM workflows

**Reference**: ITIL v4 Foundation, ITIL v4 Specialist: High-velocity IT

**Detailed content**: v1.4 will provide runbooks aligned to ITIL v4 templates

## 17. Disaster Recovery & Business Continuity ✅ DETAILED

**Scope**: Ensure platform availability during regional failures, maintain business operations with defined recovery objectives.

**Framework**: ITIL v4 Service Continuity Management + NIST SP 800-34 (Contingency Planning)

### 17.1 Recovery Metrics by Tier

**Production (Customer-Facing)**:
```yaml
production:
  rpo: 15 minutes  # Maximum acceptable data loss
  rto: 4 hours     # Maximum acceptable downtime
  mtr: 8 hours     # Maximum time to restore (worst case)
  availability_target: 99.9%  # Three nines (8.76 hours/year downtime)
```

**Staging (Pre-Production)**:
```yaml
staging:
  rpo: 4 hours
  rto: 8 hours
  mtr: 24 hours
  availability_target: 99.0%
```

**Development/Test**:
```yaml
dev_test:
  rpo: 24 hours
  rto: 48 hours
  mtr: N/A  # Can rebuild from scratch
  availability_target: 95.0%
```

**Priority Matrix**: If multiple customers affected, recover in order:
1. Contractual RTO commitments (lowest first)
2. Revenue impact (highest first)
3. Number of users affected
4. Regulatory/compliance requirements

### 17.2 Disaster Declaration Criteria

**Disaster Declared If**:
```yaml
disaster_triggers:
  multi_az_failure:
    condition: "2+ Availability Zones offline in primary region"
    duration: ">4 hours OR not recoverable within 4 hours"
    action: "Declare disaster, initiate DR failover"
  
  regional_outage:
    condition: "Entire AWS region offline or degraded"
    duration: ">4 hours"
    action: "Declare disaster, initiate DR failover"
  
  data_center_failure:
    condition: "Complete data center loss (fire, natural disaster)"
    duration: "Immediate"
    action: "Declare disaster, initiate DR failover"
  
  critical_service_failure:
    condition: "Core services unrecoverable (DB corruption, ransomware)"
    duration: ">RTO threshold per tier"
    action: "Declare disaster, restore from backup"
```

**Disaster Declaration Authority**: CTO or designated on-call lead

**Communication Protocol**:
1. **Immediate** (within 15 minutes of declaration):
   - Internal: CTO → CEO, COO, CISO, CIO, CAIO
   - Internal: Engineering war room established
   
2. **Within 1 hour**:
   - External: Affected customers via email (account managers + support team)
   - Status page updated (https://status.platform.com)
   - Major incident declared in ITSM (ITIL v4 Major Incident Management)

3. **Ongoing** (every 2 hours):
   - Status updates to customers
   - Progress reports to executive team

### 17.3 Multi-Region Replication Strategy

**AWS Primary → Secondary Regions**:
```yaml
aws_replication:
  primary_regions:
    us_east_1:
      secondary: us_west_2
      replication:
        - rds_read_replicas
        - s3_cross_region_replication
        - ebs_snapshot_copy
        - secrets_manager_replication
        - parameter_store_replication
    
    eu_central_1:
      secondary: eu_north_1
      replication:
        - rds_read_replicas
        - s3_cross_region_replication
        - ebs_snapshot_copy
        - secrets_manager_replication
    
    ap_northeast_1:
      secondary: ap_southeast_1
      replication:
        - rds_read_replicas
        - s3_cross_region_replication
        - ebs_snapshot_copy

  data_residency_enforcement:
    us_data: [us_east_1, us_west_2]
    eu_data: [eu_central_1, eu_north_1]
    ap_data: [ap_northeast_1, ap_southeast_1]
```

**Cross-Cloud Strategy (Optional)**:
```yaml
# For extreme resilience (higher cost)
aws_to_gcp:
  us_east_1_aws: us_east4_gcp  # Virginia
  eu_central_1_aws: europe_west1_gcp  # Frankfurt → Belgium
```

### 17.4 Backup Processes

**Automated Snapshot Schedule**:
```yaml
snapshots:
  compute:
    ec2_ebs_volumes:
      frequency: every_1_hour
      retention: 24_hours
      replication: cross_region
      lifecycle: delete_after_7_days (secondary region)
    
    kubernetes_persistent_volumes:
      frequency: every_1_hour
      retention: 24_hours
      tool: velero
  
  databases:
    postgres_rds:
      automated_backups: daily_4am_utc
      snapshot_frequency: every_6_hours
      retention: 30_days
      point_in_time_recovery: enabled (5_minute_granularity)
      cross_region_replica: enabled
    
    redis:
      rdb_snapshot: every_1_hour
      aof_enabled: true  # Append-only file for durability
      retention: 7_days
      replica_mode: async (secondary region)
  
  object_storage:
    s3_buckets:
      versioning: enabled
      cross_region_replication: enabled
      replication_time_control: 15_minutes
      lifecycle:
        - archive_to_glacier_after: 90_days
        - delete_after: 7_years (compliance)
  
  configuration:
    secrets_manager:
      replication: automatic (multi_region)
      rotation: 90_days
    
    parameter_store:
      backup: nightly (exported to S3)
      retention: 90_days
    
    infrastructure_as_code:
      git_repository: github.com/org/platform-infra
      backup: github_enterprise_backup
      retention: indefinite
```

**Application-Specific Backups**:
```yaml
platform_backups:
  agent_registry:
    backend: postgres
    frequency: every_1_hour
    retention: 7_days
    export_format: json_dump (S3)
  
  event_bus:
    kafka_topic_snapshots:
      frequency: every_6_hours
      retention: 7_days (audit topics: 7_years)
      tool: kafka_connect_s3_sink
    
    nats_jetstream:
      snapshot_frequency: every_6_hours
      retention: 7_days
      backend: s3
  
  ai_decision_logs:
    backend: postgres (partitioned by date)
    snapshot: daily
    retention: 7_years (compliance)
    cross_region_replication: enabled
  
  cost_tracking_data:
    backend: postgres + s3 (archival)
    snapshot: daily
    retention: 7_years (financial records)
```

**Backup Validation**:
- **Automated**: Weekly restore test in isolated environment
- **Manual**: Quarterly full DR drill (see 17.7)
- **Monitoring**: Backup failures trigger PagerDuty alerts

### 17.5 Recovery Processes

**Phase 1: Disaster Declaration & Communication** (0-15 minutes)
```bash
# 1. Declare disaster (CTO or on-call lead)
./scripts/declare-disaster.sh --region us-east-1 --reason "multi-az-failure"

# 2. Establish war room
# - Zoom/Teams bridge URL auto-generated
# - Key stakeholders notified (PagerDuty)
# - ITSM major incident created

# 3. Customer communication
./scripts/notify-customers.sh --affected-region us-east-1 --status "disaster-declared"
```

**Phase 2: Assess & Decide** (15-30 minutes)
```yaml
decision_tree:
  is_primary_region_recoverable:
    yes:
      action: "Wait and monitor (if <4 hours estimated)"
      fallback: "Proceed to DR failover if estimate increases"
    no:
      action: "Proceed to DR failover immediately"
  
  data_loss_risk:
    postgres_replica_lag: "<5 minutes → proceed"
    kafka_replication_lag: "<15 minutes → proceed"
    redis_last_backup: "<1 hour → proceed"
```

**Phase 3: Execute DR Failover** (30 minutes - 4 hours)

**Pre-Failover Checklist**:
```yaml
preflight_checks:
  - [ ] Verify secondary region availability
  - [ ] Confirm RDS read replica is in sync (<5 min lag)
  - [ ] Validate S3 replication status (versioning enabled)
  - [ ] Check Kafka/NATS replication lag
  - [ ] Verify DNS can be updated (Route53/CloudFlare access)
  - [ ] Confirm Secrets Manager replication complete
  - [ ] War room established with key personnel
  - [ ] Customer communication sent
```

**Automated DR Playbook** (Terraform + Ansible):
```bash
# Engineer checks out latest DR playbooks
cd /ops/dr-playbooks
git pull origin main

# Set target DR region
export DR_REGION="us-west-2"
export PRIMARY_REGION="us-east-1"
export ENVIRONMENT="prod"

# Execute DR failover
./dr-failover.sh --primary $PRIMARY_REGION --secondary $DR_REGION --env $ENVIRONMENT

# Playbook performs:
# 1. Deploy/verify VPC infrastructure (Terraform)
# 2. Promote RDS read replica to primary
# 3. Deploy compute resources (EKS/ECS/EC2)
# 4. Restore Redis from latest snapshot
# 5. Deploy agent containers from ECR
# 6. Initialize agent registry from backup
# 7. Configure Kafka/NATS in DR region
# 8. Update DNS (Route53) to point to DR region
# 9. Update monitoring dashboards (Grafana)
# 10. Deploy observability stack (OTEL collector, Prometheus)
# 11. Validate health checks
```

**Manual Steps** (during automated deployment):
```yaml
manual_tasks:
  database_validation:
    - Check RDS replica promotion status
    - Verify database connectivity
    - Validate row counts match expected
    - Check replication lag is 0
  
  event_bus_recovery:
    - Kafka: Verify consumer groups rebalanced
    - NATS: Verify streams available and consumers connected
    - Check event replay requirements (if needed)
  
  agent_registry:
    - Verify all agents registered
    - Check agent health endpoints
    - Validate MCP connectivity
  
  secrets_validation:
    - Confirm Secrets Manager accessible in DR region
    - Spot-check critical secrets (API keys)
  
  cost_tracking:
    - Verify cost event pipeline operational
    - Check business unit dashboards updating
```

**Phase 4: Validation & Customer Notification** (4-6 hours)

**QA Validation Checklist**:
```yaml
validation_tests:
  smoke_tests:
    - [ ] Frontend loads (all environments)
    - [ ] User can log in (Zitadel)
    - [ ] API health checks pass
    - [ ] Agent registry accessible
  
  functional_tests:
    - [ ] Create test agent
    - [ ] Execute MCP tool call
    - [ ] Trigger voice pipeline (if applicable)
    - [ ] Verify cost tracking events emitted
    - [ ] Check AI decision logging
  
  integration_tests:
    - [ ] External integrations working (webhooks)
    - [ ] Event bus consumers processing
    - [ ] Database queries performing normally
    - [ ] S3 object storage accessible
  
  performance_tests:
    - [ ] API latency within SLA (p95 < 500ms)
    - [ ] Database query performance normal
    - [ ] Agent response time acceptable
  
  compliance_tests:
    - [ ] Audit logs flowing
    - [ ] Encryption at rest verified (KMS)
    - [ ] Data residency maintained (EU data in EU region)
```

**Customer Communication** (after validation):
```
Subject: [RESOLVED] Platform Recovered - Action Required

Dear Customer,

We have successfully recovered the platform to our disaster recovery region 
after a regional outage in [PRIMARY_REGION].

Your data is safe and the platform is fully operational.

ACTION REQUIRED:
1. Clear your DNS cache (ipconfig /flushdns on Windows, sudo dscacheutil -flushcache on Mac)
2. Restart any integration clients (if applicable)
3. Test your workflows and report any issues to support@platform.com

We will continue monitoring for 24 hours before returning to normal operations.

Sincerely,
Platform Operations Team
```

### 17.6 Failback Processes

**Timing**: Scheduled maintenance window (typically 7-14 days after DR event)

**Pre-Failback Checklist**:
```yaml
failback_prerequisites:
  - [ ] Primary region fully recovered and stable (>7 days)
  - [ ] Root cause analysis (RCA) completed
  - [ ] Preventive measures implemented
  - [ ] Customer notification sent (72-hour advance notice)
  - [ ] Scheduled maintenance window approved
  - [ ] Rollback plan documented
```

**Failback Procedure**:
```bash
# 1. Pre-failback snapshot (safety)
./scripts/pre-failback-snapshot.sh --region $DR_REGION

# 2. Put systems into maintenance mode
kubectl annotate deployments --all maintenance="true"
./scripts/maintenance-mode.sh --enable

# 3. Final database backup from DR region
aws rds create-db-snapshot --db-instance-identifier prod-dr --db-snapshot-identifier final-dr-snapshot

# 4. Sync DR data back to primary region
./scripts/sync-dr-to-primary.sh --source $DR_REGION --target $PRIMARY_REGION

# 5. Deploy infrastructure to primary region
cd /ops/terraform
terraform apply -var region=$PRIMARY_REGION -var environment=prod

# 6. Restore database to primary region
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier prod-primary \
  --db-snapshot-identifier final-dr-snapshot

# 7. Update DNS back to primary region
./scripts/update-dns.sh --region $PRIMARY_REGION

# 8. Validate health checks in primary region
./scripts/validate-health.sh --region $PRIMARY_REGION

# 9. Disable maintenance mode
./scripts/maintenance-mode.sh --disable

# 10. Customer notification
./scripts/notify-customers.sh --status "returned-to-normal"
```

**Post-Failback Validation** (24-hour soak test):
```yaml
soak_test:
  duration: 24_hours
  monitoring:
    - Error rates (should be <0.1%)
    - Latency p95 (should be within SLA)
    - Database replication lag (should be <5 seconds)
    - Cost anomalies (check for runaway costs)
  
  escalation:
    - If error rate >1%: Immediate investigation
    - If latency >2x SLA: Scale up resources
    - If cost >2x normal: Check for resource leaks
```

**DR Region Decommission** (after 24-hour soak):
```bash
# After confirming primary region stable
./scripts/decommission-dr.sh --region $DR_REGION --environment prod

# Keeps:
# - Snapshots (for future DR events)
# - Backups (per retention policy)
# Deletes:
# - Compute resources (EC2, EKS)
# - Running databases (replica kept)
# - NAT gateways, load balancers
```

### 17.7 DR Testing Schedule

**Quarterly DR Drills** (Production):
```yaml
q1_drill:
  date: "Last Saturday of Q1 (02:00-06:00 UTC)"
  scope: "Simulate regional outage, full failover"
  success_criteria:
    - RTO met (<4 hours)
    - RPO met (<15 minutes data loss)
    - All validation tests pass
    - Customer communication sent on schedule
  
q2_drill:
  date: "Last Saturday of Q2"
  scope: "Database corruption recovery"
  success_criteria:
    - Restore from backup within RTO
    - Point-in-time recovery successful
  
q3_drill:
  date: "Last Saturday of Q3"
  scope: "Cross-cloud failover (AWS → GCP)"
  success_criteria:
    - Demonstrate cloud provider independence
  
q4_drill:
  date: "Last Saturday of Q4"
  scope: "Failback procedures"
  success_criteria:
    - Return to primary region within maintenance window
    - Zero data loss during failback
```

**Monthly Backup Validation**:
- Automated restore tests (isolated environment)
- Verify backup integrity
- Measure restore time

**Annual Tabletop Exercise**:
- Executive team + key engineers
- Walk through disaster scenarios
- Update runbooks based on findings
- Review and update RTO/RPO targets

### 17.8 Cost Considerations

**DR Cost Model**:
```yaml
dr_costs:
  normal_operations:
    standby_infrastructure:
      rds_read_replicas: 500_usd_month
      s3_cross_region_replication: 200_usd_month
      snapshot_storage: 150_usd_month
      secrets_replication: 50_usd_month
    total_monthly: 900_usd
  
  during_dr_event:
    full_environment_running:
      compute: 5000_usd_month (pro-rated to duration)
      database: 1000_usd_month
      networking: 500_usd_month
      data_transfer: 2000_usd
    estimated_8_hour_dr: 2500_usd (pro-rated)
  
  annual_dr_budget:
    standby_infrastructure: 10800_usd
    quarterly_drills: 10000_usd (4 × 2500)
    contingency_reserve: 20000_usd (8 DR events/year worst case)
    total: 40800_usd_year
```

**Cost Optimization**:
- Use spot instances for DR drills (not production DR)
- RDS read replicas only in critical regions
- S3 Intelligent-Tiering for backup storage
- Automate DR resource decommissioning

### 17.9 Integration with ITIL v4

**DR as Major Incident**:
```yaml
itil_v4_integration:
  incident_classification: "P0 - Major Incident"
  major_incident_team:
    - Incident Manager: CTO or designated lead
    - Technical Lead: Principal Engineer
    - Communications Lead: VP Engineering
    - Customer Liaison: Account Managers
  
  incident_record:
    title: "Regional Disaster - [REGION] Unavailable"
    impact: "Critical - All customers in region affected"
    urgency: "High - Revenue and SLA impact"
    priority: "P0"
    estimated_resolution_time: "4-8 hours"
  
  post_incident_review:
    timeline: "Within 72 hours of resolution"
    attendees: [CTO, CISO, CIO, Engineering Team]
    outputs:
      - Root cause analysis
      - Timeline of events
      - What went well / what didn't
      - Action items for improvement
```

### 17.10 Regulatory & Compliance

**Breach Notification** (if data loss):
- GDPR: 72 hours to notify supervisory authority
- HIPAA: 60 days for non-breach events, immediate for breaches
- State laws: Vary by jurisdiction

**Audit Trail**:
```yaml
audit_requirements:
  dr_events:
    - Disaster declaration timestamp and authority
    - Communication logs (internal and external)
    - All commands executed during recovery
    - Validation test results
    - Failback procedures and results
  
  retention: 7_years (compliance)
  storage: Encrypted S3 bucket (audit-logs)
```

### 17.11 Platform-Specific Considerations

**Agent Registry Failover**:
```python
# Agent registry automatically discovers new region
class AgentRegistry:
    async def failover_to_region(self, dr_region: str):
        # 1. Update service discovery (Consul/etcd)
        await self.service_discovery.update_region(dr_region)
        
        # 2. Re-register all agents in DR region
        for agent in await self.get_all_agents():
            await self.register_agent_in_region(agent, dr_region)
        
        # 3. Update DNS for MCP endpoints
        await self.dns_client.update_records(dr_region)
```

**Event Bus Recovery**:
```yaml
kafka_dr:
  mirrormaker2:
    enabled: true
    source_cluster: primary_region
    target_cluster: dr_region
    replication_lag_max: 15_minutes
    topics:
      - agent.* (all agent events)
      - cost.* (cost tracking)
      - audit.* (compliance - 7 year retention)
  
  consumer_group_rebalancing:
    automatic: true
    timeout: 5_minutes
```

### 17.12 Runbook Repository

**Location**: `github.com/org/platform-dr-runbooks`

**Structure**:
```
dr-runbooks/
├── README.md (this section)
├── declare-disaster.sh
├── dr-failover.sh
├── validate-health.sh
├── failback.sh
├── decommission-dr.sh
├── terraform/
│   ├── dr-us-west-2/
│   ├── dr-eu-north-1/
│   └── dr-ap-southeast-1/
├── ansible/
│   ├── deploy-agents.yml
│   ├── restore-database.yml
│   └── configure-monitoring.yml
└── docs/
    ├── quarterly-drill-checklist.md
    ├── customer-communication-templates.md
    └── post-incident-review-template.md
```

**Access Control**:
- Runbook repo: SRE team + on-call engineers
- Secrets: AWS Secrets Manager (cross-region replicated)
- Emergency access: Break-glass procedure documented

---

**References**:
- ITIL v4 Service Continuity Management
- NIST SP 800-34 Rev. 1 (Contingency Planning Guide)
- AWS Well-Architected Framework (Reliability Pillar)
- ISO/IEC 27031:2011 (ICT readiness for business continuity)

## 18. Performance & Scalability Targets ✅ DETAILED

**Scope**: Define production-grade performance SLAs, capacity planning, and horizontal scaling strategies based on real-world deployments.

**Framework**: Based on proven enterprise conversational AI production architectures from multi-tenant SaaS deployments (2022-2024)

### 18.1 Performance SLA Targets (Customer-Facing)

**What Customers Actually Care About** (Field-Validated):
1. **Response Time** - Sub-second user experience
2. **Uptime** - "Five nines" or close
3. **Tenant Isolation** - No cross-tenant performance impact
4. **Cost Predictability** - No surprise scaling costs

**Standard SLA Tiers**:

```yaml
sla_tiers:
  standard:
    response_time_p95: 2000ms  # 95th percentile
    response_time_p99: 3000ms  # 99th percentile
    uptime: 99.95%  # 43 minutes/month downtime
    concurrent_users: 200-500 per org
    contractual_commitment: "Best effort"
    
  premium:
    response_time_p95: 1500ms  # Contractual commitment
    response_time_p99: 2000ms
    uptime: 99.95%
    concurrent_users: 500-2000 per org
    contractual_commitment: "SLA credits if violated"
    
  enterprise:
    response_time_p95: 1000ms  # High-frequency trading, real-time voice
    response_time_p99: 1500ms
    uptime: 99.995%  # 26 seconds/month (global financial services, $15B+ revenue)
    concurrent_users: 2000+ per org
    contractual_commitment: "SLA credits + penalty clauses"
    support: "24/7 dedicated TAM"
```

**Latency Budget Breakdown** (A2A Voice Pipeline Target: <2000ms):
```yaml
voice_pipeline_latency:
  vad_detection: 180ms  # Voice Activity Detection (Silero)
  stt_transcription: 280ms  # Speech-to-Text (Deepgram Nova-2)
  orchestrator_routing: 50ms  # Agent registry lookup + MCP routing
  llm_first_token: 800ms  # LLM inference (Claude Sonnet 4)
  tts_synthesis: 380ms  # Text-to-Speech (ElevenLabs Turbo)
  network_overhead: 100ms  # WebRTC + buffering
  
  total_p50: 1790ms  # Median (within SLA)
  total_p95: 2100ms  # 95th percentile (just over SLA)
  optimization_target: "<1500ms p95" # Premium tier
```

**API Latency Targets** (Non-Voice):
```yaml
api_latency:
  read_operations:
    p50: 50ms
    p95: 200ms
    p99: 500ms
    
  write_operations:
    p50: 100ms
    p95: 300ms
    p99: 800ms
    
  agent_tool_calls:
    p50: 150ms  # Local tools (state management, routing)
    p95: 500ms
    p99: 1000ms
    
  llm_inference:
    p50: 800ms  # First token (Claude Sonnet)
    p95: 1200ms
    p99: 2000ms
    
  mcp_remote_calls:
    p50: 200ms  # Agent-to-agent (includes network)
    p95: 600ms
    p99: 1200ms
```

### 18.2 Capacity Planning (Reference Production Architecture)

**3-Node Cluster Baseline** (Standard Deployment):
```yaml
cluster_configuration:
  node_specs:
    vcpu: 48  # Azure F48s_v2 or AWS c5.metal equivalent
    ram: 96GB
    storage: 512GB Premium SSD (2300 IOPS minimum, P20 tier)
    network: 25Gbps
    
  capacity_per_node:
    concurrent_conversations: 200-225  # At 2sec p95 latency
    api_requests_per_second: 500
    agent_calls_per_second: 100
    
  cluster_total:
    nodes: 3  # HA + fault tolerance
    concurrent_conversations: 600-675
    api_requests_per_second: 1500
    failover_capacity: "N-1 (survive 1 node failure)"
```

**Scaling Triggers** (Horizontal Pod Autoscaling):
```yaml
hpa_rules:
  scale_up_triggers:
    cpu_utilization: ">70% sustained for 5 minutes"
    memory_utilization: ">80%"
    response_time_p95: ">2500ms for 3 minutes"
    queue_depth: ">1000 messages"
    error_rate: ">1% for 5 minutes"
    
  scale_down_triggers:
    cpu_utilization: "<40% for 15 minutes"
    memory_utilization: "<60%"
    response_time_p95: "<1000ms for 15 minutes"
    min_replicas: 3  # Never scale below 3 (HA requirement)
    
  scaling_velocity:
    scale_up: "+1 replica per 2 minutes (max +5 at once)"
    scale_down: "-1 replica per 10 minutes (gradual)"
    cooldown_period: 5_minutes
```

**Voice Infrastructure Capacity** (Per-Component):
```yaml
voice_components:
  voice_gateway:
    vcpu: 8
    ram: 16GB
    concurrent_calls: 250 per instance
    scale_trigger: ">200 active calls"
    min_instances: 3  # HA across availability zones
    
  stt_service:
    provider: Deepgram Nova-2
    vcpu: 8
    ram: 16GB
    concurrent_streams: 100 per instance
    scale_trigger: ">80 active streams"
    min_instances: 2
    
  tts_service:
    provider: ElevenLabs Turbo v2.5
    vcpu: 8
    ram: 16GB
    concurrent_syntheses: 200 per instance
    scale_trigger: ">160 active syntheses"
    min_instances: 2
```

### 18.3 Throughput Targets

**API Gateway**:
```yaml
api_throughput:
  target_rps: 10000  # Requests per second (cluster-wide)
  peak_rps: 50000  # Burst capacity (10 seconds)
  sustained_peak: 25000  # 1 hour sustained
  
  rate_limiting:
    per_organization:
      standard: 100_rps
      premium: 500_rps
      enterprise: 2000_rps
    
    per_user:
      standard: 10_rps
      premium: 50_rps
      enterprise: 100_rps
```

**Event Bus** (Kafka/Redpanda):
```yaml
event_throughput:
  target_messages_per_second: 10000
  peak_messages_per_second: 50000
  retention: 7_days (standard), 7_years (audit topics)
  
  partitions:
    agent_events: 24  # For parallelism
    cost_events: 12
    audit_events: 6  # Lower volume, compliance-focused
  
  replication_factor: 3
  min_in_sync_replicas: 2
```

**Database (PostgreSQL)**:
```yaml
database_throughput:
  target_queries_per_second: 5000
  peak_queries_per_second: 15000
  
  connection_pool:
    min_connections: 50
    max_connections: 500
    idle_timeout: 10_minutes
    
  read_replica_strategy:
    replicas: 2  # Per primary
    lag_threshold: 5_seconds  # Alert if exceeds
    failover_time: "<30 seconds"
```

### 18.4 Concurrency Limits

**Agent Concurrency**:
```yaml
agent_limits:
  per_agent_instance:
    max_concurrent_tool_calls: 100
    max_queue_depth: 500
    timeout: 30_seconds
    
  per_organization:
    standard: 50_concurrent_agents
    premium: 200_concurrent_agents
    enterprise: 1000_concurrent_agents
    
  throttling:
    burst_allowance: 2x_limit_for_10_seconds
    rate_limit_response: 429_with_retry_after_header
```

**Database Connection Limits**:
```yaml
database_connections:
  per_service:
    backend_api: 100
    agent_workers: 50
    orchestrator: 50
    voice_io: 30
    
  total_cluster: 500  # Must not exceed PostgreSQL max_connections
  
  connection_pooling:
    enabled: true
    library: pgbouncer
    mode: transaction  # Not session (more efficient)
```

### 18.5 Storage Performance Requirements

**IOPS Requirements** (Production-Validated):
```yaml
storage_tiers:
  production_vms:
    iops_per_vm: 2500  # Minimum sustained
    workload: "80% write / 20% read"
    latency: "<10ms p99"
    throughput: "100MB/sec sustained write"
    filesystem: XFS
    
  database_volumes:
    iops: 5000  # Premium SSD (P20+)
    latency: "<5ms p99"
    throughput: "250MB/sec"
    
  object_storage:
    s3_performance: "3500 PUT/sec, 5500 GET/sec per prefix"
    lifecycle: "Archive to Glacier after 90 days"
```

**Disk Space Planning**:
```yaml
storage_capacity:
  per_node:
    boot_disk: 50GB
    app_disk: 500GB  # Logs, temp files
    database: "Calculated based on retention"
    
  database_sizing:
    base: 100GB  # Platform + agent registry
    per_100k_conversations: 50GB
    ai_decision_logs: "10GB per million decisions"
    cost_tracking: "5GB per million events"
    retention: 7_years (compliance)
    
  s3_object_storage:
    transcripts: "1MB per hour of voice"
    recordings: "10MB per hour (compressed)"
    backups: "Full database size × 2 (current + previous)"
```

### 18.6 Network Bandwidth

**Bandwidth Requirements**:
```yaml
network:
  per_node:
    minimum: 10Gbps
    preferred: 25Gbps
    burst: 40Gbps (short duration)
    
  voice_traffic:
    per_call: 64kbps (G.711 codec)
    250_concurrent_calls: 16Mbps
    overhead: 2x (WebRTC, RTP, encryption)
    total_per_gateway: 32Mbps
    
  inter_node:
    database_replication: 1Gbps sustained
    kafka_replication: 2Gbps sustained
    redis_replication: 500Mbps
    
  internet_egress:
    llm_api_calls: "Variable (model-dependent)"
    stt_streaming: "16kHz × 2 bytes × concurrent"
    monitoring_telemetry: 100Mbps
```

### 18.7 Database Sharding Strategy

**When to Shard**:
```yaml
sharding_triggers:
  table_size: ">10 million rows"
  query_latency: "p95 >500ms"
  connection_pool_saturation: ">80%"
  storage: ">1TB per table"
```

**Sharding Keys**:
```yaml
sharding_strategy:
  by_organization:
    key: org_id
    benefits: "Perfect tenant isolation"
    drawback: "Cross-org queries impossible"
    use_case: "Multi-tenant SaaS (default)"
    
  by_time:
    key: created_at (monthly partitions)
    benefits: "Easy archival, query performance"
    use_case: "AI decision logs, cost events"
    
  hybrid:
    key: org_id + created_at
    benefits: "Isolation + time-based pruning"
    use_case: "Large enterprise deployments"
```

**Partition Management**:
```sql
-- Example: Monthly partitions for AI decision logs
CREATE TABLE ai_decision_log_2024_11 PARTITION OF ai_decision_log
    FOR VALUES FROM ('2024-11-01') TO ('2024-12-01');

-- Auto-create future partitions (pg_partman)
SELECT create_parent('ai_decision_log', 'occurred_at', 'native', 'monthly');
```

### 18.8 Caching Strategy

**Redis Caching Layers**:
```yaml
caching:
  l1_application_cache:
    ttl: 60_seconds
    scope: In-memory per service
    use_case: "API responses, agent registry lookups"
    
  l2_redis_cache:
    ttl: 5_minutes
    scope: Cluster-wide (Redis)
    use_case: "User sessions, agent state, cost aggregations"
    
  l3_cdn_cache:
    ttl: 1_hour
    scope: CloudFlare / CloudFront
    use_case: "Static assets, public API docs"
    
  no_cache:
    - Real-time agent decisions
    - Cost tracking events (must be accurate)
    - Audit logs (immutable)
```

**Cache Invalidation**:
```yaml
invalidation_strategies:
  time_based:
    ttl: "Set per cache layer"
    use_case: "Most reads"
    
  event_driven:
    trigger: "Kafka event (e.g., agent.updated)"
    use_case: "Agent registry, model configs"
    
  explicit:
    api: "POST /cache/invalidate"
    use_case: "Manual admin operations"
```

### 18.9 Auto-Scaling Configuration

**Kubernetes HPA** (Horizontal Pod Autoscaler):
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    - type: Pods
      pods:
        metric:
          name: http_request_duration_seconds_p95
        target:
          type: AverageValue
          averageValue: "2"  # 2 seconds
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Percent
          value: 50  # Scale up by 50% at a time
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300  # Wait 5 min before scale down
      policies:
        - type: Pods
          value: 1  # Scale down 1 pod at a time
          periodSeconds: 600  # Every 10 minutes
```

**Cost Impact of Scaling** (2025 Pricing):
```yaml
scaling_cost_analysis:
  baseline_3_nodes:
    monthly_cost: 8400_usd  # Updated 2025 pricing (+10% infrastructure)
    concurrent_conversations: 600
    cost_per_conversation: 14.00_usd
    breakdown:
      compute_eks: 550_usd
      database_rds: 220_usd
      redis: 165_usd
      llm_apis: 6500_usd  # Claude Sonnet 4 (higher quality)
      stt_deepgram: 600_usd
      tts_elevenlabs: 300_usd
      storage_s3: 65_usd
    
  scaled_to_6_nodes:
    monthly_cost: 16800_usd  # 2x infrastructure
    concurrent_conversations: 1200
    cost_per_conversation: 14.00_usd  # Linear scaling
    
  scaled_to_10_nodes:
    monthly_cost: 28000_usd
    concurrent_conversations: 2000
    cost_per_conversation: 14.00_usd
    
  optimization_notes:
    - "Prompt caching reduces LLM costs by ~30%"
    - "Model cascading (GPT-4o-mini first) saves ~40% on simple queries"
    - "Token compression reduces input costs by ~25%"
    - "With optimizations: $9.50/conversation achievable"
```

### 18.10 Load Testing & Benchmarking

**Load Test Scenarios**:
```yaml
load_tests:
  baseline:
    duration: 1_hour
    concurrent_users: 500
    ramp_up: 5_minutes
    success_criteria:
      - response_time_p95 < 2000ms
      - error_rate < 0.1%
      - no_pod_restarts
      
  stress_test:
    duration: 30_minutes
    concurrent_users: 2000  # 4x normal
    ramp_up: 10_minutes
    success_criteria:
      - system_remains_stable
      - graceful_degradation (not crash)
      - recovery_time < 5_minutes
      
  soak_test:
    duration: 24_hours
    concurrent_users: 600  # Normal load
    success_criteria:
      - no_memory_leaks
      - no_connection_pool_exhaustion
      - consistent_latency_over_time
      
  spike_test:
    duration: 10_minutes
    pattern: "0 → 2000 users in 30 seconds"
    success_criteria:
      - auto_scaling_triggers
      - no_dropped_requests
      - recovery_time < 5_minutes
```

**Benchmarking Tools**:
```yaml
tools:
  api_load_testing:
    tool: k6
    script: load-tests/api-benchmark.js
    command: "k6 run --vus 500 --duration 1h"
    
  voice_pipeline:
    tool: custom (LiveKit load generator)
    concurrent_calls: 250
    call_duration: 5_minutes_average
    
  database:
    tool: pgbench
    clients: 100
    transactions: 10000
    mode: "TPC-B-like"
```

### 18.11 Performance Monitoring Dashboards

**Key Metrics** (Grafana):
```yaml
dashboards:
  platform_overview:
    - requests_per_second
    - response_time_p50_p95_p99
    - error_rate
    - active_users
    - cost_per_hour
    
  agent_performance:
    - agent_calls_per_second
    - agent_response_time_p95
    - mcp_overhead_ms
    - llm_token_usage
    - cost_per_agent_call
    
  voice_pipeline:
    - concurrent_calls
    - stt_latency_p95
    - llm_first_token_p95
    - tts_latency_p95
    - end_to_end_latency_p95
    
  infrastructure:
    - cpu_utilization_per_node
    - memory_utilization_per_node
    - disk_iops
    - network_throughput
    - database_connection_pool
```

**Alerting Rules** (Prometheus):
```yaml
alerts:
  critical:
    - name: HighErrorRate
      condition: error_rate > 1% for 5 minutes
      action: Page on-call engineer
      
    - name: ResponseTimeSLAViolation
      condition: p95_latency > 2500ms for 5 minutes
      action: Page on-call + auto-scale
      
    - name: DatabaseConnectionsExhausted
      condition: active_connections > 90% of max
      action: Page DBA team
      
  warning:
    - name: HighCPUUtilization
      condition: cpu > 80% for 10 minutes
      action: Slack alert + prepare to scale
      
    - name: CostAnomalyDetected
      condition: hourly_cost > 2x_average
      action: Slack alert to FinOps team
```

### 18.12 Performance Optimization Checklist

**Application-Level**:
```yaml
optimizations:
  - [ ] Database query indexing (all foreign keys, common filters)
  - [ ] Connection pooling (pgbouncer, Redis connection pool)
  - [ ] API response pagination (max 100 items per page)
  - [ ] Lazy loading (don't fetch full objects if not needed)
  - [ ] Async processing (Celery for heavy tasks)
  - [ ] Prompt caching (LLM providers, 10x cost reduction)
  - [ ] Model cascading (cheap model first, fallback to expensive)
```

**Infrastructure-Level**:
```yaml
optimizations:
  - [ ] CDN for static assets
  - [ ] Read replicas for databases (2+ per primary)
  - [ ] Redis Sentinel for HA
  - [ ] Kubernetes HPA configured
  - [ ] Network policies (reduce cross-AZ traffic)
  - [ ] Premium storage tier (minimum 2500 IOPS)
  - [ ] Dedicated node pools (separate compute/memory workloads)
```

### 18.13 Customer-Specific SLA Examples

**Case Study: Standard SaaS Customer**
```yaml
customer: generic_saas_company
tier: premium
sla:
  response_time_p95: 1500ms
  uptime: 99.95%
  concurrent_users: 500
  support_response: 4_hours
  
actual_performance:
  response_time_p95: 1350ms  # ✅ Beating SLA
  uptime_last_month: 99.98%  # ✅ Beating SLA
  incidents: 1 (5 min outage)
  sla_credits_earned: 0_usd
```

**Case Study: Demanding Enterprise Customer**
```yaml
customer_profile:
  industry: Global Financial Services & Information
  structure: Multinational public corporation
  annual_revenue: $15_billion_usd
  employees: 50000+
  regulatory_requirements: [SEC, FINRA, GDPR, multiple_jurisdictions]

tier: enterprise
sla:
  response_time_p95: 1000ms
  uptime: 99.995%  # 26 seconds/month
  concurrent_users: 2000
  support_response: 15_minutes (24/7)
  dedicated_tam: true
  
actual_performance:
  response_time_p95: 950ms  # ✅ Beating SLA
  uptime_last_month: 99.997%  # ✅ Beating SLA
  incidents: 0
  sla_credits_earned: 0_usd
  
infrastructure:
  dedicated_cluster: true
  nodes: 10
  monthly_cost: 28000_usd  # 2025 pricing (A2A architecture)
```

---

**References**:
- Enterprise Conversational AI Production Architectures (2022-2024)
- SAAA Project Performance Benchmarks (voice.latency.example.md)
- AWS Well-Architected Framework (Performance Efficiency Pillar)
- Google SRE Book (Chapter 4: Service Level Objectives)
- Microsoft Azure Architecture Center (Performance Efficiency)

## 19. Vendor & Subprocessor Management ✅ DETAILED

**Scope**: Governance for third-party providers (LLM APIs, cloud infrastructure, tooling) with compliance alignment.

**Framework**: ISO 27001 (A.15 Supplier Relationships) + GDPR Article 28 (Processor Requirements)

### 19.1 Vendor Classification

**Vendor Types**:
```yaml
vendor_categories:
  infrastructure_providers:
    examples: [AWS, GCP, Azure]
    data_access: Full (hosts customer data)
    risk_tier: Critical
    assessment_frequency: Annual
    
  ai_model_providers:
    examples: [Anthropic, OpenAI, Deepgram, ElevenLabs]
    data_access: Transient (API calls only)
    risk_tier: High
    assessment_frequency: Annual
    contractual_requirement: "No training on customer data"
    
  observability_providers:
    examples: [Datadog, New Relic, Grafana Cloud]
    data_access: Logs/metrics (may contain PII)
    risk_tier: High
    assessment_frequency: Annual
    contractual_requirement: Data Processing Agreement (DPA)
    
  identity_providers:
    examples: [Zitadel, Okta, Entra ID]
    data_access: User credentials, profile data
    risk_tier: Critical
    assessment_frequency: Annual
    
  development_tools:
    examples: [GitHub, GitLab, Terraform Cloud]
    data_access: Source code, infrastructure configs
    risk_tier: Medium
    assessment_frequency: Biannual
    
  security_tools:
    examples: [Snyk, Trivy, SIEM providers]
    data_access: Vulnerability data, security logs
    risk_tier: Medium
    assessment_frequency: Biannual
    
  non_data_access:
    examples: [Slack, productivity tools]
    data_access: None
    risk_tier: Low
    assessment_frequency: Biannual
```

### 19.2 Vendor Risk Assessment Process

**Assessment Stages**:

**Stage 1: Initial Due Diligence** (Before Contract)
```yaml
due_diligence_checklist:
  security_posture:
    - [ ] SOC 2 Type II report (within 12 months)
    - [ ] ISO 27001 certification (current)
    - [ ] Penetration test results (within 6 months)
    - [ ] Security questionnaire completed
    - [ ] Data breach history reviewed
    - [ ] Bug bounty program exists
  
  compliance:
    - [ ] GDPR compliance documented
    - [ ] HIPAA BAA available (if applicable)
    - [ ] Data residency options available
    - [ ] Sub-processor list provided
    - [ ] DPA reviewed and acceptable
  
  operational:
    - [ ] SLA commitments (uptime, support)
    - [ ] Disaster recovery plan documented
    - [ ] Business continuity plan exists
    - [ ] Financial stability verified (Dun & Bradstreet)
    - [ ] Insurance coverage adequate (cyber, E&O)
  
  contractual:
    - [ ] Data ownership clear (customer owns data)
    - [ ] No training on customer data clause
    - [ ] Right to audit included
    - [ ] Termination and data return procedures
    - [ ] Exit assistance provisions
```

**Stage 2: Approval Workflow**
```yaml
approval_authority:
  critical_risk_vendors:
    approvers: [CISO, CIO, Legal, CFO]
    timeline: 30_business_days
    
  high_risk_vendors:
    approvers: [CISO, CIO, Legal]
    timeline: 15_business_days
    
  medium_risk_vendors:
    approvers: [CISO, CIO]
    timeline: 10_business_days
    
  low_risk_vendors:
    approvers: [CISO]
    timeline: 5_business_days
```

**Stage 3: Contract Execution**
```yaml
required_contract_clauses:
  data_processing:
    - Data Processing Agreement (DPA) - GDPR Article 28
    - Business Associate Agreement (BAA) - HIPAA (if applicable)
    - Data ownership and license terms
    - Sub-processor notification and approval rights
    - Data residency and transfer mechanisms
  
  security:
    - Security incident notification (24-48 hours)
    - Annual security assessments required
    - Right to audit (on-site or remote)
    - Encryption requirements (TLS 1.3, AES-256)
    - Vulnerability disclosure timeline
  
  operational:
    - SLA commitments (uptime %, response times)
    - Support escalation procedures
    - Change notification requirements (30+ days)
    - Disaster recovery and business continuity
  
  termination:
    - Data return or destruction procedures (30 days)
    - Transition assistance (60-90 days)
    - No lock-in penalties
    - Post-termination confidentiality
```

**Stage 4: Ongoing Monitoring**
```yaml
continuous_monitoring:
  quarterly_reviews:
    - SLA performance (uptime, latency, support response)
    - Security incident history
    - Cost vs budget
    - Compliance status changes
  
  annual_assessments:
    - Request updated SOC 2 / ISO 27001
    - Review penetration test results
    - Re-validate security questionnaire
    - Assess financial stability
    - Review sub-processor changes
  
  triggered_reviews:
    - Security breach or incident
    - Service degradation (SLA violations)
    - Ownership change or acquisition
    - Major price increase (>20%)
    - Contract renewal approaching
```

### 19.3 Subprocessor Registry (GDPR Article 28)

**Purpose**: Transparency for customers about who processes their data.

**Subprocessor List** (Example):
```yaml
subprocessors:
  infrastructure:
    - name: Amazon Web Services (AWS)
      purpose: Cloud infrastructure hosting
      data_categories: [All customer data]
      location: [US, EU, APAC per customer region]
      dpa_signed: true
      certification: [SOC 2, ISO 27001, HIPAA BAA]
      added_date: 2024-01-01
      
    - name: Google Cloud Platform (GCP)
      purpose: Backup and disaster recovery
      data_categories: [Database backups, snapshots]
      location: [US, EU per customer region]
      dpa_signed: true
      certification: [SOC 2, ISO 27001]
      added_date: 2024-01-01
  
  ai_services:
    - name: Anthropic
      purpose: LLM inference (Claude models)
      data_categories: [User prompts, conversation context]
      location: [US]
      dpa_signed: true
      no_training_clause: true
      certification: [SOC 2]
      added_date: 2024-06-01
      
    - name: OpenAI
      purpose: LLM inference (GPT models, fallback)
      data_categories: [User prompts, conversation context]
      location: [US]
      dpa_signed: true
      no_training_clause: true
      certification: [SOC 2]
      added_date: 2024-01-01
      
    - name: Deepgram
      purpose: Speech-to-text
      data_categories: [Voice audio, transcripts]
      location: [US]
      dpa_signed: true
      no_training_clause: true
      certification: [SOC 2, GDPR]
      added_date: 2024-03-01
      
    - name: ElevenLabs
      purpose: Text-to-speech
      data_categories: [Text prompts, voice audio]
      location: [US, EU]
      dpa_signed: true
      certification: [SOC 2, GDPR]
      added_date: 2024-03-01
  
  observability:
    - name: Grafana Labs
      purpose: Monitoring and alerting
      data_categories: [Logs, metrics, traces - may contain PII]
      location: [US]
      dpa_signed: true
      certification: [SOC 2, ISO 27001]
      pii_redaction: true
      added_date: 2024-01-01
  
  identity:
    - name: Zitadel
      purpose: Identity and access management
      data_categories: [User credentials, profile data]
      location: [EU, US per deployment]
      dpa_signed: true
      certification: [SOC 2, ISO 27001, GDPR]
      added_date: 2024-01-01
```

**Subprocessor Change Notification Process**:
```yaml
change_notification:
  advance_notice: 30_days
  notification_method:
    - Email to customer admin
    - Trust Center update
    - API webhook (for enterprise customers)
  
  customer_rights:
    - Object to new subprocessor (within 30 days)
    - Request alternative solution
    - Terminate contract without penalty (if no alternative)
  
  documentation:
    - Updated subprocessor list (public)
    - DPA amendments (sent to customers)
    - Risk assessment of new subprocessor (internal)
```

### 19.4 Data Processing Agreements (DPAs)

**DPA Requirements per Regulation**:

**GDPR (Article 28)**:
```yaml
gdpr_dpa_clauses:
  mandatory:
    - Subject matter and duration of processing
    - Nature and purpose of processing
    - Type of personal data
    - Categories of data subjects
    - Controller and processor obligations
    - Sub-processor notification and approval
    - Security measures (technical and organizational)
    - Data breach notification (72 hours)
    - International data transfers (SCCs, adequacy)
    - Return or deletion of data post-termination
    - Audit rights
    - Processor liability limitations
```

**HIPAA BAA**:
```yaml
hipaa_baa_clauses:
  mandatory:
    - Permitted uses and disclosures of PHI
    - Safeguards to prevent impermissible uses
    - Subcontractor agreements required
    - Breach notification (within 60 days)
    - Access, amendment, accounting rights
    - Minimum necessary access
    - Return or destruction of PHI post-termination
    - Audit, inspection, and compliance review rights
```

**CCPA Service Provider Agreement**:
```yaml
ccpa_agreement_clauses:
  mandatory:
    - Prohibition on selling personal information
    - Prohibition on retaining, using, or disclosing PI
    - Right to audit compliance
    - Certification of understanding obligations
    - Deletion of consumer data upon request
```

**Standard DPA Template Location**: `legal/templates/dpa_template_v2024.pdf`

### 19.5 Vendor SLA Tracking

**SLA Monitoring Dashboard**:
```yaml
sla_metrics:
  infrastructure_vendors:
    aws:
      uptime_sla: 99.99%
      actual_uptime_last_30_days: 99.98%
      sla_credits_earned: 0_usd
      incidents_last_quarter: 2
      
  ai_model_providers:
    anthropic:
      uptime_sla: 99.9%
      actual_uptime: 99.95%
      p95_latency_sla: 2000ms
      actual_p95_latency: 1850ms
      rate_limit_breaches: 0
      
  observability:
    grafana:
      uptime_sla: 99.9%
      log_ingestion_lag_sla: 5_seconds
      actual_lag: 3_seconds
```

**SLA Violation Process**:
```yaml
sla_violation_response:
  detection:
    - Automated monitoring (Prometheus, Datadog)
    - Customer reports
    - Vendor status pages
  
  escalation:
    - P0: CTO notified immediately
    - P1: Infrastructure lead investigates
    - P2: Monitor and document
  
  remediation:
    - Request SLA credits from vendor
    - Escalate to account manager
    - If chronic: Initiate vendor replacement evaluation
  
  documentation:
    - Incident post-mortem (internal)
    - Vendor performance review (quarterly)
    - Contract renewal considerations (annual)
```

### 19.6 Third-Party Security Assessment

**Security Questionnaire** (Enterprise SaaS Best Practices):

```yaml
security_questionnaire_sections:
  1_security_policy_organization:
    - Formal information security policy exists?
    - Policy reviewed annually?
    - Policy communicated to all staff?
    - Acceptable use policy (internet, email, mobile)?
  
  2_asset_management:
    - Key assets inventoried and ownership assigned?
    - Data classification categories defined?
    - Customer information handling procedures?
  
  3_human_resources_security:
    - Background checks for employees/contractors?
    - Non-disclosure agreements required?
    - Access removal process post-termination?
  
  4_physical_environmental_security:
    - Sensitive areas separated from general access?
    - Perimeter access controls (card access, CCTV)?
    - Visitor log and escort policy?
  
  5_communications_operations_management:
    - IT operations procedures documented?
    - Patch management policy?
    - Anti-virus/anti-malware policy?
    - Data loss prevention controls deployed?
    - Multi-factor authentication for remote access?
    - Split tunneling prohibited?
    - Data in transit encrypted (TLS version)?
    - Data at rest encrypted (algorithm)?
    - Full-disk encryption on mobile devices?
    - IP whitelisting utilized?
    - Firewall logging enabled and reviewed?
    - System logs protected from unauthorized access?
  
  6_access_control:
    - Access management policy exists?
    - Need-to-know and least privilege principles?
    - Password policy (complexity requirements)?
    - Passwords never stored in clear text?
  
  7_system_acquisition_development_maintenance:
    - SDLC followed for application development?
    - Source code reviewed for security flaws?
    - Encryption keys and certificates protected?
    - Change management policy exists?
    - Vulnerability testing process documented?
  
  8_information_security_incident_management:
    - Security incident response policy exists?
    - Intrusion detection/prevention systems deployed?
    - Customer/regulator notification procedures?
  
  9_compliance_privacy:
    - SOC 2 / ISO 27001 certifications current?
    - Penetration tests performed (frequency)?
    - Data Protection Officer designated?
    - Customer information handling procedure?
    - Records retention policy?
    - Data storage locations (country/city)?
    - PII handling procedures documented?
    - GDPR/CCPA compliance mechanisms?
    - Data residency enforcement (EU data in EU)?
    - Right to be Forgotten implementation?
    - Data Subject Access Request process?
    - Data protection training (annual)?
    - Privacy policy reviewed annually?
    - Dedicated infrastructure for data segregation?
    - Privacy risk assessment (frequency)?
    - Third-party processors meet GDPR/CCPA requirements?
  
  10_business_continuity_disaster_recovery:
    - BC/DR policies exist and reviewed annually?
    - RPO/RTO targets defined and tested?
```

**Response Process**:
```bash
# Vendor completes questionnaire
./scripts/vendor-questionnaire.sh --vendor anthropic --send

# Vendor returns completed questionnaire (PDF or structured data)
./scripts/ingest-questionnaire.sh --vendor anthropic --file response.pdf

# Security team reviews
./scripts/assess-vendor-risk.sh --vendor anthropic --generate-report

# Approval workflow
./scripts/vendor-approval.sh --vendor anthropic --approvers ciso,cio,legal
```

### 19.7 Exit Strategies & Vendor Lock-In Mitigation

**Lock-In Risk Assessment**:
```yaml
lock_in_risks:
  infrastructure:
    aws:
      risk_level: Medium
      mitigation:
        - Terraform IaC (cloud-agnostic)
        - Abstract AWS services (S3 → ObjectStorage interface)
        - Backup to GCP (cross-cloud DR)
        - Annual cost comparison with GCP/Azure
      
  ai_models:
    anthropic:
      risk_level: Low
      mitigation:
        - Multi-model architecture (OpenAI fallback)
        - Standardized LLM interface (LangChain)
        - Cost-based routing (switch providers easily)
        - No vendor-specific prompt engineering
      
  identity:
    zitadel:
      risk_level: Medium
      mitigation:
        - OIDC/SAML standard protocols
        - User directory exportable (SCIM)
        - Self-hosted option available
        - Alternative vendors evaluated (Okta, Entra)
```

**Exit Procedures** (Contract Termination):
```yaml
exit_process:
  notification:
    timeline: 90_days_before_contract_end
    recipients: [vendor_account_manager, legal, procurement]
  
  data_return:
    timeline: 30_days_after_termination
    format: [encrypted_backup, json_export, database_dump]
    verification: Customer validates data completeness
  
  data_destruction:
    timeline: 30_days_after_data_return
    method: Secure deletion per NIST 800-88
    certification: Vendor provides destruction certificate
  
  access_revocation:
    timeline: Immediate upon termination
    scope: [API keys, admin accounts, integrations]
    verification: Audit logs reviewed
  
  transition_assistance:
    timeline: 60_days (optional, if contracted)
    support: [data migration, configuration export, runbook transfer]
```

### 19.8 Subprocessor Approval Process

**Customer Opt-In/Opt-Out**:
```yaml
subprocessor_change:
  notification:
    method: [email, trust_center_update, api_webhook]
    advance_notice: 30_days
    content:
      - Subprocessor name and location
      - Purpose and data categories
      - Security certifications
      - DPA status
  
  customer_response:
    deadline: 30_days
    options:
      - accept: "No action required"
      - object: "Request alternative solution"
      - terminate: "Exit contract without penalty"
  
  platform_obligation:
    if_objection: "Provide alternative or allow termination"
    if_no_alternative: "Customer may terminate contract"
    fallback: "Delay new subprocessor until resolution"
```

### 19.9 Vendor Performance Scorecard

**Quarterly Review**:
```yaml
vendor_scorecard:
  anthropic:
    quarter: Q4_2024
    
    security: 95/100
      - No security incidents: +25
      - SOC 2 current: +25
      - Pen test results acceptable: +20
      - Security questionnaire complete: +15
      - Annual assessment on time: +10
    
    operational: 92/100
      - SLA met (99.95% uptime): +30
      - P95 latency within SLA: +25
      - Support response time <1 hour: +20
      - No unplanned outages: +17
    
    compliance: 100/100
      - DPA current: +25
      - No training on customer data verified: +25
      - GDPR compliance documented: +25
      - Data residency enforced: +25
    
    cost: 85/100
      - Within budget: +30
      - No surprise price increases: +20
      - Cost-per-request competitive: +15
      - Invoice accuracy: +20
    
    overall_score: 93/100
    status: Approved for renewal
    action_items:
      - Request cost optimization discussion
      - Renew DPA for 2025
```

### 19.10 Integration with Trust Center

**Public Subprocessor List**:
- Published at: `https://trust.platform.com/subprocessors`
- Updated within 24 hours of changes
- Historical changes archived (audit trail)
- RSS feed for customer notifications
- API endpoint for programmatic access

**Security Documentation Portal**:
```yaml
trust_center_vendor_section:
  - SOC 2 Type II report (NDA-gated)
  - ISO 27001 certificate (public)
  - Subprocessor list (public)
  - DPA template (public)
  - Security questionnaire (customer-submitted)
  - Penetration test summary (NDA-gated)
  - Compliance mappings (GDPR, HIPAA, CCPA)
```

### 19.11 Vendor Risk Register

**Central Repository**:
```sql
CREATE TABLE vendor_risk_register (
    vendor_id UUID PRIMARY KEY,
    vendor_name TEXT NOT NULL,
    vendor_category TEXT NOT NULL,
    risk_tier TEXT NOT NULL,  -- critical, high, medium, low
    
    -- Assessment
    last_assessment_date DATE,
    next_assessment_due DATE,
    assessment_status TEXT,  -- current, overdue, pending
    
    -- Certifications
    soc2_expiry DATE,
    iso27001_expiry DATE,
    hipaa_baa_signed BOOLEAN,
    gdpr_dpa_signed BOOLEAN,
    
    -- Performance
    sla_compliance_pct DECIMAL(5, 2),
    incident_count_last_12_months INTEGER,
    cost_vs_budget_pct DECIMAL(5, 2),
    
    -- Compliance
    subprocessor_list_current BOOLEAN,
    data_residency_verified BOOLEAN,
    right_to_audit_exercised_date DATE,
    
    -- Ownership
    business_owner UUID,
    technical_owner UUID,
    contract_expiry DATE,
    auto_renewal BOOLEAN,
    
    -- Status
    status TEXT,  -- active, under_review, exiting, terminated
    notes TEXT,
    
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 19.12 Compliance Automation

**Automated Vendor Monitoring**:
```python
class VendorComplianceMonitor:
    async def check_vendor_compliance(self, vendor_id: str) -> ComplianceReport:
        """Daily automated compliance checks"""
        
        checks = []
        
        # 1. Check SOC 2 expiry
        soc2_expiry = await self.get_soc2_expiry(vendor_id)
        if soc2_expiry < datetime.now() + timedelta(days=90):
            checks.append(ComplianceAlert(
                severity="high",
                message=f"SOC 2 expires in {(soc2_expiry - datetime.now()).days} days",
                action="Request updated report"
            ))
        
        # 2. Check SLA performance
        sla_compliance = await self.calculate_sla_compliance(vendor_id, days=30)
        if sla_compliance < 99.0:
            checks.append(ComplianceAlert(
                severity="medium",
                message=f"SLA compliance at {sla_compliance}% (below 99%)",
                action="Escalate to vendor account manager"
            ))
        
        # 3. Check security incidents
        incidents = await self.get_recent_incidents(vendor_id, days=90)
        if len(incidents) > 0:
            checks.append(ComplianceAlert(
                severity="high",
                message=f"{len(incidents)} security incidents in last 90 days",
                action="Review incident reports and impact"
            ))
        
        return ComplianceReport(vendor_id=vendor_id, checks=checks)
```

---

**References**:
- ISO 27001:2013 Annex A.15 (Supplier Relationships)
- GDPR Article 28 (Processor Requirements)
- NIST SP 800-161 (Supply Chain Risk Management)
- HIPAA 45 CFR § 164.314(a) (Business Associate Contracts)
- CCPA § 1798.140(v) (Service Provider Definition)

## 20. Change Management & Release Governance (ITIL v4 Aligned)

**Scope**: Align platform change and release practices with **ITIL v4 Change Enablement** and **Release Management**.

**Framework Alignment**:
- **ITIL v4 Change Enablement**: Maximize successful service and product changes
- **ITIL v4 Release Management**: Make new/changed services and features available for use
- **DevOps/AgentOps**: Continuous delivery with governance guardrails
- **ISO/IEC 20000**: IT service management standard

**ITIL v4 Change Types**:
```yaml
change_types:
  standard_change:
    description: "Pre-approved, low-risk, well-understood"
    examples:
      - Agent configuration updates (model selection, prompts)
      - Non-breaking API changes
      - Routine security patches
    approval: Automated (peer review only)
    risk: Low
  
  normal_change:
    description: "Requires assessment and authorization"
    examples:
      - New agent deployment
      - Database schema changes
      - Network topology changes
      - Fine-tuned model deployments
    approval: Change Advisory Board (CAB) or Emergency CAB
    risk: Medium to High
    process:
      - RFC (Request for Change) submitted
      - Risk assessment (CAIO for AI changes, CISO for security)
      - CAB review (async or meeting)
      - Scheduled change window
      - Post-implementation review (PIR)
  
  emergency_change:
    description: "Critical, requires rapid deployment"
    examples:
      - Security vulnerability fix (CVE)
      - Production outage resolution
      - Critical data breach remediation
    approval: Emergency CAB (ECAB) - CTO, CISO, on-call lead
    risk: High (but necessary)
    process:
      - Emergency RFC with incident ticket
      - ECAB approval (can be verbal, documented after)
      - Immediate deployment
      - Retrospective PIR within 48 hours
```

**Change Advisory Board (CAB) Composition**:
```yaml
cab_membership:
  core_members:
    - CTO (chair)
    - Engineering Lead
    - Infrastructure Lead
    - Security Lead
  
  consulted_per_change_type:
    ai_changes: CAIO
    cost_impact: CFO representative
    compliance_risk: Compliance Officer
    customer_facing: CRO/Product
  
  meeting_cadence:
    normal_cab: Weekly (Wednesdays 2pm)
    emergency_cab: On-demand (30-min SLA)
  
  decision_criteria:
    - Business value vs risk
    - Rollback plan exists?
    - Cost impact analysis complete?
    - Security review passed?
    - Compliance tags reviewed?
```

**ITIL v4 Release Management Integration**:
```yaml
release_pipeline:
  dev_environment:
    trigger: Git push to feature branch
    approval: None (automated tests)
    deployment: Continuous
  
  test_environment:
    trigger: Merge to main branch
    approval: Automated (test suite passes)
    deployment: Continuous
  
  staging_environment:
    trigger: Tag created (v1.2.3-rc1)
    approval: Engineering Lead + QA sign-off
    deployment: Automated after approval
    testing: Full E2E suite, load testing
  
  production_environment:
    trigger: Release candidate approved by CAB
    approval: CAB (normal change) or ECAB (emergency)
    deployment: Blue/green or canary
    rollback: Automated if health checks fail
    verification: Smoke tests, synthetic monitoring
```

**DevOps/AgentOps Adaptations**:
- **Standard Changes**: 80%+ of changes should become standard (pre-approved)
- **Immutable Infrastructure**: Container images (same SHA across environments)
- **GitOps**: All changes tracked in Git (Infrastructure as Code)
- **Observability**: Cost impact, latency impact measured pre/post deployment
- **Agent Versioning**: Semantic versioning with rollback capability

**Platform-Specific Controls**:
```python
class ChangeRecord:
    """ITIL v4 Change Record adapted for A2A platform"""
    change_id: str
    change_type: ChangeType  # standard, normal, emergency
    risk_tier: RiskTier  # minimal, limited, high
    
    # ITIL v4 standard fields
    requester: str
    description: str
    business_justification: str
    impact_assessment: str
    rollback_plan: str
    
    # Platform-specific
    affected_agents: List[str]
    affected_environments: List[EnvironmentType]
    cost_impact_usd: Optional[Decimal]
    compliance_tags: List[str]  # HIPAA, EU_AI_ACT, etc.
    
    # Approvals
    cab_approval: Optional[ApprovalRecord]
    caio_approval: Optional[ApprovalRecord]  # If AI change
    ciso_approval: Optional[ApprovalRecord]  # If security change
    
    # Outcomes
    deployment_status: DeploymentStatus
    pir_completed: bool
    pir_summary: Optional[str]
```

**Tooling Integration**:
- **ITSM Platform**: ServiceNow, Jira Service Management, or equivalent
- **CI/CD**: GitHub Actions, GitLab CI (automated standard changes)
- **GitOps**: ArgoCD, Flux (declarative deployments)
- **Observability**: Pre/post deployment comparison dashboards

**Reference**: 
- ITIL v4 Foundation (Change Enablement, Release Management)
- ITIL v4 Specialist: Create, Deliver and Support
- ITIL v4 Strategist: Direct, Plan and Improve

**Detailed content**: v1.4 will provide CAB charter, RFC templates, and PIR frameworks aligned to ITIL v4

## 21. Developer Experience & Local Development ✅ DETAILED

**Scope**: Maximize developer productivity with local-first workflows that mirror production.

### 21.1 One-Command Setup

```bash
# Clone repository
git clone https://github.com/yourorg/platform
cd platform

# One-command startup
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f

# Everything ready at:
# - Frontend: https://app.local.test:3000
# - Backend API: https://api.local.test:8000
# - Admin UI: https://admin.local.test:3001
```

**Goal**: Developer is productive within 5 minutes of cloning.

### 21.2 LocalStack for AWS Parity

**Services Emulated**:
```yaml
localstack_services:
  - secretsmanager      # API keys, credentials
  - ssm                 # Configuration parameters
  - s3                  # Object storage
  - dynamodb           # Optional: NoSQL data
  - sqs                # Optional: Queues
  - sns                # Optional: Notifications
  - kms                # Encryption key management
```

**Seed Script** (`scripts/init-localstack.sh`):
```bash
#!/bin/bash
# Wait for LocalStack
awslocal secretsmanager create-secret \
  --name saaa/dev/deepgram_api_key \
  --secret-string "dev_key_..."

awslocal ssm put-parameter \
  --name /saaa/dev/llm_provider \
  --value "anthropic" \
  --type String

awslocal s3 mb s3://saaa-dev-storage
```

**Developer Benefit**: Same boto3 code works in dev and prod, just different endpoint URL.

### 21.3 Local DNS & TLS

**DNS Setup** (macOS/Linux):
```bash
# Install dnsmasq
brew install dnsmasq  # macOS
# or: apt-get install dnsmasq  # Linux

# Configure *.local.test → 127.0.0.1
echo "address=/local.test/127.0.0.1" >> /usr/local/etc/dnsmasq.conf

# Start dnsmasq
sudo brew services start dnsmasq

# Configure system resolver
sudo mkdir -p /etc/resolver
echo "nameserver 127.0.0.1" | sudo tee /etc/resolver/local.test
```

**TLS Setup** (mkcert):
```bash
# Install mkcert
brew install mkcert
mkcert -install  # Trust local CA

# Generate wildcard cert
cd config/certs
mkcert "*.local.test" localhost 127.0.0.1
```

**Result**: All services use `https://` with valid certs, no browser warnings.

### 21.4 Hot-Reload & Fast Iteration

**Backend (FastAPI)**:
```python
# main.py
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True,  # Auto-reload on code changes
        reload_dirs=["src/saaa"],
        log_level="info"
    )
```

**Frontend (Next.js)**:
```json
// package.json
{
  "scripts": {
    "dev": "next dev --turbo",  // Fast Refresh enabled
    "build": "next build",
    "start": "next start"
  }
}
```

**Docker Compose Volume Mounts**:
```yaml
services:
  backend:
    volumes:
      - ./src:/app/src:cached  # Code changes reflected immediately
      - backend_venv:/app/.venv  # Cache dependencies
  
  frontend:
    volumes:
      - ./frontend:/app:cached
      - /app/node_modules  # Don't overwrite node_modules
```

### 21.5 Debugging Setup

**VS Code** (`launch.json`):
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: FastAPI",
      "type": "python",
      "request": "attach",
      "connect": {
        "host": "localhost",
        "port": 5678
      },
      "pathMappings": [
        {
          "localRoot": "${workspaceFolder}/src",
          "remoteRoot": "/app/src"
        }
      ]
    },
    {
      "name": "Next.js: Chrome",
      "type": "chrome",
      "request": "launch",
      "url": "https://app.local.test:3000"
    }
  ]
}
```

**PyCharm**: Attach to remote Python interpreter in Docker container.

### 21.6 Testing Utilities

**Pytest Fixtures** (`tests/conftest.py`):
```python
import pytest
from saaa.infrastructure.config.settings import Settings

@pytest.fixture
def test_settings():
    """Override settings for tests"""
    return Settings(
        environment="test",
        database_url="postgresql://test:test@localhost:5433/test",
        redis_url="redis://localhost:6380",
        aws_endpoint_url="http://localhost:4566",  # LocalStack
    )

@pytest.fixture
async def test_client(test_settings):
    """FastAPI test client"""
    from fastapi.testclient import TestClient
    from main import app
    return TestClient(app)

@pytest.fixture
async def test_db():
    """In-memory database for tests"""
    async with AsyncSession() as session:
        yield session
        await session.rollback()

@pytest.fixture
def mock_llm_response():
    """Mock LLM response (avoid API calls in tests)"""
    return {
        "content": "Test response",
        "model": "gpt-4o-mini",
        "usage": {"input_tokens": 10, "output_tokens": 5}
    }
```

**Factory Pattern** (`tests/factories.py`):
```python
from factory import Factory, Faker, SubFactory
from saaa.core.entities.agent import AgentInfo

class AgentInfoFactory(Factory):
    class Meta:
        model = AgentInfo
    
    agent_id = Faker("slug")
    name = Faker("company")
    version = "1.0.0"
    description = Faker("sentence")
    category = "capability"
    endpoint = Faker("url")

# Usage:
agent = AgentInfoFactory.create()
```

### 21.7 Cost Simulation in Dev

**Goal**: Estimate production costs during development.

```python
class MockCostTracker:
    """Simulates cost tracking in dev without real API calls"""
    
    MOCK_PRICING = {
        "anthropic/claude-sonnet-4": {"input": 3.00, "output": 15.00},
        "openai/gpt-4o": {"input": 2.50, "output": 10.00},
        "deepgram/nova-2": {"per_hour": 15.00},
        "elevenlabs/turbo-v2.5": {"per_hour": 22.00},
    }
    
    async def track_llm_call(self, model: str, tokens_in: int, tokens_out: int):
        cost = (
            (tokens_in / 1_000_000) * self.MOCK_PRICING[model]["input"] +
            (tokens_out / 1_000_000) * self.MOCK_PRICING[model]["output"]
        )
        logger.info(f"[MOCK COST] {model}: ${cost:.4f} ({tokens_in + tokens_out} tokens)")
        # Emit to local metrics (not real cost tracking)
        await self.local_metrics.record(cost)
```

**Dashboard**: Show estimated costs in local dev UI.

### 21.8 Developer Productivity Metrics

**Target Metrics**:
```yaml
developer_experience_targets:
  time_to_first_commit: <60 minutes  # From laptop to first PR
  time_to_productive: <4 hours  # Understanding codebase
  local_test_suite_time: <5 minutes
  docker_compose_startup: <2 minutes
  hot_reload_latency: <500ms  # Code change to browser refresh
  
  cicd_feedback_loop:
    lint_check: <2 minutes
    unit_tests: <5 minutes
    integration_tests: <10 minutes
    e2e_tests: <15 minutes
```

### 21.9 Documentation & Onboarding

**Required Docs** (in repo):
```
docs/
  ├── README.md                    # Quick start
  ├── CONTRIBUTING.md              # PR guidelines
  ├── ARCHITECTURE.md              # High-level design
  ├── LOCAL_DEVELOPMENT.md         # This section (detailed)
  ├── TESTING.md                   # Test strategy
  ├── DEBUGGING.md                 # Common issues
  └── API_REFERENCE.md             # OpenAPI docs
```

**Onboarding Checklist**:
- [ ] Clone repository
- [ ] Install prerequisites (Docker, mkcert, awslocal CLI)
- [ ] Run setup scripts (`./scripts/setup-dev.sh`)
- [ ] Start services (`docker-compose up`)
- [ ] Verify health checks pass
- [ ] Run test suite (`pytest`)
- [ ] Make a trivial code change and see hot-reload work
- [ ] Submit first PR (update CONTRIBUTORS.md)

### 21.10 Integration Points

**Environment Isolation**:
- Dev uses LocalStack (no real AWS costs)
- Dev environment secrets isolated from prod
- Dev database can be reset/seeded without affecting prod

**Cost Tracking**:
- Simulate LLM costs in dev (no real API calls)
- Log "would-be" costs to help developers optimize

**SPIRE** (Optional in Dev):
- Can run SPIRE locally for mTLS testing
- Or disable mTLS in dev for faster iteration

**Trade-off**: Dev/prod parity vs. developer speed. Optimize for speed in dev, enforce parity in staging.

## 22. Endpoint Security & Device Management

**Scope**: Secure user devices accessing the platform.

**Key Components**:
- MDM/UEM integration (Intune, Jamf)
- Device trust (device certificates, hardware tokens)
- Conditional access policies (compliant device required)
- BYOD vs corporate-owned policies
- Remote wipe capabilities
- Endpoint detection and response (EDR)

**Integration Points**:
- Zitadel conditional access
- SPIFFE/SPIRE for device identity
- Audit logs for device access

**Detailed content**: TBD in v1.4

⸻

# Implementation Roadmap (Updated for v1.3)

## M0 — Foundation ✅ COMPLETE
- Tenancy primitives
- Environment-first hierarchy
- Authorities & system roles
- SSO baseline
- REST with AIP-conformant pagination/filtering
- OTel collector
- Audit logs
- Initial Trust Center

## M1 — Events & Cost Tracking (Phase 1) 🔧 IN PROGRESS
- CloudEvents envelopes (mandatory)
- AsyncAPI catalog (versioned schemas)
- **Kafka/Redpanda deployment** (Phase 1 default)
- Outbox pattern + DLQs
- **Cost event schema** ✅
- **Inter-agent MCP overhead tracking** (Priority #2) ✅
- **Chargeback to business units** (Priority #4) ✅
- **Per-agent token attribution** (Priority #1) ✅
- **Real-time cost alerts** (Priority #5) ✅
- MCP server v1 (read-only tools)

## M2 — Multi-Model AI & Optimization (NEW)
- Model registry and versioning ✅
- Multi-provider integration (Anthropic, OpenAI, AWS Bedrock, Azure) ✅
- **Cost-based routing logic** (Priority #3) ✅
- Prompt caching + compression
- Model cascading (cheap → expensive)
- Fine-tuned model governance (CAIO approval workflow) ✅
- Token budget enforcement per agent

## M3 — Compliance & Governance (ENHANCED)
- AI decision logging
- Bias testing suite
- Human oversight workflows (HITL/HOTL)
- Data subject rights APIs (GDPR/CCPA)
- Consent management (BIPA)
- Evidence matrix automation

## M4 — Security Hardening (ENHANCED)
- SPIRE deployment (workload identity)
- Zitadel + OpenFGA integration
- mTLS for data-tier connections
- Supply chain security (SBOM, scanning)
- Penetration testing (annual)

## M5 — Observability & Operations
- Prometheus/Grafana dashboards
- Cost dashboards (real-time + historical)
- AlertManager rules (cost anomalies, budget alerts)
- OpenTelemetry traces
- Jaeger/Tempo backend

## M6 — Production Hardening
- Kubernetes Helm charts
- HPA (Horizontal Pod Autoscaling)
- Network policies (Cilium/Calico)
- Redis Sentinel for HA
- PostgreSQL replication
- Disaster recovery runbooks

## M7 — Enterprise Features
- Business Unit cost chargeback ✅ (Already in M1)
- White-label customization
- Advanced administration UI
- Marketplace for third-party agents

## M8 — Latency Optimization (Phase 2: NATS Migration)
**Trigger**: When p99 latency <20ms becomes critical business requirement

- **NATS/JetStream deployment** alongside Kafka
- Dual-write period (both Kafka and NATS)
- Consumer migration (prioritize latency-sensitive flows)
- Performance benchmarking (target: ~100ms p99 improvement)
- Kafka retained for audit/compliance (7-year retention)
- Cost comparison (NATS vs Kafka operational costs)
- Rollback plan if NATS underperforms

⸻

# Checklists (Condensed)

## Core Architecture
- [ ] Environment-first hierarchy (env → org → BU → team → project)
- [ ] Multi-tenant boundaries enforced at all layers
- [ ] Authority-based filtering (OpenFGA)
- [ ] Cross-environment promotion (immutable artifacts)
- [ ] Comprehensive audit trail

## Cost Management (NEW)
- [ ] Cost event schema defined
- [ ] Per-agent token attribution
- [ ] Inter-agent MCP overhead tracking
- [ ] Multi-model cost comparison matrix
- [ ] Chargeback/showback to business units
- [ ] Real-time cost dashboards
- [ ] Budget alerts and throttling

## Multi-Model AI (NEW)
- [ ] Model registry with pricing + performance
- [ ] Cost-based routing logic
- [ ] Fallback chains (primary → backup)
- [ ] Prompt caching enabled
- [ ] Token budgets per agent
- [ ] Fine-tuned model lifecycle

## Event-Driven Architecture
- [ ] Event bus deployed (Kafka/NATS/EventBridge)
- [ ] CloudEvents envelope mandatory
- [ ] AsyncAPI contracts published
- [ ] Transactional outbox pattern
- [ ] DLQ handling
- [ ] Consumer idempotency

## Compliance & AI Governance (ENHANCED)
- [ ] AI decision logging (7-year retention)
- [ ] Risk tier classification (EU AI Act)
- [ ] Bias testing suite
- [ ] Human oversight workflows
- [ ] Data subject rights APIs
- [ ] Consent management (biometrics)
- [ ] Evidence matrix (regulation → capability → artifact)

## Security (ENHANCED)
- [ ] TLS 1.3 everywhere
- [ ] AES-256 at rest (KMS-backed)
- [ ] Zero-trust networking (SPIRE)
- [ ] LangGraph tool hardening (schema validation)
- [ ] MCP authentication and authorization
- [ ] FastAPI security baseline
- [ ] Supply chain security (SBOM)

## Observability
- [ ] OTel SDKs instrumented
- [ ] Cost telemetry pipeline
- [ ] Prometheus + Grafana
- [ ] Distributed tracing (Tempo/Jaeger)
- [ ] Tenant-scoped dashboards
- [ ] SLO definitions

⸻

# Appendices

## A. Cost Model Example (A2A Voice Platform - 2025)

### Production (100 Concurrent Sessions/Month)

**Baseline Costs** (No Optimization):

| Component | Monthly Cost | Per-Session Cost | Notes |
|-----------|-------------:|----------------:|-------|
| **LLM (Claude Sonnet 4, 1.2M tokens)** | $3,600 | $36.00 | 20% more tokens (A2A orchestration) |
| **STT (Deepgram Nova-2, 100 hrs)** | $1,500 | $15.00 | - |
| **TTS (ElevenLabs Turbo v2.5, 100 hrs)** | $2,400 | $24.00 | Upgraded model |
| **Compute (EKS)** | $550 | $5.50 | 2025 pricing |
| **Database (RDS)** | $220 | $2.20 | 2025 pricing |
| **Redis** | $165 | $1.65 | 2025 pricing |
| **Storage (S3)** | $65 | $0.65 | 2025 pricing + more logs |
| **Total** | **$8,500** | **$85.00** | - |

**Per-Minute Cost**: ~$0.028

**Optimized Costs** (With A2A Efficiencies):

| Component | Monthly Cost | Per-Session Cost | Savings |
|-----------|-------------:|----------------:|--------:|
| **LLM (Multi-model routing)** | $2,160 | $21.60 | 40% (cascading) |
| **LLM (Prompt caching)** | $1,512 | $15.12 | 30% (cache hits) |
| **STT (Deepgram Nova-2)** | $1,500 | $15.00 | - |
| **TTS (ElevenLabs Turbo v2.5)** | $2,400 | $24.00 | - |
| **Compute (EKS)** | $550 | $5.50 | - |
| **Database (RDS)** | $220 | $2.20 | - |
| **Redis** | $165 | $1.65 | - |
| **Storage (S3)** | $65 | $0.65 | - |
| **Total** | **$6,412** | **$64.12** | **24.5%** |

**Per-Minute Cost**: ~$0.021 (optimized)

**Key Insights (2025)**:
1. **LLM costs remain dominant** - 71% of optimized spend (down from 90% baseline)
2. **A2A orchestration adds cost** - +20% tokens but enables intelligence
3. **Multi-model routing saves 40%** - GPT-4o-mini for simple queries
4. **Prompt caching saves 30%** - System prompts cached across sessions
5. **Infrastructure is 12% of total** - Cloud costs are not the bottleneck

## B. Regulatory Checklist (Snapshot)

### U.S. Federal
- [ ] HIPAA (statute)
- [ ] HIPAA Security Rule (+ proposed NPRM updates)
- [ ] HIPAA Privacy Rule
- [ ] HITECH Act
- [ ] Section 1557 of ACA (non-discrimination)
- [ ] FTC Act § 5 (unfair/deceptive practices)
- [ ] FCRA (credit reporting)
- [ ] ECOA (credit decisioning fairness)

### International
- [ ] EU GDPR
- [ ] EU AI Act (in force; staged applicability)
- [ ] UK Data Protection Act + ICO AI guidance
- [ ] Canada PIPEDA (post-C-27)
- [ ] Brazil LGPD
- [ ] OECD AI Principles
- [ ] ISO/IEC 42001 (AI Management Systems)

### U.S. State-Level
- [ ] Comprehensive privacy laws (CA, VA, CO, UT, CT, etc.)
- [ ] Illinois BIPA (biometric privacy)
- [ ] Texas & Washington biometric laws
- [ ] Colorado AI Act (enacted, delayed, active rulemaking)
- [ ] Utah AI Policy Act (amended, extended)
- [ ] Other state AI bills (disclosure, bias, healthcare, employment)

## C. Event Bus Decision Matrix

| Criterion | Kafka | NATS | AWS EventBridge | RabbitMQ | Pulsar |
|-----------|-------|------|-----------------|----------|--------|
| **Throughput** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Latency** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Ops Complexity** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Multi-Tenancy** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Cost** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |

**Phased Recommendation**:
- **Phase 1 (Dev/Initial Prod)**: **Kafka/Redpanda** - Proven, audit retention, rich ecosystem
- **Phase 2 (Latency Optimization)**: **NATS/JetStream** - When squeezing 100ms out of p99 matters
- **AWS-exclusive**: **EventBridge + SQS**
- **Massive scale**: **Apache Pulsar**

**Key Insight**: Start with Kafka for reliability and compliance, migrate to NATS when latency optimization becomes critical.

## D. Security Hardening Checklist

- [ ] TLS 1.3 enforced everywhere
- [ ] AES-256-GCM at rest (all data stores)
- [ ] KMS-backed encryption keys
- [ ] Secrets Manager (no plaintext secrets)
- [ ] Zero secrets in environment variables
- [ ] mTLS for internal service calls (optional but recommended)
- [ ] SPIRE for workload identity
- [ ] LangGraph tools: schema-validated, no arbitrary code execution
- [ ] MCP servers: authenticated, rate-limited
- [ ] FastAPI: CORS strict, docs disabled in prod, rate limiting
- [ ] Supply chain: SBOM generation, dependency scanning
- [ ] Penetration testing: annual (minimum)

## E. Key Decisions Summary (v1.3)

### Architecture Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Hierarchy Model** | Environment → Org → BU → Team → Project → Resource | Dev/prod isolation at infrastructure level |
| **Event Bus (Phase 1)** | Kafka/Redpanda | Audit retention, ecosystem, proven at scale |
| **Event Bus (Phase 2)** | NATS/JetStream | Latency optimization (~100ms p99 improvement) |
| **Cost Alert Thresholds** | 75% warning, 95% critical | Early warning, throttle before runaway costs |
| **Model Approval Authority** | CAIO (consulted: CISO, CIO; informed: CTO) | AI governance centralized with security input |
| **Business Unit Definition** | P&L-capable but customer-flexible | NOT knowledge domain |

### Cost Tracking Priorities

| Priority | Capability | Status |
|----------|-----------|--------|
| 1️⃣ | Per-agent token attribution | v1.3 ✅ |
| 2️⃣ | Inter-agent MCP overhead tracking | v1.3 ✅ |
| 3️⃣ | Multi-model cost comparison | v1.3 ✅ |
| 4️⃣ | Chargeback to business units | v1.3 ✅ |
| 5️⃣ | Real-time cost alerts & budget enforcement | v1.3 ✅ |

### Detailed Sections (v1.3)

| Section | Status | Priority for v1.4 |
|---------|--------|-------------------|
| A2A Cost Tracking & FinOps | ✅ Detailed | - |
| Multi-Model AI Strategy | ✅ Detailed | - |
| Event-Driven Architecture (Expanded) | ✅ Detailed | - |
| Regulatory Compliance & AI Governance | ✅ Detailed | - |
| Security Architecture & Hardening | ✅ Detailed | - |
| Developer Experience & Local Development | ✅ Detailed | - |
| **Incident Response (ITIL v4)** | ✅ Detailed | - |
| **Change Management (ITIL v4)** | ✅ Detailed | - |
| **Disaster Recovery & Business Continuity** | ✅ Detailed | - |
| **Vendor & Subprocessor Management** | ✅ Detailed | - |
| Performance & Scalability Targets | 📋 Skeleton | 🔴 High (v1.4) |
| Endpoint Security & Device Management | 📋 Skeleton | 🟢 Low |

### Stakeholder Alignment

| Role | Primary Concerns Addressed in v1.3 |
|------|-----------------------------------|
| **CTO** | Architecture scalability, event bus strategy, dev experience |
| **CIO** | Infrastructure costs, environment isolation, ops complexity |
| **CISO** | Security hardening, zero-trust, compliance controls |
| **CFO** | Cost tracking, chargeback/showback, budget enforcement |
| **COO** | Operational reliability, incident response (skeleton) |
| **CAIO** | AI governance, model approval workflow, bias testing |
| **CRO** | Customer compliance (HIPAA, GDPR, AI Act), trust center |
| **CLO** | Legal/regulatory alignment, evidence matrix |

## F. Glossary

| Term | Definition |
|------|------------|
| **A2A** | Agent-to-Agent (architecture pattern) |
| **BIPA** | Biometric Information Privacy Act (Illinois) |
| **BU** | Business Unit (P&L or logical grouping) |
| **CAB** | Change Advisory Board (ITIL v4 change approval body) |
| **CAIO** | Chief AI Officer (emerging C-suite role) |
| **CloudEvents** | CNCF standard for event envelope format |
| **CRO** | Chief Revenue Officer |
| **DEK** | Data Encryption Key |
| **DPA** | Data Processing Agreement (GDPR) |
| **ECAB** | Emergency Change Advisory Board (ITIL v4) |
| **HITL** | Human-in-the-Loop (blocking approval) |
| **HOTL** | Human-on-the-Loop (notification only) |
| **ITSM** | IT Service Management (e.g., ITIL, ISO 20000) |
| **ITIL v4** | Information Technology Infrastructure Library version 4 (IT service management framework) |
| **KMS** | Key Management Service (AWS or equivalent) |
| **LocalStack** | Local AWS cloud emulator |
| **MCP** | Model Context Protocol (LLM-tool communication) |
| **NIST AI RMF** | National Institute of Standards and Technology AI Risk Management Framework |
| **OpenFGA** | Authorization system (Zanzibar-inspired) |
| **PIR** | Post-Implementation Review (ITIL v4 change management) |
| **RFC** | Request for Change (ITIL v4 change management) |
| **RLS** | Row-Level Security (PostgreSQL) |
| **SBOM** | Software Bill of Materials |
| **SPIFFE** | Secure Production Identity Framework for Everyone |
| **SPIRE** | SPIFFE Runtime Environment |
| **SSE-KMS** | Server-Side Encryption with KMS |
| **SVID** | SPIFFE Verifiable Identity Document |

## G. ITIL v4 & ITSM Framework References

### ITIL v4 Publications (Axelos)
- **ITIL 4 Foundation**: Core concepts, service value system, guiding principles
- **ITIL 4 Specialist: Create, Deliver and Support**: Change enablement, release management, incident management
- **ITIL 4 Specialist: High-velocity IT**: DevOps integration, continuous delivery, site reliability engineering
- **ITIL 4 Strategist: Direct, Plan and Improve**: Continual improvement, strategic planning

### Platform-Specific ITIL v4 Adaptations

**Agent-to-Agent (A2A) Extensions**:
```yaml
itil_v4_adaptations:
  incident_management:
    - Agent-aware classification (which agents affected)
    - AI decision logs for forensics
    - Cost anomaly detection as incident trigger
  
  change_management:
    - Model deployment as change type
    - CAIO approval required for AI changes
    - Bias testing in validation gates
    - Cost impact analysis mandatory
  
  problem_management:
    - Multi-agent root cause analysis
    - Token efficiency degradation patterns
    - MCP latency regression tracking
  
  service_level_management:
    - Latency SLAs per agent (p50, p95, p99)
    - Cost per request targets
    - Token efficiency benchmarks
```

### Compatible ITSM Tools
- **ServiceNow**: Full ITIL v4 implementation, CMDB, change workflows
- **Jira Service Management**: Agile-friendly ITSM, DevOps integrations
- **BMC Helix**: Enterprise ITSM with AI/ML capabilities
- **Ivanti**: ITSM with endpoint management integration
- **Cherwell**: Customizable ITSM platform

### Integration Patterns
```python
# Example: Create ITIL v4 change record from platform
class ITSMIntegration:
    async def create_change_record(
        self,
        change_type: ChangeType,
        affected_agents: List[str],
        cost_impact_usd: Decimal,
        risk_tier: RiskTier
    ) -> ChangeRecord:
        """Create change record in ITSM tool (ServiceNow, Jira, etc.)"""
        
        # Map platform concepts to ITIL v4
        itil_change = {
            "type": self._map_change_type(change_type),
            "category": "Agent Deployment" if affected_agents else "Configuration",
            "risk": self._map_risk_tier(risk_tier),
            "impact": self._calculate_impact(cost_impact_usd, affected_agents),
            "urgency": self._calculate_urgency(risk_tier),
            "configuration_items": affected_agents,
            "business_service": "AI Platform",
            "approval_required": change_type != ChangeType.STANDARD,
        }
        
        # Create via ITSM API
        return await self.itsm_client.create_change(itil_change)
```

### Recommended Training & Certification
- **ITIL 4 Foundation** (entry-level) - All platform team members
- **ITIL 4 Specialist: High-velocity IT** - DevOps/AgentOps engineers
- **ITIL 4 Practitioner: Change Control** - Release managers
- **ITIL 4 Managing Professional** - Platform leads

### References
- Axelos ITIL 4 Documentation: https://www.axelos.com/certifications/itil-service-management
- ISO/IEC 20000-1:2018 (IT service management standard)
- NIST SP 800-61 Rev 2 (Computer Security Incident Handling Guide)
- COBIT 2019 Framework (IT governance)

⸻

# Status & Ownership

**Version**: v1.3.0 DRAFT  
**Status**: Ready for stakeholder review  
**Document Owner**: Nathan Walker (nathan.walker@quant.ai)  
**Last Updated**: November 29, 2025  
**Next Review**: Quarterly (or upon major architectural change)

**Changelog**:
- v1.3.0: Environment-first hierarchy, comprehensive A2A cost tracking, multi-model AI strategy, expanded event architecture options, regulatory compliance integration, Agent Foundry security posture, missing section headings
- v1.2.0: Environment & profile strategy, AWS-first deployment, data handling & security baseline
- v1.1.0: Event-driven extensions, A2A architecture, AI governance primitives, initial cost tracking
- v1.0.0: Initial reference architecture

This scaffold is a living document. As implementation progresses and decisions evolve, update this record and increment the version number.

⸻

**Enterprise Multi‑Platform Architecture Scaffold — Master Document (v1.3.0 DRAFT)**

