# Chapter 5 Lab: Agents, Tools, and Function Calling

**Estimated time:** 30 minutes

## Exercise

Lab: Build a ReAct agent with 2 tools: (1) query_vector_search (retrieves chunks), (2) calculate (Python eval for math). Ask the agent a question requiring both retrieval and calculation. Trace the agent's tool calls. Log with MLflow.

## Key takeaway

Agents use tools for external data/actions; tool descriptions must be precise or agent picks wrong tool; log agents with mlflow.langchain including tools; few-shot examples teach agents correct tool usage patterns.
