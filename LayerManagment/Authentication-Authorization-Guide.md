# Authentication & Authorization Management Across Web Stacks
### Blazor (C#/.NET) · Angular · React · Python Web Apps · Gen AI Applications

---

## 1. Authentication vs. Authorization

| Term | Question it answers | Example |
|---|---|---|
| **Authentication (AuthN)** | Who are you? | Logging in with email/password, OAuth, or SSO |
| **Authorization (AuthZ)** | What are you allowed to do? | Only "Admin" role can access `/admin` |

Common building blocks across every stack below:

| Concern | Typical mechanism |
|---|---|
| Identity verification | Password hash check, OAuth/OIDC provider, magic link |
| Session persistence | Cookie session, JWT (access/refresh tokens) |
| Authorization | Roles, claims/scopes, policies |
| Route/endpoint protection | Guards, middleware, decorators |
| Token issuance/validation | Identity provider (IdP) or self-issued JWT |
| Gen AI equivalent | API keys, OAuth for tool/MCP access, per-agent permission scopes |

---

## 2. Blazor (C# / .NET)

ASP.NET Core's identity/auth system underlies Blazor — Blazor Server uses the normal cookie/JWT pipeline over SignalR; Blazor WASM validates tokens client-side and calls protected APIs.

### 2.1 Basic setup — cookie auth (Blazor Server)
```csharp
// Program.cs
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options => options.LoginPath = "/login");
builder.Services.AddAuthorization();

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
```

### 2.2 JWT bearer auth (Blazor WASM calling an API)
```csharp
// API project Program.cs
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-idp.example.com";
        options.Audience = "your-api";
    });
```
```csharp
// Blazor WASM client — attach token to outgoing requests
builder.Services.AddHttpClient("API", client => client.BaseAddress = new Uri("https://api.example.com"))
    .AddHttpMessageHandler<AuthorizationMessageHandler>();
```

### 2.3 Authorizing components and routes
```csharp
@page "/admin"
@attribute [Authorize(Roles = "Admin")]

<h3>Admin Panel</h3>
```
```csharp
<!-- Inline conditional rendering -->
<AuthorizeView Roles="Admin">
    <Authorized>
        <p>Welcome, admin.</p>
    </Authorized>
    <NotAuthorized>
        <p>Access denied.</p>
    </NotAuthorized>
</AuthorizeView>
```

### 2.4 Policy-based authorization
```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("CanEditOrders", policy =>
        policy.RequireClaim("permission", "orders.edit"));
});
```
```csharp
@attribute [Authorize(Policy = "CanEditOrders")]
```

### 2.5 Getting the current user
```csharp
@inject AuthenticationStateProvider AuthStateProvider

@code {
    protected override async Task OnInitializedAsync()
    {
        var authState = await AuthStateProvider.GetAuthenticationStateAsync();
        var user = authState.User;
        var isAuthenticated = user.Identity?.IsAuthenticated ?? false;
    }
}
```

### 2.6 Enterprise identity — Microsoft Entra ID (Azure AD) via MSAL
```csharp
// Program.cs (Blazor WASM)
builder.Services.AddMsalAuthentication(options =>
{
    builder.Configuration.Bind("AzureAd", options.ProviderOptions.Authentication);
});
```

### 2.7 Recommendation summary
| Need | Recommended approach |
|---|---|
| Server-rendered app, simple login | Cookie authentication |
| SPA/WASM calling an API | JWT bearer + `AuthorizationMessageHandler` |
| Role-gated UI/routes | `[Authorize(Roles=...)]` / `<AuthorizeView>` |
| Fine-grained rules | Policy-based authorization (claims/requirements) |
| Enterprise SSO | MSAL + Microsoft Entra ID |

---

## 3. Angular

Angular has no built-in auth system — you implement it with `HttpInterceptor` for tokens, route guards for authorization, and either a hand-rolled service or a library (Auth0, MSAL, `angular-oauth2-oidc`) for OAuth/OIDC.

### 3.1 Auth service (login, token storage, current user)
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private tokenSignal = signal<string | null>(null);

  login(email: string, password: string) {
    return this.http.post<{ token: string }>('/api/login', { email, password })
      .pipe(tap(res => this.tokenSignal.set(res.token)));
  }

  logout() { this.tokenSignal.set(null); }
  isAuthenticated = computed(() => !!this.tokenSignal());

  constructor(private http: HttpClient) {}
}
```

### 3.2 Attaching the token — HTTP interceptor
```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  const token = auth.token();
  const cloned = token
    ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } })
    : req;
  return next(cloned);
};
```
```typescript
// app.config.ts
providers: [provideHttpClient(withInterceptors([authInterceptor]))]
```

### 3.3 Route guards (authorization)
```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isAuthenticated() ? true : router.parseUrl('/login');
};

export const roleGuard = (role: string): CanActivateFn => () => {
  const auth = inject(AuthService);
  return auth.hasRole(role) ? true : inject(Router).parseUrl('/forbidden');
};
```
```typescript
{ path: 'admin', component: AdminComponent, canActivate: [authGuard, roleGuard('Admin')] }
```

### 3.4 OAuth/OIDC — `angular-oauth2-oidc`
```typescript
const authConfig: AuthConfig = {
  issuer: 'https://your-idp.example.com',
  clientId: 'your-client-id',
  redirectUri: window.location.origin,
  responseType: 'code',
  scope: 'openid profile email',
};

this.oauthService.configure(authConfig);
this.oauthService.loadDiscoveryDocumentAndTryLogin();
```

### 3.5 Handling 401s — auto-logout / refresh
```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((err: HttpErrorResponse) => {
      if (err.status === 401) inject(AuthService).logout();
      return throwError(() => err);
    })
  );
};
```

### 3.6 Recommendation summary
| Need | Recommended approach |
|---|---|
| Token storage/state | Signal-based `AuthService` |
| Attaching tokens to requests | `HttpInterceptorFn` |
| Route protection | `CanActivateFn` guards |
| Role/permission checks | Parameterized guards or structural directives |
| Standards-based SSO | `angular-oauth2-oidc` / MSAL Angular / Auth0 Angular SDK |

---

## 4. React

Like Angular, React has no built-in auth — the ecosystem leans on **Auth.js (formerly NextAuth.js)**, **Clerk**, **Auth0**, or a hand-rolled Context + JWT approach.

### 4.1 Auth context (hand-rolled, token-based)
```jsx
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(() => localStorage.getItem("token"));

  const login = async (email, password) => {
    const res = await fetch("/api/login", {
      method: "POST",
      body: JSON.stringify({ email, password }),
    });
    const data = await res.json();
    setToken(data.token);
    setUser(data.user);
  };

  const logout = () => { setToken(null); setUser(null); };

  return (
    <AuthContext.Provider value={{ user, token, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

const useAuth = () => useContext(AuthContext);
```

### 4.2 Attaching tokens to requests
```jsx
async function apiFetch(url, options = {}) {
  const token = localStorage.getItem("token");
  return fetch(url, {
    ...options,
    headers: { ...options.headers, Authorization: `Bearer ${token}` },
  });
}
```

### 4.3 Protected routes (authorization)
```jsx
function ProtectedRoute({ children, requiredRole }) {
  const { user } = useAuth();
  if (!user) return <Navigate to="/login" replace />;
  if (requiredRole && user.role !== requiredRole) return <Navigate to="/forbidden" replace />;
  return children;
}

// route config
{ path: "/admin", element: <ProtectedRoute requiredRole="admin"><Admin /></ProtectedRoute> }
```

### 4.4 OAuth/session auth — Auth.js (NextAuth)
```jsx
// auth.ts
export const { handlers, auth, signIn, signOut } = NextAuth({
  providers: [GitHub, Google],
});
```
```jsx
// Server component
const session = await auth();
if (!session) redirect("/login");
```

### 4.5 Role-based UI rendering
```jsx
function AdminPanelLink() {
  const { user } = useAuth();
  if (user?.role !== "admin") return null;
  return <Link to="/admin">Admin Panel</Link>;
}
```

### 4.6 Recommendation summary
| Need | Recommended approach |
|---|---|
| Simple SPA, own backend | Context + JWT (hand-rolled) |
| Next.js app, OAuth providers | Auth.js (NextAuth) |
| Managed auth, minimal setup | Clerk or Auth0 React SDK |
| Route protection | Wrapper component + `<Navigate>` |
| Role-based UI | Conditional rendering off `user.role`/claims |

---

## 5. Python Web Apps

Auth mechanics vary the most here — Flask/FastAPI are typically hand-assembled or use an extension, Django ships a full auth system out of the box.

### 5.1 Django — built-in authentication
```python
from django.contrib.auth import authenticate, login, logout

def login_view(request):
    user = authenticate(request, username=request.POST["username"], password=request.POST["password"])
    if user is not None:
        login(request, user)
        return redirect("home")
    return render(request, "login.html", {"error": "Invalid credentials"})
```

### 5.2 Django — authorization via decorators & permissions
```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required
def dashboard(request):
    ...

@permission_required("orders.can_edit")
def edit_order(request, order_id):
    ...
```
```python
# settings.py
INSTALLED_APPS += ["django.contrib.auth", "django.contrib.contenttypes"]
```

### 5.3 Django — social/OAuth login via `django-allauth`
```python
INSTALLED_APPS += ["allauth", "allauth.account", "allauth.socialaccount", "allauth.socialaccount.providers.google"]
```

### 5.4 Flask — session-based login with Flask-Login
```python
from flask_login import LoginManager, login_user, login_required, current_user

login_manager = LoginManager()
login_manager.init_app(app)

@app.route("/login", methods=["POST"])
def login():
    user = User.query.filter_by(email=request.form["email"]).first()
    if user and user.check_password(request.form["password"]):
        login_user(user)
        return redirect("/dashboard")
    return "Invalid credentials", 401

@app.route("/dashboard")
@login_required
def dashboard():
    return f"Hello, {current_user.name}"
```

### 5.5 Flask — role-based authorization
```python
def require_role(role):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            if not current_user.is_authenticated or current_user.role != role:
                abort(403)
            return f(*args, **kwargs)
        return wrapper
    return decorator

@app.route("/admin")
@require_role("admin")
def admin_panel():
    ...
```

### 5.6 FastAPI — JWT auth with `python-jose` and `passlib`
```python
from fastapi import Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer
from jose import jwt, JWTError

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def get_current_user(token: str = Depends(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload["sub"]
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid credentials")

@app.get("/me")
def read_me(user=Depends(get_current_user)):
    return {"user": user}
```

### 5.7 FastAPI — batteries-included auth via `fastapi-users`
```python
from fastapi_users import FastAPIUsers

fastapi_users = FastAPIUsers(get_user_manager, [auth_backend])
app.include_router(fastapi_users.get_auth_router(auth_backend), prefix="/auth/jwt")
app.include_router(fastapi_users.get_register_router(UserRead, UserCreate), prefix="/auth")
```

### 5.8 FastAPI/Flask — role-based dependency/guard
```python
def require_role(required_role: str):
    def dependency(user = Depends(get_current_user)):
        if user.role != required_role:
            raise HTTPException(status_code=403, detail="Forbidden")
        return user
    return dependency

@app.get("/admin", dependencies=[Depends(require_role("admin"))])
def admin_panel():
    ...
```

### 5.9 Streamlit — simple session-based auth
```python
if "authenticated" not in st.session_state:
    st.session_state.authenticated = False

if not st.session_state.authenticated:
    password = st.text_input("Password", type="password")
    if st.button("Login") and check_password(password):
        st.session_state.authenticated = True
        st.rerun()
    st.stop()
```

### 5.10 Recommendation summary
| Framework | Recommended approach |
|---|---|
| Django | Built-in `django.contrib.auth` + `django-allauth` for OAuth |
| Flask (session-based) | Flask-Login |
| Flask/FastAPI (stateless API) | JWT via `python-jose`/`PyJWT` + `passlib` for hashing |
| FastAPI (full solution) | `fastapi-users` |
| Streamlit | `st.session_state` gate (or `streamlit-authenticator` package) |
| Any framework, enterprise SSO | OAuth2/OIDC against an external IdP (Auth0, Entra ID, Okta) |

---

## 6. Authentication & Authorization in Gen AI Applications

Gen AI systems introduce auth concerns beyond "who is the human user" — they also need to authenticate **the app to the model provider**, authenticate **agents/tools to third-party services**, and authorize **what an agent is allowed to do on a user's behalf**.

### 6.1 Authenticating to the model provider — API keys
```python
import anthropic

client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])  # never hardcode
```
> Store API keys in environment variables or a secrets manager (AWS Secrets Manager, Azure Key Vault, HashiCorp Vault) — never in source control or client-side code.

### 6.2 Authenticating end users to your Gen AI app
This layer is identical to a normal web app — use the patterns from Sections 2–5 (session cookies, JWT, OAuth) to authenticate the human user before letting them use your chat/agent endpoint.

```python
@app.post("/chat")
def chat(message: str, user = Depends(get_current_user)):  # reuse normal app auth
    response = client.messages.create(
        model="claude-sonnet-5",
        max_tokens=1000,
        messages=[{"role": "user", "content": message}],
    )
    return {"reply": response.content[0].text}
```

### 6.3 Authorizing what a user can ask for — per-user scoping
Always constrain retrieval and tool access to what the authenticated user is permitted to see — the model has no innate concept of "this user's data only."

```python
def search_documents(query: str, user_id: str):
    # Row-level filtering enforced in your retrieval layer, not by the model
    return vector_store.query(query, filter={"owner_id": user_id})
```

### 6.4 Authenticating tools/agents to third-party services — OAuth for tool use
When an agent calls external services on the user's behalf (e.g., "check my calendar", "send this email"), it needs the user's delegated OAuth token — the agent itself never holds the user's raw password.

```python
def get_calendar_events(access_token: str):
    return requests.get(
        "https://api.calendarservice.com/events",
        headers={"Authorization": f"Bearer {access_token}"},
    ).json()

# The access_token is obtained via a standard OAuth 2.0 authorization-code flow,
# scoped to only what the tool needs (e.g., calendar.read).
```

### 6.5 MCP (Model Context Protocol) authorization
When connecting to MCP servers/tools, the MCP spec supports OAuth 2.1-based authorization so a connected tool only acts within the scopes the user granted — the same principle as 6.4, standardized for tool connectors.

```json
{
  "mcpServers": {
    "calendar": {
      "url": "https://mcp.calendarservice.com",
      "auth": { "type": "oauth2", "scopes": ["calendar.read"] }
    }
  }
}
```

### 6.6 Authorizing agent actions — permission scopes & human-in-the-loop
For agents that can take consequential actions (send money, delete data, send messages), gate the action behind an explicit authorization check or a human confirmation step — don't rely on the model to self-limit.

```python
SENSITIVE_ACTIONS = {"delete_account", "transfer_funds", "send_email"}

def execute_tool_call(tool_name: str, args: dict, user):
    if tool_name in SENSITIVE_ACTIONS and not user.has_permission(tool_name):
        raise PermissionError(f"User not authorized for {tool_name}")
    if tool_name in SENSITIVE_ACTIONS:
        require_human_confirmation(tool_name, args)
    return TOOLS[tool_name](**args)
```

### 6.7 Rate limiting & key rotation as an authorization control
Treat API-key scope and rate limits as part of authorization — restrict keys to the minimum model/endpoint access needed, and rotate them regularly.

```python
# Prefer per-environment, least-privilege keys over one shared key
ANTHROPIC_API_KEY_PROD = os.environ["ANTHROPIC_API_KEY_PROD"]
ANTHROPIC_API_KEY_STAGING = os.environ["ANTHROPIC_API_KEY_STAGING"]
```

### 6.8 Recommendation summary
| Need | Recommended approach |
|---|---|
| App → model provider auth | API key in env var / secrets manager |
| End-user → app auth | Standard session/JWT/OAuth (reuse Sections 2–5) |
| Scoping retrieval/data access | Enforce `user_id`/ownership filters at the data layer, not via prompting |
| Agent → third-party service auth | Delegated OAuth 2.0 tokens, least-privilege scopes |
| Tool/connector auth (MCP) | OAuth 2.1-based MCP authorization |
| Consequential agent actions | Explicit permission checks + human-in-the-loop confirmation |
| Key management | Least-privilege, environment-separated, rotated API keys |

---

## 7. Cross-Stack Decision Cheat Sheet

| Concern | Blazor | Angular | React | Python | Gen AI |
|---|---|---|---|---|---|
| Login/session | Cookie auth / `AuthenticationStateProvider` | Auth service + token signal | Context/provider + JWT or Auth.js | Django auth / Flask-Login / FastAPI JWT | Reuse app-layer auth for the human user |
| Attaching tokens | `AuthorizationMessageHandler` | `HttpInterceptorFn` | fetch wrapper / Auth.js session | `Authorization` header / cookie | API key env var; delegated OAuth token per tool |
| Route/endpoint guard | `[Authorize]` / `AuthorizeView` | `CanActivateFn` | Wrapper + `<Navigate>` | Decorators / `Depends()` | Permission check before tool execution |
| Role/policy checks | Policy-based authorization | Parameterized guards | Conditional rendering on `user.role` | `permission_required` / custom dependency | Per-user data scoping + sensitive-action gating |
| Standards-based SSO | MSAL + Entra ID | `angular-oauth2-oidc` / MSAL Angular | Auth.js / Auth0 / Clerk | `django-allauth` / Authlib | OAuth 2.1 (MCP) for tool/connector access |

---

## 8. Library & Package Reference

Versions verified against npm/NuGet/PyPI in **August 2026**.

### 8.1 Blazor (C# / NuGet)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `Microsoft.AspNetCore.Authentication.JwtBearer` | JWT bearer token authentication middleware | 10.0.10 | `dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer` |
| `Microsoft.AspNetCore.Components.Authorization` | `AuthorizeView`, `[Authorize]` for Blazor components | ships with .NET SDK | — |
| `Microsoft.Authentication.WebAssembly.Msal` | MSAL-based login for Blazor WASM (Entra ID) | check NuGet | `dotnet add package Microsoft.Authentication.WebAssembly.Msal` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| Built-in | `AddAuthentication()` | Registers the authentication service and default scheme. |
| Built-in | `AddAuthorization()` | Registers the authorization service and policy engine. |
| Built-in | `AuthenticationStateProvider.GetAuthenticationStateAsync()` | Retrieves the current user's identity/claims. |
| Components.Authorization | `<AuthorizeView>` | Conditionally renders UI based on auth state/role/policy. |
| Components.Authorization | `[Authorize(Roles="...")]` | Restricts a page/component to specific roles. |
| JwtBearer | `AddJwtBearer(options => ...)` | Configures the app to validate incoming JWT bearer tokens. |
| WebAssembly.Msal | `AddMsalAuthentication()` | Wires up MSAL login flow for a Blazor WASM app. |

### 8.2 Angular (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `angular-oauth2-oidc` | OAuth 2.0 / OIDC client for Angular | check npm | `npm i angular-oauth2-oidc` |
| `@azure/msal-angular` | Microsoft Entra ID (Azure AD) authentication | check npm | `npm i @azure/msal-angular` |
| `@auth0/auth0-angular` | Auth0 SDK for Angular | check npm | `npm i @auth0/auth0-angular` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `@angular/common/http` | `HttpInterceptorFn` | Intercepts outgoing requests to attach auth headers. |
| `@angular/router` | `CanActivateFn` | Function-based guard that blocks/allows entering a route. |
| `angular-oauth2-oidc` | `OAuthService.configure()` | Sets issuer, client ID, and scopes for OIDC login. |
| `angular-oauth2-oidc` | `loadDiscoveryDocumentAndTryLogin()` | Loads IdP metadata and completes login if a code/token is present. |
| `angular-oauth2-oidc` | `OAuthService.getAccessToken()` | Retrieves the current access token. |
| `@azure/msal-angular` | `MsalGuard` | Route guard enforcing Entra ID authentication. |
| `@azure/msal-angular` | `MsalInterceptor` | Automatically attaches Entra ID tokens to outgoing requests. |

Check latest: `npm view angular-oauth2-oidc version`

### 8.3 React (npm)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `next-auth` (Auth.js) | Authentication for Next.js (OAuth, credentials, email) | 4.24.15 (stable) / 5.x beta ("Auth.js") | `npm i next-auth` |
| `@auth0/auth0-react` | Auth0 SDK for React SPAs | check npm | `npm i @auth0/auth0-react` |
| `@clerk/clerk-react` | Clerk managed auth for React | check npm | `npm i @clerk/clerk-react` |
| `jsonwebtoken` | Sign/verify JWTs on a Node backend | check npm | `npm i jsonwebtoken` |

> **Note:** Auth.js v5 (formerly NextAuth v5) is now part of Better Auth; new projects are encouraged to evaluate Better Auth directly unless they need Auth.js's stateless, database-less session support.

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| React (built-in) | `useContext(AuthContext)` | Reads the current user/token from a hand-rolled auth context. |
| `next-auth` | `signIn(provider)` | Triggers the sign-in flow for a configured OAuth provider. |
| `next-auth` | `signOut()` | Clears the session and logs the user out. |
| `next-auth` | `auth()` | Server-side helper to read the current session. |
| `@auth0/auth0-react` | `useAuth0()` | Hook exposing `user`, `isAuthenticated`, `loginWithRedirect`, `logout`. |
| `@clerk/clerk-react` | `useUser()` / `useAuth()` | Hooks exposing the current Clerk user and session/token helpers. |
| `react-router` | `<Navigate to="/login" />` | Redirects unauthenticated users away from a protected route. |

Check latest: `npm view next-auth version`

### 8.4 Python (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `django` (built-in `django.contrib.auth`) | Full authentication/authorization framework | check PyPI | ships with `django` |
| `django-allauth` | Social/OAuth login for Django | check PyPI | `pip install django-allauth` |
| `flask-login` | Session-based login management for Flask | check PyPI | `pip install flask-login` |
| `python-jose` | JWT creation/validation (used with FastAPI) | check PyPI | `pip install python-jose` |
| `pyjwt` | Alternative JWT library | check PyPI | `pip install pyjwt` |
| `passlib` | Password hashing (bcrypt, argon2, etc.) | check PyPI | `pip install passlib` |
| `fastapi-users` | Batteries-included auth for FastAPI (JWT, OAuth, registration) | check PyPI | `pip install fastapi-users` |
| `authlib` | OAuth 1.0/2.0 and OIDC client/server toolkit | check PyPI | `pip install authlib` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `django.contrib.auth` | `authenticate(request, username, password)` | Validates credentials and returns a `User` or `None`. |
| `django.contrib.auth` | `login(request, user)` | Starts an authenticated session for the given user. |
| `django.contrib.auth.decorators` | `@login_required` | Blocks a view unless the user is authenticated. |
| `django.contrib.auth.decorators` | `@permission_required(perm)` | Blocks a view unless the user has a specific permission. |
| `flask-login` | `login_user(user)` | Starts a session for the given user object. |
| `flask-login` | `@login_required` | Decorator that blocks a route unless a user is logged in. |
| `flask-login` | `current_user` | Proxy giving access to the currently logged-in user. |
| `python-jose` | `jwt.encode(payload, key, algorithm)` | Creates a signed JWT. |
| `python-jose` | `jwt.decode(token, key, algorithms)` | Verifies and decodes a JWT. |
| `passlib` | `CryptContext.hash(password)` | Hashes a plaintext password securely. |
| `passlib` | `CryptContext.verify(password, hash)` | Checks a plaintext password against a stored hash. |
| `fastapi-users` | `FastAPIUsers.current_user()` | Dependency that resolves the current authenticated user. |

Check latest: `pip index versions django` / `pip index versions fastapi-users`

### 8.5 Gen AI (PyPI)

| Package | Purpose | Latest version | Install |
|---|---|---|---|
| `anthropic` | Official Claude SDK — API key–based authentication | check PyPI | `pip install anthropic` |
| `authlib` | OAuth 2.0/2.1 flows for agent-to-tool authorization | check PyPI | `pip install authlib` |
| `python-dotenv` | Loads API keys/secrets from a `.env` file for local dev | check PyPI | `pip install python-dotenv` |

**Useful methods**

| Package | Method / API | Short description |
|---|---|---|
| `anthropic` | `Anthropic(api_key=...)` | Initializes the client, authenticating via API key. |
| `anthropic` | `client.messages.create(tools=[...])` | Runs a request where returned `tool_use` calls should be permission-checked before execution. |
| `authlib` | `OAuth2Session.authorization_url()` | Builds the URL to start a delegated OAuth authorization flow for a tool. |
| `authlib` | `OAuth2Session.fetch_token()` | Exchanges an authorization code for an access token. |
| `python-dotenv` | `load_dotenv()` | Loads environment variables (e.g., API keys) from a `.env` file. |

Check latest: `pip index versions anthropic`

---

*This guide covers the mainstream authentication and authorization patterns and package versions as of August 2026. Library APIs (especially in the fast-moving Gen AI space) change frequently — check current docs/registries before pinning versions in production. Always follow the principle of least privilege: grant only the scopes/roles/permissions actually needed, whether for a human user, a service account, or an AI agent.*
