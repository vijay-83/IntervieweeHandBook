# Volume 06 – Kubernetes, Istio & AKS Architecture
# Chapters 11–15 Full Edition

## Chapter 11 – Istio Architecture Deep Dive

### Learning Objectives
- Understand Service Mesh concepts
- Understand Istio control plane

### Architecture

```text
Istiod
 |
Envoy Sidecars
 |
Application Pods
```

### Components

#### Istiod
Control plane component.

#### Envoy
Data plane proxy.

### Benefits

- Security
- Traffic control
- Observability

### Interview Questions

1. What is a Service Mesh?
2. Why use Istio?
3. What is Istiod?

---

## Chapter 12 – Envoy, Gateway & VirtualService

### Envoy Proxy

Handles:

- Routing
- Security
- Telemetry

### Gateway

Entry point into the mesh.

### VirtualService

Defines traffic routing.

Example:

```yaml
kind: VirtualService
```

### Interview Questions

1. Gateway vs Ingress?
2. What does VirtualService do?

---

## Chapter 13 – DestinationRule, ServiceEntry & Egress

### DestinationRule

Defines traffic policies.

Examples:

- Load balancing
- Connection pools
- Circuit breakers

### ServiceEntry

Allows access to external services.

Example:

```text
MongoDB Atlas
External APIs
Azure Services
```

### Egress Gateway

Controls outbound traffic.

### Architecture Discussion

Useful for compliance and audit requirements.

---

## Chapter 14 – mTLS, Canary & Blue-Green Deployments

### Mutual TLS

Provides service-to-service encryption.

### Benefits

- Identity verification
- Encryption
- Zero-trust networking

### Canary Deployment

```text
90% -> v1
10% -> v2
```

### Blue-Green Deployment

```text
Blue Environment
Green Environment
```

Switch traffic when validated.

### Interview Questions

1. Canary vs Blue-Green?
2. Why use mTLS?

---

## Chapter 15 – Observability & Istio Troubleshooting

### Observability Tools

- Prometheus
- Grafana
- Jaeger
- Kiali

### Common Problems

#### 503 Errors

Check:

- DestinationRule
- VirtualService
- Endpoint health

#### Traffic Not Routing

Check:

- Gateway
- Hostnames
- DNS

#### External Access Failure

Check:

- ServiceEntry
- Egress rules

### AKS Best Practices

- Resource limits
- Sidecar monitoring
- Version compatibility
- Progressive rollouts

---

# Production Case Study

## AKS + Istio + MongoDB Atlas

```text
Client
 |
Gateway
 |
Istio Mesh
 |
FastAPI Services
 |
MongoDB Atlas
 |
Redis
```

### Design Goals

- Security
- Observability
- Scalability
- Reliability

---

# Staff Engineer Questions

1. How would you standardize Istio configuration?
2. How would you govern service mesh adoption?
3. How would you troubleshoot mesh-wide failures?

---

# Solution Architect Questions

1. When should Istio be avoided?
2. How would you secure east-west traffic?
3. How would you design multi-region service meshes?

---

# Summary

Key Takeaways

- Istio provides service mesh capabilities.
- Envoy powers the data plane.
- VirtualServices control routing.
- ServiceEntry enables external access.
- mTLS improves security.
- Observability is essential for production meshes.

# End of Volume 06 Chapters 11–15
