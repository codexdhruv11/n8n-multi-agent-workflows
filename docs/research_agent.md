# 🧠 Research Agent – Workflow Documentation

## 1. **Workflow Title**

**Research Agent**

## 2. **Workflow Purpose**

The **Research Agent** is designed to act as an AI-powered assistant that can answer user queries using a combination of **Wikipedia** and **SerpAPI** search results. It's part of a broader multi-agent architecture, likely triggered by a Supervisor Agent, to assist in tasks involving internet-based fact retrieval or research summaries.

## 3. **Trigger Type**

* **Type:** `Execute Workflow Trigger`
* **Description:** This workflow is initiated via the `Execute Workflow` node from another parent workflow (such as the Supervisor Agent). It expects an input JSON with a `query` key.
* **Example Input:**

```json
{
  "query": "search for 1971 india-pak war"
}
```

---

## 4. **Node Breakdown**

### 🔹 Execute Workflow Trigger

* **Type:** `n8n-nodes-base.executeWorkflowTrigger`
* **Purpose:** Acts as the entry point for the workflow, accepting input from parent workflows.
* **Inputs:** JSON containing the key `query`
* **Outputs:** Passes query to the `Research Agent` node
* **Dependencies:** None

---

### 🔹 Research Agent

* **Type:** `@n8n/n8n-nodes-langchain.agent`
* **Purpose:** A LangChain Agent that routes the query using tools (Wikipedia, SerpAPI) and an LLM model to construct a final answer.
* **Inputs:**

  * `query`: A natural language question
* **Outputs:**

  * `output`: Generated answer text
* **System Message Logic:**

  ```
  You are a research assistant with access to Wikipedia and SerpAPI.
  First, search Wikipedia. If you don’t find the answer, use SerpAPI.
  ```
* **Dependencies:** Relies on all three supporting nodes:

  * `Wikipedia` Tool
  * `SerpAPI`
  * `OpenAI Chat Model`

---

### 🔹 OpenAI Chat Model

* **Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
* **Purpose:** Processes the query and integrates the outputs from tools to generate human-like responses.
* **Inputs:** Internal processing through LangChain agent
* **Outputs:** Used internally by agent to formulate final response
* **Credentials Required:** `OpenAi account`

---

### 🔹 Wikipedia

* **Type:** `@n8n/n8n-nodes-langchain.toolWikipedia`
* **Purpose:** Searches Wikipedia for the input query.
* **Inputs:** Derived from the `query` field via LangChain agent
* **Outputs:** Search results (text/data)
* **Dependencies:** None (no credentials needed)

---

### 🔹 SerpAPI

* **Type:** `@n8n/n8n-nodes-langchain.toolSerpApi`
* **Purpose:** Executes a Google-like web search using SerpAPI if Wikipedia doesn't yield sufficient results.
* **Inputs:** Derived from the `query` field via LangChain agent
* **Outputs:** JSON-formatted search results
* **Credentials Required:** `SerpAPI account`

---

## 5. **Credential Requirements**

| Node              | Credential Required |
| ----------------- | ------------------- |
| OpenAI Chat Model | `OpenAi account`    |
| SerpAPI           | `SerpAPI account`   |
| Wikipedia         | None                |

---

## 6. **Inputs/Outputs**

### **Input:**

* JSON with a key: `query`
* Example:

```json
{
  "query": "search for 1971 india-pak war"
}
```

### **Output:**

* JSON object with the `output` key containing the answer.
* Example:

```json
{
  "output": "The 1971 India-Pakistan war resulted in the creation of Bangladesh..."
}
```

---

## 7. **How This Workflow is Called**

* **Trigger Type:** `Execute Workflow`
* **Caller Workflow:** `Supervisor_agent` (via `Research Agent` node of type `@n8n/n8n-nodes-langchain.toolWorkflow`)
* **Execution Chain:**

  * The Supervisor Agent detects a query related to research.
  * It calls the `Research Agent` by executing this workflow and passing the query string.

---

## 8. **Notes for Extension**

### 🔧 Extension Suggestions:

* **Error Handling:** Add a `Try/Catch` or `IF` node to handle cases where Wikipedia and SerpAPI return no results.
* **Logging:** Add a `Function` node or `HTTP Request` to log queries and responses.
* **Memory Integration:** Use LangChain’s `memoryBufferWindow` to provide continuity in multi-turn conversations.
* **Response Type Handling:** Support different output types (e.g., short summary, detailed explanation, source references).

### ⚠️ Assumptions:

* The SerpAPI and OpenAI credentials are correctly set.
* The `Supervisor Agent` provides well-formatted JSON with the `query` key.
* This workflow only handles search/research tasks, not actions or system commands.

