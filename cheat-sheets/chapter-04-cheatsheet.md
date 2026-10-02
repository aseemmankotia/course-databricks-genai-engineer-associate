# Multi-Stage Chains and Prompt Engineering - Exam Cheat Sheet

## LangChain Chain Types (Heavily Tested)

| Chain Type | Use Case | Exam Keyword |
|------------|----------|-------------|
| **SequentialChain** | Multi-step workflows (extract → summarize → classify) | Multi-step processing |
| **RetrievalQA** | Question answering with document retrieval (RAG) | Retrieval-augmented generation |
| **LLMChain** | Single LLM call with a prompt template | Simple generation |

**Exam Trap**: RetrievalQA is ONLY for retrieval patterns. For non-retrieval multi-step workflows, use SequentialChain.

## Databricks Foundation Model API in LangChain

```python
# Correct syntax (exam tests this)
from langchain_community.chat_models import ChatDatabricks

llm = ChatDatabricks(
    endpoint="databricks-llama-2-70b-chat",  # endpoint name
    max_tokens=512,
    temperature=0.1
)
```

**Exam Note**: Use `ChatDatabricks` (not `OpenAI`, not generic `LLM` class). Parameter is `endpoint` (not `endpoint_name`).

## Prompt Engineering Techniques

| Technique | Definition | Example | When to Use |
|-----------|------------|---------|-------------|
| **Few-shot** | 2-3 input-output examples in prompt | "Classify: 'Great product!' → Positive\nClassify: 'Terrible' → Negative\nClassify: {input}" | Classification, extraction, formatting |
| **Chain-of-thought** | Instruct model to show reasoning steps | "Let's solve step by step:\nStep 1: ...\nStep 2: ..." | Math, logic, complex reasoning |
| **Few-shot + CoT** | Examples include reasoning steps | Show 2-3 examples with step-by-step work | Complex classification requiring explanation |

**Exam Numbers**: Few-shot = **2-3 examples** (not 1, not 10+). More examples = higher cost/latency.

## MLflow Integration (Heavily Tested)

```python
import mlflow

# Log a SequentialChain (exam syntax)
mlflow.langchain.log_model(
    lc_model=my_sequential_chain,
    artifact_path="model",
    input_example={"text": "sample input"}
)
```

**Exam Traps**:
- Use `mlflow.langchain.log_model` (NOT `mlflow.log_model` or `mlflow.pyfunc.log_model`)
- Logs the ENTIRE chain (all steps) as one artifact
- Captures prompts, LLM configs, and dependencies

## SequentialChain Pattern (Exam Favorite)

```python
from langchain.chains import SequentialChain, LLMChain
from langchain.prompts import PromptTemplate

# Step 1: Extract entities
extract_chain = LLMChain(llm=llm, prompt=extract_prompt, output_key="entities")

# Step 2: Summarize
summary_chain = LLMChain(llm=llm, prompt=summary_prompt, output_key="summary")

# Step 3: Classify
classify_chain = LLMChain(llm=llm, prompt=classify_prompt, output_key="category")

# Combine
overall_chain = SequentialChain(
    chains=[extract_chain, summary_chain, classify_chain],
    input_variables=["review_text"],
    output_variables=["entities", "summary", "category"]
)
```

**Exam Note**: Output of step N becomes input to step N+1. Log the entire `overall_chain` with `mlflow.langchain.log_model`.

## RetrievalQA with Databricks Vector Search

```python
from langchain.chains import RetrievalQA
from databricks.vector_search.client import VectorSearchClient

# Vector Search as retriever
vs_client = VectorSearchClient()
retriever = vs_client.get_index(
    endpoint_name="vector_search_endpoint",
    index_name="my_index"
).as_retriever()

# RetrievalQA chain
qa_chain = RetrievalQA.from_chain_type(
    llm=ChatDatabricks(endpoint="databricks-llama-2-70b-chat"),
    chain_type="stuff",  # exam uses 'stuff' type
    retriever=retriever
)
```

**Exam Keyword**: RAG (Retrieval-Augmented Generation) = RetrievalQA pattern.

## Prompt Template Construction

```python
from langchain.prompts import PromptTemplate, ChatPromptTemplate

# PromptTemplate (single text)
prompt = PromptTemplate(
    input_variables=["product", "review"],
    template="Summarize this review for {product}: {review}"
)

# ChatPromptTemplate (structured messages)
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "Classify: {text}")
])
```

**Exam Comparison**: Use `ChatPromptTemplate` with chat models (`ChatDatabricks`). Use `PromptTemplate` for completion models.

## Common Exam Scenarios

| Scenario | Solution | Why |
|----------|----------|-----|
| Multi-step workflow (extract → summarize → classify) | SequentialChain | Chains multiple LLMChains |
| Question answering over docs in Vector Search | RetrievalQA | Built for RAG pattern |
| Improve classification with examples | Few-shot (2-3 examples) | Shows desired output format |
| Complex reasoning task (math problems) | Chain-of-thought | Forces step-by-step reasoning |
| Version a multi-step chain | `mlflow.langchain.log_model(SequentialChain)` | Logs entire workflow |
| Use Foundation Model API in LangChain | `ChatDatabricks(endpoint=...)` | Databricks-specific LLM class |

## Key Exam Numbers
- **Few-shot examples**: 2-3 (exam standard)
- **Chain-of-thought**: Shows intermediate steps, improves reasoning accuracy
- **SequentialChain**: N steps, output of step i → input of step i+1
- **RetrievalQA**: 1 retriever + 1 LLM = RAG pattern
