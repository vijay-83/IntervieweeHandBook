# Volume 02 – OOP, SOLID & Clean Architecture
# Chapters 11–15 Full Edition

## Chapter 11 – Hexagonal Architecture

### Learning Objectives
- Understand Ports and Adapters
- Decouple business logic from infrastructure

### Short Interview Answer

Hexagonal Architecture isolates the domain from external systems through ports and adapters.

### Core Components

#### Ports
Interfaces defining required behavior.

#### Adapters
Implementations connecting external systems.

### Example

```text
Application
   |
Port
   |
Adapter
   |
Database / API
```

### Benefits

- Testability
- Technology independence
- Maintainability

---

## Chapter 12 – Onion Architecture

### Learning Objectives
- Organize applications around the domain

### Layers

```text
Domain
Application
Infrastructure
Presentation
```

### Rule

Dependencies point inward.

### Interview Question

How is Onion Architecture different from layered architecture?

### Answer

Business logic remains at the center and does not depend on infrastructure.

---

## Chapter 13 – Clean Architecture Deep Dive

### Learning Objectives
- Master enterprise architecture patterns

### Layers

#### Domain
Business rules.

#### Application
Use cases.

#### Infrastructure
External systems.

#### Presentation
UI / APIs.

### Dependency Rule

Source code dependencies always point inward.

### Benefits

- Testability
- Flexibility
- Long-term maintainability

### Interview Questions

1. Why use Clean Architecture?
2. What belongs in the Domain layer?
3. Why avoid infrastructure dependencies?

---

## Chapter 14 – Application Services & Domain Services

### Application Service

Coordinates use cases.

Example:

```python
class CreateOrderService:
    pass
```

### Domain Service

Contains business logic not belonging to an entity.

Example:

```python
class PricingService:
    pass
```

### Interview Question

Application Service vs Domain Service?

### Answer

Application Services orchestrate workflows.

Domain Services contain business rules.

---

## Chapter 15 – Anti-Corruption Layer & Event Sourcing

### Anti-Corruption Layer (ACL)

Protects a domain from external systems.

### Example

```text
Legacy System
      |
ACL
      |
Modern Domain
```

### Benefits

- Prevents domain pollution
- Simplifies migrations

### Event Sourcing

Store events instead of current state.

Example:

```text
OrderCreated
OrderUpdated
OrderShipped
```

### Benefits

- Auditability
- Replay capability

### Challenges

- Complexity
- Event evolution

### Architect Discussion

Event sourcing works best where audit history is critical.

---

# Architecture Case Study

## E-Commerce Platform

### Components

```text
API Gateway
 |
Order Service
 |
Payment Service
 |
Inventory Service
 |
Kafka
 |
Database
```

### Recommended Patterns

- Clean Architecture
- DDD
- CQRS
- Domain Events

---

# Architecture Review Checklist

## Domain

- Is business logic isolated?
- Are aggregates small?

## Application

- Clear use cases?
- Proper orchestration?

## Infrastructure

- Hidden behind interfaces?
- Replaceable?

## Observability

- Logs?
- Metrics?
- Traces?

---

# Staff Engineer Questions

1. How would you standardize architecture across teams?
2. How would you review service boundaries?
3. How would you introduce DDD gradually?

---

# Solution Architect Questions

1. Hexagonal vs Clean Architecture?
2. When should Event Sourcing be used?
3. How would you modernize a monolith?

---

# Summary

Key Takeaways

- Hexagonal Architecture uses ports and adapters.
- Onion Architecture centers around the domain.
- Clean Architecture enforces dependency rules.
- Application and Domain Services have distinct responsibilities.
- ACL protects domains from external complexity.
- Event Sourcing enables full audit history.

# End of Volume 02 Chapters 11–15
