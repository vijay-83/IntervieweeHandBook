# Applied AI Developer Interview Q&A (Microsoft)

Covers LLM, RAG, AI Agents, Agentic AI, the Microsoft/Azure stack, evaluation and safety, system design, coding, and behavioral questions. Questions marked **(Added)** were missing from the first question bank.

> **Note:** Microsoft product names change often. Azure AI Foundry has been rebranded (now Microsoft Foundry), and Semantic Kernel and AutoGen are being consolidated into the Microsoft Agent Framework. Verify current names before your interview. The concepts stay the same.

## Contents
1. [LLM Fundamentals](#1-llm-fundamentals)
2. [RAG](#2-rag)
3. [AI Agents](#3-ai-agents)
4. [Agentic AI Systems](#4-agentic-ai-systems)
5. [Microsoft and Azure Stack](#5-microsoft-and-azure-stack)
6. [Evaluation, Safety, Responsible AI](#6-evaluation-safety-and-responsible-ai)
7. [System Design](#7-system-design)
8. [Coding and Hands-on](#8-coding-and-hands-on)
9. [Behavioral](#9-behavioral)
10. [Quick-Fire Round](#10-quick-fire-round-added)

---

## 1. LLM Fundamentals

### Q1. How does a transformer generate text token by token? What does the KV cache do?
**Answer:** The model is autoregressive. It takes the prompt tokens, runs them through stacked layers of self-attention and feed-forward blocks, and outputs a probability distribution over the vocabulary for the *next* token. One token is sampled, appended to the sequence, and the process repeats until a stop token or length limit.

Attention needs keys (K) and values (V) for every previous token. Recomputing them at every step would be quadratic waste. The **KV cache** stores K and V for already-processed tokens, so each new step only computes them for the newest token. This makes decoding much faster, at the cost of memory that grows with sequence length and batch size. It is why long contexts are memory-bound and why "prefill" (processing the prompt in parallel) and "decode" (sequential generation) have different performance profiles.

### Q2. Explain temperature, top-p, and top-k. When would you set temperature to 0?
**Answer:**
- **Temperature** rescales logits before softmax. Low values sharpen the distribution (more deterministic), high values flatten it (more diverse).
- **Top-k** samples only from the k most probable tokens.
- **Top-p (nucleus)** samples from the smallest set of tokens whose cumulative probability is at least p.

Use temperature 0 (or near 0) for extraction, classification, structured output, code generation, and evals, where you want repeatability. Use higher values for brainstorming and creative writing. Note that even at temperature 0, outputs are not perfectly deterministic because of floating-point and batching effects on GPUs. Typically tune temperature *or* top-p, not both.

### Q3. What is a context window? What happens to quality as you fill it (lost-in-the-middle)?
**Answer:** The context window is the maximum number of tokens (input plus output) the model can attend to in one call. Filling it has costs: latency and price rise, and quality can drop. Research shows models often use information at the beginning and end of the context better than the middle ("lost in the middle"). Irrelevant text also distracts the model.

Mitigations: retrieve fewer, better chunks; rerank so the best evidence is at the edges; summarize or compress history; place instructions and key facts prominently; and use long context only when needed.

### Q4. Prompting vs. fine-tuning vs. RAG: how do you choose?
**Answer:**
- **Prompting** (with few-shot examples) first: cheapest and fastest to iterate. Solves most format and behavior problems.
- **RAG** when the model needs *knowledge* that is private, changing, or too large to fit: fresh data, citations, access control.
- **Fine-tuning** when you need consistent *style, format, or behavior* that prompting cannot achieve, want to shrink prompts, or want a smaller cheaper model to match a bigger one on a narrow task. It is poor at injecting facts reliably.

They combine: a fine-tuned model with RAG and a good prompt is common. Start with prompting, add RAG for knowledge, fine-tune last, and always measure with an eval set.

### Q5. What causes hallucinations, and what are three ways to reduce them?
**Answer:** LLMs are trained to produce plausible continuations, not verified facts. Hallucinations come from missing or stale knowledge, ambiguous prompts, training objectives that reward confident guessing, and poor retrieval context.

Reduction:
1. **Ground** with RAG and require citations; instruct the model to say "I don't know" when context lacks the answer.
2. **Constrain** outputs (structured schemas, tool calls for facts and math, lower temperature).
3. **Verify**: groundedness checks, a second-pass verifier, or self-consistency; plus evals and human review for high-stakes flows.

### Q6. What is the difference between LoRA and full fine-tuning?
**Answer:** Full fine-tuning updates all model weights: high quality potential but heavy on GPU memory, storage (a full copy per task), and risk of catastrophic forgetting. **LoRA** freezes the base weights and trains small low-rank adapter matrices injected into layers, typically well under 1% of parameters. It is cheap, fast, easy to swap per task, and often close in quality. QLoRA adds quantization of the base model to cut memory further.

### Q7. How do you get reliable structured output?
**Answer:** Layered approach:
1. **Structured outputs / JSON schema mode** where the API constrains decoding to a schema (strongest guarantee).
2. **Function/tool calling** with a defined parameter schema.
3. Clear prompt with an example and field descriptions.
4. **Validate** with Pydantic/JSON Schema; on failure, retry with the validation error fed back ("repair loop"), with a retry cap.
5. Keep schemas simple, use enums for categories, and use temperature 0.

### Q8. How do you reduce latency and cost?
**Answer:**
- Use smaller/cheaper models for easy steps; route hard ones to bigger models.
- **Prompt caching** (stable prefix first) and **semantic caching** for repeated queries.
- Shorten prompts; retrieve fewer chunks; compress history.
- **Stream** responses for perceived latency.
- Batch offline work (batch APIs are cheaper).
- Limit max output tokens; parallelize independent calls.
- Fine-tune or distill a small model for narrow tasks.
- Measure time-to-first-token, tokens/sec, and cost per request.

### Q9. What is tokenization, and why can it break on numbers, code, or non-English text?
**Answer:** Text is split into subword tokens (BPE and similar) before being fed to the model. Numbers may split into arbitrary chunks, making arithmetic and digit-level tasks unreliable. Code whitespace and rare identifiers can consume many tokens. Non-English languages (especially low-resource ones) tokenize into more tokens per word, raising cost and shrinking effective context. Implications: budget tokens per language, use tools for math, and never assume characters equal tokens.

### Q10. (Added) What are common prompt engineering techniques?
**Answer:** Clear role and task; explicit output format; few-shot examples; delimiters separating instructions from data; step-by-step reasoning where useful (chain-of-thought); decomposing into subtasks (prompt chaining); giving the model an "out" ("if not found, say so"); and iterating against an eval set rather than by feel. For reasoning models, prefer concise goals over detailed step-by-step instructions.

### Q11. (Added) What are reasoning models, and how do you use them differently?
**Answer:** Reasoning models spend extra "thinking" tokens before answering, improving multi-step tasks like math, coding, and planning. Trade-offs: higher latency and cost. Use them for hard, high-value steps (planning, complex analysis) and cheaper non-reasoning models for simple ones. Avoid over-prompting them with manual chain-of-thought; give the goal and constraints.

### Q12. (Added) What are embeddings and how do you choose an embedding model?
**Answer:** Embeddings map text to dense vectors so semantically similar text is close in vector space. Choose by: retrieval quality on *your* domain (test on your data, use benchmarks like MTEB as a guide), dimensionality (storage and speed), max input length, multilingual support, and cost/latency. Changing the embedding model means re-embedding the whole corpus, so version your index.

### Q13. (Added) What is quantization and distillation?
**Answer:** **Quantization** stores weights in lower precision (for example 8-bit or 4-bit) to cut memory and speed inference, with small quality loss. **Distillation** trains a smaller "student" model to mimic a larger "teacher," giving cheaper inference for a narrow task.

### Q14. (Added) What are multimodal models and typical applied use cases?
**Answer:** Models that take images, audio, or documents alongside text. Use cases: document/invoice extraction, screenshot understanding, image Q&A, speech transcription plus summarization. Considerations: image token cost, resolution limits, and evaluating vision accuracy separately from text.

---

## 2. RAG

### Q1. Walk through a RAG pipeline end to end.
**Answer:**
1. **Ingest** documents (connectors, parsing PDFs/HTML, OCR).
2. **Clean and chunk** into passages with metadata (source, section, ACLs, timestamps).
3. **Embed** each chunk.
4. **Index** in a vector store (plus keyword index).
5. **Retrieve** top-k for a query (hybrid search, filters).
6. **Rerank** with a cross-encoder or semantic ranker.
7. **Generate**: prompt the LLM with the query and selected chunks, instruct it to answer only from context and cite sources.
8. **Post-process**: citations, groundedness check, logging.

### Q2. How do you pick a chunking strategy? Trade-offs of size and overlap?
**Answer:** Options: fixed-size (simple), recursive by structure (paragraphs/headings), semantic (split on topic shifts), and hierarchical (small chunks for retrieval, parent chunk for context). Small chunks improve precision but lose context; large chunks keep context but dilute relevance and waste tokens. Overlap (10 to 20%) prevents splitting an idea across a boundary but increases index size and duplicates. Tune empirically with retrieval metrics on a labeled set; respect document structure (tables, code, headings).

### Q3. Compare vector search, keyword (BM25), and hybrid. Why is hybrid plus a semantic reranker often best?
**Answer:** Vector search captures meaning and paraphrases but can miss exact terms (IDs, product codes, rare names). BM25 nails exact lexical matches but misses synonyms. **Hybrid** runs both and fuses ranks (for example Reciprocal Rank Fusion), covering both failure modes. A **reranker** (cross-encoder or Azure semantic ranker) then reads query and candidate together for much better precision on the top results. This "recall wide, rerank narrow" pattern is a standard high-quality baseline.

### Q4. Retrieval returns the wrong chunks. How do you debug?
**Answer:** Work systematically:
1. Inspect the actual retrieved chunks for failing queries.
2. Check whether the answer exists in the index at all (ingestion/parsing failures, dropped tables).
3. Check chunking (answer split across chunks? chunks too big?).
4. Check the embedding model fit and query/document formatting.
5. Try hybrid search, query rewriting, metadata filters, larger k plus reranker.
6. Build a labeled retrieval set and measure recall@k before and after each change.

### Q5. What are query rewriting, HyDE, and multi-query retrieval?
**Answer:**
- **Query rewriting**: an LLM rewrites a vague or conversational query (resolving pronouns using chat history) into a good search query.
- **HyDE**: the LLM writes a hypothetical answer, and that text is embedded for retrieval (answers often look more like documents than questions do).
- **Multi-query**: generate several paraphrases or sub-questions, retrieve for each, and merge results. Improves recall at the cost of latency and tokens.

### Q6. How do you ground answers and add citations, given RAG is "not guaranteed correct"?
**Answer:** Instruct the model to use only provided context and to say when it is insufficient; attach chunk IDs and have the model cite them; validate that cited chunks actually support each claim (groundedness detection or an LLM verifier); show sources in the UI; and abstain or escalate when retrieval confidence is low. Measure groundedness in evals.

### Q7. How do you enforce access control (security trimming)?
**Answer:** Store ACL metadata (user/group IDs) with each chunk at ingestion and apply **filters at query time** based on the caller's identity, so unauthorized chunks are never retrieved or sent to the LLM. Never rely on the prompt to hide data. Sync permission changes from the source (for example SharePoint/Entra groups), use the user's identity (on-behalf-of flow), and audit access. In Azure AI Search this is done with security filters or native document-level access support.

### Q8. How do you keep the index fresh?
**Answer:** Use incremental ingestion: change detection (timestamps, hashes, change feeds/webhooks), upsert changed chunks, delete removed ones, and re-embed only what changed. Schedule indexers, monitor for lag, keep a full-rebuild path (for example when the embedding model changes), and version indexes for safe rollout via aliases.

### Q9. What is GraphRAG and when does it beat vanilla RAG?
**Answer:** GraphRAG extracts entities and relationships into a knowledge graph, builds community summaries, and retrieves over the graph structure. It beats vector RAG on **global or multi-hop questions** ("What are the main themes across these documents?", "How is A connected to C?") where relevant facts are scattered. Costs: expensive indexing and complexity. For simple lookups, standard hybrid RAG is better and cheaper.

### Q10. How do you evaluate retrieval separately from generation?
**Answer:**
- **Retrieval**: recall@k, precision@k, MRR, nDCG against labeled relevant chunks.
- **Generation**: groundedness/faithfulness (supported by context), answer relevance, completeness, and citation accuracy; LLM-as-judge plus human spot checks.
- Evaluating separately lets you tell whether failures come from bad retrieval or bad generation. Tools: Azure AI Foundry evaluators, Ragas, promptfoo.

### Q11. (Added) How does an approximate nearest neighbor index like HNSW work?
**Answer:** HNSW builds a layered proximity graph: sparse upper layers for long jumps, dense lower layers for local refinement. Search greedily walks the graph from the top layer down. It trades a small recall loss for large speedups over exact search. Key parameters: `M` (connections per node), `efConstruction` (build quality), and `efSearch` (query-time recall vs latency). Alternatives: IVF, DiskANN (large-scale, disk-based).

### Q12. (Added) How do you handle tables, PDFs, and images in RAG?
**Answer:** Use layout-aware parsing (Azure AI Document Intelligence) to preserve headings and tables; convert tables to markdown or summarize them with an LLM and index both the summary and the raw table; run OCR for scans; use a multimodal model to describe figures; keep page numbers and section metadata for citations.

### Q13. (Added) How do you handle multi-turn conversations in RAG?
**Answer:** Rewrite the latest user turn into a standalone query using recent history, retrieve on that, and keep a bounded history (summarize older turns). Decide per turn whether retrieval is even needed. Cache and reuse context where appropriate.

### Q14. (Added) When should you cache in RAG, and what are the risks?
**Answer:** Cache embeddings, retrieval results for popular queries, and full answers via semantic caching. Risks: stale answers after data changes (use TTLs and invalidation on updates) and **leaking one user's permitted content to another** (cache keys must include the permission scope).

### Q15. (Added) What are common RAG failure modes?
**Answer:** Missing content (ingestion gaps), wrong chunking, embedding mismatch, low recall, irrelevant chunks diluting context, lost-in-the-middle, model ignoring context, stale data, permission leaks, and prompt injection embedded in retrieved documents.

---

## 3. AI Agents

### Q1. What is the difference between an LLM app, a RAG app, and an agent?
**Answer:**
- **LLM app**: prompt in, response out. Knowledge limited to training data plus the prompt.
- **RAG app**: adds a retrieval step to ground the response in external knowledge. Flow is fixed and single pass.
- **Agent**: given a *goal*, the LLM decides which actions (tools) to take, observes results, updates state, and loops until done. Control flow is decided by the model at runtime.

The key shift: from a predefined pipeline to a model-driven loop with tools and state.

### Q2. Explain the agent loop: goal, decide, act, observe, update task state. What are its failure modes?
**Answer:** The agent core (LLM + instructions + task state) receives a goal, **decides** the next step, **acts** by calling a tool (API, file, app, database), **observes** the result, updates task state, and repeats until the goal is met.

Failure modes: infinite or wasteful loops, wrong tool or bad arguments, hallucinated tool results, goal drift over long runs, context bloat, compounding errors, prompt injection via tool output, and runaway cost. Mitigations: max steps and budgets, validated tool schemas, progress checks, summarizing state, guardrails, and tracing.

### Q3. How does function/tool calling work? How do you design good tool schemas?
**Answer:** You send the model a list of tool definitions (name, description, JSON schema for parameters). The model returns a structured call (tool name plus arguments) instead of text. **Your code** validates and executes it, then returns the result in the next message; the model then continues or answers.

Good design: clear verb-based names; precise descriptions (when to use and not use); minimal, typed parameters with enums; helpful, concise error messages; few tools rather than many overlapping ones; return only needed fields; validate all inputs server-side (never trust model arguments).

### Q4. What is ReAct? How does it differ from plan-and-execute?
**Answer:** **ReAct** interleaves reasoning and acting: think, act, observe, think again. Flexible and adaptive, but can wander and costs many calls. **Plan-and-execute** first creates a full plan, then executes steps (possibly with a cheaper model), replanning only when needed. It is more predictable and efficient for well-structured tasks but less adaptive to surprises. Hybrids are common.

### Q5. How do you manage short-term and long-term memory and task state?
**Answer:**
- **Short-term**: the current context and task state (goal, plan, completed steps, intermediate results). Keep it compact by summarizing or trimming.
- **Long-term**: external store (vector DB, key-value, database) for user preferences, past interactions, and learned facts; retrieved when relevant.
- Store *structured* task state outside the prompt (for example a JSON object or database record) so runs are resumable and auditable. Add write policies (what deserves saving) and privacy controls (retention, deletion, per-user isolation).

### Q6. How do you set stopping conditions, max steps, and budget limits?
**Answer:** Define explicit success criteria and a "done" tool or final-answer format; cap iterations, tokens, wall-clock time, and cost per run; detect repeated identical actions (loop detection); require progress between steps; and on hitting limits, return a partial result with a clear explanation or escalate to a human.

### Q7. How do you handle tool failures, timeouts, and retries safely?
**Answer:** Set timeouts; retry transient errors with exponential backoff and jitter; return structured error messages to the model so it can adapt; make write operations **idempotent** (idempotency keys) so retries don't duplicate side effects; use circuit breakers; separate read tools from write tools; and log every call for audit.

### Q8. What is MCP (Model Context Protocol), and why does it matter?
**Answer:** MCP is an open standard for connecting AI applications to tools, data sources, and prompts through a common client-server protocol. Instead of writing custom integrations per model and per tool, you expose a capability once as an MCP server and any MCP-compatible client can use it. It matters for interoperability, reuse, and ecosystem growth (supported across Microsoft's agent tooling). Security caveats: tool poisoning, over-permissioned servers, and untrusted servers, so vet servers, scope permissions, and require approvals for risky actions.

### Q9. How do you keep a human in the loop for high-risk actions?
**Answer:** Classify actions by risk. Auto-approve read-only or low-risk actions; require explicit confirmation for irreversible, costly, or external-facing ones (payments, deletions, emails). Show the proposed action with parameters and reasoning, let the user approve, edit, or reject, and persist state so the run can pause and resume. Log approvals for audit.

### Q10. (Added) How do you evaluate an agent?
**Answer:** Evaluate the **outcome** (task success rate), the **trajectory** (were tool choices and arguments correct, any unnecessary steps), and **operational metrics** (steps, latency, cost, failure rate). Build scenario-based test suites with mocked tools for determinism, run multiple trials (agents are non-deterministic), use LLM judges plus programmatic checks, and include adversarial cases (injection, tool errors).

### Q11. (Added) How do you decide between giving an agent many tools vs. few?
**Answer:** Too many tools confuse selection and bloat the prompt. Group tools by domain, expose only relevant subsets per task (dynamic tool selection or retrieval over tool descriptions), or delegate to specialized sub-agents with focused toolsets.

### Q12. (Added) What is "excessive agency" and how do you limit it?
**Answer:** An agent having more permissions, tools, or autonomy than the task requires. Limit with least privilege, read-only defaults, scoped credentials, allow-lists, spending caps, sandboxed execution, and approval gates for side effects.

---

## 4. Agentic AI Systems

### Q1. When should you use multiple agents instead of one? What does each extra agent cost?
**Answer:** Use multiple agents when tasks have clearly separable specialties, need parallelism, require different tools/permissions/models, or when a single agent's context and tool list becomes unmanageable. Costs: more tokens and latency, coordination complexity, harder debugging, error propagation, and non-determinism. Start with a single agent (or a workflow) and split only when there is evidence it helps.

### Q2. Compare orchestration patterns.
**Answer:**
- **Sequential**: agents in a pipeline, each passing output to the next (predictable).
- **Concurrent**: agents work in parallel on the same or different inputs, results aggregated (fast, good for diverse perspectives).
- **Group chat**: agents converse in a shared thread, often with a moderator (good for brainstorming/review, can be chatty).
- **Handoff**: one agent transfers control to a more suitable agent (routing/triage).
- **Manager/worker (magentic/supervisor)**: a planner decomposes the goal, assigns subtasks, tracks progress in a task ledger, and replans.

### Q3. How is shared task state managed between agents? How do you avoid conflicting writes?
**Answer:** Keep a central, structured task state (plan, assignments, results, status) in a durable store, with a defined schema. Avoid conflicts via a single writer (the orchestrator) or ownership per section, optimistic concurrency (version numbers/ETags), append-only event logs, and transactions where needed. Agents read state, do work, and report results; the orchestrator merges.

### Q4. Explain "evaluate progress, then replan." How does the system decide the goal is met?
**Answer:** After each round, an evaluator (LLM judge, rules, or tests) compares results and task state against explicit success criteria. If met, finalize; if not, the orchestrator replans (new steps, different agent, more info, or human escalation). Good criteria are measurable (tests pass, all fields filled, sources cited). Add step and budget limits so replanning cannot run forever.

### Q5. What is the difference between a deterministic workflow and an autonomous agent? When is a workflow better?
**Answer:** A **workflow** has code-defined steps (LLM calls inside fixed control flow): predictable, testable, cheaper, easier to secure. An **agent** decides steps itself: flexible for open-ended problems but less predictable and costlier. Prefer a workflow when steps are known, reliability and auditability matter, and latency/cost are tight. Use agents only where the path genuinely cannot be predetermined. Many production systems mix the two.

### Q6. How do you trace and debug a multi-agent run?
**Answer:** Instrument with distributed tracing (OpenTelemetry): one trace per run, spans per agent step, model call, and tool call, recording prompts, responses, tool arguments/results, token counts, latency, and errors. Use correlation IDs, keep state snapshots per step, support replay of a failed run, and visualize traces (Azure Monitor/Application Insights, Foundry tracing). Redact sensitive data in logs.

### Q7. What are the main security risks and mitigations?
**Answer:**
- **Direct prompt injection** (user tries to override instructions): system prompt hardening, input filtering, prompt shields.
- **Indirect prompt injection** (malicious instructions hidden in documents, web pages, emails, tool output): treat all external content as untrusted data, isolate it, use prompt shields/spotlighting, limit what actions can follow untrusted content.
- **Tool poisoning** (malicious tool descriptions or outputs): vet tools/MCP servers, pin versions, review descriptions.
- **Excessive agency**: least privilege, approvals, caps.
- **Data exfiltration** (leaking data via links, images, or tool calls): egress allow-lists, output filtering, no auto-rendering of external URLs, DLP.
Defense in depth: assume the model can be tricked and make sure a tricked model cannot do serious damage.

### Q8. How do agents authenticate and get least-privilege access?
**Answer:** Give each agent its own identity (Entra Agent ID / managed identity or service principal), not a shared admin credential. Use RBAC with narrowly scoped roles, short-lived tokens, secrets in Key Vault, and **on-behalf-of** flows so the agent acts with the *user's* permissions when accessing user data. Audit all access and rotate credentials.

### Q9. (Added) What is A2A (Agent2Agent) and how does it relate to MCP?
**Answer:** MCP standardizes how an agent connects to **tools and data**; A2A standardizes how **agents communicate with other agents** (capability discovery, task delegation across vendors/frameworks). They are complementary layers.

### Q10. (Added) How do you handle long-running agent tasks?
**Answer:** Use durable execution (queues, Durable Functions, workflow engines) with checkpointed state so runs survive crashes and can resume; run asynchronously with status polling or webhooks; support human approval pauses; set timeouts and cost limits; and provide progress updates to the user.

### Q11. (Added) How do you reduce error compounding in multi-step agent chains?
**Answer:** If each step is 95% reliable, ten steps give roughly 60%. Reduce steps, validate intermediate outputs, use deterministic code for deterministic parts, add verification/critic steps, checkpoints, and retries, and choose stronger models for critical steps.

---

## 5. Microsoft and Azure Stack

### Q1. What is Azure OpenAI Service, and how does it differ from calling OpenAI directly?
**Answer:** It hosts OpenAI models within Azure with enterprise features: Azure data-privacy commitments (prompts and completions are not used to train models), regional deployment and data residency options, private networking (Private Link/VNet), Entra ID auth and RBAC, managed identity, built-in content filtering, Azure Monitor integration, and SLAs and compliance certifications. Also offers provisioned throughput for predictable latency. Available through the Foundry model catalog alongside other model providers.

### Q2. How would you build RAG with Azure AI Search?
**Answer:**
1. Store source data (Blob, SharePoint, SQL, Cosmos DB).
2. Create an **index** with text fields, a **vector field** (with HNSW config), and filterable metadata (including ACLs).
3. Use **indexers plus skillsets** to crack documents, chunk (Text Split), and embed (integrated vectorization), or push data via SDK.
4. Query with **hybrid search** (keyword plus vector) and enable the **semantic ranker** for reranking.
5. Pass top results to Azure OpenAI with a grounding prompt and citations.
6. Add security filters, monitoring, and evaluation. Agentic retrieval features can decompose complex queries into subqueries.

### Q3. What are Semantic Kernel, AutoGen, and the Microsoft Agent Framework? When to use each?
**Answer:**
- **Semantic Kernel**: enterprise-oriented SDK (C#, Python, Java) with plugins, planners, memory, and model connectors; strong for embedding AI into existing apps.
- **AutoGen**: research-born framework for multi-agent conversation and orchestration.
- **Microsoft Agent Framework**: the successor that unifies both, combining SK's enterprise foundation with AutoGen's multi-agent patterns, with workflows, state, telemetry, and MCP/A2A support.
For new projects, look at the Agent Framework; for existing SK or AutoGen code, follow migration guidance. Verify current status in the docs.

### Q4. Compare Copilot Studio, Azure AI Foundry, and custom code. Who is each for?
**Answer:**
- **Copilot Studio**: low-code, for makers and business users; quick agents with connectors, Power Platform integration, and M365 channels.
- **Foundry**: pro-code platform for developers: model catalog, agent service, evaluation, tracing, safety, and deployment.
- **Custom code** (SDKs/frameworks): maximum control for complex logic, custom orchestration, and unusual integrations, at the cost of more engineering.
Choose by complexity, control, team skills, and time-to-value. They can be combined.

### Q5. How do you authenticate to Azure services without secrets?
**Answer:** Use **Managed Identity** (system- or user-assigned) on the compute resource, with RBAC role assignments (for example *Cognitive Services OpenAI User*, *Search Index Data Reader*). In code, use `DefaultAzureCredential`, which works with managed identity in Azure and developer login locally. Store any unavoidable secrets in **Key Vault**. Disable key-based auth where possible and use Entra ID for users.

### Q6. How do you deploy and scale an AI backend?
**Answer:** Options: **Azure Functions** (event-driven, bursty), **Container Apps** (simple containers with autoscale, common choice), **App Service**, **AKS** (full control, complex needs). Front with **API Management** as an AI gateway (auth, rate limits, token quotas, load balancing across deployments, logging). Use queues for async work, Redis for caching, autoscaling rules, multi-region for resilience, IaC (Bicep/Terraform) and CI/CD.

### Q7. How do you handle rate limits and quota?
**Answer:** Limits are expressed as tokens per minute (TPM) and requests per minute (RPM). On HTTP 429, respect `Retry-After` and use exponential backoff with jitter; queue and smooth traffic; load-balance across multiple deployments/regions; use APIM token-limit policies to divide quota across teams; cache; reduce tokens. **Provisioned throughput (PTU)** gives reserved capacity and predictable latency, versus pay-as-you-go standard/global deployments. Often mix: PTU for baseline, pay-as-you-go for spillover.

### Q8. What does Azure AI Content Safety provide?
**Answer:** Text and image moderation for harm categories (hate, sexual, violence, self-harm) with severity levels; **Prompt Shields** (detect direct jailbreaks and indirect injection in documents); **groundedness detection** (flag ungrounded claims); protected material detection; and custom categories/blocklists. Applied at input and output, integrated into Azure OpenAI/Foundry content filters.

### Q9. How would you extend Microsoft 365 Copilot?
**Answer:** Options: **declarative agents** (custom instructions, knowledge sources, actions, no custom orchestration), **API plugins/actions** (OpenAPI-described APIs Copilot can call), **Microsoft Graph connectors** (bring external data into Copilot's index), **Teams apps/message extensions**, and **custom engine agents** (own orchestration and models, using the Microsoft 365 Agents SDK). Choose based on how much control you need over orchestration.

### Q10. (Added) What is Azure AI Foundry (Microsoft Foundry) and its key capabilities?
**Answer:** A unified platform for building AI apps and agents: model catalog (OpenAI plus open and partner models), Agent Service (managed agent runtime with tools such as search, code interpreter, and function calling), prompt and agent playgrounds, evaluation and tracing, safety and content filters, fine-tuning, and deployment/monitoring. Verify the current name and features.

### Q11. (Added) Where do Cosmos DB, Redis, Fabric, and Document Intelligence fit?
**Answer:**
- **Cosmos DB**: chat history, agent state, and even vector search with global distribution.
- **Azure Cache for Redis**: caching, semantic cache, session state.
- **Microsoft Fabric**: unified data platform for data engineering/analytics feeding AI (OneLake, data agents).
- **Document Intelligence**: layout-aware extraction from PDFs, forms, invoices.
- **Azure AI Speech/Vision/Language**: specialized services for audio, images, and NLP tasks.

### Q12. (Added) How do you control and monitor cost on Azure OpenAI?
**Answer:** Track token usage per app/team via APIM and Azure Monitor; set budgets and alerts in Cost Management; use quotas per team; route by model tier; cache; limit max tokens; use batch deployments for offline jobs; review PTU utilization; and report cost per successful task, not just per call.

### Q13. (Added) How do you set up networking and data protection for an enterprise AI app?
**Answer:** Private endpoints, VNet integration, disable public network access, customer-managed keys where required, data residency via regional deployments, DLP and Purview integration for sensitive data classification, diagnostic logging, and Defender for Cloud (AI threat protection).

---

## 6. Evaluation, Safety, and Responsible AI

### Q1. How do you build an evaluation set? How do you use LLM-as-judge, and what biases does it bring?
**Answer:** Collect real (anonymized) queries plus synthetic ones, cover core use cases, edge cases, and adversarial inputs; write expected answers or grading criteria; label a golden set with domain experts; version it. Size: start with dozens to a couple hundred well-chosen cases and grow.

**LLM-as-judge**: give a strong model a rubric and have it score outputs (groundedness, relevance, tone). Known biases: **position bias**, **verbosity bias**, **self-preference** (favors its own family's style), and inconsistency. Mitigate with clear rubrics, pairwise comparison with swapped order, calibration against human labels, temperature 0, and periodic human review.

### Q2. How do you catch regressions when you change the prompt, model, or retriever?
**Answer:** Run the eval suite automatically in CI on every change; set thresholds as quality gates; compare against the baseline per metric and per slice; do shadow testing and canary releases; A/B test online with user metrics; and version prompts, models, and indexes together so you can roll back.

### Q3. What are Microsoft's Responsible AI principles, and how do you apply them?
**Answer:** **Fairness, reliability and safety, privacy and security, inclusiveness, transparency, accountability.** Application example: test outputs across demographic groups (fairness); red-team and add safety filters and fallbacks (reliability/safety); minimize and protect data (privacy); design for accessibility and multiple languages (inclusiveness); disclose AI use, show sources, explain limits (transparency); assign owners, keep audit logs, run impact assessments and human oversight (accountability).

### Q4. How do you red-team an AI application?
**Answer:** Try to make it fail deliberately: jailbreaks, direct and indirect prompt injection, data exfiltration, harmful content, bias probes, privilege escalation via tools, and abuse of tokens/cost. Combine manual expert testing with automated tools (for example PyRIT from Microsoft, Foundry AI Red Teaming agent), log findings, fix, and add cases to the regression suite. Repeat before each major release.

### Q5. How do you handle PII, data residency, and compliance?
**Answer:** Minimize data sent to models; detect and redact PII (Azure AI Language PII detection, Presidio) before logging or prompting; choose regions that satisfy residency; apply retention and deletion policies; honor GDPR rights (access, erasure) including for memory stores and logs; encrypt in transit and at rest; document processing purposes; use Purview for classification. Involve legal/privacy early.

### Q6. What do you monitor in production?
**Answer:** Latency (time to first token, total), error and 429 rates, token usage and cost, retrieval quality signals, groundedness and relevance (sampled online evals), safety filter triggers, user feedback (thumbs, edits, abandonment), tool success rates, drift in input distribution, and agent step counts. Set alerts and dashboards; sample traces for review.

### Q7. (Added) What is model and prompt versioning, and why does it matter (LLMOps)?
**Answer:** Treat prompts, model versions, retrieval configs, and eval sets as versioned artifacts. Model providers deprecate and update versions, which can change behavior, so pin versions, test upgrades against your evals, and keep rollback paths.

### Q8. (Added) How do you collect and use user feedback?
**Answer:** Capture explicit signals (thumbs, ratings, comments) and implicit ones (regenerations, copy actions, follow-up corrections). Review negative cases, add them to the eval set, and use them to improve retrieval, prompts, or fine-tuning data, respecting privacy.

### Q9. (Added) How do you communicate uncertainty and limits to users?
**Answer:** Show citations, indicate when the system is not confident, provide "I don't know" responses, label AI-generated content, allow easy correction and feedback, and keep a human fallback for critical decisions.

---

## 7. System Design

Use this framework for each: **Requirements, Architecture, Data flow, Failure modes, Security, Evaluation, Cost, Scaling.**

### D1. Enterprise Q&A copilot over SharePoint with per-user permissions
**Requirements:** Answer employee questions from SharePoint docs with citations; respect document permissions; sub-5s response; multilingual.
**Architecture:** SharePoint connector or Graph-based ingestion, Document Intelligence for parsing, chunking plus embeddings, Azure AI Search index (hybrid plus semantic ranker) with ACL fields, orchestration API (Container Apps) calling Azure OpenAI, web/Teams front end, Entra ID sign-in.
**Data flow:** User query, on-behalf-of token gives user/group IDs, query rewrite, hybrid search with security filter, rerank, prompt with top chunks, generate with citations, groundedness check, respond and log.
**Failure modes:** Stale index, permission drift, poor parsing of tables, hallucination, quota 429s.
**Security:** Security trimming at retrieval, private endpoints, managed identity, prompt shields, PII handling, no cross-user caching.
**Evaluation:** Retrieval recall@k on labeled set, groundedness, user feedback, permission-leak tests.
**Cost/Scaling:** Cache popular queries, smaller model for rewriting, autoscale API, incremental indexing, PTU if steady load.

### D2. Customer-support agent (tickets, KB, refunds with approval)
**Requirements:** Read tickets, search KB, draft replies, issue refunds above a threshold only with human approval; full audit.
**Architecture:** Agent (Agent Framework/Foundry Agent Service) with tools: `get_ticket`, `search_kb` (RAG), `get_order`, `propose_refund`, `send_reply`; approval service; state in Cosmos DB; queue for async runs.
**Data flow:** New ticket triggers run, agent gathers context, drafts reply, if refund needed calls `propose_refund` which creates a pending approval, human approves in UI, then executes an idempotent `issue_refund`, then reply is sent.
**Failure modes:** Wrong refund, loops, tool timeouts, injection through ticket text.
**Security:** Least-privilege identity, refund caps, treat ticket content as untrusted, approval gates, idempotency keys, audit log.
**Evaluation:** Scenario suite with mocked tools, resolution accuracy, wrong-action rate (target near zero), cost per ticket.
**Scaling:** Queue-based workers, rate limits per tenant, fallback to human on low confidence.

### D3. Multi-agent code review / incident triage
**Requirements:** Analyze a PR or alert, produce findings, propose fixes, minimize false positives.
**Architecture:** Orchestrator (manager) delegates to specialists (security, performance, style/tests) that run concurrently; each has read-only repo/log tools; a synthesizer merges and dedupes; results posted as comments/tickets.
**Data flow:** Trigger (webhook), orchestrator plans, concurrent agents analyze, results into shared state, evaluator ranks by severity/confidence, replan if gaps, publish.
**Failure modes:** Noisy or conflicting findings, large diffs exceeding context, cost blowup.
**Security:** Read-only by default, sandboxed execution, secrets scanning, no auto-merge without human.
**Evaluation:** Precision/recall against known-bug benchmarks, developer acceptance rate.
**Scaling:** Chunk large diffs, cache per-file analysis, cap agents per run.

### D4. Document-processing pipeline (OCR, extraction, validation)
**Requirements:** Ingest invoices/contracts, extract fields, validate, route exceptions to humans.
**Architecture:** Blob storage, Event Grid trigger, Document Intelligence (OCR/layout), LLM for fields the model cannot extract or normalize (structured outputs), rules validation, confidence scoring, human review UI, output to database/ERP.
**Failure modes:** OCR errors, ambiguous fields, schema drift, hallucinated values.
**Controls:** Schema-constrained output, cross-field validation (totals match), confidence thresholds, human-in-loop for low confidence, audit trail.
**Evaluation:** Field-level precision/recall, straight-through-processing rate, cost per document.

### D5. AI gateway for many teams
**Requirements:** Central quotas, caching, fallback models, logging, cost attribution, safety.
**Architecture:** API Management in front of multiple Azure OpenAI deployments/regions; policies for auth (Entra), per-team token limits, load balancing with circuit breaker and priority/fallback, semantic cache (Redis), content safety hooks, logging to Event Hub/Log Analytics, cost dashboards.
**Failure modes:** Region outage, 429 storms, cache leaking across teams, log data sensitivity.
**Controls:** Cache keys scoped by tenant, redaction in logs, per-team budgets and alerts, model allow-lists.
**Scaling:** Horizontal gateway, multiple regions, PTU baseline plus spillover.

---

## 8. Coding and Hands-on

### C1. Chunk text with overlap and return chunks with metadata
```python
def chunk_text(text: str, source: str, size: int = 500, overlap: int = 50):
    if overlap >= size:
        raise ValueError("overlap must be smaller than size")
    chunks, start, idx = [], 0, 0
    step = size - overlap
    while start < len(text):
        end = min(start + size, len(text))
        chunks.append({
            "id": f"{source}-{idx}",
            "text": text[start:end],
            "source": source,
            "chunk_index": idx,
            "start_char": start,
            "end_char": end,
        })
        if end == len(text):
            break
        start += step
        idx += 1
    return chunks
```
**Talking points:** Character vs token-based sizing; splitting on sentence/paragraph boundaries; edge cases (empty text, overlap >= size).

### C2. Cosine similarity and top-k retrieval without a library
```python
import math, heapq

def cosine(a, b):
    dot = sum(x * y for x, y in zip(a, b))
    na = math.sqrt(sum(x * x for x in a))
    nb = math.sqrt(sum(y * y for y in b))
    return dot / (na * nb) if na and nb else 0.0

def top_k(query_vec, docs, k=3):
    """docs: list of (doc_id, vector). Returns k best (score, doc_id)."""
    scored = ((cosine(query_vec, vec), doc_id) for doc_id, vec in docs)
    return heapq.nlargest(k, scored)
```
**Talking points:** O(n·d) brute force; pre-normalize vectors so cosine equals dot product; use ANN (HNSW) at scale; heap keeps O(n log k).

### C3. Minimal tool-calling loop
```python
import json

def run_agent(client, model, messages, tools, tool_impls, max_steps=8):
    for _ in range(max_steps):
        resp = client.chat.completions.create(
            model=model, messages=messages, tools=tools
        )
        msg = resp.choices[0].message
        messages.append(msg)

        if not msg.tool_calls:          # final answer
            return msg.content

        for call in msg.tool_calls:
            name = call.function.name
            try:
                args = json.loads(call.function.arguments)
                result = tool_impls[name](**args)      # validate args in real code
            except Exception as e:
                result = {"error": str(e)}
            messages.append({
                "role": "tool",
                "tool_call_id": call.id,
                "content": json.dumps(result),
            })
    return "Stopped: max steps reached."
```
**Talking points:** Max-step guard, error returned to the model, argument validation, allow-listed tools only, approval for risky tools, logging.

### C4. Azure OpenAI streaming with retry/backoff
```python
import time, random
from openai import AzureOpenAI, RateLimitError, APIConnectionError
from azure.identity import DefaultAzureCredential, get_bearer_token_provider

token_provider = get_bearer_token_provider(
    DefaultAzureCredential(), "https://cognitiveservices.azure.com/.default"
)
client = AzureOpenAI(
    azure_endpoint="https://<resource>.openai.azure.com",
    azure_ad_token_provider=token_provider,
    api_version="2024-10-21",
)

def stream_chat(messages, deployment, max_retries=5):
    for attempt in range(max_retries):
        try:
            stream = client.chat.completions.create(
                model=deployment, messages=messages, stream=True
            )
            for chunk in stream:
                if chunk.choices and chunk.choices[0].delta.content:
                    yield chunk.choices[0].delta.content
            return
        except (RateLimitError, APIConnectionError):
            if attempt == max_retries - 1:
                raise
            time.sleep(min(2 ** attempt, 30) + random.random())  # backoff + jitter
```
**Talking points:** Managed identity (no keys); honor `Retry-After`; do not retry after partial output without a strategy; the API version string changes, so check the current one.

### C5. Structured extraction with validation and repair
```python
from pydantic import BaseModel, ValidationError
from typing import Optional

class Invoice(BaseModel):
    vendor: str
    invoice_number: str
    total: float
    currency: str
    due_date: Optional[str] = None

def extract_invoice(client, model, text, max_repairs=2):
    messages = [
        {"role": "system", "content": "Extract invoice fields. Return JSON only."},
        {"role": "user", "content": text},
    ]
    for _ in range(max_repairs + 1):
        resp = client.chat.completions.create(
            model=model, messages=messages, temperature=0,
            response_format={"type": "json_object"},
        )
        raw = resp.choices[0].message.content
        try:
            return Invoice.model_validate_json(raw)
        except ValidationError as e:
            messages += [
                {"role": "assistant", "content": raw},
                {"role": "user", "content": f"Invalid: {e}. Fix and return JSON only."},
            ]
    raise ValueError("Extraction failed after repairs")
```
**Talking points:** Prefer schema-constrained structured outputs where supported; cross-field checks (line items sum to total); route failures to human review.

### C6. Caching and rate limiting for an LLM endpoint
```python
import time, hashlib, threading

class TokenBucket:
    def __init__(self, rate_per_sec, capacity):
        self.rate, self.capacity = rate_per_sec, capacity
        self.tokens, self.last = capacity, time.monotonic()
        self.lock = threading.Lock()

    def allow(self, cost=1):
        with self.lock:
            now = time.monotonic()
            self.tokens = min(self.capacity, self.tokens + (now - self.last) * self.rate)
            self.last = now
            if self.tokens >= cost:
                self.tokens -= cost
                return True
            return False

cache = {}   # use Redis with TTL in production

def cached_llm(user_scope, prompt, call_llm, bucket):
    key = hashlib.sha256(f"{user_scope}:{prompt}".encode()).hexdigest()
    if key in cache:
        return cache[key]
    if not bucket.allow():
        raise RuntimeError("429: rate limited")
    cache[key] = call_llm(prompt)
    return cache[key]
```
**Talking points:** Cache key includes user/permission scope; TTL and invalidation; semantic cache with embeddings; per-user and per-tenant limits; token-based (not just request-based) limits.

### C7. (Added) Reciprocal Rank Fusion for hybrid search
```python
def rrf(rankings, k=60):
    """rankings: list of ranked lists of doc ids (best first)."""
    scores = {}
    for ranking in rankings:
        for rank, doc in enumerate(ranking, start=1):
            scores[doc] = scores.get(doc, 0) + 1 / (k + rank)
    return sorted(scores, key=scores.get, reverse=True)

# fused = rrf([bm25_ids, vector_ids])
```

### C8. (Added) Async concurrent LLM calls with a concurrency limit
```python
import asyncio

async def bounded_gather(items, worker, limit=5):
    sem = asyncio.Semaphore(limit)
    async def run(item):
        async with sem:
            return await worker(item)
    return await asyncio.gather(*(run(i) for i in items), return_exceptions=True)
```
**Talking points:** Respect rate limits with the semaphore; handle exceptions per item; C# equivalent uses `SemaphoreSlim` with `Task.WhenAll`.

### C9. (Added) Recall@k and MRR
```python
def recall_at_k(retrieved, relevant, k):
    return len(set(retrieved[:k]) & set(relevant)) / len(relevant) if relevant else 0.0

def mrr(retrieved, relevant):
    for i, doc in enumerate(retrieved, start=1):
        if doc in relevant:
            return 1 / i
    return 0.0
```

### C10. Standard DSA and async expectations
Expect medium-difficulty problems: arrays/strings, hash maps, two pointers, sliding window, BFS/DFS on graphs and trees, heaps, intervals, basic dynamic programming. Also expect: explaining async/await, concurrency pitfalls (race conditions, deadlocks), REST API design, unit testing with mocks (mock the LLM), and clean code. Practice in the language you will interview in (Python or C#).

---

## 9. Behavioral

Use **STAR** (Situation, Task, Action, Result) and quantify results. Below are what interviewers look for and a sample structure.

### B1. Tell me about an AI project you shipped. What went wrong, and how did you fix it?
**How to answer:** Give context and the business goal, your role, key design decisions (why RAG vs fine-tuning, model choice), a *specific* failure (for example hallucinations on edge cases, retrieval misses, cost overrun, latency), how you diagnosed it (evals, traces, error analysis), the fix, and the measurable result (for example groundedness up from 78% to 93%, cost down 40%). Reflect on what you would do differently.

### B2. Describe a time you had to say no to an AI feature because of risk or quality.
**How to answer:** Show judgment: identify the risk (safety, privacy, accuracy), gather evidence (eval results, red-team findings), propose alternatives (narrower scope, human-in-loop, phased rollout), communicate clearly to stakeholders, and describe the outcome. Demonstrates Responsible AI mindset and customer obsession.

### B3. How do you explain non-determinism and limitations to non-technical stakeholders?
**How to answer:** Use analogies (a smart new hire who is sometimes confidently wrong), show concrete examples, explain metrics in business terms (accuracy on real tasks, cost per query), set expectations with confidence levels, and describe mitigations (citations, human review, monitoring). Avoid jargon.

### B4. Tell me about a time you disagreed with a teammate on a technical approach.
**How to answer:** Present the disagreement respectfully, how you gathered data (prototype, benchmark, eval), listened to their view, reached a decision (or disagreed and committed), and the outcome. Show humility and focus on the customer and the result.

### B5. How do you keep up with the pace of change in AI?
**How to answer:** Concrete habits: read papers and release notes, build small prototypes, follow official docs and engineering blogs, take part in communities, and internal knowledge sharing. Emphasize learning fundamentals (which change slowly) and evaluating new tools critically rather than chasing hype.

### B6. Why Microsoft? Why applied AI rather than research?
**How to answer:** Tie to Microsoft's mission (empower every person and organization), scale of enterprise and developer impact, the Azure/Foundry/Copilot ecosystem, and its focus on responsible AI. For applied AI: you enjoy turning capabilities into reliable products, measuring real-world impact, and solving customer problems end to end.

### B7. (Added) Tell me about a time you worked with ambiguity.
**How to answer:** Describe unclear requirements, how you clarified goals, ran quick experiments, made assumptions explicit, and iterated with feedback.

### B8. (Added) Tell me about a time you failed or made a mistake.
**How to answer:** Pick a real mistake, own it, explain the impact, what you did to fix it and prevent recurrence. Microsoft values a growth mindset.

### B9. (Added) Tell me about a time you influenced without authority or worked across teams.
**How to answer:** Show how you built alignment with data and demos, understood others' priorities, and delivered shared results.

### B10. (Added) How would you handle a production incident where your AI feature produced harmful or wrong output?
**How to answer:** Contain first (disable feature or tighten filters), communicate, investigate using traces, fix root cause, add regression tests and monitoring, run a blameless post-mortem, and follow Responsible AI incident processes.

---

## 10. Quick-Fire Round (Added)

| Question | Short answer |
|---|---|
| RAG vs fine-tuning for new company knowledge? | RAG. Fine-tuning is for behavior/style. |
| Why hybrid search? | Vectors miss exact terms, keywords miss synonyms. |
| First step when RAG answers are wrong? | Inspect retrieved chunks. |
| What does a reranker do? | Re-scores top candidates jointly with the query for precision. |
| Best way to stop cross-user data leaks in RAG? | Query-time ACL filters, never prompt-based hiding. |
| How to make retries safe? | Idempotency keys. |
| Handling 429 errors? | Backoff with jitter, honor Retry-After, load balance, PTU. |
| Auth to Azure services? | Managed Identity plus RBAC, no keys. |
| What is indirect prompt injection? | Malicious instructions hidden in retrieved or tool content. |
| Agent vs workflow? | Workflow when the steps are known; agent when the path is unknown. |
| MCP vs A2A? | MCP is agent-to-tool; A2A is agent-to-agent. |
| Temperature 0 means fully deterministic? | No, only mostly. |
| Which metrics for retrieval? | recall@k, precision@k, MRR, nDCG. |
| Which metrics for generation? | Groundedness, relevance, completeness, citation accuracy. |
| LoRA in one line? | Train small adapters, freeze base weights. |
| KV cache in one line? | Reuse past keys and values to speed up decoding. |
| Where to put approval gates? | Before irreversible or external side effects. |
| What to log for agents? | Prompts, tool calls, results, tokens, latency, errors (with redaction). |
| Semantic cache risk? | Serving one user's cached answer to another user. |
| Microsoft RAI principles? | Fairness, reliability and safety, privacy and security, inclusiveness, transparency, accountability. |

---

## Final Prep Checklist
- [ ] Build one small end-to-end RAG app on Azure AI Search plus Azure OpenAI with evals.
- [ ] Build one tool-calling agent with an approval step and tracing.
- [ ] Practice two system designs out loud using the 8-part framework.
- [ ] Prepare 4 or 5 STAR stories mapped to Microsoft values (growth mindset, customer obsession, diverse and inclusive, one Microsoft).
- [ ] Review current Microsoft product names and recent announcements.
- [ ] Practice coding in Python or C# without autocomplete.
