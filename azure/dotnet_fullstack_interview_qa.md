# .NET Full-Stack Interview Q&A (All Levels)

**Role:** Senior .NET Developer (C#, .NET Core, REST APIs, Microservices, System Design, DSA, React/Angular)
**Experience target:** 7-15 years

**Level legend**
- 🟢 **Basic**: fundamentals every candidate must know
- 🟡 **Intermediate**: practical, hands-on knowledge
- 🟠 **Advanced**: senior-level depth and trade-offs
- 🔴 **Expert / Architect**: design decisions, scale, leadership

## Table of Contents
1. [C# Language](#1-c-language)
2. [.NET Core Runtime and Internals](#2-net-core-runtime-and-internals)
3. [ASP.NET Core and REST APIs](#3-aspnet-core-and-rest-apis)
4. [Entity Framework Core, SQL and Data Access](#4-entity-framework-core-sql-and-data-access)
5. [Microservices and Distributed Systems](#5-microservices-and-distributed-systems)
6. [SOLID, Design Patterns and Clean Code](#6-solid-design-patterns-and-clean-code)
7. [Data Structures and Algorithms](#7-data-structures-and-algorithms)
8. [System Design and Architecture](#8-system-design-and-architecture)
9. [Performance and Troubleshooting](#9-performance-and-troubleshooting)
10. [Security](#10-security)
11. [React](#11-react)
12. [Angular](#12-angular)
13. [Testing, CI/CD and DevOps](#13-testing-cicd-and-devops)
14. [Behavioral and Ownership](#14-behavioral-and-ownership)
15. [Rapid-Fire Round](#15-rapid-fire-round)
16. [Preparation Checklist](#16-preparation-checklist)

---

## 1. C# Language

### 🟢 Basic

**Q1. What is the difference between value types and reference types?**
Value types (`int`, `struct`, `enum`, `bool`) hold their data directly and are copied on assignment. Reference types (`class`, `string`, arrays, delegates) hold a reference to heap data, and assignment copies the reference.

**Q2. What is the difference between `==` and `.Equals()`?**
`==` is an operator (reference equality for classes by default, value equality for primitives or when overloaded). `.Equals()` is a virtual method that can be overridden for custom equality. `string` overloads both for value comparison.

**Q3. What are `abstract` classes vs `interfaces`?**
An abstract class can hold state, constructors and partial implementations, and supports single inheritance. An interface defines a contract (C# 8+ allows default implementations), and a class can implement many. Use abstract for a shared base with state, interfaces for capabilities.

**Q4. Explain `const` vs `readonly` vs `static`.**
`const` is a compile-time constant, implicitly static. `readonly` is assigned at declaration or in the constructor (runtime constant). `static` belongs to the type rather than an instance.

**Q5. What are `ref`, `out` and `in` parameters?**
`ref` passes by reference (must be initialized). `out` passes by reference and must be assigned inside the method. `in` passes by reference as read-only, which avoids copying large structs.

**Q6. What is the difference between `string` and `StringBuilder`?**
`string` is immutable, so every modification creates a new object. `StringBuilder` is mutable and efficient for repeated concatenation in loops.

**Q7. What are nullable types and the `?.`, `??` operators?**
`int?` allows null for value types. `?.` is null-conditional (short-circuits on null). `??` returns the right side when the left is null. `??=` assigns only if null. Nullable reference types (C# 8+) add compile-time null-safety warnings.

### 🟡 Intermediate

**Q8. Explain `IEnumerable` vs `ICollection` vs `IList` vs `IQueryable`.**
`IEnumerable` is forward-only iteration. `ICollection` adds `Count`, `Add` and `Remove`. `IList` adds indexing. `IQueryable` builds an expression tree translated by a provider (EF Core) to SQL. Calling `.ToList()` early on an `IQueryable` loads everything into memory and filters in the app.

**Q9. What is LINQ deferred execution?**
Most LINQ operators (`Where`, `Select`) don't run until enumerated (`foreach`, `ToList`, `Count`). Pitfalls include multiple enumeration (re-running queries) and closures capturing changed variables. Immediate operators are `ToList`, `ToArray`, `First`, `Count`, `Sum`.

**Q10. Explain delegates, `Func`, `Action`, `Predicate` and events.**
A delegate is a type-safe function pointer. `Func<T,TResult>` returns a value, `Action<T>` returns void, and `Predicate<T>` returns bool. An event is a delegate with restricted access, so only the owner can raise it and others can only subscribe or unsubscribe.

**Q11. What are generics, and what constraints exist?**
Generics give type safety and avoid boxing. Constraints include `where T : class`, `struct`, `new()`, a base class or interface, and `notnull`. Covariance (`out`) and contravariance (`in`) apply to interfaces and delegates, for example `IEnumerable<out T>`.

**Q12. What are extension methods, and when would you avoid them?**
Static methods in static classes with `this` on the first parameter, giving instance-like syntax. Avoid them for core domain behavior, and watch for namespace pollution and hidden dependencies.

**Q13. What is the difference between `Dispose` and a finalizer? Show the `IDisposable` pattern.**
A finalizer (`~Class`) is called non-deterministically by the GC and costs performance. `IDisposable.Dispose()` releases resources deterministically via `using`. Only implement the full dispose pattern when you own unmanaged resources directly, and call `GC.SuppressFinalize(this)`.

**Q14. What is the difference between `Task` and `Thread`?**
A `Thread` is an OS-level execution unit. A `Task` is an abstraction for an asynchronous operation scheduled on the thread pool. Tasks compose with `await`, cancellation, continuations and exception propagation.

**Q15. What are records, and when should you use them?**
Records (C# 9+) are reference types (or `record struct`) with value-based equality, immutability via `init`, `with` expressions and built-in `ToString`. Use them for DTOs, messages and value objects.

### 🟠 Advanced

**Q16. Explain `async`/`await` internals and common pitfalls.**
The compiler rewrites an async method into a state machine. `await` captures the `SynchronizationContext` (not in ASP.NET Core) and resumes when the awaited task completes.
Pitfalls:
- Blocking with `.Result` or `.Wait()` (deadlocks, thread-pool starvation).
- `async void` outside event handlers.
- Missing `CancellationToken`.
- `Task.Run` on I/O-bound work in ASP.NET.
- Not awaiting tasks (fire and forget losing exceptions).
- Needless `async` wrappers on pass-through methods.

**Q17. `Task` vs `ValueTask`?**
`ValueTask` avoids heap allocation when the result is often synchronously available (cache hits). It must be awaited once and never `.Result`'d concurrently. Use it only where profiling shows allocation pressure.

**Q18. What is `Span<T>`, `Memory<T>` and `ref struct`? Why do they matter?**
`Span<T>` is a stack-only window over contiguous memory (arrays, stackalloc, native) with zero allocations, ideal for parsing. It cannot be used across `await` or stored on the heap. `Memory<T>` is the heap-friendly counterpart usable in async code.

**Q19. How does the .NET garbage collector work?**
Generational GC (Gen 0/1/2) plus the Large Object Heap (85 KB and over, collected with Gen 2). It is a mark-and-compact collector. Workstation vs Server GC, and concurrent (background) GC, trade throughput against latency. Reduce pressure by pooling (`ArrayPool<T>`, `ObjectPool`), avoiding boxing and large temporary allocations, and using `struct` carefully.

**Q20. What causes memory leaks in managed code?**
Common causes:
- Event handlers never unsubscribed.
- Static collections or caches without eviction.
- Captured closures.
- Undisposed `HttpClient`, streams or timers.
- Long-lived `DbContext`.
- Thread-local or pinned objects.
Diagnose with dotMemory, `dotnet-gcdump` or `dotnet-dump`.

**Q21. Explain `IEnumerable` + `yield return`. What are the risks?**
`yield` generates a lazy state-machine iterator. Risks include deferred exceptions (argument validation runs on first `MoveNext`), re-enumeration cost, and holding resources open longer than expected.

**Q22. What is pattern matching in modern C#?**
Type, property, relational, `switch` expression, list and positional patterns, for example:
```csharp
var fee = order switch
{
    { Total: > 1000, IsPremium: true } => 0m,
    { Total: > 1000 }                  => 5m,
    _                                  => 10m
};
```

### 🔴 Expert

**Q23. How would you design a thread-safe, high-throughput in-memory cache?**
Use `ConcurrentDictionary` with `Lazy<Task<T>>` values (prevents duplicate factory calls), a sliding or absolute expiry, size limits with an eviction policy (LRU), and cache stampede protection (per-key lock or `SemaphoreSlim`). Mention `IMemoryCache` and `HybridCache` (.NET 9) and when to move to Redis.

**Q24. Explain memory model concerns: `volatile`, `lock`, `Interlocked`, `SemaphoreSlim`, `Channel<T>`.**
`lock` gives mutual exclusion, `Interlocked` gives atomic operations, `volatile` prevents certain reorderings (not atomicity), `SemaphoreSlim` supports async throttling (`WaitAsync`), and `Channel<T>` is an async producer/consumer pipeline. Prefer immutability and message passing over shared mutable state.

---

## 2. .NET Core Runtime and Internals

### 🟢 Basic

**Q25. What is .NET Core / modern .NET? How is it different from .NET Framework?**
Cross-platform, open source, modular, high-performance, side-by-side versioning, built-in DI and cloud/container friendly. .NET Framework is Windows-only and legacy. Since .NET 5 the product is simply ".NET" (LTS: 6, 8, 10).

**Q26. What is the difference between SDK, runtime and `dotnet` CLI?**
The SDK builds and publishes (compiler, MSBuild, CLI). The runtime (CLR plus base libraries) only runs apps. The CLI (`dotnet new/build/run/test/publish`) drives both.

**Q27. What is Dependency Injection, and what are the service lifetimes?**
DI supplies dependencies from outside (constructor injection).
- **Transient:** new per resolve.
- **Scoped:** one per request (scope).
- **Singleton:** one per app lifetime.

### 🟡 Intermediate

**Q28. Explain the middleware pipeline and write a custom middleware.**
```csharp
public class RequestTimingMiddleware(RequestDelegate next, ILogger<RequestTimingMiddleware> log)
{
    public async Task InvokeAsync(HttpContext ctx)
    {
        var sw = Stopwatch.StartNew();
        await next(ctx);
        log.LogInformation("{Path} took {Ms} ms", ctx.Request.Path, sw.ElapsedMilliseconds);
    }
}
// app.UseMiddleware<RequestTimingMiddleware>();
```
Order matters: exception handling → HTTPS → static files → routing → CORS → authentication → authorization → endpoints.

**Q29. How does configuration work? (`appsettings`, environment, Options pattern)**
Layered providers: `appsettings.json`, `appsettings.{Env}.json`, user secrets, environment variables, command line, Key Vault (later overrides earlier). Use `IOptions<T>` (singleton snapshot), `IOptionsSnapshot<T>` (per request) and `IOptionsMonitor<T>` (live change notifications), with validation (`ValidateDataAnnotations().ValidateOnStart()`).

**Q30. What is `IHttpClientFactory` and why use it?**
It manages `HttpMessageHandler` lifetimes, which avoids socket exhaustion from `new HttpClient()` per call and DNS staleness from a static client. It supports named and typed clients and Polly integration.

**Q31. What are hosted services / `BackgroundService`?**
Long-running background work in the host: `IHostedService` (Start/Stop) or `BackgroundService` (`ExecuteAsync`). Use it for queue consumers and schedulers. Handle exceptions and cancellation, and create scopes manually for scoped dependencies.

### 🟠 Advanced

**Q32. Captive dependency problem?**
A singleton holding a scoped or transient service keeps it alive past its intended lifetime (stale `DbContext`, thread-safety bugs). Enable `ValidateScopes` in development, and use `IServiceScopeFactory` inside singletons.

**Q33. Kestrel, reverse proxies and hosting models?**
Kestrel is the cross-platform web server. Typically it runs behind IIS, Nginx, YARP or a cloud load balancer (forwarded headers middleware is required for correct scheme and IP). In-process vs out-of-process IIS hosting, and Docker/Kubernetes for container hosting.

**Q34. What is Native AOT / ReadyToRun / trimming?**
ReadyToRun pre-compiles IL to improve startup. Native AOT compiles to a self-contained native binary with the fastest startup and smallest footprint, but with limits on reflection and dynamic code (Minimal APIs and source generators are AOT-friendly). Trimming removes unused code.

**Q35. What are source generators and how do they help?**
Compile-time code generation (logging `[LoggerMessage]`, `System.Text.Json` source-gen, regex) that removes reflection and improves startup and throughput.

---

## 3. ASP.NET Core and REST APIs

### 🟢 Basic

**Q36. What are the main HTTP verbs and status codes?**
GET (read), POST (create), PUT (replace), PATCH (partial update), DELETE. Codes: 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500, 503.

**Q37. What are the principles of REST?**
Client-server, stateless, cacheable, uniform interface (resources, representations), layered system, and optional code on demand.

**Q38. Controllers vs Minimal APIs?**
Minimal APIs: low ceremony and fast, good for small services. Controllers: filters, conventions, model binding, richer ecosystem. Both share the same pipeline.

### 🟡 Intermediate

**Q39. How do you design a good REST API?**
- Resource-oriented nouns (`/orders/{id}/items`).
- Proper verbs and status codes.
- Idempotency (PUT/DELETE, idempotency keys for POST).
- Pagination, filtering, sorting (`?page=2&pageSize=20`).
- Versioning (URL `/v1/`, header or media type).
- Consistent errors with `ProblemDetails` (RFC 7807).
- OpenAPI/Swagger docs.
- Rate limiting and caching headers (`ETag`, `Cache-Control`).

**Q40. Model binding and validation?**
`[FromBody]`, `[FromQuery]`, `[FromRoute]`, `[FromHeader]`. Validate with data annotations, FluentValidation, or `[ApiController]` for automatic 400 responses. Never trust client input, and keep request DTOs separate from domain entities.

**Q41. What are filters in ASP.NET Core?**
Authorization → Resource → Action → Exception → Result filters. Use them for cross-cutting concerns (logging, validation, caching). Middleware runs for all requests, while filters run only inside MVC and endpoints.

**Q42. How do you implement global exception handling?**
`app.UseExceptionHandler()` or `IExceptionHandler` (.NET 8+), mapping exception types to `ProblemDetails`, correlating with a trace ID, logging once, and never returning stack traces.

**Q43. PUT vs PATCH vs POST?**
POST creates (not idempotent). PUT replaces the full resource (idempotent). PATCH applies a partial update (JSON Patch or Merge Patch).

### 🟠 Advanced

**Q44. How do you version APIs without breaking clients?**
Prefer additive, backward-compatible changes. Use `Asp.Versioning` with URL or header versioning, deprecate with `Sunset`/`Deprecation` headers, run versions side by side, and track usage before removal.

**Q45. How do you handle large payloads, file uploads and streaming?**
Use streaming (`IAsyncEnumerable<T>`, `Stream`), avoid buffering the entire file in memory, set request size limits, use pre-signed URLs to blob storage for big uploads, and use chunked or resumable uploads.

**Q46. REST vs gRPC vs GraphQL?**
REST suits public APIs, is cache-friendly and universally supported. gRPC (HTTP/2 and Protobuf) is fast and strongly typed, ideal for internal service-to-service calls and streaming. GraphQL lets clients shape responses and avoids over- and under-fetching, but adds complexity in caching, authorization and N+1.

**Q47. How do you implement idempotency for POST (e.g., payments)?**
The client sends an `Idempotency-Key` header. The server stores the key with the response (or in-progress state) in a DB or Redis, and replays the stored result on retry. Use a unique constraint to prevent races.

**Q48. How do you implement rate limiting?**
Built-in `Microsoft.AspNetCore.RateLimiting` (fixed window, sliding window, token bucket, concurrency), partitioned by user or IP. At scale use a distributed limiter (Redis) or enforce at the API gateway. Return `429` with `Retry-After`.

### 🔴 Expert

**Q49. How would you design a public API platform for external partners?**
Cover the gateway (auth, throttling, quotas), versioning and deprecation policy, OAuth2 client credentials, webhooks with retries and signatures, an SDK and OpenAPI contract, sandbox environments, observability per consumer, SLAs, and backward compatibility testing.

---

## 4. Entity Framework Core, SQL and Data Access

### 🟢 Basic

**Q50. What is EF Core, and what are code-first vs database-first?**
EF Core is an ORM that maps C# classes to tables. Code-first defines models in code and uses migrations. Database-first scaffolds models from an existing schema (`dotnet ef dbcontext scaffold`).

**Q51. Explain `DbContext` and `DbSet`.**
`DbContext` is a unit of work with change tracking and a session to the DB. `DbSet<T>` represents a queryable table. A context should be short-lived (scoped per request) and is not thread-safe.

**Q52. What are the SQL JOIN types and when do you use each?**
INNER (matching rows), LEFT/RIGHT (all from one side), FULL (all from both), CROSS (cartesian product), SELF (a table joined to itself).

### 🟡 Intermediate

**Q53. Eager, lazy and explicit loading?**
Eager: `Include/ThenInclude`. Lazy: proxies load on access (risk of N+1). Explicit: `Entry(x).Collection(...).Load()`. Prefer projection (`Select`) for read queries.

**Q54. What is the N+1 problem and how do you fix it?**
One query for the parent plus one query per child. Fix with `Include`, projection, `AsSplitQuery`, or batching. Verify via query logging.

**Q55. Tracking vs `AsNoTracking`?**
Use `AsNoTracking()` for read-only queries to reduce memory and CPU. Use tracking only when you intend to update.

**Q56. SQL indexing basics?**
Clustered (data order, one per table) vs non-clustered. Covering indexes (`INCLUDE`), composite index column order, selectivity, and the write-cost trade-off. Read execution plans to spot scans and key lookups.

**Q57. What are transactions and isolation levels?**
ACID. Isolation levels: Read Uncommitted, Read Committed (default in SQL Server), Repeatable Read, Serializable, Snapshot. They trade consistency (dirty, non-repeatable and phantom reads) against concurrency.

### 🟠 Advanced

**Q58. How do you handle concurrency in EF Core?**
Optimistic concurrency via `rowversion` or `[ConcurrencyCheck]`, catching `DbUpdateConcurrencyException` and resolving (client wins, store wins or merge). Pessimistic locking only for high-contention cases.

**Q59. EF Core vs Dapper?**
EF Core: productivity, migrations, LINQ, change tracking. Dapper: minimal overhead, hand-tuned SQL, great for reporting and hot paths. Many systems use EF Core for writes and Dapper for reads.

**Q60. Repository and Unit of Work over EF Core, is it needed?**
`DbContext` is already a UoW and `DbSet` is repository-like. Add a repository only to encapsulate complex queries, aid testing and hide persistence details. Avoid generic repositories that leak `IQueryable`.

**Q61. How do you do bulk operations and batching efficiently?**
`ExecuteUpdate/ExecuteDelete` (EF Core 7+), `AddRange` with batching, `SqlBulkCopy` or EFCore.BulkExtensions for large imports, and disabling auto-detect changes for large inserts.

**Q62. How do you manage schema migrations safely in production?**
Generate migration scripts (`dotnet ef migrations script --idempotent`), review them, use expand-contract (backward-compatible) changes for zero downtime, apply them in the pipeline (not at app start in multi-instance setups), and take backups with a rollback plan.

### 🔴 Expert

**Q63. How do you scale the data layer?**
Read replicas, caching, partitioning and sharding, CQRS read models, archiving cold data, choosing the right store per workload (polyglot persistence), connection pooling, and avoiding cross-shard transactions.

**Q64. Optimize a slow SQL query. What's your approach?**
Capture the actual execution plan, check for scans, key lookups, missing indexes and parameter sniffing, update statistics, rewrite non-sargable predicates, reduce selected columns, paginate with keyset instead of `OFFSET`, and measure before and after.

---

## 5. Microservices and Distributed Systems

### 🟢 Basic

**Q65. What is a microservice architecture?**
An application split into small, independently deployable services, each owning a business capability and its own data, communicating over the network.

**Q66. Monolith vs microservices: pros and cons?**
Monolith: simple deployment and debugging, in-process calls, but coupled scaling and releases. Microservices: independent scaling and deployment and team autonomy, but they add network latency, distributed data, operational complexity and observability needs.

### 🟡 Intermediate

**Q67. How do microservices communicate?**
Sync: REST, gRPC. Async: message brokers (RabbitMQ, Azure Service Bus, Kafka) using commands and events. Prefer async for decoupling and resilience.

**Q68. What is an API Gateway?**
A single entry point (YARP, Ocelot, Azure APIM, Kong) handling routing, authentication, rate limiting, aggregation and SSL termination. The BFF pattern provides one gateway per client type.

**Q69. What is service discovery and load balancing?**
Services find each other dynamically (Kubernetes DNS, Consul, Eureka). Load balancing can be client-side or server-side (round robin, least connections).

**Q70. Containerization basics: Docker and Kubernetes?**
A Dockerfile (multi-stage build) produces an image. Kubernetes handles deployments, services, ingress, config maps, secrets, HPA autoscaling and rolling updates. Add liveness and readiness probes.

### 🟠 Advanced

**Q71. How do you handle distributed transactions?**
Avoid 2PC. Use the **Saga** pattern, with choreography (event-driven) or orchestration (a coordinator, e.g. MassTransit state machines), and compensating actions on failure. Pair it with the **Outbox pattern** so the DB write and event publish are atomic.

**Q72. Explain the Outbox and Inbox patterns.**
Outbox: write the business change and an outgoing message in the same DB transaction, and a relay publishes it later, giving at-least-once delivery. Inbox: record processed message IDs so duplicate deliveries are ignored (idempotent consumer).

**Q73. How do you build resilient services?**
Polly or `Microsoft.Extensions.Http.Resilience`: retry with exponential backoff and jitter, circuit breaker, timeout, bulkhead and fallback. Combine with idempotent consumers, dead-letter queues, health checks and graceful degradation.

**Q74. Explain the CAP theorem and eventual consistency.**
During a network partition you choose consistency or availability. Many microservice systems choose availability with eventual consistency, so the design must handle stale reads, reconciliation and user-visible states such as "pending".

**Q75. CQRS and Event Sourcing: when and when not?**
CQRS separates reads and writes for independent optimization. Event sourcing stores state as a sequence of events (audit trail, temporal queries) but adds complexity (versioning, projections, replay). Avoid both for simple CRUD.

**Q76. How do you handle cross-service queries with database-per-service?**
API composition (aggregator), CQRS read models fed by events, or replicated data via events. Never share a database between services.

**Q77. Observability in microservices?**
Three pillars: logs (Serilog with Seq or ELK), metrics (Prometheus and Grafana), traces (OpenTelemetry with correlation IDs, Jaeger or Application Insights). Add health checks (`/health/live`, `/health/ready`) and SLO-based alerts.

### 🔴 Expert

**Q78. How do you decompose a monolith into microservices?**
Identify bounded contexts (DDD), apply the strangler fig pattern with a facade or gateway, extract the least-coupled, highest-value module first, split data gradually (shared DB, then synchronized, then separate), add anti-corruption layers, and measure outcomes. Avoid a distributed monolith.

**Q79. How do you manage contracts and versioning between services?**
Consumer-driven contract tests (Pact), schema registry for events (Avro or Protobuf), backward- and forward-compatible changes, tolerant readers and event versioning.

**Q80. Kafka vs RabbitMQ vs Azure Service Bus?**
Kafka: distributed log, high throughput, replay, partition ordering, event streaming. RabbitMQ: flexible routing, a traditional broker for task queues. Service Bus: managed, with sessions, dead-lettering, transactions and scheduled delivery. Choose based on ordering, throughput, replay and operational needs.

**Q81. How do you achieve exactly-once processing?**
It is essentially unattainable end to end. Achieve effectively-once with at-least-once delivery plus idempotent consumers, deduplication keys, transactional outbox and inbox, and Kafka idempotent producers and transactions where applicable.

---

## 6. SOLID, Design Patterns and Clean Code

### 🟢 Basic

**Q82. Explain SOLID with examples.**
- **S**ingle Responsibility: separate `OrderService` from `EmailSender`.
- **O**pen/Closed: add discount rules via new classes (Strategy) instead of editing `if/else`.
- **L**iskov Substitution: subtypes must honor the base contract.
- **I**nterface Segregation: several small interfaces beat one fat one.
- **D**ependency Inversion: depend on `IRepository`, not `SqlRepository`.

**Q83. What is the difference between composition and inheritance?**
Inheritance models "is-a" and couples tightly. Composition models "has-a" and is more flexible. Favor composition over inheritance.

### 🟡 Intermediate

**Q84. Which design patterns have you used in production?**
Creational: Factory, Abstract Factory, Builder, Singleton (via DI).
Structural: Adapter, Decorator, Facade, Proxy.
Behavioral: Strategy, Observer, Command, Mediator, Template Method, Chain of Responsibility.
Be ready with a real example for each one you claim.

**Q85. Strategy vs State vs Command pattern?**
Strategy swaps an algorithm chosen by the caller. State changes behavior as internal state changes. Command encapsulates a request as an object (undo, queueing).

**Q86. Decorator pattern use case?**
Add caching, logging or retries around a service without modifying it, for example `CachedProductRepository : IProductRepository` wrapping the real repository.

### 🟠 Advanced

**Q87. What is Domain-Driven Design?**
Ubiquitous language, bounded contexts, aggregates (consistency boundary with an aggregate root), entities vs value objects, domain events, repositories and domain services. Keep aggregates small and reference other aggregates by ID.

**Q88. Clean Architecture / Onion / Hexagonal. How do you structure a solution?**
Layers: Domain (entities, no dependencies) → Application (use cases, interfaces) → Infrastructure (EF Core, external services) → API/UI. Dependencies point inward. Use MediatR or plain handlers for use cases.

**Q89. What is the Mediator pattern and MediatR? Any downsides?**
It decouples senders from handlers, giving a pipeline for cross-cutting concerns (validation, logging, transactions). Downsides are indirection, harder navigation and overuse for trivial calls.

**Q90. How do you identify and refactor code smells?**
Smells include long methods, large classes, feature envy, primitive obsession, shotgun surgery and duplicate code. Refactor under test coverage in small steps (extract method, introduce parameter object, replace conditional with polymorphism).

### 🔴 Expert

**Q91. How do you enforce architectural rules in a large codebase?**
Use architecture tests (NetArchTest or ArchUnitNET), analyzers and `.editorconfig`, modular boundaries (separate projects or internal visibility), ADRs, code review checklists, and CI quality gates (SonarQube).

---

## 7. Data Structures and Algorithms

### 🟢 Basic

**Q92. Array vs Linked List vs `List<T>`?**
An array is fixed-size with O(1) index access. A linked list has O(1) insert/remove at a known node but O(n) access. `List<T>` is a dynamic array with amortized O(1) append and O(n) insert in the middle.

**Q93. Stack vs Queue?**
Stack is LIFO (`Stack<T>`), used for undo and parsing. Queue is FIFO (`Queue<T>`), used for scheduling and BFS.

**Q94. Explain Big-O notation.**
It describes growth of time or space with input size: O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ). Know the complexity of the collections you use daily.

**Q95. Reverse a string.**
```csharp
public static string Reverse(string s)
{
    var chars = s.ToCharArray();
    Array.Reverse(chars);
    return new string(chars);
}
```

**Q96. Check whether a string is a palindrome.**
```csharp
public static bool IsPalindrome(string s)
{
    int i = 0, j = s.Length - 1;
    while (i < j)
    {
        while (i < j && !char.IsLetterOrDigit(s[i])) i++;
        while (i < j && !char.IsLetterOrDigit(s[j])) j--;
        if (char.ToLower(s[i++]) != char.ToLower(s[j--])) return false;
    }
    return true;
}
```

### 🟡 Intermediate

**Q97. `Dictionary` vs `List` vs `HashSet` vs `SortedDictionary`?**
`Dictionary`: O(1) average key lookup. `HashSet`: O(1) membership and uniqueness. `List`: O(n) search. `SortedDictionary`: O(log n) with ordered keys.

**Q98. Two Sum (O(n)).**
```csharp
public static (int, int)? TwoSum(int[] nums, int target)
{
    var seen = new Dictionary<int, int>();
    for (int i = 0; i < nums.Length; i++)
    {
        if (seen.TryGetValue(target - nums[i], out int j)) return (j, i);
        seen[nums[i]] = i;
    }
    return null;
}
```

**Q99. Valid parentheses (stack).**
```csharp
public static bool IsValid(string s)
{
    var stack = new Stack<char>();
    var pairs = new Dictionary<char, char> { [')'] = '(', [']'] = '[', ['}'] = '{' };
    foreach (var c in s)
    {
        if (pairs.TryGetValue(c, out var open))
        {
            if (stack.Count == 0 || stack.Pop() != open) return false;
        }
        else stack.Push(c);
    }
    return stack.Count == 0;
}
```

**Q100. Binary search.**
```csharp
public static int BinarySearch(int[] a, int target)
{
    int lo = 0, hi = a.Length - 1;
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == target) return mid;
        if (a[mid] < target) lo = mid + 1; else hi = mid - 1;
    }
    return -1;
}
```

**Q101. Find duplicates in an array.**
Use a `HashSet<int>` and add each item, collecting those where `Add` returns false. That is O(n) time and O(n) space. With a sorted array, compare neighbors in O(1) space.

### 🟠 Advanced

**Q102. Longest substring without repeating characters (sliding window).**
```csharp
public static int LengthOfLongestSubstring(string s)
{
    var last = new Dictionary<char, int>();
    int best = 0, start = 0;
    for (int i = 0; i < s.Length; i++)
    {
        if (last.TryGetValue(s[i], out int prev) && prev >= start) start = prev + 1;
        last[s[i]] = i;
        best = Math.Max(best, i - start + 1);
    }
    return best;
}
```

**Q103. Detect a cycle in a linked list.**
Floyd's tortoise and hare: a slow pointer moves 1 step and a fast pointer moves 2. If they meet, there is a cycle. O(n) time, O(1) space.

**Q104. Implement an LRU cache.**
A `Dictionary<K, LinkedListNode<(K,V)>>` plus a `LinkedList`. On get or put, move the node to the front. When over capacity, remove the tail. Both operations are O(1).

**Q105. Merge intervals.**
Sort by start, then iterate and merge when `current.start <= last.end`. O(n log n).

**Q106. Top K frequent elements.**
Count with a dictionary, then use a min-heap (`PriorityQueue<T, int>`) of size K. O(n log k).

**Q107. BFS vs DFS, and when to use each?**
BFS uses a queue and finds the shortest path in unweighted graphs and level-order traversals. DFS uses a stack or recursion and suits path exploration, cycle detection and topological sort. Mind recursion depth (stack overflow) and prefer iterative DFS for deep graphs.

**Q108. Dynamic programming example: climbing stairs / coin change.**
Identify overlapping subproblems and optimal substructure. For coin change: `dp[a] = min(dp[a - coin] + 1)`. Discuss memoization (top-down) vs tabulation (bottom-up) and space optimization.

### 🔴 Expert

**Q109. Design a rate limiter algorithmically.**
Compare fixed window (simple but bursty at the edges), sliding window log (accurate but memory-heavy), sliding window counter (a good approximation) and token bucket (allows bursts, smooth refill). For distributed systems use Redis with Lua scripts for atomicity.

**Q110. How do DSA choices show up in real applications?**
Examples: replacing a nested-loop lookup with a dictionary (O(n²) → O(n)), using a trie for autocomplete, a heap for scheduling, a bloom filter to avoid needless DB lookups, and pagination with keyset instead of offset.

---

## 8. System Design and Architecture

### 🟡 Intermediate

**Q111. How do you approach a system design question?**
Follow this order:
1. Clarify functional and non-functional requirements.
2. Estimate scale (QPS, storage, bandwidth).
3. Define APIs.
4. Design the data model.
5. Sketch the high-level architecture.
6. Deep-dive into bottlenecks.
7. Cover scaling, caching and failure handling.
8. Discuss trade-offs.

**Q112. What is horizontal vs vertical scaling?**
Vertical: bigger machine (limited, single point of failure). Horizontal: more instances behind a load balancer (requires stateless services).

**Q113. Caching strategies?**
Cache-aside, read-through, write-through, write-behind. Cover TTL, invalidation, cache stampede protection and the layers (in-memory, Redis, CDN).

### 🟠 Advanced

**Q114. Design a URL shortener.**
- Requirements: create and redirect, high read ratio, optional expiry and analytics.
- ID generation: base62 of a distributed counter or Snowflake-style ID (avoid hash collisions).
- Storage: key-value or SQL with an index on the short key.
- Redirect: 301 vs 302 (302 if you need analytics).
- Scale: Redis cache for hot URLs, read replicas, CDN, async analytics via queue.

**Q115. Design an order management / e-commerce checkout system.**
Services: Catalog, Cart, Order, Payment, Inventory and Notification. Flow: create the order as Pending, reserve inventory, charge payment, and confirm or compensate via a Saga. Use the outbox pattern, idempotency keys for payments, an event-driven status update, and eventual consistency with user-visible states.

**Q116. Design a notification system (email/SMS/push).**
An API accepts requests and enqueues them. Workers per channel consume the queue, with a template service, user preferences, retries with backoff, a dead-letter queue, deduplication and rate limiting per provider. Track delivery status via webhooks.

**Q117. Design a real-time chat or live dashboard.**
SignalR (WebSockets with fallbacks) with a Redis or Azure SignalR backplane for scale-out, a message broker for fan-out, persistence for history, presence tracking, and reconnect and ordering handling.

**Q118. How do you design for high availability and disaster recovery?**
Multi-AZ deployment, redundancy with no single point of failure, health checks and automatic failover, data replication, backups with tested restores, RTO/RPO targets, graceful degradation, and chaos testing.

### 🔴 Expert

**Q119. How do you make technical decisions and document them?**
Use ADRs (context, options, decision, consequences). Evaluate options against non-functional requirements and cost, prototype risky choices (POC), involve stakeholders, and revisit decisions when assumptions change.

**Q120. Design a multi-tenant SaaS application.**
Compare tenancy models: database per tenant (strong isolation, high cost), schema per tenant, or shared schema with a `TenantId` discriminator (cheap, needs strict filtering via EF global query filters). Cover tenant resolution (subdomain or claim), per-tenant config and throttling, noisy-neighbor protection and data residency.

**Q121. How do you evaluate build vs buy, and cloud service choices?**
Consider total cost of ownership, time to market, vendor lock-in, team skills, compliance, scalability and operational burden, and keep an exit strategy.

---

## 9. Performance and Troubleshooting

### 🟡 Intermediate

**Q122. How do you improve API performance?**
Async I/O, caching (memory and Redis), pagination, `AsNoTracking`, projection, response compression, HTTP/2, connection pooling, avoiding N+1 queries and reducing payload size. Always measure first.

**Q123. What tools do you use for profiling and diagnostics?**
`dotnet-counters`, `dotnet-trace`, `dotnet-dump`, `dotnet-gcdump`, PerfView, dotTrace/dotMemory, Application Insights, BenchmarkDotNet for micro-benchmarks, and SQL Profiler or query store.

### 🟠 Advanced

**Q124. A production API suddenly became slow. How do you investigate?**
1. Check dashboards and traces to find where time is spent (DB, external call, CPU, GC, queue).
2. Check recent deployments and config changes.
3. Examine slow queries and execution plans, and look for missing indexes or locks and blocking.
4. Look for thread-pool starvation (sync-over-async), connection pool exhaustion and memory pressure.
5. Mitigate first (rollback, scale out, feature flag), then root-cause.
6. Reproduce under load, fix, add alerts and write a postmortem.

**Q125. What is thread-pool starvation and how do you detect it?**
Blocking calls (`.Result`, `.Wait()`, sync I/O) hold pool threads, so queued work waits and latency spikes while CPU stays low. Detect with `dotnet-counters` (`ThreadPool Queue Length`, `ThreadPool Thread Count`) and fix by going async end to end.

**Q126. How do you do load and performance testing?**
Define SLOs and realistic workloads, then use k6, JMeter, NBomber or Azure Load Testing. Run in a production-like environment, watch percentiles (p95/p99) rather than only averages, find the saturation point and run soak tests.

**Q127. High CPU or high memory in production. What do you do?**
Capture a dump or trace (`dotnet-dump collect`, `dotnet-trace`), analyze hot methods or retained objects (dotMemory, `dumpheap`), and check GC settings, allocations in hot paths, unbounded caches and leaked handlers.

---

## 10. Security

### 🟡 Intermediate

**Q128. Authentication vs authorization?**
Authentication verifies identity. Authorization decides what the identity may do. In ASP.NET Core: `UseAuthentication()` then `UseAuthorization()`, with `[Authorize]`, roles and policies.

**Q129. How does JWT authentication work? What are the pitfalls?**
A signed token (header, payload, signature) sent as a Bearer token. Validate the signature, issuer, audience and expiry. Keep tokens short-lived and use refresh tokens. Never store sensitive data in the payload, and handle revocation (short TTL, rotation, a deny list).

**Q130. What are the OWASP Top 10 risks relevant to APIs?**
Broken access control, injection (SQL, command), cryptographic failures, insecure design, security misconfiguration, vulnerable components, authentication failures, SSRF and logging failures. Mitigate via parameterized queries, input validation, least privilege and dependency scanning.

### 🟠 Advanced

**Q131. Explain OAuth2 and OIDC flows.**
Authorization Code with PKCE (SPAs and mobile), Client Credentials (service to service), and Device Code. OIDC adds identity (ID token) on top of OAuth2. Implement with Azure AD/Entra ID, Duende IdentityServer or Keycloak.

**Q132. Policy-based authorization?**
Define requirements and handlers (`IAuthorizationHandler`) and register policies (`RequireClaim`, custom requirements). It supports resource-based authorization (e.g., "user can edit only their own document").

**Q133. How do you manage secrets and sensitive data?**
Azure Key Vault or HashiCorp Vault, managed identities (no secrets in code or config), user secrets locally, encryption at rest and in transit (TLS), Data Protection API for app-level encryption, and secret rotation.

**Q134. CORS: what is it and how do you configure it safely?**
A browser mechanism restricting cross-origin requests. Configure explicit allowed origins, methods and headers (no `AllowAnyOrigin` with credentials), and remember that CORS is not an authorization mechanism.

**Q135. How do you protect against CSRF and XSS in SPA + API setups?**
XSS: output encoding, Content Security Policy, avoiding `dangerouslySetInnerHTML` and `bypassSecurityTrust*`. CSRF: SameSite cookies, anti-forgery tokens for cookie auth. Bearer tokens in headers are not auto-attached, which reduces CSRF, but avoid `localStorage` for tokens and prefer HttpOnly cookies via a BFF.

---

## 11. React

### 🟢 Basic

**Q136. What is React, and what is the virtual DOM?**
A component-based UI library. The virtual DOM is an in-memory representation of the UI. React diffs the new tree against the previous one (reconciliation) and applies minimal real DOM updates.

**Q137. Functional vs class components?**
Functional components with hooks are the modern standard. Class components use lifecycle methods (`componentDidMount` and so on) and are legacy.

**Q138. What are props vs state?**
Props are read-only inputs from the parent. State is internal, mutable data managed with `useState`. A state change triggers a re-render.

**Q139. Why are keys important in lists?**
Keys let React identify items across renders to update, reorder or remove them correctly. Use stable unique IDs, not array indexes, when the list can reorder.

**Q140. Controlled vs uncontrolled components?**
Controlled: form value is held in React state (`value` + `onChange`). Uncontrolled: the DOM holds the value, read via `ref`.

### 🟡 Intermediate

**Q141. Explain `useEffect`, its dependency array and cleanup.**
It runs side effects after render. `[]` runs on mount only, `[a, b]` re-runs when they change, and omitting the array runs after every render. The returned function is cleanup (unsubscribe, abort fetch). In Strict Mode dev, effects run twice to expose missing cleanup.

**Q142. `useMemo` vs `useCallback` vs `React.memo`?**
`useMemo` memoizes a computed value, `useCallback` memoizes a function reference, and `React.memo` skips re-rendering a component when its props are unchanged. Use them only where measurable cost exists, since they add overhead and complexity.

**Q143. What is `useRef` used for?**
Access DOM nodes and hold mutable values that persist across renders without causing re-renders.

**Q144. How do you share state between components?**
Lift state up, use Context (good for low-frequency global values such as theme or auth), or use a state library (Redux Toolkit, Zustand, Jotai). Use TanStack Query or SWR for server state.

**Q145. Explain React Router basics and protected routes.**
Declarative routes (`<Routes>`, `<Route>`, `useNavigate`, `useParams`). A protected route is a wrapper that checks auth and redirects (`<Navigate to="/login" />`).

**Q146. How do you call a .NET API from React?**
Use `fetch` or axios, preferably wrapped in TanStack Query for caching, retries and loading and error states. Centralize the base URL and an interceptor for the auth token, and handle 401 with token refresh.
```jsx
const { data, isLoading, error } = useQuery({
  queryKey: ['orders', page],
  queryFn: () => api.get(`/orders?page=${page}`).then(r => r.data),
});
```

### 🟠 Advanced

**Q147. How does React reconciliation and Fiber work?**
Fiber is the reconciler that breaks rendering into units of work that can be paused, prioritized and resumed (concurrent rendering). The render phase is pure and interruptible, and the commit phase applies DOM changes.

**Q148. Redux Toolkit vs Context vs Zustand?**
Context is simple but re-renders all consumers when the value changes. Redux Toolkit suits large apps with complex flows, devtools and middleware. Zustand is lightweight with selector-based subscriptions. Choose by scale and team familiarity, and separate server state from client state.

**Q149. How do you optimize React performance?**
Code splitting (`React.lazy`, `Suspense`), list virtualization (react-window), avoiding unnecessary re-renders (memoization, proper state placement), debouncing inputs, image optimization, bundle analysis and the React Profiler.

**Q150. What are custom hooks and when do you write one?**
Functions starting with `use` that encapsulate reusable stateful logic (`useDebounce`, `useFetch`, `useAuth`) and make components cleaner and testable.

**Q151. What are error boundaries?**
Class components implementing `componentDidCatch` or `getDerivedStateFromError` to catch render errors in a subtree and show a fallback. They do not catch event handler or async errors.

**Q152. React 18+ features: concurrent rendering, `useTransition`, `useDeferredValue`, Suspense, Server Components?**
`useTransition` marks updates as non-urgent, keeping the UI responsive. `useDeferredValue` defers a value. Suspense handles loading boundaries. Server Components (Next.js) render on the server and ship less JS.

**Q153. How do you test React components?**
Jest or Vitest with React Testing Library (test behavior as the user sees it, not implementation), MSW to mock APIs, and Playwright or Cypress for end-to-end tests.

### 🔴 Expert

**Q154. How would you architect a large React application?**
Feature-based folder structure, a shared design system, a typed API client (generated from OpenAPI), centralized auth and error handling, route-level code splitting, micro-frontends (module federation) if multiple teams need independent deploys, a performance budget, accessibility standards and CI checks (lint, type-check, tests).

**Q155. How do you secure a React SPA that calls a .NET API?**
OIDC Authorization Code + PKCE (MSAL or oidc-client), short-lived access tokens, a BFF pattern with HttpOnly SameSite cookies to keep tokens out of JS, CSP headers, sanitization of any HTML and strict CORS.

---

## 12. Angular

### 🟢 Basic

**Q156. What is Angular, and what are its core building blocks?**
A TypeScript-based framework with components, templates, directives, services (DI), pipes, modules (or standalone components) and routing.

**Q157. What are components, directives and pipes?**
Components are directives with templates. Structural directives change the DOM (`*ngIf`, `*ngFor`, or the new `@if`/`@for` control flow). Attribute directives change appearance or behavior. Pipes transform values in templates (`date`, `async`).

**Q158. Explain data binding types.**
Interpolation `{{ }}`, property binding `[prop]`, event binding `(event)` and two-way binding `[(ngModel)]`.

**Q159. What is dependency injection in Angular?**
Hierarchical injectors supply services declared with `@Injectable({ providedIn: 'root' })` or via `inject()`. Providers can be scoped at the root, component or route level.

### 🟡 Intermediate

**Q160. Component lifecycle hooks?**
`ngOnChanges`, `ngOnInit`, `ngDoCheck`, `ngAfterContentInit`, `ngAfterViewInit`, `ngOnDestroy`. Use `ngOnInit` for initialization and `ngOnDestroy` to clean up subscriptions.

**Q161. Template-driven vs reactive forms?**
Template-driven is simple and directive-based. Reactive forms (`FormGroup`, `FormControl`, `FormBuilder`) are explicit, testable and scalable, with dynamic controls and custom validators. Prefer reactive for complex forms.

**Q162. What is RxJS and why does Angular use it?**
Reactive streams (Observables) for async data. Key operators: `map`, `filter`, `switchMap` (cancel the previous, good for search), `mergeMap`, `concatMap` (ordered), `exhaustMap` (ignore while busy), `debounceTime`, `distinctUntilChanged`, `catchError`, `takeUntil`, `combineLatest` and `forkJoin`.

**Q163. How do you avoid memory leaks with subscriptions?**
Use the `async` pipe, `takeUntilDestroyed()`, `takeUntil(destroy$)`, or unsubscribe in `ngOnDestroy`. HTTP observables complete on their own.

**Q164. How do you call a .NET API from Angular?**
`HttpClient` in a typed service, with `HttpInterceptor` (or functional interceptors) to attach tokens, handle errors and show loaders. Handle 401 with refresh logic and use environment-based API URLs.
```typescript
@Injectable({ providedIn: 'root' })
export class OrdersService {
  private http = inject(HttpClient);
  getOrders(page: number) {
    return this.http.get<Order[]>(`${environment.apiUrl}/orders`, { params: { page } });
  }
}
```

**Q165. Routing: guards, resolvers and lazy loading?**
Guards (`canActivate`, `canMatch`, `canDeactivate`) control navigation. Resolvers pre-fetch data. Lazy loading (`loadComponent` / `loadChildren`) splits bundles per route.

### 🟠 Advanced

**Q166. How does change detection work? What is `OnPush`?**
By default Zone.js triggers change detection across the tree after async events. `OnPush` checks a component only when its inputs change by reference, an event fires inside it, an `async` pipe emits or it is marked manually. It improves performance, and works best with immutable data.

**Q167. What are Angular Signals?**
A fine-grained reactivity primitive (`signal()`, `computed()`, `effect()`) that enables precise updates and, with zoneless change detection, reduces reliance on Zone.js. Use `toSignal`/`toObservable` to interoperate with RxJS.

**Q168. NgModules vs standalone components?**
Modern Angular favors standalone components (imports declared directly, less boilerplate, easier lazy loading). NgModules remain in legacy codebases and can be migrated incrementally.

**Q169. State management options in Angular?**
Services with `BehaviorSubject` or signals for small and medium apps. NgRx (store, effects, selectors) or NGXS/Akita for large apps needing predictable state, devtools and complex async flows.

**Q170. How do you optimize Angular performance?**
`OnPush`, `trackBy` (or `track` in `@for`), lazy loading and deferrable views (`@defer`), pure pipes instead of template method calls, virtual scrolling (CDK), bundle analysis, preloading strategies, SSR/hydration and avoiding heavy work in templates.

**Q171. How do you test Angular apps?**
Jasmine/Karma or Jest for units, `TestBed` with `HttpTestingController` for services, component harnesses, and Cypress or Playwright for end-to-end tests.

**Q172. Custom validators and async validators?**
A validator is a function returning `ValidationErrors | null`. Async validators return an Observable or Promise (e.g., a username availability check, with `debounceTime` and `updateOn: 'blur'`).

### 🔴 Expert

**Q173. How would you architect a large enterprise Angular application?**
An Nx workspace or monorepo with feature libraries and enforced boundaries, a shared UI library and design system, lazy-loaded feature areas, typed API clients (OpenAPI generator), centralized error handling and auth interceptors, NgRx for complex state, micro-frontends with Module Federation when teams need independent releases, strict TypeScript and lint rules, and CI with unit, e2e and bundle-size budgets.

**Q174. How do you secure an Angular SPA with a .NET backend?**
OIDC Authorization Code + PKCE (angular-auth-oidc-client or MSAL Angular), route guards (UX only, since the server enforces real authorization), CSP, Angular's built-in sanitization (avoid bypassing it), a BFF with HttpOnly cookies, and strict CORS.

**Q175. React vs Angular: how do you choose?**
React: flexible library, large ecosystem, you pick the router and state tools, a good fit for teams who want control. Angular: opinionated full framework (DI, forms, routing, HTTP, CLI), consistent structure for large teams and enterprise apps. Decide on team skills, project size and long-term maintainability.

---

## 13. Testing, CI/CD and DevOps

### 🟢 Basic

**Q176. What are unit, integration and end-to-end tests?**
Unit tests check isolated logic quickly. Integration tests check components working together (DB, HTTP). End-to-end tests check full user flows. Follow the test pyramid: many unit, fewer integration, few E2E.

**Q177. What is TDD?**
Red (write a failing test), Green (make it pass), Refactor. It drives design and gives safe refactoring.

### 🟡 Intermediate

**Q178. xUnit vs NUnit vs MSTest, and which mocking libraries do you use?**
All are fine. xUnit is the most common in modern .NET. Mocking: Moq or NSubstitute. Assertions: FluentAssertions or Shouldly. Data generation: Bogus or AutoFixture.

**Q179. How do you write integration tests for an ASP.NET Core API?**
`WebApplicationFactory<Program>` to host the app in memory, replace dependencies, and use Testcontainers for real SQL Server, PostgreSQL or Redis. Reset data between tests.

**Q180. What does your CI/CD pipeline look like?**
Build → restore → unit tests → static analysis (SonarQube, analyzers) → security scan (dependency check) → container build → push to a registry → deploy to dev/test/prod with approvals → smoke tests. Use Azure DevOps or GitHub Actions, with Infrastructure as Code (Bicep, Terraform).

### 🟠 Advanced

**Q181. Deployment strategies?**
Rolling, blue-green, canary and feature flags, each with an automated rollback trigger based on health or metrics. Database changes use expand-contract migrations to stay backward compatible.

**Q182. What are contract tests, and why do they matter in microservices?**
Consumer-driven contract tests (Pact) verify that a provider still meets what consumers expect, catching breaking API changes without a full end-to-end environment.

**Q183. How do you measure code quality?**
Meaningful coverage (not a vanity number), mutation testing, static analysis, cyclomatic complexity, code review standards, DORA metrics (deployment frequency, lead time, change failure rate, MTTR) and tech-debt tracking.

---

## 14. Behavioral and Ownership

**Q184. Tell me about a critical production issue you resolved. (STAR)**
Situation (what broke, business impact), Task (your role), Action (diagnosis, mitigation, communication), Result (fix, metrics, prevention such as alerts, tests and postmortem).

**Q185. How do you translate vague business requirements into technical solutions?**
Ask clarifying questions, identify stakeholders and constraints, write acceptance criteria, break work into small deliverables, prototype or demo early and raise risks upfront.

**Q186. How do you handle a technical disagreement with a teammate or architect?**
Bring data (POC, benchmarks), compare trade-offs, document the decision (ADR), then disagree and commit.

**Q187. How do you conduct and receive code reviews?**
Review for correctness, readability, security, performance, tests and design. Be specific and kind, explain the "why" and distinguish blocking from nit-level comments. When receiving feedback, stay open and ask questions.

**Q188. How do you mentor junior developers?**
Pair programming, constructive code reviews, stretch assignments with support, sharing design context and encouraging ownership and learning.

**Q189. Describe a time you improved application performance. What was the impact?**
Give the baseline metric, how you found the bottleneck, what you changed, and the result (for example "p95 dropped from 2.4s to 400ms"), plus how you guarded against regression.

**Q190. How do you work effectively in a hybrid team?**
Written, async-first communication, documented decisions, clear ownership and deadlines, regular stand-ups, in-office days for design and collaboration, and visible status updates.

**Q191. How do you handle tight deadlines and changing priorities?**
Clarify what is truly must-have, cut scope rather than quality, communicate trade-offs early, deliver incrementally and flag risks to stakeholders.

**Q192. What is the biggest technical mistake you made, and what did you learn?**
Be honest, own it, explain the impact, the fix and the process change you introduced.

**Q193. Why should we hire you?**
Map your hands-on experience (APIs, microservices, performance, mentoring, frontend exposure) to the JD, with one concrete, measurable example.

**Q194. Questions you can ask the interviewer.**
- What does the architecture look like today, and where is it headed?
- How do you handle on-call, incidents and technical debt?
- How are decisions made and documented?
- What does success look like in the first 6 months?

---

## 15. Rapid-Fire Round

| Question | Short answer |
|---|---|
| `IEnumerable` vs `IQueryable`? | In-memory iteration vs expression tree translated to SQL |
| Scoped service lifetime? | One instance per HTTP request |
| `async void` allowed when? | Event handlers only |
| `PUT` idempotent? | Yes |
| `POST` idempotent? | No (unless an idempotency key is used) |
| Status code for resource created? | 201 |
| Status code for validation error? | 400 or 422 |
| Status code for conflict? | 409 |
| Too many requests? | 429 |
| Default SQL Server isolation level? | Read Committed |
| `Dictionary` lookup complexity? | O(1) average |
| `List<T>.Contains` complexity? | O(n) |
| Binary search complexity? | O(log n) |
| Saga purpose? | Distributed transaction via compensating actions |
| Outbox purpose? | Atomic DB write plus event publish |
| Circuit breaker purpose? | Stop calling a failing dependency to let it recover |
| `useEffect` with `[]`? | Runs once after mount |
| Angular `OnPush`? | Check only on input reference change or events |
| `switchMap` use? | Cancel the previous inner observable (search-as-you-type) |
| `ValueTask` use? | Avoid allocations for often-synchronous results |
| LOH threshold? | 85,000 bytes |
| Captive dependency? | Singleton holding a scoped service |

---

## 16. Preparation Checklist

**Technical**
- [ ] Revise async/await, DI lifetimes, middleware order and EF Core pitfalls.
- [ ] Practice 15-20 DSA problems in C# (arrays, strings, hashing, stacks, sliding window, trees, graphs).
- [ ] Rehearse 2-3 system designs aloud (URL shortener, order management, notification system).
- [ ] Be able to draw your current project's architecture and explain every trade-off.
- [ ] Refresh React or Angular (whichever you will claim) including state, forms, API integration and performance.

**Project stories (prepare 3)**
- [ ] A scalable or high-performance system you built (with metrics).
- [ ] A production incident you resolved.
- [ ] A design decision with trade-offs you led or influenced.

**Interview day**
- [ ] Clarify requirements before answering design questions.
- [ ] Think aloud and state trade-offs ("I'd choose X because..., the downside is...").
- [ ] Give real examples from experience, not only definitions.
- [ ] Be honest about what you don't know, and explain how you would find out.

*Good luck!*
