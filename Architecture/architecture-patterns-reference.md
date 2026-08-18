# Software Architecture & Design Patterns — Reference Guide

A consolidated reference of principles, design patterns, and enterprise architecture frameworks — each with details, a real-world example, and a short code snippet.

---

## 1. SOLID Principles

### Single Responsibility
One class/module should have only one job or reason to change.
```python
class Logger:
    def log(self, message):
        print(f"[LOG] {message}")
```

### Open/Closed
Open for extension, closed for modification.
```python
class PaymentMethod:
    def pay(self, amount): pass

class CreditCard(PaymentMethod):
    def pay(self, amount):
        print(f"Paid {amount} via Credit Card")

class PayPal(PaymentMethod):   # new method added without touching existing code
    def pay(self, amount):
        print(f"Paid {amount} via PayPal")
```

### Liskov Substitution
Subtypes must be substitutable for their base types.
```python
class Rectangle:
    def area(self, w, h): return w * h

class Square(Rectangle):
    def area(self, side, _=None): return side * side  # behaves like Rectangle
```

### Interface Segregation
Small, specific interfaces instead of large general ones.
```python
class IPrinter:
    def print(self, doc): pass

class IScanner:
    def scan(self, doc): pass  # separate from printing concerns
```

### Dependency Inversion
Depend on abstractions, not concrete classes.
```python
class DatabaseInterface:
    def save(self, data): pass

class Service:
    def __init__(self, db: DatabaseInterface):
        self.db = db  # depends on abstraction, not a specific DB
```

---

## 2. DRY & KISS

### DRY (Don't Repeat Yourself)
Centralize reusable logic.
```python
def format_date(date):
    return date.strftime("%Y-%m-%d")
# used everywhere instead of re-writing formatting logic
```

### KISS (Keep It Simple, Stupid)
Avoid over-engineering.
```python
# KISS
total = sum(numbers)

# Over-engineered version avoided:
# total = reduce(lambda a,b: a+b, map(lambda x: x, numbers))
```

---

## 3. GoF (Gang of Four) Design Patterns

### 3.1 Creational Patterns

**Factory Method** — Creates objects without specifying the exact class.
```python
class ShapeFactory:
    def create(self, kind):
        return {"circle": Circle(), "square": Square()}[kind]
```

**Singleton** — Ensures only one instance exists.
```python
class ConfigManager:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

**Builder** — Constructs complex objects step by step.
```python
class ReportBuilder:
    def __init__(self):
        self.report = {}
    def add_title(self, title):
        self.report["title"] = title
        return self
    def add_body(self, body):
        self.report["body"] = body
        return self
```

### 3.2 Structural Patterns

**Adapter** — Bridges incompatible interfaces.
```python
class LegacyAPI:
    def old_method(self): return "legacy data"

class Adapter:
    def __init__(self, legacy): self.legacy = legacy
    def new_method(self): return self.legacy.old_method()
```

**Decorator** — Adds behavior dynamically.
```python
def with_logging(func):
    def wrapper(*args):
        print(f"Calling {func.__name__}")
        return func(*args)
    return wrapper

@with_logging
def process(): print("processing...")
```

**Composite** — Tree structures treated uniformly.
```python
class File:
    def show(self): print("File")

class Folder:
    def __init__(self): self.children = []
    def show(self):
        for child in self.children: child.show()
```

### 3.3 Behavioral Patterns

**Observer** — Notifies dependents on state change.
```python
class ChatRoom:
    def __init__(self): self.subscribers = []
    def notify(self, msg):
        for s in self.subscribers: s.update(msg)
```

**Strategy** — Swaps algorithms at runtime.
```python
class SortContext:
    def __init__(self, strategy): self.strategy = strategy
    def sort(self, data): return self.strategy(data)

SortContext(sorted).sort([3, 1, 2])
```

**Command** — Encapsulates a request as an object.
```python
class Command:
    def execute(self): pass

class SaveCommand(Command):
    def execute(self): print("Saving...")  # enables undo/redo stacks
```

**Chain of Responsibility** — Passes request along handlers.
```python
class Handler:
    def __init__(self, next_handler=None): self.next = next_handler
    def handle(self, request):
        if self.next: return self.next.handle(request)
```

---

## 4. Domain-Driven Design (DDD) Patterns

**Entity** — Object defined by identity.
```python
class User:
    def __init__(self, user_id, name):
        self.id = user_id   # identity matters, not just attributes
        self.name = name
```

**Value Object** — Immutable, defined by attributes.
```python
class Money:
    def __init__(self, amount, currency):
        self.amount = amount
        self.currency = currency  # equality based on values, not identity
```

**Aggregate** — Cluster of entities/values as one unit.
```python
class Order:
    def __init__(self):
        self.line_items = []  # Order is the aggregate root
```

**Repository** — Data access abstraction.
```python
class UserRepository:
    def get(self, user_id): ...
    def save(self, user): ...
```

**Service** — Domain logic outside entities.
```python
class PaymentService:
    def charge(self, order, card): ...
```

**Factory** — Creates aggregates.
```python
class OrderFactory:
    def create(self, items): return Order(items)
```

**Domain Event** — Something happened.
```python
class OrderPlaced:
    def __init__(self, order_id): self.order_id = order_id
```

**Bounded Context** — Clear sub-domain boundary (Billing vs Shipping modules with their own models).

**Anti-Corruption Layer** — Protects domain from external models.
```python
class ExternalApiAdapter:
    def to_domain_model(self, external_data):
        return Order(id=external_data["order_ref"])
```

---

## 5. Other Architectural Patterns

**CQRS** — Split read/write models.
```python
class OrderCommandService:
    def create_order(self, data): ...   # writes

class OrderQueryService:
    def get_order(self, id): ...        # reads
```

**Event Sourcing** — Store state as events.
```python
events = [{"type": "OrderPlaced"}, {"type": "OrderShipped"}]
state = reduce(apply_event, events, initial_state)
```

**Saga Pattern** — Manage distributed transactions.
```python
def book_trip():
    try:
        book_flight()
        book_hotel()
    except Exception:
        cancel_flight()  # compensating action
```

**Circuit Breaker** — Prevent cascading failures.
```python
if failure_count > threshold:
    raise CircuitOpenError("Service unavailable, skipping call")
```

**Mediator** — Centralize communication.
```python
class ChatMediator:
    def send(self, message, sender):
        for user in self.users:
            if user != sender: user.receive(message)
```

**Proxy** — Control access.
```python
class SecurityProxy:
    def __init__(self, real_service): self.real_service = real_service
    def request(self, user):
        if user.is_authorized: return self.real_service.request()
```

---

## 6. Enterprise Architecture Frameworks

These are process/organizational frameworks rather than code-level patterns, so a small diagram fits better than a snippet.

**TOGAF** — ADM cycle for enterprise IT roadmap planning.
```
Preliminary → Architecture Vision → Business Arch → Info Systems Arch
   → Technology Arch → Opportunities & Solutions → Migration Planning
   → Implementation Governance → Change Management → (back to start)
```

**Zachman Framework** — Matrix of perspectives.
```
            Data     Function   Network   People   Time    Motivation
Planner      ...        ...       ...       ...     ...       ...
Owner        ...        ...       ...       ...     ...       ...
Designer     ...        ...       ...       ...     ...       ...
```

**FEAF** — Aligns US federal agency IT with mission.
```
Mission/Strategy → Business Architecture → Data Architecture
    → Application Architecture → Technology Architecture
```

**DoDAF** — Defense system interoperability views.
```
Operational View (what) ↔ Systems View (how) ↔ Technical View (standards)
```

**Gartner EA Framework** — Links IT investment to ROI.
```
Business Strategy → EA Roadmap → IT Investment Decisions → Business Outcomes
```

**ArchiMate** — Layers of enterprise modeling.
```
Business Layer   (processes, actors)
     ↓
Application Layer (services, components)
     ↓
Technology Layer  (infrastructure, devices)
```

---

## Summary

- **Principles (SOLID, DRY, KISS)** → Keep code clean and maintainable.
- **GoF Patterns** → Solve recurring object-oriented design problems.
- **DDD Patterns** → Model complex business domains accurately.
- **Other Architectural Patterns** → Handle scalability, reliability, and distributed system concerns.
- **Enterprise Frameworks** → Align IT systems and architecture with overall business strategy.
