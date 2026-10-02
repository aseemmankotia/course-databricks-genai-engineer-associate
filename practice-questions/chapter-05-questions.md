## Chapter 5: Agents, Tools, and Function Calling — Practice Questions

**Q1.** A company is building a LangChain agent on Databricks that needs to query Delta tables, perform calculations, and make API calls to external services. The agent must explain its reasoning before each action. Which agent type BEST meets these requirements?

A) Plan-and-Execute agent with custom tool wrappers
B) Zero-shot agent with Databricks SQL functions only
C) Conversational agent with memory buffer and external API tools
D) ReAct agent with Python functions wrapped as LangChain Tool objects

<details>
<summary>Answer</summary>

**Correct: D**

ReAct (Reason + Act) agent explicitly implements observe → think (reason) → act pattern, showing reasoning before each tool use. Python functions wrapped as Tool objects handle Delta queries, calculations, and API calls in unified format.

**Why the others are wrong:**
- Plan-and-Execute creates full plan upfront, doesn't show reasoning before each step; over-engineered for this use case
- Zero-shot agent is deprecated LangChain pattern; limiting to SQL functions only ignores requirement for calculations and API calls
- Conversational agent focuses on maintaining chat history, not on reasoning visibility; doesn't inherently show think-before-act pattern

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An engineer defines a tool for a Databricks agent to query customer orders from a Delta table. The tool randomly fails with 'Table not found' errors due to network timeouts. Which approach BEST handles this scenario in the agent execution loop?

A) Configure agent with max_iterations=1 to fail fast and alert developer
B) Use LangChain's RetryOutputParser to automatically retry failed tool calls up to 3 times
C) Implement exponential backoff in the tool function itself and raise exception after 3 failures
D) Wrap the tool function with try-except and return error message string to agent; agent decides whether to retry

<details>
<summary>Answer</summary>

**Correct: D**

Returning error message as string keeps agent in control; agent can reason about the error and decide next action (retry, try different approach, or ask for clarification). Follows ReAct pattern where agent observes results including errors.

**Why the others are wrong:**
- max_iterations=1 prevents any retry logic; fails on first error without giving agent chance to recover or try alternative approach
- RetryOutputParser retries parsing of agent output, not tool execution failures; wrong use case for this parser
- Implementing retry in tool itself removes agent's ability to reason about failures; agent should decide retry strategy based on context

</details>

---

**Q3.** A Databricks agent uses DBRX Instruct for function calling. The tool schema includes a 'price_threshold' parameter defined as: {"type": "number", "description": "Minimum price in USD"}. The agent passes price_threshold=50 to the tool but the tool receives the string '50'. What is the root cause?

A) The tool wrapper must parse string arguments to correct types; LLM output is always strings that match the schema structure but not Python types
B) DBRX Instruct doesn't support function calling; use GPT-4 or Llama 3.1 instead
C) JSON Schema type 'number' must be changed to 'integer' for whole numbers
D) The input schema needs additionalProperties: false to enforce strict typing

<details>
<summary>Answer</summary>

**Correct: A**

Function calling models return JSON-formatted strings matching schema structure; the tool wrapper (LangChain Tool or custom code) must convert string values to declared types. JSON Schema describes expected format, not Python runtime types.

**Why the others are wrong:**
- DBRX Instruct does support function calling via Databricks Foundation Model APIs; not a model limitation
- JSON Schema integer vs number doesn't affect this issue; both would still arrive as string '50' from LLM output requiring parsing
- additionalProperties controls whether extra fields are allowed; doesn't affect type conversion of declared fields

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** An agent on Databricks needs to execute SQL queries against Delta tables as one of its tools. Which implementation follows Databricks best practices for SQL tools in agents?

A) Use LangChain's built-in SQLDatabaseToolkit with Databricks JDBC connection string
B) Implement a custom agent that generates SQL directly without tool abstraction
C) Use Databricks SQL Warehouse REST API calls wrapped in a generic HTTP request tool
D) Create a Python function that uses spark.sql() and wrap it as a LangChain Tool with proper input schema

<details>
<summary>Answer</summary>

**Correct: D**

spark.sql() is native Databricks API with access to Delta tables; wrapping as LangChain Tool with schema provides agent clear interface, proper documentation, and type safety. Follows LangChain tool pattern while using Databricks-native execution.

**Why the others are wrong:**
- SQLDatabaseToolkit designed for external databases via SQLAlchemy; adds unnecessary complexity and doesn't leverage Databricks Spark session
- Generating SQL without tool abstraction bypasses agent's tool selection logic; loses observability and error handling benefits of tool pattern
- REST API adds network overhead and authentication complexity when spark.sql() provides direct access; overcomplicates simple requirement

</details>

---

**Q5.** A company logs their LangChain ReAct agent to MLflow using mlflow.langchain.log_model. When loading the model for inference, the agent fails because custom tools are not available. Which statement about MLflow logging of agents is correct?

A) mlflow.langchain.log_model logs agent configuration but not tool implementations; tools must be registered separately as model dependencies
B) mlflow.langchain.log_model logs the agent and automatically serializes all attached Tool objects including their Python function code
C) Custom tools must be packaged as pip-installable packages and listed in conda.yaml for the MLflow model
D) LangChain agents cannot be logged to MLflow; use mlflow.pyfunc.log_model with custom wrapper instead

<details>
<summary>Answer</summary>

**Correct: B**

mlflow.langchain.log_model serializes entire agent chain including Tool objects with their wrapped Python functions using cloudpickle. Tools are restored automatically when loading the model.

**Why the others are wrong:**
- Tools ARE included in serialization; this is incorrect statement of MLflow LangChain logging behavior
- While best practice for production, pip packages aren't required; MLflow serializes tool code directly via cloudpickle for standard Python functions
- LangChain agents are fully supported by mlflow.langchain.log_model; this is the recommended approach, not pyfunc wrapper

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** An agent using GPT-4 for function calling consistently selects the wrong tool from a set of 5 available tools with similar purposes. Tool descriptions are all 2-3 words. What is the MOST likely cause?

A) GPT-4 temperature is set too high causing random tool selection
B) The tool input schemas are missing required fields causing model confusion
C) GPT-4 requires tools to be sorted alphabetically in the schema for proper selection
D) Tool descriptions are too brief; function calling models need detailed descriptions explaining when to use each tool and example scenarios

<details>
<summary>Answer</summary>

**Correct: D**

Function calling models use tool descriptions as primary signal for selection. Brief descriptions don't provide enough context to differentiate similar tools. Best practice: 1-2 sentence descriptions with when-to-use guidance and examples.

**Why the others are wrong:**
- Temperature affects output randomness but function calling uses structured output mode with lower effective randomness; unlikely root cause for consistent wrong selection
- Missing required fields would cause execution errors after selection, not wrong selection; selection happens before parameter validation
- Tool order is irrelevant; models select based on semantic matching of description to task, not position or alphabetical order

</details>

---

**Q7.** A ReAct agent executes the following loop: User asks question → Agent thinks → Agent calls tool A → Tool A returns data → Agent thinks → Agent calls tool B → Tool B returns answer → Agent responds to user. At which point should the agent's thought process be logged for exam debugging?

A) Only after final tool call before generating user response
B) Only once at the end of the complete agent execution
C) After each 'Agent thinks' step and after each tool execution result
D) Before first tool call and after last tool call only

<details>
<summary>Answer</summary>

**Correct: C**

ReAct pattern is observe → think → act → observe → think → act. Logging after each think step shows reasoning; logging after each tool result shows observations. Both are needed to debug agent decision-making and understand why specific tools were chosen.

**Why the others are wrong:**
- Missing intermediate reasoning steps; can't debug why tool A was called before tool B or if agent considered alternatives
- Single log at end loses step-by-step decision trail; can't identify where agent logic diverged from expected behavior
- Misses all intermediate think steps; only captures first reasoning and final reasoning, losing entire middle decision chain

</details>

---

**Q8.** An organization needs to build an agent on Databricks that first creates a complete plan of all steps, then executes each step without re-planning. The agent must query Delta tables, call external APIs, and perform calculations. Which approach is correct?

A) Use Plan-and-Execute agent with Databricks DBRX as planner LLM and separate executor agent with tool access
B) Use ReAct agent with max_iterations set to number of planned steps
C) Create custom agent that generates plan string first, then uses ReAct agent to execute plan steps
D) Use LangChain's built-in OpenAI Functions agent with planning prompt

<details>
<summary>Answer</summary>

**Correct: A**

Plan-and-Execute agent architecture separates planning phase (generate all steps upfront using planner LLM) from execution phase (executor agent with tools runs each step). DBRX can serve as planner; executor gets tool access for Delta queries, APIs, calculations.

**Why the others are wrong:**
- ReAct re-plans after each observation; setting max_iterations doesn't change its adaptive nature, just limits total steps
- This describes Plan-and-Execute but with unnecessary custom wrapper; LangChain provides Plan-and-Execute agent type directly
- OpenAI Functions agent is similar to ReAct; adding planning prompt doesn't fundamentally change it to plan-then-execute architecture

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

