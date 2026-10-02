# Answer Key — Practice Test 1

**Q1. Correct: D**

Databricks Vector Search is specifically designed for production RAG applications at scale. It provides managed vector indexing with Delta Sync, automatic embedding updates, and sub-second retrieval. The serving endpoint handles auto-scaling for concurrent users, and semantic search with filters enables accurate retrieval from large document sets. This is the only option that meets all requirements without custom infrastructure management.

---

**Q2. Correct: A**

Databricks Model Serving with provisioned throughput is the only option designed for real-time inference with SLA guarantees. GPU-enabled compute handles LLM inference efficiently, auto-scaling policies handle the 500 RPS peak requirement while scaling down during off-peak (cost optimization), and the service provides latency monitoring and guarantees. This is the production-grade solution for LLM serving.

---

**Q3. Correct: D**

Unity Catalog's Data Lineage feature automatically tracks dependencies across tables, views, models, and notebooks - essential for compound AI systems where multiple agents interact with various data sources and models. It provides end-to-end visibility of how data flows through agent interactions, which models consume which datasets, and how outputs propagate. This is the Unity Catalog-native solution for tracking complex AI system dependencies.

---

**Q4. Correct: C**

To provide accurate citations, the RAG system must track which source documents were retrieved and pass this metadata (document IDs, page numbers, URLs) to the LLM in the prompt. The LLM can then reference these sources in its response. Metadata filtering ensures proper tracking, and including document IDs in chunks enables the system to cite specific sources. This is the standard RAG pattern for citation-required applications.

---

**Q5. Correct: A**

Answer correctness using LLM-as-judge is the modern standard for evaluating factual accuracy of LLM responses (can compare semantic meaning against ground truth, not just surface-level text matching). Custom safety metrics with PII detection directly address the sensitive information requirement. This combination covers both evaluation needs: factual correctness and safety/compliance.

---

**Q6. Correct: B**

Tool calling (function calling) is the standard pattern for LLMs to interact with external APIs dynamically. The LLM analyzes the user query, determines which tools/APIs are needed, calls them with appropriate parameters, and incorporates the results into the response. This enables real-time data access based on actual user needs, not pre-fetched data. Major LLM providers (OpenAI, Anthropic) support this pattern natively.

---

**Q7. Correct: A**

Unity Catalog provides built-in PII detection through data profiling capabilities that can automatically identify sensitive data patterns across multiple PII types. This integrates with governance policies to systematically handle PII at scale while preserving data utility for model training.

---

**Q8. Correct: A**

Using Spark's binary file data source distributes PDF processing across the cluster for parallel execution. PyMuPDF handles complex PDF elements (tables, code) effectively, and pandas UDFs enable semantic chunking logic while maintaining Spark's distributed processing benefits. Sliding windows preserve context across page boundaries, which is critical for RAG quality.

---

**Q9. Correct: D**

MLlib's HashingTF and IDF are designed for distributed processing at scale and integrate natively with PySpark pipelines. Using Delta MERGE enables efficient incremental updates of only changed/new records rather than full rewrites. This combination is production-ready, maintainable, and leverages Spark's distributed architecture for 2M+ rows.

---

**Q10. Correct: C**

wholeTextFiles is specifically designed for loading many text files of varying sizes into a distributed DataFrame with filename as key. Map transformations efficiently handle custom tokenization and syntax validation in parallel. sampleBy() enables stratified sampling by package to achieve class balancing, all within Spark's distributed framework.

---

**Q11. Correct: A**

pandas_udf with GPU-enabled workers distributes embedding computation across multiple GPUs efficiently. Partitioning by product_id enables parallel processing while keeping related data together. Multi-node GPU cluster can process 10M reviews within 2 hours while optimize ensures efficient Delta storage. This balances cost (right-sized cluster for SLA) with performance.

---

**Q12. Correct: B**

Grouping by conversation_id and collecting messages into arrays preserves conversation structure. A UDF can then iterate through the ordered array to build cumulative prompt-completion pairs correctly (turn 1-2, turns 1-2→3, turns 1-2-3→4, etc.). This approach correctly handles the sequential, multi-turn nature of conversations and generates all valid training pairs.

---

**Q13. Correct: C**

Inference tables are the native Databricks Model Serving feature specifically designed to automatically capture request/response data, timestamps, and metadata including token usage for LLM endpoints. They provide immediate queryability through Delta tables without requiring custom implementation, making them the most efficient and reliable solution for monitoring GenAI applications.

---

**Q14. Correct: C**

Applying PII detection and redaction as a processing step in the retrieval pipeline (both pre-embedding and post-retrieval) ensures that sensitive information is handled consistently. Using a Python UDF with established PII detection libraries provides flexibility, can be integrated into the RAG workflow, and allows for customization of redaction rules while maintaining the semantic meaning needed for retrieval relevance.

---

**Q15. Correct: B**

Databricks Repos provides native Git integration that enables version control, collaboration, and audit logging through Git history. Prompt templates can be stored as code or configuration files, referenced programmatically in applications, and deployed consistently across environments. This approach aligns with software engineering best practices for managing application assets.

---

**Q16. Correct: C**

MLflow Tracing is specifically designed for GenAI applications and provides automatic instrumentation of LLM chains, capturing hierarchical spans, inputs/outputs, latency, and metadata for each step. It integrates natively with Databricks Model Serving and provides a unified view of complex multi-step workflows, making it the purpose-built solution for this use case.

---

**Q17. Correct: B**

Deploying open-source models on Databricks Model Serving GPU endpoints within the compliant region ensures complete control over data residency. All inference processing, logging, and storage remain within Databricks infrastructure in the specified region. Unity Catalog can be configured to store metadata and inference tables in the same region, providing end-to-end compliance.

---

**Q18. Correct: C**

Retrieving more candidates (k=15) and using a reranker model creates a two-stage retrieval process. The initial semantic search casts a wider net, then the reranker (which can consider query-document interaction more deeply) selects the truly most relevant documents. This pattern directly addresses the issue of relevant documents ranking just outside the top-k and is a proven RAG optimization technique with manageable latency impact.

---

**Q19. Correct: B**

Streaming responses allows the application to return tokens to the user incrementally as the LLM generates them, dramatically improving perceived latency and time-to-first-token. The overall generation time remains similar, but users see results immediately rather than waiting 8 seconds for the complete response. This is a standard optimization for production GenAI applications.

---

**Q20. Correct: C**

Custom callbacks via BaseCallbackHandler provide precise control over what gets logged at each stage (retrieval, LLM calls, chain execution). This allows capturing all required metadata (timestamps, user IDs, token counts) in a structured format suitable for compliance systems. Callbacks integrate cleanly with enterprise logging infrastructure and can send data to audit systems in real-time.

---

**Q21. Correct: A**

Architectural separation using a classifier model that lacks knowledge of available functions prevents the injection attack from even reaching the execution context. The classifier determines intent without knowing about refund processing capabilities, then a separate execution layer (which users cannot directly influence) handles the action. This defense-in-depth approach is robust against evolving prompt injection techniques.

---

**Q22. Correct: C**

The low GPU utilization (45%) indicates the model is underutilized - the GPUs are waiting between requests rather than being saturated. Dynamic batching allows the serving endpoint to group multiple concurrent requests and process them together, dramatically improving throughput and reducing per-request latency without adding hardware. This directly addresses the observed performance pattern.

---

**Q23. Correct: D**

Hosting the model on Databricks infrastructure gives complete control over data flow and ensures PHI never leaves the organization's environment. This is the only option that guarantees the LLM provider (OpenAI/Azure) never receives PHI. For HIPAA compliance, this architectural approach provides the strongest privacy guarantees.

---

**Q24. Correct: C**

A two-tier approach uses a lightweight, low-cost BERT model (inference cost <$0.0001 per classification on Databricks) to handle clear-cut safe/unsafe cases (likely 70-80% of posts), only using expensive GPT-4 for ambiguous cases. This hybrid architecture easily achieves 70%+ cost reduction while maintaining high accuracy, as the cheaper model handles obvious cases and the expensive model handles edge cases.

---

**Q25. Correct: A**

Reranking is a proven RAG optimization technique that retrieves more candidates initially (e.g., k=15-20) then uses a cross-encoder or reranking model to select the most relevant subset (e.g., top 5) before LLM processing. This improves answer quality by ensuring better context selection while controlling latency and token costs since only the reranked subset goes to the LLM. Databricks recommends this two-stage retrieval pattern for production RAG applications.

---

**Q26. Correct: B**

Databricks serverless compute for Model Serving with min_provisioned_throughput=0 allows complete scale-to-zero during idle periods (maximum cost savings) while max_provisioned_throughput=1000 provides headroom for traffic spikes. Serverless automatically scales based on demand and maintains sub-second cold start times for foundation models, meeting the 2-second requirement during normal operations while eliminating costs during low-traffic periods. This is the optimal configuration for variable workloads.

---

**Q27. Correct: B**

MLflow models registered with the RAG chain as a pyfunc model can be deployed directly to Databricks Model Serving endpoints, which provide built-in versioning, A/B testing, rollback capabilities, automatic scaling, and comprehensive endpoint monitoring. This is the recommended pattern for production RAG deployments.

---

**Q28. Correct: D**

MLflow's evaluate() function with LLM-as-a-judge is the standard approach for automated prompt testing. It allows you to compare prompt versions against golden test datasets using metrics like faithfulness, relevance, and correctness scored by a judge model. This provides quantitative validation before deployment.

---

**Q29. Correct: B**

Semantic caching stores responses for semantically similar queries (not just exact matches), dramatically reducing latency for repeated or similar questions. For RAG applications, many user queries are variations of the same intent, making semantic caching highly effective at reducing both latency and LLM API costs.

---

**Q30. Correct: C**

Creating a custom pyfunc model is the correct way to package complex multi-step chains with custom logic. The pyfunc interface allows you to wrap arbitrary Python code in the predict() method, include all dependencies, and deploy as a single model to Model Serving. This ensures the entire chain logic deploys atomically.

---

**Q31. Correct: C**

Precision@k measures what percentage of the top-k retrieved documents are actually relevant to the query. This directly evaluates retrieval quality, which is critical for RAG performance. Without relevant context, the LLM cannot generate accurate answers regardless of model quality.

---

**Q32. Correct: D**

PII redaction must happen in the serving model's predict() method before the user query is sent to the external LLM. This ensures PII is removed from the actual request payload sent to third-party APIs. The pyfunc model has access to the full request and can preprocess it before LLM invocation.

---

**Q33. Correct: B**

MLflow Model Serving endpoints provide native support for GenAI applications with auto-scaling (handles traffic spikes), low-latency serving infrastructure (minimizes cold starts), and built-in A/B testing via endpoint traffic splitting across multiple model versions. This is the recommended Databricks-native approach for production GenAI deployments.

---

**Q34. Correct: A**

Creating a custom PyFunc model is the most flexible and recommended approach for complex GenAI applications. It allows full control over the predict() method to handle preprocessing, chain execution, and postprocessing. Dependencies are properly managed via conda.yaml or requirements.txt, and the model signature ensures type safety. This approach works with any Python code and integrates seamlessly with Model Serving.

---

**Q35. Correct: A**

For Unity Catalog registered models, Model Serving endpoints use the three-level namespace (catalog.schema.model), support GPU workload sizes (GPU_SMALL, GPU_MEDIUM, GPU_LARGE), and authentication/authorization is automatically handled through Unity Catalog permissions. Auto-scaling from 0 (scale-to-zero) is supported and cost-effective for variable workloads.

---

**Q36. Correct: D**

Enabling auto-scaling and increasing max instances directly addresses the concurrent request limit issue. Model Serving endpoints auto-scale based on load, and increasing max instances allows the endpoint to handle more parallel requests during traffic spikes. Queue depth metrics help verify the scaling is working correctly and requests aren't being dropped.

---

**Q37. Correct: D**

AI Gateway provides built-in PII detection and redaction filters that operate in real-time before requests reach the model, and Model Serving inference tables automatically capture all inputs and outputs to Delta tables for compliance auditing. This is the native Databricks approach for governed GenAI deployments.

---

**Q38. Correct: A**

Unity Catalog metastores can be deployed regionally, and workspace-to-metastore assignments create hard boundaries ensuring models in a workspace can only query data in their assigned regional metastore. This provides the strongest data residency guarantee at the architectural level.

---

**Q39. Correct: D**

AI Gateway automatically logs all requests with user identity, timestamps, prompts, and responses. Combined with Unity Catalog audit logging (which captures access to document tables) and extended retention policies on Delta tables, this provides complete immutable audit trails queryable via SQL for the required 7-year period.

---

**Q40. Correct: D**

EXECUTE privilege on a Unity Catalog registered model is the specific privilege that allows a user to deploy that model to Model Serving endpoints. This is the least-privilege approach for model deployment in Databricks governance.

---

**Q41. Correct: D**

faithfulness (also called groundedness) specifically measures whether the generated answer is factually consistent with and supported by the retrieved context. It's the primary metric for verifying that LLM responses don't hallucinate beyond what's in the source documents.

---

**Q42. Correct: B**

Inference Tables automatically capture all model inputs and outputs in Delta tables, and can be configured with additional monitoring models (like toxicity classifiers) that run in real-time to flag harmful content. This is the purpose-built solution for LLM observability in Databricks.

---

**Q43. Correct: A**

mlflow.evaluate() with model_type='question-answering' and ground truth labels leverages LLM-as-judge metrics (answer_similarity, answer_correctness) across all 200 examples, providing quantitative, reproducible, comprehensive evaluation. This uses your existing labeled data efficiently and computes standardized metrics for comparison.

---

**Q44. Correct: A**

Precision = relevant_retrieved / total_retrieved. If precision drops while recall stays stable, it means more irrelevant documents are being retrieved. Adding new documents without adjusting similarity thresholds is the most common cause - the system retrieves the same number of documents (maintaining recall) but more are now irrelevant (lowering precision).

---

**Q45. Correct: B**

This is the correct Databricks-native pattern: Inference Tables capture all predictions, Lakehouse Monitoring can compute custom LLM metrics (faithfulness, relevance, etc.) and trigger alerts when thresholds are breached, and Databricks Jobs can be programmatically triggered to execute retraining pipelines. This creates a complete MLOps loop.

---

