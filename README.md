
# 🤖 BrainMate AI -  AI Mental Health Therapist
BrainMate AI is a tool-augmented, LLM-driven mental health support system designed to demonstrate how modern AI agents can integrate real-world services in a safe and structured manner. The project implements a chat-first architecture that blends conversational intelligence with practical utilities for user safety and resource discovery.

BrainMate AI combines:

- A therapeutic LLM backend (MedGemma via Groq) for context-aware, empathetic dialogue

- Automated crisis-scenario handling, powered by a Twilio Voice tool for initiating emergency calls

- A location-aware therapist lookup tool, implemented using the Google Maps API

- Multi-channel interaction, including a Streamlit-based web chat UI and a Twilio WhatsApp webhook

This repository provides a compact, opinionated reference for building a production-style AI agent that can reason, call external tools, and provide mental-health–oriented support through both web and messaging platforms. It is intended as a solid starting point for developers exploring safe LLM tooling, agent orchestration, and multi-channel deployment workflows.

#### ❗Important Note: This project is for educational/demo purposes only and is not a substitute for professional mental health care. Do not rely on it for emergency situations.

## 🖧 Technical Architecture
![Logo](https://github.com/user-attachments/assets/f248b3c6-6fc8-4852-87c9-e4a9b9c36cec)


## 📁 Project Structure

```text
BrainMate-AI/
├── backend/
│   ├── ai_agent.py         # LangGraph-based AI agent + tools (LLM, emergency call, therapist finder)
│   ├── config.py           # API keys and configuration (not shown here; you create it)
│   ├── custom_tools.py     # Low-level tool implementations (MedGemma, Twilio call, etc.)
│   ├── main.py             # FastAPI backend (JSON /ask + Twilio WhatsApp /whatsapp_ask)
│   └── test_location_tool.py      # Tests/examples for the location tool
├── images/
├── .gitignore
├── frontend.py              # Streamlit chat UI (web client) talking to FastAPI backend
├── pyproject.toml           # Project metadata and Python dependencies (managed with uv)
└── README.md                # Main project README
```


## 🧩 Key Components

1️⃣ **`frontend.py`** 
- A Streamlit-based UI that provides a simple chat workflow:
   - Renders the conversation using `st.chat_message` and accepts user input via `st.chat_input` .
  - Sends each user query to the backend at `POST/ask` at `http://localhost:8000/ask` .
  - Displays the agent’s final response and surfaces any tool invoked during the reasoning process.



2️⃣ **`backend/main.py`** 
- Defines the FastAPI server that powers BrainMate AI. It includes:
  - `POST/ask` — JSON API endpoint consumed by the Streamlit frontend.
  - `POST/whatsapp_ask` — Twilio WhatsApp webhook that parses form-encoded messages (Body field) and responds using TwiML.
  - Loads and executes the REAct graph defined in `ai_agent.graph` .
  - Uses `parse_response()` to extract the model’s final message and tool activity.
  - Implements `_twiml_message()` to return minimal TwiML XML payloads for WhatsApp.



3️⃣ **`backend/ai_agent.py`** 
- Configures the tool-using LLM agent and exposes all available capabilities using `@tool`:
  - `ask_mental_health_specialist(query: str)` — Calls `query_medgemma()` to generate therapeutic responses using the MedGemma model.
  - `emergency_call_tool()` — Invokes `call_emergency()` to trigger a safety helpline call through Twilio Voice.
  - `find_nearby_therapists_by_location(location: str)` — Uses the Google Maps API to geocode the provided location and return nearby therapist details (name, address, phone).

- LLM & Agent Setup:
  - Uses ChatGroq with model `openai/gpt-oss-120b`, keyed via `GROQ_API_KEY` from `config.py`
  - Creates a REAct-style LangGraph agent with the full toolset.
  - Defines a structured `SYSTEM_PROMPT` guiding when and how tools should be used.
  - Provides `parse_response(stream)` to read the streamed LangGraph output and determine final text + selected tool.



4️⃣ **`backend/config.py`** (user-generated)

- Holds required configuration values, for example:
  - `GROQ_API_KEY`
  - `GOOGLE_MAPS_API_KEY`
  - Twilio credentials (`TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, phone numbers)

This file is intentionally excluded from version control.





5️⃣ **`backend/custom_tools.py`** (user-generated)
- Contains the concrete integrations used by the agent:
  - `query_medgemma(query: str)` — calls the MedGemma-based therapeutic model.
  - `call_emergency()` — triggers the Twilio emergency call workflow.
  - Optional helpers for Google Maps lookups, Twilio operations, and other external services.



## 🛠️ Tech Stack

- **Language:** Python (>= 3.11)
- **Environment & packaging:** uv (for virtualenv + dependency management via pyproject.toml)
- **Backend:** FastAPI + Uvicorn
- **Frontend:** Streamlit
- **LLM / Agent:** `langchain`, `langgraph` and `langchain-groq` with ChatGroq

- **Integrations:** Twilio (WhatsApp + Voice), Google Maps API (Places + Geocoding) & Geopy / Requests (supporting utilities)

Dependencies (from `pyproject.toml`):
- `fastapi`
- `geopy`
- `googlemaps`
- `langchain`
- `langchain-groq`
- `langchain-openai`
- `langgraph`
- `ollama`
- `pydantic`
- `python-multipart` (needed for FastAPI form parsing, e.g. Twilio webhooks)
- `requests`
- `streamlit`
- `twilio`
- `uvicorn`



## 📦 Prerequisites

- Python 3.11+ installed on your system.
- `uv` installed (for virtual environment + dependency management).
- API keys / credentials for:
   - Groq (LLM): `GROQ_API_KEY`
   - Google Maps: `GOOGLE_MAPS_API_KEY`
   - Twilio:
     - `TWILIO_ACCOUNT_SID`
     - `TWILIO_AUTH_TOKEN`
     - Verified phone numbers / WhatsApp sandbox setup.




## ⚙️ Setup with `uv`

All commands below assume you are in the project root: `BrainMate-AI/`.

### 1. Clone and install

```bash
git clone https://github.com/MadtorXD/BrainMate-AI.git
cd BrainMate-AI

# Install dependencies and create .venv using uv
uv sync
```

### 2. Activate the virtual environment

```bash
# windowsOS / powerShell
.venv\Scripts\Activate.ps1
```

If you prefer not to activate the venv manually, you can also run commands through uv directly (see examples below).



### 3. Configure environment variables and `config.py`

Create a file `backend/config.py` with your keys. For example:

```python
# backend/config.py

GROQ_API_KEY = "your_groq_api_key_here"
GOOGLE_MAPS_API_KEY = "your_google_maps_api_key_here"

# Twilio (used by custom_tools.py / emergency_call_tool)
TWILIO_ACCOUNT_SID = "your_twilio_account_sid_here"
TWILIO_AUTH_TOKEN = "your_twilio_auth_token_here"
TWILIO_FROM_NUMBER = "+1234567890"  # your Twilio phone or WhatsApp-enabled number
TWILIO_EMERGENCY_TO_NUMBER = "+1987654321"  # safety helpline / emergency contact
```

> **Security note:** Don’t commit real keys; use `.env` or environment variables in production.

If you prefer environment variables, adapt `config.py` to read from `os.environ`.




## ⚡ Running the Backend (FastAPI + Twilio webhook)

### Option A: Using uv directly

From the project root:

```bash
uv run uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload
```

### Option B: Using the activated venv

```bash
cd backend
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

This exposes:

- `POST /ask` – JSON API for the Streamlit frontend.
- `POST /whatsapp_ask` – Twilio WhatsApp webhook endpoint.

### `POST /ask` – JSON endpoint

- **URL:** `http://localhost:8000/ask`
- **Method:** `POST`
- **Content-Type:** `application/json`
- **Body:**

```json
{
  "message": "I’ve been feeling really anxious lately."
}
```

- **Response (example):**

```json
{
  "response": "Empathetic therapeutic guidance here...",
  "tool_called": "ask_mental_health_specialist"
}
```

### `POST /whatsapp_ask` – Twilio WhatsApp webhook

- **Local URL:** `http://localhost:8000/whatsapp_ask`
- **Public URL (via ngrok or similar):** `https://<your-ngrok-domain>/whatsapp_ask`
- **Method:** `POST`
- **Expected content-type:** `application/x-www-form-urlencoded`
- **Key parameters (from Twilio):**
  - `Body`: the incoming text message
  - `From`: the sender’s WhatsApp number (may be used in tools)

Example local test with `curl`:

```bash
curl -X POST \
  http://localhost:8000/whatsapp_ask \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "Body=Hello, I’m feeling overwhelmed."
```

Response (simplified):

```xml
<Response>
  <Message>AI therapist response here...</Message>
</Response>
```

> If you see `422 Unprocessable Entity`, it usually means the `Body` form field is missing or the content-type is not `application/x-www-form-urlencoded`.




## 🖥️ Running the Frontend (Streamlit)

With the backend active on `http://localhost:8000`, you can launch the Streamlit interface from the project root:

```bash
uv run streamlit run frontend.py
```

This command will:
- Start the Streamlit application (typically available at `http://localhost:8501`).
- Render the chat UI titled “🧠 BrainMate AI – Mental Health Companion”.
- Accept user queries through `st.chat_input`.
- Forward each message to the backend via `POST` request to `http://localhost:8000/ask`.
- Display the assistant’s generated response along with any tool invoked during processing.




## 🔗 Twilio WhatsApp Integration

### 1. Expose the backend via a public URL

Twilio needs to reach your FastAPI server over the public internet. A common approach is to use [ngrok](https://ngrok.com/).

With the FastAPI server running on port 8000:

```bash
ngrok http 8000
```

Take note of the public HTTPS URL, e.g.: `https://abcd1234.ngrok.io`.

### 2. Configure Twilio Sandbox / WhatsApp number

In the Twilio Console:

- Go to your WhatsApp Sandbox or phone number configuration.
- Set the **Webhook URL** for incoming messages to:

   ```text
   https://abcd1234.ngrok.io/whatsapp_ask
   ```

- Ensure the method is `POST` and the body is `application/x-www-form-urlencoded` (the default for Twilio).

Now, messages sent to your Twilio WhatsApp number should be forwarded to `/whatsapp_ask`, which will:

- Extract `Body` via `Form` parsing.
- Run the LangGraph-based agent.
- Return a TwiML `<Message>` body as the reply.

### 3. Common pitfalls & troubleshooting

- **422 Unprocessable Entity**
  - Usually means FastAPI could not validate the request body.
  - Check that `Body` is present in the form data.
  - Ensure you are using `application/x-www-form-urlencoded`, not JSON.
  - Confirm the path in Twilio (`/whatsapp_ask`) matches the backend route exactly.

- **No response or Twilio error**
  - Make sure the FastAPI server is running and the ngrok tunnel is active.
  - Check logs in your terminal for Python exceptions (e.g., misconfigured `config.py`).

- **Twilio signature validation** (optional hardening)
  - For production, you should validate `X-Twilio-Signature` headers to ensure requests are genuinely from Twilio.




## 🗺️ Google Maps Therapist Location Finder Tool

The `find_nearby_therapists_by_location(location: str)` tool in `backend/ai_agent.py` enables the agent to look up mental-health professionals based on a user-specified location:

**How It Works?**

- This tool uses the `GOOGLE_MAPS_API_KEY` along with the official `googlemaps` Python client. The workflow is :
  - Geocodes the user-provided location string to latitude/longitude.
  - Calls `places_nearby` with `keyword="Psychotherapist"` and a 5km radius.
  - Retrieves up to 5 top results and fetches phone numbers via `gmaps.place`.
  - Returns a formatted string listing therapists near the location.

The React agent automatically decides to trigger this tool whenever the user asks for help finding a therapist — for example:
- “Find a therapist near Mumbai”
- “Is there any psychotherapist close to Gorakhpur?”
- “Show mental health specialists around Noida”




## 🚨 Emergency Call Tool

The `emergency_call_tool()` in `backend/ai_agent.py`:

- Calls `call_emergency()` from `backend/custom_tools.py`.
- Expected behavior:
  - Initiate a Twilio voice call to a predefined emergency / helpline number.
  - Provide a script or connect the user to human support.

> **Ethical note:** Use extreme caution if you adapt this to real-world scenarios. Always comply with local regulations and best practices for crisis support.




## 📡 API Reference

#### Below are the required API keys and tokens that are required for the proper functioning of the project:

| Parameter |  Description               |
| :-------- | :------------------------- |
| `TWILIO_ACCOUNT_SID` | **Required**. Your Twilio SID key |
| `TWILIO_AUTH_TOKEN` | **Required**. Your Twilio AUTH token number |
| `TWILIO_FROM_NUMBER` | **Required**. Your Twilio Phone number that you'll purchase |
| `EMERGENCY_CONTACT` | **Required**. Your local number or country's emergency number |
| `GROQ_API_KEY` | **Required**. Your GROQ API key |
| `GOOGLE_MAPS_API_KEY` | **Required**. Your Google Maps API key to be found in Google Cloud Console |


> **Note:** Don't forget to add the correct country code in the right format.




## 📝 Development Notes

### Code style & structure

- Separation of concerns:
  - `backend/main.py`: HTTP layer (FastAPI routes, Twilio TwiML responses).
  - `backend/ai_agent.py`: Agent orchestration and tool definitions.
  - `backend/custom_tools.py`: Concrete integrations (LLM, Twilio, etc.).
  - `frontend.py`: UI only – no business logic.

- Streaming agent:
  - The agent is executed via `graph.stream(inputs, stream_mode="updates")`.
  - `parse_response()` walks through the streaming updates to detect:
    - Which tool (if any) was called.
    - The final agent message to return.

### Testing

- There is a sample/test file `backend/test_location_tool.py` for validating the location-based therapist finder.
- You can run tests (if configured) with:

```bash
uv run pytest
```

(If no tests are defined yet, you can create them under a `tests/` folder or alongside backend modules.)




## 🧪 Suggested Next Steps

To further strengthen the reliability, safety, and production readiness of BrainMate AI, consider implementing the following enhancements:

- **Enable Twilio signature validation** for the `/whatsapp_ask` endpoint to ensure incoming webhook requests are authentic and tamper-proof.

- **Improve error handling** around all third-party integrations (Groq, Google Maps, Twilio), including retries, graceful fallbacks, and structured error reporting.

- **Add lightweight rate-limiting or session tracking** to prevent misuse, control backend load, and maintain a safer interaction flow.

- **Expand automated test coverage**, focusing particularly on:
  - Tool invocation logic
  - Failure scenarios (e.g., geocoding failures, API throttling, network timeouts)
  - End-to-end agent behavior under edge cases

- **Provide an in-app safety disclaimer**, clarifying that BrainMate AI is not a substitute for emergency or professional mental-health services and outlining appropriate usage boundaries for users.





## ⚠️ Disclaimer

**BrainMate AI is a technical demonstration**, showcasing how to integrate LLM tooling, Twilio services, and Google Maps–based location features within a unified agent framework.

It is **not** a medical product and **must not** be used as a substitute for licensed mental-health professionals, clinical therapy, or emergency assistance.

If you or someone you know is experiencing a crisis or is in immediate danger, **contact your local emergency services or a recognized crisis hotline immediately**.

- Uses `call_emergency()` (from `custom_tools.py`) to trigger a Twilio voice call to a safety helpline.
    - `find_nearby_therapists_by_location(location: str) -> str`
      - Uses the Google Maps API to geocode a location and return nearby therapists (name, address, phone).
  - Configures the LLM:
    - Uses `ChatGroq` with model `"openai/gpt-oss-120b"` and `GROQ_API_KEY` from `config.py`.
    - Creates a REAct agent via `create_react_agent(llm, tools=tools)`.
  - Defines a `SYSTEM_PROMPT` with instructions for when to use each tool.
  - Provides `parse_response(stream)` to extract the final message and which tool was called from the streaming agent output.

- **`backend/config.py`** (user-generated, not shown)
  - Should provide configuration values such as:
    - `GROQ_API_KEY`
    - `GOOGLE_MAPS_API_KEY`
    - Twilio credentials (e.g., `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, phone numbers)
  - You create this file yourself and **do not commit your secrets**.

- **`backend/custom_tools.py`** (user-generated, not shown)
  - Implements the concrete integrations:
    - `query_medgemma(query: str) -> str`
    - `call_emergency() -> None`
    - Optional helpers for Google Maps / Twilio






## ⚖️ License

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](https://choosealicense.com/licenses/mit/)

