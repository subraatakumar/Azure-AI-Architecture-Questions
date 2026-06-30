Here are 10 questions, ordered from foundational to deep, with what a strong answer should cover:

---

**1. You need to build a RAG system on Azure. Walk me through your complete architecture from document ingestion to response generation.**

*Expect:* Azure Blob/ADLS for storage, Azure Document Intelligence for parsing, Azure AI Search with hybrid retrieval (vector + keyword), chunking strategy reasoning, Azure OpenAI for embeddings and generation, re-ranking, and prompt construction. Weak candidates just say "store in vector DB and query."

---

**2. Why would you choose Azure AI Foundry Agent Service over building your own agent loop with Semantic Kernel or LangGraph?**

*Expect:* Clear tradeoffs — Foundry gives managed infrastructure, built-in tool calling, and state management but less flexibility; Semantic Kernel gives more control for complex orchestration; LangGraph for highly stateful conditional flows. Strong candidates give context-specific reasoning, not a generic answer.

---

**3. How do you enforce data boundaries in a multi-tenant Azure AI application serving regulated-industry clients?**

*Expect:* Entra ID for identity, Private Link to keep traffic off public internet, RBAC on Azure AI Search indexes per tenant, Key Vault for secrets, network isolation via VNet. Bonus: mention of data residency and HIPAA BAA with Microsoft.

---

**4. An agent in your multi-agent system is making an external API call that occasionally times out. How do you handle this at the architecture level, not just the code level?**

*Expect:* Retry with exponential backoff, circuit breaker pattern, fallback agent or graceful degradation path, dead letter queue for failed tasks, Azure API Management as a gateway with retry policies built in. Red flag: only talking about deployment rollback like Manas did.

---

**5. How do you evaluate whether your RAG system is actually working correctly in production?**

*Expect:* Ragas metrics (faithfulness, answer relevance, context precision/recall), LangSmith or Azure Monitor for tracing, regression eval suites against a golden dataset, drift detection over time, cost and latency telemetry. Weak candidates only mention "accuracy."

---

**6. A client asks you to deploy an LLM-backed assistant on Azure that must never expose PII in responses. What is your implementation approach?**

*Expect:* Azure AI Content Safety for output scanning, PII detection via Azure AI Language service, prompt-level instructions for grounding, output post-processing layer, audit logging via Application Insights. Strong candidates mention testing this with adversarial inputs before go-live.

---

**7. When would you use Azure AI Search hybrid retrieval over pure vector search, and how do you tune it?**

*Expect:* Hybrid combines BM25 keyword search with vector similarity — better for queries where exact terms matter (product codes, names, IDs). Tuning via RRF (Reciprocal Rank Fusion) score weighting, semantic reranking layer on top, adjusting top-K and score thresholds based on eval results.

---

**8. How would you design the deployment topology for a high-traffic Azure AI application that needs to stay within cost and latency SLAs?**

*Expect:* Azure Container Apps or AKS for the application layer, APIM as the AI gateway (rate limiting, token quotas, routing to multiple Azure OpenAI deployments), PTU (Provisioned Throughput Units) vs. pay-as-you-go decision, caching frequent queries via Azure Cache for Redis, autoscaling policies.

---

**9. A client in healthcare wants to use your AI platform but needs a full audit trail of every LLM interaction. How do you implement that on Azure?**

*Expect:* Application Insights for tracing with custom telemetry, Azure Monitor Log Analytics workspace, storing prompt/response pairs in Cosmos DB or ADLS with retention policies, Entra ID audit logs for access, immutable storage for compliance. Mention of HIPAA BAA and who in the Azure stack is covered under it.

---

**10. You are presenting a RAG-based solution architecture to a client CTO who is worried about hallucinations in a regulated environment. What do you tell them and what architectural guardrails do you propose?**

*Expect:* Grounding via retrieval (model only answers from retrieved documents), confidence scoring, Azure AI Content Safety, human-in-the-loop for low-confidence responses, eval harness with faithfulness metrics, citation of sources in responses so users can verify. Strong candidates also mention ongoing monitoring, not just pre-launch testing.

---

These 10 questions together will clearly separate someone with genuine hands-on Azure AI Architect experience from someone who has only read about it or worked at a surface level.
