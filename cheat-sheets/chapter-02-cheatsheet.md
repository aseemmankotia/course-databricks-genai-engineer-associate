# Data Preparation for RAG — Exam Cheat Sheet

## Chunking Strategies Comparison

| Strategy | How It Works | Best For | Pros | Cons |
|----------|-------------|----------|------|------|
| **Fixed-size** | Split every N tokens with overlap | Simple docs, blogs, uniform content | Fast, predictable size | Ignores structure, can split mid-sentence |
| **Recursive** | Split on delimiters (paragraphs, sections) then size | Structured docs with hierarchy | Respects boundaries, better context | Slower, variable chunk sizes |
| **Semantic** | Detect topic changes using embeddings | Documents with clear topic shifts | Aligns chunks to concepts | Compute-intensive, variable sizes |

**Exam Tip**: Default to recursive unless speed (fixed) or clear topic structure (semantic) is priority.

---

## Chunk Size Impact

| Chunk Size | Effect on Context | Effect on Retrieval | Typical Use Case |
|------------|------------------|---------------------|------------------|
| **<256 tokens** | Loss of context | High precision, low recall | Definitions, facts, QA pairs |
| **512-1024 tokens** | Good balance | Best precision/recall trade-off | **Default sweet spot** |
| **>2048 tokens** | Excessive context | Low precision (too much noise) | Full document retrieval |

**Exam Tip**: Too small = fragments meaning; too large = retrieves irrelevant content.

---

## Chunk Overlap Guidelines

- **Typical range**: 10-20%
- **10%**: Simple content, clear boundaries
- **15-20%**: Complex content, preserves context continuity
- **>25%**: Rarely justified (wastes storage)

**Calculation**: Overlap = chunk_size × overlap_percentage  
**Example**: 800 tokens, 15% overlap = 120 tokens overlap

**Exam Tip**: Overlap prevents splitting concepts; increases chunk count per document.

---

## Delta Table Schema for Chunks

```sql
CREATE TABLE chunks (
  id STRING,                     -- Unique chunk identifier (required)
  text STRING,                   -- Original chunk text (recommended)
  embedding ARRAY<FLOAT>,        -- Vector embedding (required for Vector Search)
  doc_source STRING,             -- Document filename/URL (metadata for filtering)
  page_num INT,                  -- Page number (metadata)
  created_at TIMESTAMP,          -- Ingestion timestamp (metadata)
  chunk_index INT                -- Position in document (metadata)
);

-- Enable Change Data Feed for Vector Search auto-sync
ALTER TABLE chunks SET TBLPROPERTIES (delta.enableChangeDataFeed = true);
```

**Exam Tip**: `embedding` column name is convention; Vector Search looks for this by default.

---

## Embedding Model Selection

| Model | Accuracy | Speed | Use Case | Serving |
|-------|----------|-------|----------|----------|
| **BGE-large-en-v1.5** | Highest (English) | Fast (<100ms on Model Serving) | Technical docs, English-only | Databricks Model Serving |
| **E5-large** | High (multilingual) | Medium | Multi-language RAG | Databricks Model Serving |
| **all-MiniLM-L6-v2** | Medium | Fastest | Low-latency, less critical accuracy | Databricks Model Serving |
| **OpenAI ada-002** | High | Slow (API latency 200-500ms) | External API required | OpenAI API |

**Exam Tip**: Host on Databricks for low latency and cost; use API only if accuracy gain justifies latency.

---

## Vector Search Sync from Delta

**Incremental Sync (Recommended)**:
```python
# Enable CDF on source table
spark.sql("""
  ALTER TABLE chunks 
  SET TBLPROPERTIES (delta.enableChangeDataFeed = true)
""")

# Vector Search auto-syncs using CDF
# Detects: INSERT, UPDATE, DELETE
```

**Manual Refresh** (if CDF not enabled):
```python
vector_search_client.get_index(index_name).sync()
```

**Exam Tip**: CDF enables automatic incremental updates; without it, manual refresh or full rebuild required.

---

## Data Quality Filters (Apply BEFORE Indexing)

1. **Remove short chunks**: `LENGTH(text) < 50` (headers, page numbers)
2. **Remove boilerplate**: Cosine similarity >0.9 to template embedding
3. **Language filtering**: Detect and remove non-target language chunks
4. **Deduplication**: Remove exact or near-duplicate chunks

**Exam Tip**: Pre-indexing filters improve retrieval quality and reduce storage/compute costs.

---

## PDF/HTML Extraction Best Practices

| Content Type | Extraction Tool | Notes |
|--------------|----------------|-------|
| **Simple PDFs** | PyPDF2 | Text-based PDFs only |
| **PDFs with tables** | pdfplumber, Camelot | Preserves table structure |
| **Scanned PDFs** | OCR (Tesseract) + layout analysis | Extracts text from images |
| **HTML** | BeautifulSoup, Trafilatura | Remove nav, footer, boilerplate |
| **JSON** | Native parsing | Extract specific fields |

**Exam Tip**: Standard text extraction fails on tables; use specialized libraries.

---

## Common Exam Scenarios

**Scenario 1**: "Chunks lack context" → Increase chunk size OR increase overlap OR add surrounding chunk metadata  
**Scenario 2**: "Irrelevant results" → Decrease chunk size OR apply data quality filters OR use metadata filtering  
**Scenario 3**: "Table data not retrieved" → Fix PDF extraction to preserve table structure  
**Scenario 4**: "Slow retrieval" → Use Databricks-hosted embeddings (not API) OR reduce index size via filtering  
**Scenario 5**: "Index not updating" → Enable Change Data Feed on source Delta table
