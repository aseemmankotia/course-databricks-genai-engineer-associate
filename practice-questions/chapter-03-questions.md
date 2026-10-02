## Chapter 3: Building RAG Applications with Vector Search — Practice Questions

**Q1.** A RAG application retrieves 10 documents but the LLM generates an answer containing facts not present in any retrieved document. Which combination of prompt engineering techniques BEST reduces this hallucination?

A) Increase num_results to 20 and add a reranking step with a cross-encoder model
B) Lower the similarity threshold from 0.7 to 0.5 to retrieve more diverse documents
C) Use metadata filtering to restrict results to documents from the last 30 days only
D) Add explicit instructions in the system prompt to only use provided context and respond 'I don't have enough information' when context is insufficient, plus cite source documents in the response

<details>
<summary>Answer</summary>

**Correct: D**

Explicit grounding instructions in the system prompt directly address hallucination by constraining the model to only use retrieved context and providing a fallback ('I don't have enough information'). Citing sources adds traceability and further discourages fabrication. This is the primary hallucination mitigation technique tested on the exam.

**Why the others are wrong:**
- Retrieving more documents (higher num_results) and reranking increases the chance of finding relevant content but doesn't prevent the LLM from hallucinating if the prompt doesn't enforce grounding. More documents can actually add noise.
- Lowering similarity threshold retrieves lower-quality, less relevant documents, which increases noise and can worsen hallucination rather than reduce it.
- Metadata filtering helps retrieve relevant subsets but doesn't address the core issue of the model generating content beyond retrieved context. Time-based filtering alone won't prevent hallucination.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A company needs to retrieve product documentation chunks for a customer support chatbot. Testing shows that setting num_results=3 misses relevant answers 40% of the time, while num_results=15 includes too much irrelevant content. What is the BEST approach to balance precision and recall?

A) Set num_results=3 and remove the similarity threshold to ensure all retrieved documents are used
B) Use num_results=10 and implement metadata filtering to restrict results to documents tagged with the user's product category
C) Set num_results=3 and increase the embedding model dimensions to capture more semantic detail
D) Set num_results=15 and apply a similarity score threshold of 0.65 to filter out low-relevance results, then pass filtered results to the LLM

<details>
<summary>Answer</summary>

**Correct: D**

Higher num_results (15) increases recall by retrieving more potentially relevant documents. Applying a similarity threshold (0.65) then filters out low-quality matches to maintain precision. This two-step approach—cast a wide net, then filter—is the standard pattern for balancing precision/recall in vector search.

**Why the others are wrong:**
- Removing similarity threshold with only 3 results doesn't solve the recall problem (still missing 40% of answers) and may include irrelevant documents in the small set.
- Metadata filtering helps but doesn't address the core precision/recall tradeoff. If only 3 results are retrieved, recall remains poor even with filtering. This could work if combined with higher num_results.
- Embedding model dimensions affect semantic representation quality but don't directly solve the num_results tuning problem. This is a retrieval parameter issue, not an embedding issue.

</details>

---

**Q3.** An engineer is building a RAG pipeline using Databricks Vector Search. Which code snippet correctly retrieves the top 5 most similar documents with a minimum similarity score of 0.7?

A) index.similarity_search(query_vector=embedding, num_results=5) followed by manual filtering of results where score >= 0.7
B) index.similarity_search(query_vector=embedding, num_results=5, filters='score >= 0.7')
C) index.similarity_search(query_vector=embedding, num_results=5, score_threshold=0.7)
D) index.similarity_search(query_vector=embedding, num_results=5, filters={'similarity': {'$gte': 0.7}})

<details>
<summary>Answer</summary>

**Correct: A**

Databricks Vector Search similarity_search does not have a built-in score_threshold parameter in the API call itself. The correct approach is to retrieve num_results documents and then filter the returned results based on the similarity score in your application code. The exam tests knowing what parameters actually exist in the API.

**Why the others are wrong:**
- The 'filters' parameter in Vector Search is for metadata filtering (e.g., doc_category='FAQ'), not for similarity score filtering. This syntax is incorrect.
- score_threshold is not a valid parameter for the Databricks Vector Search similarity_search API as of 2026. This is a common misconception from other vector databases.
- The filters parameter uses metadata field filtering syntax but 'similarity' is not a metadata field you can filter on—it's the computed score. This confuses metadata filtering with score filtering.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A legal document retrieval system needs to return only contracts signed after January 1, 2025, and marked as 'active'. Which similarity_search call correctly applies these metadata filters?

A) index.similarity_search(query_vector=emb, num_results=10, filters='status = active AND sign_date > 2025-01-01')
B) index.similarity_search(query_vector=emb, num_results=10, filters={'status': 'active', 'sign_date': {'$gt': '2025-01-01'}})
C) index.similarity_search(query_vector=emb, num_results=10, metadata={'status': 'active', 'sign_date__gte': '2025-01-01'})
D) index.similarity_search(query_vector=emb, num_results=10, where={'status': 'active', 'sign_date': {'>=': '2025-01-01'}})

<details>
<summary>Answer</summary>

**Correct: B**

Databricks Vector Search filters parameter uses dictionary syntax with MongoDB-style operators: {'field': 'value'} for equality and {'field': {'$gt': 'value'}} for comparisons. Multiple conditions are AND-ed by including them in the same dictionary. This is the correct syntax tested on the exam.

**Why the others are wrong:**
- SQL-style string filters ('status = active AND...') are not valid syntax for Databricks Vector Search. The filters parameter expects a dictionary, not a string.
- The 'metadata' parameter doesn't exist in the similarity_search API. The correct parameter name is 'filters', and the double-underscore syntax (sign_date__gte) is not used.
- The 'where' parameter doesn't exist in Databricks Vector Search API. Additionally, the operator syntax {'>=': 'value'} is incorrect—should be {'$gte': 'value'}.

</details>

---

**Q5.** A RAG application's prompt template includes: system prompt with grounding instructions, retrieved context injection, and user query. During testing, the model frequently responds 'I cannot answer that' even when relevant information is in the retrieved context. What is the MOST likely cause?

A) The num_results parameter is set too low (only 3 documents retrieved)
B) The similarity threshold is set too high (0.9), filtering out relevant but not perfectly matching documents
C) The system prompt instructions are too strict, telling the model to say 'I don't know' even when partial information is available, without guidance on using incomplete context
D) The retrieved context is injected after the user query instead of before it in the prompt template

<details>
<summary>Answer</summary>

**Correct: C**

Overly strict grounding instructions can cause the model to refuse to answer even when useful information exists in context. The system prompt should instruct the model to acknowledge limitations ('based on available information...') while still attempting to answer using partial context. This balance is a common exam scenario—testing understanding of prompt engineering nuance.

**Why the others are wrong:**
- Low num_results would cause missing answers due to recall issues, but the question states relevant information IS in the retrieved context, so retrieval count isn't the issue.
- High similarity threshold could filter out relevant docs, but again, the scenario confirms relevant information is already in the retrieved context being passed to the model. The problem is prompt-side.
- Prompt template ordering matters for some models, but modern LLMs handle context placement flexibly. The issue described (refusing to answer despite having info) points to instruction-following behavior, not context positioning.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** An engineer wants to improve retrieval quality by reranking the top 20 documents from Vector Search using a cross-encoder model before passing the top 5 to the LLM. What is the primary tradeoff of this approach?

A) Reranking reduces hallucination by filtering out low-quality documents, but requires fine-tuning the cross-encoder on domain data
B) Reranking improves recall by finding documents the vector search missed, but increases infrastructure costs substantially
C) Reranking eliminates the need for prompt-based grounding instructions, but requires storing additional metadata in the vector index
D) Reranking increases retrieval precision by scoring query-document pairs more accurately, but adds significant latency to the pipeline

<details>
<summary>Answer</summary>

**Correct: D**

Cross-encoder reranking runs a transformer model on each (query, document) pair to compute a relevance score, which is more accurate than vector similarity alone (improves precision). However, this requires N forward passes through the model (e.g., 20 passes for 20 docs), adding significant latency. This precision/latency tradeoff is the key exam point about reranking.

**Why the others are wrong:**
- While fine-tuning a cross-encoder can improve performance, it's not required—pretrained rerankers work out-of-box. The primary tradeoff is latency, not training requirements. Reranking also doesn't directly reduce hallucination (that's a prompt engineering concern).
- Reranking doesn't improve recall—it reorders already-retrieved documents. Recall is determined by the initial vector search num_results. Infrastructure costs are minimal compared to LLM inference costs.
- Reranking does NOT eliminate the need for grounding instructions in the prompt. Reranking improves which documents are passed to the LLM, but the LLM still needs prompt instructions to avoid hallucination. No additional metadata storage is needed.

</details>

---

**Q7.** A medical knowledge RAG system must retrieve patient education documents. Compliance requires that responses cite the specific source document title and section. Which prompt template component BEST implements this requirement?

A) Add a post-processing step after LLM generation that uses regex to extract text from the response and match it back to source documents
B) Set num_results=1 to ensure only one source document is used, making citation implicit
C) Include document IDs in the retrieved context and instruct the system prompt: 'Always cite the document ID and section for each fact you use in your response'
D) Use metadata filtering to restrict retrieval to only documents marked as 'citation_required=true'

<details>
<summary>Answer</summary>

**Correct: C**

Including source metadata (document title, section, ID) in the retrieved context and explicitly instructing the model in the system prompt to cite sources is the standard approach for citation requirements. This leverages the LLM's ability to follow instructions and reference provided metadata. Exam tests understanding that citation is a prompt engineering task.

**Why the others are wrong:**
- Post-processing with regex is fragile and error-prone (models paraphrase, making exact matching hard). It also adds complexity. The better approach is to have the model generate citations natively via prompt instructions.
- Restricting to 1 result sacrifices recall and doesn't actually cite the source—it just uses one document. Compliance requires explicit citation regardless of how many sources are used.
- Metadata filtering controls which documents are retrieved but doesn't make the LLM cite them in the response. Citation is an output formatting requirement handled by prompt instructions, not retrieval filtering.

</details>

---

**Q8.** When tuning a RAG system's retrieval parameters, an engineer observes: num_results=5 achieves 60% recall, num_results=10 achieves 85% recall, num_results=20 achieves 90% recall. The average relevance score of documents 11-20 is 0.55. What is the BEST recommendation to optimize the system?

A) Use num_results=10 as it provides a good balance of recall (85%) without the low-relevance tail present in 11-20
B) Use num_results=20 to maximize recall, then apply a similarity threshold of 0.65 to filter low-relevance documents from positions 11-20
C) Use num_results=5 and improve the embedding model to capture more semantic detail and increase recall
D) Use num_results=20 and implement a reranking model to reorder documents by query-specific relevance before generation

<details>
<summary>Answer</summary>

**Correct: B**

The data shows significant recall gain from 85% to 90% in positions 11-20 (5 percentage points), but those documents have lower relevance (0.55). The best approach is to retrieve all 20 (capture that 5% recall gain) but filter out documents below a quality threshold (0.65). This maximizes recall while maintaining precision. Exam tests data-driven parameter tuning.

**Why the others are wrong:**
- num_results=10 leaves 5 percentage points of recall on the table. If filtering can remove the low-quality documents from 11-20 while keeping any high-quality ones, num_results=20 with filtering is superior.
- Improving the embedding model is a longer-term solution and doesn't address the immediate parameter tuning question. The data shows num_results=5 has poor recall (60%)—that's a retrieval count issue, not necessarily an embedding quality issue.
- Reranking could work but adds latency and complexity. The simpler solution of threshold filtering achieves the same goal (remove low-relevance docs) with less overhead. Reranking is better when relative ordering matters, not just filtering.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

