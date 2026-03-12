# 25 Real Cloud Architect Interview Design Problems

> Each problem includes: requirements, scale, high-level design, key decisions, and trade-offs.

---

## 1. Design a URL Shortener (bit.ly)

**Requirements:** Shorten URLs, redirect, analytics, custom aliases
**Scale:** 100M URLs, 10B redirects/day (~115K req/sec reads)

**Design:**
```
[Client] → [API Gateway] → [Write Service] → [DynamoDB: shortCode → longURL]
                         → [Read Service]  → [Redis Cache] → [DynamoDB]
                         → [Analytics]    → [Kinesis] → [S3/Redshift]
```

**Key Decisions:**
- Short code: base62 encoding of auto-increment ID (7 chars = 62^7 = 3.5T URLs)
- Read-heavy: cache in Redis (TTL 24h), 99% cache hit rate
- DynamoDB: partition key = shortCode, fast O(1) lookup
- Custom aliases: check uniqueness before insert

**Trade-offs:** Eventual consistency for analytics is acceptable. Strong consistency for redirect (read-your-writes).

---

## 2. Design Twitter/X

**Requirements:** Post tweets, follow users, home timeline, search
**Scale:** 300M DAU, 500M tweets/day, 28K tweets/sec peak

**Design:**
```
[Client] → [API Gateway] → [Tweet Service] → [Cassandra: tweets by userId]
                         → [Timeline Service] → [Redis: precomputed timelines]
                         → [Follow Service] → [Graph DB / DynamoDB]
                         → [Search Service] → [Elasticsearch]
                         → [Notification Service] → [Kafka] → [Push/Email]
```

**Timeline Generation (Fan-out on Write):**
- On tweet: push to followers' Redis timeline lists (async via Kafka)
- Celebrity problem (10M followers): fan-out on read for celebrities
- Hybrid: fan-out on write for regular users, fan-out on read for celebrities

**Key Decisions:**
- Cassandra for tweets: wide-column, high write throughput, time-series
- Redis for timelines: O(1) read, sorted sets by timestamp
- Kafka for async fan-out: decouple write from fan-out

---

## 3. Design Netflix

**Requirements:** Video upload, encoding, streaming, recommendations
**Scale:** 220M subscribers, 15% of global internet traffic

**Design:**
```
[Upload] → [S3 Raw] → [Encoding Farm (EC2/ECS)] → [S3 Encoded (multiple resolutions)]
                                                  → [CloudFront CDN]
[Client] → [API Gateway] → [Playback Service] → [CloudFront] → [Client]
                         → [Recommendation Service] → [Spark ML] → [Cassandra]
                         → [Search Service] → [Elasticsearch]
```

**Video Encoding:**
- Multiple resolutions: 240p, 360p, 480p, 720p, 1080p, 4K
- Multiple codecs: H.264, H.265, AV1
- Adaptive bitrate streaming: HLS or DASH
- Encoding is CPU-intensive: use Spot instances

**CDN Strategy:**
- Open Connect Appliances: Netflix's own CDN boxes in ISP data centers
- Pre-position popular content at edge
- Fallback to CloudFront for long-tail content

---

## 4. Design WhatsApp / Messaging System

**Requirements:** 1-1 messaging, group chat, online status, read receipts, media
**Scale:** 2B users, 100B messages/day

**Design:**
```
[Client A] → [WebSocket Server] → [Message Service] → [Cassandra: messages]
                                → [Presence Service] → [Redis: online status]
                                → [Push Service] → [APNs/FCM] (offline users)
[Client B] ← [WebSocket Server] ← [Message Queue (Kafka)]
```

**Message Delivery:**
- Online: deliver via WebSocket connection
- Offline: store in Cassandra, push notification via APNs/FCM
- Read receipts: client sends ACK, server updates status

**Group Chat:**
- Fan-out: write message to each member's inbox (for small groups)
- For large groups (1000+): fan-out on read

**Key Decisions:**
- WebSocket for real-time bidirectional communication
- Cassandra: partition by (userId, conversationId), sort by timestamp
- Message ordering: Lamport timestamps or server-assigned sequence numbers

---

## 5. Design Google Drive / Dropbox

**Requirements:** Upload/download files, sync across devices, share, versioning
**Scale:** 1B users, 2B files/day uploaded

**Design:**
```
[Client] → [API Gateway] → [Upload Service] → [S3 (chunked)]
                         → [Metadata Service] → [PostgreSQL: file metadata]
                         → [Sync Service] → [Kafka] → [Client devices]
                         → [Share Service] → [DynamoDB: permissions]
```

**Chunked Upload:**
- Split files into 4MB chunks
- Upload chunks in parallel
- Deduplication: hash each chunk (SHA-256), skip if already exists
- Resume interrupted uploads

**Sync:**
- Client polls for changes (long polling) or WebSocket
- Delta sync: only sync changed chunks
- Conflict resolution: last-write-wins or create conflict copy

---

## 6. Design Uber / Ride Sharing

**Requirements:** Request ride, match driver, real-time tracking, pricing, payments
**Scale:** 19M trips/day, 5M drivers

**Design:**
```
[Rider App] → [API Gateway] → [Ride Service] → [PostgreSQL: rides]
                            → [Matching Service] → [Redis: driver locations]
                            → [Pricing Service] → [ML model]
                            → [Payment Service] → [Stripe]
[Driver App] → [Location Service] → [Redis Geo: driver positions]
                                  → [Kafka: location stream]
```

**Driver Location:**
- Drivers send GPS every 4 seconds
- Store in Redis with GEOADD (geospatial index)
- Find nearby drivers: GEORADIUS command

**Matching Algorithm:**
- Find drivers within 2km radius
- Score by: distance, rating, car type
- Assign to nearest available driver
- If no driver: expand radius

**Surge Pricing:**
- Demand/supply ratio in geohash cell
- ML model predicts surge multiplier
- Real-time pricing update

---

## 7. Design a Payment System

**Requirements:** Process payments, handle failures, prevent double-charge, refunds
**Scale:** 1000 transactions/sec, 99.999% availability

**Design:**
```
[Client] → [API Gateway] → [Payment Service] → [Idempotency Store (Redis)]
                                             → [Payment Processor (Stripe/Adyen)]
                                             → [Transaction DB (PostgreSQL)]
                                             → [Ledger Service] → [Append-only DB]
                                             → [Notification Service]
```

**Idempotency:**
- Client sends idempotency key (UUID) with each request
- Server checks Redis: if key exists, return cached response
- Prevents double-charge on retry

**Exactly-once processing:**
- Outbox pattern: write to DB + outbox table in same transaction
- Worker reads outbox, calls payment processor, marks as processed

**Ledger:**
- Append-only (never update/delete)
- Double-entry bookkeeping
- Immutable audit trail

---

## 8. Design YouTube

**Requirements:** Upload video, transcode, stream, search, recommendations
**Scale:** 500 hours of video uploaded per minute, 1B hours watched/day

**Design:**
```
[Upload] → [API GW] → [Upload Service] → [S3 Raw]
                                       → [SQS] → [Transcoding Workers (EC2 Spot)]
                                               → [S3 Encoded] → [CloudFront]
[Watch] → [API GW] → [Video Service] → [CloudFront] → [Client (adaptive bitrate)]
[Search] → [Search Service] → [Elasticsearch]
[Recommend] → [ML Service] → [Spark] → [Cassandra]
```

**Transcoding Pipeline:**
- Multiple resolutions and codecs in parallel
- Use Spot instances (fault-tolerant, checkpointing)
- Thumbnail generation
- Audio extraction for captions

---

## 9. Design a Notification System

**Requirements:** Push, email, SMS notifications; user preferences; rate limiting
**Scale:** 10M notifications/day across channels

**Design:**
```
[Services] → [Notification API] → [Kafka: notification events]
                                → [Preference Service] → [DynamoDB: user prefs]
                                → [Rate Limiter] → [Redis: token bucket]
                                → [Channel Workers]
                                    → [Push Worker] → [APNs/FCM]
                                    → [Email Worker] → [SES/SendGrid]
                                    → [SMS Worker] → [Twilio/SNS]
                                → [Delivery Tracker] → [DynamoDB]
```

**Rate Limiting:**
- Per user: max 10 push/hour, 5 email/day
- Token bucket in Redis
- Digest: batch low-priority notifications

---

## 10. Design a Search Autocomplete System

**Requirements:** Real-time suggestions as user types, ranked by popularity
**Scale:** 10M queries/day, <100ms latency

**Design:**
```
[Client] → [API GW] → [Autocomplete Service] → [Redis Trie / Sorted Sets]
[Analytics] → [Kafka] → [Aggregation Service] → [Update Redis scores]
```

**Data Structure:**
- Trie in Redis: each prefix maps to top-10 suggestions
- Sorted sets: score = search frequency
- Update scores async (batch every hour)

**Optimization:**
- Cache top prefixes (1-3 chars) in memory
- CDN for common prefixes
- Debounce client requests (wait 200ms after keystroke)

---

## 11. Design a Rate Limiter

**Requirements:** Limit API requests per user/IP, distributed, low latency
**Scale:** 10M req/sec

**Algorithms:**
- Token Bucket: smooth bursts, allows burst up to bucket size
- Leaky Bucket: constant output rate, queues excess
- Fixed Window: simple, boundary problem
- Sliding Window Log: accurate, memory-intensive
- Sliding Window Counter: balance of accuracy and memory

**Design:**
```
[Client] → [API GW] → [Rate Limiter Middleware]
                           → [Redis: INCR + EXPIRE per key]
                           → Allow/Deny
                      → [Backend Service]
```

**Redis Implementation:**
```
key = "rate:{userId}:{window}"
INCR key → count
EXPIRE key 60 → TTL
if count > limit: return 429
```

---

## 12. Design a Distributed Job Scheduler

**Requirements:** Schedule jobs (cron + one-time), distributed, fault-tolerant
**Scale:** 1M jobs/day

**Design:**
```
[Job API] → [Job Store (PostgreSQL)] → [Scheduler Service]
                                            → [Redis: next-run priority queue]
                                            → [Kafka: job dispatch]
                                            → [Worker Pool]
                                            → [Job Status Store (DynamoDB)]
```

**Fault Tolerance:**
- Leader election for scheduler (Zookeeper/etcd)
- At-least-once delivery (idempotent jobs)
- Dead letter queue for failed jobs
- Heartbeat from workers

---

## 13. Design a Real-Time Leaderboard

**Requirements:** Update scores, get top-N, get user rank, real-time
**Scale:** 10M users, 1000 score updates/sec

**Design:**
```
[Game Service] → [Score API] → [Redis Sorted Set: ZADD, ZRANK, ZRANGE]
[Client] → [Leaderboard API] → [Redis] → [Top-N: ZREVRANGE 0 N]
                                       → [User rank: ZREVRANK userId]
```

**Redis Sorted Set:**
- ZADD leaderboard score userId
- ZREVRANGE leaderboard 0 99 WITHSCORES → top 100
- ZREVRANK leaderboard userId → user's rank
- O(log N) for all operations

---

## 14. Design a Content Delivery Network (CDN)

**Requirements:** Cache static content at edge, reduce origin load, global
**Scale:** 1B requests/day, 100TB data

**Design:**
```
[Client] → [DNS: GeoDNS routes to nearest PoP]
        → [Edge Server (PoP)]
              → Cache hit: return content
              → Cache miss: fetch from origin, cache, return
        → [Origin (S3/servers)]
```

**Cache Strategy:**
- Cache-Control headers: max-age, s-maxage
- Vary header: cache by Accept-Encoding, Accept-Language
- Purge API: invalidate on content update
- Consistent hashing: route same URL to same edge server

---

## 15. Design a Distributed Cache (Redis-like)

**Requirements:** Get/Set/Delete, TTL, high availability, horizontal scaling
**Scale:** 1M req/sec, 1TB data

**Design:**
```
[Client] → [Client Library (consistent hashing)]
        → [Cache Node 1] [Cache Node 2] [Cache Node 3]
              ↕ replication
        [Replica 1]     [Replica 2]     [Replica 3]
```

**Consistent Hashing:**
- Virtual nodes: each physical node has 150 virtual nodes
- Minimizes redistribution when nodes added/removed
- Client library routes to correct node

**Replication:**
- Primary-replica: writes to primary, reads from replica
- Async replication (low latency, risk of data loss)
- Sentinel: automatic failover

---

## 16. Design an E-Commerce Platform

**Requirements:** Product catalog, cart, orders, payments, inventory, search
**Scale:** 10M DAU, 100K orders/day, Black Friday 10x spike

**Design:**
```
[CloudFront + WAF]
[ALB]
[API Gateway]
  → [Product Service] → [DynamoDB: catalog] → [Elasticsearch: search]
  → [Cart Service] → [Redis: cart data]
  → [Order Service] → [Aurora PostgreSQL: orders]
      → [SQS] → [Inventory Service] → [DynamoDB: stock]
      → [SQS] → [Payment Service] → [Stripe]
      → [SNS] → [Notification Service]
[S3 + CloudFront: product images]
```

**Inventory:**
- Optimistic locking: check stock, decrement in transaction
- Reserve stock on add-to-cart (TTL 15 min)
- Release on cart expiry or order cancellation

---

## 17. Design a Banking System

**Requirements:** Accounts, transfers, transaction history, fraud detection
**Scale:** 1M accounts, 100K transactions/day, 99.999% availability

**Design:**
```
[API GW] → [Account Service] → [PostgreSQL: accounts, ACID]
         → [Transfer Service] → [Saga Orchestrator]
                                    → [Debit Account A]
                                    → [Credit Account B]
                                    → [Compensate on failure]
         → [Fraud Service] → [ML Model] → [Redis: rules]
         → [Ledger Service] → [Append-only DB]
         → [Audit Service] → [Immutable log (S3 + Glacier)]
```

**Consistency:**
- ACID transactions for account balance
- Saga for cross-account transfers
- Idempotency keys for all operations
- Two-phase commit for critical operations

---

## 18. Design a Hotel Booking System (like Booking.com)

**Requirements:** Search hotels, check availability, book, payments, reviews
**Scale:** 1M searches/day, 100K bookings/day

**Design:**
```
[Search] → [Elasticsearch: hotels, location, amenities]
[Availability] → [Redis: room availability cache] → [PostgreSQL: bookings]
[Booking] → [Reservation Service] → [Optimistic locking on rooms]
                                  → [Payment Service]
                                  → [Notification Service]
[Reviews] → [Review Service] → [DynamoDB]
```

**Availability:**
- Cache availability in Redis (TTL 5 min)
- Optimistic locking: check + reserve in transaction
- Overbooking prevention: row-level lock on room

---

## 19. Design a Stock Trading Platform

**Requirements:** Place orders, match orders, real-time prices, portfolio
**Scale:** 1M orders/day, microsecond matching latency

**Design:**
```
[Order API] → [Order Validator] → [Order Book Service (in-memory)]
                                      → [Matching Engine]
                                      → [Trade Execution]
                                      → [Kafka: trade events]
[Market Data] → [Kafka] → [Price Service] → [WebSocket: real-time prices]
[Portfolio] → [Portfolio Service] → [PostgreSQL]
```

**Order Book:**
- In-memory data structure (price-time priority)
- Persistent to Kafka for replay
- Matching: price-time priority (FIFO at same price)

---

## 20. Design a Recommendation Engine

**Requirements:** Personalized recommendations, real-time, A/B testing
**Scale:** 100M users, 1M recommendations/sec

**Design:**
```
[User Events] → [Kafka] → [Feature Store (Redis/DynamoDB)]
                        → [Batch Training (Spark)] → [Model Store (S3)]
[Recommendation API] → [Model Server] → [Feature Store]
                                      → [Candidate Generation]
                                      → [Ranking Model]
                                      → [Filter (already seen, blocked)]
                                      → [Top-N results]
```

**Algorithms:**
- Collaborative filtering: users with similar behavior
- Content-based: similar items
- Two-tower model: user embedding + item embedding
- Real-time features: recent clicks, current session

---

## 21. Design a Log Aggregation System (like Splunk)

**Requirements:** Collect logs, index, search, alert, dashboard
**Scale:** 1TB logs/day, 10K log sources

**Design:**
```
[Log Sources] → [Fluentd/Fluent Bit agents] → [Kafka]
                                            → [Elasticsearch (indexing)]
                                            → [S3 (archival)]
[Search API] → [Elasticsearch]
[Alert Engine] → [Kafka Streams] → [Alert Rules] → [PagerDuty/Slack]
[Dashboard] → [Kibana / Grafana]
```

---

## 22. Design a Multi-Region Active-Active System

**Requirements:** Zero downtime, data consistency, automatic failover
**Scale:** 3 regions, 99.999% availability

**Design:**
```
[Route 53 Latency Routing]
  ↓              ↓              ↓
[us-east-1]  [eu-west-1]  [ap-southeast-1]
  ↓              ↓              ↓
[ALB]          [ALB]          [ALB]
  ↓              ↓              ↓
[EKS]          [EKS]          [EKS]
  ↓              ↓              ↓
[DynamoDB Global Tables — multi-active replication]
[Aurora Global DB — primary in us-east-1, read replicas elsewhere]
[S3 CRR — cross-region replication]
```

**Conflict Resolution:**
- DynamoDB: last-write-wins (timestamp)
- Application-level: version vectors, CRDTs for counters

---

## 23. Design a CI/CD Pipeline

**Requirements:** Build, test, scan, deploy to multiple environments
**Scale:** 1000 deployments/day, 500 developers

**Design:**
```
[Git Push] → [GitHub Actions / Jenkins]
                → [Build: Docker image]
                → [Unit Tests]
                → [Security Scan: Trivy, SAST]
                → [Push to ECR/ACR]
                → [Update config repo (GitOps)]
                → [ArgoCD: deploy to dev]
                → [Integration Tests]
                → [Manual approval: staging]
                → [ArgoCD: deploy to staging]
                → [E2E Tests]
                → [Manual approval: production]
                → [ArgoCD: deploy to production (canary)]
                → [Monitor: error rate, latency]
                → [Auto-rollback if SLO breach]
```

---

## 24. Design a Healthcare Data Platform (HIPAA)

**Requirements:** Store PHI, audit access, encryption, compliance
**Scale:** 10M patient records, 1000 providers

**Design:**
```
[Provider App] → [API GW + WAF] → [Auth Service (OAuth 2.0 + MFA)]
                                → [Patient Service] → [Aurora (encrypted)]
                                → [Audit Service] → [Immutable log]
                                → [Document Service] → [S3 (SSE-KMS)]
[Compliance]
  - All data encrypted at rest (KMS CMK)
  - All data encrypted in transit (TLS 1.2+)
  - VPC with private subnets only
  - CloudTrail + Config for audit
  - GuardDuty for threat detection
  - BAA with AWS/Azure
```

---

## 25. Design a Global Gaming Backend

**Requirements:** Matchmaking, game state, leaderboards, real-time multiplayer
**Scale:** 10M concurrent players, <50ms latency

**Design:**
```
[Game Client] → [AWS Global Accelerator] → [Nearest Region]
                                         → [Matchmaking Service] → [Redis: player pool]
                                         → [Game Session Service] → [GameLift / EC2]
                                         → [State Sync] → [WebSocket / UDP]
                                         → [Leaderboard] → [Redis Sorted Sets]
                                         → [Analytics] → [Kinesis] → [S3]
```

**Matchmaking:**
- Skill-based: ELO rating within ±100 range
- Expand range if no match in 10 seconds
- Region preference: same region first

**Game State:**
- Authoritative server: server is source of truth
- Client-side prediction: reduce perceived latency
- Reconciliation: correct client state on mismatch
