# System Design Frameworks & Architect Thinking Models

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This document gives you the frameworks and mental models to structure your thinking in architecture interviews and real-world design work. It's not about specific technologies — it's about HOW to think and communicate as an architect.

The most important framework here is **RESHADED** — a step-by-step approach to answering any system design question in an interview.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| RESHADED | Requirements, Estimation, Storage, High-level, API, Detailed, Edge cases, Diagrams | A framework for structuring system design interview answers. |
| NFR | Non-Functional Requirement | How the system behaves (speed, reliability, security) vs what it does (features). |
| DAU | Daily Active Users | How many unique users use the system per day. |
| QPS | Queries Per Second | How many requests the system handles per second. |
| SLA | Service Level Agreement | A contract: "We guarantee X% uptime." |
| SLO | Service Level Objective | Internal target: "We aim for X% uptime." |
| ACID | Atomicity, Consistency, Isolation, Durability | Guarantees of traditional relational databases. |
| BASE | Basically Available, Soft state, Eventually consistent | Looser guarantees of NoSQL databases. |
| CAP | Consistency, Availability, Partition Tolerance | You can only guarantee 2 of 3 in a distributed system. |
| CQRS | Command Query Responsibility Segregation | Separate the write path from the read path. |
| DDD | Domain-Driven Design | A way of designing software around business domains. |
| ADR | Architecture Decision Record | A document that records WHY an architecture decision was made. |
| C4 | Context, Container, Component, Code | A model for drawing architecture diagrams at 4 levels of detail. |
| ATAM | Architecture Trade-off Analysis Method | A formal method for evaluating architecture trade-offs. |
| RFC | Request for Comments | A document proposing a change, shared for feedback before implementation. |
| TCO | Total Cost of Ownership | The full cost including hidden costs (maintenance, operations, etc.). |
| LRU | Least Recently Used | Cache eviction: remove the item used least recently when cache is full. |
| LFU | Least Frequently Used | Cache eviction: remove the item accessed fewest times. |
| TTL | Time To Live | How long a cached item is valid before it expires. |
| CDN | Content Delivery Network | Servers around the world that cache content close to users. |
| API | Application Programming Interface | A defined way for software to communicate. |
| REST | Representational State Transfer | A style for building APIs using HTTP verbs (GET, POST, PUT, DELETE). |
| gRPC | Google Remote Procedure Call | A fast binary protocol for service-to-service communication. |
| GraphQL | (not an acronym) | A query language for APIs. Clients specify exactly what data they need. |
| 2PC | Two-Phase Commit | A way to do transactions across multiple databases. Slow and fragile — avoid it. |
| Saga | (not an acronym) | A pattern for distributed transactions using compensating actions instead of 2PC. |
| DLQ | Dead Letter Queue | Where failed messages go after too many retry attempts. |
| P99 | 99th Percentile | 99% of requests are faster than this. The slowest 1% are slower. |

---

### RESHADED Framework in Plain English

When asked "design a Twitter" or "design a payment system" in an interview, use this structure:

**R — Requirements** (5 minutes):
Don't start drawing! Ask questions first. "How many users? What features are must-have? What's the latency requirement?" This shows you think before you act.

**E — Estimation** (3 minutes):
Quick math. "100M users × 10 requests/day = 1 billion requests/day = ~12,000 requests/second." This shows you understand scale.

**S — Storage** (2 minutes):
What data do you need to store? What database fits? (SQL for transactions, NoSQL for scale, Redis for caching)

**H — High-Level Design** (10 minutes):
Draw the big boxes: CDN, Load Balancer, Services, Databases, Cache, Queue. Don't go deep yet.

**A — API Design** (3 minutes):
Define the key endpoints: `POST /tweets`, `GET /timeline/{userId}`. Shows you think about interfaces.

**D — Detailed Design** (15 minutes):
Deep dive on 2-3 interesting components. The interviewer will guide you here.

**E — Edge Cases** (3 minutes):
What breaks? "What if the database goes down? What if we get 10x traffic?"

**D — Diagrams**:
Draw throughout. Always have a diagram on the board.

---

### Trade-off Articulation in Plain English

The magic phrase: **"I chose X over Y because [reason], with the trade-off that [downside], which we mitigate by [solution]."**

Example: "I chose DynamoDB over PostgreSQL because our access pattern is simple key lookups at 100K requests/second. The trade-off is we lose complex query capability, which is acceptable since we don't need ad-hoc queries in this service."

This shows you understand there's no perfect solution — only trade-offs.

---

### Database Selection in Plain English

Ask yourself:
1. **What are the access patterns?** Simple key lookups → NoSQL. Complex queries with joins → SQL. Full-text search → Elasticsearch.
2. **Do I need ACID transactions?** Yes → SQL. No → NoSQL is fine.
3. **What's the scale?** Under 10K req/sec → almost anything works. Over 100K req/sec → NoSQL or sharding.

---

### Circuit Breaker in Plain English

Imagine a light switch. When too many failures happen, the circuit "trips" (opens) and stops sending requests to the failing service. After a timeout, it tries again (half-open). If the service is healthy, the circuit closes again.

Without circuit breaker: Service A calls Service B. Service B is slow. Service A's threads pile up waiting. Service A crashes too. Cascading failure.

With circuit breaker: Service B is slow → circuit opens → Service A immediately returns an error or fallback response → Service A stays healthy.

---

### Resilience Patterns in Plain English

- **Circuit Breaker**: Stop calling a failing service. Fast-fail instead of waiting.
- **Retry with Backoff**: Try again, but wait longer each time (1s, 2s, 4s, 8s). Add random jitter to avoid thundering herd.
- **Bulkhead**: Isolate resources. If Service B is slow, only the "Service B thread pool" is exhausted — Service A can still serve other requests.
- **Timeout**: Always set timeouts. Never wait forever. "If no response in 5 seconds, give up."
- **Idempotency**: The same request can be safely retried. "If you charge me twice, the second charge is ignored."

---

> These frameworks help you structure your thinking and communication in interviews and real-world architecture work.

---

## 1. RESHADED Framework (System Design Interviews)

Use this to structure any system design answer:

### R — Requirements
```
Functional Requirements (what the system does):
  - Core features only (don't gold-plate)
  - Ask: "What are the must-have features for MVP?"
  - Example: "Users can post tweets, follow others, see timeline"

Non-Functional Requirements (how the system behaves):
  - Scale: "How many users? DAU? Requests/sec?"
  - Latency: "What's acceptable response time?"
  - Availability: "What's the SLA? 99.9%? 99.99%?"
  - Consistency: "Is eventual consistency acceptable?"
  - Durability: "Can we lose any data?"
  - Security: "Any compliance requirements?"
```

### E — Estimation
```
Back-of-envelope calculations:
  - Users: 100M DAU
  - Requests: 100M * 10 req/day = 1B req/day = ~12K req/sec
  - Storage: 100M * 1KB/tweet * 365 = ~36TB/year
  - Bandwidth: 12K req/sec * 1KB = 12MB/sec

Key numbers to memorize:
  - 1 million seconds ≈ 11.5 days
  - 1 billion = 10^9
  - 1 TB = 10^12 bytes
  - SSD: 500MB/s read, 200MB/s write
  - Network: 1Gbps = 125MB/s
  - Memory: 100ns access
  - SSD: 100μs access
  - HDD: 10ms access
  - Network (same DC): 0.5ms
  - Network (cross-region): 100-300ms
```

### S — Storage
```
Data model:
  - What entities exist? (User, Tweet, Follow, Like)
  - What are the relationships?
  - What are the access patterns? (read-heavy? write-heavy?)

Database choice:
  - Relational (ACID, complex queries, joins)
  - Document (flexible schema, nested data)
  - Key-Value (simple lookups, high throughput)
  - Wide-column (time-series, high write throughput)
  - Graph (relationship traversal)
  - Search (full-text search)
  - Time-series (metrics, IoT)
```

### H — High-Level Design
```
Draw the major components:
  - Client (web, mobile)
  - CDN / Edge
  - Load Balancer
  - API Gateway
  - Services (list them)
  - Databases
  - Cache
  - Message Queue
  - Storage

Don't go deep yet — just the boxes and arrows
```

### A — API Design
```
Define key APIs:
  POST /tweets
  GET /timeline/{userId}
  POST /follow/{userId}
  GET /search?q={query}

For each API:
  - Method + path
  - Request body/params
  - Response format
  - Auth requirements
```

### D — Detailed Design
```
Deep dive on 2-3 critical components:
  - The interviewer will guide you here
  - Common deep dives: database schema, caching strategy, 
    feed generation, search, notifications

For each component:
  - How does it work internally?
  - What are the trade-offs?
  - How does it scale?
```

### E — Edge Cases
```
Think about failure modes:
  - What if the database goes down?
  - What if a service is slow?
  - What if we get 10x traffic spike?
  - What about hot partitions?
  - What about duplicate messages?
  - What about network partitions?
```

### D — Diagrams
```
Always draw:
  - High-level architecture diagram
  - Data flow diagram
  - Database schema (if relevant)
  - Sequence diagram for critical flows
```

---

## 2. How to Run a 45-Minute Design Interview

```
Minutes 0-5: Requirements clarification
  - Ask about scale, users, features
  - Confirm functional + non-functional requirements
  - "Before I start designing, let me clarify a few things..."

Minutes 5-10: Estimation
  - Quick back-of-envelope
  - "Let me estimate the scale we're dealing with..."

Minutes 10-20: High-level design
  - Draw the major components
  - Explain your choices
  - "Here's my initial high-level design..."

Minutes 20-35: Deep dive
  - Go deep on 2-3 components
  - Let interviewer guide you
  - "Let me deep dive on the feed generation..."

Minutes 35-40: Bottlenecks & scaling
  - Identify bottlenecks
  - Propose solutions
  - "The main bottleneck here is the database..."

Minutes 40-45: Summary & trade-offs
  - Summarize your design
  - Acknowledge trade-offs
  - "To summarize, I chose X over Y because..."
```

---

## 3. Trade-off Articulation Framework

Always structure trade-offs as:
```
"I chose [Option A] over [Option B] because [reason], 
 with the trade-off that [downside of A], 
 which we can mitigate by [mitigation]."
```

Examples:
```
"I chose eventual consistency over strong consistency because 
 our read latency requirement is <50ms and strong consistency 
 would require cross-region coordination. The trade-off is 
 users might see slightly stale data, which we mitigate by 
 using read-your-writes consistency for the user's own actions."

"I chose microservices over a monolith because we have 
 10 independent teams and need to deploy independently. 
 The trade-off is operational complexity, which we address 
 with a strong platform team and GitOps."

"I chose DynamoDB over PostgreSQL because our access pattern 
 is simple key-value lookups at 100K req/sec. The trade-off 
 is we lose complex query capability, which is acceptable 
 since we don't need ad-hoc queries in this service."
```

---

## 4. Database Selection Framework

```
Step 1: What are the access patterns?
  - Simple key lookups → Key-Value (DynamoDB, Redis)
  - Complex queries, joins → Relational (PostgreSQL, MySQL)
  - Flexible schema, nested → Document (MongoDB, Cosmos DB)
  - Time-series data → TimescaleDB, InfluxDB, Prometheus
  - Graph traversal → Neo4j, Amazon Neptune
  - Full-text search → Elasticsearch, OpenSearch

Step 2: What are the consistency requirements?
  - ACID transactions → Relational
  - Eventual consistency OK → NoSQL

Step 3: What is the scale?
  - < 10K req/sec → Almost any database
  - 10K-100K req/sec → Need caching, read replicas
  - > 100K req/sec → NoSQL, sharding, or specialized

Step 4: What is the data model?
  - Structured, relational → RDBMS
  - Semi-structured → Document
  - Flat, high-throughput → Key-Value
  - Wide rows, time-series → Wide-column (Cassandra, HBase)
```

---

## 5. Caching Strategy Framework

```
Where to cache:
  1. Client-side (browser cache, mobile cache)
  2. CDN (static assets, API responses)
  3. API Gateway (response caching)
  4. Application cache (in-memory, Redis)
  5. Database cache (query cache, buffer pool)

Cache invalidation strategies:
  1. TTL (Time-To-Live): Simple, may serve stale data
  2. Write-through: Update cache on write (consistent, slower writes)
  3. Write-behind: Async cache update (fast writes, risk of loss)
  4. Cache-aside (lazy loading): Load on miss (most common)
  5. Read-through: Cache handles DB reads

Cache eviction policies:
  - LRU (Least Recently Used): Most common
  - LFU (Least Frequently Used): Better for skewed access
  - FIFO: Simple, not optimal
  - Random: Simple, acceptable

Cache sizing:
  - 80/20 rule: 20% of data = 80% of traffic
  - Cache the hot 20%
  - Monitor hit rate (target > 90%)
```

---

## 6. Scalability Patterns

### Horizontal Scaling
```
Add more instances (scale out):
  - Stateless services: easy to scale
  - Stateful services: need session management
  - Use load balancer to distribute traffic
  - Auto-scaling based on metrics
```

### Vertical Scaling
```
Add more resources to existing instance (scale up):
  - Simpler (no distributed system complexity)
  - Has limits (biggest instance type)
  - Downtime for resize (usually)
  - Use for: databases (initially), single-threaded apps
```

### Database Scaling
```
Read scaling:
  - Read replicas (async replication)
  - Caching layer (Redis, Memcached)
  - CQRS (separate read/write models)

Write scaling:
  - Sharding (horizontal partitioning)
  - Partitioning strategies:
    - Range: by date, ID range
    - Hash: consistent hashing
    - Directory: lookup table
  - Multi-master (complex, conflict resolution)

Connection scaling:
  - Connection pooling (PgBouncer, RDS Proxy)
  - Reduce connection overhead
```

### Async Processing
```
Problem: Synchronous processing limits throughput
Solution: Decouple with message queue

Before: Client → API → Process → DB → Response (slow)
After:  Client → API → Queue → Response (fast)
                          ↓
                    Worker → Process → DB
                    (async, can scale independently)
```

---

## 7. Microservices Design Principles

### Service Boundaries (Domain-Driven Design)
```
Bounded Context: A boundary within which a domain model is consistent
  - User Context: user profile, authentication
  - Order Context: order lifecycle, fulfillment
  - Payment Context: payment processing, refunds
  - Inventory Context: stock management

Each bounded context = one or more microservices
Services communicate via APIs or events (not shared DB)
```

### Database per Service
```
Why: Loose coupling, independent scaling, polyglot persistence
How: Each service owns its data, no shared tables

Service A → DB A (PostgreSQL)
Service B → DB B (DynamoDB)
Service C → DB C (MongoDB)

Challenge: Joins across services
Solution: API composition, CQRS, event-driven denormalization
```

### Service Communication
```
Synchronous (request-response):
  - REST: Simple, widely understood, HTTP overhead
  - gRPC: Binary, fast, strong typing, streaming
  - GraphQL: Flexible queries, reduces over-fetching

Asynchronous (event-driven):
  - Message queue (SQS, Service Bus): Point-to-point
  - Pub/Sub (SNS, Event Grid): One-to-many
  - Event streaming (Kafka, Event Hubs): Ordered, replayable

When to use async:
  - Long-running operations
  - Fan-out to multiple services
  - Decoupling for resilience
  - When caller doesn't need immediate response
```

---

## 8. Resilience Patterns

### Circuit Breaker
```
States:
  CLOSED: Normal operation, requests pass through
  OPEN: Too many failures, fast-fail (no requests to service)
  HALF-OPEN: Test request to see if service recovered

Thresholds:
  - Open when: 50% failure rate in 10-second window
  - Half-open after: 30 seconds
  - Close when: 3 consecutive successes

Libraries: Resilience4j (Java), Polly (.NET), Hystrix (deprecated)
```

### Retry with Exponential Backoff
```
Attempt 1: immediate
Attempt 2: wait 1s
Attempt 3: wait 2s
Attempt 4: wait 4s
Attempt 5: wait 8s + jitter (random 0-1s to avoid thundering herd)
Give up after 5 attempts

Retry on: transient errors (503, 429, network timeout)
Don't retry on: client errors (400, 401, 404)
```

### Bulkhead
```
Isolate resources to prevent cascade failures:
  - Thread pool per downstream service
  - Connection pool per database
  - Separate queues per priority

If Service B is slow:
  Without bulkhead: all threads blocked → Service A fails
  With bulkhead: only Service B thread pool exhausted → Service A still serves other requests
```

### Timeout
```
Always set timeouts:
  - Connection timeout: 1-3 seconds
  - Read timeout: 5-30 seconds (depends on operation)
  - Total timeout: sum of all downstream timeouts + buffer

Timeout hierarchy:
  Client timeout > API Gateway timeout > Service timeout > DB timeout
```

---

## 9. Architect Communication Framework

### C4 Model (for diagrams)
```
Level 1: Context Diagram
  - System in context of users and external systems
  - Audience: everyone (non-technical)

Level 2: Container Diagram
  - Major deployable units (web app, API, database)
  - Audience: technical stakeholders

Level 3: Component Diagram
  - Components within a container
  - Audience: developers

Level 4: Code Diagram
  - Classes, interfaces (rarely needed)
  - Audience: developers
```

### Presenting to Executives
```
Structure:
  1. Business problem (1 sentence)
  2. Proposed solution (1 sentence)
  3. Key benefits (3 bullets)
  4. Risks and mitigations (2-3 bullets)
  5. Cost and timeline (numbers)
  6. Recommendation

Avoid:
  - Technical jargon
  - Implementation details
  - Acronyms without explanation

Use:
  - Business outcomes (revenue, cost, risk)
  - Analogies
  - Simple diagrams
```

### Architecture Decision Record (ADR) Template
```markdown
# ADR-001: Use PostgreSQL for Order Service

## Status
Accepted

## Context
The Order Service needs a database that supports ACID transactions
for order state management. We process ~1000 orders/minute.

## Decision
We will use PostgreSQL (AWS Aurora PostgreSQL) for the Order Service.

## Consequences
Positive:
- ACID transactions ensure order consistency
- Rich query capabilities for reporting
- Familiar to the team

Negative:
- Vertical scaling limits (mitigated by Aurora's auto-scaling)
- Schema migrations require careful planning

## Alternatives Considered
- DynamoDB: Rejected because we need complex queries and transactions
- MongoDB: Rejected because ACID transactions are critical
```
