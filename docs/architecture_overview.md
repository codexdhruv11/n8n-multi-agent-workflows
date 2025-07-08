# 🧠 n8n Multi-Agent Architecture Overview

## 🌐 System Summary

This architecture implements a **modular LangChain-based agent system on n8n**, where a **Supervisor Agent** intelligently routes user queries (from Telegram) to specialized sub-agents: **Email Agent**, **Calendar Agent**, or **Research Agent**. Each sub-agent handles its domain tasks using AI reasoning (OpenAI) and external APIs (Gmail, Google Calendar, SerpAPI, Wikipedia).

---

## 🧩 Agent Components & Responsibilities

| Agent                | Role & Responsibility                                                                   |
| -------------------- | --------------------------------------------------------------------------------------- |
| **Supervisor Agent** | Receives Telegram input, processes the intent, routes tasks to appropriate agents.      |
| **Email Agent**      | Sends or retrieves Gmail messages based on natural language instructions.               |
| **Calendar Agent**   | Creates events, schedules meetings, or retrieves events from Google Calendar.           |
| **Research Agent**   | Fetches factual information using Wikipedia or SerpAPI and returns a summarized answer. |

---

## 🗺️ Architecture Diagram (Text Version)

```
┌─────────────────────────────┐
│       Telegram Bot          │
│  (Receives user messages)   │
└────────────┬────────────────┘
             │
             ▼
   ┌──────────────────────────┐
   │    Supervisor Agent      │
   │  (LangChain + Memory)    │
   └────────┬────┬────┬───────┘
            │    │    │
      ┌─────▼─┐  │ ┌──▼────────┐
      │ Email │  │ │ Calendar  │
      │ Agent │  │ │  Agent    │
      └────▲──┘  │ └────┬──────┘
           │     │      │
           ▼     ▼      ▼
      Gmail API  Google Calendar
                   API
           
              ┌─────────────┐
              │ Research    │
              │  Agent      │
              └─────┬───────┘
                    ▼
          Wikipedia & SerpAPI
```

---

## 🔄 Data Flow

### 1. **User Input**

* Received via **Telegram** (`Telegram Trigger`)
* May be text or voice (voice is transcribed)

### 2. **Supervisor Agent**

* Uses `LangChain Agent`, `Groq/OpenAI`, and `Window Buffer Memory`
* Based on user intent, it routes to:

  * `Email Agent` → For "send email to..." or "get last 5 emails"
  * `Calendar Agent` → For "create a meeting on Friday"
  * `Research Agent` → For "search 1971 India-Pak war"

### 3. **Sub-Agent Execution**

* Each sub-agent is a separate workflow triggered via `toolWorkflow`
* Each uses its own set of external tools/APIs:

  * OpenAI for reasoning
  * Gmail API (Email)
  * Google Calendar API (Calendar)
  * Wikipedia/SerpAPI (Research)

### 4. **Supervisor Summarizes**

* Uses OpenAI again to create clean, plain-text responses
* Optionally converts text-to-speech (via ElevenLabs or OpenAI audio) before sending response

### 5. **Output**

* Delivered back to the user via **Telegram**

  * As text
  * Or as audio (optional)

---

## 🛠️ Technologies Used

| Component        | Technology/Service                  |
| ---------------- | ----------------------------------- |
| Workflow Engine  | [n8n](https://n8n.io/)              |
| Language Model   | OpenAI (GPT-4o), Groq (LLaMA-70B)   |
| Agents Framework | LangChain (via n8n LangChain nodes) |
| Calendar API     | Google Calendar                     |
| Email API        | Gmail (OAuth2)                      |
| Search Tools     | Wikipedia, SerpAPI                  |
| Messaging Input  | Telegram Bot API                    |
| Audio (optional) | ElevenLabs, OpenAI Audio API        |

---

## 🧩 Modular Agent List

| Agent          | LangChain Tool Name | Workflow ID (example) |
| -------------- | ------------------- | --------------------- |
| Email Agent    | `EMAIL_AGENT`       | `C688FekAFtpIacuH`    |
| Calendar Agent | `Calendar_Agent`    | `fR2xXIy5NoPfa6Vb`    |
| Research Agent | `Research_Agent`    | `Gk2g6uuC48i3Gz8k`    |

Each agent can be extended independently and reused across projects.

---

## 🧱 Design Philosophy

* 🧠 **AI-First Reasoning**: Decisions are delegated to the LLM for parsing natural language into structured actions.
* 🧩 **Composable Workflows**: Each agent is modular and self-contained.
* 🔐 **Credential Separation**: Credentials are injected per-agent and can be revoked independently.
* 💬 **Multi-Modal Input**: Accepts both text and voice via Telegram.
* 🔄 **Memory-Aware Conversations**: Tracks context using `Window Buffer Memory`.


