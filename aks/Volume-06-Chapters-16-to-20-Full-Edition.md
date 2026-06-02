# Volume 06 – Kubernetes, Istio & AKS Architecture
# Chapters 16–20 Full Edition

## Chapter 16 – AKS Architecture Deep Dive

### Learning Objectives
- Understand AKS architecture
- Understand Azure-managed Kubernetes

### AKS Components

```text
Azure Control Plane
 |
Node Pools
 |
Pods
```

### Benefits

- Managed control plane
- Automatic upgrades
- Azure integrations

### Best Practices

- Separate system and user node pools
- Enable monitoring
- Use workload identity

### Interview Questions

1. What does Azure manage in AKS?
2. Why separate node pools?

---

## Chapter 17 – Azure Networking for AKS

### Azure CNI

Pods receive VNet IP addresses.

### Kubenet

Pods use overlay networking.

### Azure CNI Overlay

Combines scale and simplified networking.

### Comparison

| Feature | Azure CNI | Kubenet | Overlay |
|----------|------------|----------|---------|
| VNet IPs | Yes | No | No |
| Scale | Medium | High | High |
| Simplicity | Medium | High | High |

### Interview Questions

1. Azure CNI vs Kubenet?
2. When should Overlay be used?

---

## Chapter 18 – Workload Identity & Key Vault Integration

### Workload Identity

Provides pod-level Azure authentication.

### Architecture

```text
Pod
 |
Federated Identity
 |
Managed Identity
 |
Azure Resource
```

### Use Cases

- Key Vault
- Storage Account
- Service Bus
- Redis

### Benefits

- No secrets
- Better security
- Simplified rotation

### Interview Questions

1. Workload Identity vs Managed Identity?
2. Why avoid client secrets?

---

## Chapter 19 – Fleet Manager & Multi-Cluster AKS

### Fleet Manager

Centralized management for multiple clusters.

### Benefits

- Governance
- Consistency
- Multi-region operations

### Multi-Cluster Architecture

```text
Fleet Manager
 |
Cluster A
Cluster B
Cluster C
```

### Common Use Cases

- Regional deployments
- Regulatory boundaries
- Blue/Green clusters

### Staff Engineer Discussion

How would you standardize policies across clusters?

---

## Chapter 20 – Multi-Region Failover & Disaster Recovery

### Architecture

```text
Azure Front Door
 |
Primary Region
 |
Secondary Region
```

### Failover Strategies

#### Active-Passive

Lower cost.

#### Active-Active

Higher availability.

### Recovery Objectives

#### RTO

Recovery Time Objective

#### RPO

Recovery Point Objective

### Common Failure Scenarios

- Region outage
- Database outage
- DNS issues

### Mitigation

- Replication
- Automated failover
- Health probes

### Architect Questions

1. Active-Active vs Active-Passive?
2. How would you design DR for AKS?

---

# Production Case Study

## Multi-Region FastAPI Platform

```text
Azure Front Door
 |
AKS (SCUS)
 |
AKS (EUS)
 |
Redis
 |
MongoDB Atlas
```

### Goals

- High Availability
- Low Latency
- Disaster Recovery

---

# Staff Engineer Questions

1. How would you govern AKS clusters globally?
2. How would you standardize Workload Identity?
3. How would you reduce operational overhead?

---

# Solution Architect Questions

1. How would you design a global AKS platform?
2. How would you handle regional outages?
3. How would you secure multi-cluster communication?

---

# Summary

Key Takeaways

- AKS simplifies Kubernetes operations.
- Azure networking choices affect scale and design.
- Workload Identity improves security.
- Fleet Manager simplifies multi-cluster governance.
- Disaster recovery planning is essential.
- Multi-region architectures improve availability.

# End of Volume 06 Chapters 16–20
