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