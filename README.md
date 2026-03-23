# Claude Cowork Backend – Technical Specification

A production-ready backend specification for the **Claude Cowork** mobile AI assistant—a high-performance, collaborative workspace where teams interact with Claude as a coworker on projects.

---

## 📁 Workspace Contents

This folder contains the complete technical specification for Claude Cowork's serverless backend:

- **[Technical Spec Claude Cowork Backend.md](Technical%20Spec%20Claude%20Cowork%20Backend.md)** — Full 19-section specification covering architecture, data model, security, operations, and deployment roadmap
- **[ClaudeCoworkBE.png](ClaudeCoworkBE.png)** — Visual architecture diagram (color-coded layers, component dependencies, streaming flow emphasis)
- **[quesions.md](quesions.md)** — Design questions and requirements driving the architecture
- **README.md** — This file

---

## 🎯 Key Highlights

### Architecture
- **Hub-and-Spoke orchestration** centered on an asynchronous Lambda layer
- **Split API ingress:** AppSync (GraphQL) for mutations & state; API Gateway **WebSocket + Lambda Response Streaming** for per-token AI delivery
- **AI backbone:** AWS Bedrock (Claude 3.5 Sonnet) with Amazon Bedrock Knowledge Bases for **VPC-resident RAG** (no third-party vector DB)
- **Multi-step resilience:** SQS FIFO + Dead-Letter Queue for durable task queuing; AWS Step Functions for long-running workflows

### Performance Targets
| Metric | Target |
|--------|--------|
| API Availability | 99.95% (monthly) |
| Time-to-First-Token (p95) | ≤ 1.2s |
| Full Response Latency (p95) | ≤ 6.5s |
| Capacity | 750 concurrent users, 8,000 turns/hour |

### Security & Compliance
- **Multi-tenant isolation** via projected-scoped ProjectID (Cognito JWT)
- **VPC-resident data** (DynamoDB, Bedrock Knowledge Bases, OpenSearch)
- **PII scrubbing** pre-processor before LLM invocation
- **Encryption at rest** (KMS) and in transit (TLS 1.3)

### Operational Excellence
- **Idempotency guarantees** via RequestID-derived keys (prevent duplicate billing)
- **Model versioning** decoupled from deployments (SSM Parameter Store)
- **Adaptive fallback** to secondary model on timeout/throttle
- **Cached reads** (ElastiCache Redis) reduce DynamoDB churn on every turn
- **Distributed tracing** (AWS X-Ray) end-to-end

---

## 🚀 Quick Navigation

### For Architecture Review
1. **Start here:** [Section 2 - Architecture Overview](Technical%20Spec%20Claude%20Cowork%20Backend.md#2-architecture-overview) — Component roles and design decisions
2. **Then review:** [Visual Diagram](ClaudeCoworkBE.png) — See components and dataflow
3. **Deep dive:** [Section 3 - Data Model](Technical%20Spec%20Claude%20Cowork%20Backend.md#3-data-model--access-patterns) — DynamoDB schema and access patterns

### For Operations & Deployment
- **SLOs & capacity:** [Section 8](Technical%20Spec%20Claude%20Cowork%20Backend.md#8-service-level-objectives-slos--capacity-targets) — Availability targets, latency thresholds, and baseline capacity
- **Error handling:** [Section 10](Technical%20Spec%20Claude%20Cowork%20Backend.md#10-error-handling-retries-and-idempotency-matrix) — Retry policies and idempotency strategies
- **Operational profiles:** [Section 19](Technical%20Spec%20Claude%20Cowork%20Backend.md#19-operating-profiles) — Cost-first (Startup) vs. reliability-first (Enterprise) deployments
- **Roadmap:** [Section 18](Technical%20Spec%20Claude%20Cowork%20Backend.md#18-development-roadmap-phases-1-3) — Three-phase rollout plan with milestones

### For Security & Compliance
- [Section 5 - Security & Multi-Tenancy](Technical%20Spec%20Claude%20Cowork%20Backend.md#5-security--multi-tenancy)
- [Section 12 - AI Safety & Governance](Technical%20Spec%20Claude%20Cowork%20Backend.md#12-ai-safety--governance)

### For Testing & Deployments
- [Section 17 - Test Strategy](Technical%20Spec%20Claude%20Cowork%20Backend.md#17-test-strategy)
- [Section 16 - Release & Rollback Strategy](Technical%20Spec%20Claude%20Cowork%20Backend.md#16-release--rollback-strategy)

---

## 🏗️ Architecture at a Glance

```
┌─────────────────────────────────────────────────────────────┐
│                    MOBILE CLIENT                             │
└────────────────┬──────────────────────────────┬──────────────┘
                 │                              │
        ┌────────▼────────┐          ┌──────────▼────────┐
        │   AppSync GraphQL│          │ API Gateway WSS   │
        │   (queries + state)         │  (token stream)   │
        └────────┬────────┘          └──────────┬────────┘
                 │                              │
                 └────────────────┬─────────────┘
                                  │
                      ┌───────────▼───────────┐
                      │  Orchestrator Lambda  │
                      │  (Provisioned Concur.)│
                      └───────────┬───────────┘
                                  │
          ┌───────────┬───────────┼───────────┬──────────┐
          │           │           │           │          │
     ┌────▼──┐  ┌─────▼──┐  ┌────▼───┐  ┌───▼────┐  ┌──▼───┐
     │ SQS   │  │DynamoDB│  │Bedrock │  │ECache  │  │ Step │
     │ FIFO  │  │ (state)│  │(LLM)   │  │(cache) │  │ Func │
     └────┬──┘  └────────┘  │+ KB RAG│  └────────┘  │(jobs)│
          │                 └────────┘              └──────┘
          │
     ┌────▼──┐
     │  DLQ  │
     └───────┘
```

---

## 📚 How to Read This Project

**If you have 15 minutes:**
1. Read this README (you are here)
2. Scan [Section 2 - Architecture Overview](Technical%20Spec%20Claude%20Cowork%20Backend.md#2-architecture-overview)
3. Review the [diagram](ClaudeCoworkBE.png)

**If you have 1 hour:**
1. Read Sections 1–2 (Executive Summary & Architecture)
2. Skim Sections 3–7 (Data Model, Mobile UX, Security, AI Orchestration, Scalability)
3. Review [Section 8](Technical%20Spec%20Claude%20Cowork%20Backend.md#8-service-level-objectives-slos--capacity-targets) (SLOs & Capacity)

**If you're implementing Phase 1:**
1. Read Sections 1–8 (core architecture + SLOs)
2. Deep-dive [Section 3](Technical%20Spec%20Claude%20Cowork%20Backend.md#3-data-model--access-patterns) (DynamoDB patterns)
3. Review [Section 9](Technical%20Spec%20Claude%20Cowork%20Backend.md#9-api-contract-appendix-minimum-required) (API contracts)
4. Check [Section 18](Technical%20Spec%20Claude%20Cowork%20Backend.md#18-development-roadmap-phases-1-3) (Phase 1 deliverables)

**If you're planning operations:**
1. Section 8 (SLOs and capacity thresholds)
2. Section 10 (retry & error handling matrix)
3. Sections 14–17 (observability, cost model, release strategy, test strategy)
4. Section 19 (operating profiles)

---

## 🎮 Operating Profiles

Two deployment profiles are provided. Select based on your constraints:

| Profile | Startup (Phase 1) | Enterprise (Phase 3+) |
|---------|------------------|----------------------|
| **Availability Target** | 99.9% | 99.99% |
| **Time-to-First-Token (p95)** | 1.6s | 900ms |
| **Concurrent Users** | 400 | 2,000+ |
| **Provisioned Concurrency** | Peak-window only | Always-on |
| **Disaster Recovery RTO** | 4 hours | 30 minutes |
| **Best For** | MVP, cost-conscious | Mission-critical, SLA-driven |

See **[Section 19 - Operating Profiles](Technical%20Spec%20Claude%20Cowork%20Backend.md#19-operating-profiles)** for full override values.

---

## 🎓 Key Design Decisions

### Why API Gateway WebSocket Instead of AppSync for Streaming?
AppSync is event-driven and designed for subscriptions (e.g., "notify me when a message is posted"). It cannot sustain a continuous byte stream for per-token AI output. API Gateway WebSocket API natively supports Bedrock's `InvokeModelWithResponseStream` end-to-end.

### Why Amazon Bedrock Knowledge Bases Instead of Pinecone?
Bedrock Knowledge Bases is OpenSearch Serverless-backed and **VPC-resident**—all data stays within AWS. This aligns with enterprise compliance posture and eliminates third-party SaaS risk. Pinecone would violate data residency requirements.

### Why SQS + DLQ Instead of Direct Lambda Invocation?
Provides durability: if the Async Lambda is throttled or fails, tasks are buffered in SQS and replayed automatically (up to 3 times). The DLQ captures persistent failures for manual inspection and replay—preventing silent data loss.

### Why Step Functions for Long-Running Jobs?
Lambda has a 15-minute timeout. Analyzing 50 PDFs can exceed this. Step Functions offers durable state checkpointing between steps, built-in retry logic, and an audit trail—critical for batch workflows.

### Why Provisioned Concurrency on the Orchestrator?
Cold starts (1–3 seconds) are unacceptable on the critical interactive chat path. Provisioned Concurrency eliminates this penalty on the Orchestrator Lambda. Background/async Lambdas use on-demand scaling (cost trade-off is favorable).

---

## 📊 Document Structure

| Section | Focus | Key Takeaway |
|---------|-------|--------------|
| 1 | Executive Summary | Project scope and "Serverless-First" philosophy |
| 2 | Architecture Overview | Hub-and-spoke model; component table |
| 3 | Data Model | DynamoDB access patterns (5 defined) |
| 4 | Mobile-First Optimization | Streaming, optimistic UI, push-to-compute |
| 5 | Security & Multi-Tenancy | ProjectID isolation, encryption, PII scrubbing |
| 6 | AI Orchestration | System prompt, model versioning, fallback strategy |
| 7 | Scalability & Cost Guardrails | Auto-scaling, cold start mitigation, caching, observability |
| 8 | SLOs & Capacity | Availability, latency, throughput targets |
| 9 | API Contract | GraphQL operations, WebSocket envelope, error codes |
| 10 | Error Handling & Retry Matrix | Per-component policies, idempotency |
| 11 | Disaster Recovery | RTO/RPO targets, data retention, backup strategy |
| 12 | AI Safety & Governance | Model guardrails, token budgets, audit logging |
| 13 | Cost Model | Unit economics, budget alerts, scaling costs |
| 14 | Observability & Monitoring | X-Ray traces, CloudWatch metrics, alerting |
| 15 | Governance & Compliance | Encryption, audit trails, regulatory alignment |
| 16 | Release & Rollback Strategy | Canary rollouts, regression gates, rollback triggers |
| 17 | Test Strategy | Load testing, integration, chaos engineering |
| 18 | Development Roadmap | Phase 1 (streaming), Phase 2 (RAG knowledge), Phase 3 (scale) |
| 19 | Operating Profiles | Startup vs. Enterprise deployment variants |

---

## 🔗 External Links & Resources

- **[AWS Bedrock Docs](https://docs.aws.amazon.com/bedrock/)** — LLM inference, knowledge bases
- **[AWS AppSync Docs](https://docs.aws.amazon.com/appsync/)** — GraphQL service
- **[AWS Lambda Response Streaming](https://docs.aws.amazon.com/lambda/latest/dg/API_ResponseStreamingInvokeConfig.html)** — Per-token delivery
- **[DynamoDB Design Patterns](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)** — Access patterns & capacity planning
- **[AWS X-Ray](https://docs.aws.amazon.com/xray/)** — Distributed tracing

---

## ✅ Production Readiness Checklist

This specification is **production-ready** and includes:

- ✅ All 11 critical architectural issues resolved
- ✅ 19 comprehensive sections (architecture → operations → roadmap)
- ✅ Concrete SLO thresholds, metrics, and rollback triggers
- ✅ Two operating profiles (cost vs. reliability)
- ✅ Error handling matrix (retry policies per component)
- ✅ Multi-tenant security model with compliance alignment
- ✅ Visual architecture diagram with emphasis on critical paths
- ✅ Three-phase delivery roadmap with milestones

**Next Steps:**
1. Share this spec with your engineering team for Phase 1 planning
2. Select an operating profile based on launch constraints
3. Reference [Section 18](Technical%20Spec%20Claude%20Cowork%20Backend.md#18-development-roadmap-phases-1-3) for implementation sequencing

---

**Last Updated:** March 22, 2026  
**Status:** Production-Ready ✅
