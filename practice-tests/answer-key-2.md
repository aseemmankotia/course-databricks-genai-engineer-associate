# Answer Key — Practice Test 2

**Q1. Correct: D**

This solution addresses all constraints: Vector Search with pre-computed embeddings provides sub-second retrieval at scale; Redis caching reduces redundant LLM calls for common queries (critical for cost with 10K concurrent users); smaller open-source model significantly reduces token costs while few-shot prompting maintains accuracy for specialized compliance tasks. Pre-computed embeddings eliminate query-time embedding latency.

---

**Q2. Correct: C**

This approach enforces ground-truth constraints at generation time. Retrieving specifications from Unity Catalog ensures factual accuracy; constrained prompting with JSON schema validation (using tools like Pydantic or function calling) forces the LLM to select only from provided values rather than generate free-form specifications. This prevents hallucination structurally, not just probabilistically. The LLM can still personalize tone and style while being constrained to factual specifications.

---

**Q3. Correct: A**

Semantic compression addresses the core challenge: at 8 turns average with 50K users, raw history creates massive token consumption. This approach extracts and stores only salient facts (e.g., 'patient asked about cholesterol on turn 3, value was 215'), then retrieves relevant compressed context. This dramatically reduces tokens while maintaining semantic accuracy. Vector similarity ensures the right context is retrieved when needed (e.g., cholesterol question retrieves prior cholesterol discussion).

---

**Q4. Correct: D**

Citation errors occur because retrieved chunks lack explicit source metadata that the LLM can reference. Including document_id, page_number, paragraph_number as metadata with each chunk (stored in Vector Search metadata fields) ensures this information flows to the LLM. Structured output formats (JSON schema, function calling) force the LLM to explicitly populate citation fields from the metadata rather than hallucinating page numbers. This is an indexing and prompt engineering solution to a retrieval-generation alignment problem.

---

**Q5. Correct: C**

This approach separates expensive processing (embedding all frames, done once during indexing) from cheap query-time operations (retrieve top 5 frames, send only those to LLM). At 1,000 queries/hour, sending 5 frames is far cheaper than processing 500MB videos. Vector search over frame embeddings intelligently selects the most visually distinct/representative frames, so the LLM sees video diversity without processing all content. This is a classic pre-computation + query-time refinement pattern.

---

**Q6. Correct: A**

This addresses the core failure mode: classification errors cascade through the pipeline. Confidence-based fallback catches uncertain classifications before they cause retrieval errors. Re-ranking retrieved articles against the original query (not classified intent) provides error correction - even if intent is misclassified, relevant articles can still surface. This adds redundancy and error tolerance without abandoning the efficiency benefits of specialized routing. The confidence threshold provides a tunable quality/cost tradeoff.

---

**Q7. Correct: D**

Semantic chunking with overlap preserves context across boundaries, filtering extreme outliers prevents OOM errors, and creating separate examples from chunks maximizes information retention while maintaining trainable batch sizes. This approach balances token limits (typically 4K-32K for modern models) with data quality.

---

**Q8. Correct: B**

Databricks Vector Search provides managed ANN indexing (HNSW-based) with native Delta Sync for automatic updates, sub-100ms query performance at scale, and built-in metadata filtering. The delta-sync index ensures the vector store stays current with source Delta tables without manual intervention.

---

**Q9. Correct: D**

Entity token replacement removes PII while preserving conversational structure, sentence length, and context flow. The model learns to handle placeholder tokens that can be replaced with appropriate values at inference time. This is standard practice in PII-aware NLP preprocessing.

---

**Q10. Correct: C**

Static analysis provides deterministic, scalable validation. Linters catch syntax errors, deprecation checks verify API currency, and automated tests validate logic. This creates an objective, reproducible quality filter that directly addresses the three identified error types without requiring labeled data or manual review.

---

**Q11. Correct: D**

Structure-aware parsing respects document semantics by treating tables and code as atomic units (preventing fragmentation), using header hierarchy for natural boundaries, and allowing variable chunk sizes to accommodate complete semantic units. The 256-2048 token range handles both brief sections and complex diagrams while overlap at transitions preserves cross-reference context.

---

**Q12. Correct: A**

Micro-batching with GPU acceleration leverages parallel processing to amortize model loading overhead and exploit GPU's parallel compute. Processing 100 messages simultaneously can reduce per-message latency from 800ms to <50ms while maintaining full classifier accuracy. This is the standard approach for scaling neural inference.

---

**Q13. Correct: C**

mlflow.search_runs() supports filter_string for querying runs by parameters (like prompt template version), can sort results by any metric column including metrics.f1_score, and returns a DataFrame making it easy to select the top run. The run_id can then be passed directly to mlflow.register_model() for model registry and deployment. This is the most efficient programmatic approach.

---

**Q14. Correct: D**

Explicit grounding instructions in the system prompt combined with validation is the standard RAG pattern to prevent hallucination. Instructing the model to acknowledge when context is insufficient prevents fabrication, and response validation can detect when the model deviates from provided context. This addresses the root cause: the model generating information not present in retrieved documents.

---

**Q15. Correct: A**

Few-shot prompting with schema-specific examples is proven to dramatically improve SQL generation accuracy. Providing 3-5 concrete examples of correct queries for the specific database schema helps the LLM understand table relationships, column names, and query patterns. This is the standard approach for improving Text-to-SQL accuracy in production.

---

**Q16. Correct: D**

The sliding window + summarization pattern is the industry-standard approach for managing long conversation context. It keeps recent exchanges for immediate context (which matter most for response quality) while compressing older history into summaries, preventing context window overflow and maintaining low latency. This is how production chat systems like ChatGPT handle long conversations.

---

**Q17. Correct: C**

Task-level max_retries in Databricks Jobs retries only the specific failed task (Step 2) without re-executing upstream tasks (Step 1). The retry_on_timeout parameter specifically handles transient availability errors like model endpoint failures. Databricks preserves task outputs, so retried tasks can use results from previously completed tasks.

---

**Q18. Correct: A**

Parameterized queries are the gold-standard defense against SQL injection. By designing the tool to enforce parameter binding at the implementation level (not relying on the LLM's behavior), you create a hard security boundary. The tool description guides the agent to provide inputs in the correct format, but the code enforcement ensures even if the agent misbehaves, injection is impossible. Defense in depth: technical control > prompt instruction.

---

**Q19. Correct: B**

Broadcasting the model ensures it's loaded once per executor (not once per task), and pandas_udf with Iterator input hint processes data in batches while reusing the model instance. This pattern is specifically designed for distributed inference with large models in Databricks, maintains Unity Catalog lineage, and solves the memory issue by avoiding repeated model loading.

---

**Q20. Correct: C**

With 8 seconds total latency and only 1.7 seconds accounted for (200ms + 1500ms), approximately 6.3 seconds is vector search time. Retrieving 10 chunks and then re-ranking to select the best 5 reduces initial vector search cost while maintaining quality. Two-stage retrieval (cheap first-pass, expensive re-rank on fewer candidates) is a proven pattern for latency-sensitive RAG applications.

---

**Q21. Correct: D**

The instructor library (or similar structured output libraries) uses Pydantic models to define schemas, incorporates schema into prompts, validates outputs, and automatically retries with validation errors fed back to the LLM. This provides the strongest guarantee of schema compliance through a validation-retry loop, which directly addresses the 15% failure rate.

---

**Q22. Correct: B**

Unity Catalog encryption with customer-managed keys covers MLflow artifacts when logged to UC volumes. Custom prediction logging to a MANAGED Delta table ensures predictions are stored in Unity Catalog with encryption, and the table can enforce schema with user_id for attribution. This keeps all compliance-sensitive data within Unity Catalog's governance boundary with proper encryption.

---

**Q23. Correct: A**

The scenario explicitly states memory retrieval successfully fetches ALL messages but context is lost after ~20 turns. With typical messages consuming 100-200 tokens each, 20+ turns likely exceed the 8,192 token context window. The LLM API automatically truncates input that exceeds its limit, causing older messages to be silently dropped even though they were retrieved from the Delta table.

---

**Q24. Correct: C**

Registering each agent separately maintains proper versioning boundaries (agents can be updated independently), conda environments ensure true dependency isolation per agent, and Model Serving provides scalability and monitoring per agent. Databricks workflows orchestrating via REST API is the standard pattern for multi-model systems and maintains Unity Catalog governance for all components.

---

**Q25. Correct: D**

GPU acceleration is critical for transformer-based models (Hugging Face pipelines). GPU_MEDIUM instances provide hardware acceleration that reduces inference latency by 10-20x for LLM operations. The batch inference configuration allows the endpoint to process multiple requests together, further optimizing GPU utilization and reducing per-request latency to well under 500ms.

---

**Q26. Correct: B**

Embedding model consistency is mandatory for vector similarity search. The query embedding must use the same model (gte-large) as the indexed document embeddings. Different embedding models produce incompatible vector spaces, causing cosine similarity to fail. Updating the retriever to use gte-large immediately fixes retrieval accuracy without requiring index rebuild or data re-ingestion.

---

**Q27. Correct: B**

Request batching with proper configuration (max_batch_size=32, max_wait_time=100ms) processes multiple requests simultaneously, dramatically reducing queue depth while maintaining sub-2s latency. Setting min_workers=3 ensures baseline capacity during traffic spikes. This approach optimizes throughput without over-provisioning.

---

**Q28. Correct: B**

Deploying both models to a shared GPU endpoint minimizes infrastructure costs while maintaining isolation via separate vector indices. Feature flags enable instant A/B test control and sub-30-second rollback by simply toggling the flag based on error rate monitoring. This architecture separates deployment (models) from traffic routing (feature flags), providing maximum flexibility.

---

**Q29. Correct: B**

LangGraph's MemorySaver with Delta table checkpointing natively integrates with the LangGraph execution model, storing agent state and conversation history directly in Unity Catalog. This ensures data residency compliance, provides ACID guarantees, supports concurrent access through Delta's versioning, and enables automatic lifecycle management through Unity Catalog policies.

---

**Q30. Correct: A**

Speculative decoding with a smaller draft model (Llama 3.1 8B) can achieve 2-3x speedup for large models by generating multiple tokens per forward pass when the draft model's predictions are accepted. This directly addresses the 12 tokens/sec bottleneck without requiring hardware changes. KV-cache optimization further reduces memory bandwidth constraints during generation.

---

**Q31. Correct: C**

Mosaic AI Gateway provides native middleware support for custom PII detection that processes requests/responses before logging, ensuring redacted data in audit trails. Unity Catalog tables with Delta sharing enable secure compliance team access. MLflow Model Registry stage transitions (Production→Archived, Staging→Production) enable sub-5-minute rollback without redeployment, meeting all requirements in an integrated platform.

---

**Q32. Correct: B**

Serverless endpoints for embedding/reranker handle bursty traffic efficiently with instant autoscaling and zero cost during idle periods (80% of the hour). Provisioned GPU with min_workers=2 for the LLM ensures the latency-critical component has baseline capacity during bursts while avoiding cold-start delays that would violate P99<5s. This right-sizes infrastructure for actual usage patterns.

---

**Q33. Correct: A**

Vector database query performance is the bottleneck, so optimizing the database layer directly with Photon acceleration improves query performance. Connection pooling reduces database connection overhead under concurrent load, and client-side caching for frequent queries (common in RAG patterns) reduces redundant database hits. This addresses the specific bottleneck without overprovisioning other components.

---

**Q34. Correct: C**

Provisioned throughput endpoints have TWO separate limits: token throughput (10,000 tokens/min in this case) AND concurrent request count (default 10). High concurrency with lower per-request token counts can hit the concurrent request limit while staying under the token throughput limit. This is a common production issue where burst traffic patterns exceed concurrency limits despite acceptable average throughput.

---

**Q35. Correct: D**

AI Gateway is specifically designed for routing, governance, and consistent request handling across multiple model endpoints. Custom routing rules enable the classification-based routing logic. Backing routing decisions with a Delta table provides the persistence needed for consistent routing of identical inputs. Gateway's built-in support for multiple providers and routes naturally supports A/B testing through configuration changes without code deployment.

---

**Q36. Correct: B**

Packaging preprocessing as a reusable Python package in Unity Catalog provides centralized version control, governance, and easy dependency management. Both the embedding model (via model.py dependencies) and retrieval service can import the same versioned package, ensuring consistency. This follows software engineering best practices for shared libraries without introducing unnecessary serving overhead or architectural complexity.

---

**Q37. Correct: D**

AI Gateway with inference tables provides automatic logging of all requests/responses with user attribution, PII detection rules can block SSNs in the request policy before reaching the model, and tag-based tracking (via workspace or endpoint tags) enables departmental cost allocation - meeting all three requirements in a unified governance layer.

---

**Q38. Correct: A**

Model Serving with Customer-Managed Keys (CMK) for GPU model serving allows the model weights to remain encrypted with customer-managed keys and be served from customer-managed infrastructure (compute) while still using Databricks' serving capabilities - this is the only option that addresses both model weight residency and CMK encryption for inference.

---

**Q39. Correct: A**

Creating a view without the id column and granting appropriate permissions (SELECT on full table for data scientists, SELECT on view for production apps) is the standard Unity Catalog pattern for column-level access control. Unity Catalog audit logs automatically capture all access with user/service principal identity. This is the most straightforward and maintainable approach using native UC features.

---

**Q40. Correct: C**

AI Gateway with custom request ID propagation allows tracking costs across chained calls (since all go through the gateway), toxicity filters in response policy provide real-time detection and blocking, and Unity Catalog's model versioning with retention policies ensures the exact model artifacts are available for 90 days. This provides unified governance at the gateway layer rather than bolting on monitoring afterward.

---

**Q41. Correct: A**

Conditional logging based on performance thresholds (relevance < 0.7) captures detailed context only for problematic queries, making root cause analysis efficient. This approach balances storage costs with diagnostic value and enables SQL-based aggregation to identify patterns. The exam tests understanding of practical monitoring strategies that scale.

---

**Q42. Correct: B**

NLI models directly test whether summary content is supported by source text, which is the definition of hallucination detection. The sentence-level analysis provides granular identification of hallucinated content. This is a grounded, verifiable approach that scales. The exam tests understanding that hallucination detection requires checking against source, not reference summaries.

---

**Q43. Correct: C**

Logging at agent boundaries with correlation IDs enables tracing failures to specific agents. Schema validation on data contracts between agents catches structural and semantic inconsistencies at the source. This provides immediate, actionable diagnostics. The exam tests practical distributed system monitoring patterns applied to LLM pipelines.

---

**Q44. Correct: A**

Creating temporal-specific test sets directly measures whether newer documents are disadvantaged, which isolates temporal bias from other retrieval issues. This targeted evaluation reveals if the problem is specific to recent content. The exam tests diagnostic strategies that isolate specific failure modes.

---

**Q45. Correct: B**

Rule-based detection using regex and keywords has deterministic <100ms latency and can run synchronously on every request at 500 req/min scale. While less sophisticated than ML-based detection, it catches known injection patterns reliably within the latency constraint. The exam tests understanding of performance-security tradeoffs.

---

