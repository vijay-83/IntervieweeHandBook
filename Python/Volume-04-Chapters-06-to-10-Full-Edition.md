# Volume 04 – PostgreSQL, MongoDB & Redis Architecture
# Chapters 6–10 Full Edition

## Chapter 6 – MongoDB Internals

### Learning Objectives
- Understand MongoDB architecture
- Understand document storage

### Architecture

```text
Client
 |
MongoDB
 |
Collections
 |
Documents (BSON)
```

### Key Features

- Schema flexibility
- Horizontal scaling
- High availability

### Interview Questions

1. What is BSON?
2. How does MongoDB store data?
3. MongoDB vs PostgreSQL?

---

## Chapter 7 – Collections, Documents & Indexes

### Document Example

```json
{
  "_id": 1,
  "name": "Vijay",
  "city": "Bangalore"
}
```

### Index Types

- Single Field
- Compound
- Text
- Geospatial
- TTL

### Best Practices

- Index frequently queried fields
- Avoid excessive indexes

### Interview Questions

1. What is a collection?
2. Compound vs single index?
3. What is a TTL index?

---

## Chapter 8 – Aggregation Framework

### Purpose

Perform complex analytics within MongoDB.

### Example

```javascript
db.orders.aggregate([
 { $group: { _id: "$status", count: { $sum: 1 } } }
])
```

### Common Stages

- $match
- $project
- $group
- $sort
- $lookup

### Architecture Discussion

Aggregation pipelines reduce application-side processing.

---

## Chapter 9 – Replica Sets & Sharding

### Replica Set

```text
Primary
 |
Secondary
 |
Secondary
```

Benefits:

- High Availability
- Automatic Failover

### Sharding

```text
Router
 |
Shard 1
Shard 2
Shard 3
```

Benefits:

- Horizontal scaling
- Large dataset support

### Interview Questions

1. What is a replica set?
2. What is sharding?
3. When should sharding be used?

---

## Chapter 10 – Data Modeling & Performance Tuning

### Embedding

Store related data together.

### Referencing

Store relationships separately.

### Example Decision

Embedding:
- User + Addresses

Referencing:
- Orders + Products

### Performance Tips

- Proper indexes
- Avoid large documents
- Optimize aggregation pipelines

### MongoDB vs PostgreSQL

MongoDB:
- Flexible schema
- Horizontal scaling

PostgreSQL:
- Strong consistency
- Complex joins

### Architect Discussion

Choose databases based on workload characteristics, not popularity.

---

# Production Troubleshooting

## Slow Queries

Steps:

1. Check indexes
2. Review execution plan
3. Analyze aggregation stages

## Replication Lag

Steps:

1. Review network latency
2. Check write volume
3. Monitor replica health

---

# Staff Engineer Questions

1. How would you standardize MongoDB schema design?
2. How would you govern indexing strategy?
3. How would you monitor MongoDB clusters?

---

# Solution Architect Questions

1. MongoDB vs PostgreSQL for e-commerce?
2. When should sharding be introduced?
3. How would you design a globally distributed database?

---

# Summary

Key Takeaways

- MongoDB stores BSON documents.
- Aggregation Framework enables analytics.
- Replica Sets provide HA.
- Sharding provides scale.
- Data modeling decisions impact performance.

# End of Volume 04 Chapters 6–10
