# API Management Across Web Stacks
### Blazor (C#/.NET) · Angular · React · Python Web Apps · Gen AI Applications

---

## 1. What "API Management" Covers

"API management" spans both **consuming** APIs (calling them reliably from a client) and **exposing** APIs (building endpoints others can call safely). The concerns are largely the same across stacks:

| Concern | What it solves |
|---|---|
| HTTP client setup | Base URLs, headers, timeouts, serialization |
| Centralized error handling | Consistent handling of 4xx/5xx across all calls |
| Retries & resilience | Surviving transient network/server failures |
| Caching | Avoiding redundant calls, faster responses |
| Rate limiting / throttling | Protecting an API (yours or a third party's) from overload |
| Versioning | Evolving an API without breaking existing clients |
| Documentation | OpenAPI/Swagger so consumers know what's available |
| Gateway/aggregation | Routing, auth, and policy enforcement in front of many services |
| Gen AI equivalent | Managing calls to LLM providers: retries, rate limits, cost/token budgets, multi-provider routing |

---

## 2. Blazor (C# / .NET)

.NET's `HttpClient` plus `IHttpClientFactory` is the foundation; **Polly** adds resilience (retry, circuit breaker, timeout), and **Refit** offers a typed, declarative client.

### 2.1 Basic typed HTTP client via `IHttpClientFactory`
```csharp
// Program.cs
builder.Services.AddHttpClient("ProductsApi", client =>
{
    client.BaseAddress = new Uri("https://api.example.com/");
    client.DefaultRequestHeaders.Add("Accept", "application/json");
});
```
```csharp
@inject IHttpClientFactory ClientFactory

@code {
    private async Task<List<Product>> GetProductsAsync()
    {
        var client = ClientFactory.CreateClient("ProductsApi");
        return await client.GetFromJsonAsync<List<Product>>("products") ?? new();
    }
}
```

### 2.2 Declarative typed clients — Refit
```csharp
public interface IProductsApi
{
    [Get("/products/{id}")]
    Task<Product> GetProduct(int id);

    [Post("/products")]
    Task<Product> CreateProduct([Body] Product product);
}
```
```csharp
builder.Services.AddRefitClient<IProductsApi>()
    .ConfigureHttpClient(c => c.BaseAddress = new Uri("https://api.example.com/"));
```

### 2.3 Resilience — retries, circuit breakers, timeouts with Polly
```csharp
builder.Services.AddHttpClient("ProductsApi")
    .AddResilienceHandler("default", builder =>
    {
        builder.AddRetry(new HttpRetryStrategyOptions { MaxRetryAttempts = 3 });
        builder.AddCircuitBreaker(new HttpCircuitBreakerStrategyOptions());
        builder.AddTimeout(TimeSpan.FromSeconds(10));
    });
```

### 2.4 Centralized error handling
```csharp
public class ApiExceptionHandler : DelegatingHandler
{
    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken ct)
    {
        var response = await base.SendAsync(request, ct);
        if (!response.IsSuccessStatusCode)
        {
            var error = await response.Content.ReadAsStringAsync(ct);
            throw new ApiException(response.StatusCode, error);
        }
        return response;
    }
}
```

### 2.5 Exposing an API — minimal APIs + versioning
```csharp
// Program.cs
var app = builder.Build();
var v1 = app.MapGroup("/api/v1");

v1.MapGet("/products/{id:int}", (int id, ProductService svc) => svc.GetById(id));
v1.MapPost("/products", (Product p, ProductService svc) => svc.Create(p));
```
```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
});
```

### 2.6 Rate limiting an exposed API
```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
    });
});
app.UseRateLimiter();
```

### 2.7 API documentation — Swagger/OpenAPI
```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
app.UseSwagger();
app.UseSwaggerUI();
```

### 2.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic HTTP calls | `IHttpClientFactory` + named/typed clients |
| Declarative/typed API clients | Refit |
| Retries, circuit breakers, timeouts | Polly (via `AddResilienceHandler`) |
| Exposing endpoints | Minimal APIs or Controllers + `Microsoft.AspNetCore.OpenApi` |
| Protecting your API | Built-in `RateLimiter` middleware |
| API docs | Swashbuckle / built-in OpenAPI + Swagger UI |

---

## 3. Angular

Angular's `HttpClient` (standalone, via `provideHttpClient`) is the built-in mechanism for consuming APIs; interceptors provide the centralized cross-cutting behavior.

### 3.1 Basic HTTP client setup
```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(withInterceptors([apiInterceptor]))],
};
```
```typescript
@Injectable({ providedIn: 'root' })
export class ProductsService {
  constructor(private http: HttpClient) {}
  getProducts() { return this.http.get<Product[]>('/api/products'); }
  getProduct(id: string) { return this.http.get<Product>(`/api/products/${id}`); }
}
```

### 3.2 Centralized request/response handling — interceptors
```typescript
export const apiInterceptor: HttpInterceptorFn = (req, next) => {
  const cloned = req.clone({ setHeaders: { 'X-Client': 'web-app' } });
  return next(cloned).pipe(
    catchError((err: HttpErrorResponse) => {
      console.error('API error:', err.status, err.message);
      return throwError(() => err);
    })
  );
};
```

### 3.3 Retries with RxJS
```typescript
getProducts() {
  return this.http.get<Product[]>('/api/products').pipe(
    retry({ count: 3, delay: 1000 }),
    catchError(this.handleError)
  );
}
```

### 3.4 Caching responses
```typescript
private cache$?: Observable<Product[]>;

getProducts() {
  if (!this.cache$) {
    this.cache$ = this.http.get<Product[]>('/api/products').pipe(shareReplay(1));
  }
  return this.cache$;
}
```
> For richer caching (staleness, invalidation, background refetch), pair `HttpClient` with **TanStack Query for Angular** instead of hand-rolling it.

### 3.5 Typed API clients — OpenAPI code generation
```bash
npx openapi-typescript-codegen --input openapi.json --output src/app/api-client
```

### 3.6 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic HTTP calls | `HttpClient` in an injectable service |
| Cross-cutting headers/error handling | `HttpInterceptorFn` |
| Retries | RxJS `retry()` operator |
| Caching | `shareReplay()` or TanStack Query for Angular |
| Typed clients from a spec | OpenAPI code generators |

---

## 4. React

React has no built-in HTTP client — teams typically use **`fetch`** or **Axios** for the call itself, and **TanStack Query** to manage caching, retries, and loading state around it.

### 4.1 Basic API call
```jsx
async function getProducts() {
  const res = await fetch("/api/products");
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

### 4.2 Axios client with interceptors (centralized handling)
```jsx
const api = axios.create({ baseURL: "/api", timeout: 10000 });

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) logoutUser();
    return Promise.reject(error);
  }
);
```

### 4.3 Data fetching, caching, and retries — TanStack Query
```jsx
function useProducts() {
  return useQuery({
    queryKey: ["products"],
    queryFn: () => api.get("/products").then((r) => r.data),
    staleTime: 60_000,
    retry: 3,
  });
}

function ProductList() {
  const { data, isLoading, error } = useProducts();
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <ul>{data.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

### 4.4 Mutations with optimistic updates
```jsx
function useAddProduct() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (product) => api.post("/products", product),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["products"] }),
  });
}
```

### 4.5 Typed clients from an OpenAPI spec
```bash
npx openapi-typescript https://api.example.com/openapi.json -o src/api/schema.ts
```

### 4.6 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic API calls | `fetch` (built-in) or Axios |
| Centralized headers/error handling | Axios interceptors |
| Caching, retries, loading state | TanStack Query |
| Optimistic UI updates | TanStack Query `useMutation` + `invalidateQueries` |
| Type-safe API contracts | OpenAPI-generated TypeScript types |

---

## 5. Python Web Apps

Python apps sit on both sides: **exposing** APIs (Flask/Django/FastAPI) and **consuming** external ones (via `requests`/`httpx`).

### 5.1 Exposing an API — FastAPI (typed, auto-documented)
```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="Products API", version="1.0.0")

class Product(BaseModel):
    id: int
    name: str
    price: float

@app.get("/products/{product_id}", response_model=Product)
def get_product(product_id: int):
    return product_repo.get(product_id)
```
> FastAPI auto-generates OpenAPI docs at `/docs` and `/redoc` from your Pydantic models — no extra setup needed.

### 5.2 Exposing an API — Django REST Framework
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(["GET"])
def product_detail(request, product_id):
    product = Product.objects.get(id=product_id)
    return Response({"id": product.id, "name": product.name, "price": product.price})
```

### 5.3 API versioning
```python
# FastAPI — versioned routers
v1_router = APIRouter(prefix="/api/v1")
v2_router = APIRouter(prefix="/api/v2")
app.include_router(v1_router)
app.include_router(v2_router)
```
```python
# Django REST Framework — URL-based versioning
REST_FRAMEWORK = {
    "DEFAULT_VERSIONING_CLASS": "rest_framework.versioning.URLPathVersioning",
}
```

### 5.4 Rate limiting an exposed API — `slowapi` (FastAPI)
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.get("/products")
@limiter.limit("100/minute")
async def list_products(request: Request):
    ...
```

### 5.5 Consuming external APIs — `httpx` with retries
```python
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
async def fetch_exchange_rate():
    async with httpx.AsyncClient(timeout=10) as client:
        response = await client.get("https://api.exchangerate.example.com/latest")
        response.raise_for_status()
        return response.json()
```

### 5.6 Centralized error handling — FastAPI exception handlers
```python
@app.exception_handler(httpx.HTTPStatusError)
def handle_upstream_error(request, exc):
    return JSONResponse(status_code=502, content={"error": "Upstream API failed"})
```

### 5.7 Caching responses
```python
from fastapi_cache.decorator import cache

@app.get("/products")
@cache(expire=60)  # seconds
async def list_products():
    return await product_repo.list_all()
```

### 5.8 API documentation
```python
# FastAPI: automatic, generated from type hints/Pydantic models — no config needed
# Django REST Framework:
INSTALLED_APPS += ["drf_spectacular"]
REST_FRAMEWORK["DEFAULT_SCHEMA_CLASS"] = "drf_spectacular.openapi.AutoSchema"
```

### 5.9 Recommendation summary
| Need | Recommended approach |
|---|---|
| Exposing a typed, self-documenting API | FastAPI + Pydantic |
| Exposing an API in an existing Django app | Django REST Framework |
| Versioning | Prefixed routers (FastAPI) / `URLPathVersioning` (DRF) |
| Rate limiting your API | `slowapi` (FastAPI) / `django-ratelimit` |
| Calling external APIs reliably | `httpx` + `tenacity` for retries |
| Caching responses | `fastapi-cache2` / Django's cache framework |
| API docs | FastAPI's built-in OpenAPI / `drf-spectacular` for DRF |

---

## 6. API Management in Gen AI Applications

Calling an LLM provider is "just an API call," but at scale it needs the same discipline as any external API — plus Gen AI–specific concerns: token/cost budgets, prompt versioning, and multi-provider fallback.

### 6.1 Basic call with a client SDK
```python
import anthropic

client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1000,
    messages=[{"role": "user", "content": "Summarize this document."}],
)
```

### 6.2 Retries & backoff for transient failures (rate limits, timeouts)
```python
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type
import anthropic

@retry(
    retry=retry_if_exception_type((anthropic.RateLimitError, anthropic.APITimeoutError)),
    wait=wait_exponential(multiplier=1, min=2, max=30),
    stop=stop_after_attempt(5),
)
def call_model(prompt: str):
    return client.messages.create(model="claude-sonnet-5", max_tokens=1000,
                                   messages=[{"role": "user", "content": prompt}])
```
> Most official SDKs (including `anthropic`) already retry transient errors automatically with sensible defaults — check the SDK's `max_retries` option before adding your own layer on top.

### 6.3 Rate limiting outbound calls (respecting provider limits)
```python
from asyncio import Semaphore

request_limiter = Semaphore(10)  # cap concurrent in-flight requests

async def call_model_limited(prompt: str):
    async with request_limiter:
        return await client.messages.create(model="claude-sonnet-5", max_tokens=1000,
                                             messages=[{"role": "user", "content": prompt}])
```

### 6.4 Multi-provider routing & fallback — LiteLLM
For apps that call multiple model providers (or need automatic failover), a unified API layer avoids hand-writing per-provider clients.

```python
from litellm import completion

response = completion(
    model="claude-sonnet-5",
    messages=[{"role": "user", "content": "Hello"}],
    fallbacks=["gpt-4o", "gemini-1.5-pro"],  # tries next provider on failure
)
```

### 6.5 Cost & token budget tracking
```python
def call_with_budget_check(prompt: str, user_id: str, max_tokens_per_day: int = 100_000):
    used_today = get_token_usage(user_id)
    if used_today >= max_tokens_per_day:
        raise BudgetExceededError(user_id)

    response = client.messages.create(model="claude-sonnet-5", max_tokens=1000,
                                       messages=[{"role": "user", "content": prompt}])
    record_token_usage(user_id, response.usage.input_tokens + response.usage.output_tokens)
    return response
```

### 6.6 Streaming responses (reduces perceived latency)
```python
with client.messages.stream(
    model="claude-sonnet-5",
    max_tokens=1000,
    messages=[{"role": "user", "content": "Write a short poem."}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### 6.7 Structured output validation — `instructor`
Ensures the model's response conforms to a schema, similar to how you'd validate a REST API response body.

```python
import instructor
from pydantic import BaseModel

class ProductInfo(BaseModel):
    name: str
    price: float

client = instructor.from_anthropic(anthropic.Anthropic())
product = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=500,
    response_model=ProductInfo,
    messages=[{"role": "user", "content": "Extract product info: Widget, $19.99"}],
)
```

### 6.8 Exposing your own Gen AI feature as an API
Wrap the model call behind your own versioned, rate-limited, documented endpoint — same practices as Section 5.

```python
@app.post("/api/v1/chat", response_model=ChatResponse)
@limiter.limit("20/minute")
async def chat(request: Request, body: ChatRequest, user=Depends(get_current_user)):
    response = client.messages.create(
        model="claude-sonnet-5", max_tokens=1000,
        messages=[{"role": "user", "content": body.message}],
    )
    return ChatResponse(reply=response.content[0].text)
```

### 6.9 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic model calls | Official provider SDK (e.g., `anthropic`) |
| Handling rate limits/timeouts | SDK's built-in retry config, or `tenacity` for custom policies |
| Capping concurrency | `asyncio.Semaphore` or a request queue |
| Multi-provider/fallback | LiteLLM (or a similar unified gateway) |
| Cost control | Per-user/per-day token budget tracking |
| Lower perceived latency | Streaming responses |
| Reliable structured output | `instructor` (schema-validated responses) |
| Exposing the feature to your users | Wrap in your own versioned, rate-limited API endpoint |

---

## 7. Cross-Stack Decision Cheat Sheet

| Concern | Blazor | Angular | React | Python | Gen AI |
|---|---|---|---|---|---|
| Basic HTTP client | `IHttpClientFactory` | `HttpClient` service | `fetch` / Axios | `httpx` / `requests` | Provider SDK (`anthropic`) |
| Centralized handling | `DelegatingHandler` | `HttpInterceptorFn` | Axios interceptors | FastAPI exception handlers | SDK error types + custom wrapper |
| Retries/resilience | Polly | RxJS `retry()` | TanStack Query `retry` | `tenacity` | SDK `max_retries` / `tenacity` |
| Caching | manual / `IMemoryCache` | `shareReplay()` / TanStack Query | TanStack Query | `fastapi-cache2` / Django cache | Token/response caching layer |
| Rate limiting (exposing) | `RateLimiter` middleware | n/a (client-side) | n/a (client-side) | `slowapi` / `django-ratelimit` | Own endpoint's rate limiter |
| Typed clients | Refit | OpenAPI codegen | OpenAPI codegen (`openapi-typescript`) | Pydantic models (FastAPI) | `instructor` schema validation |
| Docs | Swashbuckle/OpenAPI | n/a (consumes docs) | n/a (consumes docs) | FastAPI auto-docs / `drf-spectacular` | Model/tool schemas (JSON Schema) |
| Multi-provider routing | n/a | n/a | n/a | n/a | LiteLLM |

---

## 8. Library & Package Reference

Versions verified against npm/NuGet/PyPI in **August 2026**.

### 8.1 Blazor (C# / NuGet)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| Built-in (`System.Net.Http.Json`) | JSON HTTP calls via `HttpClient` | ships with .NET SDK | — |
| `Polly` | Resilience: retry, circuit breaker, timeout, fallback | 8.7.0 | `dotnet add package Polly` |
| `Polly.RateLimiting` | Rate-limiting resilience strategy for Polly pipelines | 8.7.0 | `dotnet add package Polly.RateLimiting` |
| `Refit` | Declarative, typed REST client | check NuGet | `dotnet add package Refit` |
| `Refit.HttpClientFactory` | DI integration for Refit clients | check NuGet | `dotnet add package Refit.HttpClientFactory` |
| `Swashbuckle.AspNetCore` | Swagger/OpenAPI generation + UI | check NuGet | `dotnet add package Swashbuckle.AspNetCore` |
| `Asp.Versioning.Http` | API versioning for minimal APIs/controllers | check NuGet | `dotnet add package Asp.Versioning.Http` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| Built-in | `HttpClient.GetFromJsonAsync<T>()` | GETs a URL and deserializes the JSON response. |
| Built-in | `HttpClient.PostAsJsonAsync()` | POSTs an object serialized as JSON. |
| Built-in | `IHttpClientFactory.CreateClient(name)` | Retrieves a pre-configured named `HttpClient`. |
| Polly | `AddResilienceHandler(name, builder)` | Attaches a resilience pipeline (retry/breaker/timeout) to a client. |
| Polly | `builder.AddRetry(options)` | Adds automatic retry behavior with backoff. |
| Polly | `builder.AddCircuitBreaker(options)` | Stops calling a failing dependency temporarily. |
| Refit | `[Get("/path/{id}")]` | Declares a typed GET endpoint on an interface. |
| Refit | `AddRefitClient<T>()` | Registers a Refit interface as an injectable typed client. |
| Built-in (minimal APIs) | `app.MapGroup(prefix)` | Groups related endpoints under a shared route prefix. |
| RateLimiter | `AddFixedWindowLimiter(name, opts)` | Configures a fixed-window rate limit policy. |

### 8.2 Angular (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `@angular/common/http` | Built-in HTTP client | ships with `@angular/core` | — |
| `openapi-typescript-codegen` | Generates typed API clients from an OpenAPI spec | check npm | `npm i -D openapi-typescript-codegen` |
| `@tanstack/angular-query-experimental` | Caching/retry layer for server state | tracks `@tanstack/query-core` 5.101.x | `npm i @tanstack/angular-query-experimental` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `@angular/common/http` | `HttpClient.get<T>(url)` | Issues a typed GET request, returns an Observable. |
| `@angular/common/http` | `HttpClient.post<T>(url, body)` | Issues a typed POST request. |
| `@angular/common/http` | `HttpInterceptorFn` | Function-based interceptor for headers/errors on every request. |
| RxJS | `retry({count, delay})` | Automatically retries a failed request N times. |
| RxJS | `shareReplay(1)` | Caches and replays the latest emitted response to new subscribers. |
| RxJS | `catchError(fn)` | Handles/transforms errors in an Observable pipeline. |

Check latest: `npm view openapi-typescript-codegen version`

### 8.3 React (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `axios` | Promise-based HTTP client with interceptors | 1.19.0 | `npm i axios` |
| `@tanstack/react-query` | Data fetching, caching, retries, mutations | 5.101.4 | `npm i @tanstack/react-query` |
| `openapi-typescript` | Generates TypeScript types from an OpenAPI spec | check npm | `npm i -D openapi-typescript` |
| `openapi-fetch` | Lightweight, type-safe fetch client using generated types | check npm | `npm i openapi-fetch` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `axios` | `axios.create(config)` | Creates a pre-configured client instance (base URL, timeout, headers). |
| `axios` | `instance.interceptors.request.use(fn)` | Runs logic (e.g., attach auth header) before every request. |
| `axios` | `instance.interceptors.response.use(onFulfilled, onRejected)` | Centralizes success/error handling for every response. |
| `@tanstack/react-query` | `useQuery({queryKey, queryFn})` | Fetches and caches data with automatic retries/refetch. |
| `@tanstack/react-query` | `useMutation({mutationFn})` | Manages create/update/delete calls and their pending/error state. |
| `@tanstack/react-query` | `queryClient.invalidateQueries()` | Forces cached queries to refetch after a mutation. |
| `openapi-fetch` | `createClient<paths>()` | Builds a fully typed client from generated OpenAPI types. |

Check latest: `npm view axios version` / `npm view @tanstack/react-query version`

### 8.4 Python (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `fastapi` | Typed API framework with automatic OpenAPI docs | check PyPI | `pip install fastapi` |
| `djangorestframework` | REST framework for Django | check PyPI | `pip install djangorestframework` |
| `drf-spectacular` | OpenAPI schema generation for DRF | check PyPI | `pip install drf-spectacular` |
| `httpx` | Modern sync/async HTTP client | check PyPI | `pip install httpx` |
| `requests` | Classic sync HTTP client | check PyPI | `pip install requests` |
| `tenacity` | General-purpose retry library (backoff, jitter, stop conditions) | check PyPI | `pip install tenacity` |
| `slowapi` | Rate limiting for FastAPI/Starlette | 0.1.10 | `pip install slowapi` |
| `fastapi-cache2` | Response caching for FastAPI | check PyPI | `pip install fastapi-cache2` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `fastapi` | `@app.get(path, response_model=Model)` | Declares a typed, auto-documented GET endpoint. |
| `fastapi` | `APIRouter(prefix=...)` | Groups and versions related endpoints. |
| `djangorestframework` | `@api_view(["GET"])` | Declares a function-based DRF endpoint. |
| `djangorestframework` | `serializers.Serializer` | Validates and (de)serializes request/response data. |
| `httpx` | `httpx.AsyncClient().get(url)` | Issues an async HTTP GET request. |
| `httpx` | `response.raise_for_status()` | Raises an exception for 4xx/5xx responses. |
| `tenacity` | `@retry(stop=..., wait=...)` | Decorator that retries a function with configurable backoff. |
| `slowapi` | `@limiter.limit("100/minute")` | Applies a rate limit to a specific endpoint. |
| `fastapi-cache2` | `@cache(expire=seconds)` | Caches an endpoint's response for a given duration. |

Check latest: `pip index versions fastapi` / `pip index versions httpx`

### 8.5 Gen AI (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `anthropic` | Official Claude SDK (built-in retries, streaming) | check PyPI | `pip install anthropic` |
| `litellm` | Unified API across LLM providers, with fallback/routing | check PyPI | `pip install litellm` |
| `instructor` | Schema-validated structured output from LLM calls | check PyPI | `pip install instructor` |
| `tenacity` | Custom retry/backoff policies around model calls | check PyPI | `pip install tenacity` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `anthropic` | `Anthropic(api_key=..., max_retries=...)` | Initializes the client with a built-in retry policy. |
| `anthropic` | `client.messages.stream()` | Streams a model response incrementally. |
| `anthropic` | `response.usage.input_tokens` / `.output_tokens` | Reports token counts for cost tracking. |
| `litellm` | `completion(model, messages, fallbacks=[...])` | Calls a model with automatic fallback to alternate providers. |
| `instructor` | `client.messages.create(response_model=Model)` | Returns a validated Pydantic object instead of raw text. |
| `tenacity` | `retry_if_exception_type(ExceptionType)` | Retries only on specific exception types (e.g., rate limits). |

Check latest: `pip index versions litellm` / `pip index versions instructor`

---

*This guide covers the mainstream API management patterns and package versions as of August 2026. Library APIs (especially in the fast-moving Gen AI space) change frequently — check current docs/registries before pinning versions in production.*
