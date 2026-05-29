# Volume 01 – Python Core & Advanced Python
# Chapters 6–10 Full Edition

## Chapter 6 – Variables & References

### Learning Objectives
- Understand object references
- Understand assignment behavior
- Understand identity vs equality

### Theory

Python variables do not store objects directly. Variables store references to objects.

```python
a = [1, 2, 3]
b = a
```

Both variables point to the same object in memory.

### Short Interview Answer

Python variables are references to objects, not containers that store the object value itself.

### Detailed Interview Answer

When an assignment occurs, Python binds a name to an object.

```python
x = 10
y = x
```

Both variables reference the same integer object until one variable is rebound.

### Identity vs Equality

```python
a = [1,2]
b = [1,2]

a == b   # True
a is b   # False
```

`==` compares values.

`is` compares object identity.

### Architecture Discussion

Understanding references is critical when:

- Sharing state between services
- Building caches
- Passing objects through layers

### Common Mistakes

- Using mutable default arguments
- Assuming assignment creates copies

### Interview Questions

1. Difference between `is` and `==`?
2. What happens during assignment?
3. Explain object identity.

### Exercise

Explain the output:

```python
a = [1]
b = a
b.append(2)
print(a)
```

### Solution

Both variables reference the same list.

---

## Chapter 7 – Mutable vs Immutable Objects

### Learning Objectives

- Understand mutability
- Understand side effects
- Understand thread-safety implications

### Mutable Types

- list
- dict
- set

### Immutable Types

- int
- float
- str
- tuple

### Short Interview Answer

Mutable objects can change after creation. Immutable objects cannot.

### Detailed Interview Answer

When modifying a mutable object, Python changes the existing object.

When modifying an immutable object, Python creates a new object.

Example:

```python
a = "hello"
a += " world"
```

A new string object is created.

### Architecture Discussion

Immutable objects are preferred because:

- Easier reasoning
- Safer concurrency
- Better caching behavior

### Production Example

Configuration objects are often immutable to prevent accidental modification.

### Common Interview Trap

Question:

Why are strings immutable?

Expected Discussion:

- Performance
- Hashing
- Thread safety
- Dictionary keys

### Staff Engineer Question

How would you design immutable domain models?

---

## Chapter 8 – Namespaces

### Learning Objectives

- Understand namespace concepts
- Understand name resolution

### Theory

A namespace is a mapping between names and objects.

Examples:

- Built-in Namespace
- Global Namespace
- Local Namespace

### Example

```python
x = 100

def test():
    y = 200
```

`x` belongs to the global namespace.

`y` belongs to the local namespace.

### Short Interview Answer

Namespaces prevent naming collisions by providing scope boundaries.

### Architecture Discussion

Namespaces improve maintainability in large applications with hundreds of modules.

### Common Mistakes

- Overusing global variables
- Accidentally shadowing names

### Interview Questions

1. What is a namespace?
2. What namespaces exist in Python?
3. Why are namespaces useful?

---

## Chapter 9 – LEGB Rule

### Learning Objectives

- Understand scope resolution
- Understand closures

### Theory

Python resolves names using LEGB:

L = Local

E = Enclosing

G = Global

B = Built-In

### Example

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        print(x)

    inner()
```

Output:

```text
enclosing
```

### Short Interview Answer

Python searches for names in Local, Enclosing, Global, and Built-in scopes.

### Detailed Interview Answer

The interpreter checks scopes in order and stops when it finds a matching name.

### Architecture Discussion

Closures and dependency injection frameworks rely heavily on scope resolution.

### Interview Questions

1. Explain LEGB.
2. What is a closure?
3. What is `nonlocal`?

### Exercise

Predict the output of nested scope examples.

---

## Chapter 10 – Import System Internals

### Learning Objectives

- Understand import mechanics
- Understand module caching
- Understand startup performance

### Import Flow

1. Check `sys.modules`
2. Search module paths
3. Compile if needed
4. Execute module
5. Cache module

### Example

```python
import math
```

Python first checks whether the module is already loaded.

### Short Interview Answer

Python loads modules once and caches them in `sys.modules`.

### Detailed Interview Answer

Repeated imports usually return the cached module rather than re-executing code.

### Architecture Discussion

Poor import design can increase:

- API startup time
- Container startup time
- Serverless cold starts

### Production Example

Large FastAPI projects often suffer from slow startup due to expensive imports.

### Common Mistakes

- Running heavy logic at import time
- Circular imports

### Interview Questions

1. What is `sys.modules`?
2. Why are circular imports problematic?
3. How does Python find modules?

### Staff Engineer Question

How would you reduce startup time across hundreds of Python services?

### Architect Question

How would you optimize cold-start performance for serverless workloads?

---

# Summary

Key Takeaways

- Variables store references.
- Identity differs from equality.
- Immutable objects improve safety.
- Namespaces prevent collisions.
- LEGB controls scope resolution.
- Imports are cached in `sys.modules`.
- Import design affects production performance.

# End of Chapters 6–10
