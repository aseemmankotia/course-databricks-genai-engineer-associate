# RAG with Vector Search - Exam Cheat Sheet

## Databricks Vector Search API

### similarity_search Method
```python
results = index.similarity_search(
    query_vector=embedding,      # Required: embedding array
    num_results=10,               # Default 10, max depends on index config
    filters={'category': 'FAQ'},  # Optional: metadata filtering
    columns=['id', 'text', 'metadata']  # Optional: which columns to return
)
```

**Key Points:**
- **NO score_threshold parameter** in API (common exam trap)
- Filter similarity scores manually after retrieval: `[r for r in results if r['score'] >= 0.7]`
- Returns results in descending similarity order

## Metadata Filtering Syntax

| Operation | Syntax | Example |
|-----------|--------|----------|
| Equality | `{'field': 'value'}` | `{'status': 'active'}` |
| Greater than | `{'field': {'$gt': value}}` | `{'date': {'$gt': '2025-01-01'}}` |
| Greater/equal | `{'field': {'$gte': value}}` | `{'price': {'$gte': 100}}` |
| Less than | `{'field': {'$lt': value}}` | `{'priority': {'$lt': 5}}` |
| AND (multiple) | Combine in dict | `{'status': 'active', 'date': {'$gt': '2025-01-01'}}` |

**Exam Note:** MongoDB-style syntax, NOT SQL strings. Common wrong answer: `filters="status = 'active'"`

## Precision vs Recall Tuning

| Parameter | Effect on Precision | Effect on Recall | When to Use |
|-----------|-------------------|-----------------|-------------|
| **High num_results** (20+) | Lower (more noise) | Higher (find more docs) | Maximize coverage, use with filtering |
| **Low num_results** (3-5) | Higher (top matches only) | Lower (may miss docs) | High-confidence answers only |
| **High similarity threshold** (0.8+) | Higher (strict matching) | Lower (filters more docs) | Precision-critical applications |
| **Low similarity threshold** (0.5-0.6) | Lower (includes marginal matches) | Higher (keeps more docs) | Recall-critical applications |

**Optimization Pattern:** Retrieve high num_results (15-20) → Apply similarity threshold (0.65-0.75) → Pass filtered results to LLM

## Prompt Template Structure

```
[SYSTEM PROMPT]
You are a helpful assistant. Answer questions using ONLY the provided context.
If the context doesn't contain enough information, respond "I don't have enough information to answer that."
Always cite the source document ID for facts you use.

[RETRIEVED CONTEXT]
Document 1 (ID: doc_123): [chunk text]
Document 2 (ID: doc_456): [chunk text]
...

[USER QUERY]
{user_question}
```

### Grounding Techniques (Hallucination Mitigation)

1. **Explicit constraints:** "Use ONLY provided context"
2. **Fallback instruction:** "Say 'I don't have enough information' if uncertain"
3. **Citation requirement:** "Cite document ID for each fact"
4. **Negative instruction:** "Do NOT use external knowledge or make assumptions"

**Exam Tip:** Grounding via prompt instructions is PRIMARY hallucination defense (tested heavily)

## Reranking with Cross-Encoder

**Purpose:** Improve precision by scoring query-document pairs more accurately than vector similarity

**Process:**
1. Retrieve top 20 docs via Vector Search (high recall)
2. Pass each (query, doc) pair through cross-encoder
3. Reorder by cross-encoder scores
4. Take top 5 for LLM generation

**Tradeoff:**
- ✅ Benefit: Higher precision (better relevance ordering)
- ❌ Cost: Significant latency (N transformer forward passes)
- ❌ Does NOT improve recall (only reorders existing results)

**When to Use:** High-value applications where precision matters more than speed

## Common Exam Traps

| Wrong Answer | Why It's Wrong | Right Answer |
|--------------|----------------|---------------|
| `score_threshold=0.7` parameter | Not in API | Manual filtering: `[r for r in results if r['score'] >= 0.7]` |
| `filters="category = 'FAQ'"` | SQL syntax not supported | `filters={'category': 'FAQ'}` (dict) |
| Reranking improves recall | Reranking only reorders | Increase num_results to improve recall |
| Citation via post-processing | Fragile, complex | Prompt instruction: "Cite sources" + include metadata |
| Lower num_results = more recall | Opposite | Higher num_results = more recall (but more noise) |

## Quick Decision Matrix

**Scenario: Missing relevant answers (low recall)**  
→ Increase `num_results` OR lower similarity threshold

**Scenario: Too much irrelevant content (low precision)**  
→ Decrease `num_results` OR raise similarity threshold OR add reranking

**Scenario: Model hallucinates facts**  
→ Add grounding instructions to system prompt + citation requirement

**Scenario: Need to filter by document type/date**  
→ Use `filters` parameter with metadata dict

**Scenario: Improve relevance ordering**  
→ Add cross-encoder reranking (if latency acceptable)

## Numbers to Remember

- **Typical num_results range:** 3-20 (depends on use case)
- **Typical similarity threshold:** 0.6-0.8 (0.7 common default)
- **Reranking overhead:** ~50-200ms per document (depends on model)
- **Precision/Recall tradeoff:** num_results=5 (~high precision), num_results=20 (~high recall)
