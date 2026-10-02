# Chapter 9 Lab: Evaluation and Monitoring of GenAI Applications

**Estimated time:** 25 minutes

## Exercise

Lab: Run mlflow.evaluate on a RAG system with 50 question-answer pairs. Use LLM-as-a-judge to compute faithfulness and answer_relevance. Enable Lakehouse Monitoring on the inference table of a deployed Model Serving endpoint. Check drift dashboard.

## Key takeaway

mlflow.evaluate with LLM-as-a-judge for offline testing; faithfulness = grounding, answer_relevance = on-topic, context precision/recall = retrieval quality; Lakehouse Monitoring detects statistical drift in production.
