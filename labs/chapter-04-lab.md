# Chapter 4 Lab: Multi-Stage Chains and Prompt Engineering

**Estimated time:** 25 minutes

## Exercise

Lab: Build a 2-stage LangChain SequentialChain: Stage 1 retrieves chunks from Vector Search, Stage 2 generates a summary with chain-of-thought reasoning. Log the chain with MLflow. Test with a sample query.

## Key takeaway

LangChain on Databricks uses Foundation Model APIs as LLM endpoints; log chains with mlflow.langchain flavor; use few-shot for examples, chain-of-thought for reasoning, not both for simple tasks.
