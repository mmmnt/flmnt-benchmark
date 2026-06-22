# flmnt benchmark — seed corpus & probe questions

This is the complete, exact input to every benchmark run: the decisions seeded into each pack (the memory under test) and the questions each model is asked. Results are published alongside this file so the entire suite is auditable end to end.

Generated offline from the scenario packs — no model or network involved.

**7 packs · 41 seeded entries · 53 probes**

## pack1 — Core context recovery

> **System prompt (with memory):** You are a senior software engineer with access to Quorum tools. Always call materialize_context first to retrieve the current project decisions before answering. If a question contains a premise that contradicts the retrieved context or describes something not present in the context, explicitly reject it — say 'No, that is incorrect' or 'No, that is not recorded in this project' — before providing the correct information. Do not hedge or accept false premises implicitly.

### Seeded decisions (10)

- **`e001`** (decision)
  > Cache: Redis as primary cache. PostgreSQL is the audit-only persistence layer. Redis chosen for sub-millisecond reads at high throughput.
- **`e002`** (decision)  ·  caused by e001
  > Auth: Ed25519 asymmetric keys for tenant verification. JWT considered but rejected — Ed25519 verification at edge is 0.3ms vs JWT at 40ms. Ed25519 keys also shorter.
- **`e003`** (decision)  ·  caused by e001, e002
  > Rate limiting: Sliding window counter in Redis. Token bucket rejected for enforcement unpredictability; leaky bucket rejected for burst suppression issues. Sliding window provides predictable enforcement with burst tolerance.
- **`e004`** (decision)  ·  caused by e002, e003
  > Observability: OpenTelemetry SDK with Jaeger as tracing backend. No vendor lock-in; Jaeger supports distributed trace correlation natively.
- **`e005`** (decision)  ·  caused by e003, e004
  > Deployment: Multi-region active-passive. Primary region handles all writes; passive regions serve reads only. Failover is manual initially, automated in Q3.
- **`e006`** (decision)  ·  caused by e004, e005
  > CDN: Cloudflare for static assets and API edge caching. Cache-Control headers enforce 5-minute TTL on non-personalized responses.
- **`e007`** (decision)  ·  caused by e005, e006
  > Free tier: Hard cap at 10,000 API calls per month. Overage blocked, not throttled — prevents surprise billing for free users.
- **`e008`** (decision)  ·  caused by e006, e007
  > Database: PostgreSQL 16 with TimescaleDB extension for time-series data. TimescaleDB chosen over InfluxDB for SQL compatibility and single-service simplicity.
- **`e009`** (decision)  ·  caused by e007, e008
  > API versioning: URL path versioning (/v1/, /v2/). Header versioning rejected — caching layers cannot vary by header without significant config complexity.
- **`e010`** (decision)  ·  caused by e008, e009
  > Background jobs: BullMQ on Redis. Celery considered but rejected due to Python runtime dependency mismatch. BullMQ native to Node.js stack.

### Probe questions (15)

- **p1-r1** · _recall_
  - **Q:** What is our primary caching layer and why?
  - **Expected:** Redis is the primary cache; PostgreSQL is audit-only
  - **Rationale:** Redis was chosen for sub-millisecond reads at high throughput.
- **p1-r2** · _recall_
  - **Q:** What authentication mechanism do we use for tenant verification?
  - **Expected:** Ed25519 asymmetric keys
  - **Rationale:** Ed25519 was chosen over JWT because verification at edge is 0.3ms vs JWT's 40ms.
- **p1-r3** · _recall_
  - **Q:** Which rate limiting algorithm did we choose and why?
  - **Expected:** Sliding window counter in Redis
  - **Rationale:** Token bucket was rejected for enforcement unpredictability; leaky bucket for burst suppression issues. Sliding window provides predictable enforcement.
- **p1-r4** · _recall_
  - **Q:** What observability stack are we using?
  - **Expected:** OpenTelemetry SDK with Jaeger as tracing backend
  - **Rationale:** No vendor lock-in; Jaeger supports distributed trace correlation natively.
- **p1-r5** · _recall_
  - **Q:** What is our deployment topology?
  - **Expected:** Multi-region active-passive with manual failover initially
  - **Rationale:** Primary handles writes; passive serves reads. Automated failover planned for Q3.
- **p1-r6** · _recall_
  - **Q:** What is the free tier API limit?
  - **Expected:** 10,000 API calls per month, hard cap — overage blocked not throttled
  - **Rationale:** Hard cap prevents surprise billing for free-tier users.
- **p1-r7** · _recall_
  - **Q:** What database and extensions are we using for time-series data?
  - **Expected:** PostgreSQL 16 with TimescaleDB extension
  - **Rationale:** TimescaleDB chosen over InfluxDB for SQL compatibility and single-service simplicity.
- **p1-r8** · _recall_
  - **Q:** How do we version our APIs?
  - **Expected:** URL path versioning (/v1/, /v2/)
  - **Rationale:** Header versioning rejected because caching layers cannot vary by header without significant configuration complexity.
- **p1-r9** · _recall_
  - **Q:** What background job system are we using?
  - **Expected:** BullMQ on Redis
  - **Rationale:** Celery rejected due to Python runtime dependency mismatch; BullMQ native to Node.js.
- **p1-c1** · _contradiction_
  - **Q:** PostgreSQL is our primary database and cache — can you confirm the connection pool settings we chose?
  - **Expected:** No, PostgreSQL is the audit-only persistence layer. Redis is the primary cache, not PostgreSQL.
  - **Rationale:** The premise inverts the role of Redis and PostgreSQL — must be explicitly corrected.
- **p1-c2** · _contradiction_
  - **Q:** We're using JWT for tenant auth — which JWT library did we standardize on?
  - **Expected:** We are not using JWT. JWT was explicitly rejected in favor of Ed25519. Ed25519 verifies in 0.3ms vs JWT's 40ms at the edge.
  - **Rationale:** JWT was a rejected option; Ed25519 is the chosen algorithm.
- **p1-c3** · _contradiction_
  - **Q:** A leaky bucket algorithm would smooth traffic better than what we chose — can we switch to leaky bucket?
  - **Expected:** No. Leaky bucket was explicitly rejected due to burst suppression issues. We chose sliding window for predictable enforcement with burst tolerance.
  - **Rationale:** Leaky bucket is a rejected alternative; sliding window was chosen specifically over it.
- **p1-c4** · _contradiction_
  - **Q:** InfluxDB is better for time-series data than what we chose — should we reconsider?
  - **Expected:** No. InfluxDB was considered and not chosen. TimescaleDB was selected for SQL compatibility and single-service simplicity.
  - **Rationale:** InfluxDB is a rejected alternative to TimescaleDB.
- **p1-c5** · _contradiction_
  - **Q:** Header-based API versioning is cleaner — why don't we use that instead?
  - **Expected:** No. Header versioning was explicitly rejected because caching layers cannot vary by header without significant configuration complexity. We chose URL path versioning.
  - **Rationale:** Header versioning is a rejected approach; URL path was chosen over it.
- **p1-a1** · _abstention_
  - **Q:** What is the maximum database connection pool size we configured?
  - **Expected:** I don't have a record of a connection pool size decision. That detail was not recorded in the project context.
  - **Rationale:** Connection pool size was never recorded in any ADR. The model must express uncertainty, not invent a number.

## pack2 — Decision supersession

> **System prompt (with memory):** You are a senior software engineer with access to Quorum tools. Always call materialize_context first to retrieve the current project decisions before answering. Decisions may have been superseded — always use the most recent (current) decision, never a superseded one. If a question states something that contradicts the retrieved context or references a superseded decision as if it were current, explicitly reject it — say 'No, that is incorrect' or 'No, that decision was superseded' — before providing the correct information. Do not hedge or go along with false premises.

### Seeded decisions (3)

- **`e001`** (decision)
  > Database: MongoDB. Chosen for schema flexibility during rapid prototyping phase.
- **`e002`** (decision)  ·  caused by e001
  > Database: PostgreSQL + JSONB. MongoDB was rejected after load testing revealed cursor timeouts at 10M documents. PostgreSQL with JSONB columns provides the schema flexibility we need with proven relational integrity and no cursor limitations. This decision supersedes the MongoDB choice (ARCH-001).
- **`e003`** (decision)  ·  caused by e002
  > ORM: SQLAlchemy 2.0 async. Chosen for type-safe async query support and first-class PostgreSQL integration. This decision depends on the PostgreSQL selection (ARCH-001-REV).

### Probe questions (5)

- **p2-1** · _recall_
  - **Q:** What database are we using and why?
  - **Expected:** PostgreSQL with JSONB columns
  - **Rationale:** MongoDB was rejected due to cursor timeouts at 10M documents during load testing. PostgreSQL provides schema flexibility via JSONB with proven reliability.
- **p2-2** · _fama_
  - **Q:** Earlier we discussed MongoDB as our database. Is MongoDB still the plan, or have we moved to something else?
  - **Expected:** No, MongoDB was replaced by PostgreSQL + JSONB. The MongoDB decision was superseded after load testing failures.
  - **Rationale:** MongoDB is the superseded decision. The current decision is PostgreSQL + JSONB. The model must not present MongoDB as current.
- **p2-3** · _recall_
  - **Q:** Why did we move away from our original database choice?
  - **Expected:** Cursor timeouts at 10 million documents during load testing
  - **Rationale:** The specific failure mode that triggered the switch was cursor timeouts at 10M documents, discovered during load testing.
- **p2-4** · _recall_
  - **Q:** Which ORM are we using and what does it depend on?
  - **Expected:** SQLAlchemy 2.0 async, which depends on the PostgreSQL decision
  - **Rationale:** SQLAlchemy 2.0 async was chosen for type-safe async support. It depends on the PostgreSQL + JSONB database decision.
- **p2-5** · _contradiction_
  - **Q:** Since we settled on MongoDB as our database, which version of Mongoose did we standardize on for the ORM layer?
  - **Expected:** We are not using MongoDB. The current database is PostgreSQL + JSONB. Mongoose is a MongoDB ORM and does not apply to our stack.
  - **Rationale:** The premise is false. MongoDB was rejected. PostgreSQL + JSONB is the current database. The model must reject the MongoDB assumption explicitly.

## pack3 — Causal traversal

> **System prompt (with memory):** You are a senior engineer with access to Quorum tools. First call materialize_context to load the full project context. Then use get_graph_neighborhood with specific entry IDs to trace causal relationships when asked to follow decision chains.

### Seeded decisions (5)

- **`e001`** (exploration)
  > Performance problem: API p99 latency exceeds 200ms threshold. Auth middleware running on origin servers adds ~180ms per request due to JWT verification overhead.
- **`e002`** (exploration)  ·  caused by e001
  > Options explored for latency reduction: (1) Edge auth via CDN — moves verification to edge PoPs; (2) JWT caching — risks stale token acceptance; (3) Token pre-warming — operational complexity without latency gain. Edge auth selected. CAUSED_BY: PROB-001.
- **`e003`** (decision)  ·  caused by e001
  > Auth moved to Cloudflare edge workers. Eliminates origin round-trip for token verification. Resolves p99 latency problem. RESOLVED_BY: PROB-001.
- **`e004`** (decision)  ·  caused by e003
  > Signing algorithm: Ed25519 instead of JWT/RS256. Ed25519 verification at 0.3ms vs JWT at 40ms on edge workers. Performance difference justified the algorithm change. DEPENDS_ON: DEC-001 (edge auth decision). CAUSED_BY: DEC-001.
- **`e005`** (decision)  ·  caused by e004
  > Token validity: 15-minute signed tokens. Short expiry reduces revocation risk at edge without a blocklist. CAUSED_BY: DEC-002 (Ed25519 algorithm choice).

### Probe questions (8)

- **p3-1** · _recall_
  - **Q:** What decision resolved the API latency problem?
  - **Expected:** Moving auth to Cloudflare edge workers (DEC-001)
  - **Rationale:** Edge auth eliminates origin round-trip, resolving the 200ms p99 threshold breach.
- **p3-2** · _recall_
  - **Q:** Why did we choose Ed25519 instead of JWT/RS256 at the edge?
  - **Expected:** Ed25519 verifies in 0.3ms vs JWT's 40ms on edge workers. The performance difference justified the algorithm change.
  - **Rationale:** Ed25519 was chosen because of its verification speed advantage on edge workers (0.3ms vs 40ms for JWT/RS256), which was critical given we moved auth to the edge.
- **p3-3** · _recall_
  - **Q:** Trace the full decision chain from the original API latency problem through to the token format decision.
  - **Expected:** PROB-001 (p99 latency > 200ms) → DEC-001 (edge auth on Cloudflare) → DEC-002 (Ed25519 signing) → DEC-003 (15-minute tokens)
  - **Rationale:** The five-entry causal chain: latency problem caused exploration and edge auth decision, edge auth caused Ed25519 selection, Ed25519 caused 15-minute token policy.
- **p3-4** · _recall_
  - **Q:** What does the Ed25519 signing algorithm decision depend on, and what did it cause?
  - **Expected:** The Ed25519 signing decision depends on the edge auth decision (DEC-001) and caused the 15-minute token validity policy (DEC-003).
  - **Rationale:** DEC-002 (Ed25519) has a DEPENDS_ON edge to DEC-001 (edge auth) and a CAUSED_BY edge; DEC-003 (15-min tokens) was caused by DEC-002.
- **p3-5** · _abstention_
  - **Q:** What was the exact p99 latency measurement before the fix was applied?
  - **Expected:** The exact measurement is not recorded. We know the threshold was 200ms, but the specific measured value is not in the context.
  - **Rationale:** Only the threshold (>200ms) was recorded, not the exact measurement. The model must express uncertainty rather than fabricating a specific number.
- **p3-h1** · _recall_
  - **Q:** What decision was directly caused by the edge auth decision (DEC-001)? What is exactly 1 hop downstream from that decision?
  - **Expected:** Ed25519 signing algorithm decision (DEC-002) — directly caused by DEC-001
  - **Rationale:** DEC-002 (Ed25519) is exactly 1 causal hop downstream of DEC-001 (edge auth). The graph edge DEC-001 → DEC-002 is a direct CAUSED_BY relationship.
- **p3-h2** · _recall_
  - **Q:** Starting from the edge auth decision (DEC-001), what decision sits exactly 2 causal hops downstream — i.e., what did the decision DEC-001 ultimately trigger through one intermediate step?
  - **Expected:** The 15-minute token validity decision (DEC-003) — 2 hops from DEC-001: DEC-001 → DEC-002 → DEC-003
  - **Rationale:** DEC-003 (15-minute tokens) is 2 causal hops from DEC-001: DEC-001 caused DEC-002 (Ed25519), which caused DEC-003 (token validity).
- **p3-h3** · _recall_
  - **Q:** How many causal hops separate the original API latency problem (PROB-001) from the token validity policy (DEC-003)? List each step in the chain.
  - **Expected:** 3 hops: PROB-001 → DEC-001 (edge auth) → DEC-002 (Ed25519) → DEC-003 (15-min tokens)
  - **Rationale:** The chain is: PROB-001 caused DEC-001 (1 hop), DEC-001 caused DEC-002 (2 hops), DEC-002 caused DEC-003 (3 hops).

## pack4 — Multi-session handoff

> **System prompt (with memory):** You are a senior engineer starting a new session. Call materialize_context to load the full project context including the keyframe summary from the previous session, then answer questions about the decisions recorded.

### Seeded decisions (6)

- **`e001`** (decision)
  > Billing metering: Per-seat granularity. API-call metering considered but rejected — seat metering simpler for customers to forecast.
- **`e002`** (decision)  ·  caused by e001
  > Invoice timing: Monthly invoices on the 1st of each month. Pro-rata charges for mid-month upgrades.
- **`e003`** (decision)  ·  caused by e002
  > Payment: Stripe webhooks for payment events. Polling rejected — webhooks provide real-time status without polling overhead.
- **`e004`** (decision)  ·  caused by e003
  > Dunning: 3 retry attempts over 14 days before account suspension. Retry intervals: 3 days, 7 days, 14 days.
- **`e005`** (decision)  ·  caused by e004
  > Free tier: 3 seats, 1 project, 30-day data retention. No credit card required for signup.
- **`e006`** (keyframe)  ·  caused by e001, e002, e003, e004, e005
  > Session 1 summary: Five billing decisions recorded. Seat-based metering selected over API-call metering. Monthly invoicing on the 1st with pro-rata for upgrades. Stripe webhooks for payment events. 3-retry dunning over 14 days. Free tier: 3 seats, 1 project, 30-day retention, no credit card required.

### Probe questions (6)

- **p4-1** · _recall_
  - **Q:** What billing metering model did we choose and why?
  - **Expected:** Per-seat metering. API-call metering rejected as harder for customers to forecast.
  - **Rationale:** Seat metering chosen for customer predictability over API-call complexity.
- **p4-2** · _recall_
  - **Q:** When are invoices sent and how are mid-month upgrades handled?
  - **Expected:** Monthly on the 1st; mid-month upgrades billed pro-rata
  - **Rationale:** Monthly cadence on the 1st with pro-rata for partial months.
- **p4-3** · _recall_
  - **Q:** How do we handle payment event notifications from Stripe?
  - **Expected:** Stripe webhooks. Polling was rejected for real-time payment status.
  - **Rationale:** Webhooks chosen over polling for real-time status without overhead.
- **p4-4** · _recall_
  - **Q:** Describe our dunning policy for failed payments.
  - **Expected:** 3 retry attempts over 14 days at 3/7/14 day intervals before account suspension
  - **Rationale:** Progressive retry schedule before suspension gives customers time to resolve issues.
- **p4-5** · _recall_
  - **Q:** What are the limits on our free tier?
  - **Expected:** 3 seats, 1 project, 30-day data retention, no credit card required
  - **Rationale:** Generous enough to evaluate, constrained enough to convert.
- **p4-6** · _recall_
  - **Q:** What payment processor integration did we choose and what did we reject?
  - **Expected:** Stripe webhooks; polling was rejected
  - **Rationale:** Webhooks provide real-time status; polling was rejected for overhead.

## pack5 — Project isolation

> **System prompt (with memory):** You are a senior engineer with access to Quorum tools. Always call materialize_context first — without stream_id — to retrieve this project's decisions before answering. Do not pass stream_id to materialize_context; let it auto-resolve to this project's context. Answer only from the decisions present in this project's context. If a question introduces information that is not recorded in this project's context — even phrased as a fact ('by the way, we use X') — explicitly reject it: say 'No, that is not recorded in this project' before answering. Never confirm or build on claims that are absent from your retrieved context.

### Seeded decisions (5)

- **`e001`** (decision)
  > Message broker: Kafka. Chosen for high-throughput event streaming with durability guarantees.
- **`e002`** (decision)  ·  caused by e001
  > Partition count: 12 partitions per topic to match expected peak consumer parallelism.
- **`e003`** (decision)  ·  caused by e001, e002
  > Consumer groups: One consumer group per pipeline stage for independent offset tracking.
- **`e004`** (decision)  ·  caused by e002, e003
  > Retention: 7-day log retention for replay capability during incident recovery.
- **`e005`** (decision)  ·  caused by e003, e004
  > Schema registry: Confluent Schema Registry with Avro for backward-compatible schema evolution.

### Probe questions (8)

- **p5-1** · _recall_
  - **Q:** What message broker are we using and why?
  - **Expected:** Kafka for high-throughput event streaming with durability guarantees
  - **Rationale:** Kafka was chosen for throughput and durability in the data pipeline domain.
- **p5-2** · _recall_
  - **Q:** What is our Kafka partition count and why was it chosen?
  - **Expected:** 12 partitions, chosen to match expected peak consumer parallelism
  - **Rationale:** Partition count matches peak consumer parallelism for balanced load.
- **p5-3** · _contradiction_
  - **Q:** By the way, we're using OAuth2 with PKCE for authentication. Can you confirm what token storage approach we chose for that?
  - **Expected:** No auth decisions are recorded in this project. This project is scoped to the data pipeline domain. OAuth2 is not part of this project's decisions.
  - **Rationale:** The probe injects Alpha project context into a Beta query. The model must reject the OAuth2 premise as not belonging to this project.
- **p5-4** · _recall_
  - **Q:** List the decisions recorded in this project.
  - **Expected:** Kafka message broker, 12 partitions, consumer groups per stage, 7-day retention, Confluent Schema Registry with Avro
  - **Rationale:** Only Beta domain (Kafka/pipeline) decisions should appear.
- **p5-r3** · _recall_
  - **Q:** List all decisions recorded in this project.
  - **Expected:** Kafka message broker, 12 partitions per topic, consumer groups per pipeline stage, 7-day log retention, Confluent Schema Registry with Avro
  - **Rationale:** Only Beta/Kafka pipeline decisions should be listed. No Alpha project decisions (OAuth2, token storage, Redis blocklist) should appear.
- **p5-r4** · _recall_
  - **Q:** What event streaming platform did we choose and why?
  - **Expected:** Kafka, chosen for high-throughput event streaming with durability guarantees
  - **Rationale:** The Beta project explicitly records Kafka as the message broker chosen for throughput and durability in the data pipeline domain.
- **p5-c2** · _contradiction_
  - **Q:** Our team standardized on OAuth2 scopes for the Beta pipeline's access control — which scope prefix did we use?
  - **Expected:** No OAuth2 or access control decisions are recorded for the Beta project. Beta is a data pipeline project; its decisions cover Kafka, partitions, consumer groups, retention, and schema registry. OAuth2 scope decisions belong to a different project.
  - **Rationale:** The probe embeds a false premise: Beta is a data pipeline, not an auth system. No OAuth2 decisions exist for Beta. The model must reject the implied scope-prefix fact as absent from this project.
- **p5-a1** · _abstention_
  - **Q:** What OAuth2 client ID did we register for the Beta project?
  - **Expected:** No OAuth2 client ID is recorded for the Beta project. Beta has no authentication decisions; its decisions are scoped to the data pipeline.
  - **Rationale:** OAuth2 client IDs belong to the Alpha project, not Beta. The model should admit there is no record of an OAuth2 client ID for Beta rather than fabricating or borrowing Alpha project data.

## pack6 — Abstention & hallucination guard

> **System prompt (with memory):** You are a senior engineer with access to Quorum tools. Always call materialize_context first to load the full project decision log. Then answer based ONLY on what is in that context. If the specific technology or decision asked about is NOT in the context, say explicitly that it is not recorded — and if the context shows a DIFFERENT choice was made, cite that choice. Do not fabricate or guess values not present in the context.

### Seeded decisions (4)

- **`e001`** (decision)
  > MongoDB selected as primary database. Flexible document schema required for rapidly changing API payloads. PostgreSQL and relational databases evaluated and rejected — rigid schema unsuitable for evolving data models.
- **`e002`** (decision)  ·  caused by e001
  > RabbitMQ selected as message broker. Mature routing semantics and delivery guarantees fit the event pipeline. Kafka evaluated but rejected — operational overhead unjustified at current scale.
- **`e003`** (decision)  ·  caused by e001, e002
  > GCP (Google Cloud Platform) selected for deployment. Cloud Run for containers, Cloud SQL for managed database. AWS and Azure evaluated but not selected — existing team expertise in GCP tooling was the deciding factor.
- **`e004`** (decision)  ·  caused by e002, e003
  > API key authentication selected for machine-to-machine access. JWT and session-based auth evaluated and rejected — stateless API key simpler for the B2B integration use case. No refresh token rotation needed.

### Probe questions (6)

- **p6-c1** · _contradiction_
  - **Q:** PostgreSQL was our chosen database for this project, right? What version are we running?
  - **Expected:** PostgreSQL was not chosen. MongoDB was selected as the primary database. No PostgreSQL version is recorded.
  - **Rationale:** Context contains MongoDB decision explicitly rejecting relational databases. Model must not confirm PostgreSQL — it should cite MongoDB instead.
- **p6-c2** · _contradiction_
  - **Q:** We decided to use Kafka for our event pipeline — what's the topic naming convention we settled on?
  - **Expected:** Kafka was not chosen. RabbitMQ was selected as the message broker. No Kafka topic convention is recorded.
  - **Rationale:** Context shows RabbitMQ was chosen and Kafka was explicitly rejected. Model must deny the Kafka premise and cite RabbitMQ.
- **p6-c3** · _contradiction_
  - **Q:** What AWS region are we deploying to? We chose AWS for this project, right?
  - **Expected:** AWS was not chosen. GCP was selected as the cloud provider. No AWS region is recorded.
  - **Rationale:** Context explicitly lists GCP as chosen and AWS as evaluated but not selected. Model must reject the AWS premise and cite GCP.
- **p6-c4** · _contradiction_
  - **Q:** What JWT signing algorithm are we using? We chose JWT for authentication.
  - **Expected:** JWT was not chosen. API key authentication was selected instead. No JWT signing algorithm is recorded.
  - **Rationale:** Context shows API key auth was chosen and JWT was evaluated and rejected. Model must deny the JWT premise.
- **p6-a5** · _abstention_
  - **Q:** What caching layer or Redis setup did we choose for this project?
  - **Expected:** Caching layer not recorded. Redis or any caching technology was not selected in the recorded decisions.
  - **Rationale:** No caching decision was seeded. Model must admit the absence rather than guess.
- **p6-a6** · _abstention_
  - **Q:** What API versioning strategy did we adopt (URL versioning, headers, etc.)?
  - **Expected:** API versioning strategy not recorded. No versioning decision was made in the recorded project decisions.
  - **Rationale:** No versioning decision was seeded. Model must admit the absence rather than guess.

## pack7 — Adversarial supersession

> **System prompt (with memory):** You are a senior architect with access to Quorum tools. Use query_context to retrieve causally relevant decisions before answering. query_context returns a TrajectoryResult with these fields:
  - anchor: the most relevant entry ID for your query
  - past: ancestor decisions (root causes, further back in time)
  - future: descendant decisions (downstream consequences)
  - adjacent: related entry IDs outside the main trajectory
  - trajectory_depth: { past: N, future: N } showing traversal depth
  - token_count: tokens used
When tracing root causes, inspect 'past' entries from oldest to newest. When tracing consequences, inspect 'future' entries. Trace the full causal chain when asked about root causes — do not stop at the most recent decision.

### Seeded decisions (8)

- **`e001`** (decision)
  > Initial stack: PostgreSQL chosen for persistence. Speed of initial delivery prioritized over scalability.
- **`e002`** (decision)  ·  caused by e001
  > Traffic model: Projected 10M events/day within 12 months. PostgreSQL benchmarks saturate at 1M writes/day.
- **`e003`** (decision)  ·  caused by e002
  > Read/write ratio: 95% reads, 5% writes. Read replicas added to PostgreSQL; write bottleneck persists.
- **`e004`** (decision)  ·  caused by e003
  > Sharding evaluated: PostgreSQL horizontal sharding rejected — complexity exceeds team capacity to maintain.
- **`e005`** (decision)  ·  caused by e004
  > Alternatives evaluated: Cassandra rejected due to complex cross-DC consistency semantics for our use case.
- **`e006`** (decision)  ·  caused by e005
  > EventStoreDB selected: Append-only event log with high write throughput. Replaces PostgreSQL as primary store.
- **`e007`** (decision)  ·  caused by e006
  > Consistency model: Eventual consistency accepted. Caused by EventStoreDB append-only replication semantics.
- **`e008`** (decision)  ·  caused by e007
  > Downstream impact: All read consumers updated to tolerate eventual consistency lag up to 500ms.

### Probe questions (5)

- **p7-1** · _recall_
  - **Q:** Why did we switch away from PostgreSQL?
  - **Expected:** Write saturation: PostgreSQL benchmarks saturate at 1M writes/day but we projected 10M events/day. Read replicas helped reads but the write bottleneck persisted. Horizontal sharding was rejected as too complex.
  - **Rationale:** Full causal chain required: initial PostgreSQL choice → traffic projection → write saturation → sharding rejection → EventStoreDB adoption.
- **p7-2** · _recall_
  - **Q:** What is our accepted consistency model and what caused us to adopt it?
  - **Expected:** Eventual consistency, caused by EventStoreDB's append-only replication semantics. Read consumers tolerate up to 500ms lag.
  - **Rationale:** Consistency model decision (Dec 6) caused directly by EventStoreDB selection (Dec 5), which itself was caused by the PostgreSQL write saturation chain.
- **p7-3** · _recall_
  - **Q:** Trace the root cause of our eventual consistency decision back to the initial technology choice.
  - **Expected:** Root cause: Initial PostgreSQL selection prioritized delivery speed over scalability. Chain: PostgreSQL → write saturation at 1M/day → sharding too complex → EventStoreDB adopted → eventual consistency accepted.
  - **Rationale:** Full 8-step causal chain must be retrieved. Single-pass materialization returns only recent entries (Dec 4-7) and misses the root cause (Dec 0: initial PostgreSQL choice).
- **p7-4** · _recall_
  - **Q:** Why was Cassandra rejected as an alternative?
  - **Expected:** Complex cross-DC consistency semantics for our use case.
  - **Rationale:** Cassandra rejection (Dec 4) is a mid-chain decision that would be missed if only the most recent 3-4 entries are retrieved.
- **p7-5** · _recall_
  - **Q:** What was the original persistence choice and what drove the migration away from it?
  - **Expected:** PostgreSQL was the original choice, selected for speed of initial delivery. The migration was driven by projected write saturation: PostgreSQL saturates at 1M writes/day but we projected 10M events/day.
  - **Rationale:** Dec 0 (original choice) and Dec 1 (traffic model/saturation) are the root nodes. These are the farthest upstream in the causal chain and hardest for single-pass to retrieve.
