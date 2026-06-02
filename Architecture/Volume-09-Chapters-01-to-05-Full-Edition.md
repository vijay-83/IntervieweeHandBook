# Volume 09 – System Design & Solution Architecture
# Chapters 1–5 Full Edition

## Chapter 1 – Scalability Fundamentals

### Learning Objectives
- Understand horizontal and vertical scaling
- Design scalable systems

### Vertical Scaling

```text
More CPU
More Memory
```

### Horizontal Scaling

```text
Instance 1
Instance 2
Instance 3
```

### Trade-Offs

Vertical:
- Simple
- Limited growth

Horizontal:
- Better scalability
- Increased complexity

### Interview Questions

1. Horizontal vs Vertical scaling?
2. When should each be used?

---

## Chapter 2 – CAP Theorem

### Definition

A distributed system can only guarantee two of:

- Consistency
- Availability
- Partition Tolerance

### Examples

CP Systems:
- MongoDB (certain modes)

AP Systems:
- Cassandra

### Architecture Discussion

Network partitions are unavoidable.

### Interview Questions

1. Explain CAP theorem.
2. Why is partition tolerance mandatory?

---

## Chapter 3 – Consistency Models

### Strong Consistency

Latest data always returned.

### Eventual Consistency

Data converges over time.

### Session Consistency

User sees their own writes.

### Examples

- Banking → Strong Consistency
- Social Media → Eventual Consistency

### Interview Questions

1. Strong vs Eventual consistency?
2. When is eventual consistency acceptable?

---

## Chapter 4 – Load Balancers & Reverse Proxies

### Load Balancer

Distributes traffic.

### Algorithms

- Round Robin
- Least Connections
- Weighted Routing

### Reverse Proxy

Acts on behalf of servers.

### Architecture

```text
Users
 |
Load Balancer
 |
Application Servers
```

### Interview Questions

1. Reverse Proxy vs Load Balancer?
2. Why use load balancing?

---

## Chapter 5 – API Gateway & High Availability

### API Gateway Responsibilities

- Authentication
- Authorization
- Routing
- Rate Limiting

### High Availability Principles

- Redundancy
- Failover
- Health Checks

### Architecture

```text
Users
 |
API Gateway
 |
Services
```

### Interview Questions

1. API Gateway vs Service Mesh?
2. What is High Availability?

---

# Staff Engineer Questions

1. How would you scale a FastAPI platform?
2. How would you reduce single points of failure?

---

# Solution Architect Questions

1. Design a globally available API platform.
2. Design for 99.99% uptime.

---

# Summary

Key Takeaways

- Scalability drives architecture decisions.
- CAP theorem explains distributed system tradeoffs.
- Consistency models impact user experience.
- Load balancers improve reliability.
- API Gateways centralize concerns.

# End of Volume 09 Chapters 1–5
