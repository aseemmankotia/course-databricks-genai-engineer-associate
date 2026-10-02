# Chapter 9: Evaluation and Monitoring Cheat Sheet

## mlflow.evaluate() for GenAI

### Required Dataset Columns
| Metric | Required Columns |
|--------|------------------|
| Faithfulness | question, generated_answer, retrieved_context |
| Answer Correctness | question, generated_answer, ground_truth |
| Answer Relevance | question, generated_answer |
| Context Precision | question, retrieved_context, ground_truth |
| Context Recall | question, retrieved_context, ground_truth |

**Exam Key**: `ground_truth` NOT needed for faithfulness (only for answer correctness)

## Core Metrics Comparison

| Metric | What It Measures | Evaluates | Detects |
|--------|------------------|-----------|----------|
| **Faithfulness** | Answer only uses info from context | Generation | Hallucination |
| **Answer Relevance** | Answer addresses the question | Generation | Off-topic responses |
| **Context Precision** | Relevant chunks ranked first | Retrieval | Poor ranking |
| **Context Recall** | All relevant chunks retrieved | Retrieval | Missing chunks |

**Exam Trap**: Precision = ranking quality, Recall = coverage (not the same!)

## LLM-as-a-Judge

**How it works**: API call to strong LLM (GPT-4, DBRX) per example to score outputs

**Primary cost driver**: LLM API calls (not compute or storage)

**Use case**: Custom criteria evaluation beyond standard metrics

**Exam tip**: More expensive but more flexible than rule-based metrics

## Evaluation Types

| Type | When | Data Source | Use Case |
|------|------|-------------|----------|
| **Offline** | Before deployment | Static test dataset | Initial validation |
| **Online** | Production | Sampled live traffic via inference tables | Detect drift, real-world performance |

**Pattern for online eval**: Inference tables → sample → mlflow.evaluate()

## Databricks Production Monitoring

### Inference Tables
- **What**: Model Serving auto-logs requests/responses to Delta table
- **Enable**: Configure on model endpoint
- **Contains**: Input questions, generated answers, timestamps, metadata
- **Exam key**: Foundation for production monitoring

### Lakehouse Monitoring
- **Input**: Delta table (e.g., inference table)
- **Tracks**: Quality metrics, data drift, model drift
- **Alerts**: Threshold-based notifications
- **Exam scenario**: Inference tables + Lakehouse Monitoring = complete production monitoring solution

## Model Size Trade-offs

| Model Size | Latency | Cost (per 1K tokens) | Accuracy | Use When |
|------------|---------|----------------------|----------|----------|
| **7B params** | ~200ms | ~$0.001 | Lower | High volume, latency-sensitive, cost-constrained |
| **70B params** | ~800ms | ~$0.02 | Higher | Low volume, accuracy-critical, budget available |

**Exam calculation**: 10,000 queries/day × 20× cost difference = significant budget impact

## Common Metric Patterns

### High Precision, Low Recall
- **Meaning**: Retrieved chunks are relevant but missing many
- **Fix**: Increase chunk count, broaden retrieval parameters

### Low Precision, High Recall
- **Meaning**: Retrieving many chunks but irrelevant ones ranked high
- **Fix**: Improve ranking algorithm, better embeddings

### Low Faithfulness, High Relevance
- **Meaning**: Answers on-topic but hallucinating facts
- **Fix**: Stricter prompts, better grounding instructions

### High Faithfulness, Low Relevance
- **Meaning**: No hallucination but doesn't answer question
- **Fix**: Improve answer generation prompt

## Exam Gotchas

1. **Faithfulness ≠ Answer Correctness**: Faithfulness checks grounding in context, correctness checks match to ground truth
2. **Context metrics are retrieval-only**: Don't measure generation quality
3. **Inference tables are passive logging**: Need Lakehouse Monitoring for active alerting
4. **Online eval requires sampling**: Don't evaluate 100% of production traffic (cost prohibitive)
5. **LLM-as-judge costs scale with dataset size**: Each example = API call

## Quick Commands

```python
# Run evaluation with multiple metrics
results = mlflow.evaluate(
    model=model_uri,
    data=eval_df,  # must have required columns
    targets="ground_truth",
    predictions="generated_answer",
    extra_metrics=[
        mlflow.metrics.faithfulness(),
        mlflow.metrics.answer_relevance(),
    ],
    evaluators="default"
)

# Access metric scores
print(results.metrics)
```

**Exam weight**: 12% of total exam — heavily tested on LLM-as-judge and RAG-specific metrics
