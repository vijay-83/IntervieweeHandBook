# Volume 05 – Microservices, Kafka & RabbitMQ Architecture
# Chapters 11–15 Full Edition

## Chapter 11 – RabbitMQ Internals

### Learning Objectives
- Understand RabbitMQ architecture
- Understand message brokers

### Architecture

```text
Producer
 |
Exchange
 |
Queue
 |
Consumer
```

### Exchange Types

- Direct
- Topic
- Fanout
- Headers

### Benefits

- Reliable messaging
- Routing flexibility
- Decoupled services

### Interview Questions

1. Kafka vs RabbitMQ?
2. What is an Exchange?
3. What is a Queue?

---

## Chapter 12 – Routing Keys & Dead Letter Queues

### Routing Key

Determines how messages are routed.

### Example

```text
order.created
order.completed
```

### Dead Letter Queue (DLQ)

Stores failed messages.

### Benefits

- Failure isolation
- Easier debugging
- Safe retries

### Interview Questions

1. Why use DLQs?
2. What causes messages to enter a DLQ?

---

## Chapter 13 – Retry Patterns & Competing Consumers

### Retry Strategies

- Immediate Retry
- Delayed Retry
- Exponential Backoff

### Competing Consumers

```text
Queue
 |
Consumer1
Consumer2
Consumer3
```

### Benefits

- Horizontal scaling
- Improved throughput

### Common Problems

- Duplicate processing
- Poison messages

---

## Chapter 14 – Saga Pattern & Distributed Transactions

### Problem

Microservices cannot use traditional database transactions.

### Saga Pattern

Coordinates distributed transactions.

### Choreography

```text
Order -> Payment -> Inventory
```

### Orchestration

```text
Saga Orchestrator
 |
Services
```

### Benefits

- Scalability
- Decoupling

### Challenges

- Compensation logic
- Complexity

### Interview Questions

1. What is a Saga?
2. Choreography vs Orchestration?

---

## Chapter 15 – Outbox Pattern & Reliability

### Outbox Pattern

Ensures database updates and event publishing remain consistent.

### Flow

```text
Database Transaction
 |
Outbox Table
 |
Publisher
 |
Broker
```

### Benefits

- Reliability
- Event consistency

### Eventual Consistency

Systems become consistent over time rather than immediately.

### Production Troubleshooting

#### Message Backlog

Check:
- Consumer lag
- Queue depth
- Processing latency

#### Duplicate Events

Mitigation:
- Idempotency
- Event versioning

---

# Staff Engineer Questions

1. How would you standardize messaging patterns?
2. How would you govern event schemas?
3. How would you prevent duplicate processing?

---

# Solution Architect Questions

1. Kafka vs RabbitMQ?
2. Saga vs 2PC?
3. How would you design resilient event-driven systems?

---

# Summary

Key Takeaways

- RabbitMQ excels at message routing.
- DLQs improve reliability.
- Retry patterns improve resilience.
- Sagas manage distributed transactions.
- Outbox pattern improves consistency.
- Eventual consistency is fundamental in microservices.

# End of Volume 05 Chapters 11–15
