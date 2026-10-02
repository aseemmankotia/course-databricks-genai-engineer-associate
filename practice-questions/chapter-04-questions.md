## Chapter 4: Multi-Stage Chains and Prompt Engineering — Practice Questions

**Q1.** A data science team needs to build a GenAI application that extracts key entities from customer reviews, summarizes the sentiment, and then classifies the urgency level. They want to track the entire workflow as a single unit in MLflow. Which LangChain component BEST meets these requirements?

A) Use a SequentialChain that chains three LLMChain steps (extract, summarize, classify) and log the SequentialChain with mlflow.langchain.log_model
B) Use three separate LLMChain objects and log each one individually with mlflow.langchain.log_model
C) Use a RetrievalQA chain with a custom retriever that handles extraction, summarization, and classification
D) Use a single LLMChain with a very long prompt that asks the model to do all three tasks in one output

<details>
<summary>Answer</summary>

**Correct: A**

SequentialChain is designed for multi-step workflows where the output of one chain becomes the input to the next. Logging the entire SequentialChain with mlflow.langchain.log_model captures the complete workflow as a single versioned artifact, enabling reproducibility and tracking.

**Why the others are wrong:**
- Logging three separate chains loses the workflow structure and makes it harder to version and deploy the complete application as a unit
- RetrievalQA is specifically for retrieval-augmented generation patterns (question answering with document retrieval), not for sequential processing tasks
- A single chain with one complex prompt is less maintainable, harder to debug individual steps, and doesn't leverage the modularity benefits of LangChain's chain composition

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An engineer is building a LangChain application on Databricks and needs to use a Foundation Model API endpoint (llama-2-70b-chat) as the LLM. Which code snippet correctly initializes the LLM for use in LangChain chains?

A) llm = Databricks(endpoint_name='databricks-llama-2-70b-chat', temperature=0.1)
B) llm = ChatDatabricks(endpoint='databricks-llama-2-70b-chat', max_tokens=512)
C) llm = OpenAI(model='llama-2-70b-chat', databricks_endpoint=True)
D) llm = LangChainLLM(foundation_model='llama-2-70b-chat', provider='databricks')

<details>
<summary>Answer</summary>

**Correct: B**

ChatDatabricks is the LangChain wrapper for Databricks Foundation Model API endpoints. The endpoint parameter specifies the model serving endpoint name, and parameters like max_tokens control generation.

**Why the others are wrong:**
- Databricks class exists but the parameter is endpoint (not endpoint_name), and this is less commonly used than ChatDatabricks for chat models
- OpenAI is for OpenAI's API, not Databricks Foundation Model APIs. There is no databricks_endpoint parameter
- LangChainLLM is not a valid LangChain class. The correct classes are ChatDatabricks or Databricks

</details>

---

**Q3.** A company is building a customer support bot that needs to answer questions using internal documentation stored in Databricks Vector Search. The bot should retrieve relevant documents and generate answers based on them. Which LangChain chain type is MOST appropriate?

A) SequentialChain with two steps: first retrieve documents, then generate answer
B) LLMChain with a custom prompt that includes retrieved documents manually
C) TransformChain that converts the question into a vector search query
D) RetrievalQA chain with Databricks Vector Search as the retriever

<details>
<summary>Answer</summary>

**Correct: D**

RetrievalQA is specifically designed for retrieval-augmented generation (RAG) patterns. It integrates a retriever (Databricks Vector Search) with an LLM to automatically retrieve relevant documents and generate answers based on them. This is the standard pattern for question answering over custom data.

**Why the others are wrong:**
- SequentialChain requires manual orchestration of retrieval and generation steps. RetrievalQA already handles this pattern, making SequentialChain unnecessary overhead
- LLMChain with manual document retrieval bypasses LangChain's retrieval abstractions and makes the code harder to maintain and version
- TransformChain is for data transformations, not for the complete retrieve-and-answer pattern. It doesn't handle LLM generation

</details>

---

**Q4.** An engineer wants to improve a classification model's accuracy by providing examples in the prompt. The task is to classify customer feedback as 'bug', 'feature_request', or 'question'. Which prompt engineering technique should they use, and what is the recommended number of examples for the exam scenario?

A) Chain-of-thought prompting with step-by-step reasoning for each classification
B) Few-shot prompting with 2-3 example input-output pairs showing the classification
C) Zero-shot prompting with a detailed description of each category
D) Few-shot prompting with 10-15 examples to cover all edge cases

<details>
<summary>Answer</summary>

**Correct: B**

Few-shot prompting (providing 2-3 example input-output pairs) is the standard technique for classification tasks. It helps the model understand the task format and desired output structure. The exam typically references 2-3 examples as the sweet spot for few-shot learning.

**Why the others are wrong:**
- Chain-of-thought is for tasks requiring multi-step reasoning (e.g., math problems), not simple classification. It would add unnecessary complexity here
- Zero-shot can work but is less reliable for classification than few-shot. The question specifically asks which technique to use when providing examples
- 10-15 examples may improve accuracy but increases prompt cost and latency. The exam focuses on 2-3 examples as the efficient approach for few-shot prompting

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A team needs to log a LangChain SequentialChain that uses Databricks Foundation Model APIs for versioning in MLflow. The chain includes prompt templates and multiple LLM calls. Which code correctly logs the chain?

A) mlflow.log_model(chain, 'langchain_model', registered_model_name='my-chain')
B) mlflow.langchain.log_model(chain, 'model', input_example={'text': 'sample'})
C) mlflow.pyfunc.log_model('model', python_model=chain)
D) mlflow.langchain.save_model(chain, path='model_dir')

<details>
<summary>Answer</summary>

**Correct: B**

mlflow.langchain.log_model is the correct method for logging LangChain chains (including SequentialChain) in MLflow. The artifact_path parameter ('model') specifies where to store it, and input_example helps with model serving validation.

**Why the others are wrong:**
- mlflow.log_model is a generic method but doesn't exist as a top-level function. The correct namespace is mlflow.langchain.log_model
- mlflow.pyfunc.log_model requires wrapping the chain in a PyFunc model class. mlflow.langchain.log_model handles this automatically and is the recommended approach
- mlflow.langchain.save_model saves to disk but doesn't log to the MLflow tracking server. log_model both saves and tracks the model with run metadata

</details>

---

**Q6.** A GenAI application needs to solve multi-step math word problems. The model sometimes gives wrong answers when asked directly. Which prompt engineering technique would MOST improve accuracy?

A) Few-shot prompting with 2-3 example solved problems showing only the final answers
B) Zero-shot prompting with a detailed explanation of mathematical concepts
C) Chain-of-thought prompting instructing the model to show each reasoning step before the final answer
D) RetrievalQA to retrieve similar solved problems from a vector database

<details>
<summary>Answer</summary>

**Correct: C**

Chain-of-thought prompting instructs the model to show intermediate reasoning steps (e.g., 'Let's solve this step by step'). This significantly improves accuracy on multi-step reasoning tasks like math problems by forcing the model to decompose the problem.

**Why the others are wrong:**
- Few-shot with only final answers doesn't show the reasoning process. The model needs to see the step-by-step work to learn the reasoning pattern
- Zero-shot with explanations might help but chain-of-thought explicitly prompts for step-by-step reasoning, which is more effective for complex reasoning tasks
- RetrievalQA is for answering questions using external documents, not for improving reasoning on new problems. It doesn't teach the model how to reason

</details>

---

**Q7.** An engineer is building a LangChain application that needs to first extract product names from a review, then look up each product in Databricks Vector Search, then generate a summary comparing the products. The application must be versioned in MLflow. Which combination of LangChain components BEST meets these requirements?

A) Single RetrievalQA chain with a custom prompt that handles extraction, lookup, and summarization
B) Three separate chains logged individually in MLflow, orchestrated with Python code
C) Single LLMChain with few-shot examples showing the complete extraction-lookup-summary process
D) SequentialChain with three steps: LLMChain for extraction, RetrievalQA for lookup, LLMChain for summary, logged with mlflow.langchain.log_model

<details>
<summary>Answer</summary>

**Correct: D**

SequentialChain allows chaining different chain types (LLMChain for extraction, RetrievalQA for retrieval, LLMChain for summary). This maintains modularity while creating a complete workflow. Logging the entire SequentialChain with mlflow.langchain.log_model versions the complete application as a single artifact.

**Why the others are wrong:**
- RetrievalQA is designed for question answering, not for multi-step processing that includes extraction and comparison. It doesn't naturally support chaining different operations
- Logging three separate chains loses the workflow structure and requires custom orchestration code that isn't versioned with the chains
- A single LLMChain can't perform vector search retrieval. It would require manual retrieval code outside the chain, breaking the LangChain abstraction

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A prompt engineer wants to combine few-shot and chain-of-thought techniques for a complex classification task that requires reasoning. How should they structure the prompt?

A) Provide 2-3 examples that show only the input and final classification label
B) Use chain-of-thought for the system prompt and few-shot for the user prompt
C) Provide 2-3 examples where each shows the input, the step-by-step reasoning process, and then the final classification
D) Provide 10+ examples without reasoning steps, then add an instruction to think step-by-step

<details>
<summary>Answer</summary>

**Correct: C**

Combining few-shot and chain-of-thought means the examples themselves demonstrate the reasoning process. Each example shows input → reasoning steps → final answer. This teaches the model both the task and the reasoning pattern to use.

**Why the others are wrong:**
- Examples without reasoning steps are just few-shot, not few-shot + chain-of-thought. The model won't learn to show its reasoning
- Chain-of-thought and few-shot aren't split between system and user prompts; they're integrated in the examples within the prompt
- More examples without reasoning don't demonstrate chain-of-thought. The key is showing the reasoning process in each example, not the number of examples

</details>

---

