## Chapter 9: Evaluation and Monitoring of GenAI Applications — Practice Questions

**Q1.** A team is evaluating their RAG application using mlflow.evaluate(). They have a dataset with columns: question, ground_truth_answer, retrieved_context, and generated_answer. They want to measure if the model's answers contain hallucinations. Which metric should they prioritize?

A) Answer relevance, because it checks if the answer addresses the question
B) Faithfulness, because it verifies the answer only uses information from retrieved_context
C) Context precision, because it ensures relevant chunks are ranked first
D) Context recall, because it measures coverage of all relevant chunks

<details>
<summary>Answer</summary>

**Correct: B**

Faithfulness specifically measures whether the generated answer contains only information present in the retrieved context, directly detecting hallucinations. This is the primary anti-hallucination metric.

**Why the others are wrong:**
- Answer relevance checks if the answer addresses the question but doesn't verify if facts are grounded in context
- Context precision evaluates retrieval quality (ranking), not whether the answer hallucinates
- Context recall evaluates retrieval coverage, not generation hallucination

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A company's RAG chatbot is deployed to production using Databricks Model Serving. The team needs to monitor for degradation in answer quality and detect drift in user queries over time. Which combination provides the BEST solution?

A) Enable inference tables to log requests/responses to Delta, then use Lakehouse Monitoring to track metrics and drift on that table
B) Use mlflow.evaluate() on a static test dataset daily and alert if scores drop
C) Set up CloudWatch alarms on API latency and error rate metrics
D) Sample production traffic weekly and manually review responses in a spreadsheet

<details>
<summary>Answer</summary>

**Correct: A**

Inference tables automatically log all Model Serving requests/responses to Delta tables. Lakehouse Monitoring can then track quality metrics, data drift (query patterns), and model drift continuously on this data. This is the native Databricks solution for production monitoring.

**Why the others are wrong:**
- Static test datasets don't capture real production distribution shift or actual user queries
- CloudWatch tracks infrastructure metrics but not GenAI-specific quality metrics like faithfulness or relevance
- Manual sampling is not scalable, misses drift between samples, and lacks automated alerting

</details>

---

**Q3.** An engineering team runs mlflow.evaluate() with an LLM-as-a-judge metric using GPT-4 to score answer correctness. The evaluation completes but costs are higher than expected. What is the PRIMARY cost driver?

A) The Delta table storage costs for the evaluation dataset
B) The API calls to GPT-4 for scoring each answer in the evaluation dataset
C) The Databricks cluster compute costs for running mlflow.evaluate()
D) The MLflow tracking server storage for logging evaluation results

<details>
<summary>Answer</summary>

**Correct: B**

LLM-as-a-judge metrics make API calls to a strong LLM (GPT-4, DBRX) for each example in the dataset. For large datasets, these API calls dominate costs. GPT-4 API costs are per-token and expensive.

**Why the others are wrong:**
- Delta storage for evaluation datasets is minimal compared to LLM API costs
- Cluster compute for running mlflow.evaluate() is relatively cheap; the heavy cost is external API calls
- MLflow tracking storage is negligible compared to GPT-4 API call volume

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A RAG system retrieves 10 chunks per query. Context recall is measured at 0.6 while context precision is 0.9. What does this indicate?

A) The system retrieves most relevant chunks but ranks irrelevant chunks too high
B) The system has high faithfulness but low answer relevance
C) The system retrieves too many chunks and should reduce to 6 chunks
D) The system ranks relevant chunks well but misses 40% of relevant chunks that should be retrieved

<details>
<summary>Answer</summary>

**Correct: D**

Context recall = 0.6 means only 60% of relevant chunks are retrieved (missing 40%). Context precision = 0.9 means the retrieved chunks are well-ranked (relevant ones first). The issue is coverage, not ranking.

**Why the others are wrong:**
- Low precision would mean irrelevant chunks are ranked high; here precision is high (0.9)
- Faithfulness and answer relevance measure generation quality, not retrieval; these are retrieval metrics
- The number 6 is arbitrary; the metric indicates poor recall, not that chunk count should equal 0.6×10

</details>

---

**Q5.** A team wants to implement online evaluation of their production GenAI app. They currently have offline evaluation with mlflow.evaluate() running on a static test set. What additional step enables online evaluation?

A) Re-run mlflow.evaluate() daily on the same test set to track metric trends
B) Increase the size of the static test dataset to 10,000 examples
C) Enable inference tables on Model Serving, sample logged production traffic, and run mlflow.evaluate() on sampled real requests/responses
D) Deploy the model to multiple regions and compare metrics across regions

<details>
<summary>Answer</summary>

**Correct: C**

Online evaluation means evaluating real production traffic, not static datasets. Inference tables log production requests/responses. Sampling this data and running mlflow.evaluate() on it provides online evaluation of actual user interactions.

**Why the others are wrong:**
- Re-running on the same static test set is still offline evaluation; it doesn't capture production distribution
- Larger static datasets are still offline evaluation; they don't reflect actual production queries
- Multi-region deployment tests infrastructure, not online evaluation of real user traffic quality

</details>

---

**Q6.** A company is choosing between two models for their RAG chatbot: a 7B parameter model with 200ms latency and $0.001 per 1K tokens, versus a 70B parameter model with 800ms latency and $0.02 per 1K tokens. The 70B model has 5% higher answer correctness. Which scenario BEST justifies the 70B model?

A) Internal HR policy Q&A with 50 queries per day where accuracy is critical and budget is not constrained
B) High-volume customer support with 10,000 queries per day where response time under 500ms is critical
C) Real-time chatbot for website visitors where sub-second response is required for user experience
D) Automated email response system processing 100,000 emails per day with tight cost constraints

<details>
<summary>Answer</summary>

**Correct: A**

Low query volume (50/day) means the 20× cost difference is affordable. No latency constraint mentioned, and accuracy is critical. This justifies the higher cost and latency for better accuracy.

**Why the others are wrong:**
- 10,000 queries/day × 20× cost difference = significant budget impact; 800ms exceeds 500ms latency requirement
- Sub-second latency requirement rules out 800ms model regardless of accuracy
- 100,000 emails/day × 20× cost = $2000/day cost increase, conflicts with tight cost constraints

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** When using mlflow.evaluate() for a RAG application, which columns are REQUIRED in the evaluation dataset to compute faithfulness using LLM-as-a-judge?

A) question and ground_truth only
B) generated_answer and ground_truth only
C) question, generated_answer, and retrieved_context
D) question, generated_answer, ground_truth, and retrieved_context

<details>
<summary>Answer</summary>

**Correct: C**

Faithfulness checks if generated_answer only uses information from retrieved_context. The LLM judge needs both to compare. Question provides context for the judge. Ground truth is not required for faithfulness (it's for answer correctness).

**Why the others are wrong:**
- Missing generated_answer and retrieved_context which are essential for faithfulness scoring
- Faithfulness requires retrieved_context to verify grounding; ground_truth is not needed
- Ground_truth is needed for answer correctness, not faithfulness; faithfulness only needs answer and context

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A team observes that their RAG application's context precision metric dropped from 0.85 to 0.60 over two weeks in production, while context recall remained stable at 0.75. What is the MOST likely cause?

A) The embedding model has degraded and is retrieving fewer total chunks
B) User queries have become more complex and require more context
C) The retrieval system is now returning more irrelevant chunks ranked higher than relevant chunks
D) The generation model is producing lower quality answers

<details>
<summary>Answer</summary>

**Correct: C**

Context precision measures ranking quality - are relevant chunks ranked first? A drop from 0.85 to 0.60 while recall stays stable means irrelevant chunks are now ranked higher, degrading precision. This could be from vector index drift or embedding model issues affecting similarity scoring.

**Why the others are wrong:**
- Fewer chunks would affect recall (coverage), not precision (ranking); recall is stable
- Complex queries might reduce recall if system can't find all relevant chunks, but precision measures ranking of what IS retrieved
- Generation quality affects faithfulness and answer relevance, not retrieval metrics like context precision

</details>

---

