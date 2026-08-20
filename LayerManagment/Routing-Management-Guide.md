# Routing Management Across Web Stacks
### Blazor (C#/.NET) · Angular · React · Python Web Apps · Gen AI Applications

---

## 1. Why Routing Matters

Routing maps a URL (or an intent, in Gen AI systems) to the code that should handle it — a page, a component, an API endpoint, or an agent/model. Good routing gives you:

| Concern | Example |
|---|---|
| Navigation | Mapping `/products/42` to a product detail page |
| Parameters | Extracting `42` as a route parameter |
| Nesting/layouts | Shared shell (nav bar) around child pages |
| Guards | Blocking `/admin` unless the user is authorized |
| Lazy loading | Only downloading a route's code when it's visited |
| Data loading | Fetching data before/while rendering a route |
| API routing | Mapping `POST /api/orders` to a handler function |
| Intent routing (Gen AI) | Deciding which tool, agent, or model should handle a prompt |

Each section below goes from **basic routing** to **guards, lazy loading, and nested/data routing**.

---

## 2. Blazor (C# / .NET)

Blazor routing is built into the framework via the `Router` component and `@page` directives — no extra package needed for basic routing.

### 2.1 Basic route
```csharp
@page "/products"

<h3>Products</h3>
```

### 2.2 Route parameters
```csharp
@page "/products/{id:int}"

<h3>Product @Id</h3>

@code {
    [Parameter] public int Id { get; set; }
}
```

### 2.3 Multiple route templates for one component
```csharp
@page "/orders"
@page "/orders/{status}"

@code {
    [Parameter] public string? Status { get; set; }
}
```

### 2.4 Root router setup
```csharp
<!-- App.razor -->
<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing here.</p>
        </LayoutView>
    </NotFound>
</Router>
```

### 2.5 Programmatic navigation
```csharp
@inject NavigationManager Navigation

<button @onclick="GoToProduct">View</button>

@code {
    private void GoToProduct() => Navigation.NavigateTo("/products/42");
}
```

### 2.6 Route guards (authorization)
```csharp
@page "/admin"
@attribute [Authorize(Roles = "Admin")]

<h3>Admin Panel</h3>
```

```csharp
<!-- App.razor: wrap Router in AuthorizeRouteView -->
<Found Context="routeData">
    <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
        <NotAuthorized>
            <p>You are not authorized.</p>
        </NotAuthorized>
    </AuthorizeRouteView>
</Found>
```

### 2.7 Lazy loading (Blazor WebAssembly)
```csharp
// Program.cs
builder.Services.AddSingleton(new LazyAssemblyLoader(...));
```
```csharp
@page "/reports"
@inject LazyAssemblyLoader AssemblyLoader

@code {
    protected override async Task OnInitializedAsync()
    {
        await AssemblyLoader.LoadAssembliesAsync(new[] { "ReportsModule.dll" });
    }
}
```

### 2.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic page routing | `@page` directive |
| Parameters | `{param}` route tokens + `[Parameter]` |
| Auth-gated pages | `[Authorize]` + `AuthorizeRouteView` |
| Reduce initial bundle (WASM) | `LazyAssemblyLoader` |
| Programmatic navigation | `NavigationManager.NavigateTo()` |

---

## 3. Angular

Angular ships a full-featured router (`@angular/router`) as part of the framework — configuration is typically centralized in route arrays.

### 3.1 Basic route configuration
```typescript
// app.routes.ts
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'products', component: ProductListComponent },
  { path: 'products/:id', component: ProductDetailComponent },
  { path: '**', component: NotFoundComponent }, // wildcard/404
];
```
```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)],
};
```

### 3.2 Route parameters
```typescript
export class ProductDetailComponent {
  id = input.required<string>(); // with withComponentInputBinding()
  // or classic approach:
  constructor(private route: ActivatedRoute) {
    this.route.paramMap.subscribe(params => {
      const id = params.get('id');
    });
  }
}
```

### 3.3 Nested (child) routes + layouts
```typescript
export const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardLayoutComponent,
    children: [
      { path: '', component: DashboardHomeComponent },
      { path: 'settings', component: SettingsComponent },
    ],
  },
];
```
```html
<!-- DashboardLayoutComponent template -->
<nav>...</nav>
<router-outlet></router-outlet>
```

### 3.4 Route guards
```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() ? true : router.parseUrl('/login');
};
```
```typescript
{ path: 'admin', component: AdminComponent, canActivate: [authGuard] }
```

### 3.5 Lazy loading
```typescript
{
  path: 'reports',
  loadComponent: () => import('./reports/reports.component').then(m => m.ReportsComponent),
}
```

### 3.6 Data resolvers (pre-fetch before navigation)
```typescript
export const productResolver: ResolveFn<Product> = (route) => {
  const service = inject(ProductService);
  return service.getById(route.paramMap.get('id')!);
};
```
```typescript
{ path: 'products/:id', component: ProductDetailComponent, resolve: { product: productResolver } }
```

### 3.7 Programmatic navigation
```typescript
constructor(private router: Router) {}
goToProduct(id: string) {
  this.router.navigate(['/products', id]);
}
```

### 3.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic routing | `Routes` array + `provideRouter()` |
| Nested layouts | `children` + `<router-outlet>` |
| Auth-gated routes | `CanActivateFn` guards |
| Reduce initial bundle | `loadComponent` / `loadChildren` |
| Pre-fetch data | `ResolveFn` resolvers |

---

## 4. React

React has no built-in router — routing is handled by a library, overwhelmingly **React Router** (now in its v7/v8 "framework mode" generation) or, increasingly, **TanStack Router** for type-safe routing.

### 4.1 Basic route configuration (React Router, declarative mode)
```jsx
import { createBrowserRouter, RouterProvider } from "react-router";

const router = createBrowserRouter([
  { path: "/", element: <Home /> },
  { path: "/products", element: <ProductList /> },
  { path: "/products/:id", element: <ProductDetail /> },
  { path: "*", element: <NotFound /> },
]);

createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```

### 4.2 Route parameters
```jsx
import { useParams } from "react-router";

function ProductDetail() {
  const { id } = useParams();
  return <h3>Product {id}</h3>;
}
```

### 4.3 Nested routes + layouts
```jsx
const router = createBrowserRouter([
  {
    path: "/dashboard",
    element: <DashboardLayout />,
    children: [
      { index: true, element: <DashboardHome /> },
      { path: "settings", element: <Settings /> },
    ],
  },
]);
```
```jsx
function DashboardLayout() {
  return (
    <div>
      <nav>...</nav>
      <Outlet />
    </div>
  );
}
```

### 4.4 Route guards (protected routes)
```jsx
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useAuth();
  return isAuthenticated ? children : <Navigate to="/login" replace />;
}

// usage in route config
{ path: "/admin", element: <ProtectedRoute><Admin /></ProtectedRoute> }
```

### 4.5 Lazy loading
```jsx
import { lazy, Suspense } from "react";
const Reports = lazy(() => import("./Reports"));

{
  path: "/reports",
  element: (
    <Suspense fallback={<Spinner />}>
      <Reports />
    </Suspense>
  ),
}
```

### 4.6 Data loading (loaders — framework/data mode)
```jsx
const router = createBrowserRouter([
  {
    path: "/products/:id",
    element: <ProductDetail />,
    loader: async ({ params }) => fetch(`/api/products/${params.id}`),
  },
]);

function ProductDetail() {
  const product = useLoaderData();
  return <h3>{product.name}</h3>;
}
```

### 4.7 Programmatic navigation
```jsx
import { useNavigate } from "react-router";

function Buy() {
  const navigate = useNavigate();
  return <button onClick={() => navigate("/checkout")}>Buy</button>;
}
```

### 4.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Basic routing | React Router `createBrowserRouter` |
| Nested layouts | `children` + `<Outlet />` |
| Auth-gated routes | Wrapper component + `<Navigate>` |
| Reduce initial bundle | `React.lazy()` + `<Suspense>` |
| Pre-fetch route data | `loader` (React Router data mode) |
| Type-safe, file-based routing | TanStack Router (alternative to React Router) |

---

## 5. Python Web Apps

Python frameworks route incoming HTTP requests to handler functions — the mechanics differ noticeably between Flask, Django, FastAPI, and Streamlit.

### 5.1 Flask — decorator-based routing
```python
from flask import Flask
app = Flask(__name__)

@app.route("/products/<int:product_id>")
def product_detail(product_id):
    return {"id": product_id}

@app.route("/orders", methods=["GET", "POST"])
def orders():
    ...
```

### 5.2 Flask — Blueprints (modular/nested routing)
```python
# products/routes.py
products_bp = Blueprint("products", __name__, url_prefix="/products")

@products_bp.route("/<int:id>")
def detail(id):
    return {"id": id}
```
```python
# app.py
app.register_blueprint(products_bp)
```

### 5.3 Django — URLconf
```python
# urls.py
urlpatterns = [
    path("products/<int:product_id>/", views.product_detail, name="product_detail"),
    path("orders/", views.orders, name="orders"),
    path("admin/", include("myapp.admin_urls")),  # nested/included routing
]
```
```python
# views.py
def product_detail(request, product_id):
    return JsonResponse({"id": product_id})
```

### 5.4 FastAPI — path operations + APIRouter (nested routing)
```python
from fastapi import FastAPI, APIRouter

app = FastAPI()
router = APIRouter(prefix="/products", tags=["products"])

@router.get("/{product_id}")
def get_product(product_id: int):
    return {"id": product_id}

app.include_router(router)
```

### 5.5 Route guards (auth-gated routes)
```python
# FastAPI
from fastapi import Depends, HTTPException

def require_admin(user = Depends(get_current_user)):
    if not user.is_admin:
        raise HTTPException(status_code=403)
    return user

@router.get("/admin", dependencies=[Depends(require_admin)])
def admin_panel():
    ...
```
```python
# Django
from django.contrib.auth.decorators import login_required

@login_required
def admin_panel(request):
    ...
```
```python
# Flask
from functools import wraps
from flask import redirect

def login_required(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        if not current_user.is_authenticated:
            return redirect("/login")
        return f(*args, **kwargs)
    return wrapper
```

### 5.6 Streamlit — multipage app routing
Streamlit routes between pages via the filesystem (`pages/` directory) or the newer `st.navigation` API.

```python
# streamlit_app.py
import streamlit as st

home = st.Page("views/home.py", title="Home", default=True)
products = st.Page("views/products.py", title="Products")
admin = st.Page("views/admin.py", title="Admin")

pg = st.navigation([home, products, admin])
pg.run()
```

### 5.7 Programmatic navigation / redirects
```python
# Flask
from flask import redirect, url_for
return redirect(url_for("products.detail", id=42))

# Django
from django.shortcuts import redirect
return redirect("product_detail", product_id=42)

# FastAPI
from fastapi.responses import RedirectResponse
return RedirectResponse(url="/products/42")

# Streamlit
st.switch_page("views/products.py")
```

### 5.8 Recommendation summary
| Framework | Recommended approach |
|---|---|
| Flask (small) | `@app.route()` decorators |
| Flask (larger) | Blueprints for modular routing |
| Django | URLconf (`urls.py`) + `include()` for nesting |
| FastAPI | `APIRouter` per feature/module, mounted via `include_router` |
| Streamlit | `st.Page` + `st.navigation()` (multipage apps) |
| Any framework | Dependency/decorator-based guards for auth |

---

## 6. Routing in Gen AI Applications

Gen AI systems add a new kind of routing: not "which URL maps to which page," but **"which tool, agent, retriever, or model should handle this request."** This is often called **intent routing**, **semantic routing**, or **agent routing**.

### 6.1 The core problem
A single LLM call can't optimally handle every request — a coding question, a refund request, and a search query may need different tools, prompts, or even different models (cheap/fast vs. large/accurate). Routing decides that path before (or during) generation.

### 6.2 Manual/rule-based routing
The simplest approach: pattern-match or classify intent yourself, then dispatch.

```python
def route_request(user_input: str) -> str:
    if "refund" in user_input.lower():
        return "billing_agent"
    if "error" in user_input.lower() or "bug" in user_input.lower():
        return "support_agent"
    return "general_agent"

handler = AGENTS[route_request(user_input)]
response = handler.run(user_input)
```

### 6.3 LLM-based routing (classification call)
Use a small/cheap model call to classify intent, then route to the right handler or larger model.

```python
def classify_intent(user_input: str) -> str:
    result = client.messages.create(
        model="claude-haiku-4-5-20251001",  # fast/cheap for routing
        max_tokens=10,
        messages=[{
            "role": "user",
            "content": f"Classify as billing, support, or general: {user_input}"
        }],
    )
    return result.content[0].text.strip().lower()
```

### 6.4 Semantic routing (embedding-based, no LLM call)
Faster and cheaper than an LLM classification call — routes by embedding similarity to example utterances per route.

```python
from semantic_router import Route
from semantic_router.encoders import OpenAIEncoder
from semantic_router.layer import RouteLayer

billing = Route(name="billing", utterances=[
    "I want a refund", "why was I charged twice", "cancel my subscription",
])
support = Route(name="support", utterances=[
    "the app keeps crashing", "I found a bug", "getting an error message",
])

router_layer = RouteLayer(encoder=OpenAIEncoder(), routes=[billing, support])
decision = router_layer("I was charged twice this month")
print(decision.name)  # "billing"
```

### 6.5 Tool routing (function calling)
Instead of routing to a fixed agent, let the model itself pick which tool to call — the most common "routing" pattern in modern LLM apps.

```python
tools = [
    {"name": "get_weather", "description": "Get current weather for a location", "input_schema": {...}},
    {"name": "search_orders", "description": "Search a user's order history", "input_schema": {...}},
]

response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=1000,
    tools=tools,
    messages=[{"role": "user", "content": user_input}],
)
# response.content may include a tool_use block naming the chosen tool
```

### 6.6 Graph-based conditional routing — LangGraph
For multi-step agents, routing is expressed as **conditional edges** between graph nodes — the graph decides at runtime which node runs next.

```python
def route_after_classification(state: AgentState) -> str:
    if state["intent"] == "billing":
        return "billing_node"
    elif state["intent"] == "support":
        return "support_node"
    return "general_node"

graph.add_conditional_edges(
    "classify_node",
    route_after_classification,
    {"billing_node": "billing_node", "support_node": "support_node", "general_node": "general_node"},
)
```

### 6.7 Model routing (cost/latency-aware)
Route requests to different model tiers based on complexity, cost budget, or latency needs — e.g., simple queries to a fast model, complex reasoning to a larger one.

```python
def choose_model(complexity_score: float) -> str:
    if complexity_score < 0.3:
        return "claude-haiku-4-5-20251001"
    elif complexity_score < 0.7:
        return "claude-sonnet-5"
    return "claude-opus-4-8"
```

### 6.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| Few, clearly distinct intents | Manual/rule-based routing (keywords/regex) |
| Nuanced intent classification | LLM-based classification call (small/cheap model) |
| Low-latency, high-volume routing | Semantic/embedding-based routing |
| Model picks among many capabilities | Native tool/function calling |
| Multi-step agent workflows | LangGraph conditional edges |
| Cost/latency optimization | Complexity-aware model routing (tiered models) |

---

## 7. Cross-Stack Decision Cheat Sheet

| Concern | Blazor | Angular | React | Python | Gen AI |
|---|---|---|---|---|---|
| Basic routing | `@page` directive | `Routes` array | React Router `createBrowserRouter` | `@app.route` / URLconf / `APIRouter` | Manual/rule-based |
| Parameters | `{id:int}` + `[Parameter]` | `:id` + `ActivatedRoute`/`input()` | `:id` + `useParams()` | `<int:id>` path converters | Extracted via LLM/tool schema |
| Nested/layout routing | Nested `@page` + layouts | `children` + `<router-outlet>` | `children` + `<Outlet>` | Blueprints / `include()` / `APIRouter` | Sub-graphs (LangGraph) |
| Guards | `[Authorize]` | `CanActivateFn` | Wrapper + `<Navigate>` | Dependency/decorator guards | Policy/safety filters before dispatch |
| Lazy loading | `LazyAssemblyLoader` | `loadComponent`/`loadChildren` | `React.lazy()` + `Suspense` | n/a (server-rendered) | On-demand tool/model loading |
| Data pre-fetch | `OnInitializedAsync` | `ResolveFn` | `loader` (data mode) | View function query | Retrieval before generation (RAG) |
| "Intent" routing | n/a | n/a | n/a | n/a | Semantic router / tool calling / LangGraph edges |

---

## 8. Library & Package Reference

Versions verified against npm/NuGet/PyPI in **August 2026**.

### 8.1 Blazor (C# / NuGet)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| Built-in (`Microsoft.AspNetCore.Components.Routing`) | `@page`, `Router`, `NavigationManager` | ships with .NET SDK | — |
| `Microsoft.AspNetCore.Components.Authorization` | `[Authorize]`, `AuthorizeRouteView` | ships with .NET SDK | — |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| Built-in | `NavigationManager.NavigateTo(uri)` | Programmatically navigates to another route. |
| Built-in | `NavigationManager.LocationChanged` | Event fired whenever the current URL changes. |
| Built-in | `[Parameter]` | Binds a route/query segment to a component property. |
| Built-in | `<NavLink>` | Renders a link that auto-applies an "active" CSS class. |
| Authorization | `[Authorize(Roles = "...")]` | Restricts a routable component to authorized users/roles. |
| Authorization | `AuthorizeRouteView` | Wraps the router to enforce auth before rendering a page. |

### 8.2 Angular (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `@angular/router` | Built-in routing (part of Angular framework) | tracks `@angular/core` version | ships with `@angular/core` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `@angular/router` | `provideRouter(routes)` | Registers the route configuration for the app. |
| `@angular/router` | `Router.navigate([path])` | Programmatically navigates to a route. |
| `@angular/router` | `Router.navigateByUrl(url)` | Navigates using a full URL string. |
| `@angular/router` | `ActivatedRoute.paramMap` | Observable of the current route's parameters. |
| `@angular/router` | `CanActivateFn` | Function-based guard run before entering a route. |
| `@angular/router` | `ResolveFn` | Function-based resolver that pre-fetches data for a route. |
| `@angular/router` | `loadComponent` / `loadChildren` | Lazy-loads a standalone component or child route module. |
| `@angular/router` | `RouterLink` directive | Declarative link binding in templates. |

Check latest: `npm view @angular/router version`

### 8.3 React (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `react-router` | Primary routing package (declarative/data/framework modes) | 8.3.0 | `npm i react-router` |
| `react-router-dom` | Legacy re-export package for v6-style upgrades | 7.18.2 (maintenance) | `npm i react-router-dom` |
| `@tanstack/react-router` | Fully type-safe, file-based routing alternative | check npm | `npm i @tanstack/react-router` |

> **Note:** As of React Router v8 (June 2026), `react-router-dom` is deprecated in favor of installing `react-router` directly; v6 and Remix v2 have reached End of Life and no longer receive security updates.

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `react-router` | `createBrowserRouter(routes)` | Builds the router instance from a route config array. |
| `react-router` | `useParams()` | Reads dynamic segments (e.g., `:id`) from the current route. |
| `react-router` | `useNavigate()` | Returns a function to navigate programmatically. |
| `react-router` | `useLoaderData()` | Reads data returned by a route's `loader` function. |
| `react-router` | `<Outlet />` | Renders the matched child route inside a parent layout. |
| `react-router` | `<Navigate to="..." />` | Declaratively redirects during render (e.g., in guards). |
| `react-router` | `<Link to="...">` | Declarative, client-side navigation link. |

Check latest: `npm view react-router version`

### 8.4 Python (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `flask` | Decorator-based routing, Blueprints | check PyPI | `pip install flask` |
| `django` | URLconf-based routing | check PyPI | `pip install django` |
| `fastapi` | Path operations, `APIRouter` | check PyPI | `pip install fastapi` |
| `streamlit` | Multipage app routing (`st.navigation`) | check PyPI | `pip install streamlit` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `flask` | `@app.route(path)` | Registers a view function for a URL path. |
| `flask` | `Blueprint(name, url_prefix=...)` | Groups related routes into a reusable, prefixed module. |
| `flask` | `url_for(endpoint)` | Builds a URL from a view function's name (avoids hardcoding). |
| `django` | `path(route, view)` | Maps a URL pattern to a view function/class. |
| `django` | `include(module)` | Nests another app's URLconf under a prefix. |
| `django` | `reverse(name)` | Builds a URL from a named route (avoids hardcoding). |
| `fastapi` | `APIRouter(prefix=...)` | Groups related path operations into a mountable router. |
| `fastapi` | `app.include_router(router)` | Mounts an `APIRouter` onto the main app. |
| `fastapi` | `Depends(guard_fn)` | Runs a dependency (e.g., an auth check) before a route handler. |
| `streamlit` | `st.navigation(pages)` | Defines the app's page set and renders navigation UI. |
| `streamlit` | `st.switch_page(page)` | Programmatically navigates to another page. |

Check latest: `pip index versions flask` / `pip index versions fastapi`

### 8.5 Gen AI (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `semantic-router` | Embedding-based intent/route classification | check PyPI | `pip install semantic-router` |
| `langgraph` | Conditional-edge routing for agent graphs | 1.2.10 | `pip install langgraph` |
| `anthropic` | Native tool/function calling for model-driven routing | check PyPI | `pip install anthropic` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `semantic-router` | `Route(name, utterances)` | Defines an intent with example phrases for matching. |
| `semantic-router` | `RouteLayer(encoder, routes)` | Builds a router that classifies input by embedding similarity. |
| `semantic-router` | `router_layer(text)` | Returns the best-matching route (or `None`) for input text. |
| `langgraph` | `graph.add_conditional_edges(node, fn, mapping)` | Routes execution to different nodes based on a function's output. |
| `langgraph` | `graph.set_conditional_entry_point(fn)` | Chooses the graph's starting node dynamically. |
| `anthropic` | `client.messages.create(tools=[...])` | Lets the model choose and call the appropriate tool. |
| `anthropic` | response `tool_use` block | Indicates which tool the model selected and with what input. |

Check latest: `pip index versions semantic-router` / `pip index versions langgraph`

---

*This guide covers the mainstream routing patterns and package versions as of August 2026. Library APIs (especially in the fast-moving Gen AI space) change frequently — check current docs/registries before pinning versions in production.*
