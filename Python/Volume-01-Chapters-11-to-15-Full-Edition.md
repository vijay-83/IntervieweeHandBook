# Volume 01 – Python Core & Advanced Python
# Chapters 11–15 Full Edition

## Chapter 11 – Functions Deep Dive

### Learning Objectives
- Understand function internals
- Understand parameter passing
- Understand return values

### Theory

Functions are reusable units of execution.

```python
def add(a, b):
    return a + b
```

### Short Interview Answer

Functions encapsulate behavior and allow code reuse.

### Detailed Interview Answer

Functions in Python are first-class objects. They can be:
- Assigned to variables
- Passed as arguments
- Returned from functions

### Parameter Types

```python
def demo(a, b=10, *args, **kwargs):
    pass
```

### Architecture Discussion

Functions should:
- Have a single responsibility
- Be easy to test
- Avoid side effects

### Interview Questions

1. What are positional arguments?
2. What are keyword arguments?
3. What are default arguments?
4. What are *args and **kwargs?

---

## Chapter 12 – First-Class Functions

### Learning Objectives

- Understand functional programming concepts
- Understand callback design

### Theory

Functions are objects.

```python
def greet():
    return "hello"

x = greet
```

### Short Interview Answer

Python treats functions as first-class citizens.

### Detailed Interview Answer

Functions can be:

- Stored in variables
- Passed to other functions
- Returned from functions

### Production Example

FastAPI dependency injection relies heavily on callable objects.

### Architecture Discussion

First-class functions enable:
- Middleware
- Event handlers
- Plugin systems

### Interview Questions

1. What is a first-class function?
2. Why is it useful?

---

## Chapter 13 – Lambda Functions

### Learning Objectives

- Understand anonymous functions
- Understand functional programming utilities

### Example

```python
square = lambda x: x * x
```

### Short Interview Answer

Lambda creates small anonymous functions.

### Detailed Interview Answer

Lambda functions are commonly used with:

- map()
- filter()
- sorted()

Example:

```python
numbers = [3, 1, 2]

sorted(numbers, key=lambda x: x)
```

### Common Mistakes

Using lambda for complex business logic.

### Interview Questions

1. Lambda vs normal functions?
2. When should lambda be avoided?

---

## Chapter 14 – Closures

### Learning Objectives

- Understand lexical scope
- Understand state retention

### Example

```python
def outer(x):

    def inner():
        return x

    return inner
```

### Short Interview Answer

A closure remembers variables from its enclosing scope.

### Detailed Interview Answer

Even after outer() finishes execution, the inner function still retains access to variables.

### Production Example

Closures are useful for:

- Configuration wrappers
- Middleware
- Factory functions

### Architecture Discussion

Closures allow creation of highly reusable framework components.

### Interview Questions

1. What is a closure?
2. What problem does a closure solve?
3. Difference between closure and class?

---

## Chapter 15 – Decorators

### Learning Objectives

- Understand decorators
- Understand cross-cutting concerns

### Theory

Decorators modify behavior without changing original code.

### Example

```python
def logger(func):

    def wrapper(*args, **kwargs):
        print("executing")
        return func(*args, **kwargs)

    return wrapper
```

### Usage

```python
@logger
def process():
    pass
```

### Short Interview Answer

Decorators extend behavior of functions and classes.

### Detailed Interview Answer

Decorators are functions that accept another function and return a modified function.

### Real Production Uses

- Authentication
- Authorization
- Logging
- Tracing
- Metrics
- Retry Logic

### FastAPI Example

```python
@app.get("/users")
async def users():
    return []
```

FastAPI uses decorators extensively.

### Parameterized Decorators

```python
def repeat(times):

    def decorator(func):

        def wrapper():
            for _ in range(times):
                func()

        return wrapper

    return decorator
```

### Architecture Discussion

Decorators help implement platform-wide concerns consistently.

### Common Mistakes

- Deep decorator chains
- Hidden side effects
- Difficult debugging

### Interview Questions

1. What is a decorator?
2. How do decorators work internally?
3. What is functools.wraps?
4. What are parameterized decorators?

### Staff Engineer Questions

1. How would you standardize logging across hundreds of APIs?
2. How would decorators help implement observability?

### Architect Questions

1. When should decorators not be used?
2. What are the performance implications?

### Coding Exercise

Create a decorator that measures execution time.

### Solution

```python
import time

def timer(func):

    def wrapper(*args, **kwargs):

        start = time.time()

        result = func(*args, **kwargs)

        print(time.time() - start)

        return result

    return wrapper
```

---

# Summary

Key Takeaways

- Functions are first-class objects.
- Lambdas support concise functional programming.
- Closures retain state.
- Decorators extend behavior.
- FastAPI heavily uses decorators.
- Decorators are ideal for cross-cutting concerns.

# End of Chapters 11–15
