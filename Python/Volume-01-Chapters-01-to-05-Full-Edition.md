# Volume 01 – Python Core & Advanced Python
# Chapters 1–5 Full Edition

## Chapter 1 – Python Execution Model

### Learning Objectives
- Understand how Python executes code
- Understand compilation and interpretation
- Understand bytecode generation

### Theory
Python is often called an interpreted language, but internally it performs both compilation and interpretation.

Execution Flow:

Source Code -> AST -> Bytecode -> Python Virtual Machine

### Short Interview Answer
Python compiles source code into bytecode and executes it through the Python Virtual Machine (PVM).

### Detailed Interview Answer
When Python executes a program:
1. Parses source code.
2. Builds an Abstract Syntax Tree (AST).
3. Compiles AST into bytecode.
4. Executes bytecode in the Python Virtual Machine.

### Architecture Discussion
Understanding startup cost is important for:
- FastAPI applications
- Serverless functions
- Microservices

### Code Example

```python
print("Hello World")
```

### Common Mistakes
- Assuming Python executes source code directly.
- Confusing bytecode with machine code.

### Interview Questions
1. Is Python compiled or interpreted?
2. What is bytecode?
3. What is AST?
4. What is PVM?

### Summary
Python uses both compilation and interpretation.

---

## Chapter 2 – Bytecode & Python Virtual Machine

### Learning Objectives
- Understand bytecode
- Understand PVM

### Theory
Bytecode is an intermediate representation generated from Python source code.

### Short Interview Answer
Bytecode is executed by the Python Virtual Machine.

### Detailed Interview Answer
Python compiles source code into platform-independent bytecode which is later executed by the interpreter.

### Code Example

```python
import dis

def add(a,b):
    return a+b

dis.dis(add)
```

### Architecture Discussion
Bytecode caching improves startup performance.

### Interview Questions
1. What is bytecode?
2. Where are .pyc files stored?
3. Why use bytecode?

### Summary
Bytecode improves portability and execution efficiency.

---

## Chapter 3 – CPython Architecture

### Learning Objectives
- Understand CPython internals
- Compare implementations

### Components
- Parser
- Compiler
- Interpreter
- Memory Manager

### Short Interview Answer
CPython is the reference implementation of Python written in C.

### Detailed Interview Answer
CPython handles parsing, bytecode execution, memory management, and garbage collection.

### CPython vs PyPy

| Feature | CPython | PyPy |
|----------|----------|----------|
| Speed | Standard | Faster for some workloads |
| JIT | No | Yes |
| Compatibility | Excellent | High |

### Architecture Discussion
PyPy may improve long-running workloads but compatibility should be validated.

### Interview Questions
1. What is CPython?
2. What is PyPy?
3. Why does PyPy perform better?

### Summary
CPython is the standard implementation used in production.

---

## Chapter 4 – Memory Management

### Learning Objectives
- Understand memory allocation
- Understand object lifecycle

### Theory
Python manages memory using:
- Private Heap
- Reference Counting
- Garbage Collection

### Short Interview Answer
Python manages memory automatically through reference counting and garbage collection.

### Detailed Interview Answer
Every Python object maintains metadata including a reference count. Objects are cleaned when references disappear.

### Code Example

```python
a = []
b = a
```

Reference count becomes 2.

### Architecture Discussion
Memory management directly impacts:
- APIs
- Data processing systems
- AI workloads

### Common Mistakes
- Holding global references unnecessarily.
- Keeping large objects in memory.

### Interview Questions
1. How does Python manage memory?
2. What is a memory leak?
3. What tools identify memory leaks?

### Summary
Understanding memory behavior is critical for scalable systems.

---

## Chapter 5 – Garbage Collection

### Learning Objectives
- Understand cyclic references
- Understand GC internals

### Theory
Reference counting cannot handle cyclic references.

### Example

```python
class Node:
    pass

a = Node()
a.self = a
```

### Short Interview Answer
Garbage collection removes cyclic references that reference counting cannot clean up.

### Detailed Interview Answer
Python periodically scans objects and identifies unreachable cycles.

### Tools
- gc
- tracemalloc
- memory_profiler

### Architecture Discussion
GC behavior becomes important in:
- Long-running services
- ETL pipelines
- Machine learning systems

### Interview Questions
1. Why does Python need GC?
2. What are cyclic references?
3. How do you debug memory growth?

### Staff Engineer Questions
1. How would you investigate memory growth in production?
2. How would you reduce memory pressure in microservices?

### Solution Architect Questions
1. How does memory management affect cloud costs?
2. How would you monitor memory across hundreds of services?

### Summary
Garbage collection complements reference counting and prevents memory leaks from cyclic references.

---

# End of Volume 01 Chapters 1–5 Full Edition
