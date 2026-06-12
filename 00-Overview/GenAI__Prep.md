# GenAI / Agentic AI Architect — Interview Prep

> **Role Focus:** Design, build, and govern enterprise-grade GenAI and Agentic AI systems. This document is structured architect-first — starting with system design, architecture patterns, and decision frameworks before diving into implementation details.

---

## Table of Contents

### Architecture & Design
1. [GenAI System Design & Architecture](#genai-system-design--architecture)
2. [Enterprise Problem Scenarios (Interview Questions)](#enterprise-problem-scenarios-interview-questions)
3. [GenAI Lifecycle](#genai-lifecycle)

### AI Core
4. [LLM Fundamentals](#llm-fundamentals)
5. [RAG in Depth (Enterprise Solutions)](#rag-in-depth-enterprise-solutions)
6. [Agent Orchestration](#agent-orchestration)
7. [AI Memory](#ai-memory)
8. [Prompt Engineering](#prompt-engineering)
9. [Fine-Tuning](#fine-tuning)
10. [AI Optimization Topics](#ai-optimization-topics)

### Frameworks & Protocols
11. [LangChain / LangGraph / LangFlow / LangFuse](#langchain--langgraph--langflow--langfuse)
12. [MCP Server (Model Context Protocol)](#mcp-server-model-context-protocol)
13. [AG-UI (Agent-User Interface Protocol)](#ag-ui-agent-user-interface-protocol)

### Azure Platform
14. [Azure Cloud Topics](#azure-cloud-topics)
15. [Azure AI Foundry](#azure-ai-foundry)
16. [Document Intelligence & OCR](#document-intelligence--ocr)

### Operations & Governance
17. [GenAIOps / LLMOps / MLOps](#genaiops--llmops--mlops)
18. [Observability and Tracing](#observability-and-tracing)
19. [Evaluation (Online and Offline)](#evaluation-online-and-offline)
20. [Cost Governance for LLMs](#cost-governance-for-llms)

### Security & Ethics
21. [AI Security Topics](#ai-security-topics)
22. [Red Teaming](#red-teaming)
23. [AI Ethics and Responsible AI](#ai-ethics-and-responsible-ai)
24. [EU AI Act / Governance](#eu-ai-act--governance)

### Infrastructure & Data
25. [DevOps for AI Applications](#devops-for-ai-applications)
26. [Database Technologies](#database-technologies)

### Development
27. [.NET (For AI Development)](#net-for-ai-development)
28. [C# Basics (For AI Development)](#c-basics-for-ai-development)
29. [SQL Basics (For AI Development)](#sql-basics-for-ai-development)
30. [Python (For AI Development)](#python-for-ai-development)

### Interview Prep
31. [Managerial / Architect Interview Prep](#managerial--architect-interview-prep)
32. [Quick Reference — Framework Comparison](#quick-reference--framework-comparison)
33. [Interview Cheat Sheet — Key Numbers to Remember](#interview-cheat-sheet--key-numbers-to-remember)
34. [Final Preparation Checklist](#final-preparation-checklist)

---

## GenAI System Design & Architecture

### Architect's Mental Model

* **When to use GenAI** — classification (use traditional ML), generation/reasoning/summarization (use LLMs), structured data queries (use SQL/APIs)
* **Build vs. buy vs. integrate** — custom fine-tuned model vs. API-based (OpenAI/Azure) vs. open-source self-hosted
* **Synchronous vs. asynchronous** — real-time chat vs. batch document processing vs. event-driven pipelines
* **Stateless vs. stateful agents** — API-per-request vs. persistent agent sessions with memory

### Non-Functional Requirements (NFRs) for GenAI

* **Latency** — time-to-first-token (TTFT), tokens-per-second (TPS), P95/P99 latency targets
* **Throughput** — concurrent requests, requests per minute (RPM), tokens per minute (TPM)
* **Availability** — multi-region failover, graceful degradation when LLM unavailable
* **Scalability** — horizontal scaling of inference, auto-scaling based on queue depth
* **Cost** — cost-per-query, monthly budget caps, cost attribution per tenant/feature
* **Security** — data residency, PII handling, prompt injection defense, audit logging
* **Compliance** — GDPR, HIPAA, SOC2, EU AI Act considerations

### High-Level Architecture Patterns

| Pattern | When to Use | Key Components |
|---------|-------------|----------------|
| **Simple Chat API** | Single-turn Q&A, chatbots | API Gateway → LLM → Response |
| **RAG Pipeline** | Knowledge-grounded answers | Ingestion → Vector DB → Retrieval → LLM |
| **Event-Driven RAG** | Async document processing | Blob → Event/Queue → Processor → Index |
| **Multi-Agent System** | Complex workflows, tool use | Orchestrator → Specialized Agents → Tools |
| **Human-in-the-Loop** | Approval workflows, sensitive actions | Agent → Approval Queue → Human → Resume |
| **Agentic RAG** | Multi-hop reasoning, dynamic retrieval | Agent decides when/what to retrieve |

### Capacity Planning for LLMs

* **Token budget estimation** — avg tokens per request × requests per day × cost per token
* **Rate limit planning** — Azure OpenAI TPM/RPM limits, APIM throttling policies
* **PTU vs. consumption** — PTU for >1M tokens/day predictable load; consumption for spiky/dev
* **Embedding throughput** — batch embedding jobs, queue-based processing for large corpus
* **Vector DB sizing** — dimensions × vectors × metadata overhead; index type (HNSW vs. IVF)

### Reference Architectures (Azure)

* **Baseline RAG** — Azure OpenAI + AI Search + App Service/Container Apps
* **Enterprise RAG** — + APIM (AI Gateway) + Private Endpoints + Managed Identity + Key Vault
* **Multi-tenant RAG** — per-tenant indexes OR metadata filtering + RBAC
* **Agentic Workloads** — Azure Functions (Durable) / Container Apps + Service Bus + Agent Framework
* **Batch Processing** — Blob trigger → Service Bus → Functions → AI Search indexer

### Decision Framework for Model Selection

| Criteria | GPT-4o | GPT-4o-mini | Claude 3.5 | Llama 3 / Mistral | Phi-3/4 |
|----------|--------|-------------|------------|-------------------|---------|
| Reasoning | ★★★★★ | ★★★ | ★★★★★ | ★★★★ | ★★★ |
| Cost | $$$$$ | $$ | $$$$ | $ (self-hosted) | $ |
| Latency | Medium | Fast | Medium | Variable | Fast |
| Data Privacy | Cloud | Cloud | Cloud | On-prem possible | On-prem/Edge |
| Context Window | 128K | 128K | 200K | 8K-128K | 4K-128K |
| Best For | Complex tasks | High volume simple | Long context | Cost-sensitive | Edge/Mobile |

---

## Enterprise Problem Scenarios (Interview Questions)

> **Format:** Each scenario describes a real enterprise problem. Be prepared to whiteboard the architecture, discuss tradeoffs, and explain your decisions.

### Scenario 1: Event-Driven RAG Pipeline

**Problem:** "Design an enterprise-grade, event-driven RAG pipeline on Azure. Files uploaded to Blob Storage should be automatically processed, chunked, embedded, and indexed. Provide a query endpoint for GPT-4o-powered answers grounded in documents."

**Architecture:**
```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│ Blob Storage│────▶│ Service Bus  │────▶│ Azure Function  │────▶│ AI Search    │
│ (Upload)    │     │ (Queue)      │     │ (Processor)     │     │ (Index)      │
└─────────────┘     └──────────────┘     └─────────────────┘     └──────────────┘
                                                │                        │
                                                ▼                        │
                                         ┌─────────────┐                 │
                                         │ Azure OpenAI │                │
                                         │ (Embedding)  │                │
                                         └─────────────┘                 │
                                                                         │
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐            │
│ User Query  │────▶│ APIM         │────▶│ Query Function  │◀───────────┘
│ (API)       │     │ (AI Gateway) │     │ (RAG Endpoint)  │
└─────────────┘     └──────────────┘     └─────────────────┘
                                                │
                                                ▼
                                         ┌─────────────┐
                                         │ Azure OpenAI │
                                         │ (GPT-4o)     │
                                         └─────────────┘
```

**Key Discussion Points:**
* **Why Service Bus?** — Decoupling, retry semantics, dead-letter queue for poison messages, sessions for ordering
* **Chunking strategy** — Semantic chunking with overlap, parent-child hierarchy for context
* **Error handling** — DLQ monitoring, alerting, manual reprocessing workflow
* **Multi-tenancy** — Metadata filtering vs. separate indexes per tenant
* **Security** — Managed identity, private endpoints, Key Vault for secrets
* **Cost optimization** — Batch embedding, integrated vectorization in AI Search

---

### Scenario 2: Multi-Agent Customer Support System

**Problem:** "Design a multi-agent system for customer support. Agents should handle: (1) FAQ/knowledge queries, (2) order status lookups, (3) refund processing with human approval, (4) escalation to human agents."

**Architecture:**
```
┌────────────────┐
│  User Message  │
└───────┬────────┘
        ▼
┌────────────────┐
│  Router Agent  │──────────────────────────────────────┐
│  (Classifier)  │                                      │
└───────┬────────┘                                      │
        │                                               │
   ┌────┴────┬──────────┬──────────┐                   │
   ▼         ▼          ▼          ▼                   ▼
┌──────┐ ┌──────┐ ┌──────────┐ ┌──────────┐    ┌──────────┐
│ FAQ  │ │Order │ │ Refund   │ │ Escalate │    │ Fallback │
│Agent │ │Agent │ │ Agent    │ │ Agent    │    │ (GPT-4o) │
└──┬───┘ └──┬───┘ └────┬─────┘ └────┬─────┘    └──────────┘
   │        │          │            │
   │        │          ▼            ▼
   │        │    ┌──────────┐  ┌──────────┐
   │        │    │ Human    │  │ Ticket   │
   │        │    │ Approval │  │ System   │
   │        │    │ Queue    │  │ (Zendesk)│
   │        │    └──────────┘  └──────────┘
   │        │
   │        ▼
   │   ┌──────────┐
   │   │ Order DB │
   │   │ (API)    │
   │   └──────────┘
   ▼
┌──────────┐
│ AI Search│
│ (KB)     │
└──────────┘
```

**Key Discussion Points:**
* **Router design** — LLM-based classification vs. intent detection model vs. keyword rules
* **Agent Framework choice** — Microsoft Agent Framework for .NET/Azure, LangGraph for Python
* **Human-in-the-loop** — `RequestInfoExecutor` pattern, approval queue with SLA
* **State management** — Conversation history, session persistence, handoff context
* **Guardrails** — Refund limits, PII masking, action confirmation prompts
* **Observability** — Trace across agent hops, tool call logging, latency per agent

---

### Scenario 3: Document Intelligence Pipeline

**Problem:** "A legal firm wants to process contracts (PDFs, scans) and extract key clauses, dates, parties, and obligations. Build a pipeline that handles 10,000 documents/day with 99% accuracy requirement."

**Architecture:**
```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│ Document    │────▶│ Queue        │────▶│ OCR/Layout      │
│ Upload      │     │ (Service Bus)│     │ (Doc Intel)     │
└─────────────┘     └──────────────┘     └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ Extraction      │
                                         │ (GPT-4o Vision) │
                                         └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ Validation      │
                                         │ (Rules + LLM)   │
                                         └────────┬────────┘
                                                  │
                           ┌──────────────────────┼──────────────────────┐
                           ▼                      ▼                      ▼
                    ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
                    │ Structured  │       │ Vector DB   │       │ Human Review│
                    │ DB (SQL)    │       │ (Search)    │       │ Queue       │
                    └─────────────┘       └─────────────┘       └─────────────┘
```

**Key Discussion Points:**
* **OCR choice** — Azure Document Intelligence (layout model) vs. GPT-4o Vision for complex layouts
* **Extraction prompt design** — Few-shot examples, structured JSON output, confidence scores
* **Accuracy vs. cost** — Two-pass extraction: fast model first, GPT-4o for low-confidence
* **Human-in-the-loop** — Confidence threshold routing, active learning from corrections
* **Scaling** — Parallel processing, batch API for cost, Flex Consumption functions
* **Evaluation** — Ground truth dataset, precision/recall per field, regression testing

---

### Scenario 4: Real-Time AI Co-Pilot for Sales

**Problem:** "Build a real-time AI assistant that helps sales reps during calls. It should: listen to conversation, suggest responses, surface relevant product info, and auto-generate call summaries."

**Architecture:**
```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│ Audio Stream│────▶│ Speech-to-   │────▶│ Real-time       │
│ (WebSocket) │     │ Text (Azure) │     │ Transcript      │
└─────────────┘     └──────────────┘     └────────┬────────┘
                                                  │
                                                  ▼
┌─────────────────────────────────────────────────────────────┐
│                    Agent Orchestrator                        │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐│
│  │ Response  │  │ Product   │  │ Objection │  │ Summary   ││
│  │ Suggester │  │ Retriever │  │ Handler   │  │ Generator ││
│  └───────────┘  └───────────┘  └───────────┘  └───────────┘│
└─────────────────────────────────────────────────────────────┘
        │                 │                            │
        ▼                 ▼                            ▼
┌─────────────┐   ┌─────────────┐              ┌─────────────┐
│ AG-UI       │   │ Product KB  │              │ CRM         │
│ (SSE Stream)│   │ (AI Search) │              │ (Salesforce)│
└─────────────┘   └─────────────┘              └─────────────┘
```

**Key Discussion Points:**
* **Latency requirements** — <500ms for suggestions, streaming via AG-UI protocol
* **Speech-to-text** — Azure Speech real-time transcription, speaker diarization
* **Context window management** — Rolling window of recent transcript, summarization of older parts
* **Product retrieval** — Hybrid search, triggered by detected topics/keywords
* **Privacy** — Call recording consent, PII redaction, data retention policies
* **UI integration** — AG-UI for streaming state updates, CopilotKit for React frontend

---

### Scenario 5: Cost-Optimized High-Volume Classification

**Problem:** "Classify 1 million customer support tickets per day into 50 categories. Budget is $500/day. Design for accuracy and cost efficiency."

**Key Discussion Points:**
* **Model tiering** — Phi-3/Mistral for 80% of tickets, GPT-4o-mini for uncertain, GPT-4o for edge cases
* **Batch API** — OpenAI Batch API (50% discount) for non-real-time processing
* **Fine-tuning ROI** — Fine-tune smaller model on labeled data; break-even analysis
* **Caching** — Semantic cache for similar tickets (e.g., "where's my order" variants)
* **Cost math** — 1M tickets × ~100 tokens × $0.15/1M = $15 base; show optimization savings
* **Evaluation** — Confusion matrix, per-category accuracy, drift monitoring

---

### Scenario 6: Secure Multi-Tenant RAG with Access Control

**Problem:** "Build a RAG system for a consulting firm where each client's documents must be isolated. Users should only see answers from documents they have permission to access."

**Key Discussion Points:**
* **Isolation strategy** — Separate index per tenant vs. single index with metadata filtering
* **Access control** — Document-level ACLs in metadata, filter at query time by user permissions
* **Identity integration** — Azure AD/Entra ID groups, token claims for permissions
* **Data residency** — Per-region indexes for compliance, geo-routing
* **Audit logging** — Who queried what documents, response content logging (redacted)
* **Cost allocation** — Per-tenant usage tracking, chargeback via APIM subscription keys

---

### Scenario 7: Agentic Workflow for Financial Report Generation

**Problem:** "Design an agent that generates quarterly financial reports by: querying databases, pulling market data, generating charts, writing analysis, and producing a formatted PDF — with human approval before sending to executives."

**Key Discussion Points:**
* **Agent decomposition** — Data Agent, Analysis Agent, Chart Agent, Writer Agent, Formatter Agent
* **Tool integration** — SQL connector, market data API, matplotlib/chart tool, LaTeX/PDF generator
* **Workflow orchestration** — Microsoft Agent Framework Workflows, checkpoint after each stage
* **Human-in-the-loop** — Review gate before final PDF, edit/regenerate loop
* **Error handling** — Data validation, retry with different data sources, graceful degradation
* **Reproducibility** — Deterministic data pulls, versioned prompts, audit trail

---

### Scenario 8: Red Team and Guardrails Architecture

**Problem:** "The CISO is concerned about prompt injection and data leakage in your customer-facing chatbot. Design a defense-in-depth security architecture."

**Architecture:**
```
┌─────────────┐     ┌──────────────┐     ┌─────────────────┐
│ User Input  │────▶│ Input        │────▶│ PII Detection   │
│             │     │ Sanitizer    │     │ (Presidio)      │
└─────────────┘     └──────────────┘     └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ Prompt Injection│
                                         │ Detector        │
                                         └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ LLM             │
                                         │ (GPT-4o)        │
                                         └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ Output          │
                                         │ Guardrails      │
                                         │ (Content Safety)│
                                         └────────┬────────┘
                                                  │
                                                  ▼
                                         ┌─────────────────┐
                                         │ Audit Logger    │
                                         │ (Immutable)     │
                                         └─────────────────┘
```

**Key Discussion Points:**
* **OWASP LLM Top 10** — Address each: prompt injection, data leakage, excessive agency, etc.
* **Layered defense** — Input validation → prompt hardening → output filtering → audit
* **Prompt injection detection** — Heuristic rules + classifier model, Azure AI Content Safety
* **Indirect injection** — Sanitize retrieved documents, separate system/user context
* **Rate limiting** — Per-user token limits, abuse detection patterns
* **Testing** — PyRIT for automated red teaming, Garak for vulnerability scanning

---

### Scenario 9: LLMOps Pipeline for Continuous Improvement

**Problem:** "How do you set up CI/CD for an LLM application? Include prompt versioning, automated evaluation, and safe deployments."

**Key Discussion Points:**
* **Prompt as code** — Git versioning, PR reviews for prompt changes
* **Eval suite in CI** — Run RAGAS/DeepEval on golden dataset before merge
* **Deployment stages** — Dev → Staging (shadow mode) → Canary (5%) → Production
* **Rollback triggers** — Eval metric drop >5%, error rate spike, latency degradation
* **A/B testing** — Traffic splitting via APIM, statistical significance before promotion
* **Observability** — LangFuse traces, cost tracking, quality dashboards

---

### Scenario 10: Migrating from POC to Production

**Problem:** "You have a working RAG POC (Jupyter notebook). Walk me through the steps to make it production-ready for 10,000 users."

**Checklist to discuss:**
1. **Infrastructure** — Containerize, deploy to ACA/AKS, IaC (Bicep/Terraform)
2. **Security** — Private endpoints, managed identity, Key Vault, RBAC
3. **API Gateway** — APIM for auth, rate limiting, semantic caching
4. **Observability** — OpenTelemetry, Application Insights, LangFuse
5. **Evaluation** — Automated eval pipeline, quality metrics dashboard
6. **Cost controls** — Token budgets, model tiering, alerts
7. **CI/CD** — GitHub Actions, eval gates, blue-green deployment
8. **Documentation** — ADRs, runbooks, on-call procedures
9. **Load testing** — Concurrent user simulation, latency under load
10. **Compliance** — PII handling, audit logs, data retention

---

## GenAI Lifecycle

> The end-to-end journey of a GenAI application — from business problem to production and continuous improvement. Every architect should be able to narrate this lifecycle fluently.

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  1. Define  │──▶│  2. Build   │──▶│  3. Evaluate│──▶│  4. Deploy  │──▶│  5. Monitor │
│  & Scope    │   │  & Iterate  │   │  & Validate │   │  & Release  │   │  & Improve  │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
        │                 │                 │                 │                 │
   Use case          Prompt eng.       Offline eval      Blue-green       Drift detect
   NFRs              RAG / agents      RAGAS / DeepEval  Canary           A/B testing
   Data audit        Fine-tuning       Red teaming       Feature flags    RLHF loop
   Compliance        Guardrails        Human review      Rollback plan    Cost tracking
```

### Phase 1 — Define & Scope
* **Use case validation** — is GenAI the right tool? Compare vs. traditional ML, rules engine, search
* **NFR definition** — latency, throughput, cost/query, accuracy targets, SLA
* **Data audit** — available data sources, PII inventory, data residency requirements
* **Compliance check** — GDPR, HIPAA, EU AI Act risk classification, internal AI governance policies
* **Build vs. buy** — API-based (Azure OpenAI) vs. open-source self-hosted vs. fine-tuned
* **Success metrics** — define KPIs upfront: RAGAS faithfulness >0.85, latency <2s, cost <$0.01/query

### Phase 2 — Build & Iterate
* **Prompt engineering** — system prompt design, few-shot examples, output format constraints
* **RAG pipeline** — chunking strategy, embedding model selection, vector DB choice, hybrid search
* **Agent design** — tool selection, orchestration framework, memory strategy, HITL gates
* **Guardrails** — input validation, output filtering, PII redaction, content safety
* **Rapid prototyping** — LangFlow / Jupyter → structured codebase; iterate in days not months
* **Version control** — prompts, configs, and eval datasets in Git from day one

### Phase 3 — Evaluate & Validate
* **Offline evaluation** — RAGAS, DeepEval, LLM-as-judge on golden dataset before any deployment
* **Red teaming** — prompt injection, jailbreak, data leakage tests; PyRIT for automation
* **Human evaluation** — domain expert review of sampled outputs; define rubric
* **Load testing** — simulate concurrent users, measure P95 latency under load
* **Security review** — OWASP LLM Top 10 checklist sign-off
* **Eval gate** — automated CI check; block promotion if metrics regress >5%

### Phase 4 — Deploy & Release
* **Environment promotion** — Dev → Staging (shadow mode) → Canary (5-10%) → Production
* **Feature flags** — decouple deploy from release; toggle new model/prompt per user segment
* **Blue-green deployment** — zero-downtime swap; instant rollback capability
* **Rollback triggers** — automated: eval metric drop, error rate spike, latency P95 breach
* **Documentation** — ADR for model/prompt choices, runbook for on-call, user-facing AI disclosure
* **Stakeholder sign-off** — product, legal, security, compliance approval before GA

### Phase 5 — Monitor & Improve
* **Production observability** — LangFuse traces, token cost, latency, error rate dashboards
* **Drift detection** — input distribution shift, output quality degradation, prompt regression alerts
* **Online evaluation** — implicit signals (follow-up questions), explicit feedback (thumbs up/down)
* **A/B testing** — traffic split for prompt variants; statistical significance before promotion
* **RLHF data collection** — store preference pairs from corrections for future fine-tuning
* **Continuous improvement loop** — monthly prompt review, quarterly model upgrade assessment
* **Cost governance** — per-query cost trending, budget alerts, model tiering adjustments

### GenAI Lifecycle vs. Traditional Software

| Aspect | Traditional Software | GenAI Application |
|--------|---------------------|-------------------|
| **Testing** | Unit/integration tests | Eval suites + human review |
| **Versioning** | Code + DB schema | Code + prompts + model version + embeddings |
| **Deployment risk** | Regression bugs | Quality regression, hallucination, bias |
| **Monitoring** | CPU/memory/errors | Token cost, faithfulness, latency, drift |
| **Rollback trigger** | Error rate spike | Eval metric drop OR cost spike |
| **Feedback loop** | Bug reports | User ratings + RLHF preference data |

---

## LLM Fundamentals

### Architecture Deep Dive
* **Transformer architecture** — encoder-decoder vs. decoder-only (GPT style) vs. encoder-only (BERT style)
* **Attention mechanism** — self-attention, multi-head attention, scaled dot-product attention formula: `Attention(Q,K,V) = softmax(QK^T/√d_k)V`
* **Positional encoding** — sinusoidal (original), RoPE (Rotary Position Embedding used in Llama/GPT-4), ALiBi
* **Tokenization** — BPE (Byte Pair Encoding), WordPiece, SentencePiece; token vs. word vs. character
* **Context window** — how KV cache grows with context length, quadratic attention complexity O(n²)
* **Mixture of Experts (MoE)** — sparse activation, routing mechanism, why Mixtral/GPT-4 use it for efficiency

### Model Selection (Architect Perspective)
* **Model families** — GPT-4o, Claude 3.x, Gemini 1.5 Pro, Llama 3, Mistral, Phi-3/4 — know strengths/weaknesses
* **Open source vs. closed source** — data privacy, customization, cost, latency tradeoffs for enterprise decisions
* **Embedding models** — `text-embedding-3-large`, `text-embedding-ada-002`, cosine similarity, dimensions (1536 vs 3072)
* **When to use which model**:
  - GPT-4o: Complex reasoning, multimodal, highest quality (but expensive)
  - GPT-4o-mini: High-volume, cost-sensitive, good-enough quality
  - Claude 3.5: Long context (200K), strong reasoning, good for docs
  - Llama 3/Mistral: Self-hosted, data privacy, cost optimization
  - Phi-3/4: Edge deployment, mobile, resource-constrained

### Inference Parameters
* **Temperature** — 0.0 for deterministic, 0.7-1.0 for creative, affects sampling distribution
* **Top-p (nucleus sampling)** — cumulative probability threshold, typically 0.9-0.95
* **Top-k** — limit vocabulary to top k tokens, use with top-p
* **Repetition penalty** — penalize repeated tokens, useful for generation
* **Stop sequences** — early termination patterns, cost savings

### Training Paradigms
* **Pre-training vs. fine-tuning vs. in-context learning** — cost, data requirements, when to use each
* **Hallucination** — causes (training distribution gaps, confidence miscalibration), mitigation (RAG, grounding, CoT)
* **Emergent abilities** — capabilities that appear at scale (chain-of-thought, in-context learning)

### Benchmarks and Evaluation
* **Model benchmarks** — MMLU (knowledge), HumanEval (code), MT-Bench (chat), HellaSwag (reasoning), TruthfulQA (honesty)
* **Token limits and pricing** — input vs. output token pricing, context window tradeoffs vs. cost

---

## RAG in Depth (Enterprise Solutions)

### Core Pipeline

* Indexing — document loading, chunking, embedding, storing in vector DB
* Retrieval — similarity search (cosine, dot product), top-k, threshold filtering
* Generation — prompt assembly with context, answer grounding

### Advanced RAG Techniques

* HyDE (Hypothetical Document Embeddings) — generate hypothetical answer, embed it, search with that
* Multi-query retrieval — LLM generates multiple query variants, merge results
* Query expansion and decomposition
* Hybrid search — dense (vector) + sparse (BM25/TF-IDF) with RRF (Reciprocal Rank Fusion)
* Re-ranking — cross-encoder models, Cohere Rerank, `FlashRank`
* Corrective RAG (CRAG) — evaluate retrieved docs, web search if insufficient
* Self-RAG — model decides when to retrieve, generates reflection tokens
* Agentic RAG — multi-hop retrieval, agent decides which knowledge base to query
* GraphRAG (Microsoft) — knowledge graph extraction, community summaries, local+global search

### Chunking Strategies

* Fixed-size with overlap
* Recursive character text splitting
* Semantic chunking (embedding-based boundary detection)
* Hierarchical — parent-child chunks (retrieve child, pass parent for context)

### Enterprise Concerns

* Multi-tenancy — per-tenant index isolation, metadata filtering
* Document-level access control — filtering results by user permissions
* Metadata filtering — date range, document type, source system filters
* Vector DBs — Azure AI Search, Pinecone, Weaviate, pgvector, Chroma, Qdrant
* Integrated vectorization in Azure AI Search — automatic chunking + embedding at ingest
* Evaluation — RAGAS faithfulness and context recall as KPIs

---

## Agent Orchestration

### Core Patterns

* Single-agent vs. multi-agent architecture — when to split responsibilities
* Tool use and function calling — OpenAI function calling, Anthropic tool use, schema design
* ReAct pattern — interleaving reasoning (Thought) and acting (Action/Observation)
* Plan-and-execute pattern — planner agent generates steps, executor runs them
* Reflection and self-critique agents — agent evaluates its own output and retries
* Human-in-the-loop — approval gates, clarification requests
* Error recovery — retry with different tools, fallback agents, graceful degradation
* Memory and state handoff between agents

### Frameworks — Side by Side

|Feature|LangGraph|Microsoft Agent Framework|CrewAI|
|-|-|-|-|
|Paradigm|Graph state machine|Agents + graph-based Workflows|Role-based crew|
|Checkpointing|Built-in (`MemorySaver`)|Built-in (superstep checkpoints)|Limited|
|Human-in-loop|Native interrupt/resume|Native (`RequestInfoExecutor` / `ctx.request_info()`)|Limited|
|.NET support|No|Yes (C# + Python)|No|
|Predecessor of|—|AutoGen + Semantic Kernel (unified successor)|—|
|Best for|Complex cyclic agentic flows|Enterprise .NET/Azure production apps|Quick role-based prototypes|

### Microsoft Agent Framework

> **What it is:** The direct successor to both **AutoGen** and **Semantic Kernel**, built by the same Microsoft teams. Combines AutoGen's simple agent abstractions with Semantic Kernel's enterprise features (session state, type safety, middleware, telemetry) and adds graph-based Workflows. As of 2026 this is Microsoft's recommended framework for building production AI agents.
> **Docs:** https://learn.microsoft.com/en-us/agent-framework/overview/

**Core Architecture — Two capability pillars:**

* **Agents** — Individual units that use LLMs to process inputs, call tools/MCP servers, and generate responses
* **Workflows** — Graph-based orchestration connecting agents and functions for multi-step tasks with type-safe routing

**Agent Types (`AIAgent` base class for all):**

* `ChatClientAgent` — wraps any `IChatClient` implementation; supports function calling, multi-turn, structured outputs, MCP tools
* **Microsoft Foundry Agent** — uses Azure AI Foundry Agent Service as backend (stateful, code execution support)
* **Azure OpenAI / OpenAI / Anthropic / Ollama agents** — provider-specific helpers via `.AsAIAgent()` extension
* **Custom agents** — subclass `AIAgent` for full control over agent behavior
* **A2A Proxy agent** — connects to remote agents via Google's A2A protocol

**Key building blocks:**

* **Agent Session** — session-based state management for multi-turn conversations
* **Middleware** — intercept agent actions for logging, safety, telemetry (`IFunctionInvocationFilter` equivalent)
* **Context providers** — agent memory and context injection
* **MCP clients** — built-in tool integration via Model Context Protocol

**Supported providers:** Microsoft Foundry, Azure OpenAI, OpenAI, Anthropic (via Foundry or direct), Ollama, any `IChatClient`

**NuGet / pip:**

* .NET: `dotnet add package Microsoft.Agents.AI.Foundry --prerelease`
* Python: `pip install microsoft-agents-ai`

**Workflows — two APIs:**

* **Functional API** (Python, `@workflow` / `@step` decorators) — write workflows as plain `async` functions; use `asyncio.gather` for parallelism; best starting point
* **Workflow Builder API** (`WorkflowBuilder`, `executors`, `edges`) — directed graph with type-validated message routing; superstep-based parallel execution; best for fixed topologies

**Workflow core concepts:**

* **Executors** — processing units (AI agents or custom logic); receive input messages, produce output messages
* **Edges** — connections between executors with conditional routing based on message content
* **Checkpointing** — save/restore workflow state at superstep boundaries for long-running processes
* **HITL** — `RequestInfoExecutor` (.NET) / `ctx.request_info()` (Python) to pause and collect human input
* **Multi-agent patterns** — sequential, concurrent, hand-off, MagenticOne built-in

**Integrations:** A2A protocol, AG-UI, Azure Functions, Microsoft 365

**Migration note:** If you know AutoGen or Semantic Kernel — Agent Framework supersedes both. See [Migration from SK](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/) and [Migration from AutoGen](https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/)

### Inference and Serving

* Inference parameters — temperature, top-p, top-k, repetition penalty, stop sequences
* Model serving / inference harness — vLLM, Ollama, Text Generation Inference (TGI), ONNX Runtime
* Streaming responses — SSE, chunked transfer, token-by-token display

---

## AI Memory

* Short-term / working memory — in-context window, conversation history truncation strategies
* Long-term memory — external vector store, recalled at query time via similarity search
* Episodic memory — timestamped conversation events, retrievable by recency or relevance
* Semantic memory — factual knowledge store (user preferences, domain facts)
* Procedural memory — stored tool usage patterns, agent skill library
* Memory compression — periodic summarization of older history to save tokens
* **Mem0** — AI memory layer library, user/session/agent scoped memory
* **MemGPT / Letta** — OS-inspired memory management for unlimited context
* Redis / vector DB as memory backend
* Memory management strategies — forgetting curve simulation, importance scoring

---

## Prompt Engineering

* Zero-shot, one-shot, few-shot prompting
* Chain-of-Thought (CoT) — "Let's think step by step"
* Self-consistency — sample multiple reasoning paths, majority vote
* Tree of Thoughts (ToT) — exploring multiple reasoning branches
* ReAct prompting — structured Thought/Action/Observation format
* System prompt design — persona, constraints, output format, tone
* Structured output prompting — JSON mode, `response_format` in OpenAI API
* Prompt compression — `LLMLingua`, context distillation for long documents
* Meta-prompting — using an LLM to generate/improve prompts
* Context window management — chunking, prioritizing recent context, system prompt length budget
* Prompt versioning — tracking changes, A/B testing prompt variants

---

## Fine-Tuning

### When to Fine-Tune (Decision Framework)
| Requirement | Solution |
|-------------|----------|
| Task-specific behavior | Prompt engineering first |
| Domain vocabulary | Fine-tune embeddings |
| Consistent output format | Fine-tune or constrained generation |
| Proprietary knowledge | RAG (usually better) |
| Style/tone consistency | Fine-tune |
| Cost reduction at scale | Fine-tune smaller model |

### Techniques
* **LoRA (Low-Rank Adaptation)** — adds trainable rank decomposition matrices, ~1% of parameters
* **QLoRA** — LoRA + 4-bit quantization, fits on consumer GPUs
* **Full fine-tuning** — all parameters, expensive, risk of catastrophic forgetting
* **Instruction tuning** — fine-tune on instruction-response pairs
* **RLHF** — Reinforcement Learning from Human Feedback, alignment
* **DPO (Direct Preference Optimization)** — simpler alternative to RLHF

### Implementation
* **PEFT library** — `LoraConfig`, `get_peft_model`, `PeftModel`
* **Hugging Face `Trainer` API** — standard training loop
* **`SFTTrainer` (TRL)** — supervised fine-tuning trainer
* **Dataset format** — `{"instruction": ..., "input": ..., "output": ...}`

### Hyperparameters
* Learning rate: 1e-4 to 1e-5 for LoRA
* Batch size: maximize GPU memory
* Epochs: 1-3 for instruction tuning
* Gradient accumulation: simulate larger batches

### Evaluation
* Perplexity on held-out set
* Task-specific benchmarks
* Human evaluation for quality

### Distributed Training
* **FSDP** — Fully Sharded Data Parallel
* **DeepSpeed ZeRO** — stages 1/2/3 for memory optimization

### When NOT to Fine-Tune
* Prompt engineering and RAG often sufficient
* Fine-tune only for: consistent style/format, domain vocabulary, cost reduction at scale
* Consider: maintenance burden, model updates break fine-tunes

---

## AI Optimization Topics

* **Quantization** — INT8, INT4, GPTQ, AWQ, GGUF formats for local/edge deployment
* **Knowledge Distillation** — teacher-student model compression
* **Pruning** — removing low-importance weights
* **Speculative Decoding** — draft model proposes tokens, large model verifies (2-5x speedup)
* **Flash Attention** — memory-efficient attention computation (Flash Attention 2/3)
* **Continuous Batching** — dynamic batching for inference throughput (used in vLLM)
* **KV Cache optimization** — PagedAttention (vLLM), prefix caching
* **Prompt Caching** — Anthropic and OpenAI cache repeated system prompt prefixes
* **Context window optimization** — retrieval over extending context, `LLMLingua` compression
* **Model serving frameworks** — vLLM, TGI (Text Generation Inference), Triton Inference Server, Ollama
* **Cost optimization** — model routing (cheap model for simple queries), caching identical responses
* **Latency optimization** — streaming, async calls, parallel tool execution

---

## LangChain / LangGraph / LangFlow / LangFuse

### LangChain

* Chains — `LLMChain`, `SequentialChain`, `LCEL` (LangChain Expression Language)
* Agents — `AgentExecutor`, tool binding, structured output agents
* Tools — custom tool creation, `@tool` decorator, `StructuredTool`
* Memory — `ConversationBufferMemory`, `ConversationSummaryMemory`, `VectorStoreRetrieverMemory`
* Callbacks and streaming — `StreamingStdOutCallbackHandler`, async streaming
* Retrievers — `VectorStoreRetriever`, `MultiQueryRetriever`, `ContextualCompressionRetriever`
* Prompt templates — `ChatPromptTemplate`, `MessagesPlaceholder`, partial templates

### LangGraph

* Graph-based state machine architecture — nodes, edges, conditional edges
* `StateGraph` — defining state schema with `TypedDict`
* Checkpointing and persistence — `MemorySaver`, `SqliteSaver`
* Multi-agent supervisor pattern — routing between specialized agents
* Human-in-the-loop — interrupt and resume patterns
* Parallel node execution, `Send` API for dynamic branching
* LangGraph vs. plain LangChain — LangGraph for cyclic/agentic flows; LangChain for linear pipelines

### LangFlow

* Visual drag-and-drop agent/workflow builder
* Component wiring — chaining LLMs, retrievers, tools visually
* Exporting flows to Python code
* Use case: rapid prototyping for stakeholder demos

### LangFuse

* Tracing LLM calls — spans, generations, scores
* Cost and token usage tracking per trace
* Dataset creation from production traces
* Prompt versioning and management
* Integration with LangChain, LlamaIndex, OpenAI SDK
* Evaluation scoring — LLM-as-judge within LangFuse

---

## MCP Server (Model Context Protocol)

* What is MCP — Anthropic's open standard for connecting AI agents to external tools/data sources
* Architecture — **MCP Hosts** (Claude, Copilot), **MCP Clients** (in-host connector), **MCP Servers** (tool providers)
* Core primitives — **Tools** (executable functions), **Resources** (data/files), **Prompts** (reusable templates)
* Transport layers — `stdio` (local), HTTP + SSE (remote), WebSockets
* Building MCP servers — Python (`mcp` SDK), TypeScript (`@modelcontextprotocol/sdk`)
* Tool schema definition — JSON Schema for input validation, tool descriptions for LLM selection
* Authentication in remote MCP — OAuth 2.0, API key headers
* Deploying MCP servers — Azure App Service, Azure Container Apps
* MCP clients — Claude Desktop, VS Code GitHub Copilot, Cursor, Continue
* MCP vs. alternatives — OpenAI function calling (model-specific), LangChain tools (framework-specific), MCP (universal)
* **A2A (Agent-to-Agent) protocol** — Google's complementary standard for agent-to-agent communication
* Security — input validation on MCP server side, prompt injection via malicious resource content

---

## AG-UI (Agent-User Interface Protocol)

* AG-UI protocol — open standard for streaming agent state to frontend
* Transport — Server-Sent Events (SSE) for real-time streaming from agent to browser
* Event types — `TEXT_MESSAGE_CONTENT`, `TOOL_CALL_START`, `TOOL_CALL_END`, `STATE_DELTA`
* CopilotKit — React library implementing AG-UI, `useCoAgent` hook, `CopilotChat` component
* Tool call visualization — showing "Agent is searching..." in real time
* Human-in-the-loop UI — interrupt agent, collect user input, resume execution
* Streaming text rendering — progressive token display, markdown streaming
* Shared state between agent and frontend — `useCoAgentStateRender`

---

## Azure Cloud Topics

### Azure OpenAI Service
* Model deployments — standard vs. provisioned (PTU), deployment slots, model versioning
* Quotas and limits — TPM/RPM per deployment, regional capacity, quota increase requests
* PTU vs. consumption pricing — PTU for >1M tokens/day steady state; consumption for dev/spiky
* Managed identity access — system-assigned MI, RBAC role `Cognitive Services OpenAI User`
* Content filtering — configurable severity levels, custom blocklists, async filter for batch
* Fine-tuning — GPT-4o fine-tuning in Azure, training data format, deployment of fine-tuned models

### AI Gateway via APIM
* Semantic caching — cache responses for semantically similar prompts, TTL configuration
* Token rate limiting — `azure-openai-token-limit` policy, per-subscription/per-user limits
* Backend load balancing — round-robin, priority-based, circuit breaker across OpenAI endpoints
* Request/response logging — emit to Event Hub/Log Analytics, PII masking in policies
* Cost attribution — `user` field injection, subscription-based chargeback

### APIM (General)
* Policies — inbound/outbound/on-error, policy expressions, named values
* Versioning and revisions — API versioning strategies, non-breaking changes
* Subscriptions and products — developer portal, approval workflows
* Private endpoints — VNET integration for internal APIs

### Azure AI Search
* Vector search — HNSW algorithm, dimensions (1536/3072), distance metrics (cosine, dot product)
* Hybrid search — vector + BM25 keyword, Reciprocal Rank Fusion (RRF)
* Semantic ranking — L2 re-ranker, semantic captions, query rewriting
* Integrated vectorization — Skills pipeline: crack → chunk → embed → index
* Indexers — blob indexer, SQL indexer, change detection, incremental enrichment
* Security — RBAC on indexes, document-level security via filters

### Azure Functions for AI
* Flex Consumption plan — scale-to-zero, per-second billing, always-ready instances
* Durable Functions — orchestrator/activity pattern, fan-out/fan-in, eternal orchestrations
* Bindings — Service Bus trigger, Blob trigger, HTTP trigger with streaming response
* Isolated worker model — .NET 8+, better dependency injection, middleware support

### Azure Container Apps
* KEDA autoscaling — scale on HTTP, Service Bus queue depth, custom metrics
* Dapr integration — service invocation, pub/sub, state management
* Revision management — traffic splitting, blue-green deployments
* Jobs — scheduled and event-driven batch processing

### Messaging and Event-Driven
* Service Bus — queues vs. topics, sessions for ordering, dead-letter queue, poison messages
* Event Grid — event-driven blob triggers, filtering, dead-lettering
* Event Hubs — high-throughput streaming, Kafka compatibility, capture to storage

### Security and Identity
* Managed Identity — system-assigned vs. user-assigned, RBAC assignments, usage in AI pipelines
* Key Vault — secret rotation, references in App Settings, certificate management
* Private endpoints — network isolation for all AI services
* VNET integration — service endpoints vs. private endpoints decision

### Implementing RBAC in Agents and AI Services

**Core principle:** Agents and AI services should never use hardcoded API keys. Use Managed Identity + RBAC to grant least-privilege access to each resource.

**How to achieve RBAC for an Agent (step-by-step):**
1. **Enable Managed Identity** on the hosting resource (Azure Container App, Function App, AKS pod identity)
2. **Assign RBAC roles** to that managed identity on each backing AI service:
   - Azure OpenAI → `Cognitive Services OpenAI User`
   - Azure AI Search → `Search Index Data Reader` (query) or `Search Index Data Contributor` (ingest)
   - Azure Blob Storage → `Storage Blob Data Reader`
   - Key Vault → `Key Vault Secrets User`
   - AI Foundry Project → `Azure AI Developer`
3. **Use `DefaultAzureCredential`** in code — automatically picks up managed identity in Azure, developer credentials locally
4. **No secrets in code or config** — managed identity token is fetched transparently from IMDS endpoint

**RBAC patterns for common AI architectures:**

* **RAG query agent** — managed identity needs: `Cognitive Services OpenAI User` + `Search Index Data Reader`
* **RAG ingestion pipeline** — needs: `Cognitive Services OpenAI User` + `Search Index Data Contributor` + `Storage Blob Data Reader`
* **Multi-agent system** — each agent gets its own managed identity with only the roles it needs (least privilege per agent)
* **Foundry-hosted agent** — assign `Azure AI Developer` role on the Foundry Project; agent inherits project connections
* **Agent calling external APIs via MCP** — MCP server authenticates with its own managed identity; agent calls MCP server via OAuth/API key scoped to that server only

**RBAC for user-facing AI (end-user access control):**
* **Entra ID groups → AI features** — map Entra groups to app roles (`AI.Basic`, `AI.PowerUser`); check claims in middleware
* **Document-level access control in RAG** — store user permissions as metadata on documents; inject `$filter` at query time based on user token claims
* **Multi-tenant isolation** — separate Foundry Projects per tenant, or use metadata filtering + tenant ID from JWT claim
* **API key per team via APIM** — use APIM subscriptions for team-level rate limits and cost attribution; backend still uses managed identity to OpenAI

---

## Azure AI Foundry

* **What is it** — Microsoft's unified platform for building, evaluating, deploying, and governing enterprise AI; successor to Azure ML Studio and Azure AI Studio
* **Foundry Hub** — top-level resource; manages shared compute, connections, and networking across multiple projects
* **Foundry Project** — isolated workspace per team/product with own model deployments, datasets, evals, and prompt flows
* **Connections** — link Azure OpenAI, AI Search, Storage, Key Vault to projects without exposing secrets; use managed identity
* **Model catalog** — deploy GPT-4o, Claude, Llama 3, Mistral, Phi-3/4, DALL-E, Whisper and 1000+ OSS models
* **Deployment types** — standard (pay-per-token), PTU (provisioned throughput), Serverless API/MaaS (non-OpenAI models), managed compute (self-hosted OSS)
* **Azure AI Agents** — hosted stateful agent service with built-in thread management, file search (RAG), code interpreter, Bing grounding, MCP tools
* **Prompt Flow** — DAG-based and flex (code-first) flows; batch runs against datasets; variants for A/B testing; CI/CD export
* **Evaluation framework** — built-in evaluators: `groundedness`, `relevance`, `fluency`, `coherence`, `similarity`, safety (violence/hate/sexual/self-harm); custom LLM-as-judge evaluators
* **Azure AI Content Safety** — harm category detection (severity 0-7), groundedness detection, jailbreak/prompt shields, custom blocklists
* **Fine-tuning** — GPT-4o, GPT-4o-mini, Phi-3/4; JSONL format; upload → train → evaluate → deploy as standard endpoint
* **AI Search integration** — create knowledge indexes directly in Foundry; integrated vectorization pipeline
* **Observability** — OpenTelemetry traces for agent runs/tool calls/LLM invocations; Application Insights integration; token cost tracking
* **Governance** — RBAC roles (`Azure AI Developer`, `Cognitive Services OpenAI User`), private endpoints, customer-managed keys, Responsible AI dashboard
* **SDK** — `azure-ai-projects` (Python), `azure-ai-inference` for model calls; `azure-ai-contentsafety` for content safety

---

## Document Intelligence & OCR

### Service Selection (Architect Perspective)
| Scenario | Recommended Service | Why |
|----------|-------------------|-----|
| Standard forms (invoice, receipt) | Azure Document Intelligence (prebuilt) | High accuracy, no training |
| Custom forms | Azure Document Intelligence (custom) | Train on your templates |
| Complex layouts, handwriting | GPT-4o Vision | Handles edge cases |
| High volume, cost-sensitive | Tesseract + post-processing | Free, batch-friendly |
| Multi-cloud | AWS Textract / Google Document AI | Vendor requirements |

### Azure Document Intelligence
* **Prebuilt models** — invoice, receipt, ID, W2, health insurance card
* **Layout model** — tables, paragraphs, selection marks, barcodes
* **Custom models** — train on your document types, composition models
* **Read model** — OCR-only, 2 billion character limit

### Vision LLMs for Documents
* **GPT-4o Vision** — best for complex layouts, handwriting, non-standard formats
* **Phi-3 Vision** — cost-effective for simpler documents, edge deployment
* **LLaVA** — open-source alternative, self-hosted

### PDF Processing Pipeline
* **Cracking** — `PyMuPDF (fitz)`, `pdfplumber`, `pdfminer.six`
* **Image extraction** — embedded images, scanned pages
* **Table extraction** — `camelot`, `tabula-py`, vision model for complex tables
* **Layout detection** — LayoutLMv3, DocLayNet for document understanding

### Enterprise Pipeline Design
```
Document Upload → Format Detection → OCR/Extraction → Validation → Chunking → Embedding → Index
                        │                                │
                        ▼                                ▼
                 [Layout Analysis]              [Confidence Score]
                        │                                │
                        ▼                                ▼
                 [Table/Figure                    [Human Review
                  Extraction]                    for Low Confidence]
```

### Quality Assurance
* **Confidence thresholds** — route low-confidence extractions to human review
* **Validation rules** — date formats, amount ranges, required fields
* **Ground truth datasets** — measure accuracy per document type
* **Active learning** — use corrections to improve custom models

---

## GenAIOps / LLMOps / MLOps

> **Hierarchy:** MLOps → LLMOps → GenAIOps. Each layer adds new concerns specific to the AI type.

| Discipline | Focus | Key Additions Over Previous |
|------------|-------|-----------------------------|
| **MLOps** | Traditional ML models | Training pipelines, feature stores, model registry |
| **LLMOps** | Large language models | Prompt versioning, token cost tracking, eval pipelines |
| **GenAIOps** | Full GenAI apps (RAG, agents) | Agent state, guardrails, multi-component tracing, HITL ops |

### GenAIOps — What's New
* **Agent lifecycle management** — deploy, version, monitor, and retire individual agents in a multi-agent system
* **Prompt operations** — prompt registry (LangFuse/PromptLayer), A/B test prompt variants in production, rollback bad prompts
* **Guardrail pipeline ops** — monitor bypass rates, update blocklists, retrain content classifiers
* **Multi-component tracing** — end-to-end trace across retriever → reranker → LLM → agent → tool calls
* **Eval-driven deployment** — automated eval gate blocks promotion; RAGAS faithfulness as deployment criterion
* **RAG index ops** — index versioning, incremental re-indexing on document updates, embedding drift detection
* **HITL workflow ops** — SLA monitoring on approval queues, escalation paths, audit trail for human decisions
* **Token cost ops** — per-agent, per-feature cost attribution; anomaly alerts; model swap recommendations
* **Context window ops** — monitor context fill rate, detect over-truncation, optimize chunking dynamically

### LLMOps
* **What is LLMOps** — operationalizing LLM-based applications; extends MLOps with prompt, eval, and guardrail pipelines
* **Prompt versioning** — treating prompts as code; version in Git, track in LangFuse / PromptLayer
* **CI/CD for AI** — GitHub Actions / Azure DevOps pipelines that run eval suites on every PR before deployment
* **Model drift detection** — statistical tests (PSI, KS test) on input distributions; output quality degradation alerts
* **Canary and blue-green deployments** — gradual traffic shifting for new model/prompt versions
* **Rollback strategy** — automatic rollback trigger when eval metrics drop below threshold
* **Shadow mode deployment** — run new model in parallel, compare outputs without serving to users
* **A/B test framework** — traffic splitting, statistical significance testing for model comparison
* **Environment parity** — dev / staging / prod with identical model versions and prompt configs

### MLOps (Foundations)
* **Experiment tracking** — MLflow (`mlflow.log_params`, `mlflow.log_metrics`), Azure ML experiments, W&B
* **Model registry** — versioning models with metadata, staging → production promotion workflow
* **Data versioning** — DVC (Data Version Control) for datasets and embeddings
* **Feature stores** — Feast, Azure ML Feature Store for consistent feature serving train vs. inference
* **Automated retraining triggers** — drift threshold breach, scheduled retraining, new labeled data available
* **Model lineage** — tracking training data → model → deployment → predictions end-to-end (Azure ML lineage graph)
* **Containerized model serving** — Docker image per model version, immutable deployments

### Tooling Comparison

| Tool | Primary Use | Best For |
|------|------------|----------|
| **LangFuse** | LLM tracing, prompt versioning, evals | LLMOps / GenAIOps |
| **LangSmith** | LangChain-native tracing and evals | LangChain apps |
| **MLflow** | Experiment tracking, model registry | MLOps / fine-tuning |
| **Azure AI Foundry** | End-to-end eval, fine-tune, deploy | Azure GenAI stack |
| **Weights & Biases** | Experiment tracking, sweeps | Training / fine-tuning |
| **Arize Phoenix** | LLM observability, dataset curation | Production monitoring |

---

## Observability and Tracing

* OpenTelemetry — traces, metrics, logs; instrumentation for LLM apps
* LangFuse / LangSmith / Arize Phoenix — LLM-specific observability platforms
* Azure Monitor + Application Insights — custom events, KQL queries, dashboards for AI apps
* What to log — prompt, response, model version, token count, latency, cost, user ID, session ID
* Distributed tracing — correlation IDs across microservices and agent hops
* Alerting — latency spikes, error rate thresholds, cost budget alerts
* Cost monitoring — per-user, per-feature token consumption tracking
* Evaluation metrics dashboards — faithfulness, relevancy trends over time
* Prompt regression detection — alert when output quality degrades after prompt change

---

## Evaluation (Online and Offline)

### Offline Evaluation

* RAGAS — `faithfulness`, `answer_relevancy`, `context_precision`, `context_recall` metrics
* DeepEval — test cases, `GEval` for custom criteria, regression test suites
* TruLens — TruChain, TruLlama, RAG triad evaluation
* LLM-as-judge pattern — GPT-4o evaluating response quality with rubric
* Benchmark datasets — MMLU, HumanEval, MT-Bench, domain-specific curated datasets
* Prompt regression testing — CI/CD integration to catch quality drops on every PR

### Online Evaluation

* Implicit signals — session length, follow-up questions indicating confusion
* Explicit signals — thumbs up/down, star ratings, correction inputs
* Self-reflection — model critiques its own answer against a rubric (groundedness, completeness, safety) before finalizing
* External reflection — secondary judge model or human reviewer scores output and sends revise/approve feedback
* A/B testing — routing % of traffic to prompt variant B, measuring task completion
* Production trace sampling — periodic human review of sampled conversations
* RLHF data collection pipeline — storing preference pairs for future fine-tuning
* Azure AI Foundry online evaluation — continuous evaluation jobs on live traffic

---

## Cost Governance for LLMs

* **Token budgeting** — setting max token limits per request, per user, per feature to cap spend
* **Model tiering strategy** — route simple queries to `gpt-4o-mini` / `Phi-3`, complex queries to `GPT-4o`; saves 60-80% cost
* **Prompt caching** — reuse cached KV state for repeated system prompt prefix (Anthropic/OpenAI feature)
* **Semantic caching** — cache responses for semantically similar queries (Azure APIM AI Gateway, GPTCache)
* **PTU vs. consumption pricing** — Provisioned Throughput Units for predictable high-volume; pay-per-token for spiky workloads
* **Showback / chargeback** — tag Azure OpenAI requests by team/product using `user` field or APIM subscription keys; report via Cost Management
* **Token usage dashboards** — LangFuse cost tracking, Azure Monitor custom metrics, APIM analytics for per-API token consumption
* **Batch API** — use OpenAI Batch API (50% cheaper) for non-real-time workloads (nightly reports, bulk classification)
* **Embedding cost reduction** — cache embeddings in vector DB; never re-embed unchanged documents
* **Context window discipline** — trim conversation history, compress with summarization; every unnecessary token = cost
* **FinOps for AI** — tagging strategy, budget alerts in Azure Cost Management, anomaly detection on spend spikes
* **Cost per query / cost per user KPIs** — define and track in dashboards; set budget alerts at 80% threshold
* **Open-source fallback** — deploy Phi-3 / Mistral on Azure Container Apps for cost-sensitive high-volume tasks
* **Rate limit as cost control** — APIM token-rate-limit policy prevents runaway spend from a single consumer

---

## AI Security Topics

* **OWASP Top 10 for LLMs** — LLM01 Prompt Injection, LLM06 Sensitive Info Disclosure, LLM09 Misinformation (memorize all 10)
* **Prompt injection** — direct and indirect; defense via input validation, privilege separation
* **Data poisoning** — corrupting training/fine-tuning data or RAG knowledge base
* **PII detection and redaction** — `presidio` (Microsoft), regex-based PII scrubbing before logging
* **Model inversion and membership inference** — awareness of model privacy attacks
* **Jailbreaking defenses** — NeMo Guardrails, Llama Guard, output classifiers
* **API security for AI endpoints** — auth (managed identity, API keys in Key Vault), rate limiting, CORS
* **Supply chain security** — verifying model sources, Hugging Face model scanning
* **Secrets management** — never hardcode API keys; use Azure Key Vault + managed identity
* **Zero-trust for AI systems** — least-privilege access to vector DBs, document stores, APIs
* **Audit logging** — immutable logs of all AI interactions for compliance
* **Network isolation** — private endpoints for Azure OpenAI, VNET integration

---

## Red Teaming

* Prompt injection — direct (user input hijacks system prompt) and indirect (injected via retrieved documents)
* Jailbreaking techniques — DAN, role-play attacks, encoded inputs, multi-turn manipulation
* Data exfiltration attempts — tricking the model to reveal system prompt or retrieved data
* Adversarial inputs — typos, Unicode tricks, token boundary attacks
* Multi-turn attack scenarios — building context across turns to bypass guardrails
* **PyRIT** (Microsoft Python Risk Identification Toolkit) — automated red teaming for generative AI
* **Garak** — open-source LLM vulnerability scanner
* OWASP Top 10 for LLMs — LLM01 Prompt Injection through LLM10 Model Theft
* Defense mechanisms — input sanitization, output validation, NeMo Guardrails, Llama Guard
* Content safety testing — testing across all harm categories before production launch
* Red team documentation — findings report, risk severity classification, remediation tracking

---

## AI Ethics and Responsible AI

* Microsoft's 6 RAI principles — Fairness, Reliability & Safety, Privacy & Security, Inclusiveness, Transparency, Accountability
* Bias detection — disparate impact analysis, `Fairlearn` toolkit
* Explainability — SHAP values, LIME, `InterpretML`
* Content moderation — Azure AI Content Safety, NeMo Guardrails, Llama Guard
* GDPR and data privacy — right to deletion, data minimization, PII handling in AI pipelines
* EU AI Act — high-risk AI system classification, conformity assessments awareness
* Model governance — model cards, datasheets, versioning, audit trails
* Transparency — disclosing AI-generated content to users
* Accountability — human oversight requirements, escalation paths

---

## EU AI Act / Governance

> **Why this matters:** The EU AI Act (effective August 2024, full enforcement by August 2026) is the world's first comprehensive AI regulation. Architects must understand risk classification and compliance requirements.

### Risk Classification

| Risk Level | Examples | Requirements |
|------------|----------|-------------|
| **Unacceptable** | Social scoring, real-time biometric ID in public | Prohibited |
| **High-Risk** | HR screening, credit scoring, medical diagnosis, legal AI | Conformity assessment, CE marking, human oversight |
| **Limited Risk** | Chatbots, emotion recognition | Transparency obligations (disclose AI) |
| **Minimal Risk** | Spam filters, AI in games | No specific requirements |

### High-Risk AI System Requirements
* **Risk management system** — identify, analyze, evaluate, mitigate risks throughout lifecycle
* **Data governance** — training data must be relevant, representative, error-free; bias examination
* **Technical documentation** — detailed system description, design choices, performance metrics
* **Record-keeping** — automatic logging of events for traceability
* **Transparency** — clear instructions for deployers on capabilities and limitations
* **Human oversight** — designed to allow effective human intervention
* **Accuracy, robustness, cybersecurity** — appropriate levels for intended purpose

### General-Purpose AI (GPAI) Rules
* **All GPAI models** — technical documentation, training data summary, copyright compliance
* **GPAI with systemic risk** (>10^25 FLOPs training) — model evaluation, adversarial testing, incident reporting, cybersecurity measures
* **Open-source exception** — reduced obligations if model weights publicly released (except systemic risk models)

### Compliance Timeline
* **Feb 2025** — Prohibited practices banned
* **Aug 2025** — GPAI rules apply, governance structure active
* **Aug 2026** — Full enforcement including high-risk AI systems

### Architect Responsibilities
* **Classification** — determine if your AI system is high-risk (most enterprise RAG/agents are limited/minimal risk)
* **Documentation** — maintain technical docs, risk assessments, data governance records
* **Audit trail** — logging for all AI decisions in high-risk contexts
* **Human-in-the-loop** — design override capabilities for high-risk applications
* **Transparency** — clearly disclose AI-generated content to users
* **Vendor due diligence** — ensure AI service providers (Azure OpenAI, etc.) provide compliance documentation

### Internal AI Governance (Beyond EU AI Act)
* **AI Ethics Board** — cross-functional review for new AI use cases
* **Model registry** — catalog all deployed models with risk classification, owners, data sources
* **Use case approval workflow** — gate for high-risk applications before production
* **Incident response plan** — procedure for AI-related failures, bias incidents, security breaches
* **Training requirements** — AI literacy for developers, product managers, executives
* **Third-party AI policy** — rules for using external AI APIs, model marketplaces

---

## DevOps for AI Applications

### Containerization
* Docker — multi-stage builds, `.dockerignore`, slim base images, layer caching
* AI-specific images — CUDA base images for GPU, model weights baked in vs. downloaded at startup
* Image size optimization — separate runtime from build deps, model quantization for smaller images

### Kubernetes for AI
* Pods, Deployments, Services, ConfigMaps, Secrets, Ingress, Namespaces
* GPU scheduling — `nvidia.com/gpu` resource requests, node selectors, taints/tolerations
* Model serving on K8s — KServe, Triton Inference Server, vLLM on AKS

### Autoscaling for AI Workloads
* HPA (CPU/memory) — limited usefulness for LLM inference (GPU-bound)
* KEDA (event-driven) — scale on Service Bus queue depth, HTTP request rate, custom metrics
* VPA — vertical scaling for memory-intensive embedding jobs
* Predictive scaling — based on historical patterns for batch jobs

### CI/CD for AI
* GitHub Actions — CI/CD pipelines for AI applications, model deployment workflows
* Azure DevOps — pipelines, release gates, environment approvals
* Eval gates — run evaluation suite in PR, block merge if metrics degrade
* Model deployment pipelines — separate model artifacts from code, versioned deployments

### Container Registry
* ACR — image tagging strategy, vulnerability scanning, geo-replication
* Model artifact storage — ACR for model images, Azure ML registry for model files

### Infrastructure as Code
* Bicep — modules for AI resources (OpenAI, AI Search, APIM), what-if validation
* Terraform — provider for Azure AI services, state management, drift detection
* azd (Azure Developer CLI) — template-based deployment, `azd up` workflow

### GitOps and Progressive Delivery
* ArgoCD concept awareness — declarative GitOps for K8s
* Flux — GitOps for AKS
* Feature flags for AI — gradual rollout of new models/prompts

---

## Database Technologies

### SQL Server / Azure SQL

* **T-SQL essentials** — CTEs, window functions (`ROW_NUMBER`, `RANK`, `LAG/LEAD`), temp tables vs. table variables
* **Indexes** — clustered vs. non-clustered, covering indexes, columnstore indexes for analytics
* **Query optimization** — execution plans, index hints, `SET STATISTICS IO/TIME`, query store
* **Transactions** — ACID properties, isolation levels (`READ COMMITTED`, `SNAPSHOT`, `SERIALIZABLE`), deadlock detection
* **JSON support** — `FOR JSON`, `OPENJSON`, storing and querying JSON documents natively
* **Azure SQL features** — serverless tier, Hyperscale, elastic pools, Always On availability groups
* **EF Core integration** — `DbContext`, migrations, raw SQL with `FromSqlRaw`, compiled queries
* **AI relevance** — storing RAG metadata, structured output storage, audit logs

### Azure Cosmos DB (NoSQL)

* **APIs** — NoSQL (core), MongoDB, Cassandra, Table, Gremlin (graph) — know when to pick each
* **Partition key design** — high cardinality, even distribution, avoid hot partitions; bad key = performance killer
* **Consistency levels** — Strong, Bounded Staleness, Session (default), Consistent Prefix, Eventual
* **Request Units (RUs)** — RU/s provisioning vs. autoscale vs. serverless; how to estimate and optimize
* **Global distribution** — multi-region writes, conflict resolution policies, geo-redundancy
* **Change Feed** — event-driven processing, trigger Azure Functions on document changes
* **AI use cases** — conversation history, agent state, RAG document metadata, session data
* **AI Search integration** — Cosmos DB indexer in Azure AI Search for automatic RAG ingestion

### Vector Databases

> **Core concept:** Store high-dimensional embedding vectors; support ANN (Approximate Nearest Neighbor) search for semantic similarity. Backbone of all RAG systems.

#### pgvector (PostgreSQL extension)
* **Setup** — `CREATE EXTENSION vector;`, `vector(1536)` column type
* **Index types** — `ivfflat` (faster build), `hnsw` (faster query, PostgreSQL 16+)
* **Query operators** — `<->` L2 distance, `<=>` cosine distance
* **Hybrid search** — combine `tsvector` full-text with vector similarity in one query
* **Managed** — Azure Database for PostgreSQL Flexible Server; private endpoints, HA
* **Best for** — <5M vectors, existing Postgres apps, need transactional consistency alongside vectors

#### Pinecone
* **Managed, cloud-native** — serverless and pod-based plans; zero infrastructure
* **Namespaces** — logical separation within an index (multi-tenancy pattern)
* **Metadata filtering** — filter at query time, complex boolean expressions
* **SDK** — `upsert(vectors)`, `query(vector, top_k, filter)` via REST/Python
* **Best for** — pure vector workloads, global scale, no relational data needed

### Redis / Azure Cache for Redis

* **Data structures** — strings, hashes, lists, sets, sorted sets, streams
* **AI use cases** — semantic cache (cache LLM responses by embedding similarity), session state, rate limiting counters
* **Vector search** — `RediSearch` module, HNSW index, cosine/L2 distance, hybrid BM25 + vector
* **Pub/Sub and Streams** — broadcast real-time agent state to frontend, event-driven pipelines
* **Cache-aside pattern** — load on miss, TTL expiry, cache invalidation strategies
* **Azure Managed Redis** — active geo-replication, zone redundancy, Memory Optimized and Compute Optimized tiers
* **Best for** — low-latency ephemeral state, semantic response caching, rate limiting; not for durable long-term storage

### Quick Comparison

| Criteria | Azure AI Search | pgvector | Pinecone | Cosmos DB | Redis |
|----------|----------------|----------|----------|-----------|-------|
| **Primary role** | Vector + full-text | Vector in Postgres | Pure vector | Document store | Cache + vector |
| **Hybrid search** | Native (RRF) | Manual SQL | Limited | No | Yes (RediSearch) |
| **Transactional** | No | Yes | No | Yes | No |
| **Azure native** | Yes | Yes (Azure PG) | No | Yes | Yes |
| **Best for** | Enterprise RAG | Existing PG apps | SaaS vector | Agent state, chat history | Semantic cache, rate limit |

### Database Patterns for AI

* **RAG vectors + metadata** — Azure AI Search (vectors) + Azure SQL or Cosmos DB (structured metadata)
* **Conversation history** — Cosmos DB NoSQL (TTL, change feed, per-user partition key)
* **Agent state** — Cosmos DB (durable, global) or Redis (fast, ephemeral)
* **Semantic response cache** — Redis with RediSearch vector index; saves 20-40% on repeat queries
* **Audit / compliance logs** — Azure SQL with append-only design and row-level security
* **Embedding cache** — pgvector; hash document content as cache key to avoid re-embedding

---

## .NET (For AI Development)

* Design Patterns — Singleton, Factory, Repository, Strategy, Observer, CQRS, Mediator
* OOPs Concepts — abstraction, encapsulation, polymorphism, inheritance with real examples
* SOLID principles — enterprise-level examples for each (e.g., Open/Closed in plugin architectures)
* Microservice-based distributed architecture — service discovery, API gateway, saga pattern, event sourcing
* Multithreading — `Task`, `async/await`, `CancellationToken`, thread safety, `SemaphoreSlim`
* Entity Framework — migrations, lazy vs. eager loading, `IQueryable` vs `IEnumerable`, raw SQL, transactions
* Authentication and Authorization — JWT, OAuth 2.0, OpenID Connect, role-based vs. policy-based, MSAL
* gRPC vs REST vs SignalR — when to use which
* Minimal APIs vs. Controllers in .NET 8+
* Health checks and circuit breaker patterns (`Polly`)
* Middleware essentials — `UseHttpLogging`, exception handling middleware, `UseSerilogRequestLogging`, `UseCors`, `UseRateLimiter`, `UseResponseCaching`, custom middleware ordering
* **Microsoft.Extensions.AI** — unified `IChatClient` abstraction for AI providers
* **Microsoft Agent Framework (.NET)** — `AIAgent`, `ChatClientAgent`, Workflows, MCP integration

### Popular .NET Interview Questions

* How does ASP.NET Core middleware pipeline ordering work, and why does order matter?
* When should you use `UseHttpLogging` vs structured request logging (`UseSerilogRequestLogging`)?
* What is the difference between custom exception middleware and exception filters?
* How do `IQueryable` and `IEnumerable` differ in EF Core, and what are common performance pitfalls?
* When do you choose Minimal APIs over Controllers in enterprise applications?
* How do you implement cancellation correctly with `CancellationToken` in async APIs?
* How would you design resilient outbound HTTP calls with `HttpClientFactory` + `Polly`?
* gRPC vs REST vs SignalR: what tradeoffs drive your decision in production?
* How do role-based and policy-based authorization differ in real systems?
* How do you secure and observe AI endpoints in .NET (middleware, auth, rate limit, telemetry)?

---

## C# Basics (For AI Development)

* C# syntax fundamentals — variables, data types, operators, conditions, loops
* Functions and classes — methods, constructors, access modifiers, namespaces
* OOP in C# — class, interface, abstract class, inheritance, polymorphism
* Collections and LINQ — `List<T>`, `Dictionary<TKey, TValue>`, filtering/projection with LINQ
* Exception handling — `try/catch/finally`, custom exceptions
* Async basics — `Task`, `async/await`, cancellation basics

---

## SQL Basics (For AI Development)

* Core queries — `SELECT`, `WHERE`, `ORDER BY`, `TOP`, aliases
* Joins — `INNER`, `LEFT`, `RIGHT`, `FULL` and when to use each
* Aggregation — `GROUP BY`, `HAVING`, `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`
* Data changes — `INSERT`, `UPDATE`, `DELETE`, `MERGE` basics
* Common SQL patterns — CTE basics, subqueries, pagination
* Performance basics — indexes, execution plan awareness, avoiding `SELECT *`

---

## Python (For AI Development)

* Pydantic — model validation, `BaseModel`, validators, `model_config`, nested models
* FastAPI — routing, dependency injection, background tasks, middleware, async endpoints, OpenAPI docs
* Hugging Face — `pipeline`, `AutoModel`, `Tokenizer`, `datasets`, `transformers` library, Hub usage
* `asyncio` — `async/await`, event loops, `gather`, `TaskGroup`
* Type hints — `Optional`, `Union`, `Literal`, `TypeVar`, `Protocol`
* Decorators, context managers, generators
* OOPs in Python — dunder methods, `dataclass`, `ABC`
* Error handling patterns — custom exceptions, retry logic
* `pytest` basics — fixtures, mocking, parametrize
* Common interview programs — two-sum, sliding window, recursion, string manipulation, list comprehensions

---

---

## Managerial / Architect Interview Prep

### Behavioral Questions (STAR Method)
* **STAR method** — Situation, Task, Action, Result; prepare 5-6 stories covering leadership, conflict, delivery under pressure
* **Failure stories** — be ready to explain a project that didn't go as planned and what you learned
* **Conflict resolution** — technical disagreements, scope creep, resource constraints

### Architecture Communication
* **Architectural Decision Records (ADRs)** — how you document and communicate design decisions
* **Technical roadmap communication** — presenting a 6/12/18 month AI adoption roadmap to non-technical stakeholders
* **C4 Model** — Context, Container, Component, Code diagrams for architecture documentation
* **Whiteboarding** — practice drawing architectures live, explain tradeoffs as you draw

### Strategic Decisions
* **Build vs. buy** — articulating criteria (time-to-market, maintenance cost, vendor lock-in, customization needs)
* **Cost vs. quality tradeoffs** — justifying GPT-4o vs. GPT-4o-mini vs. fine-tuned smaller model decisions
* **Vendor evaluation** — how to evaluate AI vendor/tool choices (OpenAI vs. Azure OpenAI vs. Bedrock)
* **Technology selection** — framework choices, cloud provider decisions, model selection rationale

### Stakeholder Management
* **Stakeholder alignment** — aligning business requirements with technical feasibility
* **Managing expectations** — AI project uncertainty, communicating probabilistic outcomes
* **Executive communication** — translating technical concepts for non-technical leadership
* **Risk communication** — explaining AI risks (hallucination, bias) without creating fear

### Team Leadership
* **Team mentoring** — how you upskill developers on AI/LLM concepts
* **Agile/Scrum for AI projects** — sprint planning with experimentation tasks, managing non-deterministic work
* **Cross-functional collaboration** — working with data scientists, product managers, security teams
* **Technical debt management** — balancing speed vs. quality in AI projects

### Production Journey
* **PoC to Production** — describe how you take an AI prototype to enterprise-grade production
* **Scaling challenges** — moving from demo to 10K users, infrastructure evolution
* **Production incidents** — how you handled AI-related production issues

### Questions to Ask Interviewers
* "What's your current AI/ML maturity level?"
* "What are the biggest challenges your team faces with AI adoption?"
* "How do you handle model governance and responsible AI?"
* "What's the tech stack for your AI infrastructure?"
* "How do you measure success for AI initiatives?"

---

## Quick Reference — Framework Comparison

|Capability|LangChain|LangGraph|Microsoft Agent Framework|CrewAI|
|-|-|-|-|-|
|Language|Python|Python|C# / Python|Python|
|Agent loops|Basic|Graph-based cycles|Agents + graph Workflows|Role-based|
|State/checkpointing|Limited|Native (`MemorySaver`)|Native (superstep checkpoints)|Limited|
|Human-in-loop|Plugin|Native interrupt/resume|Native (`RequestInfoExecutor`)|Limited|
|Azure/MS ecosystem|Partial|Partial|Native (Foundry, Azure OpenAI, M365)|No|
|Predecessor of|—|—|AutoGen + Semantic Kernel (unified)|—|
|Best use case|Pipelines \& RAG|Complex Python agentic flows|Enterprise .NET/Azure production apps|Quick crew prototypes|

> **Note:** AutoGen and Semantic Kernel are now legacy. Microsoft Agent Framework is their unified successor as of 2026.

---

## Interview Cheat Sheet — Key Numbers to Remember

### Token Pricing (as of 2026)
| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|----------------------|------------------------|
| GPT-4o | $2.50 | $10.00 |
| GPT-4o-mini | $0.15 | $0.60 |
| text-embedding-3-large | $0.13 | — |
| Claude 3.5 Sonnet | $3.00 | $15.00 |

### Context Windows
| Model | Context Window |
|-------|---------------|
| GPT-4o | 128K tokens |
| Claude 3.5 | 200K tokens |
| Gemini 1.5 Pro | 1M tokens |
| Llama 3 | 8K-128K (varies) |

### Embedding Dimensions
| Model | Dimensions |
|-------|-----------|
| text-embedding-3-large | 3072 (or 1536 reduced) |
| text-embedding-ada-002 | 1536 |
| Cohere embed-v3 | 1024 |

### Latency Targets
| Scenario | Target |
|----------|--------|
| Chat response TTFT | <500ms |
| RAG end-to-end | <2s |
| Batch processing | <5s per doc |
| Real-time agent | <1s per turn |

### Cost Optimization Impact
| Technique | Typical Savings |
|-----------|----------------|
| Model tiering (mini for simple) | 60-80% |
| Semantic caching | 20-40% |
| Batch API | 50% |
| Prompt compression | 30-50% |

---

## Final Preparation Checklist

### Before the Interview
- [ ] Review all enterprise scenarios — be ready to whiteboard any of them
- [ ] Prepare 5-6 STAR stories covering: leadership, technical decisions, failures, cross-team collaboration
- [ ] Know the tradeoffs for every architectural decision
- [ ] Practice explaining complex concepts simply (elevator pitch for RAG, agents, etc.)
- [ ] Review Azure pricing and limits — interviewers love cost discussions

### During the Interview
- [ ] Ask clarifying questions before designing
- [ ] State assumptions explicitly
- [ ] Start with high-level architecture, then drill down
- [ ] Discuss NFRs proactively (security, cost, scale, observability)
- [ ] Mention evaluation and monitoring early
- [ ] Acknowledge tradeoffs — "We could also do X, but Y is better because..."

### Key Differentiators for Architects
- [ ] Think about enterprise concerns: multi-tenancy, compliance, cost governance
- [ ] Mention observability and evaluation in every design
- [ ] Consider security at every layer
- [ ] Show awareness of responsible AI principles
- [ ] Demonstrate cost consciousness
- [ ] Balance innovation with pragmatism

---

> **Last Updated:** 2026-06-08  
> **Focus:** GenAI / Agentic AI Architect Role  
> **Structure:** Architecture-first, then implementation details
