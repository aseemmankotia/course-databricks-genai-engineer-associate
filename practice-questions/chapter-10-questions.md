## Chapter 10: Full Practice Exam and Test-Taking Strategy — Practice Questions

**Q1.** A company is deploying a RAG chatbot that must respond within 200ms and handle 1000 concurrent users during peak hours. The solution must minimize cost while meeting these requirements. Which deployment approach BEST meets these needs?

A) Deploy the embedding model and LLM on separate Model Serving endpoints with GPU_SMALL compute, enable autoscaling with min instances = 0
B) Deploy both models on a single Model Serving endpoint with GPU_MEDIUM compute, enable autoscaling with min instances = 2
C) Deploy the embedding model on CPU_SMALL with min instances = 1, deploy the LLM on GPU_SMALL with min instances = 1, both with autoscaling enabled
D) Use Databricks Foundation Model API for both embedding and generation, no dedicated endpoints needed

<details>
<summary>Answer</summary>

**Correct: C**

This approach balances latency and cost: embedding models are lightweight and work well on CPU, keeping min instances = 1 ensures sub-200ms response (no cold start), LLM needs GPU but only 1 min instance saves cost, autoscaling handles 1000 concurrent users during peaks

**Why the others are wrong:**
- Min instances = 0 means cold starts of 30-60 seconds, violating the 200ms requirement. GPU_SMALL for embeddings wastes money
- Single endpoint couples scaling of two different workloads, GPU_MEDIUM is oversized for embeddings, min = 2 for both is unnecessarily expensive during off-peak
- Foundation Model API has higher latency (external call) and doesn't guarantee 200ms, no control over scaling or compute allocation

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An engineer is evaluating a text classification model. The model achieves 92% accuracy but stakeholders report it frequently misclassifies the minority class (10% of data). The exam shows precision=0.95, recall=0.45 for the minority class. Which metric should be prioritized to improve performance on the minority class?

A) Increase overall accuracy by tuning the decision threshold
B) Optimize for recall by adjusting class weights or resampling techniques
C) Improve precision by filtering low-confidence predictions
D) Focus on F1-score which balances precision and recall equally

<details>
<summary>Answer</summary>

**Correct: B**

Recall = 0.45 means the model only identifies 45% of minority class instances (high false negatives). For minority class problems, increasing recall is critical. Class weights or resampling (SMOTE, oversampling) directly address class imbalance

**Why the others are wrong:**
- Accuracy is already high (92%) but misleading due to class imbalance; threshold tuning won't fix the fundamental imbalance issue
- Precision is already high (0.95); filtering would reduce recall further, making the minority class problem worse
- F1-score is useful for monitoring improvement, but doesn't specify the action needed. The problem specifically requires increasing recall, not balancing metrics

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A GenAI application uses Vector Search with a 768-dimension embedding model. After deploying to production, query latency increases from 50ms to 800ms when the index reaches 5 million vectors. Which action will MOST effectively reduce query latency?

A) Switch from Delta Sync to Direct Vector Access index type
B) Reduce embedding dimensions from 768 to 384 using PCA or a smaller model
C) Increase the number of index replicas and enable query routing
D) Change the similarity metric from cosine to dot product

<details>
<summary>Answer</summary>

**Correct: C**

At 5M vectors, query load is the bottleneck. Increasing replicas distributes query load across multiple index copies, reducing per-replica load and latency. Query routing automatically sends queries to the least-loaded replica

**Why the others are wrong:**
- Delta Sync vs Direct Vector Access affects write performance and data freshness, not query latency on an existing large index
- Reducing dimensions improves query speed marginally (~20-30%) but requires retraining the embedding model and reindexing all 5M vectors—massive operational overhead for uncertain gain
- Similarity metric change has negligible impact on latency; cosine and dot product have similar computational complexity for normalized vectors

</details>

---

**Q4.** A data scientist is building a RAG system and must choose between storing documents in a Delta table with Vector Search versus using an external vector database. The system must support real-time document updates, Unity Catalog governance, and integrate with existing Databricks notebooks. Which solution BEST meets these requirements?

A) Use external vector database (Pinecone) with Unity Catalog external location for governance
B) Use Direct Vector Access index with manual sync from Delta table to external storage
C) Store embeddings in Delta table, use external vector database for queries only
D) Use Delta table with Vector Search (Delta Sync index) for unified governance and real-time sync

<details>
<summary>Answer</summary>

**Correct: D**

Delta Sync index automatically syncs changes from Delta table to Vector Search index, supports Unity Catalog permissions natively, and integrates seamlessly with Databricks notebooks. Meets all requirements without external dependencies

**Why the others are wrong:**
- External databases don't support Unity Catalog governance directly; external location only provides storage access, not query-level permissions or lineage
- Direct Vector Access requires manual sync code and doesn't provide real-time updates automatically; adds complexity without benefit
- Split architecture (Delta for storage, external for queries) creates governance gaps and sync complexity; Unity Catalog can't enforce permissions on external database queries

</details>

---

**Q5.** An MLflow experiment tracks a fine-tuned LLM with the following logged items: model weights (2.5GB .safetensors file), training metrics (loss, perplexity), hyperparameters (learning_rate=1e-5, batch_size=8), and a text file of sample outputs. The engineer needs to deploy this model to Model Serving. Which MLflow component is REQUIRED for deployment?

A) All metrics and parameters must be logged to create a valid deployment
B) The model must be logged with the 'transformers' flavor to be deployable
C) The model artifact must include a conda.yaml environment file
D) Sample outputs must be logged as MLflow artifacts for validation

<details>
<summary>Answer</summary>

**Correct: B**

Model Serving requires a recognized MLflow flavor (transformers, sentence_transformers, langchain, openai, etc.) to know how to load and serve the model. The flavor defines the interface and serving behavior

**Why the others are wrong:**
- Metrics and parameters are for tracking/comparison only; Model Serving only needs the model artifact and its flavor
- conda.yaml is auto-generated when using mlflow.transformers.log_model(); manually creating it is not required and Model Serving uses containerized environments
- Sample outputs are useful for evaluation but not required for deployment; Model Serving needs the model artifact and serving configuration

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A team is preparing training data for a fine-tuning job. The dataset contains 10,000 customer support conversations. After reviewing the data, they find: 60% of conversations are greetings/small talk, 25% are product questions, 15% are technical troubleshooting. The fine-tuning goal is to improve technical troubleshooting responses. Which data preparation strategy is MOST effective?

A) Use all 10,000 conversations to maximize training data volume
B) Oversample the technical troubleshooting examples to balance the three categories equally
C) Filter to only the 1,500 technical troubleshooting conversations, augment to 3,000 examples using paraphrasing
D) Use the 2,500 product question examples plus 1,500 technical examples, exclude greetings

<details>
<summary>Answer</summary>

**Correct: C**

Fine-tuning works best with focused, high-quality data aligned to the target task. Filtering to relevant examples (technical troubleshooting) and augmenting to 3,000 provides sufficient volume without dilution. Greetings/small talk add noise and bias the model away from the target behavior

**Why the others are wrong:**
- Including 60% irrelevant data (greetings) biases the model toward generic responses, reducing performance on the actual target task
- Simply oversampling (duplicating) creates memorization risk and doesn't add new signal; equal balancing includes irrelevant categories
- Product questions are adjacent but not the target; including them dilutes focus. The model should excel at technical troubleshooting, not be mediocre at everything

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A Databricks workspace has Unity Catalog enabled. An engineer needs to grant a data science team access to query a vector search index 'prod_catalog.rag_schema.doc_embeddings_index' and read the underlying Delta table, but NOT modify the index or table. Which permission set is MINIMALLY sufficient?

A) SELECT on catalog and schema, READ on table, QUERY on index
B) USAGE on catalog and schema, SELECT on table, no index-specific permission needed
C) ALL_PRIVILEGES on catalog, USAGE on schema, SELECT on table and index
D) USAGE on catalog and schema, SELECT on table, USAGE on index

<details>
<summary>Answer</summary>

**Correct: D**

Unity Catalog requires: USAGE on catalog and schema to navigate the namespace, SELECT on the Delta table to read data, and USAGE on the index to query it. This is the minimal set for read-only access

**Why the others are wrong:**
- SELECT is a table/view permission, not valid for catalogs/schemas. READ is not a Unity Catalog permission type. QUERY is not a valid index permission
- Querying a Vector Search index requires explicit index-level USAGE permission; table permissions alone are insufficient
- ALL_PRIVILEGES grants modify permissions (CREATE, MODIFY, DELETE) which violates the requirement for read-only access

</details>

---

**Q8.** During a practice exam, a candidate has 30 minutes remaining and 15 questions left to answer. Five questions are flagged for review (already answered but uncertain). Which time management strategy should the candidate follow?

A) Spend 2 minutes per remaining question (10 questions × 2 min = 20 min), then review flagged questions with remaining time
B) Review all 5 flagged questions first (10 minutes), then answer remaining 10 questions at 2 min each
C) Answer all 10 unanswered questions quickly (1 min each), spend remaining 20 minutes on flagged questions
D) Skip to the end to review flagged questions first, then return to unanswered questions if time permits

<details>
<summary>Answer</summary>

**Correct: A**

Unanswered questions give 0 points; answered-but-uncertain questions might be correct. Prioritize answering all questions first (ensures maximum possible score), then use remaining time to improve flagged answers. 2 min/question is the target pace

**Why the others are wrong:**
- Reviewing flagged questions first risks running out of time for unanswered questions, leaving guaranteed 0-point items
- 1 min per question is too rushed and increases error rate; better to answer methodically and leave fewer flagged questions
- Skipping to flagged questions first is the worst strategy—if time runs out, unanswered questions guarantee lost points while flagged questions might already be correct

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

