## Chapter 2: Preparing Data for RAG on Databricks — Practice Questions

**Q1.** A company is building a RAG system for legal documents averaging 50 pages each. They need to balance context preservation with retrieval precision. Documents contain complex cross-references and legal citations. Which chunking strategy BEST meets these requirements?

A) Semantic chunking based on legal section boundaries with metadata tracking
B) Fixed-size chunking with 512 tokens and 10% overlap
C) Recursive chunking with 2048 tokens splitting on paragraph boundaries
D) Fixed-size chunking with 128 tokens and 25% overlap

<details>
<summary>Answer</summary>

**Correct: A**

Semantic chunking preserves legal document structure by respecting natural boundaries (sections, clauses). Legal documents have meaningful structural divisions that should not be broken. Metadata tracking allows precise retrieval of specific legal sections. This prevents splitting citations or cross-references mid-sentence.

**Why the others are wrong:**
- Fixed 512 tokens will arbitrarily split legal sections and citations, losing structural context critical for legal interpretation
- 2048 tokens is too large - will retrieve irrelevant content from different legal topics in the same chunk, reducing precision
- 128 tokens is too small - legal clauses and citations require more context; will fragment reasoning across too many chunks

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A data engineer creates a Delta table to store document chunks for Vector Search with schema: chunk_id STRING, chunk_text STRING, embedding ARRAY<FLOAT>, doc_source STRING, page_num INT. After indexing 100K chunks, retrieval quality is poor with many irrelevant results. Which change would MOST improve retrieval accuracy?

A) Change embedding from ARRAY<FLOAT> to ARRAY<DOUBLE> for higher precision
B) Add a chunk_length INT column to filter out chunks under 50 characters before indexing
C) Add chunk_overlap_start INT and chunk_overlap_end INT columns
D) Create a composite index on (doc_source, page_num)

<details>
<summary>Answer</summary>

**Correct: B**

Data quality filtering is critical before indexing. Chunks under 50 characters (headers, page numbers, whitespace artifacts) lack semantic meaning but generate embeddings that pollute the vector space. Filtering these low-quality chunks before indexing directly improves retrieval precision by removing noise from the search index.

**Why the others are wrong:**
- ARRAY<DOUBLE> vs ARRAY<FLOAT> has negligible impact on retrieval quality; embedding models output float32; storage cost increases without benefit
- Overlap metadata doesn't affect retrieval quality; overlap is a chunking parameter, not a search parameter
- Traditional indexes don't improve vector similarity search; Vector Search uses approximate nearest neighbor algorithms (ANN), not SQL indexes

</details>

---

**Q3.** An MLOps team needs to choose an embedding model for a RAG system that will answer technical questions from internal engineering documentation. Documents are in English only. Latency must be under 100ms per query. Which embedding model choice is MOST appropriate?

A) BGE-large-en-v1.5 served on Databricks Model Serving
B) OpenAI text-embedding-ada-002 via API for highest accuracy
C) Multilingual E5-large for future language support
D) sentence-transformers/all-MiniLM-L6-v2 for fastest inference

<details>
<summary>Answer</summary>

**Correct: A**

BGE-large-en-v1.5 offers the best balance: state-of-the-art accuracy for English retrieval tasks, can be served on Databricks Model Serving for low latency (<100ms), and keeps embeddings in-platform. For technical documentation, retrieval quality is critical, and BGE-large outperforms smaller models while meeting latency requirements when properly deployed.

**Why the others are wrong:**
- OpenAI API adds network latency (200-500ms typical), external dependency, and per-token costs; doesn't meet <100ms requirement reliably
- Multilingual models sacrifice English accuracy for multi-language support that isn't needed; E5-large is slower than BGE with no benefit for English-only content
- all-MiniLM-L6-v2 is fast but significantly less accurate than BGE for technical retrieval; will miss relevant documents and hurt RAG quality

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A GenAI application uses recursive chunking with 1000-token chunks and 20% overlap. Users report that retrieved chunks often lack sufficient context to answer questions. The development team wants to improve context while minimizing retrieval of irrelevant content. What is the BEST approach?

A) Increase chunk size to 2000 tokens and reduce overlap to 10%
B) Keep chunk size at 1000 tokens, increase overlap to 30%, and add metadata with surrounding chunk IDs
C) Increase overlap to 40% while keeping chunk size at 1000 tokens
D) Decrease chunk size to 500 tokens and increase overlap to 30%

<details>
<summary>Answer</summary>

**Correct: B**

This solution addresses context loss without introducing too much irrelevance. Increased overlap (30%) ensures important content appears in multiple chunks. Adding metadata with surrounding chunk IDs enables retrieval augmentation: if top chunk lacks context, the application can programmatically fetch adjacent chunks. This provides dynamic context expansion without forcing every query to retrieve oversized chunks.

**Why the others are wrong:**
- 2000-token chunks will retrieve too much irrelevant content; most questions need focused context, not entire document sections; reduces precision
- 40% overlap creates massive redundancy (400 tokens duplicated) without solving the core issue; wastes storage and compute without improving retrieval logic
- 500-token chunks fragment context even more; increasing overlap to 30% (150 tokens) still doesn't provide enough context per chunk; makes the problem worse

</details>

---

**Q5.** A company extracts text from 10,000 PDF documents containing tables, images with captions, and multi-column layouts. After chunking and indexing, retrieval fails to find information from tables. What is the MOST likely cause?

A) The embedding model was not trained on tabular data
B) Chunk size is too small to capture entire tables
C) PDF extraction did not preserve table structure as readable text
D) Vector Search does not support structured data retrieval

<details>
<summary>Answer</summary>

**Correct: C**

PDF extraction is the root cause. Standard PDF text extraction often fails on tables, extracting cells in random order or omitting structure entirely. If table data becomes garbled text like 'Revenue Q1 2023 100M Q2 150M' instead of a structured representation, embeddings cannot capture semantic meaning. Document parsing must use table-aware extraction (pdfplumber, PyPDF2 with table detection, or OCR with layout analysis) to convert tables to readable format.

**Why the others are wrong:**
- Embedding models encode any text; if tables are extracted as readable text (e.g., 'Q1 2023 Revenue: $100M'), embeddings will work fine; model training is not the issue
- Chunk size affects context but doesn't cause complete retrieval failure; even partial table content should retrieve if text is readable
- Vector Search works on any text embeddings; 'structured data' is a red herring - once tables are text, they're searchable like any content

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A Delta table stores document chunks with schema: id STRING, text STRING, embedding ARRAY<FLOAT>, created_at TIMESTAMP. A Databricks Vector Search index syncs from this table. New documents are added daily via MERGE operations. Which statement about index synchronization is correct?

A) Vector Search automatically detects Delta table changes and updates the index in real-time
B) The index must be manually refreshed after each MERGE operation
C) The entire index must be rebuilt daily to reflect new documents
D) Vector Search uses Delta table change data feed (CDF) to incrementally sync updates

<details>
<summary>Answer</summary>

**Correct: D**

Databricks Vector Search uses Delta's Change Data Feed (CDF) for incremental synchronization. When CDF is enabled on the source Delta table, Vector Search automatically detects inserts, updates, and deletes through the CDF and applies only the changes to the index. This is efficient for continuous updates and avoids full index rebuilds. CDF must be enabled on the table (ALTER TABLE SET TBLPROPERTIES (delta.enableChangeDataFeed = true)).

**Why the others are wrong:**
- Not real-time without CDF; Vector Search syncs on a schedule or trigger, not on every write; 'automatically' is misleading without understanding CDF is the mechanism
- Manual refresh is not required; Vector Search is designed for automated sync; this would be operationally burdensome and error-prone
- Full rebuilds are inefficient and unnecessary; Vector Search supports incremental updates via CDF, making daily rebuilds wasteful

</details>

---

**Q7.** An engineering team processes HTML documentation for a RAG system. Documents contain navigation menus, footers, and boilerplate text that appear on every page. After chunking, many retrieved chunks contain this repetitive content instead of relevant technical information. What preprocessing step would MOST improve retrieval quality?

A) Filter out chunks with high cosine similarity to a 'boilerplate template' embedding
B) Use semantic chunking to automatically exclude repetitive sections
C) Remove HTML tags and extract only paragraph and heading text before chunking
D) Increase chunk size to dilute boilerplate content with more relevant text

<details>
<summary>Answer</summary>

**Correct: A**

Filtering by similarity to known boilerplate directly addresses the problem. Create an embedding of common boilerplate text (navigation, footer, copyright), then exclude chunks with >0.9 cosine similarity to this template during preprocessing. This is a data quality filter that removes noise before indexing. It's precise because boilerplate is semantically similar across pages, making it detectable via embeddings.

**Why the others are wrong:**
- Semantic chunking splits on topic boundaries but doesn't identify or remove boilerplate; navigation menus can appear as semantic units and still get indexed
- Removing HTML tags helps but doesn't exclude boilerplate content; paragraph text can still be navigation links or footer text that's semantically meaningless
- Larger chunks don't solve the problem; boilerplate will still be present and can still dominate the embedding if it's a significant portion of the chunk text

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A RAG system uses fixed-size chunking with 800 tokens and 15% overlap (120 tokens). Documents average 10,000 tokens each. Approximately how many chunks will be generated per document?

A) 10 chunks
B) 13 chunks
C) 18 chunks
D) 15 chunks

<details>
<summary>Answer</summary>

**Correct: D**

Calculation: effective stride = chunk_size - overlap = 800 - 120 = 680 tokens per chunk advance. Number of chunks = ceiling((doc_size - chunk_size) / stride) + 1 = ceiling((10000 - 800) / 680) + 1 = ceiling(13.5) + 1 = 14 + 1 = 15 chunks. The overlap means chunks share content, so you need more chunks to cover the document than simple division would suggest.

**Why the others are wrong:**
- 10 chunks assumes no overlap (10000/800 = 12.5, rounded) but ignores the 15% overlap which increases chunk count
- 13 chunks is close but doesn't account for the full document coverage with proper overlap math; off by 2
- 18 chunks over-counts; this would result from miscalculating overlap as additive rather than a stride reduction

</details>

---

