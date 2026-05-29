# Volume 01 – Python Core & Advanced Python
# Chapters 16–20 Full Edition

## Chapter 16 – Iterators

### Learning Objectives
- Understand iteration mechanics
- Understand iterator protocol
- Build custom iterators

### Theory

An iterator is an object that returns items one at a time.

Python iteration uses:

- __iter__()
- __next__()

### Short Interview Answer

Iterators allow sequential access to elements without exposing underlying implementation.

### Example

```python
numbers = [1,2,3]

it = iter(numbers)

print(next(it))
```

### Detailed Interview Answer

An iterator maintains state and produces values until StopIteration is raised.

### Architecture Discussion

Iterators are useful for:

- Large datasets
- File processing
- Stream processing

### Interview Questions

1. What is an iterator?
2. Difference between iterable and iterator?
3. What is StopIteration?

---

## Chapter 17 – Iterator Protocol

### Learning Objectives

- Implement custom iterators
- Understand iteration internals

### Example

```python
class Counter:

    def __init__(self, limit):
        self.limit = limit
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):

        if self.current >= self.limit:
            raise StopIteration

        self.current += 1
        return self.current
```

### Short Interview Answer

Custom iterators implement __iter__ and __next__.

### Production Example

Streaming records from databases.

### Staff Engineer Question

How would you process billions of records efficiently?

---

## Chapter 18 – Generators

### Learning Objectives

- Understand yield
- Understand lazy execution

### Theory

Generators produce values on demand.

### Example

```python
def generate():
    yield 1
    yield 2
    yield 3
```

### Short Interview Answer

Generators use the yield keyword to produce values lazily.

### Detailed Interview Answer

Execution pauses at yield and resumes later.

### Benefits

- Reduced memory consumption
- Better scalability
- Streaming support

### Interview Questions

1. Generator vs Iterator?
2. yield vs return?
3. Why are generators memory efficient?

### Architecture Discussion

Generators are heavily used in:

- ETL
- Data Engineering
- AI Pipelines

---

## Chapter 19 – Generator Expressions & Lazy Evaluation

### Learning Objectives

- Understand lazy evaluation
- Optimize memory usage

### Example

```python
squares = (x*x for x in range(1000000))
```

### Comparison

List:

```python
[x*x for x in range(1000000)]
```

Generator:

```python
(x*x for x in range(1000000))
```

### Short Interview Answer

Generator expressions compute values only when needed.

### Architecture Discussion

Lazy evaluation is essential for:

- Large datasets
- Streaming APIs
- Big data systems

### Production Example

Processing 100GB files without loading everything into memory.

### Common Mistakes

Converting generators to lists immediately.

---

## Chapter 20 – Context Managers

### Learning Objectives

- Manage resources safely
- Understand with statement

### Theory

Context managers guarantee cleanup.

### Example

```python
with open("data.txt") as f:
    data = f.read()
```

### Short Interview Answer

Context managers automatically acquire and release resources.

### Detailed Interview Answer

They implement:

- __enter__
- __exit__

### Custom Context Manager

```python
class Resource:

    def __enter__(self):
        print("start")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("end")
```

### contextlib Example

```python
from contextlib import contextmanager

@contextmanager
def connection():
    yield
```

### Production Use Cases

- Database Connections
- Transactions
- Distributed Locks
- File Handling

### Interview Questions

1. What is a context manager?
2. Why use with?
3. What does __exit__ do?
4. What is contextlib?

### Architecture Discussion

Context managers improve:

- Reliability
- Resource management
- Error handling

### Staff Engineer Questions

1. How would you standardize transaction management?
2. How would you prevent resource leaks?

### Architect Questions

1. How would context managers help distributed systems?
2. How would you implement transaction boundaries?

---

# Coding Exercises

## Exercise 1

Create a Fibonacci generator.

### Solution

```python
def fibonacci():
    a, b = 0, 1

    while True:
        yield a
        a, b = b, a + b
```

---

## Exercise 2

Implement a custom iterator returning numbers from 1 to N.

---

## Exercise 3

Implement a custom context manager.

---

# Summary

Key Takeaways

- Iterators implement iteration protocol.
- Generators use yield.
- Lazy evaluation reduces memory usage.
- Context managers handle resources safely.
- These concepts are critical for scalable Python systems.

# End of Chapters 16–20
