# Agentic Management Systems — Intentions

> This document defines the vision, architecture, and technical scaffold for Agentic Management Systems (AMS). It serves as the north star for all implementation decisions.

---

## Vision

**One sentence:** A modular control plane for AI agents and automation systems — deploy only what you need.

**The problem we solve:** Organizations are deploying AI agents and automation systems faster than they can manage them. There is no visibility into what agents are doing, no control over costs, no systematic quality assurance, and no audit trail for compliance. This creates operational risk that compounds silently until it becomes a crisis.

**Our position:** We provide modular infrastructure that makes AI agents and automation systems observable, controllable, and accountable — enabling organizations to start small and scale confidently.

**Our approach:** Build incrementally. Ship the core platform with one module. Get customers. Learn what they need. Build the next module based on demand. The platform grows with validated customer needs, not assumptions.

---

## Architecture Philosophy

### Modular by Design

AMS is not a monolithic platform. It's a core foundation with independent feature modules that customers enable based on their needs.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AMS Core Platform                            │
├─────────────────────────────────────────────────────────────────────┤
│  Auth & Accounts  │  Agent Registry  │  Ingestion  │  Dashboard    │
└─────────────────────────────────────────────────────────────────────┘
                                 │
        ┌────────────┬───────────┼───────────┬────────────┐
        │            │           │           │            │
   ┌────┴────┐ ┌─────┴─────┐ ┌───┴───┐ ┌─────┴─────┐ ┌────┴────┐
   │  Cost   │ │  Trace    │ │ Alert │ │   Eval    │ │  Audit  │
   │ Intel   │ │ Observ.   │ │Engine │ │ Framework │ │Compliance│
   └─────────┘ └───────────┘ └───────┘ └───────────┘ └─────────┘
      MODULE       MODULE      MODULE      MODULE       MODULE
```

**Why modular:**
- Customers pay for what they use
- We build what customers actually want
- Lower barrier to entry (start with one module, add more)
- Each module validates independently
- Reduces upfront engineering investment

---

## Core Platform

The foundation that all modules depend on. Always included.

### Auth & Accounts
- User authentication (email/password, SSO later)
- Organization/workspace management
- API key management
- Basic RBAC (admin, member, viewer)

### Agent Registry
- CRUD for agents
- Agent metadata (name, version, owner, status, environment)
- Tagging and grouping
- Configuration storage
- Dependency tracking

**This is foundational** — you can't manage agents you don't know about. Every module references the registry.

### Ingestion Gateway
- Single endpoint for all telemetry
- Protocol: REST (simple) + OTLP (standard) 
- Authentication via API key
- Routes data to enabled modules
- Buffers/batches for efficiency

### Dashboard Shell
- Navigation and layout
- Module mounting points
- Settings and configuration
- User/org management UI

---

## Feature Modules

Each module is independent. Customers enable what they need. Modules have defined dependencies.

### Module 1: Cost Intelligence

**Purpose:** Track and control AI/automation spend.

**Dependencies:** Core only

**Capabilities:**
- Token usage tracking (by agent, model, workflow, customer)
- Cost calculation (configurable rates per provider/model)
- Budget limits and alerts
- Spend dashboards with drill-down
- Cost attribution and tagging
- Export for chargeback

**Data captured:**
```typescript
interface CostEvent {
  agentId: string;
  timestamp: Date;
  provider: string;        // 'openai', 'anthropic', etc.
  model: string;
  tokensPrompt: number;
  tokensCompletion: number;
  cost: number;            // Calculated USD
  attribution?: {
    customerId?: string;
    workflowId?: string;
    tags?: string[];
  };
}
```

**Why first:** Immediate tangible value. Easy to show ROI. Every agent user has this problem.

---

### Module 2: Trace Observability

**Purpose:** See exactly what agents are doing, step by step.

**Dependencies:** Core only

**Capabilities:**
- Full trace capture (requests, LLM calls, tool calls, responses)
- Trace viewer UI (waterfall, timeline, detail panels)
- Search and filtering
- Payload inspection (prompts, completions, tool inputs/outputs)
- Latency breakdown
- Error highlighting

**Data captured:**
```typescript
interface Span {
  traceId: string;
  spanId: string;
  parentSpanId?: string;
  agentId: string;
  
  name: string;
  kind: 'llm' | 'tool' | 'chain' | 'retrieval' | 'custom';
  
  startTime: Date;
  endTime: Date;
  duration: number;
  
  status: 'ok' | 'error';
  error?: { type: string; message: string; stack?: string; };
  
  attributes: {
    'llm.provider'?: string;
    'llm.model'?: string;
    'llm.prompt'?: string;
    'llm.completion'?: string;
    'llm.tokens.prompt'?: number;
    'llm.tokens.completion'?: number;
    'tool.name'?: string;
    'tool.input'?: any;
    'tool.output'?: any;
    [key: string]: any;
  };
}
```

**Why second:** Debugging is the other universal pain point. "WTF happened?" needs an answer.

---

### Module 3: Alerting Engine

**Purpose:** Proactive notification when things go wrong.

**Dependencies:** Core + (Cost Intelligence OR Trace Observability)

Alerting needs data to alert on. Customer must have at least one data-producing module enabled.

**Capabilities:**
- Rule-based alerts (thresholds, conditions)
- Anomaly detection (statistical baselines)
- Alert routing (Slack, email, webhook, PagerDuty)
- Escalation policies
- Alert history and acknowledgment
- Deduplication and throttling

**Alert types:**
- Cost: Spend exceeds budget, unexpected spike
- Traces: Error rate spike, latency degradation
- Availability: Agent not reporting, ingestion gap

**Data model:**
```typescript
interface AlertRule {
  id: string;
  name: string;
  enabled: boolean;
  
  scope: {
    agentIds?: string[];
    tags?: string[];
  };
  
  condition: {
    metric: string;           // 'error_rate', 'p99_latency', 'daily_cost'
    operator: 'gt' | 'lt' | 'gte' | 'lte';
    threshold: number;
    window: string;           // '5m', '1h', '24h'
  };
  
  notifications: {
    channels: string[];       // Slack channel IDs, email addresses, etc.
    throttle: string;         // '15m' — don't re-alert within window
  };
}
```

---

### Module 4: Evaluation Framework

**Purpose:** Continuous quality assurance for agent outputs.

**Dependencies:** Core + Trace Observability

Evals run against trace data. Trace module must be enabled.

**Capabilities:**
- Correctness checks (expected vs actual)
- Regression testing (compare against baseline)
- Safety checks (PII, harmful content)
- Quality scoring (relevance, coherence)
- Custom evaluators (user-defined criteria)
- Scheduled batch evals
- Pre-deployment CI integration

**Evaluation types:**
```typescript
interface Evaluator {
  id: string;
  name: string;
  type: 'deterministic' | 'llm_judge' | 'custom';
  
  // For deterministic
  rules?: {
    field: string;
    condition: string;
    expected: any;
  }[];
  
  // For LLM judge
  judgeConfig?: {
    model: string;
    prompt: string;
    scoreRange: [number, number];
  };
  
  // For custom
  webhookUrl?: string;
}
```

---

### Module 5: Audit & Compliance

**Purpose:** Security, permissions, and compliance infrastructure.

**Dependencies:** Core only

**Capabilities:**
- Immutable audit log of all agent actions
- User activity logging within AMS
- Compliance reports (who, what, when, why)
- Data retention policies
- Export for auditors
- Enhanced RBAC (custom roles, fine-grained permissions)

**Audit record:**
```typescript
interface AuditEntry {
  id: string;
  timestamp: Date;
  
  actor: {
    type: 'agent' | 'user' | 'system';
    id: string;
    name: string;
  };
  
  action: string;             // 'llm.completion', 'tool.execute', 'config.update'
  resource: string;           // What was acted upon
  
  outcome: 'success' | 'failure';
  
  details: Record<string, any>;
  
  retention: {
    policy: string;
    expiresAt?: Date;
  };
}
```

---

## Module Dependency Matrix

| Module | Requires |
|--------|----------|
| Cost Intelligence | Core |
| Trace Observability | Core |
| Alerting Engine | Core + (Cost OR Traces) |
| Evaluation Framework | Core + Traces |
| Audit & Compliance | Core |

---

## Tech Stack

### Core Platform

| Component | Technology | Rationale |
|-----------|------------|-----------|
| Language | Go | Performance, single binary, good for services |
| API | REST (Chi/Echo) | Simple, universal |
| Database | PostgreSQL | Reliable, JSONB for flexibility, great ecosystem |
| Auth | Custom + JWT | Keep it simple initially, add SSO later |
| Frontend | React + TypeScript | Ecosystem, hiring, component libraries |
| Styling | Tailwind CSS | Fast iteration |

### Module-Specific Additions

| Module | Additional Tech | Why |
|--------|-----------------|-----|
| Cost Intelligence | PostgreSQL (same DB) | Aggregations are simple, no new infra |
| Trace Observability | ClickHouse | Columnar, built for traces at scale |
| Alerting Engine | Redis (for state) | Fast checks, rule evaluation |
| Evaluation Framework | Job queue (River/Faktory) | Async eval execution |
| Audit & Compliance | PostgreSQL (append-only table) | Immutability, querying |

### Principle: Add Infrastructure Only When Needed

- Phase 1: PostgreSQL only
- Phase 2: Add ClickHouse when trace volume requires it
- Phase 3: Add Redis when alerting needs fast state
- Don't pre-optimize. Let customer scale drive infrastructure decisions.

---

## SDK Strategy

### Core SDK (ships with platform)

Minimal, non-blocking, fail-safe.

```python
from ams import AMS

ams = AMS(api_key="...", agent_id="my-agent")

# Register cost event
ams.track_cost(
    provider="openai",
    model="gpt-4",
    tokens_prompt=150,
    tokens_completion=80
)

# Register trace span
with ams.span("llm.completion", kind="llm") as span:
    span.set_attribute("llm.model", "gpt-4")
    response = openai.chat.completions.create(...)
    span.set_attribute("llm.completion", response.choices[0].message.content)
```

### Auto-Instrumentation (later)

- OpenAI SDK wrapper
- Anthropic SDK wrapper  
- LangChain callback handler
- LlamaIndex callback handler

Build these as customer demand reveals which frameworks matter.

---

## Implementation Phases

### Phase 1: Core + Cost Intelligence
**Goal:** Paying customers tracking agent costs.

**Deliverables:**
- [ ] Core platform (auth, registry, ingestion, dashboard shell)
- [ ] Cost Intelligence module (full)
- [ ] Python SDK (manual instrumentation)
- [ ] Basic documentation

**Tech:** PostgreSQL only. Single Go service. Simple React dashboard.

**Deployment:** Single VM or Railway/Render. No Kubernetes.

**Timeline:** 4-6 weeks

**Exit criteria:** 3 paying customers using cost tracking.

---

### Phase 2: Trace Observability
**Goal:** Customers can debug agent behavior.

**Deliverables:**
- [ ] Trace Observability module (full)
- [ ] Trace viewer UI
- [ ] SDK span instrumentation
- [ ] Search and filtering

**Tech:** Add ClickHouse for trace storage.

**Timeline:** 4-6 weeks

**Exit criteria:** Existing customers adopt traces. 2+ new customers for traces specifically.

---

### Phase 3: Alerting Engine
**Goal:** Proactive monitoring, not just dashboards.

**Deliverables:**
- [ ] Alerting Engine module
- [ ] Rule builder UI
- [ ] Slack + email integrations
- [ ] Anomaly detection (basic)

**Tech:** Add Redis for alert state.

**Timeline:** 4-6 weeks

**Exit criteria:** Customers creating alerts. Alert-driven customer acquisition.

---

### Phase 4: Evals + Audit
**Goal:** Enterprise-ready capabilities.

**Deliverables:**
- [ ] Evaluation Framework module
- [ ] Audit & Compliance module
- [ ] Enhanced RBAC
- [ ] Compliance exports

**Timeline:** 6-8 weeks

**Exit criteria:** Enterprise pilot or LOI.

---

### Phase 5: Scale & Polish
**Goal:** Production-grade for multiple organizations.

**Deliverables:**
- [ ] Multi-tenant hardening
- [ ] SSO/SAML
- [ ] TypeScript SDK
- [ ] Auto-instrumentation libraries
- [ ] API documentation / developer portal
- [ ] High-availability deployment

**Timeline:** Ongoing

---

## Pricing Model (Draft)

### Option A: Per-Module Pricing

| Tier | Modules Included | Price |
|------|------------------|-------|
| Starter | Core + 1 module | $500/mo |
| Pro | Core + 3 modules | $1,500/mo |
| Enterprise | All modules + support | Custom |

### Option B: Usage-Based

- Base platform: $200/mo
- Per 1M cost events: $50
- Per 1M spans: $100
- Alerting: $100/mo flat
- Evals: $0.01 per eval run

### Option C: Bundled Tiers

| Tier | Agents | Modules | Price |
|------|--------|---------|-------|
| Starter | Up to 10 | Cost + Traces | $500/mo |
| Growth | Up to 50 | All except Audit | $2,000/mo |
| Enterprise | Unlimited | All + SLA | Custom |

**Decision:** Validate with first customers. Start with simple flat pricing, add usage-based as we understand consumption patterns.

---

## Success Metrics

### Phase 1 Targets
- 3 paying customers
- $1,500+ MRR
- <1 week from signup to first cost event

### Phase 2 Targets
- 10 paying customers
- $10,000+ MRR
- 50% of customers using 2+ modules

### Long-term Targets
- 100 organizations
- $100K+ MRR
- Net revenue retention >120%

---

## Open Questions

1. **Managed service vs pure SaaS?**
   - Some customers may want us to operate AMS for them (white-glove)
   - This is higher margin but less scalable
   - Offer both? Managed as premium tier?

2. **Self-hosted option?**
   - Enterprises may require on-prem
   - Significant engineering overhead
   - Defer until enterprise demand is validated

3. **Which integrations first?**
   - Slack for alerts is obvious
   - What about PagerDuty, Opsgenie, Datadog forwarding?
   - Let customer requests drive priority

4. **Open source any components?**
   - SDK could be open source (drives adoption)
   - Core platform stays proprietary
   - Evaluate after Phase 2

---

## Principles

1. **Ship → Learn → Build.** Don't build features on assumptions. Ship the minimum, learn from customers, then build what they actually need.

2. **Modules earn their place.** A module gets built when customers ask for it (or pay for it). No speculative features.

3. **Infrastructure follows demand.** Start with PostgreSQL. Add ClickHouse when traces need it. Add Redis when alerting needs it. Don't pre-optimize.

4. **Simple > Clever.** Boring technology choices. Clear code. Obvious architecture. Complexity is earned, not assumed.

5. **Revenue validates.** The best signal that something matters is someone paying for it. Prioritize based on what customers will pay for.

---

*Last updated: 2026-03-04*
*Author: Appalachia Devs*
