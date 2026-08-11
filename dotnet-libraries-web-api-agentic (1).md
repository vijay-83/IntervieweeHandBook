# Powerful .NET Libraries — Web, API & Agentic App Development

A curated reference of high-value libraries/frameworks in the .NET ecosystem, grouped by purpose, with what each is used for and where to find official guidance.

---

## 1. Web App & API Development

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **ASP.NET Core** | Core framework for building web apps, REST/gRPC APIs, and Minimal APIs — cross-platform, high performance | https://learn.microsoft.com/en-us/aspnet/core/ |
| **Minimal APIs** | Lightweight, low-ceremony way to build HTTP APIs without controllers | https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis |
| **Entity Framework Core (EF Core)** | ORM for data access — LINQ-to-SQL, migrations, multiple DB providers | https://learn.microsoft.com/en-us/ef/core/ |
| **Dapper** | Lightweight, high-performance micro-ORM for raw SQL when EF Core is overkill | https://github.com/DapperLib/Dapper |
| **AutoMapper** | Object-to-object mapping (DTOs ↔ entities) to reduce boilerplate mapping code | https://docs.automapper.org/en/stable/ |
| **FluentValidation** | Strongly-typed, fluent validation rules for models/DTOs instead of data annotations | https://docs.fluentvalidation.net/en/latest/ |
| **MediatR** | In-process mediator implementing CQRS/request-handler pattern, decouples controllers from business logic | https://github.com/jbogard/MediatR |
| **Swashbuckle / Swagger (OpenAPI)** | Generates interactive OpenAPI/Swagger docs for your APIs | https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger |
| **Serilog** | Structured, sink-based logging (console, file, Seq, Elasticsearch, App Insights) | https://serilog.net/ |
| **Polly** | Resilience & transient-fault-handling — retry, circuit breaker, bulkhead, timeout | https://www.pollydocs.org/ |
| **Refit** | Type-safe REST client generated from interfaces — great for calling other services | https://github.com/reactiveui/refit |
| **IdentityServer / Duende IdentityServer** | OAuth2/OpenID Connect authentication & authorization server | https://docs.duendesoftware.com/identityserver/ |
| **ASP.NET Core Identity** | Built-in membership system for authentication, roles, and user management | https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity |
| **SignalR** | Real-time bi-directional communication (WebSockets) for live updates, chats, notifications | https://learn.microsoft.com/en-us/aspnet/core/signalr/introduction |
| **gRPC for .NET** | High-performance, contract-first RPC framework for service-to-service communication | https://learn.microsoft.com/en-us/aspnet/core/grpc/ |
| **MassTransit** | Distributed application framework — message bus abstraction over RabbitMQ, Azure Service Bus, Kafka | https://masstransit.io/ |
| **Hangfire** | Background job processing — fire-and-forget, delayed, and recurring jobs, with a dashboard | https://www.hangfire.io/ |
| **Quartz.NET** | Full-featured job scheduling library for cron-style recurring tasks | https://www.quartz-scheduler.net/ |
| **Ocelot** | Lightweight, code-first .NET API Gateway for microservices | https://ocelot.readthedocs.io/ |
| **YARP (Yet Another Reverse Proxy)** | Microsoft's high-performance reverse proxy toolkit, used to build gateways | https://microsoft.github.io/reverse-proxy/ |
| **xUnit / NUnit** | Unit testing frameworks for .NET | https://xunit.net/ · https://nunit.org/ |
| **FluentAssertions** | Readable, fluent assertion library to make test assertions more expressive | https://fluentassertions.com/ |
| **Bogus** | Fake/test data generator (names, addresses, orders, etc.) for seeding and tests | https://github.com/bchavez/Bogus |
| **WireMock.Net / Testcontainers for .NET** | HTTP mocking and spinning up real dependencies (DB, Kafka, Redis) in Docker for integration tests | https://github.com/WireMock-Net/WireMock.Net · https://dotnet.testcontainers.org/ |
| **OpenTelemetry .NET** | Vendor-neutral distributed tracing, metrics, and logging instrumentation | https://opentelemetry.io/docs/languages/net/ |
| **NSwag** | OpenAPI/Swagger toolchain — generates clients and API docs from .NET code | https://github.com/RicoSuter/NSwag |

---

## 2. Agentic App Development (.NET)

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Microsoft Agent Framework** (`Microsoft.Agents.AI`) | Microsoft's production SDK for building AI agents & multi-agent workflows — the unified successor to Semantic Kernel + AutoGen (GA as of April 2026); supports graph-based workflows, checkpointing, human-in-the-loop, and A2A/MCP protocols | https://learn.microsoft.com/en-us/agent-framework/ · https://github.com/microsoft/agent-framework |
| **Semantic Kernel** (`Microsoft.SemanticKernel`) | Lightweight SDK to integrate LLMs (OpenAI, Azure OpenAI, Anthropic, Gemini, etc.) into .NET apps — plugins, planners, memory/vector store abstractions; still actively maintained for AI integration without full multi-agent orchestration | https://learn.microsoft.com/en-us/semantic-kernel/ · https://github.com/microsoft/semantic-kernel |
| **Model Context Protocol (MCP) C# SDK** | Official SDK to build MCP servers/clients so .NET apps and agents can expose or consume tools in a standardized way | https://github.com/modelcontextprotocol/csharp-sdk |
| **Microsoft.Extensions.AI** | Unified, provider-agnostic abstraction layer (`IChatClient`, `IEmbeddingGenerator`) for wiring any LLM/embedding provider into .NET apps via DI | https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai |
| **Azure AI Foundry SDK** | SDK/service for deploying, hosting, and managing production agents on Azure, with built-in observability and A2A interoperability | https://learn.microsoft.com/en-us/azure/ai-foundry/ |
| **LangChain for .NET (LangChain.NET)** | Community port of LangChain concepts (chains, agents, tools, memory) for .NET developers | https://github.com/tryAGI/LangChain |
| **Kernel Memory** | Microsoft's library for building RAG pipelines — ingestion, chunking, embeddings, and vector search on top of Semantic Kernel | https://github.com/microsoft/kernel-memory |
| **Vector store connectors (Qdrant, Pinecone, Azure AI Search, Redis, Postgres/pgvector)** | Store & query embeddings for RAG-powered agents, all exposed via a common `VectorStore` abstraction in Semantic Kernel/Extensions.AI | https://learn.microsoft.com/en-us/semantic-kernel/concepts/vector-store-connectors/ |
| **Betalgo.OpenAI** | Popular community .NET client for the OpenAI API (chat, embeddings, assistants, tools) | https://github.com/betalgo/openai |
| **Anthropic SDK for .NET** | Official/community client for calling Claude models (chat, tool use, streaming) from .NET | https://github.com/anthropics/anthropic-sdk-dotnet |

---

## 3. Database & Data Access

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Entity Framework Core (EF Core)** | Primary ORM — LINQ queries, migrations, change tracking; providers for SQL Server, PostgreSQL, MySQL, SQLite, Cosmos DB | https://learn.microsoft.com/en-us/ef/core/ |
| **Dapper** | High-performance micro-ORM for raw SQL, used where EF Core's overhead isn't wanted (hot paths, reporting queries) | https://github.com/DapperLib/Dapper |
| **Npgsql** | .NET data provider for PostgreSQL (ADO.NET + EF Core provider) | https://www.npgsql.org/ |
| **MongoDB .NET Driver** | Official driver + LINQ provider for MongoDB document databases | https://www.mongodb.com/docs/drivers/csharp/current/ |
| **StackExchange.Redis** | High-performance Redis client — caching, distributed locks, pub/sub, rate limiting | https://stackexchange.github.io/StackExchange.Redis/ |
| **Cosmos DB .NET SDK** | Official SDK for Azure Cosmos DB (SQL API, change feed, partitioning) | https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/quickstart-dotnet |
| **RepoDB** | Lightweight hybrid ORM (raw SQL + repository conveniences), alternative to Dapper/EF | https://repodb.net/ |
| **FluentMigrator / EF Core Migrations** | Version-controlled, code-first database schema migrations | https://fluentmigrator.github.io/ |
| **DbUp** | Deployment-friendly SQL script–based database migrations, popular for CI/CD pipelines | https://dbup.readthedocs.io/ |
| **Elasticsearch.Net / NEST / Elastic.Clients.Elasticsearch** | .NET clients for Elasticsearch — full-text search & log analytics | https://www.elastic.co/guide/en/elasticsearch/client/net-api/current/index.html |
| **LiteDB** | Embedded NoSQL document database (single-file), good for local/offline scenarios | https://www.litedb.org/ |

---

## 4. Cloud Provider Integration

### Azure

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Azure SDK for .NET** (`Azure.*` packages) | Unified SDKs for Storage, Service Bus, Key Vault, Cosmos DB, Event Hubs, App Configuration, etc. | https://learn.microsoft.com/en-us/dotnet/azure/sdk/azure-sdk-for-dotnet |
| **Azure Identity** | Unified authentication (Managed Identity, DefaultAzureCredential, Entra ID) across all Azure SDKs | https://learn.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme |
| **Azure App Configuration + Key Vault** | Centralized configuration & secrets management for enterprise apps | https://learn.microsoft.com/en-us/azure/azure-app-configuration/ |
| **Azure Functions (.NET isolated worker)** | Serverless compute for event-driven and background workloads | https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide |
| **Azure Service Bus / Event Grid / Event Hubs SDKs** | Enterprise messaging, pub/sub, and event streaming on Azure | https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues |
| **Dapr .NET SDK** | Distributed Application Runtime — cloud-agnostic building blocks for state, pub/sub, service invocation, secrets across any cloud/Kubernetes | https://docs.dapr.io/developing-applications/sdks/dotnet/ |

### AWS

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **AWS SDK for .NET** | Official SDK for S3, DynamoDB, SQS, SNS, Lambda, and the full AWS service catalog | https://docs.aws.amazon.com/sdk-for-net/ |
| **AWS Lambda .NET (Amazon.Lambda.*)** | Tools/annotations to build and deploy .NET Lambda functions | https://docs.aws.amazon.com/lambda/latest/dg/lambda-csharp.html |
| **AWSSDK.Extensions.NETCore.Setup** | DI-friendly configuration of AWS clients in ASP.NET Core apps | https://github.com/aws/aws-sdk-net |

### Google Cloud

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Google Cloud Client Libraries for .NET** | SDKs for BigQuery, Pub/Sub, Cloud Storage, Firestore, Spanner, etc. | https://cloud.google.com/dotnet/docs/reference |

### Multi-cloud / Cloud-native

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **.NET Aspire** | Microsoft's opinionated stack for building observable, cloud-native, distributed .NET apps — local orchestration + service defaults (health checks, telemetry, resilience) | https://learn.microsoft.com/en-us/dotnet/aspire/ |
| **KubeClient / k8s (Kubernetes C# client)** | Interact with Kubernetes clusters programmatically from .NET | https://github.com/kubernetes-client/csharp |
| **Steeltoe** | Cloud-native building blocks (config, service discovery, circuit breakers) portable across Cloud Foundry, Kubernetes, and other platforms | https://docs.steeltoe.io/ |

---

## 5. Threading, Concurrency & Parallelism

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Task Parallel Library (TPL) / async-await** | Core .NET async programming model — `Task`, `Task<T>`, `async`/`await` | https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming |
| **System.Threading.Channels** | Producer/consumer pipelines — high-performance in-process async queues | https://learn.microsoft.com/en-us/dotnet/core/extensions/channels |
| **Parallel LINQ (PLINQ)** | Declarative data-parallel query execution over `IEnumerable<T>` | https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/introduction-to-plinq |
| **System.Threading.Tasks.Dataflow (TPL Dataflow)** | Actor/pipeline-style concurrent data processing blocks (BufferBlock, TransformBlock, ActionBlock) | https://learn.microsoft.com/en-us/dotnet/standard/parallel-programming/dataflow-task-parallel-library |
| **Reactive Extensions (Rx.NET / System.Reactive)** | Composable, LINQ-style event/async stream processing (observables) | https://github.com/dotnet/reactive |
| **SemaphoreSlim / Interlocked / ReaderWriterLockSlim** | Built-in low-level synchronization primitives for shared-state coordination | https://learn.microsoft.com/en-us/dotnet/standard/threading/overview-of-synchronization-primitives |
| **IHostedService / BackgroundService** | ASP.NET Core's built-in pattern for long-running/background threads within the app host | https://learn.microsoft.com/en-us/dotnet/core/extensions/workers |
| **AsyncEx (Nito.AsyncEx)** | Community library filling async/await gaps — `AsyncLock`, `AsyncManualResetEvent`, async-friendly collections | https://github.com/StephenCleary/AsyncEx |
| **Polly (Bulkhead policy)** | Concurrency isolation/throttling as a resilience policy (max parallelization per dependency) | https://www.pollydocs.org/strategies/bulkhead.html |

---

## 6. Enterprise Cross-Cutting Concerns

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Microsoft.Extensions.Configuration** | Layered configuration (appsettings.json, env vars, Key Vault, App Configuration) | https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration |
| **Microsoft.Extensions.DependencyInjection** | Built-in DI container used throughout ASP.NET Core and enterprise apps | https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection |
| **Microsoft.Extensions.Caching (Memory & Distributed / Redis)** | In-memory and distributed caching abstractions | https://learn.microsoft.com/en-us/aspnet/core/performance/caching/overview |
| **Serilog + OpenTelemetry + Application Insights** | Full observability stack — structured logs, distributed traces, metrics | https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel |
| **FluentValidation** | Centralized, testable business/input validation rules | https://docs.fluentvalidation.net/en/latest/ |
| **AutoMapper / Mapster** | Object-to-object mapping; Mapster is a faster, source-generator-based alternative to AutoMapper | https://docs.automapper.org/ · https://github.com/MapsterMapper/Mapster |
| **Duende IdentityServer / Microsoft Entra ID (Azure AD)** | Enterprise authN/authZ — OAuth2, OpenID Connect, SSO | https://docs.duendesoftware.com/identityserver/ · https://learn.microsoft.com/en-us/entra/identity-platform/ |
| **Polly + Microsoft.Extensions.Http.Resilience** | Standardized resilience pipelines (retry, circuit breaker, timeout) wired into `HttpClientFactory` | https://learn.microsoft.com/en-us/dotnet/core/resilience/http-resilience |
| **MassTransit / NServiceBus** | Enterprise service bus abstractions for messaging, sagas, and outbox pattern across RabbitMQ/Azure Service Bus/Kafka | https://masstransit.io/ · https://particular.net/nservicebus |
| **Hangfire / Quartz.NET** | Reliable background job scheduling and processing with persistence | https://www.hangfire.io/ · https://www.quartz-scheduler.net/ |
| **FluentAssertions + xUnit + Testcontainers** | Enterprise-grade automated testing (unit + integration against real containerized dependencies) | https://fluentassertions.com/ · https://xunit.net/ · https://dotnet.testcontainers.org/ |
| **.NET Aspire** | Standardizes service defaults (health checks, telemetry, resilience, service discovery) across all services in an enterprise solution | https://learn.microsoft.com/en-us/dotnet/aspire/ |

---

## 7. React + Vite (Frontend)

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Vite** | Next-gen frontend build tool — fast dev server (native ESM/HMR) and optimized production bundling | https://vite.dev/ |
| **React** | Core UI library for building component-based frontends | https://react.dev/ |
| **TypeScript** | Static typing for React/Vite apps — catches errors at compile time, essential for enterprise scale | https://www.typescriptlang.org/docs/ |
| **React Router** | Client-side routing — nested routes, loaders, actions for data-driven navigation | https://reactrouter.com/ |
| **TanStack Query (React Query)** | Server-state management — caching, background refetching, mutations for API data | https://tanstack.com/query/latest |
| **TanStack Table** | Headless, powerful data-table/grid logic (sorting, filtering, pagination, virtualization) | https://tanstack.com/table/latest |
| **Zustand** | Minimal, hook-based global state management — simpler alternative to Redux | https://zustand.docs.pmnd.rs/ |
| **Redux Toolkit** | Official, opinionated Redux setup for complex, large-scale client-state management | https://redux-toolkit.js.org/ |
| **Axios** | Promise-based HTTP client with interceptors, easier than raw `fetch` for enterprise API calls | https://axios-http.com/docs/intro |
| **React Hook Form** | Performant, minimal-re-render form state management | https://react-hook-form.com/ |
| **Zod** | TypeScript-first schema validation — pairs with React Hook Form for form/API validation | https://zod.dev/ |
| **Tailwind CSS** | Utility-first CSS framework for fast, consistent styling | https://tailwindcss.com/docs/installation/using-vite |
| **shadcn/ui** | Copy-in, accessible, unstyled-by-default component library built on Radix UI + Tailwind | https://ui.shadcn.com/ |
| **Radix UI** | Unstyled, accessible primitive components (dialogs, dropdowns, tooltips) — the base for many design systems | https://www.radix-ui.com/primitives |
| **MUI (Material UI)** | Full-featured, pre-styled enterprise component library implementing Material Design | https://mui.com/material-ui/getting-started/ |
| **Ant Design** | Enterprise-grade component library popular for admin dashboards and data-heavy apps | https://ant.design/docs/react/introduce |
| **React Hook Form + Yup/Zod** | Combined form + schema validation stack (see above) | https://react-hook-form.com/get-started |
| **Recharts / Nivo / Visx** | Charting libraries for dashboards and data visualization | https://recharts.org/ · https://nivo.rocks/ · https://airbnb.io/visx/ |
| **i18next / react-i18next** | Internationalization (i18n) — multi-language enterprise apps | https://react.i18next.com/ |
| **MSW (Mock Service Worker)** | API mocking at the network level for development and testing | https://mswjs.io/ |
| **Vitest** | Vite-native unit testing framework — fast, Jest-compatible API | https://vitest.dev/ |
| **React Testing Library** | Component testing focused on user behavior, not implementation details | https://testing-library.com/docs/react-testing-library/intro/ |
| **Playwright** | End-to-end browser testing across Chromium, Firefox, WebKit | https://playwright.dev/ |
| **ESLint + Prettier** | Linting and code formatting for consistent, enterprise-grade code quality | https://eslint.org/docs/latest/ · https://prettier.io/docs/en/ |
| **Storybook** | Isolated component development, documentation, and visual testing | https://storybook.js.org/docs |
| **@microsoft/fetch-event-source / EventSource polyfills** | Streaming responses (SSE) — useful for consuming streaming APIs from agentic/.NET backends | https://github.com/Azure/fetch-event-source |
| **vite-plugin-pwa** | Turns a Vite app into an installable, offline-capable Progressive Web App | https://vite-pwa-org.netlify.app/ |
| **@tanstack/react-router** | Fully type-safe, file-based routing alternative to React Router for large TS codebases | https://tanstack.com/router/latest |

---

## Notes

- **For new agentic projects on the Microsoft stack**, start with **Microsoft Agent Framework** — it reached GA in April 2026 as the unified, production-ready successor to Semantic Kernel and AutoGen, with stable APIs and long-term support.
- **Semantic Kernel** remains a solid, actively maintained choice if you need simpler LLM integration (plugins, RAG, planners) without full multi-agent orchestration, or already have an existing SK codebase.
- **Microsoft.Extensions.AI** is worth adopting even outside agent scenarios — it's the standard DI-friendly abstraction so you're not locked into one model provider's SDK.
- Pair agent frameworks with **MCP** to expose your existing APIs/tools to agents in a standardized, reusable way.
- **Prefer EF Core by default**, and drop to **Dapper** only for performance-critical hot paths or complex reporting queries.
- For **threading**, default to `async`/`await` + TPL for I/O-bound work; reach for `Channels` or `TPL Dataflow` for producer/consumer pipelines, and Rx.NET when you need composable event streams.
- Use **Managed Identity (Azure) / IAM roles (AWS)** instead of hardcoded credentials wherever the cloud SDK supports it — this is standard enterprise security practice.
- **.NET Aspire** is worth adopting early in new enterprise/microservices solutions — it standardizes health checks, telemetry, resilience, and local orchestration across every service from day one.
- For **React + Vite**, a solid enterprise default stack is: TypeScript + React Router + TanStack Query (server state) + Zustand or Redux Toolkit (client state) + Tailwind/shadcn (UI) + Zod + React Hook Form (forms/validation) + Vitest/Playwright (testing).
