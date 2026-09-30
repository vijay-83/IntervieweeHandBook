# Microsoft AI Ecosystem: Tools, Resources and Packages

Reference for an Applied AI Developer. Last checked: 30 Sep 2026.

> **Read this first.** Microsoft renames and merges AI products often. Items marked ✅ were confirmed against Microsoft sources in Sept 2026. Items marked 🔍 come from general knowledge and should be checked on Microsoft Learn before you quote them in an interview. Package names in particular should be confirmed on PyPI, NuGet or npm.

## Contents
1. [Big picture](#1-big-picture)
2. [Platforms and services](#2-platforms-and-services)
3. [Models](#3-models)
4. [Agent and orchestration frameworks](#4-agent-and-orchestration-frameworks)
5. [Azure AI services (task-specific)](#5-azure-ai-services-task-specific)
6. [Search, data and memory](#6-search-data-and-memory)
7. [Safety, security and Responsible AI](#7-safety-security-and-responsible-ai)
8. [Evaluation, observability and LLMOps](#8-evaluation-observability-and-llmops)
9. [Machine learning and training](#9-machine-learning-and-training)
10. [Copilot products and extensibility](#10-copilot-products-and-extensibility)
11. [Developer tools](#11-developer-tools)
12. [Local, edge and Windows AI](#12-local-edge-and-windows-ai)
13. [Packages by language](#13-packages-by-language)
14. [Open-source repos worth knowing](#14-open-source-repos-worth-knowing)
15. [Learning resources](#15-learning-resources)
16. [Interview cheat sheet](#16-interview-cheat-sheet)

---

## 1. Big picture

| Layer | What lives here | Main products |
|---|---|---|
| Models | First-party, OpenAI, partner and open models | Foundry model catalog, MAI, Phi, Azure OpenAI |
| Platform | Build, host, govern and monitor apps and agents | Microsoft Foundry, Foundry Agent Service, Azure ML |
| Frameworks | Code-level orchestration | Microsoft Agent Framework, Semantic Kernel, AutoGen |
| Knowledge | Retrieval and grounding | Azure AI Search, Foundry IQ, Microsoft IQ, Fabric, Cosmos DB |
| Safety | Filters, red-teaming, governance | Azure AI Content Safety, Prompt Shields, PyRIT, Purview, Entra |
| Apps | End-user and low-code | Microsoft 365 Copilot, Copilot Studio, GitHub Copilot, Security Copilot |
| Edge | On-device AI | Foundry Local, Windows ML, ONNX Runtime |

---

## 2. Platforms and services

| Product | Purpose | Status / notes |
|---|---|---|
| **Microsoft Foundry** (formerly Azure AI Foundry; portal at ai.azure.com) | Unified platform: model catalog, agents, evaluation, tracing, safety, deployment | ✅ Current name |
| **Foundry Agent Service** | Managed runtime for agents (tools, memory, threads, private networking) | ✅ GA |
| **Hosted agents** (in Agent Service) | Run your own agent code in managed, sandboxed sessions with state and filesystem | ✅ GA per July/Aug 2026 update |
| **Toolboxes in Foundry** | One managed endpoint for tools, skills, MCP clients and enterprise data; register once, discover at runtime | ✅ GA per July/Aug 2026 update |
| **Foundry Memory** | Managed agent memory: user, session and procedural | ✅ Announced at Build 2026 (preview at the time) |
| **Foundry IQ** | Unified knowledge and retrieval layer for agents | ✅ Announced GA around Build 2026 |
| **Microsoft IQ / Work IQ / Fabric IQ** | Context layers grounding agents in enterprise and world knowledge | ✅ Announced at Build 2026 |
| **Voice Live** | Real-time speech-to-speech for agents | ✅ GA per July/Aug 2026 update |
| **Model Router** | Routes each request to a suitable model to balance cost and quality | ✅ |
| **Foundry Managed Compute** | Managed hosting for models | ✅ Announced 2026 |
| **Azure OpenAI Service** | OpenAI models with Azure security, regions and networking | 🔍 Now surfaced inside Foundry |
| **Azure Machine Learning** | Full ML lifecycle: training, MLOps, endpoints | 🔍 |
| **Azure AI Search** | Vector, keyword and hybrid search plus semantic ranker | 🔍 |
| **Azure API Management** | AI gateway: quotas, load balancing, token limits, logging | 🔍 |
| **Azure Container Apps / Functions / AKS** | Hosting for AI backends | 🔍 |
| **Microsoft Fabric** | Data platform (OneLake, data agents) feeding AI | 🔍 |
| **Foundry Labs** | Early-access research prototypes | ✅ |

---

## 3. Models

### 3.1 Microsoft first-party (MAI)
| Model | Type | Notes |
|---|---|---|
| **MAI-Thinking-1** | Reasoning LLM | ✅ Announced Build 2026; private preview on Foundry at launch |
| **MAI-Code-1 / Flash** | Coding | ✅ In GitHub Copilot and VS Code |
| **MAI-Image-2 / 2.5** (and flash variants) | Image generation | ✅ Foundry, Copilot, Bing, PowerPoint |
| **MAI-Voice-1 / 2** | Speech synthesis | ✅ |
| **MAI-Transcribe-1 / 1.5** | Speech-to-text | ✅ 43 languages for 1.5 |

### 3.2 Phi family (small language models) 🔍
Small, efficient models for edge, on-device and cost-sensitive work (Phi-3, Phi-4 and variants including multimodal and reasoning versions). Available on Foundry, Hugging Face and via Foundry Local.

### 3.3 Third-party and open models on Foundry
- **OpenAI** (GPT family, o-series reasoning, image, audio, embeddings such as `text-embedding-3`) 🔍
- **Anthropic Claude** ✅ (generally available in Foundry; Opus-class models added during 2026)
- **xAI Grok**, **Kimi**, **Cohere**, **Mistral**, **Meta Llama**, **DeepSeek**, **Fireworks AI-hosted models** ✅ (several confirmed in 2026 updates)
- Hugging Face catalog models 🔍

> Model lineups change monthly. Always check the catalog for current model names, regions and deprecation dates.

---

## 4. Agent and orchestration frameworks

| Framework | Languages | Role |
|---|---|---|
| **Microsoft Agent Framework** ✅ | .NET, Python | Successor unifying Semantic Kernel and AutoGen ideas. v1.0 GA in April 2026. Includes agent harness, skills, memory, middleware, workflows, multi-agent patterns (including Magentic-One), MCP support, GitHub Copilot SDK and Claude Agent SDK integrations |
| **Semantic Kernel** 🔍 | C#, Python, Java | Enterprise SDK: plugins, planners, memory connectors, filters. Still widely deployed |
| **AutoGen** 🔍 | Python, .NET | Multi-agent conversation framework from Microsoft Research; concepts folded into Agent Framework |
| **Microsoft.Extensions.AI** 🔍 | .NET | Common abstractions (`IChatClient`, embeddings) so code is provider-neutral |
| **Foundry Agent Service SDK** ✅ | Python, JS/TS, Java, .NET | Create and run managed agents; stable Python and JS/TS 2.5.0 and Java 2.4.0 by end of Aug 2026, .NET 3.0.0 line still preview |
| **Prompt Flow** 🔍 | Python | Visual and code flows for LLM apps; check current status before recommending |
| **Guidance / TypeChat / LMQL-style tools** 🔍 | Python / TS | Constrained generation and typed outputs |
| **Magentic-UI** 🔍 | Python | Human-in-the-loop multi-agent research prototype |

**Protocols:** MCP (Model Context Protocol) ✅ supported across the stack; A2A (agent-to-agent) 🔍 supported in Agent Service and Agent Framework.

---

## 5. Azure AI services (task-specific)

| Service | What it does | Typical use |
|---|---|---|
| **Azure AI Document Intelligence** 🔍 | Layout, tables, invoices, receipts, custom models | Document extraction, RAG ingestion |
| **Azure Content Understanding** ✅ | Multimodal extraction (documents, audio, video, images) with prebuilt analyzers; expanded at Build 2026 | Workflow automation, document pipelines |
| **Azure AI Speech** 🔍 | Speech-to-text, text-to-speech, translation, custom voice | Voice agents, transcription |
| **Azure AI Vision** 🔍 | Image analysis, OCR, video | Visual inspection, accessibility |
| **Azure AI Language** 🔍 | NER, PII detection, summarization, sentiment, CLU | Text analytics, redaction |
| **Azure AI Translator** 🔍 | Text and document translation | Multilingual apps |
| **Azure AI Video Indexer** 🔍 | Video insights | Media search |
| **Azure Bot Service / Microsoft 365 Agents SDK** 🔍 | Channel connectivity (Teams, web, etc.) | Custom engine agents |
| **Azure Health Data Services / Text Analytics for Health** 🔍 | Healthcare NLP and data | Clinical workflows |

---

## 6. Search, data and memory

| Product | Role in AI apps |
|---|---|
| **Azure AI Search** 🔍 | Hybrid retrieval, semantic ranker, integrated vectorization, security filters, agentic retrieval |
| **Foundry IQ** ✅ | Managed knowledge layer over Work IQ, Fabric IQ, Azure SQL, file search and other sources behind one retrieval endpoint |
| **Azure Cosmos DB** 🔍 | Chat history, agent state, vector search, global distribution |
| **Azure SQL / SQL Server 2025** 🔍 | Native vector data type and search |
| **PostgreSQL (Azure Database) with pgvector / DiskANN** 🔍 | Vector search in relational stores |
| **Azure Cache for Redis / Azure Managed Redis** 🔍 | Semantic cache, session state |
| **Microsoft Fabric / OneLake** 🔍 | Lakehouse data for AI, data agents |
| **Microsoft Graph** 🔍 | Access to M365 data (mail, files, calendar) for agents |
| **Graph connectors** 🔍 | Bring external content into Microsoft 365 Copilot |
| **Kernel Memory** 🔍 | Open-source RAG/memory service (verify activity status) |
| **GraphRAG** 🔍 | Graph-based RAG from Microsoft Research (open source) |
| **Azure Blob Storage / Data Lake** 🔍 | Raw document storage |

---

## 7. Safety, security and Responsible AI

| Tool | Purpose |
|---|---|
| **Azure AI Content Safety** 🔍 | Harm-category filters for text and images, blocklists |
| **Prompt Shields** 🔍 | Detect direct jailbreaks and indirect prompt injection |
| **Groundedness detection** 🔍 | Flag claims unsupported by source material |
| **Protected material detection** 🔍 | Detect copyrighted text or code in output |
| **Guided Guardrail Setup, Agent Control Specification (ACS), ASSERT** ✅ | Governance and trust tooling announced at Build 2026 |
| **PyRIT** 🔍 | Open-source Python Risk Identification Tool for red-teaming generative AI |
| **AI Red Teaming Agent (Foundry)** 🔍 | Automated adversarial testing |
| **Microsoft Presidio** 🔍 | Open-source PII detection and anonymization |
| **Microsoft Entra ID / Entra Agent ID** ✅ | Identity for users and agents (agents get their own identity) |
| **Managed Identity, Key Vault, RBAC** 🔍 | Secretless auth and least privilege |
| **Microsoft Purview** 🔍 | Data classification, DLP, governance for AI data |
| **Microsoft Defender for Cloud (AI workloads)** 🔍 | Threat protection for AI apps |
| **Microsoft Security Copilot** 🔍 | AI assistant for security operations |
| **Responsible AI Standard and Impact Assessment template** 🔍 | Governance documents |
| **Responsible AI Toolbox / Fairlearn / InterpretML** 🔍 | Fairness and interpretability (open source) |

---

## 8. Evaluation, observability and LLMOps

| Tool | Purpose |
|---|---|
| **Foundry evaluators** 🔍 | Built-in metrics: groundedness, relevance, coherence, safety, agent evaluators |
| **`azure-ai-evaluation` SDK** 🔍 | Run evaluations in code and CI |
| **Rubric, Agent Optimizer, Agent ROI, tracing** ✅ | Build 2026 additions for scoring and improving agents |
| **AI Observability Starter Kit for Foundry Agents** ✅ | Sample observability setup |
| **Azure Monitor / Application Insights** 🔍 | Telemetry, dashboards, alerts |
| **OpenTelemetry (GenAI semantic conventions)** 🔍 | Standard tracing for LLM and agent calls |
| **Azure Load Testing** ✅ | Includes guidance for load testing hosted MCP servers |
| **Azure DevOps / GitHub Actions** 🔍 | CI/CD with eval gates |
| **Azure Cost Management** 🔍 | Budgets and alerts for token spend |

---

## 9. Machine learning and training

| Tool | Purpose |
|---|---|
| **Azure Machine Learning** 🔍 | Training, pipelines, model registry, managed endpoints, MLOps |
| **Foundry fine-tuning** 🔍 | Supervised, preference and reinforcement fine-tuning for supported models |
| **OpenEnv with Foundry** ✅ | Reinforcement learning environments for outcome-driven agents |
| **ONNX Runtime** 🔍 | Cross-platform inference engine (open source) |
| **ONNX Runtime GenAI** 🔍 | Generative model inference on device |
| **Olive** 🔍 | Model optimization toolkit for ONNX |
| **DeepSpeed** 🔍 | Distributed training and inference optimization |
| **Microsoft Research releases** 🔍 | LoRA, LLMLingua (prompt compression), BitNet (1-bit models), Orca, and more |
| **Azure Databricks / Synapse / Fabric** 🔍 | Data and ML at scale |
| **ML.NET** 🔍 | Classic ML for .NET |
| **CNTK** 🔍 | Legacy; not recommended for new work |

---

## 10. Copilot products and extensibility

| Product | Who it is for | Extend with |
|---|---|---|
| **Microsoft 365 Copilot** 🔍 | Knowledge workers in Word, Excel, PowerPoint, Outlook, Teams | Declarative agents, API plugins/actions, Graph connectors, custom engine agents |
| **Copilot Studio** ✅ | Makers and business users (low-code agents) | Connectors, Power Platform, MCP, publishing to M365 and Teams |
| **Foundry agents published to Teams and M365 Copilot** ✅ | Developers | One governed publishing pipeline (GA June 2026) |
| **Foundry autopilot agents** ✅ | Developers (public preview in June 2026) | Agents with own Entra Agent ID, mailbox and Teams presence |
| **GitHub Copilot** ✅ | Developers | Agent mode, coding agent, MCP servers, Copilot SDK, model picker including MAI-Code |
| **Microsoft Copilot (consumer app)** ✅ | Consumers | Windows, web, iOS, Android, macOS |
| **Security Copilot** 🔍 | SOC and IT admins | Plugins and agents |
| **Dynamics 365 Copilot** 🔍 | CRM/ERP users | Copilot Studio extensions |
| **Power Platform (Power Apps, Power Automate, AI Builder)** 🔍 | Low-code developers | AI actions and agents |
| **Copilot in Azure / Fabric / Edge / Windows** 🔍 | Various | Built-in assistants |
| **Microsoft 365 Agents SDK / Agent 365** 🔍 | Developers and IT | Build and govern agents across M365 (verify current naming) |

---

## 11. Developer tools

| Tool | Purpose |
|---|---|
| **Foundry Toolkit for Visual Studio Code** ✅ | GA April 2026; build, test and deploy agents from VS Code (successor to the AI Toolkit extension) |
| **VS Code + GitHub Copilot** ✅ | AI pair programming, agent mode, MCP integration |
| **Visual Studio + GitHub Copilot** 🔍 | Same in Visual Studio |
| **Azure Developer CLI (`azd`) and AI templates** 🔍 | One-command deploy of AI reference apps |
| **Azure CLI, Bicep, Terraform** 🔍 | Infrastructure as code |
| **GitHub Models** 🔍 | Free-tier model playground and API for prototyping |
| **Foundry playground and MAI Playground** ✅ | Try models in the browser |
| **Azure Samples (GitHub)** 🔍 | Reference apps (RAG chat, agents) |
| **Dev Containers / Codespaces** 🔍 | Reproducible dev environments |
| **Agent terminal powered by GitHub Copilot on Windows** ✅ | Announced at Build 2026 |
| **`microsoft/agent-resources` (GitHub)** ✅ | Curated docs and news for Foundry and agents |

---

## 12. Local, edge and Windows AI

| Tool | Purpose |
|---|---|
| **Foundry Local** ✅ | Run models on-device with an OpenAI-compatible API; gained new capabilities in 2026 |
| **Windows ML** 🔍 | System-wide ONNX-based inference on Windows using CPU, GPU and NPU |
| **Windows AI APIs / Windows AI Foundry** 🔍 | Built-in on-device capabilities (text, vision) for apps |
| **Copilot+ PCs (NPU hardware)** 🔍 | Local AI acceleration |
| **Phi models** 🔍 | Small models suited for local use |
| **Azure Arc / Azure Local / IoT Edge** 🔍 | Hybrid and edge deployment |
| **Local models for Windows** ✅ | New on-device models announced at Build 2026 |

---

## 13. Packages by language

> Package names below are from general knowledge. Confirm exact names and versions on PyPI, NuGet and npm, since Microsoft renames SDKs frequently.

### Python (pip)
| Package | Purpose |
|---|---|
| `openai` | Official OpenAI SDK; works with Azure OpenAI (`AzureOpenAI`) |
| `azure-identity` | `DefaultAzureCredential`, managed identity |
| `azure-ai-projects` | Foundry project client (agents, connections, evaluations) |
| `azure-ai-inference` | Unified inference client for catalog models |
| `azure-search-documents` | Azure AI Search client |
| `azure-ai-documentintelligence` | Document Intelligence |
| `azure-ai-contentsafety` | Content Safety |
| `azure-ai-evaluation` | Evaluators and eval runs |
| `azure-ai-textanalytics` / `azure-ai-language-*` | Language services |
| `azure-cognitiveservices-speech` | Speech SDK |
| `azure-cosmos` | Cosmos DB |
| `azure-keyvault-secrets` | Key Vault |
| `azure-monitor-opentelemetry` | Telemetry export |
| `semantic-kernel` | Semantic Kernel |
| `agent-framework` (and provider extras) | Microsoft Agent Framework 🔍 name may vary by extra |
| `autogen-agentchat`, `autogen-ext` | AutoGen |
| `pyrit` | Red-teaming |
| `presidio-analyzer`, `presidio-anonymizer` | PII handling |
| `graphrag` | GraphRAG |
| `markitdown` | Convert Office/PDF files to Markdown for LLM ingestion |
| `llmlingua` | Prompt compression |
| `onnxruntime`, `onnxruntime-genai` | Local inference |
| `azure-ai-ml` | Azure Machine Learning SDK v2 |
| `promptflow` | Prompt Flow |
| `fairlearn`, `interpret` | Fairness and interpretability |
| `deepspeed` | Training/inference optimization |

### .NET (NuGet)
| Package | Purpose |
|---|---|
| `Azure.AI.OpenAI` / `OpenAI` | OpenAI and Azure OpenAI |
| `Azure.Identity` | Credentials |
| `Azure.AI.Projects` / `Azure.AI.Agents.Persistent` | Foundry projects and agents |
| `Azure.AI.Inference` | Model inference |
| `Azure.Search.Documents` | AI Search |
| `Azure.AI.DocumentIntelligence` | Document Intelligence |
| `Azure.AI.ContentSafety` | Content Safety |
| `Microsoft.Extensions.AI` (+ `.OpenAI`, `.Abstractions`) | Provider-neutral AI abstractions |
| `Microsoft.SemanticKernel` (+ connectors) | Semantic Kernel |
| `Microsoft.Agents.AI` | Microsoft Agent Framework 🔍 confirm exact package IDs |
| `Microsoft.Agents.*` | M365 Agents SDK |
| `ModelContextProtocol` | Official C# MCP SDK |
| `Microsoft.ML.OnnxRuntime`, `Microsoft.ML.OnnxRuntimeGenAI` | Local inference |
| `Microsoft.ML` | ML.NET |
| `Microsoft.Extensions.VectorData` | Vector store abstractions |

### JavaScript / TypeScript (npm)
| Package | Purpose |
|---|---|
| `openai` | OpenAI/Azure OpenAI |
| `@azure/identity` | Credentials |
| `@azure/ai-projects` | Foundry projects |
| `@azure-rest/ai-inference` | Inference |
| `@azure/search-documents` | AI Search |
| `@azure/ai-form-recognizer` / Document Intelligence REST client | Documents |
| `@modelcontextprotocol/sdk` | MCP (community/Anthropic-led standard, widely used with Microsoft tools) |
| `@microsoft/teams.ai` / Microsoft 365 Agents SDK for JS | Teams and M365 agents |
| `onnxruntime-node`, `onnxruntime-web` | Local inference |

### Java (Maven)
`com.azure:azure-ai-openai`, `com.azure:azure-ai-projects`, `com.azure:azure-search-documents`, `com.azure:azure-identity`, `com.microsoft.semantic-kernel:semantickernel-*`.

---

## 14. Open-source repos worth knowing

| Repo / project | What it is |
|---|---|
| `microsoft/agent-framework` 🔍 | Agent Framework |
| `microsoft/semantic-kernel` | Semantic Kernel |
| `microsoft/autogen` | AutoGen |
| `microsoft/graphrag` | GraphRAG |
| `microsoft/PyRIT` | Red-teaming toolkit |
| `microsoft/presidio` | PII detection |
| `microsoft/markitdown` | Document to Markdown |
| `microsoft/LLMLingua` | Prompt compression |
| `microsoft/onnxruntime`, `onnxruntime-genai`, `Olive` | Inference and optimization |
| `microsoft/DeepSpeed` | Training/inference at scale |
| `microsoft/promptflow` | Prompt Flow |
| `microsoft/BitNet` | 1-bit LLM inference |
| `microsoft/generative-ai-for-beginners` | Free course |
| `microsoft/ai-agents-for-beginners` | Free course |
| `microsoft/mcp-for-beginners` | MCP course |
| `microsoft/agent-resources` ✅ | Curated Foundry and agent docs/news |
| `Azure-Samples/*` | Reference architectures (RAG chat, agents, gateways) |
| `microsoft/foundry-local` 🔍 | Local model runtime |
| `microsoft/playwright-mcp` 🔍 | Browser-automation MCP server |

---

## 15. Learning resources

| Resource | Link / where |
|---|---|
| Microsoft Learn: AI learning paths | learn.microsoft.com/training (search "AI") |
| Microsoft Foundry docs | learn.microsoft.com (Foundry section) and ai.azure.com |
| Foundry DevBlog (monthly "What's new") ✅ | devblogs.microsoft.com/foundry |
| Microsoft Agent Framework docs | learn.microsoft.com (search "Agent Framework") |
| Microsoft Build 2026 sessions (on demand) | build.microsoft.com |
| Microsoft AI (MAI models, playground) ✅ | microsoft.ai |
| Azure Architecture Center: AI and RAG guides | learn.microsoft.com/azure/architecture |
| Responsible AI resources | microsoft.com/ai/responsible-ai |
| Microsoft Research blog | microsoft.com/research/blog |
| Tech Community (Azure AI, Foundry blogs) | techcommunity.microsoft.com |
| GitHub: Azure-Samples, microsoft orgs | github.com |

**Relevant certifications** 🔍 (check current availability):
- AI-900: Azure AI Fundamentals
- AI-102: Azure AI Engineer Associate
- DP-100: Azure Data Scientist Associate
- AB-100/AB-series and other newer agent-focused exams: check the certification catalog

---

## 16. Interview cheat sheet

**Say this in one breath:** "For a production agent on Microsoft, I'd use Microsoft Foundry for models, the Agent Service and Toolboxes for runtime and tools, Microsoft Agent Framework in code, Azure AI Search or Foundry IQ for grounding with security trimming, Managed Identity and Entra for auth, Content Safety and Prompt Shields for guardrails, Foundry evaluators and OpenTelemetry tracing for quality, and APIM as the AI gateway."

| If asked about... | Name-drop |
|---|---|
| Building agents in code | Microsoft Agent Framework, Semantic Kernel, Foundry Agent Service |
| Low-code agents | Copilot Studio |
| Retrieval | Azure AI Search (hybrid + semantic ranker), Foundry IQ |
| Document parsing | Document Intelligence, Content Understanding |
| Safety | Content Safety, Prompt Shields, PyRIT, groundedness detection |
| Auth | Managed Identity, Entra ID, Entra Agent ID, on-behalf-of |
| Scaling and cost | APIM AI gateway, PTU vs pay-as-you-go, Model Router, caching |
| Evaluation | Foundry evaluators, `azure-ai-evaluation`, CI gates |
| Tool integration | MCP, Toolboxes, API plugins |
| Local/edge | Foundry Local, Windows ML, ONNX Runtime, Phi |
| Own models | MAI family, Phi |
| Extend M365 Copilot | Declarative agents, API plugins, Graph connectors, custom engine agents |
