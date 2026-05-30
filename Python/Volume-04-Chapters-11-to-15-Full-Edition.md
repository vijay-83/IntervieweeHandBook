# Volume 04 – PostgreSQL, MongoDB & Redis Architecture
# Chapters 11–15 Full Edition

## Chapter 11 – Redis Internals

### Learning Objectives
- Understand Redis architecture
- Understand in-memory storage

### Architecture

```text
Client
 |
Redis Server
 |
Memory
```

### Why Redis is Fast

- In-memory storage
- Single-threaded event loop
- Efficient data structures

### Interview Questions

1. Why is Redis fast?
2. Is Redis a database or cache?
3. What are Redis persistence options?

---

## Chapter 12 – Redis Data Structures

### String

```text
user:1 = Vijay
```

### Hash

```text
user:1:name
user:1:city
```

### List

Queue-like behavior.

### Set

Unique values.

### Sorted Set

Ranking and leaderboards.

### Stream

Event streaming.

### Interview Questions

1. Hash vs String?
2. When should Sorted Sets be used?
3. Redis Streams vs Kafka?

---

## Chapter 13 – Caching Patterns

### Cache Aside

```text
App
 |
Cache
 |
Database
```

### Read Through

Cache fetches data automatically.

### Write Through

Update cache and database together.

### Write Behind

Update cache immediately, database later.

### Common Issues

- Cache Stampede
- Cache Penetration
- Cache Invalidation

### Architect Discussion

There are only two hard things:
- Cache invalidation
- Naming things

---

## Chapter 14 – Distributed Caching, Pub/Sub & Streams

### Redis Cluster

Provides horizontal scaling.

### Redis Sentinel

Provides high availability.

### Pub/Sub

```text
Publisher
 |
Redis
 |
Subscribers
```

### Streams

Event-driven architecture support.

### Use Cases

- Notifications
- Audit Events
- Event Processing

### Interview Questions

1. Sentinel vs Cluster?
2. Pub/Sub vs Streams?
3. When should Streams be used?

---

## Chapter 15 – Rate Limiting, Distributed Locks & Performance

### Rate Limiting

Example:

```text
100 requests/minute
```

### Token Bucket Pattern

Common implementation.

### Distributed Locks

Example:

```text
SET resource_lock NX EX 30
```

### Use Cases

- Scheduled jobs
- Inventory updates
- Leader election

### Performance Tuning

#### Optimize Memory

- TTL
- Eviction policies

#### Monitor

- Hit ratio
- Latency
- Memory usage

### Production Troubleshooting

#### High Memory Usage

Check:

- Large keys
- Expired data
- Eviction policy

#### Slow Redis

Check:

- Network latency
- Command complexity
- Cluster imbalance

---

# Staff Engineer Questions

1. How would you standardize caching across services?
2. How would you prevent cache stampedes?
3. How would you govern Redis usage?

---

# Solution Architect Questions

1. Redis vs Database?
2. When should Redis Cluster be used?
3. How would you design a globally distributed cache?

---

# Summary

Key Takeaways

- Redis is an in-memory data store.
- Multiple data structures support diverse workloads.
- Caching patterns affect consistency and performance.
- Sentinel provides HA.
- Cluster provides scale.
- Distributed locks enable coordination.

# End of Volume 04 Chapters 11–15
