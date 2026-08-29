# Lesson 3 — Core System Design Building Blocks

Great. Now we have learned:

```text
Lesson 1 → Requirements
Lesson 2 → Capacity Estimation
```

Now we can learn the **building blocks** that turn those requirements and numbers into an architecture.

The goal is **not** to memorize components.

For every component, learn:

> **What problem does it solve? → When do I need it? → What happens if it fails? → What are the trade-offs?**

---

# 1. The Big Picture

A typical large-scale system may look like:

```text
                         USERS
                           |
                           ↓
                     ┌──────────┐
                     │   CDN    │
                     └────┬─────┘
                          |
                          ↓
                 ┌────────────────┐
                 │ Load Balancer  │
                 └───────┬────────┘
                         |
                         ↓
                 ┌────────────────┐
                 │  API Gateway   │
                 └───────┬────────┘
                         |
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
     User Service   Order Service  Product Service
          |              |              |
          ↓              ↓              ↓
       Cache           Cache          Cache
          |              |              |
          └──────────────┼──────────────┘
                         ↓
                     Database
                         |
                  ┌──────┴──────┐
                  ↓             ↓
             Read Replica   Read Replica
                         
                         +
                         
                  Message Queue
                         |
              ┌──────────┼──────────┐
              ↓          ↓          ↓
          Payment    Inventory   Notification
```

We will break this down one component at a time.

---

# 2. Load Balancer

## Problem

Suppose you have one application server:

```text
              Users
             / | \
            /  |  \
           ↓   ↓   ↓
       ┌─────────────┐
       │ Application │
       │   Server    │
       └─────────────┘
```

What happens when:

* 10× traffic arrives?
* Server crashes?
* Server becomes overloaded?

You need multiple servers.

```text
                 Users
                   |
                   ↓
             Load Balancer
              /    |    \
             ↓     ↓     ↓
           App1   App2   App3
```

The load balancer distributes requests.

### Why?

```text
Scalability
High availability
Fault tolerance
```

---

# 3. Load Balancing Algorithms

### Round Robin

```text
Request 1 → App1
Request 2 → App2
Request 3 → App3
Request 4 → App1
```

Simple.

---

### Least Connections

Send traffic to the server handling the fewest connections.

Useful when request processing times vary.

---

### Weighted

```text
App1 → 50%
App2 → 30%
App3 → 20%
```

Useful when servers have different capacities.

---

### IP Hash / Consistent Hashing

Same client can be directed toward the same server.

Useful for certain stateful workloads, although in modern architectures it's usually better to avoid relying on server-local session state.

---

# 4. Health Checks

This is critical.

Suppose:

```text
App1 → healthy
App2 → DOWN
App3 → healthy
```

Load balancer should stop sending requests to App2.

```text
                LB
             /  |  \
            ↓   X   ↓
          App1 App2 App3
```

This is one way we handle a **server failure**.

---

# 5. API Gateway

Now imagine you have many services:

```text
User Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Should clients directly communicate with all of them?

```text
Mobile
 ├── User Service
 ├── Order Service
 ├── Payment Service
 ├── Inventory Service
 └── Notification Service
```

This becomes difficult to manage.

Instead:

```text
Client
   |
   ↓
API Gateway
   |
   ├── User Service
   ├── Order Service
   ├── Payment Service
   └── Inventory Service
```

The gateway can handle common concerns such as:

* Authentication
* Authorization
* Routing
* Rate limiting
* Request validation
* API versioning
* Observability

---

# 6. API Gateway vs Load Balancer

This is a **very common interview question**.

| Load Balancer                          | API Gateway           |
| -------------------------------------- | --------------------- |
| Distributes traffic                    | Routes APIs           |
| Primarily infrastructure/network layer | Application/API layer |
| Health checks                          | Authentication        |
| Load distribution                      | Rate limiting         |
| Failover                               | API policies          |
| Server selection                       | API versioning        |

They can coexist:

```text
Client
  ↓
Load Balancer
  ↓
API Gateway
  ↓
Services
```

Don't say:

> "API Gateway replaces Load Balancer."

They solve different problems.

---

# 7. Reverse Proxy

A reverse proxy sits between clients and backend servers.

```text
Client
   ↓
Reverse Proxy
   ↓
Backend
```

It can provide:

* TLS termination
* Routing
* Compression
* Caching
* Security controls

Popular technologies include NGINX and Envoy.

In interviews, understand the **role**, not just the product name.

---

# 8. CDN

CDN = Content Delivery Network.

Problem:

Suppose your server is in India.

A user in the US requests an image.

```text
US User
   |
   | thousands of km
   ↓
India Server
```

Latency can be significant.

Instead:

```text
              Origin
             Server
                |
          ┌─────┴─────┐
          ↓           ↓
       CDN US      CDN India
          ↑
          |
       US User
```

Static content is cached closer to users.

Great for:

```text
Images
Videos
CSS
JavaScript
Downloads
Static files
```

---

# 9. Cache

The cache stores frequently accessed data closer to the application.

Without cache:

```text
Client
  ↓
Application
  ↓
Database
```

With cache:

```text
Client
  ↓
Application
  ↓
Redis
  ↓ cache miss
Database
```

If data is in Redis:

```text
Client
  ↓
Application
  ↓
Redis → Response
```

No database request.

---

# 10. Why Cache?

Suppose:

```text
Database capacity = 5K QPS
Application traffic = 50K QPS
```

You have a problem.

But if:

```text
Cache hit ratio = 90%
```

then only approximately:

```text
50K × 10%
= 5K QPS
```

reaches the database.

That's a huge difference.

---

# 11. Cache Hit Ratio

Important metric:

```text
Cache Hit Ratio =
Cache Hits / Total Requests
```

Example:

```text
100 requests

90 → Cache
10 → Database
```

Hit ratio:

```text
90 / 100
= 90%
```

Higher isn't automatically better—you still need correct data and sensible eviction policies.

---

# 12. Cache-Aside Pattern

One of the most important patterns.

```text
Application
     |
     ↓
   Cache
     |
     | miss
     ↓
 Database
     |
     ↓
Application
     |
     ↓
Update Cache
```

Typical flow:

```java
Product product = cache.get(productId);

if (product == null) {
    product = database.find(productId);
    cache.put(productId, product);
}

return product;
```

Conceptually simple, but production systems need to consider:

* TTL
* Cache invalidation
* Stampede
* Stale data
* Cache failure

---

# 13. Cache Failure Modes

You should connect this with our previous failure-mode lesson.

### Cache Stampede

Many requests miss simultaneously:

```text
             Cache
               X
               |
     ┌─────────┼─────────┐
     ↓         ↓         ↓
    Req       Req       Req
     \         |         /
              DB
```

Potential database overload.

---

### Hot Key

One key receives huge traffic:

```text
product:123
     ↑
     │
1M requests/sec
```

That key becomes a bottleneck.

---

### Cache Failure

Redis goes down:

```text
Application
     |
  Redis X
     |
     ↓
Database
```

If all traffic suddenly falls to the DB, you can get a cascading failure.

Therefore:

> **Every cache design needs a cache-failure strategy.**

---

# 14. Database

The database is the persistent source of truth for many systems.

Two major categories:

```text
Database
│
├── Relational
│
└── NoSQL
```

---

# 15. Relational Database

Examples:

```text
PostgreSQL
MySQL
Oracle
SQL Server
```

Good when you need:

* Transactions
* Relationships
* Strong consistency
* Constraints
* Complex queries

Example:

```text
Order
   |
   ├── OrderItem
   |
   └── Payment
```

---

# 16. NoSQL

Common categories include:

```text
Key-Value
Document
Wide-column
Graph
```

Examples include:

```text
DynamoDB
Cassandra
MongoDB
```

Often useful when you need:

* Very large scale
* High throughput
* Flexible schema
* Specific access patterns

But don't say:

> "NoSQL is faster than SQL."

That's far too simplistic.

The correct question is:

> **"What data model and access pattern does the system require?"**

---

# 17. Primary + Read Replicas

Suppose database receives:

```text
90K reads/sec
10K writes/sec
```

One DB may become a bottleneck.

We can use:

```text
             Application
             /          \
        Writes           Reads
           ↓               ↓
        Primary      Read Replicas
                       /       \
                      ↓         ↓
                   Replica1   Replica2
```

Writes:

```text
Application → Primary
```

Reads:

```text
Application → Replica
```

---

# 18. Replication Lag

Important!

Suppose:

```text
Primary
price = ₹100
```

Update:

```text
price = ₹120
```

But replica hasn't received the update yet.

```text
Primary  → ₹120
Replica  → ₹100
```

User might see stale data.

This creates a consistency decision:

> Is stale data acceptable?

If yes:

```text
Read Replica
```

If no:

```text
Read Primary
```

for that critical read.

---

# 19. Database Sharding

Eventually replication isn't enough.

Suppose:

```text
1 Billion users
```

One database becomes difficult to scale.

Partition data:

```text
             Users
                |
       ┌────────┼────────┐
       ↓        ↓        ↓
    Shard 1  Shard 2  Shard 3
    A-H      I-P      Q-Z
```

This is **sharding**.

---

# 20. Choosing a Shard Key

This is a major senior interview topic.

Suppose:

```text
Shard by userId
```

You need to ask:

> Is userId evenly distributed?

And:

> Will most requests know userId?

A bad shard key can create:

```text
Hot partition
Uneven distribution
Difficult queries
Cross-shard operations
```

---

# 21. Message Queue

Now consider an order.

Should the user wait for:

```text
Order
 ↓
Payment
 ↓
Inventory
 ↓
Email
 ↓
Analytics
 ↓
Response
```

Maybe not.

We can make some work asynchronous:

```text
              Order Service
                   |
                   ↓
                 Queue
          ┌────────┼────────┐
          ↓        ↓        ↓
       Payment  Inventory  Email
```

Benefits:

* Decoupling
* Asynchronous processing
* Traffic smoothing
* Retry
* Independent scaling

---

# 22. Kafka / Message Queue

Don't simply say:

> "We'll use Kafka."

Explain the requirement.

For example:

> "Order creation should not wait for notification processing, so I'll publish an order event and let notification consumers process it asynchronously."

Now Kafka/message queue has a reason to exist.

---

# 23. Queue Failure Modes

Remember our previous lesson.

### Queue backlog

```text
Producer
100K msg/sec
     ↓
   Queue
     ↓
Consumer
20K msg/sec
```

Backlog grows.

### Duplicate message

Consumer may process:

```text
OrderCreated
```

twice.

Therefore:

> Consumers should be **idempotent**.

### Poison message

One bad message repeatedly fails.

Use:

```text
Retry
   ↓
Retry
   ↓
DLQ
```

---

# 24. Object Storage

For large files, don't generally put the actual file inside a relational database.

Use object storage:

```text
User
 ↓
Upload
 ↓
Object Storage
 ↓
CDN
 ↓
Other Users
```

Useful for:

```text
Images
Videos
PDFs
Documents
Backups
Large files
```

The database stores metadata:

```text
File
 ├── fileId
 ├── userId
 ├── filename
 └── objectLocation
```

---

# 25. Search Engine

Database queries aren't always ideal for:

```text
Full-text search
Ranking
Fuzzy search
Faceted filtering
```

A search engine can be introduced:

```text
                Product DB
                    |
                    ↓
              Search Index
                    |
                    ↓
                 Search
```

Example:

```text
"red running shoes"
```

Search engine handles:

```text
Text matching
Ranking
Filtering
Facets
```

---

# 26. Rate Limiter

Suppose one client sends:

```text
1 million requests/sec
```

It could overload your service.

Rate limiting:

```text
Client
  |
  ↓
Rate Limiter
  |
  ├── Allowed → Application
  |
  └── Rejected → 429
```

Common algorithms:

* Token Bucket
* Leaky Bucket
* Fixed Window
* Sliding Window

We'll study these later.

---

# 27. Service Discovery

In dynamic environments, services may constantly change.

```text
Order Service
   ↓
Where is Payment Service?
```

Service discovery answers:

```text
Payment Service
    ↓
10.0.1.15:8080
10.0.1.16:8080
10.0.1.17:8080
```

Modern platforms often provide service discovery through orchestration/platform networking.

---

# 28. Observability

A production system needs visibility.

```text
Application
    |
    ├── Logs
    ├── Metrics
    └── Traces
```

### Logs

Tell you:

> What happened?

### Metrics

Tell you:

> How much/how often?

Examples:

```text
QPS
Error rate
CPU
Memory
Latency
Queue depth
```

### Traces

Tell you:

> Where did the request spend its time?

Example:

```text
Request
  |
  ├── API Gateway     5ms
  ├── Order Service  20ms
  ├── Redis           2ms
  └── Database       40ms
```

Total ≈ 67ms.

---

# 29. Putting Everything Together

Let's construct a realistic e-commerce architecture.

```text
                         USERS
                           |
                           ↓
                         CDN
                           |
                           ↓
                    Load Balancer
                           |
                           ↓
                     API Gateway
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
      Product           Order             User
      Service           Service           Service
          |                |
          ↓                ↓
       Redis            Redis
          |                |
          ↓                ↓
    Product DB        Order DB
                           |
                           ↓
                     Message Queue
                           |
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Payment       Inventory    Notification
          Service       Service        Service
             |             |
             ↓             ↓
        Payment DB     Inventory DB
```

And:

```text
                    Observability
                   /      |       \
                Logs    Metrics   Traces
```

---

# 30. How a Senior Engineer Explains This

Don't say:

> "We'll use Redis, Kafka, MongoDB and Kubernetes."

Instead:

### Load Balancer

> "I'll use multiple application instances behind a load balancer so that traffic can be distributed and an unhealthy instance can be removed."

### Redis

> "Product reads are high-volume and mostly read-only, so I'll cache frequently accessed product data to reduce database load."

### Read replicas

> "If product reads exceed the primary's capacity, I'll add read replicas."

### Queue

> "Notifications don't need to block order creation, so I'll publish an event and process notifications asynchronously."

### Database

> "Orders require transactional correctness, so I'd initially prefer a relational database."

### Sharding

> "If order volume eventually exceeds the capacity of a single database cluster, I'd consider partitioning based on an access-friendly key."

🔥 **This is the level of explanation you should aim for.**

---

# 31. The Most Important Interview Question

For every component, ask yourself:

> **Why is this here?**

Example:

```text
Redis
↓
Why?
↓
Reduce DB reads
↓
What if Redis fails?
↓
Fallback + DB protection
```

Another:

```text
Kafka
↓
Why?
↓
Async processing + decoupling
↓
What if Kafka fails?
↓
Retry / durability / producer handling
```

Another:

```text
Read Replica
↓
Why?
↓
Scale reads
↓
What problem?
↓
Replication lag
```

This creates a chain of reasoning.

---

# 32. Your Core Building Blocks Cheat Sheet

| Component           | Main Problem Solved                     |
| ------------------- | --------------------------------------- |
| Load Balancer       | Distribute traffic                      |
| API Gateway         | API routing + cross-cutting concerns    |
| Reverse Proxy       | Proxy/routing/TLS/caching               |
| CDN                 | Reduce latency for static content       |
| Redis/Cache         | Reduce database load/latency            |
| SQL DB              | Transactions + relationships            |
| NoSQL               | Large-scale specialized access patterns |
| Read Replica        | Scale database reads                    |
| Sharding            | Scale database horizontally             |
| Message Queue       | Async processing + decoupling           |
| Object Storage      | Large files                             |
| Search Engine       | Full-text search                        |
| Rate Limiter        | Protect services                        |
| Service Discovery   | Locate dynamic services                 |
| Logs                | Debug events                            |
| Metrics             | Measure system health                   |
| Distributed Tracing | Trace requests across services          |

---

# 33. One Critical Rule for Your Interviews

Don't start every design with:

```text
Microservices
Kafka
Redis
Kubernetes
MongoDB
```

Start with:

```text
Requirements
      ↓
Capacity
      ↓
Bottleneck
      ↓
Component
      ↓
Trade-off
```

For example:

```text
10M DAU
   ↓
100K peak reads/sec
   ↓
Database can't handle all reads
   ↓
Cache + read replicas
```

That's **system design**.

Not:

```text
"We need Redis because Redis is fast."
```

---

# Lesson 3 — What You Should Master

Before moving on, make sure you can explain these without memorizing definitions:

```text
✓ Load Balancer
✓ API Gateway
✓ Reverse Proxy
✓ CDN
✓ Cache
✓ Cache-Aside
✓ SQL
✓ NoSQL
✓ Primary / Replica
✓ Replication Lag
✓ Sharding
✓ Message Queue
✓ Kafka
✓ Object Storage
✓ Search Engine
✓ Rate Limiter
✓ Service Discovery
✓ Logs / Metrics / Traces
```

And for each one:

```text
What problem?
      ↓
Why use it?
      ↓
When NOT to use it?
      ↓
Failure mode?
      ↓
Scaling problem?
      ↓
Trade-off?
```

---

## Next: Lesson 4 — Database Deep Dive

This is where we go deeper into one of the **most important areas for senior interviews**:

**SQL vs NoSQL → indexing → transactions → isolation levels → replication → read replicas → sharding → partition keys → hot partitions → consistency → database failure modes.**

We'll also connect the concepts to **Java/Spring Boot**, including how these decisions appear in real applications with JPA/Hibernate, transactions, connection pools, and multiple datasources.
