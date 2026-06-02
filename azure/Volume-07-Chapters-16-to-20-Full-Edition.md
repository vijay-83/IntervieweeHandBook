# Volume 07 – Azure Architecture
# Chapters 16–20 Full Edition

## Chapter 16 – Microsoft Defender for Cloud

### Learning Objectives
- Understand cloud security posture management
- Detect threats across Azure resources

### Features

- Secure Score
- Recommendations
- Threat Protection
- Regulatory Compliance

### Benefits

- Continuous assessment
- Security visibility
- Compliance monitoring

### Interview Questions

1. What is Defender for Cloud?
2. What is Secure Score?

---

## Chapter 17 – Microsoft Sentinel

### SIEM & SOAR Platform

### Data Sources

- Azure
- AWS
- On-Premises
- Applications

### Architecture

```text
Resources
 |
Log Analytics
 |
Sentinel
 |
Alerts
```

### Benefits

- Centralized security monitoring
- Automated response

### Interview Questions

1. What is Sentinel?
2. SIEM vs SOAR?

---

## Chapter 18 – Zero Trust Architecture

### Principles

- Verify explicitly
- Least privilege
- Assume breach

### Components

- Identity
- Devices
- Network
- Applications
- Data

### Enterprise Design

```text
User
 |
Identity Verification
 |
Application Access
```

### Interview Questions

1. What is Zero Trust?
2. Why assume breach?

---

## Chapter 19 – Disaster Recovery & Business Continuity

### Key Metrics

#### RTO
Recovery Time Objective

#### RPO
Recovery Point Objective

### DR Strategies

- Backup & Restore
- Active-Passive
- Active-Active

### Business Continuity

Focuses on maintaining critical services.

### Architect Questions

1. RTO vs RPO?
2. Active-Active vs Active-Passive?

---

## Chapter 20 – Enterprise Azure Architecture Case Study

### Global Platform

```text
Management Groups
 |
Landing Zones
 |
Azure Front Door
 |
AKS Regions
 |
Key Vault
 |
Azure Monitor
```

### Goals

- Security
- Governance
- Reliability
- Cost Optimization

### Design Considerations

- Multi-region
- Zero Trust
- Observability
- Compliance

---

# Staff Engineer Scenarios

1. Standardize Azure governance across hundreds of subscriptions.
2. Reduce cloud spend without impacting reliability.
3. Implement enterprise observability.

---

# Solution Architect Interview Scenarios

1. Design a multi-region SaaS platform.
2. Design enterprise landing zones.
3. Build a secure zero-trust architecture.

---

# Summary

Key Takeaways

- Defender for Cloud improves security posture.
- Sentinel provides centralized monitoring.
- Zero Trust improves security architecture.
- DR planning improves resilience.
- Enterprise architecture balances security, cost, and agility.

# End of Volume 07 Chapters 16–20
