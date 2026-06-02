# Volume 06 – Kubernetes, Istio & AKS Architecture
# Chapters 1–5 Full Edition

## Chapter 1 – Kubernetes Internals

### Learning Objectives
- Understand Kubernetes architecture
- Understand cluster components

### Architecture

```text
Control Plane
 |
Worker Nodes
 |
Pods
```

### Control Plane Components

- API Server
- Scheduler
- Controller Manager
- ETCD

### Interview Questions

1. What is Kubernetes?
2. What is ETCD?
3. Why is the API Server important?

---

## Chapter 2 – Pods, Deployments & Services

### Pod

Smallest deployable unit.

### Deployment

Manages replica sets and rolling updates.

### Service Types

- ClusterIP
- NodePort
- LoadBalancer

### Example

```yaml
apiVersion: apps/v1
kind: Deployment
```

### Interview Questions

1. Pod vs Deployment?
2. Service vs Ingress?
3. Why use replicas?

---

## Chapter 3 – ConfigMaps & Secrets

### ConfigMap

Stores configuration.

### Secret

Stores sensitive information.

### Best Practices

- Avoid hardcoded values
- Use workload identity when possible
- Rotate secrets regularly

### Interview Questions

1. ConfigMap vs Secret?
2. Why avoid secrets in code?

---

## Chapter 4 – Ingress Controllers & Networking

### Ingress

Provides external access.

### Components

```text
Client
 |
Ingress
 |
Service
 |
Pod
```

### Common Controllers

- NGINX
- Azure Application Gateway
- Istio Gateway

### Interview Questions

1. What is Ingress?
2. Ingress vs Service?

---

## Chapter 5 – HPA, VPA & Cluster Scaling

### HPA

Horizontal Pod Autoscaler

Scales pods based on:

- CPU
- Memory
- Custom Metrics

### VPA

Vertical Pod Autoscaler

Adjusts resource requests.

### Cluster Autoscaler

Adds or removes nodes automatically.

### Architect Discussion

Scaling decisions affect cost, performance, and reliability.

---

# Staff Engineer Questions

1. How would you standardize Kubernetes deployments?
2. How would you improve cluster governance?

---

# Solution Architect Questions

1. How would you design a multi-tenant Kubernetes platform?
2. How would you scale AKS globally?

---

# Summary

Key Takeaways

- Kubernetes manages containerized workloads.
- Deployments provide lifecycle management.
- Services expose applications.
- HPA improves elasticity.
- Proper governance improves platform reliability.

# End of Volume 06 Chapters 1–5
