# Volume 07 – Azure Architecture
# Chapters 6–10 Full Edition

## Chapter 6 – Hub-Spoke Architecture

### Learning Objectives
- Design enterprise Azure networks
- Centralize shared services

### Architecture

```text
Hub VNet
 |
Firewall
VPN
DNS
 |
Spoke VNets
```

### Benefits

- Centralized security
- Reduced complexity
- Better governance

### Interview Questions

1. What is Hub-Spoke?
2. Why not connect all VNets directly?

---

## Chapter 7 – Virtual WAN & ExpressRoute

### Virtual WAN

Provides global network connectivity.

### Benefits

- Branch connectivity
- Global routing
- Simplified operations

### ExpressRoute

Private connection to Azure.

### Advantages

- Lower latency
- Predictable performance
- Enhanced security

### Interview Questions

1. VPN vs ExpressRoute?
2. When should Virtual WAN be used?

---

## Chapter 8 – Azure Firewall, NSG & ASG

### Azure Firewall

Centralized network security service.

### NSG

Controls subnet and NIC traffic.

### ASG

Application Security Groups simplify rule management.

### Best Practices

- Deny by default
- Least privilege access
- Centralized logging

### Interview Questions

1. NSG vs Azure Firewall?
2. Why use ASGs?

---

## Chapter 9 – Private Link & Private Endpoints

### Private Endpoint

Private IP access to Azure services.

### Private Link

Provides secure service connectivity.

### Example Services

- Key Vault
- Storage Account
- SQL Database
- Cosmos DB

### Benefits

- No public exposure
- Reduced attack surface

### Architect Discussion

Private endpoints are preferred for enterprise workloads.

---

## Chapter 10 – Azure Front Door & Application Gateway

### Azure Front Door

Global Layer-7 load balancer.

### Features

- Global routing
- SSL termination
- WAF
- CDN integration

### Application Gateway

Regional web traffic load balancer.

### Architecture

```text
Users
 |
Front Door
 |
Application Gateway
 |
AKS
```

### Interview Questions

1. Front Door vs Application Gateway?
2. When should both be used together?

---

# Staff Engineer Questions

1. How would you standardize enterprise networking?
2. How would you govern private connectivity?
3. How would you secure multi-region deployments?

---

# Solution Architect Questions

1. How would you design global Azure networking?
2. Hub-Spoke vs Virtual WAN?
3. How would you design secure hybrid connectivity?

---

# Summary

Key Takeaways

- Hub-Spoke centralizes networking.
- Virtual WAN simplifies global connectivity.
- ExpressRoute provides private connectivity.
- Private Endpoints improve security.
- Front Door enables global traffic management.

# End of Volume 07 Chapters 6–10
