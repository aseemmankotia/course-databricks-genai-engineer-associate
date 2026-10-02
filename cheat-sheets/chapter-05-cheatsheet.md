# Chapter 5: Agents, Tools, and Function Calling - Quick Reference

## Agent Types

| Agent Type | Pattern | Use When | Key Characteristic |
|------------|---------|----------|--------------------|
| **ReAct** | Observe → Think → Act → repeat | Dynamic tasks, needs adaptation | Shows reasoning before each action |
| **Plan-and-Execute** | Plan all steps → Execute each | Known sequence, no mid-course changes | Plans once upfront, then executes |
| **Zero-shot** | (Deprecated) | Legacy codebases only | Single-step reasoning |

## ReAct Decision Loop

```
1. OBSERVE: User input or tool result
2. THINK: Agent reasons about next action (logged)
3. ACT: Call tool or respond to user
4. Repeat from OBSERVE with tool result
```

**Exam Key**: ReAct = Reason + Act. Reasoning visibility is the distinguishing feature.

## Tool Definition - Required Components

```python
from langchain.tools import Tool

tool = Tool(
    name="query_orders",  # REQUIRED: identifier
    description="Use when you need to find customer orders. Input: customer_id as string.",  # REQUIRED: when to use + input format
    func=lambda customer_id: spark.sql(f"SELECT * FROM orders WHERE customer_id='{customer_id}'").toPandas()  # REQUIRED: callable
)
```

**Best Practice**: Description = 1-2 sentences explaining when to use + example input/output

## Function Calling - Tool Schema Format

```python
# JSON Schema for tool parameters
schema = {
    "type": "object",
    "properties": {
        "price_threshold": {
            "type": "number",
            "description": "Minimum price in USD"
        }
    },
    "required": ["price_threshold"]
}
```

**Exam Trap**: LLM returns JSON strings matching schema structure. Tool must parse strings to Python types (e.g. "50" → 50).

## Databricks-Specific Tool Patterns

### SQL Query Tool
```python
def query_delta_table(query: str) -> str:
    try:
        result = spark.sql(query).toPandas()
        return result.to_string()
    except Exception as e:
        return f"Error: {str(e)}"  # Return error to agent, let agent decide retry

sql_tool = Tool(name="sql_query", description="Execute SQL on Delta tables", func=query_delta_table)
```

**Exam Key**: Use `spark.sql()` not REST APIs when running on Databricks. Return errors as strings to agent.

## Error Handling Best Practices

| Approach | Implementation | When to Use |
|----------|----------------|-------------|
| **Agent-controlled retry** | Tool returns error string; agent decides next step | Recommended - keeps agent in control |
| **Tool-level retry** | Tool implements retry logic internally | Only for transient network errors |
| **Fail fast** | Set max_iterations=1 | Development/debugging only |

**Pattern**:
```python
def tool_with_error_handling(param):
    try:
        return do_operation(param)
    except SpecificError as e:
        return f"Error: {e}. Try alternative approach."  # Agent can reason about this
```

## MLflow Agent Logging

```python
import mlflow

# Log agent with all tools included
with mlflow.start_run():
    mlflow.langchain.log_model(
        lc_model=agent,  # Agent object with tools attached
        artifact_path="agent"
    )
    # Tools are serialized automatically via cloudpickle

# Load agent (tools restored automatically)
loaded_agent = mlflow.langchain.load_model("runs:/<run_id>/agent")
```

**Exam Key**: `mlflow.langchain.log_model` serializes agent + tools + tool function code. No separate tool registration needed.

## Model Support for Function Calling

| Model | Function Calling Support | Access via |
|-------|-------------------------|------------|
| **DBRX Instruct** | ✅ Yes | Databricks Foundation Model APIs |
| **GPT-4** | ✅ Yes | External API (OpenAI) |
| **Llama 3.1** | ✅ Yes | Databricks or external |
| **Llama 2** | ❌ No | N/A |

**Exam Tip**: DBRX Instruct supports function calling; distinguish by native Databricks availability.

## Common Exam Traps

1. **Tool Selection Failure**: Caused by brief descriptions. Fix: Add 1-2 sentence descriptions with when-to-use guidance.
2. **Type Mismatches**: LLM returns strings matching JSON Schema. Fix: Parse in tool function.
3. **Retry Logic Placement**: Don't retry in tool if agent should decide. Fix: Return error string to agent.
4. **MLflow Tool Loss**: Thinking tools aren't serialized. Fix: They are - automatically via cloudpickle.
5. **Agent Type Confusion**: Using ReAct when Plan-and-Execute needed. Fix: Plan-and-Execute = plan once; ReAct = adaptive loop.

## Quick Decision Tree

**Need to show reasoning?** → ReAct  
**Need complete plan upfront?** → Plan-and-Execute  
**Querying Delta tables?** → spark.sql() tool  
**Logging agent?** → mlflow.langchain.log_model (includes tools)  
**Tool failed?** → Return error string, let agent decide retry  
**Tool description?** → 1-2 sentences: when to use + input format
