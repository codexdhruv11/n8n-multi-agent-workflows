# 📆 Calendar Agent – Workflow Documentation

## 1. **Workflow Title**

**Calendar Agent**
*(from JSON: `"Calender"` – assumed corrected spelling to “Calendar”)*

---

## 2. **Workflow Purpose**

The **Calendar Agent** acts as an AI-powered assistant responsible for managing calendar-related tasks using Google Calendar. It can create events, create meetings with attendees, or fetch existing events based on natural language instructions. This workflow integrates into a multi-agent system (e.g., Supervisor Agent) where it handles calendar-specific responsibilities.

---

## 3. **Trigger Type**

* **Type:** `Execute Workflow Trigger`
* **Description:** This workflow is invoked by another workflow, such as the Supervisor Agent, through an Execute Workflow call. It expects an input query that describes a calendar-related action in natural language.
* **Expected Input Example:**

```json
{
  "query": "setup an event at 3 pm for meeting, event name n8n"
}
```

---

## 4. **Node Breakdown**

### 🔹 Execute Workflow Trigger

* **Type:** `n8n-nodes-base.executeWorkflowTrigger`
* **Purpose:** Entry point that accepts external calls and forwards query input to the LangChain agent.
* **Inputs:** JSON object containing the `query` key
* **Outputs:** Forwards to `Calendar Agent`
* **Credentials:** None

---

### 🔹 Calendar Agent

* **Type:** `@n8n/n8n-nodes-langchain.agent`
* **Purpose:** LangChain agent that interprets the user's query and uses the proper calendar tool node (create, get, or schedule event).
* **Inputs:**

  * `query`: Natural language text
  * `ai_languageModel`: OpenAI Chat Model
  * `ai_tool`: any of the 3 calendar tools (Create Event, Get Events, Meeting with Attendee)
* **System Prompt Highlights:**

  ```
  - If no end time is given, default duration is 60 minutes.
  - Use the attendee version only when a participant is mentioned.
  - Exclude canceled events from results.
  ```
* **Outputs:** Structured JSON including `output` containing calendar event details or summaries
* **Dependencies:** Google Calendar API, OpenAI

---

### 🔹 OpenAI Chat Model

* **Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`
* **Purpose:** Processes natural language and supports reasoning for event extraction.
* **Inputs:** Internal to LangChain agent
* **Outputs:** Structured values (e.g., `starttime`, `endtime`, `attendeeEmail`)
* **Model:** Likely `gpt-4o`
* **Credentials:** `OpenAi account`

---

### 🔹 Create Event

* **Type:** `n8n-nodes-base.googleCalendarTool`
* **Purpose:** Creates a basic calendar event (no attendees).
* **Inputs:**

  * `calendar`: Calendar ID (preset as `dhruvkaushikjee9457@gmail.com`)
  * `start`, `end`: From `$fromAI('starttime')`, `$fromAI('endtime')`
* **Outputs:** Confirmation of created event
* **Credentials:** `Google Calendar account`

---

### 🔹 Meeting with Attendee

* **Type:** `n8n-nodes-base.googleCalendarTool`
* **Purpose:** Schedules a meeting with at least one specified attendee.
* **Inputs:**

  * Same as Create Event, plus:
  * `attendees`: Array of emails from `$fromAI('attendeeEmail')`
* **Outputs:** Meeting details and confirmation
* **Credentials:** `Google Calendar account`

---

### 🔹 Get Events

* **Type:** `n8n-nodes-base.googleCalendarTool`
* **Purpose:** Retrieves events within a given date range.
* **Inputs:**

  * `timeMin`, `timeMax`: From `$fromAI('startDate')`, `$fromAI('endDate')`
* **Outputs:** Array of event objects
* **Credentials:** `Google Calendar account`

---

### 🔹 Edit Fields

* **Type:** `n8n-nodes-base.set`
* **Purpose:** Stores the `output` from the agent into a named field (`response`) for easy return.
* **Inputs:** `output` from `Calendar Agent`
* **Outputs:**

```json
{
  "response": "Event created successfully for n8n at 3 PM"
}
```

* **Credentials:** None

---

## 5. **Credential Requirements**

| Node                  | Credential                |
| --------------------- | ------------------------- |
| OpenAI Chat Model     | `OpenAi account`          |
| Create Event          | `Google Calendar account` |
| Get Events            | `Google Calendar account` |
| Meeting with Attendee | `Google Calendar account` |

---

## 6. **Inputs/Outputs**

### ✅ Input:

```json
{
  "query": "Set up a 3 PM meeting on Friday with John"
}
```

### ✅ Output:

```json
{
  "response": "Meeting with John scheduled for Friday at 3 PM"
}
```

*Or for fetching events:*

```json
{
  "response": "You have 2 events scheduled for July 10th: Team Sync at 10 AM, Client Call at 2 PM"
}
```

---

## 7. **How This Workflow is Called**

* **Caller Workflow:** `Supervisor Agent`
* **Call Method:** LangChain Tool (`@n8n/n8n-nodes-langchain.toolWorkflow`)
* **Tool Name in Parent:** `"Calendar_Agent"`
* **Sample Declaration:**

```json
{
  "name": "Calendar_Agent",
  "workflowId": "fR2xXIy5NoPfa6Vb",
  "description": "Call this tool to manage the calendar."
}
```

---

## 8. **Notes for Extension**

### 💡 Extension Suggestions:

* **Add Timezone Handling:** Ensure start/end times respect user’s local timezone.
* **Event Reminders:** Add optional reminder notifications via email or Telegram.
* **Recurring Events:** Extend to support repeat rules (e.g., weekly meetings).
* **Conflict Detection:** Check for overlapping events before scheduling.

### ⚠️ Assumptions:

* Google Calendar account `dhruvkaushikjee9457@gmail.com` is properly authorized.
* User queries are clear enough for the AI to infer date/time/attendee correctly.
* This agent only handles **single-attendee** meetings; multi-participant logic must be added if needed.

