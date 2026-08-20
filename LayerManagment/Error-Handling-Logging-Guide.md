# Error Handling & Logging Mechanisms Across Web Stacks
### Blazor (C#/.NET) · Angular · React · Python Web Apps · Gen AI Applications

---

## 1. What Error Handling & Logging Cover

These two concerns are usually designed together: logging tells you *what happened*, error handling decides *what to do about it* — and good systems make sure every unhandled error is also logged somewhere a human can see it.

| Concern | What it solves |
|---|---|
| Try/catch at the right boundary | Fail gracefully instead of crashing the whole app |
| Global/unhandled error capture | Catches what individual try/catch blocks miss |
| Structured logging | Machine-parseable logs (JSON) instead of free-text strings |
| Log levels | Separating noise (Debug) from signal (Error/Critical) |
| Centralized error tracking | Aggregating errors across users/instances (Sentry, Application Insights) |
| Correlation IDs / tracing | Following one request across services and logs |
| User-facing error UX | Showing something useful without leaking internals |
| Gen AI equivalent | Handling model/tool failures, logging prompts/responses, tracing multi-step agent runs |

---

## 2. Blazor (C# / .NET)

.NET's exception model (try/catch) is standard C#; Blazor adds component-level error boundaries, and **Serilog** is the most common structured logging library, often paired with **Sentry** or **Application Insights** for centralized tracking.

### 2.1 Component-level error handling — `ErrorBoundary`
```csharp
<ErrorBoundary>
    <ChildContent>
        <ProductList />
    </ChildContent>
    <ErrorContent Context="ex">
        <p class="error">Something went wrong: @ex.Message</p>
    </ErrorContent>
</ErrorBoundary>
```

### 2.2 Try/catch at the service/API boundary
```csharp
public async Task<Product?> GetProductAsync(int id)
{
    try
    {
        return await httpClient.GetFromJsonAsync<Product>($"products/{id}");
    }
    catch (HttpRequestException ex)
    {
        logger.LogError(ex, "Failed to fetch product {ProductId}", id);
        return null;
    }
}
```

### 2.3 Global exception handling — ASP.NET Core middleware (Blazor Server host)
```csharp
// Program.cs
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var feature = context.Features.Get<IExceptionHandlerFeature>();
        logger.LogError(feature?.Error, "Unhandled exception");
        context.Response.StatusCode = 500;
        await context.Response.WriteAsJsonAsync(new { error = "An unexpected error occurred." });
    });
});
```

### 2.4 Structured logging — Serilog
```csharp
// Program.cs
builder.Host.UseSerilog((context, config) => config
    .MinimumLevel.Information()
    .WriteTo.Console(new JsonFormatter())
    .WriteTo.Seq("http://localhost:5341")
    .Enrich.FromLogContext());
```
```csharp
logger.LogInformation("Order {OrderId} placed by {UserId} for {Amount:C}", order.Id, user.Id, order.Total);
```

### 2.5 Log levels
```csharp
logger.LogTrace("Very detailed diagnostic info");
logger.LogDebug("Debugging info, dev only");
logger.LogInformation("Normal operational event");
logger.LogWarning("Unexpected but recoverable");
logger.LogError(ex, "Operation failed");
logger.LogCritical(ex, "App-wide failure, needs immediate attention");
```

### 2.6 Centralized error tracking — Sentry
```csharp
builder.WebHost.UseSentry(options =>
{
    options.Dsn = "https://your-dsn@sentry.io/id";
    options.TracesSampleRate = 0.2;
});
```

### 2.7 Correlation IDs across a request
```csharp
app.Use(async (context, next) =>
{
    var correlationId = context.Request.Headers["X-Correlation-Id"].FirstOrDefault() ?? Guid.NewGuid().ToString();
    using (LogContext.PushProperty("CorrelationId", correlationId))
    {
        context.Response.Headers["X-Correlation-Id"] = correlationId;
        await next();
    }
});
```

### 2.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Catch render errors in a component tree | `<ErrorBoundary>` |
| Handle expected failure modes | Try/catch at the service/API call site |
| Catch everything else | `UseExceptionHandler` middleware |
| Structured, queryable logs | Serilog with a JSON sink (Seq, Elasticsearch) |
| Centralized error aggregation | Sentry or Application Insights |
| Tracing a request across logs | Correlation ID middleware + `LogContext` |

---

## 3. Angular

Angular centralizes unhandled errors via `ErrorHandler`, HTTP-specific failures via interceptors, and per-view failures via template error states — logging is typically routed through a wrapper service to a backend or a tool like Sentry.

### 3.1 Global error handler
```typescript
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(private logger: LoggingService) {}

  handleError(error: unknown): void {
    console.error(error);
    this.logger.logError(error);
  }
}
```
```typescript
// app.config.ts
providers: [{ provide: ErrorHandler, useClass: GlobalErrorHandler }]
```

### 3.2 HTTP error handling — interceptor
```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      const message = error.status === 0
        ? "Network error — check your connection."
        : `Server error (${error.status})`;
      inject(NotificationService).showError(message);
      inject(LoggingService).logError(error);
      return throwError(() => error);
    })
  );
};
```

### 3.3 Component-level error state
```typescript
export class ProductListComponent {
  error = signal<string | null>(null);
  products = signal<Product[]>([]);

  constructor(private service: ProductsService) {
    this.service.getProducts().pipe(
      catchError((err) => { this.error.set("Could not load products."); return of([]); })
    ).subscribe((data) => this.products.set(data));
  }
}
```

### 3.4 Structured logging service
```typescript
@Injectable({ providedIn: 'root' })
export class LoggingService {
  logError(error: unknown, context?: Record<string, unknown>) {
    const entry = { level: "error", message: String(error), context, timestamp: new Date().toISOString() };
    // send to backend logging endpoint or a provider SDK
    this.http.post("/api/logs", entry).subscribe();
  }
}
```

### 3.5 Centralized error tracking — Sentry Angular
```typescript
Sentry.init({
  dsn: "https://your-dsn@sentry.io/id",
  integrations: [Sentry.browserTracingIntegration()],
  tracesSampleRate: 0.2,
});
```
```typescript
providers: [
  { provide: ErrorHandler, useValue: Sentry.createErrorHandler() },
]
```

### 3.6 Recommendation summary
| Need | Recommended approach |
|---|---|
| Catch anything uncaught | Custom `ErrorHandler` |
| Handle failed HTTP calls consistently | `HttpInterceptorFn` with `catchError` |
| Per-component failure UI | Local error signal/state + `catchError` |
| Centralized log shipping | A `LoggingService` posting to your backend or a provider |
| Production error aggregation | Sentry Angular SDK |

---

## 4. React

React separates **render errors** (caught by Error Boundaries — still class components, since there's no Hook equivalent) from **async/event errors** (caught with normal try/catch), with Sentry as the dominant centralized tracker.

### 4.1 Error boundaries (render errors)
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    logErrorToService(error, info.componentStack);
  }

  render() {
    if (this.state.hasError) return this.props.fallback;
    return this.props.children;
  }
}
```
```jsx
<ErrorBoundary fallback={<ErrorMessage />}>
  <ProductList />
</ErrorBoundary>
```

### 4.2 Async/event error handling
```jsx
async function loadProducts() {
  try {
    const res = await fetch("/api/products");
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    logErrorToService(error);
    throw error; // re-throw so calling code (e.g., TanStack Query) can handle UI state
  }
}
```

### 4.3 Query-level error handling — TanStack Query
```jsx
const { data, error, isError } = useQuery({
  queryKey: ["products"],
  queryFn: loadProducts,
  retry: 2,
});

if (isError) return <ErrorMessage error={error} />;
```

### 4.4 Global unhandled error/rejection listeners
```jsx
window.addEventListener("error", (event) => logErrorToService(event.error));
window.addEventListener("unhandledrejection", (event) => logErrorToService(event.reason));
```

### 4.5 Structured logging helper
```jsx
function logErrorToService(error, extra = {}) {
  const entry = {
    message: error?.message ?? String(error),
    stack: error?.stack,
    ...extra,
    timestamp: new Date().toISOString(),
  };
  navigator.sendBeacon("/api/logs", JSON.stringify(entry));
}
```

### 4.6 Centralized error tracking — Sentry React
```jsx
Sentry.init({
  dsn: "https://your-dsn@sentry.io/id",
  integrations: [Sentry.browserTracingIntegration(), Sentry.reactRouterV7BrowserTracingIntegration()],
  tracesSampleRate: 0.2,
});

export default Sentry.withErrorBoundary(App, { fallback: <ErrorMessage /> });
```

### 4.7 Recommendation summary
| Need | Recommended approach |
|---|---|
| Catch render-time errors | Class-based `ErrorBoundary` |
| Catch async/fetch errors | try/catch at the call site |
| Query/mutation error UI | TanStack Query's `error`/`isError` |
| Catch everything else | `window.onerror` / `unhandledrejection` listeners |
| Production error aggregation | Sentry React SDK |

---

## 5. Python Web Apps

Python's built-in `logging` module underlies most setups; Django, Flask, and FastAPI each add their own exception-handling hooks on top of it.

### 5.1 Basic structured logging setup
```python
import logging
import json_logging  # or structlog / python-json-logger

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("Order placed", extra={"order_id": order.id, "user_id": user.id, "amount": order.total})
```

### 5.2 Log levels
```python
logger.debug("Debugging info, dev only")
logger.info("Normal operational event")
logger.warning("Unexpected but recoverable")
logger.error("Operation failed", exc_info=True)
logger.critical("App-wide failure, needs immediate attention")
```

### 5.3 FastAPI — global exception handlers
```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(Exception)
async def unhandled_exception_handler(request: Request, exc: Exception):
    logger.error("Unhandled exception", exc_info=exc, extra={"path": request.url.path})
    return JSONResponse(status_code=500, content={"error": "Internal server error"})

@app.exception_handler(ValueError)
async def value_error_handler(request: Request, exc: ValueError):
    return JSONResponse(status_code=400, content={"error": str(exc)})
```

### 5.4 Flask — global error handlers
```python
@app.errorhandler(Exception)
def handle_unexpected_error(error):
    app.logger.error("Unhandled exception", exc_info=error)
    return jsonify({"error": "Internal server error"}), 500

@app.errorhandler(404)
def handle_not_found(error):
    return jsonify({"error": "Not found"}), 404
```

### 5.5 Django — middleware-based error handling
```python
class ErrorLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        return self.get_response(request)

    def process_exception(self, request, exception):
        logger.error("Unhandled exception on %s", request.path, exc_info=exception)
        return None  # let Django's default handling continue
```

### 5.6 Try/except at the operation boundary
```python
def get_product(product_id: int):
    try:
        return db.query(Product).filter_by(id=product_id).one()
    except NoResultFound:
        logger.warning("Product not found", extra={"product_id": product_id})
        raise HTTPException(status_code=404, detail="Product not found")
    except SQLAlchemyError as e:
        logger.error("Database error fetching product", exc_info=e)
        raise HTTPException(status_code=500, detail="Internal server error")
```

### 5.7 Structured logging — `structlog`
```python
import structlog

structlog.configure(processors=[structlog.processors.JSONRenderer()])
log = structlog.get_logger()

log.info("order_placed", order_id=order.id, user_id=user.id, amount=order.total)
```

### 5.8 Centralized error tracking — Sentry SDK
```python
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration

sentry_sdk.init(dsn="https://your-dsn@sentry.io/id", integrations=[FastApiIntegration()], traces_sample_rate=0.2)
```

### 5.9 Correlation/request IDs
```python
import contextvars
request_id_var = contextvars.ContextVar("request_id", default=None)

@app.middleware("http")
async def add_request_id(request: Request, call_next):
    request_id = request.headers.get("X-Request-ID", str(uuid4()))
    request_id_var.set(request_id)
    response = await call_next(request)
    response.headers["X-Request-ID"] = request_id
    return response
```

### 5.10 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic logging | Standard library `logging` module |
| Structured/JSON logs | `structlog` or `python-json-logger` |
| Global error handling (FastAPI) | `@app.exception_handler()` |
| Global error handling (Flask) | `@app.errorhandler()` |
| Global error handling (Django) | Middleware `process_exception` hook |
| Centralized error aggregation | Sentry SDK (framework-specific integration) |
| Tracing a request | Context vars + `X-Request-ID` header propagation |

---

## 6. Error Handling & Logging in Gen AI Applications

Gen AI systems add failure modes beyond typical HTTP errors — rate limits, content filtering, malformed structured output, tool execution failures, and multi-step agent runs that need to be traced end-to-end, not just logged line by line.

### 6.1 Handling model provider errors
```python
import anthropic

try:
    response = client.messages.create(
        model="claude-sonnet-5", max_tokens=1000,
        messages=[{"role": "user", "content": prompt}],
    )
except anthropic.RateLimitError as e:
    logger.warning("Rate limited by provider", exc_info=e)
    raise ServiceBusyError("Please try again shortly.")
except anthropic.APIConnectionError as e:
    logger.error("Could not reach model provider", exc_info=e)
    raise ServiceUnavailableError()
except anthropic.APIStatusError as e:
    logger.error("Model provider returned an error", extra={"status_code": e.status_code, "body": e.body})
    raise
```

### 6.2 Handling tool/function call failures
Tool execution failures shouldn't crash the agent loop — feed the error back to the model as a tool result so it can decide how to proceed (retry, ask the user, try another approach).

```python
def execute_tool_safely(tool_name: str, tool_input: dict) -> dict:
    try:
        return {"result": TOOLS[tool_name](**tool_input)}
    except Exception as e:
        logger.error("Tool execution failed", extra={"tool": tool_name, "input": tool_input}, exc_info=e)
        return {"error": f"Tool '{tool_name}' failed: {str(e)}"}
```

### 6.3 Validating structured output
```python
from pydantic import ValidationError

def parse_model_output(raw_text: str, schema: type[BaseModel]):
    try:
        return schema.model_validate_json(raw_text)
    except ValidationError as e:
        logger.warning("Model returned invalid structured output", extra={"raw": raw_text, "errors": e.errors()})
        raise MalformedModelOutputError()
```

### 6.4 Logging prompts and responses (with care)
Log enough to debug quality issues, but redact or avoid logging sensitive user content in plaintext, and respect data-retention policies.

```python
logger.info(
    "model_call",
    extra={
        "model": "claude-sonnet-5",
        "input_tokens": response.usage.input_tokens,
        "output_tokens": response.usage.output_tokens,
        "latency_ms": latency_ms,
        "user_id": user_id,
        # avoid logging full prompt/response content in plaintext for regulated data;
        # log a hash or truncated/redacted version if you need traceability
    },
)
```

### 6.5 Tracing multi-step agent runs — LangSmith / Langfuse
Standard logs don't capture the shape of an agent run (which tools were called, in what order, with what intermediate reasoning) — dedicated LLM observability tools trace the whole run as a unit.

```python
from langfuse.decorators import observe

@observe()
def run_agent(query: str):
    plan = generate_plan(query)
    results = [execute_tool_safely(step.tool, step.input) for step in plan]
    return synthesize_answer(results)
```

### 6.6 Fallback behavior on failure
```python
def get_response_with_fallback(prompt: str) -> str:
    try:
        return call_primary_model(prompt)
    except (anthropic.RateLimitError, anthropic.APIConnectionError):
        logger.warning("Primary model unavailable, falling back")
        try:
            return call_fallback_model(prompt)
        except Exception as e:
            logger.error("Fallback also failed", exc_info=e)
            return "I'm having trouble responding right now — please try again in a moment."
```

### 6.7 Recommendation summary
| Need | Recommended approach |
|---|---|
| Handling provider-side failures | Catch specific SDK exception types (`RateLimitError`, `APIConnectionError`, etc.) |
| Tool execution failures | Return the error to the model as a tool result rather than crashing the run |
| Malformed structured output | Validate with Pydantic/schema; catch and log `ValidationError` |
| Prompt/response logging | Log metadata (tokens, latency, model) freely; redact/avoid logging raw sensitive content |
| Tracing a multi-step agent run | LangSmith, Langfuse, or OpenTelemetry-based LLM tracing |
| Graceful degradation | Fallback to a secondary model or a safe default message |

---

## 7. Cross-Stack Decision Cheat Sheet

| Concern | Blazor | Angular | React | Python | Gen AI |
|---|---|---|---|---|---|
| Component/render error capture | `<ErrorBoundary>` | template `@if`/error state | class `ErrorBoundary` | n/a (server-rendered) | n/a |
| Global unhandled error capture | `UseExceptionHandler` middleware | custom `ErrorHandler` | `window.onerror`/`unhandledrejection` | framework exception handlers | catch provider SDK exception types |
| Structured logging | Serilog | `LoggingService` → backend | `logErrorToService` → backend | `structlog` / `logging` | log tokens/latency/metadata (redact content) |
| Log levels | `LogTrace`…`LogCritical` | console + backend levels | console + backend levels | `DEBUG`…`CRITICAL` | same, plus model/tool-specific events |
| Centralized error tracking | Sentry .NET | Sentry Angular | Sentry React | Sentry Python (framework integration) | Sentry + LangSmith/Langfuse for agent traces |
| Correlation/request tracing | `LogContext` + correlation ID middleware | pass ID via interceptor headers | pass ID via fetch/axios headers | `contextvars` + `X-Request-ID` | trace ID spanning the full agent run |

---

## 8. Library & Package Reference

Versions verified against npm/NuGet/PyPI in **August 2026**.

### 8.1 Blazor (C# / NuGet)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `Serilog.AspNetCore` | Structured logging for ASP.NET Core/Blazor | tracks `Serilog` 4.4.0 | `dotnet add package Serilog.AspNetCore` |
| `Serilog.Sinks.Seq` | Ships logs to a Seq server for querying | check NuGet | `dotnet add package Serilog.Sinks.Seq` |
| `Sentry.AspNetCore` | Centralized error tracking for ASP.NET Core/Blazor | 6.6.0 | `dotnet add package Sentry.AspNetCore` |
| `Sentry.Serilog` | Routes Serilog events into Sentry | 6.5.0 | `dotnet add package Sentry.Serilog` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| Built-in | `<ErrorBoundary>` | Catches render-time exceptions within a component subtree. |
| Built-in | `app.UseExceptionHandler()` | Registers global middleware to catch unhandled request exceptions. |
| Built-in | `ILogger.LogError(ex, template, args)` | Logs an error with structured, named parameters. |
| Serilog | `WriteTo.Console(new JsonFormatter())` | Outputs structured JSON logs to the console. |
| Serilog | `LogContext.PushProperty(name, value)` | Attaches a contextual property (e.g., correlation ID) to all logs in scope. |
| Sentry | `UseSentry(options => ...)` | Wires up automatic exception capture and performance tracing. |
| Sentry | `SentrySdk.CaptureException(ex)` | Manually reports an exception to Sentry. |

### 8.2 Angular (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `@sentry/angular` | Centralized error tracking for Angular | check npm | `npm i @sentry/angular` |
| `ngx-logger` | Configurable logging service with levels and remote logging | check npm | `npm i ngx-logger` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `@angular/core` | `ErrorHandler` (class to extend) | Global hook for catching any uncaught application error. |
| `@angular/common/http` | `catchError()` (RxJS operator) | Intercepts and handles errors in an HTTP observable stream. |
| `@sentry/angular` | `Sentry.init(options)` | Initializes error/performance tracking for the app. |
| `@sentry/angular` | `Sentry.createErrorHandler()` | Produces an `ErrorHandler` that reports errors to Sentry. |
| `@sentry/angular` | `Sentry.captureException(error)` | Manually reports a specific error. |
| `ngx-logger` | `logger.error(message, ...extra)` | Logs an error with configurable level and remote transport. |

Check latest: `npm view @sentry/angular version`

### 8.3 React (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `@sentry/react` | Centralized error tracking for React | check npm | `npm i @sentry/react` |
| `react-error-boundary` | Hook-friendly wrapper around Error Boundaries | check npm | `npm i react-error-boundary` |
| `loglevel` | Lightweight leveled logging for the browser | check npm | `npm i loglevel` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| React (built-in) | `static getDerivedStateFromError()` | Updates state to show a fallback UI after a render error. |
| React (built-in) | `componentDidCatch(error, info)` | Receives the caught error and component stack for logging. |
| `react-error-boundary` | `<ErrorBoundary FallbackComponent={...}>` | Function-component-friendly error boundary wrapper. |
| `react-error-boundary` | `useErrorBoundary()` | Hook to manually trigger the nearest error boundary. |
| `@sentry/react` | `Sentry.init(options)` | Initializes error/performance tracking for the app. |
| `@sentry/react` | `Sentry.withErrorBoundary(Component, options)` | Wraps a component with Sentry-integrated error capture. |
| `@tanstack/react-query` | `error` / `isError` (from `useQuery`) | Exposes fetch/mutation errors for UI handling. |

Check latest: `npm view @sentry/react version`

### 8.4 Python (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `structlog` | Structured, composable logging | check PyPI | `pip install structlog` |
| `python-json-logger` | JSON formatter for the standard `logging` module | check PyPI | `pip install python-json-logger` |
| `sentry-sdk` | Centralized error tracking (Flask/Django/FastAPI integrations) | check PyPI | `pip install sentry-sdk` |
| `loguru` | Simplified, batteries-included logging alternative | check PyPI | `pip install loguru` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `logging` (built-in) | `logger.error(msg, exc_info=True)` | Logs an error including the exception traceback. |
| `logging` (built-in) | `logging.basicConfig(level=...)` | Configures the root logger's level and handlers. |
| `structlog` | `structlog.get_logger()` | Retrieves a structured logger bound to the current context. |
| `structlog` | `log.info(event, **kwargs)` | Logs a structured event with arbitrary key/value context. |
| FastAPI | `@app.exception_handler(ExceptionType)` | Registers a handler for a specific exception type app-wide. |
| Flask | `@app.errorhandler(code_or_exception)` | Registers a handler for an HTTP status code or exception type. |
| Django | `process_exception(request, exception)` | Middleware hook invoked when a view raises an unhandled exception. |
| `sentry-sdk` | `sentry_sdk.init(dsn=..., integrations=[...])` | Initializes automatic error capture for the framework in use. |
| `sentry-sdk` | `sentry_sdk.capture_exception(e)` | Manually reports a specific exception. |

Check latest: `pip index versions structlog` / `pip index versions sentry-sdk`

### 8.5 Gen AI (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `anthropic` | Model SDK — typed exceptions for provider-side failures | check PyPI | `pip install anthropic` |
| `langfuse` | LLM observability/tracing for prompts, tools, and agent runs | check PyPI | `pip install langfuse` |
| `tenacity` | Retry/backoff wrapper for transient model-call failures | check PyPI | `pip install tenacity` |
| `pydantic` | Schema validation for structured model output | check PyPI | `pip install pydantic` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `anthropic` | `anthropic.RateLimitError` | Exception raised when the provider's rate limit is hit. |
| `anthropic` | `anthropic.APIConnectionError` | Exception raised when the request can't reach the provider. |
| `anthropic` | `anthropic.APIStatusError.status_code` / `.body` | Details of a non-2xx response from the provider. |
| `langfuse` | `@observe()` (decorator) | Traces a function (e.g., an agent run) as a unit in Langfuse. |
| `langfuse` | `langfuse.trace()` | Manually starts a trace to group related model/tool calls. |
| `pydantic` | `Model.model_validate_json(text)` | Parses and validates model output against a schema, raising on mismatch. |
| `pydantic` | `ValidationError.errors()` | Returns structured detail on what failed validation. |
| `tenacity` | `@retry(retry=retry_if_exception_type(...))` | Retries a model call only for specified transient exception types. |

Check latest: `pip index versions langfuse` / `pip index versions anthropic`

---

*This guide covers the mainstream error handling and logging patterns and package versions as of August 2026. Library APIs (especially in the fast-moving Gen AI observability space) change frequently — check current docs/registries before pinning versions in production. Always avoid logging secrets, passwords, tokens, or unredacted personal/sensitive data, regardless of stack.*
