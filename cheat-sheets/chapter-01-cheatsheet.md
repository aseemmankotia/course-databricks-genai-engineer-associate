# Chapter 1: Designing GenAI Applications on Databricks

## Decision Matrix: RAG vs Fine-Tuning vs Prompt Engineering

| Scenario | Approach | Why |
|----------|----------|-----|
| Knowledge changes frequently | RAG | Real-time updates, no retraining |
| Need citations/sources | RAG | Retrieval provides source documents |
| Have 1,000+ labeled examples | Fine-tuning | Learn patterns from data |
| Consistent format/style needed | Fine-tuning | Training teaches style better than prompting |
| Specialized domain language | Fine-tuning | Model learns jargon/terminology |
| Simple task, no training data | Prompt engineering | Fast iteration, no infrastructure |
| Budget-constrained MVP | Prompt engineering | Lowest cost, quickest to deploy |
| External factual knowledge | RAG | Retrieval more reliable than memorization |

## Compound AI System Components

| Component | Purpose | When Required |
|-----------|---------|---------------|
| Retriever | Fetch candidate documents | RAG applications, knowledge bases |
| Reranker | Improve precision of top results | >100K documents, multi-hop questions |
| Generator | Create response from context | All RAG systems |
| Orchestrator | Manage pipeline flow | Multi-step workflows, conditional logic |
| Agent | Execute tools, multi-step reasoning | API calls, complex decision trees |
| Tools | External functions/APIs | Any action beyond text generation |
| Memory | Maintain state across turns | Multi-turn conversations, context continuity |
| Guardrails | Enforce safety/policy rules | Production systems, regulated domains |

## Model Selection: Databricks Foundation Model APIs

| Model | Best For | Latency | Cost | Key Strength |
|-------|----------|---------|------|---------------|
| DBRX Instruct | High-throughput APIs, structured output | Low | Low | Instruction following, JSON output |
| Llama 3.1 8B | Simple tasks, batch processing | Lowest | Lowest | Speed, efficiency |
| Llama 3.1 70B | Balanced performance | Medium | Medium | General purpose |
| Llama 3.1 405B | Complex reasoning | High | High | Highest capability |
| MPT-7B | Fine-tuning base | Low | Low | Customization |

## Agent Tool Selection by Domain

| Domain | Required Tools | Critical Guardrails |
|--------|----------------|---------------------|
| SQL Generation | Schema introspector, Query validator, SQL executor | Read-only enforcement, SQL injection detection |
| Financial Services | Payment API, Balance checker, Transaction validator | Amount limits, Human confirmation, Audit logging |
| Healthcare | Medical DB retrieval, Insurance API, Scheduling API | No diagnosis claims, HIPAA compliance, Data redaction |
| Customer Support | Ticket creator, Knowledge base search, Escalation router | PII protection, Sentiment monitoring, Human handoff |
| Content Moderation | Classification model, Evidence retriever, Human review trigger | Confidence thresholds, Bias detection, Explainability |

## Structured Output Configuration

```python
# response_format parameter for guaranteed JSON
response = model.predict({
  "messages": [...],
  "response_format": {
    "type": "json_schema",
    "json_schema": {
      "name": "product_description",
      "schema": {
        "type": "object",
        "properties": {
          "title": {"type": "string"},
          "price": {"type": "number"}
        },
        "required": ["title", "price"]
      }
    }
  }
})
```

## Latency vs Accuracy Trade-offs

| Requirement | Architecture Choice | Latency Target |
|-------------|---------------------|----------------|
| Real-time chat (<2s) | DBRX with GPU serving | <2s |
| Search with synthesis (5-10s) | Retriever→Reranker→Generator | 5-10s |
| Batch processing (hours OK) | Large model (405B), batch inference | Hours |
| API rate limits matter | Smaller model, higher throughput | <1s |

## Common Exam Traps

1. **RAG vs Fine-tuning**: Exam shows frequently updated docs → RAG, not fine-tuning
2. **Reranker necessity**: Large corpus (>100K docs) → reranker required for accuracy
3. **Agent vs simple pipeline**: API calls or multi-step logic → agent needed
4. **Guardrails for actions**: Financial/medical/legal → human-in-loop confirmation mandatory
5. **Model size**: Use smallest model that meets requirements (DBRX usually sufficient)
6. **Structured output**: Use response_format parameter, not prompt + parsing
7. **Tool validation**: Agents need validator tools before executor tools (e.g., schema check before SQL run)

## Cost Optimization Rules

- Start with prompt engineering, escalate to RAG/fine-tuning only if needed
- Use DBRX for most tasks, 405B only when demonstrably insufficient
- Fine-tuning: one-time cost amortized over usage (calculate ROI vs prompting)
- RAG: retrieval cost + generation cost (optimize retrieval to reduce context size)
- Reranker: adds cost but improves accuracy (ROI high for large knowledge bases)

## Key Numbers

- Fine-tuning dataset: 1,000+ examples minimum, 10K+ ideal
- Retriever candidates: 50-100 for reranking, 5-10 for generation
- Context window: DBRX 32K tokens, Llama 405B 128K tokens
- Latency: DBRX ~1-2s, Llama 405B ~5-8s (approximate, varies by prompt)
- Reranker: Use when initial retrieval >50 candidates from >100K doc corpus
