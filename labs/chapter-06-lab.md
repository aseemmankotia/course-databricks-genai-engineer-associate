# Chapter 6 Lab: Packaging and Serving with MLflow and Model Serving

**Estimated time:** 25 minutes

## Exercise

Lab: Log a RAG chain with mlflow.langchain.log_model, register it in Unity Catalog (catalog: main, schema: genai, model: rag_v1), deploy to a Model Serving endpoint (Small compute, scale-to-zero enabled), send a test query via REST API.

## Key takeaway

Log chains with mlflow.langchain to preserve structure, register in Unity Catalog with three-level namespace, deploy to Model Serving with correct compute size for latency SLA; test with REST API.
