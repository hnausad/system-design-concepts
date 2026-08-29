# Lesson 1 — Requirements Gathering

This is **the most important first step in System Design**.

A common mistake is:

> Interviewer: "Design WhatsApp."

Candidate immediately starts drawing:

```text
Client
  ↓
API Gateway
  ↓
Microservices
  ↓
Kafka
  ↓
Redis
  ↓
Database
```

❌ Don't do this.

A senior engineer first asks:

> **"What exactly are we building, for whom, at what scale, and what constraints matter?"**

---

# 1. What is Requirements Gathering?

Requirements gathering means converting a vague problem statement into a **clear engineering problem**.

For example:

> "Design an Uber-like system."

This is too broad.

We need to determine:

```text
WHO?
 ↓
WHAT?
 ↓
HOW MUCH?
 ↓
HOW FAST?
 ↓
HOW RELIABLE?
 ↓
WHAT TRADE-OFFS?
```

---

# 2. Two Types of Requirements

Every system-design problem has two major categories:

```text
Requirements
│
├── Functional Requirements
│
└── Non-Functional Requirements
```

---

# 3. Functional Requirements

Functional requirements answer:

> **"What should the system DO?"**

For an e-commerce system:

```text
User
 │
 ├── Register/Login
 ├── Search products
 ├── View product
 ├── Add to cart
 ├── Checkout
 ├── Make payment
 └── Track order
```

These are functionalities.

### Example

If interviewer says:

> Design Amazon.

You might ask:

**Q: Can users search products?**

Interviewer:

> Yes.

**Q: Can users add products to cart?**

> Yes.

**Q: Can users place orders?**

> Yes.

**Q: Do we need payment processing?**

> Yes.

**Q: Do we need seller management?**

> No, keep it out of scope.

Now you've reduced the problem.

---

# 4. Don't Design Everything

This is extremely important in interviews.

Suppose the interviewer says:

> Design YouTube.

You could potentially design:

```text
User
Authentication
Upload
Video Processing
Recommendation
Search
Comments
Likes
Subscriptions
Notifications
Advertising
Payments
Analytics
Live Streaming
Moderation
...
```

That's enormous.

Instead say:

> "There are many components in YouTube. To keep the discussion focused, I'll concentrate on video upload, video processing, and video playback. I'll treat recommendations and advertising as out of scope unless you'd like me to cover them."

🔥 **That's a senior-level response.**

---

# 5. How to Identify Functional Requirements

Use this simple framework:

```text
Create
Read
Update
Delete
Search
Process
Notify
```

For example, an Order System:

| Operation | Requirement         |
| --------- | ------------------- |
| Create    | Create order        |
| Read      | Get order           |
| Update    | Update order status |
| Search    | Find user's orders  |
| Process   | Process payment     |
| Notify    | Notify user         |

You don't need to literally say CRUD in every interview, but thinking this way helps you discover requirements.

---

# 6. Non-Functional Requirements

Now comes the more important part for senior engineers.

Non-functional requirements answer:

> **"How well should the system work?"**

Examples:

```text
Performance
Scalability
Availability
Reliability
Consistency
Durability
Security
Latency
Throughput
Maintainability
Cost
```

---

# 7. Availability

Ask:

> **How available should the system be?**

Example:

```text
99%
99.9%
99.99%
99.999%
```

These sound similar but are very different.

Approximate yearly downtime:

| Availability | Downtime/year |
| ------------ | ------------: |
| 99%          |     3.65 days |
| 99.9%        |    8.76 hours |
| 99.99%       |  52.6 minutes |
| 99.999%      |  5.26 minutes |

So if the interviewer says:

> "The payment system needs 99.99% availability."

You immediately know reliability needs to be taken seriously.

---

# 8. Latency

Ask:

> **How quickly should the system respond?**

For example:

```text
Search API
p95 < 200 ms
```

Important terms:

### Average latency

```text
100 ms
```

But average can hide bad experiences.

### p95

95% of requests are faster than this.

### p99

99% of requests are faster than this.

Example:

```text
p50 = 50 ms
p95 = 150 ms
p99 = 500 ms
```

That tells you there are some very slow requests.

For senior interviews, get comfortable discussing **p95/p99**, not just average latency.

---

# 9. Throughput

Throughput means:

> **How much work can the system process per unit of time?**

Examples:

```text
10,000 requests/sec
50,000 messages/sec
1 million events/sec
```

This will become important in **Lesson 2: Capacity Estimation**.

---

# 10. Scalability

Ask:

> **How many users do we expect?**

Example:

```text
1 million users
10 million users
100 million users
```

But don't stop there.

Ask:

> "What's the expected growth over the next few years?"

Because:

```text
Today
1M users

Tomorrow
100M users
```

could completely change your architecture.

---

# 11. Consistency

This is a very important senior-level question.

Ask:

> **Does the user need to immediately see the latest data?**

Consider Instagram likes.

Suppose:

```text
User A likes post
       ↓
Database updated
       ↓
User B sees old like count
```

Is that acceptable?

Usually, temporary inconsistency might be acceptable.

But consider:

```text
Bank balance
Payment status
Inventory quantity
```

Temporary incorrect information could be much more serious.

So you need to determine:

```text
Strong consistency
        vs
Eventual consistency
```

---

# 12. Durability

Ask:

> **Can we ever lose data?**

For example:

### Social media likes

Maybe losing a tiny amount of analytics data is tolerable.

### Financial transaction

Losing a transaction is unacceptable.

Therefore:

```text
Payment
 ↓
Durability requirement = VERY HIGH
```

This affects:

* Database choice
* Replication
* Backups
* Disaster recovery
* Write strategy

---

# 13. Security Requirements

Ask:

> Does the system contain sensitive information?

For example:

```text
Payment
Personal information
Authentication
Private messages
Healthcare
Financial data
```

Then consider:

```text
Authentication
Authorization
Encryption
TLS
Secrets management
Audit logs
Rate limiting
```

You don't need to spend the entire interview discussing security unless it is central to the system.

---

# 14. Geographic Requirements

Ask:

> **Is this system global or regional?**

For example:

### Regional

```text
India
  ↓
Mumbai region
  ↓
Database
```

### Global

```text
              Global Users
                   |
          Global Load Balancer
          /        |        \
      US Region  EU Region  Asia Region
```

Global requirements introduce questions around:

* Multi-region
* Replication
* Data locality
* Latency
* Disaster recovery

---

# 15. Read vs Write Ratio

This is an excellent question.

Suppose we're designing a product catalog.

Maybe:

```text
Reads  = 95%
Writes = 5%
```

That's **read-heavy**.

Therefore caching and read replicas may be useful.

But a logging system might be:

```text
Reads  = 10%
Writes = 90%
```

That's **write-heavy**.

The architecture may be very different.

---

# 16. Example: Design an E-Commerce System

Let's conduct an actual interview.

### Interviewer

> Design an e-commerce system.

### Bad candidate

> I'll use microservices, Kafka, Redis, MongoDB and Kubernetes.

❌ Too early.

---

### Senior candidate

First:

> "I'd like to clarify the requirements."

Then ask:

### Functional

```text
1. Can users browse products?
2. Can users search?
3. Can users add products to cart?
4. Can users place orders?
5. Do we need payment processing?
6. Do we need inventory management?
7. Do we need order tracking?
```

Suppose interviewer says:

```text
Yes to all except seller management.
```

Now scope:

```text
IN SCOPE
──────────────
Product browsing
Search
Cart
Order
Payment
Inventory
Order tracking

OUT OF SCOPE
──────────────
Seller management
Advertising
Recommendation engine
```

---

# 17. Now Ask Non-Functional Questions

You:

> "How many users should the system support?"

Interviewer:

> 100 million registered users.

You:

> "How many daily active users?"

Interviewer:

> 10 million.

You:

> "What's the expected peak traffic?"

Interviewer:

> Around 10× average.

You:

> "What's the latency requirement for product browsing?"

Interviewer:

> p95 below 200 ms.

You:

> "What availability do we need?"

Interviewer:

> 99.99%.

You:

> "Can product information be eventually consistent?"

Interviewer:

> Yes, except inventory during checkout.

💡 Now you have meaningful architectural constraints.

---

# 18. Create an Interview Requirement Sheet

During an interview, mentally create something like this:

```text
SYSTEM
────────────────────────
E-Commerce Platform

FUNCTIONAL
────────────────────────
✓ Product browsing
✓ Product search
✓ Cart
✓ Checkout
✓ Payment
✓ Inventory
✓ Order tracking

OUT OF SCOPE
────────────────────────
✗ Seller management
✗ Recommendations
✗ Advertising

SCALE
────────────────────────
100M registered users
10M DAU
10× peak

PERFORMANCE
────────────────────────
Product API: p95 < 200ms

AVAILABILITY
────────────────────────
99.99%

CONSISTENCY
────────────────────────
Product → Eventual
Inventory → Stronger consistency

DURABILITY
────────────────────────
Orders/Payments → Very high

GEOGRAPHY
────────────────────────
Global

READ/WRITE
────────────────────────
Product → Read-heavy
Orders → Mixed
```

Now you can start designing.

---

# 19. The Magic Question: "What Matters Most?"

Sometimes requirements conflict.

For example:

```text
Very low latency
        +
Strong consistency
        +
Very high availability
        +
Very low cost
```

You probably can't maximize all four.

So ask:

> **"Which requirement is the highest priority?"**

This leads naturally to **trade-offs**.

For example:

```text
Payment System

Priority:
1. Correctness
2. Durability
3. Availability
4. Latency
```

Whereas:

```text
Social Media Feed

Priority:
1. Availability
2. Low latency
3. Scalability
4. Eventual consistency acceptable
```

---

# 20. What NOT to Ask

Avoid asking 30 questions one after another.

Bad:

```text
How many users?
What database?
What cloud?
How many servers?
What programming language?
What cache?
What queue?
What region?
What protocol?
...
```

Some of these aren't requirements.

Instead, ask questions that **change your architecture**.

That's the key.

---

# 21. The Senior-Level Requirements Checklist

Memorize this:

```text
             REQUIREMENTS
                   │
       ┌───────────┴───────────┐
       │                       │
 FUNCTIONAL              NON-FUNCTIONAL
       │                       │
       ├─ Users                ├─ Scale
       ├─ Features             ├─ Availability
       ├─ APIs                 ├─ Latency
       ├─ Workflows            ├─ Throughput
       └─ Scope                ├─ Consistency
                               ├─ Durability
                               ├─ Security
                               ├─ Geography
                               └─ Cost
```

Then ask:

```text
READ vs WRITE?
STRONG vs EVENTUAL CONSISTENCY?
REGIONAL vs GLOBAL?
NORMAL vs PEAK TRAFFIC?
WHAT IS OUT OF SCOPE?
WHAT IS THE HIGHEST PRIORITY?
```

---

# 22. A Very Important Interview Technique

Don't just ask a question.

Explain **why you're asking it**.

Instead of:

> "How many users?"

Say:

> "I'd like to understand the expected user and traffic scale because that will influence whether we need horizontal scaling, caching, database partitioning, and potentially multiple regions."

🔥 This demonstrates engineering thinking.

---

# 23. Your First Practice Exercise

Let's simulate the interview.

### Interviewer:

> **"Design a URL Shortener like Bitly."**

Don't design it yet.

As the candidate, your **first response should be requirements questions**.

For example:

```text
1. ...
2. ...
3. ...
4. ...
5. ...
```

Try to ask me **5–8 requirements questions** as if I were the interviewer.

I'll answer each one, and then we'll continue the interview step-by-step into **Lesson 2: Capacity Estimation**, using the answers we establish.
