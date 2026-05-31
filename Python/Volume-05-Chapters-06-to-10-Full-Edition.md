# Volume 05 – Microservices, Kafka & RabbitMQ Architecture
# Chapters 6–10 Full Edition

## Chapter 6 – Kafka Internals

### Learning Objectives
- Understand Kafka architecture
- Understand distributed log concepts

### Kafka Architecture

```text
Producer
 |
Broker
 |
Topic
 |
Consumer
```

### Core Components

- Broker
- Topic
- Partition
- Consumer Group

### Interview Questions

1. What is Kafka?
2. Why is Kafka scalable?
3. Why is Kafka called a distributed log?

---

## Chapter 7 – Topics, Partitions & Offsets

### Topic

Logical stream of events.

### Partition

Unit of scalability.

```text
Topic Orders
 |
P1
P2
P3
```

### Offset

Unique position within a partition.

### Benefits

- Replay capability
- Fault tolerance

### Interview Questions

1. What is an offset?
2. Why use partitions?
3. Can ordering be guaranteed?

---

## Chapter 8 – Consumer Groups & Event Ordering

### Consumer Groups

```text
Topic
 |
Consumer Group
 |
Consumer 1
Consumer 2
Consumer 3
```

### Benefits

- Horizontal scaling
- Fault tolerance

### Event Ordering

Guaranteed only within a partition.

### Common Mistakes

- Assuming global ordering
- Excessive partitions

### Architecture Discussion

Partition strategy impacts scalability and ordering guarantees.

---

## Chapter 9 – Schema Registry & Kafka Streams

### Schema Registry

Stores event schemas.

### Benefits

- Schema evolution
- Consumer compatibility

### Kafka Streams

Processes events directly from Kafka.

### Use Cases

- Aggregations
- Enrichment
- Real-time analytics

### Interview Questions

1. Why use Schema Registry?
2. What is schema evolution?
3. Kafka Streams vs Spark?

---

## Chapter 10 – Kafka Scaling & Production Troubleshooting

### Scaling Techniques

- Increase partitions
- Add brokers
- Scale consumers

### Common Problems

#### Consumer Lag

Causes:

- Slow consumers
- Large messages
- Resource limits

#### Broker Failure

Mitigation:

- Replication
- Multi-broker clusters

#### Rebalancing Storms

Causes:

- Frequent consumer restarts

### Performance Tuning

- Batch size
- Compression
- Retention policies

---

# Staff Engineer Questions

1. How would you design an event platform?
2. How would you monitor Kafka?
3. How would you govern event schemas?

---

# Solution Architect Questions

1. Kafka vs RabbitMQ?
2. When should event streaming be used?
3. How would you design a platform processing billions of events?

---

# Summary

Key Takeaways

- Kafka is a distributed event streaming platform.
- Partitions provide scale.
- Offsets enable replay.
- Consumer groups enable horizontal scaling.
- Schema Registry improves compatibility.
- Monitoring consumer lag is critical.

# End of Volume 05 Chapters 6–10
