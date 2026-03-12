# Java for Cloud Solution Architects

---

## 🟢 Plain English Summary — Read This First

### What is this document about?
This document covers Java from an architect's perspective — not how to write Java code, but how Java works under the hood, what design patterns you should know, and how Java fits into cloud-native microservices.

You're not being tested as a Java developer. You're being tested on whether you understand Java's architectural implications: memory management, concurrency, design patterns, and how Spring Boot microservices work.

---

### Abbreviation Decoder

| Abbreviation | Full Name | Plain English |
|---|---|---|
| JVM | Java Virtual Machine | The runtime that executes Java code. Java compiles to bytecode, and the JVM runs that bytecode on any platform. |
| JIT | Just-In-Time Compilation | The JVM compiles frequently-used bytecode to native machine code at runtime for speed. |
| GC | Garbage Collector | Automatically frees memory that's no longer used. You don't manually free memory in Java. |
| G1GC | Garbage First Garbage Collector | The default GC in Java 9+. Balances throughput and latency. Good for most microservices. |
| ZGC | Z Garbage Collector | Ultra-low latency GC (pauses under 10ms). Good for latency-sensitive services. |
| OOM | Out of Memory | When the JVM runs out of heap space. Causes an OOMError and the JVM crashes. |
| OOMKilled | Out of Memory Killed | Kubernetes kills a pod when it exceeds its memory limit. |
| Xmx | Maximum Heap Size | JVM flag: `-Xmx512m` means max 512MB heap. Set to ~75% of container memory limit. |
| Xms | Initial Heap Size | JVM flag: `-Xms256m` means start with 256MB heap. |
| DTO | Data Transfer Object | A simple object used to transfer data between layers. No business logic. |
| POJO | Plain Old Java Object | A simple Java class with no special requirements. |
| ORM | Object-Relational Mapping | Maps Java objects to database tables. Hibernate is the most popular ORM. |
| JPA | Java Persistence API | A standard for ORM in Java. Spring Data JPA implements this. |
| JDBC | Java Database Connectivity | The low-level API for connecting Java to databases. |
| IoC | Inversion of Control | Instead of your code creating objects, a framework (Spring) creates and injects them. |
| DI | Dependency Injection | A way to provide dependencies to a class from outside rather than creating them inside. |
| AOP | Aspect-Oriented Programming | A way to add cross-cutting concerns (logging, security) without modifying business code. |
| REST | Representational State Transfer | A style for building HTTP APIs. |
| gRPC | Google Remote Procedure Call | A fast binary protocol for service-to-service calls. |
| JWT | JSON Web Token | A small digital ticket that proves identity. |
| OAuth | Open Authorization | A standard for delegated access. |
| OIDC | OpenID Connect | OAuth + identity verification. |
| MVC | Model-View-Controller | A design pattern for web applications. Spring MVC implements this. |
| SOLID | Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion | 5 principles of good object-oriented design. |
| DRY | Don't Repeat Yourself | Don't duplicate code. Extract common logic into shared methods/classes. |
| KISS | Keep It Simple, Stupid | Prefer simple solutions over complex ones. |
| YAGNI | You Aren't Gonna Need It | Don't build features you don't need yet. |
| CAS | Compare-And-Swap | An atomic operation used for lock-free thread safety. |
| TTL | Time To Live | How long something is valid before it expires. |
| HikariCP | (not an acronym) | The default connection pool in Spring Boot. Manages database connections efficiently. |
| PgBouncer | (not an acronym) | A connection pooler for PostgreSQL. Reduces the number of actual DB connections. |
| RDS Proxy | (not an acronym) | AWS's managed connection pooler for RDS databases. |
| Resilience4j | (not an acronym) | A Java library for circuit breakers, retries, rate limiters, and bulkheads. |
| Actuator | (not an acronym) | Spring Boot's built-in health check and metrics endpoints. |
| Prometheus | (not an acronym) | A metrics collection system. Spring Boot Actuator can expose metrics in Prometheus format. |
| Loom | (not an acronym) | A Java project that introduced virtual threads in Java 21. |

---

### JVM Architecture in Plain English

When you write Java code:
1. You write `.java` files
2. The compiler (`javac`) converts them to `.class` files (bytecode)
3. The JVM runs the bytecode on any platform (Windows, Linux, Mac)

The JVM has:
- **Heap**: Where objects live. The GC manages this. Set with `-Xmx`.
- **Stack**: Where method calls and local variables live. One per thread.
- **Method Area**: Where class definitions and static variables live.

**Why this matters for architects:**
- Java uses more memory than Go or Rust — factor this into container sizing
- GC pauses can cause latency spikes — choose the right GC for your SLA
- JIT compilation means Java is slow for the first few minutes (warm-up) — use provisioned concurrency or pre-warming

---

### Design Patterns in Plain English

**Singleton**: Only one instance of a class exists. Used for database connections, configuration managers. Be careful with thread safety.

**Factory**: A method that creates objects without specifying the exact class. "Give me a PaymentProcessor" — the factory decides whether to return Stripe or PayPal based on configuration.

**Builder**: Build complex objects step by step. `new DatabaseConfig.Builder().host("db.example.com").port(5432).build()`. Avoids constructors with 10 parameters.

**Adapter**: Makes incompatible interfaces work together. Like a power adapter that lets you plug a US device into a European outlet.

**Decorator**: Add behavior to an object without changing its class. Wrap a `DatabaseService` with a `CachingDatabaseService` that adds caching, then wrap that with a `LoggingDatabaseService` that adds logging.

**Circuit Breaker**: Stop calling a failing service. Fast-fail instead of waiting. (See file 11 for full explanation.)

**Strategy**: Swap algorithms at runtime. `PricingService` can use `RegularPricing`, `PremiumPricing`, or `SurgePricing` — all implement the same interface.

**Observer**: When something happens, notify all interested parties. Like subscribing to events. Used heavily in event-driven architectures.

---

### Spring Boot in Plain English

Spring Boot is the most popular Java framework for building microservices. It:
- Auto-configures everything based on what's on the classpath
- Embeds a web server (Tomcat) so you just run a JAR file
- Provides `@RestController` for HTTP endpoints
- Provides `@Service` for business logic
- Provides `@Repository` for database access
- Provides Actuator for health checks and metrics

The key annotations:
- `@SpringBootApplication` — marks the main class, enables auto-configuration
- `@RestController` — this class handles HTTP requests
- `@Service` — this class contains business logic
- `@Transactional` — wrap this method in a database transaction
- `@Autowired` — inject this dependency automatically

---

### Virtual Threads (Java 21) — Why Architects Care

Traditional Java: 1 OS thread per Java thread. Creating thousands of threads is expensive.

Virtual threads (Java 21): Millions of lightweight threads on a small number of OS threads. Blocking I/O is cheap.

**Why this matters**: Before Java 21, you needed reactive programming (WebFlux) for high-concurrency I/O-bound services. With virtual threads, you can write simple blocking code and get the same performance. Much easier to understand and debug.

---

> You are not being tested as a Java developer. You are being tested on Java architectural concepts, design patterns, and how Java fits into cloud-native systems.

---

## 1. Java Architecture Concepts (What Architects Must Know)

### JVM Architecture
```
Java Source (.java)
  ↓ javac (compiler)
Bytecode (.class)
  ↓ JVM (Java Virtual Machine)
Machine Code (platform-specific)

JVM Components:
  Class Loader: loads .class files
  Runtime Data Areas:
    - Heap: objects (GC managed)
    - Stack: method frames, local variables (per thread)
    - Method Area: class metadata, static variables
    - PC Register: current instruction per thread
  Execution Engine: interprets + JIT compiles bytecode
  GC (Garbage Collector): manages heap memory
```

### Why This Matters for Architects
- **Heap sizing**: `-Xmx` (max heap), `-Xms` (initial heap) — critical for container sizing
- **GC pauses**: can cause latency spikes — choose right GC for your SLA
- **JIT compilation**: warm-up time — affects cold start behavior
- **Memory**: Java uses more memory than Go/Rust — factor into container resource requests

### GC Algorithms (Architect Level)
```
Serial GC: single-threaded, small apps
Parallel GC: multi-threaded, throughput-focused
G1 GC (default Java 9+): balanced, predictable pauses
ZGC (Java 15+): ultra-low latency (<10ms pauses), large heaps
Shenandoah: low-pause, concurrent
```

For microservices: G1GC or ZGC for latency-sensitive services.

---

## 2. Java Concurrency (Architect Must Know)

### Thread Safety Problems
```java
// NOT thread-safe — race condition
public class Counter {
    private int count = 0;
    public void increment() { count++; } // read-modify-write, not atomic
}

// Thread-safe option 1: synchronized
public synchronized void increment() { count++; }

// Thread-safe option 2: AtomicInteger (better performance)
private AtomicInteger count = new AtomicInteger(0);
public void increment() { count.incrementAndGet(); }

// Thread-safe option 3: volatile (only for visibility, not atomicity)
private volatile boolean running = true;
```

### Java Concurrency Tools
```java
// ExecutorService: thread pool management
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(() -> processRequest(request));

// CompletableFuture: async, non-blocking
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> callExternalService())
    .thenApply(result -> transform(result))
    .exceptionally(ex -> handleError(ex));

// CountDownLatch: wait for N threads to complete
CountDownLatch latch = new CountDownLatch(3);
// each thread calls latch.countDown() when done
latch.await(); // main thread waits

// Semaphore: limit concurrent access
Semaphore semaphore = new Semaphore(10); // max 10 concurrent
semaphore.acquire();
try { doWork(); } finally { semaphore.release(); }
```

### Virtual Threads (Java 21 — Project Loom)
```java
// Traditional: 1 OS thread per Java thread (expensive)
// Virtual threads: millions of lightweight threads on few OS threads

// Create virtual thread
Thread.ofVirtual().start(() -> handleRequest(request));

// Virtual thread executor
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

**Architect significance**: Virtual threads make blocking I/O cheap — no need for reactive programming for I/O-bound microservices. Huge for cloud-native Java.

---

## 3. Design Patterns in Java (Critical for Interview)

### Creational Patterns

#### Singleton
```java
// Thread-safe Singleton (double-checked locking)
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    
    private DatabaseConnection() {}
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}

// Better: Enum Singleton (thread-safe, serialization-safe)
public enum DatabaseConnection {
    INSTANCE;
    public void connect() { ... }
}
```

#### Factory Pattern
```java
// Factory Method
public interface PaymentProcessor {
    void process(Payment payment);
}

public class StripeProcessor implements PaymentProcessor { ... }
public class PayPalProcessor implements PaymentProcessor { ... }

public class PaymentProcessorFactory {
    public static PaymentProcessor create(String type) {
        return switch (type) {
            case "stripe" -> new StripeProcessor();
            case "paypal" -> new PayPalProcessor();
            default -> throw new IllegalArgumentException("Unknown: " + type);
        };
    }
}
```

#### Builder Pattern
```java
// Immutable object with many optional fields
public class DatabaseConfig {
    private final String host;
    private final int port;
    private final String database;
    private final int maxConnections;
    private final int connectionTimeout;
    
    private DatabaseConfig(Builder builder) {
        this.host = builder.host;
        this.port = builder.port;
        this.database = builder.database;
        this.maxConnections = builder.maxConnections;
        this.connectionTimeout = builder.connectionTimeout;
    }
    
    public static class Builder {
        private String host = "localhost";
        private int port = 5432;
        private String database;
        private int maxConnections = 10;
        private int connectionTimeout = 30;
        
        public Builder host(String host) { this.host = host; return this; }
        public Builder port(int port) { this.port = port; return this; }
        public Builder database(String database) { this.database = database; return this; }
        public Builder maxConnections(int max) { this.maxConnections = max; return this; }
        
        public DatabaseConfig build() {
            Objects.requireNonNull(database, "database required");
            return new DatabaseConfig(this);
        }
    }
}

// Usage
DatabaseConfig config = new DatabaseConfig.Builder()
    .host("prod-db.example.com")
    .database("orders")
    .maxConnections(50)
    .build();
```

### Structural Patterns

#### Adapter Pattern
```java
// Adapt legacy payment system to new interface
public interface ModernPaymentGateway {
    PaymentResult charge(String customerId, BigDecimal amount, String currency);
}

public class LegacyPaymentSystem {
    public boolean processPayment(int customerId, double amount) { ... }
}

public class LegacyPaymentAdapter implements ModernPaymentGateway {
    private final LegacyPaymentSystem legacy;
    
    public LegacyPaymentAdapter(LegacyPaymentSystem legacy) {
        this.legacy = legacy;
    }
    
    @Override
    public PaymentResult charge(String customerId, BigDecimal amount, String currency) {
        boolean success = legacy.processPayment(
            Integer.parseInt(customerId), 
            amount.doubleValue()
        );
        return new PaymentResult(success ? "SUCCESS" : "FAILED");
    }
}
```

#### Decorator Pattern
```java
// Add behavior without modifying original class
public interface DataService {
    String getData(String key);
}

public class DatabaseDataService implements DataService {
    public String getData(String key) { /* DB call */ }
}

// Add caching
public class CachingDataService implements DataService {
    private final DataService delegate;
    private final Cache cache;
    
    public CachingDataService(DataService delegate, Cache cache) {
        this.delegate = delegate;
        this.cache = cache;
    }
    
    public String getData(String key) {
        return cache.getOrCompute(key, () -> delegate.getData(key));
    }
}

// Add logging
public class LoggingDataService implements DataService {
    private final DataService delegate;
    
    public String getData(String key) {
        log.info("Getting data for key: {}", key);
        String result = delegate.getData(key);
        log.info("Got data for key: {}", key);
        return result;
    }
}

// Compose: DB + Cache + Logging
DataService service = new LoggingDataService(
    new CachingDataService(
        new DatabaseDataService(), 
        cache
    )
);
```

#### Proxy Pattern
```java
// Circuit Breaker as Proxy
public class CircuitBreakerProxy implements PaymentService {
    private final PaymentService delegate;
    private int failureCount = 0;
    private State state = State.CLOSED;
    private long lastFailureTime;
    
    public PaymentResult process(Payment payment) {
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - lastFailureTime > 30_000) {
                state = State.HALF_OPEN;
            } else {
                throw new CircuitOpenException("Circuit breaker open");
            }
        }
        
        try {
            PaymentResult result = delegate.process(payment);
            onSuccess();
            return result;
        } catch (Exception e) {
            onFailure();
            throw e;
        }
    }
    
    private void onSuccess() {
        failureCount = 0;
        state = State.CLOSED;
    }
    
    private void onFailure() {
        failureCount++;
        lastFailureTime = System.currentTimeMillis();
        if (failureCount >= 5) state = State.OPEN;
    }
}
```

### Behavioral Patterns

#### Strategy Pattern
```java
// Swap algorithms at runtime
public interface PricingStrategy {
    BigDecimal calculatePrice(Order order);
}

public class RegularPricing implements PricingStrategy {
    public BigDecimal calculatePrice(Order order) {
        return order.getBasePrice();
    }
}

public class PremiumPricing implements PricingStrategy {
    public BigDecimal calculatePrice(Order order) {
        return order.getBasePrice().multiply(BigDecimal.valueOf(0.9)); // 10% discount
    }
}

public class SurgePricing implements PricingStrategy {
    private final double multiplier;
    public BigDecimal calculatePrice(Order order) {
        return order.getBasePrice().multiply(BigDecimal.valueOf(multiplier));
    }
}

public class PricingService {
    private PricingStrategy strategy;
    
    public void setStrategy(PricingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public BigDecimal getPrice(Order order) {
        return strategy.calculatePrice(order);
    }
}
```

#### Observer Pattern
```java
// Event-driven within application
public interface OrderEventListener {
    void onOrderCreated(Order order);
}

public class OrderService {
    private final List<OrderEventListener> listeners = new ArrayList<>();
    
    public void addListener(OrderEventListener listener) {
        listeners.add(listener);
    }
    
    public Order createOrder(OrderRequest request) {
        Order order = processOrder(request);
        listeners.forEach(l -> l.onOrderCreated(order));
        return order;
    }
}

// Listeners
public class InventoryService implements OrderEventListener {
    public void onOrderCreated(Order order) { reserveInventory(order); }
}

public class NotificationService implements OrderEventListener {
    public void onOrderCreated(Order order) { sendConfirmationEmail(order); }
}
```

#### Command Pattern
```java
// Encapsulate requests as objects (useful for undo, queuing)
public interface Command {
    void execute();
    void undo();
}

public class TransferMoneyCommand implements Command {
    private final Account from;
    private final Account to;
    private final BigDecimal amount;
    
    public void execute() {
        from.debit(amount);
        to.credit(amount);
    }
    
    public void undo() {
        to.debit(amount);
        from.credit(amount);
    }
}
```

---

## 4. Spring Boot for Microservices (Architect Level)

### Spring Boot Architecture
```
Spring Boot Application
  ├── @SpringBootApplication (auto-configuration)
  ├── Embedded Server (Tomcat/Netty)
  ├── Spring MVC / WebFlux (HTTP layer)
  ├── Spring Data (database access)
  ├── Spring Security (auth/authz)
  ├── Spring Cloud (microservices patterns)
  └── Actuator (health, metrics, info)
```

### Key Annotations
```java
@RestController     // HTTP controller, returns JSON
@Service            // Business logic layer
@Repository         // Data access layer
@Component          // Generic Spring bean
@Configuration      // Configuration class
@Bean               // Declare a Spring bean
@Autowired          // Dependency injection
@Value              // Inject property value
@Transactional      // Database transaction boundary
@Async              // Run method in thread pool
@Scheduled          // Cron/fixed-rate scheduling
@EventListener      // Listen to application events
```

### Spring Boot Microservice Structure
```
src/main/java/com/company/orderservice/
  ├── OrderServiceApplication.java    (@SpringBootApplication)
  ├── controller/
  │   └── OrderController.java        (@RestController)
  ├── service/
  │   └── OrderService.java           (@Service)
  ├── repository/
  │   └── OrderRepository.java        (JpaRepository)
  ├── model/
  │   ├── Order.java                  (@Entity)
  │   └── OrderRequest.java           (DTO)
  ├── event/
  │   └── OrderCreatedEvent.java
  └── config/
      └── DatabaseConfig.java         (@Configuration)
```

### Health Check & Actuator
```yaml
# application.yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
```

```java
// Custom health indicator
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        try {
            // check DB connection
            return Health.up()
                .withDetail("database", "PostgreSQL")
                .withDetail("status", "connected")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

### Resilience4j (Circuit Breaker in Spring Boot)
```java
@Service
public class PaymentService {
    
    @CircuitBreaker(name = "payment", fallbackMethod = "paymentFallback")
    @Retry(name = "payment")
    @TimeLimiter(name = "payment")
    public CompletableFuture<PaymentResult> processPayment(Payment payment) {
        return CompletableFuture.supplyAsync(() -> 
            externalPaymentGateway.charge(payment)
        );
    }
    
    public CompletableFuture<PaymentResult> paymentFallback(Payment payment, Exception e) {
        log.error("Payment service unavailable, using fallback", e);
        return CompletableFuture.completedFuture(
            PaymentResult.pending("Payment queued for retry")
        );
    }
}
```

```yaml
# application.yaml
resilience4j:
  circuitbreaker:
    instances:
      payment:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        permittedNumberOfCallsInHalfOpenState: 3
  retry:
    instances:
      payment:
        maxAttempts: 3
        waitDuration: 1s
        exponentialBackoffMultiplier: 2
```

---

## 5. Java Programs for Architects (Common Interview Programs)

### Program 1: Thread-Safe Singleton with Lazy Loading
```java
public class ConfigurationManager {
    private static volatile ConfigurationManager instance;
    private final Map<String, String> config;
    
    private ConfigurationManager() {
        config = loadFromEnvironment();
    }
    
    public static ConfigurationManager getInstance() {
        if (instance == null) {
            synchronized (ConfigurationManager.class) {
                if (instance == null) {
                    instance = new ConfigurationManager();
                }
            }
        }
        return instance;
    }
    
    public String get(String key) {
        return config.getOrDefault(key, "");
    }
    
    private Map<String, String> loadFromEnvironment() {
        return System.getenv().entrySet().stream()
            .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
    }
}
```

### Program 2: Producer-Consumer with BlockingQueue
```java
public class MessageProcessor {
    private final BlockingQueue<Message> queue = new LinkedBlockingQueue<>(1000);
    private final ExecutorService producers = Executors.newFixedThreadPool(5);
    private final ExecutorService consumers = Executors.newFixedThreadPool(10);
    
    public void start() {
        // Producers
        for (int i = 0; i < 5; i++) {
            producers.submit(this::produce);
        }
        // Consumers
        for (int i = 0; i < 10; i++) {
            consumers.submit(this::consume);
        }
    }
    
    private void produce() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Message msg = fetchFromExternalSource();
                queue.put(msg); // blocks if queue full
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    private void consume() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Message msg = queue.poll(1, TimeUnit.SECONDS); // blocks if empty
                if (msg != null) processMessage(msg);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

### Program 3: Retry with Exponential Backoff
```java
public class RetryExecutor {
    
    public <T> T executeWithRetry(Supplier<T> operation, int maxAttempts) {
        int attempt = 0;
        long waitMs = 1000;
        
        while (attempt < maxAttempts) {
            try {
                return operation.get();
            } catch (TransientException e) {
                attempt++;
                if (attempt >= maxAttempts) throw e;
                
                long jitter = (long) (Math.random() * 500);
                long sleepMs = waitMs + jitter;
                
                log.warn("Attempt {} failed, retrying in {}ms", attempt, sleepMs);
                
                try {
                    Thread.sleep(sleepMs);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException("Interrupted during retry", ie);
                }
                
                waitMs = Math.min(waitMs * 2, 30_000); // cap at 30s
            }
        }
        throw new RuntimeException("Should not reach here");
    }
}

// Usage
String result = retryExecutor.executeWithRetry(
    () -> externalService.call(),
    5
);
```

### Program 4: Rate Limiter (Token Bucket)
```java
public class RateLimiter {
    private final int maxTokens;
    private final long refillIntervalMs;
    private int tokens;
    private long lastRefillTime;
    
    public RateLimiter(int maxTokens, long refillIntervalMs) {
        this.maxTokens = maxTokens;
        this.refillIntervalMs = refillIntervalMs;
        this.tokens = maxTokens;
        this.lastRefillTime = System.currentTimeMillis();
    }
    
    public synchronized boolean tryAcquire() {
        refill();
        if (tokens > 0) {
            tokens--;
            return true;
        }
        return false;
    }
    
    private void refill() {
        long now = System.currentTimeMillis();
        long elapsed = now - lastRefillTime;
        int tokensToAdd = (int) (elapsed / refillIntervalMs) * maxTokens;
        if (tokensToAdd > 0) {
            tokens = Math.min(maxTokens, tokens + tokensToAdd);
            lastRefillTime = now;
        }
    }
}
```

### Program 5: LRU Cache
```java
public class LRUCache<K, V> {
    private final int capacity;
    private final LinkedHashMap<K, V> cache;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.cache = new LinkedHashMap<>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > capacity;
            }
        };
    }
    
    public synchronized V get(K key) {
        return cache.getOrDefault(key, null);
    }
    
    public synchronized void put(K key, V value) {
        cache.put(key, value);
    }
    
    public synchronized int size() {
        return cache.size();
    }
}
```

### Program 6: Async Service Call with CompletableFuture
```java
@Service
public class OrderAggregatorService {
    
    public OrderDetails getOrderDetails(String orderId) {
        // Call 3 services in parallel
        CompletableFuture<Order> orderFuture = 
            CompletableFuture.supplyAsync(() -> orderService.getOrder(orderId));
        
        CompletableFuture<Customer> customerFuture = 
            orderFuture.thenCompose(order -> 
                CompletableFuture.supplyAsync(() -> 
                    customerService.getCustomer(order.getCustomerId())));
        
        CompletableFuture<List<Product>> productsFuture = 
            orderFuture.thenCompose(order -> 
                CompletableFuture.supplyAsync(() -> 
                    productService.getProducts(order.getProductIds())));
        
        // Wait for all and combine
        return CompletableFuture.allOf(customerFuture, productsFuture)
            .thenApply(v -> {
                Order order = orderFuture.join();
                Customer customer = customerFuture.join();
                List<Product> products = productsFuture.join();
                return new OrderDetails(order, customer, products);
            })
            .exceptionally(ex -> {
                log.error("Failed to get order details", ex);
                throw new OrderServiceException("Failed to aggregate order", ex);
            })
            .join();
    }
}
```

---

## 6. Java Interview Questions for Cloud Architects

**Q: What is the difference between HashMap and ConcurrentHashMap?**
A: HashMap is not thread-safe — concurrent modifications cause ConcurrentModificationException or data corruption. ConcurrentHashMap uses segment-level locking (Java 7) or CAS operations (Java 8+) for thread safety without locking the entire map. For cloud microservices, use ConcurrentHashMap for shared caches, or better yet, use Redis for distributed caching.

**Q: Explain Java memory model and why it matters for microservices.**
A: The JVM heap is divided into Young Generation (Eden + Survivor spaces) and Old Generation. Objects start in Eden, survive GC to Survivor, then promote to Old Gen. For containers, set `-Xmx` to ~75% of container memory limit. GC pauses can cause latency spikes — use G1GC or ZGC for latency-sensitive services. Monitor GC metrics in Prometheus.

**Q: What is the difference between @Transactional and distributed transactions?**
A: @Transactional handles transactions within a single database — ACID guaranteed. Distributed transactions across multiple databases/services require 2PC (two-phase commit) which is slow and fragile, or the Saga pattern which uses compensating transactions. In microservices, we avoid distributed transactions and use Saga instead.

**Q: How do you handle connection pooling in Spring Boot microservices?**
A: Spring Boot uses HikariCP by default. Key settings: `maximum-pool-size` (default 10, set based on DB max connections / number of instances), `minimum-idle`, `connection-timeout` (30s default), `idle-timeout`. For Kubernetes, calculate: DB max connections = pod replicas × pool size. Use RDS Proxy (AWS) or Azure SQL connection pooling to handle connection limits.

**Q: What is reactive programming and when would you use it?**
A: Reactive programming (Spring WebFlux, Project Reactor) uses non-blocking I/O — a small thread pool handles many concurrent requests by not blocking on I/O. Use it when: high concurrency with I/O-bound operations, streaming data, real-time systems. Don't use it when: CPU-bound work, team unfamiliar with reactive, simple CRUD. With Java 21 virtual threads, traditional blocking code can achieve similar throughput without reactive complexity.
