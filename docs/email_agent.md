# 📧 Email Agent – Workflow Documentation

## 1. **Workflow Title**

**Email Agent**
*(from JSON: `"email_Agent"`)*

---

## 2. **Workflow Purpose**

The **Email Agent** is a sub-agent designed to process natural language instructions related to email tasks. It can generate and send emails or fetch emails from a Gmail inbox based on user queries. This workflow integrates with LangChain agents and is typically triggered by a parent orchestrator like the **Supervisor Agent**.

---

## 3. **Trigger Type**

* **Type:** `Execute Workflow Trigger`
* **Description:** Activated when another workflow (e.g., Supervisor Agent) calls it using an Execute Workflow node.
* **Expected Input:**

```json
{
  "query": "can you send an email to dhruvkaushik9457@gmail.com , say hi to him"
}
```

---

## 4. **Node Breakdown**

### 🔹 Execute Workflow Trigger

* **Type:** `n8n-nodes-base.executeWorkflowTrigger`
* **Purpose:** Starts the workflow when triggered by a parent workflow.
* **Inputs:** JSON with a `query` key (natural language instruction).
* **Outputs:** Forwards the query to the Email Agent node.
* **Credentials:** None

---

### 🔹 Email Agent

* **Type:** `@n8n/n8n-nodes-langchain.agent`
* **Purpose:** LangChain agent that interprets the user’s intent from the query and decides which tool (send or fetch email) to use.
* **Inputs:**

  * `query`: The user's instruction in plain English
  * `ai_languageModel`: Linked to OpenAI Chat Model
  * `ai_tool`: Includes tools for sending and retrieving Gmail messages
* **Outputs:** JSON containing structured email-related fields like subject, body, recipient, or retrieved message summaries.
* **System Role (implicit):** Understand and act on email-based tasks.
* **Dependencies:** Gmail tools, OpenAI

---

### 🔹 OpenAI Chat Model

* **Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
* **Purpose:** Provides LLM-powered reasoning to interpret queries and extract structured commands.
* **Model Used:** `gpt-4o`
* **Inputs:** Agent prompt from `Email Agent`
* **Outputs:** AI-generated structured data (e.g., recipient email, subject)
* **Credentials:** `OpenAi account`

---

### 🔹 Send a message in Gmail

* **Type:** `n8n-nodes-base.gmailTool`
* **Purpose:** Sends a plain text email via Gmail.
* **Inputs:**

  * `sendTo`: From `$fromAI('emailAddress')`
  * `subject`: From `$fromAI('emailSubject')`
  * `message`: From `$fromAI('emailBody')`
* **Outputs:** Confirmation of sent message
* **Credentials:** `Gmail account`

---

### 🔹 Get many messages in Gmail

* **Type:** `n8n-nodes-base.gmailTool`
* **Purpose:** Retrieves emails from Gmail based on AI-defined filters (e.g., sender or limit).
* **Inputs:**

  * `limit`: From `$fromAI('limit')`
  * `filters.sender`: From `$fromAI('senderEmail')`
* **Outputs:** Array of email metadata and content
* **Credentials:** `Gmail account`

---

### 🔹 Edit Fields

* **Type:** `n8n-nodes-base.set`
* **Purpose:** Stores the `output` from the agent as a field called `response`.
* **Inputs:** `output` field from Email Agent node
* **Outputs:**

```json
{
  "response": "Email sent successfully!" // or fetched emails summary
}
```

* **Credentials:** None

---

## 5. **Credential Requirements**

| Node                       | Credential       |
| -------------------------- | ---------------- |
| OpenAI Chat Model          | `OpenAi account` |
| Send a message in Gmail    | `Gmail account`  |
| Get many messages in Gmail | `Gmail account`  |

---

## 6. **Inputs/Outputs**

### ✅ Input:

```json
{
  "query": "can you send an email to dhruvkaushik9457@gmail.com , say hi to him"
}
```

### ✅ Output:

```json
{
  "response": "Email sent to dhruvkaushik9457@gmail.com with subject 'Hi'..."
}
```

*Or if fetching emails:*

```json
{
  "response": "Here are 5 emails from example@example.com..."
}
```

---

## 7. **How This Workflow is Called**

* **Caller Workflow:** `Supervisor Agent`
* **Call Method:** LangChain Tool (`@n8n/n8n-nodes-langchain.toolWorkflow`)
* **Tool Name in Parent:** `"EMAIL_AGENT"`
* **Sample Declaration:**

```json
{
  "name": "EMAIL_AGENT",
  "workflowId": "C688FekAFtpIacuH",
  "description": "Call this tool to take action in email"
}
```

---

## 8. **Notes for Extension**

### 💡 Extension Suggestions:

* **Add Rich Text Support:** Allow HTML body formatting via a switch or AI inference.
* **Add Email Attachments:** Extend the Gmail tool to handle file uploads.
* **Add Error Handling:** Use IF/TRY nodes for Gmail API errors and rate limits.
* **Add Logging:** Append logs to a Google Sheet or database for sent/fetched emails.

### ⚠️ Assumptions:

* The Gmail account is fully authorized with necessary scopes.
* The user’s natural language input clearly indicates the intent (e.g., "send an email", "get last 3 emails").
* The workflow runs as a sub-agent and is not user-facing directly.


