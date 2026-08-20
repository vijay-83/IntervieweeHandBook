# State Management Across Web Stacks
### Blazor (C#/.NET) · Angular · React · Python Web Apps · Gen AI Applications

---

## 1. Why State Management Matters

"State" is any data that persists across renders, requests, or user interactions — form inputs, auth tokens, shopping carts, UI toggles, conversation history, etc. State generally falls into these categories:

| Type | Example | Typical Lifetime |
|---|---|---|
| Component/local state | A toggle, an input value | Single component instance |
| Application state | Logged-in user, theme | Whole app session |
| Server/remote state | Data fetched from an API | Cached, re-synced |
| URL state | Filters, pagination | Tied to route |
| Persisted state | Cart, preferences | Browser storage / DB |
| Session/conversation state | Chat history, agent memory | Session or long-term |

Each framework below is organized from **simplest/local** to **advanced/global** patterns.

---

## 2. Blazor (C# / .NET)

Blazor has two hosting models — **Blazor Server** (state lives on the server, per SignalR circuit) and **Blazor WebAssembly/WASM** (state lives in the browser). This changes how you think about state persistence.

### 2.1 Component-local state
Just C# fields/properties inside a `.razor` component — scoped to that component instance.

```csharp
@code {
    private int count = 0;
    private void Increment() => count++;
}
```

### 2.2 Sharing state between components — Cascading Values
Good for small, tree-scoped state (e.g., a theme or current user).

```csharp
// App.razor
<CascadingValue Value="@currentUser">
    <Router AppAssembly="@typeof(App).Assembly" />
</CascadingValue>

@code {
    private UserInfo currentUser = new();
}
```

```csharp
// Any descendant component
[CascadingParameter] public UserInfo CurrentUser { get; set; }
```

### 2.3 App-wide state — Scoped/Singleton Services (DI State Container)
The most common Blazor pattern: a plain C# class registered in DI, injected wherever needed, with an event to notify subscribers of changes.

```csharp
// AppState.cs
public class AppState
{
    public int CartItemCount { get; private set; }
    public event Action? OnChange;

    public void AddToCart()
    {
        CartItemCount++;
        NotifyStateChanged();
    }

    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

```csharp
// Program.cs
// Blazor Server: Scoped = one instance per user circuit (correct choice)
builder.Services.AddScoped<AppState>();

// Blazor WASM: Singleton = one instance per browser tab (correct choice)
builder.Services.AddSingleton<AppState>();
```

```csharp
// Component.razor
@inject AppState State
@implements IDisposable

<p>Items in cart: @State.CartItemCount</p>
<button @onclick="() => State.AddToCart()">Add</button>

@code {
    protected override void OnInitialized() => State.OnChange += StateHasChanged;
    public void Dispose() => State.OnChange -= StateHasChanged;
}
```

> **Server vs WASM caveat:** In Blazor Server, `Scoped` state is tied to the user's live circuit — it vanishes on disconnect/refresh unless persisted. In WASM, `Singleton` state lives in the browser tab and is lost on hard refresh unless you persist it (see 2.5).

### 2.4 Redux-style state — Fluxor
For larger apps, [Fluxor](https://github.com/mrpmorris/Fluxor) brings a Redux/Flux pattern (Store, Actions, Reducers, Effects) to Blazor.

```csharp
// State + Reducer
public record CounterState(int Count);

public static class CounterReducers
{
    [ReducerMethod]
    public static CounterState ReduceIncrement(CounterState state, IncrementAction action)
        => state with { Count = state.Count + 1 };
}

public record IncrementAction;
```

```csharp
// Component
@inject IState<CounterState> CounterState
@inject IDispatcher Dispatcher

<p>@CounterState.Value.Count</p>
<button @onclick="() => Dispatcher.Dispatch(new IncrementAction())">+</button>
```

```csharp
// Program.cs
builder.Services.AddFluxor(o => o.ScanAssemblies(typeof(Program).Assembly));
```

### 2.5 Persisted state — Browser storage & PersistentComponentState
- **Client persistence:** `Blazored.LocalStorage` / `Blazored.SessionStorage` NuGet packages.
- **Server → WASM prerender handoff:** `PersistentComponentState` avoids double data-fetching during prerendering.

```csharp
@inject ILocalStorageService LocalStorage

await LocalStorage.SetItemAsync("theme", "dark");
var theme = await LocalStorage.GetItemAsync<string>("theme");
```

### 2.6 Recommendation summary
| Scale | Recommended approach |
|---|---|
| Single component | Fields/properties |
| Small component tree | Cascading parameters |
| App-wide, simple | DI-injected state container class + event |
| App-wide, complex/testable | Fluxor (Redux pattern) |
| Must survive refresh | Blazored storage / server-side DB session |

---

## 3. Angular

Angular's DI system makes services the natural home for shared state, with RxJS (and now Signals) as the reactivity engine.

### 3.1 Component-local state
Plain class fields, or Angular **Signals** (Angular 16+) for fine-grained reactivity.

```typescript
export class CounterComponent {
  count = signal(0);
  increment() { this.count.update(v => v + 1); }
}
```
```html
<p>{{ count() }}</p>
<button (click)="increment()">+</button>
```

### 3.2 Shared state — Injectable service with RxJS
The classic Angular pattern before NgRx: a service with a `BehaviorSubject` exposed as an `Observable`.

```typescript
@Injectable({ providedIn: 'root' })
export class CartService {
  private itemsSubject = new BehaviorSubject<CartItem[]>([]);
  items$ = this.itemsSubject.asObservable();

  addItem(item: CartItem) {
    const current = this.itemsSubject.value;
    this.itemsSubject.next([...current, item]);
  }
}
```

```typescript
export class CartComponent {
  items$ = this.cartService.items$;
  constructor(private cartService: CartService) {}
}
```
```html
<div *ngFor="let item of items$ | async">{{ item.name }}</div>
```

### 3.3 Redux-style state — NgRx
For large apps needing predictable, testable, time-travel-debuggable state: **Store, Actions, Reducers, Selectors, Effects**.

```typescript
// actions
export const addItem = createAction('[Cart] Add Item', props<{ item: CartItem }>());

// reducer
export const cartReducer = createReducer(
  initialState,
  on(addItem, (state, { item }) => ({ ...state, items: [...state.items, item] }))
);

// selector
export const selectCartItems = createSelector(selectCartState, s => s.items);
```

```typescript
// component
items$ = this.store.select(selectCartItems);
addToCart(item: CartItem) { this.store.dispatch(addItem({ item })); }
```

### 3.4 Server state — Angular + HttpClient with caching, or TanStack Query
For remote data, pair `HttpClient` with `shareReplay()` caching, or use `@ngneat/query` / TanStack Query Angular adapter for automatic caching/refetching.

### 3.5 Recommendation summary
| Scale | Recommended approach |
|---|---|
| Local UI state | Signals or component fields |
| Shared, moderate complexity | Injectable service + RxJS `BehaviorSubject` |
| Large app, strict predictability | NgRx (or Akita/NGXS as lighter alternatives) |
| Remote/server data | HttpClient + caching, or TanStack Query |

---

## 4. React

React state management ranges from built-in hooks to dedicated libraries, and importantly separates **client state** from **server state**.

### 4.1 Component-local state
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### 4.2 Shared state — Context + useReducer
Good for small-to-medium apps without extra dependencies.

```jsx
const CartContext = createContext(null);

function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM': return { ...state, items: [...state.items, action.item] };
    default: return state;
  }
}

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  return (
    <CartContext.Provider value={{ state, dispatch }}>
      {children}
    </CartContext.Provider>
  );
}

// consumer
function CartButton() {
  const { state, dispatch } = useContext(CartContext);
  return <button onClick={() => dispatch({ type: 'ADD_ITEM', item })}>Add</button>;
}
```

> Context re-renders all consumers on any change — fine for low-frequency updates, not ideal for high-frequency state (e.g., typing in a large form).

### 4.3 Lightweight global state — Zustand
Minimal boilerplate, no provider wrapping needed.

```jsx
const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
}));

function CartButton() {
  const addItem = useCartStore((s) => s.addItem);
  return <button onClick={() => addItem(item)}>Add</button>;
}
```

### 4.4 Redux Toolkit — for large, strict, testable apps
```jsx
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addItem: (state, action) => { state.items.push(action.payload); },
  },
});

export const { addItem } = cartSlice.actions;
export const store = configureStore({ reducer: { cart: cartSlice.reducer } });
```

```jsx
const items = useSelector((state) => state.cart.items);
const dispatch = useDispatch();
dispatch(addItem(newItem));
```

### 4.5 Server state — TanStack Query (React Query)
Handles fetching, caching, background refetch, and mutation state — a common mistake is using Redux for API data instead of a dedicated server-state library.

```jsx
function Products() {
  const { data, isLoading } = useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
  });
  if (isLoading) return <Spinner />;
  return <ProductList products={data} />;
}
```

### 4.6 Recommendation summary
| Scale | Recommended approach |
|---|---|
| Local UI state | `useState` / `useReducer` |
| Small shared state | Context + `useReducer` |
| Medium global app state | Zustand / Jotai |
| Large, strict, team-scale | Redux Toolkit |
| Any remote/API data | TanStack Query (regardless of the above choice) |

---

## 5. Python Web Apps

Python web frameworks differ significantly (Flask/Django are stateless-per-request; Streamlit is stateful-per-session), so state strategy depends heavily on which one you're using.

### 5.1 Flask — Session state (cookie-based, signed)
```python
from flask import Flask, session

app = Flask(__name__)
app.secret_key = "change-me"

@app.route("/add-to-cart/<item>")
def add_to_cart(item):
    cart = session.get("cart", [])
    cart.append(item)
    session["cart"] = cart
    return {"cart": cart}
```
> Flask sessions are stored client-side in a signed cookie by default — keep them small. For larger data, use `Flask-Session` with a Redis/DB backend.

### 5.2 Flask/FastAPI — Server-side session store (Redis)
```python
from flask import Flask
from flask_session import Session
import redis

app = Flask(__name__)
app.config["SESSION_TYPE"] = "redis"
app.config["SESSION_REDIS"] = redis.from_url("redis://localhost:6379")
Session(app)
```

### 5.3 Django — Built-in session framework
```python
# views.py
def add_to_cart(request, item_id):
    cart = request.session.get("cart", [])
    cart.append(item_id)
    request.session["cart"] = cart
    request.session.modified = True
    return JsonResponse({"cart": cart})
```
```python
# settings.py — backend options
SESSION_ENGINE = "django.contrib.sessions.backends.cache"   # or db, cached_db, signed_cookies
SESSION_CACHE_ALIAS = "default"  # e.g., Redis
```

### 5.4 FastAPI — Explicit, since FastAPI has no built-in sessions
FastAPI is stateless by design; use signed cookies (`itsdangerous`) or a dependency-injected store.

```python
from fastapi import FastAPI, Depends, Request
from itsdangerous import URLSafeSerializer

app = FastAPI()
serializer = URLSafeSerializer("secret-key")

def get_session(request: Request) -> dict:
    cookie = request.cookies.get("session")
    return serializer.loads(cookie) if cookie else {}

@app.get("/cart")
def get_cart(session: dict = Depends(get_session)):
    return {"cart": session.get("cart", [])}
```

### 5.5 Streamlit — `st.session_state` (per-browser-session, in-memory)
Streamlit reruns the whole script on every interaction, so persistent state *must* go through `session_state`.

```python
import streamlit as st

if "count" not in st.session_state:
    st.session_state.count = 0

if st.button("Increment"):
    st.session_state.count += 1

st.write(f"Count: {st.session_state.count}")
```

### 5.6 Distributed / multi-instance state — Redis or a database
Whenever your app runs behind a load balancer with multiple worker processes/instances, in-memory state (plain dict, `st.session_state` on a single node, etc.) won't be shared across instances. Use:
- **Redis** for fast, ephemeral shared state (sessions, caches, rate limits).
- **PostgreSQL/MySQL** for durable state (user data, orders).
- **Sticky sessions** (load balancer affinity) as a workaround for Streamlit-style apps, though Redis-backed state is more robust.

### 5.7 Recommendation summary
| Framework | Recommended approach |
|---|---|
| Flask (small) | `flask.session` (cookie-based) |
| Flask/Django (scale) | Redis or DB-backed sessions |
| Django | Built-in sessions framework |
| FastAPI | Signed cookies / JWT + Depends, or Redis |
| Streamlit | `st.session_state` |
| Multi-instance deployment | Redis / DB, never in-process memory |

---

## 6. State Management in Gen AI Applications

Gen AI apps add a new kind of state: **conversation/memory state** — what the model "remembers" across turns — plus **agent state** (tool call history, intermediate reasoning, task progress) and **retrieval state** (vector store context).

### 6.1 The core problem
LLM APIs are stateless — every call must include the full context needed to respond. "Memory" is therefore an application-layer concern: you decide what history to resend, summarize, or retrieve.

```python
# Minimal manual pattern: your app owns the message list
conversation = [{"role": "system", "content": "You are a helpful assistant."}]

def chat(user_message: str) -> str:
    conversation.append({"role": "user", "content": user_message})
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1000,
        messages=conversation,
    )
    reply = response.content[0].text
    conversation.append({"role": "assistant", "content": reply})
    return reply
```

### 6.2 Short-term (conversational) memory
- **Buffer memory** — keep the raw last N messages.
- **Summary memory** — periodically summarize older turns to control token growth.
- **Windowed + summary hybrid** — keep the last few turns verbatim, summarize everything older.

```python
# Simple windowed memory
MAX_TURNS = 10

def trim_history(messages):
    system = [m for m in messages if m["role"] == "system"]
    rest = [m for m in messages if m["role"] != "system"]
    return system + rest[-MAX_TURNS:]
```

### 6.3 Long-term memory — Vector stores (RAG-style memory)
For memory that must persist across sessions (user preferences, prior facts), embed and store text in a vector DB, then retrieve relevant chunks per query instead of resending everything.

```python
# Pseudocode using a vector store (e.g., Chroma, Pinecone, pgvector)
def remember(text: str, user_id: str):
    embedding = embed(text)
    vector_store.upsert(id=uuid4(), vector=embedding, metadata={"user_id": user_id, "text": text})

def recall(query: str, user_id: str, k=5):
    embedding = embed(query)
    results = vector_store.query(vector=embedding, filter={"user_id": user_id}, top_k=k)
    return [r.metadata["text"] for r in results]
```

### 6.4 Framework-managed memory — LangChain / LlamaIndex
These frameworks provide memory abstractions so you don't hand-roll trimming/summarization logic.

```python
from langchain.memory import ConversationSummaryBufferMemory
from langchain.chains import ConversationChain

memory = ConversationSummaryBufferMemory(llm=llm, max_token_limit=1000)
chain = ConversationChain(llm=llm, memory=memory)

chain.run("My name is Priya and I work in fintech.")
chain.run("What did I say my job was?")  # memory resolves this
```

### 6.5 Agent state graphs — LangGraph
For multi-step agents (tool use, branching, retries), state is modeled explicitly as a typed object passed between graph nodes.

```python
from langgraph.graph import StateGraph
from typing import TypedDict, List

class AgentState(TypedDict):
    messages: List[dict]
    tool_results: List[dict]
    step_count: int

def call_model(state: AgentState) -> AgentState:
    # ... call LLM, possibly invoke tools
    return {**state, "step_count": state["step_count"] + 1}

graph = StateGraph(AgentState)
graph.add_node("agent", call_model)
graph.set_entry_point("agent")
app = graph.compile()

result = app.invoke({"messages": [], "tool_results": [], "step_count": 0})
```

### 6.6 Session/UI-level state for a chat app
Regardless of backend memory strategy, the frontend needs its own session state to track the active conversation, streaming status, and message list — using whichever pattern from Sections 2–5 fits your stack (e.g., a Zustand store in React, `st.session_state` in Streamlit, a DI-scoped service in Blazor).

### 6.7 Recommendation summary
| Need | Recommended approach |
|---|---|
| Simple chatbot, short sessions | Manual message list, windowed trimming |
| Long conversations | Summary or hybrid memory |
| Cross-session personalization/knowledge | Vector store (RAG-style memory) |
| Rapid prototyping with memory | LangChain memory classes |
| Multi-step agents / tool orchestration | LangGraph (or similar state-graph framework) |
| Production scale, multi-user | Persist conversation + memory in Redis/DB, keyed by session/user ID |

---

## 7. Cross-Stack Decision Cheat Sheet

| Concern | Blazor | Angular | React | Python | Gen AI |
|---|---|---|---|---|---|
| Local component state | fields | signals | `useState` | function scope | n/a |
| Shared app state | DI service + event | RxJS service | Context/Zustand | session object | in-memory conversation list |
| Complex/large-scale state | Fluxor | NgRx | Redux Toolkit | — | LangGraph state |
| Server/remote data | `HttpClient` + DI | HttpClient + TanStack Query | TanStack Query | ORM/DB session | Vector store retrieval |
| Persisted/cross-session | Blazored storage / DB | localStorage / DB | localStorage / DB | Redis / DB | Redis / DB / vector store |

---

## 8. Library & Package Reference

Versions below were verified against npm/NuGet/PyPI in **August 2026**. Package ecosystems move fast (especially Gen AI) — always run the "check latest" command shown before pinning a version in production.

### 8.1 Blazor (C# / NuGet)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| Built-in (`Microsoft.AspNetCore.Components`) | Cascading params, DI state containers | ships with .NET SDK | — |
| `Fluxor.Blazor.Web` | Redux/Flux store for Blazor | 6.11.0 | `dotnet add package Fluxor.Blazor.Web` |
| `Fluxor.Blazor.Web.ReduxDevTools` | Redux DevTools bridge for Fluxor | 6.9.0 | `dotnet add package Fluxor.Blazor.Web.ReduxDevTools` |
| `Blazored.LocalStorage` | Browser localStorage access | check NuGet | `dotnet add package Blazored.LocalStorage` |
| `Blazored.SessionStorage` | Browser sessionStorage access | check NuGet | `dotnet add package Blazored.SessionStorage` |

Check latest: `dotnet package search Fluxor.Blazor.Web` or visit `https://www.nuget.org/packages/Fluxor.Blazor.Web`

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| Built-in | `CascadingValue` / `[CascadingParameter]` | Pass a value down a component tree without prop-drilling. |
| Built-in | `StateHasChanged()` | Manually tell a component to re-render after external state changes. |
| Built-in | `IDisposable.Dispose()` | Unsubscribe from state-change events to avoid memory leaks. |
| Fluxor | `IDispatcher.Dispatch(action)` | Sends an action into the store to trigger a reducer. |
| Fluxor | `IState<T>.Value` | Reads the current slice of state, auto-updates on change. |
| Fluxor | `[ReducerMethod]` | Marks a static method as a pure reducer for a given action type. |
| Fluxor | `IStore.Subscribe` (internal) | Notifies components when any state slice changes. |
| Blazored.LocalStorage | `SetItemAsync(key, value)` | Writes a value to browser localStorage. |
| Blazored.LocalStorage | `GetItemAsync<T>(key)` | Reads and deserializes a value from localStorage. |
| Blazored.LocalStorage | `RemoveItemAsync(key)` | Deletes a stored key. |
| Blazored.LocalStorage | `ClearAsync()` | Wipes all localStorage entries for the app. |

### 8.2 Angular (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `@angular/core` (Signals) | Built-in fine-grained reactivity | ships with Angular | — |
| `@ngrx/store` | Redux-style global store | 21.1.1 (requires Angular 22+) | `npm i @ngrx/store` |
| `@ngrx/effects` | Side-effect handling for NgRx | matches `@ngrx/store` | `npm i @ngrx/effects` |
| `@ngrx/signals` | Signal-based NgRx alternative (SignalStore) | matches `@ngrx/store` | `npm i @ngrx/signals` |
| `@tanstack/angular-query-experimental` | Server-state caching (TanStack Query for Angular) | tracks `@tanstack/query-core` 5.101.x | `npm i @tanstack/angular-query-experimental` |

> Note: as of early 2026, many teams use Angular's native **Signals/`linkedSignal`/`resource()`** for state that previously required NgRx, reserving NgRx for large teams needing strict, enforced patterns.

Check latest: `npm view @ngrx/store version`

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `@angular/core` | `signal(value)` | Creates a reactive, gettable/settable state primitive. |
| `@angular/core` | `computed(fn)` | Derives a read-only reactive value from other signals. |
| `@angular/core` | `effect(fn)` | Runs a side effect automatically when signals it reads change. |
| `@angular/core` | `resource()` | Manages async data (loading/error/value) tied to signals. |
| `@ngrx/store` | `store.select(selector)` | Reads a memoized slice of state as an Observable. |
| `@ngrx/store` | `store.dispatch(action)` | Sends an action to reducers to update state. |
| `@ngrx/store` | `createReducer()` | Builds a reducer function from a set of action handlers. |
| `@ngrx/store` | `createSelector()` | Composes memoized derived-state selectors. |
| `@ngrx/effects` | `createEffect()` | Declares a side effect (e.g., API call) triggered by an action. |
| `@ngrx/signals` | `signalStore()` | Defines a signal-based store (lighter alternative to classic NgRx). |
| RxJS (`rxjs`) | `BehaviorSubject.next(value)` | Pushes a new value to all subscribers, keeps latest value cached. |
| RxJS (`rxjs`) | `Observable.pipe(operators)` | Transforms/composes a stream of state updates. |

### 8.3 React (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `react` (`useState`/`useReducer`/`useContext`) | Built-in local/shared state | ships with React | — |
| `zustand` | Minimal global state store | 5.0.15 | `npm i zustand` |
| `@reduxjs/toolkit` | Redux Toolkit (official Redux) | 2.12.0 | `npm i @reduxjs/toolkit` |
| `react-redux` | React bindings for Redux | 9.3.0 | `npm i react-redux` |
| `@tanstack/react-query` | Server-state fetching/caching | 5.101.4 | `npm i @tanstack/react-query` |
| `@tanstack/react-query-devtools` | DevTools for React Query | 5.101.2 | `npm i @tanstack/react-query-devtools` |
| `jotai` | Atomic state management (Context alternative) | check npm | `npm i jotai` |

Check latest: `npm view zustand version` / `npm view @reduxjs/toolkit version` / `npm view @tanstack/react-query version`

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `react` | `useState(initial)` | Local component state with a setter function. |
| `react` | `useReducer(reducer, initial)` | Local state managed via dispatched actions (mini-Redux). |
| `react` | `useContext(Context)` | Reads shared state provided higher in the component tree. |
| `react` | `useMemo(fn, deps)` | Caches a derived value until dependencies change. |
| `zustand` | `create(setFn)` | Defines a store with state and update functions. |
| `zustand` | `set(partialState)` | Merges a partial update into the store state. |
| `zustand` | `get()` | Reads the current store state outside a React render. |
| `zustand` | `useStore(selector)` | Subscribes a component to a slice of the store. |
| `@reduxjs/toolkit` | `configureStore()` | Sets up the Redux store with sensible defaults (DevTools, middleware). |
| `@reduxjs/toolkit` | `createSlice()` | Generates actions + reducer from a single state slice definition. |
| `@reduxjs/toolkit` | `createAsyncThunk()` | Wraps an async call (e.g., API fetch) as a dispatchable action. |
| `react-redux` | `useSelector(fn)` | Reads a slice of Redux state, re-renders on change. |
| `react-redux` | `useDispatch()` | Returns the `dispatch` function to send actions. |
| `@tanstack/react-query` | `useQuery({queryKey, queryFn})` | Fetches, caches, and auto-refetches server data. |
| `@tanstack/react-query` | `useMutation({mutationFn})` | Handles create/update/delete calls with loading/error state. |
| `@tanstack/react-query` | `queryClient.invalidateQueries()` | Marks cached queries stale so they refetch. |
| `@tanstack/react-query` | `queryClient.setQueryData()` | Manually writes to the query cache (e.g., optimistic updates). |

### 8.4 Python (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `flask` | Web framework, built-in `session` | check PyPI | `pip install flask` |
| `flask-session` | Server-side session backends (Redis, DB, etc.) | check PyPI | `pip install flask-session` |
| `django` | Web framework, built-in sessions | check PyPI | `pip install django` |
| `fastapi` | Async web framework (no built-in sessions) | check PyPI | `pip install fastapi` |
| `itsdangerous` | Signed cookies for stateless frameworks like FastAPI | check PyPI | `pip install itsdangerous` |
| `redis` | Python Redis client, shared/distributed state | check PyPI | `pip install redis` |
| `streamlit` | App framework with built-in `st.session_state` | check PyPI | `pip install streamlit` |

Check latest: `pip index versions flask` (or visit `https://pypi.org/project/<package>/`)

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `flask` | `session["key"] = value` | Stores a value in the signed session cookie. |
| `flask` | `session.get("key")` | Reads a value from the current session. |
| `flask-session` | `Session(app)` | Initializes server-side session storage (Redis/DB/filesystem). |
| `django` | `request.session["key"]` | Reads/writes a value in the Django session store. |
| `django` | `request.session.modified = True` | Forces Django to save a session after mutating a nested value. |
| `django` | `request.session.set_expiry(seconds)` | Controls how long a session stays valid. |
| `fastapi` | `Depends(get_session)` | Injects session/state data into a route via dependency injection. |
| `itsdangerous` | `URLSafeSerializer.dumps(data)` | Signs and serializes data for a tamper-proof cookie. |
| `itsdangerous` | `URLSafeSerializer.loads(token)` | Verifies and deserializes a signed cookie value. |
| `redis` | `client.set(key, value, ex=seconds)` | Stores a value in Redis with an optional expiry (TTL). |
| `redis` | `client.get(key)` | Retrieves a value from Redis. |
| `redis` | `client.delete(key)` | Removes a key from Redis. |
| `streamlit` | `st.session_state["key"]` | Persists a value across Streamlit script reruns in one session. |
| `streamlit` | `st.cache_data` | Caches the return value of a function across reruns. |

### 8.5 Gen AI (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `langchain` | LLM app framework, memory abstractions | 1.3.15 | `pip install langchain` |
| `langgraph` | Stateful multi-actor/agent graphs | 1.2.10 | `pip install langgraph` |
| `langgraph-cli` | CLI for developing/running LangGraph apps | 0.4.31 | `pip install langgraph-cli` |
| `langgraph-sdk` | Python SDK for the LangGraph API | check PyPI | `pip install langgraph-sdk` |
| `langchain-core` | Core abstractions shared across LangChain/LangGraph | tracks `langchain` releases | `pip install langchain-core` |
| `chromadb` | Local/embedded vector store | check PyPI | `pip install chromadb` |
| `pinecone` (client) | Managed vector database client | check PyPI | `pip install pinecone` |
| `anthropic` | Official Claude API SDK (for building the chat loop in §6.1) | check PyPI | `pip install anthropic` |

Check latest: `pip index versions langchain` / `pip index versions langgraph`

> **Gen AI note:** LangChain and LangGraph both ship very frequently (multiple releases per month). Pin exact versions in `requirements.txt` / `pyproject.toml` and re-verify compatibility (especially `langchain-core` version alignment) before upgrading in production.

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `langchain` | `ConversationSummaryBufferMemory` | Keeps recent turns verbatim, summarizes older ones to save tokens. |
| `langchain` | `ConversationBufferWindowMemory` | Keeps only the last N messages of a conversation. |
| `langchain` | `memory.save_context(inputs, outputs)` | Records a turn's input/output into memory. |
| `langchain` | `memory.load_memory_variables()` | Retrieves the current memory content to inject into a prompt. |
| `langchain` | `ConversationChain.run()` | Runs a chat turn with memory automatically applied. |
| `langgraph` | `StateGraph(StateSchema)` | Defines a typed state object shared across graph nodes. |
| `langgraph` | `graph.add_node(name, fn)` | Registers a step (function) in the agent's workflow graph. |
| `langgraph` | `graph.add_edge(from, to)` | Connects nodes to define execution order/flow. |
| `langgraph` | `graph.set_entry_point(name)` | Marks which node runs first. |
| `langgraph` | `graph.compile()` | Builds the graph into a runnable app. |
| `langgraph` | `app.invoke(state)` | Runs the compiled graph once with an initial state. |
| `langgraph` | `app.stream(state)` | Runs the graph and streams intermediate state updates. |
| `chromadb` | `collection.add(documents, embeddings, ids)` | Inserts text + embeddings into the vector store. |
| `chromadb` | `collection.query(query_embeddings, n_results)` | Retrieves the most similar stored chunks to a query. |
| `pinecone` | `index.upsert(vectors)` | Inserts or updates vectors in a Pinecone index. |
| `pinecone` | `index.query(vector, top_k, filter)` | Retrieves the nearest vectors, optionally filtered by metadata. |
| `anthropic` | `client.messages.create()` | Sends a chat request (with full message history as state) to Claude. |

---

*This guide covers the mainstream patterns and package versions as of August 2026. Library APIs (especially in the fast-moving Gen AI space) change frequently — check current docs/registries before pinning versions in production.*
