## Chapter 6: Packaging and Serving with MLflow and Model Serving — Practice Questions

**Q1.** A team has built a RAG application using LangChain with a custom retriever and an OpenAI LLM. They need to log the entire chain to MLflow for versioning and deployment. Which approach should they use?

A) Use mlflow.pyfunc.log_model() with a custom Python wrapper class that implements the chain logic
B) Use mlflow.transformers.log_model() since it handles both the retriever and LLM components
C) Use mlflow.deployments.log_model() to wrap the OpenAI API calls as an MLflow model
D) Use mlflow.langchain.log_model() to log the LangChain chain directly with automatic dependency tracking

<details>
<summary>Answer</summary>

**Correct: D**

mlflow.langchain.log_model() is specifically designed for LangChain objects (chains, agents, retrievers). It automatically captures the chain structure, dependencies, and LangChain version, making it the most appropriate choice for logging LangChain-based applications.

**Why the others are wrong:**
- While mlflow.pyfunc.log_model() works, it requires manual wrapper implementation and loses LangChain-specific metadata and inference optimizations that the langchain flavor provides automatically
- mlflow.transformers.log_model() is for HuggingFace Transformers models only, not LangChain chains. It cannot handle custom retrievers or LangChain's chain abstraction
- mlflow.deployments is not a logging method—it's for creating deployment endpoints that wrap external APIs. It doesn't log models to the registry

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A data scientist needs to register a logged GenAI model in Unity Catalog's model registry. The model was logged in experiment '/Shared/rag-experiments' with run_id 'abc123'. Which MLflow code correctly registers this model to the registry path 'prod_catalog.genai_models.rag_v1'?

A) mlflow.unity_catalog.register_model(run_id='abc123', catalog='prod_catalog', schema='genai_models', model='rag_v1')
B) mlflow.register_model(model_uri='runs:/abc123/model', name='prod_catalog.genai_models.rag_v1')
C) mlflow.models.register(experiment='/Shared/rag-experiments', run='abc123', path='prod_catalog.genai_models.rag_v1')
D) mlflow.create_model_version(name='prod_catalog.genai_models.rag_v1', source='experiments/abc123/model')

<details>
<summary>Answer</summary>

**Correct: B**

mlflow.register_model() with model_uri='runs:/<run_id>/model' and the three-level namespace (catalog.schema.model) as the name is the standard way to register models to Unity Catalog. The three-level name automatically targets Unity Catalog registry.

**Why the others are wrong:**
- mlflow.unity_catalog.register_model() is not a valid MLflow API method. Unity Catalog registration uses the standard mlflow.register_model() with three-level naming
- mlflow.models.register() is not a valid function. The correct function is mlflow.register_model(), and it uses model_uri not experiment path
- mlflow.create_model_version() is used to add versions to existing registered models, not for initial registration. Also, source format is incorrect (should be runs:/ URI)

</details>

---

**Q3.** A company deployed a LangChain-based Q&A model to a Databricks Model Serving endpoint with 'Small' compute size and scale-to-zero enabled. Users report that the first query after idle periods takes 45 seconds, but subsequent queries are fast. What is the BEST solution to reduce first-query latency while minimizing costs?

A) Upgrade to 'Large' compute size to handle cold starts faster
B) Enable endpoint autoscaling with minimum instances set to 1 instead of 0
C) Disable scale-to-zero to keep the endpoint always active, ensuring zero cold start delays
D) Switch from langchain flavor to pyfunc flavor for faster model loading

<details>
<summary>Answer</summary>

**Correct: B**

Setting minimum instances to 1 with autoscaling keeps one instance warm at all times, eliminating cold starts while still allowing scale-down during very low traffic (better than disabling scale-to-zero entirely). This balances latency requirements with cost optimization.

**Why the others are wrong:**
- Larger compute size doesn't solve cold start delays—it only increases per-instance compute power. The 45-second delay is from model loading, not inference compute. This also significantly increases costs without addressing root cause
- Disabling scale-to-zero completely eliminates cost savings during idle periods. While it fixes cold starts, it's not cost-optimal. Setting min instances to 1 achieves similar latency with better cost control during low traffic
- Model flavor doesn't significantly impact cold start time. The delay is from endpoint initialization and model loading, not the MLflow flavor. Both langchain and pyfunc have similar cold start characteristics

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** An MLOps team maintains two versions of a GenAI model in Unity Catalog: version 3 (production) and version 5 (testing). They want to route 90% of traffic to version 3 and 10% to version 5 for A/B testing. What is the correct approach using Databricks Model Serving?

A) Create two separate endpoints and use an external load balancer to split traffic 90/10
B) Use model aliases: set @champion to version 3 and @challenger to version 5, then configure endpoint traffic splitting between aliases
C) Deploy version 3 with 9 instances and version 5 with 1 instance using autoscaling min/max settings
D) Create a single endpoint pointing to version 3, then use mlflow.deployments client-side logic to randomly route 10% to version 5

<details>
<summary>Answer</summary>

**Correct: B**

Databricks Model Serving endpoints support native traffic splitting between model versions using aliases. Setting @champion and @challenger aliases and configuring percentage-based traffic split is the built-in, recommended approach for A/B testing without external infrastructure.

**Why the others are wrong:**
- Creating separate endpoints requires managing two endpoints, duplicates costs, and needs external load balancing infrastructure. Model Serving has built-in traffic splitting that's simpler and more cost-effective
- Instance count controls scaling for handling load, not traffic distribution between model versions. This approach doesn't actually route traffic differently—all requests would go to version 3 with this configuration
- Client-side routing is error-prone, requires custom code in every client, and doesn't provide centralized metrics. Model Serving's server-side traffic splitting is the correct solution for A/B testing

</details>

---

**Q5.** A data engineer needs to serve an external OpenAI GPT-4 model through Databricks infrastructure for governance and cost tracking. The model should be accessible via MLflow's standard inference API. Which solution meets these requirements?

A) Log the OpenAI model using mlflow.openai.log_model(), register it to Unity Catalog, and deploy to Model Serving
B) Create an MLflow Deployments endpoint configured with OpenAI provider, then reference it as an mlflow-deployments:/ URI in downstream applications
C) Use mlflow.pyfunc.log_model() with a wrapper class that calls the OpenAI API, then deploy the wrapper to Model Serving
D) Configure a Databricks External Models endpoint pointing to OpenAI's API directly

<details>
<summary>Answer</summary>

**Correct: B**

MLflow Deployments (formerly MLflow Gateway) is specifically designed to wrap external LLM APIs (OpenAI, Anthropic, etc.) as MLflow-compatible endpoints. It provides centralized governance, cost tracking, and a standard MLflow interface (mlflow-deployments:/ URI) for external models.

**Why the others are wrong:**
- mlflow.openai.log_model() is not a valid MLflow function. OpenAI models are external APIs, not artifacts to log. MLflow Deployments is the correct way to integrate external LLM providers
- While this works technically, it's unnecessary complexity. MLflow Deployments handles the wrapping, credential management, and logging automatically without requiring custom code. This approach also duplicates functionality that Deployments provides
- Databricks External Models is not the standard MLflow pattern for this use case. MLflow Deployments provides the MLflow-native integration with governance features and is the recommended approach per Databricks documentation

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A GenAI model registered as 'main.ml_models.chatbot' version 4 is deployed to a Model Serving endpoint named 'chatbot-prod'. A developer needs to query this endpoint using Python. Which code correctly sends a prediction request?

A) from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
w.serving_endpoints.query(name='chatbot-prod', inputs={'prompt': 'Hello'})
B) import mlflow.deployments
client = mlflow.deployments.get_deploy_client('databricks')
client.predict(endpoint='chatbot-prod', inputs={'prompt': 'Hello'})
C) import requests
url = 'https://<workspace>.cloud.databricks.com/serving-endpoints/chatbot-prod/invocations'
requests.post(url, json={'dataframe_records': [{'prompt': 'Hello'}]}, headers={'Authorization': f'Bearer {token}'})
D) import mlflow
loaded_model = mlflow.pyfunc.load_model('models:/main.ml_models.chatbot/4')
loaded_model.predict({'prompt': 'Hello'})

<details>
<summary>Answer</summary>

**Correct: C**

Direct REST API using requests.post() to /serving-endpoints/<endpoint-name>/invocations with proper Authorization header is the standard way to query Databricks Model Serving endpoints. The input format (dataframe_records) matches the expected MLflow serving format.

**Why the others are wrong:**
- WorkspaceClient().serving_endpoints.query() is not the correct method. The Databricks SDK's serving endpoint methods are for management (create, update), not inference. Inference uses REST API or mlflow.deployments client
- mlflow.deployments.get_deploy_client() is for MLflow Deployments (external LLM gateway), not Databricks Model Serving endpoints. While similar, they're different services with different client APIs
- mlflow.pyfunc.load_model() loads the model locally for inference—it doesn't call the deployed endpoint. This runs the model in the current Python process, not via the Model Serving infrastructure

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A team logged a HuggingFace transformer model for text generation using mlflow.transformers.log_model(). They now need to deploy it to Model Serving, but the endpoint fails with 'Unsupported model flavor' error. What is the MOST likely cause and solution?

A) The transformers model requires GPU instances; change endpoint compute configuration to GPU-enabled size
B) Model Serving only supports langchain and pyfunc flavors for GenAI models; the model must be wrapped in a LangChain chain first
C) Databricks Model Serving doesn't support transformers flavor; re-log the model using mlflow.pyfunc.log_model() with a custom loader
D) The model's transformers library version is incompatible; update the logged model's conda.yaml to match the endpoint runtime

<details>
<summary>Answer</summary>

**Correct: C**

Databricks Model Serving has limited flavor support. The transformers flavor is not natively supported for serving. The recommended approach is to log HuggingFace models using mlflow.pyfunc.log_model() with a custom Python wrapper that loads the transformers model, which is fully supported by Model Serving.

**Why the others are wrong:**
- While GPU may be needed for performance, it's not the cause of 'Unsupported model flavor' error. The error indicates the serving infrastructure doesn't recognize the transformers flavor, which is a compatibility issue not a resource issue
- Model Serving supports multiple flavors including pyfunc, langchain, and python_function, but transformers is not one of them. Wrapping in LangChain is unnecessary—pyfunc wrapper is the direct solution
- Conda environment mismatches cause dependency errors, not 'Unsupported flavor' errors. The flavor itself (transformers) isn't supported regardless of library versions. The solution is changing the flavor, not updating dependencies

</details>

---

**Q8.** An organization has a policy requiring all production GenAI models to be in the 'prod' catalog. A data scientist accidentally registered a model as 'dev.genai.summarizer' but it needs to be promoted to production. What is the correct approach to move this model to 'prod.genai.summarizer'?

A) Use mlflow.register_model() to register the same model artifact from the original run to 'prod.genai.summarizer' as a new model
B) Use Unity Catalog SQL: ALTER MODEL dev.genai.summarizer RENAME TO prod.genai.summarizer
C) Copy the model files from dev catalog storage to prod catalog storage, then use mlflow.update_model() to change the registered path
D) Create a new version in 'prod.genai.summarizer' by downloading the model from dev catalog and re-logging it to a new MLflow run

<details>
<summary>Answer</summary>

**Correct: A**

mlflow.register_model() can register the same model artifact (using runs:/<run_id>/model URI from the original training run) to a different model name in Unity Catalog. This creates a new registered model in the prod catalog while preserving the original artifact and lineage.

**Why the others are wrong:**
- Unity Catalog doesn't support renaming/moving models across catalogs with ALTER MODEL. Models are registered to specific three-level namespaces and cannot be moved—they must be re-registered
- Model Registry stores metadata, not files, and mlflow.update_model() is not a valid function. Direct file manipulation doesn't update the Model Registry. The correct approach is re-registering via MLflow API
- Re-logging breaks lineage tracing to the original experiment run. The correct approach preserves lineage by registering the original artifact (via runs:/ URI) to the new name, maintaining the connection to source training run

</details>

---

