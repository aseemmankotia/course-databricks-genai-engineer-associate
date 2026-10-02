# Foundation Model APIs & Production Serving Cheat Sheet

## Pricing Models Comparison

| **Pay-Per-Token** | **Provisioned Throughput** |
|-------------------|----------------------------|
| Billed per 1K input/output tokens | Billed per hour for reserved capacity |
| No minimum commitment | Reserved capacity (tokens/sec) |
| Variable cost based on usage | Fixed cost regardless of usage |
| Rate limits apply (shared capacity) | No rate limits (dedicated capacity) |
| **Use when:** unpredictable traffic, dev/test, low volume | **Use when:** production, SLA-critical, sustained high traffic |

**Exam rule:** Mention of "variable traffic", "prototype", "cost optimization for low volume" → pay-per-token. Mention of "strict SLA", "1000 req/sec", "rate limit errors" → provisioned.

---

## Databricks Foundation Models

| Model | Parameters | Best For | Latency |
|-------|-----------|----------|--------|
| **MPT-7B** | 7B | Speed, cost, simple tasks | Fastest |
| **MPT-30B** | 30B | Balance of speed/accuracy | Fast |
| **Llama 2 7B** | 7B | General purpose, fast | Fast |
| **Llama 2 70B** | 70B | High accuracy, complex tasks | Slower |
| **Llama 3 70B** | 70B | Latest Llama, best accuracy | Slower |
| **DBRX Instruct** | 132B MoE | Databricks flagship, efficient | Medium |

**Exam tip:** Smaller models (7B) = faster + cheaper. Larger models (70B) = more accurate + expensive. If accuracy equal, choose smaller for latency SLA.

---

## Real-Time vs Batch Inference

| **Real-Time** | **Batch** |
|---------------|----------|
| Synchronous API calls | Asynchronous Spark jobs |
| <1 sec latency | Minutes to hours |
| User-facing apps | Analytics, ETL |
| **Examples:** chatbots, fraud detection, live recommendations | **Examples:** daily reports, sentiment analysis on Delta tables, bulk scoring |
| **Pattern:** REST API → Model Serving endpoint | **Pattern:** Spark UDF → Foundation Model API → Delta table |

**Exam rule:** "User waits for response" or "200ms SLA" → real-time. "Daily dashboard" or "process 10M records" → batch.

---

## External Model Endpoints

**Supported Providers:**
- Azure OpenAI (GPT-4, GPT-3.5, etc.)
- AWS Bedrock (Claude, Titan, etc.)
- Custom HTTPS endpoints

**Configuration Pattern:**
1. Store API key/secret in **Databricks Secrets** scope
2. Create **external model endpoint** in Model Serving UI/API
3. Select provider (Azure OpenAI / AWS Bedrock)
4. Reference **secret scope** for authentication
5. Endpoint proxies requests and injects credentials securely

**Exam trap:** Don't use `dbutils.secrets.get()` in notebook code for Model Serving. Let the endpoint reference the secret scope directly.

---

## Cost Calculation Example

**Scenario:** 1,000 docs/day, 3K input tokens, 500 output tokens per doc.

**Pay-per-token rates:** $0.003/1K input, $0.015/1K output
- Input cost: (1,000 × 3,000 / 1,000) × $0.003 = $9/day
- Output cost: (1,000 × 500 / 1,000) × $0.015 = $7.50/day
- **Total: $16.50/day**

**Provisioned throughput:** $5/hour × 24 hours = **$120/day**

**Decision:** Pay-per-token is 7× cheaper for this low-volume workload.

**Exam tip:** Always calculate total tokens (input + output), then apply per-1K rates. Compare to provisioned hourly cost × 24.

---

## Rate Limiting & Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| **Rate limit exceeded (429)** | Pay-per-token shared capacity | Switch to provisioned throughput |
| **High latency (>1 sec)** | Large model or high traffic | Use smaller model or provisioned capacity |
| **API key errors** | Hardcoded credentials | Store in Secrets, use external endpoint |
| **Cost spike** | Inefficient prompts or over-processing | Optimize prompt length, use smaller model |

---

## Batch Inference Pattern (Spark UDF)

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType
import requests

# Define UDF calling Foundation Model API
@udf(returnType=StringType())
def generate_summary(text):
    response = requests.post(
        "https://<workspace>.databricks.com/serving-endpoints/<endpoint>/invocations",
        headers={"Authorization": f"Bearer {dbutils.notebook.entry_point.getDbutils().notebook().getContext().apiToken().get()}"},
        json={"inputs": [text]}
    )
    return response.json()["predictions"][0]

# Apply to Delta table
df = spark.table("customer_reviews")
df.withColumn("summary", generate_summary("review_text")).write.saveAsTable("reviews_with_summaries")
```

**Exam note:** Batch = Spark UDF + Delta table. Real-time = direct API calls from application.

---

## Key Exam Facts

- **Pay-per-token:** no rate limit guarantee, billed per token, variable cost
- **Provisioned:** guaranteed tokens/sec, fixed cost, no rate limits
- **Smaller models faster:** 7B models have lower latency than 70B
- **External endpoints:** use Secrets + Model Serving, not hardcoded keys
- **Batch inference:** Spark UDF on Delta tables for high-throughput offline jobs
- **Real-time inference:** synchronous API for user-facing, latency-sensitive apps
