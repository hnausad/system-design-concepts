# System Design Concepts
Here's a broader list of system design concepts, patterns, and principles organized by category — building on what you already know.

## Architectural Patterns
- **Microservices vs Monolith vs Modular Monolith**
- **Event-Driven Architecture**
- **Serverless Architecture**
- **Layered (N-tier) Architecture**
- **Hexagonal Architecture (Ports & Adapters)**
- **Service Mesh** (e.g., Istio, Linkerd)
- **Backend for Frontend (BFF)**
- **Strangler Fig Pattern** (you mentioned this — migrating legacy systems incrementally)
- **Sidecar Pattern**
- **Ambassador Pattern**

## Communication Patterns
- **Synchronous (REST, gRPC, GraphQL)** vs **Asynchronous (Message Queues, Pub/Sub)**
- **Request-Response**
- **Publish-Subscribe**
- **Message Queue** (Kafka, RabbitMQ, SQS)
- **Webhooks**
- **Long Polling / WebSockets / Server-Sent Events**

## Data Management Patterns
- **CQRS** (you mentioned)
- **Event Sourcing** (you mentioned)
- **Database per Service**
- **Shared Database (anti-pattern usually)**
- **Sharding / Partitioning**
- **Replication (Master-Slave, Master-Master)**
- **Data Denormalization**
- **Change Data Capture (CDC)**
- **Outbox Pattern** (for reliable event publishing)
- **Two-Phase Commit (2PC)**
- **Eventual Consistency vs Strong Consistency**

## Resilience & Fault Tolerance
- **Circuit Breaker** (you mentioned)
- **Retry Pattern** (you mentioned)
- **Timeout Pattern**
- **Bulkhead Pattern**
- **Fallback Pattern**
- **Rate Limiting / Throttling**
- **Dead Letter Queue (DLQ)**
- **Idempotency**
- **Graceful Degradation**
- **Chaos Engineering**

## Scalability & Performance
- **Vertical Scaling / Horizontal Scaling** (you mentioned)
- **Load Balancer** (you mentioned) — types: Round Robin, Least Connections, IP Hash
- **Caching** (Client-side, CDN, Server-side, Distributed — Redis/Memcached)
- **Cache Invalidation strategies** (write-through, write-back, write-around)
- **Content Delivery Network (CDN)**
- **Database Indexing**
- **Connection Pooling**
- **Read Replicas**
- **Auto-scaling**
- **Latency vs Throughput** (you mentioned)
- **Batching**
- **Compression**

## Discovery & Routing
- **Service Registry** (you mentioned)
- **Service Discovery** (Client-side vs Server-side)
- **API Gateway** (you mentioned)
- **DNS-based Routing**
- **Reverse Proxy**

## Distributed Systems Concepts
- **Saga Pattern** (you mentioned)
- **Consensus Algorithms** (Raft, Paxos)
- **Leader Election**
- **Distributed Locking**
- **CAP Theorem**
- **PACELC Theorem**
- **Vector Clocks**
- **Gossip Protocol**
- **Quorum-based Replication**
- **Distributed Tracing** (Jaeger, Zipkin)

## Security
- **Authentication vs Authorization**
- **OAuth2 / OpenID Connect / JWT**
- **API Keys**
- **Rate Limiting for abuse prevention**
- **mTLS (Mutual TLS)**
- **Zero Trust Architecture**
- **Encryption at Rest / in Transit**

## Observability
- **Logging (centralized)**
- **Metrics & Monitoring** (Prometheus, Grafana)
- **Distributed Tracing**
- **Health Checks**
- **Alerting**

## Data Storage Concepts
- **SQL vs NoSQL**
- **Key-Value, Document, Column-family, Graph databases**
- **ACID vs BASE**
- **Consistent Hashing**
- **Write-Ahead Log (WAL)**
- **LSM Tree / B-Tree (storage engines)**

## Design Principles
- **SOLID Principles**
- **DRY, KISS, YAGNI**
- **Domain-Driven Design (DDD)** — Bounded Context, Aggregate, Entity, Value Object
- **Single Responsibility per Service**
- **Twelve-Factor App**

## Other Useful Concepts
- **Rate Limiting Algorithms**: Token Bucket, Leaky Bucket, Fixed Window, Sliding Window
- **Multi-tenancy**
- **Feature Flags / Feature Toggles**
- **Blue-Green Deployment**
- **Canary Deployment**
- **A/B Testing infrastructure**
- **Disaster Recovery (RTO/RPO)**
- **Data Partitioning strategies** (Range, Hash, Geo-based)

---

Want me to organize these into a visual diagram (e.g., a map grouping them by category), or would you prefer a deep-dive explanation on a specific subset — like resilience patterns or data consistency patterns — with real-world examples?

