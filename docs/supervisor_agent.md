# 🧑‍💼 Supervisor Agent – Workflow Documentation

## 1. **Workflow Title**

**Supervisor Agent**
*(JSON title: `My workflow`)*

## 2. **Workflow Purpose**

The **Supervisor Agent** serves as the central orchestrator in a multi-agent LangChain-based architecture. It receives user input (text or voice) via Telegram, processes it using AI models, and intelligently delegates tasks to sub-agents such as Email, Calendar, Research, and Contacts. It can also transcribe audio, summarize responses, and return answers via text or audio through Telegram.

---

## 3. **Trigger Type**

* **Type:** `Telegram Trigger`
* **Description:** Activated when the bot receives a message on Telegram.
* **Expected Inputs:** Telegram message object, which could include:

  * Text messages
  * Voice messages (audio file)

---

## 4. **Node Breakdown**

Below is a breakdown of each node in the Supervisor Agent workflow.

---

### 🔹 Telegram Trigger

* **Type:** `n8n-nodes-base.telegramTrigger`
* **Purpose:** Listens for incoming Telegram messages.
* **Inputs:** Incoming message (text or voice)
* **Outputs:** Raw Telegram message object
* **Credentials:** `Telegram account`

---

### 🔹 Switch

* **Type:** `n8n-nodes-base.switch`
* **Purpose:** Routes messages based on whether they contain text or voice.
* **Inputs:** Telegram message object
* **Outputs:** Branches labeled `Audio` or `text`

---

### 🔹 download a file

* **Type:** `n8n-nodes-base.telegram`
* **Purpose:** Downloads voice/audio messages from Telegram.
* **Inputs:** `voice.file_id`
* **Outputs:** Binary audio data
* **Credentials:** `Telegram account`

---

### 🔹 OpenAI (Transcription)

* **Type:** `@n8n/n8n-nodes-langchain.openAi`
* **Purpose:** Transcribes audio into text.
* **Inputs:** Binary audio data
* **Outputs:** Transcribed text
* **Credentials:** `OpenAi account`

---

### 🔹 Edit Fields (Text Path)

* **Type:** `n8n-nodes-base.set`
* **Purpose:** Extracts plain text from the Telegram message.
* **Inputs:** `message.text`
* **Outputs:** `text`
* **Credentials:** None

---

### 🔹 Window Buffer Memory

* **Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow`
* **Purpose:** Maintains memory across conversation context using chat ID as session key.
* **Inputs:** Chat ID from Telegram message
* **Outputs:** Contextual memory
* **Credentials:** None

---

### 🔹 Supervisor AI Agent

* **Type:** `@n8n/n8n-nodes-langchain.agent`

* **Purpose:** Main LangChain agent that processes incoming text and chooses sub-agents/tools to call.

* **Inputs:** Processed user input (`text`)

* **Outputs:** Final output response (`output`)

* **System Prompt:**

  ```
  Summarize the following incoming data: {{ $json.output }}.
  For calendar events, include only the event name and time.
  For emails, provide only the email title and a brief summary of the body, excluding any timestamps.
  Ensure the output is in plain text format, without any symbols or markdown formatting.
  ```

* **Dependencies:**

  * Memory: `Window Buffer Memory`
  * Tools:

    * `Contacts Database` (Google Sheets)
    * `Email Agent`
    * `Calendar Agent`
    * `Research Agent`
  * Language Models:

    * `Groq Chat Model`

---

### 🔹 Summarise

* **Type:** `@n8n/n8n-nodes-langchain.openAi`
* **Purpose:** Final summarization step before returning to user.
* **Inputs:** `output` from Supervisor Agent
* **Outputs:** Plain-text summary
* **Credentials:** `OpenAi account`

---

### 🔹 HTTP Request (ElevenLabs)

* **Type:** `n8n-nodes-base.httpRequest`
* **Purpose:** Converts final response text into speech via ElevenLabs API.
* **Inputs:** Text to be converted
* **Outputs:** Binary audio
* **Dependencies:** ElevenLabs API (manual key inputted)

---

### 🔹 Telegram1 (Send Audio)

* **Type:** `n8n-nodes-base.telegram`
* **Purpose:** Sends the generated voice message to the user.
* **Inputs:** Binary audio data
* **Outputs:** Audio message sent to Telegram
* **Credentials:** `Telegram account`

---

### 🔹 Telegram (Send Text)

* **Type:** `n8n-nodes-base.telegram`
* **Purpose:** Sends text response to user via Telegram.
* **Inputs:** `output` (summarized response)
* **Outputs:** Text message
* **Credentials:** `Telegram account`

---

### 🔹 Contacts Database

* **Type:** `n8n-nodes-base.googleSheetsTool`
* **Purpose:** Accesses and manipulates Google Sheets for contact data.
* **Inputs:** Query from Supervisor Agent
* **Outputs:** Fetched contact info
* **Credentials:** `Google Sheets account`

---

### 🔹 Email Agent

* **Type:** `@n8n/n8n-nodes-langchain.toolWorkflow`
* **Purpose:** Calls `email_Agent` workflow to send or fetch emails.
* **Inputs:** Natural language query
* **Outputs:** Response from email action
* **Dependencies:** Gmail API, OpenAI

---

### 🔹 Calendar Agent

* **Type:** `@n8n/n8n-nodes-langchain.toolWorkflow`
* **Purpose:** Calls `Calender` workflow to manage Google Calendar events.
* **Inputs:** User queries related to events
* **Outputs:** Event details or confirmation
* **Dependencies:** Google Calendar API, OpenAI

---

### 🔹 Research Agent

* **Type:** `@n8n/n8n-nodes-langchain.toolWorkflow`
* **Purpose:** Calls `Research_agent` workflow to search the internet for answers.
* **Inputs:** `query`
* **Outputs:** Response from SerpAPI/Wikipedia via LLM
* **Dependencies:** SerpAPI, OpenAI

---

### 🔹 Groq Chat Model

* **Type:** `@n8n/n8n-nodes-langchain.lmChatGroq`
* **Purpose:** Provides low-latency inference using the Groq-hosted LLaMA-70B model.
* **Inputs:** Prompt from Supervisor Agent
* **Outputs:** Text response
* **Credentials:** `Groq account`

---

## 5. **Credential Requirements**

| Credential           | Used In Node(s)                                        |
| -------------------- | ------------------------------------------------------ |
| **Telegram account** | Telegram Trigger, download a file, Telegram, Telegram1 |
| **OpenAi account**   | OpenAI, Supervisor AI Agent, Summarise                 |
| **Google Sheets**    | Contacts Database                                      |
| **Gmail OAuth2**     | Email Agent sub-workflow                               |
| **Google Calendar**  | Calendar Agent sub-workflow                            |
| **SerpAPI account**  | Research Agent sub-workflow                            |
| **Groq account**     | Groq Chat Model                                        |
| **ElevenLabs API**   | HTTP Request (manual API key header)                   |

---

## 6. **Inputs/Outputs**

### Inputs

* **Via Telegram:**

  * Text messages
  * Voice messages (converted to text)

### Outputs

* **To Telegram:**

  * Text response (summarized or full output)
  * Optional audio response (voice-based via ElevenLabs)

---

## 7. **How This Workflow is Called**

* **Not called by other workflows.**
* This workflow is the **entry point** for the system and handles external user interaction via Telegram.

---

## 8. **Notes for Extension**

### 💡 Extension Ideas

* **Multilingual Support:** Add translation layer using DeepL or Google Translate API.
* **Rate Limiting / Abuse Control:** Integrate logic to prevent spam or overuse.
* **Response Logging:** Log queries and responses to a database or Google Sheet.
* **Tool Priority Switching:** Let user choose if they prefer voice/text/audio output.

### ⚠️ Assumptions

* Telegram bot is already configured and added to the target group.
* All sub-agents (`email_Agent`, `Calender`, `Research_agent`) are correctly set up and deployed.
* Audio transcription and generation endpoints are reliable (OpenAI/ElevenLabs).

