## Chapter 1: Designing GenAI Applications on Databricks — Practice Questions

**Q1.** A financial services company needs to build a customer support chatbot that answers questions about their 500+ product documentation pages. The answers must cite specific sections of documents and stay strictly within company policy. Response time must be under 2 seconds. Which approach BEST meets these requirements?

A) Implement RAG with vector search on documentation, use DBRX Instruct with citation prompts and guardrails
B) Fine-tune Llama 3.1 70B on all documentation and deploy with Model Serving
C) Use prompt engineering with the full documentation in context window of Llama 3.1 405B
D) Create a multi-agent system with separate agents for each product category, fine-tuned on respective docs

<details>
<summary>Answer</summary>

**Correct: A**

RAG is optimal for knowledge retrieval with citation requirements. Vector search retrieves relevant docs, DBRX Instruct generates responses with citations, guardrails ensure policy compliance. Meets <2s latency requirement and allows easy document updates without retraining.

**Why the others are wrong:**
- Fine-tuning doesn't guarantee citations or strict policy adherence, requires retraining for doc updates, and won't meet latency requirements with 70B model
- 405B model exceeds latency budget significantly and context window limits make including 500+ docs impractical
- Over-engineered solution that increases latency and complexity without addressing core requirements; fine-tuning still doesn't solve citation problem

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An e-commerce platform needs to generate personalized product descriptions based on user browsing history and product attributes. The system must process 10,000 requests per minute with consistent formatting (JSON output with specific fields). Which Foundation Model API configuration on Databricks is most appropriate?

A) Llama 3.1 405B with provisioned throughput and structured output prompting
B) MPT-7B fine-tuned on product data with CPU-based batch inference
C) External API (GPT-4) with retry logic and JSON parsing from free-form text
D) DBRX Instruct with GPU Model Serving, response_format parameter set to JSON schema

<details>
<summary>Answer</summary>

**Correct: D**

DBRX Instruct optimized for instruction following, GPU Model Serving handles 10k req/min throughput, response_format parameter guarantees valid JSON output matching schema without post-processing. Most cost-effective solution meeting all requirements.

**Why the others are wrong:**
- 405B model too expensive for this throughput, provisioned throughput cost unjustified when DBRX meets requirements
- CPU batch inference won't meet real-time throughput needs, MPT-7B lacks instruction-following capabilities needed for consistent formatting
- External API adds latency, egress costs, dependency risk; JSON parsing from text less reliable than native structured output

</details>

---

**Q3.** A healthcare application must analyze patient records and recommend diagnostic tests. The system requires medical domain expertise, must explain reasoning, and needs to call external APIs to check insurance coverage and schedule appointments. What compound AI system design is most appropriate?

A) Single LLM with medical knowledge base in RAG, all logic in prompt engineering
B) Agent with medical-fine-tuned model, tools for insurance API and scheduling API, guardrails preventing diagnosis claims
C) Orchestrator managing separate retriever (medical DB), reranker (relevance scoring), and generator (response creation) without agent capabilities
D) Multi-agent system with diagnostic agent, scheduling agent, and insurance agent, each with separate vector stores

<details>
<summary>Answer</summary>

**Correct: B**

Agent architecture needed for API tool calling and multi-step reasoning. Medical fine-tuning provides domain expertise. Tools enable external actions (insurance check, scheduling). Guardrails critical to prevent model making actual diagnoses (liability risk). Single agent sufficient for coordinated workflow.

**Why the others are wrong:**
- Cannot call external APIs without agent/tool capabilities, prompt engineering alone insufficient for complex API interactions
- Retriever-reranker-generator pipeline is for knowledge retrieval, not action execution; lacks tool-calling capability for scheduling/insurance APIs
- Over-engineered; multiple agents add coordination complexity without clear benefit; single agent with multiple tools simpler and more maintainable

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A legal firm wants to summarize case law documents. They have 50,000 historical case summaries (input document + human-written summary pairs). New cases arrive daily. Summaries must follow strict legal citation format and terminology. Budget is limited. Which approach optimizes cost and accuracy?

A) RAG with case law vector database, prompt engineering with few-shot examples of citation format
B) Prompt engineering only with citation format instructions, using base Llama 3.1 8B model
C) Fine-tune DBRX on the 50,000 examples, deploy with Model Serving for consistent formatting
D) Fine-tune MPT-7B on summaries, use RAG for citation lookup, combine both approaches

<details>
<summary>Answer</summary>

**Correct: C**

50,000 labeled examples is ideal fine-tuning dataset size. Fine-tuning teaches consistent legal format and terminology better than prompting. DBRX balances cost and quality. Legal writing style benefits from training, not just retrieval. One-time training cost amortized over daily usage.

**Why the others are wrong:**
- RAG optimal for retrieval, but task is generation with specific style/format learned from examples, not knowledge lookup; won't achieve citation format consistency
- 8B model lacks legal reasoning capability, prompt engineering alone can't match fine-tuned performance on specialized legal format requirements
- Over-complex; fine-tuning already learns citation format from training data, RAG adds unnecessary retrieval step and latency

</details>

---

**Q5.** A customer service application needs to route user queries to appropriate backend systems: billing API, technical support ticketing, or product information database. The system must decide routing without calling all APIs. Which compound AI component architecture is required?

A) Retriever to fetch relevant documents, reranker to prioritize, generator to create response
B) Agent with classifier tool that determines intent, followed by conditional tool calling based on classification
C) Three parallel agents (billing, support, product), orchestrator selects response with highest confidence
D) Single LLM with routing instructions in system prompt, all three APIs as tools, let model call all and filter

<details>
<summary>Answer</summary>

**Correct: B**

Agent with classifier tool enables intent detection before API calls. Conditional tool calling prevents unnecessary API invocations (cost/latency). Agent orchestrates decision logic: classify intent, then call only relevant API. Standard routing pattern in compound AI systems.

**Why the others are wrong:**
- Retriever-reranker-generator pipeline is for document search/synthesis, not routing/API orchestration; no decision-making capability
- Running three parallel agents wastes compute; all agents process query even when only one needed; orchestrator selection still requires all agents to run
- Calling all APIs defeats efficiency goal; requirement specifically states 'without calling all APIs'; system prompt can't prevent tool calls

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** An analytics platform generates SQL queries from natural language. Users are business analysts, not SQL experts. Queries must be validated against schema before execution to prevent errors. Which tool and guardrail combination is appropriate for the agent?

A) Tool: Schema validator (returns column types/constraints); Guardrail: SQL injection detection and read-only query enforcement
B) Tool: SQL executor; Guardrail: output length limit to prevent long queries
C) Tool: Database connection; Guardrail: prompt-based instruction to 'only generate safe queries'
D) Tool: Query result formatter; Guardrail: timeout after 30 seconds of execution

<details>
<summary>Answer</summary>

**Correct: A**

Schema validator tool enables agent to check query validity against actual schema before execution. SQL injection detection guardrail prevents security exploits. Read-only enforcement prevents data modification by business analysts. Addresses both correctness and security.

**Why the others are wrong:**
- Output length doesn't validate query correctness or safety; long queries aren't inherently problematic; doesn't check schema compatibility
- Prompt-based instructions insufficient for safety; LLMs can't be trusted to follow security rules through prompting alone; lacks technical enforcement
- Timeout is execution guardrail, not validation; doesn't prevent invalid queries from being attempted; formatting tool doesn't validate query correctness

</details>

---

**Q7.** A research platform needs to answer questions from 100TB of scientific papers (30M documents). Queries are complex, multi-hop questions requiring synthesis from multiple papers. Latency budget is 5-10 seconds. Which compound AI architecture best balances accuracy and latency?

A) Fine-tune Llama 3.1 405B on all 30M papers, direct question answering without retrieval
B) Agent with iterative retrieval tool, performs multiple searches, no reranker
C) Vector search retriever (top-100) → Reranker (top-10) → DBRX Instruct generator with retrieved context
D) Vector search retriever (top-1000) → Llama 3.1 405B with full context window for synthesis

<details>
<summary>Answer</summary>

**Correct: C**

Standard RAG pipeline with reranker. Vector search (top-100) narrows from 30M docs efficiently. Reranker improves precision (top-10 most relevant). DBRX Instruct generates synthesis from focused context. Meets 5-10s latency. Reranker critical for multi-hop accuracy from large corpus.

**Why the others are wrong:**
- Fine-tuning 405B on 30M papers technically infeasible (memory/compute), extremely expensive, and 405B latency exceeds budget significantly
- Iterative retrieval without reranker wastes time on low-quality results; agent iterations add latency beyond budget; reranker needed for precision
- 1000 docs exceed context window limits; 405B latency far exceeds 10s budget; more docs doesn't improve quality without reranking

</details>

---

**Q8.** A content moderation system must classify user-generated posts as safe/unsafe and explain decisions to human reviewers. The model must handle 50+ languages and new slang terms emerge weekly. Accuracy is critical; false negatives (missing unsafe content) are unacceptable. Which approach best addresses these requirements?

A) Ensemble: Fine-tuned model for base classification + RAG for emerging terms + human-in-loop for low confidence
B) Fine-tune multilingual model on labeled moderation data, weekly retraining on new slang examples
C) RAG with updated slang/hate speech database, prompt engineering with multilingual examples, confidence threshold guardrails
D) Prompt engineering with detailed moderation policy, use GPT-4 external API for multilingual capability

<details>
<summary>Answer</summary>

**Correct: A**

Ensemble approach addresses all requirements: fine-tuned model handles established patterns with high accuracy, RAG covers emerging slang without retraining lag, human-in-loop prevents false negatives through manual review. Multilingual covered by base model. Weekly changes handled by RAG updates.

**Why the others are wrong:**
- Weekly retraining expensive and has lag between slang emergence and deployment; can't guarantee zero false negatives during lag period
- RAG alone insufficient for safety-critical classification; prompt engineering less reliable than fine-tuning for core moderation task; confidence thresholds don't eliminate false negatives
- External API adds dependency risk for critical safety system; prompt engineering less accurate than fine-tuning; doesn't address weekly update requirement efficiently

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

