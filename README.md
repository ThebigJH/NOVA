# NOVA

NOVA is a real Python AI assistant built for the Nebius × NVIDIA Global AI Hackathon. It uses the Nebius Token Factory OpenAI-compatible API to talk to NVIDIA Nemotron models, stores conversation history in SQLite, and exposes a modular agent architecture with safe tools for math and time/date.

## 1. What NOVA is

NOVA is designed to feel like a polished, production-style assistant rather than a basic mock chatbot. It includes:

- a modern FastAPI web interface
- streaming AI responses
- persistent conversation memory in SQLite
- a tool-aware agent loop
- a safe calculator tool
- a time/date tool
- graceful handling for missing keys, rate limits, and provider failures

## 2. Architecture overview

The project is split into clear modules:

- app/main.py: FastAPI app entrypoint and frontend routing
- app/config.py: environment-based configuration
- app/ai/client.py: Nebius OpenAI-compatible model client
- app/ai/model.py: prompt assembly and message construction
- app/agent/agent.py: orchestration for tool selection and response generation
- app/agent/tool_registry.py: tool detection layer
- app/tools/: tool implementations
- app/memory/database.py: SQLite schema management
- app/memory/conversation.py: conversation storage API
- app/api/chat.py: chat streaming endpoint
- app/api/conversations.py: conversation management endpoints
- app/web/: frontend templates and static assets

## 3. Requirements

- Python 3.11+
- A Nebius account and API key
- Access to a Nemotron model through Nebius Token Factory
- Local environment capable of running FastAPI and SQLite

## 4. Installation

```bash
python -m venv .venv
. .venv/bin/activate  # macOS/Linux
# or .\.venv\Scripts\activate  # Windows PowerShell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## 5. Environment variable setup

Copy the example config and fill in your real values:

```bash
cp .env.example .env
```

Then edit .env:

```env
NEBIUS_API_KEY=your_key_here
NEBIUS_MODEL=nvidia/Nemotron-4-340B-Instruct
NEBIUS_BASE_URL=https://api.studio.nebius.ai/v1
MAX_CONTEXT_MESSAGES=12
MAX_MESSAGE_CHARS=12000
```

Note: the API key is never committed to the repository and is read only from environment variables.

## 6. How to start the server

```bash
python -m uvicorn app.main:app --reload
```

Then open:

```text
http://127.0.0.1:8000
```

## 7. How to use NOVA

1. Start the server.
2. Open the browser UI.
3. Ask a question or select a quick prompt.
4. Responses stream progressively in the UI.
5. Use New chat to start a fresh conversation.
6. Each conversation is persisted in SQLite.

## 8. How the Nebius API integration works

NOVA uses the OpenAI-compatible API style expected by Nebius Token Factory. The app reads the following from environment variables:

- NEBIUS_API_KEY
- NEBIUS_MODEL
- NEBIUS_BASE_URL

The backend creates a standard OpenAI client pointed at the Nebius base URL, then sends chat completion requests using the model name configured in the environment. If the API is unavailable, rate-limited, timed out, or malformed, the app returns a user-friendly error instead of crashing.

## 9. How Nemotron is used

The model name is configurable via NEBIUS_MODEL, so you can swap between supported Nemotron models without rewriting code. The app is designed to work with the official OpenAI-compatible endpoint pattern exposed by Nebius, which keeps the integration clean and future-proof.

## 10. Tool architecture

NOVA includes a lightweight tool interface and registry:

- CalculatorTool: safely evaluates arithmetic expressions using Python's AST with restricted allowed functions and operators
- TimeTool: returns the current date and time
- ToolRegistry: selects tools based on user intent

This allows the assistant to act as a tool-aware agent rather than a pure text-only chatbot.

## 11. Security notes

The project follows several basic protections:

- no hardcoded credentials
- frontend never receives API keys
- message validation prevents empty and oversized prompts
- unsafe Python execution is blocked
- shell execution is not exposed
- CORS is configured conservatively
- .env is ignored in git

## 12. Troubleshooting

### Missing API key

If you see a friendly error like "Unable to reach NOVA right now", make sure your .env contains valid values for NEBIUS_API_KEY and NEBIUS_MODEL.

### Model errors

If the model name is wrong or the endpoint is unavailable, check the following:

- the model name exists in Nebius Token Factory
- the API key is valid
- the base URL matches the current OpenAI-compatible Nebius endpoint

### Database issues

Delete the local SQLite file if needed and restart the app:

```bash
rm nova.db
```

## 13. How to change models

Edit the value in .env:

```env
NEBIUS_MODEL=nvidia/Nemotron-4-340B-Instruct
```

The app reads that value at runtime, so you can change models without altering the core Python code.

## 14. Additional notes

This project intentionally stays modular so future features such as web search, file analysis, document Q&A, long-term memory, multi-agent architecture, and deployment-ready integration can be added without rewriting the application.
