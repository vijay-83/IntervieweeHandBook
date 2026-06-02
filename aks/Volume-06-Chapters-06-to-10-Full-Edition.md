# Volume 06 – Kubernetes, Istio & AKS Architecture
# Chapters 6–10 Full Edition

## Chapter 6 – StatefulSets & Persistent Storage

### Learning Objectives
- Understand stateful workloads
- Manage persistent data

### StatefulSet

Used for:

- PostgreSQL
- MongoDB
- Redis
- Kafka

### Benefits

- Stable network identity
- Ordered deployment
- Persistent storage

### Example

```yaml
kind: StatefulSet
```

### Interview Questions

1. Deployment vs StatefulSet?
2. When should StatefulSets be used?

---

## Chapter 7 – Persistent Volumes & Storage Classes

### Persistent Volume (PV)

Represents storage resource.

### Persistent Volume Claim (PVC)

Requests storage.

### Storage Class

Defines dynamic provisioning.

### Flow

```text
PVC
 |
Storage Class
 |
PV
```

### AKS Examples

- Azure Disk
- Azure Files

### Interview Questions

1. PV vs PVC?
2. Why use Storage Classes?

---

## Chapter 8 – ETCD & Kubernetes Scheduling

### ETCD

Stores cluster state.

### Scheduler Responsibilities

- Node selection
- Resource placement
- Affinity rules

### Scheduling Factors

- Resources
- Labels
- Taints
- Affinity

### Interview Questions

1. What is ETCD?
2. What happens if ETCD is unavailable?

---

## Chapter 9 – Taints, Tolerations & Affinity

### Taints

Prevent scheduling.

Example:

```bash
kubectl taint nodes
```

### Tolerations

Allow scheduling onto tainted nodes.

### Node Affinity

Prefer specific nodes.

### Pod Anti-Affinity

Spread workloads.

### Production Example

Dedicated AI workloads on GPU nodes.

### Interview Questions

1. Taints vs Affinity?
2. Why use Anti-Affinity?

---

## Chapter 10 – Network Policies & Governance

### Network Policies

Control pod communication.

### Example

```text
Frontend -> Backend
Allowed

Frontend -> Database
Blocked
```

### Governance Controls

- Resource Quotas
- Limit Ranges
- Pod Security Standards

### Benefits

- Security
- Multi-tenancy
- Compliance

### Architect Discussion

Zero-trust networking should be a default design principle.

---

# Staff Engineer Questions

1. How would you standardize storage patterns?
2. How would you enforce security controls?
3. How would you govern namespaces?

---

# Solution Architect Questions

1. How would you design a multi-tenant AKS platform?
2. How would you isolate regulated workloads?
3. How would you design secure cluster networking?

---

# Summary

Key Takeaways

- StatefulSets support stateful workloads.
- PVs and PVCs manage storage.
- ETCD stores cluster state.
- Scheduling controls workload placement.
- Network Policies improve security.

# End of Volume 06 Chapters 6–10
