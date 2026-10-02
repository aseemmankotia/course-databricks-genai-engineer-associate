## Chapter 7: Foundation Model APIs and Production Serving — Practice Questions

**Q1.** A fintech startup is building a customer service chatbot expected to handle 50-200 requests per minute during business hours and near-zero traffic at night. They are prototyping with Databricks Foundation Model API using DBRX Instruct. Which pricing model should they choose for production deployment?

A) Pay-per-token pricing to align costs with actual usage patterns
B) Provisioned throughput with reserved capacity to guarantee low latency during peak hours
C) Provisioned throughput during business hours and pay-per-token during off-hours using scheduled jobs
D) Pay-per-token for the first month then migrate to provisioned throughput once traffic stabilizes

<details>
<summary>Answer</summary>

**Correct: A**

Pay-per-token pricing is ideal for unpredictable traffic patterns with high variability. The startup has massive variance (50-200 requests/min during business hours, near-zero at night). Pay-per-token means zero cost during low-traffic periods and no wasted reserved capacity. No minimum commitment required.

**Why the others are wrong:**
- Provisioned throughput requires paying for reserved capacity 24/7, wasting money during zero-traffic night hours. Only cost-effective when traffic is consistent and predictable.
- Databricks does not support switching pricing models on a schedule within the same endpoint. You cannot have hybrid provisioned/pay-per-token billing on one endpoint.
- Traffic is already highly variable (4x variance during day, near-zero at night) — this is the exact scenario where pay-per-token wins. Provisioned throughput is for stable, predictable workloads with SLA requirements.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A data science team needs to process 10 million customer support tickets stored in a Delta table to generate sentiment scores and topic classifications using Llama 3 70B. The results will populate a dashboard refreshed daily. Which inference approach minimizes cost while meeting the requirement?

A) Use batch inference with a Spark UDF calling the Foundation Model API endpoint to process the Delta table in parallel
B) Deploy a real-time Model Serving endpoint and send synchronous API calls from a Databricks notebook iterating over the Delta table
C) Export the Delta table to JSON files and use external API calls to AWS Bedrock for processing
D) Deploy multiple real-time endpoints with load balancing to handle the 10 million requests efficiently

<details>
<summary>Answer</summary>

**Correct: A**

Batch inference on Delta tables is the correct pattern for offline analytics workloads. Spark UDFs can parallelize calls to Foundation Model API across executors, processing millions of records efficiently. Daily refresh = no need for real-time latency. Batch inference maximizes throughput and minimizes cost for large-scale processing.

**Why the others are wrong:**
- Synchronous API calls in a loop is the slowest, most expensive approach. No parallelization, each call blocks until response. Real-time endpoints are for user-facing applications requiring low latency, not batch ETL.
- Exporting to JSON and using external APIs adds unnecessary data movement and API overhead. Databricks Foundation Model APIs integrate natively with Spark for batch processing. AWS Bedrock would require managing separate credentials and billing.
- Multiple real-time endpoints with load balancing is architectural overkill for a daily batch job. Real-time serving is for synchronous, latency-sensitive workloads, not offline analytics. This approach maximizes cost with no benefit.

</details>

---

**Q3.** An enterprise application requires calling Azure OpenAI GPT-4 models from Databricks notebooks and Model Serving endpoints. The Azure OpenAI API key must not be hardcoded. What is the correct configuration approach?

A) Store the Azure OpenAI API key in Databricks Secrets, then configure an external model endpoint in Model Serving that references the secret
B) Pass the Azure OpenAI API key as an environment variable in the cluster configuration and reference it in notebook code
C) Create a service principal in Azure AD and use OAuth token authentication instead of API keys
D) Store the API key in a Delta table with restricted permissions and query it at runtime

<details>
<summary>Answer</summary>

**Correct: A**

Databricks Secrets is the official secure storage for API keys and credentials. External model endpoints in Model Serving can reference secrets for Azure OpenAI, AWS Bedrock, or other external APIs. The secret is injected at runtime without exposing the key in code or logs. This is the exam-tested pattern for external model integration.

**Why the others are wrong:**
- Environment variables in cluster config are visible to all cluster users and appear in Spark UI logs. Not secure for production API keys. Secrets provide scoped access control and audit logging.
- While service principals are valid for Azure authentication, the exam tests the specific pattern of using Databricks Secrets with external model endpoints. Also, Azure OpenAI supports API key auth, which is simpler for this use case.
- Storing credentials in Delta tables is a security anti-pattern. Secrets are encrypted at rest and in transit. Delta table would expose the key to anyone with table read permissions and in query logs.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A production recommendation system serves 500 requests per second with strict 200ms p99 latency SLA. The team is choosing between DBRX Instruct and MPT-7B for generating product descriptions. Both models meet accuracy requirements in testing. Which model and configuration should they deploy?

A) DBRX Instruct with provisioned throughput to guarantee capacity for 500 req/sec
B) MPT-7B with provisioned throughput to minimize latency while meeting SLA
C) DBRX Instruct with pay-per-token pricing to reduce costs since both models meet accuracy requirements
D) MPT-7B with pay-per-token pricing and auto-scaling enabled for cost optimization

<details>
<summary>Answer</summary>

**Correct: B**

MPT-7B (7 billion parameters) is significantly smaller than DBRX Instruct (132B total parameters in MoE architecture). Smaller models = faster inference = lower latency. With a strict 200ms p99 SLA and 500 req/sec sustained traffic, provisioned throughput guarantees reserved capacity and consistent performance. Both models meet accuracy requirements, so choose the faster, cheaper model with guaranteed capacity.

**Why the others are wrong:**
- DBRX Instruct has higher latency than MPT-7B due to larger model size. While provisioned throughput guarantees capacity, it doesn't solve the latency problem. When a smaller model meets accuracy requirements, use it for better latency/cost.
- Pay-per-token does not guarantee capacity or latency SLAs. With 500 req/sec sustained traffic (predictable, high-volume), provisioned throughput is more cost-effective and ensures SLA compliance. Pay-per-token is for variable, unpredictable workloads.
- Pay-per-token with auto-scaling doesn't provide SLA guarantees. During traffic spikes, requests may queue or fail. Provisioned throughput reserves capacity. Also, 500 req/sec sustained = provisioned is cheaper than pay-per-token at high volumes.

</details>

---

**Q5.** A healthcare analytics team runs a weekly batch job that processes 5 million patient records through Llama 2 70B to extract clinical entities. The job currently uses pay-per-token pricing and costs $2,400 per run. The team wants to optimize costs. What should they do?

A) Switch to provisioned throughput with capacity reserved only during the batch job execution window
B) Keep pay-per-token pricing but increase parallelism to complete the job faster
C) Migrate to AWS Bedrock to take advantage of lower per-token pricing
D) Switch to a smaller model like MPT-7B or Llama 2 7B to reduce token processing costs

<details>
<summary>Answer</summary>

**Correct: D**

For a weekly batch job, pay-per-token is already the correct pricing model (usage is periodic, not continuous). To reduce costs, the only option is to reduce tokens processed or use a cheaper model. Switching to MPT-7B or Llama 2 7B (smaller models) means fewer parameters, lower per-token cost, and faster inference. The team should validate that the smaller model meets accuracy requirements for clinical entity extraction.

**Why the others are wrong:**
- Provisioned throughput cannot be reserved 'only during execution window' — you pay for reserved capacity continuously (by the hour or month). For a weekly job that runs a few hours, pay-per-token is cheaper. Provisioned is for sustained, predictable workloads.
- Increasing parallelism completes the job faster but processes the same number of tokens, so cost remains $2,400. Pay-per-token charges per input/output token regardless of how fast you process them. Parallelism improves time, not cost.
- Migrating to AWS Bedrock requires managing separate credentials, data egress costs from Databricks, and re-architecting the integration. Databricks Foundation Model APIs provide native integration with Spark. No evidence Bedrock is cheaper for this use case.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A real-time fraud detection system calls a Foundation Model API endpoint to score transactions. During testing, the team observes that 10% of API calls fail with 'rate limit exceeded' errors during peak traffic. The endpoint uses pay-per-token pricing. What is the BEST solution?

A) Implement exponential backoff retry logic in the application code to handle rate limits
B) Add a caching layer to reduce duplicate API calls and stay within rate limits
C) Deploy multiple pay-per-token endpoints and distribute traffic using a load balancer
D) Switch to provisioned throughput to guarantee token capacity and eliminate rate limiting

<details>
<summary>Answer</summary>

**Correct: D**

Pay-per-token endpoints have rate limits because capacity is shared across users. For production workloads that cannot tolerate failures or retries (fraud detection is latency-sensitive and high-stakes), provisioned throughput is the correct solution. Provisioned throughput reserves guaranteed tokens/sec capacity, eliminating rate limiting. The exam tests this exact scenario: when pay-per-token rate limits impact production SLAs, migrate to provisioned.

**Why the others are wrong:**
- Retry logic with backoff is a workaround, not a solution. Retries add latency (bad for real-time fraud detection) and still fail during sustained peak traffic. Provisioned throughput eliminates the root cause (insufficient capacity).
- Caching can help if there are duplicate requests, but fraud detection typically scores unique transactions. Even with caching, peak traffic will still hit rate limits. Provisioned throughput is the direct solution for guaranteed capacity.
- Databricks Foundation Model API does not support client-side load balancing across multiple pay-per-token endpoints. Each endpoint has its own rate limit, and you cannot distribute traffic. Provisioned throughput is the official solution for exceeding pay-per-token limits.

</details>

---

**Q7.** A company is building a document summarization pipeline that processes 1,000 documents per day during business hours. Average document size is 3,000 tokens input, 500 tokens output. They are choosing between pay-per-token ($0.003/1K input tokens, $0.015/1K output tokens) and provisioned throughput ($5/hour for 1,000 tokens/sec reserved capacity). Which pricing model is more cost-effective?

A) Provisioned throughput: guaranteed capacity for consistent performance justifies the cost premium
B) Pay-per-token for development, then switch to provisioned throughput for production workloads
C) Provisioned throughput reserved for 8 business hours/day at $40/day is cheaper than pay-per-token
D) Pay-per-token: total cost is approximately $16.50/day vs provisioned throughput at $120/day (24 hours)

<details>
<summary>Answer</summary>

**Correct: D**

Math: 1,000 docs × (3,000 input + 500 output) tokens = 3M input tokens + 500K output tokens per day. Cost = (3M/1K × $0.003) + (500K/1K × $0.015) = $9 + $7.50 = $16.50/day. Provisioned throughput at $5/hour × 24 hours = $120/day. Pay-per-token is 7× cheaper. For low-volume, periodic workloads, pay-per-token always wins. Exam tests this calculation skill.

**Why the others are wrong:**
- Provisioned throughput costs $120/day vs $16.50/day for pay-per-token. 'Guaranteed capacity' and 'consistent performance' are benefits, but the question asks which is more COST-EFFECTIVE. Pay-per-token is dramatically cheaper for this low-volume use case.
- This is advice for different scenarios, but doesn't answer the cost comparison question. For 1,000 docs/day, even in production, pay-per-token remains far cheaper. Provisioned is only cheaper at very high sustained volumes.
- Even if provisioned is reserved only 8 hours/day ($40), pay-per-token at $16.50/day is still 2.4× cheaper. Provisioned throughput billing is typically hourly and you cannot pause/resume reserved capacity to align with business hours.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A Databricks notebook needs to call an AWS Bedrock Claude 3 model for text generation. The AWS access key and secret key are stored in Databricks Secrets scope 'aws-keys'. What is the correct code pattern to configure the external model endpoint?

A) Use dbutils.secrets.get() to retrieve keys in the notebook, then configure an external model endpoint in Model Serving UI referencing the secret scope
B) Pass the AWS credentials directly in the API request headers when calling the Bedrock endpoint from the notebook
C) Export the secrets to environment variables using %env magic command, then call AWS Bedrock API using boto3 with default credential chain
D) Create an external model endpoint in Model Serving, configure AWS Bedrock as the provider, and reference the Databricks secret scope for authentication

<details>
<summary>Answer</summary>

**Correct: D**

External model endpoints in Databricks Model Serving support AWS Bedrock as a first-class provider. The correct pattern is: (1) store AWS credentials in Databricks Secrets, (2) create an external model endpoint via UI or API, (3) select AWS Bedrock as provider, (4) reference the secret scope for authentication. The endpoint proxies requests to Bedrock and injects credentials securely. This is the exam-tested integration pattern.

**Why the others are wrong:**
- While dbutils.secrets.get() retrieves secrets, the correct pattern is to configure the external model endpoint to reference the secret scope directly. The endpoint injects credentials at runtime. You don't manually retrieve secrets in notebook code for Model Serving endpoints.
- Passing credentials directly in request headers from the notebook is a security anti-pattern. Credentials would appear in notebook code and execution logs. External model endpoints handle credential injection securely via Secrets integration.
- Environment variables expose secrets to all notebook users and appear in Spark logs. boto3 calls bypass Databricks Model Serving governance and monitoring. The exam tests the Model Serving external endpoint pattern, not direct boto3 calls.

</details>

---

