# Chapter 6: MLflow & Model Serving Cheat Sheet

## MLflow Flavors for GenAI

| Flavor | Use Case | Command | Notes |
|--------|----------|---------|-------|
| **langchain** | LangChain chains/agents | `mlflow.langchain.log_model(chain, artifact_path)` | Auto-captures dependencies, chain structure |
| **pyfunc** | Custom Python models | `mlflow.pyfunc.log_model(artifact_path, python_model=CustomModel())` | Maximum flexibility, custom inference logic |
| **transformers** | HuggingFace models | `mlflow.transformers.log_model(transformers_model, artifact_path)` | NOT supported by Model Serving—use pyfunc wrapper |
| **deployments** | External LLM APIs | Configure MLflow Deployments endpoint | For OpenAI, Anthropic, Cohere integration |

**Exam Tip**: If question mentions LangChain chain → use langchain flavor. Custom logic or non-LangChain → use pyfunc.

---

## Unity Catalog Model Registry

### Three-Level Namespace
```python
# Format: catalog.schema.model_name
mlflow.register_model(
    model_uri="runs:/abc123/model",
    name="prod_catalog.genai_models.chatbot_v1"
)
```

**Key Commands**:
- Register model: `mlflow.register_model(model_uri, name)`
- Load model: `mlflow.pyfunc.load_model("models:/catalog.schema.model/version")`
- Model URI formats: `runs:/<run_id>/model` OR `models:/catalog.schema.model/version`

### Model Versioning & Aliases

| Alias Pattern | Purpose | Setting Command |
|---------------|---------|------------------|
| @champion | Production version | `client.set_registered_model_alias("cat.sch.mod", "champion", version=3)` |
| @challenger | A/B test candidate | `client.set_registered_model_alias("cat.sch.mod", "challenger", version=5)` |
| @staging | Pre-prod testing | Custom alias |

**Exam Tip**: Aliases enable version switching without code changes. @champion/@challenger = common A/B testing pattern.

---

## Databricks Model Serving

### Endpoint Configuration

| Setting | Options | Use Case |
|---------|---------|----------|
| **Compute Size** | Small, Medium, Large | Small=dev/low-traffic; Large=production/high-traffic |
| **Scale-to-Zero** | Enabled/Disabled | Enabled=cost savings but cold starts; Disabled=always warm |
| **Min Instances** | 0, 1, 2+ | 0=scale-to-zero; ≥1=no cold starts |
| **Max Instances** | 1-10+ | Controls max autoscaling capacity |

**Cold Start Reality**: 30-60 seconds when scaling from 0. Solution: set min_instances=1 for latency-sensitive apps.

### REST API Query Format
```python
import requests

url = "https://<workspace>.cloud.databricks.com/serving-endpoints/<endpoint-name>/invocations"
headers = {"Authorization": f"Bearer {token}"}
data = {"dataframe_records": [{"prompt": "Hello"}]}

response = requests.post(url, json=data, headers=headers)
```

**Key Points**:
- Path MUST include `/invocations`
- Authorization header: `Bearer <token>`
- Input format: `dataframe_records` for pandas-style or `inputs` for tensor-style

### Traffic Splitting (A/B Testing)
```python
# Configure 80% to @champion, 20% to @challenger
traffic_config = {
    "@champion": 80,
    "@challenger": 20
}
# Apply via endpoint update API
```

**Exam Tip**: Traffic split uses aliases or version numbers. Percentages must sum to 100.

---

## MLflow Deployments (External LLMs)

**Purpose**: Wrap external LLM APIs (OpenAI, Anthropic) as MLflow endpoints

**Configuration**:
```yaml
routes:
  - name: chat-gpt4
    route_type: llm/v1/chat
    model:
      provider: openai
      name: gpt-4
      config:
        openai_api_key: "{{secrets/scope/key}}"
```

**Query Format**:
```python
import mlflow.deployments
client = mlflow.deployments.get_deploy_client("databricks")
response = client.predict(
    endpoint="chat-gpt4",
    inputs={"messages": [{"role": "user", "content": "Hi"}]}
)
```

**Exam Distinction**:
- MLflow Deployments = external API wrapper
- Model Serving = MLflow-logged model deployment

---

## Common Exam Scenarios

| Scenario | Solution |
|----------|----------|
| Log LangChain chain | `mlflow.langchain.log_model(chain, "model")` |
| Register to Unity Catalog | `mlflow.register_model("runs:/id/model", "cat.sch.mod")` |
| Eliminate cold starts | Set min_instances=1 in endpoint config |
| A/B test two versions | Use aliases + traffic splitting |
| Integrate OpenAI API | Configure MLflow Deployments endpoint |
| Query served model | POST to `/serving-endpoints/<name>/invocations` |
| Model not in correct catalog | Re-register using `mlflow.register_model()` with new name |

**Critical Numbers**:
- Cold start delay: 30-60 seconds (when min_instances=0)
- Autoscaling range: min_instances to max_instances
- Model namespace levels: 3 (catalog.schema.model)
