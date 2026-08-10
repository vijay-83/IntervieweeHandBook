# Microservices Architecture Patterns (with C# Examples)

A reference guide to 25 common microservices design patterns — what each one does, why it matters, and a C# code example.

---

## 1. Circuit Breaker

**Details:** Wraps calls to a remote service and monitors for failures. Once failures cross a threshold, the circuit "opens" and further requests fail fast instead of hitting the struggling service — preventing cascading failures. After a cooldown, it goes "half-open" to test recovery.

```csharp
using Polly;
using Polly.CircuitBreaker;

var breaker = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(
        exceptionsAllowedBeforeBreaking: 3,
        durationOfBreak: TimeSpan.FromSeconds(30),
        onBreak: (ex, ts) => Console.WriteLine("Circuit opened!"),
        onReset: () => Console.WriteLine("Circuit closed."));

try
{
    var result = await breaker.ExecuteAsync(() => paymentClient.ChargeAsync(orderId));
}
catch (BrokenCircuitException)
{
    // Fail fast with a fallback instead of calling the failing service
    return PaymentResult.TemporarilyUnavailable();
}
```

---

## 2. Retry

**Details:** Automatically retries a failed request a set number of times, often with exponential backoff and jitter, to recover from transient issues.

```csharp
using Polly;

var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)));

await retryPolicy.ExecuteAsync(() => inventoryClient.ReserveStockAsync(itemId, qty));
```

---

## 3. Bulkhead

**Details:** Isolates resources (thread pools, connection pools) per dependency so a failure in one doesn't exhaust shared resources used by others.

```csharp
using Polly;
using Polly.Bulkhead;

var bulkheadA = Policy.BulkheadAsync(maxParallelization: 10, maxQueuingActions: 5);
var bulkheadC = Policy.BulkheadAsync(maxParallelization: 5, maxQueuingActions: 2);

// Service C's slow calls only exhaust its own pool, not Service A's
await bulkheadC.ExecuteAsync(() => serviceCClient.CallAsync());
await bulkheadA.ExecuteAsync(() => serviceAClient.CallAsync());
```

---

## 4. Saga Pattern

**Details:** Manages distributed transactions via a sequence of local transactions, each followed by an event/command triggering the next step. Failures trigger compensating transactions to undo prior steps.

```csharp
public class OrderSaga
{
    public async Task ExecuteAsync(Order order)
    {
        try
        {
            await _paymentService.ChargeAsync(order);
            await _inventoryService.ReserveAsync(order);
            await _shippingService.ScheduleAsync(order);
        }
        catch (InventoryUnavailableException)
        {
            // Compensating transactions undo prior steps
            await _paymentService.RefundAsync(order);
            await _orderService.CancelAsync(order);
        }
    }
}
```

---

## 5. API Gateway

**Details:** A single entry point handling routing, auth, rate limiting, SSL, and caching for backend microservices — commonly implemented with Ocelot or YARP in .NET.

```csharp
// ocelot.json (Ocelot API Gateway route config)
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/payments/{everything}",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [{ "Host": "payment-service", "Port": 443 }],
      "UpstreamPathTemplate": "/gateway/payments/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post" ]
    }
  ]
}

// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Configuration.AddJsonFile("ocelot.json");
builder.Services.AddOcelot();
var app = builder.Build();
await app.UseOcelot();
app.Run();
```

---

## 6. Service Discovery

**Details:** Services register with a central registry on startup and discover each other dynamically at runtime instead of using hardcoded addresses.

```csharp
// Registering with Consul on startup
using Consul;

var consulClient = new ConsulClient(c => c.Address = new Uri("http://consul:8500"));
await consulClient.Agent.ServiceRegister(new AgentServiceRegistration
{
    ID = "service-b-1",
    Name = "service-b",
    Address = "10.0.0.5",
    Port = 5001
});

// Service A discovering Service B
var services = await consulClient.Health.Service("service-b", tag: null, passingOnly: true);
var instance = services.Response.First();
var url = $"http://{instance.Service.Address}:{instance.Service.Port}";
```

---

## 7. Database per Service

**Details:** Each microservice owns and manages its own private database; other services access its data only through its API.

```csharp
// OrderService — its own DbContext, own database, own connection string
public class OrderDbContext : DbContext
{
    public DbSet<Order> Orders => Set<Order>();
    protected override void OnConfiguring(DbContextOptionsBuilder options) =>
        options.UseSqlServer("Server=order-db;Database=OrdersDb;...");
}

// If OrderService needs payment info, it calls PaymentService's API —
// it never queries PaymentDb directly.
var paymentInfo = await _paymentServiceHttpClient.GetAsync($"/payments/{orderId}");
```

---

## 8. Event-Driven Architecture

**Details:** Services communicate asynchronously by publishing and subscribing to events via a message broker, decoupling producers from consumers.

```csharp
// Publisher (Order Service) — using MassTransit + Kafka/RabbitMQ
public record OrderCreated(Guid OrderId, decimal Amount);

await _publishEndpoint.Publish(new OrderCreated(order.Id, order.Total));

// Consumer (Inventory Service)
public class OrderCreatedConsumer : IConsumer<OrderCreated>
{
    public async Task Consume(ConsumeContext<OrderCreated> context)
    {
        await _inventory.ReserveStockAsync(context.Message.OrderId);
    }
}
```

---

## 9. CQRS

**Details:** Separates the write model (commands) from the read model (queries), often backed by different stores, improving performance and scalability.

```csharp
// Command side
public record CreateOrderCommand(Guid CustomerId, List<OrderItem> Items) : IRequest<Guid>;

public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    public async Task<Guid> Handle(CreateOrderCommand cmd, CancellationToken ct)
    {
        var order = new Order(cmd.CustomerId, cmd.Items);
        await _writeDb.Orders.AddAsync(order, ct);
        await _writeDb.SaveChangesAsync(ct);
        return order.Id;
    }
}

// Query side — reads from a denormalized read replica
public record GetOrderSummaryQuery(Guid OrderId) : IRequest<OrderSummaryDto>;

public class GetOrderSummaryHandler : IRequestHandler<GetOrderSummaryQuery, OrderSummaryDto>
{
    public Task<OrderSummaryDto> Handle(GetOrderSummaryQuery q, CancellationToken ct) =>
        _readDb.OrderSummaries.FirstAsync(o => o.OrderId == q.OrderId, ct);
}
```

---

## 10. Event Sourcing

**Details:** Stores a full sequence of events rather than just current state; current state is rebuilt by replaying events.

```csharp
public abstract record AccountEvent;
public record AccountOpened(Guid AccountId) : AccountEvent;
public record Deposited(decimal Amount) : AccountEvent;
public record Withdrawn(decimal Amount) : AccountEvent;

public class Account
{
    public decimal Balance { get; private set; }

    public void Apply(AccountEvent e) => Balance = e switch
    {
        Deposited d => Balance + d.Amount,
        Withdrawn w => Balance - w.Amount,
        _ => Balance
    };

    public static Account Rebuild(IEnumerable<AccountEvent> events)
    {
        var account = new Account();
        foreach (var e in events) account.Apply(e);
        return account;
    }
}
```

---

## 11. Strangler Fig Pattern

**Details:** Gradually replaces a legacy monolith by routing specific functionality to new microservices via a gateway, until the monolith is fully replaced.

```csharp
// Gateway routing rule: new checkout goes to microservice, everything else to monolith
app.MapWhen(
    ctx => ctx.Request.Path.StartsWithSegments("/checkout"),
    branch => branch.UseHttpsRedirection().Run(async ctx =>
        await ctx.Response.WriteAsync(await checkoutServiceClient.ForwardAsync(ctx.Request))));

app.MapFallback(async ctx =>
    await ctx.Response.WriteAsync(await legacyMonolithClient.ForwardAsync(ctx.Request)));
```

---

## 12. Sidecar Pattern

**Details:** Deploys a helper container alongside the main service (same pod) to handle cross-cutting concerns like logging or monitoring without modifying the main service.

```yaml
# Kubernetes pod spec: main .NET app + sidecar
apiVersion: v1
kind: Pod
metadata:
  name: order-service
spec:
  containers:
    - name: order-service
      image: myregistry/order-service:latest
    - name: envoy-sidecar
      image: envoyproxy/envoy:v1.29
      ports:
        - containerPort: 9901
```

```csharp
// The .NET app itself stays unaware of the sidecar —
// it just logs to stdout/localhost and the sidecar handles shipping/securing it
_logger.LogInformation("Order {OrderId} processed", order.Id);
```

---

## 13. Ambassador Pattern

**Details:** A specialized sidecar proxy that handles communication with external services on behalf of the main service — useful for legacy integration or protocol translation.

```csharp
// Main service just calls a local ambassador endpoint
public class LegacyOrderClient
{
    private readonly HttpClient _http; // points to localhost:5050 (the ambassador)

    public async Task<Order> GetOrderAsync(Guid id) =>
        await _http.GetFromJsonAsync<Order>($"/orders/{id}");
}

// The ambassador process (separate container) translates
// REST/JSON <-> legacy SOAP/XML and adds retries/auth transparently
```

---

## 14. Adapter Pattern

**Details:** Provides a compatibility layer letting incompatible interfaces work together, often used to connect new services with legacy systems.

```csharp
public interface IModernPaymentGateway
{
    Task<PaymentResult> ChargeAsync(decimal amount, string cardToken);
}

// Adapts the old SOAP-based legacy system to the modern interface
public class LegacyPaymentAdapter : IModernPaymentGateway
{
    private readonly LegacySoapPaymentClient _legacyClient;

    public async Task<PaymentResult> ChargeAsync(decimal amount, string cardToken)
    {
        var soapResponse = await _legacyClient.ProcessPaymentXmlAsync(
            BuildLegacyXmlRequest(amount, cardToken));
        return MapToPaymentResult(soapResponse);
    }
}
```

---

## 15. Proxy Pattern

**Details:** Introduces a proxy that controls access to the real service, adding security, caching, or logging without the client talking to the real service directly.

```csharp
public interface IProductService
{
    Task<Product> GetProductAsync(int id);
}

public class CachingProductServiceProxy : IProductService
{
    private readonly IProductService _realService;
    private readonly IMemoryCache _cache;

    public async Task<Product> GetProductAsync(int id)
    {
        if (_cache.TryGetValue(id, out Product cached)) return cached;

        var product = await _realService.GetProductAsync(id);
        _cache.Set(id, product, TimeSpan.FromMinutes(5));
        return product;
    }
}
```

---

## 16. Factory Pattern

**Details:** Centralizes object creation logic in a Factory so client code doesn't need to know instantiation details — promotes loose coupling.

```csharp
public interface IPaymentProcessor { Task ProcessAsync(decimal amount); }

public static class PaymentProcessorFactory
{
    public static IPaymentProcessor Create(string provider) => provider switch
    {
        "stripe" => new StripeProcessor(),
        "paypal" => new PayPalProcessor(),
        _ => throw new ArgumentException("Unknown provider")
    };
}

var processor = PaymentProcessorFactory.Create("stripe");
await processor.ProcessAsync(49.99m);
```

---

## 17. Strategy Pattern

**Details:** Defines a family of interchangeable algorithms behind a common interface, allowing the algorithm to vary at runtime.

```csharp
public interface IPricingStrategy { decimal CalculatePrice(Order order); }

public class StandardPricing : IPricingStrategy
{
    public decimal CalculatePrice(Order order) => order.Subtotal;
}

public class CouponPricing : IPricingStrategy
{
    public decimal CalculatePrice(Order order) => order.Subtotal * 0.9m;
}

public class Checkout
{
    private readonly IPricingStrategy _strategy;
    public Checkout(IPricingStrategy strategy) => _strategy = strategy;
    public decimal GetTotal(Order order) => _strategy.CalculatePrice(order);
}
```

---

## 18. Observer Pattern

**Details:** Establishes a one-to-many dependency where Observers subscribe to a Subject and get notified automatically when its state changes.

```csharp
public class StockTicker
{
    public event Action<decimal>? PriceChanged;

    public void UpdatePrice(decimal newPrice) => PriceChanged?.Invoke(newPrice);
}

var ticker = new StockTicker();
ticker.PriceChanged += price => dashboard.Update(price);
ticker.PriceChanged += price => alertingService.CheckThreshold(price);
ticker.PriceChanged += price => auditLogger.Log(price);

ticker.UpdatePrice(152.30m); // notifies all subscribers
```

---

## 19. Singleton Pattern

**Details:** Ensures a class has only one instance app-wide and provides a global access point — commonly used for shared config or connection managers.

```csharp
// Simplest form in modern .NET: register as a Singleton in DI
builder.Services.AddSingleton<IConnectionPoolManager, ConnectionPoolManager>();

public class ConnectionPoolManager : IConnectionPoolManager
{
    // Same instance injected everywhere it's requested throughout the app
    public SqlConnection GetConnection() => _pool.Rent();
}
```

---

## 20. Builder Pattern

**Details:** Separates construction of a complex object from its representation, assembling it step-by-step — useful for immutable objects/DTOs with many optional fields.

```csharp
public class HttpRequestBuilder
{
    private readonly HttpRequestMessage _request = new();

    public HttpRequestBuilder WithMethod(HttpMethod method) { _request.Method = method; return this; }
    public HttpRequestBuilder WithUrl(string url) { _request.RequestUri = new Uri(url); return this; }
    public HttpRequestBuilder WithHeader(string key, string value) { _request.Headers.Add(key, value); return this; }
    public HttpRequestMessage Build() => _request;
}

var request = new HttpRequestBuilder()
    .WithMethod(HttpMethod.Post)
    .WithUrl("https://api.example.com/orders")
    .WithHeader("Authorization", "Bearer token")
    .Build();
```

---

## 21. Decorator Pattern

**Details:** Dynamically adds behavior to an object without altering its code, by wrapping it in one or more decorator layers.

```csharp
public interface IOrderService { Task PlaceOrderAsync(Order order); }

public class OrderService : IOrderService
{
    public Task PlaceOrderAsync(Order order) => Task.CompletedTask; // real logic
}

public class LoggingDecorator : IOrderService
{
    private readonly IOrderService _inner;
    public LoggingDecorator(IOrderService inner) => _inner = inner;

    public async Task PlaceOrderAsync(Order order)
    {
        Console.WriteLine($"Placing order {order.Id}");
        await _inner.PlaceOrderAsync(order);
    }
}

public class CachingDecorator : IOrderService
{
    private readonly IOrderService _inner;
    public CachingDecorator(IOrderService inner) => _inner = inner;
    public Task PlaceOrderAsync(Order order) => _inner.PlaceOrderAsync(order); // + caching logic
}

IOrderService service = new CachingDecorator(new LoggingDecorator(new OrderService()));
```

---

## 22. Repository Pattern

**Details:** Abstracts data access behind a repository interface so application code works with domain objects rather than raw queries — improves testability.

```csharp
public interface IOrderRepository
{
    Task<Order?> FindByIdAsync(Guid id);
    Task AddAsync(Order order);
}

public class OrderRepository : IOrderRepository
{
    private readonly OrderDbContext _db;
    public OrderRepository(OrderDbContext db) => _db = db;

    public Task<Order?> FindByIdAsync(Guid id) => _db.Orders.FindAsync(id).AsTask();
    public async Task AddAsync(Order order)
    {
        _db.Orders.Add(order);
        await _db.SaveChangesAsync();
    }
}

// Easy to mock IOrderRepository in unit tests without a real database
```

---

## 23. Dependency Injection Pattern

**Details:** Dependencies are provided from outside (typically by a DI container) rather than being created by the component itself — promotes loose coupling and testability.

```csharp
// Program.cs — registering dependencies with the built-in DI container
builder.Services.AddScoped<IServiceBClient, ServiceBClient>();
builder.Services.AddScoped<ServiceA>();

public class ServiceA
{
    private readonly IServiceBClient _serviceB;

    // Service B is injected, not created by Service A
    public ServiceA(IServiceBClient serviceB) => _serviceB = serviceB;

    public Task DoWorkAsync() => _serviceB.CallAsync();
}
```

---

## 24. Outbox Pattern

**Details:** Ensures reliable event publishing even if the service crashes, by writing the business change and the outgoing event in the same local DB transaction, then a relay process publishes it.

```csharp
public async Task UpdateOrderStatusAsync(Guid orderId, string status)
{
    using var transaction = await _db.Database.BeginTransactionAsync();

    var order = await _db.Orders.FindAsync(orderId);
    order.Status = status; // 1. business transaction

    _db.OutboxMessages.Add(new OutboxMessage           // 2. write to outbox table
    {
        Type = "OrderStatusChanged",
        Payload = JsonSerializer.Serialize(new { orderId, status }),
        CreatedAt = DateTime.UtcNow
    });

    await _db.SaveChangesAsync();
    await transaction.CommitAsync();
}

// 3. Separate background worker polls OutboxMessages and publishes to Kafka/RabbitMQ,
// then marks them as sent — guaranteeing at-least-once delivery.
```

---

## 25. Idempotency Pattern

**Details:** Ensures duplicate requests (e.g., from retries) don't cause duplicate processing, using a unique idempotency key checked before processing.

```csharp
[HttpPost("charge")]
public async Task<IActionResult> ChargeAsync(
    [FromHeader(Name = "Idempotency-Key")] string idempotencyKey,
    [FromBody] ChargeRequest request)
{
    var existing = await _db.IdempotentResponses
        .FirstOrDefaultAsync(r => r.Key == idempotencyKey);

    if (existing != null)
        return Content(existing.ResponseBody, "application/json"); // return stored result

    var result = await _paymentProcessor.ChargeAsync(request);

    _db.IdempotentResponses.Add(new IdempotentResponse
    {
        Key = idempotencyKey,
        ResponseBody = JsonSerializer.Serialize(result)
    });
    await _db.SaveChangesAsync();

    return Ok(result);
}
```

---

## Why These Patterns Matter

- Improve scalability
- Increase resilience
- Reduce coupling
- Enhance maintainability
- Support continuous delivery
- Handle failures gracefully

## Key Principles Behind Microservices

- Decentralized data management
- Independent deployment
- Fault isolation
- Loose coupling
- High availability

## Common Tools & Tech (.NET ecosystem equivalents)

- **ASP.NET Core / Ocelot / YARP** — service framework & API gateway
- **Consul / Eureka** — service discovery & registry
- **Polly** — circuit breaker, retry, bulkhead & resilience policies
- **OpenTelemetry / Jaeger / Zipkin** — distributed tracing
- **Serilog + ELK Stack** — structured logging
- **MassTransit + Kafka / RabbitMQ** — event streaming / message broker
- **Docker** — containerization
- **Prometheus + Grafana** — monitoring & metrics
