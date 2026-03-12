# Microservices Design Patterns — Deep Dive

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
Microservices are small, independent services that each do one thing well. But when you have 50 services talking to each other, new problems emerge: How do you handle a transaction that spans 3 services? What happens when one service is down? How do you route traffic? How do you keep data consistent?

This document covers the design patterns that solve these problems. Each pattern has a name, a problem it solves, and a solution.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| API | Application Programming Interface | A defined way for software to communicate. |
| API GW | API Gateway | A single entry point for all client requests. Handles auth, routing, rate limiting. |
| BFF | Backend for Frontend | A separate API Gateway for each type of client (web, mobile, partner). |
| APIM | API Management | Azure's API Gateway service. |
| REST | Representational State Transfer | A style for building HTTP APIs using standard verbs (GET, POST, PUT, DELETE). |
| gRPC | Google Remote Procedure Call | A fast binary protocol for service-to-service calls. Faster than REST. |
| GraphQL | (not an acronym) | A query language for APIs. Clients specify exactly what data they need. |
| mTLS | Mutual TLS | Both services prove their identity with certificates. Used for service-to-service auth. |
| SPIFFE | Secure Production Identity Framework For Everyone | A standard for giving services cryptographic identities. |
| Saga | (not an acronym) | A pattern for distributed transactions. Instead of one big transaction, use a series of smaller ones with compensating actions on failure. |
| CQRS | Command Query Responsibility Segregation | Separate the write path (commands) from the read path (queries). Different databases optimized for each. |
| CDC | Change Data Capture | Capturing every database change and streaming it elsewhere. Used to keep read models in sync. |
| DDD | Domain-Driven Design | A way of designing software around business domains (Order domain, Payment domain, etc.). |
| Bounded Context | (not an acronym) | A boundary within which a domain model is consistent. Each microservice typically owns one bounded context. |
| Aggregate | (not an acronym) | A cluster of related objects treated as a single unit. An Order aggregate includes OrderItems, ShippingAddress, etc. |
| Event Sourcing | (not an acronym) | Instead of storing current state, store every state change as an immutable event. Current state = replay of all events. |
| Outbox Pattern | (not an acronym) | Save to DB and publish an event atomically by writing to an "outbox" table in the same transaction. |
| Strangler Fig | (not an acronym) | A pattern for migrating a monolith to microservices incrementally, without a big-bang rewrite. |
| Sidecar | (not an acronym) | A helper container deployed alongside the main container. Handles cross-cutting concerns (logging, mTLS, tracing). |
| Envoy | (not an acronym) | A high-performance proxy used as a sidecar in Istio. |
| 2PC | Two-Phase Commit | A way to do transactions across multiple databases. Slow and fragile — avoid in microservices. |
| DLQ | Dead Letter Queue | Where failed messages go after too many retry attempts. |
| Idempotency | (not an acronym) | The same request can be safely retried. "Charge me twice" → second charge is ignored. |
| SQS | Simple Queue Service | AWS's message queue. |
| SNS | Simple Notification Service | AWS's pub/sub service. |
| Kafka | (not an acronym) | A high-throughput event streaming platform. Used for event sourcing and CDC. |
| Debezium | (not an acronym) | A CDC tool that reads database transaction logs and publishes changes to Kafka. |
| Temporal | (not an acronym) | A workflow orchestration platform. Good for long-running Saga orchestration. |
| Step Functions | (not an acronym) | AWS's workflow orchestration service. Good for Saga orchestration. |
| Istio | (not an acronym) | A service mesh. Handles mTLS, retries, circuit breaking, tracing for all services. |
| Linkerd | (not an acronym) | A lighter alternative to Istio. |
| Kong | (not an acronym) | An API Gateway with many plugins. |
| Traefik | (not an acronym) | A dynamic reverse proxy and load balancer. |

---

### Microservices vs Monolith in Plain English

**Monolith**: One big application. All code in one codebase, deployed together.
- Good for: small teams, simple apps, early-stage products
- Bad for: large teams (everyone steps on each other), different scaling needs per feature

**Microservices**: Many small services, each deployed independently.
- Good for: large teams, different scaling needs, different technology needs
- Bad for: small teams (too much overhead), simple apps (over-engineering)

The key question: "Do we have multiple teams that need to deploy independently?" If yes, microservices. If no, start with a monolith.

---

### API Gateway Pattern in Plain English

Without API Gateway: Every client (web app, mobile app, partner) calls each microservice directly. Each service must handle auth, rate limiting, CORS, etc.

With API Gateway: One entry point. The gateway handles:
- Authentication (validate JWT tokens)
- Rate limiting (max 100 requests/minute per user)
- Routing (send /users to User Service, /orders to Order Service)
- SSL termination
- Logging and tracing

Your microservices only need to handle business logic.

---

### Saga Pattern in Plain English

Problem: An order requires: reserve inventory + charge payment + create shipment. These span 3 services. If payment fails, you need to release the inventory.

Traditional solution (2PC): Lock all 3 databases, do all 3 operations, commit or rollback together. Slow, fragile, doesn't work across services.

Saga solution: Do each step independently. If a step fails, run "compensating transactions" to undo previous steps.

**Choreography** (event-based): Each service publishes events and listens to events. No central coordinator. Like a relay race — each runner passes the baton to the next.

**Orchestration** (central coordinator): One "saga orchestrator" calls each service in sequence and handles failures. Like a conductor directing an orchestra.

---

### CQRS in Plain English

Problem: Your Order Service needs to handle 10,000 writes/second AND serve complex reports. One database can't do both well.

Solution: Use two separate databases:
- **Write database** (PostgreSQL): Optimized for ACID transactions. Handles all writes.
- **Read database** (Elasticsearch or Redis): Optimized for fast reads. Denormalized for your query patterns.

When a write happens, publish an event. An event handler updates the read database.

Trade-off: The read database might be slightly behind (eventual consistency). Usually acceptable.

---

### Event Sourcing in Plain English

Traditional: Store current state. `Order: {status: SHIPPED, total: 100}`

Event Sourcing: Store every change. 
```
1. OrderCreated {items: [...], total: 100}
2. PaymentProcessed {amount: 100}
3. OrderShipped {trackingNumber: "ABC123"}
```

Current state = replay all events in order.

Benefits: Complete audit trail, can replay to any point in time, natural fit for event-driven systems.

Downside: More complex, need snapshots for long-lived aggregates.

---

### Outbox Pattern in Plain English

Problem: You need to save an order to the database AND publish an "OrderCreated" event. What if the database save succeeds but the event publish fails? Inconsistency.

Solution: Write to an "outbox" table in the SAME database transaction as the order. A separate process reads the outbox and publishes events. If the event publish fails, it retries. The database transaction guarantees atomicity.

---

> Every pattern explained with: problem, solution, diagram, trade-offs, and when to use.

---

## 1. API Gateway Pattern

### Problem
Clients need to call multiple microservices. Direct client-to-service calls create tight coupling, repeated auth logic, and CORS issues.

### Solution
Single entry point that handles: routing, authentication, rate limiting, SSL termination, request aggregation.

```
[Web Client]  [Mobile Client]  [3rd Party]
      ↓              ↓              ↓
      └──────────────┴──────────────┘
                     ↓
              [API Gateway]
              - Auth (JWT validation)
              - Rate limiting
              - Request routing
              - SSL termination
              - Logging/tracing
              - Request/response transform
                     ↓
    ┌──────────────┬──────────────┬──────────────┐
[User Service] [Order Service] [Product Service] [Payment Service]
```

### Implementations
- AWS: API Gateway + Lambda Authorizer
- Azure: Azure API Management (APIM)
- Kubernetes: Kong, nginx, Traefik, Istio Gateway
- Spring Cloud Gateway (code-based)

### Trade-offs
- Pro: single entry point, centralized cross-cutting concerns
- Con: potential bottleneck, single point of failure (mitigate with HA deployment)

### Backend for Frontend (BFF) Variant
```
[Web App] → [Web BFF] → [Services]
[Mobile]  → [Mobile BFF] → [Services]
[Partner] → [Partner BFF] → [Services]
```
Each client type gets its own gateway optimized for its needs.

---

## 2. Service Discovery Pattern

### Problem
In Kubernetes/cloud, service IPs change. How do services find each other?

### Solution
```
Client-side discovery:
  [Service A] → [Service Registry] → gets IP list → load balance → [Service B]

Server-side discovery (Kubernetes default):
  [Service A] → [DNS: service-b.namespace.svc.cluster.local]
              → [kube-proxy / iptables]
              → [Service B pod]
```

### Kubernetes Service Discovery
```yaml
# Service B is accessible at:
# http://order-service (same namespace)
# http://order-service.production (cross-namespace)
# http://order-service.production.svc.cluster.local (FQDN)
apiVersion: v1
kind: Service
metadata:
  name: order-service
  namespace: production
spec:
  selector:
    app: order-service
  ports:
  - port: 80
    targetPort: 8080
```

---

## 3. Circuit Breaker Pattern

### Problem
Service A calls Service B. Service B is slow/down. Service A threads pile up waiting → Service A crashes too (cascading failure).

### Solution
```
CLOSED state (normal):
  A → B (requests pass through)
  Count failures

OPEN state (B is failing):
  A → [Circuit Breaker] → fast fail (no call to B)
  Return fallback response
  Wait 30 seconds

HALF-OPEN state (testing recovery):
  A → [Circuit Breaker] → 1 test request → B
  Success? → CLOSED
  Failure? → OPEN again
```

### Implementation with Resilience4j
```java
@CircuitBreaker(name = "inventoryService", fallbackMethod = "getInventoryFallback")
public InventoryStatus getInventory(String productId) {
    return inventoryClient.getStatus(productId);
}

public InventoryStatus getInventoryFallback(String productId, Exception e) {
    log.warn("Inventory service unavailable, returning cached/default", e);
    return InventoryStatus.unknown(productId); // graceful degradation
}
```

### Trade-offs
- Pro: prevents cascading failures, fast failure response
- Con: complexity, need to define good fallbacks

---

## 4. Saga Pattern

### Problem
Order service needs to: reserve inventory + charge payment + create shipment. These span 3 services. If payment fails, inventory must be released. Traditional 2PC is too slow and fragile.

### Solution: Choreography (event-based)
```
[Order Service] → publishes: OrderCreated
  ↓
[Inventory Service] listens → reserves stock → publishes: InventoryReserved
  ↓
[Payment Service] listens → charges card → publishes: PaymentProcessed
  ↓
[Shipping Service] listens → creates shipment → publishes: ShipmentCreated
  ↓
[Order Service] listens → marks order: CONFIRMED

On failure:
[Payment Service] → publishes: PaymentFailed
[Inventory Service] listens → releases stock → publishes: InventoryReleased
[Order Service] listens → marks order: FAILED
```

### Solution: Orchestration (central coordinator)
```java
@Service
public class OrderSaga {
    
    public void execute(Order order) {
        try {
            // Step 1
            inventoryService.reserve(order.getItems());
            
            // Step 2
            PaymentResult payment = paymentService.charge(order.getCustomerId(), order.getTotal());
            
            // Step 3
            shippingService.createShipment(order);
            
            orderRepository.updateStatus(order.getId(), CONFIRMED);
            
        } catch (InventoryException e) {
            orderRepository.updateStatus(order.getId(), FAILED);
            // No compensation needed — inventory not reserved
            
        } catch (PaymentException e) {
            // Compensate: release inventory
            inventoryService.release(order.getItems());
            orderRepository.updateStatus(order.getId(), FAILED);
            
        } catch (ShippingException e) {
            // Compensate: refund payment + release inventory
            paymentService.refund(order.getPaymentId());
            inventoryService.release(order.getItems());
            orderRepository.updateStatus(order.getId(), FAILED);
        }
    }
}
```

### Choreography vs Orchestration
```
Choreography:
  + Loose coupling, no central coordinator
  + Easy to add new services (just subscribe to events)
  - Hard to track overall saga state
  - Circular dependencies possible
  - Hard to debug

Orchestration:
  + Clear flow, easy to understand
  + Easy to track state
  + Easy to debug
  - Central coordinator = potential bottleneck
  - Coordinator knows about all services (coupling)
```

---

## 5. CQRS (Command Query Responsibility Segregation)

### Problem
Read and write patterns are very different. Writes need ACID, reads need performance. Same model can't optimize for both.

### Solution
```
[Client]
  ↓ commands (write)        ↓ queries (read)
[Command Handler]         [Query Handler]
  ↓                           ↓
[Write Model]             [Read Model]
(normalized, ACID)        (denormalized, optimized)
[Write DB: PostgreSQL]    [Read DB: Elasticsearch/Redis]
  ↓
[Domain Events]
  ↓
[Event Handler] → updates Read DB
```

### Implementation
```java
// Command side
@Service
public class OrderCommandService {
    
    @Transactional
    public String createOrder(CreateOrderCommand command) {
        Order order = Order.create(command);
        orderRepository.save(order);
        
        // Publish event for read model update
        eventPublisher.publish(new OrderCreatedEvent(order));
        
        return order.getId();
    }
}

// Query side
@Service
public class OrderQueryService {
    
    public OrderSummary getOrderSummary(String orderId) {
        // Read from denormalized read model (fast)
        return orderReadRepository.findSummaryById(orderId);
    }
    
    public List<OrderSummary> getOrdersByCustomer(String customerId) {
        // Optimized query on read model
        return orderReadRepository.findByCustomerId(customerId);
    }
}

// Event handler updates read model
@EventListener
public class OrderReadModelUpdater {
    
    public void on(OrderCreatedEvent event) {
        OrderSummary summary = OrderSummary.from(event);
        orderReadRepository.save(summary);
    }
}
```

### When to Use CQRS
- Read/write ratio is very different (e.g., 100:1 reads)
- Read and write models need different optimization
- Complex domain with many aggregates
- Event sourcing (natural fit)

### When NOT to Use
- Simple CRUD applications
- Small teams (adds complexity)
- When eventual consistency is not acceptable

---

## 6. Event Sourcing

### Problem
Traditional: store current state. If you need history, you've lost it. Hard to audit, replay, or debug.

### Solution
Store every state change as an immutable event. Current state = replay of all events.

```
Traditional:
  Order table: {id: 1, status: SHIPPED, total: 100}

Event Sourcing:
  Event store:
    1. OrderCreated {orderId: 1, items: [...], total: 100}
    2. PaymentProcessed {orderId: 1, amount: 100}
    3. OrderShipped {orderId: 1, trackingNumber: "ABC123"}
  
  Current state = apply all events in sequence
```

### Implementation
```java
public class Order {
    private String id;
    private OrderStatus status;
    private List<OrderItem> items;
    private BigDecimal total;
    
    // Reconstruct from events
    public static Order reconstitute(List<DomainEvent> events) {
        Order order = new Order();
        events.forEach(order::apply);
        return order;
    }
    
    private void apply(DomainEvent event) {
        if (event instanceof OrderCreatedEvent e) {
            this.id = e.getOrderId();
            this.items = e.getItems();
            this.total = e.getTotal();
            this.status = OrderStatus.CREATED;
        } else if (event instanceof PaymentProcessedEvent e) {
            this.status = OrderStatus.PAID;
        } else if (event instanceof OrderShippedEvent e) {
            this.status = OrderStatus.SHIPPED;
        }
    }
}
```

### Benefits
- Complete audit trail
- Replay events to rebuild state
- Time travel: what was the state at time T?
- Event-driven integration: other services subscribe to events

### Challenges
- Event schema evolution (versioning)
- Snapshots needed for long-lived aggregates
- Eventual consistency for read models
- Complexity

---

## 7. Outbox Pattern

### Problem
Service needs to: save to DB AND publish event. If DB succeeds but event publish fails → inconsistency. If event published but DB fails → inconsistency.

### Solution
```
[Service]
  ↓ (single transaction)
[DB: save order + save event to outbox table]
  ↓ (separate process)
[Outbox Poller / CDC]
  ↓
[Message Broker: Kafka/SQS]
  ↓
[Consumers]
```

### Implementation
```java
@Transactional
public Order createOrder(CreateOrderCommand command) {
    // Save order
    Order order = orderRepository.save(Order.from(command));
    
    // Save event to outbox (same transaction)
    OutboxEvent event = OutboxEvent.builder()
        .aggregateId(order.getId())
        .eventType("OrderCreated")
        .payload(objectMapper.writeValueAsString(new OrderCreatedEvent(order)))
        .status(OutboxStatus.PENDING)
        .build();
    outboxRepository.save(event);
    
    return order;
    // Transaction commits: both order and outbox event saved atomically
}

// Separate scheduled job (or CDC via Debezium)
@Scheduled(fixedDelay = 1000)
public void processOutbox() {
    List<OutboxEvent> pending = outboxRepository.findByStatus(PENDING);
    pending.forEach(event -> {
        try {
            kafkaTemplate.send(event.getEventType(), event.getPayload());
            event.setStatus(PROCESSED);
            outboxRepository.save(event);
        } catch (Exception e) {
            log.error("Failed to publish event", e);
            // Will retry on next poll
        }
    });
}
```

---

## 8. Strangler Fig Pattern

### Problem
Monolith needs to be migrated to microservices. Big bang rewrite is too risky.

### Solution
Incrementally extract services, routing traffic through a proxy.

```
Phase 1: All traffic → Monolith
  [Client] → [Proxy] → [Monolith]

Phase 2: Extract User Service
  [Client] → [Proxy]
               ├── /api/users/* → [User Service (new)]
               └── /* → [Monolith]

Phase 3: Extract Order Service
  [Client] → [Proxy]
               ├── /api/users/* → [User Service]
               ├── /api/orders/* → [Order Service (new)]
               └── /* → [Monolith (shrinking)]

Phase 4: Monolith retired
  [Client] → [API Gateway] → [Microservices]
```

### Key Principles
- Never break existing functionality
- Extract one bounded context at a time
- Use feature flags to control traffic routing
- Maintain data consistency during transition (dual-write or CDC)

---

## 9. Sidecar Pattern

### Problem
Cross-cutting concerns (logging, tracing, mTLS, health checks) duplicated in every service.

### Solution
Deploy a sidecar container alongside the main container. Sidecar handles cross-cutting concerns.

```
Pod:
  [App Container] ←→ [Sidecar Container]
                      - Envoy (Istio): mTLS, tracing, traffic management
                      - Fluentd: log collection
                      - Vault Agent: secret injection
                      - OTel Collector: telemetry
```

### Istio Sidecar Injection
```yaml
# Namespace-level auto-injection
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    istio-injection: enabled  # all pods get Envoy sidecar
```

---

## 10. Database per Service Pattern

### Problem
Shared database creates tight coupling — schema changes in one service break others.

### Solution
Each service owns its data. No shared tables. Services communicate via APIs or events.

```
[User Service] → [User DB: PostgreSQL]
[Order Service] → [Order DB: PostgreSQL]
[Product Service] → [Product DB: MongoDB]
[Cart Service] → [Cart DB: Redis]
[Search Service] → [Search DB: Elasticsearch]
```

### Challenges and Solutions

**Challenge: Joins across services**
Solution: API composition (call multiple services, join in memory) or CQRS (maintain denormalized read model)

**Challenge: Distributed transactions**
Solution: Saga pattern

**Challenge: Data consistency**
Solution: Eventual consistency via events

**Challenge: Reporting across services**
Solution: Data warehouse / data lake that aggregates from all services

---

## 11. Shared Database Service Design (Swiss Re Context)

### Scenario
You are part of a Shared Services team. You need to design a Database-as-a-Service platform that multiple application teams can use on Azure and Alibaba Cloud.

### Architecture
```
[Application Teams]
  ↓ (self-service portal / Terraform modules)
[Database Service Platform]
  ├── [Provisioning Service]
  │   → Terraform + Azure Resource Manager / Alibaba Cloud API
  │   → Creates: Azure SQL / Cosmos DB / Alibaba RDS
  │   → Configures: networking, security, backup
  │
  ├── [Access Management Service]
  │   → Azure AD / Alibaba RAM integration
  │   → Managed Identities for app access
  │   → No shared passwords
  │
  ├── [Monitoring Service]
  │   → Azure Monitor / Alibaba Cloud Monitor
  │   → Centralized dashboards
  │   → Alerting to team channels
  │
  ├── [Backup & Recovery Service]
  │   → Automated backups
  │   → Cross-region replication
  │   → Self-service restore
  │
  └── [Cost Allocation Service]
      → Tag-based cost attribution
      → Chargeback reports per team
```

### Governance Controls
```
1. Approved database types only (Azure SQL, Cosmos DB, PostgreSQL Flexible)
2. Mandatory tags: team, environment, cost-center, data-classification
3. Private endpoints only (no public access)
4. Encryption at rest with customer-managed keys
5. Automated backup with defined retention
6. Network isolation: each DB in dedicated subnet
7. Access via Managed Identity only (no connection strings with passwords)
8. Audit logging enabled (all queries logged for sensitive data)
```

### Self-Service via Terraform Module
```hcl
module "database" {
  source = "git::https://github.com/company/terraform-modules//azure-sql"
  
  name            = "orders-db"
  environment     = "production"
  team            = "order-team"
  cost_center     = "CC-1234"
  
  sku             = "GP_Gen5_4"  # approved SKU
  storage_gb      = 100
  
  # Networking (managed by platform team)
  subnet_id       = var.db_subnet_id
  
  # Access (Managed Identity of the app)
  app_identity_id = module.app.identity_id
  
  # Backup
  backup_retention_days = 35
  geo_redundant_backup  = true
}
```

---

## 12. Microservices Interview Questions & Answers

**Q: How do you handle service-to-service authentication in microservices?**
A: "We use mutual TLS (mTLS) via Istio service mesh for service-to-service auth. Each service gets a SPIFFE identity certificate from istiod. When Service A calls Service B, both present certificates — this proves identity without passwords. For external-to-internal calls, we use JWT tokens validated at the API Gateway. No service accepts unauthenticated calls."

**Q: How do you ensure data consistency across microservices without distributed transactions?**
A: "We use the Saga pattern with the outbox pattern for reliability. For the Saga, we use orchestration for complex flows (easier to track state) and choreography for simple event chains. The outbox pattern ensures events are published atomically with the database write — we write to an outbox table in the same transaction, then a separate process publishes to Kafka. This gives us at-least-once delivery with idempotent consumers."

**Q: How do you handle the N+1 problem in microservices?**
A: "The N+1 problem in microservices is when you call Service A to get a list, then call Service B N times for each item. Solutions: 1) Batch API — Service B accepts a list of IDs and returns all at once. 2) GraphQL DataLoader — batches and deduplicates calls. 3) CQRS read model — maintain a denormalized view that has all needed data. 4) API composition at the gateway level."

**Q: How do you version microservice APIs?**
A: "URL versioning (/api/v1/orders, /api/v2/orders) for major breaking changes. Header versioning (Accept: application/vnd.company.v2+json) for minor changes. We maintain backward compatibility for at least 2 versions. Deprecation notices in response headers. Consumer-driven contract testing (Pact) to ensure API changes don't break consumers."

**Q: How do you handle a microservice that is a bottleneck?**
A: "First, identify the bottleneck: is it CPU, memory, I/O, or external dependency? Then: horizontal scaling (add more instances), async processing (move synchronous calls to queue), caching (reduce DB calls), database optimization (indexes, connection pooling), or splitting the service further if it's doing too much."
