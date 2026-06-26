# 🧠 AI Architect Interview Questions — Krish Services Group

> **Role:** AI Architect (M4) | **Panel Use:** Technical Screening & Technical Discussion  
> Each question is designed for a **~2 minute** answer.

---

<details>
<summary><strong>Q1. Walk us through how you would design a production-grade RAG system on Azure. What are the key architectural decisions?</strong></summary>

**Strong Answer:**

A production RAG system on Azure starts with the **retrieval layer** — Azure AI Search with hybrid retrieval (keyword + vector) is my default. I'd use Azure AI Document Intelligence for ingestion and chunking of unstructured docs, embedding them via Azure OpenAI (text-embedding-3-large), and indexing into AI Search with semantic ranker enabled.

For the LLM layer, Azure OpenAI (GPT-4o or equivalent) sits behind Azure APIM acting as the AI gateway — this gives you rate limiting, token budgeting, and a single auth surface. Orchestration goes through Semantic Kernel or Azure AI Foundry Prompt Flow depending on how complex the query routing is.

Key decisions I always lock early: **chunk size and overlap** (affects recall quality), **reranking strategy** (semantic ranker vs. cross-encoder), **data boundary enforcement** via Entra ID + Private Link so we never exfiltrate regulated data, and **eval harness** using Ragas or PromptFlow evals before any production promotion.

Observability via Azure Monitor + App Insights with token cost and latency as first-class metrics. AI systems I can't measure don't ship.

</details>

---

<details>
<summary><strong>Q2. How do you decide between Azure AI Foundry Agent Service, Semantic Kernel, LangGraph, and AutoGen for a multi-agent workflow?</strong></summary>

**Strong Answer:**

My default is **Azure AI Foundry Agent Service + Semantic Kernel** because they give you managed agent runtime, built-in tool-use, memory, and native Azure integrations out of the box — reducing infrastructure you have to own.

I reach for **LangGraph** when the workflow has complex, stateful branching — think conditional loops, human-in-the-loop approval gates, or workflows where agent state needs to be checkpointed and resumed. Its graph-based model makes those patterns explicit and debuggable.

**AutoGen** comes in when I need rapid prototyping of conversational multi-agent patterns, or when the team is research-oriented and needs flexibility. It's less opinionated, which is a strength in exploration and a liability in production.

**NVIDIA NeMo** is a fit when we're doing fine-tuning or deploying open-source models at scale with guardrails — not for pure orchestration.

The deciding factors are: how stateful is the workflow, does it need Azure-native auth and compliance hooks, and what's the team's existing familiarity. I document this as CoE policy so teams aren't re-litigating it per project.

</details>

---

<details>
<summary><strong>Q3. Describe your approach to eval and observability for LLM-based systems. What does "production-ready" mean to you in this context?</strong></summary>

**Strong Answer:**

Production-ready means you have **three layers of measurement**: pre-ship evals, runtime monitoring, and regression detection.

Pre-ship: I run automated eval suites — using Ragas for RAG quality (faithfulness, answer relevancy, context recall), PromptFlow evals or Braintrust for broader LLM quality metrics. Every prompt change triggers a regression run before merge.

Runtime: Azure Monitor + App Insights capture token usage, latency percentiles (p50/p95/p99), error rates, and cost per query. I instrument traces at the span level so I can see retrieval latency vs. generation latency separately — LangSmith or Azure AI Foundry's built-in tracing work well here.

Drift detection: I log a sample of production inputs and outputs, run periodic quality scoring, and alert on degradation. Model updates from the provider can shift output distributions silently — this catches it.

"Production-ready" to me means: eval suite is green, dashboards are live before launch, on-call knows what an alert means, and there's a rollback path. A demo that works in a notebook is not production.

</details>

---

<details>
<summary><strong>Q4. How do you enforce Responsible AI principles in practice, especially for regulated-industry clients like healthcare or financial services?</strong></summary>

**Strong Answer:**

Responsible AI in regulated industries is an architecture concern, not a checklist you add at the end.

First, **data boundaries**: Entra ID for identity, Private Link so data never leaves the tenant's network, RBAC scoped per role and data classification. For HIPAA clients, that means PHI never hits a shared endpoint or gets logged in plain text.

Second, **prompt and output safety**: Azure AI Content Safety sits in the pipeline to catch jailbreak attempts, PII leakage, and harmful content — both on the way in and on generated outputs. I configure it with client-specific category thresholds, not defaults.

Third, **audit trails**: every LLM call is logged with inputs, outputs, and the identity of the requester — immutable, queryable. This is table stakes for SOC 2 and HIPAA audit readiness.

Fourth, **governance patterns**: I build reference architectures with these controls baked in, so downstream engineers can't accidentally ship a non-compliant system. Red-teaming and adversarial eval are part of the pre-launch checklist.

The goal is making compliance the path of least resistance for the engineering team.

</details>

---

<details>
<summary><strong>Q5. You're starting a new AI engagement with a client. Walk us through how you run technical discovery and shape the solution architecture.</strong></summary>

**Strong Answer:**

Discovery starts with **four questions**: What's the business outcome they're optimizing for? What data do they have and where does it live? What does success look like in 90 days? And what are the compliance or security constraints?

From there I do a data inventory — understanding data freshness, volume, access patterns, and sensitivity drives 80% of the architecture decisions. A RAG system looks very different if the data is a static SharePoint corpus vs. a real-time operational database.

I whiteboard a candidate architecture during discovery, not after — it forces alignment on assumptions and surfaces blockers early. I'll typically propose two or three patterns with explicit tradeoffs rather than one "answer," because clients make better decisions when they understand what they're trading off.

Then I lock the MVP scope tightly: one use case, one data source, measurable eval criteria, deployable on Azure within the agreed timeline. Scope creep in AI projects is where timelines die.

Deliverables from discovery: a written solution architecture doc, a risk register, a rough LOE, and the eval criteria we'll use to call the project a success. That feeds directly into the SOW.

</details>

---

<details>
<summary><strong>Q6. How do you approach deployment topology for AI systems on Azure — when do you use AKS vs Container Apps, and where does APIM fit?</strong></summary>

**Strong Answer:**

My default for most GenAI workloads is **Azure Container Apps** — it's simpler to operate, has built-in scaling including scale-to-zero, and removes the cluster management overhead of AKS. For teams without dedicated platform engineers, that operational simplicity is a real advantage.

I move to **AKS** when: the workload needs GPU nodes for model inference, we need fine-grained control over networking (custom CNI, pod-level network policies), or we're co-locating AI inference with other services that are already AKS-resident.

**APIM** is always in the picture as the AI gateway — it gives me a single surface for auth, rate limiting per client or team, token-budget enforcement, retry logic, and a stable API contract that decouples consumers from backend model changes. When Azure OpenAI has an outage or I want to swap models, APIM is where I route around it without touching downstream code.

Cost-wise: Container Apps charges on consumption, AKS on provisioned nodes. For bursty AI workloads, Container Apps wins on cost unless you need persistent GPU capacity.

</details>

---

<details>
<summary><strong>Q7. How do you design the memory and state management layer for a long-running agent system?</strong></summary>

**Strong Answer:**

I think about agent memory in three tiers: **in-context**, **short-term**, and **long-term**.

In-context memory is just the conversation window — cheapest, most reliable, but limited by the model's context length. I manage what goes in it carefully via summarization and selective retrieval rather than stuffing everything.

Short-term memory — session-scoped state — I typically store in Cosmos DB. It's fast, globally distributed, and the vector extension means I can do similarity retrieval over recent turns without a separate vector store. Azure AI Foundry Agent Service has a built-in thread/session model that handles this well for straightforward cases.

Long-term memory — knowledge that persists across sessions — goes into Azure AI Search (vector-indexed) or Cosmos DB depending on retrieval pattern. I'm deliberate about what gets promoted to long-term: everything is noise until proven useful.

State management for multi-agent workflows is trickier. I use LangGraph's checkpointing for durable agent state when workflows can pause and resume — it writes state to a configurable backend (Redis, Postgres, or Cosmos). The key design principle: every state transition should be recoverable, and human-in-the-loop approval gates must be explicit in the graph.

</details>

---

<details>
<summary><strong>Q8. You need to build and run an R&D cadence for an AI CoE. How do you structure it so it produces real output and doesn't become just meetings?</strong></summary>

**Strong Answer:**

The failure mode for R&D cadences is exactly that — they become reading-group discussions that produce no artifacts and no decisions. I avoid it with three rules: **time-boxed spikes, mandatory prototypes, and kill criteria**.

Concretely: every R&D topic gets a two-week spike with a defined output — either a working prototype, a written decision memo, or an explicit "we evaluated this, here's why it doesn't fit." No spike runs longer than two weeks without a checkpoint. If it's not worth two weeks of focus, it's not worth putting on the roadmap.

For cadence structure: I run a monthly "tech radar" review where the team collectively moves things from "assess" to "adopt/hold/don't use" based on spikes and production experience. That output becomes the CoE's policy document — what tools we standardize on and why.

Enablement is separate from R&D: monthly workshops or code labs that translate R&D output into skill-building for the broader AI engineering team. Azure certification paths give engineers a structured external milestone to work toward alongside internal upskilling.

The measure of success: how quickly can a new engagement team stand up a production-ready pattern without reinventing decisions the CoE already made?

</details>

---

<details>
<summary><strong>Q9. How do you reason about cost and latency tradeoffs when selecting models and retrieval strategies for a client AI system?</strong></summary>

**Strong Answer:**

I start by understanding the **latency budget and the query volume** — they determine how much I can spend per call. A real-time chat interface has very different constraints from a batch summarization pipeline.

For model selection: GPT-4o is my default for reasoning-heavy tasks, but I'll drop to GPT-4o-mini or an equivalent smaller model for classification, routing, or high-volume extraction where quality requirements are lower. The cost difference is often 10-20x — that's not trivial at scale. I benchmark both against the eval criteria before committing.

For retrieval: hybrid search (keyword + vector) is almost always worth the marginal cost over pure vector search because it improves recall significantly. But the reranker is where cost accumulates — semantic reranking is expensive, so I apply it only to the top-k candidates, not the full result set.

Caching is underused. For high-traffic RAG systems, I implement a semantic cache — similar queries return cached responses without a full LLM call. Azure Redis Cache or a simple vector similarity layer on Cosmos handles this.

Every architecture I ship has a cost model: estimated tokens per query × query volume × model price, with a monthly projection. Surprises in AI cost are almost always a failure to model it upfront.

</details>

---

<details>
<summary><strong>Q10. How do you lead a small team of AI engineers while staying technically deep yourself? How do you balance architecture, code, and people responsibilities?</strong></summary>

**Strong Answer:**

The trap for senior architects is becoming a pure reviewer — you lose technical credibility and your judgment starts lagging the team's. I stay in the code by owning specific components: I'll write the eval harness, prototype the new retrieval pattern, or build the reference implementation for a new agent pattern myself. That keeps me honest about what's actually hard.

For the team, I try to make the architecture decisions clear and documented enough that engineers can move fast without blocking on me. Reference architectures, ADRs (Architecture Decision Records), and working sample implementations are my primary leverage — they scale my judgment without requiring my presence in every meeting.

On people development: I run architecture reviews as teaching sessions, not approvals. I give engineers the chance to defend their design choices before I offer mine. That builds their judgment faster than me just telling them what to do.

The balance shifts depending on the week. During presales or early project design, I'm 70% architecture. During active delivery, I'm 40% code, 40% reviews, 20% people. I've learned to be explicit about which mode I'm in so the team knows when to pull me in versus when to run with it.

The leading indicator I watch: if engineers are escalating decisions that they should be making themselves, I'm not delegating or enabling clearly enough.

</details>

---

*Generated for internal interview use — Krish Services Group AI CoE | Role: AI Architect M4*
