# Practice Test 2: Databricks GenAI Engineer Associate Exam Preparation

- **Questions:** 45
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-2.md`. Track results in `score-tracker.md`.

---

**Q1.** A financial services company is building a RAG application that must retrieve relevant documents from a 500GB corpus of regulatory filings, then use an LLM to generate compliance summaries. The application requires sub-second retrieval latency and must handle 10,000 concurrent users during market hours. Token costs must be minimized while maintaining accuracy. Which architectural approach BEST meets these requirements?

A) Store all documents in Delta Lake, use full-text search with Databricks SQL for retrieval, and call GPT-4 for each summary generation with no caching
B) Embed documents at query time using a foundation model, store embeddings in DBFS, use brute-force similarity search, and generate summaries with Claude 3.5 Sonnet
C) Use Databricks AI Gateway to route all queries to multiple LLM providers simultaneously, select the fastest response, and store results in Unity Catalog tables
D) Use Databricks Vector Search with a pre-computed embedding index, implement query result caching in Redis, and use a smaller LLM (e.g., Llama 3.1 8B) with few-shot prompting for generation

---

**Q2.** An e-commerce platform needs to generate personalized product descriptions using customer browsing history and product attributes. The system processes 2 million products across 50 categories. Descriptions must be factually accurate and include specific product specifications that cannot be hallucinated. The LLM frequently generates descriptions mentioning features the product doesn't have. Which design pattern MOST effectively prevents hallucination while maintaining personalization?

A) Use a temperature of 0.0 and provide all product specifications in the system prompt with explicit instructions not to add features
B) Fine-tune a foundation model on 10,000 product descriptions with RLHF to penalize hallucinated features, then deploy with standard prompting
C) Implement a two-stage approach: first retrieve ground-truth product specifications from Unity Catalog, then use constrained prompting with JSON schema validation requiring the LLM to only select from provided specification values
D) Chain three LLM calls: first generates description, second fact-checks against product database, third rewrites if inconsistencies found

---

**Q3.** A healthcare application uses a conversational agent built on Databricks to answer patient questions about lab results. The system must handle multi-turn conversations where context from previous messages affects interpretation (e.g., 'What about my cholesterol?' after discussing blood work). Conversations average 8 turns and the application serves 50,000 daily active users. Token costs are a primary concern. Which state management strategy BEST balances cost and context accuracy?

A) Implement semantic compression: extract and store key entities and facts from each turn in Delta Lake, then retrieve only relevant compressed context for each new query using vector similarity
B) Store full conversation history in MLflow and pass all previous messages to the LLM on each turn
C) Use a sliding window of the last 3 message pairs, discarding older context entirely
D) Fine-tune a model with extended context length (128K tokens) and always pass full conversation history

---

**Q4.** A legal tech company is deploying a RAG system that must cite specific page numbers and paragraphs from source documents in generated responses. The system uses Databricks Vector Search to retrieve relevant chunks from 100,000 legal documents. During testing, the LLM frequently provides correct information but cites incorrect page numbers or attributes information to the wrong document. Chunk size is 512 tokens with 50-token overlap. Which modification MOST effectively improves citation accuracy?

A) Reduce chunk size to 256 tokens and increase overlap to 128 tokens
B) Increase embedding model dimensions from 768 to 1536 to capture more semantic nuance
C) Switch from semantic chunking to fixed-size chunking with no overlap
D) Include rich metadata (document_id, page_number, paragraph_number, section_title) with each chunk at indexing time, and engineer prompts to explicitly extract and return citation metadata alongside generated text using structured output formats

---

**Q5.** A media company is building an application that generates video thumbnails and titles using multimodal LLMs. Videos are stored in Unity Catalog Volumes (average 500MB each, 10,000 videos). The application must process each video once for initial indexing, then handle 1,000 user queries per hour requesting thumbnail/title variations. Token costs for processing full videos are prohibitive. Which architectural pattern BEST optimizes cost while maintaining quality?

A) Downsample videos to 240p resolution and pass entire videos to the LLM for each query
B) Use Databricks AutoML to train a custom thumbnail selection model on 1,000 labeled examples, then use that model instead of an LLM
C) Extract keyframes at 1-second intervals from each video, embed all frames using a vision model, store embeddings in Vector Search, and query-time retrieve the top 5 most representative frames to send to the multimodal LLM
D) Process full videos with the multimodal LLM during initial indexing, store all possible thumbnail/title variations in Delta Lake, then serve pre-computed results for user queries

---

**Q6.** A customer support application uses a compound AI system with three components: (1) intent classifier (routes to specialized models), (2) retrieval system (fetches knowledge base articles), and (3) response generator (creates answers). During A/B testing, the system shows 15% lower accuracy than a single GPT-4 call, despite individual components each testing at >90% accuracy. Investigation shows most errors occur when the classifier misidentifies intent, causing retrieval of irrelevant articles. Which design change MOST effectively improves end-to-end accuracy?

A) Implement a confidence-based fallback: if the intent classifier confidence is below 0.85, bypass specialized routing and use a general-purpose model; add a re-ranking step after retrieval that scores article relevance to the original query, not just the classified intent
B) Replace the intent classifier with a more sophisticated model fine-tuned on 50,000 labeled support tickets
C) Remove the intent classifier entirely and use a single large model for all queries
D) Add a fourth component that validates the response against the retrieved articles using an LLM-as-judge before returning to the user

---

**Q7.** A data engineering team is preparing a dataset of customer support tickets for fine-tuning a generative model. The raw data contains a 'resolution_notes' column with values ranging from empty strings to 50,000+ character narratives. During initial training runs, the team observes inconsistent model performance and out-of-memory errors. Which preprocessing strategy BEST addresses this issue while preserving information quality for fine-tuning?

A) Truncate all resolution_notes to 512 tokens using a fixed substring operation, then pad shorter entries with whitespace to ensure uniform length
B) Convert all resolution_notes to embeddings using a pre-trained model, store only the vector representations, and fine-tune on the embedded space
C) Apply aggressive text summarization using an LLM to reduce all resolution_notes to exactly 256 tokens, ensuring consistent input length across the dataset
D) Filter out records where resolution_notes exceed 8192 tokens, implement semantic chunking with overlap for longer notes, and create separate training examples from each chunk while preserving ticket metadata

---

**Q8.** A generative AI application requires a vector database to support retrieval-augmented generation (RAG) across 40 million technical documentation chunks. The team has already generated embeddings using a 1536-dimensional model. They need to optimize for sub-100ms p95 query latency while supporting metadata filtering on 5 categorical fields. Which Databricks-native solution BEST meets these requirements?

A) Store embeddings in Delta Lake tables with ZORDER clustering on the embedding column and use broadcast joins for metadata filtering
B) Use Databricks Vector Search with a delta-sync index, configuring embedding dimensions to 1536 and enabling filtered search with the metadata columns
C) Implement a custom approximate nearest neighbor (ANN) solution using Spark MLlib's LSH (Locality-Sensitive Hashing) with manual index management
D) Store embeddings in Unity Catalog volumes as Parquet files with Hive-style partitioning on metadata fields, using PySpark UDFs for cosine similarity

---

**Q9.** During data preparation for a domain-specific chatbot, a data scientist discovers that 15% of training examples contain personally identifiable information (PII) including email addresses, phone numbers, and account IDs embedded within conversational context. The organization's compliance policy requires PII removal, but preserving conversational structure is critical for model quality. Which approach BEST balances compliance with training effectiveness?

A) Remove all training examples containing any PII, then augment the dataset by generating synthetic conversations using a large foundation model
B) Encrypt all PII fields using AES-256 before training, then decrypt model outputs in post-processing
C) Apply differential privacy noise injection to the embedding layer during training to mask PII information
D) Use regular expressions to detect and replace PII patterns with entity type tokens (e.g., '[EMAIL]', '[PHONE]', '[ACCOUNT_ID]') while maintaining sentence structure and conversational flow

---

**Q10.** A team is preparing a training dataset for instruction fine-tuning a coding assistant. The raw data consists of code snippets paired with natural language descriptions, but 30% of snippets contain syntax errors, deprecated API calls, or incomplete logic. The team wants to improve dataset quality before fine-tuning. Which validation strategy provides the MOST effective quality control?

A) Fine-tune a separate classifier model to predict code quality scores, then filter out examples below a threshold score
B) Apply clustering algorithms to code embeddings and manually review cluster centroids to identify and remove low-quality patterns
C) Use static code analysis tools (linters/parsers) to programmatically validate syntax correctness, filter out snippets with deprecated imports by checking against current library documentation, and implement automated test execution to verify code logic completeness
D) Increase the dataset size by 3x through data augmentation to dilute the impact of erroneous examples during training

---

**Q11.** An organization is building a RAG system that requires splitting 250,000 PDF documents (technical manuals averaging 400 pages each) into semantically coherent chunks for embedding. The documents contain complex tables, multi-level numbered lists, code blocks, and cross-references. Initial tests with fixed 512-token chunking resulted in fragmented tables and broken code samples. Which chunking strategy BEST preserves semantic coherence for this document type?

A) Use a sliding window approach with 512-token chunks and 50% overlap to ensure all content appears in multiple contexts
B) Apply sentence-level splitting using spaCy, then group consecutive sentences until reaching 512 tokens, creating uniform-sized chunks
C) Employ a recursive character splitting algorithm that divides text at paragraph boundaries with strict 512-token limits and no overlap
D) Implement document structure-aware parsing that identifies section boundaries using header hierarchy, treats tables and code blocks as atomic units, and creates chunks that respect these structural boundaries with size ranging from 256-2048 tokens and 10% overlap at section transitions

---

**Q12.** A data preparation pipeline processes streaming user feedback for real-time model monitoring. The pipeline must detect and flag potentially toxic content before it enters the evaluation dataset. The current approach uses a pre-trained toxicity classifier that runs on each message, but processing latency has increased to 800ms per message as volume scaled to 50,000 messages/hour. Which optimization provides the MOST effective latency reduction while maintaining detection accuracy?

A) Implement batch processing with micro-batches of 100 messages, using GPU acceleration for parallel inference on the toxicity classifier
B) Replace the classifier with a keyword blocklist containing 5,000 toxic terms and phrases for O(1) lookup time
C) Cache classifier results using a hash of message content and serve cached predictions for duplicate or near-duplicate messages
D) Sample 10% of messages for toxicity classification and apply the resulting toxicity rate proportionally to the full dataset

---

**Q13.** A GenAI application uses MLflow to track multiple LLM experiments with different prompt templates. The team needs to programmatically retrieve the run that achieved the highest F1 score for a specific prompt template version and deploy it to production. Which MLflow API combination accomplishes this?

A) Use mlflow.get_experiment_by_name() followed by mlflow.list_run_infos() to iterate through all runs and manually compare F1 scores
B) Use mlflow.tracking.MlflowClient().get_metric_history() for each run, compare F1 values, then use mlflow.pyfunc.load_model() to deploy the winner
C) Use mlflow.search_runs() with filter_string to find runs matching the prompt version, sort by metrics.f1_score descending, then use mlflow.register_model() with the best run_id
D) Use mlflow.autolog() with automatic model selection enabled and set target_metric='f1_score' in the MLproject file

---

**Q14.** A company deploys a RAG application where a Databricks vector search index is queried to retrieve context, which is then passed to a foundation model API. Users report that responses sometimes contain information contradicting the retrieved documents. The development team suspects the issue occurs when the LLM ignores retrieved context. Which implementation change BEST ensures the model strictly adheres to provided context?

A) Increase the vector search k parameter to retrieve more documents, ensuring the correct information is always present in the context
B) Switch from embedding-based vector search to keyword-based search to improve retrieval precision
C) Fine-tune the foundation model on the company's document corpus to improve its knowledge of domain-specific facts
D) Add explicit instructions in the system prompt: 'Answer ONLY using the provided context. If the context does not contain the answer, respond with: I don't have enough information.' and include response validation that checks for hallucination markers

---

**Q15.** An application uses LangChain with Databricks SQL to query structured customer data and generate natural language summaries. The chain executes: SQLDatabaseChain → LLM generates SQL → Execute query → LLM summarizes results. During testing, some generated SQL queries fail with syntax errors. Which LangChain configuration reduces SQL generation errors?

A) Replace SQLDatabaseChain with a custom chain that uses few-shot prompting with 3-5 example query pairs (natural language question → correct SQL) specific to the schema
B) Increase the LLM temperature parameter to 0.9 to generate more creative SQL queries that might match the schema better
C) Add a retry mechanism with exponential backoff to automatically re-execute failed queries
D) Switch from SQLDatabaseChain to SQLDatabaseSequentialChain which automatically validates SQL syntax before execution

---

**Q16.** A chatbot application uses Databricks Model Serving to host a fine-tuned LLM with conversational memory. The team implements session management where each user's conversation history is stored in a Databricks SQL table with columns: session_id, user_id, timestamp, role (user/assistant), message. The endpoint currently loads the full conversation history for context on every request. As conversations grow longer, inference latency increases significantly. Which optimization maintains conversation quality while reducing latency?

A) Partition the conversation table by session_id and create a Z-order index on timestamp to speed up history retrieval queries
B) Switch to storing conversation history in MLflow experiment tracking instead of Databricks SQL for faster retrieval
C) Increase the Model Serving endpoint scale-out to more instances to parallelize history loading across workers
D) Implement a sliding window approach that retrieves only the most recent N exchanges (e.g., last 10 messages) plus a summarization of older context, updating the summary periodically using a separate LLM call

---

**Q17.** A Databricks workflow orchestrates a GenAI pipeline: Step 1 preprocesses documents, Step 2 generates embeddings using a Model Serving endpoint, Step 3 updates a vector search index, Step 4 runs evaluation metrics. Step 2 occasionally fails with 'Model endpoint temporarily unavailable' errors. The team needs the workflow to automatically retry Step 2 without re-running Step 1. Which Databricks Jobs configuration achieves this?

A) Enable workflow-level retry with max_concurrent_runs: 3, which reruns the entire workflow up to 3 times until all steps succeed
B) Configure a repair run policy in the workflow settings with depends_on pointing Step 2 to Step 1's output path
C) Set max_retries: 3 and retry_on_timeout: true in the task configuration for Step 2, which automatically retries only the failed task while preserving outputs from previous tasks
D) Implement a DLT (Delta Live Tables) pipeline instead of a Jobs workflow, as DLT automatically handles transient endpoint failures with built-in retry logic

---

**Q18.** A development team builds a compound AI system where an LLM agent uses multiple tools: a calculator tool (Python function), a web search tool (external API), and a database query tool (SQL execution). The agent uses ReAct prompting to decide which tool to use. During testing, the agent sometimes calls the database tool with user-provided input directly in the SQL query, creating SQL injection risks. Which implementation pattern BEST mitigates this security vulnerability?

A) Implement the database tool to ONLY accept parameterized query templates where user input is bound as parameters, never concatenated into SQL strings. Add tool description instructions that specify the agent must provide parameters separately from the query template
B) Add input sanitization that escapes special SQL characters before passing user input to the database tool function
C) Fine-tune the LLM on examples of safe SQL queries to teach it not to generate injection-vulnerable code
D) Add a prompt instruction: 'Never include user input directly in SQL queries' and enable strict mode to force the agent to follow instructions

---

**Q19.** A GenAI application uses a custom PyFunc model that loads a fine-tuned LLM and applies prompt templates. The model is registered in Unity Catalog. During batch inference on a large dataset, the application fails with out-of-memory errors when multiple workers try to load the model simultaneously. Which approach BEST resolves this issue while maintaining Unity Catalog governance?

A) Increase the driver memory allocation and run inference on the driver node only using model.predict() in a single-threaded loop
B) Implement a broadcast variable to share the loaded model across executors and use pandas_udf with Iterator[pd.Series] input hint
C) Use model serving endpoints with auto-scaling enabled and call the REST API from a Spark UDF for distributed inference
D) Convert the PyFunc model to ONNX format and deploy to an external inference service outside Unity Catalog

---

**Q20.** A development team is building a RAG application where users query internal company documents. The vector search index contains 500,000 embedded document chunks stored in a Delta table with Unity Catalog. During testing, queries return relevant results but response latency averages 8 seconds. The embedding model takes 200ms and LLM generation takes 1.5 seconds. Which optimization would MOST reduce the overall latency?

A) Enable Delta table liquid clustering on the embedding_vector column and rebuild the vector search index
B) Increase the vector search index replication factor from 1 to 3 to distribute query load
C) Reduce the number of retrieved chunks from 10 to 5 and implement a two-stage retrieval with re-ranking
D) Cache the vector search index in memory using Databricks SQL warehouse photon acceleration

---

**Q21.** An application generates structured data from unstructured text using an LLM with JSON mode enabled via the OpenAI SDK. The schema requires nested objects with strict validation rules. In production, approximately 15% of LLM responses fail JSON parsing despite the prompt explicitly requesting valid JSON. Which solution BEST ensures schema compliance?

A) Increase the temperature parameter to 0.0 to make outputs deterministic and add 'You must return valid JSON' to the user prompt
B) Use function calling with a defined JSON schema instead of JSON mode, which provides stronger schema enforcement
C) Add a post-processing step that attempts to fix malformed JSON using regex patterns before parsing
D) Implement Pydantic models with strict validation, pass the schema to the LLM in the system prompt, and use instructor library for guided generation with automatic retry logic

---

**Q22.** A GenAI application processes sensitive customer data and uses MLflow to track experiments. Compliance requires that all PII in experiment artifacts must be encrypted at rest with customer-managed keys and all model predictions must be logged with user attribution for audit purposes. The application uses Unity Catalog for model registry. Which combination of configurations satisfies these requirements?

A) Store MLflow artifacts in an encrypted S3 bucket with SSE-KMS using customer keys, enable MLflow Model Registry webhook notifications, and use Delta Live Tables with APPLY CHANGES to track prediction events
B) Enable Unity Catalog encryption at rest with customer-managed keys, configure MLflow to log to Unity Catalog volumes, and implement custom logging in the model's predict method to write predictions to an audit Delta table with MANAGED storage
C) Configure workspace-level encryption with customer-managed keys for DBFS, use MLflow autologging with PII masking transformers, and enable Unity Catalog audit logging at the catalog level
D) Use Unity Catalog external volumes with customer-managed encryption keys for artifact storage, log predictions as MLflow metrics with user_id tags, and query system.access.audit for compliance reports

---

**Q23.** A chatbot application uses LangChain with a custom memory implementation that stores conversation history in a Delta table partitioned by session_id. Users report that after 20 conversation turns, the chatbot starts forgetting context from earlier in the session despite the memory retrieval code successfully fetching all messages. The LLM's context window is 8,192 tokens. Which issue is MOST likely causing this behavior?

A) The conversation history exceeds the LLM's context window limit, and older messages are being truncated by the LLM API despite being retrieved from memory
B) The Delta table lacks a Z-ordering optimization on session_id and timestamp columns, causing slow retrieval of older messages that timeout before reaching the LLM
C) LangChain's default ConversationBufferMemory is being used instead of ConversationSummaryMemory, causing all messages to be stored without compression
D) The memory retrieval code uses LIMIT 20 in the SQL query, which caps the number of retrieved messages regardless of session length

---

**Q24.** A data scientist is developing a multi-agent system where specialized agents (research, analysis, writing) collaborate to generate reports. Each agent is implemented as a separate MLflow PyFunc model with different dependencies. The orchestration logic determines which agents to invoke based on task requirements. Which design pattern BEST implements this in Databricks while maintaining model versioning and dependency isolation?

A) Create a single meta-model PyFunc that loads all agent models in the load_context method and implements orchestration logic in predict, using conda environments for dependency isolation
B) Package all agents into a single Docker container with a shared virtual environment, register as one model, and use environment variables to switch between agent behaviors
C) Register each agent as a separate model in Unity Catalog with its own conda environment, deploy each to a Model Serving endpoint, and implement orchestration in a Databricks workflow that calls endpoints via REST API
D) Implement each agent as a Python class, register a single orchestrator model that dynamically imports agent classes, and use %pip install in model notebooks to manage dependencies

---

**Q25.** A GenAI application uses a custom MLflow model that wraps a Hugging Face pipeline for text generation. The model logs show that inference latency increased from 200ms to 3000ms after deploying to a Model Serving endpoint with GPU acceleration disabled. The development team needs to reduce latency to under 500ms while maintaining the same model architecture. Which solution will MOST effectively meet this requirement?

A) Enable auto-scaling on the Model Serving endpoint to spawn additional CPU instances during peak load
B) Implement response caching in the application layer using Redis to store frequently requested completions
C) Redeploy the model using Databricks Feature Serving instead of Model Serving to leverage pre-computed embeddings
D) Modify the serving endpoint configuration to use GPU_MEDIUM instances and update the model signature to specify batch inference

---

**Q26.** A RAG application retrieves context from a Vector Search index built on Unity Catalog tables. The application uses langchain with DatabricksVectorSearch as the retriever. During production testing, 30% of queries return incorrect context despite relevant documents existing in the source tables. Investigation shows that the embedding model (gte-large) used during ingestion differs from the query embedding model (all-MiniLM-L6-v2) configured in the application code. The team needs to fix context retrieval without rebuilding the Vector Search index. Which approach will resolve the issue?

A) Increase the k parameter in the similarity_search call from 4 to 20 to retrieve more candidates and compensate for embedding mismatch
B) Update the application's DatabricksVectorSearch retriever configuration to use gte-large for query embeddings to match the index embedding model
C) Enable hybrid search by combining Vector Search with keyword-based BM25 retrieval using Databricks SQL queries
D) Modify the Vector Search index settings to normalize all embedding vectors to unit length before similarity computation

---

**Q27.** A GenAI application deployed using Mosaic AI Model Serving is experiencing intermittent timeout errors during peak usage hours. The model inference endpoint has autoscaling enabled with min_workers=1 and max_workers=10, but CloudWatch metrics show queue depth spiking to 500+ requests. The application requires sub-2-second response times. Which configuration change would BEST resolve this issue?

A) Increase max_workers to 25 and reduce scale-down-delay to 30 seconds
B) Enable request batching with max_batch_size=32 and max_wait_time=100ms, and set min_workers=3
C) Switch to provisioned throughput mode with 15 dedicated workers and disable autoscaling
D) Implement client-side retry logic with exponential backoff and increase endpoint timeout to 10 seconds

---

**Q28.** A team needs to deploy a RAG application that queries a 50GB vector index stored in Databricks Vector Search. The application must support A/B testing of two different embedding models while maintaining separate vector indices. Deployment requires rollback capability within 30 seconds if error rates exceed 2%. Which deployment architecture BEST meets these requirements?

A) Deploy both models as separate Model Serving endpoints with traffic splitting configured at 50/50, using Delta Live Tables for real-time index updates
B) Deploy both embedding models to the same GPU-enabled endpoint, create two separate vector search indices, and implement feature flag-based routing in the application layer with automated rollback triggered by custom metrics
C) Use MLflow Model Registry with alias-based routing ('champion'/'challenger'), deploy a single endpoint with conditional logic to query the appropriate vector index based on request metadata
D) Implement blue-green deployment using Databricks Jobs with two complete application stacks, switching traffic via DNS routing when promoting the challenger model

---

**Q29.** An organization is deploying a multi-agent LLM application using LangGraph that orchestrates three specialized agents (SQL generation, document retrieval, and response synthesis). The application must maintain conversation state across user sessions, support concurrent users, and comply with data residency requirements that mandate all conversation history remains within the Databricks workspace. Which state management approach is MOST appropriate?

A) Implement Redis cluster deployed on Databricks clusters for session state, with periodic snapshots to DBFS for disaster recovery
B) Use LangGraph's built-in MemorySaver with checkpointing to Delta tables, configuring Unity Catalog volumes for persistent storage with automated lifecycle policies
C) Store conversation state in MLflow tracking experiments with custom tags for session management, using MLflow's built-in search APIs for state retrieval
D) Use Databricks SQL with a dedicated conversation_history table, implementing row-level security and query optimization with liquid clustering on session_id

---

**Q30.** A production GenAI application uses a fine-tuned Llama 3.1 70B model served via Mosaic AI Model Serving. After deployment, the application experiences P95 latency of 8 seconds, exceeding the 3-second SLA. Token generation metrics show 12 tokens/second throughput per request. The endpoint uses g5.12xlarge instances (4x A10 GPUs) with tensor parallelism enabled. Which optimization would provide the GREATEST latency reduction?

A) Implement speculative decoding with a Llama 3.1 8B draft model and optimize KV-cache memory allocation
B) Implement prompt caching for common instruction prefixes and enable continuous batching with max_batch_size=8
C) Switch to g5.48xlarge instances (8x A10 GPUs) and increase tensor parallelism degree from 4 to 8
D) Apply 4-bit quantization (GPTQ) to reduce model memory footprint and enable larger batch sizes

---

**Q31.** A financial services company needs to deploy a GenAI application that generates investment advice based on customer data. The application must log all model inputs/outputs for regulatory audit, implement real-time PII detection with automatic redaction before logging, and support model version rollback within 5 minutes if compliance violations are detected. Which deployment pattern BEST addresses these requirements?

A) Use Model Serving with pre-processing and post-processing functions that call Databricks SQL UDFs for PII detection, write audit logs to event-driven Delta tables, and implement blue-green deployment for version control
B) Implement a custom FastAPI wrapper with Presidio for PII detection, write structured logs to Unity Catalog volumes, use Databricks Workflows to orchestrate model version deployment with Git-based rollback
C) Deploy the model with Mosaic AI Gateway configured with custom PII detection middleware, audit logging to Unity Catalog tables with Delta sharing for compliance teams, and MLflow Model Registry stage transitions for rollback
D) Deploy via Model Serving with inference tables enabled for automatic logging, use lakehouse monitoring with custom PII detection metrics, and leverage Model Serving endpoint configuration updates for rollback

---

**Q32.** A GenAI application uses a compound AI system with three components: (1) embedding model for vector search, (2) reranker model for result refinement, and (3) LLM for response generation. The system must minimize total cost while maintaining P99 latency under 5 seconds for 1000 requests/hour with bursty traffic patterns (80% of requests in 20% of time). Which resource allocation strategy is MOST cost-effective?

A) Deploy all three models on a single Model Serving endpoint with shared GPU resources and autoscaling from 0 to 5 instances
B) Use serverless endpoints for the embedding and reranker models, and a provisioned GPU endpoint with min_workers=2 for the LLM
C) Deploy embedding and reranker on CPU-based Model Serving with autoscaling, and use a provisioned GPU endpoint with min_workers=1 for the LLM with scale-up policy targeting queue depth
D) Use Databricks SQL warehouse serverless compute for embedding/reranker inference via UDFs, and Foundation Model APIs for the LLM with pay-per-token pricing

---

**Q33.** A team is deploying a compound AI system that chains three models: a fine-tuned embedding model, a retrieval component querying a vector database, and a foundation model for generation. The system must handle 500 concurrent requests with strict latency requirements (p95 < 2 seconds). During load testing, the vector database becomes the bottleneck. Which architectural change BEST addresses this bottleneck while maintaining cost efficiency?

A) Deploy the vector database on a separate cluster with photon acceleration and implement connection pooling with client-side result caching for frequent queries
B) Replace the vector database with a Databricks Feature Store and increase the foundation model serving endpoint to GPU-accelerated instances
C) Implement a caching layer using MLflow Model Registry artifacts and enable model endpoint autoscaling with provisioned throughput on the foundation model
D) Migrate the embedding model to CPU-only serving and increase vector database query parallelism by sharding the index across multiple nodes

---

**Q34.** An organization deploys a GenAI application using Model Serving with a provisioned throughput endpoint. After deployment, they notice the endpoint frequently returns 429 (Too Many Requests) errors during peak hours despite staying within the provisioned throughput limit of 10,000 tokens/minute. Monitoring shows actual token consumption averaging 7,500 tokens/minute during these periods. What is the MOST likely cause of these 429 errors?

A) The model's context window is being exceeded by individual requests, causing the endpoint to reject requests to prevent token limit violations
B) Databricks Model Serving enforces a separate character-based rate limit in addition to token limits, and the requests are exceeding the character limit
C) The endpoint's concurrent request limit (default 10 concurrent requests) is being exceeded, causing queueing and subsequent rate limiting even though total token throughput is under the provisioned limit
D) The provisioned throughput is measured per workspace rather than per endpoint, and other endpoints in the same workspace are consuming the remaining token budget

---

**Q35.** A company needs to deploy a multi-stage GenAI application where Stage 1 (classification model) routes requests to one of three Stage 2 specialist models based on category. The application must maintain consistent routing decisions for the same input across multiple requests and support A/B testing new routing logic. Which deployment architecture BEST satisfies these requirements?

A) Deploy all models as separate Model Serving endpoints and implement routing logic in a Databricks Job with Delta Live Tables for maintaining routing state
B) Deploy the classification model as a Model Serving endpoint and implement routing logic in a custom Python application using MLflow model flavors, with routing decisions stored in Unity Catalog volumes for persistence
C) Create a compound system using MLflow Deployments API where the classification model output triggers downstream model calls, with routing consistency managed via MLflow experiment tracking and model aliases for A/B testing
D) Use MLflow AI Gateway with custom routing rules defined in gateway configuration, backed by a Delta table storing request hashes and routing decisions for consistency

---

**Q36.** During deployment of a RAG application to production, the team discovers that the embedding model (deployed via Model Serving) and the vector database queries both require the same preprocessing steps on input text (lowercasing, special character removal, tokenization). Currently, preprocessing is duplicated in both the embedding endpoint code and the retrieval service code. To reduce maintenance burden and ensure consistency, where should this shared preprocessing logic be centralized?

A) Package the preprocessing logic as a custom MLflow PyFunc model flavor and deploy it as a separate preprocessing endpoint that both services call before their respective operations
B) Implement preprocessing as a reusable Python package, register it in a Unity Catalog volume, and import it in both the embedding model's model.py and the retrieval service code
C) Define preprocessing as a Spark UDF in a shared notebook and call it via Databricks Connect from both the embedding endpoint and retrieval service
D) Store preprocessing logic in an MLflow experiment as a logged artifact and load it dynamically at runtime in both services using the MLflow Tracking API

---

**Q37.** A financial services company is deploying a RAG application that processes customer financial documents containing PII. The company's compliance team requires that all prompt inputs and LLM outputs be logged with user attribution for audit purposes, and that any prompts containing SSNs be automatically blocked before reaching the model. The data science team wants to track token usage per department for cost allocation. Which combination of Databricks features provides the MOST complete solution?

A) Enable Lakehouse Monitoring on the vector database tables and use Unity Catalog data lineage to track prompt origins
B) Implement custom logging in the application code to write prompts to Delta tables and use Databricks SQL warehouses for usage reporting
C) Use MLflow prompt engineering UI with experiment tracking and enable Unity Catalog audit logs for the serving endpoint
D) Configure AI Gateway with inference tables enabled, PII detection rules in the request policy, and tag-based cost tracking through workspace tags

---

**Q38.** An enterprise has a Model Serving endpoint running a fine-tuned Llama model in production. The security team mandates that the model weights must remain within the organization's Azure tenant and that all inference traffic must be encrypted with customer-managed keys. The endpoint currently uses Databricks-managed infrastructure. Which configuration change is REQUIRED to meet these security requirements?

A) Switch the endpoint to use Model Serving with Customer-Managed Keys and deploy to a customer-managed compute cluster
B) Enable Private Link for the workspace and configure the endpoint to use a customer-managed VPC
C) Migrate the endpoint to External Models configuration pointing to an Azure ML deployment with CMK enabled
D) Register the model in Unity Catalog with a managed storage location using customer-managed encryption keys

---

**Q39.** A healthcare organization uses Unity Catalog to manage access to a vector database containing embeddings generated from patient medical records. The governance team needs to ensure that: (1) data scientists can query the vector table for similarity search during model development, (2) production applications can read embeddings but cannot see the original source document IDs that link back to patient records, and (3) all access attempts are logged with the accessing user's identity. The vector table schema includes columns: id (patient record ID), embedding (vector), metadata (JSON with diagnosis codes). What is the MOST secure approach?

A) Grant SELECT on the vector table to data scientists, create a view excluding the id column for production apps, and enable Unity Catalog audit logging on both objects
B) Create two separate vector tables (one with id, one without), use Unity Catalog tags to mark the id-containing table as sensitive, and use attribute-based access control to restrict production service principals
C) Use column-level security to mask the id column for production service principals while granting full SELECT to data scientists, and enable predictive optimization for audit tracking
D) Implement row-level security filters based on user groups, encrypt the id column using Databricks encryption functions, and use Delta Lake change data feed for audit logging

---

**Q40.** A company is deploying a compound AI system with multiple chained LLM calls through Databricks Model Serving. The risk management team requires tracking of: (1) total cost per end-user request across all LLM calls in the chain, (2) detection of requests that generate outputs flagged as toxic by a safety classifier, and (3) the ability to reproduce the exact model versions used for any request within the last 90 days for compliance investigations. The system uses a mix of provisioned throughput and pay-per-token endpoints. Which monitoring strategy BEST meets these requirements?

A) Enable inference tables on all endpoints, use a Databricks workflow to join tables and calculate per-request costs, implement a toxicity detection model as a post-processing step that writes flags to a monitoring table, and use MLflow model versioning with automatic model archival after 90 days
B) Use MLflow Tracing to capture the full request chain with automatic cost tracking, deploy a guardrail model in parallel to the main serving endpoints that evaluates outputs and logs to inference tables, and tag endpoint models with Git commit hashes for version reproduction
C) Configure AI Gateway to route all requests, enable request/response logging with custom request ID propagation across chain calls, use AI Gateway's toxicity filter in the response policy, and configure endpoint versions to be immutable with 90-day retention in Unity Catalog
D) Implement custom metadata logging in the application code with a unique trace ID, aggregate costs using Databricks SQL queries against system tables, run batch toxicity detection jobs on inference table outputs, and maintain endpoint configuration history in a Delta table

---

**Q41.** A company is monitoring a RAG application in production that uses MLflow to track retrieval quality metrics. The data science team notices that while the average relevance score is 0.82, certain user queries consistently return irrelevant documents. They want to identify which specific query patterns are failing without manually reviewing thousands of logs. Which monitoring approach would BEST help isolate the problematic query patterns?

A) Set up conditional logging in MLflow that only captures full context when relevance scores fall below 0.7, then use SQL queries on the logged data to aggregate by query semantic categories
B) Configure MLflow to log all queries and use cluster analysis on embedding vectors to identify outlier query groups, then manually review representative samples from each cluster
C) Create a custom metric in MLflow that tracks relevance score variance across query types, and set up alerts when variance exceeds threshold values
D) Enable MLflow autologging with increased sampling rate and use the built-in anomaly detection to flag queries with relevance scores more than 2 standard deviations below mean

---

**Q42.** An LLM application using GPT-4 for customer support ticket summarization is being evaluated for production readiness. The evaluation dataset contains 500 ticket-summary pairs with human-generated reference summaries. The team needs to detect when the model hallucinates information not present in the original ticket. Which evaluation strategy would provide the MOST reliable detection of hallucinations at scale?

A) Use an LLM-as-judge approach with GPT-4 to compare generated summaries against original tickets and score for factual consistency on a 1-5 scale
B) Implement NLI (Natural Language Inference) model to classify each sentence in the summary as entailed, neutral, or contradicted by the source ticket, flagging contradictions and neutral statements as potential hallucinations
C) Calculate ROUGE-L scores between generated and reference summaries, setting a threshold below which summaries are flagged for manual hallucination review
D) Use semantic similarity between generated summary embeddings and source ticket embeddings, with low similarity scores indicating hallucination

---

**Q43.** A financial services company deployed a multi-agent LLM system where Agent A retrieves portfolio data, Agent B performs risk analysis, and Agent C generates investment recommendations. In production, they observe that 8% of recommendations are flagged as inconsistent with the risk analysis. They need to implement monitoring to identify which agent is causing the inconsistency. What monitoring architecture would BEST enable root cause isolation?

A) Implement end-to-end integration tests that run the full agent pipeline on synthetic data every hour and alert when output consistency drops below 92%
B) Use MLflow to track the final recommendation quality metric and train a gradient boosting model to predict which agent likely caused each failure based on historical patterns
C) Log input/output pairs at each agent boundary with correlation IDs, implement schema validation on inter-agent data contracts, and create metrics dashboards showing validation failure rates per agent
D) Deploy a separate evaluation LLM that reviews the complete conversation history for inconsistent recommendations and uses chain-of-thought prompting to identify the faulty agent

---

**Q44.** A RAG system serving legal document Q&A is experiencing drift where newer court rulings added to the vector database are not being retrieved effectively, despite having high semantic relevance. The retrieval evaluation shows that precision@5 has dropped from 0.78 to 0.64 over three months while the vector store grew from 10K to 45K documents. Which action would MOST effectively diagnose whether the issue is embedding drift versus retrieval configuration?

A) Create a test set of queries specifically about recent rulings, measure retrieval performance on recent vs old documents separately, and analyze whether temporal bias exists in retrieved results
B) Re-embed the entire corpus using the same embedding model and compare retrieval metrics between old and new embeddings to isolate embedding quality from retrieval configuration
C) Perform ablation testing by adjusting the number of retrieved chunks (k) from 5 to 10, 20, and 50 to determine if the issue is insufficient retrieval depth rather than relevance
D) Generate synthetic queries using the LLM based on new document content, then measure whether these synthetic queries successfully retrieve their source documents

---

**Q45.** An organization is implementing Model Serving for a fine-tuned LLM on Databricks and needs to monitor for prompt injection attacks in production. The security team requires detection with <100ms latency impact per request. The model processes 500 requests per minute. Which monitoring implementation would BEST meet the security requirement while maintaining performance constraints?

A) Deploy a separate BERT-based prompt injection classifier model in parallel with the main LLM, using asynchronous logging if classification exceeds latency budget, and triggering alerts on detected attacks
B) Implement rule-based detection using regex patterns and keyword matching for common injection patterns in the model serving endpoint before forwarding to the LLM, logging matches synchronously
C) Configure the model serving endpoint to sample 10% of requests and send them to a GPT-4 based injection detector, then extrapolate detection rates and alert on anomalies
D) Enable Databricks Model Serving built-in security scanning which automatically detects prompt injections using ML-based classification with guaranteed <50ms overhead

---

