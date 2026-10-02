# Chapter 7 Lab: Foundation Model APIs and Production Serving

**Estimated time:** 20 minutes

## Exercise

Lab: Deploy DBRX with pay-per-token via Foundation Model API, send 10 test queries, check token usage in Databricks billing console. Then configure an external Azure OpenAI endpoint, route requests through Databricks Model Serving.

## Key takeaway

Pay-per-token for unpredictable traffic, provisioned throughput for high-volume predictable apps; batch inference for offline Delta table processing; external endpoints need API keys in Secrets, not hardcoded.
