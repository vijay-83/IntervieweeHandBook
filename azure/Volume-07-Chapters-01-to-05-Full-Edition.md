# Volume 07 – Azure Architecture
# Chapters 1–5 Full Edition

## Chapter 1 – Azure Landing Zones

### Learning Objectives
- Understand enterprise Azure foundations
- Implement governance at scale

### Definition

Azure Landing Zones provide a pre-configured environment for cloud adoption.

### Components

- Management Groups
- Subscriptions
- Networking
- Security
- Governance

### Benefits

- Standardization
- Security
- Scalability

### Interview Questions

1. What is a Landing Zone?
2. Why are Landing Zones important?

---

## Chapter 2 – Management Groups & Subscriptions

### Hierarchy

```text
Tenant
 |
Management Groups
 |
Subscriptions
 |
Resource Groups
```

### Best Practices

- Separate production and non-production
- Use management groups for governance
- Apply policies centrally

### Interview Questions

1. Management Group vs Subscription?
2. Why separate workloads?

---

## Chapter 3 – Resource Groups & RBAC

### Resource Group

Logical container for Azure resources.

### RBAC Roles

- Owner
- Contributor
- Reader
- Custom Roles

### Principle of Least Privilege

Grant only required permissions.

### Interview Questions

1. RBAC vs Access Policies?
2. Why use custom roles?

---

## Chapter 4 – Azure Policy & Governance

### Azure Policy

Enforces standards.

Examples:

- Allowed regions
- Approved SKUs
- Tag enforcement

### Governance Benefits

- Compliance
- Cost control
- Security

### Example

```text
Require Tags
Deny Public IPs
```

### Interview Questions

1. Azure Policy vs RBAC?
2. What governance controls should exist?

---

## Chapter 5 – Azure Networking Fundamentals

### Core Components

- VNet
- Subnets
- NSGs
- Route Tables
- Private Endpoints

### Architecture

```text
Internet
 |
Firewall
 |
VNet
 |
Subnet
 |
Resources
```

### Best Practices

- Network segmentation
- Private endpoints
- Zero-trust networking

### Architect Questions

1. How would you secure Azure networking?
2. When should private endpoints be used?

---

# Summary

Key Takeaways

- Landing Zones provide cloud foundations.
- Management Groups improve governance.
- RBAC controls access.
- Azure Policy enforces standards.
- Networking design affects security and scalability.

# End of Volume 07 Chapters 1–5
